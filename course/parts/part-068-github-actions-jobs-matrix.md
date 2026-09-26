# Part 68: GitHub Actions: Jobs, Steps, Matrix Build

> **Step ในหลักสูตรนี้:** Step 671–680
> **เฟส:** 7 — CI/CD เต็มรูปแบบด้วย GitHub Actions และ GitLab CI/CD
> **เป้าหมายของ Part นี้:** เข้าใจโครงสร้างของ Job และ Step ใน GitHub Actions อย่างลึกซึ้ง รู้วิธีควบคุมลำดับการทำงานด้วย `needs`, สร้าง Matrix Build เพื่อทดสอบโค้ดบนหลาย environment พร้อมกัน ปรับแต่งพฤติกรรมของ matrix ด้วย `include`/`exclude`/`fail-fast`/`max-parallel`, ใช้เงื่อนไข `if:` ควบคุมการรัน, ส่งค่าระหว่าง job ด้วย outputs และจัดการ timeout/error handling อย่างมืออาชีพ

---

## สารบัญของ Part นี้

- Step 671: หลาย jobs ในไฟล์เดียว ทำงานขนานกันโดย default (parallel by default)
- Step 672: `needs:` — กำหนดลำดับ job ที่ต้องรอให้ job อื่นเสร็จก่อน
- Step 673: Matrix build คืออะไร (ทดสอบหลาย version/OS/environment พร้อมกันในครั้งเดียว)
- Step 674: `strategy.matrix` syntax เต็มรูปแบบ
- Step 675: `matrix.include`/`matrix.exclude` — ปรับแต่ง combination
- Step 676: `fail-fast` และ `max-parallel` ควบคุมพฤติกรรมของ matrix
- Step 677: `if:` conditions — รัน step/job แบบมีเงื่อนไข
- Step 678: Job outputs — ส่งค่าระหว่าง job ด้วย `outputs:`
- Step 679: `timeout-minutes` และ `continue-on-error`
- Step 680: แบบฝึกหัด — เขียน workflow ทดสอบ 3 เวอร์ชัน Node.js คูณ 2 OS ด้วย matrix build ครบวงจร

---

## Step 671: หลาย jobs ในไฟล์เดียว ทำงานขนานกันโดย default (parallel by default)

ใน Part ก่อนหน้านี้เราเคยเห็น workflow ที่มีแค่ job เดียว แต่ในความเป็นจริง โปรเจกต์ระดับมืออาชีพแทบทุกโปรเจกต์จะมี **หลาย job ในไฟล์เดียวกัน** เช่น job สำหรับ lint, job สำหรับ test, job สำหรับ build และ job สำหรับ deploy

### โครงสร้างพื้นฐานของ workflow ที่มีหลาย job

```yaml
name: CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run ESLint
        run: npx eslint .

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: npm test

  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build project
        run: npm run build
```

สังเกตว่าเรามี 3 keys ภายใต้ `jobs:` คือ `lint`, `test`, `build` — แต่ละ key คือ**หนึ่ง job ที่แยกจากกันโดยสิ้นเชิง**

### สิ่งสำคัญที่สุดที่ต้องเข้าใจ: Jobs ทำงานแบบขนาน (Parallel) โดย default

ถ้าไม่มีการระบุ `needs:` (ซึ่งจะเรียนใน Step 672) **GitHub Actions จะรันทุก job พร้อมกันทันทีที่ workflow ถูก trigger** โดยแต่ละ job จะได้ runner (virtual machine) ของตัวเองแยกต่างหาก

```
เวลา:  0s ─────────────────────────────▶ เสร็จ

lint  :  ▶───────▶ (เสร็จใน 30s)
test  :  ▶───────────────▶ (เสร็จใน 90s)
build :  ▶─────────────▶ (เสร็จใน 60s)

           ↑ ทั้ง 3 job เริ่มพร้อมกันตั้งแต่วินาทีที่ 0
```

เทียบกับถ้ารันแบบ sequential (เรียงลำดับ) ทีละ job จะใช้เวลารวม 30 + 90 + 60 = 180 วินาที แต่เมื่อรันแบบขนาน เวลาทั้งหมดของ workflow จะเท่ากับ job ที่ใช้เวลานานที่สุด คือ 90 วินาทีเท่านั้น

### ทำไม GitHub ถึงออกแบบให้ parallel by default

1. **ความเร็ว** — ทีมพัฒนาต้องการ feedback เร็วที่สุดเท่าที่จะทำได้ ยิ่ง CI เร็ว ยิ่งทำงานได้คล่องตัว
2. **ความเป็นอิสระของแต่ละงาน** — โดยธรรมชาติแล้ว lint, test, build ไม่ได้ต้องพึ่งพากันเสมอไป จึงไม่มีเหตุผลที่ต้องรอ
3. **การใช้ทรัพยากรอย่างคุ้มค่า** — GitHub มี runner จำนวนมากพร้อมให้ใช้งาน การรันขนานคือการใช้ทรัพยากรเหล่านั้นให้เต็มประสิทธิภาพ

### แต่ละ Job คือ Virtual Machine ที่แยกจากกันโดยสิ้นเชิง

ข้อควรระวังที่มือใหม่มักพลาด: **job แต่ละตัวรันบน runner (VM) คนละเครื่อง** ไม่มี filesystem ร่วมกัน ไม่มีตัวแปรร่วมกัน ถ้า job `build` สร้างไฟล์ `dist/app.js` ไว้ แล้ว job `test` ต้องการใช้ไฟล์นั้น — **job `test` จะหาไฟล์นั้นไม่เจอ** เพราะมันรันอยู่คนละเครื่องกัน

```
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│   Runner VM #1   │   │   Runner VM #2   │   │   Runner VM #3   │
│                  │   │                  │   │                  │
│   job: lint      │   │   job: test      │   │   job: build     │
│   (filesystem     │   │   (filesystem    │   │   (filesystem    │
│    ของตัวเอง)      │   │    ของตัวเอง)     │   │    ของตัวเอง)     │
└─────────────────┘   └─────────────────┘   └─────────────────┘
        ไม่มีการแชร์ข้อมูลกันโดยอัตโนมัติระหว่าง job เหล่านี้
```

ถ้าต้องการส่งไฟล์ข้ามระหว่าง job จะต้องใช้ **artifacts** (`actions/upload-artifact` และ `actions/download-artifact`) ซึ่งเราจะเรียนใน **Part 69** หรือใช้ **job outputs** สำหรับส่งค่าตัวแปรสั้น ๆ (เรียนใน Step 678 ของ Part นี้)

### แต่ละ Step ภายใน Job เดียวกัน ทำงานแบบ Sequential เสมอ

ในทางกลับกัน **step ที่อยู่ภายใน job เดียวกัน จะรันตามลำดับจากบนลงล่างเสมอ** (sequential) ไม่ใช่แบบขนาน และ step ทั้งหมดใน job เดียวกันจะแชร์ filesystem เดียวกัน (เพราะรันบน runner เครื่องเดียวกัน)

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4          # Step 1: ทำงานก่อน
      - name: Install dependencies
        run: npm ci                        # Step 2: รอ Step 1 เสร็จก่อน
      - name: Build
        run: npm run build                 # Step 3: รอ Step 2 เสร็จก่อน
      - name: Show output
        run: ls -la dist/                  # Step 4: ใช้ไฟล์จาก Step 3 ได้ เพราะรันบนเครื่องเดียวกัน
```

### สรุปความแตกต่างที่สำคัญที่สุดของ Step นี้

| ระดับ | พฤติกรรม default | แชร์ filesystem กันไหม |
|---|---|---|
| **Job ↔ Job** | รันขนานกัน (parallel) | ไม่แชร์ — คนละ runner |
| **Step ↔ Step** (ใน job เดียวกัน) | รันตามลำดับ (sequential) | แชร์ — runner เดียวกัน |

การเข้าใจตารางนี้ให้แม่นคือรากฐานสำคัญที่สุดก่อนจะไปเรียนเรื่อง `needs:` และ Matrix Build ใน Step ถัดไป

---

## Step 672: `needs:` — กำหนดลำดับ job ที่ต้องรอให้ job อื่นเสร็จก่อน

จาก Step 671 เราเข้าใจแล้วว่า job รันขนานกันโดย default แต่ในความเป็นจริง มีหลายสถานการณ์ที่ **job หนึ่งต้องรอให้อีก job หนึ่งเสร็จก่อน** เช่น:

- ต้อง `build` ให้เสร็จก่อน ถึงจะ `deploy` ได้
- ต้อง `test` ผ่านก่อน ถึงจะอนุญาตให้ `merge` หรือ `publish` package
- ต้องมี job `setup` ที่เตรียม environment ก่อน job อื่น ๆ จะเริ่มทำงาน

นี่คือหน้าที่ของ keyword **`needs:`**

### Syntax พื้นฐานของ `needs:`

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

ในตัวอย่างนี้ job `deploy` มี `needs: build` หมายความว่า **`deploy` จะไม่เริ่มทำงานจนกว่า `build` จะรันเสร็จสมบูรณ์ (สำเร็จ) ก่อน**

```
เวลา:  0s ────────────────────────────▶

build :  ▶───────▶ (เสร็จ)
deploy:           ▶───────▶  (เริ่มหลังจาก build เสร็จเท่านั้น)
```

### `needs:` แบบหลาย job (array)

ถ้า job ต้องรอมากกว่าหนึ่ง job ให้ใช้ array:

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - run: npx eslint .

  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  build:
    runs-on: ubuntu-latest
    steps:
      - run: npm run build

  deploy:
    needs: [lint, test, build]
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying only after lint, test, and build all succeed"
```

```
                ┌────────┐
        ┌──────▶│  lint  │───────┐
        │       └────────┘       │
        │                        │
start ──┼──────▶┌────────┐       │
        │       │  test  │───────┼──────▶┌────────┐
        │       └────────┘       │       │ deploy │
        │                        │       └────────┘
        └──────▶┌────────┐       │
                │ build  │───────┘
                └────────┘

   deploy จะรอให้ทั้ง lint, test, build เสร็จสมบูรณ์ทั้งหมดก่อน จึงเริ่มทำงาน
```

นี่คือแนวคิดของ **DAG (Directed Acyclic Graph)** — GitHub Actions จะวิเคราะห์ความสัมพันธ์ของ `needs:` ทั้งหมดในไฟล์ แล้วสร้างกราฟการทำงานที่ไม่มีวงจรวน (acyclic) ขึ้นมาโดยอัตโนมัติ เพื่อคำนวณว่า job ไหนควรเริ่มเมื่อไหร่

### สร้าง Pipeline หลายชั้น (Multi-stage pipeline)

`needs:` สามารถ chain กันเป็นหลายชั้นได้ ไม่จำกัดแค่ 2 ชั้น:

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Preparing environment"

  build:
    needs: setup
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building"

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing"

  package:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: echo "Packaging"

  deploy-staging:
    needs: package
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to staging"

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to production"
```

```
setup → build → test → package → deploy-staging → deploy-production
```

### พฤติกรรมเมื่อ job ที่ถูก `needs:` ล้มเหลว

**กฎสำคัญ:** ถ้า job ที่อยู่ใน `needs:` ล้มเหลว (fail) **job ที่รออยู่จะถูกข้าม (skipped) โดยอัตโนมัติ** ไม่ใช่ล้มเหลวตามไปด้วย แต่จะมีสถานะเป็น "skipped" ใน UI

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: exit 1   # จำลองว่า test ล้มเหลว

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: echo "This will NEVER run because test failed"
```

ผลลัพธ์: `test` = ❌ failed, `deploy` = ⏭️ skipped (ไม่ใช่ failed แต่ก็ไม่ได้รัน)

### ถ้าต้องการให้ job รันแม้ job ก่อนหน้าจะล้มเหลว

บางครั้งเราต้องการให้ job บางตัวรันเสมอ ไม่ว่า job ก่อนหน้าจะผ่านหรือไม่ผ่าน เช่น job ที่ส่ง notification แจ้งผลลัพธ์ ให้ใช้ `if: always()` ร่วมกับ `needs:` (จะอธิบายละเอียดใน Step 677):

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  notify:
    needs: test
    if: always()          # รันเสมอ ไม่ว่า test จะสำเร็จหรือล้มเหลว
    runs-on: ubuntu-latest
    steps:
      - run: echo "Sending notification about test result: ${{ needs.test.result }}"
```

### การอ่านสถานะของ job ก่อนหน้าผ่าน `needs.<job_id>.result`

GitHub Actions มี context พิเศษชื่อ `needs` ที่เก็บผลลัพธ์ของ job ที่ถูกรอ สามารถอ่านค่า `.result` ได้ ซึ่งมีค่าที่เป็นไปได้คือ `success`, `failure`, `cancelled`, หรือ `skipped`

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: npm run build

  report:
    needs: build
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Check build result
        run: |
          if [ "${{ needs.build.result }}" == "success" ]; then
            echo "Build succeeded, proceeding with report"
          else
            echo "Build failed or was skipped: ${{ needs.build.result }}"
          fi
```

### ตัวอย่างที่ผสมทั้ง parallel และ sequential ในไฟล์เดียว

นี่คือรูปแบบที่พบบ่อยที่สุดในโปรเจกต์จริง — ให้ job ตรวจสอบคุณภาพโค้ดรันขนานกันก่อน แล้วค่อยรวมกันเป็นจุดเดียวก่อน deploy:

```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npx eslint .

  unit-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run test:unit

  integration-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run test:integration

  build:
    needs: [lint, unit-test, integration-test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to production"
```

```
   lint            ┐
   unit-test       ├──▶ build ──▶ deploy
   integration-test┘
   (ทั้ง 3 รันขนานกัน)
```

รูปแบบนี้ทำให้ pipeline เร็วที่สุดเท่าที่จะทำได้ เพราะ 3 job แรกไม่ต้องพึ่งกัน จึงรันพร้อมกัน แล้วค่อยมารวมกันที่จุดเดียวก่อนไปต่อ

---

## Step 673: Matrix build คืออะไร (ทดสอบหลาย version/OS/environment พร้อมกันในครั้งเดียว)

### ปัญหาที่ Matrix Build มาแก้

สมมติคุณกำลังพัฒนา library ที่ต้องรองรับ Node.js หลายเวอร์ชัน (เช่น 18, 20, 22) และต้องทำงานได้ทั้งบน Windows, macOS, Linux คำถามคือ: จะทดสอบให้ครบทุก combination ได้อย่างไร โดยไม่ต้องเขียน job ซ้ำ ๆ กันหลายสิบครั้ง

ถ้าเขียนแบบ **ไม่ใช้ matrix** จะต้องเขียนแบบนี้ (ซ้ำซาก น่าเบื่อ และแก้ยาก):

```yaml
jobs:
  test-node18-ubuntu:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm test

  test-node20-ubuntu:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm test

  test-node22-ubuntu:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - run: npm test

  test-node18-windows:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm test

  # ... และต้องเขียนซ้ำแบบนี้ไปเรื่อย ๆ จนครบทุก combination
```

จะเห็นว่านี่คือ **anti-pattern ที่ชัดเจน**: โค้ดซ้ำซ้อนมาก ถ้าต้องเพิ่ม Node เวอร์ชันใหม่ ต้องไปเพิ่ม job ใหม่หลายตัว ถ้าต้องแก้ step การรัน test ต้องไปแก้ทุก job พร้อมกัน — นี่คือปัญหาคลาสสิกที่ **Matrix Build** ถูกออกแบบมาเพื่อแก้โดยเฉพาะ

### แนวคิดของ Matrix Build

> **Matrix Build คือกลไกที่ให้คุณนิยาม job "ต้นแบบ" เพียงครั้งเดียว แล้วบอก GitHub Actions ว่าต้องการรัน job นั้นซ้ำกี่ครั้ง ด้วยชุดค่าตัวแปรที่แตกต่างกันแบบใด — GitHub Actions จะสร้าง job จริงขึ้นมาให้อัตโนมัติตามจำนวน combination ทั้งหมด**

เขียนใหม่ด้วย matrix จะกลายเป็นแบบนี้ (สั้นกระชับมาก):

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

Job เดียวนี้ GitHub Actions จะขยายออกมาเป็น **6 job จริง** โดยอัตโนมัติ (2 OS × 3 Node version = 6 combination):

```
1. test (ubuntu-latest, 18)
2. test (ubuntu-latest, 20)
3. test (ubuntu-latest, 22)
4. test (windows-latest, 18)
5. test (windows-latest, 20)
6. test (windows-latest, 22)
```

และทั้ง 6 job นี้จะ **รันขนานกันทั้งหมด** (ตามหลักการ default ที่เรียนใน Step 671) ทำให้ได้ผลการทดสอบครบทุก environment ภายในเวลาไม่นานกว่าการรันแค่ 1 combination มากนัก

### สถานการณ์การใช้งานจริงที่พบบ่อยที่สุด

| สถานการณ์ | ตัวอย่างค่าที่ใช้ทำ matrix |
|---|---|
| **ทดสอบหลาย version ของภาษา/runtime** | Node.js 18/20/22, Python 3.9/3.10/3.11/3.12 |
| **ทดสอบข้าม Operating System** | ubuntu-latest, windows-latest, macos-latest |
| **ทดสอบหลาย version ของ database** | PostgreSQL 14/15/16, MySQL 5.7/8.0 |
| **ทดสอบหลาย configuration ของแอป** | environment: dev/staging/production |
| **ทดสอบหลาย browser (สำหรับ E2E test)** | chromium, firefox, webkit |
| **ทดสอบหลาย architecture** | x86, arm64 |

### ทำไม Matrix Build ถึงสำคัญมากในโลกจริง

1. **จับ bug ที่เกิดเฉพาะบาง environment ได้เร็ว** — บางทีโค้ดทำงานได้ปกติบน Node 20 แต่พังบน Node 18 เพราะใช้ syntax ใหม่ที่ยังไม่รองรับ ถ้าไม่มี matrix build อาจไม่รู้จนกว่า user จริงจะเจอปัญหา
2. **มั่นใจได้ว่า library หรือ package รองรับ environment ที่ระบุไว้จริง** — ถ้า `package.json` บอกว่ารองรับ Node >= 18 ก็ควรมี matrix ทดสอบ Node 18 จริง ๆ ไม่ใช่แค่เขียนไว้เฉย ๆ
3. **ลดโค้ดซ้ำซ้อนในไฟล์ workflow มหาศาล** — จากตัวอย่างข้างบน จาก 6 job (หรือมากกว่านั้นถ้ามีมากกว่า 2 มิติ) เหลือแค่ job เดียวที่มี `strategy.matrix`
4. **บำรุงรักษาง่าย** — ถ้าต้องการเพิ่ม Node version 24 เข้ามาทดสอบ แค่เพิ่มเลข `24` ลงใน array เดียว ไม่ต้องไปสร้าง job ใหม่ทั้งก้อน

Step ถัดไปเราจะลงรายละเอียด syntax ของ `strategy.matrix` แบบเต็มรูปแบบ

---

## Step 674: `strategy.matrix` syntax เต็มรูปแบบ

### โครงสร้างพื้นฐาน

`strategy.matrix` ถูกวางไว้ในระดับเดียวกับ `runs-on` และ `steps` ภายใน job:

```yaml
jobs:
  <job_id>:
    runs-on: <ใช้ค่าจาก matrix ได้>
    strategy:
      matrix:
        <ชื่อตัวแปร 1>: [ค่า A, ค่า B, ...]
        <ชื่อตัวแปร 2>: [ค่า X, ค่า Y, ...]
    steps:
      - ...ใช้ ${{ matrix.<ชื่อตัวแปร> }} ...
```

`matrix:` รับ **key-value pairs** โดยแต่ละ key คือชื่อตัวแปรที่คุณตั้งเอง (จะตั้งชื่ออะไรก็ได้ ไม่จำเป็นต้องเป็น `os` หรือ `node-version` เสมอไป) และ value คือ **array ของค่าที่ต้องการทดสอบ**

### หลักการ Cartesian Product (การคูณไขว้)

หัวใจสำคัญของ matrix คือมันจะสร้าง **ทุก combination ที่เป็นไปได้ (cartesian product)** จากตัวแปรทั้งหมดที่ประกาศไว้ ไม่ใช่แค่จับคู่ตามลำดับ index

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node-version: [18, 20]
```

จำนวน job ที่ถูกสร้าง = จำนวนค่าของ `os` (3) × จำนวนค่าของ `node-version` (2) = **6 job**

```
              node-version: 18        node-version: 20
os: ubuntu   │  job 1                │  job 2
os: windows  │  job 3                │  job 4
os: macos    │  job 5                │  job 6
```

ทั้ง 6 combination นี้จะถูกสร้างครบทุกอัน — **ไม่ใช่แค่ 3 job** (ubuntu+18, windows+20, macos+??) ตามที่มือใหม่บางคนเข้าใจผิด

### ตัวอย่างเต็มรูปแบบ พร้อมใช้ค่าจาก matrix ในหลายจุด

```yaml
name: Cross-platform Test

on: [push, pull_request]

jobs:
  test:
    name: Test on ${{ matrix.os }} with Node ${{ matrix.node-version }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
```

จุดที่ควรสังเกต:

1. **`runs-on: ${{ matrix.os }}`** — ใช้ค่าจาก matrix กำหนด runner ที่จะใช้ นี่คือเทคนิคสำคัญที่ทำให้ทดสอบข้าม OS ได้จริง
2. **`name: Test on ${{ matrix.os }} with Node ${{ matrix.node-version }}`** — การตั้ง `name:` แบบ dynamic ทำให้เวลาไปดูผลใน tab Actions บน GitHub จะเห็นชื่อ job ที่ชัดเจนแยกแต่ละ combination เช่น "Test on ubuntu-latest with Node 18" แทนที่จะเห็นแค่ "test (ubuntu-latest, 18)" ที่ GitHub สร้างให้อัตโนมัติ
3. **`node-version: ${{ matrix.node-version }}`** — ส่งค่าจาก matrix ไปเป็น input ของ action `actions/setup-node@v4`

### Matrix ที่มีมากกว่า 2 มิติ (Multi-dimensional matrix)

Matrix ไม่ได้จำกัดแค่ 2 ตัวแปร สามารถมีกี่มิติก็ได้ (แต่ต้องระวังเรื่องจำนวน job ทั้งหมดที่จะระเบิดขึ้นแบบทวีคูณ):

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node-version: [18, 20, 22]
    database: [postgres, mysql]
```

จำนวน job = 2 × 3 × 2 = **12 job**

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20, 22]
        database: [postgres, mysql]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - name: Run tests against ${{ matrix.database }}
        run: npm test
        env:
          DB_TYPE: ${{ matrix.database }}
```

### การใช้ matrix เป็น object แทน primitive value

ค่าใน matrix ไม่จำเป็นต้องเป็น string หรือ number ธรรมดา สามารถเป็น **object ที่มีหลาย field** ได้ ซึ่งมีประโยชน์มากเมื่อค่าต่าง ๆ ผูกติดกันเป็นชุด (จะละเอียดมากขึ้นใน Step 675 เรื่อง `include`) ตัวอย่างเบื้องต้น:

```yaml
strategy:
  matrix:
    config:
      - { name: "Linux GCC", os: ubuntu-latest, compiler: gcc }
      - { name: "Linux Clang", os: ubuntu-latest, compiler: clang }
      - { name: "Windows MSVC", os: windows-latest, compiler: msvc }
steps:
  - name: Build with ${{ matrix.config.compiler }}
    run: echo "Building on ${{ matrix.config.os }} using ${{ matrix.config.compiler }}"
```

รูปแบบนี้ให้ 3 job ตามจำนวน element ใน array `config` โดยตรง (ไม่ใช่ cartesian product เพราะมีตัวแปร matrix แค่ตัวเดียวคือ `config`) วิธีนี้เหมาะเมื่อ combination ที่ต้องการไม่ใช่ทุกความเป็นไปได้ แต่เป็นชุดที่กำหนดไว้ตายตัว

### การเข้าถึงค่า matrix ในทุกจุดของ workflow

ค่า matrix เข้าถึงได้ผ่าน context `matrix` ในทุกที่ที่ expression syntax `${{ }}` ใช้งานได้ เช่น:

- `runs-on:`
- `if:`
- `env:`
- `with:`
- `run:` (ผ่าน `${{ }}` หรือ environment variable)
- `name:`

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
        node-version: [18, 20]
    env:
      NODE_ENV: test
      CURRENT_NODE: ${{ matrix.node-version }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - name: Print matrix info
        run: |
          echo "Running on OS: ${{ matrix.os }}"
          echo "Node version: ${{ matrix.node-version }}"
          echo "From env var: $CURRENT_NODE"
```

### ข้อจำกัดที่ต้องรู้

| ข้อจำกัด | ค่า |
|---|---|
| จำนวน job สูงสุดที่ matrix สร้างได้ต่อ workflow run | **256 jobs** |
| จำนวน job ที่รันพร้อมกันได้ | ขึ้นกับ plan ของ GitHub และ concurrency limit ขององค์กร |
| ความลึกของ matrix (จำนวนมิติ) | ไม่มีการจำกัดตายตัว แต่ในทางปฏิบัติควรจำกัดตัวเองไม่ให้เกิน 3-4 มิติ เพื่อไม่ให้ยากต่อการดูแล |

ถ้า matrix ของคุณคำนวณแล้วเกิน 256 combination GitHub Actions จะ**ปฏิเสธการรัน workflow** ทันที ดังนั้นควรออกแบบ matrix ให้พอเหมาะกับสิ่งที่จำเป็นต้องทดสอบจริง ๆ ไม่ใช่ใส่ทุกอย่างที่นึกออก

---

## Step 675: `matrix.include`/`matrix.exclude` — ปรับแต่ง combination ที่ไม่ต้องการ/ต้องการเพิ่มเติม

Cartesian product ที่เรียนใน Step 674 นั้นสะดวก แต่บางครั้ง **ไม่ใช่ทุก combination ที่มีความหมายหรือจำเป็นต้องทดสอบ** เช่น บาง feature อาจใช้ไม่ได้บน Windows หรือบาง combination อาจเป็นที่รู้กันอยู่แล้วว่า flaky/ไม่รองรับ — GitHub Actions มี `exclude` และ `include` ให้ปรับแต่งได้ละเอียด

### `matrix.exclude` — ตัด combination ที่ไม่ต้องการออก

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node-version: [18, 20, 22]
    exclude:
      - os: macos-latest
        node-version: 18
      - os: windows-latest
        node-version: 22
```

จาก cartesian product เดิม 3 × 3 = 9 combination เมื่อ exclude 2 รายการออก จะเหลือ **7 combination**:

```
              node 18          node 20          node 22
ubuntu       │  ✅ รัน          │  ✅ รัน          │  ✅ รัน
windows      │  ✅ รัน          │  ✅ รัน          │  ❌ excluded
macos        │  ❌ excluded     │  ✅ รัน          │  ✅ รัน
```

กฎการ match ของ `exclude`: **ต้องระบุ key-value ให้ตรงกับที่จะตัดออกเป๊ะ ๆ** ถ้าระบุแค่บางส่วน (partial match) เช่น `os: macos-latest` เฉย ๆ โดยไม่ระบุ `node-version` มันจะตัด **ทุก combination ที่มี `os: macos-latest`** ออกทั้งหมด ไม่ว่า node-version จะเป็นอะไรก็ตาม:

```yaml
exclude:
  - os: macos-latest   # ตัด macos-latest ออกทั้งหมด ไม่ว่า node-version จะเป็นอะไร
```

### `matrix.include` — เพิ่ม combination พิเศษเข้าไป

`include` มีสองแบบการใช้งานหลัก ๆ:

**แบบที่ 1: เพิ่ม combination ใหม่ที่ไม่ได้อยู่ใน cartesian product เดิม**

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node-version: [18, 20]
    include:
      - os: macos-latest
        node-version: 22
```

จาก cartesian product เดิม 2 × 2 = 4 combination บวก 1 combination พิเศษที่เพิ่มเข้ามา (macos-latest + node 22 ซึ่งไม่ได้เป็นส่วนหนึ่งของ os array หรือ node-version array เดิม) รวมเป็น **5 combination ทั้งหมด**

นี่มีประโยชน์มากเมื่อต้องการทดสอบ combination พิเศษเพิ่มเติมเพียงหนึ่งหรือสองอัน โดยไม่ต้องการให้มันไปคูณกับทุกตัวแปรอื่น (ถ้าเพิ่ม `macos-latest` ลงใน `os` array ตรง ๆ มันจะไปคูณกับ `node-version` ทั้งหมด กลายเป็น combination ที่ไม่ต้องการเพิ่มขึ้นมาโดยไม่จำเป็น)

**แบบที่ 2: เพิ่มตัวแปรพิเศษเข้าไปใน combination ที่ match กับ include**

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node-version: [18, 20]
    include:
      - os: ubuntu-latest
        node-version: 20
        experimental: true
```

ในกรณีนี้ `include` จะไป **match** กับ combination ที่มีอยู่แล้ว (`os: ubuntu-latest` + `node-version: 20`) แล้ว**เพิ่ม field ใหม่ชื่อ `experimental: true`** เข้าไปใน combination นั้นโดยเฉพาะ ส่วน combination อื่นจะไม่มี field `experimental` เลย (ค่าจะเป็น empty/undefined)

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20]
        include:
          - os: ubuntu-latest
            node-version: 20
            experimental: true
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - name: Check if experimental
        if: matrix.experimental
        run: echo "This is the experimental combination!"
```

### ลำดับการประมวลผล: `include` เกิดหลัง cartesian product เสมอ

สิ่งสำคัญที่ต้องจำ: GitHub Actions จะ**สร้าง cartesian product ก่อน แล้วค่อยประมวลผล `exclude` แล้วค่อยประมวลผล `include` เป็นลำดับสุดท้าย** ดังนั้น `include` สามารถเพิ่ม combination ที่ไม่ตรงกับ pattern ปกติได้อย่างอิสระ และแม้แต่ combination ที่ถูก `exclude` ไปแล้ว ก็สามารถถูกเพิ่มกลับเข้ามาใหม่ผ่าน `include` ได้เช่นกัน

### ตัวอย่างที่ผสมทั้ง `exclude` และ `include` เข้าด้วยกัน

```yaml
name: Advanced Matrix Example

on: [push]

jobs:
  test:
    name: Test (${{ matrix.os }}, Node ${{ matrix.node-version }})
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
        exclude:
          # macOS runner มีราคาแพงกว่า ไม่ต้องทดสอบ Node 18 (end-of-life ใกล้แล้ว)
          - os: macos-latest
            node-version: 18
        include:
          # เพิ่มการทดสอบ Node เวอร์ชัน nightly build บน ubuntu เท่านั้น
          - os: ubuntu-latest
            node-version: 'node-nightly'
            experimental: true
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Run tests
        run: npm test
        continue-on-error: ${{ matrix.experimental == true }}
```

### สรุปตารางเปรียบเทียบ `include` กับ `exclude`

| Keyword | หน้าที่ | จุดที่ต้องระวัง |
|---|---|---|
| `exclude` | ตัด combination ที่ไม่ต้องการออกจาก cartesian product | ต้องระบุ key-value ให้ตรงเป๊ะ ไม่งั้นจะตัดกว้างเกินไป |
| `include` (เพิ่ม combination ใหม่) | เพิ่ม combination ที่ไม่ได้อยู่ใน array เดิมของตัวแปรใด ๆ | ค่าที่เพิ่มจะไม่ไปคูณกับตัวแปรอื่นเหมือน cartesian product |
| `include` (เพิ่ม field ให้ combination เดิม) | แปะ field พิเศษเข้ากับ combination ที่ match เงื่อนไข | ต้องระบุ key ที่มีอยู่แล้วให้ตรงกับ combination เป้าหมาย |

---

## Step 676: `fail-fast` และ `max-parallel` ควบคุมพฤติกรรมของ matrix

เมื่อ matrix สร้าง job หลายสิบตัวขึ้นมา มีสองเรื่องสำคัญที่ต้องควบคุม: **จะยกเลิก job อื่นทันทีไหมถ้ามีตัวหนึ่ง fail** และ **จะให้รันพร้อมกันได้กี่ job สูงสุด**

### `fail-fast` — ยกเลิก job อื่นทันทีเมื่อมีตัวใดตัวหนึ่งล้มเหลว

**ค่า default ของ `fail-fast` คือ `true`** ซึ่งหมายความว่า **ถ้ามี job ใน matrix ตัวใดตัวหนึ่งล้มเหลว GitHub Actions จะยกเลิก (cancel) job อื่น ๆ ที่ยังรันอยู่ในทันที**

```yaml
strategy:
  # fail-fast: true คือค่า default (ไม่ต้องเขียนก็ได้ แต่เขียนให้ชัดเจนก็ดี)
  matrix:
    node-version: [18, 20, 22]
```

```
node 18:  ▶───✅ (ผ่าน)
node 20:  ▶───❌ (ล้มเหลวที่วินาทีที่ 30)
node 22:  ▶────🛑 (ถูกยกเลิกทันทีตอนวินาทีที่ 30 แม้จะยังไม่เสร็จ)
```

**เหตุผลที่ตั้งเป็น `true` โดย default:** ประหยัดเวลาและทรัพยากร — ถ้ารู้แล้วว่าโค้ดพังบน Node 20 ก็มักไม่มีประโยชน์ที่จะรอดู Node 22 ต่อให้จบ เพราะน่าจะแก้บั๊กแล้วรันใหม่ทั้งหมดอยู่ดี

### เมื่อไหร่ที่ควรตั้ง `fail-fast: false`

มีสถานการณ์ที่ **ไม่ต้องการให้ยกเลิก job อื่น** เพราะต้องการเห็นภาพรวมทั้งหมดว่า combination ไหนผ่านหรือไม่ผ่านบ้าง โดยเฉพาะตอนที่กำลัง debug ปัญหาข้าม environment:

```yaml
strategy:
  fail-fast: false
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node-version: [18, 20, 22]
```

```
node 18:  ▶───✅
node 20:  ▶───❌ (ล้มเหลว แต่ไม่กระทบตัวอื่น)
node 22:  ▶───✅ (รันจนจบตามปกติ)
```

ด้วยการตั้งค่านี้ คุณจะได้เห็นผลลัพธ์ครบทุก combination ในการรันครั้งเดียว เช่น อาจพบว่า "พังเฉพาะบน Windows + Node 18" ในขณะที่ combination อื่นผ่านหมด — ข้อมูลแบบนี้มีค่ามากสำหรับการ debug และควรเปิดใช้เมื่อกำลังตรวจสอบความเข้ากันได้ (compatibility) อย่างละเอียด

### กรณีใช้งานจริง: เปิด `fail-fast: false` เสมอสำหรับ compatibility testing

```yaml
name: Compatibility Matrix Test

on:
  schedule:
    - cron: '0 0 * * *'   # รันทุกวันเที่ยงคืน UTC เพื่อตรวจสอบ compatibility อย่างต่อเนื่อง

jobs:
  compatibility-check:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false     # ต้องการเห็นผลครบทุก OS/version แม้บางตัวจะพัง
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [16, 18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

### `max-parallel` — จำกัดจำนวน job ที่รันพร้อมกัน

โดย default, matrix จะพยายามรัน **ทุก job พร้อมกันทั้งหมดในคราวเดียว** (ขึ้นกับ concurrency limit ของ plan ที่ใช้อยู่) แต่บางครั้งเราต้องการจำกัดจำนวนไม่ให้รันพร้อมกันเยอะเกินไป

```yaml
strategy:
  max-parallel: 2
  matrix:
    node-version: [16, 18, 20, 22]
```

ด้วยการตั้ง `max-parallel: 2` แม้ matrix จะสร้าง 4 job แต่จะมีแค่ **2 job รันพร้อมกันสูงสุดในแต่ละช่วงเวลา** ตัวที่เหลือจะรอคิว (queued) จนกว่าจะมี slot ว่าง:

```
เวลา:     0s ─────────15s─────────30s─────────45s
node 16:  ▶───✅ (เสร็จที่ 15s)
node 18:  ▶───────✅ (เสร็จที่ 20s)
node 20:            ▶───✅ (เริ่มตอน 15s หลัง node16 จบ, เสร็จที่ 30s)
node 22:                  ▶───✅ (เริ่มตอน 20s หลัง node18 จบ)

  ณ เวลาใดก็ตาม จะมีสูงสุดแค่ 2 job ที่กำลังรันอยู่พร้อมกัน
```

### ทำไมถึงต้องการจำกัด `max-parallel`

1. **ทรัพยากรภายนอกมีจำกัด** — ถ้าแต่ละ job ต้องเชื่อมต่อไปยัง database เดียวกัน, external API ที่มี rate limit, หรือ shared test environment การรันพร้อมกันเยอะเกินไปอาจทำให้ระบบนั้นล่มหรือถูก throttle
2. **ควบคุมค่าใช้จ่าย (cost)** — ถ้าใช้ self-hosted runner ที่มีจำนวนจำกัด หรือกำลังจ่ายเงินตาม runner-minutes การจำกัด parallel ช่วยควบคุมการใช้ทรัพยากรไม่ให้พุ่งสูงเกินไปในคราวเดียว
3. **หลีกเลี่ยงการชนกันของ resource บน self-hosted runner** — ถ้า self-hosted runner มีจำนวนเครื่องจำกัด การส่ง job จำนวนมากพร้อมกันอาจทำให้ job ต้องรอคิวอยู่ดี การตั้ง `max-parallel` ให้สอดคล้องกับจำนวน runner ที่มีจะช่วยให้ pipeline คาดเดาพฤติกรรมได้ง่ายขึ้น

### ตัวอย่างผสม `fail-fast` และ `max-parallel` เข้าด้วยกัน

```yaml
jobs:
  integration-test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      max-parallel: 3
      matrix:
        service: [auth, payment, notification, inventory, shipping, reporting]
    steps:
      - uses: actions/checkout@v4
      - name: Run integration test for ${{ matrix.service }}
        run: ./scripts/test-service.sh ${{ matrix.service }}
        env:
          SHARED_TEST_DB_URL: ${{ secrets.TEST_DB_URL }}
```

ในตัวอย่างนี้มี 6 service ที่ต้องทดสอบ แต่ทั้งหมดต้องแชร์ `SHARED_TEST_DB_URL` เดียวกัน — การจำกัด `max-parallel: 3` ช่วยไม่ให้ทั้ง 6 job ยิง query เข้า database ทดสอบพร้อมกันจนระบบรับไม่ไหว ในขณะที่ `fail-fast: false` ทำให้เห็นผลลัพธ์ของทุก service แม้บาง service จะทดสอบไม่ผ่าน

### สรุปตารางเปรียบเทียบ

| ตัวเลือก | ค่า default | ผลกระทบ |
|---|---|---|
| `fail-fast` | `true` | เมื่อมี job ใดล้มเหลว จะยกเลิก job อื่นในทันที |
| `fail-fast: false` | (ต้องตั้งเอง) | ปล่อยให้ทุก job รันจนจบ ไม่ว่าจะมีตัวใดล้มเหลว |
| `max-parallel` | ไม่จำกัด (เต็มความสามารถของ plan) | จำกัดจำนวน job สูงสุดที่รันพร้อมกันในเวลาเดียวกัน |

---

## Step 677: `if:` conditions — รัน step/job แบบมีเงื่อนไข

`if:` คือ keyword ที่ใช้ควบคุมว่า job หรือ step นั้นควรจะ**รันหรือถูกข้าม (skip)** โดยพิจารณาจากเงื่อนไขที่ประเมินผลเป็น `true`/`false`

### `if:` ที่ระดับ Job

```yaml
jobs:
  deploy:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying because this is the main branch"
```

Job นี้จะรัน**ก็ต่อเมื่อ** push/merge เข้า branch `main` เท่านั้น ถ้า workflow ถูก trigger จาก branch อื่น job นี้จะแสดงสถานะเป็น "skipped" ทันที โดยไม่ต้องรอ runner เลย

### `if:` ที่ระดับ Step

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run build
      - name: Notify Slack on main branch only
        if: github.ref == 'refs/heads/main'
        run: ./scripts/notify-slack.sh
```

step อื่น ๆ ใน job จะรันตามปกติ แต่ step "Notify Slack" จะรันเฉพาะเมื่อเงื่อนไขเป็นจริงเท่านั้น

### Context และ Expression ที่ใช้บ่อยที่สุดใน `if:`

| Expression | ความหมาย |
|---|---|
| `github.ref == 'refs/heads/main'` | รันเฉพาะ branch `main` |
| `github.event_name == 'pull_request'` | รันเฉพาะเมื่อ trigger มาจาก pull request |
| `github.event_name == 'push'` | รันเฉพาะเมื่อ trigger มาจาก push |
| `startsWith(github.ref, 'refs/tags/')` | รันเฉพาะเมื่อ push เป็น tag |
| `github.actor == 'dependabot[bot]'` | รันเฉพาะเมื่อผู้กระทำการคือ dependabot |
| `contains(github.event.head_commit.message, '[skip ci]')` | ตรวจสอบข้อความใน commit message |

### Status check functions: `success()`, `failure()`, `always()`, `cancelled()`

นี่คือฟังก์ชันพิเศษ 4 ตัวที่ใช้ตรวจสอบสถานะของ step/job ก่อนหน้า:

| ฟังก์ชัน | ความหมาย | พฤติกรรม default ถ้าไม่เขียน `if:` เลย |
|---|---|---|
| `success()` | true เมื่อทุก step/job ก่อนหน้าสำเร็จ | **นี่คือค่า default โดยปริยาย** ถ้าไม่เขียน `if:` เลย GitHub จะถือว่าเป็น `if: success()` เสมอ |
| `failure()` | true เมื่อมี step/job ก่อนหน้าล้มเหลวอย่างน้อยหนึ่งตัว | ต้องเขียนเองเสมอ ไม่ใช่ default |
| `always()` | true เสมอ ไม่ว่าผลลัพธ์ก่อนหน้าจะเป็นอย่างไร (แม้จะถูก cancel ก็ยังรัน ยกเว้นกรณีพิเศษบางอย่าง) | ต้องเขียนเองเสมอ |
| `cancelled()` | true เมื่อ workflow ถูกยกเลิกโดยผู้ใช้หรือระบบ | ต้องเขียนเองเสมอ |

### ตัวอย่าง: รัน step แจ้งเตือนเมื่อ build ล้มเหลวเท่านั้น

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: npm run build

      - name: Notify on failure
        if: failure()
        run: ./scripts/notify-slack.sh "Build failed!"

      - name: Cleanup (run regardless of outcome)
        if: always()
        run: rm -rf tmp/
```

**ข้อควรระวังสำคัญ:** ถ้า step ก่อนหน้าล้มเหลว โดย default **step ถัดไปจะถูกข้ามทั้งหมด** เว้นแต่จะเขียน `if:` ที่ระบุชัดเจนว่าให้รันแม้จะล้มเหลว (เช่น `if: always()` หรือ `if: failure()`) — นี่คือเหตุผลที่ step "Cleanup" ในตัวอย่างข้างบนต้องมี `if: always()` เพื่อให้แน่ใจว่ามันรันเสมอไม่ว่า build จะสำเร็จหรือล้มเหลว

### ผสม status function กับเงื่อนไขอื่น

สามารถใช้ status function ร่วมกับ `&&` (AND) และ `||` (OR) ได้:

```yaml
steps:
  - name: Deploy only on main branch and only if build succeeded
    if: success() && github.ref == 'refs/heads/main'
    run: ./deploy.sh

  - name: Alert team on failure in main branch
    if: failure() && github.ref == 'refs/heads/main'
    run: ./scripts/alert-team.sh
```

### `if:` ระดับ Job ผสมกับ `needs:`

ตัวอย่างที่ใช้บ่อยมากในโลกจริง คือ job สำหรับ deploy ที่ต้องผ่านเงื่อนไขทั้งสองอย่าง คือทั้ง `needs` (รอ job อื่นให้เสร็จ) และ `if` (ต้องเป็น branch ที่ถูกต้อง):

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to production"

  deploy-preview:
    needs: test
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying preview environment for PR"
```

ในตัวอย่างนี้:
- ถ้า push เข้า `main` → job `deploy-production` จะรัน (หลังจาก `test` ผ่านแล้ว), `deploy-preview` จะถูกข้าม
- ถ้าเปิด pull request → job `deploy-preview` จะรัน, `deploy-production` จะถูกข้าม

### `if:` กับ matrix — ข้าม combination บางตัว

`if:` ยังใช้ร่วมกับ matrix เพื่อข้าม step หรือ combination บางอันได้ เช่นข้ามการทำ code coverage report เฉพาะตอนรันบน Windows (เพราะบางเครื่องมือทำ coverage ไม่รองรับ):

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test

      - name: Upload coverage report
        if: matrix.os == 'ubuntu-latest' && matrix.node-version == 20
        run: npm run coverage:upload
```

step "Upload coverage report" จะรันแค่ครั้งเดียวใน combination เดียว (ubuntu-latest + Node 20) แทนที่จะรันซ้ำใน 4 combination ทั้งหมด ซึ่งช่วยประหยัดเวลาและหลีกเลี่ยงการอัพโหลด report ซ้ำโดยไม่จำเป็น

---

## Step 678: Job outputs — ส่งค่าระหว่าง job ด้วย `outputs:`

จาก Step 671 เราทราบแล้วว่าแต่ละ job รันบนคนละ runner และไม่ได้แชร์ filesystem กัน แต่บ่อยครั้งที่เราต้องการส่ง**ค่าตัวแปรสั้น ๆ** (ไม่ใช่ไฟล์ทั้งไฟล์) จาก job หนึ่งไปให้อีก job หนึ่งใช้ต่อ เช่น เวอร์ชันที่ build ได้, URL ของ environment ที่ deploy ไปแล้ว, หรือผลการตรวจสอบบางอย่าง — นี่คือหน้าที่ของ **job outputs**

### กลไกการทำงาน: 3 ชั้นที่ต้องเข้าใจ

การส่งค่าจาก job หนึ่งไปอีก job หนึ่ง ต้องผ่าน 3 ชั้นตามลำดับ:

```
ชั้นที่ 1: Step เขียนค่าออกไปที่ไฟล์พิเศษ $GITHUB_OUTPUT
              ↓
ชั้นที่ 2: Job ประกาศ outputs: โดยอ้างอิงจาก steps.<step_id>.outputs.<name>
              ↓
ชั้นที่ 3: Job อื่นที่มี needs: อ่านค่าผ่าน needs.<job_id>.outputs.<name>
```

### ชั้นที่ 1: Step เขียนค่าออกที่ `$GITHUB_OUTPUT`

ภายใน step ใด ๆ สามารถเขียนค่าออกไปเก็บไว้ในไฟล์ที่ระบบกำหนดให้ผ่าน environment variable `$GITHUB_OUTPUT` ด้วย syntax `key=value`:

```yaml
steps:
  - name: Get version number
    id: get_version          # ต้องตั้ง id ให้ step นี้ เพื่ออ้างอิงในภายหลัง
    run: |
      VERSION=$(node -p "require('./package.json').version")
      echo "version=$VERSION" >> "$GITHUB_OUTPUT"
```

หมายเหตุ: ในอดีต GitHub Actions เคยใช้คำสั่ง `echo "::set-output name=version::$VERSION"` แต่ syntax นี้ **ถูก deprecate ไปแล้ว** และควรใช้ `echo "key=value" >> "$GITHUB_OUTPUT"` แทนเสมอในปัจจุบัน

### ชั้นที่ 2: Job ประกาศ `outputs:` โดยอ้างอิงจาก step

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      app-version: ${{ steps.get_version.outputs.version }}
    steps:
      - uses: actions/checkout@v4
      - name: Get version number
        id: get_version
        run: |
          VERSION=$(node -p "require('./package.json').version")
          echo "version=$VERSION" >> "$GITHUB_OUTPUT"
```

สังเกตว่า `outputs:` ของ job อยู่ในระดับเดียวกับ `runs-on:` และ `steps:` และค่าที่ประกาศ (`app-version`) อ้างอิงมาจาก `steps.<step_id>.outputs.<key>` โดย `<step_id>` ต้องตรงกับ `id:` ที่ตั้งไว้ในขั้นตอนที่ 1

### ชั้นที่ 3: Job อื่นอ่านค่าผ่าน `needs.<job_id>.outputs.<name>`

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      app-version: ${{ steps.get_version.outputs.version }}
    steps:
      - uses: actions/checkout@v4
      - name: Get version number
        id: get_version
        run: |
          VERSION=$(node -p "require('./package.json').version")
          echo "version=$VERSION" >> "$GITHUB_OUTPUT"

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Show version from previous job
        run: echo "Deploying version ${{ needs.build.outputs.app-version }}"
```

**สำคัญมาก:** เพื่อจะอ่านค่าจาก `needs.<job_id>.outputs` ได้ job นั้นต้องอยู่ใน `needs:` เท่านั้น — job ที่ไม่ได้ระบุความสัมพันธ์ผ่าน `needs:` จะไม่สามารถเข้าถึง output ของกันและกันได้เลย

### ตัวอย่างที่ครบวงจร: ส่งหลายค่าพร้อมกัน

```yaml
name: Build, Tag, and Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.value }}
      commit-sha: ${{ steps.sha.outputs.value }}
      artifact-name: ${{ steps.artifact.outputs.value }}
    steps:
      - uses: actions/checkout@v4

      - id: version
        name: Extract version
        run: echo "value=$(node -p "require('./package.json').version")" >> "$GITHUB_OUTPUT"

      - id: sha
        name: Get short SHA
        run: echo "value=$(git rev-parse --short HEAD)" >> "$GITHUB_OUTPUT"

      - id: artifact
        name: Build artifact name
        run: echo "value=app-${{ steps.version.outputs.value }}-${{ steps.sha.outputs.value }}.tar.gz" >> "$GITHUB_OUTPUT"

      - name: Build application
        run: npm run build

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Print deployment info
        run: |
          echo "Deploying version: ${{ needs.build.outputs.version }}"
          echo "Commit SHA: ${{ needs.build.outputs.commit-sha }}"
          echo "Artifact: ${{ needs.build.outputs.artifact-name }}"
```

### Job outputs ผสมกับ Matrix — ข้อควรระวัง

เมื่อ job ที่เป็น matrix มี `outputs:` จะต้องระวัง เพราะ matrix สร้างหลาย job ขึ้นมา แต่ job อื่นที่มา `needs:` จะเห็น output จาก **job ตัวใดตัวหนึ่งที่รันเสร็จล่าสุดเท่านั้น** (ผลลัพธ์อาจไม่แน่นอนว่าเป็นค่าจาก combination ไหน) ดังนั้นไม่แนะนำให้พึ่งพา output จาก matrix job โดยตรง หากต้องการค่าที่แน่นอน ควรใช้ job แยกที่ไม่ใช่ matrix สำหรับสร้างค่าที่ต้องส่งต่อ

```yaml
jobs:
  # แนะนำ: job แยกต่างหาก ไม่ใช่ matrix สำหรับสร้างค่าที่ job อื่นต้องพึ่งพา
  prepare:
    runs-on: ubuntu-latest
    outputs:
      build-id: ${{ steps.gen.outputs.id }}
    steps:
      - id: gen
        run: echo "id=build-$(date +%s)" >> "$GITHUB_OUTPUT"

  test:
    needs: prepare
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - run: echo "Testing with build ID ${{ needs.prepare.outputs.build-id }}"
```

### สรุปตารางไวยากรณ์ที่ต้องจำ

| ต้องการทำอะไร | Syntax |
|---|---|
| เขียนค่าออกจาก step | `echo "key=value" >> "$GITHUB_OUTPUT"` |
| ประกาศ output ของ job | `outputs:` ในระดับ job อ้างจาก `steps.<id>.outputs.<key>` |
| อ่าน output จาก job อื่น | `needs.<job_id>.outputs.<key>` (ต้องมี `needs:` ก่อน) |

---

## Step 679: `timeout-minutes` และ `continue-on-error`

การจัดการเรื่องเวลาและข้อผิดพลาดที่ควบคุมไม่ได้ เป็นเรื่องสำคัญมากในการออกแบบ pipeline ที่มั่นคง — สอง keyword นี้ช่วยให้ควบคุมพฤติกรรมเหล่านั้นได้อย่างชัดเจน

### `timeout-minutes` — จำกัดเวลาการรันสูงสุด

**ค่า default ของ GitHub Actions คือ 360 นาที (6 ชั่วโมง)** ต่อหนึ่ง job ถ้า job รันนานเกินกว่านี้ GitHub จะยกเลิกโดยอัตโนมัติและ mark เป็น failed แต่ 6 ชั่วโมงถือว่านานเกินไปมากสำหรับงานส่วนใหญ่ จึงควรกำหนด `timeout-minutes` ให้เหมาะสมกับงานจริงเสมอ

**ระดับ Job:**

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - run: npm test
```

ถ้า job นี้รันเกิน 15 นาที (รวมเวลาทุก step ในนั้น) GitHub จะยกเลิกทั้ง job ทันทีและ mark เป็น failed

**ระดับ Step:**

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 30       # timeout รวมของทั้ง job
    steps:
      - uses: actions/checkout@v4

      - name: Run unit tests
        timeout-minutes: 5    # timeout เฉพาะ step นี้
        run: npm run test:unit

      - name: Run E2E tests
        timeout-minutes: 20   # timeout เฉพาะ step นี้
        run: npm run test:e2e
```

### ทำไม `timeout-minutes` ถึงสำคัญมาก

1. **ป้องกัน test ค้าง (hang) กินเวลาและเงินโดยไม่จำเป็น** — บางครั้ง test ที่รอ network request ที่ไม่มีวันตอบกลับ อาจทำให้ job ค้างอยู่หลายชั่วโมงโดยไม่มีใครสังเกตเห็น ถ้าไม่ตั้ง timeout ที่เหมาะสม
2. **ควบคุมค่าใช้จ่าย runner-minutes** — GitHub เก็บเงินตามเวลาที่ runner ใช้จริง job ที่ค้างนาน ๆ จะทำให้ค่าใช้จ่ายพุ่งสูงโดยไม่ได้ตั้งใจ
3. **ทำให้ feedback loop เร็วขึ้น** — ถ้ารู้ว่า test ปกติใช้เวลาไม่เกิน 10 นาที การตั้ง timeout ที่ 15 นาทีจะทำให้รู้ปัญหาเร็วขึ้น แทนที่จะรอจนครบ 360 นาทีตาม default

### แนวทางกำหนดค่า `timeout-minutes` ที่เหมาะสม

| ประเภทงาน | timeout-minutes ที่แนะนำ (โดยประมาณ) |
|---|---|
| Lint / static analysis | 5-10 |
| Unit test | 10-15 |
| Integration test | 15-30 |
| E2E test (browser-based) | 20-45 |
| Build (compile, bundle) | 10-20 |
| Deploy | 10-15 |

ตัวเลขเหล่านี้เป็นเพียงแนวทางเริ่มต้น ควรปรับตามลักษณะโปรเจกต์จริง โดยหลักการคือ **ตั้งให้สูงกว่าเวลาที่ใช้จริงปกติสัก 1.5-2 เท่า** เพื่อให้มี buffer สำหรับความผันผวนของ runner แต่ไม่สูงจนไม่มีประโยชน์ในการจับปัญหา

### `continue-on-error` — ไม่ให้ error หยุด workflow ทั้งหมด

โดย default ถ้า step หรือ job ล้มเหลว **workflow ทั้งหมดจะถูก mark เป็น failed** (และ step/job ที่อยู่หลังจากมันจะถูกข้ามไป เว้นแต่จะใช้ `if: always()`) แต่บางครั้งเราต้องการให้ error บางจุด **ไม่ทำให้ทั้ง workflow ล้มเหลว** เช่น step ที่เป็น experimental หรือ optional check

**ระดับ Step:**

```yaml
steps:
  - name: Run experimental linter
    continue-on-error: true
    run: npx experimental-linter .

  - name: This step still runs even if the linter above failed
    run: echo "Continuing normally"
```

ถ้า step "Run experimental linter" ล้มเหลว มันจะแสดงเป็นเครื่องหมาย **⚠️ (warning icon)** ใน UI แทนที่จะเป็น ❌ และ step ถัดไปจะยังรันต่อตามปกติ **workflow โดยรวมจะไม่ถูก mark เป็น failed** จาก step นี้

**ระดับ Job (ผ่าน `strategy.matrix`) — สำคัญมากสำหรับ matrix build:**

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    continue-on-error: ${{ matrix.experimental }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20]
        experimental: [false]
        include:
          - os: ubuntu-latest
            node-version: 'node-nightly'
            experimental: true
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

นี่คือ pattern ที่พบบ่อยมากในโปรเจกต์ open source ขนาดใหญ่: ทดสอบ combination หลักที่ต้อง "ผ่านเสมอ" (`experimental: false`) ควบคู่กับการทดสอบเวอร์ชัน nightly/beta ที่ "ยอมให้พังได้" (`experimental: true`) โดยที่ผลของ combination ที่เป็น experimental จะไม่ทำให้ CI ทั้งหมดกลายเป็นสีแดง

### ความแตกต่างระหว่าง `continue-on-error` กับ `fail-fast: false`

มือใหม่มักสับสนระหว่างสองสิ่งนี้ ตารางนี้ช่วยแยกให้ชัดเจน:

| | `fail-fast: false` | `continue-on-error: true` |
|---|---|---|
| **ระดับที่ใช้** | `strategy` เท่านั้น (ผลกับทั้ง matrix) | `job` หรือ `step` |
| **ผลเมื่อล้มเหลว** | job อื่นใน matrix ยังรันต่อ แต่ job ที่ล้มเหลวยัง**นับเป็น failed** | job/step ที่ล้มเหลว**ไม่นับเป็น failed** ของ workflow โดยรวม |
| **สถานะ workflow โดยรวม** | ถ้ามี job ใดใน matrix fail → workflow โดยรวมยังคง fail | ถ้า step/job ที่มี `continue-on-error: true` fail → workflow โดยรวมยังคง**ผ่าน** (แสดงเป็น warning) |
| **ใช้ร่วมกันได้ไหม** | ได้ และมักใช้คู่กันสำหรับ matrix ที่มี experimental combination | ได้ |

---

## Step 680: แบบฝึกหัด — เขียน workflow ที่ทดสอบโปรเจกต์บน 3 เวอร์ชัน Node.js คูณ 2 OS ด้วย matrix build ครบวงจร

ถึงเวลาลงมือจริง มารวบรวมทุกสิ่งที่เรียนมาใน Part นี้เข้าด้วยกัน: หลาย job, `needs`, matrix build, `include`/`exclude`, `fail-fast`/`max-parallel`, `if:`, job outputs, และ `timeout-minutes`/`continue-on-error`

### โจทย์

สร้าง workflow ที่ทำสิ่งต่อไปนี้:

1. Job `prepare` — สร้างค่า build-id และ export เป็น output ให้ job อื่นใช้
2. Job `test` — เป็น matrix build ทดสอบ **Node.js 18, 20, 22** คูณ **ubuntu-latest, windows-latest** (รวม 6 combination) โดย:
   - ยกเว้น (exclude) combination Node 18 บน windows-latest (สมมติว่าทีมรู้อยู่แล้วว่า combination นี้ไม่รองรับ)
   - เพิ่ม (include) การทดสอบ Node 22 บน macos-latest เป็น combination พิเศษที่ทำเครื่องหมายว่า experimental
   - ตั้ง `fail-fast: false` เพื่อให้เห็นผลครบทุก combination
   - ตั้ง `continue-on-error` ให้ job ที่เป็น experimental ไม่ทำให้ workflow ทั้งหมดแดง
   - ตั้ง `timeout-minutes: 15`
3. Job `report` — รอให้ `test` เสร็จ (ไม่ว่าจะผ่านหรือไม่) แล้วรายงานสรุปผล
4. Job `deploy` — รันเฉพาะเมื่อ push เข้า branch `main` และ `test` ผ่านทั้งหมด

### เฉลย

```yaml
name: Full Matrix Build Exercise

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  prepare:
    runs-on: ubuntu-latest
    outputs:
      build-id: ${{ steps.generate.outputs.id }}
    steps:
      - uses: actions/checkout@v4

      - id: generate
        name: Generate build ID
        run: |
          BUILD_ID="build-$(date +%Y%m%d)-$(git rev-parse --short HEAD)"
          echo "id=$BUILD_ID" >> "$GITHUB_OUTPUT"
          echo "Generated build ID: $BUILD_ID"

  test:
    name: Test (${{ matrix.os }}, Node ${{ matrix.node-version }})
    needs: prepare
    runs-on: ${{ matrix.os }}
    timeout-minutes: 15
    continue-on-error: ${{ matrix.experimental == true }}
    strategy:
      fail-fast: false
      max-parallel: 4
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20, 22]
        experimental: [false]
        exclude:
          # combination นี้เป็นที่รู้กันว่ายังไม่รองรับ
          - os: windows-latest
            node-version: 18
        include:
          # เพิ่ม combination พิเศษสำหรับทดสอบ macOS บน Node 22 แบบ experimental
          - os: macos-latest
            node-version: 22
            experimental: true
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Show build info
        run: |
          echo "Build ID from prepare job: ${{ needs.prepare.outputs.build-id }}"
          echo "Running on: ${{ matrix.os }}"
          echo "Node version: ${{ matrix.node-version }}"
          echo "Experimental: ${{ matrix.experimental }}"

      - name: Install dependencies
        run: npm ci

      - name: Run lint
        run: npm run lint

      - name: Run unit tests
        timeout-minutes: 10
        run: npm test

      - name: Run build
        run: npm run build

  report:
    name: Report test results
    needs: [prepare, test]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Summarize results
        run: |
          echo "Build ID: ${{ needs.prepare.outputs.build-id }}"
          echo "Test job overall result: ${{ needs.test.result }}"
          if [ "${{ needs.test.result }}" == "success" ]; then
            echo "All required test combinations passed."
          else
            echo "Some test combinations failed or were cancelled. Check the matrix summary above."
          fi

  deploy:
    name: Deploy to production
    needs: [test, report]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push' && needs.test.result == 'success'
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4

      - name: Deploy
        run: |
          echo "Deploying build ${{ needs.prepare.outputs.build-id }} to production"
          echo "All matrix tests passed, safe to deploy."
```

### อธิบายจุดสำคัญของเฉลย ทีละส่วน

**1. Job `prepare`**

```yaml
prepare:
  runs-on: ubuntu-latest
  outputs:
    build-id: ${{ steps.generate.outputs.id }}
```

Job นี้ไม่ใช่ matrix เพราะเราต้องการค่า build-id เพียงค่าเดียวที่แน่นอน (ตามที่อธิบายไว้ใน Step 678 ว่าไม่ควรพึ่งพา output จาก matrix job โดยตรง)

**2. Job `test` — matrix ที่ผสมทุกเทคนิค**

- Cartesian product เริ่มต้น: `os` (2 ค่า) × `node-version` (3 ค่า) × `experimental` (1 ค่า) = 6 combination
- `exclude` ตัด `windows-latest` + Node 18 ออก 1 combination → เหลือ 5
- `include` เพิ่ม `macos-latest` + Node 22 + `experimental: true` เข้ามา 1 combination → รวมเป็น **6 combination ทั้งหมด**
- `fail-fast: false` ทำให้ทุก combination รันจนจบแม้บางตัวจะพัง
- `max-parallel: 4` จำกัดไม่ให้รันเกิน 4 job พร้อมกันในเวลาเดียว
- `continue-on-error: ${{ matrix.experimental == true }}` ทำให้เฉพาะ combination macOS+Node22 (ที่ `experimental: true`) ไม่ทำให้ workflow โดยรวมแดง แม้จะ test ไม่ผ่าน
- `timeout-minutes: 15` ที่ระดับ job และ `timeout-minutes: 10` เฉพาะ step "Run unit tests" ป้องกัน test ค้าง

**3. Job `report`**

```yaml
report:
  needs: [prepare, test]
  if: always()
```

`if: always()` สำคัญมาก — ถ้าไม่มีบรรทัดนี้ และ `test` ล้มเหลว job `report` จะถูก **skip** ไปเลย (ตามกฎที่เรียนใน Step 672) แต่เราต้องการให้ `report` รันเสมอเพื่อสรุปผล ไม่ว่า test จะผ่านหรือไม่

**4. Job `deploy`**

```yaml
deploy:
  needs: [test, report]
  if: github.ref == 'refs/heads/main' && github.event_name == 'push' && needs.test.result == 'success'
```

Job นี้ผสมเงื่อนไข 3 อย่างเข้าด้วยกัน: ต้องเป็น branch `main`, ต้อง trigger จาก event `push` (ไม่ใช่ pull request), และผลของ `test` ต้องเป็น `success` เท่านั้น — การเขียนแบบนี้ทำให้ deploy ปลอดภัยแม้ว่า `report` (ซึ่งมี `if: always()`) จะรันเสมอก็ตาม เพราะเราตรวจสอบผลของ `test` โดยตรง ไม่ใช่ผลของ `report`

### วิธีทดสอบและตรวจสอบผลลัพธ์จริง

1. สร้างไฟล์นี้ไว้ที่ `.github/workflows/matrix-exercise.yml` ใน repository ของคุณ
2. Push โค้ดขึ้น branch `develop` ก่อน (เพื่อไม่ให้ trigger job `deploy` ทันที)
3. เปิดแท็บ **Actions** บน GitHub แล้วดูว่า:
   - Job `prepare` รันก่อนเป็นอันดับแรก
   - Job `test` ขยายออกมาเป็น 6 job ย่อยตามชื่อที่กำหนดด้วย `name: Test (...)`
   - แต่ละ job ย่อยของ `test` แสดง badge สถานะแยกกัน และ combination ที่เป็น experimental (macOS+Node22) จะแสดงเป็นสัญลักษณ์ warning ถ้าล้มเหลว ไม่ใช่ error สีแดง
   - Job `report` รันหลังจาก `test` เสร็จเสมอ ไม่ว่าผลจะเป็นอย่างไร
4. ลอง merge เข้า `main` แล้วสังเกตว่า job `deploy` เริ่มทำงานหลังจาก `test` และ `report` เสร็จ (โดยมีเงื่อนไขว่า `test` ต้อง success)
5. ลองแก้โค้ดให้ test ล้มเหลวโดยตั้งใจใน combination ที่ไม่ใช่ experimental แล้วสังเกตว่า `deploy` ถูกข้ามไปเพราะเงื่อนไข `needs.test.result == 'success'` เป็นเท็จ

### ความท้าทายเพิ่มเติม (Optional — ลองทำเอง)

1. เพิ่มมิติที่ 3 ให้ matrix เช่น `database: [postgres, mysql]` แล้วคำนวณว่า combination ทั้งหมดควรจะมีกี่ตัวหลังจาก exclude/include
2. ลองเปลี่ยน `fail-fast: false` เป็น `true` แล้วสังเกตว่า job อื่นถูกยกเลิกทันทีเมื่อมีตัวใดตัวหนึ่งล้มเหลว
3. เพิ่ม step ที่ใช้ `actions/upload-artifact` เพื่อเก็บผล test report ของแต่ละ combination แยกกัน (จะเรียนละเอียดใน Part 69)
4. ลองเปลี่ยนเงื่อนไขใน `deploy` ให้รองรับทั้งการ deploy จาก branch `main` และจาก git tag ที่ขึ้นต้นด้วย `v` (เช่น `v1.0.0`) โดยใช้ `startsWith(github.ref, 'refs/tags/v')`

---

## สรุป Part 68

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Job หลายตัวในไฟล์เดียวรันแบบขนาน (parallel) โดย default** ในขณะที่ step ภายใน job เดียวกันรันตามลำดับ (sequential) เสมอ และแต่ละ job อยู่บนคนละ runner จึงไม่แชร์ filesystem กัน
2. **`needs:`** ใช้กำหนดลำดับความสัมพันธ์ระหว่าง job สร้างเป็น DAG ได้ และถ้า job ที่ถูกรออยู่ล้มเหลว job ที่ตามมาจะถูก skip โดยอัตโนมัติ เว้นแต่จะใช้ `if: always()`
3. **Matrix build** ช่วยให้ทดสอบหลาย version/OS/environment ได้จาก job ต้นแบบเพียงตัวเดียว โดยอาศัยหลักการ cartesian product
4. **`strategy.matrix`** รองรับได้หลายมิติ และค่าที่ประกาศเข้าถึงได้ผ่าน `${{ matrix.<key> }}` ในทุกจุดของ workflow รวมถึง `runs-on:` ด้วย
5. **`matrix.exclude`** ตัด combination ที่ไม่ต้องการออก ส่วน **`matrix.include`** ใช้ทั้งเพิ่ม combination ใหม่ที่ไม่ได้อยู่ใน cartesian product เดิม หรือเพิ่ม field พิเศษให้ combination ที่มีอยู่แล้ว
6. **`fail-fast`** (default `true`) ยกเลิก job อื่นในทันทีเมื่อมีตัวใดตัวหนึ่งล้มเหลว ส่วน **`max-parallel`** จำกัดจำนวน job ที่รันพร้อมกันในแต่ละช่วงเวลา
7. **`if:`** ควบคุมการรันแบบมีเงื่อนไข ทั้งระดับ job และ step โดยใช้ context เช่น `github.ref`, `github.event_name` ร่วมกับฟังก์ชันสถานะ `success()`, `failure()`, `always()`, `cancelled()`
8. **Job outputs** ส่งค่าระหว่าง job ผ่าน 3 ชั้น: step เขียนค่าไปที่ `$GITHUB_OUTPUT` → job ประกาศ `outputs:` → job อื่นอ่านผ่าน `needs.<job_id>.outputs.<name>`
9. **`timeout-minutes`** จำกัดเวลาการรันสูงสุด (default 360 นาที) ทั้งระดับ job และ step ส่วน **`continue-on-error`** ทำให้ error บางจุดไม่ทำให้ workflow ทั้งหมดล้มเหลว ซึ่งมีประโยชน์มากเมื่อใช้ร่วมกับ combination ที่เป็น experimental ใน matrix
10. เราได้ลงมือเขียน workflow แบบครบวงจรที่ผสมทุกเทคนิคเข้าด้วยกัน: หลาย job ที่มีความสัมพันธ์กัน, matrix build ที่มีทั้ง exclude/include, job outputs, เงื่อนไขการ deploy ที่ปลอดภัย และการจัดการ timeout/error อย่างเหมาะสม

### Checklist ก่อนไป Part 69

ก่อนไปต่อ ให้ตรวจสอบว่าคุณ:

- [ ] อธิบายได้ว่าทำไม job หลายตัวถึงรันขนานกันโดย default และ step ถึงรันตามลำดับเสมอ
- [ ] เขียน `needs:` เพื่อสร้าง pipeline ที่มีลำดับขั้นตอนได้ ทั้งแบบ single dependency และ multiple dependency
- [ ] อธิบายได้ว่า matrix build คืออะไร และแก้ปัญหาอะไรเทียบกับการเขียน job ซ้ำ ๆ
- [ ] เขียน `strategy.matrix` แบบหลายมิติได้ และเข้าใจหลักการ cartesian product
- [ ] ใช้ `exclude` ตัด combination ที่ไม่ต้องการ และใช้ `include` เพิ่ม combination พิเศษได้ถูกต้อง
- [ ] เข้าใจความแตกต่างระหว่าง `fail-fast: true` (default) กับ `fail-fast: false` และรู้ว่า `max-parallel` ทำหน้าที่อะไร
- [ ] เขียน `if:` โดยใช้ context และฟังก์ชันสถานะ (`success()`, `failure()`, `always()`, `cancelled()`) ได้อย่างถูกต้อง
- [ ] ส่งค่าระหว่าง job ด้วย `outputs:` และ `$GITHUB_OUTPUT` ได้ครบทั้ง 3 ชั้น
- [ ] ตั้งค่า `timeout-minutes` และ `continue-on-error` ได้อย่างเหมาะสมกับสถานการณ์ที่ต่างกัน
- [ ] เขียน workflow แบบครบวงจรที่ผสมทุกเทคนิคในบทนี้เข้าด้วยกันได้ด้วยตัวเอง

**ต่อไป:** [Part 69: GitHub Actions: Secrets, Environments, Artifacts](./part-069-github-actions-secrets-environments.md)
