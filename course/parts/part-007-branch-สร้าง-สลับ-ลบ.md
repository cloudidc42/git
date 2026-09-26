# Part 07: Branch คืออะไร: สร้าง สลับ ลบ Branch

> **Step ในหลักสูตรนี้:** Step 61–70
> **เฟส:** 2 — ใช้คำสั่ง Git พื้นฐานได้คล่องในชีวิตประจำวัน
> **เป้าหมายของ Part นี้:** เข้าใจแนวคิดของ Branch อย่างถ่องแท้ในระดับกลไกจริง (ไม่ใช่แค่ท่องจำคำสั่ง) รู้จักคำสั่งสร้าง สลับ ลบ และเปลี่ยนชื่อ branch ทั้งแบบเก่า (`checkout`) และแบบใหม่ (`switch`) เข้าใจว่า HEAD คืออะไรจริง ๆ และ detached HEAD คืออันตรายแบบไหน พร้อมฝึกจำลองการพัฒนาฟีเจอร์คู่ขนานด้วยมือตัวเองจริง ๆ

---

## สารบัญของ Part นี้

- Step 61: Branch คืออะไรในเชิงแนวคิด (pointer ไม่ใช่การ copy โฟลเดอร์)
- Step 62: `git branch` — สร้าง, list, ดู branch ปัจจุบัน
- Step 63: `git switch` vs `git checkout` — สลับ branch แบบใหม่กับแบบเก่า
- Step 64: `git switch -c` / `git checkout -b` — สร้างและสลับพร้อมกัน
- Step 65: HEAD คืออะไรจริง ๆ และ Detached HEAD State อันตรายอย่างไร
- Step 66: ลบ Branch — `-d` (safe delete) กับ `-D` (force delete)
- Step 67: เปลี่ยนชื่อ Branch ด้วย `git branch -m`
- Step 68: ดูกราฟ Branch ด้วย `git log --graph --all --oneline --decorate`
- Step 69: Naming Convention เบื้องต้นของ Branch (feature/, bugfix/, hotfix/)
- Step 70: แบบฝึกหัด — จำลองพัฒนาฟีเจอร์คู่ขนาน สร้าง สลับ ลบ

---

## Step 61: Branch คืออะไรในเชิงแนวคิด (pointer ไม่ใช่การ copy โฟลเดอร์)

นี่คือแนวคิดที่สำคัญที่สุดข้อหนึ่งในหลักสูตรทั้งหมด และเป็นจุดที่มือใหม่เข้าใจผิดมากที่สุด

### ความเข้าใจผิดที่พบบ่อยที่สุด

หลายคนที่มาจากพื้นเพของ VCS รุ่นเก่า (เช่น SVN) หรือแม้แต่คนที่ไม่เคยใช้ VCS มาก่อนเลย มักจะจินตนาการว่า "การสร้าง branch" คือการ **copy โฟลเดอร์ทั้งหมดของโปรเจกต์** ไปไว้อีกที่หนึ่ง เหมือนการกด Ctrl+C แล้ว Ctrl+V โฟลเดอร์ `project` ไปเป็น `project-feature-login`

```
❌ ความเข้าใจผิด:

project/              project-feature-login/   (copy ทั้งหมด)
├── index.html   -->   ├── index.html
├── style.css          ├── style.css
├── app.js             ├── app.js
└── ...                 └── ...
```

**นี่ไม่ใช่วิธีที่ Git ทำงานเลย** และถ้าคิดแบบนี้ต่อไป คุณจะงงกับพฤติกรรมของ Git ไปตลอด

### ความจริง: Branch คือ "ป้ายชื่อ" ที่ชี้ไปยัง Commit เพียงตัวเดียว

ใน Git, **branch คือไฟล์เล็ก ๆ ไฟล์หนึ่ง** ที่เก็บแค่ **SHA-1 hash ของ commit ตัวหนึ่ง** เท่านั้น ไม่มีอะไรมากไปกว่านี้

ลองดูจริง ๆ ในเครื่องได้เลย — ทุก branch ใน Git ถูกเก็บเป็นไฟล์ธรรมดาอยู่ใน `.git/refs/heads/`

```bash
cd ~/git-course
mkdir part-07-branching
cd part-07-branching
git init
```

ผลลัพธ์จำลอง:

```
Initialized empty Git repository in /home/user/git-course/part-07-branching/.git/
```

ลองสร้างไฟล์และ commit สักครั้ง:

```bash
echo "hello branch" > readme.txt
git add readme.txt
git commit -m "Initial commit"
```

ผลลัพธ์จำลอง:

```
[main (root-commit) a1b2c3d] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 readme.txt
```

ตอนนี้ลองดูว่าไฟล์ branch จริง ๆ หน้าตาเป็นอย่างไร:

```bash
cat .git/refs/heads/main
```

ผลลัพธ์จำลอง (hash จริงจะต่างกันในเครื่องคุณ):

```
a1b2c3d4e5f6789012345678901234567890abcd
```

**เห็นไหมครับ** — นั่นคือทั้งหมดของสิ่งที่เรียกว่า "branch main" มันคือไฟล์ 1 บรรทัดที่มีแค่ hash ของ commit ล่าสุดอยู่ข้างใน ไม่มีการ copy โฟลเดอร์ ไม่มีการ copy ไฟล์ใด ๆ ทั้งสิ้น

### ไดอะแกรม: Branch คือ Pointer ไม่ใช่กล่องเก็บไฟล์

```
                  Commit History (เก็บอยู่ใน .git/objects/ ที่เดียว)

        C1 ──────▶ C2 ──────▶ C3
        (init)   (add css)  (add js)
                                ▲
                                │
                          ┌─────┴─────┐
                          │   main    │  ← ไฟล์ .git/refs/heads/main
                          │ (pointer) │     เก็บแค่ hash ของ C3
                          └───────────┘
```

เมื่อคุณสร้าง branch ใหม่ชื่อ `feature-login` จาก commit C3:

```
        C1 ──────▶ C2 ──────▶ C3
                                ▲ ▲
                                │ │
                    ┌───────────┘ └───────────┐
                    │                          │
              ┌─────┴─────┐            ┌───────┴────────┐
              │   main    │            │ feature-login  │
              │ (pointer) │            │   (pointer)    │
              └───────────┘            └────────────────┘
```

สังเกตว่า **ไม่มี commit ใหม่เกิดขึ้นเลย** ทั้ง `main` และ `feature-login` ชี้ไปที่ commit C3 ตัวเดียวกัน มันคือแค่การสร้างไฟล์เล็ก ๆ อีกไฟล์หนึ่งใน `.git/refs/heads/` ที่มีค่า hash เดียวกันกับ `main` เท่านั้นเอง — ใช้เวลาน้อยกว่า 1 มิลลิวินาที และไม่กินพื้นที่เพิ่มแม้แต่ byte เดียว (นอกจากไฟล์ text เล็ก ๆ นั้น)

นี่คือเหตุผลที่ Part 01 (Step 5) บอกว่า Git มี "Cheap Branching" — เพราะมันเป็นการสร้าง pointer ไม่ใช่การ copy ข้อมูล

### เมื่อไหร่ที่ branch ทั้งสองจะแยกออกจากกันจริง ๆ

Branch จะเริ่ม "แยกทาง" กันก็ต่อเมื่อคุณ commit ใหม่ลงไปใน branch ใด branch หนึ่ง

```
        C1 ──────▶ C2 ──────▶ C3 ──────▶ C4 (commit ใหม่)
                                ▲            ▲
                                │            │
                          ┌─────┴─────┐ ┌────┴───────────┐
                          │   main    │ │ feature-login  │
                          └───────────┘ └────────────────┘
```

พอคุณ commit ใหม่บน `feature-login` เป็น C4 ตัว pointer ของ `feature-login` จะขยับไปชี้ที่ C4 โดยอัตโนมัติ ในขณะที่ `main` ยังคงชี้ที่ C3 เหมือนเดิม — นี่คือกลไกทั้งหมดของการแตก branch

### สรุปแนวคิดสำคัญของ Step นี้

> **Branch ใน Git ไม่ใช่สำเนาของโปรเจกต์ มันคือ "ป้ายชื่อ" (pointer) เล็ก ๆ ที่ชี้ไปยัง commit ตัวหนึ่งในประวัติ เมื่อคุณ commit ใหม่บน branch ที่คุณกำลังยืนอยู่ ป้ายชื่อนั้นจะขยับไปชี้ commit ใหม่โดยอัตโนมัติ**

ความเข้าใจนี้จะทำให้ทุกอย่างที่เหลือใน Part นี้ (การสร้าง สลับ ลบ เปลี่ยนชื่อ) เข้าใจง่ายขึ้นมาก เพราะทั้งหมดคือการจัดการ "ไฟล์ป้ายชื่อ" เล็ก ๆ เหล่านี้เท่านั้นเอง

---

## Step 62: `git branch` — สร้าง, list, ดู branch ปัจจุบัน

คำสั่ง `git branch` คือคำสั่งหลักในการจัดการ branch แบบพื้นฐาน มีการใช้งานหลายรูปแบบ มาดูทีละแบบ

### 62.1 ดูรายการ branch ทั้งหมด (list)

```bash
git branch
```

ผลลัพธ์จำลอง (ในโปรเจกต์ที่มีแค่ branch เดียว):

```
* main
```

เครื่องหมาย **`*`** (asterisk) ข้างหน้าชื่อ branch บอกว่า **นี่คือ branch ที่คุณกำลังยืนอยู่ตอนนี้** (current branch) — สังเกตว่าตัวอักษรสีเขียวใน terminal จริงจะช่วยเน้นให้เห็นชัดขึ้นด้วย

### 62.2 สร้าง branch ใหม่ (แต่ยังไม่สลับไป)

```bash
git branch feature-login
```

คำสั่งนี้**ไม่มี output ใด ๆ** ถ้าทำงานสำเร็จ (Git มักจะเงียบเมื่อคำสั่งสำเร็จ) ลองดูรายการ branch อีกครั้ง:

```bash
git branch
```

ผลลัพธ์จำลอง:

```
  feature-login
* main
```

สังเกตว่า:
- `feature-login` ถูกสร้างขึ้นแล้ว แต่ **ไม่มี `*` นำหน้า** แปลว่าคุณยัง**ไม่ได้สลับไปที่นั่น** คุณยังคงยืนอยู่ที่ `main` เหมือนเดิม
- นี่คือจุดสำคัญ: `git branch <ชื่อ>` แค่ **สร้าง pointer ใหม่ชี้ไปที่ commit ปัจจุบัน** แล้วก็จบ ไม่ได้พาคุณไปที่นั่นให้อัตโนมัติ

### 62.3 สร้างหลาย branch พร้อมกันเพื่อฝึก

```bash
git branch feature-signup
git branch bugfix-navbar
git branch experiment-dark-mode
```

```bash
git branch
```

ผลลัพธ์จำลอง:

```
  bugfix-navbar
  experiment-dark-mode
  feature-login
  feature-signup
* main
```

สังเกตว่า Git จะเรียง branch ตามลำดับตัวอักษร (alphabetical order) ไม่ใช่ตามลำดับที่สร้าง

### 62.4 ดูรายละเอียดเพิ่มเติมด้วย `-v` (verbose)

```bash
git branch -v
```

ผลลัพธ์จำลอง:

```
  bugfix-navbar          a1b2c3d Initial commit
  experiment-dark-mode   a1b2c3d Initial commit
  feature-login          a1b2c3d Initial commit
  feature-signup         a1b2c3d Initial commit
* main                   a1b2c3d Initial commit
```

`-v` จะโชว์ **short hash** และ **ข้อความ commit ล่าสุด** ของแต่ละ branch ด้วย — มีประโยชน์มากเวลาต้องการเช็คว่าแต่ละ branch อยู่ที่ commit ไหน โดยไม่ต้องสลับไปดูทีละอัน

### 62.5 ดูว่า branch ไหน merge เข้ากับ branch ปัจจุบันแล้วบ้าง

```bash
git branch --merged
```

ผลลัพธ์จำลอง (ตอนนี้ทุก branch ยังชี้ commit เดียวกันกับ main เพราะยังไม่มีการ commit แยก):

```
  bugfix-navbar
  experiment-dark-mode
  feature-login
  feature-signup
* main
```

และดู branch ที่ **ยังไม่ได้ merge**:

```bash
git branch --no-merged
```

ผลลัพธ์จำลอง:

```
(ไม่มี output เพราะทุก branch merge แล้วในสถานะปัจจุบัน)
```

คำสั่งสองตัวนี้จะมีประโยชน์มากใน Step 66 ตอนที่เราพูดถึงการลบ branch แบบปลอดภัย

### 62.6 ดูเฉพาะชื่อ branch ปัจจุบัน (แบบสั้น ไม่ต้องกรองด้วยตา)

```bash
git branch --show-current
```

ผลลัพธ์จำลอง:

```
main
```

คำสั่งนี้มีประโยชน์มากเวลาเขียนสคริปต์อัตโนมัติที่ต้องรู้ชื่อ branch ปัจจุบันแบบไม่มีสัญลักษณ์ `*` ปนมาด้วย

### ตารางสรุปคำสั่ง `git branch` เบื้องต้น

| คำสั่ง | ความหมาย |
|---|---|
| `git branch` | list ทุก branch พร้อมทำเครื่องหมาย `*` ที่ branch ปัจจุบัน |
| `git branch <ชื่อ>` | สร้าง branch ใหม่ (ไม่สลับไป) |
| `git branch -v` | list พร้อม short hash และข้อความ commit ล่าสุด |
| `git branch --show-current` | โชว์ชื่อ branch ปัจจุบันแบบสั้น (ไม่มี `*`) |
| `git branch --merged` | list branch ที่ merge เข้ากับปัจจุบันแล้ว |
| `git branch --no-merged` | list branch ที่ยังไม่ merge |
| `git branch -a` | list ทั้ง local branch และ remote-tracking branch (จะเจาะลึกใน Part ว่าด้วย remote) |

---

## Step 63: `git switch` vs `git checkout` — สลับ branch แบบใหม่กับแบบเก่า

การ**สร้าง** branch กับการ**สลับไปทำงาน**ที่ branch นั้นเป็นคนละขั้นตอนกัน ใน Step นี้เราจะมาดูวิธีสลับ

### ปัญหาทางประวัติศาสตร์ของ `git checkout`

ในอดีต Git มีคำสั่งเดียวชื่อ `git checkout` ที่ทำหน้าที่หลายอย่างปนกันมาก เช่น:

- สลับ branch (`git checkout main`)
- สร้างและสลับ branch พร้อมกัน (`git checkout -b feature-x`)
- กู้คืนไฟล์กลับไปเป็นเวอร์ชันใน commit ก่อนหน้า (`git checkout -- file.txt`)
- ย้ายไปดู commit เก่า ๆ โดยตรง (`git checkout a1b2c3d`)

การที่คำสั่งเดียวทำได้หลายอย่างมากขนาดนี้ทำให้มือใหม่สับสนบ่อยมาก และเสี่ยงพิมพ์ผิดจนทำข้อมูลหายโดยไม่ตั้งใจ (เช่น สับสนระหว่างการ "สลับ branch" กับ "ทิ้งการแก้ไขไฟล์ทิ้งไปเลย")

ตั้งแต่ **Git เวอร์ชัน 2.23 (ปี 2019)** เป็นต้นมา ทีมพัฒนา Git จึงแยกหน้าที่ออกเป็นคำสั่งใหม่ 2 ตัวที่ชัดเจนกว่า:

- **`git switch`** — ใช้สำหรับสลับ/สร้าง branch **เท่านั้น**
- **`git restore`** — ใช้สำหรับกู้คืนไฟล์ **เท่านั้น** (จะเรียนละเอียดใน Part ที่ว่าด้วยการย้อนกลับการเปลี่ยนแปลง)

`git checkout` แบบเก่ายังคงใช้งานได้อยู่และจะไม่ถูกเอาออกจาก Git (เพื่อความเข้ากันได้กับสคริปต์เก่า) แต่ **หลักสูตรนี้แนะนำให้ใช้ `git switch` และ `git restore` เป็นหลักในโค้ดใหม่ทุกกรณี** เพราะปลอดภัยกว่าและสื่อความหมายชัดเจนกว่ามาก

### 63.1 สลับ branch ด้วย `git switch` (แนะนำ — วิธีสมัยใหม่)

```bash
git switch feature-login
```

ผลลัพธ์จำลอง:

```
Switched to branch 'feature-login'
```

ตรวจสอบว่าอยู่ที่ branch ไหนแล้ว:

```bash
git branch
```

ผลลัพธ์จำลอง:

```
  bugfix-navbar
  experiment-dark-mode
* feature-login
  feature-signup
  main
```

เห็น `*` ย้ายมาอยู่ที่ `feature-login` แล้ว

### 63.2 สลับกลับด้วย `git switch` เช่นกัน

```bash
git switch main
```

ผลลัพธ์จำลอง:

```
Switched to branch 'main'
```

### 63.3 สลับ branch ด้วย `git checkout` (แบบเก่า — ยังพบเห็นได้บ่อยในโค้ดเก่า/บทความเก่า)

```bash
git checkout feature-signup
```

ผลลัพธ์จำลอง:

```
Switched to branch 'feature-signup'
```

สังเกตว่า **output เหมือนกันทุกประการ** กับ `git switch` เพราะเบื้องหลังมันทำสิ่งเดียวกัน แค่ `checkout` เป็นคำสั่งรุ่นเก่าที่ยังใช้ได้อยู่

### 63.4 เกิดอะไรขึ้นจริง ๆ เวลาสลับ branch (เชื่อมโยงกับ Step 61)

เมื่อคุณสั่ง `git switch feature-login` Git จะทำ 3 อย่างพร้อมกัน:

1. **ย้าย HEAD** ให้ไปชี้ที่ branch `feature-login` แทน `main` (จะอธิบายละเอียดใน Step 65)
2. **เขียนทับไฟล์ใน Working Directory** ให้ตรงกับ snapshot ของ commit ที่ `feature-login` ชี้อยู่
3. อัปเดต **Staging Area** ให้ตรงกับ snapshot นั้นด้วย

```
ก่อนสลับ:                          หลังสลับ (git switch feature-login):

HEAD ──▶ main ──▶ C3                HEAD ──▶ feature-login ──▶ C3
Working Dir = เนื้อหาจาก C3          Working Dir = เนื้อหาจาก C3 (เหมือนกัน เพราะยังไม่แยกกัน)
```

ในตัวอย่างของเรา เนื้อหาไฟล์จะเหมือนเดิมเป๊ะ ๆ เพราะทุก branch ยังชี้ไปที่ commit เดียวกัน (C3 หรือในตัวอย่างจริงคือ commit initial ของเรา) — ความแตกต่างจะเห็นชัดก็ต่อเมื่อแต่ละ branch มี commit ของตัวเองแล้ว ซึ่งเราจะฝึกใน Step 70

### 63.5 คำเตือนสำคัญ: สลับ branch ตอนที่มีไฟล์แก้ไขค้างอยู่

ลองจำลองสถานการณ์นี้:

```bash
git switch main
echo "แก้ไขบางอย่าง" >> readme.txt
git switch feature-login
```

ผลลัพธ์จำลอง (ถ้าการแก้ไขนั้นชนกับสิ่งที่ branch ปลายทางต้องการ):

```
error: Your local changes to the following files would be overwritten by checkout:
        readme.txt
Please commit your changes or stash them before you switch branches.
Aborting
```

Git **ปฏิเสธที่จะสลับ branch** ถ้าการสลับนั้นจะทำให้การแก้ไขที่ยังไม่ได้ commit ของคุณหายไปโดยไม่มีทางกู้คืน นี่คือกลไกความปลอดภัยของ Git — มันจะไม่ยอมทำลายงานของคุณเงียบ ๆ เด็ดขาด

ถ้าเจอสถานการณ์นี้ คุณมีทางเลือก:
1. `git commit` งานที่แก้ไว้ก่อน
2. `git stash` เก็บงานไว้ชั่วคราว (จะเรียนละเอียดใน Part ที่ว่าด้วย stash)
3. `git switch --discard-changes` หรือ `git switch -f` (force) ถ้ายอมทิ้งการแก้ไขนั้นจริง ๆ (ควรใช้อย่างระมัดระวังมาก)

### ตารางเปรียบเทียบ `git switch` กับ `git checkout`

| แง่มุม | `git switch` | `git checkout` |
|---|---|---|
| เปิดตัวเมื่อไหร่ | Git 2.23 (2019) | มีมาตั้งแต่ยุคแรกของ Git |
| หน้าที่ | สลับ/สร้าง branch เท่านั้น | สลับ branch + กู้คืนไฟล์ + ดู commit เก่า (หลายหน้าที่ปนกัน) |
| ความชัดเจน | ชัดเจน สื่อความหมายตรงตัว | คลุมเครือ เสี่ยงสับสน |
| แนะนำให้ใช้ในโค้ดใหม่ | ใช่ (แนะนำหลัก) | ใช้ได้ แต่ไม่แนะนำสำหรับมือใหม่ |
| พบในบทความ/สคริปต์เก่า | น้อยกว่า (คำสั่งใหม่กว่า) | พบบ่อยมาก เพราะมีมานาน |

> **ข้อแนะนำของหลักสูตรนี้:** ให้ใช้ `git switch` เป็นหลักตลอดหลักสูตร แต่ **ต้องอ่าน `git checkout` ออกด้วย** เพราะคุณจะเจอมันในเอกสารเก่า, Stack Overflow เก่า, และโค้ดของเพื่อนร่วมทีมที่คุ้นเคยกับคำสั่งเดิมอยู่แน่นอน

---

## Step 64: `git switch -c` / `git checkout -b` — สร้างและสลับ branch พร้อมกัน

ใน Step 62 เราสร้าง branch ด้วย `git branch <ชื่อ>` แล้วต้องมาสลับด้วย `git switch <ชื่อ>` อีกที ทำเป็น 2 ขั้นตอนแยกกัน แต่ในชีวิตจริง **99% ของเวลาที่คุณสร้าง branch ใหม่ คุณต้องการสลับไปทำงานที่นั่นทันที** จึงมีคำสั่งลัดที่รวม 2 ขั้นตอนนี้เป็นคำสั่งเดียว

### 64.1 วิธีใหม่: `git switch -c`

`-c` ย่อมาจาก **create**

```bash
git switch -c feature-payment
```

ผลลัพธ์จำลอง:

```
Switched to a new branch 'feature-payment'
```

สังเกตข้อความ **"a new branch"** ต่างจากตอนสลับ branch ที่มีอยู่แล้ว (ซึ่งจะขึ้นแค่ "Switched to branch") — Git แยกข้อความให้ชัดเจนว่าคุณเพิ่งสร้าง branch ใหม่หรือแค่ย้ายไปที่ branch เดิม

ตรวจสอบ:

```bash
git branch --show-current
```

ผลลัพธ์จำลอง:

```
feature-payment
```

### 64.2 วิธีเก่า: `git checkout -b`

```bash
git checkout -b bugfix-footer
```

ผลลัพธ์จำลอง:

```
Switched to a new branch 'bugfix-footer'
```

`-b` ในที่นี้ก็ย่อมาจาก **branch** — ทำหน้าที่เหมือนกับ `git switch -c` ทุกประการ เพียงแต่เป็นคำสั่งรุ่นเก่ากว่า

### 64.3 สร้าง branch ใหม่จาก commit หรือ branch อื่นที่ไม่ใช่ปัจจุบัน

คุณไม่จำเป็นต้องสร้าง branch ใหม่จากตำแหน่งที่ยืนอยู่เท่านั้น สามารถระบุจุดเริ่มต้นได้ชัดเจน:

```bash
git switch main
git switch -c feature-report main
```

ในกรณีนี้เพราะเรายืนอยู่ที่ `main` อยู่แล้ว ผลลัพธ์จะเหมือนไม่ระบุ แต่ถ้าคุณยืนอยู่ที่ branch อื่น การระบุปลายทางชัดเจนแบบนี้จะช่วยป้องกันความผิดพลาดได้มาก ตัวอย่างเช่นถ้าคุณกำลังยืนอยู่ที่ `feature-login` แต่ต้องการแตก branch ใหม่จาก `main` โดยไม่ต้องสลับไป `main` ก่อน:

```bash
git switch -c feature-checkout main
```

ผลลัพธ์จำลอง:

```
Switched to a new branch 'feature-checkout'
```

Git จะสร้าง `feature-checkout` จาก commit ที่ `main` ชี้อยู่ทันที โดยไม่สนใจว่าตอนนั้นคุณยืนอยู่ที่ branch ไหน — สะดวกมากในการทำงานจริง

รูปแบบเดียวกันด้วยคำสั่งเก่า:

```bash
git checkout -b feature-checkout main
```

### 64.4 ระวังชื่อ branch ซ้ำ

ถ้า branch ชื่อนั้นมีอยู่แล้ว Git จะปฏิเสธทันที:

```bash
git switch -c feature-login
```

ผลลัพธ์จำลอง:

```
fatal: a branch named 'feature-login' already exists
```

ถ้าต้องการสร้างทับ (force) พร้อมย้าย pointer ไปยังตำแหน่งใหม่ ใช้ `-C` (ตัวใหญ่) แทน `-c`:

```bash
git switch -C feature-login
```

ผลลัพธ์จำลอง:

```
Switched to and reset branch 'feature-login'
```

**ควรใช้ `-C` ด้วยความระมัดระวังมาก** เพราะมันจะย้าย pointer ของ branch นั้นไปยังตำแหน่งปัจจุบันทันที ถ้า branch เดิมมี commit ที่ยังไม่ได้ merge อยู่และไม่มี branch อื่นชี้ถึง commit เหล่านั้นอีก มันอาจทำให้ commit เหล่านั้นเข้าถึงยากขึ้นมาก (แม้จะยังไม่หายไปจริง ๆ ในทันที)

### ตารางสรุปคำสั่งสร้าง+สลับพร้อมกัน

| คำสั่ง | ความหมาย |
|---|---|
| `git switch -c <ชื่อ>` | สร้าง branch ใหม่จากตำแหน่งปัจจุบันแล้วสลับไปทันที (แนะนำ) |
| `git switch -c <ชื่อ> <จุดเริ่มต้น>` | สร้าง branch ใหม่จาก branch/commit ที่ระบุแล้วสลับไป |
| `git switch -C <ชื่อ>` | สร้างทับ (force) แม้ชื่อนั้นมีอยู่แล้ว — ใช้ระมัดระวัง |
| `git checkout -b <ชื่อ>` | เหมือน `switch -c` แต่เป็นคำสั่งรุ่นเก่า |
| `git checkout -b <ชื่อ> <จุดเริ่มต้น>` | เหมือน `switch -c <ชื่อ> <จุดเริ่มต้น>` แบบเก่า |

---

## Step 65: HEAD คืออะไรจริง ๆ และ Detached HEAD State อันตรายอย่างไร

Step 8 ของ Part 01 แนะนำคำว่า **HEAD** แบบผิว ๆ ไว้ว่า "ตัวชี้ไปยังตำแหน่งปัจจุบันที่คุณกำลังทำงานอยู่" ตอนนี้ถึงเวลาเจาะลึกกลไกจริงของมันแล้ว

### 65.1 HEAD คือ pointer ที่ชี้ไปยัง branch ไม่ใช่ชี้ไปยัง commit โดยตรง

นี่คือจุดที่คนมักเข้าใจผิด — หลายคนคิดว่า HEAD ชี้ไปที่ commit ตรง ๆ แต่ในสถานการณ์ปกติ **HEAD ชี้ไปที่ branch ซึ่ง branch นั้นถึงจะชี้ไปที่ commit อีกที** — เรียกว่า **symbolic reference**

ลองดูของจริง:

```bash
git switch main
cat .git/HEAD
```

ผลลัพธ์จำลอง:

```
ref: refs/heads/main
```

เห็นไหมครับ — ไฟล์ `.git/HEAD` **ไม่ได้เก็บ hash ของ commit โดยตรง** แต่เก็บข้อความว่า "ให้ไปดูที่ `refs/heads/main`" ซึ่งไฟล์นั้นถึงจะเก็บ hash ของ commit จริง ๆ อีกที

```
HEAD ──(symbolic ref)──▶ refs/heads/main ──(hash)──▶ Commit C3
```

นี่คือเหตุผลที่เวลาคุณ `commit` บน branch ปกติ ทั้ง 2 อย่างจะขยับตามกันโดยอัตโนมัติ:

1. คุณ commit → Git สร้าง commit ใหม่ (สมมติ C4)
2. Git อัปเดต `refs/heads/main` ให้ชี้ C4 แทน C3
3. HEAD ยังคงชี้ไปที่ `refs/heads/main` เหมือนเดิม (ไม่ต้องแก้ไข HEAD เลย) — แต่เพราะ `refs/heads/main` เปลี่ยนไปชี้ C4 แล้ว ผลคือ HEAD "ตามไปที่ C4 โดยอัตโนมัติ"

### 65.2 Detached HEAD State คืออะไร

**Detached HEAD** เกิดขึ้นเมื่อ HEAD **ชี้ตรงไปยัง commit hash โดยตรง แทนที่จะชี้ผ่าน branch**

```
สถานะปกติ:                          Detached HEAD:

HEAD ──▶ refs/heads/main ──▶ C3      HEAD ──────────────────────▶ C2
                                      (ไม่มี branch ใดเกี่ยวข้องเลย)
```

สถานการณ์ที่ทำให้เกิด detached HEAD บ่อยที่สุดคือการ checkout/switch ไปที่ commit hash โดยตรง แทนที่จะไปที่ชื่อ branch:

```bash
git log --oneline
```

ผลลัพธ์จำลอง:

```
c3d4e5f (HEAD -> main) Add feature C
b2c3d4e Add feature B
a1b2c3d Initial commit
```

ลอง checkout ไปที่ commit เก่าโดยตรงด้วย hash:

```bash
git checkout a1b2c3d
```

ผลลัพธ์จำลอง:

```
Note: switching to 'a1b2c3d'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at a1b2c3d Initial commit
```

Git **เตือนอย่างชัดเจนมาก** ว่าตอนนี้คุณอยู่ใน detached HEAD state แล้ว และแนะนำวิธีแก้ไขให้ในข้อความเลย นี่เป็นตัวอย่างที่ดีมากของ Git ที่พยายามป้องกันไม่ให้ผู้ใช้พลาดโดยไม่รู้ตัว

ตรวจสอบด้วย `git branch`:

```bash
git branch
```

ผลลัพธ์จำลอง:

```
* (HEAD detached at a1b2c3d)
  bugfix-footer
  feature-login
  feature-payment
  main
```

สังเกตว่าไม่มี branch ไหนมี `*` นำหน้าเลย มีแต่ข้อความพิเศษ `(HEAD detached at a1b2c3d)` แสดงว่าตอนนี้คุณ**ไม่ได้ยืนอยู่บน branch ใด ๆ ทั้งสิ้น**

ใช้ `git switch` ในเวอร์ชันใหม่ ก็จะเจอคำเตือนแบบเดียวกัน:

```bash
git switch a1b2c3d
```

ผลลัพธ์จำลอง:

```
Note: switching to 'a1b2c3d'.

You are in 'detached HEAD' state. ...
HEAD is now at a1b2c3d Initial commit
```

### 65.3 ทำไม Detached HEAD ถึงอันตราย

อันตรายที่แท้จริงเกิดขึ้นถ้าคุณ **commit ใหม่ในขณะที่อยู่ใน detached HEAD** โดยไม่รู้ตัว

```bash
echo "งานทดลอง" > experiment.txt
git add experiment.txt
git commit -m "ทดลองบางอย่าง"
```

ผลลัพธ์จำลอง:

```
[detached HEAD d4e5f6a] ทดลองบางอย่าง
 1 file changed, 1 insertion(+)
 create mode 100644 experiment.txt
```

ตอนนี้ commit ใหม่ `d4e5f6a` ถูกสร้างขึ้นจริง **แต่ไม่มี branch ใดชี้ไปที่มันเลย**

```
                          d4e5f6a (commit ใหม่ — "ลอยอยู่")
                          ▲
                          │
                          a1b2c3d ──▶ b2c3d4e ──▶ c3d4e5f
                                                    ▲
                                              ┌─────┴─────┐
                                              │   main    │
                                              └───────────┘
```

ถ้าคุณสลับกลับไปที่ `main` ตอนนี้:

```bash
git switch main
```

ผลลัพธ์จำลอง:

```
Warning: you are leaving 1 commit behind, not connected to
any of your branches:

  d4e5f6a ทดลองบางอย่าง

If you want to keep it by creating a new branch, this may be a good time
to do so with:

 git branch <new-branch-name> d4e5f6a

Switched to branch 'main'
```

Git เตือนอีกครั้งว่ากำลัง "ทิ้ง" commit ที่ไม่มี branch อ้างถึงไว้ข้างหลัง commit ตัวนี้**ยังไม่หายไปทันที** (มันยังอยู่ใน `.git/objects/` และจะถูกเก็บไว้อีกระยะหนึ่งผ่านกลไกที่เรียกว่า **reflog** ซึ่งจะเรียนใน Part ขั้นสูงกว่านี้) แต่ถ้าไม่มีอะไรอ้างอิงถึงมันเลยเป็นเวลานานพอ (ปกติ 30-90 วันตาม default ของ `git gc`) **มันจะถูกลบทิ้งอย่างถาวรโดย Garbage Collection ของ Git**

### 65.4 วิธีป้องกันและวิธีกู้คืนถ้าพลาด

**ป้องกัน:** ถ้าต้องการทดลองอะไรจาก commit เก่า ให้สร้าง branch ใหม่จากตรงนั้นเสมอแทนที่จะ checkout ตรงไปยัง hash เปล่า ๆ

```bash
git switch -c experiment-from-old a1b2c3d
```

วิธีนี้ปลอดภัย 100% เพราะตอนนี้มี branch ชื่อ `experiment-from-old` ชี้ไปยัง commit นั้นแล้ว commit ใหม่ที่คุณสร้างต่อจากนี้จะไม่มีทาง "ลอย" อย่างไม่มีใครอ้างอิงอีกต่อไป

**กู้คืนถ้าพลาดไปแล้ว:** ถ้าคุณ commit ไปใน detached HEAD แล้วเผลอสลับออกมาโดยยังไม่ได้สร้าง branch เก็บไว้ ยังพอกู้คืนได้ทันทีด้วย `git reflog` (Git จะจำ hash ล่าสุดที่ HEAD เคยชี้ไว้เสมอ):

```bash
git reflog
```

ผลลัพธ์จำลอง:

```
d4e5f6a HEAD@{0}: commit: ทดลองบางอย่าง
a1b2c3d HEAD@{1}: checkout: moving from main to a1b2c3d
c3d4e5f HEAD@{2}: checkout: moving from feature-payment to main
...
```

เจอ hash `d4e5f6a` แล้วก็สร้าง branch ใหม่ชี้ไปที่มันได้ทันที:

```bash
git branch rescued-experiment d4e5f6a
```

(รายละเอียดเชิงลึกของ `reflog` จะเรียนแบบเต็ม ๆ ใน Part ที่ว่าด้วยการกู้คืนข้อมูล)

### สรุปแนวคิดสำคัญของ Step นี้

> **HEAD ปกติคือ symbolic reference ที่ชี้ไปยัง branch (ไม่ใช่ commit โดยตรง) การ commit บน branch ปกติจะทำให้ branch นั้นขยับตาม HEAD โดยอัตโนมัติ แต่ถ้า HEAD ชี้ไปที่ commit hash ตรง ๆ (detached HEAD) การ commit ใหม่จะไม่มี branch ใดอ้างอิงถึง และเสี่ยงหายไปถาวรถ้าไม่รีบสร้าง branch เก็บไว้**

---

## Step 66: ลบ Branch — `-d` (safe delete) กับ `-D` (force delete)

เมื่อ branch เสร็จภารกิจแล้ว (merge เข้า main เรียบร้อยแล้ว) ควรลบทิ้งเพื่อให้ repository สะอาด ไม่รกด้วย branch เก่า ๆ ที่ไม่ได้ใช้แล้ว

### 66.1 ลบแบบปลอดภัย: `-d` (lowercase, safe delete)

```bash
git switch main
git branch -d bugfix-footer
```

ผลลัพธ์จำลอง (ถ้า branch นี้ยังไม่เคย merge เข้า main และมี commit ที่ยังไม่ถูกรวมเข้าที่ไหนเลย):

```
error: The branch 'bugfix-footer' is not fully merged.
If you are sure you want to delete it, run 'git branch -D bugfix-footer'.
```

**นี่คือกลไกความปลอดภัยสำคัญของ `-d`** — Git จะตรวจสอบก่อนว่า commit ทั้งหมดใน branch นี้ **ถูก merge เข้าไปใน branch ปัจจุบัน (หรือ upstream ของมัน) แล้วหรือยัง** ถ้ายัง มันจะปฏิเสธการลบทันที เพื่อป้องกันไม่ให้คุณลบ branch ที่มีงานสำคัญที่ยังไม่ได้ถูกเก็บไว้ที่ไหนเลย

ลองสร้างสถานการณ์ที่ merge สำเร็จก่อน แล้วค่อยลบ:

```bash
git switch feature-login
echo "login feature done" > login.txt
git add login.txt
git commit -m "Implement login"
git switch main
git merge feature-login
```

ผลลัพธ์จำลอง:

```
Updating a1b2c3d..e5f6a7b
Fast-forward
 login.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 login.txt
```

(รายละเอียดของ `git merge` และ conflict จะเรียนเต็ม ๆ ใน **Part 08**) ตอนนี้ `main` มี commit ของ `feature-login` รวมอยู่แล้ว จึงลบได้อย่างปลอดภัย:

```bash
git branch -d feature-login
```

ผลลัพธ์จำลอง:

```
Deleted branch feature-login (was e5f6a7b).
```

สังเกตว่า Git บอก hash ของ commit สุดท้ายที่ branch นั้นเคยชี้ไว้ด้วย (`e5f6a7b`) — มีประโยชน์มากถ้าต้องการกู้คืนภายหลัง (ผ่าน `git branch <ชื่อ> e5f6a7b` หรือ `git reflog`)

### 66.2 ลบแบบบังคับ: `-D` (uppercase, force delete)

บางครั้งคุณต้องการลบ branch ทดลองที่ **ตั้งใจจะทิ้งไปเลยโดยไม่สนใจว่า merge แล้วหรือยัง** เช่น branch ทดลองที่ทำแล้วพบว่าแนวทางนั้นใช้ไม่ได้

```bash
git branch -D experiment-dark-mode
```

ผลลัพธ์จำลอง:

```
Deleted branch experiment-dark-mode (was a1b2c3d).
```

`-D` คือคำย่อของ `--delete --force` — มันจะ**ลบทันทีโดยไม่ตรวจสอบสถานะ merge เลย** ควรใช้ด้วยความระมัดระวังสูงมาก เพราะถ้า branch นั้นมี commit ที่ยังไม่ได้ merge ที่ไหนเลย และคุณจำ hash ไม่ได้ การกู้คืนจะยากขึ้นมาก (ต้องพึ่ง `git reflog` ซึ่งก็มีอายุจำกัด)

### 66.3 ทำไม Git ถึงป้องกันแค่ตอน `-d` แต่ไม่ป้องกันตอน `-D`

นี่คือปรัชญาการออกแบบของ Git: เครื่องมือให้ทั้งเส้นทาง "ปลอดภัย" (`-d`) และเส้นทาง "ฉันรู้ว่าฉันทำอะไรอยู่" (`-D`) ไว้ให้ผู้ใช้เลือกเอง Git ไม่ได้ห้ามคุณทำสิ่งอันตราย แต่จะ**เตือนก่อนเสมอในเส้นทางปกติ** และเปิดทางให้ทำสิ่งอันตรายได้ถ้าคุณยืนยันชัดเจนผ่านการพิมพ์ flag ที่ต่างออกไป (ตัวใหญ่แทนตัวเล็ก)

### 66.4 พยายามลบ branch ที่กำลังยืนอยู่

ลองสั่ง:

```bash
git switch feature-signup
git branch -d feature-signup
```

ผลลัพธ์จำลอง:

```
error: Cannot delete branch 'feature-signup' checked out at '/home/user/git-course/part-07-branching'
```

**Git ไม่ยอมให้คุณลบ branch ที่ตัวเองกำลังยืนอยู่เด็ดขาด** ไม่ว่าจะใช้ `-d` หรือ `-D` ก็ตาม เพราะนั่นจะทำให้ HEAD ชี้ไปยังสิ่งที่ไม่มีอยู่จริง คุณต้อง `git switch` ไปที่ branch อื่นก่อนเสมอ:

```bash
git switch main
git branch -d feature-signup
```

ผลลัพธ์จำลอง:

```
Deleted branch feature-signup (was a1b2c3d).
```

### 66.5 ลบหลาย branch พร้อมกัน

`git branch -d` รับชื่อ branch หลายตัวพร้อมกันได้:

```bash
git branch -d bugfix-navbar rescued-experiment
```

ผลลัพธ์จำลอง:

```
Deleted branch bugfix-navbar (was a1b2c3d).
Deleted branch rescued-experiment (was d4e5f6a).
```

### ตารางสรุปการลบ branch

| คำสั่ง | พฤติกรรม | ใช้เมื่อไหร่ |
|---|---|---|
| `git branch -d <ชื่อ>` | ลบแบบปลอดภัย ปฏิเสธถ้ายังไม่ merge | กรณีปกติทั่วไป เมื่องาน merge เสร็จแล้ว |
| `git branch -D <ชื่อ>` | ลบแบบบังคับ ไม่สนใจสถานะ merge | branch ทดลองที่ตั้งใจทิ้ง หรือแน่ใจแล้วว่าไม่ต้องการ |
| `git branch -d/-D <ชื่อ1> <ชื่อ2> ...` | ลบหลาย branch พร้อมกัน | ทำความสะอาด repo ครั้งใหญ่ |
| (ลบ branch ที่ยืนอยู่) | Git ปฏิเสธเสมอ | ต้อง `switch` ไปที่อื่นก่อนเสมอ |

---

## Step 67: เปลี่ยนชื่อ Branch ด้วย `git branch -m`

บางครั้งคุณตั้งชื่อ branch ผิด พิมพ์ผิด หรือต้องการเปลี่ยนชื่อให้สื่อความหมายมากขึ้นภายหลัง คำสั่งที่ใช้คือ `git branch -m` (m ย่อมาจาก **move**, ในทางปฏิบัติหมายถึง rename)

### 67.1 เปลี่ยนชื่อ branch ปัจจุบันที่ยืนอยู่

```bash
git switch feature-payment
git branch -m feature-checkout-payment
```

ผลลัพธ์จำลอง: **ไม่มี output** ถ้าสำเร็จ (Git มักเงียบเมื่อสำเร็จตามที่กล่าวไปก่อนหน้านี้)

ตรวจสอบผล:

```bash
git branch --show-current
```

ผลลัพธ์จำลอง:

```
feature-checkout-payment
```

### 67.2 เปลี่ยนชื่อ branch อื่นที่ไม่ใช่ branch ปัจจุบัน

ระบุทั้งชื่อเก่าและชื่อใหม่:

```bash
git branch -m feature-report feature-monthly-report
```

ตรวจสอบ:

```bash
git branch
```

ผลลัพธ์จำลอง:

```
  bugfix-footer
* feature-checkout-payment
  feature-monthly-report
  main
```

รูปแบบคือ `git branch -m <ชื่อเก่า> <ชื่อใหม่>` — ถ้าไม่ระบุ `<ชื่อเก่า>` Git จะเข้าใจว่าหมายถึง branch ปัจจุบัน (ตามตัวอย่าง 67.1)

### 67.3 เปลี่ยนชื่อทับชื่อที่มีอยู่แล้ว (force rename)

ถ้าต้องการเปลี่ยนชื่อไปเป็นชื่อที่มี branch อยู่แล้ว (ซึ่งปกติ Git จะปฏิเสธ) ใช้ `-M` (ตัวใหญ่):

```bash
git branch -M feature-checkout-payment main
```

**คำเตือนสำคัญมาก:** คำสั่งด้านบนนี้อันตรายมาก เพราะมันจะ **ลบ branch `main` เดิมทิ้งแล้วเอา `feature-checkout-payment` ไปแทนที่ชื่อ `main`** ควรตรวจสอบให้แน่ใจเสมอก่อนใช้ `-M` — ในตัวอย่างจริงข้างต้นนี้เป็นเพียงการสาธิต ไม่ควรทำจริงถ้าไม่ได้ตั้งใจ

หมายเหตุ: คำสั่ง `git branch -M main` (ไม่มีชื่อเก่าระบุ, เปลี่ยนชื่อ branch ปัจจุบันเป็น `main`) เป็นคำสั่งที่พบบ่อยมากตอน `git init` repository ใหม่ เพราะ Git เวอร์ชันเก่าตั้งชื่อ branch แรกเป็น `master` โดย default แต่หลายองค์กร/GitHub เปลี่ยนมาตรฐานเป็น `main` แล้ว คำสั่งนี้เองที่ GitHub แนะนำให้รันตอนสร้าง repo ใหม่:

```bash
git branch -M main
```

### 67.4 ผลกระทบของการเปลี่ยนชื่อ branch ต่อ remote

สิ่งสำคัญที่ต้องรู้ (แม้จะยังไม่ได้เรียน remote เต็มรูปแบบจนถึง Part ในเฟสถัดไป): **การเปลี่ยนชื่อ branch ในเครื่อง (local) ด้วย `git branch -m` จะไม่กระทบชื่อ branch บน remote (เช่น GitHub) โดยอัตโนมัติ** ถ้า branch นั้นเคย push ขึ้น remote ไปแล้ว คุณต้อง push branch ใหม่และลบ branch เก่าบน remote แยกต่างหาก ซึ่งเราจะพูดถึงรายละเอียดเต็ม ๆ ใน Part ที่ว่าด้วยการทำงานกับ remote branch

### ตารางสรุปการเปลี่ยนชื่อ branch

| คำสั่ง | ความหมาย |
|---|---|
| `git branch -m <ชื่อใหม่>` | เปลี่ยนชื่อ branch **ปัจจุบัน** เป็นชื่อใหม่ |
| `git branch -m <ชื่อเก่า> <ชื่อใหม่>` | เปลี่ยนชื่อ branch ที่ระบุ (ไม่ต้องยืนอยู่ที่นั่น) |
| `git branch -M <ชื่อใหม่>` | เปลี่ยนชื่อแบบบังคับ ทับชื่อที่มีอยู่แล้วได้ (ใช้ระวังมาก) |

---

## Step 68: ดูกราฟ Branch ด้วย `git log --graph --all --oneline --decorate`

การเข้าใจ branch ในทางทฤษฎีอย่างเดียวไม่พอ — สิ่งที่ทำให้เข้าใจได้จริงและติดเป็นนิสัยของโปรแกรมเมอร์มืออาชีพคือ **การมองเห็นกราฟของ commit และ branch ทั้งหมดด้วยตาตัวเอง**

### 68.1 คำสั่งพื้นฐาน `git log`

```bash
git log
```

ผลลัพธ์จำลอง (แบบเต็ม ยาวเกินไปสำหรับใช้งานประจำวัน):

```
commit e5f6a7b8c9d0123456789abcdef0123456789ab (HEAD -> main)
Author: Somchai Devnote <somchai@example.com>
Date:   Fri Sep 26 10:15:22 2026 +0700

    Implement login

commit a1b2c3d4e5f6789012345678901234567890abcd
Author: Somchai Devnote <somchai@example.com>
Date:   Fri Sep 26 09:00:00 2026 +0700

    Initial commit
```

รูปแบบนี้กินพื้นที่มากและอ่านยากเมื่อมีหลาย commit หลาย branch จึงต้องใช้ flag เสริม

### 68.2 คำสั่งทรงพลังที่โปรแกรมเมอร์ทุกคนควรจำขึ้นใจ

```bash
git log --graph --all --oneline --decorate
```

มาดูความหมายของแต่ละ flag ทีละตัว:

| Flag | ความหมาย |
|---|---|
| `--graph` | วาดเส้นกราฟ ASCII แสดงความสัมพันธ์ของ commit และ branch |
| `--all` | แสดง**ทุก branch** ไม่ใช่แค่ branch ปัจจุบัน |
| `--oneline` | แสดงแต่ละ commit ในบรรทัดเดียว (short hash + ข้อความ) |
| `--decorate` | แสดงชื่อ branch/tag ที่ชี้ไปยัง commit นั้น ๆ (ค่า default เปิดอยู่แล้วในหลายเวอร์ชัน แต่ระบุชัดเจนไว้เสมอเป็นนิสัยที่ดี) |

ลองสร้างสถานการณ์ที่มีหลาย branch แยกออกจากกันจริง เพื่อดูผลลัพธ์ที่ชัดเจน:

```bash
git switch main
git switch -c feature-search
echo "search feature" > search.txt
git add search.txt
git commit -m "Add search feature"

git switch main
git switch -c feature-cart
echo "cart feature" > cart.txt
git add cart.txt
git commit -m "Add cart feature"

git switch main
```

ตอนนี้ลองรันคำสั่งกราฟ:

```bash
git log --graph --all --oneline --decorate
```

ผลลัพธ์จำลอง:

```
* f1e2d3c (feature-cart) Add cart feature
| * a9b8c7d (feature-search) Add search feature
|/
* e5f6a7b (HEAD -> main, feature-checkout-payment) Implement login
* a1b2c3d Initial commit
```

มาอ่านกราฟนี้ทีละส่วน:

```
* f1e2d3c (feature-cart) Add cart feature      ← branch feature-cart แยกออกไปมี commit ของตัวเอง
| * a9b8c7d (feature-search) Add search feature ← branch feature-search ก็แยกออกไปเช่นกัน
|/                                               ← เส้นทั้งสองมาบรรจบกันที่จุดเดียวกัน (จุดที่แตก branch)
* e5f6a7b (HEAD -> main, ...) Implement login   ← จุดร่วมที่ทั้งสอง branch แตกออกไป, HEAD อยู่ตรงนี้ (main)
* a1b2c3d Initial commit                        ← จุดเริ่มต้นของประวัติทั้งหมด
```

นี่คือภาพที่ตรงกับไดอะแกรมแนวคิดใน Step 61 เป๊ะ ๆ — คุณกำลังเห็น **pointer หลายตัวชี้ไปยัง commit ต่าง ๆ กัน** ด้วยตาตัวเองในสภาพแวดล้อมจริงแล้ว

### 68.3 สร้าง alias เพื่อไม่ต้องพิมพ์คำสั่งยาว ๆ ซ้ำทุกครั้ง

เพราะคำสั่งนี้ยาวและถูกใช้บ่อยมากในชีวิตจริง โปรแกรมเมอร์เกือบทุกคนจะตั้ง Git alias ไว้:

```bash
git config --global alias.graph "log --graph --all --oneline --decorate"
```

หลังจากนั้นแค่พิมพ์:

```bash
git graph
```

ก็จะได้ผลลัพธ์เดียวกันทันที (การตั้งค่า Git config และ alias แบบละเอียดจะมี Part เฉพาะทางในเฟสต่อ ๆ ไป)

### 68.4 เพิ่มข้อมูลผู้เขียนและวันที่เข้าไปด้วย (ทางเลือกเสริม)

```bash
git log --graph --all --oneline --decorate --pretty=format:"%h %d %s (%an, %ar)"
```

ผลลัพธ์จำลอง:

```
* f1e2d3c  (feature-cart) Add cart feature (Somchai Devnote, 2 minutes ago)
| * a9b8c7d  (feature-search) Add search feature (Somchai Devnote, 3 minutes ago)
|/
* e5f6a7b  (HEAD -> main, feature-checkout-payment) Implement login (Somchai Devnote, 10 minutes ago)
* a1b2c3d  Initial commit (Somchai Devnote, 1 hour ago)
```

(รูปแบบ `--pretty=format` แบบเต็มจะมี Part เฉพาะเจาะจงเรื่องการปรับแต่ง `git log` ในเฟสถัดไป ตอนนี้แค่รู้ไว้ว่ามันปรับแต่งได้ลึกมากก็พอ)

---

## Step 69: Naming Convention เบื้องต้นของ Branch (feature/, bugfix/, hotfix/)

การตั้งชื่อ branch แบบสุ่มสี่สุ่มห้า (`test1`, `asdf`, `my-branch`) จะกลายเป็นปัญหาใหญ่มากเมื่อทีมโตขึ้นและมี branch เป็นสิบเป็นร้อยพร้อมกัน อุตสาหกรรมจึงมีธรรมเนียมการตั้งชื่อที่ใช้กันอย่างแพร่หลาย

### 69.1 รูปแบบ prefix ที่นิยมใช้มากที่สุด

| Prefix | ใช้เมื่อไหร่ | ตัวอย่าง |
|---|---|---|
| `feature/` | กำลังพัฒนาฟีเจอร์ใหม่ | `feature/user-login`, `feature/shopping-cart` |
| `bugfix/` | กำลังแก้ไขบั๊กที่ไม่เร่งด่วนมาก (พบระหว่างพัฒนาปกติ) | `bugfix/navbar-overlap`, `bugfix/typo-in-footer` |
| `hotfix/` | แก้บั๊กด่วนที่กระทบระบบ production ต้องรีบแก้และปล่อยทันที | `hotfix/payment-crash`, `hotfix/security-patch` |
| `release/` | เตรียม branch สำหรับเตรียมออกเวอร์ชันใหม่ | `release/v2.0.0` |
| `chore/` | งานที่ไม่ใช่ฟีเจอร์/บั๊กโดยตรง เช่น อัปเดต dependency, แก้ config | `chore/update-dependencies` |
| `docs/` | แก้ไขเฉพาะเอกสารประกอบ | `docs/update-readme` |

### 69.2 ตัวอย่างการใช้งานจริง

```bash
git switch -c feature/user-profile
git switch -c bugfix/fix-date-format
git switch -c hotfix/critical-login-bug
```

ผลลัพธ์จำลอง:

```
Switched to a new branch 'feature/user-profile'
```

สังเกตว่า **เครื่องหมาย `/` ในชื่อ branch ไม่ได้สร้างโฟลเดอร์จริงในโปรเจกต์** มันเป็นแค่ส่วนหนึ่งของชื่อ branch เท่านั้น แต่ Git และเครื่องมือ GUI หลายตัว (เช่น GitHub, GitKraken, VS Code) จะแสดงผล branch เหล่านี้แบบจัดกลุ่มเป็น "โฟลเดอร์เสมือน" ให้ดูเป็นระเบียบ

ลองดูผลลัพธ์ของ `git branch` เมื่อมี branch แบบมี prefix ปนกัน:

```bash
git branch
```

ผลลัพธ์จำลอง:

```
  bugfix/fix-date-format
  feature/user-profile
  feature-cart
  feature-search
  hotfix/critical-login-bug
* main
```

### 69.3 ทำไม Naming Convention ถึงสำคัญ

1. **ค้นหาง่าย** — เมื่อมี branch หลายร้อยตัว การกรองด้วย pattern เช่น `git branch --list "feature/*"` ทำให้เห็นเฉพาะ branch ฟีเจอร์ได้ทันที
2. **สื่อสารเจตนาชัดเจน** — เพื่อนร่วมทีมเห็นชื่อ branch แล้วรู้ทันทีว่ากำลังทำอะไรอยู่ โดยไม่ต้องถามหรือเปิดดูโค้ด
3. **เชื่อมกับระบบ automation ได้** — หลายทีมตั้งค่า CI/CD ให้ทำงานต่างกันตาม prefix ของ branch เช่น branch ที่ขึ้นต้นด้วย `hotfix/` อาจถูกตั้งให้ deploy อัตโนมัติเร็วกว่าปกติ
4. **เชื่อมโยงกับ Issue Tracker** — หลายทีมนิยมใส่เลข ticket ลงไปด้วย เช่น `feature/PROJ-123-user-login` เพื่อให้เชื่อมโยงกับระบบติดตามงานอย่าง Jira ได้อัตโนมัติ

### ตัวอย่างการกรองด้วย pattern

```bash
git branch --list "feature/*"
```

ผลลัพธ์จำลอง:

```
  feature/user-profile
```

### 69.4 กติกาที่ใช้จริงจะเจาะลึกใน Part 35

Naming convention ในเนื้อหานี้เป็นแค่ **ภาพรวมเบื้องต้น** เพื่อให้คุณเริ่มฝึกใช้ได้ทันทีในเฟสนี้ หลักสูตรจะกลับมาเจาะลึกเรื่อง **Branching Strategy ระดับทีม** อย่างเต็มรูปแบบใน **Part 35** ซึ่งจะครอบคลุมหัวข้อขั้นสูงกว่านี้มาก เช่น:

- Git Flow (มี branch หลัก `main`, `develop`, `feature/*`, `release/*`, `hotfix/*` ทำงานร่วมกันเป็นระบบ)
- GitHub Flow (แนวทางที่เรียบง่ายกว่า เน้น `main` + feature branch สั้น ๆ)
- Trunk-Based Development (แนวทางที่ทีมใหญ่ระดับ Google/Meta นิยมใช้)
- การตั้งกฎ branch protection บน GitHub/GitLab ให้บังคับใช้ naming convention จริงจัง

ตอนนี้ขอให้จำแค่หลักการง่าย ๆ ไว้ก่อน: **ตั้งชื่อ branch ให้สื่อความหมายเสมอ และใช้ prefix ที่เป็นมาตรฐานเมื่อทำงานเป็นทีม**

---

## Step 70: แบบฝึกหัด — จำลองพัฒนาฟีเจอร์คู่ขนาน สร้าง สลับ ลบ

ถึงเวลาเอาทุกอย่างที่เรียนมาใน Part นี้มาฝึกจริงในสถานการณ์จำลองที่ใกล้เคียงกับงานจริงมากที่สุด

### สถานการณ์สมมติ

คุณเป็นนักพัฒนาในทีมเล็ก ๆ ที่กำลังสร้างเว็บไซต์ร้านค้าออนไลน์ ทีมมอบหมายงานให้คุณทำ 3 อย่างพร้อมกัน (คู่ขนาน) คือ:

1. ฟีเจอร์ระบบสมัครสมาชิก (`feature/signup`)
2. แก้บั๊กปุ่มค้นหาไม่ทำงานในมือถือ (`bugfix/mobile-search`)
3. ฟีเจอร์ทดลอง Dark Mode ที่ยังไม่แน่ใจว่าจะใช้จริง (`experiment/dark-mode`)

### ขั้นตอนที่ 1: เตรียมโปรเจกต์ใหม่สะอาด ๆ

```bash
cd ~/git-course
mkdir part-07-exercise
cd part-07-exercise
git init
git branch -M main
```

ผลลัพธ์จำลอง:

```
Initialized empty Git repository in /home/user/git-course/part-07-exercise/.git/
```

### ขั้นตอนที่ 2: สร้าง commit เริ่มต้นบน main

```bash
echo "# ร้านค้าออนไลน์ของฉัน" > README.md
git add README.md
git commit -m "Initial commit: add README"
```

ผลลัพธ์จำลอง:

```
[main (root-commit) 9f8e7d6] Initial commit: add README
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

### ขั้นตอนที่ 3: เริ่มงานฟีเจอร์แรก — สมัครสมาชิก

```bash
git switch -c feature/signup
```

ผลลัพธ์จำลอง:

```
Switched to a new branch 'feature/signup'
```

```bash
echo "function signup() { /* TODO */ }" > signup.js
git add signup.js
git commit -m "Add signup form skeleton"
```

ผลลัพธ์จำลอง:

```
[feature/signup 1a2b3c4] Add signup form skeleton
 1 file changed, 1 insertion(+)
 create mode 100644 signup.js
```

### ขั้นตอนที่ 4: หัวหน้าทีมขอให้แวะแก้บั๊กด่วนก่อน — สลับไป branch ใหม่

ในชีวิตจริงเรื่องแบบนี้เกิดขึ้นตลอดเวลา — งานหนึ่งยังไม่เสร็จ แต่ต้องแวะไปทำอีกงานก่อน นี่คือจุดที่ branch ของ Git ทรงพลังมาก เพราะสลับไปมาได้โดยไม่ปนกัน

```bash
git switch main
git switch -c bugfix/mobile-search
```

ผลลัพธ์จำลอง:

```
Switched to branch 'main'
Switched to a new branch 'bugfix/mobile-search'
```

```bash
echo "/* fix: increase tap target size on mobile */" > search-fix.css
git add search-fix.css
git commit -m "Fix search button tap target on mobile"
```

ผลลัพธ์จำลอง:

```
[bugfix/mobile-search 2b3c4d5] Fix search button tap target on mobile
 1 file changed, 1 insertion(+)
 create mode 100644 search-fix.css
```

### ขั้นตอนที่ 5: merge บั๊กฟิกซ์กลับเข้า main ทันที (เพราะด่วน)

```bash
git switch main
git merge bugfix/mobile-search
```

ผลลัพธ์จำลอง:

```
Updating 9f8e7d6..2b3c4d5
Fast-forward
 search-fix.css | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 search-fix.css
```

### ขั้นตอนที่ 6: ลบ branch บั๊กฟิกซ์ที่เสร็จภารกิจแล้ว

```bash
git branch -d bugfix/mobile-search
```

ผลลัพธ์จำลอง:

```
Deleted branch bugfix/mobile-search (was 2b3c4d5).
```

### ขั้นตอนที่ 7: กลับไปทำงานฟีเจอร์สมัครสมาชิกต่อ

```bash
git switch feature/signup
```

ผลลัพธ์จำลอง:

```
Switched to branch 'feature/signup'
```

```bash
echo "function validateEmail() { /* TODO */ }" >> signup.js
git add signup.js
git commit -m "Add email validation stub"
```

ผลลัพธ์จำลอง:

```
[feature/signup 3c4d5e6] Add email validation stub
 1 file changed, 1 insertion(+)
```

### ขั้นตอนที่ 8: ทดลองฟีเจอร์ Dark Mode แบบไม่มั่นใจ

```bash
git switch main
git switch -c experiment/dark-mode
echo "/* dark mode experiment */" > dark.css
git add dark.css
git commit -m "Experiment: add dark mode CSS"
```

ผลลัพธ์จำลอง:

```
Switched to branch 'main'
Switched to a new branch 'experiment/dark-mode'
[experiment/dark-mode 4d5e6f7] Experiment: add dark mode CSS
 1 file changed, 1 insertion(+)
 create mode 100644 dark.css
```

### ขั้นตอนที่ 9: ดูภาพรวมทั้งหมดด้วยกราฟ

```bash
git switch main
git log --graph --all --oneline --decorate
```

ผลลัพธ์จำลอง:

```
* 4d5e6f7 (experiment/dark-mode) Experiment: add dark mode CSS
| * 3c4d5e6 (feature/signup) Add email validation stub
| * 1a2b3c4 Add signup form skeleton
|/
* 2b3c4d5 (HEAD -> main) Fix search button tap target on mobile
* 9f8e7d6 Initial commit: add README
```

ตอนนี้คุณเห็นภาพรวมทั้งหมดชัดเจน: `main` มีบั๊กฟิกซ์รวมเข้าไปแล้ว ในขณะที่ `feature/signup` และ `experiment/dark-mode` ยังคงพัฒนาคู่ขนานกันอยู่โดยไม่ปนกันเลย

### ขั้นตอนที่ 10: ตัดสินใจว่า Dark Mode ใช้ไม่ได้จริง — ลบทิ้งด้วย force delete

สมมติว่าทีมทดลองแล้วตัดสินใจไม่เอา Dark Mode ในตอนนี้ (ยังไม่ merge เข้า main เลย):

```bash
git branch -d experiment/dark-mode
```

ผลลัพธ์จำลอง (ถูกปฏิเสธเพราะยังไม่ merge):

```
error: The branch 'experiment/dark-mode' is not fully merged.
If you are sure you want to delete it, run 'git branch -D experiment/dark-mode'.
```

เพราะตั้งใจทิ้งจริง ๆ จึงใช้ force delete:

```bash
git branch -D experiment/dark-mode
```

ผลลัพธ์จำลอง:

```
Deleted branch experiment/dark-mode (was 4d5e6f7).
```

### ขั้นตอนที่ 11: ฟีเจอร์สมัครสมาชิกเสร็จแล้ว merge เข้า main และลบทิ้ง

```bash
git switch main
git merge feature/signup
```

ผลลัพธ์จำลอง:

```
Updating 2b3c4d5..3c4d5e6
Fast-forward
 signup.js | 2 ++
 1 file changed, 2 insertions(+)
 create mode 100644 signup.js
```

```bash
git branch -d feature/signup
```

ผลลัพธ์จำลอง:

```
Deleted branch feature/signup (was 3c4d5e6).
```

### ขั้นตอนที่ 12: ตรวจสอบผลลัพธ์สุดท้าย

```bash
git branch
```

ผลลัพธ์จำลอง:

```
* main
```

```bash
git log --graph --all --oneline --decorate
```

ผลลัพธ์จำลอง:

```
* 3c4d5e6 (HEAD -> main) Add email validation stub
* 1a2b3c4 Add signup form skeleton
* 2b3c4d5 Fix search button tap target on mobile
* 9f8e7d6 Initial commit: add README
```

ตอนนี้ประวัติทั้งหมดกลับมาเป็นเส้นตรงเดียว (linear) เพราะทุก branch ถูก merge เข้ามาแบบ **fast-forward** เรียบร้อยแล้ว และ branch ที่ทำหน้าที่เสร็จแล้วก็ถูกลบทิ้งอย่างปลอดภัย เหลือแค่ `main` เพียง branch เดียวที่สะอาดและเป็นระเบียบ

### สิ่งที่คุณเพิ่งฝึกไปในแบบฝึกหัดนี้

1. สร้าง branch หลายตัวคู่ขนานกันด้วย `git switch -c`
2. สลับไปมาระหว่างงานหลายชิ้นโดยไม่ทำให้งานปนกันเลย
3. merge งานที่เสร็จแล้วกลับเข้า `main`
4. ลบ branch ที่ merge แล้วด้วย `-d` (ปลอดภัย)
5. ลบ branch ทดลองที่ไม่ต้องการด้วย `-D` (บังคับ) หลังจากเจอคำเตือนของ Git
6. อ่านกราฟ `git log --graph --all --oneline --decorate` เพื่อเข้าใจภาพรวมทั้งหมด

**หมายเหตุสำคัญ:** ในแบบฝึกหัดนี้เราใช้คำสั่ง `git merge` ไปแล้วแบบที่ยังไม่ได้อธิบายกลไกละเอียด และทุกครั้งเป็นแบบ **fast-forward merge** (กรณีง่ายที่สุดที่ไม่มี conflict เพราะ branch ปลายทางไม่มี commit ใหม่ของตัวเองเลย) ใน **Part 08** เราจะเจาะลึกเรื่อง merge อย่างเต็มรูปแบบ พร้อมสถานการณ์ที่ซับซ้อนกว่านี้ที่ทั้งสอง branch ต่างก็มี commit ใหม่ของตัวเอง และจะทำให้เกิด **conflict** ที่ต้องแก้ไขด้วยมือเป็นครั้งแรกในหลักสูตรนี้

### แบบฝึกหัดเพิ่มเติม (ทำเองเพื่อทบทวน)

ลองทำด้วยตัวเองอีกรอบโดยไม่ดูเฉลย เพื่อทดสอบความเข้าใจ:

1. สร้าง branch ชื่อ `hotfix/typo` จาก `main`
2. แก้ไข README.md เพิ่ม 1 บรรทัด แล้ว commit
3. ลอง checkout ไปที่ commit hash ของ `main` โดยตรง (ไม่ใช่ชื่อ branch) สังเกตข้อความ detached HEAD ที่ Git แจ้งเตือน
4. สลับกลับมาที่ `main` แล้วสังเกตข้อความเตือนเรื่อง "leaving commits behind" (ถ้ามี)
5. เปลี่ยนชื่อ `hotfix/typo` เป็น `hotfix/readme-typo` ด้วย `git branch -m`
6. merge เข้า `main` แล้วลบทิ้งด้วย `-d`
7. รัน `git log --graph --all --oneline --decorate` ยืนยันว่าประวัติสะอาดเรียบร้อย

---

## สรุป Part 07

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Branch คือ pointer เล็ก ๆ ที่ชี้ไปยัง commit ตัวหนึ่ง** ไม่ใช่การ copy โฟลเดอร์ทั้งหมด — ถูกเก็บเป็นไฟล์ text ธรรมดาใน `.git/refs/heads/`
2. `git branch` ใช้ list, สร้าง, และตรวจสอบสถานะ branch — เครื่องหมาย `*` บอก branch ปัจจุบันเสมอ
3. `git switch` คือคำสั่งสมัยใหม่ (ตั้งแต่ Git 2.23) สำหรับสลับ branch แทนที่ `git checkout` แบบเก่าที่ทำหลายหน้าที่ปนกันจนสับสนง่าย
4. `git switch -c` (หรือ `git checkout -b` แบบเก่า) สร้างและสลับ branch พร้อมกันในคำสั่งเดียว
5. **HEAD คือ symbolic reference ที่ชี้ไปยัง branch** ไม่ใช่ชี้ไปยัง commit โดยตรงในสถานการณ์ปกติ — **Detached HEAD** เกิดเมื่อ HEAD ชี้ไปยัง commit ตรง ๆ และเป็นอันตรายเพราะ commit ใหม่ที่สร้างในสถานะนี้อาจไม่มี branch ใดอ้างอิงและเสี่ยงหายไปถาวร
6. ลบ branch ด้วย `-d` (ปลอดภัย ตรวจสอบสถานะ merge ก่อน) หรือ `-D` (บังคับ ไม่ตรวจสอบ) — Git ไม่ยอมให้ลบ branch ที่กำลังยืนอยู่เด็ดขาด
7. เปลี่ยนชื่อ branch ด้วย `git branch -m` (ปกติ) หรือ `-M` (บังคับทับชื่อเดิม)
8. `git log --graph --all --oneline --decorate` คือคำสั่งสำคัญที่สุดคำสั่งหนึ่งที่ใช้มองภาพรวมของ branch ทั้งหมดในประวัติ
9. Naming convention เบื้องต้น (`feature/`, `bugfix/`, `hotfix/`, `release/`, `chore/`, `docs/`) ช่วยให้ทีมสื่อสารและจัดการ branch จำนวนมากได้อย่างเป็นระบบ — จะเจาะลึกเต็มรูปแบบใน Part 35
10. ฝึกจำลองสถานการณ์จริงที่ต้องพัฒนาหลายฟีเจอร์คู่ขนานกัน สลับงานไปมา merge งานที่เสร็จ และลบ branch ที่ไม่ต้องการ — ครบวงจรการทำงานกับ branch ในชีวิตประจำวันจริง

### Checklist ก่อนไป Part 08

ก่อนไปต่อ ให้ตรวจสอบว่าคุณ:

- [ ] อธิบายได้ว่า branch คือ pointer ไม่ใช่การ copy โฟลเดอร์ และรู้ว่ามันถูกเก็บเป็นไฟล์อยู่ที่ไหนใน `.git/`
- [ ] สร้าง list และดู branch ปัจจุบันด้วย `git branch` ได้คล่อง
- [ ] เข้าใจความแตกต่างและเลือกใช้ `git switch` เป็นหลักแทน `git checkout` ได้ (แต่อ่าน `git checkout` ออกเมื่อเจอในโค้ดเก่า)
- [ ] สร้างและสลับ branch พร้อมกันด้วย `git switch -c` ได้
- [ ] อธิบายได้ว่า HEAD คืออะไรจริง ๆ และรู้ว่า Detached HEAD คืออะไร อันตรายอย่างไร และป้องกัน/กู้คืนอย่างไร
- [ ] ลบ branch ได้ทั้งแบบปลอดภัย (`-d`) และแบบบังคับ (`-D`) พร้อมรู้ว่าใช้เมื่อไหร่
- [ ] เปลี่ยนชื่อ branch ด้วย `git branch -m` ได้
- [ ] อ่านกราฟจาก `git log --graph --all --oneline --decorate` เข้าใจ
- [ ] รู้จัก naming convention เบื้องต้น (`feature/`, `bugfix/`, `hotfix/`) และเหตุผลที่ควรใช้
- [ ] ทำแบบฝึกหัดจำลองพัฒนาฟีเจอร์คู่ขนานจนจบครบทุกขั้นตอนด้วยตัวเอง

---

**ต่อไป:** [Part 08: Merge เบื้องต้นและการแก้ Conflict ครั้งแรก](./part-008-merge-เบื้องต้นและการแก้-conflict.md)
