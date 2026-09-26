# Part 16: สมัครและตั้งค่าบัญชี GitHub อย่างมืออาชีพ

> **Step ในหลักสูตรนี้:** Step 151–160
> **เฟส:** 3 — GitHub เบื้องต้น (Part นี้คือ Part แรกของเฟส 3)
> **เป้าหมายของ Part นี้:** เปลี่ยนผ่านจากการใช้ Git คนเดียวบนเครื่อง (เฟส 1–2) ไปสู่การใช้แพลตฟอร์ม GitHub อย่างมืออาชีพ ตั้งแต่การสมัครบัญชี ตั้งค่าโปรไฟล์ให้น่าเชื่อถือ เปิดระบบความปลอดภัย 2FA สร้าง Personal Access Token จัดการการแจ้งเตือน ทำความรู้จัก GitHub Plans ติดตั้ง GitHub CLI ไปจนถึงรู้จัก Organization และ Keyboard Shortcut — เพื่อให้บัญชี GitHub ของคุณพร้อมใช้งานจริงในระดับมืออาชีพก่อนเริ่มลงมือสร้าง repository จริงใน Part ถัดไป

---

## เข้าสู่เฟส 3: จาก Git คนเดียว สู่ GitHub แบบทีม

ตลอดเฟส 1 และเฟส 2 (Step 1–150) เราใช้ Git อยู่บนเครื่องของเราเองล้วน ๆ — `git init`, `git commit`, `git branch`, `git merge`, `git stash`, `git tag` ทุกคำสั่งทำงานอยู่ใน **local repository** ไม่มีใครอื่นเห็นงานของเราเลย ต่อให้เคยพิมพ์ `git remote add` หรือ `git push` ไปที่ bare repository ในเครื่องตัวเองใน Part 15 นั่นก็ยังเป็นแค่การจำลองสถานการณ์ ไม่ใช่การทำงานร่วมกับคนจริงบนโลกออนไลน์

จากนี้ไปจนถึง Step 300 เราจะเข้าสู่ **เฟส 3: GitHub เบื้องต้น** ซึ่งเป็นจุดเปลี่ยนสำคัญของหลักสูตรนี้ เพราะ:

1. **Git คือเครื่องมือในเครื่อง** — แต่ **GitHub คือสถานที่ที่โค้ดของคุณไปพบเจอกับโลก** ไม่ว่าจะเป็นเพื่อนร่วมทีม ผู้ว่าจ้าง หรือชุมชน Open Source
2. งานส่วนใหญ่ในโลกการทำงานจริงไม่ได้ใช้ Git แบบเดี่ยว ๆ — เกือบทุกบริษัทใช้ GitHub (หรือ GitLab/Bitbucket ที่เราจะเรียนในเฟสหลัง ๆ) เป็นศูนย์กลางของการทำงานร่วมกัน
3. Pull Request, Code Review, Issue Tracking, CI/CD ที่จะเรียนในเฟสถัดไปทั้งหมด **ต้องมีบัญชี GitHub ที่ตั้งค่าไว้อย่างถูกต้องก่อน**
4. โปรไฟล์ GitHub ของคุณเปรียบเสมือน **Resume สาธารณะ** ของโปรแกรมเมอร์ยุคนี้ — บริษัทจำนวนมากเปิดดู GitHub profile ของผู้สมัครงานก่อนสัมภาษณ์ด้วยซ้ำ

ก่อนที่เราจะไปสร้าง repository แรกบน GitHub ใน Part 17 เราต้อง **เตรียมบัญชีให้พร้อมแบบมืออาชีพเสียก่อน** — ไม่ใช่แค่สมัครแบบขอไปที กรอกอะไรมั่ว ๆ ไม่ตั้งค่าความปลอดภัย เพราะบัญชีนี้จะเป็นตัวตนดิจิทัลของคุณในวงการซอฟต์แวร์ไปอีกหลายปี Part นี้จะพาคุณตั้งค่าทุกอย่างที่จำเป็นทีละขั้นตอนอย่างละเอียด

---

## สารบัญของ Part นี้

- Step 151: สมัครบัญชี GitHub ทีละขั้นตอน (เลือก username อย่างมืออาชีพ, ยืนยันอีเมล)
- Step 152: ตั้งค่า Profile ให้ดูเป็นมืออาชีพ (avatar, bio, social links, Profile README)
- Step 153: ตั้งค่าความปลอดภัย 2FA (Two-Factor Authentication)
- Step 154: Personal Access Token (PAT) — Classic vs Fine-grained
- Step 155: GitHub Notification Settings — จัดการแจ้งเตือนไม่ให้ล้น inbox
- Step 156: ภาพรวม GitHub Plans และ GitHub Student Developer Pack
- Step 157: ติดตั้งและ login GitHub CLI (`gh auth login`)
- Step 158: แนะนำ GitHub Organization เบื้องต้น
- Step 159: Keyboard Shortcuts และ Command Palette ของ GitHub
- Step 160: แบบฝึกหัด — ตั้งค่าบัญชี GitHub ให้ครบพร้อมใช้งานจริง

---

## Step 151: สมัครบัญชี GitHub ทีละขั้นตอน

### 151.1 ก่อนสมัคร — เตรียมอีเมลที่จะใช้จริงในระยะยาว

ก่อนกดสมัคร ให้คิดให้ดีก่อนว่าจะใช้อีเมลไหน เพราะอีเมลนี้จะผูกกับบัญชี GitHub ของคุณไปตลอด (แม้ภายหลังจะเพิ่ม/เปลี่ยนอีเมลได้ แต่อีเมลแรกมักกลายเป็นอีเมลหลักที่ผูกกับ commit history เก่า ๆ)

**คำแนะนำ:**

- ใช้อีเมลส่วนตัวที่คุณเข้าถึงได้ระยะยาว **ไม่ใช่อีเมลของบริษัทหรือมหาวิทยาลัยที่จะหมดอายุ** เมื่อคุณลาออกหรือจบการศึกษา
- ถ้ามีแผนจะสมัคร GitHub Student Developer Pack (Step 156) ให้เตรียมอีเมลของสถาบันการศึกษา (`.ac.th`, `.edu`) ไว้เพิ่มเป็นอีเมลรองภายหลังได้ ไม่จำเป็นต้องใช้สมัครบัญชีหลัก

### 151.2 ขั้นตอนการสมัครบัญชี

1. เปิดเบราว์เซอร์ไปที่ **https://github.com/signup**
2. กรอกข้อมูลตามลำดับที่หน้าเว็บถามทีละหน้า:

```
หน้าจอที่ 1: Enter your email
┌─────────────────────────────────────┐
│  Join GitHub                         │
│  ┌─────────────────────────────────┐│
│  │ Email address                    ││
│  └─────────────────────────────────┘│
│           [ Continue ]               │
└─────────────────────────────────────┘
```

```
หน้าจอที่ 2: Create a password
┌─────────────────────────────────────┐
│  Password                            │
│  ┌─────────────────────────────────┐│
│  │ ••••••••••••                     ││
│  └─────────────────────────────────┘│
│  ต้องมีอย่างน้อย 15 ตัวอักษร          │
│  หรือ 8 ตัวอักษรที่ผสมตัวเลข+อักษรเล็ก │
│           [ Continue ]               │
└─────────────────────────────────────┘
```

```
หน้าจอที่ 3: Enter a username
┌─────────────────────────────────────┐
│  Username                            │
│  ┌─────────────────────────────────┐│
│  │ your-username                    ││
│  └─────────────────────────────────┘│
│  ✓ Username is available             │
│           [ Continue ]               │
└─────────────────────────────────────┘
```

```
หน้าจอที่ 4: Preferences
┌─────────────────────────────────────┐
│  Would you like to receive product   │
│  updates and announcements via email?│
│  ( ) Yes    ( ) No                   │
│           [ Continue ]               │
└─────────────────────────────────────┘
```

3. ยืนยันตัวตนว่าไม่ใช่ bot ด้วย **puzzle แบบภาพ** (เลื่อนต่อชิ้นส่วนภาพให้ตรงกัน หรือ visual challenge อื่น ๆ ที่ GitHub สุ่มมาให้)
4. GitHub จะส่ง**รหัสยืนยัน (verification code) 6 หลัก**ไปที่อีเมลที่กรอกไว้ ให้เปิดอีเมล คัดลอกรหัส แล้วนำมากรอกในหน้าเว็บภายในเวลาที่กำหนด (ปกติประมาณ 10 นาที ถ้าหมดอายุสามารถกดขอส่งใหม่ได้)

```
หน้ายืนยันอีเมล
┌─────────────────────────────────────┐
│  Enter the code we sent to           │
│  you@example.com                     │
│  ┌─────────────────────────────────┐│
│  │  [ 8 ][ 2 ][ 4 ][ 1 ][ 9 ][ 6 ]   ││
│  └─────────────────────────────────┘│
└─────────────────────────────────────┘
```

5. หลังยืนยันสำเร็จ ระบบอาจถามคำถามสำรวจสั้น ๆ (เช่น คุณเป็นนักเรียน/มืออาชีพ/งานอดิเรก, สนใจใช้งานด้านไหน) — **สามารถกด Skip ได้** ถ้าไม่อยากตอบ ไม่มีผลต่อการใช้งาน
6. ระบบจะพาเข้าสู่หน้า Dashboard ของบัญชีใหม่ทันที พร้อมข้อความต้อนรับและคำแนะนำเบื้องต้น (onboarding tour) ซึ่งสามารถปิดข้ามได้เช่นกัน

> **หมายเหตุสำคัญ:** ตั้งแต่ปี 2021 เป็นต้นมา GitHub **ยกเลิกการใช้ password ในการ push โค้ดผ่าน HTTPS แล้ว** ต้องใช้ Personal Access Token หรือ SSH key แทน (เราจะเรียนเรื่องนี้ใน Step 154 และ Part 03/Part 12 ที่ผ่านมาสำหรับ SSH) — password ใช้ได้แค่สำหรับ **login เข้าเว็บไซต์** เท่านั้น

### 151.3 การเลือก Username ที่ดีและเป็นมืออาชีพ — เรื่องสำคัญที่สุดของ Step นี้

Username ของคุณจะกลายเป็นส่วนหนึ่งของ URL ถาวร (`github.com/username`) ที่จะปรากฏใน:

- ลิงก์โปรไฟล์ที่คุณแปะใน resume, LinkedIn, นามบัตร
- ที่อยู่อีเมลของทุก commit ที่คุณทำผ่านหน้าเว็บ GitHub (`username@users.noreply.github.com`)
- URL ของทุก repository ที่คุณสร้าง (`github.com/username/repo-name`)
- ชื่อที่แสดงในทุก Pull Request, Issue, Comment ที่คุณเคยเขียนไว้ในโลก Open Source

**กฎทางเทคนิคของ Username ใน GitHub:**

| กฎ | รายละเอียด |
|---|---|
| ความยาว | สูงสุด 39 ตัวอักษร |
| ตัวอักษรที่ใช้ได้ | ตัวอักษรภาษาอังกฤษ (a-z, A-Z), ตัวเลข (0-9), และเครื่องหมายขีด (`-`) เท่านั้น |
| ห้ามขึ้นต้น/ลงท้ายด้วยขีด | เช่น `-john` หรือ `john-` ใช้ไม่ได้ |
| ห้ามมีขีดติดกัน | เช่น `john--doe` ใช้ไม่ได้ |
| ห้ามมีช่องว่างหรืออักขระพิเศษ | ไม่รองรับ `_`, `.`, `@`, ภาษาไทย หรือ emoji |
| Case-insensitive | `JohnDoe` กับ `johndoe` ถือเป็นชื่อเดียวกัน ชนกันไม่ได้ |

**หลักการเลือก Username ให้ดูเป็นมืออาชีพในระยะยาว:**

1. **ใช้ชื่อจริงหรือใกล้เคียงชื่อจริงถ้าเป็นไปได้** — เช่น `sarawut-k`, `sarawutkanchana`, `skanchana` แทนที่จะเป็นชื่อเล่นแฟนตาซี เพราะ recruiter และเพื่อนร่วมงานในอนาคตจะค้นหาคุณด้วยชื่อจริง
2. **หลีกเลี่ยงตัวเลขวันเกิดหรือตัวเลขสุ่มต่อท้าย** เช่น `john2001`, `coder99887` — ดูไม่เป็นมืออาชีพและจำยาก
3. **หลีกเลี่ยงชื่อที่มีความหมายตลกขบขันหรือดูเป็นวัยรุ่นเกินไป** เช่น `xXDarkSlayerXx`, `noobmaster69` — บัญชีนี้จะติดตัวคุณไปหลายปี รวมถึงตอนสมัครงาน
4. **ให้สั้นและจำง่าย** เพื่อให้พิมพ์ URL ได้สะดวก และดูดีเวลาแปะในนามบัตรหรือ resume
5. **เช็คว่า Username เดียวกันว่างบนแพลตฟอร์มอื่นด้วยหรือไม่** เช่น Twitter/X, LinkedIn, npm, Docker Hub — ถ้าเป็นไปได้ควรใช้ชื่อเดียวกันทุกแพลตฟอร์มเพื่อสร้าง personal brand ที่จดจำง่าย
6. **คิดในระยะยาว** — อย่าตั้งชื่อผูกกับบริษัทปัจจุบันหรือโปรเจกต์เดียว (เช่น `acmecorp-dev`) เพราะเมื่อเปลี่ยนงานชื่อจะดูไม่เข้ากับสถานการณ์

**สิ่งที่ควรรู้เกี่ยวกับการเปลี่ยน Username ภายหลัง:**

GitHub อนุญาตให้เปลี่ยน username ได้ภายหลังใน **Settings → Account → Change username** แต่มีข้อควรระวัง:

- ลิงก์เก่าที่ชี้ไปยัง repository ของคุณ (`github.com/old-username/repo`) จะ **redirect** ไปยังชื่อใหม่โดยอัตโนมัติในระยะหนึ่ง แต่ไม่ได้รับประกันตลอดไป โดยเฉพาะถ้ามีคนอื่นมาลงทะเบียนชื่อเดิมซ้ำภายหลัง
- Commit history เก่าที่ผูกกับอีเมล `old-username@users.noreply.github.com` จะยังแสดงผลถูกต้องเพราะ GitHub เก็บ mapping ไว้ แต่ URL โปรไฟล์แบบเก่าอาจเสี่ยงมีปัญหา
- **สรุป: เปลี่ยนได้ แต่ควรคิดให้ดีตั้งแต่แรกเพื่อลดความยุ่งยากในอนาคต**

---

## Step 152: ตั้งค่า Profile ให้ดูเป็นมืออาชีพ

หน้าโปรไฟล์ (`github.com/username`) คือหน้าแรกที่คนอื่นเห็นเมื่อค้นหาคุณ เข้าไปที่ **Settings → Public profile** เพื่อเริ่มตั้งค่า

### 152.1 Profile Picture (Avatar)

```
Settings → Public profile → Profile Picture
┌───────────────────────┐
│   [   รูปโปรไฟล์   ]   │
│   Edit  |  Remove      │
└───────────────────────┘
```

- รองรับไฟล์ JPG, PNG, GIF (ภาพเคลื่อนไหวจะเล่นบนหน้าโปรไฟล์)
- แนะนำขนาดต้นฉบับอย่างน้อย 460×460 พิกเซล ระบบจะ crop เป็นวงกลม/สี่เหลี่ยมมุมโค้งอัตโนมัติ
- **คำแนะนำมืออาชีพ:** ใช้รูปหน้าตรงที่เห็นใบหน้าชัดเจน แสงสว่างเพียงพอ พื้นหลังไม่รก หลีกเลี่ยงรูปการ์ตูน โลโก้ หรือรูปกลุ่ม — โปรไฟล์ที่มีรูปจริงน่าเชื่อถือกว่าอย่างมีนัยสำคัญในสายตาผู้ว่าจ้างและผู้ร่วมงาน

### 152.2 ข้อมูลพื้นฐาน (Name, Bio, Pronouns)

| ช่อง | คำแนะนำ |
|---|---|
| **Name** | ใส่ชื่อ-นามสกุลจริง หรือชื่อที่ใช้ในวงการทำงาน จะแสดงคู่กับ username เสมอ |
| **Bio** | ข้อความสั้นไม่เกิน 160 ตัวอักษร บอกว่าคุณเป็นใคร/ทำอะไร เช่น `Backend Engineer | Go & PostgreSQL | Building open-source dev tools` |
| **Pronouns** | ตัวเลือกไม่บังคับ เช่น he/him, she/her, they/them — แสดงข้าง ๆ ชื่อในหน้าโปรไฟล์ |
| **Company** | ชื่อบริษัทที่ทำงานปัจจุบัน ใส่ `@` นำหน้าชื่อ org บน GitHub ได้ เช่น `@your-company` จะเชื่อมโยงเป็นลิงก์อัตโนมัติ |
| **Location** | เมือง/ประเทศ เช่น `Bangkok, Thailand` — ช่วยให้ recruiter ท้องถิ่นหาเจอง่ายขึ้น |

### 152.3 Social Links

ในส่วน **Social accounts** สามารถเพิ่มลิงก์ได้สูงสุด 4 ลิงก์ รองรับทั้งลิงก์เว็บไซต์ส่วนตัวและโซเชียลที่ GitHub รู้จักโดยอัตโนมัติ (จะแสดงไอคอนของแพลตฟอร์มนั้น ๆ):

- เว็บไซต์/พอร์ตโฟลิโอส่วนตัว
- LinkedIn
- X (Twitter)
- Mastodon
- Bluesky
- YouTube
- Twitch
- Facebook, Instagram

การใส่ลิงก์เหล่านี้ช่วยให้คนที่เจอโปรไฟล์คุณครั้งแรกสามารถติดตามหรือติดต่อคุณผ่านช่องทางอื่นได้ทันที

### 152.4 Pinned Repositories

ใต้ bio บนหน้าโปรไฟล์ สามารถ **ปักหมุด (pin)** repository เด่น ๆ ได้สูงสุด **6 อัน** โดยกดปุ่ม **Customize your pins** บนหน้าโปรไฟล์ตัวเอง

```
หน้าโปรไฟล์ของคุณ
┌─────────────────────────────────────────┐
│  Pinned                    Customize pins │
│  ┌───────────────┐ ┌───────────────┐      │
│  │ repo-a         │ │ repo-b         │     │
│  │ คำอธิบายสั้น ๆ │ │ คำอธิบายสั้น ๆ │     │
│  │ ★ 12  TypeScript│ │ ★ 5  Python    │     │
│  └───────────────┘ └───────────────┘      │
└─────────────────────────────────────────┘
```

ควรเลือก repository ที่แสดงฝีมือดีที่สุด มี README ที่เขียนดี มีโค้ดสะอาด ไม่ใช่โปรเจกต์ทดลองที่ทำค้างไว้

### 152.5 Profile README — README พิเศษที่แสดงบนหน้าโปรไฟล์

นี่คือฟีเจอร์ที่ทำให้โปรไฟล์ GitHub ดูมืออาชีพขึ้นมากที่สุด GitHub อนุญาตให้คุณสร้าง repository พิเศษชื่อ **เดียวกันกับ username** ของคุณ แล้วไฟล์ `README.md` ในนั้นจะถูกนำไปแสดงเป็นเนื้อหาด้านบนของหน้าโปรไฟล์โดยอัตโนมัติ

**ขั้นตอนการสร้าง Profile README:**

1. ไปที่ **github.com/new** เพื่อสร้าง repository ใหม่
2. ตั้งชื่อ repository ให้ **ตรงกับ username ของคุณเป๊ะ ๆ** (ตัวพิมพ์เล็ก-ใหญ่ไม่สำคัญ) เช่นถ้า username คือ `sarawut-k` ต้องตั้งชื่อ repo ว่า `sarawut-k`
3. GitHub จะแสดงข้อความพิเศษทันทีว่า:

```
sarawut-k/sarawut-k  ✨ 
This is a special repository. Its README.md will
appear on your public profile.
```

4. เลือก **Public** (ต้องเป็น public เท่านั้นถึงจะแสดงผล) และติ๊ก **Add a README file**
5. กด **Create repository**
6. แก้ไข `README.md` ด้วย Markdown ตามต้องการ

**ตัวอย่างเนื้อหา Profile README พื้นฐาน:**

```markdown
### สวัสดีครับ ผม Sarawut 👋

- 🔭 กำลังทำงานเกี่ยวกับ Backend Systems ด้วย Go และ PostgreSQL
- 🌱 กำลังเรียนรู้ Kubernetes และ Distributed Systems
- 💬 ถามผมได้เรื่อง Git, CI/CD, และ Database Performance
- 📫 ติดต่อได้ที่: sarawut@example.com
- ⚡ Fun fact: เขียนโค้ดครั้งแรกตอนอายุ 15 ปี

### 🛠️ Tech Stack
![Go](https://img.shields.io/badge/-Go-00ADD8?style=flat&logo=go&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/-PostgreSQL-336791?style=flat&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/-Docker-2496ED?style=flat&logo=docker&logoColor=white)
```

**เทคนิคขั้นสูงที่นิยมใช้กัน (จะยังไม่ลงมือทำในหลักสูตรตอนนี้ แต่ควรรู้ว่ามีอยู่):**

- **GitHub Stats Card** — บริการภายนอกอย่าง `github-readme-stats` สร้างภาพสถิติ (จำนวน commit, ภาษาที่ใช้บ่อย) แบบ dynamic เป็นรูปภาพฝังใน README
- **GitHub Actions ที่รันอัตโนมัติ** — เช่นดึงบทความ blog ล่าสุด หรือ Spotify ที่กำลังฟังมาแสดงในโปรไฟล์แบบ real-time (เราจะเรียน GitHub Actions แบบเต็มในเฟส 7)
- **Badges และ Shields.io** — ป้ายสถานะสวยงามต่าง ๆ

> **ข้อควรระวัง:** อย่าใส่ข้อมูลเยอะเกินจำเป็นจนดูรก โปรไฟล์ README ที่ดีควรกระชับ อ่านง่าย และสะท้อนตัวตนจริงของคุณ ไม่ใช่การก็อปปี้ template คนอื่นมาทั้งดุ้นโดยไม่แก้ไข

---

## Step 153: ตั้งค่าความปลอดภัย 2FA (Two-Factor Authentication)

### 153.1 ทำไม 2FA ถึงสำคัญมาก

บัญชี GitHub ของคุณไม่ได้เก็บแค่โค้ด — มันคือ **กุญแจเข้าถึงระบบ production ของบริษัท**, **สิทธิ์ push โค้ดเข้า package ที่คนทั่วโลกใช้งาน (เช่น npm, PyPI ที่เชื่อมกับ GitHub Actions)**, และ **ประวัติการทำงานทั้งหมดของคุณ** หากบัญชีถูกขโมย (account takeover) ผลกระทบอาจร้ายแรงถึงขั้น**ห่วงโซ่อุปทานซอฟต์แวร์ (supply chain attack)** — มีเหตุการณ์จริงหลายครั้งที่แฮกเกอร์แฮกบัญชี maintainer แล้วฝัง malware ลงใน package ที่มีคนใช้นับล้าน

ด้วยเหตุนี้ GitHub จึงได้เริ่มบังคับให้ผู้ใช้งานที่มีส่วนร่วมในโค้ด (contributors, maintainers) จำนวนมาก **ต้องเปิดใช้งาน 2FA** เป็นขั้นตอนมาตรฐานตั้งแต่ปี 2023 เป็นต้นมา และ GitHub ก็แนะนำให้ผู้ใช้ทุกคนเปิดไว้เสมอไม่ว่าจะถูกบังคับหรือไม่

### 153.2 วิธีเปิดใช้งาน 2FA

ไปที่ **Settings → Password and authentication**

```
Settings → Password and authentication
┌─────────────────────────────────────────────┐
│  Two-factor authentication                    │
│  Status: ⚠ Disabled                           │
│                                               │
│         [ Enable two-factor authentication ] │
└─────────────────────────────────────────────┘
```

กด **Enable two-factor authentication** จากนั้นเลือกวิธียืนยันตัวตนที่สอง (นอกเหนือจากรหัสผ่าน):

| วิธี | รายละเอียด | คำแนะนำ |
|---|---|---|
| **Authenticator app (TOTP)** | ใช้แอปเช่น Google Authenticator, Authy, 1Password สแกน QR code แล้วสร้างรหัส 6 หลักที่เปลี่ยนทุก 30 วินาที | **แนะนำที่สุด** ปลอดภัยสูง ใช้งานได้แม้ไม่มีสัญญาณมือถือ |
| **Security keys (WebAuthn / Passkeys)** | ใช้อุปกรณ์ physical เช่น YubiKey หรือ Passkey ในตัวเครื่อง (Face ID, Windows Hello) | ปลอดภัยสูงสุด ป้องกัน phishing ได้ดีที่สุด แนะนำสำหรับบัญชีที่สำคัญมาก |
| **SMS (ข้อความมือถือ)** | รับรหัสผ่านทาง SMS | ใช้งานง่ายแต่**มีความเสี่ยงจาก SIM-swap attack** ควรใช้เป็นตัวเลือกสำรอง ไม่ใช่วิธีหลัก |

**ขั้นตอนการตั้งค่าด้วย Authenticator App (แนะนำสำหรับผู้เริ่มต้น):**

1. เลือก **Set up using an app**
2. หน้าจอแสดง QR Code:

```
┌───────────────────────────┐
│  Scan this QR code with   │
│  your authenticator app    │
│                             │
│    ▓▓▓▓ ░░░░ ▓▓▓▓          │
│    ░░░░ ▓▓▓▓ ░░░░          │
│    ▓▓▓▓ ░░░░ ▓▓▓▓          │
│                             │
│  Can't scan? Enter this    │
│  code manually: ABCD 1234  │
└───────────────────────────┘
```

3. เปิดแอป Authenticator บนมือถือ สแกน QR code (หรือกรอกรหัสด้วยตัวเองถ้าสแกนไม่ได้)
4. แอปจะสร้างรหัส 6 หลักที่เปลี่ยนทุก 30 วินาที นำมากรอกในหน้าเว็บ GitHub เพื่อยืนยันว่าตั้งค่าถูกต้อง
5. กด **Verify**

### 153.3 Recovery Codes — สิ่งที่ห้ามลืมเด็ดขาด

หลังเปิด 2FA สำเร็จ GitHub จะแสดง **Recovery codes** ชุดหนึ่ง (ปกติ 16 รหัส) ที่ใช้กู้คืนบัญชีได้ในกรณีที่ทำมือถือหาย หรือเข้าถึง authenticator app ไม่ได้

```
┌─────────────────────────────────────────┐
│  Save your recovery codes                 │
│  1a2b3c4d   2e3f4g5h   3i4j5k6l           │
│  4m5n6o7p   5q6r7s8t   6u7v8w9x           │
│  ...                                      │
│                                           │
│  [ Download ]  [ Print ]  [ Copy ]        │
└─────────────────────────────────────────┘
```

**สิ่งที่ต้องทำทันที:**

- **ดาวน์โหลดหรือพิมพ์เก็บไว้ในที่ปลอดภัย** เช่น password manager หรือที่เก็บเอกสารสำคัญ (ไม่ใช่บันทึกไว้ในโน้ตบนมือถือเครื่องเดียวกับ authenticator app)
- แต่ละรหัสใช้ได้เพียง**ครั้งเดียว**
- ถ้าทำ recovery codes หายและเข้าบัญชีไม่ได้พร้อมกัน **การกู้คืนบัญชีจาก GitHub Support จะยากและใช้เวลานานมาก** — บางกรณีกู้คืนไม่ได้เลยถ้าพิสูจน์ตัวตนไม่ผ่าน

### 153.4 Passkeys — ทางเลือกสมัยใหม่ที่ GitHub รองรับ

นอกจาก TOTP แบบดั้งเดิม GitHub ยังรองรับ **Passkeys** ซึ่งเป็นมาตรฐานยืนยันตัวตนไร้รหัสผ่านที่ใช้ biometric ของอุปกรณ์ (Face ID, Touch ID, Windows Hello) หรือ hardware security key ตั้งค่าได้ในหน้าเดียวกันผ่านเมนู **Passkeys → Add a passkey** ซึ่งสะดวกกว่าเพราะไม่ต้องพิมพ์รหัส 6 หลักทุกครั้ง และป้องกัน phishing ได้ดีกว่า SMS/TOTP ทั่วไป

---

## Step 154: Personal Access Token (PAT) — สร้างและใช้งาน

### 154.1 PAT คืออะไร และทำไมต้องใช้

ตั้งแต่เดือนสิงหาคม 2021 GitHub **ยกเลิกการยืนยันตัวตนด้วย password ธรรมดาสำหรับการ push/pull ผ่าน HTTPS** ไปแล้ว หากคุณเชื่อมต่อ remote repository ด้วย URL แบบ `https://github.com/...` (ไม่ใช่ `git@github.com:...` แบบ SSH ที่เราเรียนไปแล้วใน Part ก่อนหน้า) คุณจะต้องใช้ **Personal Access Token (PAT)** แทนรหัสผ่านเมื่อ Git ถามหา credential

PAT คือ**รหัสสุ่มยาว** ที่ใช้แทนตัวคุณในการยืนยันตัวตนกับ API หรือ Git operation โดยที่**ไม่ต้องเปิดเผยรหัสผ่านจริง** และสามารถ**จำกัดสิทธิ์การเข้าถึง**ได้ละเอียดกว่ารหัสผ่านมาก เช่น อนุญาตให้ token นี้ทำได้แค่ push โค้ด แต่ห้ามลบ repository เป็นต้น

### 154.2 Classic Token vs Fine-grained Token

GitHub มี PAT อยู่ 2 แบบ ที่หน้า **Settings → Developer settings → Personal access tokens**

| หัวข้อ | Classic Token | Fine-grained Token |
|---|---|---|
| การกำหนดสิทธิ์ | กำหนดเป็น **scope กว้าง ๆ** เช่น `repo` (เข้าถึงได้ทุก repo ที่เป็นเจ้าของ), `workflow`, `read:org` | กำหนด**สิทธิ์ละเอียดรายสิทธิ์** เช่น "Contents: Read and write", "Pull requests: Read only" |
| ขอบเขต Repository | ใช้ได้กับ**ทุก repository**ที่คุณมีสิทธิ์เข้าถึงตาม scope ที่เลือก | เลือกได้ว่าจะให้เข้าถึง**เฉพาะ repository ที่ระบุ**หรือทุก repo ของบัญชี |
| วันหมดอายุ | เลือกได้ว่าจะ "No expiration" (ไม่มีวันหมดอายุ) ได้ — **ความเสี่ยงสูงถ้าลืม** | **บังคับต้องตั้งวันหมดอายุเสมอ** (สูงสุด 1 ปี) ปลอดภัยกว่า |
| ต้องขออนุมัติจาก Organization ไหม | โดยปกติไม่ต้อง (เว้นแต่ org ตั้งค่า restrict ไว้) | Organization สามารถบังคับให้ต้อง**อนุมัติ token ก่อนใช้งานกับ repo ของ org** ได้ |
| สถานะปัจจุบัน | ยังใช้งานได้ปกติ แต่ GitHub แนะนำให้เปลี่ยนไปใช้ fine-grained | เป็นมาตรฐานที่ GitHub แนะนำสำหรับการใช้งานใหม่ ๆ |

**คำแนะนำ:** สำหรับผู้เริ่มต้นและงานทั่วไป ให้เริ่มจาก **Fine-grained token** เพราะปลอดภัยกว่ามาก จำกัดขอบเขตความเสียหายได้ดีกว่าหาก token รั่วไหล ส่วน Classic token เหมาะกับกรณีที่ต้องการ scope กว้างและใช้กับหลาย repository จำนวนมากพร้อมกันในเครื่องมือเก่า ๆ ที่ยังไม่รองรับ fine-grained

### 154.3 วิธีสร้าง Fine-grained Personal Access Token

1. ไปที่ **Settings → Developer settings → Personal access tokens → Fine-grained tokens**
2. กด **Generate new token**
3. กรอกแบบฟอร์ม:

```
Generate new token (fine-grained)
┌─────────────────────────────────────────────┐
│ Token name:  [ my-laptop-git-cli          ]  │
│ Expiration:  [ 90 days              ▼ ]      │
│ Description: [ ใช้สำหรับ push จากเครื่อง ]     │
│                                               │
│ Resource owner: [ your-username      ▼ ]     │
│                                               │
│ Repository access:                            │
│  ( ) Public repositories (read-only)          │
│  ( ) All repositories                         │
│  (•) Only select repositories                 │
│      [ my-portfolio-project        ▼ ]        │
│                                               │
│ Permissions:                                  │
│  Contents:        [ Read and write  ▼ ]       │
│  Pull requests:   [ Read and write  ▼ ]       │
│  Metadata:        [ Read-only (auto) ▼ ]      │
│                                               │
│              [ Generate token ]               │
└─────────────────────────────────────────────┘
```

4. กด **Generate token** — ระบบจะแสดง token **เพียงครั้งเดียวเท่านั้น** ต้องคัดลอกเก็บไว้ทันที (เช่น ใน password manager) เพราะปิดหน้าไปแล้วจะดูซ้ำไม่ได้อีก
5. Token จะมีรูปแบบ `github_pat_11ABCDEFG0123456789_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### 154.4 วิธีใช้งาน PAT แทน Password

เมื่อ push ผ่าน HTTPS แล้ว Git ถามหา username/password ให้ใช้:

```bash
git push origin main
Username for 'https://github.com': your-username
Password for 'https://your-username@github.com': <วาง PAT ตรงนี้แทนรหัสผ่าน>
```

เพื่อไม่ต้องกรอกซ้ำทุกครั้ง แนะนำให้ใช้ **Credential Manager** ที่เคยตั้งค่าไปแล้วใน Part 02 (Git Credential Manager บน Windows, Keychain บน macOS, `libsecret`/`pass` บน Linux) ซึ่งจะจดจำ token ไว้ให้อัตโนมัติหลังกรอกครั้งแรก

นอกจากใช้กับ `git push`/`git pull` แล้ว PAT ยังใช้ได้กับ:

- **GitHub REST API / GraphQL API** — ใส่ใน HTTP header `Authorization: Bearer <token>`
- **GitHub CLI (`gh`)** — สามารถเลือกใช้ PAT แทนการ login ผ่านเบราว์เซอร์ได้ (จะกล่าวใน Step 157)
- **เครื่องมือภายนอกที่เชื่อมต่อ GitHub** เช่น CI/CD server ภายนอก, IDE plugin

### 154.5 แนวปฏิบัติด้านความปลอดภัยของ PAT

1. **ตั้งชื่อ token ให้สื่อความหมาย** เช่น `laptop-office-2026`, `ci-deploy-script` เพื่อให้รู้ทันทีว่า token ไหนใช้ทำอะไร เวลาต้องมา revoke ภายหลัง
2. **ให้สิทธิ์เท่าที่จำเป็นเท่านั้น (Principle of Least Privilege)** — ถ้าแค่ต้องการ push โค้ด ไม่ต้องให้สิทธิ์ `delete_repo` หรือ `admin:org`
3. **ตั้งวันหมดอายุเสมอ** แม้จะเป็น classic token ที่เลือก "No expiration" ได้ก็ตาม
4. **ห้ามใส่ token ลงในโค้ดที่ commit เข้า repository เด็ดขาด** แม้เป็น private repo ก็ตาม (GitHub มีระบบ Secret Scanning ที่จะแจ้งเตือนและ revoke token อัตโนมัติถ้าตรวจพบรั่วไหลในบาง repo)
5. **Revoke ทันทีที่สงสัยว่ารั่วไหล** ทำได้ที่หน้า token list โดยกด **Delete**
6. **ทบทวนรายการ token เป็นระยะ** ที่ **Settings → Developer settings → Personal access tokens** ลบ token เก่าที่ไม่ได้ใช้แล้วออก

---

## Step 155: GitHub Notification Settings — จัดการแจ้งเตือนไม่ให้ล้น Inbox

เมื่อเริ่มมีส่วนร่วมกับ repository หลายที่ (โดยเฉพาะเมื่อเข้าเฟส 4 ที่จะทำงานเป็นทีมจริง) อีเมลแจ้งเตือนจาก GitHub อาจท่วม inbox ได้ง่ายมาก Step นี้จะสอนวิธีตั้งค่าให้ได้รับเฉพาะสิ่งที่จำเป็น

### 155.1 ภาพรวมช่องทางการแจ้งเตือน

ไปที่ **Settings → Notifications**

GitHub ส่งการแจ้งเตือนผ่าน 2 ช่องทางหลัก:

| ช่องทาง | คำอธิบาย |
|---|---|
| **Web (Notifications inbox)** | กระดิ่งแจ้งเตือนมุมขวาบนของเว็บ GitHub และหน้า `github.com/notifications` — เป็นศูนย์รวมทุกการแจ้งเตือนที่ดูย้อนหลังได้ |
| **Email** | ส่งอีเมลไปยังที่อยู่ที่กำหนดไว้ — ควบคุมได้ละเอียดว่าจะให้ส่งเรื่องอะไรบ้าง |

### 155.2 ตั้งค่าว่าจะได้รับแจ้งเตือนเรื่องอะไรบ้าง

ส่วน **Notification settings** แบ่งเป็นหมวดสำคัญ:

```
Settings → Notifications
┌─────────────────────────────────────────────┐
│ Default notification email                   │
│  your-email@example.com          [ Edit ]     │
│                                               │
│ Automatically watch repositories              │
│  ☑ When you push to a repository...          │
│                                               │
│ Participating and @mentions                   │
│  ☑ Email   ☑ Web                              │
│                                               │
│ Watching                                      │
│  ☑ Email   ☑ Web                              │
│                                               │
│ Actions (GitHub Actions workflows)            │
│  ☐ Email   ☑ Web                              │
│  Only notify for failed workflows             │
│                                               │
│ Dependabot alerts                             │
│  ☑ Email   ☑ Web                              │
└─────────────────────────────────────────────┘
```

- **Participating and @mentions** — แจ้งเตือนเมื่อมีคน mention คุณโดยตรง หรือคุณมีส่วนร่วมในเธรดนั้น (เช่นเคย comment ไปแล้ว) — **แนะนำเปิดทั้ง Email และ Web เสมอ** เพราะเป็นเรื่องที่ต้องตอบสนองจริง ๆ
- **Watching** — แจ้งเตือนทุกความเคลื่อนไหวของ repository ที่คุณกด Watch ไว้ (ไม่ใช่แค่ที่ mention คุณ) — เป็นตัวการหลักที่ทำให้ inbox ล้น ถ้า watch repo ใหญ่ที่มีคนทำงานเยอะ
- **Actions** — แจ้งเตือนผลการรัน CI/CD — แนะนำตั้งเป็น **"Only notify for failed workflows"** เพื่อไม่ให้อีเมลท่วมทุกครั้งที่ build ผ่าน

### 155.3 การจัดการ Watch ต่อ Repository

แต่ละ repository มีปุ่ม **Watch** ที่มุมขวาบน ให้เลือกระดับการแจ้งเตือนได้ละเอียด:

```
┌──────────────────────────────────┐
│  👁 Watch  ▾                       │
├──────────────────────────────────┤
│ ( ) Participating and @mentions   │
│     รับแจ้งเตือนเฉพาะที่เกี่ยวกับคุณ │
│                                    │
│ (•) All Activity                  │
│     รับทุกการแจ้งเตือนของ repo นี้  │
│                                    │
│ ( ) Custom                        │
│     เลือกเฉพาะ Issues, PRs,       │
│     Releases, Discussions ที่ต้องการ│
│                                    │
│ ( ) Ignore                        │
│     ไม่รับแจ้งเตือนเลย (ยกเว้นถูก @mention)│
└──────────────────────────────────┘
```

**คำแนะนำการจัดการ:**

- Repository ของทีมที่คุณทำงานประจำ → ตั้งเป็น **Custom** แล้วเลือกเฉพาะ Pull Requests และ Issues
- Repository Open Source ขนาดใหญ่ที่แค่ติดตามเฉย ๆ ไม่ได้มีส่วนร่วม → ตั้งเป็น **Participating and @mentions** หรือ **Ignore**
- Repository ส่วนตัวที่ทำคนเดียว → ปกติไม่จำเป็นต้อง watch เพิ่มเพราะเป็นเจ้าของอยู่แล้ว

### 155.4 การใช้ Notifications Inbox อย่างมีประสิทธิภาพ

ที่หน้า **github.com/notifications** มี keyboard shortcut และปุ่มจัดการที่ช่วยเคลียร์ inbox ได้เร็ว:

| การกระทำ | ปุ่ม/Shortcut | ผลลัพธ์ |
|---|---|---|
| Mark as done | `y` หรือปุ่ม checkmark | ทำเครื่องหมายว่าอ่าน/จัดการแล้ว ออกจาก inbox |
| Save for later | ไอคอน bookmark | เก็บไว้ดูภายหลังโดยไม่ปิดแจ้งเตือน |
| Unsubscribe | ไอคอน "no bell" | เลิกติดตามเธรดนั้นโดยเฉพาะ ไม่ใช่ทั้ง repo |
| Mark all as done | ปุ่มด้านบน list | เคลียร์ทั้งหมดในมุมมองปัจจุบัน |

### 155.5 Custom Routing สำหรับ Organization (แนะนำผิวเผิน)

ถ้าคุณอยู่ในหลาย Organization สามารถตั้งค่าให้อีเมลแจ้งเตือนของแต่ละ org ไปคนละที่อยู่อีเมลได้ ที่ **Settings → Notifications → Custom routing** — เช่น แจ้งเตือนจากงานบริษัทไปอีเมลงาน ส่วนแจ้งเตือนจาก personal project ไปอีเมลส่วนตัว ช่วยแยก inbox ได้ชัดเจนขึ้นมาก

---

## Step 156: ภาพรวม GitHub Plans และ GitHub Student Developer Pack

### 156.1 เปรียบเทียบแผนบริการหลัก

| แผน | เหมาะกับ | จุดเด่นหลัก |
|---|---|---|
| **Free** | บุคคลทั่วไป, นักเรียน, โปรเจกต์ส่วนตัว, Open Source | Public/Private repo ไม่จำกัด, GitHub Actions 2,000 นาที/เดือน, Codespaces จำกัดชั่วโมง, Copilot รุ่นฟรีแบบจำกัดการใช้งาน |
| **Pro** | นักพัฒนาเดี่ยวที่จริงจัง, ฟรีแลนซ์ | เพิ่มโควตา GitHub Actions/Codespaces, ดู insight เชิงลึกของ repo ตัวเอง, รวม GitHub Copilot Pro, ปลดล็อก merge queue บางส่วน |
| **Team** | ทีมเล็ก-กลางในบริษัท | จัดการสิทธิ์แบบละเอียดผ่าน Team, Protected branches ขั้นสูง, Draft PR, Code owners, ต้องจ่ายรายหัวสมาชิก |
| **Enterprise** | องค์กรขนาดใหญ่ | SSO/SAML, Audit log ละเอียด, GitHub Advanced Security (secret scanning ขั้นสูง, code scanning), Support ระดับองค์กร, ตั้งค่า compliance ได้ละเอียด, ราคาแบบ custom/ต่อรอง |

**หมายเหตุสำคัญ:** ทุกแผนรวมถึง Free รองรับทั้ง **Public และ Private repository แบบไม่จำกัดจำนวน** มาตั้งแต่ปี 2019 แล้ว ความแตกต่างหลักระหว่างแผนคือ**โควตาการใช้ทรัพยากร** (Actions minutes, Codespaces hours, storage), **ฟีเจอร์การจัดการทีม/องค์กร**, และ**ระดับความปลอดภัย/compliance** ไม่ใช่เรื่องจำนวน repository เหมือนสมัยก่อนปี 2019

### 156.2 GitHub Student Developer Pack

สำหรับผู้ที่กำลังศึกษาอยู่ (มัธยม/มหาวิทยาลัย) GitHub มีโปรแกรมพิเศษชื่อ **GitHub Student Developer Pack** ที่ให้สิทธิประโยชน์จำนวนมากฟรี:

**สิ่งที่ได้รับ:**

- **GitHub Pro ฟรีตลอดช่วงที่ยังเป็นนักเรียน/นักศึกษา** (ปกติต้องเสียเงินรายเดือน)
- ส่วนลดหรือเครดิตฟรีจากพาร์ทเนอร์จำนวนมาก เช่น cloud hosting, domain name ฟรี 1 ปี, เครื่องมือออกแบบ, database service, บริการ API ต่าง ๆ ที่พันธมิตรมอบให้เฉพาะนักศึกษา
- บางช่วงเวลามีสิทธิ์เข้าถึง **GitHub Copilot ฟรี** ผ่านสถานะนักศึกษา

**ขั้นตอนการสมัคร:**

1. ไปที่ **https://education.github.com/pack**
2. กด **Get student benefits**
3. ยืนยันตัวตนความเป็นนักเรียน/นักศึกษาด้วยวิธีใดวิธีหนึ่ง:
   - อีเมลของสถาบันการศึกษา (`.ac.th`, `.edu` เป็นต้น) — ถ้ามีมักอนุมัติเร็วที่สุด
   - อัปโหลดเอกสารยืนยันสถานะนักศึกษา เช่น บัตรนักศึกษา, ใบลงทะเบียนเรียนภาคการศึกษาปัจจุบัน
4. รอการตรวจสอบ (อาจใช้เวลาไม่กี่นาทีถึงหลายวันแล้วแต่กรณี)
5. เมื่อได้รับอนุมัติ สิทธิประโยชน์ต่าง ๆ จะปรากฏในบัญชีโดยอัตโนมัติ

> เอกสารยืนยันไม่จำเป็นต้องเป็นภาษาอังกฤษ ระบบรองรับเอกสารหลายภาษารวมถึงภาษาไทย แต่ควรมีข้อมูลชื่อ, สถาบัน, และวันหมดอายุของสถานะนักศึกษาที่ชัดเจน

### 156.3 GitHub Copilot — โน้ตสั้น ๆ ที่ควรรู้ ณ จุดนี้

Copilot (ผู้ช่วย AI เขียนโค้ด) แยกเป็นแผนของตัวเอง (Free / Pro / Pro+ / Business / Enterprise) ซึ่งรวมมาให้แล้วในบางแผนของ GitHub (เช่น Pro) และมีระดับฟรีแบบจำกัดการใช้งานสำหรับบัญชี Free ด้วยเช่นกัน เรื่องนี้จะไม่ลงรายละเอียดในหลักสูตรนี้เพราะเป็นเครื่องมือ AI แยกออกไป ไม่ใช่แกนกลางของการเรียน Git/GitHub

---

## Step 157: ติดตั้งและ Login GitHub CLI (`gh`)

### 157.1 GitHub CLI คืออะไร

**GitHub CLI** (คำสั่ง `gh`) คือเครื่องมือ command-line อย่างเป็นทางการของ GitHub ที่ทำให้สามารถจัดการ repository, Pull Request, Issue, Release และอื่น ๆ ได้โดย**ไม่ต้องเปิดเบราว์เซอร์เลย** ทำงานร่วมกับคำสั่ง `git` ปกติได้อย่างกลมกลืน (`gh` ไม่ได้มาแทนที่ `git` แต่เป็นเครื่องมือเสริมที่คุยกับ GitHub API โดยเฉพาะ)

เราจะใช้ `gh` แบบเจาะลึกในหลาย Part ของเฟส 3 ถัดไป (สร้าง PR, review, จัดการ issue จาก terminal) แต่ Step นี้จะสอนแค่การติดตั้งและ login ให้พร้อมใช้งานก่อน

### 157.2 การติดตั้ง

| ระบบปฏิบัติการ | คำสั่งติดตั้ง |
|---|---|
| **macOS** (ผ่าน Homebrew) | `brew install gh` |
| **Windows** (ผ่าน winget) | `winget install --id GitHub.cli` |
| **Windows** (ผ่าน Chocolatey) | `choco install gh` |
| **Ubuntu/Debian** | `sudo apt install gh` (ต้องเพิ่ม GitHub CLI apt repository ก่อนในบางเวอร์ชัน) |
| **Fedora/RHEL** | `sudo dnf install gh` |
| **Arch Linux** | `sudo pacman -S github-cli` |

ตรวจสอบว่าติดตั้งสำเร็จ:

```bash
gh --version
```

```
gh version 2.62.0 (2026-01-15)
https://github.com/cli/cli/releases/tag/v2.62.0
```

### 157.3 การ Login ด้วย `gh auth login`

```bash
gh auth login
```

คำสั่งนี้จะเปิดโหมด interactive ถามคำถามทีละข้อ:

```
? What account do you want to log into?
  > GitHub.com
    GitHub Enterprise Server

? What is your preferred protocol for Git operations on this host?
  > HTTPS
    SSH

? How would you like to authenticate GitHub CLI?
  > Login with a web browser
    Paste an authentication token

! First copy your one-time code: A1B2-C3D4
Press Enter to open github.com in your browser...
```

**อธิบายแต่ละขั้นตอน:**

1. **What account do you want to log into?** — เลือก `GitHub.com` สำหรับบัญชีทั่วไป (เลือก `GitHub Enterprise Server` เฉพาะกรณีองค์กรมี GitHub ติดตั้งเองแบบ self-hosted)
2. **Preferred protocol** — เลือก **HTTPS** ถ้ายังไม่เคยตั้งค่า SSH key หรือเลือก **SSH** ถ้าตั้งค่า SSH key ไว้แล้วตั้งแต่ Part ก่อนหน้า
3. **How would you like to authenticate** — มี 2 ทาง:
   - **Login with a web browser** (แนะนำ, ง่ายที่สุด) — ระบบสร้าง one-time code ให้ก๊อปปี้ เปิดเบราว์เซอร์ไปที่ `github.com/login/device` แล้ววางโค้ดเพื่อยืนยัน
   - **Paste an authentication token** — ใช้ PAT ที่สร้างไว้จาก Step 154 วางแทน (เหมาะกับสภาพแวดล้อมที่ไม่มีเบราว์เซอร์ เช่น server headless)

4. เมื่อยืนยันในเบราว์เซอร์เสร็จ terminal จะแสดง:

```
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as your-username
```

### 157.4 ตรวจสอบสถานะการ Login

```bash
gh auth status
```

```
github.com
  ✓ Logged in to github.com account your-username (keyring)
  - Active account: true
  - Git operations protocol: https
  - Token: gho_************************************
  - Token scopes: 'gist', 'read:org', 'repo', 'workflow'
```

### 157.5 คำสั่งพื้นฐานที่ควรรู้จักไว้ก่อน (จะเรียนเจาะลึกใน Part ถัดไป)

```bash
gh repo list          # แสดงรายการ repository ของคุณ
gh repo view          # ดูรายละเอียด repository ปัจจุบัน (ต้องอยู่ในโฟลเดอร์ repo)
gh pr list            # แสดงรายการ Pull Request
gh issue list         # แสดงรายการ Issue
gh browse             # เปิดหน้า repo ปัจจุบันบนเบราว์เซอร์
```

ไม่ต้องจำคำสั่งเหล่านี้ตอนนี้ แค่รู้ว่ามีอยู่และบัญชีของคุณ login พร้อมใช้งานแล้ว — เราจะกลับมาใช้งานจริงแบบละเอียดในหลาย Part ถัดจากนี้

---

## Step 158: แนะนำ GitHub Organization เบื้องต้น

### 158.1 Organization คืออะไร ต่างจากบัญชีส่วนตัวอย่างไร

**Personal account** (บัญชีที่คุณเพิ่งสมัครใน Step 151) เหมาะกับการเป็นเจ้าของ repository ส่วนตัวคนเดียว ส่วน **Organization** คือ**บัญชีประเภทหนึ่งที่เป็นตัวแทนของทีม บริษัท หรือกลุ่ม Open Source** ที่มีสมาชิกหลายคนร่วมกันดูแล repository

| คุณสมบัติ | Personal Account | Organization |
|---|---|---|
| เจ้าของ | คนคนเดียว | กลุ่มคน จัดการสิทธิ์แบบ Team ได้ |
| การจัดการสิทธิ์ | ให้สิทธิ์ collaborator ทีละคนต่อ repo | สร้าง **Team** แล้วกำหนดสิทธิ์เป็นกลุ่มได้ (เช่น Team "Backend" เข้าถึง repo กลุ่ม backend ทั้งหมด) |
| Billing | ผูกกับบัตรของบุคคล | แยก billing เป็นขององค์กรต่างหาก |
| ความเหมาะสม | โปรเจกต์ส่วนตัว, portfolio | บริษัท, ทีม, โปรเจกต์ Open Source ขนาดใหญ่ |

### 158.2 วิธีสร้าง Organization

1. ไปที่ **https://github.com/account/organizations/new** หรือกดไอคอนโปรไฟล์มุมขวาบน → **Your organizations** → **New organization**
2. เลือกแผนราคา (มักเริ่มที่ **Free** ก่อนได้เสมอ แล้วอัปเกรดภายหลังได้)
3. กรอกข้อมูล:

```
Set up your organization
┌─────────────────────────────────────────┐
│ Organization account name                 │
│  [ my-awesome-team                     ]  │
│                                           │
│ Contact email                             │
│  [ team@example.com                    ]  │
│                                           │
│ This organization belongs to              │
│  (•) My personal account                  │
│  ( ) A business or institution             │
│                                           │
│              [ Next ]                     │
└─────────────────────────────────────────┘
```

4. ระบบอาจถามจำนวนสมาชิกโดยประมาณ และเชิญสมาชิกเบื้องต้น (สามารถข้ามไปเชิญทีหลังได้)

### 158.3 การเชิญสมาชิกเข้า Organization

ที่หน้า organization ไปที่แท็บ **People → Invite member**

```
Invite a member
┌─────────────────────────────────────┐
│  Search by username or email          │
│  ┌─────────────────────────────────┐│
│  │ colleague-username                ││
│  └─────────────────────────────────┘│
│  Role: (•) Member   ( ) Owner        │
│              [ Send invitation ]      │
└─────────────────────────────────────┘
```

- **Member** — สิทธิ์พื้นฐาน เข้าถึง repository ตามที่ Team กำหนดให้เท่านั้น
- **Owner** — สิทธิ์เต็มในการจัดการ organization ทั้งหมด รวมถึงการเงิน การลบ org และการจัดการสมาชิกคนอื่น

ผู้ถูกเชิญจะได้รับอีเมล/แจ้งเตือนในเว็บให้กด **Accept invitation** ก่อนจึงจะเข้าร่วมได้จริง

### 158.4 หมายเหตุ

Step นี้เป็นเพียงการแนะนำภาพรวมแบบผิวเผินเท่านั้น เรื่อง **Team, Role แบบละเอียด (Owner/Member/Billing manager), การจัดการสิทธิ์ระดับ repository ผ่าน Team, และ Organization policy** จะถูกสอนแบบเจาะลึกในเฟสหลัง ๆ ของหลักสูตร เมื่อถึงเวลาที่ต้องทำงานเป็นทีมจริงจัง ตอนนี้ขอให้แค่รู้จักว่ามันคืออะไรและมีหน้าตาประมาณไหน

---

## Step 159: Keyboard Shortcuts และ Command Palette ของ GitHub

การใช้ GitHub ผ่านคีย์บอร์ดแทนการคลิกเมาส์ตลอดเวลาจะช่วยประหยัดเวลาได้มากในระยะยาว โดยเฉพาะเมื่อคุณต้องสลับไปมาระหว่าง repository, PR, และ Issue จำนวนมากทุกวัน

### 159.1 เปิดหน้าต่างช่วยเหลือ Keyboard Shortcuts

กดปุ่ม **`?`** (Shift + /) ที่ไหนก็ได้บนเว็บ GitHub (ยกเว้นขณะกำลังพิมพ์ในช่อง text) เพื่อเปิดหน้าต่างแสดงรายการ shortcut ทั้งหมดที่ใช้ได้ในหน้านั้น ๆ

```
Keyboard shortcuts
┌─────────────────────────────────────────┐
│ Site wide shortcuts                       │
│  s หรือ /     โฟกัสไปที่ช่องค้นหา          │
│  g n          ไปที่หน้า Notifications      │
│  g d          ไปที่ Dashboard              │
│  Ctrl+K / ⌘K  เปิด Command Palette         │
│                                           │
│ Repositories                              │
│  g c          ไปที่แท็บ Code               │
│  g i          ไปที่แท็บ Issues             │
│  g p          ไปที่แท็บ Pull requests      │
│  g w          ไปที่แท็บ Actions            │
│  t            เปิดช่องค้นหาไฟล์แบบเร็ว (File Finder)│
└─────────────────────────────────────────┘
```

### 159.2 Command Palette (`Ctrl+K` / `Cmd+K`)

**Command Palette** เป็นฟีเจอร์ที่ GitHub เปิดตัวเพื่อให้เข้าถึงทุกฟังก์ชันได้จากจุดเดียวโดยไม่ต้องคลิกเมนูหลายชั้น กดปุ่ม **`Ctrl+K`** (Windows/Linux) หรือ **`Cmd+K`** (macOS) จากไหนก็ได้บนเว็บ GitHub

```
┌─────────────────────────────────────────┐
│ 🔍  Type a command or search...           │
├─────────────────────────────────────────┤
│  Go to repository my-portfolio            │
│  Go to notifications                      │
│  Create new repository                    │
│  Search issues and pull requests          │
│  Toggle theme (light/dark)                │
│  View profile                             │
└─────────────────────────────────────────┘
```

Command Palette ทำได้หลายอย่าง เช่น:

- **นำทางไปยัง repository/organization/user ใด ๆ อย่างรวดเร็ว** โดยพิมพ์ชื่อ
- **สร้าง repository ใหม่** โดยไม่ต้องกดผ่านเมนู
- **ค้นหาโค้ด, issue, PR** ข้ามทั้ง GitHub
- **สลับธีม (Light/Dark/Auto)** ได้ทันทีโดยพิมพ์ `theme`
- **เข้าถึงคำสั่งเฉพาะของ repository ที่กำลังเปิดอยู่** เช่น เปลี่ยน default branch, ไปที่ Settings ของ repo นั้น

### 159.3 Shortcut ที่มีประโยชน์อื่น ๆ ที่ควรจำ

| Shortcut | การทำงาน |
|---|---|
| `g` แล้วตามด้วย `n` | ไปหน้า Notifications |
| `g` แล้วตามด้วย `p` | ไปแท็บ Pull Requests ของ repo ปัจจุบัน |
| `t` | เปิด File Finder เพื่อค้นหาไฟล์ในหน้า repository อย่างเร็ว |
| `y` | แปลง URL ปัจจุบันให้เป็น permalink ที่อ้างอิง commit SHA ที่แน่นอน (ใช้แชร์ลิงก์โค้ดแบบไม่เปลี่ยนแม้ branch จะอัปเดตต่อ) |
| `.` (จุด) | เปิด repository ปัจจุบันใน **github.dev** (VS Code แบบรันบนเว็บ) ทันที |
| `Esc` | ปิด dialog หรือ popup ที่เปิดอยู่ |

การจำ shortcut เหล่านี้ไม่ใช่เรื่องจำเป็นเร่งด่วน แต่จะช่วยให้คุณทำงานเร็วขึ้นอย่างชัดเจนเมื่อใช้ GitHub เป็นประจำทุกวันในเฟสถัดไปของหลักสูตร

---

## Step 160: แบบฝึกหัด — ตั้งค่าบัญชี GitHub ให้ครบพร้อมใช้งานจริง

ถึงเวลาลงมือทำจริงทุกอย่างที่เรียนมาใน Part นี้ ทำตามลำดับต่อไปนี้ทีละข้อ แล้วติ๊ก checklist ท้าย Step เมื่อเสร็จแต่ละข้อ

### 160.1 ภารกิจที่ 1: สมัครบัญชี (ถ้ายังไม่มี)

- [ ] สมัครบัญชีที่ github.com/signup ด้วยอีเมลที่ใช้งานระยะยาวได้จริง
- [ ] เลือก username ที่ผ่านเกณฑ์มืออาชีพตามหลักการใน Step 151 (ลองเช็คว่า username เดียวกันว่างบน X/LinkedIn ด้วยหรือไม่)
- [ ] ยืนยันอีเมลสำเร็จและเข้าสู่ Dashboard ได้

### 160.2 ภารกิจที่ 2: ตกแต่งโปรไฟล์

- [ ] อัปโหลดรูปโปรไฟล์ที่เป็นรูปหน้าจริง ดูเป็นมืออาชีพ
- [ ] เขียน Bio สั้น ๆ ไม่เกิน 160 ตัวอักษร บอกว่าคุณทำอะไร/สนใจอะไร
- [ ] กรอก Location และเพิ่ม Social link อย่างน้อย 1 ลิงก์ (เช่น LinkedIn)
- [ ] สร้าง repository ชื่อเดียวกับ username ของตัวเอง ตั้งเป็น Public พร้อม README.md แล้วเขียนเนื้อหาแนะนำตัวลงไปอย่างน้อย 3 บรรทัด (นี่คือ Profile README)
- [ ] เข้าไปดูหน้า `github.com/username` ของตัวเองอีกครั้งเพื่อยืนยันว่า README แสดงผลถูกต้องแล้ว

### 160.3 ภารกิจที่ 3: ล็อกความปลอดภัย

- [ ] เปิดใช้งาน 2FA ด้วย Authenticator App (หรือ Passkey ถ้าอุปกรณ์รองรับ)
- [ ] ดาวน์โหลดหรือบันทึก Recovery codes ไว้ในที่ปลอดภัยที่ไม่ใช่อุปกรณ์เดียวกับ authenticator app

### 160.4 ภารกิจที่ 4: สร้างและทดสอบ Personal Access Token

- [ ] สร้าง Fine-grained token ชื่อ `my-first-token` กำหนดวันหมดอายุ 90 วัน จำกัดสิทธิ์เฉพาะ repository ที่สร้างไว้ในภารกิจที่ 2 พร้อมสิทธิ์ Contents: Read and write
- [ ] ทดลองใช้ token นี้แทนรหัสผ่านตอน `git push` ไปยัง repository ทดสอบ (ถ้ายังไม่เคย clone repo profile README ของตัวเองมาไว้ในเครื่อง ให้ clone มาก่อน แก้ไขอะไรเล็กน้อย แล้ว push ด้วย token)

### 160.5 ภารกิจที่ 5: ตั้งค่าการแจ้งเตือน

- [ ] เข้า Settings → Notifications แล้วตั้งค่า Actions ให้เป็น "Only notify for failed workflows"
- [ ] ตรวจสอบว่า Participating and @mentions เปิดทั้ง Email และ Web

### 160.6 ภารกิจที่ 6: ติดตั้งเครื่องมือเสริม

- [ ] ติดตั้ง GitHub CLI (`gh`) บนเครื่องของตัวเอง
- [ ] รัน `gh auth login` และ login สำเร็จ ยืนยันด้วย `gh auth status`
- [ ] ลองรันคำสั่ง `gh repo list` เพื่อดูว่าเห็น repository ที่เพิ่งสร้างในภารกิจที่ 2 หรือไม่

### 160.7 ภารกิจที่ 7: สำรวจ Organization และ Command Palette

- [ ] สร้าง Organization ทดลองสักหนึ่งอัน (สามารถลบทิ้งภายหลังได้ถ้าไม่ได้ใช้จริง โดยไปที่ Organization settings → Delete this organization)
- [ ] กด `Ctrl+K`/`Cmd+K` เพื่อเปิด Command Palette แล้วลองพิมพ์ค้นหา repository ของตัวเอง
- [ ] กด `?` เพื่อดูรายการ keyboard shortcut ทั้งหมดอย่างน้อยหนึ่งครั้ง

เมื่อทำครบทุกภารกิจแล้ว บัญชี GitHub ของคุณจะพร้อมใช้งานในระดับมืออาชีพอย่างแท้จริง — มีตัวตนที่น่าเชื่อถือ ปลอดภัยจากการถูกขโมยบัญชี และมีเครื่องมือครบมือสำหรับ Part ถัดไปที่จะเริ่มสร้าง repository จริงบน GitHub

---

## สรุป Part 16

ใน Part นี้เราได้เรียนรู้ว่า:

1. เราเข้าสู่**เฟส 3** ของหลักสูตรอย่างเป็นทางการ — เปลี่ยนจากการใช้ Git คนเดียวในเครื่อง ไปสู่การใช้แพลตฟอร์ม GitHub ร่วมกับผู้อื่น
2. การเลือก **username** ต้องคิดในระยะยาวเหมือนเลือกชื่อแบรนด์ส่วนตัว เพราะจะติดตัวไปทุกที่ที่คุณมีส่วนร่วมบน GitHub
3. **Profile ที่ดี** ประกอบด้วยรูปโปรไฟล์จริง, Bio กระชับ, Social links, Pinned repositories, และ **Profile README** ที่แสดงบนหน้าโปรไฟล์โดยตรง
4. **2FA** เป็นเรื่องที่ต้องเปิดใช้งานเสมอ ไม่ใช่ทางเลือก — และต้องเก็บ Recovery codes ไว้อย่างปลอดภัย
5. **Personal Access Token (PAT)** มาแทนที่ password สำหรับ push ผ่าน HTTPS ตั้งแต่ปี 2021 โดยมี 2 แบบคือ Classic (scope กว้าง) และ Fine-grained (ปลอดภัยกว่า จำกัดสิทธิ์ละเอียดกว่า)
6. การตั้งค่า **Notification** ที่เหมาะสม ช่วยป้องกัน inbox ล้นเมื่อเริ่มทำงานร่วมกับ repository จำนวนมาก
7. **GitHub Plans** มีตั้งแต่ Free ไปจนถึง Enterprise โดยความต่างหลักคือโควตาทรัพยากรและฟีเจอร์การจัดการทีม/ความปลอดภัย ไม่ใช่จำนวน repository และมี **Student Developer Pack** สำหรับผู้ที่ยังศึกษาอยู่
8. **GitHub CLI (`gh`)** คือเครื่องมือ command-line อย่างเป็นทางการที่ทำให้จัดการ GitHub ได้จาก terminal โดยไม่ต้องเปิดเบราว์เซอร์
9. **Organization** คือบัญชีสำหรับทีม/บริษัท ที่จัดการสิทธิ์แบบกลุ่มผ่าน Team ได้ ต่างจากบัญชีส่วนตัว
10. **Command Palette (`Ctrl+K`)** และ **Keyboard shortcut (`?`)** ช่วยให้ทำงานบน GitHub ได้เร็วขึ้นมากในระยะยาว

บัญชี GitHub ของคุณตอนนี้พร้อมใช้งานในระดับมืออาชีพแล้ว — มีตัวตนที่น่าเชื่อถือ ปลอดภัย และมีเครื่องมือครบมือ ใน Part ถัดไปเราจะเริ่มลงมือสร้าง repository จริงบน GitHub และเชื่อมต่อกับเครื่อง local ของคุณเป็นครั้งแรก

**ต่อไป:** [Part 17: สร้าง Repository บน GitHub และเชื่อมกับเครื่อง local](./part-017-สร้าง-repository-บน-github.md)
