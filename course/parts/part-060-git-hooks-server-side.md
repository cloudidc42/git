# Part 60: Git Hooks: Server-side Hooks และ Automation

> **Step ในหลักสูตรนี้:** Step 591–600
> **เฟส:** 6 — Git ขั้นสูงระดับ Internals, Hooks, LFS, Performance
> **เป้าหมายของ Part นี้:** เข้าใจว่า Server-side Hooks คืออะไร ทำงานต่างจาก Client-side Hooks (ที่เรียนไปใน Part 59) อย่างไร รู้จัก hook ทั้ง 4 ตัวที่ทำงานฝั่งเซิร์ฟเวอร์ (`pre-receive`, `update`, `post-receive`, `post-update`) เขียน hook จริงเพื่อบังคับกฎของทีมก่อนรับ push เข้าใจว่าทำไม GitHub/GitLab แบบ SaaS ถึงไม่เปิดให้แตะ hook เหล่านี้โดยตรง และรู้จัก Webhook ในฐานะเครื่องมือทดแทนสำหรับ automation ที่ทำงานนอกกระบวนการ push

---

## สารบัญของ Part นี้

- Step 591: Server-side Hooks คืออะไร ต่างจาก Client-side Hooks อย่างไร
- Step 592: `pre-receive` hook — บล็อกการ push ทั้งชุดก่อนรับเข้า repo
- Step 593: `update` hook — ตรวจสอบทีละ ref ที่ถูก push เข้ามา
- Step 594: `post-receive` hook — trigger automation หลัง push สำเร็จ
- Step 595: ตัวอย่างจริง — `pre-receive` บังคับ Commit Message Convention และ Branch Naming
- Step 596: ทำไม GitHub/GitLab (SaaS) ไม่เปิดให้แตะ Server-side Hook และใช้อะไรแทน
- Step 597: Webhooks คืออะไร ต่างจาก Server-side Hook อย่างไร
- Step 598: ตั้งค่า Webhook ส่ง Notification ไปยัง Slack
- Step 599: Server-side Hooks กับ Bare Repository บน Self-hosted Git Server
- Step 600: แบบฝึกหัด — เขียน `pre-receive` hook บน Bare Repository จำลอง

---

## Step 591: Server-side Hooks คืออะไร ต่างจาก Client-side Hooks อย่างไร

ใน **Part 59** เราเรียนเรื่อง **Client-side Hooks** ไปแล้ว — hook อย่าง `pre-commit`, `commit-msg`, `pre-push`, `post-checkout` ที่ทำงานอยู่ในโฟลเดอร์ `.git/hooks/` ของ **เครื่อง developer แต่ละคน** ทุกครั้งที่ทำ `git commit`, `git push` หรือคำสั่ง Git อื่น ๆ บนเครื่องตัวเอง

Part นี้เราจะมาเรียนอีกฝั่งหนึ่งของเหรียญ นั่นคือ **Server-side Hooks** — hook ที่ทำงานอยู่บน**เครื่องเซิร์ฟเวอร์**ที่โฮสต์ repository ปลายทาง (โดยทั่วไปคือ **bare repository**) และรันขึ้นมาทุกครั้งที่มีใครสักคน `git push` เข้ามาหาเซิร์ฟเวอร์นั้น

### ความแตกต่างสำคัญที่สุด: "ใครเป็นเจ้าของเครื่องที่ hook รันอยู่"

นี่คือหัวใจของเรื่องทั้งหมดใน Part นี้ ลองเทียบให้เห็นภาพชัด ๆ:

```
Client-side Hook                          Server-side Hook
─────────────────                         ─────────────────
รันบน: เครื่อง Developer เอง               รันบน: เครื่อง Server (bare repo)
ควบคุมโดย: Developer เอง                   ควบคุมโดย: ผู้ดูแล Server เท่านั้น
ถูก push ไปพร้อม repo ไหม: ไม่ (อยู่นอก      ถูก push ไปพร้อม repo ไหม: ไม่จำเป็น
  การ track ปกติของ .git/hooks/)             (ติดตั้งตายตัวอยู่บน server แล้ว)
Developer แก้ไข/ลบ/ปิดได้ไหม: ได้เสมอ         Developer ที่ push เข้ามาแก้ไขได้ไหม: ไม่ได้
  (เป็นเจ้าของเครื่องตัวเอง 100%)              (ไม่มีสิทธิ์เข้าถึง filesystem ของ server)
บังคับใช้ได้จริงหรือไม่: ไม่จริง               บังคับใช้ได้จริงหรือไม่: จริง 100%
  (ข้าม hook ได้ด้วย --no-verify,               (ไม่มีทางข้ามได้ ต่อให้ตั้งใจแค่ไหนก็ตาม
  ลบไฟล์ hook, หรือย้าย repo ไปเครื่องอื่น)      เพราะ hook รันบนเครื่องที่ตัวเองไม่มีสิทธิ์)
```

### ทำไม Client-side Hooks ถึง "บังคับใช้จริงไม่ได้"

ย้อนกลับไปทบทวนสั้น ๆ ว่าทำไม hook ฝั่ง client ถึงเป็นแค่ "เครื่องมือช่วยเตือนตัวเอง" ไม่ใช่ "กฎที่บังคับใช้ได้จริง":

1. **โฟลเดอร์ `.git/hooks/` ไม่ได้ถูก track โดย Git เอง** — เวลาคุณ `git clone` repository ไปเครื่องใหม่ ไฟล์ hook ที่คนอื่นเคยตั้งไว้จะ**ไม่ติดไปด้วย** เพราะ `.git/` ทั้งโฟลเดอร์ไม่ใช่ส่วนหนึ่งของ working tree ที่ Git commit เก็บไว้
2. **Developer เป็นเจ้าของเครื่องตัวเอง** — ต่อให้มีคนแจก hook script มาให้ติดตั้ง developer ก็สามารถแก้ไข ลบ หรือไม่ยอมติดตั้งมันเลยก็ได้ ไม่มีใครไปบังคับได้
3. **มีคำสั่งข้าม hook ให้ใช้ตรง ๆ** เช่น `git commit --no-verify` หรือ `git push --no-verify` ซึ่งจะข้าม `pre-commit`, `commit-msg`, `pre-push` ไปเลยทันที

ด้วยเหตุผลทั้งหมดนี้ Client-side Hooks จึงเหมาะกับการใช้เป็น **"ตัวช่วยส่วนตัว"** ของ developer แต่ละคน (เช่น รัน linter อัตโนมัติก่อน commit เพื่อความสบายของตัวเอง) แต่ **ไม่เหมาะกับการใช้บังคับนโยบายของทีมหรือองค์กร**

### ทำไม Server-side Hooks ถึง "บังคับใช้จริงได้"

Server-side Hooks พลิกสถานการณ์ตรงข้ามกันโดยสิ้นเชิง:

1. **hook อยู่ในโฟลเดอร์ `hooks/` ของ bare repository บนเซิร์ฟเวอร์** ซึ่งเป็นเครื่องที่ **ผู้ push ไม่มีสิทธิ์เข้าถึง filesystem โดยตรง** (เข้าถึงได้แค่ผ่านโปรโตคอล Git เช่น SSH หรือ HTTP(S) เท่านั้น)
2. **ผู้ดูแลระบบ (Git server administrator) เท่านั้น** ที่มีสิทธิ์เขียน/แก้ไข/ลบไฟล์ hook เหล่านี้ได้
3. **ไม่มีคำสั่งฝั่ง client ใดที่ข้าม server-side hook ได้เลย** — `--no-verify` ข้ามได้แค่ hook ฝั่งตัวเอง (`pre-push` บนเครื่อง client) แต่ไม่มีผลอะไรกับ hook ที่รอตรวจอยู่บนเซิร์ฟเวอร์ปลายทาง เพราะมันเป็นคนละกระบวนการ คนละเครื่องกันโดยสิ้นเชิง

พูดให้ชัดที่สุด: **Server-side Hooks คือจุดเดียวในระบบ Git ทั้งหมดที่สามารถบังคับกฎขององค์กรได้จริงแบบไม่มีทางหลีกเลี่ยง** เพราะมันควบคุมโดยฝ่ายที่ "รับ" ข้อมูล ไม่ใช่ฝ่ายที่ "ส่ง" ข้อมูล

### hook ฝั่ง server มีทั้งหมด 4 ตัว

Git กำหนด server-side hooks ไว้ 4 ตัว โดยทั้งหมดอยู่ในโฟลเดอร์ `hooks/` ของ repository ที่รับ push (มักจะเป็น bare repository):

| Hook | จังหวะที่ทำงาน | บล็อกการ push ได้ไหม | ขอบเขตการตรวจ |
|---|---|---|---|
| `pre-receive` | ก่อนอัปเดต ref ใด ๆ เลยทั้งหมด | ได้ (บล็อกทั้ง push) | เห็นทุก ref ที่ถูก push มาพร้อมกันในครั้งเดียว |
| `update` | หลัง `pre-receive` ผ่าน แต่ก่อนอัปเดตแต่ละ ref | ได้ (บล็อกเฉพาะ ref นั้น) | เห็นทีละ ref เดียว รันซ้ำหลายรอบถ้า push มาหลาย ref |
| `post-receive` | หลังอัปเดต ref สำเร็จทั้งหมดแล้ว | ไม่ได้ (สายเกินไป) | เห็นทุก ref ที่เพิ่งอัปเดตสำเร็จ |
| `post-update` | หลัง `post-receive` (legacy hook) | ไม่ได้ | ได้แค่ชื่อ ref ที่อัปเดต (ไม่มี oldrev/newrev) |

จะสังเกตว่าลำดับการทำงานจริง ๆ เมื่อมีคน `git push` เข้ามาคือ:

```
git push  ──▶  pre-receive  ──▶  update (รันต่อ ref)  ──▶  [Git อัปเดต ref จริง]  ──▶  post-receive  ──▶  post-update
                  │                    │
                  └─ exit ≠ 0 ─────────┴─ exit ≠ 0
                     บล็อกทั้ง push        บล็อกเฉพาะ ref นั้น (ref อื่นอาจผ่านได้)
```

เราจะเจาะลึกแต่ละตัวใน Step ถัดไป โดยเริ่มจาก `pre-receive` ซึ่งเป็น hook ที่ใช้งานบ่อยที่สุดในการบังคับนโยบายระดับทีม

### แนวคิดสำคัญที่ต้องจำก่อนไป Step 592: Quarantine Area

ก่อนจะลงรายละเอียด มีกลไกภายในหนึ่งอย่างที่ทำให้ server-side hooks ปลอดภัยกว่าที่หลายคนคิด: เมื่อ client `push` ข้อมูลเข้ามา Git **ไม่ได้เขียน object ใหม่เข้าไปใน object database หลักของ repository ทันที** แต่จะเก็บ object ทั้งหมดไว้ใน **quarantine directory** ชั่วคราวก่อน (เข้าถึงผ่าน environment variable `GIT_QUARANTINE_PATH` ภายใน hook) — `pre-receive` และ `update` สามารถตรวจสอบ object เหล่านี้ได้ตามปกติ (ด้วยคำสั่งอย่าง `git cat-file`, `git log`) แต่ถ้า hook ตัวใดตัวหนึ่ง reject การ push, object ทั้งหมดใน quarantine จะถูก**ทิ้งไปเลย ไม่มีร่องรอยหลงเหลือใน repository** ต่อเมื่อทุก hook ผ่านหมดแล้วเท่านั้น Git ถึงจะย้าย object จาก quarantine เข้าไปอยู่ใน object database จริงและอัปเดต ref — นี่คือเหตุผลที่การ reject การ push ด้วย server-side hook ถึง "สะอาด" มาก ไม่ทิ้งขยะหรือ commit หลุดค้างไว้ใน repository เลยแม้แต่นิดเดียว

---

## Step 592: `pre-receive` hook — บล็อกการ push ทั้งชุดก่อนรับเข้า repo

`pre-receive` คือ hook ตัวแรกที่ทำงานเมื่อมีคน push เข้ามา และเป็น hook ที่ **"เข้มงวดที่สุด"** ในบรรดา server-side hooks ทั้งหมด เพราะมันทำงาน**ก่อน**ที่ Git จะแตะ ref ใด ๆ เลยแม้แต่ตัวเดียว

### รูปแบบการรับข้อมูล: ผ่าน stdin ไม่ใช่ argument

`pre-receive` ไม่ได้รับข้อมูลผ่าน command-line arguments เหมือน hook อื่น ๆ บางตัว แต่รับผ่าน **standard input (stdin)** โดยข้อมูลที่ส่งเข้ามาจะมีรูปแบบ **หนึ่งบรรทัดต่อหนึ่ง ref** ที่กำลังจะถูกอัปเดต:

```
<old-value> SP <new-value> SP <ref-name> LF
<old-value> SP <new-value> SP <ref-name> LF
...
```

ตัวอย่างจริงถ้าคน push เข้ามาสอง branch พร้อมกัน (`main` และ `feature/checkout`):

```
a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2 f6e5d4c3b2a1f6e5d4c3b2a1f6e5d4c3b2a1f6e5 refs/heads/main
0000000000000000000000000000000000000000 9988776655443322119988776655443322119988 refs/heads/feature/checkout
```

สังเกตบรรทัดที่สอง — `<old-value>` เป็นค่า **zero hash** (เลข 0 ล้วน 40 ตัว) ซึ่งหมายความว่านี่คือการ**สร้าง branch ใหม่** (ref นี้ยังไม่เคยมีอยู่มาก่อนใน repository) ในทางกลับกัน ถ้า `<new-value>` เป็น zero hash แปลว่ากำลังจะเกิดการ**ลบ branch/tag**

### คุณสมบัติที่สำคัญที่สุดของ `pre-receive`: เห็น "ทั้ง push" พร้อมกันในครั้งเดียว

เพราะ `pre-receive` รันแค่ **ครั้งเดียวต่อการ push หนึ่งครั้ง** (ไม่ว่า push นั้นจะมี ref กี่ตัวก็ตาม) และรับข้อมูลทุก ref มาพร้อมกันทาง stdin นี่ทำให้มันเหมาะมากกับการตรวจสอบเงื่อนไขที่ **ต้องดูภาพรวมของทั้ง push** เช่น:

- "ถ้า push ครั้งนี้แตะ branch `release/*` ต้องมี tag ใหม่ตามมาด้วยเสมอ"
- "ห้าม push repository ที่มีขนาดรวมเกิน 100MB ในครั้งเดียว"
- "จำกัดจำนวน ref สูงสุดที่ push เข้ามาได้ในครั้งเดียว ไม่เกิน 10 ref"

### โครงสร้างพื้นฐานของ `pre-receive`

```bash
#!/bin/sh
# hooks/pre-receive

while read oldrev newrev refname
do
  echo "กำลังตรวจสอบ ref: $refname"
  echo "  จาก: $oldrev"
  echo "  ไป:  $newrev"

  # ตรงนี้ใส่ตรรกะตรวจสอบตามนโยบายของทีม
  # ถ้าไม่ผ่าน ให้ echo ข้อความอธิบาย แล้ว exit 1

done

exit 0
```

### กฎสำคัญเรื่อง Exit Code

- `exit 0` — **ยอมรับการ push ทั้งหมด** ทุก ref ที่ส่งมาจะถูกอัปเดตตามปกติ
- `exit ≠ 0` (เช่น `exit 1`) — **ปฏิเสธการ push ทั้งหมด** ไม่ว่า push นั้นจะมีกี่ ref ก็ตาม **ref ทุกตัวจะไม่ถูกอัปเดตแม้แต่ตัวเดียว** แม้ว่าจะมีบาง ref ที่ "ผ่านเงื่อนไข" ก็ตาม เพราะ `pre-receive` ทำงานแบบ all-or-nothing

นี่คือจุดที่มักทำให้มือใหม่สับสน: ถ้าคุณ push พร้อมกันสอง branch แล้ว branch หนึ่งผิดกฎ **ทั้งสอง branch จะถูกปฏิเสธหมด** ไม่ใช่แค่ branch ที่ผิด — ถ้าต้องการ "ปฏิเสธเฉพาะ ref ที่ผิด แต่ให้ ref อื่นผ่านได้" คุณต้องใช้ hook ชื่อ `update` แทน ซึ่งเราจะเรียนใน Step ถัดไป

### ข้อความ error ที่ผู้ push จะเห็น

ทุกอย่างที่ hook เขียนออกทาง stdout/stderr จะถูกส่งกลับไปแสดงที่ฝั่ง client โดย Git จะเติมคำว่า `remote:` นำหน้าให้อัตโนมัติ ตัวอย่างเช่น ถ้า hook เขียนว่า:

```bash
echo "REJECTED: ห้าม push ไฟล์ที่มีขนาดเกิน 50MB"
exit 1
```

ฝั่ง client ที่ push เข้ามาจะเห็นข้อความแบบนี้:

```
$ git push origin main
Enumerating objects: 5, done.
...
remote: REJECTED: ห้าม push ไฟล์ที่มีขนาดเกิน 50MB
To git.company.local:project.git
 ! [remote rejected] main -> main (pre-receive hook declined)
error: failed to push some refs to 'git.company.local:project.git'
```

นี่คือช่องทางสื่อสารสำคัญมาก — hook ที่ดีควรเขียนข้อความ error ให้ชัดเจนว่า **ผิดกฎอะไร และควรแก้ไขอย่างไร** ไม่ใช่แค่บอกว่า "rejected" เฉย ๆ

### ตัวอย่างการใช้งานจริงของ `pre-receive` ในโลกจริง

| กรณีใช้งาน | ตรวจอะไร |
|---|---|
| ป้องกัน Force Push ทับประวัติ | ตรวจว่า `oldrev` เป็น ancestor ของ `newrev` จริงหรือไม่ (`git merge-base --is-ancestor`) |
| จำกัดขนาดไฟล์ | ใช้ `git diff --stat` หรือ `git cat-file -s` ตรวจขนาด blob ที่เพิ่มเข้ามาใหม่ |
| สแกนหา Secret หลุด | รัน tool อย่าง `trufflehog`/`gitleaks` ตรวจ diff ก่อนรับเข้า |
| บังคับ Signed Commit | ตรวจว่าทุก commit มี GPG signature ที่ verify ผ่าน (`git verify-commit`) |
| จำกัดเวลาที่อนุญาตให้ push | เช่น ห้าม push เข้า `main` นอกเวลาทำการ (Change Freeze) |
| บังคับ Naming Convention | ตรวจชื่อ branch ให้ตรงรูปแบบที่กำหนด (ตัวอย่างเต็มใน Step 595) |

---

## Step 593: `update` hook — ตรวจสอบทีละ ref ที่ถูก push เข้ามา

ถ้า `pre-receive` คือด่านตรวจที่มองภาพรวมทั้งหมดพร้อมกัน `update` hook คือด่านตรวจที่ **ยืนตรวจทีละคนที่ผ่านประตู** — มันทำงานหลังจาก `pre-receive` ผ่านแล้ว และ **รันแยกกันหนึ่งครั้งต่อหนึ่ง ref** ที่กำลังจะถูกอัปเดต

### รูปแบบการรับข้อมูล: ผ่าน Command-line Arguments ไม่ใช่ stdin

ตรงข้ามกับ `pre-receive` ที่รับข้อมูลทาง stdin, `update` hook รับข้อมูลผ่าน **3 arguments** ตรง ๆ:

```bash
#!/bin/sh
# hooks/update
refname="$1"   # เช่น refs/heads/main
oldrev="$2"    # SHA เดิมก่อน push (หรือ zero hash ถ้าเป็น ref ใหม่)
newrev="$3"    # SHA ใหม่หลัง push (หรือ zero hash ถ้าเป็นการลบ ref)

echo "กำลังตรวจ ref: $refname ($oldrev -> $newrev)"

# ตรรกะตรวจสอบเฉพาะ ref นี้

exit 0
```

ถ้าคุณ push มา 3 branch พร้อมกัน Git จะเรียก `update` hook **3 รอบแยกกัน** โดยแต่ละรอบจะได้ argument ของ ref ตัวนั้น ๆ เท่านั้น — มันไม่รู้ว่ามี ref อื่นถูก push มาพร้อมกันด้วยหรือไม่ (นอกจากจะไปเช็คจากที่อื่นเอง)

### ความแตกต่างสำคัญจาก `pre-receive`: บล็อกได้เฉพาะ ref เดียว ไม่กระทบ ref อื่น

นี่คือจุดขายหลักของ `update` hook ถ้า `update` hook คืนค่า exit code ที่ไม่ใช่ 0 สำหรับ ref หนึ่ง **เฉพาะ ref นั้นเท่านั้นที่จะถูกปฏิเสธ** ส่วน ref อื่นที่ผ่านการตรวจแล้วในรอบก่อนหน้า (และ ref ที่ยังไม่ถูกตรวจ) จะยังคงถูกดำเนินการต่อไปตามปกติ (ถ้าผ่าน `update` ของมันเองด้วย)

```
git push (มี 3 ref: main, feature/a, feature/b)
        │
        ▼
   pre-receive ผ่าน (exit 0)
        │
        ├─▶ update refs/heads/main         → exit 1 (reject)   ❌ main ถูกปฏิเสธ
        ├─▶ update refs/heads/feature/a    → exit 0 (accept)   ✅ feature/a ผ่าน
        └─▶ update refs/heads/feature/b    → exit 0 (accept)   ✅ feature/b ผ่าน

ผลลัพธ์สุดท้าย: feature/a และ feature/b ถูกอัปเดตสำเร็จ
               main ถูกปฏิเสธเพียง ref เดียว
```

ผลลัพธ์ที่ client จะเห็นในกรณีนี้คือการ push แบบ "สำเร็จบางส่วน":

```
To git.company.local:project.git
 ! [remote rejected] main -> main (hook declined)
   a1b2c3d..d4e5f6a  feature/a -> feature/a
   b2c3d4e..e5f6a7b  feature/b -> feature/b
error: failed to push some refs to 'git.company.local:project.git'
```

### เมื่อไหร่ควรใช้ `pre-receive` เมื่อไหร่ควรใช้ `update`

| สถานการณ์ | ควรใช้ hook ไหน | เหตุผล |
|---|---|---|
| ต้องดูภาพรวมของทุก ref พร้อมกัน (เช่น เช็คว่ามี tag คู่กับ release branch) | `pre-receive` | เห็นทุก ref ในครั้งเดียวผ่าน stdin |
| ต้องการนโยบายที่ต่างกันไปตามชื่อ/pattern ของแต่ละ branch | `update` | รับ refname มาตรง ๆ เขียน `case` แยกกฎตาม pattern ได้ง่าย |
| ต้องการให้ ref ที่ถูกต้องผ่านได้ แม้ ref อื่นในชุดเดียวกันจะผิด | `update` | บล็อกทีละ ref ไม่กระทบ ref อื่น |
| ต้องการปฏิเสธทั้ง push ทันทีถ้ามีอะไรผิดพลาดแม้แต่นิดเดียว (all-or-nothing) | `pre-receive` | บล็อกทั้งชุดในการตัดสินใจเดียว |
| ต้องเช็คเงื่อนไขที่ "แพง" (เช่น รัน static analysis ทั้งโปรเจกต์) แค่ครั้งเดียวต่อ push | `pre-receive` | รันแค่ 1 ครั้งเท่านั้น ไม่ว่าจะมีกี่ ref |

ในทางปฏิบัติ ทีมจำนวนมากเลือกใช้แค่ `pre-receive` ตัวเดียวแล้วเขียน loop ตรวจทุก ref เอง (ดังตัวอย่างใน Part 45 และ Step 595) เพราะเขียนควบคุม logic ได้ง่ายกว่าในไฟล์เดียว แต่ถ้าต้องการพฤติกรรม "ปฏิเสธเฉพาะ ref ที่ผิด" แบบ built-in โดยไม่ต้องเขียน logic ซับซ้อนเอง `update` คือคำตอบที่ตรงจุดกว่า

### ตัวอย่าง `update` hook แยกกฎตาม branch pattern

```bash
#!/bin/sh
# hooks/update
refname="$1"
oldrev="$2"
newrev="$3"

zero="0000000000000000000000000000000000000000"

case "$refname" in
  refs/heads/main)
    # main ต้องมาจาก merge commit เท่านั้น (มากกว่า 1 parent)
    parent_count=$(git cat-file -p "$newrev" | grep -c '^parent ')
    if [ "$parent_count" -lt 2 ]; then
      echo "REJECTED [$refname]: ห้าม push commit เดี่ยวเข้า main โดยตรง ต้อง merge ผ่าน Pull Request เท่านั้น"
      exit 1
    fi
    ;;
  refs/heads/release/*)
    # release branch ห้ามลบเด็ดขาด
    if [ "$newrev" = "$zero" ]; then
      echo "REJECTED [$refname]: ห้ามลบ release branch"
      exit 1
    fi
    ;;
  refs/tags/*)
    # tag ต้องเป็น annotated tag เท่านั้น ห้ามเป็น lightweight tag
    tag_type=$(git cat-file -t "$newrev")
    if [ "$tag_type" != "tag" ]; then
      echo "REJECTED [$refname]: ต้องใช้ annotated tag (git tag -a) เท่านั้น ห้ามใช้ lightweight tag"
      exit 1
    fi
    ;;
esac

exit 0
```

สังเกตว่าเราสามารถใช้ `case "$refname" in` เขียนกฎที่ต่างกันไปตามประเภทของ ref ได้อย่างเป็นระเบียบมาก — นี่คือจุดแข็งของ `update` hook ที่ `pre-receive` ทำได้เหมือนกันแต่ต้องเขียน loop ครอบเอง

---

## Step 594: `post-receive` hook — trigger automation หลัง push สำเร็จ

`post-receive` คือ hook ที่ทำงาน **หลังจากทุก ref ถูกอัปเดตสำเร็จเรียบร้อยแล้วเท่านั้น** — ถ้า push ถูกปฏิเสธไปแล้วโดย `pre-receive` หรือ `update` (ไม่ว่า ref ใด) `post-receive` จะไม่รับรู้เรื่อง ref นั้นเลย เพราะมันไม่เคยถูกอัปเดตจริง

### หน้าที่: ไม่ใช่ "ยาม" แต่เป็น "ผู้ประกาศข่าว"

จุดสำคัญที่ต้องเข้าใจให้ชัด: **`post-receive` ไม่สามารถปฏิเสธหรือย้อนกลับการ push ได้อีกแล้ว** เพราะ ref ถูกอัปเดตเรียบร้อยไปแล้วก่อนหน้านี้ — ต่อให้ `post-receive` จะ `exit 1` ก็ไม่มีผลอะไรต่อผลลัพธ์ของการ push เลย (client จะเห็น exit code นั้นแค่เป็น warning message เท่านั้น ไม่ใช่การ reject)

หน้าที่ที่แท้จริงของ `post-receive` คือการเป็น **จุดเริ่มต้นของ automation** ทุกอย่างที่ควรเกิดขึ้น "หลังจากมีโค้ดใหม่เข้ามาใน repository แล้วจริง ๆ"

### รูปแบบการรับข้อมูล

เหมือนกับ `pre-receive` ทุกประการ คือรับผ่าน **stdin** ในรูปแบบ `<old-value> <new-value> <ref-name>` หนึ่งบรรทัดต่อหนึ่ง ref ที่เพิ่งอัปเดตสำเร็จ

```bash
#!/bin/sh
# hooks/post-receive

while read oldrev newrev refname
do
  echo "ref $refname ถูกอัปเดตสำเร็จแล้ว: $oldrev -> $newrev"
  # trigger automation ตรงนี้
done
```

### กรณีใช้งานจริงที่พบบ่อยที่สุด

#### 1. Push-to-Deploy — deploy อัตโนมัติทันทีที่มีคน push

รูปแบบคลาสสิกที่สุดของ `post-receive` คือการใช้มัน checkout โค้ดล่าสุดออกไปยังโฟลเดอร์ที่ web server ให้บริการอยู่ทันที โดยอาศัยตัวแปร environment `GIT_WORK_TREE`:

```bash
#!/bin/sh
# hooks/post-receive (บน bare repo ที่ใช้เป็น deploy target)

TARGET="/var/www/production"
GIT_DIR="/srv/git/myapp.git"

while read oldrev newrev refname
do
  branch=$(echo "$refname" | sed 's#refs/heads/##')

  if [ "$branch" = "main" ]; then
    echo "กำลัง deploy branch main ไปยัง production..."
    git --work-tree="$TARGET" --git-dir="$GIT_DIR" checkout -f main
    echo "Deploy สำเร็จ: $newrev"
  fi
done
```

เมื่อไหร่ก็ตามที่มีคน `git push origin main` เข้ามาที่ bare repository ตัวนี้ ไฟล์ล่าสุดจะถูก checkout ออกไปที่ `/var/www/production` โดยอัตโนมัติทันที — นี่คือรูปแบบ deploy ที่เรียบง่ายที่สุดในโลก (ก่อนที่จะมี CI/CD Pipeline อย่างที่เราจะเรียนใน Part 66 เป็นต้นไป) และยังใช้กันจริงในหลายองค์กรขนาดเล็กถึงกลางจนถึงทุกวันนี้

#### 2. ส่ง Notification แจ้งทีม

```bash
#!/bin/sh
while read oldrev newrev refname
do
  branch=$(echo "$refname" | sed 's#refs/heads/##')
  pusher="$(whoami)"
  commit_msg=$(git log -1 --format=%s "$newrev")

  curl -X POST -H 'Content-type: application/json' \
    --data "{\"text\":\"📦 $pusher push เข้า $branch: $commit_msg\"}" \
    "https://hooks.slack.com/services/XXX/YYY/ZZZ"
done
```

#### 3. Trigger CI Build ภายนอก

```bash
#!/bin/sh
while read oldrev newrev refname
do
  curl -X POST "https://ci.company.local/api/trigger-build" \
    -H "Authorization: Bearer $CI_TOKEN" \
    -d "commit=$newrev&ref=$refname"
done
```

#### 4. `git update-server-info` — สืบทอดหน้าที่จาก `post-update`

ในยุคที่ Git ยังรองรับ "dumb HTTP protocol" (การ serve repository ผ่าน HTTP server ธรรมดาที่ไม่มี Git อยู่เบื้องหลังเลย เช่น Apache serve static file) จำเป็นต้องมีไฟล์ metadata (`info/refs`, `objects/info/packs`) ที่อัปเดตให้ตรงกับสถานะปัจจุบันของ repository เสมอ คำสั่ง `git update-server-info` ทำหน้าที่สร้างไฟล์เหล่านี้ ซึ่งในอดีตเป็นหน้าที่หลักของ hook ชื่อ `post-update` (hook เก่าแก่ที่สุดในบรรดา 4 ตัว ได้รับเฉพาะ**ชื่อ ref** เป็น argument ไม่มี oldrev/newrev ให้) ปัจจุบันโปรโตคอล "smart HTTP" ที่ Git ใช้กันเป็นมาตรฐานแล้วไม่จำเป็นต้องพึ่งกลไกนี้อีกต่อไป แต่ `post-update` (และไฟล์ตัวอย่าง `post-update.sample` ที่มากับ Git ทุกตัว) ก็ยังถูกเก็บไว้เพื่อความเข้ากันได้ย้อนหลัง

### ตารางสรุปว่า "หน้าที่ไหนควรอยู่ที่ hook ไหน"

| งาน | ควรอยู่ที่ hook ไหน | เพราะ |
|---|---|---|
| ตรวจสอบและปฏิเสธ push ที่ผิดกฎ | `pre-receive` / `update` | ต้องบล็อกได้ก่อนรับเข้า |
| Deploy โค้ดอัตโนมัติ | `post-receive` | ต้องเกิดขึ้นหลังยืนยันว่า push สำเร็จแน่นอนแล้วเท่านั้น |
| แจ้งเตือนทีมผ่าน Slack/Email | `post-receive` | ไม่ควรเกิดถ้า push ยังไม่สำเร็จจริง |
| Trigger CI/CD Pipeline | `post-receive` | เช่นเดียวกับ deploy — ต้องมีของจริงให้ build ก่อน |
| อัปเดต metadata สำหรับ dumb HTTP | `post-update` (legacy) | เป็นหน้าที่ดั้งเดิมของ hook ตัวนี้โดยเฉพาะ |

---

## Step 595: ตัวอย่างจริง — `pre-receive` บังคับ Commit Message Convention และ Branch Naming

ใน **Part 45 (โปรเจกต์ทีม Capstone)** เราเคยเขียน `pre-receive` hook จริงไปแล้วครั้งหนึ่งเพื่อจำลอง Branch Protection บน bare repository `teamcommerce-central.git` โดย hook ตัวนั้นบังคับ 2 กฎ คือ (1) ชื่อ branch ใหม่ต้องตรง naming convention (`feature/`, `hotfix/`, `release/` หรือ `main`) และ (2) การ push เข้า `main` ต้องเป็น merge commit ที่มี trailer `Reviewed-by:` เท่านั้น

ใน Step นี้เราจะ**ต่อยอด** hook ตัวเดิมนั้นให้แข็งแกร่งขึ้นอีกขั้น โดยเพิ่มกฎที่ 3 เข้าไป: **บังคับ Commit Message Convention แบบ Conventional Commits** (ที่เรียนไปใน **Part 35**) กับทุกคอมมิตที่ถูก push เข้ามา ไม่ใช่แค่ตรวจ commit ล่าสุดเท่านั้น แต่ต้องตรวจ**ทุกคอมมิตใหม่**ในช่วง `oldrev..newrev` เพราะการ push หนึ่งครั้งอาจพ่วงมาหลายคอมมิตพร้อมกัน

### ทบทวน: รูปแบบ Conventional Commits ที่บังคับใน Part 35

```
<type>(<scope>): <description>

ตัวอย่างที่ผ่าน:
feat(checkout): เพิ่มระบบชำระเงินผ่าน QR Code
fix(auth): แก้บั๊ก token หมดอายุก่อนเวลา
docs: อัปเดตคู่มือการติดตั้ง
```

โดย `<type>` ต้องเป็นหนึ่งใน `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`

### hook ฉบับเต็ม: รวม 3 กฎเข้าด้วยกัน

```bash
#!/bin/sh
# hooks/pre-receive
# บังคับ 3 กฎ: (1) branch naming (2) main ต้องเป็น merge + Reviewed-by (3) conventional commits

zero="0000000000000000000000000000000000000000"
allowed_types="feat|fix|docs|style|refactor|perf|test|chore|build|ci"

reject() {
  echo "REJECTED [$1]: $2"
  exit 1
}

while read oldrev newrev refname
do
  branch=$(echo "$refname" | sed 's#refs/heads/##')

  # ข้ามการลบ branch ไม่ต้องตรวจอะไร
  if [ "$newrev" = "$zero" ]; then
    continue
  fi

  # ── กฎที่ 1: Branch Naming Convention (เฉพาะตอนสร้าง branch ใหม่) ──
  if [ "$oldrev" = "$zero" ]; then
    case "$branch" in
      main|feature/*|hotfix/*|release/*) : ;;
      *) reject "$refname" "ชื่อ branch ต้องขึ้นต้นด้วย feature/, hotfix/, release/ เท่านั้น (ได้รับ: $branch)" ;;
    esac
  fi

  # ── กฎที่ 2: main ต้องมาจาก merge commit ที่มี Reviewed-by ──
  if [ "$branch" = "main" ]; then
    parent_count=$(git cat-file -p "$newrev" | grep -c '^parent ')
    if [ "$parent_count" -lt 2 ]; then
      reject "$refname" "ห้าม push commit เดี่ยวเข้า main ต้อง merge ผ่าน Pull Request เท่านั้น"
    fi

    message=$(git log -1 --format=%B "$newrev")
    case "$message" in
      *"Reviewed-by:"*) : ;;
      *) reject "$refname" "merge commit เข้า main ต้องมี trailer 'Reviewed-by:' จาก Code Owner" ;;
    esac
  fi

  # ── กฎที่ 3: ทุกคอมมิตใหม่ต้องตรง Conventional Commits ──
  # หาว่าคอมมิตไหนบ้างที่ "ใหม่" ใน push ครั้งนี้ (มีอยู่ใน newrev แต่ยังไม่มีใน repo)
  if [ "$oldrev" = "$zero" ]; then
    # branch ใหม่: ตรวจทุกคอมมิตที่ยังไม่เคยอยู่ใน ref อื่นของ repo เลย
    commit_range=$(git rev-list "$newrev" --not --all)
  else
    commit_range=$(git rev-list "$oldrev".."$newrev")
  fi

  for commit in $commit_range; do
    subject=$(git log -1 --format=%s "$commit")
    if ! echo "$subject" | grep -Eq "^($allowed_types)(\([a-z0-9_-]+\))?: .+"; then
      reject "$refname" "commit $commit มีข้อความไม่ตรง Conventional Commits: '$subject' (ต้องขึ้นต้นด้วย feat:, fix:, docs: ฯลฯ)"
    fi
  done

done

exit 0
```

### อธิบายจุดที่ซับซ้อนที่สุด: การหา "คอมมิตใหม่" ในช่วง push

จุดที่มือใหม่มักพลาดคือคิดว่า `oldrev..newrev` ใช้ได้ในทุกกรณี แต่จริง ๆ แล้วมันใช้ได้เฉพาะกรณีที่ branch นั้น**มีอยู่แล้ว**ก่อนหน้า (ไม่ใช่ branch ใหม่) เพราะถ้า `oldrev` เป็น zero hash การเขียน `git rev-list zero..newrev` จะพัง (zero hash ไม่ใช่ object ที่มีอยู่จริง)

สำหรับ branch ที่**เพิ่งสร้างใหม่** (`oldrev = zero`) เราต้องใช้ `git rev-list "$newrev" --not --all` แทน ซึ่งแปลว่า "เอาทุกคอมมิตที่ไปถึงได้จาก `$newrev` **ยกเว้น** คอมมิตที่ไปถึงได้จาก ref อื่น ๆ ที่มีอยู่แล้วใน repo" — นี่คือวิธีหา "คอมมิตที่ Git server ไม่เคยเห็นมาก่อนเลย" อย่างถูกต้อง ไม่ว่าจะเป็นการสร้าง branch ใหม่จาก branch เก่า (ซึ่งจะไม่มีคอมมิตใหม่เลยถ้ายังไม่ได้ commit เพิ่ม) หรือสร้าง branch แบบไม่มีความเกี่ยวข้องกับอะไรเลย (orphan branch)

### ทดสอบ hook ด้วยสถานการณ์จริง

```bash
# กรณีที่ 1: commit message ผิดรูปแบบ
$ git commit -m "เพิ่มปุ่มใหม่"
$ git push origin feature/new-button

remote: REJECTED [refs/heads/feature/new-button]: commit a3f9c21 มีข้อความไม่ตรง Conventional Commits: 'เพิ่มปุ่มใหม่' (ต้องขึ้นต้นด้วย feat:, fix:, docs: ฯลฯ)
 ! [remote rejected] feature/new-button -> feature/new-button (pre-receive hook declined)

# กรณีที่ 2: แก้ commit message ใหม่ให้ถูกต้อง
$ git commit --amend -m "feat(ui): เพิ่มปุ่มยืนยันการสั่งซื้อ"
$ git push --force origin feature/new-button

Enumerating objects: 5, done.
...
To git.company.local:project.git
 * [new branch]      feature/new-button -> feature/new-button
```

hook นี้แสดงให้เห็นชัดเจนว่า **นโยบายที่เคยเป็นแค่ "ข้อตกลงในเอกสาร" (เช่นกฎ Conventional Commits ใน Part 35 หรือ Branch Naming ใน Part 35 เช่นกัน) สามารถกลายเป็น "กฎที่บังคับใช้ได้จริงทางเทคนิค 100%"** ได้ทันทีที่ย้ายมันมาไว้ใน server-side hook — ไม่ต้องพึ่งความมีวินัยของแต่ละคนอีกต่อไป

---

## Step 596: ทำไม GitHub/GitLab (SaaS) ไม่เปิดให้แตะ Server-side Hook และใช้อะไรแทน

ถ้าคุณลองเข้าไปหาในหน้า Settings ของ repository บน **github.com** หรือ **gitlab.com** คุณจะไม่มีทางเจอเมนูให้อัปโหลดหรือแก้ไขไฟล์ `pre-receive`, `update`, `post-receive` ได้เลยแม้แต่น้อย นี่ไม่ใช่ข้อจำกัดทางเทคนิคที่แก้ไม่ได้ แต่เป็น**การตัดสินใจเชิงสถาปัตยกรรมของแพลตฟอร์ม SaaS โดยเจตนา**

### เหตุผลที่ SaaS ไม่เปิดให้แตะ Server-side Hook โดยตรง

1. **Multi-tenancy และความปลอดภัย** — เซิร์ฟเวอร์ของ GitHub/GitLab หนึ่งเครื่องโฮสต์ repository ของ**ลูกค้าหลายพันหลายหมื่นรายพร้อมกัน** ถ้าเปิดให้ใครก็ได้อัปโหลด shell script ไปรันบนเซิร์ฟเวอร์ตอนมีคน push (ซึ่งคือหน้าที่ของ hook โดยตรง) นั่นเท่ากับเปิดช่องให้รันโค้ดที่ไม่น่าเชื่อถือ (arbitrary code execution) บนเครื่องที่ใช้ร่วมกันกับลูกค้ารายอื่น เสี่ยงต่อการโจมตีข้ามผู้เช่า (cross-tenant attack) อย่างรุนแรง
2. **ผลกระทบต่อ Performance และความเสถียรของระบบ** — hook ที่เขียนไม่ดี (เช่น loop ไม่รู้จบ หรือเรียก network call ที่ตอบช้ามาก) จะทำให้ `git push` ของผู้ใช้คนนั้นค้าง ถ้าเปิดให้ทุกคนเขียน hook เองได้อย่างอิสระบนโครงสร้างที่ใช้ร่วมกัน ความเสี่ยงที่ repository หนึ่งจะไปกระทบ performance ของทั้งระบบมีสูงมาก
3. **การดูแลรักษาและอัปเดตระบบทำได้ยากขึ้นมหาศาล** — ถ้าลูกค้าแต่ละรายมี hook script ของตัวเองอยู่บนเซิร์ฟเวอร์ การอัปเกรด Git เวอร์ชันใหม่ หรือย้าย storage backend (เช่นที่ GitHub ใช้ Spokes/DGit และ GitLab ใช้ Gitaly) จะซับซ้อนขึ้นมากเพราะต้องรับประกันว่า custom script ของทุกคนยังทำงานถูกต้องอยู่

### แต่ในเวอร์ชัน Self-hosted/Enterprise เปิดให้ใช้ได้จริง

ข้อควรรู้ที่สำคัญคือข้อจำกัดนี้ใช้กับ **github.com และ gitlab.com (บริการ SaaS สาธารณะ) เท่านั้น** ส่วนเวอร์ชันที่องค์กรติดตั้งเองนั้นเปิดให้ใช้ server-side hook ได้จริงในระดับที่ควบคุมได้ปลอดภัยกว่า:

- **GitHub Enterprise Server** (เวอร์ชัน self-hosted) มีฟีเจอร์ **"Pre-receive Hooks"** ในหน้า Site Admin โดยตรง ผู้ดูแลระบบระดับองค์กร (ไม่ใช่เจ้าของ repo ทั่วไป) สามารถอัปโหลด script ผ่าน UI ได้ แล้วนำไปผูก (enforce) กับ repository ใดก็ได้ในองค์กร
- **GitLab self-managed** มีฟีเจอร์ **"Server hooks"** และ **"File hooks"** ที่ผู้ดูแลระบบระดับ instance สามารถวางไฟล์ hook ไว้บนเครื่อง Gitaly server ได้โดยตรง

ทั้งสองกรณีนี้ยังคง**จำกัดสิทธิ์ไว้ที่ผู้ดูแลระบบระดับสูงสุดขององค์กรเท่านั้น** ไม่ใช่เปิดให้เจ้าของ repository ทั่วไปแก้ไขได้เอง — หลักการ "ต้องควบคุมโดยฝ่ายที่มีสิทธิ์เหนือกว่าผู้ push เสมอ" ยังคงเดิมทุกประการ

### แล้วบน SaaS (github.com/gitlab.com) ใช้อะไรแทน Server-side Hook

แม้จะไม่มี server-side hook ตรง ๆ แต่แพลตฟอร์ม SaaS ก็มีเครื่องมือทดแทนที่ครอบคลุมเงื่อนไขส่วนใหญ่ที่คนต้องการได้:

| ต้องการทำอะไร (เหมือน hook) | ใช้อะไรแทนบน SaaS | เทียบเท่ากับ hook ตัวไหน |
|---|---|---|
| ห้าม push ตรงเข้า main, บังคับ PR ก่อนเสมอ | **Branch Protection Rules** (GitHub) / **Protected Branches** (GitLab) | `pre-receive` / `update` |
| บังคับต้องมีคน approve ก่อน merge | Branch Protection: Require pull request reviews | `update` |
| บังคับต้อง pass test ก่อน merge | Branch Protection: Require status checks (ผูกกับ CI) | `pre-receive` (แบบ async ผ่าน CI แทน) |
| ห้าม force-push ทับประวัติ | Branch Protection: Block force pushes | `pre-receive` |
| สแกนหา secret หลุดก่อนรับเข้า | GitHub **Push Protection** (Secret Scanning) — เป็นหนึ่งในไม่กี่ฟีเจอร์ที่ทำงาน "ก่อนรับ push" จริง ๆ บน SaaS | `pre-receive` |
| trigger automation หลัง push/PR สำเร็จ | **Webhooks** หรือ **GitHub Actions / GitLab CI** | `post-receive` |
| บังคับ commit message convention | CI job ที่ตรวจสอบและ fail (ทำงานหลังรับ push แล้ว ไม่ใช่ก่อนรับ) | เทียบเท่าบางส่วนของ `pre-receive` แต่ "หลังรับ" ไม่ใช่ "ก่อนรับ" |

จุดที่ต้องสังเกตให้ดี: เครื่องมือทดแทนส่วนใหญ่ (**Branch Protection Rules**) ทำงานในระดับ **แอปพลิเคชันของแพลตฟอร์ม** ไม่ใช่ในระดับ Git protocol เหมือน hook จริง ๆ — Branch Protection ของ GitHub ทำงานโดยตรวจสอบ**ก่อน**ที่จะยอม `git receive-pack` ทำงานเลยด้วยซ้ำในบางกรณี (เช่นบล็อก push ตรงเข้า protected branch) ซึ่งให้ผลลัพธ์เหมือน `update` hook แทบทุกประการ เพียงแต่ตั้งค่าผ่าน UI/API แทนการเขียน shell script เอง ส่วน **CI Pipeline ที่เป็น required status check** นั้นทำงานหลัง push เข้าไปแล้ว (คล้าย `post-receive`) แต่เมื่อผูกเข้ากับกฎ "ห้าม merge ถ้า check ไม่ผ่าน" ของ Branch Protection มันก็กลายเป็นด่านบังคับก่อนที่โค้ดจะเข้าสู่ branch หลักได้อย่างมีประสิทธิผลเทียบเท่า `pre-receive` ในทางปฏิบัติ

เราจะเรียนเรื่อง Branch Protection Rules แบบละเอียดเต็ม ๆ อีกครั้งใน **Part 37** (ซึ่งเรียนไปแล้วก่อนหน้านี้) และ CI/CD Pipeline เต็มรูปแบบใน **Part 66 เป็นต้นไป**

---

## Step 597: Webhooks คืออะไร ต่างจาก Server-side Hook อย่างไร

**Webhook** คือกลไกที่แพลตฟอร์มอย่าง GitHub/GitLab ใช้แทนที่ `post-receive` hook แบบดั้งเดิม — มันคือการที่แพลตฟอร์ม**ส่ง HTTP request (โดยทั่วไปคือ HTTP POST) ไปยัง URL ภายนอกที่คุณกำหนดไว้ ทุกครั้งที่มีเหตุการณ์บางอย่างเกิดขึ้น** เช่น มีคน push, เปิด Pull Request, comment, สร้าง issue ฯลฯ

### ความแตกต่างพื้นฐานที่สุด: ทำงานที่ "ชั้น" ไหนของระบบ

```
Server-side Hook                          Webhook
─────────────────                         ─────────────────
ทำงานภายในกระบวนการ git receive-pack       ทำงานที่ชั้นแอปพลิเคชันของแพลตฟอร์ม
  โดยตรง (synchronous, in-process)           หลังจาก git receive-pack เสร็จสมบูรณ์แล้ว
รันเป็น local process/script บนเครื่อง       ส่งเป็น HTTP request ออกไปยัง URL
  server เดียวกับที่เก็บ repository            ภายนอก (อาจเป็นเซิร์ฟเวอร์ที่ไหนก็ได้ในโลก)
ตรวจสอบ block การ push ได้ (pre-receive/     ตรวจสอบ/block การ push ไม่ได้เลย
  update)                                      (ทำงานทีหลังเสมอ เป็น fire-and-forget)
ผลลัพธ์กระทบ push ทันที (reject/accept)      ผลลัพธ์ไม่กระทบ push แล้ว (push จบไปแล้ว
                                                ก่อนที่ webhook จะถูกยิงออกไปด้วยซ้ำ)
ติดตั้งโดย: ผู้ดูแลระบบเซิร์ฟเวอร์             ติดตั้งโดย: เจ้าของ repo เอง (ผ่าน Settings)
รูปแบบข้อมูล: text ทาง stdin/argument        รูปแบบข้อมูล: JSON payload มาตรฐาน
ความเชื่อถือได้: รันแน่นอน 100% ทุกครั้ง       ความเชื่อถือได้: อาจ delay, timeout, หรือ
  (เป็นส่วนหนึ่งของ Git protocol เอง)           ปลายทางล่มได้ (ต้องมี retry mechanism เอง)
```

### ทำไม Webhook ถึง "block push ไม่ได้"

จุดสำคัญที่สุดที่ต้องเข้าใจให้แม่นคือ **webhook ทำงานหลังจากที่ push ถูกยอมรับเข้า repository เรียบร้อยแล้วเสมอ** เมื่อ GitHub/GitLab ยอมรับการ push (ผ่านกระบวนการภายในที่เทียบเท่า `pre-receive`/`update`/`post-receive` ของแพลตฟอร์มเอง ซึ่งผู้ใช้ทั่วไปมองไม่เห็น) มันจะบันทึกข้อมูลลงในฐานข้อมูลของแพลตฟอร์มก่อน แล้วจึง**คิวงานแยกต่างหาก**เพื่อยิง HTTP request ไปหา URL ที่ตั้งค่าไว้ ซึ่งเป็นคนละกระบวนการ คนละจังหวะเวลากันโดยสิ้นเชิงกับกระบวนการรับ push จริง ๆ

ด้วยเหตุนี้ **ไม่ว่า endpoint ปลายทางของ webhook จะตอบกลับว่าอย่างไร (สำเร็จ, error 500, หรือ timeout) ก็ไม่มีผลย้อนกลับไปเปลี่ยนแปลงว่า push ครั้งนั้นสำเร็จหรือไม่แล้ว** — นี่คือสิ่งที่ทำให้ webhook เทียบเท่ากับ `post-receive` เท่านั้น ไม่มีทางทำหน้าที่แบบ `pre-receive`/`update` ได้เลยบน SaaS ทั่วไป

### รูปแบบ Payload มาตรฐานของ Webhook

เมื่อเหตุการณ์เกิดขึ้น แพลตฟอร์มจะส่ง HTTP POST พร้อม JSON body ที่บรรยายรายละเอียดของเหตุการณ์นั้นทั้งหมด ตัวอย่าง payload ของ GitHub เมื่อมีการ push (ตัดให้สั้นลง):

```json
{
  "ref": "refs/heads/main",
  "before": "a1b2c3d4e5f6...",
  "after": "f6e5d4c3b2a1...",
  "repository": {
    "full_name": "myorg/myapp"
  },
  "pusher": {
    "name": "phutjirakul"
  },
  "commits": [
    {
      "id": "f6e5d4c3b2a1...",
      "message": "feat(checkout): เพิ่มระบบชำระเงินผ่าน QR Code",
      "author": { "name": "Phutjirakul", "email": "..." }
    }
  ]
}
```

ทุก webhook request จะมี HTTP header พิเศษแนบมาด้วยเพื่อยืนยันแหล่งที่มา:

- **GitHub**: header `X-Hub-Signature-256` ที่เป็นค่า HMAC-SHA256 ของ payload เข้ารหัสด้วย secret ที่คุณตั้งไว้ตอนสร้าง webhook — ปลายทางต้องคำนวณ HMAC เทียบกลับเพื่อยืนยันว่า request นี้มาจาก GitHub จริง ไม่ใช่ถูกปลอมแปลงส่งมา
- **GitLab**: header `X-Gitlab-Token` ที่เป็นค่า secret token ตรง ๆ ที่ตั้งไว้ (ไม่ได้ hash เหมือน GitHub) ปลายทางแค่เทียบค่าตรง ๆ ว่าตรงกับที่ตั้งค่าไว้หรือไม่

การตรวจสอบ signature/token นี้**สำคัญมาก** เพราะ URL ของ webhook endpoint มักเปิดรับ request จากอินเทอร์เน็ตสาธารณะ ถ้าไม่ตรวจสอบ จะมีใครก็ได้ปลอมแปลง payload ส่งมาหลอกระบบของคุณได้

---

## Step 598: ตั้งค่า Webhook ส่ง Notification ไปยัง Slack

มาลองตั้งค่า webhook จริงเพื่อส่ง notification ไปยัง Slack ทุกครั้งที่มีการ push หรือเปิด Pull Request ในองค์กร ซึ่งเป็นหนึ่งใน use case ที่พบบ่อยที่สุดของ webhook ในโลกจริง

### ฝั่ง Slack: สร้าง Incoming Webhook ก่อน

1. เข้าไปที่ **Slack App Directory** ของ workspace แล้วค้นหา **"Incoming Webhooks"**
2. เพิ่ม App ลงในช่อง (channel) ที่ต้องการให้ notification ไปโผล่ เช่น `#dev-notifications`
3. Slack จะสร้าง URL เฉพาะให้ ในรูปแบบ `https://hooks.slack.com/services/<TEAM_ID>/<BOT_ID>/<TOKEN>` (ค่าจริงจะเป็นรหัสเฉพาะของ workspace คุณ ไม่ใช่ค่าตัวอย่างในเอกสารนี้)
4. เก็บ URL นี้ไว้ — มันคือ URL ที่จะรับ payload แล้วโพสต์เป็นข้อความในช่องที่กำหนดโดยอัตโนมัติ

### ฝั่ง GitHub: ตั้งค่า Webhook ให้ยิงไปที่ Slack

ไปที่ **Repository → Settings → Webhooks → Add webhook** แล้วกรอก:

| ช่อง | ค่าที่ตั้ง |
|---|---|
| Payload URL | URL จาก Slack Incoming Webhook (หรือ URL ของ middleware กลางถ้าต้องแปลง format ก่อน) |
| Content type | `application/json` |
| Secret | สุ่ม string ยาว ๆ ไว้ตรวจสอบ `X-Hub-Signature-256` |
| Which events | เลือก `Just the push event` หรือ `Let me select individual events` แล้วติ๊ก `Pushes` และ `Pull requests` |

> **ข้อควรระวังสำคัญ:** Payload ที่ GitHub ส่งมากับ Slack Incoming Webhook **ไม่ตรงรูปแบบกันโดยตรง** — Slack ต้องการ JSON รูปแบบเฉพาะของตัวเอง (เช่น key `text` หรือ `blocks`) ในขณะที่ GitHub ส่ง JSON โครงสร้างเหตุการณ์ของตัวเองมา (`ref`, `commits`, `pusher` ฯลฯ) ในทางปฏิบัติจริงจึงมักไม่ได้เอา URL ของ Slack ไปใส่ตรง ๆ ในช่อง Payload URL ของ GitHub แต่จะใช้ **middleware กลาง** (เช่น Zapier, n8n, หรือ AWS Lambda function เล็ก ๆ ที่เราเขียนเอง) มารับ payload จาก GitHub ก่อน แล้วแปลง (transform) เป็นรูปแบบที่ Slack เข้าใจ แล้วค่อยส่งต่อไปอีกที หรือใช้ **GitHub App สำหรับ Slack อย่างเป็นทางการ** (ค้นหา "GitHub" ใน Slack App Directory) ซึ่งจัดการเรื่องการแปลงรูปแบบให้ทั้งหมดโดยอัตโนมัติแล้ว

### ตัวอย่าง Middleware แบบง่าย (แนวคิด ไม่ใช่โค้ด production)

```
GitHub push event (JSON)
        │
        ▼
  Middleware/Lambda function
  - รับ POST จาก GitHub
  - ตรวจสอบ X-Hub-Signature-256 ด้วย secret
  - แปลง JSON ของ GitHub เป็น {"text": "..."} ตามที่ Slack ต้องการ
        │
        ▼
  ส่งต่อไปยัง Slack Incoming Webhook URL
        │
        ▼
  ข้อความโผล่ในช่อง #dev-notifications
```

ตัวอย่างการแปลง payload อย่างง่ายด้วย pseudo-code:

```python
def handle_github_webhook(request):
    verify_signature(request, secret)  # ป้องกันการปลอมแปลง

    payload = request.json
    branch = payload["ref"].split("/")[-1]
    pusher = payload["pusher"]["name"]
    messages = [c["message"] for c in payload["commits"]]

    slack_text = f"📦 *{pusher}* push เข้า `{branch}`:\n" + "\n".join(f"• {m}" for m in messages)

    post_to_slack({"text": slack_text})
```

### ฝั่ง GitLab: ตั้งค่าคล้ายกัน

GitLab มีแนวคิดเดียวกันที่ **Project → Settings → Webhooks** โดยกรอก URL, Secret Token, และเลือก event ที่สนใจ (Push events, Merge request events, Tag push events ฯลฯ) — ต่างจาก GitHub ตรงที่ GitLab ยังมี **Slack integration แบบสำเร็จรูป** อยู่ที่ **Settings → Integrations → Slack notifications** ซึ่งจัดการแปลง format ให้อัตโนมัติโดยไม่ต้องเขียน middleware เองเลย เหมาะกับกรณีใช้งานทั่วไปที่ไม่ต้องการ custom logic อะไรพิเศษ

### จุดสำคัญที่ต้องออกแบบรองรับ: Webhook อาจถูกยิงซ้ำหรือมาไม่ตามลำดับ

เพราะ webhook เป็นกลไกแบบ **asynchronous ผ่านเครือข่าย** จึงมีโอกาสที่:

- ปลายทางตอบช้าเกินไปจนแพลตฟอร์ม **retry** ส่งซ้ำมาอีกครั้ง (endpoint ของคุณอาจได้รับ event เดียวกันซ้ำสองครั้ง)
- event หลายตัวอาจมาถึงปลายทางไม่เรียงตามลำดับเวลาที่เกิดขึ้นจริง เพราะเครือข่ายและคิวงานฝั่งแพลตฟอร์ม

ระบบที่รับ webhook ควรออกแบบให้ **idempotent** (รับ event ซ้ำแล้วไม่เกิดผลข้างเคียงซ้ำ เช่น เช็ค delivery ID ที่แนบมาด้วย header `X-GitHub-Delivery` ก่อนประมวลผลซ้ำ) ซึ่งเป็นข้อควรระวังที่แตกต่างจาก server-side hook โดยสิ้นเชิง เพราะ server-side hook รันแบบ synchronous ในกระบวนการเดียวกับ push จึงไม่มีปัญหาเรื่องลำดับหรือการยิงซ้ำแบบนี้เลย

---

## Step 599: Server-side Hooks กับ Bare Repository บน Self-hosted Git Server

มาถึงจุดที่ทุกอย่างที่เรียนมาใน Part นี้ **กลับมามีประโยชน์เต็มรูปแบบที่สุด** — สถานการณ์ที่องค์กร**ไม่ได้ใช้ GitHub หรือ GitLab เลย** แต่ตั้ง Git server ขึ้นมาเองบนเครื่องของบริษัท (self-hosted)

### สถาปัตยกรรมพื้นฐานของ Self-hosted Git Server

รูปแบบที่ง่ายที่สุดคือการมีเครื่อง Linux server หนึ่งเครื่อง (เช่น `git.company.local`) ที่เก็บ **bare repository** ไว้ในโฟลเดอร์ เช่น `/srv/git/myapp.git` แล้วให้ developer เข้าถึงผ่าน SSH:

```bash
# ฝั่ง developer
git clone ssh://git@git.company.local/srv/git/myapp.git
git push origin main
```

การเข้าถึงแบบนี้ไม่มี "แอปพลิเคชันชั้นกลาง" อย่าง GitHub/GitLab คอยดักตรวจสอบ Branch Protection ให้เลย — **มีแค่ Git protocol ล้วน ๆ ที่คุยกับ bare repository ตรง ๆ ผ่าน SSH** ดังนั้น **server-side hooks จึงเป็นกลไกเดียวที่มีอยู่ในการบังคับนโยบายใด ๆ ทั้งสิ้น** ไม่มี UI ให้ตั้งค่า Branch Protection แบบ point-and-click เหมือนที่เคยเห็นบน SaaS

```
Developer เครื่อง A ──┐
Developer เครื่อง B ──┼──▶ SSH ──▶ git.company.local
Developer เครื่อง C ──┘              │
                                      ▼
                        /srv/git/myapp.git (bare repo)
                              hooks/pre-receive   ← ด่านเดียวที่บังคับกฎได้จริง
                              hooks/update
                              hooks/post-receive
```

### เครื่องมือจัดการสิทธิ์ที่นิยมคู่กับ Self-hosted Git: gitolite

เพราะ SSH access ตรง ๆ ไม่มีระบบจัดการสิทธิ์แบบละเอียด (ใครมีสิทธิ์ push repo ไหนได้บ้าง) หลายองค์กรจึงใช้เครื่องมืออย่าง **gitolite** วางไว้เป็นชั้นกลาง โดย gitolite ทำงานผ่านกลไก **SSH forced command** — กำหนดให้ทุก SSH key ที่ลงทะเบียนไว้ ไม่ว่าจะเชื่อมต่อมาอย่างไร จะถูกบังคับให้รันโปรแกรม `gitolite-shell` เท่านั้น (ไม่ใช่ shell ทั่วไป) ซึ่งจะตรวจสอบไฟล์ config กลาง (`gitolite-admin` repo) ว่า user นี้มีสิทธิ์ `R` (read) หรือ `RW` (read-write) กับ repository ที่กำลังจะเข้าถึงหรือไม่ ก่อนจะยอมส่งต่อไปยัง `git-upload-pack`/`git-receive-pack` จริง

gitolite เองก็มีกลไกที่เรียกว่า **VREF (virtual ref) hooks** ซึ่งเป็นชั้นห่อหุ้ม (wrapper) อยู่เหนือ server-side hook ธรรมดาอีกที ทำให้ตั้งกฎละเอียดระดับ "user คนนี้ push ได้แค่ branch pattern นี้เท่านั้น" ได้สะดวกกว่าการเขียน raw hook เอง แต่โดยพื้นฐานที่สุดแล้วมันก็ยังคงสร้างมาจากกลไก `pre-receive`/`update` เดิมของ Git นั่นเอง

### ปัญหาที่ตามมาเมื่อมีหลาย Repository: การกระจาย (Distribute) Hook ไปทุก Repo

ถ้าองค์กรมี repository เป็นร้อย ๆ ตัวบน server เดียวกัน การ copy ไฟล์ hook เดียวกันไปวางไว้ใน `hooks/` ของทุก repo ทีละตัวด้วยมือ**ไม่ใช่แนวทางที่ยั่งยืน** เพราะเมื่อไหร่ที่ต้องแก้กฎ ก็ต้องไล่แก้ทุก repo ใหม่หมด วิธีแก้ปัญหานี้ที่ใช้กันจริงมีสองแนวทางหลัก:

#### แนวทางที่ 1: Git Template Directory

Git มีกลไกในตัวชื่อ **template directory** — เมื่อสร้าง repository ใหม่ด้วย `git init` (หรือ `git init --bare`) Git จะ copy ไฟล์ทั้งหมดจากโฟลเดอร์ template มาใส่ใน repo ใหม่โดยอัตโนมัติ รวมถึงไฟล์ใน `hooks/` ด้วย:

```bash
# ตั้งค่า template directory ส่วนกลางไว้ครั้งเดียว
git config --global init.templateDir /srv/git-templates

# วาง hook มาตรฐานขององค์กรไว้ที่นั่น
cp /srv/git-hooks/pre-receive /srv/git-templates/hooks/pre-receive
chmod +x /srv/git-templates/hooks/pre-receive

# ต่อไปนี้ ทุกครั้งที่สร้าง bare repo ใหม่ hook จะติดมาอัตโนมัติ
git init --bare /srv/git/new-project.git
ls /srv/git/new-project.git/hooks/
# pre-receive   ← ติดมาจาก template โดยอัตโนมัติ
```

ข้อจำกัดของวิธีนี้คือมันมีผลแค่ตอน **สร้าง repo ใหม่เท่านั้น** repo ที่มีอยู่แล้วก่อนหน้าจะไม่ได้รับผลกระทบ ต้องไปอัปเดตเองแยกต่างหาก

#### แนวทางที่ 2: Symlink ไปยัง Script กลาง

วิธีที่นิยมกว่าในการดูแลรักษาระยะยาวคือการไม่ copy ไฟล์ hook ไปวางตรง ๆ แต่สร้างเป็น **symbolic link** ชี้กลับไปยัง script ต้นทางที่เก็บไว้ที่เดียวส่วนกลาง:

```bash
# script ต้นทางเก็บไว้ที่เดียว ควบคุมด้วย Git ของตัวเอง (versioned)
/srv/git-hooks-central/pre-receive

# ทุก bare repo ทำ symlink ไปหา
ln -s /srv/git-hooks-central/pre-receive /srv/git/project-a.git/hooks/pre-receive
ln -s /srv/git-hooks-central/pre-receive /srv/git/project-b.git/hooks/pre-receive
ln -s /srv/git-hooks-central/pre-receive /srv/git/project-c.git/hooks/pre-receive
```

ด้วยวิธีนี้ เมื่อผู้ดูแลระบบแก้ไข script ต้นทางเพียงไฟล์เดียว **ทุก repository ที่ symlink ไปหามันจะได้รับการอัปเดตทันทีโดยอัตโนมัติ** ไม่ต้องไล่แก้ทีละ repo — และเพราะ script ต้นทางเก็บอยู่ใน repository ของตัวเอง (versioned ด้วย Git) ทีมยังสามารถ review การเปลี่ยนแปลงกฎผ่าน Pull Request ได้เหมือนโค้ดทั่วไป ก่อนจะมีผลจริงกับทุก repo ในองค์กร

---

## Step 600: แบบฝึกหัด — เขียน `pre-receive` hook บน Bare Repository จำลอง

ถึงเวลาลงมือทำจริงเพื่อสรุปทุกอย่างที่เรียนมาใน Part นี้ เราจะสร้าง bare repository จำลองขึ้นมาบนเครื่องของคุณเอง แล้วเขียน `pre-receive` hook ที่บังคับ 2 กฎสำคัญ: **(1) ห้าม push ตรงเข้า `main`** และ **(2) บังคับรูปแบบ commit message**

### ขั้นตอนที่ 1: สร้าง Bare Repository จำลอง

```bash
mkdir -p ~/git-course/part-60-practice
cd ~/git-course/part-60-practice

# สร้าง bare repository ที่จะทำหน้าที่เป็น "server" จำลอง
git init --bare central.git
```

```
Initialized empty Git repository in /home/you/git-course/part-60-practice/central.git/
```

### ขั้นตอนที่ 2: เขียน `pre-receive` Hook

```bash
cd central.git/hooks

cat > pre-receive << 'EOF'
#!/bin/sh
# pre-receive hook แบบฝึกหัด: ห้าม push ตรงเข้า main + บังคับ commit message format

zero="0000000000000000000000000000000000000000"
allowed_types="feat|fix|docs|style|refactor|perf|test|chore"

while read oldrev newrev refname
do
  branch=$(echo "$refname" | sed 's#refs/heads/##')

  # ข้ามการลบ branch
  if [ "$newrev" = "$zero" ]; then
    continue
  fi

  # กฎที่ 1: ห้าม push commit เดี่ยว (ไม่ใช่ merge commit) เข้า main โดยตรง
  if [ "$branch" = "main" ]; then
    parent_count=$(git cat-file -p "$newrev" | grep -c '^parent ')
    if [ "$parent_count" -lt 2 ]; then
      echo "=========================================================="
      echo " REJECTED: ห้าม push ตรงเข้า main โดยตรง"
      echo " กรุณาสร้าง branch แยก แล้ว merge ผ่าน Pull Request เท่านั้น"
      echo "=========================================================="
      exit 1
    fi
  fi

  # กฎที่ 2: ทุกคอมมิตใหม่ต้องตรงรูปแบบ <type>: <description>
  if [ "$oldrev" = "$zero" ]; then
    commit_range=$(git rev-list "$newrev" --not --all)
  else
    commit_range=$(git rev-list "$oldrev".."$newrev")
  fi

  for commit in $commit_range; do
    subject=$(git log -1 --format=%s "$commit")
    if ! echo "$subject" | grep -Eq "^($allowed_types)(\([a-z0-9_-]+\))?: .+"; then
      echo "=========================================================="
      echo " REJECTED: commit $commit ผิดรูปแบบ"
      echo " ข้อความ: '$subject'"
      echo " ต้องขึ้นต้นด้วย feat:, fix:, docs:, style:, refactor:,"
      echo " perf:, test: หรือ chore: เท่านั้น"
      echo "=========================================================="
      exit 1
    fi
  done
done

exit 0
EOF

chmod +x pre-receive
cd ../..
```

### ขั้นตอนที่ 3: Clone มาทำงานเหมือน Developer จริง

```bash
git clone central.git workspace
cd workspace
git config user.name "ผู้ฝึกหัด"
git config user.email "practice@example.com"
```

### ขั้นตอนที่ 4: ทดสอบกฎที่ 1 — ห้าม push ตรงเข้า main

```bash
echo "hello" > README.md
git add README.md
git commit -m "feat: เพิ่มไฟล์ README เริ่มต้น"
git push origin main
```

**ผลลัพธ์ที่ควรเห็น (ถูกปฏิเสธ เพราะเป็น commit เดี่ยว ไม่ใช่ merge commit):**

```
remote: ==========================================================
remote:  REJECTED: ห้าม push ตรงเข้า main โดยตรง
remote:  กรุณาสร้าง branch แยก แล้ว merge ผ่าน Pull Request เท่านั้น
remote: ==========================================================
To .../central.git
 ! [remote rejected] main -> main (pre-receive hook declined)
error: failed to push some refs to '.../central.git'
```

### ขั้นตอนที่ 5: ทดสอบกฎที่ 2 — commit message ผิดรูปแบบ

```bash
git checkout -b feature/readme
git commit --allow-empty -m "แก้ readme นิดหน่อย"
git push origin feature/readme
```

**ผลลัพธ์ที่ควรเห็น (ถูกปฏิเสธ เพราะ message ไม่ตรง Conventional Commits):**

```
remote: ==========================================================
remote:  REJECTED: commit 3f9a21c ผิดรูปแบบ
remote:  ข้อความ: 'แก้ readme นิดหน่อย'
remote:  ต้องขึ้นต้นด้วย feat:, fix:, docs:, style:, refactor:,
remote:  perf:, test: หรือ chore: เท่านั้น
remote: ==========================================================
 ! [remote rejected] feature/readme -> feature/readme (pre-receive hook declined)
```

### ขั้นตอนที่ 6: แก้ไขให้ถูกต้องแล้ว push ผ่านเส้นทางที่ถูกต้อง

```bash
git commit --amend -m "docs: ปรับปรุงเนื้อหา README"
git push origin feature/readme
```

**ผลลัพธ์: ผ่าน** เพราะเป็น branch `feature/readme` ไม่ใช่ `main` และ message ตรงรูปแบบ:

```
To .../central.git
 * [new branch]      feature/readme -> feature/readme
```

จากนั้นจำลองขั้นตอน merge เข้า main ผ่าน merge commit (จำลองการที่ Pull Request ถูก approve และ merge):

```bash
git checkout main
git merge --no-ff feature/readme -m "Merge feature/readme into main

Reviewed-by: หัวหน้าทีม <lead@example.com>"
git push origin main
```

**ผลลัพธ์: ผ่าน** เพราะตอนนี้เป็น merge commit ที่มีมากกว่า 1 parent แล้ว:

```
To .../central.git
   a1b2c3d..d4e5f6a  main -> main
```

### ขั้นตอนที่ 7: ทดสอบเพิ่มเติมด้วยตัวเอง (ไม่มีเฉลย ลองทำเอง)

ลองต่อยอดแบบฝึกหัดนี้ด้วยตัวเองก่อนไป Part ถัดไป:

1. เพิ่มกฎที่ 3 เข้าไปใน hook: ห้าม push ไฟล์ที่มีนามสกุล `.env` หรือ `.pem` เข้ามาใน repository เลย (สแกนจาก object ที่ถูก push ด้วย `git diff --name-only oldrev newrev` หรือ `git diff-tree`)
2. ลองเปลี่ยน `pre-receive` ให้เป็น `update` hook แทน แล้วสังเกตความแตกต่างของพฤติกรรมเมื่อ push หลาย branch พร้อมกันโดยมีบาง branch ผิดกฎ
3. เพิ่ม `post-receive` hook ที่พิมพ์ ASCII art เล็ก ๆ พร้อมข้อความ "ขอบคุณที่ push เข้า main!" ทุกครั้งที่มีคน merge สำเร็จ เพื่อฝึกความเข้าใจว่า `post-receive` ทำงานหลัง `pre-receive` เสมอ

### Checklist ก่อนไป Part ถัดไป

- [ ] สร้าง bare repository จำลองได้ และเข้าใจว่ามันคือ "repository ฝั่ง server" ที่ไม่มี working directory
- [ ] เขียน `pre-receive` hook ที่อ่านค่าจาก stdin และใช้ exit code ควบคุมการ accept/reject การ push ได้
- [ ] เห็นการ push ถูกปฏิเสธจริงตามกฎที่กำหนด และเข้าใจข้อความ `remote:` ที่ Git แสดงกลับมา
- [ ] ทดสอบ flow เต็มรูปแบบ: branch ใหม่ → commit ผิดกฎ (ถูกปฏิเสธ) → แก้ไข → push สำเร็จ → merge เข้า main ผ่าน merge commit ที่มี `Reviewed-by:`

---

## สรุป Part 60

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Server-side Hooks ต่างจาก Client-side Hooks ตรงที่มัน "บังคับใช้ได้จริง"** เพราะรันอยู่บนเครื่องเซิร์ฟเวอร์ที่ผู้ push ไม่มีสิทธิ์เข้าถึงหรือแก้ไข ไม่มีทาง bypass ได้เลยไม่ว่าจะตั้งใจแค่ไหน
2. Git มี server-side hooks 4 ตัว: **`pre-receive`** (บล็อกทั้ง push ก่อนอัปเดต ref ใด ๆ, รับข้อมูลทาง stdin), **`update`** (ตรวจทีละ ref, บล็อกได้เฉพาะ ref นั้น, รับข้อมูลทาง argument), **`post-receive`** (trigger automation หลัง push สำเร็จ, block อะไรไม่ได้แล้ว) และ **`post-update`** (legacy hook สำหรับ dumb HTTP)
3. **`pre-receive`** เหมาะกับกฎที่ต้องดูภาพรวมทั้ง push พร้อมกัน ส่วน **`update`** เหมาะกับกฎที่ต่างกันไปตาม branch/ref และต้องการให้ ref ที่ถูกต้องผ่านได้แม้ ref อื่นจะผิด
4. ตัวอย่างจริงจาก Part 45 ที่บังคับ Branch Naming และ Branch Protection สามารถต่อยอดให้บังคับ **Conventional Commits** ได้ด้วยการวนตรวจทุกคอมมิตในช่วง `oldrev..newrev`
5. **GitHub/GitLab แบบ SaaS ไม่เปิดให้แตะ server-side hook โดยตรง** ด้วยเหตุผลด้าน multi-tenancy และความปลอดภัย แต่มี **Branch Protection Rules**, **Push Protection**, **CI required status checks** เป็นเครื่องมือทดแทนที่ให้ผลลัพธ์เทียบเท่าในทางปฏิบัติ ส่วนเวอร์ชัน Enterprise/self-managed ยังเปิดให้ใช้ server-side hook จริงได้ (จำกัดสิทธิ์แค่ระดับ admin)
6. **Webhook คือกลไกทดแทน `post-receive`** ที่ทำงานที่ชั้นแอปพลิเคชันของแพลตฟอร์ม ส่ง HTTP POST ไปยัง URL ภายนอก **ไม่สามารถบล็อกการ push ได้เลย** เพราะทำงานหลังจากรับ push เสร็จสมบูรณ์แล้วเสมอ
7. ในองค์กรที่ **self-host Git server เอง** (ไม่ผ่าน GitHub/GitLab) server-side hooks คือกลไกเดียวที่มีอยู่ในการบังคับนโยบาย และมีเทคนิคการกระจาย hook ไปยังหลาย repository ผ่าน template directory หรือ symlink ไปยัง script กลาง
8. ลงมือเขียนและทดสอบ `pre-receive` hook จริงบน bare repository จำลอง เห็นการ reject/accept การ push ตามเงื่อนไขที่กำหนดด้วยตัวเอง

**ต่อไป:** [Part 61: Git Attributes: .gitattributes และการจัดการไฟล์พิเศษ](./part-061-git-attributes.md)
