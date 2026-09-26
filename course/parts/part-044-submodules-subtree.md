# Part 44: Submodules และ Subtree: จัดการ Repo ซ้อน Repo

> **Step ในหลักสูตรนี้:** Step 431–440
> **เฟส:** 4 — ทำงานเป็นทีมด้วย Workflow มาตรฐาน (Git Flow, GitHub Flow, Rebase)
> **เป้าหมายของ Part นี้:** เข้าใจปัญหาของการมี "repo ซ้อน repo" หรือโปรเจกต์ที่ต้องพึ่งพาโค้ดจาก repo อื่นเป็น dependency แต่ยังต้อง track เวอร์ชันของมันเองอย่างละเอียด แล้วเรียนรู้เครื่องมือสองตัวที่ Git มีไว้แก้ปัญหานี้คือ **Git Submodule** และ **Git Subtree** ให้ใช้งานได้จริง เข้าใจข้อดีข้อเสียของแต่ละแบบ และเลือกใช้ได้ถูกต้องตามสถานการณ์

---

## สารบัญของ Part นี้

- Step 431: ปัญหาที่ Submodule/Subtree แก้ — เมื่อโปรเจกต์ต้องพึ่งพา repo อื่นที่ยังต้อง track เวอร์ชันเอง
- Step 432: Git Submodule คืออะไร โครงสร้างไฟล์ `.gitmodules`
- Step 433: การเพิ่ม Submodule (`git submodule add`)
- Step 434: การ Clone Repo ที่มี Submodule
- Step 435: การอัปเดต Submodule ไปเวอร์ชันใหม่ (`git submodule update --remote`)
- Step 436: ปัญหาที่พบบ่อยกับ Submodule (Detached HEAD, ทีมลืม Update)
- Step 437: Git Subtree คืออะไร ต่างจาก Submodule อย่างไร
- Step 438: การใช้งาน Subtree (`git subtree add/pull/push`)
- Step 439: ตารางเปรียบเทียบ Submodule vs Subtree แบบละเอียด — เลือกใช้เมื่อไหร่
- Step 440: แบบฝึกหัด — สร้างโปรเจกต์ที่ใช้ Submodule จริง และทดลอง Subtree เพื่อเปรียบเทียบ

---

## Step 431: ปัญหาที่ Submodule/Subtree แก้ — เมื่อโปรเจกต์ต้องพึ่งพา repo อื่นที่ยังต้อง track เวอร์ชันเอง

ก่อนจะเรียนรู้เครื่องมือ เราต้องเข้าใจปัญหาที่แท้จริงก่อนว่ามันเกิดขึ้นตอนไหน เพราะถ้าไม่เข้าใจปัญหา จะหยิบเครื่องมือมาใช้ผิดที่ผิดทางได้ง่ายมาก

### สถานการณ์ตัวอย่างที่เกิดขึ้นจริง

ลองนึกภาพสถานการณ์ต่อไปนี้ ซึ่งเกิดขึ้นบ่อยมากในการพัฒนาซอฟต์แวร์ระดับทีม/องค์กร:

1. **บริษัทมีทีมกลางที่ดูแล "shared library" หรือ "design system"** ที่หลายโปรเจกต์ในบริษัทต้องใช้ร่วมกัน เช่น component library, utility functions, หรือ shared configuration
2. **โปรเจกต์ A ต้องการใช้โค้ดจาก repo `company-ui-kit`** เป็นส่วนหนึ่งของโปรเจกต์ แต่ทีม A ไม่ได้เป็นเจ้าของ repo นั้น และไม่ต้องการ copy-paste โค้ดมาทับไว้เฉย ๆ เพราะจะทำให้อัปเดตยาก
3. **โปรเจกต์ Firmware ของอุปกรณ์ IoT ต้องพึ่งพา third-party driver library** ที่เป็น open source อยู่บน GitHub แต่ทีมต้อง pin เวอร์ชันที่แน่นอนไว้ เพราะเวอร์ชันใหม่กว่าอาจมี breaking change ที่กระทบฮาร์ดแวร์
4. **โปรเจกต์เกมที่แยก engine กับ game logic ออกจากกันเป็นคนละ repo** เพื่อให้ทีม engine พัฒนาแยกจากทีม gameplay ได้ แต่สุดท้ายต้อง build รวมกันเป็นโปรเจกต์เดียว

### ทำไมวิธีง่าย ๆ ถึงใช้ไม่ได้

ทีนี้ลองคิดว่าถ้าเราแก้ปัญหานี้แบบ "ง่าย ๆ" มันจะเกิดอะไรขึ้น:

**วิธีที่ 1: Copy-paste โค้ดเข้ามาตรง ๆ**

```
project-a/
├── src/
├── lib/
│   └── company-ui-kit/     ← copy โค้ดมาวางตรงนี้เลย
└── README.md
```

ปัญหา:
- เมื่อ `company-ui-kit` มีการอัปเดต bug fix หรือฟีเจอร์ใหม่ ทีม A ต้อง copy โค้ดมาทับใหม่เองด้วยมือ ซึ่งเสี่ยงต่อการลืม เสี่ยงต่อการทับ custom patch ที่ทีม A เคยแก้ไว้เอง
- ไม่มีทางรู้เลยว่าโค้ดที่ copy มานั้นคือ**เวอร์ชันไหน**ของ `company-ui-kit` เพราะไม่มี metadata อ้างอิงกลับไปที่ commit ต้นทาง
- Git history ของ `project-a` จะเต็มไปด้วย commit ขนาดใหญ่ที่เป็นการ "แปะโค้ดคนอื่นทับ" ทำให้ history อ่านยากและสับสนระหว่างโค้ดของทีม A เองกับโค้ดที่ยืมมา

**วิธีที่ 2: รวมทุกอย่างไว้ใน monorepo เดียวจริง ๆ**

บางทีมแก้ปัญหาด้วยการรวมทุก repo เข้าเป็น repo เดียวใหญ่ (monorepo) ซึ่งเป็นวิธีที่ใช้ได้จริงในหลายองค์กรใหญ่ (เช่น Google, Facebook) แต่มันมีข้อจำกัดคือ:
- ต้องมี tooling พิเศษรองรับ monorepo ขนาดใหญ่ (build system แบบ Bazel, incremental build ฯลฯ)
- ถ้า dependency นั้นเป็น repo ของ**บุคคลภายนอก** (open source library) เราไม่สามารถ "ยึด" มันมารวมเป็น repo เดียวกับเราได้ เพราะเจ้าของ library ยังคง maintain repo ของเขาแยกต่างหากอยู่ตลอดเวลา

### สิ่งที่เราต้องการจริง ๆ

จากปัญหาทั้งหมดนี้ สรุปได้ว่าสิ่งที่เราต้องการคือความสามารถที่จะ:

1. **ดึงโค้ดจาก repo อื่นเข้ามาเป็นส่วนหนึ่งของโปรเจกต์เรา** (ไม่ต้อง copy-paste ด้วยมือ)
2. **ยังคง track ได้ว่าโค้ดที่ดึงมานั้นคือเวอร์ชัน (commit) ไหนของ repo ต้นทาง** อย่างชัดเจน
3. **อัปเดตไปยังเวอร์ชันใหม่ได้ง่าย** เมื่อ repo ต้นทางมีการเปลี่ยนแปลง โดยไม่ต้อง copy ทับด้วยมือ
4. **repo ต้นทางยังคงเป็นอิสระ** — ทีมที่ maintain มันสามารถพัฒนาต่อไปเรื่อย ๆ โดยไม่ต้องรู้จักหรือสนใจโปรเจกต์ที่มาดึงโค้ดไปใช้เลย

นี่คือปัญหาที่แท้จริงที่ **Git Submodule** และ **Git Subtree** ถูกออกแบบมาแก้ไข ทั้งสองตัวมีเป้าหมายเดียวกันคือ "ให้ repo หนึ่งสามารถอ้างอิงและรวมโค้ดจาก repo อื่นได้อย่างมีระบบ" แต่ใช้**แนวคิดและกลไกที่ต่างกันโดยสิ้นเชิง** ซึ่งเราจะไล่เรียนรู้ทีละตัวใน Step ถัดไป

> **ข้อสังเกตสำคัญ:** ทั้ง Submodule และ Subtree ไม่ใช่ "dependency manager" แบบ npm, pip, หรือ Maven ที่จัดการเวอร์ชันผ่าน package registry มันคือกลไกระดับ **Git repository** ล้วน ๆ ที่ทำให้ repo หนึ่งฝัง repo อื่นไว้ข้างในได้ เหมาะกับกรณีที่ dependency นั้นเป็น source code ที่ต้องแก้ไข/build ร่วมกัน ไม่ใช่ package ที่ compile สำเร็จรูปแล้ว

---

## Step 432: Git Submodule คืออะไร โครงสร้างไฟล์ `.gitmodules`

### แนวคิดหลักของ Submodule

**Git Submodule** คือกลไกที่ให้คุณฝัง **repository หนึ่งไว้เป็น subdirectory ภายใน repository หลัก (parent repo)** โดยที่ repo ที่ถูกฝังเข้าไปนั้น **ยังคงเป็น repository อิสระของตัวเอง** ที่มี `.git` และประวัติ commit เป็นของตัวเองแยกขาดจาก parent repo โดยสิ้นเชิง

สิ่งสำคัญที่สุดที่ต้องเข้าใจคือ:

> **Parent repo ไม่ได้เก็บโค้ดของ submodule ไว้จริง ๆ มันเก็บแค่ "pointer" ที่ชี้ไปยัง commit hash เดียวที่แน่นอนของ submodule repo เท่านั้น**

พูดง่าย ๆ submodule ใน Git คือการบันทึกไว้ว่า "ที่ path นี้ ให้ไปดึงข้อมูลจาก repo URL นี้ ที่ commit hash นี้แม่นยำ" — มันไม่ใช่การ copy โค้ดเข้ามาเก็บใน parent repo แต่อย่างใด

### โครงสร้างของ Submodule ใน Repository

เมื่อคุณเพิ่ม submodule เข้าไปใน repo (วิธีการเพิ่มจะอธิบายละเอียดใน Step 433) โครงสร้างไฟล์จะเป็นดังนี้:

```
my-project/                    ← Parent repo
├── .git/                      ← Git ของ parent repo
├── .gitmodules                ← ไฟล์ config บอกว่ามี submodule อะไรบ้าง
├── src/
│   └── main.js
├── libs/
│   └── shared-utils/          ← Submodule (repo อื่นที่ถูกฝังไว้)
│       ├── .git                ← ไฟล์ (ไม่ใช่โฟลเดอร์!) ที่ชี้กลับไปยัง .git ของ parent
│       ├── utils.js
│       └── README.md
└── README.md
```

สังเกตว่าภายในโฟลเดอร์ `libs/shared-utils/` จะมีสิ่งที่ดูเหมือน `.git` แต่จริง ๆ แล้วมันเป็น**ไฟล์ข้อความธรรมดา** ไม่ใช่โฟลเดอร์ ที่มีเนื้อหาชี้กลับไปยังตำแหน่งจัดเก็บข้อมูล git จริงของ submodule ซึ่งถูกเก็บไว้ที่ `.git/modules/libs/shared-utils/` ของ parent repo

### ไฟล์ `.gitmodules` — หัวใจสำคัญของกลไก Submodule

เมื่อคุณเพิ่ม submodule เข้าไป Git จะสร้างไฟล์ชื่อ `.gitmodules` ไว้ที่ root ของ parent repo โดยอัตโนมัติ ไฟล์นี้เป็น text file รูปแบบคล้าย INI ที่บันทึกข้อมูลของ submodule ทุกตัวที่มีอยู่ในโปรเจกต์

ตัวอย่างเนื้อหาไฟล์ `.gitmodules`:

```ini
[submodule "libs/shared-utils"]
	path = libs/shared-utils
	url = https://github.com/company/shared-utils.git
	branch = main

[submodule "libs/vendor-driver"]
	path = libs/vendor-driver
	url = https://github.com/vendor/driver-lib.git
```

อธิบายแต่ละ field:

| Field | ความหมาย |
|---|---|
| `[submodule "<name>"]` | ชื่อของ submodule (มักตั้งตาม path) |
| `path` | ตำแหน่ง (path) ภายใน parent repo ที่ submodule นี้จะถูกวางไว้ |
| `url` | URL ของ remote repository ต้นทางของ submodule |
| `branch` | (ทางเลือก) branch ที่จะใช้อ้างอิงเวลาสั่ง `git submodule update --remote` (ถ้าไม่ระบุจะ default ไปที่ branch ที่ submodule ถูก set ไว้ตอน clone ครั้งแรก หรือ `HEAD` ของ remote) |

**สิ่งสำคัญที่ต้องเข้าใจ:** ไฟล์ `.gitmodules` เป็นไฟล์ที่ **ถูก commit เข้าไปใน parent repo เหมือนไฟล์ปกติทั่วไป** ดังนั้นทุกคนที่ clone parent repo จะเห็นไฟล์นี้และรู้ว่าโปรเจกต์นี้มี submodule อะไรบ้าง อยู่ที่ URL ไหน

### Special Entry ใน Git Index

นอกจากไฟล์ `.gitmodules` แล้ว Git ยังบันทึกข้อมูลอีกชิ้นหนึ่งที่สำคัญไม่แพ้กัน คือใน **Git index (staging area)** ของ parent repo จะมี entry พิเศษสำหรับ path ของ submodule ที่มี **mode `160000`** (แทนที่จะเป็น `100644` แบบไฟล์ปกติ) และ entry นี้จะเก็บ **SHA-1 hash ของ commit ที่ submodule ชี้ไปอยู่** ไม่ใช่เนื้อหาไฟล์

ลองดูตัวอย่างผลลัพธ์จากคำสั่ง `git ls-tree HEAD` ในโปรเจกต์ที่มี submodule:

```bash
$ git ls-tree HEAD
100644 blob a1b2c3d4...    .gitmodules
100644 blob e5f6g7h8...    README.md
040000 tree i9j0k1l2...    src
160000 commit m3n4o5p6...  libs/shared-utils
```

สังเกตบรรทัดสุดท้าย: type คือ `commit` (ไม่ใช่ `blob` หรือ `tree`) และ mode คือ `160000` นี่คือสิ่งที่ Git เรียกว่า **"gitlink"** — มันคือการอ้างอิงไปยัง commit ของ repository อื่นโดยตรง ซึ่งเป็นกลไกระดับ low-level ที่ทำให้ submodule ทำงานได้

### สรุปแนวคิดของ Submodule ด้วยประโยคเดียว

> **Submodule = parent repo เก็บแค่ "ที่อยู่ (URL) + commit hash ที่แน่นอน" ของ repo ลูก ไม่ได้เก็บโค้ดจริงของ repo ลูกไว้เลย**

การเข้าใจแนวคิดนี้ให้แม่นเป็นสิ่งสำคัญที่สุด เพราะมันอธิบายพฤติกรรมแปลก ๆ ของ submodule เกือบทั้งหมดที่มือใหม่มักงงใน Step ถัด ๆ ไป เช่น ทำไม clone แล้วโฟลเดอร์ submodule ถึงว่างเปล่า ทำไมต้อง `init` และ `update` แยกกัน

---

## Step 433: การเพิ่ม Submodule (`git submodule add`)

### คำสั่งพื้นฐาน

การเพิ่ม submodule เข้าไปใน repo ที่มีอยู่แล้วทำได้ด้วยคำสั่ง:

```bash
git submodule add <url> <path>
```

ตัวอย่างจริง:

```bash
git submodule add https://github.com/company/shared-utils.git libs/shared-utils
```

เมื่อรันคำสั่งนี้ Git จะทำสิ่งต่อไปนี้ให้อัตโนมัติ:

1. **Clone repository จาก URL ที่ระบุ** มาไว้ที่ path ที่กำหนด (`libs/shared-utils`)
2. **สร้างหรือแก้ไขไฟล์ `.gitmodules`** ที่ root ของ parent repo เพื่อบันทึกข้อมูล submodule ใหม่นี้
3. **เพิ่ม entry แบบ gitlink (mode 160000)** เข้าไปใน staging area ของ parent repo ที่ path นั้น ชี้ไปยัง commit ปัจจุบันของ submodule (โดยปกติคือ commit ล่าสุดของ default branch)
4. **stage ไฟล์ `.gitmodules` และ gitlink entry นั้นให้พร้อม commit**

### ขั้นตอนแบบละเอียด

มาดูขั้นตอนแบบเต็มตั้งแต่ต้นจนจบ:

```bash
# 1. อยู่ใน parent repo อยู่แล้ว
cd my-project

# 2. เพิ่ม submodule
git submodule add https://github.com/company/shared-utils.git libs/shared-utils

# ผลลัพธ์ที่จะเห็น:
# Cloning into '/path/to/my-project/libs/shared-utils'...
# remote: Enumerating objects: 120, done.
# ...
# Resolving deltas: 100% (45/45), done.

# 3. ตรวจสอบสถานะ
git status
```

ผลลัพธ์ของ `git status` หลังเพิ่ม submodule:

```
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   .gitmodules
	new file:   libs/shared-utils
```

สังเกตว่า Git มองว่า `libs/shared-utils` เป็น "ไฟล์" ไม่ใช่ "โฟลเดอร์ที่มีไฟล์ข้างในหลายไฟล์" ทั้งนี้เพราะจากมุมมองของ parent repo มันคือ gitlink entry เดียว ไม่ใช่ tree ของไฟล์ย่อย

### การ Commit การเพิ่ม Submodule

หลังจากเพิ่ม submodule แล้ว ต้อง commit เพื่อบันทึกการเปลี่ยนแปลงนี้ลงใน history ของ parent repo:

```bash
git commit -m "Add shared-utils as submodule"
```

**สิ่งสำคัญ:** commit นี้จะบันทึกแค่:
- ไฟล์ `.gitmodules` ที่อัปเดตแล้ว
- gitlink ที่ path `libs/shared-utils` ซึ่งชี้ไปยัง commit hash หนึ่งของ repo `shared-utils`

มันจะ**ไม่บันทึกโค้ดจริงข้างในของ `shared-utils` เข้าไปใน object database ของ parent repo เลย** — โค้ดจริงยังคงอยู่ใน `.git` ของตัว submodule เอง (ที่จริง ๆ ถูกเก็บไว้ที่ `.git/modules/libs/shared-utils/` ของ parent repo แต่ในระดับ object model มันแยกกันคนละชุดกับ object ของ parent repo)

### การระบุ Branch ที่ต้องการ track

ถ้าต้องการให้ submodule ผูกกับ branch ใดเป็นพิเศษ (แทนที่จะเป็นแค่ commit เดียวแบบตายตัว) สามารถระบุ `-b` ได้ตอนเพิ่ม:

```bash
git submodule add -b develop https://github.com/company/shared-utils.git libs/shared-utils
```

คำสั่งนี้จะเพิ่ม field `branch = develop` เข้าไปในไฟล์ `.gitmodules` ด้วย ซึ่งจะมีผลตอนใช้คำสั่ง `git submodule update --remote` (จะอธิบายใน Step 435) — มันจะไปดึง commit ล่าสุดจาก branch `develop` แทนที่จะเป็น default branch

### เพิ่ม Submodule ที่ Commit เจาะจง

บางครั้งคุณต้องการ pin submodule ไว้ที่ commit หรือ tag เฉพาะเจาะจงตั้งแต่แรก (ไม่ใช่ commit ล่าสุด) สามารถทำได้โดย `add` ตามปกติก่อน แล้วเข้าไป checkout commit ที่ต้องการภายใน submodule นั้น:

```bash
git submodule add https://github.com/vendor/driver-lib.git libs/vendor-driver
cd libs/vendor-driver
git checkout v2.3.1        # checkout ไปที่ tag เวอร์ชันที่ต้องการ
cd ../..
git add libs/vendor-driver  # stage การเปลี่ยนแปลง pointer ใหม่
git commit -m "Pin vendor-driver to v2.3.1"
```

### ไฟล์ `.git/config` ของ Parent Repo ก็ถูกแก้ไขด้วย

นอกจาก `.gitmodules` แล้ว คำสั่ง `git submodule add` ยังเขียนข้อมูล remote ของ submodule เข้าไปใน `.git/config` ของ parent repo (ไฟล์นี้**ไม่ถูก commit** เพราะเป็น local config) ตัวอย่าง:

```ini
[submodule "libs/shared-utils"]
	url = https://github.com/company/shared-utils.git
	active = true
```

การมี config ซ้อนกันสองที่ (`.gitmodules` ที่ถูก commit และ `.git/config` ที่เป็น local) เป็นสิ่งที่ทำให้ submodule มีความยืดหยุ่นแบบหนึ่ง — เช่น local เราสามารถ override URL เป็น URL อื่น (เช่นใช้ SSH แทน HTTPS) ได้โดยไม่กระทบไฟล์ `.gitmodules` ที่แชร์กับทีม ด้วยคำสั่ง:

```bash
git config submodule.libs/shared-utils.url git@github.com:company/shared-utils.git
```

---

## Step 434: การ Clone Repo ที่มี Submodule

นี่คือจุดที่มือใหม่งงกันบ่อยที่สุด เพราะพฤติกรรม default ของ Git ตอน `clone` **ไม่ได้ดึงเนื้อหาของ submodule มาให้อัตโนมัติ**

### ปัญหา: Clone แบบปกติแล้วโฟลเดอร์ Submodule ว่างเปล่า

ลองสมมติว่ามีคนอื่น clone repo `my-project` (ที่มี submodule `shared-utils`) แบบปกติ:

```bash
git clone https://github.com/company/my-project.git
cd my-project
ls libs/shared-utils/
```

ผลลัพธ์ที่ได้คือ **โฟลเดอร์ว่างเปล่า** ไม่มีไฟล์อะไรเลย! นี่ไม่ใช่บั๊ก แต่เป็นพฤติกรรมที่ตั้งใจออกแบบไว้ เพราะ `git clone` ปกติจะดึงมาแค่ตัว parent repo (รวมถึงไฟล์ `.gitmodules` และ gitlink entry) แต่**จะไม่ไป clone submodule repo ตามให้อัตโนมัติ** เพราะ Git ไม่อยากบังคับดึงข้อมูลจำนวนมากจาก URL ภายนอกโดยที่ผู้ใช้ไม่รู้ตัว (โดยเฉพาะถ้ามี submodule ซ้อนกันหลายชั้น หรือ submodule ชี้ไป private repo ที่ต้อง authenticate)

### วิธีที่ 1: Clone พร้อม Submodule ตั้งแต่ต้น (`--recurse-submodules`)

วิธีที่สะดวกที่สุดคือเพิ่ม flag `--recurse-submodules` ตอน clone:

```bash
git clone --recurse-submodules https://github.com/company/my-project.git
```

คำสั่งนี้จะทำสองอย่างในทีเดียว:
1. Clone parent repo ตามปกติ
2. หลังจาก clone เสร็จ จะอ่านไฟล์ `.gitmodules` แล้ววิ่งไป `init` และ `update` submodule ทุกตัวโดยอัตโนมัติ (รวมถึง submodule ที่ซ้อนอยู่ใน submodule อีกที ถ้ามี — เพราะเป็นการทำงานแบบ recursive ตามชื่อ flag)

หลังรันคำสั่งนี้ `libs/shared-utils/` จะมีไฟล์ครบถ้วนตามที่ควรจะเป็น

### วิธีที่ 2: Clone ปกติก่อน แล้วค่อย Init/Update ทีหลัง

ถ้าคุณ (หรือเพื่อนร่วมทีม) ลืม flag `--recurse-submodules` ตอน clone ไปแล้ว หรือ clone มาจากที่อื่นที่ไม่ผ่านคำสั่งของคุณเอง สามารถแก้ไขทีหลังได้ด้วย 2 คำสั่งนี้:

```bash
git submodule init
git submodule update
```

หรือรวมเป็นคำสั่งเดียวโดยใช้ flag `--init`:

```bash
git submodule update --init
```

และถ้ามี submodule ซ้อนกันหลายชั้น (submodule ของ submodule) ให้เพิ่ม `--recursive`:

```bash
git submodule update --init --recursive
```

### อธิบายความแตกต่างระหว่าง `init` กับ `update`

คำสั่งทั้งสองทำหน้าที่ต่างกันชัดเจน:

| คำสั่ง | หน้าที่ |
|---|---|
| `git submodule init` | อ่านไฟล์ `.gitmodules` แล้วคัดลอกข้อมูล (URL, path) เข้าไปเก็บไว้ใน local config (`.git/config`) ของ parent repo — **ยังไม่ได้ดึงโค้ดใด ๆ มา** |
| `git submodule update` | ไปดึง (clone/fetch) โค้ดจาก URL ที่ตั้งค่าไว้ (จาก `init` ก่อนหน้า) มาวางที่ path ที่กำหนด แล้ว checkout ไปยัง commit ที่ parent repo บันทึกไว้ว่าเป็น pointer ปัจจุบัน |

การแยกสองขั้นตอนนี้ออกจากกันทำให้เกิดความยืดหยุ่น เช่น คุณอาจต้องการ `init` ก่อนแล้วไปแก้ URL ใน local config ให้เป็น mirror ภายในบริษัท ก่อนจะสั่ง `update` จริง — เพื่อประหยัด bandwidth หรือหลีกเลี่ยงปัญหาการเข้าถึง network ภายนอก

### ตั้งค่าให้ Clone ในอนาคตดึง Submodule มาด้วยเสมอ (แนะนำสำหรับทีม)

เพื่อลดปัญหาคนลืม flag `--recurse-submodules` สามารถตั้งค่า Git ให้ default behavior เป็นแบบ recursive ได้ระดับ global:

```bash
git config --global submodule.recurse true
```

การตั้งค่านี้จะทำให้คำสั่งหลายตัวที่เกี่ยวข้องกับ submodule (เช่น `checkout`, `pull`) มีพฤติกรรม recursive อัตโนมัติมากขึ้น อย่างไรก็ตาม **มันไม่มีผลกับคำสั่ง `git clone` โดยตรง** — `clone` ยังคงต้องใช้ `--recurse-submodules` อยู่ดี (นี่คือข้อยกเว้นสำคัญที่ต้องจำ)

---

## Step 435: การอัปเดต Submodule ไปเวอร์ชันใหม่ (`git submodule update --remote`)

### ความเข้าใจผิดที่พบบ่อย: `git pull` ใน Parent Repo ไม่ได้อัปเดต Submodule ให้อัตโนมัติ

นี่คือจุดสำคัญมาก: ถ้าคุณสั่ง `git pull` ใน parent repo และมีคนอื่นในทีม**อัปเดต pointer ของ submodule ไปยัง commit ใหม่** (เช่นเขา `add`/`commit` การเปลี่ยน pointer submodule เข้ามาใน parent repo) การ `pull` ของคุณจะดึงแค่ **การเปลี่ยนแปลงของ gitlink entry** (คือ pointer ตัวเลข commit hash ใหม่) เข้ามา แต่**จะไม่ไปดึงเนื้อหาจริงของ submodule repo ที่ commit ใหม่นั้นมาให้อัตโนมัติ**

พูดอีกแบบ: หลัง `git pull` เสร็จ โฟลเดอร์ submodule ของคุณจะยังคง checkout อยู่ที่ **commit เก่า** และ `git status` จะรายงานว่า submodule นั้น "มีการเปลี่ยนแปลง" (เพราะ commit ที่ checkout อยู่ไม่ตรงกับ pointer ใหม่ที่ parent repo บอกไว้)

วิธีแก้คือต้องรันคำสั่งเพิ่มเติมหลัง pull เสมอ:

```bash
git pull
git submodule update --init --recursive
```

หลายทีมนิยมสร้าง alias หรือ script ผสมสองคำสั่งนี้เข้าด้วยกัน เพื่อลดโอกาสลืม เช่น:

```bash
git config --global alias.pullall '!git pull && git submodule update --init --recursive'
```

### การอัปเดต Submodule ไปยัง Commit ล่าสุดของ Remote (`--remote`)

สถานการณ์ที่ต่างออกไปคือ เมื่อคุณ**ต้องการอัปเกรด submodule เอง**ไปยังเวอร์ชันใหม่ล่าสุดที่ repo ต้นทางมีอยู่ (ไม่ใช่แค่ sync ตาม pointer ที่คนอื่น commit ไว้แล้ว) ใช้คำสั่ง:

```bash
git submodule update --remote
```

คำสั่งนี้จะทำสิ่งที่ต่างจาก `git submodule update` ธรรมดาอย่างชัดเจน:

- `git submodule update` (ไม่มี `--remote`) → checkout submodule ไปที่ **commit ที่ parent repo บันทึกไว้ (gitlink ปัจจุบัน)** เท่านั้น
- `git submodule update --remote` → ไป **fetch จาก remote ของ submodule** แล้ว checkout ไปที่ **commit ล่าสุดของ branch ที่ระบุไว้ใน `.gitmodules`** (หรือ default branch ถ้าไม่ได้ระบุ `branch` ไว้)

ตัวอย่างขั้นตอนเต็ม:

```bash
# อัปเดต submodule ตัวหนึ่งไปยังเวอร์ชันล่าสุดของ branch ที่กำหนด
git submodule update --remote libs/shared-utils

# ตรวจสอบว่า pointer เปลี่ยนไปแล้ว
git status
# Changes not staged for commit:
#   modified:   libs/shared-utils (new commits)

# ดูว่ามีอะไรเปลี่ยนแปลงบ้างระหว่าง commit เก่ากับใหม่ของ submodule
git diff libs/shared-utils

# เมื่อพอใจแล้ว ให้ commit การอัปเดต pointer นี้เข้า parent repo
git add libs/shared-utils
git commit -m "Update shared-utils submodule to latest version"
```

**ข้อควรจำสำคัญ:** การรัน `git submodule update --remote` **ไม่ commit อัตโนมัติ** — มันแค่เปลี่ยนสิ่งที่ checkout อยู่ในโฟลเดอร์ submodule เท่านั้น คุณยังต้อง `add` และ `commit` การเปลี่ยนแปลง pointer นี้เข้าไปใน parent repo ด้วยตัวเองเสมอ ไม่งั้นเพื่อนร่วมทีมคนอื่นจะไม่เห็นการอัปเดตนี้เลย

### อัปเดตทุก Submodule พร้อมกัน

ถ้าโปรเจกต์มี submodule หลายตัว สามารถอัปเดตทั้งหมดพร้อมกันได้ด้วย flag `--merge` หรือแค่ไม่ระบุ path เจาะจง:

```bash
git submodule update --remote --merge
```

flag `--merge` ในที่นี้หมายถึงการ merge การเปลี่ยนแปลงของ submodule เข้ากับ local branch ของ submodule นั้น (ถ้า submodule ของคุณ checkout อยู่บน branch ไม่ใช่ detached HEAD) ซึ่งเชื่อมโยงกับปัญหาที่เราจะพูดถึงใน Step ถัดไป

---

## Step 436: ปัญหาที่พบบ่อยกับ Submodule (Detached HEAD, ทีมลืม Update)

Submodule เป็นเครื่องมือที่ทรงพลัง แต่ก็ขึ้นชื่อเรื่องความยุ่งยากในการใช้งานจริงมากที่สุดอย่างหนึ่งใน Git ในหัวข้อนี้เราจะไล่ดูปัญหาที่พบบ่อยที่สุดทีละข้อ

### ปัญหาที่ 1: Submodule อยู่ในสถานะ Detached HEAD เสมอ

เมื่อคุณ `git submodule update` (ไม่ว่าจะครั้งแรกตอน clone หรือครั้งไหนก็ตาม) Git จะ **checkout submodule ไปที่ commit hash ที่แน่นอน** ไม่ใช่ checkout ไปที่ชื่อ branch ใด ๆ

ผลลัพธ์คือถ้าคุณเข้าไปดูสถานะข้างในโฟลเดอร์ submodule:

```bash
cd libs/shared-utils
git status
```

จะเห็นข้อความว่า:

```
HEAD detached at a1b2c3d
nothing to commit, working tree clean
```

นี่คือสถานะที่เรียกว่า **detached HEAD** — คือ HEAD ชี้ตรงไปที่ commit hash แทนที่จะชี้ผ่าน branch reference ใด ๆ

**ทำไมมันเป็นปัญหา:** ถ้าคุณลืมตัวแล้วเริ่มแก้ไขโค้ดและ `git commit` ขณะที่อยู่ใน detached HEAD ข้างใน submodule commit ใหม่นั้นจะ**ไม่ได้อยู่บน branch ไหนเลย** ถ้าคุณสลับไปทำงานที่อื่นแล้วกลับมา (หรือมีคนอื่นสั่ง `git submodule update` ทับ) commit ที่คุณทำไปอาจกลายเป็น **"orphan commit"** ที่ไม่มี reference ใดชี้ถึง และจะถูก Git garbage collector ลบทิ้งไปในที่สุด (โค้ดหายจริง ๆ!)

**วิธีป้องกัน/แก้ไข:** ถ้าต้องการแก้ไขโค้ดข้างใน submodule จริง ๆ ต้อง checkout ไปที่ branch ก่อนเสมอ:

```bash
cd libs/shared-utils
git checkout main          # หรือ branch อื่นที่เหมาะสม
# แก้ไขโค้ด, commit ตามปกติ
git push origin main        # push เข้า remote ของ submodule เอง
cd ../..
git add libs/shared-utils   # อัปเดต pointer ใน parent repo ให้ตรงกับ commit ใหม่
git commit -m "Update shared-utils pointer after fix"
```

หรือใช้ flag `--checkout` (default) เทียบกับ `--merge`/`--rebase` ตอน `submodule update` เพื่อควบคุมพฤติกรรมนี้:

```bash
git submodule update --remote --rebase   # rebase local commits ของ submodule (ถ้ามี) บน branch ใหม่
git submodule update --remote --merge    # merge แทนที่จะ checkout ตรง ๆ
```

### ปัญหาที่ 2: ทีมลืมรัน `submodule update` หลัง Pull

ปัญหานี้เป็นปัญหาที่**พบบ่อยที่สุดในทีมที่ใช้ submodule** อาการที่พบคือ:

- Developer A อัปเดต submodule pointer และ push เข้า parent repo
- Developer B สั่ง `git pull` แล้วเห็นแค่ pointer เปลี่ยน แต่ลืมรัน `git submodule update`
- Developer B จึง build/run โปรเจกต์ด้วย**โค้ดเก่าของ submodule** โดยไม่รู้ตัว ทำให้เจอบั๊กที่ "แก้ไปแล้วแต่ยังไม่หาย" หรือฟีเจอร์ใหม่ที่ "ควรมีแต่ไม่มี"

**วิธีป้องกันที่ทีมมืออาชีพนิยมใช้:**

1. **ตั้งค่า global config `submodule.recurse true`** ตามที่กล่าวไปใน Step 434 เพื่อให้คำสั่งหลายตัวมีพฤติกรรม recursive อัตโนมัติมากขึ้น
2. **ใช้ Git hook** เช่น `post-checkout` หรือ `post-merge` เพื่อรัน `git submodule update --init --recursive` อัตโนมัติทุกครั้งหลัง checkout/pull
3. **เขียนสถานะ submodule ตรวจสอบไว้ใน CI pipeline** — ถ้า commit ของ submodule ที่ checkout ใน CI ไม่ตรงกับที่ parent repo คาดหวัง ให้ build fail ทันที เพื่อจับปัญหาตั้งแต่เนิ่น ๆ
4. **สื่อสารในทีมชัดเจน** ว่าเมื่อไหร่ต้องรัน submodule update — เช่น ใส่ไว้ใน README หรือ CONTRIBUTING.md ของโปรเจกต์อย่างเด่นชัด

ตัวอย่าง git hook แบบง่าย (`.git/hooks/post-merge`):

```bash
#!/bin/sh
git submodule update --init --recursive
```

(ต้องให้สิทธิ์ execute ด้วย `chmod +x .git/hooks/post-merge` — และจำไว้ว่า hook ใน `.git/hooks/` ไม่ถูก commit เข้า repo โดย default ต้องมีกลไกแจกจ่ายให้ทีมแยกต่างหาก เช่นเก็บไว้ในโฟลเดอร์ `scripts/hooks/` แล้วให้ทุกคน symlink หรือ copy เข้า `.git/hooks/` ตอน setup โปรเจกต์)

### ปัญหาที่ 3: Merge Conflict ที่ตัว Gitlink Entry

เมื่อสอง branch แก้ไข pointer ของ submodule ตัวเดียวกันไปคนละ commit แล้วพยายาม merge กัน Git จะรายงาน conflict ที่ path ของ submodule นั้น:

```
CONFLICT (submodule): Merge conflict in libs/shared-utils
```

การแก้ conflict แบบนี้ **ไม่เหมือนการแก้ conflict ในไฟล์ข้อความทั่วไป** เพราะไม่มี `<<<<<<<` `=======` `>>>>>>>` ให้แก้ในไฟล์ — สิ่งที่ต้องทำคือตัดสินใจว่าจะให้ submodule ชี้ไปที่ commit ไหน แล้ว `checkout` submodule ไปที่ commit นั้นด้วยมือ จากนั้น `add` เพื่อ resolve:

```bash
cd libs/shared-utils
git log --oneline -5              # ดูว่า commit ทั้งสองฝั่งคืออะไร ต่างกันตรงไหน
git checkout <commit-hash-ที่ต้องการ>
cd ../..
git add libs/shared-utils
git commit
```

### ปัญหาที่ 4: Clone ช้าเพราะ Submodule ซ้อนกันหลายชั้นหรือมีขนาดใหญ่

ถ้าโปรเจกต์มี submodule จำนวนมาก หรือ submodule แต่ละตัวมีขนาดใหญ่ (เช่นมี binary asset หรือ history ยาวมาก) การ `clone --recurse-submodules` อาจใช้เวลานานมาก และดึงข้อมูลที่ไม่จำเป็นทั้งหมดมา (เพราะ default จะ clone ทั้ง history ของ submodule มาด้วย)

วิธีบรรเทาปัญหานี้คือใช้ **shallow clone สำหรับ submodule**:

```bash
git clone --recurse-submodules --shallow-submodules https://github.com/company/my-project.git
```

flag `--shallow-submodules` จะสั่งให้ clone submodule แบบ depth 1 (เอาแค่ commit ล่าสุด ไม่เอา history เต็ม) ซึ่งช่วยลดเวลาและพื้นที่ดิสก์ได้มากในโปรเจกต์ที่มี submodule จำนวนมาก

### สรุปปัญหาของ Submodule

โดยรวมแล้ว Submodule มีชื่อเสียงว่า "ใช้ยาก" เพราะ:
1. ต้องมี mental model ที่ชัดเจนว่ามันคือ **pointer ไป commit** ไม่ใช่การ copy โค้ด
2. ต้องมี**ขั้นตอนเพิ่มเติมเสมอ**ใน workflow ปกติ (init, update, remote update) ที่ Git ไม่ได้ทำให้อัตโนมัติในหลายจุด
3. Error message และพฤติกรรมของมัน (โดยเฉพาะ detached HEAD) ทำให้มือใหม่สับสนง่าย

นี่คือเหตุผลที่ Git ชุมชนพัฒนาทางเลือกอีกทางหนึ่งขึ้นมาคือ **Git Subtree** ซึ่งเราจะเรียนรู้ต่อไปใน Step ถัดไป

---

## Step 437: Git Subtree คืออะไร ต่างจาก Submodule อย่างไร

### แนวคิดหลักของ Subtree

**Git Subtree** เป็นอีกวิธีหนึ่งในการรวม repo อื่นเข้ามาในโปรเจกต์ แต่ใช้แนวคิดที่**ตรงข้ามกับ Submodule โดยสิ้นเชิง**

> **Subtree นำโค้ดจาก repo อื่นมา "รวมเป็นเนื้อเดียวกัน" กับ parent repo จริง ๆ (merge เข้า history) ไม่ใช่แค่เก็บ pointer ไว้เฉย ๆ**

พูดง่าย ๆ Subtree คือการ **copy history และไฟล์ทั้งหมดของ repo ต้นทางเข้ามาผสมรวมเป็นส่วนหนึ่งของ commit history ของ parent repo โดยตรง** ผลลัพธ์คือทุกไฟล์ของ subtree จะกลายเป็นไฟล์ปกติธรรมดาใน parent repo ทันที ไม่มี gitlink ไม่มี `.gitmodules` ไม่มีอะไรพิเศษเลย

### เปรียบเทียบโครงสร้างไฟล์: Subtree vs Submodule

หลังจากเพิ่ม library เดียวกันด้วย Subtree แทนที่จะเป็น Submodule:

```
my-project/                    ← Parent repo
├── .git/
├── src/
│   └── main.js
├── libs/
│   └── shared-utils/          ← Subtree (ไฟล์จริง ไม่ใช่ pointer!)
│       ├── utils.js             (ไฟล์ปกติ อยู่ใน object database ของ parent repo เลย)
│       └── README.md
└── README.md
```

สังเกตว่าไม่มีไฟล์ `.gitmodules` เลย และภายใน `libs/shared-utils/` ก็ไม่มี `.git` ซ่อนอยู่ — มันเป็นแค่โฟลเดอร์ธรรมดาที่มีไฟล์ปกติอยู่ข้างใน เหมือนกับว่าคุณเขียนไฟล์เหล่านี้ขึ้นมาเองในโปรเจกต์ตั้งแต่แรก

### กลไกเบื้องหลัง: Subtree Merge

Subtree ไม่ใช่ feature พิเศษที่ฝังลึกอยู่ใน core ของ Git object model แบบ submodule (ซึ่งมี mode `160000` เป็นของตัวเอง) แต่ Subtree เป็นการใช้ **`git merge` แบบพิเศษที่เรียกว่า "subtree merge strategy"** ผสมกับชุดคำสั่ง `git subtree` (ซึ่งเป็น contrib script ที่ผนวกเข้ามาใน Git ตั้งแต่ Git 1.7.11)

เบื้องหลังเมื่อคุณสั่ง `git subtree add` มันทำสิ่งต่อไปนี้ (แบบย่อ):
1. Fetch history ทั้งหมดจาก repo ต้นทาง (ผ่าน remote ชั่วคราวหรือ URL ตรง ๆ)
2. สร้าง commit merge พิเศษที่รวม history ของ repo ต้นทางเข้ากับ parent repo โดยจัดวางไฟล์ทั้งหมดของ repo ต้นทางไว้ใต้ prefix (path) ที่กำหนด

ผลลัพธ์คือ **history ของ subtree ถูกฝังรวมเข้าไปเป็นส่วนหนึ่งของ history ของ parent repo อย่างสมบูรณ์** ถ้าคุณสั่ง `git log` ที่ parent repo คุณจะเห็น commit ของ subtree ปรากฏปนอยู่ในนั้นด้วย (ถ้าใช้ `--squash` จะย่อเหลือ commit เดียว ถ้าไม่ใช้ `--squash` จะเอา history เต็มของ subtree มาด้วยทั้งหมด)

### ข้อแตกต่างสำคัญที่ต้องเข้าใจให้ชัด

| ประเด็น | Submodule | Subtree |
|---|---|---|
| สิ่งที่ parent repo เก็บ | Pointer (commit hash) ไปยัง repo อื่น | ไฟล์และ (อาจรวม) history จริงของ repo อื่น ผสานเข้าเป็นเนื้อเดียว |
| ต้องมี `.gitmodules` ไหม | ต้องมี | ไม่ต้องมี |
| คนอื่น clone แล้วได้โค้ดครบทันทีไหม | ไม่ได้ (ต้อง init/update เพิ่ม) | ได้ทันที (เพราะเป็นไฟล์ปกติใน repo) |
| repo ลูกเป็น repository อิสระในเครื่องไหม | ใช่ (มี `.git` ของตัวเอง) | ไม่ใช่ (กลายเป็นไฟล์ปกติ ไม่มี `.git` แยก) |
| การแก้ไขโค้ดของ dependency จากภายใน parent repo | ทำได้ แต่ต้องระวังเรื่อง detached HEAD | ทำได้ตามปกติเหมือนไฟล์อื่นในโปรเจกต์เลย |
| ต้องติดตั้งเครื่องมือเพิ่มไหม | เป็นฟีเจอร์หลักของ Git core | เป็น script เสริมที่มากับ Git (ตั้งแต่ 1.7.11) แต่แนวคิด "subtree merge" มีมาก่อนหน้านั้น |

### ทำไมถึงมีสองเครื่องมือที่ทำเป้าหมายคล้ายกันแต่ต่างกันสุดขั้ว

Submodule ถูกออกแบบมาสำหรับกรณีที่ **repo ต้นทางเปลี่ยนแปลงบ่อย ต้องการรักษาความเป็นอิสระของ repo ต้นทางไว้อย่างชัดเจน** และทีมยินดีจ่ายต้นทุนด้าน workflow ที่ซับซ้อนขึ้นเพื่อแลกกับความสะอาดของ history (parent repo ไม่ปนกับ history ของ dependency)

Subtree ถูกออกแบบมาสำหรับกรณีที่ **ต้องการความง่ายสำหรับคนที่ clone/ใช้งานโปรเจกต์** โดยไม่ต้องรู้จักหรือกังวลเรื่อง submodule เลย แลกกับการที่ history ของ parent repo จะมีขนาดใหญ่ขึ้นและปนกับ history ของ dependency

เราจะเจาะรายละเอียดการใช้งาน Subtree จริงใน Step ถัดไป และเปรียบเทียบแบบละเอียดยิบใน Step 439

---

## Step 438: การใช้งาน Subtree (`git subtree add/pull/push`)

Git Subtree มีคำสั่งหลักอยู่ 3 กลุ่มคือ `add`, `pull`, และ `push` มาดูรายละเอียดของแต่ละคำสั่ง

### คำสั่งพื้นฐาน: `git subtree add`

รูปแบบคำสั่ง:

```bash
git subtree add --prefix=<path> <repository> <branch> [--squash]
```

ตัวอย่างจริง:

```bash
git subtree add --prefix=libs/shared-utils https://github.com/company/shared-utils.git main --squash
```

อธิบายแต่ละส่วน:

| ส่วนของคำสั่ง | ความหมาย |
|---|---|
| `--prefix=libs/shared-utils` | path ปลายทางภายใน parent repo ที่จะวางไฟล์ของ subtree นี้ |
| `https://github.com/company/shared-utils.git` | URL ของ repo ต้นทาง (สามารถใช้ชื่อ remote ที่เพิ่มไว้ล่วงหน้าแทนก็ได้) |
| `main` | branch ของ repo ต้นทางที่ต้องการดึงเข้ามา |
| `--squash` | (แนะนำอย่างยิ่ง) ย่อ history ทั้งหมดของ subtree ให้เหลือเป็น **commit เดียว** แทนที่จะเอา commit history ทั้งหมดของ repo ต้นทางเข้ามาปนกับ parent repo |

### ทำไมควรใช้ `--squash` เกือบทุกครั้ง

ถ้าไม่ใส่ `--squash` การ `subtree add` จะดึง**ทุก commit ในประวัติของ repo ต้นทางเข้ามารวมกับ history ของ parent repo ทั้งหมด** ซึ่งถ้า repo ต้นทางมี commit นับพัน จะทำให้ `git log` ของ parent repo ยาวเหยียดและอ่านยากขึ้นมาก รวมถึงขนาดของ `.git` directory จะโตขึ้นอย่างมีนัยสำคัญ

การใช้ `--squash` ทำให้:
- History ของ parent repo สะอาด มีแค่ 1 commit ที่บอกว่า "เพิ่ม subtree นี้เข้ามา"
- ยังคงสามารถ pull อัปเดตในอนาคตได้ตามปกติ (เพราะ subtree เก็บ "จุดอ้างอิง" ที่ว่าครั้งล่าสุดที่ sync กับ repo ต้นทางคือ commit ไหน ไว้ใน commit message ของ merge commit นั้น)

### การใช้งานผ่าน Remote ที่ตั้งชื่อไว้ล่วงหน้า (แนะนำ)

เพื่อความสะดวก มักจะเพิ่ม remote ของ repo ต้นทางไว้ก่อน แล้วอ้างอิงผ่านชื่อ remote แทนการพิมพ์ URL เต็มทุกครั้ง:

```bash
# เพิ่ม remote ของ repo ต้นทาง (ตั้งชื่อเรียกสั้น ๆ)
git remote add -f shared-utils-remote https://github.com/company/shared-utils.git

# เพิ่ม subtree โดยอ้างอิงจากชื่อ remote
git subtree add --prefix=libs/shared-utils shared-utils-remote main --squash
```

flag `-f` ใน `git remote add -f` จะสั่งให้ fetch ข้อมูลจาก remote นั้นทันทีหลังเพิ่ม เพื่อให้พร้อมใช้งานกับคำสั่ง subtree ต่อได้เลย

### การอัปเดต Subtree ไปยังเวอร์ชันใหม่: `git subtree pull`

เมื่อ repo ต้นทางมีการอัปเดต และต้องการดึงการเปลี่ยนแปลงใหม่เข้ามาผสานกับ parent repo:

```bash
git subtree pull --prefix=libs/shared-utils shared-utils-remote main --squash
```

คำสั่งนี้จะ:
1. Fetch commit ใหม่จาก remote/branch ที่ระบุ
2. ทำการ merge (แบบ subtree merge strategy) การเปลี่ยนแปลงเหล่านั้นเข้ากับไฟล์ที่ path `libs/shared-utils` ของ parent repo
3. สร้าง merge commit ใหม่ที่บันทึกการอัปเดตนี้ (พร้อม squash ตาม flag ที่ระบุ)

ถ้ามีการแก้ไขไฟล์ในทั้งสองฝั่ง (คือทั้ง repo ต้นทางและ parent repo แก้ไฟล์เดียวกันในช่วงเวลาที่ผ่านมา) อาจเกิด **merge conflict แบบไฟล์ปกติทั่วไป** ซึ่งแก้ไขได้ด้วยวิธีเดียวกับการ merge conflict ทั่วไปใน Git (แก้ `<<<<<<<` `=======` `>>>>>>>` ในไฟล์ แล้ว `add` + `commit`)

### การส่งการแก้ไขกลับไปยัง Repo ต้นทาง: `git subtree push`

จุดเด่นที่น่าสนใจของ Subtree คือ ถ้าคุณแก้ไขโค้ดของ subtree นั้น**จากภายใน parent repo โดยตรง** (เพราะมันเป็นไฟล์ปกติ ไม่มีอะไรกั้น) คุณสามารถส่งการแก้ไขนั้นกลับไปยัง repo ต้นทางได้ด้วยคำสั่ง:

```bash
git subtree push --prefix=libs/shared-utils shared-utils-remote feature/my-fix
```

คำสั่งนี้จะ:
1. แยก (extract) เฉพาะ commit ที่เกี่ยวข้องกับ path `libs/shared-utils` ออกมาจาก history ของ parent repo
2. Push commit เหล่านั้นไปยัง branch `feature/my-fix` บน remote `shared-utils-remote`

จากนั้นคุณสามารถไปเปิด Pull Request ที่ repo ต้นทางตามปกติ เพื่อขอให้ maintainer ของ repo นั้น merge การแก้ไขของคุณเข้าไปใน main branch ทำให้ dependency ต้นทางดีขึ้นสำหรับทุกคนที่ใช้งานร่วมกัน

**ข้อควรระวัง:** `git subtree push` อาจใช้เวลานานพอสมควรในโปรเจกต์ที่มี history ยาว เพราะมันต้องไล่สแกน history ทั้งหมดของ parent repo เพื่อแยก commit ที่เกี่ยวข้องกับ prefix นั้นออกมา

### สรุปคำสั่งหลักของ Subtree

| คำสั่ง | หน้าที่ |
|---|---|
| `git subtree add --prefix=<path> <repo> <branch> --squash` | เพิ่ม repo ภายนอกเข้ามาเป็น subtree ครั้งแรก |
| `git subtree pull --prefix=<path> <repo> <branch> --squash` | ดึงอัปเดตใหม่จาก repo ต้นทางมา merge เข้า |
| `git subtree push --prefix=<path> <repo> <branch>` | ส่งการแก้ไขที่ทำใน parent repo กลับไปยัง repo ต้นทาง |
| `git subtree split --prefix=<path>` | แยก history ของ path นั้นออกมาเป็น branch ใหม่ (ใช้ประกอบการ push หรือย้าย subtree ไปเป็น repo อิสระในอนาคต) |

---

## Step 439: ตารางเปรียบเทียบ Submodule vs Subtree แบบละเอียด — เลือกใช้เมื่อไหร่

หลังจากเรียนรู้ทั้งสองเครื่องมือแล้ว มาสรุปเปรียบเทียบให้ครบทุกมิติ เพื่อให้ตัดสินใจเลือกใช้ได้อย่างมั่นใจในสถานการณ์จริง

### ตารางเปรียบเทียบแบบละเอียด

| มิติ | Git Submodule | Git Subtree |
|---|---|---|
| **แนวคิดหลัก** | เก็บ pointer (commit hash) ไปยัง repo อื่น | รวมไฟล์และ history ของ repo อื่นเข้าเป็นเนื้อเดียวกับ parent repo |
| **ความซับซ้อนตอนใช้งานครั้งแรก** | ปานกลาง (`git submodule add`) | ปานกลาง (`git subtree add --squash`) |
| **ความซับซ้อนตอน clone/onboarding คนใหม่** | สูง — ต้องจำ `--recurse-submodules` หรือรัน `init`/`update` เพิ่มเสมอ | ต่ำมาก — `git clone` ปกติได้ครบทุกไฟล์ทันที ไม่ต้องรู้ด้วยซ้ำว่ามี subtree |
| **ขนาดของ `.git` directory ของ parent repo** | เล็ก (เก็บแค่ pointer ไม่เก็บโค้ดจริง) | ใหญ่ขึ้น (มีไฟล์และอาจมี history ของ subtree ปนอยู่จริง) |
| **History ของ parent repo สะอาดแค่ไหน** | สะอาดมาก (ไม่ปนกับ history dependency เลย) | ปนกับ history ของ dependency (น้อยลงถ้าใช้ `--squash`) |
| **การแก้ไขโค้ดของ dependency จากใน parent repo** | ทำได้ แต่เสี่ยง detached HEAD ถ้าไม่ระวัง | ทำได้ตามปกติเหมือนไฟล์อื่น ๆ ไม่มีข้อจำกัดพิเศษ |
| **การส่งการแก้ไขกลับไป repo ต้นทาง** | ตรงไปตรงมา (เข้าไปทำงานใน submodule ปกติ, push ตามปกติ) | ต้องใช้ `git subtree push` ซึ่งอาจช้าในโปรเจกต์ใหญ่ |
| **การ pin เวอร์ชันที่แน่นอน** | ทำได้แม่นยำระดับ commit เดียว ชัดเจนมาก | ทำได้ (ผ่านการเลือก branch/tag ตอน `pull`) แต่ไม่มี "pointer" ที่ดูง่ายเท่า |
| **ต้องติดตั้งเครื่องมือเพิ่มไหม** | ไม่ต้อง เป็น core feature ของ Git | ไม่ต้อง (มีมาให้ใน Git ตั้งแต่ 1.7.11) แต่บาง distro/เวอร์ชันเก่าอาจต้องติดตั้ง `git-subtree` แยก |
| **รองรับ submodule/subtree ซ้อนกันหลายชั้น** | รองรับ (`--recursive`) แต่ยิ่งซับซ้อนยิ่งจัดการยาก | ทำได้ แต่ไม่นิยมทำซ้อนกันหลายชั้นเพราะจะยิ่งงงเรื่อง history |
| **เหมาะกับกรณีที่ dependency เปลี่ยนแปลงบ่อยและอยากเห็น diff ชัดเจน** | เหมาะมาก (diff ของ parent repo จะเห็นแค่ pointer เปลี่ยน) | ไม่เหมาะเท่า (diff จะเห็นทุกไฟล์ที่เปลี่ยนของ dependency ปนอยู่ใน log) |
| **เหมาะกับกรณีที่ต้องการให้คนภายนอก/นักพัฒนาใหม่ onboard ง่ายที่สุด** | ไม่เหมาะ (มีขั้นตอนเพิ่มที่ลืมง่าย) | เหมาะมาก (clone แล้วใช้งานได้เลย) |
| **CI/CD ที่ไม่รองรับ recurse-submodules โดย default** | ต้องตั้งค่า CI เพิ่มเพื่อ checkout submodule ให้ถูกต้อง | ไม่ต้องตั้งค่าอะไรเพิ่ม เพราะเป็นไฟล์ปกติอยู่แล้ว |
| **ความเป็นอิสระของ repo ต้นทาง** | สูงมาก — repo ต้นทางไม่รู้จัก/ไม่เกี่ยวข้องกับ parent repo เลย | สูงเช่นกัน — repo ต้นทางไม่ถูกกระทบ แต่ parent repo "ยืม" ประวัติของมันมาปนบางส่วน |
| **ความนิยมใช้ในวงการ** | นิยมมากในโปรเจกต์ open source ขนาดใหญ่ (เช่น เกม, firmware, library aggregation) | นิยมในทีมที่ให้ความสำคัญกับความง่ายของ onboarding และไม่อยากสอนทีมเรื่อง submodule |

### แนวทางการเลือกใช้ในสถานการณ์จริง

**เลือกใช้ Submodule เมื่อ:**

1. Dependency นั้นมีการเปลี่ยนแปลง**บ่อยและเป็นอิสระ**จาก parent repo มาก และคุณต้องการ pin เวอร์ชันที่แน่นอนไว้อย่างชัดเจน ตรวจสอบง่ายว่า "ตอนนี้เราใช้ dependency เวอร์ชันไหนอยู่"
2. ทีมของคุณมีความรู้ Git ในระดับที่คุ้นเคยกับ workflow ที่ซับซ้อนขึ้น และยินดี set up tooling/hook เพื่อลดความเสี่ยงเรื่องคนลืม update
3. คุณต้องการให้ history ของ parent repo **สะอาดและไม่ปนกับ** history ของ dependency เพื่อให้ `git log`, `git blame` อ่านง่าย
4. Dependency นั้นมีขนาดใหญ่มาก (เช่น ทั้ง engine เกม หรือ framework ขนาดใหญ่) การ copy history เข้ามาทั้งหมดแบบ subtree จะทำให้ repo หลักบวมเกินไป

**เลือกใช้ Subtree เมื่อ:**

1. ให้ความสำคัญกับ**ความง่ายในการ onboard คนใหม่**เป็นอันดับหนึ่ง — อยากให้ `git clone` ครั้งเดียวได้ครบทุกอย่าง ไม่ต้องสอนขั้นตอนพิเศษเพิ่ม
2. Dependency นั้นมีขนาดไม่ใหญ่มาก และไม่ค่อยเปลี่ยนแปลงบ่อย (เช่น pull อัปเดตปีละ 2-3 ครั้งก็พอ)
3. ทีมมักจะต้อง**แก้ไขโค้ดของ dependency นั้นโดยตรง**บ่อย ๆ ควบคู่ไปกับการแก้โค้ดหลัก และอยากให้การแก้ไขนั้นรู้สึกเป็นธรรมชาติเหมือนแก้ไฟล์ปกติในโปรเจกต์ ไม่ต้องเข้าไปอยู่ใน submodule context แยก
4. CI/CD ของทีมมีข้อจำกัดหรือยังไม่รองรับ submodule checkout ได้ดี (บาง CI provider รุ่นเก่าหรือ config พื้นฐานอาจตั้งค่ายากกว่า)

**ทางเลือกที่สาม (ที่ควรพิจารณาเสมอก่อนใช้ทั้งสองอย่าง):**

ในหลายกรณี ก่อนจะไปถึง submodule หรือ subtree ควรถามตัวเองก่อนว่า **dependency นี้เผยแพร่เป็น package ผ่าน package manager ได้ไหม** (เช่น npm, pip, Maven, NuGet, Go modules) เพราะถ้าทำได้ นั่นคือทางเลือกที่ดีกว่าทั้งสองแบบในกรณีส่วนใหญ่ — เพราะ package manager มีระบบ versioning, dependency resolution, และ lockfile ที่ออกแบบมาเพื่อจัดการปัญหานี้โดยเฉพาะอยู่แล้ว Submodule และ Subtree เหมาะกับกรณีที่ dependency นั้น**เป็น source code ที่ต้อง build ร่วมกับโปรเจกต์หลักโดยตรง** ไม่ได้ถูกแพ็กเป็น package สำเร็จรูป หรือกรณีที่ package manager ของภาษานั้นไม่รองรับการ pin แบบ Git commit ได้ดีพอ

---

## Step 440: แบบฝึกหัด — สร้างโปรเจกต์ที่ใช้ Submodule จริง และทดลอง Subtree เพื่อเปรียบเทียบ

ถึงเวลาลงมือทำจริง แบบฝึกหัดนี้จะให้คุณสร้าง repo จำลอง 2 ตัว (repo หลักและ repo library เล็ก ๆ) แล้วทดลองทั้ง Submodule และ Subtree เพื่อเห็นความแตกต่างด้วยตัวเองแบบจับต้องได้

### เตรียมสภาพแวดล้อม

สร้างโฟลเดอร์สำหรับฝึกฝนตาม Part 01:

```bash
mkdir -p ~/git-course/part-44-submodule-subtree
cd ~/git-course/part-44-submodule-subtree
```

### ส่วนที่ 1: สร้าง "Library Repo" จำลอง (จะทำหน้าที่เป็น dependency)

```bash
mkdir mini-lib
cd mini-lib
git init -b main
```

สร้างไฟล์ library เล็ก ๆ:

```bash
cat > math_utils.py << 'EOF'
def add(a, b):
    return a + b

def multiply(a, b):
    return a * b
EOF

cat > README.md << 'EOF'
# mini-lib

ไลบรารีจำลองขนาดเล็กสำหรับฝึก Submodule/Subtree
EOF

git add math_utils.py README.md
git commit -m "Initial version: add and multiply functions"
```

จำลองให้ library นี้มีการอัปเดตในภายหลัง (จะใช้ทดสอบตอนอัปเดต submodule/subtree):

```bash
cat >> math_utils.py << 'EOF'

def subtract(a, b):
    return a - b
EOF

git add math_utils.py
git commit -m "Add subtract function"
```

ตอนนี้ `mini-lib` มี 2 commit แล้ว ให้จำ path เต็มของโฟลเดอร์นี้ไว้ (ใช้แทน URL ของ remote repository ในแบบฝึกหัดนี้ เพื่อไม่ต้องพึ่งอินเทอร์เน็ต):

```bash
cd ..
pwd    # จดค่านี้ไว้ เช่น /home/user/git-course/part-44-submodule-subtree
```

### ส่วนที่ 2: สร้าง "Main Project" และเพิ่ม Submodule

```bash
mkdir main-project-submodule
cd main-project-submodule
git init -b main

echo "# Main Project (Submodule Version)" > README.md
git add README.md
git commit -m "Initial commit"
```

เพิ่ม `mini-lib` เป็น submodule (ใช้ path บนเครื่องแทน URL เพื่อความสะดวกในการฝึก):

```bash
git submodule add ../mini-lib libs/mini-lib
git status
```

สังเกตผลลัพธ์ว่ามีไฟล์ `.gitmodules` และ `libs/mini-lib` ถูก stage ไว้ ให้เปิดดูเนื้อหาไฟล์ `.gitmodules`:

```bash
cat .gitmodules
```

commit การเพิ่ม submodule:

```bash
git commit -m "Add mini-lib as submodule"
```

ตรวจสอบว่า submodule มีสถานะ detached HEAD ตามที่เรียนไปใน Step 436:

```bash
cd libs/mini-lib
git status
cd ../..
```

### ส่วนที่ 3: จำลองการ Clone Repo ที่มี Submodule

ลอง clone repo `main-project-submodule` ไปยังตำแหน่งใหม่แบบไม่ใส่ flag ก่อน เพื่อดูปัญหาที่เคยเรียนใน Step 434:

```bash
cd ..
git clone main-project-submodule main-project-submodule-clone1
cd main-project-submodule-clone1
ls libs/mini-lib/     # จะพบว่าโฟลเดอร์นี้ว่างเปล่า!
cd ..
```

แก้ปัญหาด้วยการ init/update:

```bash
cd main-project-submodule-clone1
git submodule update --init --recursive
ls libs/mini-lib/     # ตอนนี้ควรมีไฟล์ครบแล้ว
cd ..
```

ลอง clone อีกครั้งด้วย `--recurse-submodules` เพื่อเทียบว่าสะดวกกว่า:

```bash
git clone --recurse-submodules main-project-submodule main-project-submodule-clone2
cd main-project-submodule-clone2
ls libs/mini-lib/     # ควรมีไฟล์ครบตั้งแต่ clone เสร็จ ไม่ต้องรันคำสั่งเพิ่ม
cd ..
```

### ส่วนที่ 4: ทดลองอัปเดต Submodule ไปเวอร์ชันใหม่

กลับไปที่ `mini-lib` แล้วเพิ่มฟีเจอร์ใหม่ (จำลองว่าทีม library อัปเดตของเขา):

```bash
cd mini-lib
cat >> math_utils.py << 'EOF'

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
EOF
git add math_utils.py
git commit -m "Add divide function"
cd ..
```

กลับไปที่ `main-project-submodule` เพื่ออัปเดต submodule ให้ตามทัน:

```bash
cd main-project-submodule
git submodule update --remote libs/mini-lib
git status
git diff libs/mini-lib
git add libs/mini-lib
git commit -m "Update mini-lib submodule to include divide function"
cd ..
```

ตรวจสอบว่าไฟล์ใน `libs/mini-lib/math_utils.py` มีฟังก์ชัน `divide` แล้ว:

```bash
cat main-project-submodule/libs/mini-lib/math_utils.py
```

### ส่วนที่ 5: สร้าง "Main Project" อีกตัวและทดลองใช้ Subtree แทน

ตอนนี้มาสร้างโปรเจกต์คู่ขนานที่ใช้ Subtree แทน เพื่อเปรียบเทียบประสบการณ์การใช้งาน:

```bash
mkdir main-project-subtree
cd main-project-subtree
git init -b main

echo "# Main Project (Subtree Version)" > README.md
git add README.md
git commit -m "Initial commit"
```

เพิ่ม remote ของ `mini-lib` แล้วเพิ่มเป็น subtree:

```bash
git remote add -f mini-lib-remote ../mini-lib
git subtree add --prefix=libs/mini-lib mini-lib-remote main --squash
```

สังเกตความแตกต่างทันที:

```bash
ls libs/mini-lib/          # มีไฟล์ครบ ไม่มี .git ซ่อนอยู่ข้างใน
cat libs/mini-lib/math_utils.py  # ควรมีฟังก์ชัน divide อยู่แล้ว เพราะดึงมาจาก commit ล่าสุด
ls -la                       # ไม่มีไฟล์ .gitmodules เลย
git log --oneline            # ดู history ว่ามีกี่ commit (ถ้าใช้ --squash จะเห็นแค่ 1 commit เพิ่มเข้ามา)
```

ลองแก้ไขไฟล์ของ subtree โดยตรงจากใน parent repo (สิ่งที่ทำได้ยากกว่าใน submodule):

```bash
cat >> libs/mini-lib/math_utils.py << 'EOF'

def power(a, b):
    return a ** b
EOF
git add libs/mini-lib/math_utils.py
git commit -m "Add power function directly in main project"
```

สังเกตว่าคำสั่งนี้เป็น `git add` และ `git commit` **แบบธรรมดาที่สุด** เหมือนแก้ไฟล์อื่น ๆ ในโปรเจกต์ ไม่มีขั้นตอนพิเศษใด ๆ เลย ต่างจาก submodule ที่ต้องเข้าไปทำงานในบริบทของ submodule ก่อน

### ส่วนที่ 6: ทดลอง Clone Repo ที่ใช้ Subtree

```bash
cd ..
git clone main-project-subtree main-project-subtree-clone1
cd main-project-subtree-clone1
ls libs/mini-lib/    # ควรมีไฟล์ครบทันที ไม่ต้องรันคำสั่งเพิ่มเติมใด ๆ เลย
cat libs/mini-lib/math_utils.py
cd ..
```

### ส่วนที่ 7: สรุปสิ่งที่สังเกตได้จากแบบฝึกหัด (เขียนคำตอบของตัวเอง)

หลังทำแบบฝึกหัดครบแล้ว ให้ลองตอบคำถามต่อไปนี้ด้วยตัวเอง (เทียบกับสิ่งที่เรียนใน Step 439):

1. ระหว่าง clone repo submodule กับ subtree แบบไหนใช้คำสั่งน้อยกว่าเพื่อให้ได้โค้ดครบ
2. เมื่อแก้ไขไฟล์ของ dependency โดยตรง แบบไหนรู้สึกเป็นธรรมชาติกว่า (submodule ต้องเข้าไปอยู่ context ของ submodule ก่อน vs subtree แก้ได้ตรง ๆ)
3. ลองรัน `git log --oneline` เทียบระหว่าง `main-project-submodule` กับ `main-project-subtree` — repo ไหนมี history ที่กระชับกว่า
4. ลองพิจารณา: ถ้า `mini-lib` มีขนาดใหญ่มาก (เช่น 500MB) จะเลือกใช้แบบไหน เพราะอะไร

### (ทางเลือกเพิ่มเติม) ทดลอง `git subtree push`

ถ้าต้องการฝึกครบวงจร ลองสร้างการเปลี่ยนแปลงใน subtree แล้ว push กลับไปยัง `mini-lib`:

```bash
cd main-project-subtree
git subtree push --prefix=libs/mini-lib mini-lib-remote feature/power-function
cd ../mini-lib
git log --oneline --all    # ควรเห็น branch feature/power-function ปรากฏขึ้นมา พร้อม commit "Add power function..."
git branch -a
cd ..
```

### Checklist หลังทำแบบฝึกหัด

ก่อนไปต่อ Part 45 ให้ตรวจสอบว่าคุณ:

- [ ] เข้าใจว่า Submodule เก็บแค่ pointer (commit hash) ไม่ใช่โค้ดจริง
- [ ] เพิ่ม submodule ได้ด้วย `git submodule add <url> <path>` และเข้าใจโครงสร้างไฟล์ `.gitmodules`
- [ ] Clone repo ที่มี submodule ได้ทั้งสองวิธี (`--recurse-submodules` และ `init` + `update` แยก)
- [ ] อัปเดต submodule ไปยัง commit ล่าสุดของ remote ได้ด้วย `git submodule update --remote`
- [ ] เข้าใจปัญหา detached HEAD ภายใน submodule และรู้วิธีป้องกัน (checkout branch ก่อนแก้ไข)
- [ ] เข้าใจว่า Subtree รวมไฟล์ (และอาจรวม history) เข้าเป็นเนื้อเดียวกับ parent repo จริง ๆ
- [ ] เพิ่ม subtree ได้ด้วย `git subtree add --prefix=<path> <repo> <branch> --squash`
- [ ] อัปเดตและส่งการแก้ไขกลับได้ด้วย `git subtree pull` และ `git subtree push`
- [ ] อธิบายความแตกต่างระหว่าง Submodule กับ Subtree ได้อย่างน้อย 5 มิติ และรู้ว่าจะเลือกใช้แบบไหนในสถานการณ์ต่าง ๆ
- [ ] ทำแบบฝึกหัดสร้างโปรเจกต์จำลองทั้งสองแบบและเปรียบเทียบผลลัพธ์ด้วยตัวเองสำเร็จ

---

## สรุป Part 44

ใน Part นี้เราได้เรียนรู้ว่า:

1. ปัญหาที่ Submodule/Subtree แก้คือการที่โปรเจกต์ต้องพึ่งพา repo อื่นเป็น dependency แต่ยังต้อง track เวอร์ชันของมันอย่างแม่นยำ ซึ่ง copy-paste โค้ดหรือรวมเป็น monorepo ธรรมดาไม่สามารถแก้ปัญหานี้ได้ดีพอ
2. **Git Submodule** เก็บแค่ pointer (commit hash แบบ gitlink mode `160000`) ไปยัง repo อื่น ผ่านไฟล์ `.gitmodules` ที่ระบุ path, url, และ branch ของแต่ละ submodule
3. การเพิ่ม submodule ใช้ `git submodule add <url> <path>` ส่วนการ clone repo ที่มี submodule ต้องใช้ `--recurse-submodules` หรือรัน `init`/`update` แยกทีหลัง
4. การอัปเดต submodule ไปเวอร์ชันใหม่ใช้ `git submodule update --remote` แต่ต้อง `add` และ `commit` pointer ใหม่เข้า parent repo ด้วยตัวเองเสมอ
5. ปัญหาที่พบบ่อยของ submodule คือ detached HEAD ภายใน submodule, ทีมลืมรัน update หลัง pull, และ merge conflict ที่ gitlink entry — ทั้งหมดแก้ได้ด้วยวินัยและ tooling ที่เหมาะสม
6. **Git Subtree** ตรงข้ามกับ submodule โดยสิ้นเชิง คือรวมไฟล์ (และอาจรวม history) ของ repo อื่นเข้าเป็นเนื้อเดียวกับ parent repo จริง ๆ ผ่าน `git subtree add/pull/push --prefix=<path>`
7. การเลือกใช้ Submodule เหมาะกับกรณีที่ต้องการ pin เวอร์ชันแม่นยำและรักษา history ให้สะอาด ส่วน Subtree เหมาะกับกรณีที่ต้องการความง่ายในการ onboard และแก้ไขโค้ด dependency บ่อย ๆ โดยไม่ต้องมี workflow พิเศษ
8. ก่อนเลือกใช้ทั้งสองอย่าง ควรพิจารณาก่อนว่า dependency นั้นเผยแพร่เป็น package ผ่าน package manager ของภาษานั้นได้หรือไม่ ซึ่งมักเป็นทางเลือกที่ดีกว่าในหลายกรณี

**ต่อไป:** [Part 45: โปรเจกต์ทีม: จำลองการทำงานทีม 4-5 คนแบบมืออาชีพ](./part-045-โปรเจกต์ทีม-capstone.md)
