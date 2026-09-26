# Part 30: โปรเจกต์ฝึกหัด: Contribute ให้โปรเจกต์ Open Source จริง

> **Step ในหลักสูตรนี้:** Step 291–300
> **เฟส:** 3 — GitHub เบื้องต้น (Part นี้คือ Part สุดท้ายของเฟส 3)
> **เป้าหมายของ Part นี้:** นำทุกทักษะที่เรียนมาตั้งแต่ Part 16 ถึง Part 29 มาใช้งานจริงในภารกิจเดียวที่ครบวงจรที่สุดของเฟส 3 — ตั้งแต่การสมัครบัญชี GitHub, ตั้งค่า SSH, เขียน README, เปิด Issue, ทำ Pull Request, รับ Code Review, ใช้ GitHub Projects, Fork repository, ใช้ GitHub Pages, จัดการ Labels/Milestones/Templates, เขียน Wiki, พูดคุยใน Discussions ไปจนถึงการค้นหาด้วย GitHub Search โดยนำทั้งหมดมาประยุกต์ใช้กับการ **contribute ให้โปรเจกต์ Open Source จริงบนโลกใบนี้** ตั้งแต่การเลือกโปรเจกต์ อ่านกติกา Fork/Clone/ตั้งค่า upstream ลงมือแก้โค้ด เขียน PR ที่มีคุณภาพ รับมือกับ Code Review จริงจาก maintainer ไปจนถึงการทบทวนภาพรวมทั้งเฟส 3 ก่อนก้าวเข้าสู่เฟส 4

---

## สารบัญของ Part นี้

- Step 291: ภาพรวมภารกิจปิดเฟส 3 — เลือกเส้นทาง Contribute จริงหรือจำลองสถานการณ์เสมือนจริง
- Step 292: การเลือกโปรเจกต์ที่เหมาะสมกับระดับตัวเอง (ใช้เทคนิค Search จาก Part 29)
- Step 293: อ่าน CONTRIBUTING.md และ CODE_OF_CONDUCT.md ของโปรเจกต์เป้าหมายก่อนเริ่ม
- Step 294: Fork, Clone, ตั้งค่า Upstream Remote ตามที่เรียนใน Part 24
- Step 295: สร้าง Branch และลงมือแก้ไข/เพิ่มโค้ดจริงตามที่ Issue กำหนด
- Step 296: เขียน Commit Message และ PR Description ตามมาตรฐานของโปรเจกต์นั้น ๆ
- Step 297: ส่ง PR ข้าม Fork และรอ/ตอบสนอง Code Review จริงจาก Maintainer
- Step 298: การจัดการ Feedback และแก้ไข PR ตามคำแนะนำ
- Step 299: สิ่งที่ได้เรียนรู้จากการ Contribute จริง (Reflection Questions)
- Step 300: สรุปทบทวนภาพรวมเฟส 3 ทั้งหมด (Part 16–30) พร้อม Cheat Sheet และ Checklist ก่อนเข้าเฟส 4

---

## Step 291: ภาพรวมภารกิจปิดเฟส 3 — เลือกเส้นทาง Contribute จริงหรือจำลองสถานการณ์เสมือนจริง

ถึงจุดนี้คุณได้เรียนรู้ทักษะ GitHub มาแล้วทั้งหมด 14 Part แบบแยกชิ้น:

| Part | หัวข้อที่เรียนมา |
|---|---|
| 16 | สมัครและตั้งค่าบัญชี GitHub อย่างมืออาชีพ |
| 17 | สร้าง Repository บน GitHub และเชื่อมกับเครื่อง local |
| 18 | SSH Key และการตั้งค่าความปลอดภัยในการเชื่อมต่อ |
| 19 | README.md ที่ดีและ Markdown สำหรับ GitHub |
| 20 | GitHub Issues: การจัดการงานและบั๊ก |
| 21 | Pull Request เบื้องต้น: สร้าง PR แรกของคุณ |
| 22 | Code Review บน GitHub: ให้และรับ Feedback |
| 23 | GitHub Projects: บอร์ดจัดการงานแบบ Kanban |
| 24 | Fork และการ Contribute แบบ Open Source เบื้องต้น |
| 25 | GitHub Pages: การทำเว็บไซต์ฟรีจาก Repository |
| 26 | Labels, Milestones, Issue/PR Templates |
| 27 | GitHub Wiki และเอกสารประกอบโปรเจกต์ |
| 28 | GitHub Discussions และการสร้าง Community |
| 29 | GitHub Search และการค้นหาโค้ด/โปรเจกต์อย่างมีประสิทธิภาพ |

Part นี้จะ**ไม่สอนฟีเจอร์ใหม่ของ GitHub เลยแม้แต่อย่างเดียว** แต่จะพาคุณ **ร้อยเรียงทุกอย่างเข้าด้วยกันในภารกิจเดียวที่สมจริงที่สุด**: การส่ง Pull Request แรกของคุณไปยังโปรเจกต์ Open Source ที่มีคนอื่นใช้งานจริง

### 291.1 สองเส้นทางที่คุณเลือกได้

การ contribute ให้ open source จริงต้องมี "โจทย์" ที่เปิดรับอยู่จริง ณ เวลาที่คุณเรียน ซึ่งอาจไม่สามารถควบคุมได้ 100% หลักสูตรนี้จึงออกแบบให้เดินได้สองเส้นทาง โดยทั้งสองเส้นทางใช้ **ขั้นตอนและคำสั่ง Git/GitHub ชุดเดียวกันทุกประการ** ต่างกันแค่ปลายทางของ Pull Request

| เส้นทาง | เหมาะกับใคร | ปลายทางของ PR |
|---|---|---|
| **เส้นทาง A — Contribute จริง** | มีเวลาเพียงพอ พร้อมรอ Review จาก maintainer จริง (อาจใช้เวลาหลายวัน) | โปรเจกต์ Open Source จริงบน GitHub |
| **เส้นทาง B — จำลองสถานการณ์เสมือนจริง** | ต้องการฝึกขั้นตอนให้ครบทันที ไม่ต้องรอคนอื่น | Repository ฝึกฝนเฉพาะสำหรับมือใหม่ เช่น `firstcontributions/first-contributions` ซึ่งออกแบบมาให้ merge ได้เกือบทุก PR โดยอัตโนมัติ หรือ repo ส่วนตัวของคุณเองที่จำลองบทบาท "maintainer" กับ "contributor" คนละ branch |

> **คำแนะนำของหลักสูตร:** ให้เริ่มด้วย**เส้นทาง B** ก่อนเสมอ เพื่อฝึกกลไกทั้งหมดโดยไม่มีความกดดันเรื่องเวลา จากนั้นเมื่อคล่องแล้วค่อยลองเส้นทาง A กับโปรเจกต์จริงที่คุณสนใจ ทั้งสองเส้นทางนับเป็นการทำภารกิจของ Part นี้สำเร็จเท่ากัน เพราะเป้าหมายคือการฝึก **กระบวนการ** ไม่ใช่การได้ PR merge จริงเพียงอย่างเดียว

### 291.2 ภาพรวมลำดับภารกิจทั้ง 10 Step

| Step | สิ่งที่จะเกิดขึ้น | ทักษะจาก Part ที่ถูกใช้ |
|---|---|---|
| 292 | ค้นหาและเลือกโปรเจกต์/Issue ที่เหมาะกับระดับ | Part 29 (Search) |
| 293 | อ่านกติกาการ contribute ของโปรเจกต์ | Part 19 (Markdown/README) |
| 294 | Fork, Clone, ตั้งค่า upstream | Part 17, 18, 24 |
| 295 | สร้าง branch และแก้โค้ดจริง | Part 07–08 (เฟส 2), Part 20 (Issues) |
| 296 | เขียน commit message และ PR description | Part 19, 21 |
| 297 | ส่ง PR ข้าม fork และรอ Review | Part 21, 22, 24 |
| 298 | แก้ไข PR ตาม feedback | Part 22 |
| 299 | ทบทวนบทเรียนที่ได้ | — |
| 300 | สรุปภาพรวมเฟส 3 ทั้งหมด | Part 16–29 ทั้งหมด |

### 291.3 เตรียมโฟลเดอร์สำหรับ Part นี้

```bash
cd ~/git-course
mkdir part-30-open-source-contribution
cd part-30-open-source-contribution
```

ตลอด Part นี้เราจะอ้างอิงโฟลเดอร์นี้เป็นฐาน แต่ Fork ของคุณจะถูก clone เป็นโฟลเดอร์ย่อยแยกไปตามชื่อ repository ที่เลือก

### 291.4 ตรวจสอบความพร้อมของเครื่องมือก่อนเริ่ม

ก่อนเริ่ม ให้ตรวจสอบว่าสภาพแวดล้อมของคุณพร้อมสำหรับภารกิจนี้ตามสิ่งที่เรียนมาแล้วในเฟสก่อนหน้า:

```bash
git --version
gh --version
ssh -T git@github.com
```

ผลลัพธ์ที่ควรเห็น:

```
git version 2.43.0
gh version 2.45.0 (2024-xx-xx)
Hi <your-username>! You've successfully authenticated, but GitHub does not provide shell access.
```

- ถ้า `ssh -T git@github.com` ไม่ผ่าน ให้ย้อนกลับไปทบทวน **Part 18: SSH Key และการตั้งค่าความปลอดภัยในการเชื่อมต่อ** ก่อน
- ถ้ายังไม่ได้ติดตั้ง GitHub CLI (`gh`) แนะนำให้ติดตั้งไว้ เพราะจะช่วยให้ขั้นตอน fork/PR ใน Step ถัดไปสะดวกขึ้นมาก (ใช้ผ่านเว็บ GitHub ล้วน ๆ ก็ทำได้เช่นกัน หลักสูตรนี้จะสอนทั้งสองวิธีคู่กันไปตลอด Part)

---

## Step 292: การเลือกโปรเจกต์ที่เหมาะสมกับระดับตัวเอง (ใช้เทคนิค Search จาก Part 29)

การเลือกโปรเจกต์ผิดคือสาเหตุอันดับหนึ่งที่ทำให้มือใหม่ท้อและเลิก contribute กลางคัน ดังนั้น Step นี้จึงสำคัญมาก

### 292.1 เกณฑ์การเลือกโปรเจกต์ที่ดีสำหรับผู้เริ่มต้น

| เกณฑ์ | เหตุผล | วิธีตรวจสอบ |
|---|---|---|
| มี Label `good first issue` หรือ `help wanted` | บ่งบอกว่า maintainer เตรียมงานที่เหมาะกับมือใหม่ไว้แล้ว | ดูที่แท็บ Issues |
| Repository ยังมีการ commit สม่ำเสมอ (ไม่ถูกทิ้งร้าง) | โปรเจกต์ที่ตายแล้วจะไม่มีใคร review PR ของคุณเลย | ดูวันที่ commit ล่าสุด |
| มีไฟล์ `CONTRIBUTING.md` | แปลว่าโปรเจกต์มีกติกาชัดเจน ลด friction ตอน PR | ดูที่หน้าแรกของ repo |
| Maintainer ตอบ Issue/PR ในเวลาที่สมเหตุสมผล (ไม่กี่วันถึงไม่กี่สัปดาห์) | ถ้าเงียบเกิน 6 เดือน โอกาสได้ Review ต่ำมาก | ดู PR ที่ merge ล่าสุดว่าห่างจากวันเปิดกี่วัน |
| ภาษาโปรแกรมและขนาดโค้ดที่คุณอ่านเข้าใจ | ลด cognitive load ในการทำความเข้าใจ codebase | ดูภาษาหลักที่แสดงบนหน้า repo |
| จำนวน Star/Fork อยู่ในระดับกลาง ไม่เล็กเกินไปหรือใหญ่เกินไป | โปรเจกต์ดังมากมักมี PR ค้างจำนวนมาก แข่งขันสูง โปรเจกต์เล็กเกินไปอาจไม่มีใคร maintain | ดูตัวเลขบนหน้า repo |

### 292.2 ใช้เทคนิค GitHub Search ขั้นสูงจาก Part 29 เพื่อค้นหา Issue ที่เหมาะกับตัวเอง

ทบทวนจาก Part 29: GitHub รองรับ **search qualifiers** ที่ทรงพลังมาก เปิดหน้า https://github.com/search แล้วลองพิมพ์คำค้นแบบนี้:

```
is:issue is:open label:"good first issue" language:javascript no:assignee
```

อธิบายทีละส่วน:

| Qualifier | ความหมาย |
|---|---|
| `is:issue` | ค้นหาเฉพาะ Issue ไม่เอา Pull Request ปนมา |
| `is:open` | เอาเฉพาะ Issue ที่ยังเปิดอยู่ ยังไม่ถูกปิด |
| `label:"good first issue"` | กรองเฉพาะ Issue ที่ maintainer ติด label นี้ไว้ |
| `language:javascript` | จำกัดเฉพาะ repository ที่ใช้ภาษานี้เป็นหลัก (เปลี่ยนเป็นภาษาที่คุณถนัดได้) |
| `no:assignee` | กรองเฉพาะ Issue ที่ยังไม่มีใครรับไปทำ |

ตัวอย่าง query อื่น ๆ ที่มีประโยชน์:

```
is:issue is:open label:"help wanted" comments:<3
```

`comments:<3` กรองเฉพาะ Issue ที่มีคนคอมเมนต์น้อยกว่า 3 ครั้ง (แปลว่ายังไม่มีคนแย่งทำเยอะ)

```
is:issue is:open label:"good first issue" archived:false stars:>500
```

`stars:>500` ช่วยกรองเอาเฉพาะโปรเจกต์ที่มีคนใช้งานพอสมควร (ลด risk ว่าโปรเจกต์จะถูกทิ้งร้าง) ส่วน `archived:false` ป้องกันไม่ให้หลุดไปเจอ repo ที่ถูก archive แล้ว

### 292.3 ใช้เว็บไซต์ผู้ช่วยค้นหาโดยเฉพาะ

นอกจาก GitHub Search แล้ว มีเว็บไซต์ third-party ที่รวบรวม Issue สำหรับมือใหม่ไว้แล้วโดยเฉพาะ ทำให้สะดวกขึ้นมาก:

| เว็บไซต์ | จุดเด่น |
|---|---|
| `goodfirstissue.dev` | รวม issue label `good first issue` จากหลายภาษา กรองตาม tech stack ได้ |
| `up-for-grabs.net` | รวม repository ที่เปิดรับ contributor ใหม่โดยเฉพาะ |
| `firstcontributions/first-contributions` (บน GitHub) | Repository ที่สร้างมาเพื่อฝึกทำ PR แรกในชีวิตโดยเฉพาะ เกือบทุก PR จะถูก merge เพื่อให้กำลังใจผู้เริ่มต้น (เหมาะมากสำหรับ **เส้นทาง B** ใน Step 291) |
| `codetriage.com` | สุ่มส่ง Issue ที่เหมาะกับคุณมาทาง email ทุกวัน |

### 292.4 ตัวอย่างการตัดสินใจเลือก Issue

สมมติคุณค้นเจอ 3 ตัวเลือกจาก query ด้านบน:

| Issue | Repo | Star | คอมเมนต์ล่าสุด | ความยาก |
|---|---|---|---|---|
| "Fix typo in installation docs" | repo-x (2.3k star) | 2 วันก่อน | ง่ายมาก แก้ typo 1 บรรทัด |
| "Add unit test for `formatDate()` util" | repo-y (890 star) | 5 วันก่อน | ปานกลาง ต้องอ่าน test framework ก่อน |
| "Implement dark mode toggle" | repo-z (15k star) | 3 ชั่วโมงก่อน (มีคนแย่งจอง comment แล้ว 4 คน) | ยาก มีคนแข่งเยอะ |

สำหรับมือใหม่ **ควรเลือก Issue แรกหรือที่สอง** เพราะ:
1. เข้าใจ scope ของงานได้ชัดเจน แก้ไขได้ในเวลาจำกัด
2. ยังไม่มีคนแย่งทำ ลดโอกาสที่ PR ของคุณจะถูกปฏิเสธเพราะซ้ำกับคนอื่น
3. เหมาะกับการฝึกกระบวนการทั้งหมดของ Part นี้มากกว่าการเน้นความยากของโค้ด

### 292.5 การ "จอง" Issue อย่างสุภาพก่อนเริ่มทำ

เมื่อเลือก Issue ได้แล้ว **อย่าลงมือทำทันทีโดยไม่บอกใคร** ให้คอมเมนต์ในหน้า Issue ก่อนเสมอ (ทบทวนมารยาทจาก Part 20 และ Part 28):

```markdown
Hi! I'd like to work on this issue as part of my open source learning journey.
Could you please assign it to me? Thank you!
```

การจองก่อนช่วยลดโอกาสที่สองคนจะทำงานซ้ำกันโดยไม่รู้ตัว และเป็นมารยาทพื้นฐานของชุมชน Open Source ทั่วโลก

---

## Step 293: อ่าน CONTRIBUTING.md และ CODE_OF_CONDUCT.md ของโปรเจกต์เป้าหมายก่อนเริ่ม

นี่คือขั้นตอนที่มือใหม่ข้ามบ่อยที่สุด แล้วมักตามมาด้วยการถูกปฏิเสธ PR ทั้งที่โค้ดถูกต้องทุกอย่าง เพราะ**ไม่ทำตามกติกาของบ้านคนอื่น**

### 293.1 ทำไม CONTRIBUTING.md ถึงสำคัญกว่าที่คิด

โปรเจกต์ Open Source แต่ละที่มี "วัฒนธรรม" ของตัวเอง เปรียบเหมือนการไปเป็นแขกที่บ้านคนอื่น — กติกาบ้านแต่ละหลังไม่เหมือนกัน ไฟล์ `CONTRIBUTING.md` (มักอยู่ที่ root ของ repo หรือในโฟลเดอร์ `.github/`) คือ "กติกาบ้าน" ที่ maintainer เขียนไว้ให้อ่านก่อนเสมอ

### 293.2 สิ่งที่ต้องมองหาใน CONTRIBUTING.md

| หัวข้อที่ควรมองหา | ทำไมสำคัญ |
|---|---|
| วิธีตั้งค่า development environment | บางโปรเจกต์ต้องใช้ Node version เฉพาะ, ต้อง run script ติดตั้ง dependency ก่อน |
| รูปแบบ Branch naming convention | เช่น `fix/xxx`, `feature/xxx`, `docs/xxx` — ผิดรูปแบบอาจโดนขอให้เปลี่ยนชื่อใหม่ |
| รูปแบบ Commit message | บางโปรเจกต์บังคับใช้ Conventional Commits (`fix:`, `feat:`, `docs:`) บางที่ใช้ free-form |
| การรัน Test/Lint ก่อนส่ง PR | คำสั่งเช่น `npm test`, `npm run lint` ที่ต้องผ่านก่อน push |
| การเซ็น DCO (Developer Certificate of Origin) หรือ CLA | บางโปรเจกต์ (โดยเฉพาะของบริษัทใหญ่) บังคับให้ commit มี `Signed-off-by:` ผ่านคำสั่ง `git commit -s` |
| ขนาดของ PR ที่ยอมรับได้ | หลายโปรเจกต์ขอให้ 1 PR แก้ 1 เรื่องเท่านั้น (small, focused PRs) |
| ช่องทางสื่อสารเพิ่มเติม | Discord, Slack, GitHub Discussions (ทบทวนจาก Part 28) สำหรับถามคำถามก่อนเริ่ม |

### 293.3 ตัวอย่างเนื้อหา CONTRIBUTING.md ที่พบได้บ่อย

```markdown
# Contributing to this project

## Before you start
1. Search existing issues before opening a new one.
2. Comment on the issue you want to work on before starting.

## Development setup
​```bash
git clone https://github.com/your-username/project-name.git
cd project-name
npm install
npm run dev
​```

## Making changes
- Create a branch named `fix/short-description` or `feature/short-description`.
- Keep each pull request focused on a single change.
- Run `npm test` and `npm run lint` before pushing.

## Commit message format
We use Conventional Commits:
​```
fix: correct typo in installation guide
feat: add dark mode toggle to settings page
​```

## Sign-off requirement
All commits must be signed off:
​```bash
git commit -s -m "your message"
​```
```

สังเกตว่ากติกาแต่ละโปรเจกต์ต่างกันได้มาก การอ่านให้ครบก่อนลงมือ ช่วยประหยัดเวลาของทั้งคุณและ maintainer อย่างมหาศาล

### 293.4 CODE_OF_CONDUCT.md — กติกามารยาทของชุมชน

ไฟล์นี้ (มักใช้มาตรฐาน [Contributor Covenant](https://www.contributor-covenant.org/)) กำหนดพฤติกรรมที่ยอมรับได้และไม่ได้ในชุมชน เช่น:

- ห้ามใช้ถ้อยคำเหยียดหยาม คุกคาม หรือเลือกปฏิบัติ
- ห้าม spam หรือโฆษณาที่ไม่เกี่ยวข้องใน Issue/PR/Discussions
- วิธีรายงานพฤติกรรมที่ไม่เหมาะสม (มักมีอีเมลของทีม maintainer ระบุไว้ท้ายไฟล์)

แม้จะดูเป็นเอกสารทั่วไป แต่การอ่านก่อนช่วยให้คุณสื่อสารกับ maintainer และ contributor คนอื่นอย่างมืออาชีพตั้งแต่คอมเมนต์แรก

### 293.5 เช็คลิสต์ก่อนไป Step ถัดไป

- [ ] อ่าน `CONTRIBUTING.md` ของโปรเจกต์เป้าหมายจนจบแล้ว
- [ ] อ่าน `CODE_OF_CONDUCT.md` จนจบแล้ว
- [ ] รู้ branch naming convention ของโปรเจกต์นี้แล้ว
- [ ] รู้ว่าต้องรันคำสั่งอะไรก่อน push (test/lint/build)
- [ ] รู้ว่าโปรเจกต์นี้บังคับ sign-off (`git commit -s`) หรือไม่
- [ ] ได้รับการ assign Issue จาก maintainer เรียบร้อยแล้ว (หรือรอคอมเมนต์ตอบรับ)

---

## Step 294: Fork, Clone, ตั้งค่า Upstream Remote ตามที่เรียนใน Part 24

ทบทวนแนวคิดจาก **Part 24: Fork และการ Contribute แบบ Open Source เบื้องต้น** — เนื่องจากคุณไม่มีสิทธิ์ push เข้า repository ต้นทางโดยตรง (ไม่ใช่ collaborator ของโปรเจกต์) กระบวนการมาตรฐานคือ **Fork → Clone → ตั้งค่า upstream → ทำงานบน fork → ส่ง PR ข้ามกลับไปยัง repo ต้นทาง**

### 294.1 Fork repository ผ่านหน้าเว็บ

เปิดหน้า repository เป้าหมายบน GitHub แล้วกดปุ่ม **Fork** มุมขวาบน เลือกบัญชีของคุณเป็นปลายทาง

ผลลัพธ์: คุณจะได้ repository ใหม่ที่ `https://github.com/<your-username>/<project-name>` ซึ่งเป็น**สำเนาอิสระ**ของ repo ต้นทาง (เรียกว่า **upstream**) ที่คุณมีสิทธิ์ push เข้าไปได้เต็มที่

หรือใช้ GitHub CLI ทำในคำสั่งเดียว (รวม fork และ clone):

```bash
gh repo fork owner-name/project-name --clone=true
```

```
✓ Created fork your-username/project-name
Cloning into 'project-name'...
✓ Cloned fork
✓ Added remote upstream
```

สังเกตว่า `gh repo fork --clone=true` ฉลาดมาก — มันทำครบทั้ง fork, clone และตั้งค่า remote `upstream` ให้อัตโนมัติในคำสั่งเดียว (รายละเอียดของสิ่งที่มันทำให้อัตโนมัติ จะอธิบายแบบ manual ในหัวข้อถัดไป เผื่อกรณีที่คุณไม่ได้ใช้ `gh` หรืออยากเข้าใจกลไกเบื้องหลัง)

### 294.2 Clone fork ของคุณมาที่เครื่อง (ถ้ายังไม่ได้ clone มาจากขั้นตอนก่อนหน้า)

```bash
cd ~/git-course/part-30-open-source-contribution
git clone git@github.com:your-username/project-name.git
cd project-name
```

```
Cloning into 'project-name'...
remote: Enumerating objects: 4821, done.
Receiving objects: 100% (4821/4821), 3.2 MiB | 5.1 MiB/s, done.
Resolving deltas: 100% (2890/2890), done.
```

สังเกตว่าเราใช้ URL แบบ SSH (`git@github.com:...`) ตามที่ตั้งค่าไว้ใน Part 18 ไม่ใช่ HTTPS เพื่อไม่ต้องกรอก username/password หรือ token ทุกครั้งที่ push

### 294.3 ตรวจสอบ remote ปัจจุบัน

```bash
git remote -v
```

```
origin	git@github.com:your-username/project-name.git (fetch)
origin	git@github.com:your-username/project-name.git (push)
```

ตอนนี้มีแค่ `origin` ซึ่งชี้ไปที่ **fork ของคุณ** เท่านั้น — นี่คือจุดสำคัญที่มือใหม่มักสับสน: `origin` ในบริบทของการ contribute open source **ไม่ใช่ repo ต้นทาง** แต่เป็น fork ของตัวเอง

### 294.4 เพิ่ม remote upstream ชี้ไปที่ repo ต้นทาง

```bash
git remote add upstream git@github.com:owner-name/project-name.git
git remote -v
```

```
origin	  git@github.com:your-username/project-name.git (fetch)
origin	  git@github.com:your-username/project-name.git (push)
upstream  git@github.com:owner-name/project-name.git (fetch)
upstream  git@github.com:owner-name/project-name.git (push)
```

ตอนนี้เรามี remote สองตัวที่ทำหน้าที่ต่างกันชัดเจน:

| Remote | ชี้ไปที่ | ใช้ทำอะไร |
|---|---|---|
| `origin` | Fork ของคุณ | push branch ที่คุณทำงาน, ที่เดียวที่คุณมีสิทธิ์เขียนได้ |
| `upstream` | Repo ต้นทางของโปรเจกต์ | ดึงความเปลี่ยนแปลงล่าสุดมา sync กับ fork ของคุณ (fetch/pull เท่านั้น ปกติไม่มีสิทธิ์ push) |

> **ข้อควรระวังด้านความปลอดภัย:** ตรวจสอบเสมอว่าคุณกำลัง push ไปที่ `origin` (fork ของคุณ) ไม่ใช่ `upstream` โดยไม่ตั้งใจ — ปกติแล้วคุณจะไม่มีสิทธิ์ push ไปยัง `upstream` อยู่แล้วเพราะไม่ใช่ collaborator แต่การรู้ความแตกต่างตั้งแต่ต้นช่วยป้องกันความสับสนได้มาก

### 294.5 Sync branch หลักของ fork ให้ตรงกับ upstream ก่อนเริ่มงาน

ก่อนแตก branch ใหม่ ควรดึงความเปลี่ยนแปลงล่าสุดจาก upstream มาก่อนเสมอ เพราะ fork ของคุณอาจถูกสร้างไว้นานแล้วและตกรุ่นไปแล้ว:

```bash
git fetch upstream
git switch main
git merge upstream/main
```

```
From github.com:owner-name/project-name
 * [new branch]      main       -> upstream/main
Updating a1b2c3d..f9e8d7c
Fast-forward
 12 files changed, 340 insertions(+), 58 deletions(-)
```

จากนั้น push branch `main` ของ fork ให้ตรงกับ upstream ด้วย (ไม่บังคับ แต่เป็นนิสัยที่ดี เพื่อให้ fork บนเว็บของคุณไม่ตกรุ่นด้วย):

```bash
git push origin main
```

> **หมายเหตุ:** บางโปรเจกต์ใช้ชื่อ branch หลักว่า `master` แทน `main` — ให้ตรวจสอบด้วย `git branch -a` หรือดูจากหน้าเว็บ repo เสมอก่อนรันคำสั่งด้านบน

### 294.6 ติดตั้ง dependency และรันโปรเจกต์ตาม CONTRIBUTING.md

ตามที่อ่านมาใน Step 293 ให้ตั้งค่า development environment ตามที่โปรเจกต์กำหนด เช่น:

```bash
npm install
npm run test
```

ตรวจสอบว่า test ทั้งหมดผ่านตั้งแต่ก่อนแก้โค้ดเลย — ถ้า test พังตั้งแต่ยังไม่แตะโค้ดอะไรเลย แปลว่าปัญหาอยู่ที่การตั้งค่าเครื่องของคุณ ไม่ใช่โค้ดของโปรเจกต์ ควรแก้ตรงนี้ให้เรียบร้อยก่อนไปต่อ

---

## Step 295: สร้าง Branch และลงมือแก้ไข/เพิ่มโค้ดจริงตามที่ Issue กำหนด

### 295.1 สร้าง branch ตามรูปแบบที่โปรเจกต์กำหนด

ทบทวนจาก Part 07 (เฟส 2) เรื่องการสร้าง branch โดยครั้งนี้ต้องปฏิบัติตามข้อกำหนดใน `CONTRIBUTING.md` ที่อ่านมาใน Step 293 ด้วย ตัวอย่างสมมติว่า Issue ที่รับมาคือ **"Fix typo in installation docs (#482)"**:

```bash
git switch -c fix/482-typo-in-installation-docs
```

```
Switched to a new branch 'fix/482-typo-in-installation-docs'
```

การใส่เลข Issue ไว้ในชื่อ branch (`482`) เป็นธรรมเนียมที่หลายโปรเจกต์นิยม เพราะช่วยให้ maintainer เชื่อมโยง branch กับ Issue ได้ทันทีโดยไม่ต้องเปิดอ่าน PR description ก่อน

### 295.2 อ่านรายละเอียด Issue ให้เข้าใจ scope ก่อนแก้จริง

สมมติเนื้อหา Issue #482 ระบุว่า:

```markdown
## Bug: Typo in installation docs

In `docs/installation.md`, line 24, "recieve" should be "receive".

Steps to reproduce:
1. Open docs/installation.md
2. See line 24
```

ตรวจสอบตำแหน่งจริงในไฟล์ก่อนแก้:

```bash
grep -n "recieve" docs/installation.md
```

```
24:After installing the CLI, you will recieve a confirmation message.
```

### 295.3 ลงมือแก้ไขจริง

ใช้ editor ที่ถนัด หรือใช้คำสั่งแก้ไขตรง ๆ:

```bash
sed -i 's/recieve/receive/' docs/installation.md
```

ตรวจสอบผลลัพธ์ด้วย `git diff` เสมอ (ทบทวนนิสัยที่ดีจาก Part 05):

```bash
git diff
```

```diff
diff --git a/docs/installation.md b/docs/installation.md
index 3f2a1b0..7c9d4e2 100644
--- a/docs/installation.md
+++ b/docs/installation.md
@@ -21,7 +21,7 @@
 ## Verify installation

-After installing the CLI, you will recieve a confirmation message.
+After installing the CLI, you will receive a confirmation message.
```

### 295.4 ตัวอย่างที่ซับซ้อนกว่า: การเพิ่ม Unit Test (สำหรับผู้ที่เลือก Issue ระดับปานกลาง)

ถ้า Issue ที่คุณรับมาคือ **"Add unit test for `formatDate()` util (#510)"** แทนที่จะแก้ typo กระบวนการจะลึกขึ้น:

```bash
git switch -c feature/510-add-formatdate-test
```

ก่อนเขียน test ต้องอ่านโค้ดต้นฉบับก่อนเสมอ:

```bash
cat src/utils/formatDate.js
```

```javascript
export function formatDate(date) {
  const d = new Date(date);
  const day = String(d.getDate()).padStart(2, "0");
  const month = String(d.getMonth() + 1).padStart(2, "0");
  const year = d.getFullYear();
  return `${day}/${month}/${year}`;
}
```

จากนั้นเขียน test ตามรูปแบบ testing framework ที่โปรเจกต์ใช้อยู่แล้ว (ตรวจสอบได้จาก test ไฟล์อื่นในโปรเจกต์ — อย่าคิดค้นรูปแบบใหม่เอง):

```bash
cat > src/utils/formatDate.test.js << 'EOF'
import { formatDate } from "./formatDate";

describe("formatDate", () => {
  it("formats a date as DD/MM/YYYY", () => {
    expect(formatDate("2026-03-05")).toBe("05/03/2026");
  });

  it("pads single-digit day and month with zero", () => {
    expect(formatDate("2026-01-09")).toBe("09/01/2026");
  });
});
EOF
```

### 295.5 รัน Test และ Lint ก่อน commit ทุกครั้ง

```bash
npm run test
npm run lint
```

```
PASS  src/utils/formatDate.test.js
  formatDate
    ✓ formats a date as DD/MM/YYYY (3 ms)
    ✓ pads single-digit day and month with zero (1 ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
```

```
✔ No ESLint warnings or errors
```

> **กฎทองของการ contribute open source:** อย่า push โค้ดที่ยังไม่ผ่าน test หรือ lint ของโปรเจกต์เด็ดขาด เพราะ CI (Continuous Integration) ของโปรเจกต์ส่วนใหญ่จะรันตรวจสอบอัตโนมัติทันทีที่คุณเปิด PR และ maintainer มักจะไม่รีวิว PR ที่ CI สีแดงเลย

### 295.6 Commit งานเป็นชิ้นเล็ก มีความหมายชัดเจน (ยังไม่ push ในขั้นตอนนี้)

```bash
git status
```

```
On branch feature/510-add-formatdate-test
Changes not staged for commit:
	modified:   src/utils/formatDate.js

Untracked files:
	src/utils/formatDate.test.js
```

เราจะยังไม่ commit ทันที เพราะ Step 296 จะพาไปเจาะลึกเรื่องการเขียน commit message ให้ตรงมาตรฐานของโปรเจกต์อย่างละเอียด

---

## Step 296: เขียน Commit Message และ PR Description ตามมาตรฐานของโปรเจกต์นั้น ๆ

### 296.1 สังเกตรูปแบบ commit message จาก PR ที่ merge แล้วก่อนเขียนของตัวเอง

ก่อนเขียน commit message ของตัวเอง ให้เข้าไปดู tab **"Insights → Commits"** หรือหน้า **"Pull requests → Closed"** ของโปรเจกต์ต้นทาง เพื่อดูว่า maintainer และ contributor คนอื่นเขียนกันแบบไหน

ตัวอย่างที่อาจพบสองรูปแบบหลัก:

**รูปแบบที่ 1: Conventional Commits** (พบมากในโปรเจกต์ยุคใหม่)

```
fix: correct typo "recieve" to "receive" in installation docs
feat: add dark mode toggle to settings panel
docs: update contributing guide with new test command
test: add unit tests for formatDate utility
```

**รูปแบบที่ 2: Free-form ที่มีวินัย** (พบในโปรเจกต์เก่าแก่ เช่นสาย Linux Kernel)

```
docs: Fix "recieve" typo in installation.md

The word was misspelled in the verify installation section,
causing confusion for first-time readers.
```

> **หมายเหตุ:** เราจะเจาะลึกมาตรฐาน Conventional Commits แบบเต็มรูปแบบใน **Part 35: การตั้งชื่อ Branch และ Commit Message Convention** ของเฟส 4 — ใน Part นี้ให้เน้นแค่การ **สังเกตและทำตาม** สิ่งที่โปรเจกต์เป้าหมายใช้อยู่แล้วเป็นหลัก

### 296.2 เขียน commit message ให้ตรงกับที่สังเกตมา

สำหรับตัวอย่าง Issue #482 (แก้ typo) ที่โปรเจกต์ใช้ Conventional Commits:

```bash
git add docs/installation.md
git commit -m "docs: fix typo 'recieve' to 'receive' in installation guide"
```

```
[fix/482-typo-in-installation-docs 3a4b5c6] docs: fix typo 'recieve' to 'receive' in installation guide
 1 file changed, 1 insertion(+), 1 deletion(-)
```

สำหรับตัวอย่าง Issue #510 (เพิ่ม test):

```bash
git add src/utils/formatDate.test.js
git commit -m "test: add unit tests for formatDate utility"
```

```
[feature/510-add-formatdate-test 8d9e0f1] test: add unit tests for formatDate utility
 1 file changed, 12 insertions(+)
 create mode 100644 src/utils/formatDate.test.js
```

### 296.3 กรณีที่โปรเจกต์บังคับ Sign-off (DCO)

ถ้าใน `CONTRIBUTING.md` ระบุว่าต้องเซ็น Developer Certificate of Origin ให้ใช้ flag `-s`:

```bash
git commit -s -m "docs: fix typo 'recieve' to 'receive' in installation guide"
```

Git จะเพิ่มบรรทัดนี้ท้าย commit message ให้อัตโนมัติ:

```
Signed-off-by: Your Name <your-email@example.com>
```

หากลืม sign-off ใน commit ที่ทำไปแล้ว สามารถแก้ย้อนหลังได้ด้วย (ทบทวนแนวคิดจาก Part 14):

```bash
git commit --amend -s --no-edit
```

### 296.4 Push branch ขึ้นไปที่ fork (`origin`) ของคุณ

```bash
git push -u origin fix/482-typo-in-installation-docs
```

```
Enumerating objects: 7, done.
Writing objects: 100% (4/4), 412 bytes | 412.00 KiB/s, done.
remote: Create a pull request for 'fix/482-typo-in-installation-docs' on GitHub by visiting:
remote:      https://github.com/your-username/project-name/pull/new/fix/482-typo-in-installation-docs
To github.com:your-username/project-name.git
 * [new branch]      fix/482-typo-in-installation-docs -> fix/482-typo-in-installation-docs
```

สังเกตว่าเรา push ไปที่ `origin` (fork ของเรา) เท่านั้น ไม่ใช่ `upstream` — ตรงกับหลักการที่วางไว้ใน Step 294

### 296.5 เขียน PR Description ตาม Pull Request Template ของโปรเจกต์

หลายโปรเจกต์มีไฟล์ `.github/PULL_REQUEST_TEMPLATE.md` ที่จะถูกดึงมาใส่ในกล่องข้อความอัตโนมัติเมื่อกดเปิด PR บนเว็บ (ทบทวนจาก **Part 26: Labels, Milestones, Issue/PR Templates**) ตัวอย่าง template ที่พบบ่อย:

```markdown
## Description
<!-- What does this PR do? -->

## Related issue
Closes #

## Type of change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation update
- [ ] Other

## Checklist
- [ ] Tests pass locally
- [ ] I have read the CONTRIBUTING guide
- [ ] I have added tests that prove my fix/feature works
```

กรอกให้ครบตามตัวอย่าง Issue #482:

```markdown
## Description
Fixes a typo ("recieve" -> "receive") in the installation guide that was
reported in the linked issue.

## Related issue
Closes #482

## Type of change
- [x] Documentation update

## Checklist
- [x] Tests pass locally (docs-only change, no tests required)
- [x] I have read the CONTRIBUTING guide
- [ ] I have added tests that prove my fix/feature works
```

> **จุดสำคัญ:** การเขียน `Closes #482` (หรือ `Fixes #482`, `Resolves #482`) ในคำอธิบาย PR คือ **keyword พิเศษที่ GitHub รู้จัก** — เมื่อ PR นี้ถูก merge เข้า branch หลัก GitHub จะ**ปิด Issue #482 ให้อัตโนมัติทันที** โดยไม่ต้องปิดมือเอง

---

## Step 297: ส่ง PR ข้าม Fork และรอ/ตอบสนอง Code Review จริงจาก Maintainer

### 297.1 เปิด Pull Request แบบข้าม repository (Cross-repository Pull Request)

นี่คือความแตกต่างสำคัญจากการทำ PR ภายในโปรเจกต์ของตัวเองที่เรียนใน Part 21 — ครั้งนี้ **base repository** (ปลายทางที่จะ merge เข้า) กับ **head repository** (ต้นทางที่มีการเปลี่ยนแปลง) เป็นคนละ repository กัน

ผ่านหน้าเว็บ: เปิดหน้า fork ของคุณ จะเห็นแถบแจ้งเตือน **"This branch is X commits ahead of owner-name:main"** พร้อมปุ่ม **Compare & pull request** กดปุ่มนี้ GitHub จะตั้งค่า base/head ให้อัตโนมัติ:

```
base repository: owner-name/project-name   base: main
head repository: your-username/project-name   compare: fix/482-typo-in-installation-docs
```

หรือใช้ GitHub CLI (ทบทวนจาก Part 21):

```bash
gh pr create \
  --repo owner-name/project-name \
  --base main \
  --head your-username:fix/482-typo-in-installation-docs \
  --title "docs: fix typo 'recieve' to 'receive' in installation guide" \
  --body "Closes #482

Fixes a typo (\"recieve\" -> \"receive\") in the installation guide."
```

```
Creating pull request for your-username:fix/482-typo-in-installation-docs into main in owner-name/project-name

https://github.com/owner-name/project-name/pull/1284
```

### 297.2 ตรวจสอบสถานะ CI ที่รันอัตโนมัติบน PR

ทันทีที่เปิด PR โปรเจกต์ส่วนใหญ่จะรัน CI (ผ่าน GitHub Actions ซึ่งจะเรียนละเอียดในเฟส 7) อัตโนมัติ ให้รอตรวจสอบผลก่อนแจ้ง maintainer:

```bash
gh pr checks 1284 --repo owner-name/project-name
```

```
All checks were successful
0 cancelled, 0 failing, 3 successful, 0 skipped, 0 pending
```

ถ้ามี check ที่ล้มเหลว (แสดงเป็นกากบาทสีแดงบนหน้า PR) ให้ **แก้ไขก่อนที่จะรบกวน maintainer** เพราะการขอ review ทั้งที่ CI ยังแดงถือเป็นมารยาทที่ไม่ดีในหลายชุมชน

### 297.3 มารยาทระหว่างรอ Code Review

หลังส่ง PR แล้ว ไม่ควรเร่ง maintainer ทันที ให้ปฏิบัติตามแนวทางนี้:

| ระยะเวลาที่ผ่านไป | สิ่งที่ควรทำ |
|---|---|
| 0–3 วัน | รอเฉย ๆ ตรวจสอบ notification เป็นระยะ |
| 4–7 วัน | ตรวจสอบว่า CONTRIBUTING.md ระบุเวลารอมาตรฐานไว้หรือไม่ |
| 1–2 สัปดาห์ | คอมเมนต์สุภาพถามความคืบหน้าได้ เช่น "Hi, just checking in — happy to make any changes needed" |
| มากกว่า 1 เดือนไม่มีการตอบรับเลย | อาจเป็นสัญญาณว่าโปรเจกต์ maintain ไม่บ่อย พิจารณาหาโปรเจกต์อื่นคู่ขนานไปได้ |

### 297.4 สมัคร subscribe การแจ้งเตือนของ PR

เพื่อไม่พลาดคอมเมนต์จาก maintainer ให้เปิดการแจ้งเตือน (Watch) ที่ repository หรือกด **Subscribe** ที่มุมขวาของหน้า PR โดยเฉพาะ เพื่อรับอีเมล/notification ทันทีที่มีความเคลื่อนไหว

---

## Step 298: การจัดการ Feedback และแก้ไข PR ตามคำแนะนำ

### 298.1 ตัวอย่าง Feedback จริงที่มักได้รับจาก Maintainer

สมมติ maintainer ของ Issue #510 (unit test) รีวิวแล้วคอมเมนต์แบบนี้ (ทบทวนรูปแบบการรับ feedback จาก **Part 22: Code Review บน GitHub**):

```
💬 maintainer-name commented on src/utils/formatDate.test.js line 8:

Thanks for the PR! Could you also add a test case for an invalid date
input, e.g. formatDate("not-a-date")? We want to make sure it doesn't
throw an unhandled exception.
```

### 298.2 แก้ไขโค้ดเพิ่มตาม Feedback

```bash
git switch feature/510-add-formatdate-test
```

แก้ไขไฟล์ test เพิ่ม test case ตามที่ maintainer ขอ:

```bash
cat >> src/utils/formatDate.test.js << 'EOF'

  it("returns 'Invalid Date' string for an unparsable input", () => {
    expect(formatDate("not-a-date")).toBe("Invalid Date");
  });
EOF
```

รัน test ซ้ำเพื่อยืนยันว่าผ่านก่อน commit:

```bash
npm run test
```

```
PASS  src/utils/formatDate.test.js
  formatDate
    ✓ formats a date as DD/MM/YYYY (2 ms)
    ✓ pads single-digit day and month with zero (1 ms)
    ✓ returns 'Invalid Date' string for an unparsable input (1 ms)
```

### 298.3 Commit เพิ่มเข้า branch เดิม (ไม่สร้าง branch ใหม่)

จุดสำคัญที่สุดของ Step นี้: **ไม่ต้องปิด PR เดิมแล้วเปิดใหม่** เพียงแค่ commit เพิ่มเข้า branch เดิม แล้ว push ขึ้นไปที่ `origin` เหมือนเดิม PR ที่เปิดอยู่แล้วจะ**อัปเดตอัตโนมัติ**ทันที:

```bash
git add src/utils/formatDate.test.js
git commit -m "test: add test case for invalid date input"
git push origin feature/510-add-formatdate-test
```

```
[feature/510-add-formatdate-test 2c3d4e5] test: add test case for invalid date input
 1 file changed, 4 insertions(+)
To github.com:your-username/project-name.git
   8d9e0f1..2c3d4e5  feature/510-add-formatdate-test -> feature/510-add-formatdate-test
```

กลับไปดูหน้า PR บนเว็บ จะเห็น commit ใหม่ถูกเพิ่มเข้าไปในไทม์ไลน์ของ PR เดิมทันที พร้อมสถานะ CI รันใหม่อัตโนมัติอีกครั้ง

### 298.4 ตอบกลับคอมเมนต์และ Resolve Conversation

หลัง push แก้ไขแล้ว ให้กลับไปตอบคอมเมนต์ของ maintainer เพื่อแจ้งว่าคุณแก้ไขแล้ว:

```markdown
Good catch! I've added a test case for invalid date input in the latest commit. Thanks for the review!
```

จากนั้นกดปุ่ม **"Resolve conversation"** ที่คอมเมนต์นั้น (ทบทวนกลไกนี้จาก Part 22) เพื่อบอกทุกคนที่ตามอ่าน PR ว่าประเด็นนี้ถูกจัดการเรียบร้อยแล้ว

### 298.5 กรณีที่ต้อง Sync กับ upstream/main ก่อน merge (แก้ Conflict ระหว่าง PR ค้าง)

ถ้า branch หลักของโปรเจกต์มีการเปลี่ยนแปลงใหม่ระหว่างที่ PR ของคุณยังค้างรอ review อยู่ (พบบ่อยในโปรเจกต์ที่มีคนทำงานพร้อมกันหลายคน) GitHub อาจแสดงข้อความ **"This branch has conflicts that must be resolved"** ให้แก้ด้วยการดึง upstream มา merge เข้า branch ของคุณ:

```bash
git fetch upstream
git merge upstream/main
```

ถ้าเกิด conflict ให้แก้ตามหลักการที่เรียนมาใน Part 08 (เฟส 2) แล้ว commit และ push ซ้ำอีกครั้ง:

```bash
git add <ไฟล์ที่แก้ conflict>
git commit
git push origin feature/510-add-formatdate-test
```

> **ทางเลือกขั้นสูง:** บางโปรเจกต์นิยมให้ `git rebase upstream/main` แทน `git merge` เพื่อให้ประวัติ commit เป็นเส้นตรงสวยงามกว่า เรื่องนี้จะเจาะลึกอย่างละเอียดใน **Part 38: Rebase vs Merge** ของเฟส 4 — ในตอนนี้ให้ใช้ `merge` ไปก่อนเพื่อความปลอดภัย เว้นแต่ `CONTRIBUTING.md` จะระบุชัดเจนว่าต้องการ rebase

### 298.6 เมื่อ PR ถูก Approve และ Merge สำเร็จ

เมื่อ maintainer กด **Approve** และ **Merge** แล้ว คุณจะเห็นสถานะ PR เปลี่ยนเป็นสีม่วง **Merged** และ Issue ที่เชื่อมไว้ด้วย `Closes #xxx` จะถูกปิดให้อัตโนมัติทันที

ขั้นตอนทำความสะอาดหลัง merge สำเร็จ:

```bash
git switch main
git fetch upstream
git merge upstream/main
git push origin main
git branch -d feature/510-add-formatdate-test
git push origin --delete feature/510-add-formatdate-test
```

```
Deleted branch feature/510-add-formatdate-test (was 2c3d4e5).
To github.com:your-username/project-name.git
 - [deleted]         feature/510-add-formatdate-test
```

การลบ branch ทั้งใน local และ remote (fork) หลัง merge เสร็จ เป็นนิสัยที่ดีที่ช่วยให้ repository สะอาด ไม่มี branch ค้างเก่า ๆ สะสม

---

## Step 299: สิ่งที่ได้เรียนรู้จากการ Contribute จริง (Reflection Questions)

การ contribute ให้ open source ครั้งแรกมักให้บทเรียนมากกว่าการอ่านเอกสารหลายสิบหน้า Step นี้ออกแบบมาให้คุณ**หยุดคิดทบทวน**อย่างจริงจังก่อนไปต่อ ลองเขียนคำตอบของคำถามต่อไปนี้ลงในไฟล์บันทึกส่วนตัว (แนะนำให้เขียนจริง ไม่ใช่แค่คิดในหัว)

### 299.1 คำถามด้านกระบวนการทำงาน (Workflow)

1. ตั้งแต่ fork จนถึง merge (หรือจนถึงจุดที่คุณอยู่ตอนนี้) ขั้นตอนไหนที่ใช้เวลานานที่สุด และทำไม
2. คุณใช้ `git remote -v` ตรวจสอบ `origin` กับ `upstream` กี่ครั้งกว่าจะมั่นใจว่าไม่ push ผิดที่
3. มีจุดไหนบ้างที่คุณต้องย้อนกลับไปอ่าน Part ก่อนหน้าซ้ำ (Part 16–29) เพื่อทบทวนคำสั่งที่ลืม

### 299.2 คำถามด้านการสื่อสาร (Communication)

4. คอมเมนต์ที่คุณเขียนขอรับ Issue หรือตอบ Code Review มีอะไรที่ควรปรับปรุงให้สุภาพหรือชัดเจนกว่านี้หรือไม่
5. ถ้า maintainer ให้ feedback ที่คุณไม่เห็นด้วย คุณจะสื่อสารอย่างไรให้ยังคงมืออาชีพ (ลองนึกสถานการณ์สมมติแม้ยังไม่เจอจริง)
6. คุณอ่าน `CODE_OF_CONDUCT.md` ของโปรเจกต์จริงหรือแค่ผ่านตา — ลองสรุปใจความสำคัญ 3 ข้อจากไฟล์นั้นด้วยคำพูดตัวเอง

### 299.3 คำถามด้านเทคนิค (Technical)

7. โปรเจกต์ที่คุณเลือกใช้ branch naming convention และ commit message convention แบบไหน ต่างจากที่คุณเคยชินมาก่อนหน้านี้อย่างไร
8. คุณเจอ Merge Conflict ระหว่างทำ PR นี้หรือไม่ ถ้าเจอ แก้ด้วยวิธีไหน และมั่นใจในผลลัพธ์แค่ไหน
9. ถ้าให้ทำ PR แบบเดียวกันนี้อีกครั้งตอนนี้ (โดยไม่เปิดเอกสารนี้ดู) คุณจะจำขั้นตอนทั้งหมดได้กี่เปอร์เซ็นต์

### 299.4 คำถามเชิงภาพรวม (Big Picture)

10. ก่อนเริ่ม Part นี้ คุณคิดว่าการ contribute open source ยากหรือง่ายแค่ไหน หลังทำจริงแล้วความคิดเปลี่ยนไปอย่างไร
11. ทักษะจาก Part ไหนใน Part 16–29 ที่คุณรู้สึกว่า "ใช้จริงเยอะที่สุด" ใน Part นี้ และ Part ไหนที่ "ใช้น้อยที่สุด" (ลองอธิบายเหตุผล ไม่มีคำตอบผิด)
12. ถ้ามีคนที่เพิ่งเริ่มเรียน Git มาถามคุณว่า "จะเริ่ม contribute open source ยังไงดี" คุณจะแนะนำเขาอย่างไรเป็นข้อ ๆ

> **เคล็ดลับ:** เก็บคำตอบเหล่านี้ไว้ในไฟล์ `REFLECTION.md` ในโฟลเดอร์ฝึกฝนของคุณเอง (`~/git-course/part-30-open-source-contribution/`) แล้วกลับมาอ่านซ้ำอีกครั้งหลังจากผ่านไป 3 เดือน — คุณจะเห็นพัฒนาการของตัวเองอย่างชัดเจนมาก

---

## Step 300: สรุปทบทวนภาพรวมเฟส 3 ทั้งหมด (Part 16–30) พร้อม Cheat Sheet และ Checklist ก่อนเข้าเฟส 4

ยินดีด้วย — คุณเพิ่งผ่านภารกิจปิดเฟส 3 ที่สมบูรณ์แบบที่สุด ตั้งแต่การค้นหาโปรเจกต์ ไปจนถึงการรับมือ Code Review จริง มาถึงจุดนี้ ให้ใช้เวลาทบทวนภาพรวมทั้งหมดของเฟส 3 (Part 16–30, Step 151–300) อย่างละเอียดก่อนก้าวเข้าสู่เฟส 4

### 300.1 Cheat Sheet ฟีเจอร์และคำสั่งทั้งหมดที่เรียนมาในเฟส 3

#### จาก Part 16: บัญชี GitHub

| หัวข้อ | รายละเอียด |
|---|---|
| การตั้งค่า Profile | ชื่อ, avatar, bio, README profile พิเศษ (`username/username`) |
| การตั้งค่าความปลอดภัยบัญชี | 2FA (Two-Factor Authentication), recovery codes |
| Personal Access Token (PAT) | ใช้แทนรหัสผ่านสำหรับ HTTPS authentication |

#### จาก Part 17: สร้าง Repository และเชื่อมกับเครื่อง local

| คำสั่ง | หน้าที่ |
|---|---|
| สร้าง repo ผ่านเว็บ/`gh repo create` | สร้าง repository ใหม่บน GitHub |
| `git remote add origin <url>` | เชื่อม local repo เข้ากับ GitHub repo |
| `git push -u origin main` | push ครั้งแรกพร้อมตั้งค่า upstream tracking |

#### จาก Part 18: SSH Key

| คำสั่ง | หน้าที่ |
|---|---|
| `ssh-keygen -t ed25519 -C "email"` | สร้างคู่กุญแจ SSH |
| `ssh-add ~/.ssh/id_ed25519` | เพิ่มกุญแจเข้า ssh-agent |
| `ssh -T git@github.com` | ทดสอบการเชื่อมต่อ SSH กับ GitHub |

#### จาก Part 19: README.md และ Markdown

| องค์ประกอบ | หน้าที่ |
|---|---|
| Heading, list, table, code block | โครงสร้างพื้นฐานของเอกสาร Markdown |
| Badge (shields.io) | แสดงสถานะ build, version, license |
| Table of contents | ช่วยนำทางเอกสารยาว |

#### จาก Part 20: GitHub Issues

| คำสั่ง/ฟีเจอร์ | หน้าที่ |
|---|---|
| เปิด/ปิด Issue | รายงานบั๊กหรือขอฟีเจอร์ใหม่ |
| `Closes #123` ใน commit/PR | เชื่อม PR กับ Issue ให้ปิดอัตโนมัติเมื่อ merge |
| Assignee, Label | มอบหมายงานและจัดหมวดหมู่ |

#### จาก Part 21: Pull Request เบื้องต้น

| คำสั่ง/ฟีเจอร์ | หน้าที่ |
|---|---|
| `gh pr create` | เปิด PR ผ่าน command line |
| base branch / compare branch | กำหนดปลายทางและต้นทางของการรวมโค้ด |
| Draft Pull Request | เปิด PR แบบร่างเมื่อยังทำไม่เสร็จ |

#### จาก Part 22: Code Review

| ฟีเจอร์ | หน้าที่ |
|---|---|
| Review: Comment / Approve / Request changes | สามระดับของการให้ความเห็นต่อ PR |
| Resolve conversation | ปิดประเด็นที่แก้ไขเรียบร้อยแล้ว |
| Suggested changes | เสนอโค้ดแก้ไขให้กด apply ได้ทันที |

#### จาก Part 23: GitHub Projects

| ฟีเจอร์ | หน้าที่ |
|---|---|
| Kanban board (To do / In progress / Done) | บริหารจัดการ Issue/PR แบบมองเห็นภาพรวม |
| Automation rules | ย้ายการ์ดอัตโนมัติเมื่อสถานะ Issue/PR เปลี่ยน |

#### จาก Part 24: Fork และ Contribute เบื้องต้น

| คำสั่ง | หน้าที่ |
|---|---|
| `gh repo fork --clone=true` | Fork และ Clone ในคำสั่งเดียว |
| `git remote add upstream <url>` | เชื่อม remote ไปยัง repo ต้นทาง |
| `git fetch upstream && git merge upstream/main` | Sync fork ให้ตรงกับ repo ต้นทางล่าสุด |

#### จาก Part 25: GitHub Pages

| ฟีเจอร์ | หน้าที่ |
|---|---|
| GitHub Pages (branch `gh-pages` หรือโฟลเดอร์ `docs/`) | Host เว็บไซต์ static ฟรีจาก repository |
| Custom domain | ผูกโดเมนของตัวเองเข้ากับ GitHub Pages |

#### จาก Part 26: Labels, Milestones, Templates

| ฟีเจอร์ | หน้าที่ |
|---|---|
| Label | จัดหมวดหมู่ Issue/PR เช่น `bug`, `good first issue` |
| Milestone | กำหนดกลุ่ม Issue/PR ที่ต้องเสร็จภายในเวอร์ชัน/วันที่เดียวกัน |
| `.github/ISSUE_TEMPLATE/`, `PULL_REQUEST_TEMPLATE.md` | แม่แบบมาตรฐานสำหรับเปิด Issue/PR |

#### จาก Part 27: GitHub Wiki

| ฟีเจอร์ | หน้าที่ |
|---|---|
| Wiki pages | เอกสารประกอบโปรเจกต์ที่แก้ไขร่วมกันได้แยกจากโค้ด |
| Sidebar/Footer ของ Wiki | จัดโครงสร้างการนำทางเอกสาร |

#### จาก Part 28: GitHub Discussions

| ฟีเจอร์ | หน้าที่ |
|---|---|
| Category (Q&A, Ideas, Announcements) | จัดหมวดหมู่การพูดคุยในชุมชน |
| Mark as answer | ทำเครื่องหมายคำตอบที่ถูกต้องในกระทู้ Q&A |

#### จาก Part 29: GitHub Search

| Qualifier | หน้าที่ |
|---|---|
| `is:issue`, `is:pr`, `is:open`, `is:closed` | กรองประเภทและสถานะ |
| `label:"..."` | กรองตาม label |
| `language:...` | กรองตามภาษาโปรแกรมหลักของ repo |
| `no:assignee`, `comments:<N>` | กรองหา Issue ที่ยังไม่มีคนทำ/ยังไม่มีคนแย่งเยอะ |
| `stars:>N`, `archived:false` | กรองความน่าเชื่อถือของโปรเจกต์ |

#### จาก Part 30 (Part นี้): Contribute จริง

| ขั้นตอน | หน้าที่ |
|---|---|
| เลือก Issue ด้วย Search + เกณฑ์คุณภาพ | ลด friction และความเสี่ยง PR ถูกปฏิเสธ |
| อ่าน CONTRIBUTING.md / CODE_OF_CONDUCT.md | ทำตามกติกาบ้านคนอื่นอย่างถูกต้อง |
| Fork → Clone → upstream | โครงสร้าง remote มาตรฐานของการ contribute |
| Commit/PR ตามธรรมเนียมโปรเจกต์ | เพิ่มโอกาสถูก merge |
| ตอบสนอง Review อย่างมืออาชีพ | สร้างความสัมพันธ์ที่ดีกับ maintainer |

### 300.2 ตารางสรุป "เมื่อไหร่ควรใช้ฟีเจอร์ไหนของ GitHub" (Decision Table)

| สถานการณ์ | ฟีเจอร์/คำสั่งที่ควรใช้ |
|---|---|
| อยากรายงานบั๊กหรือขอฟีเจอร์ใหม่ | GitHub Issues (Part 20) |
| อยากเสนอการเปลี่ยนแปลงโค้ดให้ทีมพิจารณา | Pull Request (Part 21) |
| อยากให้/รับความเห็นต่อโค้ดก่อน merge | Code Review (Part 22) |
| อยากบริหารงานทั้งทีมแบบเห็นภาพรวม | GitHub Projects (Part 23) |
| อยากช่วยโปรเจกต์ที่ไม่ใช่ของตัวเอง | Fork + Pull Request ข้าม repo (Part 24, 30) |
| อยากมีเว็บไซต์ demo ฟรีจากโค้ด | GitHub Pages (Part 25) |
| อยากให้ contributor เปิด Issue/PR แบบมีมาตรฐาน | Issue/PR Templates (Part 26) |
| อยากเขียนเอกสารยาว ๆ แยกจากโค้ด | GitHub Wiki (Part 27) |
| อยากเปิดพื้นที่พูดคุยกับผู้ใช้/ชุมชน | GitHub Discussions (Part 28) |
| อยากหา Issue ที่เหมาะกับระดับตัวเอง | GitHub Search + qualifiers (Part 29) |
| อยากยืนยันว่า commit ยังไม่ push แก้ไขได้ปลอดภัย | `git reset`/`git commit --amend` (ทบทวนจากเฟส 2) |
| อยากแก้ไข PR ที่เปิดอยู่แล้วโดยไม่เปิดใหม่ | Commit เพิ่มเข้า branch เดิม + push ซ้ำ (Part 30) |

### 300.3 ทบทวนภาพรวมทุก Part ในเฟส 3 (Part 16–30)

| Part | Step | สิ่งที่เรียนรู้ | นำมาใช้จริงใน Part 30 ตรงไหน |
|---|---|---|---|
| 16 | 151–160 | สมัครและตั้งค่าบัญชี GitHub อย่างมืออาชีพ | บัญชีที่ใช้ fork/comment/สร้าง PR ตลอด Part นี้ |
| 17 | 161–170 | สร้าง Repository และเชื่อมกับเครื่อง local | พื้นฐานการเชื่อม `origin` ใน Step 294 |
| 18 | 171–180 | SSH Key และความปลอดภัยการเชื่อมต่อ | Clone/Push ผ่าน SSH ตลอด Step 294–298 |
| 19 | 181–190 | README.md ที่ดีและ Markdown | เขียน PR description และคอมเมนต์ใน Step 296–298 |
| 20 | 191–200 | GitHub Issues | เลือก จอง และปิด Issue อัตโนมัติผ่าน `Closes #` ใน Step 292, 296 |
| 21 | 201–210 | Pull Request เบื้องต้น | เปิด PR ข้าม fork ใน Step 297 |
| 22 | 211–220 | Code Review | รับและตอบสนอง feedback ใน Step 298 |
| 23 | 221–230 | GitHub Projects | แนวคิดติดตามความคืบหน้าของงานที่รับมา |
| 24 | 231–240 | Fork และการ Contribute เบื้องต้น | โครงสร้าง fork/upstream ทั้งหมดใน Step 294 |
| 25 | 241–250 | GitHub Pages | ทางเลือกโปรเจกต์ที่อาจเจอ (เว็บไซต์เอกสารของโปรเจกต์) |
| 26 | 251–260 | Labels, Milestones, Templates | เลือก Issue ด้วย label ใน Step 292, กรอก PR template ใน Step 296 |
| 27 | 261–270 | GitHub Wiki | แหล่งอ้างอิงเพิ่มเติมเวลาอ่านเอกสารโปรเจกต์เป้าหมาย |
| 28 | 271–280 | GitHub Discussions | ช่องทางถามคำถามก่อนเริ่มงานตาม CONTRIBUTING.md |
| 29 | 281–290 | GitHub Search | ใช้ search qualifiers เลือกโปรเจกต์และ Issue ใน Step 292 |
| 30 | 291–300 | โปรเจกต์ฝึกหัดรวมทุกอย่างเข้าด้วยกัน | Part นี้เอง |

### 300.4 แผนภาพรวมกระบวนการ Contribute แบบเต็มวงจร

```
ค้นหา Issue (Search + Label)
        │
        ▼
  จองงาน (comment ใน Issue)
        │
        ▼
  อ่าน CONTRIBUTING.md / CODE_OF_CONDUCT.md
        │
        ▼
  Fork → Clone → เพิ่ม upstream remote
        │
        ▼
  Sync main จาก upstream ก่อนเริ่ม
        │
        ▼
  สร้าง branch ตามธรรมเนียมโปรเจกต์
        │
        ▼
  แก้ไข/เพิ่มโค้ด → รัน test/lint
        │
        ▼
  Commit ตามธรรมเนียม (+ sign-off ถ้าจำเป็น)
        │
        ▼
  Push ไปที่ origin (fork ของเรา)
        │
        ▼
  เปิด PR ข้าม repo (base: upstream, head: fork)
        │
        ▼
  รอ/ตอบสนอง Code Review ────┐
        │                     │
        ▼                     │ (ถ้ามี feedback เพิ่ม)
  Maintainer Approve           │
        │                     │
        ▼                     │
      Merge ◄──────────────────┘
        │
        ▼
  Issue ปิดอัตโนมัติ + ลบ branch ทำความสะอาด
```

### 300.5 Checklist ทบทวนภาพรวมเฟส 3 ทั้งหมด (Part 16–30) ก่อนเข้าสู่เฟส 4

ก่อนไปต่อ Part 31 (เริ่มต้นเฟส 4: การทำงานเป็นทีมด้วย Git/GitHub) ให้ตรวจสอบตัวเองอย่างละเอียดตามรายการนี้ ถ้าข้อไหนยังไม่มั่นใจ แนะนำให้ย้อนกลับไปอ่าน Part ที่เกี่ยวข้องอีกครั้งก่อน:

**บัญชีและการเชื่อมต่อ**
- [ ] มีบัญชี GitHub ที่ตั้งค่า Profile และเปิด 2FA เรียบร้อยแล้ว
- [ ] ตั้งค่า SSH Key และ push/pull ผ่าน SSH ได้โดยไม่ต้องกรอก password ทุกครั้ง
- [ ] สร้าง repository ใหม่บน GitHub และเชื่อมกับเครื่อง local ได้เองโดยไม่ต้องเปิดเอกสารดู

**เอกสารและการสื่อสาร**
- [ ] เขียน README.md ที่มีโครงสร้างชัดเจนด้วย Markdown ได้เอง
- [ ] เข้าใจว่า Wiki กับ README ต่างกันอย่างไร และควรใช้เมื่อไหร่
- [ ] ใช้ GitHub Discussions ตั้งคำถามหรือเริ่มบทสนทนากับชุมชนได้

**การจัดการงาน**
- [ ] เปิด/ปิด/มอบหมาย Issue ได้คล่อง และรู้จัก keyword `Closes #xxx`
- [ ] ใช้ Label, Milestone และ Issue/PR Template ได้จริง
- [ ] เข้าใจภาพรวมการใช้ GitHub Projects บริหารงานแบบ Kanban

**Pull Request และ Code Review**
- [ ] สร้าง Pull Request ทั้งแบบภายใน repo เดียวกันและแบบข้าม fork ได้
- [ ] ให้และรับ Code Review อย่างสุภาพและเป็นมืออาชีพ
- [ ] Resolve conversation และแก้ไข PR ตาม feedback โดยไม่ต้องเปิด PR ใหม่

**Open Source Contribution**
- [ ] ใช้ GitHub Search qualifiers ค้นหา Issue ที่เหมาะกับระดับตัวเองได้
- [ ] อ่านและเข้าใจ CONTRIBUTING.md / CODE_OF_CONDUCT.md ก่อน contribute ทุกครั้ง
- [ ] ตั้งค่า Fork/Clone/Upstream remote ได้เองโดยไม่สับสนระหว่าง `origin` กับ `upstream`
- [ ] เขียน commit message และ PR description ตามธรรมเนียมของแต่ละโปรเจกต์ได้
- [ ] ผ่านกระบวนการส่ง PR อย่างน้อยหนึ่งครั้ง (ไม่ว่าจะเป็นเส้นทาง A หรือ B จาก Step 291)

**เว็บไซต์และการเผยแพร่**
- [ ] เข้าใจภาพรวมการใช้ GitHub Pages host เว็บไซต์ static ฟรี

**ภาพรวมทั้งเฟส**
- [ ] อธิบายความแตกต่างระหว่าง Git (เครื่องมือ) กับ GitHub (แพลตฟอร์ม) ให้คนอื่นฟังได้อย่างชัดเจน (ทบทวนจาก Part 01 อีกครั้ง)
- [ ] อธิบายวงจรการ contribute open source แบบเต็มรูปแบบให้คนอื่นฟังได้โดยไม่ต้องเปิดเอกสารนี้ดู

ถ้าคุณติ๊กครบทุกข้อ (หรือเกือบครบ) แปลว่าคุณพร้อมสำหรับเฟส 4 อย่างแท้จริงแล้ว — เฟส 3 คือจุดเปลี่ยนสำคัญที่พา Git จากเครื่องมือส่วนตัวไปสู่การทำงานร่วมกับผู้อื่นจริง ทักษะทั้งหมดที่ฝึกมาในเฟสนี้จะเป็นรากฐานให้กับเฟส 4 ซึ่งจะเจาะลึกเรื่อง Workflow Model มาตรฐานที่ทีมพัฒนาซอฟต์แวร์มืออาชีพทั่วโลกใช้กันจริง

---

## สรุป Part 30

ใน Part นี้เราได้นำทักษะทั้งหมดจากเฟส 3 มาใช้งานจริงในภารกิจเดียวที่ครบวงจรที่สุด:

1. วางแผนภารกิจปิดเฟส 3 และเลือกเส้นทางที่เหมาะกับสถานการณ์ของตัวเอง (Step 291)
2. ใช้เทคนิค GitHub Search ขั้นสูงเลือกโปรเจกต์และ Issue ที่เหมาะกับระดับตัวเอง (Step 292)
3. อ่าน CONTRIBUTING.md และ CODE_OF_CONDUCT.md เพื่อทำตามกติกาของแต่ละโปรเจกต์ (Step 293)
4. Fork, Clone และตั้งค่า upstream remote อย่างถูกต้องตามมาตรฐาน (Step 294)
5. สร้าง branch และลงมือแก้ไขโค้ดจริงตามที่ Issue กำหนด พร้อมรัน test/lint ก่อนเสมอ (Step 295)
6. เขียน commit message และ PR description ตามธรรมเนียมของแต่ละโปรเจกต์ (Step 296)
7. เปิด Pull Request ข้าม fork และปฏิบัติตามมารยาทระหว่างรอ Code Review (Step 297)
8. จัดการ feedback จาก maintainer และแก้ไข PR โดยไม่ต้องเปิดใหม่ (Step 298)
9. ทบทวนบทเรียนที่ได้จากการ contribute จริงผ่านคำถามเชิงลึก (Step 299)
10. สรุปภาพรวมทั้งเฟส 3 ผ่าน Cheat Sheet, Decision Table และ Checklist ครบทุกด้าน (Step 300)

Part นี้คือจุดปิดฉากของ **เฟส 3: GitHub เบื้องต้น (Part 16–30, Step 151–300)** อย่างสมบูรณ์ คุณได้เปลี่ยนจากคนที่ใช้ Git คนเดียวบนเครื่อง ไปสู่คนที่สามารถทำงานร่วมกับผู้อื่นผ่าน GitHub ได้อย่างมั่นใจ ตั้งแต่การสื่อสารผ่าน Issue การเสนอโค้ดผ่าน Pull Request การให้และรับ Code Review ไปจนถึงการเป็นส่วนหนึ่งของชุมชน Open Source ระดับโลกจริง ๆ

จากนี้ไป หลักสูตรจะพาคุณเข้าสู่ **เฟส 4: การทำงานเป็นทีมด้วย Git/GitHub (Part 31–45, Step 301–450)** ซึ่งจะเจาะลึกเรื่อง Workflow Model มาตรฐานที่ทีมพัฒนาซอฟต์แวร์มืออาชีพทั่วโลกใช้กันจริง ตั้งแต่ Centralized Workflow, Feature Branch Workflow, Git Flow, GitHub Flow, Trunk-Based Development ไปจนถึงเทคนิคขั้นสูงอย่าง Rebase, Cherry-pick และ Git Bisect

**ต่อไป:** [Part 31: Git Workflow Models: Centralized และ Feature Branch Workflow](./part-031-git-workflow-models.md)
