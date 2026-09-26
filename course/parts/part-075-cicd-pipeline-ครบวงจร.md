# Part 75: โปรเจกต์ฝึกหัด: สร้าง CI/CD Pipeline ครบวงจร

> **Step ในหลักสูตรนี้:** Step 741–750
> **เฟส:** 7 — CI/CD (Part นี้คือ Part ปิดท้ายของเฟส 7 ทั้งหมด)
> **เป้าหมายของ Part นี้:** นำทุกสิ่งที่เรียนมาตลอดเฟส 7 ตั้งแต่ Part 66 ถึง Part 74 มาประกอบร่างเข้าด้วยกันในภารกิจเดียวที่ครบวงจรที่สุด — สร้าง CI/CD Pipeline แบบเต็มรูปแบบให้กับโปรเจกต์ `portfolio-project` ที่มีมาตั้งแต่ Part 15 (สร้างโปรเจกต์เล็กด้วย Git), Part 19 (README/Markdown), Part 25 (GitHub Pages) และย้ายมาอยู่บน GitLab ตั้งแต่ Part 55 โดย Pipeline นี้จะครอบคลุมทุกสิ่งที่ทีมพัฒนาซอฟต์แวร์มืออาชีพใช้งานจริง: lint, automated test พร้อม coverage report, build, security scan (SAST + Secret Detection), deploy อัตโนมัติขึ้น staging ทุกครั้งที่ merge เข้า main, deploy ขึ้น production แบบต้องมี manual approval, และระบบแจ้งเตือนผลลัพธ์ของ pipeline ก่อนจะสรุปภาพรวมทั้งเฟส 7 และก้าวเข้าสู่เฟส 8

---

## สารบัญของ Part นี้

- Step 741: ภาพรวมภารกิจปิดเฟส 7 — สร้าง CI/CD Pipeline ครบวงจรให้โปรเจกต์จริง
- Step 742: วางแผน Pipeline ทั้งหมด — Stage ที่ต้องมีและโครงสร้างไฟล์เริ่มต้น
- Step 743: เขียน Stage/Job Build — Compile และ Bundle โปรเจกต์
- Step 744: เขียน Stage/Job Test — Unit Test พร้อม Coverage Report
- Step 745: เขียน Stage/Job Lint และ Code Quality Check
- Step 746: เขียน Stage/Job Security Scan — SAST และ Secret Detection
- Step 747: เขียน Stage/Job Deploy to Staging — อัตโนมัติทุกครั้งที่ Merge เข้า Main
- Step 748: เขียน Stage/Job Deploy to Production — ต้องมี Manual Approval
- Step 749: ตั้งค่า Notification เมื่อ Pipeline สำเร็จหรือล้มเหลว
- Step 750: สรุปทบทวนภาพรวมเฟส 7 ทั้งหมด (Part 66–75) พร้อม Cheat Sheet และ Checklist ก่อนเข้าสู่เฟส 8

---

## Step 741: ภาพรวมภารกิจปิดเฟส 7 — สร้าง CI/CD Pipeline ครบวงจรให้โปรเจกต์จริง

ตลอด 9 Part ที่ผ่านมาในเฟส 7 (Part 66–74) คุณได้เรียนรู้แนวคิดและเครื่องมือของโลก CI/CD แยกเป็นชิ้น ๆ มาครบแล้ว: ความหมายและความสำคัญของ CI/CD, GitHub Actions ตั้งแต่ workflow แรกไปจนถึง matrix build, secrets/environments/artifacts, custom actions และ reusable workflows, GitLab CI/CD ขั้นสูงเรื่อง stages/pipelines/cache, multi-project และ dynamic pipelines, automated testing (unit/integration/E2E) พร้อม coverage report, และการ deploy อัตโนมัติด้วย Docker/Kubernetes/Cloud

Part นี้จะ **ไม่สอนแนวคิดใหม่แม้แต่อย่างเดียว** เช่นเดียวกับที่ Part 15 ปิดเฟส 2, Part 30 ปิดเฟส 3, Part 45 ปิดเฟส 4 และ Part 55 ปิดเฟส 5 — แต่จะเอาทุกอย่างที่เรียนมาทั้งหมดของเฟส 7 มาร้อยเรียงเข้าด้วยกันในภารกิจเดียวที่สมจริงที่สุด: **การสร้าง CI/CD Pipeline แบบเต็มรูปแบบให้กับโปรเจกต์ที่มีอยู่จริง** ไม่ใช่โปรเจกต์ตัวอย่างใหม่ที่สร้างขึ้นมาแบบลอย ๆ

### 741.1 โปรเจกต์ที่เราจะสร้าง Pipeline ให้

เราจะใช้โปรเจกต์เดียวกับที่เดินทางมาด้วยกันตั้งแต่เฟส 2:

| Part | สิ่งที่ได้จาก Part นั้น |
|---|---|
| Part 15 (จบเฟส 2) | Repository `portfolio-project` พร้อมประวัติ commit เต็ม, branch `main`, tag `v1.0.0` |
| Part 19 (เฟส 3) | README.md ที่เขียนอย่างมืออาชีพ พร้อม badge |
| Part 25 (เฟส 3) | เว็บไซต์ที่ deploy อยู่บน GitHub Pages |
| Part 55 (จบเฟส 5) | โปรเจกต์ทั้งหมดถูกย้ายจาก GitHub มาอยู่บน **GitLab** เรียบร้อยแล้ว พร้อม Merge Request, Protected Branches และ `.gitlab-ci.yml` เบื้องต้น (แปลงมาจาก GitHub Actions ในตอนนั้น) |

เนื่องจากโปรเจกต์นี้อยู่บน GitLab มาตั้งแต่ Part 55 แล้ว Part นี้จะสร้าง Pipeline ฉบับเต็มด้วย **GitLab CI/CD** (`.gitlab-ci.yml`) เป็นหลัก โดยในทุก Step เราจะโยงกลับไปยังแนวคิดคู่ขนานฝั่ง GitHub Actions ที่เรียนไปในเฟสเดียวกันด้วยเสมอ เพื่อให้คุณนำไปประยุกต์ใช้ได้ไม่ว่าจะทำงานบนแพลตฟอร์มไหนในอนาคต

> **หมายเหตุ:** ถ้าคุณข้ามแบบฝึกหัดใน Part ก่อนหน้ามา ให้สร้าง repository เปล่าบน GitLab ชื่อ `portfolio-project` ใส่ไฟล์ `index.html`, `css/style.css`, `js/app.js` แบบง่าย ๆ ตาม Part 15 ก่อน แล้วค่อยตามขั้นตอนใน Part นี้ต่อได้ทันที โครงสร้างไฟล์ที่ต้องมีก่อนเริ่ม Step 742 คือ:
>
> ```
> portfolio-project/
> ├── index.html
> ├── css/
> │   └── style.css
> ├── js/
> │   └── app.js
> ├── README.md
> └── .gitlab-ci.yml   (เวอร์ชันเบื้องต้นจาก Part 55)
> ```

### 741.2 เพิ่ม Toolchain ให้โปรเจกต์มีของจริงให้ Pipeline ทำงานด้วย

Pipeline ที่ดีต้องมี **ของจริงให้ทำงาน** ไม่ใช่แค่ job เปล่า ๆ ที่ไม่ได้ตรวจสอบอะไร ก่อนเขียน `.gitlab-ci.yml` ฉบับเต็ม เราจะเพิ่มโค้ด JavaScript เชิงฟังก์ชันเล็ก ๆ เข้าไปในโปรเจกต์ พร้อม unit test, lint config และ build script เพื่อให้ทุก stage ที่เราจะสร้างมีสิ่งที่ต้อง lint จริง, test จริง, และ build จริง

โครงสร้างไฟล์เป้าหมายหลัง Step นี้:

```
portfolio-project/
├── index.html
├── css/
│   └── style.css
├── js/
│   └── app.js
├── src/
│   └── utils.js
├── tests/
│   └── utils.test.js
├── scripts/
│   └── build.js
├── package.json
├── package-lock.json
├── jest.config.js
├── .eslintrc.json
├── README.md
└── .gitlab-ci.yml
```

เริ่มจาก clone โปรเจกต์มาที่เครื่อง (ถ้ายังไม่มี):

```bash
cd ~/git-course
git clone git@gitlab.com:somchai-jaidee/portfolio-project.git
cd portfolio-project
git switch main
git pull origin main
```

สร้างฟังก์ชันช่วยเหลือสองตัวที่หน้าเว็บ portfolio จะใช้จริง — คำนวณเวลาที่ใช้อ่านบทความ และจัดรูปแบบวันที่เป็นภาษาไทย:

```bash
mkdir -p src tests scripts

cat > src/utils.js << 'EOF'
// src/utils.js: ฟังก์ชันช่วยเหลือสำหรับหน้า portfolio

function calculateReadingTime(text, wordsPerMinute = 200) {
  if (typeof text !== "string" || text.trim().length === 0) return 0;
  const wordCount = text.trim().split(/\s+/).length;
  return Math.ceil(wordCount / wordsPerMinute);
}

function formatProjectDate(isoDateString) {
  const date = new Date(isoDateString);
  if (Number.isNaN(date.getTime())) return "ไม่ทราบวันที่";
  return date.toLocaleDateString("th-TH", {
    year: "numeric",
    month: "long",
    day: "numeric",
  });
}

module.exports = { calculateReadingTime, formatProjectDate };
EOF
```

เขียน unit test คู่กันตามหลักที่จะได้เรียนละเอียดใน Part 73:

```bash
cat > tests/utils.test.js << 'EOF'
const { calculateReadingTime, formatProjectDate } = require("../src/utils");

describe("calculateReadingTime", () => {
  test("คำนวณเวลาอ่านจากจำนวนคำได้ถูกต้อง", () => {
    const text = "word ".repeat(400);
    expect(calculateReadingTime(text)).toBe(2);
  });

  test("คืนค่า 0 เมื่อข้อความว่าง", () => {
    expect(calculateReadingTime("")).toBe(0);
  });

  test("คืนค่า 0 เมื่อ input ไม่ใช่ string", () => {
    expect(calculateReadingTime(null)).toBe(0);
  });
});

describe("formatProjectDate", () => {
  test("แปลงวันที่ ISO เป็นรูปแบบภาษาไทยได้ถูกต้อง", () => {
    expect(formatProjectDate("2026-01-15")).toContain("2569");
  });

  test("คืนค่าข้อความแจ้งเตือนเมื่อวันที่ไม่ถูกต้อง", () => {
    expect(formatProjectDate("invalid-date")).toBe("ไม่ทราบวันที่");
  });
});
EOF
```

เขียน build script ที่รวมไฟล์ static ทั้งหมดไปไว้ที่ `dist/` พร้อมแปะข้อมูล build ไว้ตรวจสอบย้อนหลังได้:

```bash
cat > scripts/build.js << 'EOF'
// scripts/build.js: รวมไฟล์ static ของ portfolio-project ไปไว้ที่ dist/ สำหรับ deploy
const fs = require("fs");
const path = require("path");

const ROOT_DIR = path.join(__dirname, "..");
const DIST_DIR = path.join(ROOT_DIR, "dist");
const ASSETS = ["index.html", "css", "js"];

function copyRecursive(src, dest) {
  const stat = fs.statSync(src);
  if (stat.isDirectory()) {
    fs.mkdirSync(dest, { recursive: true });
    for (const entry of fs.readdirSync(src, { withFileTypes: true })) {
      copyRecursive(path.join(src, entry.name), path.join(dest, entry.name));
    }
  } else {
    fs.mkdirSync(path.dirname(dest), { recursive: true });
    fs.copyFileSync(src, dest);
  }
}

fs.rmSync(DIST_DIR, { recursive: true, force: true });
fs.mkdirSync(DIST_DIR, { recursive: true });

for (const asset of ASSETS) {
  copyRecursive(path.join(ROOT_DIR, asset), path.join(DIST_DIR, asset));
}

const buildInfo = {
  builtAt: new Date().toISOString(),
  commit: process.env.CI_COMMIT_SHORT_SHA || "local",
  pipeline: process.env.CI_PIPELINE_ID || "local",
};
fs.writeFileSync(
  path.join(DIST_DIR, "build-info.json"),
  JSON.stringify(buildInfo, null, 2)
);

console.log(`Build เสร็จสมบูรณ์: คัดลอกไฟล์ไปยัง ${DIST_DIR}`);
EOF
```

ตั้งค่า `package.json` ให้มี script มาตรฐานสามตัวที่ Pipeline จะเรียกใช้ตรง ๆ (`lint`, `test`, `build`) — นี่คือรูปแบบมาตรฐานที่ CI ทุกแพลตฟอร์มคาดหวังจากโปรเจกต์ Node.js:

```bash
cat > package.json << 'EOF'
{
  "name": "portfolio-project",
  "version": "1.0.0",
  "private": true,
  "description": "เว็บไซต์ portfolio ส่วนตัว ใช้ฝึกสร้าง CI/CD Pipeline ครบวงจร",
  "scripts": {
    "lint": "eslint . --ext .js",
    "test": "jest --coverage",
    "build": "node scripts/build.js"
  },
  "devDependencies": {
    "eslint": "^8.57.0",
    "jest": "^29.7.0",
    "jest-junit": "^16.0.0"
  }
}
EOF
```

ตั้งค่า ESLint:

```bash
cat > .eslintrc.json << 'EOF'
{
  "env": {
    "node": true,
    "es2021": true,
    "jest": true
  },
  "extends": "eslint:recommended",
  "parserOptions": {
    "ecmaVersion": 12,
    "sourceType": "module"
  },
  "rules": {
    "no-unused-vars": "warn",
    "no-console": "off"
  }
}
EOF
```

ตั้งค่า Jest ให้ออก coverage report เป็นทั้งแบบอ่านเองได้ (text) และแบบเครื่องอ่านได้ (cobertura) พร้อมออก JUnit report สำหรับผลการทดสอบ — ทั้งสองรูปแบบนี้คือมาตรฐานที่ GitLab (และ GitHub Actions ผ่าน action เสริม) ใช้แสดงผลใน UI ตามที่จะเรียนละเอียดใน Part 73:

```bash
cat > jest.config.js << 'EOF'
module.exports = {
  testEnvironment: "node",
  coverageReporters: ["text", "cobertura", "lcov"],
  reporters: [
    "default",
    ["jest-junit", { outputDirectory: "reports", outputName: "junit.xml" }],
  ],
};
EOF
```

Commit ทุกอย่างเข้า branch ใหม่ (ตาม naming convention จาก Part 32 ที่เรายังยึดถือมาตลอดหลักสูตร):

```bash
git switch -c feature/cicd-toolchain-setup
git add src tests scripts package.json jest.config.js .eslintrc.json
git commit -m "เพิ่ม toolchain: eslint, jest, build script สำหรับเตรียม CI/CD pipeline"
git push -u origin feature/cicd-toolchain-setup
```

เปิด Merge Request และ merge เข้า `main` ตามขั้นตอนปกติที่เรียนมาตั้งแต่ Part 48 จากนั้นดึงกลับมาที่เครื่อง:

```bash
git switch main
git pull origin main
```

### 741.3 แผนงานทั้งหมดของ Part นี้ในสายตาเดียว

ก่อนลงมือเขียน Pipeline จริง มาดูภาพรวมว่า Part นี้จะพาคุณผ่านอะไรบ้าง โดยแต่ละแถวคือการนำทักษะจาก Part ก่อนหน้าของเฟส 7 มาใช้งานจริง:

| Step | สิ่งที่สร้าง | ทักษะที่ใช้จาก Part ก่อนหน้า |
|---|---|---|
| 742 | วางโครง `.gitlab-ci.yml` ทั้งหมด | CI/CD คืออะไร (66), Stages/Pipelines (71) |
| 743 | Job `build` | GitHub Actions Jobs/Steps (68), GitLab Cache (71) |
| 744 | Job `unit_test` พร้อม coverage | Automated Testing/Coverage (73) |
| 745 | Job `eslint_check` + `code_quality` | GitHub Actions พื้นฐาน (67), GitLab Jobs (71) |
| 746 | Job `sast` + `secret_detection` | GitLab Security Features (Part 54, เฟส 5) |
| 747 | Job `deploy_staging` | Environments/Artifacts (69), Deploy อัตโนมัติ (74) |
| 748 | Job `deploy_production` (manual) | Environments + Required Reviewers (69), `when: manual` (71) |
| 749 | Job `notify_success` / `notify_failure` | Secrets (69), Rules ขั้นสูง (71-72) |
| 750 | สรุปเฟส 7 ทั้งหมด | ทุก Part 66–75 |

จากนี้ไป เราจะเดินหน้าไปทีละ Step ตามตารางด้านบน โดยแต่ละ Step จะเพิ่มเนื้อหาเข้าไปใน `.gitlab-ci.yml` ไฟล์เดียวกันเรื่อย ๆ จนครบสมบูรณ์ใน Step 750

---

## Step 742: วางแผน Pipeline ทั้งหมด — Stage ที่ต้องมีและโครงสร้างไฟล์เริ่มต้น

ก่อนเขียนโค้ดจริงสักบรรทัด วิศวกร DevOps มืออาชีพจะวางแผนโครงสร้าง Pipeline ทั้งหมดก่อนเสมอ เพื่อให้เห็นภาพรวมว่าโค้ดจะไหลจาก commit ไปจนถึง production ผ่านขั้นตอนอะไรบ้าง

### 742.1 รายการ Stage ที่ต้องมี

Pipeline ของ `portfolio-project` ต้องมี 6 stage เรียงตามลำดับที่โค้ดต้องไหลผ่าน:

```
┌────────┐   ┌────────┐   ┌───────┐   ┌──────────┐   ┌────────────────┐   ┌───────────────────┐
│  lint  │──▶│  test  │──▶│ build │──▶│ security │──▶│ deploy-staging  │──▶│ deploy-production  │
└────────┘   └────────┘   └───────┘   └──────────┘   └────────────────┘   └───────────────────┘
  eslint     unit test     bundle       SAST +          rsync ขึ้น            rsync ขึ้น
  code       + coverage    + build-     Secret          server staging       server production
  quality                  info.json    Detection       (อัตโนมัติทุก         (ต้อง manual
                                                          push เข้า main)      approve)
```

| Stage | หน้าที่ | Job ที่อยู่ใน stage นี้ |
|---|---|---|
| `lint` | ตรวจสอบคุณภาพโค้ดก่อนเสียเวลา test/build | `eslint_check`, `code_quality` |
| `test` | รัน unit test พร้อมวัด coverage | `unit_test` |
| `build` | bundle โปรเจกต์ให้พร้อม deploy | `build` |
| `security` | สแกนหาช่องโหว่และ secret ที่หลุดเข้ามาในโค้ด | `sast`, `secret_detection` |
| `deploy-staging` | deploy ขึ้นเซิร์ฟเวอร์ทดสอบอัตโนมัติ | `deploy_staging` |
| `deploy-production` | deploy ขึ้นเซิร์ฟเวอร์จริงแบบมีคนอนุมัติ | `deploy_production` |

> **ทำไมต้องเรียง lint และ test ไว้ก่อน build:** หลักการสำคัญของ Pipeline ที่ดีคือ **"fail fast"** — ถ้าโค้ดมีปัญหาเรื่องคุณภาพหรือ logic ผิดตั้งแต่ต้น ควรรู้ให้เร็วที่สุดโดยไม่ต้องเสียเวลาไป build หรือ deploy ก่อน การจัดลำดับ stage แบบนี้ช่วยประหยัดเวลาและทรัพยากรของ CI runner ได้มากในระยะยาว โดยเฉพาะเมื่อโปรเจกต์โตขึ้นและ build ใช้เวลานาน

### 742.2 ค่าตั้งต้นร่วมของทุก Job (Global Configuration)

เริ่มเขียนโครง `.gitlab-ci.yml` ด้วยส่วนที่ทุก job จะใช้ร่วมกัน ตามหลัก `stages`, `variables`, `default` และ `cache` ที่เรียนละเอียดใน Part 71:

```bash
cd ~/git-course/portfolio-project

cat > .gitlab-ci.yml << 'EOF'
# .gitlab-ci.yml ของ portfolio-project
# Pipeline ครบวงจร: lint -> test -> build -> security -> deploy-staging -> deploy-production

stages:
  - lint
  - test
  - build
  - security
  - deploy-staging
  - deploy-production

variables:
  NODE_VERSION: "20"
  DIST_DIR: "dist"

default:
  image: node:${NODE_VERSION}-alpine
  cache:
    key:
      files:
        - package-lock.json
    paths:
      - node_modules/
  before_script:
    - npm ci
EOF
```

อธิบายทีละส่วน:

- **`stages`** — ประกาศลำดับ stage ทั้ง 6 ตัวตามที่วางแผนไว้ใน 742.1 job ใดไม่ระบุ `stage:` จะถูกปฏิเสธเพราะ GitLab บังคับให้ทุก job ต้องอยู่ใน stage ที่ประกาศไว้แล้วเท่านั้น
- **`variables`** — ตัวแปรระดับ pipeline ที่ใช้ซ้ำได้ทุก job เช่น `NODE_VERSION` และ `DIST_DIR` (โฟลเดอร์ผลลัพธ์ของ build) การประกาศไว้จุดเดียวทำให้แก้ทีเดียวมีผลทั้งไฟล์
- **`default: image`** — ทุก job ใช้ image `node:20-alpine` เป็นค่าเริ่มต้น (job ไหนต้องการ image อื่น เช่น job deploy ที่ต้องใช้ `ssh`/`rsync` จะ override ทีหลัง)
- **`default: cache`** — cache โฟลเดอร์ `node_modules/` โดยใช้ hash ของ `package-lock.json` เป็น key ถ้า lock file ไม่เปลี่ยน ทุก job จะดึง cache เดิมมาใช้แทนการ `npm ci` ใหม่ทั้งหมด ทำให้ pipeline เร็วขึ้นมากตามหลักที่เรียนใน Part 71
- **`default: before_script`** — รัน `npm ci` ก่อนทุก job โดยอัตโนมัติ (ใช้ `npm ci` แทน `npm install` เพราะ `ci` เร็วกว่าและยึดตาม `package-lock.json` เป๊ะ ๆ เหมาะกับสภาพแวดล้อม CI ที่ต้องการผลลัพธ์เดิมซ้ำ ๆ เสมอ)

### 742.3 เทียบแนวคิดกับ GitHub Actions ที่เรียนไปในเฟสเดียวกัน

แม้ Pipeline ของ Part นี้จะเขียนด้วย GitLab CI/CD แต่แนวคิดเดียวกันนี้แปลงเป็น GitHub Actions ได้ตรง ๆ ตามที่เรียนใน Part 67–70:

| แนวคิด | GitLab CI/CD (Part 71) | GitHub Actions (Part 67–70) |
|---|---|---|
| หน่วยงานที่รันเรียงกัน | `stages` | `jobs` ที่เชื่อมด้วย `needs:` |
| ตัวแปรร่วมทั้งไฟล์ | `variables:` ระดับบนสุด | `env:` ระดับ workflow |
| Cache dependency | `cache:` + `key.files` | `actions/cache@v4` + `hashFiles()` |
| ค่าเริ่มต้นของทุก job | `default:` | ไม่มีโดยตรง ต้องกำหนดซ้ำในแต่ละ job หรือใช้ reusable workflow (Part 70) |
| Container ที่ job รันอยู่ | `image:` | `runs-on:` + `container:` |

การเข้าใจว่าแนวคิดหลักของ CI/CD (stage, dependency, cache, artifact, secret) เหมือนกันในทุกแพลตฟอร์ม คือสิ่งสำคัญที่สุด — syntax เปลี่ยนได้ตามเครื่องมือ แต่หลักการไม่เปลี่ยน

### 742.4 Commit โครงเริ่มต้น

```bash
git add .gitlab-ci.yml
git commit -m "วางโครง .gitlab-ci.yml: ประกาศ stages, variables, cache ร่วมของทุก job"
git push origin main
```

ตอนนี้เรามีโครง Pipeline ที่ยังไม่มี job ใดทำงานจริง ขั้นตอนถัดไปคือเติม job ทีละ stage

---

## Step 743: เขียน Stage/Job Build — Compile และ Bundle โปรเจกต์

เริ่มจาก job ที่ตรงไปตรงมาที่สุดก่อน คือ `build` ซึ่งจะเรียก `npm run build` ที่เราเตรียมไว้ใน Step 741 เพื่อรวมไฟล์ static ทั้งหมดไปไว้ที่ `dist/`

### 743.1 เขียน Job `build`

เพิ่มเข้าไปในไฟล์ `.gitlab-ci.yml`:

```yaml
build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - $DIST_DIR/
    expire_in: 1 week
```

อธิบายทีละ key:

- **`stage: build`** — ประกาศว่า job นี้อยู่ใน stage `build` ตามที่วางแผนไว้ใน Step 742
- **`script`** — คำสั่งจริงที่ job นี้รัน คือ `npm run build` ซึ่งจะไปเรียก `scripts/build.js` ที่เราเขียนไว้แล้ว
- **`artifacts: paths`** — บอก GitLab ให้เก็บโฟลเดอร์ `dist/` (ค่าจากตัวแปร `$DIST_DIR`) ไว้เป็น **artifact** หลังจาก job นี้จบ เพื่อให้ job ใน stage ถัดไป (เช่น `deploy_staging`, `deploy_production`) ดึงไปใช้ต่อได้โดยไม่ต้อง build ซ้ำ — นี่คือหลักการ "build once, deploy many" ที่สำคัญมากในโลก CI/CD จริง: build เพียงครั้งเดียว แล้วนำ artifact เดียวกันไป deploy ทั้ง staging และ production เพื่อรับประกันว่าสิ่งที่ทดสอบบน staging คือสิ่งเดียวกันเป๊ะกับที่ขึ้น production
- **`expire_in: 1 week`** — artifact จะถูกลบทิ้งอัตโนมัติหลังจาก 1 สัปดาห์ เพื่อไม่ให้พื้นที่เก็บข้อมูลของ GitLab บวมขึ้นเรื่อย ๆ จาก build เก่า ๆ ที่ไม่มีใครใช้แล้ว

### 743.2 ทดสอบรัน Pipeline ครั้งแรก

```bash
git add .gitlab-ci.yml
git commit -m "เพิ่ม job build: bundle โปรเจกต์ไปยัง dist/ พร้อมเก็บเป็น artifact"
git push origin main
```

เปิดหน้า **CI/CD > Pipelines** บน GitLab จะเห็นสถานะประมาณนี้:

```
Pipeline #482 for main (a7c4e21)
├── lint              (ยังไม่มี job — ข้าม)
├── test              (ยังไม่มี job — ข้าม)
├── build
│   └── build         ✔ passed (18s)
├── security          (ยังไม่มี job — ข้าม)
├── deploy-staging    (ยังไม่มี job — ข้าม)
└── deploy-production (ยังไม่มี job — ข้าม)
```

> **หมายเหตุ:** GitLab จะข้าม stage ที่ไม่มี job อยู่เลยโดยอัตโนมัติ ไม่ถือว่า pipeline ผิดพลาด — นี่คือเหตุผลที่เราสามารถทยอยเติม job เข้าไปทีละ stage ได้อย่างปลอดภัยตลอด Part นี้โดยไม่ทำให้ Pipeline พังระหว่างทาง

คลิกเข้าไปดู job `build` เพื่อยืนยันว่า artifact ถูกสร้างจริง:

```
Job succeeded

$ npm ci
$ npm run build
> portfolio-project@1.0.0 build
> node scripts/build.js
Build เสร็จสมบูรณ์: คัดลอกไฟล์ไปยัง /builds/somchai-jaidee/portfolio-project/dist

Uploading artifacts for successful job
Uploading artifacts...
dist: found 5 matching artifact files and directories
Uploading artifacts as "archive" to coordinator... ok
```

Job `build` ทำงานถูกต้องแล้ว ขั้นตอนถัดไปคือเพิ่ม stage ที่ควรรันมาก่อน `build` จริง ๆ ตามหลัก fail-fast คือ `test`

---

## Step 744: เขียน Stage/Job Test — Unit Test พร้อม Coverage Report

Step นี้จะทำสิ่งที่เรียนละเอียดใน **Part 73: Automated Testing ใน Pipeline** มาใช้งานจริง — รัน unit test ของ `tests/utils.test.js` ที่เตรียมไว้ พร้อมสร้าง coverage report ให้ GitLab แสดงผลใน UI

### 744.1 เขียน Job `unit_test`

```yaml
unit_test:
  stage: test
  script:
    - npm test -- --coverage
  coverage: '/All files\s*\|\s*([\d.]+)/'
  artifacts:
    when: always
    reports:
      junit: reports/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/
    expire_in: 7 days
```

อธิบายทีละส่วน:

- **`script: npm test -- --coverage`** — เรียก `jest --coverage` ผ่าน npm script ที่ตั้งไว้ใน `package.json` ทำให้ Jest รันทุก test ใน `tests/` และวัด code coverage ไปพร้อมกัน
- **`coverage:` (regex)** — นี่คือฟีเจอร์ระดับ job ของ GitLab ที่ดึงตัวเลข coverage ออกมาจาก log ของ job ด้วย regular expression แล้วนำไปแสดงเป็น badge เปอร์เซ็นต์บนหน้า pipeline และหน้า Merge Request โดยตรง regex นี้จับข้อความ `All files | 92.31 | ...` ที่ Jest พิมพ์ในตารางสรุป coverage แบบ text ออกมาเป็นตัวเลข
- **`artifacts: when: always`** — สั่งให้เก็บ artifact **แม้ job จะ fail** (ปกติ artifact จะถูกเก็บเฉพาะตอน job สำเร็จ) เพราะเราต้องการเห็นรายงาน test ที่ fail ด้วยเช่นกันเพื่อ debug
- **`artifacts: reports: junit`** — บอก GitLab ให้อ่านไฟล์ `reports/junit.xml` (ที่ `jest-junit` สร้างให้ตาม config ใน `jest.config.js`) แล้วแสดงผลการทดสอบแบบละเอียดในแท็บ **Tests** ของหน้า pipeline — เห็นได้ทันทีว่า test ไหนผ่าน ไหนไม่ผ่าน โดยไม่ต้องไล่อ่าน log ทั้งหมด
- **`artifacts: reports: coverage_report`** — ฟีเจอร์ใหม่กว่า `coverage:` regex คือให้ GitLab อ่านไฟล์ coverage แบบ Cobertura XML โดยตรง ทำให้ GitLab สามารถแสดง **coverage แบบ inline ในหน้า diff ของ Merge Request** ได้เลยว่าบรรทัดไหนถูก test ครอบคลุมหรือไม่ (สีเขียว/สีแดงข้างเลขบรรทัด) ซึ่งละเอียดกว่าตัวเลขเปอร์เซ็นต์เฉย ๆ มาก

> **ทำไมต้องใช้ทั้ง `coverage:` regex และ `coverage_report`:** ทั้งสองอย่างเสริมกัน ไม่ได้ซ้ำซ้อน — `coverage:` regex ให้ตัวเลขสรุปเร็ว ๆ ที่เห็นบนหน้า pipeline list ทันที ส่วน `coverage_report` ให้รายละเอียดระดับบรรทัดในหน้า diff ของ MR การใส่ทั้งคู่คือแนวทางที่แนะนำในทีมที่จริงจังเรื่องคุณภาพโค้ด

### 744.2 ทดสอบรัน Pipeline

```bash
git add .gitlab-ci.yml
git commit -m "เพิ่ม job unit_test: รัน jest พร้อมสร้าง coverage report (junit + cobertura)"
git push origin main
```

ผลลัพธ์บน pipeline:

```
Pipeline #483 for main (b8d5f32)
├── lint              (ยังไม่มี job — ข้าม)
├── test
│   └── unit_test     ✔ passed (12s) — coverage: 88.24%
├── build
│   └── build         ✔ passed (17s)
├── security          (ยังไม่มี job — ข้าม)
├── deploy-staging    (ยังไม่มี job — ข้าม)
└── deploy-production (ยังไม่มี job — ข้าม)
```

log ของ job `unit_test`:

```
$ npm test -- --coverage
PASS tests/utils.test.js
  calculateReadingTime
    ✓ คำนวณเวลาอ่านจากจำนวนคำได้ถูกต้อง (3 ms)
    ✓ คืนค่า 0 เมื่อข้อความว่าง (1 ms)
    ✓ คืนค่า 0 เมื่อ input ไม่ใช่ string
  formatProjectDate
    ✓ แปลงวันที่ ISO เป็นรูปแบบภาษาไทยได้ถูกต้อง (2 ms)
    ✓ คืนค่าข้อความแจ้งเตือนเมื่อวันที่ไม่ถูกต้อง

----------|---------|----------|---------|---------|
File      | % Stmts | % Branch | % Funcs | % Lines |
----------|---------|----------|---------|---------|
All files |   88.24 |    83.33 |     100 |   88.24 |
----------|---------|----------|---------|---------|

Test Suites: 1 passed, 1 total
Tests:       5 passed, 5 total
```

สังเกตว่าตัวเลข `88.24` ตรงกับที่ GitLab แสดงบนหน้า pipeline พอดี — regex ที่เขียนไว้ทำงานถูกต้อง

---

## Step 745: เขียน Stage/Job Lint และ Code Quality Check

ตาม Pipeline ที่วางแผนไว้ `lint` ควรเป็น stage แรกสุด (fail fast) แต่เราจงใจเขียนหลัง `build` และ `test` ก่อนเพื่อให้เห็น job ที่ตรงไปตรงมาก่อน — ตอนนี้ถึงเวลาเติม stage แรกสุดให้ครบ

### 745.1 เขียน Job `eslint_check`

```yaml
eslint_check:
  stage: lint
  script:
    - npm run lint
```

Job นี้เรียบง่ายมาก: รัน `npm run lint` ซึ่งเรียก `eslint . --ext .js` ตาม config ใน `.eslintrc.json` ที่ตั้งไว้ ถ้ามีโค้ดที่ผิดกฎ เช่น ตัวแปรที่ไม่ได้ใช้ (`no-unused-vars`) หรือ syntax error ระดับพื้นฐาน ESLint จะ exit code ไม่เป็น 0 ทำให้ job นี้ fail และ pipeline หยุดทันทีก่อนจะเสียเวลาไป test/build

### 745.2 เพิ่ม Job `code_quality` ด้วย Managed Template ของ GitLab

นอกจาก ESLint แบบกำหนดเองแล้ว GitLab ยังมี **Managed Template** สำเร็จรูปสำหรับตรวจสอบคุณภาพโค้ดแบบกว้าง ๆ (code smells, complexity, duplication) โดยไม่ต้องเขียนเอง — เพิ่ม `include:` ไว้ที่ด้านบนของไฟล์:

```yaml
include:
  - template: Jobs/Code-Quality.gitlab-ci.yml
```

Template นี้จะเพิ่ม job ชื่อ `code_quality` เข้ามาให้อัตโนมัติ (รันด้วย image ของ CodeClimate เบื้องหลัง) แต่ค่าเริ่มต้นของ template จะกำหนด stage เป็น `test` ซึ่งไม่ตรงกับแผนของเรา เราจึง **override เฉพาะ key ที่ต้องการ** โดยประกาศชื่อ job เดิมซ้ำแล้วใส่แค่ key ที่จะเปลี่ยน — GitLab จะรวม (merge) ค่าที่ประกาศใหม่เข้ากับของเดิมจาก template ให้อัตโนมัติ:

```yaml
code_quality:
  stage: lint
```

เพียงเท่านี้ job `code_quality` จากไฟล์ template ก็จะย้ายมาอยู่ใน stage `lint` ตามที่เราต้องการ โดยยังคงพฤติกรรมภายในอื่น ๆ (image, script, artifacts) ไว้ตามเดิมทุกประการ

### 745.3 ไฟล์ `.gitlab-ci.yml` ล่าสุด ณ จุดนี้

```yaml
stages:
  - lint
  - test
  - build
  - security
  - deploy-staging
  - deploy-production

variables:
  NODE_VERSION: "20"
  DIST_DIR: "dist"

default:
  image: node:${NODE_VERSION}-alpine
  cache:
    key:
      files:
        - package-lock.json
    paths:
      - node_modules/
  before_script:
    - npm ci

include:
  - template: Jobs/Code-Quality.gitlab-ci.yml

eslint_check:
  stage: lint
  script:
    - npm run lint

code_quality:
  stage: lint

unit_test:
  stage: test
  script:
    - npm test -- --coverage
  coverage: '/All files\s*\|\s*([\d.]+)/'
  artifacts:
    when: always
    reports:
      junit: reports/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/
    expire_in: 7 days

build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - $DIST_DIR/
    expire_in: 1 week
```

Commit และ push:

```bash
git add .gitlab-ci.yml
git commit -m "เพิ่ม job eslint_check และ code_quality เข้า stage lint"
git push origin main
```

ผลลัพธ์บน pipeline:

```
Pipeline #484 for main (c9e6a43)
├── lint
│   ├── eslint_check   ✔ passed (9s)
│   └── code_quality   ✔ passed (34s)
├── test
│   └── unit_test      ✔ passed (12s) — coverage: 88.24%
├── build
│   └── build          ✔ passed (17s)
├── security           (ยังไม่มี job — ข้าม)
├── deploy-staging     (ยังไม่มี job — ข้าม)
└── deploy-production  (ยังไม่มี job — ข้าม)
```

3 stage แรกทำงานครบและเรียงลำดับถูกต้องแล้ว — โค้ดต้องผ่าน lint และ test ก่อนจึงจะไปถึง build เสมอ

---

## Step 746: เขียน Stage/Job Security Scan — SAST และ Secret Detection

Stage นี้คือการนำสิ่งที่เรียนใน **Part 54: GitLab Security Features (SAST, Dependency Scanning เบื้องต้น)** มาใช้งานจริงแบบเต็มรูปแบบ พร้อมเพิ่ม Secret Detection เข้าไปด้วย เพื่อดักจับทั้งช่องโหว่ในโค้ดและ credential ที่หลุดเข้ามาโดยไม่ตั้งใจ

### 746.1 เพิ่ม Managed Template ของ Security Scanning

GitLab มี template สำเร็จรูปสำหรับ SAST และ Secret Detection ที่วิเคราะห์ทั้งภาษาโปรแกรมและรูปแบบ credential ที่รู้จักได้อัตโนมัติ โดยไม่ต้องติดตั้งเครื่องมือสแกนเอง เพิ่มเข้าไปใน `include:` ที่มีอยู่แล้ว:

```yaml
include:
  - template: Jobs/Code-Quality.gitlab-ci.yml
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
```

Template ทั้งสองจะเพิ่ม job เข้ามาให้อัตโนมัติตามภาษาที่ตรวจพบในโปรเจกต์ (สำหรับ `portfolio-project` ที่เป็น JavaScript จะได้ job `semgrep-sast` จาก SAST template) และ job `secret_detection` จาก Secret Detection template — ทั้งคู่มาพร้อม `rules:` ที่ถูกตั้งค่าไว้อย่างเหมาะสมแล้ว (เช่น รันทุกครั้งที่มีการ push หรือเปิด Merge Request)

override ให้ job เหล่านี้อยู่ใน stage `security` ตามแผนของเรา:

```yaml
semgrep-sast:
  stage: security

secret_detection:
  stage: security
```

> **ข้อควรระวังเรื่องชื่อ job:** ชื่อ job ที่มาจาก SAST template อาจแตกต่างกันไปตามภาษาที่ GitLab ตรวจพบในโปรเจกต์ (เช่น `semgrep-sast` สำหรับ JavaScript/TypeScript, `spotbugs-sast` สำหรับ Java) ให้ตรวจสอบชื่อ job จริงที่ถูกสร้างขึ้นในหน้า **CI/CD > Pipelines** หลัง push ครั้งแรก ก่อนจะ override stage ให้ตรงชื่อ

### 746.2 เพิ่ม Custom Secret Scan สำหรับ Plan ที่ไม่มี Secret Detection แบบเต็มรูปแบบ

Secret Detection แบบเต็มของ GitLab (พร้อมฐานข้อมูล pattern จำนวนมากและ deduplication) เป็นฟีเจอร์ระดับ Ultimate แต่ทีมที่ใช้ Free/Premium tier ยังสามารถสแกนหา credential ที่หลุดเข้ามาในโค้ดได้ด้วยเครื่องมือ open source เพิ่มเติม เป็นชั้นป้องกันสำรอง:

```yaml
gitleaks_scan:
  stage: security
  image: zricethezav/gitleaks:latest
  script:
    - gitleaks detect --source . --verbose --exit-code 1
  allow_failure: true
```

- **`image: zricethezav/gitleaks:latest`** — ใช้ image สำเร็จรูปของเครื่องมือ `gitleaks` ที่สแกนหารูปแบบ token/API key/private key ที่หลุดเข้ามาใน commit history และไฟล์ปัจจุบัน
- **`--exit-code 1`** — สั่งให้ gitleaks คืนค่า exit code 1 เมื่อพบ secret เพื่อให้ job fail และแจ้งเตือนทีมทันที
- **`allow_failure: true`** — ตั้งไว้เป็น warning ก่อนในช่วงแรกที่เพิ่งติดตั้ง เพื่อไม่ให้ทั้ง pipeline หยุดทำงานทันทีจาก false positive ที่อาจเกิดขึ้น เมื่อทีมมั่นใจว่าผลลัพธ์แม่นยำแล้วค่อยเปลี่ยนเป็น `allow_failure: false` เพื่อบังคับใช้จริงจัง

### 746.3 ทดสอบรัน Pipeline

```bash
git add .gitlab-ci.yml
git commit -m "เพิ่ม security scan: SAST, Secret Detection (managed template) และ gitleaks เสริม"
git push origin main
```

ผลลัพธ์:

```
Pipeline #485 for main (d1f7b54)
├── lint
│   ├── eslint_check      ✔ passed (9s)
│   └── code_quality      ✔ passed (33s)
├── test
│   └── unit_test         ✔ passed (12s) — coverage: 88.24%
├── build
│   └── build             ✔ passed (17s)
├── security
│   ├── semgrep-sast      ✔ passed (41s) — 0 vulnerabilities found
│   ├── secret_detection  ✔ passed (22s) — 0 leaked secrets found
│   └── gitleaks_scan     ✔ passed (15s) — no leaks found
├── deploy-staging        (ยังไม่มี job — ข้าม)
└── deploy-production     (ยังไม่มี job — ข้าม)
```

ผลการสแกนทั้งหมดจะปรากฏในแท็บ **Security** ของ Merge Request ทุกครั้งที่มีการเปลี่ยนแปลง ทำให้ทีมเห็นช่องโหว่ใหม่ที่อาจถูกเพิ่มเข้ามาก่อนที่จะ merge เข้า `main` เสมอ — นี่คือหลักการ **"Shift Left Security"** คือย้ายการตรวจสอบความปลอดภัยให้เกิดขึ้นเร็วที่สุดเท่าที่จะทำได้ในวงจรการพัฒนา แทนที่จะไปพบปัญหาตอน production แล้วเท่านั้น

---

## Step 747: เขียน Stage/Job Deploy to Staging — อัตโนมัติทุกครั้งที่ Merge เข้า Main

ถึงเวลาให้โค้ดที่ผ่านทุกด่านตรวจสอบแล้ว "ออกไปมีชีวิตจริง" บนเซิร์ฟเวอร์ทดสอบ — deploy ขึ้น **staging** ควรเกิดขึ้นอัตโนมัติทุกครั้งที่มีการ merge เข้า `main` โดยไม่ต้องรอใครกดปุ่ม เพื่อให้ทีม QA และ Product Owner เห็นผลงานล่าสุดได้เร็วที่สุดเสมอ

### 747.1 เตรียม CI/CD Variables สำหรับเชื่อมต่อเซิร์ฟเวอร์ staging

ก่อนเขียน job เราต้องเก็บข้อมูลสำหรับเชื่อมต่อเซิร์ฟเวอร์ไว้ใน **Settings > CI/CD > Variables** ของ GitLab แทนการเขียนลงในไฟล์โดยตรง (หลักการเดียวกับ Secrets ของ GitHub Actions ที่เรียนใน Part 69):

| Key | ประเภท | Protected | Masked | คำอธิบาย |
|---|---|---|---|---|
| `STAGING_SSH_PRIVATE_KEY` | File หรือ Variable | ✔ | ✔ | Private key สำหรับ SSH เข้าเซิร์ฟเวอร์ staging |
| `STAGING_HOST` | Variable | ✔ | ✘ | Hostname หรือ IP ของเซิร์ฟเวอร์ staging เช่น `<STAGING_SERVER_HOST>` |
| `STAGING_USER` | Variable | ✔ | ✘ | ชื่อผู้ใช้สำหรับ SSH เช่น `deploy` |

> **ข้อควรระวังเรื่องความปลอดภัย:** ห้าม hardcode private key, hostname หรือ credential ใด ๆ ลงในไฟล์ `.gitlab-ci.yml` โดยเด็ดขาด — ใช้ **CI/CD Variables** เสมอ และเปิด **Protected** ไว้เพื่อให้ค่านี้พร้อมใช้เฉพาะ pipeline ที่รันบน protected branch (เช่น `main`) เท่านั้น ป้องกันไม่ให้คนแอบเอา branch ทดลองไปดึงค่า secret ออกไปได้ ในตัวอย่างเอกสารนี้เราจะอ้างถึงค่าจริงด้วยชื่อตัวแปรเสมอ ไม่มีการใส่ URL หรือ key จริงลงในไฟล์ใด ๆ

### 747.2 เขียน Job `deploy_staging`

```yaml
deploy_staging:
  stage: deploy-staging
  image: alpine:3.20
  before_script:
    - apk add --no-cache openssh-client rsync
    - eval $(ssh-agent -s)
    - echo "$STAGING_SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan -H "$STAGING_HOST" >> ~/.ssh/known_hosts
  script:
    - rsync -avz --delete "$DIST_DIR/" "$STAGING_USER@$STAGING_HOST:/var/www/portfolio-staging/"
  environment:
    name: staging
    url: https://staging.portfolio-project.example.com
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
  needs:
    - build
```

อธิบายทีละส่วน:

- **`image: alpine:3.20`** — override image เริ่มต้นจาก `node:20-alpine` เพราะ job นี้ไม่ต้องใช้ Node.js เลย ต้องการแค่เครื่องมือ SSH/rsync ที่เบาที่สุด
- **`before_script`** — ติดตั้ง `openssh-client` และ `rsync`, เปิด `ssh-agent`, โหลด private key จากตัวแปร `$STAGING_SSH_PRIVATE_KEY` เข้าสู่ agent ด้วย `ssh-add`, แล้วเพิ่ม host key ของเซิร์ฟเวอร์ staging เข้า `known_hosts` ด้วย `ssh-keyscan` เพื่อไม่ให้ SSH ถามยืนยัน host แบบ interactive ระหว่างรัน pipeline (ซึ่งจะทำให้ job ค้าง)
- **`script: rsync`** — ซิงค์เนื้อหาทั้งหมดจากโฟลเดอร์ `dist/` (artifact ที่ได้จาก job `build` ใน Step 743) ขึ้นไปยังเซิร์ฟเวอร์ staging ด้วย `rsync -avz --delete` (ลบไฟล์ปลายทางที่ไม่มีอยู่ในต้นทางออกด้วย เพื่อให้ staging ตรงกับ `dist/` เป๊ะเสมอ ไม่มีไฟล์เก่าตกค้าง)
- **`environment: name/url`** — ประกาศว่า job นี้ deploy ไปยัง **Environment** ชื่อ `staging` พร้อม URL ที่คลิกดูได้ตรง ๆ จากหน้า GitLab (ฟีเจอร์ Environments เดียวกับที่เรียนใน Part 69 ฝั่ง GitHub Actions) ทำให้ทีมเห็นประวัติการ deploy ทั้งหมดของ environment นี้แยกจาก production ได้ชัดเจน พร้อมปุ่ม rollback ย้อนกลับไป deployment ก่อนหน้าได้จากหน้าเว็บโดยตรง
- **`rules: if $CI_COMMIT_BRANCH == "main"`** — job นี้จะรัน **เฉพาะ** เมื่อ pipeline นั้นเกิดจาก push/merge เข้า branch `main` เท่านั้น ไม่รันบน feature branch หรือ Merge Request ใด ๆ ตรงตามข้อกำหนดที่ต้อง deploy staging อัตโนมัติทุกครั้งที่ merge เข้า main
- **`needs: [build]`** — ประกาศชัดเจนว่า job นี้ต้องรอ artifact จาก job `build` เท่านั้น (ไม่ต้องรอ `security` หรือ job อื่นที่ไม่เกี่ยวข้องโดยตรง) ทำให้ GitLab สามารถจัดคิวรัน job แบบ DAG (Directed Acyclic Graph) ได้ ซึ่งเร็วกว่าการรอทุก stage ก่อนหน้าให้เสร็จตามลำดับเป๊ะ ๆ เสมอไป ตามแนวคิดขั้นสูงที่แตะไว้ใน Part 71-72

### 747.3 ทดสอบรัน Pipeline

```bash
git add .gitlab-ci.yml
git commit -m "เพิ่ม job deploy_staging: rsync ขึ้น staging อัตโนมัติทุกครั้งที่ merge เข้า main"
git push origin main
```

ผลลัพธ์:

```
Pipeline #486 for main (e2a8c65)
├── lint       ✔ (2 jobs passed)
├── test       ✔ unit_test passed — coverage: 88.24%
├── build      ✔ build passed
├── security   ✔ (3 jobs passed)
├── deploy-staging
│   └── deploy_staging   ✔ passed (8s)
│       Environment: staging
│       URL: https://staging.portfolio-project.example.com
└── deploy-production    (ยังไม่มี job — ข้าม)
```

เปิดหน้า **Operate > Environments** บน GitLab จะเห็น environment `staging` พร้อมประวัติ deployment ล่าสุดและลิงก์ตรงไปยังเว็บไซต์ทันที — ทุกครั้งที่มีการ merge เข้า `main` จากนี้ไป ทีม QA จะเห็นผลลัพธ์บน staging ได้ภายในไม่กี่นาทีโดยไม่ต้องมีใครทำอะไรเพิ่มเลย

---

## Step 748: เขียน Stage/Job Deploy to Production — ต้องมี Manual Approval

Production คือสภาพแวดล้อมที่ผู้ใช้จริงเข้าถึง ความผิดพลาดที่นี่ส่งผลกระทบโดยตรงต่อธุรกิจ ดังนั้น Pipeline ที่ดีจะไม่ deploy ขึ้น production แบบอัตโนมัติเงียบ ๆ เหมือน staging แต่ต้องมี **มนุษย์กดปุ่มยืนยัน** ก่อนเสมอ ตามหลักที่เรียนเรื่อง Environments/Required Reviewers ใน Part 69 (ฝั่ง GitHub Actions) และ `when: manual` ใน Part 71 (ฝั่ง GitLab CI/CD)

### 748.1 เตรียม CI/CD Variables สำหรับเซิร์ฟเวอร์ Production

เพิ่มตัวแปรชุดใหม่แยกจาก staging ไว้ใน **Settings > CI/CD > Variables** เช่นเดียวกับ 747.1:

| Key | Protected | Masked |
|---|---|---|
| `PRODUCTION_SSH_PRIVATE_KEY` | ✔ | ✔ |
| `PRODUCTION_HOST` | ✔ | ✘ |
| `PRODUCTION_USER` | ✔ | ✘ |

การแยกตัวแปรของ production ออกจาก staging อย่างเด็ดขาด (คนละ key กันทั้งหมด) เป็นแนวทางปฏิบัติที่ดี เพราะป้องกันไม่ให้ script ที่เขียนผิดพลาดไปเชื่อมต่อ production โดยไม่ได้ตั้งใจขณะทดสอบ staging

### 748.2 เขียน Job `deploy_production`

```yaml
deploy_production:
  stage: deploy-production
  image: alpine:3.20
  before_script:
    - apk add --no-cache openssh-client rsync
    - eval $(ssh-agent -s)
    - echo "$PRODUCTION_SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan -H "$PRODUCTION_HOST" >> ~/.ssh/known_hosts
  script:
    - rsync -avz --delete "$DIST_DIR/" "$PRODUCTION_USER@$PRODUCTION_HOST:/var/www/portfolio-production/"
  environment:
    name: production
    url: https://portfolio-project.example.com
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
  needs:
    - deploy_staging
  allow_failure: false
```

จุดที่ต่างจาก `deploy_staging` อย่างชัดเจน:

- **`rules: ... when: manual`** — แม้เงื่อนไข `if` จะเหมือนกับ staging (รันเฉพาะบน `main`) แต่เพิ่ม `when: manual` เข้าไป ทำให้ job นี้ **ไม่รันอัตโนมัติ** แม้เงื่อนไขจะเป็นจริง แต่จะไปปรากฏเป็นปุ่ม ▶️ รอการกดยืนยันในหน้า pipeline แทน — นี่คือกลไกเดียวกับ **Required Reviewers** บน GitHub Actions Environments ที่เรียนใน Part 69 เพียงแต่ GitLab ทำผ่าน `when: manual` ในไฟล์ YAML โดยตรง
- **`needs: [deploy_staging]`** — บังคับว่า production จะ deploy ได้ก็ต่อเมื่อ staging deploy สำเร็จไปก่อนแล้วเท่านั้น ป้องกันการข้ามขั้นตอนทดสอบไปสู่ production ตรง ๆ
- **`allow_failure: false`** — แม้ job จะเป็น manual แต่ถ้ามีคนกดรันแล้ว job ล้มเหลว จะทำให้ pipeline โดยรวมแสดงสถานะ failed ชัดเจน (ค่าเริ่มต้นของ manual job บางกรณีคือ `allow_failure: true` ซึ่งจะทำให้ pipeline ดูเหมือนผ่านทั้งที่ deploy จริงล้มเหลว จึงต้องกำหนดให้ชัดเจนแบบนี้เสมอสำหรับ production)

### 748.3 เพิ่มความเข้มงวดอีกขั้นด้วย Protected Environments (ทางเลือกขั้นสูง)

`when: manual` เปิดให้ **ใครก็ตามที่มีสิทธิ์ Developer ขึ้นไป** ในโปรเจกต์กดปุ่ม deploy ได้ ถ้าต้องการจำกัดให้เฉพาะบุคคลหรือกลุ่มที่กำหนดเท่านั้นที่กดปุ่มนี้ได้ (เทียบเท่า Required Reviewers ของ GitHub Actions Environments ใน Part 69 แบบเป๊ะ ๆ) ให้ตั้งค่าเพิ่มที่หน้าเว็บ:

1. เปิด **Operate > Environments** เลือก environment `production`
2. คลิก **Protect environment**
3. กำหนด **Allowed to deploy** เป็น role หรือกลุ่มเฉพาะ เช่น `Maintainers` เท่านั้น
4. (ถ้าใช้ GitLab Premium/Ultimate) เปิด **Multiple approval rules** เพื่อบังคับให้ต้องมีผู้อนุมัติมากกว่า 1 คนก่อน job จะรันได้จริง แม้จะกดปุ่ม manual แล้วก็ตาม

การตั้งค่านี้ทำให้แม้แต่คนที่กด "Run" บนปุ่ม manual job ก็ยังต้องรอการอนุมัติเพิ่มเติมจากผู้มีสิทธิ์ก่อน job จะเริ่มทำงานจริง — เป็นการป้องกันสองชั้นสำหรับสภาพแวดล้อมที่สำคัญที่สุดของระบบ

### 748.4 ทดสอบรัน Pipeline

```bash
git add .gitlab-ci.yml
git commit -m "เพิ่ม job deploy_production: deploy ขึ้น production แบบต้อง manual approval"
git push origin main
```

ผลลัพธ์บนหน้า pipeline:

```
Pipeline #487 for main (f3b9d76)
├── lint       ✔ (2 jobs passed)
├── test       ✔ unit_test passed — coverage: 88.24%
├── build      ✔ build passed
├── security   ✔ (3 jobs passed)
├── deploy-staging
│   └── deploy_staging      ✔ passed (8s)
└── deploy-production
    └── deploy_production   ⏸ manual (รอการกดยืนยัน)
```

Pipeline หยุดรอที่ job `deploy_production` โดยไม่ทำให้ pipeline fail — สถานะ **blocked** จะรอจนกว่าผู้มีสิทธิ์จะเข้ามากดปุ่ม ▶️ ที่หน้า pipeline หรือหน้า environment เมื่อทีมพร้อมปล่อยเวอร์ชันนี้ขึ้น production จริง หลังกดยืนยันแล้ว:

```
deploy_production   ✔ passed (9s)
    Environment: production
    URL: https://portfolio-project.example.com
```

นี่คือ Pipeline ที่สมบูรณ์ตามหลัก Continuous Delivery (ไม่ใช่ Continuous Deployment เต็มรูปแบบ) — โค้ดทุกอย่างพร้อม deploy ได้ตลอดเวลาที่ผ่านทุกด่าน แต่การกดปล่อยจริงยังเป็นการตัดสินใจของมนุษย์เสมอ

---

## Step 749: ตั้งค่า Notification เมื่อ Pipeline สำเร็จหรือล้มเหลว

ทีมงานต้องรู้ผลลัพธ์ของ Pipeline โดยไม่ต้องเปิดหน้า GitLab เช็คเองตลอดเวลา Step นี้จะเพิ่ม job แจ้งเตือนที่ทำงานเป็นลำดับสุดท้ายของ pipeline เสมอ ไม่ว่าผลลัพธ์ก่อนหน้าจะเป็นอย่างไร

### 749.1 เตรียม CI/CD Variable สำหรับ Webhook

ก่อนอื่นสร้างตัวแปรใหม่ใน **Settings > CI/CD > Variables**:

| Key | Protected | Masked |
|---|---|---|
| `NOTIFY_WEBHOOK_URL` | ✔ | ✔ |

`NOTIFY_WEBHOOK_URL` คือ URL ของช่องทางแจ้งเตือนที่ทีมเลือกใช้ (เช่น Incoming Webhook ของ Slack, Microsoft Teams, Discord หรือระบบแจ้งเตือนภายในองค์กร) โดยตัวไฟล์ `.gitlab-ci.yml` จะอ้างถึงค่านี้ผ่านชื่อตัวแปรเท่านั้น **ไม่มีการเขียน URL จริงลงในโค้ดที่ commit เข้า repository ที่ใดเลย** — เก็บค่าจริงไว้ในหน้า Settings ของ GitLab เพียงจุดเดียวเท่านั้น ถ้าต้องการเปลี่ยนช่องทางแจ้งเตือนในอนาคตก็แค่แก้ค่าตัวแปรนี้ที่เดียว ไม่ต้องแก้ไฟล์ pipeline เลย

> **ทางเลือกที่ง่ายกว่า:** ถ้าไม่อยากเขียน job แจ้งเตือนเอง GitLab มี **Integrations** สำเร็จรูปที่ **Settings > Integrations** สำหรับ Slack, Microsoft Teams, Discord และอื่น ๆ ที่ตั้งค่าผ่านหน้าเว็บได้เลยโดยไม่ต้องแตะไฟล์ `.gitlab-ci.yml` เลยแม้แต่บรรทัดเดียว วิธีนี้เหมาะกับทีมที่ต้องการแจ้งเตือนแบบมาตรฐานทั่วไป ส่วนวิธีเขียน job เองที่จะสอนต่อไปนี้เหมาะกับกรณีที่ต้องการปรับแต่งข้อความแจ้งเตือนให้มีรายละเอียดเฉพาะของทีม

### 749.2 เขียน Job `notify_success` และ `notify_failure`

เพิ่ม job สองตัวโดยใช้ stage พิเศษ `.post` ซึ่งเป็น stage สำรองของ GitLab ที่รันเป็นลำดับสุดท้ายเสมอโดยไม่ต้องประกาศไว้ใน `stages:`:

```yaml
notify_success:
  stage: .post
  image: alpine:3.20
  before_script:
    - apk add --no-cache curl
  script:
    - >
      curl --fail -X POST -H "Content-Type: application/json"
      --data "{\"text\":\"[สำเร็จ] Pipeline ของ $CI_PROJECT_NAME บน branch $CI_COMMIT_REF_NAME ผ่านทุกขั้นตอน (ดูรายละเอียด: $CI_PIPELINE_URL)\"}"
      "$NOTIFY_WEBHOOK_URL"
  rules:
    - when: on_success

notify_failure:
  stage: .post
  image: alpine:3.20
  before_script:
    - apk add --no-cache curl
  script:
    - >
      curl --fail -X POST -H "Content-Type: application/json"
      --data "{\"text\":\"[ล้มเหลว] Pipeline ของ $CI_PROJECT_NAME บน branch $CI_COMMIT_REF_NAME พบปัญหา กรุณาตรวจสอบ (ดูรายละเอียด: $CI_PIPELINE_URL)\"}"
      "$NOTIFY_WEBHOOK_URL"
  rules:
    - when: on_failure
```

อธิบายทีละส่วน:

- **`stage: .post`** — `.post` เป็น stage พิเศษที่ GitLab จองไว้ให้ (คู่กับ `.pre`) รันเป็นลำดับสุดท้ายเสมอโดยอัตโนมัติ ไม่ต้องประกาศไว้ในลิสต์ `stages:` ที่เราตั้งไว้ตอนต้น
- **`rules: - when: on_success`** — ค่า `on_success` หมายถึง "รัน job นี้ก็ต่อเมื่อทุก stage ก่อนหน้าผ่านหมดแล้วเท่านั้น" ซึ่งบังเอิญเป็นค่าเริ่มต้นของทุก job อยู่แล้ว แต่การเขียนไว้ชัดเจนช่วยให้คนอ่านเข้าใจเจตนาได้ทันทีว่า job นี้มีไว้แจ้งเตือนตอนสำเร็จเท่านั้น
- **`rules: - when: on_failure`** — ตรงกันข้ามกับ `on_success` คือรัน job นี้ **เฉพาะเมื่อมี job ใดก่อนหน้าล้มเหลว** เท่านั้น ทำให้ทีมได้รับข้อความแจ้งเตือนที่ต่างกันชัดเจนระหว่างสองสถานการณ์
- **`$CI_PROJECT_NAME`, `$CI_COMMIT_REF_NAME`, `$CI_PIPELINE_URL`** — เป็นตัวแปรที่ GitLab เตรียมไว้ให้อัตโนมัติในทุก pipeline (Predefined CI/CD Variables) ไม่ต้องตั้งค่าเอง ใช้แทรกลงในข้อความแจ้งเตือนเพื่อให้ทีมคลิกดูรายละเอียดของ pipeline ที่มีปัญหาได้ทันทีจากข้อความแจ้งเตือน
- **`curl --fail`** — ถ้า webhook ตอบกลับด้วยสถานะ error (4xx/5xx) ให้ `curl` คืน exit code ที่ไม่ใช่ 0 ทำให้ job นี้เห็นว่า "ส่งแจ้งเตือนไม่สำเร็จ" ชัดเจน แทนที่จะแสดงว่า job ผ่านทั้งที่ข้อความไม่ได้ถูกส่งออกไปจริง

> **หมายเหตุเรื่องความปลอดภัย:** สังเกตว่าตลอดทั้ง job ไม่มีการเขียน URL จริงของ webhook ลงในไฟล์เลยแม้แต่ตัวอย่างเดียว ใช้ผ่านตัวแปร `$NOTIFY_WEBHOOK_URL` เท่านั้น — นี่คือหลักการเดียวกับการจัดการ Secrets ที่เรียนตลอดทั้งเฟส 7 ไม่ว่าจะเป็น GitHub Actions Secrets (Part 69) หรือ GitLab CI/CD Variables (Part 71): **ค่าลับต้องไม่ปรากฏในซอร์สโค้ดที่อยู่ใน repository โดยเด็ดขาด ไม่ว่าจะเป็นไฟล์ pipeline, commit history หรือ log ใด ๆ**

### 749.3 ทดสอบรัน Pipeline ทั้งสองกรณี

Commit และ push job แจ้งเตือน:

```bash
git add .gitlab-ci.yml
git commit -m "เพิ่ม notify_success/notify_failure: แจ้งเตือนผลลัพธ์ pipeline ผ่าน webhook"
git push origin main
```

กรณี pipeline ผ่านทั้งหมด:

```
Pipeline #488 for main (a4c1e87) — passed
├── lint, test, build, security, deploy-staging  ✔ ทั้งหมดผ่าน
├── deploy-production  ⏸ manual (รอการกดยืนยัน)
└── .post
    └── notify_success   ✔ passed (2s)
```

ลองจำลองกรณี test ล้มเหลว (แก้โค้ดให้ผิดโดยตั้งใจเพื่อทดสอบ) เพื่อดูว่า `notify_failure` ทำงานถูกต้อง:

```
Pipeline #489 for main (b5d2f98) — failed
├── lint      ✔ passed
├── test
│   └── unit_test   ✘ failed (10s)
└── .post
    └── notify_failure   ✔ passed (2s)
```

สังเกตว่า stage `build`, `security`, `deploy-staging`, `deploy-production` ไม่ถูกรันเลยเพราะ `unit_test` fail ก่อน (fail fast ทำงานตามที่ออกแบบไว้) แต่ `.post` ยังคงรัน `notify_failure` เสมอเพื่อแจ้งให้ทีมรู้ทันที — นี่คือพฤติกรรมที่ถูกต้องตามที่ต้องการทุกประการ

---

## Step 750: สรุปทบทวนภาพรวมเฟส 7 ทั้งหมด (Part 66–75) พร้อม Cheat Sheet และ Checklist ก่อนเข้าสู่เฟส 8

ยินดีด้วย — คุณเพิ่งสร้าง CI/CD Pipeline แบบเต็มรูปแบบให้กับโปรเจกต์จริงตั้งแต่ lint ไปจนถึง production ครบทุกขั้นตอน ก่อนไปเฟส 8 มาดูไฟล์ `.gitlab-ci.yml` ฉบับสมบูรณ์ทั้งหมด แล้วทบทวนภาพรวมทุกอย่างที่เรียนมาตลอดเฟส 7

### 750.1 ไฟล์ `.gitlab-ci.yml` ฉบับสมบูรณ์

```yaml
# .gitlab-ci.yml ของ portfolio-project
# Pipeline ครบวงจร: lint -> test -> build -> security -> deploy-staging -> deploy-production

stages:
  - lint
  - test
  - build
  - security
  - deploy-staging
  - deploy-production

variables:
  NODE_VERSION: "20"
  DIST_DIR: "dist"

default:
  image: node:${NODE_VERSION}-alpine
  cache:
    key:
      files:
        - package-lock.json
    paths:
      - node_modules/
  before_script:
    - npm ci

include:
  - template: Jobs/Code-Quality.gitlab-ci.yml
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml

# ---------- lint ----------

eslint_check:
  stage: lint
  script:
    - npm run lint

code_quality:
  stage: lint

# ---------- test ----------

unit_test:
  stage: test
  script:
    - npm test -- --coverage
  coverage: '/All files\s*\|\s*([\d.]+)/'
  artifacts:
    when: always
    reports:
      junit: reports/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/
    expire_in: 7 days

# ---------- build ----------

build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - $DIST_DIR/
    expire_in: 1 week

# ---------- security ----------

semgrep-sast:
  stage: security

secret_detection:
  stage: security

gitleaks_scan:
  stage: security
  image: zricethezav/gitleaks:latest
  script:
    - gitleaks detect --source . --verbose --exit-code 1
  allow_failure: true

# ---------- deploy-staging ----------

deploy_staging:
  stage: deploy-staging
  image: alpine:3.20
  before_script:
    - apk add --no-cache openssh-client rsync
    - eval $(ssh-agent -s)
    - echo "$STAGING_SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan -H "$STAGING_HOST" >> ~/.ssh/known_hosts
  script:
    - rsync -avz --delete "$DIST_DIR/" "$STAGING_USER@$STAGING_HOST:/var/www/portfolio-staging/"
  environment:
    name: staging
    url: https://staging.portfolio-project.example.com
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
  needs:
    - build

# ---------- deploy-production ----------

deploy_production:
  stage: deploy-production
  image: alpine:3.20
  before_script:
    - apk add --no-cache openssh-client rsync
    - eval $(ssh-agent -s)
    - echo "$PRODUCTION_SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan -H "$PRODUCTION_HOST" >> ~/.ssh/known_hosts
  script:
    - rsync -avz --delete "$DIST_DIR/" "$PRODUCTION_USER@$PRODUCTION_HOST:/var/www/portfolio-production/"
  environment:
    name: production
    url: https://portfolio-project.example.com
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
  needs:
    - deploy_staging
  allow_failure: false

# ---------- แจ้งเตือนผลลัพธ์ (รันเป็นลำดับสุดท้ายเสมอ) ----------

notify_success:
  stage: .post
  image: alpine:3.20
  before_script:
    - apk add --no-cache curl
  script:
    - >
      curl --fail -X POST -H "Content-Type: application/json"
      --data "{\"text\":\"[สำเร็จ] Pipeline ของ $CI_PROJECT_NAME บน branch $CI_COMMIT_REF_NAME ผ่านทุกขั้นตอน (ดูรายละเอียด: $CI_PIPELINE_URL)\"}"
      "$NOTIFY_WEBHOOK_URL"
  rules:
    - when: on_success

notify_failure:
  stage: .post
  image: alpine:3.20
  before_script:
    - apk add --no-cache curl
  script:
    - >
      curl --fail -X POST -H "Content-Type: application/json"
      --data "{\"text\":\"[ล้มเหลว] Pipeline ของ $CI_PROJECT_NAME บน branch $CI_COMMIT_REF_NAME พบปัญหา กรุณาตรวจสอบ (ดูรายละเอียด: $CI_PIPELINE_URL)\"}"
      "$NOTIFY_WEBHOOK_URL"
  rules:
    - when: on_failure
```

ลองอ่านไฟล์นี้ทีละ job ด้วยตัวเอง — ถ้าคุณอธิบายได้ว่าแต่ละ job ทำอะไร ทำไมต้องอยู่ stage นั้น และทำไมต้องมี `rules`/`needs`/`artifacts` แบบนั้น แปลว่าคุณเข้าใจการสร้าง CI/CD Pipeline อย่างแท้จริงแล้ว ไม่ใช่แค่ copy โค้ดมาวาง

### 750.2 Cheat Sheet รวมแนวคิดและคำสั่งทั้งหมดของเฟส 7 (Part 66–75)

| Part | หัวข้อ | แนวคิด/คำสั่งหลัก |
|---|---|---|
| 66 | CI/CD คืออะไร | Continuous Integration, Continuous Delivery/Deployment, ประโยชน์ของการ automate |
| 67 | GitHub Actions เบื้องต้น | `on:`, `jobs:`, `steps:`, `runs-on:`, `uses: actions/checkout@v4` |
| 68 | Jobs, Steps, Matrix Build | `strategy: matrix`, `needs:` ระหว่าง job |
| 69 | Secrets, Environments, Artifacts | `secrets.<NAME>`, `environment:` + required reviewers, `actions/upload-artifact` |
| 70 | Custom Actions, Reusable Workflows | Composite Action, Docker Action, `workflow_call` |
| 71 | GitLab CI/CD ขั้นสูง | `stages:`, `cache:`, `artifacts:`, `rules:`, `when: manual` |
| 72 | Multi-project, Dynamic Pipelines | `trigger:`, downstream/upstream pipeline, generate `.gitlab-ci.yml` แบบ dynamic |
| 73 | Automated Testing | Unit/Integration/E2E, `coverage:` regex, `artifacts: reports: junit/coverage_report` |
| 74 | Deploy อัตโนมัติ | Docker build/push, `kubectl apply`, Blue-Green/Canary Deployment |
| 75 | Capstone (Part นี้) | รวมทุกอย่างข้างต้นในโปรเจกต์จริงตั้งแต่ lint ถึง production |

### 750.3 ตารางสรุป Syntax ที่ใช้บ่อยที่สุดใน `.gitlab-ci.yml`

| Syntax | ความหมาย |
|---|---|
| `stages:` | ประกาศลำดับ stage ทั้งหมดของ pipeline |
| `default:` | ค่าตั้งต้น (image, cache, before_script) ที่ทุก job ใช้ร่วมกัน |
| `cache: key.files / paths` | เก็บ dependency ไว้ใช้ซ้ำข้าม pipeline เพื่อความเร็ว |
| `artifacts: paths` | ส่งไฟล์ผลลัพธ์จาก job หนึ่งไปให้ job ถัดไปใช้ต่อ |
| `artifacts: reports: junit` | แสดงผล unit test แบบละเอียดในแท็บ Tests |
| `artifacts: reports: coverage_report` | แสดง code coverage แบบ inline ในหน้า diff ของ MR |
| `coverage:` (regex) | ดึงตัวเลข coverage จาก log มาแสดงบนหน้า pipeline |
| `include: - template:` | ดึง managed template สำเร็จรูปของ GitLab (Code Quality, SAST, Secret Detection) |
| `rules: - if:` | เงื่อนไขว่า job จะรันเมื่อไหร่ (เช่น เฉพาะ branch `main`) |
| `rules: - when: manual` | ต้องมีคนกดยืนยันก่อน job จะเริ่มทำงาน |
| `rules: - when: on_success / on_failure` | รัน job ตามผลลัพธ์ของ stage ก่อนหน้า |
| `needs:` | กำหนด dependency ตรง ๆ ระหว่าง job เพื่อให้รันแบบ DAG ได้เร็วขึ้น |
| `environment: name/url` | ผูก job เข้ากับ environment พร้อมประวัติ deploy และปุ่ม rollback |
| `stage: .post` | Stage สำรองที่รันเป็นลำดับสุดท้ายเสมอ ไม่ต้องประกาศใน `stages:` |

### 750.4 Checklist ทบทวนภาพรวมเฟส 7 ทั้งหมด (Part 66–75) ก่อนเข้าสู่เฟส 8

ก่อนไปต่อ Part 76 (เริ่มต้นเฟส 8: DevOps/Security/Compliance) ให้ตรวจสอบตัวเองอย่างละเอียดตามรายการนี้ ถ้าข้อไหนยังไม่มั่นใจ แนะนำให้ย้อนกลับไปอ่าน Part ที่เกี่ยวข้องอีกครั้งก่อน:

**แนวคิดพื้นฐานของ CI/CD**
- [ ] อธิบายความแตกต่างระหว่าง Continuous Integration, Continuous Delivery และ Continuous Deployment ได้ชัดเจน
- [ ] อธิบายได้ว่าทำไมหลักการ "fail fast" ถึงสำคัญ และทำไมต้องจัดลำดับ stage อย่าง lint/test ก่อน build/deploy

**GitHub Actions**
- [ ] เขียน workflow YAML พื้นฐานได้ตั้งแต่ `on:`, `jobs:`, `steps:` โดยไม่ต้องเปิดเอกสารดู
- [ ] ใช้ `strategy: matrix` ทดสอบโค้ดบนหลายเวอร์ชัน/หลาย OS พร้อมกันได้
- [ ] จัดการ Secrets และตั้งค่า Environment พร้อม Required Reviewers ได้
- [ ] เขียน Composite Action หรือ Reusable Workflow เพื่อลดโค้ดซ้ำซ้อนข้าม repository ได้

**GitLab CI/CD**
- [ ] เขียน `.gitlab-ci.yml` ที่มีหลาย stage พร้อม cache และ artifact ได้อย่างถูก syntax
- [ ] ใช้ `rules:` ควบคุมเงื่อนไขการรันของแต่ละ job ได้อย่างแม่นยำ
- [ ] เข้าใจความแตกต่างระหว่าง Multi-project Pipeline กับ Dynamic Pipeline และรู้ว่าแต่ละแบบเหมาะกับสถานการณ์ไหน

**Automated Testing และ Security**
- [ ] ตั้งค่า Pipeline ให้รัน unit test พร้อมสร้าง coverage report ที่ทั้งอ่านง่ายบน UI และแสดงผลแบบ inline ในหน้า diff ได้
- [ ] เข้าใจความแตกต่างของ Unit, Integration และ E2E test และรู้ว่าแต่ละแบบควรอยู่ตรงไหนของ pipeline
- [ ] เพิ่ม SAST และ Secret Detection เข้า Pipeline ได้ และเข้าใจหลักการ Shift Left Security

**Deploy อัตโนมัติ**
- [ ] แยกความแตกต่างระหว่าง deploy staging (อัตโนมัติ) กับ deploy production (ต้องอนุมัติ) ได้ และอธิบายเหตุผลได้ชัดเจน
- [ ] ใช้ `when: manual` (GitLab) หรือ Environment + Required Reviewers (GitHub Actions) สร้าง manual approval gate ได้
- [ ] จัดการ Secrets/Variables ของแต่ละ environment แยกจากกันอย่างปลอดภัย ไม่มี credential หลุดเข้าไปในซอร์สโค้ดเด็ดขาด

**ภาพรวมโปรเจกต์**
- [ ] สร้าง Pipeline ครบวงจรตั้งแต่ lint ถึง production ให้โปรเจกต์จริงได้ด้วยตัวเองทั้งหมด
- [ ] อ่านและอธิบายไฟล์ `.gitlab-ci.yml` ที่มีหลาย stage, `rules`, `needs`, `artifacts` ปนกันได้เข้าใจทุกจุด
- [ ] ตั้งค่าระบบแจ้งเตือนผลลัพธ์ของ Pipeline ให้ทีมรับรู้ได้ทันทีโดยไม่ต้องเข้าไปเช็คเอง

ถ้าคุณติ๊กครบทุกข้อ (หรือเกือบครบ) แปลว่าคุณพร้อมสำหรับเฟส 8 อย่างแท้จริงแล้ว — เฟส 7 คือเฟสที่เปลี่ยนคุณจากคนที่ "เขียนโค้ดแล้ว push ขึ้น repository" ไปสู่คนที่ "ส่งมอบซอฟต์แวร์อย่างเป็นระบบและปลอดภัยได้ด้วยตัวเอง" ซึ่งเป็นทักษะแกนกลางของวิศวกร DevOps ในทุกองค์กร

---

## สรุป Part 75

ใน Part นี้เราได้สร้าง CI/CD Pipeline แบบเต็มรูปแบบให้กับโปรเจกต์ `portfolio-project` จริง โดยนำทุกทักษะที่เรียนมาตลอดเฟส 7 มาใช้งานร่วมกันในภารกิจเดียว:

1. เตรียมโปรเจกต์ให้มี toolchain จริง (eslint, jest, build script) เพื่อให้ทุก stage มีของจริงให้ทำงาน (Step 741)
2. วางแผนโครงสร้าง Pipeline ทั้ง 6 stage และตั้งค่าร่วมทั้งไฟล์ด้วย `stages`, `variables`, `default`, `cache` (Step 742)
3. เขียน job `build` เพื่อ bundle โปรเจกต์และเก็บผลลัพธ์เป็น artifact ตามหลัก build-once-deploy-many (Step 743)
4. เขียน job `unit_test` พร้อม coverage report ทั้งแบบ regex และแบบ Cobertura ตามที่เรียนใน Part 73 (Step 744)
5. เขียน job `eslint_check` และดึง managed template `code_quality` เข้า stage lint (Step 745)
6. เพิ่ม security scan ด้วย SAST, Secret Detection และ gitleaks ตามหลักที่เรียนใน Part 54 (Step 746)
7. เขียน job `deploy_staging` ที่ deploy อัตโนมัติทุกครั้งที่ merge เข้า main (Step 747)
8. เขียน job `deploy_production` ที่ต้องมี manual approval ก่อน deploy เสมอ ตามที่เรียนใน Part 69/71 (Step 748)
9. ตั้งค่า job แจ้งเตือน `notify_success`/`notify_failure` ผ่าน webhook โดยใช้ CI/CD Variable แทนการ hardcode URL (Step 749)
10. ทบทวนภาพรวมแนวคิดและ syntax ทั้งหมดของเฟส 7 ผ่าน Cheat Sheet และ Checklist ครบทุกด้าน (Step 750)

Part นี้คือจุดปิดฉากของ **เฟส 7: CI/CD (Part 66–75, Step 651–750)** อย่างสมบูรณ์ คุณได้ผ่านการฝึกฝนทักษะที่แยกวิศวกรที่ "เขียนโค้ดเก่ง" ออกจากวิศวกรที่ "ส่งมอบซอฟต์แวร์อย่างเป็นระบบได้จริง" มาครบทุกมิติแล้ว ตั้งแต่การเข้าใจแนวคิด CI/CD, การใช้งาน GitHub Actions และ GitLab CI/CD ทั้งพื้นฐานและขั้นสูง, การทดสอบอัตโนมัติพร้อมวัด coverage, การสแกนความปลอดภัยของโค้ด ไปจนถึงการ deploy จริงแบบมีขั้นตอนอนุมัติที่รัดกุม

จากนี้ไป หลักสูตรจะพาคุณเข้าสู่ **เฟส 8: DevOps, Security, Compliance ระดับองค์กร (Part 76–85, Step 751–850)** ซึ่งจะขยายขอบเขตจากการส่งมอบโค้ดไปสู่การดูแลระบบทั้งวงจรชีวิต ตั้งแต่แนวคิด GitOps, Infrastructure as Code, ไปจนถึงมาตรฐานความปลอดภัยและการปฏิบัติตามข้อกำหนดที่องค์กรขนาดใหญ่ต้องการ

**ต่อไป:** [Part 76: GitOps: แนวคิดและการประยุกต์ใช้งานจริง](./part-076-gitops-แนวคิด.md)
