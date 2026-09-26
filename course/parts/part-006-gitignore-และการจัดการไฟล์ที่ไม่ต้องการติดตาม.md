# Part 06: .gitignore และการจัดการไฟล์ที่ไม่ต้องการติดตาม

> **Step ในหลักสูตรนี้:** Step 51–60
> **เฟส:** 2 — ใช้งาน Git ในชีวิตประจำวันอย่างคล่องแคล่ว
> **เป้าหมายของ Part นี้:** เข้าใจว่าทำไมโปรเจกต์จริงทุกโปรเจกต์ต้องมี `.gitignore` เขียน pattern พื้นฐานและขั้นสูงได้อย่างถูกต้อง รู้จักเทมเพลตสำเร็จรูป จัดการ ignore ได้หลายระดับ (global/per-repo/per-directory/local-only) แก้ปัญหาไฟล์ที่ track ไปแล้วโดยไม่ตั้งใจ และวินิจฉัยปัญหา ignore ที่พบบ่อยได้ด้วยเครื่องมือของ Git เอง

---

## สารบัญของ Part นี้

- Step 51: ทำไมต้องมี .gitignore — ปัญหาไฟล์ที่ไม่ควร track
- Step 52: สร้างไฟล์ .gitignore และ syntax พื้นฐาน
- Step 53: Pattern ขั้นสูงของ .gitignore (negation, `**`, comment)
- Step 54: เทมเพลตสำเร็จรูปจาก github/gitignore (Node.js, Python, Java)
- Step 55: .gitignore หลายระดับ (Global, Per-repo, Per-directory)
- Step 56: ไฟล์ที่ track ไปแล้วแต่อยากเพิ่มใน .gitignore ภายหลัง (git rm --cached)
- Step 57: .git/info/exclude — ignore เฉพาะเครื่อง ไม่แชร์กับทีม
- Step 58: ตรวจสอบว่าไฟล์ถูก ignore ด้วย git check-ignore และ git status --ignored
- Step 59: ข้อผิดพลาดที่พบบ่อยเกี่ยวกับ .gitignore
- Step 60: แบบฝึกหัด — สร้างโปรเจกต์ Node.js จำลอง เขียน .gitignore และแก้ปัญหาไฟล์หลุด track

---

## Step 51: ทำไมต้องมี .gitignore — ปัญหาไฟล์ที่ไม่ควร track

ใน Part ก่อนหน้าเราได้ฝึกใช้ `git add` และ `git commit` กันไปแล้ว แต่ในโปรเจกต์จริง ถ้าเรารัน `git add .` แบบไม่คิดอะไร มักจะเจอปัญหาใหญ่ตามมาทันที เพราะโฟลเดอร์โปรเจกต์ของเราไม่ได้มีแค่ "ซอร์สโค้ดที่เราตั้งใจเขียน" เท่านั้น แต่ยังมีไฟล์อีกจำนวนมากที่ถูกสร้างขึ้นโดยเครื่องมืออื่น ๆ ระหว่างการพัฒนาและรันโปรแกรม

### 51.1 ประเภทของไฟล์ที่ "ไม่ควร" ถูก track

ลองนึกภาพโปรเจกต์ Node.js ทั่วไปหลังจากรัน `npm install` และเริ่มพัฒนาไปสักพัก:

```
my-app/
├── node_modules/           ← ไลบรารีนับพันไฟล์ที่ npm install ให้อัตโนมัติ
├── dist/                   ← ไฟล์ที่ build แล้ว (compiled output)
├── .env                    ← ค่า secret เช่น API key, database password
├── .DS_Store               ← ไฟล์ metadata ของ macOS Finder
├── Thumbs.db               ← ไฟล์ cache รูปธัมบ์เนลของ Windows Explorer
├── npm-debug.log           ← log ที่เกิดจากการรัน npm ที่ error
├── coverage/                ← รายงานผลการทดสอบ (test coverage report)
├── .vscode/                ← การตั้งค่าเฉพาะเครื่องของ VS Code
├── src/
│   └── index.js
└── package.json
```

ถ้าเรา commit ไฟล์เหล่านี้ทั้งหมดลงไปใน repository จะเกิดปัญหาตามมาหลายข้อ:

1. **Build artifacts (ไฟล์ผลลัพธ์จากการ build)** เช่น `dist/`, `build/`, `*.class`, `*.o`, `*.pyc` — ไฟล์เหล่านี้ **สร้างขึ้นใหม่ได้เสมอ** จาก source code ด้วยคำสั่ง build เดียวกัน การเก็บไว้ใน Git จึงเป็นการเก็บข้อมูลซ้ำซ้อนที่ไม่มีประโยชน์ แถมยังทำให้ diff ระหว่าง commit อ่านยากมาก เพราะไฟล์ build มักถูก minify หรือ compile จนไม่เหลือรูปแบบที่มนุษย์อ่านได้

2. **Dependencies ขนาดใหญ่** เช่น `node_modules/` (อาจมีไฟล์หลายหมื่นไฟล์และขนาดหลายร้อย MB), `vendor/` ใน PHP, `venv/` ใน Python — ไฟล์เหล่านี้ถูกดาวน์โหลดมาจาก package manager (npm, composer, pip) และสามารถติดตั้งใหม่ได้ด้วยคำสั่งเดียว (`npm install`, `pip install -r requirements.txt`) การ commit เข้า repo จะทำให้ repo บวมขึ้นมหาศาลและ clone ช้าลงมาก

3. **ไฟล์ที่มีข้อมูลลับ (secrets)** เช่น `.env`, `config/secrets.yml`, `credentials.json` — ไฟล์เหล่านี้มักเก็บ API key, database password, private key ถ้า commit เข้า Git โดยไม่ระวัง ข้อมูลลับจะ**อยู่ในประวัติ (history) ตลอดไป** แม้จะลบไฟล์ทิ้งในภายหลังก็ตาม (การลบข้อมูลออกจาก Git history จริง ๆ ต้องใช้เครื่องมืออย่าง `git filter-repo` ซึ่งเราจะเรียนใน Part หลัง ๆ) นี่คือสาเหตุอันดับต้น ๆ ของเหตุการณ์ security incident ที่เกิดขึ้นจริงในหลายบริษัท

4. **ไฟล์ระบบปฏิบัติการ (OS-generated files)** เช่น `.DS_Store` (macOS), `Thumbs.db` (Windows), `Desktop.ini` (Windows) — ไฟล์เหล่านี้ระบบปฏิบัติการสร้างขึ้นเองอัตโนมัติเพื่อเก็บ metadata การแสดงผลของ Finder/Explorer ไม่มีประโยชน์ต่อโปรเจกต์เลย และจะสร้างความรำคาญให้เพื่อนร่วมทีมที่ใช้ OS ต่างกัน เพราะจะเห็นไฟล์เหล่านี้ปรากฏใน `git status` ตลอดเวลา

5. **ไฟล์ที่ IDE/Editor สร้างขึ้น** เช่น `.idea/` (JetBrains), `.vscode/` (VS Code — บางส่วนอาจอยากแชร์ บางส่วนไม่อยาก), `*.swp` (Vim) — เป็นการตั้งค่าเฉพาะบุคคลที่ไม่ควรบังคับให้ทั้งทีมใช้ตาม

6. **Log files และ temporary files** เช่น `*.log`, `*.tmp`, `npm-debug.log*` — ไฟล์ที่เกิดจากการรันโปรแกรมทดสอบหรือ debug ไม่มีคุณค่าต่อประวัติของโปรเจกต์

### 51.2 ผลเสียถ้าไม่มี .gitignore

หากไม่ได้ตั้งค่าการ ignore ไฟล์เหล่านี้ตั้งแต่ต้น จะเกิดปัญหาดังนี้:

| ปัญหา | ผลกระทบ |
|---|---|
| Repository มีขนาดใหญ่โดยไม่จำเป็น | Clone/Pull ช้า กิน bandwidth และพื้นที่เก็บข้อมูลบน server |
| `git status` แสดงไฟล์ที่ไม่เกี่ยวข้องนับพันไฟล์ | มองไม่เห็นไฟล์ที่เราแก้จริง ๆ ท่ามกลางไฟล์ noise |
| Merge conflict ในไฟล์ที่ไม่ควรแม้แต่จะแตะ | เช่น `.idea/workspace.xml` ที่เปลี่ยนทุกครั้งที่เปิด IDE ทำให้เกิด conflict ไร้สาระ |
| ข้อมูลลับรั่วไหล | API key, password หลุดเข้า public repository ถูกโจรกรรมนำไปใช้ในไม่กี่นาที (มี bot สแกน GitHub หาคีย์ที่หลุดตลอดเวลา) |
| ทีมงานคนละเครื่องมี "ขยะ" ปนกันไปมา | คนใช้ Mac push `.DS_Store` มา คนใช้ Windows push `Thumbs.db` มา วนไปเรื่อย ๆ |

Git จึงมีกลไกที่เรียกว่า **`.gitignore`** ซึ่งเป็นไฟล์ข้อความธรรมดาที่บอก Git ว่า **"ไฟล์หรือโฟลเดอร์ที่ตรงกับ pattern เหล่านี้ ไม่ต้องเอามาแสดงใน `git status` และห้ามถูก `git add` โดยไม่ได้ตั้งใจ"**

สิ่งสำคัญที่ต้องเข้าใจตั้งแต่แรก: **`.gitignore` ทำงานเฉพาะกับไฟล์ที่ยัง "untracked" เท่านั้น** — ถ้าไฟล์นั้นถูก track (ถูก commit เข้า Git แล้ว) มาก่อนที่จะเพิ่ม pattern ลงใน `.gitignore` การใส่ pattern นั้นจะ **ไม่มีผลอะไรเลย** Git จะยังคง track ไฟล์นั้นต่อไปตามปกติ (เราจะแก้ปัญหานี้ด้วย `git rm --cached` ใน Step 56)

---

## Step 52: สร้างไฟล์ .gitignore และ syntax พื้นฐาน

### 52.1 การสร้างไฟล์ .gitignore

`.gitignore` คือไฟล์ข้อความธรรมดา (plain text) ที่วางไว้ที่ **root ของ repository** (โดยทั่วไป) สร้างได้ง่าย ๆ ด้วยคำสั่ง:

```bash
touch .gitignore
```

หรือสร้างด้วย text editor แล้ว save เป็นชื่อ `.gitignore` (สังเกตว่าไม่มีชื่อไฟล์ก่อนจุด มีแต่นามสกุล — เป็น "dotfile" ที่ระบบปฏิบัติการมักซ่อนไว้โดย default)

จากนั้นเปิดไฟล์ด้วย editor แล้วเริ่มเขียน pattern ทีละบรรทัด:

```bash
# เปิดไฟล์ด้วย editor เพื่อแก้ไข
code .gitignore
```

### 52.2 Syntax พื้นฐาน

#### 1) ชื่อไฟล์ตรงตัว (Exact filename match)

การระบุชื่อไฟล์ตรง ๆ จะ ignore ไฟล์ที่มีชื่อนั้น **ทุกที่ในโปรเจกต์** (ทุกโฟลเดอร์ย่อยด้วย ไม่ใช่แค่ root)

```gitignore
.DS_Store
Thumbs.db
npm-debug.log
```

ตัวอย่างนี้จะ ignore ไฟล์ `.DS_Store` ไม่ว่าจะอยู่ที่ `./.DS_Store`, `./src/.DS_Store`, หรือ `./src/components/.DS_Store` ก็ตาม เพราะ Git ปฏิบัติกับ pattern ที่ไม่มี `/` เหมือนเป็น glob ที่ match ได้ในทุกระดับของ directory tree

#### 2) Wildcard `*` (จับคู่ตัวอักษรใด ๆ ก็ได้ ยกเว้น `/`)

เครื่องหมาย `*` แทนตัวอักษรกี่ตัวก็ได้ (รวมถึงศูนย์ตัว) แต่ **ไม่ข้าม directory separator (`/`)**

```gitignore
*.log        # ignore ไฟล์ทุกไฟล์ที่ลงท้ายด้วย .log เช่น error.log, npm-debug.log
*.tmp        # ignore ไฟล์ทุกไฟล์ที่ลงท้ายด้วย .tmp
temp*        # ignore ไฟล์ที่ขึ้นต้นด้วย temp เช่น temp.txt, temp_backup
```

ตัวอย่าง: `*.log` จะ match `error.log` และ `logs/debug.log` (เพราะมันไปหาที่ทุกระดับ) แต่การเขียน `log*.txt` จะ match เฉพาะไฟล์ที่ชื่อขึ้นต้นด้วย `log` และลงท้ายด้วย `.txt` เท่านั้น เช่น `log1.txt`, `logfile.txt`

#### 3) เจาะจงเฉพาะโฟลเดอร์ด้วย `/`

การใส่ `/` ต่อท้าย pattern จะบอกให้ Git ignore **เฉพาะโฟลเดอร์เท่านั้น** ไม่ใช่ไฟล์ที่ชื่อเดียวกัน

```gitignore
build/       # ignore เฉพาะโฟลเดอร์ที่ชื่อ build (ไม่ ignore ไฟล์ชื่อ build)
node_modules/
dist/
```

การใส่ `/` **นำหน้า** pattern จะบอกให้ Git จับคู่เฉพาะจาก **root ของ repository เท่านั้น** ไม่ใช่ทุกระดับ

```gitignore
/config.json     # ignore เฉพาะ config.json ที่อยู่ root เท่านั้น
                  # ไม่ ignore src/config.json หรือ test/config.json
```

เปรียบเทียบให้เห็นภาพชัด:

| Pattern | ความหมาย |
|---|---|
| `config.json` | ignore `config.json` ทุกที่ในโปรเจกต์ (root, src/, test/ ฯลฯ) |
| `/config.json` | ignore เฉพาะ `config.json` ที่ root เท่านั้น |
| `build/` | ignore โฟลเดอร์ชื่อ `build` ทุกที่ในโปรเจกต์ (ไม่ว่าจะอยู่ที่ root หรือใน `src/build/`) |
| `/build/` | ignore เฉพาะโฟลเดอร์ `build` ที่ root เท่านั้น |

### 52.3 ตัวอย่าง .gitignore เบื้องต้นสำหรับโปรเจกต์ทั่วไป

```gitignore
# Dependencies
node_modules/

# Build output
dist/
build/

# Environment variables
.env

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
```

### 52.4 ทดลองใช้งานจริง

มาลองสร้างโปรเจกต์เล็ก ๆ เพื่อทดสอบ:

```bash
mkdir ~/git-course/part-06-gitignore
cd ~/git-course/part-06-gitignore
git init
```

**ผลลัพธ์:**
```
Initialized empty Git repository in /home/user/git-course/part-06-gitignore/.git/
```

สร้างไฟล์จำลองขึ้นมาบางส่วน:

```bash
mkdir node_modules
touch node_modules/some-lib.js
touch app.js
touch .env
touch npm-debug.log
touch .DS_Store
```

ลองดูสถานะก่อนมี `.gitignore`:

```bash
git status
```

**ผลลัพธ์:**
```
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.DS_Store
	.env
	app.js
	node_modules/
	npm-debug.log

nothing added to commit but untracked files present (use "git add" to track)
```

จะเห็นว่า Git มองเห็นทั้งไฟล์ที่เราต้องการ (`app.js`) และไฟล์ที่ไม่ต้องการ (`node_modules/`, `.env`, `npm-debug.log`, `.DS_Store`) ปนกันหมด

ตอนนี้สร้าง `.gitignore`:

```bash
cat > .gitignore << 'EOF'
node_modules/
.env
*.log
.DS_Store
EOF
```

ลองดู `git status` อีกครั้ง:

```bash
git status
```

**ผลลัพธ์:**
```
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.gitignore
	app.js

nothing added to commit but untracked files present (use "git add" to track)
```

เห็นความเปลี่ยนแปลงชัดเจน — ตอนนี้ `git status` แสดงเฉพาะ `.gitignore` (ไฟล์ตั้งค่าที่เราควร commit) และ `app.js` (ซอร์สโค้ดจริง) เท่านั้น ไฟล์ขยะทั้งหมดหายไปจากสายตา

**หมายเหตุสำคัญ:** ไฟล์ `.gitignore` เอง **ควรถูก commit เข้า repository** เพราะมันเป็นส่วนหนึ่งของการตั้งค่าโปรเจกต์ที่ทุกคนในทีมต้องใช้ร่วมกัน ไม่ใช่ไฟล์ที่ต้อง ignore

```bash
git add .gitignore app.js
git commit -m "Initial commit with .gitignore"
```

**ผลลัพธ์:**
```
[main (root-commit) a1b2c3d] Initial commit with .gitignore
 2 files changed, 4 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 app.js
```

---

## Step 53: Pattern ขั้นสูงของ .gitignore

เมื่อโปรเจกต์ซับซ้อนขึ้น pattern พื้นฐานอาจไม่พอ Git รองรับ syntax ขั้นสูงอีกหลายแบบที่ทำให้เราควบคุมได้ละเอียดมาก

### 53.1 Comment ด้วย `#`

บรรทัดที่ขึ้นต้นด้วย `#` คือ comment ใช้อธิบายเหตุผลของ pattern แต่ละกลุ่ม เพื่อให้เพื่อนร่วมทีมเข้าใจง่าย:

```gitignore
# ===== Dependencies =====
node_modules/

# ===== Build outputs =====
dist/
build/

# ===== Environment & secrets =====
.env
.env.local
```

ถ้าต้องการ ignore ไฟล์ที่ชื่อขึ้นต้นด้วย `#` จริง ๆ (พบได้น้อยมาก) ต้อง escape ด้วย backslash:

```gitignore
\#backup-file#
```

### 53.2 Negation (ยกเว้น) ด้วย `!`

เครื่องหมาย `!` ที่ขึ้นต้นบรรทัด ใช้บอกว่า **"ยกเว้นไฟล์นี้ ไม่ต้อง ignore"** แม้จะ match กับ pattern ก่อนหน้าก็ตาม เป็นประโยชน์มากเมื่อเรา ignore โฟลเดอร์ทั้งหมดแต่อยากเก็บบางไฟล์ไว้

```gitignore
# ignore ทุกไฟล์ .log
*.log

# ยกเว้น important.log ที่จำเป็นต้องเก็บไว้
!important.log
```

**ข้อจำกัดสำคัญของ negation:** Git **ไม่สามารถ un-ignore ไฟล์ที่อยู่ภายในโฟลเดอร์ที่ถูก ignore ไปแล้วได้** เพราะเมื่อโฟลเดอร์ทั้งหมดถูก ignore Git จะไม่เข้าไปสำรวจ (traverse) เนื้อหาข้างในโฟลเดอร์นั้นเลยตั้งแต่แรก ทำให้ negation pattern ที่อยู่ข้างในไม่มีผลอะไร

ตัวอย่างที่ **ใช้ไม่ได้ผล**:

```gitignore
logs/
!logs/important.log
```

ในกรณีนี้ `logs/important.log` **จะยังคงถูก ignore อยู่** เพราะ Git ไม่มองเข้าไปในโฟลเดอร์ `logs/` เลยตั้งแต่ต้น (เนื่องจากทั้งโฟลเดอร์ถูก ignore ทันทีที่ pattern แรกจับคู่)

วิธีแก้ที่ถูกต้องคือ ignore เฉพาะ**ไฟล์ภายใน**โฟลเดอร์ แทนที่จะ ignore ตัวโฟลเดอร์เอง:

```gitignore
logs/*
!logs/important.log
```

การใช้ `logs/*` (มี `*` ต่อท้าย) หมายถึง ignore **ไฟล์ทุกไฟล์ที่อยู่ข้างในโฟลเดอร์ `logs/`** แต่ตัวโฟลเดอร์ `logs/` เองยังถูกสำรวจอยู่ ทำให้ negation `!logs/important.log` ทำงานได้จริง

### 53.3 `**` สำหรับจับคู่หลายระดับไดเรกทอรี

เครื่องหมาย `**` (double asterisk) ใช้จับคู่ **directory หลายระดับ** ซึ่งต่างจาก `*` ตัวเดียวที่จับคู่ได้แค่ระดับเดียว

```gitignore
# ignore โฟลเดอร์ชื่อ logs ไม่ว่าจะอยู่ลึกแค่ไหนก็ตาม
**/logs

# ignore ไฟล์ .txt ทุกไฟล์ที่อยู่ใต้โฟลเดอร์ temp ไม่ว่าจะลึกแค่ไหน
temp/**/*.txt

# ignore ทุกอย่างที่อยู่ใต้โฟลเดอร์ build ไม่ว่าจะลึกแค่ไหน
build/**
```

รูปแบบการใช้ `**` มี 3 แบบหลัก:

| Pattern | ความหมาย |
|---|---|
| `**/foo` | จับคู่ `foo` ที่อยู่ในทุกระดับของ directory tree (เทียบเท่ากับการเขียน `foo` เฉย ๆ ในกรณีส่วนใหญ่) |
| `foo/**` | จับคู่ทุกอย่างที่อยู่ **ข้างใน** โฟลเดอร์ `foo` (ทุกไฟล์ ทุก subfolder) |
| `a/**/b` | จับคู่ `b` ที่อยู่ใต้ `a` ไม่ว่าจะมีโฟลเดอร์คั่นกลางกี่ชั้นก็ตาม เช่น `a/b`, `a/x/b`, `a/x/y/b` |

### 53.4 Wildcard ตัวอักษรเดี่ยว `?`

เครื่องหมาย `?` จับคู่ตัวอักษร **ตัวเดียว** ใด ๆ ก็ได้ (ยกเว้น `/`)

```gitignore
file?.txt    # match file1.txt, file2.txt, fileA.txt แต่ไม่ match file10.txt
```

### 53.5 Character class ด้วย `[...]`

ใช้ระบุช่วงหรือกลุ่มตัวอักษรที่ต้องการจับคู่ตำแหน่งใดตำแหน่งหนึ่ง คล้ายกับ regex character class

```gitignore
log[0-9].txt      # match log0.txt ถึง log9.txt
file[abc].js      # match filea.js, fileb.js, filec.js
```

### 53.6 ตัวอย่างซับซ้อนที่ผสมหลาย pattern เข้าด้วยกัน

```gitignore
# ===== Dependencies =====
node_modules/
**/node_modules/       # เผื่อกรณี monorepo ที่มี node_modules ซ้อนหลายระดับ

# ===== Build & compiled output =====
dist/
build/
*.min.js
*.map

# ===== Logs (ยกเว้นไฟล์สำคัญ) =====
logs/*
!logs/.gitkeep
!logs/audit-important.log

# ===== Environment files =====
.env
.env.*
!.env.example         # เก็บไฟล์ตัวอย่างไว้เป็น template ให้ทีม

# ===== Test coverage =====
coverage/
*.lcov

# ===== Editor & IDE =====
.vscode/*
!.vscode/settings.json    # แชร์การตั้งค่า editor พื้นฐานร่วมกันในทีม
!.vscode/extensions.json
.idea/

# ===== OS-specific =====
.DS_Store
Thumbs.db
ehthumbs.db

# ===== Package manager lock files ที่บางทีมเลือกไม่ track =====
# (หมายเหตุ: โดยทั่วไปควร track package-lock.json/yarn.lock เพื่อ reproducible build
# แต่บางทีมที่ไม่ต้องการ lock version ก็เลือก ignore ได้)
# package-lock.json
```

ในตัวอย่างข้างต้น สังเกตว่าเราใช้ `.env.*` เพื่อ ignore ไฟล์ตระกูล `.env.local`, `.env.production` ทุกตัว แต่ใช้ `!.env.example` ยกเว้นไฟล์ตัวอย่างที่ไม่มีข้อมูลลับจริง ไว้ให้ทีมใช้เป็น template

### 53.7 ลำดับความสำคัญของ pattern (ต้องเข้าใจให้แม่น)

Git อ่าน `.gitignore` **จากบนลงล่าง** และ **pattern ที่อยู่ท้ายไฟล์จะมีความสำคัญเหนือกว่า** pattern ที่อยู่ก่อนหน้า ถ้ามีหลาย pattern ขัดแย้งกัน (เช่น pattern หนึ่งบอกให้ ignore แต่อีก pattern บอกให้ negate) **pattern ตัวสุดท้ายที่ match จะเป็นตัวตัดสิน**

```gitignore
*.log
!important.log
*.log          # บรรทัดนี้เขียนทับ negation ด้านบน — important.log จะกลับมาถูก ignore อีกครั้ง
```

หลักการนี้สำคัญมากเวลาเรามี `.gitignore` หลายไฟล์ในหลายระดับโฟลเดอร์ (จะอธิบายรายละเอียดใน Step 55) — ไฟล์ที่อยู่ **ใกล้กับไฟล์เป้าหมายมากกว่า** (อยู่ในโฟลเดอร์ย่อยที่ลึกกว่า) จะมีความสำคัญเหนือกว่าไฟล์ `.gitignore` ที่อยู่ระดับบนกว่า

---

## Step 54: เทมเพลตสำเร็จรูปจาก github/gitignore

การเขียน `.gitignore` เองตั้งแต่ศูนย์ทุกครั้งเป็นเรื่องเสียเวลาและเสี่ยงพลาด GitHub จึงดูแลโปรเจกต์ Open Source ชื่อ **[github/gitignore](https://github.com/github/gitignore)** ซึ่งรวบรวมเทมเพลต `.gitignore` มาตรฐานสำหรับภาษาโปรแกรมมิ่งและเฟรมเวิร์กเกือบทุกตัวที่มีอยู่บนโลก เป็นจุดเริ่มต้นที่ยอดเยี่ยมมากสำหรับโปรเจกต์ใหม่ทุกโปรเจกต์

### 54.1 วิธีใช้งาน

**วิธีที่ 1: ผ่านเว็บ GitHub โดยตรง (สร้าง repo ใหม่)**

เวลาสร้าง repository ใหม่บน GitHub.com จะมีตัวเลือก **"Add .gitignore"** ให้เลือก template ตามภาษาที่ใช้ได้ทันที (เราจะฝึกเรื่องนี้ละเอียดใน Part ที่ว่าด้วย GitHub)

**วิธีที่ 2: ดาวน์โหลดตรงจาก raw.githubusercontent.com**

```bash
curl -o .gitignore https://raw.githubusercontent.com/github/gitignore/main/Node.gitignore
```

**วิธีที่ 3: ใช้เครื่องมือ gitignore.io (สร้างเทมเพลตแบบผสมหลายภาษา/เฟรมเวิร์ก)**

```bash
curl -sL https://www.toptal.com/developers/gitignore/api/node,macos,windows,visualstudiocode > .gitignore
```

คำสั่งนี้จะสร้าง `.gitignore` ที่รวม pattern ของ Node.js, macOS, Windows และ VS Code เข้าด้วยกันในไฟล์เดียว เป็นวิธีที่ทีมพัฒนาจริงใช้กันบ่อยมาก เพราะสมาชิกในทีมมักใช้ OS ต่างกัน

### 54.2 ตัวอย่างเทมเพลตสำหรับ Node.js

```gitignore
# ===== Node.js (github/gitignore: Node.gitignore) =====

# Dependency directories
node_modules/
jspm_packages/

# npm
npm-debug.log*
.npm

# yarn
yarn-error.log
.yarn-integrity

# pnpm
.pnpm-debug.log*

# Build output
dist/
build/
out/

# Environment variables
.env
.env.local
.env.*.local

# Coverage directory used by tools like istanbul/nyc
coverage/
*.lcov
.nyc_output

# Optional REPL history
.node_repl_history

# TypeScript build info
*.tsbuildinfo

# Runtime data
pids/
*.pid
*.seed
*.pid.lock
```

### 54.3 ตัวอย่างเทมเพลตสำหรับ Python

```gitignore
# ===== Python (github/gitignore: Python.gitignore) =====

# Byte-compiled / optimized files
__pycache__/
*.py[cod]
*$py.class

# C extensions
*.so

# Distribution / packaging
build/
dist/
*.egg-info/
.eggs/

# Virtual environments
venv/
env/
.venv/
ENV/

# Unit test / coverage reports
htmlcov/
.coverage
.coverage.*
.pytest_cache/
.tox/

# Environment variables
.env

# Jupyter Notebook checkpoints
.ipynb_checkpoints

# mypy
.mypy_cache/
.dmypy.json

# Django
*.log
local_settings.py
db.sqlite3
```

### 54.4 ตัวอย่างเทมเพลตสำหรับ Java

```gitignore
# ===== Java (github/gitignore: Java.gitignore) =====

# Compiled class files
*.class

# Log files
*.log

# Package files
*.jar
*.war
*.ear
*.nar

# Maven
target/
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
.mvn/wrapper/maven-wrapper.jar

# Gradle
.gradle/
build/
!gradle/wrapper/gradle-wrapper.jar

# IDE - IntelliJ
.idea/
*.iml
*.iws

# IDE - Eclipse
.classpath
.project
.settings/
bin/
```

### 54.5 หลักการเลือกและปรับแต่งเทมเพลต

เทมเพลตจาก github/gitignore เป็น **จุดเริ่มต้นที่ดีมาก** แต่ไม่ควรใช้แบบ copy-paste โดยไม่พิจารณา ควรทำตามขั้นตอนนี้เสมอ:

1. **เลือกเทมเพลตหลักตามภาษา/runtime หลักของโปรเจกต์** (เช่น Node.gitignore สำหรับโปรเจกต์ Node.js)
2. **เพิ่มเทมเพลต OS/Editor** เข้าไปด้วยเสมอ (macOS.gitignore, Windows.gitignore, VisualStudioCode.gitignore) เพราะทีมมักใช้เครื่องต่างกัน
3. **อ่านทบทวนทุกบรรทัด** อย่าเชื่อ template แบบไม่ตรวจสอบ บาง pattern อาจ ignore ไฟล์ที่โปรเจกต์เราต้องการเก็บจริง ๆ
4. **เพิ่ม pattern เฉพาะโปรเจกต์** ที่ template ไม่มี เช่นไฟล์ config เฉพาะทีม หรือโฟลเดอร์ output เฉพาะของเครื่องมือ internal
5. **commit `.gitignore` ให้เร็วที่สุด** ตั้งแต่ก่อน commit แรกของโปรเจกต์ เพื่อป้องกันไฟล์ที่ไม่ควร track หลุดเข้าไปตั้งแต่ต้น

---

## Step 55: .gitignore หลายระดับ (Global, Per-repo, Per-directory)

Git รองรับการตั้งค่า ignore ได้หลายระดับ ซ้อนกันเป็นชั้น ๆ ตั้งแต่ระดับเครื่อง (ทุก repo) ไปจนถึงระดับโฟลเดอร์ย่อยเฉพาะจุด

### 55.1 ระดับที่ 1: Global .gitignore (ทุก repository บนเครื่องนี้)

บางไฟล์ไม่ได้เกี่ยวข้องกับ**โปรเจกต์**เลย แต่เกี่ยวข้องกับ**เครื่องหรือ editor ที่เราใช้** เช่น `.DS_Store` (เกิดเฉพาะบน macOS), ไฟล์ swap ของ Vim (`*.swp`) — ไฟล์แบบนี้ไม่ควรต้องเขียนซ้ำใน `.gitignore` ของทุกโปรเจกต์ ควรตั้งค่าไว้ที่ **ระดับเครื่อง (global)** ครั้งเดียวจบ

ขั้นตอนตั้งค่า Global .gitignore:

```bash
# สร้างไฟล์ global gitignore ไว้ที่ home directory
touch ~/.gitignore_global
```

ใส่เนื้อหาที่เกี่ยวกับ OS/Editor ของเราเข้าไป:

```bash
cat > ~/.gitignore_global << 'EOF'
.DS_Store
.DS_Store?
._*
Thumbs.db
*.swp
*~
.idea/
.vscode/
EOF
```

จากนั้นบอก Git ให้รู้จักไฟล์นี้ผ่าน `core.excludesFile`:

```bash
git config --global core.excludesFile ~/.gitignore_global
```

ตรวจสอบว่าตั้งค่าสำเร็จ:

```bash
git config --global --get core.excludesFile
```

**ผลลัพธ์:**
```
/home/user/.gitignore_global
```

ตั้งแต่นี้ไป **ทุก repository บนเครื่องนี้** จะ ignore pattern เหล่านี้โดยอัตโนมัติ โดยไม่ต้องเขียนซ้ำใน `.gitignore` ของแต่ละโปรเจกต์เลย และที่สำคัญคือ **ไฟล์ global นี้จะไม่ถูกแชร์กับทีม** เพราะมันไม่ได้อยู่ใน repository ของโปรเจกต์ (อยู่ที่ home directory ของเราเอง) — เหมาะมากสำหรับ preference ส่วนตัวที่ไม่เกี่ยวข้องกับโปรเจกต์

### 55.2 ระดับที่ 2: Per-repository .gitignore (ที่ root ของ repo)

นี่คือแบบที่เราใช้กันมาตลอดใน Step ก่อนหน้า คือไฟล์ `.gitignore` ที่วางไว้ที่ root ของ repository เนื้อหาในไฟล์นี้ **ควร commit เข้า Git และแชร์กับทีมทุกคน** เพราะเป็นกติกากลางของโปรเจกต์ที่ทุกคนต้องปฏิบัติตามเหมือนกัน (เช่น ignore `node_modules/`, `dist/`, `.env`)

```
my-project/
├── .gitignore        ← มีผลกับทุกไฟล์ในโปรเจกต์นี้ทั้งหมด
├── src/
└── package.json
```

### 55.3 ระดับที่ 3: Per-directory .gitignore (ในโฟลเดอร์ย่อย)

Git อนุญาตให้มี `.gitignore` ได้ **มากกว่าหนึ่งไฟล์** ในโปรเจกต์เดียวกัน โดยวางไว้ในโฟลเดอร์ย่อยต่าง ๆ pattern ในไฟล์นั้นจะมีผล**เฉพาะกับไฟล์ที่อยู่ในโฟลเดอร์นั้นลงไป** (รวม subfolder ของมันด้วย)

ตัวอย่างเช่นในโปรเจกต์ monorepo:

```
my-project/
├── .gitignore                  ← กติกากลางทั้งโปรเจกต์
├── packages/
│   ├── frontend/
│   │   ├── .gitignore          ← กติกาเฉพาะของ frontend เช่น ignore .next/
│   │   └── src/
│   └── backend/
│       ├── .gitignore          ← กติกาเฉพาะของ backend เช่น ignore uploads/
│       └── src/
```

`packages/frontend/.gitignore`:
```gitignore
.next/
.turbo/
```

`packages/backend/.gitignore`:
```gitignore
uploads/
*.sqlite3
```

**หลักการทำงาน:** เมื่อ Git ตรวจสอบว่าไฟล์ใดควรถูก ignore หรือไม่ มันจะพิจารณาไฟล์ `.gitignore` **ทุกไฟล์ที่อยู่ในเส้นทาง (path) ตั้งแต่ root ลงไปจนถึงไฟล์เป้าหมาย** โดย pattern ในไฟล์ที่อยู่**ใกล้กับไฟล์เป้าหมายมากกว่า** (อยู่ในระดับที่ลึกกว่า) จะมีความสำคัญ**เหนือกว่า**ไฟล์ที่อยู่ระดับบนกว่า

นี่หมายความว่าเราสามารถใช้ `.gitignore` ระดับโฟลเดอร์ย่อยเพื่อ **ยกเลิก (override)** pattern จากไฟล์ระดับบนได้ เช่น:

`.gitignore` (ที่ root):
```gitignore
*.log
```

`docs/.gitignore`:
```gitignore
!important-notes.log
```

ผลลัพธ์คือไฟล์ `docs/important-notes.log` จะ**ไม่ถูก ignore** แม้ว่า pattern ที่ root จะบอกให้ ignore ไฟล์ `.log` ทั้งหมดก็ตาม เพราะ `.gitignore` ที่อยู่ใกล้ไฟล์เป้าหมายกว่า (ในโฟลเดอร์ `docs/`) มีสิทธิ์เหนือกว่า

### 55.4 สรุปลำดับชั้นการทำงานของ .gitignore ทั้ง 3 ระดับ

```
ลำดับที่ Git พิจารณา (จากน้ำหนักน้อยไปมาก):

1. ~/.gitignore_global          (Global — ทุก repo บนเครื่อง)
2. .git/info/exclude            (Local-only — เฉพาะ repo นี้ ไม่แชร์ ดู Step 57)
3. .gitignore ที่ root          (Per-repo — แชร์กับทีม)
4. .gitignore ในโฟลเดอร์ย่อย    (Per-directory — แชร์กับทีม, มีผลเฉพาะโฟลเดอร์นั้นลงไป)

  → ไฟล์ที่อยู่ "ใกล้" ไฟล์เป้าหมายที่สุดในลำดับนี้ ชนะเสมอเมื่อมี pattern ขัดแย้งกัน
```

---

## Step 56: ไฟล์ที่ track ไปแล้วแต่อยากเพิ่มใน .gitignore ภายหลัง (git rm --cached)

นี่คือสถานการณ์ที่เกิดขึ้นบ่อยมากในทีมจริง: มีคน commit ไฟล์ที่ไม่ควร track เข้าไปแล้วโดยไม่ตั้งใจ (เช่น `.env`, `node_modules/`, ไฟล์ IDE) พอมาเพิ่ม pattern ใน `.gitignore` ภายหลัง กลับพบว่า **ไฟล์นั้นยังคงปรากฏใน `git status` เหมือนเดิม ไม่มีอะไรเปลี่ยนแปลง**

### 56.1 ทำไมถึงเป็นแบบนั้น

ต้องย้ำหลักการสำคัญอีกครั้ง: **`.gitignore` มีผลเฉพาะกับไฟล์ที่ Git ยังไม่เคย track เท่านั้น** ถ้าไฟล์ถูก `git add` และ `git commit` ไปแล้วครั้งหนึ่ง Git จะถือว่าไฟล์นั้นเป็นไฟล์ที่ "อยู่ในการดูแล" (tracked) ของมันอยู่แล้ว การเพิ่ม pattern ใน `.gitignore` ในภายหลังเป็นเพียงการบอกว่า **"ไม่ต้องมาแจ้งเตือนไฟล์ใหม่ที่ตรงกับ pattern นี้"** แต่ไม่ได้สั่งให้ **หยุด track ไฟล์ที่ track อยู่แล้ว**

### 56.2 วิธีแก้: git rm --cached

คำสั่ง `git rm --cached` จะลบไฟล์ออกจาก **Staging Area / Index** (คือ "หยุด track") **โดยไม่ลบไฟล์จริงออกจาก working directory**

ตัวอย่างสถานการณ์: สมมติเราเผลอ commit `.env` เข้าไปแล้ว

```bash
# ตรวจสอบก่อนว่า .env ถูก track อยู่จริงหรือไม่
git ls-files | grep .env
```

**ผลลัพธ์:**
```
.env
```

ยืนยันว่า `.env` ถูก track อยู่ ทั้งที่เราเพิ่ง pattern `.env` ลงใน `.gitignore` ไปแล้ว

แก้ปัญหาด้วยการเอาไฟล์ออกจาก tracking (แต่เก็บไฟล์จริงไว้ในเครื่อง):

```bash
git rm --cached .env
```

**ผลลัพธ์:**
```
rm '.env'
```

ตรวจสอบสถานะ:

```bash
git status
```

**ผลลัพธ์:**
```
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	deleted:    .env

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.env
```

สังเกตว่า Git บอกว่า `.env` ถูก "deleted" ใน staging area (คือหยุด track) แต่ในขณะเดียวกันก็เห็นว่า `.env` ยังปรากฏเป็น "Untracked files" อยู่ (เพราะไฟล์จริงยังอยู่บนดิสก์) — ที่ `.env` ยังโผล่มาใน Untracked files แทนที่จะถูก ignore เพราะ Git status แสดงไฟล์นี้ก่อนที่ commit การลบออกจาก cache จะเสร็จสมบูรณ์ พอ commit เสร็จแล้ว `.gitignore` จะเริ่มมีผลกับมันตามปกติ

commit การเปลี่ยนแปลงนี้:

```bash
git commit -m "Remove .env from tracking, add to .gitignore"
```

**ผลลัพธ์:**
```
[main 4f5e6a7] Remove .env from tracking, add to .gitignore
 1 file changed, 1 deletion(-)
 delete mode 100644 .env
```

ตรวจสอบอีกครั้ง:

```bash
git status
```

**ผลลัพธ์:**
```
On branch main
nothing to commit, working tree clean
```

ตอนนี้ `.env` **ยังอยู่บนดิสก์ของเราตามปกติ** (ลองเช็คด้วย `ls -la` ก็จะเห็นไฟล์อยู่) แต่ Git ไม่ track มันอีกต่อไปแล้ว และเพราะ `.gitignore` มี pattern `.env` อยู่ Git จะไม่แสดงมันเป็น untracked file อีกด้วย

### 56.3 กรณีที่มีหลายไฟล์/หลายโฟลเดอร์ที่ต้องเอาออกพร้อมกัน

ถ้าต้องเอาไฟล์ออกจากการ track จำนวนมาก (เช่นเผลอ commit `node_modules/` ทั้งโฟลเดอร์เข้าไป) ใช้ flag `-r` (recursive) ร่วมด้วย:

```bash
git rm -r --cached node_modules/
```

**ผลลัพธ์ (ตัดมาบางส่วน เพราะไฟล์มีจำนวนมาก):**
```
rm 'node_modules/lodash/index.js'
rm 'node_modules/lodash/package.json'
rm 'node_modules/express/lib/application.js'
... (อีกหลายพันบรรทัด)
```

หรือถ้าต้องการเอาไฟล์ทุกอย่างที่ตรงกับ `.gitignore` ปัจจุบันออกจาก tracking ในคราวเดียว (สะดวกมากเมื่อเพิ่งเพิ่ม `.gitignore` ทีหลังในโปรเจกต์ที่มีของค้างเยอะ) สามารถทำได้ด้วยเทคนิคนี้:

```bash
# ล้าง index ทั้งหมดออกก่อน (ไม่กระทบไฟล์จริง)
git rm -r --cached .

# แล้ว add กลับเข้าไปใหม่ — คราวนี้ .gitignore จะกรองให้อัตโนมัติ
git add .

git commit -m "Apply .gitignore retroactively to remove already-tracked ignored files"
```

**คำเตือนสำคัญ:** เทคนิคนี้ควรใช้ด้วยความระมัดระวังมาก เพราะ `git rm -r --cached .` จะเอาไฟล์**ทุกไฟล์**ออกจาก index ชั่วคราว ถ้า `.gitignore` เขียนผิดพลาดหรือ pattern กว้างเกินไป การ `git add .` กลับเข้าไปอาจไม่ครบไฟล์ที่ต้องการ **ควรตรวจสอบด้วย `git status` อย่างละเอียดก่อน commit ทุกครั้ง**

### 56.4 กรณีข้อมูลลับ (secrets) หลุดเข้า Git ไปแล้ว

ต้องเน้นย้ำว่า `git rm --cached` **แก้ปัญหาได้แค่ในอนาคต** — ไฟล์ `.env` หรือข้อมูลลับที่เคย commit ไปแล้วนั้น **ยังคงอยู่ใน Git history เดิม** สามารถถูกดึงกลับมาดูได้เสมอผ่าน `git log`, `git show` หรือแม้แต่ clone ประวัติเก่าทั้งหมด

ถ้าข้อมูลลับหลุดเข้า public repository ไปแล้วจริง ๆ ขั้นตอนที่ถูกต้องคือ:

1. **เปลี่ยน (rotate) credential นั้นทันที** (เปลี่ยน password, revoke API key เก่าแล้วสร้างใหม่) — นี่คือขั้นตอนที่สำคัญที่สุดและต้องทำก่อนอย่างอื่นเสมอ เพราะข้อมูลที่หลุดไปแล้วถือว่าถูกขโมยไปแล้ว
2. ใช้เครื่องมือลบประวัติ เช่น `git filter-repo` หรือ BFG Repo-Cleaner เพื่อลบไฟล์นั้นออกจากทุก commit ในประวัติ (เราจะเรียนเรื่องนี้แบบละเอียดใน Part ที่ว่าด้วย Git Internals และ Security)
3. Force-push ประวัติที่แก้ไขแล้วทับของเดิม และแจ้งให้ทุกคนในทีม clone ใหม่

---

## Step 57: .git/info/exclude — ignore เฉพาะเครื่อง ไม่แชร์กับทีม

นอกจาก `.gitignore` (ซึ่งถูก commit และแชร์กับทีม) Git ยังมีกลไกอีกแบบหนึ่งสำหรับ ignore ไฟล์ที่ **เกี่ยวข้องกับตัวเราคนเดียวเท่านั้น ไม่ต้องการให้คนอื่นในทีมเห็นหรือใช้ตาม** — นั่นคือไฟล์ `.git/info/exclude`

### 57.1 ความแตกต่างระหว่าง .gitignore กับ .git/info/exclude

| คุณสมบัติ | `.gitignore` | `.git/info/exclude` |
|---|---|---|
| ตำแหน่งไฟล์ | อยู่ใน working directory (เช่น root ของ repo) | อยู่ใน `.git/info/` ซึ่งเป็นส่วนหนึ่งของ `.git` directory |
| ถูก track โดย Git ไหม | ใช่ — เป็นไฟล์ปกติที่ commit ได้ | **ไม่ใช่ — `.git/` ไม่เคยถูก track ไม่ว่ากรณีใด** |
| แชร์กับทีมไหม | แชร์ (เพราะอยู่ใน repo และถูก push/pull ไปด้วย) | **ไม่แชร์เด็ดขาด** เพราะอยู่นอกขอบเขตของ repository content — clone ไปเครื่องอื่นจะไม่มีไฟล์นี้ |
| เหมาะกับ | Pattern ที่ทุกคนในทีมควรใช้ร่วมกัน (build artifacts, dependencies) | Pattern ส่วนตัวที่เกี่ยวกับ workflow ของเราคนเดียว (เช่น scratch file, note ส่วนตัวที่ไม่เกี่ยวกับโปรเจกต์) |

### 57.2 วิธีใช้งาน

Syntax ของ `.git/info/exclude` **เหมือนกับ `.gitignore` ทุกประการ** เพียงแต่อยู่คนละตำแหน่งไฟล์

```bash
# เปิดไฟล์ (โดยปกติ Git จะสร้างไฟล์นี้ให้อัตโนมัติตอน git init พร้อม comment อธิบายไว้แล้ว)
code .git/info/exclude
```

ตัวอย่างเนื้อหา:

```gitignore
# ไฟล์ note ส่วนตัวที่ใช้จด TODO ระหว่างทำงาน ไม่เกี่ยวกับโปรเจกต์
my-scratch-notes.md

# ไฟล์ config เฉพาะเครื่องของฉันที่ทดลองอะไรบางอย่าง ไม่ต้องการให้ทีมเห็น
experimental-config.local.json
```

### 57.3 กรณีใช้งานจริงที่เหมาะสม

- คุณกำลังทดลองแก้ไข configuration ไฟล์บางตัวในเครื่องตัวเองเพื่อ debug แต่ไม่อยากให้ปรากฏใน `git status` ระหว่างทำงาน และไม่อยากแก้ `.gitignore` ที่ใช้ร่วมกับทีม (เพราะไม่เกี่ยวกับคนอื่น)
- คุณมีสคริปต์ helper ส่วนตัวที่วางไว้ใน repo เพื่อความสะดวกของตัวเอง แต่ไม่ต้องการ "บังคับความเห็น" ให้ทีมต้อง ignore ไฟล์นี้ด้วย (เพราะบางคนอาจตั้งชื่อไฟล์แบบเดียวกันเพื่อจุดประสงค์อื่น)
- คุณกำลังทำงานใน repository ของ Open Source ที่ไม่ใช่ของคุณ ไม่มีสิทธิ์แก้ `.gitignore` กลาง (หรือไม่อยากส่ง PR แค่เพื่อเพิ่ม pattern ส่วนตัว) จึงใช้ `.git/info/exclude` แทน

### 57.4 ข้อควรระวัง

เนื่องจาก `.git/info/exclude` อยู่ **ภายใน** โฟลเดอร์ `.git` ของ repository นั้น ๆ (ไม่ใช่ระดับเครื่องเหมือน global .gitignore) มันจึงมีผล**เฉพาะกับ repository นี้เท่านั้น** และที่สำคัญที่สุดคือ **มันจะหายไปถ้ามีการ clone repository ใหม่** เพราะ `.git/info/exclude` ไม่เคยถูกอัปโหลดไปกับ remote repository เลย — ถ้าคุณ clone repo เดิมมาใหม่ในเครื่องอื่น หรือลบโฟลเดอร์แล้ว clone ใหม่ ไฟล์นี้จะกลับไปเป็นค่าเริ่มต้นว่างเปล่า ต้องตั้งค่าใหม่เอง

---

## Step 58: ตรวจสอบว่าไฟล์ถูก ignore ด้วย git check-ignore -v และ git status --ignored

เมื่อโปรเจกต์มี `.gitignore` หลายไฟล์ซ้อนกันหลายระดับ (global, per-repo, per-directory, `.git/info/exclude`) บางครั้งก็สับสนว่า "ทำไมไฟล์นี้ถึงถูก ignore" หรือ "ทำไมไฟล์นี้ถึงไม่ถูก ignore ทั้งที่น่าจะตรงกับ pattern" Git มีเครื่องมือช่วยวินิจฉัยปัญหานี้โดยเฉพาะ

### 58.1 git check-ignore -v

คำสั่ง `git check-ignore` ใช้ตรวจสอบว่าไฟล์ที่ระบุถูก ignore หรือไม่ และ flag `-v` (verbose) จะบอกด้วยว่า **pattern ไหนจากไฟล์ไหน บรรทัดที่เท่าไหร่** เป็นตัวที่ทำให้ไฟล์นั้นถูก ignore

```bash
git check-ignore -v node_modules/express/index.js
```

**ผลลัพธ์:**
```
.gitignore:1:node_modules/	node_modules/express/index.js
```

จากผลลัพธ์นี้อ่านได้ว่า: ไฟล์ `.gitignore` (ที่ root) บรรทัดที่ 1 ซึ่งมี pattern `node_modules/` คือสาเหตุที่ทำให้ไฟล์นี้ถูก ignore

ลองตรวจสอบไฟล์ที่ไม่ได้ถูก ignore:

```bash
git check-ignore -v app.js
```

**ผลลัพธ์:**
```
(ไม่มีผลลัพธ์อะไรเลย — แสดงว่าไฟล์นี้ไม่ถูก ignore)
```

เมื่อไฟล์ไม่ถูก ignore คำสั่งจะไม่แสดงอะไรเลยและ exit code จะเป็น 1 (สามารถใช้ตรวจสอบใน script ได้ด้วย)

ตรวจสอบหลายไฟล์พร้อมกัน:

```bash
git check-ignore -v .env dist/bundle.js src/index.js
```

**ผลลัพธ์:**
```
.gitignore:5:.env	.env
.gitignore:8:dist/	dist/bundle.js
```

สังเกตว่า `src/index.js` ไม่ปรากฏในผลลัพธ์เลย เพราะมันไม่ถูก ignore (ไฟล์ที่ไม่ตรงกับ pattern ไหนจะไม่แสดงผลลัพธ์ออกมา)

### 58.2 ตรวจสอบกรณี pattern ซับซ้อนหลายชั้น (negation)

```bash
git check-ignore -v logs/important.log
```

**ผลลัพธ์ (ถ้า negation ทำงานถูกต้อง):**
```
.gitignore:12:!logs/important.log	logs/important.log
```

สังเกตเครื่องหมาย `!` นำหน้า pattern ในผลลัพธ์ — นี่บอกว่าไฟล์นี้ **ไม่ถูก ignore** เพราะ pattern สุดท้ายที่ match คือ negation pattern (แต่คำสั่งนี้ยังคงแสดงผลลัพธ์ออกมาเพื่อบอกว่ามี rule เกี่ยวข้องกับไฟล์นี้ แม้ผลสุดท้ายจะไม่ ignore ก็ตาม — ต้องดูเครื่องหมาย `!` ประกอบด้วยเสมอ)

### 58.3 git status --ignored

ปกติแล้ว `git status` จะไม่แสดงไฟล์ที่ถูก ignore เลย (เพื่อไม่ให้รก) แต่บางครั้งเราต้องการเห็นภาพรวมว่ามีไฟล์อะไรบ้างที่ถูก ignore อยู่ ใช้ flag `--ignored`:

```bash
git status --ignored
```

**ผลลัพธ์:**
```
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
	modified:   src/index.js

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	notes.txt

Ignored files:
  (use "git add -f <file>..." to include in what will be committed)
	.DS_Store
	.env
	node_modules/
	npm-debug.log

no changes added to commit (use "git add -f" to force)
```

จะเห็นว่า Git แยกหมวดหมู่ให้ชัดเจน 3 กลุ่ม: ไฟล์ที่แก้ไขแล้ว (modified), ไฟล์ใหม่ที่ยังไม่ track (untracked), และไฟล์ที่ถูก ignore (ignored) — มีประโยชน์มากตอนต้องการตรวจสอบว่า `.gitignore` ทำงานครอบคลุมไฟล์ที่ต้องการจริงหรือไม่

หากต้องการดูรายละเอียดเนื้อหาข้างในโฟลเดอร์ที่ถูก ignore ด้วย (ปกติ Git จะแสดงแค่ชื่อโฟลเดอร์ ไม่ไล่ดูไฟล์ข้างใน) ใช้ร่วมกับ `--untracked-files=all`:

```bash
git status --ignored --untracked-files=all
```

**ผลลัพธ์ (ตัวอย่าง):**
```
Ignored files:
  (use "git add -f <file>..." to include in what will be committed)
	.DS_Store
	.env
	node_modules/express/index.js
	node_modules/express/package.json
	node_modules/lodash/index.js
	npm-debug.log
```

### 58.4 ใช้ git ls-files ตรวจสอบว่าไฟล์ถูก track อยู่หรือไม่

บางครั้งปัญหาไม่ใช่เรื่อง ignore แต่เป็นเพราะไฟล์นั้น**ถูก track ไปแล้ว** (ตามที่อธิบายใน Step 56) วิธีตรวจสอบง่าย ๆ ว่าไฟล์อยู่ใน tracking หรือไม่:

```bash
git ls-files --error-unmatch .env
```

**ผลลัพธ์ (ถ้าไฟล์ถูก track):**
```
.env
```

**ผลลัพธ์ (ถ้าไฟล์ไม่ได้ถูก track):**
```
error: pathspec '.env' did not match any file(s) known to git
```

การใช้ `git check-ignore -v` ร่วมกับ `git ls-files --error-unmatch` เป็นชุดเครื่องมือวินิจฉัยที่ครบสมบูรณ์: ตัวแรกตอบว่า "pattern ไหนกำลัง ignore ไฟล์นี้อยู่" ส่วนตัวที่สองตอบว่า "ไฟล์นี้ถูก track อยู่แล้วหรือยัง"

---

## Step 59: ข้อผิดพลาดที่พบบ่อยเกี่ยวกับ .gitignore

รวบรวมข้อผิดพลาดที่พบได้บ่อยที่สุดเมื่อทำงานกับ `.gitignore` พร้อมวิธีวินิจฉัยและแก้ไขแต่ละกรณี

### 59.1 "เพิ่ม pattern ใน .gitignore แล้วแต่ไฟล์ก็ยังโผล่ใน git status"

**สาเหตุที่พบบ่อยที่สุด:** ไฟล์นั้น**ถูก track อยู่แล้ว** ก่อนที่จะเพิ่ม pattern (ตามที่อธิบายละเอียดใน Step 56)

**วิธีวินิจฉัย:**
```bash
git ls-files --error-unmatch <ชื่อไฟล์>
```
ถ้าคำสั่งนี้คืนชื่อไฟล์ออกมา (ไม่ error) แปลว่าไฟล์นั้นถูก track อยู่แล้ว ต้องใช้ `git rm --cached` เพื่อหยุด track ก่อน

### 59.2 "Negation pattern (!) ไม่ทำงานเลย"

**สาเหตุ:** ไฟล์ที่ต้องการ un-ignore อยู่ **ภายในโฟลเดอร์ที่ถูก ignore ทั้งโฟลเดอร์** (ตามที่อธิบายใน Step 53.2) — Git จะไม่เข้าไปสำรวจเนื้อหาข้างในโฟลเดอร์ที่ถูก ignore เลย ทำให้ negation ที่อยู่ลึกกว่าไม่มีผล

**วิธีแก้:** เปลี่ยนจาก ignore ตัวโฟลเดอร์ (`logs/`) เป็น ignore เฉพาะไฟล์ข้างใน (`logs/*`) แทน เพื่อให้ Git ยังคงสำรวจโฟลเดอร์นั้นและ negation pattern ทำงานได้

```gitignore
# ผิด — negation จะไม่ทำงาน
logs/
!logs/keep-this.log

# ถูก — negation ทำงานได้ปกติ
logs/*
!logs/keep-this.log
```

### 59.3 "pattern มี `/` นำหน้าหรือไม่ ทำให้พฤติกรรมต่างกันโดยไม่รู้ตัว"

หลายคนสับสนระหว่าง `build/` กับ `/build/` เพราะมองผ่าน ๆ เหมือนกัน แต่ความหมายต่างกันโดยสิ้นเชิง

```gitignore
build/     # ignore โฟลเดอร์ build ทุกที่ในโปรเจกต์ (root, src/build/, test/build/ ฯลฯ)
/build/    # ignore เฉพาะโฟลเดอร์ build ที่ root เท่านั้น
```

**วิธีแก้:** ใช้ `git check-ignore -v` ตรวจสอบเสมอเมื่อไม่แน่ใจว่า pattern ครอบคลุมไฟล์เป้าหมายจริงหรือไม่ อย่าเดาเอาเอง

### 59.4 "ลืมว่า .gitignore ไม่มีผลย้อนหลังกับ git add -f"

คำสั่ง `git add -f` (force) จะบังคับ add ไฟล์เข้า staging area **แม้ว่าไฟล์นั้นจะตรงกับ pattern ใน `.gitignore` ก็ตาม** บางครั้งมีคนใช้คำสั่งนี้โดยไม่ตั้งใจ (เช่น copy คำสั่งมาจากที่อื่นโดยไม่อ่าน) ทำให้ไฟล์ที่ควร ignore หลุดเข้า repo ไปได้

```bash
# คำสั่งนี้จะ add .env เข้าไปได้ แม้จะอยู่ใน .gitignore ก็ตาม!
git add -f .env
```

**วิธีป้องกัน:** ระมัดระวังเสมอเวลาเห็น flag `-f` หรือ `--force` ในคำสั่ง Git ใด ๆ ก็ตาม และตรวจสอบ `git status` ก่อน commit ทุกครั้งเพื่อยืนยันว่าไม่มีไฟล์แปลกปลอมติดเข้าไป

### 59.5 "ลำดับ pattern ผิด ทำให้ pattern ท้ายไฟล์เขียนทับ pattern ที่ตั้งใจไว้ก่อนหน้า"

เนื่องจาก Git อ่านและประมวลผล `.gitignore` **จากบนลงล่าง** และ **pattern ตัวหลังชนะเสมอเมื่อขัดแย้งกัน** การเรียงลำดับ pattern ผิดอาจทำให้เกิดผลลัพธ์ที่ไม่คาดคิด

```gitignore
!important.log    # ตั้งใจจะยกเว้นไฟล์นี้
*.log             # แต่บรรทัดนี้อยู่ทีหลัง และ match ทุกไฟล์ .log รวมถึง important.log ด้วย
                   # ผลลัพธ์: important.log ยังถูก ignore อยู่ดี เพราะ *.log ที่อยู่ท้ายกว่าชนะ
```

**วิธีแก้:** เรียง pattern กว้าง ๆ ไว้ก่อน แล้วค่อยตามด้วย negation ที่เจาะจงกว่าไว้ท้ายสุดเสมอ

```gitignore
*.log             # กฎกว้าง ๆ มาก่อน
!important.log    # ข้อยกเว้นเฉพาะเจาะจง ต้องมาทีหลังเสมอ
```

### 59.6 "แก้ .gitignore แล้ว แต่ยังเห็นไฟล์เก่าค้างใน git status เพราะ cache เก่าของ Git ตัวเอง"

บางครั้งหลังแก้ `.gitignore` แล้วรัน `git status` ยังเห็นไฟล์ที่ควร ignore ค้างอยู่ — มักเกิดจาก terminal/shell แสดงผล cache เก่า หรือในบางกรณี Git อาจต้อง refresh index

**วิธีแก้:**
```bash
git rm -r --cached .
git add .
git status
```

(เทคนิคเดียวกับ Step 56.3 — เอาทุกไฟล์ออกจาก index ชั่วคราวแล้ว add กลับเข้าไปใหม่ เพื่อให้ `.gitignore` ปัจจุบันมีผลกับทุกไฟล์อย่างถูกต้อง)

### 59.7 "ไฟล์ .gitignore เองไม่ถูก commit เข้าไปด้วย"

บางครั้งมือใหม่เข้าใจผิดว่า `.gitignore` เป็นไฟล์ที่ต้อง ignore ตัวมันเองด้วย จึงลืม `git add .gitignore` ทำให้เพื่อนร่วมทีมที่ clone repo ไปไม่มี `.gitignore` เลย

**วิธีแก้:** จำไว้เสมอว่า `.gitignore` **ควรถูก track และ commit เข้า repository เหมือนไฟล์ปกติทั่วไป** เพราะมันคือ "กติกากลาง" ที่ทุกคนในทีมต้องใช้ร่วมกัน ไม่ใช่ไฟล์ขยะที่ต้องซ่อน

---

## Step 60: แบบฝึกหัด — สร้างโปรเจกต์ Node.js จำลอง เขียน .gitignore และแก้ปัญหาไฟล์หลุด track

ถึงเวลาลงมือปฏิบัติจริงแบบครบวงจร ผสมทุกอย่างที่เรียนมาใน Part นี้เข้าด้วยกัน

### 60.1 เตรียมโปรเจกต์

```bash
mkdir -p ~/git-course/part-06-practice
cd ~/git-course/part-06-practice
git init
```

**ผลลัพธ์:**
```
Initialized empty Git repository in /home/user/git-course/part-06-practice/.git/
```

### 60.2 จำลองโครงสร้างโปรเจกต์ Node.js ที่ "ยังไม่มี .gitignore" (สถานการณ์ปัญหา)

```bash
mkdir -p node_modules/express
mkdir -p node_modules/lodash
mkdir -p dist
mkdir -p coverage
mkdir -p logs
mkdir -p .vscode

echo "module.exports = {};" > node_modules/express/index.js
echo "module.exports = {};" > node_modules/lodash/index.js
echo "console.log('built');" > dist/bundle.js
echo "80% coverage" > coverage/report.html
echo "error at line 5" > logs/error.log
echo "important: do not delete" > logs/important.log
echo "SECRET_KEY=abc123" > .env
echo "{}" > .vscode/settings.json
echo "" > .DS_Store
echo "console.log('hello world');" > app.js
echo '{"name": "practice-app", "version": "1.0.0"}' > package.json
```

ตรวจสอบสถานะก่อนมี `.gitignore`:

```bash
git status
```

**ผลลัพธ์:**
```
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.DS_Store
	.env
	.vscode/
	app.js
	coverage/
	dist/
	logs/
	node_modules/
	package.json

nothing added to commit but untracked files present (use "git add" to track)
```

**สมมติว่ามือใหม่ในทีมพลาด — commit ทุกอย่างเข้าไปแบบไม่คิดอะไร:**

```bash
git add .
git commit -m "Initial commit"
```

**ผลลัพธ์:**
```
[main (root-commit) 9a1b2c3] Initial commit
 10 files changed, 10 insertions(+)
 create mode 100644 .DS_Store
 create mode 100644 .env
 create mode 100644 .vscode/settings.json
 create mode 100644 app.js
 create mode 100644 coverage/report.html
 create mode 100644 dist/bundle.js
 create mode 100644 logs/error.log
 create mode 100644 logs/important.log
 create mode 100644 node_modules/express/index.js
 create mode 100644 node_modules/lodash/index.js
 create mode 100644 package.json
```

นี่คือสถานการณ์ปัญหาจริงที่เกิดขึ้นบ่อยมาก — ไฟล์ `.env` ที่มีข้อมูลลับ, `node_modules/`, `dist/`, `.DS_Store` ทั้งหมดถูก commit เข้าไปในประวัติแล้ว ต่อไปเราจะแก้ปัญหานี้ทีละขั้นตอน

### 60.3 ขั้นตอนที่ 1: สร้าง .gitignore ที่เหมาะสม

```bash
cat > .gitignore << 'EOF'
# ===== Dependencies =====
node_modules/

# ===== Build output =====
dist/
build/

# ===== Test coverage =====
coverage/

# ===== Environment variables =====
.env
.env.local
.env.*.local

# ===== Logs (ยกเว้นไฟล์สำคัญ) =====
logs/*
!logs/important.log

# ===== Editor: VS Code (ทีมนี้เลือกไม่แชร์การตั้งค่า VS Code เลย) =====
.vscode/

# ===== OS-specific =====
.DS_Store
Thumbs.db
EOF
```

### 60.4 ขั้นตอนที่ 2: ตรวจสอบว่า pattern ทำงานถูกต้องก่อนแก้ไขจริง

```bash
git check-ignore -v node_modules/express/index.js dist/bundle.js .env logs/error.log logs/important.log .DS_Store app.js
```

**ผลลัพธ์:**
```
.gitignore:2:node_modules/	node_modules/express/index.js
.gitignore:5:dist/	dist/bundle.js
.gitignore:12:.env	.env
.gitignore:17:logs/*	logs/error.log
.gitignore:18:!logs/important.log	logs/important.log
.gitignore:24:.DS_Store	.DS_Store
```

สังเกตว่า `app.js` ไม่ปรากฏในผลลัพธ์เลย (เพราะไม่ถูก ignore ตามที่ตั้งใจ) และ `logs/important.log` ปรากฏพร้อมเครื่องหมาย `!` (แปลว่า negation ทำงานถูกต้อง ไฟล์นี้จะไม่ถูก ignore)

### 60.5 ขั้นตอนที่ 3: แก้ปัญหาไฟล์ที่ track ไปแล้วโดยไม่ตั้งใจ

เนื่องจากไฟล์เหล่านี้ถูก commit ไปแล้วก่อนมี `.gitignore` การเพิ่ม pattern อย่างเดียวไม่พอ ต้องเอาออกจาก tracking ด้วย `git rm --cached`

```bash
git rm -r --cached node_modules/ dist/ coverage/ .env .DS_Store logs/error.log .vscode/settings.json
```

**ผลลัพธ์:**
```
rm 'node_modules/express/index.js'
rm 'node_modules/lodash/index.js'
rm 'dist/bundle.js'
rm 'coverage/report.html'
rm '.env'
rm '.DS_Store'
rm 'logs/error.log'
rm '.vscode/settings.json'
```

**สังเกตสำคัญ:** เราจงใจ **ไม่** เอา `logs/important.log` ออก เพราะไฟล์นี้เราต้องการเก็บไว้ (มันมี negation pattern ยกเว้นไว้อยู่แล้ว) และ **ไม่** เอา `.vscode/settings.json` ออกในกรณีที่ทีมตกลงกันว่าจะแชร์ค่านี้ร่วมกัน — แต่ในตัวอย่างนี้เราสมมติว่า `.vscode/settings.json` เป็นค่าที่คนแรกตั้งไว้แบบเฉพาะตัว จึงเอาออกจาก tracking ด้วย (ทีมจริงต้องตกลงกันเองว่ากรณีไหนควรแชร์)

ตรวจสอบสถานะ:

```bash
git status
```

**ผลลัพธ์:**
```
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	deleted:    .DS_Store
	deleted:    .env
	deleted:    .vscode/settings.json
	deleted:    coverage/report.html
	deleted:    dist/bundle.js
	deleted:    logs/error.log
	deleted:    node_modules/express/index.js
	deleted:    node_modules/lodash/index.js

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.gitignore
```

commit การเปลี่ยนแปลงนี้:

```bash
git add .gitignore
git commit -m "Add .gitignore and remove previously tracked ignored files"
```

**ผลลัพธ์:**
```
[main 3d4e5f6] Add .gitignore and remove previously tracked ignored files
 9 files changed, 6 insertions(+), 8 deletions(-)
 create mode 100644 .gitignore
 delete mode 100644 .DS_Store
 delete mode 100644 .env
 delete mode 100644 .vscode/settings.json
 delete mode 100644 coverage/report.html
 delete mode 100644 dist/bundle.js
 delete mode 100644 logs/error.log
 delete mode 100644 node_modules/express/index.js
 delete mode 100644 node_modules/lodash/index.js
```

### 60.6 ขั้นตอนที่ 4: ตรวจสอบผลลัพธ์สุดท้ายให้แน่ใจว่าสะอาดจริง

```bash
git status --ignored
```

**ผลลัพธ์:**
```
On branch main
Ignored files:
  (use "git add -f <file>..." to include in what will be committed)
	.DS_Store
	.env
	.vscode/
	coverage/
	dist/
	logs/error.log
	node_modules/

nothing to commit, working tree clean
```

ตรวจสอบรายการไฟล์ที่ Git ยัง track อยู่ในโปรเจกต์:

```bash
git ls-files
```

**ผลลัพธ์:**
```
.gitignore
app.js
logs/important.log
package.json
```

นี่คือผลลัพธ์ที่ถูกต้องสมบูรณ์แบบ — เหลือเฉพาะไฟล์ที่ควร track จริง ๆ เท่านั้น: `.gitignore` (กติกากลาง), `app.js` และ `package.json` (ซอร์สโค้ดจริง), และ `logs/important.log` (ไฟล์ที่เรายกเว้นไว้ด้วย negation pattern อย่างตั้งใจ)

### 60.7 ขั้นตอนที่ 5: ทดสอบว่าไฟล์ใหม่ที่สร้างขึ้นในอนาคตถูก ignore อัตโนมัติ

ลองจำลองว่ามีคนรัน `npm install` ใหม่ในอนาคต (สร้างไฟล์ node_modules ใหม่):

```bash
mkdir -p node_modules/axios
echo "module.exports = {};" > node_modules/axios/index.js
echo "console.log('debug');" >> logs/error.log

git status
```

**ผลลัพธ์:**
```
On branch main
nothing to commit, working tree clean
```

ไฟล์ใหม่ที่สร้างขึ้นใน `node_modules/` และ `logs/error.log` ไม่ปรากฏใน `git status` เลย เพราะ `.gitignore` ทำงานครอบคลุมไฟล์ใหม่ ๆ ที่ตรงกับ pattern โดยอัตโนมัติ — นี่คือพฤติกรรมที่ถูกต้องและเป็นเป้าหมายของ Part นี้

### 60.8 สรุปคำสั่งทั้งหมดที่ใช้ในแบบฝึกหัดนี้

| คำสั่ง | หน้าที่ |
|---|---|
| `git status` | ดูสถานะไฟล์ปัจจุบัน |
| `git status --ignored` | ดูสถานะไฟล์รวมถึงไฟล์ที่ถูก ignore |
| `git check-ignore -v <file>` | ตรวจสอบว่า pattern ไหนทำให้ไฟล์ถูก ignore |
| `git rm --cached <file>` | หยุด track ไฟล์ (เก็บไฟล์จริงไว้) |
| `git rm -r --cached <dir>` | หยุด track ทั้งโฟลเดอร์แบบ recursive |
| `git ls-files` | แสดงรายการไฟล์ทั้งหมดที่ Git กำลัง track |
| `git ls-files --error-unmatch <file>` | ตรวจสอบว่าไฟล์นี้ถูก track อยู่หรือไม่ |

### 60.9 Checklist ก่อนไป Part 07

- [ ] เข้าใจว่าทำไมต้องมี `.gitignore` และไฟล์ประเภทไหนที่ไม่ควร track
- [ ] เขียน pattern พื้นฐานได้ (ชื่อไฟล์ตรงตัว, `*`, การใช้ `/` เจาะจงโฟลเดอร์)
- [ ] เขียน pattern ขั้นสูงได้ (negation `!`, `**`, เข้าใจลำดับความสำคัญของ pattern)
- [ ] รู้จักและใช้เทมเพลตจาก github/gitignore ได้
- [ ] เข้าใจความแตกต่างของ .gitignore ทั้ง 3 ระดับ (global, per-repo, per-directory)
- [ ] แก้ปัญหาไฟล์ที่ track ไปแล้วด้วย `git rm --cached` ได้
- [ ] รู้จัก `.git/info/exclude` และรู้ว่าต่างจาก `.gitignore` อย่างไร
- [ ] ใช้ `git check-ignore -v` และ `git status --ignored` วินิจฉัยปัญหาได้
- [ ] รู้จักข้อผิดพลาดที่พบบ่อยและวิธีป้องกัน
- [ ] ทำแบบฝึกหัดสร้างโปรเจกต์จำลองและแก้ปัญหาไฟล์หลุด track ได้ครบวงจรด้วยตัวเอง

---

## สรุป Part 06

ใน Part นี้เราได้เรียนรู้ว่า:

1. โปรเจกต์จริงมีไฟล์จำนวนมากที่ไม่ควร track เช่น build artifacts, dependencies ขนาดใหญ่, ไฟล์ที่มีข้อมูลลับ, ไฟล์ระบบปฏิบัติการ และไฟล์ของ IDE — การไม่จัดการเรื่องนี้ตั้งแต่ต้นนำไปสู่ repository ที่บวม ทำงานร่วมกันยาก และเสี่ยงข้อมูลรั่วไหล
2. `.gitignore` ใช้ syntax พื้นฐานอย่างชื่อไฟล์ตรงตัว, wildcard `*`, และ `/` เพื่อเจาะจงตำแหน่งโฟลเดอร์
3. Pattern ขั้นสูงอย่าง negation (`!`), `**` สำหรับหลายระดับไดเรกทอรี และการเข้าใจลำดับความสำคัญของ pattern (บรรทัดหลังชนะบรรทัดก่อน) เป็นกุญแจสำคัญที่ทำให้เขียน `.gitignore` ที่ซับซ้อนได้ถูกต้อง
4. เทมเพลตจาก github/gitignore เป็นจุดเริ่มต้นที่ดีเยี่ยม แต่ต้องอ่านทบทวนและปรับแต่งให้เข้ากับโปรเจกต์จริงเสมอ
5. Git รองรับการ ignore หลายระดับ: Global (`core.excludesFile`) สำหรับทุก repo บนเครื่อง, Per-repo `.gitignore` ที่แชร์กับทีม, และ Per-directory `.gitignore` ที่มีผลเฉพาะโฟลเดอร์ย่อย
6. `.gitignore` มีผลเฉพาะไฟล์ที่ยังไม่เคย track เท่านั้น — ไฟล์ที่ track ไปแล้วต้องใช้ `git rm --cached` เพื่อหยุด track โดยไม่ลบไฟล์จริง
7. `.git/info/exclude` คือกลไก ignore เฉพาะเครื่องที่ไม่ถูกแชร์กับทีม เหมาะกับ preference ส่วนตัวที่ไม่เกี่ยวกับโปรเจกต์
8. `git check-ignore -v` และ `git status --ignored` เป็นเครื่องมือวินิจฉัยที่สำคัญที่สุดเมื่อสงสัยว่าทำไมไฟล์ถูกหรือไม่ถูก ignore
9. ข้อผิดพลาดที่พบบ่อยส่วนใหญ่มาจากความเข้าใจผิดเรื่องไฟล์ที่ track ไปแล้ว, negation ที่อยู่ในโฟลเดอร์ที่ถูก ignore ทั้งโฟลเดอร์, และการสับสนระหว่าง `/build/` กับ `build/`
10. เราได้ฝึกภาคปฏิบัติแบบครบวงจร ตั้งแต่จำลองปัญหาไฟล์หลุด track ไปจนถึงแก้ไขให้ repository สะอาดสมบูรณ์

**ต่อไป:** [Part 07: Branch คืออะไร: สร้าง สลับ ลบ Branch](./part-007-branch-สร้าง-สลับ-ลบ.md)
