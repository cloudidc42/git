# Part 17: สร้าง Repository บน GitHub และเชื่อมกับเครื่อง local

> **Step ในหลักสูตรนี้:** Step 161–170
> **เฟส:** 3 — ใช้งาน GitHub อย่างมืออาชีพ ทำ Pull Request, Code Review, Open Source
> **เป้าหมายของ Part นี้:** สร้าง repository บน GitHub ได้อย่างถูกต้องตั้งแต่หน้าเว็บ เข้าใจตัวเลือกต่าง ๆ ตอนสร้าง (visibility, README, .gitignore, license) เชื่อม local repository เข้ากับ GitHub ได้ทั้งสองทิศทาง (สร้างบน GitHub ก่อนแล้ว clone และมี local อยู่ก่อนแล้วค่อยเชื่อม) รู้จักการตั้งค่า repository เบื้องต้น และเข้าใจข้อมูล Insights พื้นฐานที่ GitHub มีให้

---

## สารบัญของ Part นี้

- Step 161: สร้าง repo ใหม่บน GitHub ผ่านหน้าเว็บ (public/private, README, .gitignore, license)
- Step 162: เชื่อม local repo ที่มีอยู่แล้วเข้ากับ GitHub repo ใหม่
- Step 163: การ clone repo จาก GitHub มาที่เครื่อง (HTTPS vs SSH)
- Step 164: Repository settings เบื้องต้น (ชื่อ, visibility, transfer, archive, delete)
- Step 165: About section และ Topics สำหรับจัดหมวดหมู่
- Step 166: การ import repo จากที่อื่นเข้า GitHub ด้วย GitHub Importer
- Step 167: Repository templates — สร้าง repo ต้นแบบเพื่อใช้ซ้ำ
- Step 168: การลบ/เก็บถาวร (archive) repo อย่างปลอดภัย
- Step 169: GitHub repo Insights เบื้องต้น
- Step 170: แบบฝึกหัด — สร้าง repo ใหม่ตั้งแต่ web UI จนเชื่อมกับเครื่อง local ครบวงจร

---

## Step 161: สร้าง repo ใหม่บน GitHub ผ่านหน้าเว็บ (public/private, README, .gitignore template, license เลือกยังไง)

ก่อนหน้านี้ใน Part 1–15 เราทำงานกับ Git แบบ local ล้วน ๆ ยังไม่มี repository ไหนเลยที่อยู่บน GitHub ถึงเวลาแล้วที่เราจะเรียนรู้การสร้าง **remote repository** บน GitHub อย่างถูกต้อง เพราะนี่คือจุดเริ่มต้นของการทำงานร่วมกับคนอื่น การสำรองโค้ด และการเปิดโปรเจกต์สู่สาธารณะ

### 161.1 ขั้นตอนสร้าง repository ใหม่บนเว็บ GitHub

1. ล็อกอินเข้า https://github.com ด้วยบัญชีของคุณ
2. คลิกปุ่ม **`+`** มุมขวาบนของหน้าเว็บ แล้วเลือก **New repository** (หรือเข้าตรง ๆ ที่ https://github.com/new)
3. หน้าจอ "Create a new repository" จะแสดงฟิลด์ให้กรอกดังนี้:

| ฟิลด์ | ความหมาย |
|---|---|
| **Owner** | บัญชีหรือ organization ที่จะเป็นเจ้าของ repo |
| **Repository name** | ชื่อ repo (ต้องไม่ซ้ำกับ repo อื่นภายใต้ owner เดียวกัน) |
| **Description (optional)** | คำอธิบายสั้น ๆ ว่า repo นี้ทำอะไร แสดงใต้ชื่อ repo |
| **Public / Private** | ระดับการมองเห็น |
| **Add a README file** | สร้างไฟล์ `README.md` ให้อัตโนมัติ |
| **Add .gitignore** | เลือก template ไฟล์ `.gitignore` ตามภาษา/เฟรมเวิร์ก |
| **Choose a license** | เลือกสัญญาอนุญาตให้ใช้โค้ด |

### 161.2 การตั้งชื่อ repository ที่ดี

กฎการตั้งชื่อของ GitHub:

- ใช้ตัวอักษร (a-z, A-Z), ตัวเลข, ขีดกลาง (`-`), ขีดล่าง (`_`), และจุด (`.`) ได้
- ห้ามมีช่องว่าง (ถ้าอยากแบ่งคำ ใช้ `-` เช่น `my-first-project`)
- ควรใช้ตัวพิมพ์เล็กทั้งหมดเป็นธรรมเนียมปฏิบัติ (ไม่บังคับ แต่เป็นมาตรฐานที่นิยม)
- ชื่อควรสื่อความหมายชัดเจน เช่น `expense-tracker-api` ดีกว่า `project1` หรือ `test123`

> **หมายเหตุ:** ชื่อ repo ไม่จำเป็นต้องซ้ำกับชื่อโฟลเดอร์ local ของคุณ แต่การตั้งชื่อให้ตรงกันจะช่วยลดความสับสนในระยะยาว

### 161.3 Public กับ Private ต่างกันอย่างไร

| หัวข้อ | Public | Private |
|---|---|---|
| ใครมองเห็นได้ | ทุกคนบนอินเทอร์เน็ต มองเห็นและ clone ได้ | เฉพาะคุณและคนที่คุณเชิญเท่านั้น |
| เหมาะกับ | Open Source, portfolio, โปรเจกต์เรียนรู้ | โค้ดของบริษัท, โปรเจกต์ส่วนตัวที่ยังไม่พร้อมเปิด |
| ค้นหาเจอใน Google/GitHub Search | ได้ | ไม่ได้ |
| จำนวน collaborator บนแผนฟรี | ไม่จำกัด (public repo) | ไม่จำกัดเช่นกันในปัจจุบัน (GitHub เปิดให้ private repo ฟรีมี unlimited collaborators แล้ว) |
| GitHub Actions minutes | ฟรีไม่จำกัดสำหรับ public repo | มีโควตาต่อเดือนตามแผน (ฟรี 2,000 นาที/เดือนสำหรับบัญชีฟรี) |

**คำแนะนำ:** ถ้ายังไม่แน่ใจว่าจะเปิดสาธารณะหรือไม่ ให้เริ่มจาก **Private** ไปก่อนเสมอ เพราะสามารถเปลี่ยนเป็น Public ทีหลังได้ง่าย (ดู Step 164) แต่การเปลี่ยนจาก Public เป็น Private หลังจากมีคน fork หรือ clone ไปแล้ว **ไม่สามารถเรียกคืนสำเนาที่มีคนดาวน์โหลดไปแล้วได้**

### 161.4 ควรติ๊ก "Add a README file" ตอนสร้างไหม

ขึ้นอยู่กับสถานการณ์:

- **ถ้าคุณจะสร้าง repo ใหม่บน GitHub แล้วค่อย clone มาทำงาน (empty project)** → ติ๊กได้เลย เพราะทำให้ repo มีไฟล์อย่างน้อย 1 ไฟล์ตั้งแต่แรก (มี branch `main` เกิดขึ้นทันที)
- **ถ้าคุณมี local repository ที่มี commit อยู่แล้ว และกำลังจะเชื่อมเข้ากับ GitHub (Step 162)** → **ห้ามติ๊ก** README, .gitignore หรือ license ใด ๆ เลย เพราะจะทำให้ GitHub repo มีประวัติ commit ของตัวเอง ซึ่งจะชนกับประวัติจาก local repo ของคุณตอน push (ต้องมานั่งแก้ปัญหา "unrelated histories" ทีหลัง)

> กฎง่าย ๆ ที่ควรจำ: **repo ฝั่งไหนมี commit อยู่ก่อนแล้ว ให้สร้างอีกฝั่งแบบ "ว่างเปล่า" เสมอ**

### 161.5 .gitignore template คืออะไร เลือกยังไง

`.gitignore` คือไฟล์ที่บอก Git ว่าไฟล์/โฟลเดอร์ไหนไม่ต้องนำเข้าระบบ version control เช่น ไฟล์ที่ compiler สร้างขึ้นเอง, dependency ที่ดาวน์โหลดมา, ไฟล์ log, ไฟล์ config เฉพาะเครื่อง

ตอนสร้าง repo GitHub มี dropdown ให้เลือก template สำเร็จรูปตามภาษา/เฟรมเวิร์กที่ใช้บ่อย เช่น:

| เลือก template | เหมาะกับโปรเจกต์ |
|---|---|
| `Node` | JavaScript/TypeScript ที่ใช้ npm/yarn (ไม่ commit โฟลเดอร์ `node_modules/`) |
| `Python` | โปรเจกต์ Python (ไม่ commit `__pycache__/`, `.venv/`, `*.pyc`) |
| `Java` | โปรเจกต์ Java (ไม่ commit `*.class`, `target/`) |
| `Go` | โปรเจกต์ Go |
| `VisualStudio` | โปรเจกต์ .NET/C# |
| `Unity` | โปรเจกต์เกมที่ทำด้วย Unity |

ถ้าไม่แน่ใจว่าจะเลือกอันไหน หรือโปรเจกต์ยังไม่มีภาษาชัดเจน สามารถเลือก **None** ไปก่อนแล้วค่อยสร้างไฟล์ `.gitignore` เองทีหลังได้ (เราจะเรียนรายละเอียดเรื่อง `.gitignore` แบบเจาะลึกใน Part ถัดไปของหลักสูตร)

### 161.6 License เลือกยังไงดี

**License (สัญญาอนุญาต)** คือเอกสารทางกฎหมายที่บอกว่าคนอื่นสามารถนำโค้ดของคุณไปใช้/แก้ไข/แจกจ่ายต่อได้ในเงื่อนไขแบบไหน ถ้าไม่ใส่ license เลย ตามกฎหมายลิขสิทธิ์เริ่มต้นแล้ว **คนอื่นไม่มีสิทธิ์นำโค้ดของคุณไปใช้ต่อได้เลยแม้ repo จะเป็น public**

ตัวเลือก license ยอดนิยมที่ GitHub มีให้เลือกตอนสร้าง repo:

| License | สรุปสั้น ๆ | เหมาะกับ |
|---|---|---|
| **MIT License** | เปิดกว้างที่สุด ใครจะเอาไปใช้ แก้ไข ขาย ก็ได้ ขอแค่ใส่เครดิตชื่อผู้เขียนไว้ | โปรเจกต์ Open Source ทั่วไป, library, เครื่องมือ (ได้รับความนิยมสูงสุดในโลก) |
| **Apache License 2.0** | คล้าย MIT แต่มีเงื่อนไขเรื่อง patent (สิทธิบัตร) เพิ่มเติม ป้องกันการฟ้องร้องเรื่องสิทธิบัตร | โปรเจกต์ขนาดใหญ่ที่กังวลเรื่องสิทธิบัตร เช่นโปรเจกต์ขององค์กร |
| **GNU GPLv3** | ถ้านำโค้ดไปใช้ต่อ โปรเจกต์ที่นำไปใช้ต้อง open source ด้วยเช่นกัน (copyleft) | ต้องการบังคับให้ผู้ที่ต่อยอดเปิดซอร์สด้วย |
| **GNU AGPLv3** | เข้มกว่า GPL อีกขั้น ครอบคลุมถึงการใช้งานผ่านเครือข่าย (SaaS) ด้วย | โปรเจกต์ที่กังวลว่าจะถูกเอาไปทำ SaaS โดยไม่เปิดซอร์สกลับ |
| **BSD 2-Clause / 3-Clause** | คล้าย MIT อีกแบบหนึ่ง เปิดกว้างเช่นกัน | โปรเจกต์สาย academic/research บางส่วน |
| **Unlicense** | สละสิทธิ์ทั้งหมด ทุกคนเอาไปใช้ได้อย่างอิสระที่สุด ไม่ต้องใส่เครดิตด้วยซ้ำ | โค้ดตัวอย่าง, snippet ที่อยากให้แจกฟรีแบบไม่มีเงื่อนไขใด ๆ |

**คำแนะนำสำหรับมือใหม่:** ถ้าไม่รู้จะเลือกอะไร ให้เลือก **MIT License** ไปก่อน เพราะเป็นสัญญาอนุญาตที่เข้าใจง่ายที่สุด นิยมมากที่สุด และเปิดกว้างพอที่จะให้คนอื่นเอาไปใช้ต่อยอดได้สะดวก

ถ้า repo เป็น private และไม่มีแผนจะเปิดสาธารณะ จะไม่ใส่ license ก็ได้ (ไม่มีผลอะไรเพราะไม่มีใครเข้าถึงอยู่แล้ว)

### 161.7 สรุปหน้าจอสร้าง repo แบบเป็นขั้นตอน

```
1. Owner: your-username
2. Repository name: my-awesome-project
3. Description: "โปรเจกต์ทดลองสำหรับฝึก Git/GitHub"
4. ● Public   ○ Private
5. [x] Add a README file
6. .gitignore template: Node
7. Choose a license: MIT License
8. คลิกปุ่ม "Create repository"
```

หลังคลิก **Create repository** GitHub จะพาคุณไปที่หน้า repo ใหม่ทันที ซึ่งจะมีไฟล์ `README.md`, `.gitignore`, และ `LICENSE` อยู่แล้ว (ถ้าคุณติ๊กไว้) พร้อมปุ่ม **`<> Code`** สีเขียวสำหรับ clone repo มาที่เครื่อง — เราจะใช้ปุ่มนี้ใน Step 163

---

## Step 162: เชื่อม local repo ที่มีอยู่แล้วเข้ากับ GitHub repo ใหม่ (`git remote add origin`, `git push -u origin main`)

สถานการณ์นี้ต่างจาก Step 161 ตรงที่ **คุณมี local repository ที่ทำงานอยู่แล้ว** (มี commit มาสักพักแล้ว) และตอนนี้ต้องการ "ยก" ประวัติทั้งหมดขึ้นไปเก็บบน GitHub เป็นครั้งแรก

### 162.1 เตรียม local repository ให้พร้อม

สมมติว่าคุณมีโฟลเดอร์โปรเจกต์อยู่แล้ว:

```bash
cd ~/git-course/my-local-project
git status
```

ถ้ายังไม่เคยทำ `git init` มาก่อนเลย ให้เริ่มต้นก่อน:

```bash
git init
git add .
git commit -m "Initial commit"
```

ตรวจสอบชื่อ branch หลักของคุณด้วย:

```bash
git branch
```

ถ้าเห็น `master` แต่ต้องการให้ตรงกับมาตรฐานปัจจุบันของ GitHub (ซึ่งใช้ `main` เป็นชื่อ default) ให้เปลี่ยนชื่อ branch ก่อน:

```bash
git branch -M main
```

> `-M` คือ force rename (เขียนทับได้แม้มี branch ชื่อนั้นอยู่แล้ว) ใช้ตัวพิมพ์ใหญ่เพื่อความชัดเจน

### 162.2 สร้าง repo ว่างเปล่าบน GitHub

กลับไปทำตาม Step 161 แต่คราวนี้ **ห้ามติ๊ก** "Add a README file", "Add .gitignore" และ "Choose a license" ใด ๆ ทั้งสิ้น — ปล่อยให้ repo บน GitHub เป็น repo ที่ว่างเปล่า 100% ไม่มี commit แม้แต่ตัวเดียว

หลังสร้างเสร็จ GitHub จะแสดงหน้า **Quick setup** ที่มีคำสั่งให้คัดลอกไปใช้ทันที ภายใต้หัวข้อ **"…or push an existing repository from the command line"**

### 162.3 คำสั่งเชื่อมต่อ: `git remote add origin`

```bash
git remote add origin https://github.com/your-username/my-awesome-project.git
```

อธิบายทีละส่วน:

- `git remote add` — คำสั่งเพิ่ม remote ใหม่เข้าไปใน repository
- `origin` — **ชื่อเรียก (alias)** ของ remote นี้ เป็นชื่อที่ใช้กันเป็นธรรมเนียมมาตรฐานสำหรับ remote หลักตัวแรก (ตั้งชื่ออื่นก็ได้ แต่ `origin` คือ convention ที่ทุกคนเข้าใจตรงกัน)
- URL ต่อท้าย — ที่อยู่ของ repository บน GitHub

ตรวจสอบว่าเชื่อมสำเร็จหรือยัง:

```bash
git remote -v
```

ผลลัพธ์ที่ควรเห็น:

```
origin  https://github.com/your-username/my-awesome-project.git (fetch)
origin  https://github.com/your-username/my-awesome-project.git (push)
```

### 162.4 คำสั่ง push ครั้งแรก: `git push -u origin main`

```bash
git push -u origin main
```

อธิบายทีละส่วน:

- `git push` — ส่ง commit จากเครื่อง local ขึ้นไปยัง remote
- `-u` (ย่อจาก `--set-upstream`) — ตั้งค่าให้ branch `main` ในเครื่อง **ผูก (track)** กับ `origin/main` โดยอัตโนมัติ ผลคือครั้งต่อ ๆ ไปคุณสามารถพิมพ์แค่ `git push` หรือ `git pull` เฉย ๆ โดยไม่ต้องระบุ `origin main` ซ้ำอีก
- `origin` — remote ปลายทาง
- `main` — branch ที่จะ push ขึ้นไป

ถ้าเป็นการเชื่อมด้วย HTTPS ระบบอาจถาม username/password หรือเปิดหน้าต่างให้ยืนยันตัวตนผ่านเบราว์เซอร์ (GitHub ไม่รองรับ password ธรรมดาผ่าน command line แล้วตั้งแต่ปี 2021 ต้องใช้ **Personal Access Token** หรือ **Git Credential Manager** แทน — รายละเอียดเรื่องนี้จะอธิบายลึกใน Part 18 เรื่อง SSH Key และความปลอดภัย)

### 162.5 ผลลัพธ์ที่ควรเห็นหลัง push สำเร็จ

```
Enumerating objects: 8, done.
Counting objects: 100% (8/8), done.
Delta compression using up to 8 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (8/8), 1.02 KiB | 1.02 MiB/s, done.
Total 8 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/your-username/my-awesome-project.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

รีเฟรชหน้า GitHub repo ในเบราว์เซอร์ — คุณจะเห็นไฟล์และ commit ทั้งหมดจากเครื่อง local ปรากฏบนเว็บทันที

### 162.6 ถ้าเจอ error "failed to push some refs" หรือ "Updates were rejected"

ปัญหานี้เกิดขึ้นเมื่อ remote repository มี commit อยู่ก่อนแล้ว (เช่น ดันไปติ๊ก "Add a README" ตอนสร้างโดยไม่ได้ตั้งใจ) ทำให้ประวัติของสองฝั่งไม่ตรงกัน วิธีแก้ที่ปลอดภัยที่สุดสำหรับกรณีนี้คือ:

```bash
git pull origin main --allow-unrelated-histories
# แก้ conflict ถ้ามี แล้ว commit
git push -u origin main
```

หรือถ้ามั่นใจว่า remote repo นั้นยังไม่มีอะไรสำคัญ (แค่มี README ที่สร้างอัตโนมัติ) จะลบ repo แล้วสร้างใหม่แบบว่างเปล่าตาม Step 162.2 ก็ง่ายกว่า

### 162.7 แผนภาพสรุปความสัมพันธ์

```
เครื่อง Local                          GitHub (Remote)
┌─────────────────┐                   ┌─────────────────┐
│ .git/            │   remote add     │                  │
│ (มี commit อยู่แล้ว) │ ───────────────▶ │  origin          │
│                  │                   │                  │
│ branch: main     │   git push -u     │  (ว่างเปล่า)      │
│                  │   origin main     │                  │
└─────────────────┘ ─────────────────▶ └─────────────────┘
                                        กลายเป็นมี main
                                        พร้อม commit ทั้งหมด
```

---

## Step 163: การ clone repo จาก GitHub มาที่เครื่อง local (เทียบ HTTPS URL vs SSH URL)

นี่คือสถานการณ์กลับด้านกับ Step 162 — คุณมี repository อยู่บน GitHub แล้ว (อาจเป็น repo ที่คุณสร้างเองตาม Step 161 หรือเป็น repo ของคนอื่นที่คุณต้องการนำมาทำงานต่อ) และต้องการ **คัดลอกทั้ง repository พร้อมประวัติทั้งหมด** มาไว้ที่เครื่อง

### 163.1 คำสั่ง `git clone` พื้นฐาน

```bash
git clone <URL>
```

ที่หน้า repo บน GitHub คลิกปุ่มสีเขียว **`<> Code`** จะเห็น URL สองแบบให้เลือกใช้ (แท็บ HTTPS และแท็บ SSH)

### 163.2 HTTPS URL

```
https://github.com/your-username/my-awesome-project.git
```

```bash
git clone https://github.com/your-username/my-awesome-project.git
```

**ลักษณะของ HTTPS:**

- ใช้งานได้ทันทีแทบทุกเครือข่าย เพราะ HTTPS (พอร์ต 443) มักไม่ถูกไฟร์วอลล์บล็อก (ต่างจาก SSH ที่บางองค์กรบล็อกพอร์ต 22)
- ตอน push ครั้งแรกจะต้อง authenticate ด้วย **Personal Access Token (PAT)** หรือผ่าน Git Credential Manager/เบราว์เซอร์ (ไม่ใช้รหัสผ่าน GitHub ตรง ๆ อีกต่อไป)
- หลัง authenticate ครั้งแรก เครื่องมือ credential helper (เช่น Git Credential Manager, macOS Keychain, GNOME Keyring) มักจะจำ token ไว้ให้อัตโนมัติ ไม่ต้องใส่ซ้ำทุกครั้ง

### 163.3 SSH URL

```
git@github.com:your-username/my-awesome-project.git
```

```bash
git clone git@github.com:your-username/my-awesome-project.git
```

**ลักษณะของ SSH:**

- ต้องตั้งค่า **SSH key pair** (public/private key) และเพิ่ม public key เข้าบัญชี GitHub ไว้ล่วงหน้า ถึงจะใช้งานได้ (เราจะเรียนรายละเอียดขั้นตอนเต็ม ๆ ใน **Part 18: SSH Key และการตั้งค่าความปลอดภัย**)
- หลังตั้งค่าเสร็จครั้งเดียว จะ push/pull ได้โดยไม่ต้องใส่ password หรือ token ซ้ำอีกเลยตลอดไป (จนกว่าจะเปลี่ยนเครื่องหรือ revoke key)
- ปลอดภัยและสะดวกกว่าในระยะยาวสำหรับคนที่ push/pull บ่อย ๆ
- อาจใช้ไม่ได้ในเครือข่ายที่บล็อกพอร์ต 22 (เช่น เครือข่ายองค์กรบางแห่ง หรือ Wi-Fi สาธารณะบางที่)

### 163.4 ตารางเปรียบเทียบ HTTPS vs SSH

| หัวข้อ | HTTPS | SSH |
|---|---|---|
| การตั้งค่าเริ่มต้น | ไม่ต้องตั้งค่าอะไรล่วงหน้า | ต้องสร้างและเพิ่ม SSH key ก่อน |
| Authentication | Personal Access Token / เบราว์เซอร์ | SSH key pair (public/private) |
| ต้องใส่ credential ซ้ำไหม | ขึ้นกับ credential helper (ส่วนใหญ่จำให้) | ไม่ต้องใส่ซ้ำเลยหลังตั้งค่าเสร็จ |
| ผ่านไฟร์วอลล์องค์กร | ผ่านง่ายกว่า (พอร์ต 443) | อาจถูกบล็อก (พอร์ต 22) |
| เหมาะกับ | เครื่องที่ใช้ชั่วคราว, สภาพแวดล้อม CI บางแบบ, ผู้เริ่มต้น | เครื่องส่วนตัวที่ใช้งานประจำ, นักพัฒนาที่ push/pull บ่อย |
| ความปลอดภัยระยะยาว | ดี (ถ้าตั้ง token expiration และ scope ให้เหมาะสม) | ดีมาก (private key ไม่เคยถูกส่งผ่านเครือข่ายเลย) |

**คำแนะนำสำหรับตอนนี้:** ถ้ายังไม่เคยตั้งค่า SSH key ให้ใช้ **HTTPS ไปก่อน** เพื่อไม่ให้ติดขัดการเรียนรู้ Part นี้ แล้วเราจะไปตั้งค่า SSH ให้สมบูรณ์ใน Part 18 จากนั้นค่อยเปลี่ยน remote URL จาก HTTPS เป็น SSH ทีหลังได้ (ด้วยคำสั่ง `git remote set-url` ซึ่งจะสอนใน Part 18 เช่นกัน)

### 163.5 clone ไปไว้ในโฟลเดอร์ที่กำหนดเอง หรือ clone แบบตื้น (shallow)

ระบุชื่อโฟลเดอร์ปลายทางเองได้ โดยใส่ต่อท้าย URL:

```bash
git clone https://github.com/your-username/my-awesome-project.git my-folder-name
```

ถ้า repo มีประวัติยาวมากและต้องการแค่ commit ล่าสุดเพื่อความรวดเร็ว (ไม่ต้องการประวัติทั้งหมด) ใช้ `--depth`:

```bash
git clone --depth 1 https://github.com/your-username/my-awesome-project.git
```

คำสั่งนี้เรียกว่า **shallow clone** จะดึงมาแค่ commit ล่าสุดเท่านั้น เหมาะกับสถานการณ์เช่น CI/CD pipeline ที่ต้องการแค่โค้ดล่าสุดไปรัน build โดยไม่สนใจประวัติ (แต่ **ไม่แนะนำ** สำหรับงานพัฒนาปกติ เพราะจะทำให้คำสั่งที่ต้องใช้ประวัติ เช่น `git log`, `git blame` แสดงข้อมูลไม่ครบ)

### 163.6 หลัง clone เสร็จ ระบบตั้งค่าอะไรให้อัตโนมัติบ้าง

```bash
cd my-awesome-project
git remote -v
```

```
origin  https://github.com/your-username/my-awesome-project.git (fetch)
origin  https://github.com/your-username/my-awesome-project.git (push)
```

จะเห็นว่า `git clone` ทำสิ่งเหล่านี้ให้อัตโนมัติทั้งหมดในคำสั่งเดียว:

1. สร้างโฟลเดอร์และดาวน์โหลดไฟล์ทั้งหมดของ branch default
2. ดาวน์โหลดประวัติ commit ทั้งหมด (`.git` object database)
3. ตั้งชื่อ remote เป็น `origin` โดยอัตโนมัติ
4. สร้าง local branch (เช่น `main`) ที่ track กับ `origin/main` โดยอัตโนมัติ — ต่างจาก Step 162 ที่ต้องตั้งค่าเองด้วย `-u`

---

## Step 164: Repository settings เบื้องต้น (เปลี่ยนชื่อ, visibility public/private, transfer ownership, archive, delete repo)

ทุก repo บน GitHub มีหน้า **Settings** ที่เข้าถึงได้จากแท็บ **⚙️ Settings** บนแถบเมนูของหน้า repo (มองเห็นเฉพาะเจ้าของ repo หรือคนที่มีสิทธิ์ระดับ Admin เท่านั้น)

### 164.1 เปลี่ยนชื่อ repository

ไปที่ **Settings → General** จะเห็นช่อง **Repository name** ที่ด้านบนสุด แก้ชื่อแล้วกด **Rename**

**สิ่งสำคัญที่ต้องรู้:** เมื่อเปลี่ยนชื่อ repo แล้ว:

- GitHub จะตั้งค่า **redirect อัตโนมัติ** จาก URL เก่าไปยัง URL ใหม่ให้ระยะหนึ่ง (ใช้ได้กับการ `git clone`, `git push`, `git pull` และการเปิดหน้าเว็บด้วย) ทำให้คนที่ยังใช้ลิงก์เก่าไม่พังทันที
- อย่างไรก็ตาม **แนะนำให้อัปเดต remote URL ในเครื่อง local ของทุกคนที่เกี่ยวข้อง** ให้ตรงกับชื่อใหม่ เพื่อความชัดเจนในระยะยาว ด้วยคำสั่ง:
  ```bash
  git remote set-url origin https://github.com/your-username/new-repo-name.git
  ```
- ถ้ามี CI/CD, webhook, หรือลิงก์ที่ hardcode ชื่อ repo เก่าไว้ ควรตรวจสอบและแก้ไขด้วย

### 164.2 เปลี่ยน Visibility (Public ↔ Private)

ยังอยู่ที่ **Settings → General** เลื่อนลงไปที่ส่วน **Danger Zone** ด้านล่างสุดของหน้า จะเห็นตัวเลือก **Change repository visibility**

**ข้อควรระวังสำคัญเมื่อเปลี่ยนจาก Public เป็น Private:**

- ทุกคนที่เคย fork repo นี้ไว้ตอนที่ยังเป็น public จะยังคง fork ของตัวเองไว้ได้ (fork จะไม่ถูกลบ) แต่จะไม่สามารถเห็น repo ต้นทางที่เป็น private แล้วได้อีก
- Star, watcher เดิมจะยังคงอยู่ แต่คนทั่วไปจะเข้าถึงไม่ได้อีก
- ถ้าเคยเปิด GitHub Pages จาก repo นี้ หน้าเว็บจะยังคงออนไลน์อยู่ (เว้นแต่จะปิดเอง) แต่คนอื่นดูซอร์สโค้ดเบื้องหลังไม่ได้แล้ว

**เมื่อเปลี่ยนจาก Private เป็น Public:**

- ทุกคนบนโลกจะมองเห็นโค้ดทั้งหมด **รวมถึงประวัติ commit ทั้งหมดตั้งแต่ต้น** ด้วย — ถ้าเคย commit ข้อมูลลับ (เช่น API key, password) ไว้ในอดีต แม้จะลบไฟล์นั้นออกไปแล้วในปัจจุบัน **ข้อมูลนั้นก็ยังอยู่ในประวัติและเปิดเผยทันที** ต้องตรวจสอบและใช้เครื่องมือ เช่น `git filter-repo` หรือ BFG Repo-Cleaner ลบออกจากประวัติก่อนเปลี่ยนเป็น public เสมอ

### 164.3 Transfer ownership (โอนย้ายความเป็นเจ้าของ)

ใน **Danger Zone** เช่นกัน จะมีตัวเลือก **Transfer ownership** ใช้เมื่อต้องการย้าย repo ไปให้บัญชีอื่นหรือ organization อื่นเป็นเจ้าของ

ขั้นตอนคร่าว ๆ:

1. พิมพ์ชื่อ repo เพื่อยืนยัน
2. ระบุ username หรือชื่อ organization ปลายทางที่จะรับโอน
3. ผู้รับโอนต้องกดยืนยันรับ (ผ่านอีเมลหรือการแจ้งเตือนใน GitHub) จึงจะเสร็จสมบูรณ์

**ผลกระทบที่ตามมา:**

- Issue, Pull Request, Star, Watcher, Webhook ส่วนใหญ่จะย้ายตามไปด้วย
- URL เก่าจะ redirect ไปยัง URL ใหม่โดยอัตโนมัติ (คล้ายกับตอนเปลี่ยนชื่อ repo)
- สิทธิ์ของ collaborator เดิมอาจต้องถูกตั้งค่าใหม่โดยเจ้าของใหม่ ขึ้นอยู่กับนโยบายของปลายทาง

### 164.4 Archive repository (เก็บถาวร)

อยู่ใน **Danger Zone** เช่นกัน ตัวเลือก **Archive this repository**

การ archive จะทำให้ repo กลายเป็น **read-only แบบสมบูรณ์**:

- ไม่สามารถ push, สร้าง issue ใหม่, comment, merge PR ได้อีก
- ยังคง clone, fork, ดาวน์โหลดโค้ดได้ตามปกติ
- แถบ badge "Archived" จะแสดงชัดเจนที่หน้า repo เพื่อบอกทุกคนว่าโปรเจกต์นี้หยุดพัฒนาแล้ว
- สามารถ **unarchive** กลับมาใช้งานได้ปกติทีหลัง (ไม่ใช่การลบถาวร)

เหมาะกับโปรเจกต์ที่ทำเสร็จสมบูรณ์แล้ว ไม่มีแผนพัฒนาต่อ แต่ยังอยากเก็บไว้เป็นหลักฐานหรือให้คนอ้างอิงโค้ดได้

### 164.5 Delete repository (ลบถาวร)

ตัวเลือกอันตรายที่สุดใน **Danger Zone** — **Delete this repository**

ขั้นตอนการลบถูกออกแบบให้ทำได้ยากโดยตั้งใจ เพื่อป้องกันการลบผิดพลาด:

1. คลิก **Delete this repository**
2. ระบบจะแสดงคำเตือนว่าอะไรจะหายไปบ้าง (โค้ด, issue, PR, wiki, star, ทุกอย่าง)
3. ต้องพิมพ์ข้อความยืนยันในรูปแบบ `owner/repo-name` ให้ตรงเป๊ะ
4. กด **I understand the consequences, delete this repository**
5. บางกรณี GitHub อาจขอให้ยืนยันรหัสผ่านหรือ 2FA เพิ่มเติม

**ผลกระทบ:** การลบ repo **ไม่สามารถกู้คืนได้เองจากหน้าเว็บ** (แตกต่างจาก archive) แม้ GitHub Support อาจช่วยกู้คืนได้ในบางกรณีถ้าติดต่อเร็วพอ แต่ไม่ใช่สิ่งที่ควรพึ่งพา ให้ถือว่า **การลบคือการลบถาวรจริง** เสมอ

> **คำแนะนำ:** ก่อนลบ repo จริง ให้พิจารณา **Archive** แทนเสมอ ถ้าไม่แน่ใจ 100% ว่าจะไม่ต้องใช้อีก การ archive ปลอดภัยกว่าและย้อนกลับได้

---

## Step 165: About section (description, website link), Topics สำหรับจัดหมวดหมู่ค้นหาง่าย

### 165.1 About section คืออะไร

ที่หน้าแรกของ repo ฝั่งขวาบนจะมีกล่อง **About** ซึ่งเป็นข้อมูลสรุปสั้น ๆ ของโปรเจกต์ คลิกไอคอนรูปเฟือง ⚙️ ข้าง ๆ กล่อง About เพื่อแก้ไข จะเห็นฟิลด์:

| ฟิลด์ | ใช้ทำอะไร |
|---|---|
| **Description** | คำอธิบายสั้น ๆ (1-2 บรรทัด) ว่า repo นี้คืออะไร แสดงทั้งในหน้า repo และในผลการค้นหา |
| **Website** | ลิงก์ไปยังเว็บไซต์ที่เกี่ยวข้อง เช่น เว็บ demo, GitHub Pages ของโปรเจกต์นี้, หรือเอกสารประกอบ |
| **Topics** | คำ tag สำหรับจัดหมวดหมู่ |
| **Include in the home page** | ติ๊กให้แสดง Releases และ Packages ที่หน้าแรก repo |

**ทำไม Description ถึงสำคัญ:** เวลามีคนค้นหาโปรเจกต์บน GitHub Search หรือ Google คำอธิบายนี้คือสิ่งแรกที่คนเห็นก่อนตัดสินใจว่าจะคลิกเข้าไปดูหรือไม่ ควรเขียนให้กระชับแต่บอกจุดเด่นชัดเจน เช่น:

```
"CLI tool สำหรับแปลงไฟล์ CSV เป็น JSON ที่รองรับไฟล์ขนาดใหญ่ระดับ GB"
```

ดีกว่า

```
"my project"
```

### 165.2 Topics คืออะไร ใช้ทำไม

**Topics** คือคำ tag (keyword) ที่ติดไว้กับ repo เพื่อให้ค้นหาและจัดหมวดหมู่ได้ง่ายขึ้น ตัวอย่าง topics ที่นิยมใช้:

```
javascript, react, machine-learning, cli-tool, api,
open-source, thai-language, beginner-friendly, docker, rest-api
```

**ประโยชน์ของ Topics:**

1. **ค้นหาเจอง่ายขึ้น** — คนที่ค้นหา topic เช่น `react` บน GitHub Explore หรือหน้า https://github.com/topics/react จะเจอ repo ของคุณถ้าติด topic นี้ไว้
2. **แสดงในหน้า repo อย่างเด่นชัด** — Topics จะแสดงเป็น badge เล็ก ๆ ใต้ชื่อ repo ทำให้คนเข้าใจเนื้อหาได้เร็วโดยไม่ต้องอ่าน README
3. **เชื่อมโยงกับโปรเจกต์อื่นที่คล้ายกัน** — คนที่กำลังดู repo ที่มี topic เดียวกันมีโอกาสเจอ repo ของคุณผ่านการเชื่อมโยงนี้
4. **สื่อสารเทคโนโลยีที่ใช้ได้รวดเร็ว** — โดยไม่ต้องเปิดอ่านโค้ดหรือไฟล์ package.json/requirements.txt

### 165.3 วิธีเพิ่ม Topics

1. ที่หน้าแรกของ repo คลิกไอคอนเฟือง ⚙️ ข้างกล่อง **About**
2. ในช่อง **Topics** พิมพ์คำแล้วกด Enter ทีละคำ (เพิ่มได้หลายคำ)
3. กด **Save changes**

**คำแนะนำการเลือก Topics:**

- ใช้ตัวพิมพ์เล็กทั้งหมด และใช้ขีดกลาง `-` แทนช่องว่าง (เช่น `machine-learning` ไม่ใช่ `Machine Learning`)
- ใส่ทั้งชื่อภาษา/เฟรมเวิร์กหลัก (`python`, `django`) และคำที่บอกลักษณะโปรเจกต์ (`web-scraper`, `chatbot`, `portfolio`)
- ไม่ควรใส่มากเกินไปจนดูรก แนะนำ 5-10 topics ที่ตรงประเด็นที่สุด

---

## Step 166: การ import repo จากที่อื่นเข้า GitHub ด้วย GitHub Importer

บางครั้งคุณอาจมีโปรเจกต์ที่เก็บอยู่บนระบบอื่น เช่น GitLab, Bitbucket, SVN server, หรือแม้แต่ Git server ส่วนตัว (self-hosted) และต้องการย้ายเข้ามาอยู่บน GitHub GitHub มีเครื่องมือชื่อ **GitHub Importer** ช่วยให้ทำได้ง่ายโดยไม่ต้องใช้ command line เลย

### 166.1 เข้าถึง GitHub Importer

ไปที่ https://github.com/new/import หรือกด **`+`** มุมขวาบน แล้วเลือก **Import repository**

### 166.2 ขั้นตอนการ import

1. **The URL for your source repository** — ใส่ URL ของ repository ต้นทาง (รองรับ Git, Subversion (SVN), Mercurial (Hg), TFS)
2. ถ้า repository ต้นทางเป็น private จะต้องใส่ username/password หรือ token ของระบบต้นทางเพื่อให้ GitHub เข้าถึงได้
3. **Your new repository details** — ตั้งชื่อ repo ใหม่บน GitHub, เลือก owner, เลือก public/private
4. คลิก **Begin import**
5. GitHub จะแสดงสถานะความคืบหน้าการ import แบบ real-time (ดึงข้อมูล → แปลงข้อมูล → อัปโหลดเข้า GitHub)
6. เมื่อเสร็จจะมีอีเมลแจ้งเตือนและหน้า repo ใหม่จะพร้อมใช้งานทันที

### 166.3 สิ่งที่ import มาด้วย และสิ่งที่ไม่มา

| มาด้วย | ไม่มาด้วย (ต้องย้ายเอง) |
|---|---|
| ประวัติ commit ทั้งหมด | Issues, Pull Requests ของระบบเดิม |
| Branch และ Tag ทั้งหมด | Wiki (บางกรณี), CI/CD configuration เดิม |
| ไฟล์และโฟลเดอร์ทั้งหมด | Webhook, integration settings เดิม |

**ข้อควรระวังเมื่อ import จาก SVN:** เนื่องจาก SVN เป็น centralized VCS ที่มีแนวคิดเรื่อง branch ต่างจาก Git (ใน SVN branch คือโฟลเดอร์ธรรมดา) การแปลงโครงสร้างอาจไม่สมบูรณ์แบบเสมอไป ควรตรวจสอบผลลัพธ์หลัง import อย่างละเอียดก่อนใช้งานจริง โดยเฉพาะโปรเจกต์ที่มีโครงสร้าง branch/tag ซับซ้อน

### 166.4 ทางเลือกอื่นสำหรับการย้าย repo (กรณีที่ต้นทางเป็น Git อยู่แล้ว)

ถ้า repository ต้นทางเป็น **Git** อยู่แล้ว (เช่นย้ายจาก GitLab มา GitHub) บางครั้งการใช้ command line เองจะควบคุมได้แม่นยำกว่า GitHub Importer:

```bash
# clone แบบ bare (เก็บทุก branch/tag แบบสมบูรณ์ ไม่มี working directory)
git clone --bare https://gitlab.com/username/old-project.git
cd old-project.git

# เปลี่ยน remote ไปที่ GitHub repo ใหม่ (ที่สร้างแบบว่างเปล่าไว้ก่อน)
git push --mirror https://github.com/username/new-project.git
```

`--mirror` จะ push ทุก branch, ทุก tag ไปยังปลายทางแบบสมบูรณ์ เหมาะกับกรณีที่ต้องการย้ายทั้งหมดโดยไม่พึ่งเครื่องมือ import อัตโนมัติ

---

## Step 167: Repository templates — สร้าง repo ต้นแบบเพื่อใช้ซ้ำ (Use this template button)

### 167.1 ปัญหาที่ Repository Template แก้ให้

ถ้าคุณเริ่มโปรเจกต์ใหม่บ่อย ๆ ที่มีโครงสร้างไฟล์คล้ายเดิมทุกครั้ง เช่น โปรเจกต์ Express API ที่มีโฟลเดอร์ `src/`, `tests/`, ไฟล์ `.eslintrc`, `package.json` ที่ config เหมือนเดิมทุกครั้ง — การต้อง copy ไฟล์เหล่านี้ไปมาหรือ clone repo เก่ามาลบประวัติออกทุกครั้งเป็นเรื่องน่าเบื่อและเสียเวลา

**Repository Template** คือ repo ที่ถูกทำเครื่องหมายพิเศษไว้ ให้คนอื่น (หรือตัวคุณเอง) สามารถกด **"Use this template"** เพื่อสร้าง repo ใหม่ที่มีไฟล์เหมือนต้นแบบทุกอย่าง **แต่เริ่มต้นด้วยประวัติ commit ใหม่ทั้งหมด (ไม่ใช่ fork)**

### 167.2 วิธีตั้งค่า repo ให้เป็น Template

1. ไปที่ repo ที่ต้องการทำเป็นต้นแบบ
2. ไปที่ **Settings → General**
3. เลื่อนหาส่วน **Template repository** แล้วติ๊กช่อง **Template repository**

หลังติ๊กแล้ว หน้าแรกของ repo จะเปลี่ยนปุ่ม **Code** สีเขียวให้กลายเป็นปุ่ม **Use this template** สีเขียวแทน

### 167.3 วิธีใช้ template สร้าง repo ใหม่

1. ไปที่ repo ที่เป็น template
2. คลิกปุ่ม **Use this template → Create a new repository**
3. กรอกชื่อ repo ใหม่, เลือก owner, visibility ตามปกติเหมือนสร้าง repo ทั่วไป
4. ตัวเลือกเพิ่มเติม **Include all branches** — ถ้าติ๊ก จะคัดลอกทุก branch ของ template มาด้วย ไม่ใช่แค่ branch default
5. คลิก **Create repository from template**

Repo ใหม่ที่ได้จะมีไฟล์ทั้งหมดเหมือน template **แต่มี commit history ของตัวเองใหม่หมด (แค่ 1 commit เริ่มต้น)** ต่างจาก fork ที่จะพ่วงประวัติเดิมทั้งหมดของต้นฉบับมาด้วย

### 167.4 Template repository ต่างจาก Fork อย่างไร

| หัวข้อ | Template repository | Fork |
|---|---|---|
| ประวัติ commit | เริ่มใหม่ทั้งหมด | สืบทอดประวัติเดิมทั้งหมด |
| ความเชื่อมโยงกับต้นฉบับ | ไม่มีความเชื่อมโยงใด ๆ (เป็น repo อิสระ) | ยังเชื่อมโยงกับต้นฉบับ (เห็น "forked from") |
| ส่ง Pull Request กลับต้นฉบับได้ไหม | ไม่ได้ (เป็นคนละ repo กันโดยสมบูรณ์) | ได้ (นี่คือจุดประสงค์หลักของ fork) |
| ใช้เมื่อไหร่ | ต้องการ "จุดเริ่มต้น" ใหม่ที่ไม่เกี่ยวข้องกับต้นฉบับอีก | ต้องการมีส่วนร่วม/เสนอการเปลี่ยนแปลงกับโปรเจกต์ต้นฉบับ |
| ตัวอย่างการใช้งาน | Boilerplate โปรเจกต์, starter kit, template องค์กร | Contribute เข้า Open Source project |

### 167.5 ตัวอย่างการใช้งานจริงในองค์กร

หลายองค์กรสร้าง **Organization-level template repositories** เช่น:

- `company-node-api-template` — โครงสร้างมาตรฐานสำหรับทุก API service ที่เขียนด้วย Node.js (มี Dockerfile, CI config, lint rules, folder structure เหมือนกันหมด)
- `company-react-app-template` — โครงสร้างมาตรฐานสำหรับ frontend app ทุกตัว

เมื่อทีมจะเริ่มโปรเจกต์ใหม่ ทุกคนกด "Use this template" แทนที่จะเริ่มจากศูนย์ ทำให้โปรเจกต์ใหม่ทุกตัวมีมาตรฐานเดียวกันตั้งแต่วันแรก ลด configuration ซ้ำซ้อน และง่ายต่อการ maintain ในระยะยาว

---

## Step 168: การลบ/เก็บถาวร (archive) repo อย่างปลอดภัย ผลกระทบที่ตามมา

Step นี้จะเจาะลึกเรื่อง archive และ delete ให้ครบถ้วนกว่า Step 164 เพราะเป็นการกระทำที่ย้อนกลับได้ยากหรือย้อนกลับไม่ได้เลย จำเป็นต้องเข้าใจให้รอบด้านก่อนลงมือ

### 168.1 เช็คลิสต์ก่อนตัดสินใจ Archive หรือ Delete

ก่อนกดปุ่มใดปุ่มหนึ่ง ให้ถามตัวเองตามลำดับนี้:

1. **มีใครใช้งาน repo นี้อยู่หรือไม่?** — ตรวจสอบ Insights → Traffic ว่ามี clone/visit ล่าสุดหรือไม่ (ดู Step 169)
2. **มี Package หรือ Release ที่คนอื่นพึ่งพาอยู่หรือไม่?** — ถ้ามีคน `npm install` หรือดาวน์โหลด release จาก repo นี้ การลบจะทำให้ dependency ของคนอื่นพังทันที
3. **มีข้อมูลสำคัญที่ยังไม่ได้ backup ไว้ที่อื่นหรือไม่?** — เช่น Issue ที่บันทึกการตัดสินใจสำคัญ, Wiki ที่มีเอกสารเฉพาะ
4. **repo นี้ถูกอ้างอิงจากที่อื่นหรือไม่?** — เช่น เป็น submodule ของโปรเจกต์อื่น, ถูกลิงก์ในเอกสาร, ใช้เป็น GitHub Pages ของโดเมนหลัก

ถ้าตอบว่า "ไม่แน่ใจ" ในข้อใดข้อหนึ่ง → **ให้เลือก Archive แทน Delete เสมอ**

### 168.2 ผลกระทบของการ Archive อย่างละเอียด

เมื่อ archive repo แล้ว:

- **Pull Request และ Issue ที่เปิดอยู่จะถูกล็อก** ไม่สามารถ comment, merge, หรือปิดเปิดใหม่ได้
- **GitHub Actions workflow จะหยุดทำงาน** ไม่มี event ใดมา trigger ได้อีก (แม้จะตั้ง schedule ไว้ก็ตาม)
- **Collaborator ยังคง clone/fork/ดาวน์โหลดโค้ดได้ตามปกติ** เพียงแต่แก้ไขอะไรกลับเข้า repo ไม่ได้
- **Badge "Public archive" หรือ "Archived"** จะแสดงเด่นชัดที่หัว repo เพื่อเตือนทุกคนที่เข้ามาดู
- Star, Watcher, จำนวน Fork เดิมยังคงแสดงผลตามปกติ

**การ unarchive** ทำได้จากหน้า Settings เช่นกัน (ปุ่มจะเปลี่ยนเป็น **Unarchive this repository**) และ repo จะกลับมาใช้งานได้ปกติทุกอย่างทันที — นี่คือเหตุผลที่ archive ปลอดภัยกว่า delete มาก

### 168.3 ผลกระทบของการ Delete อย่างละเอียด

เมื่อ delete repo แล้ว **สิ่งเหล่านี้จะหายไปทันทีและถาวร**:

- ซอร์สโค้ดทั้งหมดและประวัติ commit ทั้งหมด
- Issues และ Pull Requests ทั้งหมด (รวมถึงการสนทนาในนั้น)
- Wiki ของ repo
- Releases และไฟล์แนบทั้งหมดที่อัปโหลดไว้กับ release
- GitHub Pages ที่ deploy จาก repo นี้ (เว็บไซต์จะล่มทันที)
- Webhook และ integration settings ที่ตั้งไว้
- Star count และ Watcher (แม้จะสร้าง repo ชื่อเดิมขึ้นใหม่ ตัวเลขเหล่านี้จะเริ่มนับ 0 ใหม่หมด)

**สิ่งที่ "อาจ" ไม่หายไปโดยสมบูรณ์:**

- ถ้าเคยเป็น public repo และมีคน fork ไว้ก่อนลบ **fork ของคนอื่นจะยังคงอยู่** (แต่จะไม่เห็นความเชื่อมโยง "forked from" กับ repo ต้นฉบับที่หายไปแล้ว)
- ถ้าเคย push code ขึ้น package registry (เช่น npm, Docker Hub) แยกต่างหาก แพ็กเกจที่เผยแพร่ไปแล้วจะยังอยู่ที่ registry นั้นแม้ repo ต้นทางจะถูกลบ

### 168.4 แนวปฏิบัติที่ปลอดภัยก่อนลบจริง

1. **สำรองข้อมูลไว้ก่อนเสมอ** — clone repo แบบ `--mirror` เก็บไว้ในเครื่อง หรือ export ข้อมูลสำคัญอื่น ๆ (Issues สามารถ export ผ่าน API หรือเครื่องมือ third-party)
2. **แจ้งทีม/ผู้ที่เกี่ยวข้องล่วงหน้า** ถ้าเป็น repo ที่มีคนอื่นใช้งานร่วม
3. **พิจารณา archive ก่อนเสมอ** อย่างน้อย 1-2 สัปดาห์ ก่อนตัดสินใจลบถาวร เพื่อดูว่ามีใครติดต่อมาขอใช้งานหรือ complain หรือไม่
4. **ตรวจสอบว่าไม่มีระบบ production ใดอ้างอิง repo นี้อยู่** เช่น deployment pipeline, submodule ในโปรเจกต์อื่น

---

## Step 169: GitHub repo Insights เบื้องต้น (traffic, contributors graph, commit activity, network graph)

GitHub มีแท็บ **Insights** อยู่บนแถบเมนูของทุก repo ซึ่งรวบรวมข้อมูลสถิติและภาพรวมของกิจกรรมใน repo ไว้หลายมุมมอง

### 169.1 Traffic

เข้าถึงที่ **Insights → Traffic**

แสดงข้อมูลย้อนหลัง **14 วันล่าสุด** เท่านั้น (ไม่สามารถดูย้อนหลังไกลกว่านี้ได้) ประกอบด้วย:

- **Views** — จำนวนครั้งที่หน้า repo ถูกเปิดดู และจำนวน unique visitors
- **Clones** — จำนวนครั้งที่ repo ถูก `git clone` และจำนวน unique cloners
- **Referring sites** — เว็บไซต์ที่มีคนคลิกลิงก์เข้ามายัง repo นี้ (เช่น Google, Twitter/X, Reddit, Hacker News)
- **Popular content** — ไฟล์หรือหน้าที่มีคนเข้าดูมากที่สุดใน repo (เช่น README.md มักจะสูงสุดเสมอ)

> **ข้อจำกัดสำคัญ:** เฉพาะเจ้าของ repo หรือคนที่มีสิทธิ์ push เท่านั้นที่เห็นข้อมูล Traffic ได้ (คนทั่วไปที่แค่เข้ามาดู repo แบบ read-only มองไม่เห็นหน้านี้)

### 169.2 Contributors

เข้าถึงที่ **Insights → Contributors**

แสดงกราฟจำนวน **commit ของแต่ละคน** ตลอดช่วงเวลาทั้งหมดของ repo เรียงจากคนที่ contribute มากที่สุดไปน้อยที่สุด แต่ละคนจะมีกราฟเส้นแยกแสดง commit, additions (บรรทัดที่เพิ่ม), deletions (บรรทัดที่ลบ) รายสัปดาห์

ใช้ประโยชน์ได้ในการ:

- ดูภาพรวมว่าใครคือผู้มีส่วนร่วมหลักของโปรเจกต์
- ตรวจสอบว่ากิจกรรมของโปรเจกต์ยังคึกคักอยู่หรือซบเซาลง
- ให้เครดิตกับผู้ร่วมพัฒนาอย่างเป็นรูปธรรม

### 169.3 Commit activity (Community Standards มักอยู่ใกล้กัน)

เข้าถึงที่ **Insights → Commits**

แสดงกราฟจำนวน commit **รายสัปดาห์ย้อนหลัง 1 ปี** ในรูปแบบกราฟแท่ง ทำให้เห็นภาพรวมได้เร็วว่าช่วงไหนของปีที่โปรเจกต์มีกิจกรรมสูง/ต่ำ เหมาะกับการดู pattern การทำงานของทีม เช่น มักจะมี commit พุ่งสูงก่อนถึง deadline หรือก่อนออก release

### 169.4 Network graph

เข้าถึงที่ **Insights → Network** (หรือ URL `/network`)

แสดงแผนภาพ **branch และ fork ทั้งหมด** ของ repo ในรูปแบบกราฟเส้นที่แตกแขนงออกจากกัน คล้ายกับที่ `git log --graph` แสดงในเครื่อง local แต่ครอบคลุมถึง **fork ของคนอื่นด้วย**

ประโยชน์:

- เห็นภาพรวมว่ามีใคร fork ไปแล้วพัฒนาต่อในทิศทางไหนบ้าง
- เห็นจุดที่ branch แยกออกจากกันและจุดที่ merge กลับมารวมกัน
- มีประโยชน์มากสำหรับ maintainer ของโปรเจกต์ Open Source ขนาดใหญ่ที่มี fork จำนวนมาก ใช้ตรวจสอบว่ามี fork ไหนที่พัฒนาฟีเจอร์น่าสนใจที่ควรพิจารณาดึงกลับเข้าโปรเจกต์หลัก

### 169.5 Dependency graph และ Community Standards (แถมความรู้เพิ่มเติม)

ในเมนู Insights ยังมีอีก 2 หัวข้อที่ควรรู้จักไว้คร่าว ๆ (จะเจาะลึกใน Part หลัง ๆ ของหลักสูตร):

- **Dependency graph** — แสดงรายการ library/package ที่โปรเจกต์นี้พึ่งพา (ดึงจากไฟล์อย่าง `package.json`, `requirements.txt`) และแจ้งเตือนถ้ามีช่องโหว่ความปลอดภัย (Dependabot alerts)
- **Community Standards** — checklist ว่า repo นี้มีไฟล์มาตรฐานที่โปรเจกต์ Open Source ที่ดีควรมีครบหรือไม่ เช่น README, LICENSE, CONTRIBUTING.md, CODE_OF_CONDUCT.md, Issue template

---

## Step 170: แบบฝึกหัด — สร้าง repo ใหม่ตั้งแต่ web UI จนเชื่อมกับเครื่อง local ครบวงจร

ถึงเวลาลงมือปฏิบัติจริงให้ครบทั้งสองสถานการณ์ที่เรียนมาใน Part นี้ ทำตามลำดับด้านล่างทีละขั้นตอนด้วยตัวเอง

### แบบฝึกหัดที่ 1: สร้างบน GitHub ก่อน แล้วค่อย clone มาเครื่อง

**สถานการณ์:** เริ่มโปรเจกต์ใหม่ที่ยังไม่มีไฟล์อะไรอยู่ในเครื่องเลย

1. เข้า https://github.com/new
2. ตั้งชื่อ repo ว่า `git-course-exercise-17a`
3. ใส่ description: `"แบบฝึกหัด Part 17 - สร้าง repo บน GitHub ก่อนแล้วค่อย clone"`
4. เลือก **Public**
5. ติ๊ก **Add a README file**
6. เลือก **.gitignore template: Node**
7. เลือก **License: MIT License**
8. คลิก **Create repository**
9. ที่หน้า repo ใหม่ คลิกปุ่ม **`<> Code`** สีเขียว คัดลอก HTTPS URL
10. เปิด Terminal แล้วรัน:
    ```bash
    cd ~/git-course
    git clone https://github.com/your-username/git-course-exercise-17a.git
    cd git-course-exercise-17a
    ```
11. ตรวจสอบว่า clone สำเร็จ:
    ```bash
    ls -la
    git remote -v
    git log --oneline
    ```
12. สร้างไฟล์ใหม่ทดสอบ commit และ push:
    ```bash
    echo "# บันทึกการฝึกฝน Part 17" > notes.md
    git add notes.md
    git commit -m "Add practice notes"
    git push
    ```
    (สังเกตว่าครั้งนี้พิมพ์แค่ `git push` เฉย ๆ โดยไม่ต้องระบุ `origin main` เพราะ `git clone` ตั้ง upstream tracking ให้อัตโนมัติแล้ว)
13. รีเฟรชหน้าเว็บ GitHub ตรวจสอบว่าเห็นไฟล์ `notes.md` ปรากฏขึ้นจริง

### แบบฝึกหัดที่ 2: มี local repo อยู่ก่อนแล้ว ค่อยเชื่อมเข้า GitHub

**สถานการณ์:** มีโปรเจกต์ที่ทำงานในเครื่องมาสักพักแล้ว ยังไม่เคยเชื่อมกับ GitHub เลย

1. สร้างโฟลเดอร์และ local repo ใหม่:
   ```bash
   cd ~/git-course
   mkdir git-course-exercise-17b
   cd git-course-exercise-17b
   git init
   echo "# โปรเจกต์ทดลอง 17b" > README.md
   echo "node_modules/" > .gitignore
   git add .
   git commit -m "Initial commit"
   git branch -M main
   ```
2. เพิ่ม commit อีกสัก 1-2 อันเพื่อจำลองว่าทำงานมาสักพักแล้ว:
   ```bash
   echo "console.log('hello');" > index.js
   git add index.js
   git commit -m "Add index.js"
   ```
3. ไปที่ https://github.com/new สร้าง repo ชื่อ `git-course-exercise-17b`
4. **สำคัญมาก:** คราวนี้ **ห้ามติ๊ก** README, .gitignore, license ใด ๆ ทั้งสิ้น ปล่อยให้ repo ว่างเปล่า 100%
5. คลิก Create repository
6. ในหน้าที่ปรากฏ ให้คัดลอกคำสั่งจากส่วน **"…or push an existing repository from the command line"** แล้วนำมารันในเครื่อง (หรือพิมพ์เองตามนี้):
   ```bash
   git remote add origin https://github.com/your-username/git-course-exercise-17b.git
   git push -u origin main
   ```
7. ตรวจสอบผลลัพธ์:
   ```bash
   git remote -v
   git log --oneline
   ```
8. รีเฟรชหน้าเว็บ GitHub ตรวจสอบว่าเห็น commit ทั้ง 2 อันจากเครื่อง local ปรากฏครบถ้วน

### แบบฝึกหัดที่ 3: ฝึกตั้งค่า repo เพิ่มเติม (ต่อยอดจาก exercise 1 หรือ 2)

เลือก repo ใดก็ได้จากสองอันข้างต้น แล้วลองทำสิ่งต่อไปนี้ให้ครบ:

- [ ] เพิ่ม **Description** และลอง **Topics** อย่างน้อย 3 คำที่ About section
- [ ] ลองเปลี่ยน **visibility** จาก Public เป็น Private แล้วเปลี่ยนกลับเป็น Public (สังเกตข้อความเตือนที่ GitHub แสดงระหว่างทาง)
- [ ] เข้าไปดู **Insights → Traffic**, **Insights → Contributors**, **Insights → Network** ของ repo (ถ้ายังไม่มีข้อมูลมากพอ ให้สังเกตว่าหน้าตาของแต่ละหน้าเป็นอย่างไร)
- [ ] ลองติ๊ก **Template repository** ใน Settings แล้วสังเกตว่าปุ่มบนหน้าแรกของ repo เปลี่ยนจาก "Code" เป็น "Use this template" จริงหรือไม่
- [ ] ทดลอง **Archive** repo ฝึกหัดตัวใดตัวหนึ่ง (เลือกตัวที่ไม่สำคัญ) แล้วสังเกตการเปลี่ยนแปลงของหน้า repo จากนั้นลอง **Unarchive** กลับมา

### เฉลยแนวคิด — จุดที่มักผิดพลาดบ่อยที่สุด

1. **ติ๊ก "Add a README" ตอนสร้าง repo ทั้ง ๆ ที่มี local repo อยู่แล้ว** → ทำให้ push ครั้งแรกล้มเหลวเพราะประวัติไม่ตรงกัน (unrelated histories) แก้ได้ด้วย `git pull origin main --allow-unrelated-histories` แต่ป้องกันไว้ก่อนดีกว่าแก้ทีหลัง
2. **ลืม `-u` ตอน push ครั้งแรก** → ทำให้ครั้งต่อไปต้องพิมพ์ `git push origin main` เต็ม ๆ ทุกครั้ง ไม่สามารถพิมพ์ `git push` เฉย ๆ ได้
3. **สับสนระหว่าง HTTPS กับ SSH URL** → คัดลอกผิดแท็บจากปุ่ม `<> Code` ทำให้ authenticate ไม่ผ่านหรือขึ้น error ที่ไม่คุ้นเคย
4. **ลบ repo โดยไม่ archive ดูก่อน** → เมื่อพบว่ายังมีคนใช้งานอยู่ก็สายเกินแก้ไขแล้ว

---

## สรุป Part 17

ใน Part นี้เราได้เรียนรู้ว่า:

1. การสร้าง repository บน GitHub ผ่านหน้าเว็บทำได้ง่าย แต่ต้องเข้าใจผลของแต่ละตัวเลือก โดยเฉพาะเรื่อง public/private, README, .gitignore, และ license
2. การเชื่อม local repo ที่มีอยู่แล้วเข้ากับ GitHub ใช้คำสั่ง `git remote add origin` ตามด้วย `git push -u origin main` และต้องสร้าง GitHub repo แบบว่างเปล่าเสมอเพื่อไม่ให้ประวัติชนกัน
3. การ clone repo มีสองวิธีหลักคือผ่าน HTTPS (ตั้งค่าง่ายกว่า) และ SSH (สะดวกระยะยาวกว่าแต่ต้องตั้งค่า key ก่อน) ซึ่งจะเรียนรายละเอียดเต็มใน Part 18
4. Repository Settings มีตัวเลือกสำคัญคือเปลี่ยนชื่อ, เปลี่ยน visibility, transfer ownership, archive และ delete ซึ่งแต่ละอย่างมีผลกระทบต่างกัน โดยเฉพาะ delete ที่ย้อนกลับไม่ได้
5. About section และ Topics ช่วยให้ repo ค้นหาเจอง่ายขึ้นและสื่อสารเนื้อหาของโปรเจกต์ได้ชัดเจนโดยไม่ต้องอ่านโค้ด
6. GitHub Importer และ `git push --mirror` ช่วยย้ายโปรเจกต์จากระบบอื่นเข้า GitHub ได้ทั้งแบบผ่านหน้าเว็บและแบบ command line
7. Repository Templates ช่วยให้เริ่มโปรเจกต์ใหม่ที่มีโครงสร้างมาตรฐานได้รวดเร็ว โดยไม่พ่วงประวัติเดิมมาเหมือน fork
8. GitHub Insights (Traffic, Contributors, Commit activity, Network graph) ให้ข้อมูลภาพรวมของกิจกรรมและการใช้งาน repo ที่เป็นประโยชน์ต่อการตัดสินใจดูแลโปรเจกต์
9. ได้ลงมือฝึกจริงทั้งสองสถานการณ์: สร้างบน GitHub ก่อนแล้ว clone และมี local repo อยู่ก่อนแล้วค่อยเชื่อม

**ต่อไป:** [Part 18: SSH Key และการตั้งค่าความปลอดภัยในการเชื่อมต่อ](./part-018-ssh-key-ความปลอดภัย.md)
