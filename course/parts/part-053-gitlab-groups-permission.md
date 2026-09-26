# Part 53: GitLab Groups, Permission และการจัดการทีม

> **Step ในหลักสูตรนี้:** Step 521–530
> **เฟส:** 5 — เจาะลึก GitLab และ CI/CD เบื้องต้นของมัน
> **เป้าหมายของ Part นี้:** เข้าใจโครงสร้าง Group และ Subgroup ของ GitLab อย่างลึกซึ้ง รู้จัก Permission ทั้ง 5 ระดับและสิทธิ์ที่แท้จริงของแต่ละระดับ เชิญสมาชิกเข้าทีมได้อย่างถูกวิธี ตั้งค่าที่ระดับ Group ให้ไหลลงไปยังทุกโปรเจกต์ย่อยได้ ป้องกัน branch สำคัญด้วย Protected Branches และ Approval Rules ระดับ Group เข้าใจภาพรวมเรื่อง Billing/Seat และ Audit Events เบื้องต้น เพื่อให้สามารถวางโครงสร้างทีมและสิทธิ์ให้องค์กรจริงบน GitLab ได้อย่างมืออาชีพ

---

## สารบัญของ Part นี้

- Step 521: GitLab Group คืออะไร เทียบกับ GitHub Organization ที่เรียนใน Part 16
- Step 522: Nested groups (subgroups) — โครงสร้างลำดับชั้นหลายระดับ
- Step 523: Permission levels ของ GitLab ทั้ง 5 ระดับ (Guest, Reporter, Developer, Maintainer, Owner)
- Step 524: การเชิญสมาชิกเข้า group/project (invite by email, invite by username)
- Step 525: Group-level settings ที่ inherit ลงไปยัง project ย่อยทั้งหมด
- Step 526: Protected branches ใน GitLab เทียบกับ Branch Protection Rules ของ GitHub
- Step 527: Approval rules ระดับ group (บังคับใช้กับทุก project ในกลุ่ม)
- Step 528: Billing และ seat management เบื้องต้น
- Step 529: Audit events เบื้องต้น
- Step 530: แบบฝึกหัด — สร้างโครงสร้าง group/subgroup ให้องค์กรจำลองที่มี 3 ทีม

---

## Step 521: GitLab Group คืออะไร เทียบกับ GitHub Organization ที่เรียนใน Part 16

ใน Part 16 (Step 158) เราได้รู้จัก **GitHub Organization** แบบผิวเผินไปแล้วว่ามันคือบัญชีประเภทหนึ่งที่เป็นตัวแทนของทีมหรือบริษัท ใน GitLab แนวคิดที่ทำหน้าที่คล้ายกันเรียกว่า **Group** — แต่ Group ของ GitLab มีพลังและความยืดหยุ่นมากกว่า Organization ของ GitHub อย่างมีนัยสำคัญ

### 521.1 Group คืออะไร

**Group** คือ **namespace** (พื้นที่ชื่อ) ที่ทำหน้าที่เป็น "ภาชนะ" สำหรับเก็บ:

- **Project (repository)** หลาย ๆ ตัว
- **Subgroup** (กลุ่มย่อยที่ซ้อนอยู่ข้างใน — จะอธิบายละเอียดใน Step 522)
- **สมาชิก (Members)** พร้อมสิทธิ์ (Permission) ของแต่ละคน
- **การตั้งค่าที่ใช้ร่วมกัน** เช่น CI/CD Variables, Runners, Labels, Milestones, Wiki ระดับกลุ่ม

ทุก Project บน GitLab ต้องอยู่ภายใต้ namespace ใดนamespace หนึ่งเสมอ ไม่ว่าจะเป็น:

1. **Personal namespace** — namespace ส่วนตัวของผู้ใช้แต่ละคน (เช่น `gitlab.com/somchai/my-project`) เหมาะกับโปรเจกต์ส่วนตัวคนเดียว
2. **Group namespace** — namespace ของกลุ่ม (เช่น `gitlab.com/baan-software/backend-api`) เหมาะกับงานที่ทำเป็นทีม

### 521.2 เปรียบเทียบ Group (GitLab) กับ Organization (GitHub)

| คุณสมบัติ | GitHub Organization | GitLab Group |
|---|---|---|
| เก็บ repository/project หลายตัว | ได้ | ได้ |
| มีสมาชิกพร้อมกำหนดสิทธิ์ | ได้ (ผ่าน Team + Role) | ได้ (โดยตรงที่ตัว Group เลย) |
| สร้างกลุ่มย่อยซ้อนภายในได้ (nested) | **ไม่ได้** — มีแค่ "Team" สำหรับจัดกลุ่มคน ไม่ใช่จัดกลุ่ม repo | **ได้** — สร้าง Subgroup ซ้อนกันได้หลายชั้น (ดู Step 522) |
| ตั้งค่า CI/CD variable ระดับกลุ่มแล้วไหลลง repo ย่อยอัตโนมัติ | ทำได้บางส่วนผ่าน Organization secrets | ทำได้เต็มรูปแบบและไหลลงทุกชั้นของ subgroup |
| มี "หน้ารวม" แสดงโปรเจกต์ทั้งหมดในกลุ่ม | มี | มี พร้อม breadcrumb แสดงลำดับชั้น |
| Billing/ใบเรียกเก็บเงิน | ผูกกับ Organization | ผูกกับ **top-level group** เท่านั้น (subgroup ไม่มี billing แยก) |
| การมองเห็น (Visibility) | Public / Private ต่อ repo | Public / Internal / Private ต่อทั้ง Group และ Project แยกกัน |

### 521.3 ความแตกต่างเชิงโครงสร้างที่สำคัญที่สุด

จุดที่ต่างกันชัดเจนที่สุดคือ **GitHub Organization เป็นโครงสร้างแบบ "แบน" (flat)** — repository ทุกตัวอยู่ในระดับเดียวกันภายใต้ Organization เดียว ส่วน Team ใน GitHub ใช้จัดกลุ่ม "คน" ไม่ใช่จัดกลุ่ม "repo"

แต่ **GitLab Group เป็นโครงสร้างแบบ "ต้นไม้" (tree/hierarchical)** ที่แท้จริง — คุณสามารถสร้าง Group ซ้อนอยู่ใน Group ได้ไม่จำกัดชั้น (ในทางปฏิบัติ) ทำให้ URL ของโปรเจกต์หน้าตาเป็นแบบนี้ได้:

```
gitlab.com/baan-software/engineering/backend-team/payment-service
└─ Group        └─ Subgroup   └─ Subgroup       └─ Project
   ระดับบนสุด      ระดับ 2        ระดับ 3           (repository จริง)
```

นี่คือเหตุผลที่องค์กรขนาดใหญ่จำนวนมาก (โดยเฉพาะธนาคาร หน่วยงานรัฐ และบริษัทที่มีหลายแผนก) เลือกใช้ GitLab เพราะสามารถจำลองโครงสร้างองค์กรจริงลงไปในโครงสร้าง Group ได้ตรง ๆ

### 521.4 Group มีฟีเจอร์อะไรบ้าง (ภาพรวม)

| ฟีเจอร์ | คำอธิบายสั้น ๆ |
|---|---|
| Group members | รายชื่อสมาชิกพร้อม Role ที่ inherit ลงไปยังทุก project/subgroup ข้างใน |
| Group-level CI/CD Variables | ตัวแปรที่ใช้ร่วมกันได้ทุกโปรเจกต์ในกลุ่ม |
| Group Runners | Runner ที่แชร์ให้ทุกโปรเจกต์ในกลุ่มใช้รัน pipeline |
| Group Labels / Milestones | ใช้ทำ Issue Board ข้ามหลายโปรเจกต์ในกลุ่มเดียวกันได้ |
| Group Wiki | พื้นที่เอกสารกลางของกลุ่ม (แยกจาก Wiki ของแต่ละ project) |
| Group Container Registry | เก็บ Docker image ของทุกโปรเจกต์ในกลุ่มไว้ที่เดียว |
| Epics (Premium/Ultimate) | ติดตามงานภาพใหญ่ที่ครอบคลุมหลาย Issue ข้ามหลายโปรเจกต์ |
| Group Audit Events | ดูว่าใครเปลี่ยนแปลงอะไรในกลุ่มบ้าง (Step 529) |

Group จึงไม่ใช่แค่ "โฟลเดอร์เก็บ repo" แต่เป็น **หน่วยจัดการที่สมบูรณ์แบบหนึ่งของทีม** ที่ควบคุมได้ตั้งแต่คนจนถึงการตั้งค่าทางเทคนิคทั้งหมด

---

## Step 522: Nested groups (subgroups) — โครงสร้างลำดับชั้นหลายระดับ

### 522.1 Subgroup คืออะไร

**Subgroup** คือ Group ที่ถูกสร้างซ้อนอยู่ภายใน Group อื่นอีกที ตัวมันเองก็คือ Group ปกติทุกประการ (มีสมาชิก มีการตั้งค่า มี project ของตัวเองได้) เพียงแต่ตำแหน่งของมันอยู่ใต้ Group แม่ในลำดับชั้น

ตัวอย่างโครงสร้างจริงขององค์กรขนาดกลาง:

```
baan-software/                      ← Top-level Group (มี billing ที่นี่ที่เดียว)
├── engineering/                    ← Subgroup ระดับ 1
│   ├── backend-team/               ← Subgroup ระดับ 2
│   │   ├── payment-service         ← Project
│   │   └── auth-service            ← Project
│   ├── frontend-team/              ← Subgroup ระดับ 2
│   │   └── web-app                 ← Project
│   └── mobile-team/                ← Subgroup ระดับ 2
│       └── mobile-app              ← Project
├── qa/                             ← Subgroup ระดับ 1
│   └── automation-suite            ← Project
└── design/                         ← Subgroup ระดับ 1
    └── design-system               ← Project
```

### 522.2 จำกัดความลึกไหม

GitLab ไม่ได้ตั้ง "จำนวนชั้นสูงสุด" ของ subgroup ไว้ตายตัวในเอกสาร แต่ข้อจำกัดในทางปฏิบัติที่มีจริงคือ **ความยาวของ URL/full path รวมต้องไม่เกิน 255 ตัวอักษร** ดังนั้นยิ่งซ้อนลึกมากเท่าไหร่ ชื่อของแต่ละชั้นก็ยิ่งต้องสั้นลง

ในทางปฏิบัติ ทีมส่วนใหญ่ไม่ควรซ้อนเกิน **3–4 ชั้น** เพราะ:

- URL จะยาวและพิมพ์ยาก
- การไล่ดูสิทธิ์ (permission) ย้อนขึ้นไปแต่ละชั้นจะซับซ้อนเกินจำเป็น
- Breadcrumb บนหน้าเว็บจะรกและอ่านยาก

### 522.3 วิธีสร้าง Subgroup

ภายในหน้า Group แม่ ไปที่:

```
Group overview → Subgroups → New subgroup
```

จากนั้นตั้งชื่อ, เลือก visibility (ดู 522.4), และกด Create subgroup โดยปกติ **เฉพาะ Owner ของ Group แม่** เท่านั้นที่สร้าง subgroup ได้ (สามารถจำกัดเพิ่มเติมได้ผ่านการตั้งค่าระดับ instance/top-level group ว่าใครมีสิทธิ์สร้าง subgroup)

### 522.4 กฎเรื่อง Visibility ที่ต้องจำให้ขึ้นใจ

> **Subgroup จะมี visibility ที่ "เปิดกว้างกว่า" Group แม่ไม่ได้เด็ดขาด**

ตัวอย่าง:

| Group แม่ | Subgroup ตั้งเป็น Public ได้ไหม | Subgroup ตั้งเป็น Internal ได้ไหม | Subgroup ตั้งเป็น Private ได้ไหม |
|---|---|---|---|
| Public | ได้ | ได้ | ได้ |
| Internal | ไม่ได้ | ได้ | ได้ |
| Private | ไม่ได้ | ไม่ได้ | ได้เท่านั้น |

พูดง่าย ๆ คือ Visibility ไล่จากกว้างไปแคบคือ `Public > Internal > Private` และ subgroup ทำได้แค่ "เท่ากันหรือแคบกว่า" Group แม่เท่านั้น เพื่อป้องกันไม่ให้ข้อมูลรั่วออกไปโดยไม่ตั้งใจผ่านการสร้าง subgroup ที่เปิดกว้างเกินไป

### 522.5 กฎเรื่อง Permission Inheritance

> **สมาชิกที่ถูกเพิ่มใน Group ระดับบน จะได้สิทธิ์ระดับนั้น "ไหลลง" ไปยังทุก subgroup และทุก project ข้างใต้โดยอัตโนมัติ**

และมีกฎสำคัญอีกข้อ: **ในชั้นที่ลึกกว่า สามารถ "เพิ่ม" สิทธิ์ให้สมาชิกคนเดิมได้มากกว่าที่ inherit มา แต่จะ "ลด" สิทธิ์ให้ต่ำกว่าที่ inherit มาไม่ได้**

ตัวอย่าง: ถ้าคุณเป็น Developer ที่ระดับ Group `engineering` คุณจะเป็น Developer อัตโนมัติในทุก subgroup/project ข้างใต้ (`backend-team`, `frontend-team`, ฯลฯ) แต่ทีม `backend-team` สามารถตั้งให้คุณเป็น **Maintainer เฉพาะใน subgroup ของตัวเอง** ได้ (เพิ่มสิทธิ์ขึ้น) ทว่าทีม `frontend-team` **ไม่สามารถ** ตั้งให้คุณเหลือแค่ Guest ได้ (ลดสิทธิ์ลง) — ถ้าอยากให้สิทธิ์ต่ำกว่านี้จริง ๆ ต้องไปเอาคุณออกจาก Group แม่แล้วเพิ่มเข้าเฉพาะ subgroup ที่ต้องการแทน

### 522.6 เหตุผลที่องค์กรใช้ Subgroup

1. **จำลองโครงสร้างองค์กรจริง** — แผนก → ทีม → โปรเจกต์
2. **แยกการตั้งค่าเฉพาะทีม** — แต่ละทีมมี Runner, Variable, Label ของตัวเองได้โดยไม่ปนกัน
3. **มอบอำนาจการดูแล (delegate) ให้หัวหน้าทีมย่อย** — ตั้ง Team Lead เป็น Owner ของ subgroup ตัวเอง โดยไม่ต้องให้เป็น Owner ของทั้งบริษัท
4. **Billing รวมศูนย์ที่เดียว** — ไม่ว่าจะมี subgroup กี่ชั้น การคิดเงิน (seat) จะคิดที่ **top-level group เพียงจุดเดียวเสมอ** (รายละเอียดใน Step 528)

---

## Step 523: Permission levels ของ GitLab ทั้ง 5 ระดับ

นี่คือหัวใจของการจัดการทีมบน GitLab — ต้องเข้าใจให้แม่นเพราะจะใช้ตัดสินใจทุกครั้งที่เพิ่มสมาชิกใหม่ GitLab มี Role หลักที่ใช้งานบ่อยที่สุด 5 ระดับ เรียงจากต่ำไปสูง:

```
Guest  →  Reporter  →  Developer  →  Maintainer  →  Owner
(ต่ำสุด)                                              (สูงสุด)
```

แต่ละระดับ **ไม่ได้แทนที่ระดับก่อนหน้า แต่ "รวม" สิทธิ์ของระดับที่ต่ำกว่าไว้ทั้งหมด** แล้วเพิ่มสิทธิ์ใหม่เข้าไปอีก

### 523.1 Guest

Role ต่ำสุด เหมาะกับคนนอกทีม เช่น ลูกค้า, ผู้บริหารที่อยากติดตามความคืบหน้า, ผู้ตรวจสอบภายนอก

**ทำได้:**
- ดู Issue และ Merge Request ที่เปิดอยู่ในโปรเจกต์ (ถ้าโปรเจกต์เป็น private ต้องถูกเชิญก่อน)
- เขียนคอมเมนต์ใน Issue
- ดู Wiki, ดู Milestone
- สร้าง Issue ใหม่ได้

**ทำไม่ได้:**
- **ดูซอร์สโค้ด (repository) ไม่ได้** — clone/pull ไม่ได้เลย
- ดู pipeline หรือ CI/CD job log ไม่ได้ (เพราะ log อาจมีข้อมูลลับหลุดออกมา)
- Push code ไม่ได้

> Guest เหมาะกับคนที่ต้อง "ติดตามงาน" แต่ไม่ควรเห็นโค้ดจริง

### 523.2 Reporter

**ทำได้เพิ่มจาก Guest:**
- **อ่านซอร์สโค้ดทั้งหมด** (clone, pull, ดูไฟล์ผ่านเว็บได้)
- ดู pipeline และผลการรัน CI/CD รวมถึงดาวน์โหลด artifact
- ดู Merge Request แบบละเอียด (ดู diff, ดูสถานะ)
- สร้างและจัดการ Label, Milestone (ตามการตั้งค่าโปรเจกต์)

**ทำไม่ได้:**
- Push โค้ดเข้า repository ไม่ได้ (ไม่ว่า branch ไหน)
- สร้างหรือ merge Merge Request ไม่ได้

> Reporter เหมาะกับ QA, PM, หรือคนที่ต้อง "อ่าน" ทุกอย่างแต่ไม่ต้อง "เขียน" โค้ด

### 523.3 Developer

Role ที่นักพัฒนาส่วนใหญ่ในทีมควรได้รับ

**ทำได้เพิ่มจาก Reporter:**
- Push โค้ดเข้า branch ที่ **ไม่ได้ถูก protect** ได้
- สร้าง branch ใหม่ได้
- สร้างและแก้ไข Merge Request ได้ (ยกเว้นการ merge เข้า branch ที่ protect ไว้แบบเข้มงวด)
- Trigger pipeline ได้ (รวมถึงกด run manual job ตามสิทธิ์ที่ตั้งไว้)
- จัดการ Issue Board, จัดการ Label/Milestone เต็มรูปแบบ
- เพิ่ม/ลบ deploy key ระดับที่จำกัด (ขึ้นกับการตั้งค่า)

**ทำไม่ได้:**
- Push หรือ merge เข้า protected branch ได้ **เฉพาะถ้าถูกอนุญาตในการตั้งค่า Protected Branches** (ค่า default หลายกรณีอนุญาตให้ Developer merge ได้ แต่ push ตรง ๆ มักสงวนไว้ให้ Maintainer)
- แก้ไขการตั้งค่าโปรเจกต์ เช่น CI/CD Variables, Webhook, Protected Branch rules ไม่ได้
- เชิญหรือลบสมาชิกไม่ได้
- ลบโปรเจกต์ไม่ได้

### 523.4 Maintainer

Role ของหัวหน้าทีม/Tech Lead ที่ดูแลโปรเจกต์ในระดับเทคนิค

**ทำได้เพิ่มจาก Developer:**
- Push และ merge เข้า protected branch ได้ (ตามกฎที่ตั้งไว้)
- สร้าง/ลบ branch และ tag ที่ถูก protect ได้ (รวมถึง force push ถ้าเปิดอนุญาต)
- จัดการ **Protected Branches, Protected Tags** ทั้งหมด
- จัดการ **CI/CD Variables, Runners** ระดับโปรเจกต์
- แก้ไขการตั้งค่าโปรเจกต์ทั่วไป (ชื่อ, คำอธิบาย, merge method, webhook)
- เพิ่ม/ลบสมาชิกได้ **เฉพาะในระดับ Role ที่ต่ำกว่าหรือเท่ากับ Maintainer** (เพิ่มคนเป็น Owner ไม่ได้)
- จัดการ Container Registry ของโปรเจกต์
- ลบ pipeline, ยกเลิก job, retry job ได้ทั้งหมด

**ทำไม่ได้:**
- ลบโปรเจกต์ทั้งหมดไม่ได้
- ย้าย (transfer) โปรเจกต์ไปยัง namespace อื่นไม่ได้
- เปลี่ยนแปลง visibility ของโปรเจกต์จาก private เป็น public (หรือกลับกัน) ไม่ได้ในหลายกรณี — ทำได้เฉพาะ Owner
- ลบ Group ไม่ได้

### 523.5 Owner

Role สูงสุด มีสิทธิ์ทำได้ทุกอย่าง

**ทำได้เพิ่มจาก Maintainer:**
- ลบโปรเจกต์หรือลบ Group ได้ถาวร
- ย้าย/transfer โปรเจกต์ระหว่าง namespace ได้
- เปลี่ยน visibility ของ Group/Project ได้อย่างเต็มที่
- เพิ่ม/ลบสมาชิกได้ **ทุก Role รวมถึง Owner คนอื่น**
- จัดการ Billing และ Seat (เฉพาะ Owner ของ **top-level group** เท่านั้น)
- จัดการ SSO/SAML, Compliance Framework (Premium/Ultimate)
- ล็อกหรือปลดล็อก approval rule ระดับ group ได้ (Step 527)
- ลบ subgroup ที่อยู่ข้างใต้ได้

> **ข้อควรระวังสำคัญ:** Owner ของ Group จะเป็น Owner โดยอัตโนมัติในทุก subgroup และ project ข้างใต้ทั้งหมด ดังนั้นควรให้ตำแหน่งนี้กับคนที่ไว้ใจได้จริง ๆ เท่านั้น — โดยทั่วไปแนะนำให้มี Owner จำนวนน้อยที่สุดเท่าที่จำเป็น (2–3 คน) ไม่ใช่ทุกคนในทีม

### 523.6 ตารางสรุปเปรียบเทียบทั้ง 5 ระดับ

| ความสามารถ | Guest | Reporter | Developer | Maintainer | Owner |
|---|:---:|:---:|:---:|:---:|:---:|
| ดู Issue / คอมเมนต์ | ✅ | ✅ | ✅ | ✅ | ✅ |
| อ่านซอร์สโค้ด (clone/pull) | ❌ | ✅ | ✅ | ✅ | ✅ |
| ดู CI/CD pipeline & log | ❌ | ✅ | ✅ | ✅ | ✅ |
| Push เข้า branch ปกติ | ❌ | ❌ | ✅ | ✅ | ✅ |
| สร้าง/merge Merge Request | ❌ | ❌ | ✅ | ✅ | ✅ |
| Push/merge เข้า protected branch | ❌ | ❌ | ตามกฎที่ตั้ง | ✅ | ✅ |
| จัดการ Protected Branches/Tags | ❌ | ❌ | ❌ | ✅ | ✅ |
| จัดการ CI/CD Variables | ❌ | ❌ | ❌ | ✅ | ✅ |
| เพิ่ม/ลบสมาชิก | ❌ | ❌ | ❌ | ได้ถึงระดับ Maintainer | ได้ทุกระดับ |
| ลบโปรเจกต์/Group | ❌ | ❌ | ❌ | ❌ | ✅ |
| จัดการ Billing | ❌ | ❌ | ❌ | ❌ | ✅ (top-level เท่านั้น) |

> **หมายเหตุ:** นอกจาก 5 ระดับนี้ GitLab ยังมี **No access** (ไม่มีสิทธิ์ใด ๆ) และ **Minimal Access** (เห็นแค่ชื่อ Group แต่เข้าถึงเนื้อหาข้างในไม่ได้เลย ใช้เฉพาะระดับ Group บนแผน Premium/Ultimate) แต่ 5 ระดับหลักที่ใช้งานจริงในทีมประจำวันคือ Guest ถึง Owner ตามที่อธิบายไปข้างต้น

---

## Step 524: การเชิญสมาชิกเข้า group/project

### 524.1 เชิญเข้า Group

ไปที่ Group ที่ต้องการ แล้วเลือก:

```
Group → Manage → Members → Invite members
```

หน้าต่างเชิญสมาชิกให้เลือกวิธี **2 แบบหลัก**:

**วิธีที่ 1: Invite by username (สำหรับคนที่มีบัญชี GitLab อยู่แล้ว)**

- พิมพ์ username หรือชื่อของผู้ใช้ ระบบจะค้นหาและแสดงรายชื่อให้เลือก (autocomplete)
- เลือก **Role** ที่ต้องการมอบให้ (Guest / Reporter / Developer / Maintainer / Owner)
- (ทางเลือก) ตั้ง **Access expiration date** — วันที่สิทธิ์นี้จะหมดอายุอัตโนมัติ เหมาะกับพนักงานสัญญาจ้างชั่วคราวหรือ contractor
- กด **Invite** — ผู้ใช้จะได้รับสิทธิ์ทันที (ไม่ต้องกดยอมรับก็เข้าถึงได้เลย เพราะมีบัญชีอยู่แล้ว แต่จะได้รับการแจ้งเตือน)

**วิธีที่ 2: Invite by email (สำหรับคนที่ยังไม่มีบัญชี GitLab)**

- พิมพ์อีเมลของคนที่จะเชิญ (พิมพ์ได้หลายอีเมลคั่นด้วยจุลภาคในครั้งเดียว)
- เลือก Role เหมือนกัน
- กด Invite — GitLab จะส่งอีเมลเชิญไปยังที่อยู่นั้น พร้อมลิงก์ให้สมัครบัญชี GitLab (ถ้ายังไม่มี) หรือเข้าสู่ระบบ (ถ้ามีอยู่แล้วแต่ใช้อีเมลอื่นแสดง) แล้วกดยอมรับคำเชิญ
- ก่อนถูกกดรับ สถานะสมาชิกจะขึ้นเป็น **"Invited" (รอตอบรับ)** ในรายชื่อสมาชิก สามารถกด **Resend invite** หรือ **Revoke invite** (ยกเลิกคำเชิญ) ได้ตลอดเวลาที่ยังไม่ถูกกดรับ

### 524.2 เชิญเข้า Project โดยตรง

ทำแบบเดียวกันแต่ในระดับโปรเจกต์:

```
Project → Manage → Members → Invite members
```

สิ่งที่ต่างจากการเชิญเข้า Group คือ **สิทธิ์ที่ได้จะจำกัดเฉพาะโปรเจกต์นั้นเพียงตัวเดียว** ไม่ไหลไปยังโปรเจกต์อื่นในกลุ่มเดียวกัน เหมาะกับกรณีที่มีคนนอกทีมหลักต้องเข้ามาช่วยงานเฉพาะโปรเจกต์ใดโปรเจกต์หนึ่งเท่านั้น เช่น freelance developer ที่รับงานเฉพาะ repo เดียว

### 524.3 Direct Member vs Inherited Member

ในหน้า Members ของ Project จะเห็นคอลัมน์ **Source** บอกว่าสมาชิกแต่ละคนได้สิทธิ์มาจากไหน:

| ประเภท | ความหมาย |
|---|---|
| **Direct member** | ถูกเพิ่มเข้ามาที่โปรเจกต์นี้โดยตรง |
| **Inherited member** | ได้สิทธิ์มาจาก Group หรือ Subgroup ระดับบนโดยอัตโนมัติ (บอกด้วยว่ามาจาก Group ไหน) |

การเข้าใจความแตกต่างนี้สำคัญมากตอนแก้ปัญหาสิทธิ์ เช่น ถ้าอยากลดสิทธิ์คนคนหนึ่งในโปรเจกต์เดียว แต่เขาเป็น Inherited member มาจาก Group จะลดสิทธิ์ที่หน้า Project ไม่ได้เลย (ตามกฎ Step 522.5) ต้องไปจัดการที่ต้นทางคือ Group แทน

### 524.4 การขอสิทธิ์เข้าถึง (Request Access)

ถ้า Group หรือ Project เป็น **Private** และเปิดฟีเจอร์ "Allow users to request access" ไว้ ผู้ใช้ที่ไม่ใช่สมาชิกจะเห็นปุ่ม **"Request Access"** แทนที่จะเข้าไม่ได้เลย เมื่อกดขอ คำขอจะไปแจ้งเตือน Owner/Maintainer ให้กด **Approve** (พร้อมเลือก Role ที่จะให้) หรือ **Deny**

### 524.5 ข้อควรระวังเรื่อง Seat

การเชิญสมาชิกเข้า Group หรือ Project (ยกเว้น Role บางแบบบนบางแผน) จะ **นับเป็น 1 seat ของ Group** ทันทีที่คำเชิญถูกตอบรับ ซึ่งมีผลต่อค่าใช้จ่ายถ้าใช้แผนเสียเงิน (รายละเอียดใน Step 528) ดังนั้นก่อนเชิญควรพิจารณาว่าจำเป็นต้องให้สิทธิ์ระดับ Group หรือแค่ระดับ Project ก็เพียงพอแล้ว

---

## Step 525: Group-level settings ที่ inherit ลงไปยัง project ย่อยทั้งหมด

หนึ่งในจุดแข็งที่สุดของ GitLab Group คือความสามารถในการตั้งค่า "ครั้งเดียว ใช้ได้ทุกโปรเจกต์ในกลุ่ม" ลดการตั้งค่าซ้ำซ้อนและลดความเสี่ยงที่แต่ละโปรเจกต์จะตั้งค่าไม่ตรงกัน

### 525.1 Group-level CI/CD Variables

ที่ `Group → Settings → CI/CD → Variables` สามารถตั้งตัวแปร เช่น `DOCKER_REGISTRY_PASSWORD`, `DEPLOY_TOKEN`, `AWS_ACCESS_KEY_ID` ไว้ที่ Group เพียงครั้งเดียว แล้วทุก Project (รวมถึง Project ใน Subgroup ทุกชั้นข้างใต้) จะเห็นและใช้ตัวแปรนี้ใน pipeline ของตัวเองได้ทันที โดยไม่ต้องตั้งซ้ำทีละ project

คุณสมบัติของตัวแปรที่ตั้งได้เหมือนระดับ project:

- **Protect variable** — ตัวแปรนี้จะถูกส่งเข้า pipeline เฉพาะที่รันบน protected branch หรือ protected tag เท่านั้น ป้องกันไม่ให้ branch ทดลองทั่วไปเข้าถึงค่าลับ
- **Mask variable** — ค่าของตัวแปรจะถูกซ่อน (แสดงเป็น `[MASKED]`) ใน job log อัตโนมัติ ป้องกันความลับหลุดผ่านการ echo หรือ log

**กฎการ Override:** ถ้า Project ใดตั้งตัวแปรชื่อเดียวกันไว้ที่ระดับ Project เอง **ค่าที่ Project ตั้งจะชนะ (override) ค่าที่มาจาก Group เสมอ** ทำให้แต่ละโปรเจกต์ยังปรับแต่งค่าเฉพาะของตัวเองได้เมื่อจำเป็น

### 525.2 Group File Templates (Premium/Ultimate)

องค์กรที่ต้องการให้ทุกโปรเจกต์ใช้ **Issue template, Merge Request description template, หรือแม้แต่ license/gitignore template** แบบเดียวกันทั้งหมด สามารถกำหนด "โปรเจกต์ต้นแบบ" ไว้หนึ่งตัวที่:

```
Group → Settings → General → Templates → Template repository
```

จากนั้นไฟล์ template ที่เก็บไว้ในโปรเจกต์ต้นแบบนั้น (เช่นในโฟลเดอร์ `.gitlab/issue_templates/`) จะปรากฏเป็นตัวเลือกให้ใช้ได้ในทุกโปรเจกต์ของ Group โดยอัตโนมัติ

### 525.3 Group Runners

Runner ที่ลงทะเบียนไว้ในระดับ Group (`Group → Settings → CI/CD → Runners`) จะถูกแชร์ให้ทุก Project ในกลุ่มใช้รัน pipeline ร่วมกันได้ ไม่ต้องติดตั้ง Runner แยกทีละโปรเจกต์ ประหยัดทรัพยากรและง่ายต่อการดูแล

### 525.4 Group Labels และ Milestones

Label และ Milestone ที่สร้างไว้ระดับ Group จะใช้งานร่วมกันได้ในทุกโปรเจกต์ของกลุ่ม ทำให้สร้าง **Group Issue Board** ที่รวม Issue จากหลายโปรเจกต์มาดูในบอร์ดเดียวได้ — มีประโยชน์มากเมื่อฟีเจอร์หนึ่งของสินค้าต้องใช้การเปลี่ยนแปลงจากหลายโปรเจกต์พร้อมกัน (เช่น ต้องแก้ทั้ง backend และ frontend)

### 525.5 Default Branch Protection ระดับ Group

ที่ `Group → Settings → Repository → Default branch protection` สามารถกำหนดว่าเมื่อสร้าง Project ใหม่ในกลุ่มนี้ branch หลัก (default branch) จะถูกป้องกันในระดับใดโดยอัตโนมัติทันทีที่สร้างเสร็จ เช่น "Fully protected" (Developer ห้าม push ตรง ต้องผ่าน Merge Request เท่านั้น) ทำให้ไม่มีโปรเจกต์ใหม่หลุดออกมาโดยไม่มีการป้องกันเลย

### 525.6 Push Rules ระดับ Group (Premium/Ultimate)

กำหนดกฎการ push แบบละเอียด เช่น บังคับรูปแบบข้อความ commit ต้องตรงกับ regex ที่กำหนด (เช่นต้องขึ้นต้นด้วยเลข Ticket), ห้าม force push, ห้ามไฟล์ที่มีนามสกุลอันตราย, บังคับให้อีเมลผู้ commit ต้องตรงกับโดเมนบริษัท — ตั้งครั้งเดียวที่ Group แล้วบังคับใช้กับทุก Project ข้างใน

### 525.7 ตารางสรุปการ inherit

| การตั้งค่าระดับ Group | ไหลลง Subgroup | ไหลลง Project | Override ที่ระดับล่างได้ไหม |
|---|:---:|:---:|---|
| CI/CD Variables | ✅ | ✅ | ได้ (ตั้งชื่อเดียวกันที่ project จะชนะ) |
| Members/Permission | ✅ | ✅ | เพิ่มสิทธิ์ได้ ลดสิทธิ์ไม่ได้ |
| Default branch protection | ✅ (เป็นค่าเริ่มต้นให้ project ใหม่) | ✅ | ปรับที่ project ภายหลังได้ |
| Group Runners | ✅ | ✅ | Project เลือกไม่ใช้ Runner นี้ก็ได้ |
| Merge Request Approval Rules | ✅ | ✅ | ขึ้นกับว่า Owner ล็อกไว้หรือไม่ (Step 527) |
| Visibility level ขั้นต่ำ | ✅ (จำกัดเพดานบนของลูก) | ✅ | ทำให้แคบกว่าได้ กว้างกว่าไม่ได้ |

---

## Step 526: Protected branches ใน GitLab เทียบกับ Branch Protection Rules ของ GitHub

### 526.1 Protected Branches คืออะไร

ที่ `Project → Settings → Repository → Protected branches` คุณสามารถกำหนดกฎเฉพาะ branch (หรือใช้ wildcard pattern เช่น `release/*`, `hotfix/*`) ว่า:

- **Allowed to merge** — Role หรือกลุ่มคนไหนที่ merge เข้า branch นี้ได้ (ตัวเลือกทั่วไป: No one / Developers + Maintainers / Maintainers only; บนแผน Premium/Ultimate เลือกเจาะจงเป็น user/group ที่ต้องการได้)
- **Allowed to push** — Role หรือกลุ่มคนไหนที่ push (รวมถึง push ตรง ไม่ผ่าน Merge Request) เข้า branch นี้ได้
- **Allow force push** — เปิด/ปิดการอนุญาตให้ force push ทับ branch ที่ protect (ปิดเป็นค่า default เพื่อป้องกันประวัติ commit หาย)
- **Require approval from code owners** — ถ้าเปิดไว้ การ merge จะถูกบล็อกจนกว่าเจ้าของโค้ดตามไฟล์ `CODEOWNERS` จะอนุมัติ (เชื่อมกับความรู้เรื่อง CODEOWNERS ที่เคยเรียนใน Part 36)

Branch หลัก (`main` หรือ `master`) จะถูก protect โดยอัตโนมัติทันทีที่สร้าง Project ใหม่ ด้วยกฎเริ่มต้นตามที่ Group กำหนดไว้ (Step 525.5) หรือค่า default ของระบบถ้าไม่ได้ตั้งไว้

### 526.2 Protected Tags

แยกต่างหากจาก Protected Branches คือ **Protected Tags** (`Settings → Repository → Protected tags`) ใช้หลักการเดียวกันแต่ควบคุมว่าใครสร้าง/ลบ tag ที่ตรงกับ pattern ที่กำหนดได้ (เช่น `v*` สำหรับ tag เวอร์ชัน release) เพื่อป้องกันไม่ให้ใครก็ได้มาสร้าง tag เวอร์ชันปลอมทับของจริง

### 526.3 Merge Checks — จุดที่ทำหน้าที่คล้าย "Required Status Checks" ของ GitHub

ที่ `Project → Settings → Merge requests → Merge checks` มีตัวเลือกสำคัญ:

- **Pipelines must succeed** — บล็อกการกด Merge จนกว่า pipeline ของ Merge Request นั้นจะผ่านสำเร็จ (เทียบเท่า "Require status checks to pass" ของ GitHub)
- **All threads must be resolved** — บล็อกการ merge จนกว่าทุกข้อคอมเมนต์ในหน้า review จะถูกกด Resolve (เทียบเท่า "Require conversation resolution" ของ GitHub)

### 526.4 ตารางเปรียบเทียบกับ GitHub Branch Protection Rules (Part 37)

จุดที่ทำให้ผู้เรียนสับสนบ่อยที่สุดคือ GitHub รวมทุกอย่างไว้ใน "Branch Protection Rule" กฎเดียว ในขณะที่ GitLab **แยกฟีเจอร์ออกเป็นหลายจุดที่ต่างหน้ากัน**

| ความสามารถ | GitHub (Part 37) | GitLab |
|---|---|---|
| ป้องกันการ push ตรงเข้า branch | อยู่ใน Branch Protection Rule เดียวกัน | **Protected Branches** (แยกหน้าต่างหาก) |
| บังคับจำนวนผู้อนุมัติขั้นต่ำ | "Require approvals" ในกฎเดียวกัน | **Merge Request Approval Rules** (แยกหน้าต่างหาก ดู Step 527) |
| บังคับ CI ต้องผ่านก่อน merge | "Require status checks" | **Merge Checks → Pipelines must succeed** |
| บังคับ resolve คอมเมนต์ก่อน merge | "Require conversation resolution" | **Merge Checks → All threads must be resolved** |
| CODEOWNERS บังคับ approve | "Require review from Code Owners" | **Protected Branches → Require approval from code owners** |
| บังคับ signed commit | "Require signed commits" | ตั้งได้ผ่าน Push Rules (Premium) แยกต่างหาก |
| จำกัดว่าใคร push ได้ | "Restrict who can push" | **Protected Branches → Allowed to push** |
| ป้องกัน tag ปลอม | ไม่มีแยกชัดเจนเท่า GitLab | **Protected Tags** (แยกหน้าต่างหาก) |

**สรุปแนวคิด:** GitHub ออกแบบให้ทุกกฎของ branch หนึ่งอยู่รวมกันในที่เดียว (all-in-one ruleset) ส่วน GitLab ออกแบบแบบแยกเป็นโมดูล (modular) — แต่ละฟีเจอร์มีหน้าตั้งค่าของตัวเอง ข้อดีคือยืดหยุ่นและนำกลับมาใช้ซ้ำข้ามฟีเจอร์ได้ง่ายกว่า (เช่น Approval Rule เดียวใช้ได้กับหลาย branch) แต่ข้อเสียคือผู้เริ่มต้นต้องเรียนรู้หลายหน้าจอกว่าจะเข้าใจภาพรวมทั้งหมด

---

## Step 527: Approval rules ระดับ group (บังคับใช้กับทุก project ในกลุ่ม)

### 527.1 ปัญหาที่ Approval Rules ระดับ Group แก้ไข

สมมติบริษัทกำหนดนโยบายว่า **"ทุก Merge Request ของทุกโปรเจกต์ในบริษัทต้องมีคนอนุมัติอย่างน้อย 2 คนก่อน merge เสมอ"** ถ้าต้องไปตั้งค่าทีละโปรเจกต์ นอกจากจะเสียเวลาแล้ว ยังมีความเสี่ยงที่ Maintainer ของบางโปรเจกต์จะลืมตั้ง หรือแอบลดจำนวนผู้อนุมัติในภายหลังโดยไม่มีใครรู้

GitLab แก้ปัญหานี้ด้วย **Group-level Merge Request Approval Settings** (ฟีเจอร์ระดับ **Premium/Ultimate**) ที่ `Group → Settings → Merge requests`

### 527.2 สิ่งที่กำหนดได้ในระดับ Group

- **Approval rules** — กำหนดผู้อนุมัติที่จำเป็น (เจาะจงเป็น user, group, หรือให้เลือกจาก Role) พร้อมจำนวนขั้นต่ำที่ต้องอนุมัติ (Minimum approvals)
- **Prevent approval by author** — ห้ามคนเปิด Merge Request อนุมัติ Merge Request ของตัวเอง
- **Prevent approvals by users who add commits** — ห้ามคนที่ push commit เพิ่มเข้ามาใน MR อนุมัติ MR นั้น (ป้องกันการ "อนุมัติงานตัวเอง" ผ่านการช่วยกันแก้)
- **Remove all approvals when new commits are pushed** — รีเซ็ตการอนุมัติทุกครั้งที่มี commit ใหม่เข้ามา บังคับให้ผู้อนุมัติต้องมาดูโค้ดใหม่อีกครั้งเสมอ
- **Prevent editing approval rules in projects** — ตัวเลือกที่สำคัญที่สุด: **ล็อกกฎนี้** ไม่ให้ Maintainer ระดับ Project แก้ไขหรือลดทอนกฎที่ Group กำหนดได้เลย

### 527.3 ทำไมการ "ล็อก" ถึงสำคัญมาก

ถ้าไม่ล็อกไว้ Approval Rule ที่ตั้งจาก Group จะเป็นเพียง "ค่าเริ่มต้น" ที่ Maintainer ของแต่ละ Project ยังสามารถเข้าไปแก้ไข ลดจำนวนผู้อนุมัติ หรือลบกฎทิ้งได้ตามใจ ทำให้นโยบายบริษัทไม่ถูกบังคับใช้จริง

เมื่อกดล็อก (Prevent editing approval rules in projects) แล้ว **หน้าตั้งค่า Approval Rule ในระดับ Project จะกลายเป็นแบบอ่านอย่างเดียว** — Maintainer มองเห็นกฎได้แต่แก้ไม่ได้ ต้องให้ Group Owner เป็นคนปรับเปลี่ยนเท่านั้น เหมาะมากกับองค์กรที่ต้อง compliance เข้มงวด เช่น ธนาคาร, บริษัทมหาชน, หน่วยงานที่ผ่านการตรวจสอบ ISO/SOC2

### 527.4 Security Policies เสริมความเข้มงวด (Ultimate)

นอกเหนือจาก Approval Rule ทั่วไป แผน Ultimate ยังมี **Scan Result Policies** ที่กำหนดผ่าน "security policy project" กลางของ Group ได้ เช่น "ถ้า pipeline ตรวจพบช่องโหว่ความปลอดภัยระดับ Critical ให้บังคับต้องมีผู้เชี่ยวชาญด้าน Security อนุมัติ MR นี้เพิ่มอีก 1 คนเสมอ" — เป็นการเชื่อมโยง Approval Rule เข้ากับผลของ Security Scanning โดยตรง (จะเรียนเรื่อง SAST/Dependency Scanning แบบเต็ม ๆ ใน Part 54 ถัดไป)

### 527.5 ตัวอย่างการวางนโยบายจริง

| นโยบายบริษัท | วิธีตั้งค่าใน GitLab |
|---|---|
| ทุก MR ต้องมีอนุมัติอย่างน้อย 2 คน | Group Approval Rule: Minimum approvals = 2 |
| ทีม Security ต้องอนุมัติทุกครั้งที่แตะไฟล์ config การชำระเงิน | Approval Rule แบบ Code Owner + Protected Branch require code owner approval |
| ห้ามลบ/แก้กฎนี้ในระดับโปรเจกต์เด็ดขาด | เปิด "Prevent editing approval rules in projects" |
| commit ใหม่ต้องถูกตรวจซ้ำเสมอ | เปิด "Remove all approvals when new commits are pushed" |

---

## Step 528: Billing และ seat management เบื้องต้น

หัวข้อนี้เป็นภาพรวมผิวเผินพอให้เข้าใจกลไก ไม่ได้ลงลึกเรื่องราคาที่เปลี่ยนแปลงได้ตลอดเวลา

### 528.1 แผนของ GitLab.com (SaaS)

GitLab.com มี 3 แผนหลัก: **Free, Premium, Ultimate** — แผนจะถูกผูกไว้ที่ **top-level group (namespace) เดียวเท่านั้น** ไม่ว่า Group นั้นจะมี subgroup ซ้อนกันกี่ชั้นก็ตาม subgroup และ project ทั้งหมดข้างในจะใช้แผนเดียวกับ top-level group เสมอ ไม่สามารถผสมแผนต่างกันในกลุ่มเดียวกันได้

### 528.2 Seat คืออะไร

**Seat** = สมาชิก 1 คนที่นับรวมในโควตาการเรียกเก็บเงินของ top-level group นั้น หลักการนับสำคัญคือ:

> **ผู้ใช้คนหนึ่งถูกนับเป็น 1 seat ต่อ top-level namespace เท่านั้น ไม่ว่าจะเป็นสมาชิกใน subgroup หรือ project กี่ตัวข้างในก็ตาม**

เช่น ถ้าคุณเป็นสมาชิกทั้งใน `baan-software/backend-team` และ `baan-software/frontend-team` ซึ่งทั้งคู่อยู่ใต้ top-level group เดียวกันคือ `baan-software` คุณจะถูกนับเป็นแค่ **1 seat** ไม่ใช่ 2

### 528.3 ดูการใช้งาน Seat ได้ที่ไหน

```
Group (top-level) → Settings → Usage Quotas → Seats
```

หน้านี้แสดง:
- จำนวน "Seats in subscription" (ที่จ่ายเงินซื้อไว้)
- จำนวน "Seats currently in use" (ที่ใช้งานจริงตอนนี้)
- รายชื่อสมาชิกทั้งหมดที่นับเป็น billable member พร้อม Role ของแต่ละคน

### 528.4 ข้อจำกัดของแผน Free บน GitLab.com

แผน Free ของ GitLab.com (สำหรับ top-level group) มีการจำกัดจำนวนสมาชิกสูงสุดต่อกลุ่มไว้ (นโยบายนี้เริ่มบังคับใช้ตั้งแต่ปี 2022) หากเกินโควตาที่กำหนด ระบบจะแจ้งเตือนให้ต้องนำสมาชิกส่วนเกินออก หรืออัปเกรดเป็นแผนเสียเงินก่อนจึงจะเชิญสมาชิกใหม่เพิ่มได้

### 528.5 Guest Role กับ Ultimate — สิทธิ์พิเศษเรื่อง Seat

บนแผน **Ultimate** สมาชิกที่ถูกตั้งเป็น **Guest Role** จะ **ไม่ถูกนับเป็น paid seat** (ฟีเจอร์ "Free Guest seats") ทำให้องค์กรสามารถเชิญผู้บริหาร ลูกค้า หรือผู้ตรวจสอบภายนอกจำนวนมากให้เข้ามาดู Issue/ติดตามงานได้โดยไม่เพิ่มค่าใช้จ่าย ตราบใดที่คนเหล่านั้นได้แค่ Role Guest เท่านั้น (พอถูกปรับเป็น Reporter ขึ้นไปจะเริ่มนับเป็น seat ทันที)

### 528.6 Self-managed GitLab (ติดตั้งเอง)

สำหรับองค์กรที่ติดตั้ง GitLab บน server ของตัวเอง (self-managed):

| รุ่น | ค่าใช้จ่าย | จำนวนผู้ใช้ |
|---|---|---|
| **Community Edition (CE)** | ฟรี, Open Source | ไม่จำกัดจำนวนผู้ใช้ ไม่มีระบบ seat billing |
| **Enterprise Edition (EE) — Premium/Ultimate** | ต้องซื้อ License Key | คิดตามจำนวน seat เช่นกัน โดยรายงานจำนวนผู้ใช้กลับไปยัง GitLab ผ่านกลไกที่เรียกว่า **Seat Link** (เพื่อคำนวณค่า true-up หากมีผู้ใช้เกินจำนวนที่ซื้อไว้) |

### 528.7 สรุปภาพรวม

- Billing/Seat ผูกกับ **top-level group เท่านั้น** ไม่ว่าจะมี subgroup กี่ชั้นก็ตาม
- ผู้ใช้ 1 คน = 1 seat ต่อ top-level namespace (นับครั้งเดียวไม่ว่าจะอยู่กี่ subgroup/project)
- Guest role บนแผน Ultimate ไม่กินโควตา seat
- Self-managed Community Edition ใช้ฟรีไม่จำกัดคน ต่างจาก GitLab.com ที่มีโควตาตามแผน

---

## Step 529: Audit events เบื้องต้น

### 529.1 Audit Events คืออะไร

**Audit Events** คือบันทึกละเอียดของ "เหตุการณ์สำคัญเชิงความปลอดภัยและการบริหารจัดการ" ที่เกิดขึ้นใน Group/Project — ตอบคำถามประเภท "ใครทำอะไร เมื่อไหร่" ซึ่งจำเป็นมากสำหรับการตรวจสอบ (compliance audit) และการสืบสวนเหตุการณ์ผิดปกติ

ตัวอย่างเหตุการณ์ที่ถูกบันทึก:

- เปลี่ยนแปลง Role ของสมาชิก (เช่น จาก Developer เป็น Maintainer)
- เพิ่ม/ลบสมาชิกออกจาก Group หรือ Project
- ลบ Project หรือ Group
- เปลี่ยน Visibility ของ Project/Group (private ↔ public)
- เปลี่ยนแปลงกฎ Protected Branch
- เพิ่ม/ลบ SSH Key หรือ Access Token
- เปิด/ปิด 2FA ของบัญชี
- Transfer โปรเจกต์ระหว่าง namespace
- (บนแผน Ultimate) เหตุการณ์ sign-in ระดับ instance ทั้งหมด

### 529.2 ตำแหน่งที่ดู Audit Events

```
Group → Secure → Audit events
```
หรือ
```
Project → Secure → Audit events
```

(ตำแหน่งเมนูอาจเรียกว่า "Security & Compliance → Audit events" ขึ้นอยู่กับเวอร์ชันของ GitLab) สามารถกรองผลลัพธ์ตามช่วงวันที่, ตามผู้ใช้ที่ก่อเหตุการณ์, และตามประเภทเหตุการณ์ได้

### 529.3 ข้อจำกัดตามแผน

**Audit Events แบบละเอียดเต็มรูปแบบเป็นฟีเจอร์ของแผน Premium/Ultimate** แผน Free จะมีเพียงหน้า **"Activity"** ซึ่งแสดงเหตุการณ์ทั่วไประดับผิวเผิน เช่น push, เปิด/ปิด Issue, สร้าง Merge Request — ไม่ใช่บันทึกระดับ compliance ที่ครอบคลุมการเปลี่ยนแปลงสิทธิ์และการตั้งค่าความปลอดภัยแบบที่ Audit Events ให้

### 529.4 Audit Event Streaming (Ultimate)

องค์กรขนาดใหญ่ที่ต้องเก็บ log ระยะยาวเพื่อ compliance หรือส่งเข้าระบบ SIEM ของตัวเอง สามารถตั้งค่า **Audit Event Streaming** ส่งเหตุการณ์ทุกรายการแบบ real-time ออกไปยัง HTTP endpoint ปลายทาง หรือเก็บใน Amazon S3 ได้โดยอัตโนมัติ ทำให้ไม่ต้องพึ่งพา retention period ที่ GitLab เก็บไว้ให้เพียงอย่างเดียว

### 529.5 Instance-level Audit Events (สำหรับผู้ดูแลระบบ Self-managed)

สำหรับ GitLab ที่ติดตั้งเอง ผู้ดูแลระบบระดับ **Admin Area** สามารถดู Audit Events ของ **ทั้งอินสแตนซ์** (ทุก Group ทุก Project รวมกัน) ได้จากจุดเดียว ต่างจาก Group Owner ที่เห็นได้เฉพาะขอบเขต Group ของตัวเองเท่านั้น

### 529.6 กรณีใช้งานจริง

สถานการณ์ตัวอย่าง: ทีม Security สงสัยว่ามีคน "ลบโปรเจกต์ Payment Service ไปโดยไม่ได้รับอนุมัติ" — แทนที่จะไปถามทีละคน สามารถเปิด Audit Events ของ Group แล้วกรองประเภทเหตุการณ์ "Project deleted" ในช่วงเวลาที่สงสัย ก็จะเห็นทันทีว่าใครเป็นคนกดลบ เวลาใด และ IP หรือ session ใด — เป็นเครื่องมือสืบสวนที่จำเป็นมากในองค์กรที่มีทีมขนาดใหญ่

---

## Step 530: แบบฝึกหัด — สร้างโครงสร้าง group/subgroup ให้องค์กรจำลองที่มี 3 ทีม

### 530.1 โจทย์

บริษัทสมมติชื่อ **"Baan Software"** กำลังจะย้ายทุกโปรเจกต์มาอยู่บน GitLab เดียวกัน มีทีมทำงานทั้งหมด 3 ทีม:

1. **Backend Team** — ดูแล API และฐานข้อมูล มี Tech Lead 1 คน, Developer 3 คน
2. **Frontend Team** — ดูแลเว็บแอปและมือถือ มี Tech Lead 1 คน, Developer 2 คน
3. **QA Team** — ทดสอบงานของทั้งสองทีมข้างต้น มีหัวหน้า QA 1 คน, QA Tester 2 คน

นอกจากนี้ยังมี **Product Manager (PM)** 1 คนที่ต้องติดตามความคืบหน้าของทุกทีมแต่ไม่ต้องเห็นซอร์สโค้ด และ **CTO** 1 คนที่ต้องดูแลภาพรวมทั้งหมดของบริษัท

### 530.2 ขั้นตอนที่ต้องทำ

**ขั้นตอนที่ 1 — สร้าง Top-level Group**

สร้าง Group ชื่อ `baan-software` ตั้ง Visibility เป็น **Private** (เพราะเป็นโค้ดของบริษัท) ตั้งตัวเองเป็น **Owner**

**ขั้นตอนที่ 2 — สร้าง Subgroup ให้แต่ละทีม**

สร้าง subgroup 3 ตัวภายใต้ `baan-software`:

```
baan-software/
├── backend-team/
├── frontend-team/
└── qa-team/
```

**ขั้นตอนที่ 3 — กำหนดสิทธิ์ในแต่ละ Subgroup**

| ทีม | สมาชิก | Role ที่เหมาะสม | ตั้งที่ระดับ |
|---|---|---|---|
| Backend | Tech Lead คนที่ 1 | Maintainer | Subgroup `backend-team` |
| Backend | Developer 3 คน | Developer | Subgroup `backend-team` |
| Frontend | Tech Lead คนที่ 2 | Maintainer | Subgroup `frontend-team` |
| Frontend | Developer 2 คน | Developer | Subgroup `frontend-team` |
| QA | หัวหน้า QA | Maintainer | Subgroup `qa-team` |
| QA | QA Tester 2 คน | Developer | Subgroup `qa-team` (เพื่อแก้ไข test script ของตัวเองได้) |

**ขั้นตอนที่ 4 — ให้สิทธิ์ QA อ่านโค้ดของทีมอื่นได้ (แบบอ่านอย่างเดียว)**

QA ต้องดู code ของ Backend และ Frontend เพื่อเขียนเทสให้ตรงกับของจริง แต่ไม่ควรแก้โค้ดของทีมอื่น — เพิ่ม QA Team ทั้งกลุ่มเข้าไปใน subgroup `backend-team` และ `frontend-team` ด้วย **Role: Reporter** (อ่านโค้ด+ดู pipeline ได้ แต่ push ไม่ได้)

**ขั้นตอนที่ 5 — เพิ่ม PM เป็น Guest ที่ระดับบนสุด**

เพิ่ม Product Manager เข้าที่ระดับ `baan-software` (top-level group) ด้วย **Role: Guest** เนื่องจากสิทธิ์นี้จะไหลลงไปทุก subgroup อัตโนมัติ ทำให้ PM เห็น Issue และความคืบหน้าของทุกทีมได้จากจุดเดียว โดยไม่เห็นซอร์สโค้ดเลยสักบรรทัด

**ขั้นตอนที่ 6 — เพิ่ม CTO เป็น Owner ที่ระดับบนสุด**

เพิ่ม CTO เป็น **Owner** ที่ระดับ `baan-software` เพื่อให้ดูแลภาพรวม จัดการ billing และมีสิทธิ์เต็มในทุก subgroup โดยอัตโนมัติ (ตามหลัก inheritance ใน Step 522.5)

**ขั้นตอนที่ 7 — ตั้งค่า Protected Branch ให้ทุกโปรเจกต์**

ที่ระดับ Group ตั้ง **Default branch protection** เป็นระดับที่บังคับว่า Developer ห้าม push ตรงเข้า `main` ต้องเปิด Merge Request เท่านั้น และให้ **Maintainer** เป็นคนเดียวที่ merge เข้า `main` ได้จริง

**ขั้นตอนที่ 8 — ตั้ง Group-level Approval Rule และล็อกไว้**

ตั้งกฎที่ระดับ `baan-software`: ทุก Merge Request ต้องมีผู้อนุมัติอย่างน้อย **2 คน** และเปิด **"Prevent editing approval rules in projects"** เพื่อไม่ให้ Tech Lead ของแต่ละทีมมาลดจำนวนผู้อนุมัติทีหลังโดยพลการ

**ขั้นตอนที่ 9 — ตั้ง Group CI/CD Variable ที่ใช้ร่วมกัน**

สมมติทุกทีมต้อง deploy ผ่าน token เดียวกันไปยัง registry กลางของบริษัท ตั้งตัวแปร `DEPLOY_TOKEN` ที่ระดับ `baan-software` แบบ **Protected + Masked** เพื่อให้ทุก project ในทุก subgroup ใช้ค่าเดียวกันได้โดยไม่ต้องตั้งซ้ำ

**ขั้นตอนที่ 10 — ตรวจสอบผลลัพธ์ผ่าน Audit Events**

หลังตั้งค่าทั้งหมดเสร็จ เปิด `Group → Secure → Audit events` เพื่อยืนยันว่าเหตุการณ์ทั้งหมด (การเพิ่มสมาชิก, การเปลี่ยน Role, การตั้ง Approval Rule) ถูกบันทึกไว้ครบถ้วน — ฝึกอ่าน log จริงว่าหน้าตาเป็นอย่างไร

### 530.3 คำถามทบทวนก่อนดูเฉลย

ลองตอบในใจก่อนไปดูเฉลยด้านล่าง:

1. ถ้า QA Tester คนหนึ่งอยากได้สิทธิ์ Maintainer เฉพาะใน subgroup `qa-team` เพียงกลุ่มเดียว จะขัดกับกฎ inheritance ข้อไหนหรือไม่
2. ทำไม PM ถึงควรได้ Role Guest ที่ **ระดับ Group** แทนที่จะไปเพิ่มทีละ Project 3 ครั้ง
3. ถ้าไม่ล็อก Approval Rule ไว้ (ขั้นตอนที่ 8) จะเกิดความเสี่ยงอะไรได้บ้าง

### 530.4 เฉลยแนวคิด

1. **ไม่ขัดแย้ง** — กฎ inheritance ห้ามแค่ "ลดสิทธิ์ต่ำกว่าที่ inherit มา" แต่การ "เพิ่มสิทธิ์ให้สูงขึ้นเฉพาะ subgroup ย่อย" ทำได้เสมอ ดังนั้นตั้ง QA Tester คนนั้นเป็น Maintainer ที่ subgroup `qa-team` โดยตรงได้เลย โดยไม่กระทบสิทธิ์ Developer เดิมที่เขามีอยู่จาก Role อื่น
2. เพราะสิทธิ์ระดับ Group จะ **ไหลลงอัตโนมัติ** ไปทุก subgroup/project ข้างใต้ทั้งหมด ไม่ต้องเพิ่มซ้ำทีละที่ ประหยัดเวลาและลดโอกาสลืมเพิ่มในบาง Project — ถ้ามี Project ใหม่เกิดขึ้นในอนาคต PM จะเห็นได้ทันทีโดยไม่ต้องเพิ่มสิทธิ์ใหม่เลย
3. หากไม่ล็อกไว้ **Maintainer ของแต่ละทีมย่อยสามารถเข้าไปลดจำนวนผู้อนุมัติที่ระดับ Project ของตัวเองได้เอง** เช่น ลดจาก 2 คนเหลือ 1 คน หรือลบกฎทิ้งไปเลย ทำให้นโยบายบังคับสองชั้นที่บริษัทตั้งใจไว้กลายเป็นแค่ "คำแนะนำ" ที่ไม่มีผลบังคับจริง เสี่ยงต่อการหลุด code review ที่ไม่รัดกุมพอเข้าสู่ production โดยไม่มีใครรู้ตัว

---

## สรุป Part 53

ใน Part นี้เราได้เรียนรู้ว่า:

1. **GitLab Group** คือ namespace ที่เก็บ Project, Subgroup, สมาชิก และการตั้งค่าไว้ด้วยกัน มีพลังมากกว่า GitHub Organization ตรงที่สร้างโครงสร้างแบบต้นไม้ซ้อนกันได้หลายชั้น
2. **Subgroup** ทำให้จำลองโครงสร้างองค์กรจริงลงบน GitLab ได้ตรง ๆ โดยมีกฎสำคัญ 2 ข้อที่ต้องจำ: Visibility ของลูกต้องแคบกว่าหรือเท่ากับพ่อแม่เสมอ และ Permission ที่ inherit ลงมาเพิ่มได้แต่ลดไม่ได้
3. GitLab มี Permission 5 ระดับหลัก **Guest → Reporter → Developer → Maintainer → Owner** แต่ละระดับรวมสิทธิ์ของระดับล่างไว้ทั้งหมดแล้วเพิ่มความสามารถใหม่เข้าไป
4. การเชิญสมาชิกทำได้ทั้งแบบ **invite by username** (มีบัญชีอยู่แล้ว) และ **invite by email** (ยังไม่มีบัญชี ต้องรอตอบรับ) ทั้งในระดับ Group และ Project
5. การตั้งค่าที่ระดับ Group เช่น **CI/CD Variables, Runners, Labels, Default Branch Protection, Push Rules** จะไหลลงไปทุก Project ในทุก Subgroup โดยอัตโนมัติ ลดการตั้งค่าซ้ำซ้อน
6. **Protected Branches** ของ GitLab แยกฟีเจอร์ออกจาก **Approval Rules** และ **Merge Checks** ต่างจาก GitHub ที่รวมทุกอย่างไว้ใน Branch Protection Rule เดียว
7. **Group-level Approval Rules** (Premium/Ultimate) บังคับใช้กับทุกโปรเจกต์ในกลุ่มได้ และสามารถ **ล็อก** ไม่ให้ Maintainer ระดับ Project มาลดทอนกฎทีหลังได้
8. **Billing/Seat** ผูกกับ top-level group เท่านั้น ผู้ใช้ 1 คนนับ 1 seat ไม่ว่าจะอยู่กี่ subgroup ก็ตาม และ Guest role บน Ultimate ไม่กินโควตา seat
9. **Audit Events** (Premium/Ultimate) บันทึกเหตุการณ์สำคัญเชิงความปลอดภัยและการบริหารจัดการ ใช้สืบสวนว่าใครทำอะไรในองค์กรได้อย่างละเอียด

### Checklist ก่อนไป Part 54

- [ ] อธิบายความแตกต่างระหว่าง GitLab Group กับ GitHub Organization ได้
- [ ] เข้าใจกฎ Visibility inheritance และ Permission inheritance ของ Subgroup
- [ ] บอกความสามารถของ Permission ทั้ง 5 ระดับได้ครบถ้วนโดยไม่ต้องเปิดเอกสาร
- [ ] เชิญสมาชิกเข้า Group/Project ได้ทั้งสองวิธี (username และ email)
- [ ] รู้ว่าการตั้งค่าใดบ้างที่ระดับ Group จะไหลลงไปยัง Project ย่อยโดยอัตโนมัติ
- [ ] แยกความแตกต่างระหว่าง Protected Branches, Merge Checks และ Approval Rules ของ GitLab ได้
- [ ] เข้าใจว่า Group-level Approval Rule ล็อกไว้เพื่ออะไร
- [ ] เข้าใจภาพรวมว่า Seat นับอย่างไรที่ top-level group
- [ ] รู้ว่า Audit Events ใช้ตรวจสอบอะไรและอยู่ตรงไหน
- [ ] ทำแบบฝึกหัดวางโครงสร้าง Group/Subgroup ให้องค์กรจำลอง 3 ทีมได้ครบทุกขั้นตอน

**ต่อไป:** [Part 54: GitLab Security Features: SAST, Dependency Scanning เบื้องต้น](./part-054-gitlab-security-features.md)
