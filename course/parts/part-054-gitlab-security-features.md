# Part 54: GitLab Security Features: SAST, Dependency Scanning เบื้องต้น

> **Step ในหลักสูตรนี้:** Step 531–540
> **เฟส:** 5 — เจาะลึก GitLab และ CI/CD เบื้องต้นของมัน
> **เป้าหมายของ Part นี้:** เข้าใจแนวคิด DevSecOps และปรัชญา "security ที่ฝังอยู่ในทุกขั้นตอนของ pipeline" ของ GitLab รู้จักฟีเจอร์ security หลักที่มากับ GitLab ได้แก่ SAST, Dependency Scanning, Secret Detection, Container Scanning และ License Compliance สามารถเปิดใช้งานแต่ละฟีเจอร์ผ่าน `.gitlab-ci.yml` ได้จริง อ่านผลลัพธ์จาก Security Dashboard และ Vulnerability Report เป็น และเข้าใจว่าฟีเจอร์ไหนอยู่ใน tier ไหน (Free/Premium/Ultimate) เพื่อวางแผนใช้งานในองค์กรได้อย่างเหมาะสม

---

## สารบัญของ Part นี้

- Step 531: DevSecOps คืออะไร แนวคิดของ GitLab เรื่อง Built-in Security
- Step 532: SAST (Static Application Security Testing) คืออะไร เปิดใช้งานอย่างไร
- Step 533: Dependency Scanning เบื้องต้น
- Step 534: Secret Detection — ป้องกันข้อมูลลับหลุดเข้า repo
- Step 535: Container Scanning สำหรับ Docker image
- Step 536: License Compliance เบื้องต้น
- Step 537: Security Dashboard — ภาพรวมความปลอดภัยของโปรเจกต์/กลุ่ม
- Step 538: Vulnerability Report และการจัดการ (Dismiss, Resolve, Create Issue)
- Step 539: ข้อจำกัดของแต่ละ Tier — Free vs Premium vs Ultimate
- Step 540: แบบฝึกหัด — เปิดใช้ SAST และ Secret Detection ในโปรเจกต์จริง

---

## Step 531: DevSecOps คืออะไร แนวคิดของ GitLab เรื่อง Built-in Security

### จาก DevOps สู่ DevSecOps

ใน Part ก่อน ๆ เราได้เห็นแล้วว่า **DevOps** คือแนวคิดที่รวมฝั่ง Development (Dev) กับ Operations (Ops) เข้าด้วยกัน เพื่อให้การส่งมอบซอฟต์แวร์เร็วขึ้น มีคุณภาพสูงขึ้น และมี feedback loop ที่สั้นลง

แต่ในโลกความเป็นจริง มีอีกฝ่ายหนึ่งที่มักถูกดึงเข้ามา "ทีหลัง" เสมอ นั่นคือฝ่าย **Security (Sec)** รูปแบบการทำงานแบบดั้งเดิมมักเป็นแบบนี้:

```
Developer เขียนโค้ด → ทดสอบ → Deploy ขึ้น Production
                                      │
                                      ▼
                        ทีม Security เข้ามาตรวจสอบทีหลัง
                        (บางทีหลังจาก deploy ไปแล้วหลายเดือน)
```

ปัญหาของรูปแบบนี้คือ:

1. **ค้นพบช่องโหว่ช้าเกินไป** — เมื่อโค้ดถูก deploy ไปนานแล้ว การแก้ไขช่องโหว่ (vulnerability) มีต้นทุนสูงกว่าการแก้ตั้งแต่ตอนเขียนโค้ดหลายเท่า (มีงานวิจัยระบุว่าการแก้บั๊กด้าน security ใน production แพงกว่าการแก้ตอนพัฒนาถึง 30–100 เท่า)
2. **กลายเป็นคอขวด (bottleneck)** — ทีม security ต้องมาตรวจทุกอย่างก่อนปล่อยจริง ทำให้กระบวนการ release ช้าลงมาก และมักถูกมองว่าเป็น "ทีมที่คอยขวางทาง"
3. **นักพัฒนาไม่รู้สึกเป็นเจ้าของความปลอดภัย** — เพราะคิดว่า "เดี๋ยวทีม security ตรวจให้เอง" ทำให้ไม่ได้ใส่ใจตั้งแต่ต้น

**DevSecOps** คือแนวคิดที่แก้ปัญหานี้ด้วยหลักการ:

> **"Shift Left" — ย้าย security ให้เข้ามาเร็วที่สุดเท่าที่จะทำได้ในวงจรการพัฒนา แทนที่จะรอไปตรวจตอนท้ายสุด**

```
Shift Left:

Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
  ▲      ▲       ▲       ▲
  └──────┴───────┴───────┘
   ตรวจสอบ security ตั้งแต่จุดเหล่านี้ ไม่ใช่รอถึงท้ายสุด
```

หลักการสำคัญของ DevSecOps มีดังนี้:

1. **Security เป็นความรับผิดชอบร่วมกันของทุกคน (Everyone owns security)** ไม่ใช่หน้าที่ของทีมใดทีมหนึ่งเพียงลำพัง
2. **Automate ทุกอย่างที่ทำได้** — การสแกนหาช่องโหว่ต้องเกิดขึ้นอัตโนมัติทุกครั้งที่มีการ commit หรือเปิด Merge Request ไม่ใช่พึ่งมนุษย์มานั่งตรวจด้วยมือ
3. **Feedback เร็วที่สุดเท่าที่จะทำได้** — นักพัฒนาควรเห็นผลสแกนความปลอดภัยตั้งแต่ตอนเปิด Merge Request ไม่ใช่ตอน deploy ไปแล้ว
4. **Security เป็นส่วนหนึ่งของ pipeline ไม่ใช่ gate แยกต่างหาก** — การสแกนควรรันเป็น job ปกติใน CI/CD pipeline เดียวกับ build และ test

### แนวคิด "Built-in Security" ของ GitLab

GitLab วางตำแหน่งตัวเองเป็น **"single DevOps platform"** ที่ครอบคลุมทุกขั้นตอนตั้งแต่ planning จนถึง monitoring ในเครื่องมือเดียว และ security คือหนึ่งในเสาหลักของแนวคิดนี้ แทนที่จะให้ผู้ใช้ไปหาเครื่องมือ security แยกต่างหาก (เช่น SonarQube, Snyk, Checkmarx) มาต่อเชื่อมเข้ากับ pipeline เอง GitLab ออกแบบให้ฟีเจอร์ security เป็น **"citizen" ของ `.gitlab-ci.yml` โดยตรง** ผ่านกลไกที่เรียกว่า **CI/CD Templates**

หลักการเบื้องหลังคือ:

| แนวคิดดั้งเดิม (Bolt-on Security) | แนวคิดของ GitLab (Built-in Security) |
|---|---|
| ต้องติดตั้งเครื่องมือ security แยกต่างหาก | เปิดใช้งานด้วยการ `include` template บรรทัดเดียวใน `.gitlab-ci.yml` |
| ผลสแกนอยู่คนละระบบ ต้องสลับไปมา | ผลสแกนแสดงในหน้า Merge Request, Pipeline และ Security Dashboard ของ GitLab เอง |
| ทีม security ต้องเรียนรู้เครื่องมือหลายตัว | ใช้ concept เดียวกัน (Vulnerability, Report, Dashboard) ทั่วทุกประเภทการสแกน |
| การตั้งค่ามักซับซ้อน ต้องเขียน config เยอะ | มี default configuration ที่ใช้งานได้ทันทีโดยแทบไม่ต้องปรับอะไร |

ฟีเจอร์ security หลักที่ GitLab มีให้ในรูปแบบ built-in ได้แก่:

1. **SAST (Static Application Security Testing)** — สแกนซอร์สโค้ดหาช่องโหว่โดยไม่ต้องรันโปรแกรมจริง
2. **Dependency Scanning** — ตรวจ library/package ที่โปรเจกต์ใช้ว่ามีช่องโหว่ที่รู้จักหรือไม่
3. **Secret Detection** — ตรวจจับข้อมูลลับ (API key, password, token) ที่หลุดเข้าไปใน commit
4. **Container Scanning** — ตรวจ Docker image ที่ build ขึ้นมาว่ามีช่องโหว่ในระดับ OS package หรือไม่
5. **License Compliance** — ตรวจสอบ license ของ dependency ว่าขัดกับนโยบายองค์กรหรือไม่
6. **DAST (Dynamic Application Security Testing)** — ทดสอบแอปพลิเคชันที่กำลังรันอยู่จริงจากภายนอก (จะเรียนละเอียดใน Part หลัง ๆ ของเฟส DevOps ขั้นสูง)
7. **Fuzz Testing** — ยิง input แปลก ๆ จำนวนมากเข้าไปในแอปเพื่อหา crash หรือพฤติกรรมผิดปกติ

ใน Part นี้เราจะโฟกัสที่ 5 ฟีเจอร์แรก ซึ่งเป็นฟีเจอร์ที่ใช้งานบ่อยที่สุดในการทำงานจริง และเหมาะกับผู้เริ่มต้นที่สุด

### ทำไม GitLab ถึงทำแบบนี้ได้ดี

เหตุผลสำคัญคือ GitLab ควบคุมทั้ง **source code**, **CI/CD pipeline**, และ **UI แสดงผล** ไว้ในระบบเดียวกันทั้งหมด ทำให้สามารถ:

- ให้ผลสแกนแสดงเป็น **widget บน Merge Request** ได้ทันที (เช่น "พบช่องโหว่ใหม่ 2 รายการจากการเปลี่ยนแปลงนี้")
- เชื่อมโยงผลสแกนเข้ากับ **Issue Tracker** ในตัว เพื่อสร้าง issue ติดตามช่องโหว่ได้โดยตรง
- รวมผลจากทุกโปรเจกต์เข้าเป็น **Security Dashboard** ระดับกลุ่ม (group) หรือองค์กรได้

นี่คือจุดขายสำคัญที่ทำให้หลายองค์กรเลือกใช้ GitLab แทนการต่อเครื่องมือหลายตัวเข้าด้วยกันเอง

---

## Step 532: SAST (Static Application Security Testing) คืออะไร เปิดใช้งานอย่างไร

### SAST คืออะไร

**SAST (Static Application Security Testing)** คือการวิเคราะห์ **ซอร์สโค้ด** ของแอปพลิเคชันเพื่อค้นหาช่องโหว่ด้านความปลอดภัย **โดยไม่ต้องรันโปรแกรมจริง** (จึงเรียกว่า "static" หรือ "อยู่นิ่ง" ตรงข้ามกับ DAST ที่ต้อง "รัน" แอปจริงถึงจะทดสอบได้)

ตัวอย่างช่องโหว่ที่ SAST มักตรวจพบ:

- **SQL Injection** — โค้ดที่นำ input ของผู้ใช้ไปต่อ SQL query ตรง ๆ โดยไม่ผ่านการ sanitize
- **Cross-Site Scripting (XSS)** — โค้ดที่นำ input ของผู้ใช้ไปแสดงผลบนหน้าเว็บโดยไม่ escape
- **Hardcoded Credentials** — การเขียน password หรือ API key ฝังตรงในโค้ด
- **Insecure Deserialization** — การแปลงข้อมูลจากภายนอกกลับเป็น object โดยไม่ตรวจสอบ
- **Path Traversal** — โค้ดที่ปล่อยให้ผู้ใช้ระบุ path ไฟล์ได้อย่างอิสระจนเข้าถึงไฟล์ที่ไม่ควรเข้าถึงได้
- **Use of weak cryptography** — การใช้ algorithm การเข้ารหัสที่ล้าสมัยหรือไม่ปลอดภัย (เช่น MD5 สำหรับ password)

### หลักการทำงานของ SAST

SAST ทำงานโดยการ **วิเคราะห์ syntax tree และ pattern ของโค้ด** (ไม่ใช่แค่ text matching ธรรมดา) เพื่อหา pattern ที่รู้กันว่าเสี่ยงต่อช่องโหว่ ตัวอย่างเช่น มันจะรู้ว่าโค้ดแบบนี้เสี่ยงต่อ SQL Injection:

```python
# โค้ดที่มีความเสี่ยง — SAST จะ flag บรรทัดนี้
query = "SELECT * FROM users WHERE username = '" + user_input + "'"
cursor.execute(query)
```

ในขณะที่โค้ดแบบนี้ปลอดภัยกว่าและจะไม่ถูก flag:

```python
# โค้ดที่ปลอดภัย — ใช้ parameterized query
query = "SELECT * FROM users WHERE username = %s"
cursor.execute(query, (user_input,))
```

### SAST ใน GitLab ทำงานอย่างไร

GitLab ไม่ได้เขียน security analyzer เองทั้งหมด แต่ใช้แนวทาง **รวม analyzer หลายตัวไว้ในกรอบเดียวกัน** โดยตั้งแต่ GitLab 15.4 เป็นต้นมา GitLab ได้รวม analyzer หลายภาษาเข้าเป็น **GitLab SAST engine ที่ขับเคลื่อนด้วย Semgrep** เป็นหลัก (มี analyzer เฉพาะทางอื่นประกอบสำหรับบางภาษา) ทำให้รองรับภาษาโปรแกรมมิ่งจำนวนมาก เช่น JavaScript/TypeScript, Python, Java, Go, Ruby, PHP, C/C++, C#, Kotlin, Scala เป็นต้น

จุดสำคัญคือ **คุณไม่ต้องเลือก analyzer เอง** — GitLab จะตรวจสอบภาษาที่ใช้ในโปรเจกต์อัตโนมัติ แล้วรัน analyzer ที่เหมาะสมให้เอง

### วิธีเปิดใช้งาน SAST

วิธีที่ง่ายที่สุดคือการ `include` template ที่ GitLab เตรียมไว้ให้ในไฟล์ `.gitlab-ci.yml`:

```yaml
include:
  - template: Security/SAST.gitlab-ci.yml
```

แค่นี้เอง! เมื่อ pipeline รันครั้งถัดไป GitLab จะเพิ่ม job ชื่อ `sast` (หรือ job ย่อยตามภาษาที่ตรวจพบ เช่น `semgrep-sast`) เข้าไปใน stage `test` โดยอัตโนมัติ

ตัวอย่างไฟล์ `.gitlab-ci.yml` ฉบับเต็มที่มี stage อื่นร่วมด้วย:

```yaml
stages:
  - build
  - test
  - deploy

include:
  - template: Security/SAST.gitlab-ci.yml

build-job:
  stage: build
  script:
    - echo "Building the application..."

unit-test-job:
  stage: test
  script:
    - echo "Running unit tests..."

deploy-job:
  stage: deploy
  script:
    - echo "Deploying..."
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

หลังจาก `sast` job รันเสร็จ มันจะสร้างไฟล์ผลลัพธ์ในรูปแบบ **GitLab SAST Report (JSON)** และแนบเป็น **artifact** ของ job นั้น ผลลัพธ์นี้จะถูก GitLab นำไปแสดงต่อในหลายที่ ได้แก่:

1. **หน้า Job/Pipeline** — ดูรายละเอียดช่องโหว่ที่พบได้โดยตรง
2. **หน้า Merge Request** (ถ้า tier รองรับ) — แสดง widget สรุปว่า MR นี้เพิ่มช่องโหว่ใหม่กี่รายการ
3. **Security Dashboard** (Ultimate เท่านั้น) — รวมผลจากทุก pipeline ไว้ในที่เดียว

### การปรับแต่ง SAST เพิ่มเติม

บางครั้งคุณอาจต้องการ:

**ปิดการสแกนบางไฟล์/โฟลเดอร์** ด้วยตัวแปร `SAST_EXCLUDED_PATHS`:

```yaml
include:
  - template: Security/SAST.gitlab-ci.yml

variables:
  SAST_EXCLUDED_PATHS: "spec, test, tests, tmp, node_modules, vendor"
```

**บังคับใช้ analyzer เฉพาะตัว** ด้วยตัวแปร `SAST_EXCLUDED_ANALYZERS` (เพื่อปิด analyzer ที่ไม่ต้องการ):

```yaml
variables:
  SAST_EXCLUDED_ANALYZERS: "eslint"
```

**กำหนดระดับความรุนแรงขั้นต่ำที่จะรายงาน** สามารถทำผ่าน security policy (จะกล่าวถึงในหลักสูตรช่วง Governance) แต่ในระดับพื้นฐาน ผลลัพธ์ทุกระดับ (Critical, High, Medium, Low, Info, Unknown) จะถูกรายงานทั้งหมดเสมอ การกรองระดับความรุนแรงมักทำที่ชั้น Security Dashboard หรือ Merge Request approval policy

### ข้อควรรู้สำคัญ

- SAST รันได้แม้ไม่มีการเชื่อมต่อกับ registry ภายนอกใด ๆ (ทำงาน offline ได้ถ้า mirror image analyzer ไว้ใน registry ภายใน)
- SAST **ไม่ได้รับประกันว่าจะจับช่องโหว่ได้ทุกตัว** — มันช่วยจับ pattern ที่รู้จักได้ดี แต่ตรรกะทางธุรกิจที่ซับซ้อน (business logic flaw) ยังต้องอาศัยการทำ manual code review หรือ penetration testing ควบคู่ไปด้วย
- ผลจาก SAST อาจมี **False Positive** (แจ้งเตือนทั้งที่จริง ๆ ไม่ใช่ช่องโหว่) ได้เสมอ ทีมพัฒนาต้องตรวจสอบและตัดสินใจ ไม่ใช่เชื่อผลสแกน 100% โดยไม่พิจารณา

---

## Step 533: Dependency Scanning เบื้องต้น

### ปัญหาที่ Dependency Scanning แก้

โปรเจกต์ซอฟต์แวร์สมัยใหม่แทบไม่มีใครเขียนโค้ดเองทั้งหมด 100% เราพึ่งพา **library/package จากภายนอก (dependency)** จำนวนมาก เช่น โปรเจกต์ Node.js เล็ก ๆ หนึ่งตัวอาจมี dependency (รวม transitive dependency) หลายร้อยถึงหลายพันตัว

ปัญหาคือ:

> **ต่อให้โค้ดที่คุณเขียนเองปลอดภัย 100% แต่ถ้า library ที่คุณใช้มีช่องโหว่ แอปของคุณก็มีช่องโหว่ไปด้วย**

ตัวอย่างที่โด่งดังระดับโลกคือกรณี **Log4Shell (CVE-2021-44228)** ในปี 2021 ที่ library logging ยอดนิยม (Log4j ของ Java) มีช่องโหว่ร้ายแรงจนสามารถรันโค้ดจากระยะไกลได้ (Remote Code Execution) ส่งผลกระทบต่อระบบนับล้านทั่วโลกที่ "แค่ใช้" library ตัวนี้เป็น dependency

**Dependency Scanning** คือฟีเจอร์ที่ตรวจสอบ dependency ทั้งหมดของโปรเจกต์ (ทั้งทางตรงและทางอ้อม) เทียบกับฐานข้อมูลช่องโหว่ที่รู้จักแล้ว (known vulnerability database) เพื่อแจ้งเตือนว่า "library เวอร์ชันนี้ที่คุณใช้อยู่ มีช่องโหว่ที่รู้จักแล้วนะ"

### หลักการทำงาน

1. Dependency Scanning จะอ่านไฟล์ manifest ของ package manager ที่โปรเจกต์ใช้ เช่น `package-lock.json` / `yarn.lock` (Node.js), `Gemfile.lock` (Ruby), `requirements.txt` / `poetry.lock` (Python), `pom.xml` (Java/Maven), `go.sum` (Go), `composer.lock` (PHP) เป็นต้น
2. นำรายชื่อ package และเวอร์ชันที่ใช้จริง ไปเทียบกับฐานข้อมูลช่องโหว่ (GitLab ดูแลฐานข้อมูล **GitLab Advisory Database** ของตัวเอง ซึ่งรวบรวมข้อมูลจากแหล่งสาธารณะ เช่น NVD — National Vulnerability Database และ advisory ของแต่ละ ecosystem)
3. รายงานออกมาว่าพบ dependency ที่มีช่องโหว่ตัวไหนบ้าง พร้อมระดับความรุนแรง (severity) และเวอร์ชันที่แก้ไขแล้ว (fixed version) ที่แนะนำให้ upgrade ไป

### วิธีเปิดใช้งาน Dependency Scanning

เช่นเดียวกับ SAST เปิดใช้งานผ่านการ include template:

```yaml
include:
  - template: Security/Dependency-Scanning.gitlab-ci.yml
```

เมื่อ pipeline รัน มันจะตรวจพบไฟล์ lock/manifest ที่มีอยู่ในโปรเจกต์อัตโนมัติ แล้วรัน analyzer ที่เหมาะสม (เช่น `gemnasium-dependency_scanning` สำหรับ ecosystem ส่วนใหญ่ หรือ analyzer เฉพาะสำหรับบางภาษา)

ตัวอย่างผลลัพธ์ในเชิงแนวคิด (จำลอง ไม่ใช่ format จริงเป๊ะ ๆ):

```
พบช่องโหว่ 3 รายการ:

1. [Critical] lodash@4.17.15
   CVE-2021-23337 — Prototype Pollution
   แนะนำ: อัปเกรดเป็น lodash@4.17.21 หรือใหม่กว่า

2. [High] axios@0.21.0
   CVE-2021-3749 — Regular Expression Denial of Service (ReDoS)
   แนะนำ: อัปเกรดเป็น axios@0.21.2 หรือใหม่กว่า

3. [Medium] minimist@1.2.0
   CVE-2020-7598 — Prototype Pollution
   แนะนำ: อัปเกรดเป็น minimist@1.2.3 หรือใหม่กว่า
```

### ความแตกต่างระหว่าง Dependency Scanning กับ SAST

จุดที่ผู้เริ่มต้นมักสับสน:

| | SAST | Dependency Scanning |
|---|---|---|
| ตรวจอะไร | โค้ดที่**คุณเขียนเอง** | Library/package ที่**คุณเรียกใช้จากภายนอก** |
| วิธีตรวจ | วิเคราะห์ pattern ของ syntax/logic | เทียบชื่อ+เวอร์ชัน package กับฐานข้อมูลช่องโหว่ |
| พบช่องโหว่แบบไหน | Bug เชิงตรรกะที่เขียนขึ้นเอง เช่น SQL Injection | ช่องโหว่ที่ "รู้จักแล้ว" (known CVE) ใน dependency |
| การแก้ไข | แก้โค้ดของตัวเอง | Upgrade version ของ dependency |

ทั้งสองฟีเจอร์**ทำงานเสริมกัน ไม่ใช่แทนที่กัน** โปรเจกต์ที่ดีควรเปิดทั้งคู่พร้อมกัน

### Dependency List (ฟีเจอร์ที่เกี่ยวข้อง)

เมื่อเปิด Dependency Scanning (หรือ generate SBOM) แล้ว GitLab จะสามารถแสดง **Dependency List** ซึ่งเป็นรายการ dependency ทั้งหมดของโปรเจกต์พร้อมเวอร์ชันและ license ที่หน้า **Secure > Dependency list** — มีประโยชน์มากตอนต้องทำ software inventory หรือตอบคำถามเช่น "โปรเจกต์เราใช้ library อะไรบ้างที่มี license แบบ GPL"

### ข้อควรระวัง

- Dependency Scanning ตรวจได้เฉพาะ **ช่องโหว่ที่มีคนค้นพบและรายงานเข้าฐานข้อมูลแล้วเท่านั้น** (known vulnerabilities) ถ้าเป็นช่องโหว่ใหม่ที่ยังไม่มีใครรู้ (zero-day) มันจะตรวจไม่พบ
- ต้องมีไฟล์ lock file ที่ถูกต้อง (เช่น `package-lock.json`) ไม่ใช่แค่ `package.json` เฉย ๆ เพราะ scanner ต้องรู้ **เวอร์ชันที่ resolve จริง** ไม่ใช่แค่ version range ที่ระบุไว้
- ควรรัน Dependency Scanning เป็นประจำ (ทุก pipeline หรืออย่างน้อยตาม schedule รายวัน/รายสัปดาห์) เพราะฐานข้อมูลช่องโหว่มีการอัปเดตใหม่ตลอดเวลา แม้โค้ดของคุณจะไม่เปลี่ยนเลย ก็อาจมีช่องโหว่ใหม่ถูกค้นพบใน dependency ที่คุณใช้อยู่ได้ทุกเมื่อ

---

## Step 534: Secret Detection — ป้องกันข้อมูลลับหลุดเข้า repo

### ปัญหาที่เกิดขึ้นบ่อยที่สุดในโลกจริง

ถ้าถามว่าเหตุการณ์ security incident แบบไหนที่พบบ่อยที่สุดในบริษัทซอฟต์แวร์ คำตอบหนึ่งที่ติดอันดับต้น ๆ เสมอคือ **"หลุด secret เข้า git repository โดยไม่ตั้งใจ"** เช่น:

- Developer เผลอ hardcode AWS Access Key ไว้ในโค้ดแล้ว commit ขึ้น GitHub/GitLab แบบ public
- ไฟล์ `.env` ที่มี database password หลุดเข้า repo เพราะลืมใส่ไว้ใน `.gitignore`
- API token ของบริการภายนอก (เช่น Stripe, SendGrid, OpenAI) ถูกฝังในโค้ดตัวอย่างหรือ config

ปัญหาของเรื่องนี้คือ **แม้จะลบออกจากโค้ดใน commit ล่าสุดแล้ว ข้อมูลลับนั้นก็ยังอยู่ใน git history ตลอดไป** (เพราะ Git เก็บทุก snapshot ตามที่เรียนใน Part 01) ใครก็ตามที่ clone repo แล้วไล่ดูประวัติก็จะเจอ secret นั้นได้อยู่ดี ต่อให้ commit ปัจจุบันจะไม่มีแล้วก็ตาม

### Secret Detection คืออะไร

**Secret Detection** คือฟีเจอร์ของ GitLab ที่สแกนหา pattern ของข้อมูลลับที่มีรูปแบบเฉพาะตัว (เช่น AWS key ขึ้นต้นด้วย `AKIA...`, GitHub token ขึ้นต้นด้วย `ghp_...`, private key ที่มี header `-----BEGIN PRIVATE KEY-----`) ทั้งใน **commit ปัจจุบัน** และสามารถสแกนย้อนไปใน **git history** ได้ด้วย

GitLab มาพร้อมกับ **rule set สำหรับตรวจจับ secret หลายร้อยรูปแบบ** ครอบคลุม cloud provider หลัก ๆ (AWS, GCP, Azure), payment gateway, บริการ SaaS ยอดนิยม, และ private key format ต่าง ๆ

### วิธีเปิดใช้งาน Secret Detection

```yaml
include:
  - template: Security/Secret-Detection.gitlab-ci.yml
```

เมื่อรัน pipeline job ชื่อ `secret_detection` จะถูกเพิ่มเข้ามาโดยอัตโนมัติ และจะสแกนเฉพาะ **commit ที่เปลี่ยนแปลงใน pipeline นั้น** (ไม่ใช่สแกนทั้ง history ทุกครั้งเพื่อความเร็ว) ถ้าต้องการสแกนย้อนหลังทั้ง history ต้องรันแบบพิเศษ (historical scan) ซึ่งมักทำครั้งเดียวตอนเริ่มนำ Secret Detection มาใช้กับโปรเจกต์เก่าที่มีอยู่แล้ว

ตัวอย่างผลลัพธ์เมื่อพบ secret ในโค้ด:

```yaml
# ไฟล์ config.py ที่ commit เข้ามา
DATABASE_PASSWORD = "SuperSecret123!"
AWS_SECRET_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

Secret Detection จะ flag ทั้งสองบรรทัดนี้ทันทีว่าเป็น potential secret พร้อมระบุประเภท (เช่น "AWS Secret Access Key") และตำแหน่งไฟล์/บรรทัดที่พบ

### Secret Push Protection (การป้องกันเชิงรุกก่อนหลุดเข้า repo)

นอกจาก Secret Detection แบบที่รันหลัง push (ผ่าน pipeline) แล้ว GitLab ยังมีฟีเจอร์ **Secret Push Protection** ที่ทำงานในระดับ **ก่อนที่ push จะสำเร็จด้วยซ้ำ** — เมื่อ developer พยายาม `git push` โค้ดที่มี pattern ของ secret อยู่ใน commit GitLab server จะ **ปฏิเสธ push นั้นทันที** พร้อมข้อความแจ้งเตือนว่าพบ secret ที่ตำแหน่งไหน ทำให้ secret ไม่มีโอกาสเข้าไปอยู่ใน git history เลยแม้แต่วินาทีเดียว

```
$ git push origin feature/add-payment
remote: GitLab: 
remote: Secret detected in commit abc1234:
remote:   config.py:12: AWS Secret Access Key
remote: 
remote: Push rejected. Please remove the secret and try again.
error: failed to push some refs to 'origin'
```

นี่คือแนวทางที่ปลอดภัยกว่ามาก เพราะเป็นการ **ป้องกันไม่ให้เกิดปัญหาตั้งแต่ต้นทาง** แทนที่จะปล่อยให้เกิดแล้วค่อยมาแก้ทีหลัง (ซึ่งการ "แก้" secret ที่หลุดเข้า git history ไปแล้วนั้นยุ่งยากมาก ต้องทำ history rewrite ด้วย `git filter-repo` หรือเครื่องมือคล้ายกัน แล้วยังต้อง **revoke/หมุน (rotate) secret ตัวนั้นทันที** เพราะถือว่ามันรั่วไหลไปแล้วไม่ว่าจะลบออกจาก repo ได้หรือไม่ก็ตาม)

### สิ่งที่ต้องทำถ้า Secret หลุดเข้า repo ไปแล้ว

ถ้าตรวจพบว่ามี secret หลุดเข้าไปแล้ว ขั้นตอนที่ถูกต้องคือ:

1. **Revoke/หมุน secret นั้นทันที** — เปลี่ยน password, สร้าง API key ใหม่, ปิดการใช้งาน token เก่า — ทำเป็นอันดับแรกสุดเสมอ เพราะต่อให้ลบออกจาก repo ก็ไม่ได้แปลว่าไม่มีใครเห็นมันไปแล้ว
2. **ลบ secret ออกจาก git history** — ใช้เครื่องมืออย่าง `git filter-repo` หรือ BFG Repo-Cleaner (ไม่ใช่แค่ลบออกจาก commit ล่าสุดด้วย `git rm`)
3. **สืบสวนว่ามีการใช้งาน secret นั้นโดยไม่ได้รับอนุญาตหรือไม่** ผ่าน access log ของบริการที่เกี่ยวข้อง
4. **ปรับกระบวนการ** เพื่อป้องกันไม่ให้เกิดซ้ำ เช่น เปิด Secret Push Protection, ใช้ secret manager (เช่น GitLab CI/CD Variables แบบ masked/protected, HashiCorp Vault) แทนการ hardcode

### แนวปฏิบัติที่ดี

- **ห้าม hardcode secret ในโค้ดเด็ดขาด** ใช้ **environment variable** หรือ **CI/CD Variables** ของ GitLab แทนเสมอ (ตั้งค่าที่ Settings > CI/CD > Variables พร้อมติ๊ก "Masked" และ "Protected")
- ใส่ไฟล์ที่มี secret เช่น `.env` ไว้ใน `.gitignore` ตั้งแต่วันแรกของโปรเจกต์
- เปิด Secret Detection และ Secret Push Protection ให้ทุกโปรเจกต์เป็นค่าเริ่มต้น ไม่ใช่แค่โปรเจกต์ที่ "คิดว่าสำคัญ"

---

## Step 535: Container Scanning สำหรับ Docker image

### ทำไมต้องสแกน Container Image

ในหลักสูตรช่วงหลังเราจะเรียนเรื่อง CI/CD ที่ build **Docker image** เพื่อนำไป deploy ปัญหาคือ Docker image ไม่ได้มีแค่โค้ดของเราเพียงอย่างเดียว มันยังประกอบด้วย:

```
┌─────────────────────────────────┐
│   Application Code (โค้ดของเรา)   │  ← SAST/Dependency Scanning ตรวจส่วนนี้
├─────────────────────────────────┤
│   Language Runtime               │  ← เช่น Node.js runtime, Python runtime
├─────────────────────────────────┤
│   OS Packages                    │  ← เช่น openssl, curl, libc ที่มากับ base image
├─────────────────────────────────┤
│   Base Image (เช่น ubuntu, alpine)│  ← Container Scanning ตรวจส่วนนี้เป็นหลัก
└─────────────────────────────────┘
```

แม้โค้ดแอปพลิเคชันของเราจะปลอดภัย 100% และ dependency ของภาษาที่เราเขียนก็ปลอดภัยดี แต่ถ้า **base image** (เช่น `ubuntu:20.04`) หรือ **OS-level package** ที่ติดมากับ image มีช่องโหว่ที่รู้จักแล้ว (เช่น ช่องโหว่ใน OpenSSL เวอร์ชันเก่า) container ที่ deploy ออกไปก็ยังมีความเสี่ยงอยู่ดี

**Container Scanning** คือฟีเจอร์ที่ตรวจ **layer ทั้งหมดของ Docker image** เพื่อหาช่องโหว่ที่รู้จักแล้วในระดับ OS package และ system library

### วิธีเปิดใช้งาน Container Scanning

Container Scanning ต้องใช้ควบคู่กับ job ที่ build และ push image ขึ้น container registry ก่อน (เพราะมันต้องมี image ที่ build เสร็จแล้วให้สแกน) ตัวอย่างการตั้งค่า:

```yaml
stages:
  - build
  - test

variables:
  CS_IMAGE: "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA"

build-image:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t $CS_IMAGE .
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker push $CS_IMAGE

include:
  - template: Security/Container-Scanning.gitlab-ci.yml
```

ตัวแปรสำคัญ `CS_IMAGE` บอก GitLab ว่า image ตัวไหนที่ต้องการให้สแกน (ถ้าไม่กำหนด GitLab จะพยายามเดาจาก `$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA` เป็นค่า default อยู่แล้ว)

### หลักการทำงานเบื้องหลัง

Container Scanning ของ GitLab ใช้ analyzer ที่อ้างอิงจากเครื่องมือสแกน image ที่ได้รับความนิยม (ตระกูล Trivy) โดยจะ:

1. ดึง image ที่ระบุมาวิเคราะห์ทีละ layer
2. แกะหา package manager metadata ในแต่ละ layer (เช่น dpkg สำหรับ Debian/Ubuntu, apk สำหรับ Alpine, rpm สำหรับ RHEL/CentOS)
3. เทียบรายชื่อ package + เวอร์ชัน กับฐานข้อมูลช่องโหว่ (CVE database ของแต่ละ distro)
4. รายงานช่องโหว่ที่พบพร้อมระดับความรุนแรงและเวอร์ชันที่แก้ไขแล้ว

### ตัวอย่างผลลัพธ์ (เชิงแนวคิด)

```
Image: registry.example.com/myapp:a1b2c3d
Base: ubuntu:20.04

พบช่องโหว่ 5 รายการ:

1. [Critical] libssl1.1 1.1.1f-1ubuntu2
   CVE-2022-0778 — Infinite loop in BN_mod_sqrt()
   แนะนำ: อัปเดต base image เป็นเวอร์ชันล่าสุด หรือ apt-get upgrade libssl1.1

2. [High] curl 7.68.0-1ubuntu2
   CVE-2021-22947 — TLS 1.3 session ticket...
   ...
```

### แนวทางแก้ไขเมื่อพบช่องโหว่ใน Container

ต่างจาก Dependency Scanning ที่แก้โดย upgrade version ใน lock file การแก้ปัญหาช่องโหว่ที่พบจาก Container Scanning มักทำได้หลายทาง:

1. **เปลี่ยน base image เป็นเวอร์ชันใหม่กว่า** เช่น จาก `ubuntu:20.04` เป็น `ubuntu:22.04` หรือ tag ล่าสุด
2. **ใช้ base image ที่เล็กและมี attack surface น้อยกว่า** เช่นเปลี่ยนจาก full OS image ไปเป็น **distroless image** หรือ **alpine** ที่มี package น้อยกว่ามาก จึงมีโอกาสมีช่องโหว่น้อยกว่าไปด้วย
3. **รัน `apt-get upgrade` หรือคำสั่งอัปเดต package ที่เกี่ยวข้องใน Dockerfile** ก่อน build
4. **Rebuild image ใหม่เป็นประจำ** แม้โค้ดแอปจะไม่เปลี่ยน เพราะฐานข้อมูล CVE มีการอัปเดตตลอดเวลา image เดิมที่เคย "ผ่าน" การสแกนเมื่อเดือนก่อน อาจมีช่องโหว่ใหม่ถูกค้นพบในสัปดาห์นี้ก็ได้

---

## Step 536: License Compliance เบื้องต้น

### ปัญหาด้าน License ที่มองข้ามได้ยาก

เวลาพูดถึง "security" ส่วนใหญ่คนจะนึกถึงช่องโหว่ทางเทคนิค แต่ยังมีความเสี่ยงอีกประเภทหนึ่งที่สำคัญไม่แพ้กันในเชิงธุรกิจและกฎหมาย นั่นคือ **ความเสี่ยงด้าน License ของ Open Source dependency**

ทุก library ที่เราใช้มี **license** กำกับอยู่เสมอ (เช่น MIT, Apache-2.0, GPL-3.0, BSD, AGPL) และ license แต่ละแบบมีเงื่อนไขการใช้งานที่ต่างกันมาก ตัวอย่างเช่น:

- **MIT / Apache-2.0 / BSD** — license แบบ "permissive" ใช้งานได้อิสระมาก แม้แต่ในซอฟต์แวร์เชิงพาณิชย์แบบ closed-source ก็ยังใช้ได้ (มีเงื่อนไขเพียงเล็กน้อย เช่น ต้องแนบ copyright notice)
- **GPL (GNU General Public License)** — license แบบ "copyleft" ที่มีเงื่อนไขเข้มว่า **ถ้าคุณนำโค้ดที่ใช้ GPL ไปรวมในผลิตภัณฑ์ของคุณ และแจกจ่ายผลิตภัณฑ์นั้น คุณอาจถูกบังคับให้ต้อง open source โค้ดทั้งหมดของคุณเองด้วย** (เรียกว่า "viral license")
- **AGPL (Affero GPL)** — เข้มกว่า GPL อีก โดยครอบคลุมถึงกรณีที่ให้บริการผ่านเครือข่าย (SaaS) ด้วย ไม่จำเป็นต้อง "แจกจ่าย" binary ก็ตาม

ลองจินตนาการว่าบริษัทของคุณกำลังพัฒนาผลิตภัณฑ์เชิงพาณิชย์แบบ closed-source แล้ว developer คนหนึ่งเผลอเพิ่ม library ที่ใช้ license แบบ AGPL เข้ามาโดยไม่รู้ตัว — นี่อาจกลายเป็นความเสี่ยงทางกฎหมายที่ร้ายแรงสำหรับบริษัทได้ทันที โดยที่ไม่มีใครรู้ตัวเลยจนกว่าจะสายเกินไป

### License Compliance คืออะไร

**License Compliance** (บางเอกสารเรียกว่า **License Scanning**) คือฟีเจอร์ของ GitLab ที่:

1. **สแกน dependency ทั้งหมดของโปรเจกต์** (ใช้กลไกคล้ายกับ Dependency Scanning) เพื่อดึงข้อมูลว่าแต่ละ library ใช้ license อะไร
2. **เทียบกับ policy ที่องค์กรกำหนดไว้** ว่า license แบบไหน "อนุญาต (allowed)" และแบบไหน "ห้าม (denied)"
3. **แจ้งเตือนหรือบล็อก Merge Request** ถ้ามีการเพิ่ม dependency ที่ใช้ license ที่ต้องห้ามเข้ามาในโปรเจกต์

### วิธีเปิดใช้งาน

License Compliance ใช้กลไกการสแกนร่วมกับ Dependency Scanning ในหลาย ecosystem ผ่าน template:

```yaml
include:
  - template: Security/License-Scanning.gitlab-ci.yml
```

หลังรัน pipeline ผลลัพธ์จะแสดงที่หน้า **Merge Request widget** และหน้า **Secure > License compliance** ของโปรเจกต์ ซึ่งจะแสดงรายชื่อ license ทั้งหมดที่พบพร้อมจำนวน dependency ที่ใช้ license แต่ละแบบ

### การตั้งนโยบาย License (License Approval Policy)

องค์กรสามารถกำหนดนโยบายได้ เช่น:

| License | นโยบาย |
|---|---|
| MIT | อนุญาต |
| Apache-2.0 | อนุญาต |
| BSD-3-Clause | อนุญาต |
| GPL-3.0 | ต้องขออนุมัติก่อน (denied by default) |
| AGPL-3.0 | ห้ามใช้เด็ดขาด (denied) |
| Unknown/ไม่ระบุ | ต้องตรวจสอบด้วยมือ |

เมื่อกำหนดนโยบายแบบนี้แล้ว หากมี Merge Request ที่เพิ่ม dependency ที่ใช้ license ต้องห้ามเข้ามา ระบบจะสามารถ**ขึ้น warning หรือบังคับให้ต้องมีการอนุมัติเพิ่มเติมก่อน merge ได้** (ฟีเจอร์การบังคับ approval แบบนี้อยู่ใน tier ระดับสูง ซึ่งจะกล่าวถึงในตารางสรุป tier ที่ Step 539)

### ประโยชน์ในเชิงธุรกิจ

- ป้องกันความเสี่ยงทางกฎหมายจากการใช้ license ที่ขัดต่อโมเดลธุรกิจ (เช่น ธุรกิจ closed-source ที่ดันมี GPL/AGPL ปนอยู่)
- ทำให้ทีม Legal/Compliance ตรวจสอบได้ง่ายขึ้นมากโดยไม่ต้องไล่เช็คเองทีละ dependency
- ช่วยในกระบวนการ **Open Source Software (OSS) audit** เวลาที่บริษัทต้องผ่านการตรวจสอบจากลูกค้าองค์กรใหญ่หรือก่อนการควบรวมกิจการ (M&A due diligence)

---

## Step 537: Security Dashboard — ภาพรวมความปลอดภัยของโปรเจกต์/กลุ่ม

### ปัญหาเมื่อมีหลายโปรเจกต์

เมื่อองค์กรมีหลายสิบหรือหลายร้อยโปรเจกต์ที่แต่ละอันเปิดใช้ SAST, Dependency Scanning, Container Scanning ไว้หมดแล้ว คำถามที่ตามมาคือ:

> "แล้วใครจะไล่ดูผลสแกนของทุกโปรเจกต์ทีละอันล่ะ?"

นี่คือปัญหาที่ **Security Dashboard** ถูกออกแบบมาแก้โดยเฉพาะ

### Security Dashboard คืออะไร

**Security Dashboard** คือหน้ารวมศูนย์ที่แสดงภาพรวมของช่องโหว่ (vulnerability) ที่พบทั้งหมด โดยมีให้ดูได้หลายระดับ:

1. **Project-level Security Dashboard** — ดูภาพรวมของโปรเจกต์เดียว
2. **Group-level Security Dashboard** — รวมผลจากทุกโปรเจกต์ใน group เดียวกัน เหมาะสำหรับหัวหน้าทีมหรือ security champion ที่ดูแลหลายโปรเจกต์พร้อมกัน
3. **Instance-level Security Overview** (สำหรับ self-managed) — ภาพรวมทั้งองค์กร

### สิ่งที่ Dashboard แสดง

Security Dashboard มักแสดงข้อมูลในรูปแบบ:

- **กราฟแนวโน้ม (trend)** ของจำนวนช่องโหว่ตามเวลา — ช่วยตอบคำถามว่า "ทีมเราแก้ช่องโหว่เร็วขึ้นหรือช้าลงเมื่อเทียบกับเดือนก่อน"
- **จำนวนช่องโหว่แยกตามระดับความรุนแรง** (Critical / High / Medium / Low)
- **จำนวนช่องโหว่แยกตามประเภทการสแกน** (SAST / Dependency Scanning / Container Scanning / Secret Detection)
- **รายชื่อโปรเจกต์ที่มีช่องโหว่ Critical ค้างอยู่มากที่สุด** — ช่วยให้จัดลำดับความสำคัญได้ว่าควรโฟกัสโปรเจกต์ไหนก่อน

ตัวอย่างมุมมองแบบง่าย:

```
Security Dashboard — Group: my-company

┌─────────────────────────────────────────────┐
│ ช่องโหว่ทั้งหมด: 142                          │
│   Critical: 8    High: 34   Medium: 67  Low: 33│
├─────────────────────────────────────────────┤
│ แนวโน้ม 30 วันที่ผ่านมา: ลดลง 12%              │
├─────────────────────────────────────────────┤
│ Top 5 โปรเจกต์ที่มี Critical มากที่สุด:        │
│  1. payment-service      (4 Critical)         │
│  2. auth-gateway         (2 Critical)         │
│  3. legacy-admin-panel   (2 Critical)         │
└─────────────────────────────────────────────┘
```

### การใช้งาน Dashboard ในเชิงกระบวนการ

Security Dashboard มักถูกใช้ในกระบวนการทำงานแบบนี้:

1. **Security Champion หรือหัวหน้าทีมเช็ค Dashboard เป็นประจำ** (เช่น ทุกสัปดาห์) แทนที่จะต้องไล่เข้าไปดูทีละโปรเจกต์
2. ระบุโปรเจกต์ที่มีความเสี่ยงสูงที่ต้องรีบจัดการก่อน
3. มอบหมายงานแก้ไขให้ทีมที่เกี่ยวข้อง ผ่านการสร้าง Issue โดยตรงจากช่องโหว่ที่พบ (จะกล่าวถึงใน Step ถัดไป)
4. ติดตามว่าช่องโหว่ที่มอบหมายไปแล้วถูกแก้ไข (resolved) หรือยังค้างอยู่

### ข้อจำกัดสำคัญ

Security Dashboard (ทั้งระดับ Project และ Group) เป็นฟีเจอร์ที่ต้องใช้ **GitLab Ultimate tier** เท่านั้น ผู้ใช้ Free/Premium จะยังคง**รัน**การสแกนได้ตามปกติ (SAST, Dependency Scanning ฯลฯ) และเห็นผลลัพธ์แบบ raw ในหน้า job/pipeline artifact ได้ แต่จะไม่มีหน้า Dashboard ที่รวมสรุปภาพรวมให้ดูอย่างสวยงามพร้อมกราฟแนวโน้ม เราจะสรุปรายละเอียดเรื่อง tier ทั้งหมดให้ชัดเจนใน Step 539

---

## Step 538: Vulnerability Report และการจัดการ (Dismiss, Resolve, Create Issue)

### Vulnerability Report คืออะไร

ถ้า Security Dashboard คือ "ภาพรวมระดับสูง" **Vulnerability Report** คือ **รายการช่องโหว่แบบละเอียดของแต่ละโปรเจกต์** ที่คุณสามารถกดเข้าไปดู คลิกเปิดแต่ละรายการ และ**จัดการ (manage)** สถานะของมันได้โดยตรง

หน้านี้อยู่ที่ **Secure > Vulnerability report** ของแต่ละโปรเจกต์ (หรือระดับ group) และแสดงรายการช่องโหว่ทั้งหมดที่ตรวจพบจากทุกประเภทการสแกน (SAST, Dependency Scanning, Container Scanning, Secret Detection ฯลฯ) รวมไว้ในตารางเดียว

### ข้อมูลที่แสดงต่อรายการช่องโหว่

เมื่อคลิกเข้าไปดูช่องโหว่แต่ละรายการ จะเห็นรายละเอียดครบถ้วน เช่น:

- **ชื่อและคำอธิบายช่องโหว่** (เช่น "SQL Injection", "CVE-2021-23337")
- **ระดับความรุนแรง (Severity)**: Critical, High, Medium, Low, Info, Unknown
- **สถานะ (Status)**: Detected, Confirmed, Dismissed, Resolved
- **ตำแหน่งที่พบ**: ไฟล์และบรรทัดที่แน่นอน (สำหรับ SAST) หรือชื่อ package และเวอร์ชัน (สำหรับ Dependency/Container Scanning)
- **Scanner ที่ตรวจพบ**: SAST, Dependency Scanning, Container Scanning เป็นต้น
- **Identifier**: เช่น CVE ID, CWE ID เพื่อใช้ค้นข้อมูลเพิ่มเติมจากแหล่งภายนอก
- **คำแนะนำการแก้ไข (Solution/Remediation)**: ถ้ามี เช่น "อัปเกรดเป็นเวอร์ชัน X.Y.Z"

### สถานะของ Vulnerability และการจัดการ

Vulnerability แต่ละรายการมีสถานะ (status) ที่สามารถเปลี่ยนได้ดังนี้:

| สถานะ | ความหมาย |
|---|---|
| **Detected** | สถานะเริ่มต้นเมื่อพบครั้งแรก ยังไม่มีใครตรวจสอบ |
| **Confirmed** | มีคนตรวจสอบแล้วและยืนยันว่าเป็นช่องโหว่จริง รอการแก้ไข |
| **Dismissed** | ถูกปัดตกไป เพราะพิจารณาแล้วว่าไม่เกี่ยวข้อง/ไม่ใช่ปัญหาจริง (false positive) หรือยอมรับความเสี่ยงนั้น |
| **Resolved** | ได้รับการแก้ไขแล้ว (เช่น อัปเกรด dependency เรียบร้อย และช่องโหว่หายไปในการสแกนรอบถัดไป) |

### การ Dismiss (ปัดตกช่องโหว่)

เมื่อทีมพิจารณาแล้วว่าช่องโหว่ที่พบไม่ใช่ปัญหาจริง (False Positive) หรือยอมรับความเสี่ยงนั้นได้ (Risk Accepted) สามารถกด **Dismiss** พร้อมเลือกเหตุผลและใส่ comment อธิบายได้ เช่น:

```
เหตุผลการ Dismiss:
○ False Positive — ไม่ใช่ช่องโหว่จริง
○ Acceptable Risk — รับความเสี่ยงนี้ได้ในบริบทนี้
○ Not Applicable — โค้ดส่วนนี้ไม่ได้ถูกเรียกใช้งานจริง (dead code)
○ Mitigating Control in Place — มีมาตรการป้องกันอื่นชดเชยอยู่แล้ว

Comment: "Endpoint นี้ใช้ภายใน internal network เท่านั้น ไม่ได้เปิดสู่ public 
และมี WAF กรองอยู่ชั้นหน้าแล้ว จึงรับความเสี่ยงนี้"
```

การใส่เหตุผลที่ชัดเจนสำคัญมาก เพราะเมื่อมีคนอื่น (หรือ auditor) มาตรวจสอบภายหลัง จะได้เข้าใจว่าทำไมช่องโหว่นี้ถึงถูกปัดตกไป ไม่ใช่แค่ถูกมองข้าม

### การสร้าง Issue จาก Vulnerability

จุดแข็งสำคัญของ GitLab คือ**ทุกอย่างอยู่ในระบบเดียว** เมื่อพบช่องโหว่ที่ต้องแก้ไขจริงจัง สามารถกดปุ่ม **"Create issue"** จากหน้ารายละเอียดช่องโหว่ได้เลย ระบบจะสร้าง Issue ใหม่ให้อัตโนมัติ พร้อม:

- ชื่อ Issue ที่อ้างอิงช่องโหว่นั้นโดยตรง
- คำอธิบายที่ดึงรายละเอียดทางเทคนิคของช่องโหว่มาใส่ให้อัตโนมัติ (severity, location, identifier)
- ลิงก์ย้อนกลับไปยังหน้า Vulnerability ต้นทาง เพื่อให้ทีมพัฒนาติดตามและปิดงานได้ตรงจุด

จากนั้นสามารถ assign Issue ให้ทีมที่รับผิดชอบ ใส่ label, milestone ตามกระบวนการทำงานปกติของทีม (เหมือน Issue ทั่วไปที่เราเรียนมาใน Part ก่อน ๆ) และเมื่อ Merge Request ที่แก้ไขช่องโหว่นั้นถูก merge แล้ว การสแกนรอบถัดไปจะไม่พบช่องโหว่นั้นอีก ทำให้สถานะ vulnerability เปลี่ยนเป็น **Resolved** โดยอัตโนมัติ

### ทำไมการจัดการ Vulnerability ถึงสำคัญพอ ๆ กับการตรวจพบ

การเปิดใช้ SAST/Dependency Scanning เป็นแค่ครึ่งแรกของงาน หลายทีมทำผิดพลาดด้วยการเปิดสแกนไว้ แต่**ไม่มีกระบวนการติดตามผล** ทำให้ช่องโหว่ที่พบเจอสะสมเป็นพัน ๆ รายการโดยไม่มีใครจัดการ จนสุดท้ายทุกคนเพิกเฉยต่อผลสแกนทั้งหมด (เรียกว่า "alert fatigue")

กระบวนการที่ดีควรมี:

1. **นโยบายชัดเจน**ว่าช่องโหว่ระดับไหนต้องแก้ภายในกี่วัน (เช่น Critical ต้องแก้ภายใน 7 วัน, High ภายใน 30 วัน)
2. **Assign เจ้าของ** ให้ทุกช่องโหว่ที่ confirmed แล้ว ไม่ปล่อยลอยไม่มีใครรับผิดชอบ
3. **Review Dashboard เป็นประจำ** เพื่อติดตามว่ากระบวนการทำงานจริงหรือไม่
4. **ทบทวน Dismissed vulnerabilities เป็นระยะ** เผื่อบริบทเปลี่ยนไปแล้ว (เช่น endpoint ที่เคยเป็น internal-only อาจถูกเปิดสู่ public ในภายหลัง)

---

## Step 539: ข้อจำกัดของแต่ละ Tier — Free vs Premium vs Ultimate

### ทำไมต้องรู้เรื่อง Tier

ฟีเจอร์ security ที่เรียนมาทั้งหมดใน Part นี้ **ไม่ได้เปิดให้ใช้เท่ากันทุก tier** การเข้าใจว่าฟีเจอร์ไหนอยู่ tier ไหนสำคัญมากสำหรับ:

- การวางแผนงบประมาณเวลาต้องเสนอซื้อ license ให้องค์กร
- การไม่เสียเวลาตั้งค่าฟีเจอร์ที่ instance ของคุณไม่มีสิทธิ์ใช้
- การเข้าใจว่าทำไมบางฟีเจอร์ในเอกสารถึง "รันได้แต่มองไม่เห็นผลสวย ๆ" ในบัญชีของคุณ

> **หมายเหตุสำคัญ:** GitLab ปรับเปลี่ยนขอบเขตของแต่ละ tier อยู่เป็นระยะตามการพัฒนาผลิตภัณฑ์ ตารางด้านล่างนี้สรุปภาพรวมโครงสร้างหลักที่ค่อนข้างเสถียรมานาน แต่ควรตรวจสอบหน้า pricing/feature comparison อย่างเป็นทางการของ GitLab เสมอ ก่อนตัดสินใจเชิงธุรกิจ เพราะรายละเอียดปลีกย่อยอาจเปลี่ยนแปลงไปตามเวอร์ชัน

### ภาพรวม 3 Tier ของ GitLab

| Tier | กลุ่มเป้าหมาย |
|---|---|
| **Free** | นักพัฒนารายบุคคล ทีมเล็ก โปรเจกต์ Open Source |
| **Premium** | ทีมขนาดกลางถึงใหญ่ที่ต้องการฟีเจอร์ด้าน collaboration/workflow ขั้นสูง (เช่น Code Owners ขั้นสูง, Multiple approval rules, Merge trains) |
| **Ultimate** | องค์กรที่ต้องการ Security และ Compliance ครบวงจร รวมถึง Portfolio/Value stream management |

### ตารางสรุปฟีเจอร์ Security ตาม Tier

| ฟีเจอร์ | Free | Premium | Ultimate |
|---|:---:|:---:|:---:|
| **รัน SAST ใน pipeline** | ✅ | ✅ | ✅ |
| **รัน Secret Detection ใน pipeline** | ✅ | ✅ | ✅ |
| **Secret Push Protection** (บล็อกก่อน push) | ✅ | ✅ | ✅ |
| **รัน Dependency Scanning** | ❌ | ❌ | ✅ |
| **รัน Container Scanning** | ❌ | ❌ | ✅ |
| **รัน License Compliance / License Scanning** | ❌ | ❌ | ✅ |
| **รัน DAST** | ❌ | ❌ | ✅ |
| **รัน Fuzz Testing** | ❌ | ❌ | ✅ |
| **ดูผลสแกนแบบ raw ใน Job Artifact (JSON)** | ✅ | ✅ | ✅ |
| **Merge Request Security Widget** (สรุปช่องโหว่ใหม่ใน MR) | ❌ | ❌ | ✅ |
| **Vulnerability Report** (จัดการ dismiss/resolve/create issue) | ❌ | ❌ | ✅ |
| **Security Dashboard** (Project/Group level) | ❌ | ❌ | ✅ |
| **Dependency List** | ❌ | ❌ | ✅ |
| **Security Policies** (Scan Execution Policy, Merge Request Approval Policy) | ❌ | ❌ | ✅ |
| **Compliance Framework / Compliance Center** | ❌ | บางส่วน | ✅ |

### อธิบายเพิ่มเติมในประเด็นที่มักเข้าใจผิด

**1. SAST และ Secret Detection ใช้ได้ใน Free tier จริง**

นี่คือจุดที่คนมักแปลกใจ — GitLab เปิดให้ **SAST** และ **Secret Detection** (การรันสแกน) ใช้งานได้ตั้งแต่ tier Free เพราะ GitLab มองว่าช่องโหว่ในโค้ดของตัวเองกับข้อมูลลับที่หลุด เป็นความเสี่ยงพื้นฐานที่สุดที่ทุกโปรเจกต์ควรป้องกันได้โดยไม่มีค่าใช้จ่าย นี่เป็นส่วนหนึ่งของกลยุทธ์ที่ GitLab ต้องการให้ "security เป็นเรื่องพื้นฐานที่ทุกคนเข้าถึงได้" (democratizing security)

**2. "รันได้" กับ "จัดการได้" เป็นคนละเรื่องกัน**

แม้ tier Free จะรัน SAST/Secret Detection ได้ แต่**ไม่มีหน้า Vulnerability Report ที่ให้ dismiss/resolve/create issue ได้อย่างเป็นระบบ** ผู้ใช้ Free tier จะเห็นผลลัพธ์เป็น**ไฟล์ JSON ใน job artifact เท่านั้น** ต้องเปิดไฟล์เองหรือเขียนสคริปต์แปลงผลเอาเอง ถ้าต้องการ UI ที่จัดการช่องโหว่ได้อย่างเป็นระบบ (ดู severity, dismiss, สร้าง issue, ติดตามสถานะ) ต้องใช้ **Ultimate**

**3. Dependency Scanning, Container Scanning, License Compliance เป็น Ultimate ทั้งหมด**

ต่างจาก SAST/Secret Detection ฟีเจอร์กลุ่มนี้ (ที่ต้องพึ่งฐานข้อมูลช่องโหว่ที่ GitLab ดูแลและอัปเดตต่อเนื่อง เช่น GitLab Advisory Database) จัดอยู่ใน **Ultimate tier** เท่านั้น ทั้งการรันสแกนและการดูผลลัพธ์

**4. Premium ไม่ได้เพิ่มฟีเจอร์ security หลักมากนัก**

Premium tier เน้นเพิ่มฟีเจอร์ด้าน **workflow และ collaboration** เป็นหลัก เช่น Multiple approval rules สำหรับ Merge Request, Code Owners ขั้นสูง, Merge Trains, Epics ระดับกลุ่ม ฯลฯ ไม่ใช่ฟีเจอร์ security แบบ deep scanning การจะได้ฟีเจอร์ security แบบเต็มรูปแบบต้องกระโดดไปที่ Ultimate โดยตรง

**5. Self-managed vs SaaS (GitLab.com)**

ไม่ว่าจะใช้ GitLab แบบ **SaaS** (GitLab.com) หรือ **self-managed** (ติดตั้งเองบน server องค์กร) โครงสร้าง tier และฟีเจอร์ security จะเหมือนกัน ความแตกต่างหลักคือเรื่องการจัดการโครงสร้างพื้นฐาน (infrastructure) ไม่ใช่ตัวฟีเจอร์ security เอง

### ตารางสรุปแบบเข้าใจง่ายที่สุด

| คำถาม | คำตอบ |
|---|---|
| อยากรู้ว่าโค้ดของฉันมีช่องโหว่ SQL Injection ไหม ฟรีไหม | **ได้ฟรี** (SAST อยู่ Free tier) |
| อยากรู้ว่ามี API key หลุดเข้า repo ไหม ฟรีไหม | **ได้ฟรี** (Secret Detection อยู่ Free tier) |
| อยากรู้ว่า library ที่ใช้มีช่องโหว่ที่รู้จักแล้วไหม ฟรีไหม | **ไม่ฟรี ต้อง Ultimate** (Dependency Scanning) |
| อยากรู้ว่า Docker image มีช่องโหว่ในระดับ OS ไหม ฟรีไหม | **ไม่ฟรี ต้อง Ultimate** (Container Scanning) |
| อยากได้หน้า Dashboard สวย ๆ ดูภาพรวมทุกโปรเจกต์ ฟรีไหม | **ไม่ฟรี ต้อง Ultimate** |
| อยากกด Dismiss/Resolve ช่องโหว่อย่างเป็นระบบ ฟรีไหม | **ไม่ฟรี ต้อง Ultimate** |

---

## Step 540: แบบฝึกหัด — เปิดใช้ SAST และ Secret Detection ในโปรเจกต์จริง

ถึงเวลาลงมือทำจริงแล้ว! แบบฝึกหัดนี้จะให้คุณเปิดใช้งาน **SAST** และ **Secret Detection** ในโปรเจกต์จริง เพราะทั้งสองฟีเจอร์นี้ใช้ได้ฟรีในทุก tier รวมถึง Free — ทุกคนที่เรียนหลักสูตรนี้จึงทำตามได้แน่นอนโดยไม่ต้องมี license ระดับ Ultimate

### ขั้นตอนที่ 1: เตรียมโปรเจกต์ทดสอบ

ถ้ายังไม่มีโปรเจกต์ทดสอบจาก Part ก่อน ๆ ให้สร้างโปรเจกต์ใหม่บน GitLab:

1. ไปที่ GitLab แล้วสร้าง **New project** ชื่อ `security-scanning-lab`
2. Clone มาที่เครื่อง:

```bash
git clone https://gitlab.com/<your-username>/security-scanning-lab.git
cd security-scanning-lab
```

### ขั้นตอนที่ 2: สร้างโค้ดตัวอย่างที่ตั้งใจให้มีช่องโหว่ (เพื่อการเรียนรู้เท่านั้น)

สร้างไฟล์ `app.py` ที่จงใจมีโค้ดไม่ปลอดภัย เพื่อดูว่า SAST จะจับได้จริงหรือไม่:

```python
import sqlite3

def get_user(username):
    conn = sqlite3.connect("app.db")
    cursor = conn.cursor()
    # ช่องโหว่ตั้งใจ: SQL Injection จากการต่อ string ตรง ๆ
    query = "SELECT * FROM users WHERE username = '" + username + "'"
    cursor.execute(query)
    return cursor.fetchall()

# ช่องโหว่ตั้งใจ: hardcoded credential
DATABASE_PASSWORD = "MySecretPass123!"
API_TOKEN = "sk_live_<REPLACE_WITH_REAL_KEY_EXAMPLE_ONLY>"

if __name__ == "__main__":
    print(get_user("admin"))
```

> **คำเตือน:** โค้ดข้างต้นเขียนขึ้นเพื่อการฝึกฝนในบทเรียนนี้เท่านั้น ห้ามเขียนโค้ดแบบนี้ในโปรเจกต์จริงเด็ดขาด

### ขั้นตอนที่ 3: สร้างไฟล์ `.gitlab-ci.yml`

สร้างไฟล์ `.gitlab-ci.yml` ที่ root ของโปรเจกต์:

```yaml
stages:
  - test

include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml

variables:
  SAST_EXCLUDED_PATHS: "spec, test, tests, tmp"
```

### ขั้นตอนที่ 4: Commit และ Push

```bash
git add app.py .gitlab-ci.yml
git commit -m "เพิ่ม SAST และ Secret Detection ให้โปรเจกต์"
git push origin main
```

> **สังเกต:** ถ้าคุณเปิด **Secret Push Protection** ไว้ในโปรเจกต์นี้แล้ว การ push ครั้งนี้อาจถูก**ปฏิเสธทันที** เพราะไฟล์ `app.py` มี pattern ที่คล้าย API token อยู่ (`API_TOKEN = "sk_live_..."`) นี่คือพฤติกรรมที่ถูกต้องและเป็นตัวอย่างที่ดีว่า Secret Push Protection ทำงานจริง หากเจอกรณีนี้ ให้ลองพิจารณาว่าในโลกจริงคุณจะต้องแก้ไขโค้ดก่อน push ไม่ใช่ฝืน push ผ่านไป (ถ้าต้องการทดสอบต่อให้ครบ ให้เปลี่ยนค่าตัวอย่างให้ไม่ตรง pattern จริงมากเกินไป หรือปิด Secret Push Protection ชั่วคราวเฉพาะในแบบฝึกหัดนี้)

### ขั้นตอนที่ 5: ดูผลลัพธ์ที่ CI/CD > Pipelines

1. ไปที่เมนู **Build > Pipelines** ของโปรเจกต์
2. คลิกเข้าไปที่ pipeline ล่าสุดที่รันอยู่
3. คุณควรเห็น job ชื่อประมาณ `semgrep-sast` (จาก SAST) และ `secret_detection` (จาก Secret Detection) อยู่ใน stage `test`
4. รอจน job ทั้งสองรันเสร็จ (สถานะเปลี่ยนเป็นสีเขียว หรือสีเหลืองถ้ามี warning)

### ขั้นตอนที่ 6: อ่านผลลัพธ์การสแกน

**วิธีที่ 1 — ดูผ่านหน้า Job:**

คลิกเข้าไปที่ job `semgrep-sast` แล้วดู log ของมัน จะเห็นข้อความสรุปประมาณนี้:

```
[INFO] Scan completed successfully.
[INFO] Found 2 vulnerabilities.
Uploading artifacts...
gl-sast-report.json: found 1 file
```

**วิธีที่ 2 — ดาวน์โหลด Artifact:**

ที่หน้า job จะมีปุ่ม **Download** สำหรับดาวน์โหลด artifact ชื่อ `gl-sast-report.json` และ `gl-secret-detection-report.json` เปิดไฟล์นี้ดูจะเห็นโครงสร้างประมาณนี้:

```json
{
  "vulnerabilities": [
    {
      "id": "abc123",
      "category": "sast",
      "name": "SQL Injection",
      "severity": "High",
      "location": {
        "file": "app.py",
        "start_line": 7
      },
      "identifiers": [
        { "type": "cwe", "value": "CWE-89", "name": "CWE-89" }
      ]
    }
  ]
}
```

**วิธีที่ 3 — ดูผ่าน Merge Request (ถ้าเปิดผ่าน MR แทนที่จะ push ตรงเข้า main):**

ถ้าคุณสร้าง branch ใหม่แล้วเปิด Merge Request แทนที่จะ push ตรงเข้า `main` ทันที คุณจะเห็นผลลัพธ์ในรูปแบบที่อ่านง่ายกว่ามาก โดยเฉพาะถ้าใช้ Ultimate tier จะเห็น widget สรุปในหน้า MR โดยตรง แนะนำให้ลองทำแบบนี้เพื่อฝึก workflow ที่ถูกต้องตามหลัก DevSecOps:

```bash
git checkout -b feature/add-security-scanning
# แก้ไข app.py และ .gitlab-ci.yml ตามที่ทำด้านบน
git add .
git commit -m "เพิ่ม security scanning"
git push origin feature/add-security-scanning
```

จากนั้นเปิด Merge Request จาก branch นี้ไปยัง `main` ผ่านหน้าเว็บ GitLab ตามที่เรียนมาใน Part ก่อนหน้า

### ขั้นตอนที่ 7: ทดลองแก้ไขช่องโหว่แล้วรันใหม่

แก้ไข `app.py` ให้ปลอดภัยขึ้นตามหลักที่เรียนใน Step 532:

```python
import sqlite3
import os

def get_user(username):
    conn = sqlite3.connect("app.db")
    cursor = conn.cursor()
    # แก้ไขแล้ว: ใช้ parameterized query
    query = "SELECT * FROM users WHERE username = ?"
    cursor.execute(query, (username,))
    return cursor.fetchall()

# แก้ไขแล้ว: ดึงค่าจาก environment variable แทนการ hardcode
DATABASE_PASSWORD = os.environ.get("DATABASE_PASSWORD")
API_TOKEN = os.environ.get("API_TOKEN")

if __name__ == "__main__":
    print(get_user("admin"))
```

Commit และ push อีกครั้ง แล้วสังเกตว่า pipeline รอบใหม่ควรพบช่องโหว่**น้อยลง**หรือ**ไม่พบเลย** เมื่อเทียบกับรอบแรก นี่คือการยืนยันว่า SAST และ Secret Detection ทำงานได้จริงและตอบสนองต่อการแก้ไขโค้ดของคุณ

### สรุปสิ่งที่ควรได้จากแบบฝึกหัดนี้

หลังทำแบบฝึกหัดนี้เสร็จ คุณควรจะ:

- เห็นด้วยตาตัวเองว่าการเปิดใช้ security scanning ใน GitLab ทำได้ง่ายเพียง `include` template ไม่กี่บรรทัด
- เข้าใจว่าผลลัพธ์การสแกนอยู่ที่ไหนบ้าง (job log, artifact JSON, MR widget)
- ได้สัมผัสประสบการณ์จริงว่า Secret Push Protection ทำงานอย่างไรเมื่อพยายาม push ข้อมูลลับ
- เข้าใจ workflow แบบ DevSecOps จริง ๆ: เขียนโค้ด → พบช่องโหว่จาก pipeline → แก้ไข → พิสูจน์ว่าช่องโหว่หายไปแล้ว — ทั้งหมดนี้เกิดขึ้น**ก่อน**ที่โค้ดจะถูก merge เข้า main ด้วยซ้ำ ตรงตามหลักการ "Shift Left" ที่เรียนใน Step 531

---

## สรุป Part 54

ใน Part นี้เราได้เรียนรู้ว่า:

1. **DevSecOps** คือแนวคิดที่ย้าย security ให้เข้ามาเร็วที่สุดในวงจรพัฒนา (Shift Left) แทนที่จะตรวจสอบทีหลังตอนใกล้ deploy และ GitLab ออกแบบให้ security เป็น built-in feature ของ pipeline โดยตรง ผ่านการ `include` CI/CD template
2. **SAST** สแกนหาช่องโหว่ในซอร์สโค้ดที่เราเขียนเอง (เช่น SQL Injection, XSS) โดยไม่ต้องรันโปรแกรมจริง เปิดใช้งานง่ายด้วย `Security/SAST.gitlab-ci.yml` และใช้ได้ฟรีในทุก tier
3. **Dependency Scanning** ตรวจ library ภายนอกที่โปรเจกต์พึ่งพา เทียบกับฐานข้อมูลช่องโหว่ที่รู้จักแล้ว (known CVE) ต่างจาก SAST ตรงที่ตรวจโค้ดของคนอื่น ไม่ใช่โค้ดของเรา — ฟีเจอร์นี้อยู่ใน Ultimate tier
4. **Secret Detection** ป้องกันข้อมูลลับ (API key, password) หลุดเข้า repository พร้อมมี **Secret Push Protection** ที่บล็อกการ push ตั้งแต่ต้นทางก่อนที่ secret จะเข้าไปอยู่ใน git history เลยด้วยซ้ำ — ใช้ได้ฟรีในทุก tier
5. **Container Scanning** ตรวจ Docker image ในระดับ OS package และ base image ซึ่งเป็นจุดที่ SAST/Dependency Scanning มองไม่เห็น — อยู่ใน Ultimate tier
6. **License Compliance** ตรวจสอบ license ของ dependency เทียบกับนโยบายองค์กร ป้องกันความเสี่ยงทางกฎหมายจาก license แบบ copyleft เช่น GPL/AGPL — อยู่ใน Ultimate tier
7. **Security Dashboard** ให้ภาพรวมความปลอดภัยระดับ project/group พร้อมแนวโน้มตามเวลา ช่วยให้จัดลำดับความสำคัญของงานแก้ไขได้ — อยู่ใน Ultimate tier
8. **Vulnerability Report** คือหน้าจัดการช่องโหว่แบบละเอียด สามารถ Dismiss, Resolve หรือ Create Issue จากช่องโหว่ที่พบได้โดยตรง เชื่อมทุกอย่างเข้ากับ Issue Tracker ของ GitLab — อยู่ใน Ultimate tier
9. **SAST และ Secret Detection ใช้ได้ฟรีในทุก tier** ในขณะที่ Dependency Scanning, Container Scanning, License Compliance, Security Dashboard และ Vulnerability Report ต้องใช้ **Ultimate tier** เท่านั้น — Premium tier ไม่ได้เพิ่มฟีเจอร์ security หลักมากไปกว่า Free
10. เราได้ลงมือเปิดใช้ SAST และ Secret Detection จริงในโปรเจกต์ทดสอบ เห็นผลลัพธ์การสแกน และฝึก workflow แบบ DevSecOps ที่ถูกต้องตั้งแต่ต้นจนจบ

### Checklist ก่อนไป Part ถัดไป

- [ ] เข้าใจแนวคิด DevSecOps และหลักการ "Shift Left"
- [ ] เปิดใช้งาน SAST ผ่าน `include: Security/SAST.gitlab-ci.yml` ได้ด้วยตัวเอง
- [ ] เข้าใจความแตกต่างระหว่าง SAST, Dependency Scanning, Container Scanning ว่าแต่ละตัวตรวจอะไรต่างกัน
- [ ] เข้าใจว่า Secret Detection และ Secret Push Protection ทำงานต่างกันอย่างไร (ตรวจหลัง push vs บล็อกก่อน push)
- [ ] เข้าใจความเสี่ยงด้าน License ของ Open Source dependency และหน้าที่ของ License Compliance
- [ ] รู้ว่า Security Dashboard และ Vulnerability Report ใช้ทำอะไร และอยู่ tier ไหน
- [ ] จำตารางสรุป tier ได้ อย่างน้อยว่า SAST/Secret Detection ฟรี ส่วนที่เหลือ (Dependency/Container/License/Dashboard) ต้อง Ultimate
- [ ] ทำแบบฝึกหัด Step 540 สำเร็จ เห็นผลการสแกนจริงจากโปรเจกต์ทดสอบของตัวเอง

**ต่อไป:** [Part 55: โปรเจกต์ฝึกหัด: ย้ายโปรเจกต์จาก GitHub ไป GitLab](./part-055-ย้ายโปรเจกต์-github-ไป-gitlab.md)
