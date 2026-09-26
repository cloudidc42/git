# Part 08: Merge เบื้องต้นและการแก้ Conflict ครั้งแรก

> **Step ในหลักสูตรนี้:** Step 71–80
> **เฟส:** 2 — ใช้งาน Branch, Merge และแก้ปัญหาพื้นฐานที่เจอบ่อยที่สุด
> **เป้าหมายของ Part นี้:** เข้าใจว่า `git merge` ทำงานอย่างไรในระดับกลไกจริง แยกความแตกต่างระหว่าง Fast-forward merge กับ Three-way merge ได้อย่างชัดเจน และที่สำคัญที่สุดคือสามารถเผชิญหน้ากับ **Merge Conflict** ครั้งแรกได้โดยไม่ตื่นตระหนก อ่าน conflict marker ออก แก้ไขได้ถูกต้อง และรู้จักทางออกฉุกเฉินอย่าง `git merge --abort` เมื่อจำเป็น

---

## สารบัญของ Part นี้

- Step 71: `git merge` คืออะไร แนวคิดการรวม branch
- Step 72: Fast-forward merge คืออะไร เกิดขึ้นเมื่อไหร่
- Step 73: Three-way merge (สร้าง merge commit) คืออะไร ต่างจาก Fast-forward อย่างไร
- Step 74: ขั้นตอนการ merge จริง — ทีละคำสั่ง
- Step 75: Merge Conflict คืออะไร ทำไมถึงเกิดขึ้น
- Step 76: อ่านสัญลักษณ์ conflict marker `<<<<<<<` `=======` `>>>>>>>`
- Step 77: ขั้นตอนแก้ conflict ทีละขั้นตอนอย่างละเอียด
- Step 78: การยกเลิก merge ระหว่างมี conflict ด้วย `git merge --abort`
- Step 79: เครื่องมือช่วยแก้ conflict (`git mergetool`, VS Code Merge Editor)
- Step 80: แบบฝึกหัด — จำลองสถานการณ์ conflict จริงและแก้ไขให้สำเร็จ

---

## Step 71: `git merge` คืออะไร แนวคิดการรวม branch

ใน Part ก่อนหน้าเราเรียนรู้วิธีสร้าง branch เพื่อแยกไปทำงานคู่ขนานกันโดยไม่กระทบ branch หลัก แต่สุดท้ายแล้ว งานที่ทำใน branch แยกออกไปนั้นก็ต้อง **"กลับมารวมกัน"** กับ branch หลักในสักวันหนึ่ง — นี่คือหน้าที่ของ `git merge`

> **Merge คือการนำประวัติ (commit history) จาก branch หนึ่ง เข้าไปรวมกับอีก branch หนึ่ง โดยที่ผลลัพธ์สุดท้ายจะมีการเปลี่ยนแปลงของทั้งสอง branch ครบถ้วน**

พูดให้เห็นภาพง่าย ๆ ลองนึกถึงการเขียนหนังสือร่วมกัน 2 คน:

- คุณ A แยกไปเขียนบทที่ 5 ในสำเนาของตัวเอง (branch `feature-chapter5`)
- คุณ B ยังคงแก้ไขต้นฉบับหลักต่อไป (branch `main`)
- เมื่อคุณ A เขียนบทที่ 5 เสร็จ ก็ต้องเอาบทที่ 5 นั้นไป **"รวม"** เข้ากับต้นฉบับหลักของคุณ B

การ "รวม" นี้คือสิ่งที่ `git merge` ทำให้อัตโนมัติ โดยอาศัยกลไกที่ฉลาดมาก คือมันจะเทียบ **เนื้อหาของแต่ละบรรทัด** ไม่ใช่แค่เทียบชื่อไฟล์

### คำสั่งพื้นฐาน

```bash
git merge <ชื่อ-branch-ที่ต้องการดึงเข้ามา>
```

หลักการสำคัญที่ต้องจำให้ขึ้นใจคือ:

> **`git merge` จะรวมการเปลี่ยนแปลง "เข้าหา" branch ที่คุณกำลังยืนอยู่ (current branch / HEAD) เสมอ ไม่ใช่ในทางกลับกัน**

ตัวอย่างเช่น ถ้าคุณอยู่ที่ branch `main` แล้วรัน `git merge feature-login` ความหมายคือ:

> "เอาการเปลี่ยนแปลงทั้งหมดจาก `feature-login` มารวมเข้ากับ `main` (ซึ่งเป็น branch ที่ฉันยืนอยู่ตอนนี้)"

ไดอะแกรมภาพรวมก่อน merge:

```
              A---B---C  (main)
               \
                D---E   (feature-login)

HEAD -> main
```

หลังจากสั่ง `git merge feature-login` ขณะยืนอยู่ที่ `main`:

```
              A---B---C-------F  (main)   <- merge commit ใหม่
               \             /
                D-----------E   (feature-login)

HEAD -> main
```

### สองรูปแบบหลักของการ merge ใน Git

Git จะเลือกวิธี merge ให้อัตโนมัติจากลักษณะของประวัติ (history) ของทั้งสอง branch โดยไม่ต้องให้คุณสั่งเลือกเอง มี 2 แบบหลัก:

| รูปแบบ | เกิดขึ้นเมื่อไหร่ | สร้าง commit ใหม่ไหม |
|---|---|---|
| **Fast-forward merge** | branch ปลายทางไม่มี commit ใหม่เลยนับตั้งแต่แยก branch ออกมา | ไม่สร้าง (แค่เลื่อน pointer) |
| **Three-way merge** | ทั้งสอง branch ต่างมี commit ใหม่ของตัวเองหลังจากจุดแยก (diverged) | สร้าง merge commit ใหม่ |

เราจะเจาะลึกทั้งสองแบบใน Step 72 และ 73 ตามลำดับ เพราะการเข้าใจความต่างของสองแบบนี้คือกุญแจสำคัญที่ทำให้คุณเข้าใจว่า **conflict เกิดได้เฉพาะกับ three-way merge เท่านั้น** ไม่มีทางเกิดกับ fast-forward merge เลย

### ทำไมต้องเข้าใจกลไกเบื้องหลัง ไม่ใช่แค่จำคำสั่ง

หลายคนเรียน Git แบบท่องคำสั่งอย่างเดียวโดยไม่เข้าใจว่าเบื้องหลังเกิดอะไรขึ้น พอเจอ conflict ครั้งแรกก็ตกใจและไม่รู้จะเริ่มแก้ตรงไหน ใน Part นี้เราจะอธิบายกลไกจริงทีละขั้น เพื่อให้เมื่อคุณเจอ conflict ในงานจริง คุณจะไม่ใช่แค่ "ทำตามขั้นตอนแบบท่องจำ" แต่ **เข้าใจว่าทำไม Git ถึงบอกว่ามี conflict ตรงจุดนี้**

---

## Step 72: Fast-forward merge คืออะไร เกิดขึ้นเมื่อไหร่

### นิยาม

> **Fast-forward merge คือการ merge ที่ Git ไม่ต้องสร้าง commit ใหม่เลย เพียงแค่ "เลื่อน pointer" ของ branch ปลายทางไปชี้ที่ commit ล่าสุดของ branch ที่กำลัง merge เข้ามาเท่านั้น**

เงื่อนไขที่ทำให้เกิด fast-forward merge ได้คือ:

> **commit ปัจจุบันของ branch ปลายทาง (HEAD) ต้องเป็น "บรรพบุรุษโดยตรง" (direct ancestor) ของ commit ปลายสุดของ branch ที่จะ merge เข้ามา**

พูดง่าย ๆ คือ **ตั้งแต่วันที่คุณแยก branch ออกไป จนถึงตอนนี้ ไม่มีใครไปแก้ไข commit อะไรเพิ่มเติมใน branch ปลายทางเลยแม้แต่ commit เดียว** — branch ปลายทางเหมือน "หยุดนิ่งรอ" อยู่ตรงจุดเดิม

### ไดอะแกรมก่อน merge

```
                          (main, HEAD)
                               │
              A───────B────────┘
                       \
                        C───D───E  (feature-x)
```

จากภาพ: `main` หยุดอยู่ที่ commit `B` มาตลอด ในขณะที่ `feature-x` เดินหน้าต่อไปเป็น `C → D → E` โดยที่ `B` เป็นบรรพบุรุษของ `E` อย่างชัดเจน (เดินตามเส้นจาก `B` ไปหา `E` ได้โดยตรง)

### สั่ง merge

```bash
git switch main
git merge feature-x
```

ผลลัพธ์ที่ terminal จะแสดง:

```
Updating b1a2c3d..e4f5g6h
Fast-forward
 login.js | 12 ++++++++++++
 1 file changed, 12 insertions(+)
```

สังเกตคำว่า **"Fast-forward"** ที่ Git แจ้งให้ทราบตรง ๆ — นี่คือสัญญาณยืนยันว่าเกิด fast-forward merge จริง

### ไดอะแกรมหลัง merge

```
              A───────B───────C───D───E  (main, feature-x, HEAD)
```

สังเกตว่า:

1. **ไม่มี commit ใหม่เกิดขึ้นเลย** — จำนวน commit ทั้งหมดเท่าเดิม
2. **`main` แค่ขยับ pointer ไปชี้ที่ตำแหน่งเดียวกับ `E`** เหมือนเอาไม้บรรทัดมาวางทาบเฉย ๆ
3. ประวัติ (history) กลายเป็นเส้นตรงเส้นเดียว (linear history) ไม่มีการแตกแขนงให้เห็นเลยใน `git log`

### ทำไมถึงเรียกว่า "Fast" (เร็ว)

เพราะ Git ไม่ต้องคำนวณอะไรเลย ไม่ต้องเปรียบเทียบเนื้อหาไฟล์ ไม่ต้องหาจุดร่วม ไม่ต้องสร้าง object ใหม่ใด ๆ ในระบบ — แค่เปลี่ยนค่าตัวเลข (pointer) ของ branch reference ให้ชี้ไปที่ commit hash ใหม่เท่านั้น ซึ่งใช้เวลาแทบจะเป็นศูนย์

### การบังคับพฤติกรรมของ Fast-forward merge

Git มี flag พิเศษให้ควบคุมพฤติกรรมนี้ได้:

| คำสั่ง | ความหมาย |
|---|---|
| `git merge feature-x` | ปล่อยให้ Git เลือกอัตโนมัติ (fast-forward ถ้าทำได้) |
| `git merge --ff-only feature-x` | **บังคับ**ให้ merge แบบ fast-forward เท่านั้น ถ้าทำไม่ได้ (เพราะ history แยกกันแล้ว) จะ **ล้มเหลวทันที** แทนที่จะสร้าง merge commit |
| `git merge --no-ff feature-x` | **บังคับ**ให้สร้าง merge commit เสมอ แม้ว่าเงื่อนไข fast-forward จะเข้าเงื่อนไขก็ตาม |

`--no-ff` เป็นที่นิยมมากในทีมงานจริง เพราะช่วยให้ **มองเห็นได้ชัดเจนใน history ว่ามี branch ไหนถูก merge เข้ามาบ้าง** แม้ว่าในทางเทคนิคจะทำ fast-forward ได้ก็ตาม เราจะพูดถึงเรื่องนี้ลึกขึ้นในเรื่อง Git Workflow ใน Part หลัง ๆ (Part 27 เป็นต้นไป)

```bash
# บังคับสร้าง merge commit เสมอ แม้ทำ fast-forward ได้
git merge --no-ff feature-x
```

ผลลัพธ์คือแม้ history จะเป็นเส้นตรง Git ก็จะสร้าง merge commit ปลอมขึ้นมา (ที่มี 2 parent เหมือน three-way merge) เพื่อเก็บร่องรอยว่าเคยมีการ merge branch นี้เข้ามาจริง ๆ

### สรุปสั้น ๆ ของ Step นี้

> **Fast-forward merge เกิดขึ้นเมื่อ branch ปลายทางไม่ได้เดินหน้าไปไหนเลยตั้งแต่แยก branch — Git จึงแค่ "เลื่อนป้าย" ไปยังจุดใหม่ ไม่มีการสร้าง commit ใหม่ และ**ไม่มีทางเกิด conflict ได้เลยในกรณีนี้**

---

## Step 73: Three-way merge (สร้าง merge commit) เกิดขึ้นเมื่อไหร่ ต่างจาก Fast-forward อย่างไร

### นิยาม

> **Three-way merge คือการ merge ที่เกิดขึ้นเมื่อทั้ง branch ปลายทางและ branch ที่กำลังจะ merge เข้ามา ต่างมี commit ใหม่เป็นของตัวเอง (diverged history) นับตั้งแต่จุดที่แยกออกจากกัน**

คำว่า "three-way" มาจากการที่ Git ต้องใช้ **3 จุดอ้างอิง** ในการคำนวณผลลัพธ์การรวม:

1. **Common Ancestor (Base)** — commit ล่าสุดที่ทั้งสอง branch เคยมีร่วมกันก่อนแยกทาง
2. **Tip ของ branch ปัจจุบัน (Ours)** — commit ล่าสุดของ branch ที่คุณยืนอยู่ตอนนี้
3. **Tip ของ branch ที่จะ merge เข้ามา (Theirs)** — commit ล่าสุดของ branch เป้าหมายที่ระบุใน `git merge`

### ไดอะแกรมก่อน merge

```
                       (main, HEAD)
                            │
              A───B────C────D
                   \
                    E────F────G  (feature-y)
```

จากภาพนี้ **`B` คือ Common Ancestor** ของทั้งสอง branch — หลังจากจุด `B` ทั้งสองฝั่งต่างเดินหน้าต่อกันเอง:

- `main` เดินไปเป็น `C → D`
- `feature-y` เดินไปเป็น `E → F → G`

นี่คือลักษณะของ **diverged history** — ประวัติที่แยกออกเป็นสองทางจริง ๆ ไม่ใช่แค่ทางหนึ่งหยุดนิ่งเหมือน fast-forward

### กลไกการคำนวณของ Three-way merge

Git จะนำไฟล์ทั้ง 3 เวอร์ชันมาเทียบกันทีละไฟล์ ทีละบรรทัด:

```
              Base (B)         Ours (D)         Theirs (G)
           ┌───────────┐   ┌───────────┐   ┌───────────┐
           │ line 1    │   │ line 1    │   │ line 1    │
           │ line 2    │   │ line 2    │   │ line 2 (แก้)│
           │ line 3    │   │ line 3(แก้)│   │ line 3    │
           └───────────┘   └───────────┘   └───────────┘
                     │             │
                     └──────┬──────┘
                            ▼
                    ผลลัพธ์การ merge
                 (รวมทั้งสองการแก้ไขเข้าด้วยกัน
                  ถ้าไม่ชนกัน ก็รวมอัตโนมัติได้)
```

หลักการอัตโนมัติของ Git คือ:

- ถ้าบรรทัดใดถูกแก้ไข **ฝั่งเดียว** เมื่อเทียบกับ Base → **รับการแก้ไขนั้นมาใช้ทันทีโดยอัตโนมัติ** ไม่ถามอะไร
- ถ้าบรรทัดใดถูกแก้ไข **ทั้งสองฝั่ง** และแก้เป็นค่าเดียวกัน → รวมได้อัตโนมัติเช่นกัน
- ถ้าบรรทัดใดถูกแก้ไข **ทั้งสองฝั่ง** แต่แก้เป็นค่าต่างกัน → **นี่คือจุดกำเนิดของ Merge Conflict** ที่เราจะเจาะลึกใน Step 75

### สั่ง merge และผลลัพธ์ (กรณีไม่มี conflict)

```bash
git switch main
git merge feature-y
```

ถ้าไม่มี conflict Git จะเปิดข้อความ commit เริ่มต้น (default editor) ให้แก้ไขข้อความ merge commit หรือถ้าไม่อยากแก้ก็ปิดโปรแกรมแก้ไขไปตามปกติ (เช่นบันทึกแล้วออกจาก Vim) ข้อความ default จะเป็นประมาณนี้:

```
Merge branch 'feature-y' into main
```

ผลลัพธ์ที่ terminal แสดง:

```
Merge made by the 'ort' strategy.
 profile.js | 8 +++++++-
 1 file changed, 7 insertions(+), 1 deletion(-)
```

> หมายเหตุ: `ort` คือชื่อ merge strategy เริ่มต้นของ Git ตั้งแต่เวอร์ชัน 2.34 เป็นต้นมา (ย่อมาจาก "Ostensibly Recursive's Twin") ซึ่งมาแทนที่ strategy เก่าชื่อ `recursive` โดยทำงานเร็วกว่าและแม่นยำกว่าในหลายกรณี

### ไดอะแกรมหลัง merge

```
              A───B────C────D────────M   (main, HEAD)
                   \                /
                    E────F─────────G   (feature-y)
```

สังเกตความแตกต่างจาก fast-forward อย่างชัดเจน:

1. **เกิด commit ใหม่ (`M`)** ที่ไม่เคยมีมาก่อน เรียกว่า **Merge Commit**
2. **Merge commit มี parent 2 ตัว** พร้อมกัน (ชี้ไปทั้ง `D` และ `G`) ต่างจาก commit ปกติที่มี parent แค่ 1 ตัว
3. History ไม่เป็นเส้นตรงอีกต่อไป จะเห็นเป็นรูป "เพชร" (diamond shape) เมื่อดูด้วย `git log --graph`

### ตารางเปรียบเทียบ Fast-forward vs Three-way

| คุณสมบัติ | Fast-forward Merge | Three-way Merge |
|---|---|---|
| เงื่อนไขที่เกิด | branch ปลายทางไม่มี commit ใหม่เลย | ทั้งสอง branch มี commit ใหม่ (diverged) |
| สร้าง commit ใหม่ไหม | ไม่สร้าง | สร้างเสมอ (merge commit) |
| จำนวน parent ของผลลัพธ์ | ไม่มี commit ใหม่ให้พูดถึง parent | 2 parents |
| รูปร่างใน `git log --graph` | เส้นตรง (linear) | แตกแขนงแล้วรวมกัน (diamond) |
| มีโอกาสเกิด Conflict ไหม | **ไม่มีทาง** เกิด | **มีโอกาส**เกิดถ้าแก้ไขจุดเดียวกัน |
| ความเร็ว | เร็วมาก (แค่เลื่อน pointer) | ต้องคำนวณเปรียบเทียบไฟล์ |

### ตรวจสอบด้วย `git log --graph`

คำสั่งนี้จะช่วยให้เห็นภาพรวมของ history ทั้งหมดพร้อมเส้นเชื่อม branch แบบภาพ ASCII art:

```bash
git log --graph --oneline --all --decorate
```

ตัวอย่างผลลัพธ์หลังเกิด three-way merge:

```
*   f4e8a21 (HEAD -> main) Merge branch 'feature-y' into main
|\
| * a91c3d0 (feature-y) เพิ่มฟังก์ชันคำนวณโบนัส
| * 7b2f9e1 แก้ไข validation ของฟอร์ม
* | 3c8d5a2 อัปเดต README
* | 1a2b3c4 เพิ่ม unit test
|/
* 9f8e7d6 commit เริ่มต้นของโปรเจกต์
```

คำสั่งนี้จะกลายเป็นเครื่องมือที่คุณใช้บ่อยที่สุดตัวหนึ่งตลอดหลักสูตร แนะนำให้สร้าง alias เก็บไว้ (เราจะสอนเรื่อง alias แบบละเอียดใน Part ถัดไปเกี่ยวกับการตั้งค่า Git)

---

## Step 74: ขั้นตอนการ merge จริง (สลับไป branch ปลายทางก่อน แล้ว merge branch อื่นเข้ามา)

มาถึงขั้นตอนปฏิบัติจริงกันแล้ว หลักการที่ต้อง**จำให้ขึ้นใจที่สุด**ของ `git merge` คือ:

> **ต้องสลับไปยืนอยู่ที่ branch ปลายทาง (ที่ต้องการให้รับการเปลี่ยนแปลงเข้ามา) ก่อนเสมอ แล้วค่อยสั่ง `git merge <ชื่อ branch อีกฝั่ง>`**

คนที่เพิ่งเริ่มเรียน Git มักสับสนตรงนี้บ่อยมาก เพราะคำสั่ง `git merge feature-x` ไม่ได้บอกตรง ๆ ว่า "จะเอาอะไรไปรวมกับอะไร" ต้องดูว่า **ตอนนี้คุณยืนอยู่ที่ branch ไหน** ด้วยเสมอ

### ขั้นตอนแบบเต็มทีละคำสั่ง

สมมติสถานการณ์: คุณมี branch `main` และ `feature-payment` ต้องการเอา `feature-payment` ไปรวมเข้ากับ `main`

**ขั้นตอนที่ 1: ตรวจสอบว่าตอนนี้อยู่ branch ไหน**

```bash
git branch
```

```
  feature-payment
* main
```

เครื่องหมาย `*` บอกว่าตอนนี้อยู่ที่ `main` แล้ว (ถ้ายังไม่ใช่ ให้สลับไปก่อน)

**ขั้นตอนที่ 2: สลับไป branch ปลายทาง (destination branch)**

```bash
git switch main
```

หรือใช้คำสั่งรุ่นเก่า:

```bash
git checkout main
```

> คำแนะนำ: ตั้งแต่ Git 2.23 เป็นต้นมา แนะนำให้ใช้ `git switch` สำหรับสลับ branch เพราะแยกหน้าที่ชัดเจนกว่า `git checkout` ที่ทำได้หลายอย่างปนกัน (เราอธิบายเรื่องนี้ละเอียดแล้วใน Part ก่อนหน้าเรื่อง branch)

**ขั้นตอนที่ 3: ตรวจสอบว่า working directory สะอาด (clean)**

```bash
git status
```

```
On branch main
nothing to commit, working tree clean
```

**สำคัญมาก:** ควร commit หรือ stash งานที่ยังค้างอยู่ก่อนเริ่ม merge เสมอ เพราะถ้ามีไฟล์ที่ยังไม่ commit ค้างอยู่ Git อาจปฏิเสธการ merge หรือทำให้สถานการณ์สับสนมากขึ้นเมื่อเกิด conflict

**ขั้นตอนที่ 4: ดึงข้อมูลล่าสุดจาก remote ก่อน (ถ้าทำงานเป็นทีม)**

```bash
git pull origin main
```

ขั้นตอนนี้สำคัญมากในการทำงานจริงเป็นทีม เพื่อให้แน่ใจว่า `main` ในเครื่องคุณตรงกับ `main` บน server ก่อนจะ merge อะไรเข้าไป (เราจะอธิบายเรื่อง `git pull` และ remote แบบละเอียดใน Part 09 ที่กำลังจะถึง)

**ขั้นตอนที่ 5: สั่ง merge**

```bash
git merge feature-payment
```

**ขั้นตอนที่ 6: อ่านผลลัพธ์**

กรณีสำเร็จแบบไม่มี conflict Git จะแสดงข้อความสรุปไฟล์ที่เปลี่ยนแปลงทันที (ตามที่แสดงใน Step 72 และ 73) กรณีมี conflict จะแสดงข้อความเตือนแบบนี้ (จะอธิบายละเอียดใน Step 75-77):

```
Auto-merging payment.js
CONFLICT (content): Merge conflict in payment.js
Automatic merge failed; fix conflicts and then commit the result.
```

**ขั้นตอนที่ 7: ตรวจสอบผลลัพธ์ด้วย `git log`**

```bash
git log --graph --oneline -5
```

**ขั้นตอนที่ 8 (ถ้าทำงานกับทีม): push ผลลัพธ์ขึ้น remote**

```bash
git push origin main
```

### ตารางสรุปลำดับขั้นตอนมาตรฐาน

| ลำดับ | คำสั่ง | จุดประสงค์ |
|---|---|---|
| 1 | `git status` | ตรวจสอบว่างานปัจจุบันสะอาด ไม่มีไฟล์ค้าง |
| 2 | `git switch <branch-ปลายทาง>` | ย้ายไปยืนที่ branch ที่จะรับการเปลี่ยนแปลง |
| 3 | `git pull` (ถ้าเป็นทีม) | ดึงข้อมูลล่าสุดจาก remote มาก่อน |
| 4 | `git merge <branch-ที่จะรวมเข้ามา>` | สั่ง merge จริง |
| 5 | ตรวจผลลัพธ์ / แก้ conflict ถ้ามี | ให้ merge สำเร็จสมบูรณ์ |
| 6 | `git log --graph` | ตรวจสอบภาพรวมประวัติหลัง merge |
| 7 | `git push` (ถ้าเป็นทีม) | เผยแพร่ผลลัพธ์ไปยัง remote |

### ข้อควรระวังที่พบบ่อย

1. **สลับ branch ผิด** — merge สำเร็จแต่กลายเป็นรวมผิดทิศทาง (เช่น merge `main` เข้า `feature-x` ทั้งที่ตั้งใจจะทำกลับกัน) ควรเช็ค `git status` หรือ `git branch` ก่อน merge ทุกครั้ง
2. **ลืม pull ก่อน merge** — ทำให้ merge บน local สำเร็จ แต่พอ push ขึ้น remote กลับถูกปฏิเสธเพราะ remote มีการเปลี่ยนแปลงใหม่ที่เครื่องเรายังไม่มี
3. **มีไฟล์ยังไม่ commit ค้างอยู่ก่อน merge** — อาจทำให้ Git ปฏิเสธคำสั่ง merge ทันทีพร้อมข้อความ `error: Your local changes to the following files would be overwritten by merge`

---

## Step 75: Merge Conflict คืออะไร ทำไมถึงเกิดขึ้น

### นิยาม

> **Merge Conflict คือสถานการณ์ที่ Git ไม่สามารถตัดสินใจได้เองว่าจะรวมเนื้อหาจากสอง branch เข้าด้วยกันอย่างไร เพราะทั้งสองฝั่งต่างแก้ไข "จุดเดียวกัน" ของไฟล์เดียวกัน เป็นค่าที่ต่างกัน**

จากที่อธิบายไปใน Step 73 ว่า Git จะรวมอัตโนมัติได้เมื่อมีแค่ฝั่งเดียวแก้ไขจากจุดเดิม (Base) แต่เมื่อไหร่ที่ **ทั้งสองฝั่งแก้ไขจุดเดียวกันเป็นคนละค่า** Git จะไม่กล้าตัดสินใจแทนคุณ เพราะไม่รู้ว่าคุณต้องการค่าไหนจริง ๆ จึงหยุดกระบวนการ merge ไว้ตรงกลางแล้วให้ **มนุษย์เป็นคนตัดสินใจแทน**

### สาเหตุที่พบบ่อยที่สุดของ Conflict

| สาเหตุ | คำอธิบาย |
|---|---|
| **แก้ไขบรรทัดเดียวกันของไฟล์เดียวกัน** | พบบ่อยที่สุด — สอง branch แก้ค่าตัวแปร ข้อความ หรือ logic บรรทัดเดียวกันเป็นคนละค่า |
| **branch หนึ่งลบไฟล์ อีก branch แก้ไขไฟล์นั้น** | Git ไม่รู้ว่าควรลบไฟล์ทิ้งหรือเก็บการแก้ไขไว้ |
| **ทั้งสอง branch เพิ่มไฟล์ชื่อเดียวกันแต่เนื้อหาต่างกัน** | เกิด conflict แบบ "add/add" |
| **บรรทัดที่อยู่ใกล้กันมากถูกแก้พร้อมกัน** | แม้ไม่ใช่บรรทัดเดียวกันเป๊ะ แต่ถ้าอยู่ใกล้กันเกินไป (อยู่ใน context เดียวกัน) Git อาจตัดสินใจไม่ได้ว่าจะเรียงลำดับอย่างไร |

### ตัวอย่างสถานการณ์ที่ทำให้เกิด conflict

สมมติไฟล์ `config.js` ต้นฉบับ (Base commit) มีเนื้อหา:

```javascript
const PORT = 3000;
```

**ฝั่ง `main`** แก้ไขเป็น:

```javascript
const PORT = 8080;
```

**ฝั่ง `feature-x`** แก้ไขเป็น:

```javascript
const PORT = 5000;
```

เมื่อคุณอยู่ที่ `main` แล้วสั่ง `git merge feature-x` — Git จะงงทันทีว่า **"บรรทัดนี้ตอนจบควรเป็น `8080` หรือ `5000` กันแน่?"** เพราะทั้งสองฝั่งต่างแก้ไขจากค่าเดิม (`3000`) เป็นคนละค่า Git จึงไม่กล้าเดาแทนคุณ และหยุดกระบวนการไว้เพื่อรอให้คุณเป็นคนตัดสินใจ

### สิ่งที่เกิดขึ้นเมื่อ conflict เกิดขึ้น

```bash
git merge feature-x
```

```
Auto-merging config.js
CONFLICT (content): Merge conflict in config.js
Automatic merge failed; fix conflicts and then commit the result.
```

สังเกตคำสำคัญ:

- **`Auto-merging config.js`** — Git พยายามรวมไฟล์นี้อัตโนมัติแล้ว
- **`CONFLICT (content): Merge conflict in config.js`** — แต่ล้มเหลว เพราะมีการชนกันของ "เนื้อหา" (content) ในไฟล์นี้
- **`Automatic merge failed; fix conflicts and then commit the result.`** — Git บอกตรง ๆ เลยว่าต้องทำอะไรต่อ: แก้ conflict เองแล้วค่อย commit

### ตรวจสอบสถานะด้วย `git status` ระหว่างมี conflict

```bash
git status
```

```
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   config.js

no changes added to commit (use "git add" and/or "git commit -a")
```

ข้อความ `both modified: config.js` คือหลักฐานชัดเจนว่าทั้งสองฝั่งแก้ไขไฟล์นี้จนชนกัน และ Git บอกวิธีแก้ไว้ในข้อความเลยทั้งสองทาง คือ **แก้ไขแล้ว add** หรือ **ยกเลิกด้วย `git merge --abort`** (จะพูดถึงใน Step 78)

### สิ่งสำคัญที่ต้องเข้าใจ: Conflict ไม่ใช่ "ข้อผิดพลาด" หรือ "บั๊ก"

หลายคนที่เพิ่งเจอ conflict ครั้งแรกจะตกใจและคิดว่าตัวเองทำอะไรผิด แต่ความจริงแล้ว:

> **Merge Conflict คือพฤติกรรมปกติและคาดหวังได้ของ Git เมื่อทำงานเป็นทีม มันไม่ใช่สัญญาณว่าคุณทำอะไรผิด แต่เป็นสัญญาณว่า "มีสองความตั้งใจที่ต่างกันเกิดขึ้นในจุดเดียวกัน และต้องมีคนตัดสินใจ"**

ยิ่งทีมใหญ่ขึ้น ยิ่งทำงานในไฟล์เดียวกันบ่อยขึ้น โอกาสเจอ conflict ก็ยิ่งสูงขึ้นเป็นธรรมชาติ — สิ่งสำคัญคือต้อง**รู้วิธีแก้ไขอย่างมั่นใจ** ซึ่งเราจะเรียนใน Step ถัดไป

---

## Step 76: อ่านสัญลักษณ์ conflict marker `<<<<<<<` `=======` `>>>>>>>` ในไฟล์อย่างละเอียด

เมื่อเกิด conflict Git จะ**ไม่ลบเนื้อหาของฝั่งไหนทิ้งเลย** แต่จะเขียนเนื้อหาของทั้งสองฝั่งลงในไฟล์เดียวกัน โดยคั่นด้วยสัญลักษณ์พิเศษที่เรียกว่า **Conflict Marker** เพื่อให้คุณเห็นทั้งสองเวอร์ชันและตัดสินใจเอง

### เปิดไฟล์ `config.js` ที่มี conflict ดู

```javascript
function startServer() {
<<<<<<< HEAD
  const PORT = 8080;
=======
  const PORT = 5000;
>>>>>>> feature-x
  app.listen(PORT);
}
```

มาแยกส่วนแต่ละบรรทัดของ marker ให้เข้าใจทีละส่วน:

```
<<<<<<< HEAD                 ← จุดเริ่มต้นของฝั่ง "เรา" (branch ที่ยืนอยู่ตอนนี้)
  const PORT = 8080;         ← เนื้อหาจาก branch ปัจจุบัน (ในตัวอย่างนี้คือ main)
=======                      ← เส้นแบ่งกลาง คั่นระหว่างสองฝั่ง
  const PORT = 5000;         ← เนื้อหาจาก branch ที่กำลัง merge เข้ามา
>>>>>>> feature-x             ← จุดสิ้นสุดของฝั่ง "เขา" พร้อมชื่อ branch กำกับ
```

### ตารางสรุปความหมายของแต่ละส่วน

| สัญลักษณ์ | ความหมาย |
|---|---|
| `<<<<<<< HEAD` | เริ่มต้นเนื้อหาฝั่ง **"Ours"** คือ branch ที่คุณยืนอยู่ตอนสั่ง merge (`HEAD` ชี้มาที่นี่) |
| (เนื้อหาก่อนถึง `=======`) | โค้ดเวอร์ชันของฝั่งเรา |
| `=======` | เส้นแบ่งกึ่งกลาง แยกสองฝั่งออกจากกันอย่างชัดเจน |
| (เนื้อหาหลัง `=======` จนถึง `>>>>>>>`) | โค้ดเวอร์ชันของฝั่งอีกฝ่าย |
| `>>>>>>> feature-x` | สิ้นสุดเนื้อหาฝั่ง **"Theirs"** พร้อมบอกชื่อ branch ที่มาจากตรงนี้ให้ชัดเจน |

### จำนวนตัวอักษรของ marker มีความหมาย

สังเกตว่า `<<<<<<<` และ `>>>>>>>` มีตัวอักษรซ้ำ **7 ตัว** เสมอ (ไม่ใช่ 6 หรือ 8) และ `=======` ก็มี `=` 7 ตัวเช่นกัน — นี่คือรูปแบบมาตรฐานที่ Git กำหนดไว้ตายตัว เพื่อให้เครื่องมือต่าง ๆ (editor, IDE, merge tool) รู้จักและแสดงผลได้ถูกต้องสม่ำเสมอ

### กรณีที่ไฟล์มีหลาย conflict block ในไฟล์เดียวกัน

ไฟล์หนึ่งไฟล์สามารถมี conflict ได้มากกว่า 1 จุดพร้อมกัน:

```javascript
function config() {
<<<<<<< HEAD
  const PORT = 8080;
=======
  const PORT = 5000;
>>>>>>> feature-x

  const TIMEOUT = 30;
}

function helper() {
<<<<<<< HEAD
  return "production-mode";
=======
  return "development-mode";
>>>>>>> feature-x
}
```

**ต้องแก้ให้ครบทุก block** ในไฟล์ ห้ามแก้แค่ block แรกแล้วลืม block ที่สอง — เดี๋ยว Step 77 จะสอนวิธีตรวจสอบให้ครบถ้วน

### ค่า Base เพิ่มเติม (diff3 style) — ตัวเลือกขั้นสูง

ปกติ Git จะแสดง marker แบบ 2 ฝั่ง (`HEAD` กับชื่อ branch อีกฝั่ง) แต่ Git มีโหมดพิเศษที่เรียกว่า **diff3 conflict style** ที่จะโชว์เนื้อหาต้นฉบับ (Base) เพิ่มเข้ามาด้วย ช่วยให้เห็นว่า "ค่าดั้งเดิมก่อนแก้คืออะไร" ทำให้ตัดสินใจง่ายขึ้นในบางกรณี:

```bash
git config --global merge.conflictstyle diff3
```

หลังตั้งค่านี้ conflict marker จะมีหน้าตาแบบนี้แทน:

```javascript
<<<<<<< HEAD
  const PORT = 8080;
||||||| a1b2c3d (common ancestor)
  const PORT = 3000;
=======
  const PORT = 5000;
>>>>>>> feature-x
```

ส่วน `||||||| a1b2c3d` คือเนื้อหาต้นฉบับก่อนที่ทั้งสองฝั่งจะแก้ไข ช่วยให้เห็นชัดว่าแต่ละฝั่งเปลี่ยนอะไรไปจากของเดิมบ้าง (ค่าเริ่มต้นของ Git ตั้งแต่เวอร์ชันใหม่ ๆ คือ `zdiff3` ซึ่งฉลาดกว่าเดิมอีกขั้นในการจัดเรียงส่วนที่เหมือนกัน)

---

## Step 77: ขั้นตอนแก้ conflict ทีละขั้น (เปิดไฟล์ แก้ไข ลบ marker ออก แล้ว add + commit)

มาถึงขั้นตอนที่สำคัญที่สุดของ Part นี้ — การแก้ conflict ให้สำเร็จอย่างเป็นระบบ

### ภาพรวมขั้นตอนทั้งหมด

```
1. git status               → ดูว่าไฟล์ไหน conflict บ้าง
2. เปิดไฟล์ที่ conflict       → ด้วย editor
3. อ่านและตัดสินใจ           → เลือกเก็บฝั่งไหน หรือรวมทั้งสองฝั่ง
4. ลบ marker ออกให้หมด       → <<<<<<< ======= >>>>>>>
5. บันทึกไฟล์
6. git add <ไฟล์ที่แก้แล้ว>   → บอก Git ว่า "จุดนี้แก้เสร็จแล้ว"
7. git status                → ตรวจสอบว่าแก้ครบทุกไฟล์แล้ว
8. git commit                → ปิดจบการ merge
```

### ขั้นตอนที่ 1: ดูว่าไฟล์ไหน conflict บ้าง

```bash
git status
```

```
Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   config.js
        both modified:   utils.js
```

ในตัวอย่างนี้มี **2 ไฟล์** ที่ conflict คือ `config.js` และ `utils.js` — ต้องแก้ให้ครบทั้งสองไฟล์

### ขั้นตอนที่ 2-5: เปิดไฟล์ทีละไฟล์แล้วตัดสินใจ

เปิด `config.js` ด้วย editor (เช่น VS Code):

```bash
code config.js
```

เนื้อหาก่อนแก้:

```javascript
function startServer() {
<<<<<<< HEAD
  const PORT = 8080;
=======
  const PORT = 5000;
>>>>>>> feature-x
  app.listen(PORT);
}
```

มี 3 ทางเลือกหลักในการแก้ conflict แต่ละจุด:

**ทางเลือกที่ 1: เก็บฝั่งเราไว้ (Ours)**

```javascript
function startServer() {
  const PORT = 8080;
  app.listen(PORT);
}
```

**ทางเลือกที่ 2: เก็บฝั่งเขาไว้ (Theirs)**

```javascript
function startServer() {
  const PORT = 5000;
  app.listen(PORT);
}
```

**ทางเลือกที่ 3: รวมทั้งสองแนวคิดเข้าด้วยกัน (มักเกิดกรณีนี้บ่อยที่สุดในงานจริง)**

```javascript
function startServer() {
  const PORT = process.env.PORT || 8080; // รองรับทั้งค่า default และ environment variable
  app.listen(PORT);
}
```

> **ข้อควรจำที่สำคัญที่สุด:** ไม่ว่าจะเลือกทางไหน **ต้องลบสัญลักษณ์ `<<<<<<<`, `=======`, `>>>>>>>` ออกให้หมดทุกตัว** ห้ามเหลือค้างไว้แม้แต่บรรทัดเดียว เพราะถ้าลืมลบ โค้ดจะพังทันที (JavaScript จะมองว่า `<<<<<<< HEAD` เป็น syntax error) และในบางภาษาอาจไม่ error แต่รันแล้วพฤติกรรมผิดเพี้ยนแบบที่ตรวจจับยากมาก

หลังแก้ไขและบันทึกไฟล์ทำแบบเดียวกันกับ `utils.js` ให้ครบทุกไฟล์

### ขั้นตอนที่ 6: `git add` ไฟล์ที่แก้เสร็จแล้ว

```bash
git add config.js
git add utils.js
```

หรือถ้าแก้ครบทุกไฟล์ที่ conflict แล้ว จะใช้แบบนี้ก็ได้ (แต่ควรระวังถ้ามีไฟล์อื่นที่ไม่เกี่ยวข้องปนอยู่):

```bash
git add .
```

> **สำคัญ:** คำสั่ง `git add` ในบริบทของการแก้ conflict มีความหมายพิเศษกว่าเดิม มันไม่ได้แค่ "เตรียม stage ไฟล์" แต่เป็นการ **บอก Git อย่างชัดเจนว่า "ฉันแก้ conflict ของไฟล์นี้เสร็จแล้ว พร้อมให้ merge ต่อ"**

### ขั้นตอนที่ 7: ตรวจสอบว่าแก้ครบทุกไฟล์แล้วจริง ๆ

```bash
git status
```

```
On branch main
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
        modified:   config.js
        modified:   utils.js
```

ข้อความ **`All conflicts fixed but you are still merging.`** คือสัญญาณที่ยืนยันว่า Git มองว่าคุณแก้ conflict ครบทุกจุดแล้ว พร้อมให้ commit ปิดจบได้

**ถ้ายังแก้ไม่ครบ** สถานะจะยังแสดง `Unmerged paths` อยู่เหมือนเดิม แปลว่ายังมีไฟล์ที่ต้องแก้เพิ่ม

### ขั้นตอนที่ 8: `git commit` เพื่อปิดจบการ merge

```bash
git commit
```

สิ่งพิเศษของการ commit ครั้งนี้คือ **Git จะเตรียมข้อความ commit ให้อัตโนมัติแล้ว** โดยไม่ต้องพิมพ์เอง (เพียงแค่ตรวจสอบและบันทึกในหน้าต่าง editor ที่เปิดขึ้นมา):

```
Merge branch 'feature-x' into main

# Conflicts:
#	config.js
#	utils.js
#
# It looks like you may be committing a merge.
# If this is not correct, please remove the file
#	.git/MERGE_HEAD
# and try again.
```

คุณสามารถแก้ไขข้อความนี้เพิ่มเติมได้ (เช่น อธิบายว่าทำไมถึงเลือกแก้แบบนั้น) หรือจะปล่อยเป็นค่า default ก็ได้ตามความเหมาะสมของทีม บันทึกและปิด editor เพื่อให้ commit สำเร็จ

หรือถ้าต้องการข้ามการเปิด editor ไปเลย ใช้:

```bash
git commit --no-edit
```

### ขั้นตอนที่ 9: ตรวจสอบผลลัพธ์สุดท้าย

```bash
git log --graph --oneline -5
```

```
*   9d3f2a1 (HEAD -> main) Merge branch 'feature-x' into main
|\
| * 7c1e0b3 (feature-x) ปรับ PORT ใน config
* | 4a5d8f2 อัปเดต README
|/
* 1e2f3a4 commit เริ่มต้น
```

Merge สำเร็จสมบูรณ์แล้ว! branch `main` ตอนนี้มีทั้งการเปลี่ยนแปลงของตัวเองและของ `feature-x` รวมกันอยู่

### สรุปเป็นตารางเช็กลิสต์

| ขั้นตอน | คำสั่ง | ผลลัพธ์ที่ควรเห็น |
|---|---|---|
| 1 | `git status` | รายชื่อไฟล์ที่ conflict (`both modified`) |
| 2 | เปิดไฟล์ + แก้ไข + ลบ marker | ไฟล์ไม่มี `<<<<<<<` `=======` `>>>>>>>` เหลืออยู่ |
| 3 | `git add <ไฟล์>` | ไฟล์ย้ายจาก "unmerged" เป็น "staged" |
| 4 | `git status` (เช็คซ้ำ) | ข้อความ `All conflicts fixed but you are still merging.` |
| 5 | `git commit` | Merge commit ใหม่ถูกสร้างขึ้นสำเร็จ |

---

## Step 78: การยกเลิก merge ระหว่างมี conflict ด้วย `git merge --abort`

บางครั้งเมื่อเจอ conflict คุณอาจรู้สึกว่า:

- conflict ซับซ้อนเกินไป อยากกลับไปตั้งหลักคิดใหม่ก่อน
- merge ผิด branch ตั้งแต่แรก
- อยากปรึกษาเพื่อนร่วมทีมก่อนตัดสินใจว่าจะแก้แบบไหน
- ต้องการดูโค้ดในสถานะปกติก่อนโดยไม่มี conflict marker ปนอยู่

ในสถานการณ์เหล่านี้ Git มีทางออกฉุกเฉินที่ปลอดภัยมากให้ใช้ คือ:

```bash
git merge --abort
```

### `git merge --abort` ทำอะไรบ้าง

> **`git merge --abort` จะยกเลิกกระบวนการ merge ที่กำลังดำเนินอยู่ทั้งหมด แล้วคืนสถานะของ working directory และ staging area กลับไปเป็นแบบก่อนที่คุณจะสั่ง `git merge` เป๊ะ ๆ เหมือนไม่เคยสั่ง merge มาก่อนเลย**

ไดอะแกรมเปรียบเทียบ:

```
ก่อน merge:        main อยู่ที่ commit D
                    (working directory สะอาด)

สั่ง git merge  →   เกิด conflict, ไฟล์มี marker ค้างอยู่
                    (สถานะ "กำลัง merge อยู่")

สั่ง --abort   →   กลับไปที่ commit D เหมือนเดิม
                    (working directory สะอาดเหมือนก่อน merge)
```

### ตัวอย่างการใช้งานจริง

```bash
git merge feature-x
```

```
CONFLICT (content): Merge conflict in config.js
Automatic merge failed; fix conflicts and then commit the result.
```

หากตัดสินใจไม่แก้ตอนนี้:

```bash
git merge --abort
```

```bash
git status
```

```
On branch main
nothing to commit, working tree clean
```

สังเกตว่า working directory กลับมาสะอาดทันที ไม่มีร่องรอยของ conflict marker หรือไฟล์ที่ค้างอยู่เลย เหมือนไม่เคยสั่ง merge มาก่อน

### ข้อจำกัดสำคัญที่ต้องรู้

> **`git merge --abort` ใช้ได้เฉพาะ "ระหว่างที่ยังมี conflict ค้างอยู่และยังไม่ได้ commit" เท่านั้น**

ถ้าคุณ `git add` และ `git commit` ปิดจบการ merge ไปแล้ว คำสั่งนี้จะใช้ไม่ได้อีกต่อไป — ถ้าลองรันดูจะได้ข้อความ:

```bash
git merge --abort
```

```
fatal: There is no merge to abort (MERGE_HEAD missing).
```

ในกรณีที่ merge สำเร็จไปแล้วแต่อยากยกเลิกทีหลัง ต้องใช้คำสั่งอื่นแทน เช่น `git reset --hard <commit-ก่อน-merge>` ซึ่งเป็นคำสั่งที่มีความเสี่ยงสูงกว่ามาก (จะอธิบายอย่างละเอียดพร้อมข้อควรระวังใน Part ที่ว่าด้วยเรื่อง `git reset` โดยเฉพาะ)

### `git merge --abort` vs `git reset --hard`: ต่างกันอย่างไร

| คำสั่ง | ใช้เมื่อไหร่ | ความปลอดภัย |
|---|---|---|
| `git merge --abort` | ระหว่างมี conflict ค้างอยู่ ยังไม่ commit | ปลอดภัยมาก ออกแบบมาเพื่อกรณีนี้โดยเฉพาะ |
| `git reset --hard <commit>` | หลัง merge commit สำเร็จไปแล้วแต่อยากย้อนกลับ | เสี่ยงกว่า ต้องรู้ commit hash ที่ถูกต้อง และจะลบการเปลี่ยนแปลงที่ยังไม่ commit ทิ้งไปด้วย |

### วิธีสังเกตว่ากำลังอยู่ระหว่าง merge ค้างอยู่หรือไม่

ถ้าไม่แน่ใจว่าตอนนี้กำลังอยู่ระหว่าง merge ที่ค้างอยู่หรือเปล่า มีสองวิธีตรวจสอบ:

**วิธีที่ 1: ดูจาก `git status`**

```
You have unmerged paths.
  (use "git merge --abort" to abort the merge)
```

**วิธีที่ 2: ตรวจสอบไฟล์ `.git/MERGE_HEAD`**

```bash
ls .git/MERGE_HEAD
```

ถ้าไฟล์นี้มีอยู่จริง แปลว่ากำลังอยู่ระหว่าง merge ที่ยังไม่เสร็จ (ไฟล์นี้จะถูกลบอัตโนมัติทันทีที่ merge สำเร็จหรือถูก abort)

### คำแนะนำเชิงปฏิบัติ

อย่ากลัวที่จะใช้ `git merge --abort` เมื่อรู้สึกไม่มั่นใจ — **มันปลอดภัย 100% และไม่ทำให้ข้อมูลใด ๆ ที่เคย commit ไปแล้วหายไป** เพราะสิ่งที่มันยกเลิกคือแค่ "กระบวนการ merge ที่ยังไม่เสร็จสิ้น" เท่านั้น ไม่ได้แตะต้อง commit history ที่มีอยู่แล้วเลยแม้แต่นิดเดียว

---

## Step 79: เครื่องมือช่วยแก้ conflict (git mergetool เบื้องต้น, VS Code merge editor)

การแก้ conflict ด้วยการเปิดไฟล์แล้วอ่าน marker เองแบบ manual (ที่เราทำใน Step 77) เป็นทักษะพื้นฐานที่**ทุกคนต้องทำเป็น** แต่เมื่อ conflict มีความซับซ้อนมากขึ้น หรือมีหลายไฟล์พร้อมกัน เครื่องมือช่วยเหลือจะทำให้งานง่ายขึ้นมาก

### `git mergetool` คืออะไร

> **`git mergetool` คือคำสั่งที่เปิดโปรแกรมช่วยแก้ conflict แบบ visual (มี GUI) แทนที่จะให้คุณอ่าน marker เป็นข้อความล้วน ๆ**

การใช้งานพื้นฐาน:

```bash
git mergetool
```

ถ้ายังไม่เคยตั้งค่าเครื่องมือไว้ Git จะแจ้งเตือนและแนะนำเครื่องมือที่ตรวจพบในเครื่อง เช่น:

```
This message is displayed because 'merge.tool' is not configured.
See 'git mergetool --tool-help' or 'git help config' for more details.
'git mergetool' will now attempt to use one of the following tools:
opendiff kdiff3 tkdiff xxdiff meld tortoisemerge gvimdiff diffuse diffmerge
ecmerge p4merge araxis bc codecompare vimdiff nvimdiff emerge
```

### ดูรายชื่อเครื่องมือที่รองรับทั้งหมด

```bash
git mergetool --tool-help
```

### ตั้งค่าเครื่องมือที่ต้องการใช้เป็นค่า default

ตัวอย่างการตั้งค่าให้ใช้ VS Code เป็น merge tool:

```bash
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
```

หลังจากนั้นเวลารัน `git mergetool` ระบบจะเปิด VS Code ขึ้นมาที่ไฟล์ conflict โดยอัตโนมัติ

### VS Code Merge Editor (แนะนำมากสำหรับผู้เริ่มต้น)

จริง ๆ แล้วไม่จำเป็นต้องรัน `git mergetool` เสมอไป เพราะ **VS Code ตรวจจับไฟล์ที่มี conflict marker ได้อัตโนมัติทันทีที่เปิดไฟล์นั้นขึ้นมา** และจะแสดงปุ่มช่วยเหลือเหนือแต่ละ conflict block โดยตรง

เมื่อเปิดไฟล์ที่มี conflict ใน VS Code จะเห็นปุ่มเหล่านี้ปรากฏเหนือ conflict block แต่ละจุด:

| ปุ่ม | ความหมาย |
|---|---|
| **Accept Current Change** | เก็บเฉพาะเนื้อหาฝั่ง `HEAD` (ฝั่งเรา) |
| **Accept Incoming Change** | เก็บเฉพาะเนื้อหาฝั่งที่ merge เข้ามา (ฝั่งเขา) |
| **Accept Both Changes** | เก็บเนื้อหาทั้งสองฝั่ง เรียงต่อกัน (ต้องตรวจสอบผลลัพธ์เองว่าเรียงถูกต้องหรือไม่) |
| **Compare Changes** | เปิดหน้าต่างเปรียบเทียบแบบ side-by-side เพื่อดูรายละเอียดก่อนตัดสินใจ |

เมื่อกดปุ่มใดปุ่มหนึ่งแล้ว VS Code จะ**ลบ conflict marker ออกให้อัตโนมัติ** และเก็บเฉพาะเนื้อหาที่คุณเลือกไว้ ทำให้ไม่ต้องกังวลเรื่องลืมลบ marker เหมือนตอนแก้ไขด้วยมือ

### ขั้นตอนแนะนำเมื่อใช้ VS Code Merge Editor

1. หลังจาก `git merge` แล้วเกิด conflict ให้เปิด VS Code ที่โฟลเดอร์โปรเจกต์
2. ไปที่แถบ **Source Control** (ไอคอนรูปกิ่งไม้ทางซ้ายมือ) จะเห็นไฟล์ที่ conflict ถูกจัดหมวดไว้ในหัวข้อ **"Merge Changes"**
3. คลิกเปิดไฟล์แต่ละไฟล์ แล้วเลือก Accept ตามที่ต้องการทีละ block
4. เมื่อแก้ครบทุก block ในไฟล์แล้ว บันทึกไฟล์ (`Ctrl+S` / `Cmd+S`)
5. กลับไปที่ Source Control panel แล้วกด **`+`** เพื่อ stage ไฟล์ (เทียบเท่ากับ `git add`)
6. พิมพ์ข้อความ commit แล้วกด **Commit** (เทียบเท่ากับ `git commit`)

### ตรวจสอบให้แน่ใจว่าแก้ครบทุก conflict block

VS Code มีความสามารถพิเศษ คือถ้าไฟล์ยังมี conflict block เหลืออยู่ มันจะยังคงแสดงไฟล์นั้นอยู่ในหมวด "Merge Changes" ต่อไป จนกว่าจะแก้ครบทุกจุดจริง ๆ — ใช้ฟีเจอร์นี้เป็นตัวช่วยตรวจทานขั้นสุดท้ายได้เป็นอย่างดี

### คำสั่งเสริมที่มีประโยชน์ระหว่างแก้ conflict

| คำสั่ง | ประโยชน์ |
|---|---|
| `git diff` | ดูเนื้อหาที่ยังเป็น conflict อยู่ พร้อมสัญลักษณ์บอกฝั่ง |
| `git diff --ours` | เปรียบเทียบเฉพาะฝั่งของเรา (HEAD) กับ base |
| `git diff --theirs` | เปรียบเทียบเฉพาะฝั่งที่ merge เข้ามา กับ base |
| `git diff --base` | ดูว่า base (จุดร่วมเดิม) มีเนื้อหาอย่างไร |
| `git log --merge` | แสดง commit ทั้งหมดที่เกี่ยวข้องกับ conflict นี้ในทั้งสอง branch |
| `git checkout --ours <file>` | ยกเลิกการแก้ไข marker แล้วใช้ทั้งไฟล์ฝั่งเราแทน (ใช้เมื่อมั่นใจว่าอยากเก็บทั้งไฟล์ฝั่งเดียว) |
| `git checkout --theirs <file>` | เหมือนด้านบนแต่ใช้ฝั่งที่ merge เข้ามาแทนทั้งไฟล์ |

ตัวอย่างการใช้ `git checkout --ours` เมื่อมั่นใจว่าไม่ต้องการเนื้อหาจากอีกฝั่งเลยทั้งไฟล์:

```bash
git checkout --ours package-lock.json
git add package-lock.json
```

กรณีนี้พบบ่อยกับไฟล์ที่ระบบ generate อัตโนมัติ เช่น `package-lock.json` หรือ `yarn.lock` ที่มักจะ conflict บ่อยแต่ไม่จำเป็นต้องแก้ไขด้วยมือ (สามารถลบไฟล์แล้วสั่ง `npm install` ใหม่เพื่อ generate ไฟล์ที่ถูกต้องแทนก็ได้เช่นกัน)

---

## Step 80: แบบฝึกหัด - จำลองสถานการณ์ conflict จริงจากสอง branch ที่แก้ไฟล์เดียวกัน แล้วแก้ไขให้สำเร็จ

ถึงเวลาลงมือปฏิบัติจริงเพื่อฝึกทักษะที่เรียนมาทั้งหมดใน Part นี้ เตรียมโฟลเดอร์ฝึกฝนตามที่แนะนำไว้ใน Part 01

### เตรียมโฟลเดอร์ฝึกหัด

```bash
mkdir ~/git-course/part-08-merge-conflict
cd ~/git-course/part-08-merge-conflict
git init
```

### ขั้นตอนที่ 1: สร้างไฟล์เริ่มต้นและ commit แรก

```bash
echo "การตั้งค่าเซิร์ฟเวอร์
PORT = 3000
ENVIRONMENT = development" > server-config.txt

git add server-config.txt
git commit -m "เพิ่มไฟล์ตั้งค่าเซิร์ฟเวอร์เริ่มต้น"
```

### ขั้นตอนที่ 2: สร้าง branch แรก `feature-production` แล้วแก้ไข

```bash
git switch -c feature-production
```

แก้ไขไฟล์ `server-config.txt` ให้เป็น:

```
การตั้งค่าเซิร์ฟเวอร์
PORT = 8080
ENVIRONMENT = production
```

```bash
git add server-config.txt
git commit -m "ปรับค่าให้เหมาะกับ production"
```

### ขั้นตอนที่ 3: กลับไปที่ `main` แล้วสร้าง branch ที่สองมาแก้จุดเดียวกัน

```bash
git switch main
git switch -c feature-staging
```

แก้ไขไฟล์ `server-config.txt` (คนละแบบกับ branch ก่อนหน้า) ให้เป็น:

```
การตั้งค่าเซิร์ฟเวอร์
PORT = 5000
ENVIRONMENT = staging
```

```bash
git add server-config.txt
git commit -m "ปรับค่าให้เหมาะกับ staging"
```

### ขั้นตอนที่ 4: ตรวจสอบภาพรวม branch ทั้งหมดก่อน merge

```bash
git log --graph --oneline --all
```

ผลลัพธ์ที่ควรเห็น:

```
* 3f8a1b2 (HEAD -> feature-staging) ปรับค่าให้เหมาะกับ staging
| * 9c4d2e1 (feature-production) ปรับค่าให้เหมาะกับ production
|/
* 1a2b3c4 (main) เพิ่มไฟล์ตั้งค่าเซิร์ฟเวอร์เริ่มต้น
```

สังเกตว่าทั้งสอง branch แยกออกจาก commit เดียวกัน (`1a2b3c4`) และต่างแก้ไฟล์เดียวกันเป็นคนละค่า — นี่คือสูตรสำเร็จของการเกิด conflict ที่เราเรียนใน Step 75

### ขั้นตอนที่ 5: กลับไปที่ `main` แล้ว merge `feature-production` เข้าไปก่อน (จะสำเร็จโดยไม่มี conflict)

```bash
git switch main
git merge feature-production
```

ผลลัพธ์ (fast-forward เพราะ main ไม่มี commit ใหม่เลยตั้งแต่แยก branch):

```
Updating 1a2b3c4..9c4d2e1
Fast-forward
 server-config.txt | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)
```

### ขั้นตอนที่ 6: merge `feature-staging` เข้ามาด้วย — คราวนี้จะเกิด conflict จริง

```bash
git merge feature-staging
```

ผลลัพธ์ที่ควรเห็น:

```
Auto-merging server-config.txt
CONFLICT (content): Merge conflict in server-config.txt
Automatic merge failed; fix conflicts and then commit the result.
```

**เกิด conflict ขึ้นจริงแล้ว!** เพราะตอนนี้ `main` มีการเปลี่ยนแปลงจาก `feature-production` ติดตัวอยู่แล้ว (`PORT = 8080`) ในขณะที่ `feature-staging` ก็มีการเปลี่ยนแปลงของตัวเอง (`PORT = 5000`) ทำให้ history diverged กันจริง ๆ

### ขั้นตอนที่ 7: ตรวจสอบสถานะ

```bash
git status
```

```
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   server-config.txt
```

### ขั้นตอนที่ 8: เปิดไฟล์ดู conflict marker

```bash
cat server-config.txt
```

```
การตั้งค่าเซิร์ฟเวอร์
<<<<<<< HEAD
PORT = 8080
ENVIRONMENT = production
=======
PORT = 5000
ENVIRONMENT = staging
>>>>>>> feature-staging
```

### ขั้นตอนที่ 9: ตัดสินใจแก้ไข

ในสถานการณ์นี้ เราต้องการให้ไฟล์รองรับได้ทั้งสองค่าโดยใช้ environment variable แทนการ hardcode ค่าตายตัว (วิธีแก้แบบ "รวมทั้งสองแนวคิด" ที่สอนไว้ใน Step 77) แก้ไขไฟล์ `server-config.txt` ให้เป็น:

```
การตั้งค่าเซิร์ฟเวอร์
PORT = ค่าจาก environment variable (default: 8080)
ENVIRONMENT = กำหนดผ่าน environment variable ตอน deploy (production/staging)
```

สังเกตว่า **conflict marker ทุกตัวถูกลบออกหมดแล้ว** ไม่มี `<<<<<<<`, `=======`, `>>>>>>>` เหลืออยู่เลยแม้แต่บรรทัดเดียว

### ขั้นตอนที่ 10: add และ commit เพื่อปิดจบ merge

```bash
git add server-config.txt
git status
```

```
On branch main
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
        modified:   server-config.txt
```

```bash
git commit -m "Merge branch 'feature-staging': รวมการตั้งค่า production และ staging เข้าด้วยกันผ่าน environment variable"
```

### ขั้นตอนที่ 11: ตรวจสอบผลลัพธ์สุดท้าย

```bash
git log --graph --oneline --all
```

```
*   7e9f4a2 (HEAD -> main) Merge branch 'feature-staging': รวมการตั้งค่า production และ staging เข้าด้วยกันผ่าน environment variable
|\
| * 3f8a1b2 (feature-staging) ปรับค่าให้เหมาะกับ staging
* | 9c4d2e1 (feature-production) ปรับค่าให้เหมาะกับ production
|/
* 1a2b3c4 เพิ่มไฟล์ตั้งค่าเซิร์ฟเวอร์เริ่มต้น
```

```bash
cat server-config.txt
```

```
การตั้งค่าเซิร์ฟเวอร์
PORT = ค่าจาก environment variable (default: 8080)
ENVIRONMENT = กำหนดผ่าน environment variable ตอน deploy (production/staging)
```

**สำเร็จ!** คุณเพิ่งจำลองและแก้ไข merge conflict จริงครั้งแรกด้วยตัวเองสำเร็จเรียบร้อยแล้ว

### แบบฝึกหัดเสริม (ทำเพิ่มเพื่อความชำนาญ)

ลองทำซ้ำสถานการณ์เดิม แต่คราวนี้ให้ลองใช้วิธีที่ต่างออกไป:

1. **ลองใช้ `git merge --abort` แทรก** — ก่อนจะแก้ conflict จริง ให้ลองสั่ง `git merge --abort` ดูก่อนหนึ่งครั้ง แล้วตรวจสอบด้วย `git status` ว่ากลับสู่สถานะสะอาดจริงไหม จากนั้นค่อยสั่ง `git merge feature-staging` ใหม่อีกครั้งเพื่อฝึกแก้จริง
2. **ลองใช้ VS Code Merge Editor** — เปิดโฟลเดอร์นี้ด้วย VS Code แล้วลองใช้ปุ่ม Accept Current/Incoming/Both แทนการแก้ด้วยมือ เปรียบเทียบความสะดวกกับวิธี manual
3. **ลองสร้างสถานการณ์ conflict จากไฟล์โค้ดจริง** — สร้างไฟล์ `.js` หรือ `.py` ง่าย ๆ ที่มีฟังก์ชันคำนวณ แล้วให้สอง branch แก้ไข logic ในฟังก์ชันเดียวกันคนละแบบ ฝึกอ่านและแก้ conflict ในบริบทของโค้ดจริงมากขึ้น
4. **ลองสร้าง conflict แบบไฟล์ถูกลบฝั่งหนึ่ง** — branch หนึ่งลบไฟล์ อีก branch แก้ไขไฟล์เดิม แล้วลอง merge ดูว่า Git แจ้งเตือนต่างจากกรณี content conflict อย่างไร (จะเห็นข้อความประมาณ `CONFLICT (modify/delete)`)

### Checklist ก่อนไป Part 09

ก่อนไปต่อ Part 09 ให้ตรวจสอบว่าคุณ:

- [ ] อธิบายได้ว่า `git merge` รวมการเปลี่ยนแปลง "เข้าหา" branch ที่ยืนอยู่เสมอ
- [ ] แยกความแตกต่างระหว่าง Fast-forward merge กับ Three-way merge ได้อย่างชัดเจน
- [ ] รู้ว่า fast-forward merge ไม่มีทางเกิด conflict แต่ three-way merge มีโอกาสเกิด
- [ ] ทำตามขั้นตอนการ merge จริงได้ครบ (switch ไป branch ปลายทางก่อนเสมอ)
- [ ] เข้าใจว่า merge conflict เกิดขึ้นเมื่อทั้งสอง branch แก้ไขจุดเดียวกันเป็นคนละค่า และไม่ใช่ข้อผิดพลาด
- [ ] อ่านและเข้าใจความหมายของ conflict marker `<<<<<<<`, `=======`, `>>>>>>>` ได้อย่างถูกต้อง
- [ ] แก้ conflict ได้ครบขั้นตอน: แก้ไฟล์ → ลบ marker → `git add` → `git commit`
- [ ] รู้จักและใช้ `git merge --abort` เพื่อยกเลิก merge ที่ยังไม่ commit ได้อย่างมั่นใจ
- [ ] รู้จักเครื่องมือช่วยแก้ conflict อย่าง `git mergetool` และ VS Code Merge Editor
- [ ] ลงมือจำลองสถานการณ์ conflict จริงและแก้ไขให้สำเร็จได้ด้วยตัวเองอย่างน้อย 1 ครั้ง

---

## สรุป Part 08

ใน Part นี้เราได้เรียนรู้ว่า:

1. `git merge` คือการรวมประวัติจาก branch หนึ่งเข้ากับ branch ที่เรายืนอยู่ ไม่ใช่การรวมแบบสองทิศทาง
2. **Fast-forward merge** เกิดเมื่อ branch ปลายทางไม่มี commit ใหม่เลยตั้งแต่แยก branch — Git แค่เลื่อน pointer ไม่สร้าง commit ใหม่ และไม่มีทางเกิด conflict
3. **Three-way merge** เกิดเมื่อทั้งสอง branch มี commit ใหม่แยกกัน (diverged) — Git ใช้ common ancestor, ours, theirs มาคำนวณและสร้าง merge commit ที่มี 2 parent
4. ขั้นตอนการ merge ที่ถูกต้องคือ **สลับไป branch ปลายทางก่อนเสมอ** แล้วจึงสั่ง `git merge <branch-อื่น>`
5. Merge conflict เกิดขึ้นเมื่อทั้งสอง branch แก้ไขจุดเดียวกันของไฟล์เดียวกันเป็นคนละค่า — เป็นพฤติกรรมปกติของการทำงานเป็นทีม ไม่ใช่ข้อผิดพลาด
6. Conflict marker `<<<<<<<` `=======` `>>>>>>>` แบ่งเนื้อหาออกเป็นฝั่ง "เรา" (HEAD) และฝั่ง "เขา" (branch ที่ merge เข้ามา) อย่างชัดเจน
7. การแก้ conflict ต้องทำให้ครบ: แก้เนื้อหาให้ถูกต้อง → ลบ marker ออกให้หมด → `git add` → `git commit`
8. `git merge --abort` คือทางออกฉุกเฉินที่ปลอดภัย ใช้ได้เฉพาะระหว่างที่ยังไม่ commit ปิดจบ merge เท่านั้น
9. เครื่องมืออย่าง `git mergetool` และ VS Code Merge Editor ช่วยให้การแก้ conflict สะดวกและปลอดภัยยิ่งขึ้น โดยเฉพาะการป้องกันการลืมลบ marker
10. เราได้ลงมือจำลองสถานการณ์ conflict จริงจากสอง branch ที่แก้ไฟล์เดียวกัน และแก้ไขให้สำเร็จด้วยตัวเองเรียบร้อยแล้ว

**ต่อไป:** [Part 09: Remote Repository เบื้องต้น: clone, remote add, fetch](./part-009-remote-repository-clone-fetch.md)
