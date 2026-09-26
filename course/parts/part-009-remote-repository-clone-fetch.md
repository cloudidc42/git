# Part 09: Remote Repository เบื้องต้น: clone, remote add, fetch

> **Step ในหลักสูตรนี้:** Step 81–90
> **เฟส:** 2 — ใช้คำสั่ง Git พื้นฐานให้คล่องแคล่ว และเริ่มทำงานร่วมกับ Repository ระยะไกล
> **เป้าหมายของ Part นี้:** เข้าใจว่า Remote Repository คืออะไร ความสัมพันธ์ระหว่าง local repo กับ remote repo เป็นอย่างไร เรียนรู้การ `clone` repository มาทำงาน การจัดการ remote หลายตัวด้วย `git remote`, การดึงข้อมูลอย่างปลอดภัยด้วย `git fetch` (ต่างจาก `pull` อย่างไร), แนวคิดเรื่อง remote-tracking branch, และปิดท้ายด้วยการจำลอง remote repository ของตัวเองด้วย bare repository เพื่อฝึกฝนโดยไม่ต้องพึ่งอินเทอร์เน็ตหรือบัญชี GitHub เลย

---

## สารบัญของ Part นี้

- Step 81: Remote Repository คืออะไร ความสัมพันธ์กับ Local Repo
- Step 82: `git clone` — คัดลอก Repo ทั้งหมดพร้อมประวัติ
- Step 83: `git remote -v` และการมี Remote หลายตัว (origin, upstream)
- Step 84: `git remote add` / `remove` / `rename`
- Step 85: `git fetch` คืออะไร ต่างจาก `pull` อย่างไร
- Step 86: Remote-tracking Branch คืออะไร (เช่น `origin/main`)
- Step 87: เปรียบเทียบ Local Branch กับ Remote-tracking Branch หลัง Fetch
- Step 88: `git ls-remote` — ดู Branch/Tag บน Remote โดยไม่ต้อง Clone
- Step 89: การตั้งค่า Upstream Tracking Branch
- Step 90: แบบฝึกหัด — สร้าง Bare Repo จำลอง Remote แล้ว Clone มาทำงานจริง

---

## Step 81: Remote Repository คืออะไร ความสัมพันธ์กับ Local Repo

จนถึงตอนนี้ในหลักสูตร เราทำงานกับ Git อยู่แค่ **บนเครื่องเดียว (local)** เท่านั้น — สร้าง repo ด้วย `git init`, แก้ไฟล์, `add`, `commit`, สร้าง branch, merge กันเอง ทุกอย่างเกิดขึ้นภายในโฟลเดอร์ `.git` บนเครื่องของเราเอง ไม่มีใครอื่นเห็นงานของเราเลย

แต่ในโลกจริง แทบไม่มีใครทำงานคนเดียวตลอดไป เราต้อง:

- แชร์โค้ดให้เพื่อนร่วมทีมเห็น
- สำรอง (backup) โค้ดไว้ในที่ที่ปลอดภัยกว่าเครื่องตัวเองเครื่องเดียว
- ดึงงานล่าสุดของคนอื่นมารวมกับงานของเรา
- เผยแพร่โปรเจกต์ให้คนอื่นดาวน์โหลดไปใช้

นี่คือจุดที่ **Remote Repository** เข้ามามีบทบาท

### นิยามของ Remote Repository

> **Remote Repository** คือ Git repository อีกชุดหนึ่ง ที่ไม่ได้อยู่ในเครื่อง local ของเรา แต่อยู่ในที่อื่น — อาจเป็นเซิร์ฟเวอร์บนอินเทอร์เน็ต (เช่น GitHub, GitLab, Bitbucket), เซิร์ฟเวอร์ภายในองค์กร, หรือแม้แต่**โฟลเดอร์อื่นบนเครื่องเดียวกัน**ก็นับเป็น remote ได้เช่นกัน

สิ่งสำคัญที่ต้องเข้าใจให้แม่นตั้งแต่ต้นคือ **remote repository ก็คือ Git repository ธรรมดาชุดหนึ่ง** ไม่ได้มีอะไรวิเศษไปกว่า local repo ของเราเลย — มันมี object database, มี commit, มี branch, มี tag ครบเหมือนกันทุกประการ เพียงแต่มันตั้งอยู่ **"ที่อื่น"** เมื่อเทียบกับเครื่องที่เรากำลังใช้งานอยู่

เพราะ Git เป็น **Distributed VCS** (ตามที่เราเรียนใน Part 01) ทุก repository ไม่ว่าจะเป็น local หรือ remote จึงมีสถานะเท่าเทียมกันในทางเทคนิค — คำว่า "remote" หรือ "local" เป็นเพียง**มุมมองที่สัมพันธ์กัน (relative)** เท่านั้น ไม่ใช่สถานะที่ตายตัว

ยกตัวอย่างเช่น:

- จากมุมมองของเครื่องคุณ: repo บน GitHub คือ **remote**
- จากมุมมองของเซิร์ฟเวอร์ GitHub: repo บนเครื่องคุณคือสิ่งที่มันไม่รู้จักด้วยซ้ำ (เพราะ GitHub ไม่ได้ "เชื่อมต่อ" กลับมาหาเครื่องคุณ)
- ถ้าเพื่อนคุณ clone repo จากเครื่องคุณโดยตรง (ผ่านเครือข่ายภายใน) เครื่องของคุณก็จะกลายเป็น "remote" ในมุมมองของเพื่อนคุณ

### ความสัมพันธ์ระหว่าง Local และ Remote

Local repo กับ Remote repo **ไม่ได้ผูกติดกันแบบ real-time** เหมือนไฟล์ที่ sync กันอัตโนมัติแบบ Google Drive หรือ Dropbox — Git ทำงานแบบ **"synchronize เมื่อสั่งเท่านั้น"** คุณต้องสั่งคำสั่งอย่างชัดเจนเพื่อแลกเปลี่ยนข้อมูลระหว่างสองฝั่ง:

| ทิศทาง | คำสั่งที่ใช้ | ความหมาย |
|---|---|---|
| Remote → Local | `git clone` | คัดลอก repo ทั้งหมดจาก remote มาสร้างเป็น local repo ใหม่ |
| Remote → Local | `git fetch` | ดึงข้อมูล (commit, branch ใหม่) จาก remote มาเก็บไว้ใน local โดยยังไม่ merge |
| Remote → Local | `git pull` | ดึงข้อมูลจาก remote แล้ว merge (หรือ rebase) เข้ากับ branch ปัจจุบันทันที |
| Local → Remote | `git push` | ส่ง commit ที่มีใน local ขึ้นไปเก็บไว้บน remote |

ไดอะแกรมด้านล่างสรุปภาพรวมความสัมพันธ์นี้:

```
                     ┌─────────────────────────┐
                     │   Remote Repository      │
                     │   (เช่น GitHub, GitLab,   │
                     │    หรือเซิร์ฟเวอร์บริษัท)  │
                     │                           │
                     │   .git/  (เก็บ object     │
                     │    ทั้งหมด, refs ทั้งหมด)  │
                     └────────────┬──────────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │ clone / fetch      │        push        │
              │ (ดาวน์โหลด)         │      (อัปโหลด)      │
              ▼                    │                    ▼
   ┌─────────────────────┐         │        ┌─────────────────────┐
   │  Local Repository A  │◀────────┴───────▶│  Local Repository B  │
   │  (เครื่องของคุณ)      │   ไม่เชื่อมกันตรง  │  (เครื่องเพื่อนร่วมทีม) │
   │                       │   ต้องผ่าน remote │                       │
   └─────────────────────┘                    └─────────────────────┘
```

สังเกตว่า **Local Repository A และ B ไม่เคยคุยกันโดยตรง** — ทุกการแลกเปลี่ยนข้อมูลต้อง "ผ่าน" remote repository เสมอ (ในกรณีทั่วไปที่ใช้ workflow แบบ centralized ผ่าน GitHub/GitLab) นี่คือเหตุผลที่ remote repository มักถูกเรียกว่าเป็น **"จุดนัดพบ" (point of coordination)** ของทีม ไม่ใช่ "เจ้าของข้อมูลตัวจริงเพียงหนึ่งเดียว" แบบ Centralized VCS รุ่นเก่า — เพราะทุกเครื่อง (รวมถึง remote เอง) ต่างก็มีสำเนาประวัติแบบเต็มเก็บไว้ทั้งหมด

### ทำไมต้องมี Remote Repository ทั้ง ๆ ที่ Git เป็น Distributed

คำถามที่มักเกิดขึ้นคือ ถ้า Git ถูกออกแบบมาให้ไม่ต้องพึ่ง server กลาง แล้วทำไมแทบทุกทีมถึงยังใช้ GitHub/GitLab เป็น "จุดศูนย์กลาง" อยู่ดี คำตอบคือ:

1. **ความสะดวกในการประสานงาน (Coordination)** — ถ้าไม่มีจุดกลาง ทุกคนต้อง fetch/push จากกันและกันโดยตรง (peer-to-peer) ซึ่งซับซ้อนกว่ามากเมื่อทีมมีสมาชิกหลายคน
2. **Availability** — เซิร์ฟเวอร์อย่าง GitHub เปิดให้เข้าถึงได้ตลอด 24 ชั่วโมง ต่างจากเครื่อง laptop ของเพื่อนร่วมทีมที่อาจปิดเครื่องอยู่
3. **ฟีเจอร์เสริม** — Pull Request, Code Review, CI/CD, Issue Tracking ล้วนต้องอาศัย platform ที่มี server กลางรองรับ
4. **Backup ตามธรรมชาติ** — การ push ขึ้น remote คือการสำรองข้อมูลไปในตัวโดยอัตโนมัติ

ดังนั้นในทางปฏิบัติ ทีมส่วนใหญ่จึงเลือกใช้ **Centralized Workflow บน DVCS** คือใช้ Git ที่เป็น distributed โดยธรรมชาติ แต่ตกลงกันว่าจะมี remote repository หนึ่งชุด (หรือมากกว่า) เป็น "จุดอ้างอิงหลัก" ของทีม — นี่คือสิ่งที่เราจะฝึกฝนตลอด Part นี้และ Part ถัดไป

---

## Step 82: `git clone` — คัดลอก Repo ทั้งหมดพร้อมประวัติ

### `git clone` คืออะไร

`git clone` คือคำสั่งที่ใช้ **คัดลอก Git repository ทั้งชุด** จากที่หนึ่ง (source — มักเป็น remote) มาสร้างเป็น local repository ใหม่บนเครื่องของเรา

ไวยากรณ์พื้นฐาน:

```bash
git clone <url-หรือ-path-ของ-repo>
```

ตัวอย่าง:

```bash
git clone https://github.com/example-org/example-project.git
```

หลังจากรันคำสั่งนี้ Git จะสร้างโฟลเดอร์ใหม่ชื่อ `example-project` (ตัดจากชื่อ repo ในท้าย URL โดยอัตโนมัติ) ขึ้นในตำแหน่งปัจจุบัน แล้วดาวน์โหลดข้อมูลทั้งหมดเข้าไปในนั้น

ถ้าต้องการตั้งชื่อโฟลเดอร์เอง สามารถระบุชื่อโฟลเดอร์เป็นพารามิเตอร์ที่สองได้:

```bash
git clone https://github.com/example-org/example-project.git my-folder-name
```

### สิ่งที่เกิดขึ้น "เบื้องหลัง" เมื่อ Clone

หลายคนมองว่า `git clone` เป็นแค่การ "ดาวน์โหลดไฟล์" แต่จริง ๆ แล้วมันทำงานซับซ้อนกว่านั้นมาก และเข้าใจกลไกเบื้องหลังนี้จะช่วยให้เข้าใจ Step ถัดไปทั้งหมดของ Part นี้ได้ง่ายขึ้นมาก

เมื่อคุณรัน `git clone <url>` Git จะทำสิ่งต่อไปนี้ตามลำดับ:

1. **สร้างโฟลเดอร์ใหม่** ในตำแหน่งปัจจุบัน (ตั้งชื่อตาม repo หรือชื่อที่คุณระบุ)
2. **สร้างโฟลเดอร์ `.git`** ภายในนั้น และเริ่มต้น repository เปล่าเหมือนกับตอนใช้ `git init`
3. **เชื่อมต่อไปยัง remote ที่ระบุ** และ**ดาวน์โหลด Git object ทั้งหมด** — commit ทุกตัว, tree ทุกอัน, blob (เนื้อหาไฟล์) ทุกไฟล์ในทุกเวอร์ชันที่เคย commit มา รวมถึง tag ทั้งหมด — พูดง่าย ๆ คือ **ประวัติทั้งหมดของทุก branch** ไม่ใช่แค่ branch ที่กำลังจะเปิดใช้งาน
4. **ตั้งชื่อ remote นี้โดยอัตโนมัติว่า `origin`** — นี่คือชื่อมาตรฐานที่ Git ใช้เรียก "แหล่งที่มา" ของ repo ที่ clone มา (สามารถเปลี่ยนชื่อภายหลังได้ ซึ่งเราจะเรียนใน Step 84)
5. **สร้าง Remote-tracking Branch** สำหรับทุก branch ที่มีอยู่บน remote เช่น `origin/main`, `origin/develop`, `origin/feature-x` (เราจะเจาะลึกเรื่องนี้ใน Step 86)
6. **ตรวจสอบว่า branch ใดเป็น default branch ของ remote** (ปกติคือ `main` หรือ `master` แล้วแต่การตั้งค่าของ repo นั้น) แล้ว **สร้าง local branch ชื่อเดียวกัน** ขึ้นมาโดยอัตโนมัติ พร้อมตั้งค่าให้ local branch นี้ **ติดตาม (track)** remote-tracking branch ที่สอดคล้องกัน
7. **Checkout** local branch นั้นออกมาเป็น Working Directory ให้คุณเริ่มทำงานได้ทันที

สรุปเป็นภาพรวม:

```
git clone https://github.com/org/project.git
        │
        ▼
┌───────────────────────────────────────────────┐
│  project/                                       │
│  ├── .git/                                      │
│  │   ├── objects/         ← commit, tree, blob   │
│  │   │                       ทั้งหมดถูกดาวน์โหลด   │
│  │   ├── refs/heads/main  ← local branch "main"  │
│  │   ├── refs/remotes/                          │
│  │   │   └── origin/                            │
│  │   │       ├── main     ← remote-tracking      │
│  │   │       └── develop  ← branch ทั้งหมด        │
│  │   └── config           ← ตั้งค่า remote "origin"│
│  │                            ชี้กลับไปยัง URL ต้นทาง│
│  └── (ไฟล์ของโปรเจกต์ ตาม branch main ที่ checkout)│
└───────────────────────────────────────────────┘
```

ลองตรวจสอบผลลัพธ์เหล่านี้ด้วยตัวเองหลัง clone:

```bash
cd project
cat .git/config
```

จะเห็นเนื้อหาประมาณนี้:

```ini
[core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
[remote "origin"]
    url = https://github.com/org/project.git
    fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
    remote = origin
    merge = refs/heads/main
```

สังเกต 3 ส่วนสำคัญที่ Git เขียนให้อัตโนมัติ:

- `[remote "origin"]` — บันทึกว่า remote ชื่อ `origin` ชี้ไปที่ URL ไหน และมีกฎการ fetch อย่างไร (`fetch = +refs/heads/*:refs/remotes/origin/*` หมายถึง "เอา branch ทุกอันจาก remote มาเก็บไว้ในรูป `origin/<ชื่อ branch>`")
- `[branch "main"]` — บันทึกว่า local branch `main` **ติดตาม (tracks)** อะไรอยู่ คือ remote `origin` และ branch `main` บนนั้น

นี่คือเหตุผลที่หลัง clone เสร็จ คุณสามารถพิมพ์ `git pull` หรือ `git push` เฉย ๆ โดยไม่ต้องระบุชื่อ remote หรือ branch เลยก็ได้ทันที — เพราะ Git รู้อยู่แล้วว่า `main` ของคุณผูกกับ `origin/main`

### รูปแบบ URL ที่ใช้ Clone ได้

Git รองรับ protocol หลายแบบสำหรับ clone:

| Protocol | รูปแบบ URL | ลักษณะการใช้งาน |
|---|---|---|
| **HTTPS** | `https://github.com/org/project.git` | ใช้งานง่ายที่สุด ต้องใส่ username/token เวลา push (หรือใช้ credential helper เก็บไว้) |
| **SSH** | `git@github.com:org/project.git` | ต้องตั้งค่า SSH key ล่วงหน้า แต่สะดวกกว่าระยะยาวเพราะไม่ต้อง auth ซ้ำทุกครั้ง |
| **Local path** | `/home/user/repos/project.git` หรือ `../project.git` | ใช้ clone จากโฟลเดอร์อื่นบนเครื่องเดียวกัน (จะใช้ฝึกใน Step 90) |
| **Git protocol** | `git://...` | โปรโตคอลเก่า ไม่มีการเข้ารหัส ปัจจุบันแทบไม่ใช้แล้วเพราะไม่ปลอดภัย |

เราจะเจาะลึกเรื่อง HTTPS กับ SSH authentication อย่างละเอียดใน Part ที่ว่าด้วย GitHub โดยเฉพาะ ตอนนี้ขอให้เข้าใจแค่ว่า clone รับได้ทั้ง URL จากอินเทอร์เน็ตและ path บนเครื่อง

### ตัวเลือกเสริมที่ควรรู้จักไว้ (ยังไม่ต้องใช้ตอนนี้)

```bash
# clone แค่ branch ที่ระบุ (ไม่ดึง branch อื่นมาด้วย)
git clone --branch develop --single-branch <url>

# shallow clone — ดึงมาแค่ N commit ล่าสุด ไม่เอาประวัติทั้งหมด (ใช้เมื่อ repo ใหญ่มากและไม่สนใจประวัติเก่า)
git clone --depth 1 <url>
```

ตัวเลือกเหล่านี้มีประโยชน์มากในกรณี repo ขนาดใหญ่มาก ๆ เราจะกลับมาพูดถึงอย่างละเอียดใน Part ที่ว่าด้วย Performance และ Git ขั้นสูง สำหรับตอนนี้ให้ใช้ `git clone <url>` แบบพื้นฐานไปก่อน เพราะเราต้องการประวัติทั้งหมดเพื่อฝึกฝนคำสั่งต่าง ๆ ใน Part นี้

---

## Step 83: `git remote -v` และการมี Remote หลายตัว (origin, upstream)

### `git remote` — ดูรายชื่อ Remote ทั้งหมด

หลัง clone repo มาแล้ว คุณสามารถตรวจสอบว่า repo ของคุณผูกกับ remote ตัวไหนอยู่บ้างด้วยคำสั่ง:

```bash
git remote
```

ผลลัพธ์ (ปกติหลัง clone จะมีแค่ตัวเดียว):

```
origin
```

คำสั่งนี้แสดงแค่**ชื่อ** ของ remote เท่านั้น ถ้าอยากเห็น URL ที่แต่ละ remote ชี้ไปด้วย ให้เพิ่ม flag `-v` (verbose):

```bash
git remote -v
```

ผลลัพธ์:

```
origin  https://github.com/org/project.git (fetch)
origin  https://github.com/org/project.git (push)
```

สังเกตว่า remote แต่ละตัวมี URL แสดงสองบรรทัด คือ **(fetch)** และ **(push)** — โดยปกติสอง URL นี้จะเหมือนกัน แต่ Git อนุญาตให้ตั้งค่าให้ต่างกันได้ (เช่น fetch จาก URL หนึ่ง แต่ push ไปอีก URL หนึ่ง) ซึ่งเป็นกรณีพิเศษที่ใช้น้อยมากในทางปฏิบัติ

### ทำไมต้องมี Remote มากกว่า 1 ตัว

ในหลาย ๆ สถานการณ์ โดยเฉพาะการทำงานกับ **Open Source** หรือโปรเจกต์ที่ใช้ **Fork Workflow** repository เดียวอาจต้องเชื่อมกับ remote มากกว่าหนึ่งแหล่ง ชื่อที่นิยมใช้กันตามธรรมเนียม (convention) ในวงการคือ:

| ชื่อ Remote | ความหมายตามธรรมเนียม |
|---|---|
| **origin** | repo ที่คุณ clone มาโดยตรง (โดยทั่วไปคือ fork ของคุณเอง หรือ repo หลักที่คุณมีสิทธิ์ push) |
| **upstream** | repo ต้นฉบับดั้งเดิม (ที่คุณ fork มาจากมัน) ใช้สำหรับดึงความเปลี่ยนแปลงล่าสุดจากเจ้าของโปรเจกต์จริง |

### สถานการณ์ตัวอย่าง: การ Contribute ให้ Open Source

ลองจินตนาการสถานการณ์นี้ ซึ่งเป็นสถานการณ์ที่พบบ่อยที่สุดของการมี remote สองตัว:

1. คุณเจอโปรเจกต์ Open Source ที่น่าสนใจบน GitHub ชื่อ `original-owner/cool-project`
2. คุณกด **Fork** บน GitHub เพื่อสร้างสำเนาของ repo นั้นไว้ในบัญชีตัวเอง กลายเป็น `your-username/cool-project`
3. คุณ `git clone` จาก fork ของตัวเอง (ไม่ใช่ต้นฉบับ) — ทำให้ `origin` ของคุณชี้ไปที่ fork ของคุณเอง
4. แต่คุณยังอยากรู้ความเคลื่อนไหวล่าสุดของ repo ต้นฉบับ (เผื่อมีคนอื่น merge งานใหม่เข้าไปเรื่อย ๆ) จึงเพิ่ม remote อีกตัวชื่อ `upstream` ที่ชี้ไปยัง repo ต้นฉบับโดยตรง (วิธีเพิ่มจะเรียนใน Step 84)

ไดอะแกรมแสดงความสัมพันธ์:

```
┌─────────────────────────────┐
│  original-owner/cool-project │   ← upstream (ปกติแค่ fetch เพื่อดูความเปลี่ยนแปลง
│  (repo ต้นฉบับบน GitHub)      │      ไม่มีสิทธิ์ push ตรงเข้าไป)
└──────────────┬────────────────┘
               │ fork (ทำผ่านปุ่มบน GitHub)
               ▼
┌─────────────────────────────┐
│  your-username/cool-project  │   ← origin (fork ของคุณ คุณมีสิทธิ์ push เต็มที่)
│  (fork บน GitHub)             │
└──────────────┬────────────────┘
               │ git clone
               ▼
┌─────────────────────────────┐
│      Local Repository         │
│      (เครื่องของคุณ)          │
│                                │
│  remote "origin"   → fork ของคุณ│
│  remote "upstream" → ต้นฉบับ    │
└─────────────────────────────┘
```

ประโยชน์ของการตั้งค่าแบบนี้:

- คุณ `git fetch upstream` เพื่อดูว่ามี commit ใหม่ ๆ ในโปรเจกต์ต้นฉบับหรือไม่ โดยไม่กระทบ fork ของคุณ
- คุณ merge หรือ rebase งานล่าสุดจาก `upstream/main` เข้ามาที่ branch ของคุณ เพื่อให้โค้ดของคุณไม่ตกยุค
- คุณยัง `git push origin <branch>` เพื่อส่งงานขึ้น fork ของตัวเองตามปกติ แล้วค่อยเปิด Pull Request จาก fork ของคุณไปยัง repo ต้นฉบับผ่านหน้าเว็บ GitHub

รูปแบบการทำงานนี้เรียกว่า **Fork & Pull Request Workflow** ซึ่งเป็นวิธีมาตรฐานที่โปรเจกต์ Open Source เกือบทั้งหมดใช้ในการรับ contribution จากคนภายนอก เราจะฝึกฝน workflow นี้แบบเต็มรูปแบบใน Part ที่ว่าด้วย GitHub และ Open Source โดยเฉพาะ

> **หมายเหตุสำคัญ:** ชื่อ `origin` และ `upstream` เป็นเพียง**ธรรมเนียมปฏิบัติ (convention)** ที่คนส่วนใหญ่ใช้กัน ไม่ใช่คำสงวนของ Git คุณสามารถตั้งชื่อ remote เป็นอะไรก็ได้ตามใจชอบ เช่น `myfork`, `company-server`, `backup` — Git ไม่ได้บังคับ แต่การใช้ตามธรรมเนียมจะช่วยให้ทีมอื่นเข้าใจโค้ดของคุณได้ง่ายขึ้นมาก

---

## Step 84: `git remote add` / `remove` / `rename`

เมื่อรู้จักการดูรายชื่อ remote แล้ว มาเรียนรู้วิธี**จัดการ**มันบ้าง — เพิ่ม ลบ และเปลี่ยนชื่อ

### เพิ่ม Remote ใหม่ด้วย `git remote add`

ไวยากรณ์:

```bash
git remote add <ชื่อ> <url>
```

ตัวอย่าง (ต่อจากสถานการณ์ Fork ใน Step 83):

```bash
git remote add upstream https://github.com/original-owner/cool-project.git
```

หลังรันคำสั่งนี้ ลอง `git remote -v` ดูอีกครั้ง จะเห็นสอง remote พร้อมกัน:

```
origin    https://github.com/your-username/cool-project.git (fetch)
origin    https://github.com/your-username/cool-project.git (push)
upstream  https://github.com/original-owner/cool-project.git (fetch)
upstream  https://github.com/original-owner/cool-project.git (push)
```

คำสั่ง `git remote add` เพียงแค่**บันทึกข้อมูล** ลงใน `.git/config` เท่านั้น — มันยัง **ไม่ได้ดาวน์โหลดอะไรเลย** ต้องตามด้วย `git fetch <ชื่อ>` เพื่อดึงข้อมูลจริง ๆ จาก remote ตัวนั้นเข้ามา (จะเรียนละเอียดใน Step 85):

```bash
git fetch upstream
```

### ลบ Remote ด้วย `git remote remove` (หรือ `git remote rm`)

ถ้าไม่ต้องการอ้างอิงถึง remote ตัวใดตัวหนึ่งอีกต่อไป:

```bash
git remote remove upstream
```

หรือใช้คำสั่งย่อ:

```bash
git remote rm upstream
```

ทั้งสองรูปแบบทำงานเหมือนกันทุกประการ `rm` เป็นแค่ alias ของ `remove`

**ข้อควรระวัง:** การลบ remote จะลบแค่**การอ้างอิง** (reference) ในเครื่อง local ของคุณเท่านั้น ไม่ได้ลบ repo จริงบนเซิร์ฟเวอร์แต่อย่างใด แต่จะมีผลข้างเคียงที่ควรรู้ไว้คือ **remote-tracking branch ทั้งหมดที่เกี่ยวข้องกับ remote นั้นจะถูกลบไปด้วย** (เช่น `upstream/main`, `upstream/develop` ทั้งหมดจะหายไปจาก `.git/refs/remotes/upstream/`)

### เปลี่ยนชื่อ Remote ด้วย `git remote rename`

ถ้าอยากเปลี่ยนชื่อ remote โดยไม่เปลี่ยน URL:

```bash
git remote rename origin old-origin
```

คำสั่งนี้จะเปลี่ยนชื่อ `origin` เป็น `old-origin` และ Git จะจัดการปรับ remote-tracking branch ทั้งหมดให้อัตโนมัติ (เช่น `origin/main` จะกลายเป็น `old-origin/main`) รวมถึงปรับการตั้งค่า tracking ของ local branch ที่ผูกกับ remote นั้นให้ถูกต้องด้วย

### คำสั่งเสริมที่มีประโยชน์: `git remote show` และ `git remote set-url`

**`git remote show <ชื่อ>`** แสดงรายละเอียดของ remote ตัวนั้นแบบเจาะลึก รวมถึง branch ทั้งหมดที่มีอยู่ และสถานะการ tracking:

```bash
git remote show origin
```

ตัวอย่างผลลัพธ์:

```
* remote origin
  Fetch URL: https://github.com/org/project.git
  Push  URL: https://github.com/org/project.git
  HEAD branch: main
  Remote branches:
    main     tracked
    develop  tracked
  Local branch configured for 'git pull':
    main merges with remote main
  Local ref configured for 'git push':
    main pushes to main (up to date)
```

**`git remote set-url`** ใช้เปลี่ยน URL ของ remote ที่มีอยู่แล้ว โดยไม่ต้องลบแล้วเพิ่มใหม่ (มีประโยชน์มากเมื่อเปลี่ยนจาก HTTPS เป็น SSH หรือย้าย repo ไปโฮสต์ที่อื่น):

```bash
git remote set-url origin git@github.com:org/project.git
```

### ตารางสรุปคำสั่งจัดการ Remote

| คำสั่ง | หน้าที่ |
|---|---|
| `git remote` | แสดงชื่อ remote ทั้งหมด |
| `git remote -v` | แสดงชื่อ remote พร้อม URL (fetch/push) |
| `git remote add <name> <url>` | เพิ่ม remote ใหม่ |
| `git remote remove <name>` / `git remote rm <name>` | ลบ remote (และ remote-tracking branch ที่เกี่ยวข้อง) |
| `git remote rename <old> <new>` | เปลี่ยนชื่อ remote |
| `git remote show <name>` | ดูรายละเอียดเชิงลึกของ remote |
| `git remote set-url <name> <new-url>` | เปลี่ยน URL ของ remote ที่มีอยู่ |

---

## Step 85: `git fetch` คืออะไร ต่างจาก `pull` อย่างไร

นี่คือหนึ่งใน Step ที่สำคัญที่สุดของ Part นี้ เพราะความสับสนระหว่าง `fetch` กับ `pull` เป็นสิ่งที่มือใหม่เกือบทุกคนเจอ

### `git fetch` คืออะไร

```bash
git fetch <remote>
```

ตัวอย่าง:

```bash
git fetch origin
```

`git fetch` มีหน้าที่**เดียว**อย่างชัดเจน คือ:

> **ดาวน์โหลด commit, branch, tag ใหม่ ๆ ที่มีอยู่บน remote แต่ยังไม่มีใน local เข้ามาเก็บไว้ — แล้วอัปเดต remote-tracking branch (เช่น `origin/main`) ให้ตรงกับสถานะล่าสุดของ remote**

สิ่งสำคัญที่สุดที่ต้องจำให้ขึ้นใจคือ:

> **`git fetch` จะไม่แตะต้อง Working Directory และไม่แตะต้อง local branch ของคุณเลยแม้แต่นิดเดียว**

พูดอีกแบบหนึ่งคือ `fetch` เป็นการ **"ไปสอดแนมดูว่า remote มีอะไรใหม่บ้าง"** โดยไม่เปลี่ยนแปลงอะไรในสิ่งที่คุณกำลังทำงานอยู่เลย ไฟล์ในโฟลเดอร์ทำงานของคุณจะเหมือนเดิมทุกประการก่อนและหลัง fetch

### `git pull` คืออะไร

```bash
git pull <remote> <branch>
```

ตัวอย่าง:

```bash
git pull origin main
```

`git pull` แท้จริงแล้วคือการรวมสองคำสั่งเข้าด้วยกัน:

> **`git pull` = `git fetch` + `git merge` (หรือ `git rebase` ถ้าตั้งค่าไว้)**

พูดให้ชัดคือ เมื่อคุณสั่ง `git pull origin main` Git จะทำสิ่งต่อไปนี้เบื้องหลังโดยอัตโนมัติ:

```bash
git fetch origin              # ขั้นตอนที่ 1: ดึงข้อมูลใหม่มาเก็บใน origin/main
git merge origin/main          # ขั้นตอนที่ 2: รวม origin/main เข้ากับ branch ปัจจุบันทันที
```

(หรือถ้าตั้งค่า `pull.rebase = true` ไว้ ขั้นตอนที่ 2 จะกลายเป็น `git rebase origin/main` แทน ซึ่งเราจะเรียนรายละเอียดเรื่อง rebase ใน Part ที่ว่าด้วย Rebase โดยเฉพาะ)

### ตารางเปรียบเทียบโดยตรง

| คุณสมบัติ | `git fetch` | `git pull` |
|---|---|---|
| ดาวน์โหลดข้อมูลจาก remote | ทำ | ทำ (เพราะเรียก fetch อยู่ข้างใน) |
| อัปเดต remote-tracking branch (`origin/main`) | ทำ | ทำ |
| แก้ไข Working Directory | **ไม่ทำ** | **ทำ** (merge/rebase เข้า working directory ทันที) |
| แก้ไข local branch ปัจจุบัน | **ไม่ทำ** | **ทำ** |
| มีโอกาสเกิด merge conflict ทันที | **ไม่มี** | **มี** ถ้ามีการแก้ไขที่ชนกัน |
| ระดับความปลอดภัย | ปลอดภัยกว่า — ดูก่อนตัดสินใจได้ | เสี่ยงกว่า — เปลี่ยนแปลงทันทีโดยไม่ทันได้ตรวจสอบ |

### ทำไม `git fetch` ถึง "ปลอดภัยกว่า"

เหตุผลที่มือโปรจำนวนมากแนะนำให้ใช้ `git fetch` แล้วตรวจสอบก่อน แทนที่จะใช้ `git pull` ทันที มีดังนี้:

1. **คุณได้เห็นก่อนว่ามีอะไรเปลี่ยนแปลงบ้าง** ก่อนที่จะให้มันมารวมกับงานของคุณ (จะเรียนวิธีดูใน Step 87)
2. **หลีกเลี่ยง merge commit หรือ conflict ที่ไม่คาดคิด** กลางที่ทำงานอยู่ — ถ้า pull แล้วเกิด conflict ทันที อาจทำให้คุณเสียจังหวะการทำงาน
3. **ควบคุมได้ว่าจะ merge หรือ rebase** — บางทีมต้องการ history ที่เรียบร้อยแบบไม่มี merge commit เยอะเกินไป การ fetch ก่อนแล้วเลือกวิธี integrate เองจึงยืดหยุ่นกว่า
4. **ปลอดภัยสำหรับการทำงานที่ยังไม่ commit เสร็จ** — ถ้ามีการเปลี่ยนแปลงค้างอยู่ใน working directory ที่ยังไม่ commit, `git pull` อาจไปชนกับไฟล์เหล่านั้นและทำให้เกิดปัญหาที่ซับซ้อนขึ้นได้

Workflow ที่แนะนำสำหรับมือใหม่ (และมือโปรจำนวนมากก็ยังใช้) คือ:

```bash
git fetch origin                    # ดึงข้อมูลมาดูก่อน ไม่กระทบอะไร
git log main..origin/main           # ดูว่ามี commit ใหม่อะไรบ้างที่ยังไม่มีใน local
git diff main origin/main           # ดูว่าเนื้อหาไฟล์เปลี่ยนไปตรงไหนบ้าง
git merge origin/main               # ถ้าตรวจสอบแล้วโอเค ค่อย merge เข้ามา
```

เราจะฝึกคำสั่ง `git log main..origin/main` และ `git diff main origin/main` แบบละเอียดใน Step 87

> **หมายเหตุ:** `git pull` ไม่ใช่คำสั่งที่แย่ — มันสะดวกมากในสถานการณ์ที่คุณมั่นใจอยู่แล้วว่าไม่มีอะไรขัดแย้งกัน (เช่น ทำงานคนเดียว หรือเพิ่งเริ่มวันใหม่และรู้ว่าไม่มีใครแก้ไฟล์เดียวกับคุณ) ประเด็นคือให้ **เข้าใจว่ามันทำอะไรอยู่เบื้องหลัง** เพื่อจะได้เลือกใช้อย่างเหมาะสมกับสถานการณ์

---

## Step 86: Remote-tracking Branch คืออะไร (เช่น `origin/main`)

### นิยาม

**Remote-tracking Branch** คือ **"ตัวชี้ (pointer) ในเครื่อง local ของคุณ ที่จดจำว่า branch บน remote แต่ละอันมีสถานะล่าสุดเป็นอย่างไร ณ ครั้งสุดท้ายที่คุณ fetch/pull/push"**

รูปแบบชื่อของมันคือ `<ชื่อ remote>/<ชื่อ branch>` เช่น:

- `origin/main`
- `origin/develop`
- `upstream/main`
- `origin/feature-login`

### จุดที่มือใหม่สับสนบ่อยที่สุด

หลายคนเข้าใจผิดว่า `origin/main` คือ "branch main ที่อยู่บนเซิร์ฟเวอร์จริง ๆ" แต่ความจริงแล้ว **ไม่ใช่**

> **`origin/main` คือสำเนาข้อมูล ณ ตอนที่คุณ fetch ล่าสุดเท่านั้น มันอยู่ในเครื่อง local ของคุณ 100% ไม่ได้เชื่อมสายตรงกับเซิร์ฟเวอร์แบบ real-time**

พูดง่าย ๆ คือ `origin/main` เปรียบเสมือน **"ภาพถ่ายความจำล่าสุด"** ที่ local repo ของคุณจำไว้ว่า "ครั้งล่าสุดที่ฉันคุยกับ origin, branch main ของมันอยู่ที่ commit ไหน" — ถ้าหลังจากนั้นมีคนอื่น push commit ใหม่ขึ้น remote โดยที่คุณยังไม่ได้ fetch ใหม่ `origin/main` ในเครื่องคุณก็จะ**ยังคงชี้ไปที่ commit เก่า** ไม่รู้เรื่องอะไรเลยจนกว่าคุณจะสั่ง fetch อีกครั้ง

### ตำแหน่งจัดเก็บใน `.git`

Remote-tracking branch ถูกเก็บไว้ที่:

```
.git/refs/remotes/<remote>/<branch>
```

ตัวอย่างเช่น ถ้ามี remote ชื่อ `origin` และมี branch `main`, `develop` บนนั้น หลัง fetch จะพบไฟล์:

```
.git/refs/remotes/origin/main
.git/refs/remotes/origin/develop
```

เปรียบเทียบกับ local branch ที่เก็บอยู่ที่:

```
.git/refs/heads/main
.git/refs/heads/develop
```

จะเห็นว่าโครงสร้างคล้ายกันมาก ต่างกันแค่ `refs/remotes/<remote-name>/` กับ `refs/heads/` เท่านั้น — เพราะจริง ๆ แล้วทั้งคู่คือ**ตัวชี้ไปยัง commit** เหมือนกันทุกประการในทางเทคนิค

### ข้อแตกต่างสำคัญ: Local Branch แก้ไขได้ตรง ๆ แต่ Remote-tracking Branch ไม่ควร

- **Local branch** (เช่น `main`) คุณสามารถ `checkout` ไปทำงาน, commit ใหม่, ทำให้มันขยับไปข้างหน้าได้โดยตรงด้วยคำสั่งต่าง ๆ ที่เราเรียนมาก่อนหน้านี้
- **Remote-tracking branch** (เช่น `origin/main`) เป็น **read-only ในทางปฏิบัติ** — Git ไม่อนุญาตให้คุณ `checkout` แล้วแก้ไขมันตรง ๆ (ถ้าคุณลอง `git checkout origin/main` มันจะเข้าสู่สถานะ **detached HEAD** ซึ่งไม่ใช่การทำงานปกติ) มันจะถูกอัปเดตโดยอัตโนมัติก็ต่อเมื่อคุณสั่ง `fetch`, `pull`, หรือ `push` เท่านั้น

ไดอะแกรมสรุปความสัมพันธ์:

```
                    Remote Server (GitHub)
                    ┌─────────────────┐
                    │  branch: main     │──── commit C5 (ล่าสุดบน server)
                    └────────┬─────────┘
                             │
                             │ git fetch origin
                             ▼
        Local Repository (.git/refs/remotes/origin/main)
        ┌───────────────────────────────┐
        │  origin/main  ──── ชี้ไปที่ C5   │  ← อัปเดตเฉพาะตอน fetch/pull/push
        └───────────────────────────────┘

        Local Repository (.git/refs/heads/main)
        ┌───────────────────────────────┐
        │  main  ──── ชี้ไปที่ C3 (เก่ากว่า) │  ← คุณยังไม่ได้ merge เข้ามา
        └───────────────────────────────┘
```

จากไดอะแกรมนี้ จะเห็นว่าหลัง `fetch` เสร็จ local branch `main` ของคุณ **ยังคงอยู่ที่ commit C3 เหมือนเดิม** ในขณะที่ `origin/main` ขยับไปที่ C5 แล้ว — นี่คือสถานการณ์ที่เรียกว่า **"local อยู่หลัง remote (behind)"** ซึ่งเราจะเรียนวิธีตรวจสอบและจัดการต่อใน Step ถัดไป

### การดู Remote-tracking Branch ทั้งหมด

```bash
git branch -r
```

`-r` ย่อมาจาก remote จะแสดงเฉพาะ remote-tracking branch ทั้งหมดที่มีอยู่ในเครื่อง เช่น:

```
  origin/HEAD -> origin/main
  origin/main
  origin/develop
```

ถ้าอยากเห็นทั้ง local branch และ remote-tracking branch พร้อมกัน:

```bash
git branch -a
```

`-a` ย่อมาจาก all จะแสดงทั้งสองประเภทรวมกัน

---

## Step 87: เปรียบเทียบ Local Branch กับ Remote-tracking Branch หลัง Fetch

เมื่อเข้าใจแล้วว่า `origin/main` เป็นแค่ "ภาพจำล่าสุด" ที่แยกออกจาก local branch `main` ของคุณ ขั้นตอนต่อไปคือการเรียนรู้วิธี**เปรียบเทียบ**ทั้งสองฝั่ง เพื่อตัดสินใจว่าจะทำอะไรต่อ

### ใช้ `git diff` เปรียบเทียบเนื้อหาไฟล์

```bash
git diff main origin/main
```

คำสั่งนี้จะแสดง **ผลต่างของเนื้อหาไฟล์** ระหว่าง local branch `main` กับ remote-tracking branch `origin/main` — เหมือนกับการทำ `git diff` ระหว่าง branch สองอันปกติทุกประการ เพราะในทางเทคนิคทั้งคู่ก็คือ "ตัวชี้ไปยัง commit" เหมือนกัน ไม่มีอะไรพิเศษ

ตัวอย่างผลลัพธ์:

```diff
diff --git a/README.md b/README.md
index 8f3b1a2..d4c9e01 100644
--- a/README.md
+++ b/README.md
@@ -1,3 +1,5 @@
 # My Project

-This is the old description.
+This is the updated description.
+Added a new section about installation.
```

การอ่านผลลัพธ์นี้บอกคุณว่า **ถ้าคุณ merge `origin/main` เข้ามา** ไฟล์ `README.md` จะเปลี่ยนจากเนื้อหาฝั่งซ้าย (`main` ของคุณ) ไปเป็นเนื้อหาฝั่งขวา (`origin/main`) อย่างไร

### ใช้ `git log` เปรียบเทียบรายการ Commit

บางครั้งคุณอยากรู้แค่ว่า **"มี commit อะไรใหม่บ้าง"** โดยไม่ต้องดู diff เนื้อหาไฟล์ละเอียด สามารถใช้ไวยากรณ์ **double-dot (`..`)** ของ `git log` ได้:

```bash
git log main..origin/main
```

ไวยากรณ์ `A..B` หมายถึง **"แสดง commit ทั้งหมดที่มีอยู่ใน B แต่ไม่มีใน A"** ดังนั้น `main..origin/main` จึงหมายถึง **"commit ที่มีบน remote แล้ว แต่ local ของคุณยังไม่มี"** — พูดง่าย ๆ คือ **"งานใหม่ที่รอให้คุณดึงเข้ามา"**

กลับกัน ถ้าอยากรู้ว่า **local ของคุณมี commit อะไรที่ยังไม่ได้ push ขึ้นไปบ้าง** ให้สลับด้าน:

```bash
git log origin/main..main
```

นี่คือ **"งานของคุณที่ยังไม่ได้ส่งขึ้น remote"**

### ตารางสรุปทิศทางการเปรียบเทียบ

| คำสั่ง | ความหมาย |
|---|---|
| `git log main..origin/main` | commit ที่อยู่บน remote แต่ local ยังไม่มี (รอ merge/pull เข้ามา) |
| `git log origin/main..main` | commit ที่ local มีแต่ remote ยังไม่มี (รอ push ขึ้นไป) |
| `git diff main origin/main` | ผลต่างเนื้อหาไฟล์ จาก local ไปเป็น remote |
| `git diff origin/main main` | ผลต่างเนื้อหาไฟล์ จาก remote ไปเป็น local (กลับด้าน) |

### `git status` ก็บอกข้อมูลนี้ให้เช่นกัน

ถ้า local branch ของคุณตั้งค่า tracking กับ remote-tracking branch ไว้แล้ว (ซึ่งปกติจะเป็นแบบนั้นอัตโนมัติหลัง clone) หลังจาก `git fetch` ลองสั่ง:

```bash
git status
```

Git จะรายงานสถานะความสัมพันธ์ให้ทันที เช่น:

```
On branch main
Your branch is behind 'origin/main' by 2 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)
```

หรือถ้าคุณมี commit ที่ยังไม่ push:

```
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)
```

หรือในกรณีที่ทั้งสองฝั่งต่างมี commit ใหม่ที่อีกฝั่งไม่มี (เกิดจากการทำงานคู่ขนานกัน):

```
On branch main
Your branch and 'origin/main' have diverged,
and have 1 and 2 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)
```

สถานการณ์แบบนี้เรียกว่า **branch diverged (แยกทาง)** ซึ่งต้องใช้ merge หรือ rebase เพื่อรวมกลับเข้าด้วยกัน — เราจะเจาะลึกวิธีจัดการสถานการณ์นี้แบบเต็มรูปแบบใน Part ที่ว่าด้วย push/pull โดยตรง (Part 10) และ Part ที่ว่าด้วย Merge Conflict

---

## Step 88: `git ls-remote` — ดู Branch/Tag บน Remote โดยไม่ต้อง Clone

บางครั้งคุณอยากรู้ว่า remote repository หนึ่ง ๆ มี branch หรือ tag อะไรอยู่บ้าง **โดยที่ยังไม่ต้องการ clone หรือ fetch ทั้งชุดมาเก็บไว้ในเครื่อง** — อาจเพราะแค่อยากเช็คก่อนตัดสินใจ หรือ repo นั้นมีขนาดใหญ่มากและยังไม่อยากดาวน์โหลดตอนนี้

คำสั่งที่ใช้สำหรับกรณีนี้คือ `git ls-remote`

### ไวยากรณ์พื้นฐาน

```bash
git ls-remote <url-หรือ-ชื่อ-remote>
```

ตัวอย่าง:

```bash
git ls-remote https://github.com/git/git.git
```

ผลลัพธ์ (ตัวอย่างสมมติ ย่อให้สั้นลง):

```
a1b2c3d4e5f6...    HEAD
a1b2c3d4e5f6...    refs/heads/main
b2c3d4e5f6a1...    refs/heads/maint
c3d4e5f6a1b2...    refs/tags/v2.43.0
d4e5f6a1b2c3...    refs/tags/v2.44.0
```

แต่ละบรรทัดประกอบด้วย **SHA-1 hash ของ commit ที่ ref นั้นชี้ไปถึง** ตามด้วย **ชื่อของ ref นั้น** — คุณจะเห็นได้ทันทีว่า repo มี branch อะไรบ้าง มี tag (เวอร์ชัน) อะไรบ้าง โดยที่ **ไม่มีการดาวน์โหลด object ใด ๆ เข้ามาเลย** เป็นการคุยกับ server แบบเบามาก (lightweight) เพียงเพื่อขอรายการ ref เท่านั้น

### ตัวเลือกที่มีประโยชน์: กรองเฉพาะ Branch หรือ Tag

```bash
# ดูเฉพาะ branch (heads)
git ls-remote --heads https://github.com/git/git.git

# ดูเฉพาะ tag
git ls-remote --tags https://github.com/git/git.git
```

ตัวอย่างการดูเฉพาะ branch:

```bash
git ls-remote --heads https://github.com/git/git.git
```

ผลลัพธ์:

```
a1b2c3d4e5f6...    refs/heads/main
b2c3d4e5f6a1...    refs/heads/maint
e5f6a1b2c3d4...    refs/heads/next
```

### ใช้กับ Remote ที่ตั้งค่าไว้แล้วในเครื่องก็ได้

ถ้าอยู่ในโฟลเดอร์ repo ที่มี remote ตั้งค่าไว้แล้ว สามารถระบุแค่ชื่อ remote แทน URL เต็มได้:

```bash
git ls-remote origin
```

### กรณีการใช้งานจริง (Use Cases)

1. **ตรวจสอบก่อน clone** ว่า repo นี้มี branch ชื่อที่คุณต้องการหรือไม่ (เช่นก่อน `git clone --branch develop`) โดยไม่ต้องเสียเวลาดาวน์โหลดทั้งหมดก่อนแล้วค่อยพบว่าไม่มี branch นั้น
2. **ใช้ในสคริปต์อัตโนมัติ (CI/CD)** เพื่อเช็คว่ามี tag เวอร์ชันใหม่ถูกสร้างขึ้นหรือยัง โดยไม่ต้อง clone repo ทั้งชุดทุกครั้งที่เช็ค
3. **เปรียบเทียบ SHA ของ branch บน remote กับ commit ที่ deploy อยู่ปัจจุบัน** เพื่อตรวจสอบว่าเวอร์ชันที่รันอยู่ตรงกับล่าสุดหรือไม่
4. **ตรวจสอบว่า repo หรือ URL ที่ระบุนั้นมีอยู่จริงและเข้าถึงได้** ก่อนจะเริ่มกระบวนการอื่นที่ใช้เวลานานกว่า

`git ls-remote` จึงเป็นเครื่องมือที่เบา รวดเร็ว และมีประโยชน์มากในการ "สอดแนม" ข้อมูลพื้นฐานของ remote ก่อนตัดสินใจทำอะไรที่หนักกว่านั้น

---

## Step 89: การตั้งค่า Upstream Tracking Branch

### Tracking Branch คืออะไร (ทบทวนและขยายความ)

เรารู้จาก Step 82 แล้วว่าหลัง `git clone` เสร็จ local branch หลัก (เช่น `main`) จะถูกตั้งค่าให้ **ติดตาม (track)** remote-tracking branch ที่ตรงกันโดยอัตโนมัติ (เช่น `origin/main`) แต่ในหลายสถานการณ์ คุณจะต้องตั้งค่านี้**ด้วยตัวเอง** เช่น:

- เมื่อคุณสร้าง local branch ใหม่ด้วยตัวเอง (`git branch feature-x` หรือ `git checkout -b feature-x`) — branch ใหม่นี้**จะยังไม่มีการ tracking ใด ๆ** จนกว่าคุณจะตั้งค่าเอง
- เมื่อคุณต้องการเปลี่ยนให้ local branch ที่มีอยู่แล้วไป track remote-tracking branch อื่น

### ทำไม Tracking Branch ถึงสำคัญ

การตั้งค่า tracking (หรือเรียกเต็ม ๆ ว่า **upstream tracking reference**) ทำให้คุณได้รับประโยชน์เหล่านี้:

1. **พิมพ์ `git pull` หรือ `git push` เฉย ๆ ได้เลย** โดยไม่ต้องระบุชื่อ remote และ branch ทุกครั้ง
2. **`git status` บอกสถานะ ahead/behind ได้ถูกต้อง** เทียบกับ remote-tracking branch ที่ผูกไว้
3. **`git branch -vv` แสดงข้อมูล tracking ได้อย่างชัดเจน**

### วิธีที่ 1: ตั้งค่าตอน Push ครั้งแรกด้วย `-u`

วิธีที่นิยมที่สุดคือการใช้ flag `-u` (ย่อมาจาก `--set-upstream`) ตอน push branch ใหม่ขึ้น remote เป็นครั้งแรก:

```bash
git push -u origin feature-x
```

คำสั่งนี้ทำสองอย่างพร้อมกัน:

1. Push branch `feature-x` ขึ้นไปสร้างเป็น branch ใหม่บน remote `origin`
2. ตั้งค่าให้ local branch `feature-x` **track** `origin/feature-x` โดยอัตโนมัติ

หลังจากนี้ ครั้งต่อ ๆ ไปคุณสามารถพิมพ์แค่:

```bash
git push
git pull
```

โดยไม่ต้องระบุ `origin feature-x` อีกเลย เพราะ Git จำได้แล้วว่า branch นี้ผูกกับอะไร

### วิธีที่ 2: ตั้งค่าให้ Branch ที่มีอยู่แล้วด้วย `git branch --set-upstream-to`

ถ้า branch มีอยู่แล้วทั้งสองฝั่ง (local และ remote) แต่ยังไม่ได้ผูก tracking กันไว้ ใช้คำสั่ง:

```bash
git branch --set-upstream-to=origin/feature-x feature-x
```

หรือใช้รูปแบบย่อ `-u` กับคำสั่ง `git branch` (ต้องอยู่บน branch นั้นอยู่แล้ว):

```bash
git checkout feature-x
git branch -u origin/feature-x
```

ทั้งสองรูปแบบให้ผลลัพธ์เหมือนกัน คือแก้ไข `.git/config` ให้เพิ่มส่วน:

```ini
[branch "feature-x"]
    remote = origin
    merge = refs/heads/feature-x
```

### การตรวจสอบสถานะ Tracking ด้วย `git branch -vv`

คำสั่งที่มีประโยชน์มากในการตรวจสอบว่า local branch แต่ละอัน track อะไรอยู่บ้าง คือ:

```bash
git branch -vv
```

`-vv` คือ verbose สองชั้น ผลลัพธ์ตัวอย่าง:

```
* main         a1b2c3d [origin/main] Update README
  feature-x    e5f6a1b [origin/feature-x: ahead 2] Add login form
  experiment   c3d4e5f Local-only experiment branch
```

อ่านผลลัพธ์นี้ได้ดังนี้:

- **`main`** — track `origin/main` อยู่ และตอนนี้อยู่ตรงกันพอดี (ไม่มีข้อความ ahead/behind)
- **`feature-x`** — track `origin/feature-x` อยู่ แต่ local มี commit ที่ยังไม่ push อยู่ 2 อัน (`ahead 2`)
- **`experiment`** — **ไม่มี tracking เลย** (ไม่มีวงเล็บ `[...]` ต่อท้าย SHA) — เป็น branch ที่มีอยู่แค่ในเครื่อง local เท่านั้น ยังไม่เคยผูกกับ remote branch ใด

### สรุปคำสั่งทั้งหมดเกี่ยวกับ Tracking

| คำสั่ง | ใช้เมื่อไหร่ |
|---|---|
| `git push -u origin <branch>` | Push branch ใหม่ขึ้น remote ครั้งแรก พร้อมตั้ง tracking ในทีเดียว |
| `git branch --set-upstream-to=<remote>/<branch> <local-branch>` | branch มีอยู่แล้วทั้งสองฝั่ง แค่ยังไม่ผูก tracking |
| `git branch -u <remote>/<branch>` | รูปแบบย่อของข้างบน (ต้องอยู่บน branch นั้นแล้ว) |
| `git branch -vv` | ตรวจสอบว่า branch ไหน track อะไรอยู่ และสถานะ ahead/behind |
| `git branch --unset-upstream` | ยกเลิกการ track ของ branch ปัจจุบัน (เผื่อจำเป็น) |

---

## Step 90: แบบฝึกหัด — สร้าง Bare Repo จำลอง Remote แล้ว Clone มาทำงานจริง

ถึงเวลาลงมือปฏิบัติจริงแล้ว! แบบฝึกหัดนี้จะให้คุณสร้าง **"remote repository จำลอง"** ขึ้นมาเองบนเครื่อง โดยไม่ต้องพึ่งอินเทอร์เน็ตหรือบัญชี GitHub เลย เพื่อฝึกใช้ทุกคำสั่งที่เรียนมาใน Part นี้แบบครบวงจร

### 90.1 ทำความเข้าใจ Bare Repository ก่อน

ปกติเวลาคุณ `git init` ในโฟลเดอร์โปรเจกต์ คุณจะได้ repository ที่มีทั้ง **Working Directory** (ไฟล์จริงที่แก้ไขได้) และ **`.git` directory** (เก็บประวัติ) อยู่ด้วยกัน

แต่ **Bare Repository** คือ repository ที่มี **แค่ข้อมูลของ `.git` เท่านั้น ไม่มี Working Directory เลย** — ไม่มีไฟล์ให้แก้ไข ไม่มีใครสามารถเข้าไปทำงานในนั้นโดยตรงได้ มันมีหน้าที่แค่**เก็บข้อมูลและรอรับ push/fetch จากที่อื่น**เท่านั้น

นี่คือรูปแบบเดียวกับที่ GitHub, GitLab ใช้เก็บ repository ของคุณบนเซิร์ฟเวอร์จริง ๆ — เซิร์ฟเวอร์เหล่านั้นไม่มี "หน้าจอแก้โค้ด" อยู่ในตัว repo โดยตรง มันเป็นแค่ที่เก็บข้อมูล Git ล้วน ๆ ที่รอให้ client (เครื่องของคุณ) มา clone/fetch/push เท่านั้น

ตารางเปรียบเทียบ:

| คุณสมบัติ | Repository ปกติ | Bare Repository |
|---|---|---|
| มี Working Directory | มี | **ไม่มี** |
| แก้ไขไฟล์ตรง ๆ ได้ | ได้ | **ไม่ได้** |
| ใช้ทำอะไร | ทำงานจริง (เขียนโค้ด, commit) | เป็นจุดกลางให้ push/fetch/clone |
| นามสกุลโฟลเดอร์ตามธรรมเนียม | ชื่อโปรเจกต์ปกติ | ชื่อโปรเจกต์ + `.git` (เช่น `project.git`) |
| โครงสร้างภายใน | มีโฟลเดอร์ `.git` ซ่อนอยู่ข้างในโฟลเดอร์โปรเจกต์ | เนื้อหาของ `.git` ถูกวางไว้ตรง ๆ ที่ root ของโฟลเดอร์เลย |

### 90.2 เตรียมพื้นที่ฝึกฝน

เปิด terminal แล้วสร้างโฟลเดอร์สำหรับฝึกฝน Step นี้โดยเฉพาะ:

```bash
mkdir -p ~/git-course/part-09-remote
cd ~/git-course/part-09-remote
```

### 90.3 สร้าง Bare Repository จำลอง "Remote Server"

```bash
git init --bare remote-project.git
```

ผลลัพธ์:

```
Initialized empty Git repository in /home/user/git-course/part-09-remote/remote-project.git/
```

ลองดูโครงสร้างข้างในด้วย:

```bash
ls remote-project.git
```

จะเห็นเนื้อหาแบบนี้ (เทียบกับ `.git` ปกติ แต่วางตรงที่ root เลย):

```
HEAD  branches  config  description  hooks  info  objects  refs
```

โฟลเดอร์นี้จะทำหน้าที่เป็น **"remote server จำลอง"** ของเราตลอดแบบฝึกหัดนี้ — เปรียบเสมือน repo บน GitHub แต่อยู่บนเครื่องเราเอง

### 90.4 Clone จาก Bare Repo มาเป็น "เครื่องทำงานเครื่องที่ 1"

```bash
cd ~/git-course/part-09-remote
git clone remote-project.git developer-a
```

ผลลัพธ์:

```
Cloning into 'developer-a'...
warning: You appear to have cloned an empty repository.
```

ข้อความเตือนนี้เป็นเรื่องปกติ เพราะ bare repo ที่เราสร้างยังว่างเปล่าอยู่ ยังไม่มี commit ใด ๆ เลย

ตรวจสอบ remote ที่ถูกตั้งค่าอัตโนมัติ:

```bash
cd developer-a
git remote -v
```

ผลลัพธ์ (สังเกตว่า URL เป็น path บนเครื่อง เพราะเราไม่ได้ใช้อินเทอร์เน็ตเลย):

```
origin  /home/user/git-course/part-09-remote/remote-project.git (fetch)
origin  /home/user/git-course/part-09-remote/remote-project.git (push)
```

### 90.5 สร้างงานชิ้นแรกใน `developer-a` แล้ว Push ขึ้น "Remote"

```bash
echo "# Remote Practice Project" > README.md
git add README.md
git commit -m "Initial commit: add README"
git branch -M main
git push -u origin main
```

คำสั่ง `git push -u origin main` จะ push branch `main` ขึ้นไปสร้างใน bare repo เป็นครั้งแรก พร้อมตั้งค่า tracking ให้อัตโนมัติ (ตามที่เรียนใน Step 89)

ตรวจสอบด้วย `git branch -vv`:

```bash
git branch -vv
```

```
* main a1b2c3d [origin/main] Initial commit: add README
```

### 90.6 Clone จาก Bare Repo เดียวกันมาเป็น "เครื่องทำงานเครื่องที่ 2"

จำลองสถานการณ์เพื่อนร่วมทีมอีกคนหนึ่ง clone repo เดียวกันมาทำงาน:

```bash
cd ~/git-course/part-09-remote
git clone remote-project.git developer-b
cd developer-b
```

คราวนี้จะไม่มี warning เรื่อง empty repository แล้ว เพราะมี commit อยู่แล้วจากขั้นตอนก่อนหน้า ตรวจสอบไฟล์:

```bash
cat README.md
```

จะเห็นเนื้อหา `# Remote Practice Project` ที่ `developer-a` push ขึ้นไปแล้ว

### 90.7 จำลองงานใหม่จาก `developer-b` แล้ว Push กลับขึ้น Remote

```bash
echo "## Installation" >> README.md
echo "Run 'npm install' to get started." >> README.md
git add README.md
git commit -m "Add installation section to README"
git push
```

สังเกตว่าครั้งนี้ **ไม่ต้องใช้ `-u origin main` อีกแล้ว** เพราะ clone มาก็ได้ tracking อัตโนมัติอยู่แล้วตั้งแต่ต้น (ตามที่เรียนใน Step 82)

### 90.8 กลับไปที่ `developer-a` แล้ว Fetch ดูความเปลี่ยนแปลง

นี่คือหัวใจของแบบฝึกหัดนี้ — กลับไปที่โฟลเดอร์ `developer-a` ซึ่ง**ยังไม่รู้เลย**ว่า `developer-b` ได้ push งานใหม่ขึ้นไปแล้ว:

```bash
cd ~/git-course/part-09-remote/developer-a
git fetch origin
```

ผลลัพธ์:

```
From /home/user/git-course/part-09-remote/remote-project
   a1b2c3d..f4e5d6c  main       -> origin/main
```

ผลลัพธ์นี้บอกว่า `origin/main` ขยับจาก commit `a1b2c3d` ไปเป็น `f4e5d6c` แล้ว — แต่ **local branch `main` ของ `developer-a` ยังไม่ขยับตาม** เพราะเราสั่งแค่ `fetch` ไม่ใช่ `pull`

ลองตรวจสอบด้วยคำสั่งที่เรียนใน Step 87:

```bash
git status
```

```
On branch main
Your branch is behind 'origin/main' by 1 commit, and can be fast-forwarded.
  (use "git pull" to update your local branch)
```

ดู commit ที่ยังไม่มี:

```bash
git log main..origin/main --oneline
```

```
f4e5d6c Add installation section to README
```

ดู diff เนื้อหาที่จะเปลี่ยน:

```bash
git diff main origin/main
```

```diff
diff --git a/README.md b/README.md
index 8f3b1a2..d4c9e01 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,3 @@
 # Remote Practice Project
+## Installation
+Run 'npm install' to get started.
```

ตรวจสอบไฟล์จริงในเครื่องว่ายังไม่เปลี่ยน:

```bash
cat README.md
```

จะเห็นแค่บรรทัดเดิม `# Remote Practice Project` เท่านั้น — ยืนยันว่า `fetch` ไม่แตะต้อง Working Directory จริงตามที่เรียนใน Step 85

### 90.9 ตรวจสอบ Bare Repo โดยไม่ Clone ด้วย `git ls-remote`

ลองใช้คำสั่งจาก Step 88 เพื่อดูสถานะของ bare repo โดยตรงโดยไม่ต้องเข้าไปในโฟลเดอร์ clone ใด ๆ เลย:

```bash
cd ~/git-course/part-09-remote
git ls-remote remote-project.git
```

```
f4e5d6c...    HEAD
f4e5d6c...    refs/heads/main
```

จะเห็นว่า `refs/heads/main` บน bare repo ชี้ไปที่ `f4e5d6c` แล้ว — ตรงกับที่ `developer-b` push เข้าไปล่าสุด และตรงกับที่ `origin/main` ใน `developer-a` เพิ่งอัปเดตมา (แต่ local `main` ของ `developer-a` ยังไม่ตรงกัน)

### 90.10 ทำให้ Local Branch ตามทันด้วย `git merge` (หรือใช้ `git pull` เพื่อเทียบ)

ในที่สุดถึงเวลาทำให้ `developer-a` ตามทัน — เพราะเราตรวจสอบด้วย fetch, diff, log จนมั่นใจแล้วว่าปลอดภัย:

```bash
cd ~/git-course/part-09-remote/developer-a
git merge origin/main
```

```
Updating a1b2c3d..f4e5d6c
Fast-forward
 README.md | 2 ++
 1 file changed, 2 insertions(+)
```

ตรวจสอบไฟล์อีกครั้ง:

```bash
cat README.md
```

```
# Remote Practice Project
## Installation
Run 'npm install' to get started.
```

ตอนนี้ `developer-a` มีเนื้อหาล่าสุดตรงกับที่ `developer-b` push ไว้แล้ว โดยที่คุณ**ควบคุมทุกขั้นตอนได้เอง** ผ่าน fetch → ตรวจสอบ → merge แทนที่จะใช้ `git pull` แบบทันทีทันใดโดยไม่รู้ว่ากำลังจะเปลี่ยนอะไรบ้าง

### 90.11 สรุปภาพรวมของแบบฝึกหัดนี้เป็นไดอะแกรม

```
                  remote-project.git  (Bare Repo — จำลอง "GitHub")
                         ▲       │
                  push   │       │  fetch/clone
                         │       ▼
        ┌───────────────┘       └───────────────┐
        │                                        │
┌───────────────┐                       ┌───────────────┐
│  developer-a   │                       │  developer-b   │
│  (เครื่องที่ 1)  │                       │  (เครื่องที่ 2)  │
│                │                       │                │
│ 1. commit      │                       │                │
│    README.md   │                       │                │
│ 2. push -u     │──────────────────────▶│                │
│                │                       │ 3. clone       │
│                │                       │ 4. commit      │
│                │◀──────────────────────│    (เพิ่มเนื้อหา)│
│ 5. fetch       │                       │ 5. push        │
│ 6. diff/log    │                       │                │
│ 7. merge       │                       │                │
└───────────────┘                       └───────────────┘
```

แบบฝึกหัดนี้จำลองสถานการณ์การทำงานเป็นทีมแบบสมบูรณ์ในเวอร์ชันย่อ — คุณได้ฝึกใช้คำสั่งครบทุกตัวที่เรียนใน Part นี้: `git init --bare`, `git clone`, `git remote -v`, `git push -u`, `git fetch`, `git log A..B`, `git diff`, `git branch -vv`, `git ls-remote`, และ `git merge` โดยไม่ต้องพึ่งอินเทอร์เน็ตหรือบัญชีออนไลน์ใด ๆ เลย

### 90.12 Checklist ก่อนไป Part 10

ก่อนไปต่อ Part 10 ให้ตรวจสอบว่าคุณ:

- [ ] เข้าใจว่า Remote Repository คือ Git repository อีกชุดหนึ่งที่อยู่คนละที่กับ local repo แต่มีสถานะเท่าเทียมกันในทางเทคนิค
- [ ] เข้าใจว่า `git clone` ทำอะไรบ้างเบื้องหลัง (ดาวน์โหลด object, ตั้งชื่อ `origin`, สร้าง remote-tracking branch, ตั้งค่า tracking ให้ default branch)
- [ ] อธิบายได้ว่าทำไมบางโปรเจกต์ถึงมี remote มากกว่าหนึ่งตัว (เช่น `origin` กับ `upstream`)
- [ ] ใช้ `git remote add`, `git remote remove`, `git remote rename` ได้คล่อง
- [ ] อธิบายความต่างระหว่าง `git fetch` กับ `git pull` ได้ชัดเจน (fetch ไม่แตะ working directory, pull = fetch + merge/rebase)
- [ ] เข้าใจว่า remote-tracking branch (เช่น `origin/main`) คือ "ภาพจำล่าสุด" ที่อัปเดตเฉพาะตอน fetch/pull/push เท่านั้น
- [ ] ใช้ `git diff main origin/main` และ `git log main..origin/main` เพื่อตรวจสอบความแตกต่างก่อนตัดสินใจ merge ได้
- [ ] ใช้ `git ls-remote` เพื่อดู branch/tag บน remote โดยไม่ต้อง clone ได้
- [ ] ตั้งค่า upstream tracking branch ได้ทั้งด้วย `git push -u` และ `git branch --set-upstream-to`
- [ ] ลงมือสร้าง bare repository จำลอง remote และจำลองการทำงานของสองเครื่องได้สำเร็จด้วยตัวเอง

---

## สรุป Part 09

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Remote Repository** คือ Git repository อีกชุดหนึ่งที่อยู่คนละที่ ทำหน้าที่เป็น "จุดนัดพบ" สำหรับการประสานงานของทีม แม้ Git จะเป็น Distributed VCS ที่ไม่จำเป็นต้องมี server กลางก็ตาม
2. **`git clone`** ไม่ใช่แค่การดาวน์โหลดไฟล์ แต่คัดลอกประวัติทั้งหมด ตั้งค่า remote ชื่อ `origin` โดยอัตโนมัติ พร้อมสร้าง remote-tracking branch และผูก tracking ให้ default branch ในทีเดียว
3. Repo หนึ่งสามารถมี **remote หลายตัว** ได้ ที่พบบ่อยที่สุดคือ `origin` (fork ของคุณ) และ `upstream` (repo ต้นฉบับ) ในรูปแบบ Fork & Pull Request Workflow
4. จัดการ remote ได้ด้วย `git remote add/remove/rename/set-url/show`
5. **`git fetch`** ดึงข้อมูลมาอัปเดต remote-tracking branch โดยไม่แตะต้อง working directory เลย ในขณะที่ **`git pull`** คือ fetch บวก merge (หรือ rebase) ทันที — fetch จึงปลอดภัยกว่าเพราะให้คุณตรวจสอบก่อนตัดสินใจ
6. **Remote-tracking branch** (เช่น `origin/main`) คือตัวชี้ในเครื่อง local ที่จดจำสถานะล่าสุดของ branch บน remote ณ เวลาที่ fetch ครั้งล่าสุด ไม่ใช่การเชื่อมต่อแบบ real-time
7. เปรียบเทียบ local กับ remote-tracking branch ได้ด้วย `git diff` และ `git log A..B`
8. **`git ls-remote`** ใช้ดูรายการ branch/tag บน remote ได้โดยไม่ต้อง clone ทั้งชุด
9. การตั้งค่า **upstream tracking** ผ่าน `git push -u` หรือ `git branch --set-upstream-to` ทำให้ใช้ `git push`/`git pull` แบบสั้น ๆ ได้โดยไม่ต้องระบุ remote และ branch ทุกครั้ง
10. ฝึกฝนทุกแนวคิดข้างต้นแบบครบวงจรผ่านการสร้าง **bare repository** จำลอง remote บนเครื่องตัวเอง โดยไม่ต้องพึ่งอินเทอร์เน็ตเลย

Part หน้าเราจะเจาะลึกคำสั่ง `push` และ `pull` แบบเต็มรูปแบบ รวมถึงสถานการณ์ที่พบบ่อยที่สุดเมื่อทำงานกับ `origin` จริง เช่น การจัดการ branch ที่ diverge กัน และการแก้ปัญหาที่พบบ่อยเวลา push ไม่สำเร็จ

**ต่อไป:** [Part 10: push/pull และการทำงานกับ origin](./part-010-push-pull-origin.md)
