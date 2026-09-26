# Part 14: การ Undo การเปลี่ยนแปลง: checkout, restore, reset (soft/mixed/hard)

> **Step ในหลักสูตรนี้:** Step 131–140
> **เฟส:** 2 — ใช้คำสั่งพื้นฐานของ Git ได้คล่องในชีวิตจริง
> **เป้าหมายของ Part นี้:** เข้าใจอย่างแม่นยำ 100% ว่า Git มีระดับการ "ย้อนกลับ" กี่ระดับ แต่ละคำสั่ง (`restore`, `reset --soft/--mixed/--hard`, `revert`) ทำอะไรกับ working directory, staging area และประวัติ commit บ้าง และรู้ว่าเมื่อไหร่ควรใช้คำสั่งไหนโดยไม่ทำลายงานของตัวเองหรือของทีมโดยไม่ได้ตั้งใจ

---

> **คำเตือนก่อนเริ่ม:** Part นี้เป็นหนึ่งใน Part ที่ **สำคัญที่สุด** ของทั้งหลักสูตร เพราะเป็นจุดที่มือใหม่ (และมือเก่าจำนวนไม่น้อย) ทำงานหายบ่อยที่สุด เนื่องจากสับสนระหว่างคำสั่งที่ "หน้าตาคล้ายกัน" แต่ผลลัพธ์ต่างกันโดยสิ้นเชิง เช่น `git reset --hard` กับ `git reset --soft` หรือ `git restore` กับ `git restore --staged` ขอให้อ่านช้า ๆ และลงมือทำตามแบบฝึกหัดจริงทุกข้อใน Step 140 ก่อนไปต่อ Part ถัดไป

---

## สารบัญของ Part นี้

- Step 131: ภาพรวมการ Undo ใน Git — มีกี่ระดับ และแต่ละคำสั่งทำงานที่ระดับไหน
- Step 132: `git restore <file>` — ยกเลิกการแก้ไขใน Working Directory
- Step 133: `git restore --staged <file>` — เอาไฟล์ออกจาก Staging โดยไม่เสียงานที่แก้ไว้
- Step 134: `git reset --soft <commit>` — ย้อน Commit แต่เก็บการเปลี่ยนแปลงไว้ใน Staging Area
- Step 135: `git reset --mixed <commit>` — ย้อน Commit และ Unstage แต่เก็บไฟล์ไว้ใน Working Directory (ค่า Default)
- Step 136: `git reset --hard <commit>` — ย้อนทุกอย่างและลบทิ้งถาวร (อันตรายที่สุด)
- Step 137: ตารางเปรียบเทียบ soft / mixed / hard แบบสรุปที่สุด
- Step 138: `git revert <commit>` — ย้อน Commit แบบปลอดภัยสำหรับ Branch ที่แชร์กับคนอื่น
- Step 139: Reset vs Revert — เมื่อไหร่ควรใช้อันไหน (กฎทองที่ห้ามฝ่าฝืน)
- Step 140: แบบฝึกหัด — จำลอง 4 สถานการณ์จริงเพื่อฝึก restore, reset ทั้ง 3 แบบ และ revert

---

## Step 131: ภาพรวมการ Undo ใน Git — มีกี่ระดับ และแต่ละคำสั่งทำงานที่ระดับไหน

ก่อนจะเรียนคำสั่งใด ๆ เราต้องเข้าใจโครงสร้างพื้นฐานที่สุดของ Git ก่อน นั่นคือ **Git มีพื้นที่เก็บข้อมูลอยู่ 3 ระดับ (Three Trees)** ที่ทำงานร่วมกันเสมอ:

```
┌─────────────────────┐     git add      ┌─────────────────────┐     git commit      ┌─────────────────────┐
│  1. Working          │ ───────────────▶ │  2. Staging Area     │ ──────────────────▶ │  3. Repository        │
│     Directory         │                  │     (Index)           │                     │     (Commit History)   │
│  (ไฟล์จริงบนดิสก์)      │ ◀─────────────── │  (พื้นที่พักก่อน commit) │ ◀────────────────── │  (.git object database)│
└─────────────────────┘   restore/checkout └─────────────────────┘   reset (mixed/hard) └─────────────────────┘
```

ทุกครั้งที่คุณแก้ไขไฟล์ ข้อมูลจะไหลผ่านสามระดับนี้ตามลำดับ และ **การ undo แต่ละคำสั่งก็คือการ "ดึงข้อมูลย้อนกลับ" ระหว่างระดับเหล่านี้** นั่นเอง ไม่มีอะไรลึกลับไปกว่านี้

### 1. Working Directory (พื้นที่ทำงาน)

คือไฟล์จริง ๆ ที่คุณเห็นในโฟลเดอร์โปรเจกต์ เปิดด้วย Text Editor แล้วแก้ไขได้ตามใจ Git จะมองว่าไฟล์เหล่านี้ **"ยังไม่ถูกติดตาม (untracked)"** หรือ **"ถูกแก้ไขแล้วแต่ยัง unstaged (modified)"** จนกว่าคุณจะสั่ง `git add`

### 2. Staging Area / Index (พื้นที่เตรียมพร้อม)

คือพื้นที่พักคอยระหว่าง Working Directory กับ Repository เมื่อคุณสั่ง `git add <file>` ไฟล์จะถูกก็อปปี้สถานะปัจจุบันเข้ามาไว้ที่นี่ รอให้คุณสั่ง `git commit` เพื่อบันทึกลง Repository อย่างถาวร Staging Area คือสิ่งที่ทำให้ Git พิเศษกว่า VCS หลายตัว เพราะมันให้คุณ "เลือก" ได้ว่าจะ commit อะไรบ้างในแต่ละครั้ง ไม่จำเป็นต้อง commit ทุกไฟล์ที่แก้พร้อมกันเสมอไป

### 3. Repository / Commit History (ประวัติที่บันทึกแล้ว)

คือฐานข้อมูลของ Git (โฟลเดอร์ `.git`) ที่เก็บ snapshot ของทุก commit ที่เคยสร้างไว้อย่างถาวร (จนกว่าจะถูกลบทิ้งจริง ๆ) นี่คือ "ประวัติศาสตร์" ของโปรเจกต์ ตัวชี้ที่สำคัญที่สุดในระดับนี้คือ **HEAD** ซึ่งชี้ไปยัง commit ปัจจุบันที่คุณกำลังยืนอยู่

### แผนที่คำสั่ง Undo ทั้งหมด — ตัวไหนทำงานที่ระดับไหน

นี่คือภาพรวมที่คุณควรจำไว้ตลอดทั้ง Part นี้:

```
                    ระดับที่ 3              ระดับที่ 2              ระดับที่ 1
                 Commit History          Staging Area          Working Directory
                 (ประวัติ / HEAD)         (สิ่งที่จะ commit)        (ไฟล์บนดิสก์)
                       │                        │                        │
   git restore         │                        │◀───────────────────────┤ ยกเลิกการแก้ไข
   <file>              │                        │                        │ กลับไปเหมือนค่าที่ staged/HEAD
                       │                        │                        │
   git restore         │                        │───────────────────────▶│ เอาออกจาก staging
   --staged <file>     │                        │  (ไม่แตะ working dir)   │ (ไฟล์ในดิสก์ไม่เปลี่ยน)
                       │                        │                        │
   git reset --soft    │◀───────────────────────┤                        │
   <commit>            │  ย้าย HEAD/branch เท่านั้น  (ไม่แตะ)              │  (ไม่แตะ)
                       │                        │                        │
   git reset --mixed   │◀───────────────────────┼◀───────────────────────┤
   <commit>            │  ย้าย HEAD/branch      │  รีเซ็ต staging ตาม     │  (ไม่แตะ)
                       │                        │  commit ปลายทาง        │
                       │                        │                        │
   git reset --hard    │◀───────────────────────┼◀───────────────────────┼◀───────────────────────
   <commit>            │  ย้าย HEAD/branch      │  รีเซ็ต staging         │  รีเซ็ต working dir
                       │                        │                        │  (ข้อมูลหายจริง!)
                       │                        │                        │
   git revert          │  สร้าง commit ใหม่      │                        │
   <commit>            │  ที่ทำตรงข้าม (ไม่ลบประวัติ)                       │
```

สังเกตรูปแบบ (pattern) ที่สำคัญมาก:

- `restore` ทำงานอยู่แค่ระหว่าง **ระดับ 1 กับ ระดับ 2** เท่านั้น — **ไม่แตะต้องประวัติ commit เลย**
- `reset` ทำงานโดยเริ่มจาก **ระดับ 3 (ย้าย HEAD) แล้วค่อย ๆ ลามลงมา** — ยิ่งใช้ flag ที่ "แรง" ขึ้น (`--soft` → `--mixed` → `--hard`) ยิ่งกระทบระดับที่ลึกลงไปมากขึ้น
- `revert` **ไม่ย้อนหรือลบประวัติเลยแม้แต่นิดเดียว** แต่สร้าง commit ใหม่ขึ้นมาเพิ่มในประวัติเพื่อ "หักล้าง" ผลของ commit เก่า

### ทำไมต้องแยกให้ชัด

เหตุผลที่มือใหม่สับสนบ่อยที่สุดคือ **คิดว่าทุกคำสั่งที่ขึ้นต้นด้วยคำว่า "undo" ทำงานเหมือนกันหมด** ทั้งที่จริง ๆ แล้วแต่ละคำสั่งกระทบคนละระดับ และมีความเสี่ยงต่อการสูญเสียข้อมูลไม่เท่ากันเลย ตั้งแต่ **"ปลอดภัยสนิท ย้อนกลับได้เสมอ"** ไปจนถึง **"ลบทิ้งถาวร กู้คืนยากมาก"**

ตารางสรุประดับความเสี่ยงคร่าว ๆ ก่อนเข้ารายละเอียด (จะมีตารางแบบเต็มใน Step 137):

| คำสั่ง | ระดับความเสี่ยงต่อการเสียงาน | แตะประวัติ commit ไหม |
|---|---|---|
| `git restore <file>` | ปานกลาง (เสียการแก้ไขที่ยังไม่ได้ stage) | ไม่ |
| `git restore --staged <file>` | ต่ำมาก (แค่ unstage การแก้ไขยังอยู่) | ไม่ |
| `git reset --soft` | ต่ำ (แค่ย้าย HEAD การเปลี่ยนแปลงยังอยู่ครบ) | ย้าย HEAD/branch pointer |
| `git reset --mixed` | ปานกลาง (unstage แต่ไฟล์ยังอยู่ใน working dir) | ย้าย HEAD/branch pointer |
| `git reset --hard` | **สูงมาก (ลบข้อมูลถาวร)** | ย้าย HEAD/branch pointer + ล้าง working dir |
| `git revert` | ต่ำมาก (ไม่ลบอะไรเลย เพิ่ม commit ใหม่) | เพิ่ม commit ใหม่ (ไม่ลบของเก่า) |

จำหลักการนี้ไว้ก่อนไปต่อ: **`restore` ปลอดภัยกับประวัติเสมอ, `reset` อันตรายมากขึ้นเรื่อย ๆ ตาม flag, และ `revert` คือทางเลือกที่ปลอดภัยที่สุดเมื่อทำงานกับคนอื่น**

---

## Step 132: `git restore <file>` — ยกเลิกการแก้ไขใน Working Directory

### สถานการณ์ที่ใช้

คุณกำลังแก้ไขไฟล์อยู่ แต่ยังไม่ได้ `git add` เลย แล้วพบว่าการแก้ไขที่ทำไปทั้งหมดนั้น **"ผิดทาง" หรือ "ไม่ต้องการแล้ว"** อยากได้ไฟล์กลับไปเป็นเหมือนเดิมตอนที่ commit ล่าสุด (หรือตอนที่ staged ไว้)

### คำสั่ง

```bash
git restore <file>
```

ตัวอย่างจริง:

```bash
$ echo "โค้ดทดลองที่พังจริง ๆ" >> app.js
$ git status
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   app.js

$ git restore app.js

$ git status
On branch main
nothing to commit, working tree clean
```

ไฟล์ `app.js` กลับไปเหมือนเวอร์ชันล่าสุดที่อยู่ใน commit (หรือใน staging area ถ้ามีอะไร staged อยู่ก่อนหน้า) ทันที **การเปลี่ยนแปลงที่คุณทำไปในไฟล์นั้นจะหายไปจริง ๆ และกู้คืนไม่ได้** เพราะมันไม่เคยถูก commit หรือ stage มาก่อนเลย — Git ไม่มีสำเนาของมันเก็บไว้ที่ไหน

> **สังเกต:** ข้อความจาก `git status` เองก็บอกวิธีใช้คำสั่งนี้ตรง ๆ อยู่แล้ว (`use "git restore <file>..." to discard changes in working directory`) — Git สมัยใหม่พยายามแนะนำผู้ใช้ผ่านข้อความเหล่านี้เสมอ ควรอ่านทุกครั้ง

### restore ไฟล์หลายไฟล์ / ทั้งหมด

```bash
# ยกเลิกการแก้ไขไฟล์เจาะจงหลายไฟล์
git restore app.js style.css

# ยกเลิกการแก้ไขทุกไฟล์ในโฟลเดอร์ปัจจุบันและย่อย
git restore .

# ยกเลิกการแก้ไขทุกไฟล์ทั้งโปรเจกต์ (จาก root เสมอ)
git restore :/
```

### restore จาก commit อื่นที่ไม่ใช่ HEAD

โดย default `git restore <file>` จะดึงเนื้อหาไฟล์กลับมาจาก **staging area** ถ้าไฟล์นั้นมีอะไร staged อยู่ หรือจาก **HEAD** ถ้าไม่มีอะไร staged แต่ในบางกรณีคุณอาจอยากดึงเนื้อหาไฟล์จาก commit อื่นในอดีต ใช้ flag `--source`:

```bash
# ดึงเนื้อหาไฟล์ config.json จาก commit 3 อันก่อนหน้ามาทับ working directory ตอนนี้
git restore --source=HEAD~3 config.json
```

คำสั่งนี้มีประโยชน์มากเวลาต้องการ "หยิบไฟล์เก่ากลับมาดูหรือใช้" โดยไม่ต้องย้อนทั้ง branch

### เปรียบเทียบกับคำสั่งเก่า: `git checkout -- <file>`

ก่อนที่ Git เวอร์ชัน 2.23 (ปี 2019) จะเปิดตัวคำสั่ง `restore` และ `switch` คำสั่งเดียวที่ใช้ทำสิ่งนี้ได้คือ:

```bash
git checkout -- app.js
```

ผลลัพธ์ **เหมือนกันทุกประการ** กับ `git restore app.js` แต่ปัญหาของ `git checkout` แบบเก่าคือมันเป็นคำสั่งที่ **"ทำได้หลายอย่างเกินไปในคำสั่งเดียว"** — `git checkout` ตัวเดียวสามารถ:

1. สลับ branch (`git checkout main`)
2. สลับไปดู commit เก่า (`git checkout a1b2c3d`)
3. กู้คืนไฟล์จาก HEAD/staging (`git checkout -- app.js`)
4. สร้าง branch ใหม่พร้อมสลับ (`git checkout -b new-branch`)

การใช้คำสั่งเดียวทำหลายบทบาททำให้มือใหม่สับสนและพิมพ์ผิดพลาดง่ายมาก (เช่น ลืมใส่ `--` แล้วดันไปสลับ branch โดยไม่ตั้งใจ) Git ทีมพัฒนาจึงแยกหน้าที่ออกมาเป็นคำสั่งใหม่ที่ชัดเจนกว่า:

| งาน | คำสั่งเก่า (ยังใช้ได้ ไม่ถูก deprecate) | คำสั่งใหม่ (แนะนำ) |
|---|---|---|
| กู้คืนไฟล์จาก working directory | `git checkout -- <file>` | `git restore <file>` |
| เอาไฟล์ออกจาก staging | `git reset HEAD <file>` | `git restore --staged <file>` |
| สลับ branch | `git checkout <branch>` | `git switch <branch>` |
| สร้าง branch ใหม่พร้อมสลับ | `git checkout -b <branch>` | `git switch -c <branch>` |

> **หมายเหตุสำคัญ:** `git checkout -- <file>` **ไม่ได้ถูกยกเลิก** และยังพบเห็นได้ในโค้ดเก่า บทความเก่า และทีมที่ยังไม่อัปเดต workflow ควรอ่านออกและเข้าใจว่ามันเทียบเท่ากับ `git restore` แต่ในโปรเจกต์ใหม่ของตัวเอง **แนะนำให้ใช้ `git restore` และ `git switch` เสมอ** เพราะชื่อคำสั่งสื่อความหมายตรงตัวและลดโอกาสพิมพ์ผิดพลาด

### จุดที่มือใหม่พลาดบ่อย

**พลาดข้อ 1:** ลืมว่าไฟล์ที่ `git restore` ไปแล้ว **กู้คืนไม่ได้เลย** ถ้าไม่เคย stage หรือ commit มาก่อน ก่อนสั่ง `restore` ควรเช็คด้วย `git diff` ก่อนเสมอว่าการเปลี่ยนแปลงที่จะทิ้งคืออะไรบ้าง

```bash
git diff app.js      # ดูว่ากำลังจะทิ้งอะไรไปก่อนสั่ง restore
git restore app.js
```

**พลาดข้อ 2:** คิดว่า `git restore <file>` จะเอาไฟล์ออกจาก staging ด้วย — **ไม่จริง** ถ้าไฟล์ถูก `git add` ไปแล้ว `git restore <file>` เฉย ๆ (ไม่มี `--staged`) จะไม่ทำอะไรกับ staging area เลย มันจะแค่ทำให้ **working directory ตรงกับสิ่งที่ staged อยู่** เท่านั้น เรื่องการ unstage คือหน้าที่ของ Step ถัดไป

---

## Step 133: `git restore --staged <file>` — เอาไฟล์ออกจาก Staging โดยไม่เสียงานที่แก้ไว้

### สถานการณ์ที่ใช้

คุณสั่ง `git add` ไฟล์ไปแล้ว แต่พบว่า **ยังไม่อยากรวมไฟล์นี้เข้ากับ commit ที่กำลังจะสร้าง** อาจเพราะ:

- `git add .` ไปโดยไม่ตั้งใจ ดันเอาไฟล์ที่ยังไม่พร้อมติดไปด้วย
- อยากแยก commit ออกเป็นหลาย commit เล็ก ๆ ตาม logical change (จะเรียนละเอียดเรื่อง atomic commits ใน Part 09 ซ้ำอีกครั้งเชิงลึกใน Part 40)
- ไฟล์ที่ stage ไปมีข้อมูลลับ (secret/credential) ที่ไม่ควร commit

**สิ่งสำคัญที่สุดที่ต้องเข้าใจ:** คำสั่งนี้ **ไม่ทำลายการแก้ไขของคุณเลย** มันแค่ย้ายไฟล์จาก "พื้นที่ที่จะ commit" กลับไปเป็น "พื้นที่ที่แก้ไขแล้วแต่ยังไม่ stage" เท่านั้น — เนื้อหาในไฟล์ที่คุณเห็นบนดิสก์ **ไม่เปลี่ยนแปลงแม้แต่ตัวอักษรเดียว**

### คำสั่ง

```bash
git restore --staged <file>
```

ตัวอย่างจริง:

```bash
$ git add app.js secret.env
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   app.js
        new file:   secret.env

$ git restore --staged secret.env

$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   app.js
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        secret.env
```

สังเกตว่า `secret.env` ยังคงอยู่ในโฟลเดอร์ ไม่ได้ถูกลบทิ้ง เพียงแค่กลับไปอยู่สถานะ "untracked" (สำหรับไฟล์ใหม่) หรือ "modified, not staged" (สำหรับไฟล์ที่เคย commit มาก่อน) เท่านั้น คุณยังสามารถแก้ไขต่อ หรือ `git add` กลับเข้ามาใหม่ได้ทุกเมื่อ

### unstage ทั้งหมดในครั้งเดียว

```bash
git restore --staged .
```

### เปรียบเทียบกับคำสั่งเก่า: `git reset HEAD <file>`

ก่อนมี `git restore` วิธี unstage แบบดั้งเดิมคือ:

```bash
git reset HEAD app.js
```

ผลลัพธ์เหมือนกันทุกประการกับ `git restore --staged app.js` — เหตุผลที่คำสั่งเก่าใช้ `reset` ก็เพราะในทางเทคนิคแล้ว **การ unstage ไฟล์เดียวคือการทำ `reset --mixed` แบบเจาะจงไฟล์** (ปรับ index ให้ตรงกับ HEAD เฉพาะไฟล์นั้น โดยไม่แตะ working directory) แต่การใช้คำว่า `reset` กับการ "แก้ไขไฟล์เดียว" ทำให้คนสับสนกับ `reset` แบบที่ย้อน commit ทั้ง branch (ที่เราจะเรียนใน Step 134–136) จึงเป็นอีกเหตุผลที่ Git แยกคำสั่ง `restore` ออกมาให้ชัดเจนขึ้น

| งาน | คำสั่งเก่า | คำสั่งใหม่ (แนะนำ) |
|---|---|---|
| Unstage ไฟล์เดียว | `git reset HEAD <file>` | `git restore --staged <file>` |
| Unstage ทั้งหมด | `git reset HEAD` หรือ `git reset` | `git restore --staged .` |

### ความแตกต่างสำคัญที่ต้องแยกให้ออก: restore เฉย ๆ vs restore --staged

นี่คือจุดที่สับสนกันบ่อยมาก มาดูตารางเทียบให้ชัดเจน:

| คำสั่ง | ทำงานที่ไหน | ผลลัพธ์ | เสียการแก้ไขไหม |
|---|---|---|---|
| `git restore <file>` | Staging Area → Working Directory | ไฟล์ใน working directory กลับไปเหมือนที่ staged/HEAD | **เสีย** (ถ้ามีการแก้ไขที่ยังไม่ stage) |
| `git restore --staged <file>` | Repository (HEAD) → Staging Area | ไฟล์ถูกเอาออกจาก staging area กลับไปอยู่สถานะ modified/untracked | **ไม่เสีย** เนื้อหาในดิสก์เหมือนเดิมทุกประการ |

### Workflow ที่ใช้บ่อยมาก: restore --staged แล้วค่อย restore

บางครั้งคุณต้องการ **ยกเลิกทั้ง staging และทั้ง working directory** กลับไปเป็นเหมือน HEAD เป๊ะ ๆ วิธีที่ปลอดภัยที่สุดคือทำทีละขั้น:

```bash
# ขั้น 1: unstage ก่อน (ยังไม่เสียอะไร)
git restore --staged app.js

# ขั้น 2: ตรวจดูว่าจะทิ้งอะไรไปบ้าง
git diff app.js

# ขั้น 3: ถ้ามั่นใจแล้วค่อยยกเลิกการแก้ไขจริง
git restore app.js
```

หรือจะรวบเป็นคำสั่งเดียวก็ได้ (แต่ต้องมั่นใจก่อนเสมอ):

```bash
git restore --staged --worktree app.js
```

flag `--worktree` (เป็นค่า default ของ `git restore` เมื่อไม่ใส่ `--staged` อยู่แล้ว) เมื่อรวมกับ `--staged` จะสั่งให้ restore ทั้งสองพื้นที่พร้อมกันในคำสั่งเดียว เทียบเท่ากับการรัน `restore` ปกติตามด้วย `restore --staged` แต่ในทางปฏิบัติ **แนะนำให้แยกทำทีละขั้นเสมอเมื่อไม่แน่ใจ** เพราะปลอดภัยกว่าและช่วยให้ตรวจสอบได้ก่อนสูญเสียข้อมูล

---

## Step 134: `git reset --soft <commit>` — ย้อน Commit แต่เก็บการเปลี่ยนแปลงไว้ใน Staging Area

เริ่มจาก Step นี้เป็นต้นไป เราเข้าสู่โลกของ **`git reset`** ซึ่งทำงานที่ **ระดับ commit history** โดยตรง แตกต่างจาก `restore` ที่ทำงานแค่ระหว่าง working directory กับ staging area เท่านั้น

### reset ทำอะไรโดยรวม (ก่อนแยกเป็น flag)

`git reset <commit>` คือคำสั่งที่ **ย้าย pointer ของ branch ปัจจุบัน (และ HEAD) ไปยัง commit ที่ระบุ** พูดง่าย ๆ คือ "สอนให้ Git ลืมไปว่ามี commit หลังจากจุดนี้อยู่" — แต่ **สิ่งที่เกิดขึ้นกับ staging area และ working directory นั้นขึ้นอยู่กับ flag ที่ใช้** ซึ่งมี 3 แบบหลักคือ `--soft`, `--mixed` (default), `--hard`

### ไดอะแกรมก่อน reset

สมมติคุณมีประวัติ commit แบบนี้:

```
A---B---C  (HEAD -> main)
```

โดย commit `C` คือ commit ล่าสุดที่คุณเพิ่งสร้างไป แต่พบว่า **ข้อความ commit ผิด หรืออยากรวม C เข้ากับ B เป็น commit เดียว** คุณจึงต้องการ "ย้อน" กลับไปที่ commit `B`

### คำสั่ง `reset --soft`

```bash
git reset --soft HEAD~1
```

หรือระบุ commit hash ตรง ๆ ก็ได้:

```bash
git reset --soft b2c3d4e
```

### ผลลัพธ์หลัง `reset --soft`

```
A---B  (HEAD -> main)
     \
      C  (ไม่มี branch ไหนชี้ไปแล้ว แต่ยังไม่ถูกลบทันที)
```

- **HEAD และ branch pointer ถูกย้ายไปที่ `B`** — Git มองว่า commit `C` "ไม่อยู่ใน branch นี้แล้ว"
- **Staging Area ไม่ถูกแก้ไข** — ยังคงมีเนื้อหาเดิม (สถานะก่อน reset)
- **แต่เพราะ HEAD ขยับไปที่ B แล้ว** เมื่อ Git เทียบ staging area กับ HEAD ใหม่ (คือ B) มันจะเห็นว่า **ทุกอย่างที่เคยอยู่ใน commit C ตอนนี้กลายเป็น "การเปลี่ยนแปลงที่ staged พร้อม commit"**

ลองดูตัวอย่างจริง:

```bash
$ git log --oneline
c3d4e5f (HEAD -> main) แก้ typo ในข้อความ commit ที่ผิดฟอร์แมต
b2c3d4e เพิ่มฟีเจอร์ login
a1b2c3d commit แรกของโปรเจกต์

$ git reset --soft HEAD~1

$ git log --oneline
b2c3d4e (HEAD -> main) เพิ่มฟีเจอร์ login
a1b2c3d commit แรกของโปรเจกต์

$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   auth.js
```

สังเกตว่า commit `c3d4e5f` หายไปจาก log แล้ว แต่การเปลี่ยนแปลงทั้งหมดที่เคยอยู่ใน commit นั้น **ปรากฏเป็น staged changes ทันที** พร้อมให้คุณ `git commit` ใหม่ด้วยข้อความที่ถูกต้อง

```bash
$ git commit -m "แก้ typo ในข้อความ commit"
```

### กรณีใช้งานที่พบบ่อยที่สุดของ `--soft`: รวมหลาย commit เป็นหนึ่งเดียว (Squash แบบง่าย)

สมมติคุณ commit ย่อย ๆ ไว้ 3 ครั้งระหว่างพัฒนาฟีเจอร์เดียว แต่อยากรวมเป็น commit เดียวก่อน push:

```bash
$ git log --oneline
f6a7b8c (HEAD -> feature/login) แก้ไข typo
e5f6a7b เพิ่ม validation
d4e5f6a เริ่มเขียนฟอร์ม login
c3d4e5f (main) จุดเริ่มต้น branch

$ git reset --soft c3d4e5f

$ git status
On branch feature/login
Changes to be committed:
        new file:   login.html
        new file:   login.js
        modified:   style.css

$ git commit -m "เพิ่มฟีเจอร์ login พร้อม validation"
```

3 commit ย่อยถูกรวมเป็น 1 commit เดียวที่สะอาดขึ้น โดยที่ **ไม่ต้องแก้ไขไฟล์ใหม่แม้แต่บรรทัดเดียว** เพราะทุกอย่างที่เคย commit ไว้ยังอยู่ครบใน staging area

> **หมายเหตุ:** วิธีนี้เป็นวิธี squash แบบพื้นฐานที่สุด ในทางปฏิบัติเมื่อทำงานกับหลาย branch ซับซ้อนขึ้น มักใช้ `git rebase -i` แทน ซึ่งเราจะเรียนอย่างละเอียดใน **Part 24: Interactive Rebase**

### จุดสำคัญที่ต้องจำ

> **`git reset --soft` คือ flag ที่ "อ่อนโยนที่สุด" ในบรรดา reset ทั้งหมด — มันแค่ย้าย HEAD/branch pointer เท่านั้น ไม่แตะ staging area และไม่แตะ working directory เลย**

---

## Step 135: `git reset --mixed <commit>` — ย้อน Commit และ Unstage แต่เก็บไฟล์ไว้ใน Working Directory (ค่า Default)

### ข้อเท็จจริงสำคัญที่ต้องรู้ก่อน

**`--mixed` คือค่า default ของ `git reset`** นั่นหมายความว่าถ้าคุณพิมพ์คำสั่งนี้โดยไม่ใส่ flag อะไรเลย:

```bash
git reset HEAD~1
```

**มันจะทำงานเหมือนกับ**:

```bash
git reset --mixed HEAD~1
```

**ทุกประการ** — นี่คือจุดที่มือใหม่มักพลาดโดยไม่รู้ตัว เพราะคิดว่า `git reset` เฉย ๆ "ปลอดภัย" หรือ "ไม่ทำอะไรมาก" ทั้งที่จริง ๆ มันได้ **ย้าย HEAD และรีเซ็ต staging area ไปแล้ว**

### ผลลัพธ์ของ `reset --mixed`

ใช้ประวัติเดิมจาก Step 134:

```
A---B---C  (HEAD -> main)
```

```bash
git reset --mixed HEAD~1
# หรือเขียนสั้น ๆ ว่า
git reset HEAD~1
```

ผลลัพธ์:

```
A---B  (HEAD -> main)
```

- **HEAD และ branch pointer ถูกย้ายไปที่ `B`** — เหมือนกับ `--soft`
- **Staging Area ถูกรีเซ็ตให้ตรงกับ commit `B`** — นี่คือความต่างจาก `--soft`! การเปลี่ยนแปลงที่เคยอยู่ใน `C` จะ **ไม่ถูก stage ไว้แล้ว**
- **Working Directory ไม่ถูกแตะต้อง** — ไฟล์บนดิสก์ยังคงมีเนื้อหาเหมือนตอนอยู่ที่ commit `C` เป๊ะ ๆ เพียงแต่ตอนนี้ Git มองว่ามันเป็น **"การแก้ไขที่ยังไม่ได้ stage" (unstaged changes)**

ตัวอย่างจริง:

```bash
$ git log --oneline
c3d4e5f (HEAD -> main) เพิ่มฟีเจอร์ login (ยังไม่สมบูรณ์)
b2c3d4e commit ก่อนหน้า
a1b2c3d commit แรกของโปรเจกต์

$ git reset --mixed HEAD~1

$ git log --oneline
b2c3d4e (HEAD -> main) commit ก่อนหน้า
a1b2c3d commit แรกของโปรเจกต์

$ git status
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   auth.js
        new file:   login.html
```

สังเกตความต่างจาก `--soft` ชัดเจน: ข้อความบอกว่า **"Changes not staged for commit"** ไม่ใช่ **"Changes to be committed"** และไฟล์ `login.html` ที่เคยเป็น "new file" ที่ staged ก็ตกกลับไปเป็น **untracked file** แทน (เพราะ mixed reset ล้าง staging area ทั้งหมด)

```bash
$ git status
On branch main
Changes not staged for commit:
        modified:   auth.js
Untracked files:
        login.html
```

### กรณีใช้งานที่พบบ่อยที่สุดของ `--mixed`

1. **"แกะ" commit ล่าสุดออกมาแก้ไขใหม่ทั้งหมด** — เมื่อคุณ commit เร็วเกินไป อยากเอาไฟล์กลับมาแก้ไขเพิ่มก่อน commit ใหม่แบบละเอียด
2. **แยก commit เดียวที่ใหญ่เกินไปออกเป็นหลาย commit ย่อย** — reset กลับไปก่อน commit นั้น แล้ว `git add` ทีละไฟล์/ทีละส่วน แล้ว commit แยกกันใหม่ (`git add -p` มีประโยชน์มากในสถานการณ์นี้ ซึ่งเราจะเรียนละเอียดใน Part 11)
3. **Unstage ทุกไฟล์พร้อมกัน** — `git reset` (ไม่ระบุ commit จะหมายถึง `HEAD` เป็น target) เทียบเท่ากับ `git restore --staged .`

```bash
# unstage ทุกไฟล์ที่ stage ไว้ กลับไปที่ HEAD ปัจจุบัน (ไม่ย้อน commit ใด ๆ)
git reset
```

> **ข้อสังเกต:** เมื่อ `git reset` ไม่ระบุ commit เป้าหมาย มันจะหมายถึง `HEAD` โดย default ซึ่งแปลว่า **HEAD ไม่ขยับไปไหนเลย** (เพราะ target คือตำแหน่งเดิม) ผลลัพธ์จึงเหลือแค่ "รีเซ็ต staging area ให้ตรงกับ HEAD" เท่านั้น นี่คือวิธี unstage ทุกไฟล์ที่นิยมใช้กันมานาน ก่อนที่ `git restore --staged .` จะถูกเพิ่มเข้ามาทีหลัง

### จุดสำคัญที่ต้องจำ

> **`git reset --mixed` (หรือ `git reset` เฉย ๆ) ย้าย HEAD และล้าง staging area ให้ตรงกับ commit ปลายทาง แต่ "ไม่แตะ" ไฟล์ในดิสก์เลย — งานของคุณยังอยู่ครบ เพียงแค่กลายเป็นสถานะ unstaged**

---

## Step 136: `git reset --hard <commit>` — ย้อนทุกอย่างและลบทิ้งถาวร

> ## ⚠️ คำเตือนอันตรายที่สุดใน Part นี้
>
> **`git reset --hard` คือคำสั่งที่ทำลายข้อมูลได้จริงและถาวรที่สุดในบรรดาคำสั่งพื้นฐานทั้งหมดของ Git** มันจะ **ลบการเปลี่ยนแปลงในไฟล์ที่ยังไม่ได้ commit ทิ้งไปเลยโดยไม่มีการถามยืนยันซ้ำ** และ **ไม่มีทางกู้คืนกลับมาได้** ถ้าการเปลี่ยนแปลงนั้นไม่เคยถูก commit หรือ stash ไว้ที่ไหนมาก่อน
>
> **ก่อนสั่งคำสั่งนี้ทุกครั้ง ให้ถามตัวเองก่อนเสมอว่า:** "ถ้าไฟล์พวกนี้หายไปตอนนี้เลย ฉันรับได้ไหม" ถ้าไม่แน่ใจ **ให้ `git stash` หรือ `git commit` เก็บไว้ก่อนเสมอ**

### ผลลัพธ์ของ `reset --hard`

ใช้ประวัติเดิม:

```
A---B---C  (HEAD -> main)
```

```bash
git reset --hard HEAD~1
```

ผลลัพธ์:

```
A---B  (HEAD -> main)
```

- **HEAD และ branch pointer ถูกย้ายไปที่ `B`** — เหมือนกับ `--soft` และ `--mixed`
- **Staging Area ถูกรีเซ็ตให้ตรงกับ commit `B`** — เหมือนกับ `--mixed`
- **Working Directory ก็ถูกรีเซ็ตให้ตรงกับ commit `B` ด้วย** — นี่คือความต่างที่สำคัญที่สุด! **ไฟล์บนดิสก์จริง ๆ จะถูกเขียนทับ/ลบทิ้งให้ตรงกับ snapshot ของ commit `B` ทันที**

ตัวอย่างจริงที่แสดงความอันตราย:

```bash
$ echo "โค้ดสำคัญมากที่ยังไม่ commit" >> auth.js
$ git status
On branch main
Changes not staged for commit:
        modified:   auth.js

$ git reset --hard HEAD
HEAD is now at b2c3d4e commit ก่อนหน้า

$ cat auth.js
# "โค้ดสำคัญมากที่ยังไม่ commit" หายไปแล้ว! ไม่มีทางกู้คืนจากบรรทัดนี้อีก
```

สังเกตว่าในตัวอย่างนี้ **ไม่ได้แม้แต่จะย้อน commit** (target คือ `HEAD` ตำแหน่งเดิม) แต่แค่สั่ง `reset --hard HEAD` เฉย ๆ ก็เพียงพอที่จะ **ลบการแก้ไขที่ยังไม่ได้ commit ในไฟล์นั้นทิ้งไปแล้วทันที** นี่คือกับดักที่คนใช้ผิดพลาดบ่อยที่สุด: ใช้ `reset --hard` เพื่อ "ล้างของเก่าทิ้งกลับไปเป็น commit ล่าสุด" โดยลืมไปว่าตัวเองมีงานที่ยังไม่ได้ commit ค้างอยู่

### เมื่อไหร่ที่ `--hard` เหมาะสมจริง ๆ

แม้จะอันตราย แต่ `reset --hard` ก็มีประโยชน์มากในบางสถานการณ์ **เฉพาะเมื่อคุณแน่ใจ 100% ว่าไม่ต้องการเก็บอะไรไว้เลย**:

1. **ทิ้งการทดลองที่พังทั้งหมดกลับไปที่จุดเริ่มต้น** เมื่อคุณลองทำอะไรบางอย่างในเครื่องตัวเองคนเดียว แล้วรู้แน่ชัดว่าอยากทิ้งทุกอย่างกลับไปจุดก่อนหน้า
2. **ทำให้ local branch ตรงกับ remote branch เป๊ะ ๆ** เมื่อ local เกิดความสับสนกับ commit ที่ไม่ต้องการ:
   ```bash
   git fetch origin
   git reset --hard origin/main
   ```
3. **หลังจากยกเลิก merge/rebase ที่ทำพลาดไปแล้วและยังไม่ได้ push ที่ไหน**

### วิธีป้องกันตัวเองก่อนใช้ `--hard`

**เทคนิคที่ 1: เช็คสถานะก่อนเสมอ**

```bash
git status
git diff
```

ดูให้แน่ใจว่ามีอะไรค้างอยู่บ้างก่อนสั่ง reset --hard

**เทคนิคที่ 2: ใช้ `git stash` เก็บไว้ก่อนเผื่อพลาด**

```bash
git stash push -u -m "backup ก่อน reset --hard เผื่อพลาด"
git reset --hard HEAD~1
# ถ้าพลาดจริง กู้คืนได้ด้วย
git stash pop
```

(เราจะเรียนเรื่อง `git stash` แบบละเอียดใน **Part 15**)

**เทคนิคที่ 3: รู้จัก `git reflog` — เครือข่ายความปลอดภัยสุดท้าย**

ข่าวดีคือ แม้ `reset --hard` จะดูน่ากลัวมาก แต่ **ถ้าการเปลี่ยนแปลงนั้นเคย commit ไปแล้วอย่างน้อยหนึ่งครั้ง** (แม้จะโดน reset ทิ้งไปแล้ว) Git ก็ยังมีบันทึกภายในชื่อ **reflog** ที่เก็บประวัติการขยับของ HEAD ไว้ชั่วคราว (โดย default ประมาณ 90 วัน) ทำให้กู้คืนกลับมาได้ในหลายกรณี:

```bash
$ git reflog
b2c3d4e (HEAD -> main) HEAD@{0}: reset: moving to HEAD~1
c3d4e5f HEAD@{1}: commit: เพิ่มฟีเจอร์ login (commit ที่ถูก reset ทิ้งไป)
b2c3d4e HEAD@{2}: commit: commit ก่อนหน้า

$ git reset --hard c3d4e5f
HEAD is now at c3d4e5f เพิ่มฟีเจอร์ login
```

> **ข้อควรระวังสำคัญ:** `reflog` ช่วยกู้คืน **commit ที่เคยมีอยู่** ได้เท่านั้น มันช่วย **ไม่ได้เลย** กับการเปลี่ยนแปลงที่ **ไม่เคยถูก commit หรือ stash มาก่อน** เพราะสิ่งเหล่านั้นไม่เคยถูกบันทึกเข้าไปในฐานข้อมูลของ Git เลยตั้งแต่แรก — นี่คือเหตุผลที่คำแนะนำ "commit บ่อย ๆ" หรือ "stash ก่อนทำอะไรเสี่ยง ๆ" ถึงสำคัญมาก เราจะเรียนเรื่อง `reflog` แบบละเอียดเต็ม Part ใน **Part 26: Git Reflog และการกู้คืนข้อมูลขั้นสูง**

### จุดสำคัญที่ต้องจำ

> **`git reset --hard` ย้าย HEAD, ล้าง staging area, และเขียนทับ working directory ให้ตรงกับ commit ปลายทางทั้งหมด — การเปลี่ยนแปลงใด ๆ ที่ยังไม่เคย commit หรือ stash จะหายไปทันทีและกู้คืนไม่ได้**

---

## Step 137: ตารางเปรียบเทียบ soft / mixed / hard แบบสรุปที่สุด

นี่คือตารางที่ควรจดจำ หรือแม้แต่ปริ้นท์ติดผนังไว้ เพราะเป็นหัวใจของ Part นี้ทั้งหมด

### ตารางหลัก: ผลกระทบต่อแต่ละระดับ

| Flag | HEAD / Branch Pointer ขยับไหม | Staging Area (Index) เปลี่ยนไหม | Working Directory เปลี่ยนไหม | ระดับความเสี่ยง |
|---|:---:|:---:|:---:|---|
| `--soft` | **ขยับ** ไปยัง commit ปลายทาง | **ไม่เปลี่ยน** (ยังคงค่าตามที่เป็นอยู่ก่อน reset — เมื่อเทียบกับ HEAD ใหม่จะกลายเป็น "staged") | **ไม่เปลี่ยน** | ต่ำ |
| `--mixed` (default) | **ขยับ** ไปยัง commit ปลายทาง | **เปลี่ยน** — ถูกรีเซ็ตให้ตรงกับ commit ปลายทาง (การเปลี่ยนแปลงกลายเป็น "unstaged") | **ไม่เปลี่ยน** | ปานกลาง |
| `--hard` | **ขยับ** ไปยัง commit ปลายทาง | **เปลี่ยน** — ถูกรีเซ็ตให้ตรงกับ commit ปลายทาง | **เปลี่ยน** — ถูกเขียนทับให้ตรงกับ commit ปลายทาง **(ข้อมูลที่ยังไม่ commit หายถาวร)** | **สูงมาก** |

### ตารางที่ 2: สถานะของไฟล์หลัง reset (เทียบก่อน-หลัง)

สมมติก่อน reset มีไฟล์ `X.js` ที่ถูกแก้ไขอยู่ใน commit ล่าสุดที่กำลังจะถูกย้อนออกไป:

| Flag | สถานะของการเปลี่ยนแปลงใน `X.js` หลัง reset | คำสั่งที่ต้องพิมพ์เพื่อ commit ใหม่ |
|---|---|---|
| `--soft` | อยู่ใน **Staging Area** พร้อม commit ทันที | `git commit -m "..."` เท่านั้น |
| `--mixed` | อยู่ใน **Working Directory** แบบ unstaged | `git add X.js` แล้วค่อย `git commit -m "..."` |
| `--hard` | **หายไปแล้ว** ไม่มีอยู่ทั้งใน staging และ working directory | ไม่มี — ต้องเขียนใหม่ทั้งหมด (หรือกู้จาก reflog ถ้ายังกู้ได้) |

### ไดอะแกรมเปรียบเทียบแบบเห็นภาพ

```
ก่อน reset:  A---B---C  (HEAD -> main)   staging = C, working dir = C

reset --soft B:
             A---B  (HEAD -> main)
             staging area: ยังคงเป็นเนื้อหาของ C (มองว่า staged เทียบกับ B)
             working dir:  ยังคงเป็นเนื้อหาของ C
             ผลคือ: "การเปลี่ยนแปลงของ C" กลายเป็นสถานะ "รอ commit"

reset --mixed B:
             A---B  (HEAD -> main)
             staging area: ถูกรีเซ็ตให้ตรงกับ B
             working dir:  ยังคงเป็นเนื้อหาของ C
             ผลคือ: "การเปลี่ยนแปลงของ C" กลายเป็นสถานะ "แก้ไขแล้ว ยังไม่ stage"

reset --hard B:
             A---B  (HEAD -> main)
             staging area: ถูกรีเซ็ตให้ตรงกับ B
             working dir:  ถูกรีเซ็ตให้ตรงกับ B ด้วย
             ผลคือ: "การเปลี่ยนแปลงของ C" หายไปจากทุกที่ (เว้นแต่กู้จาก reflog)
```

### คำจำง่าย ๆ สำหรับจำ 3 flag นี้ตลอดไป

> - **`--soft`** = "ใจดีที่สุด" ย้ายแค่ตัวชี้ประวัติ งานทุกอย่างยัง **staged** พร้อม commit ใหม่ทันที
> - **`--mixed`** (default) = "กลาง ๆ" ย้ายตัวชี้ + ล้าง staging แต่ **ไฟล์ในดิสก์ยังอยู่ครบ** แค่กลายเป็น unstaged
> - **`--hard`** = "โหดที่สุด" ย้ายตัวชี้ + ล้าง staging + **ลบไฟล์ในดิสก์ทิ้งด้วย** อันตรายที่สุด ต้องมั่นใจก่อนใช้เสมอ

### เกร็ดความรู้เสริม: reset ไฟล์เฉพาะ (path-limited reset)

`git reset` ยังใช้ระบุไฟล์เฉพาะได้ ซึ่งจะทำงานในโหมด `--mixed` เท่านั้นเสมอ (ไม่รองรับ `--soft`/`--hard` เมื่อระบุ path):

```bash
git reset HEAD~1 -- auth.js
```

คำสั่งนี้จะ unstage เฉพาะไฟล์ `auth.js` ให้ตรงกับเนื้อหาที่ `HEAD~1` **โดยไม่ขยับ HEAD/branch pointer เลย** (เพราะเมื่อระบุ path Git จะไม่ย้าย branch ให้ ต่างจาก reset แบบไม่ระบุ path) นี่คือความแตกต่างเล็ก ๆ ที่ควรรู้ไว้ แต่ไม่ใช่แกนหลักของ Part นี้

---

## Step 138: `git revert <commit>` — ย้อน Commit แบบปลอดภัยสำหรับ Branch ที่แชร์กับคนอื่น

### ปัญหาของ `reset` ที่ `revert` มาแก้

ทั้ง `reset --soft`, `--mixed`, และ `--hard` มีจุดร่วมกันอย่างหนึ่งคือ **มันทำงานโดยการ "ย้าย" ตัวชี้ branch ไปยัง commit เก่า** ซึ่งหมายความว่า **commit ที่เคยอยู่หลังจากจุดนั้นจะหายไปจาก branch history โดยสิ้นเชิง** (แม้จะยังพอกู้จาก reflog ได้ในเครื่องตัวเอง)

ปัญหาใหญ่เกิดขึ้นเมื่อ **commit เหล่านั้นถูก push ขึ้น remote ไปแล้ว และมีคนอื่นในทีม `git pull` ไปใช้งานแล้ว** — ถ้าคุณ `reset` แล้ว force push (`git push --force`) ทับประวัติ:

- เพื่อนร่วมทีมที่ pull ไปแล้วจะมีประวัติที่ **ไม่ตรงกับ remote** ทันที
- เกิด conflict ประวัติที่ยุ่งเหยิงมากเมื่อพวกเขาพยายาม pull/push อีกครั้ง
- ในกรณีเลวร้ายที่สุด **งานของคนอื่นอาจหายไปโดยไม่ได้ตั้งใจ** ถ้าทำ force push ทับแบบไม่ระวัง

`git revert` ถูกออกแบบมาเพื่อแก้ปัญหานี้โดยเฉพาะ ด้วยหลักการที่ต่างไปอย่างสิ้นเชิง:

> **`git revert` ไม่ลบหรือย้อนประวัติใด ๆ เลย แต่จะสร้าง commit ใหม่ขึ้นมาเพิ่มในประวัติ ที่มีเนื้อหาตรงกันข้ามกับ commit ที่ต้องการย้อน**

### ไดอะแกรมเปรียบเทียบ reset กับ revert

```
ก่อนทำอะไร:     A---B---C  (HEAD -> main)

หลัง reset --hard B:
                A---B  (HEAD -> main)
                (C หายไปจาก branch history เลย)

หลัง revert C:
                A---B---C---D  (HEAD -> main)
                            ↑
                  D คือ commit ใหม่ที่ "ทำตรงข้าม" กับ C
                (ประวัติทั้งหมดยังอยู่ครบ ไม่มีอะไรถูกลบ)
```

### คำสั่งพื้นฐาน

```bash
git revert <commit-hash>
```

ตัวอย่างจริง:

```bash
$ git log --oneline
c3d4e5f (HEAD -> main) เพิ่มฟีเจอร์ที่ทำให้เว็บพัง
b2c3d4e เพิ่มฟีเจอร์ login
a1b2c3d commit แรกของโปรเจกต์

$ git revert c3d4e5f
```

Git จะเปิด editor ให้แก้ไขข้อความ commit ของ revert (มีค่า default ให้แล้ว เช่น `Revert "เพิ่มฟีเจอร์ที่ทำให้เว็บพัง"`) บันทึกแล้วปิด editor ผลลัพธ์:

```bash
$ git log --oneline
e5f6a7b (HEAD -> main) Revert "เพิ่มฟีเจอร์ที่ทำให้เว็บพัง"
c3d4e5f เพิ่มฟีเจอร์ที่ทำให้เว็บพัง
b2c3d4e เพิ่มฟีเจอร์ login
a1b2c3d commit แรกของโปรเจกต์
```

สังเกตว่า **commit `c3d4e5f` ยังอยู่ในประวัติเหมือนเดิมทุกอย่าง** — ไม่มีการลบทิ้ง แต่มี commit ใหม่ `e5f6a7b` เพิ่มเข้ามา ซึ่งเนื้อหาไฟล์จะกลับไปเหมือนกับก่อนที่ `c3d4e5f` จะถูกสร้าง

### revert แบบไม่ commit ทันที (ตรวจสอบก่อน)

```bash
git revert --no-commit c3d4e5f
# หรือย่อว่า
git revert -n c3d4e5f
```

คำสั่งนี้จะทำการ revert ให้ในระดับ staging area และ working directory แต่ **ไม่สร้าง commit ให้ทันที** ทำให้คุณตรวจสอบผลลัพธ์ก่อนได้ด้วย `git diff --staged` แล้วค่อยสั่ง `git commit` เองเมื่อพร้อม (มีประโยชน์มากเมื่อต้อง revert หลาย commit พร้อมกันแล้วอยากรวมเป็น commit เดียว)

### revert commit ที่เป็น merge commit

การ revert commit ที่เกิดจากการ merge (มี parent 2 ตัว) ต้องระบุด้วยว่าจะให้ยึด parent ตัวไหนเป็นหลัก โดยใช้ flag `-m`:

```bash
git revert -m 1 <merge-commit-hash>
```

(`-m 1` หมายถึงยึด parent ตัวแรก ซึ่งปกติคือ branch ที่ merge เข้ามา เช่น `main` — เรื่องนี้จะลงรายละเอียดเพิ่มเติมใน Part ที่ว่าด้วย merge conflict ขั้นสูง)

### revert หลาย commit พร้อมกัน (range)

```bash
git revert HEAD~3..HEAD
```

คำสั่งนี้จะ revert 3 commit ล่าสุด **ทีละ commit ตามลำดับย้อนจากใหม่ไปเก่า** สร้าง commit revert แยกกันหลายอัน (เว้นแต่จะใช้ `--no-commit` ร่วมด้วยเพื่อรวมเป็นครั้งเดียว)

### ถ้าเกิด conflict ระหว่าง revert

`git revert` ก็สามารถเกิด conflict ได้เหมือนการ merge ปกติ ถ้าไฟล์ที่เกี่ยวข้องถูกแก้ไขทับซ้อนโดย commit หลังจากนั้น ในกรณีนี้ Git จะหยุดกลางคันและให้คุณแก้ conflict เอง:

```bash
$ git revert c3d4e5f
error: could not revert c3d4e5f... เพิ่มฟีเจอร์ที่ทำให้เว็บพัง
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'

# แก้ conflict ในไฟล์ที่ขึ้น <<<<<<< / ======= / >>>>>>> แล้ว
$ git add <file>
$ git revert --continue
```

หรือถ้าต้องการยกเลิกการ revert กลางคัน:

```bash
git revert --abort
```

### จุดสำคัญที่ต้องจำ

> **`git revert` ไม่เคยลบประวัติ ไม่เคยบังคับให้เพื่อนร่วมทีม force pull และไม่ทำลายข้อมูลของใครเลย มันแค่เพิ่ม commit ใหม่เข้าไปในประวัติที่ "หักล้าง" ผลของ commit เก่า — จึงเป็นวิธีที่ปลอดภัยที่สุดเสมอเมื่อ commit ที่จะย้อนถูก push ไปแล้วและมีคนอื่นดึงไปใช้**

---

## Step 139: Reset vs Revert — เมื่อไหร่ควรใช้อันไหน (กฎทองที่ห้ามฝ่าฝืน)

### กฎทองข้อที่สำคัญที่สุดในทั้งหลักสูตรนี้

> ## 🔑 กฎทอง: ห้าม `reset --hard` หรือแก้ไขประวัติ (rewrite history) ของ commit ที่ถูก push ไปแล้วและมีคนอื่นดึง (pull/fetch) ไปใช้งานแล้วเด็ดขาด

เหตุผลเชิงเทคนิคที่อยู่เบื้องหลังกฎนี้: `git reset` (ทุก flag) และคำสั่งอื่นที่ "เขียนประวัติใหม่" เช่น `git rebase`, `git commit --amend` (ซึ่งเราจะเรียนใน Part หลัง ๆ) ล้วนทำงานโดย **สร้าง commit ใหม่ที่มี hash ต่างไปจากเดิม แล้วย้าย branch pointer ไปชี้ที่ commit ใหม่นั้นแทน** เมื่อคุณ force push (`git push --force` หรือ `--force-with-lease`) ประวัติที่ remote จะถูกเขียนทับ

ปัญหาคือ **ทุกคนที่เคย pull ประวัติเก่าไปแล้ว จะมีประวัติที่ไม่ตรงกับ remote อีกต่อไป** เมื่อพวกเขาพยายาม pull ครั้งถัดไป Git จะมองว่าประวัติ diverge กันอย่างรุนแรง และอาจนำไปสู่:

- ข้อความ error ที่งุนงงเกี่ยวกับ "non-fast-forward" หรือ diverged branches
- การ merge ที่ยุ่งเหยิงมาก เพราะ Git พยายามผสานประวัติสองเวอร์ชันที่ไม่เกี่ยวข้องกันจริง ๆ
- ในกรณีที่แย่ที่สุด สมาชิกในทีมอาจ force push ทับกลับไปมาจนงานของคนอื่นหายจริง ๆ

### ตารางตัดสินใจ: ควรใช้ reset หรือ revert

| สถานการณ์ | ควรใช้ | เหตุผล |
|---|---|---|
| Commit ยังอยู่แค่ในเครื่องตัวเอง ไม่เคย push | `reset` (soft/mixed/hard ตามต้องการ) | ไม่มีใครได้รับผลกระทบ ปลอดภัย 100% |
| Commit push ไปแล้ว แต่เป็น branch ส่วนตัว (feature branch) ที่รู้แน่ชัดว่าไม่มีใครดึงไปใช้ | `reset` + force push ได้ (ระวังและแจ้งทีมก่อนเสมอ) | ยังพอควบคุมผลกระทบได้ ถ้าแน่ใจจริง ๆ ว่าไม่มีใครแตะ |
| Commit push ไปแล้วบน branch หลัก (`main`/`master`/`develop`) ที่ทีมใช้ร่วมกัน | **`revert` เท่านั้น** | ห้ามเขียนประวัติทับ branch ที่แชร์กันเด็ดขาด |
| Commit ถูก merge เข้า production แล้วพบบั๊กร้ายแรงต้องแก้ด่วน | **`revert`** | ปลอดภัย รวดเร็ว ตรวจสอบย้อนหลังได้ว่า "เคยมีบั๊กอะไร แก้ยังไง" |
| อยากรวม (squash) commit ย่อยหลายอันในเครื่องตัวเองก่อน push ครั้งแรก | `reset --soft` | ยังไม่ push จึงไม่กระทบใคร |
| อยากลบไฟล์ที่ commit ผิดพลาด (เช่น secret) ที่ยัง**ไม่**ได้ push | `reset --hard` หรือ `--mixed` แล้วลบไฟล์ | ยังไม่ push แก้ไขได้อิสระ |
| อยากลบไฟล์ลับ (secret) ที่ **push ไปแล้ว** | ต้องใช้เครื่องมือขั้นสูงกว่า เช่น `git filter-repo` และ **หมุนเปลี่ยนค่า secret นั้นทันที** | `revert` เพียงอย่างเดียวไม่พอ เพราะ secret ยังอยู่ในประวัติเก่า ต้องลบออกจริง ๆ (เรียนละเอียดใน Part ความปลอดภัยขั้นสูงภายหลัง) |

### วิธีเช็คว่า commit "push ไปแล้วหรือยัง" ก่อนตัดสินใจ

```bash
# เทียบ branch ปัจจุบันกับ branch เดียวกันบน remote
git log origin/main..main --oneline
```

ถ้าคำสั่งนี้แสดง commit ออกมา แปลว่า commit เหล่านั้น **ยังไม่ถูก push** (มีอยู่ใน local แต่ไม่มีใน remote) — ปลอดภัยที่จะ `reset` ได้ตามสบาย

ถ้าคำสั่งนี้ไม่แสดงอะไรเลย (ว่างเปล่า) แปลว่า local กับ remote ตรงกันหมดแล้ว **ทุก commit ถูก push ไปแล้ว** — ถ้าจะย้อน ให้คิดถึง `revert` เป็นอันดับแรกเสมอ

### สรุปเป็นแผนภาพการตัดสินใจ

```
                   ต้องการย้อนการเปลี่ยนแปลงของ commit หนึ่ง
                                  │
                 ┌────────────────┴────────────────┐
                 │                                  │
         commit นี้ push ไปแล้ว              commit นี้ยังไม่เคย push
         และมีคนอื่นใช้ branch นี้ร่วมด้วย       (อยู่แค่ในเครื่องตัวเอง)
                 │                                  │
                 ▼                                  ▼
          ใช้ git revert                     ใช้ git reset
     (ปลอดภัยเสมอ ไม่ต้อง force push)   (เลือก soft/mixed/hard ตามต้องการ)
```

### ทำไมกฎนี้ถึงเรียกว่า "กฎทอง"

เพราะการฝ่าฝืนกฎนี้คือ **สาเหตุอันดับต้น ๆ ของอุบัติเหตุร้ายแรงที่เกิดขึ้นจริงในทีมพัฒนาซอฟต์แวร์ทั่วโลก** ไม่ว่าจะเป็นทีมเล็กหรือทีมใหญ่ระดับบริษัทเทคโนโลยียักษ์ใหญ่ ล้วนเคยมีเหตุการณ์ "งานของทีมหายเพราะมีคน force push ทับประวัติ" มาแล้วทั้งนั้น การจำกฎนี้ให้ขึ้นใจตั้งแต่ตอนนี้ จะช่วยป้องกันปัญหาใหญ่ในอนาคตได้อย่างมาก

> เราจะเรียนเรื่อง `git push --force` กับ `--force-with-lease` แบบละเอียด รวมถึงวิธีกู้คืนสถานการณ์ฉุกเฉินเมื่อมีคนฝ่าฝืนกฎนี้ไปแล้ว ใน **Part 25** ของหลักสูตร

---

## Step 140: แบบฝึกหัด — จำลอง 4 สถานการณ์จริงเพื่อฝึก restore, reset ทั้ง 3 แบบ และ revert

เตรียมโฟลเดอร์ฝึกฝนใหม่ก่อนเริ่ม:

```bash
mkdir -p ~/git-course/part-14-undo
cd ~/git-course/part-14-undo
git init
git config user.name "ผู้เรียน Git"
git config user.email "learner@example.com"
```

### สถานการณ์ที่ 1: แก้ไฟล์ผิดพลาด ต้องการยกเลิกด้วย `restore`

```bash
# สร้างไฟล์เริ่มต้นและ commit
echo "บรรทัดที่ 1: เนื้อหาต้นฉบับ" > notes.txt
git add notes.txt
git commit -m "เพิ่ม notes.txt เริ่มต้น"

# แก้ไขไฟล์แบบผิดพลาด (ยังไม่ add)
echo "บรรทัดที่ 2: แก้ไขผิดพลาดโดยไม่ตั้งใจ" >> notes.txt
cat notes.txt
git status
```

**ลงมือทำ:**

1. ใช้ `git diff notes.txt` ดูว่ากำลังจะเสียอะไรไป
2. ใช้ `git restore notes.txt` ยกเลิกการแก้ไข
3. ตรวจสอบด้วย `cat notes.txt` และ `git status` ว่ากลับไปเป็นเหมือนตอน commit เป๊ะ ๆ

**คำถามทบทวน:** ถ้าคุณ `git add notes.txt` ไปก่อนแล้วค่อยสั่ง `git restore notes.txt` (ไม่มี `--staged`) ผลลัพธ์จะเป็นอย่างไร? (คำตอบ: `restore` เฉย ๆ จะเทียบกับ staging area ไม่ใช่ HEAD ถ้าไฟล์ถูก stage ไปแล้ว มันจะดึงค่าที่ staged กลับมาที่ working directory ไม่ได้ดึงจาก HEAD)

### สถานการณ์ที่ 2: `git add .` พลาดไฟล์ที่ไม่ควร commit ต้อง `restore --staged`

```bash
# จำลองการสร้างไฟล์ 2 ไฟล์
echo "ฟีเจอร์ใหม่ที่พร้อม commit" > feature.js
echo "SECRET_KEY=abc123xyz" > .env

git add .
git status
```

**ลงมือทำ:**

1. สังเกตว่าทั้งสองไฟล์ถูก stage พร้อมกันหลัง `git add .`
2. ใช้ `git restore --staged .env` เพื่อเอาไฟล์ลับออกจาก staging
3. ตรวจสอบว่า `.env` **ยังคงอยู่ในโฟลเดอร์** (ไม่ได้ถูกลบ) ด้วยคำสั่ง `cat .env`
4. Commit เฉพาะ `feature.js`:
   ```bash
   git commit -m "เพิ่มฟีเจอร์ใหม่"
   ```
5. เพิ่ม `.env` เข้า `.gitignore` เพื่อป้องกันไม่ให้พลาดซ้ำในอนาคต:
   ```bash
   echo ".env" >> .gitignore
   git add .gitignore
   git commit -m "เพิ่ม .env เข้า gitignore"
   ```

**คำถามทบทวน:** ทำไมขั้นตอนนี้ถึงสำคัญมากในโลกการทำงานจริง? (คำตอบ: ถ้า `.env` ที่มี secret หลุดเข้าไปใน commit และถูก push ไปแล้ว การลบไฟล์ทิ้งในภายหลังไม่เพียงพอ เพราะ secret ยังอยู่ในประวัติเก่า ต้องหมุนเปลี่ยนค่า secret นั้นทันทีเสมอ)

### สถานการณ์ที่ 3: ทดลอง reset ทั้ง 3 แบบเปรียบเทียบกันในสถานการณ์เดียวกัน

สร้างประวัติ commit สำหรับทดลอง:

```bash
echo "v1" > data.txt
git add data.txt
git commit -m "commit A: เวอร์ชัน 1"

echo "v2" > data.txt
git add data.txt
git commit -m "commit B: เวอร์ชัน 2"

echo "v3" > data.txt
git add data.txt
git commit -m "commit C: เวอร์ชัน 3"

git log --oneline
```

**ส่วนที่ 3.1 — ทดลอง `--soft` (สร้าง branch แยกทดลองเพื่อไม่ปนกัน):**

```bash
git branch ทดลอง-soft
git switch ทดลอง-soft

git reset --soft HEAD~1
git status          # ควรเห็น "Changes to be committed"
git log --oneline   # commit C ควรหายไปจาก log
```

**ส่วนที่ 3.2 — ทดลอง `--mixed` (กลับไป main แล้วสร้าง branch ใหม่):**

```bash
git switch main
git branch ทดลอง-mixed
git switch ทดลอง-mixed

git reset --mixed HEAD~1
git status          # ควรเห็น "Changes not staged for commit"
git log --oneline   # commit C ควรหายไปจาก log เช่นกัน
```

**ส่วนที่ 3.3 — ทดลอง `--hard` (อันตราย — ทำใน branch ทดลองเท่านั้น):**

```bash
git switch main
git branch ทดลอง-hard
git switch ทดลอง-hard

cat data.txt         # ควรเห็น "v3"
git reset --hard HEAD~1
cat data.txt         # ควรเห็น "v2" — v3 หายไปจริง เพราะ working directory ถูกเขียนทับ
git status           # ควรขึ้น "nothing to commit, working tree clean"
```

**เปรียบเทียบผลลัพธ์ทั้ง 3 branch:**

```bash
git switch ทดลอง-soft
git status
echo "---"
git switch ทดลอง-mixed
git status
echo "---"
git switch ทดลอง-hard
git status
```

**คำถามทบทวน:** ทำไม branch `ทดลอง-soft` และ `ทดลอง-mixed` ถึงมี `git log --oneline` เหมือนกัน (ไม่เห็น commit C) แต่ `git status` กลับต่างกัน? (คำตอบ: ทั้งสองย้าย HEAD ไปที่ตำแหน่งเดียวกัน แต่ `--soft` ไม่แตะ staging area จึงเห็นเป็น staged ส่วน `--mixed` ล้าง staging area จึงเห็นเป็น unstaged)

เมื่อทดลองเสร็จ กลับไปที่ `main` และลบ branch ทดลองทิ้ง:

```bash
git switch main
git branch -D ทดลอง-soft ทดลอง-mixed ทดลอง-hard
```

### สถานการณ์ที่ 4: จำลอง Commit ที่ push ไปแล้วมีบั๊ก ต้องแก้ด้วย `revert` ไม่ใช่ `reset`

จำลอง remote repository ด้วย bare repo ในเครื่อง เพื่อให้เห็นภาพการ "push" จริง:

```bash
# สร้าง bare repo จำลอง remote
mkdir -p ~/git-course/part-14-undo-remote.git
git -C ~/git-course/part-14-undo-remote.git init --bare

# เชื่อม remote เข้ากับโปรเจกต์ทดลอง
git remote add origin ~/git-course/part-14-undo-remote.git
git push -u origin main
```

จำลองสถานการณ์ commit ที่มีบั๊กแล้ว push ไปแล้ว:

```bash
echo "function calculate() { return 1/0; }" >> data.txt
git add data.txt
git commit -m "เพิ่มฟังก์ชันคำนวณ (มีบั๊กร้ายแรง หารด้วยศูนย์)"
git push origin main
```

**ลงมือทำ:**

1. ตรวจสอบก่อนว่า commit นี้ push ไปแล้วจริงหรือไม่:
   ```bash
   git log origin/main..main --oneline
   ```
   (ควรว่างเปล่า เพราะ push ไปแล้ว)
2. **ห้ามใช้ `git reset --hard` แล้ว force push** ในสถานการณ์นี้ ให้ใช้ `git revert` แทน:
   ```bash
   git revert HEAD
   ```
   (บันทึกข้อความ commit ตามค่า default หรือแก้ไขให้ชัดเจนขึ้น เช่น `Revert "เพิ่มฟังก์ชันคำนวณ (มีบั๊กร้ายแรง หารด้วยศูนย์)"`)
3. ตรวจสอบว่าประวัติยังอยู่ครบ:
   ```bash
   git log --oneline
   ```
   ควรเห็นทั้ง commit เดิมที่มีบั๊ก **และ** commit revert ใหม่ที่แก้ไขมัน
4. Push commit revert ขึ้น remote ตามปกติ (ไม่ต้อง force):
   ```bash
   git push origin main
   ```

**คำถามทบทวน:** ถ้าคุณใช้ `git reset --hard HEAD~1` แทนในสถานการณ์นี้ แล้ว `git push --force origin main` จะเกิดอะไรขึ้นกับเพื่อนร่วมทีมที่เพิ่ง `git pull` commit ที่มีบั๊กไปก่อนหน้านี้แล้ว? (คำตอบ: ประวัติของพวกเขาจะไม่ตรงกับ remote ทันที เมื่อพยายาม pull/push ครั้งถัดไปจะเจอปัญหาประวัติ diverge และอาจนำไปสู่ conflict ที่ยุ่งเหยิง หรือแม้แต่งานของพวกเขาหายไป ถ้ามีการ force push ทับกลับไปมาโดยไม่ระวัง)

### สรุปแบบฝึกหัดทั้ง 4 สถานการณ์

| สถานการณ์ | คำสั่งหลักที่ใช้ | สิ่งที่ได้เรียนรู้ |
|---|---|---|
| 1. แก้ไฟล์ผิดพลาดที่ยังไม่ stage | `git restore` | ยกเลิกการแก้ไขใน working directory อย่างปลอดภัยเมื่อยังไม่เคย stage |
| 2. `git add .` พลาดไฟล์ลับ | `git restore --staged` | เอาไฟล์ออกจาก staging โดยไม่เสียเนื้อหาไฟล์ |
| 3. เปรียบเทียบ soft/mixed/hard | `git reset --soft/--mixed/--hard` | เห็นความต่างของผลกระทบต่อ staging area และ working directory ด้วยตาตัวเอง |
| 4. Commit ที่ push แล้วมีบั๊ก | `git revert` | ย้อนผลของ commit อย่างปลอดภัยโดยไม่ทำลายประวัติที่แชร์กับทีม |

---

## สรุป Part 14

ใน Part นี้เราได้เรียนรู้เรื่องที่สำคัญที่สุดเรื่องหนึ่งของการใช้ Git ในชีวิตจริง นั่นคือการ **Undo การเปลี่ยนแปลง** อย่างถูกต้องและปลอดภัย สรุปประเด็นสำคัญทั้งหมด:

1. **Git มีพื้นที่เก็บข้อมูล 3 ระดับ**: Working Directory → Staging Area → Repository (Commit History) และคำสั่ง undo แต่ละตัวทำงานคนละระดับ
2. **`git restore <file>`** ยกเลิกการแก้ไขใน working directory กลับไปเหมือน staged/HEAD — **ไม่แตะประวัติ commit เลย** และเป็นคำสั่งใหม่แทนที่ `git checkout -- <file>` แบบเก่า
3. **`git restore --staged <file>`** เอาไฟล์ออกจาก staging area **โดยไม่เสียการแก้ไขที่ทำไว้เลยแม้แต่นิดเดียว** เป็นคำสั่งใหม่แทนที่ `git reset HEAD <file>` แบบเก่า
4. **`git reset --soft <commit>`** ย้าย HEAD/branch pointer เท่านั้น การเปลี่ยนแปลงทั้งหมดยังอยู่ครบในสถานะ **staged** พร้อม commit ใหม่ทันที — เหมาะกับการรวม commit ย่อยหลายอันเข้าด้วยกัน
5. **`git reset --mixed <commit>`** (ค่า default ของ `git reset`) ย้าย HEAD และล้าง staging area แต่ **ไฟล์ในดิสก์ยังอยู่ครบ** กลายเป็นสถานะ unstaged
6. **`git reset --hard <commit>`** ย้าย HEAD, ล้าง staging area, **และเขียนทับ working directory ด้วย** — การเปลี่ยนแปลงที่ยังไม่เคย commit หรือ stash จะ **หายไปถาวรและกู้คืนไม่ได้** ต้องใช้ด้วยความระมัดระวังสูงสุดเสมอ
7. **ตารางเปรียบเทียบ soft/mixed/hard** คือหัวใจของ Part นี้: ทั้งสามย้าย HEAD เหมือนกัน แต่ต่างกันตรงว่า staging area และ working directory ถูกแตะต้องมากน้อยแค่ไหน
8. **`git revert <commit>`** ไม่ลบประวัติเลย แต่สร้าง commit ใหม่ที่ทำตรงข้ามกับ commit เดิม — เป็นวิธีที่ปลอดภัยที่สุดเมื่อทำงานกับ branch ที่แชร์กับคนอื่น
9. **กฎทองที่ห้ามฝ่าฝืน:** ห้ามใช้ `reset --hard` หรือคำสั่งใด ๆ ที่เขียนประวัติใหม่ (rewrite history) กับ commit ที่ถูก push ไปแล้วและมีคนอื่นดึงไปใช้งานแล้ว — ให้ใช้ `git revert` แทนเสมอในสถานการณ์นั้น
10. เราได้ลงมือฝึกจริงทั้ง 4 สถานการณ์: แก้ไฟล์ผิดพลาด, `add` พลาดไฟล์ลับ, เปรียบเทียบ reset ทั้ง 3 แบบ, และ revert commit ที่ push ไปแล้ว

จากนี้ไปเมื่อคุณเผลอทำอะไรพลาดใน Git คุณจะไม่ตื่นตระหนกอีกต่อไป เพราะรู้แล้วว่าควรใช้เครื่องมือไหนในแต่ละสถานการณ์ และรู้ขอบเขตความเสี่ยงของแต่ละคำสั่งอย่างแม่นยำ

**ต่อไป:** [Part 15: โปรเจกต์ฝึกหัด: สร้างโปรเจกต์เล็กด้วย Git ตั้งแต่ต้นจนจบ](./part-015-โปรเจกต์ฝึกหัด-จบเฟส-2.md)
