# Part 85: โปรเจกต์ฝึกหัด: วาง Security Policy ให้องค์กร

> **Step ในหลักสูตรนี้:** Step 841–850
> **เฟส:** 8 — DevOps / Security / Compliance (Part นี้คือ Part ปิดท้ายของเฟส 8 ทั้งหมด)
> **เป้าหมายของ Part นี้:** นำทุกสิ่งที่เรียนมาตลอดเฟส 8 ตั้งแต่ Part 76 ถึง Part 84 มาประกอบร่างเป็นเอกสารที่ใช้งานได้จริงเพียงชุดเดียว — GitOps, Infrastructure as Code, Secret Management, Signed Commits, Dependency Scanning/Supply Chain Security, Compliance/Audit Trail, Monorepo/Polyrepo, การจัดการ Repository ระดับ Enterprise และ Disaster Recovery — โดยจำลององค์กรซอฟต์แวร์ขนาดกลางแห่งหนึ่งที่ต้องวาง **Security Policy ฉบับเต็ม** ให้ทุกทีมปฏิบัติตาม ตั้งแต่การเขียน `SECURITY.md` การกำหนดกฎ branch protection/CODEOWNERS ทั้งองค์กร ไปจนถึงการรวมทุกนโยบายเป็นเอกสารเดียวที่นำไปใช้งานจริงได้ทันที ก่อนจะข้ามไปเรียนรู้ทักษะการเป็นมืออาชีพในเฟส 9

---

## สารบัญของ Part นี้

- Step 841: ภาพรวมภารกิจปิดเฟส 8 — วาง Security Policy ฉบับเต็มให้องค์กรจำลอง
- Step 842: เขียน `SECURITY.md` — นโยบายการรายงานช่องโหว่ (Vulnerability Disclosure Policy)
- Step 843: กำหนดนโยบาย Branch Protection + CODEOWNERS มาตรฐานทั้งองค์กร
- Step 844: กำหนดนโยบาย Secret Management ทั้งองค์กร
- Step 845: กำหนดนโยบาย Signed Commits สำหรับ Repository ที่สำคัญ
- Step 846: กำหนดนโยบาย Dependency Scanning + SBOM สำหรับทุกโปรเจกต์
- Step 847: กำหนดนโยบาย Backup / Disaster Recovery
- Step 848: กำหนดนโยบาย Compliance / Audit Trail
- Step 849: จัดทำเอกสาร Security Policy ฉบับเต็มรวมทุกหัวข้อเป็นไฟล์เดียว
- Step 850: สรุปทบทวนภาพรวมเฟส 8 ทั้งหมด (Part 76–85) พร้อม Cheat Sheet และ Checklist ก่อนเข้าเฟส 9

---

## Step 841: ภาพรวมภารกิจปิดเฟส 8 — วาง Security Policy ฉบับเต็มให้องค์กรจำลอง

ตลอด 9 Part ที่ผ่านมาในเฟส 8 (Part 76–84) คุณได้เรียนรู้เครื่องมือและแนวคิดด้านความปลอดภัยและการกำกับดูแล (governance) ของ Git แยกเป็นชิ้น ๆ มาครบแล้ว: แนวคิด GitOps, การจัดการ Infrastructure as Code ร่วมกับ Git, Secret Management และการป้องกันข้อมูลรั่วไหล, Signed Commits ด้วย GPG/SSH, Dependency Scanning และ Supply Chain Security, Compliance และ Audit Trail, การตัดสินใจ Monorepo vs Polyrepo, การจัดการ Repository ขนาดใหญ่ระดับ Enterprise และ Disaster Recovery/Backup Strategy

Part นี้จะ **ไม่สอนเครื่องมือใหม่แม้แต่ตัวเดียว** เช่นเดียวกับที่ Part 45 ปิดเฟส 4 — แต่จะเอาทุกอย่างที่เรียนมาทั้งหมดมาร้อยเรียงเข้าด้วยกันในภารกิจเดียวที่ทุกบริษัทซอฟต์แวร์ต้องทำจริงไม่ช้าก็เร็ว: **การวาง Security Policy ฉบับเต็มให้กับองค์กร**

### 841.1 ทำความรู้จักองค์กรจำลองของเรา

องค์กรที่เราจะจำลองชื่อ **NetiTech Solutions** บริษัทพัฒนาซอฟต์แวร์ขนาดกลาง (ประมาณ 120 คน) ที่มีทีมวิศวกรรม 6 ทีมทำงานบน GitHub Enterprise ร่วมกัน แบ่งเป็น:

| ทีม | ดูแล Repository | ระดับความสำคัญ |
|---|---|---|
| Platform / Infra | `infra-terraform`, `k8s-manifests` (IaC + GitOps) | วิกฤต (Critical) |
| Payments | `payments-service`, `payments-sdk` | วิกฤต (Critical) |
| Core Product | `core-api`, `web-app` | สูง (High) |
| Data Platform | `data-pipelines`, `analytics-warehouse` | สูง (High) |
| Internal Tools | `internal-dashboard`, `devtools-cli` | ปานกลาง (Medium) |
| Marketing Site | `marketing-site` | ต่ำ (Low) |

ทีม Security & Compliance (นำโดย **หัวหน้าฝ่ายความปลอดภัย — CISO จำลอง**) ได้รับมอบหมายให้ร่างและบังคับใช้ **Security Policy** ที่ครอบคลุมทุก repository ในองค์กร โดยต้องตอบโจทย์ 3 กลุ่มผู้เกี่ยวข้อง:

1. **นักพัฒนา (Developers)** — ต้องรู้ว่าอะไรทำได้/ทำไม่ได้ และเครื่องมืออะไรที่ระบบจะบังคับใช้อัตโนมัติ
2. **ผู้ดูแลระบบ/DevOps** — ต้องรู้ว่าต้องตั้งค่าอะไรในระดับ organization และ repository
3. **ผู้ตรวจสอบ/ผู้บริหาร (Auditors/Management)** — ต้องมีเอกสารที่พิสูจน์ได้ว่าองค์กรปฏิบัติตามมาตรฐานความปลอดภัยจริง เมื่อถูกตรวจสอบ (audit) หรือขอใบรับรอง (เช่น SOC 2, ISO/IEC 27001)

### 841.2 โครงสร้างเอกสารที่เราจะสร้างในโปรเจกต์นี้

แทนที่จะเขียนนโยบายเป็นข้อความยาว ๆ ในไฟล์เดียวตั้งแต่ต้น เราจะใช้วิธีที่องค์กรจริงส่วนใหญ่ใช้: **แยกนโยบายย่อยตามหัวข้อก่อน แล้วค่อยรวมเป็นเอกสารกลางในตอนท้าย** เพื่อให้แต่ละทีมย่อย (เช่นทีม Infra, ทีม Compliance) ดูแลเฉพาะส่วนของตัวเองได้ง่าย โครงสร้างไฟล์ทั้งหมดที่เราจะสร้างในโฟลเดอร์ `security-policy/` มีดังนี้:

```
netitech-security-policy/
├── SECURITY.md                          ← Step 842: vulnerability disclosure
├── policies/
│   ├── 01-branch-protection.md          ← Step 843
│   ├── 02-secret-management.md          ← Step 844
│   ├── 03-signed-commits.md             ← Step 845
│   ├── 04-dependency-scanning.md        ← Step 846
│   ├── 05-backup-disaster-recovery.md   ← Step 847
│   └── 06-compliance-audit.md           ← Step 848
├── templates/
│   ├── CODEOWNERS.template
│   └── pull_request_template.md
└── SECURITY_POLICY_FULL.md              ← Step 849: เอกสารรวมฉบับสมบูรณ์
```

แนวทางนี้สอดคล้องกับหลักการที่เรียนมาตลอดเฟส 8: **นโยบายความปลอดภัยที่ดีต้องอยู่ใน Git เอง** (ตามแนวคิด "Policy as Code" ที่ต่อยอดจาก GitOps และ IaC ใน Part 76–77) เพราะทำให้นโยบายมีประวัติการแก้ไข (`git log`), มีการรีวิวก่อนเปลี่ยนแปลง (Pull Request), และตรวจสอบย้อนหลังได้เสมอ — ต่างจากนโยบายที่เขียนไว้ในเอกสาร Word หรือ Google Docs ที่ไม่มีใครรู้ว่าใครแก้อะไรไปเมื่อไหร่

### 841.3 เตรียม Repository สำหรับ Policy

```bash
mkdir -p ~/git-course/netitech-security-policy/policies
mkdir -p ~/git-course/netitech-security-policy/templates
cd ~/git-course/netitech-security-policy
git init
git config user.name "Security Team"
git config user.email "security-team@netitech.example"
```

```
Initialized empty Git repository in /home/user/git-course/netitech-security-policy/.git/
```

### 841.4 แผนงานทั้งหมดของ Part นี้ในสายตาเดียว

| Step | เอกสารที่จะสร้าง | อ้างอิงความรู้จาก Part |
|---|---|---|
| 842 | `SECURITY.md` | หลักการ Responsible Disclosure สากล (GitHub Security Advisories) |
| 843 | `policies/01-branch-protection.md` + `templates/CODEOWNERS.template` | Branch Protection (Part 33–35), CODEOWNERS (Part 36) |
| 844 | `policies/02-secret-management.md` | Secret Management (Part 78) |
| 845 | `policies/03-signed-commits.md` | Signed Commits/GPG/SSH Signing (Part 79) |
| 846 | `policies/04-dependency-scanning.md` | Dependency Scanning/Supply Chain Security (Part 80) |
| 847 | `policies/05-backup-disaster-recovery.md` | Disaster Recovery/Backup Strategy (Part 84) |
| 848 | `policies/06-compliance-audit.md` | Compliance/Audit Trail (Part 81) |
| 849 | `SECURITY_POLICY_FULL.md` | ทุก Part ข้างต้นรวมกัน |
| 850 | Cheat Sheet + Checklist | ทุก Part ในเฟส 8 (76–85) |

> **หมายเหตุ:** แม้หัวข้อ GitOps/IaC (Part 76–77), Monorepo vs Polyrepo (Part 82) และการจัดการ Repository ระดับ Enterprise (Part 83) จะไม่มีไฟล์นโยบายแยกของตัวเองใน Step นี้ (เพราะเป็นแนวคิดเชิงสถาปัตยกรรมมากกว่านโยบายที่บังคับใช้แบบกฎตายตัว) แต่หลักการของทั้งสามเรื่องนี้จะถูกอ้างอิงแทรกอยู่ในนโยบายอื่น ๆ ตลอด Part นี้ (เช่น การกำหนด CODEOWNERS ในโครงสร้าง Monorepo ที่ Step 843 และการเลือก repository ที่ต้องมี branch protection เข้มงวดตามระดับความสำคัญที่ได้จากการจัดกลุ่มแบบ Enterprise-scale) และจะถูกสรุปรวมอีกครั้งในตาราง Cheat Sheet ของ Step 850 อย่างครบถ้วน

จากนี้ไป เราจะเดินหน้าสวมบทบาทเป็นทีม Security & Compliance ของ NetiTech Solutions ทีละ Step ตามตารางด้านบน

---

## Step 842: เขียน `SECURITY.md` — นโยบายการรายงานช่องโหว่ (Vulnerability Disclosure Policy)

### 842.1 ทำไมทุก Repository ต้องมี `SECURITY.md`

GitHub รองรับไฟล์ชื่อ `SECURITY.md` (วางไว้ที่ root, ที่โฟลเดอร์ `.github/`, หรือที่โฟลเดอร์ `docs/` ก็ได้) เป็นมาตรฐาน — เมื่อมีไฟล์นี้อยู่ GitHub จะแสดงลิงก์ "Security policy" ไว้ในแท็บ **Security** ของ repository โดยอัตโนมัติ และเมื่อมีคนกดปุ่ม "Report a vulnerability" ระบบจะพาไปสร้าง **Private Security Advisory** ที่สื่อสารกับผู้ดูแล repository ได้โดยไม่เปิดเผยช่องโหว่ต่อสาธารณะก่อนเวลาอันควร

หลักการสำคัญที่สุดของนโยบายนี้คือ **Coordinated/Responsible Disclosure**: ผู้ที่พบช่องโหว่ควรมีช่องทางรายงานที่ปลอดภัยและเป็นทางการ แทนที่จะเปิดเป็น public issue ทันที (ซึ่งจะทำให้ผู้ไม่หวังดีรู้ช่องโหว่ก่อนที่ทีมจะแก้ไขทัน) และองค์กรต้องตอบสนองภายในกรอบเวลาที่ชัดเจน ไม่ปล่อยให้เงียบหาย

### 842.2 องค์ประกอบหลักของ `SECURITY.md` ที่ดี

1. **Supported Versions** — บอกว่าเวอร์ชันไหนของซอฟต์แวร์ยังได้รับการแพตช์ความปลอดภัยอยู่
2. **ช่องทางการรายงาน** — ควรมีช่องทางที่ **ไม่ใช่ public GitHub Issue** เช่น GitHub Security Advisory หรืออีเมลเฉพาะทาง
3. **สิ่งที่ควรอยู่ในรายงาน** — เพื่อให้ทีมตรวจสอบได้เร็วโดยไม่ต้องถามซ้ำ
4. **กรอบเวลาตอบสนอง (SLA)** — ระบุให้ชัดว่าใช้เวลากี่วันในการตอบรับเบื้องต้น และกี่วันในการแก้ไข ตามระดับความรุนแรง
5. **Safe Harbor** — คำมั่นว่าองค์กรจะไม่ดำเนินคดีกับผู้ที่รายงานช่องโหว่โดยสุจริตใจและปฏิบัติตามนโยบายนี้
6. **สิ่งที่ไม่ควรทำ** — เช่น ห้ามทดสอบกับข้อมูลลูกค้าจริง ห้ามเปิดเผยช่องโหว่ต่อสาธารณะก่อนได้รับอนุญาต

### 842.3 เขียน `SECURITY.md` ของ NetiTech Solutions

```bash
cd ~/git-course/netitech-security-policy

cat > SECURITY.md << 'EOF'
# Security Policy — NetiTech Solutions

องค์กร NetiTech Solutions ให้ความสำคัญกับความปลอดภัยของซอฟต์แวร์และข้อมูลผู้ใช้งานเป็นอันดับแรก
เอกสารนี้อธิบายว่าเวอร์ชันใดยังได้รับการดูแลด้านความปลอดภัย และควรรายงานช่องโหว่ที่พบอย่างไร

## เวอร์ชันที่ยังได้รับการสนับสนุนด้านความปลอดภัย (Supported Versions)

| เวอร์ชัน | ได้รับแพตช์ความปลอดภัย |
| ------- | ---------------------- |
| 5.x     | ✅ ใช่ |
| 4.x     | ✅ ใช่ (จนถึงสิ้นสุดระยะ Extended Support) |
| 3.x     | ❌ ไม่ (End of Life) |
| < 3.0   | ❌ ไม่ (End of Life) |

## การรายงานช่องโหว่ (Reporting a Vulnerability)

**กรุณาอย่ารายงานช่องโหว่ด้านความปลอดภัยผ่าน public GitHub Issue, Discussion หรือ Pull Request**
เนื่องจากช่องทางเหล่านี้เปิดเผยต่อสาธารณะทันที และอาจทำให้ผู้ไม่หวังดีรู้ช่องโหว่ก่อนที่ทีมจะแก้ไขทัน

โปรดรายงานผ่านช่องทางใดช่องทางหนึ่งต่อไปนี้แทน:

1. **GitHub Security Advisory (แนะนำ):** ไปที่แท็บ "Security" ของ repository ที่เกี่ยวข้อง
   แล้วกด "Report a vulnerability" เพื่อเปิด Private Security Advisory โดยตรงกับทีมดูแล
2. **อีเมล:** security@netitech.example (รองรับการเข้ารหัสด้วย PGP กุญแจสาธารณะเผยแพร่ที่
   https://netitech.example/.well-known/security.txt)

### ข้อมูลที่ควรใส่ในรายงาน

เพื่อให้ทีมตรวจสอบและยืนยันปัญหาได้รวดเร็วที่สุด กรุณาระบุ:

- ประเภทของช่องโหว่ (เช่น SQL Injection, Broken Access Control, Secret Leak)
- Repository, branch และ commit hash ที่พบปัญหา (ถ้าทราบ)
- ขั้นตอนการทำซ้ำ (Proof of Concept) แบบละเอียด
- ผลกระทบที่อาจเกิดขึ้น และเงื่อนไขที่จำเป็นต่อการโจมตี
- (ถ้ามี) ข้อเสนอแนะแนวทางแก้ไข

### กรอบเวลาตอบสนองของทีม (Response SLA)

| ขั้นตอน | ระยะเวลา |
| ------- | -------- |
| ตอบรับว่าได้รับรายงานแล้ว (Acknowledgement) | ภายใน 2 วันทำการ |
| ยืนยันความถูกต้องของช่องโหว่ (Triage) | ภายใน 5 วันทำการ |
| ออกแพตช์แก้ไขสำหรับช่องโหว่ระดับ Critical/High | ภายใน 14 วัน นับจากยืนยัน |
| ออกแพตช์แก้ไขสำหรับช่องโหว่ระดับ Medium/Low | ภายใน 60 วัน นับจากยืนยัน |
| เผยแพร่ Security Advisory ต่อสาธารณะ | หลังแพตช์ออกแล้วอย่างน้อย 7 วัน หรือ ตามตกลงร่วมกับผู้รายงาน |

ระดับความรุนแรงประเมินตามมาตรฐาน **CVSS v3.1** เป็นหลัก

## Safe Harbor

NetiTech Solutions ถือว่าการวิจัยด้านความปลอดภัยที่ทำโดยสุจริตใจและปฏิบัติตามนโยบายนี้
เป็นกิจกรรมที่ได้รับอนุญาต เราจะไม่ดำเนินคดีทางกฎหมายกับผู้รายงานที่:

- ไม่เข้าถึง แก้ไข หรือทำลายข้อมูลของผู้ใช้งานจริงเกินความจำเป็นต่อการพิสูจน์ช่องโหว่
- ไม่ทำการโจมตีที่ส่งผลกระทบต่อความพร้อมใช้งานของระบบจริง (เช่น Denial of Service)
- รายงานผ่านช่องทางที่กำหนดไว้ข้างต้นเท่านั้น และให้เวลาทีมแก้ไขก่อนเปิดเผยต่อสาธารณะ

## ขอบเขต (Scope)

นโยบายนี้ครอบคลุม repository ทั้งหมดภายใต้องค์กร `netitech-solutions` บน GitHub Enterprise
รวมถึงบริการที่ให้บริการภายใต้โดเมน `*.netitech.example`
ไม่รวมถึงเว็บไซต์หรือบริการของบุคคลที่สาม (third-party) ที่เพียงเชื่อมต่อกับระบบของเรา
EOF
```

```bash
git add SECURITY.md
git commit -m "เพิ่ม SECURITY.md: นโยบายการรายงานช่องโหว่ตามมาตรฐาน GitHub Security Advisory"
```

```
[main (root-commit) a1a1a1a] เพิ่ม SECURITY.md: นโยบายการรายงานช่องโหว่ตามมาตรฐาน GitHub Security Advisory
 1 file changed, 60 insertions(+)
```

> **จุดสำคัญที่ต้องจำ:** ไฟล์ `SECURITY.md` นี้ต้อง **นำไปวางไว้ที่ root ของทุก repository สำคัญในองค์กร** ไม่ใช่แค่ไว้ใน repository นโยบายกลางเพียงที่เดียว วิธีที่ทำได้จริงในองค์กรขนาดใหญ่คือใช้ **repository กลางที่เก็บไฟล์มาตรฐาน (`.github` repository พิเศษของ GitHub Organization)** ซึ่ง GitHub จะดึงไฟล์ `SECURITY.md`, `CODE_OF_CONDUCT.md`, และ issue/PR template จากที่นั่นมาใช้เป็นค่าเริ่มต้นให้ทุก repository ที่ไม่มีไฟล์ของตัวเองโดยอัตโนมัติ ทีม Security ของ NetiTech จึงวางไฟล์นี้ไว้ทั้งใน repository กลางนี้ (`.github`) และเน้นย้ำให้ repository ที่มีเงื่อนไขพิเศษ (เช่น `payments-service` ที่ต้องการ SLA เข้มงวดกว่าค่ามาตรฐาน) มีไฟล์เฉพาะของตัวเองทับค่ากลางไว้

---

## Step 843: กำหนดนโยบาย Branch Protection + CODEOWNERS มาตรฐานทั้งองค์กร

Step นี้นำความรู้จาก **Part 33–35 (Branch Protection Rules)** และ **Part 36 (CODEOWNERS)** ในเฟส 4 มาประกอบเป็นนโยบายที่บังคับใช้ **ทั้งองค์กร** ไม่ใช่แค่โปรเจกต์เดียวเหมือนที่เคยฝึกใน Part 45

### 843.1 หลักการ: กำหนดระดับการป้องกันตามระดับความสำคัญของ Repository

องค์กรขนาดใหญ่ไม่ควรบังคับกฎเดียวกันทุก repository แบบตายตัว เพราะ `marketing-site` กับ `payments-service` มีความเสี่ยงต่างกันมหาศาล ทีม Security ของ NetiTech จึงกำหนด **ระดับการป้องกัน (Protection Tier)** 3 ระดับ โดยอิงจากตารางระดับความสำคัญที่กำหนดไว้ใน Step 841.1:

```bash
cat > policies/01-branch-protection.md << 'EOF'
# นโยบาย Branch Protection และ CODEOWNERS

## ระดับการป้องกัน (Protection Tier)

| Tier | Repository ตัวอย่าง | เงื่อนไข branch protection บน `main`/`release/*` |
| ---- | ------------------- | -------------------------------------------------- |
| **Tier 1 — Critical** | `payments-service`, `payments-sdk`, `infra-terraform`, `k8s-manifests` | ทุกข้อในตารางด้านล่าง + require signed commits + require 2 approvals รวม 1 approval จาก Security team |
| **Tier 2 — High** | `core-api`, `web-app`, `data-pipelines`, `analytics-warehouse` | ทุกข้อในตารางด้านล่าง + require 1 approval จาก Code Owner |
| **Tier 3 — Medium/Low** | `internal-dashboard`, `devtools-cli`, `marketing-site` | เฉพาะข้อบังคับพื้นฐาน (แถวที่ทำเครื่องหมาย * ด้านล่าง) |

## ข้อบังคับพื้นฐานที่ทุก Repository (ทุก Tier) ต้องเปิดใช้งาน

ตั้งค่าที่ **Settings → Branches → Branch protection rules** สำหรับ branch `main` และทุก `release/*`:

| การตั้งค่า | ค่าที่บังคับใช้ | เหตุผล |
| ---------- | --------------- | ------ |
| Require a pull request before merging * | เปิด | ห้าม push ตรงเข้า main โดยไม่ผ่าน review (Part 33) |
| Require approvals * | อย่างน้อย 1 | บังคับให้มีคนที่สองตรวจสอบโค้ดก่อนเสมอ |
| Dismiss stale pull request approvals on new commits * | เปิด | ป้องกันการ push โค้ดใหม่หลัง approve แล้วสอดไส้โดยไม่ถูกตรวจซ้ำ |
| Require review from Code Owners | เปิด (Tier 1-2) | บังคับให้เจ้าของโค้ดจริงเป็นผู้ approve ตาม CODEOWNERS (Part 36) |
| Require status checks to pass before merging * | เปิด (เลือก CI pipeline หลักของโปรเจกต์) | ห้าม merge โค้ดที่ยังไม่ผ่าน automated test/scan |
| Require branches to be up to date before merging | เปิด (Tier 1-2) | ป้องกัน "semantic conflict" จากการ merge โค้ดที่ล้าหลัง main |
| Require signed commits | เปิด (Tier 1 เท่านั้น บังคับ, Tier 2 แนะนำ) | ยืนยันตัวตนผู้เขียน commit จริง (ดู Part 79 / Step 845) |
| Require linear history | เปิด (Tier 1) | ป้องกัน merge commit ที่ซ่อนการเปลี่ยนแปลงในระบบที่ต้อง audit เข้มงวด |
| Do not allow bypassing the above settings * | เปิด | แม้แต่ Admin ก็ต้องทำตามกฎเดียวกัน ไม่มีข้อยกเว้น |
| Restrict who can push to matching branches | เปิด เฉพาะ CI/CD service account (สำหรับ auto-merge จาก GitOps) | ป้องกัน push ตรงจากบุคคลทั่วไป |
| Require deployments to succeed before merging | เปิด (Tier 1, repository ที่เชื่อมกับ GitOps) | ห้าม merge เข้า main ถ้า pipeline deploy ไปยัง staging ล้มเหลว |

(* = ข้อบังคับพื้นฐานที่ต้องเปิดทุก Tier รวมถึง Tier 3)

## CODEOWNERS มาตรฐาน

ทุก repository ต้องมีไฟล์ `.github/CODEOWNERS` กำหนดเจ้าของอย่างชัดเจน ห้ามปล่อยว่าง
ดูตัวอย่างรูปแบบที่ `templates/CODEOWNERS.template`

กฎเพิ่มเติมระดับองค์กร:

1. ทุก repository ต้องมี fallback owner ระดับ root (`*`) เป็นอย่างน้อย 1 คน หรือ 1 ทีม
   เพื่อไม่ให้มีไฟล์ใดที่ "ไม่มีเจ้าของ"
2. โฟลเดอร์ที่เกี่ยวข้องกับความปลอดภัยโดยตรง (เช่น `auth/`, `.github/workflows/`,
   `infra/`, `SECURITY.md`) **ต้องมีทีม Security เป็นหนึ่งใน Code Owner เสมอ**
   ไม่ว่า repository นั้นจะอยู่ Tier ใดก็ตาม
3. ห้ามตั้งบุคคลเพียงคนเดียวเป็น Code Owner ของโค้ดที่อยู่ Tier 1 — ต้องเป็นทีม (GitHub team
   handle เช่น `@netitech-solutions/payments-team`) เพื่อไม่ให้ระบบหยุดชะงักหากคนคนเดียวลาป่วยหรือลาออก
EOF
```

### 843.2 Template CODEOWNERS มาตรฐาน

```bash
cat > templates/CODEOWNERS.template << 'EOF'
# CODEOWNERS Template มาตรฐานองค์กร NetiTech Solutions
# คัดลอกไฟล์นี้ไปวางที่ .github/CODEOWNERS ของแต่ละ repository แล้วปรับตามทีมที่ดูแลจริง
#
# กฎการอ่าน: บรรทัดที่อยู่ "ล่างสุด" ที่ pattern ตรงกันจะมีผลบังคับใช้ (last match wins)
# จึงควรวางกฎทั่วไปไว้บนสุด และกฎเฉพาะเจาะจงไว้ล่างสุดเสมอ

# --- กฎทั่วไป (fallback เจ้าของ repository ทั้งหมด) ---
*                           @netitech-solutions/platform-team

# --- โฟลเดอร์ความปลอดภัย: ทีม Security ต้อง approve เสมอ ไม่ว่าอยู่ที่ใด ---
/.github/workflows/        @netitech-solutions/security-team @netitech-solutions/platform-team
/infra/                    @netitech-solutions/security-team @netitech-solutions/infra-team
SECURITY.md                @netitech-solutions/security-team
**/auth/                   @netitech-solutions/security-team

# --- ตัวอย่างการแบ่งตามทีมในโครงสร้าง Monorepo (อ้างอิงแนวคิด Part 82) ---
/services/payments/        @netitech-solutions/payments-team
/services/core-api/        @netitech-solutions/core-team
/apps/web/                 @netitech-solutions/frontend-team
/pipelines/                @netitech-solutions/data-team
EOF

git add policies/01-branch-protection.md templates/CODEOWNERS.template
git commit -m "เพิ่มนโยบาย Branch Protection แบบแบ่ง Tier และ CODEOWNERS template มาตรฐานองค์กร"
```

```
[main b2b2b2b] เพิ่มนโยบาย Branch Protection แบบแบ่ง Tier และ CODEOWNERS template มาตรฐานองค์กร
 2 files changed, 68 insertions(+)
```

### 843.3 Pull Request Template มาตรฐาน

คู่กับ CODEOWNERS ทีม Security ยังกำหนด Pull Request template กลางที่บังคับให้ผู้เขียนโค้ดยืนยันด้วยตัวเองก่อนขอ review ว่าได้ตรวจสอบประเด็นความปลอดภัยเบื้องต้นตามนโยบายที่จะกำหนดใน Step ถัดไปแล้ว (secret, dependency, signed commit ตามความจำเป็นของ repository นั้น):

```bash
cat > templates/pull_request_template.md << 'EOF'
## คำอธิบายการเปลี่ยนแปลง

<!-- อธิบายว่าเปลี่ยนอะไร ทำไมถึงเปลี่ยน อ้างอิง ticket/issue ที่เกี่ยวข้อง -->

## Security Checklist (ตอบทุกข้อก่อนขอ review)

- [ ] ไม่มี secret, API key หรือ credential ใด ๆ ถูก hardcode ในการเปลี่ยนแปลงนี้
- [ ] Dependency ใหม่ (ถ้ามี) ผ่านการตรวจสอบตามนโยบาย Dependency Scanning แล้ว
- [ ] ไม่มีการปิด/ข้าม security check ของ CI โดยไม่มีเหตุผลรองรับเป็นลายลักษณ์อักษร
- [ ] (เฉพาะ repository Tier 1) Commit ทั้งหมดถูกเซ็นด้วย GPG/SSH key ที่ลงทะเบียนไว้แล้ว

## Reviewer ที่เกี่ยวข้อง (ตาม CODEOWNERS)
EOF

git add templates/pull_request_template.md
git commit -m "เพิ่ม Pull Request template มาตรฐานพร้อม Security Checklist บังคับตอบก่อนขอ review"
```

```
[main b3b3b3b] เพิ่ม Pull Request template มาตรฐานพร้อม Security Checklist บังคับตอบก่อนขอ review
 1 file changed, 17 insertions(+)
```

### 843.4 การบังคับใช้จริงในระดับ Organization ด้วย Ruleset

บน GitHub Enterprise การตั้งค่าทีละ repository ไม่พอสำหรับองค์กรที่มี repository เป็นร้อย ทีม Platform ของ NetiTech จึงใช้ฟีเจอร์ **Organization Rulesets** (Settings ระดับ Organization → Repository → Rulesets) เพื่อบังคับกฎเดียวกันกับ repository ที่ตรงกับ pattern ที่กำหนด เช่น ตั้ง ruleset ชื่อ `tier1-critical` ที่ผูกกับ repository ที่ติด topic `tier-1-critical` ให้บังคับ "require signed commits" และ "require 2 approvals" โดยอัตโนมัติกับทุก repository ใหม่ที่ถูกติด topic นี้ — ทำให้ไม่ต้องพึ่งความจำของมนุษย์ในการตั้งค่าซ้ำทุกครั้งที่สร้าง repository ใหม่

---

## Step 844: กำหนดนโยบาย Secret Management ทั้งองค์กร

Step นี้นำความรู้จาก **Part 78 (Secret Management และการป้องกันข้อมูลรั่วไหลใน Git)** มาสรุปเป็นนโยบายบังคับใช้

### 844.1 หลักการ 3 ชั้นของ Secret Management

นโยบายนี้วางอยู่บนหลักการป้องกัน 3 ชั้นที่ทำงานร่วมกัน (defense in depth) แทนที่จะพึ่งพาชั้นใดชั้นหนึ่งเพียงอย่างเดียว:

1. **ป้องกันก่อนเกิด (Prevention)** — เครื่องมือที่เตือนหรือบล็อกก่อนที่ secret จะเข้าไปอยู่ใน commit ตั้งแต่แรก
2. **ตรวจจับต่อเนื่อง (Detection)** — สแกนหา secret ที่หลุดรอดเข้าไปแล้วในโค้ดหรือประวัติ Git อย่างสม่ำเสมอ
3. **ตอบสนองเมื่อรั่วไหล (Response)** — ขั้นตอนบังคับเมื่อพบว่ามี secret หลุดจริง

```bash
cat > policies/02-secret-management.md << 'EOF'
# นโยบาย Secret Management

## กฎข้อบังคับ (ห้ามฝ่าฝืนโดยเด็ดขาด)

1. **ห้าม hardcode secret ทุกชนิดลงในซอร์สโค้ดหรือไฟล์ config ที่ commit เข้า Git**
   ไม่ว่าจะเป็น API key, database password, private key, access token, webhook secret
   หรือ connection string ที่มี credential ฝังอยู่ — ไม่มีข้อยกเว้น แม้แต่ใน branch ทดลอง
   หรือ commit ที่ "จะลบทีหลัง" (ประวัติ Git ไม่ลบง่ายอย่างที่คิด)
2. **Secret ทุกตัวต้องมาจากแหล่งเก็บที่ได้รับอนุมัติเท่านั้น** ได้แก่:
   - HashiCorp Vault (สำหรับ secret ที่ใช้ตอน runtime ของแอปพลิเคชัน)
   - GitHub Actions Secrets / Environments (สำหรับ secret ที่ใช้ใน CI/CD pipeline)
   - Cloud provider secret manager (เช่น AWS Secrets Manager, GCP Secret Manager)
     สำหรับ secret ที่ผูกกับ infrastructure โดยตรง
3. **ห้ามส่ง secret ผ่านช่องทางที่ไม่เข้ารหัสหรือไม่มีการควบคุมสิทธิ์** เช่น แชทที่ไม่เข้ารหัส
   อีเมลธรรมดา หรือไฟล์แชร์สาธารณะ
4. **Secret ต้องหมุนเวียน (rotate) ตามรอบที่กำหนด** — อย่างน้อยทุก 90 วันสำหรับ Tier 1/2
   และทันทีเมื่อมีพนักงานที่เข้าถึง secret นั้นลาออกหรือเปลี่ยนบทบาท

## การป้องกันชั้นที่ 1: Pre-commit และ Push Protection

- ทุกเครื่องพัฒนาต้องติดตั้ง **pre-commit hook** ที่รัน `gitleaks protect --staged`
  ก่อนอนุญาตให้ commit สำเร็จ (ผูกผ่าน `pre-commit` framework ที่กำหนดไว้ใน
  `.pre-commit-config.yaml` ของทุก repository ตามหลัก client-side hook ที่เคยเรียนมา)
- เปิดใช้งาน **GitHub Push Protection for Secret Scanning** ในทุก repository เพื่อบล็อก
  การ push ที่มี pattern ของ secret ที่รู้จัก (เช่น API key รูปแบบมาตรฐานของผู้ให้บริการรายใหญ่)
  ตั้งแต่ต้นทางก่อนที่ secret จะเข้าไปอยู่ใน remote repository เลย

ตัวอย่างการตั้งค่า pre-commit hook มาตรฐาน:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.4
    hooks:
      - id: gitleaks
```

## การป้องกันชั้นที่ 2: Continuous Secret Scanning ใน CI

ทุก repository ต้องมี job สแกน secret รันในทุก pull request และรันซ้ำแบบเต็ม repository
(รวมประวัติทั้งหมด) เป็นรายสัปดาห์ ตัวอย่าง workflow มาตรฐาน:

```yaml
# .github/workflows/secret-scan.yml
name: secret-scan
on:
  pull_request:
  schedule:
    - cron: "0 3 * * 1"   # ทุกวันจันทร์ 03:00 UTC
jobs:
  gitleaks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Run gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITLEAKS_ENABLE_UPLOAD_ARTIFACT: true
```

## การป้องกันชั้นที่ 3: การตอบสนองเมื่อพบ Secret หลุด (Incident Response)

หากพบว่ามี secret หลุดเข้าไปใน Git ไม่ว่าจะทางใดก็ตาม ต้องปฏิบัติตามลำดับนี้ทันที
**ไม่ใช่แค่ลบไฟล์แล้ว commit ทับ** เพราะ secret นั้นยังอยู่ในประวัติเก่า:

1. **หมุนเวียน (rotate) secret ตัวที่หลุดทันที** ที่ต้นทาง (Vault/Secret Manager/ผู้ให้บริการ)
   ให้ค่าเก่าใช้งานไม่ได้อีก — ขั้นตอนนี้สำคัญที่สุดและต้องทำ**ก่อน**ขั้นตอนอื่นเสมอ
2. แจ้งทีม Security ทันทีผ่านช่องทาง incident response เพื่อประเมินผลกระทบว่ามีการใช้งาน
   โดยไม่ได้รับอนุญาตเกิดขึ้นแล้วหรือไม่ (ตรวจสอบ access log ของ secret นั้น)
3. ลบ secret ออกจากประวัติ Git ด้วยเครื่องมือที่เหมาะสม (`git filter-repo` หรือใช้ GitHub
   Support สำหรับกรณี repository ขนาดใหญ่ที่มีคน clone ไปแล้วจำนวนมาก) และประสานทุกคนที่มี
   local clone ให้ clone ใหม่หลัง force-push ประวัติที่แก้ไขแล้ว
4. บันทึกเหตุการณ์เป็นลายลักษณ์อักษรลง audit log (ดูนโยบาย Compliance ใน Step 848)
   พร้อมสาเหตุ (root cause) และมาตรการป้องกันไม่ให้เกิดซ้ำ
EOF

git add policies/02-secret-management.md
git commit -m "เพิ่มนโยบาย Secret Management: ป้องกัน 3 ชั้น (prevention, detection, response)"
```

```
[main c3c3c3c] เพิ่มนโยบาย Secret Management: ป้องกัน 3 ชั้น (prevention, detection, response)
 1 file changed, 75 insertions(+)
```

> **ข้อควรระวังสำคัญ:** ทุกตัวอย่างค่า secret ที่ปรากฏในเอกสารของ Part นี้ **เป็นชื่อ placeholder ที่ไม่ใช่ค่าจริงเด็ดขาด** (เช่น `<VAULT_ADDR>`, `<DB_PASSWORD>`) การเขียนนโยบาย secret management ที่ดีจะต้องไม่มีตัวอย่างค่า secret ที่ดูสมจริงปนอยู่ในเอกสารเอง เพราะเอกสารนโยบายมักถูกเก็บไว้ใน repository ที่คนเข้าถึงได้กว้างกว่า repository จริง

---

## Step 845: กำหนดนโยบาย Signed Commits สำหรับ Repository ที่สำคัญ

Step นี้นำความรู้จาก **Part 79 (Git Security: Signed Commits, GPG, SSH Signing)** มากำหนดเป็นข้อบังคับ

### 845.1 ทำไมต้องบังคับ Signed Commits เฉพาะ Repository สำคัญ

Signed commits พิสูจน์ว่า commit นั้นถูกสร้างโดยผู้ถือ private key ที่ตรงกับ public key ที่ลงทะเบียนไว้จริง ๆ ป้องกันการปลอมแปลงชื่อผู้เขียน (`git commit --author` ปลอมชื่อใครก็ได้ถ้าไม่มีการเซ็นยืนยัน) แต่การบังคับทุก repository รวมถึง repository ทดลองเล็ก ๆ จะสร้างภาระเกินจำเป็น NetiTech จึงบังคับตามระดับ Tier ที่กำหนดไว้ใน Step 843

```bash
cat > policies/03-signed-commits.md << 'EOF'
# นโยบาย Signed Commits

## ขอบเขตการบังคับใช้

| Tier | บังคับ Signed Commits? |
| ---- | ----------------------- |
| Tier 1 — Critical (`payments-service`, `infra-terraform` ฯลฯ) | **บังคับ** ทุก commit ที่ merge เข้า `main`/`release/*` |
| Tier 2 — High | แนะนำอย่างยิ่ง (จะบังคับเต็มรูปแบบภายในไตรมาสถัดไป) |
| Tier 3 — Medium/Low | ไม่บังคับ |

Tag ที่ใช้สำหรับ production release (`v*.*.*`) ของทุก Tier ที่มีการ deploy ขึ้นระบบจริง
**ต้องเป็น annotated + signed tag เสมอ** ไม่ว่า repository จะอยู่ Tier ใดก็ตาม
เพื่อยืนยันว่า release นั้นมาจากกระบวนการที่ได้รับอนุมัติจริง ไม่ถูกปลอมแปลงระหว่างทาง

## วิธีที่บังคับใช้ทางเทคนิค

1. เปิด **"Require signed commits"** ในกฎ branch protection ของ Tier 1 ทุก repository
   (Git จะปฏิเสธการ push commit ที่ไม่ได้เซ็นเข้า branch ที่มีการป้องกันนี้ทันที)
2. พนักงานทุกคนที่ทำงานกับ repository Tier 1 ต้อง:
   - สร้างคีย์ GPG หรือคีย์ SSH สำหรับเซ็น commit (SSH signing รองรับตั้งแต่ Git 2.34)
   - ลงทะเบียน public key กับบัญชี GitHub ของตนเอง (Settings → SSH and GPG keys)
   - ตั้งค่าเครื่องของตัวเองให้เซ็นทุก commit โดยอัตโนมัติ:
     ```bash
     git config --global user.signingkey <KEY_ID>
     git config --global commit.gpgsign true
     git config --global tag.gpgsign true
     ```
3. ตรวจสอบสถานะการเซ็นด้วย `git log --show-signature` หรือดู badge "Verified"
   บนหน้า commit/PR ของ GitHub

## การจัดการวงจรชีวิตของคีย์ (Key Lifecycle)

- คีย์ที่ใช้เซ็น commit ต้องตั้งวันหมดอายุไม่เกิน **2 ปี** และต่ออายุก่อนหมดอายุล่วงหน้า
- เมื่อพนักงานลาออกหรือย้ายทีมออกจากงานที่เกี่ยวข้องกับ Tier 1 ต้อง **เพิกถอน (revoke)**
  คีย์ของบุคคลนั้นออกจากบัญชี GitHub ทันทีในวันทำงานสุดท้าย
- ทีม Security เก็บทะเบียนกลาง (key registry) ของ public key/fingerprint ทุกดวงที่ได้รับอนุญาต
  ให้เซ็น commit เข้า repository Tier 1 เพื่อใช้ตรวจสอบย้อนหลังเวลามี audit
- ห้ามใช้คีย์เดียวกันเซ็นทั้งงานส่วนตัวและงานบริษัท และห้ามแชร์ private key ร่วมกันระหว่างบุคคล
  ไม่ว่ากรณีใด (private key ผูกกับตัวบุคคลเสมอ ไม่ใช่ทีม)

## กรณีตัวอย่าง: การตรวจสอบก่อน Release

ก่อน tag เวอร์ชัน production ทุกครั้ง ทีม Release ต้องรัน:

```bash
git log --show-signature -1 HEAD
git tag -v v5.2.0
```

และยืนยันว่าลายเซ็นของทั้ง commit ล่าสุดและ tag เป็น "Good signature" จาก key ที่อยู่ใน
ทะเบียนที่ได้รับอนุญาตเท่านั้น ก่อนดำเนินการ deploy ขึ้นระบบจริงทุกครั้ง
EOF

git add policies/03-signed-commits.md
git commit -m "เพิ่มนโยบาย Signed Commits สำหรับ repository ระดับ Tier 1 และ release tag ทุก Tier"
```

```
[main d4d4d4d] เพิ่มนโยบาย Signed Commits สำหรับ repository ระดับ Tier 1 และ release tag ทุก Tier
 1 file changed, 54 insertions(+)
```

---

## Step 846: กำหนดนโยบาย Dependency Scanning + SBOM สำหรับทุกโปรเจกต์

Step นี้นำความรู้จาก **Part 80 (Dependency Scanning และ Supply Chain Security)** มากำหนดเป็นข้อบังคับที่ใช้กับ**ทุกโปรเจกต์ไม่ว่า Tier ใด** เพราะช่องโหว่จาก dependency (supply chain attack) กระทบทุกระดับความสำคัญเท่ากันได้ — โปรเจกต์ internal เล็ก ๆ ที่ใช้ library ที่ถูกฝังมัลแวร์ก็เป็นประตูเข้าสู่เครือข่ายภายในได้เหมือนกัน

### 846.1 องค์ประกอบของนโยบาย

```bash
cat > policies/04-dependency-scanning.md << 'EOF'
# นโยบาย Dependency Scanning และ Supply Chain Security

## กฎข้อบังคับ (ใช้กับทุก Tier)

1. ทุก repository ที่มีไฟล์ manifest ของ dependency (`package.json`, `requirements.txt`,
   `go.mod`, `pom.xml` ฯลฯ) **ต้องเปิดใช้งาน Dependabot** (หรือเทียบเท่า) เพื่อ:
   - แจ้งเตือนอัตโนมัติเมื่อพบช่องโหว่ที่ทราบแล้ว (known CVE) ใน dependency ที่ใช้อยู่
   - เปิด Pull Request อัปเดตเวอร์ชันโดยอัตโนมัติเมื่อมีแพตช์ความปลอดภัยออกมา
2. ทุก Pull Request ต้องผ่าน **automated dependency vulnerability scan** ใน CI ก่อน merge
   ได้เสมอ (เป็นส่วนหนึ่งของ required status check ตามนโยบาย Branch Protection ใน Step 843)
3. ห้าม merge Pull Request ที่ทำให้เกิดช่องโหว่ระดับ **Critical หรือ High** ใหม่เพิ่มขึ้น
   โดยไม่มีการอนุมัติยกเว้นเป็นลายลักษณ์อักษรจากทีม Security (พร้อมแผนแก้ไขและกำหนดเวลา)
4. ทุก repository ที่ build เป็น artifact สำหรับ deploy จริง (container image, binary,
   package ที่เผยแพร่) **ต้องสร้าง SBOM (Software Bill of Materials)** แนบไปกับทุก release
5. Dependency ใหม่ที่จะเพิ่มเข้าโปรเจกต์ Tier 1/2 ต้องผ่านการตรวจสอบเบื้องต้นก่อนอนุมัติ:
   - มีการดูแล (maintain) อย่างต่อเนื่อง ไม่ถูกทิ้งร้างเกิน 2 ปี
   - จำนวนผู้ใช้งาน/ดาวน์โหลดอยู่ในระดับที่เชื่อถือได้ ไม่ใช่ package ใหม่ที่ไม่มีประวัติ
   - Licence เข้ากันได้กับนโยบายลิขสิทธิ์ของบริษัท (ห้ามใช้ license กลุ่ม copyleft
     แบบเข้มงวดในซอฟต์แวร์ที่จำหน่ายเชิงพาณิชย์ โดยไม่ผ่านฝ่ายกฎหมายก่อน)

## Workflow มาตรฐาน: Scan + สร้าง SBOM ใน CI

```yaml
# .github/workflows/dependency-security.yml
name: dependency-security
on:
  pull_request:
  push:
    tags:
      - "v*.*.*"
jobs:
  vulnerability-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Scan dependencies for known vulnerabilities
        uses: aquasecurity/trivy-action@0.24.0
        with:
          scan-type: fs
          severity: CRITICAL,HIGH
          exit-code: 1

  generate-sbom:
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Generate SBOM (CycloneDX format)
        uses: anchore/sbom-action@v0
        with:
          format: cyclonedx-json
          output-file: sbom-${{ github.ref_name }}.cdx.json
      - name: Upload SBOM as release artifact
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: sbom-*.cdx.json
```

## มาตรฐาน SBOM ที่ใช้

องค์กรกำหนดให้ใช้รูปแบบ **CycloneDX** หรือ **SPDX** อย่างใดอย่างหนึ่งอย่างสม่ำเสมอทั้งองค์กร
(NetiTech เลือกใช้ CycloneDX เป็นมาตรฐานหลัก) โดย SBOM ของทุก release ต้องถูกเก็บรักษาไว้
อย่างน้อย **3 ปี** เพื่อให้สามารถตรวจสอบย้อนหลังได้ทันทีเมื่อมี CVE ใหม่ประกาศออกมาในอนาคต
ว่าระบบเวอร์ชันใดของบริษัทเคยใช้ library ที่มีปัญหานั้นอยู่บ้าง (ไม่ต้องไล่ build ซ้ำเพื่อตรวจสอบ)

## แนวคิด Supply Chain Security เพิ่มเติมที่นำมาใช้

- **Provenance / Attestation:** ทุก artifact ที่ build จาก CI ต้องแนบหลักฐานว่า build มาจาก
  source code และ workflow ใดจริง (แนวทางตามกรอบ SLSA — Supply-chain Levels for Software
  Artifacts) เพื่อป้องกันการสอดไส้ artifact ปลอมที่ไม่ได้มาจาก pipeline ที่ได้รับอนุมัติ
- **Pin เวอร์ชันของ dependency แบบ Lockfile เสมอ** (`package-lock.json`, `poetry.lock`,
  `go.sum`) และ commit lockfile เข้า repository เสมอ ห้ามใช้ version range แบบเปิดกว้าง
  ในสภาพแวดล้อม production
- **Pin เวอร์ชันของ GitHub Action ด้วย commit SHA หรือเวอร์ชันที่ระบุชัดเจน** แทนการใช้
  `@main` หรือ `@latest` เพื่อป้องกัน supply chain attack ผ่าน CI/CD pipeline เอง
EOF

git add policies/04-dependency-scanning.md
git commit -m "เพิ่มนโยบาย Dependency Scanning และ SBOM บังคับใช้ทุก Tier"
```

```
[main e5e5e5e] เพิ่มนโยบาย Dependency Scanning และ SBOM บังคับใช้ทุก Tier
 1 file changed, 79 insertions(+)
```

---

## Step 847: กำหนดนโยบาย Backup / Disaster Recovery

Step นี้นำความรู้จาก **Part 84 (Disaster Recovery และ Backup Strategy สำหรับ Git)** มากำหนดเป็นนโยบายที่วัดผลได้ด้วยตัวเลขชัดเจน ไม่ใช่แค่คำสัญญาลอย ๆ

### 847.1 กำหนด RPO และ RTO ตาม Tier

หัวใจของนโยบาย Disaster Recovery คือการกำหนด **RPO (Recovery Point Objective — ยอมเสียข้อมูลได้มากสุดกี่นาที/ชั่วโมง)** และ **RTO (Recovery Time Objective — กู้คืนระบบให้ใช้งานได้ภายในกี่ชั่วโมง)** ให้ชัดเจนเป็นตัวเลข เพราะ "สำรองข้อมูลไว้เป็นประจำ" เป็นคำพูดที่วัดผลไม่ได้

```bash
cat > policies/05-backup-disaster-recovery.md << 'EOF'
# นโยบาย Backup และ Disaster Recovery

## เป้าหมาย RPO/RTO ตามระดับความสำคัญ

| Tier | RPO (ข้อมูลสูญเสียได้ไม่เกิน) | RTO (กู้คืนให้ใช้งานได้ภายใน) |
| ---- | ------------------------------ | ------------------------------- |
| Tier 1 — Critical | 15 นาที | 1 ชั่วโมง |
| Tier 2 — High | 1 ชั่วโมง | 4 ชั่วโมง |
| Tier 3 — Medium/Low | 24 ชั่วโมง | 24 ชั่วโมง |

## กลไกการสำรองข้อมูลที่บังคับใช้

1. **GitHub Enterprise เป็นแหล่งความจริงหลัก (source of truth)** แต่ **ห้ามพึ่งพา GitHub
   เพียงแหล่งเดียว** ตามหลักการที่เรียนใน Part 84 — องค์กรต้องมีสำเนาสำรองอิสระที่ไม่ได้ขึ้นกับ
   ผู้ให้บริการรายเดียว (ป้องกันเหตุการณ์ GitHub ล่ม, บัญชีถูกระงับโดยไม่ได้ตั้งใจ, หรือถูกโจมตี
   จนข้อมูลเสียหาย)
2. ระบบ mirror อัตโนมัติทุก repository จาก GitHub Enterprise ไปยัง **object storage ที่บริษัท
   ควบคุมเอง** ทุก 15 นาทีสำหรับ Tier 1 และทุก 1 ชั่วโมงสำหรับ Tier 2-3 โดยใช้:

   ```bash
   # สคริปต์ backup แบบ mirror clone (รันผ่าน cron/CI scheduled job)
   for repo in $(cat repos-tier1.txt); do
     if [ -d "backups/${repo}.git" ]; then
       git -C "backups/${repo}.git" remote update --prune
     else
       git clone --mirror "git@github.com:netitech-solutions/${repo}.git" "backups/${repo}.git"
     fi
   done
   ```

3. หลัง mirror เสร็จ ให้ sync โฟลเดอร์ `backups/` ไปยัง storage แบบ **immutable/write-once**
   (เช่น object storage ที่เปิดใช้งาน object lock/versioning) เพื่อป้องกันไม่ให้ backup เอง
   ถูกลบหรือแก้ไขได้ แม้ว่าบัญชีที่ใช้รัน backup จะถูกแฮ็กก็ตาม
4. เก็บสำเนาสำรองไว้ **อย่างน้อย 2 ภูมิภาค (region)** ทางภูมิศาสตร์ที่แยกจากกัน (geographic
   redundancy) เพื่อไม่ให้เหตุการณ์ระดับภูมิภาค (เช่น ศูนย์ข้อมูลไฟไหม้) ทำลายสำเนาสำรองทั้งหมด
5. Release artifact, SBOM, และ audit log (ดู Step 846, 848) ต้อง backup ควบคู่ไปกับ Git
   repository เสมอ เพราะการกู้คืนแค่ source code โดยไม่มีบริบทประกอบไม่เพียงพอต่อการทำงานต่อ

## การซ้อมกู้คืน (Restore Drill) — บังคับ ไม่ใช่ทางเลือก

Backup ที่ไม่เคยถูกทดสอบกู้คืนจริง **ถือว่าไม่มีอยู่จริง** ทีม Platform ต้องจัดการซ้อมกู้คืน
ตามรอบต่อไปนี้ และบันทึกผลทุกครั้งลง audit log:

| Tier | ความถี่ในการซ้อมกู้คืน |
| ---- | ------------------------ |
| Tier 1 | ทุกไตรมาส (4 ครั้ง/ปี) |
| Tier 2 | ทุกครึ่งปี (2 ครั้ง/ปี) |
| Tier 3 | ทุกปี (1 ครั้ง/ปี) |

ขั้นตอนการซ้อมกู้คืนมาตรฐาน:

```bash
# 1. กู้คืนจาก mirror backup ไปยัง repository เปล่าใหม่ (จำลองว่า GitHub หายไปจริง)
git clone --mirror backups/payments-service.git restore-test/payments-service.git

# 2. ตรวจสอบความสมบูรณ์ของทุก object ในประวัติ
git -C restore-test/payments-service.git fsck --full --strict

# 3. ตรวจสอบว่า branch, tag สำคัญทั้งหมดอยู่ครบ (โดยเฉพาะ signed tag ของ release ที่ใช้งานจริง)
git -C restore-test/payments-service.git tag -l
git -C restore-test/payments-service.git branch -a

# 4. ตรวจสอบว่า tag ที่สำคัญยังคงมีลายเซ็นถูกต้องหลังกู้คืน (ไม่ถูกทำลายระหว่างขั้นตอน backup)
git -C restore-test/payments-service.git tag -v v5.2.0

# 5. วัดเวลาทั้งหมดที่ใช้ตั้งแต่เริ่มกู้คืนจนถึงระบบพร้อมใช้งาน แล้วเทียบกับ RTO ที่กำหนดไว้
```

หากการซ้อมกู้คืนครั้งใดใช้เวลาเกิน RTO ที่กำหนด ต้องเปิด incident ภายในเพื่อปรับปรุงกระบวนการ
ก่อนรอบซ้อมครั้งถัดไป ห้ามปล่อยผ่านโดยไม่มีการแก้ไข
EOF

git add policies/05-backup-disaster-recovery.md
git commit -m "เพิ่มนโยบาย Backup/Disaster Recovery พร้อมกำหนด RPO/RTO ตาม Tier และรอบซ้อมกู้คืน"
```

```
[main f6f6f6f] เพิ่มนโยบาย Backup/Disaster Recovery พร้อมกำหนด RPO/RTO ตาม Tier และรอบซ้อมกู้คืน
 1 file changed, 71 insertions(+)
```

---

## Step 848: กำหนดนโยบาย Compliance / Audit Trail

Step นี้นำความรู้จาก **Part 81 (Compliance และ Audit Trail ด้วย Git)** มากำหนดเป็นนโยบายที่เชื่อมโยงกิจกรรมใน Git เข้ากับข้อกำหนดของมาตรฐานสากลที่องค์กรต้องปฏิบัติตาม

### 848.1 เชื่อมโยงนโยบายก่อนหน้ากับมาตรฐาน Compliance

จุดสำคัญของ Step นี้คือการแสดงให้เห็นว่า **นโยบายทั้งหมดที่เขียนมาใน Step 842–847 ไม่ได้มีไว้เพื่อความปลอดภัยเพียงอย่างเดียว แต่ยังเป็นหลักฐาน (evidence) ที่ใช้ตอบข้อกำหนดของมาตรฐานการตรวจสอบสากลได้โดยตรง**

```bash
cat > policies/06-compliance-audit.md << 'EOF'
# นโยบาย Compliance และ Audit Trail

## หลักการ: ทุกการเปลี่ยนแปลงต้องตรวจสอบย้อนกลับได้ (Traceability)

องค์กรต้องสามารถตอบคำถามต่อไปนี้ได้เสมอสำหรับการเปลี่ยนแปลงโค้ดหรือ infrastructure ใด ๆ
ในระบบที่กระทบข้อมูลลูกค้าหรือเงิน:

- **ใคร** เป็นคนเปลี่ยน (Who)
- **เปลี่ยนอะไร** (What) — diff ที่ชัดเจน ไม่ใช่แค่คำอธิบาย
- **เมื่อไหร่** (When) — timestamp ที่แก้ไขไม่ได้
- **ทำไม** (Why) — เชื่อมโยงกับ ticket/issue ที่อนุมัติงานนั้น
- **ใครอนุมัติ** (Approved by) — ผ่านกระบวนการ review ที่กำหนด ไม่ใช่ self-merge

โครงสร้าง Git ที่วางไว้ในนโยบายก่อนหน้าทั้งหมด (branch protection บังคับ PR + review,
CODEOWNERS, signed commits, immutable history) **คือกลไกที่ตอบคำถามทั้ง 5 ข้อนี้โดยอัตโนมัติ**
อยู่แล้วในตัว Git เอง โดยไม่ต้องสร้างระบบ audit แยกต่างหาก

## แหล่งข้อมูล Audit Log ที่บังคับเก็บรักษา

| แหล่งข้อมูล | เก็บอะไร | ระยะเวลาเก็บรักษาขั้นต่ำ |
| ----------- | -------- | -------------------------- |
| Git commit history | Diff, ผู้เขียน, เวลา, ลายเซ็น (สำหรับ Tier 1) | ตลอดอายุของ repository (ห้าม rewrite history ของ branch ที่ merge แล้ว) |
| Pull Request records | ผู้ review, ความเห็น, เวลาที่ approve/merge | อย่างน้อย 7 ปี (ตามข้อกำหนดด้านการเงินทั่วไป) |
| GitHub Enterprise Audit Log | การเปลี่ยนแปลงระดับ organization เช่น เพิ่ม/ลบสมาชิก, เปลี่ยนสิทธิ์, เปลี่ยนกฎ branch protection | อย่างน้อย 3 ปี ส่งออกไปเก็บใน SIEM ขององค์กรทุกวัน |
| CI/CD Pipeline log | ผลการ build/test/scan ของทุก deployment | อย่างน้อย 3 ปี |
| Secret access log (จาก Vault) | ใครเข้าถึง secret ตัวใด เมื่อไหร่ | อย่างน้อย 3 ปี |

## ข้อกำหนดที่เชื่อมโยงกับมาตรฐานสากล

| ข้อกำหนดในมาตรฐาน | นโยบายของเราที่ตอบข้อกำหนดนี้ |
| -------------------- | ------------------------------- |
| Access control ต้องมีการทบทวนสิทธิ์เป็นระยะ (SOC 2 CC6, ISO/IEC 27001 A.5.18) | ทบทวนสมาชิกทีมใน CODEOWNERS/GitHub Team ทุกไตรมาส บันทึกผลการทบทวนไว้เป็นหลักฐาน |
| Change management ต้องมีการอนุมัติก่อนเปลี่ยนแปลงระบบ production (SOC 2 CC8, ISO/IEC 27001 A.8.32) | Branch Protection + Required Review ตาม Step 843 |
| ต้องป้องกันข้อมูลลับรั่วไหล (PCI-DSS Req. 3, ISO/IEC 27001 A.8.24) | นโยบาย Secret Management ตาม Step 844 |
| ต้องตรวจสอบความปลอดภัยของ dependency บุคคลที่สาม (PCI-DSS Req. 6, SOC 2 CC7) | นโยบาย Dependency Scanning + SBOM ตาม Step 846 |
| ต้องมีแผนกู้คืนระบบและทดสอบจริง (ISO/IEC 27001 A.5.29-30, SOC 2 A1) | นโยบาย Backup/DR + Restore Drill ตาม Step 847 |
| ต้องพิสูจน์ตัวตนผู้ทำการเปลี่ยนแปลงได้ (ISO/IEC 27001 A.8.5) | นโยบาย Signed Commits ตาม Step 845 |

## กฎเพิ่มเติมเพื่อรักษาความสมบูรณ์ของ Audit Trail

1. **ห้าม force-push เข้า branch ที่มีการป้องกัน** (`main`, `release/*`) ในทุกกรณี ยกเว้นได้รับ
   อนุมัติเป็นลายลักษณ์อักษรจากทีม Security สำหรับกรณีฉุกเฉินที่มีเหตุผลรองรับ (เช่น ลบ secret
   ที่หลุดตามขั้นตอนใน Step 844) และต้องบันทึกเหตุการณ์นั้นไว้ทันที
2. **ห้ามลบ Pull Request หรือ Issue ที่เกี่ยวข้องกับการเปลี่ยนแปลงระบบ production** แม้จะปิด
   งานแล้วก็ตาม เพราะเป็นส่วนหนึ่งของหลักฐาน audit
3. Export **GitHub Enterprise Audit Log** ไปเก็บใน SIEM ส่วนกลางของบริษัททุกวันโดยอัตโนมัติ
   ผ่าน Audit Log API เพื่อป้องกันไม่ให้หลักฐานหายไปหากมีการเปลี่ยนแปลงสิทธิ์ในระดับ organization
4. จัดทำรายงานสรุปสถานะ compliance ทุกไตรมาส (compliance report) ที่ดึงข้อมูลจากทั้ง 4 นโยบาย
   ก่อนหน้า (branch protection coverage, secret scan findings, dependency scan findings,
   ผลการซ้อมกู้คืน) ส่งให้ผู้บริหารและผู้ตรวจสอบภายนอกเมื่อมีการร้องขอ
EOF

git add policies/06-compliance-audit.md
git commit -m "เพิ่มนโยบาย Compliance/Audit Trail เชื่อมโยงกับมาตรฐาน SOC 2, ISO/IEC 27001, PCI-DSS"
```

```
[main 070707a] เพิ่มนโยบาย Compliance/Audit Trail เชื่อมโยงกับมาตรฐาน SOC 2, ISO/IEC 27001, PCI-DSS
 1 file changed, 62 insertions(+)
```

---

## Step 849: จัดทำเอกสาร Security Policy ฉบับเต็มรวมทุกหัวข้อเป็นไฟล์เดียว

ตอนนี้เรามีไฟล์นโยบายย่อย 6 ไฟล์ที่ทีม Security ดูแลง่าย แต่สำหรับ **ผู้บริหาร ผู้ตรวจสอบภายนอก และพนักงานใหม่** การต้องเปิดอ่านหลายไฟล์เป็นเรื่องไม่สะดวก ทีม Security ของ NetiTech จึงจัดทำเอกสารรวมฉบับเดียว `SECURITY_POLICY_FULL.md` ที่ดึงสาระสำคัญของทุกไฟล์มารวมกันในโครงสร้างที่อ่านจบแล้วเข้าใจภาพรวมทั้งหมดได้โดยไม่ต้องสลับไปมาระหว่างไฟล์

### 849.1 หลักการจัดโครงสร้างเอกสารรวม

เอกสารรวมที่ดีต้องมี:

1. **หน้าสรุปผู้บริหาร (Executive Summary)** — สรุปสั้น ๆ ว่านโยบายนี้ครอบคลุมอะไรบ้าง สำหรับคนที่ไม่มีเวลาอ่านทั้งหมด
2. **สารบัญที่ลิงก์ไปยังแต่ละหัวข้อ** — เพื่อให้ค้นหาหัวข้อที่ต้องการได้เร็ว
3. **เนื้อหาทุกนโยบายย่อยเรียงตามลำดับตรรกะ** ไม่ใช่ตามลำดับที่เขียนขึ้น (เช่น เริ่มจากภาพรวม → การควบคุมการเข้าถึง → การป้องกันข้อมูล → การตรวจสอบ → การกู้คืน)
4. **ตารางสรุปความรับผิดชอบ (RACI)** — ระบุว่าใครรับผิดชอบอะไรในการบังคับใช้นโยบายแต่ละข้อ
5. **ประวัติการแก้ไขเอกสาร (Revision History)** — สำคัญมากสำหรับ audit เพราะผู้ตรวจสอบต้องรู้ว่านโยบายมีผลบังคับใช้ตั้งแต่เมื่อไหร่

### 849.2 สร้างเอกสารรวมด้วยสคริปต์

แทนที่จะ copy-paste เนื้อหาด้วยมือ (ซึ่งเสี่ยงให้เอกสารสองชุดไม่ตรงกันในอนาคต) ทีม Platform เขียนสคริปต์ประกอบเอกสารอัตโนมัติจากไฟล์ย่อย เพื่อให้ `SECURITY_POLICY_FULL.md` ถูกสร้างขึ้นจากไฟล์ต้นทางเสมอ ไม่มีวันไม่ตรงกัน:

```bash
cat > build-full-policy.sh << 'EOF'
#!/bin/sh
# build-full-policy.sh: ประกอบ SECURITY_POLICY_FULL.md จากไฟล์นโยบายย่อยทั้งหมด
# รันสคริปต์นี้ทุกครั้งที่แก้ไขไฟล์ใน policies/ เพื่อให้เอกสารรวมตรงกับต้นทางเสมอ

OUT="SECURITY_POLICY_FULL.md"

cat > "$OUT" << HEADER
# Security Policy ฉบับสมบูรณ์ — NetiTech Solutions

> เอกสารนี้ประกอบขึ้นอัตโนมัติจากไฟล์ใน policies/ ห้ามแก้ไขไฟล์นี้โดยตรง
> ให้แก้ไขที่ไฟล์ต้นทางใน policies/ แล้วรัน build-full-policy.sh ใหม่เสมอ
>
> เวอร์ชันเอกสาร: มีผลบังคับใช้ตั้งแต่ไตรมาสที่จัดทำ (ดู Revision History ท้ายเอกสาร)

## สารบัญ

1. [สรุปสำหรับผู้บริหาร](#สรุปสำหรับผู้บริหาร)
2. [นโยบายการรายงานช่องโหว่](#นโยบายการรายงานช่องโหว่)
3. [นโยบาย Branch Protection และ CODEOWNERS](#นโยบาย-branch-protection-และ-codeowners)
4. [นโยบาย Secret Management](#นโยบาย-secret-management)
5. [นโยบาย Signed Commits](#นโยบาย-signed-commits)
6. [นโยบาย Dependency Scanning และ SBOM](#นโยบาย-dependency-scanning-และ-sbom)
7. [นโยบาย Backup และ Disaster Recovery](#นโยบาย-backup-และ-disaster-recovery)
8. [นโยบาย Compliance และ Audit Trail](#นโยบาย-compliance-และ-audit-trail)
9. [ตารางความรับผิดชอบ (RACI)](#ตารางความรับผิดชอบ-raci)
10. [ประวัติการแก้ไขเอกสาร](#ประวัติการแก้ไขเอกสาร)

## สรุปสำหรับผู้บริหาร

NetiTech Solutions บังคับใช้ Security Policy ฉบับนี้กับทุก repository ภายใต้องค์กร
โดยแบ่ง repository ออกเป็น 3 ระดับความสำคัญ (Tier) และกำหนดมาตรการป้องกัน 6 ด้าน ได้แก่
การรายงานช่องโหว่, การควบคุมการเข้าถึงโค้ด, การจัดการข้อมูลลับ, การยืนยันตัวตนผู้เขียนโค้ด,
การป้องกันความเสี่ยงจาก dependency ภายนอก, การกู้คืนระบบ และการตรวจสอบย้อนหลัง
ทุกมาตรการถูกบังคับใช้ผ่านกลไกอัตโนมัติใน Git และ CI/CD ไม่ใช่เพียงกฎที่เขียนไว้บนกระดาษ
HEADER

echo "## นโยบายการรายงานช่องโหว่" >> "$OUT"
echo "" >> "$OUT"
tail -n +2 SECURITY.md >> "$OUT"
echo "" >> "$OUT"
echo "---" >> "$OUT"
echo "" >> "$OUT"

for f in policies/01-branch-protection.md \
         policies/02-secret-management.md \
         policies/03-signed-commits.md \
         policies/04-dependency-scanning.md \
         policies/05-backup-disaster-recovery.md \
         policies/06-compliance-audit.md; do
  tail -n +2 "$f" >> "$OUT"
  echo "" >> "$OUT"
  echo "---" >> "$OUT"
  echo "" >> "$OUT"
done

cat >> "$OUT" << 'FOOTER'
## ตารางความรับผิดชอบ (RACI)

| กิจกรรม | Responsible | Accountable | Consulted | Informed |
| ------- | ----------- | ----------- | --------- | -------- |
| ตั้งค่า Branch Protection/CODEOWNERS | Platform Team | CISO | เจ้าของแต่ละ repository | นักพัฒนาทุกคน |
| ตรวจสอบ Secret Scanning ที่แจ้งเตือน | เจ้าของ repository | Security Team | Platform Team | ผู้จัดการทีมที่เกี่ยวข้อง |
| จัดการคีย์เซ็น Commit | พนักงานแต่ละคน | Security Team | Platform Team | ผู้จัดการทีมที่เกี่ยวข้อง |
| อนุมัติ dependency ใหม่ Tier 1/2 | เจ้าของ repository | Security Team | ฝ่ายกฎหมาย (license) | Platform Team |
| ซ้อมกู้คืนระบบ (Restore Drill) | Platform Team | CTO | Security Team | ผู้บริหารทุกฝ่าย |
| จัดทำรายงาน Compliance รายไตรมาส | Security Team | CISO | Platform Team, ฝ่ายกฎหมาย | คณะกรรมการบริษัท |

## ประวัติการแก้ไขเอกสาร

| เวอร์ชัน | รายละเอียดการเปลี่ยนแปลง |
| -------- | -------------------------- |
| 1.0 | จัดทำนโยบายฉบับแรก ครอบคลุม 6 ด้านหลัก ประกาศใช้ทั้งองค์กร |
FOOTER

echo "สร้างเอกสารรวมสำเร็จ: $OUT"
EOF

chmod +x build-full-policy.sh
./build-full-policy.sh
```

```
สร้างเอกสารรวมสำเร็จ: SECURITY_POLICY_FULL.md
```

### 849.3 ตรวจสอบผลลัพธ์และ commit

```bash
wc -l SECURITY_POLICY_FULL.md
```

```
412 SECURITY_POLICY_FULL.md
```

```bash
git add build-full-policy.sh SECURITY_POLICY_FULL.md
git commit -m "เพิ่มสคริปต์ประกอบเอกสารอัตโนมัติและเผยแพร่ SECURITY_POLICY_FULL.md เวอร์ชัน 1.0"
```

```
[main 181818b] เพิ่มสคริปต์ประกอบเอกสารอัตโนมัติและเผยแพร่ SECURITY_POLICY_FULL.md เวอร์ชัน 1.0
 2 files changed, 421 insertions(+)
```

### 849.4 บังคับใช้ผ่าน CI: ตรวจสอบว่าเอกสารรวมไม่ตกยุค

เพื่อไม่ให้ `SECURITY_POLICY_FULL.md` เก่ากว่าไฟล์ต้นทางในอนาคต (เช่น มีคนแก้ `policies/02-secret-management.md` แล้วลืมรันสคริปต์) ทีม Platform เพิ่ม CI job ที่รันสคริปต์ใหม่ทุกครั้งแล้วเทียบว่าผลลัพธ์ตรงกับไฟล์ที่ commit ไว้หรือไม่:

```yaml
# .github/workflows/policy-consistency-check.yml
name: policy-consistency-check
on:
  pull_request:
    paths:
      - "policies/**"
      - "SECURITY.md"
      - "SECURITY_POLICY_FULL.md"
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Rebuild full policy and diff against committed version
        run: |
          cp SECURITY_POLICY_FULL.md /tmp/before.md
          ./build-full-policy.sh
          diff /tmp/before.md SECURITY_POLICY_FULL.md
```

หาก `diff` พบความต่าง CI จะ fail ทันที บังคับให้ผู้เขียน pull request ต้องรันสคริปต์และ commit เอกสารรวมเวอร์ชันล่าสุดมาด้วยเสมอ — นี่คือการนำแนวคิด GitOps/Policy-as-Code (Part 76) มาใช้กับเอกสารนโยบายเองอย่างเป็นรูปธรรม

```bash
git add .github/workflows/policy-consistency-check.yml
git commit -m "เพิ่ม CI ตรวจสอบว่า SECURITY_POLICY_FULL.md ตรงกับไฟล์ต้นทางเสมอ"
```

```
[main 292929c] เพิ่ม CI ตรวจสอบว่า SECURITY_POLICY_FULL.md ตรงกับไฟล์ต้นทางเสมอ
 1 file changed, 17 insertions(+)
```

ตอนนี้ NetiTech Solutions มี Security Policy ฉบับสมบูรณ์ที่:

- อยู่ใน Git จริง มีประวัติการแก้ไข ตรวจสอบย้อนหลังได้ (ตรงตามหลักการของนโยบายเอง)
- แยกเป็นไฟล์ย่อยที่ทีมต่าง ๆ ดูแลได้อิสระ แต่รวมเป็นเอกสารเดียวให้ผู้บริหาร/ผู้ตรวจสอบอ่านได้ง่าย
- มีกลไกอัตโนมัติป้องกันไม่ให้เอกสารสองชุดไม่ตรงกัน
- ครอบคลุมทั้ง 6 ด้านที่ตกลงไว้ตั้งแต่ Step 841: Vulnerability Disclosure, Branch Protection/CODEOWNERS, Secret Management, Signed Commits, Dependency Scanning/SBOM, Backup/DR และ Compliance/Audit

---

## Step 850: สรุปทบทวนภาพรวมเฟส 8 ทั้งหมด (Part 76–85) พร้อม Cheat Sheet และ Checklist ก่อนเข้าเฟส 9

### 850.1 โครงสร้าง Repository สุดท้ายของโปรเจกต์ Security Policy

```bash
find . -type f -not -path "./.git/*" | sort
```

```
./SECURITY.md
./SECURITY_POLICY_FULL.md
./build-full-policy.sh
./.github/workflows/policy-consistency-check.yml
./policies/01-branch-protection.md
./policies/02-secret-management.md
./policies/03-signed-commits.md
./policies/04-dependency-scanning.md
./policies/05-backup-disaster-recovery.md
./policies/06-compliance-audit.md
./templates/CODEOWNERS.template
./templates/pull_request_template.md
```

```bash
git log --oneline
```

```
292929c เพิ่ม CI ตรวจสอบว่า SECURITY_POLICY_FULL.md ตรงกับไฟล์ต้นทางเสมอ
181818b เพิ่มสคริปต์ประกอบเอกสารอัตโนมัติและเผยแพร่ SECURITY_POLICY_FULL.md เวอร์ชัน 1.0
070707a เพิ่มนโยบาย Compliance/Audit Trail เชื่อมโยงกับมาตรฐาน SOC 2, ISO/IEC 27001, PCI-DSS
f6f6f6f เพิ่มนโยบาย Backup/Disaster Recovery พร้อมกำหนด RPO/RTO ตาม Tier และรอบซ้อมกู้คืน
e5e5e5e เพิ่มนโยบาย Dependency Scanning และ SBOM บังคับใช้ทุก Tier
d4d4d4d เพิ่มนโยบาย Signed Commits สำหรับ repository ระดับ Tier 1 และ release tag ทุก Tier
c3c3c3c เพิ่มนโยบาย Secret Management: ป้องกัน 3 ชั้น (prevention, detection, response)
b2b2b2b เพิ่มนโยบาย Branch Protection แบบแบ่ง Tier และ CODEOWNERS template มาตรฐานองค์กร
a1a1a1a เพิ่ม SECURITY.md: นโยบายการรายงานช่องโหว่ตามมาตรฐาน GitHub Security Advisory
```

ประวัติ commit ทั้งหมดนี้เอง **คือหลักฐานชิ้นแรกที่พิสูจน์ว่านโยบายนี้เกิดขึ้นจริง ผ่านกระบวนการที่ตรวจสอบได้** — ไม่ใช่เอกสาร Word ที่ไม่มีใครรู้ว่าเขียนขึ้นเมื่อไหร่

### 850.2 Cheat Sheet รวมแนวคิดทั้งหมดของเฟส 8 (Part 76–85)

| Part | หัวข้อ | แนวคิด/เครื่องมือหลัก | ใช้เมื่อไหร่ |
|---|---|---|---|
| 76 | GitOps | ใช้ Git เป็น source of truth เดียวของสถานะระบบ, reconciliation loop, pull-based deployment | ต้องการให้ทุกการเปลี่ยนแปลง infra/deployment ผ่านการ review และ audit ได้เหมือนโค้ดแอปพลิเคชัน |
| 77 | Infrastructure as Code + Git | Terraform/Pulumi ร่วมกับ Git, `terraform plan` ใน PR, state file management | จัดการ infrastructure แบบประกาศ (declarative) แทนการคลิกตั้งค่าด้วยมือ |
| 78 | Secret Management | Vault, GitHub Secrets, gitleaks/trufflehog, push protection, key rotation | ป้องกันและตรวจจับข้อมูลลับหลุดเข้า Git |
| 79 | Signed Commits | GPG/SSH signing, `commit.gpgsign`, verified badge, key registry | ยืนยันตัวตนผู้เขียน commit/tag จริงในระบบสำคัญ |
| 80 | Dependency Scanning/Supply Chain | Dependabot, SBOM (CycloneDX/SPDX), SLSA provenance, lockfile pinning | ป้องกันความเสี่ยงจาก library ภายนอกและ CI pipeline ที่ถูกโจมตี |
| 81 | Compliance/Audit Trail | Audit Log API, SIEM, traceability (who/what/when/why/approved by) | ตอบข้อกำหนดมาตรฐาน SOC 2, ISO/IEC 27001, PCI-DSS |
| 82 | Monorepo vs Polyrepo | ข้อดี/ข้อเสียของแต่ละแบบ, sparse-checkout, CODEOWNERS แบ่งตามโฟลเดอร์ | ตัดสินใจเชิงสถาปัตยกรรมตามขนาดทีมและการพึ่งพากันของโค้ด |
| 83 | Enterprise-scale Repository | Partial clone, shallow clone, Git LFS ที่สเกล, Scalar, การแบ่ง Tier ความสำคัญ | บริหารจัดการ repository ขนาดใหญ่และองค์กรที่มีทีมจำนวนมาก |
| 84 | Disaster Recovery/Backup | `git clone --mirror`, RPO/RTO, geographic redundancy, restore drill | รับประกันว่าประวัติ Git ไม่สูญหายแม้ผู้ให้บริการหลักล่ม |
| 85 | Security Policy Capstone | รวมทุกอย่างข้างต้นเป็นเอกสารนโยบายที่บังคับใช้จริง | ปิดเฟส 8 ก่อนเข้าเฟส 9 (Maintainer/Release/Metrics) |

### 850.3 ตารางสรุปคำสั่ง/เครื่องมือที่ใช้บ่อยที่สุดตลอดเฟส 8

| คำสั่ง / เครื่องมือ | ความหมาย |
|---|---|
| `git config commit.gpgsign true` | บังคับให้เซ็น commit ด้วย GPG/SSH key ทุกครั้งโดยอัตโนมัติ |
| `git log --show-signature` | ตรวจสอบว่า commit มีลายเซ็นถูกต้องหรือไม่ |
| `git tag -v <tag>` | ตรวจสอบลายเซ็นของ annotated tag ก่อน release |
| `git clone --mirror` | สร้างสำเนา repository แบบเต็มรวมทุก ref สำหรับ backup |
| `git fsck --full --strict` | ตรวจสอบความสมบูรณ์ของ object ทั้งหมดหลังกู้คืนจาก backup |
| `gitleaks protect --staged` | สแกนหา secret ก่อนอนุญาตให้ commit สำเร็จ (pre-commit hook) |
| Dependabot / `trivy` / `pip-audit` | สแกนหาช่องโหว่ที่ทราบแล้วใน dependency |
| `syft` / `anchore/sbom-action` | สร้าง SBOM ในรูปแบบ CycloneDX/SPDX จาก source หรือ image |
| GitHub Branch Protection Rules / Rulesets | บังคับ require review, require signed commits, require status checks ทั้ง repository หรือทั้ง organization |
| `.github/CODEOWNERS` | กำหนดผู้ตรวจสอบบังคับตามโฟลเดอร์/ไฟล์ |
| GitHub Enterprise Audit Log API | ดึงประวัติกิจกรรมระดับ organization ออกไปเก็บใน SIEM |

### 850.4 Checklist ทบทวนภาพรวมเฟส 8 ทั้งหมด (Part 76–85) ก่อนเข้าสู่เฟส 9

ก่อนไปต่อ Part 86 (เริ่มต้นเฟส 9: มืออาชีพ/Enterprise Practice) ให้ตรวจสอบตัวเองอย่างละเอียดตามรายการนี้ ถ้าข้อไหนยังไม่มั่นใจ แนะนำให้ย้อนกลับไปอ่าน Part ที่เกี่ยวข้องอีกครั้งก่อน:

**GitOps และ Infrastructure as Code**
- [ ] อธิบายได้ว่า GitOps ต่างจาก CI/CD แบบ push-based ทั่วไปอย่างไร และทำไม Git ถึงเหมาะเป็น source of truth ของ infrastructure
- [ ] เข้าใจการจัดการ Terraform/IaC state ร่วมกับ Pull Request workflow และความเสี่ยงของ state file ที่ต้องระวังเป็นพิเศษ

**Secret Management**
- [ ] อธิบายหลักการป้องกัน 3 ชั้น (prevention, detection, response) ของ secret ได้
- [ ] ตั้งค่า pre-commit hook และ CI scanning เพื่อดักจับ secret ก่อนหลุดเข้า repository ได้ด้วยตัวเอง
- [ ] รู้ขั้นตอนที่ถูกต้องเมื่อพบ secret รั่วไหลจริง (rotate ก่อนเสมอ ไม่ใช่แค่ลบไฟล์)

**Signed Commits**
- [ ] สร้างและตั้งค่าคีย์ GPG หรือ SSH สำหรับเซ็น commit และ tag ได้ด้วยตัวเอง
- [ ] เข้าใจว่าทำไมต้องบังคับ signed commits เฉพาะ repository ที่มีความเสี่ยงสูง ไม่ใช่บังคับทุกที่แบบเหมารวม
- [ ] จัดการวงจรชีวิตของคีย์ (สร้าง ต่ออายุ เพิกถอน) ได้อย่างเป็นระบบ

**Dependency Scanning และ Supply Chain Security**
- [ ] อธิบายความแตกต่างระหว่าง SBOM รูปแบบ CycloneDX และ SPDX ได้ในระดับแนวคิด
- [ ] ตั้งค่า Dependabot และ automated vulnerability scanning ใน CI ได้ด้วยตัวเอง
- [ ] เข้าใจแนวคิด SLSA/provenance และเหตุผลที่ต้อง pin เวอร์ชันของทั้ง dependency และ GitHub Action

**Compliance และ Audit Trail**
- [ ] เชื่อมโยงกลไกของ Git (branch protection, CODEOWNERS, signed commits) เข้ากับข้อกำหนดของมาตรฐาน SOC 2 / ISO 27001 / PCI-DSS ได้
- [ ] อธิบายได้ว่าทำไมประวัติ Git ที่ไม่ถูก rewrite คือรากฐานสำคัญของ audit trail ที่น่าเชื่อถือ

**Monorepo/Polyrepo และ Enterprise-scale Repository**
- [ ] เปรียบเทียบข้อดีข้อเสียของ Monorepo กับ Polyrepo ได้พร้อมยกตัวอย่างสถานการณ์ที่เหมาะสม
- [ ] รู้จักเทคนิคจัดการ repository ขนาดใหญ่ (partial clone, sparse-checkout, Git LFS ที่สเกล) และรู้ว่าใช้เมื่อไหร่

**Backup และ Disaster Recovery**
- [ ] กำหนด RPO/RTO ให้เหมาะกับความสำคัญของแต่ละ repository ได้ ไม่ใช้ค่าเดียวกันหมด
- [ ] ตั้งค่า mirror backup อัตโนมัติและอธิบายได้ว่าทำไมต้องมี geographic redundancy
- [ ] เข้าใจว่า backup ที่ไม่เคยซ้อมกู้คืนถือว่าเชื่อถือไม่ได้ และวางแผนซ้อมกู้คืนเป็นรอบได้

**ภาพรวมโปรเจกต์ Security Policy**
- [ ] จัดโครงสร้างเอกสารนโยบายที่แยกไฟล์ย่อยตามหัวข้อ แต่ประกอบเป็นเอกสารรวมได้อัตโนมัติโดยไม่ต้อง copy-paste ด้วยมือ
- [ ] ออกแบบระบบ Tier ความสำคัญของ repository และกำหนดมาตรการป้องกันที่แตกต่างกันตาม Tier ได้อย่างสมเหตุสมผล
- [ ] อธิบายวงจรทั้งหมดของ Security Policy หนึ่งฉบับ ตั้งแต่ร่างนโยบาย บังคับใช้ทางเทคนิค ไปจนถึงตรวจสอบและปรับปรุงได้อย่างเป็นระบบให้คนอื่นฟังได้

ถ้าคุณติ๊กครบทุกข้อ (หรือเกือบครบ) แปลว่าคุณพร้อมสำหรับเฟส 9 อย่างแท้จริงแล้ว — เฟส 8 คือเฟสที่เปลี่ยนคุณจากคนที่ "ใช้ Git ทำงานเป็นทีมได้" ไปสู่คนที่ **"เข้าใจว่า Git คือเครื่องมือกำกับดูแลความปลอดภัยระดับองค์กร"** ซึ่งเป็นทักษะที่แยกวิศวกรระดับ senior/staff ออกจากวิศวกรระดับ mid อย่างชัดเจนที่สุดในโลกการทำงานจริง โดยเฉพาะในองค์กรที่ต้องผ่านการตรวจสอบ (audit) หรือทำงานกับข้อมูลที่มีความอ่อนไหวสูง เช่น การเงิน สุขภาพ หรือข้อมูลส่วนบุคคล

---

## สรุป Part 85

ใน Part นี้เราได้จำลองทีม Security & Compliance ขององค์กร NetiTech Solutions วางแผนและจัดทำ Security Policy ฉบับเต็มตั้งแต่ต้นจนจบ โดยนำทุกทักษะที่เรียนมาตลอดเฟส 8 มาใช้งานร่วมกันในสถานการณ์ที่จำลองมาจากการทำงานจริง:

1. วางโครงสร้างองค์กรจำลอง แบ่ง repository เป็น 3 ระดับ Tier ตามความสำคัญ และวางโครงสร้างเอกสารนโยบายที่แยกไฟล์ย่อยตามหัวข้อ (Step 841)
2. เขียน `SECURITY.md` ตามมาตรฐานที่ GitHub รองรับ ครอบคลุม Supported Versions, ช่องทางรายงาน, SLA การตอบสนอง และ Safe Harbor (Step 842)
3. กำหนดนโยบาย Branch Protection แบบแบ่ง Tier พร้อม CODEOWNERS template มาตรฐานที่ใช้ได้ทั้งองค์กร (Step 843)
4. กำหนดนโยบาย Secret Management แบบป้องกัน 3 ชั้น: prevention, detection, response (Step 844)
5. กำหนดนโยบาย Signed Commits สำหรับ repository ระดับวิกฤต พร้อมการจัดการวงจรชีวิตของคีย์ (Step 845)
6. กำหนดนโยบาย Dependency Scanning และ SBOM ที่บังคับใช้กับทุก repository ไม่ว่า Tier ใด (Step 846)
7. กำหนดนโยบาย Backup/Disaster Recovery พร้อม RPO/RTO ที่วัดผลได้จริง และรอบการซ้อมกู้คืนที่บังคับ (Step 847)
8. กำหนดนโยบาย Compliance/Audit Trail ที่เชื่อมโยงกลไกของ Git เข้ากับข้อกำหนดของมาตรฐานสากล (Step 848)
9. รวมทุกนโยบายเป็นเอกสารฉบับสมบูรณ์เดียวด้วยสคริปต์อัตโนมัติ พร้อม CI ตรวจสอบความสอดคล้องของเอกสาร (Step 849)
10. ทบทวนภาพรวมคำสั่งและแนวคิดทั้งหมดของเฟส 8 ผ่าน Cheat Sheet และ Checklist ครบทุกด้าน (Step 850)

Part นี้คือจุดปิดฉากของ **เฟส 8: DevOps / Security / Compliance (Part 76–85, Step 751–850)** อย่างสมบูรณ์ คุณได้ผ่านการฝึกฝนทักษะที่แยกวิศวกรที่ "ทำงานเป็นทีมด้วย Git ได้" ออกจากวิศวกรที่ **"วางระบบกำกับดูแลความปลอดภัยของ Git ให้ทั้งองค์กรได้"** มาครบทุกมิติแล้ว ตั้งแต่การควบคุมการเข้าถึงด้วย Branch Protection/CODEOWNERS, การป้องกันข้อมูลลับ, การยืนยันตัวตนด้วย Signed Commits, การรับมือความเสี่ยงจาก Supply Chain, การกู้คืนระบบเมื่อเกิดภัยพิบัติ, ไปจนถึงการเชื่อมโยงทุกอย่างเข้ากับมาตรฐาน Compliance สากล

จากนี้ไป หลักสูตรจะพาคุณเข้าสู่ **เฟส 9: มืออาชีพ / Enterprise Practice (Part 86–95, Step 851–950)** ซึ่งจะเจาะลึกทักษะที่แยกวิศวกรมืออาชีพระดับสูงออกจากวิศวกรทั่วไป ตั้งแต่การเป็น Maintainer โปรเจกต์ Open Source, การบริหาร Release, ไปจนถึงการใช้ข้อมูลจาก Git วัดผลการทำงานของทีมทั้งองค์กร

**ต่อไป:** [Part 86: การเป็น Maintainer โปรเจกต์ Open Source](./part-086-maintainer-open-source.md)
