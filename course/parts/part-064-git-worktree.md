# Part 64: Git Worktree: ทำงานหลาย Branch พร้อมกัน

> **Step ในหลักสูตรนี้:** Step 631–640
> **เฟส:** 6 — Git ขั้นสูงระดับ Internals, Hooks, LFS, Performance
> **เป้าหมายของ Part นี้:** เข้าใจปัญหาที่เกิดจากการสลับ branch บ่อย ๆ ด้วย `git checkout`/`git switch` แบบปกติ และเรียนรู้วิธีใช้ `git worktree` เพื่อทำงานกับหลาย branch พร้อมกันในหลายโฟลเดอร์จริงบนเครื่องเดียว โดยยังใช้ `.git` ชุดเดียวกัน ครอบคลุมตั้งแต่การสร้าง จัดการ ลบ ไปจนถึงข้อจำกัดและ use case จริงที่ใช้งานบ่อยในทีมพัฒนา

---

## สารบัญของ Part นี้

- Step 631: ปัญหาที่ Git Worktree แก้ — การสลับ branch บ่อย ๆ กับของค้างที่ยังไม่เสร็จ
- Step 632: `git worktree add` — สร้าง Working Directory ใหม่ที่ Checkout Branch อื่นพร้อมกัน
- Step 633: ทำงานพร้อมกันหลาย Branch ในหลายโฟลเดอร์จริง (Share `.git` เดียวกัน)
- Step 634: `git worktree list` — ดู Worktree ทั้งหมดที่มีอยู่
- Step 635: `git worktree remove` — ลบ Worktree ที่ไม่ใช้แล้วอย่างปลอดภัย
- Step 636: ข้อจำกัดสำคัญ — Branch เดียวกัน Checkout พร้อมกันใน 2 Worktree ไม่ได้
- Step 637: Use Case จริงที่ Worktree ช่วยได้มาก
- Step 638: `git worktree prune` — ทำความสะอาด Reference ของ Worktree ที่ถูกลบทิ้งด้วยมือ
- Step 639: Worktree กับ Build Artifacts/Dependencies แยกกันต่อ Branch
- Step 640: แบบฝึกหัด — ใช้ Worktree จัดการ 3 Branch พร้อมกันในสถานการณ์จำลอง

---

## Step 631: ปัญหาที่ Git Worktree แก้ — การสลับ branch บ่อย ๆ กับของค้างที่ยังไม่เสร็จ

ก่อนจะรู้จัก `git worktree` เราต้องเข้าใจก่อนว่ามันมาแก้ "ความเจ็บปวด" อะไรในชีวิตประจำวันของคนใช้ Git

### สถานการณ์ที่เกิดขึ้นจริงแทบทุกวัน

ลองนึกภาพว่าคุณกำลังพัฒนาฟีเจอร์ใหม่อยู่บน branch `feature/payment-gateway` คุณแก้ไฟล์ไปหลายไฟล์ เพิ่ม logic ใหม่ ยังไม่เสร็จสมบูรณ์ ยัง compile ไม่ผ่านด้วยซ้ำ แล้วจู่ ๆ หัวหน้าทีมก็ทักมาว่า:

> "มีบั๊กร้ายแรงบน production ต้องรีบแก้ด่วนบน branch `main` เดี๋ยวนี้!"

ตอนนี้คุณมีทางเลือกไม่กี่ทางกับ Git แบบเดิม (ใช้ working directory เดียว):

**ทางเลือกที่ 1: `git stash` งานที่ทำค้างไว้**

```bash
git stash push -m "งานที่ทำค้างของ payment-gateway"
git switch main
# ไปแก้บั๊กด่วน
git switch feature/payment-gateway
git stash pop
```

ฟังดูใช้ได้ แต่ในทางปฏิบัติมีปัญหาแฝงหลายอย่าง:

- ถ้าคุณมี **ไฟล์ที่ยัง untracked อยู่หลายไฟล์** (เช่นไฟล์ config ใหม่ที่ยังไม่ได้ `git add`) การ stash ต้องเพิ่ม `-u` ไม่งั้นไฟล์เหล่านั้นจะไม่ถูก stash ไปด้วย ทำให้เวลาสลับ branch ไฟล์ untracked เหล่านั้นยังค้างอยู่ในโฟลเดอร์และอาจไปปนกับโค้ดของ branch `main` โดยไม่ได้ตั้งใจ
- ถ้าระหว่างที่คุณแก้บั๊กด่วนบน `main` แล้วมีการ build หรือรัน dev server ทิ้งไว้ (เช่น `npm run dev`, `webpack --watch`) กระบวนการเหล่านี้จะงงเพราะไฟล์ในโฟลเดอร์เปลี่ยนไปเป็นของ `main` กะทันหันโดยที่ process ไม่รู้ตัว
- ถ้าคุณลืม `git stash pop` ก่อนจะสลับไป branch อื่นอีกที stash ที่ค้างไว้อาจถูกลืมและสูญหายไปในที่สุด (แม้ Git จะเก็บไว้ใน reflog ระยะหนึ่ง แต่ก็เสี่ยงต่อการค้นหายาก)
- Editor/IDE ของคุณ (เช่น VS Code) ที่เปิดไฟล์ของ `feature/payment-gateway` ค้างอยู่ จะกลายเป็นแสดงเนื้อหาของ `main` ทันทีที่คุณสลับ branch ทำให้ context ที่คุณกำลังโฟกัสอยู่หายไปหมด ต้องเปิดไฟล์ใหม่ ต้องหาตำแหน่งที่ทำงานค้างอยู่ใหม่

**ทางเลือกที่ 2: commit งานที่ยังไม่เสร็จไปก่อน**

```bash
git add -A
git commit -m "WIP: ยังไม่เสร็จ อย่าเพิ่ง merge"
git switch main
```

วิธีนี้แก้ปัญหาไฟล์ untracked ได้ แต่สร้างปัญหาใหม่คือ **ประวัติ commit ของคุณเต็มไปด้วย commit ขยะที่ไม่มีความหมาย** เช่น "WIP", "fix", "temp save" ซึ่งทำให้ประวัติของโปรเจกต์อ่านยากขึ้นในระยะยาว และถ้าเผลอ push ขึ้นไปโดยไม่ได้ rebase ล้าง commit เหล่านี้ก่อน ทีมอื่นก็จะเห็น commit ที่ไม่สมบูรณ์ปนอยู่ในประวัติจริง

**ทางเลือกที่ 3: Clone repository ซ้ำอีกชุดในอีกโฟลเดอร์**

```bash
git clone <url> ../myproject-hotfix
cd ../myproject-hotfix
git switch main
```

วิธีนี้ดูเหมือนจะแก้ปัญหาได้ตรงที่สุด เพราะได้โฟลเดอร์แยกจริง ๆ ไม่ต้องยุ่งกับ stash หรือ commit ขยะ แต่ก็มีข้อเสียชัดเจน:

- **เปลืองพื้นที่ดิสก์มหาศาล** — repository ขนาดใหญ่ที่มีประวัติหลายปีอาจมีขนาดหลายร้อย MB ถึงหลาย GB การ clone ซ้ำทุกครั้งที่ต้องการทำงานคู่ขนานคือการเก็บสำเนา `.git` (object database) ซ้ำซ้อนแบบไม่จำเป็น
- **เปลืองเวลา** — การ clone repository ขนาดใหญ่ใช้เวลานาน โดยเฉพาะถ้าต้อง clone ผ่านเครือข่ายที่ช้า
- **Remote-tracking ไม่ sync กันอัตโนมัติ** — ถ้าคุณ fetch ใน clone หนึ่ง อีก clone หนึ่งจะไม่รู้เรื่องด้วย ต้อง fetch แยกกันเอง ทำให้ข้อมูลไม่ตรงกันได้ง่าย
- **การตั้งค่า local (hooks, config บางอย่าง, remote ที่เพิ่มเอง)** ต้องตั้งใหม่ทุกครั้งที่ clone

### Git Worktree คือคำตอบที่ Git ออกแบบมาให้โดยเฉพาะ

`git worktree` (เพิ่มเข้ามาใน Git ตั้งแต่เวอร์ชัน 2.5 ปี 2015) คือฟีเจอร์ที่ให้คุณ:

> **สร้าง working directory ใหม่อีกโฟลเดอร์หนึ่ง ที่ checkout branch คนละอันจาก repository เดิม โดยทั้งหมด "แชร์ object database (`.git`) เดียวกัน"**

พูดง่าย ๆ คือคุณจะได้:

- โฟลเดอร์แยกกันจริง ๆ สำหรับแต่ละ branch (ไฟล์ในแต่ละโฟลเดอร์ไม่ปนกัน)
- ไม่ต้อง stash หรือ commit ของค้าง — งานที่ทำค้างใน branch หนึ่งยังอยู่ตรงนั้น ไม่ถูกแตะต้องเลย
- ไม่ต้อง clone ซ้ำ — ประวัติ, object, config ทั้งหมดถูกแชร์กันในระดับ `.git` เดียว ไม่เปลืองพื้นที่ดิสก์เพิ่มเท่า clone ใหม่ทั้งชุด
- Fetch/pull ใน worktree ไหนก็ตาม ข้อมูล remote-tracking branch จะเห็นเหมือนกันในทุก worktree ทันที เพราะใช้ object database เดียวกัน

กลับไปที่สถานการณ์ตอนต้น ถ้าคุณใช้ worktree ตั้งแต่แรก การแก้บั๊กด่วนจะเป็นแบบนี้แทน:

```bash
git worktree add ../myproject-hotfix main
cd ../myproject-hotfix
# แก้บั๊ก, commit, push ได้เลย
# กลับไปที่โฟลเดอร์เดิม feature/payment-gateway ยังอยู่ครบ ไม่มีอะไรถูกแตะต้องเลย
```

ไม่มีการ stash, ไม่มี commit ขยะ, ไม่มีการ clone ซ้ำ และงานที่ทำค้างในโฟลเดอร์เดิมยังอยู่เหมือนเดิมทุกประการ — นี่คือแก่นของปัญหาที่ `git worktree` ถูกออกแบบมาแก้โดยตรง

### สรุปเปรียบเทียบทั้ง 4 วิธี

| วิธี | ต้อง Stash/Commit ของค้าง | เปลืองดิสก์ซ้ำซ้อน | Remote-tracking Sync กันอัตโนมัติ | เสี่ยงชนกับ Dev Server ที่รันอยู่ |
|---|---|---|---|---|
| Stash แล้วสลับ branch | ต้อง | ไม่เปลือง | Sync (ใช้ `.git` เดียว) | เสี่ยงสูง |
| Commit ของค้างเป็น WIP | ต้อง (แบบถาวรในประวัติ) | ไม่เปลือง | Sync (ใช้ `.git` เดียว) | เสี่ยงสูง |
| Clone repository ซ้ำ | ไม่ต้อง | เปลืองมาก | ไม่ Sync อัตโนมัติ | ไม่เสี่ยง (โฟลเดอร์แยก) |
| **`git worktree`** | **ไม่ต้อง** | **แทบไม่เปลือง** | **Sync (ใช้ `.git` เดียว)** | **ไม่เสี่ยง (โฟลเดอร์แยก)** |

จะเห็นว่า `git worktree` คือวิธีเดียวที่ได้ครบทั้ง 4 ข้อดีพร้อมกัน — นี่คือเหตุผลว่าทำไมมันถึงเป็นเครื่องมือสำคัญมากสำหรับนักพัฒนาที่ต้องทำงานคู่ขนานหลาย branch อยู่บ่อย ๆ

---

## Step 632: `git worktree add` — สร้าง Working Directory ใหม่ที่ Checkout Branch อื่นพร้อมกัน

คำสั่งหลักของฟีเจอร์นี้คือ `git worktree add`

### รูปแบบคำสั่งพื้นฐาน

```bash
git worktree add <path> <branch>
```

- `<path>` คือตำแหน่งโฟลเดอร์ใหม่ที่จะถูกสร้างขึ้น (ต้องเป็นโฟลเดอร์ที่ยังไม่มีอยู่ หรือเป็นโฟลเดอร์ว่างเปล่า)
- `<branch>` คือชื่อ branch ที่ต้องการ checkout ไปไว้ในโฟลเดอร์นั้น

### ตัวอย่างการใช้งานจริง

สมมติคุณกำลังอยู่ใน repository `myproject` ที่ path `~/projects/myproject` และกำลังทำงานอยู่บน branch `feature/payment-gateway`

```bash
cd ~/projects/myproject
git status
# On branch feature/payment-gateway
# ...มีไฟล์ที่ยังไม่ commit ค้างอยู่...

git worktree add ../myproject-main main
```

ผลลัพธ์ที่ได้:

```
Preparing worktree (checking out 'main')
HEAD is now at a1b2c3d ปรับปรุงระบบ login
```

Git จะสร้างโฟลเดอร์ `../myproject-main` ขึ้นมาใหม่ (อยู่ระดับเดียวกับ `myproject`) แล้ว checkout branch `main` ไว้ในนั้นให้อัตโนมัติ โดยที่โฟลเดอร์ `myproject` เดิมของคุณ **ไม่ถูกแตะต้องเลยแม้แต่นิดเดียว** — ไฟล์ที่ยังไม่ commit บน `feature/payment-gateway` ยังอยู่ครบทุกอย่าง

โครงสร้างโฟลเดอร์หลังรันคำสั่งจะเป็นแบบนี้:

```
~/projects/
├── myproject/              ← worktree เดิม (main worktree) อยู่บน feature/payment-gateway
│   ├── .git/                ← object database จริง อยู่ที่นี่ที่เดียว
│   └── ... ไฟล์โปรเจกต์ ...
└── myproject-main/          ← worktree ใหม่ อยู่บน main
    ├── .git                 ← เป็นแค่ "ไฟล์" ธรรมดา ไม่ใช่โฟลเดอร์ ชี้กลับไปหา myproject/.git
    └── ... ไฟล์โปรเจกต์ (เวอร์ชันของ main) ...
```

สังเกตว่าใน `myproject-main/` นั้น `.git` ไม่ใช่โฟลเดอร์เหมือนปกติ แต่เป็น**ไฟล์ข้อความ**ธรรมดาที่มีเนื้อหาประมาณนี้:

```
gitdir: /home/user/projects/myproject/.git/worktrees/myproject-main
```

นี่คือกลไกที่ Git ใช้เชื่อมโยง worktree ใหม่กลับไปหา object database ต้นทาง — ทุก object (blob, tree, commit) ถูกเก็บอยู่ที่ `.git` ของ **main worktree** เพียงชุดเดียว ส่วน worktree อื่น ๆ จะมีแค่ metadata เฉพาะของตัวเอง (เช่น HEAD ปัจจุบัน, index/staging area ของตัวเอง) เก็บไว้ในโฟลเดอร์ย่อย `.git/worktrees/<ชื่อ>/` ของ main worktree

### สร้าง Branch ใหม่พร้อมกับสร้าง Worktree ในคำสั่งเดียว

ถ้า branch ที่ต้องการยังไม่มีอยู่ คุณสามารถสร้างขึ้นใหม่พร้อมกับสร้าง worktree ได้เลยด้วย flag `-b`:

```bash
git worktree add -b feature/new-report ../myproject-new-report main
```

คำสั่งนี้จะ:

1. สร้าง branch ใหม่ชื่อ `feature/new-report` โดยแตกออกมาจาก `main`
2. สร้างโฟลเดอร์ `../myproject-new-report`
3. Checkout branch `feature/new-report` ที่เพิ่งสร้างไว้ในโฟลเดอร์นั้นทันที

รูปแบบเต็มคือ:

```bash
git worktree add -b <ชื่อ-branch-ใหม่> <path> <จุดเริ่มต้น>
```

โดย `<จุดเริ่มต้น>` จะเป็น branch, tag หรือ commit hash ก็ได้ ถ้าไม่ระบุ Git จะใช้ HEAD ปัจจุบันของ main worktree เป็นจุดเริ่มต้นให้อัตโนมัติ

### สร้าง Worktree แบบ Detached HEAD

บางครั้งคุณอาจต้องการแค่ดูโค้ดที่ commit หนึ่ง ๆ โดยไม่ต้องผูกกับ branch ใดเลย ใช้ flag `--detach`:

```bash
git worktree add --detach ../myproject-review a1b2c3d
```

ตรงนี้เหมาะมากสำหรับการเปิดดู snapshot ของ commit เก่า ๆ เพื่อ debug หรือเปรียบเทียบ โดยไม่ต้องกังวลว่าจะไปสร้าง branch ใหม่โดยไม่ตั้งใจ

### ถ้าไม่ระบุชื่อ branch เลย

ถ้าคุณรัน `git worktree add <path>` โดยไม่ระบุ branch เลย และ `<path>` ลงท้ายด้วยชื่อที่ตรงกับชื่อ branch ที่มีอยู่ Git จะพยายามเดาและ checkout branch ที่ชื่อตรงกับ path นั้นให้อัตโนมัติ เช่น:

```bash
git worktree add ../hotfix-urgent-bug
```

ถ้ามี branch ชื่อ `hotfix-urgent-bug` อยู่แล้ว Git จะ checkout branch นั้นให้เลย แต่ถ้าไม่มี Git จะสร้าง branch ใหม่ชื่อ `hotfix-urgent-bug` ให้แทน (พฤติกรรมคล้ายกับ `git checkout <branch>` ที่ไม่มีอยู่จะ error แต่ `git switch -c` จะสร้างให้ — ในกรณีนี้ Git จะพยายามช่วยเดาให้อัตโนมัติ)

เพื่อความชัดเจนและป้องกันความสับสน แนะนำให้ระบุ branch หรือใช้ `-b` อย่างชัดเจนเสมอในการทำงานจริง

---

## Step 633: ทำงานพร้อมกันหลาย Branch ในหลายโฟลเดอร์จริง (Share `.git` เดียวกัน)

หัวใจสำคัญที่ทำให้ worktree มีประโยชน์มากคือมันทำให้คุณ**ทำงานได้จริง**ในหลายโฟลเดอร์พร้อมกัน ไม่ใช่แค่ "ดู" branch อื่นเฉย ๆ

### เปิด Terminal หลายหน้าต่าง แต่ละหน้าต่างอยู่คนละ Worktree

สมมติคุณมีโครงสร้างแบบนี้หลังจากสร้าง worktree ไปแล้วหลายอัน:

```
~/projects/
├── myproject/              (branch: feature/payment-gateway)
├── myproject-main/         (branch: main)
└── myproject-hotfix/       (branch: hotfix/urgent-fix)
```

คุณสามารถเปิด Terminal 3 หน้าต่าง (หรือ 3 tab) แล้ว `cd` เข้าไปแต่ละโฟลเดอร์ได้เลย:

```bash
# หน้าต่างที่ 1
cd ~/projects/myproject
git status
# On branch feature/payment-gateway

# หน้าต่างที่ 2
cd ~/projects/myproject-main
git status
# On branch main

# หน้าต่างที่ 3
cd ~/projects/myproject-hotfix
git status
# On branch hotfix/urgent-fix
```

ทั้งสามโฟลเดอร์นี้เป็น**คนละไฟล์กันจริง ๆ บนดิสก์** การแก้ไขไฟล์ในโฟลเดอร์หนึ่งจะไม่ไปกระทบไฟล์ในอีกโฟลเดอร์เลย คุณสามารถ:

- เปิด IDE/Editor แยกกันคนละหน้าต่างสำหรับแต่ละโฟลเดอร์ โดยที่แต่ละหน้าต่างแสดง state ของ branch นั้น ๆ ตรงตามจริงตลอดเวลา
- รัน dev server หรือ test suite พร้อมกันในแต่ละโฟลเดอร์ โดยไม่ชนกัน (ตราบใดที่ตั้งค่า port ไม่ให้ชนกัน)
- Commit งานในแต่ละ worktree แยกกันอิสระ

### ทุก Worktree แชร์ Object Database และ Config บางส่วนร่วมกัน

แม้จะเป็นโฟลเดอร์แยกกัน แต่สิ่งที่ **แชร์กัน** ระหว่างทุก worktree มีดังนี้:

1. **Object database (`objects/`)** — commit, tree, blob ทั้งหมดถูกเก็บไว้ที่เดียว ไม่มีการคัดลอกซ้ำ
2. **Branch และ Tag references** — ถ้าคุณสร้าง branch ใหม่ในโฟลเดอร์หนึ่ง โฟลเดอร์อื่นจะเห็น branch นั้นทันที (แต่จะ checkout ไปใช้ในโฟลเดอร์ตัวเองไม่ได้ถ้า branch นั้นถูก checkout อยู่ที่อื่นแล้ว — รายละเอียดใน Step 636)
3. **Remote configuration (`remote.origin.url` ฯลฯ)** — ตั้งค่า remote ครั้งเดียว ใช้ได้ทุก worktree
4. **Stash** — `git stash list` จะแสดงผลเหมือนกันไม่ว่าจะรันจาก worktree ไหน เพราะ stash ถูกเก็บเป็น reference ใน object database เดียวกัน (ควรระวัง: `git stash pop`/`apply` จาก worktree หนึ่ง จะเอา stash ไป apply ที่โฟลเดอร์นั้น ไม่ใช่โฟลเดอร์ที่สร้าง stash)
5. **Git hooks (`.git/hooks/`)** — hooks ที่ตั้งค่าไว้ใน main worktree จะถูกใช้ร่วมกันกับทุก worktree โดยดีฟอลต์

สิ่งที่ **แยกกันเป็นของตัวเอง** ในแต่ละ worktree:

1. **HEAD** — แต่ละ worktree มี HEAD ของตัวเอง ชี้ไป branch คนละอันได้อิสระ
2. **Index/Staging area** — การ `git add` ในโฟลเดอร์หนึ่งไม่กระทบ staging area ของอีกโฟลเดอร์
3. **Working directory files** — ไฟล์จริงบนดิสก์ของแต่ละโฟลเดอร์
4. **การตั้งค่า config บางอย่างที่ตั้งแบบ per-worktree ได้** (เช่นถ้าเปิดใช้ `extensions.worktreeConfig` จะสามารถตั้ง config เฉพาะ worktree ผ่าน `git config --worktree` ได้)

### ทดสอบ Fetch จาก Worktree ไหนก็เห็นเหมือนกัน

```bash
cd ~/projects/myproject-main
git fetch origin
```

หลังจากนี้ ถ้าคุณไปที่ `~/projects/myproject` (worktree อื่น) แล้วรัน `git log origin/main` จะเห็นข้อมูลล่าสุดที่เพิ่ง fetch มาทันที เพราะ remote-tracking branch (`origin/main`) ถูกเก็บอยู่ใน object database เดียวกันที่ทุก worktree ใช้ร่วมกัน — นี่คือข้อได้เปรียบสำคัญเหนือการ clone แยกหลายชุด ที่ต้อง fetch แยกกันทุกโฟลเดอร์

---

## Step 634: `git worktree list` — ดู Worktree ทั้งหมดที่มีอยู่

เมื่อคุณสร้าง worktree ไปหลายอันแล้ว การจำว่ามีโฟลเดอร์ไหนอยู่บ้าง แต่ละโฟลเดอร์อยู่บน branch อะไร อาจเริ่มสับสน คำสั่ง `git worktree list` ช่วยให้เห็นภาพรวมทั้งหมดได้ในทีเดียว

### การใช้งานพื้นฐาน

รันจากโฟลเดอร์ไหนก็ได้ที่เป็นส่วนหนึ่งของ repository (ไม่ว่าจะเป็น main worktree หรือ worktree ใดก็ตาม):

```bash
git worktree list
```

ผลลัพธ์ตัวอย่าง:

```
/home/user/projects/myproject         a1b2c3d [feature/payment-gateway]
/home/user/projects/myproject-main    d4e5f6a [main]
/home/user/projects/myproject-hotfix  9f8e7d6 [hotfix/urgent-fix]
```

แต่ละบรรทัดแสดง:

1. **Path** เต็มของโฟลเดอร์ worktree นั้น
2. **Commit hash** (แบบย่อ) ที่ HEAD ของ worktree นั้นชี้อยู่
3. **ชื่อ branch** ที่ถูก checkout อยู่ในวงเล็บเหลี่ยม (หรือคำว่า `(detached HEAD)` ถ้าอยู่ในสถานะ detached)

### แสดงผลแบบละเอียดขึ้นด้วย `--verbose` หรือ `-v`

```bash
git worktree list --verbose
```

จะแสดงข้อมูลเพิ่มเติม เช่น สถานะ locked (ถ้ามี) และรายละเอียดอื่น ๆ ที่มีประโยชน์เวลาต้อง debug ปัญหาเกี่ยวกับ worktree

### แสดงผลแบบ Machine-readable ด้วย `--porcelain`

สำหรับใช้ใน script ที่ต้อง parse ผลลัพธ์อย่างแม่นยำ:

```bash
git worktree list --porcelain
```

ผลลัพธ์ตัวอย่าง:

```
worktree /home/user/projects/myproject
HEAD a1b2c3d4e5f6...
branch refs/heads/feature/payment-gateway

worktree /home/user/projects/myproject-main
HEAD d4e5f6a7b8c9...
branch refs/heads/main

worktree /home/user/projects/myproject-hotfix
HEAD 9f8e7d6c5b4a...
branch refs/heads/hotfix/urgent-fix
```

รูปแบบนี้แต่ละ worktree จะถูกคั่นด้วยบรรทัดว่าง และแต่ละ field จะขึ้นต้นด้วยชื่อ field ชัดเจน (`worktree`, `HEAD`, `branch`) ทำให้เขียน script อ่านค่าไปประมวลผลต่อได้ง่ายและแม่นยำ ไม่ต้องกังวลเรื่อง format เปลี่ยนไปในอนาคต (ต่างจาก output ปกติที่เป็น human-readable และอาจเปลี่ยนรูปแบบได้ในเวอร์ชันถัดไปของ Git)

### สังเกตว่า Main Worktree ก็ถูกนับรวมด้วยเสมอ

ข้อควรจำที่สำคัญคือ `git worktree list` จะแสดง **main worktree** (โฟลเดอร์ที่มี `.git` เป็นโฟลเดอร์จริง ไม่ใช่ไฟล์ชี้ไปที่อื่น) ไว้เป็นรายการแรกเสมอ ควบคู่ไปกับ worktree เพิ่มเติมที่คุณสร้างขึ้นทีหลังทั้งหมด

---

## Step 635: `git worktree remove` — ลบ Worktree ที่ไม่ใช้แล้วอย่างปลอดภัย

เมื่อทำงานกับ branch นั้นเสร็จแล้ว การลบโฟลเดอร์ทิ้งด้วยคำสั่งลบไฟล์ธรรมดา เช่น `rm -rf` **ไม่ใช่วิธีที่ถูกต้อง** เพราะจะทิ้ง metadata ค้างไว้ใน `.git/worktrees/` ของ main worktree ทำให้ Git สับสนภายหลัง ควรใช้คำสั่งเฉพาะทางแทน

### การใช้งานพื้นฐาน

```bash
git worktree remove <path>
```

ตัวอย่าง:

```bash
git worktree remove ../myproject-hotfix
```

คำสั่งนี้จะ:

1. ลบโฟลเดอร์ `../myproject-hotfix` ออกจากดิสก์ทั้งหมด
2. ลบ metadata ของ worktree นั้นออกจาก `.git/worktrees/`
3. ทำให้ `git worktree list` ไม่แสดง worktree นั้นอีกต่อไป

### Git ป้องกันการลบโดยไม่ตั้งใจ

ถ้า worktree นั้นมี **การเปลี่ยนแปลงที่ยังไม่ commit** (uncommitted changes) หรือมีไฟล์ untracked ที่สำคัญ Git จะปฏิเสธการลบและแจ้ง error ทันที:

```bash
git worktree remove ../myproject-hotfix
```

```
fatal: '../myproject-hotfix' contains modified or untracked files, use --force to delete it
```

นี่คือกลไกความปลอดภัยที่สำคัญมาก — Git จะไม่ยอมให้คุณลบงานที่ยังไม่ได้บันทึกทิ้งไปเฉย ๆ โดยไม่รู้ตัว

### บังคับลบด้วย `--force`

ถ้าคุณมั่นใจแล้วจริง ๆ ว่าไม่ต้องการไฟล์เหล่านั้นแล้ว สามารถบังคับลบได้ด้วย:

```bash
git worktree remove --force ../myproject-hotfix
```

หรือย่อเป็น `-f`:

```bash
git worktree remove -f ../myproject-hotfix
```

**คำเตือนสำคัญ:** การใช้ `--force` จะลบไฟล์ที่ยังไม่ commit ทิ้งไปอย่างถาวรโดยไม่มีการสำรองไว้เลย ควรตรวจสอบด้วย `git status` ในโฟลเดอร์นั้นก่อนเสมอว่าไม่มีงานสำคัญค้างอยู่ ก่อนที่จะสั่งลบแบบบังคับ

### ลำดับการทำงานที่แนะนำก่อนลบ Worktree

1. เข้าไปตรวจสอบสถานะก่อน:

```bash
cd ../myproject-hotfix
git status
```

2. ถ้ามีงานที่ต้องการเก็บไว้ ให้ commit หรือ push ขึ้น remote ก่อน:

```bash
git add -A
git commit -m "เก็บงานที่ทำค้างไว้ก่อนปิด worktree"
git push origin hotfix/urgent-fix
```

3. กลับไปที่ main worktree แล้วค่อยลบ:

```bash
cd ../myproject
git worktree remove ../myproject-hotfix
```

### สิ่งที่เกิดขึ้นกับ Branch หลังจากลบ Worktree

การลบ worktree **ไม่ได้ลบ branch ทิ้งไปด้วย** — branch ที่ worktree นั้นเคย checkout อยู่ยังคงอยู่ในระบบตามปกติ เพียงแต่ไม่มีโฟลเดอร์ไหน checkout อยู่แล้ว หากต้องการลบ branch ด้วย ต้องสั่งแยกต่างหาก:

```bash
git branch -d hotfix/urgent-fix
```

(ใช้ `-D` ตัวใหญ่แทนถ้า branch นั้นยังไม่ได้ merge และต้องการลบแบบบังคับ)

---

## Step 636: ข้อจำกัดสำคัญ — Branch เดียวกัน Checkout พร้อมกันใน 2 Worktree ไม่ได้

นี่คือกฎที่สำคัญที่สุดข้อหนึ่งของ `git worktree` ที่ต้องเข้าใจให้ชัดเจน เพราะเป็นสิ่งที่ผู้เริ่มต้นมักงงเมื่อเจอ error ครั้งแรก

### กฎ: หนึ่ง Branch ถูก Checkout ได้แค่ที่เดียวในเวลาเดียวกัน

Git **ป้องกันไม่ให้ branch เดียวกันถูก checkout อยู่ในสอง working directory พร้อมกัน** ไม่ว่าจะเป็น main worktree หรือ worktree เพิ่มเติมก็ตาม

ลองดูตัวอย่าง สมมติ `main` ถูก checkout อยู่ที่ main worktree `~/projects/myproject` อยู่แล้ว แล้วคุณพยายามสร้าง worktree ใหม่ด้วย branch `main` อีกอันหนึ่ง:

```bash
cd ~/projects/myproject
git branch
# * main

git worktree add ../myproject-main2 main
```

ผลลัพธ์ที่ได้คือ error ทันที:

```
fatal: 'main' is already used by worktree at '/home/user/projects/myproject'
```

### ทำไม Git ถึงต้องป้องกันแบบนี้

เหตุผลเชิงเทคนิคคือ **branch pointer คือ pointer เดียว** ที่ชี้ไปยัง commit หนึ่ง ถ้า Git ยอมให้สอง working directory ชี้ไปที่ branch เดียวกันพร้อมกัน แล้วมีการ commit เกิดขึ้นจากทั้งสองโฟลเดอร์พร้อม ๆ กัน จะทำให้เกิดสถานการณ์ที่ branch pointer ต้องขยับไปสองทิศทางพร้อมกัน ซึ่งขัดกับธรรมชาติของ Git ที่ branch คือ pointer เดียวที่ชี้ไปยัง commit ล่าสุดเพียงจุดเดียวเท่านั้น การป้องกันนี้จึงเป็นการปกป้องความสมบูรณ์ของข้อมูล (data integrity) ไม่ให้เกิดสถานะที่ขัดแย้งกันเอง

### วิธีแก้เมื่อเจอสถานการณ์นี้

**ทางเลือกที่ 1: ใช้ branch คนละอัน**

ถ้าคุณต้องการทำงานคู่ขนานจริง ๆ ให้สร้าง branch ใหม่แยกออกมาแทน:

```bash
git worktree add -b main-copy-for-testing ../myproject-main2 main
```

วิธีนี้จะสร้าง branch ใหม่ชื่อ `main-copy-for-testing` ที่แตกออกมาจาก `main` ณ จุดปัจจุบัน แล้ว checkout ไว้ที่ worktree ใหม่ — ทำให้ไม่ชนกับ branch `main` เดิมที่ถูก checkout อยู่ที่อื่นแล้ว

**ทางเลือกที่ 2: ใช้ Detached HEAD**

ถ้าคุณแค่ต้องการ "ดู" state ของ `main` ณ ตอนนี้ โดยไม่ต้องการ commit อะไรใหม่ลงไปบน branch นั้น ให้ใช้ detached HEAD แทน:

```bash
git worktree add --detach ../myproject-main-readonly main
```

วิธีนี้จะ checkout commit ล่าสุดของ `main` ไปไว้ในโฟลเดอร์ใหม่แบบ detached HEAD (ไม่ผูกกับ branch ใด) ซึ่ง Git ไม่ถือว่าเป็นการ "ใช้งาน branch main อยู่" จึงไม่ชนกับ worktree เดิมที่ checkout `main` อยู่จริง ๆ เหมาะมากสำหรับกรณีอยากรันเทสหรือดูโค้ดของ `main` เฉย ๆ โดยไม่คิดจะแก้ไขอะไร

**ทางเลือกที่ 3: ปิด Worktree เดิมที่ใช้ branch นั้นอยู่ก่อน**

ถ้าไม่จำเป็นต้องใช้ทั้งสองโฟลเดอร์พร้อมกันจริง ๆ ก็แค่ `git worktree remove` โฟลเดอร์เดิมออกก่อน แล้วค่อยสร้างใหม่ที่ path ใหม่ (แต่วิธีนี้เสียจุดประสงค์ของการทำงานคู่ขนานไป)

### ตารางสรุปข้อจำกัด

| สถานการณ์ | ทำได้หรือไม่ |
|---|---|
| Branch `A` checkout อยู่ที่ worktree 1 และพยายาม checkout `A` อีกที่ worktree 2 | ทำไม่ได้ (error ทันที) |
| Branch `A` checkout อยู่ที่ worktree 1 และสร้าง branch ใหม่ `B` (แตกจาก `A`) ที่ worktree 2 | ทำได้ |
| Branch `A` checkout อยู่ที่ worktree 1 และ checkout commit เดียวกันแบบ detached HEAD ที่ worktree 2 | ทำได้ |
| Tag เดียวกัน checkout พร้อมกันหลาย worktree (แบบ detached) | ทำได้ (เพราะไม่ใช่ branch pointer) |

### ข้อควรระวังเพิ่มเติม: Main Worktree ก็นับด้วย

หลายคนเข้าใจผิดว่ากฎนี้ใช้แค่กับ worktree เพิ่มเติมที่สร้างขึ้นทีหลัง แต่ความจริงแล้ว **main worktree ก็ถูกนับรวมในกฎนี้ด้วยเสมอ** — ถ้า branch `feature-x` ถูก checkout อยู่ใน main worktree อยู่แล้ว คุณก็ไม่สามารถสร้าง worktree ใหม่ด้วย branch `feature-x` ได้เช่นกัน จนกว่าจะสลับ main worktree ไป branch อื่นก่อน หรือใช้หนึ่งในสามทางเลือกข้างต้น

---

## Step 637: Use Case จริงที่ Worktree ช่วยได้มาก

มาดูสถานการณ์การทำงานจริงที่ `git worktree` แก้ปัญหาได้อย่างชัดเจนและคุ้มค่ามากที่จะเรียนรู้เครื่องมือนี้

### Use Case 1: รัน Test Suite บน Branch หนึ่ง ขณะพัฒนาอีก Branch

สมมติคุณกำลังพัฒนาฟีเจอร์ใหญ่บน `feature/big-refactor` และต้องแก้ไขไฟล์เยอะมาก การรัน test suite เต็มรูปแบบ (integration test, e2e test) อาจใช้เวลานาน 10-20 นาที ถ้าคุณรัน test บนโฟลเดอร์เดียวกับที่กำลังแก้โค้ดอยู่ คุณจะติดขัดไม่สามารถแก้ไฟล์ต่อได้จนกว่า test จะรันเสร็จ (เพราะไฟล์ที่กำลังแก้อาจถูก test อ่านค่าไปแล้วขณะที่คุณยังแก้ไม่เสร็จ ทำให้ผลเทสไม่น่าเชื่อถือ)

ด้วย worktree คุณสามารถแยกได้ทันที:

```bash
# Worktree หลักสำหรับพัฒนาต่อ
cd ~/projects/myproject   # อยู่บน feature/big-refactor

# สร้าง worktree แยกสำหรับรัน test ที่ commit ล่าสุดที่ push ไปแล้ว
git worktree add ../myproject-test-runner feature/big-refactor-testing
```

แนวทางที่ทำงานได้จริงคือ:

1. commit งานที่เสร็จเป็นระยะ ๆ (หรือ push ไปยัง remote branch ชั่วคราว)
2. ใน worktree ที่สอง ให้ `fetch` แล้ว checkout ไปที่ commit ล่าสุดที่ push มา
3. รัน test suite เต็มรูปแบบใน worktree ที่สอง โดยไม่ไปรบกวนโฟลเดอร์หลักที่กำลังแก้โค้ดอยู่ต่อเนื่อง

```bash
cd ../myproject-test-runner
git fetch origin
git checkout origin/feature/big-refactor
npm test  # หรือ pytest, go test ฯลฯ ปล่อยให้รันไปเรื่อย ๆ ในพื้นหลัง
```

ระหว่างที่ test suite กำลังรันอยู่ในโฟลเดอร์นี้ (อาจใช้เวลานาน) คุณสามารถกลับไปที่โฟลเดอร์หลักแล้วแก้โค้ดต่อได้ทันทีโดยไม่ต้องรอ

### Use Case 2: ทำ Hotfix ด่วนโดยไม่ทิ้งงานปัจจุบันที่ยัง Uncommitted

นี่คือ use case ที่ยกตัวอย่างไว้แล้วใน Step 631 แต่ขอขยายความให้ครบถ้วนมากขึ้น

สถานการณ์: คุณกำลังแก้ไฟล์ 15 ไฟล์อยู่บน `feature/payment-gateway` ยังไม่เสร็จ compile ไม่ผ่านด้วยซ้ำ แล้วมีบั๊กด่วนบน production ที่ต้องแก้ทันที

```bash
# ไม่ต้อง stash ไม่ต้อง commit อะไรเลย แค่สร้าง worktree ใหม่
git worktree add -b hotfix/critical-bug ../myproject-hotfix main

cd ../myproject-hotfix
# แก้บั๊ก
vim src/payment/validator.js
git add src/payment/validator.js
git commit -m "แก้บั๊ก validation ที่ทำให้ยอดเงินคำนวณผิด"
git push origin hotfix/critical-bug

# สร้าง Pull Request, ให้ทีม review, merge เข้า main
# หลังจาก merge และ deploy เรียบร้อยแล้ว ลบ worktree ทิ้ง
cd ../myproject
git worktree remove ../myproject-hotfix
```

ตลอดกระบวนการนี้ โฟลเดอร์ `myproject` (ที่มี `feature/payment-gateway` ค้างอยู่ 15 ไฟล์ที่ยังไม่เสร็จ) **ไม่ถูกแตะต้องแม้แต่นิดเดียว** คุณสามารถกลับไปทำต่อได้ทันทีในสภาพเดิมทุกประการ ไม่ต้องเสียเวลา pop stash หรือกังวลเรื่อง conflict จากการ stash

### Use Case 3: Review Pull Request โดยไม่รบกวนงานปัจจุบัน

เมื่อเพื่อนร่วมทีมเปิด Pull Request มาขอให้ review คุณสามารถสร้าง worktree แยกไว้สำหรับ review โดยเฉพาะ โดยไม่ต้องสลับออกจาก branch ที่กำลังทำงานอยู่:

```bash
git fetch origin pull/42/head:pr-42-review
git worktree add ../myproject-pr-42 pr-42-review
cd ../myproject-pr-42
# เปิด editor อ่านโค้ด รันเทส ทดลองรันแอปจริง
npm install
npm run dev
```

หลัง review เสร็จก็ `git worktree remove` ทิ้งได้เลย

### Use Case 4: เปรียบเทียบพฤติกรรมระหว่างสองเวอร์ชัน (Bisect เชิง Manual)

เวลาต้องการเปรียบเทียบพฤติกรรมของแอประหว่าง commit เก่ากับใหม่แบบเห็นภาพจริงพร้อมกัน (ไม่ใช่แค่ดู diff) worktree ช่วยให้เปิดสอง version รันคู่กันได้จริง:

```bash
git worktree add --detach ../myproject-v1.0 v1.0.0
git worktree add --detach ../myproject-v2.0 v2.0.0

# รันทั้งสองเวอร์ชันพร้อมกันคนละ port เพื่อเปรียบเทียบ
cd ../myproject-v1.0 && npm run dev -- --port 3000 &
cd ../myproject-v2.0 && npm run dev -- --port 3001 &
```

### Use Case 5: Build สำหรับหลาย Environment/Platform พร้อมกัน

ทีมที่ต้อง build โปรเจกต์สำหรับหลาย branch พร้อมกัน (เช่น branch `release/v1` สำหรับลูกค้าเก่า และ `release/v2` สำหรับลูกค้าใหม่) สามารถตั้ง worktree แยกไว้ถาวรสำหรับแต่ละ release line แล้วปล่อยให้ CI/build script รันจากแต่ละโฟลเดอร์แยกกันได้ โดยไม่ต้องสลับ checkout ไปมาบนเครื่อง build เครื่องเดียวกัน ซึ่งจะช่วยลดความเสี่ยงเรื่อง build cache ปนกันข้าม branch ได้มาก (รายละเอียดเพิ่มเติมใน Step 639)

---

## Step 638: `git worktree prune` — ทำความสะอาด Reference ของ Worktree ที่ถูกลบทิ้งด้วยมือ

ในทางปฏิบัติ บางครั้งคนอาจลบโฟลเดอร์ worktree ทิ้งไปด้วยคำสั่งลบไฟล์ธรรมดา (เช่น `rm -rf ../myproject-hotfix`) แทนที่จะใช้ `git worktree remove` อย่างถูกวิธี หรือโฟลเดอร์นั้นอาจถูกลบไปเพราะย้ายดิสก์ ลบทิ้งโดยไม่ได้ตั้งใจ หรือทำงานบน network drive ที่หลุดการเชื่อมต่อ

### ปัญหาที่เกิดขึ้นเมื่อลบโฟลเดอร์ด้วยมือ

ถ้าคุณลบโฟลเดอร์ worktree ด้วย `rm -rf` โดยตรงแล้วลองรัน:

```bash
git worktree list
```

Git จะยังคงแสดง worktree นั้นอยู่ในรายการ แม้ว่าโฟลเดอร์จริงบนดิสก์จะไม่มีอยู่แล้วก็ตาม เพราะ metadata ใน `.git/worktrees/<ชื่อ>/` ยังไม่ถูกลบออกไป Git ไม่รู้ว่าโฟลเดอร์นั้นหายไปแล้วจนกว่าจะมีการตรวจสอบ

ตัวอย่างผลลัพธ์:

```
/home/user/projects/myproject         a1b2c3d [feature/payment-gateway]
/home/user/projects/myproject-hotfix  9f8e7d6 [hotfix/urgent-fix] prunable
```

สังเกตคำว่า `prunable` ต่อท้าย — นี่คือสัญญาณที่ Git ใช้บอกว่า worktree นี้ตรวจพบว่าโฟลเดอร์จริงหายไปแล้ว และพร้อมที่จะถูก "prune" (ทำความสะอาด) ทิ้ง

ปัญหาที่ตามมาจากสถานะค้างนี้คือ:

- Branch `hotfix/urgent-fix` จะยังคงถูกมองว่า "ใช้งานอยู่โดย worktree" (ตามกฎใน Step 636) ทำให้คุณไม่สามารถสร้าง worktree ใหม่ด้วย branch นี้ที่โฟลเดอร์อื่นได้ จนกว่าจะเคลียร์ metadata ค้างนี้ทิ้งก่อน
- `git worktree list` แสดงข้อมูลที่ไม่ตรงกับความเป็นจริง ทำให้สับสนเวลาตรวจสอบภาพรวม

### วิธีแก้: `git worktree prune`

```bash
git worktree prune
```

คำสั่งนี้จะสแกนหา worktree ทั้งหมดที่ metadata ยังอยู่ แต่โฟลเดอร์จริงบนดิสก์หายไปแล้ว แล้วลบ metadata ที่ค้างอยู่ใน `.git/worktrees/` ออกให้อัตโนมัติ

หลังรันคำสั่งนี้ `git worktree list` จะไม่แสดง worktree ที่หายไปนั้นอีกต่อไป และ branch ที่เคยถูก "ล็อก" ไว้ก็จะกลับมาใช้ checkout ที่อื่นได้ตามปกติ

### ดูล่วงหน้าก่อนว่าจะ Prune อะไรบ้างด้วย `--dry-run`

ถ้าไม่แน่ใจว่า `git worktree prune` จะลบอะไรไปบ้าง สามารถดูตัวอย่างล่วงหน้าได้โดยไม่มีผลจริงด้วย:

```bash
git worktree prune --dry-run --verbose
```

คำสั่งนี้จะพิมพ์รายการ worktree ที่**จะถูก** prune ออกมาให้ดู แต่ยังไม่ลบจริง ช่วยให้ตรวจสอบความถูกต้องก่อนได้

### กำหนดระยะเวลาผ่อนผันด้วย `--expire`

โดยดีฟอลต์ Git จะไม่ prune worktree ที่เพิ่งตรวจพบว่าหายไปทันที แต่จะให้ระยะเวลาผ่อนผันสั้น ๆ ก่อน (ป้องกันกรณีที่ network drive หลุดชั่วคราวแล้วกลับมาใหม่) สามารถกำหนดระยะเวลาเองได้ด้วย:

```bash
git worktree prune --expire 3.days.ago
```

คำสั่งนี้จะ prune เฉพาะ worktree ที่หายไปนานเกิน 3 วันแล้วเท่านั้น ถ้าต้องการ prune ทันทีไม่ต้องรอเลยให้ใช้:

```bash
git worktree prune --expire now
```

### ควรใช้ `git worktree remove` เป็นหลัก ใช้ `prune` เป็นแผนสำรอง

ข้อควรจำสำคัญคือ **`git worktree prune` เป็นเครื่องมือสำหรับ "แก้ไขสถานการณ์ที่พลาดไปแล้ว"** ไม่ใช่ workflow หลักที่ควรใช้ประจำ วิธีที่ถูกต้องในการเลิกใช้ worktree คือใช้ `git worktree remove` เสมอ เพราะมันจะตรวจสอบ uncommitted changes ให้ก่อนลบ (ตามที่อธิบายใน Step 635) ในขณะที่การลบโฟลเดอร์ด้วยมือแล้วค่อย prune ทีหลังนั้นข้ามการตรวจสอบความปลอดภัยนี้ไปโดยสิ้นเชิง เสี่ยงต่อการทำงานสำคัญหายไปโดยไม่รู้ตัว

---

## Step 639: Worktree กับ Build Artifacts/Dependencies แยกกันต่อ Branch

ประเด็นที่มักถูกมองข้ามเมื่อเริ่มใช้ `git worktree` คือเรื่อง **ไฟล์ที่ไม่ได้อยู่ภายใต้การควบคุมของ Git** เช่น `node_modules/`, โฟลเดอร์ build output, ไฟล์ cache ของ compiler เป็นต้น

### ทำไมแต่ละ Worktree ต้องมี Dependencies ของตัวเอง

เนื่องจากแต่ละ worktree คือ**โฟลเดอร์แยกกันจริงบนดิสก์** ไฟล์ที่ไม่ได้ถูก track โดย Git (ซึ่งปกติจะอยู่ใน `.gitignore` เช่น `node_modules/`, `vendor/`, `build/`, `dist/`, `target/`) **จะไม่ถูกแชร์ระหว่าง worktree โดยอัตโนมัติเลย**

ตัวอย่างเช่น ถ้าคุณสร้าง worktree ใหม่:

```bash
git worktree add ../myproject-feature-x feature/x
cd ../myproject-feature-x
npm run dev
```

```
Error: Cannot find module 'express'
```

จะเกิด error ทันทีเพราะโฟลเดอร์ `../myproject-feature-x` เป็นโฟลเดอร์ใหม่ที่ยังไม่มี `node_modules/` เลย คุณจำเป็นต้อง install dependencies ใหม่ในทุก worktree ที่สร้างขึ้น:

```bash
cd ../myproject-feature-x
npm install
```

### ข้อดีที่แฝงอยู่ในข้อจำกัดนี้

แม้จะฟังดูเป็นภาระเพิ่ม แต่การที่แต่ละ worktree มี `node_modules/` แยกกันจริง ๆ กลับเป็น**ข้อดีที่สำคัญมาก**ในหลายสถานการณ์:

1. **ป้องกันปัญหา dependency version ปนกันระหว่าง branch** — ถ้า branch `main` ใช้ package เวอร์ชันหนึ่ง และ `feature/upgrade-deps` กำลังทดลองอัปเกรด package เป็นอีกเวอร์ชันหนึ่ง การมี `node_modules/` แยกกันคนละโฟลเดอร์ทำให้ไม่มีทางที่ทั้งสอง branch จะไปเหยียบ dependency กันโดยไม่ตั้งใจ ต่างจากการสลับ branch ในโฟลเดอร์เดียว ที่บางครั้ง `node_modules/` เก่าที่ install ไว้ก่อนหน้าอาจตกค้างไม่ตรงกับ `package-lock.json` ของ branch ปัจจุบัน ทำให้เกิดบั๊กประหลาดที่ debug ยาก
2. **Build cache ไม่ปนกันข้าม branch** — เช่น cache ของ Webpack, Vite, หรือ TypeScript incremental build จะแยกกันชัดเจนต่อ worktree ลดโอกาสเจอปัญหา build เพี้ยนเพราะ cache เก่าจาก branch อื่น
3. **รันหลาย dev server พร้อมกันได้จริงโดยไม่ชนกัน** — ตราบใดที่ตั้งค่า port ต่างกัน แต่ละ worktree มีชุด dependencies ของตัวเองที่สอดคล้องกับโค้ดใน branch นั้นจริง ๆ

### กลยุทธ์ที่แนะนำสำหรับจัดการ Dependencies ในหลาย Worktree

**กลยุทธ์ที่ 1: ใช้ Package Manager Cache ร่วมกัน (แนะนำ)**

Package manager สมัยใหม่ส่วนใหญ่ (npm, yarn, pnpm) มี **global cache** ที่แยกจาก `node_modules/` ของแต่ละโปรเจกต์ การ install ครั้งแรกอาจโหลดจากอินเทอร์เน็ต แต่ install ครั้งถัดไปในโฟลเดอร์อื่นจะดึงจาก cache เครื่องแทน ทำให้เร็วขึ้นมาก โดยเฉพาะ **pnpm** ที่ใช้ hard link จาก global store ทำให้ `node_modules/` ในแต่ละ worktree แทบไม่กินพื้นที่ดิสก์เพิ่มเลยแม้จะมีหลายชุด:

```bash
cd ../myproject-feature-x
pnpm install   # ใช้ global store ร่วมกัน ประหยัดพื้นที่และเวลามาก
```

**กลยุทธ์ที่ 2: เขียน Script อัตโนมัติที่ Setup Worktree ใหม่ทุกครั้ง**

ทีมที่ใช้ worktree บ่อย ๆ มักเขียน shell script ช่วยลดขั้นตอนซ้ำ ๆ เช่น:

```bash
#!/bin/bash
# scripts/new-worktree.sh
BRANCH=$1
PATH_NAME=$2

git worktree add -b "$BRANCH" "$PATH_NAME" main
cd "$PATH_NAME"
npm install
cp ../myproject/.env.example .env
echo "Worktree พร้อมใช้งานที่ $PATH_NAME"
```

**กลยุทธ์ที่ 3: ระวังไฟล์ Environment/Secret ที่ไม่ได้อยู่ใน Git**

ไฟล์อย่าง `.env` ที่มักถูกใส่ใน `.gitignore` (เพราะมี secret) จะไม่ถูกคัดลอกไปยัง worktree ใหม่โดยอัตโนมัติเช่นกัน ต้องคัดลอกด้วยมือหรือใน script setup ทุกครั้งที่สร้าง worktree ใหม่ มิฉะนั้นแอปจะรันไม่ได้เพราะขาด environment variable ที่จำเป็น

**กลยุทธ์ที่ 4: ใช้ Symlink สำหรับไฟล์ที่ต้องการแชร์จริง ๆ (ระวังให้ดี)**

ในบางกรณีที่ต้องการประหยัดพื้นที่ดิสก์อย่างมาก อาจ symlink `node_modules/` ข้าม worktree ได้ แต่ควรทำเฉพาะกรณีที่มั่นใจว่า dependency version ตรงกันทุกประการระหว่าง branch เท่านั้น เพราะถ้า `package.json`/`package-lock.json` ต่างกันระหว่าง branch การ symlink แบบนี้จะทำให้เกิดปัญหา dependency ไม่ตรงกับโค้ดได้ทันที โดยทั่วไปไม่แนะนำวิธีนี้สำหรับมือใหม่

### สรุปตาราง: สิ่งที่แชร์กับไม่แชร์ระหว่าง Worktree

| ประเภทไฟล์/ข้อมูล | แชร์ระหว่าง Worktree หรือไม่ |
|---|---|
| Git object database (commit, blob, tree) | แชร์ |
| Branch/Tag references | แชร์ |
| Remote configuration | แชร์ |
| Stash | แชร์ |
| Git hooks (ดีฟอลต์) | แชร์ |
| HEAD ปัจจุบัน | ไม่แชร์ (แยกต่อ worktree) |
| Index/Staging area | ไม่แชร์ |
| Working directory files (โค้ดจริง) | ไม่แชร์ |
| `node_modules/`, `vendor/` และ dependencies อื่น ๆ | ไม่แชร์ (ต้อง install ใหม่ทุกที่) |
| ไฟล์ `.env` และไฟล์ config ที่ไม่ได้ track | ไม่แชร์ (ต้องคัดลอกเอง) |
| Build output/cache (`dist/`, `build/`, `.next/`) | ไม่แชร์ |

---

## Step 640: แบบฝึกหัด — ใช้ Worktree จัดการ 3 Branch พร้อมกันในสถานการณ์จำลอง

ถึงเวลาลงมือทำจริง เราจะจำลองสถานการณ์ที่สมจริงที่สุด: มี branch `main` สำหรับ build/production, `feature/dashboard` สำหรับพัฒนาฟีเจอร์ใหม่ที่กำลังทำอยู่ และ `hotfix/login-error` สำหรับแก้บั๊กด่วน

### เตรียมโปรเจกต์ทดลอง

```bash
mkdir -p ~/git-course/part-64-worktree
cd ~/git-course/part-64-worktree
git init worktree-demo
cd worktree-demo

echo "# Worktree Demo Project" > README.md
git add README.md
git commit -m "commit เริ่มต้นโปรเจกต์"
```

### ขั้นตอนที่ 1: สร้าง Branch `feature/dashboard` และจำลองงานที่ทำค้าง

```bash
git switch -c feature/dashboard
echo "function renderDashboard() { /* ยังไม่เสร็จ */ }" > dashboard.js
git add dashboard.js
git commit -m "เริ่มพัฒนา dashboard (ยังไม่เสร็จ)"

# จำลองว่ากำลังแก้ไฟล์อยู่ ยังไม่ commit
echo "// TODO: เพิ่ม chart component ตรงนี้" >> dashboard.js
git status
```

ตรวจสอบด้วย `git status` จะเห็นว่า `dashboard.js` มีการแก้ไขที่ยังไม่ commit อยู่ — นี่คือ "งานที่ทำค้าง" ที่เราจะพิสูจน์ว่า worktree จะไม่ไปแตะต้องมันเลย

### ขั้นตอนที่ 2: สร้าง Worktree แยกสำหรับ `main` ไว้ทำ Build

```bash
cd ~/git-course/part-64-worktree
git -C worktree-demo worktree add ../worktree-demo-build main
```

หรือจะ `cd` เข้าไปในโฟลเดอร์ก่อนก็ได้:

```bash
cd worktree-demo
git worktree add ../worktree-demo-build main
```

ตรวจสอบผล:

```bash
git worktree list
```

ควรเห็นผลลัพธ์ประมาณนี้:

```
/home/user/git-course/part-64-worktree/worktree-demo        <hash> [feature/dashboard]
/home/user/git-course/part-64-worktree/worktree-demo-build  <hash> [main]
```

### ขั้นตอนที่ 3: สร้าง Worktree แยกสำหรับ `hotfix/login-error`

จำลองสถานการณ์ที่ต้องรีบแก้บั๊กด่วน โดยไม่รบกวนงานที่ทำค้างใน `feature/dashboard`:

```bash
git worktree add -b hotfix/login-error ../worktree-demo-hotfix main
cd ../worktree-demo-hotfix
echo "// แก้บั๊ก: validate email ก่อน login" > login-fix.js
git add login-fix.js
git commit -m "แก้บั๊ก login ที่ validate email ผิด"
```

### ขั้นตอนที่ 4: พิสูจน์ว่างานที่ทำค้างใน `feature/dashboard` ไม่ถูกแตะต้อง

กลับไปที่โฟลเดอร์เดิม:

```bash
cd ../worktree-demo
git status
cat dashboard.js
```

ควรเห็นว่า `dashboard.js` ยังคงมีทั้งบรรทัดที่ commit ไปแล้วและบรรทัด `// TODO:` ที่ยังไม่ commit อยู่ครบถ้วนทุกประการ เหมือนกับตอนก่อนที่เราจะไปสร้าง worktree อื่นเลย — นี่คือหลักฐานที่ชัดเจนว่า worktree แต่ละอันเป็นอิสระต่อกันอย่างแท้จริง

### ขั้นตอนที่ 5: ทดลองสร้าง Worktree ด้วย Branch ที่ถูกใช้อยู่แล้ว (ดูข้อจำกัด)

ลองยืนยันข้อจำกัดจาก Step 636 ด้วยตัวเอง:

```bash
git worktree add ../worktree-demo-dashboard2 feature/dashboard
```

ควรได้ error ประมาณ:

```
fatal: 'feature/dashboard' is already used by worktree at '.../worktree-demo'
```

ลองแก้ปัญหาด้วยการสร้าง branch ใหม่แทน:

```bash
git worktree add -b feature/dashboard-experiment ../worktree-demo-dashboard2 feature/dashboard
```

คราวนี้ควรสำเร็จ เพราะเป็น branch คนละชื่อ (แม้จะแตกจากจุดเดียวกัน)

### ขั้นตอนที่ 6: ตรวจสอบภาพรวมทั้งหมดด้วย `git worktree list`

```bash
cd ../worktree-demo
git worktree list --verbose
```

ตรวจสอบว่าตอนนี้มี worktree ทั้งหมด 4 อัน: `worktree-demo` (feature/dashboard), `worktree-demo-build` (main), `worktree-demo-hotfix` (hotfix/login-error), `worktree-demo-dashboard2` (feature/dashboard-experiment)

### ขั้นตอนที่ 7: จำลองการลบโฟลเดอร์ด้วยมือแล้วใช้ `prune` แก้ไข

```bash
rm -rf ../worktree-demo-dashboard2
git worktree list
```

จะเห็นว่า `worktree-demo-dashboard2` ยังคงแสดงอยู่ในรายการ (อาจมีคำว่า `prunable` ต่อท้าย) ทั้งที่โฟลเดอร์จริงหายไปแล้ว ลองเคลียร์:

```bash
git worktree prune --verbose
git worktree list
```

ควรเห็นว่า `worktree-demo-dashboard2` หายไปจากรายการเรียบร้อยแล้ว

### ขั้นตอนที่ 8: ทำความสะอาดให้ถูกวิธีด้วย `git worktree remove`

เมื่อ hotfix เสร็จแล้ว (สมมติว่า merge เข้า `main` เรียบร้อยแล้ว) ให้ลบ worktree ที่ไม่ใช้แล้วอย่างถูกวิธี:

```bash
git worktree remove ../worktree-demo-hotfix
git worktree remove ../worktree-demo-build
git worktree list
```

ควรเหลือแค่ `worktree-demo` (บน `feature/dashboard`) ที่ยังมีงานทำค้างอยู่เหมือนเดิมทุกประการ — ซึ่งพิสูจน์ให้เห็นภาพรวมทั้งหมดของแบบฝึกหัดนี้ว่า worktree ช่วยให้คุณสลับไปมาระหว่างงานด่วนกับงานหลักได้อย่างปลอดภัย โดยไม่ต้องยุ่งกับ stash หรือ commit ขยะเลยแม้แต่ครั้งเดียว

### โจทย์เพิ่มเติมสำหรับฝึกฝนต่อ (ทำเอง)

1. ลองสร้าง worktree ใหม่ด้วย `--detach` แล้วดูว่า `git status` แสดงข้อความอย่างไรต่างจาก worktree ที่ผูกกับ branch ปกติ
2. ลองรัน `git fetch` จาก worktree หนึ่ง แล้วไปตรวจสอบที่ worktree อื่นว่าเห็นข้อมูลใหม่หรือไม่
3. ลองสร้างไฟล์ `.gitignore` ที่ ignore โฟลเดอร์ `node_modules/` แล้วจำลองการ `npm init` และเพิ่ม dependency คนละตัวในแต่ละ worktree เพื่อดูว่าไม่ปนกันจริง
4. ลองใช้ `git worktree remove` โดยไม่มี `--force` กับ worktree ที่มีไฟล์ยังไม่ commit อยู่ เพื่อดู error message ที่ Git แจ้งเตือน

### Checklist ก่อนไป Part 65

ก่อนไปต่อ Part 65 ให้ตรวจสอบว่าคุณ:

- [ ] เข้าใจปัญหาที่เกิดจากการสลับ branch บ่อย ๆ ด้วย stash/commit แบบเดิม และทำไม worktree ถึงแก้ปัญหานี้ได้ดีกว่า
- [ ] สร้าง worktree ใหม่ได้ด้วย `git worktree add <path> <branch>` และรู้จักการใช้ `-b` เพื่อสร้าง branch ใหม่พร้อมกัน
- [ ] เข้าใจว่าแต่ละ worktree เป็นโฟลเดอร์จริงที่แยกกัน แต่แชร์ object database (`.git`) เดียวกันกับ main worktree
- [ ] ใช้ `git worktree list` และ `git worktree list --porcelain` เพื่อตรวจสอบภาพรวมของ worktree ทั้งหมดได้
- [ ] ใช้ `git worktree remove` เพื่อลบ worktree อย่างปลอดภัย และรู้ว่าเมื่อไหร่ต้องใช้ `--force`
- [ ] เข้าใจกฎสำคัญว่า branch เดียวกันถูก checkout พร้อมกันในสอง worktree ไม่ได้ และรู้วิธีแก้ปัญหา 3 แบบ (branch ใหม่, detached HEAD, ลบ worktree เดิมก่อน)
- [ ] นึกภาพออกว่า use case ไหนบ้างในงานจริงที่ worktree ช่วยได้มาก (รัน test คู่ขนาน, ทำ hotfix ด่วน, review PR, build หลาย release line)
- [ ] ใช้ `git worktree prune` เพื่อเคลียร์ metadata ของ worktree ที่ถูกลบโฟลเดอร์ทิ้งด้วยมือได้
- [ ] เข้าใจว่า dependencies (`node_modules/`), ไฟล์ `.env`, และ build cache ไม่ถูกแชร์ระหว่าง worktree โดยอัตโนมัติ ต้องจัดการแยกต่างหากในแต่ละโฟลเดอร์
- [ ] ทำแบบฝึกหัดจำลอง 3 branch พร้อมกันด้วยตัวเองจนครบทุกขั้นตอน

---

## สรุป Part 64

ใน Part นี้เราได้เรียนรู้ว่า:

1. การสลับ branch บ่อย ๆ ด้วยวิธีเดิม (stash, commit ขยะ, หรือ clone ซ้ำ) ล้วนมีข้อเสียชัดเจน ทั้งเสี่ยงต่อการทำงานเสียหาย เปลืองเวลา หรือเปลืองพื้นที่ดิสก์
2. `git worktree add <path> <branch>` สร้าง working directory ใหม่ที่ checkout branch อื่นพร้อมกันได้ทันที โดยยังใช้ object database (`.git`) เดียวกันกับ repository เดิม
3. แต่ละ worktree เป็นโฟลเดอร์จริงที่แยกจากกันสมบูรณ์ในแง่ของไฟล์, HEAD และ staging area แต่แชร์ commit, branch, remote configuration และ stash ร่วมกัน
4. `git worktree list` (และรูปแบบ `--porcelain` สำหรับ script) ช่วยให้เห็นภาพรวมของทุก worktree ที่มีอยู่ในระบบ
5. `git worktree remove` คือวิธีที่ถูกต้องในการลบ worktree อย่างปลอดภัย โดย Git จะป้องกันการลบงานที่ยังไม่ commit ทิ้งโดยไม่ตั้งใจ เว้นแต่จะใช้ `--force`
6. Branch เดียวกันไม่สามารถถูก checkout พร้อมกันในสอง worktree ได้ เพราะ branch คือ pointer เดียวที่ชี้ไปยัง commit จุดเดียวเท่านั้น วิธีแก้คือสร้าง branch ใหม่ ใช้ detached HEAD หรือลบ worktree เดิมก่อน
7. Use case จริงที่ worktree ช่วยได้มากคือการรัน test suite คู่ขนานกับการพัฒนา, การทำ hotfix ด่วนโดยไม่ทิ้งงานที่ทำค้าง, การ review Pull Request, และการ build หลาย release line พร้อมกัน
8. `git worktree prune` ใช้ทำความสะอาด metadata ของ worktree ที่โฟลเดอร์จริงถูกลบทิ้งไปด้วยมือแล้ว แต่ควรใช้ `git worktree remove` เป็นวิธีหลักเสมอ เพราะมันตรวจสอบความปลอดภัยให้ก่อนลบ
9. Dependencies อย่าง `node_modules/`, ไฟล์ `.env`, และ build cache ไม่ถูกแชร์ระหว่าง worktree โดยอัตโนมัติ ต้อง install หรือคัดลอกแยกต่างหากในแต่ละโฟลเดอร์เอง

**ต่อไป:** [Part 65: Custom Git Commands และการเขียน Script ต่อยอด Git](./part-065-custom-git-commands.md)
