# Part 82: Monorepo vs Polyrepo — การตัดสินใจเชิงสถาปัตยกรรม

> **Step ในหลักสูตรนี้:** Step 811–820
> **เฟส:** 8 — DevOps, Security, Compliance ระดับองค์กร
> **เป้าหมายของ Part นี้:** เข้าใจแนวคิด Monorepo และ Polyrepo อย่างลึกซึ้ง รู้ข้อดีข้อเสียของแต่ละแบบบนพื้นฐานความจริงจากองค์กรระดับโลก รู้จักเครื่องมือที่ทำให้ Monorepo ขนาดใหญ่ยังคงทำงานได้เร็ว เข้าใจแนวทาง Hybrid ที่หลายองค์กรใช้จริง และสามารถใช้ decision framework ในการตัดสินใจเลือกสถาปัตยกรรม repository ที่เหมาะสมกับบริบทขององค์กรตัวเองได้

---

## สารบัญของ Part นี้

- Step 811: Monorepo คืออะไร และใครใช้จริงบ้าง
- Step 812: Polyrepo คืออะไร และใครใช้จริงบ้าง
- Step 813: ข้อดีของ Monorepo
- Step 814: ข้อเสียของ Monorepo
- Step 815: ข้อดีของ Polyrepo
- Step 816: ข้อเสียของ Polyrepo
- Step 817: เครื่องมือรองรับ Monorepo ขนาดใหญ่ (Bazel, Nx, Turborepo, Lerna)
- Step 818: Hybrid Approach — เมื่อองค์กรใช้ทั้งสองแบบผสมกัน
- Step 819: Decision Framework — จะเลือก Monorepo หรือ Polyrepo อย่างไร
- Step 820: แบบฝึกหัด — วิเคราะห์สถานการณ์องค์กรสมมติ 3 แบบ

---

## Step 811: Monorepo คืออะไร และใครใช้จริงบ้าง

ใน **Part 63 Step 627** เราได้เกริ่นไว้แล้วว่าการตัดสินใจระหว่าง Monorepo กับ Polyrepo เป็น "การตัดสินใจเชิงสถาปัตยกรรม" ที่ส่งผลกระทบต่อทั้งองค์กร ไม่ใช่แค่เรื่องทางเทคนิคเล็ก ๆ ใน Part นี้เราจะเจาะลึกเรื่องนี้แบบเต็มรูปแบบ

### นิยามของ Monorepo

**Monorepo (Monolithic Repository)** คือแนวทางการจัดโครงสร้าง source control ที่:

> **โค้ดของทุกโปรเจกต์ ทุกทีม ทุก service ในองค์กร (หรืออย่างน้อยในขอบเขตหนึ่ง เช่น ทั้งบริษัท หรือทั้งกลุ่มผลิตภัณฑ์) ถูกเก็บไว้ใน Git repository เดียวกัน ภายใต้ history เดียวกัน**

ข้อควรระวัง: Monorepo **ไม่ใช่** monolith (สถาปัตยกรรมซอฟต์แวร์แบบก้อนเดียว) สองคำนี้มักถูกสับสนกันบ่อยมาก

- **Monolith** พูดถึง**สถาปัตยกรรมของแอปพลิเคชัน** — โค้ดทั้งหมดถูก deploy เป็นก้อนเดียว รันเป็น process เดียว
- **Monorepo** พูดถึง**สถาปัตยกรรมของ source control** — โค้ดถูกเก็บไว้ใน repository เดียว แต่ **ข้างในสามารถมีหลาย service, หลาย microservice, หลาย application ที่ build และ deploy แยกกันโดยสมบูรณ์ได้**

พูดง่าย ๆ คือ องค์กรที่ใช้สถาปัตยกรรม microservices เต็มรูปแบบ (deploy แยกกันหลายสิบ-หลายร้อย service) ก็ยังสามารถเก็บโค้ดทั้งหมดไว้ใน Monorepo เดียวได้ ไม่ขัดแย้งกันเลย

### โครงสร้างตัวอย่างของ Monorepo

```
company-monorepo/
├── services/
│   ├── auth-service/
│   ├── payment-service/
│   ├── notification-service/
│   └── search-service/
├── frontend/
│   ├── web-app/
│   ├── admin-dashboard/
│   └── mobile-app/
├── libs/
│   ├── shared-ui-components/
│   ├── shared-types/
│   └── logging-utils/
├── infra/
│   ├── terraform/
│   └── k8s-manifests/
├── tools/
│   └── build-scripts/
└── BUILD (หรือ WORKSPACE, nx.json, turbo.json ฯลฯ)
```

ทุกโฟลเดอร์ข้างบนคือส่วนหนึ่งของ **Git repository เดียว** — คนที่ `git clone` จะได้ประวัติของทั้งหมดนี้มาพร้อมกัน

### องค์กรที่ใช้ Monorepo จริงในระดับใหญ่ที่สุดของโลก

| องค์กร | รายละเอียด |
|---|---|
| **Google** | ใช้ระบบชื่อ **Piper** (ไม่ใช่ Git โดยตรง แต่เป็นแนวคิด Monorepo แบบสุดขั้ว) เก็บโค้ดเกือบทั้งหมดของบริษัท (ยกเว้น Android, Chromium บางส่วน) ไว้ใน repository เดียว มีขนาดหลายพันล้านบรรทัด และมีวิศวกรหลายหมื่นคน commit เข้าไปในที่เดียวกันทุกวัน |
| **Meta (Facebook)** | ใช้ Monorepo ขนาดใหญ่มาก โดยเฉพาะสำหรับโค้ด backend หลัก ซึ่งเป็นเหตุผลสำคัญที่ทำให้ Meta ต้องพัฒนา **Mercurial แบบ scale พิเศษ** (และภายหลังก็มีส่วนร่วมในการพัฒนา Sapling ซึ่งเป็น VCS ที่ออกแบบมาเพื่อ Monorepo ขนาดมหึมาโดยเฉพาะ) |
| **Microsoft** | Repository ของ **Windows** (ระบบปฏิบัติการ) เป็น Monorepo ขนาดใหญ่มากที่ใช้ Git จริง ๆ (ไม่ใช่ระบบอื่น) มีไฟล์กว่า 3.5 ล้านไฟล์ ขนาดกว่า 300 GB ซึ่งเป็นแรงผลักดันสำคัญที่ทำให้ Microsoft ต้องพัฒนา **VFS for Git (ปัจจุบันคือ Scalar)** ที่เราได้พูดถึงใน Part 63 |
| **Twitter (ก่อนเปลี่ยนเป็น X)** | ใช้ Monorepo ขนาดใหญ่สำหรับโค้ด backend หลักส่วนใหญ่ |
| **Uber** | ใช้ Monorepo สำหรับโค้ด backend ภาษา Go และ Java เป็นส่วนใหญ่ |
| **Airbnb** | ใช้ Monorepo สำหรับโค้ด JavaScript/TypeScript ฝั่ง frontend เป็นหลัก |

ข้อสังเกตสำคัญ: บริษัทเหล่านี้**ไม่ได้ใช้ Monorepo แบบไร้เดียงสา** (naive) — ทุกบริษัทลงทุนสร้างเครื่องมือพิเศษมหาศาลเพื่อทำให้ Monorepo ขนาดนั้นยังคงทำงานได้เร็ว (ซึ่งเราจะพูดถึงใน Step 817) การใช้ Monorepo แบบตรง ๆ โดยไม่มีเครื่องมือเสริมเหล่านี้ จะพังทันทีเมื่อ scale ถึงระดับหนึ่ง

---

## Step 812: Polyrepo คืออะไร และใครใช้จริงบ้าง

### นิยามของ Polyrepo

**Polyrepo (Multi-repository / Multiple Repositories)** คือแนวทางตรงข้ามกับ Monorepo:

> **แต่ละโปรเจกต์ แต่ละ service แต่ละไลบรารี มี Git repository ของตัวเองแยกต่างหาก โดยสมบูรณ์ พร้อม history, permission, และ lifecycle ของตัวเอง**

นี่คือรูปแบบที่ **เป็นค่าเริ่มต้นตามธรรมชาติของ Git** — เมื่อคุณสร้างโปรเจกต์ใหม่ คุณมักจะ `git init` แยกกันไปเรื่อย ๆ ทีละโปรเจกต์อยู่แล้ว Polyrepo จึงเป็นแนวทางที่นักพัฒนาส่วนใหญ่คุ้นเคยมากกว่าตั้งแต่แรก

### โครงสร้างตัวอย่างของ Polyrepo

```
github.com/company/auth-service          (repo แยก)
github.com/company/payment-service       (repo แยก)
github.com/company/notification-service  (repo แยก)
github.com/company/web-app               (repo แยก)
github.com/company/mobile-app            (repo แยก)
github.com/company/shared-ui-components  (repo แยก, เป็น package ที่ถูก publish)
github.com/company/shared-types          (repo แยก, เป็น package ที่ถูก publish)
github.com/company/terraform-infra       (repo แยก)
```

แต่ละบรรทัดข้างบนคือ Git repository ที่**สมบูรณ์ในตัวเอง** — มี commit history, branch, permission, CI/CD pipeline, release cycle ของตัวเองทั้งหมด ไม่เกี่ยวข้องกัน ถ้าจะให้ `payment-service` ใช้โค้ดจาก `shared-types` ต้อง publish `shared-types` เป็น package (เช่น npm package, Maven artifact, Go module) แล้วดึงมาใช้ผ่านระบบ dependency management ตามปกติ ไม่ใช่ import ไฟล์ตรง ๆ ข้าม repo ได้

### องค์กรที่ใช้ Polyrepo จริงในระดับใหญ่

| องค์กร | รายละเอียด |
|---|---|
| **Netflix** | ใช้สถาปัตยกรรม microservices ที่ขึ้นชื่อที่สุดในโลก (มีหลายร้อย-หลายพัน microservice) และแต่ละ service ส่วนใหญ่มี repository ของตัวเอง เพื่อให้แต่ละทีมสามารถเลือกภาษา เลือก release cycle และดูแล service ของตัวเองได้อย่างอิสระเต็มที่ (แนวคิด "You build it, you run it" ที่ Netflix ใช้ต้องการความเป็นอิสระของ repo ระดับสูง) |
| **Amazon** | ทีมภายในของ Amazon จำนวนมากใช้ Polyrepo สำหรับ microservices ของตัวเอง ตามหลักการ "two-pizza team" ที่แต่ละทีมเล็กดูแล service ของตัวเองอย่างเป็นอิสระ ตั้งแต่โค้ดไปจนถึงการ deploy (แม้ในภาพรวมระดับบริษัท Amazon จะมี repository จำนวนมหาศาลนับหมื่น ๆ repo กระจายอยู่) |
| **Spotify** | ใช้แนวทาง Polyrepo ผสมกับโมเดลทีมแบบ Squad ที่แต่ละ squad ดูแล repository และ service ของตัวเอง |
| **HashiCorp** | ผลิตภัณฑ์แต่ละตัว (Terraform, Vault, Consul, Nomad) แยก repository ของตัวเองอย่างชัดเจน เพราะแต่ละตัวมี release cycle, versioning, และชุมชนผู้ใช้ที่แตกต่างกันมาก |
| **โปรเจกต์ Open Source ส่วนใหญ่บน GitHub** | ธรรมชาติของ Open Source แทบทั้งหมดคือ Polyrepo อยู่แล้ว เพราะแต่ละโปรเจกต์มีเจ้าของ, license, และชุมชนที่แยกจากกันโดยสิ้นเชิง |

ข้อสังเกต: Polyrepo มักถูกเลือกเมื่อ**ความเป็นอิสระของทีม (team autonomy)** สำคัญกว่าความสะดวกในการแชร์โค้ดข้ามทีม โดยเฉพาะในองค์กรที่ยึดหลัก "loosely coupled teams, loosely coupled services" อย่างจริงจัง

---

## Step 813: ข้อดีของ Monorepo

มาดูรายละเอียดของแต่ละข้อดีที่สำคัญที่สุด พร้อมเหตุผลเชิงเทคนิคว่าทำไมมันถึงสำคัญ

### 1. Atomic Commit ข้ามหลายโปรเจกต์พร้อมกัน

นี่คือข้อดีที่ทรงพลังที่สุดของ Monorepo ลองพิจารณาสถานการณ์นี้:

> คุณต้องเปลี่ยน API signature ของฟังก์ชันในไลบรารีกลาง `shared-types` ซึ่งถูกใช้งานโดย `auth-service`, `payment-service`, และ `web-app` พร้อมกันทั้ง 3 ที่

**ใน Monorepo:**

```bash
git add libs/shared-types/user.ts \
        services/auth-service/handler.ts \
        services/payment-service/handler.ts \
        frontend/web-app/api-client.ts

git commit -m "refactor: เปลี่ยน signature ของ User type และอัปเดตทุกจุดที่ใช้พร้อมกัน"
```

การเปลี่ยนแปลงทั้งหมดนี้อยู่ใน **commit เดียว** — ถ้า build/test ผ่าน แปลว่าทุกอย่างสอดคล้องกันแน่นอน ณ จุดเวลานั้น ไม่มีทางที่ `auth-service` จะใช้ type เวอร์ชันเก่าในขณะที่ `payment-service` ใช้เวอร์ชันใหม่ เพราะทุกอย่างอยู่ใน snapshot เดียวกันของ commit เดียวกัน

**ใน Polyrepo** การเปลี่ยนแปลงเดียวกันนี้ต้อง:

1. เปิด PR แก้ `shared-types` repo → merge → publish package เวอร์ชันใหม่ (เช่น `v2.1.0`)
2. เปิด PR แยกใน `auth-service` เพื่ออัปเดต dependency เป็น `v2.1.0` และแก้โค้ดให้ตรงกับ signature ใหม่
3. เปิด PR แยกใน `payment-service` ทำแบบเดียวกัน
4. เปิด PR แยกใน `web-app` ทำแบบเดียวกัน

ระหว่างขั้นตอนที่ 2-4 ยังไม่เสร็จ ระบบจะอยู่ในสถานะ**ไม่สอดคล้องกัน** ชั่วคราว (บาง service ใช้ type เก่า บาง service ใช้ type ใหม่) และต้องมีการประสานงานข้าม repo หลายรอบ ซึ่งช้ากว่าและเสี่ยงต่อความผิดพลาดมากกว่ามาก

### 2. Code Sharing ทำได้ง่ายมาก

ใน Monorepo การแชร์โค้ดระหว่างโปรเจกต์ทำได้เพียงแค่ `import` ตรง ๆ โดยไม่ต้องผ่านกระบวนการ publish package เลย:

```typescript
// frontend/web-app/src/App.tsx
import { Button } from '../../../libs/shared-ui-components';
import { User } from '../../../libs/shared-types';
```

ไม่ต้อง publish ไป npm registry, ไม่ต้องรอ CI build package, ไม่ต้องจัดการ version number ของ internal library เลย เมื่อแก้ `shared-ui-components` การเปลี่ยนแปลงจะ**มีผลทันที**กับทุกโปรเจกต์ที่ใช้งานมัน (เมื่อ merge เข้า branch หลัก) ทำให้ความเร็วในการทำงานร่วมกันของทีมสูงขึ้นมาก โดยเฉพาะสำหรับ internal tooling หรือ design system ที่มีการเปลี่ยนแปลงบ่อย

### 3. Refactor ทั่วองค์กรทำได้ในครั้งเดียว

เมื่อองค์กรต้องการทำการเปลี่ยนแปลงระดับใหญ่ เช่น:

- เปลี่ยนจาก JavaScript เป็น TypeScript ทั้งบริษัท
- อัปเกรด framework เวอร์ชันหลัก (เช่น React 17 → 18) ในทุกโปรเจกต์
- เปลี่ยนมาตรฐาน logging library ทั้งองค์กร
- แก้ security vulnerability ที่กระทบทุก service ที่ใช้ library เดียวกัน

ใน Monorepo วิศวกรสามารถเขียน script อัตโนมัติ (เช่น ใช้ codemod หรือ `sed`/`jscodeshift`) รันทับทั้ง repository แล้วเปิด **PR เดียว** ที่ครอบคลุมทุกโปรเจกต์ ทดสอบทุกอย่างพร้อมกัน และ merge ครั้งเดียวจบ — Google เรียกกระบวนการนี้ว่า "large-scale change (LSC)" ซึ่งเป็นกระบวนการที่ Google ทำเป็นประจำหลายพันครั้งต่อปีในระดับทั้งบริษัท

ใน Polyrepo การ refactor แบบเดียวกันนี้ต้องเปิด PR แยกกันในทุก repository ที่ได้รับผลกระทบ (อาจเป็นหลักร้อย repo) ต้องประสานงานให้ merge พร้อม ๆ กันหรือใกล้เคียงกัน และติดตามความคืบหน้าด้วยมือ ซึ่งใช้เวลาและแรงงานมากกว่าอย่างมีนัยสำคัญ

---

## Step 814: ข้อเสียของ Monorepo

Monorepo ไม่ใช่ทางออกที่สมบูรณ์แบบ มันมีต้นทุนที่ชัดเจนเมื่อ scale ขึ้น

### 1. Scale ยากเมื่อ Repository โตมาก

ปัญหานี้เราได้พูดถึงรายละเอียดทางเทคนิคไปแล้วใน **Part 63** แต่สรุปสั้น ๆ คือ:

- `git clone` ใช้เวลานานขึ้นเรื่อย ๆ เมื่อจำนวนไฟล์และประวัติ commit เพิ่มขึ้น
- `git status` ต้องสแกนไฟล์ทั้งหมดใน working directory ซึ่งช้าลงเมื่อจำนวนไฟล์เพิ่มขึ้นเป็นล้าน
- Index file ของ Git มีขนาดใหญ่ขึ้นตามจำนวนไฟล์ ทำให้การดำเนินการพื้นฐานหลายอย่างช้าลง
- IDE ที่พยายาม index ทั้ง repository อาจทำงานหนักเกินไปหรือค้าง

องค์กรที่ต้องการใช้ Monorepo ในระดับล้านไฟล์ **จำเป็นต้อง**ลงทุนในเครื่องมือพิเศษ (ดู Step 817) มิฉะนั้นประสบการณ์ของนักพัฒนาจะแย่ลงอย่างมากเมื่อ repository โตขึ้นเรื่อย ๆ

### 2. การควบคุม Permission ต่อทีมทำได้ยากกว่า

ใน Polyrepo การให้สิทธิ์เข้าถึงทำได้อย่างเป็นธรรมชาติ — ทีม A มีสิทธิ์เต็มใน repo ของทีม A เท่านั้น ไม่มีสิทธิ์แตะ repo ของทีม B เลย

ใน Monorepo การควบคุม permission ระดับ "โฟลเดอร์ย่อย" ซับซ้อนกว่ามาก เพราะ Git ไม่ได้ออกแบบมาให้กำหนดสิทธิ์ระดับ path/directory โดยธรรมชาติ (permission ปกติจะเป็นระดับ repository ทั้งก้อน) องค์กรที่ต้องการให้ทีม A แก้ไขได้เฉพาะโฟลเดอร์ของตัวเอง แต่ห้ามแตะโฟลเดอร์ของทีมอื่น ต้องพึ่งพา:

- **CODEOWNERS file** (บังคับให้ต้องมีคนจากทีมที่เกี่ยวข้อง approve ก่อน merge เข้าโฟลเดอร์นั้น — แต่ไม่ได้ "บล็อก" การแก้ไขจริง ๆ แค่บล็อกตอน merge)
- ระบบ permission เฉพาะทางของแพลตฟอร์ม Monorepo องค์กร (เช่น Google มีระบบภายในของตัวเองที่ควบคุม path-based ACL ได้ละเอียดมาก)
- Git hooks หรือ CI check ที่ปฏิเสธ PR ถ้ามีคนแก้ไขไฟล์นอกขอบเขตที่ตัวเองได้รับอนุญาต

โดยรวมแล้ว **การแยก permission ตาม path ภายใน repository เดียวไม่ใช่ฟีเจอร์มาตรฐานของ Git** และต้องอาศัยเครื่องมือเสริมหรือ process เพิ่มเติมเสมอ

### 3. CI ช้าถ้าไม่จัดการ Selective Build ให้ดี

ถ้าทุกครั้งที่มีการ commit ใน Monorepo ระบบ CI พยายาม **build และ test ทุกโปรเจกต์ในทั้ง repository** โดยไม่แยกแยะว่าการเปลี่ยนแปลงนั้นกระทบส่วนไหนจริง ๆ ปัญหาจะรุนแรงมาก:

```
สถานการณ์ที่ไม่ดี:
แก้ไฟล์ 1 บรรทัดใน payment-service
   → CI รัน build + test ของ auth-service ด้วย (ไม่จำเป็น)
   → CI รัน build + test ของ web-app ด้วย (ไม่จำเป็น)
   → CI รัน build + test ของ mobile-app ด้วย (ไม่จำเป็น)
   → ใช้เวลารวม 45 นาที ทั้งที่ควรใช้แค่ 3 นาที
```

นี่คือเหตุผลสำคัญที่องค์กรที่ใช้ Monorepo ขนาดใหญ่ **ต้อง**ลงทุนใน build system ที่รู้จัก dependency graph (เช่น Bazel, Nx, Turborepo — ดู Step 817) เพื่อให้ CI build/test **เฉพาะส่วนที่ได้รับผลกระทบจริง (affected projects only)** ถ้าไม่มีเครื่องมือเหล่านี้ CI time จะเพิ่มขึ้นเรื่อย ๆ แบบไม่เป็นเชิงเส้นเมื่อจำนวนโปรเจกต์ในโครงสร้างเพิ่มขึ้น จนถึงจุดที่ทีมงานเสียเวลารอ CI มากกว่าเวลาที่ใช้เขียนโค้ดจริง

---

## Step 815: ข้อดีของ Polyrepo

### 1. Independent Versioning และ Release ต่อ Service

แต่ละ repository ใน Polyrepo มี version number, tag, และ release cycle ของตัวเองอย่างสมบูรณ์:

```
auth-service       → v3.4.1  (release ทุกสัปดาห์)
payment-service    → v1.12.0 (release ทุก 2 สัปดาห์ เพราะต้อง audit เข้มงวดกว่า)
notification-service → v0.9.2-beta (ยังอยู่ระหว่างพัฒนา ยังไม่ release จริง)
```

การที่แต่ละ service มี release cycle อิสระจากกันโดยสิ้นเชิงมีประโยชน์มาก โดยเฉพาะเมื่อ service ต่าง ๆ มีความสำคัญ ความเสี่ยง หรือข้อกำหนดด้าน compliance ที่แตกต่างกันมาก (เช่น `payment-service` อาจต้องผ่านกระบวนการ audit และ security review ที่เข้มงวดกว่า `notification-service` มาก การผูก release ทั้งสองเข้าด้วยกันจะทำให้ `notification-service` ถูกดึงช้าลงโดยไม่จำเป็น)

### 2. Permission ชัดเจน แยกตาม Repo

ดังที่กล่าวไปใน Step 814 การให้สิทธิ์ระดับ repository เป็นสิ่งที่ GitHub, GitLab, Bitbucket รองรับได้อย่างเป็นธรรมชาติและตรงไปตรงมาที่สุด:

- ทีม Payment มีสิทธิ์ **Admin** ใน `payment-service` แต่ไม่มีสิทธิ์อะไรเลยใน repo อื่น
- ทีม Contractor ภายนอกได้รับสิทธิ์เข้าถึงเฉพาะ repo ที่เกี่ยวข้องกับงานที่ทำ โดยไม่เห็นโค้ดส่วนอื่นขององค์กรเลยแม้แต่บรรทัดเดียว
- Audit log และ compliance report ทำได้ง่ายกว่า เพราะขอบเขตของแต่ละ repo ชัดเจนอยู่แล้ว

นี่เป็นเหตุผลสำคัญที่องค์กรที่ต้องทำงานกับ third-party contractor บ่อย หรือมีข้อกำหนดด้าน security compliance ที่เข้มงวด (เช่น สถาบันการเงิน) มักเอียงไปทาง Polyrepo

### 3. CI เร็วเพราะ Repo เล็ก

เมื่อแต่ละ repository มีขอบเขตเล็กและชัดเจน CI pipeline ของมันก็มีขอบเขตเล็กตามไปด้วยโดยธรรมชาติ **ไม่ต้องเขียน logic พิเศษเพื่อ "เลือก" ว่าจะ build อะไร** เพราะทุกอย่างในนั้นคือของ service เดียวอยู่แล้ว:

```yaml
# .github/workflows/ci.yml ใน payment-service repo
on: [push]
jobs:
  test:
    steps:
      - run: npm install
      - run: npm test
      # ไม่ต้องกังวลเรื่อง affected-project detection เลย
      # เพราะทั้ง repo คือ payment-service เพียงอย่างเดียว
```

ทีมขนาดเล็กที่ยังไม่มีทรัพยากรพอจะลงทุนสร้างระบบ selective build ที่ซับซ้อน จะได้ประโยชน์จากความเรียบง่ายนี้ทันทีโดยไม่ต้องลงทุนอะไรเพิ่มเลย

---

## Step 816: ข้อเสียของ Polyrepo

### 1. Dependency Hell ข้าม Repository

นี่คือปัญหาที่เจ็บปวดที่สุดของ Polyrepo เมื่อจำนวน repository และความสัมพันธ์ระหว่างกันเพิ่มขึ้น ลองพิจารณาสถานการณ์:

```
shared-types@2.0.0  →  auth-service ใช้ shared-types@1.8.0 (ยังไม่ได้อัปเกรด)
                    →  payment-service ใช้ shared-types@2.0.0 (อัปเกรดแล้ว)
                    →  web-app ใช้ shared-types@1.9.0 (อัปเกรดบางส่วน)
```

เมื่อมี package กลางที่หลาย repo ใช้ร่วมกัน แต่ละ repo จะอัปเกรด dependency ไปตามจังหวะของตัวเอง (บางทีมอัปเกรดเร็ว บางทีมอัปเกรดช้า) ผลลัพธ์คือ:

- ระบบทั้งองค์กรอยู่ในสถานะที่ **ใช้ dependency คนละเวอร์ชันกัน** เป็นเวลานาน
- เมื่อพบ security vulnerability ใน `shared-types` เวอร์ชันเก่า ต้องไล่ตามทุก repo ที่ยังไม่ได้อัปเกรดทีละที่ (บางองค์กรมี dashboard ติดตาม dependency version ข้าม repo โดยเฉพาะเพื่อแก้ปัญหานี้)
- Bug ที่เกิดจาก "เวอร์ชันไม่ตรงกัน" (version skew) ระหว่าง service วินิจฉัยได้ยากกว่าปกติมาก เพราะแต่ละฝั่งดูเหมือนทำงานถูกต้องแยกกัน แต่พังเมื่อทำงานร่วมกัน

### 2. Refactor ที่กระทบหลาย Service ทำได้ยากและช้า

ดังที่กล่าวไปใน Step 813 การเปลี่ยนแปลงที่กระทบหลาย repository พร้อมกันใน Polyrepo ต้องผ่านหลายขั้นตอนที่ประสานงานกันยาก:

```
ขั้นตอนการ refactor ข้าม 5 repository:
1. เปิด PR ใน shared-lib          → รอ review → merge → publish v3.0.0
2. เปิด PR ใน service-a (แก้ตามเวอร์ชันใหม่)  → รอ review → merge → deploy
3. เปิด PR ใน service-b (แก้ตามเวอร์ชันใหม่)  → รอ review → merge → deploy
4. เปิด PR ใน service-c (แก้ตามเวอร์ชันใหม่)  → รอ review → merge → deploy
5. เปิด PR ใน service-d (แก้ตามเวอร์ชันใหม่)  → รอ review → merge → deploy
```

ถ้า 5 repo นี้เป็นของทีมต่างกัน จำเป็นต้อง**ประสานงานข้ามทีม**ด้วย ซึ่งอาจใช้เวลาเป็นสัปดาห์หรือเป็นเดือน ในขณะที่การเปลี่ยนแปลงแบบเดียวกันใน Monorepo อาจใช้เวลาไม่กี่ชั่วโมงในการเปิด PR เดียวที่ครอบคลุมทุกจุด

องค์กรที่ใช้ Polyrepo และมีการเปลี่ยนแปลงข้าม service บ่อยครั้ง มักต้องแก้ปัญหานี้ด้วยการสร้าง**tooling พิเศษ** เช่น สคริปต์ที่เปิด PR อัตโนมัติในหลาย repo พร้อมกัน (เช่นเครื่องมืออย่าง `multi-gitter`, หรือ GitHub App ที่เปิด PR ข้าม organization ทั้งหมดพร้อมกันเมื่อพบ dependency ที่ต้องอัปเกรด)

---

## Step 817: เครื่องมือรองรับ Monorepo ขนาดใหญ่ (Bazel, Nx, Turborepo, Lerna)

การที่ Monorepo ขนาดใหญ่ยังคงใช้งานได้จริงในองค์กรระดับ Google หรือ Meta ไม่ใช่เพราะ Git เพียว ๆ ทำได้ดีพอ แต่เพราะมี **build system และ tooling ชั้นบน** ที่ถูกออกแบบมาเฉพาะเพื่อแก้ปัญหาการ scale ของ Monorepo โดยตรง

### Bazel

**Bazel** คือ build system แบบ open source ที่ Google พัฒนาขึ้นจากระบบภายในชื่อ **Blaze** ที่ Google ใช้จัดการ Monorepo ของตัวเองมานานหลายสิบปี

คุณสมบัติสำคัญ:

- **Dependency Graph ที่ชัดเจนแบบ explicit** — ทุก target (โมดูล, ไลบรารี, binary) ต้องประกาศ dependency ของตัวเองอย่างชัดเจนในไฟล์ `BUILD` เช่น:

```python
# BUILD file ตัวอย่าง
java_library(
    name = "payment_service",
    srcs = glob(["src/**/*.java"]),
    deps = [
        "//libs/shared-types:shared_types",
        "//libs/logging-utils:logging_utils",
    ],
)
```

- **Remote Caching และ Remote Execution** — ถ้ามีคนอื่นเคย build target เดียวกันด้วย input เดียวกันมาก่อน (ตรวจสอบด้วย hash ของ input ทั้งหมด) Bazel จะดึงผลลัพธ์จาก cache แทนที่จะ build ใหม่ ทำให้ build ซ้ำ ๆ เร็วขึ้นมหาศาลทั้งทีม
- **Hermetic Build** — รับประกันว่าผลลัพธ์การ build จะเหมือนเดิมทุกครั้งไม่ว่าจะรันที่เครื่องไหน (reproducible build) เพราะ Bazel ควบคุม dependency และ toolchain ทั้งหมดอย่างเข้มงวด
- **Selective Build/Test** — คำสั่งอย่าง `bazel test //...` ที่กระทบเฉพาะ target ที่เปลี่ยนแปลงจริง (affected targets) ทำให้ CI ไม่ต้อง build/test ทุกอย่างในทุกครั้ง

ข้อเสียของ Bazel คือ **learning curve สูงมาก** และต้องเขียนไฟล์ `BUILD` กำกับทุกโมดูลอย่างละเอียด ทำให้ต้นทุนเริ่มต้น (initial setup cost) สูง เหมาะกับองค์กรขนาดใหญ่ที่พร้อมลงทุนระยะยาว

### Nx

**Nx** เป็นเครื่องมือที่ได้รับความนิยมมากในระบบนิเวศ JavaScript/TypeScript (โดยเฉพาะทีมที่ใช้ Angular, React, Node.js) พัฒนาโดยทีม Nrwl

คุณสมบัติสำคัญ:

- **Dependency Graph อัตโนมัติ** — Nx วิเคราะห์โค้ดเพื่อสร้าง dependency graph ให้อัตโนมัติโดยไม่ต้องเขียนไฟล์ config อธิบายทุกความสัมพันธ์เอง (ต่างจาก Bazel ที่ต้อง declare ทุกอย่างชัดเจน)
- **Affected Command** — คำสั่งอย่าง `nx affected --target=test` จะรัน test เฉพาะโปรเจกต์ที่ได้รับผลกระทบจากการเปลี่ยนแปลงใน commit ปัจจุบันเท่านั้น โดยเทียบกับ branch หลัก
- **Computation Caching** — คล้าย Bazel คือ cache ผลลัพธ์ของ task ที่เคยรันด้วย input เดียวกันมาก่อน ทั้งแบบ local และแบบ distributed ผ่าน Nx Cloud
- **Generators และ Plugins** — มีระบบ scaffold โค้ดและปลั๊กอินสำหรับ framework ยอดนิยมจำนวนมาก ทำให้เริ่มต้นใช้งานง่ายกว่า Bazel มาก

Nx เหมาะกับทีมขนาดกลางถึงใหญ่ในระบบนิเวศ JavaScript/TypeScript ที่ต้องการ Monorepo tooling ที่ทรงพลังแต่ยังตั้งค่าได้ไม่ยากเกินไป

### Turborepo

**Turborepo** (ปัจจุบันเป็นของ Vercel) เป็นเครื่องมือที่เน้นความเรียบง่ายและความเร็วในการตั้งค่าเป็นหลัก เหมาะกับ JavaScript/TypeScript Monorepo ขนาดเล็กถึงกลาง

คุณสมบัติสำคัญ:

- **การตั้งค่าที่เรียบง่ายมาก** — ไฟล์ config หลักคือ `turbo.json` ที่สั้นและอ่านง่าย:

```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "test": {
      "dependsOn": ["build"]
    }
  }
}
```

- **Local และ Remote Caching** — คล้ายกับ Nx และ Bazel แต่เน้นการตั้งค่าที่ง่ายกว่า ผลลัพธ์ของ task ถูก cache ตาม hash ของ input และสามารถแชร์ cache กันได้ทั้งทีมผ่าน Vercel Remote Cache
- **Parallel Execution** — รัน task ของหลายโปรเจกต์พร้อมกันโดยอัตโนมัติตาม dependency graph เพื่อใช้ CPU ให้เต็มประสิทธิภาพ

Turborepo ได้รับความนิยมสูงในหมู่ทีม frontend และ full-stack ขนาดเล็กถึงกลางที่ต้องการผลลัพธ์เร็วโดยไม่ต้องเรียนรู้ระบบที่ซับซ้อนแบบ Bazel

### Lerna

**Lerna** เป็นเครื่องมือรุ่นบุกเบิกสำหรับจัดการ JavaScript Monorepo ที่มีหลาย npm package อยู่ใน repository เดียว (เรียกว่า "package-based monorepo") เกิดขึ้นก่อน Nx และ Turborepo มาก และเคยเป็นมาตรฐานโดยพฤตินัยของวงการ JavaScript

คุณสมบัติสำคัญ:

- **จัดการ Versioning ของหลาย package พร้อมกัน** — คำสั่ง `lerna version` ช่วยตัดสินใจว่า package ไหนควรถูก bump version เท่าไหร่ตาม commit ที่เปลี่ยนแปลง (โดยเฉพาะเมื่อใช้ conventional commits)
- **Publish หลาย package พร้อมกัน** — คำสั่ง `lerna publish` จัดการการ publish หลาย npm package จาก monorepo เดียวไปยัง registry ในคราวเดียว
- ปัจจุบัน Lerna ถูกดูแลต่อโดยทีม Nx และผสานการทำงานร่วมกับ Nx สำหรับความสามารถด้าน task running และ caching สมัยใหม่ (Lerna เวอร์ชันใหม่ใช้ Nx เป็น engine เบื้องหลังสำหรับงาน build/test)

### ตารางสรุปเปรียบเทียบ

| เครื่องมือ | เหมาะกับ | จุดเด่น | Learning Curve |
|---|---|---|---|
| **Bazel** | Monorepo ขนาดมหึมาระดับ Google, หลายภาษาผสมกัน | Hermetic build, remote execution, scale ได้สุดขีด | สูงมาก |
| **Nx** | JS/TS Monorepo ขนาดกลาง-ใหญ่ | Dependency graph อัตโนมัติ, affected commands, ecosystem plugin กว้าง | ปานกลาง |
| **Turborepo** | JS/TS Monorepo ขนาดเล็ก-กลาง | ตั้งค่าง่าย เร็ว, caching ที่ใช้งานง่าย | ต่ำ |
| **Lerna** | JS Monorepo แบบ package-based (publish หลาย npm package) | จัดการ versioning/publishing หลาย package | ต่ำ-ปานกลาง |

หมายเหตุสำคัญ: เครื่องมือเหล่านี้ทำงาน**เสริม**กับ Git ไม่ใช่**แทนที่** Git — Git ยังคงเป็น version control layer เหมือนเดิม เครื่องมือเหล่านี้เป็น build/task orchestration layer ที่วางอยู่ด้านบนเพื่อแก้ปัญหาเรื่องความเร็วของ build/test ในโครงสร้าง Monorepo โดยเฉพาะ

---

## Step 818: Hybrid Approach — เมื่อองค์กรใช้ทั้งสองแบบผสมกัน

ในความเป็นจริง องค์กรจำนวนมากไม่ได้เลือกแบบใดแบบหนึ่งแบบสุดขั้ว แต่ใช้แนวทาง **Hybrid** ที่ผสมผสานข้อดีของทั้งสองแบบเข้าด้วยกัน รูปแบบที่พบบ่อยมีดังนี้:

### รูปแบบที่ 1: Monorepo ต่อทีม, Polyrepo ระหว่างทีม

นี่คือรูปแบบ Hybrid ที่พบบ่อยที่สุดในองค์กรขนาดกลางถึงใหญ่:

```
Team Payments (Monorepo ของทีมนี้เอง)
└── payments-monorepo/
    ├── payment-api/
    ├── payment-worker/
    ├── payment-shared-lib/
    └── payment-admin-ui/

Team Search (Monorepo แยกของทีมนี้)
└── search-monorepo/
    ├── search-api/
    ├── indexing-service/
    └── search-shared-lib/

Team Identity (Monorepo แยกของทีมนี้)
└── identity-monorepo/
    ├── auth-service/
    └── user-service/
```

**เหตุผลของรูปแบบนี้:** ภายในทีมเดียวกัน สมาชิกทำงานใกล้ชิดกันมาก การแชร์โค้ดและ atomic commit ข้าม service ภายในทีมมีประโยชน์สูง (จึงใช้ Monorepo) แต่ระหว่างทีมที่แตกต่างกัน ความเป็นอิสระ (autonomy) สำคัญกว่า และการเปลี่ยนแปลงข้ามทีมเกิดขึ้นไม่บ่อยเท่าภายในทีม (จึงใช้ Polyrepo ระหว่างทีม) รูปแบบนี้ยังทำให้ **ขอบเขต permission ตรงกับขอบเขตทีมพอดี** ซึ่งแก้ปัญหาเรื่อง permission ของ Monorepo ขนาดใหญ่ไปในตัว

### รูปแบบที่ 2: Core Monorepo + Satellite Repos

องค์กรบางแห่งมี Monorepo หลักสำหรับโค้ดที่ share กันบ่อย (เช่น business logic, shared library, internal platform) แต่แยก repository ต่างหากสำหรับสิ่งที่มีลักษณะพิเศษ เช่น:

- โปรเจกต์ Open Source ที่เผยแพร่สู่สาธารณะ (แยก repo เพื่อควบคุม license และการเข้าถึงจากภายนอก)
- Mobile app ที่ต้องผูกกับ build pipeline เฉพาะของ App Store/Play Store (บางทีมแยกเพื่อความสะดวกด้าน tooling เฉพาะทาง)
- Infrastructure as Code (Terraform ฯลฯ) ที่มักมี access control และ audit requirement ต่างจากโค้ด application

### รูปแบบที่ 3: Polyrepo ที่ผูกด้วย Meta-tooling

บางองค์กรยังคงใช้ Polyrepo เป็นหลัก แต่สร้างเครื่องมือช่วยให้ "รู้สึก" เหมือน Monorepo มากขึ้น เช่น:

- ใช้ **Git Submodules** หรือ **Git Subtree** เพื่อรวม repository หลายตัวเข้าด้วยกันในระดับหนึ่ง (แต่ยังคง history แยกกันอยู่ข้างใต้)
- ใช้เครื่องมืออย่าง **meta** หรือสคริปต์ภายในที่ clone หลาย repo มาไว้ในโฟลเดอร์เดียวกันบนเครื่อง developer แล้วให้คำสั่งเดียวสามารถทำงานข้ามหลาย repo พร้อมกันได้ (เช่น `meta git status` ที่รายงานสถานะของทุก repo ในโปรเจกต์)
- ใช้ tool อย่าง **multi-gitter** หรือสคริปต์ automation เพื่อเปิด PR ในหลาย repository พร้อมกันสำหรับการเปลี่ยนแปลงที่กระทบทุกที่ (เช่น อัปเกรด dependency เดียวกันในทุก repo)

### ตารางสรุป Hybrid Patterns

| รูปแบบ | ใช้เมื่อ | ตัวอย่าง |
|---|---|---|
| Monorepo ต่อทีม, Polyrepo ระหว่างทีม | องค์กรมีหลายทีมที่ทำงานค่อนข้างอิสระจากกัน แต่ภายในทีมแชร์โค้ดกันบ่อย | บริษัทเทคโนโลยีขนาดกลาง-ใหญ่จำนวนมาก |
| Core Monorepo + Satellite Repos | มีโค้ดหลักที่แชร์กันเข้มข้น แต่มีบางส่วนที่มีข้อกำหนดพิเศษ | บริษัทที่มีทั้งโค้ด internal และโปรเจกต์ Open Source |
| Polyrepo + Meta-tooling | องค์กรลงทุนกับ Polyrepo ไปแล้วมาก แต่อยากได้ประสบการณ์บางอย่างของ Monorepo เพิ่ม | องค์กรขนาดใหญ่ที่มี repo เดิมอยู่แล้วนับร้อย ย้ายไป Monorepo เต็มรูปแบบไม่คุ้มต้นทุน |

ข้อคิดสำคัญ: **ไม่มีคำตอบที่ถูกต้องสมบูรณ์แบบเพียงหนึ่งเดียว** — Hybrid Approach คือการยอมรับว่าองค์กรจริงมีความหลากหลายของบริบทเกินกว่าจะบังคับใช้กฎเดียวกันกับทุกส่วนขององค์กร

---

## Step 819: Decision Framework — จะเลือก Monorepo หรือ Polyrepo อย่างไร

เมื่อต้องตัดสินใจเลือกสถาปัตยกรรม repository ให้กับองค์กรหรือทีมของคุณ ควรพิจารณาปัจจัยต่อไปนี้อย่างเป็นระบบ แทนที่จะเลือกตามกระแสหรือความชอบส่วนตัว

### ปัจจัยที่ 1: ขนาดทีมและโครงสร้างองค์กร

| ขนาดทีม | แนวโน้มที่เหมาะสม | เหตุผล |
|---|---|---|
| ทีมเดียว (1–15 คน) | **Monorepo** | ทุกคนรู้จักโค้ดเกือบทั้งหมดอยู่แล้ว การแยก repo เพิ่มความซับซ้อนโดยไม่จำเป็น |
| หลายทีมเล็ก ที่ทำงานประสานกันแน่นหนา (15–100 คน) | **Monorepo** หรือ **Hybrid** | ยังได้ประโยชน์จาก atomic commit และ code sharing แต่เริ่มต้องคิดเรื่อง permission |
| หลายทีมใหญ่ ที่ทำงานค่อนข้างอิสระจากกัน (100–1000 คน) | **Hybrid** (Monorepo ต่อทีม/กลุ่ม) | ต้องการทั้งความเร็วภายในทีมและความเป็นอิสระระหว่างทีม |
| องค์กรระดับพันคนขึ้นไป ที่มีวัฒนธรรมวิศวกรรมเข้มแข็งและพร้อมลงทุน tooling | **Monorepo เต็มรูปแบบ** (แบบ Google) หรือ **Polyrepo ที่มี platform team ดูแล** | ทั้งสองแบบเป็นไปได้ ขึ้นกับว่าองค์กรพร้อมลงทุนด้านไหนมากกว่า |

### ปัจจัยที่ 2: ความสัมพันธ์ของโค้ด (Code Coupling)

ถามตัวเองว่า: **โปรเจกต์ต่าง ๆ ในองค์กรมีการเปลี่ยนแปลงที่กระทบกันบ่อยแค่ไหน?**

- ถ้าโปรเจกต์ A และ B **มักต้องเปลี่ยนพร้อมกันเสมอ** (tightly coupled) เช่น backend API กับ frontend ที่พัฒนาโดยทีมเดียวกันและ deploy พร้อมกันเป็นประจำ → **Monorepo ได้ประโยชน์ชัดเจน**
- ถ้าโปรเจกต์ A และ B **แทบไม่เกี่ยวข้องกันเลย** (loosely coupled) เช่น service สอง service ที่สื่อสารกันผ่าน well-defined API/contract เท่านั้น และแทบไม่เคยต้องแก้พร้อมกัน → **Polyrepo เหมาะสมกว่า** เพราะการแยก repo ไม่สร้างต้นทุนเพิ่มเติมมากนัก ในขณะที่ได้ความเป็นอิสระกลับมา

### ปัจจัยที่ 3: เครื่องมือและทรัพยากรที่มี

ถามตัวเองว่า: **ทีมมีทรัพยากร (คน, เวลา, งบประมาณ) พอที่จะลงทุนสร้าง/ดูแล tooling สำหรับ Monorepo ขนาดใหญ่หรือไม่?**

- ถ้า**ไม่มี** ทรัพยากรสำหรับตั้งค่า Bazel/Nx/Turborepo อย่างจริงจัง และ CI ยัง build ทุกอย่างแบบง่าย ๆ → การใช้ Monorepo ขนาดใหญ่แบบไร้เครื่องมือเสริมจะสร้างปัญหา CI ช้าอย่างรวดเร็ว (ตาม Step 814) ควรพิจารณา Polyrepo หรือ Monorepo ขนาดเล็กที่ยังจัดการได้ง่าย
- ถ้า**มี** platform team หรือ DevOps team ที่พร้อมดูแล build system ระดับ enterprise → Monorepo ขนาดใหญ่เป็นไปได้และให้ผลตอบแทนสูง

### ปัจจัยที่ 4: ข้อกำหนดด้าน Compliance และ Security

- องค์กรที่มีข้อกำหนดเข้มงวดเรื่องการแยกการเข้าถึงข้อมูล (เช่น สถาบันการเงิน, healthcare, หน่วยงานรัฐ) ที่ต้องพิสูจน์ได้ชัดเจนว่า "ทีม X ไม่สามารถเข้าถึงโค้ดของทีม Y ได้เลย" มักพบว่า **Polyrepo ตอบโจทย์ audit ได้ง่ายกว่า** เพราะขอบเขตของ repository ตรงกับขอบเขตของสิทธิ์การเข้าถึงโดยธรรมชาติ
- ถ้าไม่มีข้อกำหนดลักษณะนี้ Monorepo พร้อม path-based permission tooling ก็สามารถตอบโจทย์ compliance ได้เช่นกัน เพียงแต่ต้องลงทุนตั้งค่าเพิ่มเติม

### สรุปเป็น Decision Tree แบบง่าย

```
เริ่มต้น
  │
  ├─ ทีมเดียวหรือทีมเล็กมาก? ──────────────► ใช้ Monorepo
  │
  ├─ โค้ดหลายส่วนเปลี่ยนพร้อมกันบ่อยมาก
  │  และมีทรัพยากรลงทุน tooling? ─────────► ใช้ Monorepo (พร้อม Bazel/Nx/Turborepo)
  │
  ├─ หลายทีมอิสระจากกันมาก
  │  ต้องการ permission แยกชัดเจน
  │  หรือมีข้อกำหนด compliance เข้มงวด? ───► ใช้ Polyrepo
  │
  └─ อยู่ตรงกลาง (มีทั้งงานที่ต้องแชร์
     และทีมที่ต้องการอิสระ)? ─────────────► ใช้ Hybrid Approach
```

Decision Framework นี้ไม่ใช่กฎตายตัว — มันเป็นจุดเริ่มต้นในการตั้งคำถามที่ถูกต้อง องค์กรควรทบทวนการตัดสินใจนี้เป็นระยะ เพราะบริบทขององค์กร (ขนาดทีม, ความสัมพันธ์ของโค้ด, ทรัพยากรที่มี) เปลี่ยนแปลงไปตามเวลาเสมอ สิ่งที่เหมาะกับสตาร์ทอัพ 10 คนอาจไม่เหมาะกับบริษัทเดียวกันเมื่อโตเป็น 500 คนแล้ว

---

## Step 820: แบบฝึกหัด — วิเคราะห์สถานการณ์องค์กรสมมติ 3 แบบ

ถึงเวลาฝึกใช้ Decision Framework จาก Step 819 กับสถานการณ์จริง ลองอ่านสถานการณ์แต่ละแบบ คิดคำตอบของตัวเองก่อน แล้วค่อยดูแนวการวิเคราะห์ที่ให้ไว้

### สถานการณ์ที่ 1: สตาร์ทอัพเล็ก

**บริบท:** บริษัท Fintech เริ่มต้นใหม่ มีวิศวกร 6 คน กำลังสร้าง MVP ที่ประกอบด้วย backend API (Node.js), เว็บแอปสำหรับผู้ใช้ (React), และ admin dashboard เล็ก ๆ สำหรับทีมภายใน ทุกคนในทีมทำงานใกล้ชิดกันมาก บางคนสลับไปมาระหว่างการแก้ backend และ frontend ในวันเดียวกัน ยังไม่มี DevOps engineer เฉพาะทาง ทุกคนต้องดูแล CI/CD เอง

**คำถาม:** ควรใช้ Monorepo หรือ Polyrepo?

**แนวการวิเคราะห์:**

- **ขนาดทีม:** 6 คน — เล็กมาก ตามปัจจัยที่ 1 ควรเอียงไปทาง Monorepo ชัดเจน
- **ความสัมพันธ์ของโค้ด:** สูงมาก — backend, frontend, admin dashboard ล้วนเปลี่ยนแปลงพร้อมกันบ่อยในช่วง MVP (เช่น เพิ่ม field ใหม่ใน API ต้องแก้ทั้ง backend และ frontend พร้อมกันแทบทุกครั้ง)
- **ทรัพยากร tooling:** ไม่มี DevOps engineer เฉพาะทาง และ codebase ยังเล็กมาก (ไม่ถึงหลักหมื่นไฟล์แน่นอน) จึง**ไม่จำเป็น**ต้องใช้ Bazel เลย ใช้ Turborepo หรือแม้แต่ npm/yarn workspaces ธรรมดาก็เพียงพอ
- **Compliance:** ยังไม่มีข้อกำหนดซับซ้อน เพราะเป็นทีมเล็กที่ทุกคนเชื่อใจกัน

**ข้อสรุป: ใช้ Monorepo** — โครงสร้างแบบง่าย ๆ เช่น

```
fintech-mvp/
├── apps/
│   ├── api/
│   ├── web/
│   └── admin/
├── packages/
│   └── shared-types/
└── turbo.json
```

การใช้ Monorepo ทำให้ทีมเล็กนี้ atomic commit ข้าม backend/frontend ได้ทันที แชร์ type ระหว่าง API กับ frontend ได้โดยไม่ต้อง publish package ไหนเลย และตั้งค่าเพียงครั้งเดียวด้วย Turborepo ที่เรียนรู้ง่าย เหมาะกับความเร็วที่สตาร์ทอัพต้องการในช่วงแรก การแยกเป็น Polyrepo ในจุดนี้จะเพิ่ม overhead การจัดการที่ไม่คุ้มค่ากับทีมขนาดนี้เลย

### สถานการณ์ที่ 2: บริษัทขนาดกลางที่มีหลายทีม

**บริบท:** บริษัท e-commerce มีวิศวกร 150 คน แบ่งเป็น 12 ทีมตามโดเมนธุรกิจ (Catalog, Cart & Checkout, Payments, Shipping, Search, Recommendations, Customer Support Tools, Marketing, Seller Platform, Identity, Platform/Infra, Data Engineering) แต่ละทีมมี service ของตัวเอง 2-4 service และ deploy อย่างเป็นอิสระ ทีม Payments มีข้อกำหนดด้าน PCI-DSS compliance ที่เข้มงวดกว่าทีมอื่นมาก ทีมต่าง ๆ สื่อสารกันผ่าน internal API ที่มีสัญญา (contract) ชัดเจน และไม่ค่อยต้องแก้โค้ดข้ามทีมบ่อยนัก แต่ภายในแต่ละทีมมีการแชร์โค้ดกันระหว่าง service ของทีมนั้นบ่อยมาก

**คำถาม:** ควรใช้ Monorepo หรือ Polyrepo?

**แนวการวิเคราะห์:**

- **ขนาดทีม:** 150 คน แบ่งเป็น 12 ทีมย่อย — อยู่ในโซนที่ต้องพิจารณา Hybrid ตามปัจจัยที่ 1
- **ความสัมพันธ์ของโค้ด:** สูง**ภายใน**แต่ละทีม (2-4 service ต่อทีมแชร์โค้ดกันบ่อย) แต่ต่ำ**ระหว่าง**ทีม (สื่อสารผ่าน contract ที่ชัดเจน ไม่ค่อยแก้ข้ามทีม) — รูปแบบนี้ตรงกับ pattern "Monorepo ต่อทีม, Polyrepo ระหว่างทีม" จาก Step 818 พอดี
- **ทรัพยากร tooling:** บริษัทขนาดนี้มักมี platform/infra team (12 ทีมมีทีมหนึ่งเป็น Platform/Infra อยู่แล้ว) ที่สามารถดูแล Nx หรือ Turborepo ให้แต่ละทีมได้
- **Compliance:** ทีม Payments มีข้อกำหนด PCI-DSS ที่เข้มงวดกว่าทีมอื่นมาก — นี่คือสัญญาณสำคัญที่บอกว่า **ทีม Payments ควรมี repository (หรือกลุ่ม repository) แยกออกมาต่างหากจากทีมอื่นอย่างชัดเจน** เพื่อให้ควบคุมการเข้าถึงและผ่าน audit ได้ง่าย

**ข้อสรุป: ใช้ Hybrid Approach** — แต่ละทีมมี Monorepo ของตัวเอง (เช่น `catalog-monorepo`, `payments-monorepo`, `search-monorepo` ฯลฯ) รวมเป็น 12 repository ระดับทีม โดยทีม Payments ได้รับการดูแลสิทธิ์การเข้าถึงที่เข้มงวดเป็นพิเศษ (แยก GitHub organization หรือ team permission ระดับสูงสุด) ส่วนการสื่อสารระหว่างทีมทำผ่าน internal API/contract ที่ชัดเจน และใช้ package registry ภายใน (internal npm registry หรือ internal Maven repository) สำหรับ shared library ไม่กี่ตัวที่จำเป็นต้องใช้ข้ามทีมจริง ๆ (เช่น design system, logging client)

### สถานการณ์ที่ 3: องค์กรใหญ่ระดับพันคน

**บริบท:** บริษัทเทคโนโลยีระดับโลกมีวิศวกร 8,000 คนทั่วโลก มีผลิตภัณฑ์หลักหลายสิบตัว ตั้งแต่ core platform, mobile app, web app, internal tools, data infrastructure ไปจนถึง machine learning platform บริษัทมี dedicated Developer Experience (DevEx) team ขนาดใหญ่กว่า 100 คนที่ดูแล build infrastructure โดยเฉพาะ วิศวกรรมส่วนใหญ่เชื่อในหลักการ "ทุกคนควรเห็นและแก้โค้ดของทีมอื่นได้ในกรณีจำเป็น (code visibility ทั่วบริษัท)" และมีการทำ large-scale refactor ข้ามทีมเป็นประจำ (เช่น อัปเกรด internal framework เวอร์ชันหลักปีละครั้งที่กระทบทุกทีม)

**คำถาม:** ควรใช้ Monorepo หรือ Polyrepo?

**แนวการวิเคราะห์:**

- **ขนาดทีม:** 8,000 คน — ใหญ่มาก ตามปัจจัยที่ 1 ทั้ง Monorepo เต็มรูปแบบและ Polyrepo ที่มี platform team ดูแลเป็นไปได้ทั้งคู่ แต่ต้องดูปัจจัยอื่นประกอบ
- **ความสัมพันธ์ของโค้ด:** สูงในเชิงว่ามีการทำ large-scale refactor ข้ามทีมเป็นประจำทุกปี (เช่น อัปเกรด framework ที่กระทบทุกทีม) — นี่คือสัญญาณที่ชัดเจนมากว่า **Monorepo ให้ประโยชน์สูง** เพราะการทำ large-scale change ข้ามทั้งองค์กรใน Polyrepo ที่มีหลายพัน repository จะยากและช้ากว่ามาก (ต้องเปิด PR แยกกันในทุก repo ที่ได้รับผลกระทบ)
- **ทรัพยากร tooling:** มี DevEx team กว่า 100 คนโดยเฉพาะ — เพียงพออย่างเหลือเฟือสำหรับการลงทุนสร้างและดูแล Bazel-based build system, remote caching infrastructure, และ path-based permission system ระดับ enterprise
- **วัฒนธรรมองค์กร:** เชื่อในหลักการ "code visibility ทั่วบริษัท" อย่างชัดเจน ซึ่งตรงกับปรัชญาของ Monorepo แบบ Google โดยตรง (ที่ Google เกือบทุก repository เปิดให้วิศวกรทั้งบริษัทอ่านโค้ดกันได้ ยกเว้นส่วนที่อ่อนไหวจริง ๆ เช่นระบบความปลอดภัยบางส่วน)

**ข้อสรุป: ใช้ Monorepo เต็มรูปแบบ** (ตามแบบ Google/Meta/Microsoft Windows) พร้อมลงทุนสร้าง build system ระดับ Bazel หรือเทียบเท่า, ระบบ remote caching ขนาดใหญ่, ระบบ path-based ACL ที่ควบคุมการเข้าถึงและ CODEOWNERS อย่างละเอียด, และระบบ selective CI ที่รู้ affected targets ทั้งหมด องค์กรระดับนี้มีทั้งความจำเป็น (large-scale refactor บ่อย, ต้องการ code visibility สูง) และความสามารถ (DevEx team ขนาดใหญ่) ที่จะทำให้ Monorepo ขนาดมหึมาคุ้มค่ากับการลงทุนอย่างแท้จริง — ต่างจากสถานการณ์ที่ 1 ที่สตาร์ทอัพเล็กไม่มีความจำเป็นต้องใช้เครื่องมือระดับนี้เลย

### สรุปบทเรียนจากแบบฝึกหัดทั้ง 3 สถานการณ์

| สถานการณ์ | ขนาดทีม | Code Coupling | Tooling พร้อม? | Compliance เข้มงวด? | ข้อสรุป |
|---|---|---|---|---|---|
| 1. สตาร์ทอัพเล็ก | เล็กมาก (6 คน) | สูงมาก | ไม่จำเป็นต้องมี | ไม่มี | **Monorepo** แบบง่าย |
| 2. บริษัทกลาง หลายทีม | กลาง (150 คน, 12 ทีม) | สูงในทีม/ต่ำข้ามทีม | มี platform team | มี (ทีม Payments) | **Hybrid** — Monorepo ต่อทีม |
| 3. องค์กรใหญ่ระดับพันคน | ใหญ่มาก (8,000 คน) | สูง (refactor ข้ามทีมบ่อย) | มี DevEx team ขนาดใหญ่ | ไม่ระบุว่าเข้มงวดเป็นพิเศษ | **Monorepo เต็มรูปแบบ** ด้วย Bazel-class tooling |

ข้อสังเกตสำคัญจากแบบฝึกหัดนี้คือ **ขนาดองค์กรเพียงอย่างเดียวไม่ได้เป็นตัวชี้ขาด** — สถานการณ์ที่ 2 และ 3 ต่างก็เป็นองค์กรขนาดใหญ่ แต่ได้ข้อสรุปต่างกัน (Hybrid vs Monorepo เต็มรูปแบบ) เพราะปัจจัยเรื่อง**ความสัมพันธ์ของโค้ดระหว่างทีม**, **ข้อกำหนด compliance**, และ**ความพร้อมของ tooling** ต่างกันอย่างมีนัยสำคัญ นี่คือเหตุผลที่ Decision Framework ใน Step 819 ต้องพิจารณาหลายปัจจัยร่วมกัน ไม่ใช่ดูแค่ขนาดทีมอย่างเดียว

---

## สรุป Part 82

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Monorepo** คือการเก็บโค้ดของทุกโปรเจกต์/ทีมไว้ใน Git repository เดียว องค์กรระดับโลกอย่าง Google, Meta, และ Microsoft (repository ของ Windows) ใช้แนวทางนี้ในระดับที่ใหญ่มาก
2. **Polyrepo** คือการแยก repository ต่อโปรเจกต์/service องค์กรอย่าง Netflix และทีมงานจำนวนมากใน Amazon ใช้แนวทางนี้เพื่อรองรับสถาปัตยกรรม microservices ที่ต้องการความเป็นอิสระของทีมสูง
3. ข้อดีหลักของ Monorepo คือ **atomic commit ข้ามโปรเจกต์**, **code sharing ที่ง่ายมาก**, และ **การ refactor ทั่วองค์กรในครั้งเดียว**
4. ข้อเสียหลักของ Monorepo คือ **scale ยากเมื่อโตมาก**, **การควบคุม permission ต่อทีมทำได้ยากกว่า**, และ **CI ช้าถ้าไม่มี selective build ที่ดี**
5. ข้อดีหลักของ Polyrepo คือ **independent versioning/release**, **permission ที่ชัดเจนตาม repo**, และ **CI ที่เร็วเพราะ repo เล็ก**
6. ข้อเสียหลักของ Polyrepo คือ **dependency hell ข้าม repository** และ **การ refactor ข้ามหลาย service ที่ยากและช้า**
7. เครื่องมืออย่าง **Bazel, Nx, Turborepo, และ Lerna** คือสิ่งที่ทำให้ Monorepo ขนาดใหญ่ยังคงทำงานได้เร็วในระดับองค์กร แต่ละตัวมี trade-off ระหว่างความสามารถกับความง่ายในการตั้งค่าต่างกัน
8. หลายองค์กรใช้แนวทาง **Hybrid** เช่น Monorepo ต่อทีม + Polyrepo ระหว่างทีม เพื่อผสมผสานข้อดีของทั้งสองแบบ
9. การตัดสินใจเลือก Monorepo หรือ Polyrepo ควรพิจารณา **ขนาดทีม, ความสัมพันธ์ของโค้ด, ความพร้อมของเครื่องมือ, และข้อกำหนด compliance** ร่วมกัน ไม่ใช่เลือกตามกระแสเพียงอย่างเดียว
10. จากแบบฝึกหัดทั้ง 3 สถานการณ์ เราเห็นว่าขนาดองค์กรเพียงอย่างเดียวไม่ได้เป็นตัวชี้ขาดคำตอบ — ปัจจัยอื่น ๆ ล้วนมีผลสำคัญไม่แพ้กัน

Part ถัดไปเราจะขยายความจาก Part นี้ต่อ โดยเจาะลึกเรื่องการจัดการ repository ขนาดใหญ่ระดับ Enterprise แบบเต็มรูปแบบ ทั้งในแง่ของ Monorepo และ Polyrepo — รวมถึงกลยุทธ์การ migrate จากแบบหนึ่งไปอีกแบบหนึ่งเมื่อองค์กรเติบโตหรือบริบทเปลี่ยนไป

**ต่อไป:** [Part 83: การจัดการ Repository ขนาดใหญ่ระดับ Enterprise](./part-083-repository-enterprise-scale.md)
