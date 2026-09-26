# Part 25: GitHub Pages: การทำเว็บไซต์ฟรีจาก Repository

> **Step ในหลักสูตรนี้:** Step 241–250
> **เฟส:** 3 — GitHub เบื้องต้น
> **เป้าหมายของ Part นี้:** เข้าใจว่า GitHub Pages คืออะไร เปิดใช้งานเป็น และเลือกวิธี deploy ได้ถูกต้อง (จาก branch หรือผ่าน GitHub Actions) รู้จักโครงสร้างเว็บไซต์ static ที่ GitHub Pages ต้องการ ตั้งค่า custom domain ได้ ใช้ Jekyll และ theme สำเร็จรูปเป็นเบื้องต้น เข้าใจความต่างระหว่าง Project site กับ User/Organization site และแก้ปัญหาที่พบบ่อยได้ด้วยตัวเอง ปิดท้ายด้วยการลงมือ deploy เว็บไซต์ portfolio จริงจนเข้าถึงได้ผ่าน URL สาธารณะ

---

## สารบัญของ Part นี้

- Step 241: GitHub Pages คืออะไร ใช้ทำอะไรได้บ้าง
- Step 242: เปิดใช้งาน GitHub Pages จาก repo settings
- Step 243: Deploy จาก branch เทียบกับ deploy ผ่าน GitHub Actions
- Step 244: โครงสร้างเว็บไซต์ static พื้นฐานที่ต้องมี
- Step 245: การตั้งค่า Custom domain สำหรับ GitHub Pages
- Step 246: Jekyll เบื้องต้น — Static Site Generator ในตัวของ GitHub Pages
- Step 247: การใช้ theme สำเร็จรูปของ GitHub Pages (Jekyll Themes)
- Step 248: Project site vs User/Organization site ความแตกต่าง
- Step 249: HTTPS enforcement และการแก้ปัญหาที่พบบ่อย
- Step 250: แบบฝึกหัด — deploy เว็บไซต์ portfolio ขึ้น GitHub Pages จริง

---

## Step 241: GitHub Pages คืออะไร ใช้ทำอะไรได้บ้าง

**GitHub Pages** คือบริการ **host เว็บไซต์แบบ static (Static Site Hosting) ฟรี** ที่ผูกอยู่กับ GitHub repository โดยตรง พูดง่าย ๆ คือ ถ้าคุณมีไฟล์ HTML/CSS/JavaScript อยู่ใน repository ของคุณ GitHub จะ "เสิร์ฟ" ไฟล์เหล่านั้นออกมาเป็นเว็บไซต์จริงที่เข้าถึงได้ผ่านอินเทอร์เน็ต โดยที่คุณไม่ต้องเช่า server, ไม่ต้องซื้อ hosting, และไม่ต้องตั้งค่า infrastructure ใด ๆ เลย

### "Static site" คืออะไร

ก่อนอื่นต้องเข้าใจคำว่า **static site** ให้ชัดเจน:

| ประเภทเว็บไซต์ | ลักษณะการทำงาน | ตัวอย่าง |
|---|---|---|
| **Static site** | ไฟล์ HTML/CSS/JS ถูกส่งไปหา browser ตรง ๆ ไม่มีการประมวลผลฝั่ง server (ไม่มีฐานข้อมูล ไม่มี server-side code ที่รันขณะมีคนเข้าชม) | เว็บ portfolio, เอกสารประกอบโปรเจกต์, blog ที่ generate ไว้ล่วงหน้า, landing page |
| **Dynamic site** | มี server-side code (เช่น PHP, Node.js, Python) ที่รันทุกครั้งที่มีคนเข้าชม มักเชื่อมกับฐานข้อมูล | เว็บ e-commerce, ระบบ login, dashboard ที่ต้องดึงข้อมูล real-time จาก database |

**GitHub Pages รองรับเฉพาะ static site เท่านั้น** — คุณจะรัน PHP, Node.js backend, Python Flask/Django, หรือเชื่อมต่อฐานข้อมูลโดยตรงบน GitHub Pages ไม่ได้ เพราะ GitHub Pages ไม่มี server-side runtime ให้ มันแค่เก็บไฟล์แล้วส่งออกไปเฉย ๆ (เหมือนตู้เก็บไฟล์ที่เปิดให้ทุกคนเข้ามาหยิบไฟล์ไปดูได้)

> ถ้าอยากรู้ว่าเว็บไซต์หนึ่ง ๆ เป็น static หรือ dynamic ให้ถามตัวเองว่า "ถ้าฉันเปิดไฟล์นี้ตรง ๆ ใน browser โดยไม่มี server เลย มันแสดงผลได้ไหม" ถ้าตอบว่าได้ (แค่ HTML/CSS/JS ล้วน ๆ) นั่นคือ static site ที่ GitHub Pages รองรับ

### GitHub Pages ใช้ทำอะไรได้บ้าง

ในทางปฏิบัติ นักพัฒนาและทีมต่าง ๆ ใช้ GitHub Pages ทำสิ่งเหล่านี้เป็นประจำ:

1. **เว็บไซต์ Portfolio ส่วนตัว** — โชว์ผลงาน ประวัติการทำงาน โปรเจกต์ที่เคยทำ (สิ่งที่เราจะลงมือทำจริงใน Step 250)
2. **เอกสารประกอบโปรเจกต์ (Documentation site)** — โปรเจกต์ Open Source จำนวนมากใช้ GitHub Pages ทำเว็บเอกสารประกอบ เช่น เว็บของไลบรารีต่าง ๆ
3. **Landing page ของโปรเจกต์** — หน้าแนะนำโปรเจกต์แบบสวยงามก่อนให้คนไปดู repository จริง
4. **Blog ส่วนตัว** — โดยเฉพาะเมื่อใช้ร่วมกับ Jekyll (ซึ่งเราจะเรียนใน Step 246) ทำให้เขียนบทความด้วย Markdown แล้ว generate เป็นเว็บ blog ได้ทันที
5. **Resume/CV ออนไลน์** — ทำเรซูเม่แบบเว็บที่แชร์ลิงก์ให้ HR ดูได้
6. **หน้า Demo ของโปรเจกต์** — เช่น demo ของ JavaScript library, web component, หรือ prototype ของ UI
7. **หน้าเอกสารของทีม/องค์กร** — องค์กรจำนวนมากใช้ GitHub Pages ทำเว็บไซต์ status page, style guide, หรือ internal wiki แบบง่าย ๆ

### จุดเด่นที่ทำให้ GitHub Pages ได้รับความนิยม

| จุดเด่น | รายละเอียด |
|---|---|
| **ฟรี 100%** | ไม่มีค่าใช้จ่ายทั้งสำหรับ public repository (และ private repository บนแผน GitHub Pro/Team/Enterprise) |
| **ไม่ต้องดูแล server** | GitHub จัดการ infrastructure ให้ทั้งหมด ไม่ต้องกังวลเรื่อง uptime, patching, scaling |
| **Deploy ง่ายมาก** | แค่ push โค้ดขึ้น branch ที่กำหนดไว้ เว็บไซต์ก็อัปเดตอัตโนมัติ |
| **มี HTTPS ให้ฟรี** | GitHub ออกใบรับรอง SSL/TLS ให้อัตโนมัติผ่าน Let's Encrypt |
| **รองรับ Custom Domain** | ใช้โดเมนของตัวเองแทน `username.github.io` ได้ (เรียนใน Step 245) |
| **รองรับ Jekyll ในตัว** | ไม่ต้องติดตั้ง static site generator เองก็ generate เว็บจาก Markdown ได้ (เรียนใน Step 246) |
| **เชื่อมกับ CDN** | GitHub Pages ใช้ Fastly เป็น CDN ทำให้เว็บโหลดเร็วจากทั่วโลก |
| **Version control เต็มรูปแบบ** | เว็บไซต์ของคุณมีประวัติการแก้ไขครบถ้วนเหมือนโค้ดทั่วไป ย้อนกลับเวอร์ชันเก่าได้เสมอ |

### ข้อจำกัดที่ต้องรู้ไว้ก่อน (Usage Limits)

GitHub Pages เป็นบริการฟรี จึงมีข้อจำกัดการใช้งานที่สมเหตุสมผล (GitHub เรียกว่า "soft limits"):

| ข้อจำกัด | ค่า |
|---|---|
| ขนาด repository ที่แนะนำ | ไม่เกิน 1 GB |
| ขนาดเว็บไซต์ที่ published | ไม่เกิน 1 GB |
| Bandwidth ต่อเดือน | ประมาณ 100 GB (เป็นค่าประมาณการ ไม่ใช่ hard limit ที่ตายตัว) |
| จำนวนการ build ต่อชั่วโมง | ประมาณ 10 ครั้ง (ถ้า build ถี่เกินไปอาจถูก throttle) |
| ประเภทเนื้อหา | รองรับเฉพาะ static content เท่านั้น |

ข้อจำกัดเหล่านี้เพียงพอมากสำหรับเว็บไซต์ portfolio, blog ส่วนตัว, หรือเอกสารโปรเจกต์ทั่วไป แทบไม่มีทางที่ผู้เรียนในหลักสูตรนี้จะไปชนข้อจำกัดเหล่านี้ในการใช้งานทั่วไป

---

## Step 242: เปิดใช้งาน GitHub Pages จาก repo settings

มาลงมือเปิดใช้งาน GitHub Pages กันจริง ๆ ขั้นตอนไม่ซับซ้อนเลย

### ขั้นตอนการเปิดใช้งาน

1. เปิด repository บน GitHub ที่คุณต้องการทำเป็นเว็บไซต์
2. คลิกแท็บ **Settings** (อยู่แถบเมนูด้านบนของ repository)
3. เลื่อนเมนูด้านซ้ายลงมาหา **Pages** (อยู่ในหมวด "Code and automation")
4. ในหน้า **GitHub Pages** จะเจอส่วนสำคัญคือ **Build and deployment**
5. ที่ dropdown **Source** ให้เลือกวิธี deploy หนึ่งในสองแบบ:
   - **Deploy from a branch** — ให้ GitHub Pages ดึงไฟล์จาก branch ที่คุณเลือกโดยตรง
   - **GitHub Actions** — ให้ควบคุม build/deploy process เองผ่าน workflow (เรียนละเอียดใน Step 243)

### ถ้าเลือก "Deploy from a branch"

หลังเลือก source เป็น branch ระบบจะให้เลือกต่อ 2 อย่าง:

1. **Branch** — เลือกว่าจะดึงไฟล์จาก branch ไหน เช่น `main`, `master`, หรือ branch พิเศษชื่อ `gh-pages`
2. **Folder** — เลือกว่าจะดึงไฟล์จากโฟลเดอร์ไหนใน branch นั้น มีตัวเลือกให้แค่ 2 แบบเท่านั้น:
   - `/ (root)` — ใช้ไฟล์ที่อยู่ที่ root ของ repository
   - `/docs` — ใช้ไฟล์ที่อยู่ในโฟลเดอร์ `docs/` เท่านั้น

> **ข้อควรรู้:** GitHub Pages **ไม่อนุญาตให้เลือกโฟลเดอร์อื่นนอกจาก root หรือ `/docs`** เท่านั้น ถ้าเว็บไซต์ของคุณอยู่ในโฟลเดอร์ชื่ออื่น เช่น `website/` หรือ `public/` คุณต้องย้ายไฟล์ไปไว้ที่ root หรือ `/docs`, เปลี่ยนชื่อโฟลเดอร์ หรือใช้วิธี deploy ผ่าน GitHub Actions แทน (ซึ่งยืดหยุ่นกว่ามาก)

จากนั้นกด **Save** ระบบจะเริ่มกระบวนการ deploy ให้อัตโนมัติ ซึ่งใช้เวลาประมาณ 1–3 นาที เมื่อเสร็จแล้วด้านบนของหน้า Settings > Pages จะแสดงข้อความ:

```
Your site is live at https://<username>.github.io/<repository-name>/
```

พร้อมปุ่ม **Visit site** ให้กดเข้าไปดูเว็บไซต์ได้ทันที

### ถ้าเลือก "GitHub Actions"

เมื่อเลือก source เป็น GitHub Actions ระบบจะแสดงรายการ **workflow template สำเร็จรูป** ให้เลือกตามเทคโนโลยีที่ repository ของคุณใช้ เช่น:

- **Static HTML** — สำหรับเว็บไซต์ HTML/CSS/JS ธรรมดา
- **Jekyll** — สำหรับโปรเจกต์ที่ใช้ Jekyll
- **Hugo** — สำหรับ static site generator ชื่อ Hugo
- **Next.js**, **Gatsby**, **Astro**, **VuePress** ฯลฯ — สำหรับ framework ยอดนิยมต่าง ๆ

เมื่อเลือก template ระบบจะสร้างไฟล์ workflow (`.github/workflows/xxx.yml`) ให้อัตโนมัติ ซึ่งคุณสามารถแก้ไขปรับแต่งต่อได้ (รายละเอียดเจาะลึกอยู่ใน Step 243)

### ตรวจสอบสถานะการ deploy

ทุกครั้งที่มีการ deploy คุณสามารถดูสถานะได้จาก:

- แท็บ **Actions** ของ repository — จะเห็น workflow run ชื่อ "pages build and deployment" (สำหรับ deploy แบบ branch) หรือชื่อ workflow ที่คุณตั้ง (สำหรับ deploy แบบ Actions)
- หน้า **Settings > Pages** — จะแสดงว่า deployment ล่าสุดสำเร็จหรือไม่ พร้อมเวลาที่ deploy

ถ้า build ล้มเหลว ระบบจะแจ้งเตือนผ่าน email ไปยังบัญชี GitHub ของคุณด้วย พร้อมลิงก์ไปยัง log ที่ระบุสาเหตุของปัญหา (รายละเอียดการแก้ปัญหาอยู่ใน Step 249)

---

## Step 243: Deploy จาก branch เทียบกับ deploy ผ่าน GitHub Actions

นี่คือการตัดสินใจสำคัญที่สุดอย่างหนึ่งเมื่อเริ่มต้นใช้ GitHub Pages มาทำความเข้าใจความแตกต่างของทั้งสองวิธีให้ชัดเจน

### วิธีที่ 1: Deploy from a branch

วิธีนี้เป็นวิธีดั้งเดิมและง่ายที่สุดของ GitHub Pages หลักการคือ **GitHub จะไปดึงไฟล์จาก branch ที่คุณระบุไว้ตรง ๆ แล้วเสิร์ฟออกมาเป็นเว็บไซต์**

รูปแบบที่นิยมใช้กันมี 2 แบบ:

**แบบที่ 1: ใช้ branch `main` (หรือ `master`) ตรง ๆ**

```
repository/
├── index.html        ← หน้าแรกของเว็บไซต์
├── about.html
├── style.css
└── ...
```

เหมาะกับ repository ที่ **เนื้อหาทั้งหมดคือเว็บไซต์อยู่แล้ว** ไม่มีโค้ด source อื่นที่ต้อง build เช่น repository ที่สร้างขึ้นเพื่อเป็นเว็บ portfolio โดยเฉพาะ

**แบบที่ 2: ใช้ branch พิเศษชื่อ `gh-pages`**

เป็น convention ที่ใช้กันมานาน โดยแยก branch สำหรับเก็บ**ผลลัพธ์ที่ build แล้ว**ออกจาก branch หลักที่เก็บ source code

```
main branch:        เก็บ source code, ไฟล์ config, ไฟล์ที่ยังไม่ผ่านการ build
gh-pages branch:     เก็บเฉพาะไฟล์ HTML/CSS/JS ที่ build เสร็จแล้ว พร้อมเสิร์ฟเป็นเว็บ
```

วิธีนี้เหมาะกับโปรเจกต์ที่ต้อง build ก่อน เช่น เขียนด้วย React แล้ว build ออกมาเป็นไฟล์ static, หรือใช้ static site generator ตัวอื่นที่ไม่ใช่ Jekyll โดยทั่วไปนักพัฒนาจะใช้เครื่องมือ เช่น package `gh-pages` (npm) หรือเขียน script เพื่อ build แล้ว push ผลลัพธ์ไปที่ branch `gh-pages` โดยอัตโนมัติ

### วิธีที่ 2: Deploy ผ่าน GitHub Actions

วิธีนี้ให้คุณ **ควบคุมกระบวนการ build และ deploy เองแบบเต็มรูปแบบ** ผ่านไฟล์ workflow ตัวอย่างโครงสร้าง workflow ทั่วไปสำหรับ deploy static site:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install dependencies
        run: npm ci

      - name: Build site
        run: npm run build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

จุดสำคัญของ workflow นี้:

1. `permissions: pages: write, id-token: write` — จำเป็นต้องมี เพราะ Actions ต้องใช้สิทธิ์นี้เพื่อ publish ไปยัง GitHub Pages โดยตรง
2. `actions/upload-pages-artifact` — เก็บผลลัพธ์ที่ build เสร็จแล้ว (เช่นโฟลเดอร์ `dist/` หรือ `_site/`) เป็น artifact
3. `actions/deploy-pages` — นำ artifact ที่อัปโหลดไว้ไป publish เป็นเว็บไซต์จริง

### ตารางเปรียบเทียบ

| แง่มุม | Deploy from a branch | Deploy ผ่าน GitHub Actions |
|---|---|---|
| ความง่ายในการตั้งค่า | ง่ายมาก แค่เลือก branch/folder | ต้องเขียนหรือเลือก workflow file |
| ความยืดหยุ่นของโฟลเดอร์ | จำกัดแค่ root หรือ `/docs` | เลือก path ไหนก็ได้ตามที่ workflow กำหนด |
| รองรับขั้นตอน build ซับซ้อน (React, Vue, Next.js) | ไม่รองรับโดยตรง ต้อง build แยกแล้ว push ผลลัพธ์เข้า branch เอง | รองรับเต็มรูปแบบ — build อะไรก็ได้ก่อน deploy |
| การควบคุม Jekyll plugin ที่ไม่ได้รับอนุญาตปกติ | ถูกจำกัดด้วย whitelist ปลอดภัยของ GitHub Pages | ใช้ plugin ใดก็ได้เพราะ Actions รัน Jekyll เองแบบเต็มรูปแบบ |
| เหมาะกับ | เว็บไซต์ static ง่าย ๆ, Jekyll ธรรมดา, ผู้เริ่มต้น | โปรเจกต์ที่ใช้ framework สมัยใหม่, ทีมที่ต้องการ pipeline ที่ปรับแต่งได้ |
| ความเร็วในการเห็นผล | เร็ว ไม่ต้องเขียน config เพิ่ม | ต้องรอ workflow รันจบก่อน (แต่ปรับแต่งได้ละเอียดกว่า) |

### ควรเลือกวิธีไหน

- **มือใหม่ / เว็บไซต์ HTML ธรรมดา / Jekyll ธรรมดา** → เลือก **Deploy from a branch** เพราะง่ายและเพียงพอ
- **ใช้ React, Vue, Next.js, Astro หรือ framework ที่ต้อง build ด้วย npm/yarn** → เลือก **GitHub Actions** เพราะควบคุมขั้นตอน build ได้เต็มที่
- **ต้องการใช้ Jekyll plugin ที่ไม่อยู่ใน whitelist ปกติของ GitHub Pages** → ต้องใช้ **GitHub Actions** เท่านั้น

สำหรับแบบฝึกหัดใน Part นี้ (Step 250) เราจะใช้วิธี **Deploy from a branch** เพราะเรากำลังทำเว็บไซต์ portfolio แบบ HTML/CSS ธรรมดาที่ไม่ต้อง build ซับซ้อน เหมาะกับผู้เริ่มต้นที่สุด

---

## Step 244: โครงสร้างเว็บไซต์ static พื้นฐานที่ต้องมี

ก่อนจะ deploy เว็บไซต์ขึ้น GitHub Pages ได้สำเร็จ คุณต้องรู้ว่า GitHub Pages คาดหวังโครงสร้างไฟล์แบบไหน

### ไฟล์ที่จำเป็นที่สุด: `index.html`

กฎที่สำคัญที่สุดของ static site คือ **ต้องมีไฟล์ชื่อ `index.html` อยู่ที่ root ของโฟลเดอร์ที่เลือกไว้เป็น source** (ไม่ว่าจะเป็น root ของ repository หรือใน `/docs`)

เหตุผลคือเมื่อมีคนเข้าเว็บผ่าน URL หลัก เช่น `https://username.github.io/repo-name/` โดยไม่ได้ระบุชื่อไฟล์ท้าย URL, web server จะมองหาไฟล์ชื่อ `index.html` เป็นค่าเริ่มต้นเสมอ (เป็นมาตรฐานของเว็บเซิร์ฟเวอร์ทั่วไป ไม่ใช่แค่ GitHub Pages เท่านั้น)

```
repository/  (หรือ docs/)
├── index.html    ← จำเป็นต้องมี! นี่คือหน้าแรกของเว็บไซต์
├── about.html
├── contact.html
├── css/
│   └── style.css
├── js/
│   └── script.js
└── images/
    └── profile.jpg
```

ถ้าไม่มี `index.html` เมื่อเข้า URL หลักจะได้ผลลัพธ์เป็น **404 Not Found** ทันที (ปัญหานี้เจอบ่อยมาก จะพูดถึงอีกครั้งใน Step 249)

### โครงสร้างโฟลเดอร์ที่แนะนำสำหรับเว็บ static ทั่วไป

```
my-portfolio/
├── index.html          # หน้าแรก
├── about.html          # หน้าเกี่ยวกับฉัน
├── projects.html       # หน้าแสดงผลงาน
├── contact.html        # หน้าติดต่อ
├── assets/
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── main.js
│   └── images/
│       ├── profile.jpg
│       └── project-1.png
├── README.md           # อธิบายโปรเจกต์ (ไม่ถูกแสดงเป็นเว็บ แต่แสดงในหน้า repo)
└── .gitignore
```

### ตัวอย่าง `index.html` ขั้นต่ำที่ใช้งานได้จริง

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My Portfolio</title>
  <link rel="stylesheet" href="assets/css/style.css">
</head>
<body>
  <header>
    <h1>สวัสดี ผมชื่อ...</h1>
    <p>นักพัฒนาซอฟต์แวร์</p>
  </header>

  <main>
    <section id="about">
      <h2>เกี่ยวกับฉัน</h2>
      <p>เนื้อหาแนะนำตัว...</p>
    </section>

    <section id="projects">
      <h2>ผลงาน</h2>
      <!-- รายการโปรเจกต์ -->
    </section>
  </main>

  <footer>
    <p>&copy; 2026 My Portfolio</p>
  </footer>

  <script src="assets/js/main.js"></script>
</body>
</html>
```

### ไฟล์พิเศษอีกไฟล์ที่ควรรู้จัก: `.nojekyll`

ตามค่าเริ่มต้น GitHub Pages จะพยายามประมวลผลไฟล์ทุกอันผ่าน **Jekyll** เสมอ (รายละเอียดใน Step 246) ซึ่ง Jekyll มีกฎบางอย่าง เช่น **ไม่ประมวลผลไฟล์หรือโฟลเดอร์ที่ขึ้นต้นด้วยขีดล่าง (`_`)** เช่นโฟลเดอร์ `_next/` ที่ Next.js สร้างขึ้น หรือ `_data/` ที่บาง framework ใช้

ถ้าเว็บไซต์ของคุณ **ไม่ได้ใช้ Jekyll** (เช่น build มาจาก React, Vue หรือเป็น static ธรรมดาที่มีโฟลเดอร์ขึ้นต้นด้วย `_`) ให้สร้างไฟล์เปล่า ๆ ชื่อ `.nojekyll` ไว้ที่ root ของโฟลเดอร์ source เพื่อบอก GitHub Pages ว่า **"ข้ามขั้นตอน Jekyล ไปเลย เสิร์ฟไฟล์ตรง ๆ"**

```bash
# สร้างไฟล์เปล่าไว้ที่ root
touch .nojekyll
git add .nojekyll
git commit -m "docs: add .nojekyll to skip Jekyll processing"
git push
```

### สิ่งที่ควรระวังเรื่อง path และตัวพิมพ์เล็ก-ใหญ่

GitHub Pages รันบน server ที่ **แยกความแตกต่างระหว่างตัวพิมพ์เล็ก-ใหญ่ (case-sensitive)** เหมือน Linux ทั่วไป ต่างจาก Windows ที่มักไม่สนใจเรื่องนี้ ดังนั้น:

- ถ้าไฟล์ชื่อ `Style.css` แต่ใน HTML เขียนอ้างอิงว่า `<link href="style.css">` (ตัว s เล็ก) เว็บไซต์จะโหลด CSS ไม่ขึ้นเมื่อ deploy บน GitHub Pages แม้ว่าตอนทดสอบบนเครื่อง Windows จะทำงานได้ปกติ
- ควรตั้งชื่อไฟล์และโฟลเดอร์ทั้งหมดเป็น **ตัวพิมพ์เล็กล้วน** และใช้ `-` แทนเว้นวรรค เพื่อลดปัญหานี้ตั้งแต่ต้น

---

## Step 245: การตั้งค่า Custom domain สำหรับ GitHub Pages

เมื่อเว็บไซต์ของคุณ deploy สำเร็จแล้ว ที่อยู่เริ่มต้นจะเป็น `https://username.github.io/repo-name/` ซึ่งใช้งานได้ปกติ แต่ถ้าคุณมีโดเมนของตัวเอง เช่น `mywebsite.com` คุณสามารถตั้งค่าให้ GitHub Pages เสิร์ฟเว็บผ่านโดเมนนั้นแทนได้

### ภาพรวมของกระบวนการ

การตั้งค่า Custom Domain ต้องทำ 2 ส่วนพร้อมกัน:

1. **ฝั่ง GitHub** — เพิ่มไฟล์ `CNAME` ในการตั้งค่า Pages หรือใน repository
2. **ฝั่งผู้ให้บริการโดเมน (DNS Provider)** — เพิ่ม DNS record ให้ชี้มาที่ GitHub Pages

ทั้งสองส่วนต้องตรงกัน ไม่งั้นจะเจอ error หรือเว็บเข้าไม่ได้

### ขั้นตอนที่ 1: ตั้งค่าฝั่ง GitHub

1. ไปที่ **Settings > Pages** ของ repository
2. ในช่อง **Custom domain** ให้พิมพ์ชื่อโดเมนของคุณ เช่น `www.mywebsite.com` หรือ `mywebsite.com`
3. กด **Save**

เมื่อกด Save ระบบจะสร้างไฟล์ชื่อ **`CNAME`** (ไม่มีนามสกุล) ไว้ที่ root ของ branch ที่ใช้ deploy โดยอัตโนมัติ ข้างในไฟล์มีแค่บรรทัดเดียว:

```
www.mywebsite.com
```

> หมายเหตุ: คุณสามารถสร้างไฟล์ `CNAME` นี้ด้วยตัวเองผ่าน git ปกติก็ได้ ไม่จำเป็นต้องรอให้ GitHub สร้างให้ผ่านหน้าเว็บ — แค่สร้างไฟล์ `CNAME` ไว้ที่ root แล้ว commit push ขึ้นไปตามปกติ

### ขั้นตอนที่ 2: ตั้งค่า DNS record

ต้องไปตั้งค่าที่ผู้ให้บริการโดเมนของคุณ (เช่น Namecheap, GoDaddy, Cloudflare, Google Domains) โดยมี 2 กรณีหลัก:

**กรณีที่ 1: ใช้ Apex domain (root domain)** เช่น `mywebsite.com` (ไม่มี `www` นำหน้า)

ต้องสร้าง **A record** จำนวน 4 รายการ ชี้ไปที่ IP address ของ GitHub Pages:

| Type | Host | Value |
|---|---|---|
| A | @ | `185.199.108.153` |
| A | @ | `185.199.109.153` |
| A | @ | `185.199.110.153` |
| A | @ | `185.199.111.153` |

(ควรใส่ทั้ง 4 IP นี้เสมอ เพื่อความเสถียรและ redundancy)

**กรณีที่ 2: ใช้ Subdomain** เช่น `www.mywebsite.com` หรือ `blog.mywebsite.com`

ให้สร้าง **CNAME record** เพียงรายการเดียว ชี้ไปที่ `username.github.io`:

| Type | Host | Value |
|---|---|---|
| CNAME | www | `username.github.io` |

### แนวทางที่แนะนำ: ใช้ทั้งสองแบบร่วมกัน

วิธีที่นิยมที่สุดคือตั้งค่า **apex domain ด้วย A record ทั้ง 4 รายการ** และตั้งค่า **`www` เป็น CNAME ชี้ไปที่ apex domain หรือ `username.github.io`** จากนั้นเลือกว่าจะให้โดเมนหลักของเว็บไซต์เป็น apex (`mywebsite.com`) หรือ subdomain (`www.mywebsite.com`) — GitHub Pages จะ redirect อีกฝั่งมาหาฝั่งที่คุณเลือกไว้ในช่อง Custom domain โดยอัตโนมัติ

### ระยะเวลารอผล (DNS Propagation)

การเปลี่ยนแปลง DNS ไม่ได้มีผลทันที อาจใช้เวลาตั้งแต่ **ไม่กี่นาทีจนถึง 24–48 ชั่วโมง** กว่าจะกระจายไปทั่วโลก (เรียกว่า DNS propagation) ระหว่างนี้สามารถตรวจสอบสถานะได้ด้วยคำสั่ง:

```bash
dig mywebsite.com +noall +answer
```

หรือใช้เว็บไซต์ตรวจสอบ DNS propagation ออนไลน์

### ตรวจสอบว่าตั้งค่าถูกต้อง

กลับไปที่ **Settings > Pages** อีกครั้ง หลังจากตั้งค่า DNS แล้ว GitHub จะแสดงเครื่องหมายถูกสีเขียวพร้อมข้อความ **"DNS check successful"** ถ้ายังไม่ผ่านจะแสดงคำแนะนำว่าปัญหาอยู่ตรงไหน (เช่น A record ไม่ครบ หรือ CNAME ยังไม่ถูก propagate)

### ป้องกันปัญหา Domain Takeover

ข้อควรระวังสำคัญ: **อย่าลบ CNAME record ที่ชี้ไปยัง `username.github.io` ทิ้งโดยที่ repository ยังตั้งค่า custom domain นั้นอยู่** เพราะจะเปิดช่องให้เกิดการโจมตีแบบ **Dangling DNS / Domain Takeover** ได้ ถ้าเลิกใช้โดเมนนั้นแล้วให้ไปลบการตั้งค่า Custom domain ออกจาก Settings > Pages ก่อนเสมอ

---

## Step 246: Jekyll เบื้องต้น — Static Site Generator ในตัวของ GitHub Pages

หนึ่งในจุดเด่นที่ทำให้ GitHub Pages พิเศษกว่าบริการ static hosting ทั่วไปคือ **มันรองรับ Jekyll แบบในตัว (built-in)** โดยไม่ต้องติดตั้งหรือตั้งค่าอะไรเพิ่มเติมเลย

### Jekyll คืออะไร

**Jekyll** คือ **Static Site Generator (SSG)** เขียนด้วยภาษา Ruby หน้าที่ของมันคือ **แปลงไฟล์ Markdown (`.md`) และ Layout Template ให้กลายเป็นเว็บไซต์ HTML แบบ static** โดยอัตโนมัติ

พูดง่าย ๆ คือ แทนที่คุณจะต้องเขียน HTML เต็มรูปแบบทุกหน้าเอง คุณสามารถ:

1. เขียนเนื้อหาด้วย **Markdown** (เขียนง่ายกว่า HTML มาก)
2. กำหนด **Layout** (โครงหน้าเว็บ, header, footer, navigation) แค่ครั้งเดียว
3. ให้ Jekyll **ประกอบ Markdown + Layout เข้าด้วยกัน** กลายเป็นหน้า HTML สมบูรณ์โดยอัตโนมัติ

เพราะ GitHub เป็นเจ้าของ Jekyll ในทางปฏิบัติ (ทีม GitHub เป็นผู้พัฒนาและดูแล) มันจึงถูกผสานเข้ากับ GitHub Pages อย่างแนบเนียนที่สุด — **แค่มีไฟล์ Markdown กับ config ที่ถูกต้อง GitHub Pages ก็จะรัน Jekyll build ให้อัตโนมัติทุกครั้งที่ push**

### ไฟล์สำคัญที่สุดของ Jekyll: `_config.yml`

ไฟล์นี้อยู่ที่ root ของโปรเจกต์ ใช้กำหนดการตั้งค่าทั้งหมดของเว็บไซต์ Jekyll:

```yaml
# _config.yml
title: My Portfolio
description: เว็บไซต์ผลงานของฉัน
author: ชื่อของคุณ
baseurl: "/repo-name"      # ต้องตรงกับชื่อ repository ถ้าเป็น Project site
url: "https://username.github.io"

theme: minima              # เลือก theme (รายละเอียดใน Step 247)

markdown: kramdown
permalink: pretty

exclude:
  - README.md
  - Gemfile
  - Gemfile.lock
```

### โครงสร้างโปรเจกต์ Jekyll ทั่วไป

```
my-jekyll-site/
├── _config.yml           # ไฟล์ตั้งค่าหลัก
├── _layouts/              # Template โครงหน้าเว็บ
│   ├── default.html
│   └── post.html
├── _includes/             # ชิ้นส่วน HTML ที่ใช้ซ้ำ เช่น header, footer
│   ├── header.html
│   └── footer.html
├── _posts/                # บทความ blog (ถ้าทำเว็บแบบ blog)
│   └── 2026-01-15-my-first-post.md
├── _sass/                 # ไฟล์ Sass สำหรับจัดสไตล์
├── assets/
│   ├── css/
│   └── images/
├── index.md               # หน้าแรก (เขียนด้วย Markdown ได้เลย)
├── about.md
└── Gemfile                # ระบุ dependency ของ Ruby gems
```

### Front Matter — หัวใจสำคัญของทุกไฟล์ Jekyll

ทุกไฟล์ที่ต้องการให้ Jekyll ประมวลผล (ไม่ว่าจะเป็น `.md` หรือ `.html`) ต้องมี **Front Matter** อยู่บนสุดของไฟล์ คั่นด้วยเครื่องหมาย `---` สามขีด:

```markdown
---
layout: default
title: หน้าแรก
---

# ยินดีต้อนรับสู่เว็บไซต์ของฉัน

นี่คือเนื้อหาที่เขียนด้วย **Markdown** ธรรมดา ๆ
```

Front Matter บอก Jekyll ว่า:

- `layout: default` — ให้ใช้ template จากไฟล์ `_layouts/default.html` มาห่อเนื้อหานี้
- `title: หน้าแรก` — กำหนดตัวแปร `title` ที่สามารถเรียกใช้ในไฟล์ layout ได้ผ่าน `{{ page.title }}`

ไฟล์ที่ **ไม่มี Front Matter** จะถูก Jekyll มองว่าเป็นไฟล์ static ธรรมดา (copy ไปตรง ๆ ไม่ผ่านการประมวลผล template)

### ตัวอย่าง Layout พื้นฐาน

```html
<!-- _layouts/default.html -->
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>{{ page.title }} | {{ site.title }}</title>
</head>
<body>
  {% include header.html %}

  <main>
    {{ content }}
  </main>

  {% include footer.html %}
</body>
</html>
```

`{{ content }}` คือจุดที่ Jekyll จะแทรกเนื้อหาที่แปลงจาก Markdown มาแล้ว ส่วน `{% include ... %}` คือการดึงชิ้นส่วน HTML จากโฟลเดอร์ `_includes/` มาแทรก — นี่คือภาษา template ของ Jekyll ที่ชื่อ **Liquid**

### Whitelist ปลั๊กอินของ GitHub Pages

เมื่อ deploy ด้วยวิธี "Deploy from a branch" (ไม่ใช่ Actions) GitHub Pages จะรัน Jekyll ในโหมด **safe mode** ซึ่งอนุญาตให้ใช้เฉพาะปลั๊กอินที่อยู่ใน whitelist ที่กำหนดไว้ล่วงหน้าเท่านั้น เช่น `jekyll-feed`, `jekyll-seo-tag`, `jekyll-sitemap` เป็นต้น ถ้าต้องการใช้ปลั๊กอินอื่นนอกเหนือจากนี้ ต้องเปลี่ยนไปใช้วิธี deploy ผ่าน **GitHub Actions** แทน (ตามที่กล่าวไว้ใน Step 243) เพราะการรัน Jekyll ผ่าน Actions จะไม่ถูกจำกัดด้วย whitelist นี้

### ทดสอบ Jekyll บนเครื่องตัวเองก่อน push (ทางเลือก)

ถ้าต้องการเห็นผลลัพธ์ก่อน push ขึ้น GitHub จริง สามารถติดตั้ง Jekyll บนเครื่องแล้วรันดูก่อนได้:

```bash
gem install bundler jekyll
bundle exec jekyll serve
# เปิดเบราว์เซอร์ไปที่ http://localhost:4000
```

วิธีนี้ช่วยประหยัดเวลาได้มาก เพราะไม่ต้อง push ทุกครั้งเพื่อดูผลลัพธ์บน GitHub Pages ซึ่งใช้เวลารอ build เสมอ

---

## Step 247: การใช้ theme สำเร็จรูปของ GitHub Pages (Jekyll Themes)

ไม่ต้องเขียน CSS เองตั้งแต่ศูนย์ทุกครั้ง GitHub Pages มี **Jekyll Theme สำเร็จรูป** ให้เลือกใช้ได้ฟรีจำนวนหนึ่ง ซึ่งครอบคลุมทั้งโครงหน้าเว็บ, CSS, และ layout พื้นฐาน

### วิธีใช้ theme แบบง่ายที่สุด: กำหนดใน `_config.yml`

```yaml
# _config.yml
theme: minima
```

`minima` คือ **theme เริ่มต้น (default theme)** ของ Jekyll เอง เน้นความเรียบง่าย เหมาะกับ blog ส่วนตัว

### รายชื่อ Theme ที่ GitHub Pages รองรับอย่างเป็นทางการ

GitHub Pages มี Theme ที่ผ่านการรับรองอย่างเป็นทางการ (supported themes) จำนวนหนึ่ง ซึ่งสามารถใช้งานผ่านการตั้งค่าใน `_config.yml` ได้โดยตรงโดยไม่ต้องติดตั้งอะไรเพิ่ม เช่น:

| ชื่อ theme | ลักษณะเด่น |
|---|---|
| `minima` | เรียบง่าย เหมาะกับ blog (เป็น default) |
| `jekyll-theme-cayman` | ดีไซน์ทันสมัย มี header banner ใหญ่ เหมาะกับ project page |
| `jekyll-theme-minimal` | เรียบและ minimal มาก |
| `jekyll-theme-slate` | โทนสีเข้ม (dark) ดูเป็นทางการ |
| `jekyll-theme-architect` | โทนสีสดใส เหมาะกับ portfolio |
| `jekyll-theme-dinky` | Sidebar navigation ในตัว |
| `jekyll-theme-hacker` | ธีมสไตล์ terminal/hacker เขียว-ดำ |
| `jekyll-theme-leap-day` | สีสันสดใส ทันสมัย |
| `jekyll-theme-merlot` | โทนสีน้ำตาลแดง เรียบหรู |
| `jekyll-theme-midnight` | โทนมืด เหมาะกับ documentation |
| `jekyll-theme-modernist` | มินิมอลสไตล์งาน typography |
| `jekyll-theme-photon` | ทันสมัย ใช้ font สวย |
| `jekyll-theme-tactile` | มี texture พื้นหลัง |
| `jekyll-theme-time-machine` | เน้น timeline layout |

การใช้งานแค่เปลี่ยนค่า `theme:` ใน `_config.yml` เป็นชื่อ theme ที่ต้องการ เช่น:

```yaml
theme: jekyll-theme-cayman
```

จากนั้น push ขึ้น GitHub — Jekyll จะดึง theme มาประกอบให้อัตโนมัติโดยไม่ต้อง copy ไฟล์ CSS มาเองเลย

### เลือก theme ผ่านหน้าเว็บ GitHub โดยตรง

GitHub มีเครื่องมือช่วยเลือก theme แบบ visual ผ่านหน้า **Settings > Pages** เช่นกัน:

1. ไปที่ **Settings > Pages**
2. ถ้า repository มี branch ที่เปิด Pages อยู่แล้ว จะเห็นปุ่ม **Choose a theme** (หรือ Change theme)
3. คลิกเข้าไปจะเห็นตัวอย่าง (preview) ของแต่ละ theme พร้อมเลือกได้ทันที
4. เมื่อเลือกและกด commit ระบบจะเพิ่มบรรทัด `theme: xxx` ลงใน `_config.yml` ให้อัตโนมัติ (หรือสร้างไฟล์ให้ถ้ายังไม่มี)

### การใช้ Theme จากภายนอก (Remote Theme)

นอกจาก theme ทางการที่ระบุไว้ข้างต้น ยังมี Jekyll theme จากชุมชนอีกจำนวนมากที่ทำเป็น Ruby gem หรือ GitHub repository แยกต่างหาก การใช้ theme เหล่านี้ต้องใช้ปลั๊กอิน `jekyll-remote-theme`:

```yaml
# _config.yml
remote_theme: owner/repo-name-of-theme
plugins:
  - jekyll-remote-theme
```

วิธีนี้ทำงานได้ทั้งแบบ deploy from branch (เพราะ `jekyll-remote-theme` อยู่ใน whitelist ของ GitHub Pages) และแบบ GitHub Actions

### Override บางส่วนของ Theme

ถ้าอยากปรับแต่ง theme สำเร็จรูปเล็กน้อยโดยไม่อยากเขียนทั้งเว็บใหม่ สามารถ **สร้างไฟล์ชื่อเดียวกันทับ** โครงสร้างภายในของ theme ได้ เช่น ถ้าอยากแก้ไฟล์ `_layouts/default.html` ของ theme ที่ใช้อยู่ ให้สร้างไฟล์ `_layouts/default.html` ในโปรเจกต์ตัวเองด้วยชื่อเดียวกัน — Jekyll จะให้ความสำคัญกับไฟล์ในโปรเจกต์ก่อนเสมอ (override)

### เพิ่ม CSS ของตัวเองทับ Theme

วิธีที่ง่ายที่สุดในการปรับแต่งเล็กน้อยคือสร้างไฟล์ `assets/css/style.scss`:

```scss
---
---
@import "{{ site.theme }}";

/* เพิ่ม custom style ของคุณเองด้านล่างนี้ */
h1 {
  color: #2c3e50;
}
```

บรรทัด `---` ว่างเปล่า 2 บรรทัดบนสุดคือ Front Matter เปล่า ๆ ที่บอก Jekyll ว่าไฟล์นี้ต้องผ่านการประมวลผล (สำคัญมาก ถ้าไม่มี Front Matter ไฟล์ `.scss` จะไม่ถูกแปลงเป็น CSS)

---

## Step 248: Project site vs User/Organization site ความแตกต่าง

GitHub Pages มีเว็บไซต์อยู่ 2 ประเภทหลัก ซึ่งมีกฎการตั้งชื่อ repository และ URL ที่ได้ต่างกันโดยสิ้นเชิง เรื่องนี้เป็นสิ่งที่มือใหม่สับสนบ่อยมาก

### User site / Organization site

เว็บไซต์ประเภทนี้ผูกกับ **บัญชี GitHub ของคุณโดยตรง** (หรือ Organization) มีกฎสำคัญคือ:

> **repository ต้องตั้งชื่อเป็น `<username>.github.io` เท่านั้น** (สำหรับ Organization ก็ใช้ `<orgname>.github.io`)

ตัวอย่าง: ถ้า username ของคุณคือ `nattakit`, repository ต้องชื่อ **`nattakit.github.io`** เป๊ะ ๆ (ตัวพิมพ์เล็กทั้งหมด) เท่านั้นถึงจะทำงานแบบ User site ได้

**URL ที่ได้:**

```
https://nattakit.github.io/
```

**ลักษณะเด่นของ User/Organization site:**

- แต่ละบัญชี GitHub มี User site ได้ **แค่ 1 เว็บไซต์เท่านั้น** (เพราะชื่อ repo ต้องตรงกับ username เป๊ะ ๆ ซึ่งซ้ำกันไม่ได้)
- source ของ User site (ในกรณี deploy from branch) **ต้องใช้ branch `main` เท่านั้น** เป็นค่าที่ GitHub บังคับ (ไม่สามารถใช้ branch อื่นได้เหมือน Project site)
- เหมาะเป็นเว็บไซต์หลักของตัวเอง เช่น portfolio หลัก, resume online

### Project site

เว็บไซต์ประเภทนี้ผูกกับ **repository เฉพาะโปรเจกต์หนึ่ง ๆ** ไม่ว่าจะตั้งชื่อ repository ว่าอะไรก็ได้ (ยกเว้นรูปแบบ `username.github.io`)

**URL ที่ได้:**

```
https://nattakit.github.io/my-awesome-project/
```

สังเกตว่า URL จะมี **ชื่อ repository ต่อท้ายเสมอ** (`/my-awesome-project/`) เพราะ URL path ถูกใช้เพื่อแยกแยะว่าเป็นเว็บของ repository ไหนใต้บัญชีเดียวกัน

**ลักษณะเด่นของ Project site:**

- บัญชี GitHub เดียวสามารถมี Project site ได้ **หลายเว็บไซต์พร้อมกันไม่จำกัด** (เพราะแต่ละ repository แยกกันคนละ URL path)
- เลือก branch ที่ใช้ deploy ได้อิสระ (เช่น `main`, `gh-pages`, หรือ branch อื่น ๆ)
- เหมาะกับเว็บ documentation ของแต่ละโปรเจกต์, demo page ของแต่ละไลบรารี

### ตารางเปรียบเทียบสรุป

| แง่มุม | User/Organization site | Project site |
|---|---|---|
| ชื่อ repository ที่บังคับ | `username.github.io` เท่านั้น | ชื่อใดก็ได้ |
| จำนวนที่มีได้ต่อบัญชี | 1 เว็บไซต์ | หลายเว็บไซต์ (1 ต่อ repository) |
| รูปแบบ URL | `https://username.github.io/` | `https://username.github.io/repo-name/` |
| Branch ที่ deploy ได้ (แบบ branch) | `main` เท่านั้น | เลือกได้อิสระ เช่น `main`, `gh-pages` |
| เหมาะกับ | เว็บไซต์หลักส่วนตัว/องค์กร | เว็บของแต่ละโปรเจกต์แยกกัน |

### ผลกระทบสำคัญต่อการเขียนโค้ด: `baseurl`

ความแตกต่างของ URL structure มีผลโดยตรงต่อการอ้างอิงไฟล์ (path) ในเว็บไซต์ของคุณ:

- **User site** — URL เริ่มที่ root (`/`) โดยตรง ดังนั้นอ้างอิง path แบบ `/assets/style.css` ได้ตรง ๆ
- **Project site** — URL มี prefix เป็นชื่อ repo (`/repo-name/`) เสมอ ดังนั้นถ้าอ้างอิง path แบบ absolute เช่น `/assets/style.css` จะ **ชี้ผิดที่** เพราะจริง ๆ ต้องเป็น `/repo-name/assets/style.css`

วิธีแก้ปัญหานี้มี 2 แบบ:

1. **ใช้ relative path เสมอ** เช่น `assets/style.css` แทน `/assets/style.css` (แนะนำสำหรับเว็บ HTML ธรรมดา)
2. **ถ้าใช้ Jekyll** ให้ตั้งค่า `baseurl` ใน `_config.yml` แล้วอ้างอิงผ่าน Liquid tag `{{ '/assets/style.css' | relative_url }}` ซึ่งจะเติม baseurl ให้อัตโนมัติตามประเภทเว็บไซต์

```yaml
# _config.yml (สำหรับ Project site)
baseurl: "/my-awesome-project"
url: "https://nattakit.github.io"
```

```html
<link rel="stylesheet" href="{{ '/assets/css/style.css' | relative_url }}">
```

ลืมตั้งค่า `baseurl` ให้ถูกต้อง คือสาเหตุอันดับต้น ๆ ที่ทำให้เว็บ Jekyll ที่ deploy บน Project site แสดงผลแบบไม่มี CSS เลย (หน้าตาเป็น HTML ดิบ ๆ)

---

## Step 249: HTTPS enforcement และการแก้ปัญหาที่พบบ่อย

### HTTPS บน GitHub Pages

GitHub Pages ออกใบรับรอง SSL/TLS ให้ **ฟรีโดยอัตโนมัติ** ผ่าน **Let's Encrypt** สำหรับทั้ง URL แบบ `username.github.io` และ custom domain ที่ตั้งค่าไว้ถูกต้อง

เมื่อไปที่ **Settings > Pages** จะเห็น checkbox ชื่อ **Enforce HTTPS** ซึ่งเมื่อเปิดใช้งาน (เป็นค่าเริ่มต้นสำหรับ URL แบบ `github.io`) จะทำให้:

- ทุก request ที่เข้ามาผ่าน `http://` **ถูก redirect ไปยัง `https://` โดยอัตโนมัติ**
- ป้องกันการดักฟังข้อมูลระหว่างทาง (man-in-the-middle) และเพิ่มความน่าเชื่อถือของเว็บไซต์

> สำหรับ custom domain: checkbox "Enforce HTTPS" อาจใช้เวลาสักครู่ (บางครั้งนานถึงหลายชั่วโมง) หลังตั้งค่า DNS สำเร็จ กว่าที่ GitHub จะออกใบรับรองให้เสร็จและเปิดตัวเลือกนี้ให้กดได้ ถ้ากดไม่ได้ในทันทีให้รอแล้วลองใหม่

### ปัญหาที่พบบ่อยที่สุด #1: 404 Not Found

สาเหตุที่พบบ่อยที่สุดของ 404 เมื่อเข้าเว็บไซต์:

| สาเหตุ | วิธีแก้ |
|---|---|
| ไม่มีไฟล์ `index.html` ที่ root ของโฟลเดอร์ source | สร้าง `index.html` ไว้ที่ root (หรือ `/docs` ถ้าเลือก source เป็น docs) |
| เลือก branch หรือ folder ผิดใน Settings > Pages | ตรวจสอบ Settings > Pages ว่าเลือกตรงกับที่ไฟล์เว็บไซต์อยู่จริง |
| ตัวพิมพ์เล็ก-ใหญ่ของชื่อไฟล์ไม่ตรงกับที่อ้างอิงใน HTML | ตรวจสอบว่าชื่อไฟล์และการอ้างอิงตรงกันทุกตัวอักษรรวมถึงตัวพิมพ์ |
| ยังไม่ได้ push การเปลี่ยนแปลงล่าสุดขึ้น branch ที่ใช้ deploy | เช็คด้วย `git log` และ `git push` ให้แน่ใจว่า commit ล่าสุดอยู่บน remote แล้ว |
| repository เป็น private และไม่มีแผนที่รองรับ private Pages | เปลี่ยน repository เป็น public หรืออัปเกรดเป็นแผน GitHub Pro/Team/Enterprise |
| เพิ่งตั้งค่าเสร็จ ยังไม่ถึงเวลาที่ deploy เสร็จสมบูรณ์ | รอสัก 1–3 นาทีแล้วลองใหม่ ตรวจสอบสถานะที่แท็บ Actions |

### ปัญหาที่พบบ่อยที่สุด #2: Build Failed

เมื่อ build ล้มเหลว GitHub จะส่ง email แจ้งเตือนพร้อมเหตุผลคร่าว ๆ วิธีตรวจสอบรายละเอียดเพิ่มเติม:

1. ไปที่แท็บ **Actions** ของ repository
2. คลิกดู run ล่าสุดที่มีเครื่องหมาย ❌ สีแดง
3. อ่าน log เพื่อดูว่า error เกิดที่ขั้นตอนไหน

สาเหตุที่พบบ่อยของ Build Failed (โดยเฉพาะเมื่อใช้ Jekyll):

- **Front Matter ผิดรูปแบบ** — ลืมปิด `---` หรือมี YAML syntax ผิด เช่น เว้นวรรค indent ไม่ตรงกัน
- **ใช้ plugin ที่ไม่อยู่ใน whitelist** เมื่อ deploy แบบ branch (ต้องเปลี่ยนไปใช้ GitHub Actions ตามที่กล่าวใน Step 246)
- **Liquid syntax ผิดพลาด** เช่น เขียน `{{ page.title }` ขาด `}` ไปหนึ่งตัว
- **เวอร์ชัน Jekyll หรือ gem ไม่ตรงกับที่ GitHub Pages รองรับ** — ควรตรวจสอบเวอร์ชันที่รองรับผ่านเอกสารทางการ

### ปัญหาที่พบบ่อยที่สุด #3: Cache ไม่อัปเดต (เห็นเนื้อหาเก่า)

GitHub Pages ใช้ **CDN (Fastly)** ในการกระจายเนื้อหา ทำให้บางครั้งแม้ deploy สำเร็จแล้ว แต่เปิดเว็บไซต์ยังเห็นเนื้อหาเก่าอยู่ วิธีแก้ไข:

1. **รอสักครู่** — CDN cache มักจะ invalidate ให้อัตโนมัติภายในไม่กี่นาที (บางครั้งนานถึง 10 นาที)
2. **Hard refresh browser** — กด `Ctrl+Shift+R` (Windows/Linux) หรือ `Cmd+Shift+R` (Mac) เพื่อบังคับให้ browser โหลดไฟล์ใหม่ทั้งหมดโดยไม่ใช้ cache ของตัวเอง
3. **เปิดในโหมด Incognito/Private** — เพื่อตัดปัญหา cache ฝั่ง browser ออกไปก่อน
4. **ตรวจสอบว่า commit ล่าสุด deploy สำเร็จจริง** — ดูที่แท็บ Actions ว่า workflow ล่าสุดสถานะเป็นสีเขียว (สำเร็จ) และเวลาตรงกับที่คุณคาดหวัง

### ปัญหาที่พบบ่อยอื่น ๆ

| ปัญหา | สาเหตุที่เป็นไปได้ | วิธีแก้ |
|---|---|---|
| CSS/JS ไม่โหลด | อ้างอิง path แบบ absolute (`/style.css`) บน Project site ที่มี baseurl | เปลี่ยนเป็น relative path หรือใช้ `relative_url` ของ Jekyll |
| รูปภาพไม่แสดง | path ผิด หรือไฟล์ยังไม่ได้ commit ขึ้น repository | ตรวจสอบด้วย `git status` ว่าไฟล์รูปภาพถูก track แล้ว |
| หน้าเว็บว่างเปล่า สีขาวล้วน | ลืมปิด tag HTML หรือ JavaScript error ทำให้เพจพัง | เปิด Developer Console (`F12`) ใน browser เพื่อดู error |
| Custom domain แสดง "Domain does not resolve" | DNS ยังไม่ propagate หรือ record ตั้งค่าผิด | ตรวจสอบด้วย `dig` และรอ propagation ให้ครบ |
| Enforce HTTPS กดไม่ได้ (จาง ๆ) | ใบรับรอง SSL ยังออกไม่เสร็จหลังตั้ง custom domain | รอสักพัก (อาจถึงหลายชั่วโมง) แล้วลองใหม่ |

### เครื่องมือช่วยตรวจสอบสุขภาพของเว็บไซต์

- แท็บ **Actions** — ดู log การ build/deploy ทั้งหมด
- **Settings > Pages** — ดูสถานะ deployment ล่าสุดและ DNS check
- `curl -I https://username.github.io/repo-name/` — ตรวจสอบ HTTP response code จาก terminal โดยตรง (ควรได้ `200 OK`)
- Developer Tools ของ browser (`F12` > แท็บ Network) — ดูว่าไฟล์ไหนโหลดไม่สำเร็จ (สถานะ 404 สีแดง)

---

## Step 250: แบบฝึกหัด — deploy เว็บไซต์ portfolio ขึ้น GitHub Pages จริง

ถึงเวลาลงมือทำจริงแล้ว เป้าหมายของแบบฝึกหัดนี้คือ **นำเว็บไซต์ portfolio ของคุณขึ้น GitHub Pages จนเข้าถึงได้จริงผ่าน URL สาธารณะ**

### เตรียมความพร้อม

ถ้าคุณเคยทำแบบฝึกหัดสร้างเว็บไซต์ portfolio ไว้แล้วจาก Part ก่อนหน้า (เช่นตอนฝึกเขียน HTML/CSS หรือทำ README.md แบบมืออาชีพ) ให้นำไฟล์เหล่านั้นมาใช้ในแบบฝึกหัดนี้ได้เลย แต่ถ้ายังไม่มี ให้สร้างเว็บไซต์ portfolio ง่าย ๆ ขึ้นมาใหม่ตามขั้นตอนด้านล่างนี้

### ขั้นตอนที่ 1: สร้าง Repository

ตัดสินใจก่อนว่าจะทำเป็น **Project site** หรือ **User site**:

- ถ้าต้องการให้เป็นเว็บไซต์ portfolio หลักของคุณตลอดไป แนะนำให้ทำเป็น **User site** โดยสร้าง repository ชื่อ `<your-username>.github.io`
- ถ้าต้องการทดลองก่อน หรือมีเว็บ portfolio หลักอยู่แล้ว ให้ทำเป็น **Project site** โดยตั้งชื่อ repository ตามต้องการ เช่น `my-portfolio`

```bash
mkdir my-portfolio
cd my-portfolio
git init
```

### ขั้นตอนที่ 2: สร้างไฟล์เว็บไซต์

สร้างไฟล์ `index.html` พร้อมเนื้อหาพื้นฐาน:

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Portfolio ของฉัน</title>
  <link rel="stylesheet" href="assets/css/style.css">
</head>
<body>
  <header class="hero">
    <h1>สวัสดี ผมชื่อ [ชื่อของคุณ]</h1>
    <p>นักพัฒนาซอฟต์แวร์ | กำลังเรียนรู้ Git และ GitHub</p>
  </header>

  <main>
    <section id="about">
      <h2>เกี่ยวกับฉัน</h2>
      <p>ผมกำลังเรียนรู้การพัฒนาซอฟต์แวร์ผ่านหลักสูตร Git/GitHub อย่างจริงจัง</p>
    </section>

    <section id="projects">
      <h2>ผลงานของฉัน</h2>
      <div class="project-card">
        <h3>โปรเจกต์ที่ 1</h3>
        <p>คำอธิบายโปรเจกต์สั้น ๆ</p>
        <a href="https://github.com/your-username/project-1">ดูโค้ด</a>
      </div>
    </section>

    <section id="contact">
      <h2>ติดต่อฉัน</h2>
      <p>Email: your-email@example.com</p>
      <p>GitHub: <a href="https://github.com/your-username">@your-username</a></p>
    </section>
  </main>

  <footer>
    <p>&copy; 2026 [ชื่อของคุณ]</p>
  </footer>
</body>
</html>
```

สร้างไฟล์ CSS ที่ `assets/css/style.css`:

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, "Segoe UI", "Sarabun", sans-serif;
  line-height: 1.6;
  color: #2c3e50;
}

.hero {
  background: #2c3e50;
  color: white;
  padding: 4rem 2rem;
  text-align: center;
}

main {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

section {
  margin-bottom: 3rem;
}

.project-card {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 1.5rem;
  margin-top: 1rem;
}

footer {
  text-align: center;
  padding: 2rem;
  color: #888;
}
```

### ขั้นตอนที่ 3: Commit และ Push ขึ้น GitHub

```bash
git add .
git commit -m "feat: initial portfolio website"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

### ขั้นตอนที่ 4: เปิดใช้งาน GitHub Pages

1. เปิด repository บนเว็บ GitHub
2. ไปที่ **Settings > Pages**
3. เลือก **Source: Deploy from a branch**
4. เลือก **Branch: main**, **Folder: / (root)**
5. กด **Save**

### ขั้นตอนที่ 5: รอและตรวจสอบผล

รอประมาณ 1–3 นาที แล้วรีเฟรชหน้า Settings > Pages จะเห็นข้อความ:

```
Your site is live at https://<your-username>.github.io/<repo-name>/
```

คลิก **Visit site** เพื่อเปิดดูเว็บไซต์จริง หรือเปิด URL นั้นในแท็บใหม่ด้วยตัวเอง

### ขั้นตอนที่ 6: ทดสอบวงจรการอัปเดตเว็บไซต์

ทดลองแก้ไขเนื้อหาสักจุดหนึ่ง เช่น เปลี่ยนข้อความในส่วน "เกี่ยวกับฉัน" แล้ว commit และ push อีกครั้ง:

```bash
git add .
git commit -m "docs: update about section"
git push
```

ไปที่แท็บ **Actions** เพื่อดูว่า deployment ใหม่กำลังรันอยู่ (สถานะสีเหลือง/กำลังทำงาน) จนกระทั่งเปลี่ยนเป็นสีเขียว (สำเร็จ) จากนั้นรีเฟรชหน้าเว็บไซต์จริง (อาจต้อง hard refresh ตามที่เรียนใน Step 249) เพื่อดูว่าการเปลี่ยนแปลงแสดงผลแล้ว

### เกณฑ์ความสำเร็จของแบบฝึกหัดนี้

ให้ตรวจสอบว่าคุณทำครบทุกข้อต่อไปนี้:

- [ ] มี repository บน GitHub ที่มีไฟล์เว็บไซต์ static (`index.html` เป็นอย่างน้อย)
- [ ] เปิดใช้งาน GitHub Pages สำเร็จผ่าน Settings > Pages
- [ ] เข้าเว็บไซต์ได้จริงผ่าน URL `https://<username>.github.io/...` และเห็นเนื้อหาที่ถูกต้อง
- [ ] URL เข้าผ่าน HTTPS ได้โดยไม่มี warning ด้านความปลอดภัย
- [ ] ทดสอบ workflow การแก้ไข commit push แล้วเห็นเว็บไซต์อัปเดตจริง
- [ ] (ทางเลือกเพิ่มเติม) ลองตั้งค่า custom domain ถ้ามีโดเมนของตัวเองอยู่แล้ว
- [ ] (ทางเลือกเพิ่มเติม) ลองเปลี่ยนไปใช้ Jekyll theme สำเร็จรูปสักหนึ่งแบบ

### ถ้าติดปัญหา

ให้กลับไปตรวจสอบตามลำดับนี้:

1. ตรวจสอบว่ามีไฟล์ `index.html` อยู่ที่ root ของโฟลเดอร์ที่เลือกเป็น source จริง (Step 244)
2. ตรวจสอบว่า branch/folder ที่เลือกใน Settings > Pages ตรงกับที่ push ไฟล์ไปจริง (Step 242)
3. ดู log ที่แท็บ Actions ว่ามี error อะไรหรือไม่ (Step 249)
4. ตรวจสอบตัวพิมพ์เล็ก-ใหญ่ของชื่อไฟล์ทุกจุดที่อ้างอิงถึงกัน (Step 244, Step 249)

เมื่อทำสำเร็จ คุณจะมีเว็บไซต์ portfolio ที่ใช้งานได้จริง แชร์ลิงก์ให้เพื่อน ครู หรือใส่ไว้ใน resume ได้ทันที — และที่สำคัญที่สุดคือ **ทุกครั้งที่คุณ push การเปลี่ยนแปลงใหม่ เว็บไซต์จะอัปเดตให้อัตโนมัติ** ไม่ต้องอัปโหลดไฟล์ผ่าน FTP หรือทำอะไรซ้ำซ้อนอีกเลย

---

## สรุป Part 25

ใน Part นี้เราได้เรียนรู้ว่า:

1. **GitHub Pages** คือบริการ host เว็บไซต์ static ฟรีที่ผูกกับ repository โดยตรง รองรับเฉพาะ HTML/CSS/JS ไม่มี server-side runtime
2. การเปิดใช้งานทำผ่าน **Settings > Pages** โดยเลือก source เป็น **Deploy from a branch** (ง่าย เหมาะกับเว็บ static ธรรมดา) หรือ **GitHub Actions** (ยืดหยุ่นกว่า เหมาะกับโปรเจกต์ที่ต้อง build ด้วย framework สมัยใหม่)
3. โครงสร้างเว็บไซต์ static ต้องมีไฟล์ **`index.html`** อยู่ที่ root ของโฟลเดอร์ source เสมอ และควรระวังเรื่องตัวพิมพ์เล็ก-ใหญ่ของชื่อไฟล์
4. สามารถตั้งค่า **Custom domain** ได้ด้วยไฟล์ `CNAME` ร่วมกับการตั้งค่า DNS record (A record สำหรับ apex domain, CNAME record สำหรับ subdomain)
5. GitHub Pages รองรับ **Jekyll** ในตัว ทำให้สร้างเว็บไซต์จาก Markdown + Layout ได้โดยไม่ต้องติดตั้งอะไรเพิ่ม ควบคุมผ่านไฟล์ `_config.yml`
6. มี **Jekyll theme สำเร็จรูป** ให้เลือกใช้ฟรีหลายแบบ เปลี่ยนได้ง่ายแค่แก้บรรทัด `theme:` ใน config
7. **Project site** (`username.github.io/repo`) กับ **User/Organization site** (`username.github.io`) มีกฎการตั้งชื่อ repository, จำนวนที่ทำได้, และ URL structure ที่ต่างกันชัดเจน ซึ่งมีผลต่อการอ้างอิง path ในเว็บไซต์ด้วย
8. GitHub Pages ออก **HTTPS ให้ฟรีอัตโนมัติ** ผ่าน Let's Encrypt และมีวิธีแก้ปัญหาที่พบบ่อย เช่น 404, build failed, cache ไม่อัปเดต ที่ควรรู้จักไว้ล่วงหน้า
9. เราได้ลงมือ **deploy เว็บไซต์ portfolio จริง** จนเข้าถึงได้ผ่าน URL สาธารณะ พร้อมทดสอบวงจรการอัปเดตเว็บไซต์ด้วยการ commit และ push

### Checklist ก่อนไป Part 26

- [ ] อธิบายได้ว่า GitHub Pages คืออะไรและรองรับเนื้อหาประเภทไหน
- [ ] เปิดใช้งาน GitHub Pages ผ่าน repo settings ได้ด้วยตัวเอง
- [ ] เข้าใจความต่างระหว่าง deploy from branch กับ deploy ผ่าน GitHub Actions และเลือกใช้ได้เหมาะกับสถานการณ์
- [ ] รู้ว่าเว็บไซต์ static ต้องมี `index.html` ที่ root และระวังเรื่อง case sensitivity
- [ ] ตั้งค่า custom domain ได้ (อย่างน้อยเข้าใจขั้นตอนแม้ยังไม่มีโดเมนจริง)
- [ ] เข้าใจการทำงานพื้นฐานของ Jekyll: `_config.yml`, Front Matter, Layout
- [ ] เปลี่ยน Jekyll theme ได้
- [ ] แยกแยะ Project site กับ User/Organization site ได้ถูกต้อง
- [ ] แก้ปัญหา 404, build failed, cache ไม่อัปเดตได้ด้วยตัวเอง
- [ ] มีเว็บไซต์ portfolio ที่ deploy สำเร็จจริงบน GitHub Pages เข้าถึงได้ผ่าน URL

**ต่อไป:** [Part 26: Labels, Milestones, Issue/PR Templates](./part-026-labels-milestones-templates.md)
