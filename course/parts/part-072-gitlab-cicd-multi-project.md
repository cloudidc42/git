# Part 72: GitLab CI/CD: Multi-project Pipelines, Dynamic Pipelines

> **Step ในหลักสูตรนี้:** Step 711–720
> **เฟส:** 7 — CI/CD เต็มรูปแบบด้วย GitHub Actions และ GitLab CI/CD
> **เป้าหมายของ Part นี้:** เข้าใจวิธีเชื่อม pipeline ข้าม project เข้าด้วยกันด้วย Multi-project Pipelines, แยก pipeline ย่อยออกจาก pipeline หลักด้วย Parent-child Pipelines, สร้างไฟล์ CI ขึ้นมาเองแบบ dynamic ตอน runtime, เข้าใจความแตกต่างระหว่าง Merge Request Pipelines กับ Branch Pipelines, รู้จัก Merge Trains, CI/CD Components รุ่นใหม่ที่มาแทน template แบบเดิม และเทคนิคทำให้ pipeline โดยรวมของทีมเร็วขึ้นอย่างเป็นระบบ

---

## สารบัญของ Part นี้

- Step 711: Multi-project Pipelines คืออะไร — trigger pipeline ข้าม project
- Step 712: `trigger:` keyword ใน `.gitlab-ci.yml` (downstream pipeline)
- Step 713: Parent-child Pipelines — แยก pipeline ย่อยออกจาก pipeline หลัก
- Step 714: Dynamic Pipelines — generate `.gitlab-ci.yml` ด้วย script ตอน runtime
- Step 715: Pipeline Schedules — ตั้งค่ารัน pipeline ตามเวลาผ่าน UI
- Step 716: Merge Request Pipelines vs Branch Pipelines ต่างกันอย่างไร
- Step 717: Merged Results Pipelines และ Merge Trains เบื้องต้น
- Step 718: CI/CD Components — แนวคิดใหม่แทนที่การ include template ธรรมดา
- Step 719: Pipeline Efficiency — เทคนิคลดเวลา pipeline โดยรวม
- Step 720: แบบฝึกหัด — สร้าง Parent-child หรือ Multi-project Pipeline สำหรับ 2 project

---

## Step 711: Multi-project Pipelines คืออะไร — trigger pipeline ข้าม project

จนถึงตอนนี้ในหลักสูตร เราคุ้นเคยกับ pipeline ที่อยู่ **ภายใน project เดียว** เท่านั้น — job ทั้งหมดถูกกำหนดใน `.gitlab-ci.yml` ของ repository เดียวกัน และรันจบภายในขอบเขตของ project นั้น

แต่ในองค์กรจริงที่มีขนาดใหญ่ขึ้น สถาปัตยกรรมของระบบมักถูกแบ่งออกเป็นหลาย repository (หรือที่เรียกว่า **microservices architecture** หรือ **multi-repo setup**) เช่น:

```
company-org/
├── backend-api/        (project แยกต่างหาก)
├── frontend-web/        (project แยกต่างหาก)
├── mobile-app/          (project แยกต่างหาก)
├── infrastructure/      (project แยกต่างหาก — Terraform/Helm)
└── shared-libraries/    (project แยกต่างหาก)
```

แต่ละ project มี `.gitlab-ci.yml` ของตัวเอง มี pipeline ของตัวเอง แยกออกจากกันโดยสิ้นเชิง คำถามคือ: **ถ้า `backend-api` deploy สำเร็จแล้ว เราอยากให้ `frontend-web` เริ่ม build/test ใหม่โดยอัตโนมัติ (เพราะ frontend เรียก API ผ่าน client ที่อาจต้อง regenerate จาก OpenAPI spec ใหม่) เราจะทำอย่างไร**

นี่คือปัญหาที่ **Multi-project Pipelines** ถูกออกแบบมาแก้โดยเฉพาะ

### นิยาม

> **Multi-project Pipeline คือกลไกที่ทำให้ pipeline ของ project หนึ่ง (upstream) สามารถ "trigger" ให้ pipeline ของอีก project หนึ่ง (downstream) เริ่มทำงานได้โดยอัตโนมัติ** แม้ว่าทั้งสอง project จะไม่มีความเกี่ยวข้องกันทาง Git เลยก็ตาม (คนละ repository, คนละ namespace, หรือแม้แต่คนละ GitLab instance)

```
┌─────────────────────┐        trigger        ┌─────────────────────┐
│   backend-api        │ ─────────────────────▶ │   frontend-web       │
│   (Upstream project) │                        │   (Downstream project)│
│                       │                        │                       │
│  pipeline: build →    │                        │  pipeline: build →    │
│  test → deploy        │                        │  test → deploy        │
└─────────────────────┘                        └─────────────────────┘
```

### ทำไมถึงต้องใช้ Multi-project Pipelines

1. **สถาปัตยกรรมแบบ microservices ที่แยก repo กันจริง ๆ** — แต่ละทีมเป็นเจ้าของ repo ของตัวเอง แต่ระบบต้องทำงานร่วมกัน
2. **Chain การ deploy ที่มีลำดับก่อนหลัง** — เช่น deploy database migration project ก่อน แล้วค่อย trigger backend deploy แล้วค่อย trigger frontend deploy ตามลำดับ
3. **Shared library ที่หลาย project ใช้ร่วมกัน** — เมื่อ `shared-libraries` มีการอัปเดตเวอร์ชันใหม่ อยากให้ project ที่พึ่งพามันทั้งหมด รัน pipeline ทดสอบใหม่โดยอัตโนมัติเพื่อเช็คว่า breaking change หรือไม่
4. **Infrastructure-as-Code แยกจาก Application code** — เช่น `infrastructure` project จัดการ Terraform ส่วน `backend-api` จัดการโค้ดแอปพลิเคชัน แต่ deploy ต้องรอ infra พร้อมก่อน

### เปรียบเทียบกับสิ่งที่เรารู้จักมาก่อน

ใน Part ก่อนหน้านี้เราเรียนเรื่อง `needs:` ที่ทำให้ job ข้าม stage ใน pipeline **เดียวกัน** รอกันได้แบบ DAG (Directed Acyclic Graph) — Multi-project Pipeline คือแนวคิดเดียวกันแต่ **ขยายขอบเขตออกไปข้าม project**

| แนวคิด | ขอบเขต | ใช้ keyword อะไร |
|---|---|---|
| `needs:` | job ภายใน pipeline เดียวกัน | `needs: [job_name]` |
| Parent-child pipeline | pipeline ย่อยภายใน project เดียวกัน | `trigger: include:` |
| Multi-project pipeline | pipeline ข้าม project (อาจข้าม instance) | `trigger: project:` |

### สิทธิ์ที่ต้องมี

การ trigger pipeline ข้าม project ได้ ผู้ที่ push commit ที่ trigger job ต้องมีสิทธิ์อย่างน้อย **Developer role** ใน downstream project ด้วย (หรือใช้ Trigger Token / CI/CD Job Token ที่ config permission ไว้อย่างเหมาะสม) ไม่เช่นนั้น GitLab จะปฏิเสธการ trigger ด้วยเหตุผลด้านความปลอดภัย — ไม่ให้ project ไหนก็ได้มา trigger pipeline ของ project อื่นแบบไม่มีการควบคุม

เราจะเจาะลึก syntax และการตั้งค่าจริงใน Step ถัดไป

---

## Step 712: `trigger:` keyword ใน `.gitlab-ci.yml` (downstream pipeline)

หัวใจของ Multi-project Pipeline คือ keyword ชื่อ **`trigger:`** ซึ่งเปลี่ยน job ธรรมดาให้กลายเป็น **"bridge job"** — job ที่ไม่รันคำสั่งเอง แต่ทำหน้าที่ไป trigger pipeline ของ project อื่นแทน

### Syntax พื้นฐานที่สุด

ในไฟล์ `.gitlab-ci.yml` ของ **upstream project** (เช่น `backend-api`):

```yaml
stages:
  - build
  - test
  - deploy
  - trigger-downstream

build_job:
  stage: build
  script:
    - echo "Building backend..."

test_job:
  stage: test
  script:
    - echo "Testing backend..."

deploy_job:
  stage: deploy
  script:
    - echo "Deploying backend to production..."
  environment:
    name: production

trigger_frontend_pipeline:
  stage: trigger-downstream
  trigger: my-group/frontend-web
  needs:
    - deploy_job
```

อธิบายทีละส่วน:

- `trigger: my-group/frontend-web` — ระบุ **full path** ของ downstream project (namespace/group + project name) ที่ต้องการไป trigger
- job นี้จะไม่มี `script:` เพราะ GitLab รู้ว่านี่คือ trigger job โดยอัตโนมัติเมื่อเห็น keyword `trigger:`
- เมื่อ `trigger_frontend_pipeline` รัน มันจะสร้าง pipeline ใหม่ใน `my-group/frontend-web` ทันที

### ระบุ branch ที่จะ trigger

โดย default การ trigger จะรัน pipeline บน **default branch** ของ downstream project แต่สามารถระบุ branch เฉพาะได้:

```yaml
trigger_frontend_pipeline:
  stage: trigger-downstream
  trigger:
    project: my-group/frontend-web
    branch: develop
```

สังเกตว่าเมื่อต้องการระบุ branch ต้องเปลี่ยนจาก `trigger: my-group/frontend-web` (แบบสั้น) มาเป็น mapping ที่มี key `project:` และ `branch:` แทน

### รอผลลัพธ์ของ downstream pipeline (Interactive/Blocking mode)

โดย default เมื่อ upstream job trigger downstream pipeline แล้ว job นั้นจะ**ถือว่า success ทันที** โดยไม่รอผลของ downstream pipeline เลย ซึ่งบางกรณีเราต้องการให้ upstream pipeline **รอ** ผลลัพธ์ของ downstream ก่อนว่าจะ pass หรือ fail — ทำได้ด้วย `strategy: depend`

```yaml
trigger_frontend_pipeline:
  stage: trigger-downstream
  trigger:
    project: my-group/frontend-web
    branch: main
    strategy: depend
```

เมื่อใส่ `strategy: depend` แล้ว:

- upstream job จะเปลี่ยนสถานะเป็น `running` และรอจนกว่า downstream pipeline จะจบ
- ถ้า downstream pipeline **สำเร็จ** → upstream job ก็จะ success ตาม
- ถ้า downstream pipeline **ล้มเหลว** → upstream job ก็จะ fail ตามไปด้วย ทำให้ pipeline หลักหยุดและแจ้งเตือนได้ทันที

### ส่งตัวแปรไปยัง downstream pipeline

บ่อยครั้งที่ downstream pipeline ต้องการรู้ข้อมูลบางอย่างจาก upstream เช่น commit SHA, เวอร์ชันที่ deploy ไปแล้ว หรือชื่อ environment — ใช้ `variables:` ภายใน trigger job ได้:

```yaml
trigger_frontend_pipeline:
  stage: trigger-downstream
  trigger:
    project: my-group/frontend-web
    branch: main
    strategy: depend
  variables:
    UPSTREAM_COMMIT_SHA: $CI_COMMIT_SHA
    UPSTREAM_PROJECT_NAME: $CI_PROJECT_NAME
    API_VERSION: "v2.3.1"
```

ตัวแปรเหล่านี้จะปรากฏใน downstream pipeline เป็น CI/CD variable ธรรมดา สามารถใช้ใน `.gitlab-ci.yml` ของ `frontend-web` ได้ทันที เช่น:

```yaml
# .gitlab-ci.yml ของ frontend-web (downstream)
regenerate_api_client:
  stage: build
  script:
    - echo "Regenerating client for API version $API_VERSION"
    - echo "Triggered by commit $UPSTREAM_COMMIT_SHA from $UPSTREAM_PROJECT_NAME"
```

> **ข้อควรระวัง:** ตัวแปรที่ถูกกำหนดไว้ใน downstream project (project-level หรือ group-level CI/CD variables) จะ**มีความสำคัญเหนือกว่า**ตัวแปรที่ส่งมาจาก upstream หากชื่อซ้ำกัน เพื่อป้องกันไม่ให้ upstream project มา override ค่าที่สำคัญ เช่น production secret โดยไม่ได้ตั้งใจ

### จำกัด downstream pipeline ที่ trigger พร้อมกัน (rate limiting)

หากมีหลาย job ใน upstream trigger downstream project เดียวกันพร้อมกันซ้ำ ๆ อาจทำให้เกิด pipeline ล้น ควบคุมได้ด้วยการใช้ `rules:` ร่วมกับ trigger job ตามปกติ เช่น trigger เฉพาะตอน push เข้า `main`:

```yaml
trigger_frontend_pipeline:
  stage: trigger-downstream
  trigger:
    project: my-group/frontend-web
    strategy: depend
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

### การดู downstream pipeline จากหน้า UI

เมื่อ trigger job รันเสร็จ ในหน้า Pipeline ของ upstream project (`backend-api`) จะเห็นไอคอนลูกศรเล็ก ๆ ข้าง job ที่เป็น bridge job พร้อมลิงก์ไปยัง downstream pipeline โดยตรง — ทำให้ engineer สามารถกด click ตามไปดู log ของ downstream ได้ทันทีโดยไม่ต้องสลับไปหา project เอง ซึ่งเป็นประสบการณ์การใช้งานที่ GitLab ออกแบบมาให้ดูเป็น pipeline เดียวกันแม้จริง ๆ จะเป็นคนละ project

### ข้อจำกัดสำคัญที่ต้องรู้

1. **ผู้ trigger ต้องมีสิทธิ์อย่างน้อย Developer ใน downstream project** (หรือใช้ CI/CD job token ที่ downstream project เปิด allowlist ไว้ผ่าน **CI/CD job token permissions**)
2. **จำนวน downstream pipeline ที่ trigger ต่อเนื่องกัน (chain) มีขีดจำกัด** เพื่อป้องกัน infinite loop — ปัจจุบัน GitLab จำกัดที่ระดับความลึกหนึ่งเสมอ (สามารถตรวจสอบขีดจำกัดปัจจุบันได้จากเอกสาร GitLab เนื่องจากค่านี้อาจถูกปรับตามเวอร์ชัน)
3. Multi-project pipeline ทำงานได้ทั้งระหว่าง project ใน GitLab instance เดียวกัน และแม้กระทั่ง **ข้าม GitLab instance** (self-managed ↔ SaaS) ได้ ถ้าตั้งค่า Trigger Token ให้ถูกต้อง

---

## Step 713: Parent-child Pipelines — แยก pipeline ย่อยออกจาก pipeline หลัก

Multi-project pipeline เหมาะกับการเชื่อม pipeline ข้าม **project** แต่บางครั้งปัญหาที่เราเจอไม่ใช่การข้าม project เลย — เป็นแค่ปัญหาว่า **ไฟล์ `.gitlab-ci.yml` ของเราเองใน project เดียวมันใหญ่และซับซ้อนเกินไป** จน maintain ยาก

### ปัญหาที่ Parent-child Pipeline แก้

ลองนึกภาพ monorepo ขนาดใหญ่ที่มีหลาย component ภายใน repo เดียว:

```
my-monorepo/
├── services/
│   ├── auth-service/
│   ├── payment-service/
│   └── notification-service/
├── frontend/
│   ├── admin-panel/
│   └── customer-portal/
└── .gitlab-ci.yml
```

ถ้าเขียนทุกอย่างไว้ใน `.gitlab-ci.yml` ไฟล์เดียว จะกลายเป็นไฟล์ยาวหลายพันบรรทัด อ่านยาก แก้ยาก และที่แย่กว่านั้นคือ **pipeline graph บนหน้า UI จะรกมาก** เพราะทุก job ของทุก service ถูกแสดงปนกันหมด

### แนวคิดของ Parent-child Pipeline

> **Parent-child Pipeline คือการแยก `.gitlab-ci.yml` ออกเป็นไฟล์ parent (แม่) และไฟล์ child (ลูก) หลายไฟล์ โดย parent pipeline จะ trigger ให้ child pipeline แต่ละตัวรันแยกกัน ภายใน project เดียวกัน**

```
┌───────────────────────────────────────────┐
│              Parent Pipeline                │
│  (.gitlab-ci.yml)                           │
│                                               │
│  trigger_auth_pipeline ──────┐               │
│  trigger_payment_pipeline ───┼──┐            │
│  trigger_notification_pipeline ─┼──┐         │
└──────────────────────────────┼──┼──┼─────────┘
                                 │  │  │
                    ┌────────────▼┐┌▼──────────┐┌▼──────────────┐
                    │ Child Pipeline││ Child Pipeline││ Child Pipeline │
                    │ (auth-service)││(payment-service)││(notification)│
                    └───────────────┘└───────────────┘└───────────────┘
```

ข้อแตกต่างสำคัญจาก Multi-project pipeline คือ **ทุกอย่างยังอยู่ใน project เดียวกัน** — child pipeline ไม่ได้ไป trigger project อื่นเลย เพียงแต่ใช้ไฟล์ YAML คนละไฟล์และรันแยก pipeline กัน

### Syntax: `trigger: include:`

ในไฟล์ `.gitlab-ci.yml` (parent):

```yaml
stages:
  - trigger-services

trigger_auth_service:
  stage: trigger-services
  trigger:
    include: services/auth-service/.gitlab-ci.yml
    strategy: depend
  rules:
    - changes:
        - services/auth-service/**/*

trigger_payment_service:
  stage: trigger-services
  trigger:
    include: services/payment-service/.gitlab-ci.yml
    strategy: depend
  rules:
    - changes:
        - services/payment-service/**/*

trigger_notification_service:
  stage: trigger-services
  trigger:
    include: services/notification-service/.gitlab-ci.yml
    strategy: depend
  rules:
    - changes:
        - services/notification-service/**/*
```

และในไฟล์ `services/auth-service/.gitlab-ci.yml` (child):

```yaml
stages:
  - build
  - test
  - deploy

build_auth:
  stage: build
  script:
    - cd services/auth-service
    - echo "Building auth-service..."

test_auth:
  stage: test
  script:
    - cd services/auth-service
    - echo "Testing auth-service..."

deploy_auth:
  stage: deploy
  script:
    - cd services/auth-service
    - echo "Deploying auth-service..."
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

จุดที่น่าสนใจตรงนี้คือการใช้ `rules: changes:` ร่วมกับ trigger job — ทำให้ **child pipeline ของ service ไหนที่ไม่มีไฟล์เปลี่ยนแปลง จะไม่ถูก trigger เลย** ประหยัดทั้งเวลาและ compute resource อย่างมากในโปรเจกต์ monorepo ขนาดใหญ่

### ข้อดีของ Parent-child Pipeline

1. **แยกความรับผิดชอบ (separation of concerns)** — แต่ละทีมดูแลไฟล์ CI ของ service ตัวเองได้โดยไม่ต้องแตะไฟล์ parent
2. **Pipeline graph สะอาดขึ้นมาก** — หน้า UI แสดง child pipeline เป็นบล็อกแยก ไม่ปนกันทั้งหมด
3. **รันเฉพาะส่วนที่จำเป็น** — ผ่าน `rules: changes:` ทำให้ pipeline เร็วขึ้นมากในโปรเจกต์ monorepo
4. **จำกัดจำนวน job ต่อ pipeline ได้** — GitLab มีขีดจำกัดจำนวน job สูงสุดต่อ pipeline การแยกเป็น child pipeline ช่วยให้แต่ละ pipeline ย่อยไม่ชนขีดจำกัดนี้
5. **`needs:` แบบ cross-pipeline** — สามารถใช้ `needs: pipeline:` เพื่อดึง artifact จาก pipeline อื่นได้เช่นกัน (ดูหัวข้อถัดไป)

### Dynamic child pipeline count (สร้าง trigger job หลายตัวจาก matrix)

หากมี service จำนวนมากและไม่อยากเขียน trigger job ทีละตัว สามารถใช้ `parallel: matrix:` ร่วมกับ `trigger:` ได้เช่นกัน (ตั้งแต่ GitLab เวอร์ชันที่รองรับ):

```yaml
trigger_services:
  stage: trigger-services
  trigger:
    include: services/$SERVICE_NAME/.gitlab-ci.yml
    strategy: depend
  parallel:
    matrix:
      - SERVICE_NAME: [auth-service, payment-service, notification-service]
```

โค้ดนี้จะสร้าง trigger job 3 ตัวโดยอัตโนมัติ แต่ละตัว trigger child pipeline ของ service ที่ต่างกัน โดยไม่ต้องเขียนซ้ำ

### ข้อจำกัดของ Parent-child Pipeline

1. **ความลึกสูงสุดของ child pipeline คือ 2 ระดับ** — parent → child เท่านั้น ไม่สามารถให้ child มี grandchild (child ของ child) ได้ ถ้าต้องการ multi-level จริง ๆ ต้องผสมกับ Multi-project pipeline
2. **จำนวน child pipeline สูงสุดต่อ parent pipeline มีเพดานจำกัด** (ตรวจสอบค่าปัจจุบันจากเอกสาร GitLab เพราะอาจเปลี่ยนตามแผน subscription)
3. ถ้าไม่ใส่ `strategy: depend` parent pipeline จะไม่รอผล child และถือว่า trigger job success ทันที เหมือนกับ multi-project pipeline

---

## Step 714: Dynamic Pipelines — generate `.gitlab-ci.yml` ด้วย script ตอน runtime

Parent-child pipeline ที่เราเห็นใน Step 713 ใช้ไฟล์ YAML ที่เขียนไว้ล่วงหน้าแบบ **static** — เรารู้อยู่แล้วว่ามีไฟล์ไหนบ้าง แต่ในบางสถานการณ์เราไม่สามารถรู้ล่วงหน้าได้ว่า pipeline ควรมีหน้าตาอย่างไร จนกว่าจะถึงเวลารันจริง — นี่คือจุดที่ **Dynamic Pipeline** เข้ามาช่วย

### ปัญหาที่ Dynamic Pipeline แก้

ตัวอย่างสถานการณ์จริงที่ static YAML ไม่พอ:

1. **จำนวน job ขึ้นกับข้อมูลที่รู้เฉพาะตอน runtime** เช่น ต้องการสร้าง job แยกสำหรับแต่ละไฟล์ที่เปลี่ยนแปลงใน commit นั้น ๆ (จำนวนไฟล์ไม่แน่นอน)
2. **โครงสร้าง pipeline ขึ้นกับผลลัพธ์จากขั้นตอนก่อนหน้า** เช่น query จาก API ภายนอกว่ามี environment อะไรบ้างที่ต้อง deploy ในรอบนี้
3. **สร้าง job ตาม service ที่ query จาก service discovery / infrastructure inventory** แบบไดนามิก ไม่ได้ hardcode ไว้ในไฟล์
4. **Matrix ที่ซับซ้อนเกินกว่า `parallel: matrix:` จะรองรับ** เช่น combination ที่มีเงื่อนไขซับซ้อน

### หลักการทำงาน

> **Dynamic Pipeline คือการให้ job หนึ่งรัน script เพื่อ "สร้างไฟล์ YAML ขึ้นมาใหม่" แล้วส่งไฟล์นั้นเป็น artifact จากนั้นใช้ `trigger: include: artifact:` เพื่อให้ pipeline ถัดไป "รัน" ไฟล์ YAML ที่เพิ่งถูกสร้างขึ้นมานั้น**

ขั้นตอนทำงานแบ่งเป็น 2 job หลัก:

```
┌─────────────────────┐        ┌──────────────────────────┐
│  generate-config job │  ───▶  │  trigger job              │
│  (สร้างไฟล์ YAML     │ artifact│  (include: artifact ที่    │
│   ด้วย script)        │        │   generate-config สร้าง) │
└─────────────────────┘        └──────────────────────────┘
```

### ตัวอย่างเต็ม: Dynamic Pipeline

**ไฟล์ `.gitlab-ci.yml` หลัก:**

```yaml
stages:
  - generate
  - trigger

generate-config:
  stage: generate
  image: python:3.12-slim
  script:
    - python3 generate_pipeline.py > generated-config.yml
  artifacts:
    paths:
      - generated-config.yml

trigger-dynamic-pipeline:
  stage: trigger
  trigger:
    include:
      - artifact: generated-config.yml
        job: generate-config
    strategy: depend
```

อธิบาย:

- `generate-config` job รัน script Python (จะเป็นภาษาอะไรก็ได้ เช่น bash, Node.js, Python) เพื่อสร้างไฟล์ `generated-config.yml` ขึ้นมา แล้วเก็บมันไว้เป็น **artifact**
- `trigger-dynamic-pipeline` job ใช้ `include: artifact:` ระบุว่าให้ไปหยิบไฟล์ `generated-config.yml` จาก artifact ของ job `generate-config` มาใช้เป็นไฟล์ pipeline definition
- ผลลัพธ์คือ pipeline ใหม่ที่มีเนื้อหาตามที่ script generate ขึ้นมาแบบสด ๆ

### ตัวอย่าง script ที่ generate ไฟล์ YAML แบบไดนามิก

**`generate_pipeline.py`:**

```python
import os
import json

# สมมติว่าเรา query จาก environment inventory (หรืออ่านจากไฟล์ config ใน repo)
environments = ["staging-th", "staging-sg", "staging-vn"]

pipeline = {
    "stages": ["deploy"]
}

for env in environments:
    job_name = f"deploy_{env.replace('-', '_')}"
    pipeline[job_name] = {
        "stage": "deploy",
        "script": [
            f"echo 'Deploying to {env}'",
            f"./deploy.sh --env={env}"
        ],
        "environment": {
            "name": env
        }
    }

# GitLab CI รองรับ YAML แต่เขียนออกมาเป็น JSON ก็ใช้ได้ เพราะ JSON เป็น subset ของ YAML
print(json.dumps(pipeline, indent=2))
```

ผลลัพธ์ที่ได้จาก script นี้เมื่อรันจะหน้าตาประมาณนี้ (เขียนออกมาเป็นไฟล์ `generated-config.yml`):

```json
{
  "stages": ["deploy"],
  "deploy_staging_th": {
    "stage": "deploy",
    "script": ["echo 'Deploying to staging-th'", "./deploy.sh --env=staging-th"],
    "environment": {"name": "staging-th"}
  },
  "deploy_staging_sg": {
    "stage": "deploy",
    "script": ["echo 'Deploying to staging-sg'", "./deploy.sh --env=staging-sg"],
    "environment": {"name": "staging-sg"}
  },
  "deploy_staging_vn": {
    "stage": "deploy",
    "script": ["echo 'Deploying to staging-vn'", "./deploy.sh --env=staging-vn"],
    "environment": {"name": "staging-vn"}
  }
}
```

> **หมายเหตุสำคัญ:** GitLab CI ใช้ YAML parser ที่ยอมรับ JSON เป็นไฟล์อินพุตได้ด้วย เพราะ JSON เป็น subset ที่ถูกต้องของ YAML syntax เสมอ — เทคนิคนี้ทำให้เขียน generator script ด้วยภาษาโปรแกรมมิ่งทั่วไปที่มี JSON library ในตัวได้สะดวกกว่าการเขียน YAML string เอง

### ตัวอย่างที่ใช้ bash ล้วน (ไม่ต้องพึ่งภาษาอื่น)

```yaml
generate-config:
  stage: generate
  image: alpine:3.19
  script:
    - |
      echo "stages:" > generated-config.yml
      echo "  - deploy" >> generated-config.yml
      for service in $(cat services-changed.txt); do
        echo "deploy_${service}:" >> generated-config.yml
        echo "  stage: deploy" >> generated-config.yml
        echo "  script:" >> generated-config.yml
        echo "    - echo Deploying ${service}" >> generated-config.yml
      done
  artifacts:
    paths:
      - generated-config.yml
```

### ตรวจสอบ syntax ก่อนรันจริง (CI Lint)

เนื่องจากไฟล์ที่ generate ขึ้นมาไม่ได้ผ่านการ review จากคนโดยตรง ความเสี่ยงเรื่อง syntax ผิดพลาดมีสูงกว่าไฟล์ static ปกติ ควรเพิ่มขั้นตอนตรวจสอบก่อน trigger จริงเสมอ โดยใช้ GitLab CI Lint API:

```yaml
validate-generated-config:
  stage: generate
  needs: ["generate-config"]
  image: curlimages/curl:latest
  script:
    - |
      CONFIG_CONTENT=$(cat generated-config.yml)
      curl --request POST \
        --header "PRIVATE-TOKEN: $CI_LINT_TOKEN" \
        --form "content=<generated-config.yml" \
        "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/ci/lint"
```

การเพิ่ม validation step นี้ช่วยให้เรารู้ล่วงหน้าว่าไฟล์ที่ generate มาถูกต้องตาม schema ก่อนที่จะปล่อยให้ `trigger:` เอาไปรันจริง ลดโอกาสที่ pipeline จะ fail แบบงง ๆ เพราะ syntax error ในไฟล์ที่สร้างขึ้นเอง

### เมื่อไหร่ควรใช้ Dynamic Pipeline

Dynamic Pipeline เป็นเครื่องมือที่ทรงพลังแต่ก็เพิ่มความซับซ้อนให้ debug ยากขึ้น ควรใช้เฉพาะเมื่อ:

- โครงสร้าง pipeline **ไม่สามารถรู้ล่วงหน้าได้จริง ๆ** และขึ้นกับ runtime data
- `parallel: matrix:` และ `rules:` แบบ static ไม่เพียงพอต่อ logic ที่ซับซ้อน
- ทีมมีความเข้าใจเรื่อง CI/CD ที่ลึกพอจะ maintain generator script ได้อย่างมีคุณภาพ

หากโครงสร้าง pipeline สามารถเขียนแบบ static ได้ (รู้ล่วงหน้าว่ามีกี่ job อะไรบ้าง) ควรเลือกใช้ static YAML หรือ Parent-child Pipeline ก่อนเสมอ เพราะอ่านง่ายกว่า debug ง่ายกว่า และไม่มีความเสี่ยงเรื่อง script generate ผิดพลาด

---

## Step 715: Pipeline Schedules — ตั้งค่ารัน pipeline ตามเวลา (cron syntax) ผ่าน UI

จนถึงตอนนี้ pipeline ทุกอันที่เราเห็นถูก trigger ด้วยเหตุการณ์บางอย่าง เช่น push commit, เปิด merge request หรือถูก trigger จาก pipeline อื่น แต่ในงานจริงมีหลายกรณีที่เราต้องการให้ pipeline **รันเองตามเวลาที่กำหนด** โดยไม่ต้องมีใครมา push โค้ดเลย

### กรณีใช้งานทั่วไปของ Pipeline Schedule

1. **Nightly build / Nightly test** — รันชุดทดสอบทั้งหมด (รวมถึง test ที่ใช้เวลานานมาก เช่น E2E test แบบเต็ม) ทุกคืนตอนดึก
2. **Dependency update check** — เช็คทุกสัปดาห์ว่ามี dependency เวอร์ชันใหม่ที่ต้องอัปเดตหรือไม่ (เช่นรัน `npm outdated` หรือ Dependabot-style job)
3. **Database backup / cleanup job** — รัน job ที่ backup ข้อมูลหรือลบข้อมูลเก่าตามรอบเวลา
4. **Security scan รายวัน** — รัน SAST/DAST scan แบบเต็มรูปแบบทุกวันแม้ไม่มี commit ใหม่ เพื่อจับ vulnerability ที่เพิ่งถูกประกาศออกมาใหม่ (0-day) ในไลบรารีที่ใช้อยู่
5. **Metric/Report generation** — สร้างรายงานสรุปสถานะของระบบทุกสัปดาห์

### วิธีตั้งค่าผ่าน UI

Pipeline Schedule ถูกตั้งค่าผ่านหน้าเว็บ ไม่ใช่ผ่านไฟล์ `.gitlab-ci.yml` โดยตรง (แต่ job ที่จะรันยังคงมาจากไฟล์นั้น) ขั้นตอนคือ:

1. เข้าไปที่ project → **Build → Pipeline schedules**
2. กด **New schedule**
3. กรอกข้อมูล:
   - **Description** — ชื่ออธิบาย schedule เช่น "Nightly full test suite"
   - **Interval Pattern** — ใช้ **cron syntax มาตรฐาน** (5 field: นาที ชั่วโมง วันที่ เดือน วันในสัปดาห์)
   - **Cron Timezone** — เลือก timezone ที่ต้องการอ้างอิง
   - **Target branch or tag** — branch ที่จะใช้รัน pipeline นี้
   - **Variables** — CI/CD variable พิเศษที่จะส่งเข้าไปเฉพาะตอนรันจาก schedule นี้เท่านั้น

### ตัวอย่าง Cron Syntax ที่ใช้บ่อย

| Cron Expression | ความหมาย |
|---|---|
| `0 2 * * *` | ทุกวัน เวลา 02:00 น. |
| `0 */4 * * *` | ทุก 4 ชั่วโมง |
| `0 9 * * 1-5` | จันทร์ถึงศุกร์ เวลา 09:00 น. |
| `0 0 1 * *` | วันที่ 1 ของทุกเดือน เวลาเที่ยงคืน |
| `*/15 * * * *` | ทุก 15 นาที |
| `0 22 * * 5` | ทุกวันศุกร์ เวลา 22:00 น. |

### แยก job ที่รันเฉพาะตอนมาจาก Schedule เท่านั้น

ในไฟล์ `.gitlab-ci.yml` เราสามารถใช้ตัวแปร **`$CI_PIPELINE_SOURCE`** เพื่อเช็คว่า pipeline นี้ถูก trigger มาจาก schedule หรือไม่ แล้วกำหนดว่า job ไหนควรรันเฉพาะกรณีนี้:

```yaml
nightly_full_e2e_test:
  stage: test
  script:
    - echo "Running full E2E test suite (takes ~45 minutes)..."
    - npm run test:e2e:full
  rules:
    - if: '$CI_PIPELINE_SOURCE == "schedule"'

quick_smoke_test:
  stage: test
  script:
    - npm run test:smoke
  rules:
    - if: '$CI_PIPELINE_SOURCE != "schedule"'
```

รูปแบบนี้ทำให้ pipeline ปกติ (จาก push หรือ merge request) รันแค่ smoke test ที่เร็ว ในขณะที่ pipeline จาก schedule ตอนกลางคืนถึงจะรัน E2E test แบบเต็มที่ใช้เวลานาน — เป็นวิธีบาลานซ์ระหว่างความเร็วของ feedback loop ระหว่างวัน กับความครอบคลุมของการทดสอบ

### แยก schedule ตาม tag ด้วยตัวแปรที่ส่งมาจาก Schedule config

หากต้องการสร้างหลาย schedule ที่ trigger pipeline เดียวกันแต่ให้ทำงานต่างกัน สามารถกำหนด **Variables** ในหน้าตั้งค่า schedule แต่ละอันแยกกันได้ เช่น schedule "Nightly Thailand region" ตั้งค่า `DEPLOY_REGION=th` และ schedule "Nightly Singapore region" ตั้งค่า `DEPLOY_REGION=sg` แล้วในไฟล์ CI ใช้:

```yaml
scheduled_regional_check:
  stage: test
  script:
    - echo "Checking region $DEPLOY_REGION"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
```

วิธีนี้ทำให้ schedule ทำหน้าที่เหมือน "parameterized trigger" ได้โดยไม่ต้องเขียนไฟล์ CI แยกกันหลายชุด

### สิทธิ์และผู้เป็นเจ้าของ Schedule

Pipeline Schedule แต่ละตัวจะผูกกับ **ผู้ใช้ที่สร้างมันขึ้นมา (Owner)** — pipeline ที่รันจาก schedule นั้นจะรันด้วยสิทธิ์ของ owner คนนั้นเสมอ ถ้า owner ถูกลบออกจากทีมหรือ deactivate account, schedule จะหยุดทำงานโดยอัตโนมัติ ดังนั้นแนวปฏิบัติที่ดีคือให้ **service account** เป็นเจ้าของ schedule แทนที่จะเป็น account ส่วนบุคคลของพนักงาน เพื่อไม่ให้ schedule พังตอนพนักงานลาออก

### ตรวจสอบ log และผลลัพธ์ของ Scheduled Pipeline

Pipeline ที่รันจาก schedule จะปรากฏในหน้า **Pipelines** ตามปกติ พร้อมไอคอนนาฬิกาบ่งบอกว่าเป็น scheduled pipeline สามารถกด "Run pipeline schedule" ด้วยตนเองได้ทันทีจากหน้า Pipeline schedules เพื่อทดสอบว่า schedule ทำงานถูกต้องโดยไม่ต้องรอถึงเวลาจริง

---

## Step 716: Merge Request Pipelines vs Branch Pipelines ต่างกันอย่างไร

นี่คือหนึ่งในเรื่องที่ทำให้ทีมงงบ่อยที่สุดเมื่อเริ่มเขียน `rules:` ที่ซับซ้อนขึ้น เพราะ **pipeline ตัวเดียวกันสามารถถูกสร้างขึ้นจากหลายสาเหตุ (source) ที่ต่างกัน** และพฤติกรรมของ pipeline ควรต่างกันไปตามสาเหตุนั้น

### `$CI_PIPELINE_SOURCE` คือกุญแจสำคัญ

GitLab กำหนดตัวแปร `$CI_PIPELINE_SOURCE` ให้ทุก pipeline โดยอัตโนมัติ ค่าที่พบบ่อยที่สุดมีดังนี้:

| ค่า `$CI_PIPELINE_SOURCE` | เกิดขึ้นเมื่อ |
|---|---|
| `push` | มีการ push commit ไปยัง branch (Branch Pipeline) |
| `merge_request_event` | เปิดหรืออัปเดต Merge Request (Merge Request Pipeline) |
| `schedule` | รันจาก Pipeline Schedule |
| `web` | มีคนกด "Run pipeline" จากหน้าเว็บด้วยตนเอง |
| `api` | ถูก trigger ผ่าน API |
| `trigger` | ถูก trigger จาก trigger token |
| `pipeline` | ถูก trigger จาก multi-project/parent-child pipeline (bridge job) |

### Branch Pipeline คืออะไร

**Branch Pipeline** คือ pipeline ที่รันเมื่อมีการ **push commit ไปยัง branch โดยตรง** (ค่า `$CI_PIPELINE_SOURCE` เป็น `push`) นี่คือพฤติกรรม default ของ GitLab CI มาตั้งแต่ต้น — ทุกครั้งที่ push จะมี pipeline รันตาม branch นั้น ไม่ว่า branch นั้นจะมี merge request เปิดอยู่หรือไม่ก็ตาม

```
git push origin feature/new-login
       │
       ▼
Branch Pipeline รันบน branch "feature/new-login"
(ตัวแปร CI_COMMIT_BRANCH = "feature/new-login")
```

### Merge Request Pipeline คืออะไร

**Merge Request Pipeline (MR Pipeline)** คือ pipeline ที่รันเมื่อมีการ **เปิดหรืออัปเดต Merge Request** (ค่า `$CI_PIPELINE_SOURCE` เป็น `merge_request_event`) pipeline ประเภทนี้มีตัวแปรพิเศษเพิ่มเติมที่ Branch Pipeline ไม่มี เช่น:

- `$CI_MERGE_REQUEST_ID`
- `$CI_MERGE_REQUEST_IID`
- `$CI_MERGE_REQUEST_TARGET_BRANCH_NAME`
- `$CI_MERGE_REQUEST_SOURCE_BRANCH_NAME`
- `$CI_MERGE_REQUEST_TITLE`
- `$CI_MERGE_REQUEST_LABELS`

ตัวแปรเหล่านี้มีประโยชน์มาก เช่น การรัน job เฉพาะเมื่อ MR มี label บางอย่าง หรือแสดงผลลัพธ์ test แนบไปกับหน้า MR โดยตรง

### ทำไมถึงเกิด "Duplicate Pipeline" บ่อย

ปัญหาคลาสสิกที่ทีมมือใหม่เจอคือ: เมื่อ push commit เข้า branch ที่มี Merge Request เปิดอยู่แล้ว **GitLab จะสร้าง pipeline ขึ้นมา 2 ตัวพร้อมกัน** — Branch Pipeline (จาก push) และ Merge Request Pipeline (จากการอัปเดต MR) ทำให้ CI runner ทำงานซ้ำซ้อนโดยไม่จำเป็น เปลืองเวลาและ compute resource เป็นสองเท่า

### วิธีแก้ปัญหา Duplicate Pipeline ด้วย `workflow: rules:`

วิธีที่ GitLab แนะนำอย่างเป็นทางการคือใช้ **`workflow: rules:`** เพื่อควบคุมว่า pipeline ควรถูกสร้างขึ้นในกรณีไหนบ้างที่ระดับบนสุดของไฟล์:

```yaml
workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'
    - if: '$CI_COMMIT_TAG'
```

pattern นี้เรียกว่า **"MR-first strategy"** ความหมายคือ:

1. ถ้า pipeline นี้เกิดจาก Merge Request → **สร้าง pipeline** (ให้ไปรันเป็น MR Pipeline)
2. ถ้า push ตรงเข้า branch `main` → **สร้าง pipeline** (ให้ไปรันเป็น Branch Pipeline เฉพาะ main)
3. ถ้ามีการสร้าง tag → **สร้าง pipeline**
4. **กรณีอื่นทั้งหมด (เช่น push ธรรมดาเข้า feature branch ที่มี MR เปิดอยู่แล้ว) → ไม่สร้าง pipeline เลย**

ผลลัพธ์คือเมื่อ engineer push commit เข้า feature branch ที่มี MR เปิดอยู่ จะมีแค่ **MR Pipeline เดียว** รันขึ้นมา ไม่มี Branch Pipeline ซ้ำซ้อนอีกต่อไป

### ตารางเปรียบเทียบสรุป

| คุณสมบัติ | Branch Pipeline | Merge Request Pipeline |
|---|---|---|
| Trigger จาก | push commit เข้า branch | เปิด/อัปเดต Merge Request |
| `$CI_PIPELINE_SOURCE` | `push` | `merge_request_event` |
| มีตัวแปร `CI_MERGE_REQUEST_*` | ไม่มี | มี |
| แสดงผลใน MR widget | ไม่แสดงโดยตรง | แสดงสถานะ pipeline ในหน้า MR ทันที |
| เหมาะกับ | main/production branch, tag release | feature branch ที่กำลังพัฒนาและรีวิว |

### แนวปฏิบัติที่แนะนำ

โปรเจกต์สมัยใหม่ส่วนใหญ่ใช้กลยุทธ์ **"ใช้ MR Pipeline เป็นหลักสำหรับ feature branch ทั้งหมด และใช้ Branch Pipeline เฉพาะ default branch/production branch"** ตามรูปแบบ `workflow: rules:` ด้านบน เพราะ:

- MR Pipeline แสดงผลในหน้า MR ให้ reviewer เห็นสถานะได้ทันทีโดยไม่ต้องสลับหน้า
- ลด compute resource ที่สูญเปล่าจาก duplicate pipeline
- เปิดทางไปสู่การใช้ **Merged Results Pipeline** และ **Merge Trains** ที่จะพูดถึงใน Step ถัดไป ซึ่งต้องอาศัย MR Pipeline เป็นพื้นฐาน

---

## Step 717: Merged Results Pipelines และ Merge Trains เบื้องต้น

Step ที่แล้วเราเข้าใจแล้วว่า Merge Request Pipeline คือ pipeline ที่รันตอนเปิด/อัปเดต MR แต่ยังมีรายละเอียดสำคัญที่ต้องเข้าใจเพิ่ม: **MR Pipeline แบบธรรมดา ทดสอบ "โค้ดปัจจุบันของ source branch" เท่านั้น ไม่ได้ทดสอบผลลัพธ์หลัง merge จริง**

### ปัญหาคลาสสิก: "มัน pass ใน MR แต่พอ merge เข้า main แล้วพัง"

ลองนึกภาพสถานการณ์นี้:

1. Developer A เปิด MR แก้ไขไฟล์ `config.js` เปลี่ยนค่า default timeout จาก 30 เป็น 60 วินาที — MR pipeline ผ่านหมด
2. Developer B เปิด MR อีกอันแก้ไขไฟล์ `api.js` ที่เรียกใช้ `config.js` โดยอ้างอิงค่า timeout แบบ hardcode ใหม่ — MR pipeline ก็ผ่านหมดเช่นกัน (เพราะทดสอบแยกกันคนละ branch)
3. ทั้งสอง MR ถูก approve และ merge เข้า `main` ตามลำดับ
4. **ผลลัพธ์: โค้ดใน `main` พังทันที** เพราะการเปลี่ยนแปลงทั้งสองชน (conflict ทาง logic ไม่ใช่ conflict ทาง Git) แต่ไม่มี pipeline ไหนเคยทดสอบ "โค้ดหลัง merge ทั้งสองเข้าด้วยกันจริง ๆ" เลย

นี่คือช่องโหว่ที่ **Merged Results Pipeline** ถูกออกแบบมาปิด

### Merged Results Pipeline คืออะไร

> **Merged Results Pipeline คือ pipeline ที่ทดสอบ "ผลลัพธ์ที่จะเกิดขึ้นจริง หากนำ source branch ไป merge เข้ากับ target branch ล่าสุด" แทนที่จะทดสอบแค่ source branch เดี่ยว ๆ**

GitLab จะสร้าง **commit ชั่วคราว (temporary merge ref)** ที่จำลองผลลัพธ์การ merge ไว้ล่วงหน้า แล้วรัน pipeline บน commit ชั่วคราวนั้น:

```
target branch (main)  ──────●──────●──────●──── (commit ล่าสุดของ main)
                                            \
                                             \  (จำลอง merge)
source branch (feature) ────●──────●──────●──▶ ● ← Merged Results (ชั่วคราว)
                                                  รัน pipeline ตรงนี้
```

ถ้า pipeline บน merged results นี้ fail แปลว่าการ merge ครั้งนี้ **จะทำให้ main พัง** แม้ว่า pipeline ของ source branch เดี่ยว ๆ จะผ่านก็ตาม — ทำให้เราจับปัญหาได้ **ก่อน** ที่จะกด merge จริง ไม่ใช่หลังจากพังไปแล้ว

### วิธีเปิดใช้งาน

Merged Results Pipeline เปิดใช้งานผ่านหน้า **Settings → Merge requests** ของ project โดยเลือก:

- **Pipelines must succeed** (ควรเปิดคู่กันเสมอ เพื่อบังคับว่าต้อง pipeline ผ่านก่อนถึงจะกด merge ได้)
- **Merge pipelines** — เปิด merged results pipeline
- **Merge trains** — เปิดใช้ merge train (ต้องเปิด merge pipelines ก่อน)

เมื่อเปิดแล้ว MR pipeline ที่รันจะเปลี่ยนจากการทดสอบแค่ source branch มาเป็นการทดสอบ merged results โดยอัตโนมัติ ตัวแปร `$CI_MERGE_REQUEST_*` ยังคงใช้งานได้เหมือนเดิม

### Merge Trains คืออะไร

Merged Results Pipeline แก้ปัญหาได้ดีเมื่อมี MR เดียวที่กำลังจะ merge แต่ถ้ามี **หลาย MR ที่กำลังจะ merge เข้า branch เดียวกันพร้อม ๆ กัน** ก็ยังเกิดปัญหาได้อยู่ดี เพราะแต่ละ MR อาจ merge ทับกันในลำดับที่ไม่คาดคิด

> **Merge Train คือคิว (queue) ที่จัดลำดับ MR หลายตัวที่กำลังจะ merge เข้า branch เดียวกัน โดยแต่ละ MR ในคิวจะถูกทดสอบราวกับว่า MR ที่อยู่ก่อนหน้าตัวมันในคิว ได้ merge สำเร็จไปแล้วเรียบร้อย**

```
Merge Train เข้าคิวเข้า main:

MR #101 ──▶ ทดสอบ: main + MR#101
MR #102 ──▶ ทดสอบ: main + MR#101 + MR#102  (สมมติว่า #101 merge สำเร็จแล้ว)
MR #103 ──▶ ทดสอบ: main + MR#101 + MR#102 + MR#103
```

ถ้า MR #102 pipeline fail ระบบจะเอา #102 ออกจากขบวน (train) โดยอัตโนมัติ และจัดคิวใหม่ให้ MR #103 ทดสอบกับ `main + MR#101` แทน (ไม่รวม #102 ที่ fail) — ทำให้ขบวนเดินหน้าต่อได้โดยไม่ต้องรอให้ #102 แก้เสร็จก่อน

### ทำไม Merge Train ถึงสำคัญกับทีมขนาดใหญ่

ในทีมที่มี MR จำนวนมากถูก merge เข้า `main` ทุกวัน (หลักสิบ MR ต่อวันในทีมใหญ่) การ merge ทีละตัวโดยไม่มี queue จะทำให้เกิด "merge race condition" บ่อยมาก — MR ที่ merge ทีหลังอาจทำให้ MR ที่เพิ่ง merge ไปก่อนหน้าพังโดยไม่มีใครรู้ทันที Merge Train แก้ปัญหานี้อย่างเป็นระบบด้วยการบังคับให้ทุก MR ถูกทดสอบตามลำดับที่จะเกิดขึ้นจริงเสมอ

### ข้อควรพิจารณาก่อนเปิดใช้งาน

1. **ใช้ compute resource มากขึ้น** — เพราะแต่ละ MR ในคิวอาจต้องรัน pipeline ซ้ำหลายรอบเมื่อคิวเปลี่ยนลำดับ
2. **ต้องมี GitLab Premium/Ultimate tier ขึ้นไป** สำหรับ Merge Trains (Merged Results Pipeline ธรรมดาอาจมีอยู่ใน tier ที่ต่ำกว่า ขึ้นกับ subscription — ควรตรวจสอบ feature matrix ปัจจุบันของ GitLab ก่อนวางแผนใช้งานจริง)
3. **เหมาะกับทีมที่ merge บ่อยและ `main` ต้อง deployable ตลอดเวลา** (Trunk-based development ที่เข้มงวด) มากกว่าโปรเจกต์เล็กที่ merge ไม่บ่อย

---

## Step 718: CI/CD Components — แนวคิดใหม่แทนที่การ include template ธรรมดา

ใน Part ก่อนหน้านี้เราเคยเรียนเรื่อง `include:` สำหรับดึงไฟล์ YAML จากที่อื่นมาใช้ร่วมกัน (เช่น `include: project:`, `include: remote:`, `include: template:`) ซึ่งช่วยลดการเขียนโค้ดซ้ำได้ในระดับหนึ่ง แต่ GitLab พบว่าวิธีนี้ยังมีข้อจำกัดหลายอย่างเมื่อองค์กรต้องการ "แชร์ CI configuration แบบมีเวอร์ชัน (versioned), มี input ที่ตรวจสอบได้ (validated), และค้นหาได้ง่าย (discoverable)" — จึงเกิดแนวคิดใหม่ที่เรียกว่า **CI/CD Components**

### ปัญหาของการ `include:` template แบบเดิม

```yaml
# วิธีเดิม: include remote file ตรง ๆ
include:
  - project: 'my-group/ci-templates'
    ref: main
    file: '/templates/deploy-template.yml'
```

ข้อจำกัดของวิธีนี้:

1. **`ref: main` หมายความว่าถ้า template ต้นทางเปลี่ยนแปลง ทุก project ที่ include ไว้จะได้รับผลกระทบทันที** โดยไม่มีการควบคุมเวอร์ชันที่ชัดเจน (เว้นแต่จะ pin ไปที่ commit SHA เฉพาะเจาะจง ซึ่งก็จัดการยากเมื่อมีหลาย project)
2. **ไม่มีระบบ input validation** — ถ้า template ต้องการตัวแปรบางตัว ผู้ใช้ template ต้องไปอ่านเอกสารเองว่าต้องตั้งตัวแปรอะไรบ้าง ไม่มีกลไกบังคับหรือตรวจสอบ
3. **ค้นหายาก** — ไม่มีที่ทางการรวมศูนย์ (catalog) ให้ค้นหาว่าองค์กรมี template อะไรให้ใช้บ้าง
4. **ทดสอบยาก** — ไม่มี framework มาตรฐานสำหรับทดสอบ template ก่อนปล่อยใช้จริง

### CI/CD Component คืออะไร

> **CI/CD Component คือหน่วยของ pipeline configuration ที่ถูกออกแบบมาให้ reuse ได้ มีการกำหนดเวอร์ชันอย่างชัดเจน (semantic versioning), รับ input parameter ที่ type-checked ได้, และถูกเผยแพร่ผ่าน CI/CD Catalog ที่ค้นหาได้จากทั้งองค์กร**

โครงสร้างไฟล์ของ component โดยทั่วไป:

```
my-component-project/
├── templates/
│   └── deploy.yml
├── README.md
└── tests/
    └── deploy_test.yml
```

**`templates/deploy.yml`:**

```yaml
spec:
  inputs:
    environment:
      description: "ชื่อ environment ที่จะ deploy"
      type: string
      options: ["staging", "production"]
    version:
      description: "เวอร์ชันของ application ที่จะ deploy"
      type: string
      default: "latest"
    dry_run:
      description: "รันแบบ dry-run โดยไม่ deploy จริง"
      type: boolean
      default: false
---
deploy_component_job:
  stage: deploy
  image: alpine:3.19
  script:
    - echo "Deploying version $[[ inputs.version ]] to $[[ inputs.environment ]]"
    - |
      if [ "$[[ inputs.dry_run ]]" = "true" ]; then
        echo "Dry-run mode: skipping actual deploy"
      else
        ./deploy.sh --env=$[[ inputs.environment ]] --version=$[[ inputs.version ]]
      fi
```

จุดสำคัญที่ต่างจาก template เดิม:

- ส่วน **`spec: inputs:`** ที่อยู่บนสุด กำหนด **contract** ของ component ชัดเจนว่ารับ input อะไรบ้าง มี `type` (string, boolean, array, number), มี `options` จำกัดค่าที่รับได้, และมี `default`
- การอ้างอิง input ในเนื้อ script ใช้ syntax พิเศษ **`$[[ inputs.<name> ]]`** ซึ่งต่างจากตัวแปร CI/CD ปกติที่ใช้ `$VARIABLE_NAME`

### วิธีใช้งาน Component ใน project อื่น

```yaml
include:
  - component: gitlab.com/my-group/my-component-project/deploy@1.2.0
    inputs:
      environment: production
      version: "v3.4.1"
      dry_run: false
```

สังเกตความแตกต่างสำคัญ:

- ใช้ keyword **`component:`** แทน `project:` + `file:`
- ระบุเวอร์ชันด้วย **`@1.2.0`** ต่อท้าย path — เป็น semantic version ที่ผูกกับ **release/tag** ของ component project นั้นจริง ๆ ไม่ใช่แค่ branch ref แบบเดิม
- ส่ง `inputs:` ที่ตรงกับ `spec: inputs:` ที่ component กำหนดไว้ — ถ้าใส่ input ผิด type หรือใส่ค่าที่ไม่อยู่ใน `options` GitLab จะฟ้อง error ทันทีตอน parse pipeline ก่อนรันจริงด้วยซ้ำ

### รูปแบบการอ้างอิงเวอร์ชันที่รองรับ

| รูปแบบ | ความหมาย |
|---|---|
| `@1.2.0` | เวอร์ชันที่ชัดเจนแบบ pinned (แนะนำสำหรับ production) |
| `@1.2` | เวอร์ชันล่าสุดใน minor version 1.2.x |
| `@~latest` | เวอร์ชันล่าสุดที่ถูก release (ความเสี่ยงสูงกว่า เหมาะกับการทดลอง) |
| `@main` | ใช้เนื้อหาจาก branch `main` โดยตรง (เหมือน include เดิม ไม่แนะนำสำหรับ production) |

### CI/CD Catalog

Component ที่เผยแพร่อย่างถูกต้อง (ทำ release พร้อม README ที่อธิบายชัดเจน) จะไปปรากฏใน **CI/CD Catalog** ของ GitLab ซึ่งเป็นหน้ารวมศูนย์ที่ทุกคนในองค์กร (หรือ community ทั้งหมดสำหรับ public component) สามารถค้นหา, อ่านเอกสาร, และดูตัวอย่างการใช้งานได้ในที่เดียว — คล้ายกับ concept ของ GitHub Actions Marketplace แต่สำหรับ GitLab CI

### ทำไมองค์กรควรย้ายจาก Template ธรรมดาไปเป็น Component

1. **Semantic Versioning จริง** — ทีมที่ใช้ component สามารถอัปเกรดเวอร์ชันตอนไหนก็ได้ตามจังหวะของตัวเอง ไม่ถูกกระทบทันทีเมื่อ component ต้นทางเปลี่ยน
2. **Input validation ที่ระดับ parse time** — ลด error ที่เกิดจากการตั้งค่าตัวแปรผิดโดยไม่รู้ตัว
3. **Discoverability ผ่าน Catalog** — ทีมใหม่หา component ที่มีอยู่แล้วเจอง่าย ลดการเขียนซ้ำ (duplicate effort) ทั่วทั้งองค์กร
4. **Testable ด้วยตัวเอง** — component project สามารถมี pipeline ของตัวเองที่ทดสอบว่า component ทำงานถูกต้องก่อน release ทุกครั้ง
5. **แนวทางที่ GitLab สนับสนุนอย่างเป็นทางการในระยะยาว** — เอกสารและ template ทางการของ GitLab เองก็ทยอยเปลี่ยนไปใช้รูปแบบ Component มากขึ้นเรื่อย ๆ

### ข้อควรระวัง

Component เหมาะกับ configuration ที่ต้องแชร์ข้าม project จำนวนมากและมีวงจรชีวิต (lifecycle) ของตัวเองชัดเจน หากเป็นแค่ config เล็ก ๆ ที่ใช้เฉพาะภายใน project เดียว การสร้างเป็น component แยกต่างหากอาจเป็นการเพิ่มความซับซ้อนโดยไม่จำเป็น — ในกรณีนั้นใช้ `include: local:` หรือ YAML anchor ภายใน project เดียวกันก็เพียงพอแล้ว

---

## Step 719: Pipeline Efficiency — เทคนิคลดเวลา pipeline โดยรวม

เมื่อ pipeline ขององค์กรซับซ้อนขึ้นเรื่อย ๆ ตามที่เราเรียนมาตลอด Part นี้ ปัญหาที่ตามมาอย่างหลีกเลี่ยงไม่ได้คือ **pipeline ใช้เวลานานขึ้น** ซึ่งส่งผลโดยตรงต่อความเร็วของ feedback loop ที่ทีมได้รับ Step นี้จะรวบรวมเทคนิคหลักที่ใช้ลดเวลา pipeline โดยรวมอย่างเป็นระบบ

### 1. Parallel Jobs — กระจายงานให้รันพร้อมกัน

หลักการพื้นฐานที่สุดคือ **job ที่ไม่ได้พึ่งพากันควรรันพร้อมกัน ไม่ใช่รันเรียงต่อกัน** ใช้ `parallel:` เพื่อแตก test suite ขนาดใหญ่ออกเป็นหลาย job ย่อยที่รันพร้อมกัน:

```yaml
run_tests:
  stage: test
  parallel: 5
  script:
    - npm run test -- --shard=$CI_NODE_INDEX/$CI_NODE_TOTAL
```

`CI_NODE_INDEX` และ `CI_NODE_TOTAL` เป็นตัวแปรที่ GitLab กำหนดให้อัตโนมัติเมื่อใช้ `parallel:` บอกว่า job นี้คือลำดับที่เท่าไหร่จากทั้งหมดกี่ตัว เครื่องมือ test runner ส่วนใหญ่ (Jest, RSpec, pytest-split ฯลฯ) รองรับการรับค่านี้เพื่อแบ่ง test case ให้รันคนละส่วนกัน — ถ้าแบ่งเป็น 5 job ที่สมดุลกันดี เวลารวมของ test stage อาจลดลงเหลือประมาณ 1/5 ของเดิม

### 2. ออกแบบ DAG ด้วย `needs:` แทนการรอ stage ทั้งหมด

ดังที่เรียนใน Part ก่อนหน้า การใช้ `needs:` แทนการพึ่งพา `stage:` ตามลำดับ ทำให้ job เริ่มทำงานได้ทันทีที่ dependency ของมันเสร็จ ไม่ต้องรอให้ **ทุก job ใน stage ก่อนหน้า** เสร็จหมดก่อน:

```yaml
build_backend:
  stage: build
  script: [...]

build_frontend:
  stage: build
  script: [...]

test_backend:
  stage: test
  needs: ["build_backend"]
  script: [...]

test_frontend:
  stage: test
  needs: ["build_frontend"]
  script: [...]
```

ด้วยโครงสร้างนี้ `test_backend` ไม่ต้องรอ `build_frontend` เสร็จก่อน — มันเริ่มได้ทันทีที่ `build_backend` เสร็จ ทำให้ pipeline โดยรวมเร็วขึ้นอย่างมีนัยสำคัญเมื่อมี job จำนวนมากที่ไม่เกี่ยวข้องกัน

### 3. Cache ที่ออกแบบมาอย่างถูกต้อง

Cache ที่ตั้งค่าผิดพลาดเป็นสาเหตุอันดับต้น ๆ ที่ทำให้ pipeline ช้าโดยไม่จำเป็น หลักการสำคัญ:

```yaml
build_job:
  stage: build
  cache:
    key:
      files:
        - package-lock.json
    paths:
      - node_modules/
    policy: pull-push
  script:
    - npm ci
    - npm run build
```

- ใช้ **`key: files:`** ผูก cache key กับ hash ของไฟล์ lockfile — cache จะถูกสร้างใหม่เฉพาะเมื่อ dependency เปลี่ยนจริง ๆ ไม่ใช่ทุกครั้งที่ commit
- ใช้ **`policy: pull`** สำหรับ job ที่แค่ต้องการอ่าน cache (เช่น test job ที่ไม่ได้เปลี่ยน `node_modules`) เพื่อไม่ต้องเสียเวลา upload cache กลับซ้ำโดยไม่จำเป็น
- แยก cache key ตาม job type เมื่อแต่ละ job ใช้ dependency คนละชุด เพื่อไม่ให้ cache ของ job หนึ่งไป overwrite cache ของอีก job หนึ่ง

### 4. Interruptible Pipelines — ยกเลิก pipeline เก่าอัตโนมัติเมื่อมี commit ใหม่

ปัญหาที่พบบ่อยคือ developer push commit ใหม่เข้า branch เดิมซ้ำ ๆ ระหว่างที่ pipeline ของ commit ก่อนหน้ายังรันไม่เสร็จ ทำให้มี pipeline ค้างรันพร้อมกันหลายตัวโดยไม่มีประโยชน์ (เพราะสุดท้ายมีแค่ commit ล่าสุดที่สำคัญ) แก้ได้ด้วย `interruptible:`

```yaml
build_job:
  stage: build
  interruptible: true
  script:
    - npm run build

deploy_production:
  stage: deploy
  interruptible: false
  script:
    - ./deploy-to-prod.sh
```

เมื่อตั้ง `interruptible: true` และมี pipeline ใหม่ของ branch เดียวกันเริ่มรัน GitLab จะ **auto-cancel pipeline เก่าที่ยังไม่เสร็จทันที** ปล่อยให้ compute resource ว่างไปรัน pipeline ใหม่แทน ควรตั้ง `interruptible: false` เสมอสำหรับ job ที่มีผลข้างเคียงสำคัญ เช่น deploy จริงเข้า production เพื่อไม่ให้ deploy ถูกขัดจังหวะกลางคัน

นอกจากนี้ควรเปิด **Auto-cancel redundant pipelines** ที่ระดับ project settings (Settings → CI/CD → General pipelines) คู่กันไปด้วยเพื่อให้กลไกนี้ทำงานเต็มประสิทธิภาพ

### 5. เลือก base image ให้เล็กและเหมาะสม

Image ขนาดใหญ่ทำให้เวลาที่ใช้ pull image ในแต่ละ job นานขึ้นโดยไม่จำเป็น:

```yaml
# ช้ากว่า — image เต็มขนาดใหญ่
build_job:
  image: node:20

# เร็วกว่า — ใช้ alpine variant ที่เล็กกว่ามาก
build_job:
  image: node:20-alpine
```

ควรพิจารณาสร้าง **custom base image ที่ preinstall dependency ที่ใช้บ่อยไว้แล้ว** แทนการ `apt-get install` หรือ `npm install -g` ซ้ำทุกครั้งที่ job รัน — ลด job ที่ควรใช้เวลาไม่กี่วินาทีให้ไม่ต้องเสียเวลาหลายนาทีไปกับการติดตั้งเครื่องมือซ้ำ ๆ

### 6. จำกัดขอบเขต pipeline ด้วย `rules: changes:`

ในโปรเจกต์ monorepo การรันทุก job ทุกครั้งแม้ commit นั้นแก้แค่ไฟล์เดียวเป็นการสิ้นเปลืองมาก:

```yaml
test_backend:
  stage: test
  script: [...]
  rules:
    - changes:
        - backend/**/*

test_frontend:
  stage: test
  script: [...]
  rules:
    - changes:
        - frontend/**/*
```

ผสมกับแนวคิด Parent-child Pipeline จาก Step 713 จะทำให้ monorepo ขนาดใหญ่รันเฉพาะส่วนที่จำเป็นจริง ๆ เท่านั้น ประหยัดทั้งเวลาและ compute cost อย่างมาก

### 7. ใช้ Artifact อย่างประหยัด และตั้ง `expire_in` เสมอ

Artifact ที่ใหญ่เกินจำเป็นทำให้เวลา upload/download ช้าลง และกิน storage โดยไม่จำเป็น:

```yaml
build_job:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
    exclude:
      - dist/**/*.map
```

ควรส่งต่อเฉพาะไฟล์ที่ job ถัดไปต้องใช้จริงเท่านั้น ไม่ใช่ทั้งโฟลเดอร์ทำงานทั้งหมด และตั้ง `expire_in` เสมอเพื่อไม่ให้ storage ของ GitLab เต็มไปด้วย artifact เก่าที่ไม่มีใครใช้แล้ว

### 8. วัดผลก่อนปรับ (Pipeline Analytics)

ก่อนจะไล่ optimize ทุกจุดแบบเดา ควรเข้าไปดูหน้า **Analyze → CI/CD Analytics** ของ project เพื่อดูว่า stage ไหน หรือ job ไหน กินเวลามากที่สุดจริง ๆ การ optimize job ที่ใช้เวลาแค่ 10 วินาทีไม่ได้ช่วยอะไรมาก เมื่อเทียบกับการ optimize job ที่ใช้เวลา 20 นาที — ให้ข้อมูลจริงเป็นตัวนำทางการตัดสินใจเสมอ ไม่ใช่ความรู้สึก

### สรุปเทคนิคทั้งหมดในตารางเดียว

| เทคนิค | แก้ปัญหาอะไร |
|---|---|
| `parallel:` | Test suite ใหญ่รันช้าเพราะรันเรียงลำดับ |
| `needs:` (DAG) | job รอ stage ทั้งหมดจบโดยไม่จำเป็น |
| Cache ที่ดี (`key: files:`, `policy:`) | ติดตั้ง dependency ซ้ำทุกครั้งโดยไม่จำเป็น |
| `interruptible: true` + Auto-cancel | Pipeline เก่าค้างรันทั้งที่ไม่มีประโยชน์แล้ว |
| Base image เล็ก / custom image | เวลาเสียไปกับ pull image และติดตั้งเครื่องมือซ้ำ |
| `rules: changes:` + Parent-child | Monorepo รันทุกอย่างทั้งที่แก้แค่ส่วนเดียว |
| Artifact ที่จำกัดขอบเขต + `expire_in` | Storage เต็ม, upload/download ช้า |
| CI/CD Analytics | ไม่รู้ว่าควร optimize ตรงไหนก่อน |

---

## Step 720: แบบฝึกหัด — สร้าง Parent-child Pipeline หรือ Multi-project Pipeline สำหรับ 2 project

ถึงเวลาลงมือปฏิบัติจริง แบบฝึกหัดนี้แบ่งเป็น 2 ทางเลือก — ทำทางเลือกใดทางเลือกหนึ่งก็ได้ (แนะนำให้ทำทั้งสองถ้ามีเวลา เพื่อเทียบความแตกต่างด้วยตัวเอง)

### ทางเลือกที่ 1: Multi-project Pipeline (สร้าง 2 project จริง)

**ขั้นตอนที่ 1 — สร้าง project ต้นทาง (upstream)**

สร้าง project ใหม่ชื่อ `demo-backend` แล้วเพิ่มไฟล์ `.gitlab-ci.yml`:

```yaml
stages:
  - build
  - notify-frontend

build_backend:
  stage: build
  script:
    - echo "Building backend API..."
    - echo "BACKEND_VERSION=1.0.$CI_PIPELINE_IID" >> build.env
  artifacts:
    reports:
      dotenv: build.env

trigger_frontend:
  stage: notify-frontend
  trigger:
    project: <your-namespace>/demo-frontend
    branch: main
    strategy: depend
  variables:
    UPSTREAM_BACKEND_VERSION: $BACKEND_VERSION
  needs:
    - build_backend
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

**ขั้นตอนที่ 2 — สร้าง project ปลายทาง (downstream)**

สร้าง project ใหม่ชื่อ `demo-frontend` แล้วเพิ่มไฟล์ `.gitlab-ci.yml`:

```yaml
stages:
  - build

build_frontend:
  stage: build
  script:
    - echo "Building frontend, targeting backend version $UPSTREAM_BACKEND_VERSION"
    - echo "Frontend build complete"
```

**ขั้นตอนที่ 3 — ทดสอบ**

1. ตรวจสอบว่า user ที่ push commit เข้า `demo-backend` มีสิทธิ์อย่างน้อย Developer บน `demo-frontend`
2. Push commit เข้า `main` branch ของ `demo-backend`
3. เข้าไปดูหน้า Pipelines ของ `demo-backend` — ควรเห็น job `trigger_frontend` พร้อมไอคอนลิงก์ไปยัง downstream pipeline
4. คลิกตามไปดู pipeline ของ `demo-frontend` — ควรเห็น log ที่พิมพ์ค่า `UPSTREAM_BACKEND_VERSION` ที่ส่งมาจาก `demo-backend` ถูกต้อง

**เป้าหมายที่ต้องตรวจสอบให้ได้:**

- [ ] `trigger_frontend` สามารถ trigger pipeline ของ `demo-frontend` ได้สำเร็จ
- [ ] `demo-backend` pipeline รอผลของ `demo-frontend` จริง (ทดสอบโดยทำให้ `build_frontend` fail แล้วดูว่า `trigger_frontend` ใน backend เปลี่ยนเป็น failed ตามไหม)
- [ ] ตัวแปร `UPSTREAM_BACKEND_VERSION` ถูกส่งและอ่านค่าได้ถูกต้องใน downstream

### ทางเลือกที่ 2: Parent-child Pipeline (project เดียว จำลอง monorepo)

**ขั้นตอนที่ 1 — จัดโครงสร้างโฟลเดอร์**

สร้าง project เดียวชื่อ `demo-monorepo` แล้วจัดโครงสร้างไฟล์ดังนี้:

```
demo-monorepo/
├── .gitlab-ci.yml
├── service-a/
│   ├── .gitlab-ci.yml
│   └── app.txt
└── service-b/
    ├── .gitlab-ci.yml
    └── app.txt
```

**ขั้นตอนที่ 2 — เขียน parent pipeline**

`.gitlab-ci.yml` (root):

```yaml
stages:
  - trigger

trigger_service_a:
  stage: trigger
  trigger:
    include: service-a/.gitlab-ci.yml
    strategy: depend
  rules:
    - changes:
        - service-a/**/*

trigger_service_b:
  stage: trigger
  trigger:
    include: service-b/.gitlab-ci.yml
    strategy: depend
  rules:
    - changes:
        - service-b/**/*
```

**ขั้นตอนที่ 3 — เขียน child pipeline ของแต่ละ service**

`service-a/.gitlab-ci.yml`:

```yaml
stages:
  - build
  - test

build_service_a:
  stage: build
  script:
    - echo "Building service A"

test_service_a:
  stage: test
  script:
    - echo "Testing service A"
```

`service-b/.gitlab-ci.yml`:

```yaml
stages:
  - build
  - test

build_service_b:
  stage: build
  script:
    - echo "Building service B"

test_service_b:
  stage: test
  script:
    - echo "Testing service B"
```

**ขั้นตอนที่ 4 — ทดสอบพฤติกรรม `rules: changes:`**

1. Commit และ push การเปลี่ยนแปลงที่แก้เฉพาะไฟล์ใน `service-a/` เท่านั้น
2. ตรวจสอบว่า pipeline สร้าง**เฉพาะ** `trigger_service_a` และ child pipeline ของ service A เท่านั้น — `trigger_service_b` ไม่ควรถูกสร้างขึ้นเลย
3. ลองแก้ไฟล์ทั้งสอง service พร้อมกันในอีก commit หนึ่ง แล้วสังเกตว่าทั้งสอง trigger job ถูกสร้างขึ้นพร้อมกัน

**เป้าหมายที่ต้องตรวจสอบให้ได้:**

- [ ] Child pipeline ของแต่ละ service แยกกันชัดเจนในหน้า Pipeline graph
- [ ] `rules: changes:` ทำงานถูกต้อง — trigger เฉพาะ service ที่มีไฟล์เปลี่ยนแปลงจริง
- [ ] `strategy: depend` ทำงานถูกต้อง — ถ้า child pipeline fail, parent pipeline ต้อง fail ตาม (ลองแก้ script ของ `test_service_a` ให้ `exit 1` เพื่อทดสอบ)

### ส่วนเสริม (ถ้ามีเวลาเหลือ): ลองสร้าง Dynamic Pipeline ง่าย ๆ

สำหรับผู้ที่อยากทดลองเพิ่มเติม ลองสร้าง job ที่ generate ไฟล์ YAML ตามรายชื่อ service ที่อ่านจากไฟล์ `services.txt` แล้ว trigger ด้วย `include: artifact:` ตามที่เรียนใน Step 714 — ลองเปรียบเทียบว่าความยืดหยุ่นที่ได้มาแลกกับความซับซ้อนในการ debug มากขึ้นแค่ไหน เพื่อสร้างความเข้าใจเชิงลึกด้วยตัวเองว่าเมื่อไหร่ควรใช้ static parent-child pipeline และเมื่อไหร่ถึงจำเป็นต้องใช้ dynamic pipeline จริง ๆ

---

## สรุป Part 72

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Multi-project Pipeline** ทำให้ pipeline ของ project หนึ่งสามารถ trigger pipeline ของอีก project ได้ผ่าน keyword `trigger:` พร้อม `strategy: depend` เพื่อรอผลลัพธ์ และส่งตัวแปรข้าม project ได้ด้วย `variables:`
2. **Parent-child Pipeline** ใช้ `trigger: include:` เพื่อแยก `.gitlab-ci.yml` ที่ใหญ่และซับซ้อนออกเป็นไฟล์ย่อยภายใน project เดียวกัน ช่วยให้ pipeline graph สะอาดขึ้นและรันเฉพาะส่วนที่จำเป็นด้วย `rules: changes:`
3. **Dynamic Pipeline** ใช้ script generate ไฟล์ YAML ขึ้นมาตอน runtime แล้วส่งเป็น artifact ให้ `trigger: include: artifact:` นำไปรันต่อ เหมาะกับกรณีที่โครงสร้าง pipeline ไม่สามารถรู้ล่วงหน้าได้จริง ๆ
4. **Pipeline Schedule** ใช้ cron syntax ตั้งค่าผ่าน UI เพื่อรัน pipeline ตามเวลา และแยกพฤติกรรม job ด้วย `$CI_PIPELINE_SOURCE == "schedule"`
5. **Merge Request Pipeline vs Branch Pipeline** ต่างกันที่ `$CI_PIPELINE_SOURCE` และตัวแปรที่มีให้ใช้ ควรใช้ `workflow: rules:` เพื่อป้องกัน duplicate pipeline
6. **Merged Results Pipeline** ทดสอบผลลัพธ์หลัง merge จริงก่อนกด merge ส่วน **Merge Trains** จัดคิว MR หลายตัวให้ทดสอบตามลำดับที่จะเกิดขึ้นจริง ป้องกันปัญหา "pass ใน MR แต่พังหลัง merge"
7. **CI/CD Components** คือวิวัฒนาการของการ include template แบบเดิม มี semantic versioning, input validation ผ่าน `spec: inputs:`, และเผยแพร่ผ่าน CI/CD Catalog ที่ค้นหาได้
8. **Pipeline Efficiency** ทำได้หลายทาง ทั้ง `parallel:`, `needs:` (DAG), cache ที่ออกแบบถูกต้อง, `interruptible: true`, base image ที่เล็ก, `rules: changes:` และการวัดผลด้วย CI/CD Analytics ก่อนตัดสินใจ optimize

### Checklist ก่อนไป Part 73

- [ ] เข้าใจความแตกต่างระหว่าง Multi-project Pipeline และ Parent-child Pipeline ว่าต่างกันที่ขอบเขต (ข้าม project vs ภายใน project เดียวกัน)
- [ ] เขียน `trigger:` job พร้อม `strategy: depend` และส่งตัวแปรข้าม pipeline ได้
- [ ] เข้าใจหลักการของ Dynamic Pipeline และรู้ว่าเมื่อไหร่ควร/ไม่ควรใช้
- [ ] ตั้งค่า Pipeline Schedule ด้วย cron syntax ผ่าน UI ได้
- [ ] เข้าใจ `$CI_PIPELINE_SOURCE` และเขียน `workflow: rules:` เพื่อป้องกัน duplicate pipeline
- [ ] เข้าใจแนวคิด Merged Results Pipeline และ Merge Trains ว่าแก้ปัญหาอะไร
- [ ] เข้าใจ CI/CD Components และรู้ syntax `spec: inputs:` กับการอ้างอิงเวอร์ชันด้วย `@`
- [ ] รู้จักเทคนิคลดเวลา pipeline อย่างน้อย 5 เทคนิคจากตารางสรุปใน Step 719
- [ ] ทำแบบฝึกหัดสร้าง Multi-project หรือ Parent-child Pipeline สำเร็จอย่างน้อย 1 ทางเลือก

**ต่อไป:** [Part 73: Automated Testing ใน Pipeline: Unit, Integration, E2E](./part-073-automated-testing-pipeline.md)
