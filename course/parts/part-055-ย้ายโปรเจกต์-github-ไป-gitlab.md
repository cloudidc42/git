# Part 55: โปรเจกต์ฝึกหัด: ย้ายโปรเจกต์จาก GitHub ไป GitLab

> **Step ในหลักสูตรนี้:** Step 541–550
> **เฟส:** 5 — GitLab (Part นี้คือ Part สุดท้ายของเฟส 5)
> **เป้าหมายของ Part นี้:** นำทุกทักษะที่เรียนมาตั้งแต่ Part 46 ถึง Part 54 มาใช้งานจริงในภารกิจเดียวที่ครบวงจรที่สุดของเฟส 5 — การย้ายโปรเจกต์ portfolio ที่สร้างไว้ตั้งแต่ Part 15 (โครงสร้างโปรเจกต์), Part 19 (README/Markdown) และ Part 25 (GitHub Pages) ออกจาก GitHub ไปอยู่บน GitLab แบบสมบูรณ์ ตั้งแต่การ import repository พร้อมประวัติและ Issues, การตั้งค่า Merge Request/Protected Branches ให้เทียบเท่าของเดิม, การแปลง GitHub Actions เป็น GitLab CI/CD, การย้าย GitHub Pages ไปเป็น GitLab Pages, การทดสอบว่าทุกอย่างทำงานเหมือนเดิม, การตัดสินใจเรื่อง Mirroring ระหว่างสองแพลตฟอร์ม ไปจนถึงการถอดบทเรียนและทบทวนภาพรวมทั้งเฟส 5 ก่อนก้าวเข้าสู่เฟส 6

---

## สารบัญของ Part นี้

- Step 541: ภาพรวมภารกิจปิดเฟส 5 — ย้ายโปรเจกต์ portfolio จริงจาก GitHub ไป GitLab
- Step 542: Import Repository จาก GitHub เข้า GitLab โดยตรงผ่านเครื่องมือในตัว
- Step 543: การย้าย Issues และข้อจำกัดของเครื่องมือ Import
- Step 544: ตั้งค่า Merge Request/Protected Branches ให้เทียบเท่ากับที่เคยตั้งบน GitHub
- Step 545: แปลง GitHub Actions Workflow เป็น `.gitlab-ci.yml` เทียบเคียงกัน
- Step 546: ตั้งค่า GitLab Pages แทน GitHub Pages ให้เว็บไซต์เดิม
- Step 547: ทดสอบว่าทุกอย่างทำงานเหมือนเดิมหลังย้าย (Checklist ตรวจสอบ)
- Step 548: การตัดสินใจว่าจะเก็บทั้งสองที่ (Repository Mirroring) หรือย้ายเด็ดขาด
- Step 549: Lessons Learned จากการย้ายแพลตฟอร์ม
- Step 550: สรุปทบทวนภาพรวมเฟส 5 ทั้งหมด (Part 46–55) พร้อม Cheat Sheet เทียบศัพท์ GitHub vs GitLab และ Checklist ก่อนเข้าเฟส 6

---

## Step 541: ภาพรวมภารกิจปิดเฟส 5 — ย้ายโปรเจกต์ portfolio จริงจาก GitHub ไป GitLab

ถึงจุดนี้คุณได้เรียนรู้ GitLab มาแล้วทั้งหมด 9 Part แบบแยกชิ้น:

| Part | หัวข้อที่เรียนมา |
|---|---|
| 46 | GitLab คืออะไร ต่างจาก GitHub อย่างไร |
| 47 | ติดตั้งและตั้งค่า GitLab (Cloud/Self-hosted เบื้องต้น) |
| 48 | GitLab Repository และ Merge Request เบื้องต้น |
| 49 | GitLab Issues, Boards, Milestones |
| 50 | GitLab CI/CD เบื้องต้น: `.gitlab-ci.yml` แรกของคุณ |
| 51 | GitLab Runner: การตั้งค่าและใช้งาน |
| 52 | GitLab Pages และ Container Registry |
| 53 | GitLab Groups, Permission และการจัดการทีม |
| 54 | GitLab Security Features: SAST, Dependency Scanning เบื้องต้น |

Part นี้จะ**ไม่สอนฟีเจอร์ใหม่ของ GitLab เลยแม้แต่อย่างเดียว** แต่จะพาคุณ **ร้อยเรียงทุกอย่างเข้าด้วยกันในภารกิจเดียวที่สมจริงที่สุด**: การย้ายโปรเจกต์จริงที่มีอยู่แล้ว (ไม่ใช่สร้างใหม่) จาก GitHub ไปอยู่บน GitLab แบบครบทุกมิติ — ไม่ใช่แค่โค้ด แต่รวมถึง Issues, กติกา Merge Request, Pipeline อัตโนมัติ และเว็บไซต์ที่ deploy อยู่

### 541.1 โปรเจกต์ที่เราจะย้าย

ตลอด Part นี้เราจะใช้โปรเจกต์เดียวกับที่สร้างไว้ในเฟส 2 และเฟส 3 เป็นกรณีศึกษา:

| แหล่งที่มา | สิ่งที่มีอยู่แล้วบน GitHub |
|---|---|
| Part 15 (โปรเจกต์ฝึกหัดจบเฟส 2) | Repository `portfolio-project` พร้อมประวัติ commit เต็ม, branch `main`, tag `v1.0.0` |
| Part 19 (README/Markdown) | `README.md` แบบมืออาชีพ พร้อม badge อ้างอิงถึง `somchai-jaidee/portfolio-project` และไฟล์ CI ที่ `.github/workflows/ci.yml` |
| Part 25 (GitHub Pages) | เว็บไซต์ static (`index.html`, `about.html`, `projects.html`, `assets/css/style.css`) ที่ deploy อยู่บน GitHub Pages ผ่าน branch `main` |
| เฟส 3 (Part 16–30) | Issues หลายรายการที่เปิดไว้ระหว่างฝึก, Pull Request ที่เคย merge ไปแล้ว, Label และ Milestone พื้นฐาน |

ถ้าคุณไม่มีโปรเจกต์นี้อยู่แล้ว (เช่น ข้ามแบบฝึกหัดใน Part ก่อนหน้าไป) ให้สร้าง repository เปล่าบน GitHub ชื่อ `portfolio-project` ใส่ไฟล์ `index.html` ง่าย ๆ, เปิด Issue ทดลองสัก 2–3 อัน, และตั้งค่า branch protection พื้นฐานไว้ก่อน เพื่อให้มีของจริงให้ย้ายตาม Step ต่าง ๆ ในนี้

### 541.2 ทำไมต้องมี Part แบบนี้ปิดท้ายเฟส 5

การย้ายแพลตฟอร์ม (platform migration) เป็นงานที่วิศวกรซอฟต์แวร์เจอจริงบ่อยกว่าที่คิด ไม่ว่าจะเป็นเหตุผลด้านนโยบายองค์กร (ต้องการ self-hosted เพื่อควบคุมข้อมูล), เหตุผลด้านต้นทุน, หรือการควบรวมบริษัท งานนี้ต่างจากการ "เรียนรู้ฟีเจอร์ใหม่" เพราะมันบังคับให้คุณ:

1. **เทียบเคียงแนวคิด** ระหว่างสองระบบที่หน้าตาต่างกันแต่แก้ปัญหาเดียวกัน (Pull Request vs Merge Request, Actions vs CI/CD)
2. **ระบุว่าอะไรย้ายอัตโนมัติได้ อะไรต้องทำมือ** — ไม่มีเครื่องมือ import ไหนสมบูรณ์แบบ 100%
3. **ตัดสินใจเชิงกลยุทธ์** ว่าจะเก็บของเดิมไว้เป็น backup/mirror หรือย้ายขาดไปเลย
4. **ตรวจสอบผลลัพธ์อย่างเป็นระบบ** ไม่ใช่แค่ "ดูเหมือนจะเหมือนเดิม" แต่ต้องมี checklist จริงจัง

ทักษะเหล่านี้ใช้ได้กับการย้ายแพลตฟอร์มไหนก็ตามในอนาคต ไม่ใช่แค่ GitHub↔GitLab เท่านั้น

### 541.3 ภาพรวมลำดับภารกิจทั้ง 10 Step

| Step | สิ่งที่จะเกิดขึ้น | ทักษะจาก Part ที่ถูกใช้ |
|---|---|---|
| 542 | Import repository พร้อมประวัติผ่านเครื่องมือในตัว GitLab | Part 46, 47, 48 |
| 543 | ตรวจสอบและทำความเข้าใจข้อจำกัดของการย้าย Issues | Part 49 |
| 544 | ตั้งค่า Protected Branches/Approval/CODEOWNERS เทียบเท่าของเดิม | Part 36, 37 (เฟส 4), Part 48, 53 |
| 545 | แปลง `.github/workflows/ci.yml` เป็น `.gitlab-ci.yml` | Part 50, 51 |
| 546 | ย้ายเว็บไซต์จาก GitHub Pages ไป GitLab Pages | Part 25 (เฟส 3), Part 52 |
| 547 | ตรวจสอบผลลัพธ์ทั้งหมดด้วย Checklist | Part 46–54 ทั้งหมด |
| 548 | ตัดสินใจเรื่อง Mirroring ระหว่างสองแพลตฟอร์ม | Part 47, 53 |
| 549 | ถอดบทเรียนจากการย้ายแพลตฟอร์มจริง | — |
| 550 | สรุปภาพรวมเฟส 5 ทั้งหมด | Part 46–55 ทั้งหมด |

### 541.4 เตรียมโฟลเดอร์และเครื่องมือสำหรับ Part นี้

```bash
cd ~/git-course
mkdir part-55-github-to-gitlab-migration
cd part-55-github-to-gitlab-migration
```

ตรวจสอบความพร้อมของเครื่องมือก่อนเริ่ม ตามสิ่งที่เรียนมาแล้วในเฟส 5:

```bash
git --version
git remote -v          # ยืนยันว่า repo เดิมยังชี้ไปที่ GitHub
```

และเตรียมสิ่งต่อไปนี้ให้พร้อมล่วงหน้า:

- บัญชี GitLab.com (จาก Part 47) ที่ล็อกอินได้แล้ว
- สิทธิ์ Owner หรือ Maintainer ของ repository `portfolio-project` บน GitHub เดิม
- **GitHub Personal Access Token (classic)** สโคป `repo` (และ `read:org` ถ้า repo อยู่ใต้ organization) — ใช้สำหรับให้ GitLab เชื่อมต่อไปดึงข้อมูลจาก GitHub ใน Step 542
- เวลาว่างประมาณ 1–2 ชั่วโมงสำหรับทำ Part นี้ให้ครบทุก Step แบบลงมือจริง

> **ข้อควรระวังเรื่องความปลอดภัย:** Personal Access Token คือกุญแจที่มีสิทธิ์เข้าถึง repository ของคุณ อย่าใส่ค่านี้ลงในโค้ดหรือ commit ใด ๆ เด็ดขาด ใช้แค่ตอนกรอกในหน้า import ของ GitLab แล้วปิดหน้าต่างทิ้งได้เลย และควรตั้งวันหมดอายุ (expiration) ของ token ให้สั้นที่สุดเท่าที่จำเป็น

---

## Step 542: Import Repository จาก GitHub เข้า GitLab โดยตรงผ่านเครื่องมือในตัว

GitLab มีเครื่องมือ import ในตัวที่เชื่อมต่อกับ GitHub ได้โดยตรง โดยไม่ต้อง clone/push ผ่านเครื่อง local เลยก็ได้ (แม้ Part นี้จะสอนวิธี manual คู่กันไว้ด้วยเพื่อความเข้าใจเชิงลึก)

### 542.1 ขั้นตอนการ Import ผ่านหน้าเว็บ GitLab

1. ล็อกอินเข้า GitLab.com (หรือ instance self-hosted ของคุณจาก Part 47)
2. คลิกปุ่ม **New project**
3. เลือกแท็บ **Import project**
4. เลือกตัวเลือก **GitHub**
5. กรอก **GitHub Personal Access Token** ที่เตรียมไว้ในหัวข้อ 541.4 แล้วกด **List repositories**
6. GitLab จะดึงรายชื่อ repository ทั้งหมดที่ token นั้นมีสิทธิ์เข้าถึงมาแสดง ค้นหา `portfolio-project` แล้วกดปุ่ม **Import** ที่แถวนั้น
7. เลือก **GitLab namespace** (บัญชีส่วนตัวหรือ Group ที่เรียนจาก Part 53) และตั้งชื่อโปรเจกต์ปลายทาง — ปกติใช้ชื่อเดิม `portfolio-project` เพื่อให้ URL/Path เทียบเคียงกันได้ง่าย
8. กด **Import** อีกครั้งเพื่อเริ่มกระบวนการจริง

```
[GitLab] Importing portfolio-project...
Status: started → finished
```

ระบบจะแสดงสถานะเป็น `started` ระหว่างดึงข้อมูล แล้วเปลี่ยนเป็น `finished` เมื่อเสร็จสมบูรณ์ สำหรับ repo ขนาดเล็กแบบ portfolio-project มักใช้เวลาไม่เกิน 1–2 นาที

### 542.2 สิ่งที่เครื่องมือ Import ดึงมาให้อัตโนมัติ

| ประเภทข้อมูล | ถูกย้ายอัตโนมัติหรือไม่ |
|---|---|
| Git commit history ทั้งหมด (ทุก branch) | ย้าย — และ **commit SHA เดิมทุกตัวยังคงเหมือนเดิมทุกประการ** |
| Tags (เช่น `v1.0.0`) | ย้าย |
| Issues พร้อมความคิดเห็น | ย้าย (มีข้อจำกัด ดู Step 543) |
| Pull Requests → กลายเป็น Merge Requests | ย้าย (มีข้อจำกัด ดู Step 543) |
| Labels และ Milestones | ย้าย |
| Release notes | ย้ายเป็น GitLab Release ที่ผูกกับ tag เดิม |
| Wiki | **ไม่ย้ายอัตโนมัติในหลายกรณี** ต้องย้ายมือ (ดูหัวข้อ 542.4) |
| GitHub Actions Workflows | **ไม่ย้าย** ต้องแปลงมือ (Step 545) |
| Branch protection rules | **ไม่ย้าย** ต้องตั้งค่าใหม่ (Step 544) |
| Webhooks / Integrations ภายนอก | **ไม่ย้าย** ต้องตั้งค่าใหม่ |
| GitHub Projects (Kanban board) | **ไม่ย้าย** ต้องสร้าง GitLab Issue Board ใหม่เอง (ทบทวนจาก Part 49) |

### 542.3 ตรวจสอบผลลัพธ์หลัง Import เสร็จ

Clone repository ที่เพิ่ง import มาจาก GitLab มาไว้ที่เครื่อง เพื่อยืนยันว่าประวัติมาครบจริง:

```bash
git clone git@gitlab.com:somchai-jaidee/portfolio-project.git portfolio-project-gitlab
cd portfolio-project-gitlab
git log --oneline | head -5
```

```
f9e8d7c (HEAD -> main, tag: v1.0.0, origin/main) Release v1.0.0: เว็บไซต์ Portfolio เวอร์ชันแรก
a1b2c3d Initial commit: โครงสร้างเริ่มต้นของโปรเจกต์ portfolio
```

เทียบกับ log บน GitHub เดิม (ทบทวนคำสั่งจาก Part 05):

```bash
cd ../portfolio-project
git log --oneline | head -5
```

ถ้า commit hash ตรงกันทุกตัว แปลว่าการย้ายประวัติสำเร็จสมบูรณ์ 100% — นี่คือข้อดีที่มาจากธรรมชาติของ Git เอง (ทบทวนจาก **Part 01, Step 6**): Git เก็บข้อมูลด้วย hash ที่คำนวณจากเนื้อหา ไม่ได้ผูกกับแพลตฟอร์มใดแพลตฟอร์มหนึ่ง ดังนั้นไม่ว่าจะย้ายไปเก็บที่ไหน **object เดิมก็ยังคง hash เดิมเป๊ะ ๆ เสมอ**

### 542.4 วิธี Manual (ทางเลือกสำรอง หรือเมื่อไม่ต้องการย้าย Issues)

ถ้าคุณต้องการย้ายแค่โค้ดโดยไม่ผ่านเครื่องมือ import ของ GitLab เลย (เช่น กรณีที่ policy องค์กรไม่อนุญาตให้เชื่อมต่อ token ข้ามระบบ) สามารถทำแบบ manual ได้ด้วย bare clone:

```bash
git clone --bare git@github.com:somchai-jaidee/portfolio-project.git
cd portfolio-project.git
git push --mirror git@gitlab.com:somchai-jaidee/portfolio-project.git
```

```
Enumerating objects: 142, done.
Counting objects: 100% (142/142), done.
To gitlab.com:somchai-jaidee/portfolio-project.git
 * [new branch]      main -> main
 * [new tag]         v1.0.0 -> v1.0.0
```

`git push --mirror` จะส่ง branch และ tag ทั้งหมดไปครบ **แต่ไม่พาข้อมูล Issues, Pull Request, Labels, Milestones ไปด้วยเลย** เพราะสิ่งเหล่านี้ไม่ได้เก็บอยู่ใน git object แต่เก็บอยู่ในฐานข้อมูลของแพลตฟอร์ม (ทบทวนแนวคิดนี้จาก **Part 46**: ความแตกต่างระหว่าง "ข้อมูล Git แท้ ๆ" กับ "ฟีเจอร์ที่แพลตฟอร์มเสริมเข้ามา") วิธีนี้จึงเหมาะกับกรณีที่ไม่สนใจประวัติ Issue เดิม หรือจะย้าย Issue ด้วยวิธีอื่นแยกต่างหาก

### 542.5 ตารางเปรียบเทียบสองวิธี

| | Import ผ่านเครื่องมือในตัว (Step 542.1) | Manual bare clone + push --mirror (Step 542.4) |
|---|---|---|
| ย้าย git history/branch/tag | ได้ | ได้ |
| ย้าย Issues/Merge Requests | ได้ (มีข้อจำกัด) | ไม่ได้ |
| ต้องใช้ GitHub token | ต้อง | ไม่ต้อง (ใช้แค่สิทธิ์ clone ปกติ) |
| ความเร็ว | ขึ้นกับขนาด Issue/PR ด้วย ไม่ใช่แค่ขนาดโค้ด | เร็ว เพราะส่งแค่ git object |
| เหมาะกับ | การย้ายเด็ดขาดที่ต้องการข้อมูลครบที่สุด | การทำ mirror/backup หรือย้ายเฉพาะโค้ด |

สำหรับ Part นี้ เราจะเดินหน้าต่อด้วยผลลัพธ์จาก **วิธี Import ผ่านเครื่องมือในตัว (542.1)** เพราะภารกิจของเราคือย้าย "ทุกอย่าง" ให้ได้มากที่สุดเท่าที่ทำได้อัตโนมัติ

---

## Step 543: การย้าย Issues และข้อจำกัดของเครื่องมือ Import

Step ก่อนหน้าย้ายโค้ดสำเร็จแล้ว แต่สิ่งที่ต้องตรวจสอบอย่างละเอียดกว่านั้นคือ Issues และ Pull Requests เพราะนี่คือส่วนที่มีข้อจำกัดมากที่สุดของกระบวนการ import

### 543.1 ตรวจสอบว่า Issues ถูกย้ายมาครบหรือไม่

เปิดแท็บ **Issues** ของ project ที่เพิ่ง import มาบน GitLab เทียบจำนวนกับ Issues (ทั้งเปิดและปิด) บน GitHub เดิม:

```bash
# นับ Issue ทั้งหมดบน GitHub เดิม (เปิด+ปิด) ผ่าน gh CLI ทบทวนจากเฟส 3
gh issue list --repo somchai-jaidee/portfolio-project --state all --limit 200 | wc -l
```

เทียบกับจำนวนที่ปรากฏใน GitLab Issues List (Part 49) — ถ้าตัวเลขไม่ตรงกัน ให้ตรวจสอบ log การ import (มักอยู่ที่หน้า **Settings > General > Advanced > Import history** หรือแจ้งเตือนตอน import) ว่ามี error หรือ rate limit จาก GitHub API เกิดขึ้นระหว่างทางหรือไม่ — repo ที่มี Issue จำนวนมาก (หลักร้อยขึ้นไป) มีโอกาสชน rate limit ของ GitHub API สูงกว่าที่คิด อาจต้อง import ซ้ำหรือรอแล้วลองใหม่

### 543.2 ข้อจำกัดสำคัญที่ต้องรู้ก่อนเชื่อว่าย้ายครบ 100%

| ข้อจำกัด | รายละเอียด | วิธีรับมือ |
|---|---|---|
| **การอ้างอิงผู้เขียน (Author mapping)** | ถ้าอีเมลของผู้เขียน Issue/PR เดิมบน GitHub ไม่ตรงกับบัญชี GitLab ที่มีอยู่ ระบบจะบันทึกเป็นข้อความอ้างอิงในเนื้อหา (เช่น "By original-author on GitHub") แทนที่จะ map เป็นเจ้าของบัญชีจริงบน GitLab | ยอมรับว่าประวัติผู้เขียนเดิมจะกลายเป็น "ข้อความอ้างอิง" ไม่ใช่ "ผู้ใช้จริง" เว้นแต่สมาชิกในทีมจะผูกอีเมลให้ตรงกันไว้ล่วงหน้า |
| **เลข Issue/MR ไม่ต่อเนื่องเป๊ะเสมอไป** | ระบบพยายามคงเลขเดิมให้ตรงกับ GitHub มากที่สุด แต่ Issue/PR ที่เคยถูกลบไปแล้วบน GitHub อาจทำให้ลำดับเลขมีช่องว่าง | ตรวจสอบด้วยตาว่าเลขสำคัญ ๆ ที่เคยอ้างอิงในเอกสารยังตรงกันอยู่หรือไม่ |
| **GitHub Projects (Kanban board) ไม่ถูกย้าย** | บอร์ดที่เคยจัดหมวดหมู่ Issue ไว้จะหายไปทั้งหมด | สร้าง GitLab Issue Board ใหม่เอง (ทบทวนวิธีทำจาก **Part 49**) แล้วจัด label ให้ตรงกับ column เดิม |
| **รูปภาพ/ไฟล์แนบที่ฝังใน Issue** | ไฟล์ที่อัปโหลดผ่าน GitHub's asset CDN (`user-images.githubusercontent.com`) ยังคงลิงก์ไปที่ GitHub เดิม ไม่ได้ถูกดาวน์โหลดมาเก็บใหม่บน GitLab | ตรวจสอบ Issue ที่มีรูปภาพสำคัญ ดาวน์โหลดแล้วอัปโหลดซ้ำเข้า GitLab ถ้าต้องการให้ Issue เป็นอิสระจาก GitHub อย่างแท้จริง |
| **Webhook/Integration ที่เคยผูกกับ Issue** | เช่น การเชื่อมกับ Slack, Discord ที่เคยแจ้งเตือนเมื่อมี Issue ใหม่บน GitHub | ต้องตั้งค่า Integration ใหม่ทั้งหมดบนฝั่ง GitLab (Settings > Integrations) |

### 543.3 การย้าย Wiki แบบ Manual (เมื่อเครื่องมือ import ไม่ครอบคลุม)

GitHub Wiki เป็น git repository แยกต่างหากเสมอ (`<repo>.wiki.git`) เช่นเดียวกับ GitLab Wiki (`<project>.wiki.git`) ทำให้ย้ายด้วยหลักการเดียวกับ Step 542.4 ได้ทันที โดยไม่ต้องพึ่งเครื่องมือ import เลย:

```bash
git clone --bare git@github.com:somchai-jaidee/portfolio-project.wiki.git
cd portfolio-project.wiki.git
git push --mirror git@gitlab.com:somchai-jaidee/portfolio-project.wiki.git
```

วิธีนี้ทำงานได้แน่นอนเสมอเพราะ Wiki ก็คือ Git repository ธรรมดา — ตอกย้ำหลักการเดิมจาก Part 01 อีกครั้งว่า **สิ่งที่เป็น "ข้อมูล Git แท้ ๆ" ย้ายได้ 100% เสมอไม่ว่าแพลตฟอร์มไหน**

### 543.4 Checklist ก่อนไป Step ถัดไป

- [ ] จำนวน Issues บน GitLab ตรงกับจำนวนบน GitHub เดิม (หรือรู้สาเหตุของส่วนต่างที่เกิดขึ้นแล้ว)
- [ ] Pull Requests ที่ merge แล้วปรากฏเป็น Merge Requests สถานะ Merged บน GitLab
- [ ] ตรวจสอบ Issue สำคัญ 2–3 อันแบบละเอียดว่าความคิดเห็น (comment) ทั้งหมดตามมาครบ
- [ ] รู้แล้วว่า GitHub Projects board ต้องสร้างใหม่มือใน GitLab Issue Board
- [ ] ย้าย Wiki (ถ้ามี) ด้วยวิธี manual bare clone + push --mirror เรียบร้อยแล้ว

---

## Step 544: ตั้งค่า Merge Request/Protected Branches ให้เทียบเท่ากับที่เคยตั้งบน GitHub

โค้ดและ Issue ย้ายมาแล้ว แต่ **กติกาการทำงาน** (branch protection, การรีวิว, CODEOWNERS) เป็นการตั้งค่าระดับแพลตฟอร์มที่ไม่ถูกย้ายมาโดยอัตโนมัติเลย ต้องตั้งใหม่ทั้งหมดตาม Step นี้

### 544.1 ตารางเทียบ Branch Protection: GitHub vs GitLab

| การตั้งค่าบน GitHub (ทบทวนจาก Part 37 เฟส 4) | เทียบเท่าบน GitLab | ตำแหน่งที่ตั้งค่าบน GitLab |
|---|---|---|
| Require a pull request before merging | Protected branch: "Allowed to push and merge" ตั้งเป็น "No one" (ห้าม push ตรง ต้องผ่าน MR เท่านั้น) | Settings > Repository > Protected branches |
| Require approvals (จำนวน N คน) | Merge request approval rules: "Approvals required" | Settings > Merge requests > Approval rules |
| Require review from Code Owners | เมื่อมีไฟล์ CODEOWNERS ระบบสร้าง approval rule "Code Owners" ให้อัตโนมัติ + เปิด "Require approval from code owners" | Settings > Merge requests |
| Dismiss stale approvals เมื่อมี commit ใหม่ | "Remove all approvals when commits are added to the source branch" | Settings > Merge requests > Approval settings |
| Require status checks to pass before merging | Merge checks: "Pipelines must succeed" | Settings > Merge requests > Merge checks |
| Require conversation resolution before merging | Merge checks: "All threads must be resolved" | Settings > Merge requests > Merge checks |
| Require linear history | Merge method: "Fast-forward merge" | Settings > Merge requests > Merge method |
| Restrict who can push to matching branches | Protected branch: "Allowed to push and merge" กำหนดเป็น Role/User/Group เฉพาะ | Settings > Repository > Protected branches |
| Allow force pushes | Protected branch: "Allowed to force push" (toggle เปิด/ปิด) | Settings > Repository > Protected branches |
| Require signed commits | Push Rules: "Reject unsigned commits" (ฟีเจอร์ระดับ Premium/Ultimate บน GitLab.com) | Settings > Repository > Push Rules |

### 544.2 ลงมือตั้งค่า Protected Branch สำหรับ `main`

1. เปิด **Settings > Repository > Protected branches**
2. เลือก branch `main`
3. **Allowed to push and merge:** เลือก `No one` (บังคับให้ผ่าน Merge Request เท่านั้น เทียบเท่า "Require a pull request before merging")
4. **Allowed to merge:** เลือก `Maintainers` (เทียบเท่าการจำกัดสิทธิ์ merge บน GitHub)
5. **Allowed to force push:** ปิดไว้ (ป้องกันการ force push ทับประวัติของ branch หลัก เหมือนที่เคยตั้งไว้บน GitHub)
6. กด **Protect**

### 544.3 ตั้งค่า Approval Rules ให้เทียบเท่า "Require approvals: 1"

1. เปิด **Settings > Merge requests**
2. ในส่วน **Approval rules** กด **Add approval rule**
3. ตั้งชื่อ rule เช่น `Standard review`
4. **Approvals required:** ใส่ `1` (หรือจำนวนที่เคยตั้งไว้บน GitHub)
5. เลือกกลุ่มผู้มีสิทธิ์ approve (Eligible approvers) — โดยทั่วไปคือสมาชิกที่มี Role `Developer` ขึ้นไป (ทบทวนระดับ role จาก **Part 53**)
6. บันทึกการตั้งค่า

### 544.4 แปลง CODEOWNERS จาก GitHub Syntax เป็น GitLab Syntax

ไฟล์ CODEOWNERS ของ GitHub มักอยู่ที่ root, `docs/`, หรือ `.github/` ตัวอย่างของโปรเจกต์เดิม:

```
# .github/CODEOWNERS (ของเดิมบน GitHub)
*.html   @somchai-jaidee
/assets/ @somchai-jaidee @frontend-team
```

GitLab รองรับ syntax นี้เกือบทั้งหมด แต่มีความสามารถเพิ่มเติมที่ GitHub ไม่มี คือ **การแบ่ง Section** พร้อมกำหนดจำนวน approval ที่ต้องการต่อ section ได้โดยตรงในไฟล์เดียว:

```
# CODEOWNERS (ของใหม่บน GitLab — วางที่ root, docs/ หรือ .gitlab/)

[Frontend][2]
*.html   @somchai-jaidee
/assets/ @somchai-jaidee @frontend-team

^[Documentation]
*.md     @somchai-jaidee
```

อธิบายส่วนที่ต่างจาก GitHub:

| Syntax | ความหมาย |
|---|---|
| `[ชื่อ Section]` | จัดกลุ่มกฎ CODEOWNERS เพื่อแสดงแยกหมวดในหน้า Merge Request |
| `[ชื่อ Section][N]` | กำหนดว่า section นี้ **ต้องมีอย่างน้อย N คน** จากรายชื่อ approve ก่อน merge ได้ (GitHub ไม่มีความสามารถนี้ในไฟล์ CODEOWNERS โดยตรง ต้องอาศัย branch protection แทน) |
| `^[ชื่อ Section]` | Section แบบ **Optional** — แสดงเป็นคำแนะนำแต่ไม่บังคับว่าต้อง approve ก่อน merge |

> **ข้อควรระวัง:** GitHub's CODEOWNERS ไม่มีแนวคิดเรื่อง Section หรือจำนวน approval ต่อบรรทัด ถ้าโปรเจกต์เดิมของคุณพึ่งพา branch protection rule "Require review from Code Owners" อย่างเดียว (ไม่ได้ระบุจำนวนคนต่อ pattern) เวลาย้ายมา GitLab ให้ใช้ syntax แบบไม่มี section (`*.html @somchai-jaidee`) ธรรมดาไปก่อน แล้วค่อยพิจารณาใช้ฟีเจอร์ Section เพิ่มความเข้มงวดทีหลังถ้าต้องการ

### 544.5 Checklist ก่อนไป Step ถัดไป

- [ ] Protected branch สำหรับ `main` ตั้งค่าเรียบร้อย ห้าม push ตรงเข้า
- [ ] Approval rule ตั้งจำนวนผู้ approve เทียบเท่ากับที่เคยตั้งบน GitHub
- [ ] ไฟล์ CODEOWNERS ถูกแปลง syntax และวางในตำแหน่งที่ GitLab อ่านได้ (root, `docs/`, หรือ `.gitlab/`)
- [ ] Merge checks (Pipelines must succeed, All threads must be resolved) เปิดใช้งานตรงกับพฤติกรรมเดิม
- [ ] ทดลองเปิด Merge Request ทดสอบสักอันเพื่อยืนยันว่ากฎทั้งหมดทำงานถูกต้องจริง

---

## Step 545: แปลง GitHub Actions Workflow เป็น `.gitlab-ci.yml` เทียบเคียงกัน

โปรเจกต์เดิมมีไฟล์ `.github/workflows/ci.yml` ที่รัน lint และ deploy อัตโนมัติ (อ้างอิงจาก badge ใน README ที่เขียนไว้ตั้งแต่ Part 19) Step นี้จะแปลงไฟล์นี้ให้ทำงานเทียบเท่าบน GitLab CI/CD

### 545.1 ไฟล์ต้นฉบับ: `.github/workflows/ci.yml`

```yaml
name: CI

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
      - name: Install htmlhint
        run: npm install -g htmlhint
      - name: Lint HTML files
        run: htmlhint "**/*.html"

  deploy:
    needs: lint
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build site folder
        run: |
          mkdir -p _site
          cp *.html _site/
          cp -r assets _site/
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: _site
      - name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v4
```

### 545.2 ตารางเทียบ Syntax แบบละเอียด

| GitHub Actions | GitLab CI/CD | หมายเหตุ |
|---|---|---|
| ตำแหน่งไฟล์ `.github/workflows/*.yml` | ตำแหน่งไฟล์ `.gitlab-ci.yml` (ที่ root ของ repo) | GitLab ใช้ไฟล์เดียว ไม่แยกหลายไฟล์ตาม workflow (แต่ `include:` เพื่อแยกไฟล์ย่อยได้) |
| `on: push / pull_request` | `rules: - if:` หรือ `only:`/`except:` (แบบเก่า) | `rules` คือแนวทางมาตรฐานในปัจจุบัน |
| `on.push.branches: [main]` | `rules: - if: '$CI_COMMIT_BRANCH == "main"'` | ตรวจสอบชื่อ branch ปัจจุบันผ่านตัวแปรที่ระบบมีให้ |
| `jobs.<job_id>:` | ชื่อ job เป็น key ระดับบนสุดของไฟล์โดยตรง | โครงสร้างคล้ายกันมาก |
| `runs-on: ubuntu-latest` | `image: node:20-alpine` (หรือ `tags:` ถ้าใช้ self-hosted Runner) | GitLab เลือก environment ด้วย Docker image ไม่ใช่ชื่อ OS สำเร็จรูป |
| `steps:` | `script:` | คำสั่งที่รันตามลำดับใน job |
| `uses: actions/checkout@v4` | **ไม่ต้องระบุเลย** | GitLab clone repository ให้อัตโนมัติทุก job ตั้งแต่ต้น |
| `env:` | `variables:` | ตัวแปรสภาพแวดล้อมของ job/pipeline |
| `secrets.MY_SECRET` | `$MY_SECRET` (ตั้งค่าที่ Settings > CI/CD > Variables) | ทบทวนจาก **Part 50, 54** |
| `needs:` (จัดลำดับ job ข้าม stage) | `needs:` (ชื่อเหมือนกัน ใช้แนวคิดคล้ายกัน) | ทั้งคู่ใช้ข้าม stage/ job dependency ได้ |
| `strategy.matrix` | `parallel: matrix:` | รันหลายชุด parameter พร้อมกัน |
| `actions/upload-artifact` / `download-artifact` | `artifacts: paths:` | GitLab แนบ artifact อัตโนมัติทุก job ที่ระบุ ไม่ต้องเรียก action แยก |
| `actions/cache` | `cache: key: / paths:` | Cache dependency ระหว่าง pipeline run |
| `workflow_dispatch` | `when: manual` | ทำให้ job ต้องกดรันเองผ่านหน้าเว็บ |
| `concurrency:` | `resource_group:` | ป้องกัน job ชนิดเดียวกันรันซ้อนกัน |
| `environment:` | `environment:` | แนวคิดใกล้เคียงกันมาก ผูกกับ Environment/Deployment tracking |
| `secrets.GITHUB_TOKEN` (auto) | `$CI_JOB_TOKEN` (auto) | Token ชั่วคราวสำหรับเรียก API ภายในแพลตฟอร์มเดียวกัน |

### 545.3 ไฟล์ผลลัพธ์: `.gitlab-ci.yml`

```yaml
stages:
  - lint
  - deploy

lint:
  stage: lint
  image: node:20-alpine
  script:
    - npm install -g htmlhint
    - htmlhint "**/*.html"

pages:
  stage: deploy
  image: alpine:latest
  needs:
    - lint
  script:
    - mkdir -p public
    - cp *.html public/
    - cp -r assets public/
  artifacts:
    paths:
      - public
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

สังเกตจุดสำคัญ 3 อย่างที่ต่างจากต้นฉบับ:

1. **ไม่มีขั้นตอน `actions/checkout` เลย** เพราะ GitLab clone repo ให้อัตโนมัติทุก job อยู่แล้ว (ทบทวนจาก **Part 50**)
2. **Job ที่ทำหน้าที่ deploy ต้องตั้งชื่อว่า `pages` เป๊ะ ๆ** และ artifact ต้องอยู่ในโฟลเดอร์ชื่อ `public` — นี่คือกฎเฉพาะของ GitLab Pages ที่จะอธิบายละเอียดใน Step 546
3. **ไม่ต้องเรียก action สำเร็จรูปสำหรับ deploy** (`upload-pages-artifact`, `deploy-pages`) เพราะ GitLab ผูก job ชื่อ `pages` เข้ากับระบบ Pages โดยตรงในตัว

### 545.4 ทดสอบ Pipeline หลังแปลงไฟล์

Commit และ push ไฟล์ `.gitlab-ci.yml` เข้า repository บน GitLab:

```bash
git add .gitlab-ci.yml
git commit -m "ci: migrate from GitHub Actions to GitLab CI/CD"
git push origin main
```

เปิดแท็บ **CI/CD > Pipelines** บนหน้าเว็บ GitLab เพื่อดูสถานะ (ทบทวนหน้าตาและวิธีอ่านผลจาก **Part 50, 51**):

```
Pipeline #142  passed
  ✓ lint    (12s)
  ✓ pages   (8s)
```

ถ้า job ไหนล้มเหลว ให้กดเข้าไปดู log แบบเดียวกับที่เคยดู log ของ GitHub Actions — โครงสร้างการอ่าน log ของทั้งสองระบบคล้ายกันมาก เพราะทั้งคู่แสดงผลลัพธ์ของคำสั่งที่รันตามลำดับ

---

## Step 546: ตั้งค่า GitLab Pages แทน GitHub Pages ให้เว็บไซต์เดิม

Step 545 ได้เตรียม job `pages` ไว้แล้ว Step นี้จะยืนยันการตั้งค่าให้เว็บไซต์ใช้งานได้จริงบน URL ของ GitLab

### 546.1 กฎพื้นฐานของ GitLab Pages ที่ต้องรู้

| กฎ | รายละเอียด |
|---|---|
| ชื่อ job ต้องเป็น `pages` | ระบบ GitLab Pages จะดึงเนื้อหาจาก job ที่ชื่อนี้เท่านั้น |
| Artifact ต้องอยู่ในโฟลเดอร์ `public` | ไฟล์ทุกไฟล์ที่อยู่ใต้ `public/` หลัง job รันเสร็จ จะถูกนำไป deploy เป็นเว็บไซต์ |
| ต้องมี `index.html` ที่ root ของ `public/` เป็นอย่างน้อย | เทียบเท่ากับกฎของ GitHub Pages ที่ต้องมี `index.html` ที่ root ของ source ที่เลือกไว้ |

เทียบกับ GitHub Pages ที่เลือก **Source: Deploy from a branch** แล้วชี้ไปที่ branch/โฟลเดอร์ตรง ๆ (ทบทวนจาก **Part 25, Step 250**) — GitLab Pages ใช้แนวคิด **"deploy จากผลลัพธ์ของ Pipeline"** เป็นหลัก ทำให้ยืดหยุ่นกว่าตรงที่สามารถ build/แปลงไฟล์ก่อน deploy ได้ในตัว โดยไม่ต้องพึ่ง branch แยกต่างหากแบบ `gh-pages`

### 546.2 ตรวจสอบ URL ที่ได้หลัง Pipeline สำเร็จ

เปิด **Settings > Pages** บนเว็บ GitLab หลัง pipeline รัน job `pages` สำเร็จ จะเห็น URL ปรากฏขึ้นในรูปแบบ:

```
Your Pages site is running at https://somchai-jaidee.gitlab.io/portfolio-project
```

ตารางเทียบรูปแบบ URL ระหว่างสองแพลตฟอร์ม:

| สถานการณ์ | GitHub Pages | GitLab Pages |
|---|---|---|
| Project site (repo ธรรมดา) | `https://<username>.github.io/<repo-name>/` | `https://<username>.gitlab.io/<project-name>` |
| User/Root site (repo ชื่อพิเศษ) | repo ชื่อ `<username>.github.io` → `https://<username>.github.io/` | project ชื่อ `<namespace>.gitlab.io` → `https://<namespace>.gitlab.io/` |
| Group/Organization site | ไม่มีแนวคิดนี้ตรง ๆ (ใช้ user/org site แทน) | project ชื่อ `<group>.gitlab.io` ภายใต้ Group | 

### 546.3 ตั้งค่า Custom Domain (ถ้ามีโดเมนของตัวเอง)

เทียบขั้นตอนกับที่เคยทำบน GitHub Pages (Part 25):

| ขั้นตอน | GitHub Pages | GitLab Pages |
|---|---|---|
| ที่ตั้งค่า | Settings > Pages > Custom domain (สร้างไฟล์ `CNAME` ในโค้ดอัตโนมัติ) | Settings > Pages > **New Domain** |
| การตรวจสอบ DNS | เพิ่ม record `CNAME`/`A` ชี้มาที่ GitHub Pages | เพิ่ม record `CNAME`/`A` ชี้มาที่ GitLab Pages ตามค่าที่หน้าเว็บระบุ |
| HTTPS/SSL | ออกใบรับรองอัตโนมัติผ่าน Let's Encrypt | ออกใบรับรองอัตโนมัติผ่าน Let's Encrypt เช่นกัน (เปิด "Automatic certificate management" ในหน้า Domain) |

### 546.4 ทดสอบวงจรการอัปเดตเว็บไซต์เหมือนที่เคยทำใน Part 25

แก้ไขเนื้อหาสักจุดหนึ่งในไฟล์ `about.html` แล้ว push:

```bash
git add about.html
git commit -m "docs: update about section after migrating to GitLab"
git push origin main
```

ไปที่แท็บ **CI/CD > Pipelines** เพื่อดู job `pages` รันใหม่โดยอัตโนมัติ จนสถานะเปลี่ยนเป็นสีเขียว (สำเร็จ) จากนั้นรีเฟรชหน้าเว็บไซต์จริงเพื่อยืนยันว่าการเปลี่ยนแปลงแสดงผลแล้ว — พฤติกรรมนี้เทียบเท่ากับที่เคยทดสอบใน **Part 25, Step 250, ขั้นตอนที่ 6** ทุกประการ เพียงแค่เปลี่ยนจากแท็บ Actions มาเป็นแท็บ Pipelines เท่านั้น

---

## Step 547: ทดสอบว่าทุกอย่างทำงานเหมือนเดิมหลังย้าย (Checklist ตรวจสอบ)

การย้ายแพลตฟอร์มที่ดีต้องจบด้วยการตรวจสอบอย่างเป็นระบบ ไม่ใช่แค่ความรู้สึกว่า "น่าจะโอเคแล้ว" ให้ไล่ตรวจตาม checklist ด้านล่างนี้ทีละข้อจริง ๆ

### 547.1 Checklist ด้าน Repository และประวัติ

- [ ] `git log --oneline` ของ GitLab แสดง commit hash ตรงกับ GitHub เดิมทุกตัว
- [ ] จำนวน branch บน GitLab ตรงกับ GitHub เดิม (`git branch -a` เทียบกัน)
- [ ] Tag `v1.0.0` (และ tag อื่น ๆ ถ้ามี) ปรากฏครบและชี้ไปยัง commit ที่ถูกต้อง
- [ ] ขนาดไฟล์ repository (`.git` folder) ใกล้เคียงกัน ไม่มีข้อมูลหายหรือถูกตัดทอน

### 547.2 Checklist ด้าน Issues และ Merge Requests

- [ ] จำนวน Issues (เปิด+ปิด) ตรงกันหรือรู้สาเหตุของส่วนต่างแล้ว (ทบทวนจาก Step 543)
- [ ] Pull Requests เดิมที่ merge แล้วแสดงเป็น Merge Requests สถานะ Merged
- [ ] Label และ Milestone ที่เคยใช้ยังอยู่ครบและผูกกับ Issue ถูกต้อง
- [ ] ทดลองเปิด Issue ใหม่และปิดผ่าน keyword `Closes #xxx` ใน commit message เพื่อยืนยันว่ากลไกยังทำงาน (ทบทวนจาก Part 49)

### 547.3 Checklist ด้าน Merge Request Workflow

- [ ] Push ตรงเข้า `main` ถูกปฏิเสธ (ต้องผ่าน Merge Request เท่านั้น)
- [ ] เปิด Merge Request ทดสอบแล้วระบบขอ approval ตามจำนวนที่ตั้งไว้จริง
- [ ] ไฟล์ที่ตรงกับ CODEOWNERS ทำให้ระบบ request review จากเจ้าของไฟล์อัตโนมัติ
- [ ] Merge ถูกบล็อกถ้า Pipeline ยังไม่ผ่าน (ทดสอบโดยตั้งใจทำให้ lint fail ดูสักครั้ง)

### 547.4 Checklist ด้าน CI/CD

- [ ] Pipeline รันอัตโนมัติทุกครั้งที่ push เข้า `main`
- [ ] Job `lint` ตรวจจับ error ได้เหมือนที่ GitHub Actions เคยทำ (ทดสอบโดยใส่ typo ใน HTML ดูสักครั้ง)
- [ ] Job `pages` deploy สำเร็จและใช้เวลาใกล้เคียงกับ workflow เดิมบน GitHub Actions
- [ ] Environment variable/secret ที่เคยตั้งใน GitHub Secrets ถูกสร้างใหม่ใน GitLab CI/CD Variables ครบถ้วน (**ข้อควรจำ:** ค่า secret ไม่เคยถูกย้ายอัตโนมัติไม่ว่ากรณีใด ต้องตั้งค่าใหม่ด้วยมือเสมอเพื่อความปลอดภัย)

### 547.5 Checklist ด้านเว็บไซต์ที่ Deploy

- [ ] เว็บไซต์เข้าถึงได้จริงผ่าน URL ของ GitLab Pages
- [ ] เนื้อหาทุกหน้า (`index.html`, `about.html`, `projects.html`) แสดงผลถูกต้องเหมือนเดิม
- [ ] ไฟล์ CSS/รูปภาพโหลดถูกต้อง ไม่มีลิงก์เสียเพราะ path เปลี่ยน
- [ ] HTTPS ทำงานปกติไม่มี security warning
- [ ] (ถ้ามี custom domain) โดเมนเดิมชี้มาที่ GitLab Pages ถูกต้องแล้ว

### 547.6 Checklist ด้านเอกสารและลิงก์อ้างอิง

- [ ] อัปเดต badge ใน `README.md` จาก URL GitHub Actions เป็น URL GitLab Pipeline (ทบทวนรูปแบบ badge จาก Part 19 แล้วเปลี่ยนเป็น syntax ของ GitLab CI/CD badge)
- [ ] ลิงก์ในเอกสารที่เคยชี้ไปยัง `github.com/somchai-jaidee/portfolio-project` ถูกอัปเดตเป็น `gitlab.com/somchai-jaidee/portfolio-project` ในจุดที่เกี่ยวข้อง
- [ ] ไฟล์ `CONTRIBUTING.md` (ถ้ามี) อัปเดตขั้นตอน fork/clone ให้ตรงกับ GitLab (เช่น "Merge Request" แทน "Pull Request")

ถ้าติ๊กครบทุกข้อ (หรือรู้เหตุผลของข้อที่ยังไม่ผ่านอย่างชัดเจน) แปลว่าการย้ายแพลตฟอร์มของคุณสมบูรณ์ในระดับที่ใช้งานจริงได้แล้ว

---

## Step 548: การตัดสินใจว่าจะเก็บทั้งสองที่ (Repository Mirroring) หรือย้ายเด็ดขาด

เมื่อย้ายสำเร็จแล้ว คำถามถัดไปคือ **จะปิด GitHub repository เดิมทิ้งไปเลย หรือจะเก็บทั้งสองที่ไว้คู่กัน** GitLab มีฟีเจอร์ **Repository Mirroring** ที่ช่วยให้ตัดสินใจเรื่องนี้ได้ยืดหยุ่นขึ้นมาก

### 548.1 สองรูปแบบของ Mirroring

| รูปแบบ | ทิศทางข้อมูล | ใช้เมื่อไหร่ |
|---|---|---|
| **Push Mirror** | GitLab (ต้นทางหลัก) → push การเปลี่ยนแปลงออกไปยัง remote ปลายทาง (เช่น GitHub) โดยอัตโนมัติทุกครั้งที่มีการ push เข้า GitLab | ใช้เมื่อ **ตัดสินใจแล้วว่า GitLab คือที่หลักตัวจริง** แต่ยังอยากให้โค้ดปรากฏบน GitHub ด้วย (เช่น เพื่อคง visibility, star, หรือให้ contributor เก่าที่คุ้นเคยกับ GitHub ยังตามงานได้) |
| **Pull Mirror** | GitLab ดึงการเปลี่ยนแปลงเข้ามาจาก remote ต้นทาง (เช่น GitHub) เป็นระยะ | ใช้เมื่อ **ยังไม่กล้าตัดขาด GitHub** อยากลองใช้ GitLab แบบ read-only ควบคู่ไปก่อน โดยที่ GitHub ยังเป็นที่ทำงานจริง |

### 548.2 ตั้งค่า Mirroring ผ่านหน้าเว็บ

1. เปิด **Settings > Repository > Mirroring repositories**
2. กรอก **Git repository URL** ปลายทาง (หรือต้นทางในกรณี Pull) เช่น `https://github.com/somchai-jaidee/portfolio-project.git`
3. เลือก **Mirror direction:** `Push` หรือ `Pull` ตามที่ตัดสินใจไว้
4. เลือก **Authentication method** — ปกติใช้ Password/Token หรือ SSH key ที่มีสิทธิ์เขียน/อ่านฝั่งปลายทาง
5. (ทางเลือก) เปิด **Only mirror protected branches** ถ้าต้องการ mirror เฉพาะ branch สำคัญ ไม่ต้อง mirror branch ทดลองทั้งหมด
6. กด **Mirror repository**

> **ข้อควรทราบเรื่องระดับแพ็กเกจ (Tier):** ความสามารถบางอย่างของ Repository Mirroring (เช่น ความถี่ในการ pull mirror อัตโนมัติ หรือการรองรับ private repository บางกรณี) อาจแตกต่างกันไปตามแพ็กเกจของ GitLab (Free/Premium/Ultimate) และเปลี่ยนแปลงได้ตามเวลา ก่อนวางแผนใช้งานจริงจัง ควรตรวจสอบหน้าเปรียบเทียบแพ็กเกจล่าสุดของ GitLab อีกครั้งเสมอ

### 548.3 ตารางช่วยตัดสินใจ: Mirror ทั้งสองที่ vs ย้ายเด็ดขาด

| ปัจจัย | ควรเลือก Mirror (เก็บทั้งสองที่) | ควรย้ายเด็ดขาด (ปิด GitHub เดิม) |
|---|---|---|
| ทีมงาน/ผู้ใช้ | มีคนคุ้นเคยกับ GitHub เดิมจำนวนมาก ยังไม่พร้อมเปลี่ยนพร้อมกันทั้งหมด | ทีมทั้งหมดพร้อมย้ายพร้อมกัน ไม่มีใครยังต้องพึ่ง GitHub |
| ความซับซ้อนของกติกา (Issues, MR rules) | ต้องการเวลาปรับตัว ค่อย ๆ ทดสอบกติกาใหม่ก่อนตัดขาดของเดิม | ตรวจสอบผ่าน checklist ใน Step 547 แล้วมั่นใจว่าเทียบเท่า 100% |
| ต้นทุนการดูแลสองระบบ | ยอมรับต้นทุนที่ต้องดูแล/sync สองที่ได้ในระยะสั้น | ต้องการลดความซับซ้อน ดูแลที่เดียวจบ |
| Open Source visibility | ต้องการคง star/fork/ผู้ติดตามเดิมบน GitHub ไว้ (open source มักใช้ push mirror เพื่อจุดประสงค์นี้) | โปรเจกต์ภายในองค์กร ไม่ต้องพึ่ง visibility ของ GitHub |
| ความเสี่ยงด้าน Data Sovereignty | ยอมรับความเสี่ยงที่ข้อมูลยังกระจายอยู่สองที่ | ต้องการควบคุมข้อมูลทั้งหมดไว้ที่เดียวตามนโยบายองค์กร (เหตุผลทั่วไปที่องค์กรเลือกย้ายมา GitLab self-hosted ตั้งแต่ Part 46) |

### 548.4 สำหรับโปรเจกต์ portfolio ของ Part นี้ ควรเลือกอะไร

สำหรับกรณีศึกษาของเรา (โปรเจกต์ฝึกฝนส่วนตัว ไม่ใช่โปรเจกต์องค์กรที่มีกติกา data sovereignty) คำแนะนำคือ:

- ถ้าต้องการโชว์ผลงานทั้งสองแพลตฟอร์มในเรซูเม่ (เพราะผู้ว่าจ้างบางคนคุ้นกับ GitHub มากกว่า) → ตั้งเป็น **Push Mirror** จาก GitLab ไปยัง GitHub เพื่อให้ GitLab เป็นที่ทำงานจริง ส่วน GitHub กลายเป็นสำเนาที่อัปเดตอัตโนมัติเสมอ
- ถ้าต้องการเรียนรู้ GitLab แบบเต็มตัวและไม่อยากดูแลสองที่ → **ย้ายเด็ดขาด** ปิดหรือ archive repository เดิมบน GitHub พร้อมใส่ข้อความใน README เดิมชี้ทางไปยัง repository ใหม่บน GitLab

---

## Step 549: Lessons Learned จากการย้ายแพลตฟอร์มจริง

การย้ายแพลตฟอร์มจริงมักให้บทเรียนที่ไม่ได้อยู่ในเอกสารของแพลตฟอร์มไหนเลย มาสรุปสิ่งที่ควรจดจำจากภารกิจนี้

### 549.1 สิ่งที่ต้องระวัง (Pitfalls)

| ปัญหาที่พบบ่อย | สาเหตุ | บทเรียน |
|---|---|---|
| Token หมดสิทธิ์หรือหมดอายุกลางทาง Import | ตั้งวันหมดอายุ token สั้นเกินไป หรือ scope ไม่ครบ (`repo`, `read:org`) | ตรวจสอบ scope และวันหมดอายุของ token ก่อนเริ่ม import ทุกครั้ง |
| Rate limit จาก GitHub API ระหว่าง import Issue จำนวนมาก | GitHub จำกัดจำนวน API call ต่อชั่วโมง | สำหรับ repo ที่มี Issue หลักร้อยขึ้นไป ควรวางแผนเวลาสำรองไว้ และตรวจสอบ log การ import ว่ามี error หรือไม่ |
| ลืมย้าย Secret/Environment Variable | Secret ไม่เคยถูกย้ายอัตโนมัติโดยเจตนา (เพื่อความปลอดภัย) | ทำ checklist แยกเฉพาะ secret ทุกตัวที่เคยตั้งใน GitHub Secrets แล้วสร้างใหม่ใน GitLab CI/CD Variables ทีละตัว |
| Branch protection rule ไม่ถูกย้าย ทำให้ push ตรงเข้า main ได้ชั่วขณะ | เป็นการตั้งค่าระดับแพลตฟอร์ม ไม่ใช่ข้อมูล Git | ตั้งค่า protected branch **ทันทีหลัง import เสร็จ** ก่อนที่จะมีใคร push งานเข้ามาโดยไม่ตั้งใจ |
| ลิงก์ในเอกสารเก่ายังชี้ไปที่ `github.com` | มองข้ามการอัปเดตเอกสาร มัวแต่โฟกัสที่โค้ดและ pipeline | ทำ checklist ด้านเอกสาร (Step 547.6) แยกต่างหากเสมอ อย่ารวมเข้ากับ checklist ด้านเทคนิค |
| สับสนระหว่าง syntax ของ GitHub Actions กับ GitLab CI/CD ระหว่างเปลี่ยนผ่าน | สอง syntax คล้ายกันมากพอที่จะทำให้ประมาทได้ (ทั้งคู่เป็น YAML, ทั้งคู่มี `jobs`) | ใช้ตารางเทียบ syntax (Step 545.2) อ้างอิงทุกครั้งที่ไม่แน่ใจ แทนที่จะเดาจากความจำ |
| คิดว่า Mirroring คือ "ย้ายเสร็จสมบูรณ์แล้ว" | Mirror ย้ายแค่โค้ด ไม่ย้าย Issue/การตั้งค่าใด ๆ ต่อเนื่อง | แยกความเข้าใจให้ชัดระหว่าง "sync โค้ด" (mirroring) กับ "ย้ายแพลตฟอร์มแบบสมบูรณ์" (ทุก Step ใน Part นี้) |

### 549.2 สิ่งที่ทำได้ดี (สิ่งที่ควรทำต่อไปในการย้ายแพลตฟอร์มครั้งถัดไป)

1. **เริ่มจากการทำความเข้าใจว่าอะไรคือ "ข้อมูล Git แท้ ๆ" กับ "ฟีเจอร์เสริมของแพลตฟอร์ม"** ก่อนเริ่มย้ายจริง — หลักการนี้ (จาก Part 01 และ Part 46) ช่วยให้คาดเดาได้ล่วงหน้าว่าอะไรย้ายอัตโนมัติได้แน่นอน (commit, branch, tag) และอะไรต้องเตรียมใจว่าต้องทำมือ (Issue metadata, การตั้งค่า, Secret)
2. **ใช้ checklist แยกตามหมวดหมู่ชัดเจน** (Repository, Issues, MR Workflow, CI/CD, เว็บไซต์, เอกสาร) แทนการเช็คแบบรวม ๆ ทำให้ไม่พลาดจุดใดจุดหนึ่งไป
3. **ทดสอบ workflow จริงก่อนประกาศว่าย้ายเสร็จ** เช่น ลองเปิด Merge Request จริง ลองทำให้ Pipeline fail โดยตั้งใจ แทนที่จะเชื่อแค่ว่าการตั้งค่าหน้าตา "ดูถูกต้อง"
4. **ตัดสินใจเรื่อง Mirror vs ย้ายเด็ดขาดอย่างมีเหตุผล** แทนที่จะปล่อยให้ทั้งสองที่ค้างอยู่แบบไม่มีแผนชัดเจน (ซึ่งมักนำไปสู่ความสับสนว่า "ที่ไหนคือของจริงกันแน่" ในภายหลัง)
5. **บันทึกบทเรียนไว้เป็นเอกสารทันทีหลังทำเสร็จ** (แบบที่ Step นี้กำลังทำอยู่) เพราะรายละเอียดเล็ก ๆ น้อย ๆ ระหว่างทางมักลืมเร็วมากถ้าไม่จดไว้

---

## Step 550: สรุปทบทวนภาพรวมเฟส 5 ทั้งหมด (Part 46–55) พร้อม Cheat Sheet และ Checklist ก่อนเข้าเฟส 6

ยินดีด้วย — คุณเพิ่งผ่านภารกิจปิดเฟส 5 ที่สมบูรณ์แบบที่สุด ตั้งแต่การรู้จัก GitLab ครั้งแรก ไปจนถึงการย้ายโปรเจกต์จริงข้ามแพลตฟอร์มสำเร็จ มาถึงจุดนี้ ให้ใช้เวลาทบทวนภาพรวมทั้งหมดของเฟส 5 (Part 46–55, Step 451–550) อย่างละเอียดก่อนก้าวเข้าสู่เฟส 6

### 550.1 Cheat Sheet เทียบศัพท์ GitHub vs GitLab ฉบับสมบูรณ์

| แนวคิด | GitHub | GitLab |
|---|---|---|
| คำขอรวมโค้ด | Pull Request (PR) | Merge Request (MR) |
| ระบบ CI/CD | GitHub Actions (`.github/workflows/*.yml`) | GitLab CI/CD (`.gitlab-ci.yml`) |
| เครื่องรัน Pipeline | Runner (GitHub-hosted / Self-hosted runner) | Runner (Shared runner / Self-hosted Runner — ทบทวนจาก Part 51) |
| การจัดหมวดผู้รับผิดชอบโค้ด | CODEOWNERS (`.github/CODEOWNERS`) | CODEOWNERS (root, `docs/`, หรือ `.gitlab/`) รองรับ Section + จำนวน approval ต่อ section |
| กฎป้องกัน branch หลัก | Branch protection rules | Protected branches + Merge request approval rules |
| เว็บไซต์ static ฟรี | GitHub Pages | GitLab Pages |
| ที่เก็บ Container Image | GitHub Container Registry (ghcr.io) | GitLab Container Registry (ในตัวทุกโปรเจกต์) |
| การจัดกลุ่มหลาย repository | Organization | Group (รองรับ Subgroup ซ้อนกันได้ลึกกว่า) |
| บอร์ดจัดการงาน | GitHub Projects (Kanban) | GitLab Issue Boards |
| การเปิดพื้นที่พูดคุยชุมชน | GitHub Discussions | GitLab ไม่มีฟีเจอร์นี้ตรง ๆ (มักใช้ Issue แบบ Q&A หรือเครื่องมือภายนอกแทน) |
| การสแกนช่องโหว่ในโค้ด | GitHub Advanced Security (CodeQL, Dependabot) | GitLab SAST, Dependency Scanning (built-in templates — Part 54) |
| Personal Access Token | Personal Access Token (classic/fine-grained) | Personal Access Token |
| การเชื่อมต่อผ่าน SSH | SSH Key ผูกกับบัญชี | SSH Key ผูกกับบัญชี (กลไกเหมือนกันทุกประการ เพราะเป็นมาตรฐาน Git/SSH) |
| ระดับสิทธิ์การเข้าถึง | Read / Triage / Write / Maintain / Admin | Guest / Reporter / Developer / Maintainer / Owner |
| ไฟล์ config wiki | Wiki เป็น git repo แยก (`<repo>.wiki.git`) | Wiki เป็น git repo แยก (`<project>.wiki.git`) — กลไกเดียวกันเป๊ะ |
| การนำเข้าโปรเจกต์จากแพลตฟอร์มอื่น | Import repository (จาก SVN, Mercurial ฯลฯ) | Import project (จาก GitHub, Bitbucket, GitLab อื่น ฯลฯ) |
| การ sync กับ remote ภายนอกอัตโนมัติ | ไม่มีฟีเจอร์ในตัวโดยตรง (ต้องใช้ Action เขียนเอง) | Repository Mirroring (Push/Pull) ในตัว |
| Token อัตโนมัติสำหรับ pipeline | `secrets.GITHUB_TOKEN` | `$CI_JOB_TOKEN` |

### 550.2 ทบทวนภาพรวมทุก Part ในเฟส 5 (Part 46–55)

| Part | Step | สิ่งที่เรียนรู้ | นำมาใช้จริงใน Part 55 ตรงไหน |
|---|---|---|---|
| 46 | 451–460 | GitLab คืออะไร ต่างจาก GitHub อย่างไร | หลักการแยก "ข้อมูล Git แท้ ๆ" กับ "ฟีเจอร์แพลตฟอร์ม" ที่ใช้ตัดสินใจตลอด Part นี้ |
| 47 | 461–470 | ติดตั้งและตั้งค่า GitLab (Cloud/Self-hosted) | บัญชี GitLab.com ที่ใช้ import project ใน Step 542 |
| 48 | 471–480 | GitLab Repository และ Merge Request เบื้องต้น | โครงสร้าง Merge Request ที่ตั้งกฎใน Step 544 |
| 49 | 481–490 | GitLab Issues, Boards, Milestones | ตรวจสอบการย้าย Issues ใน Step 543 และสร้าง Issue Board ใหม่ทดแทน GitHub Projects |
| 50 | 491–500 | GitLab CI/CD เบื้องต้น: `.gitlab-ci.yml` แรก | พื้นฐานที่ใช้แปลง workflow ทั้งหมดใน Step 545 |
| 51 | 501–510 | GitLab Runner: การตั้งค่าและใช้งาน | เข้าใจว่า pipeline ใน Step 545–546 รันบน Runner ประเภทไหน |
| 52 | 511–520 | GitLab Pages และ Container Registry | การตั้งค่าเว็บไซต์ทั้งหมดใน Step 546 |
| 53 | 521–530 | GitLab Groups, Permission และการจัดการทีม | Role ที่ใช้กำหนด Approval rule ใน Step 544 และ Namespace ที่เลือกตอน import |
| 54 | 531–540 | GitLab Security Features: SAST, Dependency Scanning | แนวคิดเรื่อง Secret/Variable ที่ต้องตั้งใหม่หลังย้ายใน Step 547 |
| 55 | 541–550 | โปรเจกต์ฝึกหัดรวมทุกอย่างเข้าด้วยกัน | Part นี้เอง |

### 550.3 แผนภาพรวมกระบวนการย้ายแพลตฟอร์มแบบเต็มวงจร

```
เตรียม GitHub Token + บัญชี GitLab พร้อม
        │
        ▼
  Import Repository ผ่านเครื่องมือในตัว (โค้ด + Issue + MR)
        │
        ▼
  ตรวจสอบความครบถ้วนของ Issues/MR + ย้าย Wiki แบบ manual
        │
        ▼
  ตั้งค่า Protected Branch + Approval Rule + CODEOWNERS ใหม่
        │
        ▼
  แปลง .github/workflows/*.yml → .gitlab-ci.yml
        │
        ▼
  ตั้งค่า GitLab Pages (job ชื่อ pages + artifact โฟลเดอร์ public)
        │
        ▼
  ไล่ตรวจ Checklist ทุกหมวด (Repo/Issue/MR/CI/เว็บไซต์/เอกสาร)
        │
        ▼
  ตัดสินใจ: Mirror ทั้งสองที่ ─────┐
        │                         │ (ถ้าเลือก mirror)
        ▼                         │
  ย้ายเด็ดขาด (archive ของเดิม)   ตั้งค่า Push/Pull Mirror
        │                         │
        └─────────────┬───────────┘
                       ▼
              บันทึก Lessons Learned
```

### 550.4 Checklist ทบทวนภาพรวมเฟส 5 ทั้งหมด (Part 46–55) ก่อนเข้าสู่เฟส 6

ก่อนไปต่อ Part 56 (เริ่มต้นเฟส 6: Git ขั้นสูง/Internals) ให้ตรวจสอบตัวเองอย่างละเอียดตามรายการนี้ ถ้าข้อไหนยังไม่มั่นใจ แนะนำให้ย้อนกลับไปอ่าน Part ที่เกี่ยวข้องอีกครั้งก่อน:

**พื้นฐาน GitLab และการตั้งค่า**
- [ ] อธิบายความแตกต่างระหว่าง GitHub กับ GitLab ได้ทั้งในแง่ปรัชญาผลิตภัณฑ์และฟีเจอร์ (ทบทวนจาก Part 46)
- [ ] ตั้งค่าบัญชี GitLab.com หรือ instance self-hosted ได้เอง (ทบทวนจาก Part 47)
- [ ] สร้าง Repository ใหม่และเปิด Merge Request แรกได้โดยไม่ต้องเปิดเอกสารดู (ทบทวนจาก Part 48)

**การจัดการงานและทีม**
- [ ] ใช้งาน GitLab Issues, Boards และ Milestones ได้คล่องเทียบเท่าที่เคยใช้ GitHub (ทบทวนจาก Part 49)
- [ ] เข้าใจระดับสิทธิ์ Guest/Reporter/Developer/Maintainer/Owner และเลือกใช้ได้ถูกสถานการณ์ (ทบทวนจาก Part 53)
- [ ] จัดการ Group และ Subgroup สำหรับหลายโปรเจกต์ได้ (ทบทวนจาก Part 53)

**CI/CD และ Infrastructure**
- [ ] เขียน `.gitlab-ci.yml` เบื้องต้นได้เองตั้งแต่ต้น ไม่ต้องดูตัวอย่าง (ทบทวนจาก Part 50)
- [ ] เข้าใจความแตกต่างระหว่าง Shared Runner กับ Self-hosted Runner และตั้งค่า Runner ของตัวเองได้ (ทบทวนจาก Part 51)
- [ ] Deploy เว็บไซต์ผ่าน GitLab Pages และใช้งาน Container Registry เบื้องต้นได้ (ทบทวนจาก Part 52)
- [ ] เข้าใจภาพรวมของ SAST และ Dependency Scanning ว่าเปิดใช้งานอย่างไรและอ่านผลลัพธ์อย่างไร (ทบทวนจาก Part 54)

**การย้ายแพลตฟอร์ม (Part 55)**
- [ ] Import repository จาก GitHub มา GitLab พร้อมประวัติ Git ครบ 100% ได้ด้วยตัวเอง
- [ ] อธิบายได้ว่าอะไรย้ายอัตโนมัติได้ อะไรต้องทำมือ (Issues, Wiki, CI/CD, Branch protection, Secrets)
- [ ] แปลง GitHub Actions workflow เป็น `.gitlab-ci.yml` ได้เองโดยใช้ตารางเทียบ syntax เป็นแนวทาง
- [ ] ตั้งค่า Protected Branch, Approval Rule และ CODEOWNERS บน GitLab ให้เทียบเท่ากติกาที่เคยมีบน GitHub
- [ ] ตั้งค่า GitLab Pages และเข้าใจกฎเรื่องชื่อ job `pages` กับโฟลเดอร์ `public`
- [ ] ตัดสินใจเรื่อง Repository Mirroring ได้อย่างมีเหตุผล ไม่ใช่แค่ทำตามความเคยชิน
- [ ] ไล่ตรวจ Checklist ทุกหมวดหลังย้ายแพลตฟอร์มจริงได้ครบถ้วนโดยไม่ข้ามขั้นตอน

**ภาพรวมทั้งเฟส**
- [ ] อธิบายวงจรการย้ายแพลตฟอร์ม Git แบบเต็มรูปแบบให้คนอื่นฟังได้โดยไม่ต้องเปิดเอกสารนี้ดู
- [ ] เข้าใจว่าทักษะการย้ายแพลตฟอร์มนี้ใช้ได้กับการย้ายข้ามระบบ Git อื่น ๆ ในอนาคตด้วย ไม่ใช่แค่ GitHub↔GitLab

ถ้าคุณติ๊กครบทุกข้อ (หรือเกือบครบ) แปลว่าคุณพร้อมสำหรับเฟส 6 อย่างแท้จริงแล้ว — เฟส 5 คือจุดเปลี่ยนที่พาคุณออกจากการผูกติดกับแพลตฟอร์มเดียว ไปสู่ความเข้าใจว่า Git เองต่างหากคือรากฐานที่แท้จริง ส่วนแพลตฟอร์มอย่าง GitHub และ GitLab เป็นเพียง "ชั้นบริการ" ที่สร้างขึ้นมาห่อหุ้ม Git อีกที ทักษะทั้งหมดที่ฝึกมาในเฟสนี้จะเป็นรากฐานสำคัญให้กับเฟส 6 ซึ่งจะพาคุณลงลึกไปถึงกลไกภายในของ Git เองแบบที่ไม่มีแพลตฟอร์มไหนซ่อนไว้ได้อีกต่อไป

---

## สรุป Part 55

ใน Part นี้เราได้นำทักษะทั้งหมดจากเฟส 5 มาใช้งานจริงในภารกิจเดียวที่ครบวงจรที่สุด:

1. วางแผนภารกิจปิดเฟส 5 โดยเลือกโปรเจกต์ portfolio จริงจาก Part 15/19/25 มาเป็นกรณีศึกษา (Step 541)
2. Import repository จาก GitHub เข้า GitLab ผ่านเครื่องมือในตัว พร้อมยืนยันว่าประวัติ Git ทุก commit hash คงเดิม 100% (Step 542)
3. ตรวจสอบและเข้าใจข้อจำกัดของการย้าย Issues, Pull Request และ Wiki อย่างละเอียด (Step 543)
4. ตั้งค่า Protected Branches, Approval Rules และแปลง CODEOWNERS ให้เทียบเท่ากติกาที่เคยมีบน GitHub (Step 544)
5. แปลง `.github/workflows/ci.yml` เป็น `.gitlab-ci.yml` พร้อมตารางเทียบ syntax แบบละเอียด (Step 545)
6. ตั้งค่า GitLab Pages แทนที่ GitHub Pages ให้เว็บไซต์เดิมใช้งานได้จริงบน URL ใหม่ (Step 546)
7. ไล่ตรวจ Checklist ทุกหมวดเพื่อยืนยันว่าทุกอย่างทำงานเหมือนเดิมหลังย้าย (Step 547)
8. ตัดสินใจเรื่อง Repository Mirroring ระหว่างเก็บทั้งสองที่กับย้ายเด็ดขาดอย่างมีเหตุผล (Step 548)
9. ถอดบทเรียนจากการย้ายแพลตฟอร์มจริงทั้งสิ่งที่ต้องระวังและสิ่งที่ทำได้ดี (Step 549)
10. สรุปภาพรวมทั้งเฟส 5 ผ่าน Cheat Sheet เทียบศัพท์ GitHub vs GitLab ฉบับสมบูรณ์ และ Checklist ครบทุกด้าน (Step 550)

Part นี้คือจุดปิดฉากของ **เฟส 5: GitLab (Part 46–55, Step 451–550)** อย่างสมบูรณ์ คุณได้พิสูจน์ให้ตัวเองเห็นแล้วว่าความรู้เรื่อง Git ที่แท้จริงนั้นไม่ได้ผูกติดกับแพลตฟอร์มใดแพลตฟอร์มหนึ่ง — ไม่ว่าจะเป็น GitHub, GitLab หรือระบบอื่นใดในอนาคต หลักการเดียวกันของ Git (snapshot, branch, commit hash) ยังคงทำงานเหมือนเดิมเสมอ สิ่งที่เปลี่ยนไปมีเพียง "ชั้นบริการ" ที่ห่อหุ้มอยู่ด้านบนเท่านั้น

จากนี้ไป หลักสูตรจะพาคุณเข้าสู่ **เฟส 6: Git ขั้นสูง/Internals (Part 56–65, Step 551–650)** ซึ่งจะเจาะลึกกลไกภายในที่แท้จริงของ Git เอง ตั้งแต่ Object Model (blob, tree, commit, ref), Plumbing Commands, Packfiles, Git Hooks, Git Attributes, Git LFS ไปจนถึงการจัดการ Performance ของ Repository ขนาดใหญ่และ Git Worktree — ความรู้ระดับนี้จะทำให้คุณเข้าใจว่าเบื้องหลังทุกคำสั่งที่ใช้มาตลอดหลักสูตรนี้ Git ทำงานอย่างไรจริง ๆ

**ต่อไป:** [Part 56: Git Internals: Object Model (blob, tree, commit, ref)](./part-056-git-internals-object-model.md)
