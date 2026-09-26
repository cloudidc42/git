# Part 48: GitLab Repository และ Merge Request เบื้องต้น

> **Step ในหลักสูตรนี้:** Step 471–480
> **เฟส:** 5 — เจาะลึก GitLab และ CI/CD เบื้องต้นของมัน
> **เป้าหมายของ Part นี้:** เข้าใจว่า "project" ใน GitLab คืออะไรและต่างจาก "repository" ของ GitHub อย่างไร เชื่อมต่อ local repository เข้ากับ GitLab ได้ทั้งผ่าน HTTPS และ SSH สร้าง Merge Request (MR) แรกได้อย่างครบวงจร เข้าใจ MR description, template, Diff view, Draft MR, Approval Rules, Merge method ทั้ง 3 แบบของ GitLab และวิธีเชื่อม MR กับ Issue ด้วย closing pattern จนสามารถสร้างและ merge MR แรกให้สำเร็จได้ด้วยตัวเอง

---

## สารบัญของ Part นี้

- Step 471: สร้าง Project ใหม่บน GitLab
- Step 472: เชื่อม Local Repository กับ GitLab Remote
- Step 473: Merge Request คืออะไร และขั้นตอนสร้าง MR แรกแบบเต็ม
- Step 474: MR Description และ MR Template
- Step 475: การอ่าน Diff View ใน MR (Changes Tab)
- Step 476: Draft MR — งานที่ยังไม่เสร็จ
- Step 477: Approval Rules ใน MR
- Step 478: Merge Methods ใน GitLab
- Step 479: MR กับ Issue เชื่อมกันอัตโนมัติด้วย Closing Pattern
- Step 480: แบบฝึกหัด — สร้าง Project ใหม่ สร้างและ Merge MR แรกให้สำเร็จครบวงจร

---

## Step 471: สร้าง Project ใหม่บน GitLab

### 471.1 ทำไม GitLab ถึงใช้คำว่า "Project" ไม่ใช่ "Repository"

ใน Part ก่อน ๆ ของหลักสูตรนี้ เราคุ้นเคยกับคำว่า **repository** ของ GitHub ซึ่งหมายถึงที่เก็บซอร์สโค้ดหนึ่งชุดพร้อมประวัติ Git ทั้งหมด แต่เมื่อย้ายมาที่ GitLab คุณจะเห็นว่าทุกที่ในหน้าเว็บใช้คำว่า **"Project"** แทน

นี่ไม่ใช่แค่ชื่อเรียกที่ต่างกันเฉย ๆ แต่สะท้อนแนวคิดที่ต่างกันจริง ๆ:

| แนวคิด | GitHub Repository | GitLab Project |
|---|---|---|
| แก่นกลาง | ที่เก็บ Git repository | ที่เก็บ Git repository **+ หน่วยงานทั้งหมดที่เกี่ยวข้อง** |
| สิ่งที่รวมอยู่ด้วย | Issues, Wiki, Actions (แยกเป็นแท็บ) | Issues, Wiki, CI/CD Pipelines, Container Registry, Package Registry, Security Dashboard, Environments, Feature Flags, Snippets — ทั้งหมดอยู่ใน "project" เดียว |
| ปรัชญา | Repository-centric | DevOps Platform-centric (ครบวงจรตั้งแต่ plan → code → build → test → deploy → monitor) |

พูดง่าย ๆ คือ **Git repository เป็นแค่ "ส่วนหนึ่ง" ของ GitLab project** ไม่ใช่ทั้งหมดของมัน — project ใน GitLab ครอบคลุมทุกอย่างที่เกี่ยวกับวงจรชีวิตซอฟต์แวร์ของงานนั้น ตั้งแต่การวางแผนไปจนถึงการ deploy และ monitor

### 471.2 ขั้นตอนสร้าง Project ใหม่

1. ล็อกอินเข้า GitLab (ไม่ว่าจะเป็น gitlab.com หรือ instance ที่บริษัทของคุณ self-host เอง)
2. คลิกปุ่ม **"New project"** (หรือ "Create new..." ที่มุมบนแล้วเลือก "New project/repository")
3. GitLab จะให้เลือกวิธีสร้าง 4 แบบหลัก:

| ตัวเลือก | ใช้เมื่อไหร่ |
|---|---|
| **Create blank project** | เริ่มโปรเจกต์ใหม่จากศูนย์ (ตัวเลือกที่เราจะใช้ใน Part นี้) |
| **Create from template** | เริ่มจาก template สำเร็จรูป เช่น Rails, Node.js Express, Pages template ต่าง ๆ |
| **Import project** | นำเข้าจากที่อื่น เช่น GitHub, Bitbucket, หรือไฟล์ export ของ GitLab เอง |
| **Run CI/CD for external repository** | เชื่อม GitLab CI/CD เข้ากับ repo ที่ยังอยู่ที่อื่น (เช่น GitHub) โดยไม่ย้ายโค้ดมา |

4. เลือก **Create blank project** แล้วกรอกข้อมูล:

| ฟิลด์ | ความหมาย |
|---|---|
| **Project name** | ชื่อโปรเจกต์ที่แสดงในหน้าเว็บ เช่น `My Awesome App` |
| **Project slug/URL** | ส่วนที่อยู่ใน URL จะถูกสร้างอัตโนมัติจากชื่อ (แก้ไขได้) เช่น `my-awesome-app` |
| **Project deployment target (optional)** | GitLab จะถามว่าจะ deploy ไปที่ไหน (Kubernetes, AWS ฯลฯ) — ข้ามได้ถ้ายังไม่ทราบ |
| **Visibility Level** | ระดับการมองเห็น มี 3 แบบ (อธิบายด้านล่าง) |
| **Initialize repository with a README** | ติ๊กเพื่อให้ GitLab สร้างไฟล์ README.md และ commit แรกให้อัตโนมัติ |

5. คลิก **"Create project"**

### 471.3 Visibility Level ทั้ง 3 แบบ

GitLab มีระดับการมองเห็นที่ละเอียดกว่า GitHub เล็กน้อย เพราะมี 3 ระดับแทนที่จะเป็นแค่ 2 (Public/Private) แบบ GitHub:

| Visibility | ใครเห็นได้บ้าง |
|---|---|
| **Private** | เฉพาะสมาชิกที่ถูกเชิญเข้า project เท่านั้น (เทียบเท่า Private repo ของ GitHub) |
| **Internal** | ผู้ใช้ที่ล็อกอินเข้า GitLab instance นั้นทุกคนเห็นได้ (มีประโยชน์มากในองค์กรที่ self-host GitLab เพราะพนักงานทุกคนดูโค้ดกันเองได้โดยไม่ต้องเชิญทีละคน — **GitHub ไม่มีระดับนี้**) |
| **Public** | ใครก็ได้บนอินเทอร์เน็ตเห็นได้ แม้ไม่ได้ล็อกอิน (เทียบเท่า Public repo ของ GitHub) |

> **หมายเหตุ:** ตั้งแต่ปี 2019 เป็นต้นมา gitlab.com (SaaS) **ปิดการเลือกระดับ Internal สำหรับ project/group ที่สร้างใหม่** ไปแล้ว (เพราะบน SaaS ที่เปิดให้สมัครสมาชิกได้อิสระ คำว่า "Internal" อาจทำให้เข้าใจผิดว่าปลอดภัยกว่าความเป็นจริง) โปรเจกต์เก่าที่เคยตั้งเป็น Internal ไว้ก่อนหน้านั้นยังคงใช้ค่าเดิมได้ แต่จะเปลี่ยนกลับไปเป็น Internal อีกไม่ได้ถ้าเปลี่ยนออกไปแล้ว ส่วนใน self-hosted GitLab (GitLab Self-Managed) ระดับ Internal ยังเลือกใช้งานได้ตามปกติและเป็นค่าที่นิยมใช้มากในองค์กร

### 471.4 โครงสร้าง Namespace ของ GitLab

สิ่งที่ควรรู้อีกอย่างคือ GitLab ใช้ระบบ **namespace** ที่ซ้อนกันเป็นลำดับชั้นได้ลึกกว่า GitHub:

```
gitlab.com/กลุ่มบริษัท/กลุ่มย่อย/ชื่อโปรเจกต์
เช่น
gitlab.com/my-company/backend-team/payment-service
```

นี่คือ **Group** และ **Subgroup** ของ GitLab ซึ่งเทียบได้คร่าว ๆ กับ Organization ของ GitHub แต่ GitLab อนุญาตให้ซ้อน subgroup ได้หลายชั้น ทำให้จัดโครงสร้างทีมใหญ่ ๆ ได้ยืดหยุ่นกว่า (เรื่อง Group/Subgroup แบบละเอียดจะอยู่ใน Part ถัดไปของเฟสนี้)

---

## Step 472: เชื่อม Local Repository กับ GitLab Remote

เมื่อสร้าง project บน GitLab เสร็จแล้ว หน้าจอจะแสดงคำแนะนำให้เชื่อม local repository เข้ากับ project นี้ ซึ่งมี 2 สถานการณ์หลัก

### 472.1 สถานการณ์ที่ 1: มี local repository อยู่แล้ว ต้องการเชื่อมเข้า GitLab

```bash
cd my-awesome-app

# เพิ่ม remote ชื่อ origin ชี้ไปยัง GitLab project
git remote add origin https://gitlab.com/username/my-awesome-app.git

# push branch หลัก (main) ขึ้นไปครั้งแรก พร้อมตั้งให้ track กับ remote
git push -u origin main
```

### 472.2 สถานการณ์ที่ 2: ยังไม่มี local repository เลย

```bash
git clone https://gitlab.com/username/my-awesome-app.git
cd my-awesome-app
```

### 472.3 HTTPS URL vs SSH URL ของ GitLab

GitLab ให้ URL 2 รูปแบบเสมอสำหรับทุก project (คลิกปุ่ม "Clone" บนหน้า project จะเห็นทั้งคู่):

| รูปแบบ | ตัวอย่าง | ต้องยืนยันตัวตนยังไง |
|---|---|---|
| **HTTPS** | `https://gitlab.com/username/my-awesome-app.git` | ต้องใช้ username + Personal Access Token (PAT) เมื่อ push/pull (รหัสผ่านบัญชีปกติใช้ push ไม่ได้แล้วตั้งแต่ GitLab บังคับใช้ PAT/2FA) |
| **SSH** | `git@gitlab.com:username/my-awesome-app.git` | ต้องตั้งค่า SSH key คู่หนึ่งไว้ล่วงหน้า (public key เพิ่มไว้ในหน้า GitLab > Preferences > SSH Keys) แล้วใช้งานได้โดยไม่ต้องใส่รหัสผ่านทุกครั้ง |

**ข้อสังเกตสำคัญ:** รูปแบบ SSH URL ของ GitLab ใช้ `:` คั่นระหว่าง host กับ path (`git@gitlab.com:username/repo.git`) ไม่ใช่ `/` แบบ HTTPS — นี่เป็นรูปแบบมาตรฐานของ SSH protocol ที่ Git ใช้ ไม่ใช่เรื่องเฉพาะของ GitLab เพียงอย่างเดียว (GitHub ก็ใช้รูปแบบเดียวกัน) แต่มือใหม่มักสับสนพิมพ์ผิดเป็น `git@gitlab.com/username/repo.git` ซึ่งจะใช้งานไม่ได้

### 472.4 ตรวจสอบและจัดการ remote

```bash
# ดู remote ทั้งหมดที่ผูกไว้
git remote -v

# ผลลัพธ์ตัวอย่าง
# origin  https://gitlab.com/username/my-awesome-app.git (fetch)
# origin  https://gitlab.com/username/my-awesome-app.git (push)

# เปลี่ยน URL ของ remote เดิม เช่น สลับจาก HTTPS เป็น SSH
git remote set-url origin git@gitlab.com:username/my-awesome-app.git

# ลบ remote
git remote remove origin
```

### 472.5 กรณีมีทั้ง GitHub และ GitLab remote ในโปรเจกต์เดียว

บางทีมต้องการ mirror โค้ดไว้ทั้งสองที่ (เช่น ใช้ GitLab เป็นหลักในองค์กร แต่เปิด mirror บน GitHub ให้ชุมชนภายนอกเห็น) สามารถเพิ่ม remote หลายตัวได้:

```bash
git remote add origin git@gitlab.com:username/my-awesome-app.git
git remote add github git@github.com:username/my-awesome-app.git

git push origin main
git push github main
```

GitLab เองก็มีฟีเจอร์ **Repository Mirroring** ในตัว (Settings > Repository > Mirroring repositories) ที่ตั้งให้ push ไปยัง remote อื่นอัตโนมัติทุกครั้งที่มีการเปลี่ยนแปลง โดยไม่ต้องรันคำสั่งซ้ำสองรอบเอง — รายละเอียดเรื่องนี้จะอยู่ใน Part ที่พูดถึง GitLab ขั้นสูงต่อไป

---

## Step 473: Merge Request คืออะไร และขั้นตอนสร้าง MR แรกแบบเต็ม

### 473.1 Merge Request คืออะไร

**Merge Request (MR)** คือคำที่ GitLab ใช้แทน **Pull Request (PR)** ของ GitHub — แนวคิดเบื้องหลังเหมือนกันทุกประการ:

> **MR คือคำขออย่างเป็นทางการให้นำการเปลี่ยนแปลงจาก branch หนึ่ง (source branch) ไปรวมเข้ากับอีก branch หนึ่ง (target branch) พร้อมเปิดพื้นที่ให้ทีมรีวิวโค้ด แสดงความเห็น และอนุมัติก่อนจะ merge จริง**

ทำไมชื่อถึงต่างกัน? เหตุผลเชิงประวัติศาสตร์คือ GitLab ต้องการเน้นย้ำว่านี่คือ "คำขอให้ merge" (request to merge) ไม่ใช่แค่ "คำขอให้ดึงโค้ด" (request to pull) แม้กลไกเบื้องหลังจะเหมือนกันเกือบทั้งหมดก็ตาม

MR ทำหน้าที่สำคัญหลายอย่างพร้อมกัน:

1. **Code Review** — ให้ทีมตรวจสอบโค้ดก่อนรวมเข้า branch หลัก
2. **Discussion** — พื้นที่พูดคุย ถาม-ตอบ เกี่ยวกับการเปลี่ยนแปลงนั้น
3. **CI/CD Gate** — เชื่อมกับ GitLab CI/CD ให้รัน pipeline ทดสอบก่อนอนุญาตให้ merge
4. **Audit Trail** — เก็บประวัติว่าใครขอ merge อะไร เมื่อไหร่ ใครอนุมัติ
5. **Approval Gate** — บังคับให้ต้องมีคนอนุมัติตามจำนวนที่กำหนดก่อน merge ได้ (รายละเอียดใน Step 477)

### 473.2 ขั้นตอนสร้าง MR แรกแบบเต็ม (ทีละขั้นตอน)

**ขั้นที่ 1: สร้าง feature branch และทำงานตามปกติ**

```bash
git checkout -b feature/add-login-page
# แก้ไขไฟล์ตามที่ต้องการ...
git add .
git commit -m "Add login page UI"
```

**ขั้นที่ 2: push branch ขึ้น GitLab**

```bash
git push -u origin feature/add-login-page
```

**ขั้นที่ 3: สร้าง MR — มี 2 วิธีหลัก**

**วิธีที่ 1 (เร็วที่สุด):** ทันทีที่ push branch ใหม่ GitLab จะแสดงข้อความในผลลัพธ์ terminal เป็นลิงก์พร้อมสร้าง MR ทันที:

```
remote: To create a merge request for feature/add-login-page, visit:
remote:   https://gitlab.com/username/my-awesome-app/-/merge_requests/new?merge_request%5Bsource_branch%5D=feature%2Fadd-login-page
```

คลิกลิงก์นั้นได้เลย หรือถ้าเปิดหน้าเว็บ GitLab อยู่พอดี จะเห็น banner สีฟ้าขึ้นมาบนหน้า project พร้อมปุ่ม **"Create merge request"**

**วิธีที่ 2 (สร้างเองผ่านเมนู):** ไปที่เมนูซ้าย **Code > Merge requests > New merge request** แล้วเลือก:

- **Source branch:** `feature/add-login-page`
- **Target branch:** `main` (หรือ branch หลักของโปรเจกต์)

**ขั้นที่ 4: กรอกรายละเอียด MR**

| ฟิลด์ | คำอธิบาย |
|---|---|
| **Title** | หัวข้อสั้น ๆ บอกว่า MR นี้ทำอะไร (GitLab จะดึงชื่อ commit ล่าสุดมาใส่ให้อัตโนมัติถ้ามี commit เดียว) |
| **Description** | รายละเอียดเต็ม อธิบายว่าทำไมต้องเปลี่ยน เปลี่ยนอะไรบ้าง ทดสอบยังไง (ดู Step 474) |
| **Assignee** | ผู้รับผิดชอบ MR นี้ (มักเป็นตัวผู้เขียนเอง หรือคนที่ต้อง action ต่อ) |
| **Reviewer** | ผู้ที่ถูกขอให้รีวิวโค้ดโดยเฉพาะ (แยกจาก Assignee ชัดเจน — ฟีเจอร์นี้เป็นจุดต่างจาก GitHub ที่รวม reviewer ไว้ในช่องเดียว) |
| **Milestone** | ผูกกับ milestone ที่กำหนดไว้ |
| **Labels** | ป้ายกำกับ เช่น `bug`, `feature`, `needs-review` |
| **Merge options** | เช่น "Delete source branch when merge request is accepted", "Squash commits when merge request is accepted" |

**ขั้นที่ 5: คลิก "Create merge request"**

เมื่อสร้างเสร็จ MR จะได้เลขที่อ้างอิงเฉพาะโปรเจกต์นั้น เช่น `!42` (สังเกตว่า GitLab ใช้เครื่องหมาย `!` แทน MR ต่างจาก Issue ที่ใช้ `#`)

### 473.3 ตารางสรุปคำศัพท์ MR ที่ต้องรู้

| คำศัพท์ | ความหมาย |
|---|---|
| **Source branch** | branch ต้นทางที่มีการเปลี่ยนแปลงที่อยากจะ merge |
| **Target branch** | branch ปลายทางที่จะรับการเปลี่ยนแปลงเข้าไป |
| **Overview tab** | แท็บแรกของ MR แสดง description, discussion, approval status, pipeline status |
| **Commits tab** | รายการ commit ทั้งหมดที่อยู่ใน MR นี้ |
| **Pipelines tab** | สถานะ CI/CD pipeline ที่รันกับ MR นี้ |
| **Changes tab** | หน้า diff แสดงไฟล์ทั้งหมดที่เปลี่ยนแปลง (ดู Step 475) |
| **`!` prefix** | สัญลักษณ์อ้างอิง MR ในระบบ GitLab (เทียบกับ `#` ของ Issue) |

---

## Step 474: MR Description และ MR Template

### 474.1 ความสำคัญของ MR Description ที่ดี

Description ของ MR ที่ดีช่วยให้ผู้รีวิวเข้าใจบริบทได้เร็วโดยไม่ต้องไล่อ่านโค้ดทั้งหมดก่อน โครงสร้างที่แนะนำโดยทั่วไป:

```markdown
## What does this MR do?
เพิ่มหน้า Login พร้อมระบบตรวจสอบ email/password เบื้องต้น

## Why is this change needed?
ผู้ใช้ยังไม่มีทางล็อกอินเข้าระบบได้เลยในเวอร์ชันปัจจุบัน

## How was this tested?
- รัน unit test ที่เพิ่มใหม่ทั้งหมด: `npm test`
- ทดสอบด้วยมือบน Chrome และ Firefox

## Screenshots (ถ้ามี UI เปลี่ยนแปลง)
(แนบรูปภาพ)

## Related issues
Closes #123
```

Description ของ GitLab รองรับ **Markdown เต็มรูปแบบ** เหมือน Issue รวมถึง:

- Task list (`- [ ]`, `- [x]`) — แสดงเป็น progress bar อัตโนมัติที่หัว MR
- การอ้างอิง user ด้วย `@username`
- การอ้างอิง Issue/MR อื่นด้วย `#` และ `!`
- การแนบรูปภาพ วิดีโอ โดยลากไฟล์วางลงในกล่องข้อความได้เลย
- Code block พร้อม syntax highlighting

### 474.2 MR Template คืออะไร

แทนที่จะให้ทุกคนพิมพ์ description จากศูนย์ทุกครั้ง GitLab อนุญาตให้ทีมสร้าง **MR Template** ไว้ล่วงหน้า เพื่อบังคับโครงสร้างมาตรฐานให้ทุก MR ในโปรเจกต์

**วิธีสร้าง:** สร้างไฟล์ Markdown ไว้ในโฟลเดอร์:

```
.gitlab/merge_request_templates/
```

ตัวอย่างเช่น สร้างไฟล์ `.gitlab/merge_request_templates/Default.md`:

```markdown
## Summary

<!-- อธิบายสั้น ๆ ว่า MR นี้ทำอะไร -->

## Changes

<!-- รายการการเปลี่ยนแปลงหลัก -->
-
-

## Checklist

- [ ] เพิ่ม/แก้ไข unit test แล้ว
- [ ] อัปเดตเอกสารที่เกี่ยวข้องแล้ว (ถ้ามี)
- [ ] ทดสอบด้วยมือแล้ว
- [ ] ไม่มี breaking change หรือระบุไว้ชัดเจนแล้วถ้ามี

## Related issue

Closes #
```

### 474.3 กติกาสำคัญของ Template

| กติกา | รายละเอียด |
|---|---|
| **ชื่อไฟล์ `Default.md`** | ถ้ามีไฟล์ชื่อนี้อยู่ใน `.gitlab/merge_request_templates/` มันจะถูกใส่ลงในช่อง description **โดยอัตโนมัติ** ทุกครั้งที่สร้าง MR ใหม่ โดยไม่ต้องเลือกเอง |
| **มีหลาย template ได้** | สร้างไฟล์ชื่ออื่นเพิ่มได้ เช่น `Bugfix.md`, `Feature.md`, `Hotfix.md` — ผู้สร้าง MR จะเห็น dropdown "Choose a template" ให้เลือกใช้ตามประเภทงาน |
| **ต้อง commit เข้า default branch** | template จะมีผลก็ต่อเมื่อไฟล์นั้นอยู่ใน default branch ของ project (ปกติคือ `main`) เท่านั้น |
| **แยกจาก Issue template** | Issue template อยู่คนละโฟลเดอร์ คือ `.gitlab/issue_templates/` — อย่าสับสนสองโฟลเดอร์นี้ |
| **Group-level template** | ถ้าต้องการ template เดียวใช้ร่วมกันหลาย project ในกลุ่มเดียวกัน สามารถตั้งค่า project หนึ่งเป็น "template repository" ของ group แล้วให้ project อื่นดึงมาใช้ได้ (Settings > Merge requests ระดับ Group) |

### 474.4 เปรียบเทียบกับ GitHub

GitHub ใช้ตำแหน่งไฟล์ใกล้เคียงกันคือ `.github/pull_request_template.md` หรือโฟลเดอร์ `.github/PULL_REQUEST_TEMPLATE/` สำหรับหลาย template — แนวคิดเหมือนกันทุกประการ ต่างกันแค่ชื่อโฟลเดอร์ (`.gitlab/` vs `.github/`) และคำเรียก (merge request vs pull request)

---

## Step 475: การอ่าน Diff View ใน MR (Changes Tab)

### 475.1 ตำแหน่งของ Diff View

เมื่อเปิด MR ขึ้นมา จะเห็นแท็บด้านบนหลายแท็บ: **Overview**, **Commits**, **Pipelines**, **Changes** — แท็บ **Changes** คือหน้าที่แสดง diff ทั้งหมดของ MR นี้ ซึ่งเป็นหน้าที่ผู้รีวิวใช้เวลาส่วนใหญ่อยู่ตรงนี้

### 475.2 องค์ประกอบสำคัญของหน้า Changes

| ส่วนประกอบ | หน้าที่ |
|---|---|
| **จำนวนไฟล์ที่เปลี่ยน** | แสดงที่หัวแท็บ เช่น "Changes 5" หมายถึงมี 5 ไฟล์ที่แก้ไข |
| **สรุปบวก/ลบบรรทัด** | เช่น `+120 −34` สีเขียวคือบรรทัดที่เพิ่ม สีแดงคือบรรทัดที่ลบ |
| **File tree ด้านซ้าย** | รายการไฟล์ทั้งหมดที่เปลี่ยน คลิกเพื่อกระโดดไปดู diff ของไฟล์นั้นโดยตรง |
| **Inline vs Side-by-side** | สลับมุมมอง diff ได้ 2 แบบ ที่ปุ่มมุมขวาบน |

### 475.3 Inline View vs Side-by-side View

| มุมมอง | ลักษณะ | เหมาะกับ |
|---|---|---|
| **Inline** | แสดงบรรทัดเก่า (สีแดง) กับบรรทัดใหม่ (สีเขียว) เรียงต่อกันในคอลัมน์เดียว | ไฟล์ที่มีการแก้ไขเล็กน้อยกระจายทั่วไฟล์ อ่านลำดับเหตุการณ์ได้ง่าย |
| **Side-by-side (Parallel)** | แบ่งจอเป็น 2 คอลัมน์ ซ้ายคือโค้ดเก่า ขวาคือโค้ดใหม่ วางเทียบกัน | ไฟล์ที่มีการจัดโครงสร้างใหม่เยอะ อยากเทียบตำแหน่งตรง ๆ |

### 475.4 การคอมเมนต์บน Diff (Inline Comments)

คลิกที่เครื่องหมาย `+` ข้างเลขบรรทัดในหน้า diff เพื่อเปิดกล่องคอมเมนต์เฉพาะบรรทัดนั้น สามารถ:

- คอมเมนต์บรรทัดเดียว หรือลากเลือกหลายบรรทัดเพื่อคอมเมนต์เป็นช่วง (multi-line comment)
- เลือก **"Start a review"** เพื่อรวบรวมความเห็นหลายจุดไว้ก่อน แล้วส่งพร้อมกันทีเดียว (ไม่ยิงแจ้งเตือนทีละคอมเมนต์ให้ผู้เขียน MR รำคาญ) — เทียบเท่ากับ "Review" ของ GitHub
- แก้ปัญหาเสร็จแล้วกด **"Resolve thread"** เพื่อปิดกล่องสนทนานั้น ทำให้ MR ดูสะอาดขึ้นเมื่อมีคอมเมนต์เยอะ

### 475.5 ตัวเลือกเสริมที่มีประโยชน์ในหน้า Diff

| ตัวเลือก | ประโยชน์ |
|---|---|
| **Show whitespace changes / Hide whitespace changes** | ซ่อนการเปลี่ยนแปลงที่เป็นแค่ space/tab เพื่อโฟกัสเนื้อหาจริง |
| **Expand context (ไอคอนลูกศรระหว่าง hunk)** | ขยายดูโค้ดรอบ ๆ จุดที่เปลี่ยน เพื่อดูบริบทเต็มไฟล์โดยไม่ต้องเปิดไฟล์แยก |
| **View file @ commit hash** | ดูไฟล์ทั้งไฟล์ ณ commit นั้นแบบเต็ม ไม่ใช่แค่ diff |
| **File-by-file diff navigation** | ปุ่มลูกศรเลื่อนไปไฟล์ถัดไป/ก่อนหน้าโดยไม่ต้อง scroll ยาว |
| **Compare with previous version** | ถ้า MR มีการ push เพิ่มหลังจากเปิดรีวิวแล้ว สามารถเลือกดู diff เทียบเฉพาะ "การเปลี่ยนแปลงใหม่ที่เพิ่งเพิ่มเข้ามา" แทนที่จะไล่ดูทั้งหมดซ้ำ |

---

## Step 476: Draft MR — งานที่ยังไม่เสร็จ

### 476.1 ปัญหาที่ Draft MR แก้ไข

บางครั้งคุณอยากเปิด MR ตั้งแต่เนิ่น ๆ เพื่อ:

- ให้ทีมเห็นทิศทางที่กำลังทำอยู่ (early feedback)
- ให้ CI/CD รัน pipeline ทดสอบระหว่างทาง
- กันไม่ให้คนอื่นเผลอ merge งานที่ยังไม่เสร็จเข้า branch หลัก

**Draft MR** คือกลไกที่ทำให้ MR แสดงสถานะชัดเจนว่า **"ยังไม่พร้อม merge"** โดยปุ่ม Merge จะถูกปิดใช้งานไว้จนกว่าจะปลด draft ออก

### 476.2 ประวัติชื่อ: จาก "WIP:" สู่ "Draft:"

ในอดีต GitLab (และ GitHub) ใช้วิธีให้ผู้ใช้พิมพ์คำนำหน้า title ด้วยตัวเองว่า `WIP:` (ย่อจาก Work In Progress) เพื่อสื่อว่า MR ยังไม่เสร็จ เช่น:

```
WIP: Add login page
```

ต่อมา GitLab ได้เปลี่ยนมาตรฐานอย่างเป็นทางการเป็นคำว่า **"Draft:"** แทน เพราะสื่อความหมายชัดเจนกว่า และเชื่อมกับปุ่ม/checkbox ในหน้า UI โดยตรง:

```
Draft: Add login page
```

> **สำคัญ:** ปัจจุบัน GitLab ยังรองรับ backward compatibility คือถ้า MR title ขึ้นต้นด้วย `WIP:` ระบบก็ยังตีความว่าเป็น draft เหมือนกัน แต่ผู้ใช้ใหม่ควรใช้คำว่า **Draft:** เป็นมาตรฐาน เพราะเป็นคำที่ UI ปัจจุบันใช้เรียกฟีเจอร์นี้ทั้งหมด

### 476.3 วิธีทำให้ MR เป็น Draft

มี 3 วิธีที่เทียบเท่ากัน:

1. **พิมพ์ `Draft:` นำหน้า title** ตอนสร้างหรือแก้ไข MR โดยตรง
2. **ติ๊กช่อง "Mark as draft"** ที่อยู่ใต้ช่อง title ตอนสร้าง MR
3. **คลิกปุ่ม "Mark as draft"** จากเมนู `...` (more actions) บน MR ที่เปิดอยู่แล้ว

### 476.4 วิธีปลด Draft เมื่อพร้อม merge

เมื่อทำงานเสร็จแล้ว จะเห็นปุ่มสีฟ้าเด่นชัดตรงส่วนบนของ MR ชื่อ **"Mark as ready"** — คลิกปุ่มนี้ GitLab จะ:

- ลบคำว่า `Draft:` ออกจาก title ให้อัตโนมัติ
- เปิดใช้งานปุ่ม Merge ให้กลับมากดได้ (ถ้าเงื่อนไขอื่นผ่านครบ เช่น approval, pipeline สำเร็จ)

### 476.5 พฤติกรรมของปุ่ม Merge เมื่อเป็น Draft

ตราบใดที่ MR ยังเป็น Draft อยู่ ปุ่ม **"Merge"** จะแสดงเป็นสีเทาและกดไม่ได้ พร้อมข้อความอธิบายว่า *"This merge request is still a draft."* แม้ผู้ใช้จะมีสิทธิ์ merge และเงื่อนไขอื่นผ่านหมดแล้วก็ตาม — เป็นกลไกป้องกันความผิดพลาดของมนุษย์ (human error) ที่มีประโยชน์มากในทีมที่มี pipeline auto-merge เปิดใช้งานอยู่

---

## Step 477: Approval Rules ใน MR

### 477.1 Approval Rules คืออะไร

**Approval Rules** คือกลไกที่บังคับว่า MR หนึ่ง ๆ ต้องได้รับการอนุมัติ (approve) จากจำนวนคนขั้นต่ำที่กำหนดไว้ ก่อนที่ปุ่ม Merge จะสามารถกดได้ — เป็นเครื่องมือควบคุมคุณภาพโค้ดในระดับกระบวนการ (process-level) ไม่ใช่แค่ระดับเทคนิค

ตั้งค่าได้ที่ **Project > Settings > Merge requests > Merge request approvals**

### 477.2 องค์ประกอบหลักของ Approval Rule

| องค์ประกอบ | ความหมาย |
|---|---|
| **Approvals required** | จำนวนอนุมัติขั้นต่ำที่ต้องได้ก่อนจะ merge ได้ เช่น ตั้งเป็น `2` แปลว่าต้องมีคนกด Approve อย่างน้อย 2 คน |
| **Eligible approvers** | รายชื่อคนที่ "มีสิทธิ์" กดอนุมัติได้ โดยทั่วไปคือสมาชิก project ที่มี role **Developer ขึ้นไป** (role ต่ำกว่านั้นอย่าง Reporter หรือ Guest กดอนุมัติไม่ได้) |
| **Target branch** | สามารถกำหนดให้ rule นี้มีผลเฉพาะ MR ที่ target ไปยัง branch บางแบบเท่านั้น เช่น บังคับ approval เข้มงวดเฉพาะตอน merge เข้า `main` แต่ผ่อนปรนกับ branch อื่น |
| **Specific approvers** | (Premium/Ultimate) ระบุรายชื่อบุคคลหรือกลุ่มที่ต้องอนุมัติโดยเฉพาะ ไม่ใช่แค่ "ใครก็ได้ในทีม" |
| **Code Owner approval** | (Premium/Ultimate) บังคับให้เจ้าของโค้ดในไฟล์ `CODEOWNERS` ต้องอนุมัติก่อน หากมีการแก้ไฟล์ที่ตนดูแล |

> **หมายเหตุเรื่อง tier:** ฟีเจอร์ approval rule ขั้นพื้นฐาน (กำหนดจำนวน approval ขั้นต่ำแบบ "All eligible users" หนึ่ง rule) ใช้ได้ในทุก tier รวมถึง GitLab Free แต่การสร้างหลาย rule พร้อมกัน กำหนด approver เฉพาะเจาะจง หรือผูกกับ Code Owners เป็นฟีเจอร์ระดับ **Premium** ขึ้นไป

### 477.3 กติกาสำคัญเรื่องสิทธิ์การอนุมัติ

| กติกา | ค่าเริ่มต้น |
|---|---|
| **ผู้เขียน MR อนุมัติ MR ของตัวเองได้ไหม** | ไม่ได้ — ค่าเริ่มต้นห้าม author กด approve MR ของตัวเอง |
| **คนที่ push commit เพิ่มเข้า MR อนุมัติได้ไหม** | มีตัวเลือก "Prevent approval by users who add commits" เปิดไว้เพื่อป้องกันไม่ให้ผู้ที่แก้โค้ดเองอนุมัติงานตัวเองผ่านการ push commit เพิ่ม |
| **เมื่อมี commit ใหม่ถูก push เข้า MR ที่เคยอนุมัติแล้ว** | มีตัวเลือก "Require re-approval when new commits are added" — ถ้าเปิดไว้ approval เดิมจะถูกล้างทิ้ง (reset) ต้องให้อนุมัติใหม่อีกครั้งเพื่อความปลอดภัย |

### 477.4 สถานะที่แสดงบนหน้า MR

เมื่อ approval rule มีผล จะเห็นแถบสถานะบนหัว MR เช่น:

```
Approval is optional
```
หรือ
```
Approved by @somchai. 1 of 2 approvals required.
```

เมื่อครบจำนวนที่กำหนด สถานะจะเปลี่ยนเป็นเครื่องหมายถูกสีเขียว และปุ่ม Merge จะพร้อมใช้งาน (ถ้าเงื่อนไขอื่น เช่น pipeline สำเร็จ ก็ผ่านด้วย)

### 477.5 เปรียบเทียบกับ GitHub

GitHub เรียกกลไกนี้ว่า **"Required reviewers"** ภายใต้ Branch Protection Rules (ตั้งค่าที่ Settings > Branches) แนวคิดคล้ายกันมาก คือกำหนดจำนวน reviewer ขั้นต่ำที่ต้อง approve ก่อน merge ได้ — ต่างกันตรงที่ GitLab แยกการตั้งค่า approval ออกมาเป็นเมนูของตัวเองชัดเจน (Merge request approvals) ในขณะที่ GitHub ฝังไว้เป็นส่วนหนึ่งของ Branch protection rule

---

## Step 478: Merge Methods ใน GitLab

### 478.1 ภาพรวม: GitLab มี Merge Method ให้เลือก 3 แบบ

ตั้งค่าได้ที่ **Project > Settings > Merge requests > Merge method** ซึ่งมีผลกับ**ทุก MR** ในโปรเจกต์นั้น (ไม่สามารถเลือกต่าง merge method กันเป็นราย MR ได้ ต่างจาก GitHub ที่ให้เลือกตอนกด merge แต่ละครั้ง)

| Merge Method | พฤติกรรม |
|---|---|
| **Merge commit** (ค่าเริ่มต้น) | สร้าง merge commit เสมอ แม้ว่า source branch จะ fast-forward ได้พอดีก็ตาม ประวัติจะเห็นจุดแตกและจุดรวมของ branch ชัดเจน |
| **Merge commit with semi-linear history** | **สร้าง merge commit เสมอทุกครั้ง** (เหมือนกับ Merge commit) แต่มีเงื่อนไขเพิ่มว่า**จะยอมให้กด merge ได้ก็ต่อเมื่อ source branch สามารถ fast-forward เข้ากับ target branch ได้แล้วเท่านั้น** ถ้า target branch มีการเปลี่ยนแปลงใหม่กว่าจนไม่สามารถ fast-forward ได้ ผู้ใช้ต้อง**rebase source branch ก่อน**ถึงจะกด merge ได้ — ทำให้มั่นใจได้ว่าถ้า pipeline ของ MR ผ่าน หลัง merge แล้ว pipeline ของ target branch ก็จะผ่านเช่นกัน |
| **Fast-forward merge** | ไม่มีการสร้าง merge commit เลย ประวัติเป็นเส้นตรงสมบูรณ์ (linear history) — แต่มีข้อจำกัดว่า MR ต้องสามารถ fast-forward ได้จริงเท่านั้น (ถ้า target branch เปลี่ยนไปแล้ว ต้อง rebase ก่อนถึงจะ merge ได้) |

### 478.2 Squash Commits: ตัวเลือกที่แยกออกมาต่างหาก (จุดที่มือใหม่มักเข้าใจผิด)

**ข้อควรระวังสำคัญ:** ใน GitLab **"Squash"" ไม่ใช่ merge method ตัวที่ 4** แบบที่หลายคนเข้าใจผิดโดยเทียบกับ GitHub — แต่เป็น **checkbox แยกต่างหาก** ที่ตั้งค่าคู่กับ merge method ใดก็ได้ทั้ง 3 แบบข้างต้น

ตั้งค่าที่ **Settings > Merge requests > Squash commits when merging** มีตัวเลือก:

| ค่า | ความหมาย |
|---|---|
| **Do not allow** | ห้าม squash เด็ดขาด ทุก commit ใน MR จะถูกเก็บแยกไว้ตามเดิม |
| **Allow** | ผู้สร้าง MR เลือกได้เองว่าจะ squash หรือไม่ตอนสร้าง/merge MR |
| **Encourage** | ตั้ง checkbox squash ให้ติ๊กไว้เป็นค่าเริ่มต้น แต่ผู้ใช้ยังปลดออกได้ |
| **Require** | บังคับ squash เสมอ ไม่ให้ผู้ใช้ปิดตัวเลือกนี้ได้ |

เมื่อเปิด squash ไว้ ไม่ว่าจะเลือก merge method แบบไหนก็ตาม ผลลัพธ์คือ commit ทั้งหมดใน source branch จะถูกรวมเป็น 1 commit เดียวก่อนนำไป merge ตาม method ที่เลือกไว้อีกที

### 478.3 ตารางเปรียบเทียบกับ GitHub

| แนวคิด | GitHub | GitLab |
|---|---|---|
| จุดที่ตั้งค่า merge method | เลือกได้ **ต่อครั้ง** ตอนกดปุ่ม merge (ถ้า repo เปิดให้เลือกได้มากกว่า 1 วิธี) | ตั้งค่าเป็น **ค่าคงที่ของทั้ง project** ที่ระดับ Settings (ผู้ merge ไม่สามารถเลือกวิธีอื่นนอกจากที่ตั้งไว้ได้ ยกเว้นเรื่อง squash) |
| ตัวเลือกที่มี | Merge commit, Squash and merge, Rebase and merge (3 ปุ่มแยกกัน) | Merge commit, Merge commit with semi-linear history, Fast-forward merge (เลือก method หลัก) + Squash เป็น checkbox แยก |
| Rebase and merge ของ GitHub | สร้างประวัติเป็นเส้นตรง โดย rebase commit ทีละตัวเข้า target แล้ว fast-forward — ไม่มี merge commit | เทียบเท่ากับ **Fast-forward merge** ของ GitLab (ผลลัพธ์ประวัติคล้ายกันมาก) |
| Squash and merge ของ GitHub | ปุ่มเดียวจบ รวม commit ทั้งหมดเป็น 1 แล้ว merge | ต้องเปิด "Squash" checkbox ควบคู่กับ merge method ที่เลือกไว้ |

### 478.4 ควรเลือกแบบไหนสำหรับทีมของคุณ

| สถานการณ์ | คำแนะนำ |
|---|---|
| ทีมที่อยากเห็นประวัติทุกจุดที่ merge ชัดเจนเพื่อ audit | **Merge commit** (ค่าเริ่มต้น) |
| ทีมที่อยากได้ประวัติสะอาดกว่าเดิม แต่ยังอยากรู้ว่าจุดไหนคือจุดรวมงาน | **Merge commit with semi-linear history** |
| ทีมที่ยึดหลัก linear history อย่างเคร่งครัด (เหมือนโปรเจกต์ที่เน้น `git log --oneline` อ่านง่ายเป็นเส้นตรง) | **Fast-forward merge** ร่วมกับ **Squash: Require** |

---

## Step 479: MR กับ Issue เชื่อมกันอัตโนมัติด้วย Closing Pattern

### 479.1 Closing Pattern คืออะไร

GitLab อนุญาตให้พิมพ์คำสั่งพิเศษ (keyword) ตามด้วยเลข issue ไว้ใน **MR description** เพื่อสั่งให้ GitLab **ปิด issue นั้นโดยอัตโนมัติทันทีที่ MR ถูก merge เข้า default branch ของ project**

```markdown
Closes #123
```

### 479.2 คำสั่ง (keyword) ที่ใช้ได้ทั้งหมด

GitLab รองรับคำหลายคำที่มีความหมายเดียวกัน เพื่อให้เขียนได้เป็นธรรมชาติในประโยค:

```
Close, Closes, Closed, Closing
Fix, Fixes, Fixed, Fixing
```

ตัวอย่างที่ใช้ได้ทั้งหมดนี้จะทำงานเหมือนกัน:

```markdown
Closes #123
Fixes #123
This MR closes #123
Fixed #123 and #456
```

### 479.3 ปิดหลาย Issue พร้อมกัน

คั่นด้วยเครื่องหมายจุลภาค (comma) หรือคำว่า "and":

```markdown
Closes #123, #124, #125
```

### 479.4 ปิด Issue ข้าม Project (Cross-project closing)

ถ้า Issue อยู่ใน project อื่น ต้องระบุ path เต็มของ project นั้นก่อนเครื่องหมาย `#`:

```markdown
Closes group/other-project#45
```

### 479.5 เงื่อนไขสำคัญที่ต้องเข้าใจให้ถูกต้อง

| เงื่อนไข | รายละเอียด |
|---|---|
| **ต้อง merge เข้า default branch เท่านั้น** | ถ้า MR ถูก merge เข้า branch อื่นที่ไม่ใช่ default branch ของ project (เช่น merge เข้า `develop` แทนที่จะเป็น `main`) closing pattern **จะไม่ทำงาน** — Issue จะไม่ถูกปิดอัตโนมัติ |
| **การ "reference" อย่างเดียวไม่ปิด issue** | ถ้าพิมพ์แค่ `#123` เฉย ๆ โดยไม่มีคำสั่งปิดนำหน้า จะเป็นแค่การ**อ้างอิง** (mention/link) เชื่อมโยงให้เห็นความสัมพันธ์กัน แต่ **จะไม่ปิด issue** |
| **ใส่ได้ทั้งใน title และ description** | closing pattern ทำงานได้ทั้งในช่อง title และ description ของ MR |
| **แก้ไขภายหลังได้** | ถ้าเผลอลืมใส่ตอนสร้าง MR สามารถแก้ description เพิ่ม `Closes #123` เข้าไปทีหลังได้ตราบใดที่ MR ยังไม่ถูก merge |
| **ยกเลิกได้ก่อน merge** | ถ้าลบข้อความ `Closes #123` ออกจาก description ก่อน merge จริง Issue นั้นจะไม่ถูกปิดอัตโนมัติ |

### 479.6 ประโยชน์ของการเชื่อม MR กับ Issue

1. **Traceability สมบูรณ์** — เปิด Issue ขึ้นมาจะเห็นลิงก์ MR ที่เกี่ยวข้องโดยอัตโนมัติ ย้อนดูได้ว่างานนี้แก้ด้วยโค้ดส่วนไหน
2. **ลดงานที่ต้องทำมือ** — ไม่ต้องเข้าไปปิด Issue เองทีละอันหลัง deploy
3. **Board อัปเดตอัตโนมัติ** — ถ้าใช้ GitLab Issue Board การปิด Issue อัตโนมัติจะย้ายการ์ดไปคอลัมน์ "Closed"/"Done" ให้ทันที ทำให้ทีมเห็นความคืบหน้าโปรเจกต์แบบเรียลไทม์โดยไม่ต้องอัปเดตบอร์ดด้วยมือ (รายละเอียดเรื่อง Board จะอยู่ใน **Part 49**)

---

## Step 480: แบบฝึกหัด — สร้าง Project ใหม่ สร้างและ Merge MR แรกให้สำเร็จครบวงจร

ถึงเวลาลงมือทำจริงทั้งกระบวนการตั้งแต่ต้นจนจบ ทำตามลำดับต่อไปนี้ทีละขั้นตอน

### 480.1 ขั้นที่ 1: สร้าง Project ใหม่บน GitLab

1. เข้า GitLab แล้วสร้าง project ใหม่ชื่อ `git-course-part48-practice`
2. ตั้ง Visibility เป็น **Private**
3. ติ๊ก **Initialize repository with a README**
4. คลิก Create project

### 480.2 ขั้นที่ 2: Clone project มาที่เครื่อง

```bash
git clone git@gitlab.com:<username>/git-course-part48-practice.git
cd git-course-part48-practice
```

(ถ้ายังไม่ได้ตั้งค่า SSH key ไว้ ให้ใช้ HTTPS URL แทนได้ชั่วคราว)

### 480.3 ขั้นที่ 3: เพิ่ม MR Template ให้ project

```bash
mkdir -p .gitlab/merge_request_templates
```

สร้างไฟล์ `.gitlab/merge_request_templates/Default.md` ใส่เนื้อหา:

```markdown
## Summary


## Changes
-

## Checklist
- [ ] ทดสอบแล้ว
- [ ] อัปเดตเอกสารแล้ว (ถ้าจำเป็น)

## Related issue
Closes #
```

commit และ push เข้า `main` โดยตรง (เพราะยังไม่มีใครทำงานอยู่บน branch นี้):

```bash
git add .gitlab
git commit -m "Add default merge request template"
git push origin main
```

### 480.4 ขั้นที่ 4: สร้าง Issue ไว้ทดสอบ closing pattern

ไปที่เมนู **Issues > New issue** สร้าง Issue ชื่อ `เพิ่มไฟล์ CONTRIBUTING.md` แล้วบันทึกเลข issue ที่ได้ (เช่น `#1`)

### 480.5 ขั้นที่ 5: สร้าง Feature Branch และทำงาน

```bash
git checkout -b feature/add-contributing-guide
```

สร้างไฟล์ `CONTRIBUTING.md`:

```markdown
# Contributing Guide

ขอบคุณที่สนใจร่วมพัฒนาโปรเจกต์นี้!

## ขั้นตอนการส่งงาน
1. Fork หรือสร้าง branch ใหม่จาก main
2. ทำการเปลี่ยนแปลง
3. เปิด Merge Request พร้อมอธิบายรายละเอียดให้ชัดเจน
```

```bash
git add CONTRIBUTING.md
git commit -m "Add CONTRIBUTING.md"
git push -u origin feature/add-contributing-guide
```

### 480.6 ขั้นที่ 6: สร้าง MR

1. คลิกลิงก์ที่ terminal แสดงให้ (หรือไปที่ Merge requests > New merge request)
2. ตรวจสอบ source branch = `feature/add-contributing-guide`, target branch = `main`
3. สังเกตว่า description ถูกเติมด้วย template ที่สร้างไว้ในขั้นที่ 3 โดยอัตโนมัติ — กรอกรายละเอียดให้ครบ
4. ในช่อง "Related issue" พิมพ์:
   ```
   Closes #1
   ```
   (แทน `#1` ด้วยเลข issue จริงที่คุณสร้างไว้)
5. ทดลองติ๊ก **"Mark as draft"** ก่อน แล้วสร้าง MR ขึ้นมาดูสถานะ Draft
6. คลิกปุ่ม **"Mark as ready"** เพื่อปลด draft ออก
7. เปิดแท็บ **Changes** ดู diff ของไฟล์ `CONTRIBUTING.md` ที่เพิ่มเข้ามา ลองสลับดูทั้งมุมมอง Inline และ Side-by-side

### 480.7 ขั้นที่ 7: ตั้งค่า Approval Rule (ถ้าทำงานคนเดียว ให้ข้ามหรือปรับเป็น 0)

ถ้าทำงานคนเดียว ให้ทดลองเข้าไปดูหน้า **Settings > Merge requests > Merge request approvals** เพื่อดูว่าตั้งค่าจำนวน approval ขั้นต่ำได้ตรงไหน (ไม่จำเป็นต้องตั้งเป็น 1 ขึ้นไปถ้าไม่มีสมาชิกคนอื่นในทีม เพราะจะทำให้ merge เองไม่ได้)

ถ้าทำงานเป็นทีม ให้เชิญเพื่อนร่วมทีมเข้า project (Settings > Members) แล้วตั้ง approval required = 1 แล้วให้เพื่อนกด Approve บน MR ที่สร้างไว้

### 480.8 ขั้นที่ 8: ตรวจสอบ Merge Method ของ Project

ไปที่ **Settings > Merge requests > Merge method** ดูว่าปัจจุบันตั้งเป็นค่าอะไร (ค่าเริ่มต้นคือ Merge commit) ลองเปลี่ยนเป็น **Fast-forward merge** แล้วสังเกตว่า GitLab อาจแจ้งเตือนถ้า MR นี้ไม่สามารถ fast-forward ได้ทันที (กรณีนี้ควรจะ fast-forward ได้เพราะยังไม่มีใครแก้ `main` เพิ่มระหว่างทาง)

### 480.9 ขั้นที่ 9: Merge MR ให้สำเร็จ

1. ตรวจสอบว่าปุ่ม **Merge** เป็นสีเขียวพร้อมกดได้ (ไม่ใช่ Draft, ผ่าน approval ครบ, pipeline ผ่านถ้ามี)
2. ติ๊กหรือไม่ติ๊ก **"Delete source branch"** ตามที่ต้องการ (แนะนำให้ติ๊กไว้เพื่อความสะอาดของ repository)
3. คลิก **Merge**

### 480.10 ขั้นที่ 10: ตรวจสอบผลลัพธ์

หลัง merge เสร็จ ให้ตรวจสอบสิ่งต่อไปนี้:

```bash
git checkout main
git pull origin main
cat CONTRIBUTING.md
```

- [ ] ไฟล์ `CONTRIBUTING.md` ปรากฏอยู่ใน `main` แล้ว
- [ ] Issue `#1` ที่สร้างไว้ถูกปิดสถานะเป็น **Closed** โดยอัตโนมัติ (เข้าไปดูที่หน้า Issue นั้นจะเห็นข้อความ "closed via merge request !X")
- [ ] Branch `feature/add-contributing-guide` ถูกลบไปแล้ว (ถ้าติ๊ก delete source branch ไว้)
- [ ] เข้าไปดูหน้า MR ที่เพิ่ง merge จะเห็นสถานะเปลี่ยนเป็น **"Merged"** สีม่วง พร้อมชื่อคนที่กด merge และเวลาที่ merge

### 480.11 Checklist สรุปก่อนไป Part 49

ก่อนไปต่อ Part 49 ให้ตรวจสอบว่าคุณ:

- [ ] สร้าง GitLab project ใหม่ได้ และเข้าใจความต่างระหว่าง project กับ repository
- [ ] เชื่อม local repository กับ GitLab remote ได้ทั้งแบบ HTTPS และรู้จักรูปแบบ SSH URL
- [ ] สร้าง Merge Request แรกได้ครบทุกขั้นตอนด้วยตัวเอง
- [ ] เขียน MR description ที่ดี และสร้าง MR template ใน `.gitlab/merge_request_templates/` ได้
- [ ] อ่าน diff ในแท็บ Changes ได้ทั้งมุมมอง Inline และ Side-by-side
- [ ] เข้าใจและใช้งาน Draft MR ได้ (รวมถึงรู้ประวัติจาก `WIP:` สู่ `Draft:`)
- [ ] เข้าใจ Approval Rules และ eligible approvers
- [ ] แยกแยะ Merge method ทั้ง 3 แบบของ GitLab ออกจากตัวเลือก Squash ได้อย่างถูกต้อง
- [ ] ใช้ closing pattern เชื่อม MR กับ Issue ให้ปิดอัตโนมัติได้สำเร็จ
- [ ] Merge MR แรกสำเร็จครบวงจรตั้งแต่สร้าง project จนถึง merge จริง

---

## สรุป Part 48

ใน Part นี้เราได้เรียนรู้ว่า:

1. **GitLab Project** กว้างกว่า **GitHub Repository** เพราะรวมทุกอย่างตั้งแต่ Issue, CI/CD, Registry ไว้ในหน่วยเดียว และมี Visibility level ถึง 3 ระดับ (Private, Internal, Public)
2. การเชื่อม local repository กับ GitLab ทำผ่าน `git remote add origin` เหมือน Git ทั่วไป แต่ต้องเลือกระหว่าง HTTPS URL (ใช้ PAT) กับ SSH URL (ใช้ SSH key) ให้เหมาะกับ workflow ของทีม
3. **Merge Request (MR)** คือชื่อที่ GitLab ใช้แทน Pull Request ของ GitHub แนวคิดเดียวกันแต่เน้นย้ำที่ "คำขอให้ merge" ตั้งแต่ source branch ไปยัง target branch
4. **MR Description** ที่ดีช่วยให้รีวิวเร็วขึ้น และ **MR Template** ที่เก็บไว้ใน `.gitlab/merge_request_templates/` ช่วยบังคับโครงสร้างมาตรฐานให้ทุก MR ในทีม
5. หน้า **Changes tab** คือที่อ่าน diff หลัก มีทั้งมุมมอง Inline และ Side-by-side พร้อมความสามารถคอมเมนต์เฉพาะบรรทัดและเริ่ม review แบบกลุ่มคอมเมนต์
6. **Draft MR** (เดิมใช้คำนำหน้า `WIP:` ปัจจุบันเปลี่ยนเป็น `Draft:`) ใช้บอกว่า MR ยังไม่พร้อม merge และล็อกปุ่ม Merge ไว้จนกว่าจะกด "Mark as ready"
7. **Approval Rules** บังคับจำนวนคนอนุมัติขั้นต่ำก่อน merge ได้ โดย eligible approver ต้องมี role Developer ขึ้นไป และผู้เขียน MR เองอนุมัติงานตัวเองไม่ได้ตามค่าเริ่มต้น
8. GitLab มี **Merge method 3 แบบ**: Merge commit, Merge commit with semi-linear history, และ Fast-forward merge — ส่วน **Squash เป็นตัวเลือกแยกต่างหาก** ที่ใช้ร่วมกับ method ใดก็ได้ ต่างจาก GitHub ที่รวมเป็น 3 ปุ่มให้เลือกตอน merge แต่ละครั้ง
9. การพิมพ์ **closing pattern** เช่น `Closes #123` ใน MR description จะสั่งให้ GitLab ปิด Issue นั้นอัตโนมัติทันทีที่ MR merge เข้า **default branch** เท่านั้น
10. เราได้ลงมือปฏิบัติจริงครบวงจร ตั้งแต่สร้าง project ใหม่ ไปจนถึงสร้างและ merge MR แรกให้สำเร็จ พร้อมเห็น Issue ถูกปิดอัตโนมัติจริง

**ต่อไป:** [Part 49: GitLab Issues, Boards, Milestones](./part-049-gitlab-issues-boards-milestones.md)
