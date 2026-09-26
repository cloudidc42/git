# Part 26: Labels, Milestones, Issue/PR Templates

> **Step ในหลักสูตรนี้:** Step 251–260
> **เฟส:** 3 — ใช้งาน GitHub อย่างมืออาชีพ ทำ Pull Request, Code Review, Open Source
> **เป้าหมายของ Part นี้:** เข้าใจและใช้งานเครื่องมือจัดระเบียบงานของ GitHub อย่างลึกซึ้ง — Labels สำหรับจัดหมวดหมู่ Issue/PR, Milestones สำหรับวางแผนและติดตาม release, Issue Templates (ทั้งแบบ Markdown และแบบ YAML Forms) สำหรับมาตรฐานการรายงานปัญหา และ Pull Request Templates สำหรับมาตรฐานการส่งโค้ด พร้อมฝึกสร้างชุดเครื่องมือเหล่านี้ให้ครบสำหรับโปรเจกต์จริง

---

## สารบัญของ Part นี้

- Step 251: Labels เชิงลึก — สร้าง label เอง กำหนดสี การใช้ label อย่างเป็นระบบในทีม
- Step 252: Default labels ของ GitHub และความหมายของแต่ละตัว
- Step 253: Milestones เชิงลึก — วางแผน release ด้วย milestone
- Step 254: ติดตามความคืบหน้า milestone ด้วย progress bar
- Step 255: Issue templates แบบ Markdown — สร้างฟอร์มมาตรฐานสำหรับรายงานปัญหา
- Step 256: Issue forms แบบ YAML — ฟอร์มโครงสร้างที่มี dropdown, checkbox, required fields
- Step 257: Pull Request template
- Step 258: การจัดระเบียบ label หลาย repo ในระดับ organization (label sync)
- Step 259: Best practices สำหรับตั้งชื่อและใช้ label/milestone ในทีมจริง
- Step 260: แบบฝึกหัด — สร้างชุด label, milestone, issue template และ PR template แบบเต็มรูปแบบ

---

## Step 251: Labels เชิงลึก — สร้าง label เอง กำหนดสี การใช้ label อย่างเป็นระบบในทีม

### Label คืออะไร

**Label** คือ **ป้ายกำกับสี** ที่ติดไว้กับ Issue หรือ Pull Request เพื่อช่วยจัดหมวดหมู่ กรอง และสื่อสารสถานะของงานให้ทั้งทีมเห็นภาพเดียวกันได้อย่างรวดเร็ว โดยไม่ต้องเปิดอ่านรายละเอียดทีละอัน

ลองนึกภาพ Issue tracker ที่มี Issue อยู่ 300 รายการ ถ้าไม่มี label เลย คุณจะไม่มีทางรู้ได้เลยว่าอันไหนคือบั๊กร้ายแรง อันไหนเป็นแค่คำถาม อันไหนเหมาะกับมือใหม่ที่เพิ่งเข้าร่วมทีม — label คือคำตอบของปัญหานี้

Label มีคุณสมบัติหลัก ๆ 3 อย่าง:

| คุณสมบัติ | รายละเอียด |
|---|---|
| **ชื่อ (name)** | ข้อความสั้น ๆ เช่น `bug`, `priority: high` |
| **สี (color)** | รหัสสีแบบ hex 6 หลัก เช่น `d73a4a` (ไม่ต้องใส่ `#` นำหน้าตอนกรอกในบาง context) |
| **คำอธิบาย (description)** | ข้อความอธิบายเพิ่มเติมไม่เกิน 100 ตัวอักษร แสดงเป็น tooltip เวลาชี้เมาส์ |

### วิธีสร้าง Label ผ่านหน้าเว็บ (UI)

1. ไปที่หน้า repository บน GitHub
2. คลิกแท็บ **Issues**
3. คลิกปุ่ม **Labels** (อยู่ข้าง ๆ ปุ่ม Milestones)
4. คลิกปุ่ม **New label** สีเขียวมุมขวาบน
5. กรอก **Label name**, **Description**, และเลือกสีจากช่อง Color (มีปุ่มลูกเต๋าสำหรับสุ่มสีอัตโนมัติ หรือกรอก hex code เองก็ได้)
6. กด **Create label**

หน้าจอ URL ของหน้า labels คือ:

```
https://github.com/<owner>/<repo>/labels
```

### วิธีสร้าง Label ผ่าน GitHub CLI (`gh`)

การจัดการ label ผ่าน command line สะดวกกว่ามากเมื่อต้องสร้างหลาย label พร้อมกันหรือเขียนเป็นสคริปต์:

```bash
# สร้าง label ใหม่
gh label create "priority: high" --color "d73a4a" --description "ต้องแก้ไขด่วนที่สุด"

# ดูรายการ label ทั้งหมดใน repo ปัจจุบัน
gh label list

# แก้ไข label ที่มีอยู่แล้ว (เปลี่ยนสี/ชื่อ/คำอธิบาย)
gh label edit "priority: high" --color "b60205"

# ลบ label
gh label delete "priority: high"

# clone label ทั้งหมดจาก repo หนึ่งไปยังอีก repo หนึ่ง
gh label clone <source-owner>/<source-repo> --repo <target-owner>/<target-repo>
```

### วิธีจัดการ Label ผ่าน REST API

สำหรับการเขียนสคริปต์อัตโนมัติหรือทำ CI/CD ที่ต้องจัดการ label จำนวนมาก สามารถเรียก REST API ได้โดยตรง:

```bash
# สร้าง label ใหม่
curl -X POST \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/<owner>/<repo>/labels \
  -d '{"name":"priority: high","color":"d73a4a","description":"ต้องแก้ไขด่วนที่สุด"}'

# ดูรายการ label ทั้งหมด
curl -H "Authorization: Bearer <TOKEN>" \
  https://api.github.com/repos/<owner>/<repo>/labels

# แก้ไข label (ใช้ชื่อเดิมใน URL)
curl -X PATCH \
  -H "Authorization: Bearer <TOKEN>" \
  https://api.github.com/repos/<owner>/<repo>/labels/priority:%20high \
  -d '{"new_name":"priority: critical","color":"b60205"}'

# ลบ label
curl -X DELETE \
  -H "Authorization: Bearer <TOKEN>" \
  https://api.github.com/repos/<owner>/<repo>/labels/priority:%20critical
```

### การเลือกสี (Color) อย่างมีความหมาย

สี hex เป็นค่า RRGGBB ในระบบ 0–255 (00–FF) GitHub จะคำนวณความสว่างของสีเพื่อตัดสินใจว่าจะแสดงตัวหนังสือบน label เป็นสีดำหรือสีขาวให้อ่านง่ายที่สุดโดยอัตโนมัติ คุณไม่ต้องกังวลเรื่อง contrast เอง

แนวทางเลือกสีที่ทีมส่วนใหญ่ใช้:

| โทนสี | ความหมายที่นิยมสื่อ |
|---|---|
| แดง (`#d73a4a`, `#b60205`) | บั๊ก, ความสำคัญสูง/วิกฤต |
| ส้ม/เหลือง (`#e4e669`, `#fbca04`) | ต้องการข้อมูลเพิ่มเติม, ความสำคัญปานกลาง |
| เขียว (`#0e8a16`, `#c2e0c6`) | พร้อม merge, ผ่านการรีวิวแล้ว, ความสำคัญต่ำ |
| ฟ้า/น้ำเงิน (`#0075ca`, `#a2eeef`) | เอกสาร, ฟีเจอร์ใหม่ (enhancement) |
| ม่วง (`#7057ff`, `#d876e3`) | เหมาะกับมือใหม่, คำถาม |
| เทา (`#cfd3d7`, `#ffffff`) | สถานะกลาง ๆ เช่น ซ้ำ, ปิดโดยไม่แก้ |

### การใช้ Label อย่างเป็นระบบ: แนวคิด Multi-dimensional Labeling

ทีมมืออาชีพมักไม่ใช้ label แบบสุ่ม ๆ แต่จะออกแบบให้ label แบ่งเป็น **มิติ (dimension)** ที่ชัดเจน และแต่ละ Issue สามารถมี label จากหลายมิติพร้อมกันได้ (ต่างจาก Milestone ที่ Issue หนึ่งใบผูกได้แค่อันเดียว) ตัวอย่างมิติที่นิยมใช้:

```
type: bug
type: feature
type: docs
type: chore
─────────────
priority: critical
priority: high
priority: medium
priority: low
─────────────
area: frontend
area: backend
area: database
area: ci-cd
─────────────
status: needs-triage
status: in-progress
status: blocked
status: ready-for-review
```

การใช้ prefix แบบ `namespace: value` แบบนี้ (เรียกว่า **scoped label**) ทำให้:

1. **กรองงานได้แม่นยำ** เช่น `label:"priority: critical" label:"area: backend"` เพื่อดูเฉพาะบั๊กร้ายแรงในฝั่ง backend
2. **จัดกลุ่มในหน้า label list ได้เป็นระเบียบ** เพราะ label ที่ prefix เดียวกันจะเรียงติดกัน
3. **ป้องกันความสับสน** ระหว่าง label ที่มีความหมายใกล้เคียงกัน เช่น `bug` เฉย ๆ กับ `type: bug`

GitHub ยังรองรับฟีเจอร์ **mutually exclusive labels** (ผ่านการตั้งชื่อ prefix ร่วมกับสีเดียวกัน) ซึ่งเมื่อเลือก label ในกลุ่มเดียวกันตัวใหม่ ระบบจะถามว่าต้องการเอา label เดิมในกลุ่มเดียวกันออกหรือไม่ ช่วยให้ไม่มี Issue ไหนติด `priority: high` และ `priority: low` พร้อมกันโดยไม่ตั้งใจ

### การกรอง Issue/PR ด้วย Label ในช่องค้นหา

```
is:issue is:open label:bug
is:issue label:"priority: high",bug        # OR ระหว่างสอง label (คั่นด้วย comma)
is:pr label:bug label:"area: backend"      # AND ระหว่างสอง label (เขียนแยกกัน)
is:issue -label:wontfix                    # ไม่มี label wontfix
```

หมายเหตุสำคัญ: การเขียน label สองตัวคั่นด้วย comma ในพารามิเตอร์เดียว (`label:a,b`) หมายถึง **OR** ในขณะที่การเขียนแยกเป็นคนละ `label:` (`label:a label:b`) หมายถึง **AND** — ทีมจำนวนมากสับสนจุดนี้บ่อยมาก ควรจำให้แม่น

---

## Step 252: Default labels ของ GitHub และความหมายของแต่ละตัว

เมื่อคุณสร้าง repository ใหม่บน GitHub (ที่ไม่ได้เกิดจากการ fork) ระบบจะสร้าง **default label ให้อัตโนมัติ 9 ตัว** ทันที เพื่อให้เริ่มใช้งาน Issue ได้เลยโดยไม่ต้องตั้งค่าอะไรเพิ่ม

### ตาราง Default Labels ทั้ง 9 ตัว

| Label | สี (Hex) | ความหมาย/การใช้งาน |
|---|---|---|
| `bug` | `#d73a4a` (แดง) | บอกว่า Issue นี้คือรายงานข้อผิดพลาดที่พบในโค้ด/ระบบ |
| `documentation` | `#0075ca` (น้ำเงิน) | เกี่ยวข้องกับการปรับปรุง/เพิ่มเอกสารประกอบโปรเจกต์ (README, wiki, comment) |
| `duplicate` | `#cfd3d7` (เทาอ่อน) | Issue หรือ PR นี้ซ้ำกับอันที่มีอยู่แล้ว มักใช้คู่กับการปิด Issue พร้อมลิงก์ไปยัง Issue ต้นฉบับ |
| `enhancement` | `#a2eeef` (ฟ้าอ่อน) | คำขอฟีเจอร์ใหม่หรือการปรับปรุงของเดิมให้ดีขึ้น (ไม่ใช่บั๊ก) |
| `good first issue` | `#7057ff` (ม่วง) | งานที่เหมาะสำหรับผู้มีส่วนร่วมหน้าใหม่ (new contributor) — มักไม่ซับซ้อน มีคำอธิบายชัดเจน |
| `help wanted` | `#008672` (เขียวเข้ม) | ผู้ดูแลโปรเจกต์ต้องการให้ชุมชนภายนอกช่วยทำงานนี้เป็นพิเศษ |
| `invalid` | `#e4e669` (เหลือง) | Issue นี้ไม่ถูกต้อง/ไม่เกี่ยวข้อง/รายงานผิดที่ (เช่น ไม่ใช่บั๊กจริง แต่เป็นการใช้งานผิด) |
| `question` | `#d876e3` (ชมพู/ม่วง) | เป็นคำถามเกี่ยวกับการใช้งาน ไม่ใช่บั๊กหรือคำขอฟีเจอร์ |
| `wontfix` | `#ffffff` (ขาว) | ผู้ดูแลโปรเจกต์ตัดสินใจแล้วว่าจะไม่แก้ไข/ไม่ทำตามคำขอนี้ พร้อมเหตุผลกำกับ |

### ตัวอย่างการใช้งานจริงของแต่ละ default label

- **`bug`** — ผู้ใช้รายงานว่า "กดปุ่ม Submit แล้วหน้าเว็บค้าง" → ติด label `bug` ทันที
- **`documentation`** — มีคนเสนอว่า README ขาดตัวอย่างการติดตั้งบน Windows → ติด `documentation`
- **`duplicate`** — มีคนเปิด Issue เรื่องเดียวกับที่เคยมีคนรายงานไปแล้วเมื่อสัปดาห์ก่อน → ติด `duplicate` แล้ว comment ลิงก์ไปยัง Issue เดิม จากนั้นปิด
- **`enhancement`** — มีคนขอให้เพิ่มปุ่ม Dark Mode → ติด `enhancement`
- **`good first issue`** — งานแก้ typo ในเอกสาร หรือเพิ่ม unit test ง่าย ๆ ที่มี instruction ชัดเจน → ติด `good first issue` เพื่อดึงดูดผู้มีส่วนร่วมใหม่
- **`help wanted`** — โปรเจกต์ต้องการคนที่เชี่ยวชาญ i18n มาช่วยแปลภาษา แต่ทีมหลักไม่มีเวลา → ติด `help wanted`
- **`invalid`** — มีคนรายงานบั๊กที่จริง ๆ แล้วเกิดจากการตั้งค่าเครื่องตัวเอง ไม่ใช่บั๊กของโปรเจกต์ → ติด `invalid`
- **`question`** — มีคนถามว่า "ใช้ library นี้ร่วมกับ React 18 ได้ไหม" → ติด `question`
- **`wontfix`** — มีคนขอฟีเจอร์ที่ขัดกับทิศทางของโปรเจกต์ → ผู้ดูแลติด `wontfix` พร้อมอธิบายเหตุผล แล้วปิด Issue

### สิ่งสำคัญที่ต้องเข้าใจ: Default labels ไม่ใช่กฎตายตัว

Default labels เป็นเพียง **จุดเริ่มต้น** เท่านั้น คุณสามารถ:

- **ลบ** default label ที่ไม่ได้ใช้ (เช่นหลายทีมลบ `wontfix` หรือ `invalid` เพราะไม่ตรงกับ workflow ของตัวเอง)
- **แก้ไขสี/คำอธิบาย** ของ default label ให้ตรงกับ branding ของทีม
- **เพิ่ม** label ใหม่ตามความต้องการเฉพาะของโปรเจกต์ (ดัง Step 251)

โปรเจกต์ open source ขนาดใหญ่จำนวนมาก เช่น Kubernetes, React, VS Code มักมี **label ของตัวเองเป็นร้อยตัว** ที่ออกแบบมาเฉพาะสำหรับ workflow การ triage ที่ซับซ้อนของทีมนั้น ๆ ไม่ได้ใช้แค่ 9 ตัว default เลย

### เมื่อ fork repository — default labels จะเป็นอย่างไร

เมื่อคุณ fork repository ของคนอื่น **label ทั้งหมดของ repo ต้นทาง (รวมถึง custom label ที่เขาสร้างเอง) จะถูกคัดลอกมาด้วย** ไม่ใช่แค่ default 9 ตัว เพราะ label ถือเป็นส่วนหนึ่งของ metadata ของ repository ที่ fork ตามมาทั้งหมด

---

## Step 253: Milestones เชิงลึก — วางแผน release ด้วย milestone

### Milestone คืออะไร

**Milestone** คือ **จุดหมายปลายทางที่มีกำหนดเวลา** ใช้สำหรับจัดกลุ่ม Issue และ Pull Request ที่ต้องทำให้เสร็จร่วมกันเพื่อบรรลุเป้าหมายเดียวกัน เช่น การออก release เวอร์ชันหนึ่ง หรือการปิด sprint หนึ่งรอบ

ความแตกต่างสำคัญระหว่าง Label กับ Milestone:

| | Label | Milestone |
|---|---|---|
| จำนวนที่ติดได้ต่อ Issue | ได้หลายอัน | **ได้แค่อันเดียว** |
| มีวันครบกำหนด (due date) ไหม | ไม่มี | **มี** |
| ใช้ทำอะไรหลัก ๆ | จัดหมวดหมู่/กรอง | วางแผนและติดตาม release/sprint |
| มี progress bar ในตัวไหม | ไม่มี | **มี** (ดู Step 254) |

เพราะ Issue ผูกกับ Milestone ได้แค่อันเดียว Milestone จึงเหมาะกับการตอบคำถามว่า **"งานนี้จะเสร็จในรอบไหน"** ในขณะที่ Label เหมาะกับการตอบคำถามว่า **"งานนี้เป็นเรื่องอะไร"**

### วิธีสร้าง Milestone ผ่านหน้าเว็บ

1. ไปที่แท็บ **Issues** ของ repository
2. คลิก **Milestones**
3. คลิก **New milestone**
4. กรอกข้อมูล:
   - **Title** — ชื่อ milestone เช่น `v1.2.0`, `Sprint 24`, `Q3 2026 Release`
   - **Due date** — วันครบกำหนด (ไม่บังคับ แต่แนะนำให้ใส่เสมอ)
   - **Description** — รายละเอียดของ milestone นี้ เช่น เป้าหมายหลัก ขอบเขตของ release
5. คลิก **Create milestone**

URL ของหน้า milestones:

```
https://github.com/<owner>/<repo>/milestones
```

### วิธีสร้าง Milestone ผ่าน GitHub CLI

ที่น่าสังเกตคือ `gh` CLI **ไม่มีคำสั่ง `gh milestone` แบบสำเร็จรูปในตัว core** (ต่างจาก `gh label` และ `gh issue`) แต่สามารถจัดการผ่าน `gh api` ได้โดยตรง:

```bash
# สร้าง milestone ใหม่
gh api repos/<owner>/<repo>/milestones \
  -f title="v1.2.0" \
  -f description="Release รอบเดือนตุลาคม เน้นแก้บั๊กด้าน performance" \
  -f due_on="2026-10-31T00:00:00Z"

# ดูรายการ milestone ทั้งหมด
gh api repos/<owner>/<repo>/milestones

# แก้ไข milestone (ใช้ milestone number)
gh api repos/<owner>/<repo>/milestones/3 -X PATCH \
  -f state="closed"

# ผูก Issue เข้ากับ milestone
gh issue edit 42 --milestone "v1.2.0"
```

### วิธีสร้าง Milestone ผ่าน REST API

```bash
curl -X POST \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/<owner>/<repo>/milestones \
  -d '{
        "title": "v1.2.0",
        "state": "open",
        "description": "Release รอบเดือนตุลาคม เน้นแก้บั๊กด้าน performance",
        "due_on": "2026-10-31T00:00:00Z"
      }'
```

### การผูก Issue/PR เข้ากับ Milestone

ในหน้า Issue หรือ PR แต่ละใบ จะมีแถบด้านขวา (sidebar) แสดงส่วน **Milestone** ให้คลิกเลือกจาก dropdown ได้ทันที หรือขณะสร้าง Issue ใหม่ก็สามารถกำหนด milestone ได้เลยตั้งแต่ต้น

### การใช้งาน Milestone เพื่อวางแผน Release แบบเป็นระบบ

แนวทางที่ทีมมืออาชีพนิยมใช้:

1. **ตั้งชื่อ milestone ให้ตรงกับเวอร์ชันที่จะออกจริง** เช่น `v2.0.0`, `v2.0.1` เพื่อให้เชื่อมโยงกับ Release Notes ได้ทันที (เรื่อง Release จะเรียนละเอียดใน Part หลัง)
2. **ใส่ due date เสมอ** แม้จะเป็นแค่ estimate เพราะมันทำให้เห็น timeline ชัดเจนในหน้า milestone list (ถ้าเลย due date ไปแล้วแต่ยังไม่ปิด ระบบจะโชว์คำว่า "overdue" เป็นสีแดงเตือน)
3. **เขียน description อธิบายขอบเขต (scope)** ของ release นั้นให้ชัดเจน เช่น "Release นี้เน้นเฉพาะการแก้บั๊กที่กระทบผู้ใช้จำนวนมาก ไม่รวมฟีเจอร์ใหม่"
4. **ทยอยผูก Issue/PR ที่วางแผนจะทำเข้า milestone** ตั้งแต่ตอน backlog grooming/triage ไม่ใช่ผูกทีเดียวตอนใกล้ปล่อยจริง
5. **ปิด milestone ทันทีที่ release ออกจริง** เพื่อให้หน้า milestone list สะอาด ไม่มี milestone เก่าค้างอยู่เป็น open

### Milestone vs GitHub Projects — ใช้เมื่อไหร่

| สถานการณ์ | เครื่องมือที่เหมาะสม |
|---|---|
| ต้องการติดตามงานที่ต้องเสร็จภายในวันที่กำหนด แบบเรียบง่าย | **Milestone** |
| ต้องการ board แบบ Kanban ที่มีหลายคอลัมน์ (To do/In progress/Done) และปรับแต่งฟิลด์ได้เอง | **GitHub Projects** |
| ต้องการ roadmap ข้ามหลาย repository พร้อมกัน | **GitHub Projects** (organization-level) |
| ต้องการแค่กลุ่ม Issue สำหรับ release เวอร์ชันหนึ่งแบบง่าย ๆ | **Milestone** |

ในทางปฏิบัติ ทีมจำนวนมากใช้ **ทั้งสองอย่างร่วมกัน**: Milestone สำหรับผูกกับ release version อย่างเป็นทางการ และ GitHub Projects สำหรับบริหารจัดการงานประจำวันแบบ Kanban เราจะเรียนเรื่อง GitHub Projects แบบละเอียดใน Part ถัดไปของหลักสูตรนี้

---

## Step 254: ติดตามความคืบหน้า milestone ด้วย progress bar

### Progress Bar ของ Milestone คืออะไร

เมื่อคุณเปิดหน้า `https://github.com/<owner>/<repo>/milestones` GitHub จะแสดง **แถบความคืบหน้า (progress bar)** ให้กับ milestone แต่ละอันโดยอัตโนมัติ โดยไม่ต้องตั้งค่าอะไรเพิ่มเลย

หน้าตาของข้อมูลที่แสดงประกอบด้วย:

```
v1.2.0
██████████████████░░░░░░░░  72%
18 closed   7 open
Due by October 31, 2026 (in 12 days)
```

### สูตรการคำนวณเปอร์เซ็นต์

```
% ความคืบหน้า = (จำนวน Issue/PR ที่ปิดแล้ว) / (จำนวน Issue/PR ทั้งหมดใน milestone) × 100
```

ตัวอย่าง: ถ้า milestone หนึ่งมี Issue ทั้งหมด 25 ใบ ปิดไปแล้ว 18 ใบ ยังเปิดอยู่ 7 ใบ

```
18 / (18 + 7) × 100 = 72%
```

**ข้อควรระวังสำคัญ:** ตัวเลขนี้นับทั้ง **Issue และ Pull Request** รวมกัน ถ้าทีมของคุณผูก PR เข้ากับ milestone ด้วย (ไม่ใช่แค่ Issue) จำนวนที่นับจะรวมทั้งสองประเภท ซึ่งบางทีมอาจไม่ต้องการแบบนั้น จึงควรตกลงกันในทีมว่าจะผูกเฉพาะ Issue เข้ากับ milestone หรือจะผูกทั้งคู่

### การดูรายละเอียดผ่านหน้าเว็บ

คลิกที่ชื่อ milestone จากหน้า list จะพาไปยังหน้าที่แสดง Issue/PR ทั้งหมดที่ผูกกับ milestone นั้น พร้อมตัวกรองแยก **Open** และ **Closed** ให้ดูสถานะแต่ละใบได้ทันที และยังสามารถกรองซ้อนด้วย label ในหน้านี้ได้อีกชั้นหนึ่ง เช่น ดูเฉพาะ Issue ที่เป็น `bug` และยังไม่ปิดใน milestone `v1.2.0`:

```
is:open label:bug milestone:"v1.2.0"
```

### การดึงข้อมูลความคืบหน้าผ่าน API

REST API endpoint สำหรับดูรายละเอียด milestone หนึ่งอันจะคืนค่าฟิลด์ `open_issues` และ `closed_issues` มาให้โดยตรง ทำให้คำนวณ % ต่อได้ในสคริปต์ของคุณเอง:

```bash
gh api repos/<owner>/<repo>/milestones/3
```

ตัวอย่างผลลัพธ์ (บางส่วน):

```json
{
  "title": "v1.2.0",
  "state": "open",
  "open_issues": 7,
  "closed_issues": 18,
  "due_on": "2026-10-31T00:00:00Z"
}
```

นำไปคำนวณ % ต่อได้ง่าย ๆ เช่นด้วย `jq`:

```bash
gh api repos/<owner>/<repo>/milestones/3 | \
  jq '(.closed_issues / (.closed_issues + .open_issues) * 100 | floor)'
```

### การใช้ Progress Bar เพื่อวางแผน Release อย่างมีข้อมูล

ทีมที่ทำงานเป็น sprint หรือมี release ประจำ (weekly/biweekly) มักเปิดหน้า milestone list ในการประชุม standup หรือ sprint review เพื่อดูภาพรวมทั้งหมดพร้อมกันว่า:

- Milestone ไหนใกล้ครบกำหนดแต่ยังคืบหน้าไม่ถึงครึ่ง (สัญญาณเตือนว่าต้องปรับ scope หรือเลื่อนวัน)
- Milestone ไหนคืบหน้าเกือบ 100% แล้ว พร้อมปิดและออก release ได้จริง
- มี Issue ค้างอยู่ใน milestone ที่ผ่าน due date ไปแล้ว (overdue) ต้องตัดสินใจว่าจะย้ายไป milestone ถัดไปหรือเร่งทำให้เสร็จ

**เคล็ดลับ:** เมื่อใกล้ due date แต่ progress ยังต่ำมาก แนวทางที่ดีคือ "ย้าย scope ไม่ใช่ย้ายวัน" — คือย้าย Issue ที่ยังไม่เสร็จไปไว้ใน milestone ถัดไป แล้วปล่อย release ตามกำหนดเดิมด้วยเฉพาะสิ่งที่เสร็จจริง วิธีนี้รักษาความน่าเชื่อถือของ deadline ได้ดีกว่าการเลื่อนวันไปเรื่อย ๆ

---

## Step 255: Issue templates แบบ Markdown — สร้างฟอร์มมาตรฐานสำหรับรายงานปัญหา

### ทำไมต้องมี Issue Template

ถ้าไม่มี template ผู้ใช้ที่มาเปิด Issue มักจะเขียนสั้น ๆ ว่า "โปรแกรมพัง ช่วยแก้ที" โดยไม่บอกรายละเอียดที่จำเป็น ทำให้ maintainer ต้องเสียเวลาไปกลับถามข้อมูลเพิ่มหลายรอบ **Issue Template** แก้ปัญหานี้โดยกำหนด **โครงสร้างมาตรฐาน** ที่ผู้รายงานต้องกรอกตั้งแต่ต้น

### ตำแหน่งไฟล์ที่ GitHub อ่าน

GitHub จะมองหา issue template จากโฟลเดอร์นี้ในสาขา default branch ของ repository:

```
.github/ISSUE_TEMPLATE/
```

ไฟล์แบบ Markdown ในโฟลเดอร์นี้จะมีนามสกุล `.md` และต้องมี **YAML front matter** ที่ด้านบนของไฟล์เพื่อบอก metadata ให้ GitHub รู้จัก

### โครงสร้างของไฟล์ Issue Template แบบ Markdown

```markdown
---
name: 🐛 Bug report
about: รายงานข้อผิดพลาดที่พบในระบบ เพื่อช่วยให้เราแก้ไขได้เร็วขึ้น
title: "[BUG] "
labels: bug, needs-triage
assignees: ""
---

## คำอธิบายปัญหา

<!-- อธิบายสั้น ๆ ว่าเกิดอะไรขึ้น -->

## ขั้นตอนการทำให้เกิดปัญหาซ้ำ (Steps to Reproduce)

1. ไปที่หน้า '...'
2. คลิกที่ '...'
3. เลื่อนลงไปที่ '...'
4. พบข้อผิดพลาด

## พฤติกรรมที่คาดหวัง (Expected Behavior)

<!-- อธิบายว่าควรจะเกิดอะไรขึ้นถ้าไม่มีบั๊ก -->

## Screenshot (ถ้ามี)

<!-- แนบภาพหน้าจอเพื่อช่วยอธิบาย -->

## สภาพแวดล้อม (Environment)

- OS: [เช่น Windows 11, macOS 14]
- Browser: [เช่น Chrome 128]
- เวอร์ชันของโปรเจกต์: [เช่น v1.4.2]

## ข้อมูลเพิ่มเติม (Additional Context)

<!-- ข้อมูลอื่น ๆ ที่เกี่ยวข้อง -->
```

### ความหมายของแต่ละฟิลด์ใน YAML Front Matter

| ฟิลด์ | ความหมาย |
|---|---|
| `name` | ชื่อ template ที่จะแสดงในหน้าเลือก template ตอนเปิด Issue ใหม่ |
| `about` | คำอธิบายสั้น ๆ ใต้ชื่อ ช่วยให้ผู้ใช้เลือก template ที่ถูกต้อง |
| `title` | ข้อความที่ตั้งเป็นค่าเริ่มต้นในช่อง title ของ Issue (ผู้ใช้แก้ต่อได้) |
| `labels` | label ที่จะติดให้ Issue นี้อัตโนมัติทันทีที่สร้าง (คั่นด้วย comma ได้หลายอัน) |
| `assignees` | username ที่จะถูก assign ให้อัตโนมัติ (คั่นด้วย comma ได้หลายคน) |

### ตัวอย่างที่สอง: Feature Request Template

```markdown
---
name: ✨ Feature request
about: เสนอไอเดียฟีเจอร์ใหม่สำหรับโปรเจกต์นี้
title: "[FEATURE] "
labels: enhancement
assignees: ""
---

## ปัญหาที่ต้องการแก้ (Is your feature request related to a problem?)

<!-- อธิบายปัญหาที่ทำให้คุณอยากได้ฟีเจอร์นี้ -->

## แนวทางที่คุณอยากให้เป็น (Describe the solution you'd like)

<!-- อธิบายว่าฟีเจอร์นี้ควรทำงานอย่างไร -->

## ทางเลือกอื่นที่เคยพิจารณา (Describe alternatives you've considered)

<!-- มีวิธีอื่นที่แก้ปัญหาเดียวกันได้ไหม -->

## ข้อมูลเพิ่มเติม (Additional Context)

<!-- ภาพประกอบ ลิงก์อ้างอิง หรือบริบทอื่น ๆ -->
```

### ไฟล์ตั้งค่าเสริม: `config.yml`

นอกจากไฟล์ template แต่ละแบบแล้ว ยังสามารถสร้างไฟล์ `.github/ISSUE_TEMPLATE/config.yml` เพื่อควบคุมพฤติกรรมโดยรวมของหน้าเลือก template ได้:

```yaml
blank_issues_enabled: false
contact_links:
  - name: 💬 คำถามทั่วไป / ขอความช่วยเหลือ
    url: https://github.com/<owner>/<repo>/discussions
    about: กรุณาใช้ Discussions สำหรับคำถามทั่วไป ไม่ใช่ Issue
  - name: 📖 อ่านเอกสารก่อน
    url: https://docs.example.com
    about: ตรวจสอบเอกสารก่อนเปิด Issue ใหม่
```

- `blank_issues_enabled: false` จะปิดตัวเลือก "Open a blank issue" บังคับให้ผู้ใช้ต้องเลือก template ใดอันหนึ่งเท่านั้น
- `contact_links` ใช้เพิ่มลิงก์ทางลัดไปยังช่องทางอื่นที่ไม่ใช่ Issue เช่น Discussions, Discord, เอกสาร

### ข้อจำกัดของ Markdown-based Template

Template แบบ Markdown เป็นแค่ **ข้อความตั้งต้น** ที่ผู้ใช้สามารถลบทิ้งหรือแก้ไขส่วนไหนก็ได้อย่างอิสระ ไม่มีการบังคับว่าต้องกรอกฟิลด์ไหนจริง ๆ ก่อนกด submit — นี่คือข้อจำกัดที่ **Issue Forms แบบ YAML** (Step 256) ถูกสร้างขึ้นมาเพื่อแก้ไขโดยเฉพาะ

---

## Step 256: Issue forms แบบ YAML — ฟอร์มโครงสร้างที่มี dropdown, checkbox, required fields

### Issue Forms คืออะไร

**Issue Forms** เป็นรูปแบบ template ที่ใหม่กว่าและทรงพลังกว่า Markdown template โดยเขียนด้วยไฟล์ **YAML** (นามสกุล `.yml` หรือ `.yaml`) แทนที่จะเป็น `.md` ข้อดีสำคัญคือมันสร้าง **ฟอร์มกรอกข้อมูลแบบมีโครงสร้างจริง** (structured form) ที่มี input field, dropdown, checkbox และสามารถ **บังคับให้กรอกก่อน submit ได้จริง** (required field)

ไฟล์ยังคงวางอยู่ในโฟลเดอร์เดียวกัน:

```
.github/ISSUE_TEMPLATE/bug_report.yml
```

### โครงสร้างพื้นฐานของ Issue Form

```yaml
name: 🐛 Bug Report
description: รายงานข้อผิดพลาดที่พบในระบบ
title: "[Bug]: "
labels: ["bug", "needs-triage"]
assignees:
  - octocat
body:
  - type: markdown
    attributes:
      value: |
        ขอบคุณที่สละเวลารายงานปัญหา! กรุณากรอกข้อมูลด้านล่างให้ครบถ้วนที่สุด
```

### ประเภทของ Element ที่ใช้ได้ใน `body`

| Type | ใช้ทำอะไร |
|---|---|
| `markdown` | แสดงข้อความอธิบาย ไม่รับ input จากผู้ใช้ |
| `input` | ช่องกรอกข้อความบรรทัดเดียว |
| `textarea` | ช่องกรอกข้อความหลายบรรทัด (รองรับ syntax highlighting ถ้าใส่ `render`) |
| `dropdown` | เมนูให้เลือกตัวเลือกจากรายการ (เลือกได้อันเดียวหรือหลายอันตาม `multiple`) |
| `checkboxes` | รายการช่องติ๊กถูกได้หลายอัน |

### ตัวอย่างไฟล์เต็ม: `bug_report.yml`

```yaml
name: 🐛 Bug Report
description: รายงานข้อผิดพลาดที่พบในระบบ เพื่อช่วยให้เราแก้ไขได้เร็วขึ้น
title: "[Bug]: "
labels: ["bug", "needs-triage"]
body:
  - type: markdown
    attributes:
      value: |
        ขอบคุณที่สละเวลารายงานปัญหา กรุณาตรวจสอบก่อนว่าไม่มีคนรายงานเรื่องเดียวกันมาก่อน

  - type: input
    id: summary
    attributes:
      label: สรุปปัญหาโดยย่อ
      description: อธิบายปัญหาในหนึ่งประโยค
      placeholder: "เช่น: ปุ่ม Submit กดไม่ได้บน Safari"
    validations:
      required: true

  - type: textarea
    id: reproduce-steps
    attributes:
      label: ขั้นตอนการทำให้เกิดปัญหาซ้ำ
      description: อธิบายทีละขั้นตอนว่าต้องทำอย่างไรถึงจะเจอบั๊กนี้
      placeholder: |
        1. ไปที่หน้า...
        2. คลิกที่...
        3. พบข้อผิดพลาด...
      render: markdown
    validations:
      required: true

  - type: textarea
    id: expected-behavior
    attributes:
      label: พฤติกรรมที่คาดหวัง
      description: ควรเกิดอะไรขึ้นถ้าไม่มีบั๊กนี้
    validations:
      required: true

  - type: dropdown
    id: severity
    attributes:
      label: ความรุนแรงของปัญหา
      description: ประเมินผลกระทบของบั๊กนี้
      options:
        - Critical — ระบบใช้งานไม่ได้เลย
        - High — ฟีเจอร์หลักใช้งานไม่ได้
        - Medium — ใช้งานได้แต่มีปัญหา
        - Low — ปัญหาเล็กน้อย ไม่กระทบการใช้งานหลัก
    validations:
      required: true

  - type: dropdown
    id: browsers
    attributes:
      label: เบราว์เซอร์ที่พบปัญหา
      description: เลือกได้มากกว่าหนึ่งอัน
      multiple: true
      options:
        - Chrome
        - Firefox
        - Safari
        - Microsoft Edge
        - อื่น ๆ (ระบุในช่องข้อมูลเพิ่มเติม)
    validations:
      required: false

  - type: input
    id: version
    attributes:
      label: เวอร์ชันของโปรเจกต์
      placeholder: "เช่น v1.4.2"
    validations:
      required: true

  - type: checkboxes
    id: checklist
    attributes:
      label: ก่อนส่ง กรุณายืนยัน
      options:
        - label: ฉันได้ค้นหาแล้วว่าไม่มี Issue ที่รายงานเรื่องเดียวกันมาก่อน
          required: true
        - label: ฉันได้ลองใช้งานกับเวอร์ชันล่าสุดแล้วปัญหายังคงอยู่
          required: false

  - type: textarea
    id: additional-context
    attributes:
      label: ข้อมูลเพิ่มเติม
      description: แนบ screenshot, log, หรือบริบทอื่น ๆ ที่เกี่ยวข้อง
    validations:
      required: false
```

### ตัวอย่างไฟล์เต็ม: `feature_request.yml`

```yaml
name: ✨ Feature Request
description: เสนอไอเดียฟีเจอร์ใหม่สำหรับโปรเจกต์นี้
title: "[Feature]: "
labels: ["enhancement"]
body:
  - type: textarea
    id: problem
    attributes:
      label: ปัญหาที่ต้องการแก้
      description: ฟีเจอร์นี้แก้ปัญหาอะไรให้คุณ
    validations:
      required: true

  - type: textarea
    id: solution
    attributes:
      label: แนวทางที่คุณอยากให้เป็น
      description: อธิบายว่าฟีเจอร์นี้ควรทำงานอย่างไร
    validations:
      required: true

  - type: dropdown
    id: priority
    attributes:
      label: ความสำคัญในมุมมองของคุณ
      options:
        - สำคัญมาก — กระทบการทำงานประจำวัน
        - สำคัญปานกลาง — เพิ่มความสะดวกอย่างชัดเจน
        - เป็นแค่ไอเดีย — ลองเสนอดูเฉย ๆ
    validations:
      required: true

  - type: checkboxes
    id: contribution
    attributes:
      label: การมีส่วนร่วม
      options:
        - label: ฉันยินดีที่จะช่วย implement ฟีเจอร์นี้เอง
          required: false
```

### ไฟล์ `config.yml` ใช้ร่วมกับ Issue Forms ได้เหมือนเดิม

`config.yml` ทำงานร่วมกับทั้ง Markdown template และ YAML form ได้ในโฟลเดอร์เดียวกัน ไม่ต้องแยก และหากมีทั้งสองแบบปนกันในโฟลเดอร์เดียว GitHub จะแสดงทั้งหมดให้เลือกในหน้าเดียวกันตอนกด **New issue**

### ข้อดีของ Issue Forms เทียบกับ Markdown Template

| ประเด็น | Markdown Template | Issue Forms (YAML) |
|---|---|---|
| บังคับให้กรอกฟิลด์จริง ๆ | ทำไม่ได้ | **ทำได้** ผ่าน `validations: required: true` |
| มี dropdown/checkbox แท้จริง | ไม่มี (ต้องพิมพ์ markdown checkbox เอาเอง) | **มี** เป็น UI element จริง |
| ข้อมูลที่ได้มีโครงสร้างสม่ำเสมอ | ขึ้นกับผู้ใช้ว่าจะแก้ template แค่ไหน | สม่ำเสมอกว่ามาก เพราะฟอร์มบังคับโครงสร้าง |
| ความยืดหยุ่นในการเขียนอิสระ | สูง (แก้ markdown ได้ทุกจุด) | ต่ำกว่า (จำกัดตาม element ที่กำหนด) |
| เหมาะกับ | โปรเจกต์เล็ก/ต้องการความยืดหยุ่น | โปรเจกต์ที่ต้องการมาตรฐานสูง/ทีมขนาดใหญ่ |

**ข้อควรระวัง:** id ของแต่ละ element (เช่น `id: summary`) ต้องไม่ซ้ำกันภายในไฟล์เดียว และเมื่อ Issue Form ถูก submit เนื้อหาทั้งหมดจะถูกประกอบร่างกลายเป็น Issue body แบบ Markdown ธรรมดาโดยอัตโนมัติ (แปลง label แต่ละหัวข้อเป็น heading `###`) ทำให้ Issue ที่เกิดขึ้นยังคงอ่านและแก้ไขได้เหมือน Issue ปกติทุกประการหลังจากสร้างเสร็จแล้ว

---

## Step 257: Pull Request template

### PR Template คืออะไร

เช่นเดียวกับ Issue Template, **Pull Request Template** คือข้อความมาตรฐานที่ปรากฏล่วงหน้าในช่อง description ทุกครั้งที่มีคนเปิด Pull Request ใหม่ ช่วยให้ผู้ส่งโค้ดอธิบายสิ่งที่เปลี่ยนแปลง เหตุผล และวิธีทดสอบได้อย่างครบถ้วน ทำให้ผู้รีวิวทำงานได้เร็วและแม่นยำขึ้นมาก

### ตำแหน่งไฟล์: แบบเดียว (Single Template)

ถ้าต้องการ template เดียวสำหรับทุก PR ให้สร้างไฟล์ที่ตำแหน่งใดตำแหน่งหนึ่งต่อไปนี้ (GitHub รองรับทั้งสามที่ ให้เลือกที่เดียว):

```
.github/PULL_REQUEST_TEMPLATE.md
docs/PULL_REQUEST_TEMPLATE.md
PULL_REQUEST_TEMPLATE.md          (ที่ root ของ repo)
```

ตำแหน่งที่นิยมที่สุดคือ `.github/PULL_REQUEST_TEMPLATE.md` เพื่อให้ root ของ repo สะอาด

### ตัวอย่างเนื้อหา PR Template มาตรฐาน

```markdown
## คำอธิบายการเปลี่ยนแปลง (Description)

<!-- อธิบายว่า PR นี้เปลี่ยนแปลงอะไรบ้าง และทำไมถึงต้องเปลี่ยน -->

## ประเภทของการเปลี่ยนแปลง (Type of Change)

- [ ] 🐛 Bug fix (แก้ไขปัญหาโดยไม่กระทบฟีเจอร์อื่น)
- [ ] ✨ New feature (เพิ่มฟีเจอร์ใหม่)
- [ ] 💥 Breaking change (การเปลี่ยนแปลงที่กระทบการใช้งานเดิม)
- [ ] 📝 Documentation (แก้ไขเฉพาะเอกสาร)
- [ ] ♻️ Refactor (ปรับโครงสร้างโค้ดโดยพฤติกรรมไม่เปลี่ยน)

## Issue ที่เกี่ยวข้อง (Related Issue)

<!-- ใช้คำว่า Closes #123 เพื่อให้ Issue ปิดอัตโนมัติเมื่อ merge -->

Closes #

## วิธีทดสอบ (How Has This Been Tested?)

<!-- อธิบายว่าคุณทดสอบการเปลี่ยนแปลงนี้อย่างไร -->

- [ ] Unit test
- [ ] Manual test
- [ ] ยังไม่ได้ทดสอบ (กรุณาระบุเหตุผล)

## Checklist ก่อนขอรีวิว

- [ ] โค้ดผ่าน lint/format ตามมาตรฐานโปรเจกต์แล้ว
- [ ] เพิ่ม/แก้ไข test ที่เกี่ยวข้องแล้ว
- [ ] อัปเดตเอกสารที่เกี่ยวข้องแล้ว (ถ้ามี)
- [ ] ไม่มี console.log หรือโค้ด debug หลงเหลืออยู่
- [ ] ตรวจสอบแล้วว่าไม่ทำให้ CI/CD พัง

## Screenshot / วิดีโอประกอบ (ถ้ามีการเปลี่ยนแปลงหน้า UI)

<!-- แนบภาพก่อน-หลังเพื่อให้ผู้รีวิวเห็นภาพชัดเจน -->
```

### หลายแบบ (Multiple Templates)

หากทีมของคุณต้องการ PR template หลายแบบสำหรับสถานการณ์ต่างกัน (เช่น แบบสำหรับ feature, แบบสำหรับ hotfix, แบบสำหรับ release) ให้สร้างโฟลเดอร์:

```
.github/PULL_REQUEST_TEMPLATE/
├── feature.md
├── hotfix.md
└── release.md
```

**ข้อควรระวัง:** ต่างจาก Issue Template ตรงที่ GitHub **ไม่แสดงหน้าให้เลือก template อัตโนมัติ** ตอนเปิด PR ผู้ใช้ต้องระบุ template ที่ต้องการผ่าน **query parameter** ต่อท้าย URL ตอนสร้าง PR:

```
https://github.com/<owner>/<repo>/compare/main...feature-branch?template=hotfix.md
```

หรือระบุหลายไฟล์รวมกันในคำขอเดียวด้วย `&template=` ซ้ำได้ (บาง client รองรับ, แนะนำให้ทดสอบเอง) แนวทางที่นิยมกว่าคือใส่ลิงก์สำเร็จรูปไว้ใน CONTRIBUTING.md ให้ทีมคลิกใช้ตรง ๆ โดยไม่ต้องพิมพ์เอง

### การเพิ่มลิงก์ไปยัง Issue อัตโนมัติเมื่อ PR ถูก merge

คำสั่งพิเศษที่ GitHub รู้จักใน PR description (หรือ commit message) ที่ทำให้ Issue ปิดอัตโนมัติเมื่อ PR ถูก merge เข้า default branch มีดังนี้ (ใช้ได้ทั้งภาษาอังกฤษ, ไม่สนตัวพิมพ์เล็ก-ใหญ่):

```
close #123
closes #123
closed #123
fix #123
fixes #123
fixed #123
resolve #123
resolves #123
resolved #123
```

การใส่คำเหล่านี้ไว้ใน PR template ล่วงหน้าเป็น comment ช่วยเตือนให้ผู้ส่ง PR ไม่ลืมเชื่อมโยง Issue ทุกครั้ง

### PR Template ทำงานร่วมกับ CODEOWNERS และ Branch Protection ได้

PR Template เป็นเพียงเนื้อหาตั้งต้นในช่อง description เท่านั้น มันไม่ได้บังคับให้ผู้ใช้กรอกจริง (ต่างจาก Issue Forms) — หากต้องการบังคับ workflow เพิ่มเติม เช่น ต้องมีคนรีวิวก่อน merge, ต้องผ่าน CI ก่อน merge ต้องตั้งค่าที่ **Branch Protection Rules** และ **CODEOWNERS** ซึ่งเป็นคนละกลไกกัน (จะเรียนละเอียดใน Part ถัดไปของหลักสูตรนี้)

---

## Step 258: การจัดระเบียบ label หลาย repo ในระดับ organization (label sync)

### ปัญหาที่พบเมื่อองค์กรมีหลาย Repository

Label เป็นข้อมูลที่ผูกอยู่กับ **repository เดียวเท่านั้น** — ไม่มีฟีเจอร์ในตัวของ GitHub ที่ทำให้ label ถูกสร้าง/แก้ไข/ลบพร้อมกันในหลาย repository โดยอัตโนมัติ เมื่อองค์กรหนึ่งมี repository เป็นสิบเป็นร้อยตัว จึงมักเกิดปัญหา:

- แต่ละทีมตั้งชื่อ label ไม่เหมือนกัน บาง repo ใช้ `bug` บาง repo ใช้ `type: bug` บาง repo ใช้ `defect`
- สีของ label เดียวกันไม่ตรงกันระหว่าง repo ทำให้ dashboard ที่รวมข้อมูลจากหลาย repo (เช่น GitHub Projects แบบ organization-level) แสดงผลไม่สอดคล้องกัน
- เมื่อมีการเพิ่ม label มาตรฐานใหม่ ต้องไปสร้างเองทีละ repo ด้วยมือ ซึ่งไม่ scale

### แนวทางแก้ปัญหา: Label Sync

**Label Sync** คือแนวคิดการกำหนด **ชุด label มาตรฐาน (source of truth)** ไว้ที่จุดเดียว แล้วใช้เครื่องมือ/สคริปต์ทำให้ label ในทุก repository ตรงกันโดยอัตโนมัติ วิธีที่นิยมมี 2 แนวทางหลัก

#### แนวทางที่ 1: ใช้ GitHub Action พร้อมไฟล์ labels กลาง

สร้างไฟล์กำหนดมาตรฐาน label ไว้ใน repository กลาง เช่น `labels.yml`:

```yaml
- name: "type: bug"
  color: "d73a4a"
  description: "ข้อผิดพลาดที่พบในระบบ"
- name: "type: feature"
  color: "a2eeef"
  description: "คำขอฟีเจอร์ใหม่"
- name: "type: docs"
  color: "0075ca"
  description: "เกี่ยวข้องกับเอกสารประกอบ"
- name: "priority: critical"
  color: "b60205"
  description: "ต้องแก้ไขทันที กระทบผู้ใช้จำนวนมาก"
- name: "priority: high"
  color: "d93f0b"
  description: "ควรแก้ไขโดยเร็ว"
- name: "priority: medium"
  color: "fbca04"
  description: "แก้ไขตามคิวปกติ"
- name: "priority: low"
  color: "0e8a16"
  description: "ไม่เร่งด่วน"
- name: "status: needs-triage"
  color: "ededed"
  description: "ยังไม่ได้ประเมิน/จัดลำดับความสำคัญ"
```

จากนั้นสร้าง workflow ในแต่ละ repository (หรือใน `.github` repository พิเศษขององค์กรที่ workflow แชร์ร่วมกันได้) ที่ตำแหน่ง `.github/workflows/label-sync.yml`:

```yaml
name: Sync labels

on:
  push:
    branches: [main]
    paths:
      - labels.yml
  workflow_dispatch: {}

jobs:
  sync-labels:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Sync repository labels
        uses: micnncim/action-label-syncer@v1
        with:
          manifest: labels.yml
          token: ${{ secrets.GITHUB_TOKEN }}
```

Workflow นี้จะทำงานทุกครั้งที่มีการแก้ไข `labels.yml` และรันได้ด้วยมือผ่าน `workflow_dispatch` เมื่อไหร่ก็ได้ ทำให้ label ของ repo นั้นตรงกับ manifest กลางเสมอ

#### แนวทางที่ 2: ใช้เครื่องมือ CLI แบบสคริปต์ (เช่น `github-label-sync`)

สำหรับทีมที่ต้องการ sync label ไปยังหลาย repository พร้อมกันในคำสั่งเดียว สามารถใช้เครื่องมือ open source อย่าง `github-label-sync` (ติดตั้งผ่าน npm) ได้:

```bash
npm install --global github-label-sync

# sync label จากไฟล์ labels.json ไปยัง repo เป้าหมาย
github-label-sync \
  --access-token <TOKEN> \
  --labels labels.json \
  <owner>/<repo-1>

# รันซ้ำกับหลาย repo ผ่าน loop ใน shell script
for repo in repo-1 repo-2 repo-3; do
  github-label-sync --access-token <TOKEN> --labels labels.json "<owner>/$repo"
done
```

รูปแบบไฟล์ `labels.json` ของเครื่องมือนี้:

```json
[
  { "name": "type: bug", "color": "d73a4a", "description": "ข้อผิดพลาดที่พบในระบบ" },
  { "name": "type: feature", "color": "a2eeef", "description": "คำขอฟีเจอร์ใหม่" },
  { "name": "priority: critical", "color": "b60205", "description": "ต้องแก้ไขทันที" }
]
```

**ข้อควรระวัง:** เครื่องมือประเภทนี้โดยทั่วไปทำงานแบบ **destructive sync** คือจะ **ลบ label ที่มีอยู่แล้วในปลายทางแต่ไม่อยู่ใน manifest** ด้วย ควรทดสอบด้วยโหมด dry-run (ถ้ามี) ก่อนรันจริงกับ repository สำคัญเสมอ

### ข้อควรรู้: Community Health Files ไม่ครอบคลุมถึง Label

หลายคนเข้าใจผิดว่าไฟล์ `.github` repository พิเศษระดับ organization (ที่ใช้แชร์ CODEOWNERS, Issue Template, Contributing Guide เริ่มต้นให้ทุก repo ที่ไม่มีไฟล์ของตัวเอง — เรียกว่า **Default Community Health Files**) จะครอบคลุมถึง label ด้วย แต่ในความเป็นจริง **กลไกนี้ใช้ได้กับไฟล์เอกสารเท่านั้น เช่น README, CONTRIBUTING.md, CODE_OF_CONDUCT.md, ISSUE_TEMPLATE, PULL_REQUEST_TEMPLATE — ไม่ครอบคลุมถึง label** เพราะ label ไม่ใช่ไฟล์แต่เป็นข้อมูลในฐานข้อมูลของแต่ละ repository จึงจำเป็นต้อง sync ผ่าน API/Action/สคริปต์เสมอตามที่อธิบายไว้ข้างต้น

### สรุปทางเลือกในการ sync label ระดับ organization

| วิธี | เหมาะกับ | ข้อดี | ข้อควรระวัง |
|---|---|---|---|
| GitHub Action + manifest ต่อ repo | ทีมที่มี repo ไม่เยอะมาก ต้องการควบคุมทีละ repo | ตั้งค่าง่าย โปร่งใส ตรวจสอบผ่าน PR ได้ | ต้องคัดลอก workflow ไปทุก repo (หรือใช้ reusable workflow) |
| CLI sync แบบ loop สคริปต์ | องค์กรที่มี repo จำนวนมาก ต้องการ sync ทีเดียวทั้งหมด | รันครั้งเดียวจบ จัดการจากศูนย์กลางได้ | เสี่ยง destructive sync ถ้าไม่ทดสอบก่อน ต้องเก็บ token ที่มีสิทธิ์เข้าหลาย repo อย่างปลอดภัย |
| ทำมือทีละ repo | องค์กรเล็กมาก มี repo ไม่กี่ตัว | ไม่ต้องตั้งเครื่องมือเพิ่ม | ไม่ scale เมื่อ repo เพิ่มขึ้น เสี่ยง label ไม่ตรงกัน |

---

## Step 259: Best practices สำหรับตั้งชื่อและใช้ label/milestone ในทีมจริง

### หลักการตั้งชื่อ Label (Naming Convention)

1. **ใช้ตัวพิมพ์เล็กทั้งหมด (lowercase)** เพื่อความสม่ำเสมอ และป้องกันความสับสนเวลาค้นหา เช่น `bug` ไม่ใช่ `Bug` หรือ `BUG`
2. **ใช้ prefix แบบ namespace เมื่อมี label เกิน 10 ตัว** เช่น `type:`, `priority:`, `area:`, `status:` เพื่อให้จัดกลุ่มและกรองได้ง่าย
3. **หลีกเลี่ยงชื่อที่กำกวมหรือทับซ้อนความหมาย** เช่น ไม่ควรมีทั้ง `bug` และ `defect` พร้อมกันในระบบเดียว เพราะทีมจะสับสนว่าควรใช้อันไหน
4. **ตั้งชื่อสั้น กระชับ อ่านแล้วเข้าใจทันที** หลีกเลี่ยงประโยคยาว ๆ ในชื่อ label (รายละเอียดเพิ่มเติมให้ใส่ใน description แทน)
5. **จำกัดจำนวน label ทั้งหมดไม่ให้เยอะเกินไป** โดยทั่วไปแนะนำอยู่ที่ประมาณ **15–30 label ต่อ repository** ถ้าเกินกว่านี้มาก ผู้ใช้จะเลือกไม่ถูกและ label จะเริ่มไม่ถูกใช้งานจริง

### หลักการเลือกสีให้สื่อความหมาย (Color Convention)

ทีมที่ทำงานได้ผลดีมักกำหนด **กฎสีตายตัว** ไว้ล่วงหน้า ไม่ใช่เลือกสีตามใจแต่ละครั้ง ตัวอย่างกฎที่ใช้ได้จริง:

| กลุ่ม (namespace) | โทนสีที่ใช้ | เหตุผล |
|---|---|---|
| `type: *` | โทนฟ้า/น้ำเงิน | เป็นข้อมูลที่เป็นกลาง ไม่บอกความเร่งด่วน |
| `priority: critical` / `priority: high` | โทนแดง/ส้มเข้ม | สื่อถึงอันตราย/ความเร่งด่วนทันที |
| `priority: medium` | โทนเหลือง | เตือนแบบกลาง ๆ |
| `priority: low` | โทนเขียว | ไม่น่ากังวล |
| `status: blocked` | โทนแดงเข้ม/ดำ | สื่อว่าห้ามเดินหน้าต่อจนกว่าจะแก้ |
| `status: ready-for-review` | โทนเขียวสด | สื่อว่า "ไปต่อได้" |
| `good first issue` | โทนม่วง | แยกจากกลุ่มอื่นชัดเจน ดึงดูดสายตาผู้มีส่วนร่วมใหม่ |

การรักษาความสม่ำเสมอของสีทำให้สมาชิกทีมสามารถ "อ่านภาพรวมด้วยสายตา" ได้ทันทีจากหน้า Issue list โดยไม่ต้องอ่านชื่อ label ทีละตัว

### หลักการเขียน Description ของ Label

Description ควรตอบคำถามว่า **"เมื่อไหร่ที่ควรใช้ label นี้"** อย่างชัดเจน ไม่ใช่แค่แปลชื่อ label ซ้ำ เช่น:

- ไม่ดี: `bug` → "บั๊ก" (ซ้ำกับชื่อ ไม่ได้ให้ข้อมูลเพิ่ม)
- ดี: `bug` → "ใช้เมื่อพฤติกรรมจริงของระบบไม่ตรงกับพฤติกรรมที่ตั้งใจไว้ ไม่ใช่การขอฟีเจอร์ใหม่"

### ควรเขียนคู่มือ label ไว้ใน CONTRIBUTING.md เสมอ

โปรเจกต์ที่มีผู้มีส่วนร่วมจากภายนอก (external contributors) ควรมีตารางอธิบาย label ทั้งหมดไว้ในไฟล์ `CONTRIBUTING.md` เพื่อให้คนใหม่เข้าใจระบบได้ทันทีโดยไม่ต้องถามในแชท ตัวอย่างโครงสร้างที่ควรมี:

```markdown
## ระบบ Label ของโปรเจกต์นี้

เราใช้ label 3 มิติร่วมกัน:

- `type: *` — บอกประเภทของงาน
- `priority: *` — บอกความเร่งด่วน (กำหนดโดยทีมหลักเท่านั้น)
- `status: *` — บอกสถานะปัจจุบันของงาน

ผู้มีส่วนร่วมภายนอกสามารถติด `type: *` ได้เอง แต่ `priority: *` จะถูกกำหนดโดยทีมหลักหลัง triage เท่านั้น
```

### หลักการตั้งชื่อและใช้งาน Milestone

1. **ใช้ชื่อที่ตรงกับเวอร์ชันจริงที่จะปล่อย** เช่น `v2.3.0` เพื่อให้เชื่อมโยงกับ Release Notes/Changelog ได้ทันที ตาม [Semantic Versioning](https://semver.org)
2. **สำหรับทีมที่ทำงานแบบ sprint** ใช้รูปแบบ `Sprint <เลข> (<ช่วงวันที่>)` เช่น `Sprint 24 (2026-10-01 – 2026-10-14)` เพื่อให้เห็นกรอบเวลาชัดเจนแม้ไม่เปิดดู due date
3. **ใส่ due date เสมอแม้จะเป็นแค่ประมาณการ** เพราะมันคือสิ่งที่ทำให้ progress bar และสถานะ overdue มีความหมาย
4. **เขียน scope ของ milestone ไว้ใน description อย่างชัดเจน** เพื่อป้องกัน scope creep (งานเพิ่มขึ้นเรื่อย ๆ โดยไม่มีขอบเขต)
5. **ปิด milestone ทันทีหลังปล่อย release จริง** อย่าปล่อยให้ milestone เก่าค้างเป็นสถานะ open เพราะจะทำให้หน้า milestone list รกและสร้างความสับสน
6. **อย่าใช้ milestone แทน backlog ทั้งหมด** — milestone ควรมีเฉพาะงานที่ **ตกลงแล้วจริง ๆ ว่าจะทำในรอบนี้** ส่วนงานที่ยังไม่แน่ใจควรพักไว้แบบไม่ผูก milestone หรือใช้ label `status: backlog` แทน

### หลักการใช้ Label กับ Milestone ร่วมกันอย่างเสริมพลังกัน (ไม่ใช่แทนที่กัน)

ตารางสรุปการแบ่งหน้าที่ที่ชัดเจนระหว่างสองเครื่องมือ:

| คำถามที่ต้องการตอบ | ใช้เครื่องมือ |
|---|---|
| "นี่เป็นงานประเภทไหน" | Label (`type: *`) |
| "งานนี้สำคัญแค่ไหน" | Label (`priority: *`) |
| "งานนี้ตอนนี้อยู่ขั้นตอนไหน" | Label (`status: *`) |
| "งานนี้จะเสร็จในรอบไหน" | Milestone |
| "release รอบนี้คืบหน้าไปกี่เปอร์เซ็นต์แล้ว" | Milestone (progress bar) |

การผสมทั้งสองเข้าด้วยกันในการค้นหา ทำให้ตอบคำถามเชิงลึกได้ เช่น "มีบั๊กร้ายแรงกี่ตัวที่ยังไม่เสร็จใน release ถัดไป":

```
is:open is:issue label:bug label:"priority: critical" milestone:"v2.3.0"
```

---

## Step 260: แบบฝึกหัด — สร้างชุด label, milestone, issue template และ PR template แบบเต็มรูปแบบ

ถึงเวลาลงมือทำจริงแล้ว! ในแบบฝึกหัดนี้คุณจะสร้างระบบจัดระเบียบงานแบบครบวงจรให้กับ repository ฝึกฝนของคุณเอง (ใช้ repo ที่สร้างไว้จาก Part ก่อน ๆ หรือสร้างใหม่ก็ได้)

### เป้าหมายของแบบฝึกหัด

1. สร้าง label อย่างน้อย **8 ตัว** ครอบคลุมอย่างน้อย 2 มิติ (type และ priority)
2. สร้าง milestone **2 อัน** พร้อม due date และ description ที่ชัดเจน
3. สร้าง Issue Template อย่างน้อย 1 แบบ (แนะนำให้ลองทำทั้ง Markdown และ YAML Form เพื่อเปรียบเทียบ)
4. สร้าง Pull Request Template 1 ไฟล์

### ขั้นตอนที่ 1: เตรียม repository

```bash
mkdir -p ~/git-course/part-26-practice
cd ~/git-course/part-26-practice
git init
echo "# โปรเจกต์ฝึกฝน Part 26" > README.md
git add README.md
git commit -m "chore: initial commit"
```

จากนั้น push ขึ้น GitHub (สมมติว่าคุณสร้าง repository ชื่อ `label-milestone-practice` ไว้แล้วบนเว็บ):

```bash
git remote add origin https://github.com/<your-username>/label-milestone-practice.git
git branch -M main
git push -u origin main
```

### ขั้นตอนที่ 2: สร้าง Label อย่างน้อย 8 ตัวด้วย `gh` CLI

```bash
gh label create "type: bug" --color "d73a4a" --description "ข้อผิดพลาดที่พบในระบบ"
gh label create "type: feature" --color "a2eeef" --description "คำขอฟีเจอร์ใหม่"
gh label create "type: docs" --color "0075ca" --description "เกี่ยวข้องกับเอกสารประกอบ"
gh label create "priority: critical" --color "b60205" --description "ต้องแก้ไขทันที"
gh label create "priority: high" --color "d93f0b" --description "ควรแก้ไขโดยเร็ว"
gh label create "priority: medium" --color "fbca04" --description "แก้ไขตามคิวปกติ"
gh label create "priority: low" --color "0e8a16" --description "ไม่เร่งด่วน"
gh label create "status: needs-triage" --color "ededed" --description "ยังไม่ได้ประเมินความสำคัญ"
gh label create "good first issue" --color "7057ff" --description "เหมาะกับผู้มีส่วนร่วมหน้าใหม่"
```

ตรวจสอบว่าสร้างครบด้วยคำสั่ง:

```bash
gh label list
```

### ขั้นตอนที่ 3: สร้าง Milestone 2 อัน

```bash
gh api repos/<your-username>/label-milestone-practice/milestones \
  -f title="v0.1.0" \
  -f description="Release แรกสุด: ฟีเจอร์พื้นฐานครบตาม MVP" \
  -f due_on="2026-10-15T00:00:00Z"

gh api repos/<your-username>/label-milestone-practice/milestones \
  -f title="v0.2.0" \
  -f description="Release รอบสอง: เพิ่มฟีเจอร์ตามฟีดแบ็กจากผู้ใช้ + แก้บั๊กจาก v0.1.0" \
  -f due_on="2026-11-15T00:00:00Z"
```

ตรวจสอบผลลัพธ์:

```bash
gh api repos/<your-username>/label-milestone-practice/milestones
```

### ขั้นตอนที่ 4: สร้างโฟลเดอร์ template

```bash
mkdir -p .github/ISSUE_TEMPLATE
```

สร้างไฟล์ `.github/ISSUE_TEMPLATE/config.yml`:

```yaml
blank_issues_enabled: false
contact_links:
  - name: 💬 คำถามทั่วไป
    url: https://github.com/<your-username>/label-milestone-practice/discussions
    about: ใช้ Discussions สำหรับคำถามทั่วไป
```

สร้างไฟล์ `.github/ISSUE_TEMPLATE/bug_report.yml` (Issue Form แบบ YAML ตามตัวอย่างเต็มจาก Step 256 หรือย่อลงให้เหมาะกับโปรเจกต์ฝึกฝนก็ได้):

```yaml
name: 🐛 Bug Report
description: รายงานข้อผิดพลาดที่พบในระบบ
title: "[Bug]: "
labels: ["type: bug", "status: needs-triage"]
body:
  - type: input
    id: summary
    attributes:
      label: สรุปปัญหาโดยย่อ
    validations:
      required: true
  - type: textarea
    id: reproduce-steps
    attributes:
      label: ขั้นตอนการทำให้เกิดปัญหาซ้ำ
    validations:
      required: true
  - type: dropdown
    id: severity
    attributes:
      label: ความรุนแรง
      options:
        - Critical
        - High
        - Medium
        - Low
    validations:
      required: true
  - type: checkboxes
    id: checklist
    attributes:
      label: ยืนยันก่อนส่ง
      options:
        - label: ฉันได้ค้นหาแล้วว่าไม่มี Issue ซ้ำ
          required: true
```

สร้างไฟล์ `.github/ISSUE_TEMPLATE/feature_request.md` (ลองทำแบบ Markdown เพื่อเปรียบเทียบกับแบบ YAML):

```markdown
---
name: ✨ Feature request
about: เสนอไอเดียฟีเจอร์ใหม่
title: "[Feature] "
labels: "type: feature"
---

## ปัญหาที่ต้องการแก้

## แนวทางที่อยากให้เป็น

## ข้อมูลเพิ่มเติม
```

### ขั้นตอนที่ 5: สร้าง Pull Request Template

```bash
mkdir -p .github
```

สร้างไฟล์ `.github/PULL_REQUEST_TEMPLATE.md`:

```markdown
## คำอธิบายการเปลี่ยนแปลง

## ประเภทของการเปลี่ยนแปลง
- [ ] 🐛 Bug fix
- [ ] ✨ New feature
- [ ] 📝 Documentation

## Issue ที่เกี่ยวข้อง
Closes #

## Checklist ก่อนขอรีวิว
- [ ] โค้ดผ่าน lint แล้ว
- [ ] เพิ่ม/แก้ไข test ที่เกี่ยวข้องแล้ว
```

### ขั้นตอนที่ 6: commit และ push ทุกอย่าง

```bash
git add .github
git commit -m "chore: add issue templates and PR template"
git push
```

### ขั้นตอนที่ 7: ทดสอบผลลัพธ์จริงบนเว็บ

1. เปิด `https://github.com/<your-username>/label-milestone-practice/issues/new/choose` — ควรเห็นตัวเลือก template ทั้ง 2 แบบ (Bug Report แบบฟอร์ม และ Feature request แบบ markdown) พร้อมลิงก์ Discussions ที่ด้านล่าง และ **ไม่มี** ตัวเลือก "Open a blank issue" (เพราะตั้ง `blank_issues_enabled: false`)
2. ลองสร้าง Issue จริงจาก Bug Report แล้วตรวจสอบว่าฟิลด์ required บังคับกรอกจริง (ลองกด submit โดยเว้นช่องบังคับไว้ ระบบต้องเตือน)
3. เปิด compare/PR ใหม่ (สร้าง branch ทดลองแล้วลองเปิด PR) — ตรวจสอบว่า description แสดง PR Template ที่สร้างไว้อัตโนมัติ
4. เปิดหน้า `https://github.com/<your-username>/label-milestone-practice/milestones` — ตรวจสอบว่าเห็น milestone ทั้ง 2 อัน พร้อม due date ที่ถูกต้อง
5. ลองสร้าง Issue สัก 3–4 ใบ ผูกเข้ากับ milestone `v0.1.0` แล้วปิดไปสัก 2 ใบ กลับไปดูหน้า milestone อีกครั้งว่า progress bar อัปเดตถูกต้องตามสัดส่วนที่คำนวณไว้ใน Step 254

### Checklist สรุปแบบฝึกหัด

- [ ] สร้าง label อย่างน้อย 8 ตัว ครอบคลุมอย่างน้อย 2 มิติ (type/priority)
- [ ] สร้าง milestone 2 อัน พร้อม due date และ description
- [ ] สร้าง `.github/ISSUE_TEMPLATE/config.yml`
- [ ] สร้าง Issue Form แบบ YAML อย่างน้อย 1 ไฟล์ พร้อม required field และ dropdown
- [ ] สร้าง Issue Template แบบ Markdown อย่างน้อย 1 ไฟล์ (เพื่อเปรียบเทียบ)
- [ ] สร้าง `.github/PULL_REQUEST_TEMPLATE.md`
- [ ] ทดสอบสร้าง Issue จริงบนเว็บและยืนยันว่า required field ทำงานถูกต้อง
- [ ] ทดสอบเปิด PR จริงและยืนยันว่า template แสดงถูกต้อง
- [ ] ผูก Issue เข้ากับ milestone และตรวจสอบ progress bar

---

## สรุป Part 26

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Label** คือป้ายกำกับสีที่ใช้จัดหมวดหมู่และกรอง Issue/PR ได้หลายมิติพร้อมกัน สร้าง แก้ไข ลบได้ทั้งผ่าน UI, `gh` CLI และ REST API
2. GitHub สร้าง **default label 9 ตัว** ให้อัตโนมัติทุก repository ใหม่ (`bug`, `documentation`, `duplicate`, `enhancement`, `good first issue`, `help wanted`, `invalid`, `question`, `wontfix`) ซึ่งปรับแต่งหรือลบได้อิสระ
3. **Milestone** ใช้วางแผนและติดตาม release/sprint ที่มีกำหนดเวลา — Issue หนึ่งใบผูกได้แค่ milestone เดียว ต่างจาก label ที่ติดได้หลายอัน
4. หน้า milestone แสดง **progress bar** คำนวณจากสัดส่วน Issue/PR ที่ปิดแล้วเทียบกับทั้งหมดโดยอัตโนมัติ และดึงข้อมูลนี้ผ่าน API ได้ด้วยฟิลด์ `open_issues`/`closed_issues`
5. **Issue Template แบบ Markdown** (`.github/ISSUE_TEMPLATE/*.md`) ให้โครงสร้างตั้งต้นที่ยืดหยุ่น ในขณะที่ **Issue Forms แบบ YAML** (`.github/ISSUE_TEMPLATE/*.yml`) ให้ฟอร์มที่มี dropdown, checkbox และบังคับ required field ได้จริง
6. **PR Template** (`.github/PULL_REQUEST_TEMPLATE.md` หรือโฟลเดอร์ `PULL_REQUEST_TEMPLATE/` สำหรับหลายแบบ) ช่วยมาตรฐานการอธิบายการเปลี่ยนแปลงและ checklist ก่อนขอรีวิว
7. Label เป็นข้อมูลระดับ repository เท่านั้น การ sync label ให้ตรงกันทั้งองค์กรต้องอาศัย GitHub Action หรือเครื่องมือ CLI ภายนอก ไม่มีกลไก default community health file รองรับโดยตรง
8. Best practices ที่สำคัญคือการตั้งชื่อ label แบบมี namespace, เลือกสีให้สื่อความหมายสม่ำเสมอ, จำกัดจำนวน label ไม่ให้มากเกินไป, ตั้งชื่อ milestone ให้ตรงกับเวอร์ชันจริง และใช้ label กับ milestone เสริมพลังกันแทนที่จะใช้แทนกัน
9. เราได้ลงมือสร้างชุด label, milestone, issue template และ PR template แบบครบวงจรในแบบฝึกหัดจริงแล้ว

**ต่อไป:** [Part 27: GitHub Wiki และเอกสารประกอบโปรเจกต์](./part-027-github-wiki.md)
