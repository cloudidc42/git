# Part 12: Stash: การพักงานชั่วคราวโดยไม่ commit

> **Step ในหลักสูตรนี้:** Step 111–120
> **เฟส:** 2 — ทำงานกับ Git ในสถานการณ์จริงที่ซับซ้อนขึ้นกว่าคำสั่งพื้นฐาน
> **เป้าหมายของ Part นี้:** เข้าใจว่า `git stash` คืออะไร แก้ปัญหาอะไรในชีวิตการทำงานจริง สามารถเก็บพักงานที่ยังไม่เสร็จไปชั่วคราว สลับไปทำงานด่วนอื่น แล้วกลับมาทำงานเดิมต่อได้อย่างปลอดภัย โดยไม่ต้อง commit โค้ดที่ยังไม่พร้อม และไม่ต้องกลัวงานหาย

---

## สารบัญของ Part นี้

- Step 111: Stash คืออะไร แก้ปัญหาอะไร
- Step 112: `git stash` พื้นฐาน — เก็บการเปลี่ยนแปลงไปพัก ทำให้ working directory สะอาด
- Step 113: `git stash list`, `git stash show`, `git stash show -p` — ดูรายละเอียดของสิ่งที่ถูกพักไว้
- Step 114: `git stash pop` vs `git stash apply` — ความแตกต่างที่ต้องเข้าใจให้ชัด
- Step 115: `git stash drop`, `git stash clear` — ลบทีละอันหรือลบทั้งหมด
- Step 116: ตั้งชื่อ stash ให้จำง่ายด้วย `git stash push -m "message"`
- Step 117: Stash เฉพาะบางไฟล์ด้วย `git stash push -- <file>`
- Step 118: `git stash -u` / `--include-untracked` — รวมไฟล์ที่ยังไม่ถูก track เข้าไปด้วย
- Step 119: สร้าง branch จาก stash โดยตรงด้วย `git stash branch` — แก้ปัญหา conflict ตอน apply
- Step 120: แบบฝึกหัด — จำลองสถานการณ์ต้องหยุดงานกะทันหันเพื่อไปแก้บั๊กด่วน แล้วกลับมาทำงานเดิมต่อ

---

## Step 111: Stash คืออะไร แก้ปัญหาอะไร

### สถานการณ์จริงที่ทุกคนต้องเจอ

ลองนึกภาพสถานการณ์นี้: คุณกำลังนั่งเขียนฟีเจอร์ใหม่บน branch `feature/user-profile` ทำไปได้ครึ่งทาง แก้ไขไฟล์ไปแล้ว 5-6 ไฟล์ โค้ดยังไม่สมบูรณ์ ยังคอมไพล์ไม่ผ่านด้วยซ้ำ จู่ ๆ หัวหน้าทีมก็เดินมาบอกว่า

> "มีบั๊กร้ายแรงบน production ต้องแก้ด่วนตอนนี้เลย ไปที่ branch `main` แล้ว hotfix ให้หน่อย"

คุณเจอปัญหาทันที: **คุณอยู่ตรงกลางของงานที่ยังไม่เสร็จ ยังไม่อยาก commit เพราะโค้ดยังไม่สมบูรณ์ ไม่ผ่าน test แต่ก็ต้องสลับไปทำงานอื่นเดี๋ยวนี้**

ลองมาดูว่าถ้าไม่มี `git stash` คุณจะต้องทำอย่างไรบ้าง — และทำไมแต่ละทางเลือกถึงไม่ดี:

**ทางเลือกที่ 1: commit งานที่ยังไม่เสร็จไปก่อน**

```bash
git add .
git commit -m "wip: ยังไม่เสร็จ อย่าเพิ่งดู"
```

ปัญหาคือ: history ของโปรเจกต์จะเต็มไปด้วย commit ขยะแบบ "wip", "fix", "temp" ที่ไม่มีความหมาย ทำให้ `git log` อ่านยาก และถ้าทีมอื่นดึง branch นี้ไปดูจะงงว่า commit นี้คืออะไร นอกจากนี้ถ้าโค้ดยังไม่ผ่าน pre-commit hook (เช่น linter หรือ test) การ commit อาจถูกบล็อกอีกด้วย

**ทางเลือกที่ 2: copy โฟลเดอร์ทั้งหมดไปเก็บไว้ที่อื่นก่อน**

```bash
cp -r my-project my-project-backup
```

ปัญหาคือ: เสียเวลา กินพื้นที่ดิสก์ และเสี่ยงต่อความผิดพลาดสูงมาก (ลืมเอากลับมา ลืมว่า backup ไหนคือเวอร์ชันล่าสุด) ไม่ใช่วิธีที่มืออาชีพควรทำ

**ทางเลือกที่ 3: ลบการเปลี่ยนแปลงทิ้งไปเลย แล้วค่อยพิมพ์ใหม่**

แทบเป็นไปไม่ได้ในทางปฏิบัติ และเสี่ยงต่อการลืมรายละเอียดที่คิดไว้

### Stash คือคำตอบที่ถูกต้อง

**`git stash`** คือฟีเจอร์ของ Git ที่ทำหน้าที่:

> **เก็บการเปลี่ยนแปลงทั้งหมดที่ยังไม่ถูก commit (ทั้งใน working directory และ staging area) ไปไว้ใน "ที่พักชั่วคราว" แยกต่างหาก แล้วคืนค่า working directory ให้กลับไปสะอาดเหมือนกับ commit ล่าสุด**

พูดง่าย ๆ stash คือ **"ลิ้นชักพักงาน"** ที่ให้คุณ:

1. เก็บงานที่ทำค้างไว้แบบไม่ต้อง commit
2. ทำให้ working directory สะอาด พร้อมสลับ branch หรือทำงานอื่นได้ทันที
3. เมื่อพร้อมแล้ว ดึงงานที่พักไว้กลับมาทำต่อได้เหมือนไม่มีอะไรเกิดขึ้น

### เปรียบเทียบให้เห็นภาพ

ลองนึกถึงโต๊ะทำงานจริง ๆ ของคุณ:

- **Commit** คือการเอางานที่เสร็จสมบูรณ์แล้วไปเข้าแฟ้มถาวร มีป้ายชื่อชัดเจน
- **Stash** คือการเอากระดาษที่กำลังเขียนอยู่ (ยังไม่เสร็จ ยังไม่เรียบร้อย) ไปสอดไว้ใน **ลิ้นชักข้าง ๆ โต๊ะ** ชั่วคราว เพื่อให้โต๊ะว่างสำหรับงานด่วนอื่น แล้วเมื่อว่างก็หยิบกระดาษแผ่นนั้นออกมาเขียนต่อ

### Stash เก็บอะไรบ้าง

โดยค่าเริ่มต้น `git stash` จะเก็บ:

| ประเภทการเปลี่ยนแปลง | ถูกเก็บโดยค่าเริ่มต้นหรือไม่ |
|---|---|
| ไฟล์ที่ถูกแก้ไข และอยู่ใน staging area (`git add` แล้ว) | เก็บ |
| ไฟล์ที่ถูกแก้ไข แต่ยังไม่ได้ `git add` (unstaged) | เก็บ |
| ไฟล์ใหม่ที่ยังไม่เคย track เลย (untracked files) | **ไม่เก็บ** (ต้องใช้ `-u` เพิ่ม จะสอนใน Step 118) |
| ไฟล์ที่ถูก `.gitignore` (ignored files) | **ไม่เก็บ** เลยแม้จะใช้ `-u` (ต้องใช้ `-a` หรือ `--all`) |

### จุดสำคัญที่ต้องเข้าใจ: Stash ไม่ใช่ Commit

Stash **ไม่ได้** สร้าง commit บน branch ปัจจุบัน แต่มันสร้าง **object พิเศษ** ที่เก็บไว้แยกต่างหากในโครงสร้างข้อมูลของ Git เรียกว่า **stash stack** (กองซ้อนแบบ LIFO — Last In, First Out เหมือนกองจานที่วางซ้อนกัน อันล่าสุดจะถูกหยิบออกก่อน)

เราจะเจาะลึกเรื่องนี้ในเชิงเทคนิคเพิ่มเติมใน Step 112 และ Step 119

### สรุป Step 111

- Stash แก้ปัญหา "ต้องสลับงานกะทันหัน แต่งานปัจจุบันยังไม่เสร็จและไม่อยาก commit"
- Stash เก็บการเปลี่ยนแปลงไว้ชั่วคราว แล้วคืน working directory ให้สะอาด
- Stash ไม่สร้าง commit บน branch ปัจจุบัน แต่เก็บไว้ในโครงสร้างข้อมูลแยกต่างหาก (stash stack)
- โดยค่าเริ่มต้น stash **ไม่เก็บ** untracked files และ ignored files

---

## Step 112: `git stash` พื้นฐาน — เก็บการเปลี่ยนแปลงไปพัก ทำให้ working directory สะอาด

### คำสั่งพื้นฐานที่สุด

สมมติคุณกำลังแก้ไขไฟล์อยู่:

```bash
cd ~/git-course/part-12-practice
git init
echo "console.log('hello');" > app.js
git add app.js
git commit -m "initial commit"

echo "console.log('feature ยังไม่เสร็จ');" >> app.js
git status
```

ผลลัพธ์ของ `git status`:

```
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   app.js

no changes added to commit (use "git add" and/or "git commit -y" to commit)
```

ตอนนี้ถ้าคุณต้องสลับ branch แบบด่วน ให้รันคำสั่ง:

```bash
git stash
```

ผลลัพธ์:

```
Saved working directory and index state WIP on main: a1b2c3d initial commit
```

ลองเช็ค `git status` อีกครั้ง:

```bash
git status
```

```
On branch main
nothing to commit, working tree clean
```

**working directory สะอาดสนิททันที** เหมือนไม่มีอะไรเกิดขึ้นเลย! ตอนนี้คุณสามารถ `git switch main`, `git checkout hotfix-branch`, หรือทำอะไรก็ได้ที่ต้องการ working directory ที่สะอาด

### `git stash` คือ shorthand ของอะไร

`git stash` แบบไม่ใส่ subcommand เป็นการเรียก **`git stash push`** โดยปริยาย (ในเวอร์ชัน Git ใหม่ ๆ ตั้งแต่ 2.16 เป็นต้นมา) ทั้งสองคำสั่งนี้เหมือนกันทุกประการ:

```bash
git stash
# เท่ากับ
git stash push
```

`push` เป็นชื่อ subcommand ที่ถูกต้องตามความหมาย เพราะมันคือการ "push" (ดัน) สถานะปัจจุบันเข้าไปในกองซ้อน (stack) ของ stash

> **หมายเหตุด้านความเข้ากันได้:** ก่อนหน้า Git 2.16 คำสั่ง `git stash save "message"` เป็นวิธีเก่าในการใส่ข้อความ ปัจจุบันคำสั่งนี้ถูกแทนที่ด้วย `git stash push -m "message"` แล้ว (จะสอนละเอียดใน Step 116) แต่ `git stash save` ยังใช้งานได้อยู่เพื่อความเข้ากันได้ย้อนหลัง เพียงแต่ไม่แนะนำให้ใช้ในโค้ดใหม่

### เกิดอะไรขึ้นเบื้องหลังเมื่อสั่ง `git stash`

เมื่อคุณสั่ง `git stash` Git จะทำสิ่งต่อไปนี้ตามลำดับ:

1. สร้าง commit พิเศษ (จริง ๆ แล้วสร้างถึง 2-3 commit ซ้อนกัน) ที่เก็บสถานะของ **working directory** และ **staging area** ณ ขณะนั้น
2. ผูก commit พิเศษนี้เข้ากับ reference ที่ชื่อ `refs/stash`
3. คืนค่า working directory และ staging area กลับไปให้ตรงกับ `HEAD` (commit ล่าสุด) ทุกประการ

คุณสามารถพิสูจน์ได้ว่า stash แท้จริงแล้วคือ commit ชนิดหนึ่ง โดยลองรัน:

```bash
git log --oneline -g refs/stash
```

หรือดูรายละเอียดเชิงลึกกว่านั้นได้ด้วย:

```bash
git show stash@{0}
```

### ค่า exit ของ `git stash` เมื่อไม่มีอะไรให้ stash

ถ้า working directory สะอาดอยู่แล้ว (ไม่มีการเปลี่ยนแปลงใด ๆ) แล้วสั่ง `git stash` Git จะแจ้งเตือน:

```
No local changes to save
```

และจะไม่สร้าง stash ใหม่ขึ้นมา ซึ่งเป็นพฤติกรรมที่ปลอดภัย ไม่ทำให้เกิด stash ว่างเปล่าโดยไม่ตั้งใจ

### จุดที่มือใหม่มักเข้าใจผิด

หลายคนเข้าใจผิดว่า `git stash` เป็นการ "ลบ" งานทิ้ง แต่ในความเป็นจริง **stash ไม่ได้ลบข้อมูลอะไรเลย** มันแค่ย้ายข้อมูลจาก working directory ไปเก็บไว้ในที่อื่นชั่วคราวเท่านั้น ข้อมูลยังคงอยู่ครบถ้วน 100% และสามารถดึงกลับมาได้เสมอ (ตราบใดที่ยังไม่ลบ stash ทิ้งด้วยคำสั่งที่จะสอนใน Step 115)

### สรุป Step 112

- `git stash` (เท่ากับ `git stash push`) เก็บการเปลี่ยนแปลงที่ยังไม่ commit ไปไว้ใน stash stack
- หลัง stash แล้ว working directory จะกลับไปสะอาดเหมือน commit ล่าสุดทันที
- เบื้องหลังคือการสร้าง commit พิเศษที่ผูกกับ `refs/stash`
- ถ้าไม่มีอะไรเปลี่ยนแปลงเลย Git จะแจ้ง "No local changes to save" และไม่สร้าง stash เปล่า

---

## Step 113: `git stash list`, `git stash show`, `git stash show -p` — ดูรายละเอียดของสิ่งที่ถูกพักไว้

### ทำไมต้องดูรายการ stash

ในการทำงานจริง คุณอาจ stash งานหลายครั้งติดต่อกันโดยไม่ได้ตั้งใจ (เช่น stash วันนี้ 1 ครั้ง เมื่อวาน 2 ครั้ง) ถ้าจำไม่ได้ว่าแต่ละอันคืองานอะไร คุณจะเสี่ยงต่อการดึงงานผิดอันกลับมา หรือลืมว่ามีงานค้างอยู่ในลิ้นชัก

### `git stash list` — ดูรายการ stash ทั้งหมด

```bash
git stash list
```

ตัวอย่างผลลัพธ์เมื่อมี stash หลายอัน:

```
stash@{0}: WIP on main: a1b2c3d initial commit
stash@{1}: WIP on feature/login: e4f5g6h เพิ่มฟอร์ม login
stash@{2}: On feature/payment: c7d8e9f แก้ validation
```

สังเกตรูปแบบของแต่ละบรรทัด:

```
stash@{N}: <WIP on|On> <branch-name>: <short-hash> <commit-message>
```

- **`stash@{N}`** คือ **ดัชนี (index)** ของ stash นั้น โดย `stash@{0}` คือ**อันล่าสุด**เสมอ (นี่คือหลักการ LIFO — Last In, First Out) ยิ่งตัวเลขมาก ยิ่งเป็น stash ที่เก่ากว่า
- **`WIP on <branch>`** หมายถึง stash นี้ถูกสร้างด้วย `git stash` แบบไม่ใส่ข้อความเอง (WIP = Work In Progress)
- **`On <branch>`** หมายถึง stash นี้ถูกตั้งชื่อเองด้วย `-m` (จะสอนใน Step 116)
- **`<short-hash> <commit-message>`** คือ commit ล่าสุดที่ branch นั้นชี้อยู่ตอนที่สร้าง stash — เป็นตัวช่วยจำว่า ณ ตอนนั้นคุณอยู่ที่จุดไหนของประวัติ

### `git stash show` — ดูสรุปการเปลี่ยนแปลงของ stash

คำสั่งนี้แสดง **สถิติสรุป** (คล้าย `git diff --stat`) ของ stash ล่าสุด:

```bash
git stash show
```

ผลลัพธ์ตัวอย่าง:

```
 app.js | 2 ++
 style.css | 5 +++--
 2 files changed, 5 insertions(+), 2 deletions(-)
```

ถ้าต้องการดู stash อันอื่นที่ไม่ใช่อันล่าสุด ให้ระบุ index ต่อท้าย:

```bash
git stash show stash@{2}
```

### `git stash show -p` — ดู diff แบบละเอียดเต็ม ๆ

ถ้าอยากเห็นว่าแต่ละบรรทัดถูกแก้ไขอย่างไรจริง ๆ (ไม่ใช่แค่สรุปตัวเลข) ให้ใช้ flag `-p` (patch) หรือ `--patch`:

```bash
git stash show -p
```

ผลลัพธ์จะเป็นรูปแบบ diff เต็มรูปแบบ เหมือนกับ `git diff`:

```diff
diff --git a/app.js b/app.js
index e69de29..8c7f8a1 100644
--- a/app.js
+++ b/app.js
@@ -1 +1,2 @@
 console.log('hello');
+console.log('feature ยังไม่เสร็จ');
```

และดูของ stash อันอื่นได้เช่นกัน:

```bash
git stash show -p stash@{1}
```

### ทางลัดที่ใช้บ่อย: `git stash show -p stash@{0}` เท่ากับอะไร

หากไม่ระบุ index ใด ๆ Git จะถือว่าคุณหมายถึง `stash@{0}` เสมอ ดังนั้นคำสั่งเหล่านี้เทียบเท่ากันทุกประการ:

```bash
git stash show
git stash show -p
git stash show stash@{0}
git stash show -p stash@{0}
```

### เปรียบเทียบระดับความละเอียดของการดูข้อมูล stash

| คำสั่ง | แสดงอะไร | เหมาะกับ |
|---|---|---|
| `git stash list` | รายการ stash ทั้งหมด แบบบรรทัดเดียวต่ออัน | ดูภาพรวมว่ามี stash อะไรค้างอยู่บ้าง |
| `git stash show` | สรุปไฟล์ที่เปลี่ยน + จำนวนบรรทัดที่เพิ่ม/ลบ | เช็คคร่าว ๆ ว่า stash นี้แตะไฟล์อะไรบ้าง |
| `git stash show -p` | diff แบบเต็มทุกบรรทัด | ตรวจสอบเนื้อหาจริงก่อนตัดสินใจ apply/pop |

### แนวปฏิบัติที่ดี

ก่อนจะ `pop` หรือ `apply` stash ที่เก็บไว้นานแล้ว ควรใช้ `git stash show -p` ตรวจสอบก่อนเสมอว่าเนื้อหายังตรงกับที่คุณต้องการหรือไม่ เพราะบางครั้งโค้ดใน branch ปัจจุบันอาจเปลี่ยนไปมากแล้วตั้งแต่ตอนที่ stash ไว้ การเช็คก่อนช่วยลดโอกาสเกิด conflict ที่ไม่คาดคิด

### สรุป Step 113

- `git stash list` แสดงรายการ stash ทั้งหมด โดย `stash@{0}` คืออันล่าสุดเสมอ (LIFO)
- `git stash show` แสดงสรุปไฟล์ที่เปลี่ยนแปลง
- `git stash show -p` แสดง diff แบบละเอียดเต็มรูปแบบ
- สามารถระบุ `stash@{N}` ต่อท้ายคำสั่งเหล่านี้เพื่อดู stash อันอื่นที่ไม่ใช่อันล่าสุดได้

---

## Step 114: `git stash pop` vs `git stash apply` — ความแตกต่างที่ต้องเข้าใจให้ชัด

นี่คือหนึ่งในจุดที่มือใหม่สับสนบ่อยที่สุดเกี่ยวกับ stash เพราะทั้งสองคำสั่งดูเหมือนทำสิ่งเดียวกัน คือ "เอา stash กลับมาใส่ใน working directory" แต่มีความแตกต่างสำคัญ 1 จุดที่ต้องจำให้แม่น

### `git stash apply` — นำกลับมาใช้ แต่ยังเก็บสำเนาไว้ใน stash list

```bash
git stash apply
```

พฤติกรรม:

1. นำการเปลี่ยนแปลงจาก `stash@{0}` (อันล่าสุด) มาใส่ใน working directory ปัจจุบัน
2. **stash อันนั้นยังคงอยู่ใน `git stash list` เหมือนเดิม ไม่ถูกลบออกไป**

ตัวอย่าง:

```bash
git stash apply
git stash list
```

```
stash@{0}: WIP on main: a1b2c3d initial commit
```

จะเห็นว่า stash ยังอยู่ครบ แม้จะ apply ไปแล้ว

### `git stash pop` — นำกลับมาใช้ แล้วลบออกจาก stash list ทันที

```bash
git stash pop
```

พฤติกรรม:

1. นำการเปลี่ยนแปลงจาก `stash@{0}` มาใส่ใน working directory
2. **ถ้า apply สำเร็จโดยไม่มี conflict** stash อันนั้นจะถูกลบออกจาก stash list โดยอัตโนมัติทันที (เหมือนการ "pop" ของ stack — หยิบออกแล้วก็หายไปจากกอง)

ตัวอย่าง:

```bash
git stash pop
git stash list
```

```
(ว่างเปล่า ไม่มี output ใด ๆ ถ้านั่นคือ stash อันเดียวที่มี)
```

### ตารางเปรียบเทียบสรุป

| คำสั่ง | นำการเปลี่ยนแปลงกลับมาใส่ working directory | ลบออกจาก stash list หลัง apply สำเร็จ |
|---|---|---|
| `git stash apply` | ใช่ | **ไม่ลบ** — ยังอยู่ใน list |
| `git stash pop` | ใช่ | **ลบทันที** ถ้าไม่มี conflict |

### กรณีสำคัญ: ถ้าเกิด conflict ตอน pop จะเกิดอะไรขึ้น

นี่คือจุดที่สำคัญมาก: **ถ้า `git stash pop` เจอ conflict ระหว่าง apply Git จะไม่ลบ stash ออกจาก list** เพื่อป้องกันไม่ให้ข้อมูลหายไปโดยไม่ได้ตั้งใจ คุณจะเห็นข้อความประมาณนี้:

```
Auto-merging app.js
CONFLICT (content): Merge conflict in app.js
The stash entry is kept in case you need it again.
```

ในกรณีนี้ คุณต้อง:

1. แก้ conflict ในไฟล์ให้เรียบร้อย (เหมือนแก้ merge conflict ทั่วไป)
2. `git add` ไฟล์ที่แก้แล้ว
3. ถ้าต้องการลบ stash ออกจาก list ด้วยตัวเอง ให้รัน `git stash drop` เพิ่ม (จะสอนใน Step 115)

### เมื่อไหร่ควรใช้ `apply` แทน `pop`

ควรใช้ `git stash apply` แทน `pop` ในสถานการณ์ต่อไปนี้:

1. **ต้องการนำ stash เดียวกันไปใช้กับหลาย branch** เช่น มีโค้ด config ที่อยากลองใส่ทั้งใน `branch-a` และ `branch-b` — apply ที่ `branch-a` ก่อน ตรวจสอบ แล้วค่อย `git checkout branch-b` แล้ว apply ซ้ำอีกครั้งโดยที่ stash ยังอยู่
2. **ไม่แน่ใจว่าการ apply จะสำเร็จหรือมีปัญหาหรือไม่** — apply ไว้ก่อน ถ้าพังก็ยังมี stash สำรองอยู่ให้ลองใหม่ ถ้าสำเร็จค่อยลบด้วย `git stash drop` เอง (ควบคุมได้มากกว่า)
3. **ต้องการทำงานแบบระมัดระวังเป็นพิเศษ** เช่น กับโปรเจกต์สำคัญที่ error พลาดไม่ได้เลย

### เมื่อไหร่ควรใช้ `pop`

ใช้ `git stash pop` เมื่อ:

- มั่นใจว่าต้องการนำ stash นี้กลับมาใช้แล้วจบ ไม่ต้องใช้ซ้ำที่ไหนอีก
- ต้องการให้ stash list สะอาด ไม่มีขยะค้างอยู่หลังใช้งานเสร็จ
- นี่คือกรณีการใช้งานส่วนใหญ่ในชีวิตจริง (สลับ branch ไปทำงานด่วน แล้วกลับมาทำงานเดิมต่อ)

### การระบุ stash ที่ไม่ใช่อันล่าสุดตอน pop/apply

ทั้งสองคำสั่งรับ argument ระบุ index ได้เช่นกัน:

```bash
git stash apply stash@{2}
git stash pop stash@{1}
```

ถ้าไม่ระบุ จะหมายถึง `stash@{0}` เสมอ

### สรุป Step 114

- `git stash apply` = นำกลับมาใช้ **แต่ไม่ลบ** ออกจาก stash list
- `git stash pop` = นำกลับมาใช้ **แล้วลบทันที** ถ้าไม่มี conflict
- ถ้า `pop` เจอ conflict, Git จะไม่ลบ stash ออก เพื่อความปลอดภัยของข้อมูล
- ใช้ `apply` เมื่อต้องการใช้ stash เดิมซ้ำหลายที่ ใช้ `pop` เมื่อใช้ครั้งเดียวจบ

---

## Step 115: `git stash drop`, `git stash clear` — ลบทีละอันหรือลบทั้งหมด

### `git stash drop` — ลบ stash ทีละอัน

เมื่อคุณ apply stash เสร็จแล้ว (ไม่ได้ใช้ pop) หรือมี stash เก่าที่ไม่ต้องการแล้ว ให้ลบด้วย:

```bash
git stash drop
```

คำสั่งนี้จะลบ `stash@{0}` (อันล่าสุด) ออกจาก list โดยไม่นำกลับมาใส่ใน working directory เลย (ต่างจาก `pop` ที่ทั้ง apply และลบในคำสั่งเดียว)

ถ้าต้องการลบ stash อันที่ไม่ใช่อันล่าสุด ให้ระบุ index:

```bash
git stash drop stash@{2}
```

ตัวอย่างเต็ม:

```bash
git stash list
```

```
stash@{0}: WIP on main: a1b2c3d fix
stash@{1}: WIP on main: b2c3d4e wip
stash@{2}: On feature/x: c3d4e5f temp
```

```bash
git stash drop stash@{1}
git stash list
```

```
stash@{0}: WIP on main: a1b2c3d fix
stash@{1}: On feature/x: c3d4e5f temp
```

**สังเกตสำคัญ:** เมื่อลบ `stash@{1}` ออกไปแล้ว ดัชนีของ stash ที่เหลือจะถูก **เลื่อนขึ้นมาใหม่โดยอัตโนมัติ** (อันที่เคยเป็น `stash@{2}` จะกลายเป็น `stash@{1}`) เพราะ index ของ stash ไม่ใช่ ID ตายตัว แต่คือ **ตำแหน่งในกองซ้อน (stack position)** ณ ขณะนั้น

### ทำไมต้องระวังเรื่องนี้

ถ้าคุณเขียนสคริปต์ หรือทำงานหลายคำสั่งติดกันโดยอ้างอิง `stash@{N}` แบบ hardcode ตัวเลข ต้องระวังว่าหลังจาก `drop` หรือ `pop` ไปแล้ว ดัชนีของ stash อื่น ๆ จะเปลี่ยนไป **ควรรัน `git stash list` เช็คใหม่ทุกครั้งก่อนอ้างอิง index**

### `git stash clear` — ลบ stash ทั้งหมดในครั้งเดียว

```bash
git stash clear
```

คำสั่งนี้จะ**ลบ stash ทุกอันในกองซ้อนทิ้งทั้งหมดทันที** โดยไม่มีการถามยืนยันใด ๆ

### คำเตือนสำคัญมาก: `git stash clear` เป็นคำสั่งอันตราย

`git stash clear` เป็นหนึ่งในคำสั่งที่ทำลายข้อมูลได้ง่ายที่สุดในกลุ่มคำสั่ง stash เพราะ:

1. **ลบทีเดียวทั้งหมด** ไม่สามารถเลือกลบเฉพาะบางอันได้
2. **ไม่มีการถามยืนยัน (confirmation prompt)** ก่อนลบ
3. ถึงแม้ stash ที่ถูกลบจะยังไม่หายไปจากระบบทันที (มันจะกลายเป็น "dangling commit" ที่ยังไม่ถูก garbage collect) แต่ **การกู้คืนกลับมาทำได้ยากและไม่มี Git command สำเร็จรูปสำหรับกู้คืนโดยตรง** ต้องใช้เทคนิคขั้นสูงอย่าง `git fsck --unreachable` ร่วมกับ `git show` เพื่อค้นหา commit hash ที่หายไป แล้ว `git stash apply <hash>` กลับมาด้วยตัวเอง ซึ่งซับซ้อนและไม่รับประกันว่าจะเจอ

### วิธีกู้คืน stash ที่ถูก drop หรือ clear ไปแล้ว (ในกรณีฉุกเฉิน)

ถ้าเผลอ `drop` หรือ `clear` ไปแล้วต้องการกู้คืน ให้ลองขั้นตอนนี้ทันที (ก่อนที่ Git garbage collector จะทำงานและลบ object ทิ้งถาวร):

```bash
git fsck --unreachable | grep commit
```

คำสั่งนี้จะแสดงรายการ commit ที่ "unreachable" (ไม่มี reference ใดชี้ถึงแล้ว) ซึ่งอาจรวมถึง stash ที่ถูกลบไป ตัวอย่างผลลัพธ์:

```
unreachable commit a1b2c3d4e5f6...
unreachable commit f6e5d4c3b2a1...
```

จากนั้นตรวจสอบทีละอันด้วย:

```bash
git show a1b2c3d4e5f6
```

ถ้าเจอ commit ที่ใช่ (มีข้อความและ diff ที่ตรงกับ stash ที่หายไป) ให้กู้คืนด้วย:

```bash
git stash apply a1b2c3d4e5f6
```

> **ข้อควรรู้:** วิธีนี้ไม่รับประกันผลลัพธ์ 100% เพราะถ้า garbage collection (`git gc`) ทำงานไปแล้วก่อนที่คุณจะกู้คืน ข้อมูลอาจหายไปถาวรจริง ๆ ดังนั้นบทเรียนที่สำคัญที่สุดคือ **ควรใช้ `git stash drop` ทีละอันอย่างระมัดระวัง และหลีกเลี่ยง `git stash clear` เว้นแต่มั่นใจ 100% ว่าไม่ต้องการ stash ใด ๆ ที่เหลืออยู่เลย**

### ตารางสรุปคำสั่งลบ stash

| คำสั่ง | ผลลัพธ์ | ระดับความเสี่ยง |
|---|---|---|
| `git stash drop` | ลบ stash อันล่าสุด (หรือระบุ index) ทีละอัน | ปานกลาง — ลบทีละอันควบคุมได้ |
| `git stash drop stash@{N}` | ลบ stash อันที่ระบุเจาะจง | ปานกลาง |
| `git stash clear` | ลบ stash **ทั้งหมด** ในครั้งเดียว ไม่ถามยืนยัน | **สูงมาก** — ควรใช้ด้วยความระมัดระวังสูงสุด |

### สรุป Step 115

- `git stash drop` ลบ stash ทีละอัน (ค่าเริ่มต้นคืออันล่าสุด หรือระบุ `stash@{N}` ได้)
- ดัชนี stash จะเลื่อนใหม่โดยอัตโนมัติหลังการลบ ต้องเช็ค `git stash list` ใหม่เสมอก่อนอ้างอิง index
- `git stash clear` ลบทั้งหมดทันทีโดยไม่ถามยืนยัน — เป็นคำสั่งที่มีความเสี่ยงสูง ควรใช้อย่างระมัดระวัง
- การกู้คืน stash ที่ถูกลบไปแล้วทำได้ยากมาก ควรป้องกันไว้ก่อนดีกว่าแก้ทีหลัง

---

## Step 116: ตั้งชื่อ stash ให้จำง่ายด้วย `git stash push -m "message"`

### ปัญหาของการ stash แบบไม่ตั้งชื่อ

ถ้าคุณ stash บ่อย ๆ โดยไม่ตั้งชื่อ `git stash list` จะแสดงข้อความ generic ที่ไม่มีประโยชน์แบบนี้:

```
stash@{0}: WIP on main: a1b2c3d initial commit
stash@{1}: WIP on main: a1b2c3d initial commit
stash@{2}: WIP on main: a1b2c3d initial commit
```

สังเกตว่าทั้ง 3 อันแสดงข้อความเหมือนกันทุกตัวอักษร (เพราะ commit ล่าสุดของ branch ยังไม่เปลี่ยน) ทำให้ **แยกไม่ออกเลยว่าอันไหนคือ stash ของงานอะไร** ต้องเปิดดูด้วย `git stash show -p` ทีละอันเพื่อเดาว่าอันไหนคืออันที่ต้องการ ซึ่งเสียเวลามาก

### วิธีแก้: ตั้งชื่อ (message) ให้ stash ด้วยตัวเอง

ใช้ flag `-m` (หรือ `--message`) ตามหลัง `git stash push`:

```bash
git stash push -m "งานฟอร์ม login ยังไม่เสร็จ รอเพิ่ม validation"
```

ผลลัพธ์ใน `git stash list` จะเปลี่ยนรูปแบบจาก `WIP on ...` เป็น `On ...` พร้อมข้อความที่คุณตั้งเอง:

```
stash@{0}: On main: งานฟอร์ม login ยังไม่เสร็จ รอเพิ่ม validation
```

### เปรียบเทียบก่อน-หลังตั้งชื่อ

| แบบไม่ตั้งชื่อ (`git stash`) | แบบตั้งชื่อ (`git stash push -m "..."`) |
|---|---|
| `WIP on main: a1b2c3d initial commit` | `On main: งานฟอร์ม login ยังไม่เสร็จ` |
| ต้องเปิด diff ดูทีละอันเพื่อจำว่าคืองานอะไร | อ่านชื่อแล้วรู้ทันทีว่าคืองานอะไร |

### ตัวอย่างการทำงานจริงกับหลาย stash ที่ตั้งชื่อไว้อย่างเป็นระบบ

```bash
git stash push -m "เพิ่มฟอร์ม login - รอ validation"
# ... สลับไปทำงานอื่น แก้ไขไฟล์ต่อ ...
git stash push -m "แก้ CSS responsive มือถือ - ยังไม่ผ่าน test"
# ... สลับไปทำงานอื่นอีก ...
git stash push -m "ทดลอง refactor API client - ยัง break tests อยู่"

git stash list
```

```
stash@{0}: On main: ทดลอง refactor API client - ยัง break tests อยู่
stash@{1}: On main: แก้ CSS responsive มือถือ - ยังไม่ผ่าน test
stash@{2}: On main: เพิ่มฟอร์ม login - รอ validation
```

ตอนนี้คุณสามารถเลือกดึงกลับมาทำต่อได้อย่างแม่นยำโดยไม่ต้องเดา:

```bash
git stash pop stash@{2}
```

### ข้อควรระวัง: ลำดับของ argument สำคัญมาก

`-m` ต้องมาก่อนตัวข้อความเสมอ และ **`-m` ต้องอยู่หลัง `push`** ไม่สามารถใช้กับ `git stash` เฉย ๆ แบบไม่มี `push` ได้โดยตรงในทุกกรณี (แม้ Git บางเวอร์ชันจะยอมให้ `git stash -m "..."` ทำงานได้เหมือนกันเพราะตีความเป็น `push` แต่การเขียนแบบเต็ม `git stash push -m "..."` ชัดเจนกว่าและแนะนำให้ใช้เพื่อความเข้าใจตรงกันในทีม)

```bash
# แนะนำ (ชัดเจนที่สุด)
git stash push -m "ข้อความอธิบาย"

# ใช้งานได้เช่นกันในเวอร์ชันใหม่ แต่แนะนำให้เขียนแบบเต็มกว่า
git stash -m "ข้อความอธิบาย"
```

### คำสั่งเก่าที่ควรรู้จักแต่ไม่แนะนำให้ใช้: `git stash save`

ก่อนจะมี `git stash push -m` คนใช้ `git stash save "message"` แทน:

```bash
git stash save "ข้อความอธิบาย (วิธีเก่า)"
```

คำสั่งนี้ **ยังใช้งานได้อยู่** เพื่อความเข้ากันได้ย้อนหลัง แต่ Git official documentation ระบุว่าเป็น **deprecated** (เลิกแนะนำให้ใช้) แล้ว เพราะ `push` รองรับความสามารถเพิ่มเติมที่ `save` ทำไม่ได้ เช่น การเลือก stash เฉพาะบางไฟล์ (จะสอนใน Step 117) ดังนั้นควรฝึกใช้ `git stash push -m` ให้เป็นนิสัยตั้งแต่ตอนนี้

### แนวปฏิบัติที่ดีในการตั้งชื่อ stash

1. **ระบุว่าเป็นงานอะไร** เช่น "เพิ่ม feature X"
2. **ระบุสถานะความคืบหน้า** เช่น "ยังไม่เสร็จ", "รอ test", "ยัง break อยู่"
3. **หลีกเลี่ยงชื่อกำกวมเช่น "temp" หรือ "test"** ที่ไม่บอกอะไรเลย
4. ถ้าเป็นทีม อาจใส่วันที่หรือชื่อ ticket/issue ด้วย เช่น "JIRA-1234: กำลังแก้ payment gateway"

### สรุป Step 116

- `git stash push -m "message"` ให้คุณตั้งชื่อ stash เพื่อให้จำง่ายเวลาดูใน `git stash list`
- แบบไม่ตั้งชื่อจะขึ้น `WIP on <branch>: ...` ซึ่งแยกความแตกต่างระหว่างหลาย stash ไม่ได้
- `git stash save "message"` คือวิธีเก่าที่ยังใช้ได้แต่ deprecated แล้ว ควรใช้ `push -m` แทน
- การตั้งชื่อที่ดีช่วยประหยัดเวลาอย่างมากเมื่อมี stash ค้างอยู่หลายอันพร้อมกัน

---

## Step 117: Stash เฉพาะบางไฟล์ด้วย `git stash push -- <file>`

### ปัญหา: บางครั้งไม่อยาก stash ทุกไฟล์

สมมติคุณกำลังแก้ไข 3 ไฟล์พร้อมกัน:

```bash
git status
```

```
Changes not staged for commit:
        modified:   app.js
        modified:   style.css
        modified:   config.json
```

แต่คุณต้องการ stash **เฉพาะ `app.js`** เท่านั้น เพราะ `style.css` และ `config.json` คุณแก้เสร็จแล้วและพร้อม commit ทันที ถ้าใช้ `git stash` เฉย ๆ จะ stash ทั้ง 3 ไฟล์ไปหมด ซึ่งไม่ใช่สิ่งที่ต้องการ

### วิธีแก้: ระบุไฟล์ต่อท้าย `--`

```bash
git stash push -- app.js
```

หรือใส่ message ควบคู่ไปด้วย:

```bash
git stash push -m "พัก app.js ไว้ก่อน" -- app.js
```

หลังจากรันคำสั่งนี้ ตรวจสอบ `git status`:

```bash
git status
```

```
Changes not staged for commit:
        modified:   style.css
        modified:   config.json
```

จะเห็นว่า `app.js` ถูกคืนกลับไปเป็นสถานะเดิม (เหมือน commit ล่าสุด) แล้ว ในขณะที่ `style.css` และ `config.json` ยังคงมีการเปลี่ยนแปลงค้างอยู่เหมือนเดิม ไม่ถูกแตะต้อง

### ทำไมต้องมี `--` คั่นก่อนชื่อไฟล์

เครื่องหมาย `--` เป็นรูปแบบมาตรฐานของ Git (และเครื่องมือ command-line จำนวนมาก) ที่บอกว่า **"หลังจากนี้คือ path ของไฟล์ ไม่ใช่ flag หรือ option อีกต่อไป"** สิ่งนี้สำคัญมากในกรณีที่ชื่อไฟล์ของคุณบังเอิญคล้ายกับชื่อ flag เช่น ถ้ามีไฟล์ชื่อ `-m` (แม้จะแปลกแต่เป็นไปได้) การใส่ `--` จะช่วยป้องกันความกำกวมได้

ในทางปฏิบัติ Git มักจะฉลาดพอที่จะเข้าใจได้แม้ไม่ใส่ `--` ในกรณีทั่วไป:

```bash
git stash push app.js
```

แต่การใส่ `--` เป็นธรรมเนียมปฏิบัติที่ดี (best practice) ที่แนะนำให้ทำเสมอเพื่อความชัดเจนและปลอดภัย โดยเฉพาะเมื่อเขียนสคริปต์อัตโนมัติ

### Stash หลายไฟล์พร้อมกัน (แต่ไม่ใช่ทั้งหมด)

ระบุไฟล์หลายไฟล์คั่นด้วยช่องว่างได้:

```bash
git stash push -m "พักสองไฟล์นี้ไว้ก่อน" -- app.js config.json
```

### Stash เฉพาะโฟลเดอร์

สามารถระบุ path เป็นโฟลเดอร์ได้เช่นกัน ซึ่งจะ stash ทุกไฟล์ที่เปลี่ยนแปลงภายในโฟลเดอร์นั้น:

```bash
git stash push -- src/components/
```

### ใช้ pattern แบบ glob

Git รองรับการใช้ pattern บางรูปแบบร่วมกับ shell ของระบบ เช่น:

```bash
git stash push -- "*.css"
```

คำสั่งนี้จะ stash เฉพาะไฟล์ที่ลงท้ายด้วย `.css` เท่านั้น (ใส่เครื่องหมายคำพูดครอบไว้เพื่อป้องกันไม่ให้ shell ขยาย wildcard ก่อนส่งให้ Git)

### ความแตกต่างจาก `git stash push` แบบไม่ระบุไฟล์

| คำสั่ง | ผลลัพธ์ |
|---|---|
| `git stash push` | stash **ทุกไฟล์** ที่มีการเปลี่ยนแปลง (tracked files ทั้งหมด) |
| `git stash push -- app.js` | stash **เฉพาะ `app.js`** เท่านั้น ไฟล์อื่นไม่ถูกแตะต้อง |
| `git stash push -- src/` | stash เฉพาะไฟล์ที่เปลี่ยนแปลงภายในโฟลเดอร์ `src/` |

### กรณีใช้งานจริงที่พบบ่อย

**สถานการณ์:** คุณกำลังทำงาน 2 features คู่ขนานในไฟล์คนละชุด แต่ไม่ได้แยก branch (อาจเพราะทดลองเร็ว ๆ) แล้วต้องการ commit เฉพาะ feature ที่เสร็จก่อน โดยไม่ให้ feature ที่ยังไม่เสร็จติดไปด้วย:

```bash
# มีการแก้ไขใน feature-a.js (เสร็จแล้ว) และ feature-b.js (ยังไม่เสร็จ)
git stash push -m "feature b ยังไม่เสร็จ" -- feature-b.js

# ตอนนี้ working directory เหลือแค่ feature-a.js ที่เสร็จแล้ว
git add feature-a.js
git commit -m "เพิ่ม feature A เสร็จสมบูรณ์"

# ดึง feature b กลับมาทำต่อ
git stash pop
```

วิธีนี้ช่วยให้แต่ละ commit มีความหมายชัดเจน (atomic commit) โดยไม่ปนกันระหว่างงานที่เสร็จแล้วกับงานที่ยังไม่เสร็จ

### สรุป Step 117

- `git stash push -- <file>` ให้ stash เฉพาะไฟล์ที่ระบุ ไม่กระทบไฟล์อื่นที่กำลังแก้อยู่
- `--` คือเครื่องหมายคั่นมาตรฐานที่บอกว่าหลังจากนี้คือ path ของไฟล์
- รองรับการระบุหลายไฟล์ โฟลเดอร์ หรือ pattern แบบ glob
- มีประโยชน์มากเมื่อต้องการแยก commit ที่ atomic (มีความหมายเดียวชัดเจน) จากงานหลายชิ้นที่ปนกันอยู่ใน working directory

---

## Step 118: `git stash -u` / `--include-untracked` — รวมไฟล์ที่ยังไม่ถูก track เข้าไปด้วย

### ทบทวน: Stash ปกติไม่เก็บ untracked files

อย่างที่กล่าวไว้ใน Step 111 โดยค่าเริ่มต้น `git stash` จะ**ไม่**เก็บไฟล์ที่ยังไม่เคยถูก `git add` เลยสักครั้ง (untracked files) มาดูตัวอย่างให้เห็นภาพชัดเจน:

```bash
echo "console.log('เก่า');" > app.js
git add app.js
git commit -m "initial"

# แก้ไขไฟล์เดิม (tracked)
echo "console.log('แก้ไขแล้ว');" >> app.js

# สร้างไฟล์ใหม่ที่ยังไม่เคย add เลย (untracked)
echo "temp data" > new-feature.js

git status
```

```
On branch main
Changes not staged for commit:
        modified:   app.js

Untracked files:
        new-feature.js
```

ลองสั่ง `git stash` แบบธรรมดา:

```bash
git stash
git status
```

```
On branch main
Untracked files:
        new-feature.js
```

**สังเกต:** `app.js` ถูก stash ไปแล้ว (working directory สะอาดสำหรับไฟล์นี้) แต่ **`new-feature.js` ยังคงค้างอยู่** เพราะมันเป็น untracked file ซึ่ง Git มองว่า "ยังไม่เกี่ยวข้องกับ repository" เลย จึงไม่ได้อยู่ในขอบเขตของ stash แบบปกติ

### ทำไม Git ถึงออกแบบมาแบบนี้

เหตุผลเชิงแนวคิดคือ Git มองว่า stash คือการเก็บ **"สถานะของไฟล์ที่ Git กำลังติดตามอยู่แล้ว"** (tracked files) เป็นหลัก ส่วนไฟล์ใหม่ที่ยังไม่เคย add นั้น อาจเป็นไฟล์ที่ไม่เกี่ยวข้องกับ Git เลยก็ได้ เช่น ไฟล์ config ส่วนตัว, ไฟล์ log, หรือไฟล์ที่กำลังทดลองอยู่ Git จึงไม่อยากไปยุ่งกับมันโดยไม่ได้รับอนุญาตชัดเจน

### วิธีแก้: ใช้ `-u` หรือ `--include-untracked`

```bash
git stash -u
```

หรือแบบเต็ม:

```bash
git stash --include-untracked
```

ทั้งสองแบบทำงานเหมือนกันทุกประการ (`-u` เป็นตัวย่อของ `--include-untracked`)

ลองทำซ้ำตัวอย่างเดิม แต่ครั้งนี้ใช้ `-u`:

```bash
echo "console.log('แก้ไขแล้ว');" >> app.js
echo "temp data" > new-feature.js

git stash -u
git status
```

```
On branch main
nothing to commit, working tree clean
```

คราวนี้ **ทั้ง `app.js` และ `new-feature.js` ถูกพักไว้หมด** working directory สะอาดสนิท และเมื่อ pop กลับมา ทั้งสองไฟล์จะกลับมาครบถ้วนเหมือนเดิม รวมถึง `new-feature.js` ที่จะกลับมาเป็นสถานะ untracked เหมือนก่อน stash ด้วย

### ใช้ร่วมกับ `push` และ `-m` ได้ตามปกติ

```bash
git stash push -u -m "รวมไฟล์ใหม่ที่ยังไม่ track ด้วย"
```

### ระดับที่ลึกกว่านั้น: `-a` หรือ `--all` — รวมไฟล์ที่ถูก ignore ด้วย

มีอีกระดับหนึ่งที่ลึกกว่า `-u` คือ `--all` (หรือ `-a`) ซึ่งจะรวม **ไฟล์ที่ถูก `.gitignore` กันไว้ (ignored files)** เข้าไปด้วย:

```bash
git stash --all
```

**ควรระวังอย่างมากกับ flag นี้** เพราะไฟล์ที่ถูก ignore มักจะเป็นไฟล์ที่ **ตั้งใจ** ไม่ให้ Git ยุ่งเกี่ยวด้วย เช่น:

- โฟลเดอร์ `node_modules/`
- ไฟล์ build output เช่น `dist/`, `build/`
- ไฟล์ environment variable ส่วนตัว เช่น `.env`
- ไฟล์ cache ต่าง ๆ

การใช้ `--all` จะดึงไฟล์เหล่านี้เข้ามาอยู่ใน stash ด้วย ซึ่งอาจทำให้ stash มีขนาดใหญ่มาก (เช่น `node_modules/` อาจมีหลายพันไฟล์) และไม่ค่อยมีประโยชน์ในทางปฏิบัติ **โดยทั่วไปไม่แนะนำให้ใช้ `--all` เว้นแต่มีเหตุผลเฉพาะเจาะจงจริง ๆ**

### ตารางเปรียบเทียบระดับของการรวมไฟล์

| คำสั่ง | Tracked files ที่เปลี่ยนแปลง | Untracked files | Ignored files |
|---|---|---|---|
| `git stash` (ค่าเริ่มต้น) | รวม | **ไม่รวม** | **ไม่รวม** |
| `git stash -u` / `--include-untracked` | รวม | **รวม** | ไม่รวม |
| `git stash -a` / `--all` | รวม | รวม | **รวม** |

### กรณีใช้งานจริงที่ `-u` มีประโยชน์มาก

**สถานการณ์:** คุณกำลังสร้างฟีเจอร์ใหม่ที่ต้องเพิ่มไฟล์ใหม่หลายไฟล์ (เช่น component ใหม่ในโปรเจกต์ React) พร้อมกับแก้ไฟล์เดิมด้วย แล้วต้องสลับ branch ด่วน:

```bash
# สร้างไฟล์ component ใหม่ (ยังไม่ add)
touch src/components/UserProfile.jsx
touch src/components/UserProfile.css

# แก้ไฟล์เดิมที่ import component ใหม่นี้
# (แก้ src/App.jsx)

git status
```

```
Changes not staged for commit:
        modified:   src/App.jsx

Untracked files:
        src/components/UserProfile.jsx
        src/components/UserProfile.css
```

ถ้าใช้ `git stash` เฉย ๆ ไฟล์ `UserProfile.jsx` และ `UserProfile.css` จะยังค้างอยู่ ทำให้ working directory ไม่สะอาดจริง และอาจสร้างความสับสนเมื่อสลับไป branch อื่น (ไฟล์ใหม่เหล่านี้จะติดตามไปด้วยเพราะเป็น untracked file ที่ไม่ผูกกับ branch ใด ๆ)

การใช้ `git stash -u` แก้ปัญหานี้ได้สมบูรณ์ ทำให้ working directory สะอาดจริง 100% ก่อนสลับ branch

### สรุป Step 118

- ค่าเริ่มต้นของ `git stash` ไม่รวม untracked files (ไฟล์ใหม่ที่ยังไม่เคย `git add`)
- `git stash -u` หรือ `--include-untracked` รวม untracked files เข้าไปในการ stash ด้วย
- `git stash -a` หรือ `--all` รวมแม้กระทั่งไฟล์ที่ถูก `.gitignore` — ควรใช้อย่างระมัดระวัง เพราะมักไม่จำเป็นและอาจทำให้ stash ใหญ่โดยไม่จำเป็น
- ควรใช้ `-u` เป็นนิสัยเมื่อรู้ตัวว่ากำลังสร้างไฟล์ใหม่ควบคู่กับแก้ไฟล์เดิม แล้วต้องสลับงานกะทันหัน

---

## Step 119: สร้าง branch จาก stash โดยตรงด้วย `git stash branch` — แก้ปัญหา conflict ตอน apply

### ปัญหาที่เกิดขึ้นบ่อย: Apply stash แล้วเจอ conflict

สถานการณ์นี้เกิดขึ้นได้บ่อยมาก: คุณ stash งานไว้บน branch `main` ที่จุดหนึ่ง จากนั้นทำงานต่อบน `main` เอง (หรือคนอื่นในทีม push การเปลี่ยนแปลงเข้ามา แล้วคุณ pull ลงมา) ทำให้ `main` เดินหน้าต่อไปไกลจากจุดที่คุณ stash ไว้มาก เมื่อกลับมา `git stash pop` จึงเกิด conflict:

```bash
git stash pop
```

```
Auto-merging app.js
CONFLICT (content): Merge conflict in app.js
The stash entry is kept in case you need it again.
```

ปัญหานี้เกิดเพราะ **โค้ดใน `app.js` บน `main` ปัจจุบันเปลี่ยนไปมากจนไม่ตรงกับ base ที่ stash นั้นถูกสร้างขึ้นมา** การแก้ conflict ตรงนี้อาจซับซ้อนโดยเฉพาะถ้ามีการเปลี่ยนแปลงเยอะ

### วิธีแก้ที่สง่างามกว่า: `git stash branch`

Git มีคำสั่งพิเศษที่ออกแบบมาสำหรับแก้ปัญหานี้โดยเฉพาะ:

```bash
git stash branch <ชื่อ-branch-ใหม่>
```

ตัวอย่าง:

```bash
git stash branch feature/continue-user-profile
```

### `git stash branch` ทำอะไรบ้าง (ตามลำดับ)

เมื่อรันคำสั่งนี้ Git จะทำงานตามลำดับดังนี้โดยอัตโนมัติ:

1. **สร้าง branch ใหม่** จาก commit ที่เป็น base ของ stash นั้น (คือ commit ที่ `HEAD` ชี้อยู่ ณ ตอนที่สร้าง stash นั้นขึ้นมา ไม่ใช่ commit ปัจจุบันของ branch ที่คุณอยู่ตอนนี้)
2. **สลับ (checkout) ไปยัง branch ใหม่นั้นทันที**
3. **Apply stash** ลงบน branch ใหม่นี้ (เพราะ branch ใหม่มี base ตรงกับตอนสร้าง stash เป๊ะ จึงแทบไม่มีโอกาสเกิด conflict)
4. **ถ้า apply สำเร็จ** จะ `drop` stash นั้นออกจาก stash list โดยอัตโนมัติ (คล้ายพฤติกรรมของ `pop`)

### ทำไมวิธีนี้ถึงหลีกเลี่ยง conflict ได้

หัวใจสำคัญคือข้อ 1: branch ใหม่ถูกสร้างขึ้นจาก **จุดเริ่มต้นเดียวกันกับตอนที่สร้าง stash** ไม่ใช่จุดปัจจุบันของ `main` ที่เดินหน้าไปไกลแล้ว ดังนั้นเมื่อ apply stash ลงบน branch ใหม่นี้ โค้ดจะตรงกันพอดี (เพราะไม่มีอะไรเปลี่ยนแปลงระหว่างทาง) จึงแทบไม่มีทางเกิด conflict เลย

### ตัวอย่างการทำงานเต็มรูปแบบ

```bash
# อยู่บน main, ทำงานค้างไว้
git switch main
echo "งานที่ทำค้าง" >> app.js
git stash push -m "งาน user profile ยังไม่เสร็จ"

# เวลาผ่านไป มีคน push งานใหม่เข้า main หลายรอบ
git pull origin main
# ตอนนี้ app.js บน main เปลี่ยนไปมากแล้ว

# ถ้าลอง pop ตรง ๆ จะเจอ conflict แน่นอน
# แต่แทนที่จะ pop ตรง ๆ ใช้วิธีนี้แทน:
git stash branch feature/user-profile-continue
```

ผลลัพธ์:

```
Switched to a new branch 'feature/user-profile-continue'
On branch feature/user-profile-continue
Changes not staged for commit:
        modified:   app.js

Dropped refs/stash@{0} (a1b2c3d4e5f6...)
```

ตอนนี้คุณอยู่บน branch ใหม่ `feature/user-profile-continue` ที่มีงานเดิมของคุณกลับมาครบถ้วน ไม่มี conflict ใด ๆ พร้อมทำงานต่อและค่อย merge เข้า `main` ทีหลังตามกระบวนการปกติ (Pull Request, code review ฯลฯ ซึ่งจะสอนละเอียดใน Part ถัดไปของหลักสูตรนี้)

### ข้อควรรู้เพิ่มเติม: ระบุ stash ที่ไม่ใช่อันล่าสุดได้

```bash
git stash branch <ชื่อ-branch> stash@{2}
```

ถ้าไม่ระบุ index จะใช้ `stash@{0}` (อันล่าสุด) โดยอัตโนมัติเหมือนคำสั่ง stash อื่น ๆ

### เมื่อไหร่ควรใช้ `git stash branch` แทนการ pop ตรง ๆ

| สถานการณ์ | คำแนะนำ |
|---|---|
| Stash ไว้ไม่นาน branch ปัจจุบันแทบไม่เปลี่ยน | `git stash pop` ตรง ๆ ก็เพียงพอ |
| Stash ไว้นานแล้ว หรือรู้ว่า branch เปลี่ยนไปเยอะ | ใช้ `git stash branch` เพื่อหลีกเลี่ยง conflict ตั้งแต่ต้น |
| ลอง `pop` แล้วเจอ conflict ซับซ้อนเกินจะแก้สะดวก | ยกเลิก conflict นั้น (`git checkout -- .` หรือ `git merge --abort` ตามสถานการณ์) แล้วลอง `git stash branch` แทน |
| ต้องการให้งานที่ stash ไว้กลายเป็นจุดเริ่มต้นของ feature branch ใหม่ตั้งแต่แรก | ใช้ `git stash branch` ได้ตรงตามจุดประสงค์เลย |

### สรุป Step 119

- `git stash branch <name>` สร้าง branch ใหม่จากจุดฐาน (base) ของ stash นั้นโดยตรง แล้ว apply stash ลงไปให้อัตโนมัติ
- วิธีนี้หลีกเลี่ยง conflict ได้อย่างมีประสิทธิภาพ เพราะ branch ใหม่มี base ตรงกับตอนสร้าง stash พอดี
- ถ้า apply สำเร็จ stash เดิมจะถูก drop ออกจาก list ให้อัตโนมัติ
- เหมาะมากสำหรับกรณีที่ stash ไว้นาน หรือรู้ล่วงหน้าว่า branch ปัจจุบันเปลี่ยนแปลงไปมากแล้ว

---

## Step 120: แบบฝึกหัด — จำลองสถานการณ์ต้องหยุดงานกะทันหันเพื่อไปแก้บั๊กด่วน แล้วกลับมาทำงานเดิมต่อ

ถึงเวลาลงมือปฏิบัติจริง แบบฝึกหัดนี้จำลองสถานการณ์ที่เกิดขึ้นบ่อยที่สุดในการทำงานจริง เพื่อให้คุณฝึกใช้คำสั่งทั้งหมดที่เรียนมาใน Part นี้อย่างครบวงจร

### เตรียมสภาพแวดล้อม

```bash
mkdir ~/git-course/part-12-practice
cd ~/git-course/part-12-practice
git init

echo "function calculateTotal(items) {" > shop.js
echo "  return items.length;" >> shop.js
echo "}" >> shop.js
git add shop.js
git commit -m "initial: ฟังก์ชัน calculateTotal เบื้องต้น"
```

### สถานการณ์: คุณกำลังทำฟีเจอร์คำนวณราคารวมสินค้า

```bash
git switch -c feature/shopping-cart
```

แก้ไข `shop.js` ให้เพิ่มลอจิกการคำนวณราคา (ยังไม่เสร็จ):

```bash
cat > shop.js << 'EOF'
function calculateTotal(items) {
  let total = 0;
  for (const item of items) {
    total += item.price * item.qty;
    // TODO: ยังต้องคิดเรื่องส่วนลดด้วย ยังทำไม่เสร็จ
  }
  return total;
EOF
```

สร้างไฟล์ใหม่ที่ยังไม่ add ด้วย (จำลองไฟล์ทดสอบที่กำลังร่างอยู่):

```bash
echo "// ทดสอบ calculateTotal - ยังเขียนไม่เสร็จ" > shop.test.js
```

ตรวจสอบสถานะปัจจุบัน:

```bash
git status
```

```
On branch feature/shopping-cart
Changes not staged for commit:
        modified:   shop.js

Untracked files:
        shop.test.js
```

### เหตุการณ์ด่วน: หัวหน้าทีมสั่งให้ไปแก้บั๊กบน `main` เดี๋ยวนี้

**ขั้นตอนที่ 1: พักงานปัจจุบันทั้งหมด รวมไฟล์ใหม่ที่ยังไม่ track ด้วย**

```bash
git stash push -u -m "shopping cart: กำลังคิดเรื่องส่วนลด ยังไม่เสร็จ syntax error อยู่"
```

```bash
git status
```

```
On branch feature/shopping-cart
nothing to commit, working tree clean
```

working directory สะอาดแล้ว พร้อมสลับ branch

**ขั้นตอนที่ 2: สลับไป `main` เพื่อแก้บั๊กด่วน**

```bash
git switch main
```

สมมติเจอบั๊ก: ฟังก์ชัน `calculateTotal` เดิมคำนวณผิด (คืนค่าจำนวนชิ้นแทนที่จะเป็นราคารวม) แก้ด่วน:

```bash
cat > shop.js << 'EOF'
function calculateTotal(items) {
  // hotfix: แก้บั๊กคำนวณผิด (เดิมคืนค่า items.length ซึ่งผิด)
  let total = 0;
  for (const item of items) {
    total += item.price;
  }
  return total;
}
EOF

git add shop.js
git commit -m "hotfix: แก้บั๊กคำนวณราคารวมผิดพลาดบน production"
```

**ขั้นตอนที่ 3: (จำลอง) push hotfix ขึ้น production**

```bash
# ในสถานการณ์จริงจะมีคำสั่ง git push origin main ที่นี่
# และอาจมีการ deploy ตามมา
echo "hotfix deployed"
```

### กลับมาทำงานเดิมต่อ

**ขั้นตอนที่ 4: สลับกลับไปที่ branch เดิม**

```bash
git switch feature/shopping-cart
```

**ขั้นตอนที่ 5: ตรวจสอบ stash ก่อนดึงกลับมา (แนวปฏิบัติที่ดี)**

```bash
git stash list
```

```
stash@{0}: On feature/shopping-cart: shopping cart: กำลังคิดเรื่องส่วนลด ยังไม่เสร็จ syntax error อยู่
```

ดูรายละเอียดก่อนตัดสินใจ:

```bash
git stash show -p stash@{0}
```

**ขั้นตอนที่ 6: ดึงงานกลับมาทำต่อ**

เนื่องจากมั่นใจว่าจะใช้ stash นี้ครั้งเดียวจบ ใช้ `pop`:

```bash
git stash pop
```

```
On branch feature/shopping-cart
Changes not staged for commit:
        modified:   shop.js

Untracked files:
        shop.test.js

Dropped refs/stash@{0} (a1b2c3d...)
```

ตรวจสอบว่า `shop.test.js` กลับมาแล้ว และ `shop.js` มีการเปลี่ยนแปลงที่ค้างไว้กลับมาครบ:

```bash
cat shop.js
git status
```

**ขั้นตอนที่ 7: ทำงานต่อจนเสร็จ**

```bash
cat > shop.js << 'EOF'
function calculateTotal(items) {
  let total = 0;
  for (const item of items) {
    let price = item.price * item.qty;
    if (item.discount) {
      price = price * (1 - item.discount);
    }
    total += price;
  }
  return total;
}
EOF

git add shop.js shop.test.js
git commit -m "เพิ่มลอจิกคำนวณส่วนลดใน shopping cart เสร็จสมบูรณ์"
```

### แบบฝึกหัดเพิ่มเติม: จำลอง conflict แล้วแก้ด้วย `git stash branch`

ลองสถานการณ์ที่ซับซ้อนขึ้นอีกขั้น:

```bash
git switch main
echo "// ปรับปรุงเพิ่มเติมบน main" >> shop.js
git add shop.js
git commit -m "ปรับปรุง shop.js เพิ่มเติมบน main"

git switch -c experiment/discount-v2
echo "// ทดลอง logic ส่วนลดแบบใหม่ ยังไม่เสร็จ" >> shop.js
git stash push -m "ทดลอง discount v2"

git switch main
echo "// แก้ไข main เพิ่มอีกรอบจนไฟล์เปลี่ยนไปมาก" >> shop.js
git add shop.js
git commit -m "แก้ไข main เพิ่มเติมอีกรอบ"

git switch experiment/discount-v2
```

ตอนนี้ลองสังเกตว่าถ้า `pop` ตรง ๆ อาจเจอ conflict เพราะ base เปลี่ยนไปมากแล้ว แทนที่จะเสี่ยง ให้ใช้ `git stash branch` แทน:

```bash
git stash branch experiment/discount-v2-continue
git status
git log --oneline --all --graph
```

สังเกตว่า branch ใหม่ถูกสร้างขึ้นจาก base ที่ตรงกับตอน stash พอดี และ stash ถูก apply เข้ามาโดยไม่มี conflict เลย

### Checklist ตรวจสอบความเข้าใจ

หลังทำแบบฝึกหัดนี้ ลองถามตัวเองว่าตอบคำถามเหล่านี้ได้ครบหรือไม่:

- อธิบายได้ว่าทำไม `git stash -u` ถึงจำเป็นในสถานการณ์ที่มีทั้งไฟล์แก้ไขและไฟล์ใหม่ที่ยังไม่ track
- อธิบายความแตกต่างระหว่าง `pop` กับ `apply` ได้ พร้อมยกตัวอย่างสถานการณ์ที่ควรใช้แต่ละแบบ
- รู้ว่าทำไมควรตรวจสอบ `git stash show -p` ก่อน `pop` หรือ `apply` เสมอเมื่อ stash ค้างไว้นาน
- เข้าใจว่าทำไม `git stash branch` ถึงช่วยลดโอกาสเกิด conflict ได้มากกว่าการ `pop` ตรง ๆ

---

## สรุป Part 12

ใน Part นี้เราได้เรียนรู้เครื่องมือสำคัญที่ช่วยแก้ปัญหา **"ต้องสลับงานกะทันหันแต่ยังทำงานปัจจุบันไม่เสร็จ"** ซึ่งเป็นสถานการณ์ที่เกิดขึ้นแทบทุกวันในการทำงานจริง สรุปสิ่งที่เราได้เรียนรู้:

1. **Stash คืออะไร** — ลิ้นชักพักงานชั่วคราวที่เก็บการเปลี่ยนแปลงไว้โดยไม่ต้อง commit และคืน working directory ให้สะอาดทันที
2. **`git stash` / `git stash push`** — คำสั่งพื้นฐานในการเก็บงานไปพัก เบื้องหลังคือการสร้าง commit พิเศษที่ผูกกับ `refs/stash`
3. **`git stash list`, `show`, `show -p`** — เครื่องมือตรวจสอบรายการและรายละเอียดของสิ่งที่ถูกพักไว้ ก่อนตัดสินใจนำกลับมาใช้
4. **`pop` vs `apply`** — `pop` นำกลับมาใช้แล้วลบออกจาก list ทันที ส่วน `apply` นำกลับมาใช้แต่ยังเก็บสำเนาไว้ เลือกใช้ให้เหมาะกับสถานการณ์
5. **`drop` และ `clear`** — ลบ stash ทีละอันอย่างควบคุมได้ด้วย `drop` หรือระวังให้มากกับ `clear` ที่ลบทั้งหมดโดยไม่ถามยืนยัน
6. **`push -m "message"`** — ตั้งชื่อ stash ให้จำง่าย แก้ปัญหา stash หลายอันที่แยกไม่ออกว่าอันไหนคืองานอะไร
7. **`push -- <file>`** — เลือก stash เฉพาะบางไฟล์ ช่วยแยกงานที่เสร็จแล้วออกจากงานที่ยังไม่เสร็จเพื่อทำ atomic commit
8. **`-u` / `--include-untracked`** — รวมไฟล์ใหม่ที่ยังไม่ track เข้าไปด้วย ทำให้ working directory สะอาดจริง 100% ก่อนสลับ branch
9. **`git stash branch`** — สร้าง branch ใหม่จาก base ของ stash โดยตรง เป็นวิธีที่สง่างามในการหลีกเลี่ยง conflict เมื่อ branch ปัจจุบันเปลี่ยนแปลงไปมากแล้ว
10. **ฝึกปฏิบัติจริง** — จำลองสถานการณ์ครบวงจรตั้งแต่ stash งาน ไปแก้บั๊กด่วน จนกลับมาทำงานเดิมต่อได้อย่างราบรื่น

### Checklist ก่อนไป Part 13

ก่อนไปต่อ ให้ตรวจสอบว่าคุณ:

- [ ] เข้าใจว่า stash แก้ปัญหาอะไร และทำไมการ commit งานที่ยังไม่เสร็จไม่ใช่ทางออกที่ดี
- [ ] ใช้ `git stash`, `git stash list`, `git stash show -p` ได้คล่องแคล่ว
- [ ] แยกความแตกต่างระหว่าง `git stash pop` กับ `git stash apply` ได้ชัดเจน และรู้ว่าเมื่อไหร่ควรใช้แบบไหน
- [ ] ใช้ `git stash drop` และเข้าใจความเสี่ยงของ `git stash clear` ได้
- [ ] ตั้งชื่อ stash ด้วย `git stash push -m` เป็นนิสัยทุกครั้งที่ stash งานสำคัญ
- [ ] เลือก stash เฉพาะบางไฟล์ด้วย `git stash push -- <file>` ได้
- [ ] เข้าใจและใช้ `git stash -u` เมื่อมีไฟล์ใหม่ที่ยังไม่ track ปนอยู่กับไฟล์ที่แก้ไข
- [ ] เข้าใจกลไกของ `git stash branch` และรู้ว่ามันช่วยหลีกเลี่ยง conflict ได้อย่างไร
- [ ] ทำแบบฝึกหัดจำลองสถานการณ์งานด่วนใน Step 120 จนจบและได้ผลลัพธ์ตามที่คาดไว้

**ต่อไป:** [Part 13: Git Aliases และการปรับแต่ง Workflow ส่วนตัว](./part-013-git-aliases-workflow.md)
