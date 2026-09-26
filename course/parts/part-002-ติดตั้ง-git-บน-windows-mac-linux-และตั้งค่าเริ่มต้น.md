# Part 02: ติดตั้ง Git บน Windows/Mac/Linux และตั้งค่าเริ่มต้น

> **Step ในหลักสูตรนี้:** Step 11–20
> **เฟส:** 1 — ปูพื้นฐานความคิดเรื่อง Version Control และ Git
> **เป้าหมายของ Part นี้:** ติดตั้ง Git ให้สำเร็จบนระบบปฏิบัติการที่คุณใช้งานจริง ไม่ว่าจะเป็น Windows, macOS หรือ Linux และตั้งค่าเริ่มต้นที่จำเป็นทุกอย่างให้ครบ ตั้งแต่ชื่อผู้ใช้ อีเมล text editor เริ่มต้น ชื่อ branch เริ่มต้น ไปจนถึงการจัดการเรื่อง line ending ข้ามแพลตฟอร์ม เพื่อให้พร้อมลงมือเรียนรู้แนวคิดหลักของ Git ใน Part 03

---

## สารบัญของ Part นี้

- Step 11: ติดตั้ง Git บน Windows แบบละเอียดทีละหน้าจอ
- Step 12: ติดตั้ง Git บน macOS (3 วิธี พร้อมข้อดีข้อเสีย)
- Step 13: ติดตั้ง Git บน Linux (apt, dnf, pacman, จาก source)
- Step 14: ตรวจสอบการติดตั้งและวิธีอัปเดต Git เป็นเวอร์ชันล่าสุด
- Step 15: ตั้งค่า `user.name` และ `user.email` และเข้าใจ scope ของ config
- Step 16: ตั้งค่า default text editor สำหรับเขียน commit message
- Step 17: ตั้งค่า default branch name เป็น `main`
- Step 18: เข้าใจไฟล์ config ของ Git และคำสั่งตรวจสอบค่าที่ตั้งไว้
- Step 19: ตั้งค่าสี และจัดการปัญหา line ending (CRLF vs LF)
- Step 20: แบบฝึกหัดลงมือทำ — ตั้งค่า Git ให้ครบทุกอย่าง พร้อม Checklist ก่อนไปต่อ

---

## Step 11: ติดตั้ง Git บน Windows แบบละเอียดทีละหน้าจอ

Windows เป็นระบบปฏิบัติการเดียวในสามตัวที่ **ไม่มี Git ติดตั้งมาให้ตั้งแต่แรก** ดังนั้นเราต้องดาวน์โหลดและติดตั้งเองผ่านตัวติดตั้งอย่างเป็นทางการ

### 11.1 ดาวน์โหลด Git for Windows

1. เปิดเบราว์เซอร์ไปที่เว็บไซต์ทางการ **https://git-scm.com**
2. เว็บไซต์จะตรวจจับ OS อัตโนมัติและแสดงปุ่ม **"Download for Windows"** ให้กด
3. ไฟล์ที่ได้จะเป็น `.exe` ชื่อประมาณ `Git-2.4x.x-64-bit.exe` (ตัวเลขเวอร์ชันจะเปลี่ยนไปตามช่วงเวลา)

> โปรเจกต์ที่อยู่เบื้องหลังตัวติดตั้งนี้ชื่อ **Git for Windows** (บางครั้งเรียกสั้น ๆ ว่า msysGit ในอดีต) เป็นโปรเจกต์แยกที่ทำหน้าที่ port Git ให้ทำงานบน Windows ได้อย่างสมบูรณ์ พร้อมแถม **Git Bash** มาให้ด้วย

### 11.2 ขั้นตอนการติดตั้งทีละหน้าจอ

เมื่อดับเบิลคลิกไฟล์ `.exe` แล้ว ตัวติดตั้งจะพาไปทีละหน้าจอ ต่อไปนี้คือหน้าจอสำคัญที่ต้องตัดสินใจ (หน้าจออื่นที่ไม่กระทบการใช้งานหลักสามารถกด Next ผ่านไปได้):

**หน้าที่ 1: License Agreement** — Git เป็น Open Source ภายใต้ GNU GPL v2 กด **Next**

**หน้าที่ 2: Select Destination Location** — ตำแหน่งที่จะติดตั้ง ปกติปล่อยเป็นค่า default (`C:\Program Files\Git`) ได้เลย

**หน้าที่ 3: Select Components** — เลือกส่วนประกอบเสริม แนะนำให้ติ๊กเลือก:

| Component | คำแนะนำ |
|---|---|
| Windows Explorer integration (Git Bash Here, Git GUI Here) | ✅ ติ๊กไว้ — คลิกขวาในโฟลเดอร์แล้วเปิด Git Bash ได้ทันที สะดวกมาก |
| Git LFS (Large File Storage) | ✅ ติ๊กไว้ — จะได้ใช้ตอนเรียนเรื่อง LFS ใน Part หลัง ๆ |
| Associate .git* files with default text editor | ✅ ติ๊กไว้ |
| Add a Git Bash Profile to Windows Terminal | ✅ แนะนำถ้าใช้ Windows Terminal |

**หน้าที่ 4: Select Start Menu Folder** — ปล่อย default ได้

**หน้าที่ 5: เลือก Default Editor ที่ Git จะใช้** — นี่คือหน้าจอสำคัญมาก! ตัวติดตั้งจะถามว่าจะใช้โปรแกรมอะไรเป็น editor เริ่มต้นตอนเขียน commit message เช่นตอนรัน `git commit` โดยไม่ใส่ `-m`

ตัวเลือกที่มีให้ (บางส่วน):

- **Vim** (ค่า default ดั้งเดิม) — ทรงพลังแต่ผู้เริ่มต้นมักงงว่าจะออกจากโปรแกรมยังไง (กด `Esc` แล้วพิมพ์ `:wq` แล้ว Enter)
- **Nano** — เรียบง่าย เหมาะกับมือใหม่ที่สุด มีคำสั่งแสดงอยู่ด้านล่างจอตลอด
- **Visual Studio Code** (ถ้าติดตั้งไว้แล้ว) — แนะนำที่สุดสำหรับคนที่ใช้ VS Code เป็นหลัก
- **Notepad++**, **Sublime Text**, **Notepad ธรรมดา** ก็มีให้เลือกเช่นกัน

> **คำแนะนำสำหรับหลักสูตรนี้:** ถ้าคุณเพิ่งเริ่มต้นและยังไม่คุ้นกับ Vim แนะนำเลือก **Nano** หรือถ้าติดตั้ง VS Code ไว้แล้วให้เลือก **"Use Visual Studio Code as Git's default editor"** ไปเลย เราจะสอนวิธีเปลี่ยนค่านี้ทีหลังได้ใน Step 16 อยู่แล้วถ้าอยากเปลี่ยนใจ

**หน้าที่ 6: Adjusting the name of the initial branch** — หน้านี้ให้เลือกชื่อ branch เริ่มต้นเมื่อสร้าง repository ใหม่ มีสองตัวเลือก:

- **Let Git decide** (ค่า default ของ Git เองคือ `master`)
- **Override the default branch name for new repositories** — ใส่ `main`

แนะนำให้เลือก **Override** แล้วพิมพ์ `main` เพื่อให้ตรงกับมาตรฐานอุตสาหกรรมปัจจุบัน (อธิบายเหตุผลละเอียดใน Step 17)

**หน้าที่ 7: Adjusting your PATH environment** — หน้าจอสำคัญที่สุดหน้าหนึ่ง มี 3 ตัวเลือก:

| ตัวเลือก | ความหมาย | คำแนะนำ |
|---|---|---|
| Use Git from Git Bash only | ใช้คำสั่ง `git` ได้เฉพาะใน Git Bash เท่านั้น เข้าไม่ถึงจาก CMD/PowerShell | ไม่แนะนำถ้าต้องใช้ CMD/PowerShell ด้วย |
| **Git from the command line and also from 3rd-party software** | เพิ่ม Git เข้า Windows PATH ทำให้เรียกใช้ `git` ได้จากทุกที่ ทั้ง Git Bash, CMD, PowerShell, VS Code terminal | ✅ **แนะนำ (ค่า default)** |
| Use Git and optional Unix tools from the Command Prompt | เพิ่มเครื่องมือ Unix (เช่น `ls`, `grep`) เข้า CMD ด้วย อาจทับคำสั่งของ Windows เอง | ไม่แนะนำสำหรับมือใหม่ |

เลือกตัวกลาง **"Git from the command line and also from 3rd-party software"** จะปลอดภัยและสะดวกที่สุด

**หน้าที่ 8: Choosing the SSH executable** — เลือกว่าจะใช้ OpenSSH ตัวไหน แนะนำ **"Use bundled OpenSSH"** (ค่า default)

**หน้าที่ 9: Choosing HTTPS transport backend** — เลือก **"Use the OpenSSL library"** (ค่า default) เพื่อใช้ certificate store มาตรฐาน

**หน้าที่ 10: Configuring the line ending conversions** — หน้านี้สำคัญมากและจะอธิบายละเอียดใน Step 19 มี 3 ตัวเลือก:

- **Checkout Windows-style, commit Unix-style line endings** (`core.autocrlf=true`) ✅ **แนะนำสำหรับ Windows**
- Checkout as-is, commit Unix-style line endings (`core.autocrlf=input`)
- Checkout as-is, commit as-is (`core.autocrlf=false`)

สำหรับผู้ใช้ Windows ทั่วไป แนะนำเลือกตัวแรก (ค่า default)

**หน้าที่ 11: Configuring the terminal emulator to use with Git Bash** — มี 2 ตัวเลือก:

- **Use MinTTY** (ค่า default) — terminal emulator ของ Git Bash เอง รองรับสี รองรับการ resize หน้าต่างได้ดี แนะนำ ✅
- Use Windows' default console window — ใช้ Command Prompt ปกติของ Windows แทน ฟีเจอร์จำกัดกว่า

**หน้าที่ 12: Choose the default behavior of `git pull`** — แนะนำ **"Default (fast-forward or merge)"** ไปก่อน (เดี๋ยวเราจะเรียนเรื่อง rebase กับ merge อย่างละเอียดใน Part หลัง ๆ)

**หน้าที่ 13: Choose a credential helper** — แนะนำ **Git Credential Manager** (ค่า default) เพื่อให้จำ username/password หรือ token เวลา push/pull ผ่าน HTTPS ได้โดยไม่ต้องพิมพ์ซ้ำทุกครั้ง

**หน้าที่ 14-16: ตัวเลือกเสริมอื่น ๆ** เช่น "Enable file system caching", "Enable symbolic links" — ปล่อยเป็นค่า default ได้เลย แล้วกด **Install**

### 11.3 Git Bash คืออะไร

หลังติดตั้งเสร็จ คุณจะเห็นโปรแกรมใหม่ชื่อ **Git Bash** ถูกติดตั้งมาด้วย

> **Git Bash** คือโปรแกรม terminal ที่จำลองสภาพแวดล้อมแบบ **Unix/Linux shell (Bash)** ให้ทำงานบน Windows ได้ ทำให้คุณใช้คำสั่ง Unix พื้นฐาน เช่น `ls`, `pwd`, `mkdir`, `rm`, `cat` และคำสั่ง Git ทั้งหมดได้เหมือนกับผู้ใช้ macOS/Linux เป๊ะ ๆ

เหตุผลที่ Git Bash สำคัญมากสำหรับผู้ใช้ Windows:

1. **คำสั่งในหลักสูตรนี้ (และเกือบทุกบทเรียน Git ในโลก) เขียนโดยอ้างอิง Unix shell** — ถ้าใช้ CMD หรือ PowerShell คำสั่งเสริมบางอย่าง (ที่ไม่ใช่ Git โดยตรง) อาจไม่ตรงกัน
2. รองรับ **SSH**, **shell script (.sh)** และเครื่องมือ Unix พื้นฐานได้ในตัว
3. หน้าตาและพฤติกรรมเหมือนกับที่ผู้ใช้ macOS/Linux เห็น ทำให้การเรียนรู้ transferable ข้ามแพลตฟอร์มได้จริง

> **คำแนะนำ:** ตลอดหลักสูตรนี้ ถ้าคุณใช้ Windows ให้เปิด **Git Bash** เป็นหลักในการรันคำสั่งทุกคำสั่ง จะได้ผลลัพธ์ตรงกับตัวอย่างในเอกสารมากที่สุด (คำสั่ง `git` เองใช้ได้เหมือนกันใน PowerShell/CMD ด้วย เพราะเราเลือกเพิ่มเข้า PATH แล้วในขั้นตอนติดตั้ง)

### 11.4 ทดสอบเปิด Git Bash

คลิกขวาที่ Desktop หรือในโฟลเดอร์ใด ๆ แล้วเลือก **"Git Bash Here"** (ถ้าติ๊กตัวเลือกไว้ตอนติดตั้ง) หรือค้นหา "Git Bash" จาก Start Menu จะเห็นหน้าต่าง terminal สีดำเปิดขึ้นมา พร้อม prompt ประมาณนี้:

```bash
user@DESKTOP-XXXXX MINGW64 ~/Desktop
$
```

ลองพิมพ์:

```bash
git --version
```

ถ้าเห็นผลลัพธ์แบบ `git version 2.4x.x.windows.1` แปลว่าติดตั้งสำเร็จแล้ว

---

## Step 12: ติดตั้ง Git บน macOS (3 วิธี พร้อมข้อดีข้อเสีย)

macOS มีวิธีติดตั้ง Git ได้หลากหลายกว่า Windows เพราะเป็นระบบที่มีรากฐานจาก Unix อยู่แล้ว มาดูทั้ง 3 วิธีหลัก

### 12.1 วิธีที่ 1: Xcode Command Line Tools (แนะนำสำหรับคนทั่วไป)

Apple แถม Git มาพร้อมกับชุดเครื่องมือพัฒนาที่เรียกว่า **Xcode Command Line Tools** เปิด **Terminal** แล้วพิมพ์:

```bash
git --version
```

ถ้าเครื่องยังไม่เคยติดตั้งเครื่องมือนี้มาก่อน macOS จะเด้งหน้าต่าง popup ขึ้นมาถามว่า:

```
The "git" command requires the command line developer tools.
Would you like to install the tools now?
```

กด **Install** แล้วรอสักครู่ (ใช้เวลาประมาณ 5-15 นาที ขึ้นอยู่กับความเร็วเน็ต) ระบบจะติดตั้ง Git พร้อมกับเครื่องมือพัฒนาพื้นฐานอื่น ๆ เช่น `make`, `gcc`, `clang` ให้ด้วย

หรือจะสั่งติดตั้งตรง ๆ ผ่านคำสั่งเดียวก็ได้ โดยไม่ต้องรอให้ระบบถามเอง:

```bash
xcode-select --install
```

**ข้อดี:**
- ง่ายที่สุด ไม่ต้องติดตั้งอะไรเพิ่มเติม (ไม่ต้องมี Homebrew)
- เป็นวิธีที่ Apple รองรับอย่างเป็นทางการ เสถียรมาก
- ได้เครื่องมือ dev พื้นฐานอื่นติดมาด้วย

**ข้อเสีย:**
- Git เวอร์ชันที่ได้มักจะ **ล้าหลังกว่าเวอร์ชันล่าสุด** เสมอ (Apple อัปเดตช้า)
- อัปเดตเป็นเวอร์ชันใหม่ได้ยาก ต้องรอ macOS อัปเดตระบบ หรือ full Xcode update
- ไม่มี package manager คอยจัดการ dependency ให้

### 12.2 วิธีที่ 2: Homebrew (แนะนำสำหรับนักพัฒนา)

**Homebrew** คือ package manager ยอดนิยมที่สุดสำหรับ macOS (บางคนเรียกว่า "the missing package manager for macOS")

ถ้ายังไม่มี Homebrew ติดตั้งก่อนด้วยคำสั่ง (คัดลอกจาก https://brew.sh):

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

เมื่อมี Homebrew แล้ว ติดตั้ง Git ด้วยคำสั่งเดียว:

```bash
brew install git
```

ตรวจสอบว่า Git ที่ใช้อยู่ตอนนี้มาจาก Homebrew จริงหรือไม่:

```bash
which git
```

ควรได้ผลลัพธ์เป็น `/opt/homebrew/bin/git` (บนเครื่อง Apple Silicon เช่น M1/M2/M3) หรือ `/usr/local/bin/git` (บนเครื่อง Intel) — **ไม่ใช่** `/usr/bin/git` ซึ่งเป็นตัวของ Apple/Xcode

**ข้อดี:**
- ได้ Git **เวอร์ชันล่าสุดเสมอ** เพราะ Homebrew อัปเดต formula บ่อยมาก
- อัปเดตในอนาคตทำได้ง่ายมากด้วยคำสั่งเดียว (ดู Step 14)
- ติดตั้ง package/เครื่องมืออื่น ๆ ที่จำเป็นสำหรับงานพัฒนาต่อได้สะดวกในอนาคต (Node.js, Python, ฯลฯ)

**ข้อเสีย:**
- ต้องติดตั้ง Homebrew ก่อน ซึ่งมีขั้นตอนเพิ่มขึ้นมาหนึ่งชั้น
- ใช้พื้นที่ดิสก์เพิ่มขึ้นเล็กน้อยจากตัว Homebrew เอง

### 12.3 วิธีที่ 3: ติดตั้งจาก Installer โดยตรง

ดาวน์โหลด `.dmg` installer อย่างเป็นทางการจาก **https://git-scm.com/download/mac** ซึ่งจะพาไปที่โปรเจกต์ **git-scm** ที่ build โดยทีม Git for macOS แยกต่างหาก

ขั้นตอน: ดับเบิลคลิกไฟล์ `.pkg` ที่ดาวน์โหลดมา แล้วกด Continue ไปเรื่อย ๆ ตามตัวติดตั้งมาตรฐานของ macOS (Install for all users, ใส่รหัสผ่าน admin, Close)

**ข้อดี:**
- ไม่ต้องพึ่งพา Homebrew หรือรอ Apple อัปเดต Xcode
- เป็น installer อย่างเป็นทางการ ตรงไปตรงมา

**ข้อเสีย:**
- อัปเดตเวอร์ชันใหม่ต้องดาวน์โหลด `.pkg` ใหม่มาติดตั้งทับเองทุกครั้ง ไม่มีคำสั่งอัปเดตอัตโนมัติ
- เป็นวิธีที่ใช้กันน้อยที่สุดในหมู่นักพัฒนา macOS ปัจจุบัน

### 12.4 สรุปเปรียบเทียบ 3 วิธีสำหรับ macOS

| วิธี | ความง่าย | เวอร์ชันล่าสุดไหม | อัปเดตในอนาคต | เหมาะกับใคร |
|---|---|---|---|---|
| Xcode CLT | ง่ายที่สุด | ไม่ล่าสุด | ยาก | ผู้เริ่มต้น ใช้งานเบื้องต้น |
| Homebrew | ปานกลาง | ล่าสุดเสมอ | ง่ายมาก | นักพัฒนา แนะนำที่สุด |
| Installer (.pkg) | ง่าย | ค่อนข้างใหม่ | ต้องโหลดใหม่เอง | ผู้ไม่อยากติดตั้ง Homebrew |

> **คำแนะนำของหลักสูตรนี้:** ถ้าคุณตั้งใจจะเป็นนักพัฒนาซอฟต์แวร์จริงจัง แนะนำให้ติดตั้งผ่าน **Homebrew** เพราะในอนาคตคุณจะต้องใช้ Homebrew ติดตั้งเครื่องมืออื่น ๆ อีกมากอยู่ดี

---

## Step 13: ติดตั้ง Git บน Linux (apt, dnf, pacman, จาก source)

Linux แทบทุก distro มี Git อยู่ใน package repository อย่างเป็นทางการอยู่แล้ว การติดตั้งจึงง่ายและเร็วมาก

### 13.1 Debian / Ubuntu และตระกูลที่ใช้ `apt`

```bash
sudo apt update
sudo apt install git -y
```

- `sudo apt update` — อัปเดตรายการ package ล่าสุดจาก repository ก่อนติดตั้งเสมอ (เป็นนิสัยที่ดี)
- `sudo apt install git -y` — ติดตั้ง Git โดยตอบ "yes" อัตโนมัติให้ทุกคำถาม

ตรวจสอบเวอร์ชันที่ได้:

```bash
git --version
```

> **หมายเหตุ:** Git ใน Ubuntu LTS (เช่น 22.04, 24.04) มักจะไม่ใช่เวอร์ชันล่าสุดที่สุด ถ้าต้องการเวอร์ชันใหม่กว่านั้นจริง ๆ สามารถเพิ่ม PPA ทางการของทีม Git ได้:

```bash
sudo add-apt-repository ppa:git-core/ppa -y
sudo apt update
sudo apt install git -y
```

### 13.2 Fedora / RHEL / CentOS Stream และตระกูลที่ใช้ `dnf`

```bash
sudo dnf install git -y
```

สำหรับระบบรุ่นเก่าที่ยังใช้ `yum` (เช่น CentOS 7):

```bash
sudo yum install git -y
```

### 13.3 Arch Linux / Manjaro และตระกูลที่ใช้ `pacman`

```bash
sudo pacman -Syu git
```

- `-S` = sync (ติดตั้ง package)
- `-y` = อัปเดตฐานข้อมูล package ก่อน
- `-u` = อัปเกรดระบบทั้งหมดไปด้วยในตัว (นิยมทำคู่กันใน Arch เพราะ Arch เป็น rolling release)

### 13.4 openSUSE

```bash
sudo zypper install git
```

### 13.5 ตารางสรุปคำสั่งตาม Distro

| Distro | Package Manager | คำสั่งติดตั้ง |
|---|---|---|
| Ubuntu / Debian / Linux Mint | apt | `sudo apt install git` |
| Fedora / RHEL / CentOS Stream | dnf | `sudo dnf install git` |
| Arch / Manjaro | pacman | `sudo pacman -S git` |
| openSUSE | zypper | `sudo zypper install git` |
| Alpine Linux | apk | `sudo apk add git` |

### 13.6 ติดตั้งจาก Source Code (สำหรับกรณีต้องการเวอร์ชันล่าสุดสุด ๆ)

บางครั้ง package repository ของ distro อาจมี Git เวอร์ชันเก่ามาก หรือคุณต้องการ build เอง (เช่น เพื่อ contribute โค้ดกลับให้โปรเจกต์ Git) วิธีนี้ใช้เวลานานกว่าและซับซ้อนกว่ามาก จึงแนะนำเฉพาะกรณีจำเป็นจริง ๆ เท่านั้น

```bash
# ติดตั้ง dependency ที่จำเป็นสำหรับ build (ตัวอย่างบน Ubuntu/Debian)
sudo apt install -y dh-autoreconf libcurl4-gnutls-dev libexpat1-dev \
    gettext libz-dev libssl-dev

# ดาวน์โหลด source code เวอร์ชันล่าสุด
curl -LO https://github.com/git/git/archive/refs/tags/v2.45.0.tar.gz
tar -zxf v2.45.0.tar.gz
cd git-2.45.0

# build และติดตั้ง
make configure
./configure --prefix=/usr/local
make -j$(nproc)
sudo make install
```

หลัง build เสร็จ ให้ตรวจสอบว่า Git ตัวใหม่ถูกเรียกใช้จริงด้วย `git --version` และ `which git` (อาจต้องปรับ `PATH` ให้ `/usr/local/bin` มาก่อน path เดิมของระบบ)

---

## Step 14: ตรวจสอบการติดตั้งสำเร็จ และวิธีอัปเดต Git เป็นเวอร์ชันล่าสุด

### 14.1 ตรวจสอบด้วย `git --version`

ไม่ว่าจะติดตั้งด้วยวิธีไหนบน OS ใดก็ตาม คำสั่งตรวจสอบจะเหมือนกันเสมอ:

```bash
git --version
```

ตัวอย่างผลลัพธ์บนแต่ละ OS:

```
# Windows
git version 2.45.1.windows.1

# macOS (Homebrew)
git version 2.45.2

# Linux (Ubuntu apt)
git version 2.43.0
```

สังเกตว่าเลขเวอร์ชันอาจต่างกันเล็กน้อยในแต่ละเครื่อง — **ไม่เป็นปัญหา** ตราบใดที่เป็นเวอร์ชัน **2.30 ขึ้นไป** ก็เพียงพอสำหรับการเรียนหลักสูตรนี้ (เนื้อหาส่วนใหญ่ใช้ฟีเจอร์ที่มีมานานแล้ว มีบางบทที่พูดถึงฟีเจอร์ใหม่ ๆ จะระบุเวอร์ชันขั้นต่ำกำกับไว้ชัดเจนเสมอ)

### 14.2 ตรวจสอบตำแหน่งไฟล์โปรแกรม Git ที่ถูกเรียกใช้

บางครั้งเครื่องหนึ่งอาจมี Git มากกว่า 1 ตัวติดตั้งอยู่ (เช่น ตัวจาก Xcode และตัวจาก Homebrew พร้อมกัน) ใช้คำสั่งนี้เพื่อดูว่าตัวไหนถูกเรียกใช้งานจริงตาม `PATH`:

```bash
which git        # macOS/Linux/Git Bash
where git         # Windows CMD/PowerShell
```

### 14.3 วิธีอัปเดต Git เป็นเวอร์ชันล่าสุด แยกตาม OS

| OS / วิธีติดตั้งเดิม | คำสั่งอัปเดต |
|---|---|
| Windows (Git for Windows) | เปิด Git Bash แล้วรัน `git update-git-for-windows` หรือดาวน์โหลด installer ใหม่จาก git-scm.com มาติดตั้งทับ |
| macOS (Homebrew) | `brew update && brew upgrade git` |
| macOS (Xcode CLT) | รัน `softwareupdate --list` แล้ว `softwareupdate --install` ตามชื่อ update ที่เกี่ยวกับ Command Line Tools หรือรอ macOS อัปเดตระบบ |
| Ubuntu/Debian (apt) | `sudo apt update && sudo apt upgrade git` |
| Fedora (dnf) | `sudo dnf upgrade git` |
| Arch (pacman) | `sudo pacman -Syu` (อัปเดตทั้งระบบตามธรรมชาติของ rolling release) |

ตัวอย่างการอัปเดตบน Windows ผ่าน Git Bash:

```bash
git update-git-for-windows
```

คำสั่งนี้จะเช็คเวอร์ชันล่าสุดจากอินเทอร์เน็ตและดาวน์โหลด/ติดตั้งให้อัตโนมัติถ้ามีเวอร์ชันใหม่กว่า

### 14.4 ทำไมต้องใช้เวอร์ชันที่ไม่เก่าเกินไป

Git เวอร์ชันใหม่ ๆ มักจะมาพร้อม:

- **ฟีเจอร์ใหม่ที่ช่วยชีวิต** เช่น `git switch`, `git restore` (แยกหน้าที่ออกจาก `git checkout` ที่แต่เดิมทำหลายอย่างปนกันจนสับสน)
- **การแก้ไขช่องโหว่ด้านความปลอดภัย (security patches)** — Git มีการค้นพบและแก้ CVE (Common Vulnerabilities and Exposures) อยู่เรื่อย ๆ
- **ประสิทธิภาพที่ดีขึ้น** โดยเฉพาะกับ repository ขนาดใหญ่

หลักสูตรนี้แนะนำให้ใช้ Git เวอร์ชัน **2.40 ขึ้นไป** เพื่อให้ได้ประสบการณ์ที่ตรงกับตัวอย่างในเอกสารมากที่สุด

---

## Step 15: ตั้งค่า `user.name` และ `user.email` และเข้าใจ scope ของ config

หลังติดตั้ง Git เสร็จ สิ่งแรกที่ **ต้องทำก่อนสร้าง commit แรก** คือการบอก Git ว่า "คุณคือใคร" เพราะทุก commit ใน Git จะถูกประทับชื่อผู้เขียนและอีเมลไว้ถาวรในประวัติ

### 15.1 ตั้งค่าด้วยคำสั่ง `git config`

```bash
git config --global user.name "สมชาย ใจดี"
git config --global user.email "somchai@example.com"
```

- `git config` คือคำสั่งหลักสำหรับอ่าน/เขียนค่า configuration ของ Git
- `--global` หมายถึง "ใช้ค่านี้กับทุก repository บนเครื่องนี้" (รายละเอียด scope ดูหัวข้อถัดไป)
- `user.name` และ `user.email` คือ **key** ของค่าที่ตั้ง ส่วนข้อความในเครื่องหมายคำพูดคือ **value**

> **สำคัญ:** ใส่ชื่อและอีเมลที่คุณต้องการให้ปรากฏใน commit history จริง ๆ ถ้าอนาคตคุณจะใช้งานกับ GitHub/GitLab แนะนำให้ใช้อีเมลเดียวกับที่ใช้สมัครบัญชีนั้น ๆ เพื่อให้ระบบเชื่อมโยง commit เข้ากับโปรไฟล์ของคุณได้ถูกต้อง (จะอธิบายละเอียดเรื่องนี้ใน Part ที่พูดถึง GitHub)

### 15.2 ตรวจสอบค่าที่ตั้งไปแล้ว

```bash
git config --global user.name
git config --global user.email
```

ผลลัพธ์ควรแสดงค่าที่คุณเพิ่งตั้งไป เช่น:

```
สมชาย ใจดี
somchai@example.com
```

### 15.3 ทำความเข้าใจ Scope ทั้ง 3 ระดับของ Git Config

Git config มีทั้งหมด **3 ระดับ (scope)** เรียงจากกว้างที่สุดไปแคบที่สุด:

| Scope | Flag | ไฟล์ที่เก็บค่า | มีผลกับอะไร |
|---|---|---|---|
| **System** | `--system` | `/etc/gitconfig` (Linux/Mac) หรือ `C:\Program Files\Git\etc\gitconfig` (Windows) | ทุกผู้ใช้ ทุก repository บนเครื่องนี้ |
| **Global** | `--global` | `~/.gitconfig` หรือ `~/.config/git/config` | ทุก repository ของผู้ใช้คนปัจจุบันบนเครื่องนี้ |
| **Local** | `--local` (หรือไม่ใส่ flag เลย, เป็นค่า default) | `.git/config` ภายใน repository นั้น ๆ | เฉพาะ repository ปัจจุบันเท่านั้น |

### 15.4 ลำดับความสำคัญ (Priority) เมื่อมีค่าซ้อนกัน

ถ้ามีการตั้งค่าคีย์เดียวกันไว้หลายระดับพร้อมกัน Git จะใช้กฎ **"ค่าที่แคบกว่าชนะค่าที่กว้างกว่า"**:

```
Local (.git/config)       ← ชนะทุกระดับ (สำคัญที่สุด)
    ↓ ถ้าไม่มีค่านี้ ให้ดู
Global (~/.gitconfig)     ← ชนะ System
    ↓ ถ้าไม่มีค่านี้ ให้ดู
System (/etc/gitconfig)   ← ค่าพื้นฐานสุด
```

**ตัวอย่างสถานการณ์จริง:** สมมติคุณตั้งอีเมลส่วนตัวไว้ที่ `--global` แต่มี repository หนึ่งเป็นโปรเจกต์ของบริษัท ที่ต้องการใช้อีเมลบริษัทแทน คุณสามารถเข้าไปใน repository นั้นแล้วตั้งค่า `--local` ทับได้:

```bash
cd ~/work/company-project
git config --local user.email "somchai@company.com"
```

ค่านี้จะมีผลเฉพาะใน repository `company-project` เท่านั้น ส่วน repository อื่น ๆ ทั้งหมดยังใช้อีเมลส่วนตัวจาก `--global` ตามปกติ

### 15.5 ตรวจสอบว่าค่าปัจจุบันมาจาก scope ไหน (แบบคร่าว ๆ ก่อน จะเจาะลึกใน Step 18)

```bash
git config user.email
```

คำสั่งนี้ (ไม่ใส่ `--global`/`--local`) จะคืนค่าที่ **มีผลจริง ณ ตำแหน่งที่คุณยืนอยู่** โดยไล่ตามลำดับความสำคัญด้านบนให้อัตโนมัติ — สะดวกมากเวลาต้องการเช็คว่า "ตอนนี้ Git กำลังจะใช้ชื่อ/อีเมลอะไรถ้าฉัน commit เดี๋ยวนี้"

---

## Step 16: ตั้งค่า Default Text Editor สำหรับเขียน Commit Message

เมื่อคุณรัน `git commit` โดยไม่ใส่ flag `-m "ข้อความ"` Git จะเปิด **text editor** ขึ้นมาให้พิมพ์ commit message แบบยาว ค่า default ของ Git เองมักจะเป็น **Vim** ซึ่งผู้เริ่มต้นจำนวนมากไม่คุ้นเคยและอาจ "ติดอยู่ในโปรแกรม" ออกไม่ได้

### 16.1 ตรวจสอบว่าตอนนี้ตั้ง editor อะไรอยู่

```bash
git config --global core.editor
```

ถ้าไม่มีค่าอะไรแสดงออกมาเลย แปลว่า Git จะไปใช้ค่าจาก environment variable `$VISUAL` หรือ `$EDITOR` ของระบบ หรือถ้าไม่มีอีกก็จะ fallback ไปที่ Vim (บน Unix-like) หรือ Notepad (บาง build ของ Windows)

### 16.2 ตั้งค่าเป็น Nano (แนะนำสำหรับผู้เริ่มต้น)

```bash
git config --global core.editor "nano"
```

Nano เป็น editor ที่เรียบง่ายที่สุด มีคำสั่งลัดแสดงอยู่ด้านล่างจอตลอดเวลา (เช่น `^X` หมายถึง `Ctrl+X` เพื่อออกและบันทึก) เหมาะกับมือใหม่มาก

### 16.3 ตั้งค่าเป็น Vim (สำหรับคนที่อยากฝึกใช้ให้คล่อง)

```bash
git config --global core.editor "vim"
```

**วิธีใช้งาน Vim เบื้องต้นเมื่อ Git เปิดมาให้เขียน commit message:**

1. กด `i` เพื่อเข้าโหมด **Insert** (พิมพ์ข้อความได้)
2. พิมพ์ commit message ตามต้องการ
3. กด `Esc` เพื่อออกจากโหมด Insert
4. พิมพ์ `:wq` แล้วกด `Enter` เพื่อ **บันทึกและออก** (write + quit)
   - ถ้าต้องการยกเลิกไม่บันทึก ให้พิมพ์ `:q!` แทน

### 16.4 ตั้งค่าเป็น VS Code (แนะนำสำหรับคนที่ใช้ VS Code เป็นหลัก)

```bash
git config --global core.editor "code --wait"
```

Flag `--wait` **สำคัญมาก** — มันบอกให้ VS Code เปิดหน้าต่างนั้นค้างไว้และไม่ส่งการควบคุมกลับไปให้ terminal จนกว่าคุณจะปิดแท็บนั้น (ถ้าลืมใส่ `--wait` Git จะคิดว่าคุณเขียนเสร็จทันทีที่ VS Code เปิดขึ้นมา ทำให้ commit message ว่างเปล่า)

> **หมายเหตุสำหรับ Windows:** ต้องแน่ใจว่าได้ติดตั้ง VS Code พร้อมติ๊กตัวเลือก **"Add to PATH"** ตอนติดตั้งไว้แล้ว ไม่งั้นคำสั่ง `code` จะหาไม่เจอ ทดสอบได้ด้วยการพิมพ์ `code --version` ใน terminal ก่อน

### 16.5 ตัวอย่าง Editor อื่น ๆ ที่ตั้งค่าได้

| Editor | คำสั่งตั้งค่า |
|---|---|
| Nano | `git config --global core.editor "nano"` |
| Vim | `git config --global core.editor "vim"` |
| VS Code | `git config --global core.editor "code --wait"` |
| Sublime Text | `git config --global core.editor "subl -n -w"` |
| Notepad++ (Windows) | `git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"` |

### 16.6 ทดสอบ Editor ที่ตั้งไว้

ลองสร้าง repository ทดสอบเล็ก ๆ แล้วลอง commit โดยไม่ใส่ `-m`:

```bash
mkdir ~/git-course/test-editor && cd ~/git-course/test-editor
git init
echo "hello" > test.txt
git add test.txt
git commit
```

Editor ที่ตั้งไว้ควรเปิดขึ้นมาให้พิมพ์ commit message ทันที ลองพิมพ์ข้อความอะไรก็ได้แล้วบันทึกและปิด จะเห็นผลลัพธ์ยืนยันว่า commit สำเร็จ

---

## Step 17: ตั้งค่า Default Branch Name เป็น `main`

### 17.1 ปัญหาเดิม: ทำไม Git ถึงใช้ชื่อ `master` เป็นค่า default

ตั้งแต่ Git ถือกำเนิดในปี 2005 ชื่อ branch แรกที่ถูกสร้างขึ้นอัตโนมัติเมื่อรัน `git init` คือ **`master`** ซึ่งเป็นธรรมเนียมที่สืบทอดมาจาก VCS รุ่นก่อนหน้า (เช่น BitKeeper) โดยไม่ได้มีความหมายเชิงลำดับชั้นทางสังคมแต่อย่างใดในทางเทคนิค — มันแค่หมายถึง "สำเนาหลัก" (คล้ายกับ "master copy" ของเทปหรือแผ่นเสียงต้นฉบับ)

### 17.2 การเปลี่ยนแปลงในปี 2020

ในช่วงกลางปี 2020 ท่ามกลางกระแสการตระหนักถึงประเด็นความหลากหลายและการใช้ภาษาที่ครอบคลุมมากขึ้นในอุตสาหกรรมเทคโนโลยี ชุมชน Open Source จำนวนมาก รวมถึง **GitHub** ได้ประกาศเปลี่ยนชื่อ branch เริ่มต้นสำหรับ repository ที่สร้างใหม่บนแพลตฟอร์มของตนจาก `master` เป็น **`main`**

- **GitHub** เริ่มใช้ `main` เป็นชื่อ default สำหรับ repository ใหม่ทั้งหมดตั้งแต่เดือนตุลาคม 2020
- **GitLab** ก็ทยอยปรับตามในเวลาใกล้เคียงกัน
- **ตัว Git เอง** (core Git) ไม่ได้บังคับเปลี่ยนค่า default ทันที แต่เปิดให้ผู้ใช้ **ตั้งค่าเองได้** ผ่าน `init.defaultBranch` ตั้งแต่ Git เวอร์ชัน 2.28 เป็นต้นมา

ผลคือปัจจุบัน ถ้าคุณสร้าง repository ใหม่ผ่าน `git init` บนเครื่องที่ยังไม่เคยตั้งค่านี้ไว้ ชื่อ branch เริ่มต้นจะยังคงเป็น `master` เพราะเป็นค่า default ดั้งเดิมของตัว Git โปรแกรมเอง แต่เมื่อคุณสร้าง repository บน GitHub/GitLab ผ่านหน้าเว็บ มันจะได้ branch ชื่อ `main` มาให้ทันที — ความไม่สอดคล้องกันนี้เองที่ทำให้เราควรตั้งค่าเครื่องของตัวเองให้ตรงกับมาตรฐานปัจจุบัน

### 17.3 คำสั่งตั้งค่า

```bash
git config --global init.defaultBranch main
```

หลังตั้งค่านี้แล้ว ทุกครั้งที่คุณรัน `git init` ในเครื่องนี้ branch แรกที่ถูกสร้างจะชื่อ `main` โดยอัตโนมัติ แทนที่จะเป็น `master`

### 17.4 ทดสอบผลลัพธ์

```bash
mkdir ~/git-course/test-branch && cd ~/git-course/test-branch
git init
git branch
```

**ก่อนตั้งค่า** (หรือถ้ายังไม่ได้ตั้งค่านี้): คำสั่ง `git branch` มักจะไม่แสดงอะไรเลยเพราะยังไม่มี commit แต่ถ้าดูด้วย:

```bash
git symbolic-ref --short HEAD
```

จะเห็นว่า HEAD ชี้ไปที่ `master`

**หลังตั้งค่า `init.defaultBranch main`:**

```bash
git symbolic-ref --short HEAD
```

จะได้ผลลัพธ์เป็น `main` แทน

### 17.5 ถ้ามี repository เก่าที่ยังใช้ `master` อยู่ล่ะ

การตั้งค่า `init.defaultBranch` **มีผลเฉพาะกับ repository ที่สร้างใหม่หลังจากนี้เท่านั้น** repository เก่าที่มี branch ชื่อ `master` อยู่แล้วจะไม่ถูกเปลี่ยนชื่อให้อัตโนมัติ ถ้าต้องการเปลี่ยนชื่อ branch ของ repository ที่มีอยู่แล้ว (ทั้งบนเครื่องและบน remote) เป็นเรื่องที่ต้องทำแยกต่างหาก ซึ่งเราจะสอนคำสั่ง `git branch -m` และขั้นตอนอัปเดต remote อย่างละเอียดใน Part ที่พูดถึง Branching

---

## Step 18: เข้าใจไฟล์ Config ของ Git และคำสั่งตรวจสอบค่าที่ตั้งไว้

### 18.1 ดูค่า config ทั้งหมดที่มีผลอยู่ในปัจจุบัน

```bash
git config --list
```

ผลลัพธ์ตัวอย่าง:

```
user.name=สมชาย ใจดี
user.email=somchai@example.com
core.editor=code --wait
init.defaultbranch=main
color.ui=auto
core.autocrlf=input
```

คำสั่งนี้จะรวมค่าจาก **ทุก scope** (system + global + local ถ้ารันอยู่ในโฟลเดอร์ที่เป็น repository) เข้าด้วยกัน โดยแสดงเฉพาะค่าที่ **มีผลจริง** ตามลำดับความสำคัญที่อธิบายไปใน Step 15

### 18.2 ดูว่าค่าแต่ละอันมาจากไฟล์ไหน ด้วย `--show-origin`

นี่คือคำสั่งที่มีประโยชน์มากเวลาต้องการ debug ว่า "ทำไมค่านี้ถึงเป็นแบบนี้" หรือ "ค่านี้มาจากไฟล์ไหนกันแน่":

```bash
git config --list --show-origin
```

ผลลัพธ์ตัวอย่าง:

```
file:/etc/gitconfig    core.autocrlf=input
file:/home/somchai/.gitconfig  user.name=สมชาย ใจดี
file:/home/somchai/.gitconfig  user.email=somchai@example.com
file:.git/config       core.repositoryformatversion=0
file:.git/config       core.bare=false
```

จะเห็นว่าแต่ละบรรทัดบอกชัดเจนว่าค่านั้นมาจากไฟล์ path เต็มไหน ทำให้รู้ทันทีว่าเป็นค่าระดับ system, global หรือ local

สามารถเพิ่ม `--show-scope` เพื่อดูชื่อ scope ตรง ๆ ได้เช่นกัน (รองรับตั้งแต่ Git 2.26 ขึ้นไป):

```bash
git config --list --show-scope
```

ผลลัพธ์จะขึ้นต้นแต่ละบรรทัดด้วยคำว่า `system`, `global`, หรือ `local` แทน path ไฟล์

### 18.3 ตำแหน่งไฟล์ config จริงในแต่ละ OS

| Scope | Linux / macOS | Windows |
|---|---|---|
| System | `/etc/gitconfig` | `C:\Program Files\Git\etc\gitconfig` |
| Global | `~/.gitconfig` หรือ `~/.config/git/config` | `C:\Users\<ชื่อผู้ใช้>\.gitconfig` |
| Local | `<path-to-repo>/.git/config` | `<path-to-repo>\.git\config` |

### 18.4 หน้าตาไฟล์ config จริง ๆ

ไฟล์ config ของ Git เป็นไฟล์ text ธรรมดาในรูปแบบคล้าย INI สามารถเปิดดูหรือแก้ไขตรง ๆ ด้วย text editor ก็ได้ ตัวอย่างเนื้อหาไฟล์ `~/.gitconfig`:

```ini
[user]
    name = สมชาย ใจดี
    email = somchai@example.com
[core]
    editor = code --wait
    autocrlf = input
[init]
    defaultBranch = main
[color]
    ui = auto
```

### 18.5 วิธีแก้ไขไฟล์ config โดยตรง

มี 2 วิธีหลัก:

**วิธีที่ 1: ใช้คำสั่ง `git config --global --edit`** (แนะนำ เพราะ Git จะเปิดด้วย editor ที่ตั้งไว้ใน `core.editor` ให้อัตโนมัติ และตรวจสอบ syntax ให้ก่อนบันทึก)

```bash
git config --global --edit
```

**วิธีที่ 2: เปิดไฟล์ตรง ๆ ด้วย text editor ที่ต้องการ**

```bash
code ~/.gitconfig      # เปิดด้วย VS Code
nano ~/.gitconfig       # เปิดด้วย Nano
vim ~/.gitconfig        # เปิดด้วย Vim
```

> **ข้อควรระวัง:** ถ้าแก้ไขไฟล์ตรง ๆ แล้วพิมพ์ syntax ผิด (เช่น วงเล็บเหลี่ยมไม่ปิด) คำสั่ง `git config` ทุกตัวหลังจากนั้นอาจ error หรือทำงานผิดพลาดได้ ควรตรวจสอบด้วย `git config --list` อีกครั้งหลังแก้ไขเสมอ

### 18.6 ลบค่า config ที่ไม่ต้องการ

```bash
git config --global --unset core.editor
```

หรือลบทั้งหมดใน section หนึ่ง:

```bash
git config --global --remove-section color
```

### 18.7 ตรวจสอบค่า config เฉพาะคีย์เดียว

แทนที่จะดูรายการยาว ๆ ทั้งหมด สามารถถามค่าเฉพาะเจาะจงได้:

```bash
git config user.name
git config --global core.autocrlf
```

---

## Step 19: ตั้งค่าสี และจัดการปัญหา Line Ending (CRLF vs LF)

### 19.1 เปิดใช้งานสีใน Output ของ Git

Git มีความสามารถแสดงผลลัพธ์เป็นสี (เช่น เขียวสำหรับบรรทัดที่เพิ่ม แดงสำหรับบรรทัดที่ลบใน `git diff`) ซึ่งช่วยให้อ่านง่ายขึ้นมาก ค่านี้มักเปิดเป็นค่า default อยู่แล้วใน Git เวอร์ชันใหม่ ๆ แต่ตั้งค่าให้ชัดเจนได้ด้วย:

```bash
git config --global color.ui auto
```

- `auto` หมายถึง "แสดงสีเมื่อ output ถูกแสดงบนหน้าจอ terminal โดยตรง" แต่จะ **ไม่ใส่รหัสสี** เมื่อ output ถูก redirect ไปยังไฟล์หรือส่งผ่าน pipe ไปยังโปรแกรมอื่น (เพื่อไม่ให้รหัสสีไปปนกับข้อมูลจริง)

ทดสอบผลลัพธ์:

```bash
cd ~/git-course/test-branch
echo "line 1" > demo.txt
git add demo.txt
git commit -m "add demo.txt"
echo "line 2" >> demo.txt
git diff
```

ควรเห็นบรรทัดที่เพิ่มใหม่ (`+line 2`) แสดงเป็นสีเขียวใน terminal

### 19.2 ปัญหา CRLF vs LF คืออะไร

นี่คือหนึ่งในปัญหาที่ทำให้ทีมที่มีทั้งผู้ใช้ Windows และ macOS/Linux ปวดหัวบ่อยที่สุด

ระบบปฏิบัติการแต่ละตระกูลมีวิธี "จบบรรทัด" (line ending) ในไฟล์ text ที่ต่างกัน:

| ระบบปฏิบัติการ | อักขระจบบรรทัด | ชื่อเรียก |
|---|---|---|
| Windows | `\r\n` (Carriage Return + Line Feed) | **CRLF** |
| macOS (ปัจจุบัน), Linux, Unix | `\n` (Line Feed เท่านั้น) | **LF** |

ปัญหาเกิดขึ้นเมื่อ:

1. นักพัฒนาที่ใช้ **Windows** แก้ไขไฟล์ แล้ว editor บันทึกด้วย CRLF
2. นักพัฒนาที่ใช้ **macOS/Linux** ดึงไฟล์เดียวกันมาดู
3. Git มองว่า **ทุกบรรทัดในไฟล์เปลี่ยนแปลงหมด** ทั้ง ๆ ที่เนื้อหาจริงไม่ได้เปลี่ยนอะไรเลย เพียงเพราะอักขระจบบรรทัดต่างกัน (`\r\n` เทียบกับ `\n`)

ผลลัพธ์คือ `git diff` และ `git blame` จะรก เต็มไปด้วยการเปลี่ยนแปลงปลอม ๆ ที่ไม่มีความหมาย และอาจนำไปสู่ merge conflict ที่ไม่จำเป็นบ่อยมาก

### 19.3 วิธีแก้: ตั้งค่า `core.autocrlf`

Git มีกลไกจัดการเรื่องนี้ผ่านค่า `core.autocrlf` ซึ่งควบคุมพฤติกรรมการแปลง line ending ตอน checkout (ดึงไฟล์ออกมาให้แก้ไข) และตอน commit (บันทึกกลับเข้า repository)

**สำหรับผู้ใช้ Windows:**

```bash
git config --global core.autocrlf true
```

ความหมาย: **"ตอน checkout ให้แปลงเป็น CRLF (เพื่อให้ editor บน Windows ทำงานถูกต้อง) แต่ตอน commit ให้แปลงกลับเป็น LF เสมอ (เพื่อให้ repository เก็บมาตรฐานเดียวกันทุกที่)"**

**สำหรับผู้ใช้ macOS/Linux:**

```bash
git config --global core.autocrlf input
```

ความหมาย: **"ตอน commit ถ้าเจอ CRLF ให้แปลงเป็น LF ก่อนเก็บเข้า repository แต่ตอน checkout ไม่ต้องแปลงอะไร (เพราะระบบเราใช้ LF อยู่แล้วเป็นปกติ)"**

**ถ้าไม่ต้องการให้ Git ยุ่งกับ line ending เลย** (ไม่แนะนำสำหรับทีมผสม OS):

```bash
git config --global core.autocrlf false
```

### 19.4 ตารางสรุปพฤติกรรมของ `core.autocrlf`

| ค่า | ตอน Checkout (นำไฟล์ออกมา) | ตอน Commit (เก็บเข้า repo) | แนะนำสำหรับ |
|---|---|---|---|
| `true` | แปลง LF → CRLF | แปลง CRLF → LF | Windows |
| `input` | ไม่แปลง (คงเดิม) | แปลง CRLF → LF | macOS / Linux |
| `false` | ไม่แปลง | ไม่แปลง | ไม่แนะนำ (เว้นแต่มั่นใจว่าทุกคนตั้งค่า editor สอดคล้องกันเอง) |

> **หลักการสำคัญที่ต้องจำ:** ไม่ว่าจะตั้ง `true` หรือ `input` ผลลัพธ์สุดท้ายคือ **ไฟล์ที่เก็บอยู่ใน repository (บน remote) จะเป็น LF เสมอ** — นี่คือมาตรฐานที่ Git แนะนำ ความต่างกันอยู่แค่ตอน checkout ออกมาให้ผู้ใช้ Windows เห็นเป็น CRLF (ที่ Notepad และโปรแกรมเก่า ๆ บน Windows ต้องการ) เท่านั้น

### 19.5 ควบคุมแบบละเอียดยิ่งขึ้นด้วยไฟล์ `.gitattributes`

สำหรับโปรเจกต์จริงที่มีทีมใช้ OS ผสมกัน วิธีที่แนะนำที่สุดในระยะยาวคือการสร้างไฟล์ `.gitattributes` ไว้ที่ root ของ repository เพื่อบังคับกฎ line ending แบบเดียวกันสำหรับ**ทุกคนในทีม** โดยไม่ต้องพึ่งพาการตั้งค่าเครื่องส่วนตัวของแต่ละคน:

```gitattributes
# บังคับให้ text ทุกไฟล์ใช้ LF เสมอเมื่อเก็บใน repository
* text=auto

# ไฟล์ script ที่รันบน Unix shell ต้องเป็น LF เท่านั้นเสมอ ไม่ว่า OS ไหน checkout
*.sh text eol=lf

# ไฟล์ batch ของ Windows ต้องเป็น CRLF เท่านั้นเสมอ
*.bat text eol=crlf

# ไฟล์ binary ห้าม Git แตะต้อง line ending เด็ดขาด
*.png binary
*.jpg binary
```

เราจะเจาะลึกเรื่อง `.gitattributes` แบบเต็ม ๆ ใน Part ที่พูดถึงการตั้งค่าโปรเจกต์ระดับทีม ตอนนี้ขอให้รู้ไว้ก่อนว่ามันมีอยู่และแก้ปัญหานี้ได้ดีกว่าการพึ่งค่า `core.autocrlf` ส่วนตัวเพียงอย่างเดียว

### 19.6 ตรวจสอบว่าค่า `core.autocrlf` ปัจจุบันตั้งเป็นอะไร

```bash
git config --global core.autocrlf
```

---

## Step 20: แบบฝึกหัดลงมือทำ — ตั้งค่า Git ให้ครบทุกอย่าง

ถึงเวลาลงมือทำจริงเพื่อรวบรวมทุกอย่างที่เรียนมาใน Part นี้เข้าด้วยกัน ทำตามลำดับต่อไปนี้บนเครื่องของคุณเอง

### 20.1 ขั้นตอนที่ 1: ยืนยันว่า Git ติดตั้งสำเร็จแล้ว

```bash
git --version
```

ควรได้ผลลัพธ์เป็นเวอร์ชัน 2.30 ขึ้นไป ถ้ายังไม่ผ่าน กลับไปทำ Step 11/12/13 ตาม OS ของคุณก่อน

### 20.2 ขั้นตอนที่ 2: ตั้งค่าตัวตนของคุณ

```bash
git config --global user.name "ชื่อ-นามสกุลของคุณ"
git config --global user.email "อีเมลของคุณ@example.com"
```

> ใช้ชื่อจริงและอีเมลจริงของคุณ ไม่ต้องใช้ค่าตัวอย่างข้างบน

### 20.3 ขั้นตอนที่ 3: เลือกและตั้งค่า Editor ที่คุณถนัด

เลือกอย่างใดอย่างหนึ่งตามความถนัด:

```bash
# ถ้าถนัด VS Code
git config --global core.editor "code --wait"

# ถ้าถนัด Nano
git config --global core.editor "nano"

# ถ้าถนัด Vim
git config --global core.editor "vim"
```

### 20.4 ขั้นตอนที่ 4: ตั้งค่า Default Branch เป็น `main`

```bash
git config --global init.defaultBranch main
```

### 20.5 ขั้นตอนที่ 5: ตั้งค่าสี

```bash
git config --global color.ui auto
```

### 20.6 ขั้นตอนที่ 6: ตั้งค่า Line Ending ตาม OS ของคุณ

```bash
# Windows เท่านั้น
git config --global core.autocrlf true

# macOS/Linux เท่านั้น
git config --global core.autocrlf input
```

### 20.7 ขั้นตอนที่ 7: ตรวจสอบค่าทั้งหมดที่ตั้งไปพร้อมกันทีเดียว

```bash
git config --list --global
```

ผลลัพธ์ที่ควรเห็น (ปรับตามค่าจริงของคุณ):

```
user.name=ชื่อ-นามสกุลของคุณ
user.email=อีเมลของคุณ@example.com
core.editor=code --wait
init.defaultbranch=main
color.ui=auto
core.autocrlf=input
```

### 20.8 ขั้นตอนที่ 8: ทดสอบด้วยการสร้าง Repository จริงสักตัว

```bash
mkdir ~/git-course/part-02-practice
cd ~/git-course/part-02-practice
git init
git symbolic-ref --short HEAD
```

ผลลัพธ์บรรทัดสุดท้ายควรเป็น `main` ยืนยันว่า `init.defaultBranch` ทำงานถูกต้อง

ทดสอบ commit เพื่อยืนยันว่า `user.name`/`user.email` และ editor ทำงานถูกต้องครบวงจร:

```bash
echo "# บันทึกฝึก Part 02" > README.md
git add README.md
git commit
```

พิมพ์ commit message ใน editor ที่เปิดขึ้นมา เช่น `initial commit for part 02 practice` แล้วบันทึกและปิด จากนั้นตรวจสอบว่า commit มีชื่อผู้เขียนถูกต้อง:

```bash
git log --pretty=fuller
```

ควรเห็นชื่อและอีเมลของคุณปรากฏอยู่ในช่อง `Author:` และ `Commit:`

### 20.9 Checklist ก่อนไป Part 03

ก่อนไปต่อ Part 03 ให้ตรวจสอบว่าคุณ:

- [ ] ติดตั้ง Git สำเร็จบน OS ของตัวเอง และ `git --version` แสดงเวอร์ชัน 2.30 ขึ้นไป
- [ ] (ถ้าใช้ Windows) รู้จักและเปิด Git Bash ได้แล้ว
- [ ] ตั้งค่า `user.name` และ `user.email` เรียบร้อยด้วยข้อมูลจริงของตัวเอง
- [ ] เข้าใจความแตกต่างของ config 3 scope: system / global / local และลำดับความสำคัญ
- [ ] ตั้งค่า `core.editor` เป็น editor ที่ตัวเองถนัดและทดสอบเปิด commit message ได้จริง
- [ ] ตั้งค่า `init.defaultBranch main` และยืนยันด้วย `git symbolic-ref --short HEAD` ว่าได้ `main`
- [ ] รู้ตำแหน่งไฟล์ config จริงบนเครื่องตัวเอง และเคยลองรัน `git config --list --show-origin` แล้ว
- [ ] ตั้งค่า `core.autocrlf` ให้ตรงกับ OS ของตัวเอง และเข้าใจปัญหา CRLF vs LF
- [ ] สร้าง commit แรกได้สำเร็จด้วยข้อมูลผู้เขียนที่ถูกต้อง (ตรวจสอบด้วย `git log --pretty=fuller`)

---

## สรุป Part 02

ใน Part นี้เราได้เรียนรู้ว่า:

1. Git ติดตั้งได้บน Windows ผ่าน installer จาก git-scm.com ซึ่งมาพร้อม **Git Bash** ที่จำลอง Unix shell ให้ใช้งานได้เหมือนกันทุก OS
2. macOS มี 3 วิธีติดตั้ง คือ Xcode Command Line Tools (ง่ายสุดแต่เวอร์ชันเก่า), Homebrew (แนะนำที่สุดสำหรับนักพัฒนา), และ Installer โดยตรง
3. Linux ติดตั้งง่ายผ่าน package manager ของแต่ละ distro (`apt`, `dnf`, `pacman`, `zypper`) หรือ build จาก source ในกรณีพิเศษ
4. ตรวจสอบการติดตั้งด้วย `git --version` เสมอ และรู้วิธีอัปเดต Git เป็นเวอร์ชันล่าสุดตาม OS ของตัวเอง
5. `git config` มี 3 scope: **system** (ทั้งเครื่อง) → **global** (ทั้งผู้ใช้) → **local** (เฉพาะ repository) โดยค่าที่แคบกว่าจะชนะค่าที่กว้างกว่าเสมอ
6. ตั้ง `user.name` และ `user.email` เป็นสิ่งแรกที่ต้องทำก่อน commit ใด ๆ เพราะจะถูกบันทึกถาวรในประวัติ
7. `core.editor` ควบคุมว่า Git จะเปิดโปรแกรมอะไรให้เขียน commit message แบบยาว เลือกได้ตามความถนัด (Nano ง่ายสุด, VS Code ต้องใส่ `--wait`)
8. ชื่อ branch เริ่มต้นเปลี่ยนจาก `master` เป็น `main` ตามมาตรฐานอุตสาหกรรมปัจจุบัน ตั้งค่าได้ด้วย `git config --global init.defaultBranch main`
9. ไฟล์ config จริงอยู่ที่ `/etc/gitconfig` (system), `~/.gitconfig` (global), และ `.git/config` (local) ตรวจสอบที่มาของแต่ละค่าได้ด้วย `git config --list --show-origin`
10. ปัญหา **CRLF vs LF** เกิดจากความต่างของ line ending ระหว่าง Windows กับ Unix-like แก้ได้ด้วย `core.autocrlf` (`true` สำหรับ Windows, `input` สำหรับ macOS/Linux) และในระดับทีมควรใช้ `.gitattributes` ควบคุมให้ชัดเจนยิ่งขึ้น
11. เราได้ตั้งค่า Git ให้ครบทุกอย่างและทดสอบสร้าง commit แรกสำเร็จแล้ว พร้อมที่จะเข้าใจแนวคิดหลักของ Git อย่างลึกซึ้งใน Part ถัดไป

**ต่อไป:** [Part 03: แนวคิดหลักของ Git: Repository, Working Directory, Staging Area, Commit](./part-003-แนวคิดหลักของ-git-repository-working-directory-staging-commit.md)
