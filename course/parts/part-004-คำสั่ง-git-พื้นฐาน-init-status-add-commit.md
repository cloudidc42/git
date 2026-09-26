# Part 04: คำสั่ง Git พื้นฐานชุดแรก: init, status, add, commit

> **Step ในหลักสูตรนี้:** Step 31–40
> **เฟส:** 1 — ปูพื้นฐานความคิดเรื่อง Version Control และ Git
> **เป้าหมายของ Part นี้:** ลงมือใช้คำสั่ง Git ชุดแรกที่คุณจะพิมพ์บ่อยที่สุดในชีวิตการทำงานจริง ได้แก่ `git init`, `git status`, `git add`, `git commit` ให้คล่องแคล่ว เข้าใจทุกบรรทัดของผลลัพธ์ที่ Git แสดง เขียน commit message ที่มีคุณภาพ และฝึกทำ workflow แบบเต็มรอบที่จะใช้ซ้ำทุกวันตลอดการทำงานกับ Git

---

## สารบัญของ Part นี้

- Step 31: `git init` — สร้าง Repository ใหม่ และทำความเข้าใจว่าเกิดอะไรขึ้นเบื้องหลัง
- Step 32: `git status` — อ่านผลลัพธ์อย่างละเอียดทุกส่วน
- Step 33: `git add` — เพิ่มไฟล์เข้า Staging Area ในรูปแบบต่าง ๆ
- Step 34: `git add -p` — Patch Mode สำหรับเลือก Stage เฉพาะบางส่วนของไฟล์
- Step 35: `git commit` — ศิลปะการเขียน Commit Message ที่ดี
- Step 36: `git commit -m` vs เปิด Editor และทางลัด `git commit -am`
- Step 37: `git commit --amend` — แก้ไข Commit ล่าสุด
- Step 38: `git rm` และ `git mv` — ลบและย้าย/เปลี่ยนชื่อไฟล์อย่างถูกวิธี
- Step 39: Workflow แบบเต็มรอบที่ใช้ซ้ำทุกวัน
- Step 40: แบบฝึกหัดลงมือทำ — สร้างเว็บเพจง่าย ๆ พร้อม Commit อย่างมีความหมาย 5 ครั้ง

---

## Step 31: `git init` — สร้าง Repository ใหม่

### 31.1 `git init` ทำอะไรกันแน่

คำสั่ง `git init` คือจุดเริ่มต้นของทุกอย่าง มันจะเปลี่ยนโฟลเดอร์ธรรมดา ๆ ให้กลายเป็น **Git Repository** โดยการสร้างโฟลเดอร์ซ่อนชื่อ `.git` ขึ้นมาในโฟลเดอร์นั้น

> **สิ่งสำคัญที่ต้องเข้าใจ:** `.git` คือ "ที่เก็บ" ของ Repository ทั้งหมด — ประวัติ, commit, branch, config, object ทุกอย่างของ Git อยู่ในโฟลเดอร์นี้เพียงโฟลเดอร์เดียว ถ้าคุณลบ `.git` ทิ้ง โปรเจกต์ก็จะกลายเป็นโฟลเดอร์ธรรมดาที่ไม่มีประวัติใด ๆ เหลืออยู่เลย

### 31.2 ลองสร้างโปรเจกต์ทดลองตั้งแต่ต้น

มาเริ่มลงมือจริงกัน สร้างโฟลเดอร์ใหม่สำหรับฝึก Part นี้:

```bash
$ mkdir ~/git-course/part-04-practice
$ cd ~/git-course/part-04-practice
$ ls -la
```

ผลลัพธ์ (โฟลเดอร์ว่างเปล่า):

```
total 0
drwxr-xr-x  2 user user   64  1 ต.ค. 10:00 .
drwxr-xr-x 12 user user  384  1 ต.ค. 10:00 ..
```

ตอนนี้ยังไม่มีอะไรเป็น Git เลย ลองรัน `git status` ดูก่อนว่าจะเกิดอะไรขึ้น:

```bash
$ git status
```

```
fatal: not a git repository (or any of the parent directories): .git
```

Git บอกเราตรง ๆ ว่า **โฟลเดอร์นี้ยังไม่ใช่ Git Repository** — นี่แหละคือเหตุผลที่เราต้องรัน `git init` ก่อน:

```bash
$ git init
```

ผลลัพธ์ที่คาดหวัง:

```
Initialized empty Git repository in /home/user/git-course/part-04-practice/.git/
```

ลองดูว่ามีอะไรเกิดขึ้นบ้าง:

```bash
$ ls -la
```

```
total 12
drwxr-xr-x  3 user user   96  1 ต.ค. 10:01 .
drwxr-xr-x 12 user user  384  1 ต.ค. 10:00 ..
drwxr-xr-x  8 user user  256  1 ต.ค. 10:01 .git
```

เห็นโฟลเดอร์ `.git` เกิดขึ้นมาแล้ว มาดูข้างในกันว่ามีอะไรบ้าง:

```bash
$ ls -la .git
```

```
total 40
drwxr-xr-x  8 user user  256  1 ต.ค. 10:01 .
drwxr-xr-x  3 user user   96  1 ต.ค. 10:01 ..
-rw-r--r--  1 user user   23  1 ต.ค. 10:01 HEAD
-rw-r--r--  1 user user  137  1 ต.ค. 10:01 config
-rw-r--r--  1 user user   73  1 ต.ค. 10:01 description
drwxr-xr-x  2 user user   64  1 ต.ค. 10:01 hooks
drwxr-xr-x  2 user user   64  1 ต.ค. 10:01 info
drwxr-xr-x  4 user user  128  1 ต.ค. 10:01 objects
drwxr-xr-x  4 user user  128  1 ต.ค. 10:01 refs
```

### 31.3 ไฟล์และโฟลเดอร์สำคัญใน `.git`

| ไฟล์/โฟลเดอร์ | หน้าที่ |
|---|---|
| `HEAD` | ตัวชี้ว่าตอนนี้เรากำลังอยู่ที่ branch ไหน (เช่น ชี้ไปที่ `refs/heads/main`) |
| `config` | การตั้งค่าเฉพาะของ repository นี้ (แตกต่างจาก global config) |
| `description` | ใช้โดย GitWeb เท่านั้น ไม่ค่อยได้ใช้ในทางปฏิบัติ |
| `hooks/` | Script ที่จะรันอัตโนมัติเมื่อเกิดเหตุการณ์บางอย่าง เช่น ก่อน commit (เราจะเรียนละเอียดใน Part เกี่ยวกับ Git Hooks) |
| `info/` | เก็บ pattern การ exclude ไฟล์เฉพาะ repository นี้ |
| `objects/` | **หัวใจสำคัญที่สุด** — ที่เก็บข้อมูลจริงทั้งหมดของ Git ในรูปแบบ object (blob, tree, commit) |
| `refs/` | เก็บตัวชี้ไปยัง commit ต่าง ๆ เช่น branch และ tag |

เราจะยังไม่ต้องเข้าใจลึกเรื่อง `objects/` ตอนนี้ (จะเจาะลึกใน Part ที่ว่าด้วย Git Internals ภายหลัง) แค่ต้องรู้ว่า **ทุกอย่างที่ Git จำได้ ถูกเก็บอยู่ใน `.git` โฟลเดอร์นี้เพียงที่เดียว**

ลองรัน `git status` อีกครั้งดูว่าผลลัพธ์เปลี่ยนไปอย่างไร:

```bash
$ git status
```

```
On branch main

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

ตอนนี้ Git รู้จักโฟลเดอร์นี้แล้วว่าเป็น repository แต่ยังไม่มี commit ใด ๆ เลย (เราจะอธิบายผลลัพธ์นี้แบบละเอียดใน Step 32)

### 31.4 `git init` บนโปรเจกต์ที่มีไฟล์อยู่แล้ว

ในความเป็นจริง หลายครั้งคุณจะมีโฟลเดอร์โปรเจกต์ที่มีไฟล์อยู่แล้ว แล้วอยากเริ่มใช้ Git กับมัน ทำได้เหมือนกันทุกประการ:

```bash
$ mkdir my-existing-project
$ cd my-existing-project
$ echo "# My Project" > README.md
$ echo "console.log('hello');" > app.js
$ git init
```

```
Initialized empty Git repository in /home/user/my-existing-project/.git/
```

`git init` ไม่สนใจว่าโฟลเดอร์นั้นมีไฟล์อยู่ก่อนหรือไม่ — มันแค่สร้าง `.git` ขึ้นมาเฉย ๆ ไฟล์เดิมที่มีอยู่จะกลายเป็นสถานะ "untracked" (ยังไม่ถูก Git ติดตาม) ซึ่งเราจะเห็นชัดเจนใน Step ถัดไป

### 31.5 เรื่องชื่อ branch เริ่มต้น: `main` vs `master`

Git เวอร์ชันใหม่ (2.28 ขึ้นไป) เปลี่ยนชื่อ branch เริ่มต้นจาก `master` เป็น `main` ตามค่ามาตรฐานอุตสาหกรรมปัจจุบัน ถ้าคุณอยากกำหนดชื่อ branch เริ่มต้นเองแบบถาวรทุกครั้งที่ `git init` ทำได้ด้วย:

```bash
$ git config --global init.defaultBranch main
```

ถ้ายังไม่ได้ตั้งค่านี้ Git บางเวอร์ชันจะเตือนแบบนี้ตอน `git init`:

```
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint:
hint: 	git config --global init.defaultBranch <name>
hint:
Initialized empty Git repository in /home/user/my-existing-project/.git/
```

ไม่ต้องกังวลกับ hint นี้มากในตอนนี้ แค่รู้ไว้ว่ามันคืออะไร — เราตั้งค่านี้ไปแล้วใน Part 02 ตอนติดตั้ง Git

### 31.6 ข้อควรระวัง: อย่า `git init` ซ้อนกัน (Nested Repository)

ข้อผิดพลาดที่มือใหม่ทำบ่อยมากคือการรัน `git init` ในโฟลเดอร์ที่อยู่ **ข้างใน** Git repository ที่มีอยู่แล้ว เช่น:

```bash
$ cd ~/git-course/part-04-practice     # อยู่ใน repo ที่ init ไปแล้ว
$ mkdir subfolder
$ cd subfolder
$ git init                             # ผิดพลาด! ไม่ควรทำแบบนี้
```

การทำแบบนี้จะสร้าง Git repository ซ้อนกัน (nested repository) ซึ่งจะทำให้เกิดปัญหาที่เรียกว่า **submodule เถื่อน** (Git ภายนอกจะเห็น subfolder เป็น "gitlink" แปลก ๆ แทนที่จะเห็นไฟล์ข้างในตามปกติ) วิธีตรวจสอบก่อนว่าตัวเองอยู่ใน repository อยู่แล้วหรือไม่:

```bash
$ git rev-parse --is-inside-work-tree
```

ถ้าอยู่ใน repo อยู่แล้วจะได้ผลลัพธ์:

```
true
```

ถ้าไม่ได้อยู่ใน repo ใด ๆ จะได้ error:

```
fatal: not a git repository (or any of the parent directories): .git
```

### สรุป Step 31

- `git init` สร้างโฟลเดอร์ `.git` เพื่อเปลี่ยนโฟลเดอร์ธรรมดาให้เป็น Git repository
- ทุกอย่างที่ Git จำได้อยู่ใน `.git` เพียงที่เดียว ลบโฟลเดอร์นี้ = ลบประวัติทั้งหมด
- `git init` ใช้ได้ทั้งกับโฟลเดอร์ว่างเปล่าและโฟลเดอร์ที่มีไฟล์อยู่แล้ว
- ระวังอย่า `git init` ซ้อนกันข้างในอีก repository หนึ่ง

---

## Step 32: `git status` — อ่านผลลัพธ์อย่างละเอียดทุกส่วน

`git status` คือคำสั่งที่คุณจะพิมพ์บ่อยที่สุดในชีวิตการใช้ Git — บ่อยกว่า `git commit` เสียอีก มันคือ **"กระจกส่องสถานะ"** ของ working directory และ staging area ในทุกขณะ

### 32.1 สถานะที่ 1: Repository ใหม่ที่ยังไม่มี commit

ต่อจาก Step 31 ที่เรา `git init` ไว้ ลองรัน `git status`:

```bash
$ git status
```

```
On branch main

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

อ่านทีละบรรทัด:

| บรรทัด | ความหมาย |
|---|---|
| `On branch main` | ตอนนี้คุณอยู่บน branch ชื่อ `main` |
| `No commits yet` | Repository นี้ยังไม่เคยมีการ commit เลยแม้แต่ครั้งเดียว |
| `nothing to commit ...` | ไม่มีการเปลี่ยนแปลงใด ๆ ที่รอ commit อยู่ |

### 32.2 สถานะที่ 2: มีไฟล์ใหม่ (Untracked files)

สร้างไฟล์ใหม่ขึ้นมา:

```bash
$ echo "# Part 04 Practice" > README.md
$ git status
```

```
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	README.md

nothing added to commit but untracked files present (use "git add" to track)
```

**Untracked files** หมายถึงไฟล์ที่ **มีอยู่ในโฟลเดอร์จริง แต่ Git ยังไม่เคยรู้จัก/ติดตามมันเลย** — Git เห็นไฟล์นี้อยู่ แต่ยังไม่ถือว่ามันเป็นส่วนหนึ่งของ repository จนกว่าคุณจะสั่ง `git add`

### 32.3 สถานะที่ 3: ไฟล์อยู่ใน Staging Area (Changes to be committed)

ลอง add ไฟล์เข้าไป (รายละเอียดคำสั่ง `git add` เราจะเรียนเต็ม ๆ ใน Step 33):

```bash
$ git add README.md
$ git status
```

```
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   README.md

```

ส่วน **"Changes to be committed"** คือรายการไฟล์ที่อยู่ใน **Staging Area** แล้ว พร้อมที่จะถูก commit ในครั้งต่อไป สังเกตว่า Git บอกด้วยว่าถ้าอยาก unstage (เอาออกจาก staging area โดยไม่ลบไฟล์จริง) ให้ใช้ `git rm --cached`

### 32.4 สถานะที่ 4: มี commit แล้ว แล้วแก้ไขไฟล์ต่อ (Changes not staged)

ลอง commit ไฟล์แรกไปก่อน (จะอธิบายละเอียดใน Step 35-36):

```bash
$ git commit -m "docs: add initial README"
```

```
[main (root-commit) a1b2c3d] docs: add initial README
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

ทีนี้ลองแก้ไขไฟล์ที่ commit ไปแล้ว:

```bash
$ echo "เนื้อหาเพิ่มเติมของโปรเจกต์" >> README.md
$ git status
```

```
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
	modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

**"Changes not staged for commit"** หมายถึงไฟล์นี้ **ถูก Git ติดตามอยู่แล้ว (tracked)** และมีการแก้ไขในไฟล์จริง แต่การแก้ไขนั้นยัง **ไม่ถูกนำเข้า staging area** — ถ้าคุณ commit ตอนนี้ การเปลี่ยนแปลงนี้จะไม่ถูกบันทึกเข้าไปด้วย

### 32.5 สถานะที่ 5: มีทั้ง staged และ unstaged ในเวลาเดียวกัน

สถานการณ์ที่พบบ่อยมากในการทำงานจริง คือไฟล์เดียวกันถูกแก้ไขสองรอบ โดย stage ไปแค่รอบแรก:

```bash
$ git add README.md              # stage การแก้ไขครั้งที่ 1
$ echo "แก้ไขรอบที่ 2" >> README.md   # แก้ไขต่อ (ยังไม่ add)
$ git status
```

```
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   README.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   README.md

```

สังเกตว่า `README.md` ปรากฏอยู่ **ทั้งสองส่วนพร้อมกัน** — นี่ไม่ใช่ bug แต่เป็นเพราะ Git แยกแยะระหว่าง "เนื้อหาที่ถูก stage ไว้แล้ว" (snapshot ตอน `git add`) กับ "เนื้อหาปัจจุบันในไฟล์จริง" ซึ่งต่างกันอยู่ตอนนี้ ถ้าคุณ `git add README.md` อีกครั้ง ส่วน "Changes not staged" จะหายไป เพราะ staging area จะอัปเดตให้ตรงกับไฟล์ล่าสุด

> หมายเหตุ: Git เวอร์ชันใหม่แนะนำให้ใช้ `git restore` แทน `git checkout --` สำหรับการยกเลิกการแก้ไข ซึ่งเป็นคำสั่งที่ชัดเจนกว่า (เราจะเรียนละเอียดเรื่อง `git restore` ใน Part ถัดไปที่ว่าด้วยการย้อนกลับการเปลี่ยนแปลง)

### 32.6 ตารางสรุป 3 สถานะไฟล์หลักที่ต้องจำให้ขึ้นใจ

| สถานะ | ความหมาย | คำสั่งที่ทำให้เปลี่ยนสถานะ |
|---|---|---|
| **Untracked** | ไฟล์มีอยู่จริง แต่ Git ไม่รู้จักเลย | `git add` → ไปเป็น Staged |
| **Modified (not staged)** | ไฟล์ถูก track อยู่แล้ว และถูกแก้ไข แต่ยังไม่ add | `git add` → ไปเป็น Staged |
| **Staged (to be committed)** | พร้อมจะถูกบันทึกใน commit ถัดไป | `git commit` → กลายเป็นส่วนหนึ่งของประวัติ, หรือ `git restore --staged` → กลับไปเป็น Modified |

แผนภาพอย่างง่าย:

```
 Working Directory  --git add-->  Staging Area  --git commit-->  Repository (History)
  (Untracked/Modified)              (Staged)                       (Committed)
```

### 32.7 `git status -s` (Short Format)

เมื่อไฟล์เยอะขึ้น ผลลัพธ์แบบยาวจะอ่านยาก Git มีโหมดย่อที่กระชับกว่าคือ `-s` (short) หรือ `--short`:

```bash
$ git status -s
```

ตัวอย่างผลลัพธ์เมื่อมีไฟล์หลายสถานะพร้อมกัน:

```
 M README.md
MM style.css
A  index.html
?? logo.png
```

### 32.8 อ่านรหัสสถานะแบบย่อ

รูปแบบคือ 2 ช่องตัวอักษร: **`XY ชื่อไฟล์`** โดย `X` คือสถานะใน staging area และ `Y` คือสถานะใน working directory

| รหัส | ความหมาย |
|---|---|
| `??` | Untracked — ไฟล์ใหม่ที่ยังไม่ถูก track เลย |
| `A ` | Added — ไฟล์ใหม่ถูก stage แล้ว (staging = A, working directory ตรงกัน) |
| ` M` | Modified in working directory แต่ยังไม่ stage |
| `M ` | Modified และ stage แล้ว |
| `MM` | Modified, stage ไปรอบหนึ่งแล้ว แต่หลังจากนั้นแก้ไขต่ออีกรอบโดยยังไม่ได้ add รอบใหม่ |
| `D ` | Deleted และ stage แล้ว (ลบไฟล์และเตรียม commit การลบ) |
| ` D` | Deleted ในไฟล์จริง แต่ยังไม่ stage การลบนี้ |
| `R ` | Renamed — ไฟล์ถูกเปลี่ยนชื่อ (Git ตรวจจับให้อัตโนมัติ) |
| `C ` | Copied — ไฟล์ถูกคัดลอก (พบน้อย ต้องเปิดการตรวจจับเพิ่ม) |

ตัวอย่างเปรียบเทียบแบบเต็มกับแบบย่อในสถานการณ์เดียวกัน:

```bash
$ git status
```

```
On branch main
Changes to be committed:
	new file:   index.html

Changes not staged for commit:
	modified:   README.md

Untracked files:
	logo.png
```

```bash
$ git status -s
```

```
A  index.html
 M README.md
?? logo.png
```

### 32.9 ตัวเลือกเสริมที่มีประโยชน์

| คำสั่ง | ใช้ทำอะไร |
|---|---|
| `git status -sb` | Short format พร้อมแสดงชื่อ branch และสถานะเทียบกับ remote ในบรรทัดแรก |
| `git status --long` | บังคับให้แสดงแบบยาวเต็ม (ค่า default อยู่แล้ว ถ้าไม่ได้ตั้ง alias เป็นอื่น) |
| `git status --ignored` | แสดงไฟล์ที่ถูก `.gitignore` กันไว้ด้วย (ปกติจะไม่แสดง) |

ตัวอย่าง `git status -sb`:

```bash
$ git status -sb
```

```
## main
 M README.md
?? logo.png
```

### สรุป Step 32

- `git status` คือคำสั่งเช็คสถานะที่ใช้บ่อยที่สุด ควรพิมพ์ก่อน/หลังทุกการกระทำ
- ไฟล์มี 3 สถานะหลักที่ต้องแยกให้ออก: Untracked, Modified (not staged), Staged
- ไฟล์เดียวกันสามารถอยู่ทั้ง staged และ not-staged พร้อมกันได้ ถ้าแก้ไขซ้อนหลายรอบ
- `git status -s` ให้ผลลัพธ์กระชับ อ่านง่ายเมื่อไฟล์เยอะ ด้วยรหัส 2 ตัวอักษร (XY)

---

## Step 33: `git add` — เพิ่มไฟล์เข้า Staging Area

### 33.1 `git add` คืออะไร ทำไมต้องมี Staging Area

หลายคนสงสัยว่าทำไม Git ต้องมีขั้นตอน "stage" ก่อน commit ทำไมไม่ commit ไฟล์ที่แก้ไขไปเลย เหตุผลคือ **Staging Area ให้อำนาจคุณในการเลือกอย่างละเอียดว่า commit แต่ละครั้งจะมีอะไรอยู่ในนั้นบ้าง** แม้ว่าคุณจะแก้ไขไฟล์ 10 ไฟล์พร้อมกัน คุณสามารถเลือก commit แค่ 3 ไฟล์ที่เกี่ยวข้องกันในเรื่องเดียวกันก่อนได้ ทำให้ประวัติ Git อ่านง่ายและมีความหมายมากขึ้น

มาเตรียมไฟล์ทดลองกัน:

```bash
$ cd ~/git-course/part-04-practice
$ echo "function main() {}" > app.js
$ echo "body { margin: 0; }" > style.css
$ mkdir src
$ echo "export const version = '1.0.0';" > src/version.js
$ git status -s
```

```
 M README.md
?? app.js
?? src/
?? style.css
```

### 33.2 `git add` แบบเจาะจงไฟล์เดียว

วิธีที่ปลอดภัยและแนะนำที่สุดสำหรับมือใหม่ คือระบุชื่อไฟล์ตรง ๆ:

```bash
$ git add app.js
$ git status -s
```

```
A  app.js
 M README.md
?? src/
?? style.css
```

เพิ่มได้หลายไฟล์พร้อมกันในคำสั่งเดียวโดยคั่นด้วยช่องว่าง:

```bash
$ git add style.css README.md
$ git status -s
```

```
A  app.js
M  README.md
A  style.css
?? src/
```

### 33.3 `git add` แบบเจาะจงโฟลเดอร์

ถ้าระบุชื่อโฟลเดอร์ Git จะ add ไฟล์ทั้งหมดที่อยู่ข้างในโฟลเดอร์นั้นแบบ recursive (รวมโฟลเดอร์ย่อยข้างในด้วย):

```bash
$ git add src
$ git status -s
```

```
A  app.js
M  README.md
A  src/version.js
A  style.css
```

### 33.4 `git add .` — เพิ่มทุกอย่างจากตำแหน่งปัจจุบันลงไป

`git add .` จะ add ไฟล์ที่เปลี่ยนแปลง (ใหม่, แก้ไข, ลบ) ทั้งหมด **นับจากโฟลเดอร์ปัจจุบัน (`.`) ลงไปในโฟลเดอร์ย่อย** เท่านั้น — ถ้าคุณอยู่ในโฟลเดอร์ย่อย มันจะไม่แตะไฟล์ที่อยู่นอกโฟลเดอร์นั้น

ตัวอย่าง: สมมติเรากลับไปทำให้ทุกไฟล์เป็น untracked ใหม่ (unstage ทั้งหมดเพื่อสาธิต):

```bash
$ git restore --staged .
$ git status -s
```

```
 M README.md
?? app.js
?? src/
?? style.css
```

```bash
$ git add .
$ git status -s
```

```
M  README.md
A  app.js
A  src/version.js
A  style.css
```

### 33.5 `git add -A` — เพิ่มทุกอย่างในทั้ง Repository

`git add -A` (เท่ากับ `git add --all`) จะ add การเปลี่ยนแปลง **ทั้งหมดในทุกโฟลเดอร์ของ repository** ไม่ว่าคุณจะยืนอยู่ที่ตำแหน่งไหนก็ตาม รวมถึงไฟล์ที่ถูกลบด้วย

### 33.6 ความแตกต่างระหว่าง `git add .` กับ `git add -A`

นี่คือจุดที่คนสับสนกันมากที่สุด สรุปเป็นตาราง:

| คำสั่ง | ขอบเขตที่มีผล | รวมไฟล์ที่ถูกลบไหม |
|---|---|---|
| `git add .` | เฉพาะโฟลเดอร์ปัจจุบันและย่อยลงไปเท่านั้น | รวม (ใน Git เวอร์ชัน 2.x ขึ้นไป) |
| `git add -A` | ทั้ง repository ไม่ว่ายืนตรงไหน | รวม |
| `git add --update` (`-u`) | เฉพาะไฟล์ที่ Git track อยู่แล้วเท่านั้น (ไม่ add ไฟล์ใหม่ที่ยัง untracked) | รวม |

> **หมายเหตุสำคัญด้านประวัติศาสตร์:** ใน Git เวอร์ชันเก่ามาก ๆ (ก่อน 2.0) `git add .` จะไม่รวมไฟล์ที่ถูกลบ ต้องใช้ `-A` เท่านั้น แต่ตั้งแต่ Git 2.0 เป็นต้นมา พฤติกรรมนี้เปลี่ยนให้ `git add .` รวมไฟล์ที่ลบด้วยแล้ว ความต่างหลักที่เหลืออยู่ในปัจจุบันคือ **ขอบเขตของโฟลเดอร์** เท่านั้น

ตัวอย่างสาธิตความแตกต่างเรื่องขอบเขตโฟลเดอร์:

```bash
$ cd ~/git-course/part-04-practice/src
$ echo "export const author = 'me';" > author.js
$ echo "extra change" >> ../style.css
$ git status -s
```

```
?? src/author.js
 M style.css
```

ถ้าตอนนี้เรารัน `git add .` ขณะที่ยืนอยู่ใน `src/`:

```bash
$ git add .
$ git status -s
```

```
A  src/author.js
 M style.css
```

จะเห็นว่า `style.css` ที่อยู่นอกโฟลเดอร์ `src/` **ไม่ถูก add** เพราะเราสั่ง `git add .` จากข้างในโฟลเดอร์ `src/` ถ้าต้องการ add ทุกอย่างในทั้ง repo ไม่ว่าจะยืนอยู่ที่ไหน ให้ใช้ `git add -A` แทน:

```bash
$ git add -A
$ git status -s
```

```
A  src/author.js
M  style.css
```

### 33.7 `git add` แบบใช้ Pattern (Wildcard)

Git รองรับการใช้ shell glob pattern ในการเลือกไฟล์:

```bash
$ cd ~/git-course/part-04-practice
$ echo "console.log('utils');" > utils.js
$ echo "console.log('helper');" > helper.js
$ git add *.js
$ git status -s
```

```
A  app.js
A  helper.js
A  src/author.js
A  utils.js
M  style.css
```

ตัวอย่าง pattern อื่น ๆ ที่ใช้บ่อย:

| Pattern | ความหมาย |
|---|---|
| `git add *.js` | เพิ่มไฟล์ `.js` ทั้งหมดในโฟลเดอร์ปัจจุบัน (ไม่รวมโฟลเดอร์ย่อยถ้า shell ไม่รองรับ globstar) |
| `git add src/*.js` | เพิ่มไฟล์ `.js` เฉพาะในโฟลเดอร์ `src/` เท่านั้น |
| `git add docs/*.md` | เพิ่มไฟล์ Markdown ทั้งหมดในโฟลเดอร์ `docs/` |

> **ข้อควรระวัง:** pattern แบบ `*.js` เป็นการขยายผลโดย **shell** (bash/zsh) ไม่ใช่โดย Git เอง ดังนั้นพฤติกรรมอาจต่างกันเล็กน้อยขึ้นกับว่า shell ของคุณตั้งค่า `globstar` ไว้หรือไม่ (มีผลต่อว่า `**` จะไล่ลงโฟลเดอร์ย่อยหรือไม่)

### 33.8 ตารางสรุปคำสั่ง `git add` ทั้งหมด

| คำสั่ง | ผลลัพธ์ |
|---|---|
| `git add <file>` | เพิ่มไฟล์ที่ระบุเจาะจง |
| `git add <folder>` | เพิ่มไฟล์ทั้งหมดในโฟลเดอร์นั้นแบบ recursive |
| `git add .` | เพิ่มทุกอย่างจากตำแหน่งปัจจุบันลงไป |
| `git add -A` / `git add --all` | เพิ่มทุกอย่างในทั้ง repository |
| `git add -u` / `git add --update` | เพิ่มเฉพาะไฟล์ที่ track อยู่แล้ว (ไม่รวมไฟล์ใหม่) |
| `git add *.ext` | เพิ่มไฟล์ตาม pattern ที่กำหนด |
| `git add -p` | โหมด patch — เลือก stage เฉพาะบางส่วนของไฟล์ (Step 34) |

### 33.9 การ Unstage (เอาไฟล์ออกจาก Staging Area)

ถ้า add ผิดไฟล์ไป สามารถถอนออกได้โดยไม่กระทบไฟล์จริง:

```bash
$ git restore --staged helper.js
$ git status -s
```

```
A  app.js
?? helper.js
A  src/author.js
A  utils.js
M  style.css
```

(คำสั่งรุ่นเก่าก่อน Git 2.23 คือ `git reset HEAD helper.js` ซึ่งยังใช้งานได้อยู่ในปัจจุบัน แต่ Git แนะนำให้ใช้ `git restore --staged` ซึ่งสื่อความหมายชัดเจนกว่า)

### สรุป Step 33

- `git add` ย้ายไฟล์จาก working directory เข้าสู่ staging area
- `git add <file>` เจาะจงไฟล์ปลอดภัยที่สุดสำหรับมือใหม่
- `git add .` มีขอบเขตแค่โฟลเดอร์ปัจจุบันลงไป ส่วน `git add -A` มีผลทั้ง repository
- `git add *.js` ใช้ pattern เลือกไฟล์ตามนามสกุลหรือชื่อได้
- `git restore --staged <file>` ใช้ถอนไฟล์ออกจาก staging area โดยไม่ลบการแก้ไขจริง

---

## Step 34: `git add -p` — Patch Mode สำหรับเลือก Stage เฉพาะบางส่วน

### 34.1 ทำไมต้องมี Patch Mode

บางครั้งคุณแก้ไขไฟล์เดียวกันด้วย **สองเรื่องที่ไม่เกี่ยวข้องกัน** ในเวลาเดียวกัน เช่น แก้บั๊กหนึ่งจุด และเพิ่มฟีเจอร์อีกจุดในไฟล์เดียวกัน การ `git add` ทั้งไฟล์จะทำให้ commit นั้นปนกันทั้งสองเรื่อง ซึ่งไม่ดีต่อความสะอาดของประวัติ

`git add -p` (patch mode) ช่วยให้คุณเลือก stage ได้ **ทีละส่วน (hunk)** ของไฟล์ ไม่ใช่ทั้งไฟล์

### 34.2 เตรียมสถานการณ์ตัวอย่าง

สมมติเรามีไฟล์ `app.js` ที่ commit ไปแล้วก่อนหน้านี้ด้วยเนื้อหา:

```bash
$ cat app.js
```

```
function main() {}
```

ตอนนี้เราแก้ไขสองเรื่องพร้อมกันในไฟล์เดียว: เพิ่ม comment (ไม่เกี่ยวกับ logic) และแก้ function ให้ทำงานจริง (เป็น logic ใหม่):

```bash
$ cat > app.js << 'EOF'
// TODO: refactor this later
function main() {
  console.log("Application started");
}

function helper() {
  return true;
}
EOF
```

```bash
$ git diff app.js
```

```
diff --git a/app.js b/app.js
index e69de29..3f7a9c1 100644
--- a/app.js
+++ b/app.js
@@ -1 +1,8 @@
-function main() {}
+// TODO: refactor this later
+function main() {
+  console.log("Application started");
+}
+
+function helper() {
+  return true;
+}
```

### 34.3 เริ่มโหมด Patch

```bash
$ git add -p app.js
```

Git จะแบ่งการเปลี่ยนแปลงออกเป็น "hunk" แล้วถามทีละอันว่าจะ stage หรือไม่:

```
diff --git a/app.js b/app.js
index e69de29..3f7a9c1 100644
--- a/app.js
+++ b/app.js
@@ -1 +1,8 @@
-function main() {}
+// TODO: refactor this later
+function main() {
+  console.log("Application started");
+}
+
+function helper() {
+  return true;
+}
Stage this hunk [y,n,q,a,d,s,e,?]?
```

### 34.4 ความหมายของแต่ละตัวเลือก

เมื่อพิมพ์ `?` ที่ prompt Git จะอธิบายตัวเลือกทั้งหมดให้:

| ตัวเลือก | ความหมาย |
|---|---|
| `y` | Stage hunk นี้ (yes) |
| `n` | ไม่ stage hunk นี้ (no) ข้ามไปดู hunk ถัดไป |
| `q` | ออกจากโหมด patch ทันที ไม่ถามต่ออีก (quit) — hunk ที่ยังไม่ตอบจะไม่ถูก stage |
| `a` | Stage hunk นี้และ hunk ที่เหลือทั้งหมดในไฟล์นี้ (all) |
| `d` | ไม่ stage hunk นี้และ hunk ที่เหลือทั้งหมดในไฟล์นี้ (don't) |
| `s` | แบ่ง hunk นี้ให้เป็นชิ้นเล็กลงอีก (split) — ใช้เมื่อ hunk หนึ่งมีการเปลี่ยนแปลงหลายเรื่องปนกัน |
| `e` | แก้ไข hunk ด้วยตัวเองแบบ manual ก่อน stage (edit) — สำหรับกรณีขั้นสูงที่ต้องการควบคุมละเอียดสุด |
| `?` | แสดงคำอธิบายตัวเลือกทั้งหมด (help) |

### 34.5 ตัวอย่างการใช้ `s` (split) เพื่อแยกเปลี่ยนแปลงคนละเรื่องออกจากกัน

ในตัวอย่างข้างบน hunk เดียวรวมทั้ง comment และ function `helper()` ไว้ด้วยกัน ลองสั่ง `s` เพื่อแยก:

```
Stage this hunk [y,n,q,a,d,s,e,?]? s
```

ผลลัพธ์ Git จะพยายามแบ่งให้:

```
Split into 2 hunks.
@@ -1 +1,4 @@
-function main() {}
+// TODO: refactor this later
+function main() {
+  console.log("Application started");
+}
Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]?
```

ตอบ `y` เพื่อ stage ส่วนแรก (comment + แก้ function main):

```
Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]? y
```

Git จะแสดง hunk ที่สอง (ส่วน `helper()` ที่เป็นฟีเจอร์ใหม่แยกต่างหาก):

```
@@ -1,0 +5,4 @@
+
+function helper() {
+  return true;
+}
Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]?
```

ถ้ายังไม่อยากรวม `helper()` เข้า commit นี้ (เพราะยังไม่เสร็จ หรือเป็นคนละเรื่อง) ตอบ `n`:

```
Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]? n
```

### 34.6 ตรวจสอบผลลัพธ์

```bash
$ git status -s
```

```
MM app.js
```

สังเกตรหัส `MM` — แปลว่า `app.js` มีการเปลี่ยนแปลงถูก stage ไปบางส่วนแล้ว (`M` ตัวแรก) และยังมีการเปลี่ยนแปลงส่วนที่เหลือค้างอยู่ใน working directory ที่ยังไม่ stage (`M` ตัวที่สอง) — ตรงกับที่เราต้องการเป๊ะ ๆ

ดูว่าอะไรถูก stage ไว้จริง ๆ ด้วย `git diff --staged`:

```bash
$ git diff --staged
```

```
diff --git a/app.js b/app.js
index e69de29..8c3d2a1 100644
--- a/app.js
+++ b/app.js
@@ -1 +1,4 @@
-function main() {}
+// TODO: refactor this later
+function main() {
+  console.log("Application started");
+}
```

จะเห็นว่ามีแค่ส่วน comment และแก้ `main()` เท่านั้นที่ถูก stage ส่วน `helper()` ยังไม่ถูก stage ตามที่ตั้งใจ

### 34.7 คำสั่งพี่น้องที่ทำงานคล้ายกัน

| คำสั่ง | ใช้ทำอะไร |
|---|---|
| `git add -p` / `git add --patch` | เลือก stage บางส่วนของไฟล์ |
| `git restore -p <file>` | เลือกยกเลิกการเปลี่ยนแปลงเฉพาะบางส่วนของไฟล์ในภายหลัง |
| `git checkout -p <file>` | (คำสั่งเก่า) ทำหน้าที่คล้าย `git restore -p` |
| `git stash -p` | เลือก stash เฉพาะบางส่วนของการเปลี่ยนแปลง |

### สรุป Step 34

- `git add -p` ให้คุณ stage ไฟล์แบบละเอียดระดับ "hunk" ไม่ใช่ทั้งไฟล์
- ตัวเลือกหลักที่ต้องจำ: `y` (stage), `n` (ข้าม), `s` (แบ่งย่อย), `q` (ออก), `a`/`d` (ทำกับทุก hunk ที่เหลือ)
- มีประโยชน์มากเมื่อคุณแก้ไขหลายเรื่องปนกันในไฟล์เดียว แล้วอยากแยก commit ให้สะอาด
- ใช้ `git diff --staged` ตรวจสอบเสมอว่าสิ่งที่ stage ไว้ตรงกับที่ตั้งใจจริงหรือไม่

---

## Step 35: `git commit` — ศิลปะการเขียน Commit Message ที่ดี

### 35.1 ทำไม Commit Message ถึงสำคัญมาก

Commit message ไม่ใช่แค่ "หมายเหตุ" — มันคือ **เอกสารประวัติศาสตร์ถาวรของโปรเจกต์** ที่จะถูกอ่านโดยเพื่อนร่วมทีม, ตัวคุณเองในอีก 6 เดือนข้างหน้า, และเครื่องมืออัตโนมัติต่าง ๆ (เช่น เครื่องมือสร้าง changelog อัตโนมัติ) commit message ที่ดีช่วยให้:

- ทีมเข้าใจว่า "ทำไม" ถึงมีการเปลี่ยนแปลงนี้ (ไม่ใช่แค่ "อะไร" เปลี่ยน เพราะ diff บอก "อะไร" ได้อยู่แล้ว)
- ค้นหาประวัติได้ง่ายด้วย `git log --grep`
- ใช้ `git bisect` หาสาเหตุบั๊กได้แม่นยำขึ้น (เรียนใน Part หลัง ๆ)
- สร้าง release note / changelog แบบอัตโนมัติได้

### 35.2 โครงสร้างของ Commit Message ที่ดี: กฎ 50/72

รูปแบบมาตรฐานที่ใช้กันทั่วโลก (มาจากธรรมเนียมของ Linux Kernel และถูกนำมาใช้แทบทุกที่):

```
<subject line — ไม่เกิน 50 ตัวอักษร>

<เว้นบรรทัดว่าง 1 บรรทัด>

<body — อธิบายรายละเอียด แต่ละบรรทัดไม่เกิน 72 ตัวอักษร>
```

| ส่วน | กฎ |
|---|---|
| **Subject line** (บรรทัดแรก) | สั้น กระชับ ไม่เกิน ~50 ตัวอักษร ใช้ imperative mood (คำสั่ง เช่น "Add" ไม่ใช่ "Added" หรือ "Adding") ไม่ต้องมีจุดปิดท้าย |
| **บรรทัดว่าง** | ต้องมีเสมอถ้าจะเขียน body ต่อ (เครื่องมือหลายตัว เช่น `git log --oneline` จะใช้เฉพาะ subject line เท่านั้น) |
| **Body** | อธิบาย "ทำไม" ถึงเปลี่ยน ไม่ใช่แค่ "อะไร" เปลี่ยน แต่ละบรรทัดควรตัดไม่เกิน 72 ตัวอักษรเพื่อให้อ่านง่ายใน terminal ทุกขนาดหน้าจอ |

### 35.3 ทำไมต้องเป็น Imperative Mood

ลองอ่านประโยคนี้: **"If applied, this commit will _______"** แล้วเติมคำในช่องว่างด้วย subject line ของคุณ

- ✅ "If applied, this commit will **fix login timeout bug**" → อ่านลื่น เป็น imperative mood ที่ถูกต้อง
- ❌ "If applied, this commit will **fixed login timeout bug**" → อ่านแล้วขัด ไม่ใช่ imperative

### 35.4 ตัวอย่าง Commit Message ที่ดี

```
Fix null pointer exception in user login flow

The login handler crashed when the session token was expired
because the code accessed user.profile before checking
whether the session was still valid.

This adds a null check before accessing user.profile and
returns a proper 401 response instead of crashing the server.

Fixes #142
```

จุดที่ดีของตัวอย่างนี้:
- Subject line สั้น กระชับ บอกว่าทำอะไร (ไม่ใช่ "fixed bug" ที่คลุมเครือ)
- Body อธิบาย **สาเหตุของปัญหา** และ **วิธีแก้** ไม่ใช่แค่พูดซ้ำสิ่งที่ diff บอกอยู่แล้ว
- อ้างอิง issue number ท้ายสุด (เชื่อมกับระบบ tracking บน GitHub/GitLab)

### 35.5 ตัวอย่าง Commit Message ที่ไม่ดี (และทำไมถึงไม่ดี)

| Commit message แย่ ๆ | ปัญหาคืออะไร |
|---|---|
| `fix bug` | ไม่บอกว่าบั๊กอะไร แก้ตรงไหน อ่านย้อนหลังไม่รู้เรื่องเลย |
| `update` | คลุมเครือที่สุด ไม่มีข้อมูลอะไรเลย |
| `asdasd` | ไม่มีความหมายใด ๆ (พบได้บ่อยตอนรีบ ๆ commit) |
| `Fixed the thing that was broken yesterday when John reported it in Slack` | ยาวเกินไปสำหรับ subject line เดียว (ควรตัดเป็น subject สั้น + body อธิบายรายละเอียด) |
| `WIP` | เหมาะกับ commit ชั่วคราวใน branch ส่วนตัวเท่านั้น ไม่ควรอยู่ใน branch หลัก |
| `Fix bug, update docs, refactor auth, add tests` | รวมหลายเรื่องในหนึ่ง commit — ควรแยกเป็นหลาย commit ต่างหาก |

### 35.6 แนวทาง Conventional Commits (แนวปฏิบัติที่นิยมในหลายทีม)

หลายทีม/โปรเจกต์ Open Source นิยมใช้รูปแบบ prefix แบบมาตรฐานเรียกว่า **Conventional Commits** เพื่อให้ subject line สื่อสารประเภทของการเปลี่ยนแปลงชัดเจนยิ่งขึ้น (เราจะเรียนละเอียดเรื่องนี้ใน Part ที่ว่าด้วย Workflow ของทีม แต่ขอแนะนำไว้ตั้งแต่ตอนนี้เพราะเป็นนิสัยที่ดีตั้งแต่ต้น):

| Prefix | ใช้เมื่อ |
|---|---|
| `feat:` | เพิ่มฟีเจอร์ใหม่ |
| `fix:` | แก้บั๊ก |
| `docs:` | แก้ไขเอกสารเท่านั้น ไม่กระทบโค้ด |
| `style:` | แก้ formatting, whitespace ไม่กระทบ logic |
| `refactor:` | ปรับโครงสร้างโค้ดโดยไม่เปลี่ยนพฤติกรรม |
| `test:` | เพิ่มหรือแก้ไข test |
| `chore:` | งานเบ็ดเตล็ด เช่น อัปเดต dependency |

ตัวอย่าง: `feat: add dark mode toggle to settings page`

### สรุป Step 35

- Commit message คืออธิบาย "ทำไม" ไม่ใช่แค่ "อะไร" เปลี่ยน
- ใช้กฎ 50/72: subject ไม่เกิน 50 ตัวอักษร, body แต่ละบรรทัดไม่เกิน 72 ตัวอักษร
- เขียน subject line แบบ imperative mood ("Add", "Fix" ไม่ใช่ "Added", "Fixed")
- หนึ่ง commit ควรมีเรื่องเดียว อย่ารวมหลายเรื่องปนกัน
- พิจารณาใช้ Conventional Commits prefix เพื่อความชัดเจนยิ่งขึ้น

---

## Step 36: `git commit -m` vs เปิด Editor และทางลัด `git commit -am`

### 36.1 `git commit -m "..."` — Commit แบบบรรทัดเดียวผ่าน command line

วิธีที่ใช้บ่อยที่สุดสำหรับ commit message สั้น ๆ:

```bash
$ git add app.js
$ git commit -m "fix: prevent crash when session token is null"
```

```
[main 7f3e9a2] fix: prevent crash when session token is null
 1 file changed, 4 insertions(+), 1 deletion(-)
```

ผลลัพธ์บอก 3 อย่าง: ชื่อ branch (`main`), hash ย่อของ commit ใหม่ (`7f3e9a2`), subject line, และสถิติไฟล์ที่เปลี่ยนแปลง

### 36.2 `git commit` (ไม่ใส่ `-m`) — เปิด Text Editor เพื่อเขียนหลายบรรทัด

เมื่อคุณต้องการเขียน commit message ที่มีทั้ง subject และ body (ตามกฎ 50/72 ใน Step 35) วิธีที่สะดวกกว่าคือไม่ใส่ `-m` เลย:

```bash
$ git add .
$ git commit
```

Git จะเปิด **text editor เริ่มต้น** ที่คุณตั้งค่าไว้ (ปกติคือ Vim หรือ Nano ถ้าไม่ได้ตั้งค่าอื่น — เราตั้งค่าใน Part 02 ไปแล้วว่าจะใช้ editor ตัวไหน) ขึ้นมาให้เขียน:

```
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch main
# Changes to be committed:
#	modified:   app.js
#	modified:   style.css
#
```

คุณพิมพ์เนื้อหาไว้เหนือส่วนที่เป็น comment (บรรทัดที่ขึ้นต้นด้วย `#`) เช่น:

```
feat: add responsive navigation menu

Add a hamburger menu for mobile screens under 768px width.
The menu was previously unusable on small screens because
it overflowed the viewport.

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch main
# Changes to be committed:
#	modified:   app.js
#	modified:   style.css
#
```

บันทึกแล้วปิด editor (ใน Vim คือกด `Esc` แล้วพิมพ์ `:wq` แล้ว Enter, ใน Nano คือกด `Ctrl+O` แล้ว `Ctrl+X`) Git จะนำข้อความที่ไม่ใช่ comment มาทำเป็น commit message:

```
[main 9c1d4f8] feat: add responsive navigation menu
 2 files changed, 15 insertions(+), 2 deletions(-)
```

### 36.3 ถ้าอยากยกเลิกการ commit ระหว่างเปิด editor อยู่

ลบข้อความทั้งหมดในไฟล์ให้เหลือแต่บรรทัด comment (หรือปล่อยว่างเปล่าทั้งไฟล์) แล้วบันทึกออกมา Git จะยกเลิกการ commit ให้:

```
Aborting commit due to empty commit message.
```

ไฟล์ที่ stage ไว้จะยังคงอยู่ใน staging area เหมือนเดิม ไม่มีอะไรหาย

### 36.4 `git commit -am "..."` — ทางลัดสุดฮิต

`-a` (all) จะสั่งให้ Git **add ไฟล์ที่ Git track อยู่แล้วโดยอัตโนมัติ** ก่อน commit ทันที ไม่ต้องพิมพ์ `git add` แยกต่างหาก:

```bash
$ echo "更新版本" >> README.md
$ git commit -am "docs: update README with version info"
```

```
[main 3a8b2c1] docs: update README with version info
 1 file changed, 1 insertion(+)
```

พูดง่าย ๆ `git commit -am "message"` เท่ากับการรัน `git add -u` แล้วตามด้วย `git commit -m "message"` ในคำสั่งเดียว

### 36.5 ⚠️ ข้อควรระวังสำคัญของ `-a`: ไม่รวมไฟล์ Untracked

นี่คือกับดักที่มือใหม่ (และบางทีมือเก่า) พลาดบ่อยมาก — `-a` จะ add **เฉพาะไฟล์ที่ Git เคย track อยู่แล้ว** เท่านั้น (คือไฟล์ที่เคยถูก `git add` และ commit ไปแล้วอย่างน้อยหนึ่งครั้ง) **มันจะไม่แตะไฟล์ใหม่ที่ยังเป็นสถานะ Untracked เลย**

ตัวอย่างสาธิตปัญหานี้:

```bash
$ echo "console.log('new feature');" > feature.js
$ echo "existing content updated" >> app.js
$ git status -s
```

```
 M app.js
?? feature.js
```

```bash
$ git commit -am "feat: add new feature module"
```

```
[main 5d7e1a9] feat: add new feature module
 1 file changed, 1 insertion(+)
```

ลอง `git status` ดูอีกครั้ง:

```bash
$ git status -s
```

```
?? feature.js
```

**`feature.js` ยังคงเป็น Untracked อยู่!** ทั้งที่ commit message บอกว่า "add new feature module" แต่ไฟล์จริงของฟีเจอร์นั้นไม่ได้ถูก commit ไปด้วยเลย นี่คือเหตุผลที่ต้อง **เช็ค `git status` ก่อน commit เสมอ** และระวังเป็นพิเศษเมื่อใช้ `-am`

### 36.6 ตารางเปรียบเทียบ

| คำสั่ง | ต้อง `git add` มาก่อนไหม | เปิด editor ไหม | รวมไฟล์ untracked ไหม |
|---|---|---|---|
| `git commit -m "..."` | ต้อง | ไม่ | ไม่เกี่ยว (ใช้เฉพาะที่ stage ไว้แล้ว) |
| `git commit` | ต้อง | เปิด | ไม่เกี่ยว |
| `git commit -am "..."` | ไม่ต้อง (แต่ต้องเป็นไฟล์ที่ track อยู่แล้ว) | ไม่ | **ไม่รวม** |
| `git commit -a` | ไม่ต้อง | เปิด | **ไม่รวม** |

### สรุป Step 36

- `git commit -m "..."` เหมาะกับ commit message สั้น ๆ บรรทัดเดียว
- `git commit` (ไม่มี `-m`) เปิด editor ให้เขียน subject + body ตามกฎ 50/72
- `git commit -am` เป็นทางลัดที่สะดวก แต่ **ไม่รวมไฟล์ Untracked** — ต้องเช็ค `git status` ก่อนเสมอเพื่อไม่ให้ลืมไฟล์ใหม่

---

## Step 37: `git commit --amend` — แก้ไข Commit ล่าสุด

### 37.1 ใช้ทำอะไรได้บ้าง

`git commit --amend` ใช้แก้ไข **commit ล่าสุดสุด (HEAD)** เท่านั้น ทำได้ 2 อย่างหลัก:

1. แก้ไขข้อความ commit message ที่พิมพ์ผิดหรืออยากปรับปรุง
2. เพิ่มไฟล์ที่ลืม add เข้าไปรวมกับ commit เดิม (แทนที่จะสร้าง commit ใหม่แยกต่างหากสำหรับไฟล์ที่ลืม)

### 37.2 กรณีที่ 1: แก้ไขแค่ข้อความ commit message

สมมติเรา commit ไปแล้วแต่พิมพ์ผิด:

```bash
$ git commit -m "fix: preventt crash on null token"
```

```
[main 2b4f8e1] fix: preventt crash on null token
 1 file changed, 3 insertions(+)
```

แก้ไขข้อความโดยไม่ต้องแตะไฟล์เลย:

```bash
$ git commit --amend -m "fix: prevent crash on null token"
```

```
[main a1c9d3f] fix: prevent crash on null token
 Date: Thu Oct 1 10:45:00 2026 +0700
 1 file changed, 3 insertions(+)
```

สังเกตว่า **hash ของ commit เปลี่ยนไปเลย** (จาก `2b4f8e1` เป็น `a1c9d3f`) — นี่คือประเด็นสำคัญที่จะอธิบายต่อในหัวข้อ 37.4

ถ้าไม่ใส่ `-m` ตอน `--amend` Git จะเปิด editor ขึ้นมาพร้อมข้อความเดิมให้แก้ไข:

```bash
$ git commit --amend
```

```
fix: preventt crash on null token

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Thu Oct 1 10:40:00 2026 +0700
#
# On branch main
# Changes to be committed:
#	modified:   app.js
#
```

### 37.3 กรณีที่ 2: ลืมเพิ่มไฟล์เข้า commit ก่อนหน้า

สถานการณ์ที่พบบ่อยมาก: commit ไปแล้วเพิ่งนึกได้ว่าลืมไฟล์หนึ่ง

```bash
$ git add auth.js
$ git commit -m "feat: add authentication module"
```

```
[main 6e2a1b3] feat: add authentication module
 1 file changed, 25 insertions(+)
```

อ้าว ลืม `auth.test.js` ที่ควรมากับไฟล์นี้:

```bash
$ git status -s
```

```
?? auth.test.js
```

เพิ่มไฟล์ที่ลืมแล้ว amend เข้าไปใน commit เดิม (ไม่ต้องเปลี่ยน message ก็ใช้ `--no-edit`):

```bash
$ git add auth.test.js
$ git commit --amend --no-edit
```

```
[main f8d3c2a] feat: add authentication module
 2 files changed, 40 insertions(+)
```

ตอนนี้ commit เดียวมีทั้งไฟล์ `auth.js` และ `auth.test.js` รวมกันอยู่ครบ เหมือนกับว่าตั้งแต่แรกเรา add ทั้งสองไฟล์พร้อมกัน

### 37.4 ⚠️ คำเตือนสำคัญที่สุด: ห้าม Amend Commit ที่ Push ไปแล้วให้คนอื่นดึงไปใช้

นี่คือกฎเหล็กข้อหนึ่งของการใช้ Git ในทีม:

> **`git commit --amend` ไม่ได้ "แก้ไข" commit เดิม แต่มันสร้าง commit ใหม่ทั้งหมดขึ้นมาแทนที่ (hash ใหม่เสมอ) แล้วให้ branch ปัจจุบันชี้ไปที่ commit ใหม่นั้นแทน commit เก่าจะถูกทิ้งไว้เบื้องหลัง (จะถูกเก็บกวาดทิ้งในที่สุดถ้าไม่มีอะไรอ้างอิงถึง)**

ถ้าคุณ `push` commit ไปที่ remote (เช่น GitHub) แล้ว **แล้วมีคนอื่นดึง (`pull`) commit นั้นไปใช้งานที่เครื่องเขาแล้ว** จากนั้นคุณ `--amend` แล้ว `push` ทับอีกที (ต้องใช้ `push --force` ด้วยเพราะประวัติไม่ตรงกันแล้ว) จะเกิดปัญหาใหญ่:

- ประวัติในเครื่องของเพื่อนร่วมทีมกับ remote จะไม่ตรงกันอีกต่อไป
- เพื่อนร่วมทีมจะเจอ conflict ประหลาด ๆ ที่อธิบายไม่ได้เวลา pull ครั้งถัดไป
- ในกรณีเลวร้ายที่สุด อาจทำให้งานของคนอื่นหายไปถ้าใช้ `push --force` โดยไม่ระวัง

**กฎง่าย ๆ ที่ควรจำ:**

| สถานการณ์ | Amend ได้ไหม |
|---|---|
| Commit นี้ยังอยู่ในเครื่อง ยังไม่ได้ push | ✅ Amend ได้อย่างปลอดภัย |
| Push ไปแล้ว แต่รู้แน่ชัดว่าไม่มีใครดึงไปใช้ (เช่น personal branch ของตัวเอง) | ⚠️ ทำได้แต่ต้องระวัง และต้อง `push --force-with-lease` ตอน push ทับ |
| Push ไปแล้ว และมีคนอื่นดึงไปใช้แล้ว (เช่น branch `main`/`develop` ของทีม) | ❌ **ห้ามเด็ดขาด** — ให้สร้าง commit ใหม่แก้ไขแทน (เช่นด้วย `git revert` ที่เราจะเรียนใน Part ถัดไป) |

เราจะเรียนรายละเอียดเรื่องการ "เขียนประวัติศาสตร์ใหม่" (rewriting history) แบบลึกซึ้งกว่านี้ในเฟส 4 ที่ว่าด้วย Rebase แต่ตอนนี้ขอให้จำกฎนี้ไว้ก่อนเป็นอันดับแรก

### สรุป Step 37

- `git commit --amend` แก้ไข commit ล่าสุดได้ทั้งข้อความและเนื้อหาไฟล์
- ทุกครั้งที่ amend จะได้ commit hash ใหม่เสมอ (เป็นการแทนที่ ไม่ใช่แก้ของเดิม)
- ใช้ `--amend --no-edit` เมื่อต้องการแก้แค่เนื้อหาไฟล์โดยไม่เปลี่ยนข้อความ
- **ห้าม amend commit ที่ push ไปแล้วและมีคนอื่นดึงไปใช้งานแล้วเด็ดขาด**

---

## Step 38: `git rm` และ `git mv` — ลบและย้าย/เปลี่ยนชื่อไฟล์อย่างถูกวิธี

### 38.1 ทำไมไม่ใช้ `rm` ของระบบปฏิบัติการตรง ๆ

คุณสามารถใช้คำสั่ง `rm` ปกติของระบบปฏิบัติการลบไฟล์ได้ แต่จะทำให้ต้อง `git add` เพิ่มอีกขั้นตอนเพื่อบันทึกการลบนั้น `git rm` ช่วยรวมสองขั้นตอน (ลบไฟล์จริง + stage การลบ) ให้เป็นคำสั่งเดียว

### 38.2 `git rm` — ลบไฟล์แบบที่ Git รู้จัก

ตัวอย่าง: สมมติเรามีไฟล์ที่ไม่ใช้แล้วชื่อ `old-config.json` ที่ Git track อยู่:

```bash
$ ls
app.js  old-config.json  README.md  style.css
$ git rm old-config.json
```

```
rm 'old-config.json'
```

คำสั่งนี้ทำ 2 อย่างพร้อมกัน: ลบไฟล์จริงออกจาก working directory และ stage การลบนั้นไว้พร้อม commit

```bash
$ git status -s
```

```
D  old-config.json
```

```bash
$ ls
app.js  README.md  style.css
```

```bash
$ git commit -m "chore: remove unused old-config.json"
```

```
[main d4e8f21] chore: remove unused old-config.json
 1 file changed, 12 deletions(-)
```

### 38.3 `git rm --cached` — เอาไฟล์ออกจากการ Track แต่เก็บไฟล์จริงไว้

บางครั้งคุณ commit ไฟล์บางไฟล์เข้าไปโดยไม่ตั้งใจ (เช่น ไฟล์ config ที่มี password หรือไฟล์ `.env`) แล้วอยากให้ Git หยุดติดตามไฟล์นั้นต่อไป **แต่ยังอยากเก็บไฟล์นั้นไว้ในเครื่องตามเดิม** ใช้ `--cached`:

```bash
$ cat .env
```

```
API_KEY=super-secret-key-12345
```

```bash
$ git rm --cached .env
```

```
rm '.env'
```

```bash
$ ls .env
```

```
.env
```

ไฟล์ `.env` ยังอยู่ในเครื่องเหมือนเดิม (คำสั่ง `ls` ยืนยันว่ายังเห็นไฟล์) แต่ Git จะไม่ track มันอีกต่อไปหลัง commit:

```bash
$ git status -s
```

```
D  .env
```

```bash
$ git commit -m "chore: stop tracking .env file"
```

```
[main 9a2b7c4] chore: stop tracking .env file
 1 file changed, 1 deletion(-)
```

> **สำคัญมาก:** หลังจากทำแบบนี้แล้ว ควรเพิ่ม `.env` เข้าไปใน `.gitignore` ทันที ไม่อย่างนั้น Git จะเห็นมันเป็น Untracked file โผล่มาเรื่อย ๆ ทุกครั้งที่ `git status`:

```bash
$ echo ".env" >> .gitignore
$ git add .gitignore
$ git commit -m "chore: add .env to gitignore"
```

```
[main 3d8e1f5] chore: add .env to gitignore
 1 file changed, 1 insertion(+)
```

> **หมายเหตุด้านความปลอดภัย:** การ `git rm --cached` ไฟล์ที่มีความลับ (secret) ออกไปนั้น **ไม่ได้ลบมันออกจากประวัติเก่าที่เคย commit ไปแล้ว** ใครก็ตามที่ clone repository นี้ยังสามารถย้อนดู commit เก่าแล้วเห็น secret เดิมได้อยู่ดี ถ้า secret หลุดไปแล้วจริง ต้องเปลี่ยน (rotate) secret นั้นทันที และถ้าต้องการลบออกจากประวัติทั้งหมดจริง ๆ ต้องใช้เครื่องมือขั้นสูงอย่าง `git filter-repo` ซึ่งเราจะเรียนในเฟสหลัง ๆ ของหลักสูตร

### 38.4 `git rm -r` — ลบทั้งโฟลเดอร์

```bash
$ git rm -r old-assets/
```

```
rm 'old-assets/logo-v1.png'
rm 'old-assets/logo-v2.png'
rm 'old-assets/README.md'
```

### 38.5 `git rm -f` — บังคับลบไฟล์ที่มีการแก้ไขค้างอยู่

ถ้าไฟล์มีการแก้ไขที่ยังไม่ถูก commit อยู่ Git จะปฏิเสธการลบเพื่อป้องกันข้อมูลหาย:

```bash
$ git rm app.js
```

```
error: the following file has local modifications:
    app.js
(use --cached to keep the file, or -f to force removal)
```

ถ้ามั่นใจว่าต้องการลบทิ้งจริง ๆ โดยไม่สนใจการแก้ไขที่ยังไม่ commit:

```bash
$ git rm -f app.js
```

```
rm 'app.js'
```

### 38.6 `git mv` — ย้ายหรือเปลี่ยนชื่อไฟล์

การใช้คำสั่งระบบปฏิบัติการ (`mv`) เปลี่ยนชื่อไฟล์แล้วต้อง `git add` ทั้งไฟล์เก่าที่หายไปกับไฟล์ใหม่ที่โผล่มาเอง 2 ขั้นตอน `git mv` รวมให้เป็นคำสั่งเดียว:

```bash
$ git mv style.css styles/main.css
```

```
$ git status -s
```

```
R  style.css -> styles/main.css
```

สังเกตรหัส `R` (Renamed) — Git ตรวจจับได้อัตโนมัติว่านี่คือการเปลี่ยนชื่อ/ย้ายไฟล์ ไม่ใช่การลบไฟล์เก่าแล้วสร้างไฟล์ใหม่แยกกัน (ตราบใดที่เนื้อหาไฟล์คล้ายกันมากพอ) ทำให้ประวัติอ่านง่ายกว่ามาก commit ตามปกติ:

```bash
$ git commit -m "refactor: move style.css into styles/ folder"
```

```
[main e5f2a9c] refactor: move style.css into styles/ folder
 1 file changed, 0 insertions(+), 0 deletions(-)
 rename style.css => styles/main.css (100%)
```

`git mv` ยังใช้เปลี่ยนแค่ชื่อไฟล์ (ไม่ย้ายโฟลเดอร์) ได้เช่นกัน:

```bash
$ git mv index.html home.html
$ git status -s
```

```
R  index.html -> home.html
```

### 38.7 เกร็ดความรู้: `git mv` เทียบเท่ากับ 3 คำสั่งรวมกัน

จริง ๆ แล้ว `git mv old new` เป็นเพียงคำสั่งลัดที่เทียบเท่ากับการรัน 3 คำสั่งนี้ต่อกัน:

```bash
$ mv old new
$ git rm --cached old
$ git add new
```

รู้เบื้องหลังแบบนี้ไว้จะช่วยให้เข้าใจว่าทำไม Git ถึงตรวจจับการ rename ได้ — จริง ๆ แล้ว Git ไม่ได้ "จำ" ว่ามีการ rename เกิดขึ้นโดยตรงในข้อมูล แต่มันจะ **เปรียบเทียบเนื้อหาไฟล์ที่หายไปกับไฟล์ใหม่ที่โผล่มา** แล้วถ้าเนื้อหาคล้ายกันเกินเกณฑ์ (ค่า default ประมาณ 50%) ก็จะรายงานเป็น "renamed" ให้อัตโนมัติตอนแสดงผล ไม่ว่าคุณจะใช้ `git mv` หรือใช้ `mv` ของระบบแล้ว add/rm เองก็ตาม

### สรุป Step 38

- `git rm` ลบไฟล์จริงและ stage การลบพร้อมกันในคำสั่งเดียว
- `git rm --cached` เอาไฟล์ออกจากการ track แต่เก็บไฟล์จริงไว้บนเครื่อง — เหมาะกับกรณี commit ไฟล์ลับไปโดยไม่ตั้งใจ
- `git rm -r` ลบทั้งโฟลเดอร์ `git rm -f` บังคับลบแม้มีการแก้ไขค้างอยู่
- `git mv` ย้าย/เปลี่ยนชื่อไฟล์พร้อม stage การเปลี่ยนแปลงในคำสั่งเดียว และ Git จะตรวจจับเป็น "rename" ให้อัตโนมัติ

---

## Step 39: Workflow แบบเต็มรอบที่ใช้ซ้ำทุกวัน

### 39.1 วงจรพื้นฐานที่คุณจะทำซ้ำนับพันครั้งตลอดอาชีพ

ทุกอย่างที่เรียนมาใน Part นี้ประกอบกันเป็นวงจรการทำงานพื้นฐานที่สุดของ Git:

```
  แก้ไขไฟล์ (edit)
        │
        ▼
  ตรวจสอบสถานะ (git status)
        │
        ▼
  เพิ่มเข้า staging (git add)
        │
        ▼
  ตรวจสอบอีกครั้งก่อน commit (git status / git diff --staged)
        │
        ▼
  บันทึก commit (git commit)
        │
        ▼
  ตรวจสอบประวัติ (git log)
        │
        └──── วนกลับไปแก้ไขต่อ ─────┘
```

### 39.2 สถานการณ์จริง: เพิ่มฟีเจอร์ "นับจำนวนคลิก" ให้เว็บไซต์

มาจำลองสถานการณ์การทำงานจริงตั้งแต่ต้นจนจบ เริ่มจากเช็คสถานะปัจจุบันก่อนเริ่มงาน:

```bash
$ git status
```

```
On branch main
nothing to commit, working tree clean
```

`working tree clean` แปลว่าไม่มีอะไรค้างอยู่เลย — เป็นจุดเริ่มต้นที่ดีที่สุดก่อนเริ่มงานใหม่ทุกครั้ง

**ขั้นที่ 1: แก้ไขไฟล์ HTML เพิ่มปุ่ม**

```bash
$ cat >> home.html << 'EOF'
<button id="click-btn">Click me!</button>
<p id="click-count">Clicks: 0</p>
EOF
```

**ขั้นที่ 2: เช็คสถานะ**

```bash
$ git status -s
```

```
 M home.html
```

**ขั้นที่ 3: เพิ่มเข้า staging**

```bash
$ git add home.html
```

**ขั้นที่ 4: เช็คสถานะอีกครั้ง**

```bash
$ git status -s
```

```
M  home.html
```

**ขั้นที่ 5: commit**

```bash
$ git commit -m "feat: add click button UI to home page"
```

```
[main 1a2b3c4] feat: add click button UI to home page
 1 file changed, 2 insertions(+)
```

ยังไม่เสร็จ ต้องเพิ่ม JavaScript ให้ปุ่มทำงานจริงด้วย ทำต่อในรอบถัดไปทันที:

**ขั้นที่ 1 (รอบ 2): เพิ่มไฟล์ JavaScript ใหม่**

```bash
$ cat > click-counter.js << 'EOF'
let count = 0;
document.getElementById('click-btn').addEventListener('click', () => {
  count++;
  document.getElementById('click-count').textContent = `Clicks: ${count}`;
});
EOF
```

**ขั้นที่ 2: เช็คสถานะ**

```bash
$ git status -s
```

```
?? click-counter.js
```

ไฟล์ใหม่นี้เป็น untracked ต้องเชื่อมกับ HTML ด้วย:

```bash
$ echo '<script src="click-counter.js"></script>' >> home.html
$ git status -s
```

```
 M home.html
?? click-counter.js
```

**ขั้นที่ 3: เพิ่มทั้งสองไฟล์**

```bash
$ git add home.html click-counter.js
```

**ขั้นที่ 4: ตรวจสอบก่อน commit ด้วย `git diff --staged`**

```bash
$ git diff --staged
```

```
diff --git a/click-counter.js b/click-counter.js
new file mode 100644
index 0000000..a3f8e91
--- /dev/null
+++ b/click-counter.js
@@ -0,0 +1,5 @@
+let count = 0;
+document.getElementById('click-btn').addEventListener('click', () => {
+  count++;
+  document.getElementById('click-count').textContent = `Clicks: ${count}`;
+});
diff --git a/home.html b/home.html
index 3f7a9c1..b2e5d84 100644
--- a/home.html
+++ b/home.html
@@ -10,3 +10,4 @@
 <button id="click-btn">Click me!</button>
 <p id="click-count">Clicks: 0</p>
+<script src="click-counter.js"></script>
```

ผลลัพธ์ตรงกับที่ตั้งใจ พร้อม commit

**ขั้นที่ 5: commit**

```bash
$ git commit -m "feat: wire up click counter logic to button"
```

```
[main 5e6f7a8] feat: wire up click counter logic to button
 2 files changed, 6 insertions(+)
 create mode 100644 click-counter.js
```

**ขั้นที่ 6: ตรวจสอบประวัติทั้งหมดด้วย `git log`**

```bash
$ git log --oneline
```

```
5e6f7a8 feat: wire up click counter logic to button
1a2b3c4 feat: add click button UI to home page
3d8e1f5 chore: add .env to gitignore
9a2b7c4 chore: stop tracking .env file
e5f2a9c refactor: move style.css into styles/ folder
d4e8f21 chore: remove unused old-config.json
f8d3c2a feat: add authentication module
a1c9d3f fix: prevent crash on null token
```

จะเห็นว่าประวัติทั้งหมดอ่านออกได้ทันทีว่าโปรเจกต์นี้ผ่านการเปลี่ยนแปลงอะไรมาบ้าง เรียงจากล่าสุดไปเก่าสุด — นี่คือพลังของการเขียน commit message ที่ดีและแบ่ง commit เป็นหน่วยที่มีความหมาย (เราจะเรียนคำสั่ง `git log` แบบละเอียดสุด ๆ ใน Part 05)

### 39.3 นิสัยที่ควรฝึกให้ติดตัวตั้งแต่วันแรก

| นิสัยที่ดี | เหตุผล |
|---|---|
| รัน `git status` ก่อนเริ่มงานทุกครั้ง | รู้ว่าจุดเริ่มต้นสะอาดหรือมีอะไรค้างอยู่ |
| รัน `git status` หลัง `git add` ทุกครั้ง | ยืนยันว่า stage ถูกไฟล์ ไม่ตกหล่นหรือเกิน |
| ใช้ `git diff --staged` ก่อน commit เสมอ | เห็นเนื้อหาจริงที่กำลังจะถูกบันทึก ไม่ commit มั่ว |
| commit บ่อย ๆ เป็นหน่วยเล็ก ๆ ที่มีความหมาย | ประวัติอ่านง่าย ย้อนกลับหรือ debug ได้แม่นยำกว่า |
| อย่า commit ทิ้งไว้ครึ่ง ๆ กลาง ๆ ในสถานะที่โค้ดพัง | ทีมอื่นที่ pull ไปอาจได้โค้ดที่ใช้งานไม่ได้ |

### สรุป Step 39

- Workflow หลักคือ: edit → status → add → status/diff → commit → log วนซ้ำ
- ควรเช็คสถานะทั้งก่อนและหลัง `git add` เสมอ
- `git diff --staged` คือเครื่องมือป้องกันการ commit ผิดพลาดที่ทรงพลังที่สุด
- Commit บ่อย ๆ เป็นหน่วยเล็กที่มีความหมาย ดีกว่า commit ก้อนใหญ่ที่รวมหลายเรื่อง

---

## Step 40: แบบฝึกหัดลงมือทำ — สร้างเว็บเพจง่าย ๆ พร้อม Commit 5 ครั้ง

ถึงเวลาลงมือทำเองทั้งหมดตั้งแต่ต้นจนจบ โดยใช้ทุกคำสั่งที่เรียนมาใน Part นี้

### 40.1 เป้าหมายของแบบฝึกหัด

สร้างเว็บเพจส่วนตัวง่าย ๆ (Personal Landing Page) ประกอบด้วย `index.html` และ `style.css` โดยพัฒนาเป็นขั้น ๆ และ commit อย่างมีความหมาย **5 ครั้ง** ตามลำดับการพัฒนาจริงที่โปรแกรมเมอร์ทำกันทั่วไป

### 40.2 เตรียมโฟลเดอร์

```bash
$ mkdir ~/git-course/part-04-exercise
$ cd ~/git-course/part-04-exercise
$ git init
```

```
Initialized empty Git repository in /home/user/git-course/part-04-exercise/.git/
```

### 40.3 Commit ครั้งที่ 1: โครงสร้าง HTML พื้นฐาน

```bash
$ cat > index.html << 'EOF'
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>My Personal Page</title>
</head>
<body>
  <h1>สวัสดี ฉันชื่อ...</h1>
</body>
</html>
EOF
```

```bash
$ git status -s
```

```
?? index.html
```

```bash
$ git add index.html
$ git commit -m "feat: add basic HTML skeleton for landing page"
```

```
[main a1b2c3d (root-commit) a1b2c3d] feat: add basic HTML skeleton for landing page
 1 file changed, 9 insertions(+)
 create mode 100644 index.html
```

### 40.4 Commit ครั้งที่ 2: เพิ่มเนื้อหาและ section ต่าง ๆ

```bash
$ cat > index.html << 'EOF'
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>My Personal Page</title>
</head>
<body>
  <header>
    <h1>สวัสดี ฉันชื่อ...</h1>
    <p>นักพัฒนาซอฟต์แวร์ที่กำลังเรียนรู้ Git</p>
  </header>
  <section id="about">
    <h2>เกี่ยวกับฉัน</h2>
    <p>ฉันกำลังเรียนหลักสูตร Git/GitHub/GitLab อยู่ที่ Step 40</p>
  </section>
  <footer>
    <p>&copy; 2026 My Personal Page</p>
  </footer>
</body>
</html>
EOF
```

```bash
$ git diff
```

```
diff --git a/index.html b/index.html
index 3f7a9c1..8c2d5e7 100644
--- a/index.html
+++ b/index.html
@@ -4,6 +4,14 @@
   <title>My Personal Page</title>
 </head>
 <body>
-  <h1>สวัสดี ฉันชื่อ...</h1>
+  <header>
+    <h1>สวัสดี ฉันชื่อ...</h1>
+    <p>นักพัฒนาซอฟต์แวร์ที่กำลังเรียนรู้ Git</p>
+  </header>
+  <section id="about">
+    <h2>เกี่ยวกับฉัน</h2>
+    <p>ฉันกำลังเรียนหลักสูตร Git/GitHub/GitLab อยู่ที่ Step 40</p>
+  </section>
+  <footer>
+    <p>&copy; 2026 My Personal Page</p>
+  </footer>
 </body>
 </html>
```

```bash
$ git add index.html
$ git commit -m "feat: add header, about section, and footer content"
```

```
[main b2c3d4e] feat: add header, about section, and footer content
 1 file changed, 9 insertions(+), 1 deletion(-)
```

### 40.5 Commit ครั้งที่ 3: เพิ่มไฟล์ CSS และเชื่อมกับ HTML

```bash
$ cat > style.css << 'EOF'
body {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f5f5f5;
}

header h1 {
  color: #2c3e50;
}
EOF
```

```bash
$ sed -i 's#</title>#</title>\n  <link rel="stylesheet" href="style.css">#' index.html
```

```bash
$ git status -s
```

```
 M index.html
?? style.css
```

```bash
$ git add .
$ git status -s
```

```
M  index.html
A  style.css
```

```bash
$ git diff --staged
```

```
diff --git a/index.html b/index.html
index 8c2d5e7..f1a9b3c 100644
--- a/index.html
+++ b/index.html
@@ -3,6 +3,7 @@
 <head>
   <meta charset="UTF-8">
   <title>My Personal Page</title>
+  <link rel="stylesheet" href="style.css">
 </head>
 <body>
diff --git a/style.css b/style.css
new file mode 100644
index 0000000..9e4f2a1
--- /dev/null
+++ b/style.css
@@ -0,0 +1,10 @@
+body {
+  font-family: Arial, sans-serif;
+  max-width: 800px;
+  margin: 0 auto;
+  padding: 20px;
+  background-color: #f5f5f5;
+}
+
+header h1 {
+  color: #2c3e50;
+}
```

```bash
$ git commit -m "style: add stylesheet and link it to index.html"
```

```
[main c3d4e5f] style: add stylesheet and link it to index.html
 2 files changed, 11 insertions(+)
 create mode 100644 style.css
```

### 40.6 Commit ครั้งที่ 4: เพิ่ม section ติดต่อ พร้อมใช้ `git add -p`

```bash
$ cat >> index.html << 'EOF'
<!-- TODO: add social media links later -->
EOF
$ sed -i '/<\/section>/a\\n  <section id="contact">\n    <h2>ติดต่อฉัน</h2>\n    <p>Email: example@example.com</p>\n  </section>' index.html
```

ตอนนี้มีสองการเปลี่ยนแปลงปนกัน: comment `TODO` (ยังไม่พร้อม commit) และ section ติดต่อ (พร้อม commit) ใช้ `git add -p` แยก:

```bash
$ git add -p index.html
```

```
diff --git a/index.html b/index.html
index f1a9b3c..7d8e2f4 100644
--- a/index.html
+++ b/index.html
@@ -10,5 +10,10 @@
     <h2>เกี่ยวกับฉัน</h2>
     <p>ฉันกำลังเรียนหลักสูตร Git/GitHub/GitLab อยู่ที่ Step 40</p>
   </section>
+
+  <section id="contact">
+    <h2>ติดต่อฉัน</h2>
+    <p>Email: example@example.com</p>
+  </section>
   <footer>
     <p>&copy; 2026 My Personal Page</p>
   </footer>
Stage this hunk [y,n,q,a,d,e,?]? y
```

```
@@ -19,3 +24,4 @@
 </body>
 </html>
+<!-- TODO: add social media links later -->
Stage this hunk [y,n,q,a,d,e,?]? n
```

```bash
$ git status -s
```

```
MM index.html
```

```bash
$ git commit -m "feat: add contact section with email"
```

```
[main d4e5f6a] feat: add contact section with email
 1 file changed, 5 insertions(+)
```

ยืนยันว่า comment `TODO` ยังไม่ถูกรวมเข้า commit:

```bash
$ git status -s
```

```
 M index.html
```

```bash
$ git diff
```

```
diff --git a/index.html b/index.html
index 7d8e2f4..8f3a1c5 100644
--- a/index.html
+++ b/index.html
@@ -19,3 +19,4 @@
 </body>
 </html>
+<!-- TODO: add social media links later -->
```

เนื่องจากมันเป็นแค่ TODO ที่ยังไม่ต้องใช้ตอนนี้ ลองยกเลิกการเปลี่ยนแปลงนี้ทิ้งไปก่อน:

```bash
$ git restore index.html
$ git status -s
```

```

```

(ไม่มีผลลัพธ์อะไรเลย แปลว่า working tree สะอาดแล้ว)

### 40.7 Commit ครั้งที่ 5: แก้ไข commit message ที่พิมพ์ผิดด้วย `--amend` และปิดท้ายด้วย README

สมมติเรานึกขึ้นได้ว่า commit ครั้งที่ 3 ควรจะระบุให้ชัดกว่านี้ ลองดูประวัติก่อน:

```bash
$ git log --oneline
```

```
d4e5f6a feat: add contact section with email
c3d4e5f style: add stylesheet and link it to index.html
b2c3d4e feat: add header, about section, and footer content
a1b2c3d feat: add basic HTML skeleton for landing page
```

(ในตัวอย่างนี้ commit ล่าสุดคือ `d4e5f6a` ซึ่งข้อความก็ดีอยู่แล้ว ไม่จำเป็นต้องแก้ — `--amend` แก้ได้แค่ commit บนสุดเท่านั้น เราจึงใช้โอกาสนี้สร้าง commit ที่ 5 จริง ๆ แทน โดยเพิ่มไฟล์ README ปิดท้ายโปรเจกต์)

```bash
$ cat > README.md << 'EOF'
# My Personal Page

เว็บเพจส่วนตัวง่าย ๆ ที่สร้างขึ้นเป็นแบบฝึกหัด Part 04
ของหลักสูตร Git/GitHub/GitLab

## โครงสร้างไฟล์

- `index.html` — หน้าเว็บหลัก
- `style.css` — สไตล์ของหน้าเว็บ
EOF
```

```bash
$ git status -s
```

```
?? README.md
```

```bash
$ git add README.md
$ git commit -m "docs: add README describing project structure"
```

```
[main e5f6a7b] docs: add README describing project structure
 1 file changed, 8 insertions(+)
 create mode 100644 README.md
```

ตรวจสอบผลลัพธ์สุดท้ายทั้งหมด:

```bash
$ git log --oneline
```

```
e5f6a7b docs: add README describing project structure
d4e5f6a feat: add contact section with email
c3d4e5f style: add stylesheet and link it to index.html
b2c3d4e feat: add header, about section, and footer content
a1b2c3d feat: add basic HTML skeleton for landing page
```

```bash
$ git status
```

```
On branch main
nothing to commit, working tree clean
```

ครบ 5 commit ที่มีความหมายเรียงตามลำดับพัฒนาการจริง: โครงสร้าง → เนื้อหา → สไตล์ → ฟีเจอร์เพิ่มเติม → เอกสารประกอบ พร้อมทั้งได้ฝึกใช้ `git status`, `git add` (ทั้งแบบเจาะจงไฟล์และแบบ `-p`), `git diff --staged`, `git commit`, และ `git log` ครบทุกคำสั่งที่เรียนมาใน Part นี้

### 40.8 ทดลองเพิ่มเติม (ไม่บังคับ แต่แนะนำให้ลอง)

1. ลองใช้ `git mv style.css styles/main.css` แล้วอัปเดต path ใน `index.html` ให้ตรงกัน แล้ว commit
2. ลองสร้างไฟล์ `.env` ปลอม ๆ เช่น `echo "SECRET=123" > .env` แล้ว commit ไปโดยไม่ตั้งใจ จากนั้นฝึกใช้ `git rm --cached .env` และเพิ่มลง `.gitignore`
3. ลองใช้ `git commit --amend` แก้ไข commit message ล่าสุดของคุณเองดูสักครั้งเพื่อความคุ้นเคย

### 40.9 Checklist ก่อนไป Part 05

ก่อนไปต่อ Part 05 ให้ตรวจสอบว่าคุณ:

- [ ] สร้าง Git repository ใหม่ด้วย `git init` ได้ และเข้าใจว่า `.git` คืออะไร
- [ ] อ่านผลลัพธ์ของ `git status` ได้ครบทุกส่วน (Untracked / Modified / Staged) รวมถึงโหมด `-s`
- [ ] ใช้ `git add` ได้ทั้งแบบเจาะจงไฟล์, `git add .`, `git add -A` และเข้าใจความต่างของขอบเขต
- [ ] เคยลองใช้ `git add -p` แยก stage เฉพาะบางส่วนของไฟล์อย่างน้อยหนึ่งครั้ง
- [ ] เขียน commit message ตามกฎ 50/72 และใช้ imperative mood ได้
- [ ] เข้าใจความต่างของ `git commit -m`, `git commit` (เปิด editor), และ `git commit -am` พร้อมข้อควรระวังเรื่องไฟล์ untracked
- [ ] ใช้ `git commit --amend` แก้ไข commit ล่าสุดได้ และเข้าใจว่าทำไมห้าม amend commit ที่แชร์กับคนอื่นแล้ว
- [ ] ใช้ `git rm`, `git rm --cached`, และ `git mv` ได้อย่างถูกต้อง
- [ ] ทำ workflow เต็มรอบ (edit → status → add → status → commit → log) ได้อย่างคล่องแคล่วโดยไม่ต้องเปิดโน้ตดู
- [ ] ทำแบบฝึกหัด Step 40 เสร็จครบ 5 commit บนโปรเจกต์ของตัวเอง

---

## สรุป Part 04

ใน Part นี้เราได้ลงมือปฏิบัติจริงกับคำสั่ง Git พื้นฐานชุดแรกอย่างเข้มข้น:

1. `git init` สร้าง Repository ใหม่โดยสร้างโฟลเดอร์ `.git` ที่เก็บทุกอย่างของ Git ไว้เพียงที่เดียว
2. `git status` คือคำสั่งที่ใช้บ่อยที่สุด ช่วยบอกสถานะไฟล์ทั้งสามแบบ: Untracked, Modified (not staged), และ Staged รวมถึงโหมดย่อ `-s` ที่อ่านง่ายเมื่อไฟล์เยอะ
3. `git add` มีหลายรูปแบบ ตั้งแต่เจาะจงไฟล์ ไปจนถึง `.`, `-A`, และ pattern แบบ wildcard โดยต้องเข้าใจขอบเขตของแต่ละแบบให้ชัดเจน
4. `git add -p` ให้อำนาจในการเลือก stage เฉพาะบางส่วนของไฟล์ (hunk) เหมาะกับการแยก commit ที่มีหลายเรื่องปนกันในไฟล์เดียว
5. Commit message ที่ดีต้องอธิบาย "ทำไม" ไม่ใช่แค่ "อะไร" เปลี่ยน ตามกฎ 50/72 และเขียนแบบ imperative mood
6. `git commit -m`, `git commit` (เปิด editor), และ `git commit -am` มีจุดใช้งานต่างกัน โดยเฉพาะข้อควรระวังว่า `-a` ไม่รวมไฟล์ untracked
7. `git commit --amend` ใช้แก้ไข commit ล่าสุดได้ทั้งข้อความและไฟล์ แต่ห้ามใช้กับ commit ที่ push และมีคนอื่นดึงไปใช้แล้วเด็ดขาด
8. `git rm` และ `git mv` จัดการการลบและย้าย/เปลี่ยนชื่อไฟล์พร้อม stage การเปลี่ยนแปลงในคำสั่งเดียว
9. Workflow แบบเต็มรอบ (edit → status → add → status → commit → log) คือแกนหลักของการทำงานกับ Git ทุกวัน
10. แบบฝึกหัดปิดท้ายทำให้คุณได้ประกอบทุกคำสั่งเข้าด้วยกันในสถานการณ์จำลองที่ใกล้เคียงงานจริงที่สุด

ตอนนี้คุณมีเครื่องมือพื้นฐานครบมือแล้วสำหรับการสร้างและบันทึกประวัติของโปรเจกต์ Git แต่สิ่งที่ยังขาดอยู่คือความสามารถใน **"การอ่านย้อนหลัง"** — รู้ว่าใครแก้อะไรไปบ้าง เมื่อไหร่ และต่างจากเดิมตรงไหน ซึ่งเป็นสิ่งที่เราจะเรียนกันแบบละเอียดใน Part ถัดไป

**ต่อไป:** [Part 05: การดูประวัติ: log, diff, show และการอ่าน commit graph](./part-005-การดูประวัติ-log-diff-show.md)
