# Part 57: Git Plumbing Commands: hash-object, cat-file, ls-tree

> **Step ในหลักสูตรนี้:** Step 561–570
> **เฟส:** 6 — Git ขั้นสูง / Internals
> **เป้าหมายของ Part นี้:** เข้าใจความแตกต่างระหว่าง Plumbing กับ Porcelain commands และเรียนรู้การใช้คำสั่งระดับต่ำของ Git อย่าง `hash-object`, `cat-file`, `ls-tree`, `rev-parse`, `update-ref`, `write-tree`, `commit-tree` และ `symbolic-ref` เพื่อมองทะลุเข้าไปเห็นกลไกภายในที่แท้จริงของ Git จนสามารถสร้าง commit ทั้งอันได้ด้วยมือโดยไม่ต้องพึ่งคำสั่งระดับสูงเลยแม้แต่คำสั่งเดียว

---

## สารบัญของ Part นี้

- Step 561: Plumbing vs Porcelain commands คืออะไร
- Step 562: `git hash-object` — คำนวณ SHA hash และสร้าง blob object
- Step 563: `git cat-file` — ตรวจสอบ type, content, size ของ object
- Step 564: `git ls-tree` — ดูเนื้อหาของ tree object
- Step 565: `git rev-parse` — แปลง reference เป็น full hash
- Step 566: `git update-ref` — แก้ไข ref โดยตรงแบบ low-level
- Step 567: `git write-tree`, `git commit-tree` — สร้าง commit ด้วยมือแบบ plumbing
- Step 568: `git symbolic-ref` — จัดการ HEAD โดยตรง
- Step 569: ทำไมต้องรู้ plumbing commands
- Step 570: แบบฝึกหัด — สร้าง commit ทั้งอันด้วย plumbing command ล้วน ๆ

---

## Step 561: Plumbing vs Porcelain commands คืออะไร

ใน **Part 56** เราเรียนรู้ไปแล้วว่า Git เก็บข้อมูลทุกอย่างเป็น **object** สี่ชนิด (blob, tree, commit, tag) ที่เชื่อมกันด้วย SHA hash และมี **ref** เป็นตัวชี้ที่มนุษย์อ่านง่ายไปยัง commit เหล่านั้น สิ่งที่เรายังไม่ได้ทำคือ **ลงมือแตะต้อง object เหล่านี้โดยตรงด้วยมือของเราเอง** — และนั่นคือสิ่งที่ Part นี้จะพาไปทำ

Linus Torvalds ออกแบบ Git ตั้งแต่วันแรกให้แบ่งคำสั่งออกเป็นสองชั้นอย่างชัดเจน โดยยืมคำศัพท์มาจากช่างประปา (plumber):

> **Plumbing** = ท่อน้ำและข้อต่อที่อยู่ข้างในผนัง — คนทั่วไปไม่เห็น แต่เป็นสิ่งที่ทำให้น้ำไหลได้จริง
> **Porcelain** = ก๊อกน้ำ อ่างล้างหน้า ชักโครก — สิ่งที่คนทั่วไปมองเห็นและใช้งานประจำวัน

### Porcelain commands คืออะไร

**Porcelain commands** คือคำสั่งระดับสูงที่ออกแบบมาให้ **มนุษย์ใช้งานสะดวก** มี output ที่อ่านง่าย มีข้อความช่วยเหลือ (hint) เวลาทำผิดพลาด และรวมหลายขั้นตอนภายในไว้เป็นคำสั่งเดียว

ตัวอย่างคำสั่ง porcelain ที่เราใช้กันมาตลอดหลักสูตรนี้:

```bash
git add
git commit
git push
git pull
git status
git log
git branch
git merge
git checkout
```

เวลาคุณพิมพ์ `git commit -m "fix bug"` เบื้องหลังมันไม่ได้ทำอะไรวิเศษเลย — มันแค่**เรียกใช้ plumbing commands หลายตัวเรียงต่อกัน**ให้คุณอัตโนมัติ

### Plumbing commands คืออะไร

**Plumbing commands** คือคำสั่งระดับต่ำที่ทำงานกับ object database และ ref โดยตรง แบบ **atomic** (ทำทีละอย่างเดียว ไม่รวมหลายขั้นตอน) output ของมันออกแบบมาให้**โปรแกรมอื่นอ่านต่อได้ง่าย** (machine-readable) มากกว่าจะสวยงามสำหรับมนุษย์

ตัวอย่างคำสั่ง plumbing ที่เราจะเรียนใน Part นี้:

```bash
git hash-object
git cat-file
git ls-tree
git rev-parse
git update-ref
git write-tree
git commit-tree
git symbolic-ref
```

### ทำไม Git ถึงแบ่งสองชั้นแบบนี้

เหตุผลสำคัญมี 2 ข้อ:

1. **Composability (การประกอบร่างได้)** — เพราะ plumbing commands แต่ละตัวทำงานเล็ก ๆ ชัดเจนแค่อย่างเดียว คุณสามารถเอามันมาต่อกันด้วย shell pipe เพื่อสร้างเครื่องมือใหม่ ๆ ของตัวเองได้ (นี่คือวิธีที่ porcelain commands ถูกสร้างขึ้นมาตั้งแต่แรก — คำสั่ง `git commit` เวอร์ชันแรกสุดในประวัติศาสตร์ก็เป็นแค่ shell script ที่เรียก plumbing ต่อกัน)
2. **Stability (ความเสถียรของ API)** — plumbing commands แทบไม่เปลี่ยนแปลง format output เลยตลอดหลายสิบปี เพราะมันคือ "สัญญา" ที่โปรแกรมอื่น ๆ (เช่น GUI tools, CI/CD scripts, Git hooks) พึ่งพาอยู่ ในขณะที่ porcelain commands สามารถเปลี่ยน output ที่แสดงให้มนุษย์ดูได้เรื่อย ๆ ตามความสวยงามและ UX โดยไม่กระทบระบบที่พึ่งพา plumbing

### ตารางเปรียบเทียบ

| คุณสมบัติ | Porcelain | Plumbing |
|---|---|---|
| กลุ่มเป้าหมาย | มนุษย์ | โปรแกรม/สคริปต์ |
| Output | สวยงาม อ่านง่าย มีสี | ดิบ ตรงไปตรงมา เหมาะกับ parse |
| ความซับซ้อนภายใน | รวมหลายขั้นตอนไว้ในคำสั่งเดียว | ทำทีละอย่างเดียว (atomic) |
| ความเสถียรของ format | เปลี่ยนได้ตามเวอร์ชัน | แทบไม่เปลี่ยนเลย |
| ตัวอย่าง | `commit`, `push`, `status`, `log` | `hash-object`, `cat-file`, `write-tree` |
| เจอบ่อยแค่ไหนในชีวิตประจำวัน | ใช้ทุกวัน | แทบไม่ใช้ตรง ๆ (แต่ใช้ทางอ้อมตลอดเวลา) |

### ทดลองดูด้วยตาตัวเอง

ลองสร้างโฟลเดอร์ทดลองใหม่ แล้วดูว่า repository ที่ยังไม่มีอะไรเลยมีโครงสร้างอย่างไร:

```bash
mkdir ~/git-course/part-57-plumbing
cd ~/git-course/part-57-plumbing
git init
```

ผลลัพธ์:

```
Initialized empty Git repository in /home/user/git-course/part-57-plumbing/.git/
```

ลองดูโครงสร้างโฟลเดอร์ `.git` ที่เพิ่งถูกสร้าง:

```bash
find .git -maxdepth 2 -type d
```

```
.git
.git/objects
.git/objects/info
.git/objects/pack
.git/refs
.git/refs/heads
.git/refs/tags
.git/branches
.git/hooks
.git/info
```

สังเกตว่า `.git/objects` ยังว่างเปล่า — ยังไม่มี object ใด ๆ ในระบบเลย เพราะเรายังไม่เคย commit อะไรเลย ต่อไปเราจะใช้ plumbing command ตัวแรกเพื่อสร้าง object ตัวแรกด้วยมือของเราเอง

---

## Step 562: `git hash-object` — คำนวณ SHA hash และสร้าง blob object

`git hash-object` คือ plumbing command ที่ **คำนวณ SHA hash ของเนื้อหาไฟล์** ตามรูปแบบที่ Git ใช้ภายใน และ (ถ้าสั่งด้วย flag ที่ถูกต้อง) **เขียนเนื้อหานั้นลงเป็น blob object จริง** ใน object database

### ทำความเข้าใจก่อนว่า blob คืออะไร (ทบทวนจาก Part 56)

**Blob** คือ object ที่เก็บ **เนื้อหาไฟล์ดิบ ๆ** โดยไม่มีชื่อไฟล์หรือ permission ติดมาด้วยเลย — Git มองว่า "เนื้อหา" กับ "ชื่อไฟล์" เป็นคนละเรื่องกัน ชื่อไฟล์ถูกเก็บแยกไว้ใน tree object ต่างหาก

### วิธี Git คำนวณ hash (สูตรที่ต้องเข้าใจ)

Git ไม่ได้ hash เนื้อหาไฟล์ตรง ๆ แต่จะ**เติม header ไว้ข้างหน้า**ก่อนแล้วค่อย hash ทั้งก้อน:

```
"<type> <ขนาดเป็นไบต์>\0<เนื้อหาไฟล์>"
```

ตัวอย่างเช่น ถ้าไฟล์มีเนื้อหาคือ `hello git\n` (10 ไบต์ รวม newline) header ที่ถูกเติมจะเป็น:

```
blob 10\0hello git\n
```

จากนั้น Git จะนำก้อนข้อมูลทั้งหมดนี้ไปคำนวณ SHA-1 (หรือ SHA-256 ถ้า repo ตั้งค่าไว้แบบนั้น) เพื่อได้ hash 40 ตัวอักษร (หรือ 64 ตัวสำหรับ SHA-256)

### ลองคำนวณ hash แบบไม่เขียนลงดิสก์

```bash
cd ~/git-course/part-57-plumbing
echo -n "hello git" > file1.txt
git hash-object file1.txt
```

ผลลัพธ์ (ตัวอย่าง — hash จริงจะขึ้นกับเนื้อหาไฟล์เป๊ะ ๆ):

```
a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f
```

หมายเหตุสำคัญ: คำสั่งนี้ **แค่คำนวณ hash ให้ดู** เท่านั้น **ยังไม่ได้เขียนอะไรลงใน `.git/objects` เลย** ลองตรวจสอบได้:

```bash
find .git/objects -type f
```

```
(ไม่มี output — ยังว่างเปล่า)
```

### เขียน object จริงด้วย `-w`

ถ้าต้องการให้ Git **เขียน blob object จริงลงใน object database** ต้องเติม flag `-w` (write):

```bash
git hash-object -w file1.txt
```

```
a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f
```

hash ที่ได้จะเหมือนเดิมทุกประการ (เพราะเนื้อหาไฟล์เหมือนเดิม) แต่คราวนี้ตรวจสอบ `.git/objects` อีกครั้ง:

```bash
find .git/objects -type f
```

```
.git/objects/a5/e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f
```

จะเห็นว่า Git สร้างโฟลเดอร์ย่อยตาม **2 ตัวอักษรแรกของ hash** (`a5`) แล้วเก็บไฟล์ที่เหลือ (38 ตัวอักษรที่เหลือ) เป็นชื่อไฟล์ — นี่คือเหตุผลว่าทำไมเวลาเราพิมพ์ hash แบบย่อ (เช่น `a5e07d3`) Git ถึงยังหาเจอ เพราะมันแค่เดินเข้าไปในโฟลเดอร์ `a5/` แล้วหาไฟล์ที่ขึ้นต้นด้วย `e07d3d3` เท่านั้น

### hash-object ทำงานกับหลายไฟล์พร้อมกันได้

```bash
echo -n "second file content" > file2.txt
git hash-object -w file1.txt file2.txt
```

```
a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f
b3f8b6c5d9e2a1f4c7d8e9f0a1b2c3d4e5f6a7b8
```

### hash-object ทำงานกับ stdin ได้เช่นกัน

```bash
echo -n "content from stdin" | git hash-object -w --stdin
```

```
c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7
```

flag `--stdin` บอกให้ Git อ่านเนื้อหาจาก standard input แทนที่จะอ่านจากไฟล์ — มีประโยชน์มากเวลาต้องการสร้าง blob จากข้อมูลที่ generate ขึ้นมาสด ๆ โดยไม่อยากสร้างไฟล์ชั่วคราวก่อน

### สรุป flag สำคัญของ `hash-object`

| Flag | ความหมาย |
|---|---|
| (ไม่มี flag) | คำนวณ hash แสดงผลอย่างเดียว ไม่เขียนอะไร |
| `-w` | เขียน object ลง object database จริง |
| `--stdin` | อ่านเนื้อหาจาก standard input แทนไฟล์ |
| `-t <type>` | ระบุ type ของ object (ค่า default คือ `blob`) |
| `--stdin-paths` | อ่านรายชื่อไฟล์จาก stdin ทีละบรรทัด แล้ว hash ทุกไฟล์ |

ทดลอง `-t` เพื่อความเข้าใจ (แม้ในทางปฏิบัติเราแทบไม่ใช้ hash-object กับ type อื่นนอกจาก blob):

```bash
git hash-object -t blob file1.txt
```

```
a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f
```

**ข้อสังเกตสำคัญที่ต้องจำ:** เมื่อไหร่ก็ตามที่คุณสั่ง `git add <file>` ในชีวิตประจำวัน สิ่งที่ Git ทำเบื้องหลังคือการเรียก `hash-object -w` แบบเดียวกันนี้เป๊ะ ๆ (แค่ทำผ่าน internal library ไม่ใช่การเรียก process ใหม่) แล้วเอา hash ที่ได้ไปบันทึกไว้ใน staging area (index)

---

## Step 563: `git cat-file` — ตรวจสอบ type, content, size ของ object

ตอนนี้เรามี blob object อยู่ใน `.git/objects` แล้ว 3 ตัว แต่ถ้าเราอยากรู้ว่าข้างในมันมีอะไรอยู่ เราต้องใช้ `git cat-file` ซึ่งเป็น plumbing command สำหรับ **"เปิดดูข้างใน" object** โดยไม่สนใจว่ามันจะเป็น blob, tree, commit หรือ tag

### `cat-file -t` — ดู type ของ object

```bash
git cat-file -t a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f
```

```
blob
```

flag `-t` (type) บอกว่า object ที่ระบุนี้เป็น object ประเภทอะไร — มีค่าที่เป็นไปได้ 4 แบบคือ `blob`, `tree`, `commit`, `tag`

### `cat-file -p` — ดูเนื้อหาข้างในแบบ pretty-print

```bash
git cat-file -p a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f
```

```
hello git
```

flag `-p` (pretty-print) คือ flag ที่ใช้บ่อยที่สุด มันจะแสดงเนื้อหาของ object โดยจัดรูปแบบให้เหมาะสมกับ type นั้น ๆ โดยอัตโนมัติ:

- ถ้าเป็น **blob** → แสดงเนื้อหาไฟล์ดิบ ๆ
- ถ้าเป็น **tree** → แสดงรายการไฟล์/โฟลเดอร์ (เหมือน `ls-tree` ซึ่งเราจะเรียนใน Step ถัดไป)
- ถ้าเป็น **commit** → แสดง tree hash, parent, author, committer, message
- ถ้าเป็น **tag** → แสดงข้อมูล annotated tag

### `cat-file -s` — ดูขนาดของ object (เป็นไบต์)

```bash
git cat-file -s a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f
```

```
9
```

ขนาด 9 ไบต์ตรงกับเนื้อหา `hello git` (9 ตัวอักษร ไม่รวม newline เพราะเราใช้ `echo -n` ตอนสร้างไฟล์) — ตัวเลขนี้คือขนาดของ**เนื้อหาจริง** ไม่รวม header `blob 9\0` ที่ Git เติมไว้ตอนคำนวณ hash

### ทดลองกับ hash ที่สั้นลง (abbreviated hash)

Git อนุญาตให้ใช้ hash แบบย่อได้ ตราบใดที่มันยังไม่ชนกับ object อื่นในระบบ:

```bash
git cat-file -p a5e07d3
```

```
hello git
```

### `cat-file --batch` และ `--batch-check` — ตรวจสอบหลาย object พร้อมกันแบบ script-friendly

สำหรับการเขียนสคริปต์ที่ต้องเช็ค object จำนวนมาก การเรียก `cat-file -p` ทีละครั้งช้าเกินไป (เพราะแต่ละครั้งต้อง spawn process ใหม่) Git จึงมี mode พิเศษที่รับ input ต่อเนื่องทาง stdin:

```bash
echo "a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f" | git cat-file --batch
```

```
a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f blob 9
hello git
```

ผลลัพธ์บรรทัดแรกคือ `<hash> <type> <size>` ตามด้วยเนื้อหาจริงในบรรทัดถัดไป รูปแบบนี้ถูกออกแบบมาให้โปรแกรมอ่านต่อได้ง่ายมาก (เช่น GUI tools ที่ต้องการดึงข้อมูล object จำนวนมากอย่างรวดเร็ว)

ถ้าต้องการแค่ metadata โดยไม่เอาเนื้อหา ใช้ `--batch-check`:

```bash
echo "a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f" | git cat-file --batch-check
```

```
a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f blob 9
```

### ทดสอบกับ object ที่ไม่มีอยู่จริง

```bash
git cat-file -t 0000000000000000000000000000000000000
```

```
fatal: Not a valid object name 0000000000000000000000000000000000000
```

Git จะแจ้ง error ทันทีถ้า hash ที่ระบุไม่มีอยู่จริงใน object database — นี่คือกลไกที่ทำให้ Git detect ข้อมูลเสียหาย (corruption) ได้ง่ายมาก เพราะทุก object ถูกอ้างอิงด้วย hash ของเนื้อหาตัวมันเอง ถ้าเนื้อหาถูกแก้ไข hash จะไม่ตรงกันทันที

### สรุปตาราง flag ของ `cat-file`

| Flag | ความหมาย |
|---|---|
| `-t` | แสดง type ของ object |
| `-s` | แสดงขนาดของ object (ไบต์) |
| `-p` | แสดงเนื้อหาแบบ pretty-print ตาม type |
| `-e` | ตรวจสอบว่า object มีอยู่จริงหรือไม่ (ไม่แสดงผลอะไร แค่คืน exit code) |
| `--batch` | อ่าน hash จาก stdin ทีละบรรทัด แสดง metadata + เนื้อหาเต็ม |
| `--batch-check` | เหมือน `--batch` แต่แสดงแค่ metadata ไม่เอาเนื้อหา |

ทดลอง `-e` เพื่อดูว่ามันใช้ตรวจสอบแบบ script ได้อย่างไร:

```bash
git cat-file -e a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f && echo "object มีอยู่จริง"
```

```
object มีอยู่จริง
```

---

## Step 564: `git ls-tree` — ดูเนื้อหาของ tree object

ตอนนี้เรามีแค่ blob object ลอย ๆ 3 ตัว ที่ยังไม่ได้ผูกกับชื่อไฟล์ใด ๆ เลย เพราะอย่างที่บอกไว้ — blob เก็บแค่เนื้อหา ไม่เก็บชื่อไฟล์ ชื่อไฟล์ถูกเก็บอยู่ใน **tree object** ต่างหาก

### สร้าง tree object แรกด้วย staging area แบบปกติก่อน

เพื่อให้มี tree object ไว้ทดลอง ลอง add ไฟล์เข้า staging area ด้วยคำสั่ง porcelain ตามปกติก่อน:

```bash
git add file1.txt file2.txt
```

ตอนนี้ Git ได้สร้าง tree object ไว้ใน staging area (index) แล้ว (แต่ยังไม่ถูกเขียนเป็น tree object ถาวรจนกว่าจะ commit) เราสามารถบังคับให้ Git สร้าง tree object จาก staging area ตอนนี้เลยด้วยคำสั่ง `write-tree` (ซึ่งเราจะเรียนละเอียดใน Step 567):

```bash
git write-tree
```

```
d4a8f2e1c9b6a3d7e5f8c2b1a4d9e6f3c8b5a2d1
```

hash ที่ได้คือ hash ของ tree object ที่เพิ่งถูกสร้างขึ้น เก็บค่านี้ไว้ก่อน (สมมติเราใช้ตัวแปร `$TREE`):

```bash
TREE=$(git write-tree)
echo $TREE
```

### `ls-tree` — ดูเนื้อหาข้างใน tree object

```bash
git ls-tree $TREE
```

```
100644 blob a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f    file1.txt
100644 blob b3f8b6c5d9e2a1f4c7d8e9f0a1b2c3d4e5f6a7b8    file2.txt
```

มาแยกวิเคราะห์แต่ละคอลัมน์ทีละส่วน:

| คอลัมน์ | ความหมาย |
|---|---|
| `100644` | **File mode** — บอกสิทธิ์และประเภทไฟล์ (โหมด octal ของ Unix) |
| `blob` | Type ของ object ที่ entry นี้ชี้ไป |
| `a5e07d3...` | Hash ของ object นั้น |
| `file1.txt` | ชื่อไฟล์ (path) ที่ผูกกับ object นี้ |

### ทำความเข้าใจ file mode ที่พบบ่อย

| File mode | ความหมาย |
|---|---|
| `100644` | ไฟล์ปกติ อ่าน-เขียนได้ (ไม่ executable) |
| `100755` | ไฟล์ปกติ ที่เป็น executable (มีสิทธิ์รันได้) |
| `120000` | Symbolic link |
| `040000` | Subdirectory (ชี้ไปยัง tree object อีกอันหนึ่ง ไม่ใช่ blob) |
| `160000` | Gitlink — ใช้สำหรับ submodule (ชี้ไปยัง commit ใน repository อื่น) |

### ทดลองสร้างโครงสร้างที่มี subdirectory

```bash
mkdir subdir
echo -n "nested file content" > subdir/nested.txt
git add subdir/nested.txt
git write-tree
```

```
e7c1f4b8a2d9e5f6c3b0a7d4e1f8c5b2a9d6e3f0
```

```bash
git ls-tree e7c1f4b8a2d9e5f6c3b0a7d4e1f8c5b2a9d6e3f0
```

```
100644 blob a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f    file1.txt
100644 blob b3f8b6c5d9e2a1f4c7d8e9f0a1b2c3d4e5f6a7b8    file2.txt
040000 tree f2a9d6e3f0c1b4a7d8e5f2c9b6a3d0e7f4c1b8a5    subdir
```

สังเกตว่าแถวสุดท้ายมี type เป็น `tree` ไม่ใช่ `blob` — นี่คือ **tree ที่ซ้อนอยู่ใน tree** (nested tree) ซึ่งแสดงถึงโฟลเดอร์ `subdir` ถ้าต้องการดูเนื้อหาข้างในโฟลเดอร์นี้ ต้องเรียก `ls-tree` อีกครั้งกับ hash ของ tree ย่อยนั้น:

```bash
git ls-tree f2a9d6e3f0c1b4a7d8e5f2c9b6a3d0e7f4c1b8a5
```

```
100644 blob c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7    nested.txt
```

นี่คือหลักฐานชัดเจนของสิ่งที่เราเรียนใน Part 56 ว่า **tree object สามารถชี้ไปยัง tree object อื่นได้ (nested structure)** ซึ่งจำลองโครงสร้างโฟลเดอร์ที่ซ้อนกันในระบบไฟล์จริงได้อย่างสมบูรณ์

### `ls-tree -r` — ดูแบบ recursive ในคำสั่งเดียว

การต้องไล่เปิด tree ทีละชั้นด้วยมือนั้นน่าเบื่อมากถ้ามีโฟลเดอร์ซ้อนกันลึกหลายชั้น flag `-r` ช่วยให้ Git ไล่เข้าไปดูทุกชั้นให้อัตโนมัติ และแสดงเฉพาะ blob (ไฟล์จริง) เท่านั้น พร้อม path เต็ม:

```bash
git ls-tree -r e7c1f4b8a2d9e5f6c3b0a7d4e1f8c5b2a9d6e3f0
```

```
100644 blob a5e07d3d3a29de3c4a9c3b76b28e1b6f5b0d8c9f    file1.txt
100644 blob b3f8b6c5d9e2a1f4c7d8e9f0a1b2c3d4e5f6a7b8    file2.txt
100644 blob c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7    subdir/nested.txt
```

สังเกตว่า `subdir` ที่เคยเป็น entry ประเภท `tree` หายไป และถูกแทนที่ด้วยไฟล์ข้างในที่มี path เต็มเป็น `subdir/nested.txt` แทน — นี่คือมุมมองแบบ "flat" ของทุกไฟล์ใน commit นั้น

### flag อื่น ๆ ที่มีประโยชน์ของ `ls-tree`

| Flag | ความหมาย |
|---|---|
| `-r` | แสดงแบบ recursive (ไล่เข้าไปทุก subdirectory) |
| `-t` | เมื่อใช้คู่กับ `-r` จะแสดง tree object ของ subdirectory ด้วย (ไม่ซ่อน) |
| `-d` | แสดงเฉพาะ tree entry เท่านั้น (ซ่อน blob) |
| `--name-only` | แสดงแค่ชื่อไฟล์ ไม่แสดง mode/type/hash |
| `-l` (long) | แสดงขนาดไฟล์เพิ่มด้วย (คอลัมน์ที่ 5) |

ลองใช้ `--name-only` ร่วมกับ `-r` เพื่อดู path ทั้งหมดแบบสั้น ๆ:

```bash
git ls-tree -r --name-only e7c1f4b8a2d9e5f6c3b0a7d4e1f8c5b2a9d6e3f0
```

```
file1.txt
file2.txt
subdir/nested.txt
```

รูปแบบนี้มีประโยชน์มากเวลาต้องการดึงรายชื่อไฟล์ทั้งหมดใน commit หนึ่ง ๆ ไปใช้ใน script อื่นต่อ

---

## Step 565: `git rev-parse` — แปลง reference เป็น full hash

จนถึงตอนนี้เราต้อง copy hash ยาว ๆ ไปมาตลอดเวลา ซึ่งไม่สะดวกเลยในการใช้งานจริง `git rev-parse` คือ plumbing command ที่ทำหน้าที่ **แปล (resolve) reference ทุกรูปแบบที่มนุษย์อ่านง่ายให้กลายเป็น full SHA hash**

### ทำไมต้องมีคำสั่งนี้

ใน Git มีวิธีอ้างอิง commit/object ได้หลายแบบมาก เช่น branch name, tag name, `HEAD`, `HEAD~2`, `HEAD^`, `main@{yesterday}` ฯลฯ แต่เบื้องหลัง object database ทุกอย่างต้องอ้างอิงด้วย **full SHA hash** เท่านั้น — `rev-parse` คือตัวกลางที่แปลงจากรูปแบบที่มนุษย์สะดวกไปเป็น hash ที่ระบบใช้จริง

### ตัวอย่างการใช้งานพื้นฐาน

ก่อนอื่นมาสร้าง commit จริงสักอันด้วยวิธีปกติก่อน เพื่อให้มี ref ไว้ทดลอง:

```bash
git commit -m "initial commit สำหรับทดลอง rev-parse"
```

```
[master (root-commit) 9f2c8a1] initial commit สำหรับทดลอง rev-parse
 3 files changed, 3 insertions(+)
 create mode 100644 file1.txt
 create mode 100644 file2.txt
 create mode 100644 subdir/nested.txt
```

ตอนนี้ลอง `rev-parse` กับ branch name:

```bash
git rev-parse master
```

```
9f2c8a1d4e7b3f6a9c2d5e8f1b4a7d0c3e6f9a2b
```

ลองกับ `HEAD`:

```bash
git rev-parse HEAD
```

```
9f2c8a1d4e7b3f6a9c2d5e8f1b4a7d0c3e6f9a2b
```

เพราะตอนนี้เรามีแค่ commit เดียว `HEAD` กับ `master` เลยชี้ไปที่ commit เดียวกัน

### `rev-parse` กับ relative reference

หลังจากสร้าง commit ที่สองแล้ว:

```bash
echo -n "second commit content" > file3.txt
git add file3.txt
git commit -m "second commit"
```

ลอง `rev-parse` กับ `HEAD~1` (commit ก่อนหน้า 1 อัน):

```bash
git rev-parse HEAD~1
```

```
9f2c8a1d4e7b3f6a9c2d5e8f1b4a7d0c3e6f9a2b
```

จะได้ hash ของ commit แรกกลับมา (ตรงกับที่ `git rev-parse master` ให้ผลตอนแรกก่อนมี commit ที่สอง) ลอง `HEAD^` ซึ่งหมายถึง parent ตัวแรกเช่นกัน (ใช้แทนกันได้กับ `HEAD~1` ในกรณี commit ปกติที่ไม่ใช่ merge commit):

```bash
git rev-parse HEAD^
```

```
9f2c8a1d4e7b3f6a9c2d5e8f1b4a7d0c3e6f9a2b
```

### `rev-parse` กับ hash แบบย่อ

```bash
git rev-parse 9f2c8a1
```

```
9f2c8a1d4e7b3f6a9c2d5e8f1b4a7d0c3e6f9a2b
```

นี่คือประโยชน์ที่ใช้บ่อยมากในสคริปต์: รับ hash แบบสั้นจากผู้ใช้ แล้วแปลงเป็น full hash ก่อนนำไปประมวลผลต่อ เพื่อความชัดเจนไม่กำกวม

### `rev-parse --short` — ย่อ hash กลับ

ทำงานตรงข้ามกัน คือรับ full hash แล้วย่อให้สั้นลง:

```bash
git rev-parse --short HEAD
```

```
9f2c8a1
```

### flag ที่มีประโยชน์อื่น ๆ ของ `rev-parse`

| Flag/รูปแบบ | ความหมาย |
|---|---|
| `--short` | ย่อ hash ให้สั้นลง (default 7 ตัวอักษร หรือมากกว่าถ้าจำเป็นเพื่อความไม่กำกวม) |
| `--verify` | ตรวจสอบว่า reference ที่ให้มาถูกต้องและ resolve ได้จริง (มีประโยชน์ในสคริปต์เพื่อเช็ค error) |
| `--abbrev-ref` | แปลง full ref path (เช่น `refs/heads/master`) ให้เหลือแค่ `master` |
| `--is-inside-work-tree` | เช็คว่ากำลังอยู่ใน working tree ของ Git repo หรือไม่ (คืนค่า `true`/`false`) |
| `--show-toplevel` | แสดง path เต็มของ root directory ของ repository |
| `--git-dir` | แสดง path ของโฟลเดอร์ `.git` |

ตัวอย่างการใช้ `--verify` ในสคริปต์เพื่อตรวจสอบว่า branch มีอยู่จริงก่อนใช้งานต่อ:

```bash
git rev-parse --verify feature-xyz 2>/dev/null && echo "branch มีอยู่จริง" || echo "ไม่พบ branch นี้"
```

```
ไม่พบ branch นี้
```

ตัวอย่างการใช้ `--abbrev-ref` เพื่อดูชื่อ branch ปัจจุบันแบบสั้น ๆ (เทคนิคนี้ใช้บ่อยมากในการเขียน shell prompt ที่แสดงชื่อ branch):

```bash
git rev-parse --abbrev-ref HEAD
```

```
master
```

ตัวอย่างการใช้ `--show-toplevel` เพื่อหา root ของโปรเจกต์จากที่ไหนก็ได้ในโฟลเดอร์:

```bash
cd subdir
git rev-parse --show-toplevel
```

```
/home/user/git-course/part-57-plumbing
```

```bash
cd ..
```

**ข้อสังเกต:** `rev-parse` คือคำสั่งที่ถูกเรียกใช้ภายในเบื้องหลังแทบทุกครั้งที่ porcelain command ต้องแปลง argument ที่ผู้ใช้พิมพ์เข้ามาให้กลายเป็น hash — มันคือ "ล่าม" ที่ทำงานอยู่เบื้องหลังตลอดเวลาโดยที่เราไม่เคยสังเกต

---

## Step 566: `git update-ref` — แก้ไข ref โดยตรงแบบ low-level

เราเรียนรู้จาก Part 56 มาแล้วว่า **branch คือแค่ไฟล์ text ธรรมดาใน `.git/refs/heads/` ที่เก็บ hash ของ commit เอาไว้** ปกติเราไม่เคยแก้ไฟล์นี้ตรง ๆ ด้วยมือ (เช่นใช้ text editor เปิดแก้) เพราะเสี่ยงทำผิดพลาดได้ง่ายมาก — Git จึงมี plumbing command ที่ปลอดภัยกว่าสำหรับงานนี้โดยเฉพาะ นั่นคือ `git update-ref`

### ทำไมไม่ควรแก้ไฟล์ ref ด้วยมือตรง ๆ

ในทางทฤษฎีคุณสามารถทำแบบนี้ได้:

```bash
echo "9f2c8a1d4e7b3f6a9c2d5e8f1b4a7d0c3e6f9a2b" > .git/refs/heads/master
```

แต่วิธีนี้ **อันตรายมาก** เพราะ:

1. ไม่มีการตรวจสอบว่า hash ที่ใส่ไปมีอยู่จริงในระบบหรือไม่
2. ไม่มีการบันทึกลง **reflog** (ประวัติการเปลี่ยนแปลงของ ref ที่ Git ใช้กู้คืนข้อมูลเวลาทำผิดพลาด — เราจะเรียนละเอียดใน Part ถัดไปเกี่ยวกับ Git internals)
3. ถ้ามีกระบวนการอื่นเขียน ref พร้อมกัน (race condition) อาจเกิดข้อมูลเสียหายได้
4. ในระบบใหม่ที่ใช้ **packed-refs** ref อาจไม่ได้อยู่เป็นไฟล์แยกแบบนี้เลยด้วยซ้ำ

`git update-ref` แก้ปัญหาทั้งหมดนี้ด้วยการเป็น API ที่ปลอดภัยสำหรับแก้ไข ref โดยตรง

### วิธีใช้งานพื้นฐาน

รูปแบบคำสั่ง:

```bash
git update-ref <ref> <new-value>
```

ตัวอย่าง สร้าง branch ใหม่ชื่อ `experiment` ให้ชี้ไปยัง commit ปัจจุบันโดยไม่ใช้ `git branch` เลย:

```bash
git update-ref refs/heads/experiment HEAD
```

ตรวจสอบผล:

```bash
git branch
```

```
  experiment
* master
```

จะเห็นว่า branch `experiment` ถูกสร้างขึ้นจริง ทั้ง ๆ ที่เราไม่เคยพิมพ์ `git branch experiment` เลยสักครั้ง — นี่คือหลักฐานว่า `git branch <name>` ที่เราใช้ทุกวันจริง ๆ แล้วก็แค่เรียก `update-ref` เบื้องหลังนั่นเอง

### ตรวจสอบไฟล์ที่เกิดขึ้นจริง

```bash
cat .git/refs/heads/experiment
```

```
9f2c8a1d4e7b3f6a9c2d5e8f1b4a7d0c3e6f9a2b
```

เนื้อหาไฟล์เป็นแค่ hash เปล่า ๆ 1 บรรทัด ตรงตามที่เรียนไว้ใน Part 56 ทุกประการ

### `update-ref` พร้อมค่าเก่าเพื่อความปลอดภัย (compare-and-swap)

จุดเด่นที่สำคัญที่สุดของ `update-ref` ที่การแก้ไฟล์ตรง ๆ ทำไม่ได้คือ **การระบุค่าเก่าที่คาดหวัง** เพื่อป้องกันไม่ให้ทับข้อมูลที่ถูกเปลี่ยนไปแล้วโดยกระบวนการอื่นระหว่างทาง:

```bash
git update-ref refs/heads/master <new-hash> <old-hash-ที่คาดว่าจะเป็นค่าปัจจุบัน>
```

ถ้าค่าปัจจุบันของ `refs/heads/master` ไม่ตรงกับ `<old-hash>` ที่ระบุไป คำสั่งจะ**ล้มเหลวทันที** แทนที่จะเขียนทับไปเฉย ๆ — กลไกนี้เรียกว่า **compare-and-swap (CAS)** และเป็นสิ่งที่ทำให้ Git ปลอดภัยต่อ race condition แม้จะมีหลายกระบวนการพยายามแก้ ref เดียวกันพร้อมกัน

ตัวอย่างการใช้งานจริง — ลองสร้างสถานการณ์ที่ค่าที่คาดไว้ผิด:

```bash
git update-ref refs/heads/experiment HEAD "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
```

```
fatal: update_ref failed for ref 'refs/heads/experiment': cannot lock ref 'refs/heads/experiment': is at 9f2c8a1d4e7b3f6a9c2d5e8f1b4a7d0c3e6f9a2b but expected aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

Git ปฏิเสธการเขียนทันที เพราะค่าปัจจุบันจริง ๆ (`9f2c8a1...`) ไม่ตรงกับค่าที่เราบอกว่า "คาดว่าจะเป็น" (`aaaa...`) — ฟีเจอร์นี้สำคัญมากในระบบที่มีการ push จากหลายที่พร้อมกัน (เช่น server-side ของ GitHub/GitLab เอง ก็ใช้กลไกแบบนี้ในการตรวจสอบว่า push ที่ส่งเข้ามาไม่ชนกับการเปลี่ยนแปลงที่เพิ่งเกิดขึ้น)

> **หมายเหตุ:** ค่าเก่าที่เป็นเลข `0` ล้วน 40 ตัว (`000...000`) มีความหมายพิเศษใน `update-ref` คือ "คาดว่า ref นี้ยังไม่มีอยู่เลย" (ใช้ตอนต้องการสร้าง ref ใหม่แบบปลอดภัย ป้องกันการเขียนทับ ref ที่มีอยู่ก่อนแล้วโดยไม่ตั้งใจ) ถ้า ref นั้นมีอยู่แล้วจริง จะได้ข้อความ error ว่า `reference already exists` แทน ไม่ใช่ข้อความ `is at ... but expected ...` เหมือนกรณีที่ค่าเก่าเป็น hash จริงแต่ไม่ตรงกัน

### ลบ ref ด้วย `update-ref -d`

```bash
git update-ref -d refs/heads/experiment
```

ตรวจสอบ:

```bash
git branch
```

```
* master
```

branch `experiment` หายไปแล้ว — นี่คือสิ่งที่ `git branch -d experiment` ทำเบื้องหลังนั่นเอง (บวกกับการตรวจสอบเพิ่มเติมว่า branch นั้น merge แล้วหรือยัง ซึ่งเป็นงานระดับ porcelain)

### `update-ref` ใช้แก้ tag ได้ด้วย

```bash
git update-ref refs/tags/v0.1.0 HEAD
git cat-file -t refs/tags/v0.1.0
```

```
commit
```

(สำหรับ lightweight tag แบบนี้ type ที่ได้จะเป็น `commit` ตรง ๆ เพราะ lightweight tag คือแค่ ref ที่ชี้ตรงไปยัง commit ไม่มี tag object ห่อไว้ — ต่างจาก annotated tag ที่จะมี tag object เป็นชั้นกลาง)

### สรุปรูปแบบคำสั่งของ `update-ref`

| คำสั่ง | ความหมาย |
|---|---|
| `git update-ref <ref> <value>` | ตั้งค่า ref ให้ชี้ไปยัง value ที่ระบุ |
| `git update-ref <ref> <new> <old>` | ตั้งค่าใหม่ แต่ต้องตรงกับค่าเก่าที่คาดไว้เท่านั้น (compare-and-swap) |
| `git update-ref -d <ref>` | ลบ ref นั้นทิ้ง |
| `git update-ref -d <ref> <old>` | ลบ ref แต่ต้องตรงกับค่าเก่าที่คาดไว้ก่อนถึงจะลบ |
| `git update-ref --stdin` | อ่านชุดคำสั่งหลายอันจาก stdin ทำเป็น transaction เดียว (atomic) |

---

## Step 567: `git write-tree`, `git commit-tree` — สร้าง commit ด้วยมือแบบ plumbing

ตอนนี้เรามีเครื่องมือครบพอที่จะ**ประกอบ commit object ทั้งอันขึ้นมาด้วยมือ**แล้ว โดยไม่ต้องพึ่ง `git commit` เลย นี่คือหัวใจสำคัญของ Part นี้

### ทบทวนโครงสร้างของ commit object (จาก Part 56)

commit object ประกอบด้วยข้อมูลหลัก ๆ ดังนี้:

```
tree <hash ของ tree object ที่เป็น root ของ snapshot นี้>
parent <hash ของ commit ก่อนหน้า> (มีได้หลายบรรทัด ถ้าเป็น merge commit)
author <ชื่อ> <email> <timestamp> <timezone>
committer <ชื่อ> <email> <timestamp> <timezone>

<ข้อความ commit message>
```

### `git write-tree` — สร้าง tree object จาก staging area

`git write-tree` คือคำสั่งที่อ่านสถานะปัจจุบันของ **staging area (index)** แล้วสร้าง tree object ให้ตรงกับสิ่งที่อยู่ใน staging area นั้นเป๊ะ ๆ (เราใช้คำสั่งนี้ไปแล้วใน Step 564)

ข้อสำคัญที่ต้องเข้าใจ: `write-tree` **ไม่สนใจ working directory เลย** มันสนใจแค่ staging area เท่านั้น — ถ้าคุณแก้ไฟล์ใน working directory แต่ยังไม่ `git add` การเปลี่ยนแปลงนั้นจะไม่ถูกรวมเข้า tree ที่สร้างขึ้นมา

### `git commit-tree` — สร้าง commit object ด้วยมือ

`git commit-tree` รับ tree hash เป็น argument หลัก แล้วสร้าง commit object ที่ชี้ไปยัง tree นั้น พร้อมกับ metadata ที่จำเป็น (author, committer, message, parent)

รูปแบบคำสั่งพื้นฐาน:

```bash
git commit-tree <tree-hash> -m "<commit message>"
```

หรือถ้ามี parent commit ให้ระบุด้วย `-p`:

```bash
git commit-tree <tree-hash> -p <parent-hash> -m "<commit message>"
```

**ข้อสำคัญ:** `commit-tree` **ไม่แตะ ref ใด ๆ เลย** มันแค่สร้าง commit object ลอย ๆ ไว้ใน object database แล้วคืน hash ของ commit ที่สร้างขึ้นมาให้เท่านั้น — การจะทำให้ branch ปัจจุบันขยับไปชี้ที่ commit ใหม่นี้ ต้องใช้ `update-ref` เพิ่มอีกขั้นตอนหนึ่ง (ซึ่งนี่คือสิ่งที่จะทำในแบบฝึกหัด Step 570)

### ตัวอย่างการใช้งานแบบครบวงจร

มาลองประกอบ commit ใหม่ด้วยมือทั้งหมดกัน โดยเริ่มจากแก้ไฟล์ที่มีอยู่:

```bash
echo -n "hello git - updated content" > file1.txt
git add file1.txt
NEW_TREE=$(git write-tree)
echo "Tree hash ใหม่: $NEW_TREE"
```

```
Tree hash ใหม่: 3c7e9d2a1b4f8c5e0a7d3b6c9e2f5a8d1b4c7e0a
```

ตอนนี้ประกอบ commit ด้วย `commit-tree` โดยระบุ parent เป็น commit ปัจจุบัน (`HEAD`):

```bash
PARENT=$(git rev-parse HEAD)
NEW_COMMIT=$(git commit-tree $NEW_TREE -p $PARENT -m "แก้ไข file1.txt ด้วยมือผ่าน plumbing commands ล้วน ๆ")
echo "Commit hash ใหม่: $NEW_COMMIT"
```

```
Commit hash ใหม่: 8b3f6c9e2d5a8b1c4f7e0a3d6b9c2e5f8a1d4b7c
```

ตรวจสอบเนื้อหาของ commit ที่สร้างขึ้นด้วย `cat-file -p`:

```bash
git cat-file -p $NEW_COMMIT
```

```
tree 3c7e9d2a1b4f8c5e0a7d3b6c9e2f5a8d1b4c7e0a
parent 9f2c8a1d4e7b3f6a9c2d5e8f1b4a7d0c3e6f9a2b
author Somchai Devteam <somchai@example.com> 1719400000 +0700
committer Somchai Devteam <somchai@example.com> 1719400000 +0700

แก้ไข file1.txt ด้วยมือผ่าน plumbing commands ล้วน ๆ
```

สังเกตว่า author/committer ถูกดึงมาจากค่า `user.name` และ `user.email` ที่ตั้งไว้ใน git config โดยอัตโนมัติ (เหมือนตอนใช้ `git commit` ปกติทุกประการ)

**สำคัญมาก:** ตอนนี้ commit นี้ **มีอยู่จริงใน object database แล้ว** แต่ **ไม่มี branch หรือ ref ใดชี้มาที่มันเลย** ลองเช็คดู:

```bash
git log --oneline
```

```
9f2c8a1 second commit
9f2c8a1 initial commit สำหรับทดลอง rev-parse
```

จะเห็นว่า `git log` ยังไม่เห็น commit ใหม่ที่เราเพิ่งสร้างเลย เพราะ `git log` เริ่มเดินย้อนกลับจาก `HEAD` และ `HEAD` ยังไม่ได้ถูกขยับไปที่ commit ใหม่ — commit ที่สร้างด้วย `commit-tree` แบบนี้เรียกว่าเป็น **"dangling commit"** (commit ที่ลอยอยู่ ไม่มีอะไรอ้างอิงถึง) ถ้าไม่ทำอะไรกับมันต่อ สุดท้ายมันจะถูกลบทิ้งโดย **garbage collection** (`git gc`) ในอนาคต

### ทำให้ branch ชี้ไปยัง commit ใหม่ (เชื่อมทุกอย่างเข้าด้วยกัน)

นี่คือขั้นตอนสุดท้ายที่ทำให้ commit ที่สร้างด้วยมือกลายเป็นส่วนหนึ่งของประวัติ branch จริง:

```bash
git update-ref refs/heads/master $NEW_COMMIT
```

ตรวจสอบอีกครั้ง:

```bash
git log --oneline
```

```
8b3f6c9 แก้ไข file1.txt ด้วยมือผ่าน plumbing commands ล้วน ๆ
9f2c8a1 second commit
9f2c8a1 initial commit สำหรับทดลอง rev-parse
```

commit ใหม่ปรากฏใน history แล้ว! และตรวจสอบเนื้อหาไฟล์ใน working directory:

```bash
git status
```

```
On branch master
nothing to commit, working tree clean
```

Git มองว่า working directory ตรงกับ snapshot ล่าสุดพอดี เพราะเรา `git add` ไฟล์ไว้ก่อนสร้าง tree แล้ว

### สรุปขั้นตอนทั้งหมดของการสร้าง commit แบบ plumbing

```
1. hash-object -w        → สร้าง blob object จากเนื้อหาไฟล์
2. (update-index / add)  → ผูก blob เข้ากับชื่อไฟล์ใน staging area
3. write-tree             → แปลง staging area เป็น tree object
4. commit-tree             → สร้าง commit object ที่ชี้ไปยัง tree + parent
5. update-ref               → ทำให้ branch ชี้ไปยัง commit ใหม่
```

นี่คือ **5 ขั้นตอนที่แท้จริง** ที่ซ่อนอยู่หลังคำสั่ง `git add .` และ `git commit -m "..."` เพียง 2 คำสั่งที่เราใช้กันทุกวัน

---

## Step 568: `git symbolic-ref` — จัดการ HEAD โดยตรง

เราเรียนรู้จาก Part 56 มาแล้วว่า `HEAD` ปกติไม่ได้เก็บ hash ตรง ๆ แต่เก็บเป็น **"ตัวชี้ไปยังตัวชี้อื่น"** (symbolic reference) ที่ชี้ไปยัง branch ปัจจุบัน `git symbolic-ref` คือ plumbing command สำหรับอ่านและแก้ไข symbolic reference นี้โดยตรง

### ตรวจสอบเนื้อหาไฟล์ `.git/HEAD` ตรง ๆ

```bash
cat .git/HEAD
```

```
ref: refs/heads/master
```

นี่คือรูปแบบของ symbolic ref — ไม่ใช่ hash แต่เป็นข้อความ `ref: <path ไปยัง ref อื่น>`

### อ่านค่าด้วย `symbolic-ref`

```bash
git symbolic-ref HEAD
```

```
refs/heads/master
```

คำสั่งนี้ให้ผลเหมือนการ `cat .git/HEAD` แต่ตัด prefix `ref: ` ออกให้อัตโนมัติ และมีข้อดีสำคัญคือ**ปลอดภัยกว่า** เพราะถ้า `HEAD` อยู่ในสถานะ **detached HEAD** (ชี้ตรงไปยัง commit hash โดยไม่ผ่าน branch) คำสั่งนี้จะแจ้ง error ให้ทันทีแทนที่จะคืนค่าผิด ๆ:

```bash
git checkout HEAD~1
git symbolic-ref HEAD
```

```
fatal: ref HEAD is not a symbolic ref
```

นี่คือวิธีที่สคริปต์ใช้ตรวจสอบว่า "ตอนนี้อยู่ใน detached HEAD หรือเปล่า" ได้อย่างแม่นยำ — ถ้าคำสั่งนี้ error แปลว่ากำลังอยู่ใน detached HEAD state

กลับไป branch ปกติก่อน:

```bash
git checkout master
```

### เขียนค่าด้วย `symbolic-ref`

`symbolic-ref` ไม่ได้ใช้แค่อ่านอย่างเดียว ยังใช้ **เปลี่ยนได้ด้วยว่า `HEAD` จะชี้ไป branch ไหน** โดยไม่ต้อง checkout จริง (คือไม่แตะ working directory หรือ staging area เลย):

```bash
git branch new-default-branch
git symbolic-ref HEAD refs/heads/new-default-branch
```

ตรวจสอบผล:

```bash
cat .git/HEAD
```

```
ref: refs/heads/new-default-branch
```

```bash
git branch
```

```
  master
* new-default-branch
```

จะเห็นว่า `HEAD` ถูกย้ายไปชี้ที่ `new-default-branch` แล้ว โดยที่ **working directory ไม่ถูกเปลี่ยนแปลงเลยแม้แต่ไฟล์เดียว** (ต่างจาก `git checkout` หรือ `git switch` ที่จะอัปเดตไฟล์ใน working directory ให้ตรงกับ branch ปลายทางด้วย) นี่คือความแตกต่างสำคัญที่สุดระหว่าง `symbolic-ref` (plumbing) กับ `checkout`/`switch` (porcelain)

กลับไปที่ master ให้เรียบร้อยก่อนไปต่อ:

```bash
git symbolic-ref HEAD refs/heads/master
git branch -d new-default-branch
```

### ใช้ทำอะไรได้บ้างในทางปฏิบัติ

1. **เปลี่ยนชื่อ default branch ของ repository ใหม่** (เช่นตอนที่วงการเปลี่ยนจาก `master` เป็น `main`) เครื่องมือ hosting อย่าง GitHub/GitLab ใช้กลไกคล้ายกันนี้ภายในเพื่อเปลี่ยน default branch โดยไม่ต้องแตะไฟล์ในโปรเจกต์เลย
2. **เขียนสคริปต์ที่ต้องการสลับ branch แบบ "เบา" ๆ** โดยไม่กระทบ working directory เช่น เครื่องมือจัดการ CI ที่ต้องการรู้ว่า HEAD ชี้ไปที่ branch ไหนโดยไม่ต้อง checkout จริง
3. **ซ่อมแซม repository ที่ `.git/HEAD` เสียหาย** เช่นถ้ามีคนแก้ไฟล์ `.git/HEAD` ผิดพลาดจนหน้าตาแปลกไป `symbolic-ref` ช่วยตั้งค่าใหม่ให้ถูกต้องได้อย่างปลอดภัย

### สรุปรูปแบบคำสั่งของ `symbolic-ref`

| คำสั่ง | ความหมาย |
|---|---|
| `git symbolic-ref HEAD` | อ่านว่า HEAD ชี้ไปยัง ref ไหน (error ถ้าอยู่ใน detached HEAD) |
| `git symbolic-ref HEAD <ref>` | ตั้งให้ HEAD ชี้ไปยัง ref ที่ระบุ (ไม่แตะ working directory) |
| `git symbolic-ref -d HEAD` | ลบ symbolic ref ทิ้ง (แทบไม่ใช้ในทางปฏิบัติ เพราะ HEAD ต้องมีอยู่เสมอ) |
| `git symbolic-ref --short HEAD` | อ่านค่าแบบตัด prefix `refs/heads/` ออกให้ (ได้ผลคล้าย `rev-parse --abbrev-ref HEAD`) |

---

## Step 569: ทำไมต้องรู้ plumbing commands

หลายคนอาจสงสัยว่า "ในชีวิตจริงแทบไม่มีใครพิมพ์ `git hash-object` หรือ `git commit-tree` เองเลย แล้วจะเรียนไปทำไม" — คำตอบมีอยู่ 3 เหตุผลหลักที่สำคัญมาก

### 1. Debug ปัญหาซับซ้อนที่ porcelain commands อธิบายไม่หมด

เมื่อเจอปัญหาประหลาดของ Git เช่น repository เสียหาย, ref หาย, หรือ merge conflict ที่ดูไม่เข้าใจ การรู้จัก plumbing commands ทำให้คุณสามารถ **"ผ่าตัด" ดูข้างในระบบได้โดยตรง** แทนที่จะเดาสุ่มไปเรื่อย ๆ

ตัวอย่างสถานการณ์จริง: ถ้ามีคนบอกว่า "ผม push โค้ดไปแล้วแต่หายไปจากประวัติ" คุณสามารถใช้ `git cat-file -p` ไล่ตรวจดู commit object ทีละอันเพื่อดูว่า tree ที่ผูกกับ commit นั้นมีไฟล์ที่ถูกต้องจริงหรือไม่ หรือใช้ `git rev-parse` เพื่อดูว่า ref ต่าง ๆ ชี้ไปที่ commit ใดกันแน่ในขณะนั้น

### 2. เขียน tool ต่อยอด Git ได้เอง

เครื่องมือดัง ๆ จำนวนมากที่สร้างขึ้นมาต่อยอดจาก Git (GUI clients, code review tools, deployment scripts, custom hooks) ล้วนใช้ plumbing commands เป็นฐานในการทำงาน เพราะ:

- Output ของ plumbing เสถียรและ parse ได้ง่ายกว่า porcelain มาก
- ทำงานได้เร็วกว่า เพราะไม่มี overhead ของการจัดรูปแบบสำหรับมนุษย์
- ควบคุมได้ละเอียดกว่า เพราะทำทีละขั้นตอนเดียว ไม่รวมหลายอย่างเข้าด้วยกันแบบ porcelain

ตัวอย่างเช่น ถ้าคุณต้องการเขียนสคริปต์ที่ตรวจสอบว่ามีไฟล์ที่มีขนาดใหญ่ผิดปกติแอบเข้ามาใน commit ก่อนจะยอมให้ push (pre-receive hook บน server) คุณจะใช้ `git cat-file --batch-check` และ `git ls-tree -r` เป็นเครื่องมือหลักในการไล่ตรวจ ไม่ใช่ `git log` หรือ `git show`

### 3. เข้าใจ Git อย่างลึกซึ้งจริง ๆ ไม่ใช่แค่จำคำสั่งได้

นี่คือเหตุผลที่สำคัญที่สุดในเชิงการเรียนรู้ เมื่อคุณเข้าใจว่า:

- `git add` = `hash-object -w` + ผูกเข้า staging area
- `git commit` = `write-tree` + `commit-tree` + `update-ref`
- `git branch <name>` = `update-ref refs/heads/<name> HEAD`
- `git checkout <branch>` = `symbolic-ref HEAD refs/heads/<branch>` + อัปเดต working directory ให้ตรงกับ tree ของ commit นั้น

...คุณจะไม่มีวัน "งง" กับพฤติกรรมของ Git อีกต่อไป เพราะทุกอย่างล้วนเป็นการประกอบร่างของ operation ง่าย ๆ เพียงไม่กี่แบบเท่านั้น — ความรู้สึก "Git มันมีเวทมนตร์อะไรบางอย่างที่ผมไม่เข้าใจ" จะหายไปโดยสิ้นเชิง

### เปรียบเทียบให้เห็นภาพชัด

ลองเปรียบเทียบกับการเรียนขับรถ:

> **Porcelain commands** เหมือนกับการรู้วิธีเหยียบคันเร่ง เบรก และหมุนพวงมาลัย — พอสำหรับขับรถได้ในชีวิตประจำวัน
>
> **Plumbing commands** เหมือนกับการรู้ว่าเครื่องยนต์ทำงานอย่างไร น้ำมันไหลไปที่ไหน ระบบเบรกทำงานผ่านกลไกอะไร — ไม่จำเป็นสำหรับการขับรถทั่วไป แต่จำเป็นมากถ้าอยากซ่อมรถเองได้ หรืออยากเป็นวิศวกรยานยนต์

โปรแกรมเมอร์ที่เข้าใจ Git plumbing ลึกซึ้งจะสามารถแก้ปัญหาที่คนอื่นแก้ไม่ได้ เพราะพวกเขาไม่ได้จำแค่ "สูตรสำเร็จ" ของคำสั่ง แต่เข้าใจกลไกจริง ๆ ที่อยู่ข้างใต้

---

## Step 570: แบบฝึกหัด — สร้าง commit ทั้งอันด้วย plumbing command ล้วน ๆ

ถึงเวลาพิสูจน์ความเข้าใจแล้ว โจทย์ของแบบฝึกหัดนี้คือ:

> **สร้าง commit ใหม่ทั้งอันตั้งแต่ต้นจนจบ โดยใช้ plumbing commands เท่านั้น ห้ามใช้ `git add` หรือ `git commit` แม้แต่ครั้งเดียว**

### เตรียม repository ใหม่สำหรับแบบฝึกหัด

```bash
mkdir ~/git-course/part-57-exercise
cd ~/git-course/part-57-exercise
git init
git config user.name "Somchai Devteam"
git config user.email "somchai@example.com"
```

### โจทย์: สร้าง commit แรกของ repository (root commit) ที่มีไฟล์ 2 ไฟล์ ด้วยมือล้วน ๆ

ไฟล์ที่ต้องการ:

- `README.md` เนื้อหา `# โปรเจกต์ทดลอง Plumbing`
- `main.py` เนื้อหา `print("Hello from pure plumbing")`

### ขั้นตอนที่ 1: สร้างเนื้อหาไฟล์และคำนวณ blob hash ด้วย `hash-object -w`

```bash
echo -n "# โปรเจกต์ทดลอง Plumbing" | git hash-object -w --stdin
```

```
7a1c4e8b2d5f9a3c6e0b7d4a1f8c5e2b9d6a3f0c
```

```bash
echo -n 'print("Hello from pure plumbing")' | git hash-object -w --stdin
```

```
4f8b1e6d9c2a5f8b1e4d7a0c3f6b9e2d5a8c1f4b
```

เก็บค่าไว้ในตัวแปรเพื่อใช้ต่อ:

```bash
README_BLOB=$(echo -n "# โปรเจกต์ทดลอง Plumbing" | git hash-object -w --stdin)
MAIN_BLOB=$(echo -n 'print("Hello from pure plumbing")' | git hash-object -w --stdin)
echo "README blob: $README_BLOB"
echo "main.py blob: $MAIN_BLOB"
```

### ขั้นตอนที่ 2: ตรวจสอบว่า blob ถูกเขียนจริงด้วย `cat-file`

```bash
git cat-file -p $README_BLOB
git cat-file -p $MAIN_BLOB
```

```
# โปรเจกต์ทดลอง Plumbing
print("Hello from pure plumbing")
```

### ขั้นตอนที่ 3: ผูก blob เข้ากับชื่อไฟล์โดยไม่ใช้ `git add`

นี่คือจุดสำคัญที่สุดของโจทย์นี้ — เราต้องหลีกเลี่ยง `git add` ให้ได้ ซึ่งเราจะใช้ plumbing command อีกตัวชื่อ **`git update-index`** (เป็นน้องของ `hash-object`/`write-tree` ที่ทำหน้าที่แก้ staging area โดยตรงแบบ low-level) แทน:

```bash
git update-index --add --cacheinfo 100644,$README_BLOB,README.md
git update-index --add --cacheinfo 100644,$MAIN_BLOB,main.py
```

คำอธิบาย: `--cacheinfo <mode>,<hash>,<path>` คือการบอก Git ตรง ๆ ว่า "ให้เพิ่ม entry ใน staging area ที่มี mode, hash และ path ตามนี้" โดยไม่ต้องอ่านไฟล์จาก working directory เลยด้วยซ้ำ (สังเกตว่าเรายังไม่เคยสร้างไฟล์ `README.md` หรือ `main.py` ใน working directory จริง ๆ เลย — เราสร้างแค่ blob object ในฐานข้อมูล Git เท่านั้น)

ตรวจสอบ staging area ด้วยคำสั่ง plumbing อีกตัว:

```bash
git ls-files --stage
```

```
100644 7a1c4e8b2d5f9a3c6e0b7d4a1f8c5e2b9d6a3f0c 0    README.md
100644 4f8b1e6d9c2a5f8b1e4d7a0c3f6b9e2d5a8c1f4b 0    main.py
```

### ขั้นตอนที่ 4: สร้าง tree object ด้วย `write-tree`

```bash
TREE=$(git write-tree)
echo "Tree hash: $TREE"
```

```
Tree hash: 2d9c6a3f0e7b4d1a8f5c2e9b6d3a0f7c4e1b8d5a
```

ตรวจสอบด้วย `ls-tree`:

```bash
git ls-tree $TREE
```

```
100644 blob 4f8b1e6d9c2a5f8b1e4d7a0c3f6b9e2d5a8c1f4b    main.py
100644 blob 7a1c4e8b2d5f9a3c6e0b7d4a1f8c5e2b9d6a3f0c    README.md
```

### ขั้นตอนที่ 5: สร้าง commit object ด้วย `commit-tree`

เนื่องจากนี่คือ commit แรกของ repository จึงไม่มี parent — **ห้ามใส่ flag `-p` เลย**:

```bash
COMMIT=$(git commit-tree $TREE -m "root commit สร้างด้วย plumbing commands ล้วน ๆ")
echo "Commit hash: $COMMIT"
```

```
Commit hash: 6e3b9d1c8f5a2e7b4d0c9f6a3e1d8b5c2f9a6e3d
```

ตรวจสอบด้วย `cat-file -p`:

```bash
git cat-file -p $COMMIT
```

```
tree 2d9c6a3f0e7b4d1a8f5c2e9b6d3a0f7c4e1b8d5a
author Somchai Devteam <somchai@example.com> 1719400500 +0700
committer Somchai Devteam <somchai@example.com> 1719400500 +0700

root commit สร้างด้วย plumbing commands ล้วน ๆ
```

สังเกตว่าไม่มีบรรทัด `parent` เลย ตรงตามที่คาดไว้สำหรับ root commit

### ขั้นตอนที่ 6: ทำให้ branch `master` ชี้ไปยัง commit นี้ด้วย `update-ref`

```bash
git update-ref refs/heads/master $COMMIT
```

### ขั้นตอนที่ 7: ตั้งให้ `HEAD` ชี้ไปยัง `master` ด้วย `symbolic-ref` (เผื่อกรณีที่ยังไม่เคยมี branch มาก่อน)

ในกรณีของแบบฝึกหัดนี้ `git init` ได้ตั้ง `HEAD` ให้ชี้ไปยัง `refs/heads/master` ไว้ล่วงหน้าแล้วโดยอัตโนมัติ (แม้ตอนนั้น branch จะยังไม่มีอยู่จริงก็ตาม เพราะยังไม่มี commit ใดเลย) แต่เพื่อความสมบูรณ์ ลองตรวจสอบและยืนยันด้วย `symbolic-ref`:

```bash
git symbolic-ref HEAD
```

```
refs/heads/master
```

ถ้าต้องการยืนยัน/ตั้งค่าใหม่อย่างชัดเจนด้วยมือ (ในกรณีนี้ค่าจะเหมือนเดิม แต่เป็นการฝึกใช้คำสั่งให้ครบ):

```bash
git symbolic-ref HEAD refs/heads/master
```

### ขั้นตอนที่ 8: ทำให้ working directory ตรงกับ snapshot ที่สร้างไว้

เนื่องจากเราไม่เคยสร้างไฟล์จริงใน working directory เลยตลอดกระบวนการนี้ (เราทำงานผ่าน object database และ staging area ล้วน ๆ) ให้ใช้ `git checkout-index` ซึ่งเป็น plumbing command สำหรับ "เขียนไฟล์จาก staging area ลงสู่ working directory จริง":

```bash
git checkout-index -a
```

flag `-a` หมายถึง "ทุกไฟล์ใน staging area" ตรวจสอบผล:

```bash
ls -la
cat README.md
cat main.py
```

```
README.md
main.py
# โปรเจกต์ทดลอง Plumbing
print("Hello from pure plumbing")
```

### ขั้นตอนที่ 9: ตรวจสอบผลลัพธ์สุดท้ายด้วยคำสั่ง porcelain

ตอนนี้ให้สลับมาใช้คำสั่ง porcelain เพื่อยืนยันว่าทุกอย่างที่เราสร้างด้วยมือถูกต้องสมบูรณ์:

```bash
git log --oneline
```

```
6e3b9d1 root commit สร้างด้วย plumbing commands ล้วน ๆ
```

```bash
git status
```

```
On branch master
nothing to commit, working tree clean
```

```bash
git show --stat HEAD
```

```
commit 6e3b9d1c8f5a2e7b4d0c9f6a3e1d8b5c2f9a6e3d
Author: Somchai Devteam <somchai@example.com>
Date:   ...

    root commit สร้างด้วย plumbing commands ล้วน ๆ

 README.md | 1 +
 main.py   | 1 +
 2 files changed, 2 insertions(+)
```

ถ้าคุณเห็นผลลัพธ์แบบนี้ แปลว่า**คุณเพิ่งสร้าง commit ที่สมบูรณ์แบบทั้งอันด้วยมือ โดยไม่พึ่ง `git add` หรือ `git commit` แม้แต่ครั้งเดียว** — นี่คือหลักฐานที่ชัดเจนที่สุดว่าคุณเข้าใจกลไกภายในของ Git อย่างแท้จริงแล้ว ไม่ใช่แค่จำคำสั่งได้

### ตารางสรุปคำสั่ง plumbing ที่ใช้ในแบบฝึกหัดนี้ เทียบกับสิ่งที่ porcelain ทำแทน

| Plumbing command ที่ใช้ | หน้าที่ | Porcelain ที่ทำแทนได้ |
|---|---|---|
| `git hash-object -w --stdin` | สร้าง blob object จากเนื้อหา | ส่วนหนึ่งของ `git add` |
| `git update-index --add --cacheinfo` | ผูก blob เข้ากับชื่อไฟล์ใน staging area | ส่วนหนึ่งของ `git add` |
| `git write-tree` | แปลง staging area เป็น tree object | ส่วนหนึ่งของ `git commit` |
| `git commit-tree` | สร้าง commit object | ส่วนหนึ่งของ `git commit` |
| `git update-ref` | ทำให้ branch ชี้ไปยัง commit ใหม่ | ส่วนหนึ่งของ `git commit` |
| `git symbolic-ref` | ตรวจสอบ/ตั้งค่าว่า HEAD ชี้ไปยัง branch ไหน | ส่วนหนึ่งของ `git init`/`git checkout` |
| `git checkout-index -a` | เขียนไฟล์จาก staging area ลง working directory | ส่วนหนึ่งของ `git checkout` |

### โจทย์เพิ่มเติมสำหรับฝึกฝนต่อ (ไม่บังคับ)

ถ้าต้องการฝึกให้แน่นยิ่งขึ้น ลองทำโจทย์เหล่านี้ต่อด้วยตัวเอง โดยยังคงกฎ "ห้ามใช้ `git add`/`git commit`" เช่นเดิม:

1. สร้าง **commit ที่สอง** ที่แก้ไขเนื้อหาของ `main.py` และเพิ่มไฟล์ใหม่ `utils.py` โดยต้องระบุ `-p <parent-hash>` ให้ถูกต้องตอนเรียก `commit-tree`
2. สร้างโครงสร้างที่มี **subdirectory** เช่น `src/app.py` โดยใช้ `--cacheinfo` กับ path ที่มี `/` อยู่ข้างใน แล้วดูว่า `write-tree` สร้าง nested tree ให้อัตโนมัติอย่างไร
3. ลองสร้าง **branch ที่สอง** ด้วย `update-ref` ให้ชี้ไปยัง commit แรก (ไม่ใช่ commit ล่าสุด) แล้วใช้ `symbolic-ref` สลับ `HEAD` ไปมาระหว่างสอง branch โดยสังเกตว่า working directory เปลี่ยนหรือไม่เปลี่ยนในแต่ละกรณี (คำใบ้: ต้องใช้ `checkout-index` เพิ่มเติมถ้าต้องการให้ working directory ตรงกับ branch ใหม่จริง ๆ)

---

## สรุป Part 57

ใน Part นี้เราได้เรียนรู้ว่า:

1. Git แบ่งคำสั่งออกเป็น 2 ชั้น คือ **Porcelain** (คำสั่งระดับสูงสำหรับมนุษย์ เช่น `add`, `commit`, `push`) และ **Plumbing** (คำสั่งระดับต่ำที่ทำงานกับ object database และ ref โดยตรง)
2. `git hash-object` คำนวณ SHA hash ของเนื้อหาไฟล์ตามสูตร `<type> <size>\0<content>` และเขียน blob object จริงได้ด้วย flag `-w`
3. `git cat-file` ใช้เปิดดูข้างใน object ได้ 3 มุมมองหลักคือ `-t` (type), `-p` (pretty-print เนื้อหา), `-s` (ขนาด)
4. `git ls-tree` แสดงเนื้อหาของ tree object เป็นรายการ mode/type/hash/name และรองรับโหมด recursive ด้วย `-r`
5. `git rev-parse` แปลง reference ทุกรูปแบบ (branch name, `HEAD~1`, hash แบบย่อ) ให้กลายเป็น full SHA hash และมี flag ที่มีประโยชน์อย่าง `--verify`, `--abbrev-ref`, `--show-toplevel`
6. `git update-ref` แก้ไข ref โดยตรงอย่างปลอดภัย พร้อมกลไก compare-and-swap ที่ป้องกัน race condition ซึ่งการแก้ไฟล์ ref ตรง ๆ ด้วยมือทำไม่ได้
7. `git write-tree` และ `git commit-tree` คือคู่คำสั่งหลักที่ใช้สร้าง tree object และ commit object ด้วยมือ ซึ่งรวมกันแล้วคือสิ่งที่ `git commit` ทำเบื้องหลังทั้งหมด
8. `git symbolic-ref` ใช้อ่านและแก้ไขว่า `HEAD` ชี้ไปยัง branch ใด โดยไม่แตะ working directory เลย ต่างจาก `checkout`/`switch`
9. การรู้ plumbing commands ช่วยให้ debug ปัญหาซับซ้อนได้, เขียนเครื่องมือต่อยอด Git ได้เอง, และเข้าใจ Git อย่างลึกซึ้งจริง ๆ ไม่ใช่แค่จำสูตรสำเร็จ
10. เราลงมือสร้าง commit ทั้งอันตั้งแต่ root commit จนถึงการเขียนไฟล์ลง working directory ด้วย plumbing commands ล้วน ๆ โดยไม่แตะ `git add`/`git commit` เลยแม้แต่ครั้งเดียว พิสูจน์ความเข้าใจกลไกภายในของ Git อย่างสมบูรณ์

**ต่อไป:** [Part 58: Packfiles และการจัดเก็บข้อมูลภายในของ Git](./part-058-packfiles.md)
