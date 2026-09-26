# Part 35: การตั้งชื่อ Branch และ Commit Message Convention (Conventional Commits)

> **Step ในหลักสูตรนี้:** Step 341–350
> **เฟส:** 4 — ทำงานเป็นทีมด้วย Workflow มาตรฐาน (Git Flow, GitHub Flow, Rebase)
> **เป้าหมายของ Part นี้:** เข้าใจว่าทำไมทีมพัฒนาซอฟต์แวร์ที่เป็นมืออาชีพต้องมีมาตรฐานการตั้งชื่อ branch และเขียน commit message ที่ชัดเจน สามารถออกแบบ branch naming convention ที่ใช้งานได้จริง เขียน commit message ตามสเปก Conventional Commits ได้อย่างถูกต้องแม่นยำ เข้าใจ breaking change และผลกระทบต่อ semantic versioning และรู้จักเครื่องมือที่ช่วยบังคับใช้มาตรฐานเหล่านี้ในทีมอย่างอัตโนมัติ

---

## สารบัญของ Part นี้

- Step 341: ทำไมต้องมีมาตรฐานตั้งชื่อ branch/commit ในทีม
- Step 342: Branch naming convention มาตรฐาน (`feature/`, `bugfix/`, `hotfix/`, `release/`, `chore/`, `docs/`)
- Step 343: การใส่ ticket/issue number ใน branch name
- Step 344: Conventional Commits specification เต็มรูปแบบ
- Step 345: Commit types มาตรฐานทั้งหมดพร้อมความหมายและตัวอย่าง
- Step 346: Breaking changes ใน Conventional Commits
- Step 347: ประโยชน์ของ Conventional Commits (changelog, semantic-release)
- Step 348: เครื่องมือช่วยบังคับใช้มาตรฐาน (commitlint, husky)
- Step 349: ตัวอย่าง commit message ที่ดีและแย่เทียบกันแบบละเอียด
- Step 350: แบบฝึกหัด — เขียน commit ตาม Conventional Commits 10 สถานการณ์

---

## Step 341: ทำไมต้องมีมาตรฐานตั้งชื่อ branch/commit ในทีม

เมื่อคุณทำงานคนเดียว การตั้งชื่อ branch ว่า `fix`, `test1`, `my-branch` หรือเขียน commit message ว่า `"แก้บั๊ก"`, `"update"`, `"wip"` อาจจะดูไม่ใช่ปัญหาใหญ่ เพราะคุณจำได้เองว่าทำอะไรไป แต่พอทีมโตขึ้นจาก 1 คนเป็น 5 คน 20 คน หรือ 200 คน สิ่งเหล่านี้จะกลายเป็น **หายนะที่มองไม่เห็นในวันแรก แต่ทำลายประสิทธิภาพทีมอย่างมหาศาลในระยะยาว**

### ปัญหาจริงที่เกิดขึ้นเมื่อไม่มีมาตรฐาน

ลองนึกภาพ repository ที่มี branch แบบนี้อยู่พร้อมกัน 50 branch:

```
fix
fix2
fix-final
new-feature
test
testing-something
john-branch
patch
patch-v2
อันนี้แก้ปัญหา-login
temp
temp2
asdasd
```

และมี commit log แบบนี้:

```
* a1b2c3d wip
* e4f5g6h fix
* h7i8j9k more fix
* k1l2m3n update stuff
* n4o5p6q asdf
* q7r8s9t final fix จริงๆ
* t1u2v3w แก้ตามที่พี่บอก
```

ถ้าคุณเป็นสมาชิกใหม่ที่เพิ่งเข้าทีม หรือแม้แต่เป็นคนเก่าที่กลับมาดูโค้ดหลังจากผ่านไป 3 เดือน คุณจะตอบคำถามพวกนี้ไม่ได้เลย:

- Branch ไหนคือของ feature ไหน ใครเป็นคนทำ เกี่ยวกับ ticket ไหน
- Commit `q7r8s9t` แก้ปัญหาอะไรกันแน่ ทำไมถึงเรียกว่า "final fix จริงๆ" (แปลว่ามี fix ที่ไม่ final มาก่อนกี่รอบ)
- ถ้าอยาก revert เฉพาะ feature หนึ่ง จะรู้ได้อย่างไรว่า commit ไหนบ้างที่เกี่ยวข้อง
- ถ้าอยากเขียน Release Notes ให้ลูกค้า จะต้องนั่งไล่อ่านทุก commit ทีละอันเพื่อเดาว่าอันไหนคือ feature ใหม่ อันไหนคือ bug fix

### ผลกระทบที่เกิดขึ้นจริงในทีม

| ปัญหา | ผลกระทบที่ตามมา |
|---|---|
| ชื่อ branch ไม่สื่อความหมาย | หาของยาก เสียเวลาค้นหาว่า branch ไหนทำอะไร |
| ไม่มีเลข ticket ผูกกับ branch | ตรวจสอบย้อนกลับไม่ได้ว่า branch นี้แก้ requirement ข้อไหน |
| Commit message กำกวม เช่น "fix", "update" | ไล่ debug ด้วย `git log` หรือ `git blame` แล้วไม่ได้ข้อมูลอะไรเลย |
| ไม่มีรูปแบบที่แน่นอน | เขียน script อัตโนมัติ (auto-changelog, auto-versioning) ไม่ได้ เพราะ parse ข้อความไม่ได้ |
| แต่ละคนตั้งชื่อคนละแบบ | Code review สับสน ไม่รู้ scope ของการเปลี่ยนแปลงจากชื่อ branch/PR |
| ไม่มีมาตรฐานเรื่อง breaking change | อัปเดต dependency แล้วโค้ดพังโดยไม่มีการเตือนล่วงหน้า |

### มาตรฐานคือ "สัญญาที่ทุกคนตกลงร่วมกัน"

หัวใจของการมีมาตรฐานคือ การทำให้ **มนุษย์ทุกคนในทีม และเครื่องมืออัตโนมัติทุกตัว** สามารถ "อ่าน" และ "เข้าใจ" ข้อมูลใน Git history ได้แบบเดียวกัน โดยไม่ต้องถามใคร ไม่ต้องเดา

พูดง่าย ๆ มาตรฐานการตั้งชื่อ branch และ commit message ทำหน้าที่เหมือน **ไวยากรณ์ของภาษา** — ถ้าทุกคนพูดคนละภาษา สื่อสารกันไม่ได้ แต่ถ้าทุกคนใช้ไวยากรณ์เดียวกัน แม้จะพูดเรื่องต่างกัน ก็ยังเข้าใจกันได้ทันที

### ประโยชน์ของการมีมาตรฐานที่ชัดเจน

1. **ค้นหาและกรองข้อมูลได้ง่าย** — `git branch --list "feature/*"` เพื่อดู feature ที่กำลังพัฒนาทั้งหมด หรือ `git log --grep="^fix"` เพื่อดู commit ที่เป็นการแก้บั๊กทั้งหมด
2. **สื่อสารเจตนาได้ทันทีโดยไม่ต้องเปิดโค้ดอ่าน** — เห็นชื่อ branch หรือ commit ก็รู้ทันทีว่ากำลังทำอะไรอยู่
3. **เชื่อมโยงกับระบบ ticket/issue ได้อัตโนมัติ** — เครื่องมือ เช่น Jira, Linear, GitHub Issues สามารถ auto-link branch กับ ticket ได้จากชื่อที่มีรูปแบบแน่นอน
4. **สร้าง Changelog และกำหนดเวอร์ชันอัตโนมัติได้** — เมื่อ commit message มีรูปแบบที่ machine อ่านได้ เครื่องมือเช่น `semantic-release` จะรู้เองว่าควรออกเวอร์ชัน patch, minor หรือ major
5. **ลดความขัดแย้งในทีม** — ทุกคนรู้กฎเดียวกัน ไม่ต้องเถียงกันว่า "ควรตั้งชื่อ branch แบบไหน" ทุกครั้งที่เริ่มงานใหม่
6. **ทำให้ Code Review มีบริบท** — Reviewer เห็นชื่อ PR/branch/commit ที่ชัดเจนก็เข้าใจ scope การเปลี่ยนแปลงได้เร็วขึ้นมาก

ใน Part นี้เราจะวางมาตรฐานทั้งสองด้าน คือ **Branch Naming Convention** (Step 342–343) และ **Conventional Commits** (Step 344–350) ซึ่งเป็นมาตรฐานที่ได้รับความนิยมสูงสุดในอุตสาหกรรมซอฟต์แวร์ทั่วโลกในปัจจุบัน

---

## Step 342: Branch naming convention มาตรฐาน (`feature/`, `bugfix/`, `hotfix/`, `release/`, `chore/`, `docs/`)

รูปแบบที่ได้รับความนิยมมากที่สุดคือการใช้ **prefix (คำนำหน้า) ตามด้วยเครื่องหมาย `/` แล้วตามด้วยชื่อสั้น ๆ ที่สื่อความหมาย** ในรูปแบบทั่วไปคือ:

```
<type>/<short-description>
```

### ตาราง prefix มาตรฐานที่ใช้กันทั่วไปในอุตสาหกรรม

| Prefix | ใช้เมื่อไร | ตัวอย่าง |
|---|---|---|
| `feature/` | พัฒนาฟีเจอร์ใหม่ | `feature/user-login`, `feature/dark-mode` |
| `bugfix/` | แก้บั๊กที่พบระหว่างการพัฒนา (ยังไม่ถึงมือลูกค้า หรือพบใน branch พัฒนา) | `bugfix/cart-total-wrong`, `bugfix/broken-pagination` |
| `hotfix/` | แก้บั๊กด่วนบน production ที่ต้องปล่อยทันทีโดยไม่รอรอบ release ปกติ | `hotfix/payment-gateway-down`, `hotfix/security-patch-cve-2024` |
| `release/` | เตรียม branch สำหรับปล่อยเวอร์ชันใหม่ (stabilize ก่อนขึ้น production) | `release/1.4.0`, `release/2024-q1` |
| `chore/` | งานดูแลระบบที่ไม่ใช่ feature/bug โดยตรง เช่น อัปเดต dependency, ปรับ config, cleanup | `chore/update-dependencies`, `chore/cleanup-unused-imports` |
| `docs/` | แก้ไขหรือเพิ่มเอกสารเท่านั้น ไม่กระทบโค้ดที่รันจริง | `docs/update-readme`, `docs/api-reference` |

นอกจากนี้ยังมี prefix เสริมที่หลายทีมนิยมเพิ่มเข้ามาด้วย:

| Prefix เสริม | ใช้เมื่อไร | ตัวอย่าง |
|---|---|---|
| `refactor/` | ปรับโครงสร้างโค้ดโดยไม่เปลี่ยน behavior ภายนอก | `refactor/extract-payment-service` |
| `test/` | เพิ่มหรือแก้ไข test เท่านั้น | `test/add-checkout-e2e` |
| `experiment/` หรือ `spike/` | ทดลองแนวทางใหม่ที่ยังไม่แน่ใจว่าจะใช้จริงหรือไม่ | `experiment/try-new-cache-layer` |
| `ci/` | แก้ไข pipeline หรือ workflow ของ CI/CD | `ci/fix-deploy-workflow` |

### กฎการเขียนส่วน `<short-description>`

1. **ใช้ตัวพิมพ์เล็กทั้งหมด (lowercase)** — เพื่อหลีกเลี่ยงปัญหาระบบไฟล์บางระบบที่ case-sensitive ต่างกัน และเพื่อความสม่ำเสมอ
2. **ใช้ขีดกลาง (`-`) คั่นคำ ไม่ใช้ underscore (`_`) หรือช่องว่าง** — เพราะ URL และหลายเครื่องมือจัดการขีดกลางได้ดีกว่า
3. **สั้น กระชับ แต่สื่อความหมาย** — ควรอยู่ที่ประมาณ 3–6 คำ ไม่ควรยาวเกิน 50 ตัวอักษร
4. **ห้ามใช้ภาษาไทยหรืออักขระพิเศษ** — เพราะบางเครื่องมือ, บาง terminal, หรือบาง OS อาจแสดงผลผิดเพี้ยนหรือมีปัญหา encoding
5. **หลีกเลี่ยงชื่อคน** — เช่น `john-fix-bug` เพราะทำให้ไม่รู้ว่า branch นี้ทำอะไรถ้าไม่รู้ว่า John คือใคร ควรอธิบายที่ "งาน" ไม่ใช่ "คน"

### ตัวอย่างเปรียบเทียบ ชื่อ branch ที่ไม่ดี vs ดี

| ไม่ดี | ดี | เหตุผล |
|---|---|---|
| `fix` | `bugfix/cart-total-calculation-wrong` | บอกได้ว่าคือการแก้บั๊กเรื่องอะไร |
| `new-stuff` | `feature/export-report-to-pdf` | บอกได้ว่าเป็นฟีเจอร์อะไร |
| `john_branch` | `feature/add-two-factor-auth` | ไม่ผูกกับชื่อคน สื่อความหมายของงาน |
| `Test-Branch-FINAL` | `test/add-checkout-integration-tests` | lowercase, ไม่มีคำกำกวมอย่าง "FINAL" |
| `เร่งด่วนมาก` | `hotfix/payment-timeout-production` | ใช้ภาษาอังกฤษ, สื่อความหมายชัดเจน |

### ตัวอย่างคำสั่งจริง

```bash
# สร้าง feature branch ใหม่จาก main
git checkout -b feature/user-profile-avatar main

# สร้าง bugfix branch จาก develop
git checkout -b bugfix/date-format-thai-locale develop

# สร้าง hotfix branch จาก main (ต้องรีบแก้ทันที)
git checkout -b hotfix/checkout-crash-production main

# สร้าง release branch เพื่อเตรียมปล่อยเวอร์ชัน
git checkout -b release/2.3.0 develop

# สร้าง chore branch สำหรับงานดูแลระบบ
git checkout -b chore/upgrade-node-to-v20 main
```

### ทำไมต้องแยก `bugfix/` กับ `hotfix/` ออกจากกัน

หลายคนสับสนว่าสองอันนี้ต่างกันอย่างไร ความแตกต่างหลักคือ **ความเร่งด่วนและจุดที่แตก branch ออกมา**:

- **`bugfix/`** มักแตกออกมาจาก branch พัฒนา (เช่น `develop`) เพื่อแก้บั๊กที่พบระหว่างการพัฒนาปกติ ยังไม่ได้ถูกปล่อยไปถึงมือผู้ใช้จริง ไม่มีความเร่งด่วนสูงมาก รอ merge เข้ารอบ release ปกติได้
- **`hotfix/`** มักแตกออกมาจาก branch production (เช่น `main`) โดยตรง เพราะเป็นบั๊กร้ายแรงที่กระทบผู้ใช้งานจริงอยู่ ต้องแก้และปล่อยด่วนที่สุด แล้วค่อย merge กลับเข้าทั้ง `main` และ `develop` เพื่อไม่ให้บั๊กเดิมโผล่กลับมาอีกใน release ถัดไป

เราจะพูดถึงรายละเอียดของ workflow ทั้งสองแบบนี้ในเชิงลึกเมื่อเรียนเรื่อง **Git Flow** ใน Part ถัดไปของเฟสนี้

---

## Step 343: การใส่ ticket/issue number ใน branch name (เช่น `feature/JIRA-123-add-login`)

การมี prefix อย่างเดียวยังไม่เพียงพอสำหรับทีมที่ใช้ระบบติดตามงาน (issue tracker) เช่น Jira, Linear, GitHub Issues, Asana เพราะเราต้องการ **เชื่อมโยง branch เข้ากับ ticket/issue ที่มันกำลังแก้ปัญหาให้** เพื่อให้ตรวจสอบย้อนกลับได้เสมอว่า "โค้ดส่วนนี้เขียนขึ้นมาเพื่อตอบโจทย์อะไร"

### รูปแบบมาตรฐาน

```
<type>/<TICKET-ID>-<short-description>
```

### ตัวอย่างตามระบบ ticket ต่าง ๆ

| ระบบ Ticket | รูปแบบ ID | ตัวอย่างชื่อ branch |
|---|---|---|
| Jira | `PROJECT-123` | `feature/JIRA-123-add-login`, `bugfix/PROJ-456-fix-null-pointer` |
| Linear | `ENG-123` | `feature/ENG-241-add-dark-mode` |
| GitHub Issues | `#123` (มักตัด `#` ออกเพราะ `#` มีปัญหากับบาง shell) | `feature/123-add-login`, `bugfix/gh-456-fix-crash` |
| Asana / Trello | เลข task ID | `feature/task-9081-redesign-navbar` |
| ไม่มีระบบ ticket | ใช้เลขที่ทีมกำหนดเองหรือไม่ใส่เลยก็ได้ | `feature/add-login` |

### ตัวอย่างจริงแบบละเอียด

```bash
# Jira ticket PROJ-123 ให้เพิ่มระบบ login
git checkout -b feature/PROJ-123-add-login develop

# Jira ticket PROJ-456 บั๊กเรื่อง null pointer exception
git checkout -b bugfix/PROJ-456-fix-null-pointer-exception develop

# GitHub Issue #789 ขอให้เพิ่ม export CSV
git checkout -b feature/789-export-csv-report main

# Linear ticket ENG-241 ขอ dark mode
git checkout -b feature/ENG-241-dark-mode-support main
```

### ทำไมต้องใส่เลข ticket ใน branch name

1. **Traceability (ตรวจสอบย้อนกลับได้)** — เปิด Git log แล้วเห็นชื่อ branch ก็รู้ทันทีว่าเกี่ยวกับ ticket ไหนใน Jira/Linear
2. **Auto-linking** — เครื่องมืออย่าง Jira, Linear, GitHub รองรับการ **auto-link** ระหว่าง branch/PR กับ ticket โดยอัตโนมัติ ถ้าชื่อ branch มีรูปแบบ ID ที่ระบบรู้จัก เช่น Jira Smart Commits หรือ GitHub keyword เช่น `Closes #123` ในคำอธิบาย PR ที่จะปิด issue อัตโนมัติเมื่อ merge
3. **ลดงาน manual ในการอัปเดตสถานะ** — บางระบบ (เช่น GitHub + Jira integration) จะเปลี่ยนสถานะ ticket เป็น "In Progress" อัตโนมัติทันทีที่มีการสร้าง branch หรือเปิด PR ที่มีเลข ticket ตรงกัน
4. **ค้นหาง่ายเมื่อมีปัญหา** — ถ้า production มีปัญหาแล้วสงสัยว่าเกิดจาก feature ไหน สามารถค้นหาด้วยเลข ticket ได้ทันทีทั้งใน Git และใน issue tracker

### ข้อควรระวัง

- **อย่าใส่เลข ticket ปนกับตัวอักษรพิเศษที่ shell ตีความผิด** เช่น `#` ในบาง shell จะถูกตีความเป็นคอมเมนต์ จึงนิยมตัด `#` ออกจากชื่อ branch เช่น GitHub Issue #123 เขียนเป็น `123` เฉย ๆ
- **อย่าลืมใส่คำอธิบายสั้น ๆ ต่อท้ายด้วย** เพราะแค่เลข ticket อย่างเดียว เช่น `feature/PROJ-123` ก็ยังไม่สื่อความหมายพอ ต้องเห็น context คร่าว ๆ ด้วยว่าเรื่องอะไร
- **ตรวจสอบ pattern ที่ทีมตกลงกันให้ตรงกันทุกคน** เพราะถ้าใช้ regex ตรวจสอบใน CI (เช่นบังคับว่าชื่อ branch ต้องตรง pattern `^(feature|bugfix|hotfix)/[A-Z]+-[0-9]+-.+$`) แล้วมีคนตั้งชื่อผิดรูปแบบ จะถูก pipeline reject

### ตัวอย่าง regex สำหรับตรวจสอบชื่อ branch อัตโนมัติ

หลายทีมตั้งกฎใน CI/CD ให้ตรวจสอบชื่อ branch ก่อนอนุญาตให้ merge เช่น:

```bash
#!/bin/bash
# scripts/check-branch-name.sh
BRANCH_NAME=$(git rev-parse --abbrev-ref HEAD)
PATTERN="^(feature|bugfix|hotfix|release|chore|docs)\/[A-Z]+-[0-9]+-[a-z0-9-]+$"

if [[ ! $BRANCH_NAME =~ $PATTERN ]]; then
  echo "❌ ชื่อ branch ไม่ถูกต้องตามมาตรฐาน: $BRANCH_NAME"
  echo "รูปแบบที่ถูกต้อง: <type>/<TICKET-ID>-<description>"
  echo "ตัวอย่าง: feature/PROJ-123-add-login"
  exit 1
fi

echo "✅ ชื่อ branch ถูกต้อง: $BRANCH_NAME"
```

เราจะพูดถึงการนำ script แบบนี้ไปผูกกับ CI/CD pipeline จริงในเชิงลึกเมื่อถึง Part ที่ว่าด้วย GitHub Actions และ GitLab CI/CD ในเฟสถัดไป

---

## Step 344: Conventional Commits specification เต็มรูปแบบ (`type(scope): description`)

**Conventional Commits** คือข้อกำหนด (specification) แบบเปิดสำหรับการเขียน commit message ให้มีโครงสร้างที่สม่ำเสมอ อ่านเข้าใจง่ายทั้งสำหรับมนุษย์และเครื่องจักร สเปกอย่างเป็นทางการอยู่ที่ **https://www.conventionalcommits.org**

### โครงสร้างพื้นฐานของ Conventional Commits

```
<type>[optional scope][optional !]: <description>

[optional body]

[optional footer(s)]
```

มาแยกส่วนประกอบทีละส่วนอย่างละเอียด:

#### 1. `<type>` — ประเภทของการเปลี่ยนแปลง (บังคับ)

คำสั้น ๆ ที่บอกว่า commit นี้เป็นการเปลี่ยนแปลงประเภทไหน เช่น `feat`, `fix`, `docs` (รายละเอียดเต็มใน Step 345)

#### 2. `(scope)` — ขอบเขตของการเปลี่ยนแปลง (ไม่บังคับ)

ระบุส่วนของโค้ดที่ได้รับผลกระทบ เขียนอยู่ในวงเล็บต่อจาก type ทันที เช่น `feat(auth):`, `fix(cart):`, `docs(readme):`

scope ควรเป็นคำนามที่สื่อถึงโมดูล, component, หรือส่วนของระบบ เช่น:

```
feat(api): เพิ่ม endpoint สำหรับดึงข้อมูลผู้ใช้
fix(ui): แก้ปุ่ม submit ไม่ทำงานบน Safari
docs(readme): เพิ่มคำอธิบายการติดตั้ง
refactor(database): แยก query logic ออกจาก controller
```

#### 3. `!` — สัญลักษณ์บอก breaking change (ไม่บังคับ)

ใส่เครื่องหมาย `!` ต่อท้าย type หรือ scope ก่อนเครื่องหมาย `:` เพื่อบอกว่า commit นี้มีการเปลี่ยนแปลงที่ **ทำลายความเข้ากันได้กับเวอร์ชันก่อนหน้า (breaking change)** (รายละเอียดเต็มใน Step 346)

```
feat(api)!: เปลี่ยนรูปแบบ response ของ endpoint /users
```

#### 4. `: ` — เครื่องหมาย colon ตามด้วยช่องว่างหนึ่งครั้ง (บังคับ)

ต้องมี colon และเว้นวรรคหนึ่งครั้งเสมอ ก่อนเริ่มเขียน description

#### 5. `<description>` — คำอธิบายสั้น ๆ (บังคับ)

สรุปสิ่งที่เปลี่ยนแปลงในประโยคเดียว สั้น กระชับ ได้ใจความ ตามสเปกแนะนำให้:

- ใช้กริยารูปปัจจุบัน/คำสั่ง (imperative mood) เช่น "add" ไม่ใช่ "added" หรือ "adds" (สำหรับภาษาอังกฤษ) หรือถ้าเขียนภาษาไทยให้ใช้รูปประโยคบอกการกระทำตรง ๆ เช่น "เพิ่ม...", "แก้ไข..." ไม่ใช่ "เพิ่มแล้ว...", "ได้แก้ไข..."
- ไม่ขึ้นต้นด้วยตัวพิมพ์ใหญ่ (สำหรับภาษาอังกฤษ)
- ไม่ใส่จุด (.) ปิดท้ายประโยค
- ความยาวไม่ควรเกิน 50–72 ตัวอักษร (สำหรับบรรทัดแรก)

#### 6. Body — เนื้อหาอธิบายเพิ่มเติม (ไม่บังคับ)

เว้นบรรทัดว่าง 1 บรรทัดจาก description แล้วเขียนรายละเอียดเพิ่มเติมว่า **ทำไม** ถึงเปลี่ยนแปลงแบบนี้ (ไม่ใช่แค่ "ทำอะไร" เพราะ "ทำอะไร" ควรเห็นได้จาก diff อยู่แล้ว)

```
fix(auth): แก้ปัญหา session หมดอายุเร็วเกินไป

เดิม session timeout ถูกตั้งเป็น 5 นาทีโดยไม่ได้ตั้งใจ
เนื่องจาก config ค่า default ผิดตั้งแต่ตอน migrate จากระบบเก่า
ปรับให้กลับมาเป็น 30 นาทีตามที่ทีม UX กำหนดไว้ในเอกสาร spec
```

#### 7. Footer — ข้อมูลเพิ่มเติมท้าย commit (ไม่บังคับ)

ใช้สำหรับข้อมูลเชิง metadata เช่น การอ้างอิง issue, breaking change, หรือผู้ร่วมเขียน โดยแต่ละ footer เขียนเป็นคู่ `token: value` หรือ `token #value`

```
Closes #123
Refs: PROJ-456
Reviewed-by: สมชาย ใจดี
BREAKING CHANGE: เปลี่ยนชื่อ field response จาก `user_name` เป็น `username`
```

### BNF-like grammar อย่างเป็นทางการ (สรุปจากสเปก)

```
<commit message> ::= <type>[optional scope][optional !]: <description>
                      [optional body]
                      [optional footer(s)]

<type>           ::= "feat" | "fix" | "docs" | "style" | "refactor" |
                      "test" | "chore" | "perf" | "build" | "ci" | ...

<scope>          ::= "(" <noun describing a section of the codebase> ")"

<description>    ::= <short summary in imperative mood>

<body>           ::= <free-form text, one or more paragraphs>

<footer>         ::= <token> ": " <value> | <token> " #" <value>
```

### ตัวอย่าง commit ที่ครบทุกส่วนประกอบ

```
feat(shopping-cart)!: เปลี่ยนวิธีคำนวณส่วนลดให้รองรับหลายโปรโมชันพร้อมกัน

เดิมระบบรองรับโปรโมชันได้ทีละ 1 รายการต่อออเดอร์เท่านั้น
ทำให้ทีมการตลาดไม่สามารถจัดแคมเปญซ้อนกันได้ตามที่ต้องการ
ปรับ logic การคำนวณส่วนลดใหม่ทั้งหมดให้รองรับการซ้อนโปรโมชันได้
โดยยึดลำดับความสำคัญตามที่ระบุใน docs/promotion-priority.md

BREAKING CHANGE: field `discount` ใน response ของ /api/cart
เปลี่ยนจาก number เดี่ยว เป็น array ของ object {type, amount}
ผู้ใช้ API ต้องปรับโค้ดฝั่ง client ให้อ่านค่าจาก array แทน

Closes #482
Refs: PROJ-991
```

### ตัวอย่าง commit แบบสั้น ๆ ที่ใช้บ่อยที่สุดในชีวิตประจำวัน

```
feat(auth): เพิ่มระบบ login ด้วย Google OAuth
fix(cart): แก้ปัญหาราคารวมคำนวณผิดเมื่อมีสินค้าลดราคา
docs(readme): อัปเดตขั้นตอนการติดตั้งให้ตรงกับเวอร์ชันล่าสุด
style(button): ปรับ format โค้ดตาม prettier config ใหม่
refactor(user-service): แยก validation logic ออกจาก controller
test(checkout): เพิ่ม unit test สำหรับ edge case จำนวนสินค้าเป็น 0
chore(deps): อัปเดต lodash เป็นเวอร์ชัน 4.17.21
perf(image): ลดขนาดไฟล์ภาพก่อน upload เพื่อลด bandwidth
build(webpack): ปรับ config ให้ build ไฟล์ CSS แยกจาก JS
ci(github-actions): เพิ่ม step รัน test coverage report
```

สังเกตว่าแต่ละบรรทัดเป็นการ commit แยกกัน ไม่ใช่ commit เดียวรวมกันทั้งหมด — นี่คือหัวใจสำคัญของ Conventional Commits คือ **หนึ่ง commit ควรทำหน้าที่เดียวที่ชัดเจน** ไม่ใช่รวมหลายอย่างปนกันจนไม่รู้ว่า type ไหนถูกต้อง

---

## Step 345: Commit types มาตรฐานทั้งหมด (feat, fix, docs, style, refactor, test, chore, perf, build, ci) พร้อมความหมายและตัวอย่างแต่ละแบบ

สเปกหลักของ Conventional Commits กำหนดไว้แค่ 2 type บังคับคือ `feat` และ `fix` เพราะสองอันนี้ผูกกับ Semantic Versioning โดยตรง แต่ในทางปฏิบัติ อุตสาหกรรมได้ขยายชุด type มาตรฐานเพิ่มเติมโดยอ้างอิงจากรูปแบบที่ **Angular Commit Convention** วางไว้ ซึ่งกลายเป็นมาตรฐานที่ทุกเครื่องมือ (commitlint, semantic-release ฯลฯ) รองรับเป็น default

### ตารางสรุป type มาตรฐานทั้งหมด

| Type | ความหมาย | มีผลต่อ Semantic Version | ตัวอย่างการใช้งาน |
|---|---|---|---|
| `feat` | เพิ่มฟีเจอร์ใหม่ (feature) ให้ผู้ใช้ | **MINOR** (เช่น 1.2.0 → 1.3.0) | `feat(profile): เพิ่มฟีเจอร์อัปโหลดรูปโปรไฟล์` |
| `fix` | แก้ไขบั๊ก (bug fix) | **PATCH** (เช่น 1.2.0 → 1.2.1) | `fix(payment): แก้บั๊กยอดชำระคำนวณผิดเมื่อมีภาษี` |
| `docs` | เปลี่ยนแปลงเฉพาะเอกสาร (documentation) ไม่กระทบโค้ดที่รันจริง | ไม่มีผล | `docs(api): เพิ่มตัวอย่างการเรียกใช้ endpoint /login` |
| `style` | เปลี่ยนแปลงที่ไม่กระทบความหมายของโค้ด เช่น format, เว้นวรรค, เครื่องหมายเซมิโคลอน | ไม่มีผล | `style(component): จัด indent ให้ตรงตาม eslint config` |
| `refactor` | ปรับโครงสร้างโค้ดโดยไม่เปลี่ยน behavior ภายนอก (ไม่ใช่ feat และไม่ใช่ fix) | ไม่มีผล (บางทีม map เป็น PATCH) | `refactor(order-service): แยกฟังก์ชันคำนวณราคาออกเป็น class ใหม่` |
| `test` | เพิ่มหรือแก้ไข test ที่มีอยู่ | ไม่มีผล | `test(auth): เพิ่ม test case สำหรับ token หมดอายุ` |
| `chore` | งานดูแลระบบทั่วไปที่ไม่เข้าหมวดอื่น เช่น อัปเดต dependency, ปรับ script | ไม่มีผล | `chore(deps): อัปเดต react เป็นเวอร์ชัน 18.3.0` |
| `perf` | ปรับปรุงประสิทธิภาพ (performance) | **PATCH** | `perf(query): เพิ่ม index ให้ตาราง orders ลดเวลา query 80%` |
| `build` | เปลี่ยนแปลงระบบ build หรือ dependency ภายนอกที่กระทบการ build | ไม่มีผล | `build(webpack): เปลี่ยนจาก webpack 4 เป็น webpack 5` |
| `ci` | เปลี่ยนแปลง configuration หรือ script ของ CI/CD | ไม่มีผล | `ci(github-actions): เพิ่ม matrix testing สำหรับ node 18 และ 20` |

### รายละเอียดเชิงลึกของแต่ละ type

#### `feat` — Feature (ฟีเจอร์ใหม่)

ใช้เมื่อเพิ่มความสามารถใหม่ที่ **ผู้ใช้ปลายทางสัมผัสได้** ไม่ว่าจะเป็นผู้ใช้ทั่วไปหรือนักพัฒนาที่เรียกใช้ API ของคุณ

```
feat(search): เพิ่มระบบค้นหาสินค้าด้วยเสียง
feat(export): รองรับการ export รายงานเป็นไฟล์ Excel
feat(api): เพิ่ม endpoint GET /api/v2/orders/:id/history
```

**ข้อควรระวัง:** ถ้าเป็นการเพิ่มฟีเจอร์ที่ใช้ภายในทีม dev เท่านั้น (เช่น เพิ่ม dev tool, เพิ่ม script สำหรับ debug) ไม่ควรใช้ `feat` ควรใช้ `chore` แทน เพราะ `feat` ควรสงวนไว้สำหรับสิ่งที่ผู้ใช้จริงได้ประโยชน์

#### `fix` — Bug Fix (แก้บั๊ก)

ใช้เมื่อแก้ไขพฤติกรรมที่ผิดพลาดจากที่ควรจะเป็น

```
fix(cart): แก้ปัญหาสินค้าซ้ำเมื่อกดปุ่มเพิ่มลงตะกร้าเร็วเกินไป
fix(login): แก้ปัญหา error message ไม่แสดงเมื่อรหัสผ่านผิด
fix(date-picker): แก้ timezone ผิดเมื่อผู้ใช้อยู่ต่างโซนเวลา
```

#### `docs` — Documentation (เอกสาร)

ใช้เมื่อแก้ไขเฉพาะไฟล์เอกสาร เช่น README, comment ในโค้ด (ที่ไม่กระทบ logic), CHANGELOG, wiki

```
docs(readme): เพิ่มส่วน troubleshooting สำหรับปัญหาที่พบบ่อย
docs(contributing): อัปเดตขั้นตอนการส่ง pull request
docs(code-comment): เพิ่มคำอธิบายการทำงานของฟังก์ชัน calculateTax
```

#### `style` — Code Style (รูปแบบโค้ด)

ใช้เมื่อเปลี่ยนแปลงที่ **ไม่กระทบความหมายหรือการทำงานของโค้ดเลย** เช่น การจัด format, เว้นวรรค, ลำดับการ import — ข้อควรระวังคือ อย่าสับสนกับ CSS/UI style ซึ่งถ้าเปลี่ยนหน้าตาที่ผู้ใช้เห็น ควรใช้ `feat` หรือ `fix` แทน

```
style(button): จัด format ด้วย prettier
style(imports): เรียงลำดับ import ให้เป็นไปตาม eslint-plugin-import
style: ลบช่องว่างท้ายบรรทัดที่ไม่จำเป็นทั้งโปรเจกต์
```

#### `refactor` — Refactoring (ปรับโครงสร้าง)

ใช้เมื่อเปลี่ยนแปลงโครงสร้างภายในของโค้ด แต่ **ผลลัพธ์ภายนอกยังทำงานเหมือนเดิมทุกประการ** ไม่ใช่การเพิ่มฟีเจอร์ (feat) และไม่ใช่การแก้บั๊ก (fix)

```
refactor(user-repository): เปลี่ยนจาก callback เป็น async/await
refactor(order-service): รวมฟังก์ชันคำนวณราคาที่ซ้ำกัน 3 จุดให้เหลือจุดเดียว
refactor: ย้าย utility functions ไปไว้ใน shared/utils
```

#### `test` — Test (การทดสอบ)

ใช้เมื่อเพิ่ม แก้ไข หรือลบ test เท่านั้น ไม่แตะโค้ด production

```
test(auth): เพิ่ม unit test ครอบคลุม edge case token ว่างเปล่า
test(checkout): เพิ่ม integration test สำหรับ flow ชำระเงินผ่านบัตรเครดิต
test: แก้ test ที่ flaky เนื่องจากใช้ hard-coded timeout
```

#### `chore` — Chore (งานจิปาถะ)

ใช้เป็น "ถังรวม" สำหรับงานดูแลระบบที่ไม่เข้าหมวดไหนข้างต้น เช่น การอัปเดต dependency, ปรับ script, แก้ .gitignore

```
chore(deps): อัปเดต eslint เป็นเวอร์ชัน 9.0.0
chore: เพิ่มไฟล์ .editorconfig สำหรับความสม่ำเสมอของทีม
chore(release): เตรียม version bump สำหรับ v2.5.0
```

#### `perf` — Performance (ประสิทธิภาพ)

ใช้เมื่อการเปลี่ยนแปลงมีจุดประสงค์หลักเพื่อ **เพิ่มความเร็วหรือลดการใช้ทรัพยากร** โดยไม่เปลี่ยนพฤติกรรมที่ผู้ใช้เห็น

```
perf(image-loading): เพิ่ม lazy loading ให้รูปภาพในหน้ารายการสินค้า
perf(database): เปลี่ยนจาก N+1 query เป็น JOIN เดียว
perf(bundle): ลดขนาด JavaScript bundle ด้วย code splitting
```

#### `build` — Build System (ระบบ build)

ใช้เมื่อเปลี่ยนแปลงเครื่องมือหรือ configuration ที่เกี่ยวกับกระบวนการ build เช่น webpack, vite, gradle, dependency ที่จำเป็นสำหรับ build (ไม่ใช่ dependency ทั่วไปที่ใช้ chore)

```
build(vite): เปลี่ยนจาก webpack มาใช้ vite เพื่อความเร็วในการ build
build(docker): ปรับ Dockerfile ให้ multi-stage build ลดขนาด image
build(npm): เพิ่ม script สำหรับ build production แยกจาก development
```

#### `ci` — Continuous Integration (การทำงานต่อเนื่องอัตโนมัติ)

ใช้เมื่อเปลี่ยนแปลง configuration ของระบบ CI/CD เช่น GitHub Actions, GitLab CI, Jenkins

```
ci(github-actions): เพิ่ม workflow สำหรับรัน test อัตโนมัติเมื่อเปิด PR
ci(gitlab): แก้ pipeline ที่ deploy ผิด environment
ci: เพิ่ม cache สำหรับ node_modules เพื่อลดเวลา build
```

### สรุปหลักการเลือก type ให้ถูกต้อง

ผังการตัดสินใจง่าย ๆ สำหรับเลือก type:

```
เปลี่ยนแปลงนี้ผู้ใช้เห็นความสามารถใหม่ไหม?
├── ใช่ ──────────────────────────────► feat
└── ไม่ใช่
    เป็นการแก้พฤติกรรมที่ผิดพลาดไหม?
    ├── ใช่ ──────────────────────────► fix
    └── ไม่ใช่
        เป็นการเปลี่ยนเฉพาะเอกสารไหม?
        ├── ใช่ ──────────────────────► docs
        └── ไม่ใช่
            เป็นการเปลี่ยนแค่ format/style โค้ดไหม?
            ├── ใช่ ──────────────────► style
            └── ไม่ใช่
                เป็นการปรับโครงสร้างโดยไม่เปลี่ยน behavior ไหม?
                ├── ใช่ ──────────────► refactor
                └── ไม่ใช่
                    เป้าหมายหลักคือเพิ่มความเร็ว/ลดทรัพยากรไหม?
                    ├── ใช่ ──────────► perf
                    └── ไม่ใช่
                        เป็นเรื่อง test เท่านั้นไหม?
                        ├── ใช่ ──────► test
                        └── ไม่ใช่
                            เป็นเรื่อง build system ไหม?
                            ├── ใช่ ──► build
                            └── ไม่ใช่
                                เป็นเรื่อง CI/CD ไหม?
                                ├── ใช่ ──► ci
                                └── ไม่ใช่ ► chore
```

---

## Step 346: Breaking changes ใน Conventional Commits (`!` หลัง type/scope หรือ footer `BREAKING CHANGE:`)

**Breaking change** คือการเปลี่ยนแปลงที่ **ทำลายความเข้ากันได้ (compatibility) กับโค้ดเวอร์ชันก่อนหน้า** พูดง่าย ๆ คือ ถ้าใครใช้ library หรือ API เวอร์ชันเก่าอยู่ แล้วอัปเดตมาใช้เวอร์ชันใหม่ที่มี breaking change โค้ดของเขาจะ **พังทันทีถ้าไม่ปรับตาม**

Conventional Commits กำหนดให้ breaking change เป็นข้อมูลที่ **สำคัญที่สุด** ในระบบ เพราะมันมีผลโดยตรงต่อ Semantic Versioning (ต้อง bump เวอร์ชันแบบ **MAJOR** เท่านั้น เช่น 1.5.2 → 2.0.0) และมีผลต่อการตัดสินใจของผู้ใช้ library ว่าจะอัปเดตตอนนี้เลยหรือรอปรับโค้ดก่อน

### วิธีที่ 1: ใช้เครื่องหมาย `!` ต่อท้าย type หรือ scope

วิธีนี้เหมาะกับกรณีที่ต้องการบอกแบบสั้น กระชับ ไม่ต้องเขียนรายละเอียดยาว

```
feat!: ลบการรองรับ Node.js เวอร์ชัน 14 ลงไป
```

```
feat(api)!: เปลี่ยนโครงสร้าง response ของ endpoint /users จาก object เป็น array
```

```
fix(auth)!: เปลี่ยนชื่อ environment variable จาก AUTH_SECRET เป็น JWT_SECRET
```

สังเกตว่าเครื่องหมาย `!` วางอยู่ **ก่อน** เครื่องหมาย `:` เสมอ ไม่ว่าจะมี scope หรือไม่ก็ตาม

### วิธีที่ 2: ใช้ footer `BREAKING CHANGE:`

วิธีนี้เหมาะเมื่อต้องการอธิบายรายละเอียดของ breaking change อย่างชัดเจน ว่าเปลี่ยนอะไร ทำไม และผู้ใช้ต้องทำอะไรเพื่อ migrate

```
feat(api): ปรับปรุงระบบ authentication ทั้งหมดให้ใช้ JWT

เปลี่ยนจากระบบ session-based authentication แบบเดิม
มาเป็น JWT-based authentication เพื่อรองรับการ scale
แบบ stateless ในสถาปัตยกรรม microservices

BREAKING CHANGE: endpoint /api/login ไม่คืนค่า session cookie
อีกต่อไป แต่จะคืนค่า JWT token ในฟิลด์ `token` ของ response
ผู้ใช้ต้องแนบ token นี้ใน header `Authorization: Bearer <token>`
สำหรับทุก request ที่ต้องการ authentication แทนการพึ่งพา cookie
```

**ข้อสำคัญ:** footer `BREAKING CHANGE:` ต้องเขียนด้วยตัวพิมพ์ใหญ่ทั้งหมด (ALL CAPS) เท่านั้น ตามสเปกอย่างเป็นทางการ เขียนเป็น `Breaking Change:` หรือ `breaking change:` จะไม่ถูกเครื่องมือหลายตัวจดจำว่าเป็น breaking change

### สามารถใช้ทั้งสองวิธีพร้อมกันได้

```
feat(payment)!: เปลี่ยนไปใช้ Stripe API เวอร์ชัน 2024 แทนเวอร์ชันเก่า

BREAKING CHANGE: parameter `amount` ต้องส่งเป็นหน่วยสตางค์
(integer) แทนที่จะเป็นหน่วยบาท (float) เหมือนเดิม เช่น
100 บาท ต้องส่งเป็น 10000 แทนที่จะส่ง 100.00
```

การใส่ทั้ง `!` และ footer `BREAKING CHANGE:` พร้อมกันเป็นแนวปฏิบัติที่แนะนำ เพราะ `!` ทำให้มองเห็นได้เร็วจาก `git log --oneline` ในขณะที่ footer ให้รายละเอียดสำหรับคนที่ต้องการรู้ลึก

### breaking change ไม่จำเป็นต้องมาจาก `feat` เท่านั้น

หลายคนเข้าใจผิดว่า breaking change เกิดได้เฉพาะกับ `feat` แต่จริง ๆ แล้ว **type ไหนก็มี breaking change ได้** ถ้าการเปลี่ยนแปลงนั้นทำลาย compatibility เช่น

```
fix!: แก้ bug ที่ทำให้ค่า default timeout ผิด แต่การแก้นี้เปลี่ยนพฤติกรรม
ของระบบที่ผู้ใช้บางส่วนอาจพึ่งพา behavior ผิดๆ นั้นอยู่โดยไม่รู้ตัว

BREAKING CHANGE: ค่า default ของ `requestTimeout` เปลี่ยนจาก
0 (ไม่มี timeout) เป็น 30000 (30 วินาที) ผู้ใช้ที่ต้องการ
ปิด timeout ต้องระบุค่า 0 อย่างชัดเจนในการตั้งค่าเอง
```

### ผลกระทบต่อ Semantic Versioning

| ประเภทการเปลี่ยนแปลง | ตัวอย่าง version bump |
|---|---|
| มี `BREAKING CHANGE` หรือ `!` (ไม่ว่า type ใด) | **MAJOR**: `1.4.2` → `2.0.0` |
| มี `feat` (ไม่มี breaking change) | **MINOR**: `1.4.2` → `1.5.0` |
| มี `fix`, `perf` (ไม่มี breaking change) | **PATCH**: `1.4.2` → `1.4.3` |
| มีแค่ `docs`, `style`, `chore`, `test`, `ci`, `build`, `refactor` เท่านั้น | ไม่ bump เวอร์ชัน (ในหลาย config) |

กฎนี้คือรากฐานที่ทำให้เครื่องมือ **semantic-release** สามารถตัดสินใจเองได้ทั้งหมดว่าควรออกเวอร์ชันอะไร โดยไม่ต้องให้มนุษย์มานั่งตัดสินใจเอง ซึ่งเราจะพูดถึงรายละเอียดเต็มรูปแบบใน Step 347

### ตารางสรุปการเขียน breaking change ที่ถูกต้อง

| สถานการณ์ | วิธีเขียนที่แนะนำ |
|---|---|
| Breaking change ที่อธิบายสั้น ๆ พอ | ใช้ `!` เท่านั้น เช่น `feat!: ...` |
| Breaking change ที่ต้องอธิบายวิธี migrate | ใช้ footer `BREAKING CHANGE:` พร้อมรายละเอียด |
| Breaking change ที่สำคัญมาก ต้องการให้เห็นชัดทั้งสองทาง | ใช้ทั้ง `!` และ footer พร้อมกัน |
| ไม่แน่ใจว่า breaking change หรือไม่ | ให้ถือว่า **เป็น** breaking change ไว้ก่อน (ปลอดภัยกว่าการมองข้าม) |

---

## Step 347: ประโยชน์ของ Conventional Commits (auto-generate changelog, semantic-release กำหนดเวอร์ชันอัตโนมัติ)

หลายคนอาจสงสัยว่า "ทำไมต้องเคร่งครัดกับรูปแบบ commit message ขนาดนี้ แค่เขียนให้อ่านรู้เรื่องไม่พอหรือ" คำตอบคือ **เมื่อ commit message มีรูปแบบที่แน่นอนตายตัว มันจะกลายเป็น "ข้อมูลที่เครื่องจักรอ่านได้ (machine-readable data)"** ไม่ใช่แค่ข้อความอิสระที่มนุษย์อ่านเข้าใจเท่านั้น และนี่คือจุดที่ปลดล็อกระบบอัตโนมัติทรงพลังหลายตัว

### 1. Auto-generate Changelog (สร้าง Changelog อัตโนมัติ)

แทนที่ทีมจะต้องมานั่งเขียน Release Notes ด้วยมือทุกครั้งที่ปล่อยเวอร์ชันใหม่ (ซึ่งมักถูกลืมหรือเขียนไม่ครบ) เครื่องมืออย่าง **conventional-changelog** หรือ **standard-version** จะสามารถ **ไล่อ่าน commit message ทั้งหมดตั้งแต่ tag เวอร์ชันก่อนหน้า แล้วจัดกลุ่มตาม type โดยอัตโนมัติ** เพื่อสร้างไฟล์ `CHANGELOG.md` ให้ทันที

ตัวอย่าง commit history:

```
feat(auth): เพิ่มระบบ login ด้วย Google OAuth
fix(cart): แก้ปัญหาราคารวมคำนวณผิด
feat(export): รองรับการ export เป็น PDF
fix(login): แก้ error message ไม่แสดงผล
docs(readme): อัปเดตคำแนะนำการติดตั้ง
perf(image): ลดขนาดไฟล์ภาพก่อน upload
```

Changelog ที่ generate ออกมาโดยอัตโนมัติจะมีหน้าตาประมาณนี้:

```markdown
# Changelog

## [1.5.0] - 2026-09-26

### Features
- **auth:** เพิ่มระบบ login ด้วย Google OAuth
- **export:** รองรับการ export เป็น PDF

### Bug Fixes
- **cart:** แก้ปัญหาราคารวมคำนวณผิด
- **login:** แก้ error message ไม่แสดงผล

### Performance Improvements
- **image:** ลดขนาดไฟล์ภาพก่อน upload
```

สังเกตว่า commit ประเภท `docs` ไม่ปรากฏใน Changelog เพราะโดย default มันไม่ใช่สิ่งที่ผู้ใช้ปลายทางสนใจ — นี่คือประโยชน์อีกข้อของการแยก type ให้ชัดเจน เพราะเครื่องมือสามารถ **กรอง** ได้ว่าอะไรควรโชว์ อะไรไม่ควรโชว์ในแต่ละบริบท

### 2. Semantic-release กำหนดเวอร์ชันอัตโนมัติ

**semantic-release** คือเครื่องมือที่ทำให้กระบวนการปล่อยเวอร์ชันซอฟต์แวร์ (release) เป็นไปโดยอัตโนมัติทั้งหมด โดยไม่ต้องมีมนุษย์มาตัดสินใจว่า "รอบนี้ควรออกเป็นเวอร์ชันอะไร" อีกต่อไป

ขั้นตอนการทำงานของมันคือ:

1. อ่าน commit message ทั้งหมดตั้งแต่ release ล่าสุด
2. วิเคราะห์ว่ามี `BREAKING CHANGE`/`!` (→ MAJOR), มี `feat` (→ MINOR), หรือมีแค่ `fix`/`perf` (→ PATCH)
3. คำนวณเลขเวอร์ชันถัดไปตามกฎ Semantic Versioning โดยอัตโนมัติ
4. สร้าง Git tag ใหม่ เช่น `v2.4.0`
5. สร้าง/อัปเดต `CHANGELOG.md` อัตโนมัติ
6. Publish package ขึ้น registry (เช่น npm) โดยอัตโนมัติ
7. สร้าง GitHub Release พร้อม Release Notes โดยอัตโนมัติ

ทั้งหมดนี้เกิดขึ้นได้ **เพราะ commit message เขียนตาม Conventional Commits เท่านั้น** ถ้า commit message เป็นข้อความอิสระแบบ "แก้บั๊ก", "update code" เครื่องมือจะไม่มีทางรู้เลยว่าควร bump เวอร์ชันแบบไหน

### ตัวอย่างผลลัพธ์ของ semantic-release ในโลกจริง

```
$ npx semantic-release

[semantic-release] Analyzing commits since last release (v2.3.1)...
[semantic-release] Found 3 commits with type "fix"
[semantic-release] Found 2 commits with type "feat"
[semantic-release] Found 0 breaking changes
[semantic-release] Decision: bump MINOR version
[semantic-release] Next version: 2.4.0
[semantic-release] Creating tag v2.4.0...
[semantic-release] Generating CHANGELOG.md...
[semantic-release] Publishing to npm registry...
[semantic-release] Creating GitHub Release v2.4.0...
[semantic-release] Done! 🎉
```

### ประโยชน์อื่น ๆ ที่ตามมา

1. **ลดข้อผิดพลาดจากมนุษย์** — ไม่มีการลืมอัปเดต version number หรือลืมเขียน changelog อีกต่อไป
2. **ความสม่ำเสมอของเวอร์ชัน** — ทุกทีมในองค์กรใช้กฎเดียวกันในการตัดสินใจ version bump ไม่ขึ้นกับความรู้สึกของแต่ละคน
3. **ทำให้ CI/CD pipeline ปล่อย release ได้แบบ hands-off เต็มรูปแบบ** — commit เข้า `main` แล้วปล่อยเวอร์ชันใหม่ได้เลยโดยไม่ต้องมีคนกดปุ่มใด ๆ
4. **สร้างความน่าเชื่อถือให้กับผู้ใช้ library** — เมื่อเห็นว่าเวอร์ชันเปลี่ยนจาก `2.3.1` เป็น `3.0.0` ผู้ใช้รู้ทันทีว่ามี breaking change และต้องอ่าน migration guide ก่อนอัปเดต โดยไม่ต้องเดา
5. **ใช้ในการ generate Release Notes ที่อ่านง่ายสำหรับผู้บริหารหรือลูกค้า** — เพราะข้อมูลถูกจัดหมวดหมู่ไว้แล้วตั้งแต่ต้นทาง

### เรื่องนี้จะถูกเจาะลึกใน Part 88–89

Part นี้เราแค่ปูพื้นฐานให้เห็นภาพว่า Conventional Commits เป็นรากฐานสำคัญของระบบอัตโนมัติเหล่านี้ ส่วนรายละเอียดการติดตั้งและตั้งค่า **semantic-release** แบบครบวงจร การเขียน `.releaserc` configuration, การผูกเข้ากับ CI/CD pipeline จริง, และการ generate changelog แบบ custom template จะถูกอธิบายอย่างละเอียดใน **Part 88: Semantic Versioning และ Automated Release** และ **Part 89: Automated Changelog Generation** ซึ่งอยู่ในเฟส 9 ของหลักสูตรนี้ (ทักษะมืออาชีพ: Release Management)

---

## Step 348: เครื่องมือช่วยบังคับใช้มาตรฐาน (commitlint, husky commit-msg hook) พร้อมตัวอย่าง config

การมีมาตรฐานที่ดีอย่างเดียวไม่พอ ถ้าไม่มีกลไก **บังคับใช้ (enforcement)** สุดท้ายทีมก็จะค่อย ๆ เลิกทำตามมาตรฐานเมื่อรีบเร่งหรือเหนื่อยล้า จึงต้องมีเครื่องมืออัตโนมัติมาช่วยตรวจสอบก่อนที่ commit message ที่ผิดรูปแบบจะเข้าไปอยู่ใน history จริง

### เครื่องมือหลักที่ใช้กันทั่วโลก

| เครื่องมือ | หน้าที่ |
|---|---|
| **commitlint** | ตรวจสอบว่า commit message ตรงตามกฎ Conventional Commits หรือไม่ |
| **husky** | จัดการ Git hooks ให้ทำงานอัตโนมัติ (เช่น รัน commitlint ทุกครั้งที่ commit) |
| **commitizen** | ช่วยให้ผู้ใช้พิมพ์ commit message ผ่าน interactive prompt แทนการพิมพ์เองทั้งหมด ลดโอกาสพิมพ์ผิดรูปแบบ |

### ขั้นตอนติดตั้ง commitlint + husky

```bash
# ติดตั้ง commitlint พร้อม config มาตรฐานของ Conventional Commits
npm install --save-dev @commitlint/cli @commitlint/config-conventional

# ติดตั้ง husky สำหรับจัดการ Git hooks
npm install --save-dev husky

# เปิดใช้งาน husky
npx husky init
```

### สร้างไฟล์ config ของ commitlint

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',
        'fix',
        'docs',
        'style',
        'refactor',
        'test',
        'chore',
        'perf',
        'build',
        'ci',
      ],
    ],
    'subject-case': [0], // อนุญาตให้เขียน description เป็นภาษาไทยได้ ไม่บังคับ case
    'header-max-length': [2, 'always', 100],
  },
};
```

### ผูก commitlint เข้ากับ husky `commit-msg` hook

```bash
# สร้าง hook ที่ทำงานทุกครั้งก่อน commit จะสำเร็จ
echo "npx --no -- commitlint --edit \$1" > .husky/commit-msg
chmod +x .husky/commit-msg
```

ไฟล์ `.husky/commit-msg` จะมีเนื้อหาประมาณนี้:

```bash
#!/usr/bin/env sh
npx --no -- commitlint --edit $1
```

### ผลลัพธ์เมื่อมีคน commit ผิดรูปแบบ

```bash
$ git commit -m "แก้บั๊กเล็กน้อย"

⧗   input: แก้บั๊กเล็กน้อย
✖   subject may not be empty [subject-empty]
✖   type may not be empty [type-empty]

✖   found 2 problems, 0 warnings
⓪    husky - commit-msg hook exited with code 1 (error)
```

commit จะถูก **ปฏิเสธทันที** ไม่สามารถบันทึกเข้า repository ได้จนกว่าจะแก้ไขข้อความให้ถูกต้องตามรูปแบบ:

```bash
$ git commit -m "fix(login): แก้บั๊กปุ่ม login ไม่ทำงานเมื่อกรอกอีเมลผิดรูปแบบ"

✔   commit-msg hook passed
[feature/PROJ-123-fix-login abc1234] fix(login): แก้บั๊กปุ่ม login ไม่ทำงานเมื่อกรอกอีเมลผิดรูปแบบ
 1 file changed, 3 insertions(+), 1 deletion(-)
```

### ใช้ commitizen ช่วยให้พิมพ์ commit message ง่ายขึ้น

สำหรับสมาชิกทีมที่ยังไม่คุ้นกับรูปแบบ Conventional Commits การใช้ `commitizen` จะช่วยให้ไม่ต้องจำรูปแบบด้วยตัวเอง เพราะมันจะถามทีละขั้นตอน

```bash
npm install --save-dev commitizen cz-conventional-changelog
```

เพิ่ม config ใน `package.json`:

```json
{
  "config": {
    "commitizen": {
      "path": "cz-conventional-changelog"
    }
  },
  "scripts": {
    "commit": "cz"
  }
}
```

เมื่อใช้งาน `npm run commit` แทน `git commit` จะได้ interactive prompt แบบนี้:

```
$ npm run commit

? Select the type of change that you're committing:
  feat:      A new feature
  fix:       A bug fix
  docs:      Documentation only changes
  style:     Code style changes (formatting, semicolons, etc)
  refactor:  Code change that neither fixes a bug nor adds a feature
  test:      Adding missing tests
  chore:     Changes to the build process or auxiliary tools
❯ perf:      A code change that improves performance

? What is the scope of this change (e.g. component or file name):
  cart

? Write a short, imperative tense description of the change:
  ลดเวลาโหลดหน้าตะกร้าสินค้าด้วยการทำ pagination

? Provide a longer description of the change (optional):
  (skip)

? Are there any breaking changes?
  No

? Does this change affect any open issues?
  Yes

? Add issue references (e.g. "fix #123", "re #123"):
  Closes #201
```

ผลลัพธ์ที่ได้จะถูกประกอบเป็น commit message ที่ถูกต้องตามสเปกโดยอัตโนมัติ:

```
perf(cart): ลดเวลาโหลดหน้าตะกร้าสินค้าด้วยการทำ pagination

Closes #201
```

### ตรวจสอบชื่อ branch พร้อมกันไปด้วยผ่าน `pre-push` hook

นอกจากตรวจ commit message แล้ว ทีมมักเพิ่ม hook ตรวจสอบชื่อ branch ก่อน push ด้วย:

```bash
# .husky/pre-push
#!/usr/bin/env sh
BRANCH_NAME=$(git rev-parse --abbrev-ref HEAD)
PATTERN="^(feature|bugfix|hotfix|release|chore|docs|refactor|test)\/.+$"

if [[ ! $BRANCH_NAME =~ $PATTERN ]] && [ "$BRANCH_NAME" != "main" ] && [ "$BRANCH_NAME" != "develop" ]; then
  echo "❌ ชื่อ branch '$BRANCH_NAME' ไม่ตรงตามมาตรฐานของทีม"
  echo "รูปแบบที่ถูกต้อง: feature/..., bugfix/..., hotfix/..., release/..."
  exit 1
fi
```

### สรุปสถาปัตยกรรมการบังคับใช้มาตรฐานทั้งระบบ

```
นักพัฒนาพิมพ์ commit
        │
        ▼
┌───────────────────┐
│  husky commit-msg  │ ──► เรียก commitlint ตรวจสอบรูปแบบ
│       hook          │
└───────────────────┘
        │
   ผ่านกฎ?
   ├── ไม่ผ่าน ──► ปฏิเสธ commit, แสดง error ชัดเจน
   └── ผ่าน ──► บันทึก commit สำเร็จ
        │
        ▼
┌───────────────────┐
│  husky pre-push    │ ──► ตรวจสอบชื่อ branch ก่อน push
│       hook          │
└───────────────────┘
        │
   ผ่านกฎ?
   ├── ไม่ผ่าน ──► ปฏิเสธ push
   └── ผ่าน ──► push สำเร็จขึ้น remote
        │
        ▼
┌───────────────────┐
│   CI/CD Pipeline    │ ──► ตรวจสอบซ้ำอีกชั้น (defense in depth)
│  (GitHub Actions)   │     เผื่อมีใคร bypass local hook
└───────────────────┘
```

การมี hook ตรวจสอบทั้งฝั่ง local (husky) และฝั่ง server/CI (GitHub Actions) พร้อมกันเรียกว่าหลักการ **defense in depth** เพราะ local hook สามารถถูก bypass ได้ด้วย `git commit --no-verify` แต่ CI pipeline ฝั่ง server จะตรวจจับได้เสมอไม่ว่าใครจะพยายามเลี่ยงอย่างไร

---

## Step 349: ตัวอย่าง commit message ที่ดีและแย่เทียบกันแบบละเอียด

มาดูตัวอย่างเปรียบเทียบแบบละเอียดในหลากหลายสถานการณ์ เพื่อฝึกสายตาให้แยกแยะได้ทันทีว่า commit message แบบไหนคุณภาพดีหรือแย่

### ตัวอย่างที่ 1: การแก้บั๊ก

| แย่ | ดี | เหตุผลที่ดีกว่า |
|---|---|---|
| `แก้บั๊ก` | `fix(cart): แก้ปัญหาราคารวมคำนวณผิดเมื่อมีสินค้าที่ลดราคาซ้อนกัน 2 รายการ` | บอก type, scope และปัญหาที่แก้อย่างเจาะจง ไม่ใช่แค่คำว่า "แก้บั๊ก" ลอย ๆ |
| `fix bug` | `fix(auth): แก้ token refresh ล้มเหลวเมื่อ clock ของ server เพี้ยนเกิน 5 วินาที` | ระบุ scope และสาเหตุที่ทำให้เกิดปัญหา ทำให้คนอ่านในอนาคตเข้าใจ root cause |
| `fixed the thing` | `fix(export): แก้ปัญหาไฟล์ PDF export ออกมาเป็นหน้าว่างเมื่อรายงานมีมากกว่า 100 แถว` | ระบุเงื่อนไขที่ทำให้เกิดปัญหาชัดเจน ช่วยตอน debug ย้อนหลัง |

### ตัวอย่างที่ 2: การเพิ่มฟีเจอร์

| แย่ | ดี | เหตุผลที่ดีกว่า |
|---|---|---|
| `เพิ่มของใหม่` | `feat(notification): เพิ่มระบบแจ้งเตือนผ่าน push notification บนมือถือ` | สื่อความหมายชัดว่าเพิ่มอะไร ที่ไหน |
| `new feature` | `feat(dashboard): เพิ่มกราฟแสดงยอดขายรายเดือนแบบ real-time` | ระบุฟีเจอร์ที่ชัดเจนแทนคำกว้าง ๆ |
| `add stuff` | `feat(api)!: เพิ่ม endpoint /api/v2/reports พร้อมเปลี่ยนโครงสร้าง response ให้รองรับ pagination` | ระบุ scope, ความหมายชัดเจน และมีเครื่องหมาย `!` บอก breaking change ที่ถูกต้อง |

### ตัวอย่างที่ 3: การรวมหลายอย่างในหนึ่ง commit (ข้อผิดพลาดที่พบบ่อยที่สุด)

**แย่มาก:**

```
update everything

- เพิ่มระบบ login
- แก้บั๊กตะกร้าสินค้า
- อัปเดต dependency
- แก้ typo ใน README
- ปรับ CI pipeline
```

**ปัญหา:** commit เดียวทำ 5 เรื่องที่ไม่เกี่ยวข้องกันเลย ทำให้:
- ถ้าต้อง revert แค่เรื่อง dependency แต่ต้อง revert ทั้งหมดไปด้วย เพราะอยู่ commit เดียวกัน
- `git log` ไม่สามารถบอกได้ว่า type ไหนควรใช้ (feat? fix? chore? docs? ci?)
- Code review ทำได้ยากเพราะ diff ปนกันหมด
- เครื่องมือ semantic-release ไม่สามารถวิเคราะห์ได้เลยว่าควร bump version แบบไหน

**ดี (แยกเป็น 5 commit):**

```
feat(auth): เพิ่มระบบ login ด้วยอีเมลและรหัสผ่าน
fix(cart): แก้บั๊กจำนวนสินค้าไม่อัปเดตหลังลบรายการ
chore(deps): อัปเดต axios เป็นเวอร์ชัน 1.6.0
docs(readme): แก้ typo ในหัวข้อการติดตั้ง
ci(github-actions): เพิ่ม step cache node_modules เพื่อลดเวลา build
```

### ตัวอย่างที่ 4: Description ที่กำกวมเกินไป

| แย่ | ดี | เหตุผลที่ดีกว่า |
|---|---|---|
| `fix: fix it` | `fix(payment): แก้การคำนวณดอกเบี้ยผ่อนชำระที่ปัดเศษผิดในทศนิยมตำแหน่งที่ 2` | บอกรายละเอียดที่เพียงพอให้เข้าใจโดยไม่ต้องเปิด diff |
| `chore: misc changes` | `chore(config): ย้าย environment variable ทั้งหมดไปไว้ใน .env.example` | เจาะจงว่า "misc" คืออะไรกันแน่ |
| `feat: improvements` | `feat(search): เพิ่ม autocomplete suggestion ขณะพิมพ์คำค้นหา` | คำว่า "improvements" ไม่บอกอะไรเลย ต้องระบุให้ชัดว่าปรับปรุงอะไร |

### ตัวอย่างที่ 5: การใช้ type ผิดประเภท

| ผิด | ถูก | เหตุผล |
|---|---|---|
| `feat(css): ปรับสีปุ่มให้ดูดีขึ้น` | `style(button): ปรับสีปุ่มให้ตรงตาม design system เวอร์ชันใหม่` | ถ้าไม่ใช่ฟีเจอร์ใหม่แต่เป็นการปรับ visual เฉย ๆ ควรใช้ `style` ไม่ใช่ `feat` (แต่ถ้าเปลี่ยน UX ที่มีนัยสำคัญ เช่น เปลี่ยนตำแหน่งปุ่มทั้งหมด อาจใช้ `feat` ได้ ขึ้นกับดุลยพินิจของทีม) |
| `fix(deps): อัปเดต react` | `chore(deps): อัปเดต react เป็นเวอร์ชัน 18.3.0` | การอัปเดต dependency ทั่วไปที่ไม่ได้แก้บั๊กเฉพาะเจาะจง ควรใช้ `chore` ไม่ใช่ `fix` |
| `feat: เพิ่ม test` | `test(order): เพิ่ม unit test สำหรับ order status transition` | การเพิ่ม test ควรใช้ `test` เสมอ ไม่ว่าจะรู้สึกว่าเป็น "งานใหม่" แค่ไหนก็ตาม |

### ตัวอย่างที่ 6: Commit message ที่ยาวเกินไปในบรรทัดแรก

**แย่:**

```
fix(checkout): แก้ปัญหาเมื่อผู้ใช้กดปุ่มยืนยันการสั่งซื้อหลายครั้งติดกันอย่างรวดเร็วภายในเวลาไม่ถึง 1 วินาทีจะทำให้เกิดออเดอร์ซ้ำในระบบฐานข้อมูลซึ่งกระทบต่อการจัดส่งสินค้า
```

**ดี (ย้ายรายละเอียดไปไว้ใน body แทน):**

```
fix(checkout): แก้ปัญหาออเดอร์ซ้ำเมื่อกดยืนยันหลายครั้งติดกัน

เมื่อผู้ใช้กดปุ่มยืนยันการสั่งซื้อหลายครั้งติดกันอย่างรวดเร็ว
ภายในเวลาไม่ถึง 1 วินาที ระบบจะสร้างออเดอร์ซ้ำในฐานข้อมูล
เนื่องจากไม่มีการ debounce ที่ฝั่ง client และไม่มี idempotency
key ที่ฝั่ง server ส่งผลกระทบโดยตรงต่อการจัดส่งสินค้าซ้ำซ้อน

เพิ่มการ debounce ปุ่มยืนยัน 2 วินาที และเพิ่ม idempotency
key ที่ server เพื่อป้องกันปัญหานี้ในระยะยาว
```

### ตารางสรุปหลักการเขียน commit message ที่ดี

| หลักการ | คำอธิบาย |
|---|---|
| หนึ่ง commit ทำหน้าที่เดียว | Atomic commit — เปลี่ยนแปลงเรื่องเดียวต่อหนึ่ง commit |
| บรรทัดแรกสั้น กระชับ | ไม่เกิน 50–72 ตัวอักษร ใช้ body สำหรับรายละเอียดเพิ่มเติม |
| เลือก type ให้ถูกต้อง | อ้างอิงผังการตัดสินใจใน Step 345 |
| ระบุ scope เมื่อเป็นประโยชน์ | ช่วยให้เห็น context ได้ทันทีโดยไม่ต้องเปิด diff |
| อธิบาย "ทำไม" ไม่ใช่แค่ "ทำอะไร" ใน body | diff บอก "ทำอะไร" อยู่แล้ว body ควรเสริมเหตุผลเบื้องหลัง |
| ทำเครื่องหมาย breaking change เสมอเมื่อมี | ป้องกันผู้ใช้อัปเดตแล้วโค้ดพังโดยไม่รู้ตัว |
| อ้างอิง issue/ticket ใน footer | เชื่อมโยง commit กับระบบติดตามงานได้อัตโนมัติ |

---

## Step 350: แบบฝึกหัด — เขียน commit ตาม Conventional Commits ให้ถูกต้องอย่างน้อย 10 ครั้งในสถานการณ์ที่กำหนดให้หลากหลาย

ถึงเวลาลงมือฝึกจริง ให้คุณเปิด repository ฝึกฝนที่เตรียมไว้ตั้งแต่ Part 01 (`~/git-course`) แล้วสร้างโฟลเดอร์ใหม่สำหรับ Part นี้:

```bash
mkdir ~/git-course/part-35-commit-convention
cd ~/git-course/part-35-commit-convention
git init
```

### สถานการณ์ที่ 1: เพิ่มฟีเจอร์ใหม่ทั่วไป

คุณเพิ่งเพิ่มฟีเจอร์ให้ผู้ใช้สามารถเปลี่ยนภาษาของแอปพลิเคชันได้ระหว่างไทยและอังกฤษ

```bash
echo "language switcher" > language-switcher.txt
git add language-switcher.txt
git commit -m "feat(i18n): เพิ่มฟีเจอร์เปลี่ยนภาษาระหว่างไทยและอังกฤษ"
```

### สถานการณ์ที่ 2: แก้บั๊กที่มีผลกระทบเฉพาะเจาะจง

คุณพบว่าฟอร์มสมัครสมาชิกยอมรับอีเมลที่ไม่มี `@` ได้ ต้องแก้ validation

```bash
echo "email validation fix" > email-validation.txt
git add email-validation.txt
git commit -m "fix(signup): แก้ validation อีเมลที่ปล่อยให้ผ่านแม้ไม่มีเครื่องหมาย @"
```

### สถานการณ์ที่ 3: แก้ไขเอกสารเท่านั้น

คุณอัปเดตไฟล์ README เพิ่มคำแนะนำการตั้งค่า environment variable

```bash
echo "readme update" > README-update.txt
git add README-update.txt
git commit -m "docs(readme): เพิ่มคำแนะนำการตั้งค่า environment variable สำหรับ local dev"
```

### สถานการณ์ที่ 4: ปรับ format โค้ดล้วน ๆ

ทีมตกลงกันใหม่ให้ใช้ single quote แทน double quote ทั้งโปรเจกต์ โดยไม่มีผลต่อ logic

```bash
echo "single quote style" > style-format.txt
git add style-format.txt
git commit -m "style: เปลี่ยนจาก double quote เป็น single quote ทั้งโปรเจกต์ตาม prettier config ใหม่"
```

### สถานการณ์ที่ 5: Refactor โครงสร้างโค้ด

คุณแยกฟังก์ชันคำนวณภาษีที่ซ้ำกันอยู่ 3 ที่ ให้เหลือฟังก์ชันเดียวที่ใช้ร่วมกัน โดยผลลัพธ์ยังเหมือนเดิมทุกประการ

```bash
echo "extracted tax calculation" > tax-refactor.txt
git add tax-refactor.txt
git commit -m "refactor(billing): รวมฟังก์ชันคำนวณภาษีที่ซ้ำกัน 3 จุดให้เหลือจุดเดียว"
```

### สถานการณ์ที่ 6: เพิ่ม test

คุณเพิ่ม unit test ให้ครอบคลุมกรณีที่ตะกร้าสินค้าว่างเปล่าตอนกดสั่งซื้อ

```bash
echo "empty cart test" > cart-test.txt
git add cart-test.txt
git commit -m "test(cart): เพิ่ม test case สำหรับกรณีกดสั่งซื้อขณะตะกร้าว่างเปล่า"
```

### สถานการณ์ที่ 7: งานดูแลระบบ (dependency)

คุณอัปเดต `express` จากเวอร์ชัน 4.17.1 เป็น 4.19.2 เพื่อปิดช่องโหว่ความปลอดภัยระดับต่ำ

```bash
echo "express 4.19.2" > package-update.txt
git add package-update.txt
git commit -m "chore(deps): อัปเดต express เป็นเวอร์ชัน 4.19.2 เพื่อปิดช่องโหว่ความปลอดภัย"
```

### สถานการณ์ที่ 8: ปรับปรุงประสิทธิภาพ

คุณเพิ่ม index ให้ตาราง `orders` ทำให้ query ที่เคยใช้เวลา 3 วินาที เหลือ 0.2 วินาที

```bash
echo "orders index" > perf-index.txt
git add perf-index.txt
git commit -m "perf(database): เพิ่ม index ให้ตาราง orders ลดเวลา query จาก 3 วินาทีเหลือ 0.2 วินาที"
```

### สถานการณ์ที่ 9: แก้ CI/CD pipeline

Pipeline ของ GitHub Actions รัน test ซ้ำสองรอบโดยไม่จำเป็น ทำให้เสียเวลา build เพิ่มขึ้นเท่าตัว

```bash
echo "ci fix duplicate test run" > ci-fix.txt
git add ci-fix.txt
git commit -m "ci(github-actions): แก้ pipeline ที่รัน test ซ้ำสองรอบโดยไม่จำเป็น"
```

### สถานการณ์ที่ 10: Breaking change เต็มรูปแบบ

ทีมตัดสินใจเปลี่ยนโครงสร้าง response ของ API `/api/v1/products` จาก object เดี่ยวเป็น array พร้อม pagination ซึ่งทำให้ client เดิมที่เรียกใช้ต้องแก้โค้ด

```bash
echo "products api v2 structure" > breaking-api.txt
git add breaking-api.txt
git commit -m "feat(api)!: เปลี่ยนโครงสร้าง response ของ /api/v1/products ให้รองรับ pagination

BREAKING CHANGE: response ของ endpoint /api/v1/products เปลี่ยน
จาก object เดี่ยวที่มี field 'items' เป็น array ของสินค้าโดยตรง
พร้อม header X-Total-Count และ X-Page สำหรับข้อมูล pagination
ผู้ใช้ API ต้องปรับโค้ดฝั่ง client ให้อ่าน response เป็น array แทน

Closes #305"
```

### ตรวจสอบผลลัพธ์ด้วย `git log`

หลังทำครบทั้ง 10 สถานการณ์ ให้ตรวจสอบผลลัพธ์:

```bash
git log --oneline
```

ผลลัพธ์ควรมีหน้าตาประมาณนี้ (เรียงจากล่าสุดไปเก่าสุด):

```
* j1k2l3m feat(api)!: เปลี่ยนโครงสร้าง response ของ /api/v1/products ให้รองรับ pagination
* i9j0k1l ci(github-actions): แก้ pipeline ที่รัน test ซ้ำสองรอบโดยไม่จำเป็น
* h8i9j0k perf(database): เพิ่ม index ให้ตาราง orders ลดเวลา query จาก 3 วินาทีเหลือ 0.2 วินาที
* g7h8i9j chore(deps): อัปเดต express เป็นเวอร์ชัน 4.19.2 เพื่อปิดช่องโหว่ความปลอดภัย
* f6g7h8i test(cart): เพิ่ม test case สำหรับกรณีกดสั่งซื้อขณะตะกร้าว่างเปล่า
* e5f6g7h refactor(billing): รวมฟังก์ชันคำนวณภาษีที่ซ้ำกัน 3 จุดให้เหลือจุดเดียว
* d4e5f6g style: เปลี่ยนจาก double quote เป็น single quote ทั้งโปรเจกต์ตาม prettier config ใหม่
* c3d4e5f docs(readme): เพิ่มคำแนะนำการตั้งค่า environment variable สำหรับ local dev
* b2c3d4e fix(signup): แก้ validation อีเมลที่ปล่อยให้ผ่านแม้ไม่มีเครื่องหมาย @
* a1b2c3d feat(i18n): เพิ่มฟีเจอร์เปลี่ยนภาษาระหว่างไทยและอังกฤษ
```

### แบบฝึกหัดเสริม (ถ้ามีเวลา)

1. ติดตั้ง `commitlint` และ `husky` ตามขั้นตอนใน Step 348 กับ repository ฝึกฝนนี้จริง แล้วลองพิมพ์ commit message ที่ผิดรูปแบบดูว่าถูกปฏิเสธจริงหรือไม่
2. สร้าง branch ตามมาตรฐานที่เรียนใน Step 342–343 สำหรับแต่ละสถานการณ์ข้างต้น เช่น `feature/PROJ-101-language-switcher`, `fix/PROJ-102-email-validation`
3. ลองรัน `npx conventional-changelog -p angular -i CHANGELOG.md -s` ในโฟลเดอร์ฝึกฝนนี้ (ต้องติดตั้ง `conventional-changelog-cli` ก่อน) เพื่อดูว่า Changelog อัตโนมัติที่ generate ออกมาจาก commit history ทั้ง 10 อันของคุณหน้าตาเป็นอย่างไร

### เกณฑ์ตรวจสอบว่าทำถูกต้อง

ก่อนถือว่าแบบฝึกหัดนี้เสร็จสมบูรณ์ ให้ตรวจสอบว่า commit ทั้ง 10 ของคุณ:

- [ ] มี type ที่ถูกต้องตรงกับสถานการณ์ (ใช้ผังการตัดสินใจจาก Step 345 ทวนดูอีกครั้ง)
- [ ] มี scope ที่สื่อความหมายชัดเจน
- [ ] description เขียนเป็นรูปประโยคบอกการกระทำ ไม่ใช้รูปอดีตหรือ passive
- [ ] ความยาวบรรทัดแรกไม่เกิน 72 ตัวอักษรโดยประมาณ
- [ ] มีอย่างน้อย 1 commit ที่เป็น breaking change และเขียนถูกวิธีทั้ง `!` และ footer `BREAKING CHANGE:`
- [ ] แต่ละ commit ทำหน้าที่เดียว ไม่ปนกันหลายเรื่อง

---

## สรุป Part 35

ใน Part นี้เราได้เรียนรู้ว่า:

1. การมีมาตรฐานตั้งชื่อ branch และเขียน commit message ไม่ใช่เรื่องจุกจิก แต่เป็นรากฐานสำคัญที่ทำให้ทีมสื่อสารกันได้อย่างมีประสิทธิภาพ และเปิดทางให้เครื่องมืออัตโนมัติทำงานแทนมนุษย์ได้
2. Branch naming convention มาตรฐานใช้ prefix เช่น `feature/`, `bugfix/`, `hotfix/`, `release/`, `chore/`, `docs/` ตามด้วยคำอธิบายสั้น ๆ ที่สื่อความหมาย
3. การใส่เลข ticket/issue ใน branch name เช่น `feature/JIRA-123-add-login` ช่วยให้ตรวจสอบย้อนกลับและเชื่อมโยงกับระบบติดตามงานได้อัตโนมัติ
4. Conventional Commits มีโครงสร้างมาตรฐานคือ `<type>[optional scope][optional !]: <description>` ตามด้วย body และ footer ที่ไม่บังคับ
5. Type มาตรฐานมี 10 แบบหลัก คือ `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `build`, `ci` แต่ละแบบมีความหมายและผลต่อ Semantic Version ต่างกัน
6. Breaking change แสดงได้ 2 วิธี คือใส่ `!` หลัง type/scope หรือเขียน footer `BREAKING CHANGE:` และมีผลบังคับให้ bump เวอร์ชันแบบ MAJOR เสมอ
7. Conventional Commits เปิดทางให้เครื่องมืออย่าง conventional-changelog สร้าง Changelog อัตโนมัติ และ semantic-release กำหนดเวอร์ชันอัตโนมัติได้ทั้งหมดโดยไม่ต้องพึ่งมนุษย์ตัดสินใจ ซึ่งจะเจาะลึกใน Part 88–89
8. เครื่องมือ commitlint ร่วมกับ husky commit-msg hook ช่วยบังคับใช้มาตรฐานได้จริงในทีม ป้องกัน commit message ที่ผิดรูปแบบตั้งแต่ต้นทาง
9. การฝึกเขียน commit message ที่ดีต้องอาศัยการฝึกฝนซ้ำ ๆ จนกลายเป็นความเคยชิน โดยยึดหลักหนึ่ง commit ทำหน้าที่เดียว และเลือก type ให้ตรงกับเจตนาที่แท้จริงของการเปลี่ยนแปลง

### Checklist ก่อนไป Part 36

- [ ] เข้าใจเหตุผลที่ทีมต้องมีมาตรฐานตั้งชื่อ branch และ commit message
- [ ] สามารถตั้งชื่อ branch ตาม convention `<type>/<TICKET-ID>-<description>` ได้ถูกต้อง
- [ ] จำแนก commit type ทั้ง 10 แบบได้ถูกต้องตามสถานการณ์ที่กำหนด
- [ ] เขียน breaking change ได้ทั้งสองวิธี (`!` และ footer `BREAKING CHANGE:`)
- [ ] เข้าใจว่า Conventional Commits เชื่อมโยงกับ auto-changelog และ semantic-release อย่างไร
- [ ] รู้จักเครื่องมือ commitlint, husky, commitizen และวิธีติดตั้งเบื้องต้น
- [ ] แยกแยะ commit message ที่ดีและแย่ได้ทันทีเมื่อเห็น
- [ ] ฝึกเขียน commit ตาม Conventional Commits ครบ 10 สถานการณ์ในแบบฝึกหัดแล้ว

**ต่อไป:** [Part 36: CODEOWNERS และการกำหนดผู้รับผิดชอบโค้ด](./part-036-codeowners.md)
