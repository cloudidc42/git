# Part 61: Git Attributes: .gitattributes และการจัดการไฟล์พิเศษ

> **Step ในหลักสูตรนี้:** Step 601–610
> **เฟส:** 6 — Git ขั้นสูงระดับ Internals, Hooks, LFS, Performance
> **เป้าหมายของ Part นี้:** เข้าใจว่าไฟล์ `.gitattributes` คืออะไร แก้ปัญหาอะไรที่ `.gitignore` และ `core.autocrlf` แก้ไม่ได้ เรียนรู้ attribute สำคัญทั้งหมด (`text`, `eol`, `diff`, `merge`, `export-ignore`, `export-subst`, `linguist-*`, `filter`) พร้อมตัวอย่างการใช้งานจริง และปิดท้ายด้วยการเขียน `.gitattributes` มาตรฐานสำหรับโปรเจกต์ cross-platform ที่ใช้งานได้จริงในโลกการทำงาน

---

## สารบัญของ Part นี้

- Step 601: `.gitattributes` คืออะไร แก้ปัญหาอะไร
- Step 602: `text`/`eol` attributes — ควบคุม line ending ต่อไฟล์/ต่อ pattern
- Step 603: `diff` attribute — กำหนดวิธี diff ไฟล์พิเศษ
- Step 604: `merge` attribute — custom merge strategy ต่อไฟล์
- Step 605: `export-ignore`, `export-subst` — ควบคุมพฤติกรรมตอน `git archive`
- Step 606: `linguist-*` attributes — ควบคุมการนับสัดส่วนภาษาบน GitHub
- Step 607: filter attributes (`filter=lfs`, clean/smudge) — เกริ่นก่อนเจาะลึกใน Git LFS
- Step 608: `.gitattributes` vs `.gitignore` ต่างกันอย่างไร
- Step 609: Best practices — ไฟล์ `.gitattributes` มาตรฐานสำหรับโปรเจกต์ cross-platform
- Step 610: แบบฝึกหัด — เขียน `.gitattributes` ควบคุม line ending, diff, linguist

---

## Step 601: `.gitattributes` คืออะไร แก้ปัญหาอะไร

จนถึงตอนนี้ในหลักสูตร เราคุ้นเคยกับไฟล์ `.gitignore` ที่บอก Git ว่า **"ไฟล์ไหนไม่ต้องสนใจเลย"** แต่มีอีกกลุ่มปัญหาหนึ่งที่ `.gitignore` ตอบไม่ได้ นั่นคือ:

> ไฟล์ที่ **ต้อง** อยู่ใน repository แต่ Git ควร **ปฏิบัติกับมันแตกต่างจากไฟล์ทั่วไป**

ตัวอย่างสถานการณ์จริงที่เกิดขึ้นบ่อยมากในทีมพัฒนา:

1. ทีมมีทั้งคนใช้ Windows และ macOS/Linux ทำงานร่วมกัน แล้วไฟล์ `.sh` (shell script) ถูก commit ด้วย line ending แบบ CRLF โดยไม่ได้ตั้งใจ ทำให้ script รันไม่ได้บนเครื่อง Linux เพราะ shebang line (`#!/bin/bash\r`) มี `\r` แฝงอยู่
2. ไฟล์ binary เช่น `.png`, `.zip`, `.docx` ถูก Git พยายาม diff แบบข้อความ (text) แล้วขึ้นข้อความไร้ประโยชน์เต็มหน้าจอ เช่น `Binary files a/logo.png and b/logo.png differ` ซึ่งไม่ช่วยอะไรเลย หรือแย่กว่านั้นคือ Git พยายาม merge ไฟล์ binary แล้วไฟล์เสียหาย
3. ไฟล์ lock ของ package manager เช่น `package-lock.json`, `yarn.lock` เกิด conflict บ่อยมากตอน merge branch ทั้งที่จริง ๆ ทีมอยากให้ใช้เวอร์ชันของ branch ปลายทางเสมอ ไม่อยากมานั่งแก้ conflict มือ
4. เวลาทำ `git archive` เพื่อ export โปรเจกต์เป็นไฟล์ zip/tar สำหรับส่งมอบลูกค้า ไฟล์ทดสอบ, ไฟล์ CI config, หรือโฟลเดอร์ `docs/` ภายในไม่ควรติดไปด้วย ทั้งที่ไฟล์เหล่านี้ต้องอยู่ใน repository ปกติ (จะสั่ง `.gitignore` ก็ไม่ได้ เพราะไฟล์เหล่านี้ต้อง track อยู่แล้ว)
5. หน้า repository บน GitHub แสดงผลว่าโปรเจกต์เป็น "98% JavaScript" ทั้งที่จริง ๆ โค้ดหลักเป็นภาษาอื่น เพราะมีโฟลเดอร์ `vendor/` หรือไฟล์ที่ generate อัตโนมัติ (เช่น `dist/bundle.js`) ปนอยู่ในการนับสัดส่วน

ปัญหาทั้งหมดนี้มีลักษณะร่วมกันคือ **"พฤติกรรมของ Git ต่อไฟล์บางประเภทควรแตกต่างจากพฤติกรรมมาตรฐาน โดยขึ้นอยู่กับ path หรือ pattern ของไฟล์นั้น"** — และนี่คือหน้าที่โดยตรงของไฟล์ **`.gitattributes`**

### นิยามของ `.gitattributes`

> **`.gitattributes`** คือไฟล์ configuration ของ Git ที่ใช้กำหนด **attribute (คุณสมบัติ)** ให้กับไฟล์หรือกลุ่มไฟล์ที่ match กับ pattern ที่ระบุไว้ เพื่อควบคุมว่า Git ควรปฏิบัติกับไฟล์เหล่านั้นอย่างไรในสถานการณ์ต่าง ๆ เช่น การ diff, การ merge, การแปลง line ending, การ export

รูปแบบไฟล์เป็น plain text ธรรมดา วางไว้ที่ root ของ repository (หรือใน subdirectory ก็ได้ — attribute จะมีผลกับไฟล์ในโฟลเดอร์นั้นและโฟลเดอร์ย่อย) โครงสร้างแต่ละบรรทัดคือ:

```
<pattern>  attr1 attr2 attr3=value ...
```

ตัวอย่างง่าย ๆ:

```gitattributes
*.txt           text
*.png           binary
*.sh            text eol=lf
```

### ตำแหน่งที่วางไฟล์ `.gitattributes` ได้ (เรียงตามลำดับความสำคัญจากต่ำไปสูง)

Git อ่าน attribute จากหลายแหล่งพร้อมกัน โดยไฟล์ที่อยู่ **ใกล้ไฟล์เป้าหมายที่สุด** จะมีความสำคัญสูงสุด (override ไฟล์ที่อยู่ไกลกว่า):

| ลำดับ (สูง→ต่ำ) | ตำแหน่งไฟล์ | ขอบเขตผลบังคับใช้ |
|---|---|---|
| 1 (สูงสุด) | `$GIT_DIR/info/attributes` | เฉพาะ local repository นี้เท่านั้น ไม่ถูก commit ไม่แชร์กับใคร |
| 2 | `.gitattributes` ในโฟลเดอร์ที่ไฟล์อยู่ หรือโฟลเดอร์แม่ที่ใกล้ที่สุด | เฉพาะไฟล์ในโฟลเดอร์นั้นและโฟลเดอร์ย่อย |
| 3 | `.gitattributes` ที่ root ของ repository | ทั้ง repository — **ตำแหน่งที่ใช้บ่อยที่สุดและแนะนำที่สุด** เพราะ commit ร่วมกับ repo ได้ ทุกคนในทีมได้ค่าเดียวกัน |
| 4 (ต่ำสุด) | `core.attributesFile` (global, กำหนดผ่าน `git config --global core.attributesFile`) | ทุก repository บนเครื่องนั้น (ค่า default ส่วนตัวของแต่ละคน) |

**ข้อควรจำสำคัญ:** ไฟล์ `.gitattributes` ที่ root repo **ควร commit เข้า repository เสมอ** เพื่อให้ทุกคนในทีม (และทุกเครื่องที่ clone ไป) ได้พฤติกรรมเดียวกัน ต่างจาก `$GIT_DIR/info/attributes` ที่เป็นการตั้งค่าเฉพาะเครื่องตัวเอง ไม่แชร์กับใคร

### รูปแบบ pattern ที่ใช้ได้

Pattern ใน `.gitattributes` ใช้ syntax แบบเดียวกับ `.gitignore` (glob pattern):

```gitattributes
*.md            text          # ทุกไฟล์ .md ทุกที่ในโปรเจกต์
/README.md      text          # เฉพาะ README.md ที่ root เท่านั้น (ขึ้นต้นด้วย /)
docs/*.txt      text          # ไฟล์ .txt ในโฟลเดอร์ docs/ ชั้นแรกเท่านั้น
docs/**/*.txt   text          # ไฟล์ .txt ในโฟลเดอร์ docs/ ทุกระดับชั้น
assets/         -text         # ทุกไฟล์ในโฟลเดอร์ assets/ (ไม่รวม subfolder โดยอัตโนมัติถ้าไม่ใส่ /**)
```

กฎการ match: **ถ้ามีหลาย pattern match ไฟล์เดียวกัน บรรทัดที่อยู่ล่างสุด (ถูกอ่านทีหลังสุด) จะชนะ** เหมือนกับ `.gitignore`

### รูปแบบการตั้งค่า attribute (4 แบบ)

```gitattributes
*.txt      text        # เปิด (set) attribute "text" — ใส่ชื่อ attribute เฉย ๆ
*.jpg      -text       # ปิด (unset) attribute "text" — ใส่ - นำหน้า
*.sh       eol=lf      # กำหนดค่าให้ attribute — ใส่ =value
*.dat      !diff       # ยกเลิกการตั้งค่า (unspecified) กลับสู่ค่า default — ใส่ ! นำหน้า
```

ทั้ง 4 รูปแบบนี้จะปรากฏซ้ำ ๆ ตลอด Part นี้ ขอให้จำไว้ให้แม่นเพราะเป็นไวยากรณ์หลักของทั้งไฟล์

### สรุปภาพรวมว่า `.gitattributes` ควบคุมอะไรได้บ้าง

| หมวด | Attribute | ใช้ทำอะไร | จะเรียนใน Step ไหน |
|---|---|---|---|
| Line ending | `text`, `eol` | ควบคุมการแปลง CRLF/LF | Step 602 |
| Diff | `diff` | กำหนดวิธี diff ไฟล์พิเศษ | Step 603 |
| Merge | `merge` | กำหนด merge strategy เฉพาะไฟล์ | Step 604 |
| Archive | `export-ignore`, `export-subst` | ควบคุมพฤติกรรม `git archive` | Step 605 |
| Language stats | `linguist-*` | ควบคุมการนับภาษาบน GitHub | Step 606 |
| Filter | `filter`, `clean`, `smudge` | แปลงข้อมูลไฟล์ตอน commit/checkout (พื้นฐานของ Git LFS) | Step 607 |

ต่อไปเราจะลงลึกทีละ attribute เริ่มจากเรื่องที่ผู้เรียนน่าจะคุ้นเคยที่สุดอยู่แล้ว — line ending

---

## Step 602: `text`/`eol` attributes — ควบคุม line ending ต่อไฟล์/ต่อ pattern

ใน **Part 02** เราได้เรียนเรื่อง `core.autocrlf` ไปแล้ว ซึ่งเป็นการตั้งค่าระดับ **global หรือ repository เดียวทั้งก้อน** — พูดง่าย ๆ คือ "ทั้ง repo แปลง line ending แบบเดียวกันหมด" ปัญหาคือในโลกจริง โปรเจกต์หนึ่ง ๆ มักมีไฟล์หลายประเภทปนกัน:

- ไฟล์ `.sh` (shell script) **ต้อง** เป็น LF เท่านั้นไม่ว่าจะ commit จาก OS ไหน เพราะ Linux/macOS รัน script ที่มี CRLF ไม่ได้
- ไฟล์ `.bat`, `.ps1` (Windows script) ควรเป็น CRLF เพื่อให้ Notepad และเครื่องมือ Windows บางตัวแสดงผลถูกต้อง
- ไฟล์ binary เช่น `.png`, `.exe` **ห้ามแตะ** line ending เด็ดขาด เพราะจะทำให้ไฟล์เสียหาย
- ไฟล์ source code ทั่วไปอยากให้ normalize เป็น LF เสมอในที่เก็บ (repository) แต่ตอน checkout ลงเครื่อง Windows ค่อยแปลงเป็น CRLF ให้อัตโนมัติ

นี่คือสิ่งที่ `core.autocrlf` ระดับเดียวทำไม่ได้ละเอียดพอ — ต้องใช้ **`text` และ `eol` attribute ต่อไฟล์/ต่อ pattern** ใน `.gitattributes` แทน

### Attribute `text`

`text` บอก Git ว่าไฟล์นี้เป็น **text file** และควรให้ Git จัดการเรื่อง line ending normalization ให้

| ค่า | ความหมาย |
|---|---|
| `text` (set) | บังคับว่าไฟล์นี้เป็น text เสมอ Git จะ normalize line ending เป็น LF ตอนเก็บใน repository (ในกระบวนการ "clean") และแปลงกลับเป็น line ending ของระบบตอน checkout ถ้า `core.autocrlf` เปิดอยู่ |
| `-text` (unset) | บังคับว่าไฟล์นี้ **ไม่ใช่** text — ปฏิบัติเหมือน binary เสมอ ไม่แตะ line ending ไม่ทำ diff แบบข้อความ |
| `text=auto` | ให้ Git **ตรวจจับอัตโนมัติ** ว่าไฟล์เป็น text หรือ binary (ดูจากเนื้อหาไฟล์ว่ามี null byte หรือไม่) ถ้าเป็น text ก็ normalize line ending ให้ |
| (ไม่ตั้งค่า / unspecified) | ใช้ค่า default ของ Git ซึ่งขึ้นกับ `core.autocrlf` แบบ global |

### Attribute `eol`

`eol` ใช้คู่กับ `text` เพื่อ **บังคับ line ending ชนิดใดชนิดหนึ่งอย่างเจาะจง** โดยไม่สนใจการตั้งค่า `core.autocrlf` ของผู้ใช้แต่ละคนเลย

| ค่า | ความหมาย |
|---|---|
| `eol=lf` | บังคับให้ไฟล์นี้มี line ending เป็น **LF** เสมอ ทั้งใน repository และตอน checkout ลงทุกเครื่อง ไม่ว่า `core.autocrlf` ของผู้ใช้จะตั้งเป็นอะไร |
| `eol=crlf` | บังคับให้ไฟล์นี้มี line ending เป็น **CRLF** เสมอ ทั้งใน repository และตอน checkout |

**ข้อสำคัญ:** `eol=lf` หรือ `eol=crlf` จะมีผลก็ต่อเมื่อไฟล์นั้นถูกระบุ (หรือ implied) ว่าเป็น `text` แล้วเท่านั้น — การใส่ `eol=lf` เฉย ๆ โดยไม่มี `text` จะทำให้ Git ตีความว่าเป็น `text eol=lf` โดยอัตโนมัติอยู่แล้ว (เพราะการระบุ `eol` แปลว่าไฟล์นั้นต้องเป็น text)

### ตัวอย่างการเขียนจริง

```gitattributes
# บังคับทุกไฟล์ที่ Git ตรวจจับว่าเป็น text ให้ normalize line ending อัตโนมัติ
* text=auto

# Shell script ต้องเป็น LF เสมอ ไม่ว่าจะ commit จาก OS ไหน
*.sh text eol=lf

# Windows batch/PowerShell script ควรเป็น CRLF เสมอ
*.bat text eol=crlf
*.ps1 text eol=crlf

# ไฟล์ source code ทั่วไป บังคับว่าเป็น text แน่นอน (เผื่อ text=auto ตรวจจับผิด)
*.js  text
*.ts  text
*.py  text
*.go  text

# ไฟล์ binary ห้ามแตะ line ending เด็ดขาด
*.png binary
*.jpg binary
*.exe binary
```

หมายเหตุ: `binary` ในตัวอย่างข้างต้นจริง ๆ แล้วคือ **macro attribute** ที่ Git กำหนดไว้ให้ล่วงหน้า เทียบเท่ากับการเขียน `-text -diff -merge` พร้อมกันในบรรทัดเดียว (จะอธิบายเพิ่มใน Step 603)

### กรณีศึกษา: ปัญหา shebang line พังเพราะ CRLF

ลองนึกภาพ repository ที่มีไฟล์ `deploy.sh`:

```bash
#!/bin/bash
echo "Deploying..."
```

ถ้านักพัฒนาคนหนึ่งใช้ Windows และมี `core.autocrlf=true` (แนะนำสำหรับ Windows ใน Part 02) แล้ว commit ไฟล์นี้โดยไม่มี `.gitattributes` ควบคุม ไฟล์ `deploy.sh` ที่ checkout ออกมาบนเครื่อง Windows จะมี CRLF ทุกบรรทัด ซึ่งตัว repository เองจะถูก normalize เป็น LF ให้ตอน commit ก็จริง (เพราะ `core.autocrlf=true` แปลงกลับเป็น LF ตอน commit) แต่ปัญหาที่แท้จริงคือ **ถ้ามีใครสัก 1 คนในทีมตั้งค่า `core.autocrlf=false` โดยไม่ได้ตั้งใจ** (ลืมตั้ง หรือใช้เครื่องใหม่ที่ยังไม่ได้ configure) ไฟล์ `.sh` ที่คนนั้น commit เข้ามาจะมี CRLF ติดไปใน repository ตรง ๆ และเมื่อ deploy ไปยัง server Linux ก็จะพังทันทีด้วย error แบบ:

```
bash: ./deploy.sh: /bin/bash^M: bad interpreter: No such file or directory
```

การแก้ปัญหานี้อย่างถาวรคือ **ห้ามพึ่งการตั้งค่าส่วนตัวของแต่ละคนเด็ดขาด** ให้บังคับผ่าน `.gitattributes` แทน เพราะไฟล์นี้ commit เข้า repo และมีผลกับทุกคนเหมือนกันเสมอ ไม่ว่าใครจะตั้งค่า `core.autocrlf` เป็นอะไรก็ตาม:

```gitattributes
*.sh text eol=lf
```

บรรทัดเดียวนี้การันตีว่า **ไม่ว่าใครจะ commit `deploy.sh` จากเครื่องไหนด้วยการตั้งค่าอะไรก็ตาม ไฟล์นี้จะถูก normalize เป็น LF เสมอทั้งใน repository และตอน checkout**

### คำสั่งตรวจสอบว่า attribute มีผลกับไฟล์ไหนบ้าง

Git มีคำสั่งสำหรับ debug attribute โดยเฉพาะ:

```bash
git check-attr text eol -- deploy.sh
```

ผลลัพธ์ตัวอย่าง:

```
deploy.sh: text: set
deploy.sh: eol: lf
```

หรือดูทุก attribute ที่มีผลกับไฟล์นั้นพร้อมกัน:

```bash
git check-attr --all -- deploy.sh
```

คำสั่งนี้จะใช้บ่อยมากตลอด Part นี้ เพราะเป็นวิธีเดียวที่มั่นใจได้ 100% ว่า `.gitattributes` ที่เขียนไปมีผลจริงตามที่ตั้งใจหรือไม่ — แนะนำให้รันตรวจทุกครั้งหลังแก้ไฟล์ `.gitattributes`

### ข้อควรระวัง: การเปลี่ยน `.gitattributes` ไม่มีผลย้อนหลังกับไฟล์ที่ commit ไปแล้ว

การเพิ่มหรือแก้ `text`/`eol` attribute ใน `.gitattributes` **ไม่ได้แปลงไฟล์ที่มีอยู่ใน working directory หรือใน history ทันที** มันจะมีผลกับไฟล์ที่ **checkout ใหม่** เท่านั้น ถ้าต้องการบังคับ normalize ไฟล์ที่มีอยู่แล้วให้ตรงกับ attribute ใหม่ ต้องรันคำสั่ง:

```bash
git add --renormalize .
git status   # ตรวจดูว่ามีไฟล์ไหนถูก renormalize บ้าง
git commit -m "Normalize line endings ตาม .gitattributes ใหม่"
```

คำสั่ง `git add --renormalize` จะไล่ตรวจไฟล์ทั้งหมดใน working directory เทียบกับ attribute ปัจจุบัน แล้ว stage การเปลี่ยนแปลง line ending ที่จำเป็นให้อัตโนมัติ เป็นขั้นตอนที่ทีมมักลืมทำหลังเพิ่ม `.gitattributes` ใหม่เข้าไปในโปรเจกต์เก่า

---

## Step 603: `diff` attribute — กำหนดวิธี diff ไฟล์พิเศษ

ค่า default ของ Git คือพยายาม diff ทุกไฟล์แบบข้อความ (line-by-line text diff) ซึ่งใช้ได้ดีกับ source code แต่ใช้ไม่ได้เลยกับไฟล์ประเภทอื่น เช่น ไฟล์ binary, ไฟล์ Word, ไฟล์ PDF, หรือไฟล์ที่มี custom format เฉพาะทาง

Attribute `diff` ควบคุมว่า Git จะ diff ไฟล์ที่ match pattern นั้นด้วยวิธีไหน

### ค่าพื้นฐาน: `-diff` (ปิด diff แบบข้อความ)

```gitattributes
*.png -diff
```

เมื่อไฟล์ถูกกำหนด `-diff` คำสั่ง `git diff` จะไม่พยายามแสดงผลต่างแบบบรรทัดต่อบรรทัดอีกต่อไป แต่จะขึ้นข้อความสั้น ๆ แทน:

```
Binary files a/logo.png and b/logo.png differ
```

ซึ่งดีกว่าการปล่อยให้ Git พยายาม diff แบบข้อความกับไฟล์ binary ที่จะได้ output เป็นตัวอักษรแปลก ๆ เต็มหน้าจอ terminal (และอาจทำให้ terminal ค้างในบางกรณี)

### Macro attribute `binary`

Git มี attribute สำเร็จรูปชื่อ `binary` ที่รวม 3 attribute เข้าด้วยกันในคำเดียว:

```gitattributes
*.png binary
```

เทียบเท่ากับการเขียน:

```gitattributes
*.png -text -diff -merge
```

พูดคือ: ไม่ normalize line ending (`-text`), ไม่ diff แบบข้อความ (`-diff`), และไม่พยายาม merge แบบข้อความ (`-merge` จะขึ้น conflict แบบ binary ทันทีถ้ามีการแก้ไขพร้อมกันจากทั้งสองฝั่ง) — เป็น attribute ที่ **ควรใช้กับไฟล์ binary ทุกประเภทเสมอ** เพราะครอบคลุมปัญหาทั้ง 3 ด้านในบรรทัดเดียว

### กำหนด diff driver แบบกำหนดเอง (custom diff driver)

ความสามารถที่ทรงพลังกว่านั้นคือการกำหนดให้ Git ใช้ **โปรแกรมภายนอก** ในการแปลงไฟล์ให้อยู่ในรูปแบบข้อความก่อน diff เรียกกลไกนี้ว่า **`textconv`**

ตัวอย่างคลาสสิกที่สุด: ไฟล์ Microsoft Word (`.docx`) จริง ๆ แล้วเป็นไฟล์ zip ที่บีบอัด XML ไว้ข้างใน ถ้า diff ตรง ๆ จะไม่มีความหมายอะไรเลย แต่ถ้ามีเครื่องมือแปลง `.docx` เป็นข้อความล้วนก่อน (เช่นใช้ `pandoc` หรือ `docx2txt`) ก็จะ diff เนื้อหาได้จริง

ขั้นตอนการตั้งค่ามี 2 ส่วน:

**1. กำหนดชื่อ diff driver ใน `.gitattributes`:**

```gitattributes
*.docx diff=word
```

**2. สั่งให้ diff driver ชื่อ `word` ใช้โปรแกรมอะไรแปลงไฟล์ ผ่าน `git config`:**

```bash
git config diff.word.textconv "pandoc --to=plain"
```

หรือเก็บไว้ใน `.git/config` โดยตรง:

```ini
[diff "word"]
    textconv = pandoc --to=plain
```

หลังจากตั้งค่านี้แล้ว เวลารัน `git diff` กับไฟล์ `.docx` ใด ๆ Git จะเรียก `pandoc --to=plain <ไฟล์>` แปลงทั้งเวอร์ชันเก่าและใหม่เป็นข้อความก่อน แล้วค่อยเอาผลลัพธ์ข้อความมา diff กันแบบปกติ ทำให้เห็นความเปลี่ยนแปลงของเนื้อหาจริง ๆ แทนที่จะเห็นแค่ "Binary files differ"

### ตัวอย่างอื่น: diff ไฟล์ PDF ด้วย `pdftotext`

```gitattributes
*.pdf diff=pdf
```

```bash
git config diff.pdf.textconv "pdftotext -layout"
```

### ตัวอย่างอื่น: diff ไฟล์ Jupyter Notebook (`.ipynb`) แบบอ่านง่ายขึ้น

ไฟล์ `.ipynb` เป็น JSON ที่มี metadata และ output (เช่นรูปภาพ base64) ปนอยู่กับโค้ด ทำให้ diff ดิบ ๆ อ่านยากมาก มีเครื่องมือชื่อ `nbdime` ที่ทำหน้าที่นี้โดยเฉพาะ:

```gitattributes
*.ipynb diff=jupyternotebook
```

```bash
git config diff.jupyternotebook.command "git-nbdiffdriver diff"
```

### สรุปตารางค่าที่ใช้ได้กับ `diff`

| ค่า | ความหมาย |
|---|---|
| `diff` (set, default) | diff แบบข้อความปกติ |
| `-diff` (unset) | แสดงแค่ "Binary files differ" ไม่พยายาม diff เนื้อหา |
| `diff=<driver-name>` | ใช้ diff driver ที่ตั้งชื่อไว้ใน `git config diff.<driver-name>.*` |
| `binary` (macro) | เทียบเท่า `-text -diff -merge` — ใช้กับไฟล์ binary ทั่วไป |

**ข้อควรรู้:** การตั้งค่า `diff.<name>.textconv` ผ่าน `git config` เป็นการตั้งค่า **เฉพาะเครื่อง** ไม่สามารถ commit เข้า repository ได้โดยตรง เพราะเป็นคำสั่งเรียกโปรแกรมภายนอกซึ่งอาจไม่มีในทุกเครื่อง (ด้วยเหตุผลด้านความปลอดภัย — ถ้าให้ repository กำหนดคำสั่งที่รันได้เองได้ อาจถูกใช้เป็นช่องโหว่โจมตี) ดังนั้นทีมที่อยากให้ทุกคนได้ textconv เหมือนกันต้องมีเอกสารแนะนำให้แต่ละคนรันคำสั่ง `git config` เอง หรือใช้ script ตั้งค่าอัตโนมัติตอน setup โปรเจกต์ (เช่นใน onboarding script)

---

## Step 604: `merge` attribute — custom merge strategy ต่อไฟล์

เมื่อ Git merge สอง branch เข้าด้วยกัน ค่า default คือใช้ **3-way merge algorithm** (เทียบ base, ours, theirs) แบบบรรทัดต่อบรรทัด ซึ่งเหมาะกับ source code แต่มีไฟล์บางประเภทที่การ merge แบบบรรทัดต่อบรรทัดไม่เหมาะสมเลย หรือทีมอยากกำหนดพฤติกรรมพิเศษ

### ค่าพื้นฐานที่ Git มีให้

| ค่า | ความหมาย |
|---|---|
| `merge` (default) | ใช้ 3-way merge แบบข้อความปกติ |
| `-merge` (unset) | ถ้ามีการแก้ไขทั้งสองฝั่ง (both modified) ให้ถือว่า conflict ทันที ไม่พยายาม merge อัตโนมัติเลย (เหมาะกับ binary) |
| `merge=<driver-name>` | ใช้ merge driver ที่กำหนดเองผ่าน `git config` |

### `merge=ours` — driver สำเร็จรูปที่ Git เตรียมไว้ให้

Git มี merge driver ชื่อ `ours` ให้ใช้งานได้ทันที (built-in) แนวคิดคือ **เวลา merge ไฟล์นี้ ให้ใช้เวอร์ชันของ branch ปัจจุบัน (ours) เสมอ ไม่สนใจการเปลี่ยนแปลงจากอีกฝั่งเลย ไม่มีวัน conflict**

การใช้งานต้อง**เปิดใช้ driver ก่อน**ด้วย `git config` (เพราะเหตุผลความปลอดภัยเดียวกับ textconv — driver ที่รันโปรแกรมได้จะไม่ผูกกับ repo โดยตรง) แต่ `merge=ours` เป็นข้อยกเว้นพิเศษที่ **built-in อยู่แล้วใน Git ตั้งแต่เวอร์ชันใหม่ ๆ** ไม่ต้องตั้งค่าเพิ่ม:

```gitattributes
config/production.secrets.yml merge=ours
```

ตัวอย่างสถานการณ์จริง: ไฟล์ config ที่แต่ละ environment (dev/staging/production) มีค่าต่างกัน และไฟล์นี้ถูก commit ไว้ใน branch เฉพาะของแต่ละ environment เวลา merge branch `develop` เข้า `production` ทีมไม่อยากให้ค่า config ของ production ถูกเขียนทับด้วยค่าจาก develop จึงตั้ง `merge=ours` ไว้ที่ไฟล์นั้น เพื่อบอกว่า "merge เสร็จแล้ว ไฟล์นี้ให้คงค่าของฝั่ง production (ours) ไว้เสมอ ไม่ต้องเอาการเปลี่ยนแปลงจาก develop มาปนเลย"

**ข้อควรระวังสำคัญ:** `merge=ours` ไม่ได้แปลว่า "เก็บทั้งสองเวอร์ชันไว้" หรือ "ให้คนตัดสินใจ" — มันหมายถึง **ทิ้งการเปลี่ยนแปลงของอีกฝั่งไปเลยแบบเงียบ ๆ โดยไม่มี conflict ให้เห็นด้วยซ้ำ** ถ้าใช้ผิดที่อาจทำให้ code หายไปโดยไม่มีใครรู้ตัว ควรใช้เฉพาะกับไฟล์ที่แน่ใจจริง ๆ ว่าไม่ต้องการ merge เนื้อหาจากฝั่งไหนเข้าด้วยกันเลย เช่นไฟล์ config เฉพาะ environment, ไฟล์ log, หรือไฟล์ที่ generate อัตโนมัติที่ไม่สำคัญว่าจะเป็นเวอร์ชันไหน

### custom merge driver แบบเต็มรูปแบบ

สำหรับกรณีที่ซับซ้อนกว่านั้น เช่นไฟล์ที่มีโครงสร้างพิเศษ (JSON, XML) ที่อยากให้ merge อย่างฉลาดตามโครงสร้างข้อมูลแทนที่จะ merge แบบบรรทัดต่อบรรทัด สามารถเขียน merge driver เองได้:

```gitattributes
*.json merge=json-merge
```

```ini
[merge "json-merge"]
    name = สคริปต์ merge ไฟล์ JSON แบบเข้าใจโครงสร้าง
    driver = json-merge-tool %O %A %B %L
```

ตัวแปรที่ Git ส่งให้โปรแกรม driver มี 4 ตัวหลัก:

| ตัวแปร | ความหมาย |
|---|---|
| `%O` | path ของไฟล์ชั่วคราวที่เก็บเนื้อหา **base** (common ancestor) |
| `%A` | path ของไฟล์ชั่วคราวที่เก็บเนื้อหาฝั่ง **current branch (ours)** — driver ต้องเขียนผลลัพธ์สุดท้ายกลับมาที่ไฟล์นี้ |
| `%B` | path ของไฟล์ชั่วคราวที่เก็บเนื้อหาฝั่ง **branch ที่กำลังจะ merge เข้ามา (theirs)** |
| `%L` | ความยาว (ตัวเลข) ของ conflict marker ที่ควรใช้ ถ้า driver ต้องการแสดง conflict แบบมาตรฐาน |

Driver ต้อง exit code เป็น `0` ถ้า merge สำเร็จไม่มี conflict หรือ non-zero ถ้ามี conflict (Git จะรายงานว่าไฟล์นี้ conflict และปล่อยให้ผู้ใช้แก้เอง)

### ตัวอย่างที่นิยมใช้จริง: `merge=union` สำหรับไฟล์ CHANGELOG

Git มี built-in merge driver ชื่อ `union` ที่ใช้บ่อยกับไฟล์ที่เป็น "รายการเรียงต่อกัน" เช่น CHANGELOG หรือไฟล์ list ต่าง ๆ ที่ไม่สนใจลำดับ แนวคิดคือ **แทนที่จะขึ้น conflict เวลาทั้งสองฝั่งเพิ่มบรรทัดใหม่คนละบรรทัดในตำแหน่งใกล้กัน ให้เอาบรรทัดของทั้งสองฝั่งมารวมกันหมด (union) โดยตัดบรรทัดที่ซ้ำกันทิ้ง**

```gitattributes
CHANGELOG.md merge=union
```

`union` เป็นชื่อพิเศษที่ Git รู้จักในตัวอยู่แล้วเช่นเดียวกับ `ours` ไม่ต้องตั้งค่า `git config` เพิ่มเติม

### ตารางสรุป merge attribute

| ค่า | ต้องตั้งค่า `git config` เพิ่มไหม | ใช้เมื่อไหร่ |
|---|---|---|
| `merge` (default) | ไม่ต้อง | source code ทั่วไป |
| `-merge` | ไม่ต้อง | binary ที่ไม่อยาก merge อัตโนมัติเลย ให้ conflict เสมอ |
| `merge=ours` | ไม่ต้อง (built-in) | ไฟล์ที่อยากให้ฝั่งปัจจุบันชนะเสมอ ไม่สน branch ที่เข้ามา |
| `merge=union` | ไม่ต้อง (built-in) | ไฟล์รายการที่อยากรวมทุกบรรทัดจากทั้งสองฝั่ง เช่น CHANGELOG |
| `merge=<custom>` | ต้องตั้งเอง | โครงสร้างข้อมูลพิเศษที่ต้องการ logic merge เฉพาะทาง |

---

## Step 605: `export-ignore`, `export-subst` — ควบคุมพฤติกรรมตอน `git archive`

คำสั่ง `git archive` ใช้สำหรับสร้างไฟล์ zip หรือ tar ของ snapshot ใด ๆ ใน repository โดยไม่รวมโฟลเดอร์ `.git` เข้าไปด้วย เหมาะสำหรับการส่งมอบซอร์สโค้ดให้ลูกค้าหรือแนบไปกับ release โดยไม่อยากให้เห็นประวัติทั้งหมด

```bash
git archive --format=zip --output=release-v1.0.zip HEAD
```

ปัญหาคือ **`git archive` จะเอาทุกไฟล์ที่ถูก track ใน commit นั้นมาใส่ใน zip ทั้งหมด** รวมถึงไฟล์ที่ควรอยู่ใน repository แต่ไม่ควรอยู่ในไฟล์ที่ส่งมอบ เช่น ไฟล์ CI config, ไฟล์ทดสอบภายใน, หรือไฟล์เอกสารสำหรับนักพัฒนาเท่านั้น

### `export-ignore` — ไฟล์นี้ track ใน repo แต่ไม่ต้องอยู่ใน archive

```gitattributes
.github/            export-ignore
.gitattributes      export-ignore
.gitignore          export-ignore
tests/              export-ignore
docs/internal/      export-ignore
CONTRIBUTING.md     export-ignore
```

ไฟล์/โฟลเดอร์ที่ตั้ง `export-ignore` จะ**ยังคง commit และ push อยู่ใน repository ตามปกติทุกประการ** — attribute นี้มีผลแค่ตอนเรียก `git archive` เท่านั้น ไม่กระทบพฤติกรรมอื่นเลย นี่คือความต่างสำคัญจาก `.gitignore` ที่ทำให้ไฟล์ไม่ถูก track ตั้งแต่แรก

### `export-subst` — แทนที่ placeholder ด้วยข้อมูล Git ตอน archive

`export-subst` เปิดใช้กลไก **keyword substitution** แบบง่าย ๆ ที่ Git รองรับเฉพาะตอนสร้าง archive เท่านั้น ใช้คู่กับ placeholder รูปแบบ `$Format:...$`

ตัวอย่างการใช้งาน สมมติมีไฟล์ `VERSION.txt`:

```
Build: $Format:%H$
Date:  $Format:%ci$
Tag:   $Format:%(describe)$
```

กำหนดใน `.gitattributes`:

```gitattributes
VERSION.txt export-subst
```

เมื่อรัน `git archive` เนื้อหาไฟล์ `VERSION.txt` ในไฟล์ zip ที่ได้จะถูกแทนที่ placeholder ด้วยข้อมูลจริงโดยอัตโนมัติ เช่น:

```
Build: 8f3c2a1d4e5b6789...
Date:  2026-09-20 14:32:11 +0700
Tag:   v1.4.2
```

โดยที่ **ไฟล์ต้นฉบับในเครื่องพัฒนายังคงมี placeholder `$Format:...$` เดิม ไม่ถูกแก้ไข** — การแทนที่เกิดขึ้นเฉพาะตอน archive เท่านั้น pattern ที่ใช้ได้กับ `%H`, `%h`, `%ci`, `%(describe)` เป็น format string เดียวกับที่ใช้ใน `git log --pretty=format:`

### กรณีใช้งานจริง: GitHub สร้าง release archive อัตโนมัติ

เวลาสร้าง release บน GitHub ระบบจะเรียก `git archive` เบื้องหลังเพื่อสร้างไฟล์ `.zip`/`.tar.gz` ให้ดาวน์โหลดอัตโนมัติ ดังนั้นการตั้งค่า `export-ignore` และ `export-subst` ให้ถูกต้องใน `.gitattributes` จะมีผลโดยตรงกับไฟล์ที่ผู้ใช้ทั่วไปดาวน์โหลดจากหน้า Releases ของ GitHub ด้วยเช่นกัน โดยไม่ต้องทำอะไรเพิ่ม

### ตารางสรุป

| Attribute | ผลตอน `git archive` | ผลกับ repository ปกติ |
|---|---|---|
| `export-ignore` | ไฟล์/โฟลเดอร์นี้ไม่ถูกใส่เข้าไปใน archive | ไม่มีผล — ยัง track ปกติ |
| `export-subst` | แทนที่ `$Format:...$` ด้วยค่าจริงในเนื้อหาไฟล์ที่อยู่ใน archive | ไม่มีผล — ไฟล์ต้นฉบับใน repo ไม่ถูกแก้ |

---

## Step 606: `linguist-*` attributes — ควบคุมการนับสัดส่วนภาษาโปรแกรมมิ่งที่แสดงบนหน้า GitHub repo

ทุกคนที่เคยเปิดหน้า repository บน GitHub น่าจะเคยเห็นแถบสีที่แสดงสัดส่วนภาษาโปรแกรมมิ่ง เช่น "JavaScript 82.3% · CSS 10.1% · HTML 7.6%" ที่ด้านล่างขวาของหน้า repo แถบนี้คำนวณโดยไลบรารี open source ของ GitHub ชื่อ **Linguist**

ปัญหาที่พบบ่อยคือ Linguist นับไฟล์ทุกไฟล์ที่มันจำแนกได้ว่าเป็นภาษาโปรแกรมมิ่ง โดยไม่รู้บริบทของโปรเจกต์ ทำให้ตัวเลขคลาดเคลื่อนจากความเป็นจริงบ่อยมาก เช่น:

- โฟลเดอร์ `vendor/` หรือ `node_modules/` (ถ้าเผลอ commit เข้าไป) ที่เป็น dependency ของคนอื่น ไม่ใช่โค้ดที่ทีมเขียนเอง แต่ถูกนับรวมไปด้วย
- ไฟล์ที่ generate อัตโนมัติ เช่น `dist/bundle.min.js`, ไฟล์ที่แปลจาก TypeScript เป็น JavaScript แล้ว commit ไว้ (`*.generated.ts`)
- โปรเจกต์ที่มีเอกสารจำนวนมากเป็น Markdown แต่ core logic จริง ๆ เป็นภาษาอื่นเพียงเล็กน้อย ทำให้สัดส่วนดูเพี้ยน

Linguist อ่านค่าคอนฟิกจาก `.gitattributes` โดยตรง ผ่าน attribute ตระกูล `linguist-*`

### `linguist-generated` — บอกว่าไฟล์นี้ถูก generate อัตโนมัติ

```gitattributes
dist/**              linguist-generated=true
*.min.js             linguist-generated=true
package-lock.json    linguist-generated=true
```

ไฟล์ที่ตั้ง `linguist-generated=true` จะ:
- ไม่ถูกนับรวมในสัดส่วนภาษา
- ถูกซ่อนโดย default ในหน้า diff ของ Pull Request บน GitHub (แสดงเป็น "This file has been truncated / marked as generated" พร้อมปุ่มกดดูถ้าต้องการ)

### `linguist-vendored` — บอกว่าเป็นโค้ดของบุคคลภายนอก (third-party)

```gitattributes
vendor/**            linguist-vendored=true
third_party/**       linguist-vendored=true
```

ตามค่า default จริง ๆ Linguist มีรายชื่อ path pattern มาตรฐานที่ถือว่าเป็น vendored อยู่แล้ว (เช่น `vendor/`, `node_modules/`, `*.min.js`) แต่การกำหนดเองชัดเจนใน `.gitattributes` ช่วยครอบคลุมกรณีเฉพาะของโปรเจกต์ที่ไม่ตรงกับ pattern มาตรฐาน

### `linguist-documentation` — บอกว่าเป็นไฟล์เอกสาร ไม่ใช่โค้ด

```gitattributes
docs/**              linguist-documentation=true
```

### `linguist-language` — บังคับให้จัดเป็นภาษาที่ระบุ (override การเดาอัตโนมัติ)

บางครั้ง Linguist เดาภาษาผิด เช่นไฟล์ `.h` ที่จริง ๆ เป็น C++ header แต่ Linguist อาจจัดเป็น C เพราะนามสกุลไฟล์เดียวกันใช้ได้ทั้งสองภาษา:

```gitattributes
*.h linguist-language=C++
```

### `linguist-detectable` — บังคับให้นับ/ไม่นับ แม้จะอยู่ใน path ที่ปกติถูกมองข้าม

```gitattributes
# ปกติไฟล์ในโฟลเดอร์ examples/ อาจถูก Linguist มองข้ามเป็น documentation
# แต่ทีมนี้อยากให้นับโค้ดตัวอย่างเป็นส่วนหนึ่งของสัดส่วนภาษาด้วย
examples/**.py linguist-detectable=true
```

### ตัวอย่างการตั้งค่าแบบเต็มสำหรับโปรเจกต์ทั่วไป

```gitattributes
# ไฟล์ที่ generate อัตโนมัติ ไม่ควรนับเป็นภาษาโปรแกรมมิ่งของทีม
dist/**                 linguist-generated=true
build/**                linguist-generated=true
*.lock                  linguist-generated=true
package-lock.json       linguist-generated=true
yarn.lock               linguist-generated=true
pnpm-lock.yaml          linguist-generated=true

# Dependency ของบุคคลภายนอกที่ (เผลอ หรือจำเป็นต้อง) commit เข้ามา
vendor/**               linguist-vendored=true

# เอกสารประกอบ ไม่ใช่ตัวโค้ด
docs/**                 linguist-documentation=true
*.md                    linguist-documentation=true
```

**ข้อควรรู้:** attribute ตระกูล `linguist-*` เป็น**ข้อตกลงเฉพาะของ GitHub** (GitHub-specific convention) ไม่ใช่ attribute มาตรฐานของ Git core มันทำงานได้เพราะไลบรารี Linguist ของ GitHub ถูกเขียนมาให้อ่านค่านี้จาก `.gitattributes` โดยเฉพาะ — ถ้านำ repository ไปใช้กับ GitLab หรือ Bitbucket, attribute เหล่านี้จะไม่มีผลอะไรเลย (เพราะแพลตฟอร์มอื่นมีระบบนับภาษาของตัวเองที่ไม่อ่านค่าเหล่านี้) แต่ก็ไม่ก่อให้เกิดปัญหาอะไร เพราะ Git core จะแค่เก็บไว้เฉย ๆ โดยไม่ได้เอาไปใช้ทำอะไรถ้าไม่มีเครื่องมือที่รู้จักมันอ่าน

---

## Step 607: filter attributes (`filter=lfs`, clean/smudge) — เกริ่นกลไกก่อนเจาะลึกจริงจังใน Git LFS ที่ Part 62

หนึ่งใน attribute ที่ทรงพลังที่สุดของ `.gitattributes` คือ **`filter`** ซึ่งเปิดโอกาสให้เราแทรกโปรแกรมภายนอกเข้าไปแปลงเนื้อหาไฟล์ **ทุกครั้งที่มีการ commit และ checkout** — นี่คือกลไกพื้นฐานที่ **Git LFS (Large File Storage)** ซึ่งเราจะเรียนแบบเจาะลึกทั้ง Part ใน **Part 62** ใช้เป็นแกนหลักในการทำงาน

### แนวคิดของ filter driver: clean และ smudge

Filter driver ประกอบด้วยโปรแกรม 2 ตัวที่ทำงานสวนทางกัน:

| ขั้นตอน | เรียกตอนไหน | หน้าที่ |
|---|---|---|
| **clean** | ตอน `git add` / commit (working directory → repository) | แปลงเนื้อหาไฟล์จริง **ก่อน**เก็บเข้า Git object database |
| **smudge** | ตอน `git checkout` (repository → working directory) | แปลงเนื้อหาที่เก็บไว้กลับมาเป็นไฟล์จริงในเครื่อง |

```
                    clean
Working Directory ────────▶ Git Repository (object database)
                   ◀────────
                    smudge
```

พูดง่าย ๆ : **clean คือตอน "เก็บเข้า" ส่วน smudge คือตอน "เอาออกมาใช้"** — ชื่อทั้งสองมาจากแนวคิดดั้งเดิมของ RCS/CVS ที่ clean คือ "ทำความสะอาดก่อนเก็บ" (เช่นตัดข้อมูลที่ไม่จำเป็นออก) และ smudge คือ "แต้ม/เติมข้อมูลกลับเข้าไป" (เช่นเติม metadata ที่ตัดออกไปกลับคืน)

### ตัวอย่างง่าย ๆ ที่ไม่เกี่ยวกับ LFS: filter ที่ลบ trailing whitespace อัตโนมัติ

```gitattributes
*.py filter=whitespace-clean
```

```ini
[filter "whitespace-clean"]
    clean = sed -e 's/[ \t]*$//'
    smudge = cat
```

ในตัวอย่างนี้ ทุกครั้งที่ `git add` ไฟล์ `.py` โปรแกรม `sed` จะถูกเรียกเพื่อตัด whitespace ท้ายบรรทัดออกก่อนเก็บเข้า repository โดยอัตโนมัติ (clean) ส่วนตอน checkout ก็แค่คืนไฟล์กลับมาตรง ๆ ไม่ต้องแปลงอะไร (smudge = `cat` คือส่งข้อมูลผ่านตรง ๆ ไม่แก้ไข)

### วิธีที่ Git LFS ใช้กลไกนี้ (ภาพรวมคร่าว ๆ ก่อนเจาะลึกใน Part 62)

Git LFS ใช้ `filter=lfs` เพื่อแก้ปัญหาไฟล์ขนาดใหญ่ (เช่นไฟล์วิดีโอ, dataset, ไฟล์ design) ที่ไม่ควรเก็บเนื้อหาเต็ม ๆ ไว้ใน Git object database โดยตรง (เพราะ Git ไม่ได้ออกแบบมาให้จัดการไฟล์ใหญ่ได้อย่างมีประสิทธิภาพ):

```gitattributes
*.psd filter=lfs diff=lfs merge=lfs -text
```

หลักการทำงานคร่าว ๆ (จะอธิบายละเอียดทั้งหมดใน Part 62):

1. **clean**: ตอน `git add` ไฟล์ `.psd` ตัว LFS clean filter จะ **อัปโหลดเนื้อหาไฟล์จริงไปเก็บที่ LFS storage server แยกต่างหาก** แล้วแทนที่เนื้อหาในไฟล์ที่จะเก็บเข้า Git object database ด้วย **"pointer file"** ขนาดเล็กมาก (ไม่กี่ร้อย byte) ที่มีแค่ hash และขนาดไฟล์อ้างอิงไปยังไฟล์จริง
2. **smudge**: ตอน `git checkout` ตัว LFS smudge filter จะอ่าน pointer file นั้น แล้วไป**ดาวน์โหลดเนื้อหาไฟล์จริงจาก LFS storage server** กลับมาวางไว้ใน working directory แทนที่ pointer file

ผลลัพธ์คือ **Git repository เก็บแค่ pointer file ขนาดเล็ก ไม่เก็บไฟล์ใหญ่จริง ๆ** ทำให้ clone/fetch เร็วขึ้นมหาศาลสำหรับโปรเจกต์ที่มีไฟล์ asset ขนาดใหญ่จำนวนมาก

### ทำไมต้องเรียน filter attribute ก่อนเข้า Part 62

การเข้าใจว่า `filter`, `clean`, `smudge` เป็นกลไกทั่วไปของ `.gitattributes` (ไม่ใช่ฟีเจอร์เฉพาะของ LFS) ช่วยให้เข้าใจ Git LFS ได้อย่างถ่องแท้ เพราะเมื่อไปถึง Part 62 คุณจะเห็นว่าคำสั่ง `git lfs install` และ `git lfs track "*.psd"` ที่ดูเหมือนเวทมนตร์ จริง ๆ แล้วเบื้องหลังมันแค่:

1. เขียนบรรทัด `*.psd filter=lfs diff=lfs merge=lfs -text` ลงใน `.gitattributes` ให้อัตโนมัติ
2. ตั้งค่า `git config filter.lfs.clean`, `filter.lfs.smudge`, `filter.lfs.process` ให้ชี้ไปที่โปรแกรม `git-lfs` ที่ติดตั้งไว้

ไม่มีอะไรลึกลับไปกว่ากลไก `.gitattributes` ที่เราเพิ่งเรียนไปเลย — เป็นการนำ pattern เดียวกันมาประยุกต์ใช้กับปัญหาไฟล์ใหญ่โดยเฉพาะ

---

## Step 608: `.gitattributes` vs `.gitignore` ต่างกันอย่างไร (คนละหน้าที่กันโดยสิ้นเชิง)

ผู้เรียนใหม่จำนวนมากสับสนระหว่างสองไฟล์นี้ เพราะหน้าตาคล้ายกัน (plain text, ใช้ glob pattern, วางไว้ที่ root ของ repo) แต่ **หน้าที่ของทั้งสองไฟล์ตรงข้ามกันโดยสิ้นเชิง**

### ความแตกต่างหลัก

| หัวข้อ | `.gitignore` | `.gitattributes` |
|---|---|---|
| หน้าที่หลัก | บอกว่า **ไฟล์ไหนไม่ต้อง track เลย** | บอกว่า **ไฟล์ที่ track อยู่แล้ว ควรถูกปฏิบัติอย่างไร** |
| ไฟล์ที่ match pattern | Git **ไม่รู้จัก / ไม่เก็บประวัติ** ไฟล์นั้นเลย | Git **ยัง track และเก็บประวัติปกติทุกประการ** |
| มีผลกับไฟล์ที่ commit ไปแล้วไหม | ไม่มีผล (ถ้าไฟล์ถูก track อยู่แล้วก่อนเพิ่มลง `.gitignore` มันจะยัง track ต่อไป ต้อง `git rm --cached` เอง) | มีผลย้อนหลังบางส่วน (เช่น diff, merge จะเปลี่ยนพฤติกรรมทันที) แต่ line ending ต้อง `git add --renormalize` เอง |
| ใช้ควบคุมอะไรได้บ้าง | แค่อย่างเดียว: ไม่ track/ไม่แสดงใน `git status` | หลายด้าน: line ending, diff, merge, archive, language stats, filter |
| syntax pattern | glob pattern | glob pattern (เหมือนกัน) แต่ตามด้วยชื่อ attribute |

### เปรียบเทียบด้วยตัวอย่างที่ตัดกันชัดเจน

```gitignore
# .gitignore — บอกว่า "อย่าเก็บไฟล์พวกนี้เข้า Git เลย"
node_modules/
*.log
.env
dist/
```

```gitattributes
# .gitattributes — บอกว่า "ไฟล์พวกนี้เก็บอยู่แล้ว แต่ให้ปฏิบัติแบบนี้"
*.sh          text eol=lf
*.png         binary
*.docx        diff=word
package-lock.json  merge=ours linguist-generated=true
```

สังเกตว่า `node_modules/` อยู่ใน `.gitignore` เพราะ**ไม่อยาก track เลย** ในขณะที่ `package-lock.json` อยู่ใน `.gitattributes` เพราะ**จำเป็นต้อง track (เพื่อ lock เวอร์ชัน dependency ให้ทุกคนตรงกัน) แต่อยากควบคุมพฤติกรรมการ merge และไม่อยากให้นับเป็นภาษาโปรแกรมมิ่ง**

### กรณีที่ต้องใช้ทั้งสองไฟล์ร่วมกัน

บางครั้งไฟล์ประเภทเดียวกันต้องปรากฏในทั้งสองไฟล์ด้วยเหตุผลคนละอย่าง เช่น:

```gitignore
# .gitignore
*.generated.json
```

```gitattributes
# .gitattributes — สำหรับไฟล์ .generated.json ตัวที่ยังจำเป็นต้อง track อยู่บางไฟล์
config/schema.generated.json export-ignore linguist-generated=true
```

ในตัวอย่างนี้ไฟล์ `.generated.json` ส่วนใหญ่ถูก ignore ไม่ track เลย แต่มีไฟล์เฉพาะเจาะจงหนึ่งไฟล์ (`config/schema.generated.json`) ที่ต้อง track เพราะจำเป็นต้องใช้ตอน build แต่ทีมไม่อยากให้มันติดไปกับ archive ตอน release และไม่อยากให้นับเป็นส่วนหนึ่งของภาษาโปรแกรมมิ่งของโปรเจกต์

### คำถามที่พบบ่อย: "ทำไมไม่รวมสองไฟล์นี้เป็นไฟล์เดียว"

เหตุผลเชิงออกแบบคือทั้งสองไฟล์ทำงานคนละจุดในกระบวนการของ Git โดยสิ้นเชิง:

- `.gitignore` ทำงานตอน **`git status` / `git add`** เพื่อตัดสินใจว่าไฟล์ไหนควรอยู่ใน "untracked files" หรือถูกมองข้ามไปเลย — เป็นเรื่องของ **การมองเห็น (visibility)**
- `.gitattributes` ทำงานตอน **checkout, commit, diff, merge, archive** ของไฟล์ที่ Git **รู้จักอยู่แล้ว** — เป็นเรื่องของ **พฤติกรรม (behavior)**

การแยกไฟล์ทำให้แต่ละไฟล์มีความรับผิดชอบเดียวชัดเจน (single responsibility) และทำให้อ่านเข้าใจง่ายกว่าการรวมทุกอย่างไว้ในไฟล์เดียวที่ต้องแยกแยะว่าบรรทัดไหนหมายถึงอะไร

---

## Step 609: Best practices การตั้งค่า `.gitattributes` มาตรฐานสำหรับโปรเจกต์ cross-platform

หลังจากเรียนทุก attribute แยกกันมาแล้ว มาถึงเวลาประกอบทุกอย่างเข้าด้วยกันเป็นไฟล์ `.gitattributes` ที่ใช้งานได้จริงในโปรเจกต์จริง โดยยึดหลักการต่อไปนี้:

### หลักการออกแบบ `.gitattributes` ที่ดี

1. **เริ่มด้วยกฎกว้าง ๆ ก่อน แล้วค่อยเจาะจงลงมา** — เพราะบรรทัดล่างสุดที่ match จะชนะ
2. **บังคับ `text=auto` เป็นค่าเริ่มต้นของทุกไฟล์** เพื่อให้ Git ตรวจจับ text/binary อัตโนมัติเป็น baseline
3. **ระบุนามสกุลไฟล์ที่สำคัญอย่างชัดเจน** อย่าพึ่งการตรวจจับอัตโนมัติเพียงอย่างเดียวกับไฟล์ที่มีผลกระทบสูง (เช่น shell script)
4. **binary ทุกชนิดต้องระบุ `binary` หรือ `-text` เสมอ** ไม่ปล่อยให้ Git เดา
5. **แยกส่วน comment ให้ชัดเจนเป็นหมวดหมู่** เพื่อให้คนอื่นในทีมอ่านและแก้ไขต่อได้ง่าย
6. **commit ไฟล์นี้เข้า repository เสมอ** ไม่ใช้ `$GIT_DIR/info/attributes` สำหรับกฎที่ทั้งทีมต้องใช้ร่วมกัน

### ตัวอย่างไฟล์ `.gitattributes` มาตรฐานฉบับเต็ม (ใช้งานได้จริง)

```gitattributes
# ==========================================================================
# .gitattributes — ควบคุมพฤติกรรมของ Git ต่อไฟล์แต่ละประเภทในโปรเจกต์นี้
# ==========================================================================

# --------------------------------------------------------------------------
# 1) Line ending — ค่าเริ่มต้น: ให้ Git ตรวจจับ text/binary อัตโนมัติ
#    แล้ว normalize line ending เป็น LF เสมอในฝั่ง repository
# --------------------------------------------------------------------------
* text=auto eol=lf

# --------------------------------------------------------------------------
# 2) Source code — บังคับว่าเป็น text แน่นอน (กันเคส text=auto ตรวจจับผิด)
# --------------------------------------------------------------------------
*.js    text
*.jsx   text
*.ts    text
*.tsx   text
*.json  text
*.css   text
*.scss  text
*.html  text
*.py    text
*.go    text
*.java  text
*.rb    text
*.php   text
*.c     text
*.h     text
*.cpp   text
*.md    text
*.yml   text
*.yaml  text
*.xml   text
*.sql   text

# --------------------------------------------------------------------------
# 3) Script ที่ line ending สำคัญมากต่อการรันงาน — บังคับ eol แบบเจาะจง
# --------------------------------------------------------------------------
*.sh    text eol=lf
*.bash  text eol=lf
*.bat   text eol=crlf
*.cmd   text eol=crlf
*.ps1   text eol=crlf

# --------------------------------------------------------------------------
# 4) ไฟล์ binary — ห้าม normalize line ending / diff / merge แบบข้อความ
# --------------------------------------------------------------------------
*.png   binary
*.jpg   binary
*.jpeg  binary
*.gif   binary
*.ico   binary
*.webp  binary
*.pdf   binary
*.zip   binary
*.gz    binary
*.tar   binary
*.7z    binary
*.exe   binary
*.dll   binary
*.so    binary
*.dylib binary
*.woff  binary
*.woff2 binary
*.ttf   binary
*.eot   binary

# --------------------------------------------------------------------------
# 5) ไฟล์เอกสาร Office — diff ผ่าน textconv (ต้องตั้ง git config เพิ่มเอง)
# --------------------------------------------------------------------------
*.docx  diff=word
*.pdf   diff=pdf

# --------------------------------------------------------------------------
# 6) Lock files / generated files — merge เอาฝั่งเราเสมอ ไม่นับเป็นภาษาโค้ด
# --------------------------------------------------------------------------
package-lock.json  merge=ours linguist-generated=true
yarn.lock           merge=ours linguist-generated=true
pnpm-lock.yaml       merge=ours linguist-generated=true
Cargo.lock           merge=ours linguist-generated=true

# --------------------------------------------------------------------------
# 7) CHANGELOG — รวมบรรทัดจากทั้งสองฝั่งแทนการ conflict
# --------------------------------------------------------------------------
CHANGELOG.md  merge=union

# --------------------------------------------------------------------------
# 8) Vendor / dependency ของบุคคลภายนอก — ไม่นับสัดส่วนภาษา
# --------------------------------------------------------------------------
vendor/**        linguist-vendored=true
third_party/**   linguist-vendored=true

# --------------------------------------------------------------------------
# 9) โฟลเดอร์ build/dist ที่ generate อัตโนมัติ — ไม่นับสัดส่วนภาษา
# --------------------------------------------------------------------------
dist/**  linguist-generated=true
build/** linguist-generated=true

# --------------------------------------------------------------------------
# 10) export-ignore — ไฟล์เหล่านี้ track ปกติ แต่ไม่ต้องอยู่ใน git archive
# --------------------------------------------------------------------------
.github/            export-ignore
.gitattributes      export-ignore
.gitignore          export-ignore
.editorconfig       export-ignore
tests/              export-ignore
docs/internal/      export-ignore
CONTRIBUTING.md     export-ignore

# --------------------------------------------------------------------------
# 11) export-subst — แทนที่ placeholder ด้วยข้อมูล commit จริงตอน archive
# --------------------------------------------------------------------------
VERSION.txt export-subst

# --------------------------------------------------------------------------
# 12) Git LFS — ไฟล์ asset ขนาดใหญ่ (เจาะลึกเต็ม ๆ ใน Part 62)
# --------------------------------------------------------------------------
*.psd  filter=lfs diff=lfs merge=lfs -text
*.mp4  filter=lfs diff=lfs merge=lfs -text
*.zip  filter=lfs diff=lfs merge=lfs -text
```

### หมายเหตุประกอบไฟล์ตัวอย่างข้างต้น

- **ส่วนที่ 4 กับส่วนที่ 12 มี `*.zip` ซ้ำกัน** — ในสถานการณ์จริงต้องเลือกอย่างใดอย่างหนึ่ง: ถ้าโปรเจกต์ใช้ Git LFS ให้ตัด `*.zip binary` ออกจากส่วนที่ 4 เพราะบรรทัดที่อยู่ล่างสุด (ส่วนที่ 12) จะ override อยู่ดี แต่การเขียนซ้อนแบบนี้ทำให้สับสนโดยไม่จำเป็น ควรเลือกให้ชัดเจนตั้งแต่แรกว่าไฟล์ประเภทไหนใช้ LFS
- **`diff=word` และ `diff=pdf`** ต้องมีการตั้งค่า `git config diff.word.textconv` และ `git config diff.pdf.textconv` เพิ่มเติมในเครื่องของแต่ละคน (แนะนำให้เขียนเป็น setup script หรือ README แยกต่างหาก เพราะ config นี้ commit เข้า repo ไม่ได้)
- **`filter=lfs`** ต้องติดตั้งและรัน `git lfs install` ก่อนใช้งานได้จริง (รายละเอียดเต็มใน Part 62)

### Checklist สำหรับสร้าง `.gitattributes` ในโปรเจกต์ใหม่

1. เริ่มจาก `* text=auto` เป็นบรรทัดแรกเสมอ
2. list นามสกุลไฟล์ script ที่ line ending สำคัญ (`.sh`, `.bat`, `.ps1`) แล้วบังคับ `eol` ให้ชัดเจน
3. list นามสกุลไฟล์ binary ทั้งหมดที่โปรเจกต์มี แล้วตั้ง `binary`
4. ตรวจสอบว่ามีไฟล์ lock ของ package manager ไหม ถ้ามีให้พิจารณา `merge=ours` หรือปล่อยเป็น default (บาง team อยากเห็น conflict ของ lock file เพื่อ review ว่ามีอะไรเปลี่ยนบ้าง — เป็นเรื่องที่ต้องตกลงกันในทีมก่อน ไม่มีคำตอบที่ถูกเสมอไป)
5. ถ้า publish repository เป็น public บน GitHub ให้ตั้ง `linguist-*` ให้ตัวเลขสัดส่วนภาษาสะท้อนความจริง
6. ถ้ามีการทำ release ผ่าน `git archive` หรือ GitHub Releases ให้ตั้ง `export-ignore` กับไฟล์ที่ไม่ควรอยู่ใน archive
7. ถ้าโปรเจกต์มีไฟล์ asset ขนาดใหญ่ (รูปภาพความละเอียดสูง, วิดีโอ, dataset) ให้เตรียมใช้ `filter=lfs` (ศึกษาต่อใน Part 62)
8. รัน `git check-attr --all -- <ไฟล์>` ตรวจสอบทุกกลุ่มไฟล์หลักหลังตั้งค่าเสร็จ
9. ถ้าเพิ่ม `.gitattributes` เข้าโปรเจกต์ที่มีอยู่แล้ว ให้รัน `git add --renormalize .` เพื่อให้ไฟล์เก่าถูก normalize ตามกฎใหม่

---

## Step 610: แบบฝึกหัด — เขียน `.gitattributes` ที่ควบคุม line ending, diff (สำหรับไฟล์ .png), และ linguist ให้โปรเจกต์จำลอง

ถึงเวลาลงมือปฏิบัติจริง มาสร้างโปรเจกต์จำลองแล้วเขียน `.gitattributes` ตามโจทย์ที่กำหนด

### 10.1 เตรียมโปรเจกต์จำลอง

```bash
mkdir ~/git-course/part-61-gitattributes
cd ~/git-course/part-61-gitattributes
git init
```

สร้างโครงสร้างไฟล์จำลองดังนี้:

```bash
mkdir -p src vendor/jquery dist docs
touch src/app.js src/deploy.sh src/setup.bat
touch vendor/jquery/jquery.min.js
touch dist/bundle.min.js
touch docs/guide.md
touch assets-logo.png   # จำลองไฟล์ binary (จริง ๆ ควรเป็นไฟล์ .png จริง)
touch package-lock.json
touch CHANGELOG.md
```

*(หมายเหตุ: ในการฝึกจริงแนะนำให้ใช้ไฟล์ `.png` จริงสักไฟล์แทนไฟล์เปล่า เพื่อให้เห็นพฤติกรรม binary diff ชัดเจนขึ้นตอนทดสอบ)*

### 10.2 โจทย์ที่ต้องทำ

เขียนไฟล์ `.gitattributes` ที่ root ของโปรเจกต์นี้ ให้ครอบคลุมเงื่อนไขต่อไปนี้ทั้งหมด:

1. ทุกไฟล์ที่ Git ตรวจจับว่าเป็น text ให้ normalize line ending เป็น LF โดยอัตโนมัติ (ใช้ `text=auto`)
2. ไฟล์ `.sh` ต้องบังคับเป็น LF เสมอ ไม่ว่าจะ commit จากเครื่องไหน
3. ไฟล์ `.bat` ต้องบังคับเป็น CRLF เสมอ
4. ไฟล์ `.png` ต้องไม่ถูก diff แบบข้อความ (ให้ขึ้น "Binary files differ" เท่านั้น) และห้ามแตะ line ending
5. ไฟล์ `package-lock.json` ต้อง merge โดยใช้เวอร์ชันของฝั่งเราเสมอ (`merge=ours`)
6. ไฟล์ `CHANGELOG.md` ต้อง merge แบบรวมทุกบรรทัดจากทั้งสองฝั่ง (`merge=union`)
7. โฟลเดอร์ `vendor/` ต้องไม่ถูกนับเป็นสัดส่วนภาษาโปรแกรมมิ่งบน GitHub (`linguist-vendored`)
8. โฟลเดอร์ `dist/` ต้องถือว่าเป็นไฟล์ที่ generate อัตโนมัติ ไม่นับสัดส่วนภาษา (`linguist-generated`)

### 10.3 เฉลยที่แนะนำ

```gitattributes
# .gitattributes ของโปรเจกต์จำลอง part-61-gitattributes

# 1) ค่าเริ่มต้น: normalize line ending อัตโนมัติสำหรับไฟล์ text ทุกไฟล์
* text=auto

# 2) shell script ต้องเป็น LF เสมอ
*.sh text eol=lf

# 3) windows batch script ต้องเป็น CRLF เสมอ
*.bat text eol=crlf

# 4) ไฟล์ png เป็น binary — ไม่ diff แบบข้อความ ไม่แตะ line ending ไม่ merge แบบข้อความ
*.png binary

# 5) lock file — merge เอาฝั่งเราเสมอ
package-lock.json merge=ours

# 6) changelog — รวมบรรทัดจากทั้งสองฝั่งแทนการ conflict
CHANGELOG.md merge=union

# 7) vendor — ไม่นับสัดส่วนภาษา
vendor/** linguist-vendored=true

# 8) dist — ไฟล์ generate อัตโนมัติ ไม่นับสัดส่วนภาษา
dist/** linguist-generated=true
```

### 10.4 ทดสอบว่า `.gitattributes` ทำงานถูกต้อง

รันคำสั่งตรวจสอบทีละไฟล์ ดังนี้:

```bash
git check-attr --all -- src/deploy.sh
```

ผลลัพธ์ที่ควรได้:

```
src/deploy.sh: text: set
src/deploy.sh: eol: lf
```

```bash
git check-attr --all -- src/setup.bat
```

ผลลัพธ์ที่ควรได้:

```
src/setup.bat: text: set
src/setup.bat: eol: crlf
```

```bash
git check-attr --all -- assets-logo.png
```

ผลลัพธ์ที่ควรได้ (เพราะ `binary` เทียบเท่า `-text -diff -merge`):

```
assets-logo.png: text: unset
assets-logo.png: diff: unset
assets-logo.png: merge: unset
```

```bash
git check-attr --all -- package-lock.json
```

ผลลัพธ์ที่ควรได้:

```
package-lock.json: merge: ours
```

```bash
git check-attr --all -- vendor/jquery/jquery.min.js
```

ผลลัพธ์ที่ควรได้:

```
vendor/jquery/jquery.min.js: linguist-vendored: set
```

ถ้าผลลัพธ์ตรงตามที่คาดทุกไฟล์ แปลว่า `.gitattributes` ที่เขียนทำงานถูกต้องตามโจทย์ทั้งหมดแล้ว

### 10.5 ทดสอบพฤติกรรม diff กับไฟล์ binary

ลอง commit ไฟล์ครั้งแรกแล้วแก้ไขไฟล์ `.png` (แม้จะเป็นไฟล์เปล่าในตัวอย่างนี้ ก็ลองเปลี่ยนเนื้อหาสักเล็กน้อยด้วยการเขียนตัวอักษรสุ่มเข้าไปแทน เพื่อจำลองว่าไฟล์เปลี่ยน):

```bash
git add .
git commit -m "Initial commit พร้อม .gitattributes"
echo "fake-binary-change" >> assets-logo.png
git diff
```

ผลลัพธ์ที่ควรเห็นสำหรับ `assets-logo.png`:

```
Binary files a/assets-logo.png and b/assets-logo.png differ
```

เทียบกับไฟล์ `src/app.js` ที่ยังเป็น text diff ปกติ ถ้าลองแก้ไขและ `git diff` ดู จะเห็นผลต่างแบบบรรทัดต่อบรรทัดตามปกติ — นี่คือการยืนยันว่า attribute `binary` ทำงานถูกต้องแล้ว

### 10.6 แบบฝึกหัดเพิ่มเติม (ทำเองเพื่อฝึกฝนเพิ่ม)

1. ลองเพิ่มไฟล์ `.jpg`, `.zip` เข้าไปในโปรเจกต์จำลอง แล้วเขียน pattern เพิ่มให้ครอบคลุมด้วย `binary`
2. ลองสร้างไฟล์ `VERSION.txt` ที่มีเนื้อหา `Build: $Format:%H$` แล้วตั้ง `export-subst` จากนั้นรัน `git archive --format=zip -o test.zip HEAD` แล้วแตกไฟล์ zip ดูว่า placeholder ถูกแทนที่ด้วยค่า commit hash จริงหรือไม่
3. ลองสร้างสถานการณ์ conflict จริงบน `package-lock.json` โดยแก้ไฟล์นี้ในสอง branch แล้ว merge เข้าด้วยกัน สังเกตว่า Git ไม่ขึ้น conflict เลยและเลือกเวอร์ชันของฝั่งปัจจุบันเสมอตามที่ `merge=ours` กำหนดไว้
4. ลองใช้ `git add --renormalize .` หลังจากเพิ่ม `.gitattributes` เข้าไปในโปรเจกต์ที่มี commit อยู่ก่อนแล้ว สังเกตว่ามีไฟล์ไหนถูก stage เป็นการเปลี่ยนแปลง line ending บ้าง

---

## สรุป Part 61

ใน Part นี้เราได้เรียนรู้ว่า:

1. `.gitattributes` คือไฟล์ configuration ที่กำหนดพฤติกรรมของ Git ต่อไฟล์แต่ละแบบ/แต่ละ pattern แตกต่างกัน แก้ปัญหาที่ `.gitignore` และ `core.autocrlf` ระดับ global แก้ไม่ได้ละเอียดพอ
2. `text` และ `eol` ควบคุม line ending ได้ละเอียดถึงระดับไฟล์หรือ pattern เดียว แก้ปัญหา shebang line พังเพราะ CRLF ได้อย่างถาวร
3. `diff` attribute ควบคุมวิธี diff ไฟล์พิเศษ ตั้งแต่ปิด diff แบบข้อความ (`binary`) ไปจนถึงใช้ `textconv` แปลงไฟล์ Word/PDF เป็นข้อความก่อน diff
4. `merge` attribute เปิดให้กำหนด merge strategy เฉพาะไฟล์ เช่น `merge=ours` สำหรับไฟล์ที่ไม่อยากให้ conflict และ `merge=union` สำหรับไฟล์ประเภทรายการอย่าง CHANGELOG
5. `export-ignore` และ `export-subst` ควบคุมพฤติกรรมของ `git archive` โดยไม่กระทบการ track ไฟล์ปกติเลย
6. `linguist-*` attributes ควบคุมการนับสัดส่วนภาษาโปรแกรมมิ่งบนหน้า GitHub repository ให้สะท้อนความเป็นจริงของโปรเจกต์
7. `filter`, `clean`, `smudge` คือกลไกทั่วไปที่ Git LFS นำไปใช้เป็นแกนหลัก — เข้าใจกลไกนี้ก่อนจะช่วยให้เข้าใจ Git LFS ใน Part ถัดไปได้ลึกซึ้งขึ้นมาก
8. `.gitattributes` กับ `.gitignore` ทำหน้าที่ตรงข้ามกัน: ไฟล์หนึ่งบอกว่า "ไม่ต้อง track" อีกไฟล์บอกว่า "track อยู่แล้ว แต่ปฏิบัติแบบนี้"
9. เราได้เขียนไฟล์ `.gitattributes` มาตรฐานฉบับเต็มที่ใช้งานได้จริงในโปรเจกต์ cross-platform พร้อมทำแบบฝึกหัดลงมือจริงจนครบทุกเงื่อนไข

**ต่อไป:** [Part 62: Git LFS: จัดการไฟล์ขนาดใหญ่](./part-062-git-lfs.md)
