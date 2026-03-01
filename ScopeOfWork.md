รายงานขอบเขตงาน (Scope of Work): โครงการ Enterprise AI Platform By CTS

1. บทนำและวัตถุประสงค์เชิงกลยุทธ์ (Project Overview & Strategic Objectives)

โครงการ Enterprise AI Platform โดย CTS ถูกออกแบบมาเพื่อเป็น Strategic Technical Enablement สำหรับองค์กรขนาดใหญ่ โดยมุ่งเป้าไปที่กลุ่ม Enterprise Architects และ Platform Engineers เพื่อทำลายข้อจำกัดเดิมๆ ของการจัดการทรัพยากร AI คำมั่นสัญญาหลักของแพลตฟอร์ม (Core Promise) วางอยู่บน 3 แกนหลัก: Governance, FinOps และ Self-Service

Design Rationale & Strategic Shift: ยุทธศาสตร์สถาปัตยกรรมนี้มุ่งเน้นการเปลี่ยนผ่านจาก "Ticket-based Requests" ที่ล่าช้า ไปสู่ "Self-service Consumption" ที่รวดเร็วและปลอดภัย โดยมีเหตุผลในการออกแบบ (Design Rationale) เพื่อ Isolate Blast Radius ของระบบ AI, รวมศูนย์การจัดการต้นทุน (Centralized Cost), และรักษามาตรฐานการทำงานที่สอดคล้องกับ FSI Standards (Financial Services Industry) ซึ่งเป็นมาตรฐานความปลอดภัยสูงสุดสำหรับองค์กรการเงิน

2. โครงสร้างพื้นฐานทางสถาปัตยกรรม: The 6-Layer Stack

ทรัพยากรภายใน AI Platform Account จะถูกบริหารจัดการผ่านโครงสร้าง 6 ชั้น (Layer Cake) โดยมี Platform Engineering Team เป็นผู้ดูแลรับผิดชอบทั้งหมด (Layers 1-6) เพื่อให้เกิดมาตรฐานความปลอดภัยและประสิทธิภาพสูงสุด

ชั้น (Layer)	ชื่อชั้น (Layer Name)	ส่วนประกอบสำคัญและบริการ AWS (Key Components/AWS Services)
Layer 6	Observability & FinOps	Central logs, Cost allocation tags
Layer 5	AI Runtime Layer	Amazon Bedrock, Amazon SageMaker, Amazon EKS, Vector DBs
Layer 4	AI Gateway Layer	Multi-tenant metering, AuthZ, Rate limiting
Layer 3	Provisioning Layer	GitOps, Terraform, Policy-as-Code
Layer 2	Portal / IDP Layer	Web Console & Marketplace (The Face)
Layer 1	Identity & Access	AWS IAM Identity Center (SSO), RBAC mapping


--------------------------------------------------------------------------------


3. ขอบเขตความรับผิดชอบรายทีม (Scope of Work by Team)

3.1 ทีม Cloud Infrastructure (Platform & Foundation)

รับผิดชอบการสร้างและดูแลระบบนิเวศ Cloud ทั้งหมดในฐานะ "Provider":

* จัดตั้ง AWS Landing Zone Foundation และวาง Multi-account Strategy โดยใช้ AWS Control Tower, Log Archive และ Shared Services
* บริหารจัดการ AI Platform Account เพื่อทำหน้าที่เป็น Hub สำหรับการให้บริการ AI ไปยัง Spoke App Account A และ Spoke App Account B ผ่านกลไก API Consumption
* ดูแลรับผิดชอบ Layer 1, 2, 3 และ 6 ของระบบ
* จัดการด้าน Networking: ออกแบบและดูแล Network Interface, VPC Peering หรือ Transit Gateway เพื่อให้ Spoke Accounts สามารถเชื่อมต่อเข้าใช้งาน AI Services ได้อย่างปลอดภัยภายใต้ Security Guardrails

3.2 ทีม Data Engineering (Data Pipeline & Processing)

รับผิดชอบการเตรียมทรัพยากรข้อมูลที่ใช้ขับเคลื่อนโมเดล AI:

* สร้าง Data Ingestion & Processing Layer (Clean & Transform) เพื่อจัดการข้อมูลระดับ Enterprise
* บริหารจัดการ Vector Databases และ Knowledge Bases ภายใน Amazon Bedrock เพื่อรองรับการสืบค้นข้อมูลที่แม่นยำ (Vector Search)
* เชื่อมต่อกับแหล่งข้อมูลภายในองค์กร (Enterprise Data Sources) เพื่อสร้างท่อข้อมูล (Data Pipelines) สำหรับ AI

3.3 ทีม AI/ML (Model & Inference)

รับผิดชอบวงจรชีวิตของโมเดลและการให้บริการประมวลผล (Inference):

* บริหารจัดการ Model Training & Development Layer (Train & Fine-tune) ผ่าน Amazon SageMaker
* ดูแล AI Runtime Layer (Layer 5) โดยใช้ Amazon Bedrock เป็นหัวใจหลักในการดึงความสามารถของ Foundation Models
* ออกแบบและจัดการ AI Gateway Layer (Layer 4) เพื่อทำระบบ Metering และควบคุมการเข้าถึงโมเดลในระดับ Multi-tenant

3.4 ทีม Application Development (Agent & Consumption)

รับผิดชอบการสร้าง User Experience และ AI Agents สำหรับธุรกิจ:

* พัฒนา AI Agent & Orchestration Layer (Orchestrate & Automate)
* สร้าง Unified Consumption Layer ซึ่งประกอบด้วย:
  * Chat/Assistant: ระบบ RAG (Retrieval-Augmented Generation) และ Knowledge Bases
  * Document Intelligence: ความสามารถด้าน Summarization, Extraction และ Classification
  * Developer Copilot: เครื่องมือช่วยเหลือด้าน Code assistance และ DevSecOps automation (รวมถึง PR Reviewer bots)
  * Enterprise Integration: เชื่อมต่อ Workflow bots เข้ากับระบบ CRM, ERP และ ITSM
* ดูแลส่วน Production & Consumption Layer ผ่านระบบ AI Marketplace


--------------------------------------------------------------------------------


4. ยุทธศาสตร์การบริหารจัดการต้นทุน (FinOps & Cost Model)

เราใช้ 4-Layer Cost Model เพื่อเปลี่ยนจากการเก็บเงินแบบเหมารวม (Opaque Bills) ไปสู่การเรียกเก็บตามการใช้งานจริง (Precise Chargeback) โดยหัวใจสำคัญคือ "Granularity Increases"—ยิ่งลงลึกถึงชั้นบนสุด ความละเอียดของการติดตามต้นทุนจะยิ่งสูงขึ้น

1. Account-level: โครงสร้างพื้นฐานร่วม (Shared infra เช่น Amazon EKS, Network, Logs)
2. Service-level: ทรัพยากรแยกตามการติดตั้ง (Per-deployment resources เช่น Vector DBs)
3. Inference-level: ต้นทุนราย Token (Per-token costs ของ Amazon Bedrock และ Amazon SageMaker)
4. Pipeline-level: ค่าใช้จ่ายในการสร้างระบบครั้งเดียว (One-time build costs)

กลไกการควบคุม (Mechanics of Control):

* Tagging Strategy: บังคับใช้ Tag พื้นฐาน (CostCenter, Team, App, Agent) ในทุกทรัพยากร
* Bedrock Innovation: ใช้ Application Inference Profiles เพื่อติดตามการเรียกใช้โมเดลราย Tenant/Team เพื่อการจัดสรรงบประมาณที่แม่นยำ
* Gateway Metering: ใช้ Internal SaaS Meter ภายใน Gateway เพื่อตรวจสอบการใช้งานจริง (Regulate access and billing)


--------------------------------------------------------------------------------


5. กระบวนการส่งมอบงานในรูปแบบ Industrialized Software Delivery

การส่งมอบงานจะทำผ่าน Internal Developer Platform (IDP) ซึ่งเปลี่ยนการทำงานแบบ Manual เป็นระบบอัตโนมัติ โดยมี Terraform เป็นเครื่องยนต์หลัก (The Engine):

1. Select: ผู้ใช้เลือก Blueprint ที่ต้องการจาก Marketplace
2. Generate: แพลตฟอร์มจะทำการสร้าง Terraform Code โดยอัตโนมัติจาก Blueprint ที่เลือก
3. Commit: โค้ดที่สร้างขึ้นจะถูกส่งไปยังระบบ Version Control (Git)
4. Deploy: ระบบ Pipeline ทำการติดตั้งทรัพยากรลงบน AWS (Amazon EKS / Amazon Bedrock) ทันที

คติพจน์ของระบบคือ: “You don’t build servers; you consume services.”


--------------------------------------------------------------------------------


6. รูปแบบประสบการณ์ผู้ใช้ (The User Experience: AI Marketplace)

AI Platform Console จะเป็นจุดศูนย์กลาง (Centralized Interface) ที่ลดความซับซ้อนของเทคโนโลยี AWS ให้เป็นสิ่งที่เข้าถึงได้ง่ายสำหรับทุก Persona:

* Golden Paths: เทมเพลตมาตรฐานที่ผ่านการตั้งค่าด้าน Compliance ไว้ล่วงหน้า (Slate Grey: Pre-configured used compliance)
* Taxonomy: การจัดหมวดหมู่บริการตามบทบาทของผู้ใช้ (Slate Grey: Organized by User Persona) เช่น Business, Ops, Developer หรือ Data/AI
* One-Click Provisioning: ลดความซับซ้อนของ Infrastructure ทั้งหมดให้เหลือเพียงการคลิก (Slate Grey: Abstracts AWS complexity)

ตัวอย่าง Use Cases ที่พร้อมส่งมอบ:

* PR Reviewer Bot (Developer): ระบบตรวจสอบโค้ดและสแกนความปลอดภัยอัตโนมัติ
* HR Q&A Agent (Business): ระบบตอบคำถามจากคู่มือพนักงานผ่าน RAG-based assistant
* Incident Bot (Ops): ระบบเชื่อมต่อ Slack สำหรับจัดการ Incident ระดับ P1 อัตโนมัติ โดยเชื่อมโยงกับระบบ ITSM
