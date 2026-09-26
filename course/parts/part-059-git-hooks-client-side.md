# Part 59: Git Hooks: Client-side Hooks

> **Step ในหลักสูตรนี้:** Step 581–590
> **เฟส:** 6 — Git ขั้นสูงระดับ Internals, Hooks, LFS, Performance
> **เป้าหมายของ Part นี้:** เข้าใจว่า Git Hooks คืออะไร ทำงานอย่างไรในระดับกลไกจริง เรียนรู้ hook ฝั่ง client ที่สำคัญที่สุด (`pre-commit`, `commit-msg`, `prepare-commit-msg`, `post-commit`, `pre-push`) เขียน hook script ด้วย bash ที่ใช้งานได้จริง เข้าใจปัญหาเรื่องการแชร์ hook ในทีมและวิธีแก้ด้วย husky/pre-commit framework/`core.hooksPath` และลงมือสร้าง hook จริงสำหรับโปรเจกต์ทดลอง

---

## สารบัญของ Part นี้

- Step 581: Git Hooks คืออะไร อยู่ที่ไหน ทำงานอย่างไร
- Step 582: `pre-commit` hook — ตรวจสอบก่อน commit (lint, format)
- Step 583: `commit-msg` hook — ตรวจสอบ/แก้ไข commit message
- Step 584: `prepare-commit-msg` hook — เติม template ลง commit message อัตโนมัติ
- Step 585: `post-commit` hook — ทำงานหลัง commit สำเร็จ
- Step 586: `pre-push` hook — ตรวจสอบก่อน push (รัน test ทั้งหมด)
- Step 587: การเขียน hook script ด้วย bash จริงที่ใช้งานได้ (shebang, exit code)
- Step 588: ปัญหา hook ไม่ถูกแชร์ผ่าน git และวิธีแก้ (husky, pre-commit framework, `core.hooksPath`)
- Step 589: การข้าม hook ชั่วคราวด้วย `--no-verify`
- Step 590: แบบฝึกหัด — เขียน pre-commit และ commit-msg hook จริงให้โปรเจกต์ทดลอง

---

## Step 581: Git Hooks คืออะไร อยู่ที่ไหน ทำงานอย่างไร

### Git Hooks คืออะไร

**Git Hooks** คือ **สคริปต์ (script)** ที่ Git จะ **รันโดยอัตโนมัติ** ในจุดต่าง ๆ ของ workflow เช่น ก่อน commit, หลัง commit, ก่อน push, หลัง merge เป็นต้น

พูดง่าย ๆ Hook คือ **"จุดเกี่ยวเกี่ยว (hook point)"** ที่ Git เปิดให้คุณ **แทรกโค้ดของตัวเองเข้าไปทำงาน** ในขั้นตอนที่ Git กำลังจะทำอะไรบางอย่าง โดยไม่ต้องแก้โค้ดของ Git เอง

ตัวอย่างการใช้งานจริงที่พบบ่อยที่สุด:

- ก่อน commit ให้รัน linter ตรวจสอบโค้ด ถ้าไม่ผ่านห้าม commit
- ก่อน commit ให้ format โค้ดอัตโนมัติด้วย Prettier/Black
- ตรวจสอบว่า commit message เขียนตามมาตรฐาน Conventional Commits หรือไม่
- ก่อน push ให้รัน test suite ทั้งหมด ถ้า test fail ห้าม push
- หลัง commit สำเร็จ ให้ส่งข้อความแจ้งเตือนเข้า Slack

### Hook อยู่ที่ไหน

ทุก Git repository จะมีโฟลเดอร์ **`.git/hooks/`** อยู่เสมอ (สร้างขึ้นอัตโนมัติตอน `git init` หรือ `git clone`) ลองดูได้จริง:

```bash
cd ~/git-course
mkdir hooks-demo && cd hooks-demo
git init

ls -la .git/hooks/
```

ผลลัพธ์ที่เห็นจะประมาณนี้:

```
applypatch-msg.sample
commit-msg.sample
fsmonitor-watchman.sample
post-update.sample
pre-applypatch.sample
pre-commit.sample
pre-merge-commit.sample
pre-push.sample
pre-rebase.sample
pre-receive.sample
prepare-commit-msg.sample
push-to-checkout.sample
sendemail-validate.sample
update.sample
```

สังเกตว่าไฟล์ทุกไฟล์ลงท้ายด้วย **`.sample`** — นี่คือไฟล์ตัวอย่างที่ Git ติดตั้งมาให้ดูเป็นแนวทาง **แต่ไม่ทำงาน** เพราะนามสกุล `.sample` ทำให้ Git มองข้ามไป

### จะทำให้ hook ทำงานได้อย่างไร

กฎมีแค่ 2 ข้อ:

1. **ชื่อไฟล์ต้องตรงกับชื่อ hook เป๊ะ ๆ โดยไม่มีนามสกุลใด ๆ ต่อท้าย** เช่น `pre-commit` ไม่ใช่ `pre-commit.sh` หรือ `pre-commit.sample`
2. **ไฟล์ต้องมีสิทธิ์ให้ execute ได้ (executable permission)** — บน Linux/macOS ต้องรัน `chmod +x` ก่อน

ตัวอย่างการเปิดใช้งาน hook ตัวอย่างที่มากับ Git:

```bash
cp .git/hooks/pre-commit.sample .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

จากนี้ทุกครั้งที่คุณสั่ง `git commit` Git จะรันไฟล์ `.git/hooks/pre-commit` โดยอัตโนมัติก่อนสร้าง commit

### รายชื่อ Hook ทั้งหมดที่ Git รองรับ (แบ่งตามฝั่ง)

Git แบ่ง hook ออกเป็น 2 ฝั่งใหญ่:

| ฝั่ง | ทำงานที่ไหน | ตัวอย่าง hook |
|---|---|---|
| **Client-side hooks** | ทำงานบนเครื่องของ developer เอง เวลา commit, merge, rebase, push | `pre-commit`, `commit-msg`, `prepare-commit-msg`, `post-commit`, `pre-push`, `post-checkout`, `post-merge` |
| **Server-side hooks** | ทำงานบนเซิร์ฟเวอร์ที่รับ push เข้ามา | `pre-receive`, `update`, `post-receive` |

Part นี้จะโฟกัสที่ **Client-side hooks** ทั้งหมด ส่วน Server-side hooks จะเรียนต่อใน **Part 60**

### ลำดับ hook ที่เกี่ยวกับ commit (สำคัญมาก ต้องจำลำดับให้ได้)

เวลาคุณพิมพ์ `git commit` Git จะเรียก hook ตามลำดับนี้:

```
git commit
   │
   ▼
┌─────────────────┐
│  pre-commit      │  ← ตรวจสอบก่อน (lint, format) — หยุดได้ถ้า exit ≠ 0
└────────┬─────────┘
         ▼
┌─────────────────────┐
│ prepare-commit-msg   │  ← เติม/แก้ template ของ commit message ก่อนเปิด editor
└────────┬─────────────┘
         ▼
   (เปิด editor ให้ผู้ใช้พิมพ์ message ถ้าไม่ได้ใช้ -m)
         ▼
┌─────────────────┐
│  commit-msg      │  ← ตรวจสอบ/แก้ไข commit message ที่พิมพ์เสร็จแล้ว — หยุดได้ถ้า exit ≠ 0
└────────┬─────────┘
         ▼
   (Git สร้าง commit object จริง)
         ▼
┌─────────────────┐
│  post-commit     │  ← ทำงานหลัง commit สำเร็จแล้ว (แจ้งเตือน ฯลฯ) — หยุด commit ไม่ได้แล้ว
└─────────────────┘
```

ส่วน `pre-push` จะทำงานแยกออกไปตอนสั่ง `git push` (คนละจุดกับ commit)

### สิ่งสำคัญที่ต้องเข้าใจเกี่ยวกับ Hook ตั้งแต่ต้น

1. **Hook คือแค่ไฟล์ script ธรรมดา** — เขียนเป็น bash, Python, Ruby, JavaScript หรือภาษาอะไรก็ได้ ขอแค่ไฟล์นั้น executable และมี shebang บอกว่าจะรันด้วย interpreter ตัวไหน
2. **Exit code คือกลไกสื่อสารเดียวที่ Git สนใจ** — hook exit code `0` = สำเร็จ ให้ Git ทำงานต่อ / exit code ที่ไม่ใช่ `0` = ล้มเหลว ให้ Git **หยุด** การกระทำนั้นทันที (สำหรับ hook ที่ตรวจสอบก่อนการกระทำ เช่น `pre-commit`, `commit-msg`, `pre-push`)
3. **Hook อยู่ใน `.git/hooks/` ซึ่งเป็นโฟลเดอร์ที่ไม่ถูก track โดย Git โดย default** — นี่คือปัญหาสำคัญที่เราจะพูดถึงใน Step 588
4. **Hook รันบนเครื่องที่มันติดตั้งอยู่เท่านั้น** — ถ้าคุณเขียน `pre-commit` hook ในเครื่องตัวเอง แล้ว push ขึ้น GitHub เพื่อนร่วมทีมที่ clone repo ไปจะ**ไม่ได้** hook นั้นติดไปด้วยโดยอัตโนมัติ

---

## Step 582: `pre-commit` hook — ตรวจสอบก่อน commit (lint, format, ห้าม commit ถ้าไม่ผ่าน)

### `pre-commit` ทำงานเมื่อไหร่

`pre-commit` hook จะถูกเรียกทันทีที่คุณสั่ง `git commit` **ก่อน** ที่ Git จะเปิด editor ให้พิมพ์ commit message ด้วยซ้ำ และ**ก่อน**ที่ commit object จะถูกสร้างขึ้นจริง

นี่คือจุดที่เหมาะที่สุดสำหรับ:

- รัน **linter** (ESLint, Pylint, RuboCop) ตรวจสอบว่าโค้ดที่กำลังจะ commit มี syntax error หรือละเมิด code style หรือไม่
- รัน **formatter** (Prettier, Black, gofmt) เพื่อจัดรูปแบบโค้ดให้ตรงมาตรฐานก่อน commit
- ตรวจสอบว่าไม่มีการ commit ไฟล์ที่ไม่ควร commit เช่น `.env`, ไฟล์ credential, ไฟล์ขนาดใหญ่เกินไป
- ตรวจสอบว่าไม่มี debug statement หลงเหลืออยู่ เช่น `console.log`, `debugger`, `print()` ที่ลืมลบ
- รัน unit test แบบเร็ว (fast tests เท่านั้น เพราะ `pre-commit` ควรเร็ว ไม่ควรทำให้ developer รอนาน)

### กลไกสำคัญ: `git commit` จะเห็นเฉพาะไฟล์ที่ staged เท่านั้น

Hook `pre-commit` ไม่ได้รับ argument ใด ๆ ส่งเข้ามา สิ่งที่มันทำได้คือตรวจสอบ **staging area (index)** ผ่านคำสั่ง Git ปกติ เช่น `git diff --cached`

```bash
# ดูว่าไฟล์ไหนถูก stage ไว้เพื่อจะ commit
git diff --cached --name-only

# ดู diff ของเนื้อหาที่ staged เพื่อจะ commit
git diff --cached
```

นี่คือหัวใจสำคัญ: hook ควรตรวจสอบ **สิ่งที่กำลังจะ commit เท่านั้น** ไม่ใช่ตรวจทั้งโปรเจกต์ เพราะ developer อาจมีไฟล์ที่แก้ค้างไว้ (unstaged) ที่ยังไม่พร้อม commit อยู่ในโฟลเดอร์ด้วย

### ตัวอย่างที่ 1: `pre-commit` เช็คห้าม commit ไฟล์ `.env`

```bash
#!/bin/bash
# .git/hooks/pre-commit

# ดึงรายชื่อไฟล์ที่ staged ทั้งหมด
staged_files=$(git diff --cached --name-only --diff-filter=ACM)

for file in $staged_files; do
    if [[ "$file" == *.env ]] || [[ "$file" == ".env" ]]; then
        echo "❌ ห้าม commit ไฟล์ .env เด็ดขาด: $file"
        echo "   ไฟล์นี้อาจมีข้อมูลลับ (secret/credential) กรุณา unstage ก่อน commit:"
        echo "   git restore --staged $file"
        exit 1
    fi
done

exit 0
```

สังเกตว่า:

- ใช้ `--diff-filter=ACM` เพื่อกรองเฉพาะไฟล์ที่ **A**dded, **C**opied, **M**odified (ไม่รวมไฟล์ที่ถูกลบ เพราะไฟล์ที่ถูกลบไม่มีเนื้อหาให้ตรวจ)
- ถ้าเจอไฟล์ `.env` จะ `exit 1` ทำให้ Git **หยุด commit ทันที** และแสดงข้อความ error นั้นให้ผู้ใช้เห็น
- ถ้าไม่เจอปัญหาอะไรเลย จบด้วย `exit 0` เพื่อให้ Git ทำ commit ต่อไปตามปกติ

### ตัวอย่างที่ 2: `pre-commit` รัน linter จริง (ESLint)

```bash
#!/bin/bash
# .git/hooks/pre-commit

# ดึงเฉพาะไฟล์ .js/.ts ที่ staged
staged_js_files=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(js|ts|jsx|tsx)$')

if [ -z "$staged_js_files" ]; then
    # ไม่มีไฟล์ JS/TS ให้ตรวจ ผ่านไปเลย
    exit 0
fi

echo "🔍 กำลังรัน ESLint ตรวจสอบไฟล์ที่ staged..."

npx eslint $staged_js_files

if [ $? -ne 0 ]; then
    echo "❌ ESLint พบปัญหา กรุณาแก้ไขก่อน commit"
    exit 1
fi

echo "✅ ESLint ผ่านหมด"
exit 0
```

### ตัวอย่างที่ 3: `pre-commit` ห้าม commit debug statement

```bash
#!/bin/bash
# .git/hooks/pre-commit

# ค้นหา console.log หรือ debugger ที่ถูกเพิ่มเข้ามาใหม่ในไฟล์ staged
staged_js_files=$(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(js|ts)$')

found_issue=0

for file in $staged_js_files; do
    # ดูเฉพาะบรรทัดที่ถูก "เพิ่มใหม่" (ขึ้นต้นด้วย +) ในไฟล์นี้
    matches=$(git diff --cached -- "$file" | grep -E '^\+.*\b(console\.log|debugger)\b')
    if [ -n "$matches" ]; then
        echo "⚠️  พบ console.log/debugger ที่เพิ่มใหม่ในไฟล์ $file:"
        echo "$matches"
        found_issue=1
    fi
done

if [ "$found_issue" -eq 1 ]; then
    echo "❌ กรุณาลบ console.log/debugger ออกก่อน commit"
    exit 1
fi

exit 0
```

### ข้อควรระวังสำคัญ: `pre-commit` ต้องเร็ว

Hook นี้ทำงานทุกครั้งที่ commit ถ้ามันช้า (เช่น รัน test suite เต็มรูปแบบที่ใช้เวลาหลายนาที) developer จะเริ่มรำคาญและมักหาทางข้ามมันไปด้วย `--no-verify` (จะพูดถึงใน Step 589) ทำให้ hook เสียประโยชน์ไปเลย หลักการที่ดีคือ:

- `pre-commit` ควรตรวจสอบแค่ **ไฟล์ที่ staged** ไม่ใช่ทั้งโปรเจกต์
- ใช้ **linter/formatter แบบ incremental** ที่ตรวจเฉพาะไฟล์ที่เปลี่ยน
- งานหนัก ๆ อย่าง full test suite ควรไปอยู่ที่ `pre-push` หรือ CI แทน

---

## Step 583: `commit-msg` hook — ตรวจสอบ/แก้ไข commit message

### `commit-msg` ทำงานเมื่อไหร่ และรับ argument อะไร

`commit-msg` hook ทำงาน **หลังจาก** ผู้ใช้พิมพ์ commit message เสร็จแล้ว (ไม่ว่าจะพิมพ์ผ่าน editor หรือใช้ `-m`) แต่**ก่อน**ที่ commit object จริงจะถูกสร้างขึ้น

จุดสำคัญที่ต่างจาก `pre-commit` คือ **`commit-msg` รับ argument 1 ตัว** คือ **path ของไฟล์ชั่วคราว** ที่เก็บข้อความ commit message ที่ผู้ใช้พิมพ์ไว้ (ปกติคือ `.git/COMMIT_EDITMSG`)

```bash
#!/bin/bash
# .git/hooks/commit-msg
# $1 = path ของไฟล์ที่เก็บ commit message

commit_msg_file="$1"
commit_msg=$(cat "$commit_msg_file")

echo "ข้อความ commit ที่ได้รับ: $commit_msg"
```

### ทำไม `commit-msg` ถึงเหมาะกับการตรวจสอบรูปแบบข้อความ

เพราะมันมี **เนื้อหาข้อความเต็ม ๆ** ให้ตรวจสอบแล้ว ต่างจาก `pre-commit` ที่ยังไม่รู้เลยว่าผู้ใช้จะพิมพ์ข้อความอะไร

### ตัวอย่าง: `commit-msg` เช็ครูปแบบ Conventional Commits

ต่อยอดจากที่เรียนใน **Part 35: Naming Convention & Conventional Commits** มาเขียน hook จริงที่บังคับใช้มาตรฐานนี้:

```bash
#!/bin/bash
# .git/hooks/commit-msg

commit_msg_file="$1"
commit_msg=$(head -1 "$commit_msg_file")

# รูปแบบ Conventional Commits: type(scope)?: description
pattern="^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\([a-z0-9_-]+\))?(!)?: .{1,100}$"

if ! [[ "$commit_msg" =~ $pattern ]]; then
    echo "❌ Commit message ไม่ตรงตามรูปแบบ Conventional Commits"
    echo ""
    echo "   รูปแบบที่ถูกต้อง: <type>(<scope>): <description>"
    echo "   ตัวอย่าง: feat(auth): เพิ่มระบบ login ด้วย OAuth"
    echo ""
    echo "   type ที่รองรับ: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert"
    echo ""
    echo "   ข้อความที่คุณพิมพ์: \"$commit_msg\""
    exit 1
fi

exit 0
```

ทดสอบ:

```bash
git commit -m "แก้บั๊กเล็กน้อย"
# ❌ ถูกปฏิเสธ เพราะไม่มี type นำหน้า

git commit -m "fix(login): แก้บั๊กการ redirect หลัง login"
# ✅ ผ่าน เพราะตรงรูปแบบ
```

### จุดที่ทรงพลังกว่านั้น: `commit-msg` แก้ไขข้อความได้ด้วย ไม่ใช่แค่ปฏิเสธ

เพราะ hook ได้รับ **path ของไฟล์** ไม่ใช่แค่ข้อความ มันจึงสามารถ **เขียนทับไฟล์นั้นได้โดยตรง** เพื่อ "แก้ไข" commit message ให้อัตโนมัติก่อนที่ Git จะเอาไปสร้าง commit จริง

ตัวอย่าง: เพิ่มเลข issue number ต่อท้ายข้อความอัตโนมัติ ถ้าชื่อ branch มีรูปแบบ `feature/123-some-name`:

```bash
#!/bin/bash
# .git/hooks/commit-msg

commit_msg_file="$1"
branch_name=$(git symbolic-ref --short HEAD)

# ดึงเลข issue จากชื่อ branch เช่น feature/123-fix-login -> 123
issue_number=$(echo "$branch_name" | grep -oE '^[a-zA-Z]+/([0-9]+)' | grep -oE '[0-9]+')

if [ -n "$issue_number" ]; then
    # เช็คก่อนว่ามีเลข issue นี้อยู่ในข้อความแล้วหรือยัง จะได้ไม่เพิ่มซ้ำ
    if ! grep -q "#$issue_number" "$commit_msg_file"; then
        echo "" >> "$commit_msg_file"
        echo "Refs #$issue_number" >> "$commit_msg_file"
    fi
fi

exit 0
```

นี่คือความแตกต่างสำคัญระหว่าง `pre-commit` กับ `commit-msg`:

| | `pre-commit` | `commit-msg` |
|---|---|---|
| ทำงานเมื่อไหร่ | ก่อนพิมพ์ commit message | หลังพิมพ์ commit message เสร็จแล้ว |
| รับ argument | ไม่รับ | รับ path ของไฟล์ commit message |
| ตรวจสอบอะไรได้ | เนื้อหาไฟล์ที่ staged | ข้อความ commit message |
| แก้ไขอะไรได้ | ไฟล์ที่จะ commit (ก่อน stage ใหม่) | แก้ไขข้อความ commit message ได้โดยตรง |

---

## Step 584: `prepare-commit-msg` hook — เติม template ลง commit message อัตโนมัติก่อนเปิด editor

### `prepare-commit-msg` ทำงานเมื่อไหร่

Hook นี้ทำงาน **ก่อน** editor ถูกเปิดขึ้นมาให้ผู้ใช้พิมพ์ commit message (แต่หลัง `pre-commit`) จุดประสงค์หลักคือ **เตรียมข้อความเริ่มต้น (default template)** ไว้ล่วงหน้าในไฟล์ commit message ก่อนที่ editor จะเปิดขึ้นมา ทำให้ผู้ใช้เห็นข้อความ pre-fill ไว้แล้วเมื่อ editor เปิด

### Argument ที่ `prepare-commit-msg` ได้รับ (สำคัญมาก มี 3 ตัว)

```bash
#!/bin/bash
# .git/hooks/prepare-commit-msg
# $1 = path ของไฟล์ commit message
# $2 = source ของ commit message (message, template, merge, squash, commit) — อาจว่างเปล่า
# $3 = SHA ของ commit (มีเฉพาะกรณี --amend หรือ commit จาก commit เดิม)

commit_msg_file="$1"
commit_source="$2"
commit_sha="$3"
```

ค่า `$2` (commit_source) สำคัญมาก เพราะบอกว่า commit นี้เกิดจากอะไร:

| ค่า `$2` | เกิดขึ้นเมื่อ |
|---|---|
| (ว่างเปล่า) | ผู้ใช้ commit ปกติแบบเปิด editor ให้พิมพ์เอง |
| `message` | ใช้ `git commit -m "..."` |
| `template` | ใช้ `git commit --template <file>` |
| `merge` | กำลังทำ merge commit |
| `squash` | กำลังทำ squash commit |
| `commit` | ใช้ `git commit -c <commit>` หรือ `--amend` |

### ทำไมต้องเช็ค `$2` ก่อนเติม template

**สำคัญมาก:** ถ้าไม่เช็ค `$2` ก่อน hook อาจไปเติมข้อความทับ commit message ที่ผู้ใช้พิมพ์มาแล้วด้วย `-m` หรือไปยุ่งกับ merge commit message ที่ Git สร้างไว้ให้อัตโนมัติ ซึ่งจะทำให้เกิดพฤติกรรมแปลก ๆ ที่ผู้ใช้ไม่คาดคิด

### ตัวอย่าง: เติม template พร้อมชื่อ branch และ checklist อัตโนมัติ

```bash
#!/bin/bash
# .git/hooks/prepare-commit-msg

commit_msg_file="$1"
commit_source="$2"

# เติม template เฉพาะตอน commit ปกติเท่านั้น (ไม่ใช่ merge/squash/amend/-m)
if [ -z "$commit_source" ]; then
    branch_name=$(git symbolic-ref --short HEAD 2>/dev/null)
    issue_number=$(echo "$branch_name" | grep -oE '[0-9]+' | head -1)

    template="

# Branch: $branch_name"

    if [ -n "$issue_number" ]; then
        template="$template
# เกี่ยวข้องกับ Issue #$issue_number (จะถูกลบอัตโนมัติถ้าไม่แก้ไข)"
    fi

    # เติมต่อท้ายไฟล์ commit message ที่มีอยู่ (มักว่างเปล่าอยู่แล้วตอนนี้)
    echo "$template" >> "$commit_msg_file"
fi

exit 0
```

เมื่อผู้ใช้สั่ง `git commit` (โดยไม่ใส่ `-m`) editor จะเปิดขึ้นมาพร้อมข้อความ:

```

# Branch: feature/123-fix-login
# เกี่ยวข้องกับ Issue #123 (จะถูกลบอัตโนมัติถ้าไม่แก้ไข)
```

รอให้ผู้ใช้พิมพ์ข้อความ commit จริงไว้ด้านบน บรรทัดที่ขึ้นต้นด้วย `#` จะถูก Git ตัดทิ้งอัตโนมัติเมื่อบันทึก (เพราะ Git ถือว่าเป็น comment) เว้นแต่จะตั้งค่า `core.commentChar` เป็นอย่างอื่น

### ตัวอย่าง: เติม Pull Request template แบบเต็ม

บริษัทหลายแห่งใช้ hook นี้เพื่อบังคับให้ commit message มีโครงสร้างมาตรฐาน:

```bash
#!/bin/bash
# .git/hooks/prepare-commit-msg

commit_msg_file="$1"
commit_source="$2"

if [ -z "$commit_source" ]; then
    template=$(cat <<'EOF'


# ทำไมต้องเปลี่ยน (Why):
# 

# เปลี่ยนอะไรบ้าง (What):
# 

# ทดสอบอย่างไร (How tested):
# 
EOF
)
    # ต่อ template เข้าไปในไฟล์ commit message (ซึ่งตอนนี้ยังว่างอยู่)
    cat "$commit_msg_file" > /tmp/original_msg.$$
    echo "$template" >> "$commit_msg_file"
    cat /tmp/original_msg.$$ >> "$commit_msg_file"
    rm -f /tmp/original_msg.$$
fi

exit 0
```

---

## Step 585: `post-commit` hook — ทำงานหลัง commit สำเร็จ

### `post-commit` ทำงานเมื่อไหร่

Hook นี้ทำงาน **หลังจาก** commit object ถูกสร้างขึ้นเรียบร้อยแล้ว 100% จุดสำคัญคือ:

> **`post-commit` ไม่สามารถหยุดหรือยกเลิก commit ได้อีกต่อไป ไม่ว่าจะ exit code เท่าไหร่ก็ตาม** เพราะ commit เสร็จสมบูรณ์ไปแล้วก่อนที่ hook นี้จะเริ่มทำงาน

ดังนั้น `post-commit` จึงเหมาะกับงานที่เป็น **การแจ้งเตือน (notification)** หรือ **งานเสริมที่ไม่กระทบกับตัว commit เอง** เท่านั้น เช่น:

- ส่งข้อความแจ้งเตือนเข้า Slack/Discord ว่ามีคน commit อะไรไปบ้าง
- อัปเดตไฟล์ log ภายในของทีม
- Trigger การ build เอกสารอัตโนมัติ (เช่น regenerate changelog)
- แสดงข้อความเตือนใจ เช่น "อย่าลืม push!"

### ตัวอย่าง: `post-commit` แสดงสรุปสั้น ๆ หลัง commit

```bash
#!/bin/bash
# .git/hooks/post-commit

commit_hash=$(git rev-parse --short HEAD)
commit_msg=$(git log -1 --pretty=%s)
files_changed=$(git diff-tree --no-commit-id --name-only -r HEAD | wc -l)

echo "✅ Commit สำเร็จ: [$commit_hash] $commit_msg"
echo "   ไฟล์ที่เปลี่ยนแปลง: $files_changed ไฟล์"
echo "   อย่าลืม push งานของคุณเมื่อพร้อม: git push"
```

### ตัวอย่าง: `post-commit` ส่งแจ้งเตือนเข้า Slack ผ่าน webhook

```bash
#!/bin/bash
# .git/hooks/post-commit

SLACK_WEBHOOK_URL="https://hooks.slack.com/services/XXX/YYY/ZZZ"

commit_hash=$(git rev-parse --short HEAD)
commit_msg=$(git log -1 --pretty=%s)
author=$(git log -1 --pretty=%an)
branch=$(git symbolic-ref --short HEAD)

payload=$(cat <<EOF
{
  "text": "📦 *$author* commit ใหม่บน branch \`$branch\`:\n\`$commit_hash\` $commit_msg"
}
EOF
)

curl -s -X POST -H 'Content-type: application/json' \
    --data "$payload" \
    "$SLACK_WEBHOOK_URL" > /dev/null &

exit 0
```

สังเกตว่าใช้ `&` ต่อท้าย `curl` เพื่อรันเป็น background process — เพราะเราไม่อยากให้ผู้ใช้ต้องรอ network request เสร็จก่อนจะได้ใช้งาน terminal ต่อ (การรอ network ใน hook ที่ทำงาน synchronous เป็นเรื่องที่ควรหลีกเลี่ยงถ้าเป็นไปได้)

### ข้อจำกัดสำคัญที่ต้องรู้: `post-commit` ไม่ได้รัน "ทุกครั้ง" ที่มีการเปลี่ยน HEAD

`post-commit` ทำงานเฉพาะตอนที่คุณสั่ง `git commit` เท่านั้น ไม่ทำงานตอน `git merge`, `git rebase`, `git cherry-pick` (ยกเว้นในบางกรณีที่การกระทำเหล่านั้นสร้าง commit ใหม่จริง ๆ ผ่านกระบวนการเดียวกับ commit ปกติ เช่น merge แบบ non-fast-forward ก็จะ trigger `post-commit` เพราะมันสร้าง merge commit จริง)

---

## Step 586: `pre-push` hook — ตรวจสอบก่อน push (รัน test ทั้งหมดก่อนอนุญาตให้ push)

### `pre-push` ทำงานเมื่อไหร่

Hook นี้ทำงานเมื่อคุณสั่ง `git push` **ก่อน**ที่ข้อมูลใด ๆ จะถูกส่งไปยัง remote จริง ๆ ถ้า hook นี้ exit ด้วยค่าที่ไม่ใช่ `0` การ push ทั้งหมดจะถูก **ยกเลิกทันที** โดยไม่มีข้อมูลใดถูกส่งออกไปเลย

นี่คือจุดที่เหมาะที่สุดสำหรับงานที่ **หนักกว่า** `pre-commit` เพราะ push เกิดขึ้นน้อยครั้งกว่า commit มาก (คุณอาจ commit 10 ครั้งแล้วค่อย push ทีเดียว) จึงเหมาะกับ:

- รัน **test suite แบบเต็ม** ทั้งหมดก่อนอนุญาตให้ push
- ตรวจสอบว่าไม่มีการ push branch ที่ผิดกฎ (เช่น ห้าม push ตรงเข้า `main`)
- ตรวจสอบว่า build ผ่านก่อน push
- สแกนหา secret ที่หลุดเข้ามาในโค้ดก่อนที่จะไปโผล่บน remote (ซึ่งลบยากกว่ามาก)

### Input พิเศษของ `pre-push`: อ่านจาก stdin ไม่ใช่ argument

จุดที่แตกต่างจาก hook อื่นชัดเจนคือ `pre-push` รับข้อมูลผ่าน **stdin** ไม่ใช่ argument โดยแต่ละบรรทัดจะมีรูปแบบ:

```
<local ref> <local sha1> <remote ref> <remote sha1>
```

ส่วน argument (`$1`, `$2`) ที่ได้รับคือชื่อ remote และ URL ของ remote:

```bash
#!/bin/bash
# .git/hooks/pre-push
# $1 = ชื่อ remote (เช่น "origin")
# $2 = URL ของ remote

remote_name="$1"
remote_url="$2"

while read local_ref local_sha remote_ref remote_sha; do
    echo "กำลัง push $local_ref ($local_sha) ไปยัง $remote_ref บน $remote_name"
done
```

`local_sha` ที่เป็นเลข `0000000000000000000000000000000000000000` (ศูนย์ล้วน) หมายความว่านี่คือการ**ลบ branch** ที่ remote (ไม่ใช่ push โค้ดจริง) ควรเช็คกรณีนี้ไว้เสมอ เพื่อไม่ให้ hook ไปรัน test โดยไม่จำเป็นตอนที่ผู้ใช้แค่ลบ branch

### ตัวอย่าง: `pre-push` รัน test suite เต็มก่อนอนุญาตให้ push

```bash
#!/bin/bash
# .git/hooks/pre-push

zero_sha="0000000000000000000000000000000000000000"

while read local_ref local_sha remote_ref remote_sha; do
    # ถ้าเป็นการลบ branch ที่ remote ให้ข้ามไปเลย ไม่ต้องรัน test
    if [ "$local_sha" = "$zero_sha" ]; then
        continue
    fi

    echo "🧪 กำลังรัน test suite ทั้งหมดก่อนอนุญาตให้ push..."

    npm test

    if [ $? -ne 0 ]; then
        echo "❌ Test ไม่ผ่าน ยกเลิกการ push"
        echo "   กรุณาแก้ไข test ให้ผ่านก่อน แล้วลอง push ใหม่อีกครั้ง"
        exit 1
    fi
done

echo "✅ Test ผ่านหมด อนุญาตให้ push"
exit 0
```

### ตัวอย่าง: `pre-push` ห้าม push ตรงเข้า `main`/`master`

```bash
#!/bin/bash
# .git/hooks/pre-push

protected_branch="main"
zero_sha="0000000000000000000000000000000000000000"

while read local_ref local_sha remote_ref remote_sha; do
    if [ "$local_sha" = "$zero_sha" ]; then
        continue
    fi

    current_branch=$(git symbolic-ref --short HEAD)

    if [ "$current_branch" = "$protected_branch" ]; then
        echo "❌ ห้าม push เข้า branch '$protected_branch' โดยตรง"
        echo "   กรุณาสร้าง Pull Request แทน"
        exit 1
    fi
done

exit 0
```

### เปรียบเทียบ `pre-commit` กับ `pre-push` ว่าควรใส่อะไรตรงไหน

| งาน | ควรอยู่ที่ | เหตุผล |
|---|---|---|
| Lint syntax เร็ว ๆ | `pre-commit` | commit บ่อย ต้องเร็ว |
| Format โค้ดอัตโนมัติ | `pre-commit` | ควรจัดรูปแบบทุกครั้งก่อนบันทึกประวัติ |
| Unit test แบบเร็ว (เฉพาะไฟล์ที่เปลี่ยน) | `pre-commit` (ถ้าเร็วพอ) | ยังพอรับได้ถ้าไม่เกิน 2-3 วินาที |
| Integration test / Full test suite | `pre-push` | ใช้เวลานาน แต่ push ไม่ได้เกิดบ่อยเท่า commit |
| ตรวจ commit message format | `commit-msg` | ต้องมีข้อความเต็มให้ตรวจก่อน |
| ห้าม push เข้า branch คุ้มครอง | `pre-push` | เป็นเรื่องของการ push ไม่ใช่ commit |

---

## Step 587: การเขียน hook script ด้วย bash จริงที่ใช้งานได้ (shebang, exit code)

### โครงสร้างพื้นฐานที่ทุก hook ต้องมี

```bash
#!/bin/bash
# บรรทัดแรกสุด (shebang) — บอก OS ว่าให้รันไฟล์นี้ด้วย bash

# ... เนื้อหาการตรวจสอบ ...

exit 0   # หรือ exit 1 (หรือเลขอื่นที่ไม่ใช่ 0) แล้วแต่ผลการตรวจสอบ
```

### 1. Shebang (`#!`) คืออะไร ทำไมสำคัญ

บรรทัดแรกของทุก hook script ต้องเป็น **shebang line** ที่บอกระบบปฏิบัติการว่าให้ใช้ **interpreter ตัวไหน** รันไฟล์นี้:

```bash
#!/bin/bash          # ใช้ bash
#!/usr/bin/env bash  # ใช้ bash ตัวแรกที่เจอใน PATH (ยืดหยุ่นกว่าข้ามระบบ)
#!/usr/bin/env python3  # เขียน hook ด้วย Python แทน
#!/usr/bin/env node     # เขียน hook ด้วย Node.js แทน
#!/usr/bin/env ruby     # เขียน hook ด้วย Ruby แทน
```

ถ้าไม่มี shebang หรือ shebang ผิด ระบบจะไม่รู้ว่าต้องเอาไฟล์นี้ไปรันด้วยอะไร และ hook จะไม่ทำงานหรือ error ทันที

**ข้อแนะนำ:** ใช้ `#!/usr/bin/env bash` แทน `#!/bin/bash` ตรง ๆ เพราะบางระบบ (เช่น NixOS หรือบาง container) bash ไม่ได้อยู่ที่ `/bin/bash` เสมอไป การใช้ `env` จะไปค้นหาใน `PATH` ให้แทน ทำให้ script ทำงานข้ามระบบได้กว้างกว่า

### 2. Executable permission

หลังเขียนไฟล์เสร็จ ต้องให้สิทธิ์ execute เสมอ ไม่งั้น Git จะข้าม hook นี้ไปเฉย ๆ โดยไม่แจ้ง error ใด ๆ:

```bash
chmod +x .git/hooks/pre-commit
```

ตรวจสอบว่า executable แล้วหรือยัง:

```bash
ls -l .git/hooks/pre-commit
# -rwxr-xr-x  1 user  staff  245 Jan 1 10:00 .git/hooks/pre-commit
#  ^^^ ตัว x ตรงนี้คือสิ่งที่บอกว่า executable แล้ว
```

### 3. Exit code คือภาษาเดียวที่ Git ฟัง

Git ไม่ได้สนใจว่า hook พิมพ์อะไรออกมาทาง stdout/stderr (สิ่งเหล่านั้นแค่แสดงให้ผู้ใช้เห็นเฉย ๆ) สิ่งเดียวที่ Git ใช้ตัดสินใจคือ **exit code** ของ process นั้น:

| Exit code | ความหมาย | ผลต่อ Git |
|---|---|---|
| `0` | สำเร็จ (success) | Git ทำงานต่อไปตามปกติ (commit/push สำเร็จ) |
| ไม่ใช่ `0` (เช่น `1`, `2`, ...) | ล้มเหลว (failure) | Git **หยุด** การกระทำนั้นทันที (สำหรับ hook ประเภทตรวจสอบก่อน เช่น `pre-commit`, `commit-msg`, `pre-push`) |

ตัวอย่างการเช็ค exit code ของคำสั่งก่อนหน้าใน bash:

```bash
npm run lint

if [ $? -ne 0 ]; then
    # $? คือ exit code ของคำสั่งล่าสุดที่รันไป
    echo "Lint ไม่ผ่าน"
    exit 1
fi
```

หรือเขียนแบบสั้นกว่าด้วย `&&`/`||`:

```bash
npm run lint || { echo "Lint ไม่ผ่าน"; exit 1; }
```

หรือให้ exit code ของ hook เท่ากับ exit code ของคำสั่งย่อยไปเลย (ถ้ามีคำสั่งเดียว):

```bash
#!/bin/bash
exec npm run lint
# exec จะแทนที่ process ปัจจุบันด้วย npm run lint และส่ง exit code ของมันออกไปโดยตรง
```

### 4. ควรใช้ `set -e` เพื่อความปลอดภัยเมื่อ script มีหลายคำสั่ง

```bash
#!/bin/bash
set -e   # ถ้าคำสั่งไหน exit ด้วยค่าไม่ใช่ 0 ให้หยุด script ทั้งหมดทันที (และ hook จะ exit ด้วยค่านั้นไปด้วย)

npm run lint
npm run format:check
npm run test:unit
```

`set -e` ทำให้ไม่ต้องเช็ค `$?` เองทีละบรรทัด แต่ต้องระวัง: ถ้าอยากให้ script ทำงานต่อแม้บางคำสั่งจะ fail (เช่นอยากรวบรวม error ทั้งหมดก่อนสรุป) ก็ไม่ควรใช้ `set -e`

### 5. Debug hook ที่ไม่ทำงานตามคาด

ถ้า hook ไม่ทำงานเลย ให้ไล่เช็คตามลำดับนี้:

```bash
# 1. เช็คว่าไฟล์ชื่อถูกต้องเป๊ะ ไม่มีนามสกุลต่อท้าย
ls -la .git/hooks/pre-commit

# 2. เช็คว่า executable หรือยัง
ls -l .git/hooks/pre-commit | grep 'x'

# 3. ทดสอบรัน hook ตรง ๆ ด้วยมือ ดูว่า error อะไรไหม
.git/hooks/pre-commit
echo "Exit code: $?"

# 4. เช็ค shebang ว่าถูกต้องและ interpreter มีอยู่จริงในเครื่อง
head -1 .git/hooks/pre-commit
which bash
```

### 6. เขียน hook ด้วยภาษาอื่นที่ไม่ใช่ bash ก็ได้

Git ไม่สนใจว่า hook เขียนด้วยภาษาอะไร ขอแค่ไฟล์ executable และมี shebang ที่ถูกต้อง ตัวอย่างเขียนด้วย Python:

```python
#!/usr/bin/env python3
# .git/hooks/pre-commit

import subprocess
import sys

result = subprocess.run(
    ["git", "diff", "--cached", "--name-only"],
    capture_output=True, text=True
)
staged_files = result.stdout.splitlines()

for f in staged_files:
    if f.endswith(".env"):
        print(f"❌ ห้าม commit ไฟล์ .env: {f}")
        sys.exit(1)

sys.exit(0)
```

```bash
chmod +x .git/hooks/pre-commit
```

---

## Step 588: ปัญหาสำคัญ — hook ไม่ถูก commit/แชร์ผ่าน git โดย default และวิธีแก้

### ปัญหา: `.git/hooks/` ไม่ใช่ tracked file

นี่คือข้อจำกัดที่สำคัญที่สุดของ Git Hooks แบบดั้งเดิม:

> **โฟลเดอร์ `.git/` ทั้งโฟลเดอร์ (รวมถึง `.git/hooks/`) ไม่เคยถูก commit หรือ push ไปพร้อมกับโค้ดเลย** เพราะมันคือ metadata ภายในของ Git repository เอง ไม่ใช่ไฟล์โปรเจกต์

ผลที่ตามมาคือ:

- คุณเขียน `pre-commit` hook ไว้ในเครื่องตัวเอง แล้ว push โค้ดขึ้น GitHub
- เพื่อนร่วมทีม `git clone` repo เดียวกันไป
- เพื่อนร่วมทีม **จะไม่ได้ hook นั้นติดไปด้วยเลย** เพราะ `.git/hooks/` ของเขาเป็นโฟลเดอร์เปล่า ๆ ที่ Git สร้างขึ้นใหม่ตอน clone (มีแต่ไฟล์ `.sample`)

นี่คือปัญหาใหญ่มากสำหรับทีม เพราะ hook ที่ควรบังคับใช้มาตรฐานร่วมกัน (เช่น ห้าม commit `.env`, บังคับ Conventional Commits) กลับใช้งานได้แค่คนที่ตั้งค่าเองเท่านั้น

### ทางแก้ที่ 1: `core.hooksPath` — วิธีที่ Git เองรองรับโดยตรง (ตั้งแต่ Git 2.9)

Git อนุญาตให้เปลี่ยน "ที่อยู่" ของ hooks จาก `.git/hooks/` ไปเป็นโฟลเดอร์อื่นที่อยู่**ภายในโปรเจกต์** (จึงถูก track และแชร์ผ่าน Git ได้ตามปกติ) ผ่าน config `core.hooksPath`

ขั้นตอน:

```bash
# 1. สร้างโฟลเดอร์เก็บ hooks ไว้ในโปรเจกต์ (จะถูก commit ไปพร้อมโค้ด)
mkdir -p .githooks

# 2. เขียน hook ไว้ในนี้แทน
cat > .githooks/pre-commit << 'EOF'
#!/bin/bash
staged_files=$(git diff --cached --name-only)
for file in $staged_files; do
    if [[ "$file" == *.env ]]; then
        echo "❌ ห้าม commit ไฟล์ .env: $file"
        exit 1
    fi
done
exit 0
EOF

chmod +x .githooks/pre-commit

# 3. บอก Git ให้ใช้โฟลเดอร์นี้แทน .git/hooks/
git config core.hooksPath .githooks

# 4. commit โฟลเดอร์ .githooks/ เข้า repo ตามปกติ
git add .githooks
git commit -m "chore: เพิ่ม shared git hooks ผ่าน core.hooksPath"
```

**ข้อจำกัดสำคัญ:** `git config core.hooksPath .githooks` เป็นคำสั่งที่ **ทุกคนในทีมต้องรันเองครั้งหนึ่ง** หลัง clone repo เพราะ `core.hooksPath` เป็นค่า config ระดับ repository ในเครื่องนั้น ๆ ไม่ได้ถูกบังคับใช้อัตโนมัติจากการ clone อย่างเดียว วิธีแก้คือใส่คำแนะนำนี้ไว้ใน `README.md` หรือทำ setup script อัตโนมัติให้รันตอน `npm install` (ผ่าน `postinstall` script)

### ทางแก้ที่ 2: Husky (นิยมที่สุดในโลก Node.js/JavaScript)

**Husky** คือเครื่องมือที่ทำให้การติดตั้ง Git hooks ที่แชร์กันในทีมเป็นเรื่องอัตโนมัติ โดยอาศัยกลไก `core.hooksPath` เบื้องหลัง แต่ห่อด้วย npm package ที่ทำงานให้อัตโนมัติตอน `npm install`

```bash
# ติดตั้ง husky
npm install --save-dev husky

# เปิดใช้งาน husky (จะสร้างโฟลเดอร์ .husky/ และตั้ง core.hooksPath ให้อัตโนมัติ)
npx husky init
```

หลังรันคำสั่งนี้ husky จะ:

1. สร้างโฟลเดอร์ `.husky/` ในโปรเจกต์ (ถูก track โดย Git ตามปกติ)
2. ตั้งค่า `core.hooksPath` ให้ชี้ไปที่ `.husky/_` โดยอัตโนมัติ
3. เพิ่ม script `prepare` ใน `package.json` ที่จะรัน `husky` ให้ตั้งค่า hooksPath ใหม่ทุกครั้งที่มีคน `npm install` โปรเจกต์นี้

```json
{
  "scripts": {
    "prepare": "husky"
  }
}
```

จากนี้ทุกครั้งที่เพื่อนร่วมทีม clone repo แล้วรัน `npm install` — Husky จะถูกตั้งค่าให้อัตโนมัติทันทีโดยไม่ต้องมีใครจำคำสั่งพิเศษเลย นี่คือจุดแข็งที่สำคัญที่สุดของ Husky เมื่อเทียบกับการตั้ง `core.hooksPath` มือเปล่า

เพิ่ม hook ด้วย husky:

```bash
echo "npx lint-staged" > .husky/pre-commit
chmod +x .husky/pre-commit

git add .husky
git commit -m "chore: ตั้งค่า husky pre-commit hook"
```

Husky มักใช้คู่กับ **lint-staged** เพื่อรัน linter เฉพาะไฟล์ที่ staged เท่านั้น (ไม่ใช่ทั้งโปรเจกต์):

```bash
npm install --save-dev lint-staged
```

```json
// package.json
{
  "lint-staged": {
    "*.{js,ts}": ["eslint --fix", "prettier --write"]
  }
}
```

### ทางแก้ที่ 3: `pre-commit` framework (นิยมที่สุดในโลก Python)

**pre-commit** เป็น framework ที่เขียนด้วย Python สำหรับจัดการ hooks แบบข้ามภาษา นิยมมากในโปรเจกต์ Python แต่ใช้ได้กับภาษาอื่นด้วย

```bash
# ติดตั้ง (ต้องมี Python/pip)
pip install pre-commit
```

สร้างไฟล์ config `.pre-commit-config.yaml` ที่ root ของโปรเจกต์:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-added-large-files
      - id: check-merge-conflict
  - repo: https://github.com/psf/black
    rev: 24.1.0
    hooks:
      - id: black
```

ติดตั้ง hook เข้า repository จริง (คำสั่งนี้แหละที่เขียนไฟล์ลง `.git/hooks/pre-commit` ให้อัตโนมัติ):

```bash
pre-commit install
```

**ข้อสังเกต:** `pre-commit` framework ยังคงใช้กลไก `.git/hooks/` แบบดั้งเดิมอยู่ดี (ไม่ได้ใช้ `core.hooksPath`) เพียงแต่มันสร้างไฟล์ `.pre-commit-config.yaml` ที่ถูก track โดย Git และมี "ตัวเรียก" (bootstrap script) เพียงตัวเดียวที่เขียนลง `.git/hooks/pre-commit` ทำหน้าที่อ่าน config นั้นแล้วรัน hook ที่ระบุไว้ ทุกคนในทีมยังต้องรัน `pre-commit install` เองครั้งหนึ่งหลัง clone อยู่ดี (เหมือนกับ `core.hooksPath` มือเปล่า) เพียงแต่ pre-commit framework มีระบบจัดการเวอร์ชันของ hook ที่ยืดหยุ่นกว่ามาก (ดึง hook จาก repo ภายนอกมาใช้ได้ ระบุ version ได้ชัดเจน)

### สรุปเปรียบเทียบ 3 วิธี

| วิธี | ภาษาที่เหมาะ | ต้องรันคำสั่งเองหลัง clone ไหม | จุดเด่น |
|---|---|---|---|
| `core.hooksPath` มือเปล่า | ทุกภาษา | ต้อง (`git config core.hooksPath .githooks`) | ง่ายที่สุด ไม่ต้องพึ่ง dependency ภายนอก |
| **Husky** | Node.js/JavaScript เป็นหลัก | ไม่ต้อง (อัตโนมัติผ่าน `npm install` + `prepare` script) | ติดตั้งอัตโนมัติที่สุด นิยมสูงสุดในวงการ frontend |
| **pre-commit framework** | Python เป็นหลัก (แต่ใช้ข้ามภาษาได้) | ต้อง (`pre-commit install`) | จัดการ hook จาก repo ภายนอกได้ดีเยี่ยม ระบุ version ชัดเจน |

ไม่ว่าจะเลือกวิธีไหน หลักการที่สำคัญที่สุดคือ: **เขียน hook เป็นไฟล์ที่อยู่ในโปรเจกต์ (tracked by Git) แล้วมีกลไก "ติดตั้ง" มันเข้า `.git/hooks/` หรือ `core.hooksPath` แทนที่จะพึ่งให้แต่ละคนสร้าง hook เองด้วยมือ**

---

## Step 589: การข้าม hook ชั่วคราวด้วย `--no-verify`

### `--no-verify` คืออะไร

Git มีทางออกฉุกเฉินสำหรับกรณีที่คุณต้องการ **ข้าม hook ที่เกี่ยวกับการตรวจสอบ** ไปชั่วคราว โดยใช้ flag `--no-verify`

```bash
# ข้าม pre-commit และ commit-msg hook
git commit --no-verify -m "fix: แก้ไขด่วน"

# ข้าม pre-push hook
git push --no-verify
```

**หมายเหตุสำคัญ:** `--no-verify` ข้ามได้เฉพาะ hook ที่เป็นประเภท "verification" เท่านั้น คือ `pre-commit`, `commit-msg` (สำหรับ `git commit`) และ `pre-push` (สำหรับ `git push`) — มันไม่ได้ข้าม hook ทุกตัว เช่น `post-commit` จะยังทำงานตามปกติเสมอ ไม่ว่าจะใส่ `--no-verify` หรือไม่ก็ตาม เพราะมันไม่ใช่ hook ที่ตรวจสอบก่อนการกระทำ

### เมื่อไหร่ที่ใช้ `--no-verify` อย่างเหมาะสม

1. **สถานการณ์ฉุกเฉินจริง ๆ (hotfix วิกฤต)** — ระบบ production ล่ม ต้อง push แก้ด่วนที่สุด และ hook ที่ตั้งไว้ (เช่น full test suite ที่ใช้เวลา 20 นาที) จะทำให้ล่าช้าเกินไป
2. **Hook เสียเอง (false positive)** — เช่น hook เขียนผิดพลาด หรือ dependency ที่ hook ต้องใช้ยังไม่ได้ติดตั้งในเครื่อง ทำให้ hook fail ทั้ง ๆ ที่โค้ดไม่มีปัญหาจริง
3. **Commit ชั่วคราวที่รู้อยู่แล้วว่าไม่สมบูรณ์** — เช่น commit เพื่อสลับเครื่อง (work-in-progress) ที่จะถูก squash/แก้ไขทีหลังแน่นอน ก่อน push จริง
4. **กำลัง debug ปัญหาของ hook เอง** — ต้องการ commit เพื่อทดสอบว่า hook มีปัญหาตรงไหน

### ทำไมไม่ควรใช้ `--no-verify` พร่ำเพรื่อ

1. **ทำลายจุดประสงค์ทั้งหมดของการมี hook** — ถ้าทุกคนกด `--no-verify` เป็นนิสัยเมื่อ hook ใช้เวลานานหรือน่ารำคาญ มาตรฐานที่ทีมตกลงกันไว้ (lint ผ่าน, test ผ่าน, commit message ถูกรูปแบบ) จะไม่มีความหมายอะไรเลย
2. **ปัญหาจะไปโผล่ที่ CI/CD แทน** — ถ้าข้าม `pre-push` ที่รัน test ไปเรื่อย ๆ โค้ดที่ test ไม่ผ่านจะไปพังตอน CI pipeline บน server แทน ซึ่งกว่าจะรู้ก็ช้ากว่า และกระทบทีมอื่นที่ pull โค้ดไปแล้ว
3. **Secret หลุดเข้า repository ได้ง่ายขึ้น** — ถ้า hook มีไว้เช็คห้าม commit `.env` แล้วมีคนใช้ `--no-verify` เพราะรีบ ไฟล์ credential อาจหลุดเข้า Git history ซึ่งลบออกยากมากในภายหลัง (ต้องใช้ `git filter-repo` หรือเทียบเท่า และต้อง force-push ทับประวัติทั้งหมด)
4. **สร้างวัฒนธรรมที่ไม่ดีในทีม** — ถ้าเห็นคนอื่นใช้ `--no-verify` บ่อย ๆ โดยไม่มีเหตุผลชัดเจน คนอื่นในทีมก็มักจะทำตาม จนสุดท้าย hook ที่ตั้งไว้แทบไม่มีใครสนใจอีกต่อไป

### แนวทางปฏิบัติที่ดี

- ใช้ `--no-verify` ได้ แต่ **ควรมีเหตุผลชัดเจนและบอกทีมด้วยเสมอ** เช่น commit message หรือข้อความใน PR อธิบายว่าทำไมถึงข้าม
- ถ้า hook ใช้เวลานานเกินไปจนคนอยากข้ามบ่อย ๆ **ควรแก้ที่ตัว hook ให้เร็วขึ้น** (เช่น ย้ายงานหนักจาก `pre-commit` ไป `pre-push` หรือ CI) แทนที่จะปล่อยให้ทุกคนข้ามมันไปเรื่อย ๆ
- พิจารณาใช้ CI/CD เป็น **ด่านสุดท้ายที่ข้ามไม่ได้** เสมอ (server-side enforcement) เพื่อชดเชยกรณีที่ hook ฝั่ง client ถูกข้ามไป — นี่คือเหตุผลที่ทีมมืออาชีพมักไม่พึ่ง client-side hook เพียงอย่างเดียวสำหรับกฎที่สำคัญจริง ๆ (เพราะ hook ฝั่ง client แก้ไข/ปิดใช้งานได้โดยตัว developer เอง) แต่จะมี server-side hook หรือ CI check คู่กันเสมอ ซึ่งเราจะเรียนเรื่องนี้ต่อใน **Part 60**

---

## Step 590: แบบฝึกหัด — เขียน pre-commit และ commit-msg hook จริงให้โปรเจกต์ทดลอง

มาลงมือสร้าง hook ทั้งสองตัวแบบใช้งานได้จริง ในโปรเจกต์ทดลองของคุณ

### 590.1 เตรียมโปรเจกต์ทดลอง

```bash
mkdir -p ~/git-course/part-59-hooks
cd ~/git-course/part-59-hooks
git init

echo "# โปรเจกต์ทดลอง Git Hooks" > README.md
git add README.md
git commit -m "chore: initial commit"
```

### 590.2 เขียน `pre-commit` hook — ห้าม commit ไฟล์ `.env`

```bash
cat > .git/hooks/pre-commit << 'EOF'
#!/usr/bin/env bash
#
# pre-commit hook: ห้าม commit ไฟล์ .env หรือไฟล์ที่ลงท้ายด้วย .env
# เพื่อป้องกันไม่ให้ secret/credential หลุดเข้า Git history

staged_files=$(git diff --cached --name-only --diff-filter=ACM)

blocked=0

for file in $staged_files; do
    base_name=$(basename "$file")
    if [[ "$base_name" == ".env" ]] || [[ "$base_name" == *.env ]] || [[ "$base_name" == .env.* ]]; then
        echo "❌ ห้าม commit ไฟล์ที่ดูเหมือนไฟล์ env: $file"
        blocked=1
    fi
done

if [ "$blocked" -eq 1 ]; then
    echo ""
    echo "หากไฟล์เหล่านี้ถูก stage โดยไม่ตั้งใจ ให้ unstage ด้วยคำสั่ง:"
    echo "   git restore --staged <ชื่อไฟล์>"
    echo ""
    echo "หากไฟล์นี้ควรถูก commit จริง ๆ (ซึ่งไม่ควรทำ) ให้ข้าม hook นี้ด้วย:"
    echo "   git commit --no-verify"
    exit 1
fi

exit 0
EOF

chmod +x .git/hooks/pre-commit
```

**ทดสอบว่า hook ทำงานจริง:**

```bash
# ทดสอบกรณีที่ควรถูกบล็อก
echo "SECRET_KEY=abc123" > .env
git add .env
git commit -m "feat: เพิ่มไฟล์ config"
```

ผลลัพธ์ที่คาดหวัง:

```
❌ ห้าม commit ไฟล์ที่ดูเหมือนไฟล์ env: .env

หากไฟล์เหล่านี้ถูก stage โดยไม่ตั้งใจ ให้ unstage ด้วยคำสั่ง:
   git restore --staged <ชื่อไฟล์>

หากไฟล์นี้ควรถูก commit จริง ๆ (ซึ่งไม่ควรทำ) ให้ข้าม hook นี้ด้วย:
   git commit --no-verify
```

commit ควรจะถูกยกเลิกและไฟล์ `.env` ยังคง staged อยู่ (ยังไม่ถูก commit เข้าประวัติ) ให้ unstage ทิ้งแล้วเพิ่มเข้า `.gitignore` แทน:

```bash
git restore --staged .env
echo ".env" >> .gitignore
git add .gitignore
git commit -m "chore: เพิ่ม .env เข้า gitignore"
```

commit นี้ควรจะผ่าน hook ได้ตามปกติ เพราะไม่มีไฟล์ `.env` อยู่ใน staged files แล้ว (มีแค่ `.gitignore` ที่แค่**เอ่ยถึง**ชื่อ `.env` แต่ตัวมันเองไม่ใช่ไฟล์ env)

### 590.3 เขียน `commit-msg` hook — เช็ครูปแบบ Conventional Commits

```bash
cat > .git/hooks/commit-msg << 'EOF'
#!/usr/bin/env bash
#
# commit-msg hook: บังคับให้ commit message ตรงตามรูปแบบ Conventional Commits
# รูปแบบ: <type>(<scope ไม่บังคับ>)<!ไม่บังคับ>: <description>

commit_msg_file="$1"
first_line=$(head -1 "$commit_msg_file")

valid_types="feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert"
pattern="^(${valid_types})(\([a-zA-Z0-9_./-]+\))?(!)?: .{1,100}$"

if ! [[ "$first_line" =~ $pattern ]]; then
    echo "❌ Commit message ไม่ตรงตามรูปแบบ Conventional Commits"
    echo ""
    echo "   รูปแบบที่ต้องการ:"
    echo "     <type>(<scope>): <description>"
    echo ""
    echo "   ตัวอย่างที่ถูกต้อง:"
    echo "     feat(login): เพิ่มระบบยืนยันตัวตนด้วย OTP"
    echo "     fix: แก้บั๊กการคำนวณราคาผิดพลาด"
    echo "     docs(readme): อัปเดตวิธีติดตั้ง"
    echo ""
    echo "   type ที่รองรับ: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert"
    echo ""
    echo "   ข้อความที่คุณพิมพ์มา:"
    echo "     \"$first_line\""
    exit 1
fi

exit 0
EOF

chmod +x .git/hooks/commit-msg
```

**ทดสอบว่า hook ทำงานจริง:**

```bash
# ทดสอบกรณีที่ควรถูกปฏิเสธ
echo "console.log('test')" > app.js
git add app.js
git commit -m "อัปเดตโค้ดนิดหน่อย"
```

ผลลัพธ์ที่คาดหวัง: commit ถูกปฏิเสธพร้อมข้อความอธิบายรูปแบบที่ถูกต้อง

```bash
# ทดสอบกรณีที่ควรผ่าน
git commit -m "feat(app): เพิ่มไฟล์ app.js เริ่มต้น"
```

ผลลัพธ์ที่คาดหวัง: commit สำเร็จตามปกติ เพราะข้อความตรงตามรูปแบบ `feat(app): ...`

### 590.4 ทดสอบว่า `--no-verify` ข้าม hook ได้จริง

```bash
echo "TOKEN=xyz" > .env.local
git add .env.local
git commit --no-verify -m "ข้อความไม่ตรงรูปแบบก็ยังผ่านได้"
```

ผลลัพธ์ที่คาดหวัง: commit สำเร็จ **ทั้งที่**ไฟล์เป็น `.env.local` (ควรถูกบล็อกโดย `pre-commit`) และข้อความก็ไม่ตรงรูปแบบ Conventional Commits (ควรถูกบล็อกโดย `commit-msg`) เพราะ `--no-verify` ข้ามทั้งสอง hook นี้ไปพร้อมกัน

ลอง `git log` ดูเพื่อยืนยันว่า commit นี้เข้าไปอยู่ในประวัติจริง:

```bash
git log --oneline -3
git show --stat HEAD
```

ล้างข้อมูลทดสอบก่อนไปต่อ (ทางเลือก):

```bash
git reset --soft HEAD~1
git restore --staged .env.local
rm .env.local
```

### 590.5 ทำให้ hook นี้แชร์กับทีมได้จริงด้วย `core.hooksPath`

ตอนนี้ hook ทั้งสองตัวอยู่ใน `.git/hooks/` ซึ่งจะไม่ถูก commit ไปกับโปรเจกต์ ให้ย้ายมาไว้ในโฟลเดอร์ที่ track ได้:

```bash
mkdir -p .githooks
cp .git/hooks/pre-commit .githooks/pre-commit
cp .git/hooks/commit-msg .githooks/commit-msg
chmod +x .githooks/pre-commit .githooks/commit-msg

git config core.hooksPath .githooks

git add .githooks
git commit -m "chore: ย้าย git hooks เข้า .githooks เพื่อแชร์กับทีมผ่าน core.hooksPath"
```

ทดสอบว่ายังทำงานปกติหลังย้าย:

```bash
echo "PASSWORD=123" > secret.env
git add secret.env
git commit -m "test"
```

ควรยังถูกบล็อกทั้งจาก `pre-commit` (ไฟล์ `.env`) เหมือนเดิม แม้ hook จะย้ายไปอยู่ใน `.githooks/` แล้วก็ตาม เพราะ `core.hooksPath` บอก Git ให้มองหา hook ที่โฟลเดอร์นี้แทน

ล้างไฟล์ทดสอบ:

```bash
git restore --staged secret.env
rm secret.env
```

### 590.6 Checklist ก่อนไป Part 60

- [ ] เข้าใจว่า Git Hooks คือ script ที่อยู่ใน `.git/hooks/` และ Git รันอัตโนมัติในจุดต่าง ๆ
- [ ] เข้าใจลำดับการทำงานของ hook ตอน commit: `pre-commit` → `prepare-commit-msg` → (editor) → `commit-msg` → (สร้าง commit) → `post-commit`
- [ ] เขียน `pre-commit` hook จริงที่ตรวจสอบไฟล์ staged และ block ไฟล์ `.env` ได้
- [ ] เขียน `commit-msg` hook จริงที่ตรวจสอบรูปแบบ Conventional Commits ได้
- [ ] เข้าใจว่า `pre-push` เหมาะกับงานหนักกว่า เช่นรัน full test suite ก่อนอนุญาตให้ push
- [ ] เข้าใจกลไก exit code: `0` = ผ่าน, ไม่ใช่ `0` = ถูกบล็อก
- [ ] เข้าใจปัญหาว่า `.git/hooks/` ไม่ถูก commit ไปกับ repo และรู้จักวิธีแก้อย่างน้อย 1 วิธี (`core.hooksPath`, husky, หรือ pre-commit framework)
- [ ] ตั้งค่า `core.hooksPath` ให้ hook ทำงานจากโฟลเดอร์ที่ track โดย Git ได้จริง
- [ ] รู้ว่า `--no-verify` ใช้ข้าม hook ได้ และรู้ว่าทำไมไม่ควรใช้พร่ำเพรื่อ
- [ ] ทดลองสร้างและทดสอบ hook จริงในโปรเจกต์ทดลองครบทุกขั้นตอน

---

## สรุป Part 59

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Git Hooks** คือสคริปต์ที่อยู่ใน `.git/hooks/` ที่ Git รันอัตโนมัติในจุดต่าง ๆ ของ workflow โดยชื่อไฟล์ต้องตรงเป๊ะและต้องมีสิทธิ์ execute
2. **`pre-commit`** ทำงานก่อนสร้าง commit เหมาะกับ lint, format, ตรวจสอบไฟล์ staged
3. **`commit-msg`** ทำงานหลังพิมพ์ commit message เสร็จ รับ path ไฟล์ message เป็น argument เหมาะกับตรวจสอบและแก้ไขรูปแบบข้อความ เช่น Conventional Commits
4. **`prepare-commit-msg`** ทำงานก่อนเปิด editor เหมาะกับการเติม template อัตโนมัติ ต้องเช็ค argument ตัวที่สองก่อนเสมอเพื่อไม่ให้ไปยุ่งกับ merge/squash/amend
5. **`post-commit`** ทำงานหลัง commit สำเร็จแล้ว **หยุด commit ไม่ได้อีก** เหมาะกับการแจ้งเตือนเท่านั้น
6. **`pre-push`** ทำงานก่อนส่งข้อมูลไป remote รับ input ผ่าน stdin เหมาะกับงานหนักอย่าง full test suite และการป้องกัน branch สำคัญ
7. Hook script ต้องมี **shebang** ที่ถูกต้อง มีสิทธิ์ **executable** และสื่อสารผลลัพธ์กับ Git ผ่าน **exit code** เท่านั้น (0 = ผ่าน, อื่น ๆ = ถูกบล็อก)
8. `.git/hooks/` **ไม่ถูก commit หรือแชร์ผ่าน Git โดย default** ทำให้ทีมต้องหาวิธีแชร์ hook ร่วมกัน ผ่าน `core.hooksPath`, **Husky** (นิยมในโลก Node.js) หรือ **pre-commit framework** (นิยมในโลก Python)
9. `--no-verify` ใช้ข้าม `pre-commit`, `commit-msg`, `pre-push` ได้ในกรณีฉุกเฉินจริง ๆ แต่ไม่ควรใช้พร่ำเพรื่อ เพราะจะทำลายจุดประสงค์ของการมี hook และควรมี CI/CD เป็นด่านสุดท้ายที่ข้ามไม่ได้เสมอ
10. เราได้ลงมือเขียนและทดสอบ `pre-commit` และ `commit-msg` hook จริงในโปรเจกต์ทดลอง รวมถึงตั้งค่าให้แชร์กับทีมผ่าน `core.hooksPath`

**ต่อไป:** [Part 60: Git Hooks: Server-side Hooks และ Automation](./part-060-git-hooks-server-side.md)
