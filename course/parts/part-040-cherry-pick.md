# Part 40: Cherry-pick และการย้าย Commit ข้าม Branch

> **Step ในหลักสูตรนี้:** Step 391–400
> **เฟส:** 4 — ทำงานเป็นทีมด้วย Workflow มาตรฐาน (Git Flow, GitHub Flow, Rebase, การควบคุมคุณภาพโค้ดระดับทีม)
> **เป้าหมายของ Part นี้:** เข้าใจว่า `git cherry-pick` คืออะไร แตกต่างจาก merge และ rebase อย่างไร สามารถคัดลอก commit เดียวหรือหลาย commit ข้าม branch ได้อย่างถูกต้อง แก้ conflict ระหว่าง cherry-pick ได้ ใช้ `--no-commit` เป็นในสถานการณ์ที่เหมาะสม และนำ cherry-pick ไปประยุกต์ใช้กับสถานการณ์จริงในหน้างาน เช่นการกระจาย hotfix ไปยังหลาย release branch พร้อมกัน

---

## สารบัญของ Part นี้

- Step 391: `git cherry-pick` คืออะไร ใช้ทำอะไร
- Step 392: ขั้นตอน cherry-pick commit เดียวข้าม branch แบบเต็ม
- Step 393: Cherry-pick หลาย commit พร้อมกัน (ระบุหลาย hash หรือ range `A..B`)
- Step 394: การแก้ conflict ระหว่าง cherry-pick
- Step 395: `git cherry-pick -n`/`--no-commit` ใช้ตอนไหน
- Step 396: สถานการณ์จริงที่ต้องใช้ cherry-pick (hotfix ที่ต้องเอาไปใส่หลาย release branch)
- Step 397: ผลกระทบของ cherry-pick ต่อประวัติ (commit hash ใหม่)
- Step 398: cherry-pick vs merge vs rebase เมื่อไหร่ควรใช้อะไร
- Step 399: `git cherry-pick --abort`, `--continue`, `--quit`
- Step 400: แบบฝึกหัด — จำลอง hotfix ที่ต้อง cherry-pick ไปหลาย branch release

---

## Step 391: `git cherry-pick` คืออะไร ใช้ทำอะไร

จนถึงตอนนี้เราเรียนวิธีเอาการเปลี่ยนแปลง**ทั้งหมด**จาก branch หนึ่งไปรวมกับอีก branch มาแล้วสองแบบหลัก ๆ คือ **merge** (Part 08) และ **rebase** (Part 37–39) ทั้งสองแบบทำงานกับ**ชุดของ commit ทั้งชุด**บน branch

แต่ในโลกความเป็นจริง บ่อยครั้งที่คุณต้องการแค่ **"หยิบ commit เดียวหรือไม่กี่ commit"** จาก branch หนึ่ง ไปแปะใส่อีก branch หนึ่ง โดย**ไม่เอา commit อื่น ๆ ที่เหลือติดไปด้วย** — นี่คือหน้าที่ของ `git cherry-pick`

> **Cherry-pick** คือคำสั่งที่ให้ Git **คัดลอกการเปลี่ยนแปลง (diff) ของ commit หนึ่งตัว** จากที่ใดก็ได้ในประวัติของ repository แล้ว **สร้าง commit ใหม่** ที่มีการเปลี่ยนแปลงเดียวกันนั้น บน branch ปัจจุบันที่คุณยืนอยู่

ชื่อ "cherry-pick" มาจากภาพเปรียบเทียบการ "เลือกเก็บเชอร์รี่ที่สุกที่สุดเป็นลูก ๆ" จากต้นไม้ทั้งต้น แทนที่จะเก็บทั้งพวง — คุณเลือกเฉพาะ commit ที่คุณต้องการจริง ๆ ไม่ต้องเอา commit อื่นที่ไม่เกี่ยวข้องติดมาด้วย

### กลไกการทำงานเบื้องหลัง

เวลา cherry-pick commit หนึ่งตัว Git จะทำสิ่งเหล่านี้:

1. คำนวณ **diff** ระหว่าง commit ที่ระบุ กับ **parent ของมัน** (คือดูว่า commit นั้นเปลี่ยนแปลงอะไรไปบ้างเมื่อเทียบกับก่อนหน้า)
2. นำ diff นั้นไป **apply (แปะ)** ลงบน working directory ของ branch ปัจจุบัน (ที่ HEAD ชี้อยู่)
3. ถ้า apply สำเร็จไม่มี conflict จะสร้าง **commit ใหม่** ขึ้นมาโดยอัตโนมัติ พร้อมข้อความ commit เดิม (และมี trailer `(cherry picked from commit <hash>)` เพิ่มเข้ามาถ้าใช้ `-x`)
4. commit ใหม่นี้จะมี **hash ใหม่ทั้งหมด** (รายละเอียดใน Step 397)

### ภาพประกอบแนวคิด

```
main:     A---B---C---D
                   \
feature:            E---F---G

คุณต้องการแค่ commit F (ไม่เอา E, G) ไปใส่ใน main

git checkout main
git cherry-pick F

ผลลัพธ์:
main:     A---B---C---D---F'
                   \
feature:            E---F---G
```

สังเกตว่า `F'` คือ commit ใหม่ที่มี**เนื้อหาเหมือน F** แต่เป็นคนละ object กับ F ต้นฉบับ — นี่คือหัวใจสำคัญที่ต้องจำให้ขึ้นใจตลอด Part นี้

### เมื่อไหร่ที่ควรใช้ cherry-pick

Cherry-pick เหมาะกับสถานการณ์เหล่านี้เป็นพิเศษ:

- ต้องการนำ **bug fix** ที่ทำใน branch หนึ่ง ไปใส่ใน branch อื่นที่ไม่ได้อยู่ในสาย merge เดียวกัน (เช่น release branch เก่า)
- commit บาง commit ถูก commit ผิด branch ไปโดยไม่ตั้งใจ ต้องการย้ายไปอีก branch ที่ถูกต้อง
- ต้องการทดสอบว่า commit หนึ่ง ๆ ทำงานได้ดีบน branch อื่นหรือไม่ โดยไม่ merge ทั้งหมด
- ทำ **hotfix** ที่ต้องกระจายไปหลาย branch ที่ดูแล production คนละเวอร์ชันพร้อมกัน (Step 396 จะลงรายละเอียด)

### ข้อควรระวังเบื้องต้น

Cherry-pick ไม่ใช่เครื่องมือสำหรับรวม branch ทั้งสาย ถ้าคุณต้องการรวมงานทั้งหมดของ feature branch เข้ากับ main ให้ใช้ `merge` หรือ `rebase` ตามปกติ — cherry-pick มีไว้สำหรับ**เลือกหยิบเฉพาะจุด**เท่านั้น การใช้ cherry-pick ผิดที่ผิดทางอาจทำให้ประวัติซ้ำซ้อนและสับสนได้ง่าย

---

## Step 392: ขั้นตอน cherry-pick commit เดียวข้าม branch แบบเต็ม

มาลงมือทำจริงทีละขั้นตอน เริ่มจากสร้างสถานการณ์จำลอง

### 392.1 เตรียมสถานการณ์ทดสอบ

```bash
mkdir ~/git-course/part-40-cherry-pick
cd ~/git-course/part-40-cherry-pick
git init
git config user.name "Somchai Devtest"
git config user.email "somchai@example.com"

echo "v1" > app.txt
git add app.txt
git commit -m "commit A: เริ่มต้นโปรเจกต์"

git switch -c feature/login
echo "login logic" >> app.txt
git add app.txt
git commit -m "commit B: เพิ่มระบบ login"

echo "fix null pointer bug" >> app.txt
git add app.txt
git commit -m "commit C: แก้บั๊ก null pointer ตอน login"

echo "add login analytics" >> app.txt
git add app.txt
git commit -m "commit D: เพิ่ม analytics ให้ login"
```

ตอนนี้เรามี branch `feature/login` ที่มี 3 commit ใหม่ (B, C, D) ต่อจาก `main`

```
main:            A
                  \
feature/login:     B---C---D
```

สมมติว่า commit **C** (แก้บั๊ก null pointer) เป็นสิ่งที่ทีมต้องการรีบเอาเข้า `main` ทันที โดยยังไม่พร้อมเอา B และ D เข้าไป (เพราะยังทดสอบไม่เสร็จ)

### 392.2 หา hash ของ commit ที่ต้องการ

```bash
git log --oneline feature/login
```

ผลลัพธ์ตัวอย่าง (hash จริงจะไม่เหมือนกันทุกครั้ง):

```
d4e5f6a (HEAD -> feature/login) commit D: เพิ่ม analytics ให้ login
c3d4e5f commit C: แก้บั๊ก null pointer ตอน login
b2c3d4e commit B: เพิ่มระบบ login
a1b2c3d commit A: เริ่มต้นโปรเจกต์
```

จดหรือ copy hash ของ commit C ไว้ (ในตัวอย่างนี้คือ `c3d4e5f`) — ไม่จำเป็นต้องพิมพ์เต็มความยาว แค่ 7 ตัวอักษรแรกที่ไม่ซ้ำกับ commit อื่นก็ใช้ได้แล้ว

### 392.3 สลับไปยัง branch ปลายทาง

```bash
git switch main
```

**สำคัญมาก:** ต้องแน่ใจว่าคุณอยู่บน branch ที่**ต้องการรับ**การเปลี่ยนแปลงเข้าไป (ปลายทาง) ไม่ใช่ branch ต้นทาง ลองเช็คด้วย `git status` หรือ `git branch --show-current` ก่อนเสมอ เพื่อป้องกันความผิดพลาด

### 392.4 รันคำสั่ง cherry-pick

```bash
git cherry-pick c3d4e5f
```

ถ้าสำเร็จ Git จะแสดงผลประมาณนี้:

```
[main 9f8e7d6] commit C: แก้บั๊ก null pointer ตอน login
 Date: Thu Sep 24 10:15:00 2026 +0700
 1 file changed, 1 insertion(+)
```

สังเกตว่า commit message เดิมถูกนำมาใช้ทั้งหมด (ผู้เขียนเดิม วันที่ commit เดิม) แต่ **commit hash เป็นค่าใหม่** (`9f8e7d6` แทนที่จะเป็น `c3d4e5f`)

### 392.5 ตรวจสอบผลลัพธ์

```bash
git log --oneline main
```

```
9f8e7d6 (HEAD -> main) commit C: แก้บั๊ก null pointer ตอน login
a1b2c3d commit A: เริ่มต้นโปรเจกต์
```

และตรวจสอบว่า `feature/login` ยังคงมี commit ครบทุกตัวไม่ถูกกระทบ:

```bash
git log --oneline feature/login
```

```
d4e5f6a (feature/login) commit D: เพิ่ม analytics ให้ login
c3d4e5f commit C: แก้บั๊ก null pointer ตอน login
b2c3d4e commit B: เพิ่มระบบ login
a1b2c3d commit A: เริ่มต้นโปรเจกต์
```

จะเห็นว่า commit `c3d4e5f` ตัวเดิมยัง**อยู่ครบ**บน `feature/login` — cherry-pick ไม่ได้ "ย้าย" commit ออกจากที่เดิม แต่เป็นการ**คัดลอก**เนื้อหาไปสร้างใหม่ที่ปลายทาง ทั้งสอง branch จึงมี commit ที่เนื้อหาเหมือนกันแต่คนละ hash อยู่คู่ขนานกันไป

### 392.6 ตัวเลือกที่มีประโยชน์ในคำสั่งเดียวกัน

| ตัวเลือก | ความหมาย |
|---|---|
| `git cherry-pick <hash>` | cherry-pick แบบพื้นฐาน สร้าง commit ใหม่ทันที |
| `git cherry-pick -x <hash>` | เพิ่มบรรทัด `(cherry picked from commit <hash>)` ต่อท้าย commit message อัตโนมัติ ช่วยให้ตามรอยได้ง่ายว่า commit นี้มาจากไหน (แนะนำให้ใช้เสมอในทีม) |
| `git cherry-pick --edit <hash>` | เปิดให้แก้ไข commit message ก่อน commit จริง |
| `git cherry-pick -s <hash>` | เพิ่ม `Signed-off-by:` trailer (ใช้ในโปรเจกต์ที่ต้องการ DCO เช่น Linux Kernel) |

ในทีมงานจริง แนะนำให้ใช้ `git cherry-pick -x` เป็นมาตรฐาน เพราะช่วยให้ทุกคนที่มาอ่าน log ภายหลังรู้ทันทีว่า commit นี้ถูก cherry-pick มาจากที่ใด ไม่ใช่ commit ที่เขียนขึ้นใหม่ตรง ๆ

```bash
git switch main
git cherry-pick -x c3d4e5f
git log -1
```

```
commit 9f8e7d6...
Author: Somchai Devtest <somchai@example.com>
Date:   Thu Sep 24 10:15:00 2026 +0700

    commit C: แก้บั๊ก null pointer ตอน login

    (cherry picked from commit c3d4e5f8a9b0...)
```

---

## Step 393: Cherry-pick หลาย commit พร้อมกัน (ระบุหลาย hash หรือ range `A..B`)

บางครั้งคุณต้องการ commit มากกว่าหนึ่งตัวจาก branch เดียวกัน Git รองรับการ cherry-pick หลายรูปแบบ

### 393.1 ระบุหลาย hash แยกด้วยช่องว่าง

```bash
git switch main
git cherry-pick b2c3d4e c3d4e5f
```

Git จะ cherry-pick **ตามลำดับที่คุณระบุ** ไม่ใช่ตามลำดับเวลาที่ commit เดิมเกิดขึ้น ดังนั้นถ้าระบุลำดับผิด (เช่น commit ทีหลังมาก่อน commit ก่อนหน้าที่มันพึ่งพา) อาจเกิด conflict ที่ไม่จำเป็นได้ — **ควรระบุตามลำดับเวลาเดิมเสมอ** (เก่าไปใหม่)

### 393.2 ใช้ range แบบ `A..B` (ไม่รวม A แต่รวม B)

ถ้าต้องการ cherry-pick ทุก commit ตั้งแต่ B ถึง D (ไม่รวม A) ใช้ syntax range:

```bash
git cherry-pick a1b2c3d..d4e5f6a
```

**ข้อควรระวังสำคัญมาก:** range แบบ `A..B` จะ**ไม่รวม A** แต่**รวม B** เข้าไปด้วย ตรงนี้เป็นจุดที่มือใหม่พลาดบ่อยที่สุด สมมติ log เป็น:

```
a1b2c3d (A) -- b2c3d4e (B) -- c3d4e5f (C) -- d4e5f6a (D)
```

คำสั่ง `git cherry-pick a1b2c3d..d4e5f6a` จะ cherry-pick **B, C, D** (ไม่รวม A) เรียงตามลำดับเวลาให้อัตโนมัติ

ถ้าต้องการรวม A ด้วย ให้ใช้ syntax สามจุด `A^..B`:

```bash
git cherry-pick a1b2c3d^..d4e5f6a
```

`a1b2c3d^` หมายถึง "parent ของ A" ทำให้ range กลายเป็นรวม A ไปด้วย

### 393.3 การใช้ `--no-commit` ร่วมกับหลาย commit (ตัวอย่างเบื้องต้น)

เวลา cherry-pick หลาย commit พร้อมกัน บางครั้งอยากรวมทุก diff เข้าเป็น **commit เดียว** แทนที่จะสร้างหลาย commit ทำได้ดังนี้ (รายละเอียดเต็มอยู่ Step 395):

```bash
git cherry-pick --no-commit a1b2c3d..d4e5f6a
git commit -m "รวม fix จาก feature/login เข้ามาเป็นก้อนเดียว"
```

### 393.4 ถ้า cherry-pick หลาย commit แล้วเจอ conflict กลางทาง

ถ้า cherry-pick หลาย commit พร้อมกันแล้วมี commit ใดตัวหนึ่ง conflict Git จะ**หยุดอยู่ตรงนั้น** ให้คุณแก้ conflict ก่อน แล้วค่อยสั่ง `git cherry-pick --continue` เพื่อทำ commit ที่เหลือต่อ (อธิบายละเอียดใน Step 394 และ 399)

### 393.5 ตารางสรุปรูปแบบการระบุ commit

| รูปแบบ | ความหมาย |
|---|---|
| `git cherry-pick H1` | cherry-pick commit เดียว |
| `git cherry-pick H1 H2 H3` | cherry-pick หลาย commit ตามลำดับที่ระบุ |
| `git cherry-pick A..B` | cherry-pick ทุก commit ตั้งแต่หลัง A จนถึง B (ไม่รวม A) |
| `git cherry-pick A^..B` | cherry-pick ทุก commit ตั้งแต่ A จนถึง B (รวม A) |
| `git cherry-pick --no-commit A..B` | นำ diff ทั้งหมดมารวมไว้ใน staging area โดยยังไม่ commit |

---

## Step 394: การแก้ conflict ระหว่าง cherry-pick

Cherry-pick ก็เป็นการ "apply diff" แบบหนึ่ง ดังนั้นจึง**เกิด conflict ได้เหมือน merge** ถ้าโค้ดบริเวณเดียวกันถูกแก้ไขไปคนละทางระหว่าง branch ต้นทางกับปลายทาง

### 394.1 จำลองสถานการณ์ conflict

```bash
cd ~/git-course/part-40-cherry-pick
git switch main
echo "main-only change on same line area" >> app.txt
git add app.txt
git commit -m "commit E: แก้ไข main โดยตรงบริเวณเดียวกับ commit D"

git cherry-pick d4e5f6a
```

ถ้าบริเวณที่แก้ไขทับซ้อนกัน Git จะแจ้งเตือน:

```
Auto-merging app.txt
CONFLICT (content): Merge conflict in app.txt
error: could not apply d4e5f6a... commit D: เพิ่ม analytics ให้ login
hint: After resolving the conflicts, mark them with
hint: "git add/rm <pathspec>", then run
hint: "git cherry-pick --continue".
hint: You can instead skip this commit with "git cherry-pick --skip".
hint: To abort and get back to the state before "git cherry-pick",
hint: run "git cherry-pick --abort".
```

### 394.2 ตรวจสอบสถานะ

```bash
git status
```

```
On branch main
You are currently cherry-picking commit d4e5f6a.
  (fix conflicts and run "git cherry-pick --continue")
  (use "git cherry-pick --skip" to skip this patch)
  (use "git cherry-pick --abort" to cancel the cherry-pick operation)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   app.txt
```

### 394.3 เปิดไฟล์ดู marker ความขัดแย้ง

ไฟล์ `app.txt` จะมี marker แบบเดียวกับตอน merge conflict ทุกประการ:

```
v1
login logic
fix null pointer bug
<<<<<<< HEAD
main-only change on same line area
=======
add login analytics
>>>>>>> d4e5f6a (commit D: เพิ่ม analytics ให้ login)
```

- ส่วน **`HEAD`** คือเนื้อหาปัจจุบันบน `main` (ปลายทาง)
- ส่วน **`d4e5f6a`** คือเนื้อหาจาก commit ที่กำลัง cherry-pick มา (ต้นทาง)

### 394.4 แก้ไข conflict

ตัดสินใจว่าจะเก็บทั้งสองบรรทัด หรือเลือกอันใดอันหนึ่ง สมมติว่าต้องการเก็บทั้งคู่:

```
v1
login logic
fix null pointer bug
main-only change on same line area
add login analytics
```

ลบ marker `<<<<<<<`, `=======`, `>>>>>>>` ออกให้หมด แล้วบันทึกไฟล์

### 394.5 mark ว่าแก้เสร็จแล้วและ continue

```bash
git add app.txt
git cherry-pick --continue
```

ขั้นตอนนี้จะเปิด editor ให้แก้ commit message ได้ (ค่าเริ่มต้นคือ message เดิมของ commit D) บันทึกแล้วปิด editor เพื่อให้ Git สร้าง commit ใหม่เสร็จสมบูรณ์

```bash
git log --oneline main
```

```
a9b8c7d (HEAD -> main) commit D: เพิ่ม analytics ให้ login
f1e2d3c commit E: แก้ไข main โดยตรงบริเวณเดียวกับ commit D
9f8e7d6 commit C: แก้บั๊ก null pointer ตอน login
a1b2c3d commit A: เริ่มต้นโปรเจกต์
```

### 394.6 ข้อแตกต่างสำคัญจาก merge conflict

| ประเด็น | Merge conflict | Cherry-pick conflict |
|---|---|---|
| จำนวนรอบที่ต้องแก้ | รอบเดียว (รวมทุก commit เป็น diff เดียว) | อาจต้องแก้**ทีละ commit** ถ้า cherry-pick หลายตัวพร้อมกัน |
| คำสั่งดำเนินการต่อ | `git merge --continue` (หรือ commit ตรง ๆ) | `git cherry-pick --continue` |
| คำสั่งข้าม | ไม่มีแนวคิด "ข้าม commit เดียว" | `git cherry-pick --skip` ข้าม commit นี้ไปเลย |
| commit message เริ่มต้น | ข้อความ merge อัตโนมัติ | ข้อความ commit เดิมของ commit ที่ถูก pick |

### 394.7 `git cherry-pick --skip`

ถ้าแก้ conflict แล้วพบว่า commit นี้**ไม่จำเป็นต้องเอาเข้ามาจริง ๆ** (เช่นเนื้อหาซ้ำกับที่มีอยู่แล้ว) สามารถข้ามไปได้โดยไม่ commit:

```bash
git cherry-pick --skip
```

คำสั่งนี้จะละทิ้ง commit ที่กำลังติด conflict อยู่ แล้วไปทำ commit ถัดไปในคิว (ถ้ามีการ cherry-pick หลายตัวพร้อมกัน) หรือจบ operation ถ้าเป็นตัวสุดท้าย

---

## Step 395: `git cherry-pick -n`/`--no-commit` ใช้ตอนไหน

โดยปกติ cherry-pick แต่ละ commit จะสร้าง commit ใหม่ให้ทันทีโดยอัตโนมัติ แต่บางสถานการณ์เราต้องการแค่ "เอาการเปลี่ยนแปลงมาไว้ใน working directory / staging area" โดยยังไม่อยากให้ Git commit ให้ทันที นี่คือหน้าที่ของ flag `-n` หรือ `--no-commit`

### 395.1 ความหมายที่แท้จริง

```bash
git cherry-pick -n <hash>
# เทียบเท่ากับ
git cherry-pick --no-commit <hash>
```

คำสั่งนี้จะ apply diff ของ commit ที่ระบุ เข้าไปใน **working directory และ staging area** เหมือนปกติทุกอย่าง **ยกเว้นขั้นตอนสุดท้ายคือการสร้าง commit** — คุณต้องมา `git commit` เองภายหลัง

### 395.2 สถานการณ์ที่ควรใช้

**กรณีที่ 1: ต้องการรวมหลาย commit เป็นก้อนเดียว**

```bash
git switch main
git cherry-pick -n b2c3d4e c3d4e5f d4e5f6a
git status   # จะเห็นทุกไฟล์ที่เปลี่ยนอยู่ใน staged area
git commit -m "รวม feature login ทั้งหมดเป็น commit เดียวบน main"
```

วิธีนี้มีประโยชน์มากเวลาต้องการทำ **squash แบบเลือกเฉพาะบาง commit** โดยไม่ต้องเสียเวลาทำ interactive rebase เต็มรูปแบบ

**กรณีที่ 2: ต้องการแก้ไขเนื้อหาเพิ่มเติมก่อน commit จริง**

```bash
git cherry-pick -n c3d4e5f
# แก้ไขไฟล์เพิ่มเติมตามต้องการ เช่น เพิ่ม comment อธิบาย หรือปรับโค้ดเล็กน้อย
git add .
git commit -m "commit C (ปรับปรุงเพิ่มเติม): แก้บั๊ก null pointer พร้อม comment อธิบาย"
```

**กรณีที่ 3: ต้องการตรวจสอบว่า diff จะ apply ได้สะอาดหรือไม่ ก่อนตัดสินใจ commit จริง**

```bash
git cherry-pick -n d4e5f6a
git diff --staged   # ตรวจดู diff ที่จะถูก commit ก่อนตัดสินใจ
# ถ้าไม่พอใจ สามารถยกเลิกได้ด้วย
git reset --hard HEAD
```

**กรณีที่ 4: cherry-pick แบบ "เอาแค่บางไฟล์" จาก commit**

```bash
git cherry-pick -n c3d4e5f
git restore --staged some-file-that-you-dont-want.txt
git checkout -- some-file-that-you-dont-want.txt
git commit -m "commit C (บางส่วน): เอาเฉพาะไฟล์ที่เกี่ยวข้องจริง"
```

### 395.3 ข้อควรระวัง

- ถ้าใช้ `-n` กับหลาย commit พร้อมกัน แล้ว commit หนึ่งใน middle เกิด conflict, Git จะหยุดตรงนั้นเหมือนปกติ (ไม่มีอะไรเปลี่ยนแปลงเรื่อง conflict handling) เพียงแต่ commit ก่อนหน้าที่สำเร็จแล้วจะยังคง**ค้างอยู่ใน staging area** ไม่ใช่ commit แยกแต่ละตัว
- อย่าลืม `git commit` หลังใช้ `-n` มิฉะนั้นการเปลี่ยนแปลงจะค้างอยู่ใน staging area โดยไม่ถูกบันทึกลงประวัติจริง ๆ
- `git status` จะแจ้งเตือนตลอดเวลาที่ operation cherry-pick ยังไม่เสร็จสมบูรณ์ ให้สังเกตข้อความ "You are currently cherry-picking..." เป็นสัญญาณเตือน

---

## Step 396: สถานการณ์จริงที่ต้องใช้ cherry-pick (hotfix ที่ต้องเอาไปใส่หลาย release branch พร้อมกัน)

นี่คือ use case ที่ทีมพัฒนาซอฟต์แวร์ระดับโปรดักชันเจอบ่อยที่สุด และเป็นเหตุผลหลักที่ cherry-pick ถูกออกแบบมา

### 396.1 บริบทของปัญหา

สมมติบริษัทของคุณดูแลผลิตภัณฑ์ที่มีลูกค้าใช้งานหลายเวอร์ชันพร้อมกัน (เช่น ลูกค้า enterprise บางรายยังใช้เวอร์ชัน 1.0 อยู่ ในขณะที่ลูกค้าใหม่ใช้เวอร์ชัน 2.0) ทีมจึงต้อง maintain หลาย branch พร้อมกัน:

```
main            (สำหรับพัฒนาเวอร์ชันถัดไป เช่น 3.0)
release/2.0     (branch ที่ deploy ให้ลูกค้าเวอร์ชัน 2.0 อยู่)
release/1.0     (branch ที่ deploy ให้ลูกค้าเวอร์ชัน 1.0 อยู่ — ยัง support อยู่)
```

จู่ ๆ มีการค้นพบ **security bug ร้ายแรง** ที่กระทบทุกเวอร์ชัน (เช่น ช่องโหว่ SQL Injection) ทีม security แจ้งว่าต้อง**แก้และปล่อย patch ให้ทั้ง 3 branch ทันที** เพราะลูกค้าทุกเวอร์ชันมีความเสี่ยงเท่ากัน

### 396.2 ทำไมถึงใช้ merge ธรรมดาไม่ได้

ถ้าใช้ `git merge` ธรรมดา คุณจะต้อง merge `main` เข้า `release/1.0` ทั้งหมด ซึ่งจะดึงเอาฟีเจอร์ใหม่ ๆ ของเวอร์ชัน 3.0 ที่ยังไม่เสร็จ ยังไม่ผ่าน QA เข้าไปปนกับ production ของลูกค้าเวอร์ชัน 1.0 ด้วย — เป็นสิ่งที่**ยอมรับไม่ได้เด็ดขาด**ในทางปฏิบัติ

นี่คือจุดที่ cherry-pick เข้ามาแก้ปัญหาได้ตรงจุดที่สุด เพราะมันให้คุณหยิบ**เฉพาะ commit ที่แก้บั๊กความปลอดภัยตัวนั้น**ไปใส่ในแต่ละ branch โดยไม่แตะฟีเจอร์อื่นเลย

### 396.3 ขั้นตอนปฏิบัติจริงแบบเต็ม

**ขั้นตอนที่ 1: แก้บั๊กใน branch หลักก่อน (หรือสร้าง hotfix branch เฉพาะ)**

```bash
git switch main
git switch -c hotfix/sql-injection-fix
# แก้ไขโค้ดที่มีช่องโหว่
git add .
git commit -m "security: แก้ SQL Injection ในฟอร์มค้นหาสินค้า (CVE-2026-XXXX)"
```

**ขั้นตอนที่ 2: merge เข้า main ตามปกติก่อน (เพราะ main คือเวอร์ชันที่กำลังพัฒนาต่อ)**

```bash
git switch main
git merge --no-ff hotfix/sql-injection-fix
git push origin main
```

**ขั้นตอนที่ 3: หา hash ของ commit hotfix ที่เพิ่ง merge เข้าไป**

```bash
git log --oneline -5 main
```

```
h1i2j3k (HEAD -> main) Merge branch 'hotfix/sql-injection-fix'
g9h8i7j security: แก้ SQL Injection ในฟอร์มค้นหาสินค้า (CVE-2026-XXXX)
...
```

จด hash ของ commit ที่แก้บั๊กจริง (ไม่ใช่ merge commit) คือ `g9h8i7j`

**ขั้นตอนที่ 4: cherry-pick ไปยัง release/2.0**

```bash
git switch release/2.0
git pull origin release/2.0
git cherry-pick -x g9h8i7j
```

ถ้ามี conflict ให้แก้ตามขั้นตอนใน Step 394 จากนั้น:

```bash
git push origin release/2.0
```

**ขั้นตอนที่ 5: cherry-pick ไปยัง release/1.0 เช่นเดียวกัน**

```bash
git switch release/1.0
git pull origin release/1.0
git cherry-pick -x g9h8i7j
```

แก้ conflict ถ้ามี (ในความเป็นจริง release/1.0 มักมีโค้ดแตกต่างจาก main ไปมากแล้ว จึง conflict ได้ง่ายกว่า release/2.0 ที่ใหม่กว่า) จากนั้น:

```bash
git push origin release/1.0
```

**ขั้นตอนที่ 6: แปะ tag ให้แต่ละ branch ที่ปล่อย patch**

```bash
git switch release/2.0
git tag v2.0.4-hotfix
git push origin v2.0.4-hotfix

git switch release/1.0
git tag v1.0.9-hotfix
git push origin v1.0.9-hotfix
```

### 396.4 สรุปภาพรวมการกระจาย hotfix

```
main:         ...---g9h8i7j (แก้บั๊ก)---h1i2j3k (merge)
                     │
                     │ cherry-pick -x
                     ▼
release/2.0:  ...---X---g9h8i7j'  (v2.0.4-hotfix)
                     │
                     │ cherry-pick -x
                     ▼
release/1.0:  ...---Y---g9h8i7j'' (v1.0.9-hotfix)
```

สังเกตว่า `g9h8i7j`, `g9h8i7j'`, `g9h8i7j''` คือ**เนื้อหาเดียวกัน**แต่เป็นคนละ commit object กัน (hash ต่างกันทั้งหมด) — นี่คือธรรมชาติของ cherry-pick ที่ต้องเข้าใจให้ลึกซึ้ง ซึ่งจะอธิบายผลกระทบทั้งหมดใน Step ถัดไป

### 396.5 เทคนิคเสริม: cherry-pick อัตโนมัติหลาย branch ด้วย script

ในทีมที่ต้องกระจาย hotfix บ่อย ๆ ไปยัง branch จำนวนมาก สามารถเขียน script ช่วยลดความผิดพลาดได้ เช่น:

```bash
#!/bin/bash
# distribute-hotfix.sh
HOTFIX_COMMIT="$1"
BRANCHES=("release/1.0" "release/2.0" "release/2.1")

for branch in "${BRANCHES[@]}"; do
  echo "== กำลัง cherry-pick เข้า $branch =="
  git switch "$branch"
  git pull origin "$branch"
  if git cherry-pick -x "$HOTFIX_COMMIT"; then
    git push origin "$branch"
    echo "== สำเร็จ: $branch =="
  else
    echo "== เกิด conflict ที่ $branch กรุณาแก้ไขด้วยตัวเองแล้วรัน git cherry-pick --continue =="
    exit 1
  fi
done
```

script แบบนี้ช่วยลดขั้นตอนซ้ำ ๆ แต่ยังคงต้องมีคนตรวจสอบ conflict และ push เองในกรณีที่ automation หยุดกลางทาง

---

## Step 397: ผลกระทบของ cherry-pick ต่อประวัติ (ได้ commit hash ใหม่ แม้เนื้อหาเหมือนเดิม)

จาก Step ที่ผ่านมา คุณคงสังเกตเห็นแล้วว่า commit ที่ถูก cherry-pick จะมี **hash ใหม่เสมอ** แม้เนื้อหาข้างในจะเหมือนกันทุกตัวอักษรก็ตาม เรื่องนี้สำคัญมากพอที่จะแยกมาอธิบายละเอียดเป็น Step เฉพาะ

### 397.1 ทำไม hash ถึงเปลี่ยน

จำหลักการจาก Part 01 Step 6: Git object ทุกตัว (รวมถึง commit) ถูกระบุตัวตนด้วย hash ที่คำนวณจาก**เนื้อหาของมันเอง**ประกอบกับ **metadata** เช่น:

- Tree (snapshot ของไฟล์ทั้งหมด)
- Parent commit hash
- Author และ committer
- วันเวลา commit
- ข้อความ commit

เมื่อ cherry-pick commit ไปยัง branch อื่น สิ่งที่**เปลี่ยนแปลงแน่นอน**คือ **parent commit hash** เพราะ commit ใหม่นี้ต่อจาก commit ล่าสุดของ branch ปลายทาง ไม่ใช่ต่อจาก parent เดิมของมันบน branch ต้นทาง — เมื่อ parent เปลี่ยน hash ของ commit ทั้งหมดก็เปลี่ยนตามไปด้วยเป็นลูกโซ่ (เพราะ hash คำนวณรวม parent hash เข้าไปด้วย)

นอกจากนี้ **committer date** ก็จะถูกอัปเดตเป็นเวลาปัจจุบันที่คุณ cherry-pick (ต่างจาก **author date** ที่ยังคงเดิม) ซึ่งก็เป็นอีกปัจจัยที่ทำให้ hash เปลี่ยนไปด้วย

### 397.2 ตรวจสอบด้วยตาตัวเอง

```bash
git show c3d4e5f --format="Author: %an <%ae>%nAuthorDate: %ad%nCommitter: %cn <%ce>%nCommitDate: %cd" -s
git show 9f8e7d6 --format="Author: %an <%ae>%nAuthorDate: %ad%nCommitter: %cn <%ce>%nCommitDate: %cd" -s
```

จะเห็นว่า **Author** และ **AuthorDate** เหมือนกันทุกประการ (เพราะ Git เก็บข้อมูลผู้เขียนต้นฉบับไว้อย่างซื่อสัตย์) แต่ **CommitDate** ต่างกัน เพราะเป็นเวลาที่ commit ใหม่ถูกสร้างขึ้นจริงตอน cherry-pick

### 397.3 ผลลัพธ์ในทางปฏิบัติที่ต้องเข้าใจ

1. **`git log` จะแสดง commit ที่ดูเหมือนซ้ำกัน** ถ้าคุณไล่ดูประวัติของหลาย branch พร้อมกัน (เช่นด้วย `git log --all --graph`) จะเห็น commit message เดียวกันปรากฏซ้ำในหลาย branch — นี่เป็นเรื่องปกติ ไม่ใช่ข้อผิดพลาด
2. **`git branch --merged` อาจทำงานไม่ตรงกับที่คาดหวัง** เพราะ Git เปรียบเทียบด้วย commit hash ไม่ใช่เนื้อหา ถ้า branch feature ถูก cherry-pick commit บางตัวออกไปแล้ว Git จะยังไม่ถือว่า branch นั้น "ถูก merge เข้าไปแล้ว" แม้เนื้อหาจะเหมือนกันแล้วก็ตาม ทำให้บางครั้งเกิดความสับสนว่าจะลบ branch ทิ้งได้หรือยัง
3. **ถ้าภายหลัง merge branch ต้นทางเข้ากับปลายทางจริง ๆ อาจเจอ "conflict ปลอม"** เพราะ Git เห็น diff ของ commit เดิม (ที่ยังไม่ถูก apply บน branch นี้) พยายาม apply ซ้ำบนโค้ดที่มีการเปลี่ยนแปลงนั้นอยู่แล้ว — ในกรณีนี้ Git บางเวอร์ชันฉลาดพอที่จะสังเกตว่า diff เหมือนกันทุกประการและจะ merge ได้เองโดยไม่มี conflict (เพราะผลลัพธ์สุดท้ายเหมือนกัน) แต่ก็ไม่ใช่เสมอไป โดยเฉพาะถ้ามีการแก้ไขเพิ่มเติมรอบข้างหลังจากนั้น
4. **การ revert commit ที่ถูก cherry-pick ไปแล้วในหลาย branch ต้องทำแยกกันทีละ branch** เพราะแต่ละ branch มี commit hash ของตัวเอง การ `git revert <hash>` บน branch หนึ่งจะไม่ส่งผลกับ branch อื่นที่มี commit เนื้อหาเดียวกันแต่ hash ต่างกัน

### 397.4 การตามรอยด้วย `-x` ช่วยลดความสับสน

นี่คือเหตุผลสำคัญที่ควรใช้ `git cherry-pick -x` เสมอในทีม เพราะ trailer `(cherry picked from commit <hash>)` ที่ถูกเพิ่มเข้าไปในทุก commit message ทำให้:

```bash
git log --all --grep="cherry picked from commit g9h8i7j"
```

สามารถค้นหาได้ทันทีว่า commit ต้นฉบับ `g9h8i7j` ถูกกระจายไปยัง branch ไหนบ้าง เป็นเครื่องมือช่วย audit ที่มีค่ามากในทีมที่ดูแลหลาย release branch พร้อมกัน

### 397.5 ตารางสรุปสิ่งที่เปลี่ยนและไม่เปลี่ยน

| รายการ | เปลี่ยนหรือไม่ | เหตุผล |
|---|---|---|
| Commit hash | **เปลี่ยน** | parent hash และ committer date ต่างไปจากเดิม |
| เนื้อหาไฟล์ (diff) | ไม่เปลี่ยน (ถ้าไม่มี conflict) | apply diff เดียวกันทุกประการ |
| Commit message | ไม่เปลี่ยน (ยกเว้นแก้เอง หรือมี `-x` เพิ่ม trailer) | Git คัดลอกข้อความเดิมมาให้ |
| Author และ author date | ไม่เปลี่ยน | Git เก็บข้อมูลผู้เขียนต้นฉบับไว้เสมอ |
| Committer และ commit date | **เปลี่ยน** | เป็นคนและเวลาที่รัน cherry-pick จริง |

---

## Step 398: cherry-pick vs merge vs rebase เมื่อไหร่ควรใช้อะไร

ทั้งสามคำสั่งนี้ล้วนเกี่ยวข้องกับการ "นำการเปลี่ยนแปลงจากที่หนึ่งไปอีกที่หนึ่ง" แต่มีจุดประสงค์และผลลัพธ์ที่ต่างกันโดยสิ้นเชิง มาสรุปให้ชัดเจนที่สุด

### 398.1 ความแตกต่างในระดับแนวคิด

| หัวข้อ | Merge | Rebase | Cherry-pick |
|---|---|---|---|
| ขอบเขตการทำงาน | รวม**ทั้งประวัติ**ของ branch เข้าด้วยกัน | ย้าย**ทั้งชุด commit**ของ branch ไปต่อที่ฐานใหม่ | หยิบ**เฉพาะ commit ที่เลือก**ไปสร้างใหม่ที่ปลายทาง |
| จำนวน commit ที่กระทบ | ทุก commit ใน branch ทั้งสอง | ทุก commit บน branch ที่ถูก rebase | เฉพาะ commit ที่ระบุเท่านั้น |
| สร้าง commit hash ใหม่ไหม | commit เดิมไม่เปลี่ยน hash (มีแค่ merge commit ใหม่) | **เปลี่ยน hash ทุก commit** ที่ถูก rebase | **เปลี่ยน hash เฉพาะ commit ที่ถูก pick** |
| ใช้เมื่อไหร่ | ต้องการรวมงานทั้งหมดของ branch เข้าด้วยกันอย่างสมบูรณ์ | ต้องการทำประวัติให้เรียบเป็นเส้นตรง ก่อน merge เข้า branch หลัก | ต้องการแค่ commit บางตัว ไม่ใช่ทั้งสาย |
| ความเสี่ยงต่อ shared history | ต่ำ (ไม่แก้ไข commit เดิม) | สูง ถ้า rebase branch ที่คนอื่นใช้ร่วมกันอยู่ (Part 38–39) | ปานกลาง (สร้าง commit ใหม่ แต่ไม่แตะ branch ต้นทาง) |
| ผลต่อ pull request | ปกติทำใน PR merge โดยตรง | มักใช้ก่อน push เพื่อทำความสะอาดประวัติส่วนตัว | มักใช้นอกวง PR ปกติ เช่นกระจาย hotfix |

### 398.2 กรณีศึกษาเปรียบเทียบแบบเห็นภาพ

**สถานการณ์ A: feature branch พัฒนาเสร็จแล้ว พร้อมรวมเข้า main ทั้งหมด**

```
main:     A---B
               \
feature:        C---D---E

ต้องการ: เอา C, D, E ทั้งหมดเข้า main
```

ใช้ **merge** หรือ **rebase** (ไม่ใช่ cherry-pick เพราะต้องการทั้งชุด ไม่ใช่แค่บาง commit)

```bash
git switch main
git merge feature          # หรือ
git rebase main feature && git switch main && git merge feature  # (fast-forward)
```

**สถานการณ์ B: ต้องการแค่ commit D (ไม่เอา C, E) ไปใส่ branch อื่นที่ไม่เกี่ยวข้องกับ feature เลย**

```
release/1.0:  X---Y

ต้องการ: เอาแค่ D ไปใส่ release/1.0 โดยไม่แตะ C, E เลย
```

ใช้ **cherry-pick** เท่านั้น เพราะ merge/rebase จะพยายามดึงทั้งสายประวัติของ `feature` เข้ามาด้วย ซึ่งไม่ใช่สิ่งที่ต้องการ

```bash
git switch release/1.0
git cherry-pick D
```

**สถานการณ์ C: ต้องการทำความสะอาด feature branch ของตัวเอง ก่อนส่ง PR (ยังไม่มีใครใช้ branch นี้ร่วมกัน)**

ใช้ **interactive rebase** (Part 37–39) เพื่อ squash, reorder, แก้ commit message ให้สะอาดก่อน แล้วค่อย merge เข้า main

### 398.3 กฎการตัดสินใจอย่างง่าย (decision rule)

ถามตัวเองตามลำดับนี้:

1. **ต้องการเอา commit "ทุกตัว" ของ branch หนึ่งไปรวมกับอีก branch ไหม?**
   - ใช่ → ไปดูต่อว่าต้องการ merge commit ชัดเจนหรือประวัติเรียบเป็นเส้นตรง (merge vs rebase, ดู Part 38)
   - ไม่ใช่ (ต้องการแค่บางตัว) → ใช้ **cherry-pick**

2. **branch ที่จะแก้ไขประวัติ มีคนอื่นดึงไป pull ใช้งานอยู่แล้วหรือยัง?**
   - มี → **ห้าม rebase** branch นั้น ให้ใช้ merge หรือ cherry-pick แทน (กฎทองจาก Part 39)
   - ยังไม่มี (เป็น branch ส่วนตัว) → rebase ได้อย่างปลอดภัย

3. **ต้องการรักษาความสัมพันธ์ (parent-child) ระหว่าง commit เดิมไว้หรือไม่?**
   - ต้องการ → merge (เพราะไม่แก้ไข hash ของ commit เดิมเลย)
   - ไม่จำเป็น → rebase หรือ cherry-pick ก็ได้ตามสถานการณ์

### 398.4 คำเตือนสำคัญเรื่องการใช้ cherry-pick แทน merge

อย่าใช้ cherry-pick เป็นวิธี "merge ทีละ commit" สำหรับ feature branch ทั้งสาย เพราะ:

- เสียเวลามาก ต้องคัดลอกทีละ commit
- เสี่ยง conflict ซ้ำซ้อนหลายรอบโดยไม่จำเป็น
- ทำให้ประวัติของ branch ต้นทางกับปลายทาง**ไม่เชื่อมโยงกันในเชิง Git object** (แม้เนื้อหาจะเหมือนกัน) ตามที่อธิบายใน Step 397
- Git มีเครื่องมือที่ออกแบบมาสำหรับงานนั้นโดยเฉพาะอยู่แล้วคือ merge/rebase

---

## Step 399: `git cherry-pick --abort`, `--continue`, `--quit`

เมื่อ cherry-pick เข้าสู่สถานะติด conflict หรือกำลังดำเนินการอยู่ (โดยเฉพาะตอน cherry-pick หลาย commit พร้อมกัน) มีคำสั่งควบคุม 3 ตัวที่ต้องรู้จักให้แม่นเหมือนตอนเรียน merge conflict

### 399.1 `git cherry-pick --continue`

ใช้เมื่อ **แก้ conflict เสร็จแล้ว** และพร้อมให้ Git ดำเนินการ commit ต่อ (หรือ cherry-pick commit ถัดไปในคิว ถ้ามีหลายตัว)

```bash
git add <ไฟล์ที่แก้ conflict แล้ว>
git cherry-pick --continue
```

**ข้อควรระวัง:** ต้อง `git add` ไฟล์ที่แก้ conflict เสร็จแล้ว**ก่อน**เรียก `--continue` เสมอ ไม่เช่นนั้น Git จะแจ้ง error ว่ายังมี unresolved conflict อยู่

### 399.2 `git cherry-pick --abort`

ใช้เมื่อ **ต้องการยกเลิก cherry-pick ทั้งหมด** แล้วกลับไปยังสถานะก่อนเริ่ม cherry-pick ทุกประการ เหมือนไม่มีอะไรเกิดขึ้น

```bash
git cherry-pick --abort
```

คำสั่งนี้เหมาะเมื่อคุณเจอ conflict ที่ซับซ้อนเกินไป หรือพบว่า cherry-pick commit ที่ระบุนั้น**ไม่ควรทำตั้งแต่แรก** — `--abort` จะคืนค่า working directory และ HEAD กลับไปเป็นเหมือนก่อนรันคำสั่ง cherry-pick เลยทุกจุด

**ข้อจำกัด:** `--abort` ใช้ได้เฉพาะตอนที่ cherry-pick operation ยังไม่เสร็จ (คือยังมี conflict ค้างอยู่ หรืออยู่ระหว่างขั้นตอน) ถ้า cherry-pick สำเร็จไปแล้วและสร้าง commit เรียบร้อยแล้ว ต้องใช้ `git reset` หรือ `git revert` แทนในการยกเลิกผลลัพธ์ (ดู Part 14 เรื่อง undo)

### 399.3 `git cherry-pick --quit`

ใช้เมื่อ **ต้องการหยุด cherry-pick operation แต่เก็บการเปลี่ยนแปลงที่ทำไปแล้วไว้** (ต่างจาก `--abort` ที่ยกเลิกทุกอย่างกลับไปจุดเริ่มต้น)

```bash
git cherry-pick --quit
```

หลังรันคำสั่งนี้ Git จะ**เลิกติดตามสถานะ cherry-pick** (ล้างไฟล์ `.git/CHERRY_PICK_HEAD` และ sequencer state) แต่**ไม่แตะ working directory หรือไฟล์ที่ staged ไว้** — เหมาะสำหรับตอนที่คุณแก้ conflict เสร็จแล้วในใจ แต่อยากจัดการ commit เอง ไม่ต้องการให้ Git ทำ flow ต่อให้อัตโนมัติ

### 399.4 ตารางสรุปเปรียบเทียบทั้ง 3 คำสั่ง

| คำสั่ง | working directory หลังใช้ | เหมาะกับสถานการณ์ |
|---|---|---|
| `--continue` | ดำเนินต่อ สร้าง commit สำเร็จ | แก้ conflict เสร็จแล้ว พร้อมไปต่อ |
| `--abort` | คืนกลับสู่สถานะก่อนเริ่ม cherry-pick ทั้งหมด | ต้องการยกเลิกทุกอย่าง เหมือนไม่เคยทำ |
| `--quit` | คงการเปลี่ยนแปลงปัจจุบันไว้ แต่เลิก track sequencer | ต้องการจัดการ commit ต่อเอง นอกเหนือ flow ปกติ |

### 399.5 ตรวจสอบสถานะระหว่างทางด้วย `git status`

ทุกครั้งที่ไม่แน่ใจว่า repository อยู่ในสถานะ cherry-pick ค้างอยู่หรือไม่ ให้รัน:

```bash
git status
```

ถ้ายังมี cherry-pick ค้างอยู่ Git จะแสดงข้อความ `You are currently cherry-picking commit <hash>.` เสมอ พร้อมคำแนะนำขั้นตอนถัดไปกำกับไว้ให้อัตโนมัติ นอกจากนี้ยังตรวจสอบได้ตรง ๆ ด้วยการดูว่าไฟล์นี้มีอยู่หรือไม่:

```bash
ls .git/CHERRY_PICK_HEAD 2>/dev/null && echo "กำลัง cherry-pick อยู่" || echo "ไม่มี cherry-pick ค้างอยู่"
```

---

## Step 400: แบบฝึกหัด — จำลอง hotfix ที่ต้อง cherry-pick ไปหลาย branch release

ถึงเวลาลงมือทำแบบฝึกหัดเต็มรูปแบบที่รวมทุกอย่างที่เรียนมาใน Part นี้เข้าด้วยกัน จำลองสถานการณ์การกระจาย hotfix ไปยัง `release/1.0` และ `release/2.0` เหมือนงานจริง

### 400.1 เตรียม repository จำลอง

```bash
mkdir ~/git-course/part-40-exercise
cd ~/git-course/part-40-exercise
git init
git config user.name "Somchai Devtest"
git config user.email "somchai@example.com"

echo "version: 3.0-dev" > VERSION
echo "function login() { return true; }" > auth.js
git add .
git commit -m "init: โครงสร้างเริ่มต้นของโปรเจกต์"
```

### 400.2 สร้าง release branch ทั้งสองเวอร์ชัน แยกจาก main

```bash
git branch release/1.0
git branch release/2.0

git switch release/1.0
echo "version: 1.0" > VERSION
git add VERSION
git commit -m "release: กำหนดเวอร์ชัน 1.0"

git switch release/2.0
echo "version: 2.0" > VERSION
git add VERSION
git commit -m "release: กำหนดเวอร์ชัน 2.0"

git switch main
echo "version: 3.0-dev" > VERSION
git add VERSION
git commit -m "dev: ทำงานต่อบนเวอร์ชัน 3.0-dev"
```

ตอนนี้ทั้ง 3 branch แยกออกจากกันแล้ว แต่ละ branch มีไฟล์ `VERSION` เนื้อหาต่างกัน

### 400.3 จำลองการค้นพบ security bug และแก้ไขบน main

```bash
git switch main
cat > auth.js << 'EOF'
function login(username, password) {
  if (!username || !password) {
    throw new Error("username และ password ห้ามว่าง");
  }
  // เดิม: ไม่มีการตรวจสอบ SQL injection
  // แก้ไข: ใช้ parameterized query แทนการต่อ string ตรง ๆ
  return db.query("SELECT * FROM users WHERE username = ? AND password = ?", [username, password]);
}
EOF
git add auth.js
git commit -m "security: แก้ SQL Injection ในฟังก์ชัน login (CVE-2026-0040)"
```

### 400.4 หา hash ของ commit hotfix

```bash
git log --oneline -3 main
```

จดค่า hash ของ commit `security: แก้ SQL Injection...` ไว้ (สมมติชื่อตัวแปรว่า `$HOTFIX`)

```bash
HOTFIX=$(git log --format="%H" -n 1 --grep="CVE-2026-0040")
echo "Hotfix commit: $HOTFIX"
```

### 400.5 cherry-pick เข้า release/2.0 (คาดว่าจะไม่มี conflict เพราะ auth.js ยังไม่เคยถูกแก้ไขที่ release/2.0)

```bash
git switch release/2.0
git cherry-pick -x $HOTFIX
```

ตรวจสอบผลลัพธ์:

```bash
cat auth.js
git log --oneline -3
```

ควรเห็น commit ใหม่ที่มีเนื้อหาแก้บั๊กเหมือนกัน พร้อม trailer `(cherry picked from commit ...)`

### 400.6 cherry-pick เข้า release/1.0 (จำลองสถานการณ์ที่มี conflict)

ก่อนอื่นจำลองว่า `release/1.0` มีการแก้ไข `auth.js` ไปคนละทางแล้ว (เช่นเวอร์ชันเก่ายังใช้โครงสร้างฟังก์ชันแบบเดิม):

```bash
git switch release/1.0
cat > auth.js << 'EOF'
function login(username, password) {
  // เวอร์ชัน 1.0 ใช้ logic แบบเก่า ยังไม่มีการปรับ error handling ใหม่
  return db.query("SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'");
}
EOF
git add auth.js
git commit -m "legacy: คงโครงสร้างฟังก์ชัน login แบบเดิมของเวอร์ชัน 1.0"
```

ตอนนี้ลอง cherry-pick hotfix เข้ามา:

```bash
git cherry-pick -x $HOTFIX
```

คาดว่าจะเจอ conflict เพราะทั้งสองฝั่งแก้ไขฟังก์ชัน `login` ในบริเวณเดียวกัน:

```
Auto-merging auth.js
CONFLICT (content): Merge conflict in auth.js
error: could not apply ...
```

### 400.7 แก้ conflict อย่างถูกต้อง (ต้องคง logic ความปลอดภัยไว้เสมอ)

เปิดไฟล์ `auth.js` จะเห็น marker แบบนี้:

```
function login(username, password) {
<<<<<<< HEAD
  // เวอร์ชัน 1.0 ใช้ logic แบบเก่า ยังไม่มีการปรับ error handling ใหม่
  return db.query("SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'");
=======
  if (!username || !password) {
    throw new Error("username และ password ห้ามว่าง");
  }
  // เดิม: ไม่มีการตรวจสอบ SQL injection
  // แก้ไข: ใช้ parameterized query แทนการต่อ string ตรง ๆ
  return db.query("SELECT * FROM users WHERE username = ? AND password = ?", [username, password]);
>>>>>>> ... (security: แก้ SQL Injection ในฟังก์ชัน login (CVE-2026-0040))
}
```

**หลักการแก้ที่ถูกต้อง:** เนื่องจากนี่คือ hotfix ด้านความปลอดภัย ต้อง**เลือกฝั่งที่แก้ไขปลอดภัยไว้เสมอ** แม้จะต้องปรับให้เข้ากับโครงสร้างของเวอร์ชัน 1.0 บ้างก็ตาม:

```javascript
function login(username, password) {
  if (!username || !password) {
    throw new Error("username และ password ห้ามว่าง");
  }
  // แก้ไข: ใช้ parameterized query แทนการต่อ string ตรง ๆ (นำเข้าจาก hotfix ความปลอดภัย)
  return db.query("SELECT * FROM users WHERE username = ? AND password = ?", [username, password]);
}
```

บันทึกไฟล์ แล้ว mark ว่าแก้เสร็จ:

```bash
git add auth.js
git cherry-pick --continue
```

บันทึก commit message ตามค่า default แล้วปิด editor

### 400.8 ตรวจสอบผลลัพธ์ทั้ง 3 branch

```bash
for b in main release/1.0 release/2.0; do
  echo "=== $b ==="
  git log --oneline -3 "$b"
  echo
done
```

ควรเห็นว่าทั้ง 3 branch มี commit ที่เกี่ยวข้องกับการแก้ SQL Injection อยู่ (คนละ hash กัน) และไฟล์ `VERSION` ของแต่ละ branch ยังคงแตกต่างกันตามเวอร์ชันของตัวเอง ไม่ถูกกระทบจาก cherry-pick เลย:

```bash
for b in main release/1.0 release/2.0; do
  echo "=== $b : VERSION ==="
  git show "$b:VERSION"
done
```

### 400.9 ติด tag ปิดงาน patch

```bash
git switch release/2.0
git tag v2.0.1-hotfix

git switch release/1.0
git tag v1.0.5-hotfix

git tag
```

### 400.10 คำถามทบทวนก่อนปิด Part (ลองตอบเองก่อนดูเฉลย)

1. ทำไมเราไม่ใช้ `git merge main` เข้า `release/1.0` โดยตรงในสถานการณ์นี้
   - **เฉลย:** เพราะ `main` มีการเปลี่ยนแปลงอื่น ๆ ที่ยังไม่พร้อมสำหรับเวอร์ชัน 1.0 (เช่นโค้ดของเวอร์ชัน 3.0-dev) การ merge ทั้งหมดจะดึงสิ่งที่ไม่เกี่ยวข้องเข้ามาปนด้วย
2. ถ้าตรวจสอบด้วย `git branch --contains $HOTFIX` บน `release/1.0` และ `release/2.0` จะเจอ branch เหล่านี้หรือไม่ เพราะอะไร
   - **เฉลย:** ไม่เจอ เพราะ `git branch --contains` เช็คจาก commit hash ที่แท้จริง ไม่ใช่เนื้อหา และ commit ที่ cherry-pick ไปมี hash ใหม่ทั้งหมด (ตาม Step 397) ต้องใช้ `git log --all --grep="CVE-2026-0040"` แทนถ้าต้องการค้นหาด้วยเนื้อหาหรือ trailer
3. ถ้าภายหลัง `release/1.0` และ `release/2.0` ถูก merge รวมเข้ากับ `main` จริง ๆ (เช่นตอนปิด sprint) จะเกิดอะไรขึ้นกับ commit hotfix ที่ซ้ำกันอยู่แล้วบน `main`
   - **เฉลย:** Git จะเห็นว่า diff ที่จะ apply ซ้ำกับที่มีอยู่แล้วใน `main` ทำให้ในกรณีส่วนใหญ่ merge จะผ่านไปได้โดยไม่มี conflict (เพราะผลลัพธ์สุดท้ายเหมือนกันอยู่แล้ว) แต่ `git log` ของ `main` จะยังคงมี commit hotfix ปรากฏซ้ำสองครั้ง (ตัวเดิมกับตัวที่ merge เข้ามาจาก release branch) ซึ่งเป็นเรื่องปกติที่ยอมรับได้ในทางปฏิบัติ

### 400.11 Checklist ก่อนปิด Part 40

ก่อนไปต่อ Part 41 ให้ตรวจสอบว่าคุณ:

- [ ] อธิบายได้ว่า cherry-pick คืออะไร ต่างจาก merge/rebase อย่างไรในระดับแนวคิด
- [ ] cherry-pick commit เดียวข้าม branch ได้เองโดยไม่ต้องดูตัวอย่าง
- [ ] cherry-pick หลาย commit พร้อมกันได้ ทั้งแบบระบุหลาย hash และแบบ range `A..B` / `A^..B`
- [ ] แก้ conflict ระหว่าง cherry-pick ได้ และรู้จักใช้ `--continue`, `--abort`, `--skip`, `--quit` อย่างถูกจังหวะ
- [ ] อธิบายได้ว่า `--no-commit`/`-n` ใช้ทำอะไร และใช้ในสถานการณ์ไหนบ้าง
- [ ] เข้าใจว่าทำไม commit ที่ถูก cherry-pick จึงมี hash ใหม่เสมอ แม้เนื้อหาจะเหมือนเดิม
- [ ] เลือกใช้ cherry-pick, merge หรือ rebase ได้ถูกต้องตามสถานการณ์จริง โดยไม่ใช้ผิดที่ผิดทาง
- [ ] ลงมือจำลองสถานการณ์กระจาย hotfix ไปยังหลาย release branch ได้ครบทุกขั้นตอนด้วยตัวเอง

---

## สรุป Part 40

ใน Part นี้เราได้เรียนรู้ว่า:

1. `git cherry-pick` คือเครื่องมือสำหรับคัดลอกการเปลี่ยนแปลงของ **commit เฉพาะตัว** จาก branch หนึ่งไปสร้างเป็น commit ใหม่บน branch ปัจจุบัน โดยไม่ต้องเอา commit อื่นที่ไม่เกี่ยวข้องติดมาด้วย
2. ขั้นตอนพื้นฐานคือ สลับไป branch ปลายทาง แล้วรัน `git cherry-pick <hash>` และควรใช้ `-x` เสมอเพื่อบันทึกที่มาของ commit ต้นฉบับไว้ใน commit message
3. สามารถ cherry-pick หลาย commit พร้อมกันได้ ทั้งแบบระบุหลาย hash ตามลำดับ หรือใช้ syntax range `A..B` (ไม่รวม A) และ `A^..B` (รวม A)
4. Conflict ระหว่าง cherry-pick แก้ไขด้วยวิธีเดียวกับ merge conflict คือแก้ marker ในไฟล์ แล้ว `git add` และ `git cherry-pick --continue`
5. `--no-commit`/`-n` มีประโยชน์มากเมื่อต้องการรวมหลาย commit เป็นก้อนเดียว หรือต้องการตรวจสอบ/แก้ไขเพิ่มเติมก่อน commit จริง
6. สถานการณ์จริงที่ใช้ cherry-pick บ่อยที่สุดคือการกระจาย **hotfix ด้านความปลอดภัยหรือบั๊กร้ายแรง** ไปยังหลาย release branch ที่ยังต้อง maintain พร้อมกัน โดยไม่ดึงฟีเจอร์อื่นที่ยังไม่พร้อมเข้ามาปนด้วย
7. commit ที่ผ่านการ cherry-pick จะมี **hash ใหม่เสมอ** เพราะ parent commit และ committer date เปลี่ยนไป แม้เนื้อหาไฟล์จะเหมือนเดิมทุกตัวอักษร ซึ่งส่งผลต่อการตรวจสอบด้วย `git branch --contains` และการ merge ในอนาคต
8. หลักการเลือกใช้ที่ถูกต้อง: **merge/rebase** สำหรับเอา commit ทั้งชุดของ branch, **cherry-pick** สำหรับหยิบเฉพาะบาง commit ที่ต้องการจริง ๆ เท่านั้น
9. คำสั่งควบคุม operation ที่ต้องจำ: `--continue` (ไปต่อหลังแก้ conflict), `--abort` (ยกเลิกทั้งหมดกลับจุดเริ่มต้น), `--quit` (หยุดติดตามสถานะแต่คงการเปลี่ยนแปลงไว้), `--skip` (ข้าม commit ที่ติด conflict ไปเลย)
10. ผ่านแบบฝึกหัดจำลอง hotfix ความปลอดภัยที่ต้องกระจายไปยัง `release/1.0` และ `release/2.0` พร้อมแก้ conflict จริงและติด tag ปิดงาน patch ครบวงจร

**ต่อไป:** [Part 41: การแก้ Merge Conflict ขั้นสูงและเครื่องมือช่วย](./part-041-merge-conflict-ขั้นสูง.md)
