# Part 42: Git Bisect: หาบั๊กด้วยวิธี Binary Search

> **Step ในหลักสูตรนี้:** Step 411–420
> **เฟส:** 4 — ทำงานเป็นทีมด้วย Workflow มาตรฐาน (Git Flow, GitHub Flow, Rebase)
> **เป้าหมายของ Part นี้:** เข้าใจว่า `git bisect` คืออะไร แก้ปัญหาอะไร และเรียนรู้หลักการ binary search ที่อยู่เบื้องหลังมัน จนสามารถใช้ `git bisect` ทั้งแบบ manual ทีละรอบและแบบอัตโนมัติผ่าน `git bisect run` เพื่อหา commit ที่เป็นต้นเหตุของบั๊กในประวัติที่มีหลายร้อยหรือหลายพัน commit ได้อย่างแม่นยำและรวดเร็ว

---

## สารบัญของ Part นี้

- Step 411: `git bisect` คืออะไร แก้ปัญหาอะไร
- Step 412: หลักการ Binary Search ที่ `git bisect` ใช้
- Step 413: เริ่มต้น Bisect (`git bisect start`, `git bisect bad`, `git bisect good`)
- Step 414: ขั้นตอนทำ Bisect ทีละรอบด้วยมือ
- Step 415: `git bisect run <script>` — อัตโนมัติทั้งกระบวนการ
- Step 416: การอ่านผลลัพธ์และหา "First Bad Commit"
- Step 417: `git bisect reset` — กลับสู่สถานะปกติ
- Step 418: `git bisect skip` — ข้าม Commit ที่ทดสอบไม่ได้
- Step 419: เทคนิคเขียน Test Script ที่ดีสำหรับ `bisect run`
- Step 420: แบบฝึกหัด — จำลองประวัติที่มีบั๊กแอบแฝง แล้วใช้ Bisect หาให้เจอ

---

## Step 411: `git bisect` คืออะไร แก้ปัญหาอะไร

### ปัญหาที่เกิดขึ้นจริงในทีมพัฒนา

ลองนึกภาพสถานการณ์นี้ ซึ่งเกิดขึ้นแทบทุกทีมที่ทำงานกับโปรเจกต์ใหญ่มานานหลายเดือนหรือหลายปี:

> QA รายงานว่า "ฟีเจอร์ export รายงานเป็น PDF พังตั้งแต่เมื่อไหร่ก็ไม่รู้ ตอนนี้กดแล้วได้ไฟล์เปล่า" โปรเจกต์นี้มี commit สะสมมาแล้วกว่า **800 commit** นับตั้งแต่ release ก่อนหน้าที่ยืนยันว่าฟีเจอร์นี้ยังทำงานปกติ

คำถามคือ **commit ไหนใน 800 commit นี้ที่เป็นต้นเหตุของบั๊ก?**

ถ้าไม่มีเครื่องมือช่วย คุณอาจจะต้อง:

1. อ่านโค้ด diff ทีละ commit ไล่ตั้งแต่ต้นจนจบ (ใช้เวลาเป็นวัน)
2. เดามั่ว ๆ ว่าน่าจะเป็น commit ที่แก้ไฟล์ `pdf_exporter.py` แล้วไป checkout มาไล่ดูทีละอันจากบนลงล่าง (linear search) ซึ่งในกรณีเลวร้ายที่สุดต้องเช็คถึง 800 ครั้ง
3. ถามเพื่อนร่วมทีมทีละคนว่า "จำได้ไหมว่าใครแก้ตรงนี้" ซึ่งไม่แม่นยำและเสียเวลา

วิธีเหล่านี้ **ใช้ได้ในทางทฤษฎี แต่ช้ามากในทางปฏิบัติ** และยิ่งจำนวน commit เยอะขึ้นเท่าไหร่ เวลาที่เสียไปก็ยิ่งมากขึ้นแบบเป็นเส้นตรง (linear)

### `git bisect` คือคำตอบ

**`git bisect`** คือเครื่องมือในตัว Git ที่ออกแบบมาเพื่อแก้ปัญหานี้โดยเฉพาะ มันช่วยให้คุณหา **commit แรกที่ทำให้เกิดพฤติกรรมที่ไม่ต้องการ** (ไม่ว่าจะเป็นบั๊ก, performance regression, หรือแม้แต่ test ที่ล้มเหลว) ได้อย่างรวดเร็ว โดยใช้หลักการที่เรียกว่า **binary search**

แนวคิดหลักคือ:

> **แทนที่จะเช็คทีละ commit จากต้นจนจบ (linear) ให้เช็คตรงกลางของช่วงที่สงสัยก่อน แล้วตัดครึ่งที่ไม่เกี่ยวข้องทิ้งไปทุกรอบ**

ผลลัพธ์คือ จากที่ต้องเช็คสูงสุด 800 ครั้งในกรณีเลวร้ายที่สุด (linear search) จะเหลือเพียง **ประมาณ 10 ครั้ง** เท่านั้น (เพราะ log₂(800) ≈ 9.6) ซึ่งเป็นความแตกต่างที่มหาศาลมากในการทำงานจริง

### `git bisect` แก้ปัญหาอะไรบ้าง

| สถานการณ์ | ตัวอย่าง |
|---|---|
| **หา commit ที่ทำให้เกิดบั๊ก (regression)** | ฟีเจอร์ที่เคยทำงานได้ดี จู่ ๆ ก็พังหลังจากผ่านไปหลายร้อย commit |
| **หา commit ที่ทำให้ performance แย่ลง** | เวลาตอบสนองของ API เพิ่มขึ้นจาก 50ms เป็น 2000ms โดยไม่รู้สาเหตุ |
| **หา commit ที่ทำให้ test suite ล้มเหลว** | Test เคยผ่านทั้งหมด แต่ตอนนี้มี test หนึ่งตัวล้มเหลวแบบไม่รู้สาเหตุ |
| **หา commit ที่ทำให้ build พัง** | โปรเจกต์ build ผ่านมาตลอด แต่จู่ ๆ ก็ compile ไม่ผ่านหลังจาก merge หลายครั้ง |
| **หา commit ที่ทำให้พฤติกรรมเปลี่ยนไปโดยไม่ตั้งใจ** | ค่า default ของ config เปลี่ยนไปโดยไม่มีใครรู้ตัวว่าใครเป็นคนแก้ |

### ทำไมต้องใช้ `git bisect` แทนการอ่าน `git log` เฉย ๆ

หลายคนอาจคิดว่า "ก็แค่ `git log -p` แล้วไล่อ่าน diff ทีละอันสิ" ซึ่งใช้ได้ถ้ามี 5-10 commit แต่:

- ถ้ามี commit จำนวนมาก การอ่าน diff ทุกอันเป็น**การเสียเวลามหาศาล**และมีโอกาสพลาดสูง (อ่านผ่าน ๆ แล้วมองข้ามจุดที่เป็นต้นเหตุ)
- บั๊กบางอย่าง**ไม่ได้เห็นได้จากการอ่านโค้ดตรง ๆ** ต้องรันโปรแกรมจริง ๆ ถึงจะเห็นอาการ เช่น race condition, memory leak, หรือค่าที่คำนวณผิดแบบ edge case
- `git bisect` **ไม่ต้องการให้คุณรู้ล่วงหน้าว่า commit ไหนน่าสงสัย** — มันจะพาคุณไปเจอ commit ที่ถูกต้องแม้ว่าคุณจะคาดไม่ถึงเลยว่าไฟล์นั้นเกี่ยวข้องกับบั๊ก

### จุดแข็งที่สำคัญที่สุดของ `git bisect`

1. **เป็นระบบ (systematic)** — ไม่ต้องเดา ไม่ต้องอาศัยความจำ ทำตามขั้นตอนที่ชัดเจนก็เจอคำตอบแน่นอน
2. **เร็วในทางคณิตศาสตร์** — จำนวนรอบที่ต้องเช็คโตแบบ logarithmic ไม่ใช่ linear
3. **อัตโนมัติได้เต็มรูปแบบ** — ด้วย `git bisect run` สามารถปล่อยให้ Git ทำงานทั้งหมดโดยไม่ต้องมีคนนั่งเฝ้า
4. **ทำงานร่วมกับทุกอย่างที่ Git จัดการได้** — ไม่ว่าจะเป็น branch ไหน หรือ commit history แบบไหนก็ใช้ได้ ตราบใดที่ประวัติเป็นเส้นตรงพอที่จะไล่ตาม parent ได้

ใน Step ถัดไป เราจะเจาะลึกหลักการ binary search ที่อยู่เบื้องหลังการทำงานของ `git bisect` ให้เข้าใจอย่างถ่องแท้ ก่อนจะลงมือใช้งานจริง

---

## Step 412: หลักการ Binary Search ที่ `git bisect` ใช้

### Binary Search คืออะไร (ทบทวนแนวคิดพื้นฐาน)

**Binary Search** (การค้นหาแบบทวิภาค) เป็นอัลกอริทึมพื้นฐานที่ใช้หาค่าในชุดข้อมูลที่ **เรียงลำดับแล้ว** โดยหลักการคือ:

1. เช็คตรงกลางของชุดข้อมูลก่อน
2. ถ้าค่ากลางไม่ใช่คำตอบ ให้ดูว่าคำตอบควรอยู่ครึ่งซ้ายหรือครึ่งขวา แล้ว**ตัดอีกครึ่งทิ้งไปทั้งหมด**
3. ทำซ้ำขั้นตอนที่ 1-2 กับครึ่งที่เหลือ จนกว่าจะเจอคำตอบ

ตัวอย่างคลาสสิกคือการทายเลข 1-100: ถ้าคุณทาย 50 แล้วได้คำใบ้ว่า "สูงไป" คุณก็รู้ทันทีว่าตัดตัวเลือก 50-100 ทิ้งได้เลยครึ่งหนึ่ง เหลือแค่ 1-49 ให้ทายต่อ

### ทำไม Binary Search ถึงเร็วกว่า Linear Search มาก

สมมติมี commit ทั้งหมด **N** commit ที่ต้องหาว่าอันไหนเป็นต้นเหตุของบั๊ก:

| วิธีการ | จำนวนครั้งที่ต้องเช็ค (กรณีเลวร้ายที่สุด) | สูตร |
|---|---|---|
| **Linear Search** (ไล่ทีละ commit) | N ครั้ง | O(N) |
| **Binary Search** (ตัดครึ่งทุกรอบ) | log₂(N) ครั้ง (ปัดขึ้น) | O(log N) |

ตารางเปรียบเทียบตัวเลขจริง:

| จำนวน Commit (N) | Linear Search (เลวร้ายสุด) | Binary Search (`git bisect`) |
|---|---|---|
| 10 | 10 ครั้ง | 4 ครั้ง |
| 100 | 100 ครั้ง | 7 ครั้ง |
| 800 | 800 ครั้ง | 10 ครั้ง |
| 1,000 | 1,000 ครั้ง | 10 ครั้ง |
| 10,000 | 10,000 ครั้ง | 14 ครั้ง |
| 1,000,000 | 1,000,000 ครั้ง | 20 ครั้ง |

จะเห็นว่ายิ่งจำนวน commit มากเท่าไหร่ ความแตกต่างยิ่งมหาศาลขึ้นเรื่อย ๆ นี่คือเหตุผลว่าทำไม Linux Kernel ที่มี commit หลักแสนถึงหลักล้าน ยังสามารถใช้ `git bisect` หาบั๊กได้ภายในไม่ถึง 20 รอบ

### เงื่อนไขสำคัญที่ทำให้ Binary Search ใช้งานได้กับ `git bisect`

Binary search จะใช้ได้ก็ต่อเมื่อข้อมูลมีคุณสมบัติ **monotonic** (เปลี่ยนแปลงไปในทิศทางเดียวไม่กลับไปกลับมา) กล่าวคือ:

> **สมมติฐานหลักของ `git bisect`:** เมื่อไล่ประวัติ commit จาก "จุดที่ดี (good)" ไปยัง "จุดที่เสีย (bad)" จะมี**จุดเปลี่ยนผ่านเพียงจุดเดียว** — ก่อนจุดนี้ทุก commit จะ "good" ทั้งหมด และหลังจุดนี้ทุก commit จะ "bad" ทั้งหมด

```
good ────────────────●──────────────── bad
                (first bad commit)

[good, good, good, good, good] | [bad, bad, bad, bad, bad]
                                ↑
                    จุดเปลี่ยนผ่านที่เราต้องการหา
```

ถ้าสมมติฐานนี้เป็นจริง `git bisect` จะทำงานได้อย่างสมบูรณ์แบบ แต่ถ้าบั๊ก**เกิด ๆ หาย ๆ** (flaky/intermittent bug) เช่น commit บางอันมีบั๊กแบบ intermittent ที่บางครั้ง pass บางครั้ง fail สมมติฐานนี้จะถูกทำลาย และผลลัพธ์ของ bisect อาจไม่แม่นยำ (เราจะพูดถึงวิธีรับมือกับกรณีนี้ใน Step 419)

### กระบวนการ Binary Search ของ `git bisect` แบบเป็นขั้นตอน

สมมติเรามี commit เรียงจากเก่าไปใหม่ ดังนี้ (commit 1 คือ good ที่รู้แน่นอน, commit 16 คือ bad ที่รู้แน่นอน):

```
1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16
good                                          bad
```

**รอบที่ 1:** Git เลือก commit ตรงกลางระหว่าง 1 กับ 16 คือประมาณ commit 8 หรือ 9 ให้ checkout ไปทดสอบ

```
1  2  3  4  5  6  7  [8]  9  10 11 12 13 14 15 16
good                 ?                        bad
```

สมมติเราทดสอบแล้วพบว่า commit 8 **good** (บั๊กยังไม่เกิด) แปลว่าบั๊กต้องอยู่ใน commit 9-16 เท่านั้น เราตัด commit 1-8 ทิ้งได้ทั้งหมด

```
                     9  10 11 12 13 14 15 16
                     good?...              bad
```

**รอบที่ 2:** Git เลือกตรงกลางของช่วง 9-16 คือประมาณ commit 12 หรือ 13

```
9  10 11 [12]  13 14 15 16
good      ?             bad
```

สมมติทดสอบแล้วพบว่า commit 12 **bad** แปลว่าบั๊กต้องเกิดขึ้นในช่วง 9-12 เราตัด commit 13-16 ทิ้งได้

```
9  10 11 12
good     bad
```

**รอบที่ 3:** เช็คตรงกลางของ 9-12 คือ commit 10 หรือ 11 ทำซ้ำแบบนี้ไปเรื่อย ๆ จนกว่าช่วงที่เหลือจะแคบลงเหลือ good กับ bad ที่อยู่ติดกัน — นั่นคือ **bad commit ตัวแรก** ที่เราตามหา

### สิ่งที่ Git ทำให้อัตโนมัติ

ผู้ใช้**ไม่จำเป็นต้องคำนวณเองว่า commit ไหนคือ "ตรงกลาง"** — Git จะจัดการเรื่องนี้ให้ทั้งหมด สิ่งที่ผู้ใช้ต้องทำมีเพียง:

1. บอก Git ว่า commit ไหนเป็น **good** (จุดที่ยืนยันว่ายังไม่มีบั๊ก)
2. บอก Git ว่า commit ไหนเป็น **bad** (จุดที่ยืนยันว่ามีบั๊กแล้ว)
3. ทดสอบ commit ที่ Git checkout ให้ในแต่ละรอบ แล้วบอกผลกลับไปว่า good หรือ bad
4. ทำซ้ำข้อ 3 จนกว่า Git จะประกาศผลลัพธ์สุดท้าย

Git จะแสดงข้อความบอกจำนวนรอบที่เหลือโดยประมาณทุกครั้ง เช่น:

```
Bisecting: 398 revisions left to test after this (roughly 9 steps)
```

ข้อความนี้บอกว่า ในบรรดา commit ที่ยังไม่ถูกตัดออกไป เหลือ 398 commit ที่อาจเป็นตัวการ และคาดว่าจะใช้เวลาอีกประมาณ 9 รอบในการหาคำตอบสุดท้าย (คำนวณจาก log₂ ของจำนวนที่เหลือ)

ใน Step ถัดไปเราจะเริ่มลงมือใช้คำสั่ง `git bisect` จริง ๆ

---

## Step 413: เริ่มต้น Bisect (`git bisect start`, `git bisect bad`, `git bisect good`)

### ขั้นตอนที่ 1: `git bisect start`

คำสั่งแรกสุดที่ต้องรันคือ:

```bash
git bisect start
```

คำสั่งนี้จะเข้าสู่ **"bisect mode"** ซึ่งเป็นสถานะพิเศษของ repository ระหว่างที่อยู่ใน mode นี้ Git จะสร้างไฟล์ภายใน `.git/` เพื่อติดตามสถานะของกระบวนการ bisect ได้แก่:

| ไฟล์/Ref ภายใน `.git/` | หน้าที่ |
|---|---|
| `.git/BISECT_START` | บันทึกว่า HEAD เดิมอยู่ที่ไหนก่อนเริ่ม bisect (ใช้ตอน `reset`) |
| `.git/BISECT_LOG` | บันทึก log ของทุกคำสั่ง good/bad/skip ที่รันไป |
| `.git/BISECT_NAMES` | เก็บชื่อ ref ที่ใช้เป็น good/bad |
| `.git/BISECT_TERMS` | เก็บคำศัพท์ที่ใช้ (ปกติคือ good/bad) |
| `refs/bisect/*` | reference ชั่วคราวที่ชี้ไปยัง commit good/bad ต่าง ๆ |

**ข้อควรระวังก่อนเริ่ม:** ควรตรวจสอบว่า working directory สะอาด (ไม่มีการเปลี่ยนแปลงที่ยัง commit) ก่อนเริ่ม bisect เพราะ Git จะ checkout ไปมาระหว่าง commit หลายครั้งตลอดกระบวนการ

```bash
git status
# ควรเห็น "nothing to commit, working tree clean"
```

ถ้ามีการเปลี่ยนแปลงค้างอยู่ ให้ `git stash` หรือ `git commit` ก่อน

### ขั้นตอนที่ 2: บอกจุดที่ "bad" (มีบั๊กแน่นอน)

```bash
git bisect bad
```

ถ้าไม่ระบุ commit ต่อท้าย Git จะถือว่า **HEAD ปัจจุบัน** คือจุด bad ซึ่งมักจะสมเหตุสมผล เพราะโดยทั่วไปเราจะเริ่ม bisect ตอนที่รู้อยู่แล้วว่าโค้ดปัจจุบัน (branch ล่าสุด) มีบั๊ก

หรือถ้าต้องการระบุ commit อื่นที่ไม่ใช่ HEAD ก็ทำได้:

```bash
git bisect bad <commit-hash>
```

### ขั้นตอนที่ 3: บอกจุดที่ "good" (ยืนยันว่ายังไม่มีบั๊ก)

```bash
git bisect good <commit-hash>
```

จุดนี้สำคัญมาก — คุณต้องรู้ **commit หรือ tag ในอดีตที่ยืนยันแล้วว่ายังไม่มีบั๊กนี้** เช่น อาจเป็น:

- Tag ของ release เวอร์ชันก่อนหน้าที่ยังใช้งานได้ปกติ: `git bisect good v2.3.0`
- Commit hash เฉพาะที่จำได้ว่าเคยทดสอบแล้วผ่าน: `git bisect good a1b2c3d`
- วันที่ที่รู้ว่าฟีเจอร์ยังทำงานปกติ (ใช้ร่วมกับ `git log --before` เพื่อหา commit hash ก่อน)

**ตัวอย่างการใช้งานแบบเต็ม:**

```bash
git bisect start
git bisect bad HEAD
git bisect good v2.3.0
```

### วิธีย่อ: ระบุ bad และ good พร้อมกันใน `bisect start`

Git อนุญาตให้ระบุทั้ง bad และ good ได้ในคำสั่งเดียวตอน start เพื่อความรวดเร็ว:

```bash
git bisect start HEAD v2.3.0
#                ^^^^ bad  ^^^^^^ good
```

รูปแบบคือ `git bisect start [<bad>] [<good>...]` — สามารถระบุ **good ได้หลายจุด** ด้วย (มีประโยชน์เมื่อประวัติมีหลาย branch มาบรรจบกัน และต้องการยืนยันว่าทุกจุดเริ่มต้นเป็น good จริง) เช่น:

```bash
git bisect start HEAD v2.3.0 v2.2.5
```

### สิ่งที่เกิดขึ้นทันทีหลังระบุ good ครบ

เมื่อ Git มีทั้งจุด **bad** อย่างน้อย 1 จุด และจุด **good** อย่างน้อย 1 จุดแล้ว มันจะเริ่มกระบวนการ binary search ทันที โดย:

1. คำนวณ commit ที่อยู่ "กึ่งกลาง" ของช่วงระหว่าง good กับ bad (ใช้ revision walk ผ่าน parent/merge graph จริง ไม่ใช่แค่นับจำนวนบรรทัดบน timeline)
2. `checkout` ไปยัง commit นั้นโดยอัตโนมัติ (detached HEAD state)
3. แสดงข้อความบอกจำนวนรอบที่เหลือโดยประมาณ

ตัวอย่าง output ที่จะเห็น:

```
Bisecting: 12 revisions left to test after this (roughly 4 steps)
[3f2a91c4e5d6...] Fix typo in changelog
```

หมายความว่าตอนนี้ Git ได้ checkout ไปยัง commit `3f2a91c` ให้แล้ว และรอให้คุณไปทดสอบว่า commit นี้มีบั๊กหรือไม่

### คำเตือนสำคัญ: Detached HEAD

ระหว่างอยู่ใน bisect mode คุณจะอยู่ในสถานะ **detached HEAD** เสมอ (คือ HEAD ไม่ได้ชี้ไปที่ branch ใด ๆ แต่ชี้ตรงไปยัง commit) นี่เป็นเรื่องปกติของกระบวนการนี้ **ห้าม commit งานใหม่ระหว่างอยู่ใน bisect mode** เพราะ commit ที่สร้างขึ้นตอน detached HEAD จะไม่ได้ผูกกับ branch ใด และจะถูกลืมได้ง่ายเมื่อออกจาก bisect mode (เว้นแต่จะไปสร้าง branch ไว้ก่อน)

ใน Step ถัดไปเราจะมาดูขั้นตอนการทำ bisect ทีละรอบด้วยมือแบบละเอียด

---

## Step 414: ขั้นตอนทำ Bisect ทีละรอบด้วยมือ

เมื่อเริ่มต้น bisect และระบุ good/bad ครบแล้ว Git จะ checkout ไปยัง commit กึ่งกลางให้อัตโนมัติทุกครั้ง สิ่งที่คุณต้องทำในแต่ละรอบมีเพียง 3 ขั้นตอนวนซ้ำ:

```
┌─────────────────────────────────────────────┐
│  1. Git checkout commit กึ่งกลางให้อัตโนมัติ    │
│  2. คุณทดสอบด้วยตัวเอง (รันโปรแกรม, ดูผลลัพธ์)  │
│  3. บอกผลกลับไปด้วย good/bad                  │
│         ↓ วนซ้ำจนกว่าจะเจอคำตอบ                │
└─────────────────────────────────────────────┘
```

### รอบตัวอย่างแบบละเอียด

สมมติเรากำลังหาบั๊กในฟังก์ชันคำนวณราคาสินค้าหลังหักส่วนลด สมมติสถานการณ์เริ่มต้น:

```bash
$ git bisect start
$ git bisect bad HEAD
$ git bisect good v3.1.0
Bisecting: 27 revisions left to test after this (roughly 5 steps)
[9c8b7a6 ...] Refactor discount calculation module
```

**รอบที่ 1:** Git checkout ไปยัง commit `9c8b7a6` ให้แล้ว ตอนนี้เราต้องไปทดสอบว่าบั๊กนี้เกิดหรือยัง

```bash
# ทดสอบด้วยมือ: รันโปรแกรม เช็คว่าคำนวณราคาถูกต้องไหม
$ npm test -- --grep "discount calculation"
```

สมมติผลการทดสอบคือ **test ผ่าน (โค้ดยังปกติ)** เราจึงบอก Git ว่า commit นี้ good:

```bash
$ git bisect good
Bisecting: 13 revisions left to test after this (roughly 4 steps)
[4e5f6a7 ...] Add support for percentage-based coupons
```

**รอบที่ 2:** Git ขยับไป checkout commit ใหม่ให้อัตโนมัติ (`4e5f6a7`) เราทดสอบอีกครั้ง

```bash
$ npm test -- --grep "discount calculation"
# ครั้งนี้ test ล้มเหลว! ราคาที่คำนวณได้ผิด
```

บอก Git ว่า commit นี้ bad:

```bash
$ git bisect bad
Bisecting: 6 revisions left to test after this (roughly 3 steps)
[2b3c4d5 ...] Update coupon validation logic
```

**รอบที่ 3, 4, 5 ...** ทำซ้ำแบบเดียวกันไปเรื่อย ๆ — ทดสอบ แล้วบอก `good` หรือ `bad` — จนกระทั่ง Git ไม่มี commit เหลือให้เช็คแล้ว มันจะประกาศผลลัพธ์:

```bash
$ git bisect bad
2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c is the first bad commit
commit 2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c
Author: Somchai Devteam <somchai@example.com>
Date:   Tue Mar 4 14:22:10 2025 +0700

    Update coupon validation logic

 src/discount/coupon.js | 8 ++++----
 1 file changed, 4 insertions(+), 4 deletions(-)
```

นี่คือคำตอบสุดท้าย! **commit `2b3c4d5` คือต้นเหตุของบั๊ก**

### ตารางสรุปคำสั่งที่ใช้ในแต่ละรอบ

| สถานการณ์ที่พบหลังทดสอบ | คำสั่งที่ใช้ |
|---|---|
| Commit นี้**ไม่มี**บั๊ก (พฤติกรรมปกติ) | `git bisect good` |
| Commit นี้**มี**บั๊ก (พฤติกรรมผิดปกติ) | `git bisect bad` |
| Commit นี้ทดสอบไม่ได้ (เช่น build พังด้วยเหตุผลอื่น) | `git bisect skip` (จะพูดถึงใน Step 418) |

### การดูสถานะปัจจุบันระหว่างทำ bisect

ถ้าอยากรู้ว่าตอนนี้ bisect process ไปถึงไหนแล้ว ใช้:

```bash
git bisect log
```

คำสั่งนี้จะแสดงประวัติทุกคำสั่ง good/bad ที่รันไปแล้วตั้งแต่ต้น เช่น:

```
git bisect start
# bad: [a1b2c3d...] Latest commit on main
git bisect bad a1b2c3d
# good: [f9e8d7c...] Release v3.1.0
git bisect good f9e8d7c
# good: [9c8b7a6...] Refactor discount calculation module
git bisect good 9c8b7a6
# bad: [4e5f6a7...] Add support for percentage-based coupons
git bisect bad 4e5f6a7
```

ข้อดีของ `git bisect log` คือสามารถ **บันทึกเก็บไว้เป็นไฟล์** แล้วนำมา **replay** ใหม่ในภายหลังได้ (เช่น ถ้า session ถูกขัดจังหวะ หรืออยากแชร์ผลการวิเคราะห์ให้เพื่อนร่วมทีมดู):

```bash
git bisect log > bisect-discount-bug.log

# ในภายหลัง หรือในเครื่องอื่น สามารถ replay กลับมาที่จุดเดิมได้ทันที
git bisect replay bisect-discount-bug.log
```

`git bisect replay` มีประโยชน์มากในการทำงานเป็นทีม เช่น คนหนึ่งเริ่มทำ bisect ไปครึ่งทาง แล้วส่งไฟล์ log ให้อีกคนมาทำต่อ หรือแนบไฟล์ log ไว้ใน issue tracker เพื่อให้คนอื่นตรวจสอบผลลัพธ์ได้

### การดูภาพรวมของ commit ที่เหลือ

ถ้าต้องการดูว่าตอนนี้เหลือ commit ไหนบ้างที่ยังไม่ถูกตัดออก สามารถใช้:

```bash
git bisect visualize
```

หรือในบางเวอร์ชันเก่าใช้ชื่อ `git bisect view` (เป็น alias เดียวกัน) คำสั่งนี้จะเปิด `gitk` (ถ้าติดตั้งไว้) เพื่อแสดงกราฟของ commit ที่ยังเป็นผู้ต้องสงสัยอยู่ หรือถ้าไม่มี GUI จะ fallback ไปใช้ `git log` แสดงเป็น text แทน

ใน Step ถัดไป เราจะมาดูวิธีที่ทรงพลังกว่านั้นมาก คือการปล่อยให้ Git ทำงานทั้งหมดอัตโนมัติด้วย `git bisect run`

---

## Step 415: `git bisect run <script>` — อัตโนมัติทั้งกระบวนการ

### ปัญหาของการทำ bisect ด้วยมือ

การทำ bisect ทีละรอบด้วยมืออย่างที่เห็นใน Step 414 นั้นได้ผลดี แต่มีข้อจำกัด:

- ต้องนั่งเฝ้าหน้าจอตลอดกระบวนการ ไม่สามารถปล่อยทิ้งไว้ได้
- ถ้าการทดสอบแต่ละรอบใช้เวลานาน (เช่น ต้อง build โปรเจกต์ใหญ่ก่อนถึงจะทดสอบได้) จะเสียเวลารอมาก
- มีโอกาสพิมพ์ผิดหรือจำสถานะผิดพลาดได้ (โดยเฉพาะเมื่อมีหลายสิบรอบ)

### `git bisect run` คือคำตอบ

ถ้าการทดสอบว่า commit หนึ่ง ๆ มีบั๊กหรือไม่ **สามารถเขียนเป็นสคริปต์อัตโนมัติได้** (เช่น รัน unit test, รันสคริปต์ตรวจสอบผลลัพธ์) เราสามารถให้ Git รันสคริปต์นั้นเองในทุกรอบ โดยไม่ต้องมีคนเข้าไปยุ่งเลย:

```bash
git bisect start
git bisect bad HEAD
git bisect good v3.1.0
git bisect run <path-to-script>
```

**Git จะทำสิ่งเหล่านี้อัตโนมัติทั้งหมด:**

1. Checkout commit กึ่งกลาง
2. รันสคริปต์ที่ระบุ
3. ดู **exit code** ของสคริปต์ แล้วตัดสินใจเองว่าเป็น good, bad, หรือ skip
4. วนซ้ำจนกว่าจะได้คำตอบ
5. ประกาศผลลัพธ์ first bad commit ให้อัตโนมัติ

### ตัวอย่างการใช้งานแบบง่ายที่สุด

ถ้าโปรเจกต์มีชุดทดสอบอัตโนมัติอยู่แล้ว (เช่น npm test, pytest, go test) สามารถใช้คำสั่งทดสอบนั้นตรง ๆ ได้เลย:

```bash
git bisect start
git bisect bad HEAD
git bisect good v3.1.0
git bisect run npm test -- --grep "discount calculation"
```

Git จะรันคำสั่งนี้ในทุก commit ที่ checkout ไป และดูว่า `npm test` จบด้วย exit code 0 (ผ่าน = good) หรือ exit code ที่ไม่ใช่ศูนย์ (ล้มเหลว = bad)

### ตัวอย่างสคริปต์แบบกำหนดเอง (custom script)

บ่อยครั้งที่การทดสอบซับซ้อนกว่าการรันคำสั่งเดียว จึงนิยมเขียนเป็นสคริปต์แยกไว้ เช่น `test-bug.sh`:

```bash
#!/bin/bash
set -e

# 1. Build โปรเจกต์
npm install --silent > /dev/null 2>&1
npm run build > /dev/null 2>&1

# 2. รันโปรแกรมและตรวจสอบผลลัพธ์
RESULT=$(node ./scripts/calculate-discount.js --price=100 --coupon=SAVE20)

# 3. เปรียบเทียบผลลัพธ์กับค่าที่คาดหวัง
if [ "$RESULT" == "80" ]; then
    echo "PASS: got expected result $RESULT"
    exit 0   # good
else
    echo "FAIL: got $RESULT, expected 80"
    exit 1   # bad
fi
```

จากนั้นให้สิทธิ์ execute และรันผ่าน `bisect run`:

```bash
chmod +x test-bug.sh
git bisect start
git bisect bad HEAD
git bisect good v3.1.0
git bisect run ./test-bug.sh
```

### Output ที่จะเห็นระหว่างรัน

Git จะแสดง log ของทุกรอบให้เห็นแบบ real-time เช่น:

```
running './test-bug.sh'
FAIL: got 75, expected 80
Bisecting: 6 revisions left to test after this (roughly 3 steps)
[2b3c4d5 ...] Update coupon validation logic
running './test-bug.sh'
FAIL: got 75, expected 80
Bisecting: 2 revisions left to test after this (roughly 1 step)
[8f7e6d5 ...] Simplify percentage rounding
running './test-bug.sh'
PASS: got expected result 80
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[9a8b7c6 ...] Update coupon validation logic
running './test-bug.sh'
FAIL: got 75, expected 80
2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c is the first bad commit
```

เมื่อเสร็จสิ้น Git จะแสดงบรรทัดสุดท้ายที่บอกว่า `<commit-hash> is the first bad commit` ทันที เหมือนกับตอนทำด้วยมือทุกประการ เพียงแต่ไม่ต้องมีคนคอยกดคำสั่งเองในแต่ละรอบ

### ข้อดีของ `git bisect run`

| ข้อดี | รายละเอียด |
|---|---|
| **เร็วกว่ามาก** | ไม่ต้องรอคนตัดสินใจในแต่ละรอบ รันต่อเนื่องได้ทันที |
| **แม่นยำกว่า** | ไม่มีโอกาสพิมพ์ผิดหรือจำสถานะผิดเหมือนตอนทำด้วยมือ |
| **ปล่อยทิ้งไว้ได้** | เหมาะกับกรณีที่แต่ละรอบใช้เวลานาน (build ใหญ่ ๆ) — สั่งแล้วไปทำงานอื่นได้ |
| **ทำซ้ำได้ (reproducible)** | เก็บสคริปต์ไว้ใน repo แล้วใช้ตรวจสอบ regression แบบนี้ได้อีกในอนาคต |
| **ใช้ใน CI ได้** | สามารถเรียก `git bisect run` จาก CI pipeline เพื่อ auto-diagnose regression ได้ |

### ข้อควรระวังของ `git bisect run`

1. **สคริปต์ต้อง exit code ให้ถูกต้องเสมอ** — นี่คือหัวใจสำคัญที่สุด ถ้าสคริปต์คืนค่าผิด ผลลัพธ์ของ bisect ทั้งหมดจะผิดตามไปด้วย (รายละเอียดใน Step 419)
2. **สคริปต์ต้องทำงานได้กับทุก commit ในช่วงที่ bisect** — ถ้า commit เก่ามาก ๆ มี dependency หรือ API ที่เปลี่ยนไปจนสคริปต์รันไม่ได้เลย (ไม่ใช่เพราะบั๊กที่ตามหา แต่เพราะเหตุผลอื่น) ต้องใช้ exit code พิเศษเพื่อ skip (Step 418)
3. **หลีกเลี่ยง side effects ที่ค้างข้าม commit** — เช่น ไฟล์ cache หรือ build artifact เก่าที่หลงเหลือจาก commit ก่อนหน้าอาจทำให้ผลทดสอบผิดเพี้ยน ควร clean build ให้แน่ใจ

ใน Step ถัดไป เราจะมาดูวิธีอ่านผลลัพธ์สุดท้ายของ bisect ให้เข้าใจอย่างถูกต้อง

---

## Step 416: การอ่านผลลัพธ์และหา "First Bad Commit"

### รูปแบบผลลัพธ์สุดท้าย

ไม่ว่าจะทำ bisect ด้วยมือหรือใช้ `git bisect run` เมื่อกระบวนการเสร็จสมบูรณ์ Git จะพิมพ์ผลลัพธ์ในรูปแบบเดียวกันเสมอ:

```
2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c is the first bad commit
commit 2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c
Author: Somchai Devteam <somchai@example.com>
Date:   Tue Mar 4 14:22:10 2025 +0700

    Update coupon validation logic

 src/discount/coupon.js | 8 ++++----
 1 file changed, 4 insertions(+), 4 deletions(-)
```

ข้อมูลที่ Git แสดงให้ครบถ้วนคือ:

1. **Commit hash เต็ม** ของ "first bad commit"
2. **ข้อมูล commit** — Author, วันที่, commit message
3. **Diffstat** — สรุปว่าไฟล์ไหนถูกแก้ไขบ้างใน commit นี้ (บอกจำนวนบรรทัดที่เพิ่ม/ลบ)

### ความหมายของ "First Bad Commit"

คำว่า **"first bad commit"** หมายถึง **commit แรกสุด**ตามลำดับเวลา (ในสายประวัติที่ bisect ไล่ตาม) ที่เริ่มแสดงพฤติกรรมที่เราตัดสินว่าเป็น "bad" กล่าวคือ:

- Commit ก่อนหน้านี้ (parent) ทั้งหมดที่ Git ตรวจสอบ (หรืออนุมานได้จาก topology) ยังคง **good**
- Commit นี้และทุก commit หลังจากนี้ (จนถึง bad ที่ระบุไว้ตอนแรก) เป็น **bad**

พูดง่าย ๆ คือ **นี่คือจุดที่บั๊กถูกนำเข้ามาในโค้ดจริง ๆ**

### การดู diff แบบเต็มของ commit ต้นเหตุ

เมื่อรู้ hash ของ first bad commit แล้ว ขั้นตอนถัดไปคือดู diff แบบละเอียดเพื่อเข้าใจว่าอะไรทำให้เกิดบั๊ก:

```bash
git show 2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c
```

หรือถ้าอยากดูเฉพาะไฟล์ที่เปลี่ยน:

```bash
git show --stat 2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c
```

ตัวอย่าง diff ที่อาจเจอ:

```diff
diff --git a/src/discount/coupon.js b/src/discount/coupon.js
index a1b2c3d..e4f5a6b 100644
--- a/src/discount/coupon.js
+++ b/src/discount/coupon.js
@@ -12,7 +12,7 @@ function applyPercentageCoupon(price, percentage) {
-    return price - (price * percentage / 100);
+    return price - (price * percentage) / 1000;
 }
```

จาก diff นี้จะเห็นชัดว่า **ต้นเหตุของบั๊กคือ typo** — เปลี่ยนตัวหารจาก `100` เป็น `1000` โดยไม่ตั้งใจ ทำให้ discount คำนวณผิดพลาดไปสิบเท่า นี่คือตัวอย่างคลาสสิกของสิ่งที่ `git bisect` เก่งมาก — หา root cause ของบั๊กแบบที่การอ่านโค้ดผ่าน ๆ อาจมองข้ามไปได้ง่าย ๆ

### การดูว่าใครเป็นคนเขียน commit นี้ (ใช้ประกอบการสื่อสารในทีม)

```bash
git show -s --format="%an <%ae>%n%ad%n%s" 2b3c4d5
```

ผลลัพธ์:

```
Somchai Devteam <somchai@example.com>
Tue Mar 4 14:22:10 2025 +0700
Update coupon validation logic
```

ข้อมูลนี้มีประโยชน์มากเวลาต้องรายงานกลับให้ทีม หรือเปิด PR แก้บั๊กพร้อมอ้างอิง commit ต้นเหตุ

### การใช้ผลลัพธ์เพื่อแก้บั๊กต่อ

เมื่อรู้ commit ต้นเหตุแล้ว ทางเลือกในการแก้บั๊กมีหลายแบบ:

1. **แก้ไขตรง ๆ ใน branch ปัจจุบัน** — เขียนโค้ดแก้ปัญหาที่พบ (เช่นแก้ typo กลับเป็น `100`) แล้ว commit ใหม่
2. **ใช้ `git revert` กับ first bad commit** — ถ้า commit นั้นสามารถ revert ได้อย่างปลอดภัยโดยไม่กระทบ commit อื่นที่ตามมา (มักใช้เมื่อบั๊กเพิ่งถูกนำเข้ามาไม่นานและยังไม่มีอะไรมาพึ่งพาโค้ดใหม่นั้น)
3. **นำไปประกอบการทำ post-mortem** — วิเคราะห์ว่าทำไม test/review ถึงปล่อยให้บั๊กแบบนี้หลุดเข้ามาได้ เพื่อป้องกันไม่ให้เกิดซ้ำ (เช่น อาจต้องเพิ่ม unit test เฉพาะจุดนี้)

### สิ่งที่ `git bisect` ไม่ได้บอกให้

ควรเข้าใจขอบเขตของเครื่องมือนี้ให้ชัดเจน:

- `git bisect` บอกแค่ **"commit ไหน"** เป็นต้นเหตุ ไม่ได้บอกว่า **"บรรทัดไหน" หรือ "ทำไม"** ต้องไปดู diff เองต่อ (แต่โดยมากถ้า commit เล็กพอ ก็มักจะเห็นสาเหตุได้ทันที)
- ถ้า commit ต้นเหตุมีขนาดใหญ่มาก (แก้หลายไฟล์ หลายร้อยบรรทัด) อาจต้องใช้เครื่องมืออื่นเสริม เช่น `git blame` (ซึ่งเราจะเรียนใน **Part 43**) เพื่อไล่ดูว่าบรรทัดไหนในนั้นที่เกี่ยวข้องกับบั๊กจริง ๆ
- ถ้าทีมมีวินัยในการทำ commit เล็ก ๆ และมี commit message ที่ดี (ตามหลักการที่เรียนใน Part ก่อน ๆ) ผลลัพธ์จาก `git bisect` จะยิ่งมีประโยชน์และชี้เป้าได้แม่นยำมากขึ้นเท่านั้น — นี่คือเหตุผลสำคัญอีกข้อที่ทีมควรฝึกทำ **atomic commits**

ใน Step ถัดไป เราจะเรียนวิธีกลับสู่สถานะปกติของ repository หลังจากทำ bisect เสร็จแล้ว

---

## Step 417: `git bisect reset` — กลับสู่สถานะปกติหลังจบ Bisect

### ทำไมต้อง reset

หลังจากที่ `git bisect` หา first bad commit เจอแล้ว **repository จะยังอยู่ใน bisect mode** และ HEAD จะยังอยู่ในสถานะ **detached HEAD** ที่ชี้ไปยัง commit สุดท้ายที่ถูกทดสอบ (ไม่ใช่ branch ที่คุณทำงานอยู่เดิม)

ถ้าไม่ reset แล้วพยายามทำงานต่อ (เช่น พยายาม commit หรือ push) จะเกิดปัญหาตามมาหลายอย่าง เช่น commit ที่สร้างขึ้นจะลอยอยู่ใน detached HEAD และเสี่ยงหายไปเมื่อสลับ branch

### คำสั่งที่ใช้

```bash
git bisect reset
```

คำสั่งนี้จะทำสิ่งต่อไปนี้ให้อัตโนมัติ:

1. **Checkout กลับไปยัง branch/commit เดิม** ที่คุณอยู่ก่อนรัน `git bisect start` ครั้งแรก (ข้อมูลนี้ถูกเก็บไว้ใน `.git/BISECT_START`)
2. **ลบ reference ชั่วคราวทั้งหมด** ที่ bisect สร้างขึ้น (`refs/bisect/*`)
3. **ลบไฟล์สถานะภายใน** เช่น `BISECT_LOG`, `BISECT_NAMES`, `BISECT_TERMS`, `BISECT_START`
4. **ออกจาก bisect mode** — repository กลับสู่สถานะปกติทุกประการ

### ตัวอย่างการใช้งาน

```bash
$ git bisect reset
Previous HEAD position was 2b3c4d5 Update coupon validation logic
Switched to branch 'main'
```

จะเห็นว่า Git แจ้งให้ทราบว่ากลับไปที่ branch `main` (หรือ branch ที่คุณอยู่ก่อนเริ่ม bisect) เรียบร้อยแล้ว

### การ reset ไปยัง commit อื่นที่ไม่ใช่จุดเริ่มต้นเดิม

บางครั้งอาจต้องการ checkout ไปยัง commit เฉพาะเจาะจงหลังจบ bisect แทนที่จะกลับไป branch เดิม สามารถระบุ commit ต่อท้ายได้:

```bash
git bisect reset <commit-or-branch>
```

เช่น ถ้าต้องการไปดู first bad commit ต่อทันทีหลังจบ bisect:

```bash
git bisect reset 2b3c4d5
```

### ลืม reset แล้วเป็นอย่างไร

ถ้าลืม `git bisect reset` แล้วพยายามทำงานต่อ Git อาจจะยัง**ทำงานได้ตามปกติในหลายกรณี** เพราะ detached HEAD ก็ยังเป็น commit ที่ใช้งานได้จริง แต่จะมีความเสี่ยงว่า:

- ถ้าสลับไป branch อื่นโดยไม่ได้ commit งานที่ทำค้างไว้ งานนั้นอาจหายไป (แม้ Git จะพยายามเตือนก็ตาม)
- คำสั่ง Git บางตัว (โดยเฉพาะที่เกี่ยวกับ branch) อาจแสดงพฤติกรรมแปลก ๆ เพราะยังอยู่ใน "bisect state" ภายใน

**วิธีเช็คว่ายังอยู่ใน bisect mode หรือไม่:**

```bash
git bisect log
```

ถ้ายังอยู่ใน bisect mode จะเห็น log ของคำสั่งที่รันไปแล้ว แต่ถ้า reset ไปแล้วจะเจอ error ประมาณ:

```
We are not bisecting.
```

หรือใช้คำสั่งตรวจสอบสถานะไฟล์:

```bash
ls .git/ | grep -i bisect
```

ถ้าไม่พบไฟล์ที่ขึ้นต้นด้วย `BISECT` แปลว่าออกจาก bisect mode เรียบร้อยแล้ว

### แนวปฏิบัติที่ดี

> **กฎทองของ `git bisect`:** ทุกครั้งที่เริ่ม `git bisect start` ให้จำไว้เสมอว่าต้องจบด้วย `git bisect reset` ทุกครั้ง ไม่ว่าผลลัพธ์จะเจอ first bad commit หรือไม่ก็ตาม (แม้แต่กรณีที่ยกเลิกกลางคันเพราะพบว่าให้ good/bad ผิดไป ก็ต้อง reset ก่อนเริ่มใหม่)

ถ้าให้ good/bad ผิดพลาดไปกลางทางและต้องการเริ่มใหม่ทั้งหมด วิธีที่ปลอดภัยที่สุดคือ:

```bash
git bisect reset   # ออกจาก session เดิมให้เรียบร้อยก่อน
git bisect start   # แล้วค่อยเริ่มใหม่
```

ใน Step ถัดไป เราจะเรียนรู้วิธีจัดการกับ commit ที่ **ทดสอบไม่ได้** ระหว่างทำ bisect ด้วยคำสั่ง `git bisect skip`

---

## Step 418: `git bisect skip` — ข้าม Commit ที่ทดสอบไม่ได้

### ปัญหาที่พบบ่อยระหว่างทำ Bisect

ระหว่างไล่ทดสอบ commit กลาง ๆ ในประวัติ บางครั้งจะเจอสถานการณ์ที่ **ไม่สามารถระบุ good/bad ได้เลย** เพราะ commit นั้นมีปัญหาอื่นที่ไม่เกี่ยวข้องกับบั๊กที่กำลังตามหา เช่น:

- Commit นั้น **build ไม่ผ่านเลย** เนื่องจากอยู่ระหว่างการ refactor ที่ยังไม่เสร็จสมบูรณ์ (broken commit ที่ทีมลืมทดสอบก่อน push)
- Commit นั้นอยู่ในช่วงที่ **dependency บางตัวยังไม่ได้ติดตั้งหรือเข้ากันไม่ได้** กับสภาพแวดล้อมปัจจุบัน
- Commit นั้นเป็น **merge commit ที่สร้างความสับสน** จน environment ไม่สามารถรันโค้ดได้ตามปกติ
- ฟีเจอร์ที่เรากำลังทดสอบ **ยังไม่ถูกสร้างขึ้นในจุดนั้นของประวัติเลย** จึงไม่มีทางบอกได้ว่า "good" หรือ "bad" เพราะมันยังไม่มีอยู่จริง

ในสถานการณ์เหล่านี้ การตอบ `good` หรือ `bad` แบบมั่ว ๆ จะทำให้ผลลัพธ์ของ bisect ผิดเพี้ยนไปทั้งหมด เพราะมันทำลายสมมติฐาน monotonic ที่กล่าวถึงใน Step 412

### คำสั่งที่ใช้แก้ปัญหานี้

```bash
git bisect skip
```

เมื่อสั่ง `skip` Git จะ**ไม่ตัดสินใจ**ว่า commit นี้ good หรือ bad แต่จะ**เลือก commitข้างเคียงตัวอื่น**ที่อยู่ใกล้ ๆ กันมาให้ทดสอบแทน โดยพยายามหลีกเลี่ยง commit ที่เพิ่งถูก skip ไป

### ตัวอย่างการใช้งาน

```bash
$ git bisect start
$ git bisect bad HEAD
$ git bisect good v3.1.0
Bisecting: 27 revisions left to test after this (roughly 5 steps)
[9c8b7a6 ...] Refactor discount calculation module

$ npm install && npm test
# Error: build fails, unrelated dependency issue

$ git bisect skip
Bisecting: 26 revisions left to test after this (roughly 5 steps)
[4e5f6a7 ...] Add support for percentage-based coupons
```

Git จะเลือก commit ถัดไปให้ทดสอบแทนโดยอัตโนมัติ กระบวนการดำเนินต่อไปได้ตามปกติ

### การ skip แบบระบุช่วง (range)

ถ้ารู้ล่วงหน้าว่ามีช่วง commit หนึ่งที่ build พังทั้งหมด (เช่น ช่วงที่ทีมกำลัง migrate build system) สามารถ skip เป็นช่วงได้เลยโดยใช้ syntax แบบ range:

```bash
git bisect skip <commit-A>..<commit-B>
```

ตัวอย่าง:

```bash
git bisect skip 4e5f6a7..8f7e6d5
```

วิธีนี้ช่วยประหยัดเวลาได้มากถ้ารู้อยู่แล้วว่าช่วงนั้นทั้งหมดทดสอบไม่ได้แน่นอน

### ข้อจำกัดสำคัญของ `git bisect skip`

**ถ้า commit ที่ skip ไปนั้นอยู่ติดกับจุดเปลี่ยนผ่านจริง (first bad commit) พอดี** Git อาจ**ไม่สามารถระบุ first bad commit ที่แน่ชัดได้** เพราะไม่มีข้อมูลเพียงพอที่จะยืนยันว่าจุดเปลี่ยนผ่านอยู่ตรงไหนแน่ ในกรณีนี้ Git จะแจ้งผลลัพธ์เป็น**ช่วงที่เป็นไปได้** แทนที่จะเป็น commit เดียว เช่น:

```
There are only 'skip'ped commits left to test.
The first bad commit could be any of:
4e5f6a7e8d9c0b1a2f3e4d5c6b7a8f9e0d1c2b3a
2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c
We cannot bisect more!
```

เมื่อเจอสถานการณ์นี้ วิธีแก้คือ:

1. หาทางทำให้ commit ที่ skip ไปสามารถทดสอบได้ (เช่น แก้ dependency ให้ตรงกัน หรือ cherry-pick fix เล็ก ๆ เข้าไปชั่วคราวเพื่อให้ build ผ่าน)
2. ใช้การตรวจสอบด้วยมือ (`git show`, `git diff`) กับ commit ที่เหลือไม่กี่ตัวในรายการที่ Git แจ้งมา เพื่อสรุปเองว่าตัวไหนน่าจะเป็นต้นเหตุจริง

### เปรียบเทียบ `good` / `bad` / `skip`

| คำสั่ง | ความหมาย | ผลต่อกระบวนการ |
|---|---|---|
| `git bisect good` | Commit นี้**ยืนยันว่าไม่มี**บั๊ก | ตัดครึ่งที่เป็น "ก่อนหน้า" ทิ้ง |
| `git bisect bad` | Commit นี้**ยืนยันว่ามี**บั๊ก | ตัดครึ่งที่เป็น "หลังจากนี้" ทิ้ง |
| `git bisect skip` | **ไม่สามารถตัดสินได้** ว่า good หรือ bad | ข้ามไปเลือก commit ข้างเคียงแทน ไม่ตัดข้อมูลใด ๆ ทิ้ง |

### เมื่อไหร่ควรใช้ skip เทียบกับการแก้ไขปัญหาที่ต้นตอ

`git bisect skip` เป็นเครื่องมือที่มีประโยชน์มากแต่ **ควรใช้เท่าที่จำเป็นเท่านั้น** เพราะการ skip มากเกินไปจะลดความแม่นยำของผลลัพธ์ลง หากเป็นไปได้ วิธีที่ดีกว่าคือ:

- ปรับปรุง test script ให้ทนทานต่อปัญหาที่ไม่เกี่ยวข้อง (เช่น ถ้า dependency บางตัวขาดหาย ให้ install มันแยกต่างหากก่อนทดสอบจริง)
- แยกแยะให้ชัดว่า "ทดสอบไม่ได้เพราะสภาพแวดล้อม" กับ "ทดสอบไม่ได้เพราะเป็นบั๊กที่ตามหาอยู่" เป็นคนละเรื่องกัน — เฉพาะกรณีแรกเท่านั้นที่ควร skip

ใน Step ถัดไป เราจะเจาะลึกเทคนิคการเขียน test script ที่ดีสำหรับใช้กับ `git bisect run` ซึ่งเป็นหัวใจสำคัญของการทำ bisect แบบอัตโนมัติให้แม่นยำ

---

## Step 419: เทคนิคการเขียน Test Script ที่ดีสำหรับ `bisect run`

### หลักการพื้นฐาน: Exit Code คือภาษาที่ Git เข้าใจ

หัวใจสำคัญที่สุดของการใช้ `git bisect run` คือการเข้าใจว่า **Git ตัดสินผลลัพธ์จาก exit code ของสคริปต์เท่านั้น** ไม่ได้อ่านข้อความ output หรือตีความเนื้อหาใด ๆ ทั้งสิ้น ตารางค่าที่ Git กำหนดไว้แน่นอนมีดังนี้:

| Exit Code | ความหมายที่ Git ตีความ | หมายเหตุ |
|---|---|---|
| **0** | **Good** — commit นี้ไม่มีบั๊ก | เงื่อนไขทดสอบผ่านสมบูรณ์ |
| **1–124** | **Bad** — commit นี้มีบั๊ก | ช่วงค่าปกติที่ใช้บอกว่าล้มเหลว |
| **125** | **Skip** — ไม่สามารถทดสอบ commit นี้ได้ | สงวนไว้พิเศษสำหรับ skip เท่านั้น |
| **126** | **Bad** แต่มีความหมายพิเศษ | หมายถึงสคริปต์ถูกเจอแต่**รันไม่ได้** (permission denied) |
| **127** | **Bad** แต่มีความหมายพิเศษ | หมายถึง**หาสคริปต์ไม่เจอ** (command not found) |
| **128 ขึ้นไป** (เช่นจาก signal) | **ยกเลิกกระบวนการ bisect ทันที** | เช่นโดน `kill -9` (128+9=137) Git จะหยุดทำงานทั้งหมดเพราะถือว่าเป็นความผิดพลาดร้ายแรงที่ไม่ควรดำเนินการต่อ |

**ประเด็นที่ต้องระวังเป็นพิเศษ:** โค้ด **125 สงวนไว้เฉพาะความหมาย "skip"** เท่านั้น ห้ามใช้ผิดความหมายเด็ดขาด และค่า 126, 127 แม้จะถูกตีความเป็น "bad" แต่จริง ๆ แล้วมันสื่อว่า **สคริปต์เองมีปัญหา** ไม่ใช่โค้ดที่กำลังทดสอบมีปัญหา ดังนั้นควรเขียนสคริปต์ให้ระวังไม่ให้ exit code หลุดไปโดยไม่ตั้งใจในช่วง 125-127 นี้

### เทคนิคที่ 1: ใช้ `set -e` อย่างระมัดระวังใน Bash

การใช้ `set -e` ในสคริปต์ bash ช่วยให้สคริปต์หยุดทันทีเมื่อคำสั่งใดคำสั่งหนึ่งล้มเหลว แต่ต้องระวังผลข้างเคียง:

```bash
#!/bin/bash
set -e

npm install --silent   # ถ้าคำสั่งนี้ fail จะทำให้สคริปต์จบด้วย exit code ที่ไม่ใช่ 0 ทันที
npm run build --silent
node ./test/check-result.js
```

ปัญหาคือ ถ้า `npm install` ล้มเหลวเพราะ **network error ชั่วคราว** (ไม่เกี่ยวกับบั๊กที่ตามหา) สคริปต์จะจบด้วย exit code ที่ไม่ใช่ 0 และ 125 ทำให้ Git เข้าใจผิดว่า commit นั้น "bad" ทั้งที่จริง ๆ ควรเป็น "skip"

**วิธีแก้ที่ดีกว่า:** แยกส่วน setup ที่ไม่เกี่ยวกับบั๊กออกมาตรวจสอบเองแล้วคืนค่า skip อย่างชัดเจน

```bash
#!/bin/bash

# พยายาม build ก่อน ถ้า build ไม่ผ่านเลย ถือว่า "ทดสอบไม่ได้" ไม่ใช่ "มีบั๊ก"
if ! npm install --silent > /tmp/bisect-install.log 2>&1; then
    echo "SKIP: npm install failed, likely unrelated environment issue"
    exit 125
fi

if ! npm run build --silent > /tmp/bisect-build.log 2>&1; then
    echo "SKIP: build failed, likely unrelated to the bug we are chasing"
    exit 125
fi

# ถึงตรงนี้แปลว่า build ผ่านแล้ว ค่อยทดสอบบั๊กจริง ๆ
node ./test/check-result.js
exit $?   # ส่งต่อ exit code จริงจากการทดสอบ (0 = good, ไม่ใช่ 0 = bad)
```

### เทคนิคที่ 2: แยก "การเตรียมสภาพแวดล้อม" ออกจาก "การตรวจสอบบั๊ก" อย่างชัดเจน

โครงสร้างสคริปต์ที่ดีควรแบ่งเป็น 3 ส่วนเสมอ:

```
┌─────────────────────────────────┐
│ ส่วนที่ 1: เตรียมสภาพแวดล้อม        │  → ถ้าล้มเหลว = exit 125 (skip)
├─────────────────────────────────┤
│ ส่วนที่ 2: รันการทดสอบจริง         │  → นี่คือส่วนที่เช็คบั๊กที่ตามหา
├─────────────────────────────────┤
│ ส่วนที่ 3: ตัดสินผล good/bad       │  → exit 0 (good) หรือ exit 1 (bad)
└─────────────────────────────────┘
```

ตัวอย่างสคริปต์ที่ครบทั้ง 3 ส่วน:

```bash
#!/bin/bash
# bisect-test.sh — สคริปต์ตรวจสอบบั๊กการคำนวณส่วนลด

set -uo pipefail   # เข้มงวดเรื่อง unset variable และ pipe failure แต่ไม่ใช้ -e เพื่อคุมทุก error เอง

# ===== ส่วนที่ 1: เตรียมสภาพแวดล้อม =====
if ! npm ci --silent > /tmp/bisect.log 2>&1; then
    echo "SKIP: dependency install failed"
    exit 125
fi

if ! npm run build --silent >> /tmp/bisect.log 2>&1; then
    echo "SKIP: build failed (unrelated to the bug)"
    exit 125
fi

# ===== ส่วนที่ 2: รันการทดสอบจริง =====
ACTUAL=$(node ./scripts/calculate-discount.js --price=100 --coupon=SAVE20 2>/tmp/bisect-run.log)
EXPECTED="80"

# ===== ส่วนที่ 3: ตัดสินผล =====
if [ "$ACTUAL" == "$EXPECTED" ]; then
    echo "GOOD: got $ACTUAL as expected"
    exit 0
else
    echo "BAD: got $ACTUAL, expected $EXPECTED"
    exit 1
fi
```

### เทคนิคที่ 3: ระวัง Test ที่ไม่เกี่ยวข้องกับ Commit เก่า

เมื่อ bisect ไล่ย้อนไปยัง commit เก่ามาก ๆ อาจเจอว่า**ฟีเจอร์ที่กำลังทดสอบยังไม่ถูกสร้างขึ้น**เลยในจุดนั้นของประวัติ ให้เขียนสคริปต์เช็คสิ่งนี้ล่วงหน้า:

```bash
if [ ! -f "./scripts/calculate-discount.js" ]; then
    echo "SKIP: feature not implemented yet at this point in history"
    exit 125
fi
```

### เทคนิคที่ 4: ทำความสะอาด (Clean) ก่อนทดสอบทุกครั้ง

ผลลัพธ์การทดสอบที่ผิดเพี้ยนบ่อยครั้งเกิดจาก **artifact ที่หลงเหลือจาก commit ก่อนหน้า** เช่น build cache, `node_modules` ที่ไม่ตรงเวอร์ชัน หรือไฟล์ compiled เก่า ควรเพิ่มการล้างข้อมูลก่อนเริ่มทดสอบทุกรอบ:

```bash
rm -rf dist/ build/ .cache/
npm ci --silent   # ใช้ npm ci แทน npm install เพื่อความแน่นอนของ dependency tree
```

`npm ci` (หรือเทียบเท่าในภาษาอื่น เช่น `pip install -r requirements.txt --force-reinstall`) ดีกว่า `npm install` ในบริบทนี้ เพราะรับประกันว่า dependency tree จะตรงกับ lockfile เป๊ะ ๆ ทุกครั้ง ไม่พึ่งพา cache ที่อาจตกค้าง

### เทคนิคที่ 5: จัดการกับ Flaky Test (บั๊กที่เกิดไม่สม่ำเสมอ)

ถ้าการทดสอบมีความไม่แน่นอน (เช่น เกี่ยวข้องกับ timing, race condition, หรือ network) ควร**รันซ้ำหลายครั้งแล้วดูผลส่วนใหญ่ (majority vote)** แทนที่จะเชื่อผลจากการรันครั้งเดียว:

```bash
#!/bin/bash
PASS_COUNT=0
TOTAL_RUNS=5

for i in $(seq 1 $TOTAL_RUNS); do
    if node ./test/flaky-check.js; then
        PASS_COUNT=$((PASS_COUNT + 1))
    fi
done

# ถ้าผ่านอย่างน้อย 4 ใน 5 ครั้ง ถือว่า good
if [ "$PASS_COUNT" -ge 4 ]; then
    exit 0
else
    exit 1
fi
```

อย่างไรก็ตาม วิธีนี้เป็นเพียง**การบรรเทาปัญหา** ไม่ใช่ทางแก้ที่สมบูรณ์แบบ — ถ้าเป็นไปได้ ควรแก้ flaky test ให้เสถียรก่อนใช้ bisect เพราะสมมติฐาน monotonic ที่ bisect ใช้จะไม่แม่นยำ 100% เมื่อมีความไม่แน่นอนแบบนี้ปะปนอยู่

### สรุปเช็คลิสต์สำหรับเขียน Bisect Test Script ที่ดี

- [ ] แยกส่วน "เตรียมสภาพแวดล้อม" ออกจาก "ทดสอบบั๊ก" อย่างชัดเจน
- [ ] ใช้ exit code 125 เฉพาะกรณี "ทดสอบไม่ได้เพราะเหตุผลอื่น" เท่านั้น
- [ ] หลีกเลี่ยง exit code 126, 127 และ 128 ขึ้นไปโดยไม่ตั้งใจ
- [ ] ล้าง build artifact และ cache ก่อนทดสอบทุกรอบ
- [ ] เช็คว่าไฟล์/ฟีเจอร์ที่จะทดสอบมีอยู่จริงในแต่ละ commit ก่อนเรียกใช้
- [ ] ถ้า test มีความไม่แน่นอน ให้รันซ้ำหลายครั้งเพื่อลดโอกาสผลลัพธ์ผิดพลาด
- [ ] ทดสอบสคริปต์กับ commit ที่รู้อยู่แล้วว่า good และ bad ก่อนปล่อยให้ `bisect run` ทำงานเต็มรูปแบบ (sanity check)

ใน Step สุดท้ายของ Part นี้ เราจะมาลงมือปฏิบัติจริงทั้งหมดที่เรียนมา ผ่านแบบฝึกหัดจำลองสถานการณ์บั๊กแอบแฝง

---

## Step 420: แบบฝึกหัด — จำลองประวัติ Commit ที่มีบั๊กแอบแฝง แล้วใช้ Bisect หาให้เจอ

### เป้าหมายของแบบฝึกหัด

เราจะสร้าง repository ทดลองที่มี **20 commit** โดยมี **1 commit ที่แอบใส่บั๊กเข้ามา** จากนั้นจะฝึกใช้ `git bisect` หาบั๊กนั้นให้เจอ **2 รูปแบบ**: แบบ manual ทีละรอบ และแบบอัตโนมัติด้วย `git bisect run`

### ขั้นตอนที่ 1: เตรียม Repository ทดลอง

```bash
mkdir ~/git-course/part-42-bisect-practice
cd ~/git-course/part-42-bisect-practice
git init -q
git config user.name "Practice User"
git config user.email "practice@example.com"
```

### ขั้นตอนที่ 2: สร้างไฟล์โปรแกรมตั้งต้น

เราจะใช้โปรแกรมคำนวณราคาสินค้าหลังหักส่วนลดง่าย ๆ เป็นตัวอย่าง สร้างไฟล์ `discount.js`:

```bash
cat > discount.js << 'EOF'
function applyDiscount(price, percentage) {
    return price - (price * percentage / 100);
}

module.exports = { applyDiscount };
EOF

git add discount.js
git commit -q -m "Initial commit: add discount calculation"
```

### ขั้นตอนที่ 3: สร้าง 20 Commit โดยแอบใส่บั๊กที่ Commit ที่ 13

เราจะเขียนสคริปต์เพื่อสร้าง commit จำนวนมากอัตโนมัติ โดย commit ปกติจะแค่เพิ่มคอมเมนต์หรือแก้ README เล็กน้อย (ไม่กระทบ logic) ยกเว้น **commit ที่ 13** ที่จะแอบเปลี่ยน logic ให้ผิดเป็นบั๊กจริง:

```bash
for i in $(seq 2 20); do
    if [ "$i" -eq 13 ]; then
        # commit ที่ 13: แอบใส่บั๊ก — เปลี่ยนตัวหารจาก 100 เป็น 1000 (typo จริง)
        cat > discount.js << 'EOF'
function applyDiscount(price, percentage) {
    return price - (price * percentage / 1000);
}

module.exports = { applyDiscount };
EOF
        git add discount.js
        git commit -q -m "Refactor: simplify percentage rounding logic"
    else
        # commit อื่น ๆ: แก้ไขที่ไม่เกี่ยวข้องกับ logic เลย (เพื่อจำลองประวัติที่สมจริง)
        echo "// commit number $i - unrelated change" >> CHANGELOG.md
        git add CHANGELOG.md
        git commit -q -m "Update changelog (commit $i)"
    fi
done
```

หลังรันสคริปต์นี้ เราจะมีทั้งหมด 20 commit (รวม commit แรก) ตรวจสอบด้วย:

```bash
git log --oneline
```

ผลลัพธ์ควรมีประมาณนี้ (hash จริงจะต่างกันไปในเครื่องของแต่ละคน):

```
f9a8b7c (HEAD -> main) Update changelog (commit 20)
e8d7c6b Update changelog (commit 19)
...
a3b2c1d Refactor: simplify percentage rounding logic   <-- นี่คือ commit ที่ 13 ที่มีบั๊ก
...
d4c3b2a Update changelog (commit 3)
c3b2a1f Update changelog (commit 2)
e5f6a7b Initial commit: add discount calculation
```

**สำคัญ:** จดจำ commit hash ของ **commit แรกสุด** ("Initial commit") ไว้ เพราะจะใช้เป็นจุด "good" (เรารู้แน่นอนว่า ณ จุดนี้ยังไม่มีบั๊ก)

```bash
FIRST_COMMIT=$(git log --oneline | tail -1 | awk '{print $1}')
echo "First commit (known good): $FIRST_COMMIT"
```

### ขั้นตอนที่ 4: ยืนยันว่ามีบั๊กจริงใน HEAD ปัจจุบัน

```bash
node -e "console.log(require('./discount.js').applyDiscount(100, 20))"
```

ค่าที่ควรได้คือ **80** (ราคา 100 หักส่วนลด 20% ควรเหลือ 80) แต่เพราะบั๊ก จะได้ผลลัพธ์ผิดเป็น **98** แทน (เพราะหารด้วย 1000 แทนที่จะเป็น 100)

### ส่วนที่ 1: ฝึกทำ Bisect แบบ Manual

```bash
git bisect start
git bisect bad HEAD
git bisect good $FIRST_COMMIT
```

Git จะแสดงข้อความและ checkout ไปยัง commit กึ่งกลางให้ทันที เช่น:

```
Bisecting: 9 revisions left to test after this (roughly 4 steps)
[xxxxxxx] Update changelog (commit 11)
```

ทดสอบด้วยมือทุกรอบ:

```bash
node -e "console.log(require('./discount.js').applyDiscount(100, 20))"
```

- ถ้าผลลัพธ์เป็น **80** (ถูกต้อง) → `git bisect good`
- ถ้าผลลัพธ์เป็น **98** (ผิดพลาด) → `git bisect bad`

ทำซ้ำแบบนี้ไปเรื่อย ๆ จนกว่า Git จะประกาศ:

```
a3b2c1d... is the first bad commit
commit a3b2c1d...
    Refactor: simplify percentage rounding logic

 discount.js | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

ยืนยันว่าตรงกับ commit ที่ 13 ที่เราแอบใส่บั๊กไว้จริง ๆ! จากนั้น reset กลับสู่สถานะปกติ:

```bash
git bisect reset
```

### ส่วนที่ 2: ฝึกทำ Bisect แบบอัตโนมัติด้วย `bisect run`

ก่อนอื่นสร้างสคริปต์ทดสอบ `check-discount.sh`:

```bash
cat > check-discount.sh << 'EOF'
#!/bin/bash

# ส่วนที่ 1: เช็คว่าไฟล์ที่ต้องการทดสอบมีอยู่จริง
if [ ! -f "./discount.js" ]; then
    echo "SKIP: discount.js not found at this commit"
    exit 125
fi

# ส่วนที่ 2: รันการทดสอบจริง
RESULT=$(node -e "console.log(require('./discount.js').applyDiscount(100, 20))" 2>/dev/null)

# ส่วนที่ 3: ตัดสินผล
if [ "$RESULT" == "80" ]; then
    echo "GOOD: got $RESULT as expected"
    exit 0
else
    echo "BAD: got $RESULT, expected 80"
    exit 1
fi
EOF

chmod +x check-discount.sh
```

ทดสอบสคริปต์ก่อนว่าทำงานถูกต้องกับ HEAD ปัจจุบัน (ควรได้ผล BAD) และกับ commit แรก (ควรได้ผล GOOD) เพื่อ sanity check:

```bash
./check-discount.sh                 # ควรเห็น "BAD: got 98, expected 80"
git checkout -q $FIRST_COMMIT
./check-discount.sh                 # ควรเห็น "GOOD: got 80 as expected"
git checkout -q main
```

เมื่อมั่นใจว่าสคริปต์ทำงานถูกต้องแล้ว ให้เริ่ม bisect แบบอัตโนมัติ:

```bash
git bisect start
git bisect bad HEAD
git bisect good $FIRST_COMMIT
git bisect run ./check-discount.sh
```

Git จะรันสคริปต์นี้เองในทุกรอบโดยอัตโนมัติ พร้อมแสดง output ของแต่ละรอบ:

```
running './check-discount.sh'
BAD: got 98, expected 80
Bisecting: 4 revisions left to test after this (roughly 2 steps)
[yyyyyyy] Update changelog (commit 7)
running './check-discount.sh'
GOOD: got 80 as expected
Bisecting: 2 revisions left to test after this (roughly 1 step)
[zzzzzzz] Update changelog (commit 10)
running './check-discount.sh'
GOOD: got 80 as expected
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[a3b2c1d] Refactor: simplify percentage rounding logic
running './check-discount.sh'
BAD: got 98, expected 80
a3b2c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8 is the first bad commit
commit a3b2c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8
    Refactor: simplify percentage rounding logic

 discount.js | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
bisect run success
```

สังเกตบรรทัดสุดท้าย **`bisect run success`** ซึ่งเป็นข้อความยืนยันว่ากระบวนการอัตโนมัติทำงานจนจบสมบูรณ์และหาคำตอบได้ ผลลัพธ์ตรงกับที่ได้จากการทำ manual bisect ก่อนหน้าทุกประการ

จากนั้น reset กลับสู่สถานะปกติ:

```bash
git bisect reset
```

### ขั้นตอนเสริม: ลองฝึกใช้ `git bisect skip`

เพื่อฝึกความเข้าใจเรื่อง skip ให้ลองแก้สคริปต์ `check-discount.sh` เพื่อจำลองว่ามี commit บางตัวที่ "build พังด้วยเหตุผลอื่น" เช่น สมมติว่า commit ที่ 7 มีปัญหาบางอย่างที่ไม่เกี่ยวกับบั๊กที่ตามหา:

```bash
cat > check-discount.sh << 'EOF'
#!/bin/bash

CURRENT_HASH=$(git rev-parse --short HEAD)
BROKEN_COMMIT_SHORT_HASH="เอา-hash-ของ-commit-7-มาใส่ตรงนี้"

if [ "$CURRENT_HASH" == "$BROKEN_COMMIT_SHORT_HASH" ]; then
    echo "SKIP: simulated unrelated environment failure"
    exit 125
fi

if [ ! -f "./discount.js" ]; then
    echo "SKIP: discount.js not found at this commit"
    exit 125
fi

RESULT=$(node -e "console.log(require('./discount.js').applyDiscount(100, 20))" 2>/dev/null)

if [ "$RESULT" == "80" ]; then
    exit 0
else
    exit 1
fi
EOF
chmod +x check-discount.sh

git bisect start
git bisect bad HEAD
git bisect good $FIRST_COMMIT
git bisect run ./check-discount.sh
git bisect reset
```

สังเกตว่าแม้จะมี commit ที่ต้อง skip ระหว่างทาง Git ก็ยังสามารถหา first bad commit ได้ถูกต้องเหมือนเดิม (ตราบใดที่ commit ที่ skip ไม่ได้อยู่ติดกับจุดเปลี่ยนผ่านพอดี ตามที่อธิบายไว้ใน Step 418)

### แบบฝึกหัดเพิ่มเติมสำหรับฝึกฝนด้วยตัวเอง

ลองทำสิ่งต่อไปนี้ด้วยตัวเองเพื่อทบทวนความเข้าใจ:

1. สร้างประวัติ commit ใหม่ 30 commit โดยแอบใส่บั๊ก 1 จุดที่ตำแหน่งสุ่ม (ใช้ `$RANDOM` ใน bash เพื่อสุ่มตำแหน่ง) แล้วลองหาด้วย `git bisect run` โดยไม่รู้ล่วงหน้าว่าบั๊กอยู่ตรงไหน
2. ลองจำลองสถานการณ์ **flaky test** โดยให้สคริปต์คืนค่าผิดแบบสุ่ม 20% ของเวลา แล้วสังเกตว่า `git bisect run` ให้ผลลัพธ์ผิดพลาดได้อย่างไร จากนั้นแก้สคริปต์ให้รันซ้ำหลายครั้งตามเทคนิคที่เรียนใน Step 419 แล้วดูว่าผลลัพธ์แม่นยำขึ้นหรือไม่
3. ลองใช้ `git bisect log > my-bisect.log` หลังจากทำ bisect ด้วยมือไปครึ่งทาง แล้ว `git bisect reset` แล้วลอง `git bisect replay my-bisect.log` เพื่อดูว่า Git กลับไปที่จุดเดิมที่ทำค้างไว้ได้จริงหรือไม่

### เช็คลิสต์ทบทวนก่อนไป Part 43

ก่อนไปต่อ Part 43 ให้ตรวจสอบว่าคุณ:

- [ ] เข้าใจว่า `git bisect` แก้ปัญหาอะไร และทำไมถึงจำเป็นเมื่อประวัติ commit มีจำนวนมาก
- [ ] เข้าใจหลักการ binary search และเหตุผลที่มันเร็วกว่า linear search แบบ O(log N) เทียบกับ O(N)
- [ ] สามารถเริ่มต้น bisect session ด้วย `git bisect start`, `git bisect bad`, `git bisect good` ได้อย่างถูกต้อง
- [ ] เข้าใจขั้นตอนการทำ bisect ทีละรอบด้วยมือ และรู้วิธีอ่านข้อความ "Bisecting: N revisions left" ที่ Git แสดง
- [ ] สามารถเขียนและใช้ `git bisect run` เพื่ออัตโนมัติทั้งกระบวนการได้
- [ ] เข้าใจความหมายของ "first bad commit" และรู้วิธีดู diff เพื่อวิเคราะห์ต้นเหตุของบั๊กต่อ
- [ ] รู้ว่าต้อง `git bisect reset` ทุกครั้งหลังจบกระบวนการ ไม่ว่าจะสำเร็จหรือยกเลิกกลางคัน
- [ ] เข้าใจการใช้ `git bisect skip` และข้อจำกัดของมันเมื่อ commit ที่ skip อยู่ติดกับจุดเปลี่ยนผ่านจริง
- [ ] รู้เทคนิคเขียน test script ที่ดีสำหรับ `bisect run` โดยเฉพาะเรื่อง exit code (0=good, 1-124=bad, 125=skip)
- [ ] ลงมือฝึกจำลองประวัติ commit ที่มีบั๊กแอบแฝงและใช้ bisect หาให้เจอด้วยตัวเองทั้งแบบ manual และแบบ run สำเร็จแล้ว

---

## สรุป Part 42

ใน Part นี้เราได้เรียนรู้ว่า:

1. `git bisect` คือเครื่องมือที่ช่วยหา commit ที่เป็นต้นเหตุของบั๊กในประวัติที่มีจำนวนมาก โดยไม่ต้องไล่เช็คทีละ commit
2. หลักการเบื้องหลังคือ **binary search** ที่ตัดจำนวน commit ที่ต้องเช็คลงครึ่งหนึ่งทุกรอบ ทำให้ความเร็วโตแบบ O(log N) แทนที่จะเป็น O(N)
3. การเริ่มต้นใช้งานทำผ่าน `git bisect start`, `git bisect bad`, และ `git bisect good <commit>` โดย Git จะ checkout ไปยัง commit กึ่งกลางให้อัตโนมัติทุกรอบ
4. การทำ bisect แบบ manual คือวนซ้ำ ทดสอบ แล้วบอกผลด้วย `git bisect good` หรือ `git bisect bad` จนกว่าจะเจอ first bad commit
5. `git bisect run <script>` ช่วยให้กระบวนการทั้งหมดทำงานอัตโนมัติ โดยอาศัย exit code ของสคริปต์เป็นตัวตัดสิน (0 = good, 1–124 = bad, 125 = skip)
6. ผลลัพธ์สุดท้ายจะแสดง "first bad commit" พร้อมข้อมูล author, วันที่ และ diffstat ให้นำไปวิเคราะห์ต่อด้วย `git show`
7. `git bisect reset` เป็นขั้นตอนที่จำเป็นเสมอเพื่อออกจาก bisect mode และกลับสู่ branch เดิมอย่างปลอดภัย
8. `git bisect skip` ใช้ข้าม commit ที่ทดสอบไม่ได้ด้วยเหตุผลอื่นที่ไม่เกี่ยวกับบั๊ก แต่มีข้อจำกัดเมื่อ commit ที่ skip อยู่ติดกับจุดเปลี่ยนผ่านจริง
9. การเขียน test script ที่ดีสำหรับ `bisect run` ต้องแยกส่วน "เตรียมสภาพแวดล้อม" ออกจาก "ทดสอบบั๊กจริง" อย่างชัดเจน และจัดการ exit code ให้ถูกต้องตามความหมายที่ Git กำหนด
10. เราได้ฝึกปฏิบัติจริงด้วยการจำลองประวัติ 20 commit ที่มีบั๊กแอบแฝง และใช้ bisect หาให้เจอได้สำเร็จทั้งแบบ manual และแบบอัตโนมัติ

**ต่อไป:** [Part 43: Git Blame และการตรวจสอบประวัติโค้ดเชิงลึก](./part-043-git-blame.md)
