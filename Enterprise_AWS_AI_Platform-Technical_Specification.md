# CTS-AWS-AI-Platform


# Enterprise AWS AI Platform - Technical Specification
## 6-Layer Stack Architecture

**Document Version:** 1.0  
**Date:** March 2026  
**Compliance:** AWS Well-Architected Framework (Machine Learning Lens)

---

## Executive Summary

This document outlines the technical architecture for an Enterprise-grade AWS AI Platform built on a 6-layer stack model. The platform provides secure, scalable, and cost-optimized infrastructure for deploying foundation models (LLMs) and custom ML workloads while maintaining enterprise security standards and data governance.

---

## Architecture Overview

### Design Principles
- **Zero Trust Security**: All inter-service communication via VPC PrivateLink
- **Data Residency**: No data traverses public internet
- **Role-Based Access Control (RBAC)**: Separation between Data Scientists and App Developers
- **Cost Transparency**: Tag-based cost allocation per team/project
- **Observability**: Full-stack monitoring from API to model inference

---

## Layer 1: Identity & Access Layer

### Components

#### 1.1 Microsoft Active Directory Integration
- **Service**: AWS Managed Microsoft AD
- **Purpose**: Enterprise identity source
- **Configuration**:
  - Domain: `corp.enterprise.com`
  - Edition: Enterprise (supports 500,000+ objects)
  - Multi-AZ deployment for HA

#### 1.2 IAM Identity Center (AWS SSO)
- **Purpose**: Centralized access management
- **Integration**: Federated with Microsoft AD via SAML 2.0
- **Features**:
  - Single Sign-On across AWS accounts
  - MFA enforcement
  - Session duration: 8 hours

#### 1.3 RBAC Policy Design

**Data Scientist Role** (`DataScientistRole`)
```
Permissions:
- SageMaker: Full access (notebooks, training, endpoints)
- S3: Read/Write to Silver & Gold layers
- Glue: Read access to Data Catalog
- Bedrock: InvokeModel, ListFoundationModels
- CloudWatch: Read metrics
- Cost Explorer: View own team's costs

Restrictions:
- No IAM policy modifications
- No VPC/networking changes
- No production endpoint deletion
```

**App Developer Role** (`AppDeveloperRole`)
```
Permissions:
- API Gateway: Deploy/manage APIs
- Lambda: Full access
- Bedrock: InvokeModel only
- SageMaker: InvokeEndpoint only (no training)
- S3: Read-only to Gold layer
- CloudWatch: Read logs/metrics

Restrictions:
- No access to training data (Bronze/Silver)
- No model training permissions
- No data catalog modifications
```

---

## Layer 2: Portal / IDP Layer

### Components

#### 2.1 AWS Amplify (Internal Portal)
- **Purpose**: Custom web portal for AI platform access
- **Features**:
  - Model catalog browser
  - API key management
  - Cost dashboard per user
  - Request history
- **Hosting**: Amplify Hosting with CloudFront CDN
- **Authentication**: Cognito User Pool federated with IAM Identity Center

#### 2.2 Amazon QuickSight
- **Purpose**: AI Performance Dashboards
- **Dashboards**:
  1. **Model Performance**: Latency, throughput, error rates
  2. **Cost Analytics**: Token usage, inference costs by team
  3. **Data Quality**: Data drift detection, feature statistics
  4. **Usage Patterns**: API calls by endpoint, peak hours
- **Data Sources**: 
  - CloudWatch Metrics
  - S3 (Gold layer analytics)
  - Cost and Usage Reports

#### 2.3 Internal Developer Platform (IDP)
- **Purpose**: Self-service AI/ML workflows
- **Capabilities**:
  - Model deployment pipelines
  - Feature store management
  - Experiment tracking
  - Documentation hub
- **Implementation**: Custom portal on Amplify + API Gateway

---

## Layer 3: Provision Layer (Data & Vector)

### 3.1 S3 Data Lake (Medallion Architecture)

#### Bronze Layer (Raw Data)
- **Bucket**: `s3://enterprise-ai-bronze-{account-id}`
- **Purpose**: Raw, immutable data ingestion
- **Format**: Original formats (JSON, CSV, Parquet, logs)
- **Retention**: 90 days
- **Encryption**: SSE-KMS with customer-managed key
- **Access**: Data Engineers only

#### Silver Layer (Cleaned Data)
- **Bucket**: `s3://enterprise-ai-silver-{account-id}`
- **Purpose**: Validated, cleaned, deduplicated data
- **Format**: Parquet with Snappy compression
- **Retention**: 1 year
- **Partitioning**: By date (`year=YYYY/month=MM/day=DD`)
- **Access**: Data Scientists (read/write), App Developers (no access)

#### Gold Layer (Curated Data)
- **Bucket**: `s3://enterprise-ai-gold-{account-id}`
- **Purpose**: Business-ready, aggregated datasets
- **Format**: Parquet optimized for analytics
- **Retention**: 3 years
- **Access**: All roles (read-only for App Developers)
- **Features**:
  - Versioning enabled
  - Cross-region replication for DR

### 3.2 Amazon OpenSearch Serverless (Vector Database)

#### Configuration
- **Purpose**: Vector embeddings for RAG (Retrieval-Augmented Generation)
- **Collection Type**: Vector search
- **Capacity**: Auto-scaling (4-32 OCUs)
- **Index Strategy**:
  - Dimension: 1536 (OpenAI/Bedrock Titan embeddings)
  - Engine: FAISS
  - Similarity: Cosine

#### Use Cases
- Semantic search over enterprise documents
- Context retrieval for LLM prompts
- Knowledge base for chatbots

### 3.3 Orchestration

#### AWS Step Functions
- **Purpose**: ETL and AI workflow orchestration
- **Workflows**:
  1. **Data Ingestion Pipeline**: Bronze → Silver → Gold
  2. **Model Training Pipeline**: Data prep → Training → Evaluation → Deployment
  3. **Batch Inference Pipeline**: Load data → Invoke model → Store results
- **Error Handling**: Exponential backoff with SNS alerts

#### AWS Glue ETL
- **Jobs**:
  - Data quality checks (Great Expectations)
  - PII detection and masking
  - Schema evolution handling
  - Feature engineering
- **Execution**: Glue 4.0 with Python 3.10
- **Scheduling**: EventBridge rules (hourly/daily)

---

## Layer 4: AI Gateway Layer

### Components

#### 4.1 AWS WAF (Web Application Firewall)
- **Purpose**: Protect against prompt injection and abuse
- **Rules**:
  1. **Rate Limiting**: 1000 requests/5min per IP
  2. **Prompt Injection Detection**: Block SQL injection patterns, script tags
  3. **Geo-blocking**: Allow only corporate IP ranges
  4. **Size Limits**: Max payload 100KB
- **Logging**: All blocked requests to S3 for analysis

#### 4.2 Amazon API Gateway
- **Type**: REST API (Regional)
- **Endpoints**:
  - `/bedrock/invoke` - Foundation model inference
  - `/sagemaker/invoke` - Custom model inference
  - `/embeddings` - Vector embedding generation
  - `/search` - RAG semantic search
- **Features**:
  - Request/response validation
  - API keys for external apps
  - Usage plans with throttling
  - CloudWatch Logs (full request/response)

#### 4.3 AWS Lambda Authorizer
- **Purpose**: Custom authentication and authorization
- **Logic**:
  1. Validate JWT token from IAM Identity Center
  2. Extract user role (Data Scientist / App Developer)
  3. Check permissions for requested endpoint
  4. Enforce quota limits per user
  5. Return IAM policy document
- **Caching**: 5 minutes (reduces latency)

#### 4.4 Throttling & Logging
- **Throttling**:
  - Burst: 5000 requests
  - Steady-state: 10000 requests/second
  - Per-user quota: 1000 requests/hour
- **Logging**:
  - CloudWatch Logs: Full request/response
  - S3: Long-term storage (compressed)
  - Metrics: Latency, 4xx/5xx errors, token usage

---

## Layer 5: AI Runtime Layer

### 5.1 Amazon Bedrock (Foundation Models)

#### Supported Models
- **Anthropic Claude 3 (Opus, Sonnet, Haiku)**
- **Amazon Titan (Text, Embeddings)**
- **Meta Llama 3**
- **Cohere Command**

#### Configuration
- **Inference**: On-demand (no provisioned throughput initially)
- **Guardrails**:
  - Content filtering (hate, violence, sexual)
  - PII redaction
  - Topic blocking (competitors, financials)
- **Logging**: Model invocations to CloudWatch

#### Use Cases
- Conversational AI / Chatbots
- Document summarization
- Code generation
- Sentiment analysis

### 5.2 Amazon SageMaker (Custom Models)

#### SageMaker Notebooks
- **Instance Type**: ml.t3.xlarge (development), ml.p3.2xlarge (experimentation)
- **Lifecycle Config**: Auto-stop after 1 hour idle
- **Access**: Data Scientists only
- **Git Integration**: CodeCommit for notebook versioning

#### SageMaker Training
- **Instance Types**: ml.p4d.24xlarge (large models), ml.g5.xlarge (fine-tuning)
- **Spot Instances**: Enabled (70% cost savings)
- **Checkpointing**: S3 every 10 minutes
- **Distributed Training**: Data parallelism with SageMaker distributed

#### SageMaker Endpoints
- **Deployment**:
  - Multi-model endpoints (cost optimization)
  - Auto-scaling: 2-10 instances based on invocations
  - Blue/green deployments for zero-downtime updates
- **Monitoring**: Model Monitor for data drift detection

### 5.3 AWS Glue (Feature Store)

#### Glue Data Catalog
- **Purpose**: Centralized metadata repository
- **Tables**:
  - Feature definitions
  - Data lineage
  - Schema versions
- **Integration**: Athena, SageMaker Feature Store

#### Feature Engineering
- **Glue Jobs**: Real-time feature extraction
- **Features**:
  - Customer embeddings
  - Aggregated metrics (7-day, 30-day windows)
  - Categorical encodings
- **Storage**: S3 (Gold layer) + DynamoDB (online features)

---

## Layer 6: Observability & FinOps

### 6.1 Amazon CloudWatch (Model Monitor)

#### Metrics
- **Bedrock**:
  - Invocations, latency (p50, p99)
  - Token input/output counts
  - Throttling errors
- **SageMaker**:
  - Endpoint invocations, latency
  - Model accuracy (custom metrics)
  - Instance CPU/memory utilization

#### Alarms
- Latency > 5 seconds (P99)
- Error rate > 1%
- Cost spike > 20% daily increase

#### Dashboards
- Real-time model performance
- API Gateway request patterns
- Data pipeline health

### 6.2 AWS DataZone (Data Governance)

#### Capabilities
- **Data Catalog**: Searchable inventory of all datasets
- **Business Glossary**: Standardized terminology
- **Data Lineage**: Track data from source to model
- **Access Management**: Approve/deny data access requests
- **Quality Scores**: Automated data quality metrics

#### Governance Policies
- PII data requires approval
- Production data access logged
- Data retention enforcement

### 6.3 AWS Cost Explorer & Budgets

#### Tagging Strategy
```
Tags:
- Team: data-science | app-dev | platform
- Project: chatbot | recommendation | fraud-detection
- Environment: dev | staging | prod
- CostCenter: engineering | product
```

#### Cost Allocation
- **Bedrock**: Token-based pricing tracked per API call
- **SageMaker**: Training hours, endpoint hours
- **S3**: Storage by layer (Bronze/Silver/Gold)
- **Data Transfer**: VPC endpoint charges

#### Budgets
- Monthly budget per team
- Alerts at 80%, 100%, 120% thresholds
- Automatic notifications to team leads

---

## Networking Architecture

### VPC Design

#### Configuration
- **CIDR**: 10.0.0.0/16
- **Subnets**:
  - Private Subnet AZ-A: 10.0.1.0/24
  - Private Subnet AZ-B: 10.0.2.0/24
  - Private Subnet AZ-C: 10.0.3.0/24
- **No Public Subnets**: All resources in private subnets
- **NAT Gateway**: Not required (VPC Endpoints for all AWS services)

### VPC Endpoints (PrivateLink)

#### Interface Endpoints
- `com.amazonaws.{region}.bedrock-runtime`
- `com.amazonaws.{region}.sagemaker.runtime`
- `com.amazonaws.{region}.s3`
- `com.amazonaws.{region}.glue`
- `com.amazonaws.{region}.execute-api` (API Gateway)
- `com.amazonaws.{region}.logs` (CloudWatch)
- `com.amazonaws.{region}.sts` (IAM)

#### Gateway Endpoints
- S3 (no additional cost)
- DynamoDB (no additional cost)

#### Benefits
- **Security**: No internet gateway required
- **Performance**: Lower latency via AWS backbone
- **Compliance**: Data never leaves AWS network
- **Cost**: Reduced data transfer charges

### Security Groups

#### API Gateway Security Group
```
Inbound:
- Port 443 from Amplify Security Group
- Port 443 from QuickSight IP range

Outbound:
- Port 443 to Bedrock VPC Endpoint
- Port 443 to SageMaker VPC Endpoint
```

#### SageMaker Security Group
```
Inbound:
- Port 443 from API Gateway Security Group
- Port 8888 from Data Scientist IPs (notebooks)

Outbound:
- Port 443 to S3 VPC Endpoint
- Port 443 to Glue VPC Endpoint
```

---

## Security & Compliance

### Encryption

#### At Rest
- **S3**: SSE-KMS with customer-managed CMK
- **EBS**: Encrypted with default AWS-managed key
- **RDS/DynamoDB**: Encryption enabled
- **Secrets Manager**: Automatic rotation every 30 days

#### In Transit
- **TLS 1.3**: All API communications
- **VPC Endpoints**: Encrypted via PrivateLink
- **Certificate**: ACM-managed certificates

### Audit & Compliance

#### AWS CloudTrail
- **Logging**: All API calls across all layers
- **Storage**: S3 with MFA delete protection
- **Retention**: 7 years
- **Integration**: Security Hub, GuardDuty

#### Compliance Frameworks
- **SOC 2 Type II**: Annual audit
- **GDPR**: Data residency in EU regions
- **HIPAA**: PHI data encryption and access controls

---

## Disaster Recovery & High Availability

### RTO/RPO Targets
- **RTO**: 4 hours
- **RPO**: 1 hour

### Strategies

#### Multi-AZ Deployment
- All services deployed across 3 AZs
- Automatic failover for managed services

#### Backup Strategy
- **S3**: Cross-region replication to DR region
- **Models**: Versioned in S3 with lifecycle policies
- **Databases**: Automated snapshots every 6 hours

#### DR Region
- **Primary**: us-east-1
- **DR**: us-west-2
- **Replication**: Asynchronous for S3, synchronous for critical metadata

---

## Cost Estimation (Monthly)

### Assumptions
- 10 Data Scientists, 20 App Developers
- 1M API calls/month
- 100GB data ingestion/day
- 5 SageMaker endpoints (ml.g5.xlarge)

### Breakdown
| Service | Cost (USD) |
|---------|-----------|
| Bedrock (Claude 3 Sonnet, 10M tokens) | $3,000 |
| SageMaker Endpoints (5x ml.g5.xlarge) | $2,160 |
| SageMaker Notebooks (10x ml.t3.xlarge) | $720 |
| S3 Storage (10TB) | $230 |
| OpenSearch Serverless (8 OCUs) | $1,440 |
| API Gateway (1M requests) | $3.50 |
| VPC Endpoints (10 endpoints) | $72 |
| CloudWatch Logs (100GB) | $50 |
| Data Transfer (PrivateLink) | $150 |
| **Total** | **~$7,825/month** |

---

## Implementation Roadmap

### Phase 1: Foundation (Weeks 1-4)
- [ ] VPC and networking setup
- [ ] IAM Identity Center + AD integration
- [ ] S3 Data Lake (Bronze/Silver/Gold)
- [ ] Basic monitoring (CloudWatch)

### Phase 2: Data Platform (Weeks 5-8)
- [ ] Glue ETL jobs
- [ ] Step Functions orchestration
- [ ] OpenSearch Serverless setup
- [ ] DataZone catalog

### Phase 3: AI Runtime (Weeks 9-12)
- [ ] Bedrock integration
- [ ] SageMaker environment setup
- [ ] API Gateway + Lambda Authorizer
- [ ] WAF rules

### Phase 4: Portal & Observability (Weeks 13-16)
- [ ] Amplify portal deployment
- [ ] QuickSight dashboards
- [ ] Cost allocation tags
- [ ] Security audit

---

## Appendix

### A. AWS Well-Architected Framework Alignment

#### Operational Excellence
- Infrastructure as Code (CloudFormation/Terraform)
- Automated deployments via CI/CD
- Runbooks for common operations

#### Security
- Zero Trust architecture
- Least privilege access (RBAC)
- Encryption everywhere

#### Reliability
- Multi-AZ deployment
- Auto-scaling
- Automated backups

#### Performance Efficiency
- Right-sizing instances
- Serverless where possible
- Caching strategies

#### Cost Optimization
- Spot instances for training
- S3 lifecycle policies
- Reserved instances for steady-state workloads

#### Sustainability
- Serverless reduces idle resources
- Auto-scaling minimizes waste
- Efficient data formats (Parquet)

### B. Reference Architecture Diagram
See: `enterprise_ai_platform_detailed.png`

### C. Contact Information
- **Architecture Team**: ai-platform-arch@enterprise.com
- **Support**: ai-platform-support@enterprise.com
- **Security**: security@enterprise.com

---

**Document End**
