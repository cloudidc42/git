# Part 50: GitLab CI/CD เบื้องต้น: .gitlab-ci.yml แรกของคุณ

> **Step ในหลักสูตรนี้:** Step 491–500
> **เฟส:** 5 — เจาะลึก GitLab และ CI/CD เบื้องต้นของมัน
> **เป้าหมายของ Part นี้:** เข้าใจภาพรวมว่า CI/CD ของ GitLab ทำงานอย่างไร รู้จักโครงสร้างไฟล์ `.gitlab-ci.yml` แนวคิดเรื่อง Stages/Jobs เขียน pipeline แรกที่ใช้งานได้จริงตั้งแต่ build ไปจนถึง test เข้าใจ `script`/`before_script`/`after_script` อ่านผลลัพธ์ pipeline บน GitLab UI ได้ ควบคุมเงื่อนไขการรัน job ด้วย `rules` และ `only`/`except` ใช้ CI/CD Variables พื้นฐาน ติด badge สถานะ pipeline ไว้ใน README และปิดท้ายด้วยการลงมือเขียน pipeline ที่รัน test จริงกับโปรเจกต์ทดลองของตัวเอง

---

## สารบัญของ Part นี้

- Step 491: CI/CD ใน GitLab ทำงานอย่างไรในภาพรวม
- Step 492: ไฟล์ `.gitlab-ci.yml` อยู่ที่ root ของ repo — โครงสร้างพื้นฐานของไฟล์ YAML
- Step 493: แนวคิด Stages และ Jobs พื้นฐาน
- Step 494: เขียน pipeline แรกแบบง่ายที่สุด (stage build และ test)
- Step 495: `script`, `before_script`, `after_script` ใน job
- Step 496: การดู Pipeline status และ job log ใน GitLab UI (CI/CD > Pipelines)
- Step 497: `rules` (แบบใหม่แนะนำ) และ `only`/`except` (แบบเก่า) — เงื่อนไขการรัน job
- Step 498: CI/CD Variables พื้นฐาน (`variables:` ใน yml, predefined variables)
- Step 499: Pipeline badge (สถานะ passing/failing) ติดใน README.md
- Step 500: แบบฝึกหัด — เขียน pipeline แรกที่รัน test จริงให้โปรเจกต์ทดลอง

---

## Step 491: CI/CD ใน GitLab ทำงานอย่างไรในภาพรวม

จนถึง Part นี้ เราใช้ GitLab เพื่อเก็บโค้ด จัดการ branch และทำ Merge Request มาตลอด แต่จุดที่ทำให้ GitLab ถูกเรียกว่า "**DevOps Platform ครบวงจร**" อย่างแท้จริงคือความสามารถด้าน **CI/CD (Continuous Integration / Continuous Delivery / Continuous Deployment)** ที่ฝังมาในตัวโดยไม่ต้องติดตั้งเครื่องมือเสริมใด ๆ เพิ่มเติม

### CI/CD คืออะไรโดยสรุป

| คำย่อ | ชื่อเต็ม | ความหมาย |
|---|---|---|
| **CI** | Continuous Integration | ทุกครั้งที่มีคน push โค้ดเข้า repository ระบบจะ build และรัน test ให้อัตโนมัติทันที เพื่อจับข้อผิดพลาดให้เร็วที่สุด |
| **CD (Delivery)** | Continuous Delivery | โค้ดที่ผ่านการทดสอบแล้วจะถูกเตรียมพร้อมสำหรับปล่อยขึ้น production ได้ตลอดเวลา (แต่การ deploy จริงอาจต้องมีคนกดยืนยัน) |
| **CD (Deployment)** | Continuous Deployment | โค้ดที่ผ่านการทดสอบแล้วจะถูก deploy ขึ้น production **โดยอัตโนมัติทันที** ไม่ต้องรอคนกดยืนยัน |

พูดง่าย ๆ CI/CD คือการ **ให้เครื่องทำงานที่น่าเบื่อและต้องทำซ้ำ ๆ แทนมนุษย์** เช่น การรันคำสั่ง build, การรัน test, การตรวจสอบคุณภาพโค้ด (lint), และการ deploy ขึ้น server — แทนที่จะให้นักพัฒนาต้องมานั่งรันคำสั่งเหล่านี้ด้วยมือทุกครั้งที่แก้โค้ด

### ภาพรวมการทำงานของ GitLab CI/CD

GitLab CI/CD มีองค์ประกอบหลัก 3 ส่วนที่ทำงานร่วมกัน:

```
   ┌─────────────────────┐
   │  1. คุณ push โค้ด     │
   │  เข้า GitLab repo    │
   └──────────┬───────────┘
              │
              ▼
   ┌─────────────────────────────┐
   │ 2. GitLab อ่านไฟล์             │
   │   .gitlab-ci.yml ที่ root      │
   │   ของ repo แล้วสร้าง "pipeline" │
   └──────────┬───────────────────┘
              │
              ▼
   ┌─────────────────────────────┐
   │ 3. GitLab Runner รับงานไปทำจริง │
   │   (build, test, deploy ฯลฯ)    │
   │   ตามที่กำหนดไว้ใน pipeline      │
   └──────────┬───────────────────┘
              │
              ▼
   ┌─────────────────────────────┐
   │ 4. ผลลัพธ์ (สำเร็จ/ล้มเหลว)      │
   │   แสดงกลับมาใน GitLab UI        │
   └─────────────────────────────┘
```

1. **ไฟล์ `.gitlab-ci.yml`** — ไฟล์ตั้งค่าที่คุณเขียนเองเก็บไว้ที่ root ของ repository บอก GitLab ว่า "เมื่อมีการ push โค้ด ให้ทำอะไรบ้าง"
2. **Pipeline** — ชุดของงานทั้งหมดที่ถูกสร้างขึ้นมาตามไฟล์ `.gitlab-ci.yml` ในแต่ละครั้งที่มีการ push หรือสร้าง Merge Request (คิดง่าย ๆ ว่า pipeline คือ "การรันทั้งหมด 1 รอบ")
3. **GitLab Runner** — โปรแกรม/เครื่องที่ทำหน้าที่รันคำสั่งจริง ๆ ตามที่ pipeline สั่ง อาจเป็นเครื่องที่ GitLab.com เตรียมให้ฟรี (Shared Runner) หรือเครื่องของเราเองที่ตั้งค่าไว้ (Self-hosted Runner)
4. **Job และ Stage** — หน่วยย่อยของงานภายใน pipeline ซึ่งเราจะเจาะลึกใน Step 493

### ทำไมต้องเรียนเรื่องนี้ตอนนี้

Part นี้เป็น Part แรกที่แนะนำ GitLab CI/CD ให้คุณรู้จัก โดยเน้น **แค่พื้นฐานที่สุด** ให้คุณเขียน pipeline แรกได้ด้วยตัวเอง อ่านผลลัพธ์เป็น และเข้าใจภาพรวมว่ามันทำงานอย่างไร

> **หมายเหตุสำคัญ:** เนื้อหาใน Part นี้เป็นแค่การ "เกริ่นนำ" เท่านั้น เราจะกลับมาเจาะลึก GitLab CI/CD **แบบเต็มรูปแบบ** ในภายหลัง — โดยเฉพาะ **Part 71 และ Part 72** ที่จะพาไปดู `image`, `services`, `cache`, `artifacts`, `include`, `extends`, Docker-in-Docker, Multi-stage pipeline, Environments, Manual Deployment, Protected Variables, Security Scanning และหัวข้อขั้นสูงอื่น ๆ อย่างละเอียด รวมถึง **Part 51** ที่จะสอนเรื่อง GitLab Runner การตั้งค่าและใช้งานโดยเฉพาะ ดังนั้นใน Part นี้ขอให้โฟกัสแค่การ "เข้าใจภาพรวม" และ "เขียน pipeline พื้นฐานให้เป็น" ก็เพียงพอแล้ว

### สิ่งที่ต้องมีก่อนเริ่ม

- มีบัญชี GitLab (GitLab.com หรือ self-hosted ก็ได้ แต่ตัวอย่างในหลักสูตรนี้จะอ้างอิง GitLab.com เป็นหลัก)
- มี project/repository อย่างน้อย 1 อันบน GitLab ที่คุณมีสิทธิ์ push ได้
- GitLab.com มี **Shared Runner ให้ใช้ฟรีในโควตาที่กำหนด** สำหรับบัญชีทั่วไป จึงไม่จำเป็นต้องตั้ง Runner เองก็สามารถลองรัน pipeline ตาม Part นี้ได้ทันที (การตั้ง Runner เองจะสอนละเอียดใน Part 51)

---

## Step 492: ไฟล์ `.gitlab-ci.yml` อยู่ที่ root ของ repo — โครงสร้างพื้นฐานของไฟล์ YAML

หัวใจของ GitLab CI/CD ทั้งหมดคือไฟล์เดียว: **`.gitlab-ci.yml`**

### ตำแหน่งของไฟล์

ไฟล์นี้ต้องอยู่ที่ **root ของ repository เท่านั้น** (เว้นแต่จะไปตั้งค่า custom CI/CD configuration path ไว้ ซึ่งเป็นเรื่องขั้นสูงที่จะพูดถึงใน Part หลัง ๆ)

```
my-project/
├── .gitlab-ci.yml     ← ต้องอยู่ตรงนี้ ที่ root เท่านั้น
├── README.md
├── src/
│   └── app.js
└── package.json
```

ถ้าคุณสร้างไฟล์นี้ไว้ผิดที่ เช่นใส่ไว้ใน `src/.gitlab-ci.yml` หรือ `.gitlab/gitlab-ci.yml` GitLab จะ **ไม่เห็นไฟล์นี้เลย** และจะไม่มี pipeline ใด ๆ ถูกสร้างขึ้นมา

### ชื่อไฟล์ต้องขึ้นต้นด้วยจุด (`.`)

สังเกตว่าชื่อไฟล์คือ `.gitlab-ci.yml` — มีจุดนำหน้า (hidden file แบบ Unix) ไม่ใช่ `gitlab-ci.yml` เฉย ๆ ถ้าตั้งชื่อผิดแม้แค่ไม่มีจุดนำหน้า GitLab ก็จะไม่รู้จักไฟล์นี้เลยเช่นกัน

### YAML คืออะไร (พื้นฐานที่ต้องรู้ก่อนเขียน)

`.gitlab-ci.yml` เขียนด้วยภาษา **YAML** (ย่อมาจาก "YAML Ain't Markup Language") ซึ่งเป็นภาษาสำหรับเขียนไฟล์ตั้งค่าที่เน้นให้มนุษย์อ่านง่าย กฎพื้นฐานที่สำคัญมากมีดังนี้:

1. **ใช้ Indentation (การเยื้อง) ด้วย Space เท่านั้น ห้ามใช้ Tab เด็ดขาด** — ถ้าใช้ Tab ปนเข้าไป YAML จะ parse ไม่ได้ และ pipeline จะไม่ทำงาน
2. **Indent ต้องเท่ากันในระดับเดียวกันเสมอ** เช่น ถ้า key ระดับเดียวกันอยู่ที่ 2 space ทุกตัวต้องเป็น 2 space เหมือนกันหมด
3. **`key: value`** คือรูปแบบพื้นฐานที่สุด ต้องมี space หลังเครื่องหมาย `:` เสมอ
4. **List (array)** เขียนด้วยเครื่องหมาย `-` นำหน้าแต่ละรายการ
5. **String หลายบรรทัด** สามารถเขียนโดยใช้ `|` (คงการขึ้นบรรทัดใหม่ไว้) หรือ `>` (รวมเป็นบรรทัดเดียวโดยแทน newline ด้วย space)
6. **Comment** ใช้เครื่องหมาย `#`

ตัวอย่าง YAML พื้นฐาน:

```yaml
# นี่คือ comment
name: my-pipeline
version: 1

fruits:
  - apple
  - banana
  - orange

nested:
  key1: value1
  key2: value2
```

### โครงสร้างระดับบนสุดของ `.gitlab-ci.yml`

ไฟล์ `.gitlab-ci.yml` ประกอบด้วย **keyword ระดับบนสุด (top-level keywords)** ที่ควบคุมภาพรวมของ pipeline และ **job** ที่คุณตั้งชื่อเองได้อิสระ ตัวอย่างโครงสร้างขั้นต่ำสุดที่ยังใช้งานได้:

```yaml
stages:
  - build
  - test

my-first-job:
  stage: build
  script:
    - echo "สวัสดี GitLab CI/CD"
```

จาก YAML ด้านบน:

- `stages:` เป็น **top-level keyword** — บอก GitLab ว่ามีลำดับขั้นตอนอะไรบ้าง
- `my-first-job:` คือชื่อ **job** ที่เราตั้งเอง (ตั้งชื่ออะไรก็ได้ ยกเว้นคำสงวนบางคำ เช่น `stages`, `image`, `variables`, `include`, `default`, `workflow` ซึ่งเป็น keyword ของระบบ)
- ภายใน job มี `stage:` (job นี้อยู่ใน stage ไหน) และ `script:` (คำสั่งที่จะรันจริง)

### ตรวจสอบ syntax ของไฟล์ก่อน push จริง

GitLab มีเครื่องมือชื่อ **CI Lint** ที่ช่วยตรวจสอบว่าไฟล์ `.gitlab-ci.yml` ของคุณถูกต้องตาม syntax หรือไม่ ก่อนที่จะ push เข้าไปจริง เข้าถึงได้จากเมนู **CI/CD > Editor** ในโปรเจกต์ของคุณบน GitLab (หน้านี้จะมีแท็บ "Validate" ให้กดตรวจสอบ syntax แบบ real-time) การตรวจสอบก่อนทุกครั้งจะช่วยประหยัดเวลาได้มาก เพราะไม่ต้อง push ไปแล้วรอดูว่า pipeline พังหรือไม่จาก syntax error ง่าย ๆ

---

## Step 493: แนวคิด Stages และ Jobs พื้นฐาน

นี่คือแนวคิดที่สำคัญที่สุดในการเข้าใจ pipeline ของ GitLab — ความสัมพันธ์ระหว่าง **Stage** และ **Job**

### Job คืออะไร

**Job** คือหน่วยงานที่เล็กที่สุดใน pipeline — เป็นชุดคำสั่งที่ถูกรันจริงบน GitLab Runner หนึ่งตัว แต่ละ job มีชื่อเป็นของตัวเอง (คุณตั้งชื่อเอง) และต้องมี `script:` อย่างน้อย 1 รายการเสมอ (ไม่มี script ก็ไม่ถือเป็น job ที่ทำงานได้)

```yaml
run-unit-tests:
  script:
    - echo "รัน unit test"
```

### Stage คืออะไร

**Stage** คือ "กลุ่ม" ของ job ที่ถูกจัดลำดับการรัน — job หลายตัวสามารถอยู่ใน stage เดียวกันได้ และจะ **รันพร้อมกัน (parallel)** ในขณะที่ stage ต่าง ๆ จะ **รันเรียงกันตามลำดับ (sequential)**

กฎสำคัญที่ต้องจำให้ขึ้นใจ:

> **Job ที่อยู่ใน stage เดียวกัน จะรันพร้อมกัน (ถ้ามี Runner ว่างพอ)**
> **Stage ที่อยู่คนละ stage จะรันเรียงต่อกันไปตามลำดับที่ประกาศไว้ใน `stages:`**
> **Stage ถัดไปจะเริ่มก็ต่อเมื่อ job ทั้งหมดใน stage ก่อนหน้า "ผ่าน" หมดแล้วเท่านั้น**

### ภาพประกอบความสัมพันธ์ Stage-Job

สมมติเรามี `.gitlab-ci.yml` แบบนี้:

```yaml
stages:
  - build
  - test
  - deploy

build-frontend:
  stage: build
  script:
    - echo "build frontend"

build-backend:
  stage: build
  script:
    - echo "build backend"

unit-test:
  stage: test
  script:
    - echo "รัน unit test"

lint-check:
  stage: test
  script:
    - echo "ตรวจสอบ code style"

deploy-production:
  stage: deploy
  script:
    - echo "deploy ขึ้น production"
```

การรันจะมีลักษณะดังนี้:

```
Stage: build                Stage: test                 Stage: deploy
┌─────────────────┐        ┌─────────────────┐         ┌──────────────────┐
│ build-frontend   │        │ unit-test        │         │ deploy-production │
│ build-backend    │  ───▶  │ lint-check       │  ───▶   │                    │
│ (รันพร้อมกัน)      │        │ (รันพร้อมกัน)      │         │                    │
└─────────────────┘        └─────────────────┘         └──────────────────┘
      ต้องผ่านทั้งคู่ก่อน            ต้องผ่านทั้งคู่ก่อน
      ถึงจะไป stage ถัดไป            ถึงจะไป stage ถัดไป
```

- `build-frontend` และ `build-backend` อยู่ใน stage `build` เดียวกัน → รันพร้อมกัน
- เมื่อทั้งสอง job ใน `build` ผ่านหมดแล้ว ระบบถึงจะเริ่ม stage `test`
- `unit-test` และ `lint-check` รันพร้อมกันใน stage `test`
- ถ้า job ใดใน `test` ล้มเหลว **pipeline จะหยุดทันที และจะไม่เข้า stage `deploy` เลย** (ค่าเริ่มต้น)

### ถ้าไม่ประกาศ `stages:` เลยจะเกิดอะไรขึ้น

ถ้าไม่ระบุ `stages:` เอง GitLab จะใช้ค่าเริ่มต้นที่กำหนดไว้ในระบบ ซึ่งปัจจุบันประกอบด้วย stage มาตรฐานเช่น `.pre`, `build`, `test`, `deploy`, `.post` — แต่ **แนะนำอย่างยิ่งให้ประกาศ `stages:` เองเสมอ** เพื่อความชัดเจนและควบคุมได้เต็มที่ ไม่ต้องพึ่งพาค่า default ที่อาจเปลี่ยนแปลงได้ในอนาคต

### Stage พิเศษ `.pre` และ `.post`

GitLab มี stage พิเศษ 2 ตัวที่ไม่ต้องประกาศใน `stages:` ก็ใช้ได้เลย:

- **`.pre`** — รันก่อน stage แรกสุดเสมอ ไม่ว่าจะประกาศ `stages:` อย่างไร
- **`.post`** — รันหลัง stage สุดท้ายเสมอ

```yaml
notify-start:
  stage: .pre
  script:
    - echo "เริ่มต้น pipeline แล้ว"
```

หัวข้อนี้เป็นแค่ความรู้เสริม ยังไม่ต้องใช้งานจริงในตอนนี้ก็ได้ — เดี๋ยวจะเจอบ่อยขึ้นเมื่อ pipeline ซับซ้อนขึ้นใน Part หลัง ๆ

### สรุปคำศัพท์สำคัญของ Step นี้

| คำศัพท์ | ความหมาย |
|---|---|
| **Pipeline** | การรันทั้งหมด 1 รอบ ประกอบด้วยหลาย stage |
| **Stage** | กลุ่มของ job ที่รันเรียงตามลำดับกับ stage อื่น |
| **Job** | หน่วยงานเล็กที่สุด มี script ของตัวเอง รันบน Runner 1 ตัว |
| **Runner** | เครื่อง/โปรแกรมที่รันคำสั่งจริงตามที่ job สั่ง |

---

## Step 494: เขียน pipeline แรกแบบง่ายที่สุด (stage build และ test)

ถึงเวลาลงมือเขียน pipeline จริงเป็นครั้งแรก เราจะเริ่มจาก pipeline ที่ **เรียบง่ายที่สุดเท่าที่จะทำได้** เพื่อให้เห็นภาพรวมทั้งหมดก่อน

### ขั้นตอนที่ 1: สร้างไฟล์ `.gitlab-ci.yml`

ไปที่ root ของ repository ที่คุณเตรียมไว้ แล้วสร้างไฟล์ `.gitlab-ci.yml`:

```bash
cd ~/git-course/part-50-practice
touch .gitlab-ci.yml
```

### ขั้นตอนที่ 2: เขียนเนื้อหา pipeline พื้นฐาน

เปิดไฟล์ด้วย text editor แล้วใส่เนื้อหาดังนี้:

```yaml
stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "กำลัง build โปรเจกต์..."
    - echo "ติดตั้ง dependency (จำลอง)"
    - echo "Build เสร็จสมบูรณ์"

test-job:
  stage: test
  script:
    - echo "กำลังรัน test..."
    - echo "Test ทั้งหมดผ่าน"
```

อธิบายทีละส่วน:

- `stages:` ประกาศว่ามี 2 stage คือ `build` และ `test` และ `build` จะรันก่อนเสมอ
- `build-job` เป็นชื่อ job ที่เราตั้งเอง (ตั้งชื่ออื่นก็ได้ เช่น `compile`, `build_app`) อยู่ใน `stage: build`
- `script:` คือรายการคำสั่ง shell ที่จะถูกรันเรียงตามลำดับบน Runner — ในตัวอย่างนี้เป็นแค่คำสั่ง `echo` เพื่อจำลองว่ามีการทำงานเกิดขึ้น ยังไม่ใช่การ build จริง
- `test-job` อยู่ใน `stage: test` จะรันก็ต่อเมื่อ `build-job` ผ่านแล้วเท่านั้น

### ขั้นตอนที่ 3: Commit และ Push

```bash
git add .gitlab-ci.yml
git commit -m "เพิ่ม pipeline แรกด้วย stage build และ test"
git push origin main
```

### ขั้นตอนที่ 4: ดูผลลัพธ์

ทันทีที่ push เสร็จ GitLab จะตรวจพบไฟล์ `.gitlab-ci.yml` และเริ่มสร้าง pipeline ให้อัตโนมัติ ไปที่เมนู **CI/CD > Pipelines** ในโปรเจกต์บน GitLab เพื่อดูสถานะ (รายละเอียดการอ่านผลลัพธ์แบบเจาะลึกจะอยู่ใน Step 496)

### ทำไมตัวอย่างนี้ถึงใช้ `echo` แทนคำสั่งจริง

ในขั้นตอนแรกของการเรียนรู้ CI/CD เราจงใจใช้ `echo` แทนคำสั่ง build/test จริง เพื่อ:

1. ตัดความซับซ้อนเรื่องภาษาโปรแกรมมิ่ง, dependency, environment ออกไปก่อน
2. ให้เห็นภาพว่า **pipeline คือแค่การรันคำสั่ง shell ตามลำดับที่กำหนด** เท่านั้นเอง ไม่มีเวทมนตร์ซับซ้อนอะไร
3. เมื่อเข้าใจโครงสร้างแล้ว การเปลี่ยนจาก `echo "รัน test"` ไปเป็น `npm test` หรือ `pytest` หรือ `./run-tests.sh` จริง ๆ ก็เป็นเรื่องง่ายมาก (เราจะทำจริงใน Step 500)

### ตัวอย่างที่ใกล้เคียงการใช้งานจริงมากขึ้น (โปรเจกต์ Node.js)

เพื่อให้เห็นภาพว่าเมื่อโปรเจกต์มีภาษาโปรแกรมมิ่งจริง ๆ หน้าตาจะเป็นอย่างไร ลองดูตัวอย่างสำหรับโปรเจกต์ Node.js:

```yaml
stages:
  - build
  - test

image: node:18

build-job:
  stage: build
  script:
    - npm install
    - npm run build

test-job:
  stage: test
  script:
    - npm test
```

- `image: node:18` คือการบอกว่า job ทั้งหมดใน pipeline นี้ให้รันบน Docker image ชื่อ `node:18` (มี Node.js เวอร์ชัน 18 ติดตั้งพร้อมใช้งานอยู่แล้ว) — เรื่อง `image` จะเจาะลึกอย่างละเอียดใน Part 71-72 ตอนนี้แค่รู้ว่ามันมีอยู่ก็พอ
- `npm install` ติดตั้ง dependency จริง
- `npm run build` และ `npm test` คือคำสั่งจริงตาม `package.json` ของโปรเจกต์

ยังไม่ต้องรันตัวอย่าง Node.js นี้ก็ได้ถ้าโปรเจกต์ทดลองของคุณยังไม่มี Node.js — เก็บไว้เป็นแนวทางอ้างอิงในอนาคต

---

## Step 495: `script`, `before_script`, `after_script` ใน job

ใน job หนึ่ง ๆ นอกจาก `script:` แล้ว ยังมี keyword พิเศษอีก 2 ตัวที่ช่วยจัดการขั้นตอนก่อนและหลังการรันงานหลัก คือ `before_script` และ `after_script`

### `script` — คำสั่งหลักของ job

`script:` คือ **ส่วนบังคับ** ที่ทุก job ต้องมี เป็น list ของคำสั่ง shell ที่รันเรียงตามลำดับบนบรรทัด ถ้าคำสั่งใดคำสั่งหนึ่งใน `script` คืนค่า **exit code ที่ไม่ใช่ 0** (แปลว่าเกิดข้อผิดพลาด) job นั้นจะถูกทำเครื่องหมายว่า **failed ทันที** และคำสั่งที่เหลือใน `script` จะไม่ถูกรันต่อ

```yaml
test-job:
  stage: test
  script:
    - echo "ขั้นตอนที่ 1"
    - echo "ขั้นตอนที่ 2"
    - exit 1              # ทำให้ job นี้ fail ทันที
    - echo "บรรทัดนี้จะไม่ถูกรันเลย"
```

### `before_script` — คำสั่งที่รันก่อน `script` เสมอ

`before_script:` ใช้สำหรับงานเตรียมการที่ต้องทำก่อนงานหลักเสมอ เช่น ติดตั้ง dependency, ตั้งค่า environment, login เข้าระบบต่าง ๆ

```yaml
test-job:
  stage: test
  before_script:
    - echo "กำลังเตรียม environment..."
    - npm install
  script:
    - npm test
```

ลำดับการรันจริงของ job นี้คือ: `echo "กำลังเตรียม environment..."` → `npm install` → `npm test`

### `after_script` — คำสั่งที่รันหลังเสมอ ไม่ว่าผลจะเป็นอย่างไร

`after_script:` จะถูกรัน **เสมอ** ไม่ว่า `script` หลักจะสำเร็จหรือล้มเหลวก็ตาม เหมาะสำหรับงานทำความสะอาด (cleanup) เช่น ลบไฟล์ชั่วคราว, ปิด service, ส่งการแจ้งเตือน

```yaml
test-job:
  stage: test
  before_script:
    - npm install
  script:
    - npm test
  after_script:
    - echo "ทำความสะอาดหลังรันเสร็จ (รันเสมอไม่ว่า test จะผ่านหรือ fail)"
```

> **ข้อควรระวัง:** ถ้าคำสั่งใน `after_script` ล้มเหลว **จะไม่ทำให้สถานะโดยรวมของ job เปลี่ยนไป** (คือถ้า `script` ผ่านแต่ `after_script` fail, job จะยังถือว่า "ผ่าน" อยู่ดี) นี่เป็นพฤติกรรมที่ตั้งใจออกแบบไว้ เพราะ `after_script` มีไว้เพื่อ cleanup ไม่ใช่เพื่อตัดสินผลลัพธ์ของ job

### การตั้งค่าแบบ Global ด้วย `default:`

ถ้าหลาย ๆ job ในไฟล์เดียวกันต้องการ `before_script` เหมือนกันหมด ไม่จำเป็นต้องเขียนซ้ำในทุก job — สามารถใช้ top-level keyword `default:` เพื่อกำหนดค่าเริ่มต้นให้ทุก job ในไฟล์นั้นได้เลย

```yaml
stages:
  - build
  - test

default:
  before_script:
    - echo "เตรียมสภาพแวดล้อม (รันก่อนทุก job โดยอัตโนมัติ)"
    - npm install

build-job:
  stage: build
  script:
    - npm run build

test-job:
  stage: test
  script:
    - npm test
```

ในตัวอย่างนี้ ทั้ง `build-job` และ `test-job` จะรัน `echo "เตรียมสภาพแวดล้อม..."` และ `npm install` ก่อนเสมอ โดยที่ไม่ต้องเขียน `before_script` ซ้ำในแต่ละ job

> **หมายเหตุ:** ถ้า job ใดมีการประกาศ `before_script` ของตัวเองด้วย จะ **override (เขียนทับ)** ค่าจาก `default:` ทั้งหมด ไม่ใช่การรวมกัน — ต้องระวังเรื่องนี้ให้ดี

### สรุปลำดับการทำงานทั้งหมดของ job หนึ่งตัว

```
1. before_script (จาก default: หรือจาก job เอง — เลือกอย่างใดอย่างหนึ่ง)
2. script         (คำสั่งหลัก — ถ้า fail ตรงไหน job จะหยุดและ fail ทันที)
3. after_script   (รันเสมอ ไม่ว่าขั้นตอนที่ 2 จะสำเร็จหรือไม่)
```

---

## Step 496: การดู Pipeline status และ job log ใน GitLab UI (CI/CD > Pipelines)

เมื่อ push โค้ดที่มี `.gitlab-ci.yml` เข้าไปแล้ว ขั้นตอนสำคัญถัดมาคือการอ่านผลลัพธ์ให้เป็น มาดูกันว่าต้องไปดูที่ไหนและอ่านอย่างไร

### 496.1 เข้าไปที่หน้า Pipelines

ในโปรเจกต์บน GitLab ไปที่เมนูด้านซ้าย: **CI/CD > Pipelines**

หน้านี้จะแสดงรายการ pipeline ทั้งหมดที่เคยรัน เรียงจากล่าสุดไปเก่าสุด แต่ละแถวจะมีข้อมูลสำคัญ:

| คอลัมน์ | ความหมาย |
|---|---|
| **Status** | สถานะโดยรวมของ pipeline นี้ (ไอคอนวงกลม) |
| **Pipeline** | หมายเลข pipeline และ commit ที่ทริกเกอร์มัน |
| **Branch/Tag** | branch หรือ tag ที่ push เข้ามา |
| **Stages** | ไอคอนของแต่ละ stage แสดงสถานะย่อย |
| **Duration** | เวลาที่ใช้รันทั้งหมด |

### 496.2 ความหมายของสถานะ (Status) แต่ละแบบ

| ไอคอน/สถานะ | ความหมาย |
|---|---|
| **Passed** (เครื่องหมายถูกสีเขียว) | ทุก job ในทุก stage ผ่านหมด |
| **Failed** (กากบาทสีแดง) | มีอย่างน้อย 1 job ล้มเหลว |
| **Running** (วงกลมหมุนสีน้ำเงิน) | pipeline กำลังทำงานอยู่ |
| **Pending** | กำลังรอ Runner ว่างมารับงาน |
| **Canceled** | ถูกยกเลิกโดยผู้ใช้หรือระบบ |
| **Skipped** | job/stage ถูกข้ามไปตามเงื่อนไข (เช่นจาก `rules` หรือ `only`/`except`) |
| **Manual** | job ที่ต้องกดยืนยันเองก่อนจึงจะรัน (ตั้งค่าด้วย `when: manual`) |

### 496.3 ดูรายละเอียดแต่ละ Job

คลิกเข้าไปในหมายเลข pipeline ใด ๆ จะเห็น**หน้า Pipeline Detail** ที่แสดง job ทั้งหมดแยกตาม stage เป็นแผนภาพเชื่อมกัน (visualization) แบบนี้:

```
build              test               deploy
┌───────────┐     ┌───────────┐     ┌────────────────┐
│ build-job ✓│ ──▶ │ test-job ✓ │ ──▶ │ deploy-job (manual) │
└───────────┘     └───────────┘     └────────────────┘
```

คลิกที่ชื่อ job ใด ๆ จะเข้าไปดู **Job Log** ซึ่งคือ output แบบเต็มของทุกคำสั่งที่รันไป — เหมือนกับเปิด terminal แล้วรันคำสั่งเหล่านั้นเอง เห็นทุกบรรทัดที่ถูก `echo` ออกมา รวมถึง error message เต็ม ๆ ถ้ามี

### 496.4 อ่าน Job Log เมื่อ job ล้มเหลว

เมื่อ job ล้มเหลว สิ่งแรกที่ต้องทำคือเปิด **Job Log** แล้วเลื่อนหาบรรทัดสีแดงหรือข้อความ error ใกล้ท้ายสุดของ log เพราะ:

- Job Log จะแสดงคำสั่งทุกคำสั่งตามลำดับที่รันจริง พร้อม output ของมัน
- ท้าย log จะมีบรรทัดสรุปประมาณ `ERROR: Job failed: exit code 1` บอกชัดเจนว่า job จบด้วยสถานะ fail และ exit code เท่าไร
- ถ้าเป็น syntax error ของ YAML เอง (เช่น indent ผิด) GitLab จะแจ้งตั้งแต่หน้า **CI/CD > Pipelines** เลยโดยไม่มี pipeline ถูกสร้างขึ้นด้วยซ้ำ พร้อมข้อความอธิบายตำแหน่งที่ผิด

### 496.5 การ Retry และ Cancel

ที่มุมขวาบนของแต่ละ job หรือแต่ละ pipeline จะมีปุ่ม:

- **Retry** — รัน job (หรือทั้ง pipeline) นั้นซ้ำอีกครั้งโดยไม่ต้อง push commit ใหม่ มีประโยชน์มากเมื่อ job fail เพราะปัญหาชั่วคราว เช่น network หลุด
- **Cancel** — ยกเลิก job ที่กำลังรันอยู่ ถ้าพบว่าตั้งค่าผิดหรือไม่ต้องการให้รันต่อ

### 496.6 สถานะ pipeline บนหน้า Merge Request

เมื่อสร้าง Merge Request จาก branch ที่มี `.gitlab-ci.yml` ผลลัพธ์ pipeline ล่าสุดของ branch นั้นจะแสดงอยู่บนหน้า Merge Request โดยตรงด้วย ทำให้ทีมเห็นได้ทันทีว่าโค้ดที่เสนอมาผ่านการทดสอบหรือไม่ ก่อนจะกด merge จริง — นี่คือจุดที่ CI/CD เชื่อมเข้ากับ Workflow การทำงานเป็นทีมที่เราเรียนมาใน Part ก่อน ๆ อย่างแนบเนียน

---

## Step 497: `rules` (แบบใหม่แนะนำ) และ `only`/`except` (แบบเก่า) — เงื่อนไขการรัน job

ในทางปฏิบัติ เราแทบไม่เคยอยากให้ **ทุก job รันทุกครั้งในทุก branch** เช่น job สำหรับ deploy ขึ้น production ควรรันเฉพาะตอน push เข้า branch `main` เท่านั้น ไม่ใช่ทุก feature branch — นี่คือหน้าที่ของ `rules` และ `only`/`except`

### 497.1 `only` / `except` — วิธีเก่า (Legacy)

`only` และ `except` เป็น keyword รุ่นแรกที่ GitLab ใช้ควบคุมว่า job จะรันเมื่อไร:

```yaml
deploy-job:
  stage: deploy
  script:
    - echo "กำลัง deploy ขึ้น production"
  only:
    - main
```

job นี้จะรัน **เฉพาะเมื่อ push เข้า branch `main`** เท่านั้น ถ้า push เข้า branch อื่น job นี้จะถูกข้าม (skip) ไปเลย

```yaml
deploy-job:
  stage: deploy
  script:
    - echo "deploy"
  except:
    - schedules
```

job นี้จะรัน **ทุกกรณี ยกเว้น** เมื่อ pipeline ถูกทริกเกอร์จาก scheduled pipeline

`only`/`except` รับค่าได้ทั้งชื่อ branch, ชื่อ tag, หรือ keyword พิเศษเช่น `merge_requests`, `schedules`, `tags`, `branches`

> **สถานะปัจจุบันของ `only`/`except`:** GitLab ยังคงรองรับ syntax นี้อยู่เพื่อความเข้ากันได้กับ pipeline เก่า แต่ **เอกสารทางการของ GitLab แนะนำให้ใช้ `rules` แทนในโปรเจกต์ใหม่ทั้งหมด** เพราะ `rules` ยืดหยุ่นกว่ามาก คุณควรรู้จัก `only`/`except` ไว้เพื่ออ่านโค้ดเก่าให้เข้าใจ แต่ควรเขียนของใหม่ด้วย `rules`

### 497.2 `rules` — วิธีใหม่ที่แนะนำ

`rules:` เป็น list ของเงื่อนไข ที่ GitLab จะไล่ตรวจสอบ **จากบนลงล่างทีละข้อ** พอเจอเงื่อนไขแรกที่เป็นจริง จะใช้ค่า `when` ของเงื่อนไขนั้นทันทีแล้วหยุดตรวจข้อถัดไป

```yaml
deploy-job:
  stage: deploy
  script:
    - echo "กำลัง deploy ขึ้น production"
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: on_success
    - when: never
```

อธิบาย:

- `if:` คือเงื่อนไขที่เขียนด้วย syntax คล้าย expression ทั่วไป โดยอ้างอิง CI/CD variable (เราจะเจาะลึกเรื่อง variable ใน Step 498)
- `when: on_success` หมายถึง "ถ้าเงื่อนไขนี้เป็นจริง และ stage ก่อนหน้าผ่านหมด ให้รัน job นี้"
- บรรทัดสุดท้าย `when: never` (ไม่มี `if`) หมายถึง "กรณีอื่นทั้งหมดที่ไม่ตรงเงื่อนไขด้านบน ให้ไม่รัน job นี้เลย"

### 497.3 ค่าที่ใช้กับ `when` ได้

| ค่า | ความหมาย |
|---|---|
| `on_success` (ค่า default) | รัน job นี้ถ้า job ใน stage ก่อนหน้าทั้งหมดผ่าน |
| `on_failure` | รัน job นี้เฉพาะเมื่อมี job ใน stage ก่อนหน้าล้มเหลว |
| `always` | รันเสมอ ไม่สนใจผลลัพธ์ stage ก่อนหน้า |
| `never` | ไม่รัน job นี้เลยเมื่อเงื่อนไขนี้เป็นจริง |
| `manual` | ต้องมีคนกดปุ่ม "Run" ด้วยตัวเองใน GitLab UI ก่อน job ถึงจะเริ่มทำงาน |
| `delayed` | รอตามเวลาที่กำหนดก่อนเริ่มรัน (ใช้คู่กับ `start_in`) |

### 497.4 ตัวอย่าง `rules` ที่ใช้บ่อยในงานจริง

**ตัวอย่างที่ 1: รัน test ทุกครั้งที่มี Merge Request หรือ push เข้า main**

```yaml
test-job:
  stage: test
  script:
    - npm test
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'
```

**ตัวอย่างที่ 2: deploy ขึ้น production ต้องกดยืนยันเองเสมอ**

```yaml
deploy-production:
  stage: deploy
  script:
    - echo "deploy ขึ้น production"
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
    - when: never
```

**ตัวอย่างที่ 3: ข้าม job นี้ถ้า commit message มีคำว่า `[skip-deploy]`**

```yaml
deploy-job:
  stage: deploy
  script:
    - echo "deploy"
  rules:
    - if: '$CI_COMMIT_MESSAGE =~ /\[skip-deploy\]/'
      when: never
    - if: '$CI_COMMIT_BRANCH == "main"'
```

### 497.5 ตารางเปรียบเทียบ `rules` กับ `only`/`except`

| ประเด็น | `only` / `except` | `rules` |
|---|---|---|
| สถานะ | Legacy (ของเก่า) | แนะนำให้ใช้ในโปรเจกต์ใหม่ทั้งหมด |
| ความยืดหยุ่น | จำกัด รองรับแค่บาง keyword สำเร็จรูป | เขียน expression เองได้อิสระผ่าน `if` |
| รวมกับ `when: manual` ได้ไหม | ได้ แต่ต้องเขียนแยก `when:` ต่างหาก | รวมอยู่ในเงื่อนไขเดียวกันได้เลย |
| อ่านลำดับความสำคัญ | ตรวจแบบเงื่อนไขรวม (AND บางกรณี, OR บางกรณี — สับสนง่าย) | ไล่ตรวจจากบนลงล่าง เจอจริงก่อนใช้ก่อน อ่านง่ายกว่า |
| ใช้ร่วมกับ variable ซับซ้อนได้ไหม | ทำได้จำกัดมาก | ทำได้เต็มที่ |

> **ข้อควรจำ:** ห้ามใช้ `rules` และ `only`/`except` ปนกันใน job เดียวกัน — GitLab จะแจ้ง error ทันทีเพราะเป็นวิธีคิดคนละแบบที่ขัดแย้งกัน ให้เลือกใช้อย่างใดอย่างหนึ่งต่อ job

---

## Step 498: CI/CD Variables พื้นฐาน (`variables:` ใน yml, predefined variables)

**CI/CD Variables** คือตัวแปรที่ใช้เก็บค่าต่าง ๆ ที่ script ใน job สามารถเรียกใช้ได้ ไม่ว่าจะเป็นค่าที่เรากำหนดเอง หรือค่าที่ GitLab เตรียมไว้ให้อัตโนมัติ

### 498.1 การกำหนดตัวแปรเองด้วย `variables:`

สามารถประกาศ `variables:` ได้ทั้งระดับ **global** (ใช้ได้ทุก job ในไฟล์) และระดับ **job** (ใช้ได้เฉพาะ job นั้น)

```yaml
variables:
  APP_ENV: "production"
  NODE_VERSION: "18"

stages:
  - test

test-job:
  stage: test
  variables:
    DEBUG_MODE: "true"
  script:
    - echo "ENV: $APP_ENV"
    - echo "Node version ที่ต้องการ: $NODE_VERSION"
    - echo "Debug mode: $DEBUG_MODE"
```

ในตัวอย่างนี้ `test-job` เข้าถึงได้ทั้ง `$APP_ENV`, `$NODE_VERSION` (มาจาก global) และ `$DEBUG_MODE` (มาจาก job ตัวเอง)

> **ข้อสังเกตเรื่อง syntax:** ในไฟล์ `.gitlab-ci.yml` การอ้างอิงตัวแปรใน `script` ใช้ syntax แบบ shell คือ `$ชื่อตัวแปร` (บน Linux/macOS Runner) ส่วนใน `rules.if` ก็ใช้ `$ชื่อตัวแปร` เช่นกันแต่เป็นส่วนหนึ่งของ expression ที่ครอบด้วย single quote

### 498.2 Predefined Variables — ตัวแปรที่ GitLab เตรียมไว้ให้อัตโนมัติ

ทุก pipeline ที่รันจะมีชุดตัวแปรมาตรฐานที่ GitLab **สร้างและใส่ค่าให้อัตโนมัติ** โดยไม่ต้องประกาศเอง เรียกว่า **Predefined Variables (CI/CD Predefined Variables)** ตัวอย่างที่ใช้บ่อยที่สุด:

| ตัวแปร | ความหมาย |
|---|---|
| `$CI_COMMIT_BRANCH` | ชื่อ branch ที่ push เข้ามา (มีค่าเฉพาะตอนรันจาก branch ไม่ใช่ merge request pipeline) |
| `$CI_COMMIT_SHA` | Commit hash แบบเต็ม (40 ตัวอักษร) ของ commit ที่ทริกเกอร์ pipeline นี้ |
| `$CI_COMMIT_SHORT_SHA` | Commit hash แบบย่อ (8 ตัวอักษรแรก) |
| `$CI_COMMIT_MESSAGE` | ข้อความ commit message เต็ม |
| `$CI_PROJECT_NAME` | ชื่อโปรเจกต์ |
| `$CI_PROJECT_PATH` | path เต็มของโปรเจกต์ เช่น `mygroup/myproject` |
| `$CI_PIPELINE_ID` | หมายเลขเฉพาะของ pipeline นี้ในระดับทั้งอินสแตนซ์ |
| `$CI_PIPELINE_IID` | หมายเลขลำดับของ pipeline นี้เฉพาะในโปรเจกต์นี้ |
| `$CI_JOB_NAME` | ชื่อของ job ที่กำลังรันอยู่ ณ ขณะนั้น |
| `$CI_JOB_STAGE` | ชื่อ stage ของ job ที่กำลังรันอยู่ |
| `$CI_DEFAULT_BRANCH` | ชื่อ branch หลักของโปรเจกต์ (เช่น `main`) |
| `$CI_PIPELINE_SOURCE` | แหล่งที่มาของ pipeline เช่น `push`, `merge_request_event`, `schedule`, `web` |
| `$CI_MERGE_REQUEST_IID` | หมายเลข Merge Request (มีค่าเฉพาะเมื่อ pipeline รันจาก merge request event) |
| `$GITLAB_USER_LOGIN` | username ของคนที่ทำให้เกิด pipeline นี้ |
| `$CI_SERVER_URL` | URL ของ GitLab instance ที่ใช้งานอยู่ |

### 498.3 ตัวอย่างการใช้ Predefined Variables จริง

```yaml
stages:
  - test

info-job:
  stage: test
  script:
    - echo "กำลังรันบน branch $CI_COMMIT_BRANCH"
    - echo "Commit SHA (แบบย่อ) $CI_COMMIT_SHORT_SHA"
    - echo "Pipeline นี้คือหมายเลข $CI_PIPELINE_IID ของโปรเจกต์ $CI_PROJECT_NAME"
    - echo "รันโดยผู้ใช้ $GITLAB_USER_LOGIN"
    - echo "แหล่งที่มาของ pipeline: $CI_PIPELINE_SOURCE"
```

ผลลัพธ์ที่เห็นใน Job Log อาจออกมาประมาณนี้ (ค่าจริงจะเปลี่ยนไปตาม context):

```
กำลังรันบน branch main
Commit SHA (แบบย่อ) 3f9a1b2
Pipeline นี้คือหมายเลข 42 ของโปรเจกต์ my-project
รันโดยผู้ใช้ phutjirakul
แหล่งที่มาของ pipeline: push
```

### 498.4 ตัวแปรที่เป็นความลับ (Protected/Masked Variables)

สำหรับข้อมูลอ่อนไหว เช่น API key, password, token **ห้ามเขียนค่าจริงลงไปใน `.gitlab-ci.yml` โดยตรงเด็ดขาด** เพราะไฟล์นี้อยู่ใน repository และทุกคนที่เข้าถึง repo จะเห็นค่านั้นได้ทันที

วิธีที่ถูกต้องคือไปตั้งค่าที่เมนู **Settings > CI/CD > Variables** ในโปรเจกต์ แล้วเพิ่มตัวแปรผ่าน UI ของ GitLab โดยสามารถเลือก:

- **Protect variable** — ตัวแปรนี้จะถูกส่งให้เฉพาะ pipeline ที่รันบน protected branch/tag เท่านั้น
- **Mask variable** — ค่าของตัวแปรนี้จะถูกซ่อน (แสดงเป็น `[MASKED]`) ใน Job Log โดยอัตโนมัติ ป้องกันไม่ให้หลุดออกมาโดยไม่ตั้งใจ

เมื่อประกาศไว้ในหน้า Settings แล้ว สามารถเรียกใช้ใน `.gitlab-ci.yml` ด้วยชื่อตัวแปรได้ทันทีเหมือนตัวแปรทั่วไป โดยที่ **ไม่ต้องประกาศ `variables:` ซ้ำในไฟล์เลย** — หัวข้อนี้เป็นแค่ความรู้เบื้องต้น รายละเอียดเชิงลึกเรื่อง Protected Variables, Variable Scoping ตาม environment และการจัดการ secret อย่างปลอดภัยจะอยู่ใน Part หลัง ๆ ของเฟส 7-8

### 498.5 ลำดับความสำคัญของตัวแปร (Precedence) แบบย่อ

ถ้ามีตัวแปรชื่อเดียวกันถูกกำหนดไว้หลายที่ GitLab จะใช้ค่าตามลำดับความสำคัญนี้ (จากสูงไปต่ำ โดยค่าที่สำคัญกว่าจะ override ค่าที่สำคัญน้อยกว่า):

1. ตัวแปรที่ trigger เอง (manual pipeline run) กำหนดตอนกด "Run pipeline"
2. ตัวแปรที่ตั้งไว้ใน `Settings > CI/CD > Variables` (project-level)
3. ตัวแปรที่ประกาศใน `variables:` ระดับ job ใน `.gitlab-ci.yml`
4. ตัวแปรที่ประกาศใน `variables:` ระดับ global ใน `.gitlab-ci.yml`
5. Predefined Variables ที่ GitLab สร้างให้อัตโนมัติ

ยังไม่ต้องท่องจำลำดับนี้ให้แม่นในตอนนี้ แค่รู้ว่า **ตัวแปรที่ตั้งเฉพาะเจาะจงกว่าจะชนะตัวแปรที่กว้างกว่าเสมอ** ก็เพียงพอสำหรับ Part นี้

---

## Step 499: Pipeline badge (สถานะ passing/failing) ติดใน README.md

**Pipeline Badge** คือรูปภาพเล็ก ๆ (SVG) ที่แสดงสถานะล่าสุดของ pipeline บน branch ที่กำหนด เช่น `passing` (สีเขียว) หรือ `failing` (สีแดง) นิยมติดไว้บนสุดของไฟล์ `README.md` เพื่อให้ทุกคนที่เข้ามาดู repository เห็นสถานะสุขภาพของโปรเจกต์ได้ทันทีโดยไม่ต้องเข้าไปดูหน้า Pipelines เอง

### 499.1 หา URL ของ badge จาก GitLab

วิธีที่ง่ายและแม่นยำที่สุดคือให้ GitLab สร้างลิงก์ badge ให้เอง:

1. ไปที่โปรเจกต์บน GitLab
2. ไปที่เมนู **Settings > CI/CD**
3. เลื่อนหาส่วน **General pipelines** (หรือหัวข้อที่เกี่ยวกับ Pipeline badges)
4. ในหน้านี้จะมีช่องให้เลือก branch แล้วแสดง URL ของ badge พร้อมโค้ด Markdown/HTML ให้ copy ไปใช้ได้ทันที

### 499.2 รูปแบบ URL ของ badge

รูปแบบ URL ของ pipeline status badge มีลักษณะดังนี้:

```
https://gitlab.com/<namespace>/<project>/badges/<branch>/pipeline.svg
```

ตัวอย่างเช่น ถ้าโปรเจกต์อยู่ที่ `https://gitlab.com/phutjirakul/my-project` และต้องการ badge ของ branch `main`:

```
https://gitlab.com/phutjirakul/my-project/badges/main/pipeline.svg
```

### 499.3 ฝัง badge ลงใน README.md

การฝัง badge ใน Markdown ทำโดยใช้ syntax รูปภาพ ครอบด้วยลิงก์ที่คลิกแล้วพาไปดูหน้า pipeline จริง:

```markdown
[![pipeline status](https://gitlab.com/phutjirakul/my-project/badges/main/pipeline.svg)](https://gitlab.com/phutjirakul/my-project/-/commits/main)
```

อธิบายโครงสร้าง Markdown นี้:

- `![alt text](url รูปภาพ)` คือ syntax การแสดงรูปภาพใน Markdown
- ครอบด้วย `[...](url ปลายทางเมื่อคลิก)` อีกชั้นหนึ่ง เพื่อให้คลิกที่ badge แล้วพาไปหน้าประวัติ commit/pipeline ของ branch นั้นโดยตรง

ผลลัพธ์เมื่อ render ออกมาบน README จะเห็นเป็นป้ายเล็ก ๆ เขียนว่า "pipeline" คู่กับคำว่า "passed" (สีเขียว) หรือ "failed" (สีแดง) ตามสถานะล่าสุดจริงของ pipeline บน branch นั้น และจะ**อัปเดตอัตโนมัติทุกครั้ง**ที่มี pipeline ใหม่รันเสร็จ โดยไม่ต้องแก้ไข README เองอีกเลย

### 499.4 Badge อื่น ๆ ที่มักติดคู่กัน

นอกจาก pipeline status badge แล้ว ยังมี **Coverage badge** ที่แสดงเปอร์เซ็นต์ code coverage จากผลการรัน test (ต้องตั้งค่า `coverage:` regex ใน job ก่อน ซึ่งเป็นเรื่องที่จะเจาะลึกในเฟส CI/CD ขั้นสูงถัดไป):

```markdown
[![coverage report](https://gitlab.com/phutjirakul/my-project/badges/main/coverage.svg)](https://gitlab.com/phutjirakul/my-project/-/commits/main)
```

### 499.5 ตัวอย่าง README.md ที่มี badge ครบ

```markdown
# My Project

[![pipeline status](https://gitlab.com/phutjirakul/my-project/badges/main/pipeline.svg)](https://gitlab.com/phutjirakul/my-project/-/commits/main)
[![coverage report](https://gitlab.com/phutjirakul/my-project/badges/main/coverage.svg)](https://gitlab.com/phutjirakul/my-project/-/commits/main)

โปรเจกต์ตัวอย่างสำหรับฝึกใช้งาน GitLab CI/CD
```

### 499.6 ทำไม badge ถึงมีประโยชน์มากกว่าที่คิด

1. **สร้างความน่าเชื่อถือ** — badge สีเขียวบอกทันทีว่าโปรเจกต์นี้ "โค้ดล่าสุดใช้งานได้" โดยไม่ต้องอ่านโค้ดเลย
2. **เตือนทีมทันทีเมื่อมีปัญหา** — ถ้า badge เปลี่ยนเป็นสีแดงบน README ของ branch หลัก ทุกคนที่เปิด repo มาเห็นจะรู้ทันทีว่ามีบางอย่างพัง
3. **มาตรฐานที่โปรเจกต์ Open Source เกือบทุกโปรเจกต์ทำ** — การฝึกติด badge เป็นทักษะพื้นฐานที่ควรมีติดตัวเมื่อทำงานร่วมกับโปรเจกต์จริงในอนาคต

---

## Step 500: แบบฝึกหัด — เขียน pipeline แรกที่รัน test จริงให้โปรเจกต์ทดลอง

ถึงเวลารวบยอดทุกอย่างที่เรียนมาใน Part นี้ ด้วยการเขียน pipeline ที่ **รัน test จริง** ไม่ใช่แค่ `echo` จำลองอีกต่อไป

### 500.1 เตรียมโปรเจกต์ทดลอง

สร้างโฟลเดอร์ใหม่และไฟล์สคริปต์ง่าย ๆ ที่จะทดสอบ — ในแบบฝึกหัดนี้เราจะใช้ **Bash script ล้วน ๆ** เพื่อไม่ต้องพึ่งพาการติดตั้งภาษาโปรแกรมมิ่งเพิ่มเติมใน Runner (ทุก Runner ของ GitLab มี `bash`/`sh` พร้อมใช้งานอยู่แล้วเป็นค่าเริ่มต้น)

```bash
mkdir -p ~/git-course/part-50-practice
cd ~/git-course/part-50-practice
git init
```

สร้างไฟล์ `calculator.sh` ที่เป็น "โปรแกรม" ตัวอย่างของเรา:

```bash
#!/bin/bash
# calculator.sh — ฟังก์ชันคำนวณอย่างง่ายสำหรับใช้ทดสอบ

add() {
  echo $(( $1 + $2 ))
}

subtract() {
  echo $(( $1 - $2 ))
}

multiply() {
  echo $(( $1 * $2 ))
}
```

สร้างไฟล์ `test_calculator.sh` ที่เป็นชุดทดสอบของเรา:

```bash
#!/bin/bash
# test_calculator.sh — ชุดทดสอบสำหรับ calculator.sh

set -e   # ถ้าคำสั่งใดล้มเหลว ให้หยุด script ทันที

source ./calculator.sh

fail_count=0

assert_equal() {
  local description="$1"
  local expected="$2"
  local actual="$3"

  if [ "$expected" -eq "$actual" ]; then
    echo "PASS: $description"
  else
    echo "FAIL: $description (คาดหวัง $expected แต่ได้ $actual)"
    fail_count=$((fail_count + 1))
  fi
}

assert_equal "2 + 3 ควรได้ 5" 5 "$(add 2 3)"
assert_equal "10 - 4 ควรได้ 6" 6 "$(subtract 10 4)"
assert_equal "6 * 7 ควรได้ 42" 42 "$(multiply 6 7)"

if [ "$fail_count" -gt 0 ]; then
  echo "มี test ล้มเหลวทั้งหมด $fail_count รายการ"
  exit 1
else
  echo "Test ทั้งหมดผ่านหมด!"
  exit 0
fi
```

ให้สิทธิ์รันไฟล์และทดสอบว่า test ทำงานถูกต้องบนเครื่องตัวเองก่อน:

```bash
chmod +x calculator.sh test_calculator.sh
./test_calculator.sh
```

ควรเห็นผลลัพธ์:

```
PASS: 2 + 3 ควรได้ 5
PASS: 10 - 4 ควรได้ 6
PASS: 6 * 7 ควรได้ 42
Test ทั้งหมดผ่านหมด!
```

### 500.2 เขียน `.gitlab-ci.yml` ที่รัน test จริง

สร้างไฟล์ `.gitlab-ci.yml` ที่ root ของโปรเจกต์นี้:

```yaml
stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "ตรวจสอบว่าไฟล์ที่จำเป็นครบถ้วน"
    - test -f calculator.sh
    - test -f test_calculator.sh
    - echo "ไฟล์ครบถ้วน พร้อมทดสอบ"

test-job:
  stage: test
  script:
    - chmod +x calculator.sh test_calculator.sh
    - ./test_calculator.sh
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

อธิบายจุดสำคัญ:

- `build-job` ทำหน้าที่ตรวจสอบเบื้องต้นว่าไฟล์ที่จำเป็นมีอยู่จริง (จำลองขั้นตอน "build" แบบง่าย ๆ) ด้วยคำสั่ง `test -f` ซึ่งจะคืน exit code ไม่เป็น 0 ถ้าไฟล์ไม่มีอยู่จริง ทำให้ job fail ทันทีถ้ามีคนลืม commit ไฟล์
- `test-job` รันชุดทดสอบจริงด้วย `./test_calculator.sh` — ถ้า script นี้จบด้วย `exit 1` (มี test ล้มเหลว) job นี้จะถูกทำเครื่องหมายว่า **failed** โดยอัตโนมัติ
- ใช้ `rules` แบบที่แนะนำใน Step 497 เพื่อให้ test รันทั้งตอนเปิด Merge Request และตอน push เข้า default branch โดยตรง

### 500.3 Commit, Push แล้วดูผลลัพธ์

```bash
git add calculator.sh test_calculator.sh .gitlab-ci.yml
git commit -m "เพิ่ม calculator.sh พร้อม test และ pipeline สำหรับรัน test อัตโนมัติ"
git remote add origin <URL ของ repo บน GitLab ที่คุณสร้างไว้>
git push -u origin main
```

ไปที่ **CI/CD > Pipelines** เพื่อดูว่า pipeline รันผ่านทั้งสอง stage หรือไม่ ถ้าทำถูกต้องตามขั้นตอนควรเห็นสถานะ **Passed** สีเขียวทั้งคู่

### 500.4 ทดลองทำให้ pipeline "แดง" ด้วยตัวเอง (สำคัญมาก)

ส่วนที่มักถูกมองข้ามแต่สำคัญที่สุดของแบบฝึกหัดนี้คือ **การทดลองทำให้ test fail ด้วยความตั้งใจ** เพื่อให้แน่ใจว่า pipeline ของคุณจับข้อผิดพลาดได้จริง ไม่ใช่แค่ผ่านเพราะบังเอิญ

แก้ไข `calculator.sh` ให้ฟังก์ชัน `add` ทำงานผิดโดยตั้งใจ:

```bash
add() {
  echo $(( $1 + $2 + 1 ))   # จงใจใส่บั๊ก: บวกเกินไป 1
}
```

Commit และ push อีกครั้ง:

```bash
git add calculator.sh
git commit -m "ทดลอง: จงใจใส่บั๊กเพื่อทดสอบว่า pipeline จับได้จริง"
git push
```

ครั้งนี้ pipeline ควรแสดงสถานะ **Failed** สีแดงที่ `test-job` เข้าไปดู Job Log จะเห็นบรรทัด `FAIL: 2 + 3 ควรได้ 5 (คาดหวัง 5 แต่ได้ 6)` ชัดเจน — นี่คือหลักฐานว่า pipeline ของคุณทำงานได้จริงตามที่ออกแบบไว้

หลังจากยืนยันว่า pipeline จับบั๊กได้จริงแล้ว ให้แก้ `calculator.sh` กลับให้ถูกต้อง แล้ว commit push อีกครั้งเพื่อให้ pipeline กลับมาเป็นสีเขียวตามเดิม

### 500.5 ติด Pipeline Badge ปิดท้าย

ทำตาม Step 499 เพื่อดึง badge URL ของ branch `main` มาใส่ในไฟล์ `README.md` ของโปรเจกต์ทดลองนี้ แล้ว commit push อีกครั้งหนึ่ง เพื่อให้ครบทุกองค์ประกอบที่เรียนมาทั้ง Part

### 500.6 Checklist สรุปแบบฝึกหัด

- [ ] มีไฟล์ `calculator.sh` และ `test_calculator.sh` ที่รันผ่านได้บนเครื่อง local
- [ ] มีไฟล์ `.gitlab-ci.yml` ที่มี 2 stage คือ `build` และ `test`
- [ ] `test-job` เรียก `./test_calculator.sh` จริง ไม่ใช่แค่ `echo`
- [ ] เคยเห็น pipeline สถานะ **Passed** สีเขียวมาแล้วอย่างน้อย 1 ครั้ง
- [ ] เคยจงใจทำให้ test fail แล้วเห็นสถานะ **Failed** สีแดงพร้อมอ่าน Job Log เจอสาเหตุจริง
- [ ] ติด pipeline badge ไว้ใน `README.md` เรียบร้อยแล้ว

---

## สรุป Part 50

ใน Part นี้เราได้เรียนรู้ว่า:

1. **CI/CD** คือการให้ระบบทำงานซ้ำ ๆ อย่างการ build, test, deploy แทนมนุษย์โดยอัตโนมัติ และ GitLab CI/CD ทำงานผ่าน 3 องค์ประกอบหลัก คือไฟล์ `.gitlab-ci.yml`, pipeline, และ GitLab Runner
2. ไฟล์ `.gitlab-ci.yml` ต้องอยู่ที่ **root ของ repository** เท่านั้น เขียนด้วยภาษา YAML ที่ต้อง indent ด้วย space เท่านั้น ห้ามใช้ tab
3. **Stage** คือกลุ่มของ **Job** ที่รันเรียงลำดับกับ stage อื่น ในขณะที่ job ในสาย stage เดียวกันจะรันพร้อมกัน และ stage ถัดไปจะเริ่มก็ต่อเมื่อ stage ก่อนหน้าผ่านหมดแล้วเท่านั้น
4. pipeline ที่ง่ายที่สุดต้องมีอย่างน้อย `stages:` และ job ที่มี `stage:` กับ `script:`
5. `before_script` รันก่อนงานหลักเสมอ, `script` คืองานหลัก, `after_script` รันหลังเสมอไม่ว่าผลจะเป็นอย่างไร และสามารถตั้งค่าเริ่มต้นให้ทุก job ได้ด้วย `default:`
6. อ่านผลลัพธ์ pipeline และ job log ได้จากเมนู **CI/CD > Pipelines** บน GitLab UI รวมถึงใช้ปุ่ม Retry/Cancel ได้เมื่อจำเป็น
7. `rules` เป็นวิธีแนะนำในการควบคุมเงื่อนไขการรัน job แทนที่ `only`/`except` แบบเก่า เพราะยืดหยุ่นและอ่านง่ายกว่ามาก
8. **CI/CD Variables** มีทั้งแบบที่เรากำหนดเองผ่าน `variables:` และแบบ **Predefined Variables** ที่ GitLab เตรียมไว้ให้อัตโนมัติ เช่น `$CI_COMMIT_BRANCH`, `$CI_COMMIT_SHA` และข้อมูลลับต้องเก็บผ่าน **Settings > CI/CD > Variables** เท่านั้น ห้ามเขียนลงไฟล์ตรง ๆ
9. **Pipeline badge** ช่วยแสดงสถานะสุขภาพของโปรเจกต์บน README.md แบบอัปเดตอัตโนมัติ
10. ลงมือเขียน pipeline ที่รัน test จริงกับโปรเจกต์ทดลองของตัวเอง ทั้งกรณีที่ผ่านและกรณีที่จงใจทำให้ fail เพื่อยืนยันว่า pipeline ทำงานได้จริงตามที่ออกแบบไว้

Part นี้เป็นเพียงจุดเริ่มต้นของโลก CI/CD เท่านั้น — แนวคิดเรื่อง `image`, `services`, `cache`, `artifacts`, Environments, และการ deploy ขึ้นจริงจะถูกอธิบายอย่างละเอียดใน Part 71-72 แต่ก่อนจะไปถึงจุดนั้น เราต้องเข้าใจก่อนว่า pipeline ถูกรันอยู่ "ที่ไหน" จริง ๆ นั่นคือหน้าที่ของ **GitLab Runner** ซึ่งเป็นหัวข้อของ Part ถัดไป

**ต่อไป:** [Part 51: GitLab Runner: การตั้งค่าและใช้งาน](./part-051-gitlab-runner.md)
