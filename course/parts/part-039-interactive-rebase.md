# Part 39: Interactive Rebase: squash, reorder, edit history

> **Step ในหลักสูตรนี้:** Step 381–390
> **เฟส:** 4 — การทำงานเป็นทีมด้วย Git/GitHub
> **เป้าหมายของ Part นี้:** เข้าใจ Interactive Rebase อย่างลึกซึ้ง — ใช้ `git rebase -i` เพื่อ squash, reword, edit, reorder และ drop commit ได้อย่างมั่นใจ เข้าใจความแตกต่างระหว่าง fixup กับ squash เข้าใจอันตรายที่แท้จริงของการเปลี่ยนแปลงประวัติ และฝึกทำความสะอาดประวัติ commit ที่ยุ่งเหยิงให้กลายเป็นประวัติที่อ่านง่ายและมืออาชีพ ก่อนจะส่ง Pull Request ให้ทีมรีวิว

---

## สารบัญของ Part นี้

- Step 381: Interactive rebase คืออะไร (`git rebase -i HEAD~n`)
- Step 382: หน้าจอ interactive rebase editor และคำสั่งที่มีทั้งหมด (pick, reword, edit, squash, fixup, drop, exec)
- Step 383: Squash — รวม commit หลายอันเป็นอันเดียว
- Step 384: Reword — แก้ commit message ของ commit เก่าโดยไม่แก้เนื้อหา
- Step 385: Edit — หยุดที่ commit นั้นเพื่อแก้ไขเนื้อหาไฟล์
- Step 386: Reorder commits — สลับลำดับบรรทัดใน editor เพื่อเปลี่ยนลำดับ commit
- Step 387: Drop commit — ลบ commit ทิ้งจากประวัติทั้งหมด
- Step 388: Fixup vs squash ความแตกต่าง
- Step 389: อันตรายของ interactive rebase และกฎทองที่ต้องท่องให้ขึ้นใจ
- Step 390: แบบฝึกหัด — ทำความสะอาดประวัติ 6 commit ที่ยุ่งเหยิง

---

## Step 381: Interactive rebase คืออะไร (`git rebase -i HEAD~n`)

ใน Part 38 เราเรียนไปแล้วว่า `git rebase` คือการ "ย้ายฐาน" ของ commit ชุดหนึ่งไปวางไว้บน commit อีกจุดหนึ่ง เพื่อให้ประวัติเป็นเส้นตรง (linear history) แทนที่จะมี merge commit แตกแขนงไปมา แต่สิ่งที่เราคุยกันใน Part นั้นคือ **rebase แบบธรรมดา (non-interactive rebase)** ซึ่งทำหน้าที่แค่ "ย้ายตำแหน่ง" ของ commit เท่านั้น โดยไม่แตะต้องเนื้อหาหรือลำดับของ commit เลย

**Interactive Rebase** คือ rebase อีกโหมดหนึ่งที่เปิดโอกาสให้คุณ **เข้าไปแทรกแซงกระบวนการ rebase ทีละ commit** — คุณสามารถเลือกได้ว่า commit แต่ละอันจะถูก:

- คงไว้เหมือนเดิม
- รวมเข้ากับ commit อื่น
- แก้ไขข้อความ commit message
- แก้ไขเนื้อหาไฟล์
- สลับลำดับ
- ลบทิ้งไปเลย
- หรือแม้แต่รันคำสั่งบางอย่างแทรกระหว่างทาง

พูดให้เข้าใจง่ายที่สุด:

> **Interactive Rebase คือเครื่องมือ "เขียนประวัติ commit ใหม่" (rewrite history) แบบละเอียดที่สุดที่ Git มีให้**

### คำสั่งพื้นฐาน

```bash
git rebase -i HEAD~n
```

โดยที่:

- `-i` ย่อมาจาก `--interactive`
- `HEAD~n` คือจำนวน commit ล่าสุด n ตัว นับถอยหลังจาก HEAD ที่คุณต้องการนำเข้าสู่กระบวนการ interactive rebase

ตัวอย่าง ถ้าคุณต้องการจัดการ 5 commit ล่าสุด:

```bash
git rebase -i HEAD~5
```

คำสั่งนี้จะเปิด **text editor** (ตัวที่ตั้งค่าไว้ใน `core.editor` เช่น vim, nano, VS Code) ขึ้นมาแสดง "รายการ commit ที่จะถูกจัดการ" ให้คุณแก้ไขได้

### interactive rebase กับ base อื่น ๆ ที่ไม่ใช่ HEAD~n

นอกจากระบุจำนวน commit คุณยังสามารถระบุ base เป็นชื่อ branch, tag หรือ commit hash ได้โดยตรง เช่น:

```bash
git rebase -i main
```

หมายความว่า "นำ commit ทั้งหมดที่อยู่บน branch ปัจจุบันแต่ไม่มีอยู่บน `main` เข้าสู่กระบวนการ interactive rebase" ซึ่งมีประโยชน์มากเวลาคุณทำงานบน feature branch แล้วต้องการจัดระเบียบ commit ทั้งหมดของ branch นั้นก่อนส่ง Pull Request โดยไม่ต้องมานั่งนับว่ามีกี่ commit

```bash
git rebase -i <commit-hash>
```

ก็ใช้ได้เช่นกัน โดย `<commit-hash>` คือ commit ที่อยู่ **ก่อน** commit แรกที่คุณต้องการเริ่มแก้ไข (ไม่ใช่ commit แรกที่ต้องการแก้ไข)

### เมื่อไหร่ควรใช้ Interactive Rebase

Interactive rebase เหมาะกับสถานการณ์เหล่านี้เป็นพิเศษ:

1. **ก่อนส่ง Pull Request** — ทำความสะอาด commit ที่มีข้อความอย่าง "wip", "fix typo", "oops forgot file" ให้กลายเป็นชุด commit ที่มีความหมายและอ่านง่าย
2. **รวม commit เล็ก ๆ ที่เกี่ยวข้องกันเป็นก้อนเดียว** เพื่อให้ reviewer เข้าใจการเปลี่ยนแปลงเป็นหน่วยตรรกะเดียว
3. **แก้ไข commit message ที่พิมพ์ผิดหรือสื่อความหมายไม่ชัดเจน** โดยไม่ต้องเปลี่ยนเนื้อหาโค้ด
4. **แทรกไฟล์ที่ลืมใส่เข้าไปใน commit เก่า** โดยไม่ต้องสร้าง commit ใหม่แยกต่างหากที่ดูรก
5. **จัดลำดับ commit ใหม่** ให้เรื่องราวของประวัติ (git log) อ่านแล้วเข้าใจง่ายเป็นเหตุเป็นผล
6. **ลบ commit ที่ไม่จำเป็นออกไปเลย** เช่น commit ที่ revert ตัวเองในภายหลังอยู่แล้ว

### ตัวอย่างสถานการณ์จริง

สมมติคุณทำงานบน feature branch `feature/login-form` แล้ว `git log --oneline` แสดงผลแบบนี้:

```
a1b2c3d (HEAD -> feature/login-form) fix typo in error message
9f8e7d6 oops forgot to add validation
5c4b3a2 wip login form styling
1e2d3c4 add login form html structure
```

ประวัติแบบนี้ถ้าส่งเป็น Pull Request ไปตรง ๆ reviewer จะงงว่า "wip" คืออะไร "oops" คืออะไร ทำไมต้องมี commit แก้ typo แยกออกมาต่างหาก การใช้ interactive rebase จะช่วยให้คุณรวมทั้ง 4 commit นี้ให้กลายเป็น commit เดียวที่มีความหมายชัดเจน เช่น `feat: add login form with client-side validation` หรือแยกเป็น 2 commit ที่มีเหตุผลชัดเจนก็ได้ตามต้องการ

เราจะฝึกสถานการณ์แบบนี้จริง ๆ ใน Step 390

---

## Step 382: หน้าจอ interactive rebase editor และคำสั่งที่มีทั้งหมด

เมื่อคุณรันคำสั่ง `git rebase -i HEAD~4` (จากตัวอย่างใน Step 381) หน้าจอ editor จะเปิดขึ้นมาพร้อมเนื้อหาประมาณนี้:

```
pick 1e2d3c4 add login form html structure
pick 5c4b3a2 wip login form styling
pick 9f8e7d6 oops forgot to add validation
pick a1b2c3d fix typo in error message

# Rebase 8f3a1b2..a1b2c3d onto 8f3a1b2 (4 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but discard this commit's
#                    log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C
#                    but opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>] = create a merge
#         commit using the original merge commit's message (or the oneline,
#         if no original merge commit was specified); use -c <commit> to
#         reword the commit message
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

มีรายละเอียดสำคัญที่ต้องเข้าใจก่อนจะไปดูแต่ละคำสั่ง:

### สังเกตลำดับของ commit ในรายการนี้

**commit เก่าที่สุดอยู่บนสุด commit ใหม่ที่สุดอยู่ล่างสุด** — นี่คือสิ่งที่สลับกับตอนที่คุณดู `git log` ปกติ (ที่ commit ใหม่ที่สุดอยู่บนสุด) เหตุผลคือ Git จะ **execute (ประมวลผล) รายการนี้จากบนลงล่าง** เหมือนอ่านสคริปต์ทีละบรรทัด ดังนั้นมันเรียงจากเก่าไปใหม่ตามลำดับเวลาจริงที่จะถูก apply ใหม่

### คำสั่งทั้งหมดที่ใช้ได้ (ทีละคำสั่งแบบละเอียด)

#### 1. `pick` (ย่อ: `p`)

ความหมาย: **ใช้ commit นี้ตามเดิม ไม่แก้ไขอะไรเลย**

นี่คือค่า default ของทุกบรรทัด ถ้าคุณไม่แก้ไขอะไรเลยแล้วปิด editor การ rebase จะเสร็จสิ้นโดยไม่มีอะไรเปลี่ยนแปลง (commit ทุกอันถูกสร้างใหม่ในลำดับเดิม เนื้อหาเดิม ข้อความเดิม — แต่ **hash จะเปลี่ยนหมด** เพราะเป็นการ replay commit ใหม่ทั้งหมด รายละเอียดนี้สำคัญมากและเราจะพูดถึงใน Step 389)

#### 2. `reword` (ย่อ: `r`)

ความหมาย: **ใช้ commit นี้ แต่หยุดให้แก้ไข commit message เท่านั้น เนื้อหาไฟล์ไม่เปลี่ยน**

เมื่อ Git เจอบรรทัดที่มาร์คว่า `reword` มันจะหยุดกระบวนการ rebase ชั่วคราว เปิด editor อีกครั้งให้คุณแก้ไขข้อความของ commit นั้น แล้ว rebase ต่อไปอัตโนมัติทันทีที่คุณ save ข้อความใหม่

รายละเอียดเต็มอยู่ใน Step 384

#### 3. `edit` (ย่อ: `e`)

ความหมาย: **ใช้ commit นี้ แต่หยุดกระบวนการ rebase ทั้งหมดไว้ที่จุดนี้ เพื่อให้คุณแก้ไขอะไรก็ได้**

ต่างจาก `reword` ตรงที่ `edit` จะคืนควบคุมกลับมาให้คุณเต็มรูปแบบ — คุณสามารถแก้ไขไฟล์ เพิ่มไฟล์ ลบไฟล์ รัน `git commit --amend` หรือแม้แต่สร้าง commit ใหม่แทรกเข้าไปตรงนั้นได้เลย ก่อนจะสั่ง `git rebase --continue` เพื่อไปต่อ

รายละเอียดเต็มอยู่ใน Step 385

#### 4. `squash` (ย่อ: `s`)

ความหมาย: **รวม commit นี้เข้ากับ commit ที่อยู่ก่อนหน้ามันทันที (บรรทัดด้านบน) โดยเก็บ commit message ของทั้งสองไว้ให้คุณแก้ไขรวมกัน**

commit ที่ mark เป็น `squash` จะถูก "หลอมรวม" เข้ากับ commit ก่อนหน้าเสมอ (commit ที่บรรทัดอยู่เหนือมัน) ไม่ใช่รวมกับ commit ที่อยู่หลังมัน — ดังนั้น `squash` ใช้ไม่ได้กับบรรทัดแรกสุดของรายการ (เพราะไม่มี commit ก่อนหน้าให้รวมด้วย)

รายละเอียดเต็มอยู่ใน Step 383

#### 5. `fixup` (ย่อ: `f`)

ความหมาย: **เหมือน `squash` ทุกอย่าง แต่ทิ้ง commit message ของ commit นี้ไปเลย ใช้ message ของ commit ก่อนหน้าเป็นข้อความสุดท้าย**

ตั้งแต่ Git เวอร์ชันใหม่ ๆ (2.32+) รองรับ `fixup -C <commit>` ซึ่งกลับด้าน คือเก็บ message ของ commit ที่กำลังทำ fixup แทน และ `fixup -c <commit>` ที่ทำแบบเดียวกันแต่เปิด editor ให้แก้ไขเพิ่มเติมได้

รายละเอียดเต็มอยู่ใน Step 388 (เปรียบเทียบกับ squash โดยตรง)

#### 6. `drop` (ย่อ: `d`)

ความหมาย: **ลบ commit นี้ทิ้งไปเลย ไม่ apply เข้าประวัติเลย**

วิธีลบมีสองแบบเทียบเท่ากัน: พิมพ์ `drop` หน้าบรรทัดนั้น หรือ **ลบบรรทัดนั้นทิ้งทั้งบรรทัด** ก็ได้ผลเหมือนกัน แต่ Git แนะนำให้ใช้คำว่า `drop` เพราะชัดเจนกว่าและลดโอกาสลบผิดโดยไม่ตั้งใจ (ข้อความ comment ในไฟล์ก็เตือนไว้ว่า "If you remove a line here THAT COMMIT WILL BE LOST")

รายละเอียดเต็มอยู่ใน Step 387

#### 7. `exec` (ย่อ: `x`)

ความหมาย: **รันคำสั่ง shell ใด ๆ ก็ได้ ณ จุดนั้นในกระบวนการ rebase**

เช่น คุณอยากให้ Git รันเทสต์หลังจาก apply แต่ละ commit เพื่อเช็คว่า commit นั้นยังทำให้โค้ด build/test ผ่านอยู่ (bisectability) คุณสามารถเพิ่มบรรทัด `exec npm test` แทรกไว้ระหว่างแต่ละ `pick` ได้ ถ้าคำสั่งที่ exec รันแล้ว **exit code ไม่ใช่ 0** (คือ fail) กระบวนการ rebase จะหยุดตรงนั้นทันที ให้คุณแก้ไขก่อนสั่ง `--continue`

ตัวอย่าง:

```
pick 1e2d3c4 add login form html structure
exec npm test
pick 5c4b3a2 add login form styling
exec npm test
pick 9f8e7d6 add form validation logic
exec npm test
```

หรือใช้ shortcut อัตโนมัติที่แทรก `exec` หลังทุกบรรทัดให้เองด้วยแฟล็ก `--exec`:

```bash
git rebase -i --exec "npm test" HEAD~3
```

#### 8. `break` (ย่อ: `b`)

ความหมาย: **หยุดกระบวนการ rebase ตรงจุดนี้ทันที โดยไม่ apply commit ถัดไป จนกว่าจะสั่ง `git rebase --continue` เอง**

ต่างจาก `edit` ตรงที่ `break` ไม่ผูกกับ commit ใดเป็นพิเศษ มันเป็นแค่ "จุดพักชั่วคราว" ที่คุณแทรกไว้ในลำดับได้อิสระ เหมาะกับตอนที่อยากหยุดตรวจสอบสถานะระหว่างทางโดยไม่จำเป็นต้องแก้ไข commit ใดเป็นพิเศษ

#### 9. `label` และ `reset` (ย่อ: `l`, `t`)

สองคำสั่งนี้เป็นคำสั่งขั้นสูงที่มักเห็นอัตโนมัติเมื่อใช้ `git rebase -i` กับกลยุทธ์ `--rebase-merges` (rebase ที่ยังคง merge commit ไว้) ใช้สำหรับ "ตั้งชื่อจุด" (`label`) แล้ว "ย้อนกลับไปยังจุดที่ตั้งชื่อไว้" (`reset`) เพื่อสร้างโครงสร้างกิ่งก้านที่ซับซ้อนขึ้นมาใหม่ ผู้เรียนระดับเริ่มต้นถึงกลางแทบไม่ต้องแตะคำสั่งนี้ด้วยมือเอง

#### 10. `merge` (ย่อ: `m`)

ใช้คู่กับ `--rebase-merges` เพื่อสร้าง merge commit ขึ้นมาใหม่ ณ จุดที่กำหนด โดยรักษาโครงสร้างการ merge เดิมไว้แทนที่จะทำให้ทุกอย่างกลายเป็นเส้นตรง

### ตารางสรุปคำสั่งทั้งหมด

| คำสั่ง | ย่อ | ผลลัพธ์ | แก้เนื้อหาไฟล์ไหม | แก้ message ไหม |
|---|---|---|---|---|
| `pick` | `p` | ใช้ commit ตามเดิม | ไม่ | ไม่ |
| `reword` | `r` | ใช้ commit เดิม แต่หยุดแก้ message | ไม่ | ใช่ |
| `edit` | `e` | หยุดที่ commit นี้ให้แก้ไขอิสระ | ได้ (ตามต้องการ) | ได้ (ตามต้องการ) |
| `squash` | `s` | รวมเข้ากับ commit ก่อนหน้า เก็บทั้งสอง message | รวมไฟล์เข้าด้วยกัน | รวม/แก้ไขได้ |
| `fixup` | `f` | รวมเข้ากับ commit ก่อนหน้า ทิ้ง message ของตัวเอง | รวมไฟล์เข้าด้วยกัน | ไม่ (ใช้ของ commit ก่อนหน้า) |
| `drop` | `d` | ลบ commit ทิ้งทั้งหมด | ลบทั้ง commit | ลบทั้ง commit |
| `exec` | `x` | รันคำสั่ง shell แทรก | ไม่แตะ commit ใด | ไม่แตะ commit ใด |
| `break` | `b` | หยุดชั่วคราว ณ จุดนี้ | ไม่แตะ commit ใด | ไม่แตะ commit ใด |
| `label` / `reset` | `l` / `t` | ตั้งชื่อจุด/ย้อนกลับ (ใช้กับ `--rebase-merges`) | ไม่แตะ | ไม่แตะ |
| `merge` | `m` | สร้าง merge commit ใหม่ (ใช้กับ `--rebase-merges`) | สร้าง merge ใหม่ | กำหนดได้ |

### การควบคุม flow ระหว่างทำ interactive rebase

เมื่อกระบวนการหยุด (เพราะ `edit`, `break`, conflict หรือ `exec` fail) คุณมีคำสั่งเหล่านี้ให้ใช้:

```bash
git rebase --continue   # ทำต่อจากจุดที่หยุด หลังแก้ไขเสร็จแล้ว
git rebase --skip       # ข้าม commit นี้ไปเลย (ใช้ตอนเกิด conflict ที่ตัดสินใจว่าจะไม่เอา commit นี้)
git rebase --abort      # ยกเลิกทั้งหมด กลับไปสู่สถานะก่อนเริ่ม rebase ทันที
git rebase --edit-todo  # แก้ไขรายการ todo อีกครั้งระหว่างที่ rebase ยังไม่เสร็จ
```

`git rebase --abort` เป็นคำสั่งที่ **สำคัญมากสำหรับมือใหม่** — ถ้าคุณทำ interactive rebase แล้วรู้สึกว่าสถานการณ์เริ่มยุ่งเหยิงเกินจะแก้ไข ให้สั่ง `--abort` ทันที มันจะพา repository กลับไปอยู่ในสถานะเดิมก่อนที่คุณจะเริ่ม rebase เลย เหมือนไม่มีอะไรเกิดขึ้น ปลอดภัย 100%

---

## Step 383: Squash — รวม commit หลายอันเป็นอันเดียว

**Squash** คือการนำ commit ตั้งแต่ 2 อันขึ้นไปมา "หลอมรวม" ให้กลายเป็น commit เดียว โดยที่ Git จะเสนอ commit message ของทุก commit ที่ถูกรวมให้คุณเลือกเก็บ ตัด หรือแก้ไขใหม่ทั้งหมด

### ตัวอย่างการใช้งานจริง

สมมติ `git log --oneline -4` แสดงผล:

```
a1b2c3d (HEAD -> feature/login-form) add error message styling
9f8e7d6 add form validation logic
5c4b3a2 add login form css
1e2d3c4 add login form html structure
```

คุณต้องการรวม `add login form css`, `add form validation logic` และ `add error message styling` เข้าเป็นก้อนเดียว โดยเก็บ `add login form html structure` แยกไว้ต่างหาก (เพราะมันคือการเพิ่มโครงสร้าง HTML ซึ่งเป็นคนละเรื่องกับ styling/logic)

รันคำสั่ง:

```bash
git rebase -i HEAD~4
```

แล้วแก้ไขรายการ todo จาก:

```
pick 1e2d3c4 add login form html structure
pick 5c4b3a2 add login form css
pick 9f8e7d6 add form validation logic
pick a1b2c3d add error message styling
```

เป็น:

```
pick 1e2d3c4 add login form html structure
pick 5c4b3a2 add login form css
squash 9f8e7d6 add form validation logic
squash a1b2c3d add error message styling
```

**สังเกต:** commit แรก (`1e2d3c4`) ยังคงเป็น `pick` เพราะเราต้องการเก็บมันแยกไว้ ส่วน `5c4b3a2` เป็น `pick` เพราะมันคือ "จุดเริ่มต้นของกลุ่มที่จะถูกรวม" — commit ที่จะถูกหลอมรวมเข้ากับมันคือสองบรรทัดถัดมาที่มาร์คเป็น `squash`

เมื่อ save และปิด editor Git จะเปิด editor อีกครั้งให้คุณจัดการ commit message ของ commit ที่ถูกรวม แสดงผลประมาณนี้:

```
# This is a combination of 3 commits.
# This is the 1st commit message:

add login form css

# This is the commit message #2:

add form validation logic

# This is the commit message #3:

add error message styling

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
```

คุณสามารถลบทุกอย่างแล้วเขียนข้อความใหม่ที่สรุปรวมทั้งสาม เช่น:

```
feat: add login form styling with validation and error messages

- Add CSS styling for the login form layout
- Implement client-side validation logic
- Add styled error message display
```

save แล้วปิด editor กระบวนการ rebase จะดำเนินต่อไปจนเสร็จ ผลลัพธ์สุดท้ายจาก `git log --oneline`:

```
f4e5d6c (HEAD -> feature/login-form) feat: add login form styling with validation and error messages
1e2d3c4 add login form html structure
```

จาก 4 commit เหลือ 2 commit ที่มีความหมายชัดเจนขึ้นมาก

### squash หลาย commit พร้อมกันในครั้งเดียว

คุณไม่จำเป็นต้อง squash ทีละคู่ — สามารถมาร์ค `squash` ต่อกันหลายบรรทัดติดกันได้เลย ทุกบรรทัดที่เป็น `squash` ติดต่อกันจะถูกรวมเข้ากับ `pick` (หรือ commit ที่ต่อเนื่องมา) ที่อยู่เหนือมันตามลำดับจากบนลงล่างไปเรื่อย ๆ จนกว่าจะเจอ `pick` บรรทัดใหม่

### ข้อควรระวังของ squash

1. **ลำดับของ commit ในรายการ todo มีผลกับผลลัพธ์การรวม** — squash รวมกับ commit ที่อยู่ *เหนือ* มันเสมอ ไม่ใช่ด้านล่าง
2. **การ squash รวมทั้งไฟล์และ diff เข้าด้วยกัน** ถ้า commit ที่ถูกรวมมีการแก้ไขไฟล์เดียวกันในลักษณะที่ทับซ้อนกันในรูปแบบซับซ้อน อาจเกิด conflict ระหว่างกระบวนการ rebase ได้เช่นกัน (ถึงแม้ว่า commit เหล่านี้เคย apply สำเร็จมาแล้วในอดีต แต่การ replay ใหม่อาจเจอสถานการณ์ต่างออกไปเมื่อรวมกับ commit ที่ปรับเปลี่ยนไปแล้วก่อนหน้า)
3. **squash เก็บ author date ของ commit แรกสุดในกลุ่ม** ส่วน commit date จะถูกอัปเดตเป็นเวลาที่ทำ rebase

---

## Step 384: Reword — แก้ commit message ของ commit เก่าโดยไม่แก้เนื้อหา

บ่อยครั้งปัญหาของ commit ไม่ใช่เนื้อหาโค้ด แต่คือ **ข้อความอธิบาย** ที่เขียนไม่ดี พิมพ์ผิด หรือไม่ตรงตาม convention ของทีม (เช่น Conventional Commits ที่เรียนใน Part 35) — กรณีแบบนี้ใช้ `reword` แทน `edit` เพราะเราไม่ต้องการแตะเนื้อหาไฟล์เลย

### ตัวอย่าง

`git log --oneline -3`:

```
a1b2c3d (HEAD -> feature/user-profile) add user profile pic upload
9f8e7d6 fxed bug in avatar resize
5c4b3a2 add avatar resize function
```

commit ตรงกลางมีคำสะกดผิด (`fxed` ควรเป็น `fixed`) และไม่ตรงตาม convention ที่ทีมใช้ (`fix:` prefix) รันคำสั่ง:

```bash
git rebase -i HEAD~3
```

แก้ไขจาก:

```
pick 5c4b3a2 add avatar resize function
pick 9f8e7d6 fxed bug in avatar resize
pick a1b2c3d add user profile pic upload
```

เป็น:

```
pick 5c4b3a2 add avatar resize function
reword 9f8e7d6 fxed bug in avatar resize
pick a1b2c3d add user profile pic upload
```

save แล้วปิด editor Git จะหยุดตรง commit `9f8e7d6` และเปิด editor ใหม่พร้อมข้อความเดิมให้แก้ไข:

```
fxed bug in avatar resize

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored...
```

แก้เป็น:

```
fix: correct off-by-one error in avatar resize calculation
```

save แล้วปิด editor — กระบวนการ rebase จะทำต่อไปยัง commit ถัดไปโดยอัตโนมัติทันที (ไม่ต้องพิมพ์ `git rebase --continue` เอง เพราะ `reword` ไม่ได้หยุดให้แก้ไฟล์ จึงไม่มีอะไรให้ stage ก่อนไปต่อ)

### จุดสำคัญที่ต้องเข้าใจเรื่อง reword

- `reword` **ไม่เปิดโอกาสให้แก้ไขไฟล์ใด ๆ เลย** มันจ ากัดขอบเขตไว้แค่ข้อความเท่านั้น ถ้าอยากแก้ทั้งไฟล์และ message พร้อมกัน ต้องใช้ `edit` แทน
- shortcut อีกทางหนึ่งสำหรับ reword เฉพาะ commit ล่าสุดสุดคือ `git commit --amend` (ไม่ต้องพึ่ง interactive rebase เลย) แต่ถ้าต้องการแก้ commit ที่ **ไม่ใช่ commit ล่าสุด** ต้องใช้ `git rebase -i` กับ `reword` เท่านั้น
- ข้อความใหม่จะไม่กระทบ diff ของ commit นั้นแม้แต่นิดเดียว — เนื้อหาไฟล์เหมือนเดิมทุกประการ มีแค่ metadata ข้อความเปลี่ยน

---

## Step 385: Edit — หยุดที่ commit นั้นเพื่อแก้ไขเนื้อหาไฟล์

`edit` คือคำสั่งที่ทรงพลังที่สุดในบรรดาทั้งหมด เพราะมันคืนการควบคุมแบบเต็มรูปแบบกลับมาให้คุณ ณ จุดของ commit ที่ระบุ

### สถานการณ์ที่ใช้ edit

1. ลืมใส่ไฟล์บางไฟล์ใน commit เก่า
2. commit เก่ามีโค้ดที่ผิดพลาดเล็กน้อยที่อยากแก้ไขตรงนั้นเลย ไม่อยากสร้าง commit แก้บั๊กแยกทีหลัง
3. ต้องการแบ่ง commit หนึ่งอันออกเป็นหลาย ๆ commit ย่อย (split commit)
4. ต้องการรัน command บางอย่าง (formatting, linting) เฉพาะจุดนั้นในประวัติ

### ตัวอย่าง: แก้ไขเนื้อหาไฟล์ในธุรกิจของ commit เก่า

`git log --oneline -3`:

```
a1b2c3d (HEAD -> feature/api-client) add retry logic to api client
9f8e7d6 add timeout config to api client
5c4b3a2 add base api client class
```

พบว่า commit `9f8e7d6 add timeout config to api client` มีค่า default timeout ผิดพลาด (ตั้งเป็น 500ms ทั้งที่ควรเป็น 5000ms) ต้องการแก้ไขตรงจุดนั้นในประวัติ ไม่ใช่เพิ่ม commit ใหม่ทับ

```bash
git rebase -i HEAD~3
```

แก้ todo list:

```
pick 5c4b3a2 add base api client class
edit 9f8e7d6 add timeout config to api client
pick a1b2c3d add retry logic to api client
```

save แล้วปิด editor — Git จะ apply commit `5c4b3a2` ตามปกติ จากนั้น apply commit `9f8e7d6` แล้ว **หยุดทันที** พร้อมข้อความประมาณนี้:

```
Stopped at 9f8e7d6...  add timeout config to api client
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue
```

ตอนนี้ working directory ของคุณอยู่ในสถานะเดียวกับตอนที่ commit `9f8e7d6` เพิ่งถูกสร้าง คุณสามารถแก้ไขไฟล์ได้ตามปกติ:

```bash
# แก้ไขไฟล์ config timeout
vim src/apiClient.js

git add src/apiClient.js
git commit --amend --no-edit
```

`--no-edit` หมายถึงใช้ commit message เดิม (ถ้าต้องการแก้ message ด้วยให้ตัด `--no-edit` ออก editor จะเปิดให้แก้ไข)

เมื่อแก้ไขเสร็จแล้ว สั่งให้ rebase ทำต่อ:

```bash
git rebase --continue
```

Git จะนำ commit ถัดไป (`a1b2c3d`) มา apply ต่อจาก commit ที่เพิ่งถูกแก้ไข — ถ้า commit ถัดไปมีการแก้ไขไฟล์เดียวกันในบรรทัดที่ทับซ้อนกับสิ่งที่คุณเพิ่งแก้ไข อาจเกิด conflict ให้แก้ไขตามปกติก่อน `git add` แล้ว `git rebase --continue` ต่อไป

### การแยก commit หนึ่งเป็นหลาย commit ด้วย edit

`edit` ยังใช้แยก commit ใหญ่ที่รวมหลายเรื่องไว้ในก้อนเดียวออกเป็นหลาย commit ย่อยได้ ด้วยเทคนิค **reset แล้ว commit ใหม่ทีละส่วน**:

```bash
git rebase -i HEAD~3
# มาร์ค commit ที่ต้องการแยกเป็น edit

# เมื่อ Git หยุดที่ commit นั้น:
git reset HEAD^          # ยกเลิก commit นั้น แต่เก็บการเปลี่ยนแปลงไว้ใน working directory (mixed reset)

# ตอนนี้การเปลี่ยนแปลงทั้งหมดของ commit นั้นอยู่ใน working directory แบบ unstaged
git add src/feature-a.js
git commit -m "feat: add feature A"

git add src/feature-b.js
git commit -m "feat: add feature B"

git rebase --continue
```

เทคนิคนี้มีประโยชน์มากเมื่อพบว่า commit เก่าอันหนึ่งทำหลายเรื่องปนกันจนควรแยกเพื่อให้ `git bisect` (ที่จะเรียนใน Part 42) ทำงานได้แม่นยำขึ้น หรือเพื่อให้ reviewer เห็นการเปลี่ยนแปลงแยกเป็นหน่วยตรรกะที่ชัดเจน

---

## Step 386: Reorder commits — สลับลำดับบรรทัดใน editor เพื่อเปลี่ยนลำดับ commit

เนื่องจากรายการ todo ของ interactive rebase คือ **สคริปต์ที่ถูกประมวลผลจากบนลงล่างตามลำดับที่ปรากฏ** การสลับตำแหน่งบรรทัดในไฟล์จึงเท่ากับการสลับลำดับที่ commit จะถูก apply ใหม่ทั้งหมด

### ตัวอย่าง

`git log --oneline -3`:

```
a1b2c3d (HEAD -> feature/checkout) add checkout page tests
9f8e7d6 add discount code feature
5c4b3a2 add checkout page ui
```

คุณต้องการให้ commit เรื่อง test มาอยู่หลังสุด (logical order: ui → discount → tests) ซึ่งจริง ๆ ลำดับนี้ก็เป็นแบบนั้นอยู่แล้วในตัวอย่างนี้ ลองสมมติสถานการณ์กลับกัน:

```
a1b2c3d (HEAD -> feature/checkout) add discount code feature
9f8e7d6 add checkout page tests
5c4b3a2 add checkout page ui
```

คุณอยากให้ tests ไปอยู่หลังสุด (เพราะ tests ควรครอบคลุมทั้ง ui และ discount code) รันคำสั่ง:

```bash
git rebase -i HEAD~3
```

todo list เดิม:

```
pick 5c4b3a2 add checkout page ui
pick 9f8e7d6 add checkout page tests
pick a1b2c3d add discount code feature
```

สลับบรรทัด 2 กับ 3 (ตัดวางในตัว editor หรือใช้ฟีเจอร์ move line ของ vim/VS Code):

```
pick 5c4b3a2 add checkout page ui
pick a1b2c3d add discount code feature
pick 9f8e7d6 add checkout page tests
```

save แล้วปิด editor Git จะ apply commit ตามลำดับใหม่: ui → discount → tests

### อันตรายของการ reorder: conflict ที่ไม่เคยเกิดขึ้นมาก่อน

การสลับลำดับ commit ที่แก้ไข **ไฟล์เดียวกันในบรรทัดที่เกี่ยวข้องกัน** มีความเสี่ยงสูงมากที่จะเกิด conflict ระหว่างการ replay เพราะลำดับของการเปลี่ยนแปลงที่ Git คาดหวังตอนสร้าง diff ของแต่ละ commit เปลี่ยนไปจากตอนที่มันถูกสร้างครั้งแรก

ตัวอย่างเช่น ถ้า commit `discount code feature` แก้ไขฟังก์ชันที่ `checkout page tests` เขียนเทสต์ครอบคลุมไว้แล้ว การสลับให้ `discount code feature` มาอยู่ก่อน `tests` (จากที่เดิมอยู่หลัง) อาจทำให้ diff ของ tests ที่คาดหวังโครงสร้างเดิมของโค้ด apply ไม่ลงตัว เกิด conflict ขึ้นมา ต้องแก้ไข conflict นั้นด้วยมือก่อน `git add` แล้ว `git rebase --continue`

### หลักการปฏิบัติ: reorder ทีละนิด ทดสอบบ่อย ๆ

เมื่อทำการ reorder commit จำนวนมาก แนะนำให้:

1. สลับตำแหน่งทีละคู่ ไม่สลับหลายจุดพร้อมกันในครั้งเดียว (โดยเฉพาะถ้ายังไม่ชำนาญ)
2. ใช้ `exec` แทรกคำสั่ง build/test ไว้หลังแต่ละ `pick` เพื่อจับปัญหาได้ทันทีที่จุดที่เกิด แทนที่จะรู้ตัวหลังจาก rebase เสร็จไปแล้วทั้งหมด
3. ถ้าเจอ conflict ซับซ้อนเกินคาดระหว่าง reorder ให้ `git rebase --abort` แล้วกลับมาคิดลำดับใหม่อีกครั้งแทนที่จะฝืนแก้ conflict ไปเรื่อย ๆ

---

## Step 387: Drop commit — ลบ commit ทิ้งจากประวัติทั้งหมด

`drop` คือการบอก Git ว่า "อย่า apply commit นี้เลย ตัดออกจากประวัติไปเลย เหมือนไม่เคยมีมันมาก่อน"

### สองวิธีที่ทำ drop ได้เหมือนกัน

**วิธีที่ 1:** เปลี่ยนคำสั่งหน้าบรรทัดเป็น `drop`

```
pick 5c4b3a2 add checkout page ui
drop 9f8e7d6 add debug console.log statements
pick a1b2c3d add discount code feature
```

**วิธีที่ 2:** ลบบรรทัดทั้งบรรทัดทิ้งไปเลย

```
pick 5c4b3a2 add checkout page ui
pick a1b2c3d add discount code feature
```

ทั้งสองวิธีให้ผลลัพธ์เหมือนกันทุกประการ — commit `9f8e7d6` จะหายไปจากประวัติทั้งหมด อย่างไรก็ตาม Git แนะนำให้ใช้คำว่า `drop` แบบชัดเจนแทนการลบบรรทัดเงียบ ๆ เพราะทำให้เจตนาชัดเจนกว่าเวลากลับมาอ่าน todo list อีกครั้ง (เช่นระหว่างที่ต้องหยุดแก้ conflict แล้วกลับมาดูว่าเหลืออะไรบ้าง) และลดโอกาสที่จะลบบรรทัดผิดโดยไม่ได้ตั้งใจ

### ตัวอย่างสถานการณ์การใช้ drop

สถานการณ์ที่พบบ่อยที่สุดคือ commit ที่มี debug statement หลงเหลืออยู่ หรือ commit ที่ทำอะไรบางอย่างแล้วภายหลังมี commit อื่น revert มันกลับไปอยู่ดี ทำให้ทั้งคู่ไม่มีผลอะไรต่อผลลัพธ์สุดท้ายเลย — แทนที่จะเก็บทั้งคู่ไว้ในประวัติให้รก สามารถ `drop` ทั้งสอง commit ออกไปพร้อมกันได้เลย

```
a1b2c3d revert temporary feature flag
9f8e7d6 fix unrelated bug in payment flow
5c4b3a2 add temporary feature flag for testing
```

ถ้า `a1b2c3d` คือการ revert สิ่งที่ `5c4b3a2` ทำไว้แบบสมบูรณ์ (ไม่มีผลกระทบต่อ commit อื่นระหว่างทาง) สามารถ drop ทั้งคู่:

```
pick 9f8e7d6 fix unrelated bug in payment flow
```

### ข้อควรระวังสำคัญของ drop

1. **ตรวจสอบก่อนว่า commit ที่จะ drop ไม่มี commit อื่นพึ่งพา (depend) เนื้อหาของมันอยู่** ถ้า commit ถัดไปแก้ไขไฟล์ที่ commit ที่จะ drop สร้างขึ้นมา การ drop จะทำให้เกิด conflict หรือ error ทันทีที่ apply commit ถัดไป เพราะไฟล์หรือฟังก์ชันที่มันอ้างอิงถึงไม่มีอยู่แล้ว
2. **drop เป็นการลบถาวรจากสาขาที่กำลัง rebase** — ถ้าอยากได้ commit นั้นกลับคืนมาในอนาคต ต้องใช้ `git reflog` เพื่อหา hash เดิมแล้ว `git cherry-pick` กลับเข้ามาใหม่ (ตราบใดที่ยังไม่ถูก garbage collect ทิ้งไป)
3. ก่อน drop commit ใด ๆ ที่ไม่แน่ใจ ให้ใช้ `git show <commit-hash>` ตรวจดูเนื้อหาก่อนเสมอ

---

## Step 388: Fixup vs squash ความแตกต่าง

`fixup` และ `squash` ทำงานเหมือนกันในแง่ของ**การรวมไฟล์และ diff เข้ากับ commit ก่อนหน้า** — ความแตกต่างเดียวที่สำคัญที่สุดอยู่ที่ **การจัดการ commit message**

### ความแตกต่างหลัก

| | `squash` | `fixup` |
|---|---|---|
| รวมเนื้อหาไฟล์เข้ากับ commit ก่อนหน้า | ใช่ | ใช่ |
| commit message ของ commit ที่ถูกรวม | **เก็บไว้** ให้เลือก/แก้ไขรวมกับของ commit ก่อนหน้า | **ทิ้งไปเลย** ใช้ message ของ commit ก่อนหน้าอย่างเดียว |
| หยุดเปิด editor ให้แก้ message ไหม | หยุดเสมอ | ไม่หยุด (ทำงานต่อเนื่องอัตโนมัติ) |
| เหมาะกับ | รวม commit ที่แต่ละอันมีเหตุผลควรอธิบายไว้ | รวม commit เล็ก ๆ ที่เป็นแค่ "แก้ไขเพิ่มเติม" ของ commit ก่อนหน้า ไม่มีอะไรต้องอธิบายเพิ่ม |

### ตัวอย่างเปรียบเทียบ

สมมติ todo list:

```
pick 5c4b3a2 add user registration form
squash 9f8e7d6 fix typo in registration form label
```

ผลลัพธ์: Git เปิด editor ให้แก้ไข โดยแสดงทั้งสอง message รวมกัน:

```
# This is a combination of 2 commits.
# This is the 1st commit message:

add user registration form

# This is the commit message #2:

fix typo in registration form label
```

ในทางกลับกัน ถ้าใช้:

```
pick 5c4b3a2 add user registration form
fixup 9f8e7d6 fix typo in registration form label
```

ผลลัพธ์: ไม่มี editor เปิดขึ้นมาเลย commit สุดท้ายจะใช้ message ว่า `add user registration form` เพียงอย่างเดียว ข้อความ "fix typo in registration form label" หายไปโดยสมบูรณ์ (เพราะมันไม่มีประโยชน์อะไรที่จะเก็บไว้ — มันเป็นแค่ "การแก้ไขเล็กน้อย" ของ commit ก่อนหน้าเท่านั้น)

### `--fixup` และ `--autosquash`: เวิร์กโฟลว์ที่มืออาชีพใช้กันจริง

Git มีฟีเจอร์ที่ช่วยให้การ fixup สะดวกขึ้นมาก โดยไม่ต้องมานั่งเลื่อนบรรทัดใน editor เอง:

```bash
# ระหว่างทำงาน พบว่า commit เก่า (5c4b3a2) ควรได้รับการแก้ไขเพิ่มเติม
git add .
git commit --fixup=5c4b3a2
```

คำสั่งนี้จะสร้าง commit ใหม่โดยตั้ง message ให้เองอัตโนมัติในรูปแบบ:

```
fixup! add user registration form
```

จากนั้นเมื่อทำ interactive rebase ด้วยแฟล็ก `--autosquash`:

```bash
git rebase -i --autosquash HEAD~5
```

Git จะ **จัดเรียงและมาร์ค `fixup` ให้อัตโนมัติทันที** โดยย้าย commit ที่มีคำว่า `fixup!` นำหน้าไปไว้ต่อจาก commit เป้าหมายให้เอง ไม่ต้องสลับบรรทัดหรือพิมพ์คำว่า `fixup` เองเลย คุณแค่เปิด editor มาดู แล้ว save ปิดได้ทันที

ถ้าตั้งค่า:

```bash
git config --global rebase.autosquash true
```

การรัน `git rebase -i` ทุกครั้งจะเปิดใช้ `--autosquash` โดยอัตโนมัติเสมอ (ไม่ต้องพิมพ์แฟล็กทุกครั้ง)

เช่นเดียวกัน `git commit --squash=<commit>` สร้าง commit ที่มี prefix `squash!` แทน ซึ่งเมื่อใช้กับ `--autosquash` จะถูกจัดเรียงเป็น `squash` แทน `fixup`

### ตารางสรุป

| คำสั่งสร้าง commit | prefix ที่ได้ | เมื่อใช้กับ `--autosquash` |
|---|---|---|
| `git commit --fixup=<hash>` | `fixup! <original message>` | ถูกจัดเป็น `fixup` อัตโนมัติ |
| `git commit --squash=<hash>` | `squash! <original message>` | ถูกจัดเป็น `squash` อัตโนมัติ |

เวิร์กโฟลว์นี้เป็นที่นิยมมากในทีมที่ทำงานแบบ trunk-based หรือ feature branch ระยะสั้น เพราะช่วยให้คุณ commit "แก้ไขเพิ่มเติม" ระหว่างพัฒนาได้อย่างอิสระโดยไม่ต้องกังวลเรื่องประวัติรก แล้วค่อยรวบรวมให้สะอาดในขั้นตอนสุดท้ายก่อนส่ง Pull Request ด้วยคำสั่งเดียว

---

## Step 389: อันตรายของ interactive rebase (เปลี่ยน hash ของทุก commit หลังจุดที่แก้ไข)

นี่คือหัวข้อที่สำคัญที่สุดของ Part นี้ และเป็นสิ่งที่ต้องเข้าใจให้ลึกซึ้งก่อนจะนำ interactive rebase ไปใช้ในงานจริงกับทีม

### ทำไม hash ถึงเปลี่ยน

ย้อนกลับไปที่แนวคิดพื้นฐานของ Git ที่เรียนใน Part 01 และ Part 03: **commit hash คำนวณมาจากเนื้อหาของ commit นั้นทั้งหมด** รวมถึง:

- เนื้อหาไฟล์ (tree object)
- ข้อความ commit message
- ชื่อผู้เขียนและเวลา (author, committer, timestamp)
- **hash ของ parent commit**

จุดสำคัญที่สุดคือรายการสุดท้าย — **hash ของ parent commit เป็นส่วนหนึ่งของการคำนวณ hash ของ commit ลูก** ดังนั้นถ้า commit ใด ๆ ในสายเปลี่ยนแปลงไป (ไม่ว่าจะเป็นการแก้เนื้อหา แก้ message หรือแค่ replay ใหม่โดยไม่เปลี่ยนอะไรเลยก็ตาม) **hash ของมันจะเปลี่ยนทันที** และเนื่องจาก hash เปลี่ยน มันจึงส่งผลให้ **hash ของ commit ลูกทุกตัวที่ตามมาเปลี่ยนตามไปด้วยเป็นลูกโซ่ทั้งหมด**

### ภาพประกอบ

ก่อน rebase:

```
1e2d3c4 → 5c4b3a2 → 9f8e7d6 → a1b2c3d (HEAD)
   A         B          C          D
```

สมมติเราทำ `git rebase -i HEAD~3` แล้ว reword commit `B` เพียงแค่แก้ไขข้อความนิดเดียว ผลลัพธ์ที่ได้:

```
1e2d3c4 → 5f6g7h8 → 2i3j4k5 → 6l7m8n9 (HEAD)
   A       B' (reworded)   C' (ใหม่)     D' (ใหม่)
```

สังเกตว่า:

- `1e2d3c4` (commit A) **hash ไม่เปลี่ยน** เพราะไม่ได้อยู่ในขอบเขตของ rebase (มันเป็น base ที่ commit อื่นถูกสร้างขึ้นบนมัน)
- `B` เปลี่ยนเป็น `B'` เพราะถูก reword โดยตรง — hash ใหม่เพราะ content (message) เปลี่ยน
- `C` เปลี่ยนเป็น `C'` แม้ **ไม่ได้ถูกแก้ไขอะไรเลย** เพียงเพราะ parent ของมัน (จาก `B` กลายเป็น `B'`) เปลี่ยนไป จึงต้องคำนวณ hash ใหม่
- `D` เปลี่ยนเป็น `D'` ด้วยเหตุผลเดียวกัน เป็นลูกโซ่ต่อกันไป

> **กฎเหล็ก: ถ้าคุณแก้ไข commit ใดใน interactive rebase ทุก commit ที่อยู่ "หลังจากมัน" (คือลูกหลานทั้งหมดในสายนั้น) จะได้ hash ใหม่หมด แม้จะไม่ได้ถูกแตะต้องเนื้อหาแม้แต่นิดเดียวก็ตาม**

### ทบทวนกฎทองจาก Part 38: ห้าม rebase ประวัติที่แชร์ไปแล้ว

ใน Part 38 เราได้เรียนกฎทองข้อสำคัญที่สุดของการ rebase ไปแล้ว และในบริบทของ interactive rebase กฎนี้ยิ่งสำคัญกว่าเดิมมาก เพราะ interactive rebase เปลี่ยนแปลงเนื้อหาและลำดับของ commit อย่างจงใจ ไม่ใช่แค่ย้ายตำแหน่งเฉย ๆ:

> **กฎทอง: ห้าม rebase (รวมถึง interactive rebase) กับ commit ที่ถูก push ขึ้น remote ไปแล้วและมีคนอื่นดึงไปใช้งานต่อแล้ว**

เหตุผลที่กฎนี้สำคัญมาก:

1. เมื่อคุณ interactive rebase แล้ว push ทับด้วย `git push --force` commit ชุดใหม่ (ที่มี hash ต่างไปทั้งหมด) จะเข้ามาแทนที่ประวัติเดิมบน remote
2. เพื่อนร่วมทีมที่เคย `git pull` หรือสร้าง branch ต่อจาก commit ชุดเดิม (ที่ hash เก่า) จะพบว่า **local branch ของพวกเขาอ้างอิงถึง commit ที่ "หายไป" จาก remote แล้ว**
3. เมื่อพวกเขา `git pull` อีกครั้ง Git จะพยายาม merge ประวัติเก่า (ที่มี hash เดิม) เข้ากับประวัติใหม่ (ที่มี hash ใหม่) ทั้งที่จริง ๆ แล้วเนื้อหาอาจจะเหมือนกันเป๊ะ แต่ Git มองว่าเป็นคนละ commit กันโดยสิ้นเชิง เพราะดู hash เป็นหลัก ผลลัพธ์คือเกิด **commit ซ้ำซ้อน (duplicate commits)** ทั้งชุดเดิมและชุดใหม่ปะปนกันในประวัติ หรือแย่กว่านั้นคือเกิด merge conflict จำนวนมากที่แก้ไขยากมาก

### เมื่อไหร่ที่ interactive rebase ปลอดภัย 100%

Interactive rebase ปลอดภัยอย่างสมบูรณ์เมื่อ:

- commit ที่คุณกำลังจะแก้ไข **ยังไม่เคย push ขึ้น remote เลย** (อยู่ใน local เท่านั้น)
- หรือ push ไปแล้วแต่อยู่บน **feature branch ของคุณคนเดียว** ที่ไม่มีใครอื่น pull หรือสร้าง branch ต่อจากมัน (ตรวจสอบให้แน่ใจกับทีมก่อนเสมอถ้าไม่มั่นใจ)

### ถ้าจำเป็นต้อง force push จริง ๆ

บางครั้งก็มีความจำเป็นต้อง rebase แล้ว force push แม้จะเคย push ไปแล้ว (เช่น แก้ไข Pull Request ของตัวเองก่อน merge) ในกรณีนี้ให้ใช้:

```bash
git push --force-with-lease
```

แทน `git push --force` ธรรมดา เพราะ `--force-with-lease` จะ **ตรวจสอบก่อนว่า remote branch ไม่ได้ถูกอัปเดตโดยคนอื่นนับตั้งแต่ครั้งล่าสุดที่คุณ fetch มา** ถ้ามีใครอื่น push เข้ามาก่อนหน้าคุณโดยที่คุณไม่รู้ตัว คำสั่งนี้จะ **ปฏิเสธการ push ทันที** แทนที่จะเขียนทับงานของคนอื่นไปเงียบ ๆ ซึ่งปลอดภัยกว่ามาก

```bash
# ไม่ควรใช้ (อันตราย: เขียนทับได้แม้มีคนอื่น push มาก่อน)
git push --force

# ควรใช้เสมอแทน (ปลอดภัยกว่า: เช็คก่อนว่าไม่มีใครแก้ remote ไปก่อนหน้า)
git push --force-with-lease
```

### แผนกู้คืนถ้าทำ interactive rebase พลาด: `git reflog`

Git ไม่ได้ลบข้อมูลจริง ๆ ทันทีแม้จะทำ interactive rebase ผิดพลาดไปแล้ว ตราบใดที่ยังไม่ผ่านกระบวนการ garbage collection (`git gc`) commit เก่าทั้งหมดยังอยู่ใน object database เพียงแต่ไม่มี reference (branch/tag) ใดชี้ไปหามันแล้วเท่านั้น

```bash
git reflog
```

จะแสดงประวัติการเปลี่ยนแปลงของ HEAD ทั้งหมด รวมถึงก่อนและหลังการ rebase:

```
6l7m8n9 HEAD@{0}: rebase (finish): returning to refs/heads/feature/login-form
6l7m8n9 HEAD@{1}: rebase (squash): feat: add login form styling
...
a1b2c3d HEAD@{6}: rebase -i (start): checkout HEAD~4
```

ถ้าต้องการกลับไปยังสถานะก่อน rebase ทั้งหมด:

```bash
git reset --hard HEAD@{6}
```

(แทน `HEAD@{6}` ด้วยหมายเลขที่ตรงกับจุดก่อน rebase เริ่มต้นจาก `git reflog` ของคุณเอง) นี่คือ **เซฟตี้เน็ตที่สำคัญมาก** ที่ทำให้ interactive rebase ไม่ใช่การกระทำที่ไม่มีทางย้อนกลับ ตราบใดที่คุณยังไม่ได้ push ทับหรือปล่อยเวลาให้ garbage collection ทำงาน (โดย default reflog เก็บข้อมูลไว้อย่างน้อย 90 วันสำหรับ commit ที่ reachable และ 30 วันสำหรับที่ unreachable)

---

## Step 390: แบบฝึกหัด — ทำความสะอาดประวัติ 6 commit ที่ยุ่งเหยิง

ถึงเวลาลงมือปฏิบัติจริง เราจะจำลองสถานการณ์ที่พบบ่อยที่สุดในการทำงานจริง: คุณพัฒนาฟีเจอร์หนึ่งไปเรื่อย ๆ แล้ว commit บ่อยมากแบบไม่ได้คิดถึงความสวยงามของประวัติเลย จนได้ commit ที่มีชื่อแบบ "wip", "fix typo", "oops" ปะปนกัน ก่อนส่ง Pull Request คุณต้องทำความสะอาดมันให้เรียบร้อยก่อน

### 390.1 เตรียม repository ฝึกฝน

```bash
mkdir -p ~/git-course/part-39-interactive-rebase
cd ~/git-course/part-39-interactive-rebase
git init
git config user.name "ผู้ฝึกหัด Git"
git config user.email "practice@example.com"
```

### 390.2 สร้าง 6 commit ที่ยุ่งเหยิงตามโจทย์

```bash
echo "# Todo App" > README.md
git add README.md
git commit -m "initial commit"

mkdir src
echo "function addTodo(text) { return { text, done: false }; }" > src/todo.js
git add src/todo.js
git commit -m "wip"

echo "function removeTodo(list, index) { list.splice(index, 1); return list; }" >> src/todo.js
git add src/todo.js
git commit -m "wip todo functions"

sed -i 's/functoin/function/' src/todo.js 2>/dev/null || true
echo "function toggleTodo(todo) { todo.done = !todo.done; return todo; }" >> src/todo.js
git add src/todo.js
git commit -m "fix typo"

echo "module.exports = { addTodo, removeTodo, toggleTodo };" >> src/todo.js
git add src/todo.js
git commit -m "oops forgot exports"

cat > src/todo.test.js << 'EOF'
const { addTodo, removeTodo, toggleTodo } = require('./todo');

test('addTodo creates a todo item', () => {
  expect(addTodo('learn git')).toEqual({ text: 'learn git', done: false });
});
EOF
git add src/todo.test.js
git commit -m "add tests finally"
```

ตรวจสอบผลลัพธ์:

```bash
git log --oneline
```

ผลลัพธ์ที่ควรได้ (hash จะต่างกันในเครื่องของคุณ):

```
f6e5d4c (HEAD -> main) add tests finally
b5c4d3e oops forgot exports
a4b3c2d fix typo
93c2b1a wip todo functions
82b1a09 wip
71a0918 initial commit
```

นี่คือประวัติที่ยุ่งเหยิงตามโจทย์ — มี "wip" สองอัน "fix typo" หนึ่งอัน "oops" หนึ่งอัน ที่ล้วนแล้วแต่เป็นส่วนหนึ่งของการพัฒนาโมดูล `todo.js` เรื่องเดียวกันทั้งหมด

### 390.3 วางแผนก่อนลงมือ (สิ่งสำคัญที่มือใหม่มักข้าม)

ก่อนเปิด interactive rebase ให้วางแผนผลลัพธ์ที่ต้องการก่อนเสมอ ในกรณีนี้เป้าหมายที่สมเหตุสมผลคือ:

1. เก็บ `initial commit` แยกไว้ (เป็นจุดเริ่มต้นโปรเจกต์ ไม่เกี่ยวกับฟีเจอร์ todo)
2. รวม `wip`, `wip todo functions`, `fix typo`, `oops forgot exports` ทั้ง 4 commit เข้าเป็นก้อนเดียว เพราะทั้งหมดเป็นการพัฒนาโมดูล `todo.js` แบบต่อเนื่องกัน มีความหมายจริง ๆ แค่เรื่องเดียว: "สร้างฟังก์ชันจัดการ todo"
3. เก็บ `add tests finally` แยกไว้ต่างหาก แต่เปลี่ยนชื่อให้สื่อความหมายดีกว่าเดิม

### 390.4 เริ่มทำ interactive rebase

มี 5 commit หลัง initial commit ที่ต้องจัดการ (ไม่รวม initial commit เอง เพราะเราจะไม่แตะมัน) ดังนั้นใช้:

```bash
git rebase -i HEAD~5
```

editor จะแสดง:

```
pick 82b1a09 wip
pick 93c2b1a wip todo functions
pick a4b3c2d fix typo
pick b5c4d3e oops forgot exports
pick f6e5d4c add tests finally
```

แก้ไขเป็น:

```
pick 82b1a09 wip
squash 93c2b1a wip todo functions
squash a4b3c2d fix typo
squash b5c4d3e oops forgot exports
reword f6e5d4c add tests finally
```

**อธิบายเหตุผลของแต่ละบรรทัด:**

- `82b1a09 wip` ใช้ `pick` เพราะเป็น commit แรกสุดของกลุ่มที่จะกลายเป็น "จุดรวม" ให้ 3 commit ถัดไปมารวมด้วย
- `93c2b1a`, `a4b3c2d`, `b5c4d3e` ใช้ `squash` ทั้งหมด เพราะต้องการรวมเข้ากับ `82b1a09` และต้องการเห็น message ทั้งหมดตอนเขียนสรุปใหม่ (ใช้ `squash` ไม่ใช่ `fixup` เพราะอยากอ่านทวนข้อความเดิมประกอบการเขียนสรุปให้ครบถ้วน)
- `f6e5d4c add tests finally` ใช้ `reword` เพราะต้องการแก้แค่ข้อความ ไม่ต้องการรวมเข้ากับ commit อื่น (ตั้งใจให้อยู่เป็น commit แยกต่างหาก)

save และปิด editor

### 390.5 จัดการหน้าจอ squash message

Git จะเปิด editor แสดงข้อความทั้ง 4 commit ที่ถูกรวม:

```
# This is a combination of 4 commits.
# This is the 1st commit message:

wip

# This is the commit message #2:

wip todo functions

# This is the commit message #3:

fix typo

# This is the commit message #4:

oops forgot exports
```

ลบทั้งหมดแล้วเขียนใหม่ให้สื่อความหมายชัดเจน:

```
feat: add todo CRUD functions (add, remove, toggle)

Implement core todo management functions:
- addTodo: create a new todo item
- removeTodo: remove a todo item by index
- toggleTodo: toggle the done state of a todo item

Export all functions from the module.
```

save แล้วปิด editor

### 390.6 จัดการหน้าจอ reword message

ต่อมา Git จะหยุดที่ commit `add tests finally` เพื่อให้แก้ไขข้อความ:

```
add tests finally
```

แก้เป็น:

```
test: add unit test for addTodo function
```

save แล้วปิด editor กระบวนการ rebase จะเสร็จสมบูรณ์พร้อมข้อความ:

```
Successfully rebased and updated refs/heads/main.
```

### 390.7 ตรวจสอบผลลัพธ์

```bash
git log --oneline
```

ผลลัพธ์ที่ควรได้:

```
9k8j7h6 (HEAD -> main) test: add unit test for addTodo function
3d2c1b0 feat: add todo CRUD functions (add, remove, toggle)
71a0918 initial commit
```

จาก 6 commit ที่ยุ่งเหยิง เหลือเพียง **3 commit ที่มีความหมายชัดเจนทุกอันและอ่านแล้วเข้าใจเรื่องราวการพัฒนาได้ทันที**

ตรวจสอบเนื้อหาให้แน่ใจว่าไม่มีอะไรหายไประหว่างทางด้วย:

```bash
git show 3d2c1b0 --stat
cat src/todo.js
```

ควรเห็นฟังก์ชันทั้ง 3 (`addTodo`, `removeTodo`, `toggleTodo`) และบรรทัด `module.exports` ครบถ้วนตามที่ commit เดิมทั้ง 4 อันเคยเพิ่มเข้ามาทีละส่วน

### 390.8 ทดลองสถานการณ์ผิดพลาดและกู้คืนด้วย reflog (ฝึกความมั่นใจ)

เพื่อฝึกให้มั่นใจว่าคุณกู้คืนได้เสมอถ้าพลาด ลองทำสิ่งนี้:

```bash
git reflog
```

จะเห็นรายการทั้งหมดของสิ่งที่เพิ่งเกิดขึ้น รวมถึงจุดก่อนเริ่ม rebase (มองหาบรรทัดที่มีคำว่า `rebase -i (start)`) ลองสมมติว่าคุณเปลี่ยนใจอยากกลับไปดูประวัติ 6 commit เดิม:

```bash
git reset --hard HEAD@{6}   # เปลี่ยนตัวเลขให้ตรงกับ reflog ของคุณเอง
git log --oneline           # ควรเห็น 6 commit เดิมกลับมาครบ
```

จากนั้นกลับมายังผลลัพธ์ที่สะอาดอีกครั้ง:

```bash
git reset --hard HEAD@{0}   # หรือ hash ของ commit ล่าสุดหลัง rebase สำเร็จ
```

แบบฝึกหัดนี้ยืนยันให้เห็นด้วยตัวเองว่า **`git reflog` คือเซฟตี้เน็ตที่แท้จริงของการทำงานกับ Git** — ตราบใดที่คุณยังไม่ push ทับหรือลบ local repository ทิ้งไป แทบไม่มีอะไรที่กู้คืนไม่ได้เลย

### 390.9 โจทย์เพิ่มเติมสำหรับฝึกฝนต่อ (ไม่บังคับ)

ถ้าต้องการฝึกฝนเพิ่มเติมให้คล่องขึ้น ลองทำสิ่งเหล่านี้กับ repository เดียวกัน:

1. ใช้ `edit` แก้ไข commit `feat: add todo CRUD functions` เพื่อเพิ่มฟังก์ชัน `clearAllTodos` เข้าไปในจุดนั้นโดยตรง (ไม่ใช่สร้าง commit ใหม่)
2. ทดลองใช้ `git commit --fixup=<hash>` และ `git rebase -i --autosquash` แทนการเลื่อนบรรทัดด้วยมือ
3. ทดลอง `drop` commit `initial commit` แล้วสังเกตว่าเกิดอะไรขึ้นกับ commit ที่เหลือ (คำใบ้: ถ้า README.md ไม่ถูกใช้อ้างอิงจากที่อื่น มักจะไม่เกิด conflict แต่ประวัติของโปรเจกต์จะไม่มีจุดเริ่มต้นที่ชัดเจนอีกต่อไป)
4. ลองสร้างสถานการณ์ที่ทำให้เกิด conflict ระหว่าง interactive rebase โดยตั้งใจแก้ไขบรรทัดเดียวกันในสอง commit ที่ถูก reorder สลับตำแหน่งกัน แล้วฝึกแก้ conflict นั้นให้เสร็จสมบูรณ์

---

## สรุป Part 39

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Interactive Rebase** (`git rebase -i HEAD~n`) คือเครื่องมือเขียนประวัติ commit ใหม่แบบละเอียดที่สุดของ Git ใช้ทำความสะอาดประวัติก่อนส่ง Pull Request
2. หน้าจอ editor ของ interactive rebase มีคำสั่งหลักคือ `pick`, `reword`, `edit`, `squash`, `fixup`, `exec`, `drop`, `break` และคำสั่งขั้นสูง `label`/`reset`/`merge` ที่ใช้กับ `--rebase-merges`
3. **Squash** รวม commit หลายอันเป็นหนึ่ง โดยเก็บ message ของทุก commit ให้เลือก/แก้ไขรวมกัน ส่วน **Fixup** ทำแบบเดียวกันแต่ทิ้ง message ของ commit ที่ถูกรวมไปเลย
4. **Reword** แก้ไขแค่ commit message โดยไม่แตะเนื้อหาไฟล์ ส่วน **Edit** หยุดกระบวนการทั้งหมดเพื่อให้แก้ไขไฟล์ แทรก commit ใหม่ หรือแยก commit ออกเป็นหลายส่วนได้อย่างอิสระ
5. การ **Reorder** ทำได้ด้วยการสลับตำแหน่งบรรทัดในรายการ todo แต่มีความเสี่ยงสูงที่จะเกิด conflict ถ้า commit ที่สลับแก้ไขไฟล์ในบริเวณที่เกี่ยวข้องกัน
6. `git commit --fixup=<hash>` คู่กับ `git rebase -i --autosquash` (หรือตั้งค่า `rebase.autosquash true`) เป็นเวิร์กโฟลว์ที่มืออาชีพใช้เพื่อลดขั้นตอนการจัดเรียง fixup ด้วยมือ
7. **อันตรายที่แท้จริงของ interactive rebase คือ hash ของทุก commit ที่ตามหลังจุดที่แก้ไขจะเปลี่ยนทั้งหมด** เพราะ hash คำนวณรวม parent hash เข้าไปด้วย — และกฎทองจาก Part 38 ยังคงใช้ได้เสมอ: **ห้าม rebase ประวัติที่ push ไปแล้วและมีคนอื่นดึงไปใช้ต่อ** ถ้าจำเป็นต้อง force push ให้ใช้ `git push --force-with-lease` แทน `--force` เสมอ และจำไว้ว่า `git reflog` คือเซฟตี้เน็ตที่ช่วยกู้คืนได้แทบทุกสถานการณ์
8. ฝึกทำความสะอาดประวัติ 6 commit ที่มี "wip", "fix typo", "oops" ให้กลายเป็น 3 commit ที่มีความหมายชัดเจน ด้วยการผสมผสาน `pick`, `squash`, `reword` เข้าด้วยกันในการ rebase ครั้งเดียว

### Checklist ก่อนไป Part 40

ก่อนไปต่อ Part 40 ให้ตรวจสอบว่าคุณ:

- [ ] เข้าใจความแตกต่างระหว่าง rebase ธรรมดา (Part 38) กับ interactive rebase
- [ ] อธิบายหน้าที่ของคำสั่งทั้งหมดใน todo list ได้: pick, reword, edit, squash, fixup, drop, exec, break
- [ ] ทำ squash รวมหลาย commit เป็นหนึ่งได้จริงด้วยตัวเอง
- [ ] ทำ reword แก้ commit message เก่าได้โดยไม่กระทบเนื้อหาไฟล์
- [ ] ทำ edit หยุดแก้ไขเนื้อหาไฟล์ที่ commit เก่า แล้ว `git rebase --continue` ต่อได้
- [ ] สลับลำดับ commit (reorder) ได้ และเข้าใจความเสี่ยงเรื่อง conflict ที่มาพร้อมกับมัน
- [ ] drop commit ที่ไม่ต้องการออกจากประวัติได้อย่างถูกต้อง
- [ ] อธิบายความแตกต่างระหว่าง squash กับ fixup ได้ชัดเจน และรู้จักใช้ `--fixup` กับ `--autosquash`
- [ ] เข้าใจอย่างลึกซึ้งว่าทำไม interactive rebase ถึงเปลี่ยน hash ของ commit ที่ตามมาทั้งหมด และท่องกฎทองได้ขึ้นใจ: ห้าม rebase ประวัติที่แชร์กับคนอื่นไปแล้ว
- [ ] รู้จักใช้ `git push --force-with-lease` แทน `git push --force`
- [ ] รู้จักใช้ `git reflog` เพื่อกู้คืนสถานะก่อน rebase ได้เมื่อจำเป็น
- [ ] ทำแบบฝึกหัดทำความสะอาดประวัติ 6 commit ที่ยุ่งเหยิงให้เหลือประวัติที่สะอาดและมีความหมายสำเร็จด้วยตัวเอง

**ต่อไป:** [Part 40: Cherry-pick และการย้าย Commit ข้าม Branch](./part-040-cherry-pick.md)
