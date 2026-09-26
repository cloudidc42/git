# Part 80: Dependency Scanning และ Supply Chain Security

> **Step ในหลักสูตรนี้:** Step 791–800
> **เฟส:** 8 — DevOps, Security, Compliance ระดับองค์กร
> **เป้าหมายของ Part นี้:** เข้าใจว่า Software Supply Chain คืออะไร ทำไมมันถึงกลายเป็นเป้าหมายการโจมตีที่อันตรายที่สุดในวงการซอฟต์แวร์ยุคปัจจุบัน เรียนรู้รูปแบบการโจมตีจริงที่เกิดขึ้น (Dependency Confusion, Typosquatting) เข้าใจแนวคิดและเครื่องมือป้องกัน เช่น SBOM, Lock File, Dependabot/Renovate, Sigstore/cosign และกรอบมาตรฐาน SLSA แล้วลงมือปฏิบัติจริงกับโปรเจกต์ของตัวเอง

---

## สารบัญของ Part นี้

- Step 791: Software Supply Chain คืออะไร ทำไมกลายเป็นเป้าโจมตีสำคัญ (กรณีศึกษา SolarWinds 2020)
- Step 792: Dependency Confusion Attack คืออะไร
- Step 793: Typosquatting ใน Package Registry คืออะไร
- Step 794: Software Bill of Materials (SBOM) คืออะไร
- Step 795: เครื่องมือสร้าง SBOM — Syft, CycloneDX, SPDX
- Step 796: Dependency Pinning และ Lock File — สิ่งที่ห้ามลบทิ้งเด็ดขาด
- Step 797: Automated Dependency Updates ด้วย Dependabot และ Renovate
- Step 798: Sigstore/cosign — การ Sign และ Verify Software Artifact แบบ Cryptographic
- Step 799: SLSA Framework เบื้องต้น (Supply-chain Levels for Software Artifacts)
- Step 800: แบบฝึกหัด — เปิดใช้ Dependabot จริง และสร้าง SBOM เบื้องต้น

---

## Step 791: Software Supply Chain คืออะไร ทำไมกลายเป็นเป้าโจมตีสำคัญ

### Software Supply Chain คืออะไร

ลองนึกภาพ **ห่วงโซ่อุปทาน (Supply Chain)** ของอาหารสักจาน กว่าจะมาถึงจานของคุณ วัตถุดิบต้องผ่านเกษตรกร โรงงานแปรรูป ผู้ขนส่ง คลังสินค้า ซูเปอร์มาร์เก็ต และครัวของร้านอาหาร ถ้าจุดใดจุดหนึ่งในห่วงโซ่นี้ปนเปื้อนสารพิษ ผลกระทบจะไหลไปถึงปลายทางโดยที่คุณผู้บริโภคไม่มีทางรู้เลยว่าปัญหาเกิดจากที่ไหน

**Software Supply Chain** ก็มีลักษณะเดียวกันทุกประการ ซอฟต์แวร์หนึ่งตัวที่คุณใช้งานจริงไม่ได้ถูกเขียนขึ้นมาจากศูนย์โดยทีมของคุณทั้งหมด แต่ประกอบขึ้นจากส่วนประกอบจำนวนมหาศาลที่มาจากแหล่งต่าง ๆ:

```
Source Code (โค้ดที่ทีมเขียนเอง)
        │
        ▼
Open Source Dependencies (Library/Package จาก npm, PyPI, RubyGems, Maven, ฯลฯ)
        │
        ▼
Build Tools & Compilers (webpack, tsc, gcc, Docker base image)
        │
        ▼
CI/CD Pipeline (GitHub Actions, GitLab CI, Jenkins)
        │
        ▼
Artifact Registry (Docker Hub, npm registry, internal registry)
        │
        ▼
Deployment / Distribution (Production server, App Store, Auto-update mechanism)
        │
        ▼
ผู้ใช้งานปลายทาง (End User)
```

จุดสำคัญคือ **ทุกจุดในห่วงโซ่นี้คือพื้นผิวการโจมตี (attack surface)** ไม่ใช่แค่โค้ดที่ทีมเขียนเองเท่านั้น งานวิจัยของ Sonatype (State of the Software Supply Chain) พบซ้ำแล้วซ้ำเล่าในหลายปีว่า **โค้ดในแอปพลิเคชันสมัยใหม่โดยเฉลี่ยกว่า 80–90% เป็นโค้ดจาก Open Source dependency ไม่ใช่โค้ดที่ทีมเขียนเอง** นั่นหมายความว่าถ้าคุณไม่ตรวจสอบความปลอดภัยของ dependency เลย คุณกำลังปล่อยให้ผิวหนังส่วนใหญ่ของแอปพลิเคชันไม่มีการป้องกันใด ๆ

### ทำไม Supply Chain ถึงกลายเป็นเป้าโจมตีที่ "คุ้มค่า" ที่สุดสำหรับแฮกเกอร์

ในอดีต แฮกเกอร์ต้องเจาะระบบทีละบริษัท ทีละเซิร์ฟเวอร์ แต่ถ้าแฮกเกอร์สามารถแทรกโค้ดร้ายเข้าไปใน **library ยอดนิยมตัวเดียว** ที่มีคนใช้งานหลักหมื่นถึงหลักล้านโปรเจกต์ทั่วโลก หรือแทรกเข้าไปใน **pipeline การ build** ของซอฟต์แวร์ที่มีลูกค้าองค์กรนับพันราย นั่นคือการโจมตีแบบ **"หนึ่งจุด กระทบทุกที่" (one-to-many)** ที่คุ้มค่าต่อความพยายามมากกว่าการเจาะระบบทีละที่มหาศาล

นี่คือเหตุผลที่ในช่วงปี 2020 เป็นต้นมา การโจมตี Supply Chain เพิ่มขึ้นอย่างก้าวกระโดด และหน่วยงานความปลอดภัยไซเบอร์ทั่วโลกยกให้เป็นหนึ่งในภัยคุกคามอันดับต้น ๆ ขององค์กร

### กรณีศึกษา: SolarWinds (2020) — การโจมตี Supply Chain ที่เขย่าโลก

นี่คือกรณีศึกษาที่ถูกอ้างอิงมากที่สุดในวงการความปลอดภัยไซเบอร์ระดับโลก

**เบื้องหลัง:** SolarWinds เป็นบริษัทซอฟต์แวร์สัญชาติอเมริกันที่ผลิตซอฟต์แวร์บริหารจัดการระบบเครือข่ายและ IT infrastructure ชื่อ **Orion Platform** ซึ่งถูกใช้งานโดยหน่วยงานรัฐบาลสหรัฐฯ และบริษัทขนาดใหญ่ทั่วโลกรวมกันประมาณ **18,000 องค์กร**

**สิ่งที่เกิดขึ้น:** ผู้โจมตี (ซึ่งภายหลังหน่วยงานรัฐบาลสหรัฐฯ ระบุว่าเป็นกลุ่มที่เชื่อมโยงกับหน่วยข่าวกรองต่างประเทศของรัสเซีย รู้จักในชื่อ APT29 หรือ Cozy Bear) ไม่ได้โจมตีลูกค้าของ SolarWinds โดยตรง แต่กลับ **แทรกซึมเข้าไปใน Build Environment (สภาพแวดล้อมที่ใช้คอมไพล์และสร้างซอฟต์แวร์) ของ SolarWinds เอง** แล้วฝังโค้ดร้ายที่เรียกว่า **SUNBURST** เข้าไปในกระบวนการ build ของ Orion Platform โดยตรง

ผลลัพธ์คือ **ไฟล์อัปเดตซอฟต์แวร์ที่ถูกเซ็นรับรอง (digitally signed) อย่างถูกต้องตามกฎหมายของ SolarWinds เอง กลับมีมัลแวร์ฝังอยู่ข้างใน** เมื่อลูกค้าอัปเดตซอฟต์แวร์ตามปกติผ่านช่องทางที่เชื่อถือได้ พวกเขาก็ติดตั้งมัลแวร์เข้าไปในระบบของตัวเองโดยไม่รู้ตัว

**ผลกระทบ:** หน่วยงานรัฐบาลสหรัฐฯ หลายแห่งถูกเจาะระบบ รวมถึงกระทรวงการคลัง (Treasury) กระทรวงพาณิชย์ (Commerce) กระทรวงการต่างประเทศ (State) และหน่วยงานความมั่นคงแห่งมาตุภูมิ (DHS) รวมถึงบริษัทเทคโนโลยีชั้นนำอย่าง Microsoft, Cisco, Intel และแม้แต่บริษัทความปลอดภัยไซเบอร์เอง คือ **FireEye** ก็ถูกเจาะด้วย โดยจริง ๆ แล้ว FireEye เป็นผู้ที่ **ค้นพบการโจมตีนี้เป็นรายแรก** ในเดือนธันวาคม 2020 หลังจากตรวจพบว่าเครื่องมือ Red Team ภายในของตัวเองถูกขโมยไป และสืบสาวกลับไปจนพบต้นตอที่ Orion Platform

**บทเรียนสำคัญที่วงการได้รับ:**

1. **การเซ็นรับรองซอฟต์แวร์ (digital signature) ไม่ได้แปลว่าปลอดภัย** — ถ้า build pipeline ที่สร้างซอฟต์แวร์ถูกเจาะ โค้ดร้ายจะถูกเซ็นรับรองอย่างถูกต้องตามกฎหมายไปพร้อมกับโค้ดจริง
2. **ความไว้วางใจ (trust) ที่มีต่อ vendor รายเดียวสามารถกลายเป็นจุดล้มเหลวจุดเดียว (single point of failure) ที่กระทบทั้งห่วงโซ่**
3. เหตุการณ์นี้เป็นหนึ่งในแรงผลักดันสำคัญที่ทำให้รัฐบาลสหรัฐฯ ออก **Executive Order 14028 "Improving the Nation's Cybersecurity"** ในเดือนพฤษภาคม 2021 ซึ่งกำหนดให้ผู้ขายซอฟต์แวร์ให้กับหน่วยงานรัฐต้องจัดทำ **SBOM (Software Bill of Materials)** ซึ่งเราจะเรียนใน Step 794

### กรณีศึกษาเพิ่มเติมที่ควรรู้จัก

| เหตุการณ์ | ปี | ลักษณะการโจมตี |
|---|---|---|
| **event-stream (npm)** | 2018 | ผู้ดูแล package เดิมยกสิทธิ์ maintainer ให้คนแปลกหน้า ผู้รับช่วงต่อแทรก dependency ร้าย (`flatmap-stream`) เพื่อขโมย Bitcoin wallet ของแอป Copay |
| **Codecov Bash Uploader** | 2021 | แฮกเกอร์แก้ไขสคริปต์ uploader ของ Codecov ผ่านช่องโหว่ในการสร้าง Docker image ทำให้ขโมย environment variable/secret จาก CI pipeline ของลูกค้าจำนวนมากได้นานหลายเดือนโดยไม่มีใครรู้ |
| **xz-utils backdoor (CVE-2024-3094)** | 2024 | ผู้สมทบโค้ด (contributor) ที่สร้างความไว้วางใจมานานหลายปีในโปรเจกต์ compression library `xz/liblzma` แอบฝัง backdoor ผ่านสคริปต์ build ที่ซับซ้อนและถูกออกแบบมาให้ตรวจจับยาก ถูกค้นพบโดยวิศวกรของ Microsoft (Andres Freund) โดยบังเอิญจากการสังเกตความผิดปกติของ performance ของ `sshd` |

จะเห็นว่าการโจมตี Supply Chain มีได้หลายรูปแบบ ทั้งการเจาะ build pipeline ของ vendor, การยึด maintainer account ของ open source, และการฝังโค้ดผ่านความไว้เนื้อเชื่อใจระยะยาว — ใน Step ถัดไปเราจะเจาะลึกรูปแบบการโจมตีที่พบบ่อยและเฉพาะเจาะจงกับ package registry มากขึ้น

---

## Step 792: Dependency Confusion Attack คืออะไร

### ที่มาของปัญหา

บริษัทขนาดใหญ่จำนวนมากมี **internal package** หรือ package ภายในองค์กรที่ใช้ร่วมกันระหว่างทีมต่าง ๆ เช่น package ชื่อ `company-auth-utils` หรือ `internal-logger` ซึ่งถูกเก็บไว้ใน **private registry** ภายในบริษัท (เช่น Artifactory, Nexus, หรือ private npm registry) ไม่ได้เผยแพร่สู่สาธารณะ

ปัญหาคือ **เครื่องมือจัดการ package (package manager) หลายตัว เช่น npm และ pip ถูกออกแบบมาให้สามารถค้นหา package จากหลาย registry พร้อมกันได้** และเมื่อพบ package ชื่อเดียวกันจากหลายแหล่ง หลายกรณี package manager จะเลือก **เวอร์ชันที่สูงที่สุด (highest version wins)** โดยไม่สนใจว่า package นั้นมาจาก registry ที่น่าเชื่อถือหรือไม่

### กลไกการโจมตี

**Dependency Confusion** คือการที่ผู้โจมตี:

1. ค้นหาหรือเดาชื่อ internal package ของบริษัทเป้าหมาย (บางครั้งชื่อเหล่านี้หลุดออกมาผ่านไฟล์ `package.json` ที่ commit ขึ้น public GitHub repository โดยไม่ตั้งใจ หรือผ่าน error message, job posting ที่พูดถึงชื่อ internal tool)
2. สร้าง package **ชื่อเดียวกันเป๊ะ ๆ** บน public registry (เช่น npm, PyPI) แต่ใส่ **เลขเวอร์ชันที่สูงกว่ามาก** (เช่น `999.9.9`)
3. ฝังโค้ดร้าย (มักเป็นสคริปต์ที่รันอัตโนมัติตอนติดตั้ง เช่น npm's `postinstall` hook) ไว้ใน package นั้น เพื่อขโมยข้อมูล เช่น environment variable, credential, hostname ของเครื่อง build server
4. รอให้ build system หรือ CI/CD pipeline ของบริษัทเป้าหมาย ที่ตั้งค่า registry แบบไม่รัดกุมพอ ไปดึง package เวอร์ชันสูงกว่าจาก public registry มาใช้แทน internal package ตัวจริงโดยอัตโนมัติ

```
Internal Registry:  company-auth-utils@2.3.1   (เวอร์ชันจริงของบริษัท)
Public Registry:    company-auth-utils@999.9.9 (มัลแวร์ที่แฮกเกอร์อัปโหลด)

Package manager เห็นทั้งสองแหล่ง → เลือกเวอร์ชันสูงกว่า (999.9.9) → ติดตั้งมัลแวร์โดยอัตโนมัติ
```

### กรณีศึกษาจริง: Alex Birsan (2021)

นักวิจัยด้านความปลอดภัยชื่อ **Alex Birsan** ตีพิมพ์บทความชื่อ *"Dependency Confusion: How I Hacked Into Apple, Microsoft and Dozens of Other Companies"* ในต้นปี 2021 โดยเขาใช้เทคนิคนี้ยิงเข้าไปทดสอบกับบริษัทเทคโนโลยีชั้นนำกว่า **35 บริษัท** รวมถึง **Apple, Microsoft, PayPal, Tesla, Netflix, Uber, Shopify, Yelp** และสามารถรันโค้ดของตัวเองบนระบบ internal ของบริษัทเหล่านี้ได้สำเร็จจริง เขาได้รับเงินรางวัล bug bounty รวมกันมากกว่า **130,000 ดอลลาร์สหรัฐ** จากการรายงานช่องโหว่นี้อย่างมีจริยธรรม (ไม่ได้นำไปใช้ในทางร้าย)

งานวิจัยนี้ทำให้ทั้งวงการตื่นตัวอย่างมาก เพราะมันแสดงให้เห็นว่าการโจมตีนี้ **ไม่ต้องแฮกอะไรเลย** เพียงแค่เข้าใจพฤติกรรม default ของ package manager ก็เพียงพอ

### วิธีป้องกัน

| วิธีป้องกัน | รายละเอียด |
|---|---|
| **ใช้ Scoped Package** | npm รองรับ package แบบ scope เช่น `@company/auth-utils` ซึ่งบังคับให้ต้องระบุ registry ของ scope นั้นอย่างชัดเจนใน `.npmrc` ทำให้ไม่มีทางไปชนกับ public registry โดยไม่ตั้งใจ |
| **กำหนด Registry แบบเจาะจงต่อ scope/index** | ตั้งค่าใน `.npmrc` หรือ `pip.conf` ให้ระบุชัดเจนว่า package ภายในต้องดึงจาก private registry เท่านั้น ไม่ fallback ไป public registry |
| **จองชื่อ (namespace squatting) ไว้ล่วงหน้า** | บางองค์กรเลือกที่จะ "จอง" ชื่อ internal package บน public registry ไว้เป็น placeholder เปล่า ๆ เพื่อไม่ให้แฮกเกอร์แย่งชื่อไปใช้ได้ |
| **ใช้ Lock File + Integrity Hash เสมอ** | เพื่อให้ build ดึง package ตามที่ resolve ไว้แน่นอนแล้วเท่านั้น (ดู Step 796) |
| **ตรวจสอบ Dependency Resolution ใน CI** | ใช้เครื่องมือสแกน dependency ตรวจสอบว่า package ที่ resolve จริงมาจากแหล่งที่คาดหวังหรือไม่ |

---

## Step 793: Typosquatting ใน Package Registry คืออะไร

### นิยาม

**Typosquatting** คือการที่ผู้โจมตีตั้งชื่อ package ให้ **คล้ายกับชื่อ package ยอดนิยมมาก ๆ** จนคนพิมพ์ผิดพลาดแล้วติดตั้ง package ปลอมไปโดยไม่รู้ตัว หรือแม้แต่กรณีที่คัดลอกชื่อจากบทความ/StackOverflow ที่มีคนพิมพ์ผิดไว้

ตัวอย่างที่โจทย์ยกมา: `lodash` เป็น library JavaScript ที่มีคนใช้งานมากที่สุดตัวหนึ่งในโลก ผู้โจมตีอาจสร้าง package ชื่อ **`1odash`** โดยแทนที่ตัวอักษร **l** (L ตัวเล็ก) ด้วยตัวเลข **1** ซึ่งเมื่อมองด้วยตาเปล่าในฟอนต์หลายแบบแทบแยกไม่ออกเลยว่าต่างกัน

### เทคนิคการตั้งชื่อหลอกที่พบบ่อย

| เทคนิค | ตัวอย่าง |
|---|---|
| **แทนตัวอักษรด้วยตัวเลข/สัญลักษณ์ที่หน้าตาคล้ายกัน (Homoglyph)** | `lodash` → `1odash`, `paypal` → `paypa1` |
| **สลับตำแหน่งตัวอักษร (Transposition)** | `python-dateutil` → `pyhton-dateutil` |
| **ตัดตัวอักษรออก (Omission)** | `crossenv` (มัลแวร์จริงที่เคยพบ) เลียนแบบ `cross-env` (package จริง) |
| **สลับ hyphen/underscore** | `cross_env` แทน `cross-env` |
| **เติมคำต่อท้ายที่ดูสมเหตุสมผล** | `electron` → `electorn`, หรือ `django` → `djanga` |
| **ใช้ตัวอักษรที่คล้ายกันข้าม encoding (Unicode homoglyph)** | ใช้ตัวอักษรซีริลลิก (Cyrillic) ที่หน้าตาเหมือนตัวอักษรละตินทุกประการ |

### กลไกการโจมตี

เมื่อผู้ใช้พิมพ์คำสั่งผิดโดยไม่ตั้งใจ เช่น:

```bash
# ตั้งใจจะพิมพ์
npm install cross-env

# แต่พิมพ์ผิดเป็น
npm install crossenv
```

package ปลอมจะถูกติดตั้งลงเครื่อง และมักจะฝัง **สคริปต์อัตโนมัติที่รันตอนติดตั้ง** (เช่น `postinstall` ใน npm หรือ `setup.py` ที่มีโค้ดรันตอน build ใน PyPI) เพื่อ:

- ขโมย environment variable, API key, credential ที่เก็บไว้ในเครื่อง
- ติดตั้ง cryptominer แอบขุดเหรียญคริปโตด้วยทรัพยากรเครื่องเหยื่อ
- เปิด backdoor หรือ reverse shell ให้แฮกเกอร์เข้าถึงเครื่องได้
- ในกรณีร้ายแรง อาจพยายามลอกเลียนพฤติกรรมของ package จริงเพื่อไม่ให้ถูกจับได้ทันที (มัลแวร์บางตัวจะ proxy การทำงานไปยัง package จริงด้วยซ้ำ เพื่อไม่ให้แอปพลิเคชัน error จนถูกสังเกตเห็น)

### วิธีป้องกัน

1. **พิมพ์ชื่อ package ให้ถูกต้องเสมอ และ copy-paste จากเอกสารทางการแทนการพิมพ์เอง**
2. **ตรวจสอบก่อนติดตั้งเสมอ** — ดูจำนวนยอดดาวน์โหลด, จำนวนผู้ maintain, วันที่เผยแพร่ล่าสุด, README ว่าดูสมเหตุสมผลหรือไม่
3. **ใช้ Lock File** เพื่อล็อกว่าโปรเจกต์ใช้ package ตัวไหนแน่นอน ป้องกันไม่ให้ package ปลอมหลุดเข้ามาแทนที่ในภายหลัง
4. **ใช้เครื่องมือสแกนความปลอดภัยของ dependency** เช่น `npm audit`, Snyk, Socket.dev ซึ่งมีฐานข้อมูล package ที่รู้ว่าเป็นอันตราย และจะแจ้งเตือนทันทีถ้าพบชื่อที่น่าสงสัย
5. **Registry ที่มีชื่อเสียงมีระบบตรวจจับอัตโนมัติ** — ทั้ง npm และ PyPI มีระบบสแกนอัตโนมัติที่คอยตรวจจับชื่อ package ที่คล้ายกับ package ยอดนิยมและถอดออกจาก registry เมื่อพบพฤติกรรมที่เป็นอันตราย แต่กว่าจะถูกถอดออกก็อาจมีคนติดตั้งไปแล้วจำนวนหนึ่งในช่วงเวลาสั้น ๆ ก่อนถูกจับได้ ดังนั้นการป้องกันในฝั่งของทีมพัฒนาเองยังจำเป็นเสมอ

---

## Step 794: Software Bill of Materials (SBOM) คืออะไร

### นิยาม

**SBOM (Software Bill of Materials)** คือ **รายการส่วนประกอบซอฟต์แวร์แบบละเอียดและเป็นทางการ** ที่ระบุว่าซอฟต์แวร์หนึ่งตัวประกอบขึ้นจาก library, module, dependency อะไรบ้าง ทั้งที่เป็น **direct dependency** (ที่โปรเจกต์เรียกใช้ตรง ๆ) และ **transitive dependency** (dependency ของ dependency อีกที ซึ่งบางครั้งลึกลงไปหลายชั้น) พร้อมข้อมูลกำกับแต่ละตัว เช่น ชื่อ, เวอร์ชัน, ผู้ผลิต/แหล่งที่มา, ใบอนุญาต (license), และ checksum/hash สำหรับยืนยันความถูกต้อง

### เปรียบเทียบให้เข้าใจง่าย: ฉลากส่วนประกอบอาหาร

ลองนึกถึง **ฉลากโภชนาการบนกล่องอาหารสำเร็จรูป** ที่ระบุส่วนประกอบทุกอย่างอย่างละเอียด รวมถึงสารก่อภูมิแพ้ (allergen) เช่น "มีนม มีถั่ว มีกลูเตน" — ถ้าคุณแพ้ถั่ว คุณสามารถอ่านฉลากแล้วรู้ทันทีว่ากินอาหารกล่องนี้ได้หรือไม่ โดยไม่ต้องโทรถามโรงงานผู้ผลิต

**SBOM ทำหน้าที่เดียวกันกับซอฟต์แวร์** — เมื่อมีช่องโหว่ความปลอดภัยร้ายแรงถูกประกาศออกมาสำหรับ library ตัวหนึ่ง (เช่น "สารก่อภูมิแพ้" ในโลกซอฟต์แวร์) องค์กรที่มี SBOM ครบถ้วนของทุกระบบสามารถ **ค้นหาได้ทันทีว่ามีระบบไหนบ้างที่ใช้ library ตัวนั้นอยู่** โดยไม่ต้องไล่เปิดโค้ดทีละโปรเจกต์

### ทำไม SBOM ถึงสำคัญ — กรณีศึกษา Log4Shell

ตัวอย่างที่ชัดเจนที่สุดคือเหตุการณ์ **Log4Shell (CVE-2021-44228)** ในเดือนธันวาคม 2021 ซึ่งเป็นช่องโหว่ระดับวิกฤต (Remote Code Execution) ใน **Apache Log4j** ไลบรารีสำหรับทำ logging ในภาษา Java ที่ถูกใช้งานเป็น **transitive dependency** ซ่อนอยู่ลึกในระบบนับล้านทั่วโลก แม้แต่ทีมที่ไม่เคยเรียกใช้ Log4j โดยตรงเลยก็อาจได้รับผลกระทบเพราะ library อื่นที่พวกเขาใช้ดันไปพึ่งพา Log4j อีกที

องค์กรที่ **ไม่มี SBOM** ต้องเสียเวลาหลายวันถึงหลายสัปดาห์ในการไล่ตรวจสอบโค้ดทุกโปรเจกต์ด้วยมือว่ามีการใช้ Log4j อยู่ที่ไหนบ้าง ในขณะที่องค์กรที่ **มี SBOM ที่อัปเดตอยู่เสมอ** สามารถค้นหาคำตอบได้ภายในไม่กี่นาทีด้วยการ query ข้อมูลที่มีอยู่แล้ว

### องค์ประกอบขั้นต่ำของ SBOM (NTIA Minimum Elements)

หน่วยงาน NTIA (National Telecommunications and Information Administration) ของสหรัฐฯ กำหนดองค์ประกอบขั้นต่ำที่ SBOM ควรมี ได้แก่:

| องค์ประกอบ | ความหมาย |
|---|---|
| **Supplier Name** | ผู้ผลิต/ผู้เผยแพร่ component นั้น |
| **Component Name** | ชื่อของ component/library |
| **Version** | เวอร์ชันที่ใช้งานอยู่จริง |
| **Unique Identifiers** | ตัวระบุที่ไม่ซ้ำกัน เช่น PURL (Package URL), CPE |
| **Dependency Relationship** | ความสัมพันธ์ว่า component ไหนพึ่งพา component ไหน (direct/transitive) |
| **Author of SBOM Data** | ผู้ที่สร้าง SBOM ฉบับนี้ |
| **Timestamp** | เวลาที่สร้าง SBOM ฉบับนี้ |

### แรงผลักดันเชิงนโยบาย

ดังที่กล่าวใน Step 791 **Executive Order 14028** ของรัฐบาลสหรัฐฯ (พฤษภาคม 2021) ซึ่งเป็นผลพวงส่วนหนึ่งจากเหตุการณ์ SolarWinds ได้กำหนดให้หน่วยงานรัฐบาลกลางสหรัฐฯ ต้องเรียกร้อง SBOM จากผู้ขายซอฟต์แวร์ทุกรายที่ต้องการทำธุรกิจกับภาครัฐ ทำให้ SBOM เปลี่ยนจาก "แนวคิดที่ดี" กลายเป็น **ข้อกำหนดทางกฎหมาย/สัญญาที่จับต้องได้จริง** ในหลายอุตสาหกรรมทั่วโลกตามมา

---

## Step 795: เครื่องมือสร้าง SBOM — Syft, CycloneDX, SPDX

### มาตรฐานรูปแบบ SBOM ที่ใช้กันแพร่หลาย

SBOM ต้องเป็น **machine-readable** คือคอมพิวเตอร์อ่านและประมวลผลต่อได้อัตโนมัติ ไม่ใช่แค่เอกสาร Word ธรรมดา ปัจจุบันมีมาตรฐานหลักอยู่ 2 แบบที่ได้รับความนิยมสูงสุด:

| มาตรฐาน | ผู้ดูแล | จุดเด่น |
|---|---|---|
| **CycloneDX** | โปรเจกต์ของ OWASP | ออกแบบมาเน้นเรื่องความปลอดภัยโดยเฉพาะ รองรับ VEX (Vulnerability Exploitability eXchange) สำหรับระบุว่าช่องโหว่หนึ่งกระทบระบบจริงหรือไม่ เบาและง่ายต่อการ integrate เข้ากับ pipeline ด้านความปลอดภัย |
| **SPDX (Software Package Data Exchange)** | เดิมเป็นโปรเจกต์ของ Linux Foundation ปัจจุบันเป็นมาตรฐานสากล **ISO/IEC 5962:2021** | เริ่มต้นเน้นเรื่อง license compliance เป็นหลัก ปัจจุบันรองรับข้อมูลความปลอดภัยด้วยเช่นกัน ได้รับการยอมรับเป็นมาตรฐาน ISO ทำให้เหมาะกับงานด้าน compliance/audit ระดับองค์กรและราชการ |

ทั้งสองมาตรฐานรองรับ format แบบ JSON และ XML (SPDX ยังรองรับ tag-value format แบบข้อความธรรมดาด้วย) และเครื่องมือสมัยใหม่ส่วนใหญ่รองรับทั้งสองแบบ

### Syft — เครื่องมือสร้าง SBOM ยอดนิยม

**Syft** เป็นเครื่องมือโอเพนซอร์สที่พัฒนาโดยบริษัท **Anchore** ใช้สำหรับสแกนและสร้าง SBOM จากแหล่งข้อมูลได้หลากหลายรูปแบบ ทั้ง:

- โฟลเดอร์ source code ในเครื่อง
- Container image (Docker/OCI image)
- ไฟล์ archive (tar, zip)
- Git repository

**ตัวอย่างการติดตั้ง (Linux/macOS):**

```bash
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
```

**ตัวอย่างการใช้งาน:**

```bash
# สร้าง SBOM จากโฟลเดอร์โปรเจกต์ปัจจุบัน ในรูปแบบ CycloneDX JSON
syft dir:. -o cyclonedx-json=sbom.cdx.json

# สร้าง SBOM จาก Docker image ในรูปแบบ SPDX JSON
syft packages myapp:latest -o spdx-json=sbom.spdx.json

# ดูผลลัพธ์แบบตารางอ่านง่ายบนหน้าจอโดยไม่ต้องเขียนไฟล์
syft dir:.
```

ผลลัพธ์ที่ได้จะแสดงรายการ package ทั้งหมดที่ syft ตรวจพบ พร้อมเวอร์ชัน ชนิดของ package (npm, pip, gem, apk, deb, ฯลฯ) และ location ที่พบใน filesystem

### เครื่องมืออื่น ๆ ที่ควรรู้จัก

| เครื่องมือ | จุดเด่น |
|---|---|
| **Trivy** (โดย Aqua Security) | นอกจากสร้าง SBOM ได้แล้วยังสแกนหาช่องโหว่ (CVE) ในตัวเดียวกัน นิยมใช้ในงาน container security |
| **cdxgen** | เครื่องมือสร้าง CycloneDX SBOM ที่รองรับ ecosystem หลากหลายภาษามาก |
| **GitHub Dependency Graph → Export SBOM** | GitHub มีฟีเจอร์ในตัวให้ export SBOM ของ repository เป็นไฟล์ SPDX ได้โดยตรงจากหน้า Insights → Dependency graph โดยไม่ต้องติดตั้งเครื่องมือเพิ่มเติมเลย |

### แนวทางปฏิบัติที่ดี

1. **สร้าง SBOM เป็นส่วนหนึ่งของ CI/CD pipeline** ทุกครั้งที่ build เพื่อให้ SBOM สะท้อนสถานะปัจจุบันของซอฟต์แวร์เสมอ ไม่ใช่ทำครั้งเดียวแล้วปล่อยให้ล้าสมัย
2. **เก็บ SBOM แนบไปกับ artifact ที่ release ทุกครั้ง** (เช่นแนบเป็นไฟล์ในหน้า GitHub Release)
3. **นำ SBOM ไปเชื่อมกับเครื่องมือสแกนช่องโหว่** เพื่อให้แจ้งเตือนอัตโนมัติเมื่อพบ CVE ใหม่ที่ตรงกับ component ใน SBOM

---

## Step 796: Dependency Pinning และ Lock File — สิ่งที่ห้ามลบทิ้งเด็ดขาด

### ปัญหาของ Version Range แบบหลวม ๆ

เมื่อคุณระบุ dependency ในไฟล์ manifest เช่น `package.json` คุณมักจะเห็นสัญลักษณ์แบบนี้:

```json
{
  "dependencies": {
    "express": "^4.18.0",
    "lodash": "~4.17.0"
  }
}
```

- `^4.18.0` หมายถึง "เวอร์ชันไหนก็ได้ตั้งแต่ 4.18.0 ไปจนถึงก่อน 5.0.0" (ยอมให้อัปเดต minor/patch อัตโนมัติ)
- `~4.17.0` หมายถึง "เวอร์ชันไหนก็ได้ตั้งแต่ 4.17.0 ไปจนถึงก่อน 4.18.0" (ยอมให้อัปเดตแค่ patch)

ปัญหาคือ **ช่วงเวอร์ชันแบบนี้ไม่ได้การันตีว่าทุกครั้งที่ `npm install` จะได้โค้ดชุดเดียวกันเป๊ะ ๆ** เพราะ maintainer ของ package สามารถ publish เวอร์ชัน patch ใหม่ได้ตลอดเวลา ถ้าวันนี้คุณ install ได้ `4.18.2` แต่พรุ่งนี้เพื่อนร่วมทีมอีกคน install แล้วดันได้ `4.18.5` เพราะมีการ publish เวอร์ชันใหม่ระหว่างนั้น — นี่คือปัญหา "works on my machine" แบบคลาสสิก และที่ร้ายแรงกว่านั้นในมุมความปลอดภัยคือ **ถ้าเวอร์ชัน patch ใหม่ที่ publish ออกมาดันมีมัลแวร์แฝงอยู่ (เช่นกรณี maintainer account ถูกยึด) ทุกคนที่ build โปรเจกต์ในช่วงเวลานั้นจะได้โค้ดร้ายไปโดยอัตโนมัติทันที โดยไม่มีใครต้องเปลี่ยนแปลงอะไรในโค้ดของตัวเองเลย**

### Lock File คือทางแก้

**Lock File** คือไฟล์ที่ package manager สร้างขึ้นเพื่อ **บันทึกเวอร์ชันที่ resolve ได้จริงแบบเจาะจง (exact version)** ของทุก dependency ทั้ง direct และ transitive พร้อม **integrity hash** (checksum เข้ารหัสของเนื้อหาไฟล์ package) เพื่อยืนยันว่าเนื้อหาที่ดาวน์โหลดมาไม่ถูกแก้ไขหรือปลอมแปลง

| ภาษา/Ecosystem | Lock File |
|---|---|
| **Node.js (npm)** | `package-lock.json` |
| **Node.js (Yarn)** | `yarn.lock` |
| **Node.js (pnpm)** | `pnpm-lock.yaml` |
| **Ruby (Bundler)** | `Gemfile.lock` |
| **Python (Poetry)** | `poetry.lock` |
| **Python (Pipenv)** | `Pipfile.lock` |
| **PHP (Composer)** | `composer.lock` |
| **Rust (Cargo)** | `Cargo.lock` |
| **Go** | `go.sum` (คู่กับ `go.mod`) |

ตัวอย่างส่วนหนึ่งของ `package-lock.json` ที่มี integrity hash:

```json
"node_modules/express": {
  "version": "4.18.2",
  "resolved": "https://registry.npmjs.org/express/-/express-4.18.2.tgz",
  "integrity": "sha512-5/PsL6iGPdfQ/lKM1UuielYgv3BUoJfz1aUwU9vHZ+J7gyvwdQXFEBIEIaxeGf0GIcreATNyBExtalisDbuMqQ=="
}
```

ถ้า package บน registry ถูกแก้ไขเนื้อหาโดยไม่เปลี่ยนเวอร์ชัน (ซึ่งเป็นสัญญาณของการโจมตี) hash ที่คำนวณได้จากไฟล์ที่ดาวน์โหลดมาจะไม่ตรงกับ integrity hash ใน lock file และการติดตั้งจะ **ล้มเหลวทันที** แทนที่จะติดตั้งโค้ดที่อาจถูกแทรกแซงไปโดยไม่รู้ตัว

### ทำไมห้ามลบ Lock File ทิ้งเด็ดขาด

บางครั้งนักพัฒนามือใหม่เจอปัญหา dependency แล้วลองลบ `package-lock.json` ทิ้งแล้ว `npm install` ใหม่เพื่อ "แก้ปัญหา" — นี่คือการกระทำที่ **อันตรายมาก** เพราะ:

1. คุณทำลายการรับประกันว่าทุกคนในทีม (และ CI/CD) จะได้ dependency tree แบบเดียวกันเป๊ะ
2. คุณเสี่ยงที่จะ resolve ไปเจอเวอร์ชันใหม่กว่าที่อาจมีช่องโหว่หรือมัลแวร์แฝงอยู่โดยไม่รู้ตัว
3. คุณสูญเสีย integrity hash ที่เคยตรวจสอบมาแล้ว
4. Diff ของ lock file บอกประวัติสำคัญว่า dependency ถูกเปลี่ยนแปลงเมื่อไหร่ อย่างไร ซึ่งมีประโยชน์มากตอนสืบสวนเหตุการณ์ความปลอดภัยย้อนหลัง

**หลักปฏิบัติที่ถูกต้อง:** Lock file ต้อง **commit เข้า Git repository เสมอ** ห้ามใส่ไว้ใน `.gitignore` และทุกครั้งที่มี Pull Request ที่แก้ไข lock file ทีมควรตรวจสอบ diff อย่างละเอียดว่ามีการเพิ่ม/เปลี่ยนเวอร์ชัน dependency อะไรบ้าง เพราะนี่คือจุดที่ dependency แปลกปลอมอาจแอบเข้ามาได้

### `npm ci` vs `npm install`

ใน CI/CD pipeline ควรใช้คำสั่ง `npm ci` แทน `npm install` เพราะ:

- `npm ci` จะติดตั้ง dependency **ตาม lock file แบบเป๊ะ ๆ เท่านั้น** ไม่พยายามคำนวณ resolve ใหม่
- ถ้า `package.json` กับ `package-lock.json` ไม่ตรงกัน คำสั่งจะ **ล้มเหลวทันที** แทนที่จะเงียบ ๆ แก้ไข lock file ให้เอง
- เร็วกว่า `npm install` เพราะข้ามขั้นตอนการคำนวณ dependency resolution ที่ซับซ้อน

---

## Step 797: Automated Dependency Updates ด้วย Dependabot และ Renovate

### ทำไมต้อง Automate การอัปเดต Dependency

การใช้ lock file ปักหมุดเวอร์ชันไว้แน่นอน (Step 796) ช่วยเรื่องความปลอดภัยได้ก็จริง แต่ก็มีข้อเสียคือ **ถ้าไม่มีใครคอยอัปเดตเลย โปรเจกต์จะค้างอยู่กับ dependency เวอร์ชันเก่าที่อาจมีช่องโหว่ที่ถูกค้นพบใหม่ในภายหลัง** การอัปเดต dependency ด้วยมือทีละตัวเป็นงานที่น่าเบื่อและถูกละเลยได้ง่ายมากในทีมที่งานยุ่ง จึงเป็นที่มาของเครื่องมือ automate การอัปเดต

### Dependabot

**Dependabot** เดิมเป็นบริษัท startup อิสระ ก่อนที่ **GitHub จะเข้าซื้อกิจการในปี 2019** และผนวกเข้าเป็นฟีเจอร์ในตัวของ GitHub โดยตรง ทำงานได้ 2 รูปแบบหลัก:

**1. Dependabot Version Updates** — เปิด Pull Request เพื่ออัปเดต dependency ให้เป็นเวอร์ชันใหม่ล่าสุดตามตารางเวลาที่กำหนด ต้องตั้งค่าผ่านไฟล์ `.github/dependabot.yml`

**2. Dependabot Security Updates / Alerts** — เมื่อ GitHub Advisory Database (ฐานข้อมูลช่องโหว่ที่ GitHub ดูแล) พบว่า dependency ที่โปรเจกต์ใช้อยู่มีช่องโหว่ที่ประกาศแล้ว ระบบจะ **เปิด Pull Request แก้ไขให้อัตโนมัติทันที** โดยไม่ต้องมีไฟล์ config เพิ่มเติม เพียงเปิดใช้งานผ่านเมนู Settings → Code security ของ repository

**ตัวอย่างไฟล์ `.github/dependabot.yml`:**

```yaml
version: 2
updates:
  # ตรวจสอบ dependency ของ npm ทุกสัปดาห์
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    open-pull-requests-limit: 10
    labels:
      - "dependencies"

  # ตรวจสอบ dependency ของ Python (pip) ทุกสัปดาห์
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"

  # ตรวจสอบ Dockerfile base image
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"

  # ตรวจสอบ GitHub Actions workflow เอง (ป้องกัน supply chain attack ผ่าน action ที่ใช้ใน CI)
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

สังเกตว่า Dependabot ไม่ได้ดูแลแค่ dependency ของโค้ดแอปพลิเคชันเท่านั้น แต่ยังครอบคลุมถึง **Dockerfile base image** และ **GitHub Actions ที่ใช้ใน workflow** ด้วย ซึ่งเป็นจุดที่มักถูกมองข้ามแต่ก็เป็นส่วนหนึ่งของ supply chain เช่นกัน

### Renovate

**Renovate** เป็นเครื่องมือโอเพนซอร์สที่พัฒนาโดยบริษัท **Mend** (เดิมชื่อ WhiteSource) มีจุดเด่นเหนือ Dependabot ในเรื่อง **ความยืดหยุ่นในการตั้งค่า** เช่น:

- รองรับ package ecosystem จำนวนมากกว่า
- สามารถ **จัดกลุ่ม (group)** การอัปเดตหลาย dependency เข้าเป็น PR เดียวเพื่อลดความรำคาญจาก PR จำนวนมาก
- ตั้งตารางเวลาที่ซับซ้อนได้ เช่น "อัปเดตเฉพาะ patch version อัตโนมัติทุกวัน แต่ major version ให้รอ review ก่อนเสมอ"
- ตั้งค่า **automerge** สำหรับการอัปเดตที่ความเสี่ยงต่ำ (เช่น patch version ที่ CI ผ่านทั้งหมด) ได้
- ใช้งานผ่าน GitHub App หรือ self-host เองก็ได้

ตัวอย่างไฟล์ `renovate.json` เบื้องต้น:

```json
{
  "extends": ["config:base"],
  "schedule": ["before 6am on monday"],
  "packageRules": [
    {
      "matchUpdateTypes": ["patch"],
      "automerge": true
    },
    {
      "matchUpdateTypes": ["major"],
      "automerge": false,
      "labels": ["needs-review"]
    }
  ]
}
```

### แนวทางปฏิบัติที่ดีเมื่อใช้เครื่องมืออัตโนมัติเหล่านี้

1. **อย่ากด merge PR ของ Dependabot/Renovate แบบไม่อ่านอะไรเลย** แม้จะเป็นเครื่องมือที่น่าเชื่อถือ แต่ก็ควรอ่าน changelog ของ dependency ที่อัปเดตทุกครั้ง โดยเฉพาะการอัปเดต major version ที่อาจมี breaking change
2. **ตั้งค่า branch protection ให้ PR เหล่านี้ต้องผ่าน CI/test ทั้งหมดก่อน merge ได้เสมอ**
3. **ใช้ automerge เฉพาะกับการอัปเดตความเสี่ยงต่ำ** เช่น patch version หรือ security patch ที่ทดสอบอัตโนมัติครบถ้วนแล้วเท่านั้น
4. **จัดกลุ่มการอัปเดตที่ไม่เร่งด่วน** เพื่อไม่ให้ทีม inbox ล้นไปด้วย PR จำนวนมากจนเกิดอาการ "PR fatigue" และเริ่ม merge แบบไม่ได้ตรวจสอบจริงจัง

---

## Step 798: Sigstore/cosign — การ Sign และ Verify Software Artifact แบบ Cryptographic

### ปัญหาของการเซ็นรับรองซอฟต์แวร์แบบดั้งเดิม

การเซ็นรับรอง (digital signing) เป็นวิธีมาตรฐานที่ใช้พิสูจน์ว่า software artifact (เช่น container image, binary, package) มาจากแหล่งที่เชื่อถือได้จริงและไม่ถูกแก้ไขระหว่างทาง แต่วิธีดั้งเดิมมีปัญหาสำคัญคือ **การจัดการ private key** — องค์กรต้องสร้าง เก็บรักษา หมุนเวียน (rotate) และป้องกันไม่ให้ private key รั่วไหลตลอดอายุการใช้งาน ซึ่งเป็นภาระที่หนักมากและเป็นความเสี่ยงในตัวมันเอง (ถ้า private key หลุด ผู้โจมตีก็สามารถเซ็นโค้ดร้ายให้ดูน่าเชื่อถือได้ทันที — คล้ายกับสิ่งที่อาจเกิดขึ้นได้ในกรณีแบบ SolarWinds)

### Sigstore คืออะไร

**Sigstore** เป็นโครงการโอเพนซอร์สภายใต้การดูแลของ Linux Foundation และ OpenSSF ที่แก้ปัญหานี้ด้วยแนวคิด **"Keyless Signing"** คือ **ไม่ต้องมีใครถือ private key ระยะยาวเลย** โดยประกอบด้วย 3 ส่วนหลัก:

| ส่วนประกอบ | หน้าที่ |
|---|---|
| **Fulcio** | Certificate Authority (CA) ที่ออกใบรับรอง x.509 แบบ **อายุสั้นมาก (ประมาณ 10 นาที)** โดยผูกใบรับรองนั้นเข้ากับตัวตนที่ยืนยันผ่าน OIDC (เช่น การ login ด้วยบัญชี GitHub, Google) แทนที่จะออกใบรับรองอายุยืนแบบดั้งเดิม |
| **Rekor** | Transparency Log สาธารณะที่บันทึกทุกเหตุการณ์การเซ็นรับรองแบบเปลี่ยนแปลงแก้ไขไม่ได้ (tamper-resistant) แนวคิดคล้ายกับ Certificate Transparency Log ที่ใช้ในวงการ TLS/HTTPS |
| **cosign** | เครื่องมือ command-line ที่นักพัฒนาและ CI/CD pipeline ใช้จริงในการ sign และ verify artifact (โดยเฉพาะ container image เป็นหลัก แต่ก็รองรับไฟล์ทั่วไปได้ด้วย) |

### ขั้นตอนการทำงาน (Keyless Flow)

```
1. Developer/CI รันคำสั่ง cosign sign
        │
        ▼
2. ยืนยันตัวตนผ่าน OIDC (เช่น GitHub Actions ออก OIDC token ให้อัตโนมัติ)
        │
        ▼
3. Fulcio ออกใบรับรองอายุสั้น (~10 นาที) ที่ผูกกับตัวตนนั้น พร้อมสร้างคู่กุญแจชั่วคราว (ephemeral keypair)
        │
        ▼
4. cosign ใช้กุญแจชั่วคราวเซ็นรับรอง digest ของ artifact
        │
        ▼
5. ลายเซ็นและใบรับรองถูกบันทึกลง Rekor (transparency log สาธารณะ)
        │
        ▼
6. กุญแจชั่วคราวถูกทิ้งทันที — ไม่มี private key ระยะยาวให้ต้องดูแลรักษาเลย
```

**ตัวอย่างคำสั่ง:**

```bash
# เซ็นรับรอง container image (ทำงานภายใน CI ที่รองรับ OIDC เช่น GitHub Actions)
cosign sign --yes ghcr.io/myorg/myapp:v1.2.3

# ตรวจสอบว่า image นี้ถูกเซ็นโดย workflow ที่คาดหวังจริงหรือไม่
cosign verify \
  --certificate-identity="https://github.com/myorg/myrepo/.github/workflows/release.yml@refs/heads/main" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  ghcr.io/myorg/myapp:v1.2.3
```

การ verify ไม่ได้แค่เช็คว่าลายเซ็นถูกต้องทางคณิตศาสตร์เท่านั้น แต่ยังยืนยันได้ว่า **"artifact นี้ถูกสร้างและเซ็นโดย workflow เส้นทางไหน จาก repository ไหนจริง ๆ"** ซึ่งเป็นการป้องกันที่ตรงจุดกว่าการเซ็นด้วย key ทั่วไปมาก

### การใช้งานจริงในวงการ

- **npm** เพิ่มฟีเจอร์ **provenance attestation** (ตั้งแต่ npm 9.5 ปี 2023) ที่ใช้ Sigstore เป็นรากฐาน ทำให้ package บน npm สามารถพิสูจน์ได้ว่าถูก build มาจาก source repository และ CI workflow ที่ระบุไว้จริง ผ่านคำสั่ง `npm publish --provenance`
- **PyPI** มีฟีเจอร์ Trusted Publishing ที่ทำงานบนหลักการคล้ายกัน
- ระบบนิเวศ **Kubernetes** และโปรเจกต์ CNCF จำนวนมากใช้ cosign เป็นมาตรฐานในการเซ็นและตรวจสอบ container image ก่อน deploy

---

## Step 799: SLSA Framework เบื้องต้น (Supply-chain Levels for Software Artifacts)

### SLSA คืออะไร

**SLSA** (อ่านออกเสียงว่า "salsa") ย่อมาจาก **Supply-chain Levels for Software Artifacts** เป็น **กรอบการทำงาน (framework)** ที่กำหนดชุดมาตรฐานและ checklist สำหรับเพิ่มความน่าเชื่อถือของกระบวนการ build ซอฟต์แวร์ เพื่อป้องกันการถูกแทรกแซง (tampering) ตลอดทั้งห่วงโซ่การผลิต

SLSA เริ่มต้นเป็นกรอบการทำงานภายในของ Google ก่อนจะถูกบริจาคให้กับ **OpenSSF (Open Source Security Foundation)** และพัฒนาต่อจนออก **เวอร์ชัน 1.0 ในปี 2023** ซึ่งเป็นเวอร์ชันที่ถูกปรับให้เรียบง่ายและใช้งานได้จริงมากขึ้นกว่าฉบับร่างแรก ๆ

### แนวคิดหลัก: Provenance

หัวใจของ SLSA คือแนวคิดเรื่อง **Provenance** ซึ่งหมายถึง **เอกสารข้อมูล metadata แบบที่คอมพิวเตอร์อ่านได้** ที่อธิบายอย่างละเอียดว่า artifact ตัวหนึ่งถูกสร้างขึ้นมาอย่างไร เช่น:

- มาจาก source code repository ไหน คอมมิตอะไร
- ใช้ build system/pipeline ตัวไหนเป็นคนสร้าง
- ใช้คำสั่ง build อะไรบ้าง
- ใช้ dependency อะไรเป็น input

Provenance นี้มักถูกเข้ารหัสในรูปแบบมาตรฐาน **in-toto attestation** และเพื่อให้ provenance นี้น่าเชื่อถือจริง มันมักถูก **เซ็นรับรองด้วย Sigstore/cosign** (เชื่อมโยงกับ Step 798 โดยตรง) — จะเห็นว่า SBOM, Sigstore และ SLSA ทำงานเสริมกันเป็นระบบนิเวศเดียวกัน ไม่ใช่เครื่องมือแยกจากกัน

### ระดับความปลอดภัยของ SLSA (Build Track, เวอร์ชัน 1.0)

| ระดับ | ข้อกำหนดหลัก |
|---|---|
| **Level 0** | ไม่มีการรับประกันใด ๆ (สถานะเริ่มต้นของโปรเจกต์ส่วนใหญ่ที่ยังไม่ได้ทำอะไรเลย) |
| **Level 1** | มีการสร้าง **provenance** ที่บันทึกว่า build เกิดขึ้นอย่างไร แม้ provenance นี้จะยังปลอมแปลงได้อยู่ถ้าคนมีสิทธิ์เข้าถึง build system เต็มที่ — แต่อย่างน้อยก็มีความโปร่งใสระดับพื้นฐาน |
| **Level 2** | Build ถูกรันบน **hosted build platform** (เช่น GitHub Actions, GitLab CI ที่เป็น managed service) ซึ่งสร้าง provenance ที่ **เซ็นรับรองแล้ว** และมีการป้องกันการปลอมแปลง provenance ในระดับหนึ่ง |
| **Level 3** | Build platform ถูก **hardened (แข็งแกร่งขึ้น)** อย่างจริงจัง มีการแยก (isolate) แต่ละ build ออกจากกันอย่างเข้มงวด ป้องกันไม่ให้ build หนึ่งไปแทรกแซงหรือขโมยข้อมูลจาก build อื่นได้ และ provenance ที่ได้ถือว่า **ไม่สามารถปลอมแปลงได้ (non-falsifiable)** จากมุมมองของผู้ดูแล pipeline เอง |

### ทำไม SLSA ถึงสำคัญ

ย้อนกลับไปที่กรณี **SolarWinds** ใน Step 791 — ต้นตอของปัญหาคือผู้โจมตีสามารถเข้าไปแทรกแซง **build environment** ได้โดยที่ไม่มีใครตรวจจับได้เลย เพราะไม่มีกลไกใดคอยยืนยันว่ากระบวนการ build นั้น "สะอาด" และตรงตามที่คาดหวังจริง ๆ **SLSA คือกรอบการทำงานที่ถูกออกแบบมาโดยตรงเพื่อป้องกันรูปแบบการโจมตีแบบนี้** โดยเน้นไปที่การทำให้ **ตัวกระบวนการ build เองมีความน่าเชื่อถือและตรวจสอบได้** ไม่ใช่แค่ตรวจสอบความปลอดภัยของโค้ดปลายทางเพียงอย่างเดียว

### การนำ SLSA มาใช้จริงใน GitHub Actions

GitHub มี action สำเร็จรูปชื่อ `actions/attest-build-provenance` ที่ช่วยสร้างและเซ็นรับรอง SLSA provenance ให้กับ artifact ที่ build ออกมาจาก workflow ได้โดยตรง ทำให้ทีมทั่วไปสามารถเริ่มต้นทำตาม SLSA ได้โดยไม่ต้องสร้างระบบซับซ้อนขึ้นมาเอง

สิ่งสำคัญที่ต้องเข้าใจคือ **SLSA ไม่ใช่หน่วยงานรับรอง (certification body)** ที่จะมาตรวจสอบและออกใบรับรองให้ แต่เป็น **กรอบการประเมินตัวเอง (self-assessment framework)** ที่องค์กรใช้เป็นแนวทางในการพัฒนากระบวนการ build ของตัวเองให้ปลอดภัยขึ้นทีละระดับ

---

## Step 800: แบบฝึกหัด — เปิดใช้ Dependabot จริง และสร้าง SBOM เบื้องต้น

ถึงเวลาลงมือปฏิบัติจริงกับความรู้ทั้งหมดที่เรียนมาใน Part นี้ แบบฝึกหัดนี้แบ่งเป็น 2 ส่วน

### ส่วนที่ 1: เปิดใช้งาน Dependabot ให้โปรเจกต์จริง

**ขั้นตอนที่ 1 — สร้างโฟลเดอร์ config**

ในโปรเจกต์ Git ของคุณ (ใช้โปรเจกต์ฝึกฝนจาก `~/git-course` ที่เตรียมไว้ตั้งแต่ Part 01 ก็ได้ หรือ repository จริงที่คุณดูแลอยู่) สร้างโฟลเดอร์ `.github` ที่ root ของ repository ถ้ายังไม่มี:

```bash
mkdir -p .github
```

**ขั้นตอนที่ 2 — เขียนไฟล์ `dependabot.yml`**

สร้างไฟล์ `.github/dependabot.yml` โดยปรับ `package-ecosystem` ให้ตรงกับภาษาที่โปรเจกต์คุณใช้จริง ตัวอย่างสำหรับโปรเจกต์ Node.js:

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
    labels:
      - "dependencies"
    commit-message:
      prefix: "chore(deps)"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

**ขั้นตอนที่ 3 — Commit และ Push**

```bash
git add .github/dependabot.yml
git commit -m "chore: enable Dependabot version updates"
git push
```

**ขั้นตอนที่ 4 — เปิดใช้ Security Updates**

ไปที่หน้า repository บน GitHub → **Settings → Code security** แล้วเปิดใช้งาน:

- **Dependency graph** (ต้องเปิดก่อนเป็นอันดับแรก เพราะฟีเจอร์อื่นต้องพึ่งพามัน)
- **Dependabot alerts**
- **Dependabot security updates**

**ขั้นตอนที่ 5 — สังเกตผลลัพธ์**

รอจนถึงรอบตารางเวลาที่ตั้งไว้ (หรือกด "Check for updates" ด้วยตนเองผ่านหน้า Insights → Dependency graph → Dependabot) แล้วสังเกตว่า Dependabot สร้าง Pull Request ขึ้นมาให้อัตโนมัติหรือไม่ ลองเปิดอ่าน PR ที่มันสร้าง สังเกตรายละเอียดที่มันแสดงให้ เช่น changelog ของ dependency ที่อัปเดต, release notes, และผลการรัน CI บน PR นั้น

### ส่วนที่ 2: สร้าง SBOM เบื้องต้นด้วย Syft

**ขั้นตอนที่ 1 — ติดตั้ง Syft**

```bash
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin

# ตรวจสอบว่าติดตั้งสำเร็จ
syft version
```

**ขั้นตอนที่ 2 — สร้าง SBOM จากโปรเจกต์ของคุณ**

```bash
cd ~/git-course/part-80-supply-chain   # หรือโฟลเดอร์โปรเจกต์จริงของคุณ
syft dir:. -o cyclonedx-json=sbom.cdx.json
```

**ขั้นตอนที่ 3 — เปิดดูผลลัพธ์**

เปิดไฟล์ `sbom.cdx.json` ขึ้นมาดู สังเกตโครงสร้างข้อมูล จะพบว่ามี array ของ `components` ที่แต่ละตัวมีข้อมูล `name`, `version`, `purl` (Package URL), และ `type` กำกับไว้อย่างละเอียด

**ขั้นตอนที่ 4 — ลองสร้างในรูปแบบ SPDX ด้วย**

```bash
syft dir:. -o spdx-json=sbom.spdx.json
```

เปรียบเทียบโครงสร้างของทั้งสองไฟล์ สังเกตว่าข้อมูล component ที่ตรวจพบเหมือนกัน แต่รูปแบบ (schema) การจัดเก็บต่างกันตามมาตรฐานของแต่ละฝั่ง

**ขั้นตอนที่ 5 — ทดสอบค้นหาช่องโหว่จาก SBOM (ต่อยอด)**

ถ้าต้องการต่อยอดไปอีกขั้น ลองใช้ Trivy ตรวจสอบ SBOM ที่สร้างไว้เพื่อหาช่องโหว่ที่รู้จักแล้ว (known CVE) ในรายการ dependency ที่ syft ตรวจพบ:

```bash
trivy sbom sbom.cdx.json
```

### Checklist แบบฝึกหัด Step 800

- [ ] สร้างไฟล์ `.github/dependabot.yml` และ push ขึ้น repository จริงแล้ว
- [ ] เปิดใช้งาน Dependency graph, Dependabot alerts และ Dependabot security updates ใน Settings ของ repository
- [ ] เห็น Pull Request ที่ Dependabot สร้างขึ้นมาอัตโนมัติ (หรืออย่างน้อยเข้าใจว่ามันจะปรากฏที่ไหนเมื่อถึงเวลา)
- [ ] ติดตั้ง Syft สำเร็จบนเครื่อง
- [ ] สร้าง SBOM ของโปรเจกต์ตัวเองได้ทั้งในรูปแบบ CycloneDX และ SPDX
- [ ] เปิดดูและเข้าใจโครงสร้างข้อมูลภายในไฟล์ SBOM ที่สร้างขึ้น

---

## สรุป Part 80

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Software Supply Chain** ประกอบด้วยทุกจุดตั้งแต่ source code, dependency, build tool, CI/CD pipeline ไปจนถึงการ distribute ซอฟต์แวร์ และทุกจุดคือพื้นผิวการโจมตีที่แฮกเกอร์เล็งเป้า — กรณีศึกษา **SolarWinds (2020)** แสดงให้เห็นว่าแม้แต่ซอฟต์แวร์ที่เซ็นรับรองถูกต้องตามกฎหมายก็ยังถูกแทรกแซงได้ถ้า build pipeline ถูกเจาะ
2. **Dependency Confusion** อาศัยพฤติกรรม default ของ package manager ที่เลือกเวอร์ชันสูงสุดข้าม registry โดยไม่สนใจแหล่งที่มา ทำให้แฮกเกอร์แอบอัปโหลด package ปลอมชื่อเดียวกับ internal package ขึ้น public registry ได้สำเร็จ
3. **Typosquatting** คือการตั้งชื่อ package ให้คล้ายของจริงมากจนคนพิมพ์ผิดแล้วติดตั้งมัลแวร์ไปโดยไม่รู้ตัว (เช่น `1odash` เลียนแบบ `lodash`)
4. **SBOM** คือรายการส่วนประกอบซอฟต์แวร์แบบละเอียด เปรียบเสมือนฉลากส่วนประกอบอาหาร ช่วยให้ตอบคำถามได้ทันทีว่า "เราใช้ library ที่มีช่องโหว่ตัวนี้อยู่ที่ไหนบ้าง" อย่างในกรณี Log4Shell
5. **Syft** เป็นเครื่องมือหลักในการสร้าง SBOM รองรับทั้งมาตรฐาน **CycloneDX** (เน้นความปลอดภัย) และ **SPDX** (มาตรฐาน ISO เน้น compliance)
6. **Lock File** (`package-lock.json`, `Gemfile.lock`, `poetry.lock` ฯลฯ) ล็อกเวอร์ชัน dependency แบบเจาะจงพร้อม integrity hash — ห้ามลบทิ้งเด็ดขาด เพราะเป็นเกราะป้องกันสำคัญไม่ให้ build ดึงโค้ดที่ถูกแก้ไขระหว่างทางมาใช้โดยไม่รู้ตัว
7. **Dependabot** (ในตัว GitHub) และ **Renovate** (ยืดหยุ่นกว่า) ช่วย automate การอัปเดต dependency ทั้งแบบตามตารางเวลาปกติและแบบ security patch ฉุกเฉิน
8. **Sigstore/cosign** แก้ปัญหาการจัดการ private key แบบดั้งเดิมด้วยแนวคิด keyless signing ผ่าน Fulcio (CA อายุสั้น), Rekor (transparency log) และ cosign (เครื่องมือ sign/verify)
9. **SLSA Framework** เป็นกรอบการทำงานที่กำหนดระดับความน่าเชื่อถือของกระบวนการ build (Level 0–3) โดยอาศัย provenance ที่มักถูกเซ็นรับรองผ่าน Sigstore เพื่อป้องกันการโจมตีแบบ SolarWinds ไม่ให้เกิดซ้ำ
10. ลงมือปฏิบัติจริงด้วยการเปิดใช้ **Dependabot** ผ่านไฟล์ `.github/dependabot.yml` และสร้าง **SBOM** เบื้องต้นด้วย **Syft** กับโปรเจกต์จริงของตัวเอง

### Checklist ก่อนไป Part 81

- [ ] เข้าใจว่า Software Supply Chain คืออะไร และอธิบายกรณีศึกษา SolarWinds ได้
- [ ] แยกความแตกต่างระหว่าง Dependency Confusion กับ Typosquatting ได้ชัดเจน
- [ ] เข้าใจว่า SBOM คืออะไร และทำไมมันถึงสำคัญเวลาเกิดเหตุการณ์แบบ Log4Shell
- [ ] สร้าง SBOM ด้วย Syft ได้จริงทั้งแบบ CycloneDX และ SPDX
- [ ] เข้าใจว่าทำไม Lock File ห้ามลบทิ้ง และรู้จัก Lock File ของภาษาหลัก ๆ
- [ ] ตั้งค่า Dependabot ผ่าน `.github/dependabot.yml` ได้จริง และรู้จัก Renovate เป็นทางเลือก
- [ ] เข้าใจแนวคิด Keyless Signing ของ Sigstore/cosign และบทบาทของ Fulcio กับ Rekor
- [ ] เข้าใจภาพรวมของ SLSA Framework และความเชื่อมโยงกับ Sigstore

---

**ต่อไป:** [Part 81: Compliance และ Audit Trail ด้วย Git](./part-081-compliance-audit-trail.md)
