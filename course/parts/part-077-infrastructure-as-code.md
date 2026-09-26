# Part 77: Infrastructure as Code ร่วมกับ Git (Terraform + Git)

> **Step ในหลักสูตรนี้:** Step 761–770
> **เฟส:** 8 — DevOps, Security, Compliance ระดับองค์กร
> **เป้าหมายของ Part นี้:** เข้าใจแนวคิด Infrastructure as Code (IaC) อย่างลึกซึ้งว่ามันแก้ปัญหาอะไรที่ทีม Ops เจอมานาน จากนั้นลงมือใช้ **Terraform** เครื่องมือ IaC ที่ได้รับความนิยมที่สุดในโลก ตั้งแต่การเขียนไฟล์ `.tf` เบื้องต้น การทำงานผ่าน workflow `init/plan/apply/destroy` ความเข้าใจเรื่อง state file และ remote state ไปจนถึงการนำ Terraform เข้าไปอยู่ใน Git workflow และ CI/CD pipeline อย่างมืออาชีพ — ต่อยอดจากแนวคิด GitOps ที่เรียนไปใน Part 76 ที่ Git คือ "แหล่งความจริงเดียว" (single source of truth) ของทั้งแอปพลิเคชันและโครงสร้างพื้นฐาน

---

## สารบัญของ Part นี้

- Step 761: Infrastructure as Code (IaC) คืออะไร แก้ปัญหาอะไร
- Step 762: เครื่องมือ IaC ยอดนิยม (Terraform, Pulumi, CloudFormation, Ansible)
- Step 763: Terraform เบื้องต้น — ไฟล์ `.tf`, `provider`, `resource` block
- Step 764: `terraform init` / `plan` / `apply` / `destroy` — Workflow เต็มรูปแบบ
- Step 765: State File คืออะไร ทำไมถึงสำคัญมาก และห้าม commit เข้า Git
- Step 766: Remote State และ State Locking — ทำงานเป็นทีมอย่างปลอดภัย
- Step 767: การจัดการ Terraform Code ด้วย Git (Branch, PR Review ก่อน Apply จริง)
- Step 768: Terraform ใน CI/CD Pipeline (Plan ใน PR, Apply ตอน Merge)
- Step 769: Best Practices — Module, Workspace, ห้าม Hardcode Secret
- Step 770: แบบฝึกหัด — เขียน Terraform Config ผ่าน Git Branch/PR และรัน Pipeline จำลอง

---

## Step 761: Infrastructure as Code (IaC) คืออะไร แก้ปัญหาอะไร

ตลอดหลักสูตรนี้เราใช้เวลาส่วนใหญ่พูดถึงการจัดการ **โค้ดแอปพลิเคชัน** ด้วย Git แต่ในโลกความจริง แอปพลิเคชันจะรันไม่ได้เลยถ้าไม่มี **โครงสร้างพื้นฐาน (infrastructure)** รองรับ — เซิร์ฟเวอร์, เครือข่าย, ฐานข้อมูล, load balancer, DNS, firewall rules ฯลฯ คำถามคือ: แล้วโครงสร้างพื้นฐานเหล่านี้ถูกสร้างและจัดการอย่างไร?

### วิธีดั้งเดิม: จัดการ Infrastructure ด้วยมือ (ClickOps)

ก่อนที่ IaC จะเป็นที่นิยม ทีม Ops ส่วนใหญ่จัดการ infrastructure ด้วยวิธีที่เรียกกันติดตลกว่า **"ClickOps"** — คือการคลิกเข้าไปตั้งค่าทีละอย่างผ่านหน้าเว็บ console ของ cloud provider (AWS Console, Azure Portal, GCP Console) หรือ SSH เข้าเครื่อง server แล้วรันคำสั่งตั้งค่าด้วยมือทีละขั้นตอน

ลองนึกภาพสถานการณ์นี้:

> วิศวกรคนหนึ่งชื่อ "พี่เอ" ต้องการเซิร์ฟเวอร์ใหม่สำหรับรันแอปพลิเคชัน เขา SSH เข้าเครื่อง แล้วรันคำสั่งติดตั้ง package ทีละตัว ปรับ config ไฟล์หลายจุดด้วยมือ เปิด firewall port บางพอร์ตเป็นกรณีพิเศษเพื่อแก้ปัญหาเฉพาะหน้าที่เจอตอนนั้น แล้วก็ลืมจดบันทึกไว้ที่ไหนเลย

ผ่านไป 8 เดือน พี่เอลาออก เซิร์ฟเวอร์เครื่องนั้นยังรันอยู่ ทำงานได้ดีมาก แต่ **ไม่มีใครในทีมรู้เลยว่ามันถูกตั้งค่าอย่างไรบ้าง** ไม่มีใครกล้าแตะต้องมันเพราะกลัวว่าจะพังแล้วกู้คืนไม่ได้ เซิร์ฟเวอร์เครื่องนี้ถูกเรียกกันในทีมว่า **"Snowflake Server"** (เซิร์ฟเวอร์เกล็ดหิมะ) — เพราะเหมือนเกล็ดหิมะที่ไม่มีชิ้นไหนเหมือนกันเลย เป็นของที่ไม่เหมือนใครและไม่สามารถสร้างซ้ำ (reproduce) ขึ้นมาใหม่ได้อย่างแม่นยำ

### ปัญหาที่เกิดจากการจัดการ Infrastructure ด้วยมือ

| ปัญหา | รายละเอียด |
|---|---|
| **Snowflake Server** | เซิร์ฟเวอร์แต่ละเครื่องมีการตั้งค่าที่ไม่เหมือนกัน สร้างซ้ำไม่ได้ ถ้าเครื่องพังต้องมานั่งจำ/เดาว่าเคยตั้งอะไรไว้บ้าง |
| **Configuration Drift** | การตั้งค่าจริงบนเซิร์ฟเวอร์ ("actual state") ค่อย ๆ เบี่ยงเบนไปจากสิ่งที่ตั้งใจไว้แต่แรก ("desired state") เพราะมีคนแก้ไขเล็ก ๆ น้อย ๆ ด้วยมือไปเรื่อย ๆ โดยไม่มีใครบันทึกไว้ |
| **ไม่มีประวัติการเปลี่ยนแปลง** | ไม่รู้ว่าใครแก้ค่าอะไรไปเมื่อไหร่ ทำไมถึงแก้ ต่างจากโค้ดแอปพลิเคชันที่มี Git คอยบันทึกทุกอย่าง |
| **สร้าง environment ใหม่ช้ามาก** | ถ้าต้องการสร้าง staging environment ให้เหมือน production ทุกประการ ต้องมานั่งจำ/จดขั้นตอนแล้วทำซ้ำด้วยมือ ใช้เวลาเป็นวันหรือเป็นสัปดาห์ |
| **ตรวจสอบ (audit) ยาก** | เวลาตรวจสอบความปลอดภัยหรือ compliance ต้องมานั่งไล่ดูทีละเครื่องว่าตั้งค่าอะไรไว้บ้าง ไม่มีเอกสารที่เชื่อถือได้ |
| **Human Error สูง** | การพิมพ์คำสั่งด้วยมือซ้ำ ๆ ในหลายเครื่องมีโอกาสพิมพ์ผิดหรือลืมขั้นตอนสูงมาก |
| **ไม่มี Rollback ที่แน่นอน** | ถ้าตั้งค่าใหม่แล้วพัง การย้อนกลับไปสถานะเดิมทำได้ยาก เพราะไม่มีบันทึกว่าสถานะเดิม "แน่นอน" คืออะไร |

### Infrastructure as Code (IaC) คือคำตอบ

**Infrastructure as Code (IaC)** คือแนวทางการจัดการโครงสร้างพื้นฐานโดย **เขียนเป็นโค้ด (configuration file) แทนการคลิกตั้งค่าด้วยมือ** แล้วใช้เครื่องมือมาอ่านโค้ดนั้นเพื่อสร้าง/แก้ไข/ลบ infrastructure ให้ตรงกับที่โค้ดระบุไว้โดยอัตโนมัติ

> **หลักการสำคัญของ IaC:** โครงสร้างพื้นฐานทั้งหมดถูกอธิบายไว้ในไฟล์ข้อความ (text file) ที่สามารถเก็บไว้ใน Git ได้เหมือนโค้ดแอปพลิเคชันทุกประการ — มี version, มี history, มี code review, มี rollback

เมื่อ infrastructure ถูกเขียนเป็นโค้ดแล้วเก็บไว้ใน Git ปัญหาทั้งหมดข้างต้นจะถูกแก้ไปโดยอัตโนมัติ:

| ปัญหาเดิม | สิ่งที่ IaC + Git แก้ให้ |
|---|---|
| Snowflake Server | ทุกเครื่องถูกสร้างจากโค้ดเดียวกัน รับประกันว่าเหมือนกันทุกประการ (reproducible) |
| Configuration Drift | เครื่องมือ IaC ตรวจจับได้ทันทีว่า infrastructure จริงต่างจากโค้ดที่ระบุไว้ตรงไหนบ้าง |
| ไม่มีประวัติ | ทุกการเปลี่ยนแปลงคือ commit ใน Git มีคนเขียน มีเวลา มีเหตุผลกำกับ |
| สร้าง environment ช้า | รันคำสั่งเดียวก็สร้าง environment ใหม่ที่เหมือน production เป๊ะ ๆ ได้ในไม่กี่นาที |
| Audit ยาก | โค้ดคือเอกสารที่เชื่อถือได้เสมอ (code as documentation) อ่านไฟล์ก็รู้ทันทีว่า infrastructure หน้าตาเป็นอย่างไร |
| Human Error | ลด error จากการพิมพ์คำสั่งด้วยมือ เพราะเครื่องมือรันตามโค้ดอย่างสม่ำเสมอทุกครั้ง |
| ไม่มี Rollback | `git revert` ไฟล์ config แล้วรันเครื่องมือ IaC ใหม่ ก็คืนสถานะเดิมได้ |

พูดง่าย ๆ คือ IaC นำหลักการเดียวกับที่ Git เปลี่ยนวงการพัฒนาซอฟต์แวร์ มาใช้กับงาน Ops — เปลี่ยนจาก **"จำและคลิกด้วยมือ"** เป็น **"ประกาศความต้องการเป็นโค้ด แล้วให้เครื่องมือจัดการให้"**

### แนวคิดสำคัญ: Declarative vs Imperative

IaC มีสองแนวทางหลักในการเขียน:

- **Imperative (บอกทีละขั้นตอน)** — เขียนสคริปต์บอกว่า "ทำอย่างนี้ก่อน แล้วทำอย่างนี้ต่อ" เหมือนเขียนสคริปต์ shell ธรรมดา ผู้เขียนต้องคิดเองว่าต้องรันคำสั่งอะไรบ้างตามลำดับ
- **Declarative (ประกาศผลลัพธ์ที่ต้องการ)** — เขียนบอกแค่ว่า **"ผลลัพธ์สุดท้ายที่ต้องการคืออะไร"** ส่วนเครื่องมือจะคิดเองว่าต้องทำอะไรบ้างเพื่อให้ได้ผลลัพธ์นั้น เช่น "ฉันต้องการ server 3 เครื่อง" — ถ้าตอนนี้มีอยู่ 2 เครื่อง เครื่องมือจะสร้างเพิ่มให้ 1 เครื่อง ถ้ามีอยู่ 5 เครื่อง เครื่องมือจะลบส่วนเกินออกให้ 2 เครื่อง

**Terraform ที่เราจะเรียนใน Part นี้เป็นเครื่องมือแบบ Declarative ทั้งหมด** ซึ่งเป็นแนวทางที่ได้รับความนิยมมากที่สุดในโลก IaC ปัจจุบัน เพราะทำให้ infrastructure คาดเดาผลลัพธ์ได้ง่าย ไม่ต้องกังวลเรื่องลำดับขั้นตอน

---

## Step 762: เครื่องมือ IaC ยอดนิยม (Terraform, Pulumi, AWS CloudFormation, Ansible)

ตลาดเครื่องมือ IaC มีหลายตัว แต่ละตัวมีจุดเด่นและจุดเหมาะสมกับงานต่างกัน มาดูภาพรวมสั้น ๆ ก่อนที่จะเจาะลึก Terraform ใน Step ถัดไป

### ตารางเปรียบเทียบภาพรวม

| เครื่องมือ | ประเภท | ภาษาที่ใช้เขียน | รองรับ Cloud | Agent/Agentless |
|---|---|---|---|---|
| **Terraform** | Declarative | HCL (HashiCorp Configuration Language) | หลาย cloud (AWS, Azure, GCP, และอื่น ๆ อีกกว่า 3,000 provider) | Agentless |
| **Pulumi** | Declarative | ภาษาโปรแกรมจริง (TypeScript, Python, Go, C#) | หลาย cloud เหมือน Terraform | Agentless |
| **AWS CloudFormation** | Declarative | YAML/JSON | เฉพาะ AWS เท่านั้น | Agentless |
| **Ansible** | ส่วนใหญ่ Declarative (แต่เขียนแบบ step-based ได้) | YAML (Playbook) | ทำได้ทั้ง Cloud provisioning และ Configuration management ของเครื่องที่มีอยู่แล้ว | Agentless (ใช้ SSH) |

### Terraform (โดย HashiCorp)

**Terraform** เป็นเครื่องมือ IaC ที่ได้รับความนิยมสูงสุดในอุตสาหกรรม จุดเด่นคือ:

- **Cloud-agnostic** — เขียนโค้ดจัดการได้ทั้ง AWS, Azure, GCP, Kubernetes, Cloudflare, GitHub, GitLab และอีกนับพัน provider ด้วยภาษาเดียวกันคือ HCL
- **Declarative + State-based** — เก็บ "สถานะปัจจุบัน" ไว้ใน state file แล้วเทียบกับโค้ดเพื่อคำนวณว่าต้องเปลี่ยนแปลงอะไรบ้าง (เดี๋ยวเราจะเจาะลึกเรื่องนี้ใน Step 765)
- **Ecosystem ใหญ่ที่สุด** — มี module สำเร็จรูปจากชุมชนให้ใช้งานจำนวนมหาศาลผ่าน Terraform Registry
- **เหมาะกับ:** การสร้างและจัดการ infrastructure ระดับ cloud (สร้าง server, network, database, load balancer ฯลฯ) — เป็นเครื่องมือหลักที่หลักสูตรนี้จะใช้สอน

### Pulumi

**Pulumi** ทำหน้าที่คล้าย Terraform มาก แต่จุดต่างสำคัญคือ **เขียนด้วยภาษาโปรแกรมจริง** (TypeScript, Python, Go, C#, Java) แทนภาษาเฉพาะทางอย่าง HCL

- **ข้อดี:** ใช้ loop, function, class, import library ต่าง ๆ ได้เต็มรูปแบบเหมือนเขียนโปรแกรมทั่วไป เหมาะกับทีมที่มี logic ซับซ้อนมาก ๆ
- **ข้อเสีย:** ต้องมีความรู้ภาษาโปรแกรมมากกว่า และ ecosystem/community เล็กกว่า Terraform

### AWS CloudFormation

**CloudFormation** เป็นเครื่องมือ IaC ของ AWS เอง ผูกติดกับ AWS เท่านั้น

- **ข้อดี:** integrate กับบริการ AWS ได้ลึกที่สุดเพราะเป็นเจ้าของ platform เอง ไม่มีค่าใช้จ่ายเพิ่มเติมในการใช้งาน (ฟรี ใช้แค่จ่ายค่า resource ที่สร้างจริง)
- **ข้อเสีย:** ใช้ได้แค่กับ AWS เท่านั้น ถ้าทีมต้องทำงานกับหลาย cloud หรือย้าย cloud ในอนาคต จะต้องเขียนใหม่ทั้งหมด

### Ansible

**Ansible** (โดย Red Hat) จริง ๆ แล้วออกแบบมาเพื่อ **Configuration Management** เป็นหลัก (ตั้งค่าเครื่องที่มีอยู่แล้ว เช่น ติดตั้ง package, แก้ config file, restart service) มากกว่าจะเน้นสร้าง infrastructure ใหม่ตั้งแต่ต้น แต่ก็สามารถใช้สร้าง cloud resource ได้เช่นกันผ่าน module

- **จุดเด่น:** Agentless (ใช้ SSH เชื่อมต่อ ไม่ต้องติดตั้งโปรแกรมเสริมบนเครื่องเป้าหมาย) เขียนง่ายด้วย YAML
- **เหมาะกับ:** งาน configuration management เช่น ตั้งค่าเซิร์ฟเวอร์ที่มีอยู่แล้วให้มี package/service ที่ต้องการ มักถูกใช้ **คู่กับ** Terraform ในหลายทีม (Terraform สร้างเครื่อง → Ansible ตั้งค่าเครื่องหลังสร้างเสร็จ)

### สรุป: ทำไมหลักสูตรนี้เลือกสอน Terraform

Terraform ถูกเลือกเพราะเป็นเครื่องมือ **มาตรฐานอุตสาหกรรม (industry standard)** ที่มีการใช้งานกว้างขวางที่สุด รองรับหลาย cloud provider ในภาษาเดียว และที่สำคัญที่สุดสำหรับหลักสูตรนี้คือ **การผสาน Terraform เข้ากับ Git workflow เป็นแนวทางที่เป็นมาตรฐานชัดเจนที่สุด** ตรงกับสิ่งที่เราต้องการเรียนรู้ต่อจาก GitOps ใน Part 76

---

## Step 763: Terraform เบื้องต้น — ไฟล์ `.tf`, `provider`, `resource` block

มาเริ่มเขียน Terraform จริง ๆ กัน Terraform ใช้ภาษาที่เรียกว่า **HCL (HashiCorp Configuration Language)** ซึ่งออกแบบมาให้อ่านง่ายและเขียนง่ายกว่า YAML/JSON ในหลายกรณี

### โครงสร้างไฟล์ทั่วไปของโปรเจกต์ Terraform

โปรเจกต์ Terraform ทั่วไปมักแบ่งไฟล์ตามหน้าที่ (แม้ Terraform จะอ่านไฟล์ `.tf` ทุกไฟล์ในโฟลเดอร์รวมกันเป็นชุดเดียวก็ตาม การแยกไฟล์เป็นแค่ธรรมเนียมเพื่อให้อ่านง่าย):

```
infra/
├── main.tf          # ประกาศ resource หลัก
├── variables.tf      # ประกาศตัวแปร (input variables)
├── outputs.tf         # ประกาศค่าที่ต้องการแสดงผลออกมาหลัง apply
├── providers.tf       # ประกาศ provider ที่ใช้
├── terraform.tfvars   # ค่าจริงของตัวแปร (ไม่ควร commit ถ้ามี secret)
└── versions.tf        # กำหนดเวอร์ชัน Terraform และ provider ที่ต้องใช้
```

### `terraform` block และ `required_providers`

ทุกโปรเจกต์ Terraform ควรเริ่มด้วยการระบุเวอร์ชันของ Terraform เองและ provider ที่จะใช้ เพื่อป้องกันปัญหาเวอร์ชันไม่ตรงกันระหว่างสมาชิกในทีม:

```hcl
# versions.tf
terraform {
  required_version = ">= 1.7.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### `provider` block

**Provider** คือ "ปลั๊กอิน" ที่ทำให้ Terraform รู้จักวิธีคุยกับบริการภายนอก เช่น AWS, Azure, GCP, GitHub, Cloudflare ฯลฯ แต่ละ provider จะมี resource type ของตัวเองให้เรียกใช้

```hcl
# providers.tf
provider "aws" {
  region = var.aws_region
}
```

ในตัวอย่างนี้เราประกาศ provider ชื่อ `aws` และกำหนดว่าให้ทำงานใน region ตามค่าตัวแปร `aws_region` (เราจะประกาศตัวแปรนี้ในไฟล์ `variables.tf` ต่อไป)

> **หมายเหตุด้านความปลอดภัย:** สังเกตว่าใน `provider` block เราไม่ได้ใส่ Access Key/Secret Key ไว้ตรง ๆ เลย เพราะ Terraform จะไปอ่าน credential จาก environment variable (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) หรือจากไฟล์ `~/.aws/credentials` ของเครื่องที่รันคำสั่งแทน — เราจะเจาะลึกเรื่องนี้อีกครั้งใน Step 769

### `variable` block (Input Variables)

ตัวแปรทำให้โค้ด Terraform ยืดหยุ่นและนำกลับมาใช้ซ้ำได้ ไม่ต้อง hardcode ค่าตายตัวไว้ในโค้ด:

```hcl
# variables.tf
variable "aws_region" {
  description = "AWS region ที่จะสร้าง resource ทั้งหมด"
  type        = string
  default     = "ap-southeast-1"
}

variable "environment" {
  description = "ชื่อ environment เช่น dev, staging, production"
  type        = string
}

variable "bucket_name" {
  description = "ชื่อ S3 bucket สำหรับเก็บไฟล์ static ของเว็บไซต์"
  type        = string
}
```

### `resource` block

**Resource** คือหัวใจของ Terraform — คือการประกาศว่า "ฉันต้องการให้มี resource แบบนี้อยู่จริง" รูปแบบทั่วไปคือ:

```hcl
resource "<PROVIDER>_<TYPE>" "<LOCAL_NAME>" {
  <ARGUMENT> = <VALUE>
  ...
}
```

ตัวอย่างการสร้าง S3 bucket สำหรับเก็บไฟล์ static:

```hcl
# main.tf
resource "aws_s3_bucket" "static_site" {
  bucket = var.bucket_name

  tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
    Project     = "portfolio-project"
  }
}

resource "aws_s3_bucket_website_configuration" "static_site" {
  bucket = aws_s3_bucket.static_site.id

  index_document {
    suffix = "index.html"
  }

  error_document {
    key = "error.html"
  }
}
```

สังเกตจุดสำคัญ:

- `"aws_s3_bucket"` คือ **type** ของ resource (มาจาก provider `aws`)
- `"static_site"` คือ **local name** ที่เราตั้งเอง ใช้อ้างอิงถึง resource นี้จากที่อื่นในโค้ด (เช่น `aws_s3_bucket.static_site.id` ในบรรทัดถัดมา ที่ใช้ดึงค่า `id` ของ bucket ที่เพิ่งประกาศไว้ด้านบน)
- การอ้างอิงข้าม resource แบบนี้เรียกว่า **implicit dependency** — Terraform จะรู้เองโดยอัตโนมัติว่าต้องสร้าง `aws_s3_bucket.static_site` ให้เสร็จก่อน จึงจะสร้าง `aws_s3_bucket_website_configuration.static_site` ได้ ไม่ต้องเขียนบอกลำดับเอง

### `output` block

**Output** ใช้แสดงค่าบางอย่างออกมาหลังจากรัน `terraform apply` เสร็จ เช่น URL ของเว็บไซต์ที่เพิ่งสร้าง หรือ IP address ของเซิร์ฟเวอร์:

```hcl
# outputs.tf
output "bucket_arn" {
  description = "ARN ของ S3 bucket ที่สร้าง"
  value       = aws_s3_bucket.static_site.arn
}

output "website_endpoint" {
  description = "URL สำหรับเข้าถึงเว็บไซต์ static"
  value       = aws_s3_bucket_website_configuration.static_site.website_endpoint
}
```

เมื่อรวมไฟล์ทั้งหมดเข้าด้วยกัน นี่คือภาพรวมของโปรเจกต์ Terraform ที่สมบูรณ์ในระดับพื้นฐาน — เราจะเอาโครงสร้างนี้ไปรันจริงใน Step ถัดไป

---

## Step 764: `terraform init` / `plan` / `apply` / `destroy` — Workflow เต็มรูปแบบ

Terraform มี workflow หลักที่ต้องทำความเข้าใจให้แม่นก่อนใช้งานจริง ประกอบด้วย 4 คำสั่งหลักที่ต้องรันตามลำดับ:

```
terraform init  →  terraform plan  →  terraform apply  →  (เมื่อไม่ใช้แล้ว) terraform destroy
```

### 1. `terraform init` — เตรียมโปรเจกต์

คำสั่งแรกที่ต้องรันเสมอเมื่อเริ่มโปรเจกต์ใหม่ หรือเมื่อเพิ่ม provider/module ใหม่:

```bash
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 5.0"...
- Installing hashicorp/aws v5.42.0...
- Installed hashicorp/aws v5.42.0 (signed by HashiCorp)

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure.
```

`terraform init` ทำหน้าที่:

1. **ดาวน์โหลด provider plugin** ที่ระบุไว้ใน `required_providers` มาเก็บไว้ในโฟลเดอร์ `.terraform/`
2. **ตั้งค่า backend** สำหรับเก็บ state file (ถ้ามีการกำหนด remote backend ไว้ — จะพูดถึงใน Step 766)
3. **สร้างไฟล์ `.terraform.lock.hcl`** เพื่อล็อกเวอร์ชันของ provider ที่ใช้จริง ให้ทุกคนในทีมใช้เวอร์ชันเดียวกันเป๊ะ ๆ (ไฟล์นี้**ควร** commit เข้า Git เพื่อความสม่ำเสมอของทีม)

> โฟลเดอร์ `.terraform/` ที่ถูกสร้างขึ้นมา **ไม่ควร** commit เข้า Git เพราะเป็นแค่ cache ของ plugin ที่ดาวน์โหลดมาใหม่ได้เสมอ — ควรใส่ไว้ใน `.gitignore`

### 2. `terraform plan` — ดูแผนการเปลี่ยนแปลงก่อนลงมือจริง

นี่คือคำสั่งที่สำคัญที่สุดของ Terraform ในแง่ความปลอดภัย เพราะมันจะ **บอกล่วงหน้า** ว่าถ้ารัน apply จริง จะเกิดอะไรขึ้นบ้าง โดย **ยังไม่แก้ไข infrastructure จริงแต่อย่างใด**

```bash
$ terraform plan

Terraform will perform the following actions:

  # aws_s3_bucket.static_site will be created
  + resource "aws_s3_bucket" "static_site" {
      + arn                         = (known after apply)
      + bucket                      = "portfolio-project-static-site-prod"
      + id                          = (known after apply)
      + region                      = "ap-southeast-1"
      + tags                        = {
          + "Environment" = "production"
          + "ManagedBy"   = "terraform"
          + "Project"     = "portfolio-project"
        }
    }

  # aws_s3_bucket_website_configuration.static_site will be created
  + resource "aws_s3_bucket_website_configuration" "static_site" {
      + bucket = (known after apply)
      + id     = (known after apply)

      + error_document {
          + key = "error.html"
        }

      + index_document {
          + suffix = "index.html"
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.
```

สัญลักษณ์ที่ต้องรู้จักในผลลัพธ์ของ `plan`:

| สัญลักษณ์ | ความหมาย |
|---|---|
| `+` | resource ใหม่ที่จะถูก**สร้าง** |
| `-` | resource ที่จะถูก**ลบ** |
| `~` | resource ที่มีอยู่แล้วจะถูก**แก้ไข** (in-place update) |
| `-/+` | resource จะถูก**ลบแล้วสร้างใหม่** (destroy and re-create — มักเกิดกับ argument บางตัวที่แก้ไขแบบ in-place ไม่ได้) |

การอ่านผลลัพธ์ `plan` อย่างละเอียด **ทุกครั้งก่อน apply** เป็นวินัยที่สำคัญที่สุดของคนทำงานกับ Terraform โดยเฉพาะการระวังบรรทัด `-` และ `-/+` ที่หมายถึงการลบ resource ซึ่งอาจเป็นข้อมูลสำคัญที่กู้คืนไม่ได้

### 3. `terraform apply` — ลงมือสร้าง/แก้ไขจริง

```bash
$ terraform apply

Terraform will perform the following actions:
  (... เหมือนผลลัพธ์จาก plan ...)

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_s3_bucket.static_site: Creating...
aws_s3_bucket.static_site: Creation complete after 2s [id=portfolio-project-static-site-prod]
aws_s3_bucket_website_configuration.static_site: Creating...
aws_s3_bucket_website_configuration.static_site: Creation complete after 1s

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

bucket_arn = "arn:aws:s3:::portfolio-project-static-site-prod"
website_endpoint = "portfolio-project-static-site-prod.s3-website-ap-southeast-1.amazonaws.com"
```

`terraform apply` จะรัน plan ให้อีกครั้งภายในตัว แล้วถามยืนยันก่อนลงมือจริงเสมอ (เว้นแต่จะใส่ flag `-auto-approve` ซึ่งมักใช้ใน CI/CD pipeline เท่านั้น ไม่ควรใช้ตอนรันด้วยมือ)

**เทคนิคที่แนะนำ:** บันทึกผลลัพธ์จาก `plan` ไว้เป็นไฟล์ก่อน แล้วนำไฟล์นั้นไป apply ต่อ เพื่อรับประกันว่าสิ่งที่ apply ตรงกับสิ่งที่ตรวจสอบไว้เป๊ะ ๆ ไม่มีอะไรเปลี่ยนแปลงระหว่างสองขั้นตอน (สำคัญมากใน CI/CD):

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

### 4. `terraform destroy` — ลบ infrastructure ทั้งหมด

```bash
$ terraform destroy

Terraform will perform the following actions:

  # aws_s3_bucket.static_site will be destroyed
  - resource "aws_s3_bucket" "static_site" {
      - bucket = "portfolio-project-static-site-prod" -> null
      ...
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

Destroy complete! Resources: 2 destroyed.
```

`terraform destroy` ลบ resource **ทั้งหมด**ที่ Terraform จัดการอยู่ มักใช้กับ environment ทดสอบชั่วคราว (เช่น environment สำหรับ demo หรือ pull request preview) ที่ไม่ต้องการให้ค้างอยู่ตลอดไปเพื่อประหยัดค่าใช้จ่าย **ต้องระวังอย่างยิ่งที่จะไม่รันคำสั่งนี้ผิด environment** โดยเฉพาะ production

### สรุป Workflow เป็นภาพเดียว

```
เขียน/แก้ไขไฟล์ .tf
        │
        ▼
  terraform init        ← ทำครั้งแรก หรือเมื่อเพิ่ม provider/module ใหม่
        │
        ▼
  terraform plan         ← ตรวจสอบว่าจะเปลี่ยนแปลงอะไรบ้าง (ไม่แก้จริง)
        │
        ▼
  ทบทวนผลลัพธ์ plan ให้ละเอียด
        │
        ▼
  terraform apply        ← ยืนยันแล้วลงมือสร้าง/แก้ไขจริง
        │
        ▼
  infrastructure พร้อมใช้งาน
        │
        ▼
  (เมื่อไม่ใช้แล้ว) terraform destroy   ← ลบทิ้งทั้งหมด
```

---

## Step 765: State File คืออะไร ทำไมถึงสำคัญมาก และห้าม commit เข้า Git

### State File คืออะไร

หลังรัน `terraform apply` ครั้งแรก Terraform จะสร้างไฟล์ชื่อ **`terraform.tfstate`** ขึ้นมาในโฟลเดอร์โปรเจกต์ ไฟล์นี้คือ **"ความทรงจำ"** ของ Terraform ที่บันทึกว่า **infrastructure จริงตอนนี้หน้าตาเป็นอย่างไรบ้าง** โดยเก็บในรูปแบบ JSON

```json
{
  "version": 4,
  "terraform_version": "1.7.4",
  "resources": [
    {
      "mode": "managed",
      "type": "aws_s3_bucket",
      "name": "static_site",
      "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
      "instances": [
        {
          "attributes": {
            "id": "portfolio-project-static-site-prod",
            "arn": "arn:aws:s3:::portfolio-project-static-site-prod",
            "bucket": "portfolio-project-static-site-prod",
            "region": "ap-southeast-1",
            "tags": {
              "Environment": "production",
              "ManagedBy": "terraform"
            }
          }
        }
      ]
    }
  ]
}
```

### ทำไม State File ถึงสำคัญขนาดนี้

Terraform ใช้ state file ในการตอบคำถามสำคัญ 3 ข้อทุกครั้งที่รัน `plan` หรือ `apply`:

1. **โค้ด (desired state)** บอกว่าอยากได้อะไร
2. **State file** บอกว่า Terraform เคยสร้างอะไรไว้แล้วบ้าง และหน้าตาตอนที่สร้างเป็นอย่างไร
3. Terraform เทียบสอง**อย่างนี้เข้าด้วยกัน** (บางครั้งอาจ query ของจริงจาก cloud provider เพิ่มด้วย เรียกว่า refresh) แล้วคำนวณว่า **"ต้องเปลี่ยนแปลงอะไรบ้างเพื่อให้ของจริงตรงกับโค้ด"**

ถ้า **ไม่มี state file** Terraform จะไม่รู้เลยว่า resource ที่มันเคยสร้างไว้คืออันไหนในโลกจริง — มันจะพยายามสร้าง resource ใหม่ซ้ำซ้อนกับของเดิม หรือไม่รู้ว่าต้องลบตัวไหนเมื่อเราลบโค้ดออก

> **เปรียบเทียบง่าย ๆ:** ถ้าโค้ด `.tf` คือ "พิมพ์เขียว" ของบ้านที่อยากได้ state file ก็คือ "ทะเบียนบ้าน" ที่บันทึกว่าบ้านหลังไหนสร้างไปแล้วจริง ๆ เลขที่เท่าไหร่ ไม่มีทะเบียนบ้าน ต่อให้มีพิมพ์เขียวก็ไม่รู้ว่าบ้านที่สร้างไปแล้วคือหลังไหน

### ทำไมห้าม commit `terraform.tfstate` เข้า Git โดยตรง

แม้ state file จะเป็นไฟล์ข้อความที่ดู commit เข้า Git ได้ตามปกติ แต่ **ห้ามทำเด็ดขาด** ด้วยเหตุผลสำคัญหลายข้อ:

1. **มี Sensitive Data อยู่ในนั้นแบบข้อความล้วน (plaintext)** — attribute หลายตัวของ resource เช่น database password, private key, connection string ที่ provider ส่งกลับมา จะถูกบันทึกไว้ใน state file **โดยไม่เข้ารหัส** แม้ในโค้ด `.tf` เราจะกำหนด `sensitive = true` เพื่อไม่ให้ค่านั้นแสดงในผลลัพธ์บนหน้าจอ Terminal ก็ตาม แต่ **state file ยังคงบันทึกค่าจริงไว้เต็ม ๆ เสมอ** — ถ้า commit เข้า Git ที่เป็น public repo หรือแม้แต่ private repo ที่มีคนเข้าถึงจำนวนมาก secret เหล่านี้จะรั่วไหลทันที
2. **เกิด Conflict ร้ายแรงได้ง่ายมากเมื่อทำงานเป็นทีม** — ถ้าคนสองคนรัน `apply` พร้อมกันโดยต่างคนต่าง commit state file ของตัวเอง แล้วเกิด merge conflict บนไฟล์ JSON ที่ซับซ้อน การแก้ conflict ด้วยมือบน state file มีความเสี่ยงสูงมากที่จะทำให้ Terraform "หลงทาง" จำ infrastructure ผิดไปจากความจริง จนอาจนำไปสู่การลบหรือสร้าง resource ซ้ำซ้อนโดยไม่ได้ตั้งใจ
3. **State file เปลี่ยนแปลงบ่อยมาก** — ทุกครั้งที่มีคน apply แม้แต่การเปลี่ยนแปลงเล็กน้อย state file ทั้งไฟล์จะถูกเขียนทับใหม่ ทำให้ประวัติ Git เต็มไปด้วย commit ที่ diff ยากอ่านและไม่มีความหมายในเชิง code review

### แล้วต้องทำอย่างไร

คำตอบคือ:

1. ใส่ `terraform.tfstate` และ `terraform.tfstate.backup` ไว้ใน `.gitignore` เสมอ
2. ใช้ **Remote State** เก็บ state file ไว้ที่ backend กลางที่ปลอดภัยและรองรับการทำงานเป็นทีม แทนที่จะเก็บไว้ในเครื่อง local หรือใน Git — นี่คือหัวข้อของ Step ถัดไป

```gitignore
# .gitignore สำหรับโปรเจกต์ Terraform
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
!example.tfvars
crash.log
override.tf
override.tf.json
.terraform.lock.hcl
```

> **หมายเหตุ:** บางทีมเลือกที่จะ commit `.terraform.lock.hcl` เข้า Git เพื่อล็อกเวอร์ชัน provider ให้ตรงกันทั้งทีม (แนะนำให้ commit ไฟล์นี้) แต่ **ไม่ควร** commit ไฟล์ `.tfvars` ที่มีค่าจริง (เช่น secret, ชื่อ resource เฉพาะของแต่ละ environment) เข้า Git เช่นกัน

### คำสั่งสำหรับตรวจสอบ State โดยไม่ต้องเปิดไฟล์ JSON ตรง ๆ

Terraform มีคำสั่งชุด `terraform state` ให้ใช้ตรวจสอบและจัดการ state อย่างปลอดภัย โดยไม่ต้องแก้ไฟล์ JSON ด้วยมือ:

```bash
# แสดงรายการ resource ทั้งหมดที่อยู่ใน state
terraform state list

# แสดงรายละเอียดของ resource ตัวใดตัวหนึ่ง
terraform state show aws_s3_bucket.static_site

# ย้าย resource ไปยังชื่อใหม่ใน state (เช่น ตอน refactor โค้ด)
terraform state mv aws_s3_bucket.static_site aws_s3_bucket.website

# ลบ resource ออกจาก state โดยไม่ลบของจริง (ใช้เมื่อต้องการเลิกให้ Terraform จัดการ resource นี้)
terraform state rm aws_s3_bucket.static_site
```

**หลักการทอง: ห้ามแก้ไฟล์ `terraform.tfstate` ด้วยมือเด็ดขาด** ให้ใช้คำสั่งชุด `terraform state` เสมอ เพราะการแก้ไฟล์ JSON ตรง ๆ มีโอกาสสูงมากที่จะทำให้ format ผิดพลาดจน Terraform อ่านไม่ออก หรือทำให้ state ไม่ตรงกับความจริงอีกต่อไป

---

## Step 766: Remote State และ State Locking — ทำงานเป็นทีมอย่างปลอดภัย

### ปัญหาของ Local State เมื่อทำงานเป็นทีม

ถ้า state file เก็บไว้ในเครื่อง local ของแต่ละคน จะเกิดปัญหาทันทีที่มีคนมากกว่า 1 คนทำงานร่วมกัน:

- คนละคนมี state file คนละไฟล์ ไม่ตรงกัน — คนหนึ่ง apply ไปแล้ว แต่อีกคนไม่รู้ ทำให้ state ของแต่ละคนไม่ตรงกับความจริง
- ไม่มีใครแน่ใจว่า "state ตัวจริง" อยู่ที่เครื่องใคร
- ถ้าสองคนรัน `apply` **พร้อมกัน** โดยที่ไม่มีกลไกป้องกัน อาจเกิดการเขียนทับ state กันเอง หรือแก้ไข infrastructure เดียวกันพร้อมกันจนเกิดผลลัพธ์ที่ไม่คาดคิด (race condition)

### Remote State: เก็บ State ไว้ที่ Backend กลาง

**Remote State** คือการกำหนดให้ Terraform เก็บ state file ไว้ที่ **backend กลาง** ที่ทุกคนในทีมและทุก pipeline ใน CI/CD เข้าถึงได้ร่วมกัน แทนที่จะเก็บไว้ในเครื่อง local ของใครคนใดคนหนึ่ง

Backend ยอดนิยมที่ใช้เก็บ Remote State ได้แก่:

| Backend | รายละเอียด |
|---|---|
| **AWS S3** (+ DynamoDB สำหรับ lock) | นิยมที่สุดสำหรับทีมที่ใช้ AWS |
| **Terraform Cloud / HCP Terraform** | บริการของ HashiCorp เอง มี state locking, ประวัติ, และ UI ให้ในตัว |
| **Azure Storage Account** | สำหรับทีมที่ใช้ Azure |
| **Google Cloud Storage (GCS)** | สำหรับทีมที่ใช้ GCP |
| **GitLab-managed Terraform State** | ใช้ GitLab เองเป็น backend เก็บ state ได้เลย ไม่ต้องพึ่ง cloud storage แยก |

### ตั้งค่า Backend แบบ S3 + DynamoDB (ตัวอย่างที่พบบ่อยที่สุด)

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "portfolio-project-terraform-state"
    key            = "production/terraform.tfstate"
    region         = "ap-southeast-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

อธิบายแต่ละ argument:

- **`bucket`** — ชื่อ S3 bucket ที่จะเก็บไฟล์ state (ต้องสร้าง bucket นี้ไว้ล่วงหน้าก่อน มักสร้างด้วย Terraform เองในโปรเจกต์แยกต่างหากที่เรียกว่า "bootstrap")
- **`key`** — path ของไฟล์ state ภายใน bucket นั้น — สังเกตว่าเราตั้งชื่อ path แยกตาม environment (`production/terraform.tfstate`) เพื่อไม่ให้ state ของแต่ละ environment ปนกัน
- **`region`** — region ของ S3 bucket
- **`dynamodb_table`** — ชื่อ DynamoDB table ที่ใช้ทำ **State Locking** (อธิบายต่อด้านล่าง)
- **`encrypt`** — เข้ารหัสไฟล์ state ที่เก็บใน S3 ด้วย (server-side encryption) เพื่อความปลอดภัยเพิ่มเติม

หลังตั้งค่า backend แล้ว ต้องรัน `terraform init` ใหม่อีกครั้งเพื่อให้ Terraform ย้าย/เชื่อมต่อไปยัง backend ใหม่:

```bash
$ terraform init

Initializing the backend...

Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Reusing previous version of hashicorp/aws from the dependency lock file

Terraform has been successfully initialized!
```

### State Locking คืออะไร และทำไมถึงจำเป็น

**State Locking** คือกลไกที่ป้องกันไม่ให้มีคนสองคน (หรือ pipeline สองอันใน CI/CD) รัน `apply` **พร้อมกัน** บน state เดียวกันในเวลาเดียวกัน

หลักการทำงานง่าย ๆ คือ: ก่อน Terraform จะแก้ไข state file มันจะพยายาม **"ล็อก"** state นั้นไว้ก่อน (คล้ายกับการล็อกไฟล์เวลาแก้ไขเอกสารร่วมกัน) ถ้ามีคนอื่นกำลังถือ lock อยู่ Terraform จะรอ หรือแจ้ง error ทันที แทนที่จะปล่อยให้สองคนเขียนทับ state พร้อมกัน:

```bash
$ terraform apply

Error: Error acquiring the state lock

Error message: ConditionalCheckFailedException: The conditional request failed
Lock Info:
  ID:        7a4f3c21-9e8b-4d6a-b1c5-3f8e2a1d9c4b
  Path:      portfolio-project-terraform-state/production/terraform.tfstate
  Operation: OperationTypeApply
  Who:       somchai@ci-runner-42
  Version:   1.7.4
  Created:   2026-09-20 08:15:22 UTC

Terraform acquires a state lock to protect the state from being written
by multiple users at the same time. Please resolve the issue above and try
again.
```

DynamoDB table ที่ใช้ทำ lock ต้องมี primary key ชื่อ `LockID` (เป็น string) — โดยทั่วไปทีมจะสร้าง table นี้ไว้ล่วงหน้าครั้งเดียวตอนเริ่มโปรเจกต์:

```hcl
# ไฟล์แยกต่างหากสำหรับ "bootstrap" backend (รันแค่ครั้งเดียวตอนตั้งโปรเจกต์)
resource "aws_dynamodb_table" "terraform_lock" {
  name         = "terraform-state-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Purpose = "terraform-state-locking"
  }
}
```

> **หมายเหตุสำหรับ Terraform เวอร์ชันใหม่:** ตั้งแต่ Terraform 1.10 เป็นต้นมา S3 backend รองรับ **native state locking** ผ่านฟีเจอร์ `use_lockfile = true` ในตัวเองได้โดยไม่ต้องพึ่ง DynamoDB แยกต่างหากอีกต่อไป แต่แนวทาง S3 + DynamoDB ยังคงเป็นรูปแบบมาตรฐานที่พบเห็นได้ทั่วไปในระบบที่มีอยู่แล้วจำนวนมาก และยังคงใช้งานได้ดีเหมือนเดิม

### สรุปประโยชน์ของ Remote State + Locking

1. **ทีมทำงานร่วมกันได้อย่างปลอดภัย** — ทุกคนดึง state เดียวกันมาใช้เสมอ ไม่มีใครถือ state ฉบับที่ล้าสมัย
2. **ป้องกัน Race Condition** — ไม่มีสองคนแก้ไข infrastructure เดียวกันพร้อมกันจนข้อมูลเสียหาย
3. **ปลอดภัยกว่าเก็บใน Git** — ไม่มี sensitive data หลุดเข้า Git history ถ้าตั้งค่า encryption และ access control ของ backend อย่างเหมาะสม
4. **CI/CD Pipeline เข้าถึง state เดียวกับที่คนในทีมใช้ได้** — ทำให้ automated pipeline และการรันด้วยมือใช้ข้อมูลชุดเดียวกันเสมอ

---

## Step 767: การจัดการ Terraform Code ด้วย Git (Branch, PR Review ก่อน Apply จริง)

แม้ state file จะไม่ถูกเก็บใน Git แต่ **โค้ด `.tf` ทั้งหมดต้องถูกจัดการผ่าน Git เหมือนโค้ดแอปพลิเคชันทุกประการ** — นี่คือหัวใจของแนวคิด "Infrastructure as Code" ที่แท้จริง เพราะถ้าไม่มี Git คอยควบคุม เราก็แค่ย้ายปัญหา ClickOps จากหน้าเว็บ console มาเป็นการรันคำสั่งมั่ว ๆ จากเครื่อง local แทนเท่านั้นเอง

### โครงสร้าง Repository สำหรับ Terraform

ทีมส่วนใหญ่นิยมเก็บโค้ด Terraform ไว้ใน repository แยกต่างหากจากโค้ดแอปพลิเคชัน เรียกว่า **infrastructure repository**:

```
infra-repo/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── backend.tf      # key = "dev/terraform.tfstate"
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   ├── backend.tf      # key = "staging/terraform.tfstate"
│   │   └── terraform.tfvars
│   └── production/
│       ├── main.tf
│       ├── backend.tf      # key = "production/terraform.tfstate"
│       └── terraform.tfvars
├── modules/
│   ├── network/
│   ├── compute/
│   └── static-site/
├── .gitignore
└── README.md
```

### Branching Strategy สำหรับ Terraform

หลักการเดียวกับที่เราเรียนใน Git Flow / GitHub Flow (Part 27-29) นำมาใช้กับ Terraform ได้ตรง ๆ:

1. **`main` branch คือความจริงของ production เสมอ** — โค้ดใน `main` ต้องตรงกับ infrastructure ที่รันอยู่จริงบน production ทุกครั้ง (นี่คือหลักการ GitOps ที่เรียนใน Part 76)
2. **ห้าม push ตรงเข้า `main` เด็ดขาด** — ตั้งค่า branch protection rule ให้ `main` ต้องผ่าน Pull Request/Merge Request เท่านั้น
3. **สร้าง feature branch สำหรับทุกการเปลี่ยนแปลง infrastructure** เช่น `infra/add-redis-cache`, `infra/increase-instance-size`, `infra/fix-security-group`

```bash
git checkout -b infra/add-cloudfront-cdn
# แก้ไขไฟล์ .tf เพื่อเพิ่ม CloudFront distribution
git add environments/production/main.tf
git commit -m "เพิ่ม CloudFront CDN หน้า static site เพื่อลด latency"
git push -u origin infra/add-cloudfront-cdn
```

### Pull Request Review ก่อน Apply จริงเสมอ

หัวใจสำคัญที่สุดของการจัดการ Terraform ด้วย Git คือ **ไม่มีใครควรรัน `terraform apply` เข้า production ได้โดยไม่ผ่านการรีวิวจากคนอื่นก่อน** เหมือนกับที่โค้ดแอปพลิเคชันต้องผ่าน code review ก่อน merge

Checklist สิ่งที่ผู้รีวิว (reviewer) ควรตรวจสอบใน Pull Request ของ Terraform โดยเฉพาะ:

- [ ] อ่านผลลัพธ์ `terraform plan` ที่แนบมาใน PR อย่างละเอียด (ดู Step 768 ว่าทำอัตโนมัติได้อย่างไร)
- [ ] มี resource ไหนที่จะถูก **ลบ (`-`)** หรือ **สร้างใหม่ทดแทน (`-/+`)** หรือไม่ — ถ้ามี ต้องแน่ใจว่าไม่ใช่ resource ที่มีข้อมูลสำคัญ (เช่น database ที่มี production data)
- [ ] ชื่อ resource, tag, naming convention ตรงตามมาตรฐานของทีมหรือไม่
- [ ] มีการ hardcode ค่าที่ควรเป็นตัวแปรหรือไม่ (เช่น IP address, ชื่อ environment)
- [ ] มี secret หลุดอยู่ในโค้ดหรือไม่ (ดูรายละเอียดเพิ่มเติมใน Step 769 และ Part 78 ซึ่งเจาะลึกเรื่อง Secret Management โดยเฉพาะ)
- [ ] คาดการณ์ผลกระทบต่อค่าใช้จ่าย (cost) หรือไม่ — resource ใหม่จะทำให้ค่าใช้จ่ายเพิ่มขึ้นเท่าไหร่

### ใช้ CODEOWNERS กับโฟลเดอร์ Terraform

เหมือนที่เรียนใน Part 36 เรื่อง `CODEOWNERS` เราสามารถบังคับให้การเปลี่ยนแปลงโค้ด Terraform โดยเฉพาะ **ต้องผ่านการรีวิวจากทีม Infrastructure/DevOps เท่านั้น**:

```
# .github/CODEOWNERS
/environments/production/  @portfolio-project/infra-team
/modules/                  @portfolio-project/infra-team
```

### ตั้งค่า Branch Protection สำหรับ `main`

สอดคล้องกับที่เรียนใน Part 30 (Branch Protection Rules) ทีมควรตั้งค่าให้ `main` branch ของ infrastructure repository:

- ต้องผ่าน Pull Request อย่างน้อย 1-2 approval ก่อน merge ได้
- ต้องผ่าน CI check (เช่น `terraform validate`, `terraform plan` สำเร็จ) ก่อน merge ได้
- ห้าม force push เข้า `main` เด็ดขาด
- (แนะนำ) บังคับให้ branch ต้อง up-to-date กับ `main` ก่อน merge เพื่อป้องกัน plan ที่คำนวณจากโค้ดเก่า

---

## Step 768: Terraform ใน CI/CD Pipeline (Plan ใน PR, Apply ตอน Merge)

เมื่อนำ Terraform เข้าไปอยู่ใน CI/CD pipeline อย่างเต็มรูปแบบ เราจะได้ workflow ที่ทั้งปลอดภัยและอัตโนมัติ ตรงกับหลักการ GitOps ที่เรียนใน Part 76: **การเปลี่ยนแปลง infrastructure ทุกครั้งเริ่มต้นจาก Pull Request เสมอ ไม่มีใครรันคำสั่งจากเครื่อง local เข้า production โดยตรงอีกต่อไป**

### แนวคิดหลักของ Pipeline

```
เปิด Pull Request  →  Pipeline รัน "terraform plan" อัตโนมัติ  →  แสดงผล plan เป็น comment ใน PR
        │
        ▼
ทีมรีวิวโค้ดและผลลัพธ์ plan ร่วมกัน
        │
        ▼
Approve และ Merge เข้า main
        │
        ▼
Pipeline รัน "terraform apply" อัตโนมัติ (มักมี manual approval step ก่อน apply เข้า production)
```

### ตัวอย่าง GitLab CI/CD Pipeline สำหรับ Terraform

```yaml
# .gitlab-ci.yml
stages:
  - validate
  - plan
  - apply

variables:
  TF_ROOT: ${CI_PROJECT_DIR}/environments/production
  TF_STATE_NAME: production

default:
  image: hashicorp/terraform:1.7
  before_script:
    - cd "${TF_ROOT}"
    - terraform init

validate:
  stage: validate
  script:
    - terraform fmt -check -recursive
    - terraform validate
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

plan:
  stage: plan
  script:
    - terraform plan -out=tfplan
    - terraform show -no-color tfplan > plan_output.txt
  artifacts:
    paths:
      - ${TF_ROOT}/tfplan
      - ${TF_ROOT}/plan_output.txt
    expire_in: 1 week
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

apply:
  stage: apply
  script:
    - terraform apply -auto-approve tfplan
  dependencies:
    - plan
  environment:
    name: production
  when: manual
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

จุดสำคัญของ pipeline นี้:

- **`validate` stage** รันทุกครั้งที่มีการเปิด Merge Request เพื่อเช็ค syntax และ format ของโค้ด (`terraform fmt -check`, `terraform validate`) — เป็นเหมือน linter ของโลก Terraform
- **`plan` stage** รันเฉพาะตอนเป็น Merge Request เท่านั้น และเก็บผลลัพธ์เป็น `artifact` ที่ทั้งไฟล์ `tfplan` (ใช้ apply ต่อได้เป๊ะ ๆ) และไฟล์ text ที่อ่านง่ายสำหรับให้คนรีวิว
- **`apply` stage** รันเฉพาะบน `main` branch เท่านั้น (คือหลัง merge แล้ว) และตั้งเป็น `when: manual` เพื่อบังคับให้ต้องมีคนกดยืนยันก่อน apply เข้า production จริง (สอดคล้องกับหลักการ manual approval ที่เรียนใน Part 74)
- ใช้ `dependencies: [plan]` เพื่อดึงไฟล์ `tfplan` ที่ผ่านการรีวิวมาแล้วมา apply ต่อ **แทนที่จะรัน plan ใหม่ตอน apply** — รับประกันว่าสิ่งที่ apply คือสิ่งเดียวกับที่ทีมรีวิวผ่านไปแล้วเป๊ะ ๆ

### แสดงผลลัพธ์ `plan` เป็น Comment ใน Pull Request

ในทีมมืออาชีพ มักใช้เครื่องมือเสริม เช่น **Atlantis** หรือสคริปต์ง่าย ๆ เพื่อโพสต์ผลลัพธ์ `terraform plan` กลับเข้าไปเป็น comment ใน Merge Request โดยอัตโนมัติ ทำให้ผู้รีวิวไม่ต้องเปิด pipeline log เอง:

```yaml
comment_plan_on_mr:
  stage: plan
  script:
    - |
      PLAN_TEXT=$(cat plan_output.txt)
      curl --request POST \
        --header "PRIVATE-TOKEN: ${GITLAB_API_TOKEN}" \
        --header "Content-Type: application/json" \
        --data "{\"body\": \"### Terraform Plan Result\n\`\`\`\n${PLAN_TEXT}\n\`\`\`\"}" \
        "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/merge_requests/${CI_MERGE_REQUEST_IID}/notes"
  needs: ["plan"]
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
```

> **หมายเหตุ:** `GITLAB_API_TOKEN` ในตัวอย่างข้างต้นต้องถูกเก็บเป็น **CI/CD Variable แบบ Masked/Protected** เท่านั้น (ตามที่เรียนใน Part 74) ห้าม hardcode token ไว้ในไฟล์ pipeline เด็ดขาด

### ตัวอย่าง GitHub Actions สำหรับ Terraform (โดยย่อ)

สำหรับทีมที่ใช้ GitHub แนวคิดเดียวกันนี้ทำได้ด้วย GitHub Actions:

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  pull_request:
    branches: [main]
    paths: ["environments/production/**"]
  push:
    branches: [main]
    paths: ["environments/production/**"]

jobs:
  plan:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: environments/production
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
      - run: terraform init
      - run: terraform plan -no-color
        id: plan
      - uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: "### Terraform Plan\n```\n" + `${{ steps.plan.outputs.stdout }}` + "\n```"
            })

  apply:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    environment: production
    defaults:
      run:
        working-directory: environments/production
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
      - run: terraform init
      - run: terraform apply -auto-approve
```

สังเกตว่า job `apply` ใช้ `environment: production` ซึ่งเป็นฟีเจอร์ของ GitHub ที่ทำให้ตั้ง **required reviewers** ได้ก่อนที่ job นี้จะรันจริง เหมือนกับ manual approval gate ที่เราตั้งค่าไว้ฝั่ง GitLab ด้วยเช่นกัน

---

## Step 769: Best Practices — Module, Workspace, ห้าม Hardcode Secret

เมื่อโปรเจกต์ Terraform โตขึ้น มีทั้งหลาย environment หลาย team เข้ามาเกี่ยวข้อง จำเป็นต้องมีแนวปฏิบัติที่ดีเพื่อให้โค้ดยังคงดูแลรักษาได้ง่ายและปลอดภัย

### 1. แบ่งโค้ดเป็น Module เพื่อนำกลับมาใช้ซ้ำ

**Module** คือชุดของไฟล์ `.tf` ที่ถูกจัดกลุ่มไว้เพื่อนำไปเรียกใช้ซ้ำได้หลายที่ เช่น module สำหรับสร้าง "static website hosting" ที่ใช้ได้ทั้ง dev, staging, production โดยส่งค่าตัวแปรต่างกันแค่นั้น

```
modules/
└── static-site/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

```hcl
# modules/static-site/variables.tf
variable "bucket_name" {
  description = "ชื่อ bucket สำหรับ static site"
  type        = string
}

variable "environment" {
  description = "ชื่อ environment"
  type        = string
}
```

```hcl
# modules/static-site/main.tf
resource "aws_s3_bucket" "this" {
  bucket = var.bucket_name

  tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}

resource "aws_s3_bucket_website_configuration" "this" {
  bucket = aws_s3_bucket.this.id

  index_document {
    suffix = "index.html"
  }
}
```

```hcl
# modules/static-site/outputs.tf
output "website_endpoint" {
  value = aws_s3_bucket_website_configuration.this.website_endpoint
}
```

จากนั้นเรียกใช้ module นี้จากแต่ละ environment ได้ง่าย ๆ:

```hcl
# environments/production/main.tf
module "static_site" {
  source      = "../../modules/static-site"
  bucket_name = "portfolio-project-static-site-prod"
  environment = "production"
}

output "prod_website_url" {
  value = module.static_site.website_endpoint
}
```

```hcl
# environments/staging/main.tf
module "static_site" {
  source      = "../../modules/static-site"
  bucket_name = "portfolio-project-static-site-staging"
  environment = "staging"
}
```

**ประโยชน์ของ Module:** เขียนโค้ดครั้งเดียว ใช้ซ้ำได้หลาย environment ลดโอกาสเขียนผิดเพราะ copy-paste และแก้ไข logic ที่จุดเดียวก็มีผลกับทุก environment ที่เรียกใช้ module นั้น

### 2. แยก Environment ด้วย Workspace หรือแยกโฟลเดอร์

Terraform มีฟีเจอร์ **Workspace** ที่ทำให้ state file เดียวกันสามารถแยกเป็นหลายชุดได้ภายใต้ backend เดียวกัน:

```bash
terraform workspace new staging
terraform workspace new production
terraform workspace select production
terraform workspace list
```

```
$ terraform workspace list
  default
  staging
* production
```

อย่างไรก็ตาม **หลักสูตรนี้แนะนำแนวทาง "แยกโฟลเดอร์ตาม environment"** (ตามตัวอย่างโครงสร้าง `environments/dev`, `environments/staging`, `environments/production` ที่ใช้มาตลอด Part นี้) มากกว่าการใช้ Workspace ล้วน ๆ เพราะ:

- แยกโฟลเดอร์ทำให้เห็น config ของแต่ละ environment ชัดเจนแยกจากกันจริง ไม่ต้องกังวลว่าลืมสลับ workspace ผิดแล้ว apply เข้าผิด environment
- Backend/state key ของแต่ละ environment แยกกันชัดเจนตั้งแต่ระดับไฟล์ ลดความเสี่ยงจาก human error
- Workspace ยังคงมีประโยชน์มากในกรณีที่ต้องการสร้าง environment ชั่วคราวจำนวนมาก เช่น environment สำหรับทดสอบแต่ละ Pull Request (ephemeral preview environment)

### 3. ห้าม Hardcode Secret ใน `.tf` ไฟล์เด็ดขาด

นี่คือกฎเหล็กที่สำคัญที่สุดข้อหนึ่ง — **ห้ามเขียน password, API key, token ใด ๆ ลงในไฟล์ `.tf` โดยตรงเด็ดขาด** เพราะไฟล์เหล่านี้จะถูก commit เข้า Git และอยู่ใน history ตลอดไป (ต่อให้ลบออกทีหลังก็ยังค้นเจอใน commit history เก่าได้อยู่ดี ตามที่เรียนใน Part 78 เรื่อง Secret Management)

**ตัวอย่างที่ผิดอย่างร้ายแรง (ห้ามทำ):**

```hcl
# ❌ ห้ามทำแบบนี้เด็ดขาด
resource "aws_db_instance" "main" {
  engine   = "postgres"
  username = "admin"
  password = "P@ssw0rd12345"   # secret hardcode ตรงนี้ห้ามทำเด็ดขาด
}
```

**แนวทางที่ถูกต้อง:** ใช้ตัวแปรที่ทำเครื่องหมาย `sensitive = true` แล้วส่งค่าจริงเข้ามาผ่านช่องทางที่ปลอดภัย เช่น environment variable หรือ secret manager แทนการเขียนไว้ในไฟล์:

```hcl
# variables.tf
variable "db_password" {
  description = "รหัสผ่านฐานข้อมูล (ส่งเข้ามาผ่าน TF_VAR_db_password เท่านั้น ห้ามใส่ default)"
  type        = string
  sensitive   = true
}
```

```hcl
# main.tf
resource "aws_db_instance" "main" {
  engine   = "postgres"
  username = "admin"
  password = var.db_password
}
```

ค่า secret จริงถูกส่งเข้ามาตอนรันคำสั่งผ่าน environment variable (Terraform จะอ่านตัวแปรที่ขึ้นต้นด้วย `TF_VAR_` โดยอัตโนมัติ) ซึ่งในระบบ CI/CD จะดึงค่านี้มาจาก **Secret Manager** ของแพลตฟอร์ม (เช่น GitLab CI/CD Variables แบบ Masked, GitHub Actions Secrets, HashiCorp Vault, AWS Secrets Manager) ไม่ใช่จากไฟล์ในโค้ด:

```bash
export TF_VAR_db_password="ค่าจริงที่ดึงมาจาก Secret Manager เท่านั้น"
terraform apply
```

การกำหนด `sensitive = true` ยังช่วยให้ Terraform **ซ่อนค่านี้จากผลลัพธ์ที่แสดงบนหน้าจอ** ตอนรัน `plan`/`apply` (แม้ค่าจริงจะยังถูกบันทึกไว้ใน state file ตามที่อธิบายใน Step 765 ก็ตาม — ดังนั้นการป้องกัน remote state backend ให้ปลอดภัยจึงสำคัญไม่แพ้กัน)

```
  # aws_db_instance.main will be created
  + resource "aws_db_instance" "main" {
      + password = (sensitive value)
      ...
    }
```

### 4. สแกนความปลอดภัยของโค้ด Terraform อัตโนมัติ

เพิ่มเครื่องมือสแกนความปลอดภัย เช่น **tfsec** หรือ **Checkov** เข้าไปใน pipeline เพื่อตรวจจับ misconfiguration ที่พบบ่อย (เช่น S3 bucket เปิด public โดยไม่ตั้งใจ, security group เปิดพอร์ตกว้างเกินไป) ก่อนที่จะ apply จริง:

```yaml
security_scan:
  stage: validate
  image: aquasec/tfsec:latest
  script:
    - tfsec environments/production/
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
```

### สรุป Best Practices ทั้งหมด

| หลักการ | เหตุผล |
|---|---|
| แบ่งโค้ดเป็น module | นำกลับมาใช้ซ้ำได้ ลดโอกาสเขียนผิดจาก copy-paste |
| แยก environment ด้วยโฟลเดอร์ (หรือ workspace) | ลดความเสี่ยง apply ผิด environment |
| ไม่ commit state file เข้า Git | ป้องกัน secret รั่วไหลและ conflict ที่แก้ยาก |
| ไม่ hardcode secret ใน `.tf` | ป้องกันข้อมูลลับหลุดเข้า Git history ถาวร |
| ใช้ `sensitive = true` กับตัวแปรลับ | ซ่อนค่าจากผลลัพธ์บนหน้าจอ |
| plan ต้องผ่าน PR review ก่อน apply เสมอ | ป้องกันการเปลี่ยนแปลง infrastructure ที่ไม่ได้ตรวจสอบ |
| สแกนความปลอดภัยอัตโนมัติใน pipeline | จับ misconfiguration ก่อนที่จะกลายเป็นปัญหาจริง |

---

## Step 770: แบบฝึกหัด — เขียน Terraform Config ผ่าน Git Branch/PR และรัน Pipeline จำลอง

ถึงเวลาลงมือปฏิบัติจริง! แบบฝึกหัดนี้ออกแบบมาให้ทำได้โดย **ไม่ต้องมีบัญชี cloud provider จริง** — เราจะใช้ provider `local` ของ Terraform ที่จำลองการสร้าง "resource" เป็นไฟล์บนเครื่องแทน เพื่อให้ฝึกความเข้าใจ workflow เต็มรูปแบบได้โดยไม่มีค่าใช้จ่ายใด ๆ

### ขั้นตอนที่ 1: เตรียมโปรเจกต์และติดตั้ง Terraform

ตรวจสอบว่ามี Terraform ติดตั้งอยู่แล้ว:

```bash
terraform version
```

ถ้ายังไม่มี ให้ติดตั้งตามคู่มือทางการของ HashiCorp สำหรับระบบปฏิบัติการของคุณ จากนั้นเตรียมโฟลเดอร์ฝึกฝน:

```bash
mkdir -p ~/git-course/part-77-terraform-practice
cd ~/git-course/part-77-terraform-practice
git init
```

### ขั้นตอนที่ 2: เขียน Terraform Config จำลอง

สร้างไฟล์ `versions.tf`:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }
  }
}
```

สร้างไฟล์ `variables.tf`:

```hcl
variable "environment" {
  description = "ชื่อ environment ที่จำลอง"
  type        = string
  default     = "dev"
}

variable "app_name" {
  description = "ชื่อแอปพลิเคชันที่จำลอง deploy"
  type        = string
  default     = "portfolio-project"
}
```

สร้างไฟล์ `main.tf` — เราจะจำลอง "การสร้าง server config" ด้วย resource `local_file`:

```hcl
resource "local_file" "server_config" {
  filename = "${path.module}/output/${var.environment}-server-config.txt"

  content = <<-EOT
    # จำลอง Server Configuration
    application = ${var.app_name}
    environment = ${var.environment}
    managed_by  = terraform
    created_at  = ${timestamp()}
  EOT
}

resource "local_file" "readme" {
  filename = "${path.module}/output/README.md"
  content  = "โฟลเดอร์นี้ถูกสร้างโดย Terraform จำลอง infrastructure ของ ${var.app_name}\n"
}
```

สร้างไฟล์ `outputs.tf`:

```hcl
output "server_config_path" {
  description = "path ของไฟล์ config ที่จำลองสร้างขึ้น"
  value       = local_file.server_config.filename
}
```

สร้างไฟล์ `.gitignore`:

```gitignore
.terraform/
*.tfstate
*.tfstate.*
tfplan
output/
```

### ขั้นตอนที่ 3: Commit โครงไฟล์ตั้งต้นเข้า `main`

```bash
git add versions.tf variables.tf main.tf outputs.tf .gitignore
git commit -m "เริ่มต้นโปรเจกต์ Terraform จำลองสำหรับฝึกฝน"
```

### ขั้นตอนที่ 4: สร้าง Branch ใหม่เพื่อฝึกทำการเปลี่ยนแปลงผ่าน PR

```bash
git checkout -b infra/add-log-file-resource
```

แก้ไข `main.tf` เพิ่ม resource ใหม่ (จำลองการเพิ่ม "log file" ให้กับระบบ):

```hcl
resource "local_file" "log_file" {
  filename = "${path.module}/output/${var.environment}-app.log"
  content  = "log started for ${var.app_name} (${var.environment})\n"
}
```

Commit และ push branch:

```bash
git add main.tf
git commit -m "เพิ่ม resource จำลอง log file สำหรับ environment"
git push -u origin infra/add-log-file-resource
```

จากนั้นเปิด **Pull Request** จาก `infra/add-log-file-resource` เข้า `main` บน GitHub/GitLab ตามที่เคยฝึกมาแล้วใน Part ก่อนหน้า

### ขั้นตอนที่ 5: รัน Workflow เต็มรูปแบบด้วยมือก่อน (เพื่อความเข้าใจ)

ก่อนเชื่อมกับ pipeline จริง ให้ลองรัน workflow ทีละคำสั่งด้วยตัวเองก่อนเพื่อให้เห็นภาพครบ:

```bash
terraform init
terraform fmt -check -recursive
terraform validate
terraform plan -var="environment=dev" -out=tfplan
terraform apply tfplan
```

ตรวจสอบว่ามีไฟล์ถูกสร้างขึ้นจริงในโฟลเดอร์ `output/`:

```bash
ls output/
cat output/dev-server-config.txt
cat output/dev-app.log
```

ทดลองรัน `terraform plan` อีกครั้งโดยไม่แก้ไขอะไร ควรเห็นข้อความว่า **"No changes."** ซึ่งหมายความว่า infrastructure จริงตรงกับโค้ดแล้วอย่างสมบูรณ์:

```bash
$ terraform plan -var="environment=dev"

No changes. Your infrastructure matches the configuration.
```

ทดลองลบ resource ทั้งหมดทิ้งด้วย `terraform destroy` เพื่อฝึกความคุ้นเคย:

```bash
terraform destroy -var="environment=dev"
```

### ขั้นตอนที่ 6: จำลอง Pipeline ด้วยสคริปต์ (ไม่ต้องพึ่ง CI Server จริง)

เขียนสคริปต์ชื่อ `pipeline.sh` เพื่อจำลองขั้นตอนที่ CI/CD จริงจะทำ (validate → plan → แสดงผล → apply แบบต้องยืนยัน):

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "=== [Stage: validate] ==="
terraform fmt -check -recursive
terraform validate

echo ""
echo "=== [Stage: plan] ==="
terraform plan -var="environment=${1:-dev}" -out=tfplan
terraform show -no-color tfplan

echo ""
echo "=== รอการอนุมัติก่อน apply (จำลอง manual approval gate) ==="
read -r -p "ยืนยันการ apply plan นี้เข้า environment '${1:-dev}' หรือไม่? (yes/no): " CONFIRM

if [ "${CONFIRM}" = "yes" ]; then
  echo ""
  echo "=== [Stage: apply] ==="
  terraform apply tfplan
else
  echo "ยกเลิกการ apply"
  exit 1
fi
```

ให้สิทธิ์รันไฟล์และทดลองใช้งาน:

```bash
chmod +x pipeline.sh
./pipeline.sh dev
```

สคริปต์นี้จำลอง 3 stage เดียวกับที่เราเขียนไว้ใน `.gitlab-ci.yml` ของ Step 768 ทุกประการ (`validate`, `plan`, `apply` แบบ manual approval) เพียงแต่รันบนเครื่อง local เพื่อให้เห็นภาพ logic ก่อนไปตั้งค่าจริงบน CI server

### ขั้นตอนที่ 7: Merge PR แล้วอัปเดต Checklist

หลังทีม (หรือตัวเองในบทบาทผู้รีวิว) ตรวจสอบผลลัพธ์ `plan` และ approve PR แล้ว ให้ merge เข้า `main`:

```bash
git checkout main
git pull origin main
```

### Checklist สรุปแบบฝึกหัด

- [ ] เขียนไฟล์ `.tf` ครบชุด (`versions.tf`, `variables.tf`, `main.tf`, `outputs.tf`) และรัน `terraform init` สำเร็จ
- [ ] รันครบ workflow `init → plan → apply → destroy` ด้วยมืออย่างน้อยหนึ่งรอบ
- [ ] สร้าง branch แยกสำหรับการเปลี่ยนแปลง เปิด Pull Request และผ่านการรีวิวก่อน merge
- [ ] ไฟล์ state (`*.tfstate`) และโฟลเดอร์ `.terraform/` อยู่ใน `.gitignore` ไม่ถูก commit เข้า Git
- [ ] เขียนสคริปต์จำลอง pipeline ที่มี stage validate/plan/apply พร้อม manual approval gate
- [ ] เข้าใจว่าทำไม `terraform plan` ต้องถูกตรวจสอบก่อน `apply` เสมอ โดยเฉพาะเมื่อเห็นสัญลักษณ์ `-` หรือ `-/+`

---

## สรุป Part 77

ใน Part นี้เราได้เรียนรู้แนวคิด **Infrastructure as Code** และเครื่องมือ **Terraform** อย่างครบวงจร:

1. IaC แก้ปัญหา Snowflake Server และ Configuration Drift ที่เกิดจากการจัดการ infrastructure ด้วยมือ (ClickOps) โดยเปลี่ยนให้ infrastructure ทั้งหมดถูกอธิบายเป็นโค้ดที่ตรวจสอบและทำซ้ำได้ (Step 761)
2. เครื่องมือ IaC หลักในตลาดมีทั้ง Terraform (multi-cloud, declarative), Pulumi (เขียนด้วยภาษาโปรแกรมจริง), CloudFormation (เฉพาะ AWS) และ Ansible (เน้น configuration management) — หลักสูตรเลือกสอน Terraform เพราะเป็นมาตรฐานอุตสาหกรรมที่ผสานกับ Git ได้ดีที่สุด (Step 762)
3. โครงสร้างพื้นฐานของ Terraform ประกอบด้วย `terraform` block, `provider` block, `resource` block, `variable` และ `output` (Step 763)
4. Workflow หลักของ Terraform คือ `init → plan → apply → destroy` โดย `plan` คือขั้นตอนสำคัญที่สุดในการตรวจสอบก่อนลงมือจริงเสมอ (Step 764)
5. State file คือ "ความทรงจำ" ของ Terraform ที่บันทึกสถานะจริงของ infrastructure ห้าม commit เข้า Git เด็ดขาดเพราะมี sensitive data แบบ plaintext และเสี่ยง conflict ร้ายแรง (Step 765)
6. Remote State (เช่น S3 + DynamoDB) และ State Locking ทำให้ทีมทำงานร่วมกันได้อย่างปลอดภัยโดยไม่มีใครเขียนทับ state พร้อมกัน (Step 766)
7. โค้ด Terraform ต้องถูกจัดการผ่าน Git เหมือนโค้ดแอปพลิเคชัน — branch แยกทุกการเปลี่ยนแปลง, ผ่าน Pull Request review, ป้องกัน `main` ด้วย branch protection และ CODEOWNERS (Step 767)
8. นำ Terraform เข้า CI/CD pipeline เพื่อรัน `plan` อัตโนมัติบน Pull Request พร้อมโพสต์ผลลัพธ์เป็น comment และรัน `apply` อัตโนมัติแบบมี manual approval เมื่อ merge เข้า `main` (Step 768)
9. Best practices สำคัญ: แบ่งโค้ดเป็น module นำกลับมาใช้ซ้ำ, แยก environment ด้วยโฟลเดอร์หรือ workspace, และ**ห้าม hardcode secret ใน `.tf` ไฟล์เด็ดขาด** ให้ใช้ `sensitive = true` ร่วมกับ Secret Manager แทนเสมอ (Step 769)
10. ลงมือฝึกจริงด้วยการเขียน Terraform config จำลอง จัดการผ่าน Git branch/PR และรัน workflow เต็มรูปแบบผ่านสคริปต์ pipeline จำลองที่มี manual approval gate (Step 770)

Terraform และแนวคิด IaC คือรากฐานสำคัญของ DevOps ยุคใหม่ เมื่อรวมเข้ากับ Git workflow ที่เราเรียนมาตลอดหลักสูตรนี้ คุณจะสามารถจัดการ infrastructure ทั้งหมดขององค์กรได้อย่างปลอดภัย ตรวจสอบย้อนหลังได้ และทำงานเป็นทีมได้โดยไม่ต้องกลัว "Snowflake Server" อีกต่อไป

ใน Part ถัดไป เราจะเจาะลึกเรื่องที่เชื่อมโยงกับสิ่งที่เพิ่งเรียนไปโดยตรง นั่นคือการป้องกัน **secret** ต่าง ๆ (password, API key, token) ไม่ให้รั่วไหลเข้า Git repository ไม่ว่าจะเป็นในโค้ดแอปพลิเคชันหรือโค้ด Terraform ก็ตาม

**ต่อไป:** [Part 78: Secret Management และการป้องกันข้อมูลรั่วไหลใน Git](./part-078-secret-management.md)
