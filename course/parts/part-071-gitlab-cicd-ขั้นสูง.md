# Part 71: GitLab CI/CD ขั้นสูง: Stages, Pipelines, Cache

> **Step ในหลักสูตรนี้:** Step 701–710
> **เฟส:** 7 — CI/CD เต็มรูปแบบด้วย GitHub Actions และ GitLab CI/CD
> **เป้าหมายของ Part นี้:** ต่อยอดจากพื้นฐาน GitLab CI/CD ใน Part 50 ไปสู่ระดับที่ใช้งานได้จริงในทีมงานขนาดใหญ่ (production-ready) เข้าใจลำดับการรันของ stage และ job แบบละเอียด รู้จัก `needs:` เพื่อสร้าง Directed Acyclic Graph (DAG) pipeline ที่รันเร็วขึ้น เจาะลึก cache และ artifacts เพื่อลดเวลา pipeline และเก็บผลลัพธ์การทดสอบอย่างมืออาชีพ ลดโค้ดซ้ำซ้อนด้วย `extends` แยกไฟล์ config ด้วย `include` ผูก job การ deploy เข้ากับ environment และควบคุมการรัน job แบบ manual/delayed ด้วย `when` ทุกรูปแบบ ปิดท้ายด้วยการลงมือเขียน pipeline เต็มรูปแบบ 4 stage ที่ใช้ทุกเทคนิคที่เรียนมารวมกัน

---

## สารบัญของ Part นี้

- Step 701: ทบทวน `.gitlab-ci.yml` เบื้องต้นจาก Part 50 และก้าวต่อไปสู่ระดับ Production-ready
- Step 702: Stages เจาะลึก — ลำดับการรันและการรันขนานภายใน stage เดียวกัน
- Step 703: `needs:` — สร้าง Directed Acyclic Graph (DAG) Pipeline ให้ทำงานเร็วขึ้น
- Step 704: Cache เจาะลึก (`key`, `paths`, `policy`)
- Step 705: Artifacts เจาะลึก (`paths`, `expire_in`, `reports`)
- Step 706: `extends` — ลดโค้ดซ้ำซ้อนด้วย Template
- Step 707: `include` — แยกไฟล์ `.gitlab-ci.yml` เป็นหลายไฟล์ย่อย
- Step 708: Environments — ผูก Deploy Job เข้ากับ Environment
- Step 709: Manual Jobs และ `when:` ทุกค่าที่มี
- Step 710: แบบฝึกหัด — เขียน Pipeline เต็มรูปแบบ 4 Stage พร้อม Cache, Artifact และ DAG

---

## Step 701: ทบทวน `.gitlab-ci.yml` เบื้องต้นจาก Part 50 และก้าวต่อไปสู่ระดับ Production-ready

### 701.1 สิ่งที่เรารู้แล้วจาก Part 50

ใน **Part 50: GitLab CI/CD เบื้องต้น** เราได้เรียนรู้รากฐานที่สำคัญมากไปแล้ว สรุปสั้น ๆ ดังนี้:

| หัวข้อที่เรียนไปแล้ว | สรุปสั้น ๆ |
|---|---|
| ไฟล์ `.gitlab-ci.yml` | ต้องอยู่ที่ root ของ repo เขียนด้วย YAML |
| `stages:` และ `stage:` | ประกาศลำดับขั้นตอน แล้วผูก job แต่ละตัวเข้ากับ stage |
| `script`, `before_script`, `after_script` | คำสั่งหลัก คำสั่งเตรียมการ และคำสั่ง cleanup |
| `default:` | ตั้งค่าเริ่มต้นให้ทุก job ในไฟล์ |
| `rules` และ `only`/`except` | ควบคุมเงื่อนไขว่า job จะรันเมื่อไร |
| CI/CD Variables | ตัวแปรที่กำหนดเอง และ Predefined Variables ของ GitLab |
| Pipeline badge | แสดงสถานะ passing/failing บน README |

ถ้าคุณยังไม่มั่นใจในหัวข้อเหล่านี้ แนะนำให้กลับไปทบทวน Part 50 ก่อน เพราะ Part นี้จะ **ไม่อธิบายพื้นฐานเหล่านั้นซ้ำอีก** แต่จะสมมติว่าคุณเขียน pipeline ง่าย ๆ ที่มี 2-3 stage และรัน test จริงได้แล้ว

### 701.2 ทำไมความรู้ระดับ Part 50 ยังไม่พอสำหรับงานจริง

pipeline แบบที่เราเขียนใน Part 50 มีลักษณะแบบนี้:

```yaml
stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "build"

test-job:
  stage: test
  script:
    - echo "test"
```

pipeline แบบนี้ **ใช้งานได้** แต่มีข้อจำกัดร้ายแรงหลายข้อเมื่อนำไปใช้กับโปรเจกต์จริงในทีมงาน:

1. **ไม่มี cache** — ทุกครั้งที่รัน pipeline ต้อง `npm install` หรือติดตั้ง dependency ใหม่ทั้งหมดตั้งแต่ศูนย์ ทำให้ pipeline ช้ามาก โดยเฉพาะโปรเจกต์ขนาดใหญ่ที่ dependency มีเป็นร้อย MB
2. **ไม่มี artifacts อย่างเป็นระบบ** — ไฟล์ที่ build เสร็จใน stage `build` (เช่นโฟลเดอร์ `dist/`) จะหายไปทันทีเมื่อ job จบ ทำให้ stage `deploy` ไม่มีไฟล์ให้ deploy ต่อ
3. **stage รันเรียงกันเสมอแม้ไม่จำเป็น** — ถ้า job ใน stage `test` ไม่ได้พึ่งพา job ทุกตัวใน stage `build` จริง ๆ การรอให้ทั้ง stage `build` เสร็จก่อนอาจเสียเวลาโดยใช่เหตุ
4. **ไม่มี environment ผูกกับ deploy job** — ทำให้ GitLab ไม่รู้ว่า job ไหน deploy ไปที่ไหน ไม่มีหน้า "Environments" ให้ดูประวัติการ deploy หรือกด rollback
5. **โค้ดซ้ำซ้อนเมื่อมี job เยอะขึ้น** — ถ้ามี 10 job ที่ต้องใช้ `image` และ `before_script` เดียวกันหมด การ copy-paste ซ้ำ ๆ ทำให้ไฟล์ยาวและแก้ยาก
6. **ไฟล์เดียวยาวเกินไปเมื่อโปรเจกต์ใหญ่ขึ้น** — โปรเจกต์จริงบางแห่งมี `.gitlab-ci.yml` ยาวหลายพันบรรทัด ถ้าไม่แยกไฟล์จะดูแลรักษายากมาก

### 701.3 สิ่งที่ Part นี้จะเติมเต็ม

Part 71 นี้จะพา pipeline ของคุณจากระดับ "ใช้งานได้" ไปสู่ระดับ **"production-ready"** โดยครอบคลุมเทคนิคที่ทีมวิศวกรมืออาชีพใช้กันจริงทุกวัน:

```
พื้นฐาน (Part 50)              ขั้นสูง (Part 71)
┌─────────────────┐          ┌──────────────────────────────┐
│ stages/jobs      │   ──▶    │ needs: (DAG) — รันเร็วขึ้น       │
│ script ธรรมดา     │   ──▶    │ cache — ไม่ install ซ้ำทุกครั้ง  │
│ ไม่มี artifact     │   ──▶    │ artifacts + reports (JUnit)   │
│ config ซ้ำ ๆ       │   ──▶    │ extends — ใช้ template        │
│ ไฟล์เดียวยาว        │   ──▶    │ include — แยกไฟล์ย่อย          │
│ ไม่มี environment  │   ──▶    │ environment: ผูก deploy job    │
│ when พื้นฐาน       │   ──▶    │ when ครบทุกค่า + manual jobs   │
└─────────────────┘          └──────────────────────────────┘
```

> **หมายเหตุ:** Part นี้ยังไม่ลงลึกเรื่อง Docker-in-Docker, `services:`, Protected Variables แบบเจาะลึก หรือ Security Scanning — หัวข้อเหล่านั้นจะถูกอธิบายในภายหลังของเฟส 7-8 ตามที่เกริ่นไว้ใน Part 50 ส่วน Part 72 ที่ตามมาจะพาไปดู Multi-project Pipelines และ Dynamic Pipelines ซึ่งเป็นขั้นที่สูงกว่านี้อีกขั้นหนึ่ง

### 701.4 สิ่งที่ต้องมีก่อนเริ่ม Part นี้

- ผ่าน Part 50 มาแล้ว เขียน pipeline พื้นฐานที่มี stage และ job ได้
- มีโปรเจกต์ทดลองบน GitLab ที่ push ได้ (ใช้โปรเจกต์เดิมจาก Part 50 ต่อได้เลย หรือสร้างใหม่ก็ได้)
- มี Shared Runner ใช้งานได้ (ถ้าใช้ GitLab.com จะมีให้ใช้ในโควตาฟรีอยู่แล้ว)

ต่อไปเรามาเริ่มเจาะลึกกันทีละหัวข้อ

---

## Step 702: Stages เจาะลึก — ลำดับการรันและการรันขนานภายใน stage เดียวกัน

### 702.1 ทบทวนกฎพื้นฐาน แล้วขยายความให้ลึกขึ้น

จาก Part 50 เรารู้กฎพื้นฐานแล้วว่า:

> Job ในสาย stage เดียวกันรันพร้อมกัน (parallel) ส่วน stage คนละตัวรันเรียงกัน (sequential)

แต่มีรายละเอียดสำคัญที่ Part 50 ยังไม่ได้พูดถึง ซึ่งจำเป็นมากเมื่อ pipeline เริ่มมี job จำนวนมาก

### 702.2 การรันขนานถูกจำกัดด้วยจำนวน Runner ที่ว่างจริง

คำว่า "รันพร้อมกัน" ไม่ได้แปลว่า GitLab จะรันทุก job ในเวลาเดียวกันเป๊ะ ๆ เสมอไป — มันหมายถึง **ไม่มีการบังคับให้รันเรียงกันทีละตัว** เท่านั้น ในทางปฏิบัติ ความเร็วจริงขึ้นอยู่กับจำนวน Runner ที่ว่างพอจะรับงานพร้อมกัน

```yaml
stages:
  - test

unit-test:
  stage: test
  script:
    - echo "unit test"

integration-test:
  stage: test
  script:
    - echo "integration test"

lint-check:
  stage: test
  script:
    - echo "lint"

security-scan:
  stage: test
  script:
    - echo "security scan"
```

ถ้ามี Runner ว่างพร้อมกัน 4 ตัว ทั้ง 4 job นี้จะเริ่มรันพร้อมกันทันที แต่ถ้ามี Runner ว่างแค่ 2 ตัว GitLab จะจ่ายงานให้ 2 job ก่อน แล้วรอ Runner ว่างเพื่อรับอีก 2 job ที่เหลือ — **การเขียน pipeline ให้รันขนานได้มากเป็นการเปิดโอกาสให้เร็วขึ้น ไม่ใช่การรับประกันว่าจะรันพร้อมกันเป๊ะเสมอ**

### 702.3 จำนวน Job สูงสุดที่รันพร้อมกันได้ (Concurrency)

จำนวน job ที่รันพร้อมกันได้จริงในเวลาเดียวกันขึ้นอยู่กับหลายปัจจัย:

| ปัจจัย | ผลกระทบ |
|---|---|
| จำนวน Runner ที่ลงทะเบียนไว้กับโปรเจกต์/กลุ่ม | ยิ่งมี Runner เยอะ ยิ่งรันขนานได้มาก |
| ค่า `concurrent` ที่ตั้งไว้ในแต่ละ Runner (`config.toml`) | จำกัดว่า Runner หนึ่งตัวรับกี่ job พร้อมกันได้ |
| โควตา CI/CD minutes ของแผนที่ใช้อยู่ (สำหรับ GitLab.com) | อาจจำกัดจำนวน job ที่รันพร้อมกันในบางแผน |
| `resource_group:` ที่ตั้งไว้ใน job (ดูหัวข้อถัดไป) | บังคับให้ job บางกลุ่มรันทีละตัวเท่านั้น แม้จะมี Runner ว่างพอ |

### 702.4 `resource_group:` — บังคับให้ job บางตัวรันทีละตัวเท่านั้น

บางสถานการณ์เราต้องการ**ห้าม**ไม่ให้ job สองตัวรันพร้อมกัน แม้จะอยู่คนละ pipeline ก็ตาม — ตัวอย่างคลาสสิกคือ job deploy ที่ยิงไปยัง server ตัวเดียวกัน ถ้ามีสอง pipeline พยายาม deploy พร้อมกันอาจทำให้ deployment เสียหาย

```yaml
deploy-production:
  stage: deploy
  script:
    - ./deploy.sh production
  resource_group: production
```

Job ใดก็ตามที่ประกาศ `resource_group: production` เหมือนกัน จะถูกบังคับให้ **รันทีละตัวเท่านั้น** ต่อให้มาจากคนละ pipeline หรือคนละ branch ก็ตาม โดยงานที่มาทีหลังจะรอในคิวจนกว่างานก่อนหน้าจะจบ

### 702.5 ลำดับการประกาศ `stages:` มีผลจริง

ลำดับของรายการใน `stages:` **คือลำดับการรันจริง** ไม่ใช่แค่การตั้งชื่อ ถ้าคุณสลับลำดับ ผลลัพธ์การรันจะเปลี่ยนตามทันที

```yaml
stages:
  - test      # จะรันก่อน build!
  - build
```

ตัวอย่างนี้จงใจสลับผิด — `test` จะถูกรันก่อน `build` ทั้งที่ตามธรรมเนียมทั่วไปควร build ก่อนแล้วค่อย test นี่คือข้อผิดพลาดเชิง config ที่พบได้บ่อยเมื่อ copy-paste job จากที่อื่นมาแล้วลืมเช็คลำดับ `stages:` ให้ตรงกัน

### 702.6 ถ้า Job ไม่ระบุ `stage:` เลย

ถ้า job ใดไม่ประกาศ `stage:` เอาไว้ GitLab จะให้ job นั้นอยู่ใน stage ชื่อ `test` โดยอัตโนมัติ (เป็นค่า default ของระบบ) แต่ **ไม่แนะนำให้พึ่งพาพฤติกรรมนี้** ควรระบุ `stage:` ให้ครบทุก job เสมอเพื่อความชัดเจนและป้องกันความสับสนในทีม

### 702.7 Stage ที่ไม่มี Job ใดใช้เลยจะถูกข้าม

ถ้าประกาศ `stages:` ไว้ 5 stage แต่มี stage หนึ่งที่ไม่มี job ใดผูกอยู่เลย (หรือ job ทั้งหมดใน stage นั้นถูก skip ด้วย `rules`) GitLab จะ **ข้าม stage นั้นไปเงียบ ๆ** โดยไม่ทำให้ pipeline fail แต่อย่างใด — pipeline จะเดินหน้าไปยัง stage ถัดไปที่มี job จริงทันที

### 702.8 ตัวอย่าง Pipeline ที่มีหลาย Job ในหลาย Stage แบบสมจริง

```yaml
stages:
  - build
  - test
  - package
  - deploy

build-frontend:
  stage: build
  script:
    - echo "build frontend assets"

build-backend:
  stage: build
  script:
    - echo "build backend binary"

unit-test:
  stage: test
  script:
    - echo "unit test"

integration-test:
  stage: test
  script:
    - echo "integration test"

lint:
  stage: test
  script:
    - echo "lint code"

package-app:
  stage: package
  script:
    - echo "รวมไฟล์ทั้งหมดเป็น artifact เดียว"

deploy-staging:
  stage: deploy
  script:
    - echo "deploy staging"
```

แผนภาพลำดับการรัน:

```
build (parallel)          test (parallel)              package         deploy
┌────────────────┐       ┌───────────────────┐        ┌────────┐      ┌────────────────┐
│ build-frontend  │       │ unit-test          │        │package │      │ deploy-staging  │
│ build-backend   │──────▶│ integration-test   │───────▶│  -app  │─────▶│                 │
│                 │       │ lint               │        │        │      │                 │
└────────────────┘       └───────────────────┘        └────────┘      └────────────────┘
   ต้องผ่านทั้งคู่ก่อน          ต้องผ่านทั้ง 3 job ก่อน
```

ปัญหาของ pipeline แบบนี้คือ **`unit-test` ต้องรอ `build-backend` เสร็จ ทั้งที่จริง ๆ แล้ว `unit-test` อาจไม่ได้พึ่งพา `build-frontend` เลย** — นี่คือจุดที่ `needs:` เข้ามาช่วยแก้ปัญหาใน Step ถัดไป

---

## Step 703: `needs:` — สร้าง Directed Acyclic Graph (DAG) Pipeline ให้ทำงานเร็วขึ้น

### 703.1 ปัญหาของ Pipeline แบบ Stage-based ล้วน ๆ

ใน pipeline แบบปกติ (stage-based) กฎคือ **stage ถัดไปจะเริ่มก็ต่อเมื่อ job ทุกตัวใน stage ก่อนหน้าทั้งหมดเสร็จแล้วเท่านั้น** แม้ job ของคุณจะไม่ได้ต้องพึ่งพาผลลัพธ์ของ job อื่นใน stage เดียวกันเลยก็ตาม

ตัวอย่างเช่น ถ้า `build-frontend` ใช้เวลา 30 วินาที แต่ `build-backend` ใช้เวลา 5 นาที และ `unit-test` (stage `test`) ต้องการแค่ผลลัพธ์จาก `build-frontend` เท่านั้น — `unit-test` ก็ยัง **ต้องรอ `build-backend` จบก่อนถึง 5 นาทีอยู่ดี** ทั้งที่ไม่จำเป็นเลย

### 703.2 `needs:` คืออะไร

`needs:` คือ keyword ที่ให้คุณระบุว่า **job นี้ต้องรอผลลัพธ์ของ job ไหนบ้าง** โดยเฉพาะเจาะจง แทนที่จะรอทั้ง stage ให้จบ เมื่อใช้ `needs:` GitLab จะสร้าง pipeline แบบ **DAG (Directed Acyclic Graph)** ที่ job สามารถเริ่มทำงานได้ทันทีที่ job ที่ตัวเอง `needs` เสร็จสิ้น **โดยไม่ต้องสนใจว่า job อื่นใน stage เดียวกัน (ที่ตัวเองไม่ได้ needs) จะเสร็จหรือยัง**

```yaml
stages:
  - build
  - test
  - package
  - deploy

build-frontend:
  stage: build
  script:
    - echo "build frontend (เร็ว)"

build-backend:
  stage: build
  script:
    - echo "build backend (ช้ามาก ใช้เวลา 5 นาที)"

unit-test-frontend:
  stage: test
  needs: ["build-frontend"]
  script:
    - echo "unit test frontend — เริ่มได้ทันทีที่ build-frontend เสร็จ ไม่ต้องรอ build-backend"

unit-test-backend:
  stage: test
  needs: ["build-backend"]
  script:
    - echo "unit test backend"
```

ด้วย `needs: ["build-frontend"]` job `unit-test-frontend` จะ **เริ่มทำงานทันทีที่ `build-frontend` เสร็จ** โดยไม่สนใจว่า `build-backend` จะเสร็จหรือไม่ — ทั้งที่ทั้งคู่ยังอยู่ใน stage `test` เหมือนเดิม

### 703.3 เปรียบเทียบภาพ Pipeline แบบมี/ไม่มี `needs`

**แบบไม่มี `needs` (stage-based ปกติ):**

```
build ────────────────▶ test ────────────────▶ package
(รอทุก job ใน build จบ)   (รอทุก job ใน test จบ)
```

**แบบมี `needs` (DAG):**

```
build-frontend ──▶ unit-test-frontend ──┐
                                          ├──▶ package-app
build-backend  ──▶ unit-test-backend  ──┘
     (ใช้เวลานาน)
```

จะเห็นว่า `unit-test-frontend` ไม่ต้องรอ `build-backend` เลย ทำให้ **เวลารวมของ pipeline สั้นลงอย่างมีนัยสำคัญ** โดยเฉพาะเมื่อ pipeline มี job จำนวนมากที่ทำงานอิสระต่อกัน

### 703.4 `needs: []` — ให้ Job เริ่มทำงานทันทีตั้งแต่ต้น Pipeline

ถ้าใส่ `needs: []` (list ว่าง) หมายความว่า job นี้ **ไม่ต้องรอ job ใดเลย** สามารถเริ่มทำงานได้ทันทีที่ pipeline เริ่ม ไม่ว่าจะประกาศ `stage:` เป็นอะไรก็ตาม

```yaml
stages:
  - build
  - test

lint:
  stage: test
  needs: []
  script:
    - echo "lint ไม่ต้องพึ่งพา build เลย เริ่มพร้อม build ได้ทันที"

build-app:
  stage: build
  script:
    - echo "build"
```

แม้ `lint` จะประกาศ `stage: test` (ซึ่งตามปกติควรรันหลัง `build`) แต่เพราะมี `needs: []` มันจะเริ่มทำงาน**พร้อมกับ** `build-app` ทันที ไม่ต้องรอ stage `build` จบก่อน

### 703.5 `needs:` แบบระบุรายละเอียด (ดาวน์โหลด Artifact หรือไม่)

โดย default เมื่อ job หนึ่ง `needs` อีก job หนึ่ง มันจะ **ดาวน์โหลด artifacts ของ job ที่ needs มาด้วยโดยอัตโนมัติ** ถ้าไม่ต้องการพฤติกรรมนี้ (เช่น job ที่ needs แค่ต้องการรอให้เสร็จก่อน แต่ไม่ได้ใช้ไฟล์ใด ๆ จากมัน) สามารถปิดได้ด้วย `artifacts: false`:

```yaml
notify-slack:
  stage: test
  needs:
    - job: build-backend
      artifacts: false
  script:
    - echo "แจ้งเตือนว่า build เสร็จแล้ว (ไม่ต้องการไฟล์ใด ๆ จาก build-backend)"
```

### 703.6 `needs:` แบบ `optional: true` — รอถ้ามี job นั้นจริง

ถ้า job ที่ระบุใน `needs` อาจถูก `rules` ข้ามไปในบางกรณี (ไม่ได้ถูกสร้างขึ้นในทุก pipeline) การใช้ `needs:` แบบปกติจะทำให้เกิด error ทันทีเพราะหา job ที่ needs ไม่เจอ แก้ได้ด้วย `optional: true`:

```yaml
deploy-job:
  stage: deploy
  needs:
    - job: security-scan
      optional: true
  script:
    - echo "deploy ต่อได้แม้ security-scan จะไม่ถูกสร้างขึ้นในบาง pipeline"
```

### 703.7 ข้อจำกัดสำคัญของ `needs:`

1. **Job ที่ระบุใน `needs:` ต้องอยู่ใน stage ก่อนหน้าเท่านั้น** (หรือ stage เดียวกันในบางกรณีที่รองรับ) — จะ `needs` job ที่อยูใน stage ถัดไปไม่ได้ เพราะจะเกิดการวนลูปที่ไม่มีจุดจบ (Cycle) ซึ่งขัดกับหลักการของ DAG (Directed **Acyclic** Graph — ต้องไม่มีวงจรวน)
2. **จำนวน job ที่ `needs:` ได้มีเพดานจำกัด** (ค่ามาตรฐานคือไม่เกิน 50 job ต่อการ needs หนึ่งครั้ง) — เพียงพอสำหรับ pipeline ทั่วไปเกือบทั้งหมด
3. ถ้า job ที่ถูก `needs` ล้มเหลว job ที่ needs มันจะ**ไม่ถูกรันเลย** (ค่า default) เพราะถือว่าเงื่อนไขเบื้องต้นไม่สำเร็จ

### 703.8 ตารางสรุป `needs:` เทียบกับ Stage แบบปกติ

| ประเด็น | Stage-based ปกติ | `needs:` (DAG) |
|---|---|---|
| ต้องรอ job อะไรบ้างก่อนเริ่ม | ทุก job ใน stage ก่อนหน้า | เฉพาะ job ที่ระบุใน `needs:` เท่านั้น |
| ความเร็วรวมของ pipeline | ช้ากว่าเมื่อมี job ที่ไม่พึ่งพากันแต่ต้องรอ stage เดียวกัน | เร็วกว่า เพราะ job ที่พร้อมจะเริ่มได้ทันที |
| ความซับซ้อนในการอ่าน | อ่านง่าย เข้าใจง่ายสำหรับมือใหม่ | ต้องไล่ดูความสัมพันธ์ของแต่ละ job |
| เหมาะกับ | pipeline ขนาดเล็ก-กลาง ที่ job พึ่งพากันตามลำดับ stage อยู่แล้ว | pipeline ขนาดใหญ่ ที่มี job จำนวนมากและพึ่งพากันไม่ตรงกับลำดับ stage |

> **ข้อแนะนำในทางปฏิบัติ:** ไม่จำเป็นต้องใช้ `needs:` กับทุก job ในทุก pipeline — ใช้เฉพาะจุดที่รู้ชัดว่า job บางตัวเสียเวลารอโดยไม่จำเป็นเท่านั้น การใช้ `needs:` มากเกินไปโดยไม่จำเป็นจะทำให้ไฟล์อ่านยากขึ้นโดยไม่ได้ประโยชน์ด้านความเร็วที่คุ้มค่า

---

## Step 704: Cache เจาะลึก (`key`, `paths`, `policy`)

### 704.1 ปัญหาที่ Cache แก้ให้

ทุกครั้งที่ job หนึ่งเริ่มทำงาน มันจะเริ่มต้นด้วย **สภาพแวดล้อมที่ว่างเปล่า** ไม่มี dependency ใด ๆ ติดตั้งไว้เลย (เว้นแต่จะมากับ Docker image ที่ใช้) ทำให้ต้องรันคำสั่งอย่าง `npm install`, `pip install`, `bundle install` ใหม่ทุกครั้ง — ซึ่งอาจใช้เวลาหลายนาทีถ้า dependency มีจำนวนมาก และเป็นการเสียเวลาซ้ำซ้อนโดยใช่เหตุ เพราะ dependency ส่วนใหญ่ไม่ได้เปลี่ยนแปลงบ่อย

**Cache** คือกลไกที่ให้ GitLab **เก็บไฟล์หรือโฟลเดอร์ที่ระบุไว้จาก job หนึ่ง แล้วนำกลับมาใช้ในการรัน pipeline ครั้งถัดไป** เพื่อไม่ต้องสร้างใหม่ตั้งแต่ศูนย์ทุกครั้ง

### 704.2 โครงสร้างพื้นฐานของ `cache:`

```yaml
test-job:
  stage: test
  cache:
    key: my-cache-key
    paths:
      - node_modules/
  script:
    - npm install
    - npm test
```

- `key:` — ชื่อที่ใช้ระบุตัวตนของ cache ก้อนนี้ (job ที่ใช้ `key` เดียวกันจะแชร์ cache ก้อนเดียวกัน)
- `paths:` — รายการโฟลเดอร์/ไฟล์ที่ต้องการให้เก็บไว้เป็น cache หลังจาก job รันเสร็จ

### 704.3 `key:` แบบต่าง ๆ ที่ใช้บ่อยในงานจริง

**แบบที่ 1: key คงที่ (Static Key) — ใช้ cache ก้อนเดียวกันทุก branch**

```yaml
cache:
  key: shared-cache
  paths:
    - node_modules/
```

ทุก branch, ทุก MR จะแชร์ cache ก้อนเดียวกันหมด เหมาะกับ dependency ที่ไม่ค่อยเปลี่ยนและไม่ต่างกันระหว่าง branch

**แบบที่ 2: key ตาม branch (แนะนำมากที่สุดสำหรับ dependency ทั่วไป)**

```yaml
cache:
  key: "$CI_COMMIT_REF_SLUG"
  paths:
    - node_modules/
```

`$CI_COMMIT_REF_SLUG` คือ predefined variable ที่เก็บชื่อ branch/tag ในรูปแบบที่ปลอดภัยสำหรับใช้เป็นชื่อไฟล์ (แทนที่อักขระพิเศษด้วย `-`) การใช้ key แบบนี้ทำให้แต่ละ branch มี cache แยกของตัวเอง ป้องกันปัญหา dependency ของ branch หนึ่งไปปนกับอีก branch หนึ่ง

**แบบที่ 3: key ตามไฟล์ lock (แม่นยำที่สุด — cache จะถูกสร้างใหม่เฉพาะเมื่อ dependency เปลี่ยนจริง ๆ)**

```yaml
cache:
  key:
    files:
      - package-lock.json
  paths:
    - node_modules/
```

รูปแบบนี้ให้ GitLab คำนวณ key จาก **เนื้อหาของไฟล์ `package-lock.json`** — ถ้าไฟล์นี้ไม่เปลี่ยนแปลงเลย (คือ dependency เดิมทุกตัว) cache จะยังคง valid และถูกใช้ซ้ำได้เรื่อย ๆ แต่ถ้ามีการแก้ `package-lock.json` (เพิ่ม/ลบ/อัปเดต dependency) GitLab จะรู้ทันทีว่าต้องสร้าง cache ก้อนใหม่ เพราะ key เปลี่ยนไปตามเนื้อหาไฟล์

### 704.4 `policy:` — ควบคุมว่า Job จะดาวน์โหลด/อัปโหลด Cache หรือไม่

ค่า `policy:` มี 3 แบบ:

| ค่า | ความหมาย |
|---|---|
| `pull-push` (ค่า default) | ดาวน์โหลด cache มาใช้ก่อนเริ่ม (pull) และอัปโหลด cache กลับไปหลังจบ (push) — ทำทั้งสองอย่าง |
| `pull` | ดาวน์โหลด cache มาใช้เท่านั้น ไม่อัปโหลดกลับ (ไม่แก้ไข cache) |
| `push` | อัปโหลด cache กลับเท่านั้น ไม่ดาวน์โหลดมาใช้ก่อน (สร้าง cache ใหม่จากศูนย์เสมอ) |

**เหตุผลที่ต้องใช้ `policy: pull`:** ในหลาย pipeline มี job จำนวนมากที่ **ใช้** cache แต่ไม่มีตัวไหนควร**แก้ไข**มัน (เช่น job test หลายตัวที่แค่ใช้ `node_modules/` แต่ไม่ได้ติดตั้งอะไรเพิ่ม) การกำหนดให้ job เหล่านี้ใช้ `policy: pull` จะช่วยประหยัดเวลา เพราะไม่ต้องเสียเวลาอัปโหลด cache ที่ไม่มีอะไรเปลี่ยนแปลงกลับไปซ้ำ ๆ ในทุก job

```yaml
stages:
  - install
  - test

install-dependencies:
  stage: install
  cache:
    key: "$CI_COMMIT_REF_SLUG"
    paths:
      - node_modules/
    policy: pull-push       # job นี้ทำหน้าที่สร้าง/อัปเดต cache
  script:
    - npm ci

unit-test:
  stage: test
  needs: ["install-dependencies"]
  cache:
    key: "$CI_COMMIT_REF_SLUG"
    paths:
      - node_modules/
    policy: pull             # job นี้แค่ใช้ cache ไม่ต้องอัปโหลดกลับ
  script:
    - npm run test:unit

lint:
  stage: test
  needs: ["install-dependencies"]
  cache:
    key: "$CI_COMMIT_REF_SLUG"
    paths:
      - node_modules/
    policy: pull             # เช่นเดียวกัน แค่ใช้ ไม่ต้องอัปโหลดกลับ
  script:
    - npm run lint
```

รูปแบบนี้เป็น pattern ที่นิยมมากในทีมงานจริง: มี job เดียวที่รับหน้าที่ "สร้าง cache" (`pull-push`) ส่วน job ที่เหลือทั้งหมดแค่ "ใช้" cache (`pull`) เท่านั้น ทำให้ pipeline โดยรวมเร็วขึ้นอย่างเห็นได้ชัด

### 704.5 ตั้งค่า Cache แบบ Global ด้วย `default:`

ถ้าเกือบทุก job ในไฟล์ใช้ cache แบบเดียวกัน สามารถประกาศไว้ที่ `default:` เพื่อไม่ต้องเขียนซ้ำในทุก job:

```yaml
default:
  cache:
    key: "$CI_COMMIT_REF_SLUG"
    paths:
      - node_modules/

stages:
  - test

unit-test:
  stage: test
  script:
    - npm ci
    - npm run test:unit

lint:
  stage: test
  script:
    - npm ci
    - npm run lint
```

### 704.6 Cache ไม่ใช่ Artifacts — ข้อควรจำที่สำคัญมาก

ผู้เรียนใหม่มักสับสนระหว่าง cache กับ artifacts เพราะหน้าตา YAML คล้ายกันมาก (`paths:` เหมือนกัน) แต่ทั้งสองมีจุดประสงค์ต่างกันโดยสิ้นเชิง:

| ประเด็น | Cache | Artifacts |
|---|---|---|
| จุดประสงค์หลัก | เร่งความเร็ว pipeline โดยไม่ต้องสร้างของเดิมซ้ำ | ส่งต่อผลลัพธ์งานจาก job หนึ่งไปยังอีก job หนึ่ง |
| ตัวอย่างที่ใช้เก็บ | `node_modules/`, `.m2/`, `vendor/bundle/` (dependency) | `dist/`, `build/`, ไฟล์ report ผลการทดสอบ |
| การรับประกันว่ามีอยู่จริง | **ไม่รับประกัน** — GitLab อาจลบ cache เก่าทิ้งได้ทุกเมื่อ (เช่นพื้นที่เต็ม) | รับประกันว่ามีอยู่ตราบใดที่ยังไม่หมดอายุ (`expire_in`) |
| ดาวน์โหลดได้จากหน้า UI ไหม | ดาวน์โหลดเองผ่าน UI ไม่ได้โดยตรง | ดาวน์โหลดเป็นไฟล์ zip ได้จากหน้า job โดยตรง |
| ใช้กับ `needs:` | ไม่เกี่ยวข้องกับ `needs:` โดยตรง | เชื่อมกับ `needs:` — job ที่ needs จะดาวน์โหลด artifact มาให้อัตโนมัติ |

> **กฎทองที่ต้องจำ:** ใช้ **cache** สำหรับสิ่งที่ "สร้างใหม่ได้เสมอถ้า cache หาย" เช่น dependency ที่ดาวน์โหลดใหม่ได้ ส่วน **artifacts** ใช้สำหรับ "ผลลัพธ์ของงานที่ต้องส่งต่อจริง ๆ" เช่นไฟล์ build ที่จะเอาไป deploy ห้ามใช้ cache แทน artifacts เพราะ cache ไม่รับประกันว่าจะมีอยู่จริงเสมอ

เราจะเจาะลึกเรื่อง artifacts อย่างเต็มที่ใน Step ถัดไป

---

## Step 705: Artifacts เจาะลึก (`paths`, `expire_in`, `reports`)

### 705.1 Artifacts คืออะไร (ทบทวนสั้น ๆ)

**Artifacts** คือไฟล์หรือโฟลเดอร์ที่ถูก **อัปโหลดขึ้น GitLab เก็บไว้อย่างถาวร (จนกว่าจะหมดอายุ)** หลังจาก job จบการทำงาน แตกต่างจาก cache ตรงที่ artifacts ถูกออกแบบมาเพื่อ **ส่งต่อผลลัพธ์งานจริง** ไม่ใช่แค่เร่งความเร็ว

### 705.2 `paths:` — ระบุว่าจะเก็บไฟล์ไหนเป็น Artifact

```yaml
build-job:
  stage: build
  script:
    - mkdir -p dist
    - echo "console.log('hello');" > dist/app.js
  artifacts:
    paths:
      - dist/
```

หลังจาก `build-job` จบ โฟลเดอร์ `dist/` ทั้งหมดจะถูกอัปโหลดเป็น artifact สามารถดาวน์โหลดเป็นไฟล์ zip ได้จากหน้า job บน GitLab UI โดยตรง (ปุ่ม "Download" มุมขวาบนของหน้า job)

สามารถระบุหลาย path พร้อมกัน และใช้ wildcard ได้:

```yaml
artifacts:
  paths:
    - dist/
    - build/
    - "*.log"
```

### 705.3 `expire_in:` — กำหนดวันหมดอายุของ Artifact

Artifact ที่ไม่กำหนดวันหมดอายุจะถูกเก็บไว้ตลอดไปตามค่า default ของ instance (ซึ่งอาจกินพื้นที่เก็บข้อมูลจำนวนมากถ้าไม่มีการจำกัด) การกำหนด `expire_in:` เป็นแนวปฏิบัติที่ดีเสมอ:

```yaml
artifacts:
  paths:
    - dist/
  expire_in: 1 week
```

ค่าที่รับได้มีหลายรูปแบบ เช่น `'30 minutes'`, `'1 day'`, `'1 week'`, `'3 mos'`, `'1 yr'`, หรือ `never` (ไม่ให้หมดอายุเลย)

```yaml
# ตัวอย่างค่าต่าง ๆ ที่ใช้ได้
expire_in: 1 hour
expire_in: 3 days
expire_in: 2 weeks
expire_in: never
```

> **แนวทางที่แนะนำ:** artifact ที่ใช้ส่งต่อระหว่าง job ใน pipeline เดียวกัน (เช่นไฟล์ build ที่จะถูก deploy ทันที) ตั้ง `expire_in` สั้น ๆ ได้ (เช่น `1 day`) เพราะใช้แค่ชั่วคราว ส่วน artifact ที่เป็นผลลัพธ์สุดท้ายที่ต้องการเก็บไว้ตรวจสอบย้อนหลัง (เช่น release build) อาจตั้งยาวกว่าหรือ `never`

### 705.4 `when:` ของ Artifacts — เก็บไฟล์แม้ Job ล้มเหลว

โดย default artifacts จะถูกเก็บเฉพาะเมื่อ job **สำเร็จ** เท่านั้น แต่บางครั้งเราต้องการเก็บไฟล์ผลลัพธ์แม้ job จะ **ล้มเหลว** ด้วย เช่น screenshot ของ test ที่ fail หรือ log ไฟล์ที่ช่วย debug:

```yaml
e2e-test:
  stage: test
  script:
    - npm run test:e2e
  artifacts:
    when: on_failure
    paths:
      - test-results/screenshots/
    expire_in: 3 days
```

ค่าที่ใช้ได้กับ `artifacts.when`:

| ค่า | ความหมาย |
|---|---|
| `on_success` (ค่า default) | เก็บ artifact เฉพาะเมื่อ job สำเร็จเท่านั้น |
| `on_failure` | เก็บ artifact เฉพาะเมื่อ job ล้มเหลวเท่านั้น |
| `always` | เก็บ artifact เสมอ ไม่ว่าผลลัพธ์จะเป็นอย่างไร |

### 705.5 `reports:` — Artifacts แบบพิเศษที่ GitLab เข้าใจความหมาย

นี่คือฟีเจอร์ที่ทรงพลังมาก — `reports:` เป็นชนิดพิเศษของ artifacts ที่ **GitLab รู้จักรูปแบบไฟล์และนำไปแสดงผลในหน้า UI โดยตรง** ไม่ใช่แค่เก็บไว้ให้ดาวน์โหลดเฉย ๆ

**5.1 `reports.junit` — แสดงผลการทดสอบใน Merge Request**

ถ้าเครื่องมือ test ของคุณสามารถ export ผลลัพธ์เป็นไฟล์ในรูปแบบ **JUnit XML** ได้ (เครื่องมือ test เกือบทุกภาษาโปรแกรมมิ่งทำได้ เช่น Jest, pytest, PHPUnit, JUnit เอง) สามารถส่งไฟล์นั้นเข้า `reports.junit` ได้:

```yaml
unit-test:
  stage: test
  script:
    - npm run test:unit -- --reporters=default --reporters=jest-junit
  artifacts:
    reports:
      junit: junit.xml
```

เมื่อตั้งค่านี้ไว้ GitLab จะ:

- แสดงจำนวน test ที่ผ่าน/ไม่ผ่านในหน้า **Pipeline Detail** โดยตรง (แท็บ "Tests")
- แสดงรายการ test ที่ fail พร้อมข้อความ error บนหน้า **Merge Request widget** ทันที โดยไม่ต้องเปิด Job Log เอง
- เปรียบเทียบผลลัพธ์กับ pipeline ก่อนหน้า บอกว่ามี test ไหนที่เพิ่งเริ่ม fail (regression) บ้าง

**5.2 `reports.coverage_report` — แสดง Code Coverage พร้อม Annotation ในไฟล์ Diff**

ถ้าเครื่องมือของคุณ export coverage เป็นรูปแบบ **Cobertura XML** ได้ สามารถส่งเข้า `reports.coverage_report`:

```yaml
unit-test:
  stage: test
  script:
    - npm run test:coverage
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
```

ฟีเจอร์นี้ทำให้ GitLab แสดง **เครื่องหมายสีเขียว/แดงข้างบรรทัดโค้ดที่เปลี่ยนแปลง** ในหน้า diff ของ Merge Request บอกได้ทันทีว่าโค้ดบรรทัดไหนที่เพิ่มเข้ามามี test ครอบคลุมหรือไม่

**5.3 ตารางสรุปประเภท `reports:` ที่พบบ่อย**

| ประเภท `reports:` | ใช้กับไฟล์รูปแบบ | ผลลัพธ์ที่เห็นบน GitLab UI |
|---|---|---|
| `junit` | JUnit XML | รายการ test ผ่าน/ไม่ผ่าน ในหน้า Pipeline และ MR widget |
| `coverage_report` | Cobertura XML | Annotation สีเขียว/แดงบน diff ของ MR |
| `dotenv` | ไฟล์ `.env` (key=value) | ส่งตัวแปรจาก job หนึ่งไปเป็น CI/CD variable ใน job ถัดไป |

> **หมายเหตุ:** `reports.dotenv` เป็นเทคนิคขั้นสูงที่ใช้ส่งค่าตัวแปรระหว่าง job ผ่าน `needs:` โดยไม่ต้องเขียนไฟล์ธรรมดาแล้วอ่านเอง — จะกล่าวถึงอย่างละเอียดในเนื้อหาขั้นสูงถัดไปของหลักสูตรเมื่อพูดถึง Dynamic Pipelines

### 705.6 ผสม `paths` และ `reports` ใน Job เดียวกันได้

`artifacts:` สามารถมีทั้ง `paths` (สำหรับดาวน์โหลดทั่วไป) และ `reports` (สำหรับให้ GitLab แปลผล) พร้อมกันในบล็อกเดียว:

```yaml
unit-test:
  stage: test
  script:
    - npm run test:coverage -- --reporters=default --reporters=jest-junit
  artifacts:
    when: always
    expire_in: 1 week
    paths:
      - coverage/
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
```

ตัวอย่างนี้ครบเครื่อง: เก็บโฟลเดอร์ `coverage/` ทั้งหมดไว้ดาวน์โหลดได้ (`paths`) พร้อมทั้งให้ GitLab แสดงผล test (`junit`) และ coverage annotation (`coverage_report`) — และตั้ง `when: always` เพื่อให้เก็บผลลัพธ์แม้ test จะ fail ก็ตาม ซึ่งสำคัญมาก เพราะเราอยากเห็นรายงาน test ที่ fail ด้วยเช่นกัน ไม่ใช่แค่ตอนผ่านเท่านั้น

---

## Step 706: `extends` — ลดโค้ดซ้ำซ้อนด้วย Template

### 706.1 ปัญหาโค้ดซ้ำซ้อนที่ `extends` แก้ให้

ลองดู pipeline ที่มี job หลายตัวใช้ `image` และ `before_script` เหมือนกันหมด:

```yaml
unit-test:
  stage: test
  image: node:18
  before_script:
    - npm ci
  script:
    - npm run test:unit

integration-test:
  stage: test
  image: node:18
  before_script:
    - npm ci
  script:
    - npm run test:integration

lint:
  stage: test
  image: node:18
  before_script:
    - npm ci
  script:
    - npm run lint
```

`image: node:18` และ `before_script: npm ci` ถูกเขียนซ้ำถึง 3 ครั้ง — ถ้าต้องเปลี่ยนเวอร์ชัน Node.js ในอนาคต ต้องมาแก้ทีละจุดถึง 3 ที่ ซึ่งเสี่ยงต่อการแก้ไม่ครบและลืมบางจุด

### 706.2 Hidden Job (Job ที่ขึ้นต้นด้วยจุด) — วัตถุดิบของ `extends`

GitLab มีกฎพิเศษว่า **job ที่ชื่อขึ้นต้นด้วยจุด (`.`) จะไม่ถูกรันเป็น job จริง** — GitLab จะมองข้ามมันไปตอนสร้าง pipeline เรียกว่า **Hidden Job** ซึ่งเหมาะมากที่จะใช้เป็น "template" เก็บค่าที่ใช้ร่วมกัน

```yaml
.node-defaults:
  image: node:18
  before_script:
    - npm ci
```

`.node-defaults` จะไม่ปรากฏเป็น job ใน pipeline เลย มันมีไว้เพื่อให้ job อื่นมา `extends` เท่านั้น

### 706.3 ใช้ `extends:` เพื่อดึงค่าจาก Template มาใช้

```yaml
stages:
  - test

.node-defaults:
  image: node:18
  before_script:
    - npm ci

unit-test:
  extends: .node-defaults
  stage: test
  script:
    - npm run test:unit

integration-test:
  extends: .node-defaults
  stage: test
  script:
    - npm run test:integration

lint:
  extends: .node-defaults
  stage: test
  script:
    - npm run lint
```

ผลลัพธ์เทียบเท่ากับตัวอย่างแรกเป๊ะ ๆ แต่ตอนนี้ **ถ้าต้องเปลี่ยนเวอร์ชัน Node.js แก้แค่จุดเดียว** ที่ `.node-defaults`

### 706.4 กลไกการ Merge ของ `extends` — สิ่งที่ต้องเข้าใจให้ถูกต้อง

`extends` ทำงานแบบ **deep merge (ผสานแบบลึก)** สำหรับค่าที่เป็น map (key-value) แต่ **แทนที่ทั้งก้อน (ไม่ผสาน)** สำหรับค่าที่เป็น list (array) — นี่คือจุดที่ผู้เรียนสับสนบ่อยที่สุด

```yaml
.base-job:
  variables:
    APP_ENV: "staging"
    LOG_LEVEL: "info"
  script:
    - echo "step 1"
    - echo "step 2"

my-job:
  extends: .base-job
  variables:
    LOG_LEVEL: "debug"        # override เฉพาะ key นี้ ส่วน APP_ENV ยังอยู่
  script:
    - echo "step ใหม่ทั้งหมด"   # แทนที่ script เดิมทั้งชุด ไม่ใช่ต่อท้าย
```

ผลลัพธ์จริงของ `my-job` หลัง merge คือ:

```yaml
# เทียบเท่ากับ (ผลลัพธ์หลัง GitLab ประมวลผล extends แล้ว)
my-job:
  variables:
    APP_ENV: "staging"   # ยังคงมาจาก .base-job เพราะ my-job ไม่ได้ override key นี้
    LOG_LEVEL: "debug"   # ถูก override ด้วยค่าของ my-job
  script:
    - echo "step ใหม่ทั้งหมด"   # script ทั้งก้อนถูกแทนที่ ไม่ใช่รวมกับของเดิม
```

> **กฎที่ต้องจำ:** `variables:` ผสานกันแบบ key-by-key (override เฉพาะ key ที่ซ้ำ) แต่ `script:`, `before_script:`, `after_script:`, `tags:` และ list อื่น ๆ จะถูก **แทนที่ทั้งก้อน** ถ้า job ที่ extends ประกาศ key นั้นซ้ำ ดังนั้นถ้าอยากได้ `before_script` จาก template ต่อด้วย `script` ของตัวเอง ให้ประกาศแค่ `script:` ใน job ลูก แล้วปล่อยให้ `before_script:` มาจาก template เพียงอย่างเดียว ไม่ต้องประกาศซ้ำ

### 706.5 `extends:` แบบหลาย Template พร้อมกัน (Array)

`extends:` รับเป็น list ได้ด้วย ทำให้ผสมหลาย template เข้าด้วยกันได้:

```yaml
.node-defaults:
  image: node:18
  before_script:
    - npm ci

.docker-tags:
  tags:
    - docker

unit-test:
  extends:
    - .node-defaults
    - .docker-tags
  stage: test
  script:
    - npm run test:unit
```

เมื่อ `extends:` เป็น list กฎการ merge คือ **template ที่อยู่หลังสุดในลำดับจะมีความสำคัญเหนือกว่า** (override) template ที่อยู่ก่อนหน้า ถ้ามี key ซ้ำกัน

### 706.6 `extends` vs `include` — อย่าสับสน

`extends` กับ `include` (ที่จะเรียนใน Step ถัดไป) ทำหน้าที่ต่างกันโดยสิ้นเชิง แม้จะฟังดูคล้ายกัน:

| ประเด็น | `extends` | `include` |
|---|---|---|
| ทำงานที่ระดับ | Job — ให้ job หนึ่งดึงค่าจาก job (template) อื่นในไฟล์เดียวกัน (หรือไฟล์ที่ include เข้ามาแล้ว) | ไฟล์ — ให้ไฟล์ `.gitlab-ci.yml` ดึงเนื้อหาทั้งไฟล์จากที่อื่นเข้ามารวมกัน |
| แก้ปัญหาอะไร | โค้ดซ้ำซ้อนระหว่าง job | ไฟล์ `.gitlab-ci.yml` ยาวเกินไป ต้องการแยกเป็นไฟล์ย่อย |
| ใช้ร่วมกันได้ไหม | ได้ — มักใช้คู่กันเสมอในโปรเจกต์ขนาดใหญ่ | ได้เช่นกัน |

---

## Step 707: `include` — แยกไฟล์ `.gitlab-ci.yml` เป็นหลายไฟล์ย่อย

### 707.1 ทำไมต้องแยกไฟล์

เมื่อโปรเจกต์โตขึ้น ไฟล์ `.gitlab-ci.yml` เดียวอาจยาวเป็นพันบรรทัด รวมทุก stage ทุก job ทุก template ไว้ในที่เดียว ทำให้:

- ดูแลรักษายาก หาจุดที่ต้องการแก้ไม่เจอ
- Merge Request ที่แก้ไข CI/CD มักเกิด conflict บ่อย เพราะหลายคนแก้ไฟล์เดียวกัน
- ไม่สามารถแชร์ config ชุดเดียวกันข้ามหลายโปรเจกต์ได้ง่าย ๆ

`include:` แก้ปัญหานี้โดยให้แยกเนื้อหาออกเป็นไฟล์ย่อยตามหมวดหมู่ แล้วรวมเข้าด้วยกันที่ไฟล์หลัก

### 707.2 `include.local` — รวมไฟล์จาก Repository เดียวกัน

```yaml
# .gitlab-ci.yml (ไฟล์หลักที่ root)
include:
  - local: '.gitlab/ci/build.yml'
  - local: '.gitlab/ci/test.yml'
  - local: '.gitlab/ci/deploy.yml'

stages:
  - build
  - test
  - deploy
```

โครงสร้างโฟลเดอร์ตัวอย่าง:

```
my-project/
├── .gitlab-ci.yml
├── .gitlab/
│   └── ci/
│       ├── build.yml
│       ├── test.yml
│       └── deploy.yml
```

ไฟล์ `.gitlab/ci/build.yml`:

```yaml
build-frontend:
  stage: build
  script:
    - echo "build frontend"

build-backend:
  stage: build
  script:
    - echo "build backend"
```

ไฟล์ `.gitlab/ci/test.yml`:

```yaml
unit-test:
  stage: test
  script:
    - echo "unit test"
```

เมื่อรวมกันแล้ว GitLab จะมองเห็นเหมือนกับเขียนทุกอย่างไว้ในไฟล์เดียว

### 707.3 `include.project` — ดึงไฟล์ CI/CD จากโปรเจกต์อื่นบน GitLab เดียวกัน

มีประโยชน์มากเมื่อองค์กรต้องการ **แชร์ template CI/CD กลาง** ให้หลายโปรเจกต์ใช้ร่วมกัน (เช่นทุกโปรเจกต์ในบริษัทต้องมีขั้นตอน security scan แบบเดียวกัน):

```yaml
include:
  - project: 'my-group/ci-templates'
    ref: main
    file: '/templates/security-scan.yml'
```

- `project:` คือ path เต็มของโปรเจกต์ต้นทางที่เก็บไฟล์ template
- `ref:` คือ branch/tag/commit ของโปรเจกต์นั้นที่จะดึงไฟล์มา (ถ้าไม่ระบุจะใช้ default branch)
- `file:` คือ path ของไฟล์ภายในโปรเจกต์ต้นทาง

> **ข้อกำหนด:** ผู้ที่ push โค้ดต้องมีสิทธิ์เข้าถึง (อย่างน้อย read) โปรเจกต์ต้นทางที่ระบุใน `project:` ด้วย ไม่เช่นนั้น pipeline จะไม่สามารถดึงไฟล์มาได้

### 707.4 `include.remote` — ดึงไฟล์จาก URL ภายนอก

```yaml
include:
  - remote: 'https://internal-ci-config.example-corp.local/templates/base.yml'
```

ไฟล์จะถูกดึงมาจาก URL สาธารณะที่เข้าถึงได้ (ไม่ต้อง authentication) เหมาะกับกรณีที่ต้องการดึง config จากระบบภายนอก GitLab เอง เช่นเซิร์ฟเวอร์ config กลางขององค์กร

> **ข้อควรระวัง:** เนื่องจากไฟล์มาจากภายนอก ควรตรวจสอบให้แน่ใจว่า URL นั้นน่าเชื่อถือและควบคุมได้จริง เพราะไฟล์นี้จะถูกรันเป็นส่วนหนึ่งของ pipeline โดยตรง

### 707.5 `include.template` — ใช้ Template สำเร็จรูปที่ GitLab เตรียมไว้ให้

GitLab มี template สำเร็จรูปติดตั้งมาให้ในตัวสำหรับงานที่พบบ่อย เช่น Security Scanning:

```yaml
include:
  - template: 'Security/SAST.gitlab-ci.yml'
```

Template เหล่านี้ดูแลและอัปเดตโดยทีม GitLab เอง ไม่ต้องเขียนเอง — เพียงพอสำหรับ Part นี้ที่จะรู้ว่ามันมีอยู่และเรียกใช้แบบนี้ ส่วนรายละเอียดเชิงลึกของแต่ละ template (โดยเฉพาะกลุ่ม Security) จะอยู่ในเนื้อหาเฟส 8 ต่อไป

### 707.6 ผสมทุกแบบของ `include` ในไฟล์เดียวกันได้

```yaml
include:
  - local: '.gitlab/ci/build.yml'
  - local: '.gitlab/ci/test.yml'
  - project: 'my-group/ci-templates'
    ref: main
    file: '/templates/deploy-common.yml'
  - template: 'Security/SAST.gitlab-ci.yml'

stages:
  - build
  - test
  - security
  - deploy
```

### 707.7 ลำดับความสำคัญเมื่อมี Job ชื่อซ้ำกันระหว่างไฟล์ที่ Include

ถ้า job ชื่อเดียวกันถูกประกาศทั้งในไฟล์ที่ `include` เข้ามา และในไฟล์หลัก (หรือใน `include` หลายรายการ) กฎการ override คือ:

> **ไฟล์/include ที่ประมวลผลทีหลังจะ override ไฟล์/include ที่ประมวลผลก่อนหน้า และเนื้อหาในไฟล์หลัก (`.gitlab-ci.yml`) เองจะถูกประมวลผลเป็นลำดับสุดท้ายเสมอ ทำให้ชนะทุกกรณี**

```yaml
include:
  - local: '.gitlab/ci/base.yml'   # ประกาศ test-job แบบหนึ่ง

test-job:                          # ประกาศซ้ำในไฟล์หลัก
  stage: test
  script:
    - echo "เวอร์ชันนี้จะชนะเสมอ เพราะอยู่ในไฟล์หลัก"
```

การเข้าใจกฎนี้ช่วยป้องกันความสับสนเวลา config พฤติกรรมไม่ตรงกับที่คาดไว้ เมื่อมีการ override เกิดขึ้นโดยไม่ได้ตั้งใจ

### 707.8 ตรวจสอบผลลัพธ์สุดท้ายหลังรวมไฟล์ทั้งหมดด้วย CI Lint

เมื่อไฟล์ถูกแยกออกหลายไฟล์แล้ว การอ่านว่า "pipeline จริง ๆ หน้าตาเป็นอย่างไรหลังรวมทุกไฟล์" อาจทำได้ยากด้วยตาเปล่า ใช้เมนู **CI/CD > Editor** แล้วดูที่แท็บ **View merged YAML** เพื่อดูไฟล์ผลลัพธ์สุดท้ายหลังจากรวม `include` ทั้งหมดและประมวลผล `extends` เรียบร้อยแล้ว — เป็นเครื่องมือสำคัญมากเมื่อ config เริ่มซับซ้อน

---

## Step 708: Environments — ผูก Deploy Job เข้ากับ Environment

### 708.1 Environment คืออะไร

**Environment (สภาพแวดล้อม)** คือการบอก GitLab ว่า "job นี้กำลัง deploy ไปที่ไหน" เช่น `staging`, `production`, `review/feature-x` เมื่อประกาศ `environment:` ให้กับ job ที่ทำหน้าที่ deploy แล้ว GitLab จะ:

- สร้างหน้า **Deployments > Environments** ที่แสดงประวัติการ deploy ทั้งหมดของแต่ละ environment
- แสดงลิงก์ไปยัง environment นั้นได้โดยตรงจากหน้า Merge Request หรือหน้า Pipeline
- รองรับการกด **Re-deploy** หรือ **Rollback** กลับไปยัง deployment ก่อนหน้าได้จาก UI โดยตรง
- แสดงว่า deployment ล่าสุดของ environment แต่ละตัวมาจาก commit ไหน ใครเป็นคน trigger

### 708.2 การใช้งานพื้นฐาน — `name` และ `url`

```yaml
deploy-staging:
  stage: deploy
  script:
    - echo "กำลัง deploy ขึ้น staging"
    - ./deploy.sh staging
  environment:
    name: staging
    url: https://staging.example-app.internal
```

- `name:` — ชื่อของ environment (จะไปปรากฏในหน้า Environments)
- `url:` — ลิงก์ที่คลิกแล้วพาไปยัง environment นั้นโดยตรง (ปรากฏเป็นปุ่มบนหน้า MR และหน้า Pipeline)

### 708.3 Environment แบบไดนามิก — สำหรับ Review App ต่อ Merge Request

เทคนิคที่นิยมมากคือการสร้าง environment แยกให้แต่ละ Merge Request โดยอัตโนมัติ (เรียกว่า "Review App") โดยใช้ predefined variable ประกอบชื่อ:

```yaml
deploy-review:
  stage: deploy
  script:
    - echo "deploy review app สำหรับ MR นี้โดยเฉพาะ"
    - ./deploy.sh "review-$CI_COMMIT_REF_SLUG"
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://$CI_COMMIT_REF_SLUG.review.example-app.internal
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

ทุก Merge Request จะได้ environment เป็นของตัวเอง เช่น `review/feature-login-page`, `review/fix-navbar-bug` ทำให้ทีมสามารถทดสอบแต่ละ MR แยกกันได้จริงบนสภาพแวดล้อมของตัวเอง

### 708.4 ปิด Environment อัตโนมัติเมื่อ Merge Request ถูกปิด/Merge

Environment แบบ Review App ควรถูก "ทำลาย" (stop) เมื่อ MR ถูกปิดหรือ merge แล้ว เพื่อไม่ให้ทรัพยากรค้างอยู่โดยไม่มีใครใช้ ทำได้ด้วย `on_stop:` และ job ที่มี `action: stop`:

```yaml
deploy-review:
  stage: deploy
  script:
    - ./deploy.sh "review-$CI_COMMIT_REF_SLUG"
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://$CI_COMMIT_REF_SLUG.review.example-app.internal
    on_stop: stop-review
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'

stop-review:
  stage: deploy
  script:
    - ./destroy-review-app.sh "review-$CI_COMMIT_REF_SLUG"
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop
  when: manual
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

- `on_stop: stop-review` บอก GitLab ว่า job ชื่อ `stop-review` คือ job ที่ใช้ปิด environment นี้
- `stop-review` เองต้องประกาศ `environment.action: stop` เพื่อให้ GitLab รู้ว่านี่คือ job สำหรับปิด ไม่ใช่ deploy ใหม่
- เมื่อ MR ถูกปิดหรือ merge แล้ว GitLab จะรัน `stop-review` ให้อัตโนมัติ (หรือให้กดปิดเองจากหน้า Environments ก็ได้)

### 708.5 Environment สำหรับ Production ควรผูกกับ Manual Deploy เสมอ

Environment ที่สำคัญอย่าง `production` ควรผูกกับ `when: manual` เสมอ เพื่อให้มีคนตรวจสอบและกดยืนยันก่อน deploy จริงทุกครั้ง (จะอธิบาย `when: manual` แบบละเอียดใน Step ถัดไป):

```yaml
deploy-production:
  stage: deploy
  script:
    - ./deploy.sh production
  environment:
    name: production
    url: https://example-app.com
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
      when: manual
    - when: never
```

### 708.6 ดูประวัติ Deployment ทั้งหมดที่หน้า Environments

ไปที่เมนู **Deployments > Environments** ในโปรเจกต์บน GitLab จะเห็นรายชื่อ environment ทั้งหมดที่เคยถูกสร้างผ่าน pipeline พร้อมสถานะล่าสุด, เวลาที่ deploy ล่าสุด, และปุ่มสำหรับ:

- **Re-deploy** — สั่งรัน deploy job ตัวเดิมซ้ำอีกครั้ง
- **Rollback** — deploy ย้อนกลับไปยัง deployment เวอร์ชันก่อนหน้า (ถ้า pipeline นั้นยังมี artifact ที่จำเป็นอยู่)
- **Stop** — สั่งปิด environment (ถ้ามีการตั้ง `on_stop:` ไว้)

หน้านี้ทำให้เห็นภาพรวมว่า "ตอนนี้แต่ละสภาพแวดล้อมกำลังรันโค้ดเวอร์ชันไหนอยู่" ได้ทันทีโดยไม่ต้องไล่ดู pipeline history ทีละอัน

### 708.7 ตารางสรุป Keyword ที่เกี่ยวกับ Environment

| Keyword | ใช้ที่ไหน | ความหมาย |
|---|---|---|
| `environment.name` | ใน job | ชื่อของ environment |
| `environment.url` | ใน job | ลิงก์ที่คลิกไปยัง environment นั้น |
| `environment.on_stop` | ใน job ที่ deploy | ชื่อ job ที่ใช้ปิด environment นี้ |
| `environment.action` | ใน job ที่ทำหน้าที่ปิด | ตั้งเป็น `stop` เพื่อบอกว่านี่คือ job สำหรับปิด environment |

---

## Step 709: Manual Jobs และ `when:` ทุกค่าที่มี

### 709.1 ทบทวน `when` จาก Part 50 แล้วขยายให้ครบทุกค่า

Part 50 ได้แนะนำค่า `when` ไปบ้างแล้วในบริบทของ `rules` มาดูสรุปให้ครบทุกค่าที่มีอยู่จริงอีกครั้งพร้อมตัวอย่างเชิงลึก:

| ค่า `when` | ความหมาย | ใช้บ่อยกับ |
|---|---|---|
| `on_success` | รัน job นี้ถ้า job/stage ก่อนหน้าทั้งหมดผ่าน (ค่า default ถ้าไม่ระบุ) | job ทั่วไปเกือบทั้งหมด |
| `on_failure` | รัน job นี้เฉพาะเมื่อมี job ก่อนหน้าล้มเหลว | job แจ้งเตือนเมื่อ pipeline พัง, job cleanup พิเศษ |
| `always` | รันเสมอไม่ว่าผลลัพธ์ก่อนหน้าจะเป็นอย่างไร | job ส่งรายงานสรุป, job แจ้งเตือนที่ต้องรันทุกกรณี |
| `manual` | ต้องมีคนกดปุ่ม "Run" ใน UI ก่อนถึงจะเริ่มทำงาน | deploy production, ลบข้อมูล, งานที่มีความเสี่ยงสูง |
| `delayed` | รอตามเวลาที่กำหนด (คู่กับ `start_in`) ก่อนเริ่มรันอัตโนมัติ | rollback แบบมีช่วงเวลาให้ยกเลิกได้ก่อน |
| `never` | ไม่รัน job นี้เลย (ใช้ในเงื่อนไข `rules` เพื่อปิด job ในบางกรณี) | ปิด job ในเงื่อนไขที่ไม่ต้องการให้ทำงาน |

### 709.2 `when: manual` — Job ที่ต้องกดยืนยันเอง

```yaml
deploy-production:
  stage: deploy
  script:
    - ./deploy.sh production
  environment:
    name: production
  when: manual
```

เมื่อ pipeline มาถึง job นี้ มันจะแสดงสถานะเป็น **Manual** (ไอคอนรูปคนกดปุ่ม) บนหน้า Pipeline Detail และ **จะไม่เริ่มทำงานเองโดยอัตโนมัติ** ต้องมีคนเข้าไปกดปุ่ม "▶ Run" ที่หน้า Pipeline หรือหน้า job นั้นเองก่อน job ถึงจะเริ่มทำงานจริง

**ข้อควรรู้เรื่อง Manual Job กับ Pipeline โดยรวม:**

- ค่า default ถ้า manual job ยังไม่ถูกกดรัน **pipeline โดยรวมจะแสดงสถานะ "Passed" ได้** ถึงแม้ manual job ยังไม่ได้รัน (เพราะ manual job ที่ยังไม่กด ไม่นับว่า fail)
- ถ้าต้องการให้ pipeline รอ/ขึ้นสถานะพิเศษจนกว่า manual job จะถูกกด สามารถใช้ `allow_failure: false` ร่วมกับ `when: manual` เพื่อบอกว่า "ถ้า job นี้ถูกกดแล้ว fail จริง ให้ทั้ง pipeline fail ตามไปด้วย" (ค่า default ของ manual job ที่ fail คือ `allow_failure: true` ซึ่งจะไม่ทำให้ pipeline โดยรวม fail)

```yaml
deploy-production:
  stage: deploy
  script:
    - ./deploy.sh production
  environment:
    name: production
  when: manual
  allow_failure: false
```

### 709.3 `when: manual` ผสมกับ `rules` — รูปแบบที่ใช้บ่อยที่สุดในงานจริง

```yaml
deploy-production:
  stage: deploy
  script:
    - ./deploy.sh production
  environment:
    name: production
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
      when: manual
      allow_failure: false
    - when: never
```

อธิบาย: deploy job นี้จะปรากฏเป็น manual job **เฉพาะเมื่อ pipeline รันจาก default branch เท่านั้น** (เช่น `main`) ส่วน branch อื่นทั้งหมด job นี้จะไม่ปรากฏเลย (`when: never`) — นี่คือ pattern มาตรฐานสำหรับ deploy production ในเกือบทุกองค์กร

> **ข้อควรระวังสำคัญ:** เมื่อ job ใช้ `rules:` แล้ว การใส่ `when:` ที่ระดับ job (นอก `rules`) พร้อมกันจะทำให้เกิดข้อขัดแย้งด้าน config — ให้ใส่ `when:` ไว้ **ภายในแต่ละรายการของ `rules:`** เสมอ ไม่ใช่นอก `rules:` เมื่อ job นั้นมี `rules:` อยู่แล้ว

### 709.4 `when: delayed` — หน่วงเวลาก่อนรันอัตโนมัติ

`when: delayed` ใช้คู่กับ `start_in:` เพื่อให้ job รอตามเวลาที่กำหนดก่อนเริ่มทำงานเอง โดยไม่ต้องมีคนกด:

```yaml
rollback-if-needed:
  stage: deploy
  script:
    - echo "ตรวจสอบ health check แล้วตัดสินใจว่าต้อง rollback หรือไม่"
    - ./check-and-rollback.sh
  when: delayed
  start_in: 10 minutes
```

Job นี้จะปรากฏในหน้า pipeline พร้อมตัวนับถอยหลัง 10 นาที ระหว่างนั้นสามารถกดยกเลิกได้ก่อนที่มันจะเริ่มทำงานจริง ค่าที่ใช้กับ `start_in` ได้ เช่น `'5 minutes'`, `'1 hour'`, `'1 day'` (สูงสุดไม่เกิน 1 สัปดาห์)

### 709.5 `when: on_failure` — Job ที่รันเฉพาะเมื่อมีอะไรพัง

```yaml
notify-failure:
  stage: .post
  script:
    - echo "แจ้งเตือนทีมว่า pipeline ล้มเหลว (จำลอง)"
  when: on_failure
```

Job นี้จะถูกข้ามเงียบ ๆ ถ้าทุกอย่างผ่านหมด แต่จะถูกรันทันทีถ้ามี job ใดก่อนหน้าล้มเหลว เหมาะมากสำหรับ job แจ้งเตือนทีมผ่านช่องทางต่าง ๆ เมื่อ pipeline มีปัญหา

### 709.6 `when: always` — Job ที่ต้องรันเสมอไม่ว่าอะไรจะเกิดขึ้น

```yaml
send-pipeline-summary:
  stage: .post
  script:
    - echo "ส่งสรุปผลลัพธ์ pipeline ไม่ว่าจะผ่านหรือไม่ผ่าน"
  when: always
```

แตกต่างจาก `after_script` ตรงที่ `when: always` ใช้กับ **job ทั้งตัว** (มี stage, artifacts, environment ของตัวเองได้ครบ) ในขณะที่ `after_script` เป็นแค่คำสั่งเสริมภายใน job เดียวกันเท่านั้น

### 709.7 ตารางเปรียบเทียบสถานการณ์ใช้งานจริงของแต่ละค่า `when`

| สถานการณ์ | ค่า `when` ที่เหมาะสม |
|---|---|
| Deploy ขึ้น production | `manual` (บวก `allow_failure: false` ถ้าต้องการให้ผลกระทบต่อสถานะ pipeline) |
| แจ้งเตือนทีมเมื่อ pipeline พัง | `on_failure` |
| ส่งรายงานสรุปทุกครั้งไม่ว่าผลจะเป็นอย่างไร | `always` |
| รอเวลาก่อน rollback อัตโนมัติเผื่อมีคนอยากยกเลิก | `delayed` + `start_in` |
| ปิด job ในบาง branch โดยสิ้นเชิง | `never` (ใน `rules`) |
| Job ทดสอบทั่วไป ให้รันตามปกติ | `on_success` (ค่า default ไม่ต้องระบุก็ได้) |

---

## Step 710: แบบฝึกหัด — เขียน Pipeline เต็มรูปแบบ 4 Stage พร้อม Cache, Artifact และ DAG

ถึงเวลารวบยอดทุกเทคนิคที่เรียนมาใน Part นี้เข้าด้วยกัน เขียน pipeline ที่ใกล้เคียงงานจริงมากที่สุดเท่าที่จะทำได้ในระดับนี้

### 710.1 เป้าหมายของแบบฝึกหัดนี้

เขียน `.gitlab-ci.yml` สำหรับโปรเจกต์ Node.js สมมติ ที่มี 4 stage คือ `build`, `test`, `package`, `deploy` และต้องใช้เทคนิคต่อไปนี้ให้ครบ:

- [ ] `cache` แบบ key ตาม branch เพื่อไม่ต้อง `npm ci` ซ้ำทุก job
- [ ] `artifacts` ส่งต่อไฟล์ที่ build แล้วจาก stage `build` ไปยัง stage `package`
- [ ] `artifacts.reports.junit` เพื่อแสดงผล test ใน UI
- [ ] `needs:` อย่างน้อย 1 จุด เพื่อทำให้ pipeline เป็น DAG บางส่วน
- [ ] `extends` เพื่อลดโค้ดซ้ำระหว่าง job
- [ ] `environment` สำหรับ job deploy
- [ ] `when: manual` สำหรับ deploy production

### 710.2 โครงสร้างโปรเจกต์สมมติ

```
my-app/
├── .gitlab-ci.yml
├── package.json
├── package-lock.json
└── src/
    └── index.js
```

### 710.3 เขียน Pipeline แบบเต็มรูปแบบทีละส่วน

**ส่วนที่ 1: ประกาศ stage และ template ที่ใช้ร่วมกัน**

```yaml
stages:
  - build
  - test
  - package
  - deploy

.node-defaults:
  image: node:18
  cache:
    key: "$CI_COMMIT_REF_SLUG"
    paths:
      - node_modules/
    policy: pull
  before_script:
    - npm ci
```

**ส่วนที่ 2: Stage `build` — ติดตั้ง dependency และสร้าง cache หลัก**

```yaml
install-dependencies:
  extends: .node-defaults
  stage: build
  cache:
    key: "$CI_COMMIT_REF_SLUG"
    paths:
      - node_modules/
    policy: pull-push
  before_script: []
  script:
    - npm ci
    - echo "ติดตั้ง dependency และสร้าง cache เรียบร้อย"

build-app:
  extends: .node-defaults
  stage: build
  needs: ["install-dependencies"]
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 day
```

สังเกตว่า `install-dependencies` override `before_script` เป็น `[]` (ว่างเปล่า) แล้วย้ายคำสั่งไปไว้ใน `script` แทน เพื่อให้ policy เป็น `pull-push` โดยเฉพาะสำหรับ job นี้เพียงตัวเดียว ในขณะที่ job อื่นที่ extends จาก `.node-defaults` จะใช้ `policy: pull` ตามค่าเดิมของ template

**ส่วนที่ 3: Stage `test` — รันขนานกันหลาย job โดยใช้ cache ที่สร้างไว้แล้ว**

```yaml
unit-test:
  extends: .node-defaults
  stage: test
  needs: ["install-dependencies"]
  script:
    - npm run test:unit -- --reporters=default --reporters=jest-junit
  artifacts:
    when: always
    expire_in: 3 days
    reports:
      junit: junit.xml

lint:
  extends: .node-defaults
  stage: test
  needs: ["install-dependencies"]
  script:
    - npm run lint
```

จุดสำคัญ: `unit-test` และ `lint` ทั้งคู่ใช้ `needs: ["install-dependencies"]` แทนที่จะรอ stage `build` ทั้งหมด (ซึ่งรวม `build-app` ด้วย) ทำให้ทั้งสอง job เริ่มทำงานได้ทันทีที่ `install-dependencies` เสร็จ **โดยไม่ต้องรอ `build-app` เลย** เพราะทั้งคู่ไม่ได้ใช้ผลลัพธ์จาก `build-app` แต่อย่างใด — นี่คือการประยุกต์ใช้ DAG pipeline จริงตาม Step 703

**ส่วนที่ 4: Stage `package` — รวมไฟล์ทั้งหมดที่จำเป็นสำหรับ deploy**

```yaml
package-release:
  stage: package
  needs:
    - job: build-app
      artifacts: true
    - job: unit-test
      artifacts: false
    - job: lint
      artifacts: false
  script:
    - mkdir -p release
    - cp -r dist/ release/
    - echo "$CI_COMMIT_SHORT_SHA" > release/VERSION
    - tar -czf release.tar.gz release/
  artifacts:
    paths:
      - release.tar.gz
    expire_in: 1 week
```

`package-release` ระบุ `needs` ครบทั้ง 3 job ของ stage ก่อนหน้า เพื่อรับประกันว่าทั้ง build เสร็จสมบูรณ์และ test/lint ผ่านหมดแล้วจริง ๆ ก่อนจะห่อไฟล์ปล่อยออกไป โดยดาวน์โหลด artifact มาใช้จริงเฉพาะจาก `build-app` เท่านั้น (`artifacts: true`) ส่วนอีกสอง job แค่ใช้เป็นเงื่อนไขว่า "ต้องผ่านก่อน" โดยไม่ต้องการไฟล์ใด ๆ จากมัน (`artifacts: false`)

**ส่วนที่ 5: Stage `deploy` — staging รันอัตโนมัติ, production ต้องกดยืนยัน**

```yaml
deploy-staging:
  stage: deploy
  needs: ["package-release"]
  script:
    - tar -xzf release.tar.gz
    - echo "กำลัง deploy ขึ้น staging"
    - ./deploy.sh staging release/
  environment:
    name: staging
    url: https://staging.my-app.internal
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'

deploy-production:
  stage: deploy
  needs: ["deploy-staging"]
  script:
    - tar -xzf release.tar.gz
    - echo "กำลัง deploy ขึ้น production"
    - ./deploy.sh production release/
  environment:
    name: production
    url: https://my-app.example.com
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
      when: manual
      allow_failure: false
    - when: never
```

`deploy-production` ตั้งใจให้ `needs: ["deploy-staging"]` เพื่อบังคับว่า **ต้อง deploy ขึ้น staging สำเร็จก่อนเสมอ ถึงจะมีสิทธิ์กด deploy production ได้** — เป็นแนวปฏิบัติที่ปลอดภัยมากในการทำงานจริง

### 710.4 ไฟล์ `.gitlab-ci.yml` ฉบับเต็มรวมทุกส่วนเข้าด้วยกัน

```yaml
stages:
  - build
  - test
  - package
  - deploy

.node-defaults:
  image: node:18
  cache:
    key: "$CI_COMMIT_REF_SLUG"
    paths:
      - node_modules/
    policy: pull
  before_script:
    - npm ci

install-dependencies:
  extends: .node-defaults
  stage: build
  cache:
    key: "$CI_COMMIT_REF_SLUG"
    paths:
      - node_modules/
    policy: pull-push
  before_script: []
  script:
    - npm ci
    - echo "ติดตั้ง dependency และสร้าง cache เรียบร้อย"

build-app:
  extends: .node-defaults
  stage: build
  needs: ["install-dependencies"]
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 day

unit-test:
  extends: .node-defaults
  stage: test
  needs: ["install-dependencies"]
  script:
    - npm run test:unit -- --reporters=default --reporters=jest-junit
  artifacts:
    when: always
    expire_in: 3 days
    reports:
      junit: junit.xml

lint:
  extends: .node-defaults
  stage: test
  needs: ["install-dependencies"]
  script:
    - npm run lint

package-release:
  stage: package
  needs:
    - job: build-app
      artifacts: true
    - job: unit-test
      artifacts: false
    - job: lint
      artifacts: false
  script:
    - mkdir -p release
    - cp -r dist/ release/
    - echo "$CI_COMMIT_SHORT_SHA" > release/VERSION
    - tar -czf release.tar.gz release/
  artifacts:
    paths:
      - release.tar.gz
    expire_in: 1 week

deploy-staging:
  stage: deploy
  needs: ["package-release"]
  script:
    - tar -xzf release.tar.gz
    - echo "กำลัง deploy ขึ้น staging"
    - ./deploy.sh staging release/
  environment:
    name: staging
    url: https://staging.my-app.internal
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'

deploy-production:
  stage: deploy
  needs: ["deploy-staging"]
  script:
    - tar -xzf release.tar.gz
    - echo "กำลัง deploy ขึ้น production"
    - ./deploy.sh production release/
  environment:
    name: production
    url: https://my-app.example.com
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
      when: manual
      allow_failure: false
    - when: never
```

### 710.5 แผนภาพ DAG ของ Pipeline ทั้งหมด

```
build                         test                          package            deploy
┌────────────────────┐       ┌──────────────┐
│ install-dependencies│──────▶│ unit-test    │──┐
│                     │       └──────────────┘   │
│                     │       ┌──────────────┐   │           ┌────────────────┐   ┌───────────────┐   ┌──────────────────┐
│                     │───────▶│ lint         │──┼──────────▶│ package-release │──▶│ deploy-staging │──▶│ deploy-production │
│                     │       └──────────────┘   │           └────────────────┘   └───────────────┘   └──────────────────┘
└──────────┬──────────┘                          │                                                       (when: manual)
           │           ┌──────────────┐          │
           └──────────▶│ build-app    │──────────┘
                        │ (artifacts)  │
                        └──────────────┘
```

### 710.6 ขั้นตอนลงมือทำจริง

1. เตรียมโปรเจกต์ Node.js ทดลอง (หรือใช้โปรเจกต์เดิมจาก Part 50 แล้วเพิ่ม `package.json` กับ script ทดสอบง่าย ๆ)
2. เขียนไฟล์ `.gitlab-ci.yml` ตามตัวอย่างเต็มด้านบน ปรับคำสั่งใน `script:` ให้ตรงกับ `package.json` จริงของคุณ
3. Commit และ push:

```bash
git add .gitlab-ci.yml package.json package-lock.json
git commit -m "เพิ่ม pipeline ขั้นสูงพร้อม cache, artifact และ DAG (needs)"
git push origin main
```

4. เปิด **CI/CD > Pipelines** สังเกตว่า `unit-test` และ `lint` เริ่มทำงานทันทีที่ `install-dependencies` เสร็จ โดยไม่ต้องรอ `build-app`
5. ตรวจสอบว่า `deploy-staging` รันอัตโนมัติ ส่วน `deploy-production` แสดงสถานะ **Manual** รอให้กด Run เอง
6. เปิดแท็บ **Tests** ในหน้า Pipeline Detail เพื่อดูผลลัพธ์จาก `reports.junit` ที่ตั้งไว้
7. ลองรัน pipeline ซ้ำอีกครั้งโดยไม่แก้ `package-lock.json` แล้วสังเกตว่า job ที่ `policy: pull` ใช้เวลาน้อยลงเพราะไม่ต้องอัปโหลด cache ซ้ำ

### 710.7 Checklist สรุปแบบฝึกหัด

- [ ] Pipeline มีครบ 4 stage คือ `build`, `test`, `package`, `deploy`
- [ ] มี `cache` ที่ใช้ `key` ตาม branch และแยก `policy: pull-push`/`pull` ถูกต้องตามหน้าที่ของแต่ละ job
- [ ] มี `artifacts.paths` ส่งไฟล์ build จาก stage `build` ไปยัง `package`
- [ ] มี `artifacts.reports.junit` และเห็นผลลัพธ์ test บนหน้า UI จริง
- [ ] มีอย่างน้อยหนึ่งจุดที่ `needs:` ทำให้ job ไม่ต้องรอทั้ง stage (เช่น `unit-test`/`lint` ไม่รอ `build-app`)
- [ ] ใช้ `extends` จาก hidden job (`.node-defaults`) เพื่อลดโค้ดซ้ำ
- [ ] `deploy-staging` มี `environment: name: staging` และรันอัตโนมัติ
- [ ] `deploy-production` มี `environment: name: production`, `when: manual` และ `needs: ["deploy-staging"]`

---

## สรุป Part 71

ใน Part นี้เราได้เรียนรู้ว่า:

1. Pipeline ระดับพื้นฐานจาก Part 50 ยังขาดหลายอย่างสำหรับใช้งานจริงในทีมงาน โดยเฉพาะเรื่อง cache, artifacts, ความเร็ว และ environment
2. **Stage** รันเรียงกันตามลำดับที่ประกาศไว้ ส่วน **Job** ในสาย stage เดียวกันรันขนานกันได้ตามจำนวน Runner ที่ว่าง และสามารถบังคับให้ job บางกลุ่มรันทีละตัวได้ด้วย `resource_group:`
3. **`needs:`** ทำให้ pipeline เป็น **DAG (Directed Acyclic Graph)** — job สามารถเริ่มทำงานได้ทันทีที่ job ที่ตัวเอง needs เสร็จ โดยไม่ต้องรอทั้ง stage ทำให้ pipeline เร็วขึ้นอย่างมีนัยสำคัญ
4. **Cache** เก็บ dependency ที่สร้างใหม่ได้เสมอ ควบคุมด้วย `key` (คงที่, ตาม branch, หรือตามไฟล์ lock), `paths`, และ `policy` (`pull`, `push`, `pull-push`) เพื่อลดเวลา pipeline
5. **Artifacts** ส่งต่อผลลัพธ์งานจริงระหว่าง job ควบคุมด้วย `paths`, `expire_in`, และ `reports` (`junit` สำหรับผลการทดสอบ, `coverage_report` สำหรับ code coverage) ที่ GitLab นำไปแสดงผลในหน้า UI โดยตรง
6. **`extends`** ลดโค้ดซ้ำซ้อนโดยดึงค่าจาก hidden job (job ที่ชื่อขึ้นต้นด้วยจุด) มาใช้ โดยต้องเข้าใจกฎการ merge ว่า `variables` ผสานแบบ key-by-key แต่ list อย่าง `script` จะถูกแทนที่ทั้งก้อน
7. **`include`** แยกไฟล์ `.gitlab-ci.yml` เป็นหลายไฟล์ย่อยได้ทั้งแบบ `local`, `project`, `remote`, และ `template` ช่วยให้จัดการ config ขนาดใหญ่ได้ง่ายขึ้น
8. **Environments** ผูก deploy job เข้ากับสภาพแวดล้อมที่ชัดเจน ทำให้เห็นประวัติ deployment, กด rollback ได้จาก UI และรองรับ Review App แบบไดนามิกต่อ Merge Request
9. **`when:`** มีค่าครบทั้ง `on_success`, `on_failure`, `always`, `manual`, `delayed`, `never` — ใช้ `manual` คู่กับ `environment` และ `rules` เพื่อควบคุมการ deploy ที่มีความเสี่ยงสูงอย่างปลอดภัย
10. ลงมือเขียน pipeline เต็มรูปแบบ 4 stage ที่รวมทุกเทคนิคเข้าด้วยกัน ทั้ง cache, artifact, DAG ผ่าน `needs`, `extends`, environment และ manual deploy

Part นี้ทำให้ pipeline ของคุณก้าวจากระดับ "ใช้งานได้" ไปสู่ระดับที่ใกล้เคียงกับที่ทีมวิศวกรมืออาชีพใช้งานจริงมากขึ้นอีกขั้นหนึ่งแล้ว แต่ยังมีอีกหลายเรื่องที่ต้องเรียนต่อ โดยเฉพาะเมื่อโปรเจกต์ขยายใหญ่ขึ้นจนต้องประสานงานข้ามหลาย repository พร้อมกัน หรือต้องการสร้าง job แบบไดนามิกที่ไม่รู้ล่วงหน้าว่าจะมีกี่ job — นั่นคือหัวข้อของ Part ถัดไป

**ต่อไป:** [Part 72: GitLab CI/CD: Multi-project Pipelines, Dynamic Pipelines](./part-072-gitlab-cicd-multi-project.md)
