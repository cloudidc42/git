# Part 41: การแก้ Merge Conflict ขั้นสูงและเครื่องมือช่วย

> **Step ในหลักสูตรนี้:** Step 401–410
> **เฟส:** 4 — ทำงานเป็นทีมด้วย Workflow มาตรฐาน (Git Flow, GitHub Flow, Rebase)
> **เป้าหมายของ Part นี้:** ต่อยอดจากความรู้เรื่อง conflict เบื้องต้นใน Part 08 ไปสู่การรับมือกับ conflict ที่ซับซ้อนแบบที่เจอในทีมจริง ทั้ง rename/rename, delete/modify, add/add, การตั้งค่า `diff3`/`zdiff3` เพื่อดู context เพิ่ม, การใช้ `git rerere` ให้ Git จำวิธีแก้ conflict แทนเรา, การต่อ mergetool ภายนอก, กลยุทธ์ลด conflict ตั้งแต่ต้นทาง, การแก้ conflict ในไฟล์พิเศษอย่าง binary และ lock file, การจัดการ conflict ระหว่าง rebase ยาว ๆ หลาย commit, และการรู้จัก abort อย่างปลอดภัยเมื่อสถานการณ์เกินจะแก้ไหว

---

## สารบัญของ Part นี้

- Step 401: ทบทวน conflict เบื้องต้นจาก Part 08 แล้วต่อยอดสู่ระดับที่ซับซ้อนขึ้น
- Step 402: Conflict ประเภทซับซ้อน (rename/rename, delete/modify, add/add)
- Step 403: `git config merge.conflictstyle diff3` / `zdiff3` — เพิ่ม context จาก common ancestor
- Step 404: `git rerere` — ให้ Git จดจำวิธีแก้ conflict แล้วใช้ซ้ำอัตโนมัติ
- Step 405: เครื่องมือ mergetool ภายนอก (VS Code, Meld, Beyond Compare)
- Step 406: กลยุทธ์ลด conflict ตั้งแต่ต้น
- Step 407: การแก้ conflict ในไฟล์พิเศษ (binary, lock file)
- Step 408: Conflict ตอน rebase ยาว ๆ หลาย commit ทำอย่างไรให้ไม่งง
- Step 409: การ abort อย่างปลอดภัยเมื่อ conflict ซับซ้อนเกินจะแก้ไหว
- Step 410: แบบฝึกหัด — จำลองสถานการณ์ทีมจริง แก้ conflict หลายไฟล์พร้อมกัน (รวม rename conflict)

---

## Step 401: ทบทวน conflict เบื้องต้นจาก Part 08 แล้วต่อยอดสู่ระดับที่ซับซ้อนขึ้น

ใน **Part 08** เราเรียนรู้พื้นฐานของ merge conflict ไปแล้วว่า:

1. Conflict เกิดขึ้นเมื่อ Git ไม่สามารถรวมการเปลี่ยนแปลงจากสอง branch เข้าด้วยกันโดยอัตโนมัติได้ เพราะทั้งสองฝั่งแก้ **บรรทัดเดียวกันหรือบริเวณใกล้กันมากเกินไป** ในไฟล์เดียวกัน
2. Git จะแทรก **conflict marker** (`<<<<<<<`, `=======`, `>>>>>>>`) ลงในไฟล์ตรงจุดที่ชนกัน แล้วหยุดรอให้มนุษย์ตัดสินใจ
3. ขั้นตอนแก้คือ เปิดไฟล์ → เลือก/รวมเนื้อหาที่ถูกต้อง → ลบ marker ออก → `git add` → `git commit` (หรือ `git merge --continue`)
4. ถ้าแก้ไม่ไหวจริง ๆ สามารถ `git merge --abort` เพื่อย้อนกลับไปสถานะก่อน merge ได้ทันที

นั่นคือ conflict แบบ **"content conflict"** ธรรมดา — สองฝั่งแก้เนื้อหาในไฟล์เดียวกันตรงบรรทัดที่ทับซ้อนกัน ซึ่งเป็นกรณีที่พบบ่อยที่สุดและเข้าใจง่ายที่สุด

แต่ในการทำงานจริงกับทีมขนาดใหญ่ ที่มีคนหลายสิบคนแก้ไฟล์เดียวกันตลอดเวลา คุณจะเจอ conflict ที่ซับซ้อนกว่านั้นมาก ตัวอย่างเช่น:

- ไฟล์ถูก **rename** โดยคนสองฝั่งพร้อมกัน แต่ตั้งชื่อใหม่ไม่เหมือนกัน
- คนหนึ่ง **ลบไฟล์ทิ้ง** ในขณะที่อีกคน **แก้ไขไฟล์นั้น** อยู่บน branch คู่ขนาน
- สองคน **สร้างไฟล์ใหม่ที่ชื่อเดียวกัน** โดยไม่รู้จักกันมาก่อน
- Conflict เกิดขึ้นระหว่าง `git rebase` ที่มีหลายสิบ commit ต่อเนื่องกัน ทำให้ต้องแก้ conflict ซ้ำ ๆ หลายรอบ
- Conflict เกิดในไฟล์ที่เป็น binary หรือไฟล์ที่ auto-generate เช่น `package-lock.json`

Part นี้จะพาคุณผ่านทุกสถานการณ์เหล่านี้อย่างละเอียด พร้อมเครื่องมือและ workflow ที่มืออาชีพใช้จริงในการจัดการ conflict ระดับซับซ้อน ก่อนจะเข้าสู่เรื่อง `git bisect` ใน Part 42

### ทำไมต้องเรียนเรื่องนี้ให้ลึก

หลายคนคิดว่า "แค่แก้ conflict เป็นก็พอ" แต่ความจริงคือ:

- ทีมที่ merge บ่อย ๆ จะเจอ conflict ซ้ำ ๆ แบบเดิมทุกครั้งที่ rebase branch ยาว ๆ — ถ้าไม่รู้จัก `git rerere` จะเสียเวลาแก้ซ้ำโดยไม่จำเป็น
- Conflict บางแบบ (เช่น rename/rename หรือ delete/modify) ถ้าไม่เข้าใจกลไกเบื้องหลัง จะแก้ผิดจนทำให้โค้ดเสียหายโดยไม่รู้ตัว (เช่น เผลอลบไฟล์ที่จริง ๆ ควรเก็บไว้)
- การไม่มีกลยุทธ์ป้องกัน conflict ตั้งแต่ต้น จะทำให้ทีมเสียเวลามหาศาลไปกับการแก้ conflict แทนที่จะโฟกัสกับงานจริง

มาเริ่มกันที่ประเภทของ conflict ที่ซับซ้อนกว่า content conflict ธรรมดา

---

## Step 402: Conflict ประเภทซับซ้อน (rename/rename, delete/modify, add/add)

Git ตรวจจับ conflict ได้หลายรูปแบบ ไม่ใช่แค่การชนกันของเนื้อหาในบรรทัดเดียวกัน ต่อไปนี้คือประเภทที่พบบ่อยที่สุดในทีมจริง

### 402.1 Rename/Rename Conflict

เกิดขึ้นเมื่อ **ทั้งสอง branch เปลี่ยนชื่อไฟล์เดียวกัน แต่ตั้งชื่อใหม่ไม่ตรงกัน**

ตัวอย่างสถานการณ์:

```bash
# branch main: เปลี่ยนชื่อ utils.js -> helpers.js
git checkout main
git mv utils.js helpers.js
git commit -am "rename utils.js to helpers.js"

# branch feature: เปลี่ยนชื่อ utils.js -> util-functions.js (ทำคู่ขนานกัน ไม่รู้เรื่องกัน)
git checkout feature
git mv utils.js util-functions.js
git commit -am "rename utils.js to util-functions.js"
```

เมื่อลอง merge:

```bash
git checkout main
git merge feature
```

ผลลัพธ์:

```
CONFLICT (rename/rename): Rename "utils.js"->"helpers.js" in branch "main" rename "utils.js"->"util-functions.js" in "feature"
```

Git มองเห็นว่าไฟล์ต้นฉบับ `utils.js` หายไปทั้งสองฝั่ง แต่ **ไม่รู้ว่าคุณต้องการชื่อไหน** ระบบจะสร้างไฟล์ทั้งสองชื่อไว้ในโฟลเดอร์ (บางกรณี Git จะพยายาม merge เนื้อหาเข้าไปในทั้งสองไฟล์ให้ด้วย) แล้วให้คุณเลือกเอง:

```bash
git status
```

```
both renamed:       utils.js -> helpers.js
both renamed:       utils.js -> util-functions.js
```

**วิธีแก้:** ตัดสินใจว่าจะใช้ชื่อไหนเป็นชื่อสุดท้าย แล้วลบไฟล์ที่ไม่ต้องการทิ้ง เช่น ถ้าทีมตกลงใช้ `helpers.js`:

```bash
git rm util-functions.js
git add helpers.js
git commit
```

**ข้อควรระวัง:** ถ้าทั้งสองฝั่งแก้เนื้อหาในไฟล์ด้วย ไม่ใช่แค่ rename เฉย ๆ คุณต้องเปิดไฟล์ที่เลือกไว้ตรวจดูว่ามี conflict marker `<<<<<<<` หลงเหลืออยู่ในเนื้อหาหรือไม่ เพราะ Git มักจะพยายาม merge เนื้อหาให้อัตโนมัติในไฟล์ที่ถูกเลือกเป็นปลายทางด้วย

### 402.2 Delete/Modify Conflict

เกิดขึ้นเมื่อ **branch หนึ่งลบไฟล์ทิ้ง ในขณะที่อีก branch แก้ไขเนื้อหาไฟล์นั้น**

```bash
# branch main: ลบไฟล์ old-config.json ทิ้งเพราะเลิกใช้แล้ว
git checkout main
git rm old-config.json
git commit -m "remove deprecated config file"

# branch feature: แก้ไขไฟล์ old-config.json (ไม่รู้ว่า main ลบไปแล้ว)
git checkout feature
# ... แก้ไข old-config.json ...
git commit -am "update timeout value in old-config.json"
```

Merge:

```bash
git checkout main
git merge feature
```

ผลลัพธ์:

```
CONFLICT (modify/delete): old-config.json deleted in HEAD and modified in feature. Version feature of old-config.json left in tree.
```

Git จะ **คงไฟล์เวอร์ชันที่ถูกแก้ไขไว้ในโฟลเดอร์การทำงาน** เพื่อให้คุณตัดสินใจ เพราะ Git ไม่รู้ว่าการลบนั้น "ตั้งใจ" หรือคนที่แก้ไม่รู้ว่าไฟล์ถูกลบไปแล้ว

**ตรวจสอบสถานะ:**

```bash
git status
```

```
Unmerged paths:
  deleted by us:      old-config.json
```

**ทางเลือกในการแก้:**

1. **ถ้าตัดสินใจว่าควรลบไฟล์นี้จริง ๆ** (การแก้ไขใน feature ไม่มีความหมายอีกต่อไป):

   ```bash
   git rm old-config.json
   git commit
   ```

2. **ถ้าตัดสินใจว่าไฟล์ยังต้องใช้อยู่** (การลบใน main เป็นความผิดพลาด หรือการแก้ไขใน feature ยังจำเป็น):

   ```bash
   git add old-config.json
   git commit
   ```

**สิ่งสำคัญ:** อย่าเลือกทางใดทางหนึ่งโดยไม่ตรวจสอบก่อน ควรคุยกับเจ้าของ commit ที่ลบไฟล์ (ดูจาก `git log --diff-filter=D -- old-config.json`) ว่าทำไมถึงลบ ก่อนตัดสินใจ เพราะการเลือกผิดอาจทำให้ฟีเจอร์ที่เพิ่งแก้หายไปเงียบ ๆ หรือทำให้ไฟล์ที่ควรลบกลับมาอยู่ในโปรเจกต์อีกครั้ง

### 402.3 Add/Add Conflict

เกิดขึ้นเมื่อ **สองฝั่งสร้างไฟล์ใหม่ที่ชื่อเดียวกัน** โดยไม่มี common ancestor ของไฟล์นั้นมาก่อน (เพราะไฟล์นี้ไม่เคยมีอยู่ใน merge base เลย)

```bash
# branch main: สร้างไฟล์ constants.js ใหม่
git checkout main
echo "export const MAX_RETRY = 3;" > constants.js
git add constants.js
git commit -m "add constants.js with MAX_RETRY"

# branch feature: สร้างไฟล์ชื่อเดียวกันโดยไม่รู้จักกัน
git checkout feature
echo "export const API_TIMEOUT = 5000;" > constants.js
git add constants.js
git commit -m "add constants.js with API_TIMEOUT"
```

Merge:

```bash
git checkout main
git merge feature
```

ผลลัพธ์:

```
CONFLICT (add/add): Merge conflict in constants.js
```

เปิดไฟล์ดู จะเห็น conflict marker ตามปกติ:

```javascript
<<<<<<< HEAD
export const MAX_RETRY = 3;
=======
export const API_TIMEOUT = 5000;
>>>>>>> feature
```

**จุดที่ต่างจาก content conflict ทั่วไป:** เพราะไม่มี common ancestor ของไฟล์นี้ (ไฟล์ไม่เคยมีมาก่อนใน base commit) Git จึงไม่มีทาง "เดา" ได้เลยว่าทั้งสองฝั่งตั้งใจให้ไฟล์นี้มีเนื้อหาอะไรร่วมกัน จึงถือว่าทั้งไฟล์ conflict กันทั้งหมด

**วิธีแก้:** ในกรณีนี้ส่วนใหญ่แค่รวมทั้งสองฝั่งเข้าด้วยกัน (เพราะทั้งสองค่าคงที่ไม่ได้ขัดกันจริง ๆ):

```javascript
export const MAX_RETRY = 3;
export const API_TIMEOUT = 5000;
```

```bash
git add constants.js
git commit
```

### ตารางสรุปประเภท conflict ขั้นสูง

| ประเภท | เกิดเมื่อ | คำสั่งตรวจสอบ | จุดเสี่ยง |
|---|---|---|---|
| Content conflict (พื้นฐาน) | สองฝั่งแก้บรรทัดเดียวกัน | `git status` → `both modified` | เลือกเนื้อหาผิด |
| Rename/Rename | สองฝั่ง rename ไฟล์เดียวกันคนละชื่อ | `both renamed` | เผลอเก็บไฟล์ซ้ำสองชื่อ |
| Delete/Modify | ฝั่งหนึ่งลบ อีกฝั่งแก้ไข | `deleted by us` / `deleted by them` | เลือกลบ/เก็บผิดโดยไม่เช็คเหตุผล |
| Add/Add | สองฝั่งสร้างไฟล์ชื่อเดียวกันใหม่ | `both added` | ไม่มี base ให้เทียบ ต้องตัดสินใจเองทั้งหมด |

---

## Step 403: `git config merge.conflictstyle diff3` / `zdiff3` — เพิ่ม context จาก common ancestor

Conflict marker แบบมาตรฐาน (`merge` style) ที่เราเห็นใน Part 08 แสดงแค่ **สองฝั่ง** ที่ชนกัน:

```
<<<<<<< HEAD
โค้ดฝั่งเรา
=======
โค้ดฝั่งเขา
>>>>>>> feature
```

ปัญหาคือ เมื่อเห็นแค่สองฝั่งนี้ บางครั้งเราตอบไม่ได้ว่า **"เดิมทีก่อนที่ทั้งสองฝั่งจะแก้ มันเป็นแบบไหน"** ซึ่งข้อมูลนี้สำคัญมากในการตัดสินใจว่าจะรวมยังไงให้ถูกต้อง

Git มีทางเลือกให้แสดง **เนื้อหาต้นฉบับจาก common ancestor (merge base)** เพิ่มเข้ามาด้วย เรียกว่า **diff3 style**

### 403.1 เปิดใช้งาน diff3

```bash
git config --global merge.conflictstyle diff3
```

หรือเปิดใช้แค่ repo เดียว (ไม่ใส่ `--global`):

```bash
git config merge.conflictstyle diff3
```

หลังจากตั้งค่านี้แล้ว conflict marker จะมีส่วนที่สามเพิ่มเข้ามาคือ `|||||||` ซึ่งแสดงเนื้อหาจาก common ancestor:

```
<<<<<<< HEAD
const timeout = 5000;
||||||| merged common ancestors
const timeout = 3000;
=======
const timeout = 10000;
>>>>>>> feature
```

จากตัวอย่างนี้ เราจะเห็นได้ทันทีว่า:

- เดิมที (common ancestor) `timeout = 3000`
- ฝั่งเรา (HEAD) เปลี่ยนเป็น `5000`
- ฝั่งเขา (feature) เปลี่ยนเป็น `10000`

ข้อมูลนี้ช่วยให้เราเข้าใจ **เจตนา** ของการเปลี่ยนแปลงแต่ละฝั่งได้ชัดเจนกว่ามาก แทนที่จะเห็นแค่ผลลัพธ์สุดท้ายสองอัน เราจะรู้ว่าทั้งสองฝั่งกำลัง "เพิ่มค่า timeout" ในทิศทางเดียวกัน เพียงแต่เพิ่มไม่เท่ากัน ซึ่งอาจแปลว่าควรใช้ค่าที่มากกว่า หรือควรคุยกับเจ้าของ code ทั้งสองฝั่งว่าค่าที่เหมาะสมคือเท่าไหร่

### 403.2 zdiff3 — เวอร์ชันปรับปรุงที่ดีกว่า diff3

Git ตั้งแต่เวอร์ชัน **2.35** ขึ้นไป มี style ใหม่ชื่อ **zdiff3** ที่ปรับปรุงจาก diff3 ให้อ่านง่ายขึ้น โดย **ตัดบรรทัดที่เหมือนกันทั้งสองฝั่งออกจาก conflict block** ทำให้ conflict block สั้นลงและโฟกัสเฉพาะจุดที่ต่างกันจริง ๆ

```bash
git config --global merge.conflictstyle zdiff3
```

เปรียบเทียบกับ diff3 แบบเดิม สมมติมีฟังก์ชันที่มีหลายบรรทัด แต่มีแค่บรรทัดเดียวที่ต่างกันจริง ๆ:

**diff3 (แบบเดิม)** จะแสดงทั้งฟังก์ชันซ้ำ 3 รอบเพราะมันเทียบทั้ง block:

```
<<<<<<< HEAD
function connect() {
  const timeout = 5000;
  return createConnection(timeout);
}
||||||| merged common ancestors
function connect() {
  const timeout = 3000;
  return createConnection(timeout);
}
=======
function connect() {
  const timeout = 10000;
  return createConnection(timeout);
}
>>>>>>> feature
```

**zdiff3** จะฉลาดพอที่จะดึงเฉพาะบรรทัดที่ต่างออกมา:

```
function connect() {
<<<<<<< HEAD
  const timeout = 5000;
||||||| merged common ancestors
  const timeout = 3000;
=======
  const timeout = 10000;
>>>>>>> feature
  return createConnection(timeout);
}
```

จะเห็นว่า `zdiff3` อ่านง่ายกว่ามาก เพราะตัดส่วนที่ไม่เกี่ยวข้อง (`function connect() {` และ `return createConnection(timeout);`) ออกจาก conflict block ทำให้เห็นเฉพาะจุดที่ต้องตัดสินใจจริง ๆ

### 403.3 ตรวจสอบค่าปัจจุบันและปิดกลับไปใช้ style เดิม

```bash
# ดูค่าปัจจุบัน
git config --get merge.conflictstyle

# กลับไปใช้ style มาตรฐาน (ไม่มี common ancestor)
git config --global merge.conflictstyle merge
```

### 403.4 คำแนะนำการใช้งานจริง

- แนะนำให้ตั้ง `zdiff3` เป็นค่า default ในเครื่องของทุกคนในทีม เพราะแทบไม่มีข้อเสีย มีแต่ข้อมูลเพิ่มขึ้นให้ตัดสินใจง่ายขึ้น
- ถ้าทีมใช้ Git เวอร์ชันเก่ากว่า 2.35 ให้ใช้ `diff3` แทน
- การตั้งค่านี้เป็นเรื่องส่วนบุคคล (ตั้งใน `--global` ของแต่ละคน) ไม่ใช่ตั้งค่า repo ที่บังคับทุกคนต้องใช้เหมือนกัน เพราะเป็นแค่ตัวช่วยแสดงผลตอน conflict เท่านั้น ไม่กระทบไฟล์ที่ commit จริง

---

## Step 404: `git rerere` — ให้ Git จดจำวิธีแก้ conflict แล้วใช้ซ้ำอัตโนมัติ

### 404.1 ปัญหาที่ rerere แก้

ลองนึกภาพสถานการณ์นี้: คุณกำลัง `git rebase` branch `feature` ที่มี 20 commits ทับ `main` และทุก ๆ commit มี conflict ที่บรรทัดเดียวกันซ้ำ ๆ (เช่น import statement ที่ทั้งสอง branch แก้ไขบ่อยมาก) คุณจะต้องแก้ conflict เดิม ๆ ซ้ำ 20 รอบ! นี่คือปัญหาที่พบบ่อยมากเวลา rebase branch ยาว ๆ หรือ merge branch เดิมกลับไปกลับมาหลายรอบ

**`git rerere`** ย่อมาจาก **"Reuse Recorded Resolution"** เป็นฟีเจอร์ที่ทำให้ Git **จดจำ** วิธีที่คุณแก้ conflict ไปแล้วครั้งหนึ่ง แล้ว **นำมาใช้ซ้ำอัตโนมัติ** เมื่อเจอ conflict แบบเดิมอีกในอนาคต

### 404.2 เปิดใช้งาน rerere

```bash
git config --global rerere.enabled true
```

หรือเปิดใช้เฉพาะ repo:

```bash
git config rerere.enabled true
```

ควรเปิดค่านี้ไว้เสมอ (แนะนำให้ตั้งเป็น global default) เพราะไม่มีผลเสียอะไรถ้าไม่มี conflict ซ้ำ แต่ถ้ามี conflict ซ้ำจะช่วยประหยัดเวลาได้มหาศาล

### 404.3 rerere ทำงานอย่างไร

เมื่อเปิด rerere แล้ว ทุกครั้งที่ Git เจอ conflict มันจะ:

1. **บันทึก (record)** ลักษณะของ conflict (pre-image) ไว้ใน `.git/rr-cache/`
2. เมื่อคุณแก้ conflict เสร็จและ `git add` ไฟล์นั้น Git จะบันทึก **วิธีที่คุณแก้ (post-image)** ควบคู่ไปด้วย
3. ครั้งต่อไปที่ Git เจอ conflict ที่มีลักษณะ **เหมือนเดิมทุกประการ** (pre-image ตรงกัน) มันจะ **แก้ไขให้อัตโนมัติทันที** โดยใช้ resolution ที่เคยบันทึกไว้ (**reuse**)

### 404.4 ตัวอย่างการใช้งานจริง

สมมติสถานการณ์: คุณกำลัง rebase branch `feature` ที่มี 5 commits ทับ `main` ที่อัปเดตบ่อย และทุก commit มี conflict ที่ไฟล์ `CHANGELOG.md` เพราะทั้งสอง branch เพิ่มบรรทัดใหม่ที่ด้านบนของไฟล์เหมือนกันเป๊ะทุกครั้ง

```bash
git config rerere.enabled true

git checkout feature
git rebase main
```

**Commit ที่ 1 มี conflict:**

```
CONFLICT (content): Merge conflict in CHANGELOG.md
```

คุณเปิดไฟล์แก้ conflict ด้วยมือ:

```bash
# แก้ไข CHANGELOG.md ด้วยมือ
git add CHANGELOG.md
git rebase --continue
```

ที่จุดนี้ rerere จะบันทึกวิธีที่คุณแก้ไว้เรียบร้อยแล้ว ระบบจะแสดงข้อความประมาณนี้ตอน `git add`:

```
Recorded resolution for 'CHANGELOG.md'.
```

**Commit ที่ 2, 3, 4, 5** ถ้าเกิด conflict แบบเดียวกันเป๊ะ (pre-image เหมือนกัน) Git จะขึ้นข้อความว่า:

```
Resolved 'CHANGELOG.md' using previous resolution.
```

และ **แก้ไฟล์ให้อัตโนมัติทันที** คุณแค่ตรวจสอบผลลัพธ์แล้ว `git add` + `git rebase --continue` ต่อได้เลย ไม่ต้องแก้ conflict ซ้ำอีก

### 404.5 คำสั่งเสริมของ rerere

```bash
# ดูสถานะและรายการ conflict ที่กำลังถูก track โดย rerere
git rerere status

# ดู diff ของ resolution ที่บันทึกไว้ (ก่อน/หลัง)
git rerere diff

# ลบ record การจำ resolution ทั้งหมด (เผื่อ resolution เดิมผิด)
git rerere forget <path-to-file>

# บังคับให้ rerere ทำงานตอนนี้เลย (ปกติ Git เรียกให้อัตโนมัติ)
git rerere
```

### 404.6 ข้อควรระวังสำคัญของ rerere

1. **rerere ไม่ใช่เวทมนตร์** — มันจะ apply resolution ซ้ำได้ก็ต่อเมื่อ **pre-image ของ conflict เหมือนเดิมทุกตัวอักษร** ถ้าโค้ดรอบข้างเปลี่ยนไปแม้เพียงเล็กน้อย rerere จะไม่ match และคุณต้องแก้ conflict ใหม่ด้วยมือ (ซึ่งมันจะบันทึกอันใหม่ทับ)

2. **ต้องตรวจสอบผลลัพธ์เสมอ แม้ rerere จะ resolve ให้อัตโนมัติ** — อย่าไว้ใจแบบไม่ดูเลย เพราะถ้าครั้งแรกที่คุณแก้ไปนั้น **แก้ผิด** rerere ก็จะ "แก้ผิดซ้ำ" ให้อัตโนมัติทุกครั้งเช่นกัน! ให้ใช้ `git diff --staged` ตรวจดูทุกครั้งก่อน commit

3. **rerere เก็บข้อมูลไว้ใน `.git/rr-cache/`** ซึ่งเป็น local เท่านั้น ไม่ถูก push ไปที่ remote ไม่แชร์ระหว่างเครื่อง ถ้าอยากให้ทีมได้ประโยชน์จาก resolution ที่เคยบันทึกไว้ ต้อง sync โฟลเดอร์นี้เอง (ไม่แนะนำ) หรือให้แต่ละคนสร้าง record ของตัวเอง

4. **ล้าง record เก่าเป็นระยะ** — ถ้า resolution เก่าไม่ตรงกับ pattern ปัจจุบันอีกแล้ว มันจะกลายเป็นขยะที่ไม่มีวันถูกใช้ สามารถล้างด้วย:

   ```bash
   git rerere gc
   ```

   ซึ่งจะลบ record ที่ไม่ได้ใช้งานเกินระยะเวลาที่กำหนดออกไปตามค่า `gc.rerereResolved` และ `gc.rerereUnresolved`

### 404.7 กรณีใช้งานที่ rerere เหมาะที่สุด

- Rebase branch ยาว ๆ ที่มี conflict pattern เดิมซ้ำหลาย commit
- การ merge branch สองอันที่ต้อง merge กลับไปกลับมาบ่อย ๆ (เช่น long-lived branch ที่ sync กับ main ทุกวัน)
- การ cherry-pick หลาย commit ที่มักชนกับจุดเดิมซ้ำ ๆ
- ทีมที่ทำ **branch แปล (translation) หรือ branch config เฉพาะ environment** ที่ต้อง rebase ทับ main บ่อยมากและมักชนที่จุดเดิมเสมอ

---

## Step 405: เครื่องมือ mergetool ภายนอก (VS Code, Meld, Beyond Compare)

การแก้ conflict ด้วยการเปิด text editor ธรรมดาแล้วไล่หา `<<<<<<<` เอง ใช้ได้กับ conflict เล็ก ๆ แต่เมื่อไฟล์ใหญ่หรือมีหลาย conflict block พร้อมกัน การใช้ **mergetool แบบ visual** (3-way merge view) จะช่วยได้มาก

### 405.1 แนวคิดของ 3-way merge view

Mergetool ที่ดีจะแสดงหน้าจอแบ่งเป็น 3-4 ช่อง:

```
┌─────────────┬─────────────┬─────────────┐
│   LOCAL     │    BASE     │   REMOTE    │
│  (ฝั่งเรา)   │ (ต้นฉบับร่วม) │  (ฝั่งเขา)   │
├─────────────┴─────────────┴─────────────┤
│              MERGED (ผลลัพธ์)             │
│         (แก้ไขตรงนี้แล้วบันทึก)            │
└───────────────────────────────────────────┘
```

คุณสามารถคลิกเลือกฝั่งที่ต้องการ (accept theirs/mine) หรือแก้ไขในช่อง MERGED ได้โดยตรง เห็นภาพชัดเจนกว่าดู marker ในไฟล์ดิบมาก

### 405.2 ตั้งค่า VS Code เป็น mergetool

VS Code มี merge editor ในตัวที่ดีมาก ตั้งค่าได้ดังนี้:

```bash
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
```

จากนั้นเมื่อเกิด conflict ให้รัน:

```bash
git mergetool
```

Git จะเปิด VS Code ขึ้นมาโดยอัตโนมัติที่ไฟล์ conflict พร้อม UI แบบ **3-way merge editor** ที่มีปุ่ม "Accept Current Change", "Accept Incoming Change", "Accept Both Changes" ให้เลือกในแต่ละ conflict block เมื่อแก้เสร็จและปิดไฟล์ (save + close) Git จะถามยืนยันว่าแก้เสร็จหรือยัง

### 405.3 ตั้งค่า Meld (นิยมบน Linux, ฟรีและ cross-platform)

**Meld** เป็นเครื่องมือ diff/merge แบบ open source ที่ได้รับความนิยมมากบน Linux (มีเวอร์ชัน macOS/Windows ด้วย)

ติดตั้ง (ตัวอย่าง Ubuntu):

```bash
sudo apt install meld
```

ตั้งค่า:

```bash
git config --global merge.tool meld
git config --global mergetool.meld.path /usr/bin/meld
```

เรียกใช้:

```bash
git mergetool
```

Meld จะเปิดหน้าต่างแสดง 3 คอลัมน์ (LOCAL / MERGED / REMOTE) ให้ลากเลือกเนื้อหาไปมาได้ด้วยปุ่มลูกศรระหว่างคอลัมน์

### 405.4 ตั้งค่า Beyond Compare (เครื่องมือเชิงพาณิชย์ยอดนิยมในองค์กร)

**Beyond Compare** เป็นเครื่องมือแบบเสียเงินที่นิยมมากในองค์กรขนาดใหญ่ เพราะรองรับทั้งไฟล์ text, folder comparison และ binary comparison ได้ดีมาก

ตั้งค่า (ตัวอย่างบน Linux):

```bash
git config --global merge.tool bc
git config --global mergetool.bc.path /usr/bin/bcompare
```

ตั้งค่าบน macOS:

```bash
git config --global merge.tool bc
git config --global mergetool.bc.path "/Applications/Beyond Compare.app/Contents/MacOS/bcomp"
```

ตั้งค่าบน Windows (ใน Git Bash):

```bash
git config --global merge.tool bc
git config --global mergetool.bc.path "C:/Program Files/Beyond Compare 4/BCompare.exe"
```

### 405.5 คำสั่งที่ใช้ร่วมกับ mergetool เสมอ

```bash
# เรียก mergetool สำหรับไฟล์ที่ conflict ทั้งหมด
git mergetool

# เรียกเฉพาะไฟล์ที่ระบุ
git mergetool -- path/to/file.js

# ปิดการสร้างไฟล์ .orig สำรอง (ค่า default จะสร้าง file.js.orig ทิ้งไว้)
git config --global mergetool.keepBackup false

# ดูรายชื่อ mergetool ที่ Git รู้จักในเครื่องนี้
git mergetool --tool-help
```

### 405.6 ตารางเปรียบเทียบ mergetool ยอดนิยม

| เครื่องมือ | ราคา | จุดเด่น | เหมาะกับ |
|---|---|---|---|
| VS Code (built-in merge editor) | ฟรี | ติดตั้งง่าย ใช้ editor ตัวเดียวกับที่เขียนโค้ดอยู่แล้ว | นักพัฒนาทั่วไปที่ใช้ VS Code เป็นหลัก |
| Meld | ฟรี, Open Source | UI ชัดเจน เบา รองรับ Linux ดีมาก | ผู้ใช้ Linux, ทีมที่ต้องการเครื่องมือฟรี |
| Beyond Compare | เสียเงิน (มี trial) | เทียบไฟล์/โฟลเดอร์/binary ได้ครบ เสถียรมาก | องค์กรขนาดใหญ่ที่ทำงานกับไฟล์หลากหลายชนิด |
| KDiff3 | ฟรี, Open Source | รองรับ auto-merge อัจฉริยะ | ผู้ใช้ Windows ที่ต้องการเครื่องมือฟรีที่ทรงพลัง |
| P4Merge | ฟรี | UI สวย มาจาก Perforce | ทีมที่มาจากพื้นเพ Perforce |

### 405.7 คำแนะนำการเลือกใช้

- ถ้าทำงานคนเดียวหรือทีมเล็ก ใช้ **VS Code merge editor** ก็เพียงพอและสะดวกที่สุดเพราะไม่ต้องสลับโปรแกรม
- ถ้าต้องแก้ conflict ที่ซับซ้อนมาก หลายไฟล์พร้อมกัน หรือต้องเทียบ binary/folder ด้วย ให้พิจารณา **Beyond Compare** หรือ **Meld**
- ไม่ว่าจะเลือกเครื่องมือไหน ให้ทุกคนในทีม **ตกลงกันว่าจะใช้เครื่องมืออะไรเป็นมาตรฐาน** และเขียนไว้ใน README หรือ CONTRIBUTING.md ของโปรเจกต์ เพื่อให้สมาชิกใหม่ตั้งค่าได้ตรงกันตั้งแต่วันแรก

---

## Step 406: กลยุทธ์ลด conflict ตั้งแต่ต้น

การมีเครื่องมือแก้ conflict ที่ดีเป็นเรื่องสำคัญ แต่ **กลยุทธ์ที่ดีที่สุดคือการป้องกันไม่ให้ conflict เกิดขึ้นตั้งแต่แรก** เพราะเวลาที่ดีที่สุดที่จะใช้แก้ conflict คือเวลาที่ไม่ต้องเสียไปกับมันเลย

### 406.1 ทำ Pull Request ให้เล็กและบ่อย

Conflict มีโอกาสเกิดขึ้นสูงมากขึ้นตามสัดส่วนของ:

1. **ขนาดของการเปลี่ยนแปลง** (บรรทัดที่แก้ยิ่งเยอะ ยิ่งมีโอกาสชนกับคนอื่น)
2. **ระยะเวลาที่ branch แยกออกจาก main** (ยิ่งนานยิ่งมีโอกาสที่ main จะเปลี่ยนไปมาก)

```
PR เล็ก + merge บ่อย = conflict น้อย + แก้ง่ายเมื่อเกิด
PR ใหญ่ + ค้างไว้นาน = conflict เยอะ + แก้ยากมากเมื่อเกิด
```

**แนวทางปฏิบัติ:**

- แตกงานใหญ่เป็น PR ย่อย ๆ ที่ merge ได้ภายใน 1-2 วัน แทนที่จะทำ branch เดียวยาวเป็นสัปดาห์
- ถ้างานจำเป็นต้องใช้เวลานาน ให้ใช้ **feature flag** เพื่อ merge โค้ดที่ยังไม่เสร็จเข้า main ได้อย่างปลอดภัย (จะกล่าวถึงรายละเอียดใน Part เกี่ยวกับ Trunk-Based Development)

### 406.2 Sync กับ main บ่อย ๆ

อย่าปล่อยให้ feature branch ห่างจาก main นานเกินไป ควรดึงการเปลี่ยนแปลงล่าสุดจาก main มา merge/rebase เข้า branch ตัวเองอย่างสม่ำเสมอ

```bash
# วิธีที่ 1: merge main เข้า feature บ่อย ๆ (ปลอดภัย ไม่เขียนประวัติทับ)
git checkout feature
git fetch origin
git merge origin/main

# วิธีที่ 2: rebase feature ทับ main ล่าสุด (ประวัติสะอาดกว่า แต่ต้องระวังถ้า branch นี้แชร์กับคนอื่น)
git checkout feature
git fetch origin
git rebase origin/main
```

**ประโยชน์:** ถ้ามี conflict เกิดขึ้น คุณจะเจอมันทีละนิดในแต่ละครั้งที่ sync แทนที่จะเจอ conflict มหาศาลก้อนเดียวตอนสุดท้ายที่พยายาม merge เข้า main

แนะนำให้ตั้งเป็นนิสัย: **sync กับ main อย่างน้อยวันละครั้ง** ถ้า branch ยังเปิดอยู่ข้ามวัน

### 406.3 แบ่งงานไม่ให้ชนไฟล์เดียวกัน

Conflict ส่วนใหญ่ในทีมไม่ได้เกิดจากบั๊กของ Git แต่เกิดจาก **การวางแผนงานที่ไม่ดี** ที่ปล่อยให้คนสองคนแก้ไฟล์เดียวกันพร้อมกันโดยไม่จำเป็น

**แนวทางปฏิบัติที่ทีมมืออาชีพใช้:**

1. **แบ่งงานตามขอบเขตไฟล์/โมดูลที่ชัดเจน** — ถ้าเป็นไปได้ ให้แต่ละคนรับผิดชอบไฟล์หรือโฟลเดอร์คนละส่วน
2. **สื่อสารก่อนเริ่มงานที่แตะไฟล์ร่วม** — เช่น ถ้ารู้ว่าต้องแก้ `routes.js` ที่เป็นไฟล์กลางที่หลายคนใช้ ให้แจ้งทีมก่อนเริ่มแก้ผ่าน stand-up หรือช่องแชทของทีม
3. **ออกแบบโครงสร้างโค้ดให้แยกส่วนได้ง่าย (modular)** — เช่น แทนที่จะมีไฟล์ `constants.js` ไฟล์เดียวที่ทุกคนแก้ ให้แยกเป็น `constants/api.js`, `constants/ui.js`, `constants/timeouts.js` เพื่อลดโอกาสชนกัน
4. **หลีกเลี่ยงไฟล์ "hot spot"** — ไฟล์ที่ถูกแก้บ่อยผิดปกติ (เช่น ไฟล์ config กลาง, routing table, index ที่ export ทุกอย่าง) ควรพิจารณาแยกให้เล็กลงหรือมีกระบวนการพิเศษในการแก้ (เช่น ต้องขอ approve ก่อนแก้)
5. **ใช้ CODEOWNERS** เพื่อให้ชัดเจนว่าไฟล์ไหนใครเป็นเจ้าของ ลดการแก้ไฟล์ข้ามทีมโดยไม่ประสานงาน (จะกล่าวถึงรายละเอียดของ `CODEOWNERS` ใน Part ที่เกี่ยวกับ GitHub ขั้นสูง)

### 406.4 ตกลง coding convention ให้ชัดเจนตั้งแต่ต้น

Conflict จำนวนมากไม่ได้เกิดจากการแก้ logic ที่ต่างกันจริง ๆ แต่เกิดจาก **สไตล์การจัดรูปแบบโค้ดที่ไม่ตรงกัน** เช่น การจัดเรียง import, การใช้ single quote/double quote, การจัดวรรคตอน

**แนวทางแก้:**

- ใช้ **linter/formatter อัตโนมัติ** (เช่น Prettier, ESLint --fix) ที่รันเหมือนกันทุกเครื่อง เพื่อให้ทุกคน format โค้ดออกมาเหมือนกันเป๊ะ ลด conflict ที่เกิดจาก whitespace/style ล้วน ๆ
- ตั้งค่า **pre-commit hook** ให้รัน formatter อัตโนมัติก่อน commit ทุกครั้ง (จะกล่าวถึงรายละเอียดของ Git Hooks ใน Part ที่เกี่ยวกับ Git ขั้นสูง)
- จัดเรียง import แบบมีมาตรฐานเดียวกัน (เช่นเรียงตามตัวอักษร) เพื่อลด conflict ตอนสองคนเพิ่ม import ใหม่พร้อมกันในไฟล์เดียวกัน

### 406.5 สรุปกลยุทธ์ลด conflict

| กลยุทธ์ | ผลลัพธ์ที่ได้ |
|---|---|
| PR เล็ก + merge บ่อย | conflict น้อยลง และแก้ง่ายเมื่อเกิด |
| Sync กับ main ทุกวัน | เจอ conflict ทีละนิด ไม่สะสมจนใหญ่ |
| แบ่งงานไม่ชนไฟล์เดียวกัน | ลด conflict ตั้งแต่ระดับวางแผนงาน |
| Linter/Formatter มาตรฐานเดียวกัน | ลด conflict จาก style ที่ไม่เกี่ยวกับ logic |
| CODEOWNERS ชัดเจน | ลดการแก้ไฟล์ข้ามทีมโดยไม่ประสานงาน |

---

## Step 407: การแก้ conflict ในไฟล์พิเศษ (binary file, lock file เช่น package-lock.json/yarn.lock)

### 407.1 Conflict ในไฟล์ Binary

ไฟล์ binary (รูปภาพ, PDF, ไฟล์ compiled, ไฟล์ font ฯลฯ) ไม่มีแนวคิดเรื่อง "บรรทัด" เหมือนไฟล์ text ดังนั้น Git **ไม่สามารถ merge เนื้อหาภายในไฟล์ binary ได้เลย** เมื่อทั้งสองฝั่งแก้ไฟล์ binary เดียวกัน Git จะฟ้อง conflict ทันทีโดยไม่พยายาม merge อัตโนมัติ

```bash
git merge feature
```

```
warning: Cannot merge binary files: logo.png (HEAD vs. feature)
CONFLICT (content): Merge conflict in logo.png
```

`git status`:

```
both modified:   logo.png
```

**วิธีแก้:** ต้องเลือกเอาไฟล์ของฝั่งใดฝั่งหนึ่งทั้งหมด ไม่มีทาง "รวม" เนื้อหาภายในไฟล์ binary ได้:

```bash
# เลือกเวอร์ชันของเรา (HEAD)
git checkout --ours logo.png
git add logo.png

# หรือเลือกเวอร์ชันของเขา (feature)
git checkout --theirs logo.png
git add logo.png

git commit
```

ถ้าต้องการดูความแตกต่างของไฟล์ binary ก่อนตัดสินใจ (เช่น ไฟล์รูปภาพ) แนะนำให้ใช้ mergetool แบบ visual ที่รองรับการแสดงรูปภาพเทียบกัน (Beyond Compare ทำได้ดีในเรื่องนี้) หรือเปิดไฟล์ทั้งสองเวอร์ชันด้วยโปรแกรมดูรูปเทียบกันเอง

**ทางเลือกป้องกัน:** สำหรับโปรเจกต์ที่มีไฟล์ binary ขนาดใหญ่จำนวนมาก (เช่น asset ของเกม) ควรพิจารณาใช้ **Git LFS (Large File Storage)** ซึ่งจะกล่าวถึงรายละเอียดในภายหลังของหลักสูตร (เฟส 6: Git ขั้นสูง) และควรกำหนดนโยบายว่า **ใครคนเดียวเท่านั้นที่แก้ไขไฟล์ binary แต่ละไฟล์ในแต่ละช่วงเวลา** เพื่อลดโอกาส conflict ที่แก้ไม่ได้จริง ๆ

### 407.2 Conflict ใน Lock File (package-lock.json, yarn.lock, Gemfile.lock, poetry.lock)

Lock file เป็นไฟล์ที่ **สร้างขึ้นอัตโนมัติ** โดย package manager (npm, yarn, pnpm, bundler, poetry ฯลฯ) เพื่อล็อกเวอร์ชันที่แน่นอนของทุก dependency (รวมถึง dependency ของ dependency) ไฟล์นี้มักมีขนาดใหญ่มากและถูกแก้ไขบ่อยมากทุกครั้งที่มีคน `npm install` แพ็กเกจใหม่ ทำให้เป็นจุดที่เกิด conflict บ่อยที่สุดจุดหนึ่งในโปรเจกต์

**ทำไม conflict ในไฟล์นี้ถึงแก้ด้วยมือยากและอันตราย:**

- เนื้อหาเป็น JSON/YAML ที่ซับซ้อนมาก มี hash และ dependency tree ที่เชื่อมโยงกันละเอียด
- การแก้ conflict ด้วยมือแล้วเลือกผิดจุด อาจทำให้ dependency tree เสียหาย ทำให้ `npm install` ล้มเหลว หรือแย่กว่านั้นคือติดตั้งผ่านได้แต่ได้ dependency ที่ไม่ตรงกับที่ตั้งใจ

**วิธีแก้ที่ถูกต้อง: อย่าแก้ conflict marker ด้วยมือ ให้สร้างไฟล์ใหม่แทน**

```bash
# ขั้นตอนที่ถูกต้องเมื่อ package-lock.json conflict
git status
```

```
both modified:   package-lock.json
```

```bash
# ลบไฟล์ conflict ทิ้ง
git checkout --ours package.json    # ตรวจสอบ package.json ให้แน่ใจว่าถูกต้องก่อน (ไฟล์นี้ควร merge ด้วยมือได้ปกติ เพราะเป็น JSON เรียบง่าย)
rm package-lock.json

# สร้างไฟล์ lock ใหม่จาก package.json ที่ merge เสร็จแล้ว
npm install

# เพิ่มไฟล์ lock ใหม่เข้า staging
git add package-lock.json
git commit
```

**หลักการสำคัญ:** conflict ที่แท้จริงมักจะอยู่ที่ **`package.json`** (ไฟล์ที่มนุษย์เขียนและควรจะ merge เนื้อหาได้อย่างสมเหตุสมผล เช่น การเพิ่ม dependency ใหม่คนละตัว) ส่วน **`package-lock.json` ควรถูก regenerate ใหม่เสมอ** จาก `package.json` ที่ merge เรียบร้อยแล้ว ไม่ใช่พยายามรวม lock file สองเวอร์ชันด้วยมือ

ขั้นตอนเต็มที่แนะนำ:

1. แก้ conflict ใน `package.json` ด้วยมือให้เรียบร้อยก่อน (รวม dependency ทั้งสองฝั่งเข้าด้วยกัน)
2. `git add package.json`
3. ลบ `package-lock.json` ที่มี conflict marker ทิ้ง
4. รัน `npm install` (หรือ `yarn install`, `pnpm install` ตามเครื่องมือที่ใช้) เพื่อ regenerate lock file ใหม่ทั้งหมดจาก `package.json`
5. `git add package-lock.json`
6. `git commit`

ทำแบบเดียวกันได้กับเครื่องมืออื่น ๆ:

| Package Manager | ไฟล์ lock | คำสั่ง regenerate |
|---|---|---|
| npm | `package-lock.json` | `npm install` |
| Yarn (classic) | `yarn.lock` | `yarn install` |
| pnpm | `pnpm-lock.yaml` | `pnpm install` |
| Bundler (Ruby) | `Gemfile.lock` | `bundle install` |
| Poetry (Python) | `poetry.lock` | `poetry lock` |
| Cargo (Rust) | `Cargo.lock` | `cargo build` หรือ `cargo update` |
| Composer (PHP) | `composer.lock` | `composer install` |

**เคล็ดลับ:** บางทีมตั้งค่า `.gitattributes` ให้ Git จัดการ lock file แบบพิเศษ เช่น กำหนด merge driver เฉพาะ หรือใช้เครื่องมือเสริมอย่าง `npm-merge-driver` ที่ automation การ regenerate ให้อัตโนมัติเมื่อเกิด conflict:

```bash
npx npm-merge-driver install -g
```

หลังติดตั้งแล้ว เครื่องมือนี้จะตั้งค่า merge driver ใน `.gitattributes`/`.git/config` ให้ Git เรียก `npm install` อัตโนมัติเมื่อเจอ conflict ใน `package-lock.json` แทนที่จะปล่อยให้เกิด conflict marker ที่ต้องแก้ด้วยมือ

---

## Step 408: Conflict ตอน rebase ยาว ๆ หลาย commit ทำอย่างไรให้ไม่งง

### 408.1 ทำไม rebase ยาว ๆ ถึงทำให้งงง่าย

`git rebase` ทำงานโดย **นำ commit ทีละตัวจาก branch ของเรา ไปเล่นซ้ำ (replay) บนฐานใหม่ทีละ commit ตามลำดับ** ถ้า branch มี 15 commits และแต่ละ commit มี conflict กับ base ใหม่ คุณจะต้องแก้ conflict **ทีละรอบ ทีละ commit** ไม่ใช่ครั้งเดียวจบเหมือน merge

```bash
git checkout feature
git rebase main
```

```
Rebasing (1/15)
CONFLICT (content): Merge conflict in api.js
```

ถ้าไม่เข้าใจกลไกนี้ หลายคนจะสับสนว่า "ทำไมแก้ conflict ไปแล้วแต่ยังไม่จบ ทำไมมันฟ้อง conflict อีกรอบ" — คำตอบคือ **แต่ละ commit ที่กำลังถูก replay จะสร้างสถานการณ์ conflict ของตัวเองแยกกัน**

### 408.2 ขั้นตอนที่ถูกต้องในการไล่แก้ทีละรอบ

หลักการคือ ใช้ `--continue` **ทีละรอบ** จนกว่า rebase จะเสร็จสมบูรณ์:

```bash
git checkout feature
git rebase main
```

**รอบที่ 1 — conflict ที่ commit แรก:**

```
Auto-merging api.js
CONFLICT (content): Merge conflict in api.js
error: could not apply a1b2c3d... fix: update endpoint URL
```

```bash
# ตรวจสอบว่ากำลังอยู่ commit ไหนของ rebase
git status
```

```
interactive rebase in progress; onto 9f8e7d6
Last command done (1 command done):
   pick a1b2c3d fix: update endpoint URL
No commands remaining.
You are currently rebasing branch 'feature' on '9f8e7d6'.
  (fix conflicts and run "git rebase --continue")
```

```bash
# แก้ conflict ในไฟล์ api.js
git add api.js
git rebase --continue
```

**รอบที่ 2 — ถ้ามี conflict ต่อที่ commit ถัดไป:**

```
Rebasing (2/15)
CONFLICT (content): Merge conflict in api.js
```

ทำซ้ำแบบเดิม: แก้ไฟล์ → `git add` → `git rebase --continue`

**ทำแบบนี้ไปเรื่อย ๆ จนครบทุก commit** ระบบจะแสดงข้อความสุดท้ายเมื่อเสร็จสมบูรณ์:

```
Successfully rebased and updated refs/heads/feature.
```

### 408.3 เทคนิคที่ช่วยไม่ให้งงระหว่างทาง

**1. ดูว่าตอนนี้กำลังอยู่ commit ไหน จากทั้งหมดกี่ commit**

```bash
git status
```

ข้อความจะบอกลำดับ เช่น `Rebasing (3/15)` เสมอ ให้เช็คทุกครั้งที่ไม่แน่ใจว่าอยู่ตรงไหน

**2. ดู commit message ของ commit ที่กำลังถูก replay**

```bash
cat .git/rebase-merge/message 2>/dev/null || cat .git/rebase-apply/msg-clean 2>/dev/null
```

หรือดูจาก error message ตอน rebase ค้าง ซึ่งจะโชว์ short hash และ commit message ของ commit ที่ทำให้เกิด conflict เสมอ

**3. ใช้ `git diff` ตรวจสอบก่อน `--continue` ทุกครั้ง**

```bash
git diff --staged
```

เพื่อยืนยันว่าสิ่งที่แก้ไปนั้นถูกต้องจริง ก่อนสั่ง continue ต่อ ป้องกันการรีบ continue โดยไม่ทันตรวจสอบ

**4. ถ้า commit ไหนไม่มีเนื้อหาเหลือหลัง merge (เพราะการเปลี่ยนแปลงถูกรวมเข้ากับ commit ก่อนหน้าไปหมดแล้ว) ให้ใช้ `--skip`**

```bash
git rebase --skip
```

คำสั่งนี้จะ**ข้าม commit ปัจจุบันไปเลย** ไม่นำการเปลี่ยนแปลงของมันไปรวมกับผลลัพธ์สุดท้าย ควรใช้ด้วยความระมัดระวังมาก และตรวจสอบให้แน่ใจว่าการเปลี่ยนแปลงของ commit นั้นไม่ได้หายไปอย่างไม่ได้ตั้งใจ (เช่น มันอาจถูกรวมเข้ากับการแก้ conflict ของ commit ก่อนหน้าไปแล้วโดยไม่มีอะไรเหลือให้ apply ต่อ)

**5. หยุดพักได้ถ้ารู้สึกงงเกินไป**

Rebase ที่ยังไม่เสร็จจะค้างอยู่ในสถานะกลางทางได้เรื่อย ๆ โดยไม่มีปัญหา คุณสามารถหยุดพัก ไปตรวจสอบโค้ดในโหมด detached HEAD ปัจจุบัน หรือขอความช่วยเหลือจากเพื่อนร่วมทีมก่อนดำเนินการต่อได้ ไม่จำเป็นต้องรีบ continue ให้จบในรวดเดียวถ้าไม่มั่นใจ

### 408.4 ใช้ `--rerere-autoupdate` ร่วมกับ rebase ยาว ๆ

ถ้าเปิด `git rerere` ไว้ (ตาม Step 404) จะช่วยได้มากเป็นพิเศษกับ rebase ยาว ๆ ที่มี conflict pattern ซ้ำกัน สามารถเพิ่ม flag นี้เพื่อให้ Git `git add` ไฟล์ที่ rerere แก้ให้อัตโนมัติได้ทันทีโดยไม่ต้อง `add` เอง:

```bash
git rebase --rerere-autoupdate main
```

หรือเปิดเป็นค่า default เสมอ:

```bash
git config --global rerere.autoupdate true
```

**คำเตือน:** ต้องมั่นใจว่า resolution ที่ rerere บันทึกไว้ก่อนหน้าถูกต้องจริง เพราะถ้าเปิด autoupdate ไว้ Git จะ `add` ให้ทันทีโดยไม่รอให้คุณตรวจสอบก่อน ควรใช้คู่กับการตรวจสอบด้วย `git diff --staged` ก่อน `--continue` เสมอ

### 408.5 สรุปขั้นตอน rebase ยาวที่มี conflict หลายรอบ

```
1. git rebase main
2. เจอ conflict → git status ดูว่าอยู่ commit ที่เท่าไหร่
3. แก้ไฟล์ conflict (ใช้ diff3/zdiff3 หรือ mergetool ช่วยดู)
4. git diff --staged ตรวจสอบก่อน add
5. git add <files>
6. git rebase --continue
7. วนซ้ำ 2-6 จนกว่าจะขึ้น "Successfully rebased..."
8. ถ้า commit ไหนไม่มีอะไรเหลือ ใช้ git rebase --skip (ระวังมาก)
9. ถ้างงเกินไป หรือพบว่าผลลัพธ์ผิดเพี้ยนไปไกลจนแก้ไม่ทัน → ไป Step 409 (abort)
```

---

## Step 409: การ abort อย่างปลอดภัยเมื่อ conflict ซับซ้อนเกินจะแก้ไหว

### 409.1 เมื่อไหร่ที่ควร abort แทนที่จะฝืนแก้ต่อ

ไม่ใช่ทุก conflict ที่ควรพยายามแก้ให้จบในทันที บางครั้งสัญญาณที่บอกว่าควรหยุดและ abort มีดังนี้:

- คุณไม่แน่ใจว่าการแก้ไขที่ทำไปแล้วถูกต้องหรือไม่ และเริ่มเดามั่ว ๆ เพื่อให้ conflict หายไป
- จำนวน conflict block เยอะเกินกว่าจะตรวจสอบได้อย่างรอบคอบในเวลาที่มี (เช่น เจอ conflict 40 จุดในไฟล์เดียว)
- คุณกำลังทำ rebase ยาวและเริ่มไม่แน่ใจว่า commit ที่กำลัง replay อยู่คืออันไหน ทำอะไร
- คุณค้นพบระหว่างทางว่า branch ที่กำลังจะ merge เข้ามานั้น **มีปัญหาเชิง design** ที่ต้องคุยกับทีมก่อน ไม่ใช่แค่ปัญหาทาง technical ในการรวมโค้ด
- เวลาที่มีจำกัด (เช่น ใกล้เวลาประชุม) และไม่อยากทิ้ง repository ไว้ในสถานะครึ่ง ๆ กลาง ๆ

**หลักการสำคัญ:** การ abort ไม่ใช่ความล้มเหลว แต่เป็น **การตัดสินใจที่ถูกต้องอย่างมืออาชีพ** — Git ออกแบบให้ abort ปลอดภัย 100% เสมอ ปลอดภัยกว่าการฝืนแก้ต่อในสภาพที่ไม่มั่นใจไปมาก

### 409.2 คำสั่ง abort ตามสถานการณ์

**ระหว่าง merge:**

```bash
git merge --abort
```

คืนสถานะ working directory และ index กลับไปเหมือนก่อนสั่ง `git merge` ทุกประการ

**ระหว่าง rebase:**

```bash
git rebase --abort
```

คืน branch กลับไปที่ตำแหน่งเดิมก่อนเริ่ม rebase ทั้งหมด (ยกเลิกทุก commit ที่ replay ไปแล้วบางส่วน)

**ระหว่าง cherry-pick:**

```bash
git cherry-pick --abort
```

**ระหว่าง revert:**

```bash
git revert --abort
```

**ระหว่างใช้ mergetool ค้างอยู่:** ถ้าเปิด mergetool ไว้แล้วอยากยกเลิกทั้งหมด ให้ปิดโปรแกรม mergetool ก่อน แล้วค่อยสั่ง `git merge --abort` (หรือ `git rebase --abort` แล้วแต่กรณี) ตามปกติ

### 409.3 ตรวจสอบสถานะก่อน abort เสมอ

ก่อนสั่ง abort ควรเช็คก่อนว่าตอนนี้กำลังอยู่ในกระบวนการอะไร เพราะคำสั่ง abort ต่างกันตามชนิดของ operation:

```bash
git status
```

ข้อความจะบอกชัดเจนว่าตอนนี้อยู่ระหว่างอะไร เช่น:

```
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)
```

หรือ

```
interactive rebase in progress; onto 9f8e7d6
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --abort" to check out the original branch)
```

Git จะบอกคำสั่ง abort ที่ถูกต้องให้เสมอในข้อความนี้ — **อ่านให้ครบก่อนตัดสินใจพิมพ์คำสั่งเอง**

### 409.4 ทางเลือกระหว่าง abort กับ quit/skip

Git มีคำสั่งใกล้เคียงกันที่ความหมายต่างกันมาก ต้องแยกให้ออก:

| คำสั่ง | ความหมาย | ผลลัพธ์ |
|---|---|---|
| `git rebase --abort` | ยกเลิกทั้งหมด | กลับไปจุดก่อนเริ่ม rebase ทุกประการ เหมือนไม่เคยเริ่มเลย |
| `git rebase --quit` | เลิกกระบวนการ rebase แต่ **ไม่คืนสถานะ** | branch จะค้างอยู่ที่ตำแหน่งกลางทาง (ใช้เมื่อจะไปแก้ไขสถานะเองด้วยมือ) — ใช้ยากและเสี่ยงกว่า ไม่แนะนำสำหรับมือใหม่ |
| `git rebase --skip` | ข้าม commit ปัจจุบันไปเลย แล้วทำ rebase ต่อ | commit นั้นหายไปจากผลลัพธ์สุดท้าย |

**คำแนะนำ:** สำหรับสถานการณ์ "conflict ซับซ้อนเกินจะแก้ไหว" ให้ใช้ `--abort` เสมอ ไม่ใช่ `--quit` เพราะ `--abort` รับประกันว่าคุณจะกลับไปสถานะที่ปลอดภัยและรู้จักดีอยู่แล้วเสมอ

### 409.5 สิ่งที่ควรทำหลัง abort

1. **หายใจลึก ๆ แล้วประเมินสถานการณ์ใหม่** — ทำไม conflict ถึงซับซ้อนขนาดนี้ เป็นเพราะ branch ห่างจาก main นานเกินไปหรือไม่ (ดู Step 406)
2. **ขอความช่วยเหลือจากเจ้าของโค้ดอีกฝั่ง** — คุยกับคนที่เขียน commit ที่ conflict ด้วยกัน เพื่อเข้าใจเจตนาของโค้ดทั้งสองฝั่งก่อนแก้ต่อ
3. **แบ่งการแก้ conflict เป็นก้อนเล็กลง** — เช่น ถ้า merge ทั้ง branch ทีเดียวยากเกินไป ลอง merge ทีละ commit จาก branch อื่นด้วย `git cherry-pick` ทีละตัวแทน เพื่อแก้ conflict ทีละก้อนเล็ก ๆ
4. **เปิด diff3/zdiff3 และ rerere ไว้** (Step 403, 404) ก่อนลองใหม่ เพื่อให้มีข้อมูลช่วยตัดสินใจมากขึ้น
5. **บันทึกสิ่งที่เรียนรู้ไว้** — ถ้า conflict นี้เกิดจากปัญหาเชิงโครงสร้างโค้ด (เช่นไฟล์ hot spot) ให้เสนอแนวทางแก้ระยะยาวกับทีม เช่น การแยกไฟล์ให้เล็กลงตาม Step 406.3

### 409.6 กรณีพิเศษ: ค้างระหว่าง conflict มานาน แล้วลืมว่าอยู่ระหว่าง operation อะไร

ถ้าคุณกลับมาเปิด repository อีกครั้งหลังจากปล่อยทิ้งไว้นาน แล้วจำไม่ได้ว่ากำลังทำอะไรอยู่:

```bash
git status
```

จะบอกทุกอย่างที่ต้องรู้เสมอ — ทั้งชนิดของ operation ที่ค้างอยู่ (merge/rebase/cherry-pick) และคำสั่งที่ควรใช้ต่อ (`--continue` หรือ `--abort`) ถ้ายังไม่แน่ใจอีก ให้ตรวจสอบเพิ่มด้วย:

```bash
ls .git/ | grep -E "MERGE_HEAD|rebase-merge|rebase-apply|CHERRY_PICK_HEAD"
```

การมีไฟล์/โฟลเดอร์เหล่านี้อยู่ใน `.git/` เป็นสัญญาณยืนยันว่ามี operation ค้างอยู่จริง และบอกได้ว่าเป็น operation ประเภทไหน (`MERGE_HEAD` = merge ค้าง, `rebase-merge`/`rebase-apply` = rebase ค้าง, `CHERRY_PICK_HEAD` = cherry-pick ค้าง)

---

## Step 410: แบบฝึกหัด — จำลองสถานการณ์ทีมจริง แก้ conflict หลายไฟล์พร้อมกัน (รวม rename conflict)

ถึงเวลาลงมือปฏิบัติจริง แบบฝึกหัดนี้จำลองสถานการณ์ทีมที่มีสองคนทำงานคู่ขนานกันบนโปรเจกต์เดียวกัน แล้วต้อง merge เข้าด้วยกัน โดยจะเจอ conflict หลายแบบพร้อมกันในครั้งเดียว

### 410.1 เตรียมโปรเจกต์จำลอง

```bash
mkdir ~/git-course/part-41-conflict-lab
cd ~/git-course/part-41-conflict-lab
git init

git config user.name "ทีมทดสอบ"
git config user.email "test@example.com"
git config merge.conflictstyle zdiff3
git config rerere.enabled true
```

สร้างไฟล์เริ่มต้นบน `main`:

```bash
cat > app-config.js << 'EOF'
const config = {
  apiUrl: "https://api.example.com",
  timeout: 3000,
  retries: 3,
};

module.exports = config;
EOF

cat > user-service.js << 'EOF'
function getUser(id) {
  return fetch(`/users/${id}`);
}

module.exports = { getUser };
EOF

cat > README.md << 'EOF'
# โปรเจกต์ทดสอบ Conflict

โปรเจกต์นี้ใช้สำหรับฝึกแก้ conflict ขั้นสูง
EOF

git add .
git commit -m "initial commit: base project structure"
```

### 410.2 สร้าง branch A (ตัวแทนทีม Backend)

```bash
git checkout -b team-backend

# 1) แก้ config: เพิ่ม timeout
sed -i 's/timeout: 3000,/timeout: 8000,/' app-config.js
git commit -am "team-backend: increase timeout to 8000ms for slow network"

# 2) rename ไฟล์ user-service.js -> user-repository.js
git mv user-service.js user-repository.js
git commit -am "team-backend: rename user-service.js to user-repository.js for clarity"

# 3) ลบไฟล์ README (ตั้งใจย้ายไปทำ docs แยกต่างหาก)
git rm README.md
git commit -am "team-backend: remove README, migrating docs to separate site"

# 4) สร้างไฟล์ใหม่ constants.js
cat > constants.js << 'EOF'
module.exports = {
  MAX_CONNECTIONS: 100,
};
EOF
git add constants.js
git commit -m "team-backend: add constants.js with MAX_CONNECTIONS"
```

### 410.3 กลับไปที่ main แล้วสร้าง branch B (ตัวแทนทีม Frontend)

```bash
git checkout main
git checkout -b team-frontend

# 1) แก้ config: เพิ่ม retries คนละจุดกับที่ backend แก้ timeout
sed -i 's/retries: 3,/retries: 5,/' app-config.js
git commit -am "team-frontend: increase retries to 5 for flaky connections"

# 2) rename ไฟล์ user-service.js -> user-client.js (คนละชื่อกับที่ backend เลือก!)
git mv user-service.js user-client.js
git commit -am "team-frontend: rename user-service.js to user-client.js"

# 3) แก้ไข README ต่อ (ไม่รู้ว่า backend ลบไปแล้ว)
cat >> README.md << 'EOF'

## วิธีติดตั้ง

npm install && npm start
EOF
git commit -am "team-frontend: add installation instructions to README"

# 4) สร้างไฟล์ constants.js เหมือนกัน แต่เนื้อหาต่างกัน (add/add conflict)
cat > constants.js << 'EOF'
module.exports = {
  DEFAULT_LOCALE: "th-TH",
};
EOF
git add constants.js
git commit -m "team-frontend: add constants.js with DEFAULT_LOCALE"
```

### 410.4 ลอง merge ทั้งสอง branch เข้าด้วยกัน

```bash
git checkout main
git merge team-backend
```

ควรจะ merge ผ่านแบบ fast-forward หรือสร้าง merge commit ได้โดยไม่มี conflict (เพราะ merge กับ main ที่ยังไม่มีอะไรเปลี่ยนเลยตั้งแต่แยก branch)

ทีนี้มาถึงจุดสำคัญ — merge `team-frontend` เข้ามาด้วย ซึ่งจะชนกับสิ่งที่ `team-backend` ทำไว้ในหลายมิติพร้อมกัน:

```bash
git merge team-frontend
```

คุณควรเจอ conflict หลายแบบพร้อมกันประมาณนี้:

```
CONFLICT (content): Merge conflict in app-config.js
CONFLICT (rename/rename): Rename "user-service.js"->"user-repository.js" in "HEAD" rename "user-service.js"->"user-client.js" in "team-frontend"
CONFLICT (modify/delete): README.md deleted in HEAD and modified in team-frontend. Version team-frontend of README.md left in tree.
CONFLICT (add/add): Merge conflict in constants.js
```

**สังเกตว่านี่คือ conflict ครบทั้ง 4 แบบที่เรียนมาใน Step 402 และ 407 เกิดขึ้นพร้อมกันในการ merge ครั้งเดียว** — เป็นสถานการณ์ที่สมจริงมากเมื่อสอง team ทำงานคู่ขนานกันนานเกินไปโดยไม่ sync บ่อย ๆ (ตรงกับที่เตือนไว้ใน Step 406)

### 410.5 ไล่แก้ทีละ conflict อย่างเป็นระบบ

**ตรวจสอบภาพรวมก่อนเสมอ:**

```bash
git status
```

```
Unmerged paths:
  both modified:      app-config.js
  deleted by us:      README.md
  both added:         constants.js
  both renamed:        user-service.js -> user-repository.js
                        user-service.js -> user-client.js
```

**แก้จุดที่ 1: `app-config.js` (content conflict ธรรมดา)**

```bash
cat app-config.js
```

จะเห็น (เพราะเปิด zdiff3 ไว้):

```javascript
const config = {
  apiUrl: "https://api.example.com",
<<<<<<< HEAD
  timeout: 8000,
  retries: 3,
||||||| merged common ancestors
  timeout: 3000,
  retries: 3,
=======
  timeout: 3000,
  retries: 5,
>>>>>>> team-frontend
};

module.exports = config;
```

ทั้งสองฝั่งแก้คนละค่า ไม่ได้ขัดแย้งกันจริง ๆ — รวมเข้าด้วยกันได้เลย:

```javascript
const config = {
  apiUrl: "https://api.example.com",
  timeout: 8000,
  retries: 5,
};

module.exports = config;
```

```bash
git add app-config.js
```

**แก้จุดที่ 2: rename/rename conflict ของไฟล์ user-service.js**

ทีมตกลงกันว่าจะใช้ชื่อ `user-repository.js` เป็นชื่อสุดท้าย (ชื่อของฝั่ง backend เพราะสื่อความหมายตรงกับ pattern repository ที่ใช้ทั้งโปรเจกต์):

```bash
ls user-*.js
```

```
user-client.js  user-repository.js
```

ตรวจดูเนื้อหาทั้งสองไฟล์ก่อนตัดสินใจว่ามีการแก้ไขเนื้อหาที่ต้องรวมกันหรือไม่ (ในตัวอย่างนี้ทั้งสองไฟล์เนื้อหาเหมือนกันเพราะแค่ rename เฉย ๆ ไม่ได้แก้เนื้อหา):

```bash
diff user-client.js user-repository.js
```

ถ้าไม่มีความต่างของเนื้อหา ให้เก็บไฟล์ที่ทีมตกลงกันไว้ และลบอีกไฟล์ทิ้ง:

```bash
git rm user-client.js
git add user-repository.js
```

**แก้จุดที่ 3: delete/modify conflict ของ README.md**

Backend ลบ README ไปแล้วเพราะจะย้ายไปทำ docs site แยก แต่ Frontend เพิ่งเพิ่มคำแนะนำการติดตั้งเข้าไป ต้องคุยกันว่าจะเอาอย่างไร — สมมติทีมตัดสินใจว่า **เก็บ README ไว้ก่อน** จนกว่า docs site ใหม่จะพร้อม เพื่อไม่ให้ข้อมูลการติดตั้งหายไป:

```bash
cat README.md    # Git คงไฟล์เวอร์ชันของ team-frontend ไว้ให้อัตโนมัติแล้ว
git add README.md
```

(ถ้าทีมตัดสินใจในทางกลับกันว่าจะลบจริง ๆ ก็ใช้ `git rm README.md` แทน)

**แก้จุดที่ 4: add/add conflict ของ constants.js**

```bash
cat constants.js
```

```javascript
<<<<<<< HEAD
module.exports = {
  MAX_CONNECTIONS: 100,
};
=======
module.exports = {
  DEFAULT_LOCALE: "th-TH",
};
>>>>>>> team-frontend
```

ทั้งสองค่าไม่ได้ขัดแย้งกัน รวมเข้าด้วยกัน:

```javascript
module.exports = {
  MAX_CONNECTIONS: 100,
  DEFAULT_LOCALE: "th-TH",
};
```

```bash
git add constants.js
```

### 410.6 ตรวจสอบให้ครบก่อน commit

```bash
git status
```

```
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
	modified:   app-config.js
	modified:   README.md
	new file:   constants.js
	renamed:    user-service.js -> user-repository.js
```

ตรวจสอบทุกไฟล์อีกครั้งว่าไม่มี conflict marker หลงเหลือ:

```bash
grep -rn "<<<<<<<\|=======\|>>>>>>>" --include="*.js" --include="*.md" .
```

ถ้าไม่มีผลลัพธ์ใด ๆ แสดงว่าปลอดภัย ไม่มี marker หลงเหลือ

### 410.7 ปิดจบการ merge

```bash
git commit
```

Git จะเปิด editor พร้อม commit message ที่ generate ไว้ล่วงหน้าอธิบาย merge นี้ ให้ตรวจสอบและเพิ่มรายละเอียดถ้าจำเป็น เช่น:

```
Merge branch 'team-frontend'

Resolved conflicts:
- app-config.js: combined timeout (8000) and retries (5) from both teams
- user-service.js rename conflict: kept "user-repository.js" per team agreement
- README.md: kept file with frontend's installation instructions
- constants.js: merged both MAX_CONNECTIONS and DEFAULT_LOCALE
```

บันทึกและปิด editor เพื่อยืนยัน commit

### 410.8 ตรวจสอบผลลัพธ์สุดท้าย

```bash
git log --graph --oneline --all
ls
cat app-config.js
cat constants.js
```

ควรเห็นว่า:
- ไม่มีไฟล์ `user-client.js` หรือ `user-service.js` เหลืออยู่ มีแค่ `user-repository.js`
- `README.md` ยังอยู่พร้อมเนื้อหาคำแนะนำการติดตั้ง
- `constants.js` มีทั้ง `MAX_CONNECTIONS` และ `DEFAULT_LOCALE`
- `app-config.js` มีทั้ง `timeout: 8000` และ `retries: 5`

### 410.9 ลองสถานการณ์ทางเลือก — ฝึก abort

เพื่อฝึก Step 409 ให้ลองสร้างสถานการณ์ conflict ใหม่แล้วฝึก abort ดูจริง:

```bash
git checkout -b practice-abort main
echo "// experimental change" >> app-config.js
git commit -am "practice: experimental change for abort drill"

git checkout main
git merge team-frontend    # (ถ้ายัง merge ไม่เสร็จจาก step ก่อน ให้ merge เข้ามาก่อน)

git merge practice-abort
```

เมื่อเจอ conflict สมมติว่าตัดสินใจว่ายังไม่พร้อมจะแก้ตอนนี้:

```bash
git status
git merge --abort
git status
```

ตรวจสอบว่า `git status` แสดงผล **clean** และ `app-config.js` กลับไปเป็นเนื้อหาก่อน merge ทุกประการ ยืนยันว่า abort ทำงานถูกต้องและปลอดภัย 100%

### 410.10 Checklist แบบฝึกหัด

- [ ] สร้าง repository จำลองพร้อม branch สองอันที่ conflict กันหลายแบบสำเร็จ
- [ ] เจอและแก้ content conflict ใน `app-config.js` ได้ถูกต้อง
- [ ] เจอและแก้ rename/rename conflict ของไฟล์ user service ได้ถูกต้อง
- [ ] เจอและแก้ delete/modify conflict ของ `README.md` ได้ถูกต้อง
- [ ] เจอและแก้ add/add conflict ของ `constants.js` ได้ถูกต้อง
- [ ] ตรวจสอบไม่มี conflict marker หลงเหลือก่อน commit
- [ ] commit ปิดจบ merge สำเร็จพร้อมข้อความอธิบายชัดเจน
- [ ] ฝึก `git merge --abort` และยืนยันว่าคืนสถานะได้ปลอดภัยจริง

---

## สรุป Part 41

ใน Part นี้เราได้ต่อยอดความรู้เรื่อง merge conflict จากระดับพื้นฐานใน Part 08 ไปสู่ระดับที่ใช้งานได้จริงในทีมขนาดใหญ่:

1. Conflict ไม่ได้มีแค่แบบ content conflict ธรรมดา ยังมี **rename/rename**, **delete/modify**, และ **add/add** conflict ที่ต้องเข้าใจกลไกเบื้องหลังให้ถูกต้องก่อนตัดสินใจแก้ เพราะเลือกผิดอาจทำให้โค้ดหรือไฟล์สำคัญหายไปอย่างไม่รู้ตัว
2. การตั้งค่า `merge.conflictstyle diff3` หรือ `zdiff3` ช่วยให้เห็น **common ancestor** ของ conflict ทำให้เข้าใจเจตนาของทั้งสองฝั่งได้ชัดเจนกว่าการเห็นแค่สองฝั่งที่ชนกัน
3. `git rerere` ช่วยประหยัดเวลามหาศาลเมื่อต้องแก้ conflict pattern เดิมซ้ำหลายครั้ง โดยเฉพาะระหว่าง rebase ยาว ๆ หรือ merge branch เดิมกลับไปกลับมาบ่อย — แต่ต้องตรวจสอบผลลัพธ์เสมอ ไม่ไว้ใจแบบไม่ดูเลย
4. Mergetool ภายนอกอย่าง VS Code, Meld, Beyond Compare ช่วยให้แก้ conflict แบบ visual ได้ง่ายกว่าไล่หา marker ในไฟล์ดิบ โดยเฉพาะเมื่อไฟล์มีหลาย conflict block พร้อมกัน
5. กลยุทธ์ที่ดีที่สุดในการจัดการ conflict คือ **การป้องกันไม่ให้เกิดตั้งแต่ต้น** ผ่าน PR เล็ก, sync กับ main บ่อย, แบ่งงานไม่ให้ชนไฟล์เดียวกัน และใช้ linter/formatter มาตรฐานเดียวกันทั้งทีม
6. ไฟล์พิเศษอย่าง **binary file** ต้องเลือกทั้งไฟล์ (ไม่มีทาง merge เนื้อหาภายในได้) ส่วน **lock file** อย่าง `package-lock.json`/`yarn.lock` ควร **regenerate ใหม่เสมอ** จากไฟล์ manifest ที่ merge เรียบร้อยแล้ว ไม่ใช่แก้ conflict marker ในไฟล์ lock ด้วยมือ
7. Conflict ระหว่าง rebase ยาว ๆ ต้องแก้ **ทีละ commit** ด้วย `--continue` ซ้ำ ๆ พร้อมตรวจสอบสถานะด้วย `git status` เสมอว่ากำลังอยู่ commit ไหนจากทั้งหมดกี่ commit
8. การ `--abort` ไม่ใช่ความล้มเหลว แต่เป็นการตัดสินใจที่ปลอดภัยและเป็นมืออาชีพ เมื่อ conflict ซับซ้อนเกินกว่าจะแก้ได้อย่างมั่นใจในเวลานั้น
9. แบบฝึกหัดจำลองสถานการณ์ทีมจริงช่วยให้เห็นภาพว่า conflict หลายแบบสามารถเกิดขึ้นพร้อมกันได้ในการ merge ครั้งเดียว และการไล่แก้อย่างเป็นระบบทีละจุดคือกุญแจสำคัญที่ทำให้ไม่หลงทาง

**ต่อไป:** [Part 42: Git Bisect: หาบั๊กด้วยวิธี Binary Search](./part-042-git-bisect.md)
