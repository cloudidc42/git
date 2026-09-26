# Part 89: Changelog Automation และ Release Notes

> **Step ในหลักสูตรนี้:** Step 881–890
> **เฟส:** 9 — มืออาชีพ / Enterprise Practice
> **เป้าหมายของ Part นี้:** เข้าใจว่า Changelog คืออะไรและสำคัญกับผู้ใช้อย่างไร รู้จักมาตรฐาน "Keep a Changelog" เปรียบเทียบการเขียน changelog ด้วยมือกับแบบอัตโนมัติ เจาะลึกเครื่องมืออัตโนมัติหลักอย่าง `semantic-release`, `conventional-changelog`, Release Please และฟีเจอร์ auto-generated release notes ของ GitHub รวมถึงแนวทางจัดการ changelog สำหรับ monorepo ด้วย Changesets และลงมือตั้งค่า automated changelog เต็มรูปแบบให้โปรเจกต์จริง

---

## สารบัญของ Part นี้

- Step 881: Changelog คืออะไร ทำไมสำคัญกับผู้ใช้
- Step 882: "Keep a Changelog" format มาตรฐานที่นิยมใช้กันทั่วโลก
- Step 883: การเขียน Changelog ด้วยมือ vs อัตโนมัติ ข้อดีข้อเสียแต่ละแบบ
- Step 884: `semantic-release` เจาะลึก — generate Changelog อัตโนมัติจาก Conventional Commits
- Step 885: `conventional-changelog` เครื่องมือเสริมสำหรับ generate Changelog แบบ standalone
- Step 886: Release Please ของ Google — ทางเลือกอื่นในการ automate release + changelog
- Step 887: GitHub Auto-generated Release Notes ฟีเจอร์ในตัว
- Step 888: การจัดหมวดหมู่ Changelog ให้อ่านง่าย
- Step 889: Changelog สำหรับ Monorepo ที่มีหลาย Package พร้อมกัน (แนวคิดของ Changesets)
- Step 890: แบบฝึกหัด — ตั้งค่า Automated Changelog ให้โปรเจกต์ด้วย `semantic-release` เต็มรูปแบบ

---

## Step 881: Changelog คืออะไร ทำไมสำคัญกับผู้ใช้

### Changelog คืออะไร

**Changelog** คือเอกสาร (มักเป็นไฟล์ `CHANGELOG.md` ที่วางอยู่ที่ root ของ repository) ที่บันทึก **"อะไรเปลี่ยนแปลงไปบ้างในแต่ละเวอร์ชันของโปรเจกต์"** โดยเรียงตามลำดับเวลา จากเวอร์ชันล่าสุดไปถึงเวอร์ชันเก่าที่สุด

ตัวอย่างหน้าตาคร่าว ๆ:

```markdown
# Changelog

## [2.3.0] - 2026-09-15

### Added
- เพิ่มระบบแจ้งเตือนผ่านอีเมลเมื่อมีการอัปเดตออเดอร์

### Fixed
- แก้ไขบั๊กที่ทำให้ตะกร้าสินค้าล้างค่าเองเมื่อ refresh หน้า

## [2.2.1] - 2026-08-28

### Fixed
- แก้ไขช่องโหว่ด้านความปลอดภัยใน endpoint /api/login
```

จากตัวอย่างนี้ เราจะเห็นว่า Changelog ไม่ใช่แค่ "บันทึกทางเทคนิค" แต่เป็น **เอกสารสื่อสารระหว่างทีมพัฒนากับผู้ใช้งาน**

### ทำไม Changelog ถึงสำคัญมาก

ลองจินตนาการสถานการณ์ที่ **ไม่มี Changelog**: โปรเจกต์ของคุณมี commit นับพันครั้ง ผู้ใช้ที่อยากรู้ว่าเวอร์ชัน 2.3.0 เปลี่ยนอะไรไปจาก 2.2.1 บ้าง จะต้องไปไล่อ่าน commit log ทั้งหมดระหว่างสอง tag นั้น ซึ่งอาจมีเป็นร้อย commit ที่ปนกันระหว่าง:

- commit ที่มีความหมายกับผู้ใช้จริง ๆ เช่น "เพิ่มฟีเจอร์ dark mode"
- commit ที่เป็นรายละเอียดภายในล้วน ๆ เช่น "fix typo in variable name", "refactor internal helper", "update CI config"
- commit ที่ merge มาจาก branch อื่นซึ่งข้อความไม่สื่อความหมายเลย เช่น "Merge branch 'fix-2' into 'develop'"

การให้ผู้ใช้ไปไล่อ่านสิ่งเหล่านี้เองเป็นเรื่องที่ **ไม่สมเหตุสมผลและเสียเวลามาก** Changelog จึงทำหน้าที่เป็น **ตัวกรองและสรุป** ที่บอกเฉพาะสิ่งที่ผู้ใช้ควรรู้ ในภาษาที่ผู้ใช้เข้าใจได้ ไม่ใช่ภาษาที่นักพัฒนาใช้คุยกันเอง

### ประโยชน์ของ Changelog แยกตามกลุ่มผู้อ่าน

**1. สำหรับผู้ใช้ปลายทาง (End User) หรือทีมที่ใช้ library/API ของคุณ**

- รู้ทันทีว่าอัปเดตเวอร์ชันใหม่แล้วจะได้ฟีเจอร์อะไรเพิ่ม
- รู้ว่ามี **Breaking Change** หรือไม่ ก่อนตัดสินใจอัปเดต จะได้เตรียมแก้โค้ดฝั่งตัวเองล่วงหน้า
- รู้ว่าบั๊กที่เคยรายงานไว้ถูกแก้ในเวอร์ชันไหน โดยไม่ต้องถามทีม support

**2. สำหรับทีมพัฒนาเอง**

- ใช้เป็นหลักฐานอ้างอิงเวลา debug ว่า "บั๊กนี้เริ่มเกิดตั้งแต่เวอร์ชันไหน" — ไล่ดู Changelog เพื่อหาว่ามีการเปลี่ยนแปลงอะไรในช่วงนั้น
- ใช้เป็นข้อมูลประกอบการวางแผน sprint ถัดไป เพราะเห็นภาพรวมว่าทีมทำอะไรไปแล้วบ้าง
- เป็นส่วนหนึ่งของ **Release Process** ที่แสดงความเป็นมืออาชีพขององค์กร (ต่อยอดจากที่เราเรียนเรื่อง Release Management ใน Part 88)

**3. สำหรับทีม Sales, Marketing, Customer Support**

- ทีม Support ใช้ Changelog เพื่อตอบลูกค้าได้ทันทีว่า "ปัญหาที่คุณแจ้งแก้แล้วหรือยัง"
- ทีม Marketing ใช้เนื้อหาจาก Changelog เป็นวัตถุดิบเขียนประกาศฟีเจอร์ใหม่ (release announcement, blog post)
- ทีม Sales ใช้ตอบคำถามลูกค้าองค์กรที่ต้องการรู้ roadmap และความถี่ในการอัปเดตของผลิตภัณฑ์

### Changelog ต่างจาก Commit History และ Release Notes อย่างไร

หลายคนสับสนสามคำนี้ ขอสรุปให้ชัดเจน:

| สิ่งที่เปรียบเทียบ | กลุ่มเป้าหมาย | รายละเอียด | ที่อยู่ |
|---|---|---|---|
| **Commit History** (`git log`) | นักพัฒนา (internal) | ละเอียดทุกการเปลี่ยนแปลง รวมถึงเรื่องเล็ก ๆ น้อย ๆ | อยู่ใน git object database |
| **Changelog** (`CHANGELOG.md`) | ผู้ใช้ + นักพัฒนา | สรุปเฉพาะสิ่งที่มีความหมายกับผู้ใช้ แบ่งตามเวอร์ชัน | ไฟล์ในตัว repository |
| **Release Notes** | ผู้ใช้ + สาธารณะ | คล้าย Changelog แต่มักเขียนในเชิงประกาศ/การตลาดมากกว่า อาจอยู่นอก repository | หน้า Release บน GitHub/GitLab, บล็อก, อีเมลประกาศ |

ในทางปฏิบัติ หลายทีมใช้เนื้อหาเดียวกันสำหรับทั้ง Changelog และ Release Notes เพียงแค่จัดรูปแบบการนำเสนอต่างกันเล็กน้อย — และนี่คือสิ่งที่เครื่องมืออัตโนมัติที่เราจะเรียนใน Step ถัดไปทำให้เราได้โดยแทบไม่ต้องออกแรงเพิ่ม

---

## Step 882: "Keep a Changelog" format มาตรฐานที่นิยมใช้กันทั่วโลก

### ปัญหาก่อนมีมาตรฐาน

ก่อนที่จะมีมาตรฐานกลาง แต่ละโปรเจกต์เขียน Changelog ในรูปแบบที่ต่างกันไปหมด บางที่เรียงจากเก่าไปใหม่ บางที่ใหม่ไปเก่า บางที่ไม่มีวันที่ บางที่ไม่แยกหมวดหมู่ ทำให้ผู้ใช้ที่คุ้นเคยกับโปรเจกต์หนึ่งไปเจออีกโปรเจกต์หนึ่งแล้วงงกับรูปแบบใหม่ทุกครั้ง

**Keep a Changelog** (เว็บไซต์ https://keepachangelog.com) คือข้อเสนอมาตรฐานที่ **Olivier Lacan** เริ่มต้นขึ้น เพื่อให้ทุกโปรเจกต์เขียน Changelog ในรูปแบบเดียวกัน อ่านง่าย เข้าใจตรงกันไม่ว่าจะเป็นโปรเจกต์ไหน

### หลักการสำคัญของ Keep a Changelog

Keep a Changelog วางหลักการไว้ 6 ข้อหลัก:

1. **Changelog มีไว้สำหรับมนุษย์อ่าน ไม่ใช่เครื่องจักร** — ต้องเขียนด้วยภาษาที่คนทั่วไปเข้าใจ ไม่ใช่แค่ dump commit message ดิบ ๆ
2. **ควรมีรายการสำหรับทุกเวอร์ชัน** — ไม่ข้ามเวอร์ชันไหนไป แม้จะเป็นการแก้เล็กน้อยก็ตาม
3. **จัดกลุ่มการเปลี่ยนแปลงประเภทเดียวกันไว้ด้วยกัน** — เช่น bug fix ทั้งหมดอยู่ด้วยกัน ไม่ใช่กระจัดกระจาย
4. **เวอร์ชันและส่วนต่าง ๆ ควรลิงก์ได้** — เช่นลิงก์ไปเปรียบเทียบ diff ระหว่างสองเวอร์ชันบน GitHub ได้โดยตรง
5. **เวอร์ชันล่าสุดอยู่บนสุดเสมอ** (Latest version first) — ตรงข้ามกับ commit log ที่บางครั้งเรียงเก่าสุดไปใหม่สุด
6. **ระบุวันที่ของแต่ละเวอร์ชันอย่างชัดเจน** โดยใช้รูปแบบสากล `YYYY-MM-DD` (ISO 8601) เพื่อไม่ให้สับสนระหว่างรูปแบบวันที่แบบอเมริกัน (MM-DD-YYYY) กับแบบยุโรป (DD-MM-YYYY)

### โครงสร้างไฟล์ตามมาตรฐาน

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- ฟีเจอร์ที่กำลังพัฒนาอยู่ ยังไม่ได้ปล่อยเวอร์ชันจริง

## [1.2.0] - 2026-09-10

### Added
- เพิ่มระบบ export ข้อมูลเป็นไฟล์ CSV

### Changed
- ปรับปรุงความเร็วของหน้า Dashboard ให้โหลดเร็วขึ้น 40%

### Deprecated
- ฟังก์ชัน `getUserData()` จะถูกลบในเวอร์ชัน 2.0.0 ให้ใช้ `fetchUserProfile()` แทน

### Removed
- ลบ endpoint `/api/v1/legacy-export` ที่เลิกใช้งานแล้ว

### Fixed
- แก้ไขบั๊กการคำนวณภาษีมูลค่าเพิ่มผิดพลาดในบางกรณี

### Security
- อัปเดต dependency ที่มีช่องโหว่ CVE-2026-XXXXX

## [1.1.0] - 2026-08-01

### Added
- เพิ่มการรองรับภาษาไทยในหน้า UI ทั้งหมด

[Unreleased]: https://github.com/org/project/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/org/project/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/org/project/releases/tag/v1.1.0
```

### 6 หมวดหมู่มาตรฐานที่ Keep a Changelog กำหนด

Keep a Changelog นิยามหมวดหมู่การเปลี่ยนแปลงไว้ 6 แบบ ซึ่งเราจะเจาะลึกเรื่องการจัดหมวดหมู่อีกครั้งใน Step 888:

| หมวดหมู่ | ความหมาย |
|---|---|
| **Added** | ฟีเจอร์ใหม่ที่เพิ่มเข้ามา |
| **Changed** | การเปลี่ยนแปลงพฤติกรรมของฟีเจอร์ที่มีอยู่แล้ว |
| **Deprecated** | ฟีเจอร์ที่กำลังจะถูกเลิกใช้ในอนาคต (แต่ยังใช้ได้อยู่ตอนนี้) |
| **Removed** | ฟีเจอร์ที่ถูกลบออกไปแล้วจริง ๆ |
| **Fixed** | การแก้ไขบั๊ก |
| **Security** | การแก้ไขช่องโหว่ด้านความปลอดภัย (สำคัญมากที่ต้องแยกหมวดชัดเจน เพื่อให้ผู้ใช้เห็นและอัปเดตโดยด่วน) |

### หมวด [Unreleased] สำคัญอย่างไร

สังเกตว่าตัวอย่างด้านบนมีหมวด `[Unreleased]` อยู่บนสุดเสมอ นี่คือแนวปฏิบัติที่แนะนำให้ทำ: **ทุกครั้งที่มี Pull Request ที่เปลี่ยนแปลงพฤติกรรมของโปรแกรม ให้เพิ่มบรรทัดลงในหมวด `[Unreleased]` ทันที** แทนที่จะรอมาเขียนตอน release

ข้อดีคือ:

- ไม่มีวันลืมว่าอะไรเปลี่ยนไปบ้างระหว่างทาง เพราะบันทึกสด ๆ ตอนที่ merge
- ตอนถึงเวลา release จริง แค่เปลี่ยนหัวข้อ `[Unreleased]` เป็นเลขเวอร์ชันและวันที่ แล้วเปิดหมวด `[Unreleased]` ใหม่ให้ว่างไว้รอรอบถัดไป
- ทีมที่ทำงานพร้อมกันหลายคนจะเห็นว่ามีอะไรสะสมรออยู่ก่อน release ครั้งถัดไปแล้วบ้าง

มาตรฐานนี้ถูกใช้กันอย่างแพร่หลายในโปรเจกต์ Open Source ระดับโลกจำนวนมาก เช่น Vue.js, Angular (ในยุคแรก ๆ ก่อนเปลี่ยนมาใช้ auto-generated), Docker CLI และอีกนับพันโปรเจกต์ อย่างไรก็ตาม ปัญหาที่ตามมาคือ **การรักษาวินัยให้ทุกคนในทีมเพิ่มรายการลงหมวด `[Unreleased]` ทุกครั้งนั้นทำได้ยากในทางปฏิบัติ** — นี่คือเหตุผลที่การ automate เข้ามามีบทบาทสำคัญ ซึ่งเราจะเปรียบเทียบกันใน Step ถัดไป

---

## Step 883: การเขียน Changelog ด้วยมือ vs อัตโนมัติ ข้อดีข้อเสียแต่ละแบบ

เมื่อเข้าใจแล้วว่า Changelog สำคัญและควรมีรูปแบบมาตรฐาน คำถามถัดมาคือ **ใครควรเป็นคนเขียนมัน — คนหรือเครื่องจักร?** คำตอบคือ "ขึ้นอยู่กับบริบท" มาดูรายละเอียดแต่ละแนวทาง

### แนวทางที่ 1: เขียนด้วยมือ (Manual Changelog)

วิธีนี้คือให้นักพัฒนาหรือ maintainer เขียน entry ใน `CHANGELOG.md` เอง ไม่ว่าจะเขียนตอน merge PR (เพิ่มใน `[Unreleased]`) หรือเขียนรวดเดียวตอนใกล้ release

**ข้อดี:**

1. **ควบคุมคุณภาพภาษาได้เต็มที่** — คนเขียนสามารถเลือกใช้คำที่เหมาะกับผู้อ่านเป้าหมาย เล่าเรื่องได้ลื่นไหล ใส่บริบทที่เครื่องจักรมองไม่เห็น เช่น "ฟีเจอร์นี้เกิดจากคำขอของลูกค้าองค์กรหลายรายที่ต้องการ..."
2. **กรองสิ่งที่ไม่สำคัญออกได้แม่นยำ** — คนเขียนรู้ว่า commit ไหนที่ผู้ใช้ไม่จำเป็นต้องรู้เลย เช่น การ refactor ภายในที่ไม่กระทบ behavior ใด ๆ
3. **รวมหลาย commit เป็นเรื่องเดียวได้** — ถ้าฟีเจอร์หนึ่งใช้ 15 commit กว่าจะเสร็จ คนเขียนสรุปเป็นประโยคเดียวได้ ในขณะที่เครื่องมักจะสร้างรายการแยกตาม commit
4. **ไม่ผูกติดกับรูปแบบ commit message ที่เข้มงวด** — ทีมไม่จำเป็นต้องบังคับใช้ Conventional Commits หรือมาตรฐานใด ๆ ก็เขียน Changelog ได้

**ข้อเสีย:**

1. **ต้องพึ่งวินัยของคน** — ถ้าไม่มีใครจำได้ว่าต้องอัปเดต Changelog ตอน merge PR สุดท้าย entry นั้นจะหายไปตลอดกาล เป็นปัญหาที่พบบ่อยมากในทีมที่งานยุ่ง
2. **เสียเวลา** — ยิ่งโปรเจกต์ release บ่อย (เช่น หลายครั้งต่อสัปดาห์แบบ Continuous Delivery) ยิ่งต้องเสียเวลาเขียนซ้ำ ๆ ทุกรอบ
3. **มีโอกาสผิดพลาดจาก human error** — เช่น ลืมใส่บาง breaking change ที่สำคัญมาก จนผู้ใช้อัปเดตแล้วโปรแกรมพัง โดยไม่มีการเตือนล่วงหน้าใน Changelog
4. **ไม่สอดคล้องกับเวอร์ชันที่ปล่อยจริงเสมอไป** — บางครั้งคนเขียน Changelog ไม่ตรงกับเลขเวอร์ชันที่ tag จริง (เช่น บอกว่าเป็น breaking change แต่ tag เป็น patch version) ซึ่งขัดกับหลัก Semantic Versioning ที่เราเรียนใน Part 88

### แนวทางที่ 2: เขียนอัตโนมัติ (Automated Changelog)

วิธีนี้คือใช้เครื่องมือ (เช่น `semantic-release`, `conventional-changelog` ที่เราจะเรียนใน Step ถัดไป) generate Changelog จาก commit message โดยอัตโนมัติ โดยปกติจะต้องพึ่งพา **Conventional Commits** (มาตรฐานการเขียน commit message ที่มีโครงสร้างชัดเจน ซึ่งเราเรียนไปแล้วใน Part 88) เพื่อให้เครื่องจักรแยกแยะได้ว่า commit ไหนเป็น feature, bug fix, breaking change ฯลฯ

**ข้อดี:**

1. **ไม่มีวันลืม** — ทุก commit ที่เข้าเงื่อนไขจะถูกดึงเข้า Changelog อัตโนมัติ ไม่ต้องพึ่งความจำของใคร
2. **ประหยัดเวลามหาศาล** โดยเฉพาะทีมที่ release บ่อย ๆ
3. **สอดคล้องกับเวอร์ชันเสมอ** — เครื่องมือที่ดี (เช่น `semantic-release`) จะคำนวณเลขเวอร์ชันถัดไปจาก commit message เดียวกันกับที่ใช้สร้าง Changelog ทำให้ทั้งสองอย่างตรงกันเสมอ ไม่มีทางหลุดจาก SemVer
4. **บังคับให้ทีมเขียน commit message มีวินัยมากขึ้น** เป็นผลพลอยได้ — เพราะทุกคนรู้ว่า commit message จะถูกแปลงเป็นเอกสารสาธารณะโดยตรง
5. **ผสานเข้ากับ CI/CD ได้อย่างสมบูรณ์** — เป็นส่วนหนึ่งของ pipeline การ release แบบไม่ต้องมีคนกดปุ่มเอง (Continuous Deployment เต็มรูปแบบ)

**ข้อเสีย:**

1. **ภาษาที่ได้มักจะ "ดิบ" กว่า** — Changelog ที่ได้จะใกล้เคียงกับ commit message ตรง ๆ ซึ่งบางครั้งไม่ได้เขียนมาให้ผู้ใช้ปลายทางอ่านโดยตรง แม้จะสื่อความหมายได้ แต่ไม่มีสำนวนหรือบริบทเพิ่มเติม
2. **ต้องบังคับใช้ Conventional Commits อย่างเคร่งครัด** — ถ้าทีมเขียน commit message ไม่ตรงตามรูปแบบ (เช่นลืมใส่ prefix `feat:`, `fix:`) commit นั้นจะไม่ถูกนำเข้า Changelog เลย ทำให้ Changelog ไม่ครบถ้วน
3. **ควบคุมการจัดกลุ่มเรื่องเล่าที่ซับซ้อนได้ยาก** — ถ้าฟีเจอร์หนึ่งประกอบด้วยหลาย commit เครื่องมือมักจะสร้างหลายบรรทัดแยกกัน แทนที่จะสรุปเป็นเรื่องเดียวอย่างที่คนเขียนจะทำได้
4. **Commit ที่ไม่มีความหมายกับผู้ใช้บางครั้งก็หลุดเข้ามา** ถ้าทีมตั้ง commit type ไม่รอบคอบ เช่นใส่ `feat:` กับสิ่งที่จริง ๆ เป็นแค่ internal refactor

### แนวทางที่ 3: แบบผสม (Hybrid) — ทางที่นิยมที่สุดในทางปฏิบัติ

ในโลกจริง หลายทีมมืออาชีพเลือกใช้วิธีผสม กล่าวคือ:

1. ใช้เครื่องมืออัตโนมัติ generate Changelog เบื้องต้นจาก Conventional Commits (ได้โครงร่างและความครบถ้วนแบบอัตโนมัติ)
2. ก่อน publish ให้มีขั้นตอนที่คนมารีวิวและ **ปรับปรุงถ้อยคำ** ในส่วนที่สำคัญ เช่น breaking change หรือฟีเจอร์เด่น ให้อ่านง่ายขึ้นสำหรับผู้ใช้จริง
3. ใช้ field พิเศษใน commit message (เช่น `BREAKING CHANGE:` footer ใน Conventional Commits) เพื่อเพิ่มคำอธิบายที่ละเอียดกว่าปกติสำหรับกรณีที่ต้องการความชัดเจนเป็นพิเศษ

### ตารางสรุปเปรียบเทียบ

| ประเด็น | เขียนด้วยมือ | อัตโนมัติเต็มรูปแบบ |
|---|---|---|
| ความสม่ำเสมอ | ขึ้นกับวินัยของทีม | รับประกันได้ 100% |
| คุณภาพภาษา | ดีกว่า (คนเขียนเอง) | ดิบกว่า ใกล้เคียง commit message |
| ความเร็ว/ต้นทุนเวลา | ช้า ต้องเสียเวลาทุกรอบ | เร็วมาก แทบไม่มีต้นทุนเพิ่ม |
| ความเสี่ยงตกหล่น | สูง (อาจลืม) | ต่ำมาก (ดึงจาก commit ทุกตัว) |
| ต้องมี Convention เข้มงวด | ไม่จำเป็น | จำเป็นมาก (Conventional Commits) |
| เหมาะกับ | โปรเจกต์ release ไม่บ่อย ทีมเล็ก ต้องการคุณภาพงานเขียนสูง | โปรเจกต์ release บ่อย ทีมใหญ่ Open Source ที่มี contributor หลากหลาย |

ในหลักสูตรนี้ เราจะเน้นเจาะลึกแนวทางอัตโนมัติเป็นหลัก เพราะเป็นแนวทางที่ยั่งยืนกว่าในระยะยาว และเป็นทักษะที่ตลาดงานต้องการมากขึ้นเรื่อย ๆ ในยุคที่ CI/CD และ DevOps เป็นมาตรฐานขององค์กรซอฟต์แวร์สมัยใหม่

---

## Step 884: `semantic-release` เจาะลึก — generate Changelog อัตโนมัติจาก Conventional Commits

### ทบทวนความเชื่อมโยงกับ Part 88

ใน Part 88 เราได้เรียนรู้เรื่อง **Semantic Versioning (SemVer)** และ **Conventional Commits** ไปแล้ว รวมถึงแนะนำให้รู้จัก `semantic-release` ในฐานะเครื่องมือที่ทำให้ทั้งกระบวนการตั้งแต่ตัดสินใจเลขเวอร์ชันไปจนถึงการปล่อย release เป็นแบบอัตโนมัติทั้งหมด ใน Step นี้เราจะเจาะลึกเฉพาะบทบาทของมันในฐานะเครื่องมือ **generate Changelog**

### หลักการทำงานของ `semantic-release`

`semantic-release` คือเครื่องมือที่เขียนด้วย Node.js ทำงานเป็นส่วนหนึ่งของ CI/CD pipeline โดยมีหลักการพื้นฐานคือ:

> **อ่าน commit message ทั้งหมดที่เกิดขึ้นนับตั้งแต่ tag เวอร์ชันล่าสุด แล้วตัดสินใจโดยอัตโนมัติว่า: (1) เวอร์ชันถัดไปควรเป็นเลขอะไร (2) Changelog ของเวอร์ชันนี้ควรมีเนื้อหาอะไรบ้าง (3) ควร publish ไปที่ไหนบ้าง**

ทั้งหมดนี้ทำงานจาก **แหล่งข้อมูลเดียวกัน** คือ commit message ที่เขียนตามรูปแบบ Conventional Commits ทำให้เลขเวอร์ชันกับ Changelog **สอดคล้องกันเสมอโดยธรรมชาติ** ไม่มีทางที่ทั้งสองจะขัดแย้งกัน ซึ่งแก้ปัญหาสำคัญที่เราพูดถึงใน Step 883 (Changelog เขียนด้วยมือ อาจไม่ตรงกับเลขเวอร์ชันจริง)

### สถาปัตยกรรมแบบ Plugin

`semantic-release` ออกแบบมาเป็นระบบ **plugin pipeline** ที่แต่ละขั้นตอนของ release ทำงานผ่าน plugin แยกกัน ปลั๊กอินที่เกี่ยวข้องกับ Changelog โดยตรงคือ `@semantic-release/changelog`

Pipeline หลักของ `semantic-release` มี 9 ขั้นตอน:

```
1. verifyConditions  → ตรวจสอบว่า config, credential, token ต่าง ๆ พร้อมหรือไม่
2. getLastRelease     → หา release ล่าสุดจาก git tag
3. analyzeCommits     → วิเคราะห์ commit ตั้งแต่ release ล่าสุด ว่าควร bump version แบบไหน
4. verifyRelease      → ตรวจสอบความถูกต้องของแผน release
5. generateNotes      → สร้างเนื้อหา release notes/changelog จาก commit
6. prepare            → เตรียมไฟล์ก่อน publish เช่นอัปเดต CHANGELOG.md, package.json
7. publish            → เผยแพร่ไปยัง npm, GitHub Releases ฯลฯ
8. addChannel         → จัดการ pre-release channel (beta, alpha)
9. success / fail     → แจ้งผลลัพธ์ เช่น comment กลับใน PR/Issue ที่เกี่ยวข้อง
```

`@semantic-release/changelog` จะทำงานในขั้นตอน `prepare` โดยเขียนเนื้อหาที่ได้จากขั้นตอน `generateNotes` ลงไฟล์ `CHANGELOG.md` โดยตรง

### ตัวอย่างการตั้งค่า `.releaserc.json`

```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    [
      "@semantic-release/changelog",
      {
        "changelogFile": "CHANGELOG.md",
        "changelogTitle": "# Changelog\n\nเอกสารนี้บันทึกการเปลี่ยนแปลงที่สำคัญทั้งหมดของโปรเจกต์"
      }
    ],
    [
      "@semantic-release/npm",
      {
        "npmPublish": true
      }
    ],
    [
      "@semantic-release/git",
      {
        "assets": ["CHANGELOG.md", "package.json"],
        "message": "chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}"
      }
    ],
    "@semantic-release/github"
  ]
}
```

อธิบายแต่ละ plugin:

| Plugin | หน้าที่ |
|---|---|
| `@semantic-release/commit-analyzer` | วิเคราะห์ commit message เพื่อตัดสินใจว่า bump เป็น major/minor/patch |
| `@semantic-release/release-notes-generator` | สร้างเนื้อหา release notes จาก commit (ใช้ `conventional-changelog` เป็น engine เบื้องหลัง) |
| `@semantic-release/changelog` | เขียนเนื้อหาที่ได้ลงไฟล์ `CHANGELOG.md` ใน repository จริง |
| `@semantic-release/npm` | อัปเดตเลขเวอร์ชันใน `package.json` และ publish ไปที่ npm registry |
| `@semantic-release/git` | commit ไฟล์ที่เปลี่ยนแปลง (CHANGELOG.md, package.json) กลับเข้า repository พร้อม tag เวอร์ชันใหม่ |
| `@semantic-release/github` | สร้าง GitHub Release พร้อมแนบเนื้อหา release notes อัตโนมัติ |

### ตัวอย่างผลลัพธ์ที่เกิดขึ้นจริง

สมมติมี commit ต่อไปนี้เกิดขึ้นหลัง release เวอร์ชัน `1.4.0`:

```
feat(auth): เพิ่มระบบ login ด้วย OTP ผ่าน SMS
fix(cart): แก้ไขการคำนวณส่วนลดคูปองผิดพลาดเมื่อใช้หลายใบพร้อมกัน
feat(api)!: เปลี่ยนโครงสร้าง response ของ /api/orders

BREAKING CHANGE: field `total` เปลี่ยนชื่อเป็น `totalAmount` และเปลี่ยนหน่วยจากบาทเป็นสตางค์
```

`semantic-release` จะวิเคราะห์ได้ว่า:

- มี `feat` → ต้อง bump อย่างน้อย **minor**
- มี `fix` → ต้อง bump อย่างน้อย **patch**
- มี `!` และ `BREAKING CHANGE:` footer → ต้อง bump เป็น **major**

เมื่อมี breaking change ปนอยู่ กฎของ SemVer จะชนะเสมอ (major มีลำดับความสำคัญสูงสุด) ดังนั้นเวอร์ชันถัดไปจะกลายเป็น `2.0.0` และ `CHANGELOG.md` จะถูกอัปเดตโดยอัตโนมัติเป็น:

```markdown
## [2.0.0](https://github.com/org/project/compare/v1.4.0...v2.0.0) (2026-09-20)

### ⚠ BREAKING CHANGES

* **api:** field `total` เปลี่ยนชื่อเป็น `totalAmount` และเปลี่ยนหน่วยจากบาทเป็นสตางค์

### Features

* **auth:** เพิ่มระบบ login ด้วย OTP ผ่าน SMS ([a1b2c3d](https://github.com/org/project/commit/a1b2c3d))
* **api:** เปลี่ยนโครงสร้าง response ของ /api/orders ([e4f5g6h](https://github.com/org/project/commit/e4f5g6h))

### Bug Fixes

* **cart:** แก้ไขการคำนวณส่วนลดคูปองผิดพลาดเมื่อใช้หลายใบพร้อมกัน ([i7j8k9l](https://github.com/org/project/commit/i7j8k9l))
```

จะสังเกตได้ว่า **BREAKING CHANGES ถูกดันขึ้นไปอยู่บนสุดเสมอ** เพื่อให้ผู้อ่านเห็นก่อนสิ่งอื่นใด นี่เป็นพฤติกรรม default ของ `conventional-changelog` preset ที่ `semantic-release` ใช้เป็นค่าเริ่มต้น

### ข้อควรระวังสำคัญ

1. **ต้องรัน `semantic-release` ใน CI environment เท่านั้น** ไม่แนะนำให้รันจากเครื่อง local เพราะมันต้องเขียนกลับเข้า repository และ publish จริง จำเป็นต้องมี credential ที่ปลอดภัยอยู่ใน CI secrets (ตามที่เราเรียนเรื่อง secret management ใน Part 78)
2. **branch ที่กำหนดใน `branches` ต้องตรงกับ branch ที่ merge เข้าจริง** เช่นถ้าทีมใช้ `main` เป็น production branch ต้องตั้งค่าให้ตรงกัน ไม่เช่นนั้น release จะไม่ทำงาน
3. **commit message ที่ไม่ตรง Conventional Commits จะถูกมองข้ามไปเงียบ ๆ** — ไม่มี error แต่ก็ไม่ปรากฏใน Changelog เลย ทีมต้องมีวินัยหรือใช้ `commitlint` (ที่เรียนใน Part 88) บังคับตั้งแต่ต้นทาง

---

## Step 885: `conventional-changelog` เครื่องมือเสริมสำหรับ generate Changelog แบบ standalone

### ความแตกต่างจาก `semantic-release`

หลายคนสับสนระหว่าง `semantic-release` กับ `conventional-changelog` เพราะชื่อดูคล้ายกันและทำงานร่วมกันบ่อยมาก แต่จริง ๆ แล้วทั้งสองเป็นเครื่องมือคนละระดับ:

- **`semantic-release`** คือ **orchestrator** ที่ดูแลทั้งกระบวนการ release: ตัดสินใจเวอร์ชัน, สร้าง changelog, publish, สร้าง git tag, สร้าง GitHub Release ฯลฯ แบบครบวงจร อัตโนมัติเต็มรูปแบบ ไม่ต้องมีคนกดปุ่มเลย
- **`conventional-changelog`** คือ **library/CLI เฉพาะทาง** ที่ทำหน้าที่ **แค่ generate เนื้อหา Changelog** จาก Conventional Commits เท่านั้น ไม่ยุ่งกับการ publish หรือตัดสินใจเวอร์ชันเอง (แม้จะมี helper ช่วยคำนวณ version ได้แต่ไม่ใช่ focus หลัก)

พูดง่าย ๆ ว่า `conventional-changelog` คือ **เครื่องมือระดับล่างที่เน้นความยืดหยุ่น** เหมาะกับทีมที่ต้องการควบคุมขั้นตอนอื่น ๆ ของ release เอง (เช่น publish เอง, ตัดสินใจ merge/tag เอง) แต่แค่อยากให้การเขียน Changelog เป็นอัตโนมัติ

จริง ๆ แล้ว `semantic-release` เองก็ใช้ `conventional-changelog` (ผ่าน `@semantic-release/release-notes-generator`) เป็น engine เบื้องหลังในการสร้างเนื้อหาเช่นกัน — จึงเรียกได้ว่า `conventional-changelog` เป็นรากฐานร่วมของทั้งสอง ecosystem

### วิธีใช้งานแบบ standalone

**การติดตั้ง:**

```bash
npm install --save-dev conventional-changelog-cli
```

**การรันแบบพื้นฐาน (generate ครั้งแรกจาก commit ทั้งหมดในประวัติ):**

```bash
npx conventional-changelog -p angular -i CHANGELOG.md -s -r 0
```

อธิบาย flag:

| Flag | ความหมาย |
|---|---|
| `-p angular` | ใช้ preset ชื่อ "angular" (รูปแบบ commit convention ที่ทีม Angular กำหนดไว้ ซึ่งเป็นต้นกำเนิดของ Conventional Commits) |
| `-i CHANGELOG.md` | ไฟล์ input ที่จะอ่าน/เขียนทับ |
| `-s` | (same file) เขียนผลลัพธ์กลับเข้าไฟล์เดิม แทนที่จะ print ออกทาง stdout |
| `-r 0` | สร้างใหม่ทั้งหมดจากทุก release ในประวัติ (release count = 0 หมายถึงไม่จำกัด) |

**การรันสำหรับ release ครั้งถัดไปเท่านั้น (ใช้บ่อยที่สุดในการทำงานจริง):**

```bash
npx conventional-changelog -p angular -i CHANGELOG.md -s
```

คำสั่งนี้จะอ่านเฉพาะ commit ใหม่ที่เกิดขึ้นตั้งแต่ entry ล่าสุดใน `CHANGELOG.md` แล้วเติมเข้าไปด้านบนไฟล์เดิม โดยไม่แตะเนื้อหาเก่าที่มีอยู่แล้ว

### Preset ต่าง ๆ ที่มีให้เลือก

`conventional-changelog` รองรับหลาย preset ตามแต่ละ community นิยม:

| Preset | ใช้โดย |
|---|---|
| `angular` | มาตรฐานดั้งเดิมจากทีม Angular ต้นกำเนิดของ Conventional Commits |
| `conventionalcommits` | ตาม spec Conventional Commits 1.0.0 อย่างเป็นทางการ (แนะนำให้ใช้ในโปรเจกต์ใหม่) |
| `eslint` | รูปแบบที่ทีม ESLint ใช้ |
| `atom` | รูปแบบที่ editor Atom เคยใช้ |
| `jshint` | รูปแบบของ JSHint |

การเลือก preset มีผลต่อว่าเครื่องมือจะจับ pattern อะไรจาก commit message และจัดกลุ่มหัวข้ออย่างไร ทีมส่วนใหญ่ในปัจจุบันแนะนำให้ใช้ `conventionalcommits` เพราะตรงตาม spec สากลที่สุดและรองรับ configuration ที่ยืดหยุ่นกว่า preset อื่น

### การปรับแต่ง config ละเอียด (`.versionrc` หรือ config object)

ถ้าต้องการปรับแต่งชื่อหัวข้อ หรือเพิ่ม/ลบ commit type ที่จะแสดงใน Changelog สามารถสร้างไฟล์ config เองได้ เช่นเมื่อใช้คู่กับเครื่องมือ `standard-version` (อีกเครื่องมือหนึ่งที่ห่อ `conventional-changelog` ไว้อีกชั้น):

```json
{
  "types": [
    { "type": "feat", "section": "🚀 ฟีเจอร์ใหม่" },
    { "type": "fix", "section": "🐛 แก้ไขบั๊ก" },
    { "type": "perf", "section": "⚡ ปรับปรุงประสิทธิภาพ" },
    { "type": "docs", "section": "📝 เอกสาร", "hidden": false },
    { "type": "chore", "hidden": true },
    { "type": "refactor", "hidden": true },
    { "type": "test", "hidden": true }
  ]
}
```

ในตัวอย่างนี้ กำหนดให้ `feat`, `fix`, `perf`, `docs` แสดงในหมวดของตัวเอง (พร้อม emoji ประกอบความสวยงาม) ส่วน `chore`, `refactor`, `test` จะถูกซ่อนไม่แสดงใน Changelog เลย (`hidden: true`) เพราะถือว่าเป็นรายละเอียดภายในที่ผู้ใช้ไม่จำเป็นต้องรู้

### เมื่อไหร่ควรใช้ `conventional-changelog` แบบ standalone แทน `semantic-release`

- เมื่อทีมมี process การ release ที่ซับซ้อนกว่ามาตรฐาน เช่นต้องผ่านการอนุมัติจากหลายฝ่ายก่อน publish จริง ไม่ต้องการให้ทุกอย่างอัตโนมัติ 100%
- เมื่อทีมต้องการควบคุมจังหวะการ publish เอง (เช่น publish เฉพาะวันอังคาร-พฤหัสบดี ตามนโยบายบริษัท)
- เมื่อทีมมี pipeline การ deploy ที่ซับซ้อนอยู่แล้วและแค่ต้องการปลั๊กชิ้นส่วน "generate changelog" เข้าไปแทนที่จะเปลี่ยนทั้ง pipeline

---

## Step 886: Release Please ของ Google — ทางเลือกอื่นในการ automate release + changelog

### แนวคิดที่ต่างจาก `semantic-release`

**Release Please** เป็นเครื่องมือ Open Source ที่พัฒนาโดยทีมงาน Google ใช้แนวคิดที่ต่างจาก `semantic-release` อย่างมีนัยสำคัญ แม้จะแก้ปัญหาเดียวกันคือ "automate release และ changelog จาก Conventional Commits"

ความต่างหลักคือเรื่อง **จังหวะเวลาที่เกิดการเปลี่ยนแปลงจริงในโค้ด**:

- **`semantic-release`**: เมื่อ merge เข้า main branch ปุ๊บ จะ **release ทันที** ในรอบ CI เดียวกันแบบไม่มีการหยุดรอ (fully automated, ไม่มี human gate)
- **Release Please**: เมื่อ merge PR ที่มี Conventional Commits เข้า main แทนที่จะ release ทันที มันจะ **เปิด (หรืออัปเดต) Pull Request พิเศษที่ชื่อว่า "Release PR"** ซึ่งรวบรวมการเปลี่ยนแปลงเวอร์ชันและ Changelog ที่จะเกิดขึ้นเอาไว้ล่วงหน้า รอให้ทีมกด **merge PR นั้นเมื่อพร้อมจะ release จริง ๆ**

### วิธีการทำงานแบบละเอียด

```
1. นักพัฒนา merge PR ปกติเข้า main (commit ใช้ Conventional Commits)
                          │
                          ▼
2. Release Please Action ทำงาน ตรวจพบว่ามี commit ใหม่
                          │
                          ▼
3. สร้าง/อัปเดต "Release PR" อัตโนมัติ ซึ่งมีการเปลี่ยนแปลง:
   - อัปเดตเลขเวอร์ชันในไฟล์ที่เกี่ยวข้อง (package.json, version.py, Cargo.toml ฯลฯ)
   - เพิ่มเนื้อหาใหม่ใน CHANGELOG.md
                          │
                          ▼
4. ทีม (หรือ maintainer) รีวิว Release PR นี้
   - อ่านทวนว่า Changelog ที่สร้างมาถูกต้องหรือไม่
   - แก้ไขถ้อยคำเพิ่มเติมได้ถ้าต้องการ (มันคือ PR ปกติที่แก้ไขได้)
                          │
                          ▼
5. เมื่อพร้อม กด Merge Release PR
                          │
                          ▼
6. Release Please สร้าง Git Tag, GitHub Release จริง ณ จุดนี้
```

### ตัวอย่างการตั้งค่าด้วย GitHub Actions

```yaml
# .github/workflows/release-please.yml
name: release-please

on:
  push:
    branches:
      - main

permissions:
  contents: write
  pull-requests: write

jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
        with:
          release-type: node
          config-file: release-please-config.json
          manifest-file: .release-please-manifest.json
```

ไฟล์ config เพิ่มเติม:

```json
// release-please-config.json
{
  "packages": {
    ".": {
      "release-type": "node",
      "changelog-path": "CHANGELOG.md"
    }
  }
}
```

```json
// .release-please-manifest.json
{
  ".": "1.4.0"
}
```

### ข้อดีของแนวทาง Release Please เมื่อเทียบกับ `semantic-release`

1. **มี Human Gate ก่อน release จริง** — เหมาะกับทีมที่ยังไม่พร้อมให้ทุกอย่างอัตโนมัติ 100% ต้องการให้คนตรวจทานก่อนเสมอ ลดความเสี่ยงที่ commit message ผิดพลาดจะทำให้ release โดยไม่ได้ตั้งใจ
2. **เห็นภาพรวมล่วงหน้าได้ง่าย** — Release PR ทำหน้าที่เป็น "preview" ของสิ่งที่จะเกิดขึ้น ทีมเห็นได้ตลอดเวลาว่าตอนนี้สะสมอะไรไว้รอ release บ้าง โดยไม่ต้องรันคำสั่งอะไรเพิ่ม
3. **รองรับ Monorepo ได้ดีในตัว** — Release Please ออกแบบมาให้จัดการหลาย package ในที่เดียวได้ตั้งแต่ต้น (จะเจาะลึกเรื่อง monorepo ใน Step 889)
4. **เขียนด้วยภาษาและดูแลโดยทีม Google** ซึ่งใช้งานจริงกับผลิตภัณฑ์ Open Source จำนวนมากของ Google เอง เช่น client library ต่าง ๆ ของ Google Cloud

### ข้อควรพิจารณา

1. **ไม่ใช่ fully automated แบบ hands-off 100%** เพราะยังต้องมีคนมา merge Release PR ทุกครั้ง (ซึ่งบางทีมมองว่าเป็นข้อดี บางทีมมองว่าเป็นภาระเพิ่ม)
2. **ระบบนิเวศ plugin ไม่กว้างเท่า `semantic-release`** — `semantic-release` มี community plugin จำนวนมากสำหรับ publish ไปยังปลายทางหลากหลาย (npm, Docker Hub, PyPI ฯลฯ) ในขณะที่ Release Please เน้นที่การจัดการเวอร์ชันและ Changelog เป็นหลัก การ publish จริงมักต้องเขียน step เพิ่มเองใน workflow

### ตารางเปรียบเทียบสรุป

| ประเด็น | `semantic-release` | Release Please |
|---|---|---|
| จังหวะ release | ทันทีเมื่อ merge เข้า main | รอ merge "Release PR" ก่อน |
| Human review ก่อน release | ไม่มี (fully automated) | มี (ผ่าน PR review) |
| รองรับ Monorepo | ต้องพึ่ง plugin เสริม | รองรับในตัวตั้งแต่แรก |
| Ecosystem publish plugin | กว้างขวางมาก | จำกัดกว่า เน้น version+changelog |
| เหมาะกับ | ทีมที่มั่นใจใน CI เต็มที่ ต้องการ Continuous Deployment แท้จริง | ทีมที่ต้องการจุดตรวจสอบก่อน release เสมอ |

---

## Step 887: GitHub Auto-generated Release Notes ฟีเจอร์ในตัว

### ฟีเจอร์ที่มีอยู่แล้วในตัว GitHub โดยไม่ต้องติดตั้งอะไรเพิ่ม

นอกจากเครื่องมือภายนอกอย่าง `semantic-release` และ Release Please แล้ว **GitHub เองก็มีฟีเจอร์ auto-generate release notes ในตัว** ที่ใช้งานได้ทันทีโดยไม่ต้องติดตั้ง dependency ใด ๆ เพิ่มเลย เหมาะมากสำหรับทีมขนาดเล็กหรือโปรเจกต์ที่ไม่ต้องการความซับซ้อนของเครื่องมือภายนอก

### วิธีใช้งานผ่านหน้าเว็บ GitHub

เมื่อสร้าง Release ใหม่ในหน้า **Releases** ของ repository (ไปที่ `https://github.com/<org>/<repo>/releases/new`) จะมีปุ่ม **"Generate release notes"** ให้กดที่มุมของฟอร์ม

เมื่อกดปุ่มนี้ GitHub จะ:

1. หา tag/release ก่อนหน้าเพื่อใช้เป็นจุดเริ่มเปรียบเทียบ
2. ดึงรายการ **Pull Request ทั้งหมด** (ไม่ใช่ commit ดิบ ๆ) ที่ถูก merge เข้า branch เป้าหมายนับตั้งแต่ release ก่อนหน้า
3. จัดกลุ่มตาม **label** ของแต่ละ PR (เช่น label `enhancement`, `bug`, `documentation`)
4. ใส่รายชื่อ **contributor** ที่มีส่วนร่วมใน release นี้ พร้อมระบุ **"First-time contributor"** ถ้าเป็นคนที่เพิ่ง contribute ครั้งแรกในโปรเจกต์ — เป็นฟีเจอร์ที่ทีม Open Source ชื่นชอบมากเพราะช่วยขอบคุณผู้มีส่วนร่วมใหม่ ๆ ได้อัตโนมัติ
5. สร้างลิงก์ **"Full Changelog"** ที่ลิงก์ไปหน้า compare view ระหว่างสอง tag โดยอัตโนมัติ

### ตัวอย่างผลลัพธ์ที่ได้

```markdown
## What's Changed
* feat: เพิ่มระบบแจ้งเตือนผ่าน Line Notify by @somchai in https://github.com/org/repo/pull/142
* fix: แก้ไขปัญหาการ login ซ้ำซ้อนเมื่อ token หมดอายุ by @suda in https://github.com/org/repo/pull/145
* docs: ปรับปรุงคำอธิบาย API ให้ครบถ้วนขึ้น by @arthit in https://github.com/org/repo/pull/147

## New Contributors
* @pranee made their first contribution in https://github.com/org/repo/pull/144

**Full Changelog**: https://github.com/org/repo/compare/v1.3.0...v1.4.0
```

### การปรับแต่งด้วยไฟล์ `.github/release.yml`

GitHub อนุญาตให้ปรับแต่งการจัดหมวดหมู่ของ auto-generated release notes ผ่านไฟล์ config `.github/release.yml`:

```yaml
changelog:
  exclude:
    labels:
      - ignore-for-release
      - duplicate
    authors:
      - dependabot
      - octokit-bot
  categories:
    - title: 🚨 Breaking Changes
      labels:
        - breaking-change
    - title: 🚀 ฟีเจอร์ใหม่
      labels:
        - enhancement
        - feature
    - title: 🐛 แก้ไขบั๊ก
      labels:
        - bug
        - bugfix
    - title: 📝 เอกสาร
      labels:
        - documentation
    - title: 🔧 อื่น ๆ
      labels:
        - "*"
```

จากตัวอย่างนี้:

- PR ที่มี label `dependabot` หรือมาจาก bot account `dependabot` จะถูกกันออกจาก release notes ไปเลย (มีประโยชน์มากเพราะ dependency update อัตโนมัติมักมี PR จำนวนมากที่ไม่ควรไปปนกับ Changelog หลัก)
- PR ที่มี label `breaking-change` จะถูกจัดกลุ่มไว้บนสุดในหมวด "Breaking Changes"
- หมวดสุดท้าย `"*"` คือ catch-all สำหรับ PR ที่ไม่ตรง label ไหนเลย ให้จัดไว้ในหมวด "อื่น ๆ"

### การเรียกใช้งานผ่าน GitHub API / CLI (สำหรับ automate ต่อใน pipeline)

นอกจากกดปุ่มในหน้าเว็บ ยังสามารถเรียกผ่าน GitHub CLI ได้ ทำให้ผสานเข้ากับ script อัตโนมัติได้:

```bash
gh api repos/{owner}/{repo}/releases/generate-notes \
  -f tag_name='v1.4.0' \
  -f previous_tag_name='v1.3.0'
```

หรือใช้ตอนสร้าง release ผ่าน `gh release create` โดยตรง:

```bash
gh release create v1.4.0 --generate-notes
```

คำสั่งนี้จะสร้าง GitHub Release พร้อมเนื้อหาที่ auto-generate ให้ทันที เหมาะมากสำหรับใส่ไว้เป็นขั้นตอนหนึ่งใน GitHub Actions workflow

### ข้อดีและข้อจำกัดเมื่อเทียบกับเครื่องมือภายนอก

**ข้อดี:**

- ไม่ต้องติดตั้งอะไรเพิ่มเลย ใช้งานได้ทันทีในทุก repository บน GitHub
- อิงจาก PR และ label แทน commit message ดิบ ทำให้ผลลัพธ์ตรงกับ "หน่วยงาน" ที่ทีมทำงานจริง (PR คือหน่วยของงานที่ถูกรีวิวแล้ว)
- แสดง contributor และ first-time contributor อัตโนมัติ ซึ่งเครื่องมืออื่นไม่มีฟีเจอร์นี้ในตัว

**ข้อจำกัด:**

- **ไม่ได้เขียนไฟล์ `CHANGELOG.md` ในตัว repository ให้อัตโนมัติ** — มันสร้างแค่เนื้อหาในหน้า GitHub Release เท่านั้น ถ้าต้องการให้มีไฟล์ `CHANGELOG.md` ในโค้ดด้วย ต้อง copy มาเองหรือเขียน script ดึงผ่าน API มาบันทึกเพิ่ม
- **ไม่คำนวณเลขเวอร์ชันให้** — ทีมยังต้องตัดสินใจเลข version เอง (ต่างจาก `semantic-release` และ Release Please ที่คำนวณให้อัตโนมัติ)
- **การจัดกลุ่มต้องพึ่งวินัยการติด label ของทีม** ถ้าทีมไม่ติด label ให้ PR อย่างสม่ำเสมอ ผลลัพธ์ที่ได้จะแบนราบไม่มีหมวดหมู่ที่ชัดเจน ต่างจาก Conventional Commits ที่จัดหมวดจาก syntax ของ commit message โดยตรง
- **ผูกกับ GitHub เท่านั้น** ถ้าองค์กรใช้ GitLab หรือ self-hosted Git server อื่น จะไม่มีฟีเจอร์นี้ให้ใช้ (GitLab มีฟีเจอร์คล้ายกันในชื่อ "Release" แต่กลไกรายละเอียดต่างออกไป)

---

## Step 888: การจัดหมวดหมู่ Changelog ให้อ่านง่าย

### ทำไมการจัดหมวดหมู่จึงสำคัญกว่าที่คิด

Changelog ที่ดีไม่ใช่แค่ "รายการที่ครบถ้วน" แต่ต้อง **อ่านง่ายและหาสิ่งที่ต้องการเจอเร็ว** ลองเปรียบเทียบสองตัวอย่างนี้:

**แบบที่ไม่จัดหมวดหมู่ (อ่านยาก):**

```markdown
## [2.0.0] - 2026-09-20
- เพิ่มระบบ login ด้วย OTP
- เปลี่ยนโครงสร้าง response ของ /api/orders (breaking change)
- แก้ไขการคำนวณส่วนลดคูปอง
- อัปเดตเอกสาร API
- แก้ไขช่องโหว่ SQL Injection ใน endpoint ค้นหา
- ปรับปรุงความเร็วหน้า Dashboard
```

**แบบที่จัดหมวดหมู่ (อ่านง่าย หาข้อมูลเร็ว):**

```markdown
## [2.0.0] - 2026-09-20

### ⚠ Breaking Changes
- เปลี่ยนโครงสร้าง response ของ `/api/orders`: field `total` เปลี่ยนเป็น `totalAmount`

### Added
- เพิ่มระบบ login ด้วย OTP ผ่าน SMS

### Changed
- ปรับปรุงความเร็วหน้า Dashboard ให้โหลดเร็วขึ้น 40%

### Fixed
- แก้ไขการคำนวณส่วนลดคูปองผิดพลาดเมื่อใช้หลายใบพร้อมกัน

### Security
- แก้ไขช่องโหว่ SQL Injection ใน endpoint ค้นหา

### Documentation
- อัปเดตเอกสาร API ให้ครบถ้วนขึ้น
```

ผู้ใช้ที่กังวลเรื่องความปลอดภัยสามารถกวาดตาไปที่หมวด **Security** ได้ทันที โดยไม่ต้องอ่านทุกบรรทัด ทีม DevOps ที่ต้องวางแผนอัปเดตระบบสามารถโฟกัสที่หมวด **Breaking Changes** ได้ก่อนอันดับแรกเสมอ

### หลักการจัดลำดับหมวดหมู่ (Priority Order)

แนวทางที่แนะนำคือเรียงหมวดหมู่ตามลำดับความสำคัญที่ผู้ใช้ควรอ่านก่อน-หลัง ไม่ใช่เรียงตามตัวอักษร:

1. **Breaking Changes / ⚠ BREAKING CHANGES** — สำคัญที่สุดเสมอ ต้องอยู่บนสุด เพราะกระทบการอัปเดตโดยตรง
2. **Security** — ความปลอดภัยสำคัญรองลงมา ผู้ใช้ต้องรู้เร็วที่สุดเพื่อป้องกันความเสี่ยง
3. **Added / Features** — ฟีเจอร์ใหม่ที่ผู้ใช้น่าจะสนใจอยากรู้
4. **Changed** — พฤติกรรมเดิมที่เปลี่ยนไป (ไม่ถึงกับ breaking แต่ควรรู้)
5. **Deprecated** — สิ่งที่กำลังจะเลิกใช้ในอนาคต เตือนล่วงหน้า
6. **Fixed / Bug Fixes** — การแก้บั๊ก
7. **Removed** — สิ่งที่ถูกลบไปแล้วจริง
8. **Documentation** — การปรับปรุงเอกสาร (สำคัญน้อยสุดในมุมผู้ใช้ทั่วไป แต่ยังมีประโยชน์)

### การ mapping ระหว่าง Conventional Commits type กับหมวดหมู่ Changelog

เมื่อใช้เครื่องมืออัตโนมัติ สิ่งสำคัญคือต้องกำหนด mapping ที่ชัดเจนระหว่าง **commit type** (จาก Conventional Commits) กับ **หมวดหมู่ Changelog**:

| Commit Type | หมวดหมู่ Changelog | แสดงหรือซ่อน |
|---|---|---|
| `feat` | Added / Features | แสดง |
| `fix` | Fixed / Bug Fixes | แสดง |
| `perf` | Performance Improvements | แสดง |
| มี `!` หรือ footer `BREAKING CHANGE:` | ⚠ Breaking Changes | แสดง (ดันขึ้นบนสุด) |
| commit ที่มีคำว่า "security", "CVE", "vulnerability" ใน scope/description | Security | แสดง (มักต้องกำกับด้วยมือเพิ่มเติม เพราะ Conventional Commits ไม่มี type `security` มาตรฐาน) |
| `docs` | Documentation | แสดง (บางทีมเลือกซ่อน) |
| `style` | — | ซ่อน (แค่จัดรูปแบบโค้ด ไม่กระทบผู้ใช้) |
| `refactor` | — | ซ่อน (ปรับโครงสร้างภายใน ไม่กระทบพฤติกรรม) |
| `test` | — | ซ่อน (เพิ่ม/แก้ test เท่านั้น) |
| `chore` | — | ซ่อน (งานจุกจิก เช่นอัปเดต dependency, ปรับ config) |
| `ci` | — | ซ่อน (เปลี่ยนแค่ CI pipeline) |
| `build` | — | ซ่อน (เปลี่ยนระบบ build เท่านั้น) |

**ข้อสังเกตสำคัญ:** Conventional Commits มาตรฐานไม่มี type สำหรับ `security` โดยตรง ทีมที่ต้องการหมวด Security แยกชัดเจน (ซึ่งแนะนำอย่างยิ่งสำหรับโปรเจกต์ที่มีความเสี่ยงด้านความปลอดภัย) มักจะใช้วิธีใดวิธีหนึ่งต่อไปนี้:

1. ใช้ scope พิเศษ เช่น `fix(security): ...` แล้วเขียน config ให้ดักจับ scope นี้ไปไว้หมวด Security โดยเฉพาะ
2. ใช้ label บน GitHub PR (เช่น label `security`) ควบคู่กับ GitHub auto-generated release notes ที่เรียนใน Step 887
3. เขียนเพิ่มด้วยมือในขั้นตอนรีวิวก่อน publish (ตามแนวทาง Hybrid ที่กล่าวถึงใน Step 883)

### หลักการเขียนแต่ละบรรทัดให้ผู้ใช้เข้าใจ (ไม่ใช่แค่ copy commit message)

แม้จะ automate การจัดหมวดหมู่แล้ว การเขียนคำอธิบายในแต่ละ commit ก็ยังควรตั้งใจเขียนให้ผู้ใช้เข้าใจได้ตั้งแต่ต้นทาง:

**ไม่ควรเขียนแบบนี้ (เจาะจงเทคนิคเกินไป มองจากมุมผู้พัฒนา):**
```
fix: null pointer ใน OrderService.calculateTotal() line 245
```

**ควรเขียนแบบนี้แทน (บอกผลกระทบที่ผู้ใช้สัมผัสได้):**
```
fix(cart): แก้ไขปัญหาแอปค้างเมื่อลบสินค้าชิ้นสุดท้ายออกจากตะกร้า
```

หลักการคือ **เขียนจากมุมมองว่า "ผู้ใช้จะได้อะไร หรือปัญหาอะไรถูกแก้ไป" ไม่ใช่ "โค้ดตรงไหนถูกแก้"** — นี่คือทักษะสำคัญที่ต้องฝึกฝนไม่ว่าจะเขียน Changelog ด้วยมือหรือปล่อยให้เครื่องมือ generate ให้ก็ตาม เพราะต้นทางของทุกอย่างคือ commit message ที่นักพัฒนาเขียนเอง

---

## Step 889: Changelog สำหรับ Monorepo ที่มีหลาย Package พร้อมกัน (แนวคิดของ Changesets)

### ปัญหาเฉพาะของ Monorepo

ใน Part 82 เราเรียนเรื่อง Monorepo vs Polyrepo ไปแล้ว รู้ว่า Monorepo คือ repository เดียวที่รวมหลาย package/project ไว้ด้วยกัน (เช่น React ecosystem ที่มี `react`, `react-dom`, `react-native` อยู่ใน repo เดียวกัน)

ปัญหาที่เกิดขึ้นกับ Changelog ใน Monorepo คือ:

1. **แต่ละ package อาจ release ด้วยเวอร์ชันไม่พร้อมกัน** — package A อาจอยู่ที่ v3.2.0 ในขณะที่ package B ในโปรเจกต์เดียวกันอยู่ที่ v1.0.5 (independent versioning)
2. **commit หนึ่งครั้งอาจกระทบหลาย package พร้อมกัน** — เช่นแก้ shared utility ที่ package A และ B ใช้ร่วมกัน แล้วต้องรู้ว่าควรขึ้นเวอร์ชันทั้งสอง package หรือแค่ package ใดบ้าง
3. **`semantic-release` และ `conventional-changelog` แบบดั้งเดิมออกแบบมาสำหรับ repo เดียว/package เดียวเป็นหลัก** ทำให้ใช้กับ Monorepo ตรง ๆ ได้ลำบาก ต้องพึ่ง plugin เสริม เช่น `semantic-release-monorepo` ซึ่งซับซ้อนและมีข้อจำกัดหลายอย่าง

### แนวคิดของ Changesets

**Changesets** คือเครื่องมือ (จากทีม Atlassian เดิม ปัจจุบันดูแลโดยชุมชน Open Source) ที่ออกแบบมาสำหรับแก้ปัญหา Changelog และ versioning ใน Monorepo โดยเฉพาะ แนวคิดหลักต่างจาก Conventional Commits + `semantic-release` อย่างสิ้นเชิง

**แทนที่จะอ่านความหมายจาก commit message** เหมือน `semantic-release` Changesets ใช้วิธี **ให้นักพัฒนาเขียนไฟล์ "changeset" แยกต่างหากในทุก PR ที่มีการเปลี่ยนแปลงที่ผู้ใช้ควรรู้**

### ขั้นตอนการทำงานของ Changesets

**1. นักพัฒนารัน CLI เพื่อสร้าง changeset ใหม่ ระหว่างทำงานบน PR:**

```bash
npx changeset
```

คำสั่งนี้จะถามคำถามแบบ interactive:

```
🦋  Which packages would you like to include?
◉ @myorg/ui-components
◯ @myorg/api-client
◯ @myorg/utils

🦋  Which packages should have a major bump?
◯ @myorg/ui-components

🦋  Which packages should have a minor bump?
◉ @myorg/ui-components

🦋  Please enter a summary for this change
เพิ่ม component ใหม่ <DatePicker /> รองรับการเลือกช่วงวันที่
```

**2. คำตอบจะถูกเขียนเป็นไฟล์ markdown เล็ก ๆ ไว้ในโฟลเดอร์ `.changeset/`:**

```markdown
---
"@myorg/ui-components": minor
---

เพิ่ม component ใหม่ `<DatePicker />` รองรับการเลือกช่วงวันที่
```

ไฟล์นี้จะถูก commit เข้าไปพร้อมกับโค้ดใน PR เดียวกัน — เท่ากับว่า **การเปลี่ยนแปลงที่มีความหมาย** ถูกบันทึกไว้ ณ เวลาที่นักพัฒนายังจำบริบทได้แม่นที่สุด (ตอนกำลังเขียน PR) ไม่ต้องรอมาเดาทีหลังจาก commit message

**3. เมื่อสะสม changeset จากหลาย PR แล้ว รันคำสั่งเพื่อรวมเป็น version bump จริง:**

```bash
npx changeset version
```

คำสั่งนี้จะ:

- อ่านไฟล์ทั้งหมดใน `.changeset/`
- คำนวณเวอร์ชันใหม่ของแต่ละ package ตาม bump type ที่ระบุ (major/minor/patch)
- **จัดการ dependency ระหว่าง package ภายใน monorepo เดียวกันโดยอัตโนมัติ** — เช่นถ้า `@myorg/ui-components` ขึ้นเวอร์ชัน และ package อื่นใน monorepo depend อยู่ ก็จะช่วยอัปเดต dependency version ให้ตรงกันด้วย
- เขียนเนื้อหาเข้า `CHANGELOG.md` ของ **แต่ละ package แยกกัน**
- ลบไฟล์ changeset ที่ใช้ไปแล้วออกจาก `.changeset/`

**4. Publish ทุก package ที่มีการเปลี่ยนแปลงเวอร์ชัน:**

```bash
npx changeset publish
```

### ตัวอย่างผลลัพธ์ CHANGELOG.md ต่อ package

```
packages/
├── ui-components/
│   ├── CHANGELOG.md   ← เก็บประวัติเฉพาะของ package นี้เท่านั้น
│   └── package.json
├── api-client/
│   ├── CHANGELOG.md   ← เก็บประวัติเฉพาะของ package นี้เท่านั้น
│   └── package.json
└── utils/
    ├── CHANGELOG.md
    └── package.json
```

แต่ละ package มี Changelog เป็นของตัวเอง ทำให้ผู้ใช้ที่ install แค่ `@myorg/api-client` เพียงตัวเดียว สามารถอ่าน Changelog ที่เกี่ยวข้องกับตัวเองล้วน ๆ โดยไม่ต้องกรองเนื้อหาของ package อื่นที่ไม่เกี่ยวข้องออกเอง

### เปรียบเทียบ Changesets กับแนวทาง Conventional Commits + `semantic-release`

| ประเด็น | Conventional Commits + `semantic-release` | Changesets |
|---|---|---|
| แหล่งข้อมูล | วิเคราะห์จาก commit message | ไฟล์ markdown แยกที่นักพัฒนาเขียนเอง |
| เหมาะกับ | Single package repo เป็นหลัก | Monorepo หลาย package โดยเฉพาะ |
| ความชัดเจนของเนื้อหา | ขึ้นกับคุณภาพ commit message | ชัดเจนกว่า เพราะเขียนแยกเป็นประโยคอธิบายเฉพาะ ไม่ปนกับ commit อื่น |
| การจัดการ inter-package dependency | ไม่มีในตัว ต้องพึ่ง plugin เสริม | มีในตัว จัดการให้อัตโนมัติ |
| ภาระของนักพัฒนา | แค่เขียน commit message ให้ถูกรูปแบบ | ต้องรัน `npx changeset` เพิ่มทุกครั้งที่มีการเปลี่ยนแปลงที่มีความหมาย |
| ตัวอย่างผู้ใช้งานจริง | Angular, Docker CLI (บางส่วน) | React ecosystem บางส่วน, Chakra UI, Remix |

### เมื่อไหร่ควรเลือกใช้ Changesets

- เมื่อโปรเจกต์เป็น Monorepo ที่มีหลาย package ต้อง publish แยกกันไปยัง npm registry
- เมื่อทีมมี package หลายตัวที่ independent version กัน ไม่ต้องการให้ทุก package ขึ้นเวอร์ชันพร้อมกันเสมอ
- เมื่อทีมต้องการความชัดเจนของคำอธิบายใน Changelog มากกว่าที่ commit message ดิบ ๆ จะให้ได้ โดยยอมรับภาระเพิ่มเติมที่ต้องรัน CLI สร้าง changeset ทุกครั้ง

ทั้งนี้ ทีมบางส่วนก็เลือกผสมทั้งสองแนวทางเข้าด้วยกัน เช่น ยังคงใช้ Conventional Commits สำหรับวินัยการเขียน commit message ที่ดี แต่ใช้ Changesets เป็นกลไกหลักในการ generate Changelog และตัดสินใจเวอร์ชันสำหรับ Monorepo โดยเฉพาะ

---

## Step 890: แบบฝึกหัด — ตั้งค่า Automated Changelog ให้โปรเจกต์ด้วย `semantic-release` เต็มรูปแบบ

ถึงเวลาลงมือทำจริง ในแบบฝึกหัดนี้เราจะตั้งค่าโปรเจกต์ตัวอย่างให้มี **Automated Changelog เต็มรูปแบบ** ตั้งแต่การ commit ไปจนถึง `CHANGELOG.md` ถูก generate และ commit กลับเข้า repository โดยอัตโนมัติผ่าน CI/CD

### เป้าหมายของแบบฝึกหัด

เมื่อทำเสร็จ คุณจะได้โปรเจกต์ที่:

1. ใช้ Conventional Commits ในการเขียน commit message ทุกครั้ง
2. เมื่อ push เข้า branch `main` ระบบ CI จะรัน `semantic-release` อัตโนมัติ
3. `semantic-release` จะวิเคราะห์ commit, ตัดสินใจเวอร์ชันใหม่, generate `CHANGELOG.md`, สร้าง git tag และ GitHub Release ให้อัตโนมัติทั้งหมด โดยไม่ต้องมีคนกดปุ่มใด ๆ เพิ่มเติม

### ขั้นตอนที่ 1: เตรียมโปรเจกต์และติดตั้ง dependency

```bash
mkdir changelog-automation-demo
cd changelog-automation-demo
git init
npm init -y
```

ติดตั้ง `semantic-release` พร้อม plugin ที่จำเป็น:

```bash
npm install --save-dev \
  semantic-release \
  @semantic-release/changelog \
  @semantic-release/git \
  @semantic-release/github \
  @semantic-release/commit-analyzer \
  @semantic-release/release-notes-generator
```

### ขั้นตอนที่ 2: สร้างไฟล์ config `.releaserc.json`

```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    [
      "@semantic-release/changelog",
      {
        "changelogFile": "CHANGELOG.md"
      }
    ],
    [
      "@semantic-release/git",
      {
        "assets": ["CHANGELOG.md", "package.json"],
        "message": "chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}"
      }
    ],
    "@semantic-release/github"
  ]
}
```

### ขั้นตอนที่ 3: ตั้งค่า `commitlint` เพื่อบังคับ Conventional Commits (ต่อยอดจาก Part 88)

```bash
npm install --save-dev @commitlint/cli @commitlint/config-conventional husky
```

สร้างไฟล์ `commitlint.config.js`:

```javascript
module.exports = { extends: ['@commitlint/config-conventional'] };
```

ตั้งค่า Git hook ด้วย husky:

```bash
npx husky init
echo "npx --no-install commitlint --edit \$1" > .husky/commit-msg
```

การตั้งค่านี้ทำให้ทุก commit ที่ commit message ไม่ตรงรูปแบบ Conventional Commits จะถูก **ปฏิเสธทันทีตั้งแต่เครื่อง local** ก่อนที่จะไปถึง CI เลยด้วยซ้ำ

### ขั้นตอนที่ 4: สร้าง GitHub Actions workflow สำหรับรัน release อัตโนมัติ

สร้างไฟล์ `.github/workflows/release.yml`:

```yaml
name: Release

on:
  push:
    branches:
      - main

permissions:
  contents: write
  issues: write
  pull-requests: write
  id-token: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          persist-credentials: false

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: npx semantic-release
```

**จุดสำคัญที่ต้องระวังในไฟล์ workflow นี้:**

- `fetch-depth: 0` จำเป็นมาก เพราะ `semantic-release` ต้องอ่าน commit history ทั้งหมดเพื่อวิเคราะห์ตั้งแต่ release ล่าสุด ถ้า checkout แบบ shallow clone (default) จะไม่มีประวัติพอให้วิเคราะห์
- `permissions: contents: write` จำเป็นเพื่อให้ workflow เขียนกลับเข้า repository ได้ (commit CHANGELOG.md, สร้าง tag)
- `npm test` ควรอยู่ก่อนขั้นตอน release เสมอ เพื่อไม่ให้ release เวอร์ชันที่โค้ดพังออกไป
- ใช้ `secrets.GITHUB_TOKEN` ที่ GitHub Actions มีให้อัตโนมัติอยู่แล้ว ไม่ต้องสร้าง token เพิ่มเติมสำหรับ workflow นี้ (ยกเว้นถ้าต้องการ publish ไปที่ npm ด้วย จะต้องเพิ่ม `NPM_TOKEN` เป็น secret แยกต่างหาก)

### ขั้นตอนที่ 5: สร้างไฟล์ `CHANGELOG.md` เปล่าเริ่มต้น (ไม่บังคับ แต่แนะนำ)

```bash
touch CHANGELOG.md
git add CHANGELOG.md
git commit -m "chore: initial commit"
```

### ขั้นตอนที่ 6: ทดสอบด้วยการสร้าง commit ตาม Conventional Commits แล้ว push

```bash
echo "console.log('hello world');" > index.js
git add index.js
git commit -m "feat: เพิ่มไฟล์ entry point เริ่มต้นของโปรเจกต์"

git branch -M main
git remote add origin https://github.com/<your-username>/changelog-automation-demo.git
git push -u origin main
```

เมื่อ push เสร็จ ให้เข้าไปดูที่แท็บ **Actions** บน GitHub จะเห็น workflow `Release` เริ่มทำงาน หากทุกอย่างตั้งค่าถูกต้อง ผลลัพธ์ที่ควรเกิดขึ้นคือ:

1. มี commit ใหม่ถูกสร้างขึ้นโดยอัตโนมัติจาก `semantic-release` ชื่อ `chore(release): 1.0.0 [skip ci]`
2. ไฟล์ `CHANGELOG.md` มีเนื้อหาถูกเติมเข้าไปโดยอัตโนมัติ
3. `package.json` มี field `version` ถูกอัปเดตเป็น `1.0.0`
4. มี git tag `v1.0.0` ถูกสร้างขึ้น
5. มี GitHub Release ปรากฏในแท็บ Releases พร้อมเนื้อหาเดียวกับที่อยู่ใน `CHANGELOG.md`

### ขั้นตอนที่ 7: ทดสอบ bug fix และดูว่าเวอร์ชัน bump เป็น patch

```bash
echo "console.log('hello world v2');" > index.js
git add index.js
git commit -m "fix: แก้ไขข้อความ output ให้ถูกต้อง"
git push
```

รอบนี้ควรเห็นเวอร์ชันขยับจาก `1.0.0` เป็น `1.0.1` และ `CHANGELOG.md` มีหมวด **Bug Fixes** เพิ่มเข้ามาใหม่ต่อจากหมวดของเวอร์ชันก่อนหน้า

### ขั้นตอนที่ 8: ทดสอบ breaking change และดูว่าเวอร์ชัน bump เป็น major

```bash
git commit --allow-empty -m "feat!: เปลี่ยนรูปแบบ output เป็น JSON

BREAKING CHANGE: output จากเดิมเป็น plain text เปลี่ยนเป็น JSON format ทั้งหมด"
git push
```

รอบนี้เวอร์ชันควรกระโดดจาก `1.0.1` เป็น `2.0.0` ทันที และ `CHANGELOG.md` จะมีหมวด **⚠ BREAKING CHANGES** โผล่ขึ้นมาบนสุดของ entry ใหม่

### ขั้นตอนที่ 9: ตรวจสอบผลลัพธ์สุดท้ายทั้งหมด

เปิดไฟล์ `CHANGELOG.md` ในเครื่อง (หลังจาก `git pull` ดึงการเปลี่ยนแปลงที่ CI commit กลับมา) ควรได้ไฟล์หน้าตาประมาณนี้:

```markdown
# Changelog

All notable changes to this project will be documented in this file. See
[Conventional Commits](https://conventionalcommits.org) for commit guidelines.

## [2.0.0](https://github.com/<user>/changelog-automation-demo/compare/v1.0.1...v2.0.0) (2026-09-26)

### ⚠ BREAKING CHANGES

* เปลี่ยนรูปแบบ output เป็น JSON: output จากเดิมเป็น plain text เปลี่ยนเป็น JSON format ทั้งหมด

### Features

* เปลี่ยนรูปแบบ output เป็น JSON ([abc1234](https://github.com/.../commit/abc1234))

## [1.0.1](https://github.com/<user>/changelog-automation-demo/compare/v1.0.0...v1.0.1) (2026-09-26)

### Bug Fixes

* แก้ไขข้อความ output ให้ถูกต้อง ([def5678](https://github.com/.../commit/def5678))

## [1.0.0](https://github.com/<user>/changelog-automation-demo/releases/tag/v1.0.0) (2026-09-26)

### Features

* เพิ่มไฟล์ entry point เริ่มต้นของโปรเจกต์ ([ghi9012](https://github.com/.../commit/ghi9012))
```

### ขั้นตอนต่อยอด (ไม่บังคับ แต่แนะนำให้ลองทำ)

1. **ทดลองเพิ่ม `@semantic-release/npm`** เพื่อ publish ไปยัง npm registry จริง และดูว่าเวอร์ชันบน npm ตรงกับ `CHANGELOG.md` เสมอ
2. **ทดลองใช้ `.github/release.yml`** (จาก Step 887) ควบคู่กันไป เพื่อเปรียบเทียบผลลัพธ์ระหว่าง auto-generated release notes ของ GitHub กับของ `semantic-release`
3. **ทดลองตั้งค่า pre-release channel** เช่น branch `beta` ที่ release เป็นเวอร์ชัน `2.1.0-beta.1` เพื่อฝึกการทำ pre-release แบบที่ทีมมืออาชีพใช้ทดสอบฟีเจอร์ก่อนปล่อยจริง
4. **ลองปรับ `.releaserc.json` ให้ใช้ preset `conventionalcommits`** แทนค่า default แล้วสังเกตความแตกต่างของรูปแบบ Changelog ที่ได้

### Checklist สำหรับตรวจสอบว่าทำแบบฝึกหัดสำเร็จ

- [ ] ติดตั้งและตั้งค่า `semantic-release` พร้อม plugin ที่จำเป็นครบถ้วน
- [ ] ตั้งค่า `commitlint` + `husky` เพื่อบังคับ Conventional Commits ตั้งแต่เครื่อง local
- [ ] สร้าง GitHub Actions workflow ที่รัน `semantic-release` อัตโนมัติเมื่อ push เข้า `main`
- [ ] ทดสอบ commit แบบ `feat:` แล้วเห็นเวอร์ชัน bump เป็น minor/initial release ถูกต้อง
- [ ] ทดสอบ commit แบบ `fix:` แล้วเห็นเวอร์ชัน bump เป็น patch ถูกต้อง
- [ ] ทดสอบ commit แบบ breaking change (`!` หรือ `BREAKING CHANGE:` footer) แล้วเห็นเวอร์ชัน bump เป็น major ถูกต้อง
- [ ] ตรวจสอบว่า `CHANGELOG.md` ถูกอัปเดตอัตโนมัติและมีเนื้อหาจัดหมวดหมู่ถูกต้องตามที่คาดหวัง
- [ ] ตรวจสอบว่า GitHub Release ถูกสร้างขึ้นอัตโนมัติพร้อมเนื้อหาตรงกับ `CHANGELOG.md`

---

## สรุป Part 89

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Changelog** คือเอกสารสรุปการเปลี่ยนแปลงแต่ละเวอร์ชันที่เขียนขึ้นเพื่อผู้ใช้ ไม่ใช่แค่ dump commit log ดิบ ๆ และมีบทบาทสำคัญทั้งต่อผู้ใช้ปลายทาง ทีมพัฒนา และทีม Support/Marketing
2. **Keep a Changelog** เป็นมาตรฐานสากลที่กำหนดโครงสร้างชัดเจน ทั้งการเรียงเวอร์ชันใหม่ไปเก่า การใช้หมวด `[Unreleased]` และ 6 หมวดหมู่มาตรฐาน (Added, Changed, Deprecated, Removed, Fixed, Security)
3. การเขียน Changelog **ด้วยมือ** ให้คุณภาพภาษาที่ดีกว่าแต่เสี่ยงตกหล่นและเสียเวลา ในขณะที่การเขียนแบบ **อัตโนมัติ** รับประกันความครบถ้วนและสอดคล้องกับ SemVer เสมอ แต่ภาษาที่ได้ดิบกว่า — แนวทาง **Hybrid** จึงเป็นทางที่หลายทีมมืออาชีพเลือกใช้
4. **`semantic-release`** คือเครื่องมือ orchestrator ที่ทำให้ทั้งกระบวนการ release ตั้งแต่ตัดสินใจเวอร์ชัน generate changelog ไปจนถึง publish เป็นแบบอัตโนมัติเต็มรูปแบบ ผ่านสถาปัตยกรรม plugin pipeline
5. **`conventional-changelog`** คือ library/CLI ระดับล่างที่เน้นการ generate เนื้อหา Changelog อย่างเดียว เหมาะกับทีมที่ต้องการควบคุมขั้นตอนอื่นของ release เอง และเป็น engine เบื้องหลังที่ `semantic-release` ใช้อยู่ด้วย
6. **Release Please** ของ Google ใช้แนวคิด "Release PR" ที่ให้คนรีวิวก่อน merge เพื่อ release จริง เหมาะกับทีมที่ต้องการ human gate ก่อนปล่อยเวอร์ชันใหม่ และรองรับ Monorepo ได้ดีในตัว
7. **GitHub Auto-generated Release Notes** เป็นฟีเจอร์ในตัวที่ใช้งานได้ทันที อิงจาก PR และ label แสดง contributor อัตโนมัติ แต่ไม่คำนวณเวอร์ชันให้และไม่เขียนไฟล์ `CHANGELOG.md` ในตัว repository
8. การ**จัดหมวดหมู่** Changelog ให้อ่านง่ายสำคัญมาก ควรเรียงตามลำดับความสำคัญ (Breaking Changes → Security → Features → …) และควร mapping commit type กับหมวดหมู่ให้ชัดเจน
9. **Changesets** เป็นแนวทางที่ออกแบบมาสำหรับ **Monorepo** โดยเฉพาะ ใช้ไฟล์ changeset แยกที่นักพัฒนาเขียนเองแทนการวิเคราะห์จาก commit message ทำให้จัดการ independent versioning และ inter-package dependency ได้ดีกว่า
10. เราได้ลงมือฝึกตั้งค่า **Automated Changelog เต็มรูปแบบด้วย `semantic-release`** ตั้งแต่ commit ผ่าน `commitlint`, ตั้งค่า GitHub Actions workflow, จนถึงการเห็นผลลัพธ์จริงของ `CHANGELOG.md` และ GitHub Release ที่ถูกสร้างขึ้นอัตโนมัติ

Changelog Automation คือหนึ่งในทักษะที่แยกทีมพัฒนามืออาชีพออกจากทีมทั่วไปได้ชัดเจนที่สุด เพราะมันสะท้อนถึงวินัยในการทำงานตั้งแต่ต้นน้ำ (การเขียน commit message) ไปจนถึงปลายน้ำ (การสื่อสารกับผู้ใช้) — ทักษะนี้จะกลายเป็นรากฐานสำคัญเมื่อเราก้าวเข้าสู่การทำงานร่วมกันในระดับองค์กรขนาดใหญ่ใน Part ถัดไป

**ต่อไป:** [Part 90: Multi-team Collaboration ด้วย Git ในองค์กรขนาดใหญ่](./part-090-multi-team-collaboration.md)
