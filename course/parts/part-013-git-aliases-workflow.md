# Part 13: Git Aliases และการปรับแต่ง Workflow ส่วนตัว

> **Step ในหลักสูตรนี้:** Step 121–130
> **เฟส:** 2 — ใช้คำสั่งพื้นฐานได้คล่องและเริ่มปรับแต่งเครื่องมือให้เข้ากับตัวเอง
> **เป้าหมายของ Part นี้:** เข้าใจว่า Git Alias คืออะไร สร้างและลบ alias เป็น ใช้ alias ยอดนิยมที่โปรแกรมเมอร์มืออาชีพใช้จริงในงานประจำวัน เข้าใจการเรียก shell command ผ่าน alias เข้าใจการจัดการไฟล์ config ส่วนตัว (`.gitconfig`, global `.gitignore`) รวมถึงการแยกโปรไฟล์งาน/ส่วนตัวด้วย conditional include และปิดท้ายด้วยการสร้าง Workflow เฉพาะตัวที่ผสม Git alias กับ Shell alias เข้าด้วยกัน

---

## สารบัญของ Part นี้

- Step 121: Alias คืออะไร ทำไมช่วยเพิ่ม Productivity ของโปรแกรมเมอร์
- Step 122: สร้าง Alias พื้นฐานตัวแรกของคุณ
- Step 123: Alias ยอดนิยมที่โปรแกรมเมอร์มืออาชีพใช้จริง
- Step 124: Alias ขั้นสูงด้วยการเรียก Shell Command (เครื่องหมาย `!`)
- Step 125: แก้ไข Alias ผ่านไฟล์ `~/.gitconfig` โดยตรงด้วย Text Editor
- Step 126: ลบ Alias ที่ไม่ใช้แล้ว
- Step 127: ไฟล์ Global `.gitignore` และ `.gitconfig` ที่ควรมีติดเครื่องทุกเครื่อง
- Step 128: จัดการ Git Config หลายโปรไฟล์ (งาน vs ส่วนตัว) ด้วย Conditional Includes
- Step 129: Shell Alias/Function เสริมที่ครอบคำสั่ง Git ให้สั้นลงไปอีก
- Step 130: แบบฝึกหัด — สร้างชุด Alias ส่วนตัวและทดสอบใช้งานจริง

---

## Step 121: Alias คืออะไร ทำไมช่วยเพิ่ม Productivity ของโปรแกรมเมอร์

### Alias คืออะไรในความหมายทั่วไป

คำว่า **Alias** แปลตรงตัวว่า "ชื่อเล่น" หรือ "นามแฝง" ในบริบทของซอฟต์แวร์ Alias หมายถึง **การตั้งชื่อสั้น ๆ ให้แทนคำสั่งที่ยาวหรือซับซ้อนกว่า** เพื่อให้เรียกใช้ได้เร็วขึ้นโดยไม่ต้องพิมพ์คำสั่งเต็มทุกครั้ง

แนวคิดนี้ไม่ได้มีเฉพาะใน Git — ระบบปฏิบัติการ Unix/Linux มี shell alias มานานแล้ว เช่น การตั้ง `ll` แทน `ls -la` แต่ในหลักสูตรนี้เราจะโฟกัสที่ **Git Alias** ก่อน แล้วค่อยขยายไปที่ Shell Alias ใน Step 129

### Git Alias คืออะไรโดยเฉพาะ

**Git Alias** คือกลไกที่ Git มีให้ในตัวเอง สำหรับ**สร้างคำสั่งย่อยใหม่** ที่ผูกกับคำสั่ง Git ยาว ๆ หรือชุดของ flag ที่คุณใช้บ่อย โดยเก็บไว้ในไฟล์ config ของ Git (ไม่ว่าจะเป็นระดับ global, local หรือ system)

ตัวอย่างที่เห็นภาพชัดที่สุด:

```bash
# แทนที่จะพิมพ์คำสั่งนี้ทุกครั้ง
git status

# คุณสร้าง alias แล้วพิมพ์แค่นี้
git st
```

ทั้งสองคำสั่งทำงาน**เหมือนกันทุกประการ** — `git st` เป็นเพียง "ทางลัด" ที่ Git แปลงกลับไปเป็น `git status` ก่อนรันจริงเบื้องหลัง

### ทำไม Alias ถึงสำคัญต่อ Productivity มากกว่าที่คิด

หลายคนมองข้าม alias เพราะคิดว่าเป็นเรื่อง "สะดวกเฉย ๆ" ไม่ได้จำเป็นจริงจัง แต่ในความเป็นจริงแล้ว alias ส่งผลต่อประสิทธิภาพการทำงานของโปรแกรมเมอร์ในหลายมิติ:

1. **ลดจำนวนตัวอักษรที่ต้องพิมพ์ต่อวัน** — โปรแกรมเมอร์ที่ใช้ Git เป็นหลักอาจพิมพ์คำสั่ง Git หลายสิบถึงหลายร้อยครั้งต่อวัน ถ้าแต่ละคำสั่งประหยัดได้ 10–20 ตัวอักษร เมื่อคูณด้วยจำนวนครั้งต่อวันแล้ว จะประหยัดเวลาได้อย่างมีนัยสำคัญตลอดทั้งปี

2. **ลดโอกาสพิมพ์ผิด (Typo)** — คำสั่งอย่าง `git log --graph --oneline --all --decorate` มีโอกาสพิมพ์ผิดสูงกว่า `git lg` มาก ยิ่งคำสั่งซับซ้อนเท่าไหร่ ความเสี่ยงที่จะพิมพ์ flag ผิดหรือลืม flag ก็ยิ่งสูงขึ้น

3. **รักษา Flow State (สภาวะจดจ่อ)** — เวลาที่โปรแกรมเมอร์ต้องหยุดคิดว่า "คำสั่งเต็ม ๆ มัน flag อะไรบ้างนะ" แม้จะเป็นเวลาแค่ไม่กี่วินาที แต่มันตัดขาดสมาธิจากงานหลักที่กำลังทำอยู่ (context switching) การมี alias ที่จำได้ขึ้นใจช่วยให้มือพิมพ์คำสั่งได้โดยแทบไม่ต้องคิด เหมือนเป็นส่วนหนึ่งของ muscle memory

4. **มาตรฐานส่วนตัวที่สม่ำเสมอ** — เมื่อคุณตั้ง alias ที่ใช้ทุกเครื่องเหมือนกัน (ผ่านการ sync ไฟล์ dotfiles ซึ่งจะพูดถึงใน Step 127) คุณจะมี Workflow ที่เหมือนกันไม่ว่าจะทำงานที่เครื่องไหน ลดความสับสนเวลาสลับเครื่อง

5. **เป็นจุดเริ่มต้นของการปรับแต่งเครื่องมือให้เข้ากับตัวเอง (Tool Customization)** — นี่คือทักษะที่แยกโปรแกรมเมอร์มือใหม่กับมือโปรออกจากกันชัดเจนมาก มือโปรมักลงทุนเวลาปรับแต่ง terminal, editor และเครื่องมือที่ใช้ทุกวันให้เข้ากับสไตล์ตัวเองมากที่สุด เพราะรู้ว่าการลงทุนครั้งเดียวจะได้ผลตอบแทนคืนมาทุกวันตลอดการทำงาน

### Alias ไม่ได้แทนที่การเข้าใจคำสั่งจริง

ข้อควรระวังสำคัญ: **Alias ไม่ใช่ทางลัดให้ไม่ต้องเข้าใจ Git** คุณควรเข้าใจคำสั่งเต็มที่อยู่เบื้องหลัง alias ทุกตัวเสมอ เพราะ:

- เวลาทำงานบนเครื่องคนอื่นที่ไม่มี alias ของคุณ คุณต้องพิมพ์คำสั่งเต็มได้
- เวลาอ่าน commit message, script หรือ CI/CD pipeline ของทีม คุณจะเจอคำสั่งเต็ม ไม่ใช่ alias
- Alias ที่สร้างโดยไม่เข้าใจ อาจทำให้เกิดพฤติกรรมที่ไม่คาดคิดเมื่อใช้ผิดบริบท

ดังนั้นแนวทางที่ถูกต้องคือ: **เรียนรู้คำสั่งเต็มให้เข้าใจก่อน แล้วค่อยสร้าง alias เพื่อความเร็วในภายหลัง** ซึ่งเป็นเหตุผลที่หลักสูตรนี้เพิ่งมาสอนเรื่อง alias ใน Part 13 หลังจากที่คุณได้เรียนคำสั่งพื้นฐานของ Git ไปมากพอสมควรแล้วในเฟส 1–2

---

## Step 122: สร้าง Alias พื้นฐานตัวแรกของคุณ

### คำสั่งพื้นฐานสำหรับสร้าง Alias

Git alias ถูกสร้างผ่านคำสั่ง `git config` โดยใช้ namespace ชื่อ `alias.<ชื่อที่คุณตั้ง>`

```bash
git config --global alias.st status
```

คำสั่งนี้ทำสิ่งต่อไปนี้:

1. เปิดไฟล์ config ระดับ global (คือไฟล์ `~/.gitconfig` หรือ `~/.config/git/config`)
2. เพิ่ม (หรือแก้ไข) ส่วน `[alias]` แล้วเซ็ตค่า `st = status`
3. บันทึกไฟล์กลับ

หลังจากรันคำสั่งนี้แล้ว คุณสามารถทดสอบได้ทันที:

```bash
git st
```

ผลลัพธ์ที่ได้จะเหมือนกับการรัน `git status` ทุกประการ เพราะ Git เพียงแค่ **แทนที่ (substitute)** คำว่า `st` ด้วย `status` ก่อนประมวลผลคำสั่งจริง

### ทำความเข้าใจ Scope: `--global` vs ไม่มี Flag

คำสั่ง `git config` รับ flag ที่กำหนด **ขอบเขต (scope)** ว่า alias นี้จะถูกเก็บไว้ที่ไหน:

| Flag | ไฟล์ที่ถูกแก้ไข | ขอบเขตการใช้งาน |
|---|---|---|
| `--system` | ไฟล์ config ระดับระบบปฏิบัติการ (เช่น `/etc/gitconfig`) | ใช้ได้กับทุก user บนเครื่องนั้น (ต้องมีสิทธิ์ admin) |
| `--global` | `~/.gitconfig` ของ user ปัจจุบัน | ใช้ได้กับทุก repository ที่ user นี้ทำงานด้วย |
| (ไม่ใส่ flag) หรือ `--local` | `.git/config` ภายใน repository ปัจจุบัน | ใช้ได้เฉพาะ repository นั้นเท่านั้น |

ตัวอย่างการสร้าง local alias (ใช้ได้เฉพาะ repo ปัจจุบัน):

```bash
cd my-project
git config alias.deploy '!npm run build && git push origin main'
```

Alias `deploy` นี้จะทำงานได้เฉพาะเวลาที่คุณอยู่ใน repository `my-project` เท่านั้น ถ้าไปเรียก `git deploy` ที่ repo อื่นจะได้ error ว่า `git: 'deploy' is not a git command`

**ในหลักสูตรนี้ เราจะเน้น `--global` เป็นหลัก** เพราะ alias ส่วนใหญ่ที่จะเรียนเป็นสิ่งที่คุณอยากใช้ได้ในทุกโปรเจกต์ที่คุณทำงานด้วย

### ลำดับความสำคัญเมื่อมี Alias ชื่อเดียวกันหลาย Scope

ถ้า alias ชื่อเดียวกันถูกกำหนดไว้ทั้งใน local, global และ system Git จะใช้ค่าตามลำดับความสำคัญจากมากไปน้อยดังนี้:

```
local (.git/config) > global (~/.gitconfig) > system (/etc/gitconfig)
```

พูดง่าย ๆ คือ **ไฟล์ที่ใกล้ตัว repository ที่สุดจะชนะเสมอ** ซึ่งเป็นหลักการเดียวกับที่ใช้กับค่า config อื่น ๆ ของ Git ทั้งหมด ไม่ใช่แค่ alias

### ตรวจสอบ Alias ที่มีอยู่ทั้งหมด

หลังสร้าง alias แล้ว คุณสามารถตรวจสอบรายการ alias ทั้งหมดที่มีอยู่ได้ด้วย:

```bash
git config --get-regexp alias
```

ตัวอย่างผลลัพธ์:

```
alias.st status
```

หรือถ้าอยากดู config ทั้งหมด (ไม่เฉพาะ alias) พร้อมระบุว่ามาจากไฟล์ไหน:

```bash
git config --list --show-origin
```

ผลลัพธ์จะแสดงบรรทัดแบบนี้:

```
file:/home/user/.gitconfig	alias.st=status
```

ซึ่งบอกชัดเจนว่า alias นี้มาจากไฟล์ `~/.gitconfig`

### ทดสอบด้วยตัวเอง

ลองสร้าง alias ง่าย ๆ อีกสองสามตัวเพื่อความคุ้นเคย:

```bash
git config --global alias.co checkout
git config --global alias.br branch
```

จากนั้นเข้าไปในโฟลเดอร์ฝึกฝนที่คุณสร้างไว้ตั้งแต่ Part 01 (`~/git-course`) แล้วลองใช้งานจริง:

```bash
cd ~/git-course
mkdir part-13-practice && cd part-13-practice
git init
git co -b feature/test    # เทียบเท่ากับ git checkout -b feature/test
git br                    # เทียบเท่ากับ git branch
```

คุณจะเห็นว่าคำสั่งทำงานได้เหมือนคำสั่งเต็มทุกประการ นี่คือหัวใจของ alias — **มันคือคำสั่งเดิม เพียงแค่ผ่านชื่อที่สั้นลง**

---

## Step 123: Alias ยอดนิยมที่โปรแกรมเมอร์มืออาชีพใช้จริง

หลังจากเข้าใจกลไกพื้นฐานแล้ว มาดู alias ที่เป็น "มาตรฐานที่ไม่เป็นทางการ" ในวงการ ที่โปรแกรมเมอร์ทั่วโลกใช้ซ้ำ ๆ กันจนแทบจะเป็นธรรมเนียมปฏิบัติ

### 1. `co` — Checkout

```bash
git config --global alias.co checkout
```

ใช้งาน:

```bash
git co main              # git checkout main
git co -b hotfix/login   # git checkout -b hotfix/login
```

### 2. `br` — Branch

```bash
git config --global alias.br branch
```

ใช้งาน:

```bash
git br                   # git branch (ดูรายการ branch)
git br -a                # git branch -a (ดูทั้ง local และ remote)
git br -d old-feature    # git branch -d old-feature (ลบ branch)
```

### 3. `ci` — Commit

```bash
git config --global alias.ci commit
```

ใช้งาน:

```bash
git ci -m "fix: แก้บั๊กการคำนวณราคา"
git ci -am "update: ปรับปรุงหน้า login"
```

> **ข้อควรระวัง:** ระวังอย่าสับสน `ci` (alias ของ commit) กับคำว่า CI ที่หมายถึง Continuous Integration ในบริบทของ DevOps — เป็นคนละเรื่องกันโดยสิ้นเชิง แค่บังเอิญตัวย่อซ้ำกัน

### 4. `unstage` — เอาไฟล์ออกจาก Staging Area

```bash
git config --global alias.unstage 'reset HEAD --'
```

นี่คือ alias คลาสสิกที่มาจากเอกสารทางการของ Git เอง (Pro Git book) ใช้สำหรับกรณีที่คุณ `git add` ไฟล์ไปแล้วแต่เปลี่ยนใจอยากเอาออกจาก staging area โดยไม่แตะเนื้อหาไฟล์เลย

ใช้งาน:

```bash
git add file1.txt file2.txt
git unstage file2.txt     # เทียบเท่ากับ git reset HEAD -- file2.txt
```

ผลลัพธ์คือ `file2.txt` จะกลับไปอยู่ในสถานะ "Changes not staged for commit" แต่เนื้อหาในไฟล์ยังคงอยู่เหมือนเดิมทุกประการ

สังเกตว่า alias นี้มี **เว้นวรรค** อยู่ภายใน (`reset HEAD --`) จึงต้องครอบด้วยเครื่องหมายคำพูดเมื่อรันคำสั่ง `git config` มิฉะนั้น shell จะตัดคำสั่งผิด

### 5. `last` — ดู Commit ล่าสุดแบบละเอียด

```bash
git config --global alias.last 'log -1 HEAD'
```

ใช้งาน:

```bash
git last
```

ผลลัพธ์ตัวอย่าง:

```
commit 8f3a1c2e9b7d4f0a1e2c3d4e5f6a7b8c9d0e1f2a
Author: Somchai Devcode <somchai@example.com>
Date:   Fri Sep 25 14:32:10 2026 +0700

    fix: แก้บั๊กการคำนวณราคาสินค้าตอนมีส่วนลด
```

เป็น alias ที่ใช้บ่อยมากตอนอยากเช็คว่า commit ล่าสุดที่เพิ่งทำไปมีข้อความว่าอะไร เขียนถูกหรือยัง โดยไม่ต้องเปิด log ทั้งหมด

### 6. `lg` — Log แบบ Graph สวยงาม (ที่ใช้กันแพร่หลายที่สุด)

นี่คือ alias ที่โด่งดังที่สุดในวงการ Git แทบทุกคนที่ใช้ Git จริงจังจะมี alias คล้าย ๆ แบบนี้ติดเครื่องไว้:

```bash
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

มาแยกส่วนประกอบทีละส่วนเพื่อความเข้าใจ:

| ส่วนของคำสั่ง | ความหมาย |
|---|---|
| `--color` | เปิดใช้สี |
| `--graph` | วาดเส้น branch/merge แบบ ASCII art |
| `--pretty=format:'...'` | กำหนดรูปแบบการแสดงผลของแต่ละ commit เอง |
| `%Cred%h%Creset` | hash ย่อของ commit สีแดง |
| `%C(yellow)%d%Creset` | ชื่อ branch/tag ที่ชี้มาที่ commit นี้ สีเหลือง |
| `%s` | ข้อความ commit message (subject line) |
| `%Cgreen(%cr)%Creset` | เวลาแบบ relative (เช่น "2 hours ago") สีเขียว |
| `%C(bold blue)<%an>%Creset` | ชื่อผู้เขียน commit สีน้ำเงินตัวหนา |
| `--abbrev-commit` | แสดง hash แบบย่อแทนที่จะเป็น hash เต็ม 40 ตัวอักษร |

ใช้งาน:

```bash
git lg
```

ผลลัพธ์ตัวอย่าง (ในเทอร์มินัลจะมีสีจริง):

```
* a1b2c3d -  (HEAD -> main, origin/main) fix: แก้บั๊กหน้า login (2 hours ago) <Somchai>
* 9f8e7d6 -  (feature/payment) add: เพิ่มระบบชำระเงิน (5 hours ago) <Malee>
| * 4c5d6e7 - (feature/report) wip: เริ่มทำหน้ารายงาน (1 day ago) <Somchai>
|/
* 2b3c4d5 -  init: สร้างโปรเจกต์เริ่มต้น (3 days ago) <Somchai>
```

alias นี้มีประโยชน์มากในการดูภาพรวมว่า branch ไหนแตกจากไหน merge กลับมาตรงไหนบ้าง แบบอ่านง่ายกว่า `git log --graph` เปล่า ๆ มาก

ถ้าต้องการดูทุก branch (ไม่ใช่แค่ branch ปัจจุบัน) ให้เพิ่ม `--all`:

```bash
git config --global alias.lga "log --color --graph --all --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

### 7. `st` แบบละเอียดขึ้น — Status แบบย่อ

ในตอนต้นเราสร้าง `st = status` ธรรมดา แต่หลายคนนิยมเพิ่ม flag `-sb` เพื่อให้ผลลัพธ์กระชับขึ้น:

```bash
git config --global alias.st 'status -sb'
```

`-s` คือ short format และ `-b` คือแสดงชื่อ branch ปัจจุบันด้วย ตัวอย่างผลลัพธ์:

```
## main...origin/main [ahead 1]
 M src/app.js
?? notes.txt
```

เทียบกับ `git status` แบบเต็มที่ยาวกว่ามาก การใช้ `-sb` ทำให้อ่านสถานะไฟล์จำนวนมากได้เร็วขึ้นเยอะ

### 8. Alias เสริมอื่น ๆ ที่นิยมมาก

```bash
git config --global alias.df diff
git config --global alias.dc 'diff --cached'
git config --global alias.aa 'add --all'
git config --global alias.cm 'commit -m'
git config --global alias.amend 'commit --amend --no-edit'
```

- `df` — ดู diff ของไฟล์ที่ยังไม่ได้ stage
- `dc` — ดู diff ของไฟล์ที่ stage ไว้แล้ว (เทียบเท่า `diff --staged`)
- `aa` — add ทุกไฟล์ที่เปลี่ยนแปลง
- `cm` — commit พร้อมข้อความในคำสั่งเดียว: `git cm "fix bug"`
- `amend` — แก้ไข commit ล่าสุดโดยไม่เปลี่ยนข้อความ (ใช้ตอนลืมใส่ไฟล์บางไฟล์ใน commit ก่อนหน้า)

### สรุปตาราง Alias ยอดนิยมทั้งหมดใน Step นี้

| Alias | คำสั่งเต็ม | ใช้ทำอะไร |
|---|---|---|
| `st` | `status -sb` | ดูสถานะไฟล์แบบย่อ พร้อมชื่อ branch |
| `co` | `checkout` | สลับ/สร้าง branch |
| `br` | `branch` | จัดการ branch |
| `ci` | `commit` | สร้าง commit |
| `unstage` | `reset HEAD --` | เอาไฟล์ออกจาก staging area |
| `last` | `log -1 HEAD` | ดู commit ล่าสุดแบบละเอียด |
| `lg` | `log --graph ...` | ดูประวัติแบบกราฟสวยงาม |
| `df` | `diff` | ดูความเปลี่ยนแปลงที่ยังไม่ stage |
| `dc` | `diff --cached` | ดูความเปลี่ยนแปลงที่ stage แล้ว |
| `amend` | `commit --amend --no-edit` | แก้ commit ล่าสุดแบบไม่เปลี่ยนข้อความ |

---

## Step 124: Alias ขั้นสูงด้วยการเรียก Shell Command (เครื่องหมาย `!`)

### ข้อจำกัดของ Alias แบบธรรมดา

Alias ที่เราสร้างมาทั้งหมดใน Step 123 มีข้อจำกัดสำคัญอย่างหนึ่ง: **มันต้องขึ้นต้นด้วยคำสั่งย่อยของ Git เท่านั้น** (เช่น `status`, `checkout`, `log`) และอาร์กิวเมนต์ที่คุณพิมพ์ตามหลัง alias จะถูก **ต่อท้าย** เข้าไปเสมอ

ตัวอย่างเช่น alias `unstage = reset HEAD --` เวลาคุณพิมพ์ `git unstage file.txt` มันจะกลายเป็น `git reset HEAD -- file.txt` — สังเกตว่า `file.txt` ถูกต่อท้ายเข้าไปพอดี

แต่ถ้าคุณต้องการทำสิ่งที่ซับซ้อนกว่านั้น เช่น:

- รันหลายคำสั่งต่อกัน (`git pull` แล้วตามด้วย `git push`)
- เรียกใช้โปรแกรมภายนอกที่ไม่ใช่ Git (เช่น `npm`, `echo`, `pwd`)
- ควบคุมตำแหน่งของอาร์กิวเมนต์เอง (ไม่ใช่แค่ต่อท้าย)

คุณจะต้องใช้ **เครื่องหมายอัศเจรีย์ (`!`)** นำหน้า

### หลักการทำงานของ `!`

เมื่อ alias ขึ้นต้นด้วย `!` Git จะไม่ตีความมันเป็นคำสั่งย่อยของ Git อีกต่อไป แต่จะส่งข้อความทั้งหมดที่ตามหลัง `!` ไปให้ **shell** รันโดยตรง (ผ่าน `sh -c`)

> ตามเอกสารทางการของ Git (`git help config`): "ถ้าคำที่ใช้ขยาย alias ขึ้นต้นด้วยเครื่องหมายอัศเจรีย์ มันจะถูกปฏิบัติเป็นคำสั่ง shell คำสั่งดังกล่าวจะถูกรันจากโฟลเดอร์ระดับบนสุด (top-level directory) ของ repository ซึ่งอาจไม่ใช่โฟลเดอร์ปัจจุบันที่คุณยืนอยู่ก็ได้"

ข้อสังเกตสำคัญตรงนี้: alias แบบ `!` จะรันจาก **root ของ repository เสมอ** ไม่ใช่จากโฟลเดอร์ย่อยที่คุณอยู่ตอนนั้น ซึ่งต่างจาก alias ธรรมดาที่ไม่สนใจตำแหน่งโฟลเดอร์เลยเพราะมันเป็นแค่คำสั่ง Git

### ตัวอย่างที่ 1: รันหลายคำสั่งต่อกัน

```bash
git config --global alias.sync '!git pull && git push'
```

ใช้งาน:

```bash
git sync
```

ผลลัพธ์คือ Git จะ pull ก่อน แล้วถ้าสำเร็จค่อย push ต่อ (ใช้ `&&` เพื่อให้ push รันเฉพาะตอน pull สำเร็จเท่านั้น ป้องกันการ push ทับข้อมูลที่ยังไม่ได้ sync)

### ตัวอย่างที่ 2: เรียกโปรแกรมภายนอกที่ไม่ใช่ Git

```bash
git config --global alias.visual '!gitk'
git config --global alias.root '!pwd'
```

`git visual` จะเปิดโปรแกรม `gitk` (GUI ดูประวัติ commit ที่มากับ Git) ส่วน `git root` จะพิมพ์ path ปัจจุบันออกมา — สิ่งเหล่านี้ทำไม่ได้เลยถ้าไม่ใช้ `!` เพราะ `gitk` และ `pwd` ไม่ใช่คำสั่งย่อยของ Git

### ตัวอย่างที่ 3: ใช้ร่วมกับเครื่องมืออื่นในโปรเจกต์

```bash
git config --global alias.contributors '!git shortlog -sn --all'
```

`git contributors` จะแสดงรายชื่อผู้ร่วมพัฒนาทั้งหมดเรียงตามจำนวน commit จากมากไปน้อย — มีประโยชน์มากตอนอยากรู้ว่าใครมีส่วนร่วมในโปรเจกต์มากที่สุด

### ตัวอย่างที่ 4: alias ที่รับอาร์กิวเมนต์แบบควบคุมตำแหน่งเอง

นี่คือจุดที่ `!` ทรงพลังมากกว่าปกติ เพราะคุณสามารถเขียน shell function เต็มรูปแบบภายใน alias ได้ ทำให้ควบคุมอาร์กิวเมนต์ `$1`, `$2` ได้อย่างอิสระ ไม่ใช่แค่ต่อท้ายอัตโนมัติ

```bash
git config --global alias.ignore '!f() { echo "$1" >> .gitignore; }; f'
```

ใช้งาน:

```bash
git ignore "*.log"
```

ผลลัพธ์คือ Git จะเพิ่มบรรทัด `*.log` เข้าไปในไฟล์ `.gitignore` ที่ root ของ repository ให้อัตโนมัติ (สร้างไฟล์ใหม่ถ้ายังไม่มี)

อธิบายกลไก: ส่วน `f() { echo "$1" >> .gitignore; }; f` คือการ**ประกาศฟังก์ชันชื่อ `f`** แล้ว**เรียกมันทันที** โดย argument ที่คุณพิมพ์ตามหลัง `git ignore` จะกลายเป็น `$1` ของฟังก์ชันนั้น เทคนิคนี้ (ประกาศฟังก์ชันแล้วเรียกทันที) เป็นแพทเทิร์นมาตรฐานที่ใช้กันทั่วไปเมื่อต้องการ alias ที่ซับซ้อนกว่าการต่อท้ายอาร์กิวเมนต์ธรรมดา

### ตัวอย่างที่ 5: undo commit ล่าสุดแบบเก็บการเปลี่ยนแปลงไว้

```bash
git config --global alias.undo '!git reset --soft HEAD~1'
```

`git undo` จะยกเลิก commit ล่าสุด แต่**เก็บการเปลี่ยนแปลงไว้ใน staging area** ไม่ได้ลบทิ้ง (เราจะเจาะลึกเรื่อง `reset` แบบละเอียดใน Part 14 ที่กำลังจะถึง) alias นี้มีประโยชน์มากเวลา commit ผิดหรือรีบ commit เกินไป

### เปรียบเทียบ Alias ธรรมดา กับ Alias แบบ `!`

| คุณสมบัติ | Alias ธรรมดา | Alias แบบ `!` |
|---|---|---|
| ต้องขึ้นต้นด้วยคำสั่งย่อยของ Git | ใช่ | ไม่จำเป็น |
| รันได้กี่คำสั่ง | คำสั่งเดียว | หลายคำสั่งต่อกันได้ |
| เรียกโปรแกรมนอก Git ได้ไหม | ไม่ได้ | ได้ |
| ตำแหน่งที่รัน | โฟลเดอร์ปัจจุบัน (ผ่าน Git ที่รู้ตำแหน่งอยู่แล้ว) | top-level directory ของ repo |
| ควบคุมตำแหน่งอาร์กิวเมนต์เอง | ไม่ได้ (ต่อท้ายอัตโนมัติเสมอ) | ได้ (ผ่าน `$1`, `$2`, ...) |
| ความซับซ้อนในการเขียน | ต่ำ | สูงกว่า ต้องเข้าใจ shell scripting พื้นฐาน |

### ข้อควรระวังเมื่อใช้ Alias แบบ `!`

1. **ระวังเรื่อง Escape เครื่องหมายคำพูด** — เมื่อ alias มีทั้งเครื่องหมายคำพูดเดี่ยวและคู่ปนกัน ต้อง escape ให้ถูกต้อง ไม่งั้น shell จะตัดคำสั่งผิดตำแหน่ง
2. **จำไว้ว่ามันรันจาก root ของ repo เสมอ** — ถ้า alias ของคุณอ้างอิง path แบบ relative และคุณคาดหวังว่าจะอ้างอิงจากโฟลเดอร์ปัจจุบัน อาจได้ผลลัพธ์ที่ไม่ตรงกับที่คิด
3. **อย่าใส่คำสั่งอันตรายโดยไม่ระวัง** — เพราะ `!` ให้อำนาจเต็มที่เหมือนรัน shell script จึงควรตรวจสอบให้ดีก่อนว่า alias ที่สร้างจะไม่ลบไฟล์หรือ push ทับข้อมูลโดยไม่ตั้งใจ

---

## Step 125: แก้ไข Alias ผ่านไฟล์ `~/.gitconfig` โดยตรงด้วย Text Editor

### ทำไมบางครั้งต้องแก้ไฟล์ตรง ๆ แทนใช้คำสั่ง `git config`

การใช้คำสั่ง `git config --global alias.xxx yyy` ทีละตัวเหมาะกับตอนสร้าง alias ทีละหนึ่งหรือสองตัว แต่เมื่อคุณมี alias จำนวนมาก (10, 20, 30 ตัว) การพิมพ์คำสั่งทีละบรรทัดจะช้าและน่าเบื่อ วิธีที่มืออาชีพนิยมใช้กันคือ **เปิดไฟล์ `~/.gitconfig` ด้วย text editor แล้วแก้ไข/เพิ่มหลายบรรทัดพร้อมกันในครั้งเดียว**

### วิธีเปิดไฟล์ `.gitconfig` เพื่อแก้ไข

Git มีคำสั่งช่วยเปิดไฟล์นี้ให้อัตโนมัติด้วย editor ที่คุณตั้งค่าไว้ (`core.editor`):

```bash
git config --global -e
```

หรือถ้าอยากเปิดด้วย editor ที่ระบุเอง (ไม่ผ่าน `core.editor`) ก็เปิดไฟล์ตรง ๆ ได้เลย:

```bash
code ~/.gitconfig       # เปิดด้วย VS Code
nano ~/.gitconfig        # เปิดด้วย nano
vim ~/.gitconfig         # เปิดด้วย vim
```

### โครงสร้างไฟล์ `.gitconfig` (รูปแบบ INI)

ไฟล์ `.gitconfig` ใช้รูปแบบที่เรียกว่า **INI format** ประกอบด้วย section (คร่อมด้วยวงเล็บเหลี่ยม) และ key-value pair ภายใน section นั้น ตัวอย่างไฟล์ที่มี alias หลายตัว:

```ini
[user]
	name = Somchai Devcode
	email = somchai@example.com

[core]
	editor = code --wait
	excludesFile = ~/.gitignore_global

[alias]
	st = status -sb
	co = checkout
	br = branch
	ci = commit
	unstage = reset HEAD --
	last = log -1 HEAD
	lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
	df = diff
	dc = diff --cached
	amend = commit --amend --no-edit
	sync = !git pull && git push
	undo = !git reset --soft HEAD~1
	contributors = !git shortlog -sn --all

[color]
	ui = auto
```

สังเกตว่าส่วน `[alias]` คือ section เดียวที่รวม alias ทั้งหมดไว้ด้วยกัน แต่ละบรรทัดคือ `<ชื่อ alias> = <คำสั่งที่ขยายออกมา>`

### ข้อดีของการแก้ไฟล์ตรง ๆ

1. **แก้ทีละหลายตัวได้ในครั้งเดียว** — ไม่ต้องรันคำสั่ง `git config` ซ้ำ ๆ หลายสิบครั้ง
2. **ใส่คอมเมนต์อธิบายได้** — ไฟล์ config ของ Git รองรับคอมเมนต์ด้วยเครื่องหมาย `#` หรือ `;`

   ```ini
   [alias]
   	# alias สำหรับดู log แบบกราฟสวยงาม ใช้บ่อยตอน review ประวัติ
   	lg = log --graph --oneline --all --decorate
   ```

3. **คัดลอกจากเครื่องอื่นหรือจาก dotfiles repo ได้ง่าย** — แค่ copy ทั้งไฟล์หรือทั้ง section ไปวางในเครื่องใหม่
4. **จัดกลุ่ม/จัดเรียง alias ให้อ่านง่ายตามหมวดหมู่ที่ต้องการเองได้** เช่น จัดกลุ่ม alias เกี่ยวกับ branch ไว้ด้วยกัน, alias เกี่ยวกับ log ไว้ด้วยกัน

### ข้อควรระวังเมื่อแก้ไฟล์ตรง ๆ

**ความเสี่ยงหลักคือ Syntax Error** — ถ้าคุณพิมพ์ผิด เช่น ลืมปิดเครื่องหมายคำพูด หรือวงเล็บเหลี่ยม section ผิดรูปแบบ **คำสั่ง Git ทุกคำสั่งจะใช้งานไม่ได้ทันที** จนกว่าจะแก้ไฟล์ให้ถูกต้อง เพราะ Git จะพยายามอ่าน config ทุกครั้งที่ถูกเรียกใช้งาน

ตัวอย่าง error ที่จะเจอถ้า syntax ผิด:

```
fatal: bad config line 15 in file /home/user/.gitconfig
```

**วิธีป้องกันและตรวจสอบ:**

1. หลังแก้ไฟล์เสร็จ ให้รันคำสั่งตรวจสอบทันที:

   ```bash
   git config --global --list
   ```

   ถ้าไฟล์มี syntax ผิด คำสั่งนี้จะแสดง error ทันทีแทนที่จะแสดงรายการ config

2. สำรองไฟล์ก่อนแก้ไขใหญ่ ๆ เสมอ:

   ```bash
   cp ~/.gitconfig ~/.gitconfig.backup
   ```

3. ใช้ editor ที่มี syntax highlighting สำหรับไฟล์ INI (VS Code รองรับได้ดีอยู่แล้วโดยไม่ต้องติดตั้ง extension เพิ่ม)

4. ถ้าพลาดจนไฟล์เสียและจำไม่ได้ว่าแก้ตรงไหน ให้กู้คืนจากไฟล์ backup ที่สำรองไว้ในข้อ 2

### แบบฝึกหัดสั้น ๆ

ลองเปิดไฟล์ `~/.gitconfig` ของคุณตอนนี้ด้วยคำสั่ง `git config --global -e` แล้วดูว่า section `[alias]` ที่คุณสร้างไว้ใน Step 122–124 หน้าตาเป็นอย่างไร ลองจัดเรียงลำดับ alias ใหม่ให้เป็นหมวดหมู่ที่คุณอ่านง่ายขึ้น แล้วบันทึก จากนั้นตรวจสอบด้วย `git config --global --list` ว่ายังใช้งานได้ปกติ

---

## Step 126: ลบ Alias ที่ไม่ใช้แล้ว

### ลบ Alias ทีละตัวด้วยคำสั่ง `--unset`

เมื่อคุณสร้าง alias ไปสักพักแล้วพบว่าบางตัวไม่ได้ใช้ หรือตั้งชื่อไม่ดี อยากเปลี่ยนใหม่ วิธีลบที่ปลอดภัยที่สุดคือใช้คำสั่ง:

```bash
git config --global --unset alias.<ชื่อ alias>
```

ตัวอย่าง:

```bash
git config --global --unset alias.ci
```

คำสั่งนี้จะลบบรรทัด `ci = commit` ออกจาก section `[alias]` ในไฟล์ `~/.gitconfig` โดยอัตโนมัติ โดยไม่ต้องเปิดไฟล์แก้เอง

### ตรวจสอบว่าลบสำเร็จแล้ว

```bash
git config --global --get alias.ci
```

ถ้าลบสำเร็จ คำสั่งนี้จะไม่แสดงผลลัพธ์อะไรเลย (และ exit code จะเป็น 1 ซึ่งหมายถึง "ไม่พบค่านี้") ถ้ายังเจอค่าอยู่แสดงว่าลบไม่สำเร็จ หรือยังมี alias ชื่อเดียวกันหลงเหลืออยู่ใน scope อื่น (เช่น system หรือ local)

ลองรัน `git ci` ดูอีกครั้งหลังลบ จะได้ error:

```
git: 'ci' is not a git command. See 'git --help'.
```

### ลบทั้ง Section `[alias]` ในครั้งเดียว

ถ้าต้องการล้าง alias ทั้งหมดที่เคยตั้งไว้ในครั้งเดียว (เช่น อยากเริ่มต้นใหม่) ใช้คำสั่ง:

```bash
git config --global --remove-section alias
```

คำสั่งนี้จะลบทั้ง section `[alias]` ออกจากไฟล์ `~/.gitconfig` ไปเลย รวมถึง alias ทุกตัวที่อยู่ข้างในด้วย **ใช้อย่างระมัดระวัง** เพราะไม่มีการถามยืนยันและไม่มี undo built-in (ถ้าไม่ได้สำรองไฟล์ไว้ก่อน)

### ระวังเรื่อง Scope เวลาลบ

จำหลักการเดิมจาก Step 122 ไว้: ถ้า alias ตัวเดียวกันถูกกำหนดไว้ทั้งใน local และ global การ `--unset` แบบ `--global` จะลบเฉพาะที่อยู่ใน global เท่านั้น ถ้า local ยังมีอยู่ alias นั้นก็ยังทำงานได้อยู่ (เพราะ local มีสิทธิ์เหนือกว่า)

ตัวอย่าง: ถ้าคุณสงสัยว่าทำไมลบ alias ใน global แล้วยังใช้งานได้อยู่ ให้ตรวจสอบว่ามันถูกกำหนดซ้ำใน local ของ repository ปัจจุบันหรือไม่:

```bash
git config --local --get alias.ci
```

ถ้าเจอค่า ให้ลบด้วยคำสั่งเดียวกันแต่เปลี่ยนเป็น `--local`:

```bash
git config --local --unset alias.ci
```

### ตารางสรุปคำสั่งจัดการ Alias ทั้งหมด

| การกระทำ | คำสั่ง |
|---|---|
| สร้าง/แก้ alias (global) | `git config --global alias.<name> "<expansion>"` |
| ดู alias ตัวเดียว | `git config --global --get alias.<name>` |
| ดู alias ทั้งหมด | `git config --get-regexp alias` |
| ลบ alias ตัวเดียว | `git config --global --unset alias.<name>` |
| ลบ alias ทั้งหมด | `git config --global --remove-section alias` |
| แก้ไฟล์ config ตรง ๆ | `git config --global -e` |

---

## Step 127: ไฟล์ Global `.gitignore` และ `.gitconfig` ที่ควรมีติดเครื่องทุกเครื่อง

### ปัญหา: `.gitignore` ของโปรเจกต์ไม่ควรมีขยะส่วนตัวของคุณปนอยู่

`.gitignore` ที่อยู่ใน repository (ไฟล์ที่ commit และแชร์กับทีม) ควรมีแค่รายการไฟล์ที่**เกี่ยวข้องกับโปรเจกต์นั้นโดยเฉพาะ** เช่น โฟลเดอร์ build, ไฟล์ dependency, ไฟล์ environment variable

แต่ในความเป็นจริง เครื่องของคุณอาจสร้างไฟล์ขยะที่**ไม่เกี่ยวกับโปรเจกต์เลย แต่เกี่ยวกับเครื่องมือหรือระบบปฏิบัติการที่คุณใช้ส่วนตัว** เช่น:

- macOS สร้างไฟล์ `.DS_Store` ในทุกโฟลเดอร์
- Windows สร้างไฟล์ `Thumbs.db`
- VS Code สร้างโฟลเดอร์ `.vscode/` (บางครั้งทีมอยากแชร์ แต่บางทีมไม่อยาก)
- JetBrains IDE (IntelliJ, WebStorm, PyCharm) สร้างโฟลเดอร์ `.idea/`
- Vim/Emacs สร้างไฟล์ swap เช่น `*.swp`, `*~`

ถ้าคุณเพิ่มไฟล์เหล่านี้ลงใน `.gitignore` ของทุกโปรเจกต์ที่ทำ จะกลายเป็นภาระซ้ำซ้อน และยังทำให้ `.gitignore` ของโปรเจกต์ (ที่ทีมแชร์กัน) เปื้อนไปด้วยสิ่งที่เกี่ยวกับเครื่องคุณคนเดียว ไม่เกี่ยวกับเพื่อนร่วมทีมที่ใช้เครื่องมือคนละชุด

### ทางออก: Global `.gitignore` (Excludes File)

Git มีกลไกสำหรับ ignore ไฟล์แบบ**เฉพาะเครื่องคุณ** โดยไม่ต้องแตะ `.gitignore` ของ repository เลย เรียกว่า **Global Excludes File**

ขั้นตอนการตั้งค่า:

1. สร้างไฟล์ที่ไหนก็ได้ในเครื่อง (นิยมไว้ที่ home directory):

   ```bash
   touch ~/.gitignore_global
   ```

2. เพิ่มเนื้อหาไฟล์ขยะที่เกี่ยวกับระบบ/เครื่องมือส่วนตัวของคุณ:

   ```gitignore
   # macOS
   .DS_Store
   .AppleDouble
   .LSOverride

   # Windows
   Thumbs.db
   ehthumbs.db
   Desktop.ini

   # Editor / IDE
   .vscode/
   .idea/
   *.swp
   *.swo
   *~

   # Log ทั่วไปที่เกิดจากเครื่องมือ debug ส่วนตัว
   *.local.log
   ```

3. บอก Git ให้ใช้ไฟล์นี้เป็น global excludes:

   ```bash
   git config --global core.excludesFile '~/.gitignore_global'
   ```

จากนี้ไป **ทุก repository ในเครื่องนี้** จะ ignore ไฟล์ตามรายการใน `~/.gitignore_global` โดยอัตโนมัติ โดยที่ไฟล์ `.gitignore` ของ repository เองไม่ต้องรู้เรื่องนี้เลย และเพื่อนร่วมทีมที่ clone repo เดียวกันไปก็จะไม่เห็นรายการเหล่านี้ในของเขา (เพราะมันอยู่ในเครื่องคุณเท่านั้น)

### หมายเหตุ: ค่า Default ของ Git เอง

ถ้าคุณไม่ได้ตั้งค่า `core.excludesFile` เอง Git บางเวอร์ชันจะมีค่า default อยู่แล้วที่ `$XDG_CONFIG_HOME/git/ignore` (โดยทั่วไปคือ `~/.config/git/ignore`) แต่การตั้งชื่อไฟล์เองแบบ `~/.gitignore_global` ที่ตำแหน่งชัดเจน ทำให้จำง่ายและย้ายไป sync กับเครื่องอื่นได้สะดวกกว่า

### `.gitignore` ของ Repo vs Global `.gitignore`: ใครควรมีอะไร

| ประเภทไฟล์ | ควรอยู่ใน `.gitignore` (repo) | ควรอยู่ใน Global `.gitignore_global` |
|---|---|---|
| โฟลเดอร์ build เฉพาะโปรเจกต์ (`dist/`, `build/`) | ใช่ | ไม่ |
| Dependency folder (`node_modules/`, `vendor/`) | ใช่ | ไม่ |
| ไฟล์ environment (`.env`) | ใช่ | ไม่ |
| ไฟล์ระบบปฏิบัติการ (`.DS_Store`, `Thumbs.db`) | ไม่ (ไม่เกี่ยวกับโปรเจกต์) | ใช่ |
| ไฟล์ของ Editor ส่วนตัว (`.vscode/`, `*.swp`) | ขึ้นอยู่กับทีม (บางทีมแชร์ config editor) | ใช่ (ถ้าทีมไม่ได้ตกลงแชร์) |

### เกริ่นแนวคิด Dotfiles

ไฟล์อย่าง `~/.gitconfig` และ `~/.gitignore_global` ที่เราสร้างในบทนี้ จัดอยู่ในกลุ่มไฟล์ที่เรียกกันในวงการว่า **Dotfiles** — คือไฟล์ config ส่วนตัวที่ชื่อขึ้นต้นด้วยจุด (`.`) ซึ่งเก็บการตั้งค่าเฉพาะตัวของโปรแกรมเมอร์แต่ละคน เช่น `.bashrc`, `.zshrc`, `.vimrc`, `.gitconfig`, `.gitignore_global`, `.tmux.conf` เป็นต้น

แนวคิดที่โปรแกรมเมอร์มืออาชีพจำนวนมากทำกันคือ **เก็บไฟล์ dotfiles เหล่านี้ไว้ใน Git repository ของตัวเอง** (มักตั้งชื่อ repo ว่า `dotfiles`) แล้วใช้เครื่องมืออย่าง **GNU Stow** หรือ **chezmoi** ช่วยจัดการการ symlink ไฟล์เหล่านี้ไปวางในตำแหน่งที่ถูกต้องเมื่อตั้งเครื่องใหม่

ประโยชน์ของแนวทางนี้:

1. เมื่อเปลี่ยนเครื่องใหม่หรือ format เครื่อง สามารถ clone repo dotfiles แล้ว "สร้างสภาพแวดล้อมเดิม" กลับมาได้ในไม่กี่นาที
2. มีประวัติการเปลี่ยนแปลง config ของตัวเอง (เพราะมันคือ Git repo) ย้อนดูได้ว่าเคยตั้งค่าอะไรไว้ก่อนหน้า
3. แชร์ config ระหว่างเครื่องหลายเครื่อง (เช่น เครื่องทำงาน กับเครื่องส่วนตัว) ได้ง่ายผ่านการ `git pull`

เรื่อง Dotfiles แบบเต็มรูปแบบ รวมถึงการใช้ GNU Stow และ chezmoi จะถูกอธิบายอย่างละเอียดใน Part ที่เกี่ยวกับ Git ขั้นสูงและ Developer Workflow ช่วงเฟส 6 ของหลักสูตรนี้ ตอนนี้ขอให้จำแค่แนวคิดหลักไว้ก่อน: **ไฟล์ config ส่วนตัวของคุณ ก็สามารถอยู่ภายใต้ Version Control ได้เหมือนกับโค้ดโปรเจกต์**

---

## Step 128: จัดการ Git Config หลายโปรไฟล์ (งาน vs ส่วนตัว) ด้วย Conditional Includes

### ปัญหาที่พบบ่อยมาก: Commit ด้วยอีเมลผิด

โปรแกรมเมอร์จำนวนมากมีโปรเจกต์ทั้งแบบ**งานบริษัท** (ที่ต้องใช้อีเมลบริษัท) และ**โปรเจกต์ส่วนตัว/Open Source** (ที่ใช้อีเมลส่วนตัว) ปะปนกันอยู่บนเครื่องเดียวกัน

ถ้าคุณตั้งค่า `user.name` และ `user.email` แบบ `--global` ตัวเดียว แล้วลืมเปลี่ยนก่อน commit ในอีกบริบทหนึ่ง จะเกิดปัญหาที่น่าอายและแก้ไขยาก เช่น:

- Commit งานบริษัทด้วยอีเมลส่วนตัว ทำให้ระบบตรวจสอบสิทธิ์ของบริษัทมองไม่เห็นว่าเป็น commit ของพนักงาน
- Commit โปรเจกต์ Open Source ด้วยอีเมลบริษัท ทำให้ข้อมูลที่ไม่ควรเปิดเผยสาธารณะรั่วไหลออกไปในประวัติ Git ที่ไม่มีวันลบออกได้ง่าย ๆ

### ทางออก: `includeIf` แบบ Conditional Include

Git รองรับการ**แยก config ออกเป็นหลายไฟล์** แล้วกำหนดเงื่อนไขว่าจะใช้ไฟล์ config ไหน ขึ้นอยู่กับว่า repository นั้นอยู่ใน**โฟลเดอร์** ไหน โดยใช้ syntax `includeIf "gitdir:<pattern>"`

### ขั้นตอนการตั้งค่าจริง

สมมติว่าคุณจัดโครงสร้างโฟลเดอร์แบบนี้:

```
~/work/        ← โปรเจกต์ของบริษัททั้งหมดอยู่ในนี้
~/personal/    ← โปรเจกต์ส่วนตัว/Open Source ทั้งหมดอยู่ในนี้
```

**ขั้นตอนที่ 1:** สร้างไฟล์ config แยกสำหรับแต่ละโปรไฟล์

ไฟล์ `~/.gitconfig-work`:

```ini
[user]
	name = Somchai Devcode
	email = somchai@company.com
```

ไฟล์ `~/.gitconfig-personal`:

```ini
[user]
	name = Somchai Dev
	email = somchai.personal@gmail.com
```

**ขั้นตอนที่ 2:** ในไฟล์ `~/.gitconfig` หลัก เพิ่ม `includeIf` เข้าไป

```ini
[user]
	name = Somchai Devcode
	email = somchai.personal@gmail.com   # ค่า default เผื่อไม่เข้าเงื่อนไขไหนเลย

[includeIf "gitdir:~/work/"]
	path = ~/.gitconfig-work

[includeIf "gitdir:~/personal/"]
	path = ~/.gitconfig-personal
```

### กติกาสำคัญของ `gitdir:` Pattern

1. **ต้องมี `/` ปิดท้ายเสมอ** ถ้าต้องการให้ครอบคลุมทุก repository ภายใต้โฟลเดอร์นั้น เช่น `gitdir:~/work/` จะครอบคลุม `~/work/project-a/`, `~/work/project-b/` และโฟลเดอร์ย่อยทั้งหมด
2. **Path ต้องเป็น absolute path หรือขึ้นต้นด้วย `~/`** (Git จะขยาย `~` เป็น home directory ให้อัตโนมัติ)
3. รองรับ wildcard เช่น `gitdir:~/work/**` เพื่อความยืดหยุ่นเพิ่มเติม
4. ถ้าต้องการให้ pattern **ไม่สนใจตัวพิมพ์เล็ก-ใหญ่** (มีประโยชน์มากบน Windows ที่ path อาจไม่ตรง case กัน) ให้ใช้ `gitdir/i:` แทน `gitdir:`

   ```ini
   [includeIf "gitdir/i:~/Work/"]
   	path = ~/.gitconfig-work
   ```

### ทดสอบว่าทำงานถูกต้อง

หลังตั้งค่าเสร็จแล้ว ให้เข้าไปทดสอบใน repository ที่อยู่ในแต่ละโฟลเดอร์:

```bash
cd ~/work/some-project
git config user.email
# ควรได้: somchai@company.com

cd ~/personal/some-open-source-project
git config user.email
# ควรได้: somchai.personal@gmail.com
```

ถ้าผลลัพธ์ตรงกับที่ตั้งไว้ แปลว่า conditional include ทำงานถูกต้อง — ตอนนี้คุณสามารถ commit ในแต่ละบริบทได้โดยไม่ต้องกังวลว่าจะสลับอีเมลผิดอีกต่อไป เพราะ Git จะเลือกใช้ config ที่ถูกต้องให้อัตโนมัติตามตำแหน่งโฟลเดอร์

### ประยุกต์ใช้ Conditional Include กับค่า Config อื่นนอกจาก User

Conditional include ไม่ได้จำกัดแค่ `user.name`/`user.email` เท่านั้น คุณสามารถแยกค่าอื่นได้ด้วย เช่น การใช้ SSH key คนละดอกสำหรับงานกับส่วนตัว:

ไฟล์ `~/.gitconfig-work`:

```ini
[user]
	name = Somchai Devcode
	email = somchai@company.com

[core]
	sshCommand = "ssh -i ~/.ssh/id_ed25519_work"
```

ไฟล์ `~/.gitconfig-personal`:

```ini
[user]
	name = Somchai Dev
	email = somchai.personal@gmail.com

[core]
	sshCommand = "ssh -i ~/.ssh/id_ed25519_personal"
```

วิธีนี้ทำให้เวลา push/pull ผ่าน SSH ในแต่ละบริบท Git จะเลือกใช้ SSH key ที่ถูกต้องให้อัตโนมัติเช่นกัน (เรื่อง SSH key แบบละเอียดจะอยู่ใน Part ที่ว่าด้วยการเชื่อมต่อ Remote Repository)

### ตารางสรุป Use Case ของ Conditional Include

| สถานการณ์ | ค่าที่ควรแยกโปรไฟล์ |
|---|---|
| อีเมลงาน vs อีเมลส่วนตัว | `user.name`, `user.email` |
| SSH key คนละดอกสำหรับงาน/ส่วนตัว | `core.sshCommand` |
| ลายเซ็น GPG คนละชุด | `user.signingkey`, `commit.gpgsign` |
| Proxy คนละตัวระหว่างเครือข่ายบริษัทกับที่บ้าน | `http.proxy` |

---

## Step 129: Shell Alias/Function เสริมที่ครอบคำสั่ง Git ให้สั้นลงไปอีก

### Git Alias vs Shell Alias: คนละชั้นกัน

จนถึงตอนนี้เราพูดถึง **Git Alias** ทั้งหมด ซึ่งทำงานภายในขอบเขตของ Git เท่านั้น — คุณยังต้องพิมพ์คำว่า `git` นำหน้าเสมอ (เช่น `git st`, `git co`)

แต่ยังมีอีกชั้นหนึ่งที่อยู่**เหนือ Git ขึ้นไป** นั่นคือ **Shell Alias/Function** ซึ่งเป็นกลไกของ shell เอง (Bash หรือ Zsh) ไม่เกี่ยวข้องกับ Git โดยตรง แต่ใช้ครอบคำสั่ง Git (หรือคำสั่งอะไรก็ได้) ให้สั้นลงไปอีกขั้นหนึ่ง

จุดต่างที่สำคัญ: Shell alias ทำให้คุณ**ไม่ต้องพิมพ์คำว่า `git` เลยด้วยซ้ำ**

### ตั้ง Shell Alias พื้นฐานใน Bash หรือ Zsh

ไฟล์ที่ต้องแก้ไขขึ้นอยู่กับว่าคุณใช้ shell ตัวไหน:

| Shell | ไฟล์ Config |
|---|---|
| Bash | `~/.bashrc` (Linux) หรือ `~/.bash_profile` (macOS) |
| Zsh | `~/.zshrc` |

เปิดไฟล์แล้วเพิ่มบรรทัดเหล่านี้ต่อท้าย:

```bash
alias gs='git status'
alias ga='git add'
alias gaa='git add --all'
alias gc='git commit'
alias gp='git push'
alias gl='git pull'
alias gco='git checkout'
alias gb='git branch'
```

หลังแก้ไฟล์เสร็จ ต้อง **reload shell** เพื่อให้การเปลี่ยนแปลงมีผล:

```bash
source ~/.bashrc    # สำหรับ Bash
source ~/.zshrc      # สำหรับ Zsh
```

หรือปิด terminal แล้วเปิดใหม่ก็ได้เช่นกัน

ทดสอบใช้งาน:

```bash
gs      # เทียบเท่ากับ git status
gaa     # เทียบเท่ากับ git add --all
gp      # เทียบเท่ากับ git push
```

### Shell Function สำหรับกรณีที่ต้องรับอาร์กิวเมนต์ซับซ้อนกว่า Alias ธรรมดา

Shell alias ธรรมดา (`alias`) เหมาะกับคำสั่งที่ไม่ต้องมีการประมวลผลอาร์กิวเมนต์ซับซ้อน แต่ถ้าต้องการรวมหลายคำสั่งเข้าด้วยกันแบบมี logic ต้องใช้ **Shell Function** แทน

ตัวอย่าง function ที่รวม `add` และ `commit` เป็นคำสั่งเดียว:

```bash
gac() {
  git add --all
  git commit -m "$1"
}
```

ใช้งาน:

```bash
gac "fix: แก้บั๊กหน้า login"
```

ผลลัพธ์คือมันจะ `git add --all` แล้วตามด้วย `git commit -m "fix: แก้บั๊กหน้า login"` ในคำสั่งเดียว

อีกตัวอย่างที่นิยมมาก คือ function สำหรับสร้าง branch ใหม่แล้วสลับไปทันที พร้อม push branch นั้นขึ้น remote และตั้ง upstream ให้อัตโนมัติ:

```bash
gnb() {
  git checkout -b "$1"
  git push -u origin "$1"
}
```

ใช้งาน:

```bash
gnb feature/new-checkout-flow
```

จะสร้าง branch `feature/new-checkout-flow`, สลับไปที่ branch นั้น, push ขึ้น remote และตั้งค่า tracking (upstream) ให้ในคำสั่งเดียว — งานที่ปกติต้องพิมพ์ 2 คำสั่งแยกกัน (`git checkout -b` แล้ว `git push -u origin ...`) ทำเสร็จในคำสั่งเดียว

### ทำไม Shell Alias/Function ถึงต่างจาก Git Alias ในเชิงความสามารถ

| ความสามารถ | Git Alias | Shell Alias/Function |
|---|---|---|
| ต้องพิมพ์ `git` นำหน้าไหม | ต้อง | ไม่ต้อง |
| ใช้ได้กับคำสั่งนอก Git ไหม (ผ่าน `!`) | ได้ (แบบจำกัด) | ได้เต็มรูปแบบ (เพราะเป็น shell native) |
| Sync ข้ามเครื่องผ่าน dotfiles | ผ่านไฟล์ `.gitconfig` | ผ่านไฟล์ `.bashrc`/`.zshrc` |
| ความซับซ้อนของ logic ที่รองรับ | จำกัดกว่า (ต้องพึ่ง `!` + shell function ซ้อนอีกที) | ยืดหยุ่นเต็มที่ (เขียน Bash/Zsh script เต็มรูปแบบได้เลย) |
| พกพาข้าม shell ต่างชนิด (bash ↔ zsh ↔ fish) | พกพาได้ (เพราะเก็บใน `.gitconfig` ซึ่งไม่ผูกกับ shell) | ต้องปรับ syntax ตาม shell ที่ใช้ |

จากตารางจะเห็นว่าทั้งสองแบบมีจุดแข็งต่างกัน ในทางปฏิบัติโปรแกรมเมอร์มืออาชีพจำนวนมากจึงใช้ **ทั้งสองแบบผสมกัน**: ใช้ Git alias สำหรับสิ่งที่อยากให้พกพาข้ามเครื่อง/ข้าม shell ได้ง่าย และใช้ shell alias/function สำหรับ workflow เฉพาะตัวที่ซับซ้อนกว่าและอยากให้สั้นที่สุดเท่าที่จะทำได้

### กล่าวถึง Oh My Zsh Git Plugin (ระบบสำเร็จรูปที่ใช้กันแพร่หลาย)

สำหรับคนที่ใช้ Zsh ร่วมกับเฟรมเวิร์ก **Oh My Zsh** มี plugin ชื่อ `git` ที่มากับชุด shell alias สำเร็จรูปกว่า 100 ตัวติดตั้งมาให้ทันที เช่น `gst` (git status), `gaa` (git add --all), `gcmsg` (git commit -m), `gp` (git push), `gl` (git pull), `gco` (git checkout) เป็นต้น

การเปิดใช้งาน plugin นี้ทำได้โดยแก้ไฟล์ `~/.zshrc`:

```bash
plugins=(git)
```

นี่เป็นตัวอย่างที่ดีว่าแนวคิด alias ที่เราเรียนในบทนี้ถูกนำไปทำเป็นเครื่องมือสำเร็จรูปให้ใช้งานได้ทันทีโดยไม่ต้องตั้งเองทั้งหมด แต่การเข้าใจกลไกเบื้องหลัง (อย่างที่เราเรียนมาในทุก Step ก่อนหน้า) ยังคงสำคัญ เพราะทำให้คุณสามารถปรับแต่ง ตรวจสอบ หรือแก้ปัญหาเมื่อ alias สำเร็จรูปเหล่านี้ทำงานไม่ตรงกับที่คาดหวังได้

---

## Step 130: แบบฝึกหัด — สร้างชุด Alias ส่วนตัวและทดสอบใช้งานจริง

ถึงเวลาลงมือทำจริงแล้ว แบบฝึกหัดนี้จะให้คุณสร้างชุด alias ส่วนตัวอย่างน้อย 8 ตัว ผสมทั้ง Git alias ธรรมดา, Git alias แบบ `!`, และ shell alias/function จากนั้นทดสอบใช้งานจริงในโปรเจกต์ทดลอง

### ขั้นตอนที่ 1: เตรียมโปรเจกต์ทดลอง

```bash
cd ~/git-course
mkdir part-13-alias-lab && cd part-13-alias-lab
git init
echo "# Alias Lab" > README.md
git add README.md
git commit -m "init: เริ่มโปรเจกต์ทดลอง alias"
```

### ขั้นตอนที่ 2: สร้าง Git Alias อย่างน้อย 5 ตัว

ตั้งเป้าให้ครอบคลุมทั้งแบบธรรมดาและแบบ `!` อย่างน้อยแบบละ 1 ตัว ตัวอย่างชุดที่แนะนำ (ปรับชื่อให้เข้ากับสไตล์ตัวเองได้):

```bash
git config --global alias.st 'status -sb'
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
git config --global alias.sync '!git pull && git push'
git config --global alias.undo '!git reset --soft HEAD~1'
```

นับดูจะได้ 8 ตัวพอดี ครอบคลุมทั้งสถานะ (`st`), การสลับ branch (`co`, `br`), การจัดการ staging (`unstage`), การดูประวัติ (`last`, `lg`), และแบบ shell command (`sync`, `undo`)

### ขั้นตอนที่ 3: ทดสอบ Alias ทีละตัวในโปรเจกต์จริง

```bash
# ทดสอบ st
echo "console.log('hello');" > app.js
git st
# ควรเห็นสถานะแบบย่อพร้อมชื่อ branch

# ทดสอบ br และ co
git br -M main            # เปลี่ยนชื่อ branch หลักเป็น main (ถ้ายังไม่ใช่)
git co -b feature/logging
git br
# ควรเห็นรายการ branch พร้อม * หน้า branch ปัจจุบัน

# ทดสอบ unstage
git add app.js
git st                     # ควรเห็น app.js อยู่ใน staged
git unstage app.js
git st                     # ควรเห็น app.js กลับไปเป็น unstaged

# ทดสอบ last
git add app.js
git ci -m "add: เพิ่ม console.log ทดสอบ"
git last

# ทดสอบ lg
echo "console.log('world');" >> app.js
git ci -am "update: เพิ่มบรรทัด log อีกบรรทัด"
git lg

# ทดสอบ undo
echo "temp" > temp.txt
git add temp.txt
git ci -m "commit ที่พิมพ์ผิดโดยไม่ตั้งใจ"
git undo
git st
# ควรเห็นว่า commit ล่าสุดถูกยกเลิก แต่ temp.txt ยังอยู่ใน staging area
```

> **หมายเหตุ:** alias `sync` ต้องมี remote repository ตั้งไว้แล้วถึงจะทดสอบได้เต็มรูปแบบ (เพราะมันต้อง `pull` และ `push` จริง) ถ้ายังไม่ได้เรียนเรื่อง remote ในหลักสูตรนี้ ให้ข้ามการทดสอบตัวนี้ไปก่อน แล้วกลับมาทดสอบอีกครั้งหลังเรียน Part ที่ว่าด้วย Remote Repository

### ขั้นตอนที่ 4: เพิ่ม Shell Alias/Function อย่างน้อย 2 ตัว

เปิดไฟล์ `~/.bashrc` หรือ `~/.zshrc` แล้วเพิ่ม:

```bash
alias gs='git status -sb'

gac() {
  git add --all
  git commit -m "$1"
}
```

Reload shell แล้วทดสอบ:

```bash
source ~/.zshrc   # หรือ ~/.bashrc ตาม shell ที่ใช้

gs

echo "test shell alias" >> app.js
gac "test: ทดสอบ shell function gac"
git last
```

### ขั้นตอนที่ 5: ตรวจสอบผลลัพธ์ทั้งหมดด้วยการดูไฟล์ config

ปิดท้ายด้วยการเปิดไฟล์ `~/.gitconfig` เพื่อดูว่า section `[alias]` มีรายการครบตามที่ตั้งใจไว้หรือไม่:

```bash
git config --global -e
```

และตรวจสอบรายการทั้งหมดผ่านคำสั่ง:

```bash
git config --get-regexp alias
```

### เกณฑ์ความสำเร็จของแบบฝึกหัดนี้

- [ ] มี Git alias อย่างน้อย 8 ตัว ครอบคลุมทั้งแบบธรรมดาและแบบ `!`
- [ ] ทดสอบใช้งานจริงกับ commit/branch/log ในโปรเจกต์ทดลองได้ครบทุกตัว (ยกเว้น alias ที่ต้องพึ่ง remote)
- [ ] มี Shell alias หรือ function อย่างน้อย 2 ตัวใน `~/.bashrc` หรือ `~/.zshrc`
- [ ] อธิบายได้ว่า alias แต่ละตัวที่สร้างขึ้น เทียบเท่ากับคำสั่งเต็มอะไร
- [ ] เข้าใจว่าทำไม alias แบบ `!` ถึงรันจาก root ของ repository เสมอ ไม่ใช่จากโฟลเดอร์ปัจจุบัน

---

## สรุป Part 13

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Alias คือทางลัดที่ช่วยเพิ่ม Productivity จริง** ทั้งในแง่ลดตัวอักษรที่ต้องพิมพ์ ลดโอกาสพิมพ์ผิด และรักษา Flow State ระหว่างทำงาน แต่ไม่ควรใช้แทนความเข้าใจคำสั่งเต็ม
2. **สร้าง Git Alias พื้นฐานได้ด้วย** `git config --global alias.<ชื่อ> <คำสั่ง>` และเข้าใจลำดับความสำคัญของ scope (local > global > system)
3. **Alias ยอดนิยมที่โปรแกรมเมอร์มืออาชีพใช้จริง** เช่น `co`, `br`, `ci`, `unstage`, `last`, และ `lg` สำหรับดู log แบบกราฟสวยงาม
4. **Alias ที่ขึ้นต้นด้วย `!`** ทำให้เรียก shell command ได้เต็มรูปแบบ รันหลายคำสั่งต่อกันได้ เรียกโปรแกรมนอก Git ได้ และควบคุมอาร์กิวเมนต์เองได้ด้วยการเขียน shell function ภายใน alias — แต่ต้องจำไว้ว่ามันรันจาก top-level directory ของ repo เสมอ
5. **แก้ไข `~/.gitconfig` ตรง ๆ ด้วย text editor** ได้เมื่อต้องจัดการ alias จำนวนมาก โดยต้องระวังเรื่อง syntax error ที่จะทำให้ Git ใช้งานไม่ได้ทั้งหมดจนกว่าจะแก้ไข
6. **ลบ Alias ได้ด้วย** `git config --global --unset alias.<ชื่อ>` หรือ ลบทั้ง section ด้วย `--remove-section alias`
7. **Global `.gitignore`** (`core.excludesFile`) แยกไฟล์ขยะเฉพาะเครื่อง/เครื่องมือส่วนตัวออกจาก `.gitignore` ของโปรเจกต์ที่ทีมแชร์กัน และเริ่มรู้จักแนวคิด Dotfiles สำหรับเก็บไฟล์ config ส่วนตัวไว้ใน Version Control
8. **Conditional Include (`includeIf "gitdir:"`)** ช่วยแยก config ระหว่างโปรไฟล์งานกับโปรไฟล์ส่วนตัวได้อัตโนมัติตามตำแหน่งโฟลเดอร์ ป้องกันปัญหา commit ด้วยอีเมลผิด
9. **Shell Alias/Function** เป็นอีกชั้นหนึ่งที่อยู่เหนือ Git Alias ทำให้ไม่ต้องพิมพ์คำว่า `git` เลยด้วยซ้ำ และรองรับ logic ที่ซับซ้อนกว่าผ่านการเขียน function เต็มรูปแบบใน `.bashrc`/`.zshrc`
10. ลงมือสร้างชุด alias ส่วนตัวอย่างน้อย 8 ตัวและทดสอบใช้งานจริงแล้วในแบบฝึกหัดท้ายบท

### Checklist ก่อนไป Part 14

- [ ] เข้าใจความแตกต่างระหว่าง Git Alias, Git Alias แบบ `!`, และ Shell Alias/Function
- [ ] สร้างและลบ Git alias ได้คล่องทั้งผ่านคำสั่ง `git config` และการแก้ไฟล์ `.gitconfig` ตรง ๆ
- [ ] ตั้งค่า Global `.gitignore` (`core.excludesFile`) ติดเครื่องแล้ว
- [ ] เข้าใจและ (ถ้ามีสถานการณ์จริง) ตั้งค่า Conditional Include แยกโปรไฟล์งาน/ส่วนตัวได้
- [ ] มี Shell alias/function ของตัวเองอย่างน้อย 2 ตัวใช้งานอยู่จริง
- [ ] ทำแบบฝึกหัด Step 130 ครบทุกข้อ และมี alias อย่างน้อย 8 ตัวพร้อมใช้งานจริง

**ต่อไป:** [Part 14: การ Undo การเปลี่ยนแปลง: checkout, restore, reset](./part-014-undo-checkout-restore-reset.md)
