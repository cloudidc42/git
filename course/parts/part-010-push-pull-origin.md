# Part 10: push/pull และการทำงานกับ origin

> **Step ในหลักสูตรนี้:** Step 91–100
> **เฟส:** 2 — การทำงานกับไฟล์และ Branch เบื้องต้น
> **เป้าหมายของ Part นี้:** เข้าใจกลไกจริงเบื้องหลัง `git push` และ `git pull` อย่างละเอียด ไม่ใช่แค่จำคำสั่งได้ แต่รู้ว่าเกิดอะไรขึ้นกับ ref ทั้งฝั่ง local และ remote ในแต่ละขั้นตอน สามารถแก้ปัญหา push ถูกปฏิเสธได้อย่างถูกวิธี ใช้ force push อย่างปลอดภัยเมื่อจำเป็นจริง ๆ จัดการ remote branch และ tag บน origin ได้ครบวงจร และฝึกจำลองสถานการณ์ทำงานร่วมกันหลายเครื่องด้วยตัวเองจนคล่อง

---

## สารบัญของ Part นี้

- Step 91: `git push` พื้นฐาน — ส่ง commit ขึ้น remote
- Step 92: `git push -u origin <branch>` — ตั้งค่า tracking ครั้งแรก ทำไมต้องมี `-u`
- Step 93: `git pull` คืออะไรจริง ๆ (คือ fetch + merge โดย default)
- Step 94: `git pull --rebase` คืออะไร ต่างจาก pull ธรรมดาอย่างไร
- Step 95: ปัญหา push ถูกปฏิเสธ (rejected — non-fast-forward) สาเหตุและวิธีแก้ที่ถูกต้อง
- Step 96: Force push (`--force`, `--force-with-lease`) อันตรายอย่างไร ต่างกันอย่างไร เมื่อไหร่พอยอมรับได้
- Step 97: การลบ remote branch (`git push origin --delete <branch>`)
- Step 98: การ push tag ขึ้น remote (`git push origin <tag>`, `git push --tags`)
- Step 99: Workflow ทำงานกับ remote แบบเต็มวงจร (clone → branch → commit → push)
- Step 100: แบบฝึกหัด — จำลอง 2 "เครื่อง" push/pull สลับกันไปมา รวมถึงจำลอง conflict ตอน push ถูกปฏิเสธ

---

## Step 91: `git push` พื้นฐาน — ส่ง commit ขึ้น remote

ใน Part 09 เราเรียนรู้เรื่อง `git fetch` ไปแล้วว่ามันดึงข้อมูลจาก remote เข้ามา **โดยไม่แตะ working directory หรือ local branch เลย** ทีนี้ถึงเวลาของฝั่งตรงข้าม นั่นคือการ **ส่ง** ข้อมูลของเราออกไปหา remote ด้วย `git push`

### `git push` ทำอะไรกันแน่

```bash
git push <remote> <branch>
```

ตัวอย่างที่ใช้บ่อยที่สุด:

```bash
git push origin main
```

เมื่อรันคำสั่งนี้ Git จะทำงานเป็นลำดับดังนี้:

1. **เชื่อมต่อกับ remote** ที่ชื่อ `origin` (ตาม URL ที่ตั้งไว้ด้วย `git remote add`)
2. **เปรียบเทียบ** ว่า commit ล่าสุดของ branch `main` บนเครื่องเรา อยู่ที่ตำแหน่งไหนเทียบกับ `main` บน remote
3. **คำนวณว่ามี object (commit, tree, blob) ตัวไหนบ้าง** ที่ remote ยังไม่มี แล้ว**อัดแพ็ก (pack)** ส่งไปเฉพาะส่วนที่ขาด — ไม่ได้ส่งทั้ง repository ใหม่ทุกครั้ง
4. **ขอให้ remote ขยับ ref** `refs/heads/main` ของมันไปชี้ที่ commit ใหม่ล่าสุด — **แต่ remote จะยอมขยับก็ต่อเมื่อการขยับนั้นเป็นแบบ fast-forward เท่านั้น** (คือ commit ใหม่ต้องมี commit เดิมของ remote เป็นบรรพบุรุษ)
5. ถ้าสำเร็จ Git จะรายงานผลแบบนี้:

```
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 320 bytes | 320.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
To github.com:username/my-project.git
   a1b2c3d..e4f5g6h  main -> main
```

บรรทัดสุดท้าย `a1b2c3d..e4f5g6h  main -> main` คือใจความสำคัญ: ref `main` บน `origin` ถูกขยับจาก commit `a1b2c3d` ไปเป็น `e4f5g6h` เรียบร้อยแล้ว

### สิ่งที่ push **ไม่ได้** ส่งไป

ต้องย้ำให้ชัดเจนเพราะมือใหม่มักเข้าใจผิด:

| สิ่งที่ push ส่งไป | สิ่งที่ push **ไม่ส่ง** |
|---|---|
| Commit ที่ทำไว้แล้วบน branch ที่ระบุ | การเปลี่ยนแปลงใน Working Directory ที่ยังไม่ได้ add |
| Object ทั้งหมดที่ commit นั้นอ้างอิงถึง (tree, blob) | การเปลี่ยนแปลงที่อยู่ใน Staging Area แต่ยังไม่ได้ commit |
| — | Branch อื่นที่ไม่ได้ระบุใน push (ปกติ push แค่ branch เดียวที่สั่ง) |
| — | Tag (ต้อง push แยกต่างหาก ดู Step 98) |

พูดง่าย ๆ คือ **push ส่งเฉพาะสิ่งที่ commit ไปแล้วเท่านั้น** ถ้าลืม commit ก็ไม่มีอะไรให้ push

### รูปแบบ refspec แบบเต็มของ push

คำสั่ง `git push origin main` จริง ๆ แล้วเป็นรูปแบบย่อของ:

```bash
git push origin main:main
```

ซึ่งใช้ **refspec** รูปแบบ `<local-ref>:<remote-ref>` คุณสามารถ push branch local ไปตั้งชื่อใหม่บน remote ก็ได้ เช่น:

```bash
git push origin my-local-fix:hotfix/urgent-bug
```

คำสั่งนี้เอา branch `my-local-fix` บนเครื่องเรา ส่งไปสร้างเป็น branch ชื่อ `hotfix/urgent-bug` บน `origin` — เป็นเทคนิคที่มีประโยชน์เวลาต้องการตั้งชื่อ branch บน remote ต่างจากชื่อ local

### push โดยไม่ระบุ branch เลย

ถ้าพิมพ์แค่ `git push` เฉย ๆ Git จะใช้พฤติกรรมตามค่า config `push.default` ซึ่งตั้งแต่ Git 2.0 เป็นต้นมาค่าเริ่มต้นคือ `simple`:

> **`simple`**: push เฉพาะ branch ปัจจุบันที่เรายืนอยู่ ไปยัง branch ชื่อเดียวกันบน remote **และ** branch นั้นต้องมี upstream (tracking branch) ตั้งไว้แล้วเท่านั้น ถ้ายังไม่มี Git จะปฏิเสธและบอกให้ตั้งค่าก่อน — นี่คือเหตุผลที่เราต้องเรียนเรื่อง `-u` ใน Step ถัดไป

ตรวจสอบค่าปัจจุบันได้ด้วย:

```bash
git config --get push.default
```

---

## Step 92: `git push -u origin <branch>` — ตั้งค่า tracking ครั้งแรก ทำไมต้องมี `-u`

### ปัญหาที่เจอบ่อยที่สุดของมือใหม่

สร้าง branch ใหม่ในเครื่อง แล้วลอง push ตรง ๆ:

```bash
git checkout -b feature/login-page
# ... commit งาน ...
git push
```

ผลลัพธ์ที่ได้:

```
fatal: The current branch feature/login-page has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin feature/login-page
```

นี่ไม่ใช่ error ที่แปลกประหลาด — Git กำลังบอกตรง ๆ ว่า **branch นี้ยังไม่เคย "ผูก" กับ branch ไหนบน remote เลย** จึงไม่รู้ว่าจะ push ไปที่ไหน

### tracking branch (upstream branch) คืออะไร

**Tracking branch** คือความสัมพันธ์ที่ Git จดจำไว้ระหว่าง local branch หนึ่งตัว กับ remote branch หนึ่งตัว เพื่อให้คำสั่งอย่าง `git push`, `git pull`, `git status` รู้โดยอัตโนมัติว่า "ถ้าไม่ระบุอะไรเพิ่ม ให้ทำงานกับ branch ไหนบน remote"

ตั้งค่าครั้งแรกด้วย flag `-u` (ย่อของ `--set-upstream`):

```bash
git push -u origin feature/login-page
```

คำสั่งนี้ทำ 2 อย่างพร้อมกัน:

1. **push** branch `feature/login-page` ขึ้นไปสร้างบน `origin` (ถ้ายังไม่มี)
2. **บันทึกความสัมพันธ์ tracking** ไว้ใน `.git/config` ของเรา

ลองเปิดดู `.git/config` หลัง push จะเห็นส่วนที่เพิ่มเข้ามา:

```ini
[branch "feature/login-page"]
    remote = origin
    merge = refs/heads/feature/login-page
```

`remote = origin` บอกว่า branch นี้ผูกกับ remote ชื่อ origin และ `merge = refs/heads/feature/login-page` บอกว่า ref ฝั่ง remote ที่เกี่ยวข้องคือ `feature/login-page` เช่นกัน

### ผลลัพธ์หลังตั้ง tracking แล้ว

จากนี้ไปสามารถพิมพ์สั้น ๆ ได้เลยโดยไม่ต้องระบุ remote/branch:

```bash
git push
git pull
```

Git จะรู้เองว่าหมายถึง `origin feature/login-page` เพราะมี tracking ผูกไว้แล้ว

### ดู tracking ทั้งหมดด้วย `git branch -vv`

```bash
git branch -vv
```

```
  main                 a1b2c3d [origin/main] เพิ่มหน้า README
* feature/login-page   f7e8d9c [origin/feature/login-page: ahead 2] เพิ่มฟอร์ม login
  old-experiment       0011223 ทดลอง layout ใหม่
```

สังเกตความหมายของแต่ละคอลัมน์:

| ส่วน | ความหมาย |
|---|---|
| `[origin/main]` | branch นี้ track กับ `origin/main` และปัจจุบัน sync กันพอดี ไม่ ahead ไม่ behind |
| `[origin/feature/login-page: ahead 2]` | track อยู่ แต่ local มี commit ที่ยังไม่ได้ push ไปอีก 2 commit (ahead 2) |
| `old-experiment` (ไม่มีวงเล็บ) | branch นี้**ยังไม่มี tracking** เลย ยังไม่เคย push หรือยังไม่เคยตั้งค่า upstream |

ข้อความ `ahead N` / `behind N` / `ahead N, behind N` ที่เห็นนี้คือสิ่งเดียวกับที่ `git status` แสดงเวลาบอกว่า "Your branch is ahead of 'origin/main' by 2 commits."

### วิธีตั้ง tracking แบบอื่น (ไม่ต้อง push ใหม่)

ถ้า branch นี้มีอยู่บน remote แล้ว (เช่นดึงมาจากคนอื่น) แต่ local branch ยังไม่ track อยู่ ใช้:

```bash
git branch --set-upstream-to=origin/feature/login-page feature/login-page
# หรือแบบสั้น ถ้ายืนอยู่บน branch นั้นอยู่แล้ว
git branch -u origin/feature/login-page
```

หรือใช้ทางลัดตอน push โดยไม่ต้องพิมพ์ชื่อ branch ซ้ำ:

```bash
git push -u origin HEAD
```

`HEAD` ในที่นี้หมายถึง "branch ปัจจุบันที่ยืนอยู่" ทำให้ push ไป branch ชื่อเดียวกันบน origin และตั้ง tracking ให้เลย เป็นคำสั่งที่โปรแกรมเมอร์มืออาชีพใช้บ่อยมากเวลาสร้าง branch ใหม่

### สรุปกฎง่าย ๆ

> **push branch ใหม่ครั้งแรก ให้ใส่ `-u` เสมอ** พอตั้ง tracking ครั้งเดียวแล้ว ครั้งต่อ ๆ ไปพิมพ์แค่ `git push` เฉย ๆ ก็พอ

---

## Step 93: `git pull` คืออะไรจริง ๆ (คือ fetch + merge โดย default)

### ทบทวนโครงสร้าง 3 ชั้นจาก Part 09

ก่อนเข้าใจ `pull` ต้องนึกภาพ 3 สิ่งนี้ให้ออกก่อน (ทวนจาก Part 09):

```
┌─────────────────────┐     git fetch      ┌──────────────────────────┐     git merge      ┌─────────────┐
│   Remote Repository  │ ─────────────────▶ │  origin/main              │ ─────────────────▶ │  main       │
│   (บน GitHub/GitLab) │   (ดาวน์โหลด object) │  (remote-tracking branch) │  (รวมเข้า local)     │  (local)     │
└─────────────────────┘                     └──────────────────────────┘                     └─────────────┘
```

- **Remote repository** — ตัวจริงที่อยู่บนเซิร์ฟเวอร์ เราแตะต้องโดยตรงไม่ได้ ต้องผ่านเครือข่าย
- **`origin/main`** — สำเนา (remote-tracking branch) ที่เก็บไว้ในเครื่องเรา คอยจำว่า "ครั้งล่าสุดที่เราคุยกับ remote มันอยู่ตรงไหน" — อัปเดตได้ด้วย `git fetch` เท่านั้น
- **`main`** — local branch จริงที่เรากำลังทำงานอยู่ ที่ Working Directory สะท้อนอยู่

### `git pull` คือคำสั่งประกอบ (compound command)

นี่คือประเด็นที่คนจำนวนมากไม่รู้: **`git pull` ไม่ใช่คำสั่งเดี่ยว ๆ ที่ทำอะไรพิเศษของตัวเอง** แต่มันคือ **shortcut ที่รันสองคำสั่งต่อกัน**:

```bash
git pull
```

เทียบเท่ากับ (โดย default):

```bash
git fetch origin
git merge origin/main     # (สมมติว่ายืนอยู่บน branch main และ track กับ origin/main)
```

พูดให้ชัดเป็นสูตร:

> **`git pull` = `git fetch` + `git merge FETCH_HEAD`**

### เดินตามขั้นตอนจริงทีละสเต็ป

สมมติเพื่อนร่วมทีม push commit ใหม่ขึ้น `origin/main` ไปแล้ว แต่เรายังไม่รู้ ตอนนี้ local `main` ของเรากับ `origin/main` (ตัวที่เครื่องเราจำไว้) ยัง**เท่ากันอยู่** (เพราะยังไม่ fetch)

**ขั้นที่ 1 — fetch:**

```bash
git fetch origin
```

```
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
Unpacking objects: 100% (3/3), 512 bytes | 512.00 KiB/s, done.
From github.com:username/my-project
   a1b2c3d..9f8e7d6  main       -> origin/main
```

ตอนนี้ `origin/main` ในเครื่องเราขยับไปที่ `9f8e7d6` แล้ว แต่ `main` (local) **ยังอยู่ที่ `a1b2c3d` เหมือนเดิม** และ Working Directory ก็ยังไม่เปลี่ยนอะไรเลย

**ขั้นที่ 2 — merge:**

```bash
git merge origin/main
```

ถ้า local `main` ไม่มี commit ใหม่ของตัวเองเลย (ไม่ ahead) Git จะทำ **fast-forward merge** — แค่ขยับ pointer ของ `main` ไปชี้ที่ `9f8e7d6` ตามหลัง `origin/main` ทันที:

```
Updating a1b2c3d..9f8e7d6
Fast-forward
 README.md | 3 +++
 1 file changed, 3 insertions(+)
```

แต่ถ้า local `main` มี commit ของตัวเองที่ `origin/main` ไม่มี (แยกทางกันแล้ว — diverged) Git จะสร้าง **merge commit** ใหม่ขึ้นมาเพื่อรวมสองประวัติเข้าด้วยกัน:

```
Merge made by the 'ort' strategy.
 app.js | 2 ++
 1 file changed, 2 insertions(+)
```

และถ้าทั้งสองฝั่งแก้ไฟล์บรรทัดเดียวกัน — เกิด **conflict** ต้องแก้ไขเองแบบที่เรียนไปแล้วใน Part 08

### แผนภาพสรุปสามเส้นทางที่เป็นไปได้เมื่อ pull

```
กรณีที่ 1: local ไม่มี commit ใหม่ของตัวเอง (ไม่ ahead)
   a1b2c3d ── 9f8e7d6                origin/main
   a1b2c3d                            main (ก่อน pull)
   → fast-forward: main ขยับตาม origin/main ตรง ๆ ไม่มี merge commit

กรณีที่ 2: local มี commit ใหม่ของตัวเอง แต่คนละไฟล์/คนละส่วน
   a1b2c3d ── 9f8e7d6                origin/main
        └──── b2c3d4e                main (ก่อน pull)
   → สร้าง merge commit ใหม่ รวมสองสาย ไม่มี conflict

กรณีที่ 3: local และ remote แก้ไฟล์เดียวกัน บรรทัดเดียวกัน
   → หยุดกลางทาง ให้แก้ conflict เอง แล้ว commit เพื่อจบการ merge
```

### คำเตือนจาก Git เวอร์ชันใหม่

Git ตั้งแต่เวอร์ชัน 2.27 เป็นต้นมา ถ้ายังไม่เคยตั้งค่าว่าจะให้ pull ทำงานแบบไหน จะเจอ warning แบบนี้ทุกครั้งที่ pull:

```
hint: You have divergent branches and need to specify how to reconcile them.
hint: You can do so by running one of the following commands sometime before
hint: your next pull:
hint:
hint:   git config pull.rebase false  # merge
hint:   git config pull.rebase true   # rebase
hint:   git config pull.ff only       # fast-forward only
```

ตั้งค่าถาวรเพื่อไม่ให้เจอ warning นี้อีก (เลือกพฤติกรรมที่ต้องการ):

```bash
# ให้ pull ใช้ merge เสมอ (พฤติกรรมดั้งเดิม/default ของ Git)
git config --global pull.rebase false

# ให้ pull ใช้ rebase เสมอ (ดู Step 94)
git config --global pull.rebase true

# อนุญาตเฉพาะ fast-forward เท่านั้น ถ้า diverged ให้ error แทนที่จะสร้าง merge commit
git config --global pull.ff only
```

---

## Step 94: `git pull --rebase` คืออะไร ต่างจาก pull ธรรมดา

### สูตรของ pull --rebase

ถ้า `git pull` ปกติ คือ `fetch` + `merge` แล้ว `git pull --rebase` ก็คือ:

> **`git pull --rebase` = `git fetch` + `git rebase origin/main`**

แทนที่จะ **merge** สองประวัติเข้าด้วยกัน (ซึ่งสร้าง merge commit ใหม่) มันจะ **"ยกเอา commit ของเรา" ไปวางต่อท้าย commit ล่าสุดที่ fetch มาแทน** ราวกับว่าเราเพิ่งเริ่มทำงานหลังจากที่คนอื่น push เสร็จแล้ว

### เปรียบเทียบด้วยภาพ

**ก่อน pull** — local มี commit `X` ของตัวเอง ในขณะที่ remote มี commit `Y` ที่เราไม่มี:

```
              X   (main, local)
             /
... ─ A ─ B
             \
              Y   (origin/main, หลัง fetch)
```

**หลัง `git pull` (merge แบบ default):**

```
... ─ A ─ B ─── Y ────┐
             \        │
              X ───── M   (main)   ← เกิด merge commit M ใหม่
```

**หลัง `git pull --rebase`:**

```
... ─ A ─ B ─ Y ─ X'   (main)   ← commit X ถูก "เขียนใหม่" เป็น X' วางต่อจาก Y
```

สังเกตว่า:

- **merge**: ประวัติแตกกิ่งแล้วค่อยรวมกลับ — เห็นชัดเจนว่าเกิดอะไรขึ้นจริง แต่ commit graph จะมีกิ่งก้านเยอะเมื่อทำบ่อย ๆ
- **rebase**: ประวัติเป็นเส้นตรงเดียว (linear history) อ่านง่ายเหมือนไม่มีอะไรเกิดขึ้นคู่ขนานเลย แต่ **commit ของเราถูกเขียนใหม่ทั้งหมด (hash เปลี่ยน)** จาก `X` กลายเป็น `X'`

### ทำไม hash ถึงเปลี่ยน

เพราะ commit hash ใน Git คำนวณมาจากเนื้อหา + parent ของมัน (ตามที่เรียนใน Part 01 เรื่อง snapshot) เมื่อ rebase ย้าย commit `X` ไปมี parent ใหม่เป็น `Y` แทนที่จะเป็น `B` เดิม เนื้อหาที่ใช้คำนวณ hash จึงเปลี่ยน ทำให้ได้ commit ใหม่ทั้งหมด (`X'`) แม้ diff ของโค้ดจะเหมือนเดิมทุกตัวอักษรก็ตาม

### ตัวอย่างการใช้งานจริง

```bash
git pull --rebase
```

```
Successfully rebased and updated refs/heads/main.
```

หรือถ้าอยากให้เป็นพฤติกรรม default ตลอดไปโดยไม่ต้องพิมพ์ `--rebase` ทุกครั้ง:

```bash
git config --global pull.rebase true
```

จากนั้นแค่ `git pull` เฉย ๆ ก็จะ rebase ให้อัตโนมัติ

### ตารางเปรียบเทียบสรุป

| หัวข้อ | `git pull` (merge) | `git pull --rebase` |
|---|---|---|
| Commit graph | แตกกิ่งแล้วรวม มี merge commit | เส้นตรง (linear) ไม่มี merge commit พิเศษ |
| Hash ของ commit ท้องถิ่น | **ไม่เปลี่ยน** | **เปลี่ยนทั้งหมด** (ถูกเขียนใหม่) |
| ปลอดภัยกับ commit ที่แชร์ให้คนอื่นแล้ว | ปลอดภัย | **อันตราย** ถ้า commit นั้นมีคนอื่นดึงไปใช้แล้ว (ดู Step 96 เรื่องกฎการห้าม rewrite ประวัติที่แชร์แล้ว) |
| อ่าน log ย้อนหลัง | เห็นว่ามีการทำงานคู่ขนานจริง ๆ | ดูเหมือนทำงานเรียงกันเป็นเส้นเดียว ทั้งที่จริงคู่ขนาน |
| เหมาะกับ | branch ที่แชร์กันหลายคนระยะยาว (เช่น main) | feature branch ส่วนตัวที่ยังไม่ได้แชร์ให้ใคร ก่อนเปิด PR |

### เกริ่นล่วงหน้า

หัวข้อ **Rebase vs Merge** และปรัชญาการเลือกใช้แต่ละแบบให้เหมาะกับสถานการณ์ เราจะเจาะลึกแบบเต็ม ๆ ใน **Part 38: Rebase vs Merge: เลือกใช้ให้ถูกต้อง** ส่วนกลไกเบื้องหลังของ `git rebase` แบบละเอียด รวมถึง **Interactive Rebase** (squash, reorder, edit history) จะเรียนใน **Part 39** ตอนนี้ขอให้จำแค่หลักการสำคัญไว้ก่อน:

> **อย่า rebase (หรือ pull --rebase) commit ที่คุณ push ให้คนอื่นเห็น/ดึงไปใช้แล้ว** เพราะมันจะเขียนประวัติใหม่ทำให้เกิดความสับสนและ conflict ซ้ำซ้อนกับทุกคนที่ใช้ commit เดิม

---

## Step 95: ปัญหา push ถูกปฏิเสธ (rejected — non-fast-forward)

นี่คือ error ที่ทุกคนที่ทำงานร่วมกับคนอื่นบน Git ต้องเจอสักวันหนึ่งแน่นอน มาทำความเข้าใจสาเหตุที่แท้จริงและวิธีแก้ที่ถูกต้อง

### สถานการณ์ที่ทำให้เกิดปัญหานี้

1. เพื่อนร่วมทีม (หรือตัวเราเองจากอีกเครื่อง) push commit ใหม่ขึ้น `origin/main` ไปก่อนแล้ว
2. เรายังไม่ได้ `fetch`/`pull` เข้ามา ดังนั้น local `main` ของเรายัง**ตามหลัง** remote อยู่ (หรือแย่กว่านั้นคือ **แยกทางกันไปแล้ว** — เรามี commit ของตัวเองด้วย)
3. เรา commit งานของตัวเองแล้วพยายาม push

```bash
git push origin main
```

ผลลัพธ์:

```
To github.com:username/my-project.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'github.com:username/my-project.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

### ทำไม Git ถึงปฏิเสธ (นี่คือกลไกป้องกัน ไม่ใช่บั๊ก)

ย้อนกลับไปที่กฎใน Step 91 ข้อ 4: **remote จะยอมขยับ ref ให้ก็ต่อเมื่อเป็น fast-forward เท่านั้น** (โดย default) นั่นคือ commit ใหม่ที่เราจะ push ไป ต้องมี commit ปัจจุบันของ remote เป็น "บรรพบุรุษ" อยู่ในสายของมัน

ในสถานการณ์นี้ ทั้งสองฝั่งมีประวัติแยกกัน:

```
                    C (commit ของเพื่อน — อยู่บน origin/main แล้ว)
                   /
... ─ A ─ B ──────
                   \
                    D (commit ของเรา — ยังไม่ได้ push)
```

ถ้า Git ยอมให้เรา push `D` ทับ `main` บน remote ตรง ๆ **commit `C` ของเพื่อนจะกลายเป็น commit ที่ไม่มี branch ไหนชี้ถึงอีกต่อไป (unreachable)** — เหมือนงานของเพื่อนหายไปจากหน้า branch หลักไปเลย (ถึงแม้ Git จะยังไม่ลบ object จริง ๆ ทันทีก็ตาม แต่คนอื่นจะมองไม่เห็นมันแล้วผ่าน branch ปกติ)

**นี่คือกลไกป้องกันความปลอดภัยที่สำคัญที่สุดอย่างหนึ่งของ Git** — มันปฏิเสธการ push ที่จะ "ทำให้งานของคนอื่นหายไปแบบเงียบ ๆ" โดยอัตโนมัติ

### วิธีแก้ที่ถูกต้อง

ทำตามที่ hint ของ Git บอกจริง ๆ นั่นคือ **ดึงงานของ remote เข้ามารวมกับของเราก่อน แล้วค่อย push ใหม่**:

```bash
git pull origin main
# แก้ conflict ถ้ามี แล้ว commit ให้จบการ merge
git push origin main
```

หรือถ้าอยากได้ประวัติเส้นตรง ใช้ rebase แทน merge (ตามที่เรียนใน Step 94):

```bash
git pull --rebase origin main
git push origin main
```

ทั้งสองวิธีทำให้ local `main` ของเรามี commit `C` ของเพื่อนรวมอยู่ด้วยแล้ว (ไม่ว่าจะผ่าน merge commit หรือ rebase มาวางต่อ) การ push ครั้งถัดไปจึงเป็น fast-forward จริง (หรืออย่างน้อยก็มี C เป็นบรรพบุรุษ) ทำให้ remote ยอมรับ

### สิ่งที่ **ไม่ควร** ทำเด็ดขาด

> อย่าแก้ปัญหานี้ด้วย `git push --force` โดยไม่คิดหน้าคิดหลัง! การ force push ทับตรงนี้จะลบ commit `C` ของเพื่อนออกจาก `main` ไปเลยจริง ๆ — เราจะพูดถึงอันตรายของ force push แบบละเอียดใน Step ถัดไป

### สรุปเป็นผังการตัดสินใจ

```
push ถูก reject (non-fast-forward)?
        │
        ▼
มั่นใจหรือไม่ว่า commit บน remote คืองานของคนอื่น/ตัวเราเองจากอีกเครื่องที่ "ต้องการเก็บไว้"?
        │
   ใช่ (เกือบทุกกรณี) ──────────────▶  git pull (หรือ git pull --rebase) แล้ว push ใหม่
        │
   ไม่ใช่ (แน่ใจ 100% ว่า commit นั้นเป็นขยะ/ทดลองที่ตั้งใจทิ้ง
   และไม่มีใครอื่นดึงไปใช้)
        │
        ▼
   พิจารณา force push แบบระมัดระวัง (ดู Step 96)
```

---

## Step 96: Force push (`--force`, `--force-with-lease`) อันตรายอย่างไร เมื่อไหร่พอยอมรับได้

### `--force` คืออะไร

```bash
git push --force origin main
# หรือแบบย่อ
git push -f origin main
```

`--force` (หรือ `-f`) สั่งให้ Git **ข้ามการตรวจสอบ fast-forward ไปเลย** แล้วบังคับให้ ref บน remote ชี้ไปที่ commit ที่เราส่งไป **ไม่ว่าประวัติเดิมบน remote จะเป็นอย่างไรก็ตาม**

ผลที่ตามมาคือ **commit ใด ๆ บน remote ที่ไม่ได้เป็นบรรพบุรุษของสิ่งที่เราเพิ่ง push จะถูกตัดขาดออกจาก branch นั้นทันที** ถ้าไม่มี branch/tag/reflog อื่นชี้ถึงมันอีก มันจะกลายเป็น "commit ลอย" ที่หาทางกลับมาได้ยากมาก (สุดท้ายจะถูก garbage collect ทิ้งไปจริง ๆ ตามเวลา)

### สถานการณ์อันตรายจริงที่เกิดขึ้นบ่อย

1. เพื่อนร่วมทีม push commit สำคัญขึ้น `main`
2. เรายังไม่ได้ pull เข้ามา แต่ local ของเรามี branch `main` ที่หลุดจากความเป็นจริงไปแล้ว (เช่นแก้ history ด้วย rebase/reset ไปเอง)
3. push ถูกปฏิเสธ (Step 95) แต่เรา**หงุดหงิดแล้วสั่ง `--force` โดยไม่เข้าใจ**
4. commit ของเพื่อนหายไปจาก `main` ทันที — เพื่อนต้องมานั่งงมหา commit นั้นจาก reflog หรือ backup กันวุ่นวาย

### `--force-with-lease` — เวอร์ชันปลอดภัยกว่า

```bash
git push --force-with-lease origin main
```

`--force-with-lease` ทำงานฉลาดกว่า `--force` ตรงที่: **ก่อนจะยอมบังคับเขียนทับ มันจะเช็คก่อนว่า remote-tracking branch (`origin/main`) ในเครื่องเรา ตรงกับสถานะจริงบน remote ตอนนี้หรือไม่**

พูดง่าย ๆ คือมันถามตัวเองว่า:

> "ตั้งแต่ครั้งล่าสุดที่ฉัน fetch จาก remote มา มีใครมา push อะไรทับที่นั่นอีกไหมโดยที่ฉันไม่รู้?"

- ถ้า **ไม่มีใครแตะต้อง remote เพิ่มเติม** ตั้งแต่เรา fetch ครั้งล่าสุด → อนุญาตให้ force push ได้ (ปลอดภัย เพราะเรารู้แน่ชัดว่ากำลังทับอะไรอยู่)
- ถ้า **มีคนอื่น push เพิ่มเข้าไปแล้ว** โดยที่เรายังไม่ได้ fetch มาเห็น → **ปฏิเสธการ force push** ป้องกันไม่ให้เราทับงานที่เรายังไม่เคยเห็นด้วยซ้ำ

```
To github.com:username/my-project.git
 ! [rejected]        main -> main (stale info)
error: failed to push some refs to '...'
```

ข้อความ `stale info` (ข้อมูลเก่าไม่ทันสมัย) คือสัญญาณเตือนว่า ควร `git fetch` ก่อนแล้วดูสถานการณ์ใหม่อีกครั้ง ก่อนตัดสินใจต่อ

### ความปลอดภัยขั้นสูงกว่านั้นอีก: `--force-if-includes`

Git เวอร์ชัน 2.30 ขึ้นไปมี flag เสริม `--force-if-includes` ที่ใช้คู่กับ `--force-with-lease` เพื่อตรวจสอบเพิ่มเติมว่า **commit เดิมที่กำลังจะถูกทับนั้น ถูกรวมอยู่ใน reflog ของ remote-tracking branch ของเราจริง ๆ** (ป้องกันกรณี edge case ที่ `--force-with-lease` เพียงอย่างเดียวอาจพลาดได้ เช่นตอนที่มี background fetch เกิดขึ้นระหว่างทาง)

```bash
git config --global push.useForceIfIncludes true
```

ตั้งค่านี้ไว้ทำให้ `--force-with-lease` ทุกครั้งพ่วง `--force-if-includes` ให้อัตโนมัติ เพิ่มความปลอดภัยโดยแทบไม่มีข้อเสีย

### ตารางเปรียบเทียบ

| | `--force` | `--force-with-lease` |
|---|---|---|
| ตรวจสอบก่อนทับหรือไม่ | ไม่ตรวจสอบเลย ทับทันที | ตรวจสอบว่า remote ตรงกับที่เรา fetch มาล่าสุดหรือไม่ |
| เสี่ยงทับงานคนอื่นที่เพิ่ง push โดยไม่รู้ตัว | เสี่ยงสูงมาก | ป้องกันกรณีนี้ได้ |
| แนะนำให้ตั้งเป็นนิสัยเริ่มต้นเวลาต้อง force push | ไม่แนะนำ | **แนะนำ** ใช้แทน `--force` เสมอเมื่อเป็นไปได้ |

### เมื่อไหร่ถึงพอยอมรับให้ force push ได้

Force push ไม่ใช่สิ่งชั่วร้ายเสมอไป มีสถานการณ์ที่เหมาะสมจริง ๆ:

1. **แก้ไข feature branch ส่วนตัวของตัวเอง** ที่ยังไม่มีใครดึงไปใช้ต่อ เช่น หลังจากทำ interactive rebase เพื่อ squash commit ให้สะอาดก่อนเปิด Pull Request
2. **branch ที่ตกลงกับทีมไว้แล้วว่าเป็น branch ทดลองส่วนตัว** ไม่ใช่ branch หลักที่ใช้ deploy หรือใช้ร่วมกัน
3. หลังจากที่ทำผิดพลาดร้ายแรง (เช่น commit ข้อมูลลับหลุดเข้าไป) และ**ทีมทั้งหมดตกลงร่วมกันแล้ว**ว่าจะ rewrite history และทุกคนจะต้อง re-clone/re-pull ใหม่

### กฎเหล็กที่ต้องจำ

> **ห้าม force push ทับ branch หลักที่ใช้ร่วมกัน (`main`, `master`, `develop`) โดยเด็ดขาด** เว้นแต่เป็นสถานการณ์ฉุกเฉินร้ายแรงที่ทีมตกลงร่วมกันแล้วเท่านั้น ในทางปฏิบัติ repository ระดับทีมมักตั้ง **Branch Protection Rules** (จะเรียนใน Part 37) เพื่อ**ปิดกั้น force push บน branch สำคัญไปเลยในระดับเซิร์ฟเวอร์** ไม่ต้องพึ่งวินัยส่วนตัวอย่างเดียว

---

## Step 97: การลบ remote branch (`git push origin --delete <branch>`)

เมื่องานบน feature branch เสร็จสิ้นแล้ว (merge เข้า main ไปแล้ว) การเก็บ branch นั้นไว้บน remote ต่อไปจะทำให้รายชื่อ branch รกและสับสน จึงควรลบทิ้ง

### คำสั่งลบ remote branch

```bash
git push origin --delete feature/login-page
```

```
To github.com:username/my-project.git
 - [deleted]         feature/login-page
```

### รูปแบบเก่าที่ยังใช้ได้อยู่ (คุ้มค่าที่จะเข้าใจ)

ก่อนที่ Git จะมี flag `--delete` แบบอ่านง่าย คนใช้ refspec แบบตรง ๆ นี้:

```bash
git push origin :feature/login-page
```

จำสูตร refspec จาก Step 91 ได้ไหมว่า `<local-ref>:<remote-ref>` — ในที่นี้ **ฝั่ง local-ref ถูกปล่อยว่างเปล่า** ซึ่งแปลว่า "ไม่มีอะไรจะ push ไปสร้างที่ remote-ref นั้น" ผลคือ Git **ลบ** ref นั้นทิ้งบน remote นั่นเอง ทั้งสองคำสั่งนี้ทำสิ่งเดียวกันทุกประการ แค่ `--delete` อ่านเข้าใจง่ายกว่ามาก จึงเป็นที่นิยมในปัจจุบัน

### สิ่งที่เกิดขึ้นกับเครื่องคนอื่นหลัง branch ถูกลบบน remote

นี่คือจุดที่มือใหม่มักงงบ่อย: **การลบ branch บน remote ไม่ได้ลบ remote-tracking branch (`origin/feature/login-page`) ที่อยู่ในเครื่องคนอื่นโดยอัตโนมัติทันที** เพื่อนร่วมทีมที่รัน `git branch -a` อาจยังเห็น `remotes/origin/feature/login-page` ค้างอยู่ ทั้งที่จริง ๆ มันถูกลบไปจากเซิร์ฟเวอร์แล้ว

ต้องสั่งให้ Git "เก็บกวาด" ref ที่ตายแล้วเหล่านี้ทิ้งด้วยตัวเอง:

```bash
git fetch --prune
# หรือแบบย่อ
git fetch -p
```

หรือใช้คำสั่งเฉพาะทางที่ทำแค่การเก็บกวาดอย่างเดียวโดยไม่ดึงข้อมูลใหม่:

```bash
git remote prune origin
```

ทั้งสองคำสั่งจะรายงานผลแบบนี้:

```
From github.com:username/my-project
 - [deleted]         (none)     -> origin/feature/login-page
```

### ตั้งค่าให้ prune อัตโนมัติทุกครั้งที่ fetch

ถ้าทำงานเป็นทีมและ branch ถูกลบบ่อย ตั้งค่านี้ไว้จะสะดวกมาก:

```bash
git config --global fetch.prune true
```

หลังจากตั้งค่านี้ ทุกครั้งที่ `git fetch` (หรือ `git pull` ซึ่งเรียก fetch ภายใน) จะเก็บกวาด remote-tracking branch ที่ตายแล้วให้อัตโนมัติเสมอ ไม่ต้องพิมพ์ `-p` เพิ่มอีก

### แยกให้ชัด: ลบ local branch vs ลบ remote branch

นี่คือ **การกระทำคนละอย่างกันโดยสิ้นเชิง** ต้องทำแยกกัน:

| คำสั่ง | ผลกระทบ |
|---|---|
| `git branch -d feature/login-page` | ลบ branch **บนเครื่องเราเท่านั้น** ไม่กระทบ remote เลย (ปฏิเสธถ้ายังไม่ได้ merge) |
| `git branch -D feature/login-page` | เหมือนข้างบน แต่บังคับลบแม้ยังไม่ merge |
| `git push origin --delete feature/login-page` | ลบ branch **บน remote เท่านั้น** ไม่กระทบ local branch ของเราเอง |

ในทางปฏิบัติเมื่องาน merge เสร็จแล้ว มักทำทั้งสองอย่างคู่กัน:

```bash
git checkout main
git pull
git branch -d feature/login-page              # ลบ local
git push origin --delete feature/login-page   # ลบ remote
```

หลาย platform อย่าง GitHub มีปุ่ม "Delete branch" ให้กดหลัง merge Pull Request เสร็จ ซึ่งเบื้องหลังก็คือการรัน `git push origin --delete` แบบเดียวกันนี้เอง (จะเรียนเรื่อง Pull Request เต็มรูปแบบใน Part 21)

---

## Step 98: การ push tag ขึ้น remote (`git push origin <tag>`, `git push --tags`)

### ข้อควรรู้ที่สำคัญที่สุด: push ปกติไม่พา tag ไปด้วย

ต่างจาก commit ที่ถูกส่งไปโดยอัตโนมัติเมื่อมันอยู่ในสายของ branch ที่ push **tag ไม่ถูกส่งไปพร้อมกับ `git push` ธรรมดาโดยอัตโนมัติ** ต้องสั่ง push tag แยกต่างหากเสมอ นี่เป็นการออกแบบที่ตั้งใจ เพื่อไม่ให้ tag ทดลองที่สร้างไว้ใช้ส่วนตัวหลุดขึ้นไปบน remote โดยไม่ตั้งใจ

(หมายเหตุ: หลักสูตรนี้จะเจาะลึกเรื่อง `git tag`, annotated tag vs lightweight tag, และ Semantic Versioning แบบเต็มใน **Part 11: Tag และการทำ Versioning ด้วย Git** — ที่นี่ขอโฟกัสเฉพาะฝั่ง push/pull tag กับ remote เท่านั้น)

### push tag เดี่ยว ๆ

```bash
git push origin v1.0.0
```

```
Enumerating objects: 1, done.
Counting objects: 100% (1/1), done.
Writing objects: 100% (1/1), 165 bytes | 165.00 KiB/s, done.
Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:username/my-project.git
 * [new tag]         v1.0.0 -> v1.0.0
```

### push tag ทั้งหมดในครั้งเดียว

ถ้าสร้าง tag ไว้หลายตัวในเครื่องแล้วยังไม่เคย push เลยสักตัว:

```bash
git push origin --tags
```

คำสั่งนี้จะ push **tag ทุกตัวที่มีในเครื่อง ที่ยังไม่มีบน remote** ไปให้หมดในครั้งเดียว (ทั้ง lightweight tag และ annotated tag)

### `--follow-tags` — ทางเลือกที่ปลอดภัยกว่า `--tags`

ปัญหาของ `--tags` คือมันส่ง **ทุก tag ในเครื่อง** ไปหมด แม้แต่ tag ทดลองที่ไม่เกี่ยวกับ branch ที่กำลัง push อยู่เลยก็ตาม ทางเลือกที่ปลอดภัยกว่าคือ:

```bash
git push --follow-tags
```

`--follow-tags` จะ push เฉพาะ **annotated tag ที่ชี้ไปยัง commit ที่อยู่ในสายของสิ่งที่กำลังถูก push อยู่จริง ๆ** เท่านั้น (ไม่ push lightweight tag ด้วย) เหมาะกับ workflow ที่สร้าง annotated tag ไว้เพื่อ mark จุด release แล้วต้องการ push commit กับ tag นั้นไปพร้อมกันในคำสั่งเดียวอย่างปลอดภัย

ตั้งค่าให้ push ธรรมดา (`git push` เฉย ๆ) พา tag ที่เกี่ยวข้องไปด้วยอัตโนมัติเสมอ:

```bash
git config --global push.followTags true
```

### ลบ tag บน remote

เช่นเดียวกับ branch tag ก็ลบบน remote ได้แยกจาก local:

```bash
git push origin --delete v1.0.0-beta
```

หรือรูปแบบ refspec เต็มแบบเก่า (ต้องระบุ `refs/tags/` ให้ชัดเจน เพราะถ้าเขียนแค่ชื่อเฉย ๆ Git อาจตีความสับสนกับชื่อ branch ที่ซ้ำกันได้):

```bash
git push origin :refs/tags/v1.0.0-beta
```

### แยกให้ชัด: ลบ tag local vs ลบ tag remote

| คำสั่ง | ผลกระทบ |
|---|---|
| `git tag -d v1.0.0-beta` | ลบ tag **บนเครื่องเราเท่านั้น** |
| `git push origin --delete v1.0.0-beta` | ลบ tag **บน remote เท่านั้น** |

เหมือนกับ branch เป๊ะ ๆ — ต้องทำทั้งสองคำสั่งถ้าต้องการลบให้หายไปจากทุกที่จริง ๆ

### ตารางสรุปคำสั่ง push ที่เกี่ยวกับ tag

| คำสั่ง | ทำอะไร |
|---|---|
| `git push origin <tagname>` | push tag ตัวเดียวที่ระบุชื่อ |
| `git push origin --tags` | push tag **ทุกตัว** ที่ยังไม่มีบน remote |
| `git push --follow-tags` | push เฉพาะ annotated tag ที่เกี่ยวข้องกับ commit ที่กำลัง push |
| `git push origin --delete <tagname>` | ลบ tag บน remote |

---

## Step 99: Workflow ทำงานกับ remote แบบเต็มวงจร (clone → branch → commit → push)

มาถึงจุดนี้เราได้เรียนครบทุกชิ้นส่วนแล้ว (`clone`, `remote add`, `fetch` จาก Part 09 และ `push`, `pull` จาก Part นี้) ถึงเวลาประกอบทุกอย่างเข้าด้วยกันเป็น **workflow เต็มรูปแบบ** ที่โปรแกรมเมอร์มืออาชีพใช้ทำงานร่วมกับทีมทุกวัน

### ภาพรวมทั้งวงจร

```
1. clone         →  ได้สำเนาเต็มของ repository มาไว้ในเครื่อง
2. branch        →  สร้าง feature branch แยกออกจาก main เพื่อทำงานอย่างปลอดภัย
3. commit        →  บันทึกความคืบหน้าเป็น snapshot เล็ก ๆ ทีละขั้น
4. push -u       →  ส่ง branch ขึ้น remote ครั้งแรก พร้อมตั้ง tracking
5. (Pull Request)→  เปิดคำขอให้ทีมรีวิวก่อนรวมเข้า main (เรียนเต็ม ๆ ใน Part 21)
6. merge & delete→  หลังรีวิวผ่าน รวมเข้า main แล้วลบ feature branch ทิ้งทั้ง local/remote
```

### เดินตามขั้นตอนแบบละเอียดพร้อมคำสั่งจริง

**1. Clone repository ของทีม:**

```bash
git clone git@github.com:my-team/my-project.git
cd my-project
```

**2. อัปเดต main ให้ล่าสุดก่อนเริ่มงานใหม่เสมอ:**

```bash
git checkout main
git pull
```

การ pull ก่อนเริ่มงานทุกครั้งเป็นนิสัยสำคัญมาก — ป้องกันไม่ให้เราสร้าง branch ใหม่จากจุดที่ล้าหลังเกินไป ซึ่งจะทำให้ต้อง merge/rebase มากขึ้นภายหลัง

**3. สร้าง feature branch แยกออกจาก main:**

```bash
git checkout -b feature/add-search-bar
```

**4. ทำงาน แล้ว commit เป็นก้อนเล็ก ๆ มีความหมายในตัวเอง:**

```bash
# แก้ไฟล์...
git add src/components/SearchBar.js
git commit -m "เพิ่ม component SearchBar เบื้องต้น"

# แก้ไฟล์เพิ่ม...
git add src/components/SearchBar.js src/App.js
git commit -m "เชื่อม SearchBar เข้ากับหน้า App หลัก"
```

**5. ระหว่างทำงาน ถ้าใช้เวลานาน ควร sync กับ main เป็นระยะ:**

```bash
git checkout main
git pull
git checkout feature/add-search-bar
git rebase main        # หรือ git merge main แล้วแต่ข้อตกลงของทีม
```

การทำแบบนี้เป็นระยะช่วยลดโอกาสเจอ conflict ก้อนใหญ่ตอนท้ายเมื่อ branch ทำงานนานหลายวัน

**6. push branch ขึ้น remote ครั้งแรก (พร้อมตั้ง tracking):**

```bash
git push -u origin feature/add-search-bar
```

**7. เปิด Pull Request บนแพลตฟอร์ม (เกริ่นล่วงหน้า):**

หลัง push แล้ว บน GitHub/GitLab จะมีลิงก์ให้เปิด **Pull Request (PR)** หรือ **Merge Request (MR)** ทันที ซึ่งเป็นหน้าเว็บสำหรับ:

- แสดง diff ทั้งหมดของ branch เทียบกับ main ให้ทีมเห็นชัดเจน
- ให้เพื่อนร่วมทีม comment รีวิวโค้ดทีละบรรทัดได้
- รัน automated check (CI) อัตโนมัติ เช่น test, lint (จะเรียนใน Part 66 เป็นต้นไป)
- เป็นจุดที่ทีมกด "Approve" ก่อนอนุญาตให้ merge เข้า main ได้จริง

เราจะเรียนรายละเอียดการสร้างและใช้งาน Pull Request เต็มรูปแบบใน **Part 21: Pull Request เบื้องต้น: สร้าง PR แรกของคุณ** ตอนนี้แค่ให้เห็นภาพว่ามันคือขั้นตอนที่มาต่อจาก push โดยธรรมชาติ

**8. หลัง PR ถูก approve และ merge เข้า main แล้ว เก็บกวาด branch ทิ้ง:**

```bash
git checkout main
git pull
git branch -d feature/add-search-bar
git push origin --delete feature/add-search-bar
```

### แผนภาพรวมทุก object และ ref ที่เกี่ยวข้องตลอด workflow

```
Local:                              Remote (origin):
┌─────────────────────┐            ┌─────────────────────┐
│ main                 │  push/pull │ main                 │
│ feature/add-search   │◀──────────▶│ feature/add-search   │
│ origin/main          │  (fetch)   │                      │
│ origin/feature/...   │            │                      │
└─────────────────────┘            └─────────────────────┘
```

### หลักปฏิบัติที่มืออาชีพยึดถือ

1. **`pull` (หรือ `fetch`) ก่อนเริ่มงานใหม่ทุกครั้ง** เพื่อเริ่มจากจุดล่าสุดเสมอ
2. **commit บ่อย ๆ เป็นก้อนเล็ก มีความหมายในตัวเอง** ดีกว่า commit ก้อนใหญ่ทีเดียวตอนจบ
3. **push บ่อย ๆ เพื่อสำรองงานไว้บน remote** ไม่ต้องรอให้งานเสร็จสมบูรณ์ก่อนค่อย push (feature branch ของตัวเอง force push ทับได้อย่างปลอดภัยตาม Step 96 หากจำเป็นต้องแก้ประวัติ)
4. **ไม่ commit ตรงบน `main` โดยตรง** ในโปรเจกต์ทีม — ทำงานผ่าน feature branch แล้วขอรีวิวผ่าน PR เสมอ
5. **ลบ branch ทิ้งทันทีหลัง merge** ทั้ง local และ remote เพื่อไม่ให้รายชื่อ branch รกในระยะยาว

---

## Step 100: แบบฝึกหัด — จำลอง 2 "เครื่อง" push/pull สลับกันไปมา

ถึงเวลาลงมือทำจริงเพื่อให้ทุกอย่างที่เรียนไปใน Part นี้ฝังเข้าไปในกล้ามเนื้อความจำ เราจะจำลองสถานการณ์ทำงานร่วมกัน 2 คน (หรือ 2 เครื่อง) โดยใช้ **bare repository** เป็นตัวแทน "remote server" อยู่ในเครื่องเราเอง — ไม่ต้องพึ่ง GitHub/GitLab เลยก็ฝึกกลไก push/pull ได้ครบถ้วน

### เตรียมพื้นที่ทำงาน

```bash
mkdir -p ~/git-course/part-10-practice
cd ~/git-course/part-10-practice
```

### ขั้นที่ 1: สร้าง "remote server" จำลองด้วย bare repository

**Bare repository** คือ repository ที่**ไม่มี Working Directory** — เก็บแค่ข้อมูลภายใน (`.git` content ล้วน ๆ) เหมาะสำหรับใช้เป็นจุดศูนย์กลางให้หลายคน push/pull เข้าออก (นี่คือรูปแบบเดียวกับที่ GitHub/GitLab ใช้เก็บ repository ของเราจริง ๆ เบื้องหลัง)

```bash
git init --bare server.git
```

```
Initialized empty Git repository in .../part-10-practice/server.git/
```

### ขั้นที่ 2: จำลอง "เครื่อง A" — clone แล้วสร้างงานแรก

```bash
git clone ./server.git machine-a
cd machine-a
git config user.name "Machine A"
git config user.email "machine-a@example.com"
```

```
warning: You appear to have cloned an empty repository.
```

คำเตือนนี้ปกติมาก เพราะ `server.git` ยังไม่มี commit อะไรเลย

สร้างไฟล์แรกและ push:

```bash
echo "# โปรเจกต์ฝึกหัด Part 10" > README.md
git add README.md
git commit -m "เริ่มโปรเจกต์: เพิ่ม README"
git push -u origin main
```

(ถ้า Git แจ้งว่า default branch เป็น `master` แทน `main` ให้ใช้ `git branch -M main` ก่อน push เพื่อให้ตรงกับตัวอย่าง)

### ขั้นที่ 3: จำลอง "เครื่อง B" — clone ของที่มีอยู่แล้ว

เปิด terminal อีกหน้าต่างหนึ่ง (หรือกลับมาที่โฟลเดอร์ `part-10-practice`):

```bash
cd ~/git-course/part-10-practice
git clone ./server.git machine-b
cd machine-b
git config user.name "Machine B"
git config user.email "machine-b@example.com"
```

ตรวจสอบว่าเห็น README.md ที่ Machine A push ไปแล้วจริง:

```bash
cat README.md
git log --oneline
```

### ขั้นที่ 4: Machine B ทำงานและ push กลับ

```bash
echo "## Section: Features" >> README.md
git add README.md
git commit -m "เพิ่มหัวข้อ Features ใน README"
git push
```

สังเกตว่าครั้งนี้ไม่ต้องใส่ `-u origin main` เพราะ `git clone` ตั้ง tracking branch ให้อัตโนมัติอยู่แล้วตั้งแต่ต้น

### ขั้นที่ 5: จำลอง push ถูกปฏิเสธ (rejected) แบบตั้งใจ

นี่คือหัวใจของแบบฝึกหัดนี้ กลับไปที่ **Machine A** (ซึ่งยังไม่รู้ว่า Machine B เพิ่ง push อะไรไป):

```bash
cd ~/git-course/part-10-practice/machine-a
echo "## Section: Installation" >> README.md
git add README.md
git commit -m "เพิ่มหัวข้อ Installation ใน README"
git push
```

ผลลัพธ์ที่ควรได้ (ถ้าทำตามลำดับถูกต้อง):

```
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to '.../server.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally...
```

**นี่คือสถานการณ์เดียวกับ Step 95 ทุกประการ** — Machine A พยายาม push ทับสิ่งที่ Machine B เพิ่งเพิ่มเข้าไป โดยที่ Machine A ยังไม่เคยเห็นการเปลี่ยนแปลงนั้นเลย

### ขั้นที่ 6: แก้ปัญหาอย่างถูกวิธีด้วย pull

ยังอยู่ที่ **Machine A**:

```bash
git pull
```

เนื่องจากทั้งสองฝั่งแก้ไฟล์ `README.md` แต่**คนละบรรทัด** (Machine B เพิ่ม "Features", Machine A เพิ่ม "Installation") Git ควรจะ merge อัตโนมัติได้โดยไม่ conflict สังเกตข้อความ:

```
Merge made by the 'ort' strategy.
 README.md | 1 +
 1 file changed, 1 insertion(+)
```

ตรวจสอบไฟล์ว่ามีทั้งสองหัวข้อรวมกันแล้ว:

```bash
cat README.md
git log --oneline --graph --all
```

จะเห็น commit graph แตกกิ่งแล้วมารวมกันเป็นรูปเพชร (diamond shape) ตามที่อธิบายไว้ใน Step 93

ตอนนี้ push ใหม่อีกครั้ง ควรสำเร็จ:

```bash
git push
```

### ขั้นที่ 7: ลองสร้างสถานการณ์ conflict จริง (ไม่ใช่แค่ merge เฉย ๆ)

คราวนี้ลองแก้ **บรรทัดเดียวกัน** ทั้งสองฝั่งเพื่อบังคับให้เกิด conflict จริง:

ที่ **Machine B**:

```bash
cd ~/git-course/part-10-practice/machine-b
git pull    # sync ให้ทันสมัยก่อน
```

แก้บรรทัดแรกของ README.md:

```bash
sed -i '1s/.*/# โปรเจกต์ฝึกหัด Part 10 (แก้โดย Machine B)/' README.md
git add README.md
git commit -m "แก้ไขชื่อโปรเจกต์จาก Machine B"
git push
```

ที่ **Machine A** (ยังไม่ pull):

```bash
cd ~/git-course/part-10-practice/machine-a
sed -i '1s/.*/# โปรเจกต์ฝึกหัด Part 10 (แก้โดย Machine A)/' README.md
git add README.md
git commit -m "แก้ไขชื่อโปรเจกต์จาก Machine A"
git push
```

Push ถูก reject เหมือนเดิม (ตามคาด) คราวนี้ pull เพื่อดู conflict จริง:

```bash
git pull
```

```
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

เปิดไฟล์ README.md จะเห็น conflict marker แบบที่เรียนไปแล้วใน Part 08:

```
<<<<<<< HEAD
# โปรเจกต์ฝึกหัด Part 10 (แก้โดย Machine A)
=======
# โปรเจกต์ฝึกหัด Part 10 (แก้โดย Machine B)
>>>>>>> <commit-hash-ของ-Machine-B>
```

แก้ conflict ด้วยตัวเอง (เลือกเก็บทั้งคู่ หรือเลือกอันใดอันหนึ่ง) แล้ว commit ให้จบการ merge:

```bash
# แก้ไฟล์ README.md ให้เหลือข้อความที่ต้องการจริง ๆ แล้วลบ marker ทิ้งให้หมด
git add README.md
git commit -m "แก้ conflict: รวมการเปลี่ยนแปลงชื่อโปรเจกต์จากทั้งสองเครื่อง"
git push
```

### ขั้นที่ 8 (โบนัส): ทดลอง `--force-with-lease` อย่างปลอดภัย

ที่ **Machine A** ลองแก้ commit ล่าสุดของตัวเองก่อน push (ในสถานการณ์ที่ยังไม่มีใคร push อะไรทับตั้งแต่เรา fetch ครั้งล่าสุด):

```bash
git commit --amend -m "แก้ conflict: รวมการเปลี่ยนแปลงชื่อโปรเจกต์ (แก้ข้อความ commit ใหม่)"
git push --force-with-lease
```

ควร push สำเร็จเพราะไม่มีใครแตะ `server.git` เพิ่มเติมนับตั้งแต่เรา fetch ล่าสุด ลองสลับไป **Machine B** แล้ว `git fetch` (ยังไม่ pull) จากนั้นกลับมาลองสั่ง `git commit --amend` แล้ว `push --force-with-lease` จาก Machine B **โดยไม่ fetch ก่อน** จะเห็นว่า Git ปฏิเสธด้วยเหตุผล `stale info` ทันที — เป็นการพิสูจน์ด้วยตัวเองว่า `--force-with-lease` ปกป้องเราจริงตามที่เรียนใน Step 96

### ขั้นที่ 9: ลบ branch ทดลองเพื่อฝึก Step 97–98 เพิ่ม

ลองสร้าง branch ทดลอง push ขึ้นไป แล้วลบทิ้งเพื่อฝึกคำสั่งจาก Step 97:

```bash
cd ~/git-course/part-10-practice/machine-a
git checkout -b throwaway-branch
git push -u origin throwaway-branch
git checkout main
git branch -d throwaway-branch
git push origin --delete throwaway-branch
```

แล้วไปที่ Machine B ลอง `git fetch --prune` เพื่อดูว่า remote-tracking branch ของ `throwaway-branch` หายไปจริงตามที่เรียนไว้

ลองสร้างและ push tag ตาม Step 98 ด้วย:

```bash
git tag -a v0.1.0 -m "เวอร์ชันฝึกหัดแรก"
git push origin v0.1.0
```

### Checklist ท้ายแบบฝึกหัด

- [ ] สร้าง bare repository จำลอง remote server ได้สำเร็จ
- [ ] clone เป็น 2 "เครื่อง" จาก server เดียวกันได้
- [ ] push จากเครื่องหนึ่งแล้ว pull เข้าอีกเครื่องได้ถูกต้อง
- [ ] จำลองสถานการณ์ push ถูก reject (non-fast-forward) และเข้าใจสาเหตุจริง
- [ ] แก้ปัญหาด้วย `git pull` จนสำเร็จ ไม่ใช้ force push แก้ปัญหาแบบมั่ว ๆ
- [ ] จำลอง merge conflict จริงระหว่างสองเครื่อง และแก้ไขจนจบได้ด้วยตัวเอง
- [ ] ทดลอง `--force-with-lease` และเห็นความแตกต่างเมื่อมี/ไม่มี fetch ล่าสุดก่อนหน้า
- [ ] ลบ remote branch และเห็นผลของ `git fetch --prune`
- [ ] push และเห็น tag ปรากฏบน remote จำลองสำเร็จ

---

## สรุป Part 10

ใน Part นี้เราได้เรียนรู้ว่า:

1. **`git push`** ส่งเฉพาะ commit ที่ทำไปแล้วขึ้น remote และ remote จะยอมรับก็ต่อเมื่อเป็น fast-forward เท่านั้น
2. **`-u` / `--set-upstream`** ตั้งค่า tracking branch ให้ local branch ผูกกับ remote branch ทำให้ใช้ `git push`/`git pull` แบบสั้น ๆ ได้ในครั้งต่อไป
3. **`git pull` คือ `git fetch` + `git merge` โดย default** — ไม่ใช่คำสั่งวิเศษที่ทำอะไรเป็นพิเศษของตัวเอง
4. **`git pull --rebase`** ใช้ rebase แทน merge หลัง fetch ทำให้ได้ประวัติเส้นตรง แต่เขียน hash ของ commit ท้องถิ่นใหม่ทั้งหมด
5. **push ถูกปฏิเสธ (non-fast-forward)** คือกลไกป้องกันไม่ให้เราทับงานของคนอื่นโดยไม่รู้ตัว วิธีแก้ที่ถูกต้องคือ pull ก่อนแล้วค่อย push ใหม่ ไม่ใช่ force push
6. **`--force` อันตรายมาก** เพราะเขียนทับประวัติ remote โดยไม่ตรวจสอบอะไรเลย ส่วน **`--force-with-lease`** ปลอดภัยกว่าเพราะเช็คก่อนว่า remote ไม่ได้ถูกเปลี่ยนแปลงตั้งแต่เรา fetch ล่าสุด
7. การลบ remote branch (`git push origin --delete`) และการลบ local branch เป็นคนละการกระทำกัน ต้องเก็บกวาดด้วย `git fetch --prune` เพื่อความสะอาด
8. **Tag ไม่ถูก push อัตโนมัติ** ต้องสั่ง push แยกด้วย `git push origin <tag>`, `--tags` หรือ `--follow-tags`
9. Workflow เต็มวงจรของการทำงานกับ remote คือ clone → branch → commit → push → (Pull Request) → merge → ลบ branch ทิ้ง
10. ผ่านแบบฝึกหัดจำลอง 2 เครื่องจริง เราได้ลงมือแก้ปัญหา push rejected และ merge conflict ด้วยตัวเองจนครบวงจร

**ต่อไป:** [Part 11: Tag และการทำ Versioning ด้วย Git](./part-011-tag-และการทำ-versioning.md)
