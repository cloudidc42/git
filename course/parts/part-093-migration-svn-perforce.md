# Part 93: Migration: ย้ายจาก SVN/Perforce มาสู่ Git

> **Step ในหลักสูตรนี้:** Step 921–930
> **เฟส:** 9 — ทักษะมืออาชีพ: Maintainer, Release Management, Metrics
> **เป้าหมายของ Part นี้:** เข้าใจว่าทำไมองค์กรจำนวนมากยังคงใช้ SVN หรือ Perforce อยู่ในปี 2026 และทำไมการย้ายมาสู่ Git ยังคุ้มค่า เรียนรู้เครื่องมือจริงอย่าง `git svn` และ `git-p4` สำหรับย้ายประวัติทั้งหมดมาสู่ Git อย่างถูกต้อง เข้าใจการแปลงโครงสร้าง branch/tag ของ SVN ให้เข้ากับ Git การจัดการไฟล์ binary ขนาดใหญ่ระหว่าง migrate กลยุทธ์ parallel run เพื่อลดความเสี่ยง การ training ทีมงานที่คุ้นเคยกับระบบเดิมมานาน และวิธี validate ว่าการย้ายประวัติสำเร็จครบถ้วนจริง ก่อนจะปิดท้ายด้วยแบบฝึกหัดวางแผน migration แบบเต็มรูปแบบให้องค์กรสมมติ

---

## สารบัญของ Part นี้

- Step 921: ทำไมองค์กรบางแห่งยังใช้ SVN/Perforce อยู่ และทำไมยังต้อง migrate มาสู่ Git ในปัจจุบัน
- Step 922: ความท้าทายของการ migrate (ประวัติเก่าจำนวนมาก, ไฟล์ binary ใหญ่, ทีมคุ้นเคยกับ workflow เดิม)
- Step 923: `git svn` — เครื่องมือ migrate จาก SVN มาสู่ Git พร้อมประวัติ
- Step 924: การ migrate จาก Perforce (`git-p4` และเครื่องมือเฉพาะทางอย่าง Josh)
- Step 925: การแปลง branch/tag structure จาก SVN ให้เข้ากับรูปแบบ branch/tag จริงของ Git
- Step 926: การจัดการไฟล์ binary ขนาดใหญ่ระหว่าง migrate (เชื่อมโยงกับ Git LFS)
- Step 927: Parallel Run Strategy — รันสองระบบคู่ขนานเพื่อลดความเสี่ยง
- Step 928: การ training ทีมให้คุ้นเคยกับ Git หลัง migrate
- Step 929: Post-migration validation — ตรวจสอบความถูกต้องครบถ้วนของประวัติ
- Step 930: แบบฝึกหัด — วางแผน migration timeline ให้องค์กรสมมติที่ใช้ SVN มา 10 ปี ทีม 50 คน

---

## Step 921: ทำไมองค์กรบางแห่งยังใช้ SVN/Perforce อยู่ และทำไมยังต้อง migrate มาสู่ Git ในปัจจุบัน

หลายคนที่เติบโตมากับ Git อาจแปลกใจว่า ทำไมในปี 2026 ยังมีองค์กรจำนวนไม่น้อยที่ใช้ **Subversion (SVN)** หรือ **Perforce (Helix Core)** เป็นระบบควบคุมเวอร์ชันหลักอยู่เลย คำตอบไม่ใช่เพราะพวกเขา "ล้าหลัง" แต่เป็นเพราะเหตุผลเชิงเทคนิคและองค์กรที่หนักแน่นพอสมควร

### 1.1 ทำไมองค์กรถึงยังใช้ SVN/Perforce อยู่

**เหตุผลที่ยังใช้ SVN:**

1. **โปรเจกต์เก่าแก่มาก** — หลายองค์กรมี repository ที่สร้างมาตั้งแต่ปี 2000–2010 ก่อนที่ Git จะเป็นมาตรฐาน การย้ายประวัติ 10-20 ปีมีความเสี่ยงสูงและใช้ทรัพยากรมาก
2. **Centralized model ที่ทีมคุ้นเคย** — SVN มี mental model ที่ตรงไปตรงมา: มี "เวอร์ชันล่าสุด" เดียวบน server ไม่มีแนวคิดเรื่อง local commit หรือ distributed history ให้ทีมที่ไม่ใช่สาย technical จัดการยาก ทีมที่ไม่ใช่นักพัฒนาเต็มตัว (เช่น QA, Technical Writer, Designer) มักชอบความเรียบง่ายแบบนี้
3. **Partial checkout ที่ทำได้ง่ายกว่า** — SVN รองรับการ checkout เฉพาะบางโฟลเดอร์ย่อยของ repository ขนาดใหญ่ได้ตั้งแต่ต้น (sparse checkout) โดยไม่ต้องตั้งค่าซับซ้อนเหมือน Git
4. **Permission ระดับโฟลเดอร์ (path-based ACL)** — SVN และ Perforce รองรับการกำหนดสิทธิ์เข้าถึงเป็นรายโฟลเดอร์ได้ละเอียดกว่า Git แบบ native (Git ต้องพึ่ง submodule หรือ monorepo tooling เพิ่มเติม)
5. **Tooling เฉพาะทางที่ผูกกับ SVN/Perforce มานาน** — เช่นระบบ build/release ภายในองค์กรที่เขียน script ผูกกับคำสั่ง `svn` หรือ `p4` มาหลายปี

**เหตุผลที่ยังใช้ Perforce (Helix Core) โดยเฉพาะ:**

1. **อุตสาหกรรมเกมและ VFX** — Perforce ครองตลาดนี้เกือบเบ็ดเสร็จ เพราะรองรับไฟล์ binary ขนาดใหญ่ (asset, texture, model 3D, video) ได้ดีกว่า Git มาก ด้วยกลไก **exclusive file locking** ที่ป้องกันไม่ให้สอง artist แก้ไฟล์ binary เดียวกันพร้อมกันจนเกิด conflict ที่ merge ไม่ได้
2. **Repository ขนาดหลายเทราไบต์** — Perforce ถูกออกแบบมาให้จัดการ repository ขนาดใหญ่มากได้ตั้งแต่ต้น (Git ต้องอาศัย Git LFS หรือ partial clone เพิ่มเติมถึงจะสู้ได้)
3. **Semiconductor และ Hardware design** — ทีมที่ทำงานกับไฟล์ CAD, RTL, FPGA มักใช้ Perforce เพราะต้องการ locking แบบเข้มงวดและ audit trail ที่ regulator ต้องการ

### 1.2 แล้วทำไมยังต้อง migrate มาสู่ Git ในปัจจุบัน

แม้จะมีเหตุผลให้อยู่กับระบบเดิม แต่แรงผลักดันให้ migrate มาสู่ Git ก็แข็งแกร่งขึ้นเรื่อย ๆ ในช่วงหลายปีที่ผ่านมา:

1. **ตลาดแรงงาน** — วิศวกรรุ่นใหม่แทบทั้งหมดเรียนรู้ Git เป็นเครื่องมือแรก การรับคนใหม่เข้าทีมที่ใช้ SVN/Perforce ต้องเสียเวลา onboarding สอน tool ที่ตลาดไม่ได้ใช้กันแล้ว ทำให้ต้นทุนการจ้างงานสูงขึ้นและลด pool ผู้สมัครลง
2. **Ecosystem ของ CI/CD และ DevOps สมัยใหม่ผูกกับ Git เป็นหลัก** — GitHub Actions, GitLab CI, Jenkins pipeline สมัยใหม่, ArgoCD, Terraform Cloud ฯลฯ ล้วนออกแบบมาให้ทำงานกับ Git-based repository เป็นอันดับแรก การใช้ SVN/Perforce ต่อทำให้เข้าถึงเครื่องมือเหล่านี้ได้ยากขึ้นหรือต้องเขียน integration เอง
3. **Branching model ที่ยืดหยุ่นกว่ามาก** — องค์กรที่ต้องการทำ trunk-based development, feature flag, หรือ GitOps จริงจัง จะพบว่า branching แบบ Git (ที่ถูกและเร็ว) เอื้อต่อ workflow เหล่านี้มากกว่า SVN ที่ branch คือการ copy ทั้งโฟลเดอร์
4. **Offline-first workflow** — ทีมที่กระจายตัวทำงานจากหลายที่ (remote-first) ต้องการความสามารถ commit/diff/log แบบ offline ซึ่ง distributed model ของ Git ตอบโจทย์นี้ได้ดีกว่า centralized model มาก
5. **Security และ Compliance สมัยใหม่** — เครื่องมือ scan security ของโค้ด (SAST/DAST), signed commit, SBOM generation จำนวนมากรองรับ Git โดยตรงและอาจไม่รองรับ SVN/Perforce เลย
6. **ต้นทุน license** — Perforce เป็นซอฟต์แวร์เชิงพาณิชย์ที่มีค่า license ต่อ seat ในขณะที่ Git เป็น open source ฟรี 100% การย้ายช่วยลดต้นทุนระยะยาวได้อย่างมีนัยสำคัญสำหรับองค์กรขนาดใหญ่

### 1.3 คำถามสำคัญ: ควร migrate หรือไม่

ก่อนตัดสินใจ migrate ทีมควรตอบคำถามต่อไปนี้อย่างตรงไปตรงมา:

| คำถาม | ถ้าตอบว่า "ใช่" |
|---|---|
| ทีมส่วนใหญ่ทำงานกับ text-based source code เป็นหลักหรือไม่ | สนับสนุนการย้ายไป Git |
| มีไฟล์ binary ขนาดใหญ่จำนวนมากที่ต้อง lock แบบ exclusive หรือไม่ | ต้องวางแผน Git LFS หรือพิจารณาเก็บ Perforce ไว้บางส่วน |
| ทีมมีแผนใช้ CI/CD สมัยใหม่ในอนาคตอันใกล้หรือไม่ | เหตุผลหนักแน่นให้ย้าย |
| ประวัติเก่ามีความสำคัญทาง compliance/audit หรือไม่ (เช่น อุตสาหกรรมการเงิน การแพทย์) | ต้องวางแผน validation ที่ Step 929 อย่างละเอียด |
| องค์กรมีงบและเวลาสำหรับ training ทีมหรือไม่ | ถ้าไม่มี ต้องวางแผน phased migration |

ใน Step ถัดไปเราจะเจาะลึกความท้าทายที่แท้จริงของการ migrate ก่อนจะลงมือใช้เครื่องมือจริง

---

## Step 922: ความท้าทายของการ migrate (ประวัติเก่าจำนวนมาก, ขนาดไฟล์ binary ใหญ่, ทีมคุ้นเคยกับ workflow เดิม)

การ migrate จาก SVN/Perforce มาสู่ Git ไม่ใช่แค่การรันคำสั่งเดียวแล้วจบ มันมีความท้าทายหลายมิติที่ต้องวางแผนล่วงหน้า

### 2.1 ความท้าทายด้านประวัติ (History) จำนวนมหาศาล

องค์กรที่ใช้ SVN มา 10-15 ปีอาจมี:

- **Revision นับแสนถึงล้าน** — SVN repository ขนาดใหญ่มักมี revision number สูงถึงหลักแสนหรือหลักล้าน ในขณะที่ Git commit hash ไม่ได้เรียงลำดับเชิงตัวเลขแบบนั้น การแปลงต้อง map revision number เดิมเข้ากับ commit hash ใหม่ให้ครบถ้วน
- **Commit message และ author ที่ไม่เป็นมาตรฐาน** — SVN ใช้ username แบบง่าย (เช่น `jsmith`) ในขณะที่ Git ต้องการ `Name <email>` การแปลงต้องมี **author mapping file** ที่ครบถ้วนทุกคนที่เคย commit ตลอดประวัติ ไม่เช่นนั้นชื่อผู้เขียนจะกลายเป็น `jsmith <jsmith@repo-uuid>` ที่ดูไม่เป็นมืออาชีพ
- **ขนาด repository ที่โตมาก** — ประวัติที่สะสมมานานทำให้ dump/clone ใช้เวลานานหลายชั่วโมงถึงหลายวันสำหรับ repository ขนาดใหญ่มาก ๆ
- **Merge history ที่ไม่ชัดเจน** — SVN merge (โดยเฉพาะก่อนเวอร์ชัน 1.5 ที่ยังไม่มี merge tracking) มักไม่มีข้อมูลว่า commit ไหน merge มาจากไหน ทำให้ Git ที่แปลงมาอาจแสดง branch history เป็นเส้นตรงที่ไม่สะท้อนความจริง (linear history แทนที่จะเป็น graph ที่ถูกต้อง)

### 2.2 ความท้าทายด้านไฟล์ binary ขนาดใหญ่

นี่คือความท้าทายที่หนักที่สุดสำหรับทีมที่ทำงานกับเกม, VFX หรือ hardware design:

- **Git เก็บทุก object เป็น snapshot** — ถ้า binary ไฟล์ขนาด 500MB ถูกแก้ไข 200 ครั้งตลอดประวัติ การแปลงตรง ๆ จะทำให้ `.git` directory บวมขึ้นเป็นหลายร้อย GB ทันที เพราะ Git ไม่รู้จักการทำ delta compression ที่ดีสำหรับไฟล์ binary แบบ Perforce
- **Clone ที่ช้าลงมหาศาล** — ทุกคนที่ clone repository ใหม่ต้องดาวน์โหลดประวัติ binary ทั้งหมด แม้จะไม่เคยต้องใช้เวอร์ชันเก่าเลย
- **ไม่มี exclusive locking แบบ native** — Perforce ป้องกัน conflict ของไฟล์ binary ด้วยการ lock ไฟล์ก่อนแก้ไข (`p4 edit` ที่ทำ exclusive checkout) แต่ Git ไม่มีกลไกนี้โดยตรง ต้องอาศัย Git LFS ร่วมกับ **File Locking feature** ของ LFS เอง (`git lfs lock`)

### 2.3 ความท้าทายด้าน workflow และ mindset ของทีม

- **Mental model ที่ต่างกันโดยสิ้นเชิง** — คนที่ใช้ SVN มา 10 ปีคุ้นเคยกับแนวคิด "มีเวอร์ชันเดียวที่ถูกต้องบน server เสมอ" (single source of truth บน central server) แต่ Git ให้ทุกคนมี local repository เต็มรูปแบบ การ "commit" ใน Git ไม่เท่ากับการ "ส่งงานให้ทีม" เหมือนใน SVN ความสับสนนี้ทำให้เกิดข้อผิดพลาดบ่อยในช่วงแรก (เช่น คิดว่า commit local แล้วคือเสร็จ ลืม push)
- **ความกลัวการสูญเสีย control** — ผู้บริหารทีมบางคนที่คุ้นกับ path-based permission ของ SVN/Perforce กังวลว่า Git จะทำให้ควบคุมสิทธิ์เข้าถึงยากขึ้น (ซึ่งแก้ได้ด้วย GitHub/GitLab permission หรือ CODEOWNERS แต่ต้องอธิบายให้ชัดเจน)
- **Branching model ที่ต้องเรียนรู้ใหม่ทั้งหมด** — ทีมที่ไม่เคยทำ feature branch แบบ Git-native (merge/rebase, pull request) ต้องปรับตัวมาก โดยเฉพาะการแก้ conflict ซึ่งใน SVN มักเจอไม่บ่อยเพราะทุกคนทำงานบน trunk เดียวเป็นหลัก

### 2.4 ความท้าทายด้าน tooling และ integration

- **CI/CD script ที่ผูกกับ `svn`/`p4` command** — ต้องเขียนใหม่ทั้งหมดให้เรียก `git` แทน
- **ระบบ issue tracking ที่ผูกกับ SVN revision number** — เช่น commit message ที่อ้างอิง `r12345` ต้อง map ไปยัง Git commit hash เพื่อให้ traceability ยังใช้งานได้
- **External tools ที่ integrate กับ Perforce โดยเฉพาะ** — เช่น Unreal Engine Editor ที่มี Perforce plugin ในตัว (Unreal ก็รองรับ Git แล้วแต่ต้องตั้งค่าเพิ่ม เช่น `.gitattributes` สำหรับ merge ไฟล์ `.uasset`)

ตารางสรุปความท้าทายและแนวทางแก้ไขคร่าว ๆ:

| ความท้าทาย | แนวทางแก้ไข (จะเจาะลึกใน Step ถัดไป) |
|---|---|
| ประวัติจำนวนมาก, author ไม่เป็นมาตรฐาน | ใช้ `git svn` พร้อม authors-file (Step 923) |
| Branch/tag เป็นแค่โฟลเดอร์ใน SVN | แปลงด้วย `git svn` พร้อม standard layout (Step 925) |
| ไฟล์ binary ใหญ่ | ใช้ Git LFS ร่วมกับการกรองประวัติ (Step 926) |
| ความเสี่ยงช่วง transition | ทำ Parallel Run (Step 927) |
| ทีมไม่คุ้นเคย Git | จัด Training Program (Step 928) |
| ต้องมั่นใจว่าประวัติถูกต้องครบ | Post-migration Validation (Step 929) |

ต่อไปเราจะเริ่มลงมือจริงกับเครื่องมือ `git svn`

---

## Step 923: `git svn` — เครื่องมือ migrate จาก SVN มาสู่ Git พร้อมประวัติ (`git svn clone`)

`git svn` เป็นเครื่องมือที่มาพร้อมกับ Git เองตั้งแต่ต้น (ต้องติดตั้ง package เสริมในบาง distro เช่น `git-svn` บน Debian/Ubuntu) ทำหน้าที่เป็นสะพานเชื่อมสองทาง: มันสามารถอ่านประวัติจาก SVN repository และแปลงเป็น Git commit ได้ รวมถึงยังใช้เป็น "bridge" ให้ทำงานกับ SVN repository ต่อไปในช่วง transition ก็ได้ (Step 927 จะใช้ความสามารถนี้)

### 3.1 ติดตั้ง git-svn

```bash
# Debian/Ubuntu
sudo apt-get install git-svn

# macOS (Homebrew)
brew install git

# ตรวจสอบว่าใช้งานได้
git svn --version
```

### 3.2 ทำความเข้าใจโครงสร้าง SVN ก่อน migrate

SVN repository แบบมาตรฐาน (standard layout) จะมีโครงสร้างดังนี้:

```
my-project/
├── trunk/       ← เทียบเท่า "main" branch ใน Git
├── branches/    ← โฟลเดอร์ที่เก็บ branch ต่าง ๆ (แต่ละ branch คือโฟลเดอร์ย่อย)
│   ├── feature-x/
│   └── release-2.0/
└── tags/        ← โฟลเดอร์ที่เก็บ tag ต่าง ๆ (แต่ละ tag คือโฟลเดอร์ย่อยเช่นกัน)
    ├── v1.0/
    └── v1.1/
```

ข้อสำคัญคือ **ใน SVN, branch และ tag ไม่ใช่แนวคิดพิเศษของระบบเหมือน Git — มันเป็นแค่ "การ copy โฟลเดอร์" ธรรมดา** (`svn copy`) ซึ่งจะกลายเป็นประเด็นสำคัญใน Step 925

### 3.3 ขั้นตอนที่ 1: สร้าง Authors File

นี่คือขั้นตอนที่สำคัญที่สุดและมักถูกมองข้าม ถ้าไม่ทำ ผู้เขียนทุก commit ที่แปลงมาจะกลายเป็นรูปแบบ `username (no author)` หรือ `username@repo-uuid` ที่ไม่เป็นมาตรฐาน

ขั้นแรก หา username ทั้งหมดที่เคย commit ในประวัติ:

```bash
svn log --xml -q http://svn.example.com/repos/my-project \
  | grep author \
  | sed -e 's/<author>//' -e 's/<\/author>//' -e 's/^\s*//' \
  | sort -u
```

จากนั้นสร้างไฟล์ `authors.txt` ที่ map แต่ละ username เข้ากับชื่อและอีเมลจริง:

```
jsmith = John Smith <john.smith@example.com>
mwong = Maria Wong <maria.wong@example.com>
apatel = Amit Patel <amit.patel@example.com>
(no author) = Unknown Author <unknown@example.com>
```

**ข้อควรระวัง:** ต้องครอบคลุมทุก username ให้ครบ 100% ถ้าเจอ username ที่ไม่มีใน mapping file ระหว่างการ clone คำสั่ง `git svn` จะหยุดทำงานทันที (fail fast) เพื่อป้องกันไม่ให้ข้อมูล author ผิดพลาดหลุดเข้าไปในประวัติ Git

### 3.4 ขั้นตอนที่ 2: Clone ด้วย `git svn clone`

กรณี repository ใช้ standard layout (`trunk`, `branches`, `tags`):

```bash
git svn clone http://svn.example.com/repos/my-project \
  --authors-file=authors.txt \
  --stdlayout \
  --prefix=svn/ \
  my-project-git
```

อธิบายแต่ละ flag:

- `--stdlayout` — บอกให้ `git svn` รู้ว่า repository ใช้โครงสร้างมาตรฐาน `trunk/branches/tags` และจะ map `trunk` เป็น `refs/remotes/svn/trunk` โดยอัตโนมัติ พร้อมดึง branch/tag ทั้งหมดมาด้วย
- `--authors-file` — ระบุไฟล์ mapping ที่สร้างไว้ใน 3.3
- `--prefix=svn/` — กำหนด prefix ของ remote-tracking reference (แนะนำให้ใส่เสมอ ไม่เช่นนั้น `git svn` รุ่นเก่าจะใช้ prefix ว่างซึ่งทำให้สับสนกับ Git tag จริง)

ถ้า repository ไม่ได้ใช้ standard layout ให้ระบุ path เองแบบละเอียด:

```bash
git svn clone http://svn.example.com/repos/my-project \
  --authors-file=authors.txt \
  --trunk=dev/main \
  --branches=dev/branches \
  --tags=dev/tags \
  --prefix=svn/ \
  my-project-git
```

### 3.5 การ clone แบบทยอย (Incremental Fetch) สำหรับ repository ขนาดใหญ่

สำหรับ repository ที่มี revision จำนวนมาก การ clone ทั้งหมดในครั้งเดียวอาจใช้เวลาหลายชั่วโมงถึงหลายวัน และเสี่ยงต่อการหลุดการเชื่อมต่อกลางคัน แนะนำให้ clone แบบจำกัด revision ก่อน แล้ว fetch เพิ่มทีละช่วง:

```bash
# Clone เฉพาะ revision 1 ถึง 10000 ก่อน (โครงสร้าง metadata เท่านั้น)
git svn clone http://svn.example.com/repos/my-project \
  --authors-file=authors.txt \
  --stdlayout \
  -r1:10000 \
  my-project-git

cd my-project-git

# fetch revision ถัดไปทีละช่วง
git svn fetch -r10001:20000
git svn fetch -r20001:30000
# ทำซ้ำจนครบทุก revision (หรือใช้ git svn fetch เฉย ๆ เพื่อดึงจนถึง HEAD)
git svn fetch
```

การทำแบบนี้ช่วยให้ตรวจสอบความถูกต้องเป็นช่วง ๆ ได้ และถ้าการเชื่อมต่อหลุด ก็ resume ต่อจากจุดที่ค้างได้โดยไม่ต้องเริ่มใหม่ทั้งหมด

### 3.6 ตรวจสอบผลลัพธ์เบื้องต้นหลัง clone

```bash
cd my-project-git

# ดู branch ที่ถูกแปลงมาจาก SVN (จะอยู่ใน remote-tracking, ยังไม่ใช่ local branch)
git branch -r

# ตัวอย่างผลลัพธ์:
#   svn/trunk
#   svn/feature-x
#   svn/release-2.0
#   svn/tags/v1.0
#   svn/tags/v1.1

# ดูจำนวน commit ที่แปลงมาทั้งหมด
git log svn/trunk --oneline | wc -l

# เทียบกับจำนวน revision ต้นทางใน SVN
svn log http://svn.example.com/repos/my-project/trunk -q | grep -c '^r'
```

ตัวเลขทั้งสองควรใกล้เคียงกัน (อาจไม่เท่ากันเป๊ะเพราะ revision บางอันอาจเป็นการเปลี่ยนแปลง path อื่นที่ไม่เกี่ยวกับ `trunk`)

### 3.7 คำสั่ง `git svn` ที่ควรรู้เพิ่มเติม

| คำสั่ง | หน้าที่ |
|---|---|
| `git svn fetch` | ดึง revision ใหม่จาก SVN มาต่อประวัติเดิม (ไม่ rewrite ของเก่า) |
| `git svn rebase` | เหมือน `git pull --rebase` — ใช้ตอนทำงานคู่ขนานกับ SVN (Parallel Run) |
| `git svn dcommit` | ส่ง local commit ของ Git กลับไปเป็น revision ใหม่บน SVN (ใช้ตอนยังต้องซิงค์สองทาง) |
| `git svn log` | ดู log ในรูปแบบคล้าย SVN แต่ดึงจากข้อมูล Git ที่แปลงแล้ว |
| `git svn show-ignore` | แปลง `svn:ignore` property เป็นเนื้อหาที่ใส่ใน `.gitignore` ได้ |

ในหัวข้อถัดไปเราจะพูดถึงการ migrate จาก Perforce ซึ่งใช้เครื่องมือและแนวคิดที่ต่างออกไปพอสมควร

---

## Step 924: การ migrate จาก Perforce (`git-p4` หรือเครื่องมือเฉพาะทาง เช่น Josh) ภาพรวม

Perforce (Helix Core) มีสถาปัตยกรรมที่ต่างจาก SVN พอสมควร โดยเฉพาะแนวคิดเรื่อง **Depot**, **Changelist**, และ **Workspace/Client Spec** ทำให้การ migrate ต้องใช้เครื่องมือที่ออกแบบมาเฉพาะ

### 4.1 ทำความเข้าใจโครงสร้าง Perforce ก่อน migrate

- **Depot** — พื้นที่เก็บไฟล์ระดับบนสุดของ Perforce (เทียบเท่า repository) เช่น `//depot/myproject/...`
- **Changelist** — ชุดของการเปลี่ยนแปลงที่ submit พร้อมกัน (เทียบเท่า commit ใน Git แต่ atomic ในระดับที่ต่างกันเล็กน้อย)
- **Client Spec (Workspace)** — การกำหนดว่าไฟล์ส่วนไหนของ depot จะถูก sync มาที่เครื่อง (คล้าย sparse checkout)
- **Stream** — Perforce เวอร์ชันใหม่ (Streams Depot) มีแนวคิดคล้าย branch ของ Git มากขึ้น มี parent-child relationship ระหว่าง stream ที่ชัดเจน

### 4.2 `git-p4` — เครื่องมือมาตรฐานที่มากับ Git

`git-p4` เป็น script Python ที่มาพร้อมกับ Git source (อยู่ใน `contrib/`) ใช้สำหรับ clone และซิงค์กับ Perforce depot

**ติดตั้ง:**

```bash
# ต้องมี Perforce command-line client (p4) ติดตั้งไว้ก่อน
# Ubuntu/Debian
sudo apt-get install git-p4

# ตรวจสอบ
git p4 --version
```

**ตั้งค่าการเชื่อมต่อ Perforce:**

```bash
export P4PORT=perforce.example.com:1666
export P4USER=jsmith
export P4CLIENT=jsmith-migration-client
p4 login
```

**Clone depot มาเป็น Git repository:**

```bash
git p4 clone //depot/myproject@all my-project-git
```

อธิบาย:

- `//depot/myproject` — path ของ depot ที่ต้องการ migrate
- `@all` — ดึงทุก changelist ตั้งแต่ต้นจนถึงปัจจุบัน (ถ้าไม่ใส่จะดึงแค่ changelist ล่าสุด)

**การจับคู่ author (คล้ายกับ SVN):**

`git-p4` จะพยายามดึงข้อมูล user จาก Perforce user database (`p4 users`) มาสร้าง mapping อัตโนมัติ แต่แนะนำให้ตรวจสอบและแก้ไขเองผ่านตัวแปรสภาพแวดล้อมหรือแก้หลัง commit ด้วย `git filter-repo` หากพบชื่อที่ผิดเพี้ยน:

```bash
p4 users > p4-users-list.txt
```

จากนั้นตรวจสอบว่าทุกคนมี email ที่ถูกต้องในระบบ Perforce ก่อน clone เพราะ `git-p4` จะดึงข้อมูลนี้มาใส่ commit โดยตรง

**การ sync การเปลี่ยนแปลงใหม่ระหว่าง transition:**

```bash
git p4 rebase
```

**การส่ง commit จาก Git กลับไป Perforce (สำหรับ parallel run):**

```bash
git p4 submit
```

### 4.3 ข้อจำกัดของ `git-p4` ที่ต้องรู้

1. **ประสิทธิภาพต่ำกับ depot ขนาดใหญ่มาก** — `git-p4` clone ทีละ changelist ตามลำดับ ทำให้ repository ที่มีหลักแสน changelist ใช้เวลานานมาก (อาจเป็นหลักสัปดาห์)
2. **ไม่รองรับ Streams Depot อย่างสมบูรณ์** — โครงสร้าง Stream ที่ซับซ้อน (parent/child stream, virtual stream) ต้อง map ด้วยมือค่อนข้างมาก
3. **ไฟล์ binary และ integration history** — `git-p4` ไม่ได้แปลง Perforce "integration record" (ประวัติการ merge ระหว่าง branch ของ Perforce) มาเป็น Git merge commit โดยอัตโนมัติเสมอไป มักได้ประวัติแบบเส้นตรง (linear) เท่านั้น

### 4.4 เครื่องมือเฉพาะทางสำหรับ Enterprise: Josh

สำหรับองค์กรขนาดใหญ่ที่มี depot ระดับหลายร้อย GB ถึงระดับ TB และมี changelist นับล้าน เครื่องมือ open source อย่าง **Josh** (Just One Single History) และเครื่องมือเชิงพาณิชย์อื่น ๆ เช่น **p4-fusion** (จาก Salesforce, เขียนด้วย C++ เพื่อความเร็ว) มักถูกเลือกใช้แทน `git-p4` มาตรฐาน เพราะ:

- **p4-fusion** สามารถ parallelize การดึง changelist ได้ ทำให้เร็วกว่า `git-p4` มาก (เร็วกว่าหลายสิบเท่าในบาง benchมาrk) เหมาะกับ depot ขนาดหลักล้าน changelist
- **Josh** เน้นการทำ monorepo-to-multi-repo filtering แบบ real-time ผ่าน virtual Git remote ทำให้สามารถ "ตัดเฉพาะบางส่วน" ของ depot ขนาดใหญ่มาเป็น Git repository ย่อยได้อย่างมีประสิทธิภาพ และยังรองรับการ sync สองทางระหว่าง monorepo กับ Git repo ย่อยได้ต่อเนื่อง เหมาะกับองค์กรที่อยากค่อย ๆ แตก monorepo ของ Perforce ออกเป็นหลาย repository ใน Git

**แนวทางเลือกเครื่องมือสำหรับ Perforce migration:**

| ขนาด Depot | Changelist | เครื่องมือแนะนำ |
|---|---|---|
| เล็ก–กลาง (< 50GB) | < 50,000 | `git-p4` มาตรฐานเพียงพอ |
| ใหญ่ (50GB–500GB) | 50,000–500,000 | `p4-fusion` เพื่อความเร็ว |
| ใหญ่มาก (> 500GB), ต้องการแตก monorepo | > 500,000 | Josh หรือบริการ migration เชิงพาณิชย์ (เช่น ทีมที่ปรึกษาเฉพาะทาง) |

### 4.5 ตัวอย่างการใช้ p4-fusion เบื้องต้น

```bash
# ติดตั้ง (build จาก source หรือดาวน์โหลด binary release)
git clone https://github.com/salesforce/p4-fusion.git
cd p4-fusion && ./generate_cache.sh && cmake -B build && cmake --build build

# รัน migration
./build/p4-fusion \
  --path "//depot/myproject/..." \
  --user jsmith \
  --port perforce.example.com:1666 \
  --client jsmith-migration-client \
  --src ./p4-workspace \
  --networkThreads 32 \
  --printBatch 1000 \
  --lookAhead 2000 \
  --fsyncEnable \
  --noColorDiff
```

ผลลัพธ์จะเป็น bare Git repository ที่มี commit หนึ่งต่อหนึ่ง changelist ของ Perforce ซึ่งจากนั้นต้องนำไป push เข้า Git server จริง (GitHub/GitLab) พร้อมตรวจสอบตาม Step 929

ต่อไปเราจะมาดูปัญหาสำคัญที่เกิดขึ้นทั้งจาก SVN และ Perforce เหมือนกัน คือเรื่องการแปลงโครงสร้าง branch/tag

---

## Step 925: การแปลงโครงสร้าง branch/tag จาก SVN (ที่เป็นแค่โฟลเดอร์) ให้เข้ากับรูปแบบ branch/tag จริงของ Git

นี่คือหนึ่งในจุดที่ทีมมือใหม่มักทำผิดพลาดมากที่สุด เพราะแนวคิดเรื่อง branch/tag ของ SVN และ Git **แตกต่างกันโดยพื้นฐาน**

### 5.1 ทำไมถึงต่างกัน

ใน **SVN**: repository ทั้งหมดคือ "ต้นไม้ไฟล์เดียว" (single filesystem tree) ที่มีเวอร์ชันตามเวลา การสร้าง branch คือคำสั่ง `svn copy` ที่ copy โฟลเดอร์ `trunk` ไปเป็นโฟลเดอร์ใหม่ใน `branches/feature-x` — ในทางเทคนิคแล้วมันเป็นแค่ **cheap copy** (copy-on-write ระดับ metadata ไม่ใช่การ copy ไฟล์จริงทั้งหมด) แต่ในมุมมองของ Git มันไม่ใช่ "branch" จริง ๆ มันคือโฟลเดอร์คู่ขนานภายใต้ tree เดียวกัน

ใน **Git**: branch คือ **pointer (reference)** ที่ชี้ไปยัง commit หนึ่ง ๆ ไม่ใช่โฟลเดอร์ และ tag คือ pointer ถาวรที่ชี้ไปยัง commit หนึ่ง ๆ เช่นกัน (ปกติไม่เปลี่ยนแปลงอีก)

### 5.2 สิ่งที่ `git svn --stdlayout` ทำให้อัตโนมัติ

เมื่อใช้ `--stdlayout` ตามที่แสดงใน Step 923 คำสั่ง `git svn` จะพยายามตีความ:

- โฟลเดอร์ `trunk` → กลายเป็น branch ชื่อ `svn/trunk` ใน remote-tracking references
- แต่ละโฟลเดอร์ย่อยใน `branches/` → กลายเป็น branch แยกกัน เช่น `svn/feature-x`, `svn/release-2.0`
- แต่ละโฟลเดอร์ย่อยใน `tags/` → กลายเป็น **branch ชั่วคราว** ก่อน (ไม่ใช่ Git tag จริง ๆ) เพราะ `git svn` ไม่รู้ล่วงหน้าว่าโฟลเดอร์ใน `tags/` จะไม่ถูกแก้ไขอีกในอนาคตหรือไม่ (ทาง technical แล้ว SVN ไม่ได้บังคับว่า tag ห้ามแก้ไข มันเป็นแค่ธรรมเนียมปฏิบัติเท่านั้น)

ดังนั้นหลังจาก clone เสร็จ เราต้อง **แปลง branch ชั่วคราวเหล่านี้ให้เป็น Git tag จริง** ด้วยตนเอง

### 5.3 ขั้นตอนแปลง SVN tags ให้เป็น Git tags จริง

```bash
cd my-project-git

# ดูรายการ "tag-like" branch ที่ git svn สร้างไว้
git branch -r | grep 'tags/'

# ตัวอย่างผลลัพธ์:
#   svn/tags/v1.0
#   svn/tags/v1.1
#   svn/tags/v2.0

# แปลงแต่ละอันเป็น annotated tag จริง แล้วลบ branch ชั่วคราวทิ้ง
for tag in $(git branch -r | grep 'tags/' | sed 's/svn\/tags\///'); do
  git tag -a "$tag" -m "Migrated from SVN tag: $tag" "refs/remotes/svn/tags/$tag"
  git branch -r -d "svn/tags/$tag"
done

# ตรวจสอบผลลัพธ์
git tag -l
```

### 5.4 ขั้นตอนแปลง SVN branches ให้เป็น Git local branches

```bash
# แปลงแต่ละ remote-tracking branch (ยกเว้น trunk และ tags) ให้เป็น local branch จริง
for branch in $(git branch -r | grep -v 'tags/' | grep -v 'trunk' | sed 's/svn\///'); do
  git branch "$branch" "refs/remotes/svn/$branch"
done

# แปลง trunk ให้เป็น main (ตามธรรมเนียมสมัยใหม่ของ Git)
git branch main svn/trunk
git symbolic-ref HEAD refs/heads/main

git branch -a
```

### 5.5 กรณีที่ SVN repository ไม่ได้ใช้ standard layout

ในหลายองค์กร โครงสร้าง SVN เดิมอาจไม่เป๊ะตามมาตรฐาน เช่น:

```
myproject/
├── main-dev/            ← ใช้แทน trunk
├── stable-branches/     ← ใช้แทน branches
│   └── v3-stable/
└── release-tags/        ← ใช้แทน tags
```

ในกรณีนี้ต้องระบุ path เองตอน clone (ตามที่แสดงใน Step 923.4) และหลังแปลงแล้ว ควรพิจารณาตั้งชื่อ branch ใหม่ให้เข้ากับธรรมเนียมของ Git สมัยใหม่ เช่น เปลี่ยน `stable-branches/v3-stable` เป็น `release/v3` เพื่อให้สอดคล้องกับ naming convention ที่เรียนไปใน Part ก่อนหน้าเรื่อง Git Flow / GitHub Flow

### 5.6 ปัญหาที่พบบ่อย: Branch ที่ถูกลบไปแล้วใน SVN

SVN เก็บประวัติของทุกอย่างรวมถึง branch ที่ถูกลบไปแล้ว (เพราะ "ลบ" ใน SVN คือแค่การสร้าง revision ใหม่ที่บอกว่า path นั้นไม่มีอยู่แล้ว ณ เวลานั้น) ทำให้เมื่อ clone ด้วย `git svn` บาง branch ที่ควรจะหายไปนานแล้วอาจโผล่มาเป็น branch ที่ "จบตัน" (dead-end) ใน Git

**แนวทางแก้ไข:** ทำ audit รายการ branch ทั้งหมดหลัง clone แล้วตัดสินใจร่วมกับทีมว่า branch ไหนควรเก็บไว้เป็น archive tag (เผื่อมีคนอยากดูย้อนหลัง) และ branch ไหนควรลบทิ้งไปเลยเพื่อไม่ให้ repository รกเกินไป:

```bash
# ดูวันที่ commit ล่าสุดของแต่ละ branch เพื่อประเมินว่า branch ไหน "ตาย" ไปแล้ว
for b in $(git branch -r | grep -v trunk); do
  echo "$b: $(git log -1 --format=%ci $b)"
done | sort -k2
```

Branch ที่ commit ล่าสุดห่างไปหลายปีและไม่มีใครอ้างอิงถึงแล้ว ควรแปลงเป็น **archive tag** เช่น `archive/feature-x-2016` แทนที่จะเก็บเป็น branch ที่ใช้งานจริง เพื่อให้รายการ branch ที่ทีมเห็นบน GitHub/GitLab สะอาดและไม่สับสน

### 5.7 กรณี Perforce: การแปลง Stream เป็น Git branch

สำหรับ Perforce ที่ใช้ Streams Depot การ map จะตรงไปตรงมากว่า SVN เพราะ stream มีแนวคิด parent-child ที่ชัดเจนอยู่แล้ว:

```
//streams/main       → main
//streams/dev-teamA  → dev-teamA
//streams/release-3  → release/3
```

`git-p4` และ `p4-fusion` จะสร้าง branch ตาม path ของ stream ให้อัตโนมัติในระดับหนึ่ง แต่ยังคงต้องตรวจสอบว่า naming ตรงกับธรรมเนียมใหม่ของ Git หรือไม่ และควรรัน `git branch -m` เพื่อ rename ให้เข้ากับมาตรฐานทีมก่อน push ขึ้น server จริง

---

## Step 926: การจัดการไฟล์ binary ขนาดใหญ่ระหว่าง migrate (เชื่อมโยงกับ Git LFS จาก Part 62)

ดังที่กล่าวใน Step 922 ไฟล์ binary ขนาดใหญ่คือความท้าทายที่หนักที่สุดของการ migrate จาก SVN/Perforce เพราะทั้งสองระบบสามารถจัดการไฟล์ขนาดใหญ่ได้ดีกว่า Git โดย native ในขณะที่ Git ต้องอาศัย **Git LFS (Large File Storage)** ที่เราเรียนไปแล้วใน **Part 62**

### 6.1 ทบทวนสั้น ๆ ว่าทำไม Git ธรรมดาถึงจัดการ binary ไม่ดี

Git เก็บทุก object เป็น snapshot แบบ compressed (ตามที่เรียนใน Part 01 และ Part 56) สำหรับไฟล์ text การทำ delta compression ระหว่างเวอร์ชันได้ผลดีมาก แต่สำหรับไฟล์ binary (เช่น `.psd`, `.fbx`, `.uasset`, `.zip`) การเปลี่ยนแปลงเพียงเล็กน้อยมักทำให้ไฟล์ทั้งไฟล์เปลี่ยนไบต์เกือบทั้งหมด ทำให้ delta compression ได้ผลแทบเป็นศูนย์ — ทุกเวอร์ชันของไฟล์ถูกเก็บแบบเกือบเต็มขนาดจริงใน `.git`

Git LFS แก้ปัญหานี้ด้วยการเก็บ **pointer file** ขนาดเล็ก (ไม่กี่ร้อยไบต์) ไว้ใน Git repository แทนที่จะเก็บไฟล์ binary จริง แล้วเก็บไฟล์ binary จริงไว้ใน LFS storage แยกต่างหาก (ดาวน์โหลดตามต้องการ)

### 6.2 กลยุทธ์การ migrate binary จาก SVN มาสู่ Git LFS

**ขั้นตอนที่ 1: ระบุประเภทไฟล์ที่ต้องเข้า LFS**

```bash
# วิเคราะห์ repository ที่ clone มาแล้วว่าไฟล์ประเภทไหนใหญ่ที่สุด
git rev-list --objects --all \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | awk '/^blob/ {print substr($0,6)}' \
  | sort -k2 -n -r \
  | head -50
```

คำสั่งนี้จะไล่ดู object ทั้งหมดใน history และเรียงจากไฟล์ที่กิน storage มากที่สุด ช่วยให้เห็นภาพชัดว่าอะไรควรย้ายเข้า LFS

**ขั้นตอนที่ 2: ใช้ `git lfs migrate` เพื่อ rewrite ประวัติ**

Git LFS มีเครื่องมือ `git lfs migrate import` ที่ **rewrite ประวัติทั้งหมด** ให้ไฟล์ประเภทที่กำหนดถูกแทนที่ด้วย pointer file ย้อนหลังไปถึง commit แรกสุด:

```bash
git lfs install

git lfs migrate import \
  --include="*.psd,*.fbx,*.uasset,*.png,*.zip" \
  --everything
```

- `--include` — ระบุ pattern ของไฟล์ที่ต้องการย้ายเข้า LFS
- `--everything` — rewrite ทุก branch และ tag ในประวัติ ไม่ใช่แค่ branch ปัจจุบัน (สำคัญมากสำหรับ repository ที่ migrate มาจาก SVN ซึ่งมีหลาย branch/tag)

**คำเตือนสำคัญ:** คำสั่งนี้ **rewrite commit hash ทั้งหมด** เหมือนกับ `git filter-repo` เพราะเนื้อหาของทุก commit ที่มีไฟล์เหล่านี้เปลี่ยนไป ต้องทำ **ก่อน** push เข้า server จริงเท่านั้น ไม่ควรทำหลังจากที่ทีมเริ่มทำงานบน repository ใหม่แล้ว

**ขั้นตอนที่ 3: ตรวจสอบขนาด repository ก่อน-หลัง**

```bash
# ก่อน migrate
du -sh .git

# หลัง migrate ต้อง gc เพื่อเก็บ object เก่าที่ไม่ใช้แล้ว
git reflog expire --expire=now --all
git gc --prune=now --aggressive

du -sh .git
```

ควรเห็นขนาด `.git` ลดลงอย่างมีนัยสำคัญ (มักลดลง 70-95% สำหรับ repository ที่มีสัดส่วนไฟล์ binary สูง)

### 6.3 กรณี Perforce: จัดการเรื่อง Exclusive Lock

จุดที่ต้องเตือนทีมล่วงหน้าให้ชัดเจนคือ Perforce มีกลไก **exclusive checkout/lock** ในตัว (ใช้ `p4 edit` แบบ `+l` filetype modifier) ที่ป้องกันไม่ให้สองคนแก้ไฟล์ binary เดียวกันพร้อมกันโดยอัตโนมัติ แต่ Git ไม่มีสิ่งนี้ในตัวจริง ๆ

Git LFS แก้ปัญหานี้บางส่วนด้วยฟีเจอร์ **File Locking**:

```bash
# ตั้งค่าไฟล์ประเภทไหนที่ต้อง lock ก่อนแก้ไข (ระบุใน .gitattributes)
git lfs track "*.psd" --lockable
git lfs track "*.fbx" --lockable

# artist ต้อง lock ไฟล์ก่อนแก้ไขเสมอ (เป็นข้อตกลงของทีม ไม่ใช่การบังคับระดับ filesystem)
git lfs lock assets/character-hero.fbx

# ดูว่าใคร lock ไฟล์ไหนอยู่บ้าง
git lfs locks

# ปลด lock หลังแก้เสร็จและ push แล้ว
git lfs unlock assets/character-hero.fbx
```

**ข้อควรเข้าใจ:** การ lock ของ Git LFS เป็น "ข้อตกลงทางสังคม" ที่ต้อง enforce ผ่าน server-side hook หรือ policy ของ GitHub/GitLab (เช่น branch protection ที่เช็คสถานะ lock ก่อนรับ push) ไม่ใช่การล็อกระดับ filesystem แบบ Perforce ต้องอธิบายความแตกต่างนี้ให้ทีม artist/designer เข้าใจชัดเจนตั้งแต่ก่อน migrate เพื่อไม่ให้เกิดการเขียนทับงานกันโดยไม่ตั้งใจ

### 6.4 ทางเลือกอื่นสำหรับ binary จำนวนมหาศาลจริง ๆ

สำหรับองค์กรที่มี binary asset ขนาดรวมหลาย TB (เช่นสตูดิโอเกมขนาดใหญ่) การย้ายทุกอย่างเข้า Git LFS เต็มรูปแบบอาจไม่ใช่คำตอบที่ดีที่สุดเสมอไป ทางเลือกที่ควรพิจารณาร่วมด้วย:

1. **Hybrid approach** — เก็บ source code ไว้ใน Git ตามปกติ แต่ยังคง asset binary ขนาดใหญ่มาก ๆ ไว้ใน Perforce ต่อไป (ใช้ submodule หรือ build-time fetch เชื่อมสองระบบ) เหมาะกับองค์กรที่ยังไม่พร้อม migrate ทุกอย่างในครั้งเดียว
2. **Git Virtual File System (VFS for Git) หรือ partial clone/sparse checkout** — สำหรับ repository ขนาดใหญ่มากที่ยังต้องการรวมทุกอย่างไว้ใน Git เดียว ช่วยให้ผู้ใช้ดาวน์โหลดเฉพาะไฟล์ที่ต้องการจริง ๆ
3. **จำกัดความลึกของประวัติ binary** — พิจารณา "ตัดประวัติ" ของไฟล์ binary เก่าที่ไม่มีใครใช้แล้ว (เก็บแค่ snapshot ปัจจุบันใน LFS โดยไม่ import ประวัติเก่าทั้งหมด) แล้วเก็บ SVN/Perforce เดิมไว้เป็น read-only archive สำหรับอ้างอิงประวัติเก่าแทน

การตัดสินใจเรื่องนี้ควรทำร่วมกับทีมที่เกี่ยวข้องโดยตรง (เช่นทีม art, ทีม infrastructure) ก่อนเริ่ม migrate จริง ไม่ใช่ตัดสินใจฝ่ายเดียวโดยทีม engineering เท่านั้น

---

## Step 927: Parallel Run Strategy — รันทั้งสองระบบคู่ขนานกันในช่วง transition เพื่อลดความเสี่ยง

การ migrate แบบ "big bang" (เปลี่ยนทีเดียวทั้งองค์กรในวันเดียว) มีความเสี่ยงสูงมาก โดยเฉพาะกับทีมขนาดใหญ่ที่พึ่งพา repository นี้ในการทำงานประจำวัน กลยุทธ์ที่ปลอดภัยกว่าคือ **Parallel Run** — รันทั้งสองระบบคู่ขนานกันชั่วคราว

### 7.1 แนวคิดหลักของ Parallel Run

```
ช่วงที่ 1: SVN เป็นระบบหลัก, Git เป็น mirror สำหรับทดสอบ
ช่วงที่ 2: Git เป็นระบบหลัก, SVN เป็น read-only fallback
ช่วงที่ 3: ปิด SVN ถาวร, เหลือแต่ Git
```

เป้าหมายคือให้ทีมมีเวลาปรับตัวและมี "ทางถอย" ถ้าเกิดปัญหาร้ายแรงระหว่างทาง โดยไม่ต้อง rollback การตัดสินใจทั้งองค์กรในทันที

### 7.2 ช่วงที่ 1: Read-only Mirror (สัปดาห์ที่ 1-4)

ในช่วงนี้ SVN ยังคงเป็นระบบหลักที่ทุกคนทำงานจริง แต่ตั้ง sync อัตโนมัติให้ Git repository อัปเดตตามทุกครั้งที่มี commit ใหม่เข้า SVN:

```bash
# รัน cron job หรือ scheduled CI job ทุก 15-30 นาที
cd my-project-git
git svn fetch
git svn rebase --local

# push เข้า Git server กลาง (GitHub/GitLab) เพื่อให้ทีมทดลองใช้งานได้
git push origin --all
git push origin --tags
```

ในช่วงนี้ทีมสามารถ:
- ทดลอง clone, ทดลอง CI/CD pipeline บน Git โดยไม่กระทบงานจริง
- เริ่ม training (Step 928) โดยใช้ repository ตัวนี้เป็นสนามฝึก
- ตรวจพบปัญหาการแปลงข้อมูลตั้งแต่เนิ่น ๆ (Step 929) ก่อนที่จะสายเกินไป

**ข้อควรระวัง:** ห้ามให้ใครเริ่ม commit ตรง ๆ เข้า Git repository ในช่วงนี้ เพราะ commit เหล่านั้นจะหายไปทุกครั้งที่ sync รอบใหม่ทับเข้ามา (เพราะ Git repository ฝั่งนี้เป็นแค่ mirror ที่ถูก rebase ทับอยู่ตลอด)

### 7.3 ช่วงที่ 2: Git เป็นหลัก, SVN เป็น Fallback (สัปดาห์ที่ 5-8)

เมื่อทีมมั่นใจว่า Git repository ถูกต้องครบถ้วนแล้ว (ผ่านการ validate ตาม Step 929) ให้เปลี่ยนทิศทางการ sync:

1. **ประกาศ Freeze Window สั้น ๆ** (เช่น 2-4 ชั่วโมงนอกเวลาทำงาน) ที่ห้ามใครแก้ไข SVN
2. **ทำ final sync ครั้งสุดท้าย** จาก SVN เข้า Git (`git svn fetch` รอบสุดท้าย)
3. **เปลี่ยน SVN repository เป็น read-only** (ปิดสิทธิ์ commit ทุกคนยกเว้น admin) เพื่อป้องกันไม่ให้มีใครแก้ SVN อีกโดยไม่ตั้งใจ
4. **เปิดให้ทีมเริ่ม push/pull บน Git repository จริง** ผ่าน pull request workflow ตามปกติ

ในช่วงนี้ให้คง SVN ไว้ในสถานะ read-only เป็นเวลา 4-8 สัปดาห์ เผื่อกรณีที่:
- มีคนต้องอ้างอิงประวัติเก่าที่อาจแปลงมาไม่ครบ 100%
- ระบบ external บางตัว (เช่น build server เก่า) ยังต้องพึ่งพา SVN URL อยู่ระหว่างที่ทีม infrastructure ทยอยปรับ config

### 7.4 ช่วงที่ 3: Decommission SVN (หลังสัปดาห์ที่ 12 เป็นต้นไป)

เมื่อผ่านช่วง fallback ไปแล้วโดยไม่มีปัญหาสำคัญเกิดขึ้น:

1. **สำรอง SVN repository แบบเต็ม** (full dump) เก็บไว้ใน cold storage เพื่อการ compliance/audit ในระยะยาว แม้จะไม่ใช้งานแล้ว
   ```bash
   svnadmin dump /path/to/svn/repo > full-backup-$(date +%Y%m%d).dump
   gzip full-backup-*.dump
   ```
2. **ปิดการเข้าถึง SVN server** อย่างเป็นทางการ (แต่ยังคง backup ไว้อย่างน้อย 1-3 ปีตามนโยบายองค์กร)
3. **อัปเดตเอกสารภายในองค์กรทั้งหมด** ให้ชี้ไปที่ Git repository ใหม่ (README, wiki, onboarding guide)
4. **ปิด service ที่เกี่ยวข้อง** เช่น SVN web viewer, SVN backup job, license ของ SVN/Perforce ถ้ามี

### 7.5 ตารางสรุป Parallel Run Timeline

| ช่วง | ระยะเวลาแนะนำ | ระบบหลัก | สถานะ Git |
|---|---|---|---|
| 1: Mirror | 4 สัปดาห์ | SVN | อ่านอย่างเดียว, sync อัตโนมัติ |
| 2: Fallback | 4-8 สัปดาห์ | Git | ใช้งานจริง, SVN เป็น read-only |
| 3: Decommission | หลังสัปดาห์ที่ 12 | Git | ระบบเดียวที่เหลืออยู่ |

**ข้อควรระวังสำคัญที่สุดของกลยุทธ์นี้:** ต้องสื่อสารกับทีมอย่างชัดเจนว่า **ช่วงไหนคือ "แหล่งความจริง" (source of truth)** เพียงหนึ่งเดียว ห้ามให้เกิดสถานการณ์ที่บางคนยัง commit เข้า SVN ในขณะที่บางคนเริ่ม push เข้า Git แล้วโดยไม่มีการ sync กัน เพราะจะทำให้ประวัติแตกแขนงและกู้คืนยากมาก ควรมีประกาศทางการ (announcement) ที่ชัดเจนทุกครั้งที่เปลี่ยนช่วง พร้อม deadline ที่แน่นอน

---

## Step 928: การ training ทีมให้คุ้นเคยกับ Git หลัง migrate (คนที่ใช้ SVN มา 10 ปีต้องปรับ mindset อย่างไร)

การย้ายเครื่องมือทางเทคนิคทำได้ไม่ยากเท่ากับการเปลี่ยน **mindset** ของคนที่ทำงานกับ SVN มา 10 ปี นี่คือหัวข้อที่ทีม migrate มักประเมินความสำคัญต่ำเกินไป และกลายเป็นสาเหตุอันดับหนึ่งที่ทำให้ migration ล้มเหลวในทางปฏิบัติ (แม้ข้อมูลจะย้ายมาสำเร็จ 100% ก็ตาม)

### 8.1 ความแตกต่างของ Mental Model ที่ต้องอธิบายให้ชัด

| แนวคิดใน SVN | แนวคิดที่เทียบเคียงใน Git | สิ่งที่ต้องเน้นย้ำ |
|---|---|---|
| `svn commit` = ส่งงานให้ทีมเห็นทันที | `git commit` (local) + `git push` (ส่งจริง) | commit local ไม่ใช่การแชร์งาน ต้อง push ด้วยเสมอ |
| `svn update` = ดึงงานล่าสุดจาก server | `git pull` = fetch + merge/rebase | อาจเกิด conflict บ่อยกว่าที่เคยเจอใน SVN |
| Revision number เรียงลำดับ (r1, r2, r3...) | Commit hash ไม่เรียงลำดับเชิงเวลาแบบเดียวกัน | ต้องใช้ `git log`, `git blame` แทนการจำเลขไว้ |
| Branch = โฟลเดอร์แยกที่ "หนัก" ในการสร้าง | Branch = pointer ที่ "เบา" สร้างได้ไม่จำกัด | ส่งเสริมให้สร้าง feature branch บ่อย ๆ โดยไม่ต้องกลัว |
| Lock ไฟล์ก่อนแก้ (โดยเฉพาะ Perforce) | ไม่มี lock โดย native (ยกเว้นตั้งค่า LFS lock) | ต้องสื่อสารกันเองมากขึ้น หรือใช้ LFS lock สำหรับ binary |
| Permission based on path (ACL) | Permission ผ่าน platform (branch protection, CODEOWNERS) | ต้องตั้งค่าใหม่ผ่าน GitHub/GitLab settings |

### 8.2 โครงสร้างโปรแกรม Training ที่แนะนำ

**ระดับ 1: Git Fundamentals (สำหรับทุกคน, บังคับ) — ครึ่งวัน**

- `git status`, `git add`, `git commit`, `git push`, `git pull` — คำสั่งพื้นฐานที่สุด
- ความแตกต่างระหว่าง Working Directory / Staging Area / Local Repository / Remote Repository
- เข้าใจว่า **local commit ไม่เท่ากับการแชร์งาน** — ต้อง `push` เสมอ (แก้ mindset ข้อ 1 ในตารางด้านบน)
- Workshop ภาคปฏิบัติ: จำลองสถานการณ์ที่เคยทำใน SVN แล้วให้ทำแบบเดียวกันใน Git

**ระดับ 2: Branching และ Collaboration (สำหรับ engineer, บังคับ) — 1 วัน**

- สร้าง feature branch, push, เปิด Pull Request/Merge Request
- การแก้ conflict เบื้องต้น (จำลองสถานการณ์ conflict จริงให้ฝึก)
- Code Review workflow ผ่าน Pull Request (แนวคิดที่ SVN ไม่มีมาก่อน)
- แนะนำ Git GUI tool สำหรับคนที่ไม่ถนัด command line (เช่น GitKraken, Sourcetree, GitHub Desktop, VS Code Git integration)

**ระดับ 3: Advanced Workflow (สำหรับ Tech Lead/Senior, สมัครใจ) — 1 วัน**

- Rebase vs Merge, เมื่อไหร่ควรใช้แบบไหน
- Git Flow / GitHub Flow ที่ทีมจะใช้จริงหลัง migrate
- การใช้ `git bisect` แก้บั๊ก, `git blame` วิเคราะห์ประวัติ
- Git LFS workflow สำหรับทีมที่ทำงานกับ binary asset (ต่อเนื่องจาก Step 926)

**ระดับ 4: Office Hours ต่อเนื่อง 4-8 สัปดาห์**

- จัดช่วงเวลาให้คำปรึกษาสด (office hours) สัปดาห์ละ 1-2 ครั้งในช่วง transition
- ตั้งช่องทาง chat เฉพาะ (เช่น `#git-migration-help`) ให้ถามคำถามได้ตลอดเวลา
- มอบหมาย "Git Champion" ในแต่ละทีมย่อย — คนที่เรียนรู้เร็วและช่วยสอนเพื่อนร่วมทีมต่อ (peer support model)

### 8.3 เทคนิคเฉพาะสำหรับคนที่ใช้ SVN มานาน (10+ ปี)

จากประสบการณ์ของทีม migrate จำนวนมาก พบรูปแบบความกังวลที่พบบ่อยในกลุ่มนี้ และวิธีรับมือ:

1. **ความกังวล: "ทำไมต้องมีขั้นตอนเยอะกว่าเดิม (add, commit, push)"**
   คำอธิบาย: เปรียบเทียบว่าขั้นตอนที่เพิ่มขึ้นแลกมาด้วยความสามารถทำงาน offline และมี "sandbox" ส่วนตัวก่อนแชร์งานจริง ซึ่งป้องกันไม่ให้โค้ดที่ยังไม่สมบูรณ์ไปกระทบทีมอื่นโดยไม่ตั้งใจ — เป็นข้อดีไม่ใช่ภาระ

2. **ความกังวล: "ทำไม branch เยอะจัง ดูสับสน"**
   คำอธิบาย: สอนให้ตั้งชื่อ branch ตามธรรมเนียม (`feature/`, `bugfix/`, `release/`) ที่เรียนไปแล้วใน Part ก่อนหน้า และแสดงให้เห็นว่า branch ที่ merge แล้วสามารถลบทิ้งได้โดยไม่กระทบประวัติ (ต่างจาก SVN ที่มักเก็บโฟลเดอร์ branch เก่าไว้ตลอดไปเพราะกลัว "ลบแล้วหาย")

3. **ความกังวล: "ข้อมูลหายไปหรือเปล่าถ้า force push โดยไม่ตั้งใจ"**
   คำอธิบาย: สอนเรื่อง `git reflog` ที่ช่วยกู้คืนได้เกือบทุกกรณี (คนละแนวคิดกับ SVN ที่การ revert มักซับซ้อนกว่า) พร้อมตั้งค่า **branch protection rule** บน main/release branch เพื่อป้องกัน force push โดยไม่ได้รับอนุญาตตั้งแต่ต้น

4. **ความกังวล: "งงเรื่อง merge conflict ที่ไม่เคยเจอบ่อยขนาดนี้ใน SVN"**
   คำอธิบาย: อธิบายว่าเป็นเพราะ SVN มักให้ทุกคนทำงานบน trunk เดียวกันเป็นหลัก (ทำให้ conflict เกิดน้อยแต่กระทบวงกว้างเมื่อเกิด) ในขณะที่ Git ส่งเสริมให้แยก branch ทำงาน conflict จึงเกิดบ่อยกว่าแต่ **ขอบเขตเล็กกว่าและแก้ง่ายกว่ามาก** เพราะเห็นเฉพาะส่วนที่ต่างกันจริง ๆ

### 8.4 การวัดผลความสำเร็จของ Training

ควรตั้ง metric ที่ตรวจสอบได้จริงเพื่อประเมินว่าทีมพร้อมหรือยัง เช่น:

- **จำนวน pull request ที่ทีมเปิดได้เองโดยไม่ต้องขอความช่วยเหลือ** ควรเพิ่มขึ้นต่อเนื่องทุกสัปดาห์
- **จำนวนคำถามใน office hours** ควรลดลงหลังสัปดาห์ที่ 4-6
- **แบบสอบถามความมั่นใจ (confidence survey)** ก่อนและหลัง training — วัดคะแนนความมั่นใจในการใช้คำสั่งพื้นฐาน 1-5
- **จำนวนครั้งที่ต้องขอ admin ช่วยกู้คืนจากความผิดพลาด** ควรลดลงเมื่อเวลาผ่านไป

---

## Step 929: Post-migration Validation — ตรวจสอบว่าประวัติถูกย้ายมาครบถ้วนถูกต้อง (เทียบจำนวน commit, hash ของไฟล์)

การ migrate ที่ดีต้องพิสูจน์ได้ว่าข้อมูลถูกย้ายมา "ครบถ้วนและถูกต้อง" ไม่ใช่แค่ "ดูเหมือนจะโอเค" ขั้นตอนนี้สำคัญมากโดยเฉพาะสำหรับองค์กรที่มีข้อกำหนดด้าน compliance หรือ audit trail

### 9.1 ตรวจสอบจำนวน Commit/Revision ให้ตรงกัน

**สำหรับ SVN:**

```bash
# นับจำนวน revision ทั้งหมดใน SVN (เฉพาะ path ของ trunk)
svn log http://svn.example.com/repos/my-project/trunk -q | grep -c '^r'

# นับจำนวน commit ใน Git branch ที่แปลงมาจาก trunk
git log main --oneline | wc -l
```

ตัวเลขทั้งสองไม่จำเป็นต้องเท่ากันเป๊ะ 100% เสมอไป (เพราะ revision ของ SVN นับรวมทุก path ในขณะที่การ filter เฉพาะ `trunk` อาจตัดบาง revision ที่ไม่กระทบ path นั้นออกไป) แต่ **ต้องอธิบายส่วนต่างได้ทุกกรณี** ห้ามมีส่วนต่างที่ไม่มีคำอธิบาย

**สำหรับ Perforce:**

```bash
# นับจำนวน changelist ทั้งหมดของ depot path
p4 changes -s submitted //depot/myproject/... | wc -l

# นับจำนวน commit ใน Git ที่แปลงมา
git log main --oneline | wc -l
```

### 9.2 ตรวจสอบความถูกต้องของเนื้อหาไฟล์ (Content Hash Comparison)

การนับจำนวน commit อย่างเดียวไม่พอ — ต้องตรวจสอบว่า **เนื้อหาไฟล์ ณ จุดสุดท้าย** ตรงกันทุกไบต์ระหว่างระบบเก่ากับระบบใหม่ด้วย

```bash
# Checkout เวอร์ชันล่าสุดจาก SVN ไปไว้ที่โฟลเดอร์แยก
svn checkout http://svn.example.com/repos/my-project/trunk svn-latest-check

# Checkout เวอร์ชันล่าสุดจาก Git repository ที่แปลงมา
git clone my-project-git.git git-latest-check
cd git-latest-check && git checkout main

# เปรียบเทียบไฟล์ทั้งหมด (ยกเว้น metadata directory ของแต่ละระบบ)
diff -rq \
  --exclude=".svn" \
  --exclude=".git" \
  ../svn-latest-check \
  .
```

ถ้าคำสั่ง `diff -rq` ไม่แสดงผลลัพธ์อะไรเลย แปลว่าไฟล์ทั้งหมดตรงกันทุกไบต์ ถ้ามีความต่าง ต้องไล่ตรวจแต่ละไฟล์ว่าเป็นเพราะ:

- Line ending แตกต่างกัน (CRLF vs LF) — มักเกิดจากการตั้งค่า `core.autocrlf` หรือ `svn:eol-style` ที่ไม่ตรงกัน
- Keyword substitution ของ SVN (`svn:keywords` เช่น `$Id$`, `$Revision$`) ที่ไม่มีการแปลงเทียบเท่าใน Git โดยตรง (ต้องตัดสินใจว่าจะลบ keyword เหล่านี้ทิ้งหรือแปลงเป็นข้อความคงที่)
- ไฟล์ที่ถูก `svn:ignore` หรือ `p4 ignore` กันไว้ ซึ่งอาจไม่ถูกแปลงเข้ามาโดยตั้งใจ (ต้องยืนยันว่าตรงตามที่ตั้งใจจริง)

### 9.3 ตรวจสอบ Checksum ระดับไฟล์แบบอัตโนมัติ

สำหรับ repository ขนาดใหญ่ที่ diff ด้วยตาไม่ไหว ให้เขียนสคริปต์เปรียบเทียบ checksum ของทุกไฟล์แบบอัตโนมัติ:

```bash
#!/bin/bash
# compare-checksums.sh

find ../svn-latest-check -type f -not -path "*/.svn/*" \
  | sed "s|../svn-latest-check/||" \
  | sort > /tmp/svn-files.txt

find . -type f -not -path "*/.git/*" \
  | sed "s|^\./||" \
  | sort > /tmp/git-files.txt

echo "=== ไฟล์ที่มีใน SVN แต่ไม่มีใน Git ==="
comm -23 /tmp/svn-files.txt /tmp/git-files.txt

echo "=== ไฟล์ที่มีใน Git แต่ไม่มีใน SVN ==="
comm -13 /tmp/svn-files.txt /tmp/git-files.txt

echo "=== เปรียบเทียบ checksum ของไฟล์ที่มีทั้งสองฝั่ง ==="
comm -12 /tmp/svn-files.txt /tmp/git-files.txt | while read -r f; do
  svn_hash=$(sha256sum "../svn-latest-check/$f" | awk '{print $1}')
  git_hash=$(sha256sum "$f" | awk '{print $1}')
  if [ "$svn_hash" != "$git_hash" ]; then
    echo "MISMATCH: $f (svn=$svn_hash, git=$git_hash)"
  fi
done

echo "ตรวจสอบเสร็จสิ้น"
```

รายงานที่ได้ควรมี **ศูนย์ mismatch** ก่อนจะประกาศว่า migration สำเร็จอย่างเป็นทางการ

### 9.4 ตรวจสอบ Author Mapping ว่าครบถ้วน

```bash
# ดูรายชื่อ author ทั้งหมดที่ปรากฏใน Git หลัง migrate
git log --all --format='%an <%ae>' | sort -u

# ตรวจสอบว่าไม่มี pattern แปลก ๆ หลงเหลืออยู่ เช่น username@repo-uuid หรือ "(no author)"
git log --all --format='%an <%ae>' | sort -u | grep -E '@[a-f0-9-]{36}|no author'
```

ถ้าคำสั่งสุดท้ายเจอผลลัพธ์ใด ๆ แปลว่ายังมี author mapping ที่ตกหล่น ต้องกลับไปแก้ authors-file แล้ว re-run การ migrate ใหม่ตั้งแต่ต้น (ไม่ควรแก้ author หลังจากประวัติถูก push เข้า server จริงแล้ว เพราะจะกลายเป็นการ rewrite history ที่กระทบทุกคน)

### 9.5 ตรวจสอบ Branch/Tag ให้ครบตามต้นฉบับ

```bash
# เทียบจำนวน tag
svn list http://svn.example.com/repos/my-project/tags | wc -l
git tag -l | wc -l

# เทียบจำนวน branch ที่ยังใช้งานอยู่ (ไม่รวมที่จงใจ archive/ลบทิ้งตาม Step 925.6)
svn list http://svn.example.com/repos/my-project/branches | wc -l
git branch -a | grep -v 'tags/' | wc -l
```

ถ้าจำนวนไม่ตรงกัน ต้องมีเอกสารบันทึกเหตุผลไว้ชัดเจน (เช่น "branch feature-old-2015 ถูกจงใจไม่ import เพราะไม่มีการใช้งานมา 8 ปีและทีมเจ้าของ sign-off แล้วว่าไม่ต้องการ")

### 9.6 สร้าง Migration Validation Report อย่างเป็นทางการ

ก่อนประกาศว่า migration สำเร็จ ควรมีเอกสารสรุปผลการตรวจสอบที่ลงนามรับรองโดยทีมที่เกี่ยวข้อง ประกอบด้วยอย่างน้อย:

| หัวข้อตรวจสอบ | ผลลัพธ์ | ผู้ตรวจสอบ | วันที่ |
|---|---|---|---|
| จำนวน commit/revision ตรงกัน (พร้อมคำอธิบายส่วนต่างถ้ามี) | ผ่าน/ไม่ผ่าน | | |
| Checksum ไฟล์ล่าสุดตรงกัน 100% | ผ่าน/ไม่ผ่าน | | |
| Author mapping ครบถ้วน ไม่มี placeholder หลงเหลือ | ผ่าน/ไม่ผ่าน | | |
| Branch/Tag ครบตามที่ตกลงกันไว้ | ผ่าน/ไม่ผ่าน | | |
| ไฟล์ binary ทดสอบเปิดใช้งานได้ปกติหลังผ่าน Git LFS | ผ่าน/ไม่ผ่าน | | |
| CI/CD pipeline ทดสอบ build สำเร็จบน Git repository ใหม่ | ผ่าน/ไม่ผ่าน | | |
| ทีมตัวแทนแต่ละแผนกทดลองใช้งานจริง 1 สัปดาห์แล้วไม่พบปัญหาสำคัญ | ผ่าน/ไม่ผ่าน | | |

เมื่อทุกข้อผ่านและมีการลงนามรับรองแล้วเท่านั้น จึงควรเดินหน้าเข้าสู่ Step 927 ช่วงที่ 2 (Git เป็นระบบหลัก)

---

## Step 930: แบบฝึกหัด — วางแผน migration timeline แบบละเอียดให้องค์กรสมมติที่ใช้ SVN มา 10 ปี มีทีม 50 คน

### 10.1 โจทย์

บริษัทสมมติชื่อ **"NorthBridge Software"** มีลักษณะดังนี้:

- ใช้ SVN มาตั้งแต่ปี 2016 (10 ปี) มี repository หลัก 1 ตัวขนาดประมาณ 45 GB (รวมประวัติ) มี revision ล่าสุดอยู่ที่ประมาณ **revision 187,000**
- มีทีมวิศวกร 50 คน แบ่งเป็น 6 ทีมย่อย (Backend, Frontend, Mobile, QA, DevOps, Data)
- มี branch ที่ยังใช้งานจริงอยู่ 12 branch และ tag ของ release ทั้งหมด 64 tag
- มีไฟล์ binary ขนาดใหญ่ (สื่อการตลาด, ไฟล์ design mockup) ประมาณ 8 GB ปะปนอยู่ในบาง path
- CI/CD ปัจจุบันเป็น Jenkins ที่ผูก script กับคำสั่ง `svn` โดยตรงในหลาย pipeline
- มีระบบ issue tracking ภายในที่อ้างอิง SVN revision number ในทุก ticket ย้อนหลังไปหลายพันใบ
- ผู้บริหารต้องการให้ migration เสร็จสมบูรณ์ภายใน **16 สัปดาห์** โดยกระทบงานประจำวันของทีมให้น้อยที่สุด

**โจทย์:** จงวางแผน migration timeline แบบละเอียด ระบุ:

1. ทีมงานที่ต้องเกี่ยวข้องและบทบาทของแต่ละคน
2. Timeline แบ่งเป็นสัปดาห์ พร้อมงานหลักของแต่ละสัปดาห์
3. เกณฑ์ (criteria) ที่ใช้ตัดสินว่าจะเข้าสู่ช่วงถัดไปได้หรือไม่ (go/no-go decision point)
4. แผนสำรอง (rollback plan) หากพบปัญหาร้ายแรงระหว่างทาง

### 10.2 แนวทางเฉลย (Sample Solution)

**ทีมงานที่เกี่ยวข้อง:**

| บทบาท | จำนวนคน | หน้าที่หลัก |
|---|---|---|
| Migration Lead | 1 | ควบคุมภาพรวม ตัดสินใจ go/no-go ทุกช่วง |
| Git/DevOps Engineer | 2 | รัน `git svn`, ตั้งค่า LFS, เขียน CI/CD ใหม่ |
| QA Validator | 2 | รัน validation script (Step 929), ตรวจสอบ checksum |
| Training Facilitator | 1 | ออกแบบและจัดอบรม (Step 928) |
| Git Champion ต่อทีมย่อย | 6 (1 ต่อทีม) | ช่วยสอนเพื่อนร่วมทีม, รวบรวมปัญหาแจ้งกลับ |
| ตัวแทนผู้บริหาร (Sponsor) | 1 | อนุมัติ freeze window, สื่อสารกับทั้งองค์กร |

**Timeline แบ่งเป็น 16 สัปดาห์:**

**สัปดาห์ที่ 1-2 — เตรียมการและวิเคราะห์ (Discovery)**
- รวบรวมรายชื่อ author ทั้งหมดจาก `svn log` (ตาม Step 923.3) และสร้าง authors-file ฉบับร่าง
- วิเคราะห์ path ที่มีไฟล์ binary ใหญ่ (ตาม Step 926.2) เพื่อวางแผน LFS pattern
- audit branch/tag ทั้งหมด ตัดสินใจว่า branch ไหนควร archive (ตาม Step 925.6)
- ตั้งค่า environment ทดสอบ (staging) สำหรับรัน `git svn clone` แบบเต็ม
- **Go/No-go:** authors-file ครอบคลุม 100% ของ username ที่พบใน SVN log

**สัปดาห์ที่ 3-5 — Migration ทดลอง (Trial Migration)**
- รัน `git svn clone --stdlayout` แบบเต็มบน environment ทดสอบ (ใช้ incremental fetch ตาม Step 923.5 เพราะ revision เยอะ)
- แปลง tag ชั่วคราวเป็น Git tag จริง, แปลง branch (ตาม Step 925)
- รัน `git lfs migrate import` สำหรับไฟล์ binary 8 GB (ตาม Step 926.2)
- รัน validation script ชุดแรก (ตาม Step 929) เทียบจำนวน commit และ checksum
- **Go/No-go:** checksum mismatch ต้องเป็นศูนย์, จำนวน commit อธิบายส่วนต่างได้ครบทุกกรณี

**สัปดาห์ที่ 6-7 — เตรียม Infrastructure**
- เขียน CI/CD pipeline ใหม่บน Jenkins (หรือพิจารณาย้ายไป GitHub Actions/GitLab CI ควบคู่กันถ้าองค์กรต้องการ) ให้ทำงานกับ Git แทน `svn`
- ตั้งค่า Git server จริง (GitHub Enterprise/GitLab Self-managed) พร้อม branch protection, CODEOWNERS ให้ครอบคลุม permission เดิมที่เคยมีใน SVN
- Map SVN revision number เก่ากับ Git commit hash ใหม่ ส่งให้ทีม issue tracking ใช้เป็น reference table (แก้ปัญหาที่ ticket เก่าอ้างอิง revision number)
- **Go/No-go:** CI pipeline ทดสอบ build สำเร็จบน Git repository อย่างน้อย 3 ครั้งติดต่อกัน

**สัปดาห์ที่ 8 — เริ่ม Parallel Run ช่วงที่ 1 (Mirror)**
- ตั้ง cron sync `git svn fetch` ทุก 30 นาที (ตาม Step 927.2)
- เปิดให้ทีมเข้าถึง Git repository แบบ read-only เพื่อทดลอง clone/สำรวจ

**สัปดาห์ที่ 9-11 — Training เต็มรูปแบบ**
- จัด Training ระดับ 1 (Git Fundamentals) ให้ทุกคน 50 คน แบ่งเป็นรอบละ 10-15 คน
- จัด Training ระดับ 2 (Branching/Collaboration) ให้ engineer ทุกคน
- จัด Training ระดับ 3 (Advanced) ให้ Tech Lead และ Git Champion ของแต่ละทีม
- เปิด office hours สัปดาห์ละ 2 ครั้ง
- **Go/No-go:** ผลสำรวจความมั่นใจเฉลี่ยของทีม ≥ 3.5/5 และ Git Champion ทุกทีมยืนยันว่าทีมตนพร้อม

**สัปดาห์ที่ 12 — Freeze Window และสลับระบบหลัก**
- ประกาศ Freeze Window ล่วงหน้า 1 สัปดาห์ (เช่น วันศุกร์เย็นถึงเช้าวันจันทร์)
- รัน final sync ครั้งสุดท้ายจาก SVN เข้า Git
- เปลี่ยน SVN เป็น read-only (ปิดสิทธิ์ commit ทุกคนยกเว้น admin)
- เปิดให้ทีมเริ่มทำงานบน Git repository จริงในเช้าวันจันทร์
- **นี่คือจุดเข้าสู่ Parallel Run ช่วงที่ 2 (Git เป็นหลัก, SVN เป็น fallback)**

**สัปดาห์ที่ 13-15 — Stabilization**
- Git Champion และ Migration Lead ติดตามปัญหาที่เกิดขึ้นจริงอย่างใกล้ชิด
- แก้ไข edge case ที่พบ (เช่น script ภายในบางตัวที่ยังลืมแก้ให้ใช้ `git` แทน `svn`)
- คง SVN ไว้ในสถานะ read-only เผื่อฉุกเฉิน
- ทำ validation รอบสุดท้ายอีกครั้งเพื่อยืนยันว่าไม่มีข้อมูลตกหล่นจากการทำงานจริงในช่วงนี้

**สัปดาห์ที่ 16 — Decommission และปิดโครงการ**
- สำรอง SVN แบบเต็ม (`svnadmin dump`) เก็บเข้า cold storage อย่างน้อย 3 ปีตามนโยบาย compliance
- ปิดการเข้าถึง SVN server อย่างเป็นทางการ
- จัดประชุมสรุปบทเรียน (retrospective) ร่วมกับทุกทีมที่เกี่ยวข้อง
- ส่งมอบ Migration Validation Report ฉบับสมบูรณ์ (ตาม Step 929.6) ให้ผู้บริหารลงนามปิดโครงการ

**แผนสำรอง (Rollback Plan):**

- ถ้าพบปัญหาร้ายแรงในสัปดาห์ที่ 12 ทันทีหลังสลับระบบ (เช่น พบว่าข้อมูลสำคัญตกหล่นซึ่งไม่ถูกตรวจพบใน validation) ให้เปิดสิทธิ์ commit กลับเข้า SVN ทันที และประกาศให้ทีมกลับไปทำงานบน SVN ชั่วคราว โดยยังคง Git repository ไว้เป็น "ของที่รอแก้ไข" ไม่ลบทิ้ง
- Rollback point ที่ชัดเจนที่สุดคือ **ก่อนสัปดาห์ที่ 12** เพราะยังไม่มีใคร commit ข้อมูลใหม่ตรง ๆ ลงบน Git repository จริง การ rollback จึงแทบไม่มีข้อมูลสูญหาย
- หลังผ่านสัปดาห์ที่ 12 ไปแล้ว การ rollback จะซับซ้อนขึ้นมาก (ต้อง merge งานที่ทำใน Git กลับเข้า SVN ด้วยมือ) จึงควรตั้งเกณฑ์ **go/no-go ให้เข้มงวดที่สุดในจุดนี้**

### 10.3 คำถามต่อยอดสำหรับผู้เรียน

หลังทำแบบฝึกหัดข้างต้นแล้ว ลองตอบคำถามเพิ่มเติมเหล่านี้เพื่อฝึกคิดวิเคราะห์ให้ลึกขึ้น:

1. ถ้าองค์กรนี้มีทีมที่ทำงาน remote คนละ time zone (เช่น มีทีมที่อเมริกาและเอเชีย) จะปรับ Freeze Window อย่างไรให้กระทบทุกฝั่งน้อยที่สุด
2. ถ้า budget training มีจำกัดมาก ควรตัดงบส่วนไหนออกก่อน โดยกระทบความเสี่ยงของ migration น้อยที่สุด
3. ถ้าหลัง migrate ไปแล้ว 2 เดือน พบว่ามี branch สำคัญที่ archive ทิ้งไปโดยไม่ตั้งใจ (ไม่ได้อยู่ใน 12 branch ที่วางแผนไว้) ควรทำอย่างไรโดยไม่กระทบทีมที่ทำงานอยู่แล้วบน Git repository จริง

คำถามเหล่านี้ไม่มีคำตอบที่ถูกต้องเพียงหนึ่งเดียว — เป้าหมายคือฝึกให้ผู้เรียนคิดถึงมิติความเสี่ยง คน และกระบวนการ ควบคู่ไปกับความถูกต้องทางเทคนิคเสมอ ซึ่งเป็นทักษะที่แยก **Git user ธรรมดา** ออกจาก **ผู้ที่นำการเปลี่ยนแปลงระดับองค์กร (Tech Lead / Migration Lead)** ได้อย่างแท้จริง

---

## สรุป Part 93

ใน Part นี้เราได้เรียนรู้ว่า:

1. องค์กรจำนวนมากยังใช้ SVN/Perforce ด้วยเหตุผลที่สมเหตุสมผล เช่น mental model ที่ง่ายกว่า, path-based permission, และความสามารถจัดการไฟล์ binary ขนาดใหญ่ของ Perforce แต่แรงผลักดันให้ migrate มาสู่ Git ก็แข็งแกร่งขึ้นเรื่อย ๆ จากตลาดแรงงาน, ecosystem CI/CD สมัยใหม่ และต้นทุน license
2. ความท้าทายหลักของการ migrate มีสามมิติ: ประวัติจำนวนมหาศาลที่ต้อง map author ให้ครบ, ไฟล์ binary ขนาดใหญ่ที่ Git จัดการได้ไม่ดีโดย native, และ mindset ของทีมที่คุ้นเคยกับ workflow เดิมมานาน
3. `git svn clone --stdlayout --authors-file` คือเครื่องมือหลักสำหรับ migrate จาก SVN พร้อมประวัติเต็มรูปแบบ ส่วน `git-p4`, `p4-fusion`, และ Josh คือทางเลือกสำหรับ migrate จาก Perforce ตามขนาดของ depot
4. SVN branch/tag เป็นเพียง "โฟลเดอร์" ไม่ใช่แนวคิดพิเศษเหมือน Git ต้องแปลง tag ชั่วคราวให้เป็น annotated tag จริง และแปลง branch ให้เข้ากับธรรมเนียมการตั้งชื่อสมัยใหม่
5. ไฟล์ binary ขนาดใหญ่ต้องย้ายเข้า Git LFS ด้วย `git lfs migrate import --everything` และตั้งค่า File Locking เพื่อทดแทน exclusive lock ของ Perforce ที่ Git ไม่มีโดย native
6. กลยุทธ์ Parallel Run (Mirror → Fallback → Decommission) ช่วยลดความเสี่ยงของการเปลี่ยนระบบทั้งองค์กรในคราวเดียว โดยให้ทีมมีเวลาปรับตัวและมีทางถอยเสมอ
7. การ training ทีมที่คุ้นเคยกับ SVN มานานต้องเน้นปรับ mental model เรื่อง local commit vs push, branching ที่เบา, และ conflict ที่เกิดบ่อยขึ้นแต่ขอบเขตเล็กลง ไม่ใช่แค่สอนคำสั่ง
8. Post-migration validation ต้องพิสูจน์ด้วยตัวเลขจริง ทั้งจำนวน commit, checksum ของไฟล์ทุกไฟล์, author mapping ที่ครบถ้วน และ branch/tag ที่ตรงตามแผน ก่อนจะประกาศว่า migration สำเร็จอย่างเป็นทางการ
9. การวางแผน migration timeline ระดับองค์กรต้องคำนึงถึงคน กระบวนการ และความเสี่ยง ควบคู่ไปกับความถูกต้องทางเทคนิค — นี่คือทักษะที่แยกผู้นำการเปลี่ยนแปลงระดับองค์กรออกจากผู้ใช้ Git ทั่วไป

### Checklist ก่อนไป Part 94

- [ ] เข้าใจเหตุผลที่องค์กรยังใช้ SVN/Perforce และเหตุผลที่ควร migrate
- [ ] ใช้ `git svn clone` พร้อม authors-file และ `--stdlayout` ได้อย่างถูกต้อง
- [ ] เข้าใจภาพรวมของ `git-p4`, `p4-fusion` และ Josh สำหรับ migrate จาก Perforce
- [ ] แปลง SVN tag ชั่วคราวเป็น Git annotated tag จริงได้
- [ ] ใช้ `git lfs migrate import` ย้ายไฟล์ binary เข้า Git LFS พร้อมตั้งค่า File Locking ได้
- [ ] ออกแบบ Parallel Run strategy สามช่วง (Mirror → Fallback → Decommission) ได้
- [ ] วางแผนโปรแกรม training สี่ระดับให้ทีมที่ไม่คุ้นเคย Git ได้
- [ ] เขียนและรัน validation script เปรียบเทียบ checksum และจำนวน commit ได้
- [ ] วางแผน migration timeline แบบละเอียดให้องค์กรขนาด 50 คนได้ครบทุกมิติ

**ต่อไป:** [Part 94: การ Debug และแก้ปัญหา Git ที่ซับซ้อนในองค์กร](./part-094-debug-ปัญหา-git-ซับซ้อน.md)
