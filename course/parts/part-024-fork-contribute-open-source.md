# Part 24: Fork และการ Contribute แบบ Open Source เบื้องต้น

> **Step ในหลักสูตรนี้:** Step 231–240
> **เฟส:** 3 — ใช้งาน GitHub อย่างมืออาชีพ ทำ Pull Request, Code Review, Open Source
> **เป้าหมายของ Part นี้:** เข้าใจว่า Fork คืออะไรและต่างจาก Clone อย่างไร ฝึก Fork repository จริงมา clone ทำงานที่เครื่อง ตั้งค่า upstream remote เพื่อติดตามการอัปเดตจากต้นฉบับ Sync fork ให้ทันสมัยอยู่เสมอ ส่ง Pull Request ข้าม fork ได้สำเร็จ และเข้าใจมารยาทพื้นฐานของการ Contribute โปรเจกต์ Open Source ครั้งแรกในชีวิต

---

## สารบัญของ Part นี้

- Step 231: Fork คืออะไร ต่างจาก Clone อย่างไร
- Step 232: ขั้นตอน Fork Repository แล้ว Clone มาทำงานที่เครื่อง
- Step 233: การตั้งค่า Upstream Remote เพื่อติดตามการอัปเดตจาก Repo ต้นฉบับ
- Step 234: Sync Fork กับ Upstream (Fetch, Merge/Rebase เข้า Main ของ Fork)
- Step 235: สร้าง Branch สำหรับ Contribute แล้วส่ง PR ข้าม Fork
- Step 236: การอ่าน CONTRIBUTING.md ก่อน Contribute
- Step 237: มารยาทการ Contribute โปรเจกต์ Open Source ครั้งแรก
- Step 238: การหาโปรเจกต์ที่เหมาะกับมือใหม่ (good first issue, help wanted)
- Step 239: สิ่งที่ควรทำและไม่ควรทำเมื่อ PR ถูก Reject หรือขอให้แก้ไข
- Step 240: แบบฝึกหัด — Fork โปรเจกต์ตัวอย่าง แก้ไขเล็กน้อย และส่ง PR ข้าม Fork ให้สำเร็จครบวงจร

---

## Step 231: Fork คืออะไร ต่างจาก Clone อย่างไร

ใน Part ก่อน ๆ เราคุ้นเคยกับคำว่า **Clone** มาแล้ว — การคัดลอก repository ทั้งหมด (พร้อมประวัติ) จาก remote มาไว้ที่เครื่อง local ของเรา แต่ Clone เพียงอย่างเดียวมีข้อจำกัดสำคัญข้อหนึ่ง: **ถ้าคุณไม่มีสิทธิ์ write เข้า repository นั้น (ไม่ได้เป็นเจ้าของหรือไม่ได้ถูกเชิญเป็น collaborator) คุณจะ push การเปลี่ยนแปลงกลับขึ้นไปไม่ได้เลย**

นี่คือปัญหาคลาสสิกที่เกิดขึ้นทุกครั้งที่คุณอยากช่วยแก้บั๊กหรือเพิ่มฟีเจอร์ให้โปรเจกต์ Open Source ของคนอื่นที่คุณไม่ได้เป็นเจ้าของ — คุณ clone repo เขามาแก้ไขในเครื่องได้สบาย แต่พอจะ `git push` กลับ GitHub จะตอบกลับมาด้วย error ประมาณนี้:

```
remote: Permission to torvalds/linux.git denied to yourusername.
fatal: unable to access 'https://github.com/torvalds/linux.git/': The requested URL returned error: 403
```

นี่คือจุดที่ **Fork** เข้ามาแก้ปัญหา

### Fork คืออะไร

> **Fork คือการสร้าง "สำเนาเต็ม" ของ repository ทั้งหมด (พร้อมประวัติทั้งหมด, branch ทั้งหมด) ไว้ใน**บัญชี GitHub ของคุณเอง**โดยที่ GitHub ยังจำได้ว่า repo นี้ fork มาจากที่ไหน (เก็บความสัมพันธ์ไว้)**

พูดง่าย ๆ Fork คือการกด "Copy" repository จากบัญชีคนอื่น มาเป็นของคุณเองบน GitHub — คุณเป็นเจ้าของ repo ที่ fork มานี้เต็มตัว มีสิทธิ์ push, สร้าง branch, ลบ branch, ตั้งค่าทุกอย่างได้อย่างอิสระ **โดยไม่ไปกระทบ repo ต้นฉบับเลยแม้แต่นิดเดียว**

### ตัวอย่างการมองเห็น Fork บน GitHub

สมมติว่า repo ต้นฉบับคือ `octocat/Hello-World` และคุณชื่อผู้ใช้ `yourusername` เมื่อคุณกด Fork บนหน้าเว็บของ repo นั้น GitHub จะสร้าง repo ใหม่ให้คุณที่:

```
https://github.com/yourusername/Hello-World
```

และในหน้าเว็บของ repo ที่ fork มา คุณจะเห็นข้อความกำกับไว้ใต้ชื่อ repo ว่า:

```
yourusername/Hello-World
forked from octocat/Hello-World
```

ข้อความนี้สำคัญมาก เพราะ GitHub ใช้ความสัมพันธ์นี้ในการทำหลายอย่างให้คุณอัตโนมัติ เช่น การเปิด Pull Request ข้าม repository (จาก fork กลับไปยัง repo ต้นฉบับ) ทำได้ในไม่กี่คลิก ซึ่งเราจะเรียนใน Step 235

### Fork vs Clone: ตารางเปรียบเทียบให้เห็นภาพชัด

| หัวข้อ | Clone | Fork |
|---|---|---|
| ทำงานที่ไหน | จาก remote (GitHub) → เครื่อง local ของคุณ | จาก repo คนอื่นบน GitHub → repo ใหม่ในบัญชี GitHub ของคุณเอง |
| ผลลัพธ์ | โฟลเดอร์บนเครื่องคุณ (working directory + `.git`) | Repository ใหม่บน GitHub ที่คุณเป็นเจ้าของ |
| คำสั่งที่ใช้ | `git clone <url>` | กดปุ่ม "Fork" บนหน้าเว็บ GitHub (หรือใช้ `gh repo fork`) |
| สิทธิ์ในการ push | ต้องมีสิทธิ์ write ใน repo ต้นฉบับ | เป็นเจ้าของเต็มตัว push ได้อิสระ |
| กระทบ repo ต้นฉบับไหม | ไม่กระทบ (แค่ copy มาที่เครื่อง) | ไม่กระทบเช่นกัน (เป็นแค่การสร้างสำเนาใหม่) |
| ใช้เมื่อไหร่ | เมื่อคุณเป็นเจ้าของหรือ collaborator ของ repo อยู่แล้ว | เมื่อคุณ**ไม่มี**สิทธิ์ write ใน repo และอยากช่วย contribute |
| ทำงานที่ไหน (protocol) | เกิดขึ้นระหว่างเครื่องคุณกับ server (ผ่าน Git protocol) | เกิดขึ้นทั้งหมดบน server ของ GitHub (server-side copy) |

### ทำไมต้องใช้ทั้งสองอย่างร่วมกัน

ประเด็นสำคัญที่มือใหม่มักสับสนคือ **Fork และ Clone ไม่ใช่สิ่งที่ใช้แทนกันได้ แต่เป็นสองขั้นตอนที่ทำงานร่วมกัน** เวลาคุณจะ contribute ให้โปรเจกต์ Open Source ที่คุณไม่ได้เป็นเจ้าของ ลำดับที่ถูกต้องคือ:

```
1. Fork repo ต้นฉบับ (บน GitHub)  →  ได้ repo สำเนาในบัญชีคุณเอง
2. Clone repo ที่ fork มา (จากบัญชีคุณ)  →  ได้โฟลเดอร์บนเครื่อง local
3. แก้ไขโค้ด, commit, push ขึ้น fork ของคุณ (ซึ่งคุณมีสิทธิ์เต็ม)
4. เปิด Pull Request จาก fork ของคุณ ไปยัง repo ต้นฉบับ
```

แผนภาพรวมของความสัมพันธ์ทั้งสามฝ่าย:

```
┌─────────────────────────┐
│   octocat/Hello-World    │  ← Repo ต้นฉบับ (Upstream)
│   (คุณไม่มีสิทธิ์ push)     │
└────────────▲─────────────┘
             │ Pull Request (Step 235)
             │
┌─────────────────────────┐
│ yourusername/Hello-World │  ← Fork ของคุณ (Origin)
│   (คุณเป็นเจ้าของเต็มตัว)   │
└────────────▲─────────────┘
             │ git push (คุณมีสิทธิ์)
             │
┌─────────────────────────┐
│  เครื่อง local ของคุณ       │  ← ได้จาก git clone
│  (working directory)     │
└──────────────────────────┘
```

ในหัวข้อ Step ถัดไป เราจะลงมือทำจริงตั้งแต่การกด Fork จนถึงการ clone มาทำงานที่เครื่อง

---

## Step 232: ขั้นตอน Fork Repository แล้ว Clone มาทำงานที่เครื่อง

มาลงมือทำทีละขั้นตอนกัน สมมติว่าคุณอยากช่วย contribute ให้โปรเจกต์ตัวอย่างชื่อ `some-org/awesome-project` (ในแบบฝึกหัด Step 240 เราจะใช้ repo จริงที่ปลอดภัยสำหรับมือใหม่)

### 232.1 ขั้นตอนที่ 1: กด Fork บนหน้าเว็บ GitHub

1. เปิดหน้า repository ต้นฉบับ เช่น `https://github.com/some-org/awesome-project`
2. มองไปที่มุมขวาบนของหน้า จะเห็นปุ่ม **Fork** พร้อมตัวเลขจำนวนคนที่ fork ไปแล้ว
3. กดปุ่ม **Fork**
4. GitHub จะพาไปหน้า "Create a new fork" ให้คุณตั้งค่าดังนี้:
   - **Owner** — เลือกบัญชีของคุณเอง (หรือ organization ที่คุณเป็นสมาชิก ถ้าต้องการ fork เข้าไปที่นั่น)
   - **Repository name** — ค่าเริ่มต้นจะใช้ชื่อเดียวกับต้นฉบับ (แนะนำให้คงชื่อเดิมไว้ เพื่อไม่ให้สับสน)
   - **Description** — ดึงมาจากต้นฉบับอัตโนมัติ แก้ไขได้ถ้าต้องการ
   - **Copy the `main` branch only** — ค่าเริ่มต้นเปิดไว้ (✓) หมายความว่า Fork จะดึงมาแค่ branch หลัก (เช่น `main`) เท่านั้น ไม่ดึง branch อื่น ๆ ทั้งหมดของต้นฉบับมาด้วย ถ้าคุณต้องการ branch อื่นด้วย ให้ปิด checkbox นี้
5. กดปุ่ม **Create fork**

หลังจากนั้นไม่กี่วินาที GitHub จะพาคุณไปยังหน้า repo ใหม่ของคุณที่ `https://github.com/yourusername/awesome-project` ซึ่งมีโค้ดและประวัติทั้งหมดเหมือนต้นฉบับทุกประการ ณ ขณะที่ fork

### 232.2 ขั้นตอนที่ 2: Clone Fork มาที่เครื่อง

**สำคัญมาก:** เวลา clone ให้ clone จาก **URL ของ fork ในบัญชีคุณเอง** ไม่ใช่จาก repo ต้นฉบับ

```bash
git clone https://github.com/yourusername/awesome-project.git
cd awesome-project
```

หรือถ้าตั้งค่า SSH key ไว้แล้ว (ตามที่เรียนใน Part ก่อนหน้า):

```bash
git clone git@github.com:yourusername/awesome-project.git
cd awesome-project
```

### 232.3 ตรวจสอบ remote ที่ได้มา

ทันทีหลัง clone ให้ตรวจสอบว่า remote `origin` ชี้ไปที่ fork ของคุณจริง ๆ:

```bash
git remote -v
```

ผลลัพธ์ที่ควรเห็น:

```
origin  https://github.com/yourusername/awesome-project.git (fetch)
origin  https://github.com/yourusername/awesome-project.git (push)
```

สังเกตว่า `origin` ตอนนี้ชี้ไปที่ **fork ของคุณ** ไม่ใช่ repo ต้นฉบับ นี่คือเหตุผลที่คุณ push ได้อย่างอิสระ — เพราะ `origin` คือ repo ที่คุณเป็นเจ้าของ

### 232.4 ทำไมต้อง Clone จาก Fork ไม่ใช่จากต้นฉบับ

ถ้าคุณ clone repo ต้นฉบับโดยตรง (`git clone https://github.com/some-org/awesome-project.git`) แล้วพยายาม push จะได้ error 403 Permission denied ทันที เพราะ `origin` ในกรณีนั้นจะชี้ไปที่ repo ต้นฉบับซึ่งคุณไม่มีสิทธิ์เขียน

**สรุปหลักการ:** เมื่อไหร่ที่คุณไม่ใช่เจ้าของหรือ collaborator ของ repo ให้ **Fork ก่อนเสมอ แล้วค่อย Clone จาก fork ของตัวเอง**

### 232.5 ตารางสรุปคำสั่งของ Step นี้

| ขั้นตอน | ทำที่ไหน | คำสั่ง/การกระทำ |
|---|---|---|
| 1. Fork | หน้าเว็บ GitHub (repo ต้นฉบับ) | กดปุ่ม **Fork** → **Create fork** |
| 2. Clone | Terminal บนเครื่อง | `git clone <URL ของ fork>` |
| 3. เข้าโฟลเดอร์ | Terminal | `cd <ชื่อโปรเจกต์>` |
| 4. ตรวจสอบ remote | Terminal | `git remote -v` |

ตอนนี้คุณมี repo ในเครื่องพร้อมแก้ไขแล้ว แต่ยังขาดอีกสิ่งสำคัญหนึ่งอย่าง คือการเชื่อมต่อกับ repo ต้นฉบับเพื่อติดตามความเปลี่ยนแปลงใหม่ ๆ ที่เกิดขึ้นหลังจากคุณ fork มา ซึ่งเราจะเรียนใน Step ถัดไป

---

## Step 233: การตั้งค่า Upstream Remote เพื่อติดตามการอัปเดตจาก Repo ต้นฉบับ

### ปัญหาที่เกิดขึ้นถ้าไม่ตั้งค่า Upstream

ลองนึกภาพสถานการณ์นี้: คุณ fork repo `some-org/awesome-project` ไว้เมื่อ 2 เดือนก่อน ระหว่างนั้นทีมต้นฉบับ push commit ใหม่เข้า `main` ไปแล้วกว่า 50 commit แต่ fork ของคุณ (`origin`) ยังคงค้างอยู่ที่จุดเดิมตอนที่คุณ fork มา — GitHub **ไม่ได้ sync fork ให้อัตโนมัติ**

ถ้าคุณแก้ไขโค้ดบนฐานเก่านี้แล้วส่ง Pull Request กลับไป มีโอกาสสูงมากที่จะเกิด **conflict** จำนวนมาก เพราะโค้ดที่คุณอ้างอิงล้าสมัยไปแล้ว นี่คือเหตุผลที่ต้องตั้งค่า **upstream remote** ไว้ตั้งแต่ต้น

### Upstream คืออะไร

> **Upstream** คือชื่อเรียกตามธรรมเนียม (convention) ที่ใช้อ้างถึง **remote ที่ชี้ไปยัง repo ต้นฉบับ** (ตัวที่คุณ fork มา) เพื่อแยกให้ชัดเจนจาก `origin` ซึ่งชี้ไปยัง fork ของคุณเอง

ธรรมเนียมนี้ไม่ใช่กฎบังคับของ Git แต่เป็นมาตรฐานที่ทั้งชุมชน Open Source ทั่วโลกใช้ตรงกัน คุณจะเจอคำว่า `upstream` ในเอกสาร CONTRIBUTING.md ของแทบทุกโปรเจกต์

### คำสั่งเพิ่ม Upstream Remote

```bash
git remote add upstream https://github.com/some-org/awesome-project.git
```

ตรวจสอบว่าเพิ่มสำเร็จ:

```bash
git remote -v
```

ผลลัพธ์ที่ควรเห็น:

```
origin    https://github.com/yourusername/awesome-project.git (fetch)
origin    https://github.com/yourusername/awesome-project.git (push)
upstream  https://github.com/some-org/awesome-project.git (fetch)
upstream  https://github.com/some-org/awesome-project.git (push)
```

ตอนนี้ repository ของคุณมี **2 remote** พร้อมกัน:

| Remote | ชี้ไปที่ | ใช้ทำอะไร |
|---|---|---|
| `origin` | Fork ของคุณเอง | Push branch ที่คุณแก้ไขขึ้นไป, สร้าง branch ทดลอง |
| `upstream` | Repo ต้นฉบับ | Fetch/Pull การเปลี่ยนแปลงล่าสุดจากทีมเจ้าของโปรเจกต์ |

### ควร Push ขึ้น Upstream ได้ไหม

**ไม่ได้ และไม่ควรพยายามด้วย** เพราะคุณไม่มีสิทธิ์ write ใน repo ต้นฉบับ (เว้นแต่คุณจะถูกเชิญเป็น collaborator ในภายหลัง) `upstream` มีไว้เพื่อ **fetch/pull เท่านั้น** ไม่ใช่เพื่อ push

หากคุณลอง push ไปยัง `upstream` โดยไม่มีสิทธิ์ จะได้ error แบบเดียวกับที่เห็นใน Step 231:

```
remote: Permission to some-org/awesome-project.git denied to yourusername.
fatal: unable to access '...' The requested URL returned error: 403
```

### กรณีตั้งชื่อ Remote ผิดหรืออยากแก้ URL

ถ้าพิมพ์ URL ผิดตอนเพิ่ม upstream สามารถแก้ไขได้ด้วย:

```bash
git remote set-url upstream https://github.com/some-org/awesome-project.git
```

หรือถ้าอยากลบแล้วเพิ่มใหม่:

```bash
git remote remove upstream
git remote add upstream https://github.com/some-org/awesome-project.git
```

### ทางลัดด้วย GitHub CLI

ถ้าคุณติดตั้ง [GitHub CLI](https://cli.github.com) (`gh`) ไว้แล้ว คำสั่งเดียวสามารถ fork + clone + ตั้งค่า upstream ให้อัตโนมัติได้เลย:

```bash
gh repo fork some-org/awesome-project --clone=true
```

คำสั่งนี้จะ:
1. Fork repo ต้นฉบับเข้าบัญชีของคุณบน GitHub
2. Clone fork นั้นมาที่เครื่อง
3. ตั้งค่า `origin` ให้ชี้ไปที่ fork ของคุณ
4. ตั้งค่า `upstream` ให้ชี้ไปที่ repo ต้นฉบับให้อัตโนมัติทันที

แม้ `gh` จะช่วยย่นเวลาได้มาก แต่การเข้าใจว่าเบื้องหลังมันทำอะไรบ้าง (ตามที่อธิบายในหัวข้อนี้) สำคัญกว่า เพราะเมื่อเกิดปัญหาคุณจะแก้ไขเองได้โดยไม่ต้องพึ่งเครื่องมือช่วยเสมอไป

### Checklist ก่อนไป Step ถัดไป

- [ ] เพิ่ม `upstream` remote ชี้ไปยัง repo ต้นฉบับสำเร็จ
- [ ] รัน `git remote -v` แล้วเห็นทั้ง `origin` (fork) และ `upstream` (ต้นฉบับ) ครบถ้วน
- [ ] เข้าใจว่า `upstream` ใช้สำหรับ fetch/pull เท่านั้น ไม่ใช้ push

---

## Step 234: Sync Fork กับ Upstream (Fetch Upstream, Merge/Rebase เข้า Main ของ Fork)

เมื่อมี `upstream` remote พร้อมแล้ว ขั้นตอนถัดไปคือการ **sync** หรือทำให้ fork ของคุณทันสมัยเท่ากับ repo ต้นฉบับอยู่เสมอ ควรทำเป็นประจำ **ทุกครั้งก่อนเริ่มงานใหม่**

### 234.1 ขั้นตอนที่ 1: Fetch ข้อมูลล่าสุดจาก Upstream

```bash
git fetch upstream
```

คำสั่งนี้จะดึงข้อมูล commit, branch ทั้งหมดจาก repo ต้นฉบับมาเก็บไว้ใน local repository ของคุณ ภายใต้ชื่อ `upstream/main` (หรือชื่อ branch หลักอื่น ๆ เช่น `upstream/master`) **แต่ยังไม่รวมเข้ากับ branch ที่คุณกำลังทำงานอยู่**

ผลลัพธ์ตัวอย่าง:

```
remote: Enumerating objects: 45, done.
remote: Counting objects: 100% (45/45), done.
remote: Compressing objects: 100% (30/30), done.
remote: Total 45 (delta 20), reused 10 (delta 5), pack-reused 0
Unpacking objects: 100% (45/45), 12.34 KiB | 500.00 KiB/s, done.
From https://github.com/some-org/awesome-project
 * [new branch]      main       -> upstream/main
```

### 234.2 ขั้นตอนที่ 2: สลับไปยัง Branch main ของ Fork ตัวเอง

```bash
git checkout main
```

### 234.3 ขั้นตอนที่ 3: รวมการเปลี่ยนแปลงเข้ากับ main ของคุณ

มีสองวิธีหลักที่ใช้กันทั่วไป — **merge** และ **rebase**

**วิธีที่ 1: Merge (ปลอดภัยกว่า เหมาะกับมือใหม่)**

```bash
git merge upstream/main
```

วิธีนี้จะสร้าง commit merge ใหม่ (ถ้ามีการแตกต่างกัน) และรวมประวัติทั้งสองฝั่งเข้าด้วยกันแบบตรงไปตรงมา เหมาะกับผู้เริ่มต้นเพราะเข้าใจง่ายและมีความเสี่ยงต่อการทำประวัติเสียหายต่ำที่สุด

**วิธีที่ 2: Rebase (ทำให้ประวัติสะอาดกว่า เหมาะกับผู้ที่คุ้นเคย Git แล้ว)**

```bash
git rebase upstream/main
```

วิธีนี้จะ "ยก" commit ของคุณไปวางต่อท้าย commit ล่าสุดจาก upstream ทำให้ประวัติเรียงเป็นเส้นตรง ไม่มี merge commit แทรก แต่ต้องระวังเรื่อง conflict ที่อาจซับซ้อนกว่า และ **ห้าม rebase branch ที่มีคน push ร่วมกับคุณอยู่แล้ว** (เพราะจะเขียนประวัติทับของเดิม) เรื่องนี้เราจะเจาะลึกเรื่อง rebase อย่างละเอียดใน **Part 42: Rebase ขั้นสูงและข้อควรระวัง**

สำหรับ Part นี้ เนื่องจากเรากำลังทำงานบน `main` ของ fork ที่ยังไม่มีคนอื่นร่วม push ด้วย ทั้ง merge และ rebase ปลอดภัยพอ ๆ กัน แต่แนะนำให้มือใหม่เริ่มจาก **merge** ก่อน

### 234.4 ขั้นตอนที่ 4: Push การอัปเดตกลับไปที่ Fork (origin) ของคุณ

หลังจาก merge/rebase เรียบร้อยแล้ว อย่าลืม push อัปเดต `main` ของ fork บน GitHub ให้ทันสมัยด้วย:

```bash
git push origin main
```

ถ้าคุณใช้ rebase และ branch `main` บน `origin` เคย push ไปก่อนหน้าแล้ว อาจต้องใช้:

```bash
git push origin main --force-with-lease
```

**ข้อควรระวัง:** ใช้ `--force-with-lease` แทน `--force` เสมอ เพราะมันจะตรวจสอบก่อนว่าไม่มีใครมา push ทับ branch นั้นระหว่างที่คุณไม่ได้ดู ถ้ามีคนอื่น push เข้าไปแล้วจริง คำสั่งจะปฏิเสธและเตือนคุณ แทนที่จะเขียนทับงานของคนอื่นไปเงียบ ๆ

### 234.5 สรุปเป็นขั้นตอนต่อเนื่องที่ควรทำก่อนเริ่มงานใหม่ทุกครั้ง

```bash
git checkout main
git fetch upstream
git merge upstream/main
git push origin main
```

สี่บรรทัดนี้ควรกลายเป็นความเคยชินของคุณทุกครั้งที่กลับมาทำงานกับ fork หลังจากผ่านไประยะหนึ่ง เพื่อให้แน่ใจว่าคุณเริ่มงานจากฐานโค้ดที่ทันสมัยที่สุดเสมอ

### 234.6 แผนภาพสรุปการไหลของข้อมูล

```
        ┌────────────────────────────┐
        │  some-org/awesome-project  │  ← Upstream (ต้นฉบับ)
        └──────────────┬─────────────┘
                        │ git fetch upstream
                        ▼
        ┌────────────────────────────┐
        │   เครื่อง local ของคุณ       │
        │   (upstream/main พร้อมแล้ว)  │
        └──────────────┬─────────────┘
                        │ git merge upstream/main
                        ▼
        ┌────────────────────────────┐
        │   local main (อัปเดตแล้ว)   │
        └──────────────┬─────────────┘
                        │ git push origin main
                        ▼
        ┌────────────────────────────┐
        │ yourusername/awesome-project│  ← Fork (origin) ทันสมัยแล้ว
        └────────────────────────────┘
```

### 234.7 คำถามที่พบบ่อย: ทำไมไม่ใช้ปุ่ม "Sync fork" บนหน้าเว็บ GitHub แทน

GitHub มีปุ่ม **Sync fork** อยู่บนหน้าเว็บของ repo ที่ fork มา ซึ่งทำหน้าที่คล้ายกัน (ดึงการเปลี่ยนแปลงจาก upstream มา merge เข้า fork โดยอัตโนมัติผ่านเว็บ ไม่ต้องเปิด terminal) ปุ่มนี้สะดวกมากสำหรับกรณีง่าย ๆ ที่ไม่มี conflict แต่มีข้อจำกัดคือ:

- ใช้ได้เฉพาะการ sync branch หลัก ไม่เหมาะกับสถานการณ์ซับซ้อน
- ถ้าเกิด conflict ปุ่มนี้จะ sync ไม่สำเร็จ และแนะนำให้กลับมาทำผ่าน command line แทน
- ไม่ยืดหยุ่นเท่าการควบคุมเองผ่าน `git fetch` / `git merge` / `git rebase`

หลักสูตรนี้แนะนำให้ฝึกทำผ่าน command line เป็นหลัก เพื่อให้เข้าใจกลไกเบื้องหลังอย่างแท้จริง ปุ่มบนเว็บเป็นเพียงทางลัดเสริมเท่านั้น

---

## Step 235: สร้าง Branch สำหรับ Contribute แล้วส่ง PR ข้าม Fork

ตอนนี้ fork ของคุณทันสมัยเท่ากับต้นฉบับแล้ว ถึงเวลาลงมือแก้ไขโค้ดจริง ๆ

### 235.1 หลักการสำคัญ: ห้ามแก้ไขบน main โดยตรง

ธรรมเนียมมาตรฐานของการ contribute Open Source (และการทำงานเป็นทีมทั่วไป) คือ **ห้าม commit งานใหม่ลงบน branch `main` โดยตรงเด็ดขาด** ให้สร้าง branch ใหม่เฉพาะสำหรับงานแต่ละชิ้นเสมอ เหตุผลคือ:

1. ทำให้ `main` ของ fork คุณสะอาด พร้อม sync จาก upstream ได้ทุกเมื่อโดยไม่มี conflict จากงานที่ยังไม่เสร็จ
2. คุณสามารถทำงานหลายอย่างพร้อมกันได้ (แต่ละอย่างแยก branch)
3. เมื่อ PR ถูก merge หรือถูกปิด คุณลบ branch นั้นทิ้งได้โดยไม่กระทบ `main`

### 235.2 สร้าง Branch ใหม่จาก main ที่อัปเดตล่าสุด

```bash
git checkout main
git checkout -b fix-typo-in-readme
```

หรือรวบเป็นคำสั่งเดียว:

```bash
git checkout -b fix-typo-in-readme main
```

**ควรตั้งชื่อ branch ให้สื่อความหมาย** เช่น:

| ประเภทงาน | ตัวอย่างชื่อ branch |
|---|---|
| แก้บั๊ก | `fix-null-pointer-in-login` |
| เพิ่มฟีเจอร์ | `add-dark-mode-toggle` |
| แก้เอกสาร | `docs-update-installation-guide` |
| แก้ typo เล็กน้อย | `fix-typo-in-readme` |

### 235.3 แก้ไขไฟล์ Commit และ Push ขึ้น Fork

```bash
# แก้ไขไฟล์ด้วย editor ตามปกติ
git add README.md
git commit -m "fix: correct typo in installation section"
git push origin fix-typo-in-readme
```

สังเกตว่าเรา push ไปที่ `origin` (fork ของเรา) ไม่ใช่ `upstream` เพราะเราไม่มีสิทธิ์ push เข้า upstream

### 235.4 เปิด Pull Request ข้าม Fork (Cross-Repository Pull Request)

หลัง push แล้ว GitHub มักจะแสดงข้อความแจ้งเตือนอัตโนมัติบนหน้าเว็บของ repo ต้นฉบับหรือ fork ของคุณว่า:

```
fix-typo-in-readme had recent pushes less than a minute ago
[Compare & pull request]
```

กดปุ่ม **Compare & pull request** หรือทำตามขั้นตอนด้วยตัวเองดังนี้:

1. ไปที่หน้า repo **ต้นฉบับ** (`some-org/awesome-project`) ไม่ใช่หน้า fork ของคุณ
2. คลิกแท็บ **Pull requests** → **New pull request**
3. GitHub จะแสดงตัวเลือก **compare across forks** — คลิกเพื่อเปิดใช้งาน
4. ตั้งค่าฝั่งซ้ายและขวาให้ถูกต้อง:

```
base repository: some-org/awesome-project   base: main
head repository: yourusername/awesome-project   compare: fix-typo-in-readme
```

**คำอธิบายศัพท์ที่มักสับสน:**

| ศัพท์ | ความหมาย |
|---|---|
| **base repository** | Repo ที่จะรับการเปลี่ยนแปลงเข้าไป (ปกติคือ repo ต้นฉบับ) |
| **base branch** | Branch ปลายทางใน base repository (มักเป็น `main`) |
| **head repository** | Repo ที่มีการเปลี่ยนแปลงของคุณอยู่ (fork ของคุณ) |
| **compare branch** (head branch) | Branch ที่มีงานของคุณ (`fix-typo-in-readme`) |

5. ตรวจสอบ diff ที่ GitHub แสดงให้ดูว่าตรงกับที่ตั้งใจไว้จริง
6. ใส่หัวข้อ (Title) และรายละเอียด (Description) ของ PR ให้ชัดเจน
7. กด **Create pull request**

### 235.5 ตัวอย่างข้อความ PR ที่ดี

```markdown
## What this PR does

Fixes a typo in the installation section of README.md
where "Instalation" should be "Installation".

## Why

Small documentation fix to improve clarity for new contributors.

## Checklist
- [x] I have read CONTRIBUTING.md
- [x] This change does not require tests (docs-only change)
```

### 235.6 หลังส่ง PR แล้วเกิดอะไรขึ้น

- Maintainer ของ repo ต้นฉบับจะได้รับแจ้งเตือนว่ามี PR ใหม่รอ review
- อาจมี CI/CD (GitHub Actions) รันตรวจสอบอัตโนมัติทันที (เราจะเรียนเรื่องนี้ละเอียดใน **Part 67: GitHub Actions เบื้องต้น**)
- Maintainer อาจ comment ขอให้แก้ไขเพิ่มเติม, approve และ merge, หรือปิด PR โดยไม่ merge

### 235.7 การอัปเดต PR หลังส่งไปแล้ว

ถ้า maintainer ขอให้แก้ไขเพิ่มเติม คุณไม่ต้องเปิด PR ใหม่ — แค่ commit เพิ่มบน branch เดิมแล้ว push ขึ้น `origin` อีกครั้ง PR ที่เปิดอยู่จะอัปเดตเนื้อหาให้อัตโนมัติ:

```bash
# แก้ไขเพิ่มเติมตามที่ maintainer ขอ
git add .
git commit -m "address review feedback: rephrase sentence"
git push origin fix-typo-in-readme
```

PR บนหน้าเว็บจะแสดง commit ใหม่นี้เพิ่มเข้ามาโดยอัตโนมัติทันที ไม่ต้องทำอะไรเพิ่มอีก

---

## Step 236: การอ่าน CONTRIBUTING.md ก่อน Contribute

### CONTRIBUTING.md คืออะไร

หลายโปรเจกต์ Open Source ที่จัดการอย่างเป็นระบบจะมีไฟล์ชื่อ **`CONTRIBUTING.md`** อยู่ที่ root ของ repository (หรือในโฟลเดอร์ `.github/`) ไฟล์นี้คือ **"คู่มือกฎกติกา" สำหรับผู้ที่ต้องการ contribute** เขียนขึ้นโดยเจ้าของหรือทีม maintainer ของโปรเจกต์นั้นโดยเฉพาะ

### ทำไมต้องอ่านก่อนเสมอ

การข้ามขั้นตอนนี้เป็นสาเหตุอันดับต้น ๆ ที่ทำให้ PR ของมือใหม่ถูก reject หรือถูกขอให้แก้ไขจำนวนมาก เพราะแต่ละโปรเจกต์มีกฎที่แตกต่างกัน เช่น:

- รูปแบบของ commit message ที่ต้องใช้ (บางโปรเจกต์บังคับใช้ [Conventional Commits](https://www.conventionalcommits.org))
- ต้องรันคำสั่งอะไรก่อน commit เช่น linter, formatter, test suite
- ต้อง sign-off commit ด้วย `git commit -s` หรือไม่ (Developer Certificate of Origin — DCO)
- ต้องกรอกฟอร์ม Contributor License Agreement (CLA) หรือไม่ ก่อน PR จะถูกพิจารณา
- ขนาดของ PR ที่ยอมรับ (บางโปรเจกต์ขอให้ PR เล็ก ๆ ทีละเรื่อง ไม่รวมหลายฟีเจอร์ในครั้งเดียว)
- Branch ที่ต้องใช้เป็นฐาน (บางโปรเจกต์ใช้ `develop` แทน `main` เป็น base branch สำหรับ PR)

### ตัวอย่างเนื้อหาที่มักพบใน CONTRIBUTING.md

```markdown
# Contributing to Awesome Project

## Before you start
- Search existing issues before opening a new one
- Discuss large changes in an issue first before submitting a PR

## Development setup
```bash
npm install
npm run test
```

## Commit message format
We use Conventional Commits:
  feat: add new feature
  fix: correct a bug
  docs: documentation only changes

## Pull request process
1. Fork the repo and create your branch from `main`
2. Run `npm run lint` and `npm test` before submitting
3. Make sure your PR description explains the "why", not just the "what"
```

### ไฟล์อื่น ๆ ที่ควรตรวจสอบควบคู่กัน

| ไฟล์ | มีไว้ทำอะไร |
|---|---|
| `CONTRIBUTING.md` | กฎกติกาการ contribute |
| `CODE_OF_CONDUCT.md` | มาตรฐานพฤติกรรมที่ยอมรับได้ในชุมชน |
| `.github/PULL_REQUEST_TEMPLATE.md` | Template ที่ต้องกรอกเวลาสร้าง PR |
| `.github/ISSUE_TEMPLATE/` | Template สำหรับเปิด issue แต่ละประเภท |
| `SECURITY.md` | วิธีรายงานช่องโหว่ความปลอดภัย (ไม่ควรเปิดเป็น public issue) |

หากโปรเจกต์มี `.github/PULL_REQUEST_TEMPLATE.md` เมื่อคุณกด "Create pull request" กล่อง Description จะถูกเติมเนื้อหาจาก template นี้มาให้อัตโนมัติ ให้กรอกข้อมูลตามหัวข้อที่ template กำหนดให้ครบถ้วน อย่าลบทิ้งแล้วเขียนใหม่ตามใจตัวเอง

### หมายเหตุสำคัญ

Part นี้แนะนำเพียงหลักการกว้าง ๆ ของ CONTRIBUTING.md เท่านั้น เราจะ **เจาะลึกการอ่านและปฏิบัติตาม CONTRIBUTING.md แบบละเอียดมาก พร้อมกรณีศึกษาจากโปรเจกต์ดังระดับโลก** ใน **Part 87: การอ่านและปฏิบัติตาม Contributing Guidelines ขั้นสูง** ซึ่งจะครอบคลุมเรื่อง DCO, CLA, และมาตรฐาน commit message แบบละเอียดยิ่งขึ้น

---

## Step 237: มารยาทการ Contribute โปรเจกต์ Open Source ครั้งแรก

การ contribute Open Source ไม่ใช่แค่เรื่องเทคนิคของ Git เท่านั้น แต่ยังเป็นเรื่องของ **การอยู่ร่วมกันในชุมชน** ด้วย ต่อไปนี้คือมารยาทพื้นฐานที่มือใหม่ควรรู้

### 237.1 อ่านกฎก่อนลงมือเสมอ

ตามที่กล่าวใน Step 236 — อ่าน `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md` และ README ให้ครบก่อนเริ่มงานทุกครั้ง การข้ามขั้นตอนนี้แล้วส่ง PR ที่ผิดรูปแบบไปเลยมักทำให้ maintainer รู้สึกว่าคุณไม่ให้ความเคารพกับเวลาของพวกเขา

### 237.2 อย่าทำ PR ใหญ่เกินความจำเป็น

มือใหม่หลายคนอยากช่วยเหลือให้มากที่สุดในครั้งเดียว จึงส่ง PR ที่แก้ไขหลายสิบไฟล์ ปรับ style ทั้งโปรเจกต์ เพิ่มฟีเจอร์ใหญ่ พร้อมกับแก้บั๊กเล็ก ๆ ไปในคราวเดียว — นี่คือสิ่งที่**ไม่ควรทำ**

**เหตุผล:**

- PR ที่ใหญ่ review ยากมาก maintainer ต้องใช้เวลามากในการทำความเข้าใจการเปลี่ยนแปลงทั้งหมด
- ถ้ามีปัญหาแค่จุดเดียวใน PR ใหญ่ ทั้ง PR อาจถูกปฏิเสธไปทั้งหมด แม้ส่วนอื่นจะดีก็ตาม
- ยากต่อการ revert ถ้าพบปัญหาภายหลัง เพราะแก้หลายเรื่องปนกัน

**แนวทางที่ดี:** หนึ่ง PR ควรทำเรื่องเดียว (Single Responsibility) เช่น PR หนึ่งแก้บั๊กเดียว หรือเพิ่มฟีเจอร์เดียว ถ้ามีหลายเรื่องที่อยากช่วย ให้แยกเป็นหลาย PR ทีละเรื่อง

### 237.3 เริ่มจากการเปิด Issue หรือคอมเมนต์ก่อน สำหรับงานใหญ่

ถ้างานที่คุณอยากทำมีขนาดใหญ่หรือเปลี่ยนแปลงสถาปัตยกรรมสำคัญ **ควรเปิด issue เพื่อพูดคุยแนวทางกับ maintainer ก่อน** แทนที่จะลงมือเขียนโค้ดหลายร้อยบรรทัดแล้วส่ง PR ไปเลยโดยไม่มีการพูดคุย เพราะ:

- Maintainer อาจมีแผนอื่นอยู่แล้วสำหรับส่วนนั้น
- อาจมีเหตุผลที่โค้ดปัจจุบันถูกออกแบบมาแบบนั้น ซึ่งคุณอาจไม่ทราบ
- ป้องกันไม่ให้คุณเสียเวลาทำงานที่สุดท้ายไม่ถูก merge

### 237.4 สื่อสารอย่างสุภาพและชัดเจนเสมอ

- เขียนข้อความ PR/issue เป็นภาษาอังกฤษ (ถ้าโปรเจกต์เป็นสากล) อย่างชัดเจน กระชับ ตรงประเด็น
- ไม่ใช้ถ้อยคำก้าวร้าวหรือตำหนิ แม้ maintainer จะตอบช้าหรือปฏิเสธ PR ของคุณ
- ให้เครดิตและขอบคุณ maintainer ที่สละเวลามารีวิวงานของคุณ — อย่าลืมว่าโปรเจกต์ Open Source ส่วนใหญ่ดูแลโดยอาสาสมัครที่ไม่ได้รับค่าตอบแทน

### 237.5 อดทนรอการตอบกลับ

Maintainer จำนวนมากดูแลโปรเจกต์ในเวลาว่างของตัวเอง การตอบกลับอาจใช้เวลาหลายวันถึงหลายสัปดาห์ **อย่า** ไป comment ทวงถามซ้ำ ๆ ในเวลาสั้น ๆ เช่นทุกวัน — ถ้าผ่านไปนานพอสมควร (เช่น 2-3 สัปดาห์) โดยไม่มีการตอบรับเลย การ comment ถามอย่างสุภาพครั้งเดียวถือว่าเหมาะสม

### 237.6 อย่าลืมทดสอบก่อนส่ง

ก่อนส่ง PR ให้รันชุดทดสอบ (test suite) ของโปรเจกต์เองก่อนเสมอ (ตามที่ CONTRIBUTING.md ระบุ) ไม่ควรพึ่ง CI ของ GitHub เพียงอย่างเดียวในการหาข้อผิดพลาด เพราะการปล่อยให้ CI fail ซ้ำ ๆ หลายรอบสร้างภาระให้ maintainer ต้องคอยติดตาม

### 237.7 สรุปมารยาทเป็นตาราง

| ควรทำ | ไม่ควรทำ |
|---|---|
| อ่านกฎก่อนเริ่มงานเสมอ | ข้ามการอ่าน CONTRIBUTING.md แล้วส่ง PR ตามใจตัวเอง |
| ทำ PR เล็ก โฟกัสเรื่องเดียว | รวมหลายฟีเจอร์/การแก้ไขไว้ใน PR เดียว |
| เปิด issue คุยก่อนสำหรับงานใหญ่ | เขียนโค้ดจำนวนมากไปก่อนโดยไม่ปรึกษาใคร |
| สื่อสารสุภาพ ให้เครดิตผู้รีวิว | ใช้ถ้อยคำก้าวร้าวเมื่อถูกปฏิเสธ |
| ทดสอบโค้ดก่อนส่งเสมอ | ปล่อยให้ CI เป็นตัวหาบั๊กแทนตัวเอง |
| รอคอยการตอบกลับอย่างอดทน | Comment ทวงถามซ้ำ ๆ ถี่เกินไป |

---

## Step 238: การหาโปรเจกต์ที่เหมาะกับมือใหม่ (good first issue, help wanted)

หนึ่งในอุปสรรคใหญ่ที่สุดของมือใหม่ที่อยากเริ่ม contribute Open Source คือ **"ไม่รู้จะเริ่มจากโปรเจกต์ไหนดี"** โชคดีที่ GitHub มีระบบ **Label (ป้ายกำกับ)** บน Issue ที่ช่วยแก้ปัญหานี้โดยเฉพาะ

### 238.1 Label `good first issue`

Label นี้ถูกใช้กันอย่างแพร่หลายทั่วทั้งวงการ Open Source เพื่อบอกว่า:

> **Issue นี้เหมาะสำหรับผู้ที่เพิ่งเริ่มต้น contribute โปรเจกต์นี้เป็นครั้งแรก มีขอบเขตงานเล็ก ชัดเจน และไม่ต้องเข้าใจสถาปัตยกรรมทั้งหมดของโปรเจกต์**

ตัวอย่าง issue ที่มักติด label นี้:

- แก้ typo ในเอกสาร
- เพิ่ม unit test ที่ขาดหายไปสำหรับฟังก์ชันเดียว
- ปรับปรุงข้อความ error message ให้เข้าใจง่ายขึ้น
- เพิ่ม validation เล็กน้อยในฟอร์ม

### 238.2 Label `help wanted`

> **Issue นี้ทีม maintainer เปิดรับความช่วยเหลือจากภายนอกอย่างชัดเจน** อาจจะเป็นงานที่ทีมไม่มีเวลาทำเอง หรือรู้ปัญหาแต่ยังไม่ได้แก้

Label นี้มักมีขอบเขตงานหลากหลายกว่า `good first issue` บางอันอาจเหมาะกับมือใหม่ บางอันอาจต้องมีประสบการณ์มากกว่า ควรอ่านรายละเอียดใน issue อย่างละเอียดก่อนตัดสินใจ

### 238.3 วิธีค้นหา Issue เหล่านี้ผ่าน GitHub Search

GitHub มีหน้าเว็บพิเศษสำหรับค้นหา issue ข้ามทุก repository ทั่วทั้งแพลตฟอร์มที่ `https://github.com/issues` โดยสามารถใช้ search qualifier ได้ เช่น:

```
is:issue is:open label:"good first issue" language:javascript
```

หรือค้นเฉพาะภาษาที่ถนัด:

```
is:issue is:open label:"good first issue" language:python archived:false
```

**คำอธิบาย qualifier ที่ใช้บ่อย:**

| Qualifier | ความหมาย |
|---|---|
| `is:issue` | ค้นเฉพาะ issue (ไม่รวม pull request) |
| `is:open` | เฉพาะ issue ที่ยังเปิดอยู่ ยังไม่ถูกปิด |
| `label:"good first issue"` | เฉพาะ issue ที่ติด label นี้ |
| `language:javascript` | เฉพาะ repo ที่ภาษาหลักคือ JavaScript |
| `no:assignee` | ยังไม่มีใครรับงานนี้ไป |
| `sort:updated-desc` | เรียงตาม issue ที่มีการอัปเดตล่าสุดก่อน |

### 238.4 เว็บไซต์ช่วยค้นหาที่นิยม

นอกจาก GitHub Search โดยตรงแล้ว ยังมีเว็บไซต์ third-party ที่รวบรวม good first issue จากทั่วทั้ง GitHub มาไว้ในที่เดียว ทำให้ค้นหาง่ายขึ้น เช่น:

- **goodfirstissue.dev** — ค้นหาตามภาษาโปรแกรมมิ่งที่สนใจ
- **firsttimersonly.com** — เน้นโปรเจกต์ที่เป็นมิตรกับผู้เริ่ม contribute เป็นครั้งแรกโดยเฉพาะ
- **up-for-grabs.net** — รวบรวมโปรเจกต์ที่เปิดรับความช่วยเหลือจากหลากหลายภาษา

### 238.5 สัญญาณอื่น ๆ ที่บอกว่าโปรเจกต์เป็นมิตรกับมือใหม่

นอกจาก label แล้ว ให้สังเกตสัญญาณเหล่านี้ก่อนเลือกโปรเจกต์ที่จะ contribute:

1. **มี CONTRIBUTING.md ที่เขียนละเอียดและอัปเดต** — แสดงว่าทีมใส่ใจผู้มาใหม่
2. **Maintainer ตอบ issue/PR เก่า ๆ อย่างสม่ำเสมอ** — เช็คได้จาก tab Issues/Pull requests ว่ามี activity ล่าสุดเมื่อไหร่
3. **มี Code of Conduct ชัดเจน** — บ่งบอกถึงชุมชนที่มีมาตรฐานพฤติกรรมที่ดี
4. **README อธิบายวิธี setup โปรเจกต์อย่างชัดเจน** — ถ้า setup โปรเจกต์เองไม่ได้ตั้งแต่แรก จะแก้บั๊กได้ยากมาก
5. **จำนวน open issue ที่ค้างไม่มากเกินไปเมื่อเทียบกับจำนวน contributor** — บ่งบอกว่าทีมตามงานทัน

### 238.6 คำแนะนำสำหรับการเลือกโปรเจกต์แรกในชีวิต

สำหรับผู้เริ่มต้นครั้งแรก แนะนำให้เลือกโปรเจกต์ที่:

- ใช้ภาษาโปรแกรมมิ่งที่คุณคุ้นเคยอยู่แล้ว
- มีขนาดไม่ใหญ่เกินไป (setup ง่าย เข้าใจโครงสร้างได้เร็ว)
- มี issue label `good first issue` เปิดอยู่จำนวนหนึ่ง
- เป็นโปรเจกต์ที่คุณใช้งานจริงอยู่แล้ว (เช่น library ที่คุณใช้ในงานประจำ) เพราะจะเข้าใจ context ของโค้ดได้เร็วกว่าโปรเจกต์ที่ไม่เคยใช้เลย

---

## Step 239: สิ่งที่ควรทำและไม่ควรทำเมื่อ PR ถูก Reject หรือขอให้แก้ไข

การถูกขอให้แก้ไขหรือแม้กระทั่งถูกปิด PR โดยไม่ merge เป็นเรื่องปกติมากในโลก Open Source แม้แต่นักพัฒนาระดับ senior ก็เจอเหตุการณ์นี้อยู่เสมอ สิ่งสำคัญคือการรับมือกับมันอย่างเหมาะสม

### 239.1 กรณีที่ 1: Maintainer ขอให้แก้ไข (Request Changes)

นี่คือกรณีที่พบบ่อยที่สุด และ**ไม่ใช่เรื่องแย่เลย** — มันหมายความว่า maintainer สนใจงานของคุณและต้องการให้มันสมบูรณ์พอที่จะ merge ได้

**สิ่งที่ควรทำ:**

- อ่าน comment ทุกจุดอย่างละเอียด ไม่รีบตอบโต้ทันที
- ถ้าไม่เข้าใจ comment ให้ถามกลับอย่างสุภาพ เช่น "Could you clarify what you mean by X? I want to make sure I fix this correctly."
- แก้ไขตามที่ขอ แล้ว push commit ใหม่เข้า branch เดิม (ตามที่อธิบายใน Step 235.7)
- comment กลับไปบอกว่าคุณแก้ไขจุดไหนแล้วบ้าง เช่น "Updated based on your feedback — moved the validation logic to the utils file as suggested."
- ถ้าคุณไม่เห็นด้วยกับข้อเสนอแนะบางจุด สามารถอธิบายเหตุผลของตัวเองได้อย่างสุภาพ แต่ให้เปิดใจรับฟังเหตุผลของ maintainer ด้วยเช่นกัน — ท้ายที่สุด maintainer เป็นผู้ตัดสินใจสุดท้ายว่าโค้ดแบบไหนจะเข้าไปอยู่ในโปรเจกต์ของพวกเขา

**สิ่งที่ไม่ควรทำ:**

- ปิด PR ทิ้งทันทีเพราะรู้สึกท้อ โดยไม่ลองแก้ไขตามที่ขอก่อน
- เถียงกลับด้วยอารมณ์ หรือมองว่า feedback เป็นการโจมตีส่วนตัว
- เพิกเฉยไม่ตอบกลับเป็นเวลานาน แล้วหายไปเฉย ๆ (ถ้าไม่มีเวลาทำต่อจริง ๆ ควร comment แจ้งอย่างสุภาพ)

### 239.2 กรณีที่ 2: PR ถูกปิดโดยไม่ Merge (Closed without merging)

บางครั้ง maintainer อาจตัดสินใจปิด PR โดยไม่รวมเข้าโปรเจกต์เลย ด้วยเหตุผลต่าง ๆ เช่น:

- แนวทางที่คุณเสนอไม่ตรงกับทิศทางของโปรเจกต์
- มีคนอื่นแก้ปัญหาเดียวกันไปแล้วในอีก PR หนึ่ง
- Feature นั้นถูกตัดสินใจแล้วว่าจะไม่รับเข้าโปรเจกต์ (out of scope)
- โปรเจกต์หยุดดูแลไปแล้ว (unmaintained)

**สิ่งที่ควรทำ:**

- ขอบคุณ maintainer สำหรับเวลาที่ใช้พิจารณา ไม่ว่าผลจะเป็นอย่างไร
- ถามอย่างสุภาพว่าทำไมถึงไม่ถูกรับ ถ้า comment ที่ให้มาไม่ชัดเจนพอ เพื่อเรียนรู้สำหรับครั้งต่อไป
- เก็บโค้ดของคุณไว้ในเครื่องหรือใน fork — งานที่ทำไปไม่ได้สูญเปล่า อาจนำไปปรับใช้กับโปรเจกต์อื่นหรือใช้ในโปรเจกต์ส่วนตัวได้
- มองเป็นบทเรียน แล้วลองหาโปรเจกต์หรือ issue อื่นที่เหมาะสมกว่าต่อไป

**สิ่งที่ไม่ควรทำ:**

- ต่อว่า maintainer ต่อสาธารณะ หรือแสดงความไม่พอใจในที่สาธารณะ
- เปิด PR ใหม่ทันทีโดยใช้เนื้อหาเดิมโดยไม่แก้ไขอะไรเลย หวังว่าผลจะเปลี่ยน
- ตัดสินไปเองว่าจะไม่ contribute Open Source อีกเลยเพราะประสบการณ์แย่ครั้งเดียว — นี่เป็นเรื่องปกติมากที่เกิดขึ้นกับทุกคน แม้แต่ maintainer ระดับสูงเองก็เคยถูกปฏิเสธ PR มาก่อน

### 239.3 การรับมือกับ CI ที่ Fail

ถ้า GitHub Actions หรือ CI/CD ของโปรเจกต์แสดงผล **fail** สีแดงบน PR ของคุณ:

1. คลิกที่ลิงก์ **Details** ข้าง check ที่ fail เพื่อดู log ว่าผิดพลาดตรงไหน
2. แก้ไขปัญหาในเครื่อง local แล้วรันทดสอบซ้ำก่อน push อีกครั้ง
3. Push แก้ไขเข้า branch เดิม — CI จะรันตรวจสอบใหม่ให้อัตโนมัติ
4. อย่า push โค้ดที่ไม่ผ่าน test ซ้ำ ๆ หลายรอบโดยไม่ตรวจสอบในเครื่องตัวเองก่อน

### 239.4 ตารางสรุปการรับมือ

| สถานการณ์ | ควรทำ | ไม่ควรทำ |
|---|---|---|
| ถูกขอให้แก้ไข | อ่าน feedback ละเอียด แก้ไข push ใหม่ | ปิด PR ทิ้งทันที หรือเถียงด้วยอารมณ์ |
| PR ถูกปิดไม่ merge | ขอบคุณ ถามเหตุผล เก็บเป็นบทเรียน | ต่อว่า maintainer ในที่สาธารณะ |
| CI Fail | ดู log แก้ไข test ในเครื่องก่อน push ใหม่ | Push ซ้ำ ๆ โดยไม่ตรวจสอบเอง |
| ไม่มีการตอบกลับนาน | รออย่างอดทน comment ถามอย่างสุภาพเมื่อผ่านไปนานพอ | ทวงถามถี่เกินไปทุกวัน |

---

## Step 240: แบบฝึกหัด — Fork โปรเจกต์ตัวอย่าง แก้ไขเล็กน้อย และส่ง PR ข้าม Fork ให้สำเร็จครบวงจร

ถึงเวลาลงมือทำจริงตั้งแต่ต้นจนจบ แบบฝึกหัดนี้จะพาคุณผ่านทุกขั้นตอนที่เรียนมาใน Part นี้ครบทุก Step

### 240.1 ทางเลือกที่ 1: ใช้ Repository ฝึกฝนที่ออกแบบมาสำหรับมือใหม่โดยเฉพาะ

GitHub มี repository ที่สร้างขึ้นมาเพื่อให้มือใหม่ฝึก fork และ PR โดยเฉพาะ ไม่ต้องกลัวว่าจะไปรบกวนโปรเจกต์จริง ตัวอย่างที่นิยมใช้กันทั่วโลก:

- `github.com/firstcontributions/first-contributions` — ออกแบบมาเพื่อให้ทุกคนฝึกเพิ่มชื่อตัวเองในไฟล์ list แล้วส่ง PR ได้จริง โดยไม่มีการปฏิเสธ
- `github.com/octocat/Spoon-Knife` — repo ตัวอย่างของ GitHub เองสำหรับฝึก fork

### 240.2 ทางเลือกที่ 2: จำลองสถานการณ์ด้วย 2 บัญชีของตัวเอง (ถ้าไม่อยากยุ่งกับ repo คนอื่น)

ถ้าต้องการควบคุมสถานการณ์ทั้งหมดเอง สามารถจำลองการ fork และ PR ข้าม fork ได้โดยไม่ต้องพึ่งโปรเจกต์คนอื่นเลย:

1. สร้าง repository ใหม่บนบัญชีของคุณเอง ชื่อ `git-course-fork-practice` (ตั้งเป็น public)
2. ใส่ไฟล์เริ่มต้น เช่น `README.md` และ `CONTRIBUTORS.md` ที่มีแค่หัวข้อเปล่า ๆ
3. สมมติว่านี่คือ repo "ต้นฉบับ" ที่คนอื่นเป็นเจ้าของ

จากนั้นทำตามขั้นตอนด้านล่างเสมือนว่าคุณเป็นผู้ contribute ภายนอก (ในทางปฏิบัติจริง คุณต้องมีบัญชี GitHub อีกบัญชีหนึ่งเพื่อทดสอบการ fork ข้ามบัญชีจริง ๆ แต่ถ้าต้องการฝึกแค่กลไก คุณสามารถใช้บัญชีเดียวฝึกคำสั่งทั้งหมดได้ ยกเว้นขั้นตอนกด Fork ที่ต้องมีบัญชีที่สองจริง ๆ)

### 240.3 ขั้นตอนแบบฝึกหัดแบบเต็ม (ใช้ `first-contributions` เป็นตัวอย่าง)

**ขั้นตอนที่ 1 — Fork**

1. เปิด `https://github.com/firstcontributions/first-contributions`
2. กดปุ่ม **Fork** → **Create fork**
3. รอจนได้ repo ใหม่ที่ `https://github.com/yourusername/first-contributions`

**ขั้นตอนที่ 2 — Clone**

```bash
git clone https://github.com/yourusername/first-contributions.git
cd first-contributions
```

**ขั้นตอนที่ 3 — ตั้งค่า Upstream**

```bash
git remote add upstream https://github.com/firstcontributions/first-contributions.git
git remote -v
```

**ขั้นตอนที่ 4 — Sync ให้ทันสมัย (แม้เพิ่ง fork มาก็ควรฝึกขั้นตอนนี้ให้เป็นนิสัย)**

```bash
git checkout main
git fetch upstream
git merge upstream/main
```

**ขั้นตอนที่ 5 — สร้าง Branch ใหม่**

```bash
git checkout -b add-my-name
```

**ขั้นตอนที่ 6 — แก้ไขไฟล์**

เปิดไฟล์ `Contributors.md` ด้วย editor แล้วเพิ่มชื่อของคุณต่อท้ายรายการที่มีอยู่ เช่น:

```markdown
- Your Name (@yourusername)
```

**ขั้นตอนที่ 7 — Commit**

```bash
git add Contributors.md
git commit -m "docs: add my name to contributors list"
```

**ขั้นตอนที่ 8 — Push ขึ้น Fork**

```bash
git push origin add-my-name
```

**ขั้นตอนที่ 9 — เปิด Pull Request ข้าม Fork**

1. ไปที่หน้า repo ต้นฉบับ `firstcontributions/first-contributions`
2. คลิก **Pull requests** → **New pull request** → **compare across forks**
3. ตั้งค่า:
   ```
   base repository: firstcontributions/first-contributions   base: main
   head repository: yourusername/first-contributions          compare: add-my-name
   ```
4. ตรวจสอบ diff ว่าแสดงเฉพาะบรรทัดที่คุณเพิ่มเข้าไปจริง
5. ใส่หัวข้อ PR เช่น "Add [Your Name] to contributors list"
6. กด **Create pull request**

**ขั้นตอนที่ 10 — ตรวจสอบผลลัพธ์**

รอสักครู่ ระบบอัตโนมัติของ repo นี้ (ซึ่งออกแบบมาเพื่อรับ PR ฝึกฝนโดยเฉพาะ) มักจะ merge ให้อัตโนมัติหรือภายในเวลาไม่นาน คุณจะได้เห็น PR แรกในชีวิตของคุณถูก merge เข้าโปรเจกต์ Open Source จริงบน GitHub

### 240.4 การล้างข้อมูลหลังฝึกเสร็จ (Cleanup)

หลัง PR ถูก merge หรือปิดแล้ว ควรลบ branch ที่ใช้งานเสร็จแล้วทั้งบน fork และในเครื่อง เพื่อความเป็นระเบียบ:

```bash
# ลบ branch ในเครื่อง
git checkout main
git branch -d add-my-name

# ลบ branch บน fork (origin) บน GitHub
git push origin --delete add-my-name
```

GitHub เองก็มักจะมีปุ่ม **Delete branch** ปรากฏขึ้นบนหน้า PR ทันทีหลังจากถูก merge ให้กดปุ่มนั้นได้เลยเพื่อความสะดวก

### 240.5 Checklist สรุปแบบฝึกหัด

ก่อนถือว่าแบบฝึกหัดนี้เสร็จสมบูรณ์ ตรวจสอบว่าคุณทำครบทุกข้อต่อไปนี้:

- [ ] Fork repository ตัวอย่างสำเร็จ เห็น repo ใหม่ในบัญชีตัวเองบน GitHub
- [ ] Clone fork มาที่เครื่อง local สำเร็จ
- [ ] ตั้งค่า `upstream` remote ชี้ไปยัง repo ต้นฉบับสำเร็จ และตรวจสอบด้วย `git remote -v`
- [ ] ฝึก sync fork ด้วย `git fetch upstream` + `git merge upstream/main`
- [ ] สร้าง branch ใหม่เฉพาะสำหรับงานนี้ (ไม่ทำงานบน `main` โดยตรง)
- [ ] แก้ไขไฟล์ commit และ push ขึ้น `origin` (fork ของตัวเอง) สำเร็จ
- [ ] เปิด Pull Request แบบข้าม fork (cross-repository) ได้สำเร็จ พร้อมตั้งค่า base/head ถูกต้อง
- [ ] เขียนคำอธิบาย PR ที่ชัดเจน สื่อสารสุภาพ
- [ ] เข้าใจว่าเมื่อ PR ถูกขอให้แก้ไข ต้อง push commit ใหม่เข้า branch เดิม ไม่ต้องเปิด PR ใหม่
- [ ] ลบ branch ที่ใช้งานเสร็จแล้วทั้งในเครื่องและบน fork

---

## สรุป Part 24

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Fork** คือการสร้างสำเนาเต็มของ repository ไว้ในบัญชี GitHub ของตัวเอง แตกต่างจาก **Clone** ซึ่งเป็นการคัดลอกมาที่เครื่อง local เท่านั้น — ทั้งสองขั้นตอนต้องทำร่วมกันเมื่อ contribute ให้ repo ที่คุณไม่มีสิทธิ์ write
2. ขั้นตอน Fork → Clone มาที่เครื่องต้องระวังการ clone จาก URL ของ **fork** ไม่ใช่จาก repo ต้นฉบับ เพื่อให้ `origin` ชี้ไปยัง repo ที่คุณมีสิทธิ์ push
3. การตั้งค่า **`upstream` remote** ด้วย `git remote add upstream <url>` เป็นธรรมเนียมมาตรฐานเพื่อติดตามความเปลี่ยนแปลงจาก repo ต้นฉบับ โดย `upstream` ใช้สำหรับ fetch/pull เท่านั้น ไม่ใช้ push
4. การ **Sync fork** ทำได้ด้วย `git fetch upstream` ตามด้วย `git merge upstream/main` (หรือ `git rebase upstream/main`) แล้ว push กลับไปที่ `origin` — ควรทำเป็นประจำก่อนเริ่มงานใหม่ทุกครั้ง
5. การส่ง **Pull Request ข้าม fork** ต้องเข้าใจความแตกต่างระหว่าง base repository/branch (ปลายทาง คือ repo ต้นฉบับ) และ head repository/branch (ต้นทาง คือ fork ของคุณ)
6. **CONTRIBUTING.md** คือคู่มือกฎกติกาเฉพาะของแต่ละโปรเจกต์ที่ต้องอ่านก่อนลงมือ contribute เสมอ — เราจะเจาะลึกเรื่องนี้อีกครั้งใน Part 87
7. มารยาทสำคัญของการ contribute Open Source ครั้งแรก ได้แก่ การทำ PR ขนาดเล็กโฟกัสเรื่องเดียว การสื่อสารสุภาพ และการเปิด issue พูดคุยก่อนสำหรับงานใหญ่
8. การหาโปรเจกต์ที่เหมาะกับมือใหม่ทำได้ผ่าน label `good first issue` และ `help wanted` บน GitHub รวมถึงเว็บไซต์ third-party ที่ช่วยรวบรวมไว้ให้
9. เมื่อ PR ถูกขอให้แก้ไขหรือถูกปิดโดยไม่ merge ควรรับมืออย่างมืออาชีพ ไม่ท้อแท้หรือแสดงพฤติกรรมไม่เหมาะสม เพราะเป็นเรื่องปกติที่เกิดขึ้นกับนักพัฒนาทุกระดับ
10. คุณได้ฝึกปฏิบัติจริงครบวงจรตั้งแต่ Fork จนถึงส่ง Pull Request ข้าม fork สำเร็จแล้วในแบบฝึกหัดของ Step 240

**ต่อไป:** [Part 25: GitHub Pages: การทำเว็บไซต์ฟรีจาก Repository](./part-025-github-pages.md)
