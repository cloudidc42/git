# Part 83: การจัดการ Repository ขนาดใหญ่ระดับ Enterprise

> **Step ในหลักสูตรนี้:** Step 821–830
> **เฟส:** 8 — DevOps, Security, Compliance ระดับองค์กร
> **เป้าหมายของ Part นี้:** เข้าใจความท้าทายที่แท้จริงเมื่อ Git และ Git hosting platform ต้องรองรับองค์กรระดับพนักงานหลายพันคนและ repository นับพัน ต่อยอด CODEOWNERS ให้ทำงานได้ที่ scale ระดับหลายร้อยทีม เข้าใจกลไก SSO/SAML/SCIM สำหรับจัดการผู้ใช้อัตโนมัติ ออกแบบ Repository Governance ด้วย Organization-level Ruleset เข้าใจแนวคิด InnerSource เชื่อมโยง Reusable Workflows และ CI/CD Components เข้ากับ Standardized CI/CD Templates ทั่วองค์กร รู้จัก Internal Developer Platform (IDP) ผ่านตัวอย่าง Backstage เกริ่นนำ DORA Metrics และเข้าใจโครงสร้างต้นทุนของ Git hosting ระดับ enterprise พร้อมฝึกออกแบบ Repository Governance Policy ให้องค์กรสมมติขนาด 500 คนได้จริงในหน้างาน

---

## สารบัญของ Part นี้

- Step 821: ความท้าทายของ Repository และองค์กรระดับ Enterprise
- Step 822: Code Ownership ที่ Scale ใหญ่ — จาก CODEOWNERS สู่หลายร้อยทีม
- Step 823: Access Control ระดับองค์กร — SSO, SAML, SCIM
- Step 824: Repository Governance — นโยบายกลางที่บังคับใช้ทุก Repo (Organization-level Ruleset)
- Step 825: Internal Open Source / InnerSource Concept
- Step 826: Standardized CI/CD Templates ทั่วทั้งองค์กร
- Step 827: Developer Portal / Internal Developer Platform (IDP) เบื้องต้น
- Step 828: Metrics ระดับองค์กร — เกริ่น DORA Metrics
- Step 829: Cost Management ของ Git Hosting ระดับ Enterprise
- Step 830: แบบฝึกหัด — ออกแบบ Repository Governance Policy สำหรับองค์กร 500 คน

---

## Step 821: ความท้าทายของ Repository และองค์กรระดับ Enterprise

ตลอดหลักสูตรนี้ เราเรียนรู้ Git และ Git hosting platform ในบริบทของทีมขนาดเล็กถึงขนาดกลาง — ทีมที่มีสมาชิกไม่กี่คนถึงไม่กี่สิบคน ดูแล repository ไม่กี่สิบตัว ใช้ CODEOWNERS, Branch Protection, CI/CD pipeline ที่ตั้งค่าเองในแต่ละ repo ได้โดยไม่ยุ่งยากนัก

แต่เมื่อองค์กรเติบโตขึ้นถึงระดับ **enterprise จริง** — พนักงานฝ่ายวิศวกรรมหลักพันคน กระจายอยู่หลายทวีป หลายไทม์โซน หลายภาษา หลายวัฒนธรรมการทำงาน — ปัญหาที่เจอจะเปลี่ยนลักษณะไปอย่างสิ้นเชิง ไม่ใช่แค่ "ปัญหาเดิมที่ใหญ่ขึ้น" แต่เป็น **ปัญหาคนละชนิด** ที่ต้องใช้แนวคิดและเครื่องมือคนละแบบในการแก้

### 821.1 ตัวเลขที่เปลี่ยนธรรมชาติของปัญหา

ลองนึกภาพองค์กรวิศวกรรมซอฟต์แวร์ระดับ enterprise ทั่วไป เช่น บริษัทเทคโนโลยีขนาดใหญ่ ธนาคารข้ามชาติ หรือบริษัทที่มีทีม engineering เกิน 2,000 คน ตัวเลขที่มักพบเจอ:

| มิติ | ทีมขนาดเล็ก/กลาง | องค์กรระดับ Enterprise |
|---|---|---|
| จำนวนวิศวกร | 5–200 คน | 1,000–50,000+ คน |
| จำนวน Repository | 5–500 ตัว | 5,000–500,000+ ตัว |
| จำนวน Commit สะสม | หลักพัน–หลักแสน | หลายสิบล้านถึงหลายร้อยล้าน commit |
| จำนวนทีม/ฝ่ายที่ดูแลโค้ดแยกกัน | 1–20 ทีม | 100–2,000+ ทีม |
| จำนวน Pull Request ต่อวัน | หลักสิบ | หลักหมื่นถึงหลักแสน |
| จำนวน CI/CD pipeline run ต่อวัน | หลักร้อย | หลักล้าน |
| จำนวนภาษาโปรแกรมที่ใช้งานจริง | 1–5 ภาษา | 20–50+ ภาษา |
| Regulatory framework ที่ต้องปฏิบัติตาม | อาจไม่มี | หลายกรอบพร้อมกัน (SOC 2, ISO 27001, PCI-DSS, GDPR, HIPAA, ฯลฯ) |

เมื่อตัวเลขต่างกันขนาดนี้ **สิ่งที่เคยเป็น "แค่ทำงานหนักขึ้นหน่อย" กลายเป็น "ทำแบบเดิมไม่ได้เลย"** เพราะกลไกที่ต้องพึ่งพามนุษย์ในการตัดสินใจทีละจุด (manual process) จะพังทลายเมื่อจำนวนจุดตัดสินใจมากเกินกว่าที่มนุษย์กลุ่มเล็ก ๆ จะตามทันได้

### 821.2 ความท้าทายหลัก 6 ด้าน

**1. Discoverability (การค้นหาและทำความเข้าใจ)**

เมื่อมี repository 10,000 ตัว วิศวกรคนหนึ่งจะรู้ได้อย่างไรว่า:
- มี library ภายในองค์กรที่ทำสิ่งที่ตัวเองกำลังจะเขียนใหม่อยู่แล้วหรือไม่ (ป้องกันการเขียนโค้ดซ้ำซ้อน — reinventing the wheel)
- repository ไหน "ยังมีคนดูแล" (active) และไหน "ถูกทิ้งร้าง" (abandoned/deprecated)
- ใครคือเจ้าของหรือผู้เชี่ยวชาญของ service ตัวหนึ่ง เมื่อเกิดปัญหาฉุกเฉินตอนตี 3

ในทีมเล็ก คำถามเหล่านี้ตอบได้ด้วยการถามเพื่อนร่วมทีมในห้องเดียวกัน แต่ในองค์กรระดับ enterprise ต้องมีระบบรองรับ เช่น Service Catalog, Developer Portal ซึ่งเราจะพูดถึงใน Step 827

**2. Consistency at scale (ความสม่ำเสมอในระดับใหญ่)**

ถ้าแต่ละทีมตั้งค่า CI/CD pipeline เอง กำหนดมาตรฐาน branch protection เอง เลือก linter/formatter เอง ผลลัพธ์คือความหลากหลายที่ไร้การควบคุม (uncontrolled diversity) ซึ่งนำไปสู่:
- ต้นทุนการ onboard วิศวกรใหม่สูงขึ้นทุกครั้งที่ย้ายทีม เพราะแต่ละทีมทำงานคนละแบบ
- ความเสี่ยงด้านความปลอดภัยที่ไม่สม่ำเสมอ (บาง repo สแกน secret บาง repo ไม่สแกน)
- ต้นทุนการบำรุงรักษาเครื่องมือ (tooling) ที่ต้องรองรับความหลากหลายมากเกินจำเป็น

**3. Access control ที่ซับซ้อนแบบทวีคูณ**

ทีมเล็กอาจให้สมาชิกทุกคน "Write access" กับทุก repo ได้โดยไม่มีปัญหา แต่องค์กรระดับ enterprise ต้องรองรับ:
- พนักงานที่เข้า-ออกทุกวัน (onboarding/offboarding นับร้อยครั้งต่อสัปดาห์)
- Contractor ภายนอกที่ต้องมี access ชั่วคราวและถูกต้องตามสัญญา
- ทีมที่ต้องแยกสิทธิ์กันตามกฎหมาย (เช่น ทีมที่ทำงานกับข้อมูลการเงินต้องแยกจากทีมทั่วไปตาม compliance)
- การ audit ว่า "ใครมีสิทธิ์เข้าถึงอะไรบ้าง ณ เวลาใดเวลาหนึ่ง" เพื่อตอบผู้ตรวจสอบภายนอก (external auditor)

**4. Governance ที่ต้อง centralize แต่ไม่ทำลายความคล่องตัวของทีมย่อย**

นี่คือ tension (แรงตึง) สำคัญที่สุดของ enterprise engineering: องค์กรต้องการ **มาตรฐานกลาง** (เพื่อความปลอดภัย ความสม่ำเสมอ compliance) แต่ในขณะเดียวกันทีมย่อยแต่ละทีมต้องการ **ความเป็นอิสระ** (autonomy) เพื่อให้ทำงานได้เร็วโดยไม่ต้องรอขออนุมัติจากส่วนกลางทุกเรื่อง

องค์กรที่จัดการเรื่องนี้ได้ดี (เช่น Spotify, Netflix, Google) มักใช้โมเดล **"Freedom within a Framework"** — กำหนดกรอบกลางที่ห้ามฝ่าฝืน (guardrails) แต่ปล่อยให้รายละเอียดภายในกรอบเป็นสิทธิ์ของทีมย่อยตัดสินใจเอง

**5. Cost ที่ไม่เป็นเชิงเส้น (non-linear)**

ต้นทุนของ Git hosting ไม่ได้เพิ่มแบบเชิงเส้นตามจำนวนพนักงาน เพราะมีปัจจัยอื่นร่วมด้วย เช่น ปริมาณ CI/CD compute minutes ที่ใช้, ปริมาณ storage ของ artifact/LFS, จำนวน seat ของเครื่องมือเสริมต่าง ๆ เราจะเจาะลึกเรื่องนี้ใน Step 829

**6. การรักษาความเร็วในการพัฒนา (Developer velocity) ไม่ให้ตกลง**

งานวิจัยและข้อมูลจากหลายองค์กรระดับโลกชี้ตรงกันว่า เมื่อองค์กรโตขึ้น ความเร็วในการ deploy โค้ดต่อวิศวกรคนหนึ่งมีแนวโน้ม **ลดลง** ถ้าไม่มีการลงทุนด้าน platform engineering อย่างจริงจัง เพราะ overhead ของการประสานงาน (coordination overhead) เพิ่มขึ้นเร็วกว่าจำนวนคนที่เพิ่มเข้ามา — นี่คือเหตุผลที่บริษัทระดับ enterprise ลงทุนหนักกับทีม **Developer Experience (DX)** หรือ **Platform Engineering** โดยเฉพาะ

### 821.3 กฎของ Conway (Conway's Law) กับโครงสร้าง Repository

หลักการสำคัญที่วิศวกรระดับ enterprise ต้องเข้าใจคือ **Conway's Law**:

> "องค์กรที่ออกแบบระบบ จะสร้างระบบที่มีโครงสร้างสื่อสารเหมือนกับโครงสร้างองค์กรของตัวเอง"
> — Melvin Conway, 1967

พูดง่าย ๆ คือ ถ้าทีมของคุณแบ่งเป็น silo (แยกขาดจากกัน ไม่คุยกัน) โครงสร้างโค้ดและ repository ก็มักจะกลายเป็น silo ตามไปด้วย และในทางกลับกัน หากต้องการให้สถาปัตยกรรมซอฟต์แวร์เป็นแบบใด มักต้องออกแบบโครงสร้างทีมให้สอดคล้องกันด้วย (บางครั้งเรียกว่า "Inverse Conway Maneuver")

นี่คือเหตุผลที่การออกแบบ Repository Governance ระดับ enterprise ไม่ใช่แค่เรื่องเทคนิค แต่เป็นเรื่องขององค์กรและการสื่อสารด้วย — สิ่งที่เราจะออกแบบใน Step ถัดไปทั้งหมดต้องคำนึงถึงมิตินี้เสมอ

---

## Step 822: Code Ownership ที่ Scale ใหญ่ — จาก CODEOWNERS สู่หลายร้อยทีม

ใน **Part 36** เราเรียนรู้ไฟล์ `CODEOWNERS` สำหรับกำหนดผู้รับผิดชอบโค้ดในระดับ repository เดียว หรือ monorepo ที่มีไม่กี่ทีม ใน Step นี้เราจะขยายแนวคิดนั้นให้ทำงานได้จริงเมื่อองค์กรมี **หลายร้อยทีม** และ repository นับพันตัว

### 822.1 ทบทวนสั้น ๆ: CODEOWNERS ทำอะไรได้บ้าง

```
# ตัวอย่างจาก Part 36
/frontend/          @team-frontend
/backend/api/       @team-backend-api
/infra/             @team-platform
*.sql                @team-data-platform
```

CODEOWNERS ตอบคำถาม "ใครควร review โค้ดส่วนนี้" ได้ดีในระดับ repository เดียว แต่เมื่อ scale ใหญ่ขึ้น จะเจอข้อจำกัดใหม่ ๆ ที่ CODEOWNERS เพียว ๆ ตอบไม่ได้:

1. **CODEOWNERS ไม่รู้ว่า "ทีมนี้" คือใครในระดับองค์กร** — มันแค่อ้างอิง GitHub team/GitLab group ที่ต้องมีคนไปดูแล membership เอง
2. **ไม่มีกลไกบังคับว่าทุก repo ต้องมี CODEOWNERS** — ถ้าทีมลืมใส่ไฟล์นี้ ก็ไม่มีใครรู้จนกว่าจะเกิดปัญหา
3. **ไม่รู้ว่า "เจ้าของ" คนนั้นยังทำงานอยู่ในบริษัทหรือย้ายทีมไปแล้ว** — CODEOWNERS ไม่ sync กับระบบ HR/identity โดยอัตโนมัติ
4. **ไม่มีแนวคิดเรื่อง "ownership ที่ระดับ service/domain"** ที่ครอบคลุมหลาย repository พร้อมกัน

### 822.2 แนวคิด Ownership ระดับองค์กร: จาก File-level สู่ Service-level

องค์กรระดับ enterprise ที่จัดการเรื่องนี้ได้ดีมักมีแนวคิด **3 ชั้น** ของ ownership:

**ชั้นที่ 1: File-level ownership (สิ่งที่ CODEOWNERS ทำ)**
- ระบุว่าไฟล์/โฟลเดอร์ไหนในหนึ่ง repo ใครต้อง review
- ใช้บังคับผ่าน Branch Protection / Ruleset ที่ require CODEOWNERS review

**ชั้นที่ 2: Repository-level ownership**
- ทุก repository ต้องประกาศ "เจ้าของ" ที่ชัดเจน มักเก็บไว้ใน metadata file มาตรฐาน เช่น `catalog-info.yaml` (รูปแบบที่ Backstage ใช้ — จะพูดถึงใน Step 827) หรือ `OWNERS.yaml`
- ตัวอย่าง metadata:

```yaml
# catalog-info.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: payment-gateway-service
  description: บริการประมวลผลการชำระเงินหลัก
  annotations:
    github.com/team-slug: payments-platform
spec:
  type: service
  lifecycle: production
  owner: team-payments-platform
  system: payments
```

- ข้อมูลนี้ทำให้ระบบส่วนกลาง (เช่น Developer Portal) สามารถตอบคำถาม "repo นี้ใครดูแล" ได้แบบอัตโนมัติ ไม่ต้องเปิดไฟล์ CODEOWNERS ทีละ repo

**ชั้นที่ 3: Domain/Team-level ownership**
- จับกลุ่ม repository ที่เกี่ยวข้องกันเป็น "domain" หรือ "system" เช่น domain "Payments" อาจประกอบด้วย 40 repository ที่ดูแลโดย 3 ทีมย่อย
- ใช้สำหรับ escalation ในกรณีฉุกเฉิน (incident response) — เมื่อระบบ monitoring แจ้งเตือนปัญหาที่ domain "Payments" ทีม on-call ที่ถูกต้องจะถูกแจ้งเตือนทันทีโดยไม่ต้องเดา

### 822.3 ปัญหาที่พบบ่อยเมื่อ Ownership Scale ใหญ่ และวิธีแก้

**ปัญหา 1: Orphaned repository (repo ไร้เจ้าของ)**

เมื่อทีมถูกยุบหรือคนย้ายงาน repository จำนวนมากจะกลายเป็น "ไม่มีใครดูแล" องค์กรระดับ enterprise จึงต้องมีกระบวนการอัตโนมัติตรวจจับ:

```yaml
# ตัวอย่างแนวคิด policy-as-code สำหรับตรวจสอบ orphaned repo
# รันเป็น scheduled job ทุกสัปดาห์
rule: no-orphaned-repository
check:
  - CODEOWNERS ต้องมีอย่างน้อย 1 team ที่ยัง active ใน identity provider
  - team นั้นต้องมีสมาชิกอย่างน้อย 2 คน (ไม่ใช่แค่ bot account)
action-if-failed:
  - เปิด issue อัตโนมัติใน repo นั้น พร้อม tag @platform-team
  - แจ้งเตือนใน dashboard ส่วนกลางว่า repo นี้เป็น "at risk"
```

**ปัญหา 2: CODEOWNERS ที่ระบุทีมผิด หรือทีมที่ไม่มีสิทธิ์เข้าถึง repo นั้นจริง**

GitHub และ GitLab มีการตรวจสอบพื้นฐาน (เช่น GitHub จะเตือนถ้า owner ใน CODEOWNERS ไม่มีสิทธิ์ write ต่อ repo) แต่ในระดับองค์กรควรมี **CI check เฉพาะ** ที่ validate ไฟล์ CODEOWNERS ทุกครั้งที่มีการแก้ไข เพื่อป้องกันการพิมพ์ชื่อทีมผิด (typo) ซึ่งพบได้บ่อยมากในทางปฏิบัติ

**ปัญหา 3: Bottleneck จากทีมที่ต้อง approve มากเกินไป**

เมื่อทีม platform หรือทีม security ถูกกำหนดให้เป็น CODEOWNERS ของโค้ดจำนวนมากทั่วองค์กร (เช่น ทุกไฟล์ที่เกี่ยวกับ infrastructure-as-code) ทีมนั้นจะกลายเป็นคอขวด (bottleneck) วิธีแก้ที่นิยมใช้:

- ใช้ **CODEOWNERS แบบ tiered** — ให้ทีมเจ้าของ domain เป็น primary reviewer และทีม platform เป็น required reviewer เฉพาะไฟล์ที่มีความเสี่ยงสูงจริง ๆ (เช่น IAM policy, secret configuration) ไม่ใช่ทุกไฟล์
- จัดสรร "on-call reviewer rotation" ให้ทีมกลางเพื่อกระจายภาระ ไม่ให้ตกอยู่ที่คนใดคนหนึ่ง
- ตั้งเป้าหมาย SLA การ review ที่ชัดเจน (เช่น ต้อง review ภายใน 4 ชั่วโมงทำการ) และ track เป็น metric

### 822.4 ตัวอย่างโครงสร้าง Ownership ระดับองค์กร (Reference Model)

```
องค์กร (Organization)
 └── Domain: Payments (เจ้าของระดับ Domain: VP Engineering - Payments)
      ├── Team: payments-core
      │    ├── repo: payment-gateway-service   (CODEOWNERS: @payments-core)
      │    ├── repo: payment-reconciliation      (CODEOWNERS: @payments-core)
      │    └── repo: payment-shared-libs         (CODEOWNERS: @payments-core, @payments-consumers)
      ├── Team: fraud-detection
      │    ├── repo: fraud-scoring-engine        (CODEOWNERS: @fraud-detection)
      │    └── repo: fraud-rules-config          (CODEOWNERS: @fraud-detection, @compliance-team)
      └── Team: payments-platform (ทีมกลางของ domain)
           └── repo: payments-terraform-modules  (CODEOWNERS: @payments-platform, @security-team)
```

โครงสร้างนี้ทำให้เมื่อเกิดคำถาม "ใครรับผิดชอบ" ในระดับใดก็ตาม (ไฟล์, repository, domain) มีคำตอบที่ตรวจสอบได้เสมอ (traceable) ไม่ต้องอาศัยความจำของใครคนใดคนหนึ่ง — นี่คือหัวใจของ ownership ที่ scale ได้จริง

---

## Step 823: Access Control ระดับองค์กร — SSO, SAML, SCIM

ในทีมขนาดเล็ก การเพิ่มสมาชิกเข้า organization บน GitHub/GitLab ทำได้ง่ายมาก แค่ invite ทีละคนผ่านอีเมล แต่เมื่อองค์กรมีพนักงานหลักพันคน ที่มีการเข้า-ออกทุกวัน วิธีแบบ manual นี้ **ใช้งานไม่ได้เลย** ทั้งในแง่ความเร็วและความปลอดภัย

### 823.1 ปัญหาของการจัดการ Access แบบ Manual ที่ Scale ใหญ่

1. **Onboarding ช้า** — พนักงานใหม่ต้องรอผู้ดูแลระบบ (admin) มา invite เข้า repository ทีละตัวที่เกี่ยวข้อง อาจใช้เวลาหลายวันกว่าจะเริ่มงานได้จริง
2. **Offboarding ไม่สมบูรณ์ (ความเสี่ยงด้านความปลอดภัยที่ใหญ่ที่สุด)** — เมื่อพนักงานลาออก ถ้า admin ลืมถอด access ออกจากบาง repository หรือบาง tool พนักงานคนนั้นอาจยังเข้าถึงโค้ดของบริษัทได้ต่อไปแม้จะออกจากงานแล้ว นี่คือช่องโหว่ด้านความปลอดภัยที่พบบ่อยที่สุดในองค์กรขนาดใหญ่
3. **ไม่มี Single Source of Truth** — ถ้า "สถานะการเป็นพนักงาน" อยู่ในระบบ HR แต่ "สิทธิ์เข้าถึง repository" อยู่ใน GitHub แยกกันโดยไม่ sync กัน ข้อมูลทั้งสองฝั่งจะไม่ตรงกันเสมอเมื่อเวลาผ่านไป
4. **Audit ยาก** — เมื่อผู้ตรวจสอบ (auditor) ถามว่า "ใครมีสิทธิ์เข้าถึง repository ที่เก็บข้อมูลลูกค้าบ้าง" การตอบคำถามนี้จากระบบที่จัดการ manual แทบเป็นไปไม่ได้ให้แม่นยำ

ทางแก้คือการเชื่อมต่อ Git hosting platform เข้ากับ **Identity Provider (IdP)** ขององค์กรผ่านมาตรฐาน 3 ตัวหลัก: **SSO, SAML, และ SCIM**

### 823.2 SSO (Single Sign-On) คืออะไร

**SSO** คือกลไกที่ให้ผู้ใช้ **login เพียงครั้งเดียว** ผ่านระบบยืนยันตัวตนกลางขององค์กร (เช่น Okta, Azure AD/Entra ID, Google Workspace, Ping Identity) แล้วสามารถเข้าใช้งานระบบต่าง ๆ ในองค์กรได้ทั้งหมดโดยไม่ต้อง login ซ้ำในแต่ละระบบ

**ประโยชน์หลัก:**
- ผู้ใช้จำรหัสผ่านเดียว ไม่ต้องมีรหัสผ่านแยกสำหรับ GitHub, GitLab, Jira, Confluence ฯลฯ
- องค์กรบังคับใช้นโยบายความปลอดภัย (password policy, MFA) จากจุดเดียว แทนที่จะต้องตั้งค่าทีละระบบ
- เมื่อพนักงานถูกปิด account ที่ IdP (เช่น ตอนลาออก) จะไม่สามารถ login เข้า GitHub/GitLab ได้อีกทันที แม้ account บน platform นั้นจะยังไม่ถูกลบ

### 823.3 SAML (Security Assertion Markup Language) — กลไกเบื้องหลัง SSO

**SAML** คือ**โปรโตคอลมาตรฐาน**ที่ใช้แลกเปลี่ยนข้อมูลการยืนยันตัวตนระหว่าง Identity Provider (IdP) กับ Service Provider (SP อย่าง GitHub/GitLab) โดยใช้ XML-based assertion

**ขั้นตอนการทำงานโดยสรุป (SAML flow):**

```
1. ผู้ใช้เข้า https://github.com/orgs/your-company
2. GitHub redirect ผู้ใช้ไปที่ Identity Provider (เช่น Okta)
3. ผู้ใช้ login ที่ Okta (อาจต้องผ่าน MFA ด้วย)
4. Okta สร้าง "SAML Assertion" (เอกสารยืนยันตัวตนที่เซ็นด้วย digital signature)
   ส่งกลับไปที่ GitHub ผ่าน browser ของผู้ใช้
5. GitHub ตรวจสอบลายเซ็นของ Assertion ว่าถูกต้องและมาจาก Okta จริง
6. ถ้าถูกต้อง GitHub อนุญาตให้ผู้ใช้เข้าใช้งาน organization ได้
```

**สิ่งที่ต้องรู้เกี่ยวกับ SAML ในบริบท GitHub Enterprise / GitLab:**

- GitHub Enterprise Cloud รองรับ **SAML SSO แบบบังคับ (enforced)** ระดับ organization ได้ — หมายความว่าทุกคนต้อง login ผ่าน IdP เท่านั้นถึงจะเข้าถึง organization ได้ แม้จะมี personal access token ก็ตาม token นั้นก็ต้องผ่านการ "authorize" กับ SSO ก่อนใช้งานกับ organization นั้น
- GitLab รองรับทั้ง SAML สำหรับ **Self-managed** (ติดตั้งเอง) และ **GitLab.com SaaS** ระดับ group
- SAML แก้ปัญหาเรื่อง "การยืนยันตัวตน" (Authentication) เท่านั้น — มันไม่ได้จัดการเรื่อง "ใครควรมีสิทธิ์อะไรใน repository ไหน" (Authorization) นั่นคือหน้าที่ของ SCIM และการจัดการ team/group membership

### 823.4 SCIM (System for Cross-domain Identity Management) — จัดการ User Lifecycle อัตโนมัติ

ถ้า SAML แก้ปัญหา "login เข้าได้ไหม" **SCIM** แก้ปัญหา "account นี้ควรมีอยู่ในระบบไหม และควรอยู่ใน group ไหนบ้าง" แบบอัตโนมัติ

**SCIM คือมาตรฐานสำหรับ provisioning และ deprovisioning ผู้ใช้แบบอัตโนมัติ** ระหว่าง Identity Provider กับ Service Provider โดยไม่ต้องมีมนุษย์กดปุ่ม invite/remove เอง

**ตัวอย่างการทำงานของ SCIM แบบ end-to-end:**

```
เหตุการณ์ใน HR System: พนักงานคนใหม่เริ่มงานวันนี้ แผนก Backend Engineering
         │
         ▼
Identity Provider (Okta) sync ข้อมูลจาก HR System
         │
         ▼
Okta ส่ง SCIM request ไปที่ GitHub Enterprise:
  POST /scim/v2/Users
  {
    "userName": "somchai.p@company.com",
    "name": { "givenName": "สมชาย", "familyName": "พงศ์วิทยา" },
    "active": true
  }
         │
         ▼
GitHub สร้าง account/invite อัตโนมัติ
         │
         ▼
Okta ส่ง SCIM request เพิ่มผู้ใช้เข้า Group ที่ map กับ GitHub Team:
  PATCH /scim/v2/Groups/{team-backend-id}
  { "operations": [{ "op": "add", "value": { "members": [...] } }] }
         │
         ▼
พนักงานใหม่มีสิทธิ์เข้าถึง repository ของทีม Backend ทันที
โดยไม่มี admin คนใดต้องกดปุ่มเลยแม้แต่ครั้งเดียว
```

**เมื่อพนักงานลาออก (deprovisioning):**

```
HR System ปรับสถานะพนักงานเป็น "Terminated" วันสุดท้ายของการทำงาน
         │
         ▼
Okta ส่ง SCIM request:
  PATCH /scim/v2/Users/{user-id}
  { "active": false }
         │
         ▼
GitHub/GitLab ระงับ account ทันที (suspend)
  - ไม่สามารถ login ได้อีก
  - session ที่ค้างอยู่ถูก revoke
  - personal access token ทั้งหมดถูกยกเลิกโดยอัตโนมัติ
  - ถูกถอดออกจากทุก team/group ที่เคยเป็นสมาชิก
```

**ทำไม SCIM ถึงสำคัญมากในระดับ enterprise:**

1. **ลดความเสี่ยงด้าน security ที่ใหญ่ที่สุดขององค์กรใหญ่** คือ "ex-employee ยังมี access อยู่" — SCIM ทำให้การ offboarding เกิดขึ้นทันทีและครบถ้วนทุกระบบพร้อมกัน (ไม่ใช่แค่ Git hosting แต่รวมถึง Slack, Jira, cloud provider ฯลฯ ที่รองรับ SCIM เดียวกัน)
2. **ลดภาระงานของทีม IT/Platform มหาศาล** — ไม่ต้องมีทีมคอยกด invite/remove สมาชิกหลักร้อยคนต่อสัปดาห์ด้วยมือ
3. **สร้าง audit trail ที่สมบูรณ์** — ทุกการเปลี่ยนแปลงสิทธิ์มีบันทึกอัตโนมัติจากระบบกลาง ไม่ต้องพึ่งพาความจำของ admin
4. **รองรับ "Joiner-Mover-Leaver" (JML) process** — เมื่อพนักงานย้ายทีม (mover) SCIM จะปรับ group membership ให้ตรงกับทีมใหม่โดยอัตโนมัติ ถอด access ของทีมเก่าออก และให้ access ของทีมใหม่เข้ามาแทนที่ ซึ่งเป็นจุดที่ระบบ manual มักพลาดบ่อยที่สุด (ลืมถอด access ทีมเก่า)

### 823.5 สรุปความสัมพันธ์ของ SSO, SAML, SCIM

| แนวคิด | ตอบคำถาม | เปรียบเสมือน |
|---|---|---|
| **SSO** | "ผู้ใช้ login ได้อย่างไรโดยไม่ต้องจำหลายรหัสผ่าน" | บัตรพนักงานใบเดียวที่แตะเข้าได้ทุกประตูในอาคาร |
| **SAML** | "ระบบยืนยันตัวตนคุยกันด้วยภาษาอะไร" (โปรโตคอลเบื้องหลัง SSO) | มาตรฐานการเซ็นชื่อรับรองที่ทุกประตูอ่านเข้าใจตรงกัน |
| **SCIM** | "account และสิทธิ์ควรถูกสร้าง/แก้ไข/ลบเมื่อไหร่โดยอัตโนมัติ" | ระบบที่ทำบัตรพนักงานใหม่ให้อัตโนมัติวันแรกที่เข้างาน และเก็บบัตรคืนทันทีวันที่ลาออก |

ทั้งสามอย่างทำงานร่วมกัน: **SAML/SSO จัดการ "การยืนยันตัวตนตอน login"** ส่วน **SCIM จัดการ "วงจรชีวิตของ account และสิทธิ์" อยู่เบื้องหลังตลอดเวลา** — องค์กรระดับ enterprise ที่จริงจังเรื่องความปลอดภัยจะต้องมีทั้งสองส่วนทำงานร่วมกันเสมอ ไม่ใช่แค่มี SSO อย่างเดียว

---

## Step 824: Repository Governance — นโยบายกลางที่บังคับใช้ทุก Repo

เมื่อมี repository นับพันตัว การพึ่งพาให้แต่ละทีมตั้งค่า Branch Protection, security scanning, และมาตรฐานอื่น ๆ **ด้วยตัวเองในแต่ละ repo** เป็นเรื่องที่ไม่สามารถควบคุมได้จริง เพราะไม่มีทางรู้ว่ามี repo ไหนที่ตั้งค่าไม่ครบ ลืมเปิด branch protection หรือปิด security scan ไปโดยไม่ได้ตั้งใจ

นี่คือปัญหาที่ **Organization-level Ruleset** (คำที่ GitHub ใช้) หรือ **Group-level Push Rules / Compliance Framework** (คำที่ GitLab ใช้) ถูกออกแบบมาแก้โดยเฉพาะ

### 824.1 ปัญหาของ Branch Protection แบบเดิม (Repo-level)

ใน Part ก่อน ๆ เราเรียนรู้การตั้งค่า Branch Protection Rule ในระดับ repository เดียว ซึ่งมีข้อจำกัดสำคัญเมื่อ scale ใหญ่:

1. **ต้องตั้งค่าซ้ำทุก repo** — ถ้าองค์กรมี 5,000 repository การไปตั้งค่าทีละตัวเป็นไปไม่ได้ในทางปฏิบัติ
2. **repo ใหม่ที่ถูกสร้างขึ้นไม่มีการป้องกันตั้งแต่แรก** — ต้องรอให้มีคนมาตั้งค่าเอง ซึ่งมักถูกลืม
3. **Repository admin สามารถปิดการป้องกันได้เอง** — เพราะ Branch Protection ระดับ repo มักแก้ไขได้โดย admin ของ repo นั้น ทำให้นโยบายความปลอดภัยส่วนกลางถูก override ได้ง่าย
4. **ไม่มีการรายงานภาพรวม** — ไม่สามารถตอบคำถามได้ว่า "มีกี่ repo ที่ไม่ได้เปิด required review" โดยไม่ต้องไล่เช็คทีละตัว

### 824.2 Organization-level Ruleset คืออะไร

**Ruleset ระดับองค์กร** คือกลไกที่ให้ผู้ดูแลระดับ organization กำหนดกฎที่ **มีผลบังคับกับหลาย repository พร้อมกัน** โดยอาศัย pattern matching เลือก repository ที่กฎนั้นจะครอบคลุม และสำคัญที่สุดคือ **admin ของ repository แต่ละตัวไม่สามารถ override กฎระดับองค์กรได้** (เว้นแต่จะได้รับสิทธิ์ bypass ที่กำหนดไว้อย่างชัดเจน)

**ตัวอย่างแนวคิด Ruleset ระดับองค์กร (GitHub Enterprise):**

```yaml
# แนวคิด Organization Ruleset (ตั้งค่าจริงผ่านหน้าเว็บ/API ของ GitHub)
name: "นโยบายความปลอดภัยขั้นต่ำทั้งองค์กร"
target: branch
enforcement: active
conditions:
  repository_name:
    include:
      - "*"                     # ครอบคลุมทุก repository ในองค์กร
    exclude:
      - "sandbox-*"             # ยกเว้น repo ทดลองส่วนตัว
rules:
  - type: pull_request
    parameters:
      required_approving_review_count: 1
      require_code_owner_review: true
      dismiss_stale_reviews_on_push: true
  - type: required_status_checks
    parameters:
      required_checks:
        - context: "security/secret-scan"
        - context: "security/dependency-check"
  - type: deletion
    parameters: {}              # ห้ามลบ branch ที่ตรงเงื่อนไข
  - type: non_fast_forward
    parameters: {}              # ห้าม force-push ทับประวัติ
bypass_actors:
  - actor: "security-incident-response-team"
    mode: "always"              # อนุญาตให้ทีมนี้ bypass ได้ในกรณีฉุกเฉินเท่านั้น
```

**ประเด็นสำคัญของ Ruleset ระดับองค์กร:**

1. **Layered enforcement (การบังคับใช้แบบหลายชั้น)** — Ruleset สามารถกำหนดแยกตาม "ชั้น" ได้ เช่น
   - ชั้นที่ 1: กฎขั้นต่ำที่บังคับทุก repo (เช่น ห้าม force-push บน branch หลัก)
   - ชั้นที่ 2: กฎเฉพาะกลุ่ม repo ที่มีความเสี่ยงสูง (เช่น repo ที่แตะข้อมูลการเงินต้องมี 2 approval ขึ้นไป)
   - ชั้นที่ 3: กฎเพิ่มเติมที่แต่ละทีมตั้งเองได้ตามความต้องการเฉพาะทาง (เข้ากับแนวคิด "Freedom within a Framework" จาก Step 821)

2. **Targeting ด้วย pattern และ property** — ไม่จำเป็นต้องระบุชื่อ repo ทีละตัว สามารถใช้ wildcard, topic/label ของ repo หรือ custom property (เช่น `data-classification: confidential`) เพื่อกำหนดว่ากฎไหนใช้กับกลุ่มไหน

3. **Bypass ที่ถูกควบคุมและมี audit log** — ในกรณีฉุกเฉินจริง ๆ (เช่น incident ที่ต้อง hotfix ด่วนมาก) ควรมีกลไก bypass ที่จำกัดเฉพาะบุคคล/ทีมที่กำหนดไว้ล่วงหน้า และทุกครั้งที่ bypass ถูกใช้งานต้องมี log บันทึกไว้เพื่อ audit ย้อนหลัง — ไม่ใช่การปิดกฎแบบไม่มีร่องรอย

### 824.3 GitLab Compliance Framework — แนวคิดที่คล้ายกัน

ฝั่ง GitLab ใช้แนวคิดที่เรียกว่า **Compliance Framework** ซึ่งทำหน้าที่คล้ายกัน:

```yaml
# ตัวอย่างแนวคิด Compliance Framework Label ของ GitLab
name: "SOX-Compliant"
description: "โครงการที่อยู่ภายใต้การควบคุมตาม Sarbanes-Oxley Act"
pipeline_configuration: compliance-pipelines/sox.yml
requirements:
  - require_two_approvers: true
  - prevent_approval_by_author: true
  - prevent_approval_by_committer: true
  - require_signed_commits: true
```

- Compliance Framework ผูก **project** เข้ากับชุดกฎ (label) และสามารถบังคับ **compliance pipeline** ที่รันทับหรือร่วมกับ pipeline ปกติของ project นั้น
- ใช้สำหรับ project ที่อยู่ภายใต้ regulation เฉพาะ เช่น การเงิน สุขภาพ หรือข้อมูลส่วนบุคคล ซึ่งต้องมีการควบคุมที่เข้มกว่า project ทั่วไป

### 824.4 หลักการออกแบบ Governance ที่ดี

จากประสบการณ์จริงขององค์กรระดับ enterprise หลักการสำคัญในการออกแบบ governance คือ:

1. **Default-secure, opt-out ที่มี friction** — ค่าเริ่มต้นของทุก repository ใหม่ควรปลอดภัยที่สุดเท่าที่จะทำได้ (secure by default) การจะลดระดับความปลอดภัยลงควรต้องมีขั้นตอนขออนุมัติที่ชัดเจน ไม่ใช่ทำได้ง่าย ๆ ด้วย 1 คลิก
2. **Policy as Code** — เขียนกฎ governance เป็นไฟล์ configuration ที่เก็บใน version control เอง (เช่น เก็บ Ruleset definition เป็น Terraform หรือ YAML ใน repo กลาง) เพื่อให้ตรวจสอบ (review), ย้อนดูประวัติ (audit) และ rollback ได้เหมือนโค้ดทั่วไป
3. **แยกกฎที่ "ต้องทำ" (mandatory) กับ "แนะนำ" (recommended) อย่างชัดเจน** — ไม่ใช่ทุกอย่างต้องบังคับ บางเรื่องปล่อยให้เป็น best practice ที่แนะนำก็เพียงพอ การบังคับทุกอย่างจะทำลายความคล่องตัวของทีม
4. **มี Exception process ที่โปร่งใส** — ทีมที่มีเหตุผลทางธุรกิจที่ต้อง exception จากกฎกลาง ควรมีช่องทางขออนุมัติที่มีบันทึก ไม่ใช่การหลบเลี่ยงแบบไม่เป็นทางการ
5. **วัดผลการปฏิบัติตามกฎ (compliance rate) เป็น metric** — ควรมี dashboard ที่แสดง "กี่ % ของ repository ที่ปฏิบัติตามนโยบายกลางครบถ้วน" เพื่อติดตามแนวโน้มและหาทีมที่ต้องช่วยเหลือเพิ่มเติม

---

## Step 825: Internal Open Source / InnerSource Concept

หนึ่งในแนวคิดที่ทรงพลังที่สุดสำหรับองค์กรระดับ enterprise ที่ต้องการเพิ่มประสิทธิภาพการใช้โค้ดร่วมกันคือ **InnerSource** — การนำวัฒนธรรมและวิธีปฏิบัติของ Open Source มาใช้ **ภายในองค์กรของตัวเอง**

### 825.1 InnerSource คืออะไร

**InnerSource** (บางครั้งเขียนว่า "Inner Source") คือแนวคิดที่นำ**หลักปฏิบัติ**ของโครงการ Open Source สาธารณะ — เช่น Linux Kernel, Kubernetes — มาประยุกต์ใช้กับโค้ดที่ยัง **เป็นกรรมสิทธิ์ภายในบริษัท (proprietary)** โดยไม่ได้เปิดเผยสู่สาธารณะ

หลักการสำคัญที่ยืมมาจาก Open Source ได้แก่:

1. **Code visibility ข้ามทีม** — ใครในองค์กรก็สามารถ "อ่าน" โค้ดของทีมอื่นได้ (ไม่จำเป็นต้องเขียนได้) เหมือนที่ทุกคนอ่านโค้ด Open Source บน GitHub ได้
2. **Contribution จากภายนอกทีม** — วิศวกรจากทีม A สามารถส่ง Pull Request ไปยัง repository ของทีม B ได้โดยตรง แทนที่จะต้องรอทีม B ทำให้ หรือต้องเปิด ticket ขอความช่วยเหลือ
3. **Trusted Committer model** — แต่ละ InnerSource project มี "Trusted Committer" (เทียบเท่า maintainer ของ Open Source) ที่ทำหน้าที่ review และ merge contribution จากทีมอื่น
4. **เอกสารประกอบที่เปิดกว้าง** — README, CONTRIBUTING.md, Architecture Decision Record (ADR) ที่เขียนดีพอให้คนนอกทีมเข้าใจและ contribute ได้โดยไม่ต้องถามคนในทีมตลอดเวลา

### 825.2 ปัญหาที่ InnerSource แก้

**ปัญหา: การเขียนโค้ดซ้ำซ้อนข้ามทีม (Duplicate effort)**

ในองค์กรที่ไม่มี InnerSource มักเกิดสถานการณ์แบบนี้:

```
ทีม A ต้องการ library สำหรับ retry logic กับ external API
  → ทีม A ไม่รู้ว่าทีม B เคยเขียน library แบบนี้ไว้แล้วเมื่อ 6 เดือนก่อน
  → ทีม A เขียนใหม่เองทั้งหมด ใช้เวลา 2 สัปดาห์
  → เกิด library retry-logic 2 เวอร์ชันที่ทำงานคล้ายกันแต่ไม่เหมือนกันเป๊ะ
  → เมื่อพบบั๊กด้านความปลอดภัยใน pattern retry logic แบบหนึ่ง
    ทีมความปลอดภัยต้องตามแก้ 2 ที่แยกกัน (และอาจมีเวอร์ชันที่ 3, 4, 5 ที่ไม่มีใครรู้)
```

InnerSource แก้ปัญหานี้โดยทำให้ library ของทีม B **มองเห็นได้และ contribute ได้ง่าย** จนทีม A เลือกที่จะไปเสนอ Pull Request เพิ่ม feature ที่ต้องการเข้า library เดิม แทนที่จะเขียนใหม่

**ปัญหา: Silo ความรู้ (Knowledge Silo)**

เมื่อโค้ดของแต่ละทีมถูกปิดไม่ให้ทีมอื่นเห็น (private by default แบบเข้มงวดเกินจำเป็น) ความรู้ทางเทคนิคจะไม่ไหลข้ามทีม InnerSource เปิดให้ "อ่าน" ได้อย่างน้อยที่สุด ทำให้วิศวกรเรียนรู้ pattern ที่ดีจากทีมอื่นได้ (เช่น เห็นว่าทีม platform เขียน error handling อย่างไร แล้วนำ pattern นั้นไปใช้)

### 825.3 องค์ประกอบของโปรแกรม InnerSource ที่ประสบความสำเร็จ

**1. Default visibility ที่เปิดกว้างภายในองค์กร**

Repository ควรตั้งค่าเป็น **"Internal"** (มองเห็นได้จากทุกคนในองค์กร แต่ไม่ public สู่ภายนอก) เป็นค่าเริ่มต้น แทนที่จะเป็น "Private" (มองเห็นได้เฉพาะสมาชิกทีมที่ถูก invite เท่านั้น) เว้นแต่มีเหตุผลชัดเจนที่ต้องปิดเป็น private จริง ๆ (เช่น โค้ดที่เกี่ยวกับความปลอดภัยขั้นสูงมาก หรือข้อมูลที่ sensitive เป็นพิเศษ)

**2. เอกสารมาตรฐานที่ทุก InnerSource project ต้องมี**

```
project-root/
├── README.md              # อธิบายว่าโปรเจกต์นี้ทำอะไร ใช้งานอย่างไร
├── CONTRIBUTING.md        # วิธี contribute: coding standard, วิธี submit PR, วิธีรัน test
├── CODEOWNERS              # ใครคือ Trusted Committer ที่ต้อง review
├── GOVERNANCE.md          # กระบวนการตัดสินใจของโปรเจกต์ (ใครมีสิทธิ์ตัดสินใจอะไร)
└── docs/
    └── architecture.md    # ภาพรวมสถาปัตยกรรมสำหรับคนนอกทีมที่อยากเข้าใจเร็ว
```

**3. บทบาท Trusted Committer**

Trusted Committer คือวิศวกรในทีมเจ้าของ project ที่ได้รับมอบหมายให้ทำหน้าที่คล้าย maintainer ของ Open Source:
- Review และ merge Pull Request จากทั้งคนในทีมและคนนอกทีม
- รักษาคุณภาพและทิศทางของโปรเจกต์
- ให้คำแนะนำกับ contributor ใหม่ที่ไม่คุ้นเคยกับ codebase

**4. Recognition และ Incentive**

องค์กรที่ InnerSource สำเร็จมักมีระบบยกย่อง contributor ข้ามทีม เช่น leaderboard ภายใน, การนับ InnerSource contribution เป็นส่วนหนึ่งของการประเมินผลงานประจำปี เพื่อสร้างแรงจูงใจให้วิศวกร "ให้เวลา" กับการช่วยเหลือโปรเจกต์อื่นนอกทีมตัวเอง ซึ่งปกติจะถูกมองข้ามเพราะไม่ใช่ OKR ตรงของทีมตัวเอง

### 825.4 ระดับความเป็นผู้ใหญ่ของ InnerSource (InnerSource Maturity Model)

InnerSource Commons Foundation (องค์กรไม่แสวงหาผลกำไรที่พัฒนามาตรฐาน InnerSource) แบ่งระดับความเป็นผู้ใหญ่ขององค์กรออกเป็นประมาณนี้:

| ระดับ | ลักษณะ |
|---|---|
| **1. Ad hoc** | มีการแชร์โค้ดข้ามทีมแบบไม่เป็นทางการ ขึ้นกับความสัมพันธ์ส่วนตัวของวิศวกร |
| **2. Repeatable** | มีบาง project ที่ตั้งใจเปิดรับ contribution จากนอกทีมอย่างเป็นระบบ มี CONTRIBUTING.md |
| **3. Defined** | มีนโยบายระดับองค์กรที่ชัดเจนว่า repo แบบไหนควรเป็น InnerSource มี tooling รองรับ |
| **4. Managed** | มีการวัดผล (metric) ของ InnerSource activity เช่น จำนวน cross-team PR ต่อเดือน และใช้ข้อมูลนี้ปรับปรุงกระบวนการ |
| **5. Optimizing** | InnerSource เป็นส่วนหนึ่งของวัฒนธรรมองค์กรอย่างเต็มรูปแบบ ทีมใหม่ที่เข้ามาซึมซับวิธีคิดนี้โดยอัตโนมัติ |

องค์กรส่วนใหญ่ที่เริ่มต้นมักอยู่ระดับ 1-2 การขยับไประดับ 3 ขึ้นไปต้องอาศัยการสนับสนุนจากผู้บริหารระดับสูง (executive sponsorship) และทีม Platform/Developer Experience ที่ลงทุนสร้าง tooling รองรับอย่างจริงจัง

---

## Step 826: Standardized CI/CD Templates ทั่วทั้งองค์กร

ปัญหาหนึ่งที่ตามมาจาก governance ใน Step 824 คือ ถ้าแต่ละทีมเขียน CI/CD pipeline เองตั้งแต่ศูนย์ (from scratch) จะเกิดความไม่สม่ำเสมอมหาศาล และเป็นภาระในการบำรุงรักษาเมื่อองค์กรต้องการเปลี่ยนมาตรฐาน (เช่น เปลี่ยน security scanner ตัวใหม่) เพราะต้องไปแก้ทุก repository ทีละตัว

ทางแก้คือ **Standardized CI/CD Templates** ซึ่งต่อยอดโดยตรงจากสิ่งที่เราเรียนไปแล้วใน **Part 70 (GitHub Actions: Reusable Workflows)** และ **Part 72 (GitLab CI/CD Components)**

### 826.1 ทบทวน: Reusable Workflows และ CI/CD Components คืออะไร

จาก Part 70 เรารู้ว่า **Reusable Workflow** ของ GitHub Actions ช่วยให้ repository หนึ่งเรียกใช้ workflow ที่นิยามไว้ใน repository กลางได้ ผ่าน `workflow_call`:

```yaml
# .github/workflows/ci.yml ใน repo ของแต่ละทีม
jobs:
  call-standard-pipeline:
    uses: my-org/ci-templates/.github/workflows/standard-build.yml@v2
    with:
      node-version: "20"
    secrets: inherit
```

จาก Part 72 เรารู้ว่า **CI/CD Components** ของ GitLab (รูปแบบใหม่ที่มาแทน `include` template แบบเดิม) ทำหน้าที่คล้ายกัน:

```yaml
# .gitlab-ci.yml ใน repo ของแต่ละทีม
include:
  - component: gitlab.company.com/platform/ci-components/security-scan@2.1.0
    inputs:
      severity_threshold: high
```

### 826.2 การขยายแนวคิดนี้สู่ระดับองค์กร: CI/CD Template Catalog

ในระดับ enterprise สิ่งที่ต้องเพิ่มเติมจากแค่ "การมี reusable workflow" คือการสร้าง **Template Catalog** ที่เป็นทางการ ซึ่งมีองค์ประกอบดังนี้:

**1. Golden Path Templates (เส้นทางมาตรฐานที่แนะนำ)**

องค์กรกำหนด "Golden Path" — ชุด template มาตรฐานสำหรับแต่ละประเภทงานทั่วไป เช่น:

```
ci-templates/
├── golden-path/
│   ├── nodejs-service.yml       # มาตรฐานสำหรับ Node.js microservice
│   ├── python-batch-job.yml     # มาตรฐานสำหรับ Python batch job
│   ├── java-spring-service.yml  # มาตรฐานสำหรับ Java Spring service
│   ├── terraform-module.yml     # มาตรฐานสำหรับ Terraform module
│   └── frontend-react-app.yml   # มาตรฐานสำหรับ React frontend
├── components/
│   ├── security-scan.yml        # component ย่อยที่ template หลักเรียกใช้
│   ├── dependency-audit.yml
│   ├── sbom-generation.yml       # สร้าง Software Bill of Materials
│   └── deploy-to-k8s.yml
└── docs/
    └── how-to-adopt.md
```

แนวคิด "Golden Path" มาจากคำว่า **"Paved Road"** ที่ Netflix ใช้เรียกวิธีการนี้ — เส้นทางที่องค์กร**ลาดยางให้เรียบร้อยและง่ายที่สุด**สำหรับกรณีการใช้งานทั่วไป ทีมที่เดินตาม paved road จะได้รับการสนับสนุนเต็มที่จากทีม platform แต่ทีมที่ต้องการทำอะไรพิเศษนอกเส้นทางนี้ก็ยังทำได้ เพียงแต่ต้องรับผิดชอบดูแลเองมากขึ้น

**2. Version Pinning และ Migration Strategy**

Template กลางต้องมีระบบ versioning ที่ชัดเจน (semantic versioning) เพื่อให้ทีมย่อยควบคุมได้ว่าจะอัปเดตเมื่อไหร่:

```yaml
# ทีมที่ต้องการความเสถียรสูงสุด — pin เวอร์ชันตายตัว
uses: my-org/ci-templates/.github/workflows/nodejs-service.yml@v3.2.1

# ทีมที่ยอมรับการอัปเดต patch อัตโนมัติ
uses: my-org/ci-templates/.github/workflows/nodejs-service.yml@v3

# ไม่แนะนำสำหรับ production — ใช้เวอร์ชันล่าสุดเสมอ (ความเสี่ยงสูง)
uses: my-org/ci-templates/.github/workflows/nodejs-service.yml@main
```

เมื่อต้องเปลี่ยนแปลงแบบ breaking change (เช่น เปลี่ยน security scanner ตัวใหม่ทั้งหมด) ทีม platform ควรมีแผน migration ที่ชัดเจน เช่น ประกาศ deprecation ล่วงหน้า 90 วัน พร้อม migration guide และ automated codemod (สคริปต์ช่วยแก้โค้ดอัตโนมัติ) ถ้าเป็นไปได้

**3. Compliance Gate ที่ฝังอยู่ใน Template**

Template กลางเป็นจุดที่เหมาะที่สุดในการฝังการตรวจสอบด้าน compliance/security แบบรวมศูนย์ เพราะทีม platform แก้ครั้งเดียวมีผลกับทุก repository ที่ใช้ template นั้นทันที:

```yaml
# ตัวอย่างส่วนหนึ่งของ golden-path template
jobs:
  mandatory-security-gate:
    steps:
      - name: SAST scan (บังคับ ไม่สามารถข้ามได้)
        uses: my-org/actions/sast-scan@v1
      - name: License compliance check
        uses: my-org/actions/license-check@v1
      - name: SBOM generation
        uses: my-org/actions/generate-sbom@v1
      - name: Sign artifact (supply chain security)
        uses: my-org/actions/sign-artifact@v1
```

**4. Self-service Onboarding**

Template ที่ดีต้องมี **CLI หรือ scaffolding tool** ที่ทำให้ทีมใหม่เริ่มต้นใช้งานได้เองโดยไม่ต้องขอความช่วยเหลือจากทีม platform ทุกครั้ง เช่น คำสั่งเดียวที่ generate โครง repository พร้อม CI/CD ที่ผูกกับ golden path ไว้ให้เรียบร้อยตั้งแต่ต้น

### 826.3 ข้อควรระวัง: อย่าทำให้ Template กลายเป็นคอขวด

การรวมศูนย์ CI/CD มากเกินไปมีความเสี่ยงด้านลบเช่นกัน:

- **Blast radius ใหญ่เมื่อ template มีบั๊ก** — ถ้า golden path template มีข้อผิดพลาด อาจทำให้ pipeline ของหลายร้อย repository พังพร้อมกัน จึงต้องมีกระบวนการ test template เองอย่างเข้มงวด (dogfooding, canary rollout ให้บาง repo ทดลองก่อน)
- **ทีม platform กลายเป็นคอขวดของทุกคำขอเปลี่ยนแปลง** — ควรออกแบบให้ templateมีจุดที่ทีมย่อย customize ได้เอง (extension points) โดยไม่ต้องรอทีม platform แก้ template หลักทุกครั้ง
- **Version fragmentation** — ถ้าไม่มีการผลักดันให้ทีมอัปเดต version ของ template สม่ำเสมอ จะเกิดสถานการณ์ที่มีหลายสิบ version ของ template ใช้งานพร้อมกันทั่วองค์กร ทำให้การบำรุงรักษายากขึ้นเรื่อย ๆ

---

## Step 827: Developer Portal / Internal Developer Platform (IDP) เบื้องต้น

เมื่อจำนวน repository, service, และ template มากขึ้นเรื่อย ๆ ตามที่กล่าวใน Step ก่อนหน้า จะถึงจุดที่วิศวกรไม่สามารถ "จำ" ได้อีกต่อไปว่า service ไหนอยู่ที่ไหน ใครดูแล ใช้ template อะไร มี documentation ที่ไหน — นี่คือจุดที่องค์กรระดับ enterprise เริ่มลงทุนสร้าง **Developer Portal** หรือในชื่อที่เป็นทางการกว่าคือ **Internal Developer Platform (IDP)**

### 827.1 ปัญหาที่ Developer Portal แก้

ลองนึกภาพวิศวกรคนหนึ่งที่ join บริษัทใหม่ในสัปดาห์แรก โดยไม่มี Developer Portal:

- ต้องถามเพื่อนร่วมทีมว่า "service X repository อยู่ที่ไหน"
- ต้องขอ access เข้า repository ทีละตัวผ่านการเปิด ticket แล้วรอ approve
- ไม่รู้ว่า service ที่ตัวเองดูแลอยู่ depend on service อะไรบ้าง (dependency graph)
- เมื่อเกิด incident ตอนดึก ไม่รู้ว่าจะติดต่อทีมเจ้าของ service ที่เกี่ยวข้องได้อย่างไร
- ต้องการสร้าง microservice ใหม่ ต้อง copy-paste จาก repo เก่าแล้วแก้เอง เพราะไม่รู้ว่ามี template มาตรฐานให้ใช้

Developer Portal แก้ปัญหาทั้งหมดนี้โดยเป็น **"หน้าต่างเดียว" (single pane of glass)** ที่รวมข้อมูลทุกอย่างเกี่ยวกับ software ecosystem ขององค์กรไว้ในที่เดียว

### 827.2 Backstage — ตัวอย่างที่มีชื่อเสียงที่สุด

**Backstage** คือ Internal Developer Platform แบบ open source ที่พัฒนาโดย **Spotify** และภายหลังบริจาคให้ **Cloud Native Computing Foundation (CNCF)** ดูแลต่อ ปัจจุบันเป็นหนึ่งใน CNCF graduated project และถูกใช้อย่างแพร่หลายในองค์กรระดับ enterprise ทั่วโลก (เช่น American Airlines, Netflix บางส่วน, Epic Games)

**องค์ประกอบหลักของ Backstage:**

**1. Software Catalog**

หัวใจของ Backstage คือ **Software Catalog** — ฐานข้อมูลกลางที่เก็บ metadata ของทุก component (service, library, website, resource) ในองค์กร โดยแต่ละ repository ประกาศตัวเองผ่านไฟล์ `catalog-info.yaml` (ตัวอย่างที่เราเห็นแล้วใน Step 822):

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: order-service
  description: บริการจัดการคำสั่งซื้อของระบบ E-commerce
  tags:
    - java
    - spring-boot
  links:
    - url: https://wiki.company.com/order-service
      title: Runbook
spec:
  type: service
  lifecycle: production
  owner: team-order-management
  system: ecommerce-platform
  dependsOn:
    - component:default/payment-service
    - resource:default/orders-database
```

จากไฟล์นี้ Backstage สามารถสร้าง:
- หน้าเว็บที่แสดงรายละเอียดของ `order-service` ทั้งหมด รวมถึง link ไปยัง repository, CI/CD status, documentation, on-call schedule
- **Dependency graph** ที่แสดงว่า service ไหน depend on service ไหน — มีประโยชน์มากตอนวิเคราะห์ผลกระทบก่อน deploy หรือตอนสืบสวน incident
- รายการ component ทั้งหมดที่ทีมหนึ่งเป็นเจ้าของ

**2. Software Templates (Scaffolder)**

ฟีเจอร์ที่เชื่อมโยงตรงกับ Step 826 — Backstage มีระบบ **Scaffolder** ที่ให้วิศวกรสร้าง repository ใหม่จาก Golden Path Template ได้ผ่าน UI แบบฟอร์มง่าย ๆ โดยไม่ต้องเขียน boilerplate เอง:

```
วิศวกรเปิดหน้า Backstage → เลือก "Create Component"
  → เลือก template "Node.js Microservice (Golden Path)"
  → กรอกฟอร์ม: ชื่อ service, ทีมเจ้าของ, database ที่ต้องการ
  → กด "Create"
  → Backstage สร้าง repository ใหม่ให้อัตโนมัติ พร้อม:
      - CI/CD pipeline ที่ผูกกับ golden path template
      - catalog-info.yaml ที่กรอกข้อมูลเจ้าของไว้แล้ว
      - Branch protection ที่ตั้งค่าตามมาตรฐานองค์กรทันที
      - Slack channel และ on-call schedule เริ่มต้น (ถ้ามีการเชื่อมต่อ integration)
```

นี่คือการนำ governance (Step 824) และ template มาตรฐาน (Step 826) มาผูกเข้ากับประสบการณ์ผู้ใช้ที่ดี ทำให้ **"ทางที่ถูกต้องเป็นทางที่ง่ายที่สุด"** (the right way is the easy way) ซึ่งเป็นหลักการสำคัญของ Platform Engineering ยุคใหม่

**3. TechDocs**

ระบบ documentation ที่ให้แต่ละทีมเขียน documentation เป็น Markdown เก็บไว้**ในตัว repository เอง** (docs-as-code) แล้ว Backstage จะ build และแสดงผลรวมศูนย์ไว้ในหน้าเดียวกับ Software Catalog — แก้ปัญหา documentation กระจัดกระจายอยู่คนละที่ (Confluence, Notion, Wiki, README ของแต่ละ repo)

**4. Plugin Ecosystem**

Backstage ออกแบบมาให้ขยายได้ผ่านระบบ plugin เช่น plugin แสดงผล CI/CD status จาก GitHub Actions/GitLab CI, plugin แสดง cost ของ cloud resource, plugin เชื่อมกับระบบ incident management เป็นต้น องค์กรสามารถเขียน plugin เองเพื่อเชื่อมกับเครื่องมือภายในที่มีอยู่แล้ว

### 827.3 ทางเลือกอื่นนอกจาก Backstage

Backstage ไม่ใช่ตัวเลือกเดียว องค์กรอาจพิจารณาทางเลือกอื่นตามความเหมาะสม:

| แพลตฟอร์ม | ลักษณะ |
|---|---|
| **Backstage** | Open source, ปรับแต่งได้สูงสุด, ต้องดูแลเอง (self-hosted), เหมาะกับองค์กรที่มีทีม platform engineering แข็งแรง |
| **Port** | SaaS-based Internal Developer Portal, ตั้งค่าเร็วกว่า Backstage, มี pricing model ตาม seat |
| **Cortex** | เน้น Service Catalog และ Scorecard (ให้คะแนนคุณภาพของแต่ละ service ตามเกณฑ์ที่กำหนด) |
| **GitHub/GitLab เอง** | ทั้งสอง platform เริ่มมีฟีเจอร์ในตัวที่ทำหน้าที่บางส่วนของ IDP เช่น GitHub มี "Repository Insights", GitLab มี "Value Stream Management" ซึ่งเหมาะกับองค์กรที่ไม่ต้องการเพิ่มเครื่องมือใหม่ |

### 827.4 IDP กับ Platform Engineering

Developer Portal เป็นเพียงส่วนหนึ่ง (มักเป็น "หน้าตา" ที่มองเห็นได้) ของแนวคิดที่ใหญ่กว่าคือ **Platform Engineering** — การสร้างทีมภายในที่ทำหน้าที่เหมือน "ผู้ให้บริการภายใน" (internal product team) ที่มองวิศวกรคนอื่นในองค์กรเป็น "ลูกค้า" (customer) ของตัวเอง

หลักคิดสำคัญคือ Internal Developer Platform ที่ดีต้องถูกสร้างและดูแลเหมือนสร้าง product จริง ๆ — มีการเก็บ feedback จากผู้ใช้ (วิศวกรในองค์กร) มี roadmap มีการวัดผลความพึงพอใจ ไม่ใช่แค่ "ทำเสร็จแล้ววางทิ้งไว้"

---

## Step 828: Metrics ระดับองค์กร — เกริ่น DORA Metrics

เมื่อองค์กรลงทุนกับ governance, InnerSource, CI/CD templates และ Developer Portal ตามที่กล่าวมาทั้งหมด คำถามสำคัญคือ **"เรารู้ได้อย่างไรว่าสิ่งที่ลงทุนไปมันได้ผลจริง"** นี่คือบทบาทของ metrics ระดับองค์กร

### 828.1 ทำไมต้องวัดผลในระดับองค์กร ไม่ใช่แค่ระดับทีม

ในทีมเล็ก การรู้ว่าทีมทำงานได้ดีหรือไม่อาจอาศัยความรู้สึกและการสังเกตแบบไม่เป็นทางการก็เพียงพอ แต่เมื่อมีทีมนับร้อยทีม ผู้บริหารด้านวิศวกรรม (VP Engineering, CTO) ต้องการข้อมูลเชิงปริมาณที่**เปรียบเทียบกันได้ระหว่างทีม** เพื่อ:

- ระบุว่าทีมไหนกำลังประสบปัญหาและต้องการความช่วยเหลือเพิ่มเติม (ไม่ใช่เพื่อลงโทษ แต่เพื่อสนับสนุน)
- วัดผลตอบแทนของการลงทุนด้าน platform engineering (ROI) — เช่น หลังจากทำ golden path template ความเร็วในการ deploy เพิ่มขึ้นจริงหรือไม่
- สื่อสารกับผู้บริหารระดับสูงเกี่ยวกับสุขภาพโดยรวมของ engineering organization ด้วยตัวเลขที่เป็นกลาง

### 828.2 DORA Metrics คืออะไร (ภาพรวมเบื้องต้น)

**DORA** ย่อมาจาก **DevOps Research and Assessment** ซึ่งเป็นทีมวิจัยที่เริ่มต้นจาก Dr. Nicole Forsgren, Jez Humble และ Gene Kim ผ่านหนังสือ **"Accelerate"** อันโด่งดัง และภายหลังกลายเป็นทีมวิจัยภายใต้ Google Cloud

จากการวิจัยหลายปีกับองค์กรหลายพันแห่งทั่วโลก DORA พบว่ามี **4 ตัวชี้วัดหลัก (key metrics)** ที่สามารถแยกแยะองค์กรที่มีประสิทธิภาพการพัฒนาซอฟต์แวร์สูง (elite performer) ออกจากองค์กรที่มีประสิทธิภาพต่ำได้อย่างชัดเจน:

| Metric | ความหมาย | คำถามที่ตอบ |
|---|---|---|
| **Deployment Frequency** | ความถี่ในการ deploy โค้ดขึ้น production | "ทีมนี้ deploy บ่อยแค่ไหน" |
| **Lead Time for Changes** | เวลาตั้งแต่ commit โค้ดจนถึง deploy ขึ้น production สำเร็จ | "จากไอเดียถึงมือผู้ใช้ใช้เวลานานแค่ไหน" |
| **Change Failure Rate** | สัดส่วนของการ deploy ที่ทำให้เกิดปัญหาใน production (ต้อง rollback, hotfix, หรือ incident) | "การเปลี่ยนแปลงที่ปล่อยออกไปมีคุณภาพแค่ไหน" |
| **Time to Restore Service (MTTR)** | เวลาเฉลี่ยที่ใช้ในการกู้คืนระบบเมื่อเกิด incident | "เมื่อพังแล้ว กลับมาปกติเร็วแค่ไหน" |

**ข้อค้นพบสำคัญจากงานวิจัย DORA** คือ ตัวชี้วัดทั้ง 4 นี้ **ไม่ได้ขัดแย้งกันเอง** — องค์กรที่ deploy บ่อยและเร็ว (velocity สูง) ไม่ได้แปลว่าคุณภาพต่ำลงเสมอไป ในทางตรงกันข้าม องค์กรระดับ elite performer มักทำได้ **ทั้งเร็วและมีคุณภาพสูงไปพร้อมกัน** เพราะการ deploy บ่อยครั้งด้วย batch ขนาดเล็กมักมีความเสี่ยงต่อครั้งต่ำกว่าการ deploy แบบ batch ใหญ่ไม่บ่อยครั้ง

### 828.3 ทำไมต้องเกริ่นในที่นี้ (และทำไมยังไม่เจาะลึก)

Git และ Git hosting platform คือ **แหล่งข้อมูลดิบที่สำคัญที่สุด** สำหรับคำนวณ DORA Metrics เพราะ:
- Deployment Frequency คำนวณได้จากจำนวน merge เข้า main branch หรือจำนวน deployment ที่ trigger จาก CI/CD pipeline
- Lead Time for Changes คำนวณได้จากช่วงเวลาระหว่าง first commit ของ Pull Request จนถึงเวลาที่ pipeline deploy สำเร็จ
- Change Failure Rate คำนวณได้จากอัตราส่วนของ deployment ที่ตามมาด้วย revert commit, hotfix PR หรือ incident ที่เชื่อมโยงกับ deployment นั้น

การจัดการ repository ระดับ enterprise ที่ดี (governance ที่สม่ำเสมอ, CI/CD template มาตรฐาน, metadata ที่ครบถ้วนใน Software Catalog) เป็น**เงื่อนไขเบื้องต้นที่จำเป็น**ต่อการเก็บข้อมูล DORA Metrics ได้อย่างแม่นยำและสม่ำเสมอทั่วทั้งองค์กร — ถ้าแต่ละทีมมี pipeline ที่ทำงานคนละแบบ การรวบรวมข้อมูลเปรียบเทียบข้ามทีมจะทำได้ยากมากหรือทำไม่ได้เลย

เนื่องจากเนื้อหาเรื่อง DORA Metrics มีรายละเอียดลึกมาก ทั้งวิธีคำนวณอย่างละเอียด เครื่องมือที่ใช้เก็บข้อมูล (เช่น Google Cloud's DORA quick check, Sleuth, LinearB, Faros AI) การตีความผลลัพธ์ และการนำไปปรับปรุงกระบวนการทำงานจริง เราจะ**เจาะลึกเรื่องนี้แบบเต็มรูปแบบใน Part 92** ซึ่งอยู่ในเฟส 9 ของหลักสูตร (ทักษะมืออาชีพ: Maintainer, Release Management, Metrics) — ใน Part นี้ขอให้จำแค่หลักการและความสำคัญของมันไว้ก่อน

---

## Step 829: Cost Management ของ Git Hosting ระดับ Enterprise

การตัดสินใจด้าน governance, tooling และ platform ทั้งหมดที่กล่าวมาไม่ได้เกิดขึ้นในสุญญากาศ — ทุกอย่างมีต้นทุน และในระดับ enterprise ต้นทุนของ Git hosting สามารถสูงถึงหลักล้านดอลลาร์ต่อปีสำหรับองค์กรขนาดใหญ่จริง ๆ การเข้าใจโครงสร้างต้นทุนจึงเป็นทักษะสำคัญของวิศวกรระดับ senior/staff ที่ต้องมีส่วนร่วมในการตัดสินใจเชิงสถาปัตยกรรม

### 829.1 องค์ประกอบหลักของต้นทุน

**1. Seat-based licensing (ค่าธรรมเนียมตามจำนวนผู้ใช้)**

ทั้ง GitHub Enterprise และ GitLab (Premium/Ultimate tier) คิดค่าบริการเป็นรายเดือนต่อ "seat" (ต่อผู้ใช้ 1 คนที่มี license) โดยทั่วไปมีหลายระดับ tier:

| Tier ตัวอย่าง (แนวคิดทั่วไป ราคาจริงเปลี่ยนแปลงตามเวลา) | ลักษณะ |
|---|---|
| Free / Community | ฟีเจอร์พื้นฐาน จำกัดจำนวน collaborator หรือ CI minutes |
| Team / Premium | เพิ่ม feature ด้าน collaboration, protected branch, เพิ่ม CI minutes |
| Enterprise / Ultimate | SSO/SAML/SCIM, audit log, advanced security scanning, compliance framework, support ระดับองค์กร |

ประเด็นสำคัญคือ **ไม่ใช่ทุกคนต้องการ tier สูงสุด** — องค์กรควรวิเคราะห์ว่าใครต้องการฟีเจอร์ enterprise จริง ๆ (เช่น วิศวกร full-time ที่ push โค้ดทุกวัน) กับใครที่แค่ต้องการดูโค้ดเป็นครั้งคราว (เช่น product manager, ทีมสนับสนุนที่ไม่ได้เขียนโค้ด) ซึ่งอาจใช้ license tier ที่ถูกกว่าได้

**2. CI/CD Compute Minutes**

นี่มักเป็นต้นทุนที่**เติบโตเร็วที่สุดและควบคุมยากที่สุด** เพราะ:
- Pipeline ที่ไม่ได้ optimize (ไม่มี caching, รัน test ที่ไม่จำเป็นซ้ำ ๆ) กิน compute minutes มากเกินจำเป็น
- Self-hosted runner ช่วยประหยัดค่า compute minutes ของ SaaS runner แต่แลกกับต้นทุนการดูแล infrastructure เอง (ค่า server, ค่าดูแลระบบ, ความปลอดภัยของ runner เอง)
- Matrix build (รัน test พร้อมกันหลาย environment/version) ทำให้ minutes คูณขึ้นอย่างรวดเร็วถ้าไม่ระมัดระวัง

**ตัวอย่างการคำนวณอย่างง่าย:**

```
สมมติ: องค์กรมี 3,000 repository, เฉลี่ย repo ละ 20 pipeline run ต่อวัน
        แต่ละ run ใช้เวลาเฉลี่ย 8 นาที

รวม compute minutes ต่อวัน = 3,000 × 20 × 8 = 480,000 นาทีต่อวัน
รวมต่อเดือน (30 วัน)        = 14,400,000 นาที

หากอัตราค่าบริการ SaaS runner อยู่ที่ระดับเซนต์ต่อนาที (ขึ้นกับชนิด runner และผู้ให้บริการ)
ต้นทุนต่อเดือนอาจอยู่ในหลักแสนถึงหลักล้านบาทได้ง่าย ๆ
ถ้าไม่มีการ optimize pipeline หรือใช้ self-hosted runner สำหรับ workload ที่หนักและสม่ำเสมอ
```

ตัวเลขข้างต้นเป็นเพียงตัวอย่างสมมติเพื่อแสดงให้เห็นว่าต้นทุนสามารถขยายตัวเร็วเพียงใดเมื่อ scale ใหญ่ ราคาจริงขึ้นอยู่กับผู้ให้บริการ ประเภท runner (Linux/Windows/macOS, ขนาด CPU/RAM) และเงื่อนไขสัญญาที่องค์กรเจรจาไว้

**3. Storage (Git LFS, Artifact, Container Registry)**

- **Git LFS (Large File Storage)** — repository ที่เก็บไฟล์ binary ขนาดใหญ่ (asset ของเกม, model ของ machine learning, ไฟล์ media) มีต้นทุน storage และ bandwidth แยกต่างหากจากโค้ดปกติ
- **Artifact storage** — ผลลัพธ์จาก CI/CD (build output, test report, coverage report) ที่เก็บสะสมไว้นานเกินจำเป็นจะเพิ่มต้นทุน storage โดยไม่จำเป็น องค์กรควรมี retention policy ที่ชัดเจน (เช่น เก็บ artifact ไว้แค่ 30 วัน ยกเว้น release build ที่ต้องเก็บถาวร)
- **Container Registry** — เก็บ container image จำนวนมาก โดยเฉพาะถ้าไม่มีการลบ image เก่าที่ไม่ได้ใช้แล้ว (image ของทุก commit สะสมไปเรื่อย ๆ)

**4. Third-party Tool Integration**

ระบบนิเวศรอบ Git hosting มักมีเครื่องมือเสริมที่คิดค่าบริการแยกต่างหาก เช่น security scanning tool เฉพาะทาง (Snyk, Semgrep commercial tier), Developer Portal แบบ SaaS (Port, Cortex), DORA metrics platform (LinearB, Faros AI) ซึ่งแต่ละตัวมักคิดค่าบริการตาม seat หรือตามจำนวน repository ที่เชื่อมต่อ

### 829.2 กลยุทธ์การควบคุมต้นทุน

**1. Chargeback / Showback model**

องค์กรขนาดใหญ่มักใช้ระบบ **"Showback"** (แสดงต้นทุนที่แต่ละทีมสร้างขึ้น โดยไม่เรียกเก็บเงินจริง) หรือ **"Chargeback"** (หักงบประมาณของแต่ละทีมตามการใช้งานจริง) เพื่อสร้างแรงจูงใจให้ทีมย่อยใส่ใจต้นทุนของตัวเอง แทนที่จะมองว่า "เป็นค่าใช้จ่ายกลางที่ไม่เกี่ยวกับฉัน"

**2. Pipeline optimization เป็นนโยบายบังคับ**

ฝัง best practice ด้านการประหยัด compute เข้าไปใน golden path template (Step 826) โดยตรง เช่น บังคับใช้ dependency caching, จำกัดจำนวน parallel job สูงสุดต่อ repository, ตั้งค่า timeout ที่เหมาะสมเพื่อป้องกัน job ที่ค้าง (hang) กินเวลาไม่จำกัด

**3. License audit สม่ำเสมอ**

ตรวจสอบเป็นระยะว่ามี seat ที่จ่ายเงินแต่ไม่มีการใช้งานจริงหรือไม่ (เช่น พนักงานที่ลาออกไปแล้วแต่ SCIM deprovisioning ทำงานไม่สมบูรณ์ ทำให้ยังนับเป็น seat ที่เสียเงินอยู่ — นี่คือเหตุผลอีกข้อที่ SCIM ใน Step 823 สำคัญมาก ไม่ใช่แค่เรื่องความปลอดภัยแต่รวมถึงเรื่องต้นทุนด้วย)

**4. Right-sizing runner**

เลือกขนาด runner (CPU/RAM) ให้เหมาะสมกับ workload จริง ไม่ใช้ runner ขนาดใหญ่เกินจำเป็นสำหรับงานเบา ๆ และพิจารณาใช้ self-hosted runner (บน cloud provider ขององค์กรเองหรือ on-premise) สำหรับ workload ที่มีปริมาณสม่ำเสมอสูงมาก ซึ่งมักคุ้มค่ากว่าการจ่ายตามการใช้งานแบบ SaaS runner ในระยะยาว

**5. Storage lifecycle policy**

ตั้งค่า retention policy อัตโนมัติสำหรับ artifact, container image, และ package registry เพื่อลบข้อมูลเก่าที่ไม่จำเป็นโดยอัตโนมัติ แทนที่จะปล่อยให้สะสมไปเรื่อย ๆ

### 829.3 มุมมองที่สำคัญ: Cost vs. Developer Velocity

ข้อควรระวังสำคัญคือ **การประหยัดต้นทุนต้องไม่แลกมาด้วยความเร็วในการพัฒนาที่ลดลงอย่างไม่สมเหตุสมผล** — เช่น การจำกัด CI/CD compute จนวิศวกรต้องรอ pipeline นานผิดปกติ อาจประหยัดเงินค่า compute ได้จริง แต่ต้นทุนที่แท้จริง (เวลาของวิศวกรที่รอคอย ซึ่งมักแพงกว่าค่า compute มาก) อาจสูงกว่าที่ประหยัดได้มาก

แนวทางที่สมดุลคือการมองต้นทุนแบบ **Total Cost of Ownership (TCO)** ที่รวมทั้งต้นทุนเครื่องมือ (tooling cost) และต้นทุนแฝงด้านเวลาของวิศวกร (opportunity cost) เข้าด้วยกัน แล้วจึงตัดสินใจว่าจุดสมดุลที่เหมาะสมอยู่ตรงไหนสำหรับองค์กรของตัวเอง

---

## Step 830: แบบฝึกหัด — ออกแบบ Repository Governance Policy สำหรับองค์กร 500 คน

ถึงเวลาลงมือปฏิบัติจริง โดยนำทุกแนวคิดจาก Step 821–829 มาประยุกต์ใช้ในสถานการณ์สมมติที่ใกล้เคียงความเป็นจริงมากที่สุด

### 830.1 สถานการณ์สมมติ

คุณเป็น **Staff Engineer** ในทีม Platform Engineering ของบริษัท **"FinTechCo"** ซึ่งเป็นบริษัท FinTech ที่กำลังเติบโตอย่างรวดเร็ว

**ข้อมูลองค์กร:**
- พนักงานฝ่ายวิศวกรรมทั้งหมด: 500 คน แบ่งเป็น 45 ทีมย่อย
- Repository ปัจจุบัน: ประมาณ 800 ตัว (เพิ่มขึ้นเฉลี่ยเดือนละ 15-20 ตัว)
- ใช้ GitHub Enterprise Cloud เป็น Git hosting platform หลัก
- ต้องปฏิบัติตาม **PCI-DSS** (เนื่องจากประมวลผลข้อมูลบัตรเครดิต) และ **SOC 2 Type II**
- มีทีมที่ทำงานกับข้อมูลลูกค้าโดยตรง (Payments, Identity Verification) และทีมที่ไม่แตะข้อมูล sensitive (Internal Tools, Marketing Website)
- เพิ่งเกิดเหตุการณ์ที่พนักงานลาออกแล้วยังพบว่ามีสิทธิ์เข้าถึง repository อยู่ 3 สัปดาห์หลังวันสุดท้ายที่ทำงาน (ตรวจพบระหว่างการเตรียม SOC 2 audit)
- CEO และ CTO ต้องการเห็นแผน governance ที่ชัดเจนภายใน 2 สัปดาห์ เพื่อนำเสนอต่อ Board of Directors ก่อนรอบ audit ถัดไป

### 830.2 โจทย์: จัดทำเอกสาร Repository Governance Policy

จัดทำเอกสารที่ครอบคลุมหัวข้อต่อไปนี้ให้ครบถ้วน โดยอ้างอิงแนวคิดที่เรียนมาทั้งหมดใน Part นี้

**ส่วนที่ 1: Access Control Model**

ให้ระบุ:
1. องค์กรควรเชื่อมต่อ GitHub Enterprise กับ Identity Provider ตัวไหน (เลือกจาก Okta/Azure AD/อื่น ๆ) และเหตุผล
2. ควรบังคับ SAML SSO แบบ enforced หรือไม่ เพราะเหตุใด
3. ออกแบบ SCIM provisioning flow ที่แก้ปัญหา "พนักงานลาออกแล้วยังมีสิทธิ์อยู่ 3 สัปดาห์" ที่เกิดขึ้นจริง — ระบุว่าจุดล้มเหลว (failure point) ที่เป็นไปได้ในกระบวนการเดิมคืออะไร และ SCIM แก้ตรงจุดไหน
4. กำหนด permission model แบบ role-based สำหรับ 3 กลุ่ม: (a) วิศวกร full-time, (b) contractor ภายนอก, (c) ทีมที่แตะข้อมูล PCI-DSS scope โดยเฉพาะ

**ส่วนที่ 2: Repository Ruleset Design**

ออกแบบ Organization-level Ruleset อย่างน้อย 3 ระดับ:
1. **Baseline ruleset** ที่บังคับใช้กับทุก repository ไม่มีข้อยกเว้น (ห้าม force-push, ต้องมี branch protection, ฯลฯ)
2. **Elevated ruleset** สำหรับ repository ที่อยู่ใน PCI-DSS scope (เช่น ต้องมี 2 approver, ต้อง sign commit, ต้องผ่าน security scan ที่เข้มกว่าปกติ)
3. **Bypass policy** — ใครมีสิทธิ์ bypass กฎได้ในกรณีฉุกเฉิน และต้องมีการบันทึกอย่างไร

ระบุด้วยว่าจะใช้กลไกอะไรในการ "targeting" ว่า repo ไหนอยู่ใน PCI-DSS scope (เช่น custom property, topic label, naming convention)

**ส่วนที่ 3: CI/CD Standard**

1. ออกแบบโครงสร้าง Golden Path Template อย่างน้อย 2 แบบที่เหมาะกับ FinTechCo (เช่น "PCI-scope microservice" กับ "internal tool")
2. ระบุ mandatory security gate ที่ต้องฝังในทุก template (อ้างอิงจาก compliance requirement ของ PCI-DSS/SOC 2)
3. กำหนดนโยบาย versioning ของ template และแผน migration เมื่อต้องอัปเดตแบบ breaking change

**ส่วนที่ 4: Ownership และ Governance Process**

1. ออกแบบโครงสร้าง ownership 3 ชั้น (file/repository/domain) ตามที่เรียนใน Step 822 ให้เหมาะกับ 45 ทีมของ FinTechCo
2. เสนอกระบวนการตรวจจับ orphaned repository และ SLA ในการแก้ไข
3. เสนอว่าจะเริ่มโครงการ InnerSource หรือไม่ในสถานการณ์นี้ พร้อมเหตุผล (คำใบ้: พิจารณาว่า FinTechCo มี regulatory constraint ที่อาจจำกัดขอบเขตของ InnerSource ในบาง domain)

**ส่วนที่ 5: Metrics และการรายงานต่อผู้บริหาร**

1. ระบุว่าจะเริ่มเก็บ DORA Metrics ตัวไหนก่อนเป็นลำดับแรก และเพราะเหตุใด (ยังไม่ต้องเจาะลึกวิธีคำนวณ เพราะจะเรียนเต็มรูปแบบใน Part 92)
2. ออกแบบ dashboard แนวคิดคร่าว ๆ ที่จะนำเสนอต่อ CEO/CTO/Board เพื่อแสดงสถานะ compliance ของ repository governance (เช่น "% ของ repo ที่ผ่าน baseline ruleset ครบถ้วน")

### 830.3 แนวทางเฉลยโดยสรุป (แนวคิดหลัก ไม่ใช่คำตอบสำเร็จรูปเดียว)

โจทย์นี้ไม่มีคำตอบตายตัวเพียงหนึ่งเดียว แต่แนวทางที่สอดคล้องกับหลักการที่ถูกต้องควรมีองค์ประกอบประมาณนี้:

**Access Control:** เลือก IdP ที่องค์กรมีอยู่แล้วหรือเป็นมาตรฐานอุตสาหกรรม (Okta หรือ Azure AD เป็นตัวเลือกยอดนิยมทั้งคู่) บังคับ SAML SSO แบบ enforced เนื่องจาก FinTechCo อยู่ภายใต้ PCI-DSS ซึ่งกำหนดให้มีการควบคุมการเข้าถึงที่เข้มงวด จุดล้มเหลวของกระบวนการเดิมคือการ offboarding ที่พึ่งพา manual process (HR แจ้ง IT แล้ว IT ไปลบสิทธิ์ทีละระบบ ซึ่งมีโอกาสตกหล่นสูง) การแก้ไขคือเชื่อม HR system เข้ากับ IdP โดยตรง แล้วให้ SCIM sync สถานะแบบ real-time หรืออย่างน้อยเป็นรอบที่ถี่มาก (เช่น ทุกชั่วโมง) ไปยัง GitHub Enterprise เพื่อให้การ deactivate เกิดขึ้นในนาทีเดียวกับที่ HR ปรับสถานะ ไม่ใช่รอหลายสัปดาห์

**Ruleset:** Baseline ควรครอบคลุมทุก repo ด้วย wildcard `*` ส่วน Elevated ruleset ควร target ผ่าน custom property เช่น `compliance-scope: pci-dss` ที่ทีมต้องประกาศตอนสร้าง repo (และทีม security ตรวจสอบเป็นระยะว่าประกาศถูกต้อง) Bypass ควรจำกัดเฉพาะ Incident Response team และทุกการ bypass ต้องสร้าง audit log อัตโนมัติที่ส่งแจ้งเตือนไปยังทีม security ทันที

**CI/CD:** PCI-scope template ต้องมี mandatory step อย่างน้อย secret scanning, SAST, dependency vulnerability scan, SBOM generation, และ signed commit verification ส่วน internal tool template ใช้ baseline security gate ที่เบากว่าได้ Versioning ควรใช้ semantic versioning พร้อมกำหนด deprecation window อย่างน้อย 60-90 วันก่อน sunset เวอร์ชันเก่า

**Ownership:** ใช้โครงสร้าง Domain (เช่น Payments, Identity, Internal Platform, Customer-facing Product) ครอบทีมย่อยหลายทีม แต่ละ domain มี owner ระดับผู้บริหาร (Director/VP) แต่ละ repository ต้องมี `catalog-info.yaml` พร้อม CODEOWNERS ที่ CI ตรวจสอบความถูกต้องทุกครั้งที่มีการแก้ไข ส่วน InnerSource ควรเริ่มในโดเมนที่ไม่ติด PCI-DSS scope ก่อน (เช่น Internal Tools, Developer Platform เอง) เพื่อสร้างวัฒนธรรมและเรียนรู้ก่อนขยายไปยัง domain ที่มี regulatory constraint สูงกว่า ซึ่งอาจต้องมีการควบคุมการเข้าถึงที่เข้มงวดกว่าปกติของ InnerSource ทั่วไป

**Metrics:** ควรเริ่มจาก **Change Failure Rate** และ **Deployment Frequency** ก่อน เพราะข้อมูลทั้งสองนี้ดึงได้ตรงจาก CI/CD pipeline log ที่มีอยู่แล้วโดยไม่ต้องสร้างเครื่องมือใหม่ทั้งหมด และตอบคำถามด้าน compliance ได้ตรงจุด (เช่น การแสดงว่า deployment มีความเสี่ยงต่ำลงหลังจากมี governance ที่ดีขึ้น) ส่วน dashboard ต่อผู้บริหารควรเน้นภาพรวมง่าย ๆ เช่น เปอร์เซ็นต์ compliance, แนวโน้มตามเวลา (trend line) ไม่ใช่รายละเอียดทางเทคนิคเชิงลึก

### 830.4 การขยายผล

หลังจากทำแบบฝึกหัดนี้เสร็จ ให้ลองพิจารณาต่อว่า:

- แผนงานนี้ควรทยอย roll out อย่างไร (เริ่มจาก domain ไหนก่อน ทำไมถึงเลือกลำดับนั้น)
- ถ้ามีทีมที่ต่อต้านการเปลี่ยนแปลง (resistance to change) เพราะรู้สึกว่า governance ทำให้ทำงานช้าลง จะสื่อสารและจัดการอย่างไรให้ได้ทั้งความปลอดภัยและการยอมรับจากทีม
- Timeline ที่สมเหตุสมผลสำหรับการ implement ทั้งหมดนี้คือเท่าไหร่ (สัปดาห์/เดือน) โดยพิจารณาว่าองค์กรต้องผ่าน audit รอบถัดไปด้วย

การฝึกคิดแบบนี้คือทักษะที่แยก **Senior Engineer** ที่เก่งเรื่องเทคนิคออกจาก **Staff/Principal Engineer** ที่ต้องมองภาพองค์กรทั้งระบบ เข้าใจ trade-off ระหว่างความปลอดภัย ความคล่องตัว และต้นทุน แล้วออกแบบนโยบายที่ใช้งานได้จริงในบริบทที่ซับซ้อนของ enterprise

---

## สรุป Part 83

ใน Part นี้เราได้เรียนรู้ว่า:

1. องค์กรระดับ enterprise เผชิญปัญหาที่ **เปลี่ยนธรรมชาติไปโดยสิ้นเชิง** เมื่อเทียบกับทีมขนาดเล็ก — ไม่ใช่แค่ "ปัญหาเดิมที่ใหญ่ขึ้น" แต่ต้องการแนวคิดและเครื่องมือคนละแบบ โดยมี Conway's Law เป็นหลักคิดพื้นฐานที่อธิบายความสัมพันธ์ระหว่างโครงสร้างองค์กรกับโครงสร้างซอฟต์แวร์
2. Code Ownership ต้องขยายจาก CODEOWNERS ระดับไฟล์ (Part 36) ไปสู่โครงสร้าง 3 ชั้น (file/repository/domain) เพื่อรองรับหลายร้อยทีม พร้อมกระบวนการตรวจจับ orphaned repository อัตโนมัติ
3. **SSO, SAML, SCIM** ทำงานร่วมกันเพื่อจัดการ user lifecycle อัตโนมัติ — SAML/SSO จัดการการยืนยันตัวตน ส่วน SCIM จัดการ provisioning/deprovisioning ที่ป้องกันความเสี่ยงด้านความปลอดภัยที่ใหญ่ที่สุดขององค์กรใหญ่ คือ ex-employee ที่ยังมี access ค้างอยู่
4. **Organization-level Ruleset** (GitHub) และ **Compliance Framework** (GitLab) ทำให้บังคับใช้นโยบายความปลอดภัยกลางกับ repository นับพันตัวพร้อมกันได้ โดยไม่ต้องพึ่งพาการตั้งค่า manual ทีละ repo
5. **InnerSource** นำวัฒนธรรม Open Source มาใช้ภายในองค์กร แก้ปัญหาการเขียนโค้ดซ้ำซ้อนและ knowledge silo ผ่าน visibility ที่เปิดกว้าง, Trusted Committer model, และเอกสารมาตรฐาน
6. **Standardized CI/CD Templates** ต่อยอดจาก Reusable Workflows (Part 70) และ CI/CD Components (Part 72) สู่แนวคิด Golden Path/Paved Road ที่ทำให้ "ทางที่ถูกต้องเป็นทางที่ง่ายที่สุด"
7. **Developer Portal / IDP** อย่าง Backstage รวมข้อมูล Software Catalog, Scaffolder และ TechDocs ไว้ในที่เดียว แก้ปัญหา discoverability ที่ทวีความรุนแรงขึ้นตามจำนวน repository
8. **DORA Metrics** (Deployment Frequency, Lead Time for Changes, Change Failure Rate, Time to Restore Service) คือมาตรฐานการวัดประสิทธิภาพ engineering organization ที่ได้รับการยอมรับกว้างขวางที่สุด — จะเจาะลึกเต็มรูปแบบใน Part 92
9. ต้นทุนของ Git hosting ระดับ enterprise ประกอบด้วย seat license, CI/CD compute minutes, storage และ third-party tool integration ซึ่งต้องบริหารด้วย showback/chargeback model และมองในกรอบ Total Cost of Ownership ที่รวมต้นทุนเวลาของวิศวกรด้วย ไม่ใช่แค่ค่าเครื่องมือ
10. แบบฝึกหัดออกแบบ Repository Governance Policy สำหรับองค์กร 500 คน ช่วยฝึกทักษะการมองภาพรวมและจัดการ trade-off ระหว่างความปลอดภัย ความคล่องตัว และต้นทุน ซึ่งเป็นทักษะสำคัญของ Staff/Principal Engineer

### Checklist ก่อนไป Part 84

ก่อนไปต่อ Part 84 ให้ตรวจสอบว่าคุณ:

- [ ] เข้าใจว่าทำไมปัญหาระดับ enterprise ถึงต่างจากปัญหาของทีมเล็กในเชิงคุณภาพ ไม่ใช่แค่เชิงปริมาณ
- [ ] อธิบายความแตกต่างและความสัมพันธ์ระหว่าง SSO, SAML, และ SCIM ได้
- [ ] เข้าใจว่า Organization-level Ruleset แก้ปัญหาอะไรที่ Branch Protection ระดับ repo เดียวแก้ไม่ได้
- [ ] เข้าใจแนวคิด InnerSource และองค์ประกอบสำคัญ (visibility, Trusted Committer, เอกสารมาตรฐาน)
- [ ] เข้าใจแนวคิด Golden Path/Paved Road และเชื่อมโยงกับ Reusable Workflows/CI/CD Components ที่เรียนมาก่อนหน้าได้
- [ ] รู้จัก Backstage และองค์ประกอบหลัก (Software Catalog, Scaffolder, TechDocs)
- [ ] จำ 4 ตัวชี้วัดของ DORA Metrics ได้ (Deployment Frequency, Lead Time for Changes, Change Failure Rate, Time to Restore Service)
- [ ] เข้าใจโครงสร้างต้นทุนหลักของ Git hosting ระดับ enterprise 4 ด้าน (seat, compute, storage, third-party tool)
- [ ] ลองทำแบบฝึกหัดออกแบบ Governance Policy ด้วยตัวเองอย่างน้อยในหัวข้อ Access Control และ Ruleset Design

**ต่อไป:** [Part 84: Disaster Recovery และ Backup Strategy สำหรับ Git](./part-084-disaster-recovery-backup.md)
