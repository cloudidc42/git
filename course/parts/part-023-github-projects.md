# Part 23: GitHub Projects: บอร์ดจัดการงานแบบ Kanban

> **Step ในหลักสูตรนี้:** Step 221–230
> **เฟส:** 3 — GitHub เบื้องต้น
> **เป้าหมายของ Part นี้:** เข้าใจ GitHub Projects (v2) อย่างลึกซึ้ง ตั้งแต่แนวคิดพื้นฐาน การสร้างบอร์ด การเพิ่ม Issue/PR จากหลาย Repository เข้าบอร์ดเดียว การสร้างและใช้งาน Custom Fields การตั้ง Automation ให้บอร์ดขยับเองเมื่อสถานะงานเปลี่ยน การใช้งาน Board View แบบ Kanban และ Roadmap View แบบ Timeline การ Filter/Group ข้อมูล ไปจนถึงการแชร์ Project ระดับ Organization และลงมือสร้างบอร์ดจำลองบริหารโปรเจกต์จริงครบวงจร

---

## สารบัญของ Part นี้

- Step 221: GitHub Projects คืออะไร (เวอร์ชัน v2 ปัจจุบัน) ต่างจาก Issues อย่างไร
- Step 222: สร้าง Project board ใหม่ และ view ต่างๆ ที่มี (Table, Board, Roadmap)
- Step 223: เพิ่ม Issue/PR เข้า Project (จากหลาย repo ได้ด้วย)
- Step 224: Custom fields (status, priority, estimate, iteration) การสร้างและใช้งาน
- Step 225: Automation ใน Projects (workflow เช่น auto-move card เป็น Done เมื่อ PR merge)
- Step 226: Board view แบบ Kanban — คอลัมน์ To do, In Progress, Done การลาก card
- Step 227: Roadmap view — timeline planning สำหรับวางแผนระยะยาว
- Step 228: Filtering และ grouping ใน Project (group by status, assignee, label)
- Step 229: การแชร์ Project ระหว่างหลาย repo (Organization-level project)
- Step 230: แบบฝึกหัด — สร้าง Kanban board บริหารงานโปรเจกต์จำลองครบวงจร (อย่างน้อย 10 การ์ด กระจาย 3 คอลัมน์)

---

## Step 221: GitHub Projects คืออะไร (เวอร์ชัน v2 ปัจจุบัน) ต่างจาก Issues อย่างไร

ใน Part 20 เราได้เรียนรู้เรื่อง **GitHub Issues** ไปแล้วว่ามันคือระบบติดตามงานและบั๊กแบบ "รายการเดี่ยว ๆ" — แต่ละ Issue คือหนึ่งงาน หนึ่งบั๊ก หรือหนึ่งฟีเจอร์ที่ต้องทำ ปัญหาคือเมื่อมี Issue เยอะขึ้นเรื่อย ๆ (หลักสิบหรือหลักร้อย) การไล่ดูทีละอันในรูปแบบลิสต์อย่างเดียวไม่เพียงพอที่จะ **มองภาพรวมของทั้งโปรเจกต์** ว่างานไหนกำลังทำอยู่ งานไหนเสร็จแล้ว งานไหนยังไม่เริ่ม และใครกำลังทำอะไรอยู่

**GitHub Projects** คือเลเยอร์การจัดการงาน (project management layer) ที่วางอยู่ *เหนือ* Issues และ Pull Requests โดยไม่ได้แทนที่มัน แต่ทำหน้าที่เป็น **"กระดานควบคุม" (control board)** ที่ดึงเอา Issue/PR จากที่ต่าง ๆ มาแสดงผลในมุมมองที่หลากหลาย พร้อมเพิ่มข้อมูลเชิงการจัดการ (metadata) ที่ Issue เพียว ๆ ไม่มี เช่น สถานะความคืบหน้า ลำดับความสำคัญ ระยะเวลาที่คาดว่าจะใช้ (estimate) และรอบการทำงาน (iteration/sprint)

### ทำไมต้องมี "v2" — Projects แบบเก่ากับแบบใหม่ต่างกันอย่างไร

GitHub เคยมีฟีเจอร์ชื่อ "Projects" มาก่อนแล้ว (ปัจจุบันเรียกย้อนหลังว่า **Projects (classic)**) ซึ่งเป็นบอร์ดแบบง่าย ผูกติดกับ repository เดียวเท่านั้น และมี field แบบตายตัว ในปี 2022 GitHub ได้เปิดตัว **Projects v2** ซึ่งเขียนขึ้นใหม่ทั้งหมดโดยมีสถาปัตยกรรมต่างไปอย่างสิ้นเชิง และปัจจุบัน (ปี 2026) **Projects (classic) ถูกปิดใช้งานถาวรแล้ว** — GitHub ประกาศ deprecate Projects แบบเก่าไปตั้งแต่ปี 2024 และปิดการใช้งานทั้งหมดในปี 2025 ดังนั้นเมื่อพูดถึง "GitHub Projects" ในปัจจุบัน หมายถึง **Projects v2** เพียงอย่างเดียว ซึ่งเป็นสิ่งที่หลักสูตรนี้จะสอนทั้งหมด

ความแตกต่างสำคัญของ Projects v2 เมื่อเทียบกับของเก่า:

| คุณสมบัติ | Projects (classic) | Projects v2 |
|---|---|---|
| ขอบเขตการใช้งาน | ผูกกับ repo เดียว หรือ org แบบจำกัด | ผูกกับ **user หรือ organization** ดึง Issue/PR จากกี่ repo ก็ได้ |
| Custom Fields | ไม่มี มีแค่คอลัมน์ตายตัว | มี Custom Fields หลายชนิด (text, number, date, single select, iteration) |
| Views | มีแค่ Board เดียว | มีหลาย View: Table, Board (Kanban), Roadmap และสร้างเพิ่มได้เอง |
| Automation | จำกัดมาก (built-in workflow น้อย) | Workflow แบบยืดหยุ่น + เชื่อมกับ GitHub Actions และ GraphQL API ได้เต็มรูปแบบ |
| การเก็บข้อมูล | ผูกกับ card แบบง่าย | เป็นเหมือน "spreadsheet ที่มีชีวิต" เชื่อมกับ Issue/PR จริงแบบ real-time |
| Roadmap/Timeline | ไม่มี | มี Roadmap View ในตัว |

### Projects ต่างจาก Issues อย่างไร (ภาพให้เห็นชัด)

จุดที่มือใหม่สับสนบ่อยที่สุดคือ "แล้ว Issues กับ Projects ต่างกันตรงไหน ในเมื่อทั้งคู่ก็ใช้ติดตามงาน"

คำตอบคือ **Issues คือ "หน่วยของงาน" (the unit of work) ส่วน Projects คือ "มุมมองการบริหารจัดการงานเหล่านั้น" (the management view)**

ลองเปรียบเทียบกับการทำงานในออฟฟิศ:

- **Issue** = โพสต์อิท (sticky note) หนึ่งใบ ที่เขียนว่า "แก้บั๊กปุ่มล็อกอินไม่ทำงานบน Safari"
- **Project** = กระดานไวท์บอร์ดที่แปะโพสต์อิทเหล่านั้นเรียงเป็นคอลัมน์ To Do / Doing / Done พร้อมป้ายสีบอกความสำคัญ

ข้อสำคัญคือ **Issue หนึ่งใบสามารถถูกเพิ่มเข้าไปอยู่ใน Project ได้หลายบอร์ดพร้อมกัน** เช่น Issue เดียวกันอาจอยู่ใน "Sprint Board" ของทีม Backend และอยู่ใน "Roadmap Q3" ของทีมผู้บริหารพร้อมกันได้ เพราะ Project เป็นเพียง "การอ้างอิง" ไปยัง Issue/PR ตัวจริง ไม่ใช่การคัดลอกข้อมูล — เมื่อ Issue ถูกปิดหรือแก้ไขที่ไหนก็ตาม การเปลี่ยนแปลงจะสะท้อนไปยังทุก Project ที่อ้างอิงถึงมันโดยอัตโนมัติ

### Projects v2 ทำงานบนพื้นฐานอะไร

ในเชิงเทคนิค Projects v2 เป็นเหมือน **"ตารางข้อมูล (table) ที่มี column แบบ custom"** ซึ่งแต่ละแถว (row) คือ Issue, Pull Request หรือแม้กระทั่ง **Draft Issue** (รายการที่ยังไม่ผูกกับ repo ใด ๆ ก็ได้ — ใช้สำหรับบันทึกไอเดียคร่าว ๆ ก่อนแตกเป็น Issue จริง) แต่ละคอลัมน์ในตารางนี้คือ Field ซึ่งอาจเป็น field มาตรฐาน (title, assignees, labels) หรือ Custom Field ที่คุณสร้างเอง (status, priority, estimate ฯลฯ)

จากตารางข้อมูลนี้ Projects v2 จะสร้าง **View** ที่หลากหลายให้เลือกดูข้อมูลชุดเดียวกันในรูปแบบต่างกัน — นี่คือสิ่งที่เราจะเรียนรู้ใน Step ถัดไป

### สรุป Step 221

- GitHub Projects (v2) คือเลเยอร์บริหารจัดการงานที่วางอยู่เหนือ Issues/PR ไม่ใช่สิ่งเดียวกัน
- Projects (classic) ถูกยกเลิกไปแล้ว ปัจจุบันมีแต่ v2
- Project ทำงานเหมือนตารางข้อมูลที่มี Field แบบ custom ได้ และแสดงผลได้หลาย View
- Issue หนึ่งใบสามารถอยู่ในหลาย Project พร้อมกันได้ โดยข้อมูลจะ sync กันแบบ real-time

---

## Step 222: สร้าง Project board ใหม่ และ view ต่างๆ ที่มี (Table, Board, Roadmap)

มาลงมือสร้าง Project แรกของคุณกัน

### วิธีสร้าง Project ใหม่

มีอยู่ 3 จุดหลักที่คุณสามารถสร้าง Project ใหม่ได้:

**1. จากหน้า Profile ส่วนตัว (สร้างเป็น User-level Project)**

1. ไปที่โปรไฟล์ของคุณ แล้วคลิกแท็บ **Projects**
2. คลิกปุ่ม **New project**
3. เลือก template เริ่มต้น (GitHub มี template สำเร็จรูปให้ เช่น "Board", "Team planning", "Feature", "Bug tracker" หรือจะเริ่มจาก "Table" เปล่า ๆ ก็ได้)
4. ตั้งชื่อ Project เช่น "My Personal Roadmap"
5. คลิก **Create project**

**2. จากหน้า Repository**

1. เข้าไปที่ repository ที่ต้องการ
2. คลิกแท็บ **Projects** (อยู่แถวเดียวกับ Issues, Pull requests)
3. คลิก **Link a project** เพื่อผูก Project ที่มีอยู่แล้วเข้ากับ repo นี้ หรือคลิก **New project** เพื่อสร้างใหม่

**3. จากหน้า Organization (สำหรับ Project ระดับองค์กร)**

1. ไปที่หน้า Organization → แท็บ **Projects**
2. คลิก **New project**

จุดสำคัญคือ **Project ที่สร้างจาก Organization หรือจาก Profile จะไม่ได้ผูกติดกับ repo ใด repo หนึ่งโดยเฉพาะ** — คุณสามารถดึง Issue/PR จากกี่ repo ก็ได้เข้ามาในบอร์ดเดียว ในขณะที่การสร้างผ่านหน้า repo เป็นเพียงทางลัดที่จะ **link** repo นั้นเข้ากับ project อัตโนมัติ แต่ตัว Project เองยังคงเป็น entity อิสระเสมอ (Projects v2 ไม่มีแนวคิด "ผูกติดกับ repo เดียวตายตัว" อีกต่อไปแล้ว)

### View คืออะไร

เมื่อสร้าง Project เสร็จ คุณจะเห็นแท็บเล็ก ๆ ด้านบนของบอร์ด นี่คือ **Views** — วิธีต่าง ๆ ในการแสดงข้อมูลชุดเดียวกัน โดย default ทุก Project ใหม่จะมี View ชื่อ "View 1" เป็น Table view ให้เริ่มต้น

Projects v2 รองรับ View หลักอยู่ 3 แบบ:

#### 1. Table View

หน้าตาเหมือนสเปรดชีต (คล้าย Excel/Google Sheets) แสดงข้อมูลเป็นแถว-คอลัมน์ เหมาะสำหรับ:

- ดูข้อมูลจำนวนมากพร้อมกันแบบละเอียด
- แก้ไข field หลายรายการพร้อมกันแบบรวดเร็ว (bulk edit)
- Sort/Filter ข้อมูลแบบตารางทั่วไป
- Export ข้อมูลออกไปทำรายงาน

#### 2. Board View

หน้าตาเป็นบอร์ดแบบ **Kanban** มีคอลัมน์แนวตั้งแบ่งตามค่าของ field ใดฟิลด์หนึ่ง (ปกติคือ field "Status") แต่ละ Issue/PR จะกลายเป็น "การ์ด" ที่ลากย้ายไปมาระหว่างคอลัมน์ได้ เหมาะสำหรับ:

- ดูภาพรวมความคืบหน้าของงานทั้งหมดแบบรวดเร็ว
- ใช้ในการประชุม stand-up ประจำวัน
- บริหารจัดการ Sprint หรือ workflow ของทีม

เราจะลงลึกเรื่อง Board View ใน Step 226

#### 3. Roadmap View

หน้าตาเป็น **timeline แนวนอน** แสดง Issue/PR เรียงตามช่วงเวลา (start date → target date/due date) เหมาะสำหรับ:

- วางแผนระยะยาว (quarterly planning, release planning)
- ดูว่างานไหนทับซ้อนช่วงเวลากันบ้าง
- นำเสนอแผนงานให้ผู้บริหารหรือลูกค้าดูภาพรวม

เราจะลงลึกเรื่อง Roadmap View ใน Step 227

### การสร้าง View ใหม่และปรับแต่ง

คุณสามารถสร้าง View เพิ่มได้ไม่จำกัดจำนวนภายใน Project เดียว โดยแต่ละ View จะมีการตั้งค่า filter, sort, group และ field ที่แสดงผลเป็นของตัวเอง (ไม่กระทบ View อื่น) วิธีสร้าง:

1. คลิกเครื่องหมาย **+** ข้าง ๆ แท็บ View ที่มีอยู่
2. เลือกประเภท View (Table, Board หรือ Roadmap)
3. ตั้งชื่อ View เช่น "Sprint Board", "Bug List", "Q3 Roadmap"

ตัวอย่างการใช้งานจริง: ทีมหนึ่งอาจมี Project เดียวที่มี 4 Views:

- **"All Items"** — Table view แสดงทุกอย่างแบบละเอียด
- **"Sprint Board"** — Board view กรองเฉพาะ iteration ปัจจุบัน
- **"By Assignee"** — Board view จัดกลุ่มตามผู้รับผิดชอบ
- **"Release Roadmap"** — Roadmap view วางแผนตาม milestone

### สรุป Step 222

- สร้าง Project ได้จาก Profile, Organization หรือ Repository
- Project ระดับ Profile/Organization ไม่ผูกกับ repo เดียว ดึงข้อมูลข้าม repo ได้
- View มี 3 แบบหลัก: Table (สเปรดชีต), Board (Kanban), Roadmap (timeline)
- สร้าง View เพิ่มได้ไม่จำกัด แต่ละ View ตั้งค่า filter/sort/group แยกกันได้อิสระ

---

## Step 223: เพิ่ม Issue/PR เข้า Project (จากหลาย repo ได้ด้วย)

Project ที่สร้างใหม่จะว่างเปล่า ขั้นตอนถัดไปคือการดึง Issue และ Pull Request เข้ามา

### วิธีที่ 1: เพิ่มจากภายในหน้า Project โดยตรง

1. เปิด Project ที่ต้องการ ไปที่ View แบบ Table
2. ที่แถวล่างสุดของตาราง จะมีช่องให้พิมพ์ **"+ Add item"**
3. พิมพ์ชื่อเรื่องแล้วกด Enter — สิ่งนี้จะกลายเป็น **Draft Issue** (ยังไม่ผูกกับ repo ใด ๆ)
4. หรือพิมพ์ `#` ตามด้วยชื่อ repo และเลขที่ issue/PR เพื่อค้นหาและเชื่อมกับ Issue/PR ที่มีอยู่แล้ว เช่น พิมพ์ `myorg/frontend#42` เพื่อดึง Issue #42 ของ repo `frontend` เข้ามา

### วิธีที่ 2: เพิ่ม repo ทั้งหมดเข้า Project (Auto-add)

หากต้องการให้ Issue/PR ใหม่ทุกอันจาก repo หนึ่ง ๆ ถูกดึงเข้า Project โดยอัตโนมัติ (ไม่ต้องเพิ่มมือทีละอัน) ให้ตั้งค่า **Workflow: Auto-add to project**:

1. ไปที่เมนู **... (More options)** มุมขวาบนของ Project → **Workflows**
2. เลือก workflow **Auto-add to project**
3. คลิก **Edit** เพื่อตั้งเงื่อนไข เช่น "เพิ่ม Issue ทุกอันที่ถูกเปิดใหม่ใน repo `frontend`" หรือ "เพิ่มเฉพาะ Issue ที่มี label `bug`"
4. เปิดใช้งาน (Enable) workflow

นี่คือจุดสำคัญมากของ Projects v2: **workflow นี้รองรับการเลือก repo ได้หลายตัวพร้อมกัน** ซึ่งหมายความว่า Project เดียวสามารถดึง Issue จาก repo `frontend`, `backend`, `mobile-app` และ `infra` เข้ามารวมกันในบอร์ดเดียวได้ทันที — นี่คือความสามารถที่ Projects (classic) ไม่มีเลย และเป็นเหตุผลหลักที่องค์กรจำนวนมากย้ายมาใช้ v2

### วิธีที่ 3: เพิ่มโดยตรงจากหน้า Issue หรือ Pull Request

เวลาเปิดดู Issue หรือ PR ใด ๆ จะมี sidebar ด้านขวาแสดงหัวข้อ **Projects** — คลิกที่นั่นแล้วเลือก Project ที่ต้องการเพิ่ม Issue/PR นี้เข้าไป วิธีนี้สะดวกมากเวลาคุณกำลังทำงานอยู่ในหน้า Issue อยู่แล้วและนึกขึ้นได้ว่าอยากให้มันปรากฏในบอร์ดใดบอร์ดหนึ่งด้วย

### วิธีที่ 4: Bulk-add จากหน้า Issues list

จากหน้ารายการ Issues ของ repo คุณสามารถเลือก (checkbox) หลาย Issue พร้อมกัน แล้วใช้เมนู **Add to project** เพื่อเพิ่มเข้า Project ทีเดียวหลายรายการ วิธีนี้เหมาะมากตอนตั้งบอร์ดครั้งแรกที่ต้องดึง Issue เก่าจำนวนมากเข้ามา

### Draft Issue คืออะไร ใช้ตอนไหน

Draft Issue คือรายการที่มีแค่ชื่อเรื่องและรายละเอียด แต่ **ไม่ได้ผูกกับ repository ใดเลย** เหมาะสำหรับ:

- บันทึกไอเดียเร็ว ๆ ระหว่างประชุม โดยยังไม่ต้องตัดสินใจว่าจะไปอยู่ repo ไหน
- งานที่เป็น task บริหารทั่วไป ไม่ใช่โค้ด เช่น "จัดประชุม kickoff กับลูกค้า"

เมื่อพร้อมแล้ว คุณสามารถ **แปลง Draft Issue เป็น Issue จริง** ได้ทุกเมื่อ โดยคลิกที่การ์ด แล้วเลือก **Convert to issue** จากนั้นเลือก repository ปลายทางที่ต้องการให้ Issue นี้ไปสังกัด

### ลบ Issue/PR ออกจาก Project

การ "ลบออกจาก Project" ไม่เหมือนกับการลบ Issue จริง — คุณแค่คลิกขวาที่การ์ด แล้วเลือก **Remove from project** ตัว Issue ตัวจริงจะยังอยู่ใน repo เหมือนเดิม เพียงแต่จะไม่ปรากฏในบอร์ดนี้อีกต่อไป

### สรุป Step 223

- เพิ่ม Issue/PR เข้า Project ได้หลายวิธี: พิมพ์ตรง, bulk-add, จากหน้า Issue, หรือ auto-add ผ่าน workflow
- Auto-add workflow รองรับหลาย repo พร้อมกัน — นี่คือจุดแข็งหลักของ Projects v2
- Draft Issue ใช้บันทึกไอเดียที่ยังไม่ผูก repo และแปลงเป็น Issue จริงได้ภายหลัง
- การลบออกจาก Project ไม่กระทบ Issue ตัวจริงใน repo

---

## Step 224: Custom fields (status, priority, estimate, iteration) การสร้างและใช้งาน

นี่คือหัวใจสำคัญที่ทำให้ Projects v2 ทรงพลังกว่า Issue tracker ทั่วไปมาก — **Custom Fields**

### Field มาตรฐานที่มีให้ตั้งแต่แรก

ทุก Project จะมี field พื้นฐานติดมาให้อัตโนมัติ ได้แก่ Title, Assignees, Status (มีค่าเริ่มต้น Todo/In Progress/Done), Labels, Repository, Milestone และ Linked pull requests

แต่ field เหล่านี้อาจไม่พอสำหรับการบริหารจัดการที่ซับซ้อนขึ้น จึงต้องสร้าง Custom Field เพิ่มเอง

### ประเภทของ Custom Field ที่สร้างได้

| ประเภท (Field type) | ใช้ทำอะไร | ตัวอย่าง |
|---|---|---|
| **Text** | ข้อความอิสระ | หมายเหตุเพิ่มเติม, ลิงก์เอกสาร |
| **Number** | ตัวเลข | Story point, จำนวนชั่วโมง |
| **Date** | วันที่ | Due date, Start date |
| **Single select** | เลือกค่าจากตัวเลือกที่กำหนดไว้ล่วงหน้า (มีสีกำกับได้) | Priority (High/Medium/Low), Status |
| **Iteration** | กำหนดรอบเวลาซ้ำ ๆ (เช่น sprint ทุก 2 สัปดาห์) โดยระบบจะสร้างช่วงเวลาให้อัตโนมัติต่อเนื่องกัน | Sprint 1, Sprint 2, Sprint 3, ... |

### วิธีสร้าง Custom Field

1. เปิด Project ไปที่ Table view
2. เลื่อนไปขวาสุดของตาราง คลิกเครื่องหมาย **+** ที่หัวคอลัมน์
3. ตั้งชื่อ field เช่น "Priority"
4. เลือกประเภท field เช่น **Single select**
5. ถ้าเป็น Single select ให้เพิ่มตัวเลือก เช่น `P0 - Critical`, `P1 - High`, `P2 - Medium`, `P3 - Low` และเลือกสีให้แต่ละอัน

### ตัวอย่าง Field ที่ทีมส่วนใหญ่นิยมสร้างเพิ่ม

**1. Priority (Single select)**

ใช้กำหนดลำดับความสำคัญของงาน ตัวอย่างค่าที่นิยมตั้ง:
- `🔴 Critical`
- `🟠 High`
- `🟡 Medium`
- `🟢 Low`

**2. Estimate (Number)**

ใช้ระบุ story point หรือจำนวนชั่วโมงที่คาดว่าจะใช้ทำงานนี้ เช่น `1`, `2`, `3`, `5`, `8` (ตามแนวคิด Fibonacci ของ Agile/Scrum) มีประโยชน์มากเมื่อใช้ร่วมกับ **Insights** ของ Project เพื่อดูภาระงานรวม (workload) ของแต่ละ sprint

**3. Iteration**

Field ประเภทนี้พิเศษกว่าตัวอื่น เพราะ GitHub จะช่วยจัดการช่วงเวลาให้อัตโนมัติ:

1. สร้าง field ประเภท **Iteration**
2. ตั้งความยาวของแต่ละ iteration (เช่น 2 สัปดาห์ — มาตรฐาน sprint ทั่วไป)
3. ตั้งวันเริ่มต้นของ iteration แรก
4. GitHub จะสร้าง iteration ถัดไปให้ต่อเนื่องกันอัตโนมัติ (Iteration 1, Iteration 2, Iteration 3, ...) และมี "Iteration breaks" ให้ตั้งช่วงพักได้ด้วย (เช่น ช่วงวันหยุดยาว)

field นี้เหมาะมากสำหรับทีมที่ทำงานแบบ Scrum เพราะสามารถ filter/group เพื่อดูว่างานไหนอยู่ sprint ปัจจุบัน งานไหนถูกเลื่อนไป sprint หน้า

**4. Status (Single select ที่มีมาให้ default แต่ปรับแต่งได้)**

field นี้มีมาให้ตั้งแต่สร้าง Project แล้ว แต่คุณสามารถแก้ไขชื่อค่า เพิ่มค่าใหม่ หรือเปลี่ยนสีได้ตามต้องการ ทีมจำนวนมากปรับจาก 3 คอลัมน์เริ่มต้น (Todo, In Progress, Done) เป็นละเอียดขึ้น เช่น `Backlog`, `Todo`, `In Progress`, `In Review`, `Blocked`, `Done`

### การแก้ไขค่า Field ให้กับแต่ละ Item

จาก Table view คุณสามารถคลิกที่ cell ใด ๆ แล้วแก้ไขค่าได้โดยตรง เหมือนแก้สเปรดชีต — เลือกวันที่จาก date picker, พิมพ์ตัวเลข, หรือเลือกจาก dropdown ของ single select

หากต้องการแก้หลายรายการพร้อมกัน (bulk update) ให้เลือกหลายแถวด้วย checkbox แล้วคลิกขวา เลือก field ที่จะแก้ และตั้งค่าใหม่ให้ทุกแถวที่เลือกพร้อมกันในคลิกเดียว

### สรุป Step 224

- Custom Field ทำให้ Project เก็บข้อมูลเชิงบริหารจัดการที่ Issue เพียว ๆ ไม่มี
- ประเภท field หลัก: Text, Number, Date, Single select, Iteration
- Iteration field พิเศษตรงที่ GitHub จัดการช่วงเวลาซ้ำให้อัตโนมัติ เหมาะกับทีม Scrum
- แก้ไขค่า field ได้ทั้งทีละรายการและแบบ bulk update

---

## Step 225: Automation ใน Projects (workflow เช่น auto-move card เป็น Done เมื่อ PR merge)

หนึ่งในจุดที่ทำให้ทีมประหยัดเวลาได้มากที่สุดคือการตั้งให้บอร์ด **ขยับเองอัตโนมัติ** โดยไม่ต้องมีใครมานั่งลาก card ทุกครั้งที่สถานะงานเปลี่ยน

### เข้าถึงเมนู Workflows

จากหน้า Project คลิกเมนู **... (More options)** ที่มุมขวาบน แล้วเลือก **Workflows** จะเห็นรายการ built-in workflow ที่พร้อมใช้งานทันที เพียงเปิด/ปิดและปรับแต่งเงื่อนไข

### Built-in Workflows ที่มีให้ใช้

**1. Item added to project**

เมื่อ Issue/PR ถูกเพิ่มเข้า Project ให้ตั้งค่า Status field เป็นค่าเริ่มต้นที่กำหนด (เช่น ตั้งเป็น `Todo` โดยอัตโนมัติทุกครั้ง เพื่อไม่ให้มีรายการที่ไม่มีสถานะค้างอยู่)

**2. Item reopened**

เมื่อ Issue/PR ที่เคยถูกปิดไปแล้วถูกเปิดกลับมาใหม่ ให้ย้าย Status กลับไปที่คอลัมน์ที่กำหนด เช่นย้ายกลับไป `In Progress`

**3. Item closed**

เมื่อ Issue ถูกปิด (close) ให้ย้าย Status เป็นค่าที่กำหนด ปกติตั้งเป็น `Done`

**4. Pull request merged**

นี่คือ workflow ที่ทีมส่วนใหญ่ใช้บ่อยที่สุด — **เมื่อ Pull Request ถูก merge เข้า branch หลักสำเร็จ ให้ย้าย card ของ PR นั้น (และ Issue ที่เชื่อมโยงกันผ่านคำสั่งเช่น `closes #42`) ไปยังคอลัมน์ `Done` โดยอัตโนมัติทันที** ไม่ต้องมีใครมาลากมือ

**5. Code changes requested / Code review approved**

ย้าย card ไปคอลัมน์ที่กำหนดเมื่อ PR ถูก request changes หรือถูก approve — เช่นตั้งให้ PR ที่ถูก approve แล้วขยับไปคอลัมน์ `Ready to merge` โดยอัตโนมัติ เพื่อให้ทีมเห็นชัดว่า PR ไหนพร้อม merge แล้ว

**6. Auto-close issue / Auto-archive items**

ตั้งให้ archive รายการที่อยู่ในสถานะ `Done` เกินระยะเวลาที่กำหนดโดยอัตโนมัติ เพื่อไม่ให้บอร์ดรกไปด้วยงานเก่าที่เสร็จไปนานแล้ว

### ตัวอย่างการตั้งค่าจริง

สมมติทีมต้องการ workflow แบบนี้:

1. Issue ใหม่ทุกอันจาก repo `backend` ที่มี label `bug` → เพิ่มเข้า Project อัตโนมัติ และตั้ง Status เป็น `Todo`
2. เมื่อมีคน assign ตัวเองและเริ่มทำ (เปิด PR ที่ลิงก์กับ Issue) → ทีมต้องลากมือเปลี่ยนเป็น `In Progress` (เพราะ default ไม่มี workflow ตรวจจับ "เริ่มทำงาน" อัตโนมัติ 100% แต่สามารถใช้ GitHub Actions เสริมได้ตามที่จะกล่าวถึงด้านล่าง)
3. เมื่อ PR ถูก merge → ย้าย Issue และ PR เป็น `Done` อัตโนมัติ

การตั้งค่าทำได้โดย:

1. ไปที่ **Workflows** → เปิด **Auto-add to project** → ตั้งเงื่อนไข `repo:backend label:bug`
2. เปิด **Item added to project** → ตั้ง target status = `Todo`
3. เปิด **Pull request merged** → ตั้ง target status = `Done`

### Automation ขั้นสูงกว่านั้น: GraphQL API + GitHub Actions

Built-in workflow ครอบคลุมเคสพื้นฐานส่วนใหญ่ แต่ถ้าต้องการ logic ที่ซับซ้อนกว่านั้น เช่น "ย้าย card ไป `Blocked` อัตโนมัติเมื่อ PR ค้าง review เกิน 3 วัน" คุณสามารถ:

- ใช้ **GitHub Actions** ร่วมกับ action สำเร็จรูปอย่าง `actions/add-to-project` เพื่อเพิ่มรายการเข้า Project จาก workflow ของคุณเอง
- เรียก **GraphQL API** ของ GitHub โดยตรง (Projects v2 ควบคุมผ่าน GraphQL เท่านั้น ไม่มี REST API เต็มรูปแบบสำหรับจัดการ field/item) เพื่อเขียนสคริปต์อัปเดตค่า field ตามเงื่อนไขที่ซับซ้อนกว่าที่ built-in workflow รองรับ

เรื่องการเขียน GitHub Actions มาเชื่อมกับ Projects แบบละเอียดจะอยู่ใน Part ที่พูดถึง GitHub Actions โดยเฉพาะในเฟส 7 ของหลักสูตรนี้

### สรุป Step 225

- Workflows ทำให้บอร์ดขยับเองเมื่อเหตุการณ์บางอย่างเกิดขึ้น ไม่ต้องลากมือทุกครั้ง
- Built-in workflow ที่สำคัญที่สุดคือ **Pull request merged** ซึ่งย้าย card เป็น Done อัตโนมัติเมื่อ PR merge สำเร็จ
- Auto-add to project ช่วยดึง Issue/PR เข้าบอร์ดตามเงื่อนไขที่ตั้งไว้ล่วงหน้า
- งานที่ซับซ้อนกว่า built-in รองรับ ให้ใช้ GitHub Actions ร่วมกับ GraphQL API

---

## Step 226: Board view แบบ Kanban — คอลัมน์ To do, In Progress, Done การลาก card

มาเจาะลึก View ที่คนใช้งานบ่อยที่สุดใน Projects v2 นั่นคือ **Board View**

### แนวคิดของ Kanban

**Kanban** (แปลว่า "ป้ายสัญญาณ" ในภาษาญี่ปุ่น) เป็นแนวคิดการบริหารงานที่มาจากระบบการผลิตของโตโยต้า หลักการคือการมองเห็นงานทั้งหมดเป็น "การ์ด" ที่เคลื่อนผ่านคอลัมน์ต่าง ๆ ตามลำดับขั้นตอนของงาน (workflow stage) ทำให้ทุกคนในทีมเห็นสถานะงานได้ทันทีโดยไม่ต้องถามใคร

### โครงสร้างพื้นฐานของ Board View

Board View ของ Projects v2 จะจัดกลุ่มการ์ดเป็นคอลัมน์ โดย **default จะ group ตาม field "Status"** ซึ่งมีค่าเริ่มต้น 3 ค่า:

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│    Todo     │  │ In Progress │  │    Done     │
├─────────────┤  ├─────────────┤  ├─────────────┤
│ Issue #12   │  │ Issue #15   │  │ Issue #8    │
│ Issue #14   │  │ PR #20      │  │ Issue #9    │
│ Issue #18   │  │             │  │ PR #11      │
│ Issue #22   │  │             │  │             │
└─────────────┘  └─────────────┘  └─────────────┘
```

### สิ่งที่แสดงบนการ์ดแต่ละใบ

การ์ดแต่ละใบใน Board View แสดงข้อมูลสรุปของ Issue/PR นั้น ๆ ได้แก่:

- Title (ชื่อเรื่อง)
- Assignee (avatar ของผู้รับผิดชอบ)
- Labels (ป้ายสี)
- Repository ที่มันสังกัดอยู่ (สำคัญมากเมื่อบอร์ดรวมหลาย repo)
- หมายเลข Issue/PR
- Custom field อื่น ๆ ที่คุณเลือกให้แสดง (เช่น Priority, Estimate)

คุณสามารถเลือกได้ว่าจะให้ field ไหนแสดงบนการ์ด ผ่านเมนู **View settings** (ไอคอนรูปเฟือง หรือ "..." ที่มุมขวาบนของ View) แล้วเลือกหัวข้อ **Fields** เพื่อ tick/untick field ที่ต้องการโชว์

### การลาก card ย้ายคอลัมน์ (Drag and Drop)

การลากการ์ดข้ามคอลัมน์ทำได้ง่ายมาก เพียงคลิกค้างที่การ์ดแล้วลากไปวางในคอลัมน์ปลายทาง — เมื่อปล่อยมือ **ค่าของ field ที่ใช้ group อยู่ (ปกติคือ Status) จะถูกอัปเดตทันทีให้ตรงกับคอลัมน์ปลายทางโดยอัตโนมัติ** เช่น ลาก Issue #14 จากคอลัมน์ `Todo` ไปยัง `In Progress` ระบบจะเปลี่ยนค่า Status field ของ Issue #14 เป็น `In Progress` ทันที ซึ่งจะสะท้อนไปยัง View อื่น ๆ ทั้งหมดที่อ้างอิงข้อมูลเดียวกันด้วย

### การเปลี่ยน field ที่ใช้จัดกลุ่มคอลัมน์

คุณไม่จำเป็นต้อง group ด้วย Status เสมอไป สามารถเปลี่ยนไปใช้ field อื่นแทนได้ เช่น:

- Group by **Assignee** — แต่ละคอลัมน์คือพนักงานหนึ่งคน เห็นภาระงานของแต่ละคนชัดเจน
- Group by **Priority** — แต่ละคอลัมน์คือระดับความสำคัญ
- Group by **Iteration** — แต่ละคอลัมน์คือ sprint แต่ละรอบ

วิธีเปลี่ยน: คลิก **View settings** (ไอคอนเฟือง) → เลือกหัวข้อ **Group by** → เลือก field ใหม่ที่ต้องการ

### การเพิ่มคอลัมน์ใหม่ (เพิ่มค่าให้ field Single select)

ถ้าต้องการเพิ่มคอลัมน์ เช่น `In Review` หรือ `Blocked` แทรกระหว่าง `In Progress` กับ `Done`:

1. ไปที่ Board View
2. คลิกเครื่องหมาย **+** ที่ปลายแถวคอลัมน์ หรือแก้ไข field `Status` โดยตรงจาก field settings
3. เพิ่มค่าใหม่ ตั้งชื่อและเลือกสี
4. คอลัมน์ใหม่จะปรากฏขึ้นทันทีในทุก Board View ที่ group ด้วย field นี้

### จำกัดจำนวนงานต่อคอลัมน์ (WIP Limit) — ยังไม่มี built-in

ทีมที่ใช้ Kanban แบบเข้มข้นมักตั้ง **WIP Limit** (Work In Progress Limit) คือจำกัดว่าคอลัมน์ `In Progress` ห้ามมีเกิน N การ์ดพร้อมกัน เพื่อบังคับให้ทีมโฟกัสทำงานให้เสร็จก่อนรับงานใหม่ ปัจจุบัน Projects v2 **ยังไม่มีฟีเจอร์ตั้ง WIP Limit แบบบังคับในตัว** ทีมจึงต้องอาศัยวินัยของตัวเองหรือดูจำนวนการ์ดในคอลัมน์ด้วยตา (ตัวเลขจำนวนการ์ดจะแสดงอยู่ข้างชื่อคอลัมน์ให้อัตโนมัติ) เป็นแนวทางกำกับดูแลแทน

### สรุป Step 226

- Board View คือมุมมองแบบ Kanban ที่แปลงข้อมูลใน Project เป็นการ์ดกระจายตามคอลัมน์
- Default group ด้วย field Status แต่เปลี่ยนไป group ด้วย field ไหนก็ได้
- ลากการ์ดข้ามคอลัมน์จะอัปเดตค่า field อัตโนมัติ ไม่ต้องเข้าไปแก้ทีละรายการ
- เพิ่มคอลัมน์ใหม่ได้โดยเพิ่มค่าใน single select field ที่ใช้ group อยู่
- ยังไม่มี WIP Limit บังคับในตัว ต้องอาศัยวินัยทีม

---

## Step 227: Roadmap view — timeline planning สำหรับวางแผนระยะยาว

ถ้า Board View เหมาะกับ "วันนี้ทำอะไรอยู่" **Roadmap View** ก็เหมาะกับคำถามที่ว่า "ไตรมาสนี้เราจะส่งมอบอะไรบ้าง และงานไหนจะเสร็จก่อน-หลังกัน"

### หน้าตาของ Roadmap View

Roadmap View แสดงผลเป็น **timeline แนวนอน** โดยแกนซ้าย-ขวาคือเวลา (วัน สัปดาห์ เดือน หรือไตรมาส ขึ้นอยู่กับระดับการซูม) และแต่ละแถวคือ Issue/PR หนึ่งรายการที่แสดงเป็น "แท่ง" (bar) ยาวตามระยะเวลาที่กำหนด

```
                Q1 2026              Q2 2026
        ├───────────┼───────────┼───────────┤
Feature A   [████████████]
Feature B          [██████████████]
Bug fix C   [███]
Feature D                  [████████████████]
```

### ข้อกำหนดเบื้องต้นก่อนใช้ Roadmap View ได้จริง

Roadmap View จะแสดงแท่งเวลาได้ก็ต่อเมื่อ Issue/PR นั้นมีค่าใน field ประเภท **Date** อย่างน้อย 2 field คือ **วันเริ่มต้น (Start date)** และ **วันสิ้นสุด/กำหนดส่ง (Target date หรือ Due date)** — ถ้ายังไม่มี field เหล่านี้ ต้องสร้างขึ้นก่อน (ตามวิธีใน Step 224 โดยเลือกประเภท Date) แล้วกรอกค่าให้แต่ละ Issue

รายการที่ยังไม่มีวันที่กำหนดจะไปปรากฏอยู่ในส่วน **"Items without a start or target date"** ด้านล่างของ timeline แทน

### การจัดกลุ่มใน Roadmap View

เช่นเดียวกับ Board View คุณสามารถ **group by** field ต่าง ๆ ได้ เช่น:

- Group by **Milestone** — เห็นภาพว่าแต่ละ milestone ประกอบด้วยงานอะไรบ้างและกินเวลานานแค่ไหน
- Group by **Repository** — เหมาะมากเมื่อ Roadmap รวมงานจากหลายทีม/หลาย repo แล้วอยากแยกดูเป็นแถบตามทีม
- Group by **Iteration** — เห็นว่าแต่ละ sprint ครอบคลุมช่วงเวลาไหนบ้าง

### การซูมระดับเวลา (Zoom level)

มุมขวาบนของ Roadmap View มีตัวเลือกปรับระดับการมองเวลา:

- **Week** — เหมาะกับการวางแผนระยะสั้น 1-2 เดือน
- **Month** — เหมาะกับการวางแผนไตรมาส
- **Quarter** — เหมาะกับการวางแผนทั้งปี ดูภาพกว้างของ roadmap ผลิตภัณฑ์

### การปรับช่วงเวลาโดยการลาก (Drag to resize)

จุดที่ทำให้ Roadmap View ใช้งานสะดวกมากคือ คุณสามารถ **ลากขอบซ้าย-ขวาของแท่งเวลา** เพื่อเปลี่ยน start date/target date ได้โดยตรงบนหน้าจอ โดยไม่ต้องเปิดไปแก้ทีละ field ในตาราง หรือลากทั้งแท่งเพื่อเลื่อนทั้งช่วงเวลาไปพร้อมกันก็ได้ (เลื่อนทั้ง start และ target date พร้อมกันโดยรักษาระยะเวลาเท่าเดิม)

### เส้น Marker แสดงวันสำคัญ

Roadmap View รองรับการแสดง **Markers** ซึ่งเป็นเส้นแนวตั้งปักไว้บน timeline เพื่อระบุวันสำคัญ เช่น "วันเปิดตัวสินค้า" หรือ "deadline ของลูกค้า" ช่วยให้ทีมเห็นชัดว่างานที่วางแผนไว้จะเสร็จทันเส้นตายหรือไม่เมื่อมองจากภาพรวม

### เมื่อไหร่ควรใช้ Roadmap View เทียบกับ Board View

| สถานการณ์ | View ที่เหมาะสม |
|---|---|
| ประชุม stand-up รายวัน ดูว่าใครทำอะไรอยู่ | Board View |
| วางแผน sprint ถัดไป | Board View (group by Iteration) |
| นำเสนอแผนงานไตรมาสให้ผู้บริหาร | Roadmap View |
| เช็คว่างานสองอย่างจะเสร็จทับซ้อนช่วงเวลากันหรือไม่ | Roadmap View |
| ดูรายละเอียดข้อมูลทุก field พร้อมกันเพื่อแก้ไขจำนวนมาก | Table View |

### สรุป Step 227

- Roadmap View แสดง Issue/PR เป็นแท่งเวลาบน timeline แนวนอน เหมาะกับการวางแผนระยะยาว
- ต้องมี Date field (start date + target date) ก่อนถึงจะแสดงแท่งเวลาได้
- ปรับ zoom ได้ตั้งแต่ระดับสัปดาห์ถึงไตรมาส และลากขอบแท่งเพื่อเปลี่ยนวันที่ได้โดยตรง
- ใช้ Markers ปักหมุดวันสำคัญบน timeline ได้

---

## Step 228: Filtering และ grouping ใน Project (group by status, assignee, label)

เมื่อ Project มีรายการเป็นร้อยเป็นพัน การมองเห็นทุกอย่างพร้อมกันจะทำให้สับสน ฟีเจอร์ **Filter** และ **Group** จึงสำคัญมากในการทำให้ข้อมูลอ่านง่ายขึ้นตามบริบทที่ต้องการ ณ ขณะนั้น

### Filtering — กรองให้เห็นเฉพาะที่ต้องการ

แถบค้นหา/filter อยู่ด้านบนของทุก View ใช้ไวยากรณ์คล้ายกับการค้นหา Issue ทั่วไปของ GitHub เช่น:

```
is:open assignee:@me
```
กรองให้เห็นเฉพาะรายการที่ยังเปิดอยู่และ assign ให้ตัวเอง

```
label:bug status:"In Progress"
```
กรองเฉพาะรายการที่มี label `bug` และ Status เป็น `In Progress`

```
repo:myorg/backend priority:P0
```
กรองเฉพาะรายการจาก repo `backend` ที่มี priority `P0`

```
no:assignee
```
กรองหารายการที่ยังไม่มีใคร assign เลย — มีประโยชน์มากตอนประชุม planning เพื่อหางานที่ยังไม่มีเจ้าของ

คุณสามารถรวมเงื่อนไขหลายอย่างพร้อมกันได้ในแถบเดียว และ Filter ที่ตั้งไว้จะถูกบันทึกแยกต่างหากในแต่ละ View — สลับไป View อื่นแล้วกลับมา Filter เดิมจะยังอยู่

### บันทึก Filter เป็นส่วนหนึ่งของ View

จุดสำคัญคือ **Filter ที่ตั้งไว้ใน View หนึ่งจะไม่กระทบ View อื่น** เพราะการตั้งค่าการแสดงผล (filter, sort, group, field ที่โชว์) ถูกผูกติดกับแต่ละ View แยกกันโดยสมบูรณ์ — นี่คือเหตุผลที่ทีมนิยมสร้างหลาย View สำหรับจุดประสงค์ต่างกัน เช่น View "My items" ที่ filter `assignee:@me` ค้างไว้ถาวร โดยไม่ต้องพิมพ์ filter ใหม่ทุกครั้งที่เข้ามาดู

### Grouping — จัดกลุ่มข้อมูลให้อ่านง่าย

Grouping ใช้ได้ทั้งใน Table View และ Board View (ใน Roadmap View เรียกว่า "Group by" เช่นกันตามที่กล่าวไปใน Step 227) วิธีตั้งค่า:

1. คลิก **View settings** (ไอคอนเฟือง มุมขวาบนของ View)
2. เลือกหัวข้อ **Group by**
3. เลือก field ที่ต้องการจัดกลุ่ม

ตัวอย่างการ group ที่ทีมนิยมใช้:

- **Group by Status** — มาตรฐานของ Board View (ค่า default)
- **Group by Assignee** — เห็นภาระงานของแต่ละคนในทีมชัดเจน ใครมีงานเยอะเกินไปก็ปรับ workload ให้สมดุลได้ทันที
- **Group by Label** — เหมาะเมื่ออยากดูภาพรวมตามหมวดหมู่ เช่น `bug`, `feature`, `documentation`
- **Group by Repository** — เมื่อ Project รวมหลาย repo อยากแยกดูว่าแต่ละ repo มีงานค้างเท่าไหร่
- **Group by Milestone** — ดูความคืบหน้าของแต่ละ milestone/release
- **Group by Iteration** — ดูงานแยกตาม sprint

### Sorting — เรียงลำดับภายในแต่ละกลุ่ม

นอกจาก group แล้ว ยังตั้ง **Sort** ได้ เช่น เรียงตาม Priority จากสูงไปต่ำ หรือเรียงตามวันที่สร้าง (created date) จากใหม่ไปเก่า การ sort จะทำงานภายในแต่ละกลุ่มที่ group ไว้ ทำให้ภายในคอลัมน์เดียวกันของ Board View การ์ดที่ priority สูงสุดจะลอยขึ้นมาอยู่บนสุดเสมอ

### Slicing — แบ่งข้อมูลเป็นแถบข้าง (เฉพาะบาง View)

Table View ยังมีฟีเจอร์ **Slice by** ซึ่งจะแสดงแถบด้านซ้ายมือแบ่งข้อมูลตามค่าของ field ที่เลือก (คล้าย pivot table) เช่น slice by Assignee จะให้คุณคลิกเลือกชื่อคนในแถบซ้าย แล้วตารางด้านขวาจะกรองอัตโนมัติให้เห็นเฉพาะงานของคนนั้น เป็นอีกวิธีสำรวจข้อมูลแบบโต้ตอบได้เร็ว

### ตัวอย่างการผสมผสาน Filter + Group + Sort

สถานการณ์: Scrum Master ต้องการดูเฉพาะงานของ sprint ปัจจุบันที่ยังไม่เสร็จ จัดกลุ่มตามคนรับผิดชอบ และเรียงตาม priority

- **Filter:** `iteration:"Sprint 12" -status:Done`
- **Group by:** Assignee
- **Sort by:** Priority (descending)

ผลลัพธ์คือมุมมองที่ตอบคำถาม "sprint นี้ใครเหลืองานอะไรอยู่บ้าง เรียงจากสำคัญที่สุดก่อน" ได้ในหน้าจอเดียว

### สรุป Step 228

- Filter ใช้ syntax คล้าย GitHub search เพื่อกรองข้อมูลตามเงื่อนไข และผูกกับแต่ละ View แยกกัน
- Group by จัดกลุ่มข้อมูลตาม field ใดก็ได้ ใช้ได้ทั้ง Table, Board และ Roadmap View
- Sort เรียงลำดับภายในแต่ละกลุ่ม
- Slice by ใน Table View ช่วยสำรวจข้อมูลแบบ pivot table แบบโต้ตอบ

---

## Step 229: การแชร์ Project ระหว่างหลาย repo (Organization-level project)

หัวข้อนี้ขยายความจาก Step 221-223 ให้ลึกขึ้น เจาะจงไปที่การใช้งาน Project ในระดับองค์กรที่มีหลายทีม หลาย repository ทำงานร่วมกัน

### ทำไมต้องมี Organization-level Project

บริษัทซอฟต์แวร์จริงมักมี repository แยกกันตามลักษณะงาน เช่น:

- `frontend` — เว็บแอปฝั่ง client
- `backend-api` — server และ API
- `mobile-app` — แอปมือถือ
- `infra` — infrastructure as code
- `docs` — เอกสารประกอบ

ถ้าผู้บริหารหรือ Product Manager ต้องการเห็นภาพรวมของ "ฟีเจอร์ X" ที่ต้องใช้แรงจากทั้ง 4 repo พร้อมกัน การเปิดดู Issue ทีละ repo แยกกันจะไม่มีทางเห็นภาพรวมได้เลย **Organization-level Project** จึงถูกออกแบบมาเพื่อแก้ปัญหานี้โดยเฉพาะ — สร้าง Project เดียวที่ผูกกับ Organization แล้วดึง Issue/PR จากทุก repo ในองค์กรเข้ามารวมกันในบอร์ดเดียว

### สิทธิ์การเข้าถึง (Permissions) ของ Organization Project

Project ระดับ Organization มีระบบสิทธิ์แยกต่างหากจากสิทธิ์ของ repository โดยแบ่งเป็นระดับ:

- **Read** — ดูอย่างเดียว ไม่สามารถแก้ไขได้
- **Write** — เพิ่ม/แก้ไข item และ field ได้ แต่แก้การตั้งค่า Project เองไม่ได้
- **Admin** — ควบคุมได้ทุกอย่างรวมถึงตั้งค่า Project, ลบ Project, จัดการสิทธิ์คนอื่น

จุดสำคัญคือ **สิทธิ์ในการเข้าถึง Project ไม่ได้ผูกกับสิทธิ์ในการเข้าถึง repository โดยอัตโนมัติ** เช่น คนคนหนึ่งอาจมีสิทธิ์ Write ใน Project แต่ไม่มีสิทธิ์เข้าถึง repo บาง repo ที่ Issue นั้นสังกัดอยู่เลยก็ได้ — ในกรณีนี้ผู้ใช้จะเห็นการ์ดใน Project แต่ถ้าคลิกเข้าไปดูรายละเอียด Issue จะไม่สามารถเปิดดูเนื้อหาจริงได้เพราะไม่มีสิทธิ์ใน repo ต้นทาง การตั้งค่าสิทธิ์ทั้งสองชั้นนี้จึงต้องวางแผนให้สอดคล้องกัน

### ตั้งค่า Visibility ของ Project

Project สามารถตั้งเป็น:

- **Public** — ใครก็เห็นได้ (เหมาะกับโปรเจกต์ Open Source ที่ต้องการความโปร่งใส)
- **Private** — เห็นเฉพาะสมาชิกในองค์กรที่ได้รับสิทธิ์เท่านั้น

ตั้งค่าได้จากเมนู **Settings** ของ Project → หัวข้อ **Visibility**

### การจัดการหลายทีมในบอร์ดเดียว

เมื่อ Project รวมข้อมูลจากหลาย repo/หลายทีม เทคนิคที่ทีมมืออาชีพนิยมใช้เพื่อไม่ให้บอร์ดสับสน:

1. **สร้าง Custom Field ชื่อ "Team"** (Single select) แล้วกำหนดค่าให้แต่ละ Issue ว่าสังกัดทีมไหน (Frontend, Backend, Mobile, Infra) แม้ Issue นั้นจะมาจาก repo เดียวกันแต่ทีมที่รับผิดชอบอาจต่างกันได้
2. **สร้าง View แยกตามทีม** โดยตั้ง Filter `team:Frontend` ไว้ในแต่ละ View เพื่อให้แต่ละทีมเข้ามาดูเฉพาะงานของตัวเอง โดยไม่รบกวนการมองเห็นของทีมอื่น
3. **ใช้ Roadmap View รวมทุกทีม** สำหรับการประชุมระดับผู้บริหารที่ต้องการเห็นภาพรวมทั้งหมดพร้อมกันว่างานจากทุกทีมจะมาบรรจบกันที่วันเปิดตัวสินค้าได้ตรงเวลาหรือไม่

### การย้าย Project จากระดับ Personal ไปเป็น Organization

หากเริ่มสร้าง Project ไว้ในระดับส่วนตัวก่อน (Profile) แล้วภายหลังต้องการยกระดับให้เป็นของทีม สามารถ **transfer ownership** ได้ผ่านหน้า Settings ของ Project → **Danger zone** → **Transfer ownership** — เลือกโอนไปเป็นของ Organization ที่ต้องการ (ผู้โอนต้องมีสิทธิ์ในการสร้าง Project ใน Organization ปลายทางด้วย)

### Insights — ภาพรวมเชิงสถิติของ Project

Project ระดับ Organization ยังมีแท็บ **Insights** ที่สร้างกราฟสรุปข้อมูลอัตโนมัติจาก field ต่าง ๆ เช่น กราฟแท่งแสดงจำนวน Issue แยกตาม Status, กราฟ burndown แบบง่ายจาก Iteration field และ Estimate field ช่วยให้ทีมประเมินความคืบหน้าของ sprint ได้โดยไม่ต้องนับมือ

### สรุป Step 229

- Organization-level Project รวม Issue/PR จากหลาย repo ทั่วทั้งองค์กรเข้าบอร์ดเดียว
- สิทธิ์ของ Project (Read/Write/Admin) แยกต่างหากจากสิทธิ์ของ repository แต่ละตัว
- ใช้ Custom Field + View แยกตามทีม เพื่อให้แต่ละทีมโฟกัสเฉพาะงานตัวเองได้ในบอร์ดเดียวกัน
- ย้าย ownership ของ Project จาก personal ไป organization ได้ผ่าน Transfer ownership
- แท็บ Insights ให้กราฟสรุปข้อมูลอัตโนมัติ

---

## Step 230: แบบฝึกหัด — สร้าง Kanban board บริหารงานโปรเจกต์จำลองครบวงจร (อย่างน้อย 10 การ์ด กระจาย 3 คอลัมน์)

ถึงเวลาลงมือทำจริงแล้ว แบบฝึกหัดนี้จะรวมทุกอย่างที่เรียนมาใน Part นี้เข้าด้วยกัน

### โจทย์

สมมติคุณกำลังพัฒนาแอปจำลองชื่อ **"TaskFlow"** — แอปจัดการงานส่วนตัวขนาดเล็ก คุณต้องสร้าง GitHub Project เพื่อบริหารจัดการงานพัฒนาทั้งหมด โดยมีข้อกำหนดดังนี้

### ขั้นตอนที่ 1: เตรียม Repository

ถ้ายังไม่มี repo ฝึกหัดจาก Part ก่อนหน้า ให้สร้าง repo ใหม่ชื่อ `taskflow-demo` (ตาม Part 17 ที่เคยเรียนไปแล้ว) หรือใช้ repo ฝึกหัดเดิมที่มีอยู่ก็ได้

### ขั้นตอนที่ 2: สร้าง Issue อย่างน้อย 10 รายการ

สร้าง Issue จำลองต่อไปนี้ในหรือหลาย repo (จะสร้างใน repo เดียว หรือกระจาย 2 repo เพื่อฝึกความสามารถ "ดึงจากหลาย repo" ก็ได้):

1. `ออกแบบหน้า Login`
2. `เชื่อมต่อฐานข้อมูล PostgreSQL`
3. `สร้างระบบ Authentication ด้วย JWT`
4. `เขียนหน้า Dashboard แสดงรายการงาน`
5. `แก้บั๊ก: ปุ่มลบงานกดไม่ติดบน Safari`
6. `เพิ่มฟีเจอร์ Drag and Drop จัดลำดับงาน`
7. `เขียน Unit Test สำหรับ API endpoint /tasks`
8. `ตั้งค่า CI pipeline ด้วย GitHub Actions`
9. `เขียนเอกสาร README และ API documentation`
10. `Deploy เวอร์ชันแรกขึ้น production`
11. (โบนัส) `รีวิวและปรับปรุง UI/UX หน้า Settings`

### ขั้นตอนที่ 3: สร้าง Project ใหม่

1. สร้าง Project ใหม่ชื่อ **"TaskFlow Development Board"** จากระดับ Profile หรือ Organization
2. ใช้ template "Board" หรือเริ่มจาก Table เปล่าก็ได้

### ขั้นตอนที่ 4: เพิ่ม Custom Fields

สร้าง field เพิ่มอย่างน้อยตามนี้:

- **Priority** (Single select): `P0 - Critical`, `P1 - High`, `P2 - Medium`, `P3 - Low`
- **Estimate** (Number): กรอกเป็นจำนวนชั่วโมงโดยประมาณ เช่น 2, 4, 8, 16
- **Iteration** (Iteration): ตั้งความยาว 1 สัปดาห์ต่อรอบ เริ่มจากวันนี้

กำหนดค่าให้ Issue ทั้ง 10-11 รายการที่สร้างไว้ ให้มี Priority และ Estimate ครบทุกใบ

### ขั้นตอนที่ 5: เพิ่ม Issue ทั้งหมดเข้า Project

ใช้วิธี bulk-add จากหน้า Issues list ของ repo (เลือกทั้งหมดพร้อมกันแล้ว Add to project) หรือจะตั้ง **Auto-add to project** workflow ก็ได้ เพื่อฝึกทั้งสองวิธี

### ขั้นตอนที่ 6: จัดกระจาย Issue ลง 3 คอลัมน์

สร้าง Board View (ถ้ายังไม่มี) และปรับ Status field ให้มีอย่างน้อย 3 ค่า: `Todo`, `In Progress`, `Done` จากนั้นกระจาย Issue ทั้ง 10 ใบลง 3 คอลัมน์นี้ให้ครบทั้งสามคอลัมน์ (ไม่ต้องเท่ากันเป๊ะ) เช่น:

- **Todo:** 5 ใบ (งานที่ยังไม่เริ่ม)
- **In Progress:** 3 ใบ (งานที่กำลังทำ)
- **Done:** 3 ใบ (งานที่เสร็จแล้ว)

ลองลาก card อย่างน้อย 2-3 ใบข้ามคอลัมน์ด้วยมือ เพื่อฝึกความชินกับ drag-and-drop

### ขั้นตอนที่ 7: ตั้ง Automation

เปิดใช้งาน built-in workflow อย่างน้อย 2 อัน:

1. **Item added to project** → ตั้ง default status เป็น `Todo`
2. **Pull request merged** → ตั้ง target status เป็น `Done`

ทดสอบจริงโดยสร้าง Pull Request เล็ก ๆ ที่ผูกกับ Issue หนึ่งใบ (ใช้คำสั่ง `closes #<เลข issue>` ในคำอธิบาย PR ตามที่เรียนใน Part 21) แล้วลอง merge PR นั้น สังเกตว่า card ของ Issue ที่ถูก close ขยับไปที่ `Done` เองโดยอัตโนมัติหรือไม่

### ขั้นตอนที่ 8: ทดลอง Group และ Filter

- เปลี่ยน Board View ให้ group by **Priority** แทน Status แล้วสังเกตว่าหน้าตาบอร์ดเปลี่ยนไปอย่างไร
- ใช้ filter `priority:"P0 - Critical"` เพื่อดูเฉพาะงานด่วนที่สุด
- เปลี่ยนกลับมา group by Status ตามเดิม

### ขั้นตอนที่ 9: ลองสร้าง Roadmap View

1. เพิ่ม Date field สองตัว: `Start date` และ `Target date`
2. กรอกวันที่ให้ Issue อย่างน้อย 5 ใบ (ให้บางใบซ้อนช่วงเวลากัน เพื่อทดสอบการมองเห็น overlap)
3. สร้าง View ใหม่แบบ Roadmap แล้วดูผลลัพธ์
4. ลองลากขอบแท่งเวลาเพื่อเปลี่ยนวันที่ดูสักหนึ่งรายการ

### Checklist ตรวจสอบความสำเร็จของแบบฝึกหัด

- [ ] สร้าง Issue อย่างน้อย 10 รายการสำเร็จ
- [ ] สร้าง Project ใหม่ชื่อ "TaskFlow Development Board" สำเร็จ
- [ ] สร้าง Custom Field ครบ 3 ชนิด: Single select (Priority), Number (Estimate), Iteration
- [ ] เพิ่ม Issue ทั้งหมดเข้า Project ได้สำเร็จ (ด้วยวิธี bulk-add หรือ auto-add)
- [ ] Board View มีอย่างน้อย 3 คอลัมน์ (Todo/In Progress/Done) และมีการ์ดกระจายอยู่ครบทั้ง 3 คอลัมน์
- [ ] ลาก card ข้ามคอลัมน์ได้จริงอย่างน้อย 1 ครั้งและเห็นค่า Status เปลี่ยนตาม
- [ ] ตั้ง Automation workflow อย่างน้อย 2 อัน และทดสอบว่า "Pull request merged" ทำงานได้จริง
- [ ] ทดลอง group by field อื่นนอกจาก Status ได้สำเร็จ (เช่น Priority)
- [ ] ทดลองใช้ filter อย่างน้อย 1 เงื่อนไข
- [ ] สร้าง Roadmap View และเห็นแท่งเวลาของ Issue ที่มี Date field ครบ

เมื่อทำครบทุกข้อแล้ว แสดงว่าคุณเข้าใจการทำงานของ GitHub Projects v2 อย่างครบวงจรตั้งแต่การสร้างบอร์ด การจัดการข้อมูล ไปจนถึงการทำ automation แล้ว

---

## สรุป Part 23

ใน Part นี้เราได้เรียนรู้ว่า:

1. **GitHub Projects (v2)** คือเลเยอร์บริหารจัดการงานที่วางอยู่เหนือ Issues/PR ไม่ใช่สิ่งเดียวกัน และ Projects (classic) ถูกยกเลิกไปแล้วอย่างสมบูรณ์
2. Project ทำงานเหมือนตารางข้อมูลที่มี Custom Field ได้ และแสดงผลได้หลาย View: **Table, Board, Roadmap**
3. เพิ่ม Issue/PR เข้า Project ได้จากหลายวิธี รวมถึงการ **auto-add จากหลาย repository พร้อมกัน** ซึ่งเป็นจุดแข็งสำคัญของ v2
4. **Custom Fields** เช่น Status, Priority, Estimate และ Iteration ทำให้ Project เก็บข้อมูลเชิงบริหารจัดการได้ลึกกว่า Issue tracker ทั่วไป
5. **Automation/Workflows** ทำให้บอร์ดขยับเองอัตโนมัติ เช่นย้าย card เป็น Done เมื่อ PR merge สำเร็จ ลดงานที่ต้องทำมือ
6. **Board View** คือมุมมอง Kanban สำหรับดูงานประจำวัน ลากการ์ดข้ามคอลัมน์ได้ทันที
7. **Roadmap View** คือมุมมอง timeline สำหรับวางแผนระยะยาวและดูภาพรวมการส่งมอบงาน
8. **Filter, Group, Sort** ช่วยให้ทีมมองเห็นข้อมูลชุดเดียวกันในมุมที่ต่างกันได้ตามความต้องการ
9. **Organization-level Project** ทำให้หลายทีมหลาย repo บริหารจัดการงานร่วมกันในบอร์ดเดียวได้ พร้อมระบบสิทธิ์ที่แยกอิสระจากสิทธิ์ของ repository
10. ลงมือสร้าง Kanban board บริหารโปรเจกต์จำลอง "TaskFlow" ครบวงจรตั้งแต่สร้าง Issue จนถึงทดสอบ automation จริง

**ต่อไป:** [Part 24: Fork และการ Contribute แบบ Open Source เบื้องต้น](./part-024-fork-contribute-open-source.md)
