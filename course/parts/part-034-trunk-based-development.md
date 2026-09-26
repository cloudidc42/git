# Part 34: Trunk-Based Development

> **Step ในหลักสูตรนี้:** Step 331–340
> **เฟส:** 4 — การทำงานเป็นทีมด้วย Git/GitHub (Workflow Model มาตรฐาน)
> **เป้าหมายของ Part นี้:** เข้าใจ **Trunk-Based Development (TBD)** อย่างลึกซึ้ง ตั้งแต่หลักการพื้นฐาน (commit เข้า main บ่อยมาก อายุ branch สั้นมาก) เครื่องมือสำคัญอย่าง Feature Flags ที่ทำให้ TBD ใช้งานได้จริงในทีมขนาดใหญ่ระดับ Google และ Meta ไปจนถึงเงื่อนไขจำเป็นอย่าง CI ที่แข็งแรง เปรียบเทียบ TBD กับ Git Flow และ GitHub Flow ที่เรียนมาก่อนหน้านี้ และปิดท้ายด้วยกรอบการตัดสินใจเลือก Workflow ให้เหมาะกับ maturity ของทีมจริง พร้อมแบบฝึกหัดเขียนโค้ดจำลอง Feature Flag ด้วยตัวเอง

---

## สารบัญของ Part นี้

- Step 331: Trunk-Based Development (TBD) คืออะไร
- Step 332: หลักการ — commit เข้า trunk (main) บ่อยมาก อายุ branch สั้นมาก
- Step 333: Feature Flags/Toggles — เครื่องมือสำคัญที่ทำให้ TBD ปลอดภัย
- Step 334: TBD กับทีมขนาดใหญ่ในอุตสาหกรรม (Google, Meta และ Monorepo)
- Step 335: Short-lived Feature Branch ใน TBD ต่างจาก Feature Branch Workflow อย่างไร
- Step 336: CI ที่แข็งแรง — เงื่อนไขจำเป็นของ TBD
- Step 337: ข้อดีและข้อเสียของ Trunk-Based Development
- Step 338: ตารางเปรียบเทียบ Trunk-Based vs Git Flow vs GitHub Flow
- Step 339: การเลือก Workflow ให้เหมาะกับ Maturity ของทีม (Decision Framework)
- Step 340: แบบฝึกหัด — จำลอง TBD ด้วย Feature Flag ในโปรเจกต์ทดลอง

---

## Step 331: Trunk-Based Development (TBD) คืออะไร

ใน Part 31 และ Part 32 คุณได้เรียนรู้ **Centralized Workflow**, **Feature Branch Workflow** และ **Git Flow** ไปแล้ว ส่วนใน Part 33 คุณได้เรียนรู้ **GitHub Flow** ซึ่งเป็นการลดความซับซ้อนของ Git Flow ลงให้เหลือแค่ `main` กับ feature branch สั้น ๆ

ใน Part นี้เราจะมาเรียนรู้ Workflow อีกแบบหนึ่งที่ถูกพูดถึงมากที่สุดในวงการ DevOps และ Continuous Delivery ยุคปัจจุบัน นั่นคือ **Trunk-Based Development (TBD)**

### นิยามของ Trunk-Based Development

> **Trunk-Based Development คือแนวทางการพัฒนาซอฟต์แวร์ที่นักพัฒนาทุกคนใน team commit โค้ดของตัวเองเข้าไปยัง branch หลักเพียงเส้นเดียว (เรียกว่า "trunk" หรือใน Git เรียกว่า `main`/`master`) อย่างสม่ำเสมอและบ่อยครั้ง (อย่างน้อยวันละครั้ง หรือบ่อยกว่านั้น) โดยหลีกเลี่ยงการสร้าง branch ที่มีอายุยืนยาว (long-lived branch)**

คำว่า **"trunk"** มาจากภาพเปรียบเทียบต้นไม้ — trunk คือ **ลำต้นหลัก** ของต้นไม้ ส่วน feature ต่าง ๆ ที่นักพัฒนาทำเปรียบเหมือน **กิ่งก้าน (branch)** เล็ก ๆ ที่แตกออกจากลำต้นแล้ว**รีบกลับมารวมกับลำต้นโดยเร็วที่สุด** ไม่ปล่อยให้กิ่งนั้นเติบโตแยกออกไปไกลจนกลับมารวมยาก

### แนวคิดหลักที่ทำให้ TBD ต่างจาก Workflow อื่น ๆ ที่เคยเรียน

ใน Git Flow คุณมี `develop`, `feature/*`, `release/*`, `hotfix/*` หลาย branch พร้อมกัน และ branch เหล่านี้อาจมีอายุเป็นสัปดาห์หรือเป็นเดือน

ใน GitHub Flow คุณมี `main` กับ feature branch ที่ควรมีอายุสั้น (แนะนำไม่กี่วัน) แต่ในทางปฏิบัติหลายทีมก็ยังปล่อยให้ branch มีอายุเป็นสัปดาห์ได้อยู่ดีถ้าไม่มีวินัยพอ

**Trunk-Based Development เข้มงวดกว่านั้นมาก** — หัวใจของมันคือ:

1. มี branch หลักเพียงเส้นเดียวที่ **"จริง" (source of truth)** คือ `main`
2. นักพัฒนา **commit เข้า `main` โดยตรง หรือผ่าน branch ที่มีอายุสั้นมาก (เป็นชั่วโมง ไม่ใช่เป็นวันหรือสัปดาห์)**
3. `main` ต้อง **build ผ่านและ deploy ได้ตลอดเวลา (always releasable)**
4. โค้ดที่ยังไม่เสร็จสมบูรณ์ **ถูกซ่อนไว้ด้วย Feature Flag** แทนที่จะถูกซ่อนไว้ในกิ่ง branch ที่แยกออกไป

### ภาพเปรียบเทียบระหว่าง Workflow ที่เคยเรียนกับ TBD

```
Git Flow (Part 32):
main     ●─────────────────────●─────────●
develop  ●──●──●──●──●──●──●──●──●──●──●
feature       ╲__╱     ╲______╱
release                        ╲____╱
(หลาย branch มีอายุยืนยาว หลายสัปดาห์)

GitHub Flow (Part 33):
main     ●──●────●───●─────●──●───●──●
feature      ╲__╱    ╲___╱   ╲__╱
(feature branch อายุสั้น เป็นวันถึงไม่กี่วัน)

Trunk-Based Development (Part นี้):
main     ●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●
              ╲╱  ╲╱   ╲╱  ╲╱
(commit ตรงเข้า main แทบตลอดเวลา หรือ branch อายุแค่ไม่กี่ชั่วโมง)
```

จะเห็นว่ายิ่งไปทางขวา ความถี่ในการ integrate โค้ดกลับเข้า main ยิ่งสูงขึ้นเรื่อย ๆ และอายุของ branch ยิ่งสั้นลงเรื่อย ๆ นี่คือแกนกลางของสิ่งที่เรียกว่า **"Continuous Integration" ในความหมายที่แท้จริงตามที่ผู้บุกเบิกอย่าง Martin Fowler เคยนิยามไว้**

### ทำไมต้องเรียนรู้ TBD ทั้งที่เพิ่งเรียน Git Flow กับ GitHub Flow ไป

เพราะ TBD ไม่ใช่แค่ "อีกหนึ่งวิธีการแยก branch" แต่มันคือ **ปรัชญาที่แตกต่างโดยพื้นฐาน** เกี่ยวกับวิธีที่ทีมจัดการความเสี่ยงจากการรวมโค้ด (integration risk)

- Git Flow และ Feature Branch Workflow แก้ปัญหาความเสี่ยงด้วยการ **แยกงานออกจากกันให้นานที่สุดเท่าที่จำเป็น** แล้วค่อยมารวมทีเดียวตอนท้าย (deferred integration)
- Trunk-Based Development แก้ปัญหาความเสี่ยงด้วยการ **รวมงานเข้าด้วยกันให้บ่อยที่สุดเท่าที่เป็นไปได้** เพื่อไม่ให้ความแตกต่างสะสมมากจนรวมยาก (continuous integration)

สองปรัชญานี้ตรงข้ามกันโดยสิ้นเชิง และองค์กรเทคโนโลยีชั้นนำของโลกจำนวนมากในปัจจุบัน (Google, Meta, Netflix, Amazon) เลือกใช้แนวทางที่สองนี้เป็นค่าเริ่มต้น เราจะมาดูเหตุผลกันตลอด Part นี้

---

## Step 332: หลักการ — Commit เข้า Trunk (main) บ่อยมาก อายุ Branch สั้นมาก

มาลงรายละเอียดของหลักการที่เป็นแก่นแท้ของ TBD กันให้ชัดเจน

### หลักการที่ 1: ความถี่ในการ Commit เข้า Trunk

Trunk-Based Development แบ่งได้เป็น 2 รูปแบบตามความถี่และวิธีการ commit:

**แบบที่ 1: Commit ตรงเข้า `main` (Direct Commit to Trunk)**

นักพัฒนาทำงานบนเครื่องตัวเอง เมื่อเขียนโค้ดเสร็จเป็นชิ้นเล็ก ๆ (small, incremental change) ก็ `git commit` และ `git push` เข้า `main` โดยตรงทันที ไม่ต้องผ่าน branch หรือ Pull Request

```bash
git checkout main
git pull origin main
# แก้ไขโค้ดเล็กน้อย
git add .
git commit -m "feat: เพิ่มการตรวจสอบ email format ใน registration form"
git push origin main
```

รูปแบบนี้พบได้ในทีมที่มี**วินัยสูงมาก**และมี **pair programming / mob programming** เป็นวัฒนธรรมหลัก เพราะการรีวิวโค้ดเกิดขึ้น "แบบ real-time" ระหว่างเขียนโค้ดอยู่แล้ว ไม่ต้องพึ่งพา PR review แบบ async

**แบบที่ 2: Commit ผ่าน Short-Lived Feature Branch (ที่นิยมมากกว่าในทางปฏิบัติ)**

นักพัฒนาสร้าง branch สั้น ๆ เพื่อทำงานชิ้นเล็ก ๆ เปิด Pull Request ให้ทีมรีวิวอย่างรวดเร็ว แล้ว merge กลับเข้า `main` **ภายในไม่กี่ชั่วโมงถึงอย่างมาก 1-2 วัน**

```bash
git checkout main
git pull origin main
git checkout -b add-email-validation
# ทำงานเล็ก ๆ ให้เสร็จเร็วที่สุด (ภายในไม่กี่ชั่วโมง)
git add .
git commit -m "feat: เพิ่มการตรวจสอบ email format"
git push origin add-email-validation
# เปิด PR, รอรีวิวเร็ว ๆ, merge เข้า main ทันทีที่ผ่าน
```

ไม่ว่าจะใช้แบบไหน **หัวใจสำคัญคือความถี่และความเร็วในการ integrate กลับเข้า main** ไม่ใช่ตัว mechanism ว่าจะผ่าน PR หรือไม่

### หลักการที่ 2: อายุของ Branch ต้องสั้นมาก

นี่คือตัวเลขที่ผู้เชี่ยวชาญด้าน Continuous Delivery อย่าง Paul Hammant (ผู้ดูแลเว็บไซต์ trunkbaseddevelopment.com ซึ่งเป็นแหล่งอ้างอิงหลักของแนวคิดนี้) และงานวิจัย **DevOps Research and Assessment (DORA)** ที่ใช้เป็นเกณฑ์วัด:

| ระดับ | อายุของ Branch | ระดับ TBD |
|---|---|---|
| ดีที่สุด | ไม่กี่ชั่วโมง (เช่น เปิดเช้า ปิดเย็นวันเดียวกัน) | Trunk-Based Development เต็มรูปแบบ |
| ยอมรับได้ | ไม่เกิน 1 วัน | TBD ที่ยังถือว่าอยู่ในเกณฑ์ |
| เริ่มมีปัญหา | 1-2 วัน | ขอบเขตบนสุดที่ยังพอเรียกว่า TBD ได้ |
| ไม่ใช่ TBD แล้ว | มากกว่า 2-3 วัน | กลายเป็น Feature Branch Workflow ทั่วไป |
| ไม่ใช่ TBD อย่างชัดเจน | เป็นสัปดาห์ถึงเดือน | Git Flow / Long-lived Feature Branch |

งานวิจัยของ DORA (ตีพิมพ์ใน *Accelerate* โดย Nicole Forsgren, Jez Humble และ Gene Kim) พบว่า **ทีมที่มี performance สูงสุด (Elite Performers)** มักมี branch ที่อายุสั้นกว่า 1 วัน และ merge เข้า trunk อย่างน้อยวันละครั้งต่อนักพัฒนาหนึ่งคน ในขณะที่ทีมที่มี performance ต่ำมักมี branch ที่มีอายุเป็นสัปดาห์หรือมากกว่านั้น

### หลักการที่ 3: Trunk ต้อง Build ผ่านและ Deploy ได้เสมอ (Always Green, Always Releasable)

นี่คือกฎเหล็กของ TBD ที่ขาดไม่ได้:

> **`main` ต้องอยู่ในสถานะที่ build ผ่าน ทดสอบผ่าน และพร้อม deploy ขึ้น production ได้ทุกเวลา**

ถ้ามีใคร commit โค้ดที่ทำให้ `main` พัง (CI แดง) นั่นถือเป็น**เหตุการณ์วิกฤต (incident)** ที่ทีมต้องหยุดทุกอย่างแล้วช่วยกันแก้ให้ `main` กลับมาเขียวโดยเร็วที่สุด — แนวคิดนี้เรียกว่า **"Stop the Line"** ยืมมาจากระบบการผลิตแบบ Toyota (Andon Cord) ที่พนักงานทุกคนมีสิทธิ์หยุดสายการผลิตทั้งหมดทันทีที่พบปัญหา

```
     ┌─────────────────────────────────┐
     │   main ต้อง build ผ่านเสมอ      │
     │                                  │
     │   ถ้า CI แดง = ทั้งทีมหยุดงาน   │
     │   ช่วยกันแก้ก่อน ไม่ทำอย่างอื่น │
     └─────────────────────────────────┘
```

### หลักการที่ 4: ขนาดของแต่ละ Commit/Change ต้องเล็ก

เพราะต้อง integrate บ่อยและเร็ว การเปลี่ยนแปลงแต่ละครั้งจึงต้อง **เล็กและ atomic** (ทำสิ่งเดียวให้เสร็จสมบูรณ์) แทนที่จะทำ feature ใหญ่ทั้งก้อนแล้วค่อย commit ทีเดียว

หลักที่นิยมพูดถึงคือ **"Small Batch Size"** — แนวคิดที่ยืมมาจาก Lean Manufacturing เช่นกัน ยิ่ง batch (ก้อนของการเปลี่ยนแปลง) เล็กเท่าไหร่ ความเสี่ยงต่อ commit ก็ยิ่งต่ำเท่านั้น และถ้าเกิดปัญหาก็หาสาเหตุและ revert ได้ง่ายกว่ามาก

---

## Step 333: Feature Flags/Toggles — เครื่องมือสำคัญที่ทำให้ TBD ปลอดภัย

คำถามที่เกิดขึ้นทันทีเมื่อได้ยินหลักการของ TBD คือ:

> "ถ้าฉันกำลังพัฒนาฟีเจอร์ใหญ่ที่ใช้เวลาหลายสัปดาห์ แต่ต้อง commit เข้า `main` ทุกวัน แล้วโค้ดที่ยังทำไม่เสร็จจะไปอยู่ใน production ได้อย่างไรโดยไม่พังระบบที่ผู้ใช้กำลังใช้อยู่จริง?"

คำตอบคือเครื่องมือที่สำคัญที่สุดของ Trunk-Based Development: **Feature Flags** (เรียกอีกชื่อว่า **Feature Toggles**, **Feature Switches** หรือ **Feature Gates**)

### Feature Flag คืออะไร

> **Feature Flag คือกลไกในโค้ดที่ใช้ตัวแปร (มักเป็น boolean หรือ configuration) ในการเปิด/ปิดการทำงานของฟีเจอร์หนึ่ง ๆ แบบ runtime โดยไม่ต้องแก้โค้ดหรือ deploy ใหม่**

พูดง่าย ๆ คือแทนที่จะซ่อนโค้ดที่ยังไม่เสร็จไว้ใน branch ที่แยกออกไปจาก `main` (แบบที่ Git Flow ทำ) TBD เลือกที่จะ **merge โค้ดที่ยังไม่เสร็จเข้า `main` ไปเลย แต่ซ่อนมันไว้ด้วย `if` statement** ที่ควบคุมด้วย flag

### ตัวอย่างแนวคิดพื้นฐาน

```javascript
// แบบไม่มี Feature Flag — merge เข้า main ไม่ได้จนกว่าจะเสร็จสมบูรณ์
function renderCheckoutPage() {
  // ฟีเจอร์ใหม่ "one-click checkout" ยังทำไม่เสร็จ
  // ถ้า merge เข้า main ตอนนี้ ผู้ใช้จริงจะเจอฟีเจอร์ที่พังครึ่ง ๆ กลาง ๆ
  return renderOneClickCheckout(); // อันตราย!
}
```

```javascript
// แบบมี Feature Flag — merge เข้า main ได้ทันที แม้ยังทำไม่เสร็จ
function renderCheckoutPage() {
  if (featureFlags.isEnabled('one-click-checkout')) {
    return renderOneClickCheckout(); // เห็นเฉพาะคนที่เปิด flag
  }
  return renderClassicCheckout(); // ผู้ใช้ทั่วไปยังเห็นของเดิมตามปกติ
}
```

ด้วยวิธีนี้ นักพัฒนาสามารถ:

1. Commit เข้า `main` ได้ทุกวัน แม้ฟีเจอร์ยังไม่เสร็จ 100%
2. โค้ดที่ยังไม่เสร็จ **ไม่กระทบผู้ใช้จริง** เพราะ flag ปิดอยู่
3. เปิด flag ให้เฉพาะทีม QA หรือ internal user ทดสอบก่อนได้
4. ค่อย ๆ เปิด flag ให้ผู้ใช้จริงทีละเปอร์เซ็นต์ (Canary/Gradual rollout)
5. ถ้าพบปัญหาหลัง release ก็แค่ **ปิด flag** ได้ทันทีโดยไม่ต้อง revert code หรือ deploy ใหม่

### ประเภทของ Feature Flag (แบ่งตามวัตถุประสงค์การใช้งาน)

| ประเภท | วัตถุประสงค์ | อายุการใช้งาน |
|---|---|---|
| **Release Toggle** | ซ่อนฟีเจอร์ที่ยังพัฒนาไม่เสร็จจาก production จนกว่าจะพร้อม | ระยะสั้น (ลบทิ้งหลัง release) |
| **Experiment Toggle (A/B Testing)** | ทดสอบว่า design/feature แบบไหนดีกว่ากันกับผู้ใช้กลุ่มต่าง ๆ | ระยะกลาง (จนกว่าการทดลองจะจบ) |
| **Ops Toggle (Operational Toggle)** | เปิด/ปิดฟีเจอร์เพื่อควบคุม load หรือปิดฟีเจอร์ที่มีปัญหาแบบฉุกเฉิน (kill switch) | ระยะยาวหรือถาวร |
| **Permission Toggle** | เปิดฟีเจอร์ให้เฉพาะผู้ใช้บางกลุ่ม เช่น Premium user, Beta tester | ระยะยาวหรือถาวร |

### วิธี Implement Feature Flag แบบง่ายที่สุดไปจนถึงระดับ Enterprise

**ระดับ 1: Config file หรือ Environment Variable ธรรมดา**

```javascript
// config.js
module.exports = {
  featureFlags: {
    oneClickCheckout: process.env.FEATURE_ONE_CLICK_CHECKOUT === 'true'
  }
};
```

เหมาะกับทีมเล็กหรือโปรเจกต์ที่ยังไม่ซับซ้อน ข้อเสียคือการเปลี่ยน flag ต้อง restart service หรือ deploy ใหม่

**ระดับ 2: Feature Flag Service ภายในทีมเอง (Database-backed)**

เก็บสถานะ flag ไว้ในฐานข้อมูล แล้วให้แอปพลิเคชันอ่านค่าจาก database หรือ cache แบบ real-time ทำให้เปิด/ปิด flag ได้ทันทีโดยไม่ต้อง deploy ใหม่

**ระดับ 3: Feature Flag Platform ระดับ Enterprise**

บริษัทขนาดใหญ่มักใช้บริการเฉพาะทาง เช่น **LaunchDarkly**, **Split.io**, **Unleash (Open Source)**, **Flagsmith** หรือระบบภายในที่สร้างเอง (Google และ Meta มีระบบ internal ของตัวเองที่ซับซ้อนมาก) ซึ่งรองรับ:

- เปิด flag ให้ผู้ใช้เฉพาะกลุ่ม (targeting rules) เช่น ตาม country, user segment, device
- Gradual rollout (เปิดทีละ 1% → 5% → 25% → 100%)
- Kill switch แบบ real-time พร้อม dashboard ติดตามผล
- การเชื่อมโยงกับระบบ Analytics เพื่อวัดผลกระทบของฟีเจอร์

### ข้อควรระวังของ Feature Flag

Feature Flag ไม่ใช่ยาวิเศษที่ไม่มีต้นทุน มันมีข้อควรระวังสำคัญที่ต้องรู้:

1. **Flag Debt (หนี้เชิงเทคนิคจาก Flag ที่ค้างอยู่)** — ถ้าไม่ลบ flag ที่ไม่ใช้แล้วออกจากโค้ด จะทำให้โค้ดเต็มไปด้วย `if` statement ที่ซับซ้อนขึ้นเรื่อย ๆ จนอ่านยากและ maintain ยาก ทีมที่ทำ TBD จริงจังจึงต้องมีวินัยในการ **"clean up flag" ทันทีที่ฟีเจอร์ release สมบูรณ์แล้ว**
2. **Testing Complexity เพิ่มขึ้น** — ถ้ามี flag หลายตัวพร้อมกัน จำนวน combination ของสถานะที่ต้อง test จะเพิ่มขึ้นแบบทวีคูณ (2 flag = 4 combination, 3 flag = 8 combination)
3. **ต้องมีระบบติดตาม (Flag Inventory)** — ทีมต้องรู้ว่ามี flag อะไรบ้างในระบบ ใครเป็นเจ้าของ และมีแผนจะลบเมื่อไหร่ ไม่เช่นนั้นจะกลายเป็น "flag ที่ไม่มีใครกล้าลบ" สะสมไปเรื่อย ๆ

---

## Step 334: TBD กับทีมขนาดใหญ่ในอุตสาหกรรม (Google, Meta ใช้ Trunk-Based กับ Monorepo)

Trunk-Based Development ไม่ใช่แนวคิดใหม่หรือทฤษฎีที่ยังไม่ผ่านการพิสูจน์ — มันคือแนวทางที่บริษัทเทคโนโลยีที่ใหญ่ที่สุดในโลกใช้งานจริงในระดับที่ใหญ่โตมาก

### Google และ Monorepo ขนาดมหึมา

Google เป็นตัวอย่างที่มีชื่อเสียงที่สุดของการทำ Trunk-Based Development ในสเกลที่ใหญ่โตอย่างไม่น่าเชื่อ:

- Google เก็บโค้ดเกือบทั้งหมดของบริษัท (ยกเว้นบางส่วน เช่น Android, Chromium ที่แยกต่างหาก) ไว้ใน **Repository เดียวขนาดมหึมา** เรียกว่า **Monorepo** ซึ่งมีขนาดใหญ่กว่า Linux Kernel repository หลายเท่า
- วิศวกรหลายหมื่นคนทั่วโลก commit เข้า trunk เดียวกันนี้ **หลายหมื่นครั้งต่อวัน**
- Google ใช้ระบบ version control ภายในชื่อ **Piper** (ไม่ใช่ Git โดยตรง แม้จะมี Git interface ชื่อ "Google Git" ให้ใช้งานได้) ที่ออกแบบมาเพื่อรองรับการ commit จำนวนมหาศาลเข้า trunk เดียว
- แทบไม่มี long-lived branch เลยในระบบนี้ — แทบทุกอย่างพัฒนาบน trunk โดยตรง และใช้ feature flag (ภายในเรียกระบบนี้ว่า "Configuration" หรือคล้ายกัน) เพื่อควบคุมการเปิดใช้งานฟีเจอร์

สาเหตุที่ Google เลือกทางนี้เพราะ **ที่สเกลขนาดนี้ การ merge branch จำนวนมากที่แยกกันนานเข้าด้วยกันเป็นเรื่องที่แทบเป็นไปไม่ได้ในทางปฏิบัติ** — ยิ่งมีคนคนแก้ไขโค้ดพร้อมกันมากเท่าไหร่ ยิ่งต้อง integrate บ่อยเท่านั้นเพื่อไม่ให้ความขัดแย้งสะสม

### Meta (Facebook) และปรัชญา "Move Fast"

Meta ก็ใช้แนวทางคล้ายกันกับ Monorepo ขนาดใหญ่ของตัวเอง (ชื่อภายในเกี่ยวข้องกับระบบที่เรียกว่า Sapling ซึ่งเป็น VCS ที่ Meta พัฒนาเองในภายหลัง โดยได้แรงบันดาลใจจากทั้ง Git และ Mercurial) หลักการที่ Meta ยึดถือคล้ายกับ Google คือ:

- นักพัฒนา commit เข้า trunk บ่อยมาก มักหลายครั้งต่อวัน
- ใช้ **Feature Flag อย่างหนัก** ในการควบคุมการเปิดตัวฟีเจอร์ — Facebook News Feed หลายฟีเจอร์ถูก "ซ่อน" อยู่ในโค้ดที่ deploy ไปแล้วเป็นเดือนก่อนที่จะเปิดใช้งานจริงให้ผู้ใช้เห็น
- มี Automated Testing และ Automated Canary Deployment ที่แข็งแกร่งมาก เพื่อจับปัญหาก่อนที่จะกระทบผู้ใช้วงกว้าง
- Facebook มีวัฒนธรรมที่มีชื่อเสียงคือ **"Move Fast"** (เดิมคือ "Move Fast and Break Things" ก่อนจะเปลี่ยนเป็น "Move Fast with Stable Infra" ในภายหลัง) ซึ่งอาศัย TBD และ Feature Flag เป็นเสาหลักสำคัญ

### ทำไม Monorepo กับ TBD ถึงมักไปด้วยกัน

Monorepo (repository เดียวที่เก็บโค้ดของหลายโปรเจกต์/ทีม) กับ Trunk-Based Development มีความสัมพันธ์ที่เสริมกันโดยธรรมชาติ:

1. **Monorepo ทำให้เห็นผลกระทบข้าม-ทีมได้ทันที** — ถ้าทีม A แก้ shared library แล้วมันกระทบทีม B, CI ของ Monorepo จะรันเทสของทีม B ด้วยทันทีในเวลาเดียวกัน ทำให้พบปัญหาเร็วที่สุด (ตรงข้ามกับ Multi-repo ที่กว่าจะรู้ว่ากระทบกันก็ตอน integrate จริงซึ่งอาจช้าไปมาก)
2. **การมี branch อายุยาวใน Monorepo อันตรายกว่ามาก** — เพราะ Monorepo มีคนแก้ไขพร้อมกันจำนวนมหาศาล ยิ่งปล่อยให้ branch แยกออกไปนาน ความขัดแย้งที่จะเกิดตอน merge ยิ่งซับซ้อนมากขึ้นแบบทวีคูณ
3. **TBD ทำให้ Monorepo ยังคง Build ได้อยู่เสมอ** — ด้วยกฎ "main ต้องเขียวเสมอ" ทำให้ Monorepo ที่มีคนนับหมื่นแก้ไขพร้อมกันยังสามารถอยู่ในสถานะที่ deploy ได้ตลอดเวลา

### แต่ TBD ไม่ได้ผูกติดกับ Monorepo เสมอไป

ต้องเข้าใจให้ชัดเจนว่า **Trunk-Based Development เป็นแนวคิดที่ใช้ได้ทั้งกับ Monorepo และ Multi-repo** ทีมขนาดเล็กหรือขนาดกลางที่ใช้ repository แยกตามโปรเจกต์ก็สามารถใช้หลักการ TBD ได้เช่นกัน เพียงแค่ยึดหลักการ "commit บ่อย branch อายุสั้น main ต้องเขียวเสมอ" — ไม่จำเป็นต้องมี Monorepo ขนาดเท่า Google ถึงจะใช้ TBD ได้

---

## Step 335: Short-Lived Feature Branch ใน TBD ต่างจาก Feature Branch Workflow อย่างไร

หลายคนสับสนระหว่าง **Short-Lived Feature Branch ใน TBD** กับ **Feature Branch ใน Feature Branch Workflow (ที่เรียนใน Part 31)** เพราะทั้งสองแบบก็ใช้คำว่า "feature branch" เหมือนกัน แต่จริง ๆ แล้วมีความแตกต่างที่สำคัญมาก

### ตารางเปรียบเทียบโดยตรง

| มิติ | Feature Branch Workflow (Part 31) | Short-Lived Branch ใน TBD |
|---|---|---|
| **อายุ branch โดยทั่วไป** | หลายวันถึงหลายสัปดาห์ | ไม่กี่ชั่วโมงถึงอย่างมาก 1-2 วัน |
| **ขนาดของงานต่อ branch** | มักเป็น 1 feature เต็ม ๆ | ชิ้นงานเล็กที่สุดที่แยกทดสอบได้ (thin slice) |
| **การจัดการ feature ที่ยังไม่เสร็จ** | ปล่อยไว้ใน branch จนกว่าจะเสร็จ | Merge เข้า main แล้วซ่อนด้วย Feature Flag |
| **ความถี่ในการ sync กับ main** | Pull/rebase จาก main เป็นครั้งคราว | เกือบจะไม่ต้อง sync เพราะ branch มีอายุสั้นมาก |
| **จำนวน commit สะสมก่อน merge** | อาจมีหลายสิบ commit | มักมีแค่ไม่กี่ commit หรือ 1 commit เดียว |
| **ความเสี่ยงของ Merge Conflict** | สูง (ยิ่งอายุมากยิ่งเสี่ยง) | ต่ำมาก (อายุสั้นมาก โอกาสชนกันน้อย) |
| **การแบ่งงานของนักพัฒนา** | แบ่งตาม feature ทั้งก้อน | แบ่งงานเป็นชิ้นเล็กที่สุดเท่าที่จะทำได้ (task decomposition) |

### กุญแจสำคัญ: การแบ่งงานให้เป็น "Thin Slice"

สิ่งที่ทำให้ Short-Lived Branch ใน TBD เป็นไปได้จริงคือทักษะการแบ่งงานที่เรียกว่า **"Slicing"** หรือ **"Thin Vertical Slice"** — คือการแบ่ง feature ใหญ่ให้เป็นชิ้นเล็กที่สุดที่ยัง**merge เข้า main ได้อย่างปลอดภัย**โดยไม่ทำให้ระบบพัง แม้ feature โดยรวมจะยังไม่เสร็จ

ตัวอย่างการแบ่ง feature "ระบบ Comment ใต้โพสต์" แบบ Feature Branch Workflow ทั่วไป เทียบกับแบบ TBD:

**แบบ Feature Branch Workflow (ทำทั้งก้อนใน branch เดียว ใช้เวลา 2 สัปดาห์):**

```
feature/comment-system (อายุ 2 สัปดาห์)
├── สร้าง Comment model + database schema
├── สร้าง API endpoint สำหรับ create/read/update/delete comment
├── สร้าง UI component สำหรับแสดง comment
├── เพิ่มระบบ reply/nested comment
├── เพิ่มระบบ like บน comment
└── เขียน test ครอบคลุมทั้งหมด แล้วเปิด PR ทีเดียว
```

**แบบ Trunk-Based Development (แบ่งเป็นชิ้นเล็ก ๆ แต่ละชิ้น merge เข้า main ภายในไม่กี่ชั่วโมงถึง 1 วัน):**

```
Day 1 เช้า:  merge → เพิ่ม Comment model + migration (ยังไม่มี UI/API ใช้งาน = ไม่กระทบผู้ใช้)
Day 1 บ่าย:  merge → เพิ่ม API endpoint (ซ่อนหลัง feature flag "comments-api")
Day 2 เช้า:  merge → เพิ่ม UI component พื้นฐาน (ซ่อนหลัง flag "comments-ui")
Day 2 บ่าย:  merge → เพิ่มระบบ reply (ยังซ่อนอยู่หลัง flag เดิม)
Day 3:       merge → เพิ่มระบบ like (ยังซ่อนอยู่)
Day 3 บ่าย:  ทดสอบครบแล้ว → เปิด flag "comments-ui" ให้ผู้ใช้ภายในทดสอบ
Day 4:       เปิด flag ให้ผู้ใช้จริง 5% → 25% → 100% ตามลำดับ
```

จะเห็นว่าแบบที่สอง **โค้ดทุกชิ้นถูก integrate เข้า main อย่างต่อเนื่องตลอดเวลา** แทนที่จะสะสมไว้ 2 สัปดาห์แล้วมารวมทีเดียว ความเสี่ยงจากการรวมโค้ด (integration risk) จึงถูกกระจายออกเป็นชิ้นเล็ก ๆ ที่จัดการง่ายกว่ามาก

### ทำไมการแบ่งงานแบบนี้ถึงยากกว่าที่คิด

การฝึกแบ่งงานให้เป็น thin slice ที่ merge เข้า main ได้อย่างปลอดภัยทุกวันคือ **ทักษะที่ต้องฝึกฝน** ไม่ใช่เรื่องที่ทำได้ทันทีสำหรับทีมที่คุ้นเคยกับการทำ feature ใหญ่ทีเดียว มันต้องอาศัย:

- ความสามารถในการออกแบบสถาปัตยกรรมที่รองรับการเพิ่มทีละส่วน (incremental architecture)
- วินัยในการเขียน database migration ที่ **backward-compatible** เสมอ (เพิ่ม column ได้ แต่อย่าเพิ่งลบ column เก่าจนกว่าโค้ดทุกส่วนจะไม่ใช้แล้ว)
- ความเข้าใจเรื่อง API versioning และ contract testing เพื่อไม่ให้ endpoint ที่ยังทำไม่เสร็จกระทบ client ที่มีอยู่

---

## Step 336: CI ที่แข็งแรง (Automated Test ครอบคลุมสูง) เป็นเงื่อนไขจำเป็นของ TBD

ถ้ามีคำถามเดียวที่สำคัญที่สุดก่อนจะตัดสินใจใช้ Trunk-Based Development คำถามนั้นคือ:

> **"ทีมของเรามี automated test ที่ครอบคลุมและเชื่อถือได้พอหรือยัง?"**

ถ้าคำตอบคือ "ไม่" การใช้ TBD จะยิ่งอันตรายกว่า Workflow อื่น ไม่ใช่ปลอดภัยกว่า

### ทำไม CI ที่แข็งแรงถึงเป็น "เงื่อนไขจำเป็น" ไม่ใช่ "ทางเลือก"

ใน Git Flow หรือ Feature Branch Workflow แบบดั้งเดิม การรีวิวโค้ดโดยมนุษย์ (Code Review) และการทดสอบด้วยมือ (Manual QA) ก่อน merge เข้า `develop` หรือ `main` ยังพอเป็น "ด่านกรอง" ที่รับภาระความถูกต้องของโค้ดได้อยู่บ้าง เพราะ branch มีอายุยาว มีเวลาให้ตรวจสอบละเอียด

แต่ใน Trunk-Based Development ที่โค้ด **ถูก merge เข้า main และพร้อม deploy แทบจะทันที** มนุษย์ไม่มีเวลาพอที่จะตรวจสอบทุกอย่างด้วยมือได้ทัน ดังนั้น**ภาระในการตรวจจับบั๊กเกือบทั้งหมดจึงตกไปอยู่ที่ Automated Test และ CI Pipeline**

```
Feature Branch Workflow:
  Code → Code Review (มนุษย์, ละเอียด) → Manual QA → Merge → Deploy
  (มีเวลาหลายวันให้ตรวจสอบ)

Trunk-Based Development:
  Code → Automated Test (วินาที-นาที) → Merge → Automated Deploy
  (มีเวลาแค่ไม่กี่นาทีก่อนจะกระทบ production)
```

### ระดับของ Test Coverage และ Test Pyramid ที่ TBD ต้องการ

TBD ต้องพึ่งพาแนวคิด **Test Pyramid** ที่แข็งแรงมาก:

```
        ▲
       ╱ ╲          E2E Tests (จำนวนน้อย, ช้า, แต่ครอบคลุม flow สำคัญ)
      ╱───╲
     ╱     ╲        Integration Tests (จำนวนปานกลาง)
    ╱───────╲
   ╱         ╲      Unit Tests (จำนวนมากที่สุด, เร็วที่สุด, รันได้ในไม่กี่วินาที)
  ╱───────────╲
```

- **Unit Test จำนวนมาก** ต้องรันเสร็จภายในไม่กี่นาที (บางทีมตั้งเป้าไว้ไม่เกิน 5-10 นาทีสำหรับทั้ง suite)
- **Integration Test** ตรวจสอบว่าส่วนต่าง ๆ ทำงานร่วมกันถูกต้อง
- **End-to-End Test** จำนวนไม่มาก แต่ครอบคลุม critical path ที่สำคัญที่สุดของระบบ
- **Static Analysis / Linting** ตรวจจับปัญหาเชิงโครงสร้างก่อนรันเทสจริงด้วยซ้ำ

### CI Pipeline ที่รองรับ TBD ต้องมีคุณสมบัติอะไรบ้าง

1. **เร็ว (Fast Feedback Loop)** — ถ้า CI ใช้เวลา 1 ชั่วโมงกว่าจะรู้ผล นักพัฒนาจะ commit เข้า main บ่อย ๆ ไม่ได้จริง เพราะรอผลไม่ทัน เป้าหมายที่ดีคือ CI ควรรันเสร็จภายใน **ไม่กี่นาที**
2. **เชื่อถือได้ (Reliable, ไม่มี Flaky Test)** — ถ้าเทสบางตัว fail แบบสุ่ม (flaky) โดยไม่เกี่ยวกับโค้ดที่แก้จริง ทีมจะเริ่ม "เพิกเฉยต่อ CI สีแดง" ซึ่งทำลายหลักการ "main ต้องเขียวเสมอ" ไปโดยสิ้นเชิง
3. **ครอบคลุมพอ (High Coverage on Critical Path)** — ไม่จำเป็นต้องมี coverage 100% แต่ต้องครอบคลุม business logic หลักและ critical path ที่สำคัญที่สุด
4. **รันอัตโนมัติทุกครั้งที่มีการ push** — ไม่มีขั้นตอนที่ต้องรอมนุษย์กดปุ่มก่อน CI จะเริ่มทำงาน
5. **บล็อกการ merge ถ้า test ไม่ผ่าน (Branch Protection)** — ตั้งค่า required status check ให้ CI ต้องผ่านก่อนจะ merge เข้า `main` ได้เสมอ (เราจะเรียนการตั้งค่านี้แบบละเอียดใน Part ว่าด้วย Branch Protection Rules)

### สิ่งที่เกิดขึ้นถ้าใช้ TBD โดยไม่มี CI ที่แข็งแรงพอ

นี่คือกับดักที่ทีมจำนวนมากตกลงไปเมื่อพยายามลอกเลียนแบบ Google/Meta โดยไม่เข้าใจเงื่อนไขเบื้องหลัง:

- Bug หลุดเข้า production บ่อยขึ้น เพราะไม่มีด่านกรองที่แข็งแรงพอมาจับก่อน
- ทีมเริ่มไม่ไว้ใจ `main` ทำให้นักพัฒนาแต่ละคนเริ่มสร้าง local branch ที่แยกออกไปนานขึ้นเรื่อย ๆ "เผื่อไว้ก่อน" ซึ่งขัดกับหลักการ TBD โดยสิ้นเชิง และสุดท้ายกลายเป็น Workflow ที่แย่ที่สุดของทั้งสองโลก (integrate ช้าแต่ก็ยังไม่มี process รีวิวที่ดี)
- Production กลายเป็นสนามทดลอง โดยที่ทีมไม่มีเครื่องมือ (feature flag, canary release) มาช่วยจำกัดความเสียหาย

ข้อสรุปสำคัญคือ: **Trunk-Based Development ไม่ใช่แค่การเปลี่ยนวิธีใช้ Git แต่คือการเปลี่ยน "วัฒนธรรมและโครงสร้างพื้นฐานวิศวกรรมทั้งหมด" ของทีม** — ถ้าพื้นฐานยังไม่พร้อม การบังคับใช้ TBD จะสร้างความเสียหายมากกว่าประโยชน์

---

## Step 337: ข้อดีและข้อเสียของ Trunk-Based Development

มาสรุปข้อดีและข้อเสียของ TBD อย่างเป็นระบบ เพื่อให้เห็นภาพรวมที่สมดุลก่อนจะตัดสินใจว่าทีมของคุณเหมาะกับแนวทางนี้หรือไม่

### ข้อดี

**1. ลดปัญหา Merge Conflict สะสม (Reduced Integration Debt)**

ยิ่ง branch แยกออกไปนานเท่าไหร่ ความแตกต่างระหว่าง branch กับ `main` ยิ่งสะสมมากขึ้นแบบทวีคูณ ไม่ใช่แบบเชิงเส้น — สอง branch ที่แยกกัน 1 วันมักจะ merge กันได้ง่ายมาก แต่สอง branch ที่แยกกัน 1 เดือนอาจมี conflict ที่ซับซ้อนจนต้องนั่งแก้กันเป็นวัน TBD ตัดปัญหานี้ตั้งแต่ต้นทางด้วยการไม่ปล่อยให้ branch แยกออกไปนานพอที่จะสะสมความแตกต่างขนาดนั้น

**2. Integrate เร็ว ได้ Feedback เร็ว (Fast Feedback Loop)**

เมื่อโค้ดถูก merge เข้า `main` แทบจะทันที ปัญหาที่เกิดจากการที่โค้ดของสองคนขัดแย้งกันในเชิง logic (ไม่ใช่แค่ merge conflict ในระดับ text) จะถูกค้นพบเร็วมาก แทนที่จะค้นพบตอนที่ทั้งสอง feature เสร็จสมบูรณ์แล้วและแก้ยากกว่ามาก

**3. ลดความซับซ้อนของ Branch Topology**

ไม่ต้องจดจำว่า branch ไหนอยู่ระหว่างไหน ไม่ต้องมี `develop`, `release/*` ให้สับสน มี `main` เดียวที่ทุกคนโฟกัส ลดภาระทางความคิด (cognitive load) ในการจัดการ branch จำนวนมาก

**4. Deployment Frequency สูงขึ้นตามธรรมชาติ**

เพราะ `main` ต้องพร้อม deploy ได้เสมอ ทีมที่ทำ TBD มักจะ deploy บ่อยขึ้นตามธรรมชาติ ซึ่งตรงกับตัวชี้วัดที่ DORA พบว่าสัมพันธ์กับ performance สูงของทีมวิศวกรรม (Deployment Frequency เป็นหนึ่งใน 4 Key Metrics ของ DORA)

**5. ลด "Merge Hell" ก่อน Release (เทียบกับ Git Flow)**

Git Flow มักมีช่วงเวลาที่เจ็บปวดตอนต้อง merge `develop` เข้า `release` branch แล้วพบ conflict มหาศาลที่สะสมมาจากหลาย feature branch ที่พัฒนาคู่ขนานกันมานาน TBD ไม่มีปัญหานี้เพราะไม่มีการสะสมความแตกต่างตั้งแต่แรก

**6. บังคับให้ทีมมี Engineering Practice ที่ดีขึ้นโดยอัตโนมัติ**

เพื่อจะทำ TBD ได้จริง ทีมจำเป็นต้องพัฒนา CI/CD, automated testing, feature flag และการออกแบบสถาปัตยกรรมที่ดีขึ้น — ผลพลอยได้คือคุณภาพวิศวกรรมโดยรวมของทีมมักดีขึ้นตามไปด้วย

### ข้อเสีย

**1. ต้องการวินัย (Discipline) สูงมากจากทุกคนในทีม**

ทุกคนต้อง commit บ่อย ทุกคนต้อง review เร็ว ทุกคนต้องรับผิดชอบเมื่อ `main` พัง ถ้ามีแม้แต่คนเดียวในทีมที่ไม่ทำตามวินัยนี้ (เช่น สะสมงานไว้ในเครื่องนานแล้วค่อย push) ระบบทั้งหมดจะเริ่มพังลง

**2. ต้องการ Test Coverage สูงมากอย่างที่กล่าวไปใน Step 336**

นี่คือ**ข้อเสียที่ร้ายแรงที่สุด**สำหรับทีมที่ยังไม่มี automated test ที่ครอบคลุมพอ การพยายามใช้ TBD โดยไม่มีพื้นฐานนี้จะยิ่งเพิ่มความเสี่ยงแทนที่จะลด

**3. ความซับซ้อนจาก Feature Flag ที่เพิ่มขึ้น**

ตามที่กล่าวไปใน Step 333 — flag debt, testing complexity ที่เพิ่มขึ้นแบบทวีคูณ และภาระในการดูแล flag inventory คือต้นทุนจริงที่ต้องจ่าย ไม่ใช่ของฟรี

**4. ต้องมี Code Review ที่รวดเร็วมาก**

ถ้าทีมใช้แบบ short-lived branch + PR การรีวิวต้องเกิดขึ้นเร็วมาก (ภายในไม่กี่ชั่วโมง) ถ้าทีมมี culture ที่รีวิว PR ช้า (เช่น รอข้ามวันกว่าจะมีคนรีวิว) หลักการ TBD จะพังทันที เพราะ branch จะกลายเป็น "ยาว" โดยปริยาย

**5. ไม่เหมาะกับซอฟต์แวร์บางประเภทที่ต้อง Release เป็นรอบใหญ่**

ซอฟต์แวร์บางประเภท เช่น firmware ของอุปกรณ์ทางการแพทย์ หรือระบบที่ต้องผ่านการรับรองมาตรฐานอย่างเข้มงวดก่อน release แต่ละครั้ง (regulatory compliance) อาจไม่เหมาะกับการ deploy บ่อย ๆ แบบ continuous — กรณีนี้ Git Flow ที่มี release branch ชัดเจนอาจเหมาะกว่า

**6. ยากสำหรับทีมกระจาย Timezone ที่ Async สูง**

TBD ต้องการการสื่อสารและรีวิวที่รวดเร็ว ถ้าทีมกระจายอยู่คนละ timezone กันมาก (เช่น ครึ่งทีมอยู่อเมริกา ครึ่งทีมอยู่เอเชีย) การรีวิว PR ให้เสร็จภายในไม่กี่ชั่วโมงอาจทำได้ยากกว่าทีมที่อยู่ timezone เดียวกัน

### สรุปเป็นตาราง

| ข้อดี | ข้อเสีย |
|---|---|
| ลด Merge Conflict สะสม | ต้องการวินัยสูงมากจากทุกคน |
| Integrate เร็ว feedback เร็ว | ต้องมี Test Coverage สูงมาก |
| Branch Topology เรียบง่าย | Feature Flag เพิ่มความซับซ้อน (flag debt) |
| Deployment Frequency สูงขึ้น | ต้องการ Code Review ที่รวดเร็วมาก |
| ไม่มี "Merge Hell" ก่อน release | ไม่เหมาะกับซอฟต์แวร์ที่ต้อง release เป็นรอบใหญ่ตามกฎระเบียบ |
| ผลักดันให้ Engineering Practice ดีขึ้น | ยากสำหรับทีมกระจาย timezone ที่พึ่ง async สูง |

---

## Step 338: ตารางเปรียบเทียบสรุป 3 แบบ — Trunk-Based vs Git Flow vs GitHub Flow

ตอนนี้คุณได้เรียนรู้ Workflow หลักทั้ง 3 แบบที่ใช้กันมากที่สุดในอุตสาหกรรมแล้ว (Git Flow ใน Part 32, GitHub Flow ใน Part 33, Trunk-Based Development ใน Part นี้) มาสรุปเปรียบเทียบทั้งหมดในตารางเดียวเพื่อให้เห็นภาพรวมชัดเจนที่สุด

### ตารางเปรียบเทียบหลัก

| มิติ | Git Flow | GitHub Flow | Trunk-Based Development |
|---|---|---|---|
| **ความซับซ้อนของ Branch Model** | สูงมาก (main, develop, feature/*, release/*, hotfix/*) | ต่ำ (main + feature branch) | ต่ำที่สุด (main แทบจะเดียวเท่านั้น) |
| **จำนวน Branch หลักที่ต้องดูแล** | 2 branch ถาวร (main, develop) + branch ชั่วคราวหลายประเภท | 1 branch ถาวร (main) | 1 branch ถาวร (main/trunk) |
| **อายุของ Feature Branch โดยทั่วไป** | หลายวันถึงหลายสัปดาห์ | ไม่กี่วัน (แนะนำ) แต่ในทางปฏิบัติมักยาวกว่านั้น | ไม่กี่ชั่วโมงถึงอย่างมาก 1-2 วัน (เข้มงวดจริง) |
| **ความถี่ในการ Integrate เข้า Main** | ต่ำ (เฉพาะตอน release) | ปานกลางถึงสูง | สูงมาก (หลายครั้งต่อวันต่อคน) |
| **การจัดการ Release** | มี release branch แยกชัดเจน เตรียมตัวก่อน release ได้ | Deploy จาก main ทันทีหลัง merge (Continuous Deployment) | Deploy จาก main ทันที ควบคุมด้วย Feature Flag แยกจากการ deploy |
| **ต้องพึ่งพา Feature Flag ไหม** | ไม่จำเป็น (ใช้ branch แยกแทน) | ไม่บังคับ แต่แนะนำสำหรับฟีเจอร์ใหญ่ | จำเป็นอย่างยิ่ง (ขาดไม่ได้) |
| **ระดับ CI/CD ที่ต้องการ** | ปานกลาง | สูง | สูงมาก (เงื่อนไขจำเป็น) |
| **เหมาะกับซอฟต์แวร์ประเภทไหน** | ซอฟต์แวร์ที่มีหลายเวอร์ชันใช้งานพร้อมกัน (เช่น software แบบ on-premise, mobile app ที่ผู้ใช้ update ช้า) | Web application / SaaS ที่ deploy สม่ำเสมอ | SaaS / Web-scale service ที่ deploy บ่อยมาก โดยเฉพาะ Monorepo ขนาดใหญ่ |
| **เหมาะกับทีมขนาดไหน** | ทีมขนาดกลางถึงใหญ่ที่ต้อง maintain หลายเวอร์ชัน | ทีมขนาดเล็กถึงกลาง | ทีมทุกขนาด แต่ต้องมี engineering maturity สูง |
| **ตัวอย่างองค์กรที่ใช้จริง** | ซอฟต์แวร์ enterprise, ผลิตภัณฑ์ที่มี LTS version | GitHub เอง, Heroku, ทีม startup ส่วนใหญ่ | Google, Meta, Netflix, Amazon |
| **ความเสี่ยงจาก Merge Conflict** | สูง (สะสมนาน) | ปานกลาง | ต่ำมาก |
| **เหมาะกับทีมที่เพิ่งเริ่มทำ CI/CD ไหม** | เหมาะ (ผ่อนปรนกว่า) | พอเหมาะถ้ามี CI พื้นฐานแล้ว | ไม่เหมาะ (อันตรายถ้า CI ยังไม่แข็งแรง) |

### เปรียบเทียบด้วยภาพ Spectrum ความถี่ในการ Integrate

```
Integrate ไม่บ่อย                                          Integrate บ่อยที่สุด
     │                                                              │
  Git Flow ─────────── GitHub Flow ─────────── Trunk-Based Development
     │                                                              │
Branch อายุยาวที่สุด                                    Branch อายุสั้นที่สุด
(สัปดาห์-เดือน)                                          (ชั่วโมง-1 วัน)
```

### จุดที่ควรจำให้แม่น

Workflow ทั้ง 3 แบบนี้ **ไม่ใช่คู่แข่งที่ต้องเลือกอันใดอันหนึ่งตลอดไป** — หลายทีมในโลกจริงเริ่มต้นด้วย Git Flow เพราะเข้าใจง่ายและปลอดภัยสำหรับทีมที่ยังไม่มี CI/CD ที่ดี แล้วค่อย ๆ พัฒนา engineering practice ของตัวเองขึ้นไปจนสามารถเปลี่ยนมาใช้ GitHub Flow และสุดท้ายอาจไปถึง Trunk-Based Development เมื่อทีมพร้อมจริง ๆ — นี่คือสิ่งที่เราจะพูดถึงในรายละเอียดใน Step ถัดไป

---

## Step 339: การเลือก Workflow ให้เหมาะกับ Maturity ของทีม (Decision Framework)

หลังจากเรียนรู้ Workflow ทั้ง 3 แบบแล้ว คำถามที่สำคัญที่สุดคือ **"แล้วทีมของเราควรใช้อันไหน?"** Step นี้จะให้กรอบการตัดสินใจ (Decision Framework) ที่เป็นระบบ แทนที่จะเลือกตามกระแสหรือตามชื่อเสียงของบริษัทใหญ่

### ปัจจัยที่ 1: CI/CD Maturity ของทีม

ถามตัวเองด้วยคำถามเหล่านี้:

- ทีมมี automated test ที่รันอัตโนมัติทุกครั้งที่ push หรือไม่
- Test suite รันเสร็จเร็วแค่ไหน (นาที หรือ ชั่วโมง)
- Test มีความน่าเชื่อถือแค่ไหน (มี flaky test เยอะไหม)
- ทีมมีระบบ deploy อัตโนมัติ (ไม่ต้องมีคนกด deploy ด้วยมือทีละขั้นตอน) หรือไม่
- ทีมเคยใช้ Feature Flag มาก่อนหรือไม่

| ระดับ CI/CD Maturity | Workflow ที่แนะนำ |
|---|---|
| ยังไม่มี automated test หรือมีน้อยมาก | **Git Flow** หรือ **Feature Branch Workflow** — ให้เวลา Code Review/Manual QA ทำหน้าที่เป็นด่านกรองแทน |
| มี automated test พื้นฐาน + CI รันอัตโนมัติแล้ว | **GitHub Flow** — integrate บ่อยขึ้น พร้อม deploy จาก main ได้ |
| มี automated test ครอบคลุมสูง + Deploy อัตโนมัติเต็มรูปแบบ + เคยใช้ Feature Flag มาก่อน | **Trunk-Based Development** — พร้อมสำหรับการ integrate ที่ถี่ที่สุด |

### ปัจจัยที่ 2: ขนาดทีมและโครงสร้างองค์กร

| ขนาดทีม | ข้อพิจารณา |
|---|---|
| ทีมเล็กมาก (1-3 คน) | Workflow ไหนก็ได้ที่ทีมสบายใจ ความเสี่ยงจาก conflict ต่ำอยู่แล้วเพราะคนน้อย มักเลือก GitHub Flow เพราะเรียบง่าย |
| ทีมขนาดกลาง (4-15 คน) | นี่คือช่วงที่ต้องเริ่มคิดจริงจัง — ถ้า CI/CD ยังไม่แข็งแรง แนะนำ GitHub Flow ก่อน ถ้าพร้อมแล้วค่อยขยับไป TBD |
| ทีมขนาดใหญ่ (หลายสิบถึงหลายร้อยคนในโปรเจกต์เดียวกัน) | ยิ่งคนเยอะ ยิ่งต้องพิจารณา TBD อย่างจริงจัง เพราะ branch อายุยาวจะกลายเป็นปัญหาคอขวดสำคัญเมื่อคนจำนวนมากแก้โค้ดพร้อมกัน |
| องค์กรระดับ Google/Meta (หลายพันถึงหลายหมื่นคน) | แทบจำเป็นต้องใช้ TBD เพราะ Workflow อื่นจะขยายสเกล (scale) ไม่ไหวเลย |

### ปัจจัยที่ 3: ประเภทของซอฟต์แวร์

| ประเภทซอฟต์แวร์ | ข้อพิจารณา |
|---|---|
| **Web Application / SaaS ที่ deploy บ่อย** | เหมาะกับ GitHub Flow หรือ TBD เพราะควบคุมได้ผ่าน server ฝั่งเดียว ผู้ใช้ไม่ต้อง update เอง |
| **Mobile Application** | ต้องผ่านการ review ของ App Store/Play Store ก่อน release แต่ละครั้ง มักผสมผสาน: ใช้ TBD ภายในทีม แต่มี release branch สั้น ๆ ตอนใกล้ส่ง store |
| **Software แบบ On-premise / ที่ต้อง Maintain หลายเวอร์ชัน** | เหมาะกับ Git Flow เพราะต้องมี branch แยกสำหรับแต่ละเวอร์ชันที่ลูกค้ายังใช้อยู่ (LTS) |
| **Firmware / Embedded / Safety-critical System** | มักต้องมี release cycle ที่ผ่านการทดสอบและรับรองอย่างเข้มงวด เหมาะกับ Git Flow มากกว่า เพราะ regulatory compliance ไม่รองรับการ deploy บ่อย ๆ แบบ TBD |
| **Open Source Library/Framework** | มักใช้รูปแบบผสมระหว่าง GitHub Flow กับการมี release branch สำหรับ major version (ดูตัวอย่างจาก Part ที่ผ่านมาเรื่อง Open Source Contribution) |
| **Internal Tool / Microservice ขนาดเล็กที่ทีมเดียวดูแล** | เหมาะกับ TBD ได้ง่ายที่สุด เพราะ scope เล็ก ควบคุมความเสี่ยงได้ง่าย |

### Decision Flowchart แบบสรุป

```
เริ่มต้น
   │
   ▼
มี automated test ครอบคลุมสูง + CI/CD เต็มรูปแบบ + เข้าใจ Feature Flag แล้วหรือไม่?
   │
   ├── ไม่ ──▶ ต้อง maintain หลายเวอร์ชันพร้อมกันหรือมีข้อบังคับด้าน compliance หรือไม่?
   │                │
   │                ├── ใช่ ──▶ ใช้ Git Flow
   │                │
   │                └── ไม่ ──▶ ใช้ GitHub Flow (แล้วค่อยพัฒนา CI/CD ให้แข็งแรงขึ้นไปเรื่อย ๆ)
   │
   └── ใช่ ──▶ ทีมมีวินัยสูงพอที่จะ commit บ่อย + review เร็ว + ดูแล flag debt ไหม?
                    │
                    ├── ใช่ ──▶ ใช้ Trunk-Based Development
                    │
                    └── ไม่ ──▶ ใช้ GitHub Flow ไปก่อน แล้วค่อยปรับวัฒนธรรมทีมให้พร้อม
```

### ข้อคิดสำคัญที่สุดของ Step นี้

> **ไม่มี Workflow ไหนที่ "ดีที่สุด" ในตัวมันเอง มีแต่ Workflow ที่ "เหมาะสมกับ maturity ปัจจุบันของทีม" เท่านั้น**

การพยายามบังคับใช้ Trunk-Based Development ทั้งที่ทีมยังไม่มี automated test ที่ดีพอ อันตรายกว่าการใช้ Git Flow ที่ "ดูล้าสมัย" แต่เหมาะกับสถานการณ์จริงของทีม การเลือก Workflow ที่ดีที่สุดคือการเลือกที่ **ตรงกับความสามารถปัจจุบันของทีม แล้วค่อย ๆ ยกระดับ engineering practice ขึ้นไปเรื่อย ๆ** จนสามารถขยับไปใช้ Workflow ที่ integrate ถี่ขึ้นได้อย่างปลอดภัย

---

## Step 340: แบบฝึกหัด — จำลอง TBD ด้วย Feature Flag ง่าย ๆ ในโปรเจกต์ทดลอง

ถึงเวลาลงมือปฏิบัติจริง มาจำลองการทำ Trunk-Based Development พร้อม Feature Flag แบบง่าย ๆ ในโปรเจกต์ทดลองของคุณ

### เตรียมโปรเจกต์ทดลอง

```bash
mkdir ~/git-course/part-34-trunk-based
cd ~/git-course/part-34-trunk-based
git init
```

สร้างไฟล์ `app.js` เป็นแอปพลิเคชันจำลองร้านค้าออนไลน์ง่าย ๆ:

```bash
touch app.js
```

### ขั้นที่ 1: สร้างฟีเจอร์พื้นฐานที่ยังไม่มี Feature Flag (commit แรกเข้า main)

เปิด `app.js` แล้วใส่โค้ดต่อไปนี้:

```javascript
// app.js
function renderCheckoutPage(cart) {
  console.log('--- หน้าชำระเงิน (แบบเดิม) ---');
  console.log(`ยอดรวม: ${cart.total} บาท`);
  console.log('ช่องทางชำระเงิน: บัตรเครดิต, โอนผ่านธนาคาร');
  return 'classic-checkout';
}

const cart = { total: 1250 };
renderCheckoutPage(cart);
```

Commit เข้า `main` โดยตรง (จำลองการทำงานแบบ TBD ที่ commit บ่อย):

```bash
git add app.js
git commit -m "feat: เพิ่มหน้า checkout แบบพื้นฐาน"
```

### ขั้นที่ 2: เพิ่มระบบ Feature Flag แบบง่ายที่สุด

สร้างไฟล์ `featureFlags.js` เพื่อจำลอง feature flag service ง่าย ๆ:

```javascript
// featureFlags.js
const flags = {
  'one-click-checkout': false,   // ปิดอยู่ ยังไม่พร้อมให้ผู้ใช้จริงเห็น
  'dark-mode': false,
  'new-recommendation-engine': false
};

function isEnabled(flagName) {
  // ในระบบจริง ค่านี้อาจอ่านจาก environment variable, database
  // หรือ feature flag service เช่น LaunchDarkly / Unleash
  return flags[flagName] === true;
}

function enable(flagName) {
  flags[flagName] = true;
}

function disable(flagName) {
  flags[flagName] = false;
}

module.exports = { isEnabled, enable, disable };
```

Commit ชิ้นนี้เข้า `main` ทันที (นี่คือชิ้นงานเล็กชิ้นที่ 2 ของวันเดียวกันตามหลัก TBD):

```bash
git add featureFlags.js
git commit -m "feat: เพิ่มระบบ feature flag พื้นฐาน"
```

### ขั้นที่ 3: พัฒนาฟีเจอร์ใหม่ "One-Click Checkout" ที่ยังไม่เสร็จ แต่ merge เข้า main ได้ทันที

นี่คือหัวใจของแบบฝึกหัดนี้ — เราจะเขียนฟีเจอร์ใหม่ที่ **ยังทำไม่เสร็จสมบูรณ์** แต่ยังคง commit เข้า `main` ได้อย่างปลอดภัยเพราะมันถูกซ่อนไว้ด้วย flag

แก้ไข `app.js` ให้เป็นดังนี้:

```javascript
// app.js
const featureFlags = require('./featureFlags');

function renderClassicCheckout(cart) {
  console.log('--- หน้าชำระเงิน (แบบเดิม) ---');
  console.log(`ยอดรวม: ${cart.total} บาท`);
  console.log('ช่องทางชำระเงิน: บัตรเครดิต, โอนผ่านธนาคาร');
  return 'classic-checkout';
}

function renderOneClickCheckout(cart) {
  // ฟีเจอร์ใหม่ที่ยังพัฒนาไม่เสร็จสมบูรณ์ 100%
  // TODO: ยังไม่รองรับ promotion code
  // TODO: ยังไม่มี fallback เมื่อบัตรที่บันทึกไว้หมดอายุ
  console.log('--- หน้าชำระเงินแบบคลิกเดียว (One-Click Checkout) ---');
  console.log(`ยอดรวม: ${cart.total} บาท`);
  console.log('ใช้บัตรที่บันทึกไว้ล่าสุดโดยอัตโนมัติ ✓');
  return 'one-click-checkout';
}

function renderCheckoutPage(cart) {
  if (featureFlags.isEnabled('one-click-checkout')) {
    return renderOneClickCheckout(cart);
  }
  return renderClassicCheckout(cart);
}

const cart = { total: 1250 };
renderCheckoutPage(cart);

module.exports = { renderCheckoutPage };
```

ทดสอบว่ายังทำงานปกติ (flag ปิดอยู่ ผู้ใช้เห็นแบบเดิม):

```bash
node app.js
```

ผลลัพธ์ที่ควรเห็น:

```
--- หน้าชำระเงิน (แบบเดิม) ---
ยอดรวม: 1250 บาท
ช่องทางชำระเงิน: บัตรเครดิต, โอนผ่านธนาคาร
```

Commit เข้า `main` ได้ทันที แม้ `renderOneClickCheckout` จะยังมี `TODO` ค้างอยู่ เพราะมันถูกซ่อนไว้หลัง flag ที่ปิดอยู่ ไม่กระทบผู้ใช้จริง:

```bash
git add app.js
git commit -m "feat: เพิ่ม one-click checkout (ซ่อนหลัง feature flag, ยังพัฒนาไม่เสร็จ)"
```

### ขั้นที่ 4: จำลองการทดสอบภายใน (Internal Testing) ด้วยการเปิด Flag ชั่วคราว

สร้างไฟล์ทดสอบ `test-internal.js` เพื่อจำลองว่าทีม QA เปิด flag มาทดสอบก่อนใครโดยที่ผู้ใช้จริงยังไม่เห็น:

```javascript
// test-internal.js
const featureFlags = require('./featureFlags');
const { renderCheckoutPage } = require('./app');

console.log('=== ทดสอบภายในทีม: เปิด flag one-click-checkout ===');
featureFlags.enable('one-click-checkout');

const cart = { total: 990 };
const result = renderCheckoutPage(cart);

console.log(`ผลลัพธ์ที่ได้: ${result}`);
console.log(result === 'one-click-checkout' ? 'ผ่าน: แสดงฟีเจอร์ใหม่ตามที่คาดไว้' : 'ไม่ผ่าน');
```

รันดู:

```bash
node test-internal.js
```

Commit ไฟล์ทดสอบนี้เข้า `main` เช่นกัน:

```bash
git add test-internal.js
git commit -m "test: เพิ่ม internal test สำหรับ one-click checkout"
```

### ขั้นที่ 5: จำลองการ Gradual Rollout ด้วย Percentage-based Flag

ในโลกจริง การเปิด flag มักไม่ใช่ all-or-nothing แต่เปิดให้ผู้ใช้ทีละเปอร์เซ็นต์ มาจำลองแนวคิดนี้ด้วยโค้ดง่าย ๆ

แก้ไข `featureFlags.js` เพิ่มฟังก์ชัน rollout แบบสุ่มตาม percentage:

```javascript
// featureFlags.js (เพิ่มเติมจากเดิม)
const rolloutPercentage = {
  'one-click-checkout': 0
};

function isEnabledForUser(flagName, userId) {
  if (flags[flagName] === true) return true;
  const percentage = rolloutPercentage[flagName] || 0;
  if (percentage === 0) return false;
  // ใช้ userId แปลงเป็นตัวเลขคงที่ (deterministic) เพื่อให้ user คนเดิมเห็นผลเหมือนเดิมทุกครั้ง
  const hash = String(userId).split('').reduce((acc, ch) => acc + ch.charCodeAt(0), 0);
  return (hash % 100) < percentage;
}

function setRollout(flagName, percentage) {
  rolloutPercentage[flagName] = percentage;
}

module.exports = { isEnabled, enable, disable, isEnabledForUser, setRollout };
```

สร้างไฟล์ `test-rollout.js` เพื่อดูผลลัพธ์ของการ rollout แบบ 25%:

```javascript
// test-rollout.js
const featureFlags = require('./featureFlags');

featureFlags.setRollout('one-click-checkout', 25);

let enabledCount = 0;
const totalUsers = 1000;

for (let userId = 1; userId <= totalUsers; userId++) {
  if (featureFlags.isEnabledForUser('one-click-checkout', `user-${userId}`)) {
    enabledCount++;
  }
}

console.log(`เปิดใช้งานให้ผู้ใช้ ${enabledCount} จาก ${totalUsers} คน (ประมาณ ${(enabledCount / totalUsers * 100).toFixed(1)}%)`);
```

รันดูผลลัพธ์:

```bash
node test-rollout.js
```

ผลลัพธ์ควรใกล้เคียง 25% (เช่น "เปิดใช้งานให้ผู้ใช้ 240 จาก 1000 คน (ประมาณ 24.0%)") ซึ่งแสดงให้เห็นแนวคิดของ **Gradual Rollout** ที่ทีมจริงใช้ในการค่อย ๆ เปิดฟีเจอร์ให้ผู้ใช้มากขึ้นเรื่อย ๆ พร้อมติดตามผลกระทบก่อนเปิด 100%

Commit เข้า `main`:

```bash
git add featureFlags.js test-rollout.js
git commit -m "feat: เพิ่มระบบ gradual rollout แบบ percentage-based"
```

### ขั้นที่ 6: จำลองสถานการณ์ "พบปัญหา" แล้วใช้ Kill Switch

จำลองสถานการณ์ที่ฟีเจอร์ one-click checkout มีบั๊กร้ายแรงหลัง rollout ไปแล้ว และทีมต้องปิดมันทันทีโดยไม่ต้อง revert code หรือ deploy ใหม่:

สร้างไฟล์ `incident-response.js`:

```javascript
// incident-response.js
const featureFlags = require('./featureFlags');

console.log('!!! พบปัญหา: one-click-checkout ทำให้เกิด double-charge บางกรณี !!!');
console.log('ดำเนินการ kill switch ทันที...');

featureFlags.disable('one-click-checkout');
featureFlags.setRollout('one-click-checkout', 0);

console.log('ปิดฟีเจอร์เรียบร้อยแล้ว โดยไม่ต้อง revert code หรือ deploy ใหม่');
console.log('ทีมสามารถแก้บั๊กใน background แล้วค่อย rollout ใหม่ภายหลังได้');
```

รันดู:

```bash
node incident-response.js
```

Commit:

```bash
git add incident-response.js
git commit -m "docs: จำลองสถานการณ์ incident response ด้วย feature flag kill switch"
```

### ขั้นที่ 7: ตรวจสอบประวัติทั้งหมดที่ commit เข้า main โดยตรง

```bash
git log --oneline
```

คุณควรเห็น commit หลายรายการที่ **ทั้งหมด commit เข้า `main` โดยตรง ไม่มี branch แยกเลย** ซึ่งจำลองพฤติกรรมของ Trunk-Based Development ได้ — โค้ดถูก integrate เข้า trunk อย่างต่อเนื่อง แต่ละ commit มีขนาดเล็ก และฟีเจอร์ที่ยังไม่เสร็จสมบูรณ์ (one-click checkout) ก็อยู่ใน `main` ได้อย่างปลอดภัยเพราะถูกควบคุมด้วย Feature Flag ตลอดเวลา

### คำถามทบทวนหลังทำแบบฝึกหัด

ลองตอบคำถามเหล่านี้เพื่อทบทวนความเข้าใจ:

1. ถ้าไม่มี Feature Flag ในแบบฝึกหัดนี้ ฟังก์ชัน `renderOneClickCheckout` ที่ยังมี `TODO` ค้างอยู่จะ commit เข้า `main` ได้อย่างปลอดภัยหรือไม่ เพราะอะไร
2. ในขั้นที่ 6 ที่จำลอง incident response — ถ้าทีมใช้ Git Flow แทนที่จะเป็น TBD + Feature Flag การแก้ปัญหาแบบเดียวกันนี้จะต้องทำอย่างไร (คำใบ้: ต้อง revert commit หรือ deploy hotfix ใหม่)
3. ระบบ `isEnabledForUser` ที่ทำ gradual rollout ในแบบฝึกหัดนี้ ยังขาดคุณสมบัติอะไรบ้างเมื่อเทียบกับ Feature Flag Platform ระดับ Enterprise อย่าง LaunchDarkly ที่กล่าวถึงใน Step 333 (คำใบ้: targeting rule ตาม segment, analytics integration, audit log)

---

## สรุป Part 34

ใน Part นี้เราได้เรียนรู้ **Trunk-Based Development (TBD)** อย่างละเอียด:

1. **Trunk-Based Development คืออะไร** — แนวทางที่ commit เข้า `main` (trunk) บ่อยมากและหลีกเลี่ยง branch ที่มีอายุยืนยาว (Step 331)
2. **หลักการหลักของ TBD** — commit บ่อย branch อายุสั้นมาก (ไม่กี่ชั่วโมงถึง 1-2 วัน) main ต้อง build ผ่านและ deploy ได้เสมอ และขนาด commit ต้องเล็ก (Step 332)
3. **Feature Flags/Toggles** — เครื่องมือสำคัญที่สุดของ TBD ที่ทำให้ merge โค้ดยังไม่เสร็จเข้า main ได้อย่างปลอดภัย พร้อมประเภทของ flag และข้อควรระวังเรื่อง flag debt (Step 333)
4. **TBD ในทีมขนาดใหญ่จริง** — Google กับ Monorepo ขนาดมหึมา, Meta กับปรัชญา "Move Fast" และความสัมพันธ์ระหว่าง Monorepo กับ TBD (Step 334)
5. **Short-Lived Branch ใน TBD ต่างจาก Feature Branch Workflow** — ต่างกันที่อายุ ขนาดงาน และวิธีจัดการ feature ที่ยังไม่เสร็จ พร้อมแนวคิด Thin Vertical Slice (Step 335)
6. **CI ที่แข็งแรงเป็นเงื่อนไขจำเป็น** — Test Pyramid, ความเร็วและความน่าเชื่อถือของ CI Pipeline ที่ TBD ต้องการ และผลเสียถ้าใช้ TBD โดยไม่มี CI ที่ดีพอ (Step 336)
7. **ข้อดีข้อเสียของ TBD** — ลด merge conflict สะสม integrate เร็ว แต่ต้องการวินัยและ test coverage สูงมาก (Step 337)
8. **ตารางเปรียบเทียบ Git Flow, GitHub Flow, Trunk-Based Development** ในทุกมิติสำคัญ (Step 338)
9. **กรอบการตัดสินใจเลือก Workflow** ตาม CI/CD maturity ขนาดทีม และประเภทซอฟต์แวร์ พร้อม decision flowchart (Step 339)
10. **แบบฝึกหัดลงมือจริง** — จำลอง TBD ด้วย Feature Flag ตั้งแต่พื้นฐาน ไปจนถึง gradual rollout และ kill switch สำหรับ incident response (Step 340)

ประเด็นที่สำคัญที่สุดที่ควรจำจาก Part นี้คือ **Trunk-Based Development ไม่ใช่แค่เทคนิคการใช้ Git แต่คือปรัชญาการจัดการความเสี่ยงในการรวมโค้ดที่ตรงข้ามกับ Git Flow โดยสิ้นเชิง** — Git Flow เลือกแยกงานออกจากกันให้นานที่สุดแล้วค่อยรวมทีเดียว ในขณะที่ TBD เลือกรวมงานเข้าด้วยกันให้บ่อยที่สุดเพื่อไม่ให้ความแตกต่างสะสม และ Feature Flag คือกุญแจสำคัญที่ทำให้แนวทางนี้เป็นไปได้จริงโดยไม่ทำให้ผู้ใช้เจอฟีเจอร์ที่ยังไม่เสร็จ

การเลือกว่าทีมของคุณควรใช้ Workflow แบบไหนไม่มีคำตอบที่ตายตัว — ต้องพิจารณา CI/CD maturity ขนาดทีม และประเภทซอฟต์แวร์ประกอบกัน และที่สำคัญที่สุดคือต้องไม่ฝืนใช้ TBD ทั้งที่พื้นฐานยังไม่พร้อม เพราะจะยิ่งสร้างความเสี่ยงมากกว่าประโยชน์

**ต่อไป:** [Part 35: การตั้งชื่อ Branch และ Commit Message Convention (Conventional Commits)](./part-035-naming-convention-conventional-commits.md)
