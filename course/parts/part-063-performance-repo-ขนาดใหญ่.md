# Part 63: การจัดการ Performance ของ Repository ขนาดใหญ่

> **Step ในหลักสูตรนี้:** Step 621–630
> **เฟส:** 6 — Git ขั้นสูงระดับ Internals, Hooks, LFS, Performance
> **เป้าหมายของ Part นี้:** เข้าใจว่าเมื่อ repository โตขึ้นจนมีขนาดหลายกิกะไบต์และมีประวัติหลายล้าน commit จะเกิดปัญหาอะไรบ้าง และเรียนรู้เทคนิคระดับมืออาชีพที่ Git เตรียมไว้ให้แก้ปัญหานั้น ตั้งแต่ shallow clone, partial clone, sparse checkout, ไปจนถึงระบบ maintenance อัตโนมัติ และเครื่องมือระดับ enterprise อย่าง Scalar

---

## สารบัญของ Part นี้

- Step 621: ปัญหาที่เกิดกับ Repository ขนาดใหญ่มาก ๆ
- Step 622: Shallow Clone — `git clone --depth <n>`
- Step 623: Partial Clone — `git clone --filter=blob:none`
- Step 624: Sparse Checkout — เอาเฉพาะบางโฟลเดอร์มาทำงานจริง
- Step 625: `git maintenance` — ระบบดูแล Repository อัตโนมัติเบื้องหลัง
- Step 626: การวัด Performance ด้วย `GIT_TRACE` และ `git count-objects`
- Step 627: Monorepo Scaling Techniques ภาพรวม
- Step 628: Git Protocol v2 ช่วยเรื่อง Performance อย่างไร
- Step 629: เครื่องมือเสริมระดับ Enterprise — VFS for Git และ Scalar
- Step 630: แบบฝึกหัด — ทดลองเปรียบเทียบ Shallow / Partial Clone / Sparse Checkout

---

## Step 621: ปัญหาที่เกิดกับ Repository ขนาดใหญ่มาก ๆ

ตลอดหลักสูตรนี้ เราฝึกกับ repository ขนาดเล็กที่มีไฟล์ไม่กี่สิบไฟล์และ commit ไม่กี่ร้อยตัว คำสั่งทุกตัวจึงทำงานเร็วจนแทบไม่รู้สึกถึงเวลาที่ใช้เลย แต่ในโลกจริง มี repository จำนวนไม่น้อยที่โตจนกลายเป็น **monorepo ระดับองค์กร** เช่น

- Repository ของ **Windows** ที่ Microsoft ดูแล มีไฟล์กว่า 3.5 ล้านไฟล์ และมีขนาดกว่า 300 GB
- Repository ของ **Google** (แม้จะไม่ได้ใช้ Git ตรง ๆ แต่ใช้แนวคิดคล้ายกันผ่าน Piper) มีโค้ดหลายพันล้านบรรทัด
- Repository โอเพนซอร์สขนาดใหญ่บางตัวมีประวัติ commit เกินหนึ่งล้าน commit สะสมมาหลายสิบปี

เมื่อ repository โตถึงระดับนี้ Git แบบที่เราใช้กันปกติ (full clone, full checkout) จะเริ่มมีปัญหาชัดเจนหลายอย่าง:

### 1. `git clone` ช้ามาก และกินพื้นที่มหาศาล

ปกติ `git clone` จะดาวน์โหลด **ทุก object** ในประวัติทั้งหมดของ repository ลงมาเก็บไว้ในเครื่อง ไม่ว่าจะเป็น commit, tree, blob ตั้งแต่ commit แรกสุดจนถึงปัจจุบัน ถ้า repository มีขนาด 200 GB การ clone ครั้งแรกอาจใช้เวลาหลายชั่วโมง และกินพื้นที่ดิสก์เท่ากับขนาดของ pack file ทั้งหมด บวกกับพื้นที่สำหรับ working directory ที่ checkout ออกมาอีกชุดหนึ่ง

### 2. คำสั่งพื้นฐานอย่าง `git status` และ `git checkout` ช้าลงอย่างเห็นได้ชัด

`git status` ต้องเดินสำรวจ (walk) ไฟล์ทุกไฟล์ใน working directory เพื่อเทียบกับ index ว่ามีอะไรเปลี่ยนไปบ้าง ถ้ามีไฟล์เป็นล้านไฟล์ การสแกน filesystem ทั้งหมดในทุกครั้งที่รันคำสั่งจะกินเวลานานเกินกว่าจะใช้งานได้จริงในชีวิตประจำวัน (บางองค์กรรายงานว่า `git status` ใช้เวลาเป็นนาทีในบาง monorepo ที่ไม่ได้ปรับแต่งอะไรเลย)

### 3. ประวัตินับล้าน commit ทำให้คำสั่งที่ต้องไล่ history ช้าลง

คำสั่งที่ต้องเดินไล่กราฟของ commit เช่น `git log`, `git blame`, `git merge-base`, หรือแม้แต่การคำนวณว่า branch ไหนอยู่ข้างหน้าอีก branch กี่ commit (`ahead/behind`) ต้องเดินสำรวจกราฟ commit ซึ่งถ้ามีจำนวนโหนดมหาศาลและไม่มีโครงสร้างช่วยเร่งความเร็ว การคำนวณจะช้าลงเป็นเชิงเส้นหรือแย่กว่านั้นตามจำนวน commit

### 4. การส่งข้อมูลระหว่าง client กับ server (negotiation) หนักขึ้น

ทุกครั้งที่ `git fetch` หรือ `git push` ฝั่ง client และ server ต้อง **negotiate** กันว่าฝ่ายไหนมี object อะไรอยู่แล้วบ้าง เพื่อจะได้ส่งเฉพาะ object ที่ขาดไป ถ้า repository มี branch และ tag (เรียกรวมว่า refs) เป็นจำนวนมาก (บาง monorepo มี ref นับหมื่นนับแสนอัน) แค่ขั้นตอนแลกเปลี่ยนรายชื่อ ref ก่อนเริ่ม negotiate จริงก็ใช้เวลาและ bandwidth มากแล้ว

### 5. Object database บวมขึ้นเรื่อย ๆ จนต้องดูแลเป็นพิเศษ

ทุก commit, ทุกไฟล์ที่เคย add เข้ามา (แม้จะถูกลบไปแล้วในภายหลัง) ยังคงถูกเก็บไว้ใน object database ตลอดไป (เว้นแต่จะทำ history rewriting ซึ่งเราเรียนไปแล้วใน Part ก่อนหน้าเรื่อง `git filter-repo`) ถ้าไม่มีการบีบอัด (repack) และทำความสะอาด (garbage collect) เป็นประจำ จำนวน loose object จะเพิ่มขึ้นเรื่อย ๆ จนกระทบ performance ของทุกคำสั่ง

### สรุปปัญหาเป็นตาราง

| ปัญหา | สาเหตุหลัก | ผลกระทบที่ผู้ใช้รู้สึก |
|---|---|---|
| Clone ช้า/กินพื้นที่มาก | ต้องดาวน์โหลด object ทุกตัวในทุกประวัติ | รอนานตอนเริ่มงานครั้งแรก, ดิสก์เต็ม |
| `status`/`checkout` ช้า | ต้องสแกนไฟล์เป็นล้านไฟล์ใน working directory | รู้สึกหน่วงทุกครั้งที่พิมพ์คำสั่งพื้นฐาน |
| `log`/`blame` ช้า | ต้องเดินไล่กราฟ commit ที่มีโหนดจำนวนมาก | รอหลายวินาทีถึงหลายนาทีต่อคำสั่ง |
| Fetch/Push หนัก | Negotiate ref และ object จำนวนมากทุกครั้ง | เครือข่ายช้า, CI/CD ใช้เวลานานขึ้น |
| Object database บวม | ไม่มีการ repack/gc สม่ำเสมอ | ทุกคำสั่งช้าลงเรื่อย ๆ ตามเวลา |

ข่าวดีคือ Git เองก็เติบโตมาพร้อมกับปัญหานี้ — ทีมพัฒนา Git หลัก, GitHub, GitLab และ Microsoft (ที่มี monorepo ของ Windows) ต่างช่วยกันพัฒนาฟีเจอร์เฉพาะทางขึ้นมาแก้ปัญหาเหล่านี้ทีละจุด ซึ่งเราจะไล่เรียนรู้ทีละเทคนิคใน Step ถัดไปนี้

> **หมายเหตุสำคัญ:** เทคนิคทั้งหมดใน Part นี้เหมาะกับสถานการณ์ "repository ใหญ่จริง ๆ" เท่านั้น ถ้า repository ของคุณมีขนาดปกติ (ไม่กี่ร้อยเมกะไบต์ ไม่กี่หมื่น commit) การใช้ full clone แบบปกติยังคงเป็นทางเลือกที่ดีที่สุดและง่ายที่สุด — อย่าเพิ่งไปใช้เทคนิคขั้นสูงเหล่านี้กับ repository เล็ก ๆ เพราะจะเพิ่มความซับซ้อนโดยไม่จำเป็น

---

## Step 622: Shallow Clone — `git clone --depth <n>`

เทคนิคแรกและง่ายที่สุดในการลดเวลาและพื้นที่ตอน clone คือ **Shallow Clone** (การ clone แบบตื้น)

### แนวคิด

ปกติ `git clone` จะดึงประวัติ **ทั้งหมด** ตั้งแต่ commit แรกสุดของ repository มาเก็บไว้ในเครื่อง แต่ในหลายกรณี เราไม่จำเป็นต้องมีประวัติทั้งหมดเลย เช่น:

- CI/CD pipeline ที่แค่ต้องการ build โค้ดจาก commit ล่าสุด ไม่สนใจประวัติเก่า
- นักพัฒนาที่แค่ต้องการดูโค้ดปัจจุบันเพื่อแก้บั๊กเร่งด่วน ไม่ได้สนใจ `git log` ย้อนหลังหลายปี

`git clone --depth <n>` จะบอก Git ว่า **ให้ดึงมาแค่ N commit ล่าสุดเท่านั้น** นับจาก branch ที่ checkout มา

```bash
git clone --depth 1 https://github.com/example/big-repo.git
```

คำสั่งนี้จะดึงมาแค่ **1 commit ล่าสุด** (commit ปัจจุบันของ default branch) พร้อมไฟล์ทั้งหมด ณ commit นั้น แต่ **ไม่มีประวัติก่อนหน้าเลยแม้แต่ commit เดียว**

### จะเกิดอะไรขึ้นกับ `.git` เมื่อทำ Shallow Clone

Git จะสร้างไฟล์พิเศษชื่อ `.git/shallow` ขึ้นมา เก็บรายชื่อ commit hash ที่เป็น "ขอบเขต" ของประวัติที่มี (เรียกว่า **grafted commit** หรือจุดตัด) ลองดูตัวอย่าง:

```bash
git clone --depth 3 https://github.com/example/repo.git
cd repo
cat .git/shallow
```

```
a1b2c3d4e5f6...   ← commit hash ที่เป็นขอบเขตล่างสุดที่มีอยู่
```

เมื่อรัน `git log` ใน shallow clone คุณจะเห็นแค่ 3 commit เท่านั้น แล้ว Git จะไม่สามารถเดินย้อนไปไกลกว่านั้นได้ เพราะไม่มี object ของ commit ก่อนหน้าอยู่ในเครื่องเลย

```bash
git log --oneline
```

```
a1b2c3d (HEAD -> main) fix: bug ล่าสุด
9f8e7d6 feat: เพิ่มฟีเจอร์ X
1234abc chore: อัปเดต dependency
```

ถ้าลอง `git log` ต่อจากตรงนี้จะขึ้น error หรือหยุดแค่นี้ทันที เพราะ commit ก่อนหน้าไม่มีอยู่จริงในเครื่อง

### ข้อจำกัดที่ต้องรู้ก่อนใช้

Shallow clone ไม่ใช่ยาวิเศษที่ไม่มีข้อเสีย มันมีข้อจำกัดสำคัญที่ต้องเข้าใจก่อนนำไปใช้จริง:

1. **`git blame` และ `git log <file>` ทำงานได้ไม่สมบูรณ์** — เพราะไม่มีประวัติเก่าให้ไล่ดู
2. **การ push กลับไปที่ remote อาจมีข้อจำกัด** — บาง server (เช่น GitHub ในบางเงื่อนไข) ไม่อนุญาตให้ push จาก shallow repository เข้าไปสร้างเป็น branch ใหม่ในบางกรณี แม้ปัจจุบัน Git และ GitHub ส่วนใหญ่จะรองรับการ push จาก shallow clone ได้แล้วก็ตาม
3. **`git merge-base` และการคำนวณ ahead/behind อาจผิดพลาดหรือทำไม่ได้** — เพราะ Git ไม่รู้จุดร่วมที่แท้จริงในประวัติที่ไม่มีอยู่
4. **ไม่สามารถ checkout ไปยัง commit ที่เก่ากว่าขอบเขต shallow ได้** จนกว่าจะดึงประวัติเพิ่ม

### การขยายประวัติภายหลัง (Deepen หรือ Unshallow)

ถ้า clone แบบตื้นไปแล้วภายหลังต้องการประวัติเพิ่ม สามารถทำได้ 2 แบบ:

**1. ดึงเพิ่มทีละจำนวนที่กำหนด (deepen):**

```bash
git fetch --deepen=10
```

คำสั่งนี้จะดึงประวัติเพิ่มอีก 10 commit ถัดจากขอบเขตปัจจุบัน

**2. ดึงประวัติทั้งหมดกลับมา (unshallow):**

```bash
git fetch --unshallow
```

คำสั่งนี้จะเปลี่ยน shallow clone ให้กลายเป็น full clone ที่มีประวัติครบสมบูรณ์เหมือน clone ปกติทุกประการ (ไฟล์ `.git/shallow` จะถูกลบไป)

### ตัวเลือกที่ใช้ร่วมกันบ่อย

| คำสั่ง/ตัวเลือก | ความหมาย |
|---|---|
| `git clone --depth 1` | ดึงมาแค่ commit ล่าสุด 1 อัน |
| `git clone --depth 1 --branch develop` | ดึง 1 commit ล่าสุดของ branch `develop` เท่านั้น |
| `git clone --shallow-since=2024-01-01` | ดึงเฉพาะ commit ตั้งแต่วันที่กำหนดเป็นต้นมา |
| `git clone --shallow-exclude=<ref>` | ดึงประวัติจนถึงก่อนถึง ref ที่ระบุ (ไม่รวม ref นั้น) |
| `git fetch --deepen=<n>` | ขยายประวัติ shallow clone เพิ่มอีก n commit |
| `git fetch --unshallow` | แปลง shallow clone ให้เป็น full history |

### กรณีใช้งานที่เหมาะสม

- **CI/CD build**: แทบทุก CI system (GitHub Actions, GitLab CI) ใช้ `--depth 1` เป็นค่าเริ่มต้นอยู่แล้วเมื่อ checkout โค้ด เพราะ pipeline ส่วนใหญ่ต้องการแค่โค้ดปัจจุบันไปทดสอบ/build เท่านั้น
- **ตรวจสอบโค้ดแบบเร่งด่วน**: เมื่อต้องการดูโค้ดปัจจุบันของ repository ใหญ่โดยไม่สนใจประวัติ
- **Docker image build**: การ build image มักต้องการแค่ไฟล์ปัจจุบัน ไม่จำเป็นต้องมีประวัติ

### กรณีที่ไม่ควรใช้

- งานพัฒนาปกติที่ต้องใช้ `git blame`, `git bisect`, หรือย้อนดูประวัติบ่อย ๆ
- Repository ที่ทีมยังทำงาน rebase/merge ซับซ้อนบน branch เดิมอยู่เรื่อย ๆ

Shallow clone แก้ปัญหาเรื่อง **จำนวน commit ในประวัติ** ได้ดี แต่ยังไม่ได้แก้ปัญหาเรื่อง **ขนาดไฟล์ปัจจุบันที่มีจำนวนมาก** (เช่น repository ที่มีไฟล์เป็นล้านไฟล์ในโฟลเดอร์เดียว) นั่นคือสิ่งที่ Step ถัดไปจะเข้ามาช่วยแก้

---

## Step 623: Partial Clone — `git clone --filter=blob:none`

Shallow clone ตัดปัญหาเรื่อง **ประวัติ (commit history)** ออกไป แต่ถ้าปัญหาของคุณคือ **repository ปัจจุบันมีขนาดไฟล์ (blob) ใหญ่มาก** เช่น มีไฟล์ binary, asset, หรือไฟล์ข้อมูลจำนวนมหาศาลที่คุณไม่ได้ใช้งานทั้งหมดในคราวเดียว **Partial Clone** คือคำตอบ

### แนวคิดของ Partial Clone

Partial Clone (เปิดตัวอย่างเป็นทางการใน Git 2.19 และพัฒนาต่อเนื่องมาเรื่อย ๆ) อนุญาตให้ client **clone โดยไม่ต้องดึง object บางประเภทมาด้วย** แล้วค่อยดึงมาแบบ **on-demand** (ตามต้องการ) ในภายหลังเมื่อจำเป็นต้องใช้จริง เช่น ตอน `checkout` หรือ `diff` ไฟล์นั้น

Git แยก object ออกเป็น 3 ประเภทหลักที่เกี่ยวข้องกับการ filter ได้คือ:

- **Commit** — เก็บ metadata ของแต่ละ commit
- **Tree** — เก็บโครงสร้างโฟลเดอร์ ชี้ไปยัง blob และ tree ย่อย
- **Blob** — เนื้อหาไฟล์จริง (มักเป็นส่วนที่กินพื้นที่มากที่สุดใน repository)

### `--filter=blob:none` (Blobless Clone)

```bash
git clone --filter=blob:none https://github.com/example/big-repo.git
```

คำสั่งนี้จะดึง **commit และ tree ทั้งหมด** มาครบ (จึงยังมีประวัติสมบูรณ์ ใช้ `git log` ได้ปกติ) แต่จะ **ไม่ดึงเนื้อหาไฟล์ (blob) มาก่อน** — blob จะถูกดึงมาแบบ **lazy** (ขี้เกียจ, ดึงเมื่อจำเป็นเท่านั้น) เมื่อคุณ:

- `git checkout` ไปยัง commit ที่ต้องใช้ไฟล์นั้น
- `git diff` ที่ต้องเปรียบเทียบเนื้อหาไฟล์
- `git blame` ที่ต้องดูเนื้อหาไฟล์ย้อนหลัง

Object ที่ยังไม่ถูกดึงมาจะถูกเรียกว่า **missing object** และ Git จะติดต่อไปยัง remote (ที่เรียกว่า **promisor remote**) เพื่อขอ object นั้นมาโดยอัตโนมัติแบบโปร่งใส ผู้ใช้แทบไม่รู้สึกว่ามีอะไรพิเศษเกิดขึ้น นอกจากจะสังเกตว่าคำสั่งบางตัวใช้เวลานานขึ้นเล็กน้อยเพราะต้องดาวน์โหลดเพิ่มระหว่างทาง

### `--filter=tree:0` (Treeless Clone)

```bash
git clone --filter=tree:0 https://github.com/example/big-repo.git
```

เข้มงวดกว่า blobless clone อีกขั้น คือดึงมาแค่ **commit object** เท่านั้น ไม่ดึงทั้ง tree และ blob มาก่อนเลย ทำให้ clone เร็วขึ้นไปอีก แต่คำสั่งที่ต้องเดินดูโครงสร้างไฟล์ (เช่น `git checkout`, `git log -- <path>`) จะต้องดึง tree object เพิ่มระหว่างทางบ่อยกว่า blobless clone มาก ในทางปฏิบัติ treeless clone จึงถูกใช้น้อยกว่า blobless clone เพราะการดึง tree ทีละชั้นบ่อย ๆ อาจช้ากว่าที่คาดไว้

### `--filter=blob:limit=<size>`

ตัวเลือกแบบผสมที่ยืดหยุ่นกว่า `blob:none` คือกำหนดขนาดไฟล์ที่จะดึงมาทันที ไฟล์ที่เล็กกว่าค่าที่กำหนดจะถูกดึงมาตั้งแต่ตอน clone (เพราะไฟล์เล็กส่วนใหญ่คือซอร์สโค้ดที่ต้องใช้งานบ่อย) ส่วนไฟล์ที่ใหญ่กว่าจะถูกดึงแบบ lazy

```bash
git clone --filter=blob:limit=1m https://github.com/example/big-repo.git
```

ตัวอย่างนี้จะดึง blob ที่มีขนาดเล็กกว่า 1 MB มาทันที ส่วน blob ที่ใหญ่กว่า 1 MB (มักเป็นไฟล์ asset, binary, รูปภาพขนาดใหญ่) จะถูกดึงมาทีหลังเมื่อจำเป็น

### ความต้องการฝั่ง Server

Partial clone ไม่ใช่ฟีเจอร์ที่ client ทำได้ฝ่ายเดียว — **server ต้องรองรับ** การส่ง object แบบ filter ได้ด้วย โดยเปิดผ่านการตั้งค่า:

```bash
git config uploadpack.allowFilter true
git config uploadpack.allowAnySHA1InWant true
```

ผู้ให้บริการรายใหญ่อย่าง **GitHub และ GitLab รองรับ partial clone แล้ว** จึงสามารถใช้ `--filter=blob:none` ได้ตรง ๆ กับ repository บนแพลตฟอร์มเหล่านี้ทันทีโดยไม่ต้องตั้งค่าเพิ่มฝั่ง client

### ตรวจสอบว่า Repository เป็น Partial Clone อยู่หรือไม่

```bash
git config --get remote.origin.partialclonefilter
git rev-parse --is-shallow-repository   # false สำหรับ partial clone (คนละกลไกกับ shallow)
cat .git/config | grep -A2 '\[remote "origin"\]'
```

Partial clone จะบันทึกค่า filter ไว้ใน `.git/config` ในหัวข้อของ remote นั้น เช่น:

```ini
[remote "origin"]
    url = https://github.com/example/big-repo.git
    fetch = +refs/heads/*:refs/remotes/origin/*
    promisor = true
    partialclonefilter = blob:none
```

### เปรียบเทียบ Shallow Clone กับ Partial Clone

| คุณสมบัติ | Shallow Clone | Partial Clone |
|---|---|---|
| ตัดอะไรออก | ตัดจำนวน **commit** ในประวัติ | ตัด **เนื้อหา object** (blob/tree) ตามเงื่อนไข |
| ประวัติ (`git log`) | ไม่ครบ มองย้อนได้จำกัด | ครบสมบูรณ์ (ถ้าใช้ blob:none) |
| ต้องต่อเน็ตภายหลังไหม | ไม่ต้อง (ยกเว้นจะ deepen) | ต้อง (เพื่อดึง object ที่ขาดแบบ lazy) |
| เหมาะกับ | CI/CD ที่ต้องการโค้ดปัจจุบันอย่างเดียว | งานพัฒนาจริงบน repo ที่มีไฟล์ใหญ่จำนวนมาก |
| Server ต้องรองรับพิเศษไหม | ไม่ต้อง (รองรับมานานแล้ว) | ต้อง (`uploadpack.allowFilter`) |

### ข้อควรระวัง

- ถ้าตัดการเชื่อมต่อเครือข่ายกับ remote ถาวร (เช่น ยกเลิก URL ของ origin) คำสั่งที่ต้องการ object ที่ยังไม่ได้ดึงมาจะ **ล้มเหลว** เพราะไม่มีทางไปหา object นั้นมาได้อีก
- Partial clone เหมาะกับการทำงานที่ยังเชื่อมต่อกับ remote อยู่เสมอ ไม่เหมาะกับสถานการณ์ offline ระยะยาว
- ควรใช้ร่วมกับ **Sparse Checkout** (Step ถัดไป) เพื่อประสิทธิภาพสูงสุด เพราะ partial clone ช่วยเรื่องการดาวน์โหลด แต่ยังไม่ได้ช่วยเรื่องจำนวนไฟล์ที่ต้อง checkout ลง working directory

---

## Step 624: Sparse Checkout — เอาเฉพาะบางโฟลเดอร์มาทำงานจริงในเครื่อง

Partial clone ช่วยลดข้อมูลที่ **ดาวน์โหลด** แต่ปัญหาอีกด้านของ monorepo ขนาดใหญ่คือ **working directory เอง** ก็มีไฟล์มากเกินความจำเป็นสำหรับงานที่คุณกำลังทำอยู่ ถ้า repository มีไฟล์ 2 ล้านไฟล์ แต่คุณทำงานอยู่แค่ในโฟลเดอร์ `services/payment/` เพียงโฟลเดอร์เดียว การมีไฟล์อีก 1.9 ล้านไฟล์นอนอยู่ใน working directory ก็ทำให้ `git status`, การเปิดไฟล์ใน IDE, หรือแม้แต่การ index ของระบบปฏิบัติการช้าลงโดยไม่จำเป็น

**Sparse Checkout** คือฟีเจอร์ที่ทำให้คุณสามารถ **เลือก checkout เฉพาะบางโฟลเดอร์/ไฟล์** ลงมาใน working directory จริง ส่วนที่เหลือจะไม่ถูก checkout ลงดิสก์เลย แม้ว่า object ของมันอาจจะยังอยู่ใน `.git` (หรือไม่อยู่เลยถ้าใช้ร่วมกับ partial clone)

### โหมดของ Sparse Checkout: Cone Mode vs Non-Cone Mode

Git มี sparse checkout อยู่ 2 โหมด:

1. **Cone Mode (แนะนำ, เร็วกว่ามาก)** — จำกัดเฉพาะการเลือกทั้ง "โฟลเดอร์" เท่านั้น (ไม่รองรับ pattern ซับซ้อนแบบ `.gitignore`) แต่แลกมาด้วยความเร็วที่สูงกว่าอย่างมาก เพราะ Git ใช้อัลกอริทึมพิเศษที่ตรวจสอบเฉพาะ path prefix แทนการเทียบ pattern ทีละไฟล์
2. **Non-Cone Mode (ของเดิม, ยืดหยุ่นแต่ช้ากว่า)** — ใช้ pattern แบบ `.gitignore` เต็มรูปแบบ เลือกไฟล์ทีละไฟล์หรือใช้ wildcard ได้อย่างอิสระ แต่การประมวลผล pattern จำนวนมากบน repository ใหญ่จะช้ากว่า cone mode มาก

**คำแนะนำมาตรฐาน: ใช้ Cone Mode เสมอสำหรับ repository ขนาดใหญ่** เว้นแต่จะมีความจำเป็นต้องเลือกไฟล์แบบ pattern ที่ซับซ้อนจริง ๆ

### ขั้นตอนการใช้งาน Cone Mode

**1. เริ่มต้น sparse checkout ในโหมด cone:**

```bash
git sparse-checkout init --cone
```

คำสั่งนี้จะทำให้ working directory เหลือแค่ไฟล์ที่อยู่ที่ root ของ repository เท่านั้น (โฟลเดอร์ย่อยทั้งหมดจะหายไปจาก working directory ชั่วคราว แต่ object ยังอยู่ใน `.git`)

**2. เลือกโฟลเดอร์ที่ต้องการทำงานจริง:**

```bash
git sparse-checkout set services/payment services/shared
```

หลังจากรันคำสั่งนี้ working directory จะมีเฉพาะไฟล์ที่ root บวกกับเนื้อหาของ `services/payment/` และ `services/shared/` เท่านั้น โฟลเดอร์อื่น ๆ ทั้งหมดจะไม่ปรากฏใน working directory เลย

**3. เพิ่มโฟลเดอร์เข้าไปอีกในภายหลัง (ไม่ลบของเดิม):**

```bash
git sparse-checkout add services/notification
```

**4. ดูรายการ pattern ที่ตั้งไว้ปัจจุบัน:**

```bash
git sparse-checkout list
```

```
services/payment
services/shared
services/notification
```

**5. ปิดการใช้งาน sparse checkout (กลับมาเห็นไฟล์ทั้งหมด):**

```bash
git sparse-checkout disable
```

### ตัวอย่างการใช้งานร่วมกับ Partial Clone (แนวทางที่แนะนำสำหรับ Monorepo)

ในทางปฏิบัติ องค์กรที่ดูแล monorepo ขนาดใหญ่แทบทุกที่จะใช้ **partial clone + sparse checkout ร่วมกัน** เพื่อให้ทั้งการดาวน์โหลดและ working directory เล็กที่สุดเท่าที่จำเป็น:

```bash
# Clone แบบไม่ดึง blob มาก่อน และไม่ checkout ไฟล์ทั้งหมดทันที
git clone --filter=blob:none --sparse https://github.com/example/monorepo.git
cd monorepo

# กำหนดว่าจะทำงานจริงในโฟลเดอร์ไหน
git sparse-checkout set services/payment
```

ตัวเลือก `--sparse` ตอน clone จะบอกให้ Git ตั้งค่า sparse checkout ให้อัตโนมัติแบบเลือกเฉพาะไฟล์ที่ root ตั้งแต่แรก แทนที่จะ checkout ไฟล์ทั้งหมดมาก่อนแล้วค่อยไปลบทีหลัง (ซึ่งเสียเวลาโดยใช่เหตุ)

ผลลัพธ์คือ:

- Object ที่ดาวน์โหลดมามีแค่ blob ของไฟล์ใน `services/payment` เท่านั้น (เพราะ blob:none + lazy fetch เมื่อ checkout)
- Working directory มีแค่ไฟล์ใน `services/payment` เท่านั้น (เพราะ sparse checkout)
- `git status`, IDE indexing, และ filesystem watcher ทำงานเร็วขึ้นมหาศาลเพราะมีไฟล์น้อยลง

### ไฟล์ที่เกี่ยวข้องเบื้องหลัง

Sparse checkout เก็บ pattern ที่เลือกไว้ในไฟล์:

```
.git/info/sparse-checkout
```

ซึ่งมีรูปแบบคล้าย `.gitignore` (ในโหมด cone จะมีรูปแบบพิเศษที่ Git จัดการให้อัตโนมัติผ่านคำสั่ง ไม่แนะนำให้แก้ไฟล์นี้ด้วยมือโดยตรงถ้าใช้ cone mode)

### ข้อควรระวังเรื่อง Sparse Checkout

1. **ไฟล์ที่ไม่ได้เลือกจะหายไปจาก working directory แต่ยังอยู่ใน index/`.git`** — ถ้าเปิด sparse checkout ไม่ถูกต้อง (เช่น ตั้ง path ผิด) แล้วจู่ ๆ ไฟล์ที่เคยเห็นหายไป อย่าตกใจว่าไฟล์ถูกลบจริง ให้ตรวจสอบด้วย `git sparse-checkout list` ก่อน
2. **การ commit ยังคง commit ได้เฉพาะไฟล์ที่อยู่ใน working directory** — คุณจะไม่สามารถแก้ไฟล์ที่ไม่ได้ sparse checkout ไว้ได้ เพราะมันไม่มีอยู่ในเครื่องเลย
3. **ทำงานร่วมกับ `.gitignore` คนละกลไก** — `.gitignore` ใช้ควบคุมไฟล์ที่ยังไม่ถูก track ส่วน sparse checkout ควบคุมไฟล์ที่ track อยู่แล้วว่าจะ checkout ออกมาจริงหรือไม่ อย่าสับสนสองกลไกนี้

---

## Step 625: `git maintenance` — ระบบดูแล Repository อัตโนมัติเบื้องหลัง

แม้จะลด object ที่ต้อง clone และไฟล์ที่ต้อง checkout ได้แล้ว แต่ repository ที่ใช้งานต่อเนื่องเป็นเวลานานยังต้องการ **การดูแลรักษาสม่ำเสมอ** เพื่อไม่ให้ประสิทธิภาพเสื่อมลงตามเวลา ปกติงานเหล่านี้ต้องรันด้วยมือ เช่น `git gc` แต่การรันด้วยมือมีปัญหาคือคนมักลืมรัน หรือรันตอนที่ไม่เหมาะสม (เช่น รัน `git gc` ตอนกำลังใช้งานหนัก ทำให้เครื่องหน่วง)

Git จึงมีระบบ **`git maintenance`** (เปิดตัวใน Git 2.31) ที่ทำหน้าที่จัดตารางงานดูแล repository ให้ทำงานอัตโนมัติเบื้องหลัง โดยไม่รบกวนการใช้งานปกติ

### งานดูแลรักษา (Maintenance Tasks) ที่มีให้เลือก

| Task | หน้าที่ |
|---|---|
| `gc` | รัน garbage collection แบบเต็มรูปแบบ ลบ object ที่ไม่ถูกอ้างอิงแล้วและไม่จำเป็น |
| `commit-graph` | สร้าง/อัปเดตไฟล์ commit-graph ที่ช่วยให้คำสั่งที่ต้องไล่กราฟ commit (เช่น `git log --graph`, `git merge-base`) เร็วขึ้นมาก |
| `prefetch` | ดึงข้อมูลใหม่จาก remote มาเก็บไว้ล่วงหน้าแบบเงียบ ๆ (ไม่ merge เข้ากับ branch ปัจจุบัน) เพื่อให้ `git fetch` ครั้งถัดไปที่ผู้ใช้สั่งเองรู้สึกเร็วขึ้น |
| `loose-objects` | รวบรวม loose object (object เดี่ยว ๆ ที่ยังไม่ถูกอัดเข้า pack file) ให้กลายเป็น pack file เพื่อลดจำนวนไฟล์และเพิ่มความเร็วในการอ่าน |
| `incremental-repack` | บีบอัด pack file หลาย ๆ ไฟล์ให้รวมกันแบบค่อยเป็นค่อยไป โดยไม่ต้อง repack ทั้งหมดในครั้งเดียว (ต่างจาก `git gc` ที่ repack แบบเต็ม) |
| `pack-refs` | รวบรวม ref (branch/tag) ที่กระจัดกระจายอยู่เป็นไฟล์แยก ให้มารวมอยู่ในไฟล์ `packed-refs` ไฟล์เดียว ลด I/O เวลาต้องอ่าน ref จำนวนมาก |

### การเริ่มต้นใช้งาน

**1. เปิดใช้งาน maintenance สำหรับ repository ปัจจุบัน พร้อมตั้งตารางเวลาอัตโนมัติ:**

```bash
git maintenance start
```

คำสั่งนี้จะทำ 2 อย่างพร้อมกัน:

1. เปิดใช้งาน task พื้นฐาน (`gc`, `commit-graph`, `loose-objects`, `incremental-repack`, `pack-refs`) สำหรับ repository นี้
2. ลงทะเบียนตารางเวลา (schedule) กับระบบปฏิบัติการ เพื่อให้งานเหล่านี้ทำงานอัตโนมัติเบื้องหลังแม้ไม่ได้เปิด terminal อยู่ก็ตาม โดยเบื้องหลังจะใช้:
   - **`cron`** บน Linux
   - **`launchd`** บน macOS
   - **Windows Scheduled Task** บน Windows

**2. รัน task ใดก็ได้ทันทีด้วยตัวเอง (ไม่ต้องรอตามตาราง):**

```bash
git maintenance run --task=gc
git maintenance run --task=commit-graph
git maintenance run             # รันทุก task ที่เปิดใช้งานอยู่
```

**3. หยุดการทำงานอัตโนมัติ:**

```bash
git maintenance stop
```

คำสั่งนี้จะยกเลิกตารางเวลาที่ลงทะเบียนไว้กับระบบปฏิบัติการ แต่การตั้งค่าใน `.git/config` ยังคงอยู่

### การตั้งค่าความถี่ของแต่ละ Task

แต่ละ task สามารถกำหนดความถี่ (schedule) แยกกันได้ผ่าน git config โดยค่าที่ใช้ได้คือ `hourly`, `daily`, `weekly`:

```bash
git config maintenance.gc.schedule daily
git config maintenance.commit-graph.schedule hourly
git config maintenance.prefetch.schedule hourly
git config maintenance.loose-objects.schedule daily
git config maintenance.incremental-repack.schedule daily
```

ค่าเริ่มต้นที่ Git ใช้เมื่อรัน `git maintenance start` โดยทั่วไปคือ:

| Task | ความถี่เริ่มต้น |
|---|---|
| `gc` | รายวัน (daily) |
| `commit-graph` | รายชั่วโมง (hourly) |
| `prefetch` | รายชั่วโมง (hourly) |
| `loose-objects` | รายวัน (daily) |
| `incremental-repack` | รายวัน (daily) |
| `pack-refs` | รายสัปดาห์ (weekly) |

### ตรวจสอบสถานะการตั้งค่าปัจจุบัน

```bash
git config --get-regexp maintenance
```

```
maintenance.auto false
maintenance.gc.enabled true
maintenance.gc.schedule daily
maintenance.commit-graph.enabled true
maintenance.commit-graph.schedule hourly
...
```

### เปิดใช้งานแบบ Global สำหรับทุก Repository ในเครื่อง

ถ้าต้องการให้ repository ทุกตัวที่คุณ clone มาในอนาคตเปิดใช้ maintenance อัตโนมัติโดยไม่ต้องตั้งค่าทีละอัน:

```bash
git config --global maintenance.strategy incremental
```

ค่า `incremental` เป็นชุดตั้งค่าสำเร็จรูปที่แนะนำโดยทีม Git เอง ซึ่งจะเปิด task ที่เหมาะสมและตั้งความถี่ที่สมดุลระหว่างประสิทธิภาพกับภาระของเครื่องให้อัตโนมัติ

### ทำไม `git maintenance` ถึงดีกว่าการรัน `git gc` ด้วยมือ

1. **ทำงานเบื้องหลังโดยผู้ใช้ไม่ต้องจำ** — ไม่ต้องมานั่งจำว่าต้องรัน `git gc` เมื่อไหร่
2. **แบ่งงานเป็นชิ้นเล็ก ๆ (incremental)** — ไม่บีบอัดข้อมูลก้อนใหญ่ทีเดียวจนเครื่องหน่วงเหมือน `git gc` แบบเต็มรูปแบบ
3. **มี `prefetch`** ที่ `git gc` ธรรมดาไม่มี ทำให้ `git fetch`/`git pull` ที่ผู้ใช้สั่งเองรู้สึกเร็วขึ้นเพราะข้อมูลส่วนใหญ่ถูกดึงมาล่วงหน้าไว้แล้วเงียบ ๆ
4. **มี `commit-graph`** ที่ช่วยเร่งคำสั่งเกี่ยวกับกราฟ commit ได้อย่างมีนัยสำคัญ โดยเฉพาะใน repository ที่มี commit จำนวนมาก

---

## Step 626: การวัด Performance ด้วย `GIT_TRACE=1` และ `git count-objects -v`

ก่อนจะปรับแต่งอะไร เราต้อง **วัดได้ก่อนว่าปัญหาอยู่ตรงไหน** Git มีเครื่องมือในตัวสำหรับ debug และวัด performance อยู่หลายตัว

### `GIT_TRACE=1` — ดูว่า Git กำลังเรียกอะไรอยู่บ้าง

ตัวแปรสภาพแวดล้อม (environment variable) `GIT_TRACE` เมื่อตั้งเป็น `1` จะทำให้ Git พิมพ์ข้อมูล trace ออกมาที่ stderr บอกว่ากำลังเรียกโปรแกรมภายใน (เช่น `git-upload-pack`, `git-index-pack`) ตัวไหนอยู่บ้าง พร้อมเวลาที่ใช้ในแต่ละขั้นตอน

```bash
GIT_TRACE=1 git fetch origin
```

ตัวอย่างผลลัพธ์ (แบบย่อ):

```
12:03:41.221532 git.c:465               trace: built-in: git fetch origin
12:03:41.223011 run-command.c:663       trace: run_command: git-remote-https origin https://github.com/example/repo.git
12:03:42.884012 run-command.c:663       trace: run_command: git index-pack --stdin ...
```

ข้อมูลนี้ช่วยให้เห็นว่าเวลาส่วนใหญ่ถูกใช้ไปกับขั้นตอนไหน เช่น ขั้นตอนติดต่อ remote, ขั้นตอนแกะ pack file, หรือขั้นตอน checkout

### `GIT_TRACE_PERFORMANCE=1` — วัดเวลาเจาะจงยิ่งขึ้น

ถ้าต้องการเห็นเวลาที่ใช้ในแต่ละฟังก์ชันภายในแบบละเอียดกว่า ใช้:

```bash
GIT_TRACE_PERFORMANCE=1 git status
```

```
12:05:01.001 performance: 0.842349000 s: read_directory (index)
12:05:01.203 performance: 0.201233000 s: refresh_index
12:05:01.204 performance: 1.043912000 s: git command: git status
```

ค่าที่ได้จะบอกได้ทันทีว่าเวลาส่วนใหญ่ถูกใช้ไปกับการอ่าน directory หรือการ refresh index ซึ่งช่วยยืนยันว่าปัญหาคือจำนวนไฟล์ใน working directory มากเกินไปหรือไม่ (ถ้าใช่ คำตอบคือ sparse checkout จาก Step 624)

### `GIT_TRACE_PACKET=1` — ดูข้อมูลระดับ protocol

สำหรับการ debug ปัญหาที่เกี่ยวกับการสื่อสารระหว่าง client กับ server (เช่น fetch/push ช้าผิดปกติ) สามารถดูข้อมูลระดับ packet ของ Git protocol ได้:

```bash
GIT_TRACE_PACKET=1 git fetch origin 2> trace.log
```

ไฟล์ `trace.log` จะมีรายละเอียดของทุก packet ที่แลกเปลี่ยนกัน ซึ่งมีประโยชน์มากตอนจะตรวจสอบว่า Git กำลังใช้ **protocol version ไหน** (ดู Step 628) และมีการส่งข้อมูลที่ไม่จำเป็นออกไปมากน้อยแค่ไหน

### `git count-objects -v` — สรุปสถิติของ Object Database

คำสั่งนี้ให้ภาพรวมของสถานะ object database ในเครื่องได้อย่างรวดเร็ว โดยไม่ต้องรันอะไรหนัก ๆ:

```bash
git count-objects -v
```

ตัวอย่างผลลัพธ์:

```
count: 142
size: 1024
in-pack: 58204
packs: 3
size-pack: 215430
prune-packable: 12
garbage: 0
size-garbage: 0
```

ความหมายของแต่ละบรรทัด:

| ฟิลด์ | ความหมาย |
|---|---|
| `count` | จำนวน **loose object** (object เดี่ยว ๆ ที่ยังไม่ถูกอัดเข้า pack) |
| `size` | ขนาดรวมของ loose object (หน่วย KiB) |
| `in-pack` | จำนวน object ที่ถูกอัดอยู่ใน pack file แล้ว |
| `packs` | จำนวนไฟล์ pack (`.pack`) ที่มีอยู่ |
| `size-pack` | ขนาดรวมของ pack file ทั้งหมด (หน่วย KiB) |
| `prune-packable` | จำนวน loose object ที่มีอยู่ใน pack แล้วด้วย สามารถลบ loose ทิ้งได้เพื่อประหยัดพื้นที่ |
| `garbage` | จำนวนไฟล์ขยะที่ Git ตรวจพบใน object database (เช่นไฟล์เสียหาย) |
| `size-garbage` | ขนาดของไฟล์ขยะเหล่านั้น |

**วิธีอ่านผลลัพธ์เพื่อวินิจฉัยปัญหา:**

- ถ้า `count` (loose object) มีค่าสูงมาก (หลักหมื่นขึ้นไป) แปลว่า repository ยังไม่ได้ repack มานาน → ควรรัน `git maintenance run --task=loose-objects` หรือ `git gc`
- ถ้า `packs` มีจำนวนมาก (มากกว่า 10-20 ไฟล์) แปลว่ามี pack file กระจัดกระจายมาก → ควรรัน `git maintenance run --task=incremental-repack`
- ถ้า `size-pack` ใหญ่ผิดปกติเทียบกับขนาดโค้ดจริง อาจมีไฟล์ binary ขนาดใหญ่แอบซ่อนอยู่ในประวัติ ควรตรวจสอบด้วยเครื่องมืออย่าง `git verify-pack` หรือ `git rev-list --objects --all` ร่วมกับการเรียงตามขนาด (ซึ่งเราเคยพูดถึงแนวทางคล้ายกันตอนเรียนเรื่อง `git filter-repo`)

### เครื่องมือเสริมอื่น ๆ ที่ควรรู้จัก

```bash
git count-objects -v --human-readable   # เหมือน git count-objects -v แต่แสดงขนาดเป็นหน่วยอ่านง่าย (KiB/MiB/GiB) แทนตัวเลข KiB ดิบ
git verify-pack -v .git/objects/pack/pack-xxxx.idx | sort -k3 -n | tail -20   # หา object ที่ใหญ่ที่สุด 20 อันดับใน pack
```

> **ข้อควรรู้:** `git gc` **ไม่มี flag `--dry-run`** ถ้าต้องการดูสถิติของ object database ก่อนตัดสินใจรัน `git gc` ให้ใช้ `git count-objects -v` (ด้านบน) แทน ซึ่งเป็นคำสั่งอ่านอย่างเดียว ไม่กระทบข้อมูลใด ๆ ในเครื่อง

การวัดผลก่อนและหลังการปรับแต่งทุกครั้งเป็นนิสัยที่สำคัญมาก — อย่าเชื่อว่าเทคนิคที่เรียนใน Part นี้จะช่วยได้เสมอโดยไม่วัดผลจริง เพราะบาง repository ขนาดเล็กอาจไม่ได้ประโยชน์อะไรเลยจากเทคนิคเหล่านี้ และการเพิ่มความซับซ้อนโดยไม่จำเป็นอาจทำให้ทีมสับสนมากกว่าเดิม

---

## Step 627: Monorepo Scaling Techniques ภาพรวม

เราได้เรียนเทคนิคระดับ Git client ไปแล้ว (shallow, partial clone, sparse checkout, maintenance) แต่การดูแล **monorepo ระดับองค์กรจริง ๆ** ที่มีทีมนักพัฒนาหลายพันคนทำงานพร้อมกันในที่เดียว ต้องอาศัยเทคนิคเพิ่มเติมอีกหลายชั้นที่กว้างกว่าแค่การตั้งค่า Git ธรรมดา ในที่นี้ขอสรุปภาพรวมไว้ก่อน เพราะเราจะ **เจาะลึกเรื่อง Monorepo แบบเต็มรูปแบบใน Part 82–83** ของหลักสูตรนี้

### ภาพรวมเทคนิคที่ใช้ในระดับองค์กร

1. **Repository Splitting กับ Monorepo — การตัดสินใจเชิงสถาปัตยกรรม**
   องค์กรต้องเลือกระหว่างการแยกโค้ดเป็นหลาย repository เล็ก ๆ (polyrepo) หรือรวมทุกอย่างไว้ใน repository เดียว (monorepo) แต่ละแบบมีข้อดีข้อเสียต่างกันเรื่อง dependency management, atomic commit ข้าม service, และ performance

2. **Build System ที่รู้จัก Dependency Graph (เช่น Bazel, Buck, Nx)**
   Monorepo ขนาดใหญ่มักใช้ build system พิเศษที่รู้ว่าไฟล์ไหนพึ่งพาไฟล์ไหน เพื่อ build/test เฉพาะส่วนที่เปลี่ยนแปลงจริง แทนที่จะ build ทั้งหมดทุกครั้ง

3. **Virtual File System / Lazy Materialization**
   แทนที่จะให้ทุกไฟล์อยู่บนดิสก์จริง บางระบบ (เช่น VFS for Git ที่จะพูดถึงใน Step 629) จะ "หลอก" ให้ระบบปฏิบัติการคิดว่าไฟล์มีอยู่ แต่ดึงเนื้อหาจริงมาต่อเมื่อถูกเปิดใช้งาน

4. **Commit Server / Trunk-Based Development ที่เข้มงวด**
   Monorepo ขนาดใหญ่มักบังคับใช้ trunk-based development ควบคู่กับระบบ CI ที่ตรวจสอบทุก commit ก่อนเข้า main branch อย่างเข้มงวด เพื่อป้องกันไม่ให้ branch หลักพังบ่อย ๆ

5. **Code Ownership และ Access Control ระดับโฟลเดอร์**
   เมื่อทุกทีมอยู่ใน repository เดียวกัน ต้องมีระบบกำหนดสิทธิ์และความรับผิดชอบระดับโฟลเดอร์อย่างชัดเจน (ซึ่งเราเรียนหลักการเบื้องต้นไปแล้วใน Part 36 เรื่อง CODEOWNERS)

6. **Caching และ Remote Execution สำหรับ CI/CD**
   เพื่อไม่ให้ CI ต้อง build ทุกอย่างใหม่ทุกครั้ง องค์กรขนาดใหญ่มักลงทุนกับระบบ cache ผลลัพธ์การ build/test แบบกระจาย (distributed build cache)

### ทำไมต้องเลื่อนไปเจาะลึกใน Part 82-83

เทคนิคเหล่านี้ (โดยเฉพาะข้อ 2, 3, 6) ไม่ใช่ฟีเจอร์ของ Git โดยตรง แต่เป็น **ระบบนิเวศรอบข้าง** ที่ต้องอาศัยความเข้าใจเรื่อง CI/CD, DevOps, และสถาปัตยกรรมซอฟต์แวร์ในภาพกว้างมากกว่าจะเข้าใจได้ในบทเรียนเดียว หลักสูตรนี้จึงเก็บหัวข้อ Monorepo แบบเต็มรูปแบบไว้ใน **เฟส 9 (Step 851-950)** ที่ Part 82-83 ซึ่งเราจะได้เรียนหลังจากมีพื้นฐาน CI/CD และ DevOps ที่แข็งแรงพอแล้วจาก เฟส 7 และเฟส 8

สำหรับตอนนี้ ให้จำหลักการสำคัญไว้เพียงประโยคเดียว:

> **การจัดการ Repository ขนาดใหญ่ในระดับ Git คือจุดเริ่มต้น แต่การจัดการ Monorepo ระดับองค์กรจริง ๆ ต้องอาศัยเครื่องมือและกระบวนการรอบข้าง Git อีกหลายชั้น**

---

## Step 628: Git Protocol v2 ช่วยเรื่อง Performance อย่างไร

หนึ่งในจุดที่คนมองข้ามบ่อยเมื่อพูดถึง performance ของ Git คือ **protocol ที่ใช้สื่อสารระหว่าง client กับ server** ตอน fetch/push/clone

### ปัญหาของ Protocol v0 (Protocol เดิม)

Git protocol เวอร์ชันดั้งเดิม (ที่ไม่มีเลขระบุชัดเจน บางครั้งเรียกว่า v0) มีขั้นตอนการทำงานคือ:

1. เมื่อ client ติดต่อ server เพื่อ fetch, **server จะส่งรายชื่อ ref (branch/tag) ทั้งหมดที่มีมาให้ก่อนเลยทันที** ไม่ว่า client จะสนใจ ref นั้นหรือไม่
2. ถ้า repository มี ref จำนวนมาก (เช่น monorepo ที่มี feature branch ค้างอยู่นับหมื่นอัน หรือมี tag เก่าสะสมมานาน) ขั้นตอนนี้เพียงอย่างเดียวก็ใช้เวลาและ bandwidth มากแล้ว **ก่อนที่การ negotiate object จริง ๆ จะเริ่มด้วยซ้ำ**
3. Client ไม่มีทางบอก server ล่วงหน้าว่า "สนใจแค่ ref ที่ขึ้นต้นด้วย `refs/heads/release-*`" ได้ ต้องรับรายชื่อทั้งหมดมาก่อนเสมอ

### Protocol v2 แก้ปัญหานี้อย่างไร

**Git Protocol version 2** (เริ่มใช้งานได้ตั้งแต่ Git 2.18) ออกแบบใหม่ให้การสื่อสารเป็นแบบ **command-based** ที่ client ร้องขอเฉพาะสิ่งที่ต้องการเท่านั้น จุดเปลี่ยนสำคัญที่สุดคือคำสั่ง **`ls-refs`**

- ใน protocol v2 client จะส่งคำขอ `ls-refs` พร้อมระบุ **ref-prefix** ที่สนใจได้ เช่น ขอเฉพาะ ref ที่ขึ้นต้นด้วย `refs/heads/main`
- Server จะตอบกลับมาเฉพาะ ref ที่ตรงเงื่อนไขเท่านั้น **ไม่ส่งรายชื่อ ref ทั้งหมดที่ไม่เกี่ยวข้องมาให้เสียเวลา**
- สำหรับ repository ที่มี ref จำนวนมาก (หลักหมื่นถึงหลักแสน) การประหยัดขั้นตอนนี้ทำให้เวลาที่ใช้ในการเริ่ม fetch/clone ลดลงอย่างมีนัยสำคัญ โดยเฉพาะกรณีที่ client สนใจแค่ branch เดียว

### วิธีเปิดใช้งาน Protocol v2

Git เวอร์ชันใหม่ (2.26 ขึ้นไป) เปิดใช้ protocol v2 เป็นค่าเริ่มต้นอยู่แล้วเมื่อทั้ง client และ server รองรับ (Git จะ negotiate อัตโนมัติว่าจะใช้ version ไหนตามที่ทั้งสองฝ่ายรองรับสูงสุดร่วมกัน) แต่สามารถบังคับหรือตรวจสอบได้ด้วยตัวเอง:

```bash
# บังคับใช้ protocol v2 อย่างชัดเจนสำหรับคำสั่งเดียว
git -c protocol.version=2 fetch origin

# ตั้งค่าถาวรสำหรับทุก repository ในเครื่อง
git config --global protocol.version 2

# ตรวจสอบว่ากำลังใช้ protocol version ไหนอยู่จริง
GIT_TRACE_PACKET=1 git fetch origin 2>&1 | grep version
```

ตัวอย่างข้อมูลใน trace เมื่อใช้ protocol v2:

```
packet:          git< version 2
packet:          git< agent=git/2.43.0
packet:          git< ls-refs=unborn
packet:          git< fetch=shallow wait-for-done filter
packet:          git< server-option
```

จะเห็นว่า server ประกาศ **capability** ของตัวเองออกมาให้ client เลือกใช้ เช่น `filter` (รองรับ partial clone), `shallow` (รองรับ shallow fetch) ซึ่งเป็นกลไกที่ทำให้ฟีเจอร์ที่เราเรียนใน Step 622-624 ทำงานได้อย่างมีประสิทธิภาพเมื่อใช้ร่วมกับ protocol v2

### ประโยชน์สรุปของ Protocol v2

| ประเด็น | Protocol v0 (เดิม) | Protocol v2 (ใหม่) |
|---|---|---|
| การส่งรายชื่อ ref | ส่งทั้งหมดเสมอ แม้ client ไม่ได้ต้องการ | ส่งเฉพาะที่ client ร้องขอผ่าน `ls-refs` |
| โครงสร้าง protocol | อิงคำสั่งเดียวแบบตายตัว | แบบ command-based ขยายฟีเจอร์ใหม่ได้ง่ายในอนาคต |
| ประกาศความสามารถของ server | จำกัด | ประกาศ capability ได้ยืดหยุ่นกว่า (filter, shallow ฯลฯ) |
| ประสิทธิภาพกับ repo ที่มี ref จำนวนมาก | แย่ลงเรื่อย ๆ ตามจำนวน ref | คงที่ไม่ขึ้นกับจำนวน ref ทั้งหมด (ขึ้นกับจำนวนที่ match เท่านั้น) |

Protocol v2 ทำงานอยู่เบื้องหลังแบบที่ผู้ใช้ทั่วไปแทบไม่ต้องตั้งค่าอะไรเลยในปัจจุบัน แต่การเข้าใจว่ามันทำงานอย่างไรช่วยให้เข้าใจว่าทำไม Git เวอร์ชันใหม่ ๆ ถึงเร็วขึ้นเรื่อย ๆ เมื่อทำงานกับ repository ขนาดใหญ่ที่มี ref จำนวนมาก แม้จะไม่ได้เปลี่ยนแปลงอะไรฝั่ง client เองเลยก็ตาม

---

## Step 629: เครื่องมือเสริมระดับ Enterprise — VFS for Git และ Scalar

เทคนิคทั้งหมดที่เรียนมาใน Step 622-628 เป็นฟีเจอร์ที่มากับ Git core โดยตรง แต่สำหรับองค์กรระดับที่มี monorepo ขนาดหลายร้อยกิกะไบต์จริง ๆ (ระดับเดียวกับ repository ของ Windows หรือ Office ที่ Microsoft ดูแล) แม้แต่ partial clone และ sparse checkout ร่วมกันก็อาจยังไม่พอ Microsoft จึงพัฒนาเครื่องมือเสริมขึ้นมาโดยเฉพาะ

### VFS for Git (เดิมชื่อ GVFS)

**VFS for Git** (Virtual File System for Git, ชื่อเดิมคือ **GVFS — Git Virtual File System**) คือโครงการที่ Microsoft พัฒนาขึ้นราวปี 2017 เพื่อแก้ปัญหาการทำงานกับ repository ของ Windows ที่มีขนาดใหญ่มากเป็นพิเศษ

**แนวคิดหลัก:** แทนที่จะ checkout ไฟล์ทุกไฟล์ลงดิสก์จริงเหมือน sparse checkout ที่เลือกเป็นโฟลเดอร์ VFS for Git ใช้เทคนิค **virtualize ระดับ filesystem driver** กล่าวคือ:

- ระบบปฏิบัติการ **มองเห็น** ไฟล์ทุกไฟล์ในโปรเจกต์เหมือนมันอยู่บนดิสก์จริง (รายชื่อไฟล์ครบถ้วน)
- แต่เนื้อหาไฟล์จริง ๆ **จะถูกดาวน์โหลดมาก็ต่อเมื่อมีโปรแกรม (เช่น text editor, compiler) เปิดอ่านไฟล์นั้นจริง ๆ** เท่านั้น
- ทำได้โดยติดตั้ง filesystem driver พิเศษ (บน Windows ใช้ ProjFS — Windows Projected File System) ที่ดักจับการเรียกอ่านไฟล์และไปดึงเนื้อหามาจาก Git object database หรือ remote แบบโปร่งใส

ข้อดีของแนวทางนี้คือผู้ใช้ **ไม่ต้องรู้ล่วงหน้าว่าจะต้องใช้โฟลเดอร์ไหนบ้าง** เหมือน sparse checkout ที่ต้องเลือกเองล่วงหน้า เพราะ VFS for Git จะดึงมาให้อัตโนมัติตามการใช้งานจริงแบบเรียลไทม์

**ข้อจำกัด:** VFS for Git ผูกกับเทคโนโลยี ProjFS ของ Windows เป็นหลัก แม้จะมีความพยายามรองรับ macOS ในบางช่วง แต่ก็ไม่ได้แพร่หลายเท่าบน Windows และการติดตั้งซับซ้อนกว่าฟีเจอร์มาตรฐานของ Git มาก จึงเหมาะกับองค์กรขนาดใหญ่ที่มีทีม infrastructure คอยดูแลโดยเฉพาะเท่านั้น

### Scalar — วิวัฒนาการที่ใช้งานง่ายกว่า และถูกรวมเข้า Git core แล้ว

หลังจากได้บทเรียนจาก VFS for Git ทีม Microsoft จึงพัฒนา **Scalar** ขึ้นมาเป็นรุ่นถัดไป โดยแนวคิดต่างจาก VFS for Git ตรงที่ **Scalar ไม่ได้ virtualize filesystem** แต่เป็นการ **รวมเอาฟีเจอร์มาตรฐานของ Git ที่เราเรียนมาทั้งหมดใน Part นี้เข้าไว้ด้วยกันเป็นชุดเดียว พร้อมตั้งค่าที่เหมาะสมที่สุดให้อัตโนมัติ** ได้แก่:

- Partial clone (`--filter=blob:none`)
- Sparse checkout (cone mode)
- `git maintenance` (เปิดใช้งานอัตโนมัติพร้อม schedule ที่เหมาะสม)
- Multi-pack-index และ commit-graph (ฟีเจอร์เร่งความเร็วระดับ internals ที่ Scalar เปิดใช้ให้อัตโนมัติ)

จุดสำคัญที่สุดคือ **Scalar ถูกรวมเข้าไปเป็นส่วนหนึ่งของ Git core อย่างเป็นทางการตั้งแต่ Git 2.38** ทำให้ทุกคนที่ติดตั้ง Git เวอร์ชันใหม่สามารถใช้คำสั่ง `git scalar` (หรือ `scalar` แบบ standalone) ได้ทันทีโดยไม่ต้องติดตั้งอะไรเพิ่ม

### ตัวอย่างการใช้งาน Scalar เบื้องต้น

```bash
# Clone repository ขนาดใหญ่ด้วย Scalar (ตั้งค่า partial clone + sparse + maintenance ให้อัตโนมัติ)
scalar clone https://github.com/example/huge-monorepo.git

# หรือใช้ผ่านคำสั่ง git ที่รวมเข้า core แล้ว
git scalar clone https://github.com/example/huge-monorepo.git

# ลงทะเบียน repository ที่มีอยู่แล้วให้ Scalar ดูแล maintenance ให้อัตโนมัติ
git scalar register /path/to/existing-repo

# ดูรายชื่อ repository ที่ Scalar กำลังดูแลอยู่
git scalar list
```

เมื่อ clone ด้วย Scalar แล้ว repository ที่ได้จะถูกตั้งค่าไว้ล่วงหน้าครบทุกอย่างที่เราเรียนมาใน Part นี้ในคราวเดียว โดยผู้ใช้ทั่วไปไม่จำเป็นต้องรู้รายละเอียดของแต่ละฟีเจอร์เลยก็สามารถได้ประสิทธิภาพที่ดีที่สุดสำหรับ repository ขนาดใหญ่ทันที

### ตารางเปรียบเทียบภาพรวม

| เครื่องมือ | แนวทาง | สถานะปัจจุบัน | เหมาะกับ |
|---|---|---|---|
| VFS for Git (GVFS) | Virtualize filesystem ระดับ driver (ProjFS) | ยังใช้งานในบางองค์กร แต่ไม่ใช่แนวทางหลักที่แนะนำแล้ว | Windows monorepo ขนาดใหญ่มากเป็นพิเศษที่มีทีม infra เฉพาะ |
| Scalar | รวมฟีเจอร์มาตรฐาน Git (partial clone, sparse checkout, maintenance) เข้าด้วยกัน | เป็นส่วนหนึ่งของ Git core ตั้งแต่ 2.38 แนวทางที่แนะนำในปัจจุบัน | Monorepo ขนาดใหญ่ทั่วไปที่ต้องการตั้งค่าเดียวจบ |

### สรุปข้อคิดสำคัญของ Step นี้

> เครื่องมือระดับ enterprise เหล่านี้ไม่ได้สร้างเวทมนตร์ใหม่ที่ Git ไม่มี — สิ่งที่มันทำคือ **นำฟีเจอร์ที่เราเพิ่งเรียนมาทั้ง Part นี้ (partial clone, sparse checkout, maintenance) มารวมและตั้งค่าให้เหมาะสมโดยอัตโนมัติ** เพื่อลดภาระของผู้ใช้ทั่วไปที่ไม่ต้องการเจาะลึกรายละเอียดเอง

การเข้าใจพื้นฐานที่เรียนมาใน Step 622-628 จึงสำคัญกว่าการจำวิธีใช้ Scalar อย่างเดียว เพราะเมื่อเข้าใจพื้นฐานแล้ว การ debug ปัญหาที่เกิดขึ้นแม้จะใช้ Scalar อยู่ก็จะทำได้ง่ายขึ้นมาก

---

## Step 630: แบบฝึกหัด — ทดลองเปรียบเทียบ Shallow / Partial Clone / Sparse Checkout

ถึงเวลาลงมือปฏิบัติจริง แบบฝึกหัดนี้จะให้คุณทดลอง clone repository โอเพนซอร์สขนาดใหญ่จริงด้วยวิธีต่าง ๆ แล้วเปรียบเทียบเวลาและขนาดที่ใช้ด้วยตัวเอง เพื่อให้เห็นผลต่างอย่างเป็นรูปธรรม ไม่ใช่แค่อ่านทฤษฎีเฉย ๆ

### เตรียมตัวก่อนเริ่ม

สร้างโฟลเดอร์แยกสำหรับแบบฝึกหัดนี้โดยเฉพาะ:

```bash
mkdir ~/git-course/part-63-performance
cd ~/git-course/part-63-performance
```

เราจะใช้ repository ของ **Linux Kernel** (`torvalds/linux`) เป็นตัวอย่าง เพราะเป็น repository โอเพนซอร์สขนาดใหญ่จริง มีประวัติหลายแสน commit และมีขนาดใหญ่พอที่จะเห็นผลต่างชัดเจน (ถ้าอินเทอร์เน็ตของคุณช้าหรือต้องการ repository ที่เล็กกว่านี้ สามารถเปลี่ยนไปใช้ repository ขนาดกลางอื่นแทนได้ เช่น `microsoft/vscode` หรือ `torvalds/linux` เอง URL: `https://github.com/torvalds/linux.git`)

### แบบฝึกหัดที่ 1: วัดเวลา Full Clone (baseline)

```bash
time git clone https://github.com/torvalds/linux.git full-clone
```

จดบันทึกผลลัพธ์ที่ได้:

```bash
du -sh full-clone/.git
git -C full-clone count-objects -v
```

บันทึกค่า **เวลาที่ใช้ (real)**, **ขนาดของ `.git`**, และจำนวน object ทั้งหมด ไว้เป็นค่าฐาน (baseline) สำหรับเปรียบเทียบ

### แบบฝึกหัดที่ 2: ทดลอง Shallow Clone

```bash
time git clone --depth 1 https://github.com/torvalds/linux.git shallow-clone
du -sh shallow-clone/.git
```

ลองรันคำสั่งต่อไปนี้และสังเกตความแตกต่างจาก full clone:

```bash
cd shallow-clone
git log --oneline          # เห็นแค่กี่ commit?
git log --oneline | wc -l
cat .git/shallow           # ดูว่ามี commit hash อะไรบันทึกไว้เป็นขอบเขต
cd ..
```

**คำถามให้คิด:** ขนาดของ `.git` ต่างจาก full clone กี่เท่า และเวลาที่ใช้ต่างกันแค่ไหน

### แบบฝึกหัดที่ 3: ทดลอง Partial Clone (Blobless)

```bash
time git clone --filter=blob:none https://github.com/torvalds/linux.git partial-clone
du -sh partial-clone/.git
```

ทดลองดูว่าประวัติยังครบหรือไม่ (ต่างจาก shallow clone):

```bash
cd partial-clone
git log --oneline | wc -l    # ควรเห็นจำนวน commit ใกล้เคียง full clone
cd ..
```

จากนั้นลอง checkout ไปยัง commit เก่า ๆ แล้วสังเกตว่า Git ต้องดึงข้อมูลเพิ่มจาก remote หรือไม่ (ลองรันพร้อม `GIT_TRACE_PACKET=1` เพื่อดูว่ามีการติดต่อ remote เพิ่มระหว่าง checkout จริง):

```bash
cd partial-clone
git log --oneline | tail -5           # หา commit hash เก่า ๆ มาสัก 1 อัน
GIT_TRACE_PACKET=1 git checkout <commit-hash-เก่า> 2>&1 | grep -i "want\|fetch" | head -20
cd ..
```

### แบบฝึกหัดที่ 4: ทดลอง Partial Clone + Sparse Checkout ร่วมกัน

```bash
time git clone --filter=blob:none --sparse https://github.com/torvalds/linux.git sparse-clone
cd sparse-clone
du -sh .git
ls                                    # ควรเห็นแค่ไฟล์ที่ root เท่านั้น

git sparse-checkout set drivers/usb
ls                                    # ตอนนี้ควรเห็นโฟลเดอร์ drivers/ ที่มีแค่ usb/ อยู่ข้างใน
du -sh .                              # เทียบขนาด working directory กับ full clone
cd ..
```

เปรียบเทียบขนาดของ working directory (ไม่นับ `.git`) ระหว่าง `full-clone` กับ `sparse-clone`:

```bash
du -sh full-clone --exclude=.git 2>/dev/null || du -sh full-clone/*
du -sh sparse-clone/*
```

### แบบฝึกหัดที่ 5: ทดลอง `git maintenance` และวัดผลก่อน-หลัง

ใช้ `full-clone` ที่ทำไว้ในแบบฝึกหัดที่ 1:

```bash
cd full-clone
git count-objects -v                          # บันทึกค่าก่อน
GIT_TRACE_PERFORMANCE=1 git log --oneline -20 2>&1 | tail -5   # จับเวลาก่อน

git maintenance run --task=commit-graph
git maintenance run --task=gc

git count-objects -v                          # บันทึกค่าหลัง เทียบดูว่า packs/count เปลี่ยนไปอย่างไร
GIT_TRACE_PERFORMANCE=1 git log --oneline -20 2>&1 | tail -5   # จับเวลาหลัง เทียบว่าเร็วขึ้นไหม
cd ..
```

### สรุปผลการทดลองลงในตาราง

ทำตารางสรุปผลการทดลองของตัวเอง (เขียนใส่ไฟล์ text หรือกระดาษก็ได้) ในรูปแบบนี้:

| วิธี Clone | เวลาที่ใช้ (วินาที) | ขนาด `.git` | จำนวน commit ที่เห็นใน `git log` |
|---|---|---|---|
| Full clone | ... | ... | ... |
| Shallow (`--depth 1`) | ... | ... | ... |
| Partial (`--filter=blob:none`) | ... | ... | ... |
| Partial + Sparse | ... | ... | ... |

### คำถามท้ายแบบฝึกหัด (ตอบด้วยตัวเองเพื่อเช็คความเข้าใจ)

1. ทำไม partial clone ถึงมีจำนวน commit เท่ากับ full clone แต่ shallow clone ถึงไม่เท่า
2. ถ้าคุณลบ remote (`git remote remove origin`) ออกจาก partial clone แล้วลอง checkout ไปยัง commit ที่ยังไม่เคยดึง blob มาก่อน จะเกิดอะไรขึ้น เพราะเหตุใด (ลองทำจริงเพื่อดูข้อความ error)
3. ในสถานการณ์ที่ทีมของคุณมีแค่ CI/CD pipeline ที่ build จาก commit ล่าสุดอย่างเดียว ไม่มีใครต้องดู blame หรือ log ย้อนหลัง ควรเลือกใช้ shallow clone หรือ partial clone และเพราะเหตุใด
4. ถ้า repository ของทีมคุณมีขนาดแค่ 50 MB และมี commit ไม่ถึง 5,000 commit ควรใช้เทคนิคใน Part นี้เลยหรือไม่ เพราะเหตุใด

### Checklist ก่อนไป Part 64

ก่อนไปต่อ Part 64 ให้ตรวจสอบว่าคุณ:

- [ ] อธิบายได้ว่า repository ขนาดใหญ่มาก ๆ มีปัญหาอะไรบ้าง และเพราะเหตุใด
- [ ] ใช้ `git clone --depth <n>` และเข้าใจข้อจำกัดของ shallow clone ได้
- [ ] ใช้ `git clone --filter=blob:none` และอธิบายความแตกต่างจาก shallow clone ได้
- [ ] ตั้งค่า `git sparse-checkout` แบบ cone mode เพื่อเลือกทำงานเฉพาะบางโฟลเดอร์ได้
- [ ] เปิดใช้งาน `git maintenance start` และอธิบายหน้าที่ของแต่ละ task ได้
- [ ] ใช้ `GIT_TRACE=1`, `GIT_TRACE_PERFORMANCE=1` และ `git count-objects -v` เพื่อวินิจฉัยปัญหา performance ได้
- [ ] เข้าใจภาพรวมว่า Monorepo scaling ต้องอาศัยเทคนิคเพิ่มเติมนอกเหนือจาก Git core (จะเจาะลึกใน Part 82-83)
- [ ] อธิบายได้ว่า Git Protocol v2 ช่วยลดข้อมูลตอน negotiate อย่างไรผ่านคำสั่ง `ls-refs`
- [ ] รู้จัก VFS for Git และ Scalar ในระดับภาพรวม และเข้าใจว่า Scalar คือการรวมฟีเจอร์มาตรฐานของ Git เข้าด้วยกัน
- [ ] ทำแบบฝึกหัดเปรียบเทียบ shallow/partial/sparse clone ด้วยตัวเองและบันทึกผลลัพธ์จริงได้

---

## สรุป Part 63

ใน Part นี้เราได้เรียนรู้ว่า:

1. Repository ขนาดใหญ่มาก ๆ ก่อปัญหาหลายด้าน ทั้งเวลา clone ที่นานขึ้น, คำสั่งพื้นฐานอย่าง `git status` ที่ช้าลง, และการ negotiate ระหว่าง fetch/push ที่หนักขึ้นตามจำนวน ref และ commit
2. **Shallow Clone** (`--depth <n>`) ตัดจำนวน commit ในประวัติออก เหมาะกับ CI/CD ที่ต้องการแค่โค้ดปัจจุบัน แต่แลกมาด้วยข้อจำกัดเรื่อง `blame`, `log`, และการคำนวณ merge-base
3. **Partial Clone** (`--filter=blob:none` / `tree:0` / `blob:limit=<n>`) ตัดเนื้อหาไฟล์ออกแทนที่จะตัดประวัติ ทำให้ยังมี `git log` ครบสมบูรณ์ พร้อมดึง object ที่ขาดมาแบบ lazy เมื่อจำเป็น
4. **Sparse Checkout** ในโหมด cone ช่วยให้ working directory มีเฉพาะโฟลเดอร์ที่คุณต้องการทำงานจริง ลดภาระของ filesystem และ `git status` ได้อย่างมาก โดยเฉพาะเมื่อใช้ร่วมกับ partial clone
5. **`git maintenance`** เป็นระบบดูแล repository อัตโนมัติเบื้องหลัง ครอบคลุมงาน `gc`, `commit-graph`, `prefetch`, `loose-objects`, `incremental-repack`, และ `pack-refs` ที่ทำงานตามตารางเวลาโดยไม่รบกวนผู้ใช้
6. เครื่องมือวัดผลอย่าง `GIT_TRACE=1`, `GIT_TRACE_PERFORMANCE=1`, `GIT_TRACE_PACKET=1` และ `git count-objects -v` ช่วยให้วินิจฉัยได้ว่าปัญหาที่แท้จริงอยู่ตรงไหนก่อนจะเลือกใช้เทคนิคใด
7. **Monorepo scaling** ระดับองค์กรต้องอาศัยเทคนิคเพิ่มเติมนอกเหนือจาก Git core เช่น build system ที่รู้จัก dependency graph และ distributed build cache ซึ่งเราจะเจาะลึกใน Part 82-83
8. **Git Protocol v2** ลดข้อมูลที่แลกเปลี่ยนตอน negotiate ด้วยคำสั่ง `ls-refs` ที่ให้ client ร้องขอเฉพาะ ref ที่สนใจ แทนที่จะรับรายชื่อ ref ทั้งหมดเหมือน protocol เดิม
9. **VFS for Git (GVFS)** และ **Scalar** คือเครื่องมือระดับ enterprise ที่ Microsoft พัฒนาขึ้นเพื่อดูแล monorepo ขนาดมหาศาล โดย Scalar ได้ถูกรวมเข้าเป็นส่วนหนึ่งของ Git core ตั้งแต่เวอร์ชัน 2.38 แล้ว
10. เราได้ลงมือทดลองเปรียบเทียบวิธี clone ต่าง ๆ ด้วยตัวเองจริง ทั้งเวลาที่ใช้และขนาดที่ประหยัดได้

**ต่อไป:** [Part 64: Git Worktree: ทำงานหลาย Branch พร้อมกัน](./part-064-git-worktree.md)
