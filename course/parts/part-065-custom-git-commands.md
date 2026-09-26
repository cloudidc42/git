# Part 65: Custom Git Commands และการเขียน Script ต่อยอด Git

> **Step ในหลักสูตรนี้:** Step 641–650
> **เฟส:** 6 — Git ขั้นสูงระดับ Internals, Hooks, LFS, Performance
> **เป้าหมายของ Part นี้:** เข้าใจกลไกที่แท้จริงที่ทำให้ Git ยอมให้เราสร้างคำสั่งของตัวเองแล้วเรียกใช้เหมือนเป็นส่วนหนึ่งของ Git (เช่น `git undo`, `git sync`, `git lfs`, `git flow`) เขียน custom command ได้เองทั้งด้วย Bash และ Python เข้าใจวิธีรับ argument และเข้าถึง context ของ repository ปัจจุบันอย่างถูกต้อง สร้างเครื่องมือที่ใช้งานจริงได้ แจกจ่ายให้ทีมใช้ร่วมกันได้ เปรียบเทียบกับ Alias จาก Part 13 ว่าควรเลือกใช้แบบไหนเมื่อไหร่ และปิดท้ายเฟส 6 ทั้งหมดด้วยการทบทวนภาพรวมตั้งแต่ Part 56 ถึง Part 65

---

## สารบัญของ Part นี้

- Step 641: Git อนุญาตให้สร้างคำสั่งเองได้อย่างไร (กลไก `git-xxx` ใน `PATH`)
- Step 642: เขียน Custom Git Command ตัวแรกด้วย Bash Script
- Step 643: เขียน Custom Git Command ด้วย Python สำหรับ Logic ที่ซับซ้อนกว่า
- Step 644: การรับ Argument และเข้าถึง Context ของ Repository ปัจจุบัน
- Step 645: ตัวอย่างใช้งานจริง — `git undo`, `git sync`, `git cleanup-branches`
- Step 646: การแจกจ่าย Custom Command ให้ทีมใช้งานร่วมกัน
- Step 647: Custom Command vs Alias — เมื่อไหร่ควรใช้แบบไหน
- Step 648: ให้ Custom Command อ่านค่าจาก Git Config ของผู้ใช้
- Step 649: เครื่องมือดังในวงการที่สร้างจากแนวคิดนี้ (git-extras, hub/gh, git-flow)
- Step 650: แบบฝึกหัด — สร้าง Custom Command ของตัวเอง และสรุปภาพรวมเฟส 6 ทั้งหมด

---

## Step 641: Git อนุญาตให้สร้างคำสั่งเองได้อย่างไร (กลไก `git-xxx` ใน `PATH`)

ตลอดหลักสูตรนี้เราใช้คำสั่งอย่าง `git status`, `git commit`, `git log`, `git flow feature start`, `git lfs track` มาเรื่อย ๆ จนหลายคนอาจคิดว่าคำสั่งเหล่านี้ "ถูกฝังไว้ในตัว Git ทั้งหมด" แต่ความจริงแล้ว **Git ถูกออกแบบมาตั้งแต่ต้นให้เป็นระบบที่ต่อยอดได้ (extensible)** และกลไกเบื้องหลังนั้นเรียบง่ายอย่างน่าประหลาดใจ

### กลไกการค้นหาคำสั่งของ Git

เวลาคุณพิมพ์ `git <คำ>` ไม่ว่าจะเป็น `git status` หรือ `git ของฉันเอง` Git จะทำงานตามลำดับประมาณนี้:

1. **ตรวจว่า `<คำ>` เป็น built-in subcommand หรือไม่** — คือคำสั่งที่ compile ไว้ในตัวโปรแกรม `git` เอง เช่น `status`, `commit`, `log`, `branch`, `rebase` ฯลฯ ถ้าใช่ ก็เรียกใช้ฟังก์ชันภายในนั้นตรง ๆ
2. **ถ้าไม่ใช่ built-in** Git จะไปค้นหาโปรแกรม (executable) ที่ชื่อว่า **`git-<คำ>`** ใน `PATH` ของระบบ (รวมถึงโฟลเดอร์ `--exec-path` ของ Git เองด้วย)
3. **ถ้าเจอไฟล์ `git-<คำ>` ที่ถูกทำเครื่องหมายว่า executable ได้** (`chmod +x`) Git จะ **exec** โปรแกรมนั้นทันที พร้อมส่ง argument ที่เหลือทั้งหมดต่อให้
4. **ถ้าไม่เจอเลย** Git จะแสดง error ประมาณ `git: '<คำ>' is not a git command. See 'git --help'.`

พูดให้ชัดเป็นสมการเดียว:

```
git foo arg1 arg2
        │
        ▼
ถ้า "foo" ไม่ใช่ built-in
        │
        ▼
Git จะพยายามรันไฟล์ชื่อ: git-foo arg1 arg2
```

### ทดลองดูให้เห็นภาพ

ลองสร้างไฟล์ทดสอบดูก่อนแบบง่ายที่สุด:

```bash
# สร้างโฟลเดอร์สำหรับเก็บคำสั่งส่วนตัว (ยังไม่ต้องมีอยู่ก็ได้)
mkdir -p ~/bin

# สร้างไฟล์ชื่อ git-foo
cat > ~/bin/git-foo << 'EOF'
#!/usr/bin/env bash
echo "สวัสดี ฉันคือ git-foo ที่แปลงร่างเป็น git foo ได้!"
EOF

# ทำให้ไฟล์เป็น executable
chmod +x ~/bin/git-foo

# เพิ่ม ~/bin เข้าไปใน PATH (ใส่ไว้ใน ~/.bashrc หรือ ~/.zshrc ให้ถาวร)
export PATH="$HOME/bin:$PATH"

# ทดสอบ
git foo
```

ผลลัพธ์ที่ได้:

```
สวัสดี ฉันคือ git-foo ที่แปลงร่างเป็น git foo ได้!
```

**นี่คือสิ่งเดียวกันเป๊ะ ๆ** กับที่เกิดขึ้นเวลาคุณพิมพ์ `git lfs`, `git flow`, `git extras` — เบื้องหลังของทุกคำสั่งเหล่านี้คือไฟล์ที่ชื่อ `git-lfs`, `git-flow`, `git-extras` วางอยู่ใน `PATH` ของเครื่องคุณเท่านั้นเอง ไม่มีเวทมนตร์อะไรซับซ้อนไปกว่านี้

### เงื่อนไขที่ต้องมีครบทั้ง 3 ข้อ

ให้ไฟล์กลายเป็น `git <คำสั่ง>` ได้ ไฟล์นั้นต้อง:

1. **ชื่อไฟล์ขึ้นต้นด้วย `git-`** ตามด้วยชื่อ subcommand ที่ต้องการ เช่น `git-undo`, `git-sync`, `git-cleanup-branches`
2. **มี execute permission** (`chmod +x ไฟล์นั้น`) — ถ้าลืมขั้นตอนนี้ Git จะมองไม่เห็นว่ามันเป็นคำสั่งที่รันได้
3. **อยู่ในไดเรกทอรีที่ Git ค้นหา** ซึ่งได้แก่ทุกไดเรกทอรีใน environment variable `$PATH` รวมถึงไดเรกทอรีที่ `git --exec-path` รายงาน (ที่เก็บ built-in helper ของ Git เอง)

ลองดูว่า `git --exec-path` ของเครื่องคุณชี้ไปที่ไหน:

```bash
git --exec-path
# เช่น: /usr/lib/git-core
```

โฟลเดอร์นี้คือที่ที่ Git เก็บ helper executable ของตัวเองจำนวนมาก (เช่น `git-add--interactive`, `git-submodule` แบบ script เก่า) คุณ**ไม่จำเป็น**ต้องเอา custom command ของตัวเองไปวางในโฟลเดอร์นี้ (และโดยทั่วไปไม่ควรทำ เพราะมักต้องใช้สิทธิ์ผู้ดูแลระบบและจะหายไปเวลาอัปเดต Git) — การวางไว้ใน `PATH` ปกติของผู้ใช้อย่าง `~/bin` หรือ `/usr/local/bin` เพียงพอแล้ว

### ภาษาโปรแกรมไม่จำกัด

Git ไม่สนใจเลยว่าไฟล์ `git-xxx` นั้นเขียนด้วยภาษาอะไร ขอแค่ **เมื่อสั่งรันแล้วมันทำงานได้ (executable)** เท่านั้น ดังนั้นคุณสามารถเขียนได้ด้วย:

| ภาษา | Shebang ตัวอย่าง | เหมาะกับ |
|---|---|---|
| Bash / Shell | `#!/usr/bin/env bash` | งานสั้น ๆ ที่เรียกคำสั่ง Git/Unix ต่อกันเป็นสาย (pipeline) |
| Python | `#!/usr/bin/env python3` | Logic ซับซ้อน, การ parse ข้อมูล, cross-platform |
| Ruby | `#!/usr/bin/env ruby` | เหมือน Python (ประวัติศาสตร์: `git-flow` รุ่นแรก ๆ และเครื่องมือยุค Rails นิยมใช้) |
| Node.js | `#!/usr/bin/env node` | ทีมที่ถนัด JavaScript อยู่แล้ว |
| Perl | `#!/usr/bin/env perl` | Git เองก็มี helper script ภายในบางตัวที่เขียนด้วย Perl |
| Compiled binary (Go, Rust, C) | (ไม่ต้องมี shebang) | ต้องการความเร็วสูงสุด หรือแจกจ่ายเป็นไฟล์เดียวไม่ต้องพึ่ง interpreter (เช่น `git-lfs` เขียนด้วย Go) |

### ข้อควรระวังสำคัญ

- **ห้ามตั้งชื่อไฟล์ซ้ำกับ built-in subcommand ที่มีอยู่แล้ว** เช่น อย่าสร้าง `git-commit` หรือ `git-status` ของตัวเอง เพราะ built-in command จะถูกเรียกก่อนเสมอ (custom command จะไม่มีวันถูกเรียกถึง) แต่ที่อันตรายกว่านั้นคือถ้าใครใน PATH ของคุณมีไฟล์ชื่อชนกับคำสั่งที่ **ไม่ใช่ built-in** (เช่น external command ของคนอื่นที่ติดตั้งไว้ก่อน) การเรียงลำดับ PATH จะเป็นตัวตัดสินว่าตัวไหนถูกเรียก — ควรตั้งชื่อคำสั่งของตัวเองให้ไม่ซ้ำกับของที่มีอยู่แล้วในระบบนิเวศ (เช่นตรวจสอบก่อนว่ามี `git-extras` หรือ plugin อื่นใช้ชื่อนั้นไปแล้วหรือยัง)
- `git help -a` (หรือ `git --help -a`) จะแสดงรายการคำสั่งทั้งหมดที่ Git มองเห็น **รวมถึง external command ที่ค้นเจอใน PATH ด้วย** ลองรันดูหลังจากสร้าง `git-foo` แล้วจะเห็นมันโผล่ในรายการ "external commands" ท้ายผลลัพธ์
- `git foo` และ `git-foo` ให้ผลเหมือนกันทุกประการ (คุณสามารถรัน `git-foo` ตรง ๆ โดยไม่ผ่าน `git` ก็ได้ ถ้ามันอยู่ใน PATH) — Git แค่เป็นตัว "ต่อคำ" ให้เท่านั้น

Step นี้คือรากฐานของทั้ง Part เลย — เมื่อเข้าใจกลไกนี้แล้ว Step ถัดไปเราจะเริ่มเขียนคำสั่งจริงกันทันที

---

## Step 642: เขียน Custom Git Command ตัวแรกด้วย Bash Script

มาลงมือเขียน custom command ที่ **มีประโยชน์จริง** ตัวแรกกันแบบละเอียดทีละขั้นตอน

### เป้าหมาย: `git today` — ดูว่าวันนี้เรา commit อะไรไปบ้าง

เป็นคำสั่งง่าย ๆ ที่โปรแกรมเมอร์หลายคนอยากมีไว้ใช้ทุกเช้า/ทุกเย็นเพื่อสรุปงานของตัวเอง

### ขั้นตอนที่ 1: เตรียมโฟลเดอร์เก็บ script

```bash
mkdir -p ~/bin
# ตรวจสอบว่า ~/bin อยู่ใน PATH แล้วหรือยัง
echo $PATH | tr ':' '\n' | grep "$HOME/bin"
```

ถ้ายังไม่มี ให้เพิ่มบรรทัดนี้ในไฟล์ `~/.bashrc` หรือ `~/.zshrc` แล้วสั่ง `source` ใหม่:

```bash
export PATH="$HOME/bin:$PATH"
```

### ขั้นตอนที่ 2: เขียน script

```bash
cat > ~/bin/git-today << 'EOF'
#!/usr/bin/env bash
set -euo pipefail

# ต้องอยู่ใน git repository เท่านั้น
if ! git rev-parse --is-inside-work-tree > /dev/null 2>&1; then
    echo "error: ไม่ได้อยู่ใน Git repository" >&2
    exit 1
fi

AUTHOR="$(git config user.email)"

echo "== Commit ของ $AUTHOR ที่ทำวันนี้ (branch: $(git rev-parse --abbrev-ref HEAD)) =="
git log \
    --since=midnight \
    --author="$AUTHOR" \
    --pretty=format:'%h  %ad  %s' \
    --date=format:'%H:%M'
echo
EOF

chmod +x ~/bin/git-today
```

### ขั้นตอนที่ 3: ทดสอบ

```bash
cd ~/git-course/some-project
git today
```

ผลลัพธ์ตัวอย่าง:

```
== Commit ของ you@example.com ที่ทำวันนี้ (branch: feature/login) ==
a1b2c3d  09:14  เพิ่มฟอร์ม login
d4e5f6a  11:02  แก้ validation ของ email
```

### อธิบายทีละส่วนของ Script

- `set -euo pipefail` — เป็น **best practice มาตรฐานของ Bash script ทุกตัว**:
  - `-e` ทำให้ script หยุดทันทีถ้ามีคำสั่งไหน exit ด้วย error code ที่ไม่ใช่ 0
  - `-u` ทำให้ script error ทันทีถ้าอ้างถึงตัวแปรที่ไม่เคยถูกกำหนดค่า (ป้องกัน typo)
  - `pipefail` ทำให้ pipeline (`cmd1 | cmd2`) รายงาน error ถ้า `cmd1` fail แม้ `cmd2` จะสำเร็จก็ตาม
- `git rev-parse --is-inside-work-tree` — ใช้ตรวจสอบว่า script กำลังถูกเรียกจากภายใน working tree ของ Git repository จริงหรือไม่ (คืนค่า `true`/`false` และ exit code 0/128) — เป็นแพทเทิร์นมาตรฐานที่ custom command เกือบทุกตัวควรมี เพื่อกัน error ที่อ่านยากถ้าผู้ใช้เผลอรันนอก repo
- `--since=midnight` — เป็น flag ของ `git log` ที่รองรับการเขียนเวลาแบบธรรมชาติ (natural language date) ตั้งแต่เที่ยงคืนของวันนี้ถึงปัจจุบัน
- `--author="$AUTHOR"` — กรองเฉพาะ commit ของผู้ใช้คนปัจจุบัน (อ่านจาก `git config user.email`)
- `--pretty=format:` และ `--date=format:` — จัดรูปแบบผลลัพธ์ให้อ่านง่าย (เราเคยเรียนเรื่อง pretty format นี้ไปแล้วใน Part 05)

### ทำไมต้องใช้ Bash สำหรับงานแบบนี้

Bash เหมาะมากกับ custom command ที่ทำหน้าที่หลักคือ **เรียกคำสั่ง Git หรือ Unix ต่อกันเป็นสาย** เพราะ:

- ไม่ต้องติดตั้ง interpreter เพิ่ม (มีอยู่แล้วในทุกเครื่อง Linux/macOS)
- Syntax สำหรับเรียกโปรแกรมภายนอกและต่อ pipe สั้นและเป็นธรรมชาติที่สุด
- Startup time เร็วมาก (ไม่มี overhead ของการโหลด runtime ภาษาอื่น)

แต่พอ logic เริ่มซับซ้อนขึ้น — ต้อง parse JSON, ทำ data structure ซับซ้อน, เขียน unit test ให้กับ script, หรือรองรับ argument หลายรูปแบบ — Bash จะเริ่มอ่านยากและเสี่ยง bug มากขึ้นเรื่อย ๆ นี่คือจุดที่ Step ถัดไปจะเข้ามาช่วย

---

## Step 643: เขียน Custom Git Command ด้วย Python สำหรับ Logic ที่ซับซ้อนกว่า

เมื่อ custom command ของคุณต้องทำอะไรที่ซับซ้อนกว่าการต่อคำสั่งเป็นสาย เช่น การนับสถิติ, จัดกลุ่มข้อมูล, สร้างรายงาน, หรือรองรับหลาย flag พร้อม validation ที่ดี — **Python คือตัวเลือกยอดนิยมที่สุด** สำหรับ Git tooling (Git เองก็เคยใช้ Python ใน contrib tools หลายตัว และเครื่องมือดัง ๆ อย่าง `git-review` ก็เขียนด้วย Python)

### เป้าหมาย: `git stats` — สรุปสถิติการ commit ของแต่ละคนใน repository

```bash
cat > ~/bin/git-stats << 'EOF'
#!/usr/bin/env python3
"""git-stats: สรุปจำนวน commit และไฟล์ที่เปลี่ยนแปลงของแต่ละคนใน repository"""

import subprocess
import sys
from collections import defaultdict


def run_git(args):
    """เรียกคำสั่ง git และคืนค่า stdout เป็น string"""
    result = subprocess.run(
        ["git"] + args,
        capture_output=True,
        text=True,
    )
    if result.returncode != 0:
        print(result.stderr.strip(), file=sys.stderr)
        sys.exit(result.returncode)
    return result.stdout


def check_inside_repo():
    result = subprocess.run(
        ["git", "rev-parse", "--is-inside-work-tree"],
        capture_output=True,
        text=True,
    )
    if result.returncode != 0:
        print("error: ไม่ได้อยู่ใน Git repository", file=sys.stderr)
        sys.exit(1)


def main():
    check_inside_repo()

    # จำกัดจำนวน commit ที่ดูย้อนหลังผ่าน argument แรก (ถ้ามี) เช่น git stats 100
    limit = sys.argv[1] if len(sys.argv) > 1 else "-n"
    max_count = sys.argv[1] if len(sys.argv) > 1 else "500"

    log_output = run_git([
        "log",
        f"-n{max_count}",
        "--pretty=format:%an",
    ])

    counts = defaultdict(int)
    for line in log_output.splitlines():
        if line.strip():
            counts[line.strip()] += 1

    if not counts:
        print("ไม่พบ commit ใด ๆ")
        return

    print(f"== สรุปสถิติ commit ({max_count} commit ล่าสุด) ==")
    for author, count in sorted(counts.items(), key=lambda kv: kv[1], reverse=True):
        bar = "#" * min(count, 40)
        print(f"{author:<25} {count:>4}  {bar}")


if __name__ == "__main__":
    main()
EOF

chmod +x ~/bin/git-stats
```

ทดสอบ:

```bash
git stats
git stats 50   # ดูแค่ 50 commit ล่าสุด
```

ผลลัพธ์ตัวอย่าง:

```
== สรุปสถิติ commit (500 commit ล่าสุด) ==
Somchai Devteam            42  ########################################
Suda Backend                18  ##################
Anan Frontend                7  #######
```

### ทำไม Python ถึงเหมาะกับงานแบบนี้มากกว่า

| คุณสมบัติ | Bash | Python |
|---|---|---|
| Parse ข้อความ/สร้าง data structure ซับซ้อน | ยากและอ่านยาก (ต้องพึ่ง `awk`/`sed`) | มี `dict`, `list`, `collections` ให้ใช้ตรง ๆ |
| จัดการ error/exception | ต้องเช็ค exit code เอง | มี `try/except` ที่ชัดเจน |
| เขียน unit test | ทำได้ยาก (ต้อง mock shell) | ใช้ `unittest`/`pytest` ได้ตรง ๆ |
| Cross-platform (รองรับ Windows โดยไม่ต้องพึ่ง Git Bash) | ต้องพึ่ง POSIX shell | รันได้บน Windows ถ้ามี Python ติดตั้ง |
| จัดการ JSON/YAML (เช่นอ่าน config ไฟล์เพิ่มเติม) | ยุ่งยากมาก | มี `json` module ในตัว |
| Startup time | เร็วมาก | ช้ากว่าเล็กน้อย (ต้องโหลด interpreter) |

### ข้อควรระวังของการใช้ Python

1. **ต้องมี Python 3 ติดตั้งอยู่ในเครื่องผู้ใช้ทุกคนในทีม** — ถ้าทีมมีเครื่องที่ไม่มี Python (พบได้ในบางเครื่อง Windows ที่ไม่ได้ติดตั้งไว้) คำสั่งจะใช้งานไม่ได้ทันที ต้องพิจารณาเรื่องนี้ตอนแจกจ่าย (ดู Step 646)
2. **`#!/usr/bin/env python3` ดีกว่า `#!/usr/bin/python3`** เพราะ `env` จะค้นหา `python3` ที่อยู่ใน `PATH` ปัจจุบันของผู้ใช้ ทำให้ทำงานได้แม้ Python ถูกติดตั้งผ่าน `pyenv`, `conda`, หรือ virtual environment
3. **หลีกเลี่ยง third-party library ที่ต้อง `pip install`** ถ้าเป็นไปได้ — ใช้แค่ standard library (`subprocess`, `sys`, `os`, `json`, `argparse`) เพื่อให้ script รันได้ทันทีโดยไม่ต้องติดตั้งอะไรเพิ่ม ถ้าจำเป็นต้องพึ่ง library ภายนอกจริง ๆ ให้บันทึกไว้ชัดเจนใน README ตอนแจกจ่าย

---

## Step 644: การรับ Argument และเข้าถึง Context ของ Repository ปัจจุบัน

Custom command ที่มีประโยชน์จริงต้องทำ 2 อย่างให้ดี: **รับ argument จากผู้ใช้อย่างถูกต้อง** และ **รู้ context ของ repository ที่กำลังทำงานอยู่**

### 644.1 การรับ Argument

เมื่อคุณพิมพ์ `git undo --hard 3` สิ่งที่ Git ทำคือเรียก:

```
git-undo --hard 3
```

นั่นคือ **argument ทุกตัวที่ตามหลังชื่อ subcommand จะถูกส่งต่อให้ script ทั้งหมด แบบเดียวกับที่ programโปรแกรมปกติรับ argument จาก command line**

ใน Bash เข้าถึงผ่านตัวแปรมาตรฐาน:

```bash
#!/usr/bin/env bash
echo "argument ตัวที่ 1: $1"
echo "argument ตัวที่ 2: $2"
echo "argument ทั้งหมด: $@"
echo "จำนวน argument: $#"
```

ใน Python เข้าถึงผ่าน `sys.argv` (โดย `sys.argv[0]` คือ path ของ script เอง ส่วน argument จริงเริ่มที่ `sys.argv[1]`):

```python
import sys
print("argument ทั้งหมด:", sys.argv[1:])
```

สำหรับ custom command ที่มี flag หลายแบบ (`--hard`, `-n`, `--branch=main`) แนะนำให้ใช้ library `argparse` ของ Python แทนการ parse เอง เพราะได้ `--help` อัตโนมัติและ error message ที่เป็นมาตรฐาน:

```python
import argparse

parser = argparse.ArgumentParser(prog="git undo", description="ย้อน commit ล่าสุดอย่างปลอดภัย")
parser.add_argument("count", nargs="?", type=int, default=1, help="จำนวน commit ที่จะย้อน")
parser.add_argument("--hard", action="store_true", help="ทิ้งการเปลี่ยนแปลงทั้งหมด (อันตราย)")
args = parser.parse_args()
```

ฝั่ง Bash ที่ต้องการ flag ซับซ้อนสามารถใช้ builtin `getopts` ได้ แต่โดยทั่วไป ถ้า custom command เริ่มต้องมี flag เยอะ นี่คือสัญญาณว่าถึงเวลาย้ายไปเขียนด้วย Python แล้ว (ตามที่พูดถึงใน Step 643)

### 644.2 การเข้าถึง Context ของ Repository ปัจจุบัน

สิ่งสำคัญที่ต้องเข้าใจให้แม่นคือ: **เมื่อ Git dispatch ไปยัง `git-xxx` มันไม่ได้เปลี่ยน current working directory ให้คุณ** — script จะเริ่มทำงานที่ directory เดียวกับที่ผู้ใช้พิมพ์คำสั่ง `git xxx` เป๊ะ ๆ ถึงแม้ผู้ใช้จะอยู่ในโฟลเดอร์ย่อยลึก ๆ ของ repository ก็ตาม (เช่น `src/components/`)

ดังนั้น custom command ที่ต้องอ้างอิงถึงไฟล์แบบ absolute path หรือทำงานกับทั้ง repository จำเป็นต้องถามหา context เหล่านี้เองผ่านคำสั่ง plumbing ที่เราเรียนไปแล้วใน Part ก่อนหน้าของเฟสนี้:

| ต้องการรู้ | คำสั่งที่ใช้ | ตัวอย่างผลลัพธ์ |
|---|---|---|
| Path เต็มของ root ของ working tree | `git rev-parse --show-toplevel` | `/home/user/myproject` |
| Path ของ `.git` directory | `git rev-parse --git-dir` | `.git` หรือ `/home/user/myproject/.git` |
| อยู่ใน working tree ของ Git หรือไม่ | `git rev-parse --is-inside-work-tree` | `true` / `false` |
| ชื่อ branch ปัจจุบัน | `git rev-parse --abbrev-ref HEAD` | `feature/login` (หรือ `HEAD` ถ้า detached) |
| Path ของ cwd relative จาก root (prefix) | `git rev-parse --show-prefix` | `src/components/` |
| ref ที่ HEAD ชี้ไปแบบเต็ม | `git symbolic-ref HEAD` | `refs/heads/feature/login` |
| Upstream branch ของ branch ปัจจุบัน | `git rev-parse --abbrev-ref --symbolic-full-name @{u}` | `origin/feature/login` |

ตัวอย่างการใช้ใน Bash เพื่อทำให้ script ทำงานถูกต้องไม่ว่าผู้ใช้จะยืนอยู่ตรงไหนใน repository:

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO_ROOT="$(git rev-parse --show-toplevel)"
CURRENT_BRANCH="$(git rev-parse --abbrev-ref HEAD)"

echo "ทำงานอยู่ใน repository: $REPO_ROOT"
echo "branch ปัจจุบัน: $CURRENT_BRANCH"

# ตัวอย่าง: cd ไปที่ root ก่อนทำงานกับไฟล์แบบ absolute
cd "$REPO_ROOT"
```

และใน Python:

```python
import subprocess

def git_output(args):
    return subprocess.run(
        ["git"] + args, capture_output=True, text=True, check=True
    ).stdout.strip()

repo_root = git_output(["rev-parse", "--show-toplevel"])
current_branch = git_output(["rev-parse", "--abbrev-ref", "HEAD"])
```

### 644.3 ตัวแปรสภาพแวดล้อมที่ Git ตั้งค่าให้

นอกจากการเรียก `git rev-parse` เองแล้ว Git ยังตั้งค่า environment variable บางตัวให้กับคำสั่งภายนอกและ shell alias โดยอัตโนมัติ ที่สำคัญที่สุดคือ:

- **`GIT_PREFIX`** — เท่ากับผลลัพธ์ของ `git rev-parse --show-prefix` คือ path ของ directory ปัจจุบันเทียบกับ root ของ repository (มี `/` ปิดท้าย หรือเป็นค่าว่างถ้าอยู่ที่ root พอดี) ตัวแปรนี้มีประโยชน์เวลา custom command ต้องรายงาน path ของไฟล์กลับไปให้ผู้ใช้ในรูปแบบเดียวกับที่ Git เองใช้ (relative จาก cwd ไม่ใช่จาก root)
- **`GIT_DIR`** — ถ้าถูกตั้งค่าไว้ (เช่นตอนถูกเรียกจาก hook หรือใน environment พิเศษ) จะชี้ไปยัง `.git` directory โดยตรง

เพื่อความปลอดภัยและรองรับ Git ได้หลายเวอร์ชัน แนวทางที่แนะนำคือ **ใช้ค่าจาก environment variable ถ้ามี แต่ fallback ไปเรียก `git rev-parse` เองเสมอถ้าไม่มี** แทนที่จะพึ่งพา environment variable เพียงอย่างเดียว:

```bash
PREFIX="${GIT_PREFIX:-$(git rev-parse --show-prefix)}"
```

แนวทางนี้ทำให้ custom command ของคุณทำงานถูกต้องไม่ว่าจะถูกเรียกจากที่ไหนใน repository ก็ตาม — ซึ่งเป็นพฤติกรรมมาตรฐานที่ผู้ใช้ Git คาดหวังจากทุกคำสั่ง (built-in หรือ custom ก็ตาม)

---

## Step 645: ตัวอย่างใช้งานจริง — `git undo`, `git sync`, `git cleanup-branches`

ถึงเวลาสร้างเครื่องมือที่นักพัฒนาจะใช้จริงทุกวัน ทั้ง 3 ตัวนี้ถูกออกแบบให้ **ปลอดภัยเป็นอันดับแรก** เสมอ (fail-safe) ไม่ใช่แค่ทำงานได้

### 645.1 `git undo` — ย้อน commit ล่าสุดแบบปลอดภัย

ปัญหาที่พบบ่อย: commit ไปแล้วรู้สึกผิด อยากย้อนกลับ แต่กลัวเผลอใช้ `git reset --hard` แล้วโค้ดหาย

```bash
cat > ~/bin/git-undo << 'EOF'
#!/usr/bin/env bash
set -euo pipefail

if ! git rev-parse --is-inside-work-tree > /dev/null 2>&1; then
    echo "error: ไม่ได้อยู่ใน Git repository" >&2
    exit 1
fi

# ตรวจสอบว่ามี commit อย่างน้อย 1 ตัวให้ย้อน
if ! git rev-parse HEAD~1 > /dev/null 2>&1; then
    echo "error: ไม่มี commit ก่อนหน้าให้ย้อนกลับ" >&2
    exit 1
fi

CURRENT_BRANCH="$(git rev-parse --abbrev-ref HEAD)"
LAST_COMMIT_MSG="$(git log -1 --pretty=format:'%h %s')"

# เช็คว่า commit ล่าสุดถูก push ขึ้น upstream ไปแล้วหรือยัง
if git rev-parse --abbrev-ref --symbolic-full-name '@{u}' > /dev/null 2>&1; then
    UPSTREAM="$(git rev-parse --abbrev-ref --symbolic-full-name '@{u}')"
    if git merge-base --is-ancestor HEAD "$UPSTREAM" 2>/dev/null; then
        : # HEAD เก่ากว่าหรือเท่ากับ upstream อยู่แล้ว ไม่มีอะไรใหม่ต้อง undo
    fi
    if git merge-base --is-ancestor "$(git rev-parse HEAD)" "$UPSTREAM" && \
       [ "$(git rev-parse HEAD)" != "$(git rev-parse "$UPSTREAM")" ]; then
        echo "คำเตือน: commit ล่าสุดดูเหมือนถูก push ไปที่ $UPSTREAM แล้ว" >&2
        echo "การ undo อาจทำให้ประวัติของคุณกับของทีมไม่ตรงกัน" >&2
        read -r -p "ยืนยันจะทำต่อหรือไม่? (y/N) " confirm
        if [[ "$confirm" != "y" && "$confirm" != "Y" ]]; then
            echo "ยกเลิกการ undo"
            exit 0
        fi
    fi
fi

echo "กำลังย้อน commit: $LAST_COMMIT_MSG (branch: $CURRENT_BRANCH)"
echo "การเปลี่ยนแปลงจะถูกเก็บไว้ในสถานะ staged (ไม่หาย)"

# --soft: ย้าย HEAD กลับ 1 commit แต่ "ไม่แตะ" working directory และ staging area
# ทำให้การเปลี่ยนแปลงทั้งหมดของ commit ที่ถูก undo กลับไปอยู่ในสถานะ staged พร้อม commit ใหม่
git reset --soft HEAD~1

echo "เสร็จแล้ว รันคำสั่ง 'git status' เพื่อดูการเปลี่ยนแปลงที่ถูกย้อนกลับมา"
EOF

chmod +x ~/bin/git-undo
```

**จุดออกแบบที่สำคัญ:**

- ใช้ `git reset --soft HEAD~1` เป็นค่าเริ่มต้นเสมอ **ไม่ใช่** `--hard` — เพราะ `--soft` ย้ายแค่ตำแหน่งของ branch pointer (HEAD) กลับไป 1 commit โดยไม่แตะ staging area หรือ working directory เลย ผลคือโค้ดทั้งหมดที่เคยอยู่ใน commit นั้นจะกลับมาอยู่ในสถานะ staged ให้คุณแก้ไขหรือ commit ใหม่ได้ทันที **ไม่มีข้อมูลสูญหาย**
- ตรวจสอบก่อนว่า commit ที่จะ undo ถูก push ไป upstream แล้วหรือยัง (`git merge-base --is-ancestor`) — ถ้าถูก push ไปแล้ว การ reset จะทำให้ local branch กับ remote branch ไม่ตรงกัน (ต้อง force push ทีหลัง ซึ่งเสี่ยงกับเพื่อนร่วมทีม) จึงเตือนและขอ confirm ก่อนเสมอ
- เพราะใช้ `git reset --soft` เท่านั้น (ไม่มีการลบ object ใด ๆ ออกจาก Git) แม้พลาดไปจริง ๆ ก็ยังกู้คืน commit เดิมได้เสมอผ่าน `git reflog` (ที่เรียนไปแล้วใน Part 14)

### 645.2 `git sync` — fetch + rebase อัตโนมัติกับ upstream

ปัญหาที่พบบ่อย: ทุกเช้าต้องพิมพ์ `git fetch origin && git rebase origin/main` ซ้ำ ๆ

```bash
cat > ~/bin/git-sync << 'EOF'
#!/usr/bin/env bash
set -euo pipefail

if ! git rev-parse --is-inside-work-tree > /dev/null 2>&1; then
    echo "error: ไม่ได้อยู่ใน Git repository" >&2
    exit 1
fi

# ต้องมี working directory สะอาดก่อน rebase เสมอ
if [ -n "$(git status --porcelain)" ]; then
    echo "error: มีการเปลี่ยนแปลงที่ยังไม่ commit อยู่ กรุณา commit หรือ 'git stash' ก่อน" >&2
    exit 1
fi

CURRENT_BRANCH="$(git rev-parse --abbrev-ref HEAD)"

if ! git rev-parse --abbrev-ref --symbolic-full-name '@{u}' > /dev/null 2>&1; then
    echo "error: branch '$CURRENT_BRANCH' ยังไม่มี upstream branch ให้ sync ด้วย" >&2
    echo "ตั้งค่าก่อนด้วย: git branch --set-upstream-to=origin/$CURRENT_BRANCH" >&2
    exit 1
fi

REMOTE_NAME="$(git config "branch.${CURRENT_BRANCH}.remote")"

echo "กำลัง fetch ล่าสุดจาก $REMOTE_NAME ..."
git fetch "$REMOTE_NAME"

echo "กำลัง rebase '$CURRENT_BRANCH' บน '@{u}' ..."
if git rebase '@{u}'; then
    echo "sync สำเร็จ! '$CURRENT_BRANCH' อยู่บนสุดของ upstream แล้ว"
else
    echo "เกิด conflict ระหว่าง rebase — แก้ conflict แล้วรัน 'git rebase --continue'" >&2
    echo "หรือยกเลิกทั้งหมดด้วย 'git rebase --abort'" >&2
    exit 1
fi
EOF

chmod +x ~/bin/git-sync
```

**จุดออกแบบที่สำคัญ:**

- เช็ค `git status --porcelain` ก่อนเสมอ — ถ้ามีการเปลี่ยนแปลงค้างอยู่ (uncommitted changes) จะปฏิเสธไม่ทำงานทันที เพราะการ rebase ทับ working directory ที่ยังไม่ clean มีความเสี่ยงสูงต่อ conflict ที่ไม่คาดคิด
- ใช้ `@{u}` (upstream shorthand ของ Git) แทนการ hardcode ชื่อ branch/remote — ทำให้ script ใช้ได้กับทุก branch ไม่ว่าจะ track กับ remote ชื่ออะไร หรือ branch ชื่ออะไรก็ตาม
- เมื่อ rebase ล้มเหลว (มี conflict) จะไม่พยายามแก้ไขอัตโนมัติ แต่บอกผู้ใช้อย่างชัดเจนว่าต้องทำอะไรต่อ (`--continue` หรือ `--abort`) — custom command ที่ดีไม่ควรพยายาม "ฉลาดเกินไป" ในจุดที่ต้องใช้การตัดสินใจของมนุษย์

### 645.3 `git cleanup-branches` — ลบ local branch ที่ merge แล้วทั้งหมด

ปัญหาที่พบบ่อย: หลังจากทำงานมานาน มี local branch ที่ merge เข้า `main` ไปแล้วค้างอยู่เป็นสิบ ๆ branch ทำให้ `git branch` รกและหา branch ที่ยังใช้งานจริงยาก

```bash
cat > ~/bin/git-cleanup-branches << 'EOF'
#!/usr/bin/env bash
set -euo pipefail

if ! git rev-parse --is-inside-work-tree > /dev/null 2>&1; then
    echo "error: ไม่ได้อยู่ใน Git repository" >&2
    exit 1
fi

# กำหนด base branch ที่ใช้เทียบว่า merge แล้วหรือยัง (ปรับได้ผ่าน git config เดี๋ยวเรียนใน Step 648)
BASE_BRANCH="${1:-$(git config --get cleanup-branches.base || echo main)}"

if ! git show-ref --verify --quiet "refs/heads/$BASE_BRANCH"; then
    echo "error: ไม่พบ base branch '$BASE_BRANCH' ในเครื่อง" >&2
    exit 1
fi

CURRENT_BRANCH="$(git rev-parse --abbrev-ref HEAD)"

echo "กำลังค้นหา branch ที่ merge เข้า '$BASE_BRANCH' แล้ว ..."

# --merged แสดง branch ที่ทุก commit ของมันอยู่ใน history ของ BASE_BRANCH แล้ว
# กรอง BASE_BRANCH เอง, branch ปัจจุบัน, และ branch สำคัญออกจากรายการที่จะลบเสมอ
PROTECTED_PATTERN="^\*|^\s*${BASE_BRANCH}$|^\s*main$|^\s*master$|^\s*develop$|^\s*${CURRENT_BRANCH}$"

MERGED_BRANCHES="$(git branch --merged "$BASE_BRANCH" | grep -vE "$PROTECTED_PATTERN" || true)"

if [ -z "$MERGED_BRANCHES" ]; then
    echo "ไม่มี branch ที่ merge แล้วให้ลบ (repository สะอาดอยู่แล้ว)"
    exit 0
fi

echo "branch ต่อไปนี้จะถูกลบ (merge เข้า '$BASE_BRANCH' แล้ว):"
echo "$MERGED_BRANCHES" | sed 's/^/  - /'
echo
read -r -p "ยืนยันลบ branch เหล่านี้หรือไม่? (y/N) " confirm

if [[ "$confirm" == "y" || "$confirm" == "Y" ]]; then
    echo "$MERGED_BRANCHES" | xargs -r git branch -d
    echo "ลบเสร็จแล้ว"
else
    echo "ยกเลิก"
fi
EOF

chmod +x ~/bin/git-cleanup-branches
```

ทดสอบ:

```bash
git cleanup-branches         # เทียบกับ main โดย default
git cleanup-branches develop # เทียบกับ develop แทน
```

**จุดออกแบบที่สำคัญ:**

- ใช้ `git branch --merged` ซึ่งเป็นคำสั่งที่ปลอดภัยตั้งแต่ต้น — มันแสดงเฉพาะ branch ที่ **ทุก commit ของมันเป็นบรรพบุรุษของ base branch แล้ว** เท่านั้น ถ้า branch ไหนยังมีงานที่ไม่ได้ merge Git จะไม่แสดงชื่อมันออกมาเลย
- ใช้ `git branch -d` (ตัวพิมพ์เล็ก) ไม่ใช่ `-D` — `-d` จะปฏิเสธการลบถ้า branch นั้นยังไม่ถูก merge จริง (เป็นการป้องกันซ้อนอีกชั้นแม้ script จะกรองมาให้แล้วก็ตาม) ในขณะที่ `-D` จะลบแบบบังคับโดยไม่สนใจว่า merge แล้วหรือยัง ซึ่งอันตรายเกินไปสำหรับ automation
- กัน branch สำคัญ (`main`, `master`, `develop`, base branch ที่เลือก, และ branch ปัจจุบัน) ไม่ให้ถูกลบโดยไม่ตั้งใจเสมอ
- มีขั้นตอนถามยืนยัน (`read -r -p`) ก่อนลบจริงทุกครั้ง — ไม่ลบอัตโนมัติแบบไม่ถามผู้ใช้ ถึงแม้จะเป็นแค่ local branch ก็ตาม

ทั้ง 3 ตัวอย่างนี้แสดงหลักการออกแบบ custom command ที่ดีเหมือนกันหมด: **ตรวจสอบสถานะก่อนเสมอ, เลือกกลไกที่ปลอดภัยที่สุดของ Git เป็นค่าเริ่มต้น, และสื่อสารกับผู้ใช้อย่างชัดเจนเมื่อมีความเสี่ยง**

---

## Step 646: การแจกจ่าย Custom Command ให้ทีมใช้งานร่วมกัน

เขียน custom command ไว้ใช้คนเดียวเป็นเรื่องหนึ่ง แต่การทำให้ **ทั้งทีมใช้ชุดเครื่องมือเดียวกัน** เป็นอีกเรื่องหนึ่งที่ต้องวางแผน มี 3 แนวทางหลักที่นิยมใช้ในวงการจริง

### 646.1 แจกจ่ายผ่าน Dotfiles Repository

วิธีที่นิยมที่สุดในหมู่นักพัฒนาที่มี dotfiles repo อยู่แล้ว (repo ที่เก็บไฟล์ config ส่วนตัวอย่าง `.bashrc`, `.vimrc`, `.gitconfig`) คือการเพิ่มโฟลเดอร์ `bin/` เข้าไปในนั้น:

```
dotfiles/
├── .bashrc
├── .gitconfig
├── bin/
│   ├── git-undo
│   ├── git-sync
│   └── git-cleanup-branches
└── install.sh
```

โดย `install.sh` จะทำหน้าที่ symlink หรือ copy ไฟล์เหล่านี้ไปวางในตำแหน่งที่ถูกต้องและเพิ่มเข้า `PATH`:

```bash
#!/usr/bin/env bash
set -euo pipefail

DOTFILES_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

mkdir -p "$HOME/bin"
for cmd in "$DOTFILES_DIR"/bin/git-*; do
    ln -sf "$cmd" "$HOME/bin/$(basename "$cmd")"
done

echo "ติดตั้ง custom git commands เสร็จแล้ว"
echo "ตรวจสอบว่า \$HOME/bin อยู่ใน PATH ของคุณ (เพิ่มใน ~/.bashrc หรือ ~/.zshrc ถ้ายังไม่มี):"
echo '  export PATH="$HOME/bin:$PATH"'
```

สมาชิกในทีมแค่ `git clone` dotfiles repo แล้วรัน `./install.sh` ครั้งเดียวก็ได้ครบทุกคำสั่งพร้อมกัน และเมื่อมีการอัปเดต script ก็แค่ `git pull` ใน dotfiles repo (เพราะใช้ symlink ไม่ใช่ copy ไฟล์ตรง ๆ)

### 646.2 แจกจ่ายผ่าน Script Installer แบบ One-liner

สำหรับทีมที่ต้องการติดตั้งเร็วโดยไม่ต้อง clone repo เต็ม อาจทำ installer แบบดาวน์โหลดแล้วรันทันที (คล้ายวิธีที่เครื่องมือดัง ๆ อย่าง `nvm` หรือ `rustup` ใช้):

```bash
curl -fsSL https://internal.company.com/git-tools/install.sh | bash
```

**ข้อควรระวังสำคัญ:** วิธีนี้สะดวกแต่มีความเสี่ยงด้านความปลอดภัย เพราะเป็นการรัน script จากอินเทอร์เน็ตโดยตรงโดยไม่ได้ตรวจสอบเนื้อหาก่อน ควรใช้เฉพาะกับ URL ภายในองค์กรที่เชื่อถือได้เท่านั้น และควรมี checksum หรือ signature verification ประกอบเพื่อความปลอดภัยเพิ่มเติมในสภาพแวดล้อมที่จริงจัง

### 646.3 แจกจ่ายผ่าน Package Manager

สำหรับเครื่องมือที่โตขึ้นเป็นโปรเจกต์จริงจัง การห่อเป็น package แล้ว publish ผ่าน package manager มาตรฐานของแต่ละแพลตฟอร์มเป็นวิธีที่เป็นมืออาชีพที่สุด และเป็นวิธีที่เครื่องมือดังในวงการ (ดู Step 649) เลือกใช้จริง:

| Package Manager | แพลตฟอร์ม | ตัวอย่างคำสั่งติดตั้ง |
|---|---|---|
| Homebrew | macOS / Linux | `brew install our-team/tap/git-tools` |
| APT | Debian/Ubuntu | `apt install git-tools` (ต้องมี `.deb` และ repository ของตัวเอง) |
| npm (แบบ global) | ทุกแพลตฟอร์มที่มี Node.js | `npm install -g @company/git-tools` |
| pipx | ทุกแพลตฟอร์มที่มี Python | `pipx install git-tools` |
| Scoop / Chocolatey | Windows | `scoop install git-tools` |

ข้อดีของวิธีนี้คือมีระบบ **version management, uninstall, และ update** ให้ในตัวโดยไม่ต้องทำเอง และผู้ใช้คุ้นเคยกับคำสั่งติดตั้งมาตรฐานอยู่แล้ว ข้อเสียคือต้องใช้เวลาตั้งค่า packaging และดูแล release process เพิ่มเติม เหมาะกับเครื่องมือที่จะใช้ระยะยาวและมีคนดูแลเป็นทีมจริงจัง ไม่เหมาะกับ script เล็ก ๆ ที่ทำไว้ใช้ในทีมเดียว

### หลักการเลือกวิธีแจกจ่าย

| สถานการณ์ | วิธีที่แนะนำ |
|---|---|
| ทีมเล็ก ใช้ dotfiles กันอยู่แล้ว | Dotfiles repo + install script |
| ต้องการติดตั้งเร็วในเครื่อง CI/CD หรือ container | One-liner installer (จาก URL ภายในที่เชื่อถือได้) |
| เครื่องมือใช้ทั้งบริษัท มีคนดูแลระยะยาว | Package manager (Homebrew tap / npm / pipx) |
| ทดลองใช้คนเดียวหรือทีมเล็กมาก | Copy ไฟล์ตรง ๆ ไป `~/bin` (แบบที่ทำใน Step 642-645) |

---

## Step 647: Custom Command vs Alias — เมื่อไหร่ควรใช้แบบไหน

ใน Part 13 เราเรียนเรื่อง **Git Alias** ไปแล้ว ซึ่งก็เป็นวิธีสร้าง "คำสั่งย่อ" ให้ Git เหมือนกัน คำถามที่ตามมาคือ: แล้วเมื่อไหร่ควรใช้ Alias เมื่อไหร่ควรเขียนเป็น Custom Command (`git-xxx` script) แยก?

### ความแตกต่างเชิงกลไก

| ประเด็น | Git Alias | Custom Command (`git-xxx`) |
|---|---|---|
| เก็บไว้ที่ไหน | ใน `.gitconfig` (section `[alias]`) | ไฟล์แยกต่างหากใน `PATH` |
| ต้องมี executable bit ไหม | ไม่ต้อง | ต้องมี (`chmod +x`) |
| รองรับหลายบรรทัด/logic ซับซ้อนไหม | ได้ถ้าใช้ `!` เรียก shell แต่จะยาวและอ่านยากในบรรทัดเดียว | ได้เต็มที่ เพราะเป็นไฟล์โปรแกรมจริง เขียนกี่บรรทัดก็ได้ |
| Syntax highlighting ตอนแก้ไข | ไม่มี (เป็นแค่ค่า string ใน config) | มี (เป็นไฟล์ `.sh`/`.py` จริงที่ editor รู้จัก) |
| เขียน automated test ได้ไหม | ยากมาก | ได้ตรง ๆ (เรียก script รันแล้วเช็คผลลัพธ์) |
| แจกจ่ายให้ทีม | ต้องแชร์ block `.gitconfig` หรือใช้ `git config --global alias.xxx` | แจกไฟล์ script (ดู Step 646) |
| ความเร็วในการเขียน/แก้ครั้งแรก | เร็วมาก (บรรทัดเดียวจบ) | ต้องสร้างไฟล์ ตั้ง permission ก่อน |
| รองรับภาษาโปรแกรมอื่นนอกจาก shell | ไม่ได้โดยตรง (ต้องเรียกผ่าน `!`) | ได้ทุกภาษา (Python, Ruby, Go, ฯลฯ) |
| ปรากฏใน `git help -a` เป็น external command | ไม่ปรากฏ | ปรากฏ |

### ตัวอย่างเทียบให้เห็นภาพ

**งานง่าย ๆ ที่เหมาะกับ Alias** (แค่ย่อคำสั่งหรือรวม flag ที่ใช้บ่อย):

```ini
[alias]
    st = status -sb
    lg = log --oneline --graph --all
    unstage = restore --staged
```

**งานที่เหมาะกับ Custom Command** (มี logic, การตัดสินใจแบบมีเงื่อนไข, ต้องอ่าน/parse ข้อมูล):

- `git undo` — ต้องเช็คว่า commit ถูก push แล้วหรือยัง ต้องถามยืนยันแบบมีเงื่อนไข
- `git stats` — ต้อง parse ข้อมูลจำนวนมากแล้วจัดกลุ่มนับ
- `git cleanup-branches` — ต้องกรอง branch ตาม pattern หลายชั้น มี interactive confirmation

### กฎง่าย ๆ ที่ใช้ตัดสินใจ

ถามตัวเองด้วยคำถามเหล่านี้ ถ้า **ตอบ "ใช่" ข้อใดข้อหนึ่ง** ให้เลือกเขียนเป็น Custom Command แทน Alias:

1. Logic มี `if/else` มากกว่า 1-2 เงื่อนไข หรือมี loop หรือไม่?
2. ต้อง parse ผลลัพธ์ของคำสั่ง Git อื่นแล้วนำมาประมวลผลต่อหรือไม่ (ไม่ใช่แค่ต่อ pipe ตรง ๆ)?
3. ต้องการ `--help` ที่อธิบาย option อย่างเป็นทางการหรือไม่?
4. ต้องการเขียน automated test ให้กับ logic นี้หรือไม่?
5. บรรทัดของ alias ยาวจนอ่านไม่รู้เรื่องแล้วหรือยัง (ส่วนตัวมักเกิน ~3-4 บรรทัดของ shell command)?

ในทางกลับกัน ถ้าสิ่งที่ต้องการแค่ **"ย่อคำสั่งที่มีอยู่แล้วให้พิมพ์สั้นลง"** หรือ **"รวม flag ที่ใช้บ่อยเข้าด้วยกัน"** โดยไม่มีการตัดสินใจอะไรซับซ้อน Alias ยังคงเป็นตัวเลือกที่เร็วและง่ายกว่าเสมอ — ทั้งสองแนวทางไม่ได้แข่งกัน แต่**เสริมกัน**: หลายทีมใช้ Alias สำหรับทางลัดสั้น ๆ ควบคู่ไปกับ Custom Command สำหรับเครื่องมือที่มี logic จริงจัง

---

## Step 648: ให้ Custom Command อ่านค่าจาก Git Config ของผู้ใช้

Custom command ที่ดีไม่ควร hardcode ค่าคงที่ไว้ในโค้ด (เช่น ชื่อ base branch, จำนวน commit สูงสุดที่จะแสดง) เพราะแต่ละ repository หรือแต่ละคนอาจต้องการค่าที่ต่างกัน วิธีที่ถูกต้องคือให้ script **อ่านค่าจาก Git config เหมือนกับที่ Git เองทำ**

### กลไก `git config --get`

Git config รองรับ section ที่ไม่ได้ถูก reserve ไว้สำหรับ Git เองด้วย — เราสามารถกำหนด section ที่ตั้งชื่อเองได้ตามใจ (เช่น `[cleanup-branches]`, `[undo]`, หรือ namespace ของทีม) แล้วให้ custom command อ่านค่าจากตรงนั้น:

```ini
# ตัวอย่างใน ~/.gitconfig หรือ .git/config ของ repo ใดก็ได้
[cleanup-branches]
    base = develop
    protected = main,develop,staging

[undo]
    confirmpushed = true
```

จาก script ก็แค่เรียก `git config --get <key>`:

```bash
# ใน Bash — มี fallback เป็นค่า default ถ้าไม่พบ config
BASE_BRANCH="$(git config --get cleanup-branches.base || echo "main")"
```

หรือใน Python:

```python
import subprocess

def get_config(key, default=None):
    result = subprocess.run(
        ["git", "config", "--get", key],
        capture_output=True, text=True,
    )
    if result.returncode == 0:
        return result.stdout.strip()
    return default

base_branch = get_config("cleanup-branches.base", "main")
```

### ทำไมวิธีนี้ถึงสำคัญ

1. **ทำงานสอดคล้องกับ Git ทั้งระบบ** — ผู้ใช้ที่คุ้นเคยกับ `git config` อยู่แล้ว (จาก Part 13) จะปรับแต่งพฤติกรรมของ custom command ได้ทันทีโดยไม่ต้องเรียนรู้วิธีตั้งค่าใหม่
2. **รองรับ scope ตามลำดับความสำคัญเดียวกับ Git config ทั่วไป** — ผู้ใช้สามารถตั้งค่า default ไว้ระดับ `--global` แล้ว override เฉพาะ repository ที่ต้องการต่างออกไปด้วย `--local` ได้ทันที (ทบทวน: `--local` > `--global` > `--system` ตามลำดับความสำคัญที่เรียนไปแล้วในเฟสก่อนหน้า)
3. **ไม่ต้องมีไฟล์ config แยกของตัวเอง** — ลดความซับซ้อนในการดูแลรักษา ไม่ต้องเขียน parser สำหรับไฟล์ config รูปแบบใหม่เอง เพราะ Git จัดการให้หมดแล้ว
4. **ตรวจสอบ/แก้ไขค่าได้ผ่านคำสั่งเดียวกับที่ใช้กับ Git ปกติ** เช่น:

```bash
git config cleanup-branches.base develop
git config --get cleanup-branches.base
git config --unset cleanup-branches.base
```

### ตัวอย่างค่า boolean

ถ้าต้องการเก็บค่าที่เป็น true/false ให้ใช้ `--type=bool` เพื่อให้ Git ช่วย validate และ normalize ค่าให้ (รองรับทั้ง `true`/`false`, `yes`/`no`, `1`/`0`):

```bash
CONFIRM_PUSHED="$(git config --type=bool --get undo.confirmpushed || echo true)"

if [ "$CONFIRM_PUSHED" = "true" ]; then
    # แสดงคำเตือนก่อนย้อน commit ที่ push แล้ว
    ...
fi
```

การออกแบบ custom command ให้ปรับแต่งได้ผ่าน `git config` แบบนี้เป็นแพทเทิร์นเดียวกับที่เครื่องมือดังหลายตัวใช้จริง (เช่น `git-lfs` มี config หลายตัวภายใต้ section `[lfs]`, `git-flow` มี config ภายใต้ `[gitflow]`) — ทำให้เครื่องมือของคุณ "รู้สึกเป็นธรรมชาติ" สำหรับผู้ใช้ Git โดยไม่ต้องเรียนรู้อะไรใหม่

---

## Step 649: เครื่องมือดังในวงการที่สร้างจากแนวคิดนี้

กลไกที่เราเรียนมาทั้งหมดใน Part นี้ไม่ใช่แค่ทฤษฎี — เครื่องมือที่นักพัฒนาทั่วโลกใช้กันทุกวันจำนวนมากถูกสร้างขึ้นจากแนวคิด `git-xxx` นี้เป๊ะ ๆ

### Git LFS (`git-lfs`) — ตัวอย่างที่ตรงที่สุด

**Git LFS** ที่เราเรียนไปแล้วในเฟสนี้ (Part 62) เป็นตัวอย่างที่ชัดเจนที่สุดของกลไก custom command: ตัวโปรแกรมจริง ๆ คือไบนารีที่ชื่อ **`git-lfs`** (เขียนด้วยภาษา Go) เมื่อติดตั้งแล้วมันจะถูกวางไว้ใน `PATH` ทำให้เมื่อคุณพิมพ์:

```bash
git lfs track "*.psd"
git lfs push origin main
```

สิ่งที่เกิดขึ้นเบื้องหลังคือ Git ค้นหา `foo`/`lfs` ไม่เจอใน built-in command แล้วไป exec ไฟล์ `git-lfs track "*.psd"` ต่อ — **เป็นกลไกเดียวกันกับ `git-undo` ที่เราเขียนเองใน Step 645 ทุกประการ** เพียงแต่ `git-lfs` ซับซ้อนกว่ามาก (ต้องคุยกับ smart-http server, จัดการ pointer file, ผูกกับ hook อีกด้วย)

### Git Flow (`git-flow`) — ต่อยอดจาก Part 32

Extension `git-flow` โดย Vincent Driessen ที่เราเรียนไปแล้วใน **Part 32** ก็สร้างขึ้นด้วยกลไกเดียวกันนี้ ตัว installer จะติดตั้งชุด script ที่ชื่อ `git-flow`, `git-flow-feature`, `git-flow-release`, `git-flow-hotfix` ไว้ใน `PATH` ทำให้คำสั่งอย่าง:

```bash
git flow feature start login-page
git flow release start 1.2.0
```

ทำงานผ่านการ dispatch แบบเดียวกัน — `git flow` เรียก `git-flow` แล้ว `git-flow` (ซึ่งเขียนด้วย Bash ล้วน ๆ) จะไป parse argument ต่อ (`feature start login-page`) แล้วเรียก sub-script อย่าง `git-flow-feature` ต่อไปอีกทีภายใน เป็นตัวอย่างที่ดีว่า custom command หนึ่งตัวสามารถเรียก custom command อื่นต่อกันเป็นชั้น ๆ ได้เหมือนโปรแกรมทั่วไป

### git-extras — คลังรวม custom command นับสิบตัว

**git-extras** เป็นโปรเจกต์ Open Source ที่รวบรวม custom command ที่มีประโยชน์ไว้เป็นชุดเดียว ติดตั้งครั้งเดียว (ผ่าน Homebrew, APT หรือ npm) ได้คำสั่งเพิ่มมาเป็นสิบ ๆ ตัวทันที เช่น:

- `git summary` — สรุปภาพรวมของ repository (จำนวน commit, ผู้เขียน, อายุของ repo)
- `git effort` — วัดว่าไฟล์ไหนถูกแก้ไขบ่อยที่สุด
- `git changelog` — สร้าง CHANGELOG.md จากประวัติ commit อัตโนมัติ
- `git obsolete` — หา branch ที่ไม่ได้ merge และไม่มีการเปลี่ยนแปลงมานาน

ทุกคำสั่งเหล่านี้คือไฟล์ `git-summary`, `git-effort`, `git-changelog`, `git-obsolete` ที่ถูกวางไว้ใน `PATH` ตอนติดตั้ง — เป็นตัวอย่างที่ดีมากว่าแนวคิดเดียวกับที่เราเรียนใน Part นี้ สามารถขยายเป็นชุดเครื่องมือระดับ production ที่คนทั้งวงการใช้ได้จริง

### hub และ gh — กรณีที่ต่างออกไปเล็กน้อย (เพื่อความแม่นยำ)

**hub** (เครื่องมือดั้งเดิมของ GitHub ก่อนจะมี `gh`) มักถูกเข้าใจผิดว่าใช้กลไก `git-xxx` เดียวกัน แต่จริง ๆ แล้ว **hub ทำงานต่างออกไป**: มันเป็นโปรแกรมแยกชื่อ `hub` ที่ผู้ใช้ต้องตั้ง shell alias ให้ `git` เรียก `hub` แทน (`alias git=hub`) จากนั้น `hub` จะทำหน้าที่ดักจับคำสั่งที่มันรู้จักเป็นพิเศษ (เช่น `git pull-request`) แล้ว **ส่งคำสั่งที่เหลือทั้งหมดต่อไปให้ Git ตัวจริงทำงานตามปกติ** — เป็นการ "ห่อ (wrap)" ตัว `git` เอง ไม่ใช่การขยายผ่านกลไก dispatch ของ Git แบบ `git-xxx`

ส่วน **`gh` (GitHub CLI)** ซึ่งเป็นเครื่องมือรุ่นใหม่ที่มาแทน `hub` เป็น **โปรแกรมอิสระที่แยกออกจาก Git โดยสิ้นเชิง** เรียกใช้งานด้วยคำสั่ง `gh` ตรง ๆ (เช่น `gh pr create`, `gh issue list`) ไม่ได้ถูกเรียกผ่าน `git gh` และไม่ได้ใช้กลไก dispatch ของ Git เลย — แม้ `gh` เองจะมีระบบ extension ของตัวเอง (`gh extension install`) แต่ก็เป็นระบบ plugin ของ `gh` เอง คนละกลไกกับสิ่งที่เราเรียนใน Part นี้

การแยกแยะให้ถูกต้องแบบนี้สำคัญ เพราะช่วยให้เข้าใจว่า **"การต่อยอด Git" ทำได้หลายวิธี** — บาง tool ใช้กลไก `git-xxx` โดยตรง (LFS, git-flow, git-extras) บาง tool เลือกห่อ (wrap) ตัว Git เอง (hub) และบาง tool เลือกเป็นโปรแกรมอิสระที่แค่ทำงานร่วมกับ Git repository (gh) — ทั้งสามแนวทางมีที่ใช้งานต่างกัน แต่สิ่งที่เราเรียนไปทั้ง Part นี้คือแนวทางแรกซึ่งเป็นแนวทางที่ **Git ออกแบบมาให้รองรับโดยตรงที่สุด**

### สรุปตารางเปรียบเทียบ

| เครื่องมือ | ใช้กลไก `git-xxx` dispatch หรือไม่ | ลักษณะการเรียกใช้ |
|---|---|---|
| Git LFS | ใช่ | `git lfs <คำสั่ง>` |
| git-flow | ใช่ | `git flow <คำสั่ง>` |
| git-extras | ใช่ (หลายสิบคำสั่ง) | `git summary`, `git effort`, ฯลฯ |
| hub | ไม่ใช่ (wrap ตัว `git` เองผ่าน shell alias) | `git <คำสั่งปกติ หรือคำสั่งพิเศษของ hub>` |
| gh (GitHub CLI) | ไม่ใช่ (โปรแกรมอิสระ) | `gh <คำสั่ง>` |

---

## Step 650: แบบฝึกหัด — สร้าง Custom Command ของตัวเอง และสรุปภาพรวมเฟส 6 ทั้งหมด

### แบบฝึกหัด: สร้าง Custom Git Command อย่างน้อย 2 ตัว

ให้คุณเขียน custom command ของตัวเองอย่างน้อย **2 ตัว** ที่ใช้งานได้จริง โดยทำตามขั้นตอนต่อไปนี้ครบทุกข้อ:

**ข้อกำหนดของแบบฝึกหัด:**

1. เลือกหัวข้อจากตัวเลือกด้านล่าง หรือคิดปัญหาที่คุณเจอเองในงานประจำวันแล้วแก้ด้วย custom command
2. ต้องมีการตรวจสอบ `git rev-parse --is-inside-work-tree` ก่อนเสมอ
3. ต้องมี error handling ที่สื่อสารชัดเจนเมื่อผู้ใช้ทำผิดเงื่อนไข (เช่น argument ไม่ครบ, ไม่มี upstream)
4. อย่างน้อย 1 ตัวต้องอ่านค่าจาก `git config` เพื่อให้ผู้ใช้ปรับแต่งพฤติกรรมได้ (ตาม Step 648)
5. ทดสอบให้มั่นใจว่าเรียกจาก sub-directory ลึก ๆ ของ repository แล้วยังทำงานถูกต้อง (ตาม Step 644)

**ไอเดียตัวอย่างให้เลือก (หรือคิดเอง):**

| ชื่อคำสั่ง | หน้าที่ |
|---|---|
| `git who` | แสดงว่าใครแก้ไขไฟล์ที่ระบุล่าสุด พร้อมวันที่และข้อความ commit (ใช้ `git log -1 -- <file>`) |
| `git recent` | แสดงรายชื่อ branch ที่เพิ่ง checkout ล่าสุด (parse จาก `git reflog`) |
| `git open` | เปิด URL ของ remote repository ปัจจุบันในเว็บเบราว์เซอร์ (parse จาก `git remote get-url origin` แล้วแปลง SSH URL เป็น HTTPS) |
| `git amend-no-edit` | ทางลัดสำหรับ `git commit --amend --no-edit` แต่เช็คก่อนว่า commit ล่าสุดยังไม่ถูก push |
| `git branch-age` | แสดงอายุของทุก local branch เรียงจากเก่าสุดไปใหม่สุด (ใช้ `for-each-ref` กับ `--sort=committerdate`) |

**Checklist ตรวจสอบตัวเองก่อนถือว่าทำแบบฝึกหัดเสร็จ:**

- [ ] สร้าง custom command อย่างน้อย 2 ตัว ที่ตั้งชื่อไฟล์ `git-<ชื่อ>` และ `chmod +x` แล้ว
- [ ] ทดสอบเรียก `git <ชื่อ>` (ไม่ใช่เรียกไฟล์ตรง ๆ) สำเร็จจาก root ของ repository
- [ ] ทดสอบเรียกจาก sub-directory ลึก ๆ แล้วยังทำงานถูกต้อง
- [ ] มีการเช็ค `git rev-parse --is-inside-work-tree` และแสดง error ที่อ่านเข้าใจง่ายเมื่อไม่ได้อยู่ใน repo
- [ ] อย่างน้อย 1 ตัวอ่านค่าจาก `git config` ได้ พร้อมมี default fallback ที่สมเหตุสมผล
- [ ] รัน `git help -a` แล้วเห็นชื่อคำสั่งของคุณโผล่ในหมวด external commands
- [ ] ลองแจกจ่ายให้เพื่อนร่วมทีม (หรือจำลองด้วยการ clone dotfiles repo ไปเครื่องอื่น) แล้วใช้งานได้จริงตาม Step 646

---

### สรุปภาพรวมเฟส 6 ทั้งหมด (Part 56–65): Git ขั้นสูงระดับ Internals, Hooks, LFS, Performance

เฟส 6 ของหลักสูตรนี้ (Step 551–650) พาเราลงลึกไปในกลไกภายในของ Git ที่ผู้ใช้งานทั่วไประดับ intermediate มักไม่เคยแตะ แต่เป็นความรู้ที่แยกโปรแกรมเมอร์ระดับ "ใช้ Git เป็น" ออกจากระดับ "เข้าใจ Git จริง ๆ" ตารางด้านล่างคือ cheat sheet สรุปทั้ง 10 Part ของเฟสนี้ไว้ในที่เดียว:

| Part | หัวข้อหลัก | Step | แนวคิด/คำสั่งสำคัญที่ต้องจำ |
|---|---|---|---|
| 56 | Git Object Model | 551–560 | Git เก็บข้อมูลเป็น 4 ชนิด object: **blob** (เนื้อหาไฟล์), **tree** (โครงสร้างโฟลเดอร์), **commit** (snapshot + metadata), **tag** (annotated tag object) ทุก object ถูกระบุด้วย SHA hash ของเนื้อหาตัวเอง (content-addressable storage) |
| 57 | Git Plumbing Commands | 561–570 | แยก **porcelain** (คำสั่งระดับสูงที่ใช้งานประจำวัน เช่น `commit`, `merge`) ออกจาก **plumbing** (คำสั่งระดับต่ำที่ porcelain เรียกใช้ภายใน เช่น `git hash-object`, `git cat-file`, `git rev-parse`, `git update-ref`) ซึ่งเป็นฐานสำคัญของการเขียน custom command ใน Part นี้ |
| 58 | Packfiles | 571–580 | Git บีบอัดและรวม object จำนวนมากเป็นไฟล์ `.pack` เดียวผ่าน `git gc`/`git repack` ใช้ **delta compression** เทียบ object ที่คล้ายกันเพื่อลดขนาด ทำให้ repository ขนาดใหญ่ยัง clone/fetch ได้เร็ว |
| 59 | Git Hooks (Client-side) | 581–590 | Script ในโฟลเดอร์ `.git/hooks/` ที่ Git เรียกอัตโนมัติตามเหตุการณ์บนเครื่อง เช่น `pre-commit` (ตรวจก่อน commit), `commit-msg` (ตรวจข้อความ commit), `pre-push` (ตรวจก่อน push) ใช้บังคับมาตรฐานโค้ดก่อนเข้า repository |
| 60 | Git Hooks (Server-side) | 591–600 | Hook ที่ทำงานบนฝั่ง server เช่น `pre-receive`, `update`, `post-receive` ใช้บังคับกฎระดับองค์กร (เช่น ห้าม force-push เข้า `main`) ที่ client-side hook หลีกเลี่ยงได้แต่ server-side hook หลีกเลี่ยงไม่ได้ |
| 61 | Git Attributes | 601–610 | ไฟล์ `.gitattributes` กำหนดพฤติกรรมพิเศษต่อไฟล์แต่ละประเภท เช่น การแปลง line ending (`text=auto`), การทำ `diff`/`merge` แบบกำหนดเอง, การทำเครื่องหมายไฟล์ให้ export-ignore, และเป็นจุดเชื่อมกับ Git LFS |
| 62 | Git LFS (Large File Storage) | 611–620 | ระบบจัดการไฟล์ขนาดใหญ่ (binary, asset) โดยเก็บ **pointer file** เล็ก ๆ ไว้ใน Git object ปกติ ส่วนเนื้อหาไฟล์จริงเก็บแยกไว้บน LFS server ทำงานผ่านโปรแกรม `git-lfs` ที่ผูกกับ `.gitattributes` และ smudge/clean filter |
| 63 | Git Performance | 621–630 | เทคนิคจัดการ repository ขนาดใหญ่: `git gc`, `shallow clone` (`--depth`), `partial clone`, `sparse-checkout`, `commit-graph`, `git maintenance` เพื่อให้ clone/fetch/status เร็วขึ้นในระดับ repository ที่มีประวัติหลายแสน commit |
| 64 | Git Worktree | 631–640 | `git worktree add` ทำให้เช็คเอาต์หลาย branch พร้อมกันได้ในหลายโฟลเดอร์ที่แชร์ `.git` เดียวกัน แก้ปัญหาการต้อง stash/switch branch บ่อย ๆ เมื่อต้องทำงานหลายอย่างพร้อมกัน |
| 65 | Custom Git Commands (Part นี้) | 641–650 | ไฟล์ `git-<ชื่อ>` ที่ executable และอยู่ใน `PATH` กลายเป็น `git <ชื่อ>` อัตโนมัติ ใช้เขียนเครื่องมือเสริมด้วย Bash/Python ได้ตามใจ พร้อมอ่าน context ของ repo ผ่าน `git rev-parse` และอ่านค่าปรับแต่งผ่าน `git config` |

### เส้นเรื่องของเฟส 6 ในภาพใหญ่

ถ้ามองภาพรวมทั้งเฟส จะเห็นเส้นเรื่องที่ต่อเนื่องกันอย่างมีเหตุผล:

1. **Part 56–57 (Object Model + Plumbing)** สอนให้เข้าใจว่า Git "คิด" อย่างไรในระดับข้อมูลดิบที่สุด — ทุกอย่างคือ object ที่อ้างอิงกันด้วย hash และมีคำสั่ง plumbing ระดับต่ำที่ porcelain command ทุกตัวใช้เป็นฐาน
2. **Part 58 (Packfiles)** ต่อยอดจาก Object Model อธิบายว่า object เหล่านั้นถูกจัดเก็บอย่างมีประสิทธิภาพบน disk จริง ๆ อย่างไร
3. **Part 59–60 (Hooks)** แสดงให้เห็นว่า Git เปิดจุดให้ "แทรกโค้ดของเราเอง" เข้าไปในทุกขั้นตอนของ workflow ได้ ทั้งฝั่ง client และฝั่ง server
4. **Part 61–62 (Attributes + LFS)** ขยายความสามารถของ Git ให้จัดการไฟล์ประเภทพิเศษ (binary, asset ขนาดใหญ่) ที่การเก็บแบบ snapshot ปกติไม่เหมาะ
5. **Part 63 (Performance)** สอนวิธีทำให้ทุกกลไกที่เรียนมาทั้งหมดยังคงทำงานเร็วแม้ repository จะโตขึ้นระดับองค์กรขนาดใหญ่
6. **Part 64 (Worktree)** แก้ปัญหาการทำงานหลาย branch พร้อมกันโดยไม่ต้องเสียเวลา switch/stash ซ้ำ ๆ
7. **Part 65 (Custom Commands)** ปิดเฟสด้วยการสอนให้ **นำความรู้ทั้งหมดของเฟสนี้มาประกอบเป็นเครื่องมือของตัวเอง** — เพราะ custom command ที่ดีมักต้องใช้ plumbing command (Part 57), เข้าใจ object model (Part 56), และบางครั้งต้องผูกกับ hook (Part 59-60) ร่วมด้วย

### Checklist ทบทวนภาพรวมเฟส 6 ทั้งหมด (Part 56–65)

ก่อนไปเฟส 7 (CI/CD) ให้ตรวจสอบว่าคุณ:

- [ ] อธิบายได้ว่า Git object ทั้ง 4 ชนิด (blob, tree, commit, tag) แต่ละตัวเก็บอะไร และเชื่อมโยงกันอย่างไร (Part 56)
- [ ] แยกความแตกต่างระหว่าง porcelain command กับ plumbing command ได้ และเคยใช้ plumbing command อย่าง `git cat-file`, `git rev-parse`, `git hash-object` มาแล้ว (Part 57)
- [ ] เข้าใจว่า packfile คืออะไร ทำไม Git repository ถึงไม่โตแบบเส้นตรงตามจำนวน commit เสมอไป (Part 58)
- [ ] เขียน client-side hook อย่างน้อย 1 ตัว (เช่น `pre-commit`) และเข้าใจว่า hook ฝั่ง client ผู้ใช้สามารถข้ามได้ (Part 59)
- [ ] เข้าใจความแตกต่างของ hook ฝั่ง server และรู้ว่าทำไมกฎสำคัญระดับองค์กรต้องบังคับที่ server ไม่ใช่ client (Part 60)
- [ ] ตั้งค่า `.gitattributes` เพื่อจัดการ line ending หรือ diff แบบกำหนดเองได้ (Part 61)
- [ ] ติดตั้งและใช้งาน Git LFS กับไฟล์ binary ขนาดใหญ่ได้จริง เข้าใจกลไก pointer file (Part 62)
- [ ] รู้จักเทคนิคอย่างน้อย 3 อย่างสำหรับจัดการ repository ขนาดใหญ่ให้เร็วขึ้น (Part 63)
- [ ] ใช้ `git worktree` เพื่อทำงานหลาย branch พร้อมกันได้โดยไม่ต้อง stash (Part 64)
- [ ] เขียนและแจกจ่าย custom Git command ของตัวเองได้อย่างน้อย 2 ตัว พร้อมเข้าใจว่าเมื่อไหร่ควรใช้ Alias แทน (Part 65)
- [ ] อธิบายภาพรวมทั้งเฟส 6 ให้เพื่อนร่วมทีมที่ไม่เคยเรียนฟังได้ภายใน 5 นาที โดยไม่ต้องเปิดเอกสาร

ถ้าติ๊กครบทุกข้อแล้ว แสดงว่าคุณมีความเข้าใจ Git ในระดับที่ลึกกว่าผู้ใช้งานทั่วไปส่วนใหญ่มาก — ระดับความรู้นี้เพียงพอสำหรับการเป็น **Git-savvy engineer** ที่ทีมพึ่งพาได้เวลาเกิดปัญหาซับซ้อน เช่น repository เสียหาย, ประวัติปนเปื้อน, หรือ workflow ของทีมต้องการเครื่องมือเฉพาะทาง

จากนี้ไป หลักสูตรจะเปลี่ยนทิศทางจาก "Git ขั้นสูงเชิงกลไก" ไปสู่ **"การนำ Git ไปต่อยอดเป็นระบบอัตโนมัติระดับทีม"** เริ่มต้นด้วยเฟส 7 ว่าด้วยเรื่อง CI/CD เต็มรูปแบบ

---

## สรุป Part 65

ใน Part นี้เราได้เรียนรู้ว่า:

1. Git ถูกออกแบบให้ต่อยอดได้ตั้งแต่ต้น ผ่านกลไกง่าย ๆ คือการค้นหาไฟล์ `git-<ชื่อ>` ที่ executable ได้ใน `PATH` แล้ว dispatch คำสั่งไปให้โดยอัตโนมัติ
2. เขียน custom command ได้ทั้งด้วย Bash (เหมาะกับงานสั้น ๆ ที่ต่อคำสั่งเป็นสาย) และ Python (เหมาะกับ logic ซับซ้อนที่ต้อง parse/จัดกลุ่มข้อมูล)
3. การเข้าถึง context ของ repository ที่ถูกต้องต้องอาศัยคำสั่ง plumbing อย่าง `git rev-parse --show-toplevel`, `--is-inside-work-tree`, `--abbrev-ref HEAD` เพราะ Git ไม่ได้เปลี่ยน working directory ให้อัตโนมัติ
4. สร้างเครื่องมือที่ใช้งานจริงได้ 3 ตัว: `git undo` (ปลอดภัยด้วย `reset --soft`), `git sync` (fetch + rebase อัตโนมัติ), `git cleanup-branches` (ลบ branch ที่ merge แล้วอย่างปลอดภัย)
5. แจกจ่ายเครื่องมือให้ทีมได้ผ่าน dotfiles repo, installer script, หรือ package manager ขึ้นอยู่กับขนาดและความจริงจังของทีม
6. เลือกระหว่าง Alias (Part 13) กับ Custom Command ตามความซับซ้อนของ logic ที่ต้องการ — ทั้งสองไม่ได้แข่งกันแต่เสริมกัน
7. Custom command ที่ดีควรอ่านค่าปรับแต่งจาก `git config` แทนการ hardcode เพื่อให้สอดคล้องกับวิธีที่ผู้ใช้ Git คุ้นเคยอยู่แล้ว
8. เครื่องมือดังในวงการอย่าง Git LFS และ git-flow สร้างจากกลไกเดียวกันนี้เป๊ะ ๆ ในขณะที่ hub และ gh ใช้แนวทางต่างออกไป (wrap ตัว Git หรือเป็นโปรแกรมอิสระ)
9. Part นี้เป็น Part ปิดท้ายเฟส 6 ทั้งหมด (Part 56–65) ที่พาเราลงลึกตั้งแต่ Object Model, Plumbing, Packfiles, Hooks ทั้งสองฝั่ง, Attributes, LFS, Performance, Worktree ไปจนถึงการสร้างเครื่องมือของตัวเอง

**ต่อไป:** [Part 66: CI/CD คืออะไร ทำไมสำคัญกับทีมพัฒนาซอฟต์แวร์](./part-066-cicd-คืออะไร.md)
