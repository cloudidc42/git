# Part 62: Git LFS: จัดการไฟล์ขนาดใหญ่

> **Step ในหลักสูตรนี้:** Step 611–620
> **เฟส:** 6 — Git ขั้นสูงระดับ Internals, Hooks, LFS และ Performance
> **เป้าหมายของ Part นี้:** เข้าใจว่าทำไม Git แบบปกติถึง "เอาไม่อยู่" กับไฟล์ binary ขนาดใหญ่ เข้าใจหลักการทำงานของ Git LFS ผ่านกลไก clean/smudge filter ที่เราเคยเรียนมาก่อนหน้านี้ ติดตั้งและใช้งาน Git LFS ได้จริงตั้งแต่ track ไฟล์ commit push จนถึง clone repo ที่มี LFS พร้อมเข้าใจเรื่อง storage/bandwidth quota และวิธี migrate ไฟล์เก่าเข้า LFS ทีหลังอย่างปลอดภัย

---

## สารบัญของ Part นี้

- Step 611: ปัญหาที่ Git LFS แก้ — ทำไมไฟล์ binary ขนาดใหญ่ถึงทำให้ Git พัง
- Step 612: Git LFS ทำงานอย่างไรในหลักการ (Pointer File + Clean/Smudge Filter)
- Step 613: ติดตั้งและเปิดใช้งาน Git LFS (`git lfs install`)
- Step 614: Track ไฟล์ประเภทไหนด้วย LFS (`git lfs track`)
- Step 615: อ่านและเข้าใจ `.gitattributes` ที่ LFS สร้างให้อัตโนมัติ
- Step 616: Commit/Push ไฟล์ที่ track ด้วย LFS — Workflow เหมือนเดิมทุกอย่าง
- Step 617: Clone Repo ที่มี LFS — ข้อควรระวังที่พลาดกันบ่อย
- Step 618: LFS Storage และ Bandwidth Quota บน GitHub/GitLab
- Step 619: Migrate ไฟล์เก่าที่ track แบบปกติเข้า LFS ด้วย `git lfs migrate import`
- Step 620: แบบฝึกหัด — ตั้งค่า Git LFS ให้โปรเจกต์ที่มีไฟล์รูปภาพขนาดใหญ่แบบครบวงจร

---

## Step 611: ปัญหาที่ Git LFS แก้ — ทำไมไฟล์ binary ขนาดใหญ่ถึงทำให้ Git พัง

### จุดแข็งของ Git ที่กลายเป็นจุดอ่อนกับไฟล์ binary

ย้อนกลับไปที่ Part 01 เราพูดถึงหลักการสำคัญที่สุดข้อหนึ่งของ Git คือ **Git คิดถึงข้อมูลในรูปแบบ Snapshot ของทั้งโปรเจกต์ ไม่ใช่ diff ของไฟล์ทีละไฟล์** และ Git ก็ฉลาดพอที่จะไม่เก็บไฟล์ที่ไม่เปลี่ยนแปลงซ้ำ รวมถึงยังมีกลไก **delta compression** (การบีบอัดโดยเก็บแค่ "ผลต่าง" ระหว่างไฟล์เวอร์ชันต่าง ๆ ที่คล้ายกัน) อยู่ในขั้นตอน `git gc` เพื่อบีบอัด object ที่คล้ายกันให้ใช้พื้นที่น้อยลง

กลไก delta compression นี้ **ทำงานได้ดีมากกับไฟล์ text** เช่น source code เพราะไฟล์ text ที่ถูกแก้ไขเล็กน้อยจะมีโครงสร้างไบต์ที่คล้ายกันมากระหว่างเวอร์ชัน ทำให้ Git คำนวณผลต่างและบีบอัดได้อย่างมีประสิทธิภาพสูง

แต่กับ **ไฟล์ binary ขนาดใหญ่** เช่น:

- ไฟล์วิดีโอ (`.mp4`, `.mov`, `.avi`)
- ไฟล์ภาพความละเอียดสูงหรือไฟล์ต้นฉบับงานออกแบบ (`.psd`, `.ai`, `.sketch`, `.tiff`)
- ไฟล์โมเดล 3 มิติ (`.fbx`, `.blend`, `.obj`, `.max`)
- ไฟล์เสียงคุณภาพสูง (`.wav`, `.flac`)
- ไฟล์ archive หรือ dataset (`.zip`, `.tar.gz`, `.csv` ขนาดใหญ่, ไฟล์ model ของ Machine Learning เช่น `.pkl`, `.h5`, `.onnx`)

กลไกเดียวกันนี้กลับกลายเป็นปัญหาใหญ่ ด้วยเหตุผลหลัก ๆ ดังนี้

### 1. Delta Compression ทำงานไม่ดีกับ Binary

ไฟล์ binary ที่ถูกบันทึกทับใหม่ (เช่น export วิดีโอใหม่ หรือ save ไฟล์ PSD ใหม่) มักจะมี**โครงสร้างไบต์ที่เปลี่ยนไปทั้งไฟล์** แม้เนื้อหาจะดูคล้ายเดิมในสายตามนุษย์ก็ตาม เพราะไฟล์ประเภทนี้มักถูกบีบอัดภายในตัวมันเองอยู่แล้ว (เช่น `.mp4` ใช้ video codec บีบอัดข้อมูล, `.psd` อาจมี metadata และ layer structure ที่เปลี่ยนตำแหน่งไบต์ทั้งไฟล์แม้แก้ไขนิดเดียว)

ผลคือ algorithm การหา delta ของ Git **แทบไม่สามารถหาความคล้ายกันระหว่างเวอร์ชันได้เลย** สุดท้าย Git ก็ต้องเก็บไฟล์ทั้งไฟล์ซ้ำใหม่ทุกครั้งที่มีการ commit เวอร์ชันใหม่ ไม่ต่างจากการไม่มี compression เลย

### 2. Repository บวมอย่างรวดเร็ว (Repository Bloat)

ลองจินตนาการทีมงานตัดต่อวิดีโอที่ commit ไฟล์วิดีโอต้นฉบับ 500 MB แล้วแก้ไข export ใหม่สัปดาห์ละครั้งเป็นเวลา 1 ปี:

```
ไฟล์ video.mp4 เวอร์ชัน 1  : 500 MB  (เก็บเต็มไฟล์)
ไฟล์ video.mp4 เวอร์ชัน 2  : 510 MB  (เก็บเต็มไฟล์ใหม่ทั้งหมด เพราะ delta หาไม่เจอ)
ไฟล์ video.mp4 เวอร์ชัน 3  : 495 MB  (เก็บเต็มไฟล์ใหม่ทั้งหมดอีกครั้ง)
...
รวม 52 สัปดาห์ ≈ 26 GB สำหรับไฟล์เดียว ที่ใช้งานจริงแค่เวอร์ชันล่าสุดเวอร์ชันเดียว
```

และนี่คือแค่ไฟล์เดียว ถ้าโปรเจกต์มีไฟล์ asset แบบนี้หลายสิบหรือหลายร้อยไฟล์ ขนาด `.git` directory จะบวมขึ้นเป็นหลัก GB หรือหลัก TB ได้อย่างรวดเร็ว **ทั้งที่ทุกคนต้องการแค่ไฟล์เวอร์ชันล่าสุดในการทำงานจริง แทบไม่มีใครย้อนไปดูไฟล์วิดีโอเวอร์ชันเก่า ๆ เลย**

### 3. Clone และ Fetch ช้าลงมหาศาล

เพราะ Git ต้อง **ดาวน์โหลดประวัติทั้งหมด (full history)** มาไว้ในเครื่อง (ตามหลักการ Distributed VCS ที่เราเรียนใน Part 01) ทุกครั้งที่มีคน `git clone` repo ใหม่ เขาจะต้องดาวน์โหลด **ทุกเวอร์ชันของไฟล์ binary ทุกไฟล์ที่เคย commit มา** แม้เขาจะสนใจแค่ไฟล์เวอร์ชันล่าสุดก็ตาม

ผลลัพธ์ที่พบได้จริง:

| สถานการณ์ | ขนาด Repo (ประมาณการ) | เวลา Clone (เน็ตทั่วไป) |
|---|---|---|
| โปรเจกต์ source code ล้วน ๆ | 50–200 MB | ไม่กี่วินาทีถึงไม่กี่สิบวินาที |
| โปรเจกต์เกมที่เก็บไฟล์ asset ปกติใน Git | 20–100 GB | หลายชั่วโมงหรือ clone ไม่สำเร็จเลย |
| โปรเจกต์เดียวกันหลัง migrate ไปใช้ Git LFS | 200–500 MB (เฉพาะ pointer + LFS เวอร์ชันล่าสุดที่ต้องใช้) | ไม่กี่นาที |

### 4. CI/CD และ Backup ช้าและแพงขึ้น

ทุก pipeline ที่ต้อง checkout repo, ทุก backup ของ server, ทุกการ mirror repo ไปอีกที่หนึ่ง ล้วนต้องแบกรับขนาดข้อมูลที่บวมขึ้นนี้ไปด้วย ทำให้ต้นทุนทั้งด้านเวลาและพื้นที่จัดเก็บสูงขึ้นเรื่อย ๆ แบบไม่มีเพดาน

### สรุปปัญหาในหนึ่งประโยค

> **Git ถูกออกแบบมาให้ยอดเยี่ยมกับไฟล์ text ที่เปลี่ยนแปลงทีละนิดและมี delta ที่ดี แต่เมื่อเจอไฟล์ binary ขนาดใหญ่ที่เปลี่ยนแปลงทั้งไฟล์ทุกครั้ง Git จะเก็บทุกเวอร์ชันแบบเต็มไฟล์ตลอดไป ทำให้ repository บวมและช้าลงเรื่อย ๆ แบบที่แก้ไม่ได้ด้วยวิธีปกติ**

**Git LFS (Large File Storage)** คือส่วนขยาย (extension) อย่างเป็นทางการของ Git ที่ถูกสร้างขึ้นมาเพื่อแก้ปัญหานี้โดยเฉพาะ โดยแยกการเก็บ "เนื้อหาไฟล์จริง" ของไฟล์ขนาดใหญ่ออกจาก Git repository หลัก แล้วเก็บไว้ในที่เก็บข้อมูลแยกต่างหากแทน ซึ่งเราจะเจาะลึกหลักการทำงานใน Step ถัดไป

---

## Step 612: Git LFS ทำงานอย่างไรในหลักการ (Pointer File + Clean/Smudge Filter)

### ต่อยอดจากเรื่อง Clean/Smudge Filter

ก่อนหน้านี้เราเคยพูดถึงกลไก **filter driver** ของ Git ที่ทำงานผ่านคู่คำสั่ง **clean** และ **smudge** ซึ่งถูกกำหนดไว้ในไฟล์ `.gitattributes` หลักการพื้นฐานของมันคือ:

- **smudge filter** ทำงานตอน **checkout** — แปลงข้อมูลที่เก็บอยู่ใน Git object ให้กลายเป็นไฟล์ในรูปแบบที่ใช้งานได้จริงใน working directory
- **clean filter** ทำงานตอน **add/commit** — แปลงไฟล์จาก working directory ให้อยู่ในรูปแบบที่จะถูกเก็บลงใน Git object

Git LFS คือ **ตัวอย่างการใช้งาน clean/smudge filter ที่ซับซ้อนและมีประโยชน์ที่สุดตัวหนึ่ง** ในระบบนิเวศ Git ทั้งหมด

### หลักการทำงานของ Git LFS แบบเจาะลึก

เมื่อไฟล์ถูก track ด้วย Git LFS (ผ่าน `.gitattributes` ที่เราจะดูใน Step 615) ทุกครั้งที่คุณ `git add` ไฟล์นั้น กลไก **clean filter ของ LFS** จะทำงานดังนี้:

1. อ่านเนื้อหาไฟล์จริงทั้งหมด (เช่น ไฟล์ `photo.psd` ขนาด 300 MB)
2. คำนวณ **SHA-256 hash** ของเนื้อหาไฟล์นั้น
3. คัดลอกเนื้อหาไฟล์จริงไปเก็บไว้ใน **โฟลเดอร์ cache ของ LFS ในเครื่อง** (`.git/lfs/objects/`) โดยจัดเก็บตาม hash
4. สร้าง **"pointer file"** — ไฟล์ text ขนาดเล็กมาก (แค่ไม่กี่ร้อยไบต์) ที่มีข้อมูลอ้างอิงไปยังไฟล์จริง
5. ส่ง **pointer file** นี้เข้าไปเก็บใน Git object แทนไฟล์จริง

หน้าตาของ pointer file จะประมาณนี้ (นี่คือสิ่งที่ **ถูกเก็บจริงใน Git repository**):

```
version https://git-lfs.github.com/spec/v1
oid sha256:4d7a214614ab2935c943f9e0ff69d22eadbb8f32b1258daaa5e2ca24d17e239
size 314572800
```

สังเกตว่า:

- `version` — ระบุ specification ของ LFS pointer format ที่ใช้
- `oid` (object id) — hash แบบ SHA-256 ของเนื้อหาไฟล์จริง ใช้เป็น "ที่อยู่" อ้างอิงไปหาไฟล์จริง
- `size` — ขนาดไฟล์จริงเป็นไบต์ (ในตัวอย่างนี้คือ 300 MB)

ไฟล์ pointer นี้มีขนาดเพียง **~130 ไบต์เท่านั้น** ไม่ว่าไฟล์จริงจะมีขนาด 1 MB หรือ 10 GB ก็ตาม!

### เมื่อ Checkout กลับมา (Smudge Filter)

ในทางกลับกัน เมื่อคุณ `git checkout` หรือสลับ branch กลับมา **smudge filter ของ LFS** จะทำงานดังนี้:

1. อ่าน pointer file จาก Git object
2. อ่านค่า `oid` จากใน pointer file
3. ไปค้นหาไฟล์จริงที่มี hash ตรงกันใน **local LFS cache** (`.git/lfs/objects/`) ก่อน
4. ถ้าไม่มีในเครื่อง ให้ **ดาวน์โหลดไฟล์จริงจาก LFS server** (ผ่าน HTTPS)
5. นำไฟล์จริงมาวางไว้ใน working directory แทนที่ pointer file

กระบวนการทั้งหมดนี้เกิดขึ้น **อัตโนมัติและโปร่งใส** ผู้ใช้งานแทบไม่รู้สึกว่ามีอะไรพิเศษเกิดขึ้นเลย ตราบใดที่ติดตั้ง `git-lfs` ไว้ในเครื่องแล้ว

### ภาพรวมสถาปัตยกรรม: Git Repo แยกจาก LFS Storage

จุดสำคัญที่สุดที่ต้องเข้าใจคือ **Git LFS แยกที่เก็บข้อมูลออกเป็น 2 ส่วน**:

```
                     ┌─────────────────────────────┐
                     │   Git Repository (ปกติ)      │
                     │   เก็บแค่ "pointer file"      │
                     │   ขนาดเล็กมาก ไม่กี่ร้อยไบต์   │
                     │   ต่อไฟล์ ทุกเวอร์ชัน          │
                     └───────────────┬─────────────┘
                                     │ อ้างอิงด้วย oid (SHA-256)
                                     ▼
                     ┌─────────────────────────────┐
                     │   LFS Storage (แยกต่างหาก)   │
                     │   เก็บเนื้อหาไฟล์จริง          │
                     │   (บน GitHub/GitLab จะเป็น    │
                     │   พื้นที่เก็บข้อมูลคนละส่วนกับ  │
                     │   Git repo หลัก)              │
                     └─────────────────────────────┘
```

เมื่อคุณ `git push` โปรเจกต์ที่มี LFS:

- **pointer file** จะถูก push ไปที่ Git server ตามปกติ (เร็วมาก เพราะมีขนาดเล็ก)
- **เนื้อหาไฟล์จริง** จะถูกอัปโหลดแยกต่างหากไปยัง **LFS server** ผ่าน HTTPS โดย Git LFS client (เกิดขึ้นผ่าน pre-push hook ที่ `git lfs install` ติดตั้งไว้ให้อัตโนมัติ)

ผลลัพธ์คือ **ประวัติ Git ยังคงมีทุก commit ทุกเวอร์ชันของไฟล์เหมือนเดิมทุกประการ** (คุณยัง `git log`, `git checkout` ไปเวอร์ชันเก่าได้ตามปกติ) แต่ **ขนาดของ `.git` directory ที่ต้อง clone ทันทีเล็กลงมหาศาล** เพราะเก็บแค่ pointer ไม่ใช่ไฟล์จริงทุกเวอร์ชัน (ไฟล์จริงในเวอร์ชันเก่า ๆ ที่ไม่ได้ใช้งานจะถูกดาวน์โหลดจาก LFS server ก็ต่อเมื่อคุณ checkout ไปยัง commit นั้นจริง ๆ)

### เปรียบเทียบให้เห็นภาพชัด

| | Git ปกติ (ไม่มี LFS) | Git + Git LFS |
|---|---|---|
| สิ่งที่เก็บใน `.git` | เนื้อหาไฟล์จริงทุกเวอร์ชัน | pointer file (ไม่กี่ร้อยไบต์) ทุกเวอร์ชัน |
| เนื้อหาไฟล์จริงอยู่ที่ไหน | อยู่ใน Git object ทั้งหมด | อยู่ใน LFS storage แยกต่างหาก + local cache |
| ขนาด repo เมื่อ clone | โตขึ้นเรื่อย ๆ ตามทุกเวอร์ชันที่เคย commit | เล็กและคงที่มากขึ้น เพราะ pointer มีขนาดคงที่ |
| ต้องติดตั้งอะไรเพิ่ม | ไม่ต้อง | ต้องติดตั้ง `git-lfs` แยกต่างหาก |
| Diff ของไฟล์ binary | Git แสดง "Binary files differ" | เหมือนกัน (LFS ไม่ได้ช่วยเรื่อง diff เนื้อหา binary) |

ในหัวข้อถัดไป เราจะเริ่มลงมือติดตั้งและใช้งาน Git LFS จริง ๆ

---

## Step 613: ติดตั้งและเปิดใช้งาน Git LFS (`git lfs install`)

### Git LFS ไม่ได้ติดตั้งมาพร้อม Git

สิ่งสำคัญที่ต้องรู้ก่อนคือ **`git-lfs` เป็นโปรแกรมแยกต่างหากจาก `git`** ไม่ได้ถูกติดตั้งมาให้อัตโนมัติเมื่อคุณติดตั้ง Git (ต่างจากฟีเจอร์หลักของ Git เช่น branch, merge ที่มีอยู่ในตัว) คุณต้องติดตั้งเพิ่มเอง

### วิธีติดตั้งตามระบบปฏิบัติการ

**macOS (ผ่าน Homebrew):**

```bash
brew install git-lfs
```

**Windows:**

- ถ้าติดตั้ง **Git for Windows** เวอร์ชันใหม่ ๆ มักจะมีตัวเลือกให้ติดตั้ง Git LFS มาด้วยระหว่างขั้นตอนติดตั้งอยู่แล้ว
- หรือดาวน์โหลด installer แยกได้จาก https://git-lfs.com

**Linux (Debian/Ubuntu):**

```bash
sudo apt-get install git-lfs
```

**Linux (RHEL/CentOS/Fedora):**

```bash
sudo dnf install git-lfs
```

### ตรวจสอบว่าติดตั้งสำเร็จ

```bash
git lfs version
```

ผลลัพธ์ที่ควรเห็น:

```
git-lfs/3.5.1 (GitHub; linux amd64; go 1.21.5)
```

ถ้าเห็น error ประมาณ `git: 'lfs' is not a git command` แปลว่ายังติดตั้งไม่สำเร็จ หรือ `git-lfs` ไม่ได้อยู่ใน `PATH`

### ขั้นตอนสำคัญที่มักถูกลืม: `git lfs install`

หลังติดตั้งโปรแกรม `git-lfs` เสร็จแล้ว **ยังมีอีกขั้นตอนหนึ่งที่จำเป็นมาก** คือการรัน:

```bash
git lfs install
```

คำสั่งนี้ทำหน้าที่ **ลงทะเบียน Git LFS เข้ากับการตั้งค่า Git ระดับ global ของเครื่องคุณ** โดยเฉพาะ:

1. เพิ่ม filter driver ชื่อ `lfs` เข้าไปในไฟล์ global git config (`~/.gitconfig`) เพื่อให้ Git รู้จักคำสั่ง clean/smudge ของ LFS
2. ทำให้ Git สามารถแปลความหมาย attribute `filter=lfs` ใน `.gitattributes` ได้ (ซึ่งเราจะเห็นใน Step 615)

ลองดูผลลัพธ์ใน global config หลังรันคำสั่งนี้:

```bash
git config --global --list | grep lfs
```

```
filter.lfs.clean=git-lfs clean -- %f
filter.lfs.smudge=git-lfs smudge -- %f
filter.lfs.process=git-lfs filter-process
filter.lfs.required=true
```

จะเห็นว่ามันตั้งค่า `filter.lfs.clean`, `filter.lfs.smudge` ตรงตามหลักการ clean/smudge filter ที่อธิบายใน Step 612 เป๊ะ ๆ

**คุณต้องรัน `git lfs install` แค่ครั้งเดียวต่อเครื่อง** (ไม่ใช่ต่อ repo) เพราะมันตั้งค่าระดับ global ไว้ให้แล้ว หลังจากนี้ไม่ว่าคุณจะ clone หรือสร้าง repo ใหม่กี่อันบนเครื่องเดียวกัน ก็ไม่ต้องรันซ้ำอีก

### รูปแบบอื่น ๆ ของคำสั่ง `install`

| คำสั่ง | ความหมาย |
|---|---|
| `git lfs install` | ตั้งค่าระดับ global (ค่าเริ่มต้น) |
| `git lfs install --local` | ตั้งค่าเฉพาะ repo ปัจจุบัน (เขียนลงใน `.git/config` แทน `~/.gitconfig`) มีประโยชน์เมื่อต้องการควบคุมแยกเป็นรายโปรเจกต์ |
| `git lfs install --system` | ตั้งค่าระดับ system-wide สำหรับผู้ใช้ทุกคนบนเครื่อง (ต้องมีสิทธิ์ admin/root) |
| `git lfs install --force` | บังคับเขียนทับการตั้งค่าเดิม ใช้เมื่อ config เดิมเสียหายหรือถูกแก้ไขผิดพลาด |
| `git lfs uninstall` | ถอนการตั้งค่า filter ออกจาก global config (ไม่ได้ถอนโปรแกรม แค่ปิดการทำงานของ filter) |

### เช็คลิสต์ก่อนไปต่อ

ก่อนจะไป track ไฟล์แรกด้วย Git LFS ให้แน่ใจว่า:

```bash
git lfs version      # ต้องแสดงเลขเวอร์ชัน ไม่ error
git lfs env           # แสดงรายละเอียด environment ของ LFS ทั้งหมด ใช้ debug ได้
```

`git lfs env` เป็นคำสั่งที่มีประโยชน์มากเวลาต้อง debug ปัญหา เพราะมันแสดงทั้ง endpoint ของ LFS server ที่กำลังใช้, เวอร์ชัน git-lfs, การตั้งค่า filter ทั้งหมด และ path ของ cache ในเครื่อง

---

## Step 614: Track ไฟล์ประเภทไหนด้วย LFS (`git lfs track`)

### หลักการ: LFS ไม่ได้เปิดใช้งานอัตโนมัติกับทุกไฟล์

Git LFS **ไม่ได้จับทุกไฟล์ binary มาจัดการให้อัตโนมัติ** คุณต้องบอก Git อย่างชัดเจนว่า **ไฟล์ประเภทไหน (หรือไฟล์ไหนเป็นการเฉพาะ) ที่ต้องการให้ LFS จัดการ** ผ่านคำสั่ง `git lfs track`

### วิธีใช้งานพื้นฐาน

สมมติโปรเจกต์งานออกแบบของคุณมีไฟล์ `.psd` จำนวนมาก:

```bash
git lfs track "*.psd"
```

ผลลัพธ์:

```
Tracking "*.psd"
```

คำสั่งนี้จะเขียน pattern `*.psd` ลงในไฟล์ `.gitattributes` ที่ root ของ repo โดยอัตโนมัติ (ถ้ายังไม่มีไฟล์นี้ Git LFS จะสร้างให้ใหม่)

### Track หลายประเภทไฟล์พร้อมกัน

```bash
git lfs track "*.psd"
git lfs track "*.ai"
git lfs track "*.mp4"
git lfs track "*.mov"
git lfs track "*.zip"
git lfs track "*.fbx"
```

หรือจะ track แบบเจาะจงเฉพาะโฟลเดอร์ก็ได้ เช่น:

```bash
git lfs track "assets/videos/*.mp4"
git lfs track "models/**/*.fbx"
```

รูปแบบ pattern ที่ใช้ใน `git lfs track` เป็น **glob pattern แบบเดียวกับที่ใช้ใน `.gitignore`** ทุกประการ เพราะสุดท้ายมันก็ไปเขียนเป็น pattern ใน `.gitattributes` ที่ใช้ syntax เดียวกัน

### Track ไฟล์เจาะจงเป็นรายไฟล์

บางครั้งคุณอาจต้องการ track แค่ไฟล์เดียวที่ใหญ่มาก โดยไม่อยากให้ทุกไฟล์นามสกุลเดียวกันถูก track ไปด้วย:

```bash
git lfs track "assets/hero-video-4k.mp4"
```

### ดูรายการ pattern ที่ track อยู่ทั้งหมด

```bash
git lfs track
```

```
Listing tracked patterns
    *.psd (.gitattributes)
    *.ai (.gitattributes)
    *.mp4 (.gitattributes)
    *.mov (.gitattributes)
    *.zip (.gitattributes)
    *.fbx (.gitattributes)
Listing excluded patterns
```

### เลิก Track ไฟล์ด้วย `git lfs untrack`

ถ้าต้องการเอาไฟล์ประเภทใดออกจากการจัดการของ LFS:

```bash
git lfs untrack "*.zip"
```

คำสั่งนี้จะ **ลบ pattern ออกจาก `.gitattributes`** แต่ **ไม่ได้ทำให้ไฟล์ที่ commit ไปแล้วในอดีตกลับมาเป็น pointer หรือไม่ใช่ pointer โดยอัตโนมัติ** — มันมีผลแค่กับไฟล์ที่จะถูก add/commit ใหม่ในอนาคตเท่านั้น การจัดการไฟล์ที่ track ไปแล้วในประวัติเก่าต้องใช้ `git lfs migrate` ซึ่งจะพูดถึงใน Step 619

### หลักการเลือกว่าควร Track อะไรบ้าง (แนวปฏิบัติที่ดี)

| ควร Track ด้วย LFS | ไม่ควร Track ด้วย LFS |
|---|---|
| ไฟล์ binary ที่มีขนาดใหญ่ (โดยทั่วไปเกิน 1–5 MB ต่อไฟล์) | ไฟล์ source code (`.js`, `.py`, `.go`) แม้จะมีจำนวนมาก |
| ไฟล์ที่เปลี่ยนแปลงทั้งไฟล์บ่อย ๆ (video, binary asset) | ไฟล์ text/config ขนาดเล็ก (`.json`, `.yaml`, `.md`) |
| ไฟล์ที่ diff ไม่มีความหมายอยู่แล้ว (ดู binary diff ไม่รู้เรื่อง) | ไฟล์ text ที่ diff มีประโยชน์ในการรีวิวโค้ด |
| Asset ที่ทีม design/media/data science ใช้ร่วมกัน | ไฟล์ที่มีขนาดเล็กมากแม้เป็น binary (เช่น icon 5 KB) เพราะ overhead ของ LFS อาจไม่คุ้ม |

### เกร็ดสำคัญ: ต้อง Track ก่อน Add เสมอ

ลำดับที่ถูกต้องคือ **track ก่อน แล้วค่อย add ไฟล์** เพราะถ้าคุณ `git add` ไฟล์ไปแล้วก่อนตั้งค่า track มันจะถูกเก็บเป็น blob ปกติของ Git ไปแล้ว การมา track ทีหลังจะไม่มีผลย้อนหลังกับไฟล์ที่ถูก add/commit ไปแล้ว (ต้องใช้ `git lfs migrate` แก้ไขในภายหลังตามที่จะพูดถึงใน Step 619)

ลำดับที่แนะนำ:

```bash
git lfs install                 # ครั้งเดียวต่อเครื่อง
git lfs track "*.psd"           # ก่อนเพิ่มไฟล์ .psd เข้า repo
git add .gitattributes          # commit pattern เอาไว้ก่อน
git add design/cover.psd        # ตอนนี้ไฟล์จะถูกจัดการโดย LFS แล้ว
git commit -m "Add cover design (tracked via Git LFS)"
```

---

## Step 615: อ่านและเข้าใจ `.gitattributes` ที่ LFS สร้างให้อัตโนมัติ

### เนื้อหาที่ถูกเขียนลงไปจริง ๆ

หลังจากรัน `git lfs track "*.psd"` ลองเปิดไฟล์ `.gitattributes` ดู:

```bash
cat .gitattributes
```

```
*.psd filter=lfs diff=lfs merge=lfs -text
```

ทุกครั้งที่ track pattern ใหม่เพิ่ม จะมีบรรทัดใหม่ต่อท้ายในรูปแบบเดียวกัน:

```
*.psd filter=lfs diff=lfs merge=lfs -text
*.ai filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text
assets/hero-video-4k.mp4 filter=lfs diff=lfs merge=lfs -text
```

### แกะความหมายทีละ attribute

| Attribute | ความหมาย |
|---|---|
| `filter=lfs` | บอก Git ว่าไฟล์ที่ตรง pattern นี้ ต้องผ่าน filter driver ชื่อ `lfs` ทุกครั้งที่ add (clean) และ checkout (smudge) — นี่คือ attribute ที่ทำงานร่วมกับค่า `filter.lfs.clean` / `filter.lfs.smudge` ที่ `git lfs install` ตั้งไว้ใน global config |
| `diff=lfs` | บอก Git ให้ใช้ diff driver ชื่อ `lfs` เวลาต้องแสดง diff ของไฟล์นี้ — ทำให้ `git diff` แสดงแค่ข้อความว่า pointer เปลี่ยนไป (เช่น oid เปลี่ยน) แทนที่จะพยายามเทียบ binary เนื้อหาแบบดิบ ๆ ซึ่งไม่มีประโยชน์อยู่แล้ว |
| `merge=lfs` | บอก Git ให้ใช้ merge driver ชื่อ `lfs` เวลาต้อง merge ไฟล์นี้ — เพราะไฟล์ binary ส่วนใหญ่ merge กันแบบ 3-way merge ตามปกติไม่ได้ (ไม่มีทางรวมเนื้อหาสองเวอร์ชันของไฟล์ .psd เข้าด้วยกันแบบอัตโนมัติ) LFS merge driver จะจัดการให้เกิด conflict อย่างถูกต้องแทนที่จะพยายาม merge แบบผิด ๆ |
| `-text` | บอก Git ว่า **ไฟล์นี้ไม่ใช่ text file** ปิดการทำงานของกลไกจัดการ line ending อัตโนมัติ (เช่น การแปลง CRLF/LF) ที่ Git ทำกับไฟล์ text ตามปกติ เพราะการไปแตะต้อง byte ของไฟล์ binary แม้แต่นิดเดียวจะทำให้ไฟล์เสียหายทันที |

### สิ่งสำคัญ: `.gitattributes` ต้องถูก Commit ด้วย!

ไฟล์ `.gitattributes` **เป็นไฟล์ text ธรรมดา ไม่ใช่ pointer file** และ **ต้องถูก commit เข้า repo เหมือนไฟล์ปกติทุกไฟล์** เพราะนี่คือกลไกที่ทำให้ **ทุกคนในทีมที่ clone repo นี้ไปได้กฎการ track เดียวกัน** โดยอัตโนมัติ — ไม่ต้องให้แต่ละคนมานั่ง `git lfs track` เองใหม่ทุกเครื่อง

```bash
git add .gitattributes
git commit -m "Configure Git LFS tracking for design assets"
```

ถ้าลืม commit ไฟล์นี้ คนอื่นที่ clone repo ไปจะไม่รู้ว่าไฟล์ประเภทไหนควรถูกจัดการโดย LFS และไฟล์ binary ใหม่ที่พวกเขา add จะถูกเก็บแบบ blob ปกติแทน ทำให้เกิดความไม่สอดคล้องกันในทีม

### ตรวจสอบว่าไฟล์ไหนถูก LFS จัดการอยู่จริง ๆ

```bash
git check-attr filter -- design/cover.psd
```

```
design/cover.psd: filter: lfs
```

ถ้าไฟล์ไหนไม่ได้อยู่ภายใต้กฎ LFS เลย ผลลัพธ์จะเป็น:

```
readme.md: filter: unspecified
```

### `.gitattributes` สามารถอยู่ได้หลายระดับโฟลเดอร์

เช่นเดียวกับ `.gitignore` คุณสามารถมี `.gitattributes` แยกในแต่ละโฟลเดอร์ย่อยได้ กฎที่อยู่ใกล้ไฟล์ที่สุด (ในโฟลเดอร์ลึกสุด) จะมีความสำคัญเหนือกฎที่อยู่ใน root — มีประโยชน์เมื่อบางโฟลเดอร์ย่อยมีกฎการ track ที่ต่างจากส่วนอื่นของโปรเจกต์

---

## Step 616: Commit/Push ไฟล์ที่ track ด้วย LFS — Workflow เหมือนเดิมทุกอย่าง

### ข่าวดี: คุณไม่ต้องเรียนคำสั่งใหม่เลย

จุดที่ยอดเยี่ยมที่สุดของการออกแบบ Git LFS คือ **workflow ประจำวันของคุณแทบไม่เปลี่ยนแปลงเลยแม้แต่น้อย** คุณยังใช้คำสั่งเดิมที่คุ้นเคยทุกอย่าง:

```bash
git add design/cover.psd
git status
git commit -m "Add new cover design"
git push origin main
```

ไม่มีคำสั่งพิเศษที่ต้องจำเพิ่มสำหรับการ commit หรือ push ไฟล์ที่ถูก track ด้วย LFS — **ความแตกต่างทั้งหมดเกิดขึ้นเบื้องหลังผ่านกลไก filter และ hook**

### สิ่งที่เกิดขึ้นเบื้องหลังตอน `git add`

1. Git เห็นว่าไฟล์ `design/cover.psd` ตรงกับ pattern `filter=lfs` ใน `.gitattributes`
2. เรียก **clean filter** (`git-lfs clean`) ให้ประมวลผลไฟล์นี้ก่อนเก็บลง staging area
3. clean filter คำนวณ hash, เก็บสำเนาไฟล์จริงไว้ใน `.git/lfs/objects/` และคืนค่า **pointer file** กลับมาให้ Git เก็บแทน

ลอง `git status` ดูจะเห็นว่าหน้าตาปกติเหมือนเดิม ไม่มีอะไรพิเศษให้เห็น:

```bash
git status
```

```
On branch main
Changes to be committed:
  new file:   design/cover.psd
```

### สิ่งที่เกิดขึ้นเบื้องหลังตอน `git commit`

Commit ทำงานตามปกติทุกประการ สิ่งที่ถูกเก็บลงใน Git object ของ commit นี้คือ **pointer file ขนาดเล็ก** ไม่ใช่ไฟล์ `.psd` จริง 300 MB

### สิ่งที่เกิดขึ้นเบื้องหลังตอน `git push` — Pre-push Hook

นี่คือจุดที่สำคัญที่สุด เมื่อคุณรัน `git lfs install` ครั้งแรก มันไม่ได้ตั้งค่าแค่ global filter อย่างเดียว แต่ยังติดตั้ง **pre-push hook** ให้กับทุก repo ที่คุณ `git init` หรือ `git clone` ใหม่หลังจากนั้นด้วย (สามารถตรวจสอบได้ที่ `.git/hooks/pre-push`)

เมื่อคุณ `git push`:

1. **pre-push hook ของ LFS ทำงานก่อน** การส่งข้อมูลไปยัง remote จริง
2. hook จะสแกนหา commit ที่กำลังจะ push ว่ามี pointer file อ้างอิงถึง LFS object ตัวไหนบ้าง
3. อัปโหลด **เนื้อหาไฟล์จริง** ของ object เหล่านั้นไปยัง **LFS server** ก่อน (ผ่าน HTTPS API ของ LFS)
4. เมื่ออัปโหลดไฟล์จริงสำเร็จครบทุกไฟล์ ถึงจะปล่อยให้ `git push` ดำเนินการ push ตัว commit (ที่มีแค่ pointer file) ไปยัง Git server ตามปกติ

```
git push origin main
```

```
Uploading LFS objects: 100% (2/2), 620 MB | 4.2 MB/s, done.
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
...
To github.com:your-team/design-assets.git
   a1b2c3d..e4f5g6h  main -> main
```

สังเกตบรรทัด `Uploading LFS objects` ที่ปรากฏ**ก่อน**ขั้นตอน push ปกติเสมอ — นี่คือสัญญาณว่า Git LFS กำลังทำงานอยู่เบื้องหลัง

### คำสั่งเสริมที่มีประโยชน์ระหว่างทำงาน

| คำสั่ง | ใช้ทำอะไร |
|---|---|
| `git lfs status` | คล้าย `git status` แต่โฟกัสที่ไฟล์ LFS โดยเฉพาะ บอกว่าไฟล์ไหนพร้อม push ไฟล์ไหนยังไม่ได้ upload |
| `git lfs ls-files` | แสดงรายการไฟล์ทั้งหมดที่ถูกจัดการโดย LFS ใน commit/branch ปัจจุบัน พร้อม oid แบบย่อ |
| `git lfs ls-files -l` | เหมือนด้านบนแต่แสดง oid แบบเต็ม |

ตัวอย่างผลลัพธ์ `git lfs ls-files`:

```
4d7a214614 * design/cover.psd
9f2c8e01aa * assets/hero-video-4k.mp4
```

เครื่องหมาย `*` หมายถึงไฟล์นี้มีเนื้อหาอยู่ครบใน local cache แล้ว ถ้าเป็น `-` แปลว่ามีแค่ pointer แต่ยังไม่ได้ดาวน์โหลดเนื้อหาจริงมา

---

## Step 617: Clone Repo ที่มี LFS — ข้อควรระวังที่พลาดกันบ่อย

### กรณีที่ 1: เครื่องปลายทางมี `git-lfs` ติดตั้งอยู่แล้ว (กรณีปกติ)

ถ้าเครื่องที่จะ clone มี git-lfs ติดตั้งและรัน `git lfs install` ไว้แล้ว ทุกอย่างจะทำงานอัตโนมัติและโปร่งใส:

```bash
git clone https://github.com/your-team/design-assets.git
```

ระหว่าง clone จะเห็นข้อความ:

```
Cloning into 'design-assets'...
remote: Enumerating objects: 120, done.
...
Filtering content: 100% (8/8), 2.1 GB | 5.5 MB/s, done.
```

บรรทัด **"Filtering content"** คือขั้นตอนที่ Git LFS กำลังดาวน์โหลดเนื้อหาไฟล์จริงของทุก pointer file ที่อยู่ใน **branch ที่ checkout ออกมาปัจจุบัน** (สำคัญ: ไม่ใช่ทุกเวอร์ชันในทุก branch/history ทั้งหมด แต่เป็นเนื้อหาที่ต้องใช้จริงสำหรับ working directory ปัจจุบันเท่านั้น — เวอร์ชันเก่าที่อยู่ใน history จะถูกดาวน์โหลดก็ต่อเมื่อคุณ checkout ไปยัง commit นั้นจริง ๆ)

หลัง clone เสร็จ ไฟล์ `design/cover.psd` ใน working directory จะเป็นไฟล์ `.psd` จริงตามปกติ เปิดใช้งานได้เลย

### กรณีที่ 2: เครื่องปลายทาง**ไม่มี** `git-lfs` ติดตั้ง (ปัญหาที่พบบ่อยที่สุด)

นี่คือกับดักที่มือใหม่ (และแม้แต่คนมีประสบการณ์) เจอบ่อยมาก ถ้าเครื่องที่ clone **ไม่มี git-lfs ติดตั้งอยู่เลย** การ clone จะสำเร็จโดยไม่มี error ใด ๆ ทั้งสิ้น แต่สิ่งที่ได้จะไม่ใช่ไฟล์จริง!

```bash
cat design/cover.psd
```

```
version https://git-lfs.github.com/spec/v1
oid sha256:4d7a214614ab2935c943f9e0ff69d22eadbb8f32b1258daaa5e2ca24d17e239
size 314572800
```

ไฟล์ที่ได้คือ **pointer file แบบ raw text ที่ยังไม่ถูกแปลง** เพราะไม่มี smudge filter ของ LFS ทำงานให้ (Git มองว่า attribute `filter=lfs` ไม่รู้จัก จึงข้ามการ process ไปเฉย ๆ) โปรแกรมกราฟิกที่พยายามเปิดไฟล์นี้จะ error ทันทีเพราะมันไม่ใช่ไฟล์ `.psd` จริง

**วิธีแก้:** ติดตั้ง `git-lfs` แล้วรันคำสั่งดึงไฟล์จริงย้อนหลังโดยไม่ต้อง clone ใหม่:

```bash
git lfs install
git lfs pull
```

### ความแตกต่างระหว่าง `git lfs fetch` กับ `git lfs pull`

| คำสั่ง | พฤติกรรม |
|---|---|
| `git lfs fetch` | ดาวน์โหลดเนื้อหาไฟล์ LFS มาเก็บใน local cache (`.git/lfs/objects/`) เท่านั้น **ไม่แตะ working directory** คล้ายกับ `git fetch` ปกติที่ไม่กระทบไฟล์ที่กำลังแก้ไขอยู่ |
| `git lfs pull` | ทำงานเหมือน `git lfs fetch` บวกกับ **checkout ไฟล์จริงเข้า working directory ทันที** แทนที่ pointer file — เทียบเท่ากับ `git fetch` + `git checkout` รวมกัน |

### เทคนิค: Clone แบบไม่ดาวน์โหลด LFS Content ทันที

บางสถานการณ์ เช่น CI pipeline ที่ไม่จำเป็นต้องใช้ไฟล์ asset จริง หรือ ต้องการ clone ให้เร็วที่สุดก่อนแล้วค่อยดึงไฟล์ทีหลัง สามารถสั่งข้าม smudge ตอน clone ได้:

```bash
GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/your-team/design-assets.git
```

จะได้ repo ที่ทุกไฟล์ LFS ยังเป็น pointer file อยู่ (clone เร็วมากเพราะไม่ต้องดาวน์โหลดเนื้อหาไฟล์จริงเลย) แล้วค่อยดึงเฉพาะไฟล์ที่ต้องการทีหลังด้วย:

```bash
cd design-assets
git lfs pull                              # ดึงทั้งหมด
git lfs pull --include="design/*.psd"     # ดึงเฉพาะบาง pattern
```

### เช็คลิสต์สำหรับทีมที่ใช้ Git LFS

ทุกคนในทีมต้อง:

1. ติดตั้ง `git-lfs` ก่อน clone repo ที่ใช้ LFS
2. รัน `git lfs install` อย่างน้อยหนึ่งครั้งบนเครื่องนั้น
3. เข้าใจว่าถ้าเปิดไฟล์แล้วเจอข้อความ pointer แปลก ๆ แทนไฟล์จริง แปลว่า LFS ยังไม่ถูกติดตั้ง/ทำงานอย่างถูกต้อง

---

## Step 618: LFS Storage และ Bandwidth Quota บน GitHub/GitLab

### ทำไมถึงมี Quota

เนื่องจาก LFS storage เป็น**พื้นที่จัดเก็บข้อมูลแยกต่างหากจาก Git repository หลัก** ผู้ให้บริการอย่าง GitHub และ GitLab จึงคิดค่าใช้จ่ายและจำกัดโควตาของมันแยกต่างหากด้วยเช่นกัน ไม่ได้นับรวมกับพื้นที่เก็บ repository ปกติเสมอไป

Quota ของ Git LFS แบ่งออกเป็น 2 มิติหลักที่ต้องเข้าใจแยกกัน:

| มิติ | ความหมาย |
|---|---|
| **Storage (พื้นที่จัดเก็บ)** | ปริมาณข้อมูลทั้งหมดที่ถูกเก็บสะสมไว้ใน LFS server ของ repo (หรือ organization) นั้น รวมทุกเวอร์ชันของทุกไฟล์ที่เคย push เข้าไป |
| **Bandwidth (แบนด์วิดท์)** | ปริมาณข้อมูลที่ถูก**ดาวน์โหลด**ออกจาก LFS server ในแต่ละเดือน (เช่น ทุกครั้งที่มีคน clone หรือ `git lfs pull`) นับใหม่ทุกรอบเดือน |

### GitHub

GitHub ให้โควตาฟรีเริ่มต้นสำหรับบัญชีทุกประเภท (ทั้ง Free และ paid plan) ในรูปแบบ **"Git LFS Data Pack"**:

- โควตาฟรีเริ่มต้น: พื้นที่เก็บข้อมูล (storage) และแบนด์วิดท์ (bandwidth) ต่อเดือนในปริมาณจำกัดต่อบัญชี (คิดแยกจากโควตาพื้นที่ repository ปกติ)
- เมื่อใช้เกินโควตาฟรี สามารถซื้อ **Data Pack เพิ่มเติม** ได้เป็นรายเดือน ซึ่งจะเพิ่มทั้ง storage และ bandwidth ไปพร้อมกันเป็นก้อน
- Bandwidth จะถูกนับ**ทุกครั้งที่มีการดาวน์โหลดไฟล์ LFS** ไม่ว่าจะเป็นจากการ clone repo, `git lfs pull`, หรือแม้แต่ CI/CD pipeline ที่ checkout repo ซ้ำ ๆ หลายรอบต่อวัน — ทีมที่มี CI รันบ่อยมากอาจใช้ bandwidth หมดเร็วกว่าที่คาดโดยไม่รู้ตัว

> **หมายเหตุสำคัญ:** ตัวเลขโควตาที่แน่นอนและราคาของ Data Pack เปลี่ยนแปลงได้ตามนโยบายของ GitHub ในแต่ละช่วงเวลา ควรตรวจสอบหน้า pricing และ documentation อย่างเป็นทางการของ GitHub เสมอก่อนวางแผนใช้งานจริงในโปรเจกต์ขนาดใหญ่ อย่ายึดตัวเลขจากที่ใดที่หนึ่งเป็นค่าตายตัวถาวร

### GitLab

GitLab จัดการพื้นที่เก็บข้อมูลในเชิงภาพรวมมากกว่า โดยทั่วไป:

- พื้นที่ของ Git LFS objects จะถูก**นับรวมอยู่ในโควตาพื้นที่จัดเก็บข้อมูลรวม (total storage) ของ namespace/project** ไม่ได้แยกโควตาเป็นก้อนต่างหากชัดเจนแบบ GitHub เสมอไป
- แผนแบบ self-managed (ติดตั้งเซิร์ฟเวอร์ GitLab เอง) สามารถกำหนดขนาด LFS storage ได้เองแทบไม่จำกัด ขึ้นอยู่กับพื้นที่ดิสก์ของ server ที่ติดตั้ง — เป็นข้อได้เปรียบสำคัญของ GitLab self-hosted ที่เราเคยพูดถึงใน Part เกี่ยวกับ GitLab
- แผนแบบ GitLab.com (SaaS) มีโควตาตามระดับแผนสมาชิก (Free/Premium/Ultimate) เช่นเดียวกับ GitHub และมีตัวเลือกซื้อพื้นที่เพิ่มเติมเป็นแพ็กเกจ

> เช่นเดียวกับ GitHub ตัวเลขโควตาที่แน่นอนของ GitLab เปลี่ยนแปลงตามช่วงเวลาและแผนสมาชิก ให้ตรวจสอบหน้าราคาอย่างเป็นทางการก่อนใช้งานจริงเสมอ

### วิธีตรวจสอบการใช้งาน LFS ของตัวเอง

**บน GitHub:** เข้าไปที่ **Settings → Billing and plans** ของบัญชีหรือ organization จะเห็นหัวข้อ **Git LFS Data** แสดงปริมาณ storage และ bandwidth ที่ใช้ไปในรอบเดือนปัจจุบัน

**บน GitLab:** เข้าไปที่ **Project → Settings → Usage Quotas** จะเห็นรายละเอียดพื้นที่จัดเก็บแยกตามประเภท ซึ่งรวมถึงส่วนของ LFS objects ด้วย

### แนวทางบริหารจัดการ Quota อย่างมีประสิทธิภาพ

1. **Track เฉพาะไฟล์ที่จำเป็นจริง ๆ** — อย่า track ไฟล์ที่ไม่ค่อยเปลี่ยนแปลงหรือมีขนาดเล็กเกินความจำเป็น
2. **ตั้งค่า CI ให้ไม่ดึงไฟล์ LFS โดยไม่จำเป็น** — ใช้ `GIT_LFS_SKIP_SMUDGE=1` ใน pipeline ที่ไม่ต้องใช้ asset จริง (เช่น job ที่แค่รัน unit test บน source code)
3. **ลบไฟล์เก่าที่ไม่ใช้แล้วออกจาก LFS cache ในเครื่อง** ด้วย `git lfs prune` เพื่อประหยัดพื้นที่ดิสก์ในเครื่อง (คำสั่งนี้ลบเฉพาะ local cache ไม่กระทบข้อมูลบน server)
4. **พิจารณาใช้บริการ LFS server แยกต่างหาก** (self-hosted LFS server หรือ third-party storage เช่น S3-compatible LFS server) สำหรับโปรเจกต์ที่มีปริมาณข้อมูลมหาศาลเกินกว่าที่แผนของ GitHub/GitLab จะรองรับได้คุ้มค่า — Git LFS protocol เป็น open specification ที่รองรับ server หลากหลายเจ้า ไม่ได้ผูกติดกับ GitHub/GitLab เท่านั้น

---

## Step 619: Migrate ไฟล์เก่าที่ track แบบปกติเข้า LFS ด้วย `git lfs migrate import`

### ปัญหาที่พบบ่อยในโลกจริง

สถานการณ์ที่เกิดขึ้นบ่อยมากคือ **ทีมเริ่มโปรเจกต์โดยไม่รู้จัก Git LFS มาก่อน** จึง commit ไฟล์ binary ขนาดใหญ่ (เช่น `.psd`, `.mp4`) เข้าไปใน Git repository แบบปกติมาเป็นเวลานาน จนกระทั่งวันหนึ่งพบว่า repo บวมมาก clone ช้าจนทนไม่ไหว แล้วถึงมาตัดสินใจ**ย้ายมาใช้ Git LFS** ในภายหลัง

ปัญหาคือ **แค่รัน `git lfs track` แล้ว commit ใหม่ ไม่ช่วยอะไรกับไฟล์เก่าที่ฝังอยู่ในประวัติ (history) แล้ว** เพราะไฟล์เวอร์ชันเก่าเหล่านั้นถูกเก็บเป็น blob ปกติไปแล้วอย่างถาวรใน object ของ commit เก่า ๆ ที่ผ่านมา — การ track ใหม่มีผลแค่กับไฟล์ที่ add ใหม่นับจากนี้เท่านั้น

นี่คือจุดที่ **`git lfs migrate`** เข้ามาช่วยแก้ปัญหา

### หลักการทำงานของ `git lfs migrate import`

คำสั่งนี้ทำงานโดย **เขียนประวัติ Git ใหม่ทั้งหมด (rewrite history)** คล้ายกับหลักการของ `git filter-branch` หรือ `git rebase` ขนาดใหญ่ โดยจะ:

1. ไล่สแกนทุก commit ในประวัติที่ระบุ (ค่าเริ่มต้นคือ branch ปัจจุบัน)
2. สำหรับไฟล์ที่ตรงกับ pattern ที่ระบุ ให้ **แทนที่เนื้อหาไฟล์ blob เดิม ด้วย pointer file** พร้อมย้ายเนื้อหาจริงเข้า LFS storage
3. สร้าง commit ใหม่ทั้งหมดที่มี **SHA ต่างไปจากเดิม** (เพราะเนื้อหาเปลี่ยน จึงต้องคำนวณ hash ใหม่ ซึ่งจะกระทบ SHA ของทุก commit ที่ตามมาหลังจากนั้นด้วย เนื่องจาก Git ใช้โครงสร้างแบบ chain ที่แต่ละ commit อ้างอิง hash ของ commit ก่อนหน้า)

### ตัวอย่างการใช้งานพื้นฐาน

Migrate ไฟล์ `.psd` ทั้งหมดใน branch ปัจจุบันเข้า LFS:

```bash
git lfs migrate import --include="*.psd"
```

Migrate หลายนามสกุลไฟล์พร้อมกัน:

```bash
git lfs migrate import --include="*.psd,*.ai,*.mp4"
```

### Flag สำคัญที่ต้องรู้จัก

| Flag | ความหมาย |
|---|---|
| `--include="<pattern>"` | ระบุ pattern ของไฟล์ที่จะ migrate เข้า LFS |
| `--exclude="<pattern>"` | ยกเว้นไฟล์บาง pattern ออกจากการ migrate แม้จะตรงกับ `--include` |
| `--everything` | migrate ทุก branch, tag, และ ref ทั้งหมดในโปรเจกต์ ไม่ใช่แค่ branch ปัจจุบัน (สำคัญมากถ้ามีหลาย branch ที่ยังใช้งานอยู่) |
| `--above=<size>` | migrate เฉพาะไฟล์ที่มีขนาด**ใหญ่กว่า**ค่าที่กำหนด เช่น `--above=10Mb` — มีประโยชน์มากเมื่อไม่รู้แน่ชัดว่าไฟล์ประเภทไหนใหญ่บ้าง สามารถให้ Git หาไฟล์ใหญ่ทั้งหมดในประวัติมา migrate ให้อัตโนมัติโดยไม่ต้องระบุนามสกุลเอง |
| `--no-rewrite` | เพิ่ม commit ใหม่ที่ migrate ไฟล์ แทนที่จะเขียนประวัติเดิมทับ (ปลอดภัยกว่าแต่ไม่ช่วยลดขนาด repo เดิมที่บวมอยู่แล้ว) |
| `--dry-run` (ในบาง flow ใช้ร่วมกับการตรวจสอบก่อน) | ใช้ตรวจสอบว่าอะไรจะถูกกระทบก่อนรันจริง (แนะนำให้ตรวจสอบผลกระทบก่อนเสมอด้วยการสำรองข้อมูลและทดสอบใน branch ทดลอง) |

ตัวอย่างการหาไฟล์ใหญ่ทั้งหมดในทุก branch โดยไม่ต้องรู้ pattern ล่วงหน้า:

```bash
git lfs migrate import --everything --above=10Mb
```

### ⚠️ คำเตือนสำคัญมาก — อ่านก่อนใช้งานจริงเสมอ

เพราะ `git lfs migrate import` **เขียนประวัติ Git ใหม่ทั้งหมด (rewrite history)** จึงมีผลกระทบร้ายแรงที่ต้องระวังอย่างยิ่ง:

1. **SHA ของทุก commit ที่ถูกกระทบจะเปลี่ยนไปทั้งหมด** — commit ที่เคย reference กันไว้ที่อื่น (เช่น ลิงก์ไปยัง commit เฉพาะใน issue tracker, tag ที่คนอื่น pull ไปแล้ว) จะใช้ไม่ได้อีกต่อไป
2. **ต้อง force-push** เพื่ออัปเดต branch บน remote server เพราะประวัติเดิมกับใหม่ไม่ตรงกันแล้ว (`git push --force-with-lease`)
3. **ทุกคนในทีมที่มี clone ของ repo เดิมอยู่ต้อง clone ใหม่** หรือทำตามขั้นตอน reset ที่ซับซ้อนเพื่อ sync กับประวัติใหม่ — การพยายาม pull แบบปกติจะเกิด conflict ทันทีเพราะประวัติไม่ตรงกันเลย
4. **ควร backup repository ทั้งหมดก่อนเสมอ** (เช่น clone แบบ bare ไว้อีกที่หนึ่ง) เผื่อเกิดข้อผิดพลาดระหว่าง migrate
5. **ควรแจ้งทีมงานทุกคนล่วงหน้า** และเลือกช่วงเวลาที่ไม่มีใครกำลัง push งานเข้า repo (เช่น นอกเวลาทำงาน) เพื่อลด conflict ที่จะเกิดขึ้น
6. **Pull Request หรือ Merge Request ที่เปิดค้างอยู่จะได้รับผลกระทบ** เพราะ branch ต้นทางที่มันอ้างอิงถึงมี SHA เปลี่ยนไป ควร merge หรือปิดงานค้างให้เรียบร้อยก่อน migrate

### ขั้นตอนแนะนำสำหรับการ Migrate ในทีมจริง

```bash
# 1. สำรองข้อมูลก่อนเสมอ
git clone --mirror https://github.com/your-team/old-repo.git backup-repo.git

# 2. ติดตั้งและเปิดใช้ Git LFS
git lfs install

# 3. Migrate ไฟล์ที่ต้องการ (รวมทุก branch/tag)
git lfs migrate import --everything --include="*.psd,*.mp4,*.zip"

# 4. ตรวจสอบผลลัพธ์ว่าไฟล์กลายเป็น pointer จริง และขนาด repo เล็กลง
git lfs ls-files
du -sh .git

# 5. Force-push ประวัติใหม่ทั้งหมดขึ้น remote
git push --force --all
git push --force --tags

# 6. แจ้งทีมให้ clone repo ใหม่ทั้งหมด
```

### ทางเลือกที่ปลอดภัยกว่าถ้าไม่ต้องการรื้อประวัติเก่า

ถ้าทีมไม่อยากเสี่ยงกับการ rewrite history ของ repo ที่มีอยู่แล้ว (เช่น repo ที่มีคนภายนอกจำนวนมาก clone ไปแล้ว) อีกทางเลือกหนึ่งคือ:

- ปล่อยประวัติเก่าไว้ตามเดิม (repo จะยังคงมีขนาดใหญ่จากไฟล์เก่าอยู่)
- ใช้ `git lfs track` เพื่อจัดการแค่ **ไฟล์ใหม่ที่จะเพิ่มเข้ามาต่อจากนี้** ให้ไม่บวมเพิ่มขึ้นอีก
- หรือสร้าง **repository ใหม่** ที่เริ่มต้นด้วยไฟล์เวอร์ชันปัจจุบันที่ผ่านการ track ด้วย LFS ตั้งแต่ commit แรก โดยไม่แบก history เก่าที่บวมมาด้วย (เหมาะกับกรณีที่ประวัติเก่าไม่มีความสำคัญมากนัก)

การเลือกวิธีไหนขึ้นอยู่กับว่าทีมให้ความสำคัญกับ **ความสมบูรณ์ของประวัติเดิม** มากกว่า หรือ **ขนาด repo ที่เล็กลงทันที** มากกว่า

---

## Step 620: แบบฝึกหัด — ตั้งค่า Git LFS ให้โปรเจกต์ที่มีไฟล์รูปภาพขนาดใหญ่แบบครบวงจร

ถึงเวลาลงมือทำจริงตั้งแต่ต้นจนจบ แบบฝึกหัดนี้จะพาคุณผ่านทุกขั้นตอนที่เรียนมาใน Part นี้

### ขั้นตอนที่ 1: เตรียมโปรเจกต์ทดลอง

```bash
mkdir ~/git-course/part-62-lfs-practice
cd ~/git-course/part-62-lfs-practice
git init
```

### ขั้นตอนที่ 2: ติดตั้งและเปิดใช้งาน Git LFS (ถ้ายังไม่ได้ทำ)

```bash
git lfs version
git lfs install
```

ควรเห็นข้อความ:

```
Updated Git hooks.
Git LFS initialized.
```

### ขั้นตอนที่ 3: จำลองไฟล์รูปภาพขนาดใหญ่

เนื่องจากเราไม่มีไฟล์รูปภาพจริงขนาดใหญ่ในมือ ให้ใช้คำสั่งสร้างไฟล์จำลองขนาด 50 MB ขึ้นมาแทน (สมมติว่านี่คือไฟล์ภาพถ่ายความละเอียดสูงจากกล้อง):

```bash
mkdir -p assets/photos
# macOS/Linux
dd if=/dev/urandom of=assets/photos/landscape-4k.jpg bs=1M count=50
```

(สำหรับ Windows PowerShell สามารถใช้ `fsutil file createnew assets\photos\landscape-4k.jpg 52428800` แทนได้)

ตรวจสอบขนาดไฟล์:

```bash
ls -lh assets/photos/landscape-4k.jpg
```

```
-rw-r--r--  1 user  staff    50M  Sep 26 10:00 assets/photos/landscape-4k.jpg
```

### ขั้นตอนที่ 4: Track ไฟล์รูปภาพด้วย Git LFS

```bash
git lfs track "assets/photos/*.jpg"
git lfs track "*.png"
```

ตรวจสอบว่า pattern ถูกบันทึกแล้ว:

```bash
cat .gitattributes
```

```
assets/photos/*.jpg filter=lfs diff=lfs merge=lfs -text
*.png filter=lfs diff=lfs merge=lfs -text
```

### ขั้นตอนที่ 5: Commit `.gitattributes` ก่อนเสมอ

```bash
git add .gitattributes
git commit -m "Configure Git LFS tracking for photo assets"
```

### ขั้นตอนที่ 6: Add และ Commit ไฟล์รูปภาพ

```bash
git add assets/photos/landscape-4k.jpg
git status
```

```
On branch main
Changes to be committed:
  new file:   assets/photos/landscape-4k.jpg
```

```bash
git commit -m "Add 4K landscape photo (tracked via Git LFS)"
```

### ขั้นตอนที่ 7: ตรวจสอบว่า LFS ทำงานจริง

ตรวจสอบว่าไฟล์ถูกจัดการโดย LFS filter:

```bash
git check-attr filter -- assets/photos/landscape-4k.jpg
```

```
assets/photos/landscape-4k.jpg: filter: lfs
```

ดูรายการไฟล์ที่ LFS จัดการอยู่:

```bash
git lfs ls-files
```

```
a1b2c3d4e5 * assets/photos/landscape-4k.jpg
```

เปรียบเทียบขนาด object ที่แท้จริงใน Git กับขนาดไฟล์จริง เพื่อพิสูจน์ว่า Git เก็บแค่ pointer:

```bash
git cat-file -p HEAD:assets/photos/landscape-4k.jpg
```

ควรเห็นเนื้อหาสั้น ๆ แบบนี้ (นี่คือสิ่งที่ Git เก็บจริง ไม่ใช่ไฟล์ 50 MB):

```
version https://git-lfs.github.com/spec/v1
oid sha256:2f8a9c1b7e3d5f0a...(ตัดให้สั้นลง)...
size 52428800
```

### ขั้นตอนที่ 8: เชื่อมต่อ Remote และ Push

สร้าง repository เปล่าบน GitHub หรือ GitLab (ผ่านหน้าเว็บตามที่เคยฝึกมาใน Part ก่อนหน้า) แล้วเชื่อมต่อ:

```bash
git remote add origin https://github.com/<your-username>/lfs-practice.git
git push -u origin main
```

สังเกตข้อความระหว่าง push อย่างละเอียด:

```
Uploading LFS objects: 100% (1/1), 50 MB | 3.1 MB/s, done.
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Writing objects: 100% (4/4), 512 bytes | 512.00 KiB/s, done.
To github.com:<your-username>/lfs-practice.git
 * [new branch]      main -> main
```

บันทึกไว้ว่า **"Writing objects" มีขนาดแค่ 512 bytes** (แค่ pointer file กับ metadata) ในขณะที่ **"Uploading LFS objects" มีขนาด 50 MB** (เนื้อหาไฟล์จริงที่อัปโหลดแยกไปยัง LFS storage) — นี่คือหลักฐานชัดเจนที่สุดว่ากลไก 2 ชั้นของ LFS ทำงานจริงตามที่เรียนมาใน Step 612

### ขั้นตอนที่ 9: ทดสอบ Clone ใหม่เพื่อยืนยันว่าทำงานถูกต้อง

จำลองสถานการณ์เหมือนเพื่อนร่วมทีมอีกคนมา clone repo นี้ครั้งแรก:

```bash
cd ~/git-course
git clone https://github.com/<your-username>/lfs-practice.git lfs-practice-clone
cd lfs-practice-clone
ls -lh assets/photos/landscape-4k.jpg
```

ควรเห็นว่าไฟล์มีขนาดเต็ม 50 MB ตามปกติ (ไม่ใช่ pointer file) เพราะเครื่องนี้มี git-lfs ติดตั้งและ smudge filter ทำงานให้อัตโนมัติระหว่าง clone

### ขั้นตอนที่ 10 (ทางเลือกเสริม): จำลองกรณีไม่มี Git LFS ติดตั้ง

เพื่อความเข้าใจที่ลึกซึ้งยิ่งขึ้น ลองจำลองว่าถ้า clone ด้วยเครื่องที่ไม่มี git-lfs จะเกิดอะไรขึ้น โดยสั่งข้าม smudge ชั่วคราว:

```bash
cd ~/git-course
GIT_LFS_SKIP_SMUDGE=1 git clone https://github.com/<your-username>/lfs-practice.git lfs-practice-nosmudge
cd lfs-practice-nosmudge
cat assets/photos/landscape-4k.jpg
```

คุณจะเห็น pointer file แบบ raw text แทนที่จะเป็นไฟล์ภาพจริง ซึ่งจำลองสถานการณ์ที่อธิบายไว้ใน Step 617 ได้อย่างชัดเจน จากนั้นลองแก้ปัญหาด้วยตัวเองด้วยคำสั่ง:

```bash
git lfs pull
cat assets/photos/landscape-4k.jpg | head -c 20
```

ครั้งนี้ควรเห็นข้อมูล binary จริง (แสดงเป็นตัวอักษรแปลก ๆ ในเทอร์มินัล) แทนที่จะเป็นข้อความ pointer แบบเดิม

### เกณฑ์ตรวจสอบว่าทำแบบฝึกหัดสำเร็จ

- [ ] `git lfs version` แสดงเลขเวอร์ชันโดยไม่มี error
- [ ] `.gitattributes` มีบรรทัด `filter=lfs diff=lfs merge=lfs -text` สำหรับ pattern ที่ track
- [ ] `git cat-file -p HEAD:<path>` ของไฟล์ที่ track แสดง pointer file ไม่ใช่เนื้อหาไฟล์จริง
- [ ] `git push` แสดงบรรทัด "Uploading LFS objects" แยกจาก "Writing objects"
- [ ] Clone ใหม่ในเครื่องที่มี git-lfs ได้ไฟล์เต็มขนาดจริงอัตโนมัติ
- [ ] เข้าใจและทดลองเห็นผลต่างเมื่อ clone แบบข้าม smudge (`GIT_LFS_SKIP_SMUDGE=1`) แล้วใช้ `git lfs pull` แก้ไข

---

## สรุป Part 62

ใน Part นี้เราได้เรียนรู้ว่า:

1. Git ทำงานได้ยอดเยี่ยมกับไฟล์ text แต่มีปัญหาร้ายแรงกับไฟล์ binary ขนาดใหญ่ เพราะ delta compression ทำงานไม่ได้ผลกับไฟล์ประเภทนี้ ทำให้ repository บวมและ clone ช้าลงเรื่อย ๆ
2. Git LFS แก้ปัญหานี้ด้วยการเก็บแค่ **pointer file** ขนาดเล็กไว้ใน Git repository ส่วนเนื้อหาไฟล์จริงถูกแยกไปเก็บใน **LFS storage** ต่างหาก โดยอาศัยกลไก **clean/smudge filter** ที่เป็นแนวคิดพื้นฐานของ Git อยู่แล้ว
3. การใช้งานเริ่มจาก `git lfs install` (ครั้งเดียวต่อเครื่อง) แล้ว `git lfs track "<pattern>"` เพื่อกำหนดว่าไฟล์ประเภทไหนที่ต้องการให้ LFS จัดการ ซึ่งจะเขียนกฎลงใน `.gitattributes` ที่ต้อง commit เข้า repo เสมอ
4. Workflow การ commit/push ไฟล์ที่ track ด้วย LFS **เหมือนเดิมทุกประการ** ความแตกต่างทั้งหมดเกิดขึ้นโดยอัตโนมัติผ่าน filter และ pre-push hook เบื้องหลัง
5. เครื่องที่ clone repo ที่มี LFS **ต้องติดตั้ง git-lfs ก่อนเสมอ** ไม่เช่นนั้นจะได้แค่ pointer file แทนไฟล์จริง แก้ไขได้ด้วย `git lfs pull` ภายหลัง
6. GitHub และ GitLab มีโควตา storage/bandwidth แยกต่างหากสำหรับ LFS ที่ต้องวางแผนบริหารจัดการอย่างเหมาะสม และตัวเลขเหล่านี้เปลี่ยนแปลงได้ตามนโยบายของผู้ให้บริการ
7. ไฟล์เก่าที่ถูก commit แบบปกติไปแล้วก่อนรู้จัก LFS สามารถย้ายเข้า LFS ได้ทีหลังด้วย `git lfs migrate import` แต่ต้องระวังอย่างมากเพราะเป็นการ **rewrite history** ที่กระทบ SHA ของทุก commit และต้อง force-push พร้อมประสานงานกับทั้งทีม
8. เราได้ลงมือฝึกฝนตั้งค่า Git LFS แบบครบวงจรตั้งแต่ track ไฟล์ commit push จนถึง clone และทดสอบพฤติกรรมเมื่อไม่มี git-lfs ติดตั้ง

**ต่อไป:** [Part 63: การจัดการ Performance ของ Repository ขนาดใหญ่](./part-063-performance-repo-ขนาดใหญ่.md)
