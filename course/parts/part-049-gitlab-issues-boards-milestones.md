# Part 49: GitLab Issues, Boards, Milestones

> **Step ในหลักสูตรนี้:** Step 481–490
> **เฟส:** 5 — เจาะลึก GitLab และ CI/CD เบื้องต้นของมัน
> **เป้าหมายของ Part นี้:** เข้าใจระบบจัดการงาน (Issue Tracking) ของ GitLab อย่างลึกซึ้ง ตั้งแต่ Issues, Labels (รวมถึง Scoped Labels ที่ exclusive กันเอง), Milestones ทั้งระดับ Project และ Group, Issue Boards แบบ Kanban ในตัว, การประเมินงานด้วย Weight/Due date/Time tracking, การใช้ Issue Templates, การเชื่อมโยง issue ด้วย Related/Linked issues, ไปจนถึงภาพรวมของ Service Desk และ Epics — เพื่อให้สามารถบริหารจัดการ backlog ของทีมบน GitLab ได้อย่างมืออาชีพ เทียบเคียงกับสิ่งที่เคยเรียนไปแล้วบน GitHub ใน Part 20, 23 และ 26

---

## สารบัญของ Part นี้

- Step 481: GitLab Issues คืออะไร เทียบกับ GitHub Issues ที่เรียนไปแล้วใน Part 20
- Step 482: Labels ใน GitLab และ Scoped Labels ที่ exclusive กันเอง (เช่น `priority::high`, `priority::low`)
- Step 483: Milestones ใน GitLab — Project Milestone กับ Group Milestone
- Step 484: GitLab Issue Boards — Kanban ในตัว (list ตาม label, assignee, milestone)
- Step 485: Weight, Due date และ Time Tracking ของ issue (`/estimate`, `/spend`)
- Step 486: Issue Templates ใน GitLab (`.gitlab/issue_templates/`)
- Step 487: Related Issues และ Linked Issues (blocks / is blocked by)
- Step 488: Service Desk เบื้องต้น — รับอีเมลแล้วกลายเป็น issue อัตโนมัติ
- Step 489: Epics เบื้องต้น — จัดกลุ่ม issue ข้าม project หลายตัว (Premium ขึ้นไป)
- Step 490: แบบฝึกหัด — จัดการ issue tracker เต็มรูปแบบด้วย Issue Board บน GitLab

---

## Step 481: GitLab Issues คืออะไร เทียบกับ GitHub Issues ที่เรียนไปแล้วใน Part 20

ใน **Part 20** เราเรียนรู้ไปแล้วว่า GitHub Issues คือฟีเจอร์ของแพลตฟอร์ม GitHub (ไม่ใช่ส่วนหนึ่งของ Git) ที่ใช้ติดตามงาน บั๊ก และไอเดียฟีเจอร์ ผูกอยู่กับแต่ละ repository เมื่อเรามาถึง GitLab สิ่งแรกที่ควรรู้คือ **แนวคิดพื้นฐานเหมือนกันทุกประการ** — Issue คือ "รายการงานที่ต้องติดตาม" ผูกกับ project หนึ่ง ๆ มีหมายเลขประจำตัว (`#42`), title, description แบบ Markdown, สถานะ Open/Closed, ผู้เปิด, ผู้รับผิดชอบ, label, milestone และ comment

แต่ GitLab ในฐานะแพลตฟอร์ม **"DevOps ครบวงจร"** ได้ต่อยอด Issue ให้มีความสามารถที่ลึกกว่า GitHub Issues แบบพื้นฐานอยู่หลายจุด และฟีเจอร์เหล่านี้คือสิ่งที่เราจะเจาะลึกกันตลอด Part นี้

### สิ่งที่เหมือนกันระหว่าง GitHub Issues และ GitLab Issues

| ความสามารถ | GitHub Issues | GitLab Issues |
|---|---|---|
| ผูกกับ repository/project | ใช่ | ใช่ |
| Title + Description แบบ Markdown | ใช่ | ใช่ |
| Label | ใช่ | ใช่ |
| Assignee | ใช่ (รองรับหลายคนเช่นกัน) | ใช่ (รองรับหลายคน — multiple assignees) |
| Milestone | ใช่ (ระดับ repo เท่านั้น) | ใช่ (มีทั้งระดับ Project และ **ระดับ Group**) |
| Closing keywords (`Fixes #12`) | ใช่ | ใช่ (`Closes #12`, `Fixes #12` เหมือนกัน) |
| Comment / Reaction | ใช่ | ใช่ |
| Board แบบ Kanban | ต้องใช้ GitHub Projects แยกต่างหาก | มีในตัว ("Issue Boards") ผูกกับ project โดยตรง |

### สิ่งที่ GitLab มีเพิ่มเติมจาก GitHub Issues แบบพื้นฐาน

1. **Scoped labels** — label แบบ `key::value` ที่ exclusive กันเองในกลุ่มเดียวกัน (เช่นมี `priority::high` ได้แค่ตัวเดียวต่อ issue) — GitHub ไม่มีกลไกนี้ในตัว ต้องอาศัยวินัยของทีมเอาเอง
2. **Weight** — ตัวเลขบอกความยากหรือขนาดงาน ใช้ประเมิน effort คล้าย "story point"
3. **Time Tracking ในตัว** — พิมพ์คำสั่ง `/estimate` และ `/spend` เพื่อบันทึกเวลาประเมินและเวลาที่ใช้จริง โดยไม่ต้องพึ่งปลั๊กอินภายนอก (ต่างจาก GitHub ที่ไม่มี time tracking ในตัวเลย ต้องใช้ third-party app)
4. **Confidential Issues** — ทำ issue ให้มองเห็นได้เฉพาะสมาชิกที่มีสิทธิ์ระดับ Reporter ขึ้นไปของ project เท่านั้น เหมาะกับรายงานช่องโหว่ความปลอดภัยหรือประเด็น sensitive
5. **Related issues / Linked items** — เชื่อมโยง issue แบบมีความสัมพันธ์เชิงตรรกะ (blocks/is blocked by) ไม่ใช่แค่การพิมพ์อ้างอิงเฉย ๆ
6. **Health status** — ธง Red/Yellow/Green บอกสถานะสุขภาพของงานเมื่อดูภาพรวมใน Epic/Board
7. **Iterations** — คล้าย milestone แต่ใช้สำหรับ timebox แบบ Sprint (มีวันเริ่ม-จบตายตัวเป็นรอบ ๆ)
8. **Service Desk** — แปลงอีเมลจากลูกค้าให้กลายเป็น issue โดยอัตโนมัติ (Step 488)
9. **Epics** — จัดกลุ่ม issue ข้าม project หลายตัวในระดับ Group (Step 489)

### ปรัชญาการออกแบบที่ต่างกัน

GitHub เลือกให้ Issues เรียบง่าย แล้วผลักฟีเจอร์ที่ซับซ้อนกว่า (board, roadmap) ไปอยู่ใน **GitHub Projects** ซึ่งเป็นเครื่องมือแยกต่างหากที่เชื่อมกับ issue ทีหลัง ส่วน GitLab เลือกฝังความสามารถระดับโปรเจกต์บริหารจัดการ (project management) เข้าไปใน object "Issue" โดยตรงตั้งแต่ต้น เพราะ GitLab วางตัวเองเป็น **"single application for the DevOps lifecycle"** ตั้งแต่ planning ไปจนถึง monitoring ในเครื่องมือเดียว

สิ่งที่ควรระวังคือ ฟีเจอร์ขั้นสูงบางตัวของ GitLab Issues (Weight, Epics, Group Issue Board, Iteration) เป็นฟีเจอร์ที่ผูกกับ **tier การใช้งาน** (Free / Premium / Ultimate) เราจะระบุไว้ชัดเจนทุกครั้งที่เจอฟีเจอร์แบบนี้ตลอด Part นี้ และแนะนำให้ตรวจสอบหน้า pricing ล่าสุดของ GitLab เสมอ เพราะ GitLab ปรับ tier ของฟีเจอร์อยู่เป็นระยะ

### ตำแหน่งของ Issues ในเมนู GitLab

เมื่อเข้าไปที่ project บน GitLab เมนูด้านซ้ายจะมีหมวด **Plan** ซึ่งรวมทุกอย่างที่เกี่ยวกับการวางแผนงานไว้ด้วยกัน:

```
Plan
├── Issues          ← รายการ issue ทั้งหมด
├── Issue boards    ← มุมมอง Kanban
├── Milestones      ← รายการ milestone
├── Iterations      ← รายการ iteration (sprint)
├── Wiki
└── Requirements
```

ต่างจาก GitHub ที่ Issues, Pull Requests และ Projects แยกเมนูกันคนละที่ GitLab จัดทุกอย่างที่เกี่ยวกับ "planning" ไว้ในหมวดเดียวเพื่อสะท้อนแนวคิด all-in-one platform

---

## Step 482: Labels ใน GitLab และ Scoped Labels ที่ exclusive กันเอง (เช่น `priority::high`, `priority::low`)

### Label พื้นฐาน — เหมือนที่เคยเรียนใน Part 26

Label ใน GitLab ทำหน้าที่เหมือนใน GitHub ทุกประการ: เป็นป้ายสี ๆ ติดกับ issue หรือ merge request เพื่อจัดหมวดหมู่ เช่น `bug`, `feature`, `documentation`, `help wanted` สร้างได้จากเมนู **Plan > Labels** โดยกำหนด:

- **Title** — ชื่อ label
- **Description** — คำอธิบาย (แสดงเป็น tooltip เวลา hover)
- **Color** — เลือกจากพาเลตสำเร็จรูป หรือใส่ hex code เอง (เช่น `#FF0000`)

### Label สองระดับ: Project label กับ Group label

จุดที่ต่างจาก GitHub Labels (ซึ่งอยู่ระดับ repo อย่างเดียว) คือ GitLab แบ่ง label เป็น 2 ระดับ:

| ระดับ | ขอบเขตการใช้งาน |
|---|---|
| **Project label** | สร้างในหน้า project ใดหน้าหนึ่ง ใช้ได้เฉพาะ issue/MR ของ project นั้น |
| **Group label** | สร้างในระดับ Group ใช้ได้กับทุก project ที่อยู่ภายใต้ group นั้น (และ subgroup) |

Group label มีประโยชน์มากเมื่อองค์กรมีหลาย project ภายใต้ทีมเดียวกัน (เช่น `frontend-repo`, `backend-repo`, `mobile-repo` อยู่ใต้ group `acme-team`) เพราะสามารถกำหนด label มาตรฐานเช่น `priority::high`, `type::bug` ให้ใช้ร่วมกันได้ทุก project โดยไม่ต้องสร้างซ้ำทีละที่ นอกจากนี้ project ยังสามารถ **"Promote to group label"** เพื่อยกระดับ project label ที่มีอยู่แล้วให้กลายเป็น group label ได้ในภายหลัง

### Scoped Labels — ฟีเจอร์เด่นที่ GitHub ไม่มี

**Scoped label** คือ label ที่ตั้งชื่อในรูปแบบ `key::value` โดยใช้เครื่องหมาย `::` (double colon) คั่นระหว่าง scope กับค่า เช่น:

```
priority::high
priority::medium
priority::low

workflow::todo
workflow::in-progress
workflow::in-review
workflow::done

type::bug
type::feature
```

**กติกาสำคัญของ Scoped Labels คือ "Exclusive กันเองภายใน scope เดียวกัน"** — พูดง่าย ๆ คือ issue หนึ่งใบสามารถมี label ที่ scope `priority::` ได้ **แค่ตัวเดียวเท่านั้น** ในเวลาเดียวกัน

ตัวอย่างเช่น ถ้า issue #101 ติด label `priority::high` อยู่แล้ว แล้วมีคนเผลอกด label `priority::low` เพิ่มเข้าไปอีก GitLab จะ **เอา `priority::high` ออกโดยอัตโนมัติ** แล้วใส่ `priority::low` แทน เพราะ GitLab ตีความว่า scope `priority` ควรมีค่าได้ค่าเดียวเสมอ (คล้ายกับปุ่ม radio button ไม่ใช่ checkbox)

```
ก่อน:  Issue #101 → [priority::high] [type::bug]
กด label priority::low เพิ่ม
หลัง:  Issue #101 → [priority::low] [type::bug]   ← priority::high ถูกแทนที่อัตโนมัติ
```

เทียบกับ label ปกติที่เป็น non-exclusive คือติดพร้อมกันได้ไม่จำกัด เช่น issue หนึ่งใบสามารถมีทั้ง `bug` และ `documentation` พร้อมกันได้แบบไม่มีปัญหา

### ทำไม Scoped Labels ถึงมีประโยชน์มาก

ก่อนมี Scoped labels ทีมที่ใช้ label ธรรมดามักเจอปัญหาแบบนี้: มีคนติด `priority-high` และ `priority-low` พร้อมกันในใบเดียวโดยไม่ได้ตั้งใจ (เพราะลืมเอา label เก่าออก) ทำให้ report/board สับสน — Scoped labels แก้ปัญหานี้ที่ **ระดับกลไกของระบบ** ไม่ต้องพึ่งวินัยของทีมอีกต่อไป

การใช้งานทั่วไปที่นิยมทำเป็น scope:

| Scope | ตัวอย่างค่า | ใช้ทำอะไร |
|---|---|---|
| `priority::` | high, medium, low | ระดับความสำคัญ |
| `severity::` | critical, major, minor | ความรุนแรงของบั๊ก |
| `workflow::` | todo, doing, review, done | สถานะงานสำหรับใช้กับ board |
| `team::` | frontend, backend, devops | ทีมที่รับผิดชอบ |
| `platform::` | ios, android, web | แพลตฟอร์มที่เกี่ยวข้อง |

Scoped label แถวเดียวกันจะแสดงผลบนหน้า UI เป็นสองสีต่อกัน (สีของ scope กับสีของ value คนละเฉดกัน) ทำให้แยกแยะได้ง่ายด้วยตา

### วิธีสร้าง Scoped Label

ขั้นตอนสร้างเหมือน label ปกติทุกอย่าง เพียงแค่ตั้งชื่อในช่อง Title ให้มีรูปแบบ `scope::value`:

1. ไปที่ **Plan > Labels > New label**
2. ตั้งชื่อ เช่น `priority::high`
3. เลือกสี (แนะนำให้ตั้งสีเดียวกันในทุก value ของ scope เดียวกัน เพื่อความสม่ำเสมอ)
4. กด **Create label**
5. ทำซ้ำสำหรับ `priority::medium`, `priority::low`

### Quick actions ที่เกี่ยวกับ label

ในช่อง comment ของ issue สามารถพิมพ์ **quick actions** (คำสั่งลัดขึ้นต้นด้วย `/`) เพื่อจัดการ label ได้ทันทีโดยไม่ต้องคลิกเมนู:

```
/label ~"priority::high"
/label ~bug ~"team::backend"
/unlabel ~"priority::low"
/relabel ~"priority::medium"
```

- `/label` — เพิ่ม label (ถ้าเป็น scoped label ที่ scope เดียวกันมีอยู่แล้ว จะแทนที่ตัวเก่าอัตโนมัติ)
- `/unlabel` — เอา label ออก
- `/relabel` — ล้าง label เดิมทั้งหมดแล้วใส่ label ใหม่ตามที่ระบุ

### Priority ของ Label (การจัดลำดับ label)

GitLab อนุญาตให้ "ปักดาว" (star) label บางตัวให้เป็น **Prioritized label** ซึ่งจะกำหนดลำดับการเรียงเมื่อ sort issue list ด้วยตัวเลือก "Label priority" — มีประโยชน์เมื่อต้องการให้ label `priority::high` ลอยขึ้นมาบนสุดของ list เสมอ

---

## Step 483: Milestones ใน GitLab — Project Milestone กับ Group Milestone

### Milestone คืออะไร (ทบทวนสั้น ๆ)

Milestone คือการจัดกลุ่ม issue และ merge request ที่มีเป้าหมายร่วมกัน เช่น "จะปล่อย release อะไร" หรือ "จะทำงานให้เสร็จภายในสัปดาห์ไหน" แนวคิดเหมือนที่เคยเรียนใน GitHub (Part 26) คือมี:

- **Title** — ชื่อ milestone เช่น `v2.0.0`, `Sprint 14`
- **Description** — รายละเอียดเป้าหมาย
- **Start date** และ **Due date** — ช่วงเวลาของ milestone
- รายการ issue/MR ที่ผูกอยู่ พร้อม progress bar แสดง % ที่เสร็จแล้ว

### จุดต่างสำคัญ: GitLab มี Milestone สองระดับ

นี่คือสิ่งที่ GitHub ไม่มี — GitHub milestone อยู่ระดับ repository เท่านั้น แต่ GitLab แบ่งเป็น:

| ระดับ | สร้างที่ไหน | ใช้กับอะไรได้บ้าง |
|---|---|---|
| **Project Milestone** | หน้า Project > Plan > Milestones | ใช้กับ issue/MR ของ project นั้นเพียงอย่างเดียว |
| **Group Milestone** | หน้า Group > Plan > Milestones | ใช้ร่วมกันได้กับทุก project ภายใต้ group นั้น (รวม subgroup) |

**Group Milestone มีประโยชน์อย่างมากในองค์กรที่แบ่งงานเป็นหลาย repository** เช่น บริษัทหนึ่งอาจมี group ชื่อ `acme` ที่มี project ย่อย `web-app`, `mobile-app`, `api-server` แยกกัน แต่ทั้งสามทีมต้องการปล่อย **Release 3.0** พร้อมกัน — การสร้าง Group Milestone ชื่อ `Release 3.0` ที่ระดับ group ทำให้ทุก project เห็น milestone เดียวกัน และสามารถดูภาพรวมความคืบหน้าของทั้งสาม project รวมกันได้ในหน้าเดียว โดยไม่ต้องสร้าง milestone ซ้ำ 3 รอบในแต่ละ project

```
Group: acme
│
├── Group Milestone: "Release 3.0" (start: 1 ต.ค. / due: 31 ต.ค.)
│
├── Project: web-app       → issue #12, #15 ผูกกับ Release 3.0
├── Project: mobile-app    → issue #7          ผูกกับ Release 3.0
└── Project: api-server    → issue #23, #24, #30 ผูกกับ Release 3.0
```

ข้อควรรู้: issue หนึ่งใบเลือกผูกได้กับ **milestone เดียวเท่านั้น** ไม่ว่าจะเป็น project milestone หรือ group milestone ก็ตาม — และเมื่อเลือก group milestone จาก dropdown ในหน้า issue ระบบจะดึงรายชื่อ milestone จากทุกระดับที่ project นั้นมองเห็นมาให้เลือก (ทั้ง milestone ของตัว project เองและของ group ที่ครอบอยู่)

### Milestone Dashboard: Burndown Chart และ Burnup Chart

เมื่อเปิดหน้า milestone ใดหน้าหนึ่งที่ตั้ง start date และ due date ไว้ครบ GitLab จะแสดงกราฟให้อัตโนมัติ:

- **Burndown chart** — แสดงจำนวนงาน (issue count หรือ weight รวม) ที่ "เหลือ" ลดลงตามเวลา เทียบกับเส้น guideline ในอุดมคติ ช่วยดูว่าทีมกำลังทำงานทันตามแผนหรือช้ากว่ากำหนด
- **Burnup chart** — แสดงงานที่ "เสร็จแล้ว" สะสมขึ้นไปเรื่อย ๆ พร้อมกับเส้น scope รวมทั้งหมด (มีประโยชน์เวลามีการเพิ่ม/ลด scope งานระหว่างทาง เพราะ burndown เพียวๆ จะมองไม่เห็นการเปลี่ยนแปลง scope)

ฟีเจอร์กราฟเหล่านี้เป็นสิ่งที่ทำให้ GitLab milestone ทรงพลังกว่า GitHub milestone แบบพื้นฐานที่มีแค่ progress bar เดียว

### วิธีสร้างและผูก Milestone

1. ไปที่ **Plan > Milestones > New milestone**
2. ใส่ title, description, start date, due date
3. กด **Create milestone**
4. ในหน้า issue ให้เลือก milestone จาก sidebar ด้านขวา หรือใช้ quick action:

```
/milestone %"v2.0.0"
/milestone %"Release 3.0"
/remove_milestone
```

เครื่องหมาย `%` ใช้อ้างอิง milestone เหมือนที่ `#` ใช้อ้างอิง issue และ `!` ใช้อ้างอิง merge request — เป็นรูปแบบการอ้างอิงมาตรฐานของ GitLab ที่ควรจำ:

| สัญลักษณ์ | อ้างอิงถึง |
|---|---|
| `#123` | Issue หมายเลข 123 |
| `!45` | Merge Request หมายเลข 45 |
| `%"v2.0.0"` | Milestone ชื่อ v2.0.0 |
| `~bug` หรือ `~"priority::high"` | Label |
| `&8` หรือ `group&8` | Epic หมายเลข 8 |
| `@username` | ผู้ใช้ |

### Milestone ที่ปิดแล้ว (Closed Milestone)

เมื่อถึงกำหนด due date หรืองานทั้งหมดเสร็จแล้ว สามารถกด **Close milestone** ได้ด้วยตนเอง — GitLab **ไม่ปิด milestone ให้อัตโนมัติ** แม้ issue ทุกใบจะถูกปิดหมดแล้วก็ตาม (ต่างจาก milestone progress ที่คำนวณอัตโนมัติ) การปิดด้วยตนเองทำให้ทีมมีจังหวะ "สรุปงาน" ก่อนที่จะเก็บ milestone เข้าคลังประวัติศาสตร์อย่างเป็นทางการ

---

## Step 484: GitLab Issue Boards — Kanban ในตัว (list ตาม label, assignee, milestone)

### Issue Board คืออะไร

**Issue Board** คือมุมมองแบบ **Kanban** ของ issue ทั้งหมดใน project (หรือ group) แสดงเป็นคอลัมน์ (เรียกว่า **list**) ให้ลาก issue ย้ายไปมาระหว่างคอลัมน์ได้ด้วยเมาส์ (drag and drop) เข้าถึงได้ที่เมนู **Plan > Issue boards**

ต่างจาก GitHub ที่ต้องไปสร้าง "Project" (Kanban board) แยกต่างหากแล้วค่อยไป add item จาก issue เข้ามา (ตามที่เรียนใน Part 23) **GitLab ผูก Board เข้ากับ Label ของ issue โดยตรง** ทำให้การย้าย issue บนบอร์ดคือการ **เปลี่ยน label ของ issue นั้นจริง ๆ** ไม่ใช่แค่ field สถานะที่แยกต่างหากเหมือน GitHub Projects

```
┌─────────────┐  ┌──────────────────┐  ┌──────────────────┐  ┌───────────┐
│    Open     │  │  workflow::todo  │  │workflow::in-review│  │   Done    │
├─────────────┤  ├──────────────────┤  ├──────────────────┤  ├───────────┤
│ #101 แก้บั๊ก│  │ #98  เพิ่มฟีเจอร์│  │ #95  รอ review    │  │ #90 เสร็จ │
│ #103 งานใหม่│  │ #99  ปรับ UI     │  │                   │  │ #91 เสร็จ │
└─────────────┘  └──────────────────┘  └──────────────────┘  └───────────┘
```

เมื่อลาก issue #98 จากคอลัมน์ `workflow::todo` ไปยังคอลัมน์ `workflow::in-review` เบื้องหลังคือ GitLab ทำการ **ลบ label `workflow::todo` ออกแล้วใส่ label `workflow::in-review` แทน** ให้กับ issue #98 โดยอัตโนมัติ — นี่คือเหตุผลที่ **Scoped labels (Step 482) เข้ากันได้ดีมากกับ Board** เพราะ exclusivity ของ scoped label ทำให้ issue อยู่ได้แค่คอลัมน์เดียวในเวลาเดียวกันเสมอ ไม่มีทางไปโผล่สองคอลัมน์พร้อมกัน

### ประเภทของ List บน Board

Board ของ GitLab ไม่ได้จำกัดแค่ list ที่อิงจาก label เท่านั้น ยังสร้าง list แบบอื่นได้ด้วย (ความพร้อมใช้งานของบาง list type ขึ้นกับ tier):

| ประเภท List | คอลัมน์แบ่งตาม | หมายเหตุ tier |
|---|---|---|
| **Label list** | Label หนึ่งตัว (นิยมใช้ scoped label) | มีในทุก tier รวม Free |
| **Assignee list** | ผู้รับผิดชอบหนึ่งคน | โดยทั่วไปต้องใช้ tier Premium ขึ้นไป |
| **Milestone list** | Milestone หนึ่งตัว | โดยทั่วไปต้องใช้ tier Premium ขึ้นไป |
| **Iteration list** | Iteration (sprint) หนึ่งรอบ | โดยทั่วไปต้องใช้ tier Premium ขึ้นไป |

> **หมายเหตุเรื่อง tier:** GitLab ปรับเปลี่ยนว่าฟีเจอร์ไหนอยู่ tier ไหนอยู่เป็นระยะตามนโยบายราคาที่เปลี่ยนไปในแต่ละปี ตัวเลขและชื่อ tier ที่กล่าวถึงในหลักสูตรนี้ (Free/Premium/Ultimate) ใช้เพื่อให้เห็นภาพลำดับความซับซ้อนของฟีเจอร์ ก่อนใช้งานจริงควรตรวจสอบหน้า pricing/feature comparison ล่าสุดของ GitLab เสมอ

### Multiple Issue Boards และ Group Issue Board

- **Multiple issue boards ต่อ project** — สร้างบอร์ดได้มากกว่าหนึ่งบอร์ดในโปรเจกต์เดียว เช่น บอร์ดหนึ่งไว้ดูภาพรวม workflow ทั่วไป อีกบอร์ดไว้เฉพาะ sprint ปัจจุบัน (ความสามารถนี้โดยทั่วไปต้องใช้ tier ที่สูงกว่า Free)
- **Group Issue Board** — บอร์ดที่สร้างในระดับ Group จะรวม issue จาก **ทุก project ภายใต้ group นั้น** มาแสดงในบอร์ดเดียว มีประโยชน์มากเมื่อทีมทำงานข้าม repository หลายตัวแต่ต้องการเห็นภาพรวมงานทั้งหมดในที่เดียว (โดยทั่วไปก็อยู่ tier ที่สูงกว่า Free เช่นกัน)

### Swimlanes — จัดกลุ่มแนวนอนตาม Epic

เมื่อเปิดใช้งาน **Swimlanes** (มุมมองแถวแนวนอนซ้อนทับบนคอลัมน์แนวตั้ง) บอร์ดจะแบ่งแถวตาม **Epic** ที่ issue สังกัดอยู่ ทำให้เห็นพร้อมกันทั้ง "แนวตั้ง = สถานะงาน" และ "แนวนอน = อยู่ภายใต้เป้าหมายใหญ่อันไหน" — ฟีเจอร์นี้ต้องใช้ทั้ง Board และ Epics ร่วมกัน (Epics เป็นฟีเจอร์ tier สูงตามที่จะกล่าวใน Step 489)

```
                Open         workflow::todo    workflow::in-review     Done
Epic: Payment  ┌─────────┐  ┌──────────────┐  ┌───────────────────┐  ┌──────┐
  Revamp       │ #101    │  │ #98          │  │                    │  │ #90  │
               └─────────┘  └──────────────┘  └───────────────────┘  └──────┘
Epic: Mobile   ┌─────────┐  ┌──────────────┐  ┌───────────────────┐  ┌──────┐
  Redesign     │         │  │ #99          │  │ #95                │  │ #91  │
               └─────────┘  └──────────────┘  └───────────────────┘  └──────┘
```

### WIP Limit (Work In Progress Limit)

ในบางบอร์ด (โดยทั่วไปต้องใช้ tier Premium ขึ้นไป) สามารถกำหนด **WIP limit** ให้กับแต่ละ list ได้ เช่น จำกัดว่าคอลัมน์ `workflow::in-review` มี issue ค้างได้ไม่เกิน 3 ใบพร้อมกัน หากเกินจะมีตัวเลขสีแดงเตือนที่หัวคอลัมน์ทันที เป็นการบังคับใช้หลักการของ **Kanban แท้ ๆ** ที่เน้นจำกัดงานระหว่างทำเพื่อลด context switching ของทีม

### Filter บนหน้า Board

เหนือบอร์ดมีแถบค้นหา/กรองที่ทรงพลัง สามารถกรองด้วย label, assignee, milestone, weight, author และอื่น ๆ พร้อมกันได้หลายเงื่อนไข เพื่อโฟกัสดูเฉพาะ issue ที่เกี่ยวข้องกับตนเองในบอร์ดที่มี issue จำนวนมาก เช่นพิมพ์ `assignee:@me milestone:%"Release 3.0"` เพื่อดูเฉพาะงานของตัวเองใน milestone นั้น

---

## Step 485: Weight, Due date และ Time Tracking ของ issue (`/estimate`, `/spend`)

Step นี้จะพาดูฟีเจอร์สามตัวที่ช่วยให้ทีม **ประเมินและติดตามปริมาณงาน** ได้แม่นยำขึ้น

### Due Date — กำหนดเส้นตายของ issue

**Due date** คือวันครบกำหนดของ issue ใบนั้นโดยเฉพาะ (แยกจาก due date ของ milestone ที่ issue สังกัดอยู่) ตั้งค่าได้จาก sidebar ของ issue หรือด้วย quick action:

```
/due 2026-10-15
/due tomorrow
/due next friday
/remove_due_date
```

GitLab รองรับการพิมพ์วันที่แบบ **natural language** อย่าง `tomorrow`, `next monday`, `in 2 weeks` ได้ด้วย ทำให้สะดวกเวลาพิมพ์คำสั่งเร็ว ๆ ในช่อง comment โดยไม่ต้องเปิด date picker

issue ที่ due date ผ่านมาแล้วแต่ยังไม่ปิด จะแสดงวันที่เป็น **สีแดง** ทั้งในหน้ารายการ issue และในหน้า Board เพื่อเตือนสายตาให้ทีมสังเกตเห็นได้ทันที ฟีเจอร์นี้เป็นฟีเจอร์ที่เปิดให้ใช้งานได้ในทุก tier รวมถึง Free

### Weight — ตัวเลขบอกขนาด/ความซับซ้อนของงาน

**Weight** คือตัวเลข (ปกติเป็นจำนวนเต็มไม่ติดลบ) ที่ใช้แทนความยากหรือปริมาณงานของ issue คล้ายกับแนวคิด **Story Point** ในโลก Agile/Scrum ตั้งค่าได้จาก sidebar หรือคำสั่ง:

```
/weight 5
/clear_weight
```

เมื่อกำหนด weight ให้ทุก issue ในบอร์ดหรือ milestone แล้ว GitLab จะช่วยคำนวณผลรวม weight ให้อัตโนมัติ เช่น หัวคอลัมน์บน Board จะโชว์ตัวเลข weight รวมของ issue ทั้งหมดในคอลัมน์นั้น หรือ milestone burndown chart ก็สามารถเลือกให้แสดงเป็น "burndown ตาม weight" แทน "burndown ตามจำนวน issue" ได้ — มีประโยชน์มากสำหรับทีมที่ใช้ estimation แบบ Fibonacci/point-based (1, 2, 3, 5, 8, 13, ...) แทนการนับจำนวน issue เฉย ๆ ซึ่งบางใบอาจใช้เวลาต่างกันมาก

> Weight เป็นฟีเจอร์ที่โดยทั่วไปต้องใช้ tier Premium ขึ้นไป (ไม่มีใน Free tier) ควรตรวจสอบ pricing ปัจจุบันก่อนวางแผนใช้งานจริงกับทีม

### Time Tracking — `/estimate` และ `/spend`

ต่างจาก Weight ที่เป็นตัวเลขนามธรรม **Time Tracking** ของ GitLab ใช้หน่วยเวลาจริง (ชั่วโมง/วัน/สัปดาห์) และ **เป็นฟีเจอร์ที่เปิดให้ใช้ได้ในทุก tier รวม Free** ทำงานผ่าน quick action สองตัวหลัก:

```
/estimate 1w 2d 3h
```

คำสั่งนี้บอกว่า "คาดว่างานนี้ต้องใช้เวลา 1 สัปดาห์ 2 วัน 3 ชั่วโมง" (ตั้งได้ครั้งเดียว ถ้าพิมพ์ซ้ำจะ**แทนที่**ค่าประมาณเดิม ไม่ใช่บวกเพิ่ม)

```
/spend 3h
/spend 1d
/spend -2h
```

คำสั่งนี้บอกว่า "ใช้เวลาไปแล้วเท่านี้" (พิมพ์ได้หลายครั้ง แต่ละครั้งจะ**บวกสะสม**เข้าไปเรื่อย ๆ) ใส่ค่าติดลบ (เช่น `/spend -2h`) เพื่อลบเวลาที่บันทึกผิดออก

สามารถระบุวันที่ย้อนหลังของการ spend ได้ด้วย เผื่อลืมบันทึกตอนทำงานจริง:

```
/spend 3h 2026-09-20
```

ล้างค่าทั้งหมดด้วย:

```
/remove_time_estimate
/remove_time_spent
```

### หน่วยเวลาที่ GitLab ใช้ (ค่าเริ่มต้น)

GitLab แปลงหน่วยเวลาแบบ "วันทำงาน" ไม่ใช่วันปฏิทินจริง ค่าเริ่มต้น (ปรับได้ที่ระดับ instance/group settings) คือ:

| หน่วย | เท่ากับ |
|---|---|
| 1 เดือน (`mo`) | 4 สัปดาห์ |
| 1 สัปดาห์ (`w`) | 5 วันทำงาน |
| 1 วัน (`d`) | 8 ชั่วโมง |
| 1 ชั่วโมง (`h`) | 60 นาที |
| 1 นาที (`m`) | — |

ดังนั้น `/estimate 1w` จะเท่ากับ 40 ชั่วโมง (5 วัน x 8 ชั่วโมง) ไม่ใช่ 168 ชั่วโมง (7 วัน x 24 ชั่วโมง) — เป็นจุดที่มือใหม่มักเข้าใจผิด ควรจำไว้ว่า Time Tracking ของ GitLab นับ **เวลาทำงาน** ไม่ใช่เวลาปฏิทิน

### การดูสรุปเวลาที่ใช้ไป

เมื่อตั้ง estimate และ spend แล้ว sidebar ของ issue จะแสดงแถบเปรียบเทียบให้เห็นทันทีว่า:

```
Time tracking
Spent: 5h / Estimated: 1d
```

พร้อมแถบสัดส่วน (progress bar) และถ้าใช้เวลาเกิน estimate แถบจะเปลี่ยนเป็นสีที่เตือนสายตา ผู้จัดการโปรเจกต์ยังสามารถดูสรุปเวลาที่ทีมใช้ไปทั้งหมดในระดับ milestone หรือ group ผ่านหน้า Analytics ได้เช่นกัน

---

## Step 486: Issue Templates ใน GitLab (`.gitlab/issue_templates/`)

### ปัญหาที่ Issue Templates แก้

เหมือนที่เคยเรียนไปแล้วใน Part 26 (GitHub Issue Templates) ปัญหาคือเวลามีคนเปิด issue ใหม่จำนวนมาก แต่ละคนเขียนรายละเอียดไม่เหมือนกัน บางคนลืมใส่ steps to reproduce บางคนลืมบอกเวอร์ชัน ทำให้ทีมพัฒนาต้องเสียเวลาถามกลับไปกลับมา **Issue Template** แก้ปัญหานี้โดยกำหนดโครงร่าง (skeleton) ของ description ล่วงหน้าให้ผู้เปิด issue กรอกตาม

### ตำแหน่งไฟล์และวิธีสร้าง

GitLab ต้องการให้ไฟล์ template เป็นไฟล์ **Markdown (`.md`)** เก็บไว้ในโฟลเดอร์พิเศษที่ root ของ repository:

```
.gitlab/
└── issue_templates/
    ├── Default.md
    ├── Bug.md
    └── Feature Request.md
```

- ชื่อไฟล์ (ไม่รวมนามสกุล `.md`) จะกลายเป็นชื่อ template ที่ปรากฏใน dropdown ตอนสร้าง issue
- ไฟล์ที่ชื่อ **`Default.md`** มีความพิเศษ: GitLab จะ **นำเนื้อหามาใส่ในช่อง description ให้อัตโนมัติทันที** ทุกครั้งที่มีคนกด "New issue" ในโปรเจกต์นั้น โดยไม่ต้องเลือก template เอง (template อื่นต้องเลือกจาก dropdown ก่อนถึงจะถูกใส่ให้)

ตัวอย่างเนื้อหา `Bug.md`:

```markdown
## สรุปปัญหา (Summary)

<!-- อธิบายสั้น ๆ ว่าเจออะไร -->

## ขั้นตอนการทำให้เกิดซ้ำ (Steps to reproduce)

1.
2.
3.

## ผลลัพธ์ที่คาดหวัง (Expected behavior)

## ผลลัพธ์ที่เกิดขึ้นจริง (Actual behavior)

## สภาพแวดล้อม (Environment)

- เวอร์ชันแอป:
- เบราว์เซอร์/OS:

/label ~bug
/label ~"priority::medium"
```

สังเกตว่าใน template สามารถใส่ **quick action** เช่น `/label ~bug` ไว้ในเนื้อหาได้เลย เมื่อผู้ใช้กดสร้าง issue โดยไม่ลบบรรทัดนี้ออก GitLab จะรัน quick action นั้นให้อัตโนมัติทันทีที่ issue ถูกสร้าง เช่นติด label `bug` และ `priority::medium` ให้ทันทีโดยผู้เปิด issue ไม่ต้องมาคลิกเพิ่มเอง — นี่คือความสามารถที่ลึกกว่า GitHub Issue Template แบบ Markdown ธรรมดา

### เทียบกับ GitHub Issue Templates

| คุณสมบัติ | GitHub | GitLab |
|---|---|---|
| ตำแหน่งไฟล์ | `.github/ISSUE_TEMPLATE/` | `.gitlab/issue_templates/` |
| รูปแบบไฟล์ | Markdown (`.md`) หรือ **YAML Forms** (`.yml` — มี field แบบ dropdown, checkbox, required field) | Markdown (`.md`) เท่านั้น ไม่มีระบบ form แบบ YAML |
| Template อัตโนมัติไม่ต้องเลือก | ต้องใช้ `config.yml` ปิดเทมเพลตว่าง หรือใช้ special naming | ไฟล์ชื่อ `Default.md` ถูกใช้อัตโนมัติเสมอ |
| ฝัง quick action ในเทมเพลต | ไม่มีแนวคิดนี้ | มี — ฝังคำสั่ง `/label`, `/assign`, `/milestone` ได้เลย |
| Merge Request / PR Template | `.github/PULL_REQUEST_TEMPLATE.md` | `.gitlab/merge_request_templates/*.md` (รูปแบบเดียวกับ issue template) |

จุดที่ GitLab ยังตามหลัง GitHub คือ **ไม่มีระบบ Issue Forms แบบ YAML** ที่ทำให้ผู้ใช้กรอกผ่าน field แบบ dropdown/checkbox ได้เหมือน GitHub (ที่เรียนไว้ใน Part 26) — GitLab ยังคงใช้ raw Markdown เป็นหลัก ซึ่งง่ายกว่าแต่ก็ยืดหยุ่นน้อยกว่าในแง่การบังคับ field ที่จำเป็น

### การเลือกใช้ Template ตอนสร้าง Issue

เมื่อกด **New issue** จะมี dropdown "Choose a template" อยู่เหนือช่อง description ให้เลือกจากชื่อไฟล์ทั้งหมดใน `.gitlab/issue_templates/` — เมื่อเลือกแล้วเนื้อหาจะแทนที่สิ่งที่พิมพ์ไว้ในช่อง description ทันที (ควรเลือก template ก่อนพิมพ์อะไรเอง เพื่อไม่ให้ข้อความหาย)

### Template ระดับ Group (Instance Template Repository)

องค์กรขนาดใหญ่ที่มีหลาย project สามารถตั้งค่า **"Instance template repository"** หรือ **group-level file templates** เพื่อให้ทุก project ในองค์กรใช้ template ชุดเดียวกันได้ โดยชี้ไปยัง project กลางที่เก็บไฟล์ `.gitlab/issue_templates/` ไว้ ลดการ copy ไฟล์ template ซ้ำไปมาในหลาย repository

---

## Step 487: Related Issues และ Linked Issues (blocks / is blocked by)

### ความแตกต่างระหว่างการ "อ้างอิง" กับการ "เชื่อมโยงอย่างเป็นทางการ"

ใน Part 20 เราเรียนไปแล้วว่าการพิมพ์ `#42` ในข้อความ comment จะสร้างลิงก์อ้างอิงไปยัง issue #42 โดยอัตโนมัติ ทั้ง GitHub และ GitLab ทำแบบนี้เหมือนกัน — แต่การอ้างอิงแบบนี้เป็นแค่ **"noise" ในข้อความ** ไม่มีความหมายเชิงโครงสร้างอะไร

GitLab มีกลไกที่จริงจังกว่านั้นเรียกว่า **Related issues / Linked items** ซึ่งสร้างความสัมพันธ์ที่ **เก็บเป็นข้อมูลถาวรในระบบ** ปรากฏเป็น section แยกต่างหากในหน้า sidebar ของ issue (ในเวอร์ชันปัจจุบันเรียกรวมว่า **"Linked items"** ซึ่งครอบคลุมทั้ง issue และประเภทงานอื่น ๆ เช่น incident — ในเอกสารรุ่นเก่าอาจยังเห็นชื่อ "Linked issues")

### ความสัมพันธ์สามแบบ

| ความสัมพันธ์ | Quick action | ความหมาย |
|---|---|---|
| **relates to** | `/relate #42` | เกี่ยวข้องกันเฉย ๆ ไม่มีนัยเรื่องลำดับก่อนหลัง |
| **blocks** | `/blocks #42` | issue ปัจจุบัน "บล็อก" #42 ไว้ (ต้องทำ issue นี้ให้เสร็จก่อน #42 ถึงจะไปต่อได้) |
| **is blocked by** | `/blocked_by #42` | issue ปัจจุบัน "ถูกบล็อก" โดย #42 (ต้องรอ #42 เสร็จก่อน) |

ตัวอย่างการใช้งาน: ถ้า issue #50 "ออกแบบ Database Schema" ต้องเสร็จก่อนถึงจะเริ่ม issue #55 "เขียน API endpoint" ได้ ให้ไปที่ issue #55 แล้วพิมพ์:

```
/blocked_by #50
```

หรือไปที่ issue #50 แล้วพิมพ์:

```
/blocks #55
```

ทั้งสองคำสั่งให้ผลลัพธ์เดียวกันคือสร้างความสัมพันธ์ "50 blocks 55" ในระบบ

### ผลกระทบของ Blocking Relationship ต่อการปิด Issue

เมื่อ issue ใบหนึ่งถูก "blocked by" issue ที่ยังไม่ปิด และมีคนพยายามกด **Close issue** GitLab จะแสดง **คำเตือน** ให้ทราบว่า issue นี้ยังมีตัวบล็อกที่ยังไม่เสร็จ พร้อมให้ยืนยันอีกครั้งก่อนปิดจริง (เป็นการเตือนเพื่อความรอบคอบ ไม่ใช่การล็อกปุ่มปิดแบบ hard block) นอกจากนี้ในหน้ารายการ issue และหน้า Board จะมี **ไอคอนรูปโซ่ (blocked icon)** ปรากฏข้าง title ของ issue ที่ถูกบล็อกอยู่ ทำให้มองเห็นสถานะได้จากภาพรวมโดยไม่ต้องเปิดเข้าไปดูทีละใบ

### เปรียบเทียบกับ GitHub

GitHub เพิ่งเริ่มมีแนวคิด "Tracked by / Tracks" (sub-issues) และการเชื่อม issue กับ Projects (v2) ในช่วงหลัง แต่ในเชิงประวัติศาสตร์ GitHub ไม่มีกลไก blocks/is-blocked-by ในตัว core Issues แบบที่ GitLab มีมานานแล้ว ทีมที่ใช้ GitHub มักต้องพึ่งการพิมพ์ข้อความ เช่น "Blocked by #42" ไว้ใน description เฉย ๆ ซึ่งไม่มีผลเชิงกลไกใด ๆ ต่างจาก GitLab ที่ความสัมพันธ์นี้ถูกเก็บเป็นข้อมูลโครงสร้างจริง ค้นหา/กรองได้ และมี UI แสดงผลชัดเจน

### ความสัมพันธ์ข้าม Project

Related/Linked issues ใน GitLab เชื่อมโยงข้าม project ได้ด้วย เพียงใช้ full reference แทนแค่เลขหมายเลข เช่น:

```
/relate group-name/project-name#42
/blocked_by other-team/other-project#7
```

ทำให้ทีมที่แยก repository กันสามารถสื่อสารความสัมพันธ์ของงานข้ามทีมได้โดยไม่ต้องรวม repository เป็นก้อนเดียว

### การลบความสัมพันธ์

ลบได้จากปุ่มกากบาทข้าง item ใน section "Linked items" ของ sidebar โดยตรง ไม่มี quick action สำหรับลบเฉพาะเจาะจง (ต้องเข้าไปกดลบผ่านหน้า UI)

---

## Step 488: Service Desk เบื้องต้น — รับอีเมลแล้วกลายเป็น issue อัตโนมัติ

### Service Desk คืออะไร

**Service Desk** คือฟีเจอร์ที่ทำให้ project หนึ่ง ๆ มี **ที่อยู่อีเมลเฉพาะ (unique email address)** ของตัวเอง เมื่อมีอีเมลส่งเข้ามาที่ที่อยู่นั้น GitLab จะ **สร้าง issue ใหม่ขึ้นมาโดยอัตโนมัติ** โดยดึงหัวข้ออีเมลมาเป็น title และเนื้อหาอีเมลมาเป็น description ให้ทันที เหมาะกับทีม Support ที่รับเรื่องร้องเรียนหรือปัญหาจากลูกค้าทางอีเมล แล้วอยากให้ทุกเรื่องกลายเป็น issue ที่ติดตามได้อย่างเป็นระบบโดยอัตโนมัติ แทนที่จะต้อง copy ข้อความจากอีเมลมาสร้าง issue ทีละอันด้วยมือ

### การทำงานโดยสรุป (ภาพรวม ไม่ลงรายละเอียดขั้นตั้งค่าเซิร์ฟเวอร์อีเมล)

```
ลูกค้าส่งอีเมล
       │
       ▼
support-project-xxxx@incoming.gitlab.com   ← ที่อยู่เฉพาะของ project นั้น
       │
       ▼
GitLab แปลงอีเมลเป็น Issue ใหม่โดยอัตโนมัติ
       │
       ├── Title = หัวข้ออีเมล
       ├── Description = เนื้อหาอีเมล
       ├── Author = บัญชีพิเศษ "Support Bot"
       └── (ค่าเริ่มต้น) เป็น Confidential Issue เพื่อไม่ให้ข้อมูลลูกค้ารั่วไหลสู่สาธารณะ
       │
       ▼
ทีม Support ตอบกลับผ่านการ comment ใน issue
       │
       ▼
GitLab ส่งข้อความ comment นั้นกลับไปเป็นอีเมลหาลูกค้าโดยอัตโนมัติ
```

จุดสำคัญคือการสื่อสารเป็น **"two-way sync"** — ทีมงานไม่จำเป็นต้องออกจากหน้า GitLab เลย เพียงตอบ comment ใน issue ตามปกติ ลูกค้าก็จะได้รับอีเมลตอบกลับโดยอัตโนมัติ และถ้าลูกค้าตอบอีเมลกลับมาอีก ข้อความนั้นก็จะถูกเติมเข้าไปเป็น comment ใหม่ใน issue เดิมต่อเนื่องกันเป็น thread เดียว

### วิธีเปิดใช้งาน (ภาพรวม)

1. ไปที่ **Settings > Monitor** (หรือ Settings ที่เกี่ยวกับ Service Desk ตามเวอร์ชัน UI) ของ project
2. เปิด toggle **Activate Service Desk**
3. GitLab จะสร้างที่อยู่อีเมลเฉพาะให้ทันที (รูปแบบทั่วไปคล้าย `<project-key>-<random-id>-issue-@incoming.gitlab.com` สำหรับ GitLab.com หรือใช้ email server ของตนเองในกรณี self-hosted)
4. เลือก project label ที่จะติดให้ทุก issue ที่เกิดจาก Service Desk อัตโนมัติ (เช่นติด label `Service Desk` ไว้เพื่อกรองแยกจาก issue ปกติได้ง่าย)
5. ปรับแต่ง **template** ของอีเมลตอบกลับอัตโนมัติ (auto-reply) และ template ของ description ได้ผ่านไฟล์ในโฟลเดอร์ `.gitlab/service_desk_templates/` ของ repository (แนวคิดคล้ายกับ issue template ใน Step 486)

### ข้อควรระวังเรื่อง Confidential Issue

โดยค่าเริ่มต้น issue ที่สร้างจาก Service Desk มักถูกตั้งเป็น **confidential** เพื่อป้องกันไม่ให้ข้อมูลส่วนตัวของลูกค้า (เช่น อีเมล, รายละเอียดปัญหาที่ sensitive) หลุดไปให้สมาชิกทั่วไปของ project เห็น มีเพียงสมาชิกที่มีสิทธิ์ระดับ Reporter ขึ้นไปเท่านั้นที่จะมองเห็น issue เหล่านี้ได้

### ข้อจำกัดที่ควรรู้ (ระดับผิวเผิน)

- ฟีเจอร์นี้ต้องอาศัยการตั้งค่าฝั่งเซิร์ฟเวอร์อีเมล (incoming email) ให้ถูกต้อง ซึ่งใน self-hosted GitLab ต้องมี admin ตั้งค่า SMTP/IMAP server เพิ่มเติม ส่วนบน GitLab.com ระบบเตรียมโครงสร้างพื้นฐานนี้ไว้ให้อยู่แล้วจึงเปิดใช้ได้ทันที
- การปรับแต่ง template ของอีเมลตอบกลับแบบละเอียด (custom branding, HTML email) มักเป็นความสามารถที่ลึกขึ้นใน tier ที่สูงกว่า แม้ความสามารถพื้นฐาน (เปิดใช้งาน + แปลงอีเมลเป็น issue) จะใช้งานได้ในระดับ tier พื้นฐานทั่วไปก็ตาม ควรตรวจสอบรายละเอียดล่าสุดกับเอกสารทางการของ GitLab เสมอ เพราะการแบ่ง tier ของฟีเจอร์ย่อยอาจเปลี่ยนแปลงได้

หลักสูตรนี้พาดูภาพรวมของ Service Desk แค่ระดับผิวเผินตามที่ระบุไว้ในเป้าหมายของ Step นี้ ส่วนรายละเอียดเชิงลึกของการตั้งค่าอีเมลระดับ infrastructure จะไม่ครอบคลุมในหลักสูตรนี้

---

## Step 489: Epics เบื้องต้น — จัดกลุ่ม issue ข้าม project หลายตัว (Premium ขึ้นไป)

### ปัญหาที่ Epics แก้: Milestone ไม่พอสำหรับเป้าหมายใหญ่ระดับหลายเดือน

Milestone (Step 483) เหมาะกับเป้าหมายระยะสั้นถึงกลาง เช่น release หนึ่งรอบ หรือ sprint หนึ่งรอบ แต่เมื่อองค์กรมีเป้าหมายเชิงกลยุทธ์ขนาดใหญ่ที่ **กินเวลาหลายเดือนและครอบคลุมหลาย project พร้อมกัน** เช่น "Redesign ระบบ Payment ทั้งหมดขององค์กร" ซึ่งอาจเกี่ยวข้องกับทั้ง `web-app`, `mobile-app`, `payment-service`, `notification-service` พร้อมกัน — การใช้ milestone อย่างเดียวจะเริ่มไม่พอ เพราะ milestone ผูกกับกรอบเวลาสั้น ๆ เป็นหลัก

**Epic** คือ object ระดับ **Group** ที่ออกแบบมาเพื่อจัดกลุ่มเป้าหมายใหญ่แบบนี้โดยเฉพาะ — Epic หนึ่งตัวสามารถ "บรรจุ" issue จากหลาย project ที่อยู่ภายใต้ group เดียวกันไว้ด้วยกันได้

```
Group: acme
│
└── Epic: "Payment System Redesign" (ระยะเวลา: Q3–Q4 ปีนี้)
    │
    ├── Issue: web-app#120        "ปรับหน้า Checkout UI"
    ├── Issue: mobile-app#45      "รองรับ Apple Pay"
    ├── Issue: payment-service#8  "เปลี่ยนไปใช้ payment gateway ใหม่"
    └── Issue: notification-service#3 "แจ้งเตือนเมื่อชำระเงินสำเร็จ"
```

### Epic เป็นฟีเจอร์ tier Premium ขึ้นไป

Epics เป็นหนึ่งในฟีเจอร์ที่ **ไม่มีใน GitLab Free** ต้องใช้ tier **Premium หรือ Ultimate** ขึ้นไป (Ultimate เพิ่มความสามารถเรื่อง Epic แบบซ้อนหลายชั้นและ roadmap ขั้นสูงกว่า) เหตุผลที่ GitLab จัดให้อยู่ tier บนคือ Epic ถือเป็นเครื่องมือระดับ "portfolio/program management" ที่มักใช้โดยผู้บริหารระดับ Product Manager หรือ Program Manager ในองค์กรขนาดกลางถึงใหญ่ ไม่ใช่ฟีเจอร์ระดับพื้นฐานที่ทีมพัฒนาเล็ก ๆ จำเป็นต้องใช้ทุกวัน

### ความสามารถหลักของ Epic

1. **สร้างที่ระดับ Group** — ไปที่ **Group > Plan > Epics > New epic** ตั้ง title, description, ช่วงวันเริ่ม/สิ้นสุด (คำนวณอัตโนมัติจาก milestone ของ issue ลูกก็ได้ หรือกำหนดตายตัวเองก็ได้)
2. **เพิ่ม issue เข้า epic** — จากหน้า issue ใช้ quick action `/epic <group>&<epic_id>` เช่น `/epic acme&5` หรือเพิ่มผ่านปุ่มในหน้า Epic โดยตรง
3. **Nested epics (Epic ซ้อน Epic)** — สร้าง epic ย่อยภายใต้ epic แม่ได้หลายชั้น (multi-level epic tree) ทำให้แตกเป้าหมายใหญ่ระดับปีลงมาเป็นเป้าหมายย่อยระดับไตรมาส แล้วค่อยแตกลงมาเป็น issue ระดับสัปดาห์ได้ในโครงสร้างเดียวกัน
4. **Roadmap view** — แสดง epic ทั้งหมดเป็น timeline แนวนอน (คล้าย Gantt chart แบบง่าย) ให้เห็นภาพว่างานใหญ่แต่ละก้อนจะเกิดขึ้นช่วงไหนของปี ซ้อนทับกันหรือไม่
5. **Health status roll-up** — เมื่อ issue ลูกแต่ละใบตั้งค่า Health status (On track / Needs attention / At risk) ไว้ Epic แม่จะสามารถสรุปภาพรวมสุขภาพของทั้งกลุ่มงานให้เห็นในหน้าเดียว
6. **Promote issue เป็น Epic** — issue ที่ค้นพบภายหลังว่าจริง ๆ แล้วมีขนาดใหญ่เกินกว่าจะเป็น issue เดียว สามารถ "เลื่อนขั้น (promote)" ให้กลายเป็น Epic ใหม่ได้จากเมนูในหน้า issue โดยไม่ต้องสร้างใหม่จากศูนย์

### Epic Board

คล้ายกับ Issue Board (Step 484) แต่ทำงานในระดับ Epic แทน — คอลัมน์แบ่งตาม label ของ epic เอง แล้วลาก epic ทั้งก้อนย้ายสถานะได้ เหมาะกับการประชุมระดับผู้บริหารที่ต้องการเห็นภาพความคืบหน้าระดับ "โครงการใหญ่" โดยไม่ต้องลงรายละเอียดถึงระดับ issue ย่อย

### เทียบกับ GitHub

GitHub ไม่มี object ที่ชื่อ "Epic" ในตัว core Issues แต่ทีมที่ใช้ GitHub มักจำลองแนวคิด epic ด้วยวิธีอื่น เช่น ใช้ label ชื่อ `epic` ติดกับ issue หลัก แล้วใช้ **Task list** (checkbox `- [ ]` ที่อ้างอิง `#number`) ในหน้า description เพื่อ list issue ย่อยที่เกี่ยวข้อง หรือใช้ GitHub Projects (v2) ซึ่งมีสนาม (field) แบบ custom ให้จัดกลุ่มงานข้าม repository ได้ในระดับหนึ่ง — แต่ก็ยังไม่ใช่ object ระดับ Group ที่ผูกความสัมพันธ์เชิงโครงสร้างเหมือน Epic ของ GitLab

### สรุปสั้น ๆ ว่าเมื่อไหร่ควรใช้อะไร

| ต้องการจัดกลุ่มระดับ | ใช้เครื่องมือ |
|---|---|
| งานที่สัมพันธ์กันสองสามใบ | Related/Linked issues (Step 487) |
| งานทั้งหมดของ release/sprint หนึ่งรอบ ใน project เดียว | Project Milestone (Step 483) |
| งานทั้งหมดของ release เดียวกัน ข้ามหลาย project ในทีมเดียวกัน | Group Milestone (Step 483) |
| เป้าหมายเชิงกลยุทธ์ขนาดใหญ่ กินเวลาหลายเดือน ข้ามหลาย project/หลายทีม | Epic (Step 489, Premium ขึ้นไป) |

---

## Step 490: แบบฝึกหัด — จัดการ issue tracker เต็มรูปแบบด้วย Issue Board บน GitLab

ถึงเวลาลงมือทำจริง! แบบฝึกหัดนี้จะรวมทุกอย่างที่เรียนมาใน Part นี้เข้าด้วยกัน โดยใช้ project ที่คุณสร้างไว้แล้วจาก Part ก่อนหน้าในเฟส 5 (หรือสร้าง project ทดลองใหม่ชื่อ `issue-board-practice` ก็ได้)

### 490.1 เตรียม Project

ถ้ายังไม่มี project สำหรับฝึก ให้สร้างใหม่บน GitLab:

```bash
# บนเครื่อง local
mkdir issue-board-practice
cd issue-board-practice
git init
echo "# Issue Board Practice" > README.md
git add README.md
git commit -m "initial commit"
git remote add origin <URL ของ project ที่สร้างบน GitLab>
git push -u origin main
```

### 490.2 สร้าง Scoped Labels สำหรับ Workflow

ไปที่ **Plan > Labels > New label** แล้วสร้าง scoped labels ต่อไปนี้ให้ครบ (ตั้งสีให้แต่ละกลุ่ม scope ใช้โทนเดียวกันเพื่อความสม่ำเสมอ):

```
workflow::todo
workflow::doing
workflow::review
workflow::done

priority::high
priority::medium
priority::low
```

ทดสอบ exclusivity: สร้าง issue ทดลองสักใบ ติด `priority::high` แล้วลองติด `priority::low` เพิ่ม สังเกตว่า `priority::high` หายไปโดยอัตโนมัติหรือไม่ (ถ้าใช่ แปลว่าเข้าใจ scoped label ถูกต้องแล้ว)

### 490.3 สร้าง Milestone

ไปที่ **Plan > Milestones > New milestone** สร้าง milestone ชื่อ `Sprint 1` กำหนด start date เป็นวันนี้ และ due date เป็น 2 สัปดาห์ถัดไป

### 490.4 สร้าง Issue อย่างน้อย 6 ใบ

สร้าง issue อย่างน้อย 6 ใบ ครอบคลุมสถานการณ์ให้หลากหลาย เช่น:

| # | Title | Label ที่ควรติด | Milestone |
|---|---|---|---|
| 1 | ออกแบบหน้า Login | `workflow::todo`, `priority::high` | Sprint 1 |
| 2 | เขียน API สำหรับ Register | `workflow::todo`, `priority::medium` | Sprint 1 |
| 3 | เชื่อมต่อฐานข้อมูล | `workflow::doing`, `priority::high` | Sprint 1 |
| 4 | เขียน Unit Test สำหรับ Login | `workflow::review`, `priority::medium` | Sprint 1 |
| 5 | Setup CI Pipeline เบื้องต้น | `workflow::done`, `priority::low` | Sprint 1 |
| 6 | แก้บั๊ก Validation ฟอร์ม | `workflow::todo`, `priority::high` | Sprint 1 |

ลองใช้ quick action ตอนสร้าง comment แรกของ issue เพื่อฝึกความเร็ว เช่นพิมพ์ในช่อง description ของ issue #3:

```
/label ~"workflow::doing" ~"priority::high"
/milestone %"Sprint 1"
/estimate 1d
/spend 3h
/due 3 days from now
```

### 490.5 สร้างความสัมพันธ์ Blocking อย่างน้อย 1 คู่

เลือก issue สองใบที่สมเหตุสมผลว่างานหนึ่งต้องรอให้อีกงานเสร็จก่อน เช่น issue #2 "เขียน API สำหรับ Register" ควรรอ issue #3 "เชื่อมต่อฐานข้อมูล" ให้เสร็จก่อน ไปที่ issue #2 แล้วพิมพ์:

```
/blocked_by #3
```

ตรวจสอบว่ามีไอคอน blocked ปรากฏข้าง issue #2 ในหน้ารายการ issue หรือไม่

### 490.6 สร้าง Issue Template

สร้างไฟล์ `.gitlab/issue_templates/Bug.md` ใน repository (สร้างผ่านหน้าเว็บ GitLab โดยตรงก็ได้ ผ่านปุ่ม "New file" หรือ clone ลงมาแก้ที่เครื่องแล้ว push กลับ) ให้มีเนื้อหาอย่างน้อยประกอบด้วยหัวข้อ Summary, Steps to reproduce, Expected/Actual behavior และฝัง quick action `/label ~bug` ไว้ในไฟล์ด้วย จากนั้นลองกด **New issue** อีกครั้งเพื่อดูว่า dropdown "Choose a template" มีตัวเลือก `Bug` ปรากฏขึ้นมาหรือไม่

### 490.7 สร้าง Issue Board อย่างน้อย 3 คอลัมน์

ไปที่ **Plan > Issue boards > Create board** (หรือใช้บอร์ด default ที่มีมาให้) แล้วปรับให้มีคอลัมน์อย่างน้อย 3 คอลัมน์นอกเหนือจาก Open/Closed เริ่มต้น โดยผูกแต่ละคอลัมน์เข้ากับ scoped label ที่สร้างไว้ในข้อ 490.2:

```
┌─────────────┐  ┌──────────────────┐  ┌───────────────────┐  ┌──────────────┐
│    Open     │  │  workflow::todo  │  │  workflow::doing   │  │workflow::review│
├─────────────┤  ├──────────────────┤  ├───────────────────┤  ├──────────────┤
│  (ว่าง)     │  │ #1 ออกแบบ Login │  │ #3 เชื่อมต่อฐานข้อมูล│  │ #4 Unit Test │
│             │  │ #2 API Register  │  │                     │  │              │
│             │  │ #6 แก้บั๊ก Form  │  │                     │  │              │
└─────────────┘  └──────────────────┘  └───────────────────┘  └──────────────┘
```

(ถ้าต้องการคอลัมน์ `workflow::done` เพิ่มด้วยก็ทำได้เช่นกัน — ยิ่งครบยิ่งดี)

ลองลาก issue #1 จากคอลัมน์ `workflow::todo` ไปยัง `workflow::doing` ด้วยเมาส์ แล้วเปิดหน้า issue #1 ตรวจสอบว่า label เปลี่ยนจาก `workflow::todo` เป็น `workflow::doing` จริงหรือไม่ (นี่คือการยืนยันว่าเข้าใจกลไกเบื้องหลังของ Board ถูกต้อง)

### 490.8 ทดลอง Filter บนหน้า Board

ใช้แถบค้นหาเหนือบอร์ด กรองดูเฉพาะ issue ที่มี `priority::high` ทั้งหมด สังเกตว่าทุกคอลัมน์กรองพร้อมกันหรือไม่ แล้วลองกรองด้วย assignee ของตัวเองเพิ่มเข้าไปอีกเงื่อนไข

### 490.9 Checklist ก่อนไป Part 50

ก่อนไปต่อ Part 50 ให้ตรวจสอบว่าคุณ:

- [ ] เข้าใจความแตกต่างระหว่าง GitHub Issues กับ GitLab Issues อย่างน้อย 5 จุด
- [ ] สร้าง Scoped label ได้ และอธิบายกลไก exclusivity ได้ด้วยคำพูดตัวเอง
- [ ] แยกความแตกต่างระหว่าง Project Milestone กับ Group Milestone ได้
- [ ] สร้าง Issue Board ที่มีอย่างน้อย 3 คอลัมน์ และลาก issue ย้ายคอลัมน์ได้จริง พร้อมยืนยันว่า label เปลี่ยนตามจริง
- [ ] ใช้ quick action `/estimate`, `/spend`, `/due`, `/weight` ได้อย่างน้อยคนละ 1 ครั้ง
- [ ] สร้างไฟล์ issue template ใน `.gitlab/issue_templates/` และใช้งานได้จริง
- [ ] สร้างความสัมพันธ์ `/blocks` หรือ `/blocked_by` ระหว่าง issue สองใบได้สำเร็จ
- [ ] อธิบายได้ว่า Service Desk ทำงานอย่างไรในภาพรวม แม้จะยังไม่ได้ตั้งค่าจริงระดับ production
- [ ] อธิบายได้ว่า Epic ต่างจาก Milestone อย่างไร และทำไมถึงเป็นฟีเจอร์ tier สูง

---

## สรุป Part 49

ใน Part นี้เราได้เจาะลึกระบบจัดการงานของ GitLab ซึ่งเป็นหนึ่งในจุดแข็งที่ทำให้ GitLab ถูกเลือกใช้ในองค์กรที่ต้องการเครื่องมือ "all-in-one" ครบวงจร:

1. **GitLab Issues** มีรากฐานเหมือน GitHub Issues ที่เรียนใน Part 20 แต่ต่อยอดด้วย Weight, Time Tracking ในตัว, Confidential Issues, Related/Linked Issues, Health Status, Iterations, Service Desk และ Epics
2. **Labels** มีทั้งระดับ Project และ Group และมี **Scoped Labels** (`key::value`) ที่ exclusive กันเองภายใน scope เดียวกัน ช่วยแก้ปัญหา label ขัดแย้งกันที่ label ธรรมดาแก้ไม่ได้
3. **Milestones** มีทั้งระดับ Project และ **Group Milestone** ที่ใช้ร่วมกันได้ข้ามหลาย project พร้อมกราฟ Burndown/Burnup ในตัว
4. **Issue Boards** เป็น Kanban ที่ผูกกับ label โดยตรง การลาก issue ย้ายคอลัมน์คือการเปลี่ยน label จริง ๆ รองรับ list แบบ label/assignee/milestone/iteration และ Swimlanes ตาม Epic
5. **Weight, Due date, Time Tracking** ช่วยประเมินและติดตามปริมาณงานด้วยตัวเลขและเวลาจริงผ่าน quick actions `/weight`, `/due`, `/estimate`, `/spend`
6. **Issue Templates** เก็บไว้ที่ `.gitlab/issue_templates/` รองรับการฝัง quick action ไว้ในเทมเพลตได้เลย และไฟล์ `Default.md` จะถูกใช้อัตโนมัติเสมอ
7. **Related/Linked Issues** สร้างความสัมพันธ์เชิงโครงสร้างจริง (`relates_to`, `blocks`, `is_blocked_by`) ต่างจากการพิมพ์ `#number` เฉย ๆ ที่เป็นแค่การอ้างอิงในข้อความ
8. **Service Desk** แปลงอีเมลจากลูกค้าให้กลายเป็น issue โดยอัตโนมัติ พร้อม two-way sync ระหว่าง comment กับอีเมล
9. **Epics** จัดกลุ่ม issue ข้าม project หลายตัวในระดับ Group เหมาะกับเป้าหมายเชิงกลยุทธ์ระยะยาว เป็นฟีเจอร์ tier Premium ขึ้นไป
10. ปิดท้ายด้วยแบบฝึกหัดที่รวมทุกอย่างเข้าด้วยกัน — สร้าง label, milestone, issue, template, blocking relationship และ Issue Board ที่มีอย่างน้อย 3 คอลัมน์ด้วยตัวเอง

ตอนนี้คุณมีพื้นฐานที่แข็งแรงพอสำหรับการบริหารจัดการ backlog ของทีมบน GitLab แล้ว ขั้นต่อไปเราจะเปลี่ยนโฟกัสจาก "การวางแผนงาน (Plan)" ไปสู่ "การสร้างระบบอัตโนมัติ (Build/CI)" ซึ่งเป็นอีกหนึ่งจุดแข็งที่โดดเด่นที่สุดของ GitLab

**ต่อไป:** [Part 50: GitLab CI/CD เบื้องต้น: .gitlab-ci.yml แรกของคุณ](./part-050-gitlab-cicd-เบื้องต้น.md)
