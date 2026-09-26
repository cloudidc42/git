# Part 70: GitHub Actions: Custom Actions และ Reusable Workflows

> **Step ในหลักสูตรนี้:** Step 691–700
> **เฟส:** 7 — CI/CD เต็มรูปแบบด้วย GitHub Actions และ GitLab CI/CD
> **เป้าหมายของ Part นี้:** เข้าใจว่า Custom Action คืออะไร สร้าง Action เองได้ทั้ง 3 แบบ (Composite, JavaScript, Docker) รู้วิธี publish ขึ้น GitHub Marketplace เข้าใจ Reusable Workflows และวิธีเรียกใช้งานข้าม repository พร้อมเปรียบเทียบว่าเมื่อไหร่ควรเลือกใช้แบบไหน และวางแนวทาง Best Practice สำหรับองค์กรที่มีหลาย repository

---

## สารบัญของ Part นี้

- Step 691: Custom Action คืออะไร สร้างเองได้ 3 แบบ (Composite, JavaScript, Docker)
- Step 692: สร้าง Composite Action ง่าย ๆ (`action.yml` พร้อม `runs.using: composite`)
- Step 693: สร้าง JavaScript Action เบื้องต้น (`runs.using: node20`, ใช้ `@actions/core`)
- Step 694: สร้าง Docker Action เบื้องต้น (`runs.using: docker`, `Dockerfile`)
- Step 695: การ publish Action ขึ้น GitHub Marketplace
- Step 696: Reusable Workflows คืออะไร (`on: workflow_call`)
- Step 697: เรียกใช้ reusable workflow ด้วย `uses:` พร้อมส่ง `inputs`/`secrets`
- Step 698: เปรียบเทียบ Composite Action vs Reusable Workflow
- Step 699: Best Practices จัดระเบียบ workflow ในองค์กรที่มีหลาย repository
- Step 700: แบบฝึกหัด — สร้าง Composite Action + Reusable Workflow ของตัวเอง

---

## Step 691: Custom Action คืออะไร สร้างเองได้ 3 แบบ (Composite Action, JavaScript Action, Docker Action)

ตลอด Part ก่อนหน้านี้ในเฟส 7 เราใช้ **Action สำเร็จรูป** จาก GitHub Marketplace มาตลอด เช่น `actions/checkout@v4`, `actions/setup-node@v4`, `docker/build-push-action@v5` เป็นต้น สิ่งเหล่านี้คือ **Custom Action ที่คนอื่นสร้างไว้แล้วแบ่งปันให้เราใช้ฟรี**

คำถามคือ: **เราสร้าง Action ของเราเองได้ไหม?** คำตอบคือได้แน่นอน และเป็นทักษะสำคัญมากเมื่อทำงานในทีม/องค์กรที่มี workflow ซ้ำ ๆ กันหลาย repository

### ทำไมต้องสร้าง Custom Action เอง

1. **ลดความซ้ำซ้อน (DRY — Don't Repeat Yourself)** — ถ้าทุก repository ในองค์กรต้องรัน step เดิม ๆ เช่น "setup environment + notify Slack" การเขียนซ้ำทุกที่ทำให้ maintain ยาก
2. **encapsulate logic ที่ซับซ้อน** — บาง logic ซับซ้อนเกินกว่าจะเขียนเป็น shell script บรรทัดเดียวใน YAML จึงควรห่อเป็น Action แยกต่างหาก
3. **แชร์ให้ทีมอื่นหรือสาธารณะใช้** — เผยแพร่ขึ้น GitHub Marketplace ให้คนทั้งโลกใช้งานได้
4. **ทดสอบและ version ได้อย่างอิสระ** — Action มี version ของตัวเอง (tag) แยกจาก workflow ที่เรียกใช้มัน ทำให้ควบคุมการอัปเดตได้ละเอียด

### Custom Action มี 3 แบบหลัก

GitHub รองรับการสร้าง Action 3 รูปแบบ โดยแต่ละแบบมีจุดเด่น-จุดด้อยต่างกัน:

| แบบ | ทำงานอย่างไร | ใช้ภาษาอะไร | ความเร็ว | เหมาะกับ |
|---|---|---|---|---|
| **Composite Action** | รวม step หลาย ๆ อันเข้าด้วยกันเป็น Action เดียว | YAML + shell script | เร็วที่สุด (ไม่ต้อง build/pull image) | รวม step ซ้ำ ๆ ที่ใช้ action อื่นประกอบกัน |
| **JavaScript Action** | รันโค้ด JavaScript/TypeScript บน Node.js runtime โดยตรง | JavaScript/TypeScript | เร็ว (รันตรงบน runner ไม่ต้อง containerize) | Logic ที่ซับซ้อน ต้องเรียก GitHub API หรือจัดการ input/output ละเอียด |
| **Docker Action** | รันภายใน Docker container ที่กำหนด environment เองทั้งหมด | ภาษาอะไรก็ได้ (อยู่ใน container) | ช้าที่สุด (ต้อง build/pull image ก่อนรัน) | ต้องการ environment เฉพาะทาง หรือ dependency ที่ติดตั้งยากบน runner ปกติ |

### โครงสร้างไฟล์พื้นฐานที่ทุก Action ต้องมี

ไม่ว่าจะเป็นแบบไหน ทุก Action ต้องมีไฟล์ **`action.yml`** (หรือ `action.yaml`) อยู่ที่ root ของ repository (หรือ root ของโฟลเดอร์ย่อยถ้าเก็บหลาย Action ไว้ใน repository เดียว) ไฟล์นี้คือ **"ใบสมัคร" ที่บอก GitHub Actions ว่า Action นี้ชื่ออะไร รับ input อะไร คืน output อะไร และรันอย่างไร**

โครงสร้างพื้นฐานของ `action.yml`:

```yaml
name: 'ชื่อ Action'
description: 'คำอธิบายสั้น ๆ ว่า Action นี้ทำอะไร'
author: 'ชื่อผู้เขียนหรือองค์กร'

inputs:
  input-name:
    description: 'คำอธิบาย input นี้'
    required: true
    default: 'ค่า default ถ้ามี'

outputs:
  output-name:
    description: 'คำอธิบาย output นี้'
    value: ${{ steps.some-step.outputs.some-value }}

runs:
  using: 'composite'   # หรือ 'node20' หรือ 'docker'
  # รายละเอียดต่อจากนี้ขึ้นอยู่กับประเภทของ Action

branding:
  icon: 'zap'
  color: 'blue'
```

ส่วน `branding` เป็น optional ใช้กำหนดไอคอนและสีที่จะแสดงบน GitHub Marketplace เท่านั้น ไม่มีผลต่อการทำงาน

### แผนภาพรวมของ Part นี้

```
Custom Actions (3 แบบ)
├── Composite Action  (Step 692) → รวม step YAML
├── JavaScript Action (Step 693) → รันโค้ด JS บน Node.js
└── Docker Action     (Step 694) → รันใน container

Publish ขึ้น Marketplace (Step 695)

Reusable Workflows (Step 696-697)
├── workflow_call
└── uses: + inputs/secrets

เปรียบเทียบ + Best Practice (Step 698-699)

แบบฝึกหัดสร้างเอง (Step 700)
```

ต่อไปเรามาลงมือสร้าง Action ทั้ง 3 แบบทีละแบบ

---

## Step 692: สร้าง Composite Action ง่าย ๆ (`action.yml` พร้อม `runs.using: composite`)

**Composite Action** คือ Action ที่ง่ายที่สุดในการสร้าง เพราะมันคือการ **"ห่อ" หลาย ๆ step ที่เราเขียนใน workflow ปกติอยู่แล้วให้กลายเป็น Action เดียว** ไม่ต้องเรียนรู้ภาษาใหม่ ไม่ต้อง build Docker image — ใช้ YAML และ shell script ที่เราคุ้นเคยอยู่แล้ว

### สถานการณ์ตัวอย่าง

สมมติว่าทุก repository Node.js ในองค์กรของเราต้องทำ 3 อย่างเหมือนกันทุกครั้งก่อนรัน job:

1. Checkout โค้ด
2. ติดตั้ง Node.js เวอร์ชันที่กำหนด
3. รัน `npm ci` เพื่อติดตั้ง dependency แบบ deterministic
4. Cache `node_modules` เพื่อความเร็ว

แทนที่จะ copy-paste 4 step นี้ในทุก workflow file ของทุก repository เราจะห่อมันเป็น Composite Action ชื่อ `setup-node-project`

### โครงสร้าง Repository ของ Action

```
setup-node-project/
├── action.yml
└── README.md
```

### เขียน `action.yml`

```yaml
name: 'Setup Node Project'
description: 'Checkout, setup Node.js, ติดตั้ง dependency พร้อม cache ให้อัตโนมัติ'
author: 'platform-team'

inputs:
  node-version:
    description: 'Node.js version ที่จะใช้'
    required: false
    default: '20'
  working-directory:
    description: 'โฟลเดอร์ที่มี package.json'
    required: false
    default: '.'

outputs:
  cache-hit:
    description: 'บอกว่า cache ของ node_modules ถูก hit หรือไม่'
    value: ${{ steps.cache-deps.outputs.cache-hit }}

runs:
  using: 'composite'
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}

    - name: Cache node_modules
      id: cache-deps
      uses: actions/cache@v4
      with:
        path: ${{ inputs.working-directory }}/node_modules
        key: ${{ runner.os }}-node-${{ inputs.node-version }}-${{ hashFiles(format('{0}/package-lock.json', inputs.working-directory)) }}

    - name: Install dependencies
      if: steps.cache-deps.outputs.cache-hit != 'true'
      shell: bash
      working-directory: ${{ inputs.working-directory }}
      run: npm ci

    - name: Print summary
      shell: bash
      run: echo "ติดตั้ง dependency เสร็จแล้วด้วย Node.js ${{ inputs.node-version }}"
```

### จุดที่ต้องระวังเป็นพิเศษใน Composite Action

1. **ทุก step ที่เป็น `run:` ต้องมี `shell:` ระบุเสมอ** — ต่างจาก workflow ปกติที่ GitHub เดา shell ให้อัตโนมัติ (bash บน Linux/macOS, pwsh บน Windows) ใน Composite Action เราต้องระบุ `shell: bash` (หรือ `shell: pwsh`, `shell: sh` ฯลฯ) ทุกครั้งไม่งั้นจะ error ทันที
2. **อ้างอิง input ด้วย `${{ inputs.ชื่อ-input }}`** ไม่ใช่ `${{ github.event.inputs.ชื่อ }}` (แบบนั้นใช้กับ `workflow_dispatch`)
3. **`working-directory:` ใช้ได้กับ step ที่เป็น `run:`** แต่ถ้าเป็น `uses:` (เรียก action อื่นซ้อนใน composite action) จะไม่มี `working-directory` ให้ใช้โดยตรง ต้องส่งผ่าน `with:` ของ action นั้นแทนถ้า action รองรับ
4. **output ต้องดึงจาก `steps.<id>.outputs.<key>`** เหมือน workflow ปกติทุกประการ

### วิธีเรียกใช้ Composite Action จาก workflow อื่น

ถ้า Action อยู่ใน repository เดียวกัน (เก็บไว้ในโฟลเดอร์ `.github/actions/setup-node-project/`):

```yaml
name: CI

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup project
        id: setup
        uses: ./.github/actions/setup-node-project
        with:
          node-version: '20'
          working-directory: 'packages/api'

      - name: แสดงผล cache-hit
        run: echo "Cache hit คือ ${{ steps.setup.outputs.cache-hit }}"

      - run: npm test
        working-directory: 'packages/api'
```

ข้อสังเกต: เมื่อ Action อยู่ใน repository เดียวกัน เราใช้ path แบบ `uses: ./.github/actions/ชื่อโฟลเดอร์` (ต้องขึ้นต้นด้วย `./`) และต้อง `actions/checkout@v4` ก่อนเสมอ เพราะ runner ต้องมีไฟล์ `action.yml` อยู่ใน working directory จริง ๆ ก่อนถึงจะเรียกใช้ path แบบนี้ได้

ถ้า Action อยู่คนละ repository (public หรือ private ที่มีสิทธิ์เข้าถึง):

```yaml
      - name: Setup project (จาก repo แยก)
        uses: my-org/setup-node-project@v1
        with:
          node-version: '20'
```

รูปแบบ `owner/repo@ref` นี้คือรูปแบบเดียวกับที่เราใช้เรียก `actions/checkout@v4` มาตลอด — เพราะ Action สาธารณะทุกตัวก็คือ repository ที่มี `action.yml` อยู่ที่ root นั่นเอง

### ทดสอบ Composite Action ในเครื่อง (แนวคิด)

Composite Action ทดสอบยากกว่าฟังก์ชันปกติเพราะมันต้องรันผ่าน GitHub Actions runner จริง ๆ แนวทางที่แนะนำคือ:

1. สร้าง workflow ทดสอบเฉพาะ (เช่น `.github/workflows/test-action.yml`) ที่ trigger ด้วย `push` ไปยัง branch ทดลอง
2. ใช้เครื่องมืออย่าง [`act`](https://github.com/nektos/act) เพื่อจำลองการรัน GitHub Actions บนเครื่อง local (รองรับ Composite Action ได้ดี แต่มีข้อจำกัดกับบาง feature)
3. เขียน matrix test ให้ครอบคลุมหลาย input เช่น `node-version: [18, 20, 22]`

---

## Step 693: สร้าง JavaScript Action เบื้องต้น (`runs.using: node20`, ใช้ `@actions/core` package)

**JavaScript Action** เหมาะกับกรณีที่ logic ซับซ้อนเกินกว่าจะเขียนเป็น shell script เช่น ต้องการ parse JSON ซับซ้อน, เรียก GitHub REST API, หรือทำ retry logic ที่มีเงื่อนไขเยอะ

### แพ็กเกจสำคัญ: `@actions/core` และ `@actions/github`

GitHub ให้ SDK อย่างเป็นทางการมาช่วยเขียน JavaScript Action:

- **`@actions/core`** — จัดการ input/output, ตั้งค่า exit code, เขียน log ระดับต่าง ๆ (`info`, `warning`, `error`, `debug`)
- **`@actions/github`** — สร้าง octokit client ที่ผูก token ให้อัตโนมัติ พร้อม context ของ event ที่ trigger workflow
- **`@actions/exec`** — รันคำสั่ง shell จากใน JavaScript
- **`@actions/tool-cache`** — ดาวน์โหลดและ cache tool (ใช้ในการเขียน setup-action ต่าง ๆ)

### โครงสร้าง Repository ของ JavaScript Action

```
notify-status/
├── action.yml
├── package.json
├── package-lock.json
├── index.js
├── dist/
│   └── index.js      ← ไฟล์ที่ build แล้ว รวม dependency ทั้งหมด (bundled)
└── node_modules/      ← ไม่ควร commit (อยู่ใน .gitignore)
```

### เขียน `package.json`

```json
{
  "name": "notify-status",
  "version": "1.0.0",
  "description": "Custom Action สำหรับส่งสถานะ workflow ไปยัง webhook ภายนอก",
  "main": "index.js",
  "scripts": {
    "build": "ncc build index.js -o dist"
  },
  "dependencies": {
    "@actions/core": "^1.10.1",
    "@actions/github": "^6.0.0"
  },
  "devDependencies": {
    "@vercel/ncc": "^0.38.1"
  }
}
```

### เขียน `index.js`

```javascript
const core = require('@actions/core');
const github = require('@actions/github');

async function run() {
  try {
    // อ่าน input ที่กำหนดใน action.yml
    const webhookUrl = core.getInput('webhook-url', { required: true });
    const status = core.getInput('status', { required: true });
    const message = core.getInput('message') || '';

    // ตรวจสอบค่า status ให้ถูกต้อง
    const allowedStatuses = ['success', 'failure', 'cancelled'];
    if (!allowedStatuses.includes(status)) {
      core.setFailed(`status ต้องเป็นหนึ่งใน: ${allowedStatuses.join(', ')} แต่ได้รับ '${status}'`);
      return;
    }

    core.info(`กำลังส่งสถานะ "${status}" ไปยัง webhook...`);

    // ดึง context ของ workflow ที่กำลังรันอยู่
    const context = github.context;
    const payload = {
      status,
      message,
      repository: context.repo.repo,
      owner: context.repo.owner,
      ref: context.ref,
      sha: context.sha,
      runId: context.runId,
      actor: context.actor,
    };

    core.debug(`Payload ที่จะส่ง: ${JSON.stringify(payload)}`);

    const response = await fetch(webhookUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });

    if (!response.ok) {
      throw new Error(`Webhook ตอบกลับด้วยสถานะ HTTP ${response.status}`);
    }

    core.setOutput('sent-at', new Date().toISOString());
    core.info('ส่งสถานะสำเร็จแล้ว');
  } catch (error) {
    // setFailed จะทำให้ Action job ล้มเหลวและแสดง error message ใน log
    core.setFailed(`Action ล้มเหลว: ${error.message}`);
  }
}

run();
```

### เขียน `action.yml` สำหรับ JavaScript Action

```yaml
name: 'Notify Status'
description: 'ส่งสถานะของ workflow ไปยัง webhook ภายนอก เช่น Slack หรือระบบภายในองค์กร'
author: 'platform-team'

inputs:
  webhook-url:
    description: 'URL ปลายทางที่จะส่ง POST request ไป'
    required: true
  status:
    description: 'สถานะของงาน (success, failure, cancelled)'
    required: true
  message:
    description: 'ข้อความเพิ่มเติม'
    required: false
    default: ''

outputs:
  sent-at:
    description: 'เวลาที่ส่งสำเร็จ (ISO 8601)'

runs:
  using: 'node20'
  main: 'dist/index.js'

branding:
  icon: 'send'
  color: 'purple'
```

จุดสำคัญ: ค่า `main:` ต้องชี้ไปที่ไฟล์ **ที่ build แล้ว** (`dist/index.js`) ไม่ใช่ `index.js` ต้นฉบับโดยตรง เพราะ `index.js` ต้นฉบับยัง `require()` แพ็กเกจจาก `node_modules` ซึ่งปกติแล้วเราจะไม่ commit `node_modules` ขึ้น repository (ไฟล์เยอะเกินไปและมีปัญหาเรื่อง cross-platform)

### ทำไมต้อง Bundle ด้วย `ncc`

เนื่องจาก GitHub Actions runner จะดึงแค่ไฟล์ที่ระบุใน `main:` ไปรันตรง ๆ โดยไม่รัน `npm install` ให้ก่อน ถ้าเราไม่ bundle dependency เข้าไปในไฟล์เดียว การรันจะ error ทันทีเพราะหา module ที่ `require()` ไม่เจอ

`@vercel/ncc` คือเครื่องมือที่ JavaScript Action ส่วนใหญ่ในระบบนิเวศ GitHub Actions ใช้กัน เพื่อ **compile ทั้งโปรเจกต์ + dependency ทั้งหมดให้เหลือเป็นไฟล์ JavaScript เดียว**

คำสั่งที่ใช้ build:

```bash
npm install
npm run build
```

หลัง build เสร็จ ต้อง **commit โฟลเดอร์ `dist/` เข้า repository ด้วย** (แม้ว่าปกติเราจะไม่ commit build artifact ก็ตาม — กรณีนี้เป็นข้อยกเว้นเพราะ GitHub Actions ต้องเห็นไฟล์ที่ build แล้วโดยตรงเมื่อมีคนเรียกใช้ Action ของเรา)

### เพิ่ม CI ให้ตรวจสอบว่า `dist/` sync กับ source เสมอ

ปัญหาที่พบบ่อยที่สุดคือคนแก้ `index.js` แต่ลืม build ใหม่ ทำให้ `dist/index.js` เก่าค้างอยู่ เราจึงควรเพิ่ม workflow ตรวจสอบ:

```yaml
name: Check dist is up to date

on:
  pull_request:
    branches: [main]

jobs:
  check-dist:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm ci

      - run: npm run build

      - name: ตรวจว่า dist/ เปลี่ยนไปหรือไม่หลัง build
        run: |
          if [ -n "$(git status --porcelain dist/)" ]; then
            echo "::error::โฟลเดอร์ dist/ ไม่ตรงกับ source code กรุณารัน npm run build แล้ว commit ใหม่"
            git diff dist/
            exit 1
          fi
```

Pattern นี้เป็น pattern มาตรฐานที่ Action ยอดนิยมบน Marketplace เกือบทุกตัวใช้กัน

---

## Step 694: สร้าง Docker Action เบื้องต้น (`runs.using: docker`, `Dockerfile`)

**Docker Action** ให้อิสระสูงสุดเพราะเรากำหนด environment เองได้ทั้งหมดผ่าน `Dockerfile` — ใช้ภาษาอะไรก็ได้ (Python, Go, Rust, หรือแม้แต่ shell script ล้วน ๆ) ไม่จำกัดอยู่แค่ Node.js เหมือน JavaScript Action

### เมื่อไหร่ควรเลือก Docker Action

- ต้องการ dependency ระบบ (system-level) ที่ติดตั้งยากหรือใช้เวลานานถ้าติดตั้งทุกครั้งบน runner ปกติ
- อยากเขียน logic ด้วยภาษาที่ไม่ใช่ JavaScript เช่น Python พร้อม library เฉพาะทาง
- ต้องการ environment ที่ reproducible แบบ 100% ไม่ว่าจะรันบน runner ไหนก็ได้ผลลัพธ์เหมือนกันเป๊ะ

### ข้อจำกัดที่ต้องรู้ก่อนใช้

- **Docker Action รันได้เฉพาะบน Linux runner เท่านั้น** (ไม่รองรับ Windows หรือ macOS runner)
- ทุกครั้งที่รัน (ถ้าไม่ pull image สำเร็จรูปจาก registry) runner จะต้อง **build image ใหม่ทุกครั้ง** ซึ่งทำให้ job ช้ากว่าแบบอื่นชัดเจน — วิธีแก้คือ publish image ขึ้น registry ล่วงหน้าแล้วใช้ `image:` แบบ pre-built แทนการ build จาก `Dockerfile` สด ๆ ทุกครั้ง

### โครงสร้าง Repository ของ Docker Action

```
lint-commit-message/
├── action.yml
├── Dockerfile
├── entrypoint.sh
└── README.md
```

### เขียน `Dockerfile`

```dockerfile
FROM python:3.12-slim

# ติดตั้ง dependency ที่จำเป็นสำหรับตรวจสอบ commit message
RUN pip install --no-cache-dir gitlint==0.19.0

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```

### เขียน `entrypoint.sh`

```bash
#!/bin/bash
set -euo pipefail

# input ที่ GitHub Actions ส่งเข้ามาจะอยู่ในรูปแบบ environment variable
# ชื่อ INPUT_<ชื่อ-input-ตัวพิมพ์ใหญ่-ขีดกลางเป็นขีดล่าง>
COMMIT_RANGE="${INPUT_COMMIT-RANGE:-HEAD~1..HEAD}"

echo "กำลังตรวจสอบ commit message ในช่วง: $COMMIT_RANGE"

if gitlint --commits "$COMMIT_RANGE"; then
  echo "commit message ผ่านมาตรฐานทั้งหมด"
  echo "result=passed" >> "$GITHUB_OUTPUT"
else
  echo "::error::พบ commit message ที่ไม่ตรงตามมาตรฐาน"
  echo "result=failed" >> "$GITHUB_OUTPUT"
  exit 1
fi
```

หมายเหตุสำคัญ: GitHub Actions สร้าง environment variable ของ input โดย **แปลงตัวอักษรเป็นตัวพิมพ์ใหญ่และแทนที่ space ด้วย `_` เท่านั้น — แต่ไม่ได้แปลงเครื่องหมายขีดกลาง (`-`) ให้เป็นขีดล่าง (`_`) ให้** ดังนั้น input ชื่อ `commit-range` จะถูกสร้างเป็น environment variable ชื่อ **`INPUT_COMMIT-RANGE`** (มีขีดกลางติดอยู่ตรง ๆ) ปัญหาคือ bash **ไม่อนุญาตให้ชื่อตัวแปรมีขีดกลาง** ทำให้ไม่สามารถเขียน `$INPUT_COMMIT-RANGE` แล้วให้ bash ตีความว่าเป็นตัวแปรเดียวได้ (bash จะอ่านเป็น `$INPUT_COMMIT` ต่อด้วยข้อความ `-RANGE` เฉย ๆ) โค้ด `entrypoint.sh` ตัวแรกด้านบนจึงเป็นตัวอย่างของ**ข้อผิดพลาดที่พบบ่อย** วิธีที่ปลอดภัยและแนะนำที่สุดคือ **หลีกเลี่ยงขีดกลางในชื่อ input ของ Docker Action ตั้งแต่ต้น** แล้วใช้ขีดล่างแทน เช่น `commit_range` เพื่อให้ได้ environment variable ที่ชื่อ `INPUT_COMMIT_RANGE` ซึ่งเป็นชื่อตัวแปรที่ bash อ่านได้ถูกต้องตรงไปตรงมา

แก้ไข `entrypoint.sh` ให้ถูกต้องและปลอดภัยกว่าเดิม:

```bash
#!/bin/bash
set -euo pipefail

COMMIT_RANGE="${INPUT_COMMIT_RANGE:-HEAD~1..HEAD}"

echo "กำลังตรวจสอบ commit message ในช่วง: $COMMIT_RANGE"

if gitlint --commits "$COMMIT_RANGE"; then
  echo "commit message ผ่านมาตรฐานทั้งหมด"
  echo "result=passed" >> "$GITHUB_OUTPUT"
else
  echo "::error::พบ commit message ที่ไม่ตรงตามมาตรฐาน"
  echo "result=failed" >> "$GITHUB_OUTPUT"
  exit 1
fi
```

และเปลี่ยนชื่อ input ใน `action.yml` เป็น `commit_range` ให้ตรงกัน

### เขียน `action.yml` สำหรับ Docker Action

```yaml
name: 'Lint Commit Message'
description: 'ตรวจสอบว่า commit message เป็นไปตามมาตรฐาน Conventional Commits หรือไม่'
author: 'platform-team'

inputs:
  commit_range:
    description: 'ช่วงของ commit ที่ต้องการตรวจสอบ (git range syntax)'
    required: false
    default: 'HEAD~1..HEAD'

outputs:
  result:
    description: 'ผลการตรวจสอบ (passed หรือ failed)'

runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.commit_range }}

branding:
  icon: 'check-circle'
  color: 'green'
```

ข้อสังเกตสำคัญ:

1. `image: 'Dockerfile'` บอกให้ runner **build image จาก Dockerfile ในโฟลเดอร์เดียวกัน** ทุกครั้งที่ Action ถูกเรียกใช้
2. อีกทางเลือกคือ `image: 'docker://ghcr.io/my-org/lint-commit-message:1.0.0'` เพื่อใช้ **image ที่ build และ push ขึ้น registry ไว้ล่วงหน้าแล้ว** วิธีนี้เร็วกว่ามากเพราะไม่ต้อง build ใหม่ทุกครั้ง
3. `args:` คือค่าที่จะถูกส่งเป็น command-line arguments ให้กับ `ENTRYPOINT` ของ container — แต่ในตัวอย่างข้างต้นเราเลือกอ่านผ่าน environment variable (`INPUT_COMMIT_RANGE`) แทน ซึ่งทั้งสองวิธีใช้ได้ ขึ้นอยู่กับความสะดวกในการเขียนสคริปต์

### ใช้ pre-built image เพื่อความเร็ว (แนะนำสำหรับ production)

```yaml
runs:
  using: 'docker'
  image: 'docker://ghcr.io/my-org/lint-commit-message:1.0.0'
  args:
    - ${{ inputs.commit_range }}
```

การ build image ล่วงหน้าและ push ขึ้น GitHub Container Registry (`ghcr.io`) ทำได้ด้วย workflow แยกต่างหาก ซึ่งเราเคยเรียนวิธี build/push Docker image ไปแล้วใน Part ก่อนหน้าของเฟส 7

### เปรียบเทียบความเร็วโดยประมาณ

| วิธี | ต้อง build ทุกครั้งไหม | เวลาเฉลี่ยที่เพิ่มขึ้น |
|---|---|---|
| `image: 'Dockerfile'` (build สด) | ใช่ | 20-90 วินาที ขึ้นกับขนาด image |
| `image: 'docker://...'` (pre-built) | ไม่ (แค่ pull) | 2-10 วินาที (หรือเร็วกว่าถ้า layer cache ไว้) |

---

## Step 695: การ publish Action ขึ้น GitHub Marketplace (ต้องมี `action.yml` ที่ root และ tag version)

หลังจากสร้าง Custom Action เสร็จแล้ว ถ้าต้องการแบ่งปันให้คนอื่นในองค์กรหรือทั้งโลกใช้งาน เราสามารถ **publish ขึ้น GitHub Marketplace** ได้

### ข้อกำหนดพื้นฐานก่อน publish

1. **Repository ต้องเป็น public** — Marketplace ไม่รองรับ Action จาก private repository (แต่ private repository ยังคงเรียกใช้ Action ภายในองค์กรตัวเองได้ปกติโดยไม่ต้อง publish)
2. **ต้องมี `action.yml` (หรือ `action.yaml`) อยู่ที่ root ของ repository** — ถ้า repository เก็บ Action หลายตัว (monorepo of actions) จะไม่สามารถ publish หลายตัวจาก repository เดียวขึ้น Marketplace ได้ตรง ๆ ต้องแยก repository ต่อ 1 Action ที่จะ publish
3. **ต้องมี README.md อธิบายวิธีใช้งาน** — เป็น best practice ที่ Marketplace บังคับให้มีก่อน publish
4. **ต้อง tag เป็น semantic version** เช่น `v1.0.0`, `v1.1.0`, `v2.0.0`

### ขั้นตอนการ publish

#### 1. เตรียม README.md ที่ดี

README ควรมีอย่างน้อย:

```markdown
# Setup Node Project

Composite Action สำหรับ checkout, setup Node.js และติดตั้ง dependency พร้อม cache

## การใช้งาน

```yaml
- uses: my-org/setup-node-project@v1
  with:
    node-version: '20'
    working-directory: '.'
```

## Inputs

| ชื่อ | จำเป็น | ค่า default | คำอธิบาย |
|---|---|---|---|
| `node-version` | ไม่ | `20` | Node.js version ที่จะใช้ |
| `working-directory` | ไม่ | `.` | โฟลเดอร์ที่มี package.json |

## Outputs

| ชื่อ | คำอธิบาย |
|---|---|
| `cache-hit` | บอกว่า cache ของ node_modules ถูก hit หรือไม่ |
```

#### 2. Commit และ push โค้ดทั้งหมดขึ้น branch หลัก

```bash
git add action.yml README.md
git commit -m "feat: initial release ของ setup-node-project action"
git push origin main
```

#### 3. สร้าง Git tag ตาม Semantic Versioning

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

#### 4. สร้าง Release บน GitHub

ไปที่หน้า repository → แท็บ **Releases** → **Draft a new release** → เลือก tag `v1.0.0` ที่เพิ่งสร้าง → กรอกชื่อและ changelog → จะมี checkbox **"Publish this Action to the GitHub Marketplace"** ปรากฏขึ้นมาโดยอัตโนมัติ (ถ้า repository มี `action.yml` ที่ถูกต้อง) → เลือก category ที่เหมาะสม (เช่น "Continuous Integration") → กด **Publish release**

### การจัดการ Major Version Tag (แนวปฏิบัติมาตรฐานของวงการ)

สังเกตว่า Action ยอดนิยมอย่าง `actions/checkout@v4` ให้ผู้ใช้อ้างอิงแค่ `@v4` ไม่ต้องระบุ `@v4.1.7` แบบเต็ม นี่คือ pattern ที่เรียกว่า **"moving major version tag"**

วิธีทำ:

```bash
# หลังจาก release v1.0.0 แล้ว ให้สร้าง/ย้าย tag v1 ให้ชี้ไปที่ commit เดียวกับ v1.0.0
git tag -fa v1 -m "อัปเดต v1 ให้ชี้ไปที่ v1.0.0"
git push origin v1 --force
```

เมื่อ release เวอร์ชันย่อยใหม่ (เช่น `v1.1.0`) ที่ **ไม่ทำลาย backward compatibility** ให้ทำซ้ำ:

```bash
git tag -a v1.1.0 -m "Release v1.1.0"
git push origin v1.1.0

git tag -fa v1 -m "อัปเดต v1 ให้ชี้ไปที่ v1.1.0"
git push origin v1 --force
```

แต่ถ้า release เวอร์ชันที่ **มี breaking change** (เช่น `v2.0.0`) ให้สร้าง tag ใหม่ `v2` แยกต่างหาก และคง `v1` ไว้ชี้ที่ของเดิม เพื่อไม่ให้ผู้ใช้ที่ pin `@v1` โดนกระทบโดยไม่รู้ตัว

> **คำเตือนสำคัญ:** การ force-push tag แบบนี้เป็นสิ่งที่ยอมรับได้เฉพาะกับ major version tag (`v1`, `v2`) เท่านั้น **ห้าม force-push tag ที่เป็น full version (`v1.0.0`) เด็ดขาด** เพราะผู้ใช้ที่ pin เวอร์ชันเต็มไว้เพื่อความปลอดภัย (reproducibility) คาดหวังว่า tag นั้นจะไม่เปลี่ยนแปลงตลอดไป การเปลี่ยนแปลงจะทำลายความน่าเชื่อถือทันที

### ตารางสรุปวิธี pin เวอร์ชันของผู้ใช้ Action

| วิธี pin | ตัวอย่าง | ข้อดี | ข้อเสีย |
|---|---|---|---|
| Major version tag | `@v1` | ได้ patch/minor update อัตโนมัติ สะดวก | เสี่ยงถ้า maintainer ทำ breaking change โดยไม่ตั้งใจ |
| Full version tag | `@v1.2.3` | เสถียรที่สุด reproducible 100% | ต้องอัปเดตเองทุกครั้งที่มี fix ใหม่ |
| Commit SHA | `@a1b2c3d...` | ปลอดภัยสูงสุด (supply chain security) ป้องกัน tag ถูกแก้ย้อนหลัง | อ่านยาก ต้องอัปเดตเองเสมอ |
| Branch name | `@main` | เห็นความเปลี่ยนแปลงล่าสุดเสมอ | **ไม่แนะนำสำหรับ production เด็ดขาด** เพราะไม่เสถียรและเสี่ยงต่อการโดนแก้ไขโค้ดโดยไม่รู้ตัว |

องค์กรที่ให้ความสำคัญกับความปลอดภัยสูง (เช่นผ่านการตรวจสอบ supply chain security) มักบังคับให้ pin ด้วย commit SHA เท่านั้น และใช้เครื่องมืออย่าง Dependabot คอยอัปเดต SHA ให้อัตโนมัติเมื่อมีเวอร์ชันใหม่

---

## Step 696: Reusable Workflows คืออะไร (`on: workflow_call`)

นอกจาก Custom Action แล้ว GitHub ยังมีกลไกอีกแบบหนึ่งที่ใช้แก้ปัญหาความซ้ำซ้อนได้เช่นกัน นั่นคือ **Reusable Workflows**

### ความแตกต่างพื้นฐานระหว่าง Action กับ Reusable Workflow

- **Custom Action** คือการห่อ **step ระดับเดียว** ให้เรียกใช้ซ้ำได้ (ใช้แทนที่ step หนึ่งใน job)
- **Reusable Workflow** คือการห่อ **job ทั้งหมด (หรือหลาย job)** ให้เรียกใช้ซ้ำได้ (ใช้แทนที่ job ทั้งก้อน)

พูดง่าย ๆ: Reusable Workflow ทำงานในระดับที่ **ใหญ่กว่า** Custom Action มาก

### Trigger พิเศษ: `workflow_call`

Workflow ที่จะถูกเรียกใช้จาก workflow อื่นได้ ต้องมี trigger พิเศษชื่อ `workflow_call` อยู่ใน `on:`

```yaml
name: Reusable CI Workflow

on:
  workflow_call:
    inputs:
      node-version:
        description: 'Node.js version ที่จะใช้'
        type: string
        default: '20'
      run-e2e:
        description: 'รัน end-to-end test ด้วยหรือไม่'
        type: boolean
        default: false
    secrets:
      npm-token:
        description: 'Token สำหรับเข้าถึง private npm registry'
        required: false
    outputs:
      build-version:
        description: 'เวอร์ชันที่ build ออกมา'
        value: ${{ jobs.build.outputs.version }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.get-version.outputs.version }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          registry-url: 'https://registry.npmjs.org'

      - name: Install
        run: npm ci
        env:
          NODE_AUTH_TOKEN: ${{ secrets.npm-token }}

      - name: Build
        run: npm run build

      - name: Get version
        id: get-version
        run: echo "version=$(node -p "require('./package.json').version")" >> "$GITHUB_OUTPUT"

      - name: Run unit tests
        run: npm test

  e2e:
    needs: build
    if: inputs.run-e2e == true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm run test:e2e
```

### จุดสำคัญที่ต้องเข้าใจให้ชัดเจน

1. **`inputs:` ใน `workflow_call` ต้องระบุ `type:` เสมอ** (`string`, `boolean`, `number`) — ต่างจาก Custom Action ที่ input ทุกตัวเป็น string หมด
2. **`secrets:` ต้องประกาศไว้ล่วงหน้าอย่างชัดเจน** — reusable workflow **ไม่ได้รับ secret ทั้งหมดของ caller โดยอัตโนมัติ** ต้องส่งผ่านมาทีละตัวเท่านั้น (นี่คือฟีเจอร์ด้านความปลอดภัยที่สำคัญมาก)
3. **`outputs:` ระดับ workflow ต้องดึงมาจาก `outputs` ของ job ภายใน** ผ่าน syntax `${{ jobs.<job_id>.outputs.<output_name> }}`
4. Reusable Workflow สามารถมีได้หลาย job และ job เหล่านั้นสามารถมี `needs:` เชื่อมกันได้ตามปกติเหมือน workflow ทั่วไป
5. reusable workflow **เรียกซ้อนกันได้สูงสุด 10 ระดับ** (workflow ผู้เรียกระดับบนสุด บวก reusable workflow อีกไม่เกิน 9 ชั้นถัดไป) และในหนึ่ง workflow เรียกใช้ reusable workflow ที่ไม่ซ้ำกันได้รวมสูงสุด 50 workflow ทั้ง tree (ตัวเลขนี้เป็นค่าปัจจุบันบน GitHub.com — เอกสารเก่าหรือบาง GitHub Enterprise Server เวอร์ชันเก่าอาจยังระบุ 4 ระดับ/20 workflow อยู่ ควรตรวจสอบเอกสารทางการล่าสุดตามเวอร์ชันที่ใช้งานจริงเสมอ)

### สิ่งที่ reusable workflow "มองไม่เห็น" จาก caller

- **Environment variables ที่ตั้งไว้ระดับ `env:` ของ workflow ผู้เรียก** จะไม่ถูกส่งต่อเข้าไปอัตโนมัติ ต้องส่งผ่าน `inputs:` เท่านั้น
- **`GITHUB_TOKEN` ของ default** จะถูกสร้างใหม่เป็นของตัวเอง (ไม่ใช้ token เดียวกับ caller) โดย permission ของมันจะถูกกำหนดจาก `permissions:` ที่ตั้งไว้ในตัว reusable workflow เอง หรือรับ scope ที่แคบกว่าระหว่าง caller กับ callee

---

## Step 697: การเรียกใช้ reusable workflow จาก workflow อื่นด้วย `uses:` พร้อมส่ง `inputs`/`secrets`

เมื่อมี Reusable Workflow พร้อมใช้งานแล้ว (สมมติชื่อไฟล์ `.github/workflows/reusable-ci.yml`) เราเรียกใช้จาก workflow อื่นได้โดยใช้ `uses:` **ที่ระดับ job** (ไม่ใช่ระดับ step เหมือน Action)

### เรียกใช้จาก workflow ในไฟล์เดียวกัน repository

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  call-reusable-ci:
    uses: ./.github/workflows/reusable-ci.yml
    with:
      node-version: '20'
      run-e2e: true
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}
```

### เรียกใช้จาก repository อื่น

```yaml
jobs:
  call-reusable-ci:
    uses: my-org/shared-workflows/.github/workflows/reusable-ci.yml@v1
    with:
      node-version: '20'
      run-e2e: false
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}
```

รูปแบบคือ `owner/repo/.github/workflows/ชื่อไฟล์.yml@ref` — สังเกตว่า reusable workflow **ต้องเก็บไว้ในโฟลเดอร์ `.github/workflows/` เท่านั้น** จะเก็บไว้ที่ path อื่นเหมือน Custom Action ไม่ได้

### ส่ง secrets ทั้งหมดแบบสั้น ๆ ด้วย `secrets: inherit`

ถ้า reusable workflow อยู่ใน **repository เดียวกันในองค์กรเดียวกัน** และเราต้องการส่ง secret ทุกตัวที่ caller มีไปให้ทั้งหมดโดยไม่ต้องระบุทีละตัว สามารถใช้:

```yaml
jobs:
  call-reusable-ci:
    uses: my-org/shared-workflows/.github/workflows/reusable-ci.yml@v1
    with:
      node-version: '20'
    secrets: inherit
```

`secrets: inherit` สะดวกมากแต่ควรใช้อย่างระมัดระวัง เพราะมันทำให้ reusable workflow เข้าถึง secret **ทุกตัว** ของ caller ได้ทั้งหมด ซึ่งอาจขัดกับหลัก **Principle of Least Privilege** ถ้า reusable workflow นั้นมาจาก repository ที่ไม่ได้อยู่ในการควบคุมโดยตรงของทีมเรา 100%

### เรียกใช้ reusable workflow พร้อมกันหลายชุดด้วย matrix

จุดที่มีประโยชน์มากคือเราสามารถใช้ `strategy.matrix` ร่วมกับการเรียก reusable workflow ได้ เช่น รัน CI เดียวกันกับหลายเวอร์ชันของ Node.js:

```yaml
jobs:
  call-reusable-ci:
    strategy:
      matrix:
        node-version: ['18', '20', '22']
    uses: ./.github/workflows/reusable-ci.yml
    with:
      node-version: ${{ matrix.node-version }}
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}
```

### ใช้ output จาก reusable workflow ต่อใน job อื่น

```yaml
jobs:
  call-reusable-ci:
    uses: ./.github/workflows/reusable-ci.yml
    with:
      node-version: '20'
    secrets:
      npm-token: ${{ secrets.NPM_TOKEN }}

  deploy:
    needs: call-reusable-ci
    runs-on: ubuntu-latest
    steps:
      - name: แสดงเวอร์ชันที่ build ได้
        run: echo "กำลัง deploy เวอร์ชัน ${{ needs.call-reusable-ci.outputs.build-version }}"
```

สังเกตว่าเราอ้างอิง output ของ job ที่เรียก reusable workflow ผ่าน `needs.<job_id>.outputs.<output_name>` เหมือนกับ job ปกติทุกประการ — reusable workflow ที่ถูกเรียกจะถูกมองเป็น "job หนึ่ง" ในสายตาของ workflow ผู้เรียกเสมอ

### ข้อจำกัดที่ต้องระวัง

1. `uses:` สำหรับเรียก reusable workflow **ต้องอยู่ตรงกับ job โดยตรง** จะผสมกับ `steps:` ใน job เดียวกันไม่ได้ — job ที่เรียก reusable workflow จะมีแค่ `uses:`, `with:`, `secrets:`, `strategy:`, `needs:`, `if:` เท่านั้น ใส่ `runs-on:` หรือ `steps:` เพิ่มเข้าไปไม่ได้
2. Reusable workflow ที่เรียกจาก private repository อื่น ผู้เรียกต้องมีสิทธิ์เข้าถึง repository นั้น และถ้าอยู่คนละ organization ต้องเปิด setting ให้ organization นั้นอนุญาตให้ repository ภายนอกเรียกใช้ workflow ของตนได้ก่อน (ตั้งค่าที่ Organization Settings → Actions → General)

---

## Step 698: เปรียบเทียบ Composite Action vs Reusable Workflow เมื่อไหร่ควรใช้แบบไหน

นี่คือคำถามที่พบบ่อยที่สุดเมื่อเริ่มออกแบบระบบ CI/CD ที่ต้องใช้ซ้ำในหลาย repository — **ควรห่อเป็น Composite Action หรือ Reusable Workflow ดี?**

### ตารางเปรียบเทียบละเอียด

| คุณสมบัติ | Composite Action | Reusable Workflow |
|---|---|---|
| ระดับที่ครอบคลุม | ระดับ **step** (ส่วนหนึ่งของ job) | ระดับ **job เต็ม ๆ** (หรือหลาย job) |
| เรียกใช้ตรงไหน | `steps:` ภายใน job | `jobs.<id>.uses:` ที่ระดับ job |
| กำหนด `runs-on:` เองได้ไหม | ไม่ได้ (ใช้ runner เดียวกับ job ที่เรียกมัน) | ได้ (แต่ละ job ใน reusable workflow กำหนด `runs-on:` ของตัวเอง) |
| ใช้ `needs:` ภายในตัวมันเองได้ไหม | ไม่ได้ (เป็นแค่ step เรียงลำดับ) | ได้ (มีหลาย job เชื่อมกันด้วย `needs:` ได้เต็มรูปแบบ) |
| รับ secrets อย่างไร | รับผ่าน `env:` ของ workflow ผู้เรียกโดยอัตโนมัติ (ถ้า workflow ผู้เรียก pass เข้ามาทาง input หรือ env) | ต้องประกาศ `secrets:` อย่างชัดเจน หรือใช้ `secrets: inherit` |
| ผสม job ที่รันบน runner คนละชนิดได้ไหม | ไม่ได้ (ต้องรันบน runner เดียวกับ job แม่) | ได้ (แต่ละ job เลือก runner ของตัวเองอิสระ) |
| ความเร็วในการเรียก | เร็วมาก (ไม่มี overhead พิเศษ) | มี overhead เล็กน้อยจากการ spawn job ใหม่ |
| เหมาะกับงานขนาด | เล็ก–กลาง (เช่น setup environment, notify, lint) | กลาง–ใหญ่ (เช่น pipeline ทั้งชุด build+test+deploy) |
| Publish ขึ้น Marketplace ได้ไหม | ได้ | **ไม่ได้** (Marketplace รองรับเฉพาะ Action ไม่รองรับ reusable workflow) |
| เขียนด้วยภาษาอะไรได้บ้าง | YAML + shell (หรือเรียก JS/Docker action ซ้อนได้) | YAML ล้วน (แต่ job ข้างในเรียก action ภาษาไหนก็ได้) |

### หลักการตัดสินใจง่าย ๆ

ใช้คำถามเหล่านี้ช่วยตัดสินใจ:

1. **สิ่งที่จะห่อ คือ "ขั้นตอนย่อยในงานเดียว" หรือ "งานทั้งชุด"?**
   - ขั้นตอนย่อย (เช่น "setup + cache dependency") → **Composite Action**
   - งานทั้งชุด (เช่น "build → test → security scan → deploy") → **Reusable Workflow**

2. **ต้องการรัน job หลายตัวพร้อมกัน หรือ job ที่รันบน runner คนละชนิดกันไหม?**
   - ต้องการ → **Reusable Workflow** (เพราะมันมี job หลายตัวในตัวเองได้)
   - ไม่ต้องการ (รันใน job เดียวพอ) → **Composite Action** ก็เพียงพอ

3. **ต้องการเผยแพร่ให้คนทั่วโลกใช้ผ่าน Marketplace ไหม?**
   - ต้องการ → ต้องเป็น **Custom Action** (Composite, JS หรือ Docker) เพราะ Marketplace ไม่รองรับ reusable workflow

4. **ต้องการ logic ที่ซับซ้อนระดับเขียนโปรแกรมเต็มรูปแบบ (เช่น เรียก API หลายชั้น, error handling ซับซ้อน)?**
   - ต้องการ → **JavaScript Action** หรือ **Docker Action**
   - ไม่ต้องการ (แค่รวม step ที่มีอยู่แล้ว) → **Composite Action**

### สถานการณ์จริงที่มักผสมทั้งสองแบบเข้าด้วยกัน

ในทางปฏิบัติ องค์กรขนาดใหญ่มักใช้ **ทั้งสองแบบร่วมกัน** โดย:

- ใช้ **Composite Action** ห่อ step เล็ก ๆ ที่ใช้ซ้ำบ่อยมาก เช่น setup environment, notify Slack, upload artifact แบบมาตรฐาน
- ใช้ **Reusable Workflow** ห่อ pipeline ทั้งชุดที่ประกอบด้วยหลาย job เช่น "standard-microservice-pipeline.yml" ที่รวม build, test, scan, deploy เข้าด้วยกัน
- ภายใน Reusable Workflow นั้นเอง ก็เรียกใช้ Composite Action ต่าง ๆ ที่สร้างไว้อีกที

```
Reusable Workflow: standard-microservice-pipeline.yml
├── job: build
│   ├── uses: my-org/setup-node-project@v1   (Composite Action)
│   └── run: npm run build
├── job: test
│   ├── uses: my-org/setup-node-project@v1   (Composite Action)
│   └── run: npm test
├── job: security-scan
│   └── uses: my-org/scan-vulnerabilities@v2 (Docker Action)
└── job: notify
    └── uses: my-org/notify-status@v1        (JavaScript Action)
```

นี่คือสถาปัตยกรรมที่พบได้บ่อยที่สุดในองค์กรที่มี repository จำนวนมากและต้องการมาตรฐาน CI/CD ที่สม่ำเสมอ

---

## Step 699: Best Practices การจัดระเบียบ workflow ในองค์กรที่มีหลาย repository

เมื่อองค์กรมี repository เป็นสิบเป็นร้อย การจัดระเบียบ Custom Action และ Reusable Workflow ให้ดีตั้งแต่ต้นเป็นสิ่งที่สำคัญมาก มิเช่นนั้นจะเกิดปัญหา "ทุก repository มี workflow ที่แตกต่างกันเล็กน้อยจนดูแลไม่ไหว"

### แนวคิดหลัก: Central Actions Repository

สร้าง **repository กลาง** เพียงที่เดียว (เช่น `my-org/shared-workflows` หรือ `my-org/actions-library`) เพื่อเก็บ Custom Action และ Reusable Workflow ทั้งหมดที่ใช้ร่วมกันทั่วทั้งองค์กร แทนที่จะกระจัดกระจายอยู่คนละ repository

### โครงสร้างที่แนะนำของ Central Repository

```
shared-workflows/
├── .github/
│   └── workflows/
│       ├── reusable-node-ci.yml
│       ├── reusable-docker-build-push.yml
│       ├── reusable-security-scan.yml
│       └── reusable-deploy-k8s.yml
├── actions/
│   ├── setup-node-project/
│   │   ├── action.yml
│   │   └── README.md
│   ├── notify-status/
│   │   ├── action.yml
│   │   ├── package.json
│   │   ├── index.js
│   │   └── dist/index.js
│   └── lint-commit-message/
│       ├── action.yml
│       ├── Dockerfile
│       └── entrypoint.sh
├── docs/
│   └── usage-guide.md
└── README.md
```

ข้อควรระวัง: ถ้าต้องการ publish Action ตัวใดตัวหนึ่งขึ้น **GitHub Marketplace แบบสาธารณะ** จะต้องแยก Action นั้นออกเป็น repository ของตัวเองต่างหาก (เพราะ Marketplace บังคับให้ `action.yml` อยู่ที่ root) ส่วน Action ที่ใช้ **เฉพาะภายในองค์กร** เก็บรวมกันในโครงสร้างแบบนี้ได้สบาย เพราะการเรียกใช้แบบ `uses: my-org/shared-workflows/actions/setup-node-project@v1` ทำได้ตามปกติแม้ `action.yml` จะไม่ได้อยู่ที่ root

### หลักปฏิบัติที่ 1: กำหนดมาตรฐานการตั้งชื่อ (Naming Convention)

```
reusable-<ภาษา/แพลตฟอร์ม>-<ประเภทงาน>.yml
```

ตัวอย่าง: `reusable-node-ci.yml`, `reusable-python-ci.yml`, `reusable-docker-build-push.yml`, `reusable-terraform-plan-apply.yml`

การตั้งชื่อที่สม่ำเสมอช่วยให้ทีมค้นหา workflow ที่ต้องการได้เร็วโดยไม่ต้องเปิดอ่านทุกไฟล์

### หลักปฏิบัติที่ 2: ควบคุมเวอร์ชันอย่างเคร่งครัดด้วย Semantic Versioning

ทุกครั้งที่แก้ไข Reusable Workflow หรือ Custom Action ในองค์กร ให้ยึดหลัก Semantic Versioning เดียวกับ Step 695:

- **PATCH** (`v1.0.1`) — แก้บั๊กเล็กน้อย ไม่กระทบ interface
- **MINOR** (`v1.1.0`) — เพิ่ม input/output ใหม่แบบ backward compatible
- **MAJOR** (`v2.0.0`) — เปลี่ยนแปลงที่ทำลาย backward compatibility เช่น ลบ input ที่เคยมี หรือเปลี่ยนพฤติกรรม default

และคง moving major tag (`v1`, `v2`) ไว้เสมอเพื่อความสะดวกของทีมที่เรียกใช้

### หลักปฏิบัติที่ 3: เขียนเอกสารและตัวอย่างการใช้งานให้ครบทุกตัว

แต่ละ Action/Workflow ควรมี README ที่บอก:

- Input/Output ทั้งหมดพร้อมชนิดข้อมูลและค่า default
- ตัวอย่าง YAML การเรียกใช้แบบ copy-paste ได้ทันที
- Changelog หรือลิงก์ไปหน้า Releases

### หลักปฏิบัติที่ 4: มี CI ทดสอบ Action/Workflow ของตัวเอง (Dogfooding)

Central repository ควรมี workflow ที่ทดสอบ Action ของตัวเองก่อนปล่อยให้ทีมอื่นใช้ เช่น:

```yaml
name: Test setup-node-project action

on:
  pull_request:
    paths:
      - 'actions/setup-node-project/**'

jobs:
  test-composite-action:
    strategy:
      matrix:
        node-version: ['18', '20', '22']
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4

      - name: ทดสอบเรียกใช้ action
        uses: ./actions/setup-node-project
        with:
          node-version: ${{ matrix.node-version }}
```

การใช้ `paths:` filter ทำให้ workflow นี้รันเฉพาะเมื่อมีคนแก้ไข Action ที่เกี่ยวข้องจริง ๆ ไม่รันทุกครั้งที่มีการเปลี่ยนแปลงไฟล์อื่นในโปรเจกต์

### หลักปฏิบัติที่ 5: จำกัดสิทธิ์ (Least Privilege) ในทุก workflow

ตั้ง `permissions:` ให้แคบที่สุดเท่าที่จำเป็นเสมอ ทั้งใน Reusable Workflow และใน workflow ที่เรียกใช้มัน:

```yaml
permissions:
  contents: read
  # เพิ่มเฉพาะที่จำเป็นจริง ๆ เท่านั้น เช่น
  # pull-requests: write   ← ถ้าต้อง comment บน PR
  # packages: write        ← ถ้าต้อง push image ขึ้น registry
```

หลีกเลี่ยงการใช้ `secrets: inherit` แบบพร่ำเพรื่อ โดยเฉพาะเมื่อ workflow ที่ถูกเรียกไม่ได้อยู่ภายใต้การควบคุมของทีมเดียวกัน 100%

### หลักปฏิบัติที่ 6: ใช้ CODEOWNERS ควบคุมการเปลี่ยนแปลง Central Repository

เนื่องจาก repository นี้ส่งผลกระทบต่อทุก repository ในองค์กร การเปลี่ยนแปลงใด ๆ ควรผ่านการรีวิวจากทีม platform/DevOps โดยเฉพาะ ตั้งค่าไฟล์ `.github/CODEOWNERS`:

```
/actions/          @my-org/platform-team
/.github/workflows/ @my-org/platform-team
```

### หลักปฏิบัติที่ 7: สื่อสารการเปลี่ยนแปลง Breaking Change ให้ทั่วถึง

เมื่อจะออก major version ใหม่ที่มี breaking change ควร:

1. เขียน migration guide ชัดเจนใน CHANGELOG
2. คงเวอร์ชันเก่า (`v1`) ให้ใช้งานต่อไปได้อีกระยะหนึ่ง (deprecation period) ไม่ลบทันที
3. แจ้งทีมที่เกี่ยวข้องผ่านช่องทางภายในองค์กร (เช่น Slack channel กลางของ platform team) ก่อน merge จริง

### สรุปภาพรวมสถาปัตยกรรมที่แนะนำ

```
Organization: my-org
│
├── shared-workflows (central repo)
│   ├── actions/ (Composite/JS/Docker Actions ใช้ภายใน)
│   └── .github/workflows/ (Reusable Workflows)
│
├── public-action-xyz (repo แยกสำหรับ publish ขึ้น Marketplace)
│   └── action.yml (ที่ root)
│
├── service-a (repository จริงของทีม)
│   └── .github/workflows/ci.yml → เรียก shared-workflows
│
├── service-b (repository จริงของทีม)
│   └── .github/workflows/ci.yml → เรียก shared-workflows
│
└── service-c (repository จริงของทีม)
    └── .github/workflows/ci.yml → เรียก shared-workflows
```

โครงสร้างแบบนี้ทำให้เมื่อทีม platform ต้องการอัปเดต pipeline มาตรฐาน (เช่น เพิ่ม security scan step) สามารถทำที่จุดเดียว (central repo) แล้ว repository อื่นทั้งหมดจะได้รับการอัปเดตทันทีเมื่อ bump moving tag โดยไม่ต้องไปแก้ทีละ repository

---

## Step 700: แบบฝึกหัด — สร้าง Composite Action ของตัวเอง และ Reusable Workflow ที่เรียกใช้ Action นั้น

ถึงเวลาลงมือทำจริง! แบบฝึกหัดนี้จะรวมทุกอย่างที่เรียนมาใน Part นี้เข้าด้วยกัน

### เป้าหมายของแบบฝึกหัด

1. สร้าง **Composite Action** ชื่อ `setup-and-install` ที่ทำหน้าที่ setup environment + install dependencies (รองรับทั้ง Node.js และมี fallback ถ้าไม่มี `package-lock.json`)
2. สร้าง **Reusable Workflow** ชื่อ `reusable-build-test.yml` ที่เรียกใช้ Composite Action ข้างต้น แล้วรัน build และ test
3. สร้าง workflow ตัวอย่างในอีก repository (หรือจำลองในโฟลเดอร์เดียวกัน) ที่เรียกใช้ Reusable Workflow นั้น

### ขั้นตอนที่ 1: เตรียมโครงสร้างโฟลเดอร์

```bash
mkdir -p my-actions-lab/.github/actions/setup-and-install
mkdir -p my-actions-lab/.github/workflows
cd my-actions-lab
git init
```

### ขั้นตอนที่ 2: สร้าง Composite Action

สร้างไฟล์ `.github/actions/setup-and-install/action.yml`:

```yaml
name: 'Setup and Install'
description: 'Setup Node.js และติดตั้ง dependency พร้อม cache ให้อัตโนมัติ'
author: 'my-actions-lab'

inputs:
  node-version:
    description: 'Node.js version ที่จะใช้'
    required: false
    default: '20'

outputs:
  cache-hit:
    description: 'บอกว่า cache node_modules ถูก hit หรือไม่'
    value: ${{ steps.cache-deps.outputs.cache-hit }}

runs:
  using: 'composite'
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}

    - name: ตรวจสอบว่ามี package-lock.json ไหม
      id: check-lockfile
      shell: bash
      run: |
        if [ -f "package-lock.json" ]; then
          echo "has-lockfile=true" >> "$GITHUB_OUTPUT"
        else
          echo "has-lockfile=false" >> "$GITHUB_OUTPUT"
        fi

    - name: Cache node_modules
      id: cache-deps
      if: steps.check-lockfile.outputs.has-lockfile == 'true'
      uses: actions/cache@v4
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ inputs.node-version }}-${{ hashFiles('package-lock.json') }}

    - name: Install ด้วย npm ci (มี lockfile)
      if: steps.check-lockfile.outputs.has-lockfile == 'true' && steps.cache-deps.outputs.cache-hit != 'true'
      shell: bash
      run: npm ci

    - name: Install ด้วย npm install (ไม่มี lockfile)
      if: steps.check-lockfile.outputs.has-lockfile == 'false'
      shell: bash
      run: |
        echo "::warning::ไม่พบ package-lock.json กำลังใช้ npm install แทน npm ci"
        npm install

    - name: สรุปผล
      shell: bash
      run: echo "ติดตั้ง dependency เสร็จสมบูรณ์ด้วย Node.js ${{ inputs.node-version }}"
```

### ขั้นตอนที่ 3: สร้าง Reusable Workflow ที่เรียกใช้ Composite Action

สร้างไฟล์ `.github/workflows/reusable-build-test.yml`:

```yaml
name: Reusable Build and Test

on:
  workflow_call:
    inputs:
      node-version:
        description: 'Node.js version ที่จะใช้'
        type: string
        default: '20'
      run-lint:
        description: 'รัน lint ก่อน build หรือไม่'
        type: boolean
        default: true
    outputs:
      build-status:
        description: 'ผลลัพธ์ของการ build (success/failure)'
        value: ${{ jobs.build-and-test.outputs.status }}

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    outputs:
      status: ${{ steps.set-status.outputs.status }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup และติดตั้ง dependency
        id: setup
        uses: ./.github/actions/setup-and-install
        with:
          node-version: ${{ inputs.node-version }}

      - name: แสดงผล cache
        run: echo "cache-hit คือ ${{ steps.setup.outputs.cache-hit }}"

      - name: Lint (ถ้าเปิดใช้งาน)
        if: inputs.run-lint == true
        run: npm run lint --if-present

      - name: Build
        run: npm run build --if-present

      - name: Test
        run: npm test --if-present

      - name: กำหนดสถานะผลลัพธ์
        id: set-status
        if: always()
        run: |
          if [ "${{ job.status }}" == "success" ]; then
            echo "status=success" >> "$GITHUB_OUTPUT"
          else
            echo "status=failure" >> "$GITHUB_OUTPUT"
          fi
```

### ขั้นตอนที่ 4: สร้าง Workflow หลักที่เรียกใช้ Reusable Workflow

สร้างไฟล์ `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: ./.github/workflows/reusable-build-test.yml
    with:
      node-version: '20'
      run-lint: true

  notify:
    needs: ci
    runs-on: ubuntu-latest
    steps:
      - name: แสดงผลลัพธ์สุดท้าย
        run: echo "ผล build-test คือ ${{ needs.ci.outputs.build-status }}"
```

### ขั้นตอนที่ 5: ทดสอบด้วย matrix เพื่อดูว่าใช้ซ้ำได้จริง

ขยายแบบฝึกหัดต่อยอด ให้ทดสอบ Reusable Workflow เดียวกันกับหลายเวอร์ชัน Node.js พร้อมกัน:

```yaml
name: CI Matrix

on:
  workflow_dispatch:

jobs:
  ci-matrix:
    strategy:
      matrix:
        node-version: ['18', '20', '22']
      fail-fast: false
    uses: ./.github/workflows/reusable-build-test.yml
    with:
      node-version: ${{ matrix.node-version }}
      run-lint: false
```

### ขั้นตอนที่ 6: สร้างไฟล์โปรเจกต์ทดสอบให้ครบ

เพื่อให้ workflow ข้างต้นรันผ่านได้จริง ต้องมีไฟล์ `package.json` พื้นฐาน:

```json
{
  "name": "my-actions-lab",
  "version": "1.0.0",
  "scripts": {
    "build": "echo 'build เสร็จแล้ว'",
    "test": "echo 'test ผ่านทั้งหมด'",
    "lint": "echo 'lint ผ่านทั้งหมด'"
  }
}
```

### ขั้นตอนที่ 7: Commit ทุกอย่างและเปิด Pull Request เพื่อดูผลลัพธ์จริง

```bash
git add .
git commit -m "feat: เพิ่ม composite action และ reusable workflow สำหรับ build/test"
git branch -M main
git remote add origin <URL-repository-ของคุณ>
git push -u origin main
```

จากนั้นสร้าง branch ใหม่ แก้ไขเล็กน้อย แล้วเปิด Pull Request เพื่อดูว่า `ci.yml` เรียก `reusable-build-test.yml` ซึ่งเรียก `setup-and-install` action ต่อกันเป็นทอด ๆ ได้ถูกต้องหรือไม่ โดยดูผลลัพธ์ได้จากแท็บ **Actions** ของ repository

### เกณฑ์ตรวจสอบความสำเร็จของแบบฝึกหัด

- [ ] Composite Action `setup-and-install` ทำงานได้ทั้งกรณีมีและไม่มี `package-lock.json`
- [ ] `cache-hit` output ของ Composite Action แสดงค่าได้ถูกต้อง
- [ ] Reusable Workflow `reusable-build-test.yml` รับ `inputs` และส่งต่อไปยัง Composite Action ได้ถูกต้อง
- [ ] `outputs.build-status` ของ Reusable Workflow สะท้อนผลลัพธ์จริงของ job ภายใน
- [ ] Workflow หลัก `ci.yml` เรียกใช้ Reusable Workflow ได้สำเร็จ และ job `notify` อ่าน output มาแสดงผลได้ถูกต้อง
- [ ] Matrix workflow รันได้ครบทุกเวอร์ชัน Node.js ที่กำหนดโดยไม่ error

### แบบฝึกหัดเสริม (ถ้าต้องการฝึกเพิ่ม)

1. เพิ่ม input `working-directory` ให้ Composite Action รองรับ monorepo ที่มีหลาย package
2. แปลง logic การตรวจสอบ lockfile ใน Composite Action ให้กลายเป็น JavaScript Action แทน (ฝึกใช้ `@actions/core`)
3. เพิ่ม job `security-scan` เข้าไปใน Reusable Workflow โดยใช้ Docker Action ที่สร้างขึ้นเองจาก Step 694
4. ลองสร้าง repository แยกต่างหาก แล้วเรียกใช้ Reusable Workflow จาก repository นี้ข้ามไปมา เพื่อจำลองสถานการณ์ Central Actions Repository ตาม Step 699 จริง ๆ

---

## สรุป Part 70

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Custom Action มี 3 แบบ** — Composite Action (รวม step YAML), JavaScript Action (รันโค้ด JS บน Node.js runtime), และ Docker Action (รันใน container ที่กำหนดเอง) แต่ละแบบมีจุดเด่นและข้อจำกัดต่างกันชัดเจน
2. **Composite Action** สร้างง่ายที่สุด ใช้ YAML + shell script ล้วน ๆ แต่ต้องระบุ `shell:` ทุก step ที่เป็น `run:`
3. **JavaScript Action** ต้อง bundle dependency ด้วยเครื่องมืออย่าง `ncc` ให้เหลือไฟล์เดียวก่อน commit เพราะ runner ไม่รัน `npm install` ให้
4. **Docker Action** ยืดหยุ่นสูงสุดแต่รันได้เฉพาะ Linux runner และควรใช้ pre-built image แทนการ build สดทุกครั้งเพื่อความเร็ว
5. การ **publish ขึ้น GitHub Marketplace** ต้องมี `action.yml` ที่ root, repository ต้องเป็น public, และควรดูแล moving major version tag (`v1`, `v2`) ให้ผู้ใช้ pin ได้สะดวก
6. **Reusable Workflows** (`on: workflow_call`) ทำงานในระดับ job ทั้งก้อน ต่างจาก Custom Action ที่ทำงานในระดับ step
7. เรียกใช้ Reusable Workflow ด้วย `uses:` ที่ระดับ job พร้อมส่ง `with:` และ `secrets:` (หรือ `secrets: inherit` ถ้าต้องการส่งทั้งหมด)
8. **หลักเลือกใช้**: Composite Action สำหรับขั้นตอนย่อยในงานเดียว, Reusable Workflow สำหรับ pipeline ทั้งชุดที่มีหลาย job — และมักใช้ผสมกันในทางปฏิบัติจริง
9. องค์กรขนาดใหญ่ควรมี **Central Actions Repository** เก็บ Action และ Reusable Workflow มาตรฐานไว้ที่เดียว พร้อม Semantic Versioning, CODEOWNERS, และการทดสอบตัวเองอย่างเข้มงวด
10. ลงมือสร้าง Composite Action และ Reusable Workflow ของตัวเองจริง พร้อมทดสอบด้วย matrix เพื่อยืนยันว่าใช้ซ้ำได้ตามที่ออกแบบไว้

**ต่อไป:** [Part 71: GitLab CI/CD ขั้นสูง: Stages, Pipelines, Cache](./part-071-gitlab-cicd-ขั้นสูง.md)
