# Part 05: การดูประวัติ: log, diff, show และการอ่าน commit graph

> **Step ในหลักสูตรนี้:** Step 41–50
> **เฟส:** 1 — ปูพื้นฐานความคิดเรื่อง Version Control และ Git (Part สุดท้ายของเฟส 1)
> **เป้าหมายของ Part นี้:** เข้าใจและใช้งานคำสั่งกลุ่ม "ดูประวัติ" ของ Git อย่างคล่องแคล่ว — `git log` ในทุกรูปแบบที่ใช้งานจริง, `git diff` ในทุกโหมดการเปรียบเทียบ, `git show` สำหรับดูรายละเอียด commit เดียว, การอ้างอิง commit ด้วย `~` และ `^`, และปิดท้ายด้วยแบบฝึกหัดใหญ่ที่รวบยอดทุกอย่างที่เรียนมาตลอดเฟส 1

---

## สารบัญของ Part นี้

- Step 41: `git log` พื้นฐาน — อ่านผลลัพธ์ทีละส่วน
- Step 42: `git log` options สำคัญ — `--oneline`, `--graph`, `--all`, `--decorate`
- Step 43: การกรอง log — `-n`, `--since`, `--until`, `--author`, `--grep`
- Step 44: `git diff` — เข้าใจว่า diff เทียบอะไรกับอะไร
- Step 45: `git diff HEAD` vs `git diff --staged`/`--cached`
- Step 46: `git show <commit>` — ดูรายละเอียด commit เดียวแบบเต็ม
- Step 47: การอ้างอิง commit — hash, HEAD, `~` และ `^`
- Step 48: `git log -p` และ `git log --stat`
- Step 49: ปรับแต่งรูปแบบ log ด้วย `--pretty=format` และการทำ alias
- Step 50: แบบฝึกหัดจบเฟส 1 — วิเคราะห์ประวัติโปรเจกต์ + Checklist ทบทวนภาพรวมเฟส 1

---

## Step 41: `git log` พื้นฐาน — อ่านผลลัพธ์ทีละส่วน

ตลอด Part 03–04 เราสร้าง commit ไปหลายครั้งแล้ว แต่เรายังไม่เคย "มองย้อนกลับไป" อย่างจริงจังว่าประวัติที่เราสร้างมันหน้าตาเป็นอย่างไร นี่คือหน้าที่ของคำสั่งที่สำคัญที่สุดคำสั่งหนึ่งใน Git นั่นคือ `git log`

### ทำไม `git log` ถึงสำคัญ

จำไว้ว่า Git เก็บข้อมูลแบบ **Snapshot** ตามที่เราเรียนใน Part 01 (Step 6) — ทุก commit คือภาพถ่ายหนึ่งภาพของโปรเจกต์ ณ เวลาหนึ่ง `git log` คือหน้าต่างที่ทำให้เรามองเห็น **ลำดับของภาพถ่ายเหล่านั้นทั้งหมด** เรียงจากล่าสุดไปเก่าสุด

ลองสมมติว่าคุณมีโปรเจกต์ที่สร้างและ commit มาแล้วหลายครั้งตั้งแต่ Part 04 (`git init`, สร้างไฟล์, `git add`, `git commit` ซ้ำไปเรื่อย ๆ) ให้เข้าไปที่โฟลเดอร์โปรเจกต์นั้นแล้วรัน:

```bash
cd ~/git-course/part-04-practice
git log
```

**ผลลัพธ์ตัวอย่าง:**

```
commit 7e4a3f9c1d2b8e6a5f0c3d9e8b7a6c5d4e3f2a1b (HEAD -> main)
Author: Nattapong S. <nattapong@example.com>
Date:   Fri Sep 26 14:32:10 2026 +0700

    เพิ่มไฟล์ README อธิบายโปรเจกต์

commit 3c8b1a9f6e2d4c7b5a0f9e8d7c6b5a4f3e2d1c0b
Author: Nattapong S. <nattapong@example.com>
Date:   Thu Sep 25 10:15:44 2026 +0700

    แก้ไข bug การคำนวณราคารวม

commit 9f2e6d5c4b3a2f1e0d9c8b7a6f5e4d3c2b1a0f9e
Author: Nattapong S. <nattapong@example.com>
Date:   Wed Sep 24 09:02:31 2026 +0700

    เพิ่มฟังก์ชันคำนวณราคาสินค้า

commit 1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5e4f3a2b
Author: Nattapong S. <nattapong@example.com>
Date:   Tue Sep 23 20:11:05 2026 +0700

    initial commit
```

### อ่านผลลัพธ์ทีละส่วน

มาแยกส่วนประกอบของแต่ละ block ทีละบรรทัด:

| ส่วนของผลลัพธ์ | ความหมาย |
|---|---|
| `commit 7e4a3f9c1d2b...` | **Commit hash** แบบเต็ม (SHA-1, 40 ตัวอักษร) เป็น ID ที่ไม่ซ้ำกันของ commit นี้ในจักรวาลทั้งหมด |
| `(HEAD -> main)` | บอกว่า commit นี้คือตำแหน่งปัจจุบันของ `HEAD` และ branch `main` กำลังชี้มาที่นี่ (จะอธิบายละเอียดใน Step 47) |
| `Author: Nattapong S. <...>` | ผู้เขียน commit นี้ ดึงมาจากค่าที่ตั้งด้วย `git config user.name` และ `user.email` (ที่เราตั้งค่าไว้ตั้งแต่ Part 02) |
| `Date: Fri Sep 26 ...` | วันเวลาที่สร้าง commit นี้ พร้อม timezone offset (`+0700` คือเวลาไทย) |
| `เพิ่มไฟล์ README ...` | **Commit message** ข้อความอธิบายว่า commit นี้ทำอะไร (บรรทัดว่างก่อนข้อความคือรูปแบบมาตรฐานของ Git) |

### สิ่งสำคัญที่ต้องสังเกต

1. **เรียงจากใหม่ไปเก่า (reverse chronological order)** — commit ล่าสุดอยู่บนสุดเสมอ
2. **แต่ละ commit มี hash ไม่ซ้ำกันเลย** — hash คำนวณจากเนื้อหาของ commit (snapshot, parent, author, message, เวลา) ถ้าอะไรเปลี่ยนแม้แต่ตัวเดียว hash จะเปลี่ยนทั้งหมด
3. **`git log` เป็นคำสั่งแบบ pager** — ถ้าประวัติยาวมาก มันจะเปิดโปรแกรม `less` ให้เลื่อนดู กด `q` เพื่อออก, กด spacebar เพื่อเลื่อนหน้า, กดลูกศรขึ้น-ลงเพื่อเลื่อนทีละบรรทัด

### ทดลองเพิ่มเติม

ลองดูเฉพาะ commit เดียวล่าสุดด้วย flag `-1`:

```bash
git log -1
```

```
commit 7e4a3f9c1d2b8e6a5f0c3d9e8b7a6c5d4e3f2a1b (HEAD -> main)
Author: Nattapong S. <nattapong@example.com>
Date:   Fri Sep 26 14:32:10 2026 +0700

    เพิ่มไฟล์ README อธิบายโปรเจกต์
```

> **หมายเหตุ:** ถ้ารัน `git log` ในโฟลเดอร์ที่ไม่มี commit เลย (repo เพิ่ง `git init` ใหม่) จะได้ error ว่า `fatal: your current branch 'main' does not have any commits yet` — เป็นเรื่องปกติ ไม่ใช่ปัญหา

---

## Step 42: `git log` options สำคัญ — `--oneline`, `--graph`, `--all`, `--decorate`

`git log` แบบเต็มให้ข้อมูลครบ แต่กินพื้นที่หน้าจอมาก เมื่อประวัติมีหลายสิบ หลายร้อย commit การดูแบบเต็มไม่สะดวก Git จึงมี options ที่ช่วยย่อและจัดรูปแบบผลลัพธ์ให้อ่านง่ายขึ้น

### `--oneline`: ย่อให้เหลือบรรทัดเดียวต่อ commit

```bash
git log --oneline
```

**ผลลัพธ์:**

```
7e4a3f9 เพิ่มไฟล์ README อธิบายโปรเจกต์
3c8b1a9 แก้ไข bug การคำนวณราคารวม
9f2e6d5 เพิ่มฟังก์ชันคำนวณราคาสินค้า
1a0b9c8 initial commit
```

สังเกตว่า hash ถูกย่อเหลือ 7 ตัวอักษร (short hash) ซึ่งโดยทั่วไปเพียงพอที่จะระบุ commit ได้ไม่ซ้ำกันในโปรเจกต์ขนาดทั่วไป (Git จะขยายความยาวอัตโนมัติถ้าจำเป็นในโปรเจกต์ที่มี commit เยอะมาก)

### `--graph`: วาดเส้นกราฟแสดงการแตกและรวม branch

เมื่อโปรเจกต์มีการสร้าง branch และ merge (ซึ่งเราจะเรียนลึกใน Part 07 เป็นต้นไป) `--graph` จะวาดเส้น ASCII art แสดงโครงสร้างของ commit graph:

```bash
git log --oneline --graph
```

**ผลลัพธ์ตัวอย่าง (สมมติมีการ merge branch แล้ว):**

```
*   a1b2c3d Merge branch 'feature-login'
|\
| * f9e8d7c เพิ่มหน้า login
| * c6b5a4f เพิ่ม validation ฟอร์ม login
* | 7e4a3f9 เพิ่มไฟล์ README อธิบายโปรเจกต์
|/
* 3c8b1a9 แก้ไข bug การคำนวณราคารวม
* 9f2e6d5 เพิ่มฟังก์ชันคำนวณราคาสินค้า
* 1a0b9c8 initial commit
```

ถ้าโปรเจกต์ของคุณตอนนี้ยังไม่มี branch เลย (มีแค่ `main` เส้นเดียว) ผลลัพธ์จาก `--graph` จะเห็นแค่เส้นตรงเส้นเดียว:

```
* 7e4a3f9 เพิ่มไฟล์ README อธิบายโปรเจกต์
* 3c8b1a9 แก้ไข bug การคำนวณราคารวม
* 9f2e6d5 เพิ่มฟังก์ชันคำนวณราคาสินค้า
* 1a0b9c8 initial commit
```

ไม่ต้องกังวลถ้ายังไม่เห็นเส้นแตกแขนง — เราจะได้เห็นภาพที่ซับซ้อนขึ้นมากเมื่อเรียนเรื่อง branching ใน Part 07–12

### `--all`: แสดงทุก branch ไม่ใช่แค่ branch ปัจจุบัน

โดย default `git log` จะแสดงเฉพาะประวัติของ branch ที่คุณอยู่ปัจจุบัน (ที่ `HEAD` ชี้อยู่) ถ้าอยากเห็น **ทุก branch พร้อมกัน** ต้องเพิ่ม `--all`:

```bash
git log --oneline --graph --all
```

ผลลัพธ์จะรวม branch อื่น ๆ ที่มีอยู่ในโปรเจกต์ทั้งหมด ไม่ใช่แค่ `main` — มีประโยชน์มากเมื่อคุณอยากเห็นภาพรวมว่าทีมกำลังทำอะไรอยู่บ้างในแต่ละ branch

### `--decorate`: แสดง label ของ branch/tag ที่ชี้มาที่ commit นั้น

จริง ๆ แล้ว Git สมัยใหม่ (ตั้งแต่เวอร์ชัน 2.x เป็นต้นมา) เปิด `--decorate` เป็นค่าเริ่มต้นให้อัตโนมัติอยู่แล้วเมื่อรันใน terminal (ผ่านการตั้งค่า `log.decorate`) แต่การใส่ explicit ไว้ก็ไม่เสียหาย และช่วยให้แน่ใจว่าจะเห็นเสมอไม่ว่าจะตั้งค่าอะไรไว้:

```bash
git log --oneline --decorate
```

```
7e4a3f9 (HEAD -> main) เพิ่มไฟล์ README อธิบายโปรเจกต์
3c8b1a9 แก้ไข bug การคำนวณราคารวม
9f2e6d5 เพิ่มฟังก์ชันคำนวณราคาสินค้า
1a0b9c8 initial commit
```

`--decorate` คือสิ่งที่ทำให้เราเห็น `(HEAD -> main)` ต่อท้าย hash — บอกให้รู้ว่า branch ไหนกำลังชี้อยู่ที่ commit ไหน

### รวมร่าง: คำสั่งยอดฮิตที่โปรแกรมเมอร์ใช้ทุกวัน

เมื่อรวม 4 options เข้าด้วยกัน จะได้คำสั่งที่โปรแกรมเมอร์ทั่วโลกใช้กันแทบทุกวัน จนหลายคนตั้งเป็น alias (จะสอนใน Step 49):

```bash
git log --oneline --graph --all --decorate
```

**ผลลัพธ์ตัวอย่าง:**

```
* 7e4a3f9 (HEAD -> main) เพิ่มไฟล์ README อธิบายโปรเจกต์
* 3c8b1a9 แก้ไข bug การคำนวณราคารวม
* 9f2e6d5 เพิ่มฟังก์ชันคำนวณราคาสินค้า
* 1a0b9c8 initial commit
```

| Option | ทำอะไร |
|---|---|
| `--oneline` | ย่อแต่ละ commit เหลือบรรทัดเดียว (short hash + message) |
| `--graph` | วาดเส้น ASCII แสดงโครงสร้างการแตก/รวม branch |
| `--all` | แสดงทุก branch ไม่ใช่แค่ branch ปัจจุบัน |
| `--decorate` | แสดง label ของ branch/tag/HEAD ที่ชี้มาที่แต่ละ commit |

ตารางสรุป options ที่ควรจำ:

| คำสั่ง | ใช้เมื่อ |
|---|---|
| `git log` | ดูรายละเอียดเต็มของแต่ละ commit |
| `git log --oneline` | อยากเห็นภาพรวมประวัติแบบเร็ว ๆ |
| `git log --graph` | มีหลาย branch และอยากเห็นความสัมพันธ์ |
| `git log --oneline --graph --all --decorate` | คำสั่งที่ใช้บ่อยที่สุดในการทำงานจริง ควรจำให้ขึ้นใจ |

> **เกร็ดความรู้:** หลายคนจะจำคำสั่งนี้ไม่ได้ทั้งหมด จึงมักตั้งเป็น git alias เช่น `git lg` แทน — เราจะสอนวิธีทำใน Step 49 ของ Part นี้ และจะเจาะลึกเรื่อง alias ทั้งหมดอีกครั้งใน **Part 13: การตั้งค่า Git ขั้นสูงและ Aliases**

---

## Step 43: การกรอง log — `-n`, `--since`, `--until`, `--author`, `--grep`

เมื่อประวัติโปรเจกต์มีเป็นร้อยเป็นพัน commit การไล่ดูทั้งหมดไม่มีประโยชน์ Git จึงมี options สำหรับ **กรอง (filter)** ผลลัพธ์ของ `git log` ให้ตรงกับสิ่งที่เราต้องการค้นหา

### `-n <จำนวน>`: จำกัดจำนวน commit ที่แสดง

```bash
git log --oneline -n 2
```

```
7e4a3f9 เพิ่มไฟล์ README อธิบายโปรเจกต์
3c8b1a9 แก้ไข bug การคำนวณราคารวม
```

หรือเขียนแบบย่อโดยไม่ต้องมี `-n` ก็ได้ (ผลลัพธ์เหมือนกัน):

```bash
git log --oneline -2
```

### `--since` และ `--until`: กรองตามช่วงเวลา

ใช้เมื่ออยากรู้ว่า "ตั้งแต่วันที่เท่านี้มีใครทำอะไรไปบ้าง" เช่น อยากดู commit ที่เกิดขึ้นในช่วง 3 วันที่ผ่านมา:

```bash
git log --oneline --since="3 days ago"
```

```
7e4a3f9 เพิ่มไฟล์ README อธิบายโปรเจกต์
3c8b1a9 แก้ไข bug การคำนวณราคารวม
```

ระบุวันที่ชัดเจนก็ได้:

```bash
git log --oneline --since="2026-09-24" --until="2026-09-25"
```

```
3c8b1a9 แก้ไข bug การคำนวณราคารวม
9f2e6d5 เพิ่มฟังก์ชันคำนวณราคาสินค้า
```

Git เข้าใจรูปแบบวันที่แบบธรรมชาติได้หลายแบบ เช่น `"2 weeks ago"`, `"yesterday"`, `"2026-01-01"`, `"last monday"`

### `--author`: กรองตามผู้เขียน commit

เมื่อทำงานเป็นทีม บ่อยครั้งที่อยากรู้ว่า "คนคนนี้ทำอะไรไปบ้าง" ใช้ `--author` พร้อม pattern (รองรับ regular expression บางส่วน) ของชื่อหรืออีเมล:

```bash
git log --oneline --author="Nattapong"
```

```
7e4a3f9 เพิ่มไฟล์ README อธิบายโปรเจกต์
3c8b1a9 แก้ไข bug การคำนวณราคารวม
9f2e6d5 เพิ่มฟังก์ชันคำนวณราคาสินค้า
1a0b9c8 initial commit
```

ถ้ามีหลายคนในทีม เช่นมี commit จาก `Sarawut` ปนอยู่ด้วย การกรองแบบนี้จะช่วยแยกงานของแต่ละคนได้ทันที:

```bash
git log --oneline --author="Sarawut"
```

```
b4c3d2e ปรับปรุง CSS ของหน้า login
```

### `--grep`: ค้นหาจากข้อความใน commit message

ใช้เมื่อจำได้ว่า commit message มีคำว่าอะไร แต่จำไม่ได้ว่าเป็น commit ไหน:

```bash
git log --oneline --grep="bug"
```

```
3c8b1a9 แก้ไข bug การคำนวณราคารวม
```

`--grep` รองรับ regular expression เช่นกัน และสามารถใช้ `-i` ควบคู่เพื่อค้นหาแบบไม่สนตัวพิมพ์เล็ก-ใหญ่:

```bash
git log --oneline --grep="README" -i
```

### รวมหลาย filter เข้าด้วยกัน

จุดแข็งของ options เหล่านี้คือ **ใช้ร่วมกันได้** เช่น อยากรู้ว่า "Nattapong แก้ไขอะไรเกี่ยวกับ bug บ้างในสัปดาห์ที่ผ่านมา":

```bash
git log --oneline --author="Nattapong" --grep="bug" --since="1 week ago"
```

```
3c8b1a9 แก้ไข bug การคำนวณราคารวม
```

### ตารางสรุป options การกรอง

| Option | ใช้ทำอะไร | ตัวอย่าง |
|---|---|---|
| `-n <จำนวน>` หรือ `-<จำนวน>` | จำกัดจำนวน commit ที่แสดง | `git log -5` |
| `--since="<วันที่>"` | แสดงเฉพาะ commit ตั้งแต่วันที่ที่กำหนด | `--since="2026-01-01"` |
| `--until="<วันที่>"` | แสดงเฉพาะ commit ก่อนวันที่ที่กำหนด | `--until="yesterday"` |
| `--author="<pattern>"` | กรองตามผู้เขียน | `--author="Nattapong"` |
| `--grep="<pattern>"` | ค้นหาจากข้อความใน commit message | `--grep="fix"` |

> **เทคนิค:** ถ้าอยากค้นหาจาก **เนื้อหาโค้ดที่เปลี่ยนแปลง** (ไม่ใช่แค่ commit message) มี option ชื่อ `-S"<ข้อความ>"` (เรียกว่า pickaxe) ที่จะค้นว่า commit ไหนเพิ่มหรือลบข้อความนั้นในโค้ด เราจะเจาะลึกเรื่องนี้อีกครั้งใน Part ที่ว่าด้วยการค้นหาและ debugging ขั้นสูง

---

## Step 44: `git diff` — เข้าใจว่า diff เทียบอะไรกับอะไร

`git log` บอกเราว่า "มี commit อะไรบ้าง" แต่ยังไม่บอกรายละเอียดว่า **แต่ละ commit เปลี่ยนแปลงโค้ดตรงไหนบ้าง** นี่คือหน้าที่ของ `git diff`

### ความสับสนที่พบบ่อยที่สุดเรื่อง `git diff`

มือใหม่มักสับสนว่า `git diff` เทียบไฟล์กับอะไร เพราะคำสั่งนี้มีได้หลายโหมดขึ้นอยู่กับ option ที่ใส่ แต่ก่อนจะไปถึงตรงนั้น เรามาเข้าใจ **โหมด default (ไม่ใส่ option ใด ๆ)** ก่อน

ทบทวนจาก Part 03: Git มี 3 พื้นที่หลักที่ไฟล์เดินทางผ่าน:

```
Working Directory  --git add-->  Staging Area (Index)  --git commit-->  Repository (commit history)
```

เมื่อพิมพ์ `git diff` เฉย ๆ โดยไม่มี option ใด ๆ ต่อท้าย:

```bash
git diff
```

Git จะเปรียบเทียบ **"Working Directory" กับ "Staging Area"** — พูดง่าย ๆ คือ **แสดงเฉพาะการเปลี่ยนแปลงที่คุณยังไม่ได้ `git add`**

### ทดลองจริง

สมมติเราแก้ไขไฟล์ `price.js` เพิ่มบรรทัดใหม่โดยยังไม่ได้ `git add`:

```bash
echo "// TODO: เพิ่ม validation ราคาติดลบ" >> price.js
git diff
```

**ผลลัพธ์:**

```diff
diff --git a/price.js b/price.js
index 3f8a1c2..9b2e4d1 100644
--- a/price.js
+++ b/price.js
@@ -10,3 +10,4 @@ function calculateTotal(price, quantity) {
   return price * quantity;
 }
+// TODO: เพิ่ม validation ราคาติดลบ
```

### อ่านผลลัพธ์ diff ทีละส่วน

| ส่วนของผลลัพธ์ | ความหมาย |
|---|---|
| `diff --git a/price.js b/price.js` | บอกว่ากำลังเทียบไฟล์ `price.js` เวอร์ชัน `a` (ก่อน) กับ `b` (หลัง) |
| `index 3f8a1c2..9b2e4d1 100644` | hash ของ blob object ก่อนและหลัง (จะเข้าใจลึกใน Part 56 เรื่อง Internals), `100644` คือ file permission mode |
| `--- a/price.js` | สัญลักษณ์ของไฟล์ก่อนเปลี่ยนแปลง |
| `+++ b/price.js` | สัญลักษณ์ของไฟล์หลังเปลี่ยนแปลง |
| `@@ -10,3 +10,4 @@` | **hunk header** บอกตำแหน่งบรรทัด — `-10,3` หมายถึงไฟล์เก่าเริ่มบรรทัด 10 แสดง 3 บรรทัด, `+10,4` หมายถึงไฟล์ใหม่เริ่มบรรทัด 10 แสดง 4 บรรทัด |
| บรรทัดที่ขึ้นต้นด้วย `+` (สีเขียวใน terminal) | บรรทัดที่ **เพิ่มเข้ามาใหม่** |
| บรรทัดที่ขึ้นต้นด้วย `-` (สีแดงใน terminal) | บรรทัดที่ **ถูกลบออกไป** |
| บรรทัดที่ไม่มีเครื่องหมายนำหน้า | บรรทัด **context** ที่ไม่เปลี่ยนแปลง แสดงไว้เพื่อให้เห็นบริบทรอบ ๆ |

### ถ้า `git add` ไปแล้ว `git diff` ธรรมดาจะไม่แสดงอะไร

นี่คือจุดที่มือใหม่งงบ่อยที่สุด — ลอง `git add` ไฟล์ที่แก้ไปแล้วรัน `git diff` อีกครั้ง:

```bash
git add price.js
git diff
```

**ผลลัพธ์:** (ไม่มีอะไรแสดงเลย — ว่างเปล่า)

ทำไมถึงว่างเปล่า? เพราะตอนนี้ **Working Directory กับ Staging Area เหมือนกันทุกประการแล้ว** (เพราะเราเพิ่ง add เข้าไป) `git diff` ธรรมดาจึงไม่มีอะไรให้เทียบต่าง

คำถามคือ แล้วถ้าอยากดู diff ของสิ่งที่ **staged ไว้แล้ว** (เทียบกับ commit ล่าสุด) ต้องทำอย่างไร? นี่คือสิ่งที่ Step 45 จะตอบ

### สรุปโหมด default ของ `git diff`

| คำสั่ง | เทียบอะไรกับอะไร |
|---|---|
| `git diff` (ไม่มี option) | Working Directory เทียบกับ Staging Area (แสดงเฉพาะสิ่งที่ยังไม่ได้ add) |

---

## Step 45: `git diff HEAD` และ `git diff --staged`/`--cached`

จาก Step 44 เราเข้าใจแล้วว่า `git diff` เฉย ๆ เทียบ Working Directory กับ Staging Area ทีนี้มาดู 2 คำสั่งที่สำคัญไม่แพ้กัน

### ไดอะแกรมสรุป 3 พื้นที่และคำสั่ง diff ที่เกี่ยวข้อง

```
┌───────────────────┐   git add    ┌───────────────────┐   git commit   ┌───────────────────┐
│ Working Directory  │─────────────▶│  Staging Area      │───────────────▶│  Repository        │
│ (ไฟล์จริงบนดิสก์)   │              │  (Index)            │                │  (HEAD / commit    │
│                    │              │                    │                │   ล่าสุด)          │
└───────────────────┘              └───────────────────┘                └───────────────────┘
        │                                    │                                     │
        │◀──────────── git diff ────────────▶│                                     │
        │            (WD vs Staging)         │                                     │
        │                                    │                                     │
        │◀──────────────── git diff --staged / --cached ─────────────────────────▶│
        │                                    │        (Staging vs HEAD)            │
        │                                    │                                     │
        │◀────────────────────────── git diff HEAD ────────────────────────────────▶│
                              (Working Directory vs HEAD — เทียบทั้ง 2 การเปลี่ยนแปลงรวมกัน)
```

### `git diff HEAD`: เทียบ Working Directory กับ commit ล่าสุด

`git diff HEAD` จะแสดง **ทุกการเปลี่ยนแปลงทั้งหมด** ไม่ว่าจะ add ไปแล้วหรือยัง โดยเทียบ Working Directory ปัจจุบันกับ snapshot ของ commit ล่าสุด (`HEAD`)

```bash
git diff HEAD
```

**ผลลัพธ์ (ต่อจากตัวอย่างก่อนหน้าที่ `git add price.js` ไปแล้ว):**

```diff
diff --git a/price.js b/price.js
index 3f8a1c2..9b2e4d1 100644
--- a/price.js
+++ b/price.js
@@ -10,3 +10,4 @@ function calculateTotal(price, quantity) {
   return price * quantity;
 }
+// TODO: เพิ่ม validation ราคาติดลบ
```

สังเกตว่า **แสดงผลเหมือนกับตอนที่ยังไม่ add** เพราะ `git diff HEAD` ไม่สนใจว่าไฟล์อยู่ในสถานะ staged หรือยัง มันมองข้ามพื้นที่ Staging Area ไปเลย แล้วเทียบตรงจาก Working Directory กับ commit ล่าสุด

### `git diff --staged` (หรือ `git diff --cached`): เทียบ Staging Area กับ commit ล่าสุด

ทั้งสองชื่อนี้ **เป็นคำสั่งเดียวกันทุกประการ** เพียงแค่ตั้งชื่อไว้ 2 แบบให้เลือกใช้ตามความถนัด (`--cached` มีมาก่อน, `--staged` เพิ่มมาทีหลังเพราะเข้าใจง่ายกว่า)

```bash
git diff --staged
```

หรือ

```bash
git diff --cached
```

**ผลลัพธ์:**

```diff
diff --git a/price.js b/price.js
index 3f8a1c2..9b2e4d1 100644
--- a/price.js
+++ b/price.js
@@ -10,3 +10,4 @@ function calculateTotal(price, quantity) {
   return price * quantity;
 }
+// TODO: เพิ่ม validation ราคาติดลบ
```

ในตัวอย่างนี้ผลลัพธ์เหมือนกับ `git diff HEAD` เพราะเรา add ไฟล์ไปหมดแล้วและไม่ได้แก้ไขอะไรเพิ่มหลังจาก add แต่ **ในสถานการณ์ที่ซับซ้อนกว่านี้ ผลลัพธ์จะต่างกัน**

### ตัวอย่างที่เห็นความแตกต่างชัดเจน

สมมติสถานการณ์นี้:

1. แก้ไข `price.js` เพิ่มบรรทัด A
2. `git add price.js` (บรรทัด A ถูก staged แล้ว)
3. แก้ไข `price.js` เพิ่มบรรทัด B อีกครั้ง (บรรทัด B **ยังไม่ได้** add)

```bash
echo "// เพิ่มบรรทัด A" >> price.js
git add price.js
echo "// เพิ่มบรรทัด B" >> price.js
```

ตอนนี้สถานะไฟล์คือ:
- **Repository (HEAD):** ไม่มีบรรทัด A หรือ B
- **Staging Area:** มีบรรทัด A เท่านั้น
- **Working Directory:** มีทั้งบรรทัด A และ B

ลองรันคำสั่งทั้ง 3 แบบเปรียบเทียบกัน:

```bash
git diff
```
```diff
diff --git a/price.js b/price.js
index 9b2e4d1..a7c3f5e 100644
--- a/price.js
+++ b/price.js
@@ -11,3 +11,4 @@ function calculateTotal(price, quantity) {
 }
 // เพิ่มบรรทัด A
+// เพิ่มบรรทัด B
```
→ เห็นเฉพาะบรรทัด **B** (เพราะเทียบ Staging vs Working Directory — บรรทัด A อยู่ใน staging แล้วจึงไม่ต่างกัน)

```bash
git diff --staged
```
```diff
diff --git a/price.js b/price.js
index 3f8a1c2..9b2e4d1 100644
--- a/price.js
+++ b/price.js
@@ -10,3 +10,4 @@ function calculateTotal(price, quantity) {
   return price * quantity;
 }
+// เพิ่มบรรทัด A
```
→ เห็นเฉพาะบรรทัด **A** (เพราะเทียบ HEAD vs Staging — บรรทัด A ถูก staged ไปแล้วแต่ยังไม่ commit)

```bash
git diff HEAD
```
```diff
diff --git a/price.js b/price.js
index 3f8a1c2..a7c3f5e 100644
--- a/price.js
+++ b/price.js
@@ -10,3 +10,5 @@ function calculateTotal(price, quantity) {
   return price * quantity;
 }
+// เพิ่มบรรทัด A
+// เพิ่มบรรทัด B
```
→ เห็น **ทั้ง A และ B** (เพราะเทียบ HEAD vs Working Directory ตรง ๆ ครอบคลุมทุกการเปลี่ยนแปลง)

### ตารางสรุปที่ต้องจำให้ขึ้นใจ

| คำสั่ง | เทียบอะไร กับอะไร | ใช้เมื่อ |
|---|---|---|
| `git diff` | Working Directory ↔ Staging Area | อยากรู้ว่า "อะไรที่ยังไม่ได้ add" |
| `git diff --staged` / `git diff --cached` | Staging Area ↔ HEAD (commit ล่าสุด) | อยากรู้ว่า "ถ้า commit ตอนนี้เลย จะมีอะไรถูกบันทึกบ้าง" |
| `git diff HEAD` | Working Directory ↔ HEAD (commit ล่าสุด) | อยากรู้ **ทุกอย่าง** ที่เปลี่ยนไปจาก commit ล่าสุด ไม่สนว่า add หรือยัง |

> **เทคนิคที่ใช้บ่อยมากในการทำงานจริง:** ก่อน `git commit` ทุกครั้ง ควรรัน `git diff --staged` เพื่อตรวจทานอีกรอบว่าสิ่งที่กำลังจะ commit ถูกต้องครบถ้วนตามที่ตั้งใจ ก่อนกด commit จริง

---

## Step 46: `git show <commit>` — ดูรายละเอียด commit เดียวแบบเต็ม

`git log` และ `git diff` ตอบคำถามคนละแบบ — `git log` บอกภาพรวมของหลาย commit, `git diff` เทียบระหว่าง 2 สถานะ ส่วน `git show` ตอบคำถามที่เฉพาะเจาะจงกว่า: **"commit นี้ commit เดียว มีอะไรอยู่ข้างในบ้าง"**

### `git show` พื้นฐาน

```bash
git show 3c8b1a9
```

**ผลลัพธ์:**

```
commit 3c8b1a9f6e2d4c7b5a0f9e8d7c6b5a4f3e2d1c0b
Author: Nattapong S. <nattapong@example.com>
Date:   Thu Sep 25 10:15:44 2026 +0700

    แก้ไข bug การคำนวณราคารวม

diff --git a/price.js b/price.js
index 8a1c2f3..3f8a1c2 100644
--- a/price.js
+++ b/price.js
@@ -8,5 +8,5 @@ function calculateTotal(price, quantity) {
-  return price + quantity;
+  return price * quantity;
 }
```

สังเกตว่าผลลัพธ์นี้คือการรวม **2 อย่างเข้าด้วยกัน**:

1. **ส่วนบน** — metadata ของ commit (เหมือนที่เห็นใน `git log`): hash, author, date, message
2. **ส่วนล่าง** — diff ที่แสดงว่า commit นี้เปลี่ยนแปลงอะไรไปจาก parent ของมัน (commit ก่อนหน้าโดยตรง)

พูดง่าย ๆ: `git show <commit>` ≈ `git log -1 <commit>` + diff ของ commit นั้นเทียบกับ parent

### ไม่ระบุ commit = ดู HEAD

ถ้าไม่ใส่ argument ใด ๆ `git show` จะแสดง commit ล่าสุด (`HEAD`) โดยอัตโนมัติ:

```bash
git show
```

ผลลัพธ์จะเหมือนกับ `git show HEAD`

### `git show` กับไฟล์เฉพาะ ณ commit หนึ่ง

`git show` ยังใช้ **ดูเนื้อหาไฟล์ทั้งไฟล์ ณ commit หนึ่ง ๆ** ได้ ด้วย syntax `<commit>:<path/to/file>`:

```bash
git show HEAD~2:price.js
```

**ผลลัพธ์:** (แสดงเนื้อหาทั้งไฟล์ `price.js` ตามที่มันเป็นอยู่ ณ commit ที่ 2 ก่อนหน้า `HEAD` ทันที ไม่ใช่ diff)

```javascript
function calculateTotal(price, quantity) {
  // TODO: validate input
  return price + quantity;
}
```

นี่มีประโยชน์มากเมื่อต้องการดูว่า "ไฟล์นี้เมื่อ 2 commit ที่แล้วหน้าตาเป็นอย่างไร" โดยไม่ต้องย้อนกลับไป checkout commit นั้นจริง ๆ (การ checkout ย้อนกลับจะเรียนใน Part ถัดไปเรื่อง branching และ Part เรื่อง undo)

### `git show` กับ hash เต็มหรือย่อก็ได้

ทั้งสองแบบใช้ได้เหมือนกัน:

```bash
git show 3c8b1a9
git show 3c8b1a9f6e2d4c7b5a0f9e8d7c6b5a4f3e2d1c0b
```

### ตารางสรุปการใช้งาน `git show`

| คำสั่ง | ทำอะไร |
|---|---|
| `git show` | แสดงรายละเอียดเต็มของ `HEAD` (metadata + diff) |
| `git show <hash>` | แสดงรายละเอียดเต็มของ commit ที่ระบุ |
| `git show <commit>:<path>` | แสดงเนื้อหาทั้งไฟล์ ณ commit นั้น (ไม่ใช่ diff) |
| `git show --stat <commit>` | แสดงเฉพาะสรุปไฟล์ที่เปลี่ยน ไม่แสดง diff เต็ม (จะอธิบายละเอียดใน Step 48) |

---

## Step 47: การอ้างอิง commit ในรูปแบบต่างๆ — hash, HEAD, `~`, `^`

ก่อนหน้านี้เราใช้ทั้ง full hash, short hash, และเห็นคำว่า `HEAD` ผ่านตามาหลายครั้งแล้ว ถึงเวลาทำความเข้าใจให้ชัดเจนว่า Git มี "วิธีอ้างอิงถึง commit" (เรียกรวมว่า **revision** หรือ **commit-ish**) กี่แบบ และแบบไหนใช้เมื่อไหร่

### 1. Full hash (SHA-1 เต็ม 40 ตัวอักษร)

```bash
git show 7e4a3f9c1d2b8e6a5f0c3d9e8b7a6c5d4e3f2a1b
```

แม่นยำที่สุด ไม่มีทางซ้ำกับ commit อื่น แต่พิมพ์ยากและจำไม่ได้เลยในทางปฏิบัติ

### 2. Short hash (ย่อ 7 ตัวอักษรขึ้นไป)

```bash
git show 7e4a3f9
```

Git จะขยาย short hash ให้ยาวพอที่จะไม่ซ้ำกับ commit อื่นในโปรเจกต์โดยอัตโนมัติ (ปกติ 7 ตัวก็เพียงพอสำหรับโปรเจกต์ทั่วไปที่มีไม่กี่พัน commit)

### 3. `HEAD` — ตัวชี้ไปยังตำแหน่งปัจจุบัน

`HEAD` คือคำที่ Git สงวนไว้เป็นพิเศษ หมายถึง **"commit ที่คุณกำลังยืนอยู่ตอนนี้"** ปกติแล้วมันจะชี้ไปที่ commit ล่าสุดของ branch ที่คุณอยู่

```bash
git show HEAD
```

เทียบเท่ากับการอ้าง commit ล่าสุด (`7e4a3f9` ในตัวอย่างของเรา) ทุกประการ

### 4. `HEAD~<n>` — ย้อนกลับไป n commit ตามสาย parent เส้นแรก

เครื่องหมาย tilde (`~`) ใช้สำหรับ **นับถอยหลังจาก commit ปัจจุบันไปตามลำดับเชิงเส้น**

```
1a0b9c8 (initial commit)
    ↑
9f2e6d5
    ↑
3c8b1a9
    ↑
7e4a3f9 (HEAD -> main)
```

| Reference | หมายถึง commit อะไร |
|---|---|
| `HEAD` | `7e4a3f9` |
| `HEAD~1` (หรือ `HEAD~`) | `3c8b1a9` (1 ก้อนก่อนหน้า) |
| `HEAD~2` | `9f2e6d5` (2 ก้อนก่อนหน้า) |
| `HEAD~3` | `1a0b9c8` (3 ก้อนก่อนหน้า, คือ initial commit) |

ทดลอง:

```bash
git show HEAD~1 --stat
```

```
commit 3c8b1a9f6e2d4c7b5a0f9e8d7c6b5a4f3e2d1c0b
Author: Nattapong S. <nattapong@example.com>
Date:   Thu Sep 25 10:15:44 2026 +0700

    แก้ไข bug การคำนวณราคารวม

 price.js | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

### 5. `HEAD^` และ `HEAD^^` — เครื่องหมาย caret

Caret (`^`) มีความหมายคล้าย `~` มาก และในกรณีที่มี parent เดียว (ไม่ใช่ merge commit) `HEAD^` กับ `HEAD~1` **หมายถึง commit เดียวกันทุกประการ**:

```bash
git show HEAD^ --stat
git show HEAD~1 --stat
```

ทั้งสองคำสั่งนี้จะให้ผลลัพธ์เหมือนกันเป๊ะในตัวอย่างของเรา และ `HEAD^^` ก็เทียบเท่า `HEAD~2`:

```bash
git show HEAD^^ --stat
git show HEAD~2 --stat
```

### แล้ว `~` กับ `^` ต่างกันตรงไหน?

ความแตกต่างจะปรากฏชัดเจน **เมื่อ commit นั้นเป็น merge commit ที่มี parent มากกว่า 1 ตัว** (จากการ merge 2 branch เข้าด้วยกัน ซึ่งจะเรียนละเอียดใน Part ถัดไป):

- **`~N`** จะเดินตาม **parent ตัวแรกเสมอ** ไล่ไปทีละ N ขั้น (เหมือนเดินตามเส้นเวลาหลักตรง ๆ)
- **`^N`** จะเลือก **parent ตัวที่ N** ของ merge commit นั้น (เพราะ merge commit มี parent ได้มากกว่า 1 ตัว)

ตัวอย่างเช่น ถ้า `M` คือ merge commit ที่มี parent 2 ตัวคือ `P1` (จาก main) และ `P2` (จาก feature branch ที่ถูก merge เข้ามา):

| Reference | หมายถึง |
|---|---|
| `M^1` หรือ `M^` | `P1` (parent ตัวแรก) |
| `M^2` | `P2` (parent ตัวที่สอง) |
| `M~1` | `P1` (เหมือน `M^1` เสมอ เพราะ `~` เดินตาม parent แรกเท่านั้น) |
| `M~2` | parent ของ `P1` (เดินต่อไปอีกขั้นตามสายหลัก ไม่ใช่ `P2`) |

> **สรุปสั้น ๆ ที่ต้องจำ:** ในกรณีทั่วไปที่ไม่มี merge commit `~` กับ `^` ใช้แทนกันได้เลย แต่เมื่อไหร่ที่เจอ merge commit (parent หลายตัว) ต้องระวัง — `^N` เลือกว่าจะไปทาง parent ไหน ส่วน `~N` เดินตามสายหลักเรื่อย ๆ เสมอ เราจะได้เห็นตัวอย่างจริงเรื่องนี้อีกครั้งเมื่อเรียน branching และ merge ใน Part 07 เป็นต้นไป

### ตารางสรุปวิธีอ้างอิง commit ทั้งหมด

| รูปแบบ | ตัวอย่าง | ความหมาย |
|---|---|---|
| Full hash | `7e4a3f9c1d2b8e6a5f0c3d9e8b7a6c5d4e3f2a1b` | อ้างอิงแม่นยำที่สุด |
| Short hash | `7e4a3f9` | ย่อจาก full hash ให้พิมพ์ง่ายขึ้น |
| `HEAD` | `HEAD` | commit ปัจจุบันที่ยืนอยู่ |
| `HEAD~n` | `HEAD~2` | ย้อนกลับ n ก้อนตามสายหลัก |
| `HEAD^` | `HEAD^` | parent ตัวแรก (เท่ากับ `HEAD~1`) |
| `HEAD^n` | `HEAD^2` | parent ตัวที่ n (ใช้กับ merge commit เท่านั้น) |
| `HEAD^^` | `HEAD^^` | parent ของ parent (เท่ากับ `HEAD~2` ในกรณีไม่มี merge) |
| ชื่อ branch | `main` | commit ล่าสุดของ branch นั้น |

---

## Step 48: `git log -p` และ `git log --stat`

เรารู้จัก `git show` สำหรับดู commit เดียวแบบละเอียดแล้ว แต่ถ้าอยากดู **diff ของหลาย ๆ commit ต่อเนื่องกัน** โดยไม่ต้องรัน `git show` ทีละอันล่ะ? นี่คือหน้าที่ของ `git log -p` และ `git log --stat`

### `git log -p`: แสดง diff เต็มของทุก commit

Flag `-p` (ย่อจาก "patch") จะทำให้ `git log` แสดง diff แบบเต็มของแต่ละ commit ต่อจากส่วน metadata ไปเรื่อย ๆ เหมือนเอา `git show` มาต่อกันหลาย ๆ อัน:

```bash
git log -p
```

**ผลลัพธ์ตัวอย่าง (แสดงบางส่วน):**

```
commit 7e4a3f9c1d2b8e6a5f0c3d9e8b7a6c5d4e3f2a1b (HEAD -> main)
Author: Nattapong S. <nattapong@example.com>
Date:   Fri Sep 26 14:32:10 2026 +0700

    เพิ่มไฟล์ README อธิบายโปรเจกต์

diff --git a/README.md b/README.md
new file mode 100644
index 0000000..b4c3d2e
--- /dev/null
+++ b/README.md
@@ -0,0 +1,3 @@
+# My Project
+
+โปรเจกต์ฝึกฝนสำหรับหลักสูตร Git

commit 3c8b1a9f6e2d4c7b5a0f9e8d7c6b5a4f3e2d1c0b
Author: Nattapong S. <nattapong@example.com>
Date:   Thu Sep 25 10:15:44 2026 +0700

    แก้ไข bug การคำนวณราคารวม

diff --git a/price.js b/price.js
index 8a1c2f3..3f8a1c2 100644
--- a/price.js
+++ b/price.js
@@ -8,5 +8,5 @@ function calculateTotal(price, quantity) {
-  return price + quantity;
+  return price * quantity;
 }

commit 9f2e6d5c4b3a2f1e0d9c8b7a6f5e4d3c2b1a0f9e
...
```

`git log -p` มีประโยชน์มากเมื่ออยากไล่อ่านว่า "โค้ดค่อย ๆ วิวัฒนาการมาอย่างไรตั้งแต่ต้น" แต่ข้อเสียคือผลลัพธ์จะยาวมากถ้าประวัติมีเยอะ ควรใช้ร่วมกับ `-n` เพื่อจำกัดจำนวน:

```bash
git log -p -2
```

จะแสดง diff เฉพาะ 2 commit ล่าสุดเท่านั้น

### `git log --stat`: สรุปไฟล์ที่เปลี่ยนพร้อมจำนวนบรรทัด +/-

ถ้าไม่ต้องการเห็น diff แบบละเอียดทุกบรรทัด แต่อยากเห็น **สรุปว่าแต่ละ commit แตะไฟล์ไหนบ้าง เปลี่ยนกี่บรรทัด** ให้ใช้ `--stat` แทน:

```bash
git log --stat
```

**ผลลัพธ์ตัวอย่าง:**

```
commit 7e4a3f9c1d2b8e6a5f0c3d9e8b7a6c5d4e3f2a1b (HEAD -> main)
Author: Nattapong S. <nattapong@example.com>
Date:   Fri Sep 26 14:32:10 2026 +0700

    เพิ่มไฟล์ README อธิบายโปรเจกต์

 README.md | 3 +++
 1 file changed, 3 insertions(+)

commit 3c8b1a9f6e2d4c7b5a0f9e8d7c6b5a4f3e2d1c0b
Author: Nattapong S. <nattapong@example.com>
Date:   Thu Sep 25 10:15:44 2026 +0700

    แก้ไข bug การคำนวณราคารวม

 price.js | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

commit 9f2e6d5c4b3a2f1e0d9c8b7a6f5e4d3c2b1a0f9e
Author: Nattapong S. <nattapong@example.com>
Date:   Wed Sep 24 09:02:31 2026 +0700

    เพิ่มฟังก์ชันคำนวณราคาสินค้า

 price.js | 12 ++++++++++++
 1 file changed, 12 insertions(+)

commit 1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5e4f3a2b
Author: Nattapong S. <nattapong@example.com>
Date:   Tue Sep 23 20:11:05 2026 +0700

    initial commit

 .gitignore | 2 ++
 price.js   | 1 +
 2 files changed, 3 insertions(+)
```

### อ่านผลลัพธ์ `--stat` ทีละส่วน

| ส่วนของผลลัพธ์ | ความหมาย |
|---|---|
| `price.js \| 2 +-` | ไฟล์ `price.js` มีการเปลี่ยนแปลงรวม 2 บรรทัด, สัญลักษณ์ `+` และ `-` แสดงสัดส่วนคร่าว ๆ ระหว่างบรรทัดที่เพิ่มกับลบ |
| `1 file changed, 1 insertion(+), 1 deletion(-)` | สรุปว่า commit นี้แตะ 1 ไฟล์, เพิ่ม 1 บรรทัด (insertion), ลบ 1 บรรทัด (deletion) |
| `README.md \| 3 +++` | ไฟล์ใหม่ที่เพิ่มเข้ามา 3 บรรทัด (มีแต่ `+` เพราะเป็นไฟล์ใหม่ทั้งหมด) |

### รวมกับ `--oneline` เพื่อความกระชับ

สามารถผสม `--stat` กับ `--oneline` เพื่อให้อ่านง่ายขึ้นได้เช่นกัน แต่ต้องระวังว่าผลลัพธ์จะดูแปลกเล็กน้อยเพราะ `--stat` ทำงานเป็น block ต่อ commit อยู่แล้ว วิธีที่นิยมกว่าคือใช้ `--stat` คู่กับการจำกัดจำนวน commit:

```bash
git log --stat -3
```

### เปรียบเทียบ `-p` กับ `--stat`

| Option | แสดงอะไร | เหมาะกับ |
|---|---|---|
| `git log -p` | diff แบบเต็มทุกบรรทัดของทุก commit | อยากไล่อ่านโค้ดที่เปลี่ยนแปลงแบบละเอียด (code review ย้อนหลัง) |
| `git log --stat` | สรุปไฟล์ที่เปลี่ยน + จำนวนบรรทัด +/- โดยไม่ลงรายละเอียด | อยากรู้ภาพรวมว่า commit ไหนแตะไฟล์อะไรบ้าง โดยไม่อยากเห็น diff เต็ม |

---

## Step 49: ปรับแต่งรูปแบบ log ด้วย `--pretty=format` และการทำ alias

จนถึงตอนนี้เราใช้รูปแบบสำเร็จรูปของ `git log` มาตลอด (`--oneline`, `--graph` ฯลฯ) แต่ Git ยังให้เรา **ออกแบบรูปแบบการแสดงผลของตัวเองได้อย่างละเอียด** ผ่าน `--pretty=format`

### `--pretty=format:"..."` พื้นฐาน

```bash
git log --pretty=format:"%h - %an, %ar : %s"
```

**ผลลัพธ์:**

```
7e4a3f9 - Nattapong S., 2 hours ago : เพิ่มไฟล์ README อธิบายโปรเจกต์
3c8b1a9 - Nattapong S., 1 day ago : แก้ไข bug การคำนวณราคารวม
9f2e6d5 - Nattapong S., 2 days ago : เพิ่มฟังก์ชันคำนวณราคาสินค้า
1a0b9c8 - Nattapong S., 3 days ago : initial commit
```

### ตัวย่อ (placeholders) ที่ใช้บ่อยใน `--pretty=format`

| Placeholder | ความหมาย |
|---|---|
| `%H` | Commit hash แบบเต็ม |
| `%h` | Commit hash แบบย่อ |
| `%an` | ชื่อผู้เขียน (Author name) |
| `%ae` | อีเมลผู้เขียน (Author email) |
| `%ad` | วันที่เขียน (Author date) — ใช้ร่วมกับ `--date=` เพื่อกำหนดรูปแบบวันที่ |
| `%ar` | วันที่แบบ relative (เช่น "2 hours ago") |
| `%s` | หัวข้อของ commit message (subject) |
| `%b` | เนื้อหาส่วนที่เหลือของ commit message (body) |
| `%d` | ชื่อ ref ที่ตกแต่ง (เหมือน `--decorate`) |
| `%C(สี)` / `%Creset` | กำหนดสีของข้อความ (เช่น `%C(yellow)`, `%C(red)`, `%Creset` เพื่อรีเซ็ตสี) |

### ตัวอย่างที่ใช้งานได้จริงและนิยมมาก

```bash
git log --pretty=format:"%C(yellow)%h%Creset -%C(red)%d%Creset %s %C(green)(%ar) %C(blue)<%an>%Creset" --graph
```

**ผลลัพธ์ (มีสีจริงเมื่อแสดงใน terminal):**

```
* 7e4a3f9 -(HEAD -> main) เพิ่มไฟล์ README อธิบายโปรเจกต์ (2 hours ago) <Nattapong S.>
* 3c8b1a9 - แก้ไข bug การคำนวณราคารวม (1 day ago) <Nattapong S.>
* 9f2e6d5 - เพิ่มฟังก์ชันคำนวณราคาสินค้า (2 days ago) <Nattapong S.>
* 1a0b9c8 - initial commit (3 days ago) <Nattapong S.>
```

คำสั่งนี้ยาวและซับซ้อนเกินกว่าจะพิมพ์ทุกครั้ง — นี่คือเหตุผลว่าทำไมเราต้องมี **git alias**

### การทำ git alias เพื่อเรียกใช้ซ้ำ

Git alias คือการตั้ง "ชื่อเล่น" ให้กับคำสั่งยาว ๆ เพื่อพิมพ์สั้นลง ตั้งค่าผ่าน `git config`:

```bash
git config --global alias.lg "log --oneline --graph --all --decorate"
```

หลังจากนี้ แทนที่จะพิมพ์คำสั่งยาว ๆ จาก Step 42 ทุกครั้ง เราสามารถพิมพ์แค่:

```bash
git lg
```

ก็จะได้ผลลัพธ์เดียวกันกับ `git log --oneline --graph --all --decorate` ทันที

ลองตั้ง alias สำหรับ pretty format ที่ซับซ้อนด้วยเช่นกัน:

```bash
git config --global alias.lg2 "log --pretty=format:'%C(yellow)%h%Creset -%C(red)%d%Creset %s %C(green)(%ar) %C(blue)<%an>%Creset' --graph"
```

ทดสอบ:

```bash
git lg2
```

### ตรวจสอบ alias ที่ตั้งไว้ทั้งหมด

```bash
git config --get-regexp alias
```

**ผลลัพธ์:**

```
alias.lg log --oneline --graph --all --decorate
alias.lg2 log --pretty=format:'%C(yellow)%h%Creset -%C(red)%d%Creset %s %C(green)(%ar) %C(blue)<%an>%Creset' --graph
```

### alias ถูกเก็บไว้ที่ไหน

alias ที่ตั้งด้วย `--global` จะถูกเก็บไว้ในไฟล์ config ส่วนตัวของผู้ใช้ (เช่น `~/.gitconfig` บน macOS/Linux) ลองเปิดดูได้:

```bash
cat ~/.gitconfig
```

```ini
[user]
	name = Nattapong S.
	email = nattapong@example.com
[alias]
	lg = log --oneline --graph --all --decorate
	lg2 = log --pretty=format:'%C(yellow)%h%Creset -%C(red)%d%Creset %s %C(green)(%ar) %C(blue)<%an>%Creset' --graph
```

> **เชื่อมโยงไปข้างหน้า:** เราเพิ่งแตะเรื่อง alias เป็นน้ำจิ้มเท่านั้น เพื่อให้เห็นประโยชน์ทันทีในบริบทของ `git log` ใน **Part 13: การตั้งค่า Git ขั้นสูงและ Aliases** เราจะเรียนเรื่อง alias แบบเต็มรูปแบบ ทั้งการตั้ง alias สำหรับคำสั่งอื่น ๆ นอกจาก log (เช่น `git st` แทน `git status`, `git co` แทน `git checkout`), การตั้ง alias ที่ซับซ้อนแบบเรียก shell command ได้ (ผ่าน `!` prefix), และการจัดการไฟล์ `.gitconfig` อย่างเป็นระบบ

### ตารางสรุป placeholder ที่ควรจำ

| Placeholder | ตัวอย่างผลลัพธ์ |
|---|---|
| `%h` | `7e4a3f9` |
| `%an` | `Nattapong S.` |
| `%ar` | `2 hours ago` |
| `%s` | `เพิ่มไฟล์ README อธิบายโปรเจกต์` |
| `%d` | `(HEAD -> main)` |

---

## Step 50: แบบฝึกหัดจบเฟส 1 — วิเคราะห์ประวัติของโปรเจกต์ พร้อม Checklist สรุปจบเฟส 1

ถึง Step นี้เราได้เรียนคำสั่งพื้นฐานที่สุดของ Git ครบทั้งหมดตามเป้าหมายของเฟส 1 แล้ว มาถึงเวลาลงมือทำแบบฝึกหัดใหญ่ที่รวบยอดทุกอย่างที่เรียนมาตั้งแต่ Part 01 ถึง Part 05

### โจทย์แบบฝึกหัด

ใช้โปรเจกต์ฝึกฝนที่คุณสร้างและ commit ไว้ตั้งแต่ Part 04 (โฟลเดอร์ `~/git-course/part-04-practice` หรือโปรเจกต์ใดก็ตามที่คุณมี commit history อยู่แล้วอย่างน้อย 4–5 commit) แล้วทำภารกิจต่อไปนี้ให้ครบทุกข้อ:

**ภารกิจที่ 1 — สำรวจภาพรวมประวัติ**

```bash
cd ~/git-course/part-04-practice
git log --oneline --graph --all --decorate
```

ตอบคำถาม: โปรเจกต์นี้มีทั้งหมดกี่ commit? branch ปัจจุบันชื่ออะไร?

**ภารกิจที่ 2 — ค้นหา commit ที่ต้องการ**

```bash
git log --oneline --author="<ชื่อของคุณ>"
git log --oneline --grep="bug"
git log --oneline --since="3 days ago"
```

ตอบคำถาม: มี commit กี่อันที่คุณเป็นผู้เขียน? มี commit ไหนบ้างที่เกี่ยวกับการแก้บั๊ก?

**ภารกิจที่ 3 — ดูรายละเอียด commit แรกสุดของโปรเจกต์**

หา commit แรกสุด (เก่าที่สุด) แล้วดูรายละเอียดของมันด้วย `git show`:

```bash
git log --oneline | tail -1
git show <hash ของ commit แรก>
```

**ภารกิจที่ 4 — ทดลองสร้างการเปลี่ยนแปลงแล้วดู diff ในทุกโหมด**

```bash
echo "// บรรทัดทดสอบ" >> <ไฟล์ใดไฟล์หนึ่งในโปรเจกต์>
git diff
git add <ไฟล์นั้น>
git diff --staged
git diff HEAD
```

ตอบคำถาม: ผลลัพธ์ของ `git diff` หลัง add แตกต่างจากก่อน add อย่างไร? ทำไม `git diff --staged` กับ `git diff HEAD` ถึงให้ผลลัพธ์เหมือนกันในกรณีนี้?

**ภารกิจที่ 5 — ฝึกใช้ commit reference**

```bash
git show HEAD
git show HEAD~1
git show HEAD^
```

ตอบคำถาม: `HEAD~1` และ `HEAD^` ชี้ไปที่ commit เดียวกันหรือไม่? เพราะอะไร?

**ภารกิจที่ 6 — ดูประวัติแบบละเอียดด้วย -p และ --stat**

```bash
git log --stat -3
git log -p -1
```

**ภารกิจที่ 7 — สร้าง alias ของตัวเอง**

ตั้ง alias ชื่อ `git hist` ให้แสดงผลแบบ `--oneline --graph --all --decorate`:

```bash
git config --global alias.hist "log --oneline --graph --all --decorate"
git hist
```

### เฉลยแนวทาง (ตัวอย่างคำตอบที่ควรได้)

สมมติผลลัพธ์จากภารกิจที่ 1:

```
* 7e4a3f9 (HEAD -> main) เพิ่มไฟล์ README อธิบายโปรเจกต์
* 3c8b1a9 แก้ไข bug การคำนวณราคารวม
* 9f2e6d5 เพิ่มฟังก์ชันคำนวณราคาสินค้า
* 1a0b9c8 initial commit
```

คำตอบ: โปรเจกต์นี้มี **4 commit**, branch ปัจจุบันคือ **main** (สังเกตจาก `(HEAD -> main)`)

---

### ทบทวนภาพรวมเฟส 1 ทั้งหมด (Part 01–05)

เฟส 1 ของหลักสูตรนี้ปูพื้นฐานให้คุณตั้งแต่ศูนย์จนสามารถทำงานกับ Git คนเดียวได้อย่างมั่นใจ มาทบทวนภาพรวมทีละ Part:

| Part | หัวข้อหลัก | สิ่งที่ได้เรียนรู้ |
|---|---|---|
| **Part 01** | โลกของ Version Control และทำไมต้อง Git | เข้าใจปัญหาที่ VCS แก้, วิวัฒนาการ Local → Centralized → Distributed, กำเนิดของ Git, ความแตกต่างระหว่าง Git/GitHub/GitLab, แนวคิด Snapshot |
| **Part 02** | ติดตั้ง Git และตั้งค่าเริ่มต้น | ติดตั้ง Git บน Windows/Mac/Linux, ตั้งค่า `user.name`, `user.email`, ตั้งค่า editor เริ่มต้น |
| **Part 03** | แนวคิด 3 พื้นที่ของ Git และคำสั่งพื้นฐาน | Working Directory, Staging Area, Repository, `git init`, `git status`, `git add`, `git commit` |
| **Part 04** | การสร้างและจัดการ commit | เขียน commit message ที่ดี, `git commit -m`, `git commit --amend`, ประวัติของโปรเจกต์เริ่มก่อตัว |
| **Part 05** | การดูประวัติ: log, diff, show | `git log` ทุกรูปแบบ, `git diff` ทุกโหมด, `git show`, การอ้างอิง commit ด้วย `~`/`^`, การปรับแต่ง format และ alias |

### Checklist ทบทวนก่อนไปเฟส 2

ก่อนไปเฟส 2 (Part 06 เป็นต้นไป) ให้ตรวจสอบว่าคุณทำสิ่งเหล่านี้ได้ทั้งหมดโดยไม่ต้องเปิดเอกสารดูซ้ำ:

**ความรู้พื้นฐาน (Part 01–02):**
- [ ] อธิบายได้ว่า Version Control แก้ปัญหาอะไร
- [ ] อธิบายความแตกต่างระหว่าง Git, GitHub, GitLab ได้อย่างชัดเจน
- [ ] เข้าใจว่า Git เก็บข้อมูลแบบ Snapshot ไม่ใช่ Diff
- [ ] ติดตั้ง Git และตั้งค่า `user.name`/`user.email` เรียบร้อยแล้ว

**การทำงานกับ commit พื้นฐาน (Part 03–04):**
- [ ] อธิบายความแตกต่างของ Working Directory, Staging Area, Repository ได้
- [ ] ใช้ `git init`, `git status`, `git add`, `git commit` ได้อย่างคล่องแคล่ว
- [ ] เขียน commit message ที่สื่อความหมายชัดเจนได้
- [ ] รู้วิธีแก้ไข commit ล่าสุดด้วย `git commit --amend`

**การดูประวัติ (Part 05 — Part นี้):**
- [ ] อ่านผลลัพธ์ `git log` และเข้าใจทุกส่วนประกอบ (hash, author, date, message)
- [ ] ใช้ `git log --oneline --graph --all --decorate` ได้โดยไม่ต้องเปิดดูคำสั่ง
- [ ] กรอง log ด้วย `-n`, `--since`, `--until`, `--author`, `--grep` ได้
- [ ] เข้าใจความแตกต่างระหว่าง `git diff`, `git diff --staged`, `git diff HEAD` อย่างแม่นยำ
- [ ] ใช้ `git show` ดูรายละเอียด commit เดียว และดูไฟล์ ณ commit ใดก็ได้
- [ ] แยกความแตกต่างระหว่าง `HEAD~n` กับ `HEAD^n` ได้ (อย่างน้อยในกรณีไม่มี merge commit)
- [ ] ใช้ `git log -p` และ `git log --stat` ได้ และรู้ว่าต่างกันอย่างไร
- [ ] ตั้ง git alias ของตัวเองได้อย่างน้อย 1 ตัว

**ทักษะภาพรวมของเฟส 1:**
- [ ] สามารถสร้างโปรเจกต์ Git ใหม่ตั้งแต่ศูนย์ได้โดยไม่ต้องดูคู่มือ
- [ ] สามารถ commit งานเป็นขั้นตอนเล็ก ๆ ที่มีความหมายได้ (ไม่ commit ทุกอย่างรวมกันเป็นก้อนเดียว)
- [ ] สามารถย้อนดูประวัติของโปรเจกต์ตัวเองและอธิบายได้ว่าแต่ละ commit ทำอะไร
- [ ] พร้อมที่จะเรียนรู้เรื่อง `.gitignore`, branching และการทำงานร่วมกับผู้อื่นในเฟสถัดไป

หากติ๊กได้ครบทุกข้อ แปลว่าคุณผ่านเฟส 1 อย่างสมบูรณ์แล้ว — คุณมีพื้นฐาน Git ที่แข็งแรงพอที่จะต่อยอดไปเรียนรู้เรื่องที่ซับซ้อนขึ้นในเฟสถัดไปได้อย่างมั่นใจ

---

## สรุป Part 05

ใน Part นี้เราได้เรียนรู้ว่า:

1. `git log` คือหน้าต่างสำหรับมองย้อนประวัติของโปรเจกต์ อ่านได้ทั้ง hash, author, date และ message
2. options สำคัญของ `git log` ได้แก่ `--oneline`, `--graph`, `--all`, `--decorate` และเมื่อรวมกันเป็น `git log --oneline --graph --all --decorate` จะกลายเป็นคำสั่งที่ใช้บ่อยที่สุดในการทำงานจริง
3. การกรอง log ทำได้ด้วย `-n`, `--since`, `--until`, `--author`, `--grep` และสามารถผสมกันเพื่อค้นหา commit ที่ต้องการได้อย่างแม่นยำ
4. `git diff` โดย default เทียบ Working Directory กับ Staging Area — แสดงเฉพาะสิ่งที่ยังไม่ได้ `git add`
5. `git diff --staged`/`--cached` เทียบ Staging Area กับ HEAD ส่วน `git diff HEAD` เทียบ Working Directory กับ HEAD โดยตรง ครอบคลุมทุกการเปลี่ยนแปลงไม่ว่าจะ add แล้วหรือยัง
6. `git show <commit>` แสดงรายละเอียดเต็มของ commit เดียว (metadata + diff) และใช้ดูเนื้อหาไฟล์ ณ commit ใดก็ได้ด้วย syntax `<commit>:<path>`
7. การอ้างอิง commit ทำได้หลายแบบ: full hash, short hash, `HEAD`, `HEAD~n` (เดินตาม parent แรกเสมอ), `HEAD^n` (เลือก parent ที่ n ของ merge commit)
8. `git log -p` แสดง diff เต็มของทุก commit ส่วน `git log --stat` แสดงสรุปไฟล์ที่เปลี่ยนพร้อมจำนวนบรรทัด +/-
9. `--pretty=format` ปรับแต่งรูปแบบการแสดงผล log ได้อย่างอิสระ และ git alias ช่วยให้เรียกใช้คำสั่งยาว ๆ ซ้ำได้ด้วยชื่อสั้น ๆ (จะเจาะลึกเต็มรูปแบบใน Part 13)
10. เฟส 1 (Part 01–05) ปูพื้นฐานให้คุณเข้าใจ Git ตั้งแต่แนวคิดพื้นฐานไปจนถึงการสร้าง commit และตรวจสอบประวัติได้อย่างคล่องแคล่ว พร้อมสำหรับการเรียนรู้เรื่อง `.gitignore`, branching และการทำงานร่วมกับผู้อื่นในเฟสถัดไป

**ต่อไป:** [Part 06: .gitignore และการจัดการไฟล์ที่ไม่ต้องการติดตาม](./part-006-gitignore-และการจัดการไฟล์ที่ไม่ต้องการติดตาม.md)
