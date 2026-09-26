# Part 69: GitHub Actions: Secrets, Environments, Artifacts

> **Step ในหลักสูตรนี้:** Step 681–690
> **เฟส:** 7 — CI/CD เต็มรูปแบบด้วย GitHub Actions และ GitLab CI/CD
> **เป้าหมายของ Part นี้:** เข้าใจวิธีจัดการข้อมูลลับ (Secrets) อย่างปลอดภัยใน GitHub Actions, เข้าใจแนวคิดของ Environment พร้อม protection rules สำหรับควบคุมการ deploy, และเข้าใจการใช้ Artifacts เพื่อเก็บและส่งต่อผลลัพธ์ระหว่าง job รวมถึงการทำ Caching เพื่อเพิ่มความเร็วของ workflow — ทั้งหมดนี้คือรากฐานสำคัญที่ทุกทีมต้องใช้ก่อนจะทำ CI/CD ระดับ production จริง

---

## สารบัญของ Part นี้

- Step 681: GitHub Secrets คืออะไร — เก็บข้อมูลลับอย่างปลอดภัย
- Step 682: การสร้างและใช้ secrets ใน workflow (`${{ secrets.NAME }}`)
- Step 683: Environment คืออะไร (production, staging) พร้อม protection rules
- Step 684: Environment Secrets vs Repository Secrets vs Organization Secrets
- Step 685: Environment Approval — ต้องมีคนอนุมัติก่อน job จะรันต่อ
- Step 686: Artifacts คืออะไร — `actions/upload-artifact` และ `actions/download-artifact`
- Step 687: การส่งต่อ Artifact ระหว่าง Job (build → deploy)
- Step 688: Caching Dependencies ด้วย `actions/cache`
- Step 689: ความปลอดภัยของ Secrets — Masking และข้อจำกัดกับ Fork
- Step 690: แบบฝึกหัด — ตั้งค่า Secret, Environment และ Artifact ครบวงจร

---

## Step 681: GitHub Secrets คืออะไร

เมื่อ workflow ของเราเริ่มทำงานที่ซับซ้อนขึ้น เช่น deploy ไปยัง server จริง, เรียก API ภายนอก, หรือ push image ขึ้น registry สิ่งที่หลีกเลี่ยงไม่ได้คือ **workflow จะต้องใช้ข้อมูลที่เป็นความลับ** เช่น:

- API key ของบริการภายนอก (เช่น บริการ deploy, บริการ monitoring)
- Password หรือ token สำหรับเข้าถึง database
- SSH private key สำหรับเชื่อมต่อ server
- Token สำหรับ authenticate กับ container registry
- Webhook URL ของระบบแจ้งเตือนภายในทีม

คำถามสำคัญคือ **เราจะเก็บข้อมูลเหล่านี้ไว้ที่ไหน โดยที่มันจะไม่ปรากฏในซอร์สโค้ดที่ทุกคนมองเห็นได้**

### ปัญหาถ้าเขียน secret ลงในไฟล์ workflow ตรง ๆ

ลองจินตนาการว่ามีคนเขียน workflow แบบนี้ (ตัวอย่างนี้แสดงให้เห็น**ปัญหา** ไม่ใช่วิธีที่ถูกต้อง):

```yaml
# ตัวอย่างที่ผิด — ห้ามทำแบบนี้เด็ดขาด
steps:
  - name: Deploy
    run: |
      curl -X POST https://api.example.com/deploy \
        -H "Authorization: Bearer <ค่าจริงเขียนตรงนี้>"
```

ปัญหาของโค้ดแบบนี้มีหลายชั้น:

1. **ทุกคนที่เข้าถึง repository ได้ จะเห็นค่าลับนี้ทันที** — ไม่ว่าจะเป็น public repo หรือ private repo ที่มีคนในทีมหลายคน
2. **ค่าลับจะถูกบันทึกอยู่ใน Git history ตลอดไป** — ต่อให้ลบทิ้งในภายหลัง มันยังคงอยู่ใน commit เก่า สามารถขุดกลับมาดูได้เสมอ
3. **ถ้า repo เป็น public หรือถูกทำเป็น public ในภายหลัง** ค่าลับนี้จะรั่วไหลออกสู่สาธารณะทันที และมักถูกบอทสแกนหาเพื่อนำไปใช้ในทางที่ผิดภายในไม่กี่นาที
4. **การหมุนเวียน (rotate) ค่าลับทำได้ยาก** — ถ้าต้องเปลี่ยนค่า ต้องไปแก้ไฟล์ workflow และสร้าง commit ใหม่ทุกครั้ง

นี่คือเหตุผลที่ GitHub สร้างฟีเจอร์ **Secrets** ขึ้นมาโดยเฉพาะ

### GitHub Secrets คืออะไร

> **GitHub Secrets คือพื้นที่เก็บข้อมูลลับที่ถูกเข้ารหัส (encrypted) แยกออกจากซอร์สโค้ดโดยสิ้นเชิง โดย workflow สามารถเรียกใช้ค่าเหล่านี้ได้ผ่าน context `secrets` โดยที่ค่าจริงจะไม่ปรากฏในไฟล์ YAML หรือใน log เลย**

คุณสมบัติสำคัญของ Secrets:

| คุณสมบัติ | รายละเอียด |
|---|---|
| **การเข้ารหัส** | ค่าที่บันทึกจะถูกเข้ารหัสด้วย libsodium sealed box ก่อนถูกเก็บไว้ในฐานข้อมูลของ GitHub |
| **มองไม่เห็นค่าจริงอีกเลย** | หลังจากบันทึก secret แล้ว แม้แต่เจ้าของ repository ก็ **ไม่สามารถดูค่าเดิมได้อีก** ทำได้แค่ "อัปเดตค่าใหม่ทับ" เท่านั้น |
| **Masking อัตโนมัติ** | ถ้าค่า secret หลุดไปปรากฏใน log ของ job ระบบจะแทนที่ด้วย `***` โดยอัตโนมัติ (รายละเอียดเต็มใน Step 689) |
| **เข้าถึงได้เฉพาะตอน workflow รัน** | secret จะถูกส่งเข้าไปใน environment ของ job เฉพาะตอนที่ workflow ทำงานเท่านั้น ไม่สามารถดึงออกมาดูตรง ๆ ผ่าน UI ได้ |
| **จำกัดที่มา** | Pull Request จาก fork ภายนอก (external fork) จะ**ไม่ได้รับ** secrets ตาม default เพื่อป้องกันการขโมยค่าลับผ่าน PR ที่เป็นอันตราย |

### ตำแหน่งที่ตั้งค่า Secrets ใน GitHub

Secrets ตั้งค่าได้ผ่านหน้าเว็บของ repository ที่:

```
Settings → Secrets and variables → Actions → New repository secret
```

หน้าจอจะให้กรอก 2 ช่องเท่านั้น:

- **Name** — ชื่อของ secret (ตัวพิมพ์ใหญ่ ขีดล่างคั่นคำ เช่น `DEPLOY_API_KEY`)
- **Secret** — ค่าจริงที่จะถูกเข้ารหัสทันทีหลังกดบันทึก

> **ข้อควรระวังสำคัญ:** ชื่อของ secret **ห้ามขึ้นต้นด้วย `GITHUB_`** เพราะ prefix นี้ถูกสงวนไว้สำหรับตัวแปรระบบของ GitHub เอง (เช่น `GITHUB_TOKEN` ที่ระบบสร้างให้อัตโนมัติ) การตั้งชื่อ secret ที่ขึ้นต้นด้วย `GITHUB_` จะถูกปฏิเสธทันที

### ประเภทของ Secrets ตามระดับ (ภาพรวมก่อนเจาะลึกใน Step 684)

GitHub มี secrets อยู่ 3 ระดับตามขอบเขต (scope):

1. **Repository secret** — ใช้ได้เฉพาะใน repository นั้น
2. **Environment secret** — ใช้ได้เฉพาะเมื่อ job ระบุ environment ที่ตรงกัน
3. **Organization secret** — ใช้ร่วมกันได้หลาย repository ภายในองค์กรเดียวกัน

เราจะเจาะลึกความแตกต่างและกรณีการใช้งานของแต่ละระดับใน Step 684 แต่ในตอนนี้ให้เข้าใจภาพรวมไว้ก่อนว่า **secrets ไม่ได้มีแค่ระดับเดียว มันออกแบบมาให้ยืดหยุ่นตามขนาดขององค์กร**

### เมื่อไหร่ที่ควรใช้ Secrets

ใช้ Secrets ทุกครั้งที่ข้อมูลนั้นเข้าข่ายต่อไปนี้:

- ถ้าข้อมูลนี้หลุดออกไป จะทำให้เกิดความเสียหาย (เข้าถึงระบบโดยไม่ได้รับอนุญาต, เสียค่าใช้จ่าย, ข้อมูลผู้ใช้รั่วไหล)
- ข้อมูลนี้ไม่ควรถูกมองเห็นโดยคนที่ไม่มีสิทธิ์ แม้จะเป็นคนในทีมเดียวกันก็ตาม (เช่น นักพัฒนาฝึกงานอาจไม่ควรเห็น production database password)
- ข้อมูลนี้เปลี่ยนแปลงไปตาม environment (dev, staging, production ใช้ค่าคนละชุด)

ในทางกลับกัน ค่าที่**ไม่ใช่ความลับ** เช่น ชื่อ environment, URL ของ API ที่เป็น public, เวอร์ชันของ tool ที่ใช้ ควรใช้ **Variables** (`vars` context) แทน ไม่ใช่ secrets — เพราะ variables สามารถดูค่าย้อนหลังได้และไม่ถูก mask ใน log ทำให้ debug ง่ายกว่า

---

## Step 682: การสร้างและใช้ Secrets ใน Workflow

หลังจากเข้าใจแนวคิดแล้ว มาดูขั้นตอนจริงในการสร้างและเรียกใช้ secret กัน

### 682.1 ขั้นตอนสร้าง Repository Secret ผ่านหน้าเว็บ

1. เข้าไปที่หน้า repository บน GitHub
2. คลิก **Settings** (แท็บบนสุดของ repo)
3. เมนูซ้ายมือ เลือก **Secrets and variables → Actions**
4. แท็บ **Secrets** จะถูกเลือกไว้เป็นค่าเริ่มต้น กดปุ่ม **New repository secret**
5. กรอก **Name** เช่น `DEPLOY_API_KEY`
6. กรอก **Secret** ด้วยค่าจริง (ในตัวอย่างของหลักสูตรนี้เราจะใช้ค่าปลอมเสมอ เช่น `<YOUR_API_KEY>`)
7. กดปุ่ม **Add secret**

หลังบันทึกแล้ว หน้าจอจะแสดงแค่ชื่อ secret และวันที่อัปเดตล่าสุดเท่านั้น **ค่าจริงจะไม่ถูกแสดงอีกเลย**

### 682.2 การสร้างผ่าน GitHub CLI (`gh`)

สำหรับทีมที่ต้องการ automate การตั้งค่า secret (เช่น ตอน bootstrap โปรเจกต์ใหม่) สามารถใช้คำสั่ง:

```bash
# ตั้งค่า repository secret ผ่าน CLI
gh secret set DEPLOY_API_KEY --body "<YOUR_API_KEY>"

# หรือดึงค่าจากไฟล์ (ไฟล์นี้ต้องไม่ถูก commit เข้า git เด็ดขาด)
gh secret set DEPLOY_API_KEY < ./secret-value.txt

# ดูรายชื่อ secrets ที่มีอยู่ (เห็นแค่ชื่อ ไม่เห็นค่า)
gh secret list
```

> **ข้อควรระวัง:** ถ้าใช้วิธีดึงค่าจากไฟล์ ต้องแน่ใจว่าไฟล์นั้นถูกใส่ไว้ใน `.gitignore` แล้ว และลบทิ้งทันทีหลังใช้งานเสร็จ เพื่อไม่ให้ค่าลับหลงเหลืออยู่บนเครื่อง

### 682.3 การเรียกใช้ Secret ใน Workflow

เมื่อสร้าง secret แล้ว เราเรียกใช้งานผ่าน context พิเศษชื่อ `secrets`:

```yaml
name: Deploy to Server

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy via API
        env:
          API_KEY: ${{ secrets.DEPLOY_API_KEY }}
        run: |
          curl -X POST "https://deploy.example.com/api/trigger" \
            -H "Authorization: Bearer ${API_KEY}" \
            -H "Content-Type: application/json" \
            -d '{"branch": "main"}'
```

สังเกตรูปแบบสำคัญ:

1. เราไม่เขียน `${{ secrets.DEPLOY_API_KEY }}` ตรง ๆ ในคำสั่ง `run` แต่ **ส่งผ่าน environment variable ก่อน** (`env: API_KEY: ...`) แล้วค่อยอ้างถึง `${API_KEY}` ใน shell
2. เหตุผลที่ทำแบบนี้คือเพื่อป้องกัน **shell injection** — ถ้าค่า secret มีอักขระพิเศษ (เช่น backtick, `$()`, เครื่องหมายคำพูด) การแทรกมันตรง ๆ ลงในสตริงคำสั่งของ `run:` อาจทำให้ shell ตีความผิดและเปิดช่องให้ execute คำสั่งที่ไม่ได้ตั้งใจ การผ่าน env var จะปลอดภัยกว่าเสมอ

### 682.4 ตัวอย่างที่ควรหลีกเลี่ยง (Anti-pattern)

```yaml
# ไม่แนะนำ — เสี่ยงต่อ shell injection ถ้า secret มีอักขระพิเศษ
- name: Deploy (แบบไม่ปลอดภัย)
  run: curl -H "Authorization: Bearer ${{ secrets.DEPLOY_API_KEY }}" https://deploy.example.com
```

แม้ตัวอย่างนี้จะทำงานได้ในกรณีทั่วไป แต่ผู้เชี่ยวชาญด้านความปลอดภัยของ GitHub แนะนำให้หลีกเลี่ยงการใส่ `${{ ... }}` ของ context ที่ไม่น่าเชื่อถือ (รวมถึง secrets) เข้าไปในบรรทัดคำสั่งโดยตรง ให้ผ่าน `env:` เสมอเพื่อความปลอดภัยสูงสุด

### 682.5 การใช้ Secret หลายตัวพร้อมกัน

```yaml
jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Log in to Container Registry
        env:
          REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
          REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
        run: |
          echo "${REGISTRY_PASSWORD}" | docker login registry.example.com \
            -u "${REGISTRY_USERNAME}" --password-stdin

      - name: Build image
        run: docker build -t registry.example.com/myapp:${{ github.sha }} .

      - name: Push image
        run: docker push registry.example.com/myapp:${{ github.sha }}
```

สังเกตว่าคำสั่ง `docker login` ใช้ `--password-stdin` แทนการใส่ password ต่อท้ายคำสั่งตรง ๆ นี่คือ best practice ของ Docker เองที่ป้องกันไม่ให้ password ปรากฏใน process list ของเครื่อง

### 682.6 การใช้ Secret กับ `if` condition (ข้อจำกัดที่ควรรู้)

Secrets **ไม่สามารถใช้ใน `if:` condition ได้โดยตรงในบาง context** เนื่องจากเหตุผลด้านความปลอดภัย (ป้องกันการเดาค่าผ่าน timing/behavior) แต่สามารถตรวจสอบว่า secret ถูกตั้งค่าไว้หรือยัง (ไม่ใช่ค่าจริง) ได้ผ่านการส่งผ่าน `env` ก่อน:

```yaml
jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Check if webhook is configured
        env:
          HAS_WEBHOOK: ${{ secrets.NOTIFY_WEBHOOK_URL != '' }}
        run: |
          if [ "${HAS_WEBHOOK}" = "true" ]; then
            echo "Webhook configured, sending notification..."
          else
            echo "No webhook configured, skipping notification."
          fi
```

---

## Step 683: Environment คืออะไร (Production, Staging) พร้อม Protection Rules

จนถึงตอนนี้ workflow ของเรารันแบบ "ทำเลย" ทันทีที่ trigger ตรงเงื่อนไข แต่ในโลกจริง การ deploy ไปยัง **production** ควรมีการควบคุมที่รัดกุมกว่าการ deploy ไปยัง **development** หรือ **staging** มาก

นี่คือที่มาของฟีเจอร์ **Environments** ใน GitHub Actions

### Environment คืออะไร

> **Environment คือการตั้งชื่อ "สภาพแวดล้อมการ deploy" (เช่น `production`, `staging`, `development`) ให้กับ job หนึ่ง ๆ พร้อมแนบกฎการป้องกัน (protection rules) และ secrets/variables เฉพาะของ environment นั้นเข้าไปด้วย**

Environment มักถูกใช้แทนสภาพแวดล้อมจริงในวงจรการ deploy ของทีม เช่น:

```
Developer เขียนโค้ด
        │
        ▼
   ┌─────────┐
   │   dev   │  ← deploy อัตโนมัติทุกครั้งที่ push เข้า feature branch
   └────┬────┘
        │ merge เข้า main
        ▼
   ┌─────────┐
   │ staging │  ← deploy อัตโนมัติทุกครั้งที่ merge เข้า main
   └────┬────┘
        │ ทดสอบผ่านแล้ว ต้องมีคนอนุมัติ
        ▼
   ┌─────────────┐
   │ production  │  ← ต้องรอ approval จากคนที่มีสิทธิ์ก่อนถึงจะ deploy จริง
   └─────────────┘
```

### การสร้าง Environment

Environment สร้างได้ที่:

```
Settings → Environments → New environment
```

กรอกชื่อ environment เช่น `production` แล้วกด **Configure environment** เพื่อตั้งค่า protection rules ต่อไป

### การอ้างอิง Environment ใน Workflow

ใน workflow เราระบุ environment ที่ระดับ `job` ด้วยคีย์ `environment:`

```yaml
name: Deploy Pipeline

on:
  push:
    branches: [main]

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Deploy to staging
        env:
          API_KEY: ${{ secrets.STAGING_API_KEY }}
        run: echo "Deploying to staging with key length ${#API_KEY}"

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Deploy to production
        env:
          API_KEY: ${{ secrets.PRODUCTION_API_KEY }}
        run: echo "Deploying to production with key length ${#API_KEY}"
```

สังเกตว่า `secrets.STAGING_API_KEY` และ `secrets.PRODUCTION_API_KEY` เป็นชื่อ secret **คนละตัวกัน** แต่ในทางปฏิบัติ ทีมมักตั้งชื่อ secret ให้เหมือนกัน (เช่น `API_KEY`) แต่เก็บไว้ในระดับ **Environment secret** คนละ environment แทน — เดี๋ยวเราจะเห็นตัวอย่างแบบนั้นใน Step 684

### การระบุ URL ของ Environment (Environment URL)

Environment ยังรองรับการระบุ URL ปลายทาง ซึ่งจะไปปรากฏเป็นลิงก์ในหน้า deployment ของ GitHub ให้กดเข้าไปดูผลลัพธ์ได้ทันที:

```yaml
jobs:
  deploy-production:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.example.com
    steps:
      - name: Deploy
        run: echo "Deploying..."
```

หลัง job นี้รันสำเร็จ ในหน้า **Deployments** ของ repository จะมีลิงก์ `https://myapp.example.com` ให้กดตรงไปดูเว็บไซต์ที่เพิ่ง deploy เสร็จได้ทันที สะดวกมากสำหรับทีมที่ deploy บ่อย

### Protection Rules ที่ Environment รองรับ

เมื่อกด **Configure environment** จะมีตัวเลือกการป้องกันหลัก ๆ ดังนี้:

| Protection Rule | คำอธิบาย |
|---|---|
| **Required reviewers** | ต้องมีคนใน list ที่กำหนดกดอนุมัติก่อน job ถึงจะรันต่อได้ (รายละเอียดเต็มใน Step 685) |
| **Wait timer** | หน่วงเวลาก่อนให้ job เริ่มรัน (เช่น รอ 15 นาทีก่อน deploy จริง เผื่อมีคนอยากยกเลิก) |
| **Deployment branches and tags** | จำกัดว่า environment นี้จะถูกใช้ได้จาก branch หรือ tag ไหนบ้างเท่านั้น (เช่น `production` deploy ได้จาก `main` เท่านั้น) |

ตัวอย่างการตั้งค่า **Deployment branches** ที่พบบ่อยที่สุด:

- Environment `production` → อนุญาตเฉพาะ branch `main`
- Environment `staging` → อนุญาตเฉพาะ branch `main` และ `release/*`
- Environment `development` → อนุญาตทุก branch

การตั้งค่านี้ช่วยป้องกันไม่ให้มีใคร deploy โค้ดจาก feature branch ที่ยังไม่ผ่านการรีวิวขึ้น production โดยไม่ได้ตั้งใจ แม้ว่าคนนั้นจะมีสิทธิ์เข้าถึง secret ของ production ก็ตาม เพราะ workflow จะหยุดทำงานทันทีถ้า branch ไม่ตรงเงื่อนไข

### ทำไม Environment ถึงสำคัญมากในระดับองค์กร

1. **แยกขอบเขตความรับผิดชอบ** — ทีม DevOps ดูแล production environment ในขณะที่นักพัฒนาทั่วไปเข้าถึงได้แค่ dev/staging
2. **ลดความเสี่ยงจากความผิดพลาดของมนุษย์** — การ deploy production ต้องผ่านขั้นตอนตรวจสอบเพิ่มเติมเสมอ
3. **มี audit trail ที่ชัดเจน** — หน้า Deployments ของ GitHub จะบันทึกว่าใคร deploy อะไร ไปที่ environment ไหน เมื่อไหร่ และผลลัพธ์เป็นอย่างไร
4. **รองรับ compliance ขององค์กรขนาดใหญ่** — หลายองค์กร (โดยเฉพาะธนาคาร, หน่วยงานรัฐ) กำหนดให้การ deploy production ต้องมีคนอนุมัติอย่างน้อย 1-2 คนเสมอตามนโยบายความปลอดภัย

---

## Step 684: Environment Secrets vs Repository Secrets vs Organization Secrets

ตอนนี้เราเข้าใจแล้วว่า secrets มี 3 ระดับ มาดูรายละเอียดความแตกต่างและ scope ของแต่ละแบบให้ชัดเจน

### 684.1 Repository Secret

- **ขอบเขต:** ใช้ได้เฉพาะภายใน repository เดียวที่สร้างมันขึ้นมา
- **ที่ตั้งค่า:** `Settings → Secrets and variables → Actions → Secrets` (ระดับ repo)
- **ใครเข้าถึงได้:** ทุก workflow ใน repository นั้น ไม่ว่าจะระบุ environment หรือไม่ก็ตาม
- **เหมาะกับ:** โปรเจกต์เดี่ยว ๆ ที่ไม่ได้แยก environment หลายระดับ หรือค่าที่ใช้ร่วมกันทุก environment (เช่น token สำหรับ comment บน PR)

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    # ไม่ระบุ environment — เข้าถึง repository secret ได้ตามปกติ
    steps:
      - name: Use repo secret
        env:
          TOKEN: ${{ secrets.CODECOV_TOKEN }}
        run: echo "Token length: ${#TOKEN}"
```

### 684.2 Environment Secret

- **ขอบเขต:** ใช้ได้เฉพาะเมื่อ job ระบุ `environment:` ที่ตรงกับชื่อ environment ที่สร้าง secret ไว้เท่านั้น
- **ที่ตั้งค่า:** `Settings → Environments → (เลือก environment) → Environment secrets`
- **ใครเข้าถึงได้:** เฉพาะ job ที่ประกาศ `environment: <ชื่อ>` ตรงกัน และต้องผ่าน protection rules ของ environment นั้นก่อน (ถ้ามี)
- **เหมาะกับ:** ค่าที่ต้องแตกต่างกันไปตาม environment เช่น database connection string ของ staging กับ production ที่ชี้ไปคนละฐานข้อมูล

**จุดสำคัญที่สุดของ Environment secret:** สามารถตั้งชื่อ secret **เหมือนกันทุกตัวอักษร** ในแต่ละ environment ได้ (เช่นทั้ง staging และ production ต่างก็มี secret ชื่อ `DATABASE_URL`) แต่**ค่าจริงข้างในต่างกัน** — ทำให้ workflow เขียนโค้ดชุดเดียว ใช้ได้กับทุก environment โดยไม่ต้องเขียน `if/else` แยกค่า:

```yaml
name: Multi-Environment Deploy

on:
  workflow_dispatch:
    inputs:
      target_env:
        description: "Environment ที่จะ deploy"
        required: true
        type: choice
        options:
          - staging
          - production

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.target_env }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run migration and deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          echo "Deploying to ${{ inputs.target_env }}"
          echo "Using database at (masked): ${DATABASE_URL:0:0}***"
```

ในตัวอย่างนี้ `environment: ${{ inputs.target_env }}` เป็นค่า dynamic ที่มาจาก input ตอนกด **Run workflow** ทำให้ workflow เดียวสามารถเลือก deploy ไปยัง staging หรือ production ก็ได้ และดึง secret `DATABASE_URL` ที่ **ค่าถูกต้องของ environment นั้น** มาใช้โดยอัตโนมัติ

### 684.3 Organization Secret

- **ขอบเขต:** ใช้ร่วมกันได้ในหลาย repository ภายใน organization เดียวกัน
- **ที่ตั้งค่า:** ต้องมีสิทธิ์ระดับ organization owner หรือ admin ตั้งค่าที่ `Organization Settings → Secrets and variables → Actions`
- **การควบคุมการเข้าถึง:** ตอนสร้าง organization secret จะมีตัวเลือกว่าจะให้ repository ไหนบ้างเข้าถึงได้:
  - **All repositories** — ทุก repo ในองค์กรเข้าถึงได้
  - **Private repositories** — เฉพาะ private repo เท่านั้น
  - **Selected repositories** — เลือก repo เจาะจงเท่านั้น
- **เหมาะกับ:** ค่าที่ใช้ร่วมกันในหลายโปรเจกต์ เช่น token ของบริการ monitoring กลาง, credential สำหรับ package registry ภายในองค์กร, license key ของเครื่องมือที่ใช้ทุกทีม

### 684.4 ลำดับความสำคัญเมื่อชื่อ Secret ซ้ำกัน

ถ้ามี secret ชื่อเดียวกันถูกตั้งไว้หลายระดับพร้อมกัน (เช่นมีทั้ง organization secret และ repository secret ชื่อ `API_KEY`) GitHub จะใช้กฎการ**เลือกค่าที่แคบที่สุดก่อน**:

```
Environment secret  (แคบที่สุด ชนะเสมอถ้ามี)
        ▲
        │ ถ้าไม่มี ให้ใช้
        │
Repository secret
        ▲
        │ ถ้าไม่มี ให้ใช้
        │
Organization secret (กว้างที่สุด)
```

พูดง่าย ๆ คือ **repository secret จะบังค่า organization secret ที่ชื่อเดียวกันเสมอ** และ **environment secret จะบังค่า repository secret ที่ชื่อเดียวกันเสมอ** หลักการนี้ทำให้แต่ละ repository หรือแต่ละ environment สามารถ "override" ค่าจากระดับที่กว้างกว่าได้เมื่อจำเป็น โดยไม่ต้องเปลี่ยนค่าที่ระดับ organization

### 684.5 ตารางสรุปเปรียบเทียบทั้ง 3 ระดับ

| คุณสมบัติ | Repository Secret | Environment Secret | Organization Secret |
|---|---|---|---|
| ขอบเขตการใช้งาน | ทั้ง repository | เฉพาะ job ที่ระบุ environment ตรงกัน | หลาย repository ในองค์กร |
| ใครตั้งค่าได้ | Repo admin | Repo admin | Org owner/admin |
| ผูกกับ Protection Rules ได้ไหม | ไม่ได้ | ได้ (ผ่าน environment) | ไม่ได้โดยตรง |
| ใช้แยกค่าตาม dev/staging/prod ได้ง่ายไหม | ทำได้แต่ต้องตั้งชื่อแยก (`DEV_API_KEY`, `PROD_API_KEY`) | ทำได้ง่าย ใช้ชื่อเดียวกันได้ | ไม่เหมาะ (ใช้ค่าเดียวกันทุก repo) |
| เหมาะกับ | โปรเจกต์เดี่ยว ไม่แยก environment | ทีมที่มีหลาย environment ชัดเจน | องค์กรที่มีหลาย repository ใช้ค่าเดียวกัน |

---

## Step 685: Environment Approval — ต้องมีคนอนุมัติก่อน Job จะรันต่อ

หนึ่งใน protection rule ที่สำคัญที่สุดของ Environment คือ **Required reviewers** ซึ่งทำให้ job หยุดรอการอนุมัติจากคนจริง ๆ ก่อนจะทำงานต่อ

### แนวคิด

> **เมื่อ job หนึ่งระบุ environment ที่มีการตั้งค่า Required reviewers ไว้ job นั้นจะหยุดค้างอยู่ในสถานะ "Waiting" จนกว่าจะมีคนในรายชื่อ reviewer เข้ามากดอนุมัติ (Approve) ผ่านหน้าเว็บของ GitHub เท่านั้น**

นี่คือกลไกที่ทำให้การ deploy production **ไม่มีทางเกิดขึ้นแบบอัตโนมัติทั้งหมด (fully automatic)** ได้อีกต่อไป จะต้องมี "human in the loop" เสมอ ซึ่งเป็นข้อกำหนดพื้นฐานของหลายองค์กรที่ต้องการ compliance

### วิธีตั้งค่า Required Reviewers

1. ไปที่ `Settings → Environments → (เลือก environment เช่น production)`
2. ในส่วน **Deployment protection rules** ติ๊ก **Required reviewers**
3. เพิ่มรายชื่อผู้ใช้หรือทีม (สูงสุด 6 คน/ทีมต่อ environment)
4. กด **Save protection rules**

> **หมายเหตุ:** ผู้ที่จะ push โค้ดที่ trigger workflow นั้น **ไม่สามารถอนุมัติ deployment ของตัวเองได้** (ถ้าเป็นคนเดียวในรายชื่อ reviewer และเป็นคน trigger ด้วย) เพื่อป้องกันการ self-approve ที่ขาดการตรวจสอบจากบุคคลที่สาม — ต้องมี reviewer อื่นอย่างน้อย 1 คนกดอนุมัติแทน

### ตัวอย่าง Workflow ที่มี Environment Approval

```yaml
name: Production Deployment

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Build application
        run: |
          echo "Building application..."
          mkdir -p dist
          echo "build output" > dist/app.txt

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production   # environment นี้ตั้งค่า Required reviewers ไว้แล้ว
    steps:
      - name: Deploy to production
        env:
          DEPLOY_KEY: ${{ secrets.PRODUCTION_DEPLOY_KEY }}
        run: echo "Deploying build to production servers..."
```

### สิ่งที่เกิดขึ้นจริงเมื่อรัน Workflow นี้

1. Job `build` รันจนเสร็จตามปกติ ไม่ต้องรออนุมัติ (เพราะไม่ได้ระบุ environment ที่มี protection)
2. Job `deploy` จะเข้าสู่สถานะ **"Waiting"** ทันที พร้อมข้อความ "Review pending deployments"
3. GitHub จะส่งการแจ้งเตือน (notification/email) ไปยังผู้ที่อยู่ใน required reviewers list
4. Reviewer เข้าไปที่หน้า **Actions** ของ repository จะเห็นปุ่ม **Review deployments**
5. Reviewer สามารถเลือก **Approve and deploy** หรือ **Reject** พร้อมใส่ความเห็นประกอบได้
6. ถ้า Approve → job `deploy` เริ่มทำงานต่อทันที และจะดึง secret ของ environment `production` มาใช้ได้
7. ถ้า Reject → job `deploy` จะจบด้วยสถานะ failure และไม่มีการรันคำสั่งภายในเลยแม้แต่บรรทัดเดียว

### แผนภาพลำดับการทำงาน

```
Push to main
     │
     ▼
┌─────────┐
│  build  │  ← รันทันที ไม่ต้องรออนุมัติ
└────┬────┘
     │ สำเร็จ
     ▼
┌───────────────────┐
│  deploy            │
│  environment:      │
│  production        │
│                    │
│  สถานะ: Waiting    │──────► ส่ง notification ให้ required reviewers
└─────────┬──────────┘
          │
   ┌──────┴───────┐
   │              │
Approve         Reject
   │              │
   ▼              ▼
รันคำสั่งจริง   job จบด้วย failure
เข้าถึง secret     ไม่มีการรันคำสั่งใด ๆ
ของ production
```

### การใช้ Wait Timer ร่วมกับ Required Reviewers

บาง team ต้องการ "ช่วงเวลาหน่วง" ก่อน deploy แม้จะอนุมัติแล้ว เพื่อให้มีโอกาสยกเลิกได้ทัน สามารถตั้งค่า **Wait timer** ควบคู่กันได้ เช่น ตั้งไว้ 10 นาที — job จะค้างในสถานะรอเวลาหลังผ่านการอนุมัติ (หรือแม้ไม่มี reviewer ก็ยังต้องรอครบเวลา) ก่อนจะเริ่มรันจริง ให้เวลาทีมตรวจสอบเพิ่มเติมหรือกดยกเลิกได้ทันหากพบปัญหา

### ทำไมเรื่องนี้ถึงสำคัญกับทีมจริง

- ป้องกันการ deploy โดยไม่ได้ตั้งใจจากการ merge PR ที่ไม่ควร merge
- สร้างขั้นตอนที่ตรงตามมาตรฐาน **Change Management** ที่องค์กรขนาดใหญ่ต้องปฏิบัติตาม (เช่น ISO 27001, SOC 2)
- ทำให้ audit trail สมบูรณ์ — รู้ชัดเจนว่าใครกดอนุมัติ deployment ไหน เมื่อไหร่ พร้อมความเห็นประกอบ

---

## Step 686: Artifacts คืออะไร — Upload และ Download

นอกจาก secrets และ environment แล้ว อีกหนึ่งฟีเจอร์พื้นฐานที่สำคัญมากคือ **Artifacts**

### Artifact คืออะไร

> **Artifact คือไฟล์หรือชุดไฟล์ที่ถูกสร้างขึ้นระหว่างการรัน workflow แล้วถูกเก็บไว้ (persist) บน GitHub เพื่อให้ดาวน์โหลดกลับมาดูได้ภายหลัง หรือส่งต่อให้ job อื่นในเวิร์กโฟลว์เดียวกันใช้งานต่อ**

ทำไมถึงต้องมี Artifact? เพราะ **แต่ละ job ใน workflow จะรันบน runner (เครื่องเสมือน) คนละเครื่องกันเสมอ** แม้จะอยู่ใน workflow เดียวกันก็ตาม เมื่อ job หนึ่งจบการทำงาน runner ของ job นั้นจะถูกทำลายทิ้งทันที **ไฟล์ทั้งหมดที่สร้างไว้ใน job นั้นจะหายไปด้วย** เว้นแต่จะถูก "อัปโหลดเป็น artifact" ไว้ก่อน

ตัวอย่างการใช้งาน Artifact ที่พบบ่อย:

- เก็บผลลัพธ์การ build (เช่นไฟล์ `.zip`, `dist/` folder, ไฟล์ binary)
- เก็บรายงานผลการทดสอบ (test report), coverage report
- เก็บ log file เพื่อ debug ภายหลัง
- ส่งต่อไฟล์ที่ build แล้วจาก job `build` ไปให้ job `deploy` ใช้ (รายละเอียดเต็มใน Step 687)
- เก็บ screenshot จากการทดสอบ end-to-end (เช่น Playwright, Cypress)

### 686.1 การอัปโหลด Artifact ด้วย `actions/upload-artifact`

```yaml
name: Build and Store Artifact

on: push

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/
          retention-days: 7
```

พารามิเตอร์สำคัญของ `actions/upload-artifact`:

| พารามิเตอร์ | ความหมาย |
|---|---|
| `name` | ชื่อของ artifact (ใช้อ้างอิงตอน download ภายหลัง) |
| `path` | path ของไฟล์หรือโฟลเดอร์ที่ต้องการเก็บ (รองรับ wildcard เช่น `**/*.log`) |
| `retention-days` | จำนวนวันที่จะเก็บ artifact ไว้ก่อนถูกลบอัตโนมัติ (ค่า default ขึ้นกับการตั้งค่าของ repo โดยทั่วไปคือ 90 วัน, กำหนดเองได้ระหว่าง 1-90 วัน) |
| `if-no-files-found` | พฤติกรรมเมื่อไม่พบไฟล์ตาม path ที่ระบุ (`warn` (ค่าเริ่มต้น), `error`, หรือ `ignore`) |
| `compression-level` | ระดับการบีบอัดไฟล์ (0-9) ค่ามากบีบอัดได้เล็กกว่าแต่ใช้เวลานานกว่า |

### 686.2 การอัปโหลดหลายไฟล์/หลาย path พร้อมกัน

```yaml
- name: Upload multiple paths
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: |
      coverage/
      test-report.xml
      logs/*.log
    if-no-files-found: error
```

### 686.3 การดาวน์โหลด Artifact ด้วย `actions/download-artifact`

Artifact สามารถดาวน์โหลดกลับมาใช้ได้ 2 แบบหลัก:

**แบบที่ 1: ดาวน์โหลดผ่านหน้าเว็บ (สำหรับมนุษย์)**

หลัง workflow รันเสร็จ เข้าไปที่หน้า **Actions → (เลือก workflow run นั้น)** จะเห็นส่วน **Artifacts** ที่ด้านล่างของหน้า พร้อมปุ่มดาวน์โหลดเป็น `.zip` ให้กดโหลดมาดูได้ทันที เหมาะสำหรับกรณีที่นักพัฒนาต้องการดูรายงานผลการทดสอบหรือ log ด้วยตาตัวเอง

**แบบที่ 2: ดาวน์โหลดภายใน job อื่นของ workflow เดียวกัน (สำหรับ automation)**

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  verify:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: build-output
          path: ./downloaded-dist

      - name: List downloaded files
        run: ls -la ./downloaded-dist
```

พารามิเตอร์สำคัญของ `actions/download-artifact`:

| พารามิเตอร์ | ความหมาย |
|---|---|
| `name` | ชื่อ artifact ที่ต้องการดาวน์โหลด (ต้องตรงกับตอน upload) |
| `path` | โฟลเดอร์ปลายทางที่จะนำไฟล์มาวาง (ถ้าไม่ระบุ จะวางไว้ที่ working directory) |
| `pattern` | ใช้เมื่อต้องการดาวน์โหลดหลาย artifact ที่ชื่อตรงกับ pattern (เช่น `build-*`) |
| `merge-multiple` | ถ้า `true` จะรวมไฟล์จากหลาย artifact ไว้ในโฟลเดอร์เดียวกันแทนที่จะแยกเป็นโฟลเดอร์ย่อยตามชื่อ |

### 686.4 อายุของ Artifact และการลบ

Artifact จะถูกเก็บไว้ตามระยะเวลาที่กำหนด (`retention-days`) หลังจากนั้นจะถูกลบออกจากระบบโดยอัตโนมัติ และไม่สามารถกู้คืนได้ ดังนั้น artifact **ไม่ใช่ที่เก็บถาวร (permanent storage)** — ถ้าต้องการเก็บผลลัพธ์การ build ไว้ถาวร ควรใช้ **GitHub Releases** หรือ push ไปยัง container registry / object storage แทน

### 686.5 ความแตกต่างระหว่าง Artifact กับ Cache (เกริ่นก่อนเจาะลึกใน Step 688)

หลายคนสับสนระหว่าง Artifact กับ Cache เพราะทั้งคู่คือการ "เก็บไฟล์ไว้ใช้ภายหลัง" แต่จุดประสงค์ต่างกันโดยสิ้นเชิง:

| | Artifact | Cache |
|---|---|---|
| จุดประสงค์ | เก็บ**ผลลัพธ์**ของงาน เพื่อดูหรือส่งต่อ | เก็บ**ข้อมูลที่ใช้ซ้ำ**เพื่อเร่งความเร็ว |
| ตัวอย่าง | build output, test report | `node_modules`, `~/.m2`, pip cache |
| ดาวน์โหลดผ่านหน้าเว็บได้ไหม | ได้ | ไม่ได้ (ใช้ภายใน workflow เท่านั้น) |
| ถ้าหายไป | งานที่ต้องพึ่งพามันจะพัง (ต้องมี artifact นั้นจริง) | แค่ทำงานช้าลง (สร้างใหม่ได้เสมอ) |
| อายุการเก็บ | กำหนดได้ 1-90 วัน | GitHub จะลบ cache ที่ไม่ถูกใช้เกิน 7 วัน หรือเมื่อรวมขนาด cache ทั้ง repo เกิน 10 GB |

---

## Step 687: การส่งต่อ Artifact ระหว่าง Job (Build → Deploy)

นี่คือ pattern การใช้งาน Artifact ที่พบบ่อยที่สุดในโลกจริง: **แยก job `build` ออกจาก job `deploy`** แล้วส่งต่อผลลัพธ์ของการ build ผ่าน artifact

### ทำไมต้องแยก Build กับ Deploy เป็นคนละ Job

1. **แยกความรับผิดชอบชัดเจน (Separation of concerns)** — job `build` ไม่จำเป็นต้องรู้จัก secret ของการ deploy เลย
2. **Build ครั้งเดียว ใช้ deploy ได้หลายที่** — build artifact ตัวเดียวกันสามารถนำไป deploy ทั้ง staging และ production ได้ โดยไม่ต้อง build ซ้ำ (รับประกันว่า deploy ของ 2 environment เป็นไฟล์เดียวกันเป๊ะ ไม่ build คนละครั้งแล้วได้ผลลัพธ์ต่างกัน)
3. **ประหยัดเวลา** — ถ้า deploy ไป 3 environment แต่ build ใหม่ทุกครั้ง จะเสียเวลาซ้ำซ้อนโดยไม่จำเป็น
4. **ควบคุม permission ได้ละเอียดขึ้น** — job `build` รันด้วย permission ต่ำ ในขณะที่ job `deploy` (ที่เข้าถึง secret จริง) รันแยกต่างหากพร้อม environment protection

### ตัวอย่างเต็มรูปแบบ

```yaml
name: Build then Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build production bundle
        run: npm run build

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: production-build
          path: dist/
          retention-days: 5

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: production-build
          path: ./dist

      - name: Deploy to staging server
        env:
          DEPLOY_TOKEN: ${{ secrets.STAGING_DEPLOY_TOKEN }}
        run: |
          echo "Uploading ./dist to staging..."
          # คำสั่ง deploy จริง เช่น rsync, scp, aws s3 sync ฯลฯ

  deploy-production:
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: production-build
          path: ./dist

      - name: Deploy to production server
        env:
          DEPLOY_TOKEN: ${{ secrets.PRODUCTION_DEPLOY_TOKEN }}
        run: |
          echo "Uploading ./dist to production..."
          # คำสั่ง deploy จริง
```

### วิเคราะห์การทำงานทีละส่วน

1. **`build`** — รันครั้งเดียว: checkout, ติดตั้ง dependency, รันเทสต์, build ไฟล์ แล้วอัปโหลดเป็น artifact ชื่อ `production-build`
2. **`deploy-staging`** — รอ `build` เสร็จก่อน (`needs: build`) แล้วดาวน์โหลด artifact เดิมมาใช้ ไม่ build ซ้ำ พร้อมระบุ `environment: staging`
3. **`deploy-production`** — รอทั้ง `build` และ `deploy-staging` เสร็จก่อน (`needs: [build, deploy-staging]`) หมายความว่าจะ deploy production ได้ก็ต่อเมื่อ deploy staging สำเร็จก่อนเท่านั้น เป็นการบังคับลำดับขั้นความปลอดภัย และเนื่องจากมี `environment: production` ที่ตั้ง required reviewers ไว้ job นี้จะรอการอนุมัติก่อนเริ่มด้วย

### แผนภาพเส้นทางข้อมูล

```
┌────────────────────────────────────────┐
│  Job: build                             │
│  1. npm ci                              │
│  2. npm test                            │
│  3. npm run build  →  สร้าง dist/       │
│  4. upload-artifact("production-build") │
└────────────────┬─────────────────────────┘
                 │  artifact: production-build
      ┌──────────┴───────────┐
      ▼                      ▼
┌─────────────────┐   ┌──────────────────────┐
│ deploy-staging   │   │ deploy-production      │
│ download-artifact│   │ (รอ staging สำเร็จก่อน) │
│ ("production-    │   │ download-artifact      │
│  build")         │   │ ("production-build")   │
│ → deploy จริง     │   │ → รอ approval          │
│                  │   │ → deploy จริง          │
└──────────────────┘   └──────────────────────┘
```

### ข้อควรระวังเรื่อง Artifact ข้าม Workflow

Artifact ที่ upload ไว้ **ใช้ได้เฉพาะภายใน workflow run เดียวกันเท่านั้น** (จะดาวน์โหลดข้าม job ได้ ตราบใดที่ยังอยู่ใน run เดียวกัน) ถ้าต้องการแชร์ไฟล์ข้าม workflow คนละ run กัน (เช่น workflow อื่นที่ trigger แยกต่างหาก) ต้องใช้วิธีอื่น เช่น เก็บไว้ใน GitHub Releases, container registry, หรือ cloud storage แทน — Artifact ไม่ได้ออกแบบมาเพื่อจุดประสงค์นั้น

---

## Step 688: Caching Dependencies ด้วย `actions/cache`

การติดตั้ง dependency ใหม่ทุกครั้งที่ workflow รัน (เช่น `npm ci` ที่ดาวน์โหลด package หลายร้อยตัวจาก npm registry) เป็นสิ่งที่ **กินเวลามากและไม่จำเป็น** ถ้า dependency ไม่ได้เปลี่ยนแปลงเลยตั้งแต่ครั้งก่อน

### แนวคิดของ Caching

> **Cache คือกลไกที่เก็บไฟล์หรือโฟลเดอร์ที่ใช้เวลาสร้างนาน (เช่น `node_modules`, `.m2/repository`, `~/.cache/pip`) ไว้ระหว่างการรัน workflow แต่ละครั้ง โดยจะดึงกลับมาใช้ทันทีถ้า "key" ของ cache ตรงกับที่เคยบันทึกไว้ แทนที่จะต้องสร้างใหม่ทั้งหมด**

### 688.1 การใช้ `actions/cache` แบบพื้นฐาน (ตัวอย่าง Node.js)

```yaml
name: Test with Cache

on: push

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Cache node_modules
        id: npm-cache
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: Install dependencies
        if: steps.npm-cache.outputs.cache-hit != 'true'
        run: npm ci

      - name: Run tests
        run: npm test
```

### 688.2 อธิบายแต่ละส่วนของ `key`

`key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}` คือหัวใจสำคัญของการทำ cache ให้ถูกต้อง:

- **`runner.os`** — ชื่อระบบปฏิบัติการของ runner (เช่น `Linux`) เพื่อไม่ให้ cache ของ Linux ไปปนกับ Windows หรือ macOS เพราะไฟล์ binary ต่างแพลตฟอร์มใช้แทนกันไม่ได้
- **`hashFiles('package-lock.json')`** — คำนวณ hash จากเนื้อหาไฟล์ `package-lock.json` ถ้าไฟล์นี้เปลี่ยน (เช่นมีการเพิ่ม/ลบ dependency) hash จะเปลี่ยนตาม ทำให้ **key ใหม่ไม่ตรงกับ cache เดิม** ระบบจะรู้ว่าต้องติดตั้งใหม่แทนที่จะใช้ของเก่าที่ล้าสมัย

นี่คือหลักการสำคัญที่สุดของการทำ cache ให้ปลอดภัย: **key ต้องเปลี่ยนทุกครั้งที่เนื้อหาที่ cache ไว้ควรจะเปลี่ยนตาม** ถ้า key คงที่ตลอดไปไม่ว่า dependency จะเปลี่ยนแค่ไหน จะทำให้ได้ dependency เวอร์ชันเก่าที่ไม่ตรงกับ `package-lock.json` ปัจจุบัน ซึ่งเป็นบั๊กที่ debug ยากมาก

### 688.3 `restore-keys` คืออะไร

```yaml
restore-keys: |
  ${{ runner.os }}-node-
```

`restore-keys` คือรายการ key สำรองที่ใช้ **เมื่อหา key หลักแบบตรงเป๊ะไม่เจอ** ระบบจะค้นหา cache ที่ชื่อขึ้นต้นด้วยข้อความนี้ที่ถูกบันทึกล่าสุด แล้วนำมาใช้แทน (แม้จะไม่ตรง `package-lock.json` เป๊ะ 100%) วิธีนี้ช่วยให้ `npm ci` ยังคงเร็วขึ้นบางส่วน (บาง package ที่ไม่เปลี่ยนจะถูกใช้จาก cache ที่ใกล้เคียง) แม้ว่า `package-lock.json` จะมีการแก้ไขเล็กน้อยก็ตาม

### 688.4 ทางลัด: `actions/setup-node` มี built-in cache

ในทางปฏิบัติ สำหรับ Node.js เราไม่จำเป็นต้องเขียน `actions/cache` เองเลยด้วยซ้ำ เพราะ `actions/setup-node` มีพารามิเตอร์ `cache` ในตัวที่ทำสิ่งเดียวกันให้อัตโนมัติ:

```yaml
- name: Setup Node.js with built-in cache
  uses: actions/setup-node@v4
  with:
    node-version: "20"
    cache: "npm"          # รองรับ npm, yarn, pnpm

- name: Install dependencies
  run: npm ci
```

วิธีนี้สั้นกว่าและแนะนำให้ใช้เมื่อทำได้ เพราะ action ทางการของแต่ละภาษาส่วนใหญ่ (`setup-node`, `setup-python`, `setup-java`, `setup-go`) มักมี built-in caching รองรับให้แล้ว โดยไม่ต้องคำนวณ `hashFiles` เอง

### 688.5 ตัวอย่าง Caching สำหรับภาษาอื่น ๆ

**Python (pip):**

```yaml
- name: Setup Python
  uses: actions/setup-python@v5
  with:
    python-version: "3.12"
    cache: "pip"

- name: Install dependencies
  run: pip install -r requirements.txt
```

**Java (Maven) แบบเขียน `actions/cache` เอง:**

```yaml
- name: Cache Maven packages
  uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
    restore-keys: |
      ${{ runner.os }}-maven-
```

**Go modules:**

```yaml
- name: Cache Go modules
  uses: actions/cache@v4
  with:
    path: |
      ~/go/pkg/mod
      ~/.cache/go-build
    key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
    restore-keys: |
      ${{ runner.os }}-go-
```

### 688.6 ข้อจำกัดของ Cache ที่ควรรู้

| ข้อจำกัด | รายละเอียด |
|---|---|
| **ขนาดรวมสูงสุด** | แต่ละ repository เก็บ cache รวมกันได้ไม่เกิน 10 GB เมื่อเกิน GitHub จะลบ cache ที่เก่าที่สุดออกก่อนโดยอัตโนมัติ |
| **อายุการเก็บ** | Cache ที่ไม่ถูกเข้าถึงเลยเกิน 7 วัน จะถูกลบทิ้งอัตโนมัติ |
| **ขอบเขตของ branch** | Cache ที่สร้างจาก branch หนึ่งจะถูกใช้ร่วมกับ branch อื่นได้แบบมีเงื่อนไข (โดยทั่วไปจะแชร์ได้กับ base branch หรือ branch เดียวกัน) แต่ pull request จะเข้าถึง cache ของ base branch ได้เพื่อให้ CI เร็วขึ้นตั้งแต่รันครั้งแรก |
| **ไม่ควรใช้เก็บ secret** | Cache ไม่ได้ถูกเข้ารหัสแบบเดียวกับ Secrets และสามารถถูกดาวน์โหลดออกมาดูได้ในบางกรณี (เช่นผ่าน PR ที่แชร์ cache กับ base branch) จึงห้ามใช้ cache เก็บข้อมูลที่เป็นความลับเด็ดขาด |

---

## Step 689: ความปลอดภัยของ Secrets — Masking และข้อจำกัดกับ Fork

Secrets ถูกออกแบบมาให้ปลอดภัยตั้งแต่ต้น แต่ก็ยังมีรายละเอียดสำคัญที่ทุกคนต้องเข้าใจ เพื่อไม่ให้ประมาทจนเผลอทำข้อมูลลับหลุด

### 689.1 การ Mask ค่า Secret ใน Log โดยอัตโนมัติ

ทุกครั้งที่ workflow ใช้ค่า `${{ secrets.XXX }}` GitHub Actions จะจดจำค่านั้นไว้ และถ้าค่านั้น**ปรากฏขึ้นที่ไหนก็ตามใน log ของ job** (ไม่ว่าจะตั้งใจ echo ออกมาตรง ๆ หรือหลุดออกมาจาก error message) ระบบจะแทนที่ด้วยเครื่องหมาย `***` โดยอัตโนมัติทันที

ตัวอย่าง:

```yaml
- name: Accidentally print secret
  env:
    API_KEY: ${{ secrets.DEPLOY_API_KEY }}
  run: echo "The key is: ${API_KEY}"
```

ผลลัพธ์ที่แสดงใน log จะเป็น:

```
The key is: ***
```

**แต่การ masking นี้ไม่ใช่เกราะป้องกันสมบูรณ์แบบ** มีข้อจำกัดที่สำคัญมาก:

1. **Mask ทำงานแบบ exact string match เท่านั้น** — ถ้านำค่า secret ไป encode เป็น base64 ก่อน แล้ว echo ค่าที่ encode แล้วออกมา ระบบจะ **มองไม่เห็นว่ามันคือ secret** เพราะสตริงที่ปรากฏจริงไม่ตรงกับค่าเดิมเป๊ะ จึงไม่ถูก mask
2. **Secret ที่สั้นเกินไป (เช่นสั้นกว่า 3-4 ตัวอักษร) อาจไม่ถูก mask** เพราะเสี่ยงจะไป mask ข้อความปกติที่ไม่เกี่ยวข้องจนทำให้ log อ่านไม่รู้เรื่อง
3. **Mask ใช้ได้กับ log ที่ GitHub ควบคุมเท่านั้น** — ถ้า workflow ส่งค่า secret ออกไปยังบริการภายนอก (เช่น log เข้า third-party monitoring service) การ mask ของ GitHub จะไม่มีผลกับปลายทางนั้น ต้องระวังเอง

**บทเรียนสำคัญ:** อย่าพึ่งพา masking เป็นเกราะป้องกันเดียว ต้อง**ระมัดระวังตั้งแต่การเขียน workflow** ไม่ให้ secret ไปปรากฏในที่ที่ไม่ควรอยู่ตั้งแต่แรก

### 689.2 ข้อจำกัดสำคัญที่สุด: Pull Request จาก Fork ภายนอกไม่ได้รับ Secret

นี่คือกลไกความปลอดภัยที่สำคัญที่สุดข้อหนึ่งของ GitHub Actions ที่ทุกคนต้องเข้าใจให้ถูกต้อง:

> **เมื่อมีคนภายนอก (ที่ไม่ใช่ collaborator ของ repository) fork repository แล้วเปิด Pull Request กลับมา workflow ที่ trigger ด้วย event `pull_request` จากฝั่ง fork นั้น จะไม่ได้รับสิทธิ์เข้าถึง secrets ใด ๆ ของ repository ต้นทางเลย (ค่าที่ได้จะเป็นค่าว่างเปล่า)**

### เหตุผลที่ต้องมีข้อจำกัดนี้

ลองจินตนาการว่าไม่มีข้อจำกัดนี้: ใครก็ตามในโลกสามารถ fork repository ของคุณ แล้วเขียน workflow ที่แอบส่ง secret ออกไปยังเซิร์ฟเวอร์ของตัวเอง จากนั้นเปิด PR กลับมา — ถ้า secret ถูกส่งให้กับ workflow ที่มาจาก PR นี้ ผู้ไม่หวังดีจะขโมยค่าลับของคุณไปได้ทันทีโดยที่คุณยังไม่ทันได้ merge หรือแม้แต่รีวิวโค้ดเลยด้วยซ้ำ

นี่จึงเป็นเหตุผลที่ GitHub ออกแบบให้ event `pull_request` (ไม่ใช่ `pull_request_target`) จาก fork **ไม่ได้รับ secrets และ `GITHUB_TOKEN` ก็จะมีสิทธิ์แบบ read-only เท่านั้น**

### ตารางสรุปพฤติกรรมของ Secret ตามแหล่งที่มาของ Event

| สถานการณ์ | ได้รับ Secrets ไหม | หมายเหตุ |
|---|---|---|
| Push จากภายใน repo (branch ปกติ) | ได้ | ทำงานตามปกติ |
| Pull Request ระหว่าง branch ภายใน repo เดียวกัน | ได้ | เพราะไม่ใช่ fork ภายนอก |
| Pull Request จาก **fork ภายนอก** ด้วย event `pull_request` | **ไม่ได้** | ค่า secret จะเป็นค่าว่างเปล่า เพื่อความปลอดภัย |
| Pull Request จาก fork ด้วย event `pull_request_target` | ได้ (มีความเสี่ยงถ้าใช้ไม่ระวัง) | รันด้วย context ของ base repo แต่ยัง checkout โค้ดจาก fork ได้ ต้องระวังการรันโค้ดที่ไม่น่าเชื่อถือด้วย permission สูง |
| Workflow ที่ maintainer กด "Approve and run" ให้ PR จาก fork | ขึ้นกับ event ที่ใช้ | การ approve แค่อนุญาตให้ workflow รัน ไม่ได้เปลี่ยนกฎเรื่อง secrets ของ `pull_request` event |

### ข้อควรระวังเรื่อง `pull_request_target`

`pull_request_target` เป็น event ที่ถูกสร้างมาเพื่อแก้ปัญหาบางอย่าง (เช่นต้องการ comment กลับไปที่ PR จาก fork) แต่มันมีความเสี่ยงสูงมากถ้าใช้ผิดวิธี เพราะมันให้สิทธิ์และ secrets เท่ากับ workflow ที่รันจาก base repository ทั้งที่โค้ดที่ checkout มาอาจเป็นโค้ดจาก fork ที่ไม่น่าเชื่อถือ

**หลักการที่ต้องยึดเสมอ:** ถ้าจำเป็นต้องใช้ `pull_request_target` ห้าม checkout โค้ดจาก fork แล้วนำไป `run:` โดยตรงเด็ดขาด (เช่นห้าม `npm install` หรือรันสคริปต์จากโค้ดของ fork นั้น) เพราะเท่ากับเปิดทางให้โค้ดที่ไม่รู้จักรันด้วยสิทธิ์เข้าถึง secret ของ repository จริง

### 689.3 แนวทางปฏิบัติเพื่อความปลอดภัยของ Secrets (สรุปเป็นข้อ)

1. **ใช้ secret เท่าที่จำเป็นเท่านั้น** — อย่าส่ง secret เข้าไปใน step ที่ไม่ได้ใช้งานมัน
2. **จำกัดสิทธิ์ของ `GITHUB_TOKEN`** ด้วย `permissions:` ในระดับ workflow หรือ job ให้แคบที่สุดเท่าที่จำเป็น
3. **หมุนเวียน (rotate) secret เป็นระยะ** โดยเฉพาะถ้ามีพนักงานลาออกจากทีมที่เคยเข้าถึง secret นั้น
4. **อย่า echo หรือ print ค่า secret ออกมาทั้งค่า** แม้จะคิดว่า "แค่ debug ชั่วคราว" เพราะอาจลืมลบออกและถูก commit ค้างไว้
5. **ใช้ Environment secret + Required reviewers** สำหรับ secret ที่มีความอ่อนไหวสูง เช่น production credential
6. **ตรวจสอบ third-party Actions ก่อนใช้งาน** — action ที่มาจากแหล่งที่ไม่น่าเชื่อถืออาจถูกออกแบบมาเพื่อขโมย secret โดยเฉพาะ ควร pin version ด้วย commit SHA แทนการใช้ tag ลอย ๆ สำหรับ action ที่สำคัญ
7. **เปิดใช้ Dependabot alerts และ secret scanning** ของ GitHub เพื่อตรวจจับกรณีที่ secret หลุดเข้าไปใน commit โดยไม่ได้ตั้งใจ

---

## Step 690: แบบฝึกหัด — ตั้งค่า Secret, Environment และ Artifact ครบวงจร

ถึงเวลาลงมือทำจริง! แบบฝึกหัดนี้จะให้คุณสร้างระบบเล็ก ๆ ที่ครบทั้ง 3 หัวข้อของ Part นี้: Secret, Environment, และ Artifact

> **ข้อควรระวังสำคัญ:** ในแบบฝึกหัดนี้ให้ใช้ **ค่าปลอมที่มองเห็นชัดเจนว่าเป็นตัวอย่าง** เท่านั้น เช่น `example-fake-key-do-not-use` ห้ามใช้ API key หรือ token จริงของบริการใด ๆ เด็ดขาด แม้จะเป็นบัญชีทดสอบก็ตาม เพื่อป้องกันไม่ให้เกิดปัญหาค่าลับหลุดเข้า git history โดยไม่ได้ตั้งใจ

### 690.1 ขั้นตอนที่ 1 — เตรียม Repository สำหรับฝึกฝน

```bash
mkdir -p ~/git-course/part-69-secrets-environments
cd ~/git-course/part-69-secrets-environments
git init
mkdir -p src
echo "console.log('Hello from build!');" > src/app.js
git add .
git commit -m "chore: initial commit for Part 69 exercise"
```

จากนั้นสร้าง repository บน GitHub (ผ่านหน้าเว็บหรือ `gh repo create`) แล้ว push ขึ้นไป:

```bash
gh repo create part-69-secrets-environments --private --source=. --push
```

### 690.2 ขั้นตอนที่ 2 — สร้าง Repository Secret (ค่าปลอมเท่านั้น)

ตั้งค่า secret ชื่อ `NOTIFY_WEBHOOK` ด้วยค่าปลอมที่ชัดเจนว่าไม่ใช่ของจริง:

```bash
gh secret set NOTIFY_WEBHOOK --body "https://example.invalid/webhook/<PLACEHOLDER>"
```

หรือทำผ่านหน้าเว็บที่ `Settings → Secrets and variables → Actions → New repository secret` แล้วใส่ค่า `<PLACEHOLDER_ONLY_FOR_LEARNING>` ในช่อง Secret

### 690.3 ขั้นตอนที่ 3 — สร้าง Environment สองตัว

สร้าง environment ชื่อ `staging` และ `production` ที่ `Settings → Environments → New environment`

สำหรับ `production` ให้ตั้งค่า protection rule เพิ่มเติม:

- ติ๊ก **Required reviewers** แล้วเพิ่มชื่อบัญชีของคุณเอง (หรือเพื่อนร่วมทีมถ้ามี) เข้าไปในรายชื่อ
- ตั้งค่า **Deployment branches** ให้อนุญาตเฉพาะ branch `main` เท่านั้น

จากนั้นเพิ่ม **Environment secret** ชื่อ `DEPLOY_TOKEN` ให้ทั้งสอง environment โดยใช้ค่าปลอมคนละค่ากัน เช่น:

- `staging` → `DEPLOY_TOKEN` = `example-staging-token-placeholder`
- `production` → `DEPLOY_TOKEN` = `example-production-token-placeholder`

### 690.4 ขั้นตอนที่ 4 — เขียน Workflow ครบวงจร

สร้างไฟล์ `.github/workflows/build-deploy.yml`:

```yaml
name: Build, Cache, and Deploy

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js with cache
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Simulate build
        run: |
          mkdir -p dist
          cp src/app.js dist/app.js
          echo "built at $(date -u +%Y-%m-%dT%H:%M:%SZ)" >> dist/build-info.txt

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-build
          path: dist/
          retention-days: 5

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: app-build
          path: ./dist

      - name: Show downloaded files
        run: ls -la ./dist

      - name: Notify webhook (staging)
        env:
          WEBHOOK: ${{ secrets.NOTIFY_WEBHOOK }}
          TOKEN: ${{ secrets.DEPLOY_TOKEN }}
        run: |
          echo "Deploying to staging..."
          echo "Token length (masked value hidden): ${#TOKEN} characters"
          echo "Would notify webhook at: ${WEBHOOK}"

  deploy-production:
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.invalid/production
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: app-build
          path: ./dist

      - name: Deploy to production
        env:
          TOKEN: ${{ secrets.DEPLOY_TOKEN }}
        run: |
          echo "Deploying to production after approval..."
          echo "Token length (masked value hidden): ${#TOKEN} characters"
```

### 690.5 ขั้นตอนที่ 5 — Push และสังเกตพฤติกรรม

```bash
git add .github/workflows/build-deploy.yml
git commit -m "ci: add build, cache, environment, and artifact workflow"
git push
```

จากนั้นไปที่แท็บ **Actions** บน GitHub แล้วสังเกตสิ่งต่อไปนี้:

1. Job `build` รันจนเสร็จ และมี artifact ชื่อ `app-build` ปรากฏให้ดาวน์โหลดที่ด้านล่างของหน้า workflow run
2. Job `deploy-staging` รันต่อทันที (เพราะ `staging` ไม่มี required reviewers) และดาวน์โหลด artifact จาก job `build` มาใช้ได้สำเร็จ
3. Job `deploy-production` ค้างอยู่ในสถานะ **Waiting** — ให้คุณกดเข้าไปที่ปุ่ม **Review deployments** แล้วกด **Approve and deploy** เพื่อทดสอบขั้นตอน approval ด้วยตัวเอง
4. ตรวจสอบใน log ของแต่ละ job ว่าไม่มีค่า secret จริงปรากฏออกมาเลย มีแต่ความยาวของ token ที่แสดงเป็นตัวเลขเท่านั้น
5. รัน workflow ซ้ำอีกครั้งโดยไม่แก้ `package-lock.json` แล้วสังเกตว่า step "Setup Node.js with cache" ใช้เวลาน้อยลงจากการดึง cache กลับมาใช้

### 690.6 ขั้นตอนเสริม (ถ้าต้องการฝึกเพิ่ม)

- ลองเปลี่ยนชื่อ artifact ตอน download ให้ไม่ตรงกับตอน upload แล้วสังเกต error message ที่เกิดขึ้น
- ลองลบ required reviewers ออกจาก `production` แล้วสังเกตว่า job `deploy-production` รันทันทีโดยไม่รอ approval
- ลองเปลี่ยน `Deployment branches` ของ `production` ให้อนุญาตเฉพาะ branch ที่ไม่มีอยู่จริง (เช่น `release-only`) แล้ว push จาก `main` ดูว่า job ล้มเหลวด้วยเหตุผลอะไร
- ลองสร้าง Pull Request จาก fork (ถ้ามีบัญชี GitHub สำรอง) แล้วสังเกตว่า secret ที่ได้รับในระหว่างการรัน workflow ของ PR นั้นเป็นค่าว่างเปล่าตามที่อธิบายไว้ใน Step 689

### 690.7 Checklist ก่อนไป Part ถัดไป

- [ ] เข้าใจว่า GitHub Secrets คืออะไร และทำไมห้ามเขียนค่าลับลงในไฟล์ workflow ตรง ๆ
- [ ] สร้างและเรียกใช้ repository secret ผ่าน `${{ secrets.NAME }}` ได้จริง
- [ ] เข้าใจแนวคิดของ Environment และสร้าง environment พร้อม protection rules ได้
- [ ] เข้าใจความแตกต่างระหว่าง Repository Secret, Environment Secret, และ Organization Secret รวมถึงลำดับความสำคัญเมื่อชื่อซ้ำกัน
- [ ] ตั้งค่า Required reviewers และทดสอบขั้นตอน approve deployment ได้จริง
- [ ] เข้าใจการใช้ `actions/upload-artifact` และ `actions/download-artifact`
- [ ] เขียน workflow ที่ส่งต่อ artifact จาก job build ไปยัง job deploy ได้สำเร็จ
- [ ] เข้าใจและใช้งาน `actions/cache` เพื่อเร่งความเร็วของ dependency installation
- [ ] เข้าใจข้อจำกัดของการ mask secret ใน log และรู้ว่า PR จาก fork ภายนอกไม่ได้รับ secret ตาม default
- [ ] ทำแบบฝึกหัดครบวงจรจนเห็น workflow ทำงานจริงตั้งแต่ build จนถึง production ที่ต้องผ่านการอนุมัติ

---

## สรุป Part 69

ใน Part นี้เราได้เรียนรู้ว่า:

1. **GitHub Secrets** คือพื้นที่เก็บข้อมูลลับที่เข้ารหัสไว้ แยกออกจากซอร์สโค้ดอย่างสมบูรณ์ เรียกใช้ผ่าน `${{ secrets.NAME }}` และควรส่งผ่าน `env:` ก่อนใช้ใน `run:` เสมอเพื่อความปลอดภัย
2. **Environment** ใช้แทนสภาพแวดล้อมการ deploy จริง (development, staging, production) พร้อมแนบ protection rules เช่น required reviewers, wait timer, และ deployment branch restrictions
3. Secrets มี 3 ระดับ: **Repository, Environment, Organization** โดย environment secret จะบังค่า repository secret และ repository secret จะบังค่า organization secret ที่ชื่อเดียวกันเสมอ (ระดับที่แคบกว่าชนะ)
4. **Required reviewers** ทำให้การ deploy ต้องมีมนุษย์กดอนุมัติก่อนเสมอ ป้องกันการ deploy ที่ผิดพลาดหรือไม่ได้ตั้งใจไปยัง production
5. **Artifacts** ใช้เก็บผลลัพธ์จาก job ด้วย `actions/upload-artifact` และดึงกลับมาใช้ด้วย `actions/download-artifact` — เหมาะสำหรับการส่งต่อไฟล์ build ระหว่าง job โดยไม่ต้อง build ซ้ำ
6. **Caching** ด้วย `actions/cache` (หรือ built-in cache ของ `setup-node`/`setup-python`) ช่วยลดเวลาในการติดตั้ง dependency ได้อย่างมาก โดยต้องออกแบบ `key` ให้เปลี่ยนตามเนื้อหาที่แท้จริงเสมอ
7. GitHub Actions จะ **mask ค่า secret ใน log อัตโนมัติ** แต่ไม่ใช่เกราะป้องกันสมบูรณ์แบบ และ **Pull Request จาก fork ภายนอกจะไม่ได้รับ secrets ตาม default** ซึ่งเป็นกลไกความปลอดภัยที่สำคัญที่สุดข้อหนึ่งที่ต้องเข้าใจก่อนเขียน workflow ที่ทำงานกับ PR จากภายนอก

**ต่อไป:** [Part 70: GitHub Actions: Custom Actions และ Reusable Workflows](./part-070-github-actions-custom-reusable.md)
