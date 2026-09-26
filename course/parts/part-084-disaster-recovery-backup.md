# Part 84: Disaster Recovery และ Backup Strategy สำหรับ Git

> **Step ในหลักสูตรนี้:** Step 831–840
> **เฟส:** 8 — DevOps, Security, Compliance ระดับองค์กร
> **เป้าหมายของ Part นี้:** เข้าใจว่าทำไมการฝากโค้ดไว้บน GitHub/GitLab เพียงอย่างเดียวไม่ใช่ "แผนสำรองข้อมูล" ที่สมบูรณ์ เรียนรู้วิธีทำ mirror backup, multi-remote strategy, การสำรอง metadata ที่ไม่ได้อยู่ใน Git object (issues, PR, wiki), การเขียน Disaster Recovery Plan ที่มี RTO/RPO ชัดเจน และลงมือทดสอบ restore จริงจนมั่นใจว่าระบบสำรองข้อมูลของทีมใช้งานได้จริงเมื่อเกิดเหตุฉุกเฉิน

---

## สารบัญของ Part นี้

- Step 831: ทำไมต้องมี backup ของ Git repository แม้ SaaS จะดูแล infrastructure ให้แล้ว
- Step 832: Single Point of Failure ที่เกิดขึ้นจริงในโลกความเป็นจริง
- Step 833: กลยุทธ์ backup พื้นฐานที่สุด — Mirror Clone
- Step 834: `git clone --mirror` และการตั้งค่า sync backup อัตโนมัติด้วย script/cron
- Step 835: Multi-Remote Strategy — push โค้ดไปหลาย remote พร้อมกัน
- Step 836: Backup metadata ที่ไม่ใช่โค้ด (Issues, PR, Wiki) ผ่าน API
- Step 837: Disaster Recovery Plan — RTO และ RPO ประยุกต์ใช้กับ Git Infrastructure
- Step 838: การทดสอบ Restore จริง — หลักการที่คนมองข้ามบ่อยที่สุด
- Step 839: Self-Hosted Git Server เป็นทางเลือก backup ขั้นสุด (GitLab Self-Managed, Gitea)
- Step 840: แบบฝึกหัด — ตั้งค่า mirror backup อัตโนมัติ พร้อมทดสอบ restore จริง

---

## Step 831: ทำไมต้องมี backup ของ Git repository แม้ SaaS จะดูแล infrastructure ให้แล้ว

หลายคนคิดว่า "โค้ดอยู่บน GitHub แล้ว ปลอดภัยแน่นอน" เพราะ GitHub เป็นบริษัทระดับโลก มี datacenter หลายแห่ง มี engineer มืออาชีพดูแลระบบตลอด 24 ชั่วโมง ความคิดนี้ **ถูกครึ่งเดียว**

### สิ่งที่ GitHub/GitLab รับผิดชอบให้จริง ๆ

แพลตฟอร์ม SaaS อย่าง GitHub, GitLab.com, Bitbucket มี **SLA (Service Level Agreement)** ที่ครอบคลุมเรื่อง:

1. **Infrastructure availability** — เซิร์ฟเวอร์ไม่ล่ม มี redundancy ระดับ datacenter (multi-region replication)
2. **Data durability ในระดับ storage** — ป้องกันดิสก์เสีย ฮาร์ดแวร์พัง ด้วยการทำ replication หลายชุดในระบบของเขา
3. **Security patching ของแพลตฟอร์ม** — อุดช่องโหว่ของซอฟต์แวร์ที่รันแพลตฟอร์ม

นี่คือสิ่งที่เรียกว่า **Shared Responsibility Model** (โมเดลความรับผิดชอบร่วม) ซึ่งเป็นแนวคิดเดียวกับที่ใช้ในโลก Cloud Computing (AWS, Azure, GCP) — ผู้ให้บริการรับผิดชอบ "ความปลอดภัยของ infrastructure" แต่ **ผู้ใช้งานต้องรับผิดชอบ "ความปลอดภัยของข้อมูลตัวเอง"**

### สิ่งที่ GitHub/GitLab "ไม่ได้" รับประกันให้คุณ

| สิ่งที่เกิดขึ้นได้จริง | ใครเป็นคนรับผิดชอบ |
|---|---|
| แอดมินองค์กรกดลบ repository โดยไม่ได้ตั้งใจ | คุณเอง — ไม่ใช่ GitHub |
| พนักงานที่ลาออกลบ repository ก่อนออกจากงาน | คุณเอง |
| บัญชีองค์กรถูก suspend เพราะ billing ผิดพลาด/ละเมิดนโยบาย | คุณเอง (จนกว่าจะแก้ปัญหากับฝ่าย support เสร็จ) |
| Force push ทับประวัติเก่าโดยไม่ได้ตั้งใจ แล้วไม่มีใครสังเกตทัน | คุณเอง |
| บัญชีถูก hack แล้วผู้บุกรุกลบ/เปลี่ยนแปลงข้อมูล | คุณเอง (แม้ platform จะไม่ได้ถูก hack ก็ตาม) |
| Ransomware โจมตี GitHub repository (มีเคสจริงในปี 2019 ที่แฮกเกอร์ลบโค้ดแล้วเรียกค่าไถ่) | คุณเอง |
| ผู้ให้บริการปิดกิจการ/หยุดให้บริการในบางประเทศ | คุณเอง |

จุดสำคัญคือ **GitHub ป้องกันคุณจาก "ฮาร์ดแวร์พัง" ได้ดีมาก แต่ป้องกันคุณจาก "ความผิดพลาดของมนุษย์" หรือ "การกระทำที่ตั้งใจทำลาย" ไม่ได้เลย** เพราะจากมุมมองของระบบ การที่ผู้ดูแล repository ที่มีสิทธิ์ถูกต้องกด "Delete this repository" คือการกระทำที่ถูกต้องตามสิทธิ์ ระบบไม่มีทางรู้ได้ว่านี่คือความผิดพลาดหรือความตั้งใจ

### ตัวอย่างเหตุการณ์จริงที่เคยเกิดขึ้น

- **GitLab.com (มกราคม 2017):** วิศวกรของ GitLab ลบฐานข้อมูล production โดยเข้าใจผิดว่ากำลังทำงานกับเครื่อง staging ระบบ backup 5 ชั้นที่มีอยู่ **ใช้งานไม่ได้จริงถึง 4 ใน 5 ชั้น** เพราะไม่เคยถูกทดสอบ restore มาก่อน สุดท้ายกู้คืนข้อมูลได้จาก snapshot ที่เก่าที่สุดเท่าที่หาได้ ทำให้ข้อมูลผู้ใช้ (issues, comments, merge requests) หายไปประมาณ 6 ชั่วโมง — เหตุการณ์นี้กลายเป็นกรณีศึกษาคลาสสิกด้าน DR ที่สอนเราว่า **แม้แต่บริษัทที่ทำธุรกิจ Git ก็ยังพลาดเรื่อง backup ได้**
- มีกรณีศึกษาจำนวนมากที่นักพัฒนา freelance หรือบริษัท startup โดนลบ organization บน GitHub เนื่องจากบัตรเครดิตหมดอายุและ billing ล้มเหลวต่อเนื่องหลายเดือน จนบัญชีถูกปิดและใช้เวลานานในการติดต่อขอคืนสิทธิ์

### สรุปหลักการของ Step นี้

> **"Git repository ของคุณอยู่บน infrastructure ที่แข็งแกร่ง ไม่ได้แปลว่ามันมี backup ที่แข็งแกร่งตามไปด้วย"**

Availability (ระบบไม่ล่ม) กับ Backup (มีสำเนาข้อมูลแยกต่างหากที่กู้คืนได้อิสระจากระบบหลัก) เป็นคนละเรื่องกัน และเป็นความรับผิดชอบคนละฝ่ายกัน — นี่คือเหตุผลที่ทีมวิศวกรมืออาชีพทุกทีมต้องมีกลยุทธ์ backup ของตัวเอง ไม่พึ่งพา SaaS เพียงอย่างเดียว ไม่ว่า SaaS นั้นจะน่าเชื่อถือแค่ไหนก็ตาม

---

## Step 832: Single Point of Failure ที่เกิดขึ้นจริงในโลกความเป็นจริง

**Single Point of Failure (SPOF)** คือจุดใดจุดหนึ่งในระบบที่ถ้าเสียหายแล้ว **ทำให้ทั้งระบบล่มหรือข้อมูลหายทั้งหมด** โดยไม่มีทางเลือกสำรอง

แม้ Git เองจะเป็น Distributed VCS ที่ไม่มี SPOF ในตัว (ทุกคนมีสำเนาประวัติเต็ม) แต่ในทางปฏิบัติ ทีมส่วนใหญ่กลับสร้าง SPOF ขึ้นมาเองโดยไม่รู้ตัว เพราะพึ่งพา remote repository เพียงที่เดียวเป็นแหล่งความจริงหนึ่งเดียว (single source of truth) โดยไม่มีสำเนาอื่นเลย

### สถานการณ์ SPOF ที่พบได้จริง

#### 1. บัญชีถูกลบโดยไม่ได้ตั้งใจ (Accidental Deletion)

```
สถานการณ์: Admin ต้องการลบ repository ทดลอง "myapp-test"
แต่พิมพ์ชื่อผิดในกล่องยืนยัน แล้วดัน confirm ลบ "myapp-production" แทน
```

GitHub และ GitLab มีระบบ "soft delete" ที่เก็บ repository ไว้ชั่วคราวก่อนลบถาวร (เช่น GitHub เก็บไว้ประมาณ 90 วันสำหรับบาง plan และมี "Restore deleted repository" ให้ใช้ในบางเงื่อนไข) แต่:

- ฟีเจอร์นี้**ไม่ได้มีในทุก plan**
- ระยะเวลาเก็บมี**จำกัด**
- Issues, Pull Requests, Wiki, Settings บางส่วนอาจกู้คืนไม่ได้ครบ 100%
- ถ้าเป็น self-hosted GitLab ที่ตั้งค่าเองและไม่ได้เปิด feature นี้ไว้ อาจไม่มีการกู้คืนเลย

#### 2. องค์กรถูก Suspend

```
สถานการณ์: ทีม Compliance ของแพลตฟอร์มตรวจพบกิจกรรมที่ดูน่าสงสัย
(เช่น มี bot สร้าง repository จำนวนมากผิดปกติ หรือมีการรายงานเนื้อหาละเมิดนโยบาย)
จึงระงับบัญชีองค์กรทั้งหมดทันทีเพื่อสอบสวน
```

ในกรณีนี้ **repository ทั้งหมดขององค์กรจะเข้าถึงไม่ได้ทันที** ทั้งที่ไม่มีอะไรเสียหายในทางเทคนิค แต่ก็ทำให้ทีมทำงานต่อไม่ได้จนกว่าจะแก้ปัญหากับฝ่าย support เสร็จ ซึ่งอาจใช้เวลาหลายวันถึงหลายสัปดาห์

#### 3. ผู้ให้บริการเกิดปัญหาระบบระดับใหญ่ (Provider-wide Outage)

GitHub, GitLab.com เคยมี incident ระดับที่ทำให้ผู้ใช้ทั่วโลก push/pull ไม่ได้นานหลายชั่วโมง แม้ปกติแล้ว SaaS เหล่านี้จะมี uptime สูงมาก (99.9%+) แต่ในช่วงเวลาวิกฤต เช่น deadline ส่งงานสำคัญ หรือช่วง incident response ของระบบ production การเข้าถึงไม่ได้แม้เพียงไม่กี่ชั่วโมงก็สร้างความเสียหายทางธุรกิจได้มาก

#### 4. การโจมตีทางไซเบอร์ต่อบัญชีขององค์กร

หากบัญชีที่มีสิทธิ์ Owner/Admin ถูกขโมย (เช่น ผ่าน phishing หรือ credential stuffing) ผู้บุกรุกสามารถ:

- ลบ repository ทั้งหมด
- Force push ทับประวัติด้วยโค้ดที่ถูกดัดแปลง
- เปลี่ยนสิทธิ์การเข้าถึงเพื่อล็อกทีมงานจริงออกจากระบบ

มีเคสจริงในอุตสาหกรรมที่แฮกเกอร์เข้าถึง repository ได้ แล้วลบโค้ดทั้งหมดพร้อมทิ้งข้อความเรียกค่าไถ่ (คล้ายกับ ransomware แต่ทำผ่าน Git แทนที่จะเข้ารหัสไฟล์)

#### 5. ความผิดพลาดจากคนในทีมเอง

- รัน `git push --force` ทับ branch `main` โดยไม่ได้ตั้งใจ (โดยเฉพาะถ้าไม่มี branch protection)
- รัน script อัตโนมัติที่มี bug แล้วไปลบ tag/branch จำนวนมากผ่าน API
- CI/CD pipeline ที่ตั้งค่าผิดพลาดไปรัน `git push --mirror` จาก source ที่ผิด ทำให้ข้อมูลใน remote ถูกเขียนทับ

### ตารางสรุปความเสี่ยงและระดับผลกระทบ

| ความเสี่ยง | โอกาสเกิด | ผลกระทบ | ป้องกันได้ด้วย |
|---|---|---|---|
| ลบ repo โดยไม่ตั้งใจ | ปานกลาง | สูงมาก | Backup ภายนอก + Branch protection |
| องค์กรถูก suspend | ต่ำ | สูง | Backup ภายนอก + ติดต่อ support ล่วงหน้า |
| Provider outage | ต่ำ | ปานกลาง (ชั่วคราว) | Multi-remote + local mirror |
| บัญชีถูก hack | ปานกลาง | สูงมาก | 2FA + Backup ภายนอก + Audit log |
| Force push ทับประวัติ | สูง (ถ้าไม่มี protection) | ปานกลาง-สูง | Branch protection rules + backup |

### บทเรียนสำคัญ

> **"Distributed" ไม่ได้แปลว่า "ปลอดภัยโดยอัตโนมัติ" — ถ้าทีมของคุณมี remote เดียว และไม่มีใคร clone แบบเต็มไว้ที่อื่นเลย คุณก็สร้าง Centralized VCS แบบมี SPOF ขึ้นมาเองโดยไม่รู้ตัว แม้จะใช้ Git ก็ตาม**

Step ถัดไปเราจะเริ่มแก้ปัญหานี้ด้วยกลยุทธ์ backup ที่ง่ายที่สุดแต่ทรงพลังที่สุด

---

## Step 833: กลยุทธ์ backup พื้นฐานที่สุด — Mirror Clone

ก่อนจะไปถึงเครื่องมืออัตโนมัติซับซ้อน เราต้องเข้าใจหลักการพื้นฐานที่สุดก่อน นั่นคือ **การทำสำเนา repository แบบสมบูรณ์ (mirror) เก็บไว้ในที่ที่แยกจากระบบหลักโดยสิ้นเชิง**

### หลักการ 3-2-1 ของการ Backup (ประยุกต์จากโลก Backup ทั่วไป)

หลักการ **3-2-1 Backup Rule** เป็นมาตรฐานสากลที่ใช้กับข้อมูลทุกประเภท ไม่ใช่แค่ Git:

- **3** สำเนาของข้อมูล (ต้นฉบับ 1 + สำเนา 2)
- **2** ชนิดของสื่อเก็บข้อมูลที่แตกต่างกัน (เช่น cloud storage หนึ่ง + external drive หนึ่ง)
- **1** สำเนาที่เก็บไว้นอกสถานที่ (off-site) เพื่อป้องกันภัยพิบัติทางกายภาพหรือปัญหาที่กระทบผู้ให้บริการรายเดียว

ประยุกต์กับ Git:

```
สำเนาที่ 1 (ต้นฉบับ): GitHub (ที่ทีมใช้งานประจำวัน)
สำเนาที่ 2: GitLab.com หรือ Bitbucket (ผู้ให้บริการคนละเจ้า)
สำเนาที่ 3: Bare repository บนเซิร์ฟเวอร์/NAS ภายในองค์กร หรือ cloud storage เช่น S3
```

ถ้าทำครบตามหลักการนี้ ต่อให้ GitHub ล่มทั้งระบบ องค์กรถูก suspend หรือมีคนลบ repo โดยไม่ตั้งใจ **ก็ยังมีสำเนาที่กู้คืนได้ทันทีจากที่อื่น**

### ทำไมต้องเป็น "Mirror" ไม่ใช่แค่ "Clone" ธรรมดา

`git clone` ปกติจะดึงเฉพาะ branch ที่ทำงานอยู่และ refs ที่จำเป็น แต่ไม่ได้เก็บทุกอย่างแบบสมบูรณ์ 100% เช่น:

- Branch อื่น ๆ ที่ไม่ใช่ default branch (ถ้าไม่ระบุ `--all` มันจะไม่ track ให้อัตโนมัติ)
- Tags ทั้งหมด
- Refs พิเศษ เช่น `refs/pull/*` (บน GitHub ที่เก็บ PR ไว้เป็น ref พิเศษ) หรือ `refs/merge-requests/*` (บน GitLab)
- การตั้งค่า remote-tracking แบบเต็ม

ในขณะที่ **mirror clone** จะคัดลอก **ทุก reference ในระบบ (branches, tags, notes, และ refs พิเศษทั้งหมด)** แบบไบต์ต่อไบต์ ทำให้เป็นสำเนาที่ "สมบูรณ์" จริง ๆ ในระดับ Git object

### แนวคิดของ Bare Repository

ก่อนไปต่อ ต้องเข้าใจ **Bare Repository** ก่อน — คือ repository ที่ **ไม่มี working directory** มีแต่โฟลเดอร์ `.git` (หรือเนื้อหาข้างในของ `.git` ที่ถูกวางไว้ตรง ๆ โดยไม่มีโฟลเดอร์ `.git` ห่อไว้อีกชั้น) เหมาะสำหรับใช้เป็น **ที่เก็บกลาง** ไม่ใช่ที่สำหรับแก้ไขไฟล์โดยตรง

```
repo ปกติ (non-bare):
myproject/
├── .git/          ← ข้อมูล Git ทั้งหมดอยู่ที่นี่
├── src/
├── README.md
└── ...            ← ไฟล์ที่แก้ไขได้จริง (working directory)

bare repository:
myproject.git/
├── HEAD
├── config
├── objects/
├── refs/
└── ...            ← ไม่มี working directory เลย มีแต่ข้อมูล Git ล้วน ๆ
```

Mirror clone จะสร้าง bare repository เสมอ ซึ่งเหมาะกับการใช้เป็น "backup" มาก เพราะมันคือข้อมูล Git ล้วน ๆ ไม่มีไฟล์ทำงานที่ไม่จำเป็นมาปน ประหยัดพื้นที่ และพร้อมใช้เป็น remote ให้คนอื่น push/pull ได้ทันทีถ้าจำเป็น

### ระดับความสมบูรณ์ของ backup แต่ละแบบ

| วิธี backup | ครอบคลุมแค่ไหน | เหมาะกับ |
|---|---|---|
| `git clone` ปกติ (แค่ default branch) | ต่ำ | ใช้พัฒนาต่อ ไม่เหมาะกับ backup |
| `git clone --all` (ทุก branch แต่ไม่ครบ refs พิเศษ) | ปานกลาง | ดีกว่าเดิม แต่ยังไม่สมบูรณ์ |
| `git clone --mirror` | สูงมาก (ครบทุก ref รวม tags/notes) | **มาตรฐาน backup ของ Git object** |
| Export ผ่าน API (issues, PR, wiki) | เฉพาะ metadata นอก Git object | ต้องทำเพิ่มเติม (ดู Step 836) |

Step ถัดไปเราจะลงมือทำจริงกับ `git clone --mirror` และเขียน script ให้ sync backup อัตโนมัติแบบต่อเนื่อง

---

## Step 834: `git clone --mirror` และการตั้งค่า sync backup อัตโนมัติ

### 834.1 การสร้าง Mirror Clone ครั้งแรก

คำสั่งพื้นฐานที่สุด:

```bash
git clone --mirror git@github.com:myorg/myapp.git myapp-backup.git
```

ผลลัพธ์คือโฟลเดอร์ `myapp-backup.git` ซึ่งเป็น bare repository ที่มีทุก branch, ทุก tag, และทุก ref พิเศษของ `myapp` ครบถ้วน

ตรวจสอบได้ว่ามันคือ bare repository และมี refspec แบบ mirror จริงหรือไม่:

```bash
cd myapp-backup.git
git config --get remote.origin.fetch
```

ผลลัพธ์ที่ควรเห็น:

```
+refs/*:refs/*
```

ต่างจาก clone ปกติที่ refspec จะเป็น `+refs/heads/*:refs/remotes/origin/*` (ดึงมาเก็บใน namespace `refs/remotes/origin/` แทนที่จะ mirror ตรง ๆ ที่ `refs/*`) — นี่คือสิ่งที่ทำให้ mirror clone พิเศษกว่า clone ปกติ

### 834.2 การอัปเดต Mirror ให้ตรงกับต้นฉบับล่าสุด (Sync)

เมื่อมี mirror อยู่แล้ว ไม่จำเป็นต้อง clone ใหม่ทุกครั้ง แค่รันคำสั่งอัปเดตจากภายในโฟลเดอร์ mirror:

```bash
cd myapp-backup.git
git remote update
```

หรือใช้ `git fetch` แบบระบุ prune เพื่อให้ลบ ref ที่ถูกลบไปแล้วที่ต้นทางออกจาก mirror ด้วย (สำคัญมาก เพราะ mirror ที่ดีต้อง "สะท้อน" สถานะปัจจุบันของต้นฉบับ ไม่ใช่สะสม ref เก่าที่ถูกลบไปแล้วทิ้งไว้):

```bash
cd myapp-backup.git
git fetch --prune origin
```

เนื่องจาก mirror repo ถูกตั้งค่า refspec เป็น `+refs/*:refs/*` อยู่แล้ว คำสั่งข้างต้นจะอัปเดตทุกอย่างให้ตรงกับต้นฉบับ 100% รวมถึงลบ branch/tag ที่ถูกลบไปแล้วออกจาก mirror ด้วย

> **ข้อควรระวัง:** การ prune หมายความว่าถ้ามีคนลบ branch สำคัญที่ต้นทางโดยไม่ได้ตั้งใจ แล้ว mirror sync ทำงานไปแล้ว **branch นั้นจะหายจาก mirror ด้วย** ดังนั้น mirror อย่างเดียวไม่ได้ป้องกัน "การลบผิดพลาดที่ sync ทันเวลา" — วิธีแก้คือทำ **versioned backup** ควบคู่กันไป (เช่น เก็บ snapshot ของ mirror แยกเป็นรอบ ๆ ไม่ทับของเก่าทันที) ซึ่งเราจะพูดถึงในการออกแบบ script ด้านล่าง

### 834.3 เขียน Script สำหรับ Sync อัตโนมัติ

ตัวอย่าง script แบบสมบูรณ์ที่ backup หลาย repository พร้อมกัน และเก็บ log:

```bash
#!/usr/bin/env bash
# backup-git-mirrors.sh
# สคริปต์สำหรับ sync mirror ของ repository หลายตัวไปยังที่เก็บ backup

set -euo pipefail

BACKUP_ROOT="/var/backups/git-mirrors"
LOG_FILE="/var/log/git-mirror-backup.log"
DATE_TAG=$(date +%Y%m%d-%H%M%S)

REPOS=(
  "git@github.com:myorg/myapp.git"
  "git@github.com:myorg/api-service.git"
  "git@github.com:myorg/infra-scripts.git"
)

mkdir -p "$BACKUP_ROOT"

log() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

log "=== เริ่มรอบ backup: $DATE_TAG ==="

for REPO_URL in "${REPOS[@]}"; do
  REPO_NAME=$(basename "$REPO_URL" .git)
  TARGET_DIR="$BACKUP_ROOT/${REPO_NAME}.git"

  if [ -d "$TARGET_DIR" ]; then
    log "อัปเดต mirror ที่มีอยู่แล้ว: $REPO_NAME"
    if git --git-dir="$TARGET_DIR" fetch --prune origin >>"$LOG_FILE" 2>&1; then
      log "  สำเร็จ: $REPO_NAME"
    else
      log "  ล้มเหลว: $REPO_NAME (ตรวจสอบ log ด้านบน)"
      continue
    fi
  else
    log "สร้าง mirror ใหม่: $REPO_NAME"
    if git clone --mirror "$REPO_URL" "$TARGET_DIR" >>"$LOG_FILE" 2>&1; then
      log "  สำเร็จ: $REPO_NAME"
    else
      log "  ล้มเหลว: $REPO_NAME"
      continue
    fi
  fi

  # ตรวจสอบความสมบูรณ์ของ object database หลัง sync ทุกครั้ง
  if git --git-dir="$TARGET_DIR" fsck --full >>"$LOG_FILE" 2>&1; then
    log "  fsck ผ่าน: $REPO_NAME"
  else
    log "  คำเตือน: fsck พบปัญหาใน $REPO_NAME กรุณาตรวจสอบด่วน"
  fi
done

log "=== จบรอบ backup: $DATE_TAG ==="
```

จุดสำคัญของ script นี้:

1. **`set -euo pipefail`** — ทำให้ script หยุดทันทีเมื่อมีคำสั่งล้มเหลว ไม่ปล่อยผ่านแบบเงียบ ๆ
2. **`git fsck --full`** หลัง sync ทุกครั้ง — ตรวจสอบว่า object database ไม่เสียหาย (corrupt) เพราะ backup ที่เสียหายก็ไม่ต่างอะไรจากไม่มี backup
3. **แยก log ชัดเจน** — เมื่อเกิดปัญหาสามารถย้อนดูได้ว่า repo ไหน sync ล้มเหลวตอนไหน
4. **ใช้ `git --git-dir=`** แทนการ `cd` เข้าไปในแต่ละโฟลเดอร์ ทำให้ script อ่านง่ายและปลอดภัยกว่า

### 834.4 ตั้งค่าให้รันอัตโนมัติด้วย Cron

แก้ไข crontab ของผู้ใช้ที่มีสิทธิ์รัน backup (แนะนำให้ใช้ user เฉพาะสำหรับงาน backup ไม่ใช้ root โดยตรง):

```bash
crontab -e
```

เพิ่มบรรทัดนี้เพื่อให้รันทุก 6 ชั่วโมง:

```
0 */6 * * * /usr/local/bin/backup-git-mirrors.sh
```

หรือถ้าต้องการรันทุกคืนตอนตี 2 (เวลาระบบของเซิร์ฟเวอร์):

```
0 2 * * * /usr/local/bin/backup-git-mirrors.sh
```

### 834.5 ทางเลือกสมัยใหม่: CI/CD Scheduled Pipeline

แทนที่จะพึ่งพา cron บนเซิร์ฟเวอร์เดียว (ซึ่งตัวมันเองก็เป็น SPOF ได้เช่นกัน) หลายทีมเลือกใช้ **Scheduled Pipeline** บน GitHub Actions หรือ GitLab CI/CD แทน เพราะรันบน infrastructure ของแพลตฟอร์มเอง ไม่ต้องดูแลเซิร์ฟเวอร์เพิ่ม

ตัวอย่าง GitHub Actions workflow ที่รันทุกวันเพื่อ mirror ไปเก็บใน GitLab (คนละผู้ให้บริการ):

```yaml
name: Mirror Backup to GitLab

on:
  schedule:
    - cron: '0 3 * * *'   # ทุกวันตี 3 (UTC)
  workflow_dispatch: {}     # ให้กดรันเองได้ด้วยจาก UI

jobs:
  mirror:
    runs-on: ubuntu-latest
    steps:
      - name: Clone mirror จากต้นฉบับ
        run: git clone --mirror https://github.com/myorg/myapp.git mirror-repo

      - name: Push mirror ไปยัง GitLab backup
        working-directory: mirror-repo
        env:
          GITLAB_TOKEN: ${{ secrets.GITLAB_BACKUP_TOKEN }}
        run: |
          git push --mirror "https://oauth2:${GITLAB_TOKEN}@gitlab.com/myorg-backup/myapp.git"
```

ข้อดีของวิธีนี้คือ **ไม่ต้องมีเซิร์ฟเวอร์ของตัวเองเลย** และประวัติการรันของแต่ละรอบ backup ถูกเก็บไว้ใน Actions log โดยอัตโนมัติ ทำให้ตรวจสอบย้อนหลังได้ง่ายว่ารอบไหนสำเร็จหรือล้มเหลว

เราจะเจาะลึกเรื่อง GitHub Actions และ scheduled workflow แบบละเอียดมากขึ้นใน **Part 66-70 (เฟส 7: CI/CD เต็มรูปแบบ)** แต่ ณ ตอนนี้ให้เข้าใจหลักการก่อนว่า **การ backup ก็คือ pipeline ประเภทหนึ่ง สามารถทำให้เป็นอัตโนมัติได้เหมือนงาน DevOps อื่น ๆ**

---

## Step 835: Multi-Remote Strategy — push โค้ดไปหลาย remote พร้อมกัน

Mirror backup แบบ scheduled (Step 834) มีข้อจำกัดหนึ่งข้อ คือ **มันไม่ real-time** — ถ้า sync ทุก 6 ชั่วโมง แล้วเกิดเหตุร้ายขึ้นในนาทีที่ 5 หลัง sync รอบล่าสุด งานที่ทำไปในช่วง 5 ชั่วโมง 55 นาทีนั้นอาจหายไป

**Multi-Remote Strategy** แก้ปัญหานี้โดยให้ **นักพัฒนา push ไปยัง remote มากกว่าหนึ่งที่พร้อมกันทุกครั้งที่ push** ทำให้ backup อัปเดตทันทีที่มีการ push จริง ไม่ต้องรอรอบ schedule

### 835.1 แนวคิดพื้นฐาน: Git รองรับหลาย Remote ในตัวอยู่แล้ว

Repository เดียวสามารถมี remote ได้หลายตัว:

```bash
git remote add origin git@github.com:myorg/myapp.git
git remote add backup git@gitlab.com:myorg-backup/myapp.git
```

ตรวจสอบรายการ remote ทั้งหมด:

```bash
git remote -v
```

```
origin   git@github.com:myorg/myapp.git (fetch)
origin   git@github.com:myorg/myapp.git (push)
backup   git@gitlab.com:myorg-backup/myapp.git (fetch)
backup   git@gitlab.com:myorg-backup/myapp.git (push)
```

วิธีนี้ต้อง push สองครั้งแยกกัน:

```bash
git push origin main
git push backup main
```

ซึ่งมีข้อเสียชัดเจนคือ **ถ้าคนลืม push ไปที่ `backup` ก็จะทำให้ backup ไม่ up-to-date** — ต้องพึ่งพาวินัยของคนอย่างเดียว ไม่น่าเชื่อถือพอสำหรับ production

### 835.2 วิธีที่ดีกว่า: ตั้งค่า Push URL หลายตัวใน Remote เดียว

Git อนุญาตให้ remote ตัวเดียวมี **push URL ได้หลายตัว** ทำให้คำสั่ง `git push origin` เพียงครั้งเดียว ส่งไปยังหลายปลายทางพร้อมกันโดยอัตโนมัติ:

```bash
# ตั้งค่าเริ่มต้น: origin ชี้ไปที่ GitHub ตามปกติ
git remote add origin git@github.com:myorg/myapp.git

# เพิ่ม push URL ตัวที่สองเข้าไปใน remote เดิมชื่อ origin
git remote set-url --add --push origin git@github.com:myorg/myapp.git
git remote set-url --add --push origin git@gitlab.com:myorg-backup/myapp.git
```

ตรวจสอบผลลัพธ์:

```bash
git remote -v
```

```
origin  git@github.com:myorg/myapp.git (fetch)
origin  git@github.com:myorg/myapp.git (push)
origin  git@gitlab.com:myorg-backup/myapp.git (push)
```

จากนี้ไปแค่รัน:

```bash
git push origin main
```

Git จะ**ส่งไปยังทั้งสอง URL โดยอัตโนมัติในคำสั่งเดียว** ไม่มีทางลืม เพราะเป็นพฤติกรรมเริ่มต้นของคำสั่ง push ปกติที่ทุกคนในทีมใช้อยู่แล้ว

> **หมายเหตุสำคัญ:** วิธีนี้ push แบบ**เรียงลำดับทีละ URL ไม่ใช่พร้อมกันจริง** และถ้า URL ตัวใดตัวหนึ่ง push ไม่สำเร็จ (เช่น เน็ตหลุดตอนกำลัง push ไป GitLab) Git จะรายงาน error แต่**ไม่ rollback** URL ที่ push สำเร็จไปแล้ว ทำให้บาง remote อาจ "ตามหลัง" remote อื่นชั่วคราว จึงยังจำเป็นต้องมี job สำหรับตรวจสอบและ sync ซ้ำเป็นระยะอยู่ดี (คล้าย Step 834) เพื่อจับกรณีที่ push บางส่วนล้มเหลว

### 835.3 การบังคับใช้ Multi-Remote ทั้งทีมด้วย Git Hook

การพึ่งพาให้แต่ละคนตั้งค่า `remote.origin.pushurl` เองมีความเสี่ยงที่บางคนจะลืมตั้งค่า วิธีที่แข็งแรงกว่าคือทำผ่าน **server-side hook** (เช่น `pre-receive` hook บนเซิร์�ฟเวอร์ GitLab self-hosted) หรือใช้ **CI/CD pipeline ที่ trigger ทุกครั้งที่มี push** ให้ทำหน้าที่ sync ไปยัง remote สำรองแทน ซึ่งเชื่อถือได้มากกว่าการตั้งค่าฝั่ง client เพราะไม่ขึ้นกับการตั้งค่าเครื่องของแต่ละคน

ตัวอย่างแนวทาง (แนวคิด ไม่ใช่ production-ready เต็มรูปแบบ) ด้วย GitHub Actions ที่ trigger ทุก push เข้า `main`:

```yaml
name: Real-time Mirror to Backup Remote

on:
  push:
    branches: ['**']
  create:     # branch/tag ใหม่ถูกสร้าง
  delete:     # branch/tag ถูกลบ

jobs:
  sync-backup:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout แบบเต็ม (ไม่ shallow)
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Push การเปลี่ยนแปลงไปยัง backup remote ทันที
        env:
          BACKUP_TOKEN: ${{ secrets.BACKUP_REMOTE_TOKEN }}
        run: |
          git remote add backup "https://oauth2:${BACKUP_TOKEN}@gitlab.com/myorg-backup/myapp.git"
          git push --mirror backup
```

วิธีนี้ทำให้ backup ตามทันแทบจะทันทีทุกครั้งที่มีคน push โค้ดเข้า repository หลัก โดยไม่ต้องพึ่งพาวินัยของนักพัฒนาแต่ละคนเลย

### 835.4 เปรียบเทียบกลยุทธ์ทั้งหมด

| กลยุทธ์ | ความ Real-time | ความน่าเชื่อถือ | ความซับซ้อนในการตั้งค่า |
|---|---|---|---|
| Scheduled mirror (cron/CI) | ต่ำ (ตาม schedule) | สูง (ไม่ขึ้นกับคน) | ต่ำ |
| Multi push URL (client-side) | สูง | ปานกลาง (ขึ้นกับการตั้งค่าเครื่องแต่ละคน) | ต่ำมาก |
| CI/CD trigger ทุก push (server-side) | สูงมาก | สูงมาก | ปานกลาง |

ในทางปฏิบัติ ทีมมืออาชีพมักใช้**ทั้งสองแบบร่วมกัน**: multi-remote หรือ CI trigger สำหรับความ real-time และ scheduled mirror job สำหรับเป็น safety net คอยตรวจสอบซ้ำว่าทุกอย่าง sync กันจริงอย่างสม่ำเสมอ

---

## Step 836: Backup metadata ที่ไม่ใช่โค้ด (Issues, PR, Wiki) ผ่าน API

นี่คือจุดที่คนทำ backup พลาดบ่อยที่สุด เพราะคิดว่า "mirror repo ครบแล้ว = backup ครบแล้ว" ซึ่ง**ไม่จริง**

### 836.1 อะไรบ้างที่อยู่ใน Git Object และอะไรที่ไม่อยู่

| ข้อมูล | อยู่ใน Git object ไหม | สำรองด้วย mirror clone ได้ไหม |
|---|---|---|
| ซอร์สโค้ด, commit history | ใช่ | ได้ |
| Branches, Tags | ใช่ | ได้ |
| Git Notes | ใช่ | ได้ (ถ้า mirror ครบ) |
| **Wiki** (บาง platform) | ใช่ (เก็บเป็น git repo แยกต่างหาก) | ได้ **แต่ต้อง mirror แยกต่างหาก** |
| **Issues** | **ไม่** (เก็บในฐานข้อมูลของแพลตฟอร์ม) | **ไม่ได้** |
| **Pull Requests / Merge Requests** | **ไม่** (metadata เก็บในฐานข้อมูล แม้ commit จะอยู่ใน git object) | **ไม่ได้ครบ** |
| **Comments, Reviews** | **ไม่** | **ไม่ได้** |
| **Labels, Milestones, Projects boards** | **ไม่** | **ไม่ได้** |
| **CI/CD configuration secrets, Settings, Webhooks** | **ไม่** | **ไม่ได้** |
| **Releases (ไฟล์แนบ, release notes)** | บางส่วน (tag อยู่ใน git แต่ release note/asset ไม่อยู่) | **ไม่ได้ครบ** |

จุดสำคัญคือ **Wiki ของ GitHub และ GitLab นั้นจริง ๆ แล้วเป็น Git repository อีกตัวหนึ่งแยกต่างหาก** ไม่ใช่ส่วนหนึ่งของ repository หลัก ต้อง mirror แยก:

```bash
# Wiki ของ GitHub มี URL แยกต่างหากในรูปแบบนี้
git clone --mirror git@github.com:myorg/myapp.wiki.git myapp-wiki-backup.git
```

แต่ **Issues, Pull Requests, Comments, Labels ไม่ใช่ Git object เลย** — มันถูกเก็บในฐานข้อมูลเชิงสัมพันธ์ (relational database) ของแพลตฟอร์มเอง (เช่น MySQL/PostgreSQL ที่ GitHub/GitLab ใช้ภายใน) ดังนั้น**ไม่มีทางที่ `git clone` แบบไหนจะดึงข้อมูลเหล่านี้ออกมาได้เลย** ต้องใช้ **API ของแพลตฟอร์ม** เท่านั้น

### 836.2 การ Export Issues และ Pull Requests ผ่าน GitHub API

ตัวอย่างการใช้ **GitHub CLI (`gh`)** ซึ่งเป็นเครื่องมือ official ที่คลุม REST API และ GraphQL API ของ GitHub ไว้ให้แล้ว:

```bash
# Export issue ทั้งหมด (รวมที่ปิดแล้ว) เป็นไฟล์ JSON
gh issue list \
  --repo myorg/myapp \
  --state all \
  --limit 1000 \
  --json number,title,body,state,labels,assignees,comments,createdAt,closedAt \
  > issues-backup-$(date +%Y%m%d).json
```

```bash
# Export pull request ทั้งหมด
gh pr list \
  --repo myorg/myapp \
  --state all \
  --limit 1000 \
  --json number,title,body,state,author,mergedAt,reviews,comments \
  > pull-requests-backup-$(date +%Y%m%d).json
```

สำหรับข้อมูลที่ซับซ้อนกว่านั้น (เช่น comment ทั้งหมดของแต่ละ issue แบบละเอียด) ต้องเรียก REST API โดยตรงแบบวนลูปตาม pagination:

```bash
#!/usr/bin/env bash
# export-issues-with-comments.sh
set -euo pipefail

OWNER="myorg"
REPO="myapp"
OUTPUT_DIR="./backup-metadata/$(date +%Y%m%d)"
mkdir -p "$OUTPUT_DIR"

PAGE=1
while :; do
  RESPONSE=$(gh api "repos/${OWNER}/${REPO}/issues?state=all&per_page=100&page=${PAGE}")
  COUNT=$(echo "$RESPONSE" | jq 'length')

  if [ "$COUNT" -eq 0 ]; then
    break
  fi

  echo "$RESPONSE" > "${OUTPUT_DIR}/issues-page-${PAGE}.json"
  echo "ดึงหน้า ${PAGE} สำเร็จ (${COUNT} รายการ)"
  PAGE=$((PAGE + 1))
done

echo "Export issues ทั้งหมดเสร็จสิ้นที่ ${OUTPUT_DIR}"
```

### 836.3 GitHub Migrations API — วิธี Export แบบครบวงจรที่สุด

สำหรับการ export แบบครอบคลุมที่สุดในคราวเดียว GitHub มี **Migrations API** ซึ่งออกแบบมาสำหรับการย้ายข้อมูลทั้งหมด (repository, issues, PR, wiki, releases, ไฟล์แนบ) ออกจาก GitHub เป็นไฟล์ archive เดียว:

```bash
# เริ่มกระบวนการ migration/export สำหรับ repository ที่ต้องการ
gh api -X POST /orgs/myorg/migrations \
  -f repositories[]="myorg/myapp" \
  -F lock_repositories=false \
  -F exclude_git_data=false

# ตรวจสอบสถานะการ export (จะได้ migration id กลับมาจากคำสั่งด้านบน)
gh api /orgs/myorg/migrations/MIGRATION_ID

# เมื่อสถานะเป็น "exported" แล้ว ดาวน์โหลดไฟล์ archive
gh api /orgs/myorg/migrations/MIGRATION_ID/archive > migration-archive.tar.gz
```

ไฟล์ archive ที่ได้จะมีทั้ง git data แบบเต็ม (bundle) และไฟล์ JSON ของ issues, comments, PR, milestones, labels, wiki ครบถ้วนในไฟล์เดียว เหมาะกับการเก็บเป็น **cold backup รายเดือน/รายไตรมาส** ควบคู่ไปกับ mirror clone และ API export แบบละเอียดที่ทำถี่กว่า

### 836.4 GitLab: การ Export Project

GitLab มีฟีเจอร์ทำนองเดียวกันในตัว UI และผ่าน API เรียกว่า **Project Export**:

```bash
# เริ่ม export project ผ่าน API (ต้องมี GitLab personal access token)
curl --request POST \
  --header "PRIVATE-TOKEN: <your-token>" \
  "https://gitlab.com/api/v4/projects/<project-id>/export"

# ตรวจสอบสถานะ
curl --header "PRIVATE-TOKEN: <your-token>" \
  "https://gitlab.com/api/v4/projects/<project-id>/export"

# เมื่อสถานะเป็น finished แล้ว ดาวน์โหลดไฟล์
curl --header "PRIVATE-TOKEN: <your-token>" \
  --output project-export.tar.gz \
  "https://gitlab.com/api/v4/projects/<project-id>/export/download"
```

ไฟล์ export ของ GitLab จะรวม repository, issues, merge requests, milestones, snippets, CI/CD configuration ไว้ในไฟล์เดียว และสามารถ**นำไป import กลับเข้า GitLab instance อื่นได้โดยตรง** (ไม่ว่าจะเป็น GitLab.com หรือ self-hosted) ทำให้เหมาะมากสำหรับสถานการณ์ disaster recovery แบบเต็มรูปแบบ

### 836.5 ตารางสรุปกลยุทธ์การ Backup แบบครบวงจร

| ชั้นข้อมูล | เครื่องมือที่ใช้ backup | ความถี่ที่แนะนำ |
|---|---|---|
| Source code + history (branches, tags) | `git clone --mirror` / multi-remote | Real-time ถึงทุก 6 ชม. |
| Wiki | `git clone --mirror` (URL แยก) | ทุก 6-24 ชม. |
| Issues, PR, Comments, Labels | REST/GraphQL API script | รายวัน |
| การตั้งค่า repo, Webhooks, Branch protection | Export ผ่าน API หรือ Infrastructure-as-Code (Terraform) | รายวัน/เมื่อมีการเปลี่ยนแปลง |
| ภาพรวมทั้งหมด (full snapshot) | Migrations API (GitHub) / Project Export (GitLab) | รายสัปดาห์/รายเดือน |

> **หลักการสำคัญ:** การ backup ที่สมบูรณ์ต้องมองข้อมูลเป็น **หลายชั้น (layers)** ไม่ใช่แค่ "โค้ด" อย่างเดียว เพราะในโลกจริง เมื่อเกิดภัยพิบัติแล้วต้องกู้คืน ทีมมักต้องการ **ประวัติการสนทนาใน issue, บริบทของ pull request, และเหตุผลของการตัดสินใจ** ไม่ใช่แค่ตัวโค้ดล่าสุดเพียงอย่างเดียว

---

## Step 837: Disaster Recovery Plan — RTO และ RPO ประยุกต์ใช้กับ Git Infrastructure

การมี backup อย่างเดียวไม่พอ — ทีมต้องมี **แผนการกู้คืนจากภัยพิบัติ (Disaster Recovery Plan หรือ DR Plan)** ที่ชัดเจน เพื่อให้เมื่อเกิดเหตุจริง ทุกคนรู้ว่าต้องทำอะไร ตามลำดับไหน และคาดหวังผลลัพธ์แบบไหน

### 837.1 สองตัวชี้วัดหลักของ DR Plan: RTO และ RPO

#### RTO — Recovery Time Objective (เป้าหมายเวลาในการกู้คืน)

> **"เมื่อเกิดเหตุร้ายขึ้น เรายอมรับได้ว่าระบบจะใช้เวลานานสูงสุดเท่าไหร่ก่อนที่จะกลับมาใช้งานได้ปกติ"**

ตัวอย่าง: ถ้าตั้ง RTO ไว้ที่ **4 ชั่วโมง** หมายความว่าทีมต้องออกแบบกระบวนการกู้คืนให้เสร็จภายใน 4 ชั่วโมงนับจากตรวจพบปัญหา ไม่ว่าจะเกิดอะไรขึ้นก็ตาม

#### RPO — Recovery Point Objective (เป้าหมายจุดข้อมูลที่ยอมรับให้สูญเสียได้)

> **"เรายอมรับได้ว่าจะสูญเสียข้อมูลย้อนหลังไปสูงสุดกี่นาที/ชั่วโมง นับจากจุดที่เกิดเหตุ"**

ตัวอย่าง: ถ้าตั้ง RPO ไว้ที่ **1 ชั่วโมง** หมายความว่า backup ล่าสุดที่ใช้กู้คืนต้องมีอายุไม่เกิน 1 ชั่วโมงก่อนเกิดเหตุ — ถ้า sync backup ทุก 6 ชั่วโมง แต่ตั้ง RPO ไว้ที่ 1 ชั่วโมง แปลว่า**แผน backup ปัจจุบันไม่สอดคล้องกับเป้าหมายที่ตั้งไว้** ต้องปรับความถี่ของการ sync backup ให้ถี่ขึ้น (หรือใช้ multi-remote/CI trigger แบบ real-time ตาม Step 835)

### 837.2 ภาพประกอบความสัมพันธ์ระหว่าง RTO และ RPO

```
เวลา ────────────────────────────────────────▶

  Backup ล่าสุด          เหตุการณ์เกิดขึ้น        ระบบกลับมาใช้งานได้
       │                        │                        │
       │◀──────── RPO ─────────▶│◀──────── RTO ─────────▶│
       │   (ข้อมูลที่อาจสูญเสีย)   │   (เวลาที่ใช้กู้คืน)      │
       │                        │                        │
   ─────●────────────────────────●────────────────────────●─────
     Backup                   Incident                  Recovery
     Point                    เกิดขึ้น                   สมบูรณ์
```

- **RPO** วัดจาก "จุด backup ล่าสุด" ถึง "จุดที่เกิดเหตุ" → บอกว่าจะเสียข้อมูลไปกี่นาที/ชั่วโมง
- **RTO** วัดจาก "จุดที่เกิดเหตุ" ถึง "จุดที่ระบบกลับมาใช้งานได้ปกติ" → บอกว่าจะหยุดทำงานนานแค่ไหน

### 837.3 การกำหนด RTO/RPO ที่เหมาะสมกับ Git Infrastructure

การกำหนดค่าเหล่านี้ต้องพิจารณาจาก **ความสำคัญทางธุรกิจของ repository** ไม่ใช่ตั้งค่าเดียวกันหมดทุก repo:

| ประเภท Repository | RPO ที่แนะนำ | RTO ที่แนะนำ | เหตุผล |
|---|---|---|---|
| Production monorepo หลักของบริษัท | ใกล้ 0 (real-time) | < 1 ชั่วโมง | ทุกทีมพึ่งพา หยุดงานได้กระทบวงกว้าง |
| Microservice ที่ deploy บ่อย | < 1 ชั่วโมง | < 4 ชั่วโมง | สำคัญแต่มีทีมเดียวดูแล |
| Internal tools/scripts | < 24 ชั่วโมง | < 1 วันทำการ | ผลกระทบจำกัดวงแคบ |
| Documentation/Wiki | < 24 ชั่วโมง | < 1 วันทำการ | ไม่กระทบการทำงานเร่งด่วน |
| Archived/legacy projects | รายสัปดาห์ | ไม่เร่งด่วน | แทบไม่มีการเปลี่ยนแปลงแล้ว |

### 837.4 องค์ประกอบของ DR Plan ที่สมบูรณ์

DR Plan ที่ดีไม่ใช่แค่ "มี backup" แต่ต้องมีเอกสารที่ตอบคำถามเหล่านี้ได้ชัดเจน:

1. **Detection (การตรวจจับ)** — จะรู้ได้อย่างไรว่าเกิดเหตุร้ายขึ้น? (เช่น มี monitoring/alert แจ้งเตือนเมื่อ repository เข้าถึงไม่ได้)
2. **Roles & Responsibilities (ใครทำอะไร)** — ใครคือคนตัดสินใจเริ่มกระบวนการ DR? ใครมีสิทธิ์เข้าถึง backup?
3. **Communication Plan** — ต้องแจ้งใครบ้างเมื่อเกิดเหตุ (ทีม engineering, ผู้บริหาร, ลูกค้าถ้าจำเป็น)
4. **Recovery Steps (ขั้นตอนกู้คืนแบบละเอียด)** — เอกสารทีละขั้นตอนที่ **ทำตามได้แม้คนเขียนแผนไม่อยู่ในตอนนั้น** เช่น:
   - ตำแหน่งที่เก็บ backup ล่าสุดอยู่ที่ไหน (URL, credential เก็บที่ไหน)
   - คำสั่งที่ต้องรันเพื่อ restore ทีละขั้นตอน
   - วิธีตรวจสอบว่า restore สำเร็จและข้อมูลครบถ้วน
   - วิธีปรับ DNS/CI-CD/deploy key ให้ชี้ไปยัง repository ใหม่ (ถ้าต้องย้ายจริง)
5. **Post-Incident Review** — หลังกู้คืนเสร็จ ต้องมีการทบทวนว่าเกิดอะไรขึ้น ป้องกันไม่ให้เกิดซ้ำได้อย่างไร และปรับปรุงแผน DR ให้ดีขึ้น

### 837.5 ตัวอย่างเอกสาร DR Plan แบบย่อ

```markdown
# Disaster Recovery Plan: myapp Repository

## เป้าหมาย
- RPO: 1 ชั่วโมง
- RTO: 2 ชั่วโมง

## แหล่ง Backup
1. Primary: GitHub (github.com/myorg/myapp)
2. Secondary (real-time mirror): GitLab (gitlab.com/myorg-backup/myapp)
3. Tertiary (สแนปช็อตรายวัน): S3 bucket s3://myorg-backups/git/myapp/

## ผู้รับผิดชอบ
- ผู้ตัดสินใจเริ่ม DR: Head of Engineering
- ผู้ดำเนินการกู้คืน: DevOps on-call engineer
- Credential เก็บที่: บริษัท password manager (Vault path: secret/backup/gitlab-token)

## ขั้นตอนกู้คืน (กรณี GitHub repository หายทั้งหมด)
1. ยืนยันว่า GitHub repository ใช้งานไม่ได้จริง (ตรวจสอบ GitHub Status page)
2. แจ้งทีมผ่านช่อง #incident ทันที
3. Clone จาก secondary mirror:
   git clone --mirror git@gitlab.com:myorg-backup/myapp.git myapp-restore.git
4. สร้าง repository ใหม่บน GitHub (หรือรอกู้คืนจาก GitHub support)
5. Push กลับเข้า repository ใหม่:
   cd myapp-restore.git
   git push --mirror git@github.com:myorg/myapp-restored.git
6. ปรับ CI/CD, deploy key, webhook ให้ชี้ไปยัง repository ใหม่
7. แจ้งทีมทั้งหมดให้เปลี่ยน remote ในเครื่องตัวเอง
8. Verify: รัน test suite และ deploy pipeline เพื่อยืนยันว่าทุกอย่างทำงานปกติ

## Post-Incident
- บันทึกเหตุการณ์ใน incident log
- ทบทวนแผนทุก 6 เดือน หรือหลังเกิดเหตุจริงทุกครั้ง
```

เอกสารแบบนี้ควรถูกเก็บไว้ **นอก Git repository ที่มันปกป้อง** (เช่น ใน wiki ภายในองค์กร หรือ document ที่แชร์กันแบบแยกระบบ) เพราะถ้าเก็บไว้ใน repo เดียวกันกับที่มันปกป้อง แล้ว repo นั้นหายไปจริง ๆ ก็จะไม่มีใครหาแผนกู้คืนเจอ

---

## Step 838: การทดสอบ Restore จริง — หลักการที่คนมองข้ามบ่อยที่สุด

นี่คือหลักการที่สำคัญที่สุดใน Part นี้ และเป็นสิ่งที่ทีมส่วนใหญ่ในโลกจริงมองข้าม:

> **"Backup ที่ไม่เคยถูกทดสอบ Restore ไม่ใช่ Backup ที่เชื่อถือได้ — มันเป็นแค่ 'ความหวัง' ที่ยังไม่ถูกพิสูจน์"**

### 838.1 ทำไมการทดสอบ Restore ถึงสำคัญขนาดนี้

ย้อนกลับไปดูเคสของ GitLab.com ปี 2017 ที่กล่าวถึงใน Step 831 — ทีมงานมีระบบ backup ถึง **5 ชั้น** (LVM snapshot, regular backups, Azure disk snapshots, SH replication, S3 backup) แต่เมื่อถึงเวลาต้องใช้จริง กลับพบว่า:

- LVM snapshot ไม่ได้ถูกตั้งเวลาให้ทำงานอัตโนมัติ
- Regular backup ล้มเหลวมาตั้งแต่หลายวันก่อนโดยไม่มีใครสังเกต เพราะ Postgres เวอร์ชันไม่ตรงกันทำให้ backup script รันไม่ผ่าน แต่ไม่มีการแจ้งเตือนที่ดีพอ
- S3 backup เป็นไฟล์เปล่า

**เหตุผลที่ทั้งหมดนี้เกิดขึ้นได้เพราะไม่เคยมีใครลองกู้คืนจริงมาก่อนเลย** — ระบบ backup ทำงาน (เขียนไฟล์ลงไปได้) แต่ไม่มีใครตรวจสอบว่า **ไฟล์ backup ที่ได้ ใช้กู้คืนกลับมาเป็นระบบที่ทำงานได้จริงหรือไม่**

นี่คือความแตกต่างระหว่าง "Backup" กับ "Backup ที่เชื่อถือได้":

```
Backup ธรรมดา:
  สร้างไฟล์ backup → เก็บไว้ → (ไม่เคยทดสอบ) → หวังว่าจะกู้คืนได้เมื่อถึงเวลาจริง

Backup ที่เชื่อถือได้:
  สร้างไฟล์ backup → เก็บไว้ → ทดสอบกู้คืนสม่ำเสมอ → ยืนยันว่ากู้คืนได้จริง → เชื่อมั่นได้เมื่อถึงเวลาจริง
```

### 838.2 สิ่งที่ต้องตรวจสอบเมื่อทดสอบ Restore

การทดสอบ restore ที่ดีต้องตอบคำถามเหล่านี้ให้ได้ ไม่ใช่แค่ "clone ได้" อย่างเดียว:

1. **ความสมบูรณ์ของ Git object** — รันผ่าน `git fsck` แล้วไม่มี error/corruption
2. **จำนวน branch/tag ครบถ้วน** — เทียบจำนวนกับต้นฉบับ (ถ้ายังเข้าถึงต้นฉบับได้ตอนทดสอบ)
3. **Commit ล่าสุดตรงกับที่คาดหวัง** — เทียบ commit hash ของ branch หลักให้ตรงกับที่บันทึกไว้ล่าสุด
4. **โค้ดรันได้จริง** — checkout ออกมาแล้วรัน build/test suite ผ่าน ไม่ใช่แค่ไฟล์อยู่ครบ
5. **เวลาที่ใช้ในการ restore จริง** — วัดเวลาจริงเทียบกับ RTO ที่ตั้งไว้ใน DR Plan (Step 837) ว่าเป็นไปได้จริงหรือแค่ทฤษฎี
6. **Metadata ที่ export ไว้ (issues, PR)** — ทดสอบ import กลับเข้าไปในระบบทดสอบ (staging) แล้วดูว่าข้อมูลอ่านได้ครบ ไม่เพี้ยน

### 838.3 คำสั่งตรวจสอบความสมบูรณ์ของ Mirror Backup

```bash
# ตรวจสอบว่า object database ไม่เสียหาย (corruption check แบบละเอียด)
git --git-dir=myapp-backup.git fsck --full --strict

# เปรียบเทียบ ref ทั้งหมดระหว่าง backup กับต้นฉบับ (ถ้ายังเข้าถึงต้นฉบับได้)
git --git-dir=myapp-backup.git show-ref > backup-refs.txt
git ls-remote git@github.com:myorg/myapp.git > origin-refs.txt
diff <(awk '{print $1}' backup-refs.txt | sort) <(awk '{print $1}' origin-refs.txt | sort)

# ตรวจสอบว่า commit ล่าสุดของ main ตรงกันหรือไม่
git --git-dir=myapp-backup.git rev-parse refs/heads/main
```

ถ้าคำสั่ง `diff` ด้านบนไม่มี output อะไรเลย แปลว่า ref (commit hash ของทุก branch/tag) ตรงกันทั้งหมดระหว่าง backup กับต้นฉบับ ซึ่งเป็นสัญญาณที่ดีว่า mirror ทำงานถูกต้อง

### 838.4 กำหนดตารางการทดสอบ Restore เป็นกิจวัตร (Restore Drill)

องค์กรที่จริงจังเรื่อง DR จะกำหนด **"DR Drill"** หรือ "การซ้อมกู้คืนจากภัยพิบัติ" เป็นกิจกรรมประจำ เช่น:

| ความถี่ | กิจกรรม |
|---|---|
| ทุกสัปดาห์ | รัน automated script ตรวจสอบ `git fsck` และเทียบ ref ของทุก mirror อัตโนมัติ พร้อมแจ้งเตือนถ้าพบความผิดปกติ |
| ทุกเดือน | สุ่มเลือก 1 repository สำคัญ แล้วลอง restore แบบเต็มรูปแบบไปยัง environment ทดสอบ วัดเวลาที่ใช้จริง |
| ทุกไตรมาส | จำลองสถานการณ์ "GitHub ถูก suspend" แบบเต็มรูปแบบ ทั้งทีมซ้อมตามขั้นตอนใน DR Plan จริง รวมถึงการสื่อสารและการปรับ CI/CD |
| หลังมีการเปลี่ยนแปลงระบบใหญ่ | ทดสอบซ้ำทุกครั้งที่มีการเปลี่ยนผู้ให้บริการ backup, เปลี่ยนโครงสร้าง infrastructure หรือเปลี่ยนทีมที่ดูแล |

### 838.5 Automated Restore Verification Script

ตัวอย่าง script ที่ทำการทดสอบ restore อัตโนมัติแบบง่าย ๆ เพื่อรันเป็นส่วนหนึ่งของ pipeline ตรวจสอบ backup ทุกสัปดาห์:

```bash
#!/usr/bin/env bash
# verify-backup-restore.sh
# ทดสอบว่า mirror backup สามารถ restore และ build ได้จริง

set -euo pipefail

BACKUP_PATH="/var/backups/git-mirrors/myapp.git"
TEST_DIR=$(mktemp -d)

echo "=== เริ่มทดสอบ restore จาก: $BACKUP_PATH ==="

# ขั้นที่ 1: ตรวจสอบความสมบูรณ์ของ object database
echo "[1/4] ตรวจสอบความสมบูรณ์ของ backup ด้วย fsck..."
git --git-dir="$BACKUP_PATH" fsck --full --strict

# ขั้นที่ 2: จำลองการ restore จริงไปยังโฟลเดอร์ทดสอบ
echo "[2/4] จำลองการ clone กลับจาก backup..."
git clone "$BACKUP_PATH" "$TEST_DIR/restored"

# ขั้นที่ 3: ตรวจสอบว่า branch หลักมีอยู่จริงและ checkout ได้
echo "[3/4] ตรวจสอบ branch หลักและ checkout..."
cd "$TEST_DIR/restored"
git checkout main

COMMIT_COUNT=$(git rev-list --count HEAD)
echo "  พบ $COMMIT_COUNT commits บน branch main"

if [ "$COMMIT_COUNT" -lt 1 ]; then
  echo "  ล้มเหลว: ไม่พบ commit ใด ๆ ใน backup!"
  exit 1
fi

# ขั้นที่ 4: ลองรัน build/test เพื่อยืนยันว่าโค้ดใช้งานได้จริง (ปรับตามโปรเจกต์จริง)
echo "[4/4] ลองรัน build เพื่อยืนยันว่าโค้ดสมบูรณ์..."
if [ -f "package.json" ]; then
  npm install --silent && npm run build --silent
  echo "  build สำเร็จ - backup ใช้งานได้จริง"
else
  echo "  ไม่พบ package.json ข้ามขั้นตอน build (ปรับ script ตามโปรเจกต์จริง)"
fi

# ทำความสะอาด
rm -rf "$TEST_DIR"

echo "=== การทดสอบ restore เสร็จสมบูรณ์: backup ใช้งานได้จริง ==="
```

การรัน script แบบนี้เป็นประจำ (เช่นทุกสัปดาห์ผ่าน cron หรือ CI schedule) ทำให้ทีมมั่นใจได้ตลอดเวลาว่า **ถ้าเกิดเหตุจริงขึ้นมาพรุ่งนี้ เราจะกู้คืนได้จริง ไม่ใช่แค่คิดว่าน่าจะได้**

---

## Step 839: Self-Hosted Git Server เป็นทางเลือก backup ขั้นสุด

สำหรับองค์กรที่ต้องการความควบคุมสูงสุด หรือมีความกังวลเรื่องการพึ่งพา SaaS รายเดียวมากเกินไป (vendor lock-in) ทางเลือกขั้นสุดคือการมี **Self-Hosted Git Server** เป็นระบบสำรอง หรือแม้แต่เป็นระบบหลักเลยก็ได้

### 839.1 แนวคิดของ Self-Hosted Git Server

Self-hosted หมายถึงการติดตั้งซอฟต์แวร์ที่ทำหน้าที่คล้าย GitHub/GitLab ไว้บนเซิร์ฟเวอร์ที่องค์กรควบคุมเอง (on-premise หรือ private cloud ของตัวเอง) แทนที่จะพึ่งพา SaaS ของบริษัทภายนอกทั้งหมด

**ข้อดี:**

- ควบคุมข้อมูลได้ 100% ไม่ต้องกังวลเรื่อง suspend/ban จากผู้ให้บริการภายนอก
- ปฏิบัติตามข้อกำหนด **Data Sovereignty** ได้ง่ายกว่า (ข้อมูลต้องอยู่ในประเทศ/เขตอำนาจศาลที่กำหนด) ซึ่งสำคัญมากสำหรับหน่วยงานรัฐ ธนาคาร หรือองค์กรที่อยู่ภายใต้กฎหมายคุ้มครองข้อมูลเข้มงวด
- ไม่มีค่าใช้จ่ายรายที่นั่ง (per-seat) เหมือน SaaS บาง plan เมื่อทีมโตขึ้นมาก
- ปรับแต่ง (customize) ได้ลึกกว่า เพราะควบคุม infrastructure เอง

**ข้อเสีย:**

- ทีมต้องรับผิดชอบเรื่อง uptime, security patching, scaling เอง (กลายเป็นภาระของทีม ไม่ใช่ของผู้ให้บริการอีกต่อไป)
- ต้องมีความรู้ด้าน system administration ในทีม
- ถ้าไม่มีการวางแผน backup ให้ตัวระบบ self-hosted เองอีกที ก็จะกลายเป็น SPOF ใหม่ที่แย่กว่าเดิม (เพราะไม่มี infrastructure ระดับโลกช่วยดูแลให้)

### 839.2 GitLab Self-Managed (GitLab Community Edition / Enterprise Edition)

**GitLab Self-Managed** คือการติดตั้ง GitLab เวอร์ชันเต็มบนเซิร์ฟเวอร์ของตัวเอง มีทั้ง **Community Edition (CE)** ที่ใช้ฟรี และ **Enterprise Edition (EE)** ที่มีฟีเจอร์ระดับองค์กรเพิ่มเติม (SSO ขั้นสูง, compliance dashboard, ฯลฯ)

ตัวอย่างการติดตั้งแบบง่ายที่สุดด้วย Docker (สำหรับทดสอบ ไม่ใช่ production):

```bash
docker run --detach \
  --hostname gitlab.example.local \
  --publish 443:443 --publish 80:80 --publish 2222:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

จุดเด่นของ GitLab Self-Managed ในบริบท Disaster Recovery คือมันมาพร้อม **built-in backup command** ที่ครอบคลุมทั้ง repository, database (issues/MR/comments), CI/CD configuration, และ Container Registry ในคำสั่งเดียว:

```bash
# สร้าง full backup ของ GitLab instance ทั้งหมด (รันภายใน container/เซิร์ฟเวอร์ GitLab)
gitlab-backup create

# กู้คืนจาก backup (ต้องหยุด service บางส่วนก่อน)
gitlab-ctl stop puma
gitlab-ctl stop sidekiq
gitlab-backup restore BACKUP=<timestamp_ของไฟล์backup>
gitlab-ctl start
```

นี่คือข้อได้เปรียบสำคัญของการ self-host: **backup ครอบคลุมทั้ง metadata (issues, MR) และ code ในกลไกเดียวกัน** ไม่ต้องแยกทำ API export เหมือนตอนใช้ SaaS

### 839.3 Gitea และ Forgejo — ทางเลือกที่เบากว่า

สำหรับทีมที่ไม่ต้องการความซับซ้อนระดับ GitLab (ซึ่งกิน resource ค่อนข้างเยอะ) มีทางเลือกที่เบากว่ามาก:

- **Gitea** — เขียนด้วยภาษา Go ใช้ resource น้อยมาก ติดตั้งง่าย รองรับ SQLite ได้เลยสำหรับทีมเล็ก เหมาะกับการรันบนเซิร์ฟเวอร์ขนาดเล็กหรือแม้แต่ Raspberry Pi
- **Forgejo** — โปรเจกต์ที่แตกออกมาจาก Gitea (fork) เพื่อเน้นความเป็น community-driven และ open governance มากขึ้น มี feature set ใกล้เคียงกับ Gitea มาก

ตัวอย่างการติดตั้ง Gitea ด้วย Docker Compose:

```yaml
version: "3"

services:
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
    restart: always
    volumes:
      - ./gitea-data:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"
      - "222:22"
```

Gitea และ Forgejo เหมาะมากสำหรับการเป็น **"backup instance" ที่เบาและรันง่าย** — ทีมสามารถตั้งค่าให้ mirror จาก GitHub/GitLab เข้ามาอัตโนมัติผ่านฟีเจอร์ **"Repository Mirroring"** ที่มีในตัว ซึ่งใช้หลักการเดียวกับ `git clone --mirror` ที่เรียนใน Step 834 แต่มี UI ให้ตั้งค่าและติดตามสถานะได้ง่ายกว่า ไม่ต้องเขียน script/cron เอง

### 839.4 กลยุทธ์แบบผสม (Hybrid Strategy)

ในทางปฏิบัติ องค์กรจำนวนมาก**ไม่ได้เลือกอย่างใดอย่างหนึ่งสุดโต่ง** แต่ใช้กลยุทธ์แบบผสม:

```
                    ┌─────────────────────┐
                    │   GitHub (Primary)  │  ← ใช้งานประจำวัน, PR review, CI/CD
                    └──────────┬──────────┘
                               │ mirror แบบ real-time (multi-remote)
                               ▼
                    ┌─────────────────────┐
                    │  GitLab.com (Warm    │  ← สามารถสลับมาใช้งานต่อได้เกือบทันที
                    │  Standby)            │     ถ้า GitHub มีปัญหา
                    └──────────┬──────────┘
                               │ scheduled sync (รายวัน)
                               ▼
                    ┌─────────────────────┐
                    │ Self-hosted Gitea    │  ← Cold backup ภายในองค์กร
                    │ (Internal, Cold      │     ควบคุมเต็มรูปแบบ ไม่ขึ้นกับ
                    │  Backup)             │     ผู้ให้บริการภายนอกเลย
                    └─────────────────────┘
```

แนวคิดนี้เรียกว่า **Warm Standby + Cold Backup**:

- **Warm Standby** (เช่น GitLab.com) คือสำเนาที่ up-to-date พร้อมสลับมาใช้งานได้เกือบทันที (RTO ต่ำ) แต่ยังเป็น SaaS ของผู้ให้บริการภายนอกเหมือนกัน
- **Cold Backup** (เช่น self-hosted Gitea ภายในองค์กร) คือสำเนาสุดท้ายที่ไม่ขึ้นกับผู้ให้บริการภายนอกใด ๆ เลย ใช้เวลานานกว่าในการสลับมาใช้งาน (RTO สูงกว่า) แต่ให้ความมั่นใจสูงสุดว่าข้อมูลจะไม่มีวันหายไปพร้อมกับผู้ให้บริการรายใดรายหนึ่ง

---

## Step 840: แบบฝึกหัด — ตั้งค่า mirror backup อัตโนมัติ พร้อมทดสอบ restore จริง

ถึงเวลาลงมือทำจริงตามหลักการทั้งหมดที่เรียนมาใน Part นี้ แบบฝึกหัดนี้จะให้คุณสร้างระบบ backup ที่สมบูรณ์ตั้งแต่ต้นจนจบ พร้อมทดสอบ restore จริงด้วยตัวเอง

### 840.1 เตรียมสภาพแวดล้อมสำหรับฝึกฝน

สร้างโฟลเดอร์สำหรับแบบฝึกหัดนี้ในโฟลเดอร์ `git-course` ที่เตรียมไว้ตั้งแต่ Part 01:

```bash
mkdir -p ~/git-course/part-84-disaster-recovery
cd ~/git-course/part-84-disaster-recovery
```

จำลอง repository "ต้นฉบับ" ขึ้นมาก่อน (แทนที่ GitHub จริง เพื่อให้ฝึกได้โดยไม่ต้องมี remote จริง):

```bash
mkdir origin-repo
cd origin-repo
git init --bare
cd ..

git clone origin-repo working-copy
cd working-copy

echo "# My Important Project" > README.md
git add README.md
git commit -m "commit แรก: เพิ่ม README"

echo "console.log('hello world');" > app.js
git add app.js
git commit -m "เพิ่มไฟล์ app.js"

git branch feature/login
git tag v1.0.0

git push origin main
git push origin feature/login
git push origin v1.0.0

cd ..
```

ตอนนี้คุณมี `origin-repo` ที่ทำหน้าที่เป็น "ต้นฉบับ" จำลอง GitHub/GitLab แล้ว

### 840.2 ขั้นที่ 1 — สร้าง Mirror Backup ครั้งแรก

```bash
git clone --mirror origin-repo backup-repo.git
```

ตรวจสอบว่า mirror สมบูรณ์:

```bash
git --git-dir=backup-repo.git show-ref
```

ควรเห็น ref ครบทั้ง `refs/heads/main`, `refs/heads/feature/login`, และ `refs/tags/v1.0.0`

### 840.3 ขั้นที่ 2 — เขียน Script Sync อัตโนมัติ

สร้างไฟล์ `sync-backup.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

ORIGIN_PATH="$(pwd)/origin-repo"
BACKUP_PATH="$(pwd)/backup-repo.git"

echo "[$(date '+%Y-%m-%d %H:%M:%S')] เริ่ม sync backup..."

git --git-dir="$BACKUP_PATH" fetch --prune "$ORIGIN_PATH" '+refs/*:refs/*'

echo "[$(date '+%Y-%m-%d %H:%M:%S')] ตรวจสอบความสมบูรณ์ของ backup..."
git --git-dir="$BACKUP_PATH" fsck --full

echo "[$(date '+%Y-%m-%d %H:%M:%S')] sync backup เสร็จสมบูรณ์"
```

ให้สิทธิ์รันได้:

```bash
chmod +x sync-backup.sh
```

ทดสอบ: เพิ่ม commit ใหม่ใน `working-copy` แล้ว push แล้วรัน sync:

```bash
cd working-copy
echo "// เพิ่ม feature ใหม่" >> app.js
git add app.js
git commit -m "อัปเดต app.js"
git push origin main
cd ..

./sync-backup.sh
```

ตรวจสอบว่า commit ใหม่ปรากฏใน backup แล้ว:

```bash
git --git-dir=backup-repo.git log --oneline refs/heads/main
```

### 840.4 ขั้นที่ 3 — จำลอง "ภัยพิบัติ" (ลบ Origin ทิ้งทั้งหมด)

นี่คือส่วนสำคัญที่สุดของแบบฝึกหัด — เราจะจำลองสถานการณ์ที่ repository ต้นฉบับหายไปโดยสิ้นเชิง เหมือนที่อาจเกิดขึ้นจริงตามที่เรียนใน Step 831-832:

```bash
# จำลองว่าองค์กรถูกลบ repository (สถานการณ์เลวร้ายที่สุด)
rm -rf origin-repo

echo "origin-repo ถูกลบไปแล้ว! (จำลองสถานการณ์ภัยพิบัติ)"
ls -la
```

ตอนนี้ `origin-repo` หายไปแล้วจริง ๆ เหลือแค่ `backup-repo.git` และ `working-copy` (ซึ่งเราจะแกล้งสมมติว่า working-copy ก็ไม่มีใครมีอยู่ในเครื่องเช่นกัน เพื่อจำลองสถานการณ์เลวร้ายที่สุด)

### 840.5 ขั้นที่ 4 — ทดสอบ Restore จริง

ทำตามขั้นตอนใน DR Plan (เหมือนตัวอย่างใน Step 837.5):

```bash
# ขั้นที่ 1: ตรวจสอบความสมบูรณ์ของ backup ก่อนใช้กู้คืน
git --git-dir=backup-repo.git fsck --full --strict

# ขั้นที่ 2: สร้าง "origin ใหม่" จาก backup (จำลองการสร้าง repository ใหม่บน GitHub)
mkdir origin-repo-restored
cd origin-repo-restored
git init --bare
cd ..

# ขั้นที่ 3: Push ข้อมูลทั้งหมดจาก backup กลับเข้า origin ใหม่
cd backup-repo.git
git push --mirror "$(pwd)/../origin-repo-restored"
cd ..

# ขั้นที่ 4: Clone จาก origin ใหม่มาตรวจสอบว่าทุกอย่างครบถ้วน
git clone origin-repo-restored verify-restore
cd verify-restore

echo "=== ตรวจสอบ branch ทั้งหมด ==="
git branch -a

echo "=== ตรวจสอบ tag ทั้งหมด ==="
git tag

echo "=== ตรวจสอบ commit history ==="
git log --oneline --all --graph

echo "=== ตรวจสอบเนื้อหาไฟล์ล่าสุด ==="
cat app.js
cat README.md
```

### 840.6 ขั้นที่ 5 — ยืนยันผลลัพธ์ด้วย Checklist

ตรวจสอบให้ครบทุกข้อก่อนสรุปว่า restore สำเร็จ:

- [ ] `git fsck --full --strict` ไม่พบ error หรือ corruption ใด ๆ ใน backup
- [ ] จำนวน commit บน `main` ตรงกับที่คาดหวัง (รวม commit ที่เพิ่มหลัง sync ครั้งแรกด้วย)
- [ ] Branch `feature/login` ยังอยู่ครบ
- [ ] Tag `v1.0.0` ยังอยู่ครบ
- [ ] เนื้อหาไฟล์ `app.js` มีทั้งบรรทัดแรกและบรรทัดที่เพิ่มทีหลัง (`// เพิ่ม feature ใหม่`)
- [ ] สามารถ clone จาก origin ที่กู้คืนแล้วได้โดยไม่มี error

ถ้าผ่านครบทุกข้อ แปลว่าคุณเพิ่งพิสูจน์ด้วยตัวเองแล้วว่า **ระบบ backup ของคุณใช้งานได้จริง** ไม่ใช่แค่ "น่าจะได้"

### 840.7 ส่วนขยาย (ถ้ามีเวลา) — ทดสอบกับ Repository จริงบน GitHub

สำหรับผู้ที่ต้องการฝึกกับสถานการณ์ที่ใกล้เคียงกับการทำงานจริงมากขึ้น ให้ลองทำตามขั้นตอนต่อไปนี้กับ repository ทดสอบบน GitHub ของตัวเอง (สร้าง repository เปล่าใหม่ไว้สำหรับทดลองเท่านั้น อย่าใช้ repository ที่มีข้อมูลสำคัญจริง):

1. สร้าง repository ทดสอบชื่อ `dr-practice` บน GitHub ของตัวเอง
2. เพิ่มไฟล์และ commit สัก 3-4 ครั้ง พร้อมสร้าง branch และ tag เพิ่มเติม
3. รัน `git clone --mirror` ไปเก็บไว้ในเครื่อง local
4. เขียน cron job ให้ sync backup นี้ทุก 10 นาที (เพื่อเห็นผลเร็วในการฝึก)
5. ลองสร้าง repository เปล่าใหม่อีกอันชื่อ `dr-practice-restored`
6. Push mirror backup กลับเข้าไปใน repository ใหม่นั้นด้วย `git push --mirror`
7. เปรียบเทียบว่า `dr-practice-restored` มีข้อมูลครบเหมือน `dr-practice` ต้นฉบับหรือไม่

การฝึกกับ GitHub จริงจะทำให้คุณคุ้นเคยกับ credential, SSH key, และ permission ที่ต้องจัดการในสถานการณ์จริงมากขึ้น ซึ่งเป็นรายละเอียดที่มักไม่ปรากฏตอนฝึกกับ local repository ล้วน ๆ

---

## สรุป Part 84

ใน Part นี้เราได้เรียนรู้ว่า:

1. **GitHub/GitLab รับผิดชอบ infrastructure แต่ไม่รับผิดชอบความผิดพลาดของมนุษย์** — นี่คือหลักการ Shared Responsibility Model ที่ทุกทีมต้องเข้าใจก่อนเริ่มวางกลยุทธ์ backup
2. **Single Point of Failure เกิดขึ้นได้จริง** ทั้งจากบัญชีถูกลบโดยไม่ตั้งใจ องค์กรถูก suspend ผู้ให้บริการมีปัญหาระบบ หรือบัญชีถูกโจมตี
3. **`git clone --mirror`** คือเครื่องมือพื้นฐานที่สำคัญที่สุดในการสร้างสำเนา repository แบบสมบูรณ์ 100% ครอบคลุมทุก branch, tag และ ref พิเศษ
4. **Multi-remote strategy** ทำให้ backup เป็น real-time มากขึ้น โดยตั้งค่า push URL หลายตัวในคำสั่ง push เดียว หรือใช้ CI/CD trigger ทุกครั้งที่มี push
5. **Metadata อย่าง Issues, Pull Requests, Wiki ไม่ได้อยู่ใน Git object** ต้องใช้ API เฉพาะของแพลตฟอร์ม (REST API, GraphQL, Migrations API) ในการ export แยกต่างหาก
6. **RTO และ RPO** คือตัวชี้วัดหลักของ Disaster Recovery Plan ที่ต้องกำหนดให้ชัดเจนตามความสำคัญของแต่ละ repository และออกแบบความถี่ของ backup ให้สอดคล้องกัน
7. **Backup ที่ไม่เคยทดสอบ restore ไม่ใช่ backup ที่เชื่อถือได้** — ต้องมีการซ้อม (DR Drill) เป็นประจำ พร้อมทั้งตรวจสอบความสมบูรณ์ด้วย `git fsck` และทดลอง build/test จริงหลัง restore
8. **Self-hosted Git server** (GitLab Self-Managed, Gitea, Forgejo) เป็นทางเลือกขั้นสุดสำหรับองค์กรที่ต้องการควบคุมข้อมูลเต็มรูปแบบ ไม่พึ่งพา SaaS รายเดียว
9. เราได้ลงมือฝึกทำจริงตั้งแต่สร้าง mirror backup, เขียน script sync อัตโนมัติ, จำลองสถานการณ์ภัยพิบัติ ไปจนถึงทดสอบ restore แบบครบวงจร

### Checklist ก่อนไป Part ถัดไป

- [ ] เข้าใจความแตกต่างระหว่าง "Availability" กับ "Backup" และรู้ว่า SaaS รับผิดชอบส่วนไหน
- [ ] อธิบายได้ว่า Single Point of Failure ในบริบท Git มีอะไรบ้าง
- [ ] รัน `git clone --mirror` และเข้าใจว่าทำไมมันต่างจาก `git clone` ปกติ
- [ ] เขียน script sync mirror backup อัตโนมัติได้ และรู้วิธีตั้งค่าให้รันด้วย cron
- [ ] ตั้งค่า multi-remote ด้วย `git remote set-url --add --push` ได้
- [ ] เข้าใจว่า Issues/PR/Wiki ต้อง backup แยกจาก Git object ผ่าน API
- [ ] อธิบายความหมายของ RTO และ RPO ได้ และกำหนดค่าที่เหมาะสมกับ repository ของตัวเองได้
- [ ] ทดสอบ restore จริงจาก mirror backup อย่างน้อยหนึ่งครั้งด้วยตัวเอง พร้อมตรวจสอบด้วย `git fsck`
- [ ] เข้าใจข้อดี-ข้อเสียของ Self-hosted Git server เทียบกับ SaaS
- [ ] ทำแบบฝึกหัด mirror backup + จำลองภัยพิบัติ + restore จริงใน Step 840 จนครบทุกขั้นตอน

**ต่อไป:** [Part 85: โปรเจกต์ฝึกหัด: วาง Security Policy ให้องค์กร](./part-085-security-policy-โปรเจกต์.md)
