# Part 32: Git Flow: โมเดลมาตรฐานสำหรับทีมใหญ่

> **Step ในหลักสูตรนี้:** Step 311–320
> **เฟส:** 4 — ทำงานเป็นทีมด้วย Workflow มาตรฐาน (Git Flow, GitHub Flow, Rebase)
> **เป้าหมายของ Part นี้:** เข้าใจ Git Flow อย่างละเอียดตั้งแต่แนวคิดดั้งเดิมของ Vincent Driessen ไปจนถึงการลงมือสร้าง feature, release และ hotfix branch จริงด้วยมือและด้วยเครื่องมือ `git-flow` เพื่อให้เข้าใจว่าทำไมทีมใหญ่จำนวนมากยังเลือกใช้โมเดลนี้ และทำไมบางทีมถึงเลิกใช้มันไปแล้ว

---

## สารบัญของ Part นี้

- Step 311: Git Flow คืออะไร ใครสร้าง เหมาะกับโปรเจกต์แบบไหน
- Step 312: Branch หลักของ Git Flow — `main`/`master` และ `develop`
- Step 313: Feature Branches — แตกจาก `develop` กลับเข้า `develop`
- Step 314: Release Branches — เตรียมความพร้อมก่อนออกเวอร์ชันจริง
- Step 315: Hotfix Branches — แก้บั๊กด่วนจาก `main` โดยตรง
- Step 316: Support Branches — ดูแลเวอร์ชันเก่าที่ยังใช้งานอยู่
- Step 317: เครื่องมือ git-flow extension — คำสั่งสำเร็จรูปที่ทำให้ทำงานเร็วขึ้น
- Step 318: ข้อดีข้อเสียของ Git Flow ในโลกการทำงานจริง
- Step 319: เมื่อไหร่ควรใช้ Git Flow เมื่อไหร่ไม่ควรใช้ (Decision Guide)
- Step 320: แบบฝึกหัด — จำลองวงจร release เต็มรูปแบบด้วย Git Flow

---

## Step 311: Git Flow คืออะไร ใครสร้าง เหมาะกับโปรเจกต์แบบไหน

### จุดเริ่มต้นของ Git Flow

ในเดือนมกราคม ปี **2010** นักพัฒนาชาวดัตช์ชื่อ **Vincent Driessen** ได้เผยแพร่บทความหนึ่งบนบล็อกส่วนตัวของเขา (nvie.com) ชื่อว่า **"A successful Git branching model"** บทความนี้เสนอ **แบบแผนการแตก branch (branching model)** ที่มีโครงสร้างชัดเจน เป็นระบบ และตอบโจทย์ทีมพัฒนาซอฟต์แวร์ที่ต้อง **ออก release เป็นเวอร์ชัน ๆ อย่างมีวินัย**

บทความนี้ได้รับความนิยมอย่างมหาศาลในวงการ จนคำว่า **"Git Flow"** กลายเป็นชื่อเรียกโมเดลนี้อย่างไม่เป็นทางการ (แม้ว่าตัวบทความต้นฉบับของ Vincent Driessen จะไม่ได้ใช้คำว่า "Git Flow" ตรง ๆ ก็ตาม — ชื่อนี้ติดปากมาจากชื่อเครื่องมือ command-line ที่เขาสร้างตามมาทีหลังชื่อ `git-flow` ซึ่งเราจะเรียนใน Step 317)

> **ที่มาอ้างอิง:** บทความต้นฉบับอยู่ที่ https://nvie.com/posts/a-successful-git-branching-model/ — เนื้อหาใน Part นี้ยึดตามโมเดลดั้งเดิมของ Vincent Driessen ทุกประการ ไม่ใช่เวอร์ชันดัดแปลงที่พบเห็นได้ทั่วไปตามบทความรอง ๆ ในอินเทอร์เน็ต

### ทำไม Git Flow ถึงถือกำเนิดขึ้นมา

ก่อนหน้านั้น ทีมพัฒนาจำนวนมากใช้ branch อย่างไม่มีระบบ — บางคน commit ตรงเข้า `master` เลย บางคนสร้าง branch ทดลองแล้วลืมลบ ผลคือ:

1. **ไม่รู้ว่า branch ไหนคือ "โค้ดที่กำลังใช้งานจริง (production)"** และ branch ไหนคือ "โค้ดที่กำลังพัฒนาอยู่ (in progress)"
2. **การเตรียม release ปนกับการพัฒนาฟีเจอร์ใหม่** — พอจะปล่อยเวอร์ชันใหม่ ต้องมานั่งแยกว่าอันไหนพร้อมอันไหนไม่พร้อม
3. **การแก้บั๊กด่วนบน production ทำให้โค้ดที่กำลังพัฒนาอยู่ปนเปื้อน** — ถ้าต้อง hotfix ด่วน แต่โค้ดบน `develop` ยังไม่เสร็จ จะทำอย่างไร

Vincent Driessen จึงออกแบบโมเดลที่กำหนด **บทบาทของแต่ละ branch อย่างชัดเจนตายตัว** — ทุก branch มีหน้าที่เฉพาะ มีจุดเริ่มต้นและจุดสิ้นสุดที่แน่นอน ไม่ปะปนกัน

### ภาพรวมของ Git Flow แบบสรุปเร็ว

```
main      : เก็บเฉพาะโค้ดที่ deploy ขึ้น production แล้วเท่านั้น ทุก commit บน main คือ 1 release
develop   : เก็บโค้ดที่รวมฟีเจอร์ทั้งหมดที่ "เสร็จแล้ว" รอเตรียม release ถัดไป
feature/* : พัฒนาฟีเจอร์ใหม่ แตกจาก develop กลับเข้า develop
release/* : เตรียมความพร้อมก่อนออกเวอร์ชัน แตกจาก develop กลับเข้าทั้ง main และ develop
hotfix/*  : แก้บั๊กด่วนบน production แตกจาก main กลับเข้าทั้ง main และ develop
support/* : ดูแลเวอร์ชันเก่าที่ยังต้อง maintain คู่ขนานกับเวอร์ชันปัจจุบัน
```

### Git Flow เหมาะกับโปรเจกต์แบบไหน

Git Flow ไม่ใช่โมเดลสากลที่ใช้ได้ดีกับทุกโปรเจกต์ มันถูกออกแบบมาให้เหมาะกับลักษณะงานแบบนี้โดยเฉพาะ:

| ลักษณะโปรเจกต์ | เหมาะกับ Git Flow หรือไม่ |
|---|---|
| มีเลข **version** ชัดเจน (เช่น v1.0, v1.1, v2.0) ที่ต้องประกาศอย่างเป็นทางการ | เหมาะมาก |
| ซอฟต์แวร์ที่ต้อง **แพ็กเกจแล้วส่งมอบ** เป็นรอบ ๆ (เช่น mobile app ที่ต้องผ่านการ review ของ App Store, ซอฟต์แวร์ enterprise ที่ขายเป็นเวอร์ชัน, firmware ของอุปกรณ์) | เหมาะมาก |
| ต้อง **maintain หลายเวอร์ชันพร้อมกัน** (เช่น ลูกค้าองค์กรบางรายยังใช้ v1.x บางรายใช้ v2.x) | เหมาะมาก |
| ทีมมีขนาดใหญ่ ต้องการ**ขั้นตอนที่ตายตัวและควบคุมได้** ก่อนโค้ดจะถึง production | เหมาะ |
| เว็บแอปพลิเคชันที่ **deploy ขึ้น production ได้ทุกวันหรือหลายครั้งต่อวัน** (Continuous Deployment) | **ไม่เหมาะ** — จะซับซ้อนเกินความจำเป็น (จะอธิบายเหตุผลใน Step 318–319) |

จำหลักสั้น ๆ ไว้ก่อน: **ถ้าโปรเจกต์ของคุณมีคำว่า "เวอร์ชัน" ที่ต้องประกาศเป็นทางการ Git Flow คือตัวเลือกที่ยอดเยี่ยม** แต่ถ้าโปรเจกต์ของคุณ deploy ขึ้น production ตลอดเวลาแบบไม่มีเวอร์ชันตายตัว ให้รอดู **Part 33: GitHub Flow** ซึ่งเป็นโมเดลที่เรียบง่ายกว่ามาก

---

## Step 312: Branch หลักของ Git Flow — `main`/`master` และ `develop`

Git Flow เป็นโมเดลแรก ๆ ที่ทำให้แนวคิด **"branch ระยะยาว 2 เส้นคู่ขนานกัน" (long-running branches)** กลายเป็นมาตรฐาน แทนที่จะมี branch หลักแค่เส้นเดียว โมเดลนี้กำหนดให้มี **branch หลักที่ไม่มีวันถูกลบ 2 เส้น**

### 312.1 `main` (หรือ `master` ในบทความต้นฉบับปี 2010)

> หมายเหตุเรื่องชื่อ: บทความต้นฉบับของ Vincent Driessen เขียนขึ้นในปี 2010 ตอนที่ Git ยังใช้ชื่อ branch เริ่มต้นว่า `master` เป็นมาตรฐาน ปัจจุบัน (ตั้งแต่ปี 2020 เป็นต้นมา) วงการซอฟต์แวร์ส่วนใหญ่รวมถึง GitHub เปลี่ยนมาใช้ `main` เป็นค่าเริ่มต้นแทน หลักการของ Git Flow เหมือนกันทุกประการ เพียงแค่เปลี่ยนชื่อ branch เท่านั้น ในเอกสารนี้เราจะใช้ **`main`** เป็นหลัก แต่จะพูดถึง `master` เมื่ออ้างอิงถึงบทความต้นฉบับ

**หน้าที่ของ `main`:** เก็บเฉพาะ **โค้ดที่พร้อมใช้งานจริงบน production เท่านั้น**

กฎเหล็กของ `main` ในโมเดล Git Flow ดั้งเดิม:

1. **ห้าม commit ตรงเข้า `main` โดยเด็ดขาด** ยกเว้นผ่านการ merge จาก `release/*` หรือ `hotfix/*` เท่านั้น
2. **ทุก commit บน `main` ควรมี tag กำกับเป็นเลขเวอร์ชันเสมอ** (เช่น `v1.0.0`, `v1.1.0`, `v1.1.1`) เพราะทุก commit บน `main` คือจุดที่เคย (หรือกำลัง) deploy ขึ้นจริง
3. `main` คือ branch ที่ **เสถียรที่สุดตลอดเวลา** — ใครก็ตามที่ checkout `main` ต้องได้โค้ดที่ทำงานได้สมบูรณ์เสมอ ไม่มีข้อยกเว้น

### 312.2 `develop`

**หน้าที่ของ `develop`:** เป็น **integration branch** หรือ "จุดรวมงาน" ของทุกฟีเจอร์ที่พัฒนาเสร็จแล้ว แต่ **ยังไม่ถึงคิว release ต่อไป**

กฎของ `develop`:

1. `develop` คือจุดที่ **feature branch ทุกอันแตกออกไป และกลับมารวมกันอีกครั้ง**
2. โค้ดบน `develop` ควร **compile ผ่านและรัน automated test ผ่านเสมอ** (เพราะเป็นจุดรวมงานของทั้งทีม) แต่ **ไม่จำเป็นต้องพร้อม 100% สำหรับ production** — อาจมีฟีเจอร์ที่ยังไม่สมบูรณ์แบบ หรือยังไม่ผ่าน QA เต็มรูปแบบ
3. เมื่อ `develop` มีฟีเจอร์สะสมมากพอสำหรับ release รอบใหม่ ทีมจะแตก **release branch** ออกจาก `develop` (Step 314)

### แผนภาพความสัมพันธ์ระหว่าง `main` และ `develop`

```
main     ●───────────────────●───────────────●────────▶  (production only, มี tag ทุกจุด)
         v1.0.0              v1.1.0          v1.1.1
          │                    ▲                ▲
          │                    │ merge          │ merge (hotfix)
          ▼                    │                │
develop  ●───●───●───●───●───●─┴───●───●───●────┴──────▶  (integration, รวมฟีเจอร์ทั้งหมด)
             ▲   ▲       ▲
             │   │       │
          feature feature feature   (แตกและรวมกลับเข้า develop ตลอดเวลา)
```

สังเกตว่า:

- `main` เดินหน้าแบบ **กระโดดเป็นจุด ๆ** — แต่ละจุดคือ 1 release ที่ผ่านการเตรียมมาอย่างดีแล้ว
- `develop` เดินหน้า **ต่อเนื่องตลอดเวลา** — เป็นจุดรวมงานของทั้งทีมทุกวัน
- ลูกศรจาก `develop` ไป `main` เกิดขึ้นเฉพาะตอนที่มี release branch หรือ hotfix branch merge เข้ามาเท่านั้น (ไม่เคย merge `develop` เข้า `main` ตรง ๆ)

### การตั้งค่าเริ่มต้นในทางปฏิบัติ

เมื่อเริ่มโปรเจกต์ใหม่ด้วย Git Flow ทีมจะต้องสร้าง `develop` ขึ้นจาก `main` ตั้งแต่แรกสุด:

```bash
git checkout main
git checkout -b develop
git push -u origin develop
```

จากจุดนี้เป็นต้นไป **`develop` คือ branch ที่ทุกคนในทีมจะแตก feature branch ออกไป** ไม่ใช่ `main`

---

## Step 313: Feature Branches — แตกจาก `develop` กลับเข้า `develop`

### หน้าที่ของ Feature Branch

**Feature branch** คือ branch ที่ใช้พัฒนา **ฟีเจอร์ใหม่หนึ่งอย่าง** หรือ **งานหนึ่งชิ้น** โดยไม่กระทบกับงานของคนอื่นในทีมระหว่างที่ยังทำไม่เสร็จ

กฎของ feature branch ตามโมเดลดั้งเดิม:

| กฎ | รายละเอียด |
|---|---|
| **แตกออกจาก** | `develop` เท่านั้น |
| **merge กลับเข้า** | `develop` เท่านั้น |
| **ชื่อ branch** | ไม่มีข้อบังคับตายตัว แต่นิยมใช้ prefix `feature/` เช่น `feature/login-form`, `feature/payment-gateway` |
| **อายุ** | มีอยู่ตราบเท่าที่ฟีเจอร์นั้นยังพัฒนาไม่เสร็จ (ควรจบให้เร็วที่สุด ยิ่งอยู่นานยิ่งเสี่ยง conflict) |
| **ไม่มีการติดต่อกับ `main` โดยตรง** | Feature branch ไม่รู้จัก ไม่แตะต้อง และไม่ merge เข้า `main` เลย |

### ขั้นตอนการทำงานกับ Feature Branch

```bash
# 1. เริ่มต้นฟีเจอร์ใหม่ — แตกจาก develop
git checkout develop
git pull origin develop
git checkout -b feature/login-form

# 2. ทำงาน commit ไปเรื่อย ๆ ตามปกติ
git add .
git commit -m "เพิ่มฟอร์ม login พร้อม validation เบื้องต้น"
git commit -m "เชื่อมต่อ API login กับ backend"

# 3. เมื่อฟีเจอร์เสร็จสมบูรณ์ ผ่านการทดสอบแล้ว — รวมกลับเข้า develop
git checkout develop
git pull origin develop
git merge --no-ff feature/login-form

# 4. ลบ feature branch ทิ้ง (ไม่จำเป็นต้องเก็บไว้อีกต่อไป)
git branch -d feature/login-form
git push origin develop
```

### ทำไมต้องใช้ `--no-ff` เสมอตอน merge feature เข้า develop

นี่คือรายละเอียดสำคัญมากที่ Vincent Driessen เน้นย้ำในบทความต้นฉบับ ปกติ Git จะพยายามทำ **fast-forward merge** ถ้าเป็นไปได้ (คือแค่เลื่อน pointer ของ `develop` ไปชี้ที่ commit ล่าสุดของ feature branch โดยไม่สร้าง merge commit ใหม่) แต่การทำแบบนั้นจะทำให้ **ประวัติของฟีเจอร์นั้นถูกกลืนหายไปในสายเวลาเดียวกับ commit อื่น ๆ** มองไม่ออกอีกต่อไปว่า commit กลุ่มไหนเคยเป็นฟีเจอร์เดียวกัน

การใช้ `--no-ff` (no fast-forward) จะ **บังคับให้ Git สร้าง merge commit ใหม่เสมอ** แม้ว่าจะสามารถ fast-forward ได้ก็ตาม ผลลัพธ์คือประวัติจะรักษา "จุดแตกและจุดรวม" ของฟีเจอร์นั้นไว้อย่างชัดเจน

```
ไม่ใช้ --no-ff (fast-forward)          ใช้ --no-ff (สร้าง merge commit)

develop ●──●──●──●──●──●──▶            develop ●──●──●─────────●───▶
              (มองไม่ออกว่าฟีเจอร์            \              ╱
               ไหนคือกลุ่มไหน)                  ●──●──●──●──╱
                                          feature/login-form
                                        (เห็นชัดว่านี่คือฟีเจอร์เดียวกัน
                                         ทั้งหมด รวมกันที่จุดเดียว)
```

### ตัวอย่างการทำงานพร้อมกันหลายฟีเจอร์

จุดแข็งของโมเดลนี้คือหลายคนแตก feature branch ออกจาก `develop` พร้อมกันได้โดยไม่รบกวนกัน:

```
develop     ●───●───●───────●───────────●───●───▶
             \   \           \         ╱   ╱
              \   \           ╲       ╱   ╱
               ●───●           ●─────╱   ╱
          feature/login-form  feature/search
                                          ●───●
                                     feature/dark-mode
```

ทุกฟีเจอร์พัฒนาอย่างอิสระ เสร็จเมื่อไหร่ก็ merge กลับเข้า `develop` เมื่อนั้น ไม่ต้องรอกัน

---

## Step 314: Release Branches — เตรียมความพร้อมก่อนออกเวอร์ชัน

### หน้าที่ของ Release Branch

นี่คือหัวใจสำคัญที่สุดของ Git Flow ที่ทำให้มันต่างจากโมเดลอื่น ๆ **Release branch คือพื้นที่ "กันชน" (buffer zone)** ระหว่าง `develop` กับ `main` ใช้สำหรับ **เตรียมความพร้อมก่อนออกเวอร์ชันจริง** โดยไม่ต้องหยุดทีมไม่ให้พัฒนาฟีเจอร์ใหม่ต่อบน `develop`

กฎของ release branch:

| กฎ | รายละเอียด |
|---|---|
| **แตกออกจาก** | `develop` เมื่อฟีเจอร์ที่ต้องการสำหรับ release รอบนี้ครบแล้ว (feature freeze) |
| **merge กลับเข้า** | ทั้ง `main` **และ** `develop` |
| **ชื่อ branch** | นิยมใช้ prefix `release/` ตามด้วยเลขเวอร์ชัน เช่น `release/1.2.0` |
| **สิ่งที่ทำได้บน branch นี้** | Bump เลขเวอร์ชัน, แก้ bug เล็กน้อยที่เจอตอนทดสอบ, ปรับ documentation, ทดสอบซ้ำ (regression test) |
| **สิ่งที่ห้ามทำบน branch นี้** | **ห้ามเพิ่มฟีเจอร์ใหม่เด็ดขาด** — ถ้าอยากได้ฟีเจอร์ใหม่ ต้องรอ release รอบถัดไป |

### ทำไมต้องมี Release Branch แยกจาก `develop`

เหตุผลสำคัญคือ **`develop` ต้องเดินหน้าต่อไปได้ตลอดเวลา** ทีมอื่นยังต้องพัฒนาฟีเจอร์สำหรับเวอร์ชันถัดไปต่อไป ในขณะที่อีกทีม (หรือคนเดิม) กำลังง่วนกับการทดสอบและขัดเกลาเวอร์ชันที่กำลังจะออก ถ้าไม่มี release branch แยก การพัฒนาฟีเจอร์ใหม่จะต้องหยุดชะงักทุกครั้งที่ใกล้ถึงกำหนด release

```
develop   ●───●───●───●(freeze)──●───●───●───●───▶ (ฟีเจอร์ใหม่พัฒนาต่อได้เรื่อย ๆ)
                        │
                        │ แตก release branch
                        ▼
release/1.2.0          ●───●───●───●
                    (bump version, bug fix เล็กน้อย, ทดสอบ)
                                    │
                        ┌───────────┴───────────┐
                        │ merge เข้า main         │ merge กลับเข้า develop
                        ▼                         ▼
main       ────────────●(v1.2.0, tag)    develop ───────●───●───▶
```

### ขั้นตอนการทำงานกับ Release Branch แบบละเอียด

```bash
# 1. เมื่อ develop มีฟีเจอร์ครบสำหรับ release รอบนี้แล้ว — แตก release branch
git checkout develop
git pull origin develop
git checkout -b release/1.2.0

# 2. Bump เลขเวอร์ชันในไฟล์ที่เกี่ยวข้อง (เช่น package.json, VERSION)
#    แล้ว commit
git commit -am "Bump version to 1.2.0"

# 3. ทดสอบอย่างละเอียด ถ้าเจอบั๊กเล็กน้อยให้แก้ตรงนี้เลย
git commit -am "แก้ไข bug การแสดงผลวันที่ในหน้า dashboard"

# 4. เมื่อพร้อม release จริง — merge เข้า main และติด tag
git checkout main
git pull origin main
git merge --no-ff release/1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin main --tags

# 5. merge กลับเข้า develop ด้วย (สำคัญมาก ห้ามลืมขั้นตอนนี้!)
git checkout develop
git pull origin develop
git merge --no-ff release/1.2.0
git push origin develop

# 6. ลบ release branch ทิ้ง เพราะหมดหน้าที่แล้ว
git branch -d release/1.2.0
git push origin --delete release/1.2.0
```

### จุดที่มือใหม่พลาดบ่อยที่สุดใน Release Branch

> **ข้อผิดพลาดคลาสสิก:** merge release branch เข้า `main` แล้ว **ลืม merge กลับเข้า `develop`** ด้วย

ถ้าลืมขั้นตอนนี้ การแก้ bug เล็กน้อยที่ทำไว้บน release branch (เช่นการแก้ bug วันที่ในตัวอย่างด้านบน) จะ **หายไปจาก `develop`** ทันที เพราะ `develop` ไม่เคยรู้จักการแก้ไขนั้นเลย พอถึง release รอบถัดไป bug ตัวเดิมอาจจะโผล่กลับมาอีกครั้งเพราะไม่เคยถูกแก้ใน `develop` เลยตั้งแต่ต้น — นี่คือเหตุผลที่กฎ **"release branch merge กลับเข้าทั้งสองฝั่งเสมอ"** สำคัญมาก และเป็นจุดที่ต้องระวังที่สุดในการทำ Git Flow ด้วยมือ

---

## Step 315: Hotfix Branches — แก้บั๊กด่วนจาก `main` โดยตรง

### หน้าที่ของ Hotfix Branch

บางครั้งเกิดบั๊กร้ายแรงบน production ที่ **ต้องแก้ทันที ไม่สามารถรอรอบ release ปกติได้** เช่น ช่องโหว่ด้านความปลอดภัย, ระบบชำระเงินพัง, หน้าเว็บล่มทั้งหมด สถานการณ์แบบนี้คือหน้าที่ของ **hotfix branch**

จุดสำคัญที่ต่างจาก release branch คือ **hotfix แตกออกจาก `main` โดยตรง ไม่ใช่จาก `develop`** เพราะ ณ ตอนที่เกิดปัญหา `develop` อาจจะมีฟีเจอร์ที่กำลังพัฒนาค้างอยู่ครึ่ง ๆ กลาง ๆ ซึ่งยังไม่พร้อม deploy เราไม่ต้องการให้โค้ดที่ยังไม่เสร็จเหล่านั้นติดไปกับการแก้บั๊กด่วนด้วย

กฎของ hotfix branch:

| กฎ | รายละเอียด |
|---|---|
| **แตกออกจาก** | `main` (จาก tag ล่าสุดที่ deploy อยู่บน production) |
| **merge กลับเข้า** | ทั้ง `main` **และ** `develop` (หรือ `release/*` ถ้ามี release branch กำลังทำงานอยู่พอดี) |
| **ชื่อ branch** | นิยมใช้ prefix `hotfix/` เช่น `hotfix/1.2.1`, `hotfix/security-patch-cve-2026-1234` |
| **เลขเวอร์ชัน** | ตามหลัก Semantic Versioning ให้ bump เลข **patch version** ขึ้น 1 (เช่นจาก 1.2.0 เป็น 1.2.1) |

### แผนภาพ Hotfix Branch

```
main       ●─────────────────────●─────────────●────▶
           v1.2.0                │              v1.2.1 (tag)
                                  │              ▲
                                  │ แตก hotfix    │ merge กลับ
                                  ▼              │
hotfix/1.2.1                     ●──────●───────┘
                          (แก้บั๊กด่วน, bump patch version)
                                          │
                                          │ merge เข้า develop ด้วย
                                          ▼
develop   ●───●───●───●───●──────────────●───●───▶
                  (ฟีเจอร์ใหม่พัฒนาต่อได้เรื่อย ๆ
                   ไม่ถูกรบกวนจาก hotfix)
```

### ขั้นตอนการทำงานกับ Hotfix Branch แบบละเอียด

```bash
# 1. เกิดบั๊กร้ายแรงบน production — แตก hotfix จาก main ทันที
git checkout main
git pull origin main
git checkout -b hotfix/1.2.1

# 2. แก้บั๊ก แล้ว bump patch version
git commit -am "แก้ไขช่องโหว่ SQL Injection ในหน้า search"
git commit -am "Bump version to 1.2.1"

# 3. ทดสอบให้แน่ใจว่าใช้งานได้จริง แล้ว merge เข้า main พร้อม tag
git checkout main
git merge --no-ff hotfix/1.2.1
git tag -a v1.2.1 -m "Hotfix: แก้ไขช่องโหว่ SQL Injection"
git push origin main --tags

# 4. merge กลับเข้า develop ด้วยเสมอ (สำคัญเหมือนกับ release branch)
git checkout develop
git merge --no-ff hotfix/1.2.1
git push origin develop

# 5. ลบ hotfix branch
git branch -d hotfix/1.2.1
git push origin --delete hotfix/1.2.1
```

### กรณีพิเศษ: ถ้ามี Release Branch กำลังทำงานอยู่พอดีตอนเกิด Hotfix

ตามโมเดลดั้งเดิมของ Vincent Driessen ถ้าเกิด hotfix ขึ้นในขณะที่มี `release/*` branch กำลังเปิดอยู่แล้ว (กำลังเตรียม release ถัดไป) ให้ **merge hotfix เข้า release branch แทนที่จะ merge เข้า develop ตรง ๆ** เพื่อป้องกันการ merge ซ้ำซ้อนหรือ conflict เพราะเมื่อ release branch นั้น merge เข้า develop ในภายหลัง การแก้ไขจาก hotfix ก็จะติดตามเข้าไปด้วยโดยอัตโนมัติอยู่แล้ว

```
                    release/1.3.0 กำลังเปิดอยู่
                           │
develop   ●───●───●────────┤
                            ●───●───●  (release/1.3.0)
                                 ▲
main      ●────────●────────────┼──────●───▶
          v1.2.0    │           │      v1.2.1
                     │ hotfix   │
                     ▼          │ merge hotfix เข้า
              hotfix/1.2.1 ─────┘ release/1.3.0 แทน develop
```

### ทำไม Hotfix ถึงสำคัญมากในเชิงธุรกิจ

Hotfix คือกลไกที่ทำให้ทีมสามารถ **แก้ปัญหาวิกฤตบน production ได้เร็วที่สุดเท่าที่จะทำได้** โดยไม่ต้องรอให้ฟีเจอร์ที่กำลังพัฒนาอยู่บน `develop` เสร็จสมบูรณ์ก่อน นี่คือจุดที่แสดงให้เห็นว่า Git Flow ถูกออกแบบมาอย่างรอบคอบเพื่อรองรับสถานการณ์จริงในทีมพัฒนาซอฟต์แวร์ระดับองค์กร

---

## Step 316: Support Branches — ดูแลเวอร์ชันเก่าที่ยังใช้งานอยู่

### หน้าที่ของ Support Branch

**Support branch** เป็น branch ประเภทที่ห้าที่ Vincent Driessen กล่าวถึงในบทความต้นฉบับ แต่ใช้งานน้อยกว่าสี่ประเภทแรกมาก เพราะมันมีไว้สำหรับสถานการณ์เฉพาะเจาะจง:

> **ทีมต้อง maintain เวอร์ชันเก่าที่ยังมีลูกค้าใช้งานอยู่ ในขณะที่การพัฒนาหลักได้เดินหน้าไปยังเวอร์ชันใหม่กว่ามากแล้ว**

ตัวอย่างสถานการณ์จริง: บริษัทซอฟต์แวร์ enterprise ออกเวอร์ชัน 2.0 ไปแล้ว และกำลังพัฒนาเวอร์ชัน 3.0 อยู่บน `develop` แต่มีลูกค้าองค์กรรายใหญ่หลายรายที่ **ยังใช้เวอร์ชัน 2.x อยู่และมีสัญญาต้อง maintain ต่อ** (เช่นต้องออก security patch ให้เวอร์ชัน 2.x ไปอีก 2 ปี) ทีมจึงสร้าง `support/2.x` แยกออกมาต่างหาก

### กฎของ Support Branch

| กฎ | รายละเอียด |
|---|---|
| **แตกออกจาก** | `main` ณ จุด tag ของเวอร์ชันที่ต้องการดูแลต่อ (เช่นแตกจาก tag `v2.0.0`) |
| **อายุ** | อยู่ได้ยาวนานมาก อาจเป็นปี ๆ ตราบเท่าที่ยังมีลูกค้าใช้เวอร์ชันนั้นอยู่ |
| **การแก้ไขบน branch นี้** | คล้าย hotfix — แก้บั๊กเฉพาะที่จำเป็นสำหรับเวอร์ชันนั้น ไม่รับฟีเจอร์ใหม่จากเวอร์ชันหลัก |
| **การ merge กลับ** | **ปกติจะไม่ merge กลับเข้า `develop`** เพราะ `develop` มักจะพัฒนาล้ำหน้าไปไกลมากแล้วจนโครงสร้างโค้ดต่างกันมาก การแก้ไขบางอย่างอาจไม่เกี่ยวข้องกับเวอร์ชันใหม่เลย (แต่ถ้า bug นั้นเกี่ยวข้องกับทุกเวอร์ชัน ทีมควรพิจารณา cherry-pick การแก้ไขไปใส่ `develop` ด้วยเช่นกัน) |

### แผนภาพ Support Branch

```
main         ●──────●──────────────●──────────▶ (v3.x กำลังพัฒนาต่อ)
             v2.0.0  │              v3.0.0
                     │
                     │ แตก support branch จาก tag v2.0.0
                     ▼
support/2.x         ●───●───●───●───▶  (maintain แยกต่างหาก
                                         ออก patch v2.0.1, v2.0.2, ...
                                         ให้ลูกค้าที่ยังใช้ v2.x)
```

### ทำไม Support Branch จึงพบเห็นน้อยกว่าประเภทอื่น

Support branch เหมาะกับ **ซอฟต์แวร์ที่ขายเป็นเวอร์ชันแบบ on-premise หรือ firmware** ที่ลูกค้าควบคุมเวลาการอัปเดตเอง (ไม่เหมือนเว็บแอปพลิเคชันที่ทุกคนใช้เวอร์ชันล่าสุดเสมอโดยอัตโนมัติ) เช่น ซอฟต์แวร์ ERP ขององค์กร, ระบบปฏิบัติการ, driver ของฮาร์ดแวร์ หรือ library/SDK ที่ผู้ใช้หลายรายค้างอยู่ที่ major version ต่างกัน ด้วยเหตุนี้ทีมเว็บแอปพลิเคชันหรือ SaaS ทั่วไปจึงแทบไม่เคยได้ใช้ support branch เลยตลอดอายุโปรเจกต์

---

## Step 317: เครื่องมือ git-flow extension — คำสั่งสำเร็จรูปที่ทำให้ทำงานเร็วขึ้น

### git-flow คืออะไร

หลังจากบทความของ Vincent Driessen โด่งดังขึ้นมา เขาและทีมงานได้สร้างเครื่องมือเสริม (extension) ให้กับ Git ชื่อ **`git-flow`** ซึ่งเป็นชุดคำสั่งที่ห่อหุ้ม (wrap) คำสั่ง Git พื้นฐานหลาย ๆ คำสั่งเข้าด้วยกัน ให้กลายเป็นคำสั่งเดียวที่ทำตามขั้นตอนของโมเดล Git Flow โดยอัตโนมัติ

พูดง่าย ๆ **`git-flow` ไม่ใช่ฟีเจอร์ในตัว Git เอง** แต่เป็นโปรแกรมเสริมแยกต่างหากที่ต้องติดตั้งเพิ่ม (คล้ายกับ plugin) เพื่อความสะดวก — คุณยังสามารถทำ Git Flow ได้ครบถ้วนด้วยคำสั่ง Git ธรรมดาอย่างที่แสดงใน Step 313–315 โดยไม่ต้องติดตั้งอะไรเพิ่มเลยก็ได้

### การติดตั้ง git-flow

```bash
# macOS (ผ่าน Homebrew)
brew install git-flow-avh

# Ubuntu / Debian
sudo apt-get install git-flow

# Windows (ผ่าน Git Bash ที่มาพร้อม Git for Windows มักจะมีอยู่แล้ว
# หรือติดตั้งผ่าน Chocolatey)
choco install gitflow-avh
```

> หมายเหตุ: มีสอง fork หลักของเครื่องมือนี้ คือ `gitflow` ต้นฉบับของ nvie และ `git-flow-avh` ซึ่งเป็น fork ที่ได้รับการดูแลต่อเนื่องมากกว่าและเป็นที่นิยมมากกว่าในปัจจุบัน คำสั่งที่ใช้งานเหมือนกันทุกประการ

### เริ่มต้นใช้งานด้วย `git flow init`

```bash
git flow init
```

คำสั่งนี้จะถามคำถามชุดหนึ่งเพื่อกำหนดชื่อ branch หลักและ prefix ต่าง ๆ:

```
Which branch should be used for bringing forth production releases?
   - main
Branch name for production releases: [main]

Which branch should be used for integration of the "next release"?
   - develop
Branch name for "next release" development: [develop]

How to name your supporting branch prefixes?
Feature branches? [feature/]
Bugfix branches? [bugfix/]
Release branches? [release/]
Hotfix branches? [hotfix/]
Support branches? [support/]
Version tag prefix? []
```

กด Enter ผ่านค่าเริ่มต้นได้เลยถ้าไม่ต้องการปรับแต่งอะไรพิเศษ หลังจากนี้ `git-flow` จะรู้จักโครงสร้าง branch ของโปรเจกต์คุณและพร้อมใช้คำสั่งลัดทั้งหมด

### ตารางเปรียบเทียบคำสั่งดิบกับคำสั่ง git-flow

| งานที่ต้องการทำ | คำสั่ง Git ดิบ (ตามที่เรียนใน Step 313–315) | คำสั่ง git-flow (ลัดกว่ามาก) |
|---|---|---|
| เริ่มฟีเจอร์ใหม่ | `git checkout develop`<br>`git checkout -b feature/xyz` | `git flow feature start xyz` |
| จบฟีเจอร์ (merge กลับ develop + ลบ branch) | `git checkout develop`<br>`git merge --no-ff feature/xyz`<br>`git branch -d feature/xyz` | `git flow feature finish xyz` |
| เริ่ม release | `git checkout develop`<br>`git checkout -b release/1.2.0` | `git flow release start 1.2.0` |
| จบ release (merge เข้า main + develop, tag, ลบ branch) | คำสั่งหลายบรรทัดตาม Step 314 | `git flow release finish 1.2.0` |
| เริ่ม hotfix | `git checkout main`<br>`git checkout -b hotfix/1.2.1` | `git flow hotfix start 1.2.1` |
| จบ hotfix (merge เข้า main + develop, tag, ลบ branch) | คำสั่งหลายบรรทัดตาม Step 315 | `git flow hotfix finish 1.2.1` |

### ตัวอย่างการใช้งานจริงแบบเต็มวงจร

```bash
# --- Feature ---
git flow feature start payment-gateway
# ... commit งานไปเรื่อย ๆ บน feature/payment-gateway ...
git flow feature finish payment-gateway
# ผลลัพธ์: merge เข้า develop, สลับกลับไปที่ develop, ลบ feature branch ให้อัตโนมัติ

# --- Release ---
git flow release start 1.2.0
# ... bump version, แก้ bug เล็กน้อย, commit ...
git flow release finish 1.2.0
# ผลลัพธ์: merge เข้า main พร้อมสร้าง tag v1.2.0, merge กลับเข้า develop,
#          ลบ release branch, สลับกลับไปที่ develop ให้อัตโนมัติ
# (คำสั่งนี้จะเปิด text editor ให้ใส่ tag message ด้วย)

git push origin main develop --tags   # อย่าลืม push ทั้งสอง branch และ tag ขึ้น remote

# --- Hotfix ---
git flow hotfix start 1.2.1
# ... แก้บั๊กด่วน, commit ...
git flow hotfix finish 1.2.1
# ผลลัพธ์: merge เข้า main พร้อม tag v1.2.1, merge กลับเข้า develop,
#          ลบ hotfix branch ให้อัตโนมัติ

git push origin main develop --tags
```

### ข้อควรระวังเมื่อใช้ git-flow extension

1. **`git flow finish` ไม่ได้ push ขึ้น remote ให้อัตโนมัติ** — ต้อง `git push` เองเสมอหลังจากนั้น (ทั้ง `main`, `develop` และ tag)
2. **เครื่องมือนี้เป็นแค่ตัวช่วย ไม่ใช่มาตรฐานที่ทุกทีมต้องติดตั้ง** — หลายทีมเลือกใช้คำสั่ง Git ดิบตามขั้นตอนที่เรียนใน Step 313–315 เพราะควบคุมได้ละเอียดกว่า และไม่ต้องพึ่งพาเครื่องมือเสริมที่ทุกคนในทีมต้องติดตั้งเหมือนกัน
3. **การเข้าใจคำสั่งดิบเบื้องหลังยังจำเป็นเสมอ** เพราะถ้าเกิด conflict ระหว่าง merge หรือปัญหาซับซ้อน คุณต้องแก้ปัญหาด้วยคำสั่ง Git มาตรฐานอยู่ดี เครื่องมือ git-flow ช่วยได้แค่ในกรณีที่ทุกอย่างราบรื่นเท่านั้น

---

## Step 318: ข้อดีข้อเสียของ Git Flow ในโลกการทำงานจริง

### ข้อดีของ Git Flow

1. **โครงสร้างชัดเจน คาดเดาได้** — ทุกคนในทีมรู้ทันทีว่าโค้ดชิ้นไหนอยู่ในสถานะไหน เพียงแค่ดูว่ามันอยู่บน branch ประเภทใด
2. **แยกงาน "กำลังพัฒนา" กับ "พร้อมใช้งานจริง" ออกจากกันอย่างเด็ดขาด** — `main` ไม่มีวันมีโค้ดที่ยังไม่พร้อม
3. **รองรับการทำงานแบบหลายเวอร์ชันคู่ขนาน** ได้ดีมาก (ผ่าน release, hotfix และ support branch)
4. **เหมาะกับทีมใหญ่ที่ต้องมีขั้นตอน sign-off / QA อย่างเป็นทางการ** ก่อนปล่อยแต่ละเวอร์ชัน — release branch คือช่วงเวลาที่ QA team สามารถทดสอบได้อย่างสบายใจโดยไม่ถูกฟีเจอร์ใหม่มารบกวน
5. **ประวัติ (history) อ่านง่ายเมื่อใช้ `--no-ff`** — มองย้อนกลับไปเห็นชัดว่าฟีเจอร์ไหนถูกพัฒนาช่วงไหน เข้า release ไหน
6. **มีกลไกรองรับเหตุฉุกเฉิน (hotfix) ที่ไม่รบกวนงานพัฒนาปกติ** อย่างเป็นระบบ

### ข้อเสียของ Git Flow

1. **ซับซ้อนเกินความจำเป็นสำหรับทีมเล็กหรือโปรเจกต์ที่ deploy บ่อย** — ทีมที่ทำ **Continuous Deployment** (ปล่อยโค้ดขึ้น production วันละหลายครั้ง) จะรู้สึกว่าขั้นตอน branch, release, tag มากมายเป็นภาระที่ไม่จำเป็น เพราะไม่มีแนวคิดเรื่อง "เวอร์ชัน" ที่ต้องรอสะสมฟีเจอร์อยู่แล้ว
2. **`develop` กับ `main` อาจห่างกันมากจนเกิด merge conflict ขนาดใหญ่** — ถ้าทีมไม่ merge release เข้า develop บ่อย ๆ การพัฒนาบน develop อาจเบี่ยงเบนไปไกลจน merge ลำบากในภายหลัง
3. **มี branch ที่ต้องดูแลจำนวนมาก** — feature, release, hotfix, support พร้อมกันในโปรเจกต์เดียว ถ้าทีมไม่มีวินัยพอ จะเกิดความสับสนได้ง่าย
4. **ขั้นตอนการ merge สองทาง (main + develop) เป็นจุดที่มนุษย์ทำพลาดได้ง่าย** อย่างที่กล่าวใน Step 314 — การลืม merge กลับเข้า develop เป็นปัญหาที่เกิดขึ้นจริงบ่อยมากในทีมที่เพิ่งเริ่มใช้โมเดลนี้
5. **ไม่เข้ากันกับปรัชญา Trunk-Based Development** ที่อุตสาหกรรมยุคใหม่ (โดยเฉพาะบริษัทเทคโนโลยีขนาดใหญ่อย่าง Google, Facebook) นิยมใช้ ซึ่งเน้นให้ทุกคน commit เข้า branch หลักเส้นเดียวบ่อย ๆ แทนที่จะแยก branch นาน ๆ
6. **เพิ่มต้นทุนการเรียนรู้ (learning curve) ให้กับสมาชิกใหม่ในทีม** — ต้องอธิบายกฎทั้งห้าประเภท branch ก่อนที่จะเริ่มทำงานได้อย่างถูกต้อง

### เสียงจากอุตสาหกรรมจริง

ในปี 2020 เป็นต้นมา มีการวิพากษ์วิจารณ์ Git Flow มากขึ้นเรื่อย ๆ จากวิศวกรที่ทำงานด้าน DevOps และ Continuous Delivery พวกเขาชี้ให้เห็นว่า Git Flow ถูกออกแบบขึ้นในบริบทของปี 2010 ซึ่งเป็นยุคที่ **การ release ซอฟต์แวร์ยังเป็นกิจกรรมที่เกิดขึ้นไม่บ่อย** (เช่นทุก 2-4 สัปดาห์ หรือนานกว่านั้น) แต่ปัจจุบันหลายทีมสามารถ deploy ได้ **หลายสิบครั้งต่อวัน** ด้วยระบบ CI/CD อัตโนมัติ ทำให้แนวคิดเรื่อง "release branch ที่ต้องเตรียมตัวก่อนปล่อย" กลายเป็นคอขวดที่ไม่จำเป็นอีกต่อไป

นี่คือเหตุผลที่โมเดลอย่าง **GitHub Flow** (ซึ่งเราจะเรียนใน **Part 33**) และ **Trunk-Based Development** (ซึ่งเราจะเรียนใน Part ถัดไปหลังจากนั้น) ถือกำเนิดขึ้นมาเพื่อตอบโจทย์ทีมที่ต้องการความเร็วมากกว่าโครงสร้างที่ตายตัว

---

## Step 319: เมื่อไหร่ควรใช้ Git Flow เมื่อไหร่ไม่ควรใช้ (Decision Guide)

### เกณฑ์การตัดสินใจแบบเป็นขั้นตอน

ลองถามคำถามเหล่านี้กับโปรเจกต์ของคุณตามลำดับ:

**คำถามที่ 1: โปรเจกต์นี้ deploy ขึ้น production บ่อยแค่ไหน**

- ถ้า deploy **วันละหลายครั้งหรือทุกครั้งที่ merge PR** → Git Flow ไม่เหมาะ ให้ดู GitHub Flow แทน
- ถ้า deploy **เป็นรอบ ๆ ชัดเจน** (สัปดาห์ละครั้ง, เดือนละครั้ง, หรือนานกว่านั้น) → Git Flow อาจเหมาะ ไปคำถามถัดไป

**คำถามที่ 2: ซอฟต์แวร์นี้มีแนวคิดเรื่อง "เวอร์ชัน" ที่ผู้ใช้รับรู้ได้ชัดเจนหรือไม่**

- ถ้า **ใช่** (เช่น mobile app ที่ผู้ใช้ต้องอัปเดตเอง, firmware, ซอฟต์แวร์ enterprise ที่ขายเป็น license ต่อเวอร์ชัน) → Git Flow เหมาะมาก
- ถ้า **ไม่ใช่** (เว็บแอปพลิเคชันที่ผู้ใช้เห็นแค่ URL เดียวเสมอ ไม่มีแนวคิดเรื่องเวอร์ชัน) → Git Flow อาจเกินความจำเป็น

**คำถามที่ 3: ต้อง maintain หลายเวอร์ชันพร้อมกันหรือไม่**

- ถ้า **ใช่** (มีลูกค้าใช้เวอร์ชันเก่าที่ยังต้อง patch อยู่) → Git Flow เหมาะมาก เพราะมี hotfix และ support branch รองรับโดยเฉพาะ
- ถ้า **ไม่ใช่** (มีแค่เวอร์ชันล่าสุดเวอร์ชันเดียวที่ใช้งานจริงเสมอ) → พิจารณาโมเดลที่เรียบง่ายกว่า

**คำถามที่ 4: ทีมมีขนาดใหญ่แค่ไหน และต้องการขั้นตอน sign-off ก่อน release หรือไม่**

- ถ้าทีมใหญ่ มีหลายฝ่าย (dev, QA, product) ที่ต้องอนุมัติก่อนปล่อยแต่ละเวอร์ชันอย่างเป็นทางการ → Git Flow ให้โครงสร้างที่รองรับกระบวนการนี้ได้ดี
- ถ้าทีมเล็ก ตัดสินใจเร็ว ไม่มีขั้นตอนอนุมัติซับซ้อน → โมเดลที่เบากว่าจะคล่องตัวกว่ามาก

### ตารางสรุปแบบเปรียบเทียบชัดเจน

| สถานการณ์ | ควรใช้ Git Flow | ไม่ควรใช้ Git Flow |
|---|---|---|
| Mobile app (iOS/Android) ที่ต้องผ่าน App Store review | ✅ เหมาะมาก | |
| ซอฟต์แวร์ enterprise แบบ on-premise ที่ขาย license เป็นเวอร์ชัน | ✅ เหมาะมาก | |
| Firmware / driver ของฮาร์ดแวร์ | ✅ เหมาะมาก | |
| SaaS/เว็บแอปที่ deploy อัตโนมัติทุกครั้งที่ merge (Continuous Deployment) | | ❌ ซับซ้อนเกินจำเป็น |
| Startup ทีมเล็กที่ต้องการความเร็วสูงสุด | | ❌ ควรใช้ GitHub Flow แทน |
| Open source library ที่ maintain หลาย major version พร้อมกัน | ✅ เหมาะ (โดยเฉพาะ hotfix/support) | |
| Internal tool เล็ก ๆ ที่มีคนใช้ไม่กี่คน | | ❌ overkill ใช้ trunk-based ธรรมดาก็พอ |

### หลักการจำง่าย ๆ

> **ถ้าคุณต้องเขียนคำว่า "เวอร์ชัน x.y.z" ลงในเอกสารส่งมอบงานให้ลูกค้าหรือผู้ใช้บ่อย ๆ Git Flow คือเพื่อนที่ดีของคุณ แต่ถ้างานของคุณคือเว็บที่อัปเดตต่อเนื่องไม่มีวันหยุดโดยไม่มีใครถามหาเลขเวอร์ชัน ให้มองหาโมเดลที่เรียบง่ายกว่านี้**

ในความเป็นจริง หลายทีมก็เลือก **ผสมผสาน** — ใช้แนวคิดของ Git Flow บางส่วน (เช่น hotfix branch, การติด tag) แต่ตัดส่วนที่ซับซ้อนออก (เช่นไม่มี `develop` แยกต่างหาก) นี่ไม่ใช่เรื่องผิด — Git Flow ควรถูกมองเป็น **จุดเริ่มต้นสำหรับปรับให้เข้ากับทีมของตัวเอง** ไม่ใช่กฎที่ต้องทำตามทุกตัวอักษรเสมอไป

---

## Step 320: แบบฝึกหัด — จำลองวงจร release เต็มรูปแบบด้วย Git Flow

ถึงเวลาลงมือทำจริง ในแบบฝึกหัดนี้คุณจะจำลองสถานการณ์การทำงานแบบทีมจริง ครบทั้ง feature, release และ hotfix โดยใช้คำสั่ง Git ดิบ (ไม่ใช้ extension) เพื่อให้เข้าใจกลไกเบื้องหลังอย่างถ่องแท้

### เตรียมโปรเจกต์ฝึกฝน

```bash
mkdir ~/git-course/part-32-gitflow
cd ~/git-course/part-32-gitflow
git init
echo "# ระบบจัดการงาน (Task Manager)" > README.md
echo "version: 1.0.0" > VERSION
git add .
git commit -m "Initial commit: เริ่มต้นโปรเจกต์"
git branch -M main
```

### ขั้นที่ 1: สร้าง `develop` จาก `main`

```bash
git checkout -b develop
git log --oneline --all --graph
```

ตรวจสอบว่าตอนนี้ `main` และ `develop` ชี้ไปที่ commit เดียวกัน

### ขั้นที่ 2: จำลองการพัฒนา 2 ฟีเจอร์พร้อมกัน

```bash
# ฟีเจอร์ที่ 1: ระบบ login
git checkout develop
git checkout -b feature/login
echo "function login() { /* TODO */ }" > login.js
git add login.js
git commit -m "เพิ่มฟังก์ชัน login เบื้องต้น"

# ฟีเจอร์ที่ 2: ระบบเพิ่มงาน (แตกจาก develop คู่ขนานกัน)
git checkout develop
git checkout -b feature/add-task
echo "function addTask() { /* TODO */ }" > task.js
git add task.js
git commit -m "เพิ่มฟังก์ชัน addTask เบื้องต้น"
```

### ขั้นที่ 3: จบทั้งสองฟีเจอร์ กลับเข้า `develop`

```bash
git checkout develop
git merge --no-ff feature/login -m "Merge feature/login into develop"
git branch -d feature/login

git merge --no-ff feature/add-task -m "Merge feature/add-task into develop"
git branch -d feature/add-task

git log --oneline --all --graph
```

**คาดหวังผลลัพธ์:** คุณควรเห็น `develop` มี merge commit สองอัน แต่ละอันแทนหนึ่งฟีเจอร์ที่จบแล้ว

### ขั้นที่ 4: เตรียม Release 1.1.0

```bash
git checkout develop
git checkout -b release/1.1.0

echo "version: 1.1.0" > VERSION
git commit -am "Bump version เป็น 1.1.0"

# จำลองว่าเจอ bug เล็กน้อยตอนทดสอบ
echo "function login() { console.log('login สำเร็จ'); }" > login.js
git commit -am "แก้ไข bug เล็กน้อยในฟังก์ชัน login ที่พบระหว่างทดสอบ release"
```

### ขั้นที่ 5: จบ Release — merge เข้าทั้ง `main` และ `develop`

```bash
# merge เข้า main พร้อม tag
git checkout main
git merge --no-ff release/1.1.0 -m "Merge release/1.1.0 into main"
git tag -a v1.1.0 -m "Release เวอร์ชัน 1.1.0: เพิ่มระบบ login และ add task"

# merge กลับเข้า develop (ห้ามลืมขั้นตอนนี้!)
git checkout develop
git merge --no-ff release/1.1.0 -m "Merge release/1.1.0 into develop"

git branch -d release/1.1.0
git log --oneline --all --graph
```

**คาดหวังผลลัพธ์:** `main` ควรมี tag `v1.1.0` และมีการแก้ไข bug เดียวกันนั้นควรปรากฏอยู่ทั้งใน `main` และ `develop`

### ขั้นที่ 6: จำลองสถานการณ์ฉุกเฉิน — Hotfix

สมมติว่าหลังจาก release ไปแล้ว 2 วัน มีลูกค้าแจ้งว่าระบบ login มีบั๊กร้ายแรง ในขณะที่ทีมกำลังพัฒนาฟีเจอร์ใหม่ต่อบน `develop` อยู่พอดี

```bash
# จำลองว่า develop เดินหน้าต่อไปแล้ว (ฟีเจอร์ใหม่ที่ยังไม่เสร็จ)
git checkout develop
echo "function editTask() { /* ยังทำไม่เสร็จ */ }" > edit-task.js
git add edit-task.js
git commit -m "เริ่มพัฒนาฟีเจอร์ edit task (ยังไม่เสร็จ)"

# เกิดบั๊กด่วนบน production — แตก hotfix จาก main โดยตรง ไม่ใช่จาก develop
git checkout main
git checkout -b hotfix/1.1.1
echo "function login() { console.log('login สำเร็จ (แก้ไข null check แล้ว)'); }" > login.js
git commit -am "แก้ไขบั๊กร้ายแรง: login ล่มเมื่อ username เป็นค่าว่าง"
echo "version: 1.1.1" > VERSION
git commit -am "Bump version เป็น 1.1.1"
```

### ขั้นที่ 7: จบ Hotfix — merge เข้าทั้ง `main` และ `develop`

```bash
# merge เข้า main พร้อม tag
git checkout main
git merge --no-ff hotfix/1.1.1 -m "Merge hotfix/1.1.1 into main"
git tag -a v1.1.1 -m "Hotfix: แก้ไขบั๊กร้ายแรงในระบบ login"

# merge กลับเข้า develop ด้วย — สังเกตว่า develop มีงานฟีเจอร์ edit-task ค้างอยู่
# แต่ก็ไม่เป็นปัญหา เพราะ merge จะรวมทั้งสองส่วนเข้าด้วยกัน
git checkout develop
git merge --no-ff hotfix/1.1.1 -m "Merge hotfix/1.1.1 into develop"

git branch -d hotfix/1.1.1
```

### ขั้นที่ 8: ตรวจสอบผลลัพธ์สุดท้ายทั้งหมด

```bash
git log --oneline --all --graph --decorate
git tag
```

ตรวจสอบว่า:

1. `main` มี tag `v1.1.0` และ `v1.1.1` ตามลำดับ
2. ไฟล์ `login.js` บน `main` มีทั้งการแก้ไขจาก release และจาก hotfix
3. `develop` มีทั้งฟีเจอร์ `edit-task` ที่ยังไม่เสร็จ **และ** การแก้ไข hotfix ล่าสุดอยู่ด้วย (สังเกตว่า hotfix ไม่ได้ไปรบกวนฟีเจอร์ที่ยังพัฒนาไม่เสร็จเลย)
4. ไม่มี branch ประเภท `feature/*`, `release/*`, `hotfix/*` หลงเหลืออยู่ (ทุกอันถูกลบหลังจบงานแล้ว)

### คำถามทบทวนหลังทำแบบฝึกหัด (ลองตอบเองก่อนเปิดดูคำตอบ)

1. ทำไม hotfix ถึงต้องแตกจาก `main` ไม่ใช่จาก `develop`
   > **คำตอบ:** เพราะ ณ ตอนที่เกิดปัญหา `develop` อาจมีโค้ดที่ยังพัฒนาไม่เสร็จอยู่ (เช่น `edit-task.js` ในแบบฝึกหัดนี้) การแตกจาก `main` ทำให้ hotfix มีเฉพาะโค้ดที่กำลังใช้งานจริงบน production เท่านั้น ไม่ปนกับงานที่ยังไม่พร้อม

2. ถ้าคุณลืม merge `release/1.1.0` กลับเข้า `develop` ในขั้นที่ 5 จะเกิดอะไรขึ้นกับการแก้ไข bug เล็กน้อยที่ทำไว้บน release branch
   > **คำตอบ:** การแก้ไขนั้นจะหายไปจาก `develop` ทันที เพราะ `develop` ไม่เคยรู้จักการเปลี่ยนแปลงนั้นเลย ถ้าฟีเจอร์ถัดไปที่พัฒนาบน `develop` ไปแตะไฟล์เดียวกัน bug ตัวเดิมอาจย้อนกลับมาอีกครั้งในเวอร์ชันถัดไป

3. เพราะเหตุใดเราจึงใช้ `--no-ff` ในทุกคำสั่ง merge ตลอดแบบฝึกหัดนี้
   > **คำตอบ:** เพื่อบังคับให้ Git สร้าง merge commit เสมอ แม้จะสามารถ fast-forward ได้ก็ตาม ทำให้ประวัติยังคงเห็นจุดแตกและจุดรวมของแต่ละ feature/release/hotfix ได้อย่างชัดเจน ไม่ถูกกลืนหายไปในสายเวลาเดียวกับ commit อื่น

### แบบฝึกหัดเสริม (ถ้ามีเวลา)

ลองทำซ้ำแบบฝึกหัดนี้อีกครั้งโดยใช้เครื่องมือ `git-flow` extension จาก Step 317 แทนคำสั่งดิบทั้งหมด แล้วเปรียบเทียบว่าจำนวนคำสั่งที่ต้องพิมพ์ต่างกันแค่ไหน และคุณรู้สึกว่าแบบไหนทำให้เข้าใจกลไกเบื้องหลังได้ดีกว่ากัน

---

## สรุป Part 32

ใน Part นี้เราได้เรียนรู้ Git Flow อย่างละเอียดครบทุกมิติ:

1. **Git Flow** ถูกเสนอโดย **Vincent Driessen** ในปี 2010 เป็นโมเดลการแตก branch ที่มีโครงสร้างชัดเจน เหมาะกับโปรเจกต์ที่มีเลขเวอร์ชันและรอบ release ที่ชัดเจน
2. โมเดลนี้มี branch หลัก 2 เส้นที่อยู่ตลอดไป คือ **`main`** (เก็บโค้ด production เท่านั้น) และ **`develop`** (จุดรวมงานของทั้งทีม)
3. **Feature branch** แตกจาก `develop` กลับเข้า `develop` ใช้พัฒนาฟีเจอร์ใหม่แต่ละอย่างแยกจากกัน
4. **Release branch** แตกจาก `develop` เมื่อฟีเจอร์ครบสำหรับรอบ release และ merge กลับเข้าทั้ง `main` และ `develop` — เป็นพื้นที่กันชนสำหรับ bump version, ทดสอบ, และแก้บั๊กเล็กน้อยก่อนปล่อยจริง
5. **Hotfix branch** แตกจาก `main` โดยตรงเพื่อแก้บั๊กด่วนบน production โดยไม่รบกวนงานที่กำลังพัฒนาอยู่บน `develop` แล้ว merge กลับเข้าทั้งสองฝั่งเช่นกัน
6. **Support branch** ใช้ maintain เวอร์ชันเก่าที่ยังมีผู้ใช้งานอยู่คู่ขนานกับการพัฒนาเวอร์ชันหลัก พบเห็นได้บ่อยในซอฟต์แวร์ enterprise หรือ firmware
7. เครื่องมือ **`git-flow` extension** ช่วยย่นคำสั่งหลายบรรทัดให้เหลือคำสั่งเดียว (`git flow feature/release/hotfix start/finish`) แต่การเข้าใจคำสั่งดิบเบื้องหลังยังจำเป็นเสมอ
8. Git Flow มีทั้งข้อดี (โครงสร้างชัดเจน รองรับหลายเวอร์ชัน เหมาะกับทีมใหญ่) และข้อเสีย (ซับซ้อนเกินไปสำหรับทีมที่ทำ Continuous Deployment, เสี่ยงต่อการลืม merge สองทาง)
9. การเลือกใช้ Git Flow ควรพิจารณาจาก **ความถี่ในการ deploy** และ **การมีอยู่ของแนวคิดเรื่องเวอร์ชัน** ในโปรเจกต์ของคุณเป็นหลัก
10. เราได้ลงมือจำลองวงจร release เต็มรูปแบบด้วยตัวเองแล้ว ตั้งแต่ feature ไปจนถึง release และ hotfix พร้อมเข้าใจจุดที่มือใหม่มักพลาดบ่อยที่สุด

### Checklist ก่อนไป Part 33

ก่อนไปต่อ Part 33 ให้ตรวจสอบว่าคุณ:

- [ ] อธิบายได้ว่า Git Flow คืออะไร ใครสร้าง และเหมาะกับโปรเจกต์แบบไหน
- [ ] แยกแยะหน้าที่ของ `main` และ `develop` ได้อย่างชัดเจน
- [ ] สร้าง merge และจบ feature branch ได้ด้วยคำสั่ง Git ดิบ
- [ ] เข้าใจว่าทำไม release branch ต้อง merge กลับเข้าทั้ง `main` และ `develop`
- [ ] เข้าใจว่าทำไม hotfix ต้องแตกจาก `main` ไม่ใช่จาก `develop`
- [ ] รู้จักหน้าที่ของ support branch แม้จะไม่ค่อยได้ใช้บ่อย
- [ ] ลองใช้คำสั่ง `git flow init`, `git flow feature start/finish` ได้แล้ว (หรืออย่างน้อยเข้าใจว่ามันทำอะไรอยู่เบื้องหลัง)
- [ ] อธิบายข้อดีข้อเสียของ Git Flow ได้ทั้งสองด้าน ไม่ใช่แค่ด้านดี
- [ ] ตัดสินใจได้ว่าโปรเจกต์แบบไหนควรใช้ Git Flow และแบบไหนไม่ควรใช้
- [ ] ทำแบบฝึกหัดจำลองวงจร release เต็มรูปแบบจบครบทุกขั้นตอนแล้ว

**ต่อไป:** [Part 33: GitHub Flow: โมเดลง่ายสำหรับ Continuous Delivery](./part-033-github-flow.md)
