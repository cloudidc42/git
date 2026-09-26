# Part 67: GitHub Actions เบื้องต้น: Workflow แรกของคุณ

> **Step ในหลักสูตรนี้:** Step 661–670
> **เฟส:** 7 — CI/CD เต็มรูปแบบด้วย GitHub Actions และ GitLab CI/CD
> **เป้าหมายของ Part นี้:** เข้าใจว่า GitHub Actions คืออะไร มาจากไหน โครงสร้างไฟล์ workflow ต้องวางไว้ตรงไหน คีย์เวิร์ดหลักอย่าง `on`, `jobs`, `steps`, `uses`, `run`, `runs-on` ทำหน้าที่อะไร วิธีดู log การรันบน GitHub UI วิธีติด status badge ใน README และปิดท้ายด้วยการลงมือเขียน workflow แรกที่ checkout โค้ดแล้วรัน test อัตโนมัติทุกครั้งที่มี push

---

## สารบัญของ Part นี้

- Step 661: GitHub Actions คืออะไร ประวัติโดยย่อ
- Step 662: โครงสร้างไฟล์ workflow (`.github/workflows/*.yml`) ตำแหน่งที่ถูกต้อง
- Step 663: `on:` — กำหนด trigger ว่า workflow จะรันเมื่อไหร่
- Step 664: `jobs:` และ `steps:` โครงสร้างพื้นฐานของไฟล์ YAML
- Step 665: `uses:` — เรียกใช้ Action สำเร็จรูปจาก Marketplace
- Step 666: `run:` — รันคำสั่ง shell ตรง ๆ ภายใน step
- Step 667: `runs-on:` — เลือก runner ที่จะใช้รันงาน
- Step 668: การดู workflow run และ log บน GitHub UI (แท็บ Actions)
- Step 669: Workflow status badge ติดใน README.md
- Step 670: แบบฝึกหัด — เขียน workflow แรกที่ checkout code แล้วรัน test อัตโนมัติ

---

## Step 661: GitHub Actions คืออะไร ประวัติโดยย่อ

หลังจากที่เราใช้เวลาทั้งเฟส 6 (Step 551–650) เจาะลึก Git ในระดับ Internals, Hooks, LFS และ Performance กันไปแล้ว ตอนนี้เราจะเข้าสู่เฟสใหม่ที่น่าตื่นเต้นที่สุดเฟสหนึ่งของหลักสูตร นั่นคือ **เฟส 7: CI/CD เต็มรูปแบบ** ซึ่งจะพาคุณไปรู้จักกับเครื่องมือที่เปลี่ยนวิธีการทำงานของทีมพัฒนาซอฟต์แวร์ทั่วโลกไปอย่างสิ้นเชิง นั่นคือ **GitHub Actions**

### GitHub Actions คืออะไร

**GitHub Actions** คือ **บริการ CI/CD (Continuous Integration / Continuous Deployment) ที่ฝังอยู่ในตัว GitHub เอง** พูดง่าย ๆ คือมันเป็นระบบที่ทำให้คุณสามารถ **"สั่งให้คอมพิวเตอร์ของ GitHub ทำงานอัตโนมัติ"** ทุกครั้งที่มีเหตุการณ์บางอย่างเกิดขึ้นกับ repository ของคุณ เช่น:

- มีคน push โค้ดใหม่เข้ามา
- มีคนเปิด Pull Request
- ถึงเวลาที่กำหนดไว้ล่วงหน้า (เช่น ทุกเที่ยงคืน)
- มีคนกดปุ่ม "Run workflow" ด้วยตัวเอง

เมื่อเหตุการณ์เหล่านี้เกิดขึ้น GitHub จะไปสั่ง **เครื่องเสมือน (Virtual Machine)** ขึ้นมาเครื่องหนึ่งโดยอัตโนมัติ แล้วรันชุดคำสั่งที่คุณเขียนไว้ล่วงหน้า เช่น:

- รัน test suite ทั้งหมดของโปรเจกต์
- ตรวจสอบ code style (lint)
- Build โปรเจกต์
- Deploy ขึ้น production
- ส่งข้อความแจ้งเตือนไปยัง Slack
- สร้าง release อัตโนมัติ

ทั้งหมดนี้เกิดขึ้น **โดยไม่ต้องมีใครนั่งรันคำสั่งเองเลยแม้แต่ครั้งเดียว**

### ทำไมเรื่องนี้ถึงสำคัญมาก

ก่อนมี CI/CD ทีมพัฒนาซอฟต์แวร์ต้องพึ่งพา "วินัยของมนุษย์" ในการรัน test ก่อน push ทุกครั้ง ซึ่งในความเป็นจริงแล้วมนุษย์ลืมบ่อยมาก ผลที่ตามมาคือ:

- โค้ดที่พังหลุดเข้าไปใน branch หลักโดยไม่มีใครรู้ทันที
- ต้องมานั่งไล่หาว่า commit ไหนที่ทำให้ระบบพัง
- การ deploy ขึ้น production ต้องทำมือทุกครั้ง เสี่ยงต่อความผิดพลาด

GitHub Actions แก้ปัญหานี้โดยการทำให้ **การตรวจสอบคุณภาพโค้ดและการ deploy เป็นเรื่องอัตโนมัติ 100%** ทุก push, ทุก Pull Request จะถูกตรวจสอบโดยอัตโนมัติเสมอ ไม่ขึ้นกับว่าคนที่ push จะจำได้หรือไม่ว่าต้องรัน test ก่อน

### ประวัติโดยย่อ

| ปี | เหตุการณ์ |
|---|---|
| 2018 (ตุลาคม) | GitHub เปิดตัว GitHub Actions ในรูปแบบ Beta ครั้งแรก โดยตอนนั้นเน้นไปที่การสร้าง "workflow อัตโนมัติ" ทั่วไป ยังไม่ได้เน้น CI/CD เป็นหลัก |
| 2019 (สิงหาคม) | GitHub Actions เปิดใช้งานแบบ **General Availability (GA)** พร้อมปรับโฟกัสใหม่ให้เน้นไปที่ CI/CD โดยเฉพาะ และเปลี่ยนรูปแบบไฟล์มาเป็น YAML ที่เราใช้กันในปัจจุบัน |
| 2019 (พฤศจิกายน) | เปิดให้ใช้งานฟรีสำหรับ public repository แบบไม่จำกัด และมีโควตาฟรีสำหรับ private repository ด้วย |
| 2020 | เพิ่มความสามารถ self-hosted runner, matrix build เต็มรูปแบบ และการรองรับ macOS/Windows runner |
| 2021–ปัจจุบัน | กลายเป็นหนึ่งในระบบ CI/CD ที่มีคนใช้มากที่สุดในโลก เพราะฝังอยู่ใน GitHub ที่มีนักพัฒนาใช้งานอยู่แล้วหลายสิบล้านคน ไม่ต้องไปตั้งค่า CI/CD service แยกต่างหากอีกต่อไป |

### จุดเด่นที่ทำให้ GitHub Actions ได้รับความนิยมอย่างรวดเร็ว

1. **อยู่ใน GitHub อยู่แล้ว ไม่ต้องเชื่อมต่อ service ภายนอก** — ต่างจากยุคก่อนที่ต้องไปใช้ Travis CI, CircleCI, Jenkins แยกต่างหาก แล้วต้องเชื่อม GitHub เข้ากับ service เหล่านั้นเอง
2. **ฟรีสำหรับ public repository** และมีโควตาฟรีที่ใจกว้างมากสำหรับ private repository (2,000 นาที/เดือนสำหรับบัญชีฟรี ณ เวลาที่เขียนหลักสูตรนี้)
3. **Marketplace ขนาดใหญ่มาก** — มี Action สำเร็จรูปหลายหมื่นตัวให้เรียกใช้ทันที ไม่ต้องเขียนเองตั้งแต่ศูนย์
4. **รองรับหลาย Operating System** — `ubuntu-latest`, `windows-latest`, `macos-latest` ในไฟล์เดียวกัน
5. **เขียนด้วย YAML ที่อ่านง่าย** — ไม่ต้องเรียนภาษาใหม่ ใช้โครงสร้างที่เข้าใจง่ายกว่า Jenkinsfile แบบ Groovy มาก

### เปรียบเทียบกับ CI/CD Service อื่น ๆ

| CI/CD Service | จุดเด่น | จุดสังเกต |
|---|---|---|
| **GitHub Actions** | ฝังอยู่ใน GitHub, Marketplace ใหญ่, ฟรีสำหรับ public repo | ผูกกับ GitHub เป็นหลัก |
| **GitLab CI/CD** | ฝังอยู่ใน GitLab, self-hosted ได้ง่าย | ต้องใช้ GitLab เป็น host |
| **Jenkins** | เก่าแก่ ยืดหยุ่นสูงสุด self-hosted เต็มรูปแบบ | ติดตั้ง/ดูแลเองทั้งหมด ซับซ้อนกว่ามาก |
| **CircleCI / Travis CI** | เคยเป็นผู้นำตลาดก่อนยุค GitHub Actions | ปัจจุบันความนิยมลดลงเพราะ GitHub Actions ครองตลาด |

ในเฟส 7 นี้เราจะโฟกัสที่ GitHub Actions ก่อน (Step 651–ประมาณ 700) แล้วค่อยไปเรียน GitLab CI/CD ในช่วงหลัง เพื่อให้คุณเห็นภาพครบทั้งสองค่าย

---

## Step 662: โครงสร้างไฟล์ workflow (`.github/workflows/*.yml`) ตำแหน่งที่ถูกต้อง

สิ่งแรกที่ต้องรู้ก่อนเขียน workflow คือ **GitHub Actions จะไม่ทำงานเลยถ้าไฟล์ไม่ได้อยู่ในตำแหน่งที่ถูกต้องเป๊ะ ๆ**

### ตำแหน่งไฟล์ที่ถูกต้อง

```
repository-root/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy.yml
│       └── nightly-tests.yml
├── src/
├── tests/
└── README.md
```

กฎเหล็กที่ต้องจำ:

1. โฟลเดอร์ต้องชื่อ **`.github`** (มีจุดนำหน้า) อยู่ที่ **root ของ repository เท่านั้น** ไม่ใช่ในโฟลเดอร์ย่อย
2. ข้างใน `.github` ต้องมีโฟลเดอร์ชื่อ **`workflows`** (พหูพจน์ มี s ท้าย ไม่ใช่ `workflow`) — คนเขียนผิดบ่อยมากว่าลืม `s`
3. ไฟล์ workflow ต้องเป็นนามสกุล **`.yml`** หรือ **`.yaml`** ก็ได้ทั้งคู่ (GitHub รองรับทั้งสองนามสกุล)
4. **1 repository สามารถมีไฟล์ workflow ได้หลายไฟล์** — แต่ละไฟล์คือ workflow แยกกันอิสระ ทำงานคนละเรื่องกันได้ (เช่นไฟล์หนึ่งรัน test อีกไฟล์หนึ่งทำ deploy)
5. ชื่อไฟล์ตั้งอะไรก็ได้ตามใจ (`ci.yml`, `test.yml`, `build-and-deploy.yml`) — GitHub ไม่สนใจชื่อไฟล์ แต่จะอ่านเนื้อหาข้างในเพื่อรู้ว่า workflow นี้ชื่ออะไรและต้องทำอะไร

### โครงร่างขั้นต่ำที่สุดของไฟล์ workflow

```yaml
name: My First Workflow

on: push

jobs:
  hello-world:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello, GitHub Actions!"
```

เพียงเท่านี้ก็เป็น workflow ที่รันได้จริงแล้ว มาดูความหมายของแต่ละบรรทัดคร่าว ๆ ก่อน (รายละเอียดเต็มจะอยู่ใน Step ถัดไป):

| บรรทัด | ความหมาย |
|---|---|
| `name:` | ชื่อของ workflow ที่จะแสดงในแท็บ Actions บนเว็บ GitHub (ถ้าไม่ใส่ GitHub จะใช้ชื่อไฟล์แทน) |
| `on:` | เงื่อนไขว่า workflow นี้จะถูก trigger (สั่งให้รัน) เมื่อไหร่ |
| `jobs:` | รายการของ "งาน" ที่ workflow นี้ต้องทำ (มีได้หลาย job) |
| `hello-world:` | ชื่อของ job นี้ (ตั้งเองได้ตามใจ) |
| `runs-on:` | ระบุว่างานนี้จะรันบน runner (เครื่องเสมือน) แบบไหน |
| `steps:` | รายการขั้นตอนย่อยที่ job นี้จะทำเรียงตามลำดับบนลงล่าง |

### YAML คือภาษาอะไร ทำไม GitHub Actions ถึงใช้

**YAML (YAML Ain't Markup Language)** คือรูปแบบไฟล์สำหรับเก็บข้อมูลแบบมีโครงสร้าง (structured data) ที่เน้นให้มนุษย์อ่านง่าย ต่างจาก JSON หรือ XML ที่เน้นให้เครื่องคอมพิวเตอร์ประมวลผลง่ายเป็นหลัก

กฎสำคัญของ YAML ที่ต้องรู้ก่อนเขียน workflow:

1. **ใช้ indentation (การเยื้องบรรทัด) ในการกำหนดโครงสร้าง** — ห้ามใช้ Tab เด็ดขาด ต้องใช้ **space เท่านั้น** (ปกตินิยมใช้ 2 spaces ต่อระดับ)
2. **`key: value`** คือรูปแบบพื้นฐานที่สุด ต้องมี **space หลังเครื่องหมาย colon (`:`) เสมอ**
3. **`- item`** คือการทำ list (array) — ขีดกลาง (`-`) ตามด้วย space แล้วค่อยเป็นค่า
4. Comment ใช้เครื่องหมาย `#` เหมือน Python/Bash
5. ระดับการเยื้องที่ผิดแม้แต่ 1 space ก็ทำให้ workflow ทำงานผิดพลาดหรือรันไม่ได้เลย — นี่คือสาเหตุอันดับหนึ่งที่มือใหม่เจอปัญหาตอนเขียน workflow ครั้งแรก

### ตัวอย่างการเยื้องที่ถูกต้องเทียบกับผิด

**ถูกต้อง:**

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
```

**ผิด (เยื้องไม่ตรงกัน จะทำให้ YAML parse ไม่ผ่าน):**

```yaml
jobs:
  build:
   runs-on: ubuntu-latest
      steps:
    - name: Checkout code
       uses: actions/checkout@v4
```

ข้อแนะนำ: ใช้ editor ที่มี YAML syntax highlighting และ linter (เช่น VS Code ที่ติดตั้ง extension "YAML" ของ Red Hat) เพื่อให้เห็นทันทีว่าเยื้องผิดตรงไหน จะช่วยประหยัดเวลาการ debug ได้มาก

### GitHub ตรวจจับไฟล์ workflow อย่างไร

เมื่อมีการ push โค้ดขึ้น GitHub หรือมีเหตุการณ์ที่ตรงกับเงื่อนไขใน `on:` ของไฟล์ใด ๆ ในโฟลเดอร์ `.github/workflows/` GitHub จะ:

1. อ่านไฟล์ YAML ทุกไฟล์ในโฟลเดอร์นี้
2. ตรวจสอบว่าไฟล์ไหนมีเงื่อนไข `on:` ที่ตรงกับเหตุการณ์ที่เพิ่งเกิดขึ้น
3. รัน workflow ที่ตรงเงื่อนไขทั้งหมด (อาจมีมากกว่า 1 workflow รันพร้อมกันได้)
4. แสดงผลลัพธ์ในแท็บ **Actions** ของ repository นั้น

หากไฟล์ YAML มี syntax ผิด GitHub จะไม่รัน workflow นั้นเลย และมักจะแสดง error ให้เห็นในแท็บ Actions หรือบางกรณีอาจไม่แสดงอะไรเลยถ้า syntax ผิดร้ายแรงมาก (เช่น indentation ผิดทั้งไฟล์) ดังนั้นการตรวจสอบ YAML ให้ถูกต้องก่อน push จึงสำคัญมาก

---

## Step 663: `on:` — กำหนด trigger เมื่อไหร่ที่ workflow จะรัน

คีย์เวิร์ด **`on:`** คือหัวใจของ workflow ทุกตัว เพราะมันตอบคำถามว่า **"workflow นี้ควรจะรันเมื่อไหร่"**

### รูปแบบพื้นฐานที่สุด — trigger เดียว

```yaml
on: push
```

หมายความว่า ทุกครั้งที่มีการ push โค้ดเข้า repository นี้ (ไม่ว่า branch ไหนก็ตาม) workflow นี้จะถูกรันทันที

### Trigger ยอดนิยมที่ควรรู้จัก

#### 1. `push` — รันเมื่อมีการ push โค้ด

```yaml
on:
  push:
    branches:
      - main
      - develop
```

ตัวอย่างนี้จะรัน workflow **เฉพาะเมื่อมีการ push เข้า branch `main` หรือ `develop` เท่านั้น** ถ้า push เข้า branch อื่น (เช่น `feature/login`) workflow นี้จะไม่ถูกรัน

สามารถระบุ path ที่ต้องการให้เฝ้าดูได้ด้วย:

```yaml
on:
  push:
    branches:
      - main
    paths:
      - 'src/**'
      - 'tests/**'
```

ตัวอย่างนี้จะรัน workflow เฉพาะเมื่อ push เข้า `main` **และ** มีไฟล์ที่เปลี่ยนแปลงอยู่ในโฟลเดอร์ `src/` หรือ `tests/` เท่านั้น — มีประโยชน์มากเมื่อโปรเจกต์ใหญ่และไม่อยากรัน CI ทุกครั้งที่แก้แค่ไฟล์เอกสาร

#### 2. `pull_request` — รันเมื่อมีการเปิดหรืออัปเดต Pull Request

```yaml
on:
  pull_request:
    branches:
      - main
```

นี่คือ trigger ที่สำคัญที่สุดตัวหนึ่งในการทำงานเป็นทีม เพราะทุกครั้งที่มีคนเปิด PR เข้า `main` หรือ push commit ใหม่เพิ่มเข้าไปใน PR ที่เปิดอยู่ workflow นี้จะรันทันที ทำให้ทีมเห็นผลลัพธ์การทดสอบ **ก่อนที่จะกด merge** ซึ่งช่วยป้องกันโค้ดพังไม่ให้เข้าไปใน branch หลักได้อย่างมีประสิทธิภาพมาก

สามารถระบุ activity type ที่ต้องการได้ละเอียดขึ้น:

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened]
    branches:
      - main
```

- `opened` — เมื่อ PR ถูกเปิดใหม่
- `synchronize` — เมื่อมีการ push commit ใหม่เข้า branch ของ PR ที่เปิดอยู่
- `reopened` — เมื่อ PR ที่เคยปิดถูกเปิดใหม่อีกครั้ง

#### 3. `schedule` — รันตามเวลาที่กำหนด (คล้าย cron job)

```yaml
on:
  schedule:
    - cron: '0 0 * * *'
```

รูปแบบ cron มี 5 ช่องคือ `นาที ชั่วโมง วันของเดือน เดือน วันของสัปดาห์` ตัวอย่างข้างต้น `0 0 * * *` หมายถึง **รันทุกวันเวลาเที่ยงคืน (UTC)**

ตารางตัวอย่าง cron expression ที่ใช้บ่อย:

| Cron Expression | ความหมาย |
|---|---|
| `0 0 * * *` | ทุกวัน เที่ยงคืน (UTC) |
| `0 */6 * * *` | ทุก 6 ชั่วโมง |
| `0 9 * * 1-5` | ทุกวันจันทร์-ศุกร์ เวลา 9:00 น. (UTC) |
| `0 0 1 * *` | วันที่ 1 ของทุกเดือน เที่ยงคืน |
| `*/15 * * * *` | ทุก 15 นาที |

**ข้อควรระวังสำคัญ:** เวลาใน `schedule` ของ GitHub Actions เป็น **UTC เสมอ** ไม่ใช่เวลาไทย ถ้าต้องการให้รันตอน 9 โมงเช้าเวลาไทย (UTC+7) ต้องตั้งเป็น `0 2 * * *` (2:00 UTC = 9:00 เวลาไทย) แทน

#### 4. `workflow_dispatch` — รันด้วยตัวเองผ่านปุ่มบนเว็บ

```yaml
on:
  workflow_dispatch:
```

trigger นี้เพิ่มปุ่ม **"Run workflow"** เข้าไปในแท็บ Actions บนเว็บ GitHub ทำให้คุณสามารถกดรัน workflow เองได้ทุกเมื่อโดยไม่ต้องรอ push หรือ schedule มีประโยชน์มากสำหรับ workflow ประเภท deploy ที่อยากควบคุมด้วยมือ

สามารถรับ input จากผู้ใช้ตอนกดรันได้ด้วย:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'สภาพแวดล้อมที่จะ deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
```

เมื่อกดปุ่ม "Run workflow" ผู้ใช้จะเห็น dropdown ให้เลือกว่าจะ deploy ไปที่ staging หรือ production ซึ่งค่านี้จะถูกอ่านในภายหลังผ่าน `${{ github.event.inputs.environment }}`

### รวมหลาย trigger เข้าด้วยกันในไฟล์เดียว

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:
```

Workflow นี้จะถูกรันได้จาก 3 เหตุการณ์: push เข้า main, เปิด/อัปเดต PR เข้า main, หรือกดรันเองผ่านปุ่มบนเว็บ

### ตารางสรุป Trigger ที่ใช้บ่อยที่สุด

| Trigger | รันเมื่อไหร่ | ใช้บ่อยกับ |
|---|---|---|
| `push` | มีการ push commit เข้า repo | CI ทั่วไป, build อัตโนมัติ |
| `pull_request` | เปิด/อัปเดต PR | ตรวจสอบโค้ดก่อน merge |
| `schedule` | ตามเวลาที่กำหนด (cron) | รัน test กลางดึก, ทำ backup, เช็ค dependency ที่ล้าสมัย |
| `workflow_dispatch` | กดปุ่มรันเอง | Deploy แบบควบคุมด้วยมือ |
| `release` | สร้าง release ใหม่ | สร้าง build artifact ตอนออกเวอร์ชันใหม่ |
| `issues` | มีการเปิด/แก้ไข issue | อัตโนมัติแปะ label, ตอบข้อความต้อนรับ |

---

## Step 664: `jobs:` และ `steps:` โครงสร้างพื้นฐานของไฟล์ YAML

ถัดจาก `on:` มาดูสองคีย์เวิร์ดที่เป็นแกนกลางของ "การทำงานจริง" ใน workflow นั่นคือ **`jobs:`** และ **`steps:`**

### ความสัมพันธ์ระหว่าง Workflow → Job → Step

```
Workflow (ไฟล์ .yml 1 ไฟล์)
│
├── Job 1 (รันบน runner เครื่องหนึ่ง)
│   ├── Step 1
│   ├── Step 2
│   └── Step 3
│
└── Job 2 (รันบน runner อีกเครื่องหนึ่ง แยกอิสระจาก Job 1)
    ├── Step 1
    └── Step 2
```

หลักการสำคัญที่ต้องเข้าใจ:

1. **1 workflow มีได้หลาย job** — แต่ละ job รันบน **runner (เครื่องเสมือน) แยกกันคนละเครื่อง** โดย default job ทั้งหมดจะรัน **พร้อมกัน (parallel)** ยกเว้นจะตั้งค่าให้รอกัน (`needs:`)
2. **1 job มีได้หลาย step** — step ในแต่ละ job จะรัน **เรียงตามลำดับ (sequential)** จากบนลงล่างเสมอ ถ้า step ไหนล้มเหลว (exit code ไม่ใช่ 0) step ที่เหลือใน job นั้นจะถูกข้าม (ยกเว้นตั้งค่า `continue-on-error` หรือ `if:` เป็นอย่างอื่น)
3. **แต่ละ job เริ่มต้นจาก environment ที่สะอาด (clean)** — ไม่มีไฟล์อะไรอยู่เลยจนกว่าจะสั่ง checkout โค้ดเอง

### โครงสร้าง `jobs:` แบบละเอียด

```yaml
jobs:
  build:
    name: Build Project
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install dependencies
        run: npm install

      - name: Build
        run: npm run build

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install dependencies
        run: npm install

      - name: Run test suite
        run: npm test
```

ในตัวอย่างนี้มี 2 job คือ `build` และ `test` ซึ่ง **รันพร้อมกัน** เพราะไม่มีการระบุ `needs:` ให้รอกัน สังเกตว่าคีย์ `build` และ `test` (ที่อยู่ใต้ `jobs:`) คือ **job ID** ที่ตั้งชื่อเอง ส่วน `name:` ข้างในคือชื่อที่แสดงผลบนหน้าเว็บ (จะสวยกว่า/อ่านง่ายกว่า job ID ก็ได้)

### การทำให้ job รอกัน ด้วย `needs:`

ถ้าต้องการให้ `test` รันหลังจาก `build` เสร็จสมบูรณ์แล้วเท่านั้น (ไม่ใช่รันพร้อมกัน) ให้เพิ่ม `needs:`

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install
      - run: npm run build

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install
      - run: npm test
```

ตอนนี้ `test` จะรอให้ `build` เสร็จและสำเร็จก่อน ถึงจะเริ่มรัน ถ้า `build` ล้มเหลว `test` จะไม่ถูกรันเลย (ถูกข้ามและแสดงสถานะ "skipped")

เรื่อง Job dependency, matrix build และการรัน job แบบขนานเชิงลึกจะอธิบายอย่างละเอียดใน **Part 68: GitHub Actions: Jobs, Steps, Matrix Build** ต่อจาก Part นี้

### โครงสร้าง `steps:` แบบละเอียด

แต่ละ step ใน `steps:` list สามารถมี field ต่อไปนี้ได้:

| Field | ความหมาย | จำเป็นไหม |
|---|---|---|
| `name:` | ชื่อของ step ที่จะแสดงในหน้า log บนเว็บ | ไม่จำเป็น (แต่แนะนำให้ใส่เสมอเพื่อความชัดเจน) |
| `uses:` | เรียกใช้ Action สำเร็จรูป | ใช้แทน `run:` (เลือกอย่างใดอย่างหนึ่ง) |
| `run:` | รันคำสั่ง shell ตรง ๆ | ใช้แทน `uses:` (เลือกอย่างใดอย่างหนึ่ง) |
| `with:` | ส่ง parameter เข้าไปให้ Action ที่เรียกผ่าน `uses:` | ใช้เมื่อ Action นั้นต้องการ input |
| `env:` | กำหนด environment variable เฉพาะ step นั้น | ไม่จำเป็น |
| `if:` | เงื่อนไขว่า step นี้จะรันหรือไม่ | ไม่จำเป็น |
| `id:` | ตั้งชื่ออ้างอิงเพื่อดึงผลลัพธ์ของ step นี้ไปใช้ใน step ถัดไป | ไม่จำเป็น |

ตัวอย่างการใช้ `id:` และ `if:` ร่วมกัน:

```yaml
steps:
  - name: Check file exists
    id: check
    run: |
      if [ -f "config.yml" ]; then
        echo "exists=true" >> "$GITHUB_OUTPUT"
      else
        echo "exists=false" >> "$GITHUB_OUTPUT"
      fi

  - name: Warn if config missing
    if: steps.check.outputs.exists == 'false'
    run: echo "::warning::ไม่พบไฟล์ config.yml ในโปรเจกต์"
```

### ข้อควรจำเรื่อง Working Directory ระหว่าง Job

สิ่งที่มือใหม่มักสับสนคือ: **แต่ละ job เริ่มต้นจาก working directory ที่ว่างเปล่าเสมอ** ไม่มีไฟล์โค้ดอะไรเลยจนกว่าจะสั่ง `actions/checkout` (จะอธิบายละเอียดใน Step 665) และไฟล์ที่สร้างขึ้นระหว่างการรันของ job หนึ่ง **จะไม่ถูกส่งต่อไปยังอีก job หนึ่งโดยอัตโนมัติ** ถ้าต้องการส่งไฟล์ข้าม job ต้องใช้ `actions/upload-artifact` และ `actions/download-artifact` ซึ่งจะเรียนในรายละเอียดใน Part ถัดไป

---

## Step 665: `uses:` — เรียกใช้ Action สำเร็จรูปจาก Marketplace

หนึ่งในจุดแข็งที่สุดของ GitHub Actions คือ **GitHub Marketplace** ซึ่งเป็นคลังของ "Action" สำเร็จรูปที่คนทั่วโลกเขียนไว้ให้เรียกใช้ได้ทันที โดยไม่ต้องเขียนโค้ดเองตั้งแต่ศูนย์

### `uses:` คืออะไร

คีย์เวิร์ด **`uses:`** ใช้เพื่อบอกว่า step นี้จะเรียกใช้ **Action** (แพ็กเกจโค้ดสำเร็จรูปที่ทำหน้าที่เฉพาะทาง) แทนที่จะเขียนคำสั่ง shell เอง

รูปแบบทั่วไป:

```yaml
- uses: <owner>/<repo>@<version>
```

หรือถ้าเป็น Action ในหมวดหมู่ทางการของ GitHub เอง มักขึ้นต้นด้วย `actions/`

### `actions/checkout@v4` — Action ที่ต้องมีเกือบทุก workflow

นี่คือ Action ที่สำคัญที่สุดตัวหนึ่งในระบบนิเวศทั้งหมดของ GitHub Actions เพราะอย่างที่บอกไว้ใน Step 664 ว่า **job ทุกตัวเริ่มต้นจาก working directory ที่ว่างเปล่า** — ไม่มีโค้ดของ repository อยู่เลยแม้แต่บรรทัดเดียว

`actions/checkout@v4` ทำหน้าที่ **clone repository ของคุณเข้ามาไว้ใน runner** เพื่อให้ step ถัดไปสามารถเข้าถึงไฟล์โค้ดได้

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v4
```

แค่บรรทัดเดียวนี้ ทำสิ่งต่อไปนี้ให้อัตโนมัติทั้งหมด:

1. Clone repository ของคุณ (เฉพาะ commit ที่ trigger workflow นี้)
2. Checkout ไปยัง branch/commit ที่ถูกต้องตามเหตุการณ์ที่เกิดขึ้น
3. วางไฟล์ทั้งหมดไว้ใน working directory ปัจจุบันของ runner พร้อมให้ step ถัดไปใช้งานได้ทันที

หากไม่มี step นี้ ทุก step ที่ตามมาจะไม่เห็นไฟล์โค้ดของโปรเจกต์เลย เพราะ runner เป็นเครื่องเปล่าที่เพิ่งถูกสร้างขึ้นมาใหม่ทุกครั้ง

### ทำไมต้องระบุเวอร์ชัน (`@v4`)

สังเกตว่าเราเขียน `actions/checkout@v4` ไม่ใช่แค่ `actions/checkout` เฉย ๆ ส่วน `@v4` คือการระบุ **เวอร์ชันของ Action** ที่ต้องการใช้ ซึ่งสำคัญมากเพราะ:

1. **ป้องกัน Action เปลี่ยนพฤติกรรมโดยไม่รู้ตัว** — ถ้าผู้พัฒนา Action ปล่อยเวอร์ชันใหม่ที่มี breaking change workflow ของคุณจะไม่ได้รับผลกระทบถ้า pin เวอร์ชันไว้
2. **เพื่อความปลอดภัย** — การไม่ระบุเวอร์ชัน (หรือใช้ `@main` ตรง ๆ) เสี่ยงต่อการถูกโจมตีแบบ supply chain attack ถ้า repository ต้นทางของ Action ถูก compromise

รูปแบบการระบุเวอร์ชันที่พบบ่อย:

| รูปแบบ | ความหมาย | แนะนำหรือไม่ |
|---|---|---|
| `actions/checkout@v4` | Major version tag — ได้ patch/minor update อัตโนมัติ | แนะนำ ใช้กันมากที่สุด |
| `actions/checkout@v4.1.1` | เวอร์ชันเป๊ะ ๆ ไม่มีการอัปเดตอัตโนมัติเลย | ใช้เมื่อต้องการความเสถียรสูงสุด |
| `actions/checkout@main` | ใช้โค้ดล่าสุดจาก branch main เสมอ | **ไม่แนะนำ** เสี่ยงเรื่อง breaking change และความปลอดภัย |
| `actions/checkout@8f4b7f8...` (SHA เต็ม) | ระบุ commit เป๊ะ ๆ ปลอดภัยที่สุด | ใช้ในองค์กรที่เข้มงวดเรื่อง security |

### ตัวอย่าง Action ยอดนิยมอื่น ๆ ใน Marketplace

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v4

  - name: Setup Node.js
    uses: actions/setup-node@v4
    with:
      node-version: '20'

  - name: Setup Python
    uses: actions/setup-python@v5
    with:
      python-version: '3.12'

  - name: Cache dependencies
    uses: actions/cache@v4
    with:
      path: ~/.npm
      key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

  - name: Upload build artifact
    uses: actions/upload-artifact@v4
    with:
      name: build-output
      path: dist/
```

### `with:` — การส่ง parameter เข้า Action

สังเกตว่า Action บางตัวต้องการ input เพิ่มเติม เช่น `actions/setup-node@v4` ต้องการรู้ว่าจะติดตั้ง Node.js เวอร์ชันไหน ซึ่งส่งผ่าน `with:`

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'
```

แต่ละ Action จะมี input parameter ของตัวเองแตกต่างกัน สามารถดูรายละเอียดได้จากหน้า README ของ Action นั้นบน GitHub Marketplace (https://github.com/marketplace/actions) หรือหน้า repository ของ Action นั้นโดยตรง

### วิธีค้นหา Action ที่ต้องการจาก Marketplace

1. เข้าไปที่ https://github.com/marketplace?type=actions
2. ค้นหาด้วยคำที่เกี่ยวข้อง เช่น "docker build", "slack notify", "deploy aws"
3. ตรวจสอบ **จำนวนดาว (star)**, **Verified creator badge** (สำหรับ Action จากบริษัทที่ยืนยันตัวตนแล้ว) และ **วันที่อัปเดตล่าสุด** ก่อนนำมาใช้ในโปรเจกต์จริง เพื่อความปลอดภัยและความน่าเชื่อถือ

---

## Step 666: `run:` — รันคำสั่ง shell ตรงๆ ภายใน step

ในขณะที่ `uses:` เรียก Action สำเร็จรูปจากคนอื่น คีย์เวิร์ด **`run:`** ให้คุณ **รันคำสั่ง shell ของคุณเองตรง ๆ** ภายใน step นั้น เหมือนกับพิมพ์คำสั่งใน terminal

### รูปแบบพื้นฐาน — คำสั่งเดียว

```yaml
steps:
  - name: Print current directory
    run: pwd
```

### รูปแบบหลายบรรทัด — ใช้ `|` (block scalar)

เมื่อต้องการรันหลายคำสั่งเรียงกันใน step เดียว ใช้เครื่องหมาย `|` (pipe) เพื่อบอก YAML ว่านี่คือ multi-line string:

```yaml
steps:
  - name: Setup and build
    run: |
      echo "เริ่มติดตั้ง dependencies..."
      npm install
      echo "เริ่ม build..."
      npm run build
      echo "เสร็จสิ้น"
```

แต่ละบรรทัดใน block นี้จะถูกรันเรียงตามลำดับ เสมือนเขียนเป็น shell script บรรทัดต่อบรรทัด และถ้าคำสั่งไหนล้มเหลว (return exit code ที่ไม่ใช่ 0) step ทั้งหมดจะถือว่าล้มเหลวทันที (หยุดรันคำสั่งที่เหลือใน block นั้น)

### ตัวอย่างการรัน test จริงด้วย `run:`

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test

      - name: Build project
        run: npm run build
```

Workflow นี้แสดงให้เห็นว่า `uses:` และ `run:` ทำงานผสมกันได้อย่างลงตัวใน job เดียว — ใช้ Action สำเร็จรูปสำหรับงานที่ซับซ้อน (checkout, setup Node.js) และใช้ `run:` สำหรับคำสั่งเฉพาะของโปรเจกต์ (npm install/test/build)

### การกำหนด shell ที่ใช้รัน

โดย default บน `ubuntu-latest` และ `macos-latest`, `run:` จะใช้ **bash** เป็น shell แต่บน `windows-latest` จะใช้ **PowerShell** เป็น default ถ้าต้องการเปลี่ยน shell สามารถระบุได้:

```yaml
steps:
  - name: Run with specific shell
    shell: bash
    run: |
      echo "บังคับให้ใช้ bash แม้จะรันบน Windows runner"
```

ตัวเลือก `shell:` ที่รองรับ: `bash`, `pwsh`, `python`, `sh`, `cmd`, `powershell`

### `working-directory:` — เปลี่ยนโฟลเดอร์ที่คำสั่งจะรัน

ถ้าโปรเจกต์มีโครงสร้างแบบ monorepo ที่โค้ดจริงอยู่ในโฟลเดอร์ย่อย สามารถระบุได้ว่า `run:` นี้ให้รันจากโฟลเดอร์ไหน:

```yaml
steps:
  - name: Install dependencies in backend folder
    working-directory: ./backend
    run: npm install
```

### การใช้ Environment Variable ใน `run:`

```yaml
steps:
  - name: Show environment info
    env:
      APP_NAME: my-awesome-app
    run: |
      echo "กำลัง build แอป: $APP_NAME"
      echo "รันบน branch: $GITHUB_REF_NAME"
      echo "commit SHA: $GITHUB_SHA"
```

GitHub Actions มี environment variable สำเร็จรูปให้ใช้เสมอ (เรียกว่า **default environment variables**) เช่น `GITHUB_REF_NAME`, `GITHUB_SHA`, `GITHUB_REPOSITORY`, `GITHUB_ACTOR` โดยไม่ต้องประกาศเอง — รายการแบบเต็มจะอธิบายละเอียดในบทถัดไปของเฟสนี้

### ข้อแตกต่างสำคัญระหว่าง `uses:` กับ `run:`

| | `uses:` | `run:` |
|---|---|---|
| ใช้ทำอะไร | เรียก Action สำเร็จรูปที่คนอื่นเขียนไว้ | รันคำสั่ง shell ของคุณเอง |
| เหมาะกับ | งานที่ซับซ้อน มีคนทำสำเร็จรูปไว้แล้ว (checkout, setup environment, deploy) | คำสั่งเฉพาะของโปรเจกต์ (npm test, pytest, make build) |
| ตัวอย่าง | `uses: actions/checkout@v4` | `run: npm test` |
| รับ input ผ่าน | `with:` | environment variable หรือ argument ต่อท้ายคำสั่ง |

หลักการทั่วไปคือ: **ถ้ามี Action สำเร็จรูปที่น่าเชื่อถือทำสิ่งที่ต้องการอยู่แล้ว ให้ใช้ `uses:` เพราะปลอดภัยกว่าและทดสอบมาดีแล้ว ส่วนคำสั่งเฉพาะของโปรเจกต์ตัวเอง (เช่น test script) ให้ใช้ `run:``**

---

## Step 667: `runs-on:` — เลือก runner (`ubuntu-latest`, `windows-latest`, `macos-latest`)

**`runs-on:`** คือคีย์เวิร์ดที่บอกว่า job นี้จะรันบน **runner** แบบไหน — runner คือเครื่องเสมือน (Virtual Machine) ที่ GitHub เตรียมไว้ให้ใช้ฟรี (สำหรับ public repository) หรือหักโควตาจากบัญชี (สำหรับ private repository)

### GitHub-hosted Runner ที่ใช้บ่อยที่สุด

```yaml
jobs:
  build-on-linux:
    runs-on: ubuntu-latest
    steps:
      - run: echo "รันบน Linux"

  build-on-windows:
    runs-on: windows-latest
    steps:
      - run: echo "รันบน Windows"

  build-on-mac:
    runs-on: macos-latest
    steps:
      - run: echo "รันบน macOS"
```

| Runner Label | Operating System | ใช้เมื่อไหร่ |
|---|---|---|
| `ubuntu-latest` | Ubuntu Linux เวอร์ชันล่าสุดที่ GitHub รองรับ | ตัวเลือก default ที่ใช้บ่อยที่สุด เร็วที่สุด และประหยัดโควตานาทีที่สุด (คิด 1x) |
| `windows-latest` | Windows Server เวอร์ชันล่าสุด | เมื่อต้องทดสอบแอปที่รันบน Windows โดยเฉพาะ (คิดโควตา 2x ของ Linux) |
| `macos-latest` | macOS เวอร์ชันล่าสุด | เมื่อต้อง build แอป iOS/macOS ที่ต้องใช้ Xcode (คิดโควตาแพงที่สุด 10x ของ Linux) |

**ข้อสังเกตสำคัญเรื่องโควตา:** สำหรับบัญชีแบบมี private repository ที่ใช้โควตาฟรีรายเดือน runner แต่ละแบบใช้ "อัตราคูณ" ไม่เท่ากัน การรันบน `windows-latest` 1 นาทีจะถูกหักโควตาเท่ากับรันบน `ubuntu-latest` 2 นาที ส่วน `macos-latest` จะถูกหักแพงที่สุดถึง 10 เท่า ดังนั้นถ้าไม่จำเป็นต้องทดสอบบน macOS/Windows จริง ๆ ควรใช้ `ubuntu-latest` เป็นหลักเพื่อประหยัดโควตา (สำหรับ public repository ยังคงฟรีไม่จำกัดเช่นเดิม)

### การระบุเวอร์ชันเฉพาะเจาะจงของ OS

นอกจาก `-latest` แล้วยังสามารถระบุเวอร์ชันเฉพาะได้ ในกรณีที่ต้องการความเสถียรแบบเป๊ะ ๆ ไม่อยากให้ environment เปลี่ยนแปลงตาม GitHub อัปเดต:

```yaml
jobs:
  build:
    runs-on: ubuntu-22.04
```

```yaml
jobs:
  build:
    runs-on: windows-2022
```

การใช้ `-latest` มีข้อดีคือได้ patch ความปลอดภัยล่าสุดเสมอโดยอัตโนมัติ แต่มีความเสี่ยงเล็กน้อยว่า GitHub อาจเปลี่ยนเวอร์ชัน default ในอนาคตแล้วทำให้ environment ของคุณเปลี่ยนไปโดยไม่ตั้งใจ — สำหรับโปรเจกต์ทั่วไป การใช้ `-latest` ยังคงเป็นทางเลือกที่แนะนำเพราะสะดวกและอัปเดตอัตโนมัติ

### Matrix — รันหลาย OS พร้อมกันในไฟล์เดียว

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - run: echo "ทดสอบบน ${{ matrix.os }}"
```

Workflow นี้จะสร้าง **3 job แยกกัน** ที่รันบน 3 OS พร้อมกัน โดยใช้โค้ด step เดียวกันทั้งหมด นี่คือหลักการเบื้องต้นของ **Matrix Build** ซึ่งเป็นเนื้อหาหลักของ **Part 68** ที่จะเรียนถัดจาก Part นี้ทันที

### Self-hosted Runner (แนะนำเบื้องต้น)

นอกจาก runner ที่ GitHub เตรียมให้ (GitHub-hosted runner) ยังสามารถใช้ **self-hosted runner** คือเครื่องของคุณเอง (หรือ server ของบริษัท) มาลงทะเบียนเป็น runner ได้เอง:

```yaml
jobs:
  build:
    runs-on: self-hosted
```

Self-hosted runner มีประโยชน์เมื่อ:
- ต้องการ hardware เฉพาะทาง (เช่น GPU สำหรับงาน Machine Learning)
- ต้องการเข้าถึง network ภายในองค์กรที่ GitHub-hosted runner เข้าไม่ถึง
- ต้องการควบคุม environment อย่างละเอียด หรือประหยัดค่าใช้จ่ายในระยะยาวสำหรับงานที่รันบ่อยมาก

รายละเอียดการติดตั้งและตั้งค่า self-hosted runner แบบเต็มจะอยู่ในเฟสถัดไปของหลักสูตร (เฟส 8: DevOps ระดับองค์กร) — สำหรับ Part นี้ขอให้รู้จักไว้แค่ว่ามันมีอยู่และใช้คีย์เวิร์ดเดียวกันคือ `runs-on:`

### ตารางสรุป `runs-on:`

| การใช้งาน | ตัวอย่างค่า |
|---|---|
| Linux ล่าสุด (แนะนำเป็น default) | `runs-on: ubuntu-latest` |
| Windows ล่าสุด | `runs-on: windows-latest` |
| macOS ล่าสุด | `runs-on: macos-latest` |
| เวอร์ชันเฉพาะเจาะจง | `runs-on: ubuntu-22.04` |
| Matrix (หลาย OS) | `runs-on: ${{ matrix.os }}` |
| Self-hosted | `runs-on: self-hosted` |

---

## Step 668: การดู workflow run และ log บน GitHub UI (แท็บ Actions)

เมื่อ workflow ถูก trigger แล้ว เราจะไปดูผลลัพธ์การรันได้ที่ไหน คำตอบคือ **แท็บ Actions** บนหน้า repository ของ GitHub

### วิธีเข้าไปดูแท็บ Actions

1. เปิด repository บน GitHub
2. คลิกแท็บ **Actions** ที่อยู่บนแถบเมนูด้านบน (อยู่ระหว่างแท็บ "Pull requests" และ "Projects" โดยประมาณ)
3. จะเห็นรายการ workflow ทั้งหมดที่เคยรันมา เรียงจากล่าสุดไปเก่าสุด

### โครงสร้างของหน้า Actions

```
Actions
├── All workflows (แสดงทุก workflow ที่รันมาทั้งหมด)
│   ├── ✅ CI #42 — push by phutjirakul — 2 นาทีที่แล้ว
│   ├── ❌ CI #41 — pull_request by contributor-x — 1 ชั่วโมงที่แล้ว
│   └── ✅ Nightly Tests #15 — schedule — 8 ชั่วโมงที่แล้ว
│
└── Workflows (แถบซ้าย แยกตามชื่อไฟล์ workflow)
    ├── CI
    ├── Deploy
    └── Nightly Tests
```

### สัญลักษณ์สถานะที่ต้องรู้จัก

| สัญลักษณ์ | ความหมาย |
|---|---|
| 🟡 (วงกลมสีเหลือง หมุน) | กำลังรันอยู่ (in progress) |
| ✅ (เครื่องหมายถูกสีเขียว) | รันสำเร็จทุก job (success) |
| ❌ (กากบาทสีแดง) | มี job หรือ step ล้มเหลว (failure) |
| ⚪ (วงกลมเทา) | ถูกยกเลิก (cancelled) |
| ⏭️ (ลูกศรข้าม) | ถูกข้ามเพราะเงื่อนไข `if:` ไม่ผ่าน หรือ `needs:` ล้มเหลว (skipped) |

### การเจาะลึกเข้าไปดู log แต่ละ step

เมื่อคลิกเข้าไปที่ workflow run ใดตัวหนึ่ง จะเห็นหน้าจอแสดง job ทั้งหมดของ run นั้น (แสดงเป็นกล่องเรียงต่อกันถ้ามีการใช้ `needs:` หรือเรียงขนานกันถ้ารันพร้อมกัน) คลิกเข้าไปในแต่ละ job จะเห็นรายการ step ทั้งหมดเรียงตามลำดับ พร้อมเวลาที่ใช้ในแต่ละ step

คลิกที่ step ใดตัวหนึ่งจะขยาย **log แบบเต็ม** ของ step นั้นออกมา ซึ่งเป็นข้อความเดียวกันกับที่จะเห็นถ้ารันคำสั่งเดียวกันบน terminal ของตัวเอง มีประโยชน์มากในการหาสาเหตุตอน workflow ล้มเหลว

### เทคนิคการอ่าน log ให้มีประสิทธิภาพ

1. **มองหา step ที่มีสัญลักษณ์ ❌ ก่อนเสมอ** — ไม่ต้องไล่อ่าน log ทุก step ตั้งแต่บนลงล่าง ให้กระโดดไปดู step ที่ล้มเหลวก่อน
2. **ใช้ปุ่ม "Search logs"** — GitHub มีช่องค้นหาข้อความภายใน log ทั้งหมดของ run นั้น กด Ctrl+F หรือใช้ช่องค้นหาที่ให้มาได้เลย
3. **ดาวน์โหลด log แบบเต็ม** — มีปุ่ม "Download log archive" ที่มุมขวาบนของหน้า run สำหรับดาวน์โหลด log ทั้งหมดเป็นไฟล์ .zip เก็บไว้วิเคราะห์ภายหลัง หรือแนบส่งให้ทีมช่วยดู
4. **สังเกตเวลาที่ใช้ในแต่ละ step** — ถ้า step ไหนใช้เวลานานผิดปกติ อาจเป็นสัญญาณของปัญหา performance ที่ควรปรับปรุง (เช่น ไม่ได้ใช้ cache dependency)

### การ Re-run workflow ที่ล้มเหลว

ถ้า workflow ล้มเหลวเพราะสาเหตุชั่วคราว (เช่น network flaky, dependency server ล่มชั่วคราว) สามารถกดปุ่ม **"Re-run jobs"** ที่มุมขวาบนของหน้า run นั้นได้เลย โดยมีตัวเลือกย่อย:

- **Re-run all jobs** — รันใหม่ทุก job ตั้งแต่ต้น
- **Re-run failed jobs** — รันใหม่เฉพาะ job ที่ล้มเหลว (ประหยัดเวลาและโควตานาที)

### การยกเลิก workflow ที่กำลังรันอยู่

ถ้าต้องการหยุด workflow ที่กำลังรันอยู่กลางคัน (เช่น push ผิด branch โดยไม่ตั้งใจ) สามารถกดปุ่ม **"Cancel workflow"** ที่หน้า run นั้นได้ทันที ซึ่งจะช่วยประหยัดโควตานาทีที่ไม่จำเป็น

### แจ้งเตือนอัตโนมัติเมื่อ workflow ล้มเหลว

โดย default GitHub จะส่งอีเมลแจ้งเตือนไปยังผู้ push (หรือเจ้าของ PR) โดยอัตโนมัติเมื่อ workflow ที่เกี่ยวข้องกับ commit ของตัวเองล้มเหลว สามารถปรับตั้งค่าการแจ้งเตือนได้ที่ **Settings → Notifications** ในบัญชี GitHub ส่วนตัว ซึ่งจะช่วยให้รู้ทันทีว่ามีอะไรพังโดยไม่ต้องเข้าไปเช็คหน้า Actions ด้วยตัวเองตลอดเวลา

---

## Step 669: Workflow status badge ติดใน README.md

หนึ่งในฟีเจอร์ที่ทำให้ทีมและผู้มาเยี่ยมชม repository เห็นสถานะของโปรเจกต์ได้ทันทีโดยไม่ต้องคลิกเข้าไปดูแท็บ Actions คือ **Workflow Status Badge**

ถ้าใครจำ **Part 19** ได้ (ซึ่งเราเคยสอนเรื่องการติด badge ใน README.md เช่น badge แสดง license, จำนวนดาว, เวอร์ชัน) badge ของ GitHub Actions ก็ทำงานในหลักการเดียวกัน เพียงแต่ครั้งนี้มันแสดงสถานะการรันของ workflow ล่าสุดแบบ real-time

### หน้าตาของ Badge

Badge จะแสดงเป็นรูปภาพเล็ก ๆ พร้อมข้อความ เช่น:

```
CI: passing    (พื้นหลังสีเขียว)
CI: failing    (พื้นหลังสีแดง)
```

### วิธีหา URL ของ Badge

วิธีที่ง่ายและถูกต้องที่สุดคือให้ GitHub สร้าง URL ให้อัตโนมัติ:

1. เข้าไปที่แท็บ **Actions** ของ repository
2. คลิกเลือก workflow ที่ต้องการ (จากแถบซ้าย)
3. คลิกปุ่ม **"..."** (สามจุด) มุมขวาบน แล้วเลือก **"Create status badge"**
4. GitHub จะแสดง Markdown code ให้คัดลอกไปวางใน README.md ได้ทันที

### รูปแบบ URL ของ Badge

```
https://github.com/<owner>/<repo>/actions/workflows/<workflow-file-name>.yml/badge.svg
```

ตัวอย่างจริง สมมติ repository ชื่อ `phutjirakul/my-awesome-project` และไฟล์ workflow ชื่อ `ci.yml`:

```
https://github.com/phutjirakul/my-awesome-project/actions/workflows/ci.yml/badge.svg
```

### วิธีติดใน README.md

```markdown
# My Awesome Project

[![CI](https://github.com/phutjirakul/my-awesome-project/actions/workflows/ci.yml/badge.svg)](https://github.com/phutjirakul/my-awesome-project/actions/workflows/ci.yml)

โปรเจกต์ตัวอย่างสำหรับฝึกฝน GitHub Actions
```

โครงสร้าง Markdown ของ badge แบบนี้:

```markdown
[![ข้อความ alt](URL-ของรูปภาพ badge)](URL-ที่จะพาไปเมื่อคลิก)
```

- ส่วนแรก `[![...]...]` คือรูปภาพ badge ที่ดึงมาจาก URL ของ GitHub
- ส่วนหลัง `(...)` คือลิงก์ที่จะพาไปเมื่อคลิกที่ badge — ปกตินิยมให้ลิงก์ไปที่หน้า Actions ของ workflow นั้นโดยตรง เพื่อให้คนที่คลิก badge เห็นรายละเอียดการรันล่าสุดทันที

### การระบุ branch เฉพาะใน Badge

ถ้ามีหลาย branch (เช่น `main` และ `develop`) และต้องการแสดง badge ของ branch ใดโดยเฉพาะ สามารถเพิ่ม query parameter `?branch=` ต่อท้าย URL ได้:

```markdown
[![CI](https://github.com/phutjirakul/my-awesome-project/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/phutjirakul/my-awesome-project/actions/workflows/ci.yml)
```

### ตัวอย่างการติดหลาย Badge พร้อมกันใน README

```markdown
# My Awesome Project

[![CI](https://github.com/phutjirakul/my-awesome-project/actions/workflows/ci.yml/badge.svg)](https://github.com/phutjirakul/my-awesome-project/actions/workflows/ci.yml)
[![Deploy](https://github.com/phutjirakul/my-awesome-project/actions/workflows/deploy.yml/badge.svg)](https://github.com/phutjirakul/my-awesome-project/actions/workflows/deploy.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

โปรเจกต์ตัวอย่างสำหรับฝึกฝน GitHub Actions ครอบคลุมทั้ง CI และ Deploy อัตโนมัติ
```

รูปแบบนี้ทำให้ใครก็ตามที่เข้ามาดู repository ครั้งแรก เห็นทันทีว่า:
- CI ผ่านหรือไม่ (บอกถึงคุณภาพโค้ดปัจจุบัน)
- Deploy ล่าสุดสำเร็จหรือไม่
- License ของโปรเจกต์คืออะไร

### ทำไม Badge ถึงสำคัญในเชิงมืออาชีพ

1. **สร้างความน่าเชื่อถือทันทีที่เห็น** — โดยเฉพาะสำหรับ open source project บาง badge สีเขียว "passing" ทำให้คนอยากใช้งาน/ร่วมพัฒนาโปรเจกต์มากขึ้น
2. **เตือนทีมให้เห็นปัญหาแบบทันที** — ถ้า badge เปลี่ยนเป็นสีแดงบน README ของ branch หลัก ทุกคนที่เข้ามาดูหน้าโปรเจกต์จะรู้ทันทีว่ามีบางอย่างพัง โดยไม่ต้องรอให้ใครแจ้ง
3. **เป็นมาตรฐานที่คาดหวังในวงการ** — โปรเจกต์ open source คุณภาพสูงเกือบทั้งหมดมี badge แสดงสถานะ CI ติดอยู่บน README เสมอ

---

## Step 670: แบบฝึกหัด — เขียน workflow แรกที่ checkout code แล้วรัน test ง่ายๆ

ถึงเวลาลงมือทำจริงแล้ว! แบบฝึกหัดนี้จะรวบรวมทุกสิ่งที่เรียนมาใน Part นี้ (`on:`, `jobs:`, `steps:`, `uses:`, `run:`, `runs-on:`) เข้าด้วยกันเป็น workflow ที่ใช้งานได้จริง

### เป้าหมายของแบบฝึกหัด

สร้าง workflow ที่:
1. รันทุกครั้งที่มีการ push โค้ดเข้า repository (trigger `push`)
2. Checkout โค้ดของโปรเจกต์เข้ามาใน runner
3. รัน script ทดสอบง่าย ๆ ที่เราจะเขียนขึ้นเอง
4. แสดงผลว่าผ่านหรือไม่ผ่านบนแท็บ Actions

### ขั้นตอนที่ 1: เตรียมโครงสร้างโปรเจกต์

สร้างโฟลเดอร์ฝึกฝนใหม่ (ตามธรรมเนียมที่วางไว้ตั้งแต่ Part 01):

```bash
mkdir ~/git-course/part-67-github-actions
cd ~/git-course/part-67-github-actions
git init
```

สร้างไฟล์ script ง่าย ๆ ชื่อ `test.sh` ที่จะทำหน้าที่เป็น "test suite" จำลอง:

```bash
mkdir tests
```

สร้างไฟล์ `tests/test.sh` ด้วยเนื้อหาต่อไปนี้:

```bash
#!/bin/bash
set -e

echo "=== เริ่มรัน test suite ==="

echo "Test 1: ตรวจสอบว่าไฟล์ app.py มีอยู่จริง"
if [ -f "app.py" ]; then
  echo "  ผ่าน: พบไฟล์ app.py"
else
  echo "  ล้มเหลว: ไม่พบไฟล์ app.py"
  exit 1
fi

echo "Test 2: ตรวจสอบว่าฟังก์ชัน add() คำนวณถูกต้อง"
RESULT=$(python3 -c "
import sys
sys.path.insert(0, '.')
from app import add
print(add(2, 3))
")

if [ "$RESULT" == "5" ]; then
  echo "  ผ่าน: add(2, 3) = 5 ถูกต้อง"
else
  echo "  ล้มเหลว: add(2, 3) ควรได้ 5 แต่ได้ $RESULT"
  exit 1
fi

echo "=== test suite ทั้งหมดผ่านสำเร็จ ==="
```

ให้สิทธิ์ execute กับไฟล์นี้:

```bash
chmod +x tests/test.sh
```

สร้างไฟล์ `app.py` ที่ไฟล์ test จะเข้าไปทดสอบ:

```python
def add(a, b):
    """คืนค่าผลรวมของสองจำนวน"""
    return a + b


def subtract(a, b):
    """คืนค่าผลต่างของสองจำนวน"""
    return a - b


if __name__ == "__main__":
    print("2 + 3 =", add(2, 3))
    print("5 - 2 =", subtract(5, 2))
```

### ขั้นตอนที่ 2: สร้างโฟลเดอร์ workflow

```bash
mkdir -p .github/workflows
```

### ขั้นตอนที่ 3: เขียนไฟล์ workflow แรก

สร้างไฟล์ `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches:
      - main

jobs:
  test:
    name: Run Test Suite
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Show Python version
        run: python3 --version

      - name: Run test script
        run: |
          chmod +x tests/test.sh
          ./tests/test.sh
```

มาไล่ดูทีละส่วนว่าไฟล์นี้ทำอะไรบ้าง:

| ส่วน | อธิบาย |
|---|---|
| `name: CI` | ชื่อ workflow ที่จะแสดงบนแท็บ Actions |
| `on: push: branches: [main]` | รันเฉพาะเมื่อ push เข้า branch `main` เท่านั้น |
| `jobs: test:` | มี 1 job ชื่อ `test` |
| `runs-on: ubuntu-latest` | รันบน Linux runner ล่าสุด |
| Step "Checkout code" | ดึงโค้ดของ repository เข้ามาใน runner (ขาดไม่ได้) |
| Step "Setup Python" | ติดตั้ง Python 3.12 บน runner |
| Step "Show Python version" | ตรวจสอบว่า Python ติดตั้งสำเร็จ (เพื่อ debug ได้ง่ายถ้ามีปัญหา) |
| Step "Run test script" | ให้สิทธิ์ execute แล้วรัน `tests/test.sh` ที่เราเขียนไว้ |

### ขั้นตอนที่ 4: Commit และ Push ขึ้น GitHub

สมมติว่าคุณสร้าง repository บน GitHub ไว้แล้วชื่อ `github-actions-practice` (ถ้ายังไม่มีให้กลับไปทบทวนวิธีสร้าง repository จาก Part ก่อน ๆ ในเฟส 3):

```bash
git add .
git commit -m "เพิ่ม workflow แรก: CI ที่ checkout และรัน test อัตโนมัติ"
git branch -M main
git remote add origin https://github.com/<your-username>/github-actions-practice.git
git push -u origin main
```

### ขั้นตอนที่ 5: ตรวจสอบผลลัพธ์บน GitHub

1. เข้าไปที่ repository บนเว็บ GitHub
2. คลิกแท็บ **Actions**
3. จะเห็น workflow run ชื่อ "CI" ที่เพิ่งถูก trigger จากการ push ล่าสุด
4. คลิกเข้าไปดู เห็น job ชื่อ "Run Test Suite" กำลังรัน (หรือรันเสร็จแล้ว)
5. คลิกเข้าไปดู log แต่ละ step ตรวจสอบว่าทุก step ขึ้นเครื่องหมาย ✅

ถ้าทุกอย่างถูกต้อง คุณควรเห็นข้อความในหน้า log ของ step สุดท้ายประมาณนี้:

```
=== เริ่มรัน test suite ===
Test 1: ตรวจสอบว่าไฟล์ app.py มีอยู่จริง
  ผ่าน: พบไฟล์ app.py
Test 2: ตรวจสอบว่าฟังก์ชัน add() คำนวณถูกต้อง
  ผ่าน: add(2, 3) = 5 ถูกต้อง
=== test suite ทั้งหมดผ่านสำเร็จ ===
```

### ขั้นตอนที่ 6 (ส่วนขยาย): ทดลองทำให้ test ล้มเหลวโดยตั้งใจ

เพื่อให้เข้าใจการอ่าน log ตอน workflow ล้มเหลว (ตามที่เรียนใน Step 668) ลองแก้ไฟล์ `app.py` ให้ฟังก์ชัน `add()` คำนวณผิดโดยตั้งใจ:

```python
def add(a, b):
    return a + b + 1  # แก้ให้ผิดโดยตั้งใจ เพื่อทดสอบ CI
```

Commit และ push อีกครั้ง:

```bash
git add app.py
git commit -m "ทดลอง: ทำให้ add() คำนวณผิดเพื่อทดสอบว่า CI จับได้"
git push
```

กลับไปดูแท็บ Actions อีกครั้ง คราวนี้ควรเห็น workflow run ใหม่ขึ้นเครื่องหมาย ❌ พร้อม log แสดงข้อความ:

```
Test 2: ตรวจสอบว่าฟังก์ชัน add() คำนวณถูกต้อง
  ล้มเหลว: add(2, 3) ควรได้ 5 แต่ได้ 6
```

นี่คือหัวใจของ CI — **โค้ดที่มีบั๊กจะถูกตรวจจับโดยอัตโนมัติทันทีที่ push ไม่ต้องรอให้ใครมาเจอบั๊กเองในภายหลัง** อย่าลืมแก้ `app.py` กลับให้ถูกต้อง แล้ว commit push อีกครั้งเพื่อให้ badge กลับมาเป็นสีเขียว

### ขั้นตอนที่ 7 (ส่วนขยาย): ติด Badge เข้า README.md

สร้างไฟล์ `README.md` (หรือแก้ไฟล์เดิมถ้ามีอยู่แล้ว):

```markdown
# GitHub Actions Practice

[![CI](https://github.com/<your-username>/github-actions-practice/actions/workflows/ci.yml/badge.svg)](https://github.com/<your-username>/github-actions-practice/actions/workflows/ci.yml)

โปรเจกต์ฝึกฝน GitHub Actions จาก Part 67 ของหลักสูตร Git & GitHub ภาษาไทย
```

Commit และ push:

```bash
git add README.md
git commit -m "ติด CI status badge บน README"
git push
```

เข้าไปดูหน้า repository อีกครั้ง ควรเห็น badge สีเขียว "CI: passing" ปรากฏอยู่ด้านบนของหน้า README แล้ว

### เกณฑ์ตรวจสอบว่าทำแบบฝึกหัดสำเร็จ

- [ ] มีไฟล์ `.github/workflows/ci.yml` ที่ syntax ถูกต้อง (YAML ไม่ error)
- [ ] Workflow ถูก trigger อัตโนมัติทุกครั้งที่ push เข้า `main`
- [ ] Workflow ใช้ `actions/checkout@v4` ในการดึงโค้ดเข้ามา
- [ ] Workflow รัน test script และแสดงผลผ่าน/ไม่ผ่านได้ถูกต้อง
- [ ] เคยเห็นทั้งสถานะ ✅ (สำเร็จ) และ ❌ (ล้มเหลว) อย่างน้อยคนละครั้ง
- [ ] ติด status badge บน README.md สำเร็จ และ badge แสดงสถานะถูกต้องตรงกับผลการรันล่าสุด

หากทำครบทุกข้อนี้ แปลว่าคุณเข้าใจแก่นของ GitHub Actions แล้วอย่างแท้จริง และพร้อมสำหรับเนื้อหาที่ซับซ้อนขึ้นใน Part ถัดไป

---

## สรุป Part 67

ใน Part นี้เราได้เรียนรู้ว่า:

1. **GitHub Actions** คือระบบ CI/CD ที่ฝังอยู่ในตัว GitHub เปิดตัวปี 2019 (GA) และกลายเป็นมาตรฐานของวงการอย่างรวดเร็วเพราะไม่ต้องพึ่งพา service ภายนอกและมี Marketplace ขนาดใหญ่
2. ไฟล์ workflow ต้องอยู่ที่ **`.github/workflows/*.yml`** ที่ root ของ repository เท่านั้น เขียนด้วย YAML ที่ต้องระวังเรื่อง indentation อย่างมาก
3. **`on:`** กำหนดว่า workflow จะรันเมื่อไหร่ — trigger ที่ใช้บ่อยที่สุดคือ `push`, `pull_request`, `schedule` (cron) และ `workflow_dispatch` (รันเอง)
4. **`jobs:`** และ **`steps:`** คือโครงสร้างหลักของ workflow — job รันขนานกัน (เว้นแต่ตั้ง `needs:`) ส่วน step ในแต่ละ job รันเรียงลำดับเสมอ
5. **`uses:`** เรียกใช้ Action สำเร็จรูปจาก Marketplace โดย `actions/checkout@v4` คือ Action ที่ต้องมีเกือบทุก workflow เพราะ runner เริ่มต้นแบบว่างเปล่าเสมอ
6. **`run:`** ใช้รันคำสั่ง shell ตรง ๆ ภายใน step เหมาะกับคำสั่งเฉพาะของโปรเจกต์ที่ไม่มี Action สำเร็จรูป
7. **`runs-on:`** เลือก runner ที่จะใช้ — `ubuntu-latest` เป็นตัวเลือกที่ประหยัดโควตาที่สุดและใช้บ่อยที่สุด ส่วน `windows-latest` และ `macos-latest` มีอัตราหักโควตาแพงกว่า
8. แท็บ **Actions** บน GitHub UI คือที่ที่ใช้ดูสถานะ ดู log แบบละเอียด, re-run และ cancel workflow ได้
9. **Status badge** ที่สร้างจากหน้า Actions สามารถติดบน README.md เพื่อให้ทุกคนเห็นสถานะโปรเจกต์ได้ทันทีโดยไม่ต้องคลิกเข้าไปดู เชื่อมโยงกับสิ่งที่เคยเรียนเรื่อง badge ใน Part 19
10. ลงมือเขียน workflow แรกที่ครบวงจร: checkout code → setup environment → รัน test script → เห็นผลลัพธ์ทั้งกรณีสำเร็จและล้มเหลวจริงบน GitHub

### Checklist ก่อนไป Part 68

ก่อนไปต่อ Part 68 ให้ตรวจสอบว่าคุณ:

- [ ] อธิบายได้ว่า GitHub Actions คืออะไรและแก้ปัญหาอะไรให้กับทีมพัฒนาซอฟต์แวร์
- [ ] วางไฟล์ workflow ในตำแหน่งที่ถูกต้อง (`.github/workflows/*.yml`) ได้โดยไม่ผิดพลาด
- [ ] เขียน `on:` เพื่อกำหนด trigger แบบ `push`, `pull_request`, `schedule`, `workflow_dispatch` ได้
- [ ] เข้าใจความสัมพันธ์ระหว่าง workflow → job → step และรู้ว่า job รันขนาน ส่วน step รันเรียงลำดับ
- [ ] ใช้ `uses: actions/checkout@v4` ได้อย่างถูกต้อง และเข้าใจว่าทำไมต้องมี step นี้เกือบทุก workflow
- [ ] แยกความแตกต่างระหว่าง `uses:` กับ `run:` ได้ชัดเจน
- [ ] เลือก `runs-on:` ที่เหมาะสมกับงาน และเข้าใจเรื่องอัตราหักโควตาของแต่ละ OS
- [ ] เข้าไปดู log บนแท็บ Actions ได้ และรู้วิธี re-run/cancel workflow
- [ ] ติด status badge บน README.md ของโปรเจกต์ตัวเองสำเร็จ
- [ ] เขียนและรัน workflow แรกสำเร็จจนเห็นผลลัพธ์ ✅ บน GitHub จริง

**ต่อไป:** [Part 68: GitHub Actions: Jobs, Steps, Matrix Build](./part-068-github-actions-jobs-matrix.md)
