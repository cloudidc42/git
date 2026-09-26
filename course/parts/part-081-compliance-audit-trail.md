# Part 81: Compliance และ Audit Trail ด้วย Git

> **Step ในหลักสูตรนี้:** Step 801–810
> **เฟส:** 8 — DevOps, Security, Compliance ระดับองค์กร
> **เป้าหมายของ Part นี้:** เข้าใจว่า "Compliance" ในบริบทซอฟต์แวร์คืออะไร ทำไม Git จึงเป็นเครื่องมือ audit trail ที่ทรงพลังโดยธรรมชาติ รู้จัก audit log ระดับ organization ของ GitHub/GitLab วิธีเก็บหลักฐานสำหรับผู้ตรวจสอบ (auditor) หลักการ Separation of Duties เหตุผลที่ต้องรักษา immutable history อย่างเข้มงวด นโยบายการเก็บรักษาข้อมูล (data retention) การป้องกันไม่ให้ข้อมูลส่วนบุคคล (PII) หลุดเข้าไปในโค้ดหรือ commit message และแนวคิด Compliance-as-Code ที่เปลี่ยนงานตรวจสอบด้วยมือให้กลายเป็นการตรวจสอบอัตโนมัติในไปป์ไลน์ เพื่อให้คุณสามารถออกแบบระบบ Git ขององค์กรให้ผ่านการตรวจสอบ compliance ระดับมืออาชีพได้จริง

---

## สารบัญของ Part นี้

- Step 801: Compliance คืออะไรในบริบทซอฟต์แวร์
- Step 802: ทำไม Git เป็นเครื่องมือ Audit Trail ที่ดีโดยธรรมชาติ
- Step 803: Audit Log ของ GitHub/GitLab Organization
- Step 804: การเก็บหลักฐานสำหรับ Compliance Audit
- Step 805: Separation of Duties — แยกคนเขียนโค้ดกับคนอนุมัติ merge
- Step 806: Immutable History และการจำกัด Force-Push ในสภาพแวดล้อม Regulated
- Step 807: Data Retention Policy สำหรับ Repository และ CI/CD Logs
- Step 808: การจัดการ PII ไม่ให้หลุดเข้า Commit Message หรือโค้ด
- Step 809: Compliance-as-Code — เขียนกฎ Compliance เป็น Policy Code
- Step 810: แบบฝึกหัด — ออกแบบ Audit Trail Checklist สำหรับองค์กรสมมติ

---

## Step 801: Compliance คืออะไรในบริบทซอฟต์แวร์

### นิยามของ Compliance

**Compliance** ในบริบทซอฟต์แวร์และองค์กร หมายถึง **การปฏิบัติตามกฎ ระเบียบ มาตรฐาน หรือข้อกำหนดที่ถูกกำหนดขึ้น** ไม่ว่าจะเป็นข้อกำหนดทางกฎหมาย (เช่น กฎหมายคุ้มครองข้อมูลส่วนบุคคล) ข้อกำหนดของอุตสาหกรรม (เช่น มาตรฐานความปลอดภัยของบัตรเครดิต) หรือมาตรฐานที่องค์กรเลือกทำเองเพื่อสร้างความน่าเชื่อถือให้กับลูกค้า

> **Compliance ไม่ใช่แค่ "เอกสารที่ต้องทำให้ครบ"** แต่เป็นกระบวนการที่พิสูจน์ได้ว่า **องค์กรควบคุมความเสี่ยง** ในเรื่องความปลอดภัยของข้อมูล ความถูกต้องของกระบวนการ และความรับผิดชอบต่อผู้มีส่วนได้ส่วนเสีย (stakeholder) ได้จริง

สิ่งที่ผู้ตรวจสอบ (auditor) ต้องการเห็นเสมอคือ **หลักฐาน (evidence)** ไม่ใช่แค่คำยืนยันปากเปล่าว่า "เราทำตามกระบวนการแล้ว" นี่คือเหตุผลที่ทีมพัฒนาซอฟต์แวร์ทุกทีมที่ทำงานกับองค์กรขนาดกลางถึงใหญ่ หรือทำงานกับลูกค้าองค์กร (enterprise customer) จำเป็นต้องเข้าใจเรื่อง compliance อย่างจริงจัง เพราะ **ระบบ version control ของคุณคือหนึ่งในแหล่งหลักฐานที่สำคัญที่สุด**

### ทำไมนักพัฒนาต้องสนใจเรื่อง Compliance

หลายคนคิดว่า compliance เป็นเรื่องของฝ่ายกฎหมายหรือฝ่าย security เท่านั้น แต่ในความเป็นจริง **พฤติกรรมของนักพัฒนาในการใช้ Git ทุกวันคือส่วนสำคัญของหลักฐาน compliance** เช่น:

- การที่ทุก merge เข้า branch หลักต้องผ่านการอนุมัติจากคนอื่น (ไม่ใช่ merge เอง) คือหลักฐานของ **Separation of Duties**
- การที่ commit ทุกตัวมีชื่อผู้เขียนและเวลาชัดเจน คือหลักฐานของ **Accountability**
- การที่ branch หลักไม่สามารถถูก force-push ทับประวัติได้ คือหลักฐานของ **Data Integrity**

ถ้าทีมของคุณตั้งค่า Git workflow ให้ถูกต้องตั้งแต่ต้น การผ่าน compliance audit จะกลายเป็นเรื่องที่ง่ายมาก เพราะหลักฐานเกิดขึ้นเองโดยอัตโนมัติจากการทำงานประจำวัน แต่ถ้าตั้งค่าไม่ถูกต้อง ทีมจะต้องเสียเวลามหาศาลในการ "สร้างหลักฐานย้อนหลัง" ซึ่งบางครั้งก็ทำไม่ได้เลยเพราะข้อมูลได้สูญหายไปแล้ว

### มาตรฐาน Compliance ที่พบบ่อยที่สุดในวงการซอฟต์แวร์

มาทำความรู้จักมาตรฐานหลัก ๆ แบบภาพรวม เพื่อให้เห็นว่า Git และ workflow ของทีมเข้าไปเกี่ยวข้องตรงไหนบ้าง

#### 1. SOC 2 (Service Organization Control 2)

- ออกโดย **AICPA (American Institute of Certified Public Accountants)** ของสหรัฐอเมริกา
- เน้นตรวจสอบ **Trust Service Criteria** 5 ด้าน ได้แก่ **Security, Availability, Processing Integrity, Confidentiality, Privacy**
- เป็นมาตรฐานที่ **บริษัท SaaS แทบทุกบริษัทต้องมี** เพื่อขายซอฟต์แวร์ให้กับลูกค้าองค์กรขนาดใหญ่ (enterprise customer มักถามหา SOC 2 Type II report ก่อนเซ็นสัญญาเสมอ)
- แบ่งเป็น **Type I** (ตรวจสอบว่าระบบควบคุม ณ จุดเวลาหนึ่งถูกออกแบบไว้ดีหรือไม่) และ **Type II** (ตรวจสอบว่าระบบควบคุมนั้น **ทำงานได้จริงอย่างต่อเนื่อง** เป็นระยะเวลาหนึ่ง เช่น 6-12 เดือน — Type II ยากกว่าและน่าเชื่อถือกว่ามาก)
- ในมุมของ Git: ผู้ตรวจสอบจะขอดู **audit log ของ repository, หลักฐานการอนุมัติ PR, การตั้งค่า branch protection** เพื่อยืนยันว่าไม่มีใครสามารถ push โค้ดที่ไม่ผ่านการตรวจสอบเข้า production ได้

#### 2. ISO/IEC 27001

- มาตรฐานสากลด้าน **Information Security Management System (ISMS)** ที่ออกโดยองค์กร ISO
- ครอบคลุมกว้างกว่า SOC 2 มาก เพราะมองทั้งองค์กร ไม่ใช่แค่ตัวผลิตภัณฑ์ซอฟต์แวร์
- มี **Annex A** ที่ระบุ control ย่อยจำนวนมาก (ในเวอร์ชัน 2022 มี 93 controls) ครอบคลุมตั้งแต่ access control, cryptography, change management ไปจนถึง supplier relationship
- ในมุมของ Git: control ที่เกี่ยวข้องโดยตรงคือ **Access Control (A.5.15-A.5.18)** และ **Secure Development (A.8.25-A.8.33)** ซึ่งครอบคลุมเรื่อง code review, การแยกสภาพแวดล้อม development/production และการจัดการ change control ผ่าน version control

#### 3. HIPAA (Health Insurance Portability and Accountability Act)

- กฎหมายของสหรัฐอเมริกาที่คุ้มครอง **ข้อมูลสุขภาพที่ระบุตัวตนได้ (PHI — Protected Health Information)**
- บังคับใช้กับองค์กรที่ทำงานด้าน healthcare หรือบริษัทซอฟต์แวร์ที่พัฒนาระบบให้กับองค์กรด้าน healthcare (เรียกว่า Business Associate)
- ในมุมของ Git: สิ่งที่อันตรายที่สุดคือ **ข้อมูลผู้ป่วยจริงหลุดเข้าไปใน test data, log file, หรือ commit message** เพราะ Git เก็บประวัติถาวร การลบไฟล์ออกจาก commit ล่าสุดไม่ได้ทำให้ข้อมูลหายไปจากประวัติ ต้องมีกระบวนการป้องกันตั้งแต่ต้นทาง (เราจะเจาะลึกใน Step 808)

#### 4. PCI-DSS (Payment Card Industry Data Security Standard)

- มาตรฐานที่กำหนดโดย **PCI Security Standards Council** (ก่อตั้งโดยบริษัทบัตรเครดิตรายใหญ่ เช่น Visa, Mastercard) สำหรับองค์กรที่จัดเก็บ ประมวลผล หรือส่งต่อข้อมูลบัตรเครดิต
- มีข้อกำหนดที่เข้มงวดมากเรื่อง **secure coding, code review, และ change management** เพราะช่องโหว่ในโค้ดที่จัดการข้อมูลบัตรเป็นความเสี่ยงสูงสุด
- Requirement 6 ของ PCI-DSS พูดถึง **Secure Software Development** โดยตรง กำหนดว่าต้องมีกระบวนการ review และ approve การเปลี่ยนแปลงโค้ดก่อนขึ้น production เสมอ — ซึ่งตรงกับสิ่งที่ branch protection และ mandatory PR review ใน Git ทำได้พอดี

### ตารางสรุปเปรียบเทียบ

| มาตรฐาน | ขอบเขต | ใครต้องทำ | จุดที่เกี่ยวข้องกับ Git โดยตรง |
|---|---|---|---|
| SOC 2 | บริการ/ผลิตภัณฑ์ SaaS | บริษัท SaaS ทุกขนาด | Audit log, PR approval, branch protection |
| ISO 27001 | องค์กรทั้งหมด | องค์กรทุกประเภทที่ต้องการมาตรฐานสากล | Access control, secure development lifecycle |
| HIPAA | ข้อมูลสุขภาพ (PHI) | องค์กร healthcare และ business associate | ป้องกัน PII/PHI หลุดเข้า repo, encryption |
| PCI-DSS | ข้อมูลบัตรเครดิต | องค์กรที่เกี่ยวข้องกับการชำระเงิน | Mandatory code review, change management |

สิ่งที่น่าสนใจคือ **ทุกมาตรฐานเหล่านี้ล้วนเรียกร้องสิ่งเดียวกันในระดับหนึ่ง**: ต้องมีการควบคุมการเปลี่ยนแปลงโค้ด (change control) ต้องมีการอนุมัติก่อนขึ้น production ต้องสามารถตรวจสอบย้อนหลังได้ว่าใครทำอะไร และต้องป้องกันข้อมูลอ่อนไหวไม่ให้หลุดออกไป — และนี่คือสิ่งที่ Git ถูกออกแบบมาให้ทำได้ดีอยู่แล้วโดยธรรมชาติ ซึ่งเราจะเจาะลึกใน Step ถัดไป

---

## Step 802: ทำไม Git เป็นเครื่องมือ Audit Trail ที่ดีโดยธรรมชาติ

### Audit Trail คืออะไร

**Audit Trail** (เส้นทางการตรวจสอบ) คือ **บันทึกลำดับเหตุการณ์ที่เกิดขึ้นในระบบ ซึ่งสามารถใช้สืบย้อนกลับไปดูได้ว่า "ใคร ทำอะไร เมื่อไหร่ และทำไม"** อย่างครบถ้วนและเชื่อถือได้

ระบบที่มี audit trail ที่ดีต้องตอบคำถามเหล่านี้ได้เสมอ:

1. **Who** — ใครเป็นคนทำการเปลี่ยนแปลงนี้
2. **What** — มีการเปลี่ยนแปลงอะไรเกิดขึ้นบ้าง
3. **When** — เกิดขึ้นเมื่อไหร่ (timestamp ที่แม่นยำ)
4. **Why** — เหตุผลของการเปลี่ยนแปลง (ถ้ามีการบันทึกไว้)
5. **Where** — เปลี่ยนแปลงที่ตำแหน่งไหนของระบบ

### เหตุผลที่ Git ตอบโจทย์ audit trail ได้ดีเป็นพิเศษ

Git ไม่ได้ถูกออกแบบมาเพื่อเป็นเครื่องมือ compliance โดยตรง แต่ **สถาปัตยกรรมพื้นฐานของ Git บังเอิญตอบโจทย์ audit trail ได้เกือบสมบูรณ์แบบ** ด้วยเหตุผลดังนี้:

#### 1. ทุก Commit มี Hash ที่ไม่ซ้ำกัน (Content-Addressable)

ดังที่เรียนไปแล้วใน Part ต้น ๆ ของหลักสูตรนี้ Git ใช้ SHA-1 (หรือ SHA-256 ในรุ่นใหม่) hash ในการอ้างอิงทุก object รวมถึง commit hash ของ commit ถูกคำนวณจาก **เนื้อหาของ snapshot + metadata ทั้งหมด** รวมถึง parent commit ก่อนหน้า

```
commit_hash = SHA(tree_hash + parent_hash + author + committer + timestamp + message)
```

นั่นหมายความว่า **ถ้าใครพยายามแก้ไขเนื้อหาของ commit ในอดีตแม้เพียงตัวอักษรเดียว hash ของ commit นั้นและทุก commit ที่ตามมาจะเปลี่ยนไปทั้งหมด** ทำให้การปลอมแปลงประวัติถูกตรวจจับได้ง่ายมาก (เทียบ hash ปัจจุบันกับ hash ที่เคยบันทึกไว้ที่อื่น เช่นใน backup หรือใน CI log)

#### 2. Author และ Committer ถูกบันทึกแยกกันเสมอ

Git บันทึก **สองบทบาท** ในทุก commit:

- **Author** — คนที่เขียนการเปลี่ยนแปลงนั้นจริง ๆ
- **Committer** — คนที่นำการเปลี่ยนแปลงนั้นเข้าสู่ประวัติ (อาจเป็นคนละคนกับ author เช่นในกรณี rebase, cherry-pick หรือมีคนอื่น apply patch ให้)

```bash
git show --format=fuller HEAD
```

```
commit a1b2c3d4e5f6...
Author:     Somchai Devteam <somchai@company.com>
AuthorDate: Wed Sep 20 14:32:10 2026 +0700
Commit:     Somchai Devteam <somchai@company.com>
CommitDate: Wed Sep 20 14:32:10 2026 +0700
```

ความแตกต่างระหว่างสองฟิลด์นี้เองที่บางครั้งกลายเป็นหลักฐานสำคัญ เช่น ถ้า commit ถูก apply เข้า branch หลักโดยระบบ CI/CD bot (committer เป็น bot) แต่ author เป็นนักพัฒนาจริง แสดงว่าผ่านกระบวนการอัตโนมัติที่ควบคุมได้ ไม่ใช่มีใครเข้าไปแก้ไขประวัติเอง

#### 3. Timestamp ที่ละเอียดและตรวจสอบไขว้ได้

ทุก commit มี timestamp ทั้งของ author และ committer พร้อม timezone offset ทำให้สามารถเรียงลำดับเหตุการณ์และตรวจสอบความสอดคล้องกับเหตุการณ์อื่น เช่น เวลาที่พนักงานลาออก เวลาที่เกิดเหตุการณ์ security incident หรือเวลาที่ deploy ขึ้น production ได้อย่างแม่นยำ

#### 4. ประวัติไม่สามารถแก้ไขได้ง่าย ๆ โดยไม่ทิ้งร่องรอย

การแก้ไขประวัติ Git (ผ่าน `rebase`, `commit --amend`, หรือ `filter-branch`) จะทำให้ **hash ของ commit เปลี่ยนไปเสมอ** ถ้า branch นั้นเคยถูก push ไปยัง remote แล้วและมีคนอื่น pull ไปใช้งาน การพยายาม force-push ประวัติใหม่ทับจะสร้างความขัดแย้ง (diverged history) ที่สังเกตเห็นได้ทันที และในระบบที่ตั้งค่าอย่างถูกต้อง (เช่นปิด force-push บน branch หลัก) การกระทำเช่นนี้จะถูกปฏิเสธโดยอัตโนมัติตั้งแต่แรก

#### 5. Commit Message เป็นบันทึกเหตุผลในตัวมันเอง

แม้ Git จะไม่บังคับรูปแบบ commit message แต่ทีมที่มีวินัยจะใช้ commit message เป็น **เอกสารประกอบการตัดสินใจ** เช่น อ้างอิง ticket number, เหตุผลของการเปลี่ยนแปลง, หรือแม้แต่ผลกระทบด้านความปลอดภัย — สิ่งเหล่านี้กลายเป็นหลักฐานที่ auditor อ่านและเข้าใจได้ทันทีโดยไม่ต้องถามใคร

### ข้อควรระวัง: Git เพียงอย่างเดียวไม่ใช่ Audit Trail ที่สมบูรณ์

สิ่งสำคัญที่ต้องเข้าใจให้ชัดคือ **Git object และ log ที่อยู่ในเครื่อง local ของนักพัฒนาแต่ละคนไม่ใช่แหล่งความจริงที่เชื่อถือได้เพียงแหล่งเดียว** เพราะ:

- นักพัฒนาสามารถแก้ไข local repository ของตัวเองได้อย่างอิสระ (rebase, amend, reset) ก่อนที่จะ push
- ข้อมูล metadata เช่น author name/email สามารถถูกตั้งค่าเป็นอะไรก็ได้ใน `git config` (ปลอมแปลงได้ถ้าไม่มีการยืนยันตัวตนเพิ่มเติม)

นี่คือเหตุผลที่ **หลักฐาน compliance ที่แท้จริงต้องมาจาก server-side ที่ควบคุมได้** เช่น audit log ของ GitHub/GitLab organization ที่บันทึกเหตุการณ์จากฝั่ง server ซึ่งผู้ใช้ทั่วไปแก้ไขไม่ได้ และการใช้ signed commit เพื่อยืนยันตัวตนของ author อย่างแท้จริง (จะเจาะลึกใน Step 803 และ 804)

---

## Step 803: Audit Log ของ GitHub/GitLab Organization

### ทำไม Audit Log ระดับ Organization ถึงสำคัญกว่า Git Log ธรรมดา

Git log ที่เราคุ้นเคย (`git log`) แสดงเฉพาะประวัติของ **commit** เท่านั้น แต่กิจกรรมจำนวนมากที่เกี่ยวข้องกับ compliance ไม่ได้อยู่ในรูปแบบ commit เลย เช่น:

- ใครเป็นคนเพิ่ม/ลบสมาชิกออกจากทีม
- ใครเปลี่ยนการตั้งค่า branch protection
- ใครเปลี่ยนสิทธิ์ของ repository จาก private เป็น public
- ใครดาวน์โหลด (clone) repository ที่มีข้อมูลอ่อนไหว
- ใครสร้าง/ลบ personal access token
- ใคร merge PR โดยไม่ผ่านการอนุมัติ (ในกรณีที่เป็น admin แล้ว bypass การตั้งค่า)

เหตุการณ์เหล่านี้ **ไม่ปรากฏใน `git log` เลย** เพราะมันเป็นเหตุการณ์ระดับแพลตฟอร์ม (platform-level event) ไม่ใช่ระดับ Git object ดังนั้นทั้ง GitHub และ GitLab จึงมีระบบ **Audit Log** แยกต่างหากที่บันทึกเหตุการณ์เหล่านี้ทั้งหมด

### GitHub Audit Log

GitHub มีระบบ **Audit log** ที่ระดับ **Organization** (และระดับ **Enterprise** สำหรับลูกค้าที่ใช้ GitHub Enterprise Cloud/Server) เข้าถึงได้ที่ `Settings > Audit log` ของ organization

ตัวอย่างประเภทเหตุการณ์ที่ถูกบันทึก:

| หมวดหมู่ | ตัวอย่าง Event |
|---|---|
| `repo` | `repo.create`, `repo.destroy`, `repo.access` (เปลี่ยน visibility) |
| `team` | `team.add_member`, `team.remove_member` |
| `org` | `org.add_member`, `org.remove_member`, `org.update_member` (เปลี่ยน role) |
| `protected_branch` | `protected_branch.create`, `protected_branch.update_required_status_check`, `protected_branch.policy_override` |
| `oauth_authorization` | สร้าง/เพิกถอน OAuth token |
| `git` | `git.clone`, `git.push` (เฉพาะ Enterprise ระดับสูงที่เปิด Git events) |

แต่ละ event จะมีรายละเอียด: **actor** (ใครทำ), **action** (ทำอะไร), **created_at** (เวลา), และบางครั้งมี **IP address** ที่ใช้ทำรายการด้วย

ตัวอย่างการดึงข้อมูล audit log ผ่าน GitHub REST API:

```bash
curl -H "Authorization: Bearer $TOKEN" \
  "https://api.github.com/orgs/my-company/audit-log?phrase=action:protected_branch.policy_override"
```

คำสั่งนี้จะกรองหาเฉพาะเหตุการณ์ที่มีคนใช้สิทธิ์ admin **bypass** การตั้งค่า branch protection — ซึ่งเป็นหนึ่งในเหตุการณ์ที่ auditor สนใจมากที่สุด เพราะมันคือจุดที่ control อาจถูกข้ามไป

### GitLab Audit Events

GitLab เรียกฟีเจอร์นี้ว่า **Audit Events** ซึ่งเราได้แตะเรื่องนี้ไว้บ้างแล้วใน **Part 53 (GitLab Groups, Permission และการจัดการทีม)** ตอนพูดถึงภาพรวมของ Group/Permission ในที่นี้เราจะเจาะลึกในมุม compliance โดยเฉพาะ

Audit Events ของ GitLab มีอยู่ 3 ระดับ:

1. **Project level** — เหตุการณ์ในโปรเจกต์เดียว เช่น เปลี่ยน visibility, เปลี่ยน branch protection ของ project นั้น
2. **Group level** — เหตุการณ์ในระดับ group ครอบคลุมทุก project ย่อยในนั้น เช่น เปลี่ยนสมาชิก, เปลี่ยน permission
3. **Instance level** (เฉพาะ GitLab Self-managed) — เหตุการณ์ทั้ง instance รวมถึงการตั้งค่าระบบระดับสูงสุด

ตัวอย่างเหตุการณ์ที่ GitLab Audit Events บันทึก:

- `Repository access level changed`
- `Protected branch created/updated/removed`
- `User added to project/group`
- `Permission level changed`
- `Merge request approved` (พร้อมชื่อผู้ approve)
- `CI/CD variable added/removed/updated`

GitLab ยังมีฟีเจอร์ **Audit Event Streaming** (ใน tier ระดับสูงขึ้นไป) ที่สามารถส่ง event เหล่านี้ออกไปยังระบบภายนอกแบบ real-time เช่นส่งไปยัง **SIEM (Security Information and Event Management)** ขององค์กร เพื่อให้ทีม security สามารถตรวจสอบและตั้ง alert ได้ทันทีโดยไม่ต้องเข้ามาดูใน GitLab UI เอง

### ทำไมต้องส่ง Audit Log ออกไปเก็บที่อื่นด้วย

ข้อควรระวังสำคัญ: **อย่าเก็บ audit log ไว้ที่เดียวกับระบบที่ถูกตรวจสอบเท่านั้น** เพราะถ้ามีคนได้สิทธิ์ admin ของ GitHub/GitLab organization ไปทั้งหมด (เช่นถูก compromise) ในทางทฤษฎีก็อาจพยายามลบหรือปิดการมองเห็น audit log ได้ (แม้ว่าในทางปฏิบัติ audit log ของแพลตฟอร์มใหญ่ ๆ มักถูกออกแบบให้ลบไม่ได้โดยผู้ใช้ทั่วไปก็ตาม)

แนวทางที่ดีสำหรับองค์กรที่ต้อง compliance จริงจังคือ:

1. **Export audit log ออกไปเก็บใน SIEM หรือ log aggregation platform** ที่แยกสิทธิ์การเข้าถึงจากทีม dev/admin ของ Git platform
2. **ตั้ง retention policy** ให้เก็บ log ย้อนหลังตามที่มาตรฐานกำหนด (เช่น SOC 2 มักต้องการอย่างน้อย 1 ปี)
3. **จำกัดสิทธิ์การดู/แก้ audit log** ให้เฉพาะทีม compliance/security เท่านั้น ไม่ใช่ทีม dev ทั่วไป

---

## Step 804: การเก็บหลักฐานสำหรับ Compliance Audit

### Auditor ต้องการอะไร

เมื่อถึงฤดูกาล audit จริง (ไม่ว่าจะเป็น SOC 2 Type II annual audit หรือ ISO 27001 surveillance audit) สิ่งที่ auditor จะขอดูซ้ำ ๆ ทุกปีคือ **ตัวอย่างหลักฐาน (sample evidence)** ของกระบวนการที่องค์กรอ้างว่าทำอยู่ เช่น:

> "ขอดูตัวอย่าง Pull Request 25 รายการที่ merge เข้า production branch ในไตรมาสที่ผ่านมา พร้อมหลักฐานว่าแต่ละรายการผ่านการอนุมัติจากคนที่ไม่ใช่ผู้เขียนโค้ดเอง"

คำขอแบบนี้คือหัวใจของ compliance evidence ในโลก Git และสิ่งที่ทีมต้องเตรียมให้พร้อมมีดังนี้

### 1. PR/MR Approval Record

Pull Request (GitHub) หรือ Merge Request (GitLab) เป็นแหล่งหลักฐานที่สมบูรณ์ที่สุดสำหรับกระบวนการ code review เพราะมันบันทึกไว้ครบทุกอย่าง:

- **ใครเป็นคนเปิด PR** (author)
- **ใครเป็นคน approve** และ **approve เมื่อไหร่**
- **ความคิดเห็นระหว่างรีวิว** (review comments) ที่แสดงว่ามีการตรวจสอบจริง ไม่ใช่กด approve มั่ว ๆ
- **สถานะของ required status check** ณ ตอนที่ merge (เช่น CI ผ่านหรือไม่)
- **วิธีการ merge** (merge commit, squash, rebase) และเวลาที่ merge จริง

ตัวอย่างการดึงข้อมูลการอนุมัติผ่าน GitHub API เพื่อทำเป็นหลักฐานส่ง auditor:

```bash
curl -H "Authorization: Bearer $TOKEN" \
  "https://api.github.com/repos/my-org/my-repo/pulls/482/reviews"
```

ผลลัพธ์จะแสดงรายชื่อผู้ review ทั้งหมด สถานะ (APPROVED, CHANGES_REQUESTED, COMMENTED) และเวลาที่ทำรายการ — ทีม compliance มักเขียนสคริปต์ดึงข้อมูลนี้เป็น batch สำหรับ PR ทั้งหมดในช่วงเวลาที่ auditor สนใจ แล้วสรุปเป็นรายงาน (evidence package)

### 2. Signed Commits เป็นหลักฐานว่า "ใครอนุมัติอะไรจริง"

ใน **Part 79** เราได้เรียนเรื่อง **Signed Commits** ไปแล้วว่าเป็นการใช้ GPG หรือ SSH key เซ็นกำกับ commit เพื่อยืนยันว่า commit นั้นมาจากเจ้าของ key จริง ๆ ไม่ใช่แค่ตั้งชื่อ author เฉย ๆ ใน `git config`

ในบริบท compliance สิ่งนี้สำคัญมากเพราะ:

> **Commit ที่ไม่ได้ sign สามารถปลอมแปลงชื่อผู้เขียนได้ง่ายมาก** (`git config user.name` / `user.email` ตั้งเป็นชื่อใครก็ได้) แต่ **commit ที่ signed แล้ว** ต้องมี private key ของเจ้าของตัวจริงเท่านั้นถึงจะสร้างลายเซ็นที่ผ่านการตรวจสอบได้

องค์กรที่ต้อง compliance ระดับสูง (เช่นสถาบันการเงิน หรือบริษัทที่ผ่าน PCI-DSS) มักบังคับให้:

1. ทุก commit ที่ merge เข้า branch หลัก **ต้อง signed** (บังคับผ่าน branch protection rule "Require signed commits")
2. เก็บ **public key ของพนักงานแต่ละคน** ไว้ในระบบกลาง เพื่อให้ตรวจสอบย้อนหลังได้ว่า signature นั้นเป็นของใคร
3. เมื่อพนักงานลาออก **ต้อง revoke key** ทันที เพื่อไม่ให้ signature เก่าถูกนำไปใช้ในทางที่ผิด (แม้ signature เก่าที่สร้างไว้ก่อนหน้ายังคงถูกต้องตามช่วงเวลาที่สร้าง)

GitHub และ GitLab จะแสดง badge **"Verified"** ข้าง commit ที่ signed ถูกต้อง ทำให้ auditor สามารถตรวจสอบด้วยตาได้ทันทีจากหน้าเว็บ โดยไม่ต้องรันคำสั่งใด ๆ เพิ่มเติม

### 3. การรวบรวมหลักฐานเป็น Evidence Package

ทีม compliance มืออาชีพมักไม่รอให้ auditor มาถามแล้วค่อยหาหลักฐาน แต่จะ **เตรียม evidence package ล่วงหน้าเป็นระยะ ๆ** (เช่นทุกไตรมาส) ประกอบด้วย:

| รายการหลักฐาน | แหล่งที่มา | ความถี่ในการเก็บ |
|---|---|---|
| รายชื่อ PR ที่ merge เข้า production พร้อมผู้ approve | GitHub/GitLab API | รายไตรมาส |
| สรุปการตั้งค่า branch protection ปัจจุบัน | Repository settings export | รายไตรมาส |
| Audit log การเปลี่ยนแปลงสิทธิ์สมาชิก | Organization audit log | รายเดือน |
| รายชื่อ commit ที่ไม่ผ่าน signed verification (ถ้ามี) | Git log + verification script | รายเดือน |
| หลักฐานการ revoke สิทธิ์เมื่อพนักงานลาออก | HR ticket + audit log | ทุกครั้งที่มีการลาออก |

การเก็บหลักฐานเป็นระยะแบบนี้ (แทนที่จะรอทำตอน audit) เรียกว่า **Continuous Compliance** ซึ่งเป็นแนวทางที่องค์กรระดับโลกใช้กันมากขึ้นเรื่อย ๆ เพราะลดภาระงานช่วง audit ลงอย่างมหาศาล และยังช่วยให้พบปัญหาได้เร็วก่อนที่ auditor จะเจอเสียอีก

---

## Step 805: Separation of Duties — แยกคนเขียนโค้ดกับคนอนุมัติ merge

### หลักการ Separation of Duties คืออะไร

**Separation of Duties (SoD)** หรือ **การแบ่งแยกหน้าที่** เป็นหลักการควบคุมภายใน (internal control) ที่เก่าแก่มากในโลกธุรกิจและบัญชี ก่อนจะถูกนำมาใช้ในโลกซอฟต์แวร์ หลักการพื้นฐานคือ:

> **ไม่มีบุคคลใดคนเดียวควรมีอำนาจควบคุมกระบวนการที่มีความเสี่ยงสูงได้ตั้งแต่ต้นจนจบเพียงลำพัง**

ในโลกบัญชีคลาสสิก ตัวอย่างเช่น คนที่อนุมัติการจ่ายเงินต้องไม่ใช่คนเดียวกับคนที่บันทึกบัญชีจ่ายเงินนั้น เพื่อป้องกันการทุจริต ในโลกซอฟต์แวร์ หลักการเดียวกันนี้แปลงมาเป็น:

> **คนที่เขียนโค้ด (author) ต้องไม่ใช่คนเดียวกับคนที่อนุมัติให้โค้ดนั้น merge เข้า branch หลัก (approver)**

### ทำไม Separation of Duties ถึงสำคัญมากในมุม Compliance

ถ้าไม่มีการแบ่งแยกหน้าที่นี้ นักพัฒนาคนเดียวสามารถ:

1. เขียนโค้ดที่มีช่องโหว่ (ตั้งใจหรือไม่ตั้งใจก็ตาม) เช่น backdoor, logic bomb, หรือโค้ดที่ขโมยข้อมูล
2. Merge โค้ดนั้นเข้า production **โดยไม่มีใครตรวจสอบเลย**
3. ถ้าเกิดปัญหาขึ้นภายหลัง **ไม่มีหลักฐานว่ามีกระบวนการควบคุมใด ๆ เกิดขึ้นก่อนหน้า** ทำให้องค์กรไม่ผ่าน audit และในกรณีร้ายแรงอาจต้องรับผิดทางกฎหมาย

ทุกมาตรฐาน compliance ที่กล่าวถึงใน Step 801 (SOC 2, ISO 27001, PCI-DSS) ล้วนมีข้อกำหนดเกี่ยวกับ SoD ไม่ทางใดก็ทางหนึ่ง โดยเฉพาะ PCI-DSS Requirement 6.5 ที่พูดถึงกระบวนการ code review อย่างชัดเจนว่าต้องทำโดย **บุคคลอื่นที่ไม่ใช่ผู้เขียนโค้ดต้นฉบับ**

### การนำ Separation of Duties มาปฏิบัติจริงด้วย Git

นี่คือจุดที่เชื่อมโยงกับสิ่งที่เราเรียนไปแล้วใน **Part 37 (Branch Protection Rules และ Merge Strategies)** โดยตรง การตั้งค่าที่จำเป็นมีดังนี้:

#### 1. Require Pull Request before merging

ปิดสิทธิ์การ push ตรงเข้า branch หลักโดยเด็ดขาด บังคับให้ทุกการเปลี่ยนแปลงต้องผ่าน PR เท่านั้น

#### 2. Require approvals (อย่างน้อย 1 คน ไม่ใช่ผู้เขียนเอง)

ทั้ง GitHub และ GitLab มีการตั้งค่านี้ในตัว โดยระบบจะ **ปฏิเสธไม่ให้ author กด approve PR ของตัวเอง** โดยอัตโนมัติอยู่แล้ว (ต่อให้ author พยายามกด approve เอง ระบบจะไม่นับเป็น required approval)

```yaml
# ตัวอย่างแนวคิดการตั้งค่าใน GitHub Branch Protection (ผ่าน UI หรือ Rulesets)
require_pull_request_reviews:
  required_approving_review_count: 2
  dismiss_stale_reviews: true
  require_code_owner_reviews: true
```

#### 3. Require Code Owner review สำหรับไฟล์ที่มีความอ่อนไหวสูง

ใช้ไฟล์ `CODEOWNERS` เพื่อบังคับว่าไฟล์บางประเภท (เช่นไฟล์เกี่ยวกับ authentication, payment, infrastructure-as-code) ต้องผ่านการอนุมัติจากทีมที่เชี่ยวชาญเฉพาะทางเสมอ ไม่ใช่ใครก็ได้ในทีม

```
# ตัวอย่างไฟล์ CODEOWNERS
/src/payment/          @company/payment-security-team
/infra/terraform/      @company/platform-team
/.github/workflows/    @company/devops-leads
```

#### 4. ป้องกันไม่ให้ Admin bypass การตั้งค่าโดยไม่ทิ้งร่องรอย

ข้อผิดพลาดที่พบบ่อยมากคือองค์กรตั้งค่า branch protection ครบถ้วน แต่ลืมปิดตัวเลือก **"Allow administrators to bypass these settings"** ทำให้คนที่มีสิทธิ์ admin ยัง merge เองได้โดยไม่ผ่านการอนุมัติ ซึ่งเป็นช่องโหว่ compliance ที่ auditor เจอบ่อยที่สุด — คำแนะนำคือ **ปิดสิทธิ์ bypass สำหรับทุกคนแม้แต่ admin** ในสภาพแวดล้อมที่ต้อง regulated อย่างเข้มงวด และถ้าจำเป็นต้อง bypass จริง ๆ ในกรณีฉุกเฉิน ต้องมีกระบวนการพิเศษที่ทิ้งหลักฐานไว้ใน audit log เสมอ (emergency change process)

### Separation of Duties ไม่ได้จำกัดแค่ Code Review

แนวคิดนี้ยังขยายไปถึง:

- **คนที่เขียน infrastructure-as-code ไม่ควรเป็นคนเดียวกับคนที่มีสิทธิ์ apply การเปลี่ยนแปลงนั้นเข้า production cloud account โดยตรง**
- **คนที่จัดการ CI/CD pipeline configuration ไม่ควรเป็นคนเดียวกับคนที่ตรวจสอบผลของ pipeline นั้น** เพื่อป้องกันการปรับแต่ง pipeline ให้ "ผ่าน" ทั้งที่จริงไม่ผ่าน
- **ทีม security ควรมีสิทธิ์ดู audit log ได้อย่างอิสระ โดยไม่ต้องขอสิทธิ์จากทีม dev**

---

## Step 806: Immutable History และการจำกัด Force-Push ในสภาพแวดล้อม Regulated

### Immutable History คืออะไร และทำไมมันสำคัญ

**Immutable History** หมายถึง **ประวัติที่เมื่อถูกบันทึกแล้วไม่สามารถถูกแก้ไข ลบ หรือเขียนทับได้อีก** นี่คือคุณสมบัติที่ระบบ audit trail ที่ดีทุกระบบต้องมี เพราะถ้าประวัติสามารถถูกแก้ไขย้อนหลังได้ หลักฐานทั้งหมดที่เก็บไว้ก็ไม่มีความน่าเชื่อถืออีกต่อไป — auditor จะไม่สามารถเชื่อได้เลยว่าสิ่งที่เห็นในวันนี้คือสิ่งที่เกิดขึ้นจริงในอดีต

โดยธรรมชาติ Git object (blob, tree, commit) ที่ถูกสร้างขึ้นแล้ว **ไม่สามารถแก้ไขเนื้อหาเดิมได้** เพราะ hash ผูกกับเนื้อหาโดยตรง (immutable ในระดับ object) แต่สิ่งที่ **เปลี่ยนแปลงได้** คือ **branch pointer** (reference) ที่ชี้ไปยัง commit ใดก็ได้ตามที่ผู้ใช้สั่ง — และนี่คือช่องโหว่ที่ force-push ใช้ประโยชน์

### Force-Push คืออะไร และทำไมมันอันตรายในบริบท Compliance

`git push --force` (หรือ `--force-with-lease`) คือคำสั่งที่บังคับให้ remote branch **ชี้ไปยัง commit ใหม่ที่เราระบุ โดยไม่สนใจว่า commit เดิมที่ remote เคยชี้อยู่จะหายไปจากสายตา (ไม่ถูกอ้างอิงอีกต่อไป)**

```bash
# คำสั่งอันตรายถ้าใช้กับ branch ที่ต้อง immutable
git push --force origin main
```

ผลกระทบของการ force-push บน branch ที่เป็นแหล่งหลักฐาน compliance:

1. **Commit ที่เคยถูกอนุมัติผ่าน PR อาจหายไปจากสายตา** — แม้ object ยังคงอยู่ใน Git internal storage ระยะหนึ่ง (จนกว่า garbage collection จะทำงาน) แต่จะไม่ปรากฏใน `git log` ปกติอีกต่อไป ทำให้ผู้ตรวจสอบเข้าใจผิดว่าประวัติเป็นแบบที่เห็น ณ ปัจจุบันเท่านั้น
2. **ทำลายความเชื่อมโยงระหว่าง PR record กับ commit จริง** — PR ที่เคย merge อาจแสดงผลไม่ตรงกับสิ่งที่อยู่บน branch จริงในปัจจุบันอีกต่อไป
3. **เปิดช่องให้ปกปิดการกระทำที่ไม่ต้องการ** — เช่น มีคน push โค้ดที่ไม่ผ่านการอนุมัติเข้าไปชั่วคราว แล้ว force-push ทับเพื่อ "ลบร่องรอย" ก่อนที่จะมีใครสังเกตเห็น

### แนวทางป้องกันในสภาพแวดล้อม Regulated

องค์กรที่ต้อง compliance อย่างเข้มงวดควรตั้งค่าดังนี้เสมอสำหรับ branch หลักและ branch ที่เกี่ยวข้องกับ production:

#### 1. ปิด Force-Push บน Protected Branch โดยสิ้นเชิง

ทั้ง GitHub (ผ่าน branch protection rule "Do not allow force pushes") และ GitLab (ผ่าน protected branch setting "Reject force push") มีตัวเลือกนี้ให้ตั้งค่าตรง ๆ

#### 2. ปิดสิทธิ์การลบ Branch หลัก

ป้องกันไม่ให้ branch หลักถูกลบทิ้งโดยไม่ตั้งใจหรือโดยเจตนาไม่ดี ซึ่งจะทำลาย reference ไปยังประวัติทั้งหมดทันที

#### 3. ตั้งค่า Server-Side Pre-Receive Hook เพิ่มเติม (สำหรับ Self-Hosted)

สำหรับองค์กรที่ใช้ GitLab Self-managed หรือ Git server ของตัวเอง สามารถเขียน **pre-receive hook** เพื่อปฏิเสธการ push ใด ๆ ที่จะทำให้ branch history ไม่ใช่ fast-forward (นั่นคือปฏิเสธ force-push ในระดับ server โดยไม่พึ่งการตั้งค่า UI เพียงอย่างเดียว) เป็นการป้องกันสองชั้น (defense in depth)

#### 4. เก็บ Backup ของ Repository แยกต่างหากอย่างสม่ำเสมอ

แม้จะป้องกันไว้ดีแค่ไหน ก็ควรมี **การ mirror/backup repository ไปยังที่เก็บอื่นเป็นระยะ** (เช่นรายวัน) เพื่อให้มี "หลักฐานอิสระ" ที่ยืนยันว่าประวัติ ณ เวลาหนึ่งเป็นอย่างไร ถ้าเกิดกรณีที่ต้องพิสูจน์ย้อนหลังว่าเคยมีอะไรอยู่ก่อนหน้า

#### 5. อนุญาต Force-Push เฉพาะใน Feature Branch ส่วนตัวเท่านั้น

ไม่จำเป็นต้องห้าม force-push ทุกที่ในระบบ — นักพัฒนายังคงต้องการ force-push บน feature branch ของตัวเอง (เช่นหลังจาก interactive rebase เพื่อจัดระเบียบ commit ก่อนเปิด PR) สิ่งที่ต้องควบคุมอย่างเข้มงวดคือ **branch ที่เป็นแหล่งความจริงร่วมกัน** (`main`, `release/*`, `production`) เท่านั้น

### หลักการสำคัญที่ต้องจำ

> **Immutability ไม่ได้แปลว่าห้ามแก้ไขโค้ดผิดพลาด แต่แปลว่าการแก้ไขทุกครั้งต้องเกิดเป็น commit ใหม่ที่ต่อท้ายประวัติเสมอ (forward-only) ไม่ใช่การเขียนทับประวัติเดิม** — ถ้าโค้ดผิด ให้สร้าง commit ใหม่ที่แก้ไขหรือ revert แทนที่จะไปแก้ไข/ลบประวัติเก่า

---

## Step 807: Data Retention Policy สำหรับ Repository และ CI/CD Logs

### Data Retention Policy คืออะไร

**Data Retention Policy** (นโยบายการเก็บรักษาข้อมูล) คือ **กฎที่กำหนดว่าข้อมูลประเภทหนึ่ง ๆ จะถูกเก็บไว้นานเท่าไหร่ก่อนที่จะต้องถูกลบหรือ archive** นโยบายนี้สำคัญมากในบริบท compliance เพราะมันตอบทั้งสองด้านที่ดูเหมือนขัดแย้งกัน:

1. **ต้องเก็บนานพอ** เพื่อให้มีหลักฐานย้อนหลังเพียงพอสำหรับการตรวจสอบ (หลายมาตรฐานกำหนดระยะเวลาขั้นต่ำ)
2. **ต้องไม่เก็บนานเกินความจำเป็น** เพราะการเก็บข้อมูล (โดยเฉพาะข้อมูลส่วนบุคคล) นานเกินไปโดยไม่มีเหตุผลทางธุรกิจ ถือเป็นความเสี่ยงตามกฎหมายคุ้มครองข้อมูลส่วนบุคคลในหลายประเทศ (เช่น GDPR ของยุโรป หรือ PDPA ของไทย) ที่มีหลักการ **"Data Minimization"** และ **"Storage Limitation"**

### ข้อมูลประเภทไหนบ้างที่ต้องมี Retention Policy ในระบบ Git

#### 1. Repository History (Commit, Branch, Tag)

โดยทั่วไป **ประวัติ Git ของ source code เองมักถูกเก็บไว้ถาวร** ไม่มีการลบ เพราะ:

- ตัวโค้ดเองมักไม่ใช่ "ข้อมูลส่วนบุคคล" (personal data) โดยตรง (ยกเว้นกรณีมี PII หลุดเข้าไป ซึ่งเราจะพูดถึงใน Step 808)
- การรักษาประวัติทั้งหมดไว้คือหลักฐาน compliance ที่มีค่าในตัวเอง
- ต้นทุนการเก็บ Git object (ซึ่งถูกบีบอัดและ deduplicate อย่างมีประสิทธิภาพ) ต่ำมากเมื่อเทียบกับข้อมูลประเภทอื่น

#### 2. CI/CD Pipeline Logs

Log จากการรัน pipeline (build log, test log, deployment log) มักมี **retention period ที่จำกัด** เพราะ:

- มีขนาดใหญ่มาก และมักมีข้อมูลที่ไม่จำเป็นต้องเก็บถาวร (debug output จำนวนมาก)
- อาจมีข้อมูลอ่อนไหวหลุดเข้าไปโดยไม่ตั้งใจ เช่น environment variable หรือ token ที่ log ออกมาโดยไม่ได้ mask

แนวทางทั่วไป:

| ประเภท Log | Retention ที่แนะนำ | เหตุผล |
|---|---|---|
| CI log ของ feature branch | 30–90 วัน | ใช้ debug ระหว่างพัฒนา ไม่มีค่า compliance ระยะยาว |
| CI log ของ main/release branch | 1 ปีขึ้นไป | เป็นหลักฐานว่า pipeline ผ่านก่อน deploy |
| Deployment log สู่ production | ตามข้อกำหนดมาตรฐาน (มักอย่างน้อย 1 ปี) | หลักฐานสำคัญสำหรับ change management audit |
| Audit log ของ organization | อย่างน้อย 1–7 ปี ตามมาตรฐานที่บังคับใช้ | บางมาตรฐาน (การเงิน) กำหนดยาวถึง 7 ปี |

GitHub Actions และ GitLab CI ต่างมีการตั้งค่า **artifact/log expiration** ในตัว เช่นใน GitLab CI:

```yaml
build_job:
  script:
    - make build
  artifacts:
    paths:
      - build/
    expire_in: 90 days   # ลบ artifact อัตโนมัติหลัง 90 วัน
```

#### 3. Personal Access Token และ Secret Rotation Log

บันทึกว่า token/secret ตัวไหนถูกสร้าง/revoke เมื่อไหร่ ควรเก็บไว้ตราบเท่าที่ token นั้นยังมีผล และเก็บ log การ revoke ไว้ต่ออีกระยะหนึ่งเพื่อพิสูจน์ย้อนหลังได้ว่ามีการจัดการ credential อย่างเหมาะสม

### การนำ Retention Policy ไปปฏิบัติจริง

1. **เขียนนโยบายเป็นเอกสารทางการ** ระบุชัดเจนว่าข้อมูลแต่ละประเภทเก็บนานเท่าไหร่ และใครเป็นเจ้าของการตัดสินใจ
2. **Automate การลบ/archive** ผ่านการตั้งค่าของแพลตฟอร์ม (เช่น artifact expiration ข้างต้น) แทนการลบด้วยมือ เพื่อให้สอดคล้องกับนโยบายเสมอโดยไม่ลืม
3. **แยกการเก็บ audit log ออกจาก operational log** เพราะมี retention requirement ต่างกันมาก (audit log ต้องเก็บนานกว่ามาก)
4. **ทบทวนนโยบายเป็นระยะ** เมื่อมีมาตรฐานใหม่เข้ามาบังคับใช้ หรือเมื่อกฎหมายเปลี่ยนแปลง

---

## Step 808: การจัดการ Personal Identifiable Information (PII) ไม่ให้หลุดเข้า Commit Message หรือโค้ด

### ทำไมเรื่องนี้ถึงอันตรายเป็นพิเศษใน Git

นี่คือหนึ่งในความเสี่ยงที่นักพัฒนาประเมินต่ำที่สุด เพราะ **Git ถูกออกแบบมาให้เก็บประวัติถาวรและกระจายไปยังทุกเครื่องที่ clone** ผลที่ตามมาคือ:

> **เมื่อ PII (Personally Identifiable Information) เช่น เลขบัตรประชาชน, เบอร์โทรศัพท์ลูกค้า, อีเมลจริงของผู้ใช้งาน, หรือรหัสผ่านถูก commit เข้าไปแล้ว การลบไฟล์นั้นออกใน commit ถัดไปไม่ได้ทำให้ข้อมูลหายไปเลย มันยังคงอยู่ในประวัติ และยังคงอยู่ในทุกเครื่องที่เคย clone หรือ fetch repository นั้นไปแล้ว**

ในบางกรณีที่ร้ายแรง เช่นข้อมูลผู้ป่วยตาม HIPAA หรือข้อมูลบัตรเครดิตตาม PCI-DSS การหลุดของข้อมูลแบบนี้ **ถือเป็น data breach ที่ต้องรายงานตามกฎหมาย** แม้ repository จะเป็น private ก็ตาม เพราะพนักงานหลายสิบหรือหลายร้อยคนอาจเคย clone ไปแล้ว

### แหล่งที่มาทั่วไปของ PII ที่หลุดเข้า Git

1. **Test data / seed data** — นักพัฒนาคัดลอกข้อมูลลูกค้าจริงมาใช้ทดสอบแทนที่จะสร้าง fake data
2. **Log statement ที่ debug ค้างไว้** — `console.log(user.email, user.creditCard)` ที่ลืมลบก่อน commit
3. **Configuration file ที่มี credential ฝังอยู่** — connection string ที่มี username/password จริงของฐานข้อมูล production
4. **Commit message ที่อ้างอิงข้อมูลจริงโดยไม่ตั้งใจ** เช่น "แก้บั๊กให้ user สมชาย เลขบัตร 1-2345-xxxxx-xx-x ล็อกอินไม่ได้"
5. **Screenshot หรือไฟล์แนบ** ที่แนบเข้า commit เพื่อประกอบการอธิบายบั๊ก แต่ภาพนั้นมีข้อมูลลูกค้าจริงติดอยู่

### แนวทางป้องกันเชิงระบบ (ไม่พึ่งวินัยของคนเพียงอย่างเดียว)

#### 1. Pre-Commit Hook / Secret Scanning ก่อน Commit

ใช้เครื่องมืออย่าง **git-secrets, gitleaks, หรือ TruffleHog** ตั้งเป็น pre-commit hook เพื่อสแกนหา pattern ที่น่าสงสัย (เลขบัตรเครดิต, API key, เลขบัตรประชาชน) ก่อนที่ commit จะถูกสร้างขึ้นด้วยซ้ำ

```bash
# ตัวอย่างการติดตั้ง gitleaks เป็น pre-commit hook
gitleaks protect --staged -v
```

#### 2. Server-Side Secret Scanning (GitHub Secret Scanning / GitLab Secret Detection)

ทั้ง GitHub และ GitLab มีระบบสแกนหา secret pattern ที่รู้จัก (API key ของผู้ให้บริการ cloud ต่าง ๆ, token รูปแบบมาตรฐาน) โดยอัตโนมัติทุกครั้งที่มีการ push และสามารถตั้งค่าให้ **push protection** ปฏิเสธการ push ตั้งแต่ต้นทางถ้าพบ secret ที่มีรูปแบบชัดเจน แทนที่จะให้เข้าไปอยู่ในประวัติก่อนแล้วค่อยแจ้งเตือนทีหลัง

#### 3. ห้าม Log ข้อมูล PII ในระดับ Code Review Checklist

กำหนดเป็นข้อบังคับใน code review checklist ว่าห้าม log ข้อมูลอ่อนไหวแบบ plain text เด็ดขาด และให้ใช้เทคนิค **masking/redaction** เช่นแสดงแค่ 4 ตัวท้ายของเลขบัตร

#### 4. ใช้ Synthetic/Fake Data สำหรับ Test เสมอ

บังคับใช้ไลบรารีสร้างข้อมูลปลอม (เช่น Faker) แทนการคัดลอกข้อมูลจริงมาทดสอบ และห้าม export ข้อมูล production มาใช้ใน environment พัฒนาโดยไม่ผ่านการทำ **data masking/anonymization** ก่อนเสมอ

### ถ้า PII หลุดเข้าไปในประวัติแล้วต้องทำอย่างไร

เมื่อพบว่า PII หลุดเข้าไปในประวัติ Git แล้ว การลบไฟล์ธรรมดาไม่พอ ต้องดำเนินการดังนี้:

1. **ประเมินผลกระทบทันที** — ข้อมูลนั้นหลุดไปถึงใครบ้าง (ใคร clone/fork repo นี้ไปแล้ว) และต้องรายงานตามกฎหมายหรือไม่
2. **Rewrite history เพื่อลบข้อมูลออกจากทุก commit** โดยใช้เครื่องมือเฉพาะทาง เช่น `git filter-repo` (เครื่องมือที่ Git project แนะนำอย่างเป็นทางการในปัจจุบัน แทนที่ `git filter-branch` รุ่นเก่าที่ช้าและมีข้อผิดพลาดง่าย) หรือ BFG Repo-Cleaner
3. **Force-push ประวัติใหม่ทับ** และ **แจ้งทุกคนในทีมให้ re-clone repository ใหม่ทั้งหมด** (ห้ามใช้สำเนาเก่าต่อ เพราะจะมีข้อมูลเดิมติดอยู่ในเครื่อง)
4. **Rotate credential ที่หลุดไปทันที** ถ้าสิ่งที่หลุดคือรหัสผ่านหรือ API key (ถือว่า credential นั้น "ถูกเผา" แล้วต้องเปลี่ยนใหม่เสมอ ต่อให้ลบออกจากประวัติแล้วก็ตาม เพราะไม่มีทางรู้แน่ชัดว่ามีใครเห็นไปแล้วหรือยัง)
5. **บันทึกเหตุการณ์นี้ไว้เป็นหลักฐานของกระบวนการ incident response** เพื่อแสดงให้ auditor เห็นว่าองค์กรมีกระบวนการรับมือที่เป็นระบบ ไม่ใช่ปกปิดปัญหา

> ข้อสังเกตสำคัญ: การ force-push ในกรณีนี้คือ **ข้อยกเว้นที่จำเป็น** ต่อหลักการ immutable history ใน Step 806 — compliance ที่ดีไม่ได้แปลว่าห้าม force-push โดยเด็ดขาดร้อยเปอร์เซ็นต์ แต่แปลว่า **การกระทำเช่นนี้ต้องผ่านกระบวนการอนุมัติพิเศษและถูกบันทึกไว้เป็นหลักฐานอย่างชัดเจนเสมอ**

---

## Step 809: Compliance-as-Code — เขียนกฎ Compliance เป็น Policy Code

### ปัญหาของการตรวจสอบ Compliance ด้วยมือ

วิธีดั้งเดิมในการตรวจสอบ compliance คือให้คนในทีม security/compliance **เข้าไปตรวจสอบการตั้งค่าทีละ repository ด้วยมือ** เช่น เข้าไปดูว่า repository นี้เปิด branch protection หรือยัง, บังคับ signed commit หรือไม่, มี CODEOWNERS หรือเปล่า

วิธีนี้มีปัญหาใหญ่หลายข้อ:

1. **ไม่ scale** — องค์กรที่มี repository เป็นร้อยเป็นพัน ไม่มีทางตรวจสอบด้วยมือได้ทันเวลา
2. **ตรวจสอบแค่ ณ จุดเวลาหนึ่ง** — ตั้งค่าถูกต้องวันนี้ ไม่ได้แปลว่าจะถูกต้องตลอดไป (มีคนอาจไปปิดการตั้งค่าทีหลัง)
3. **มีโอกาสผิดพลาดจากมนุษย์สูง** — ตรวจสอบไม่ครบทุกจุด หรือเข้าใจ requirement ผิด
4. **ตรวจพบปัญหาช้าเกินไป** — มักพบตอน audit ประจำปีเท่านั้น ทั้งที่ปัญหาอาจเกิดขึ้นมาหลายเดือนแล้ว

### แนวคิด Compliance-as-Code

**Compliance-as-Code** คือแนวคิดที่นำหลักการเดียวกับ **Infrastructure-as-Code** มาใช้กับกฎ compliance นั่นคือ:

> **เขียนกฎ compliance ให้อยู่ในรูปแบบโค้ด (policy code) ที่คอมพิวเตอร์อ่านและตรวจสอบได้เอง แทนที่จะเขียนเป็นเอกสาร Word ที่คนต้องอ่านแล้วไปตรวจสอบด้วยมือ**

เมื่อกฎกลายเป็นโค้ด มันจะได้รับประโยชน์ทุกอย่างที่โค้ดมี: **version control ได้เอง, รันอัตโนมัติซ้ำ ๆ ได้, ทดสอบได้, และรวมเข้ากับ pipeline ได้โดยตรง**

### ตัวอย่างการทำ Compliance-as-Code กับ Git Workflow

#### 1. ตรวจสอบการตั้งค่า Repository อัตโนมัติด้วยสคริปต์

เขียนสคริปต์ที่ดึงการตั้งค่าจริงของทุก repository ผ่าน API แล้วเปรียบเทียบกับ policy ที่กำหนดไว้:

```python
# ตัวอย่างแนวคิด (pseudocode แบบง่าย) ตรวจสอบว่า repo ทุกตัวมี branch protection ที่ถูกต้อง
import requests

REQUIRED_APPROVALS = 2
policy_violations = []

for repo in get_all_org_repos():
    protection = get_branch_protection(repo, branch="main")
    if protection is None:
        policy_violations.append(f"{repo}: ไม่มี branch protection บน main เลย")
        continue
    if protection["required_approving_review_count"] < REQUIRED_APPROVALS:
        policy_violations.append(f"{repo}: approval count ต่ำกว่าที่กำหนด")
    if not protection["required_signed_commits"]:
        policy_violations.append(f"{repo}: ไม่บังคับ signed commit")
    if protection["allow_force_pushes"]:
        policy_violations.append(f"{repo}: ยังเปิดให้ force-push ได้")

report_to_compliance_dashboard(policy_violations)
```

สคริปต์แบบนี้สามารถรันเป็น **scheduled job** ทุกวันหรือทุกสัปดาห์ และส่งผลลัพธ์เข้า dashboard หรือแจ้งเตือนทีมทันทีที่พบการตั้งค่าที่ไม่เป็นไปตามนโยบาย แทนที่จะรอให้ auditor มาเจอเอง

#### 2. Policy-as-Code ใน CI/CD Pipeline

นอกจากตรวจสอบการตั้งค่า repository แล้ว ยังสามารถฝัง policy check เข้าไปใน pipeline โดยตรง เช่นใช้เครื่องมืออย่าง **Open Policy Agent (OPA)** เพื่อกำหนดกฎ compliance ที่ต้องผ่านก่อน deploy ได้:

```rego
# ตัวอย่างแนวคิด Rego policy (OPA) ตรวจสอบว่า deployment ต้องมาจาก branch ที่ผ่าน approval แล้วเท่านั้น
package compliance.deployment

deny[msg] {
    input.source_branch != "main"
    msg := "อนุญาตให้ deploy จาก branch main ที่ผ่านการอนุมัติเท่านั้น"
}

deny[msg] {
    input.pr_approved_count < 2
    msg := "ต้องมีผู้อนุมัติอย่างน้อย 2 คนก่อน deploy"
}
```

#### 3. Compliance Gate ในขั้นตอน Merge

ตั้งค่า required status check ให้ pipeline การตรวจสอบ compliance (เช่น secret scanning, license scanning, signed commit verification) เป็นหนึ่งใน check ที่ **ต้องผ่านก่อน merge button จะกดได้** ทำให้ compliance ไม่ใช่สิ่งที่ตรวจสอบทีหลัง แต่เป็น **gate ที่บังคับใช้ล่วงหน้าโดยอัตโนมัติ**

### ประโยชน์ของ Compliance-as-Code เมื่อเทียบกับการตรวจสอบด้วยมือ

| มิติ | ตรวจสอบด้วยมือ | Compliance-as-Code |
|---|---|---|
| ความถี่ในการตรวจสอบ | นาน ๆ ครั้ง (รายไตรมาส/รายปี) | ต่อเนื่อง (real-time หรือรายวัน) |
| ความสามารถในการ scale | ต่ำ (จำกัดด้วยจำนวนคน) | สูงมาก (ตรวจได้เป็นพัน repo พร้อมกัน) |
| โอกาสผิดพลาดจากมนุษย์ | สูง | ต่ำ (เมื่อ policy code ถูกต้อง) |
| ความเร็วในการพบปัญหา | ช้า มักพบตอน audit | เร็ว พบทันทีที่เกิดการเบี่ยงเบน |
| หลักฐานสำหรับ Auditor | ต้องรวบรวมด้วยมือทุกครั้ง | ดึงรายงานได้ทันทีจากระบบ |

> **ข้อคิดสำคัญ:** Compliance-as-Code ไม่ได้แทนที่การตรวจสอบของมนุษย์ทั้งหมด แต่ทำให้มนุษย์ไปโฟกัสกับสิ่งที่ต้องใช้วิจารณญาณจริง ๆ (เช่นตัดสินใจว่า exception กรณีพิเศษควรอนุมัติหรือไม่) แทนที่จะเสียเวลาไปกับการตรวจสอบซ้ำ ๆ ที่คอมพิวเตอร์ทำได้ดีกว่าและเร็วกว่ามาก

---

## Step 810: แบบฝึกหัด — ออกแบบ Audit Trail Checklist สำหรับองค์กรสมมติ

### โจทย์

สมมติว่าคุณเป็น Tech Lead ของบริษัท **"FinPay Technology"** บริษัท fintech ที่ให้บริการชำระเงินออนไลน์ ซึ่งต้องผ่าน:

- **PCI-DSS** เพราะประมวลผลข้อมูลบัตรเครดิต
- **SOC 2 Type II** เพราะขายบริการให้ลูกค้าองค์กร
- กฎหมายคุ้มครองข้อมูลส่วนบุคคลในประเทศที่บริษัทดำเนินธุรกิจ

บริษัทกำลังจะเข้ารับการตรวจสอบ compliance ประจำปีในอีก 3 เดือนข้างหน้า และมอบหมายให้คุณออกแบบ **Audit Trail Checklist** สำหรับระบบ Git/CI-CD ทั้งหมดขององค์กร

### ขั้นตอนที่ควรทำ (แนวทางสำหรับฝึกคิด)

ลองเขียน checklist ของตัวเองก่อน แล้วเทียบกับแนวทางเฉลยด้านล่าง โดยพิจารณาหัวข้อหลักต่อไปนี้:

1. Repository configuration และ branch protection
2. Access control และ Separation of Duties
3. Evidence collection process
4. PII/Secret prevention
5. Retention และ log management
6. Automation/Compliance-as-Code

### แนวทางเฉลย: Audit Trail Checklist ฉบับสมบูรณ์

#### หมวดที่ 1: Repository Configuration และ Branch Protection

- [ ] ทุก repository ที่เกี่ยวข้องกับระบบชำระเงินมี branch protection บน `main`/`production` เปิดใช้งานครบถ้วน
- [ ] บังคับ Pull Request ก่อน merge เข้า branch หลักเสมอ (ไม่มี direct push)
- [ ] บังคับผู้อนุมัติอย่างน้อย 2 คน และไม่อนุญาตให้ author approve งานตัวเอง
- [ ] บังคับ Code Owner review สำหรับโค้ดที่เกี่ยวกับ payment processing และ authentication
- [ ] ปิดสิทธิ์ force-push และการลบ branch หลักสำหรับทุกคนรวมถึง admin
- [ ] บังคับ signed commit สำหรับทุก commit ที่ merge เข้า production branch
- [ ] บังคับ required status check (CI ต้องผ่าน, security scan ต้องผ่าน) ก่อน merge ได้

#### หมวดที่ 2: Access Control และ Separation of Duties

- [ ] มีเอกสารระบุชัดเจนว่าใครมีสิทธิ์ระดับใดใน organization/group (owner, maintainer, developer)
- [ ] ทบทวนรายชื่อสมาชิกและสิทธิ์ทุกไตรมาส (access review)
- [ ] มีกระบวนการ offboarding ที่ revoke สิทธิ์ทันทีเมื่อพนักงานลาออกหรือเปลี่ยนตำแหน่ง (ผูกกับระบบ HR)
- [ ] ทีมที่มีสิทธิ์ deploy เข้า production ไม่ใช่ทีมเดียวกับทีมที่เขียน infrastructure-as-code ทั้งหมด (หรือมีการอนุมัติแยกชั้น)
- [ ] มีบันทึกว่าใครมี admin bypass สิทธิ์ และมีเหตุผลทางธุรกิจรองรับชัดเจนสำหรับแต่ละคน

#### หมวดที่ 3: Evidence Collection Process

- [ ] มีสคริปต์/ระบบดึงรายชื่อ PR ที่ merge เข้า production พร้อมผู้อนุมัติ แบบอัตโนมัติทุกเดือน
- [ ] Export audit log ของ organization ไปเก็บใน log storage แยกต่างหาก (นอกเหนือจากที่ platform เก็บเอง)
- [ ] มี evidence package พร้อมส่งมอบ auditor ล่วงหน้าอย่างน้อย 1 เดือนก่อน audit จริง
- [ ] มีหลักฐานยืนยันตัวตนของผู้อนุมัติ (signed commit + platform-level approval record ที่ตรงกัน)
- [ ] เก็บบันทึกการ bypass/exception ทุกครั้งพร้อมเหตุผลและผู้อนุมัติ exception นั้น

#### หมวดที่ 4: PII / Secret Prevention

- [ ] เปิดใช้ secret scanning + push protection ทั้งระดับ pre-commit hook และระดับ server (platform)
- [ ] มีนโยบายห้ามใช้ข้อมูลลูกค้าจริงใน test/dev environment โดยไม่ผ่าน data masking
- [ ] มีกระบวนการตอบสนองเมื่อพบ secret/PII หลุดเข้า repository (รวมขั้นตอน rewrite history + rotate credential)
- [ ] Code review checklist มีข้อบังคับห้าม log ข้อมูลอ่อนไหวแบบ plain text

#### หมวดที่ 5: Retention และ Log Management

- [ ] มีเอกสารนโยบาย data retention ที่ระบุระยะเวลาเก็บของแต่ละประเภทข้อมูล (source code, CI log, audit log)
- [ ] Retention period สอดคล้องกับข้อกำหนดขั้นต่ำของ PCI-DSS และ SOC 2 (โดยทั่วไปอย่างน้อย 1 ปีสำหรับ log ที่เกี่ยวกับ production change)
- [ ] ตั้งค่า artifact/log expiration อัตโนมัติในระบบ CI/CD ให้ตรงกับนโยบาย
- [ ] Audit log ถูกจำกัดสิทธิ์การเข้าถึง/แก้ไข ให้เฉพาะทีม security/compliance เท่านั้น

#### หมวดที่ 6: Automation / Compliance-as-Code

- [ ] มีสคริปต์ตรวจสอบการตั้งค่า branch protection ของทุก repository โดยอัตโนมัติเป็นประจำ (รายวัน/รายสัปดาห์)
- [ ] ผลการตรวจสอบถูกส่งเข้า dashboard หรือแจ้งเตือนทีมทันทีเมื่อพบการเบี่ยงเบนจากนโยบาย
- [ ] มี policy-as-code (เช่นผ่าน OPA หรือเครื่องมือเทียบเท่า) ฝังอยู่ใน pipeline สำหรับ gate การ deploy
- [ ] Policy code เองก็ถูกเก็บใน version control และผ่านกระบวนการ review เช่นเดียวกับโค้ด production

### สิ่งที่ควรสังเกตจากแบบฝึกหัดนี้

ถ้าคุณลองทำ checklist นี้จริงกับองค์กรของตัวเอง คุณจะพบว่า **ข้อส่วนใหญ่ไม่ใช่เรื่องใหม่เลย** มันคือสิ่งที่เราเรียนไปแล้วในหลาย Part ก่อนหน้าของหลักสูตรนี้ (branch protection จาก Part 37, GitLab permission และ audit events จาก Part 53, signed commits จาก Part 79) เพียงแต่ Part นี้นำทุกอย่างมาร้อยเรียงเข้าด้วยกันภายใต้กรอบคิดของ **compliance และ audit trail**

นี่คือบทเรียนสำคัญที่สุดของ Part นี้: **Compliance ที่ดีไม่ใช่การทำงานพิเศษเพิ่มเติมนอกเหนือจากงานปกติ แต่คือการตั้งค่า Git workflow ให้ถูกต้องตั้งแต่ต้น จนหลักฐาน compliance เกิดขึ้นเองโดยอัตโนมัติจากการทำงานประจำวันของทีม**

---

## สรุป Part 81

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Compliance** คือการปฏิบัติตามมาตรฐานหรือกฎหมาย (SOC 2, ISO 27001, HIPAA, PCI-DSS) โดยสิ่งที่ auditor ต้องการเสมอคือหลักฐานที่พิสูจน์ได้ ไม่ใช่แค่คำยืนยัน
2. **Git เป็นเครื่องมือ audit trail ที่ดีโดยธรรมชาติ** เพราะทุก commit มี hash, author/committer, timestamp ที่ตรวจสอบได้ และการแก้ไขประวัติจะทิ้งร่องรอยเสมอ แต่ Git local อย่างเดียวไม่พอ ต้องอาศัยหลักฐานระดับ server-side ด้วย
3. **Audit Log ของ GitHub/GitLab organization** บันทึกเหตุการณ์ระดับแพลตฟอร์มที่ `git log` มองไม่เห็น เช่นการเปลี่ยนสิทธิ์สมาชิกหรือการ bypass branch protection ควรถูก export ไปเก็บแยกต่างหากเพื่อความน่าเชื่อถือ
4. **หลักฐานสำหรับ compliance audit** ที่สำคัญที่สุดคือ PR approval record และ signed commit ซึ่งยืนยันได้ว่าใครอนุมัติอะไรจริง ควรรวบรวมเป็น evidence package ล่วงหน้าอย่างต่อเนื่อง (continuous compliance)
5. **Separation of Duties** คือหลักการที่คนเขียนโค้ดต้องไม่ใช่คนอนุมัติ merge เอง นำไปปฏิบัติจริงผ่าน branch protection, required approvals, และ CODEOWNERS
6. **Immutable history** คือหัวใจของความน่าเชื่อถือของ audit trail การจำกัด force-push บน branch หลักอย่างเข้มงวดจึงจำเป็นมากในสภาพแวดล้อม regulated แม้จะมีข้อยกเว้นบางกรณี (เช่นการลบ PII ที่หลุดเข้าไป) ก็ต้องผ่านกระบวนการอนุมัติพิเศษเสมอ
7. **Data Retention Policy** ต้องสมดุลระหว่างการเก็บนานพอสำหรับ audit กับการไม่เก็บนานเกินความจำเป็นตามหลัก data minimization
8. **PII ต้องถูกป้องกันไม่ให้หลุดเข้า Git ตั้งแต่ต้นทาง** ด้วย secret scanning ทั้งระดับ pre-commit และ server-side เพราะการลบออกภายหลังทำได้ยากและมีความเสี่ยงสูง
9. **Compliance-as-Code** เปลี่ยนการตรวจสอบด้วยมือที่ scale ไม่ได้ ให้กลายเป็นการตรวจสอบอัตโนมัติที่ต่อเนื่อง แม่นยำ และพร้อมเป็นหลักฐานได้ตลอดเวลา
10. เราได้ฝึกออกแบบ **audit trail checklist ฉบับสมบูรณ์** สำหรับองค์กร fintech สมมติ ที่ครอบคลุมทั้ง 6 มิติของ compliance ในระบบ Git/CI-CD

**ต่อไป:** [Part 82: Monorepo vs Polyrepo: การตัดสินใจเชิงสถาปัตยกรรม](./part-082-monorepo-vs-polyrepo.md)
