# Part 79: Git Security — Signed Commits, GPG, SSH Signing

> **Step ในหลักสูตรนี้:** Step 781–790
> **เฟส:** 8 — DevOps, Security, Compliance ระดับองค์กร
> **เป้าหมายของ Part นี้:** เจาะลึกเต็มรูปแบบเรื่อง Commit Signing ที่เคยเกริ่นไว้ตั้งแต่ Part 18 (Step 178) และ Part 37 (Step 366) ให้เข้าใจตั้งแต่หลักการเข้ารหัสเบื้องหลัง ไปจนถึงลงมือสร้าง GPG key และ SSH signing key จริง ตั้งค่า Git ให้เซ็นชื่อ commit อัตโนมัติ เปิด Vigilant mode และ Branch Protection เพื่อบังคับใช้ signed commits ทั้งองค์กร พร้อมเข้าใจข้อจำกัดที่แท้จริงของกลไกนี้

---

## สารบัญของ Part นี้

- Step 781: ทำไมต้อง sign commit — ปัญหาการปลอมแปลงตัวตนใน Git
- Step 782: GPG คืออะไร หลักการ public/private key สำหรับการ sign
- Step 783: สร้าง GPG key และเพิ่ม public key เข้า GitHub/GitLab
- Step 784: ตั้งค่า Git ให้ sign commit ด้วย GPG อัตโนมัติ
- Step 785: SSH commit signing — ทางเลือกที่ง่ายกว่า GPG มาก
- Step 786: Verified badge บน GitHub/GitLab คืออะไร ทำงานอย่างไร
- Step 787: Vigilant mode — เปิดใช้เพื่อจับ commit ปลอมที่อ้างเป็นเรา
- Step 788: บังคับ signed commits ทั้งหมดด้วย Branch Protection Rule
- Step 789: ข้อจำกัดของ commit signing — สิ่งที่มันไม่ได้การันตี
- Step 790: แบบฝึกหัด — ตั้งค่า SSH commit signing จนได้ Verified badge จริงบน GitHub

---

## Step 781: ทำไมต้อง sign commit

### ย้อนความเข้าใจจาก Part 18

ใน **Part 18 (Step 178)** เราเกริ่นไว้แล้วว่า Git ไม่มีการตรวจสอบตัวตนของผู้เขียน commit เลยแม้แต่น้อย ค่า `user.name` และ `user.email` เป็นเพียงข้อความธรรมดาที่ใครก็ตั้งเป็นอะไรก็ได้ ลองพิสูจน์ด้วยตัวเองอีกครั้งเพื่อให้เห็นภาพชัดก่อนเริ่ม Part นี้:

```bash
mkdir /tmp/spoof-demo
cd /tmp/spoof-demo
git init

git config user.name "Linus Torvalds"
git config user.email "torvalds@linux-foundation.org"
git commit --allow-empty -m "แก้ไข kernel scheduler"

git log --format="%an <%ae> — %s"
```

ผลลัพธ์:

```
Linus Torvalds <torvalds@linux-foundation.org> — แก้ไข kernel scheduler
```

**ไม่มีใครถามรหัสผ่านหรือยืนยันตัวตนใด ๆ เลย** ใครก็ตามที่มี Git ติดตั้งอยู่สามารถตั้ง `user.name`/`user.email` เป็นชื่อใครก็ได้บนโลก แล้วสร้าง commit ที่ "ดูเหมือน" มาจากคนนั้นได้ทันที

### ทำไมเรื่องนี้ถึงเป็นปัญหาความปลอดภัยจริงจัง

ลองนึกภาพสถานการณ์ต่อไปนี้ ซึ่งล้วนเป็นเหตุการณ์ที่เคยเกิดขึ้นจริงในวงการ Open Source และองค์กรต่าง ๆ:

1. **การปลอมแปลง maintainer** — ผู้ไม่หวังดี clone repository แบบ public แล้วสร้าง commit ที่ตั้งชื่อผู้เขียนเป็น maintainer ตัวจริงของโปรเจกต์ แนบโค้ดที่มี backdoor ซ่อนอยู่ ถ้าไม่มีการ sign commit และไม่มีใครตรวจโค้ดอย่างละเอียด commit นั้นอาจถูกเข้าใจผิดว่ามาจาก maintainer จริง
2. **การใส่ร้ายผู้อื่น** — พนักงานคนหนึ่งอาจสร้าง commit ที่มีเนื้อหาไม่เหมาะสมหรือโค้ดที่ตั้งใจทำลายระบบ แล้วปลอมชื่อผู้เขียนเป็นเพื่อนร่วมทีม เพื่อให้ความผิดตกไปอยู่ที่คนอื่น
3. **Supply chain attack** — ผู้โจมตีแทรกโค้ดอันตรายเข้าไปในประวัติของ dependency ที่คนอื่นเชื่อถือ โดยปลอมชื่อ contributor ที่มีชื่อเสียงเพื่อลดโอกาสถูกตรวจจับตอนรีวิว
4. **การตรวจสอบย้อนหลัง (audit) ที่ไม่น่าเชื่อถือ** — ในองค์กรที่ต้องผ่านมาตรฐาน compliance (เช่น ISO 27001, SOC 2, PCI-DSS) การพิสูจน์ว่า "ใครเป็นคนแก้โค้ดจริง ๆ" คือข้อกำหนดพื้นฐาน ถ้า commit author ปลอมแปลงได้ง่าย หลักฐานการ audit ทั้งหมดก็ไร้ความหมาย

### Commit signing แก้ปัญหานี้อย่างไร

**Commit signing (การเซ็นชื่อ commit)** คือการใช้ **cryptographic signature** (ลายเซ็นดิจิทัลที่สร้างจากกุญแจส่วนตัว) แนบไปกับ commit แต่ละอัน เพื่อพิสูจน์ทางคณิตศาสตร์ว่า:

> **"คนที่สร้าง commit นี้ คือเจ้าของกุญแจส่วนตัวที่ใช้เซ็นชื่อจริง ๆ ไม่มีใครปลอมแปลงลายเซ็นนี้ได้โดยไม่มีกุญแจส่วนตัวตัวนั้น"**

หลักการสำคัญคือ **ลายเซ็นดิจิทัลปลอมแปลงไม่ได้ในทางปฏิบัติ** ต่างจากการพิมพ์ชื่อ `user.name` ที่ปลอมได้ในหนึ่งวินาที การจะสร้างลายเซ็นที่ตรวจสอบผ่านได้ ผู้สร้างต้องมี **private key** ตัวจริงเท่านั้น ไม่มีทางลัดทางคณิตศาสตร์ที่จะปลอมลายเซ็นได้โดยไม่มีกุญแจนั้น (ตราบใดที่อัลกอริทึมเข้ารหัสยังไม่ถูกทำลาย และความยาวของกุญแจเพียงพอ)

Git รองรับการ sign commit ด้วยกุญแจ 2 รูปแบบหลัก ซึ่งเราจะเรียนทั้งคู่ใน Part นี้:

| วิธี | ใช้กุญแจแบบไหน | ความซับซ้อนในการตั้งค่า |
|---|---|---|
| **GPG signing** | GPG key (สร้างแยกต่างหาก) | ซับซ้อนกว่า ต้องติดตั้งโปรแกรม GPG เพิ่ม |
| **SSH signing** | SSH key เดิมที่มีอยู่แล้ว | ง่ายกว่ามาก ถ้ามี SSH key จาก Part 18 อยู่แล้วใช้ต่อได้ทันที |

### สิ่งที่ commit signing ยืนยัน (และย้ำไว้ก่อนตั้งแต่ต้น)

commit signing ยืนยันแค่ **"ใครเป็นเจ้าของกุญแจที่สร้าง commit นี้"** เท่านั้น มันไม่ได้บอกว่าโค้ดข้างในปลอดภัยหรือถูกต้องแต่อย่างใด — เราจะพูดถึงข้อจำกัดนี้อย่างละเอียดใน **Step 789** เพื่อไม่ให้เข้าใจผิดว่า signed commit คือ "ตราประทับความปลอดภัยของโค้ด"

---

## Step 782: GPG คืออะไร หลักการ public/private key สำหรับการ sign

### GPG คืออะไร

**GPG (GNU Privacy Guard)** คือซอฟต์แวร์ open source ที่ใช้งานมาตรฐานการเข้ารหัส **OpenPGP** (Pretty Good Privacy) สำหรับการเข้ารหัสข้อมูลและการเซ็นชื่อดิจิทัล GPG ถูกใช้งานอย่างแพร่หลายมานานหลายสิบปี ทั้งในการเข้ารหัสอีเมล การเซ็นชื่อไฟล์ซอฟต์แวร์ที่แจกจ่าย (release) และแน่นอนว่ารวมถึงการเซ็นชื่อ Git commit ด้วย

### หลักการ Public-key Cryptography (asymmetric cryptography)

หัวใจของ GPG (และของ SSH signing ที่จะพูดถึงใน Step 785) คือแนวคิดที่เรียกว่า **การเข้ารหัสแบบไม่สมมาตร (asymmetric cryptography)** ซึ่งใช้กุญแจ 2 ดอกที่เกี่ยวข้องกันทางคณิตศาสตร์ แต่มีบทบาทตรงข้ามกัน:

1. **Private key (กุญแจส่วนตัว)** — เก็บไว้กับตัวเองเท่านั้น **ห้ามเปิดเผยให้ใครเห็นเด็ดขาด** ใช้สำหรับ "เซ็นชื่อ" (sign) ข้อมูล
2. **Public key (กุญแจสาธารณะ)** — เผยแพร่ให้ใครก็ได้ ไม่มีความลับ ใช้สำหรับ "ตรวจสอบ" (verify) ว่าลายเซ็นนั้นถูกสร้างโดยเจ้าของ private key คู่กันจริงหรือไม่

คุณสมบัติทางคณิตศาสตร์ที่สำคัญที่สุดของกุญแจคู่นี้คือ:

> **การคำนวณจาก private key ไปหา public key ทำได้ง่ายและเร็วมาก แต่การคำนวณย้อนกลับจาก public key ไปหา private key นั้นยากเกินกว่าจะทำได้จริงในทางปฏิบัติ (แม้จะใช้ซูเปอร์คอมพิวเตอร์ก็ตาม) ตราบใดที่ใช้ขนาดกุญแจที่เหมาะสม**

### กระบวนการ sign และ verify ทำงานอย่างไร (แนวคิดระดับสูง)

```
                 ┌─────────────────────┐
                 │   เนื้อหาของ commit   │
                 │ (author, message,   │
                 │  tree hash, parent) │
                 └──────────┬──────────┘
                            │
                     ใช้ Private Key เซ็น
                            │
                            ▼
                 ┌─────────────────────┐
                 │   ลายเซ็นดิจิทัล      │
                 │  (signature)         │
                 └──────────┬──────────┘
                            │
              แนบลายเซ็นไว้ใน commit object
                            │
                            ▼
        ┌───────────────────────────────────┐
        │  ฝั่งผู้ตรวจสอบ (GitHub/GitLab/     │
        │  ใครก็ตามที่มี Public Key)          │
        │                                    │
        │  ใช้ Public Key ตรวจสอบลายเซ็น      │
        │  ว่าตรงกับเนื้อหา commit หรือไม่     │
        └───────────────────────────────────┘
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
      ลายเซ็นตรงกับเนื้อหา          เนื้อหาถูกแก้ไข หรือ
      และมาจากเจ้าของ key จริง      ลายเซ็นไม่ตรงกับ key
              │                           │
              ▼                           ▼
        ✅ Verified                   ❌ Unverified /
                                          Invalid
```

จุดสำคัญที่ต้องเข้าใจคือ ลายเซ็นนี้ไม่ได้เป็นแค่ "ป้ายแปะ" ที่คัดลอกไปแปะที่ commit ไหนก็ได้ แต่มันถูกคำนวณมาจาก **เนื้อหาจริงของ commit** (รวมถึง tree hash, parent commit, author, timestamp, และข้อความ commit) ดังนั้น:

- ถ้ามีใครพยายามแก้ไขเนื้อหาของ commit หลังจากเซ็นชื่อไปแล้ว (แม้แค่ตัวอักษรเดียว) ลายเซ็นจะไม่ตรงกับเนื้อหาใหม่ทันที และการ verify จะล้มเหลว
- ลายเซ็นของ commit หนึ่งใช้กับ commit อื่นไม่ได้ เพราะเนื้อหาต่างกัน การคำนวณลายเซ็นก็ต่างกันไปด้วย

### GPG Key Pair ประกอบด้วยอะไรบ้าง

เมื่อสร้าง GPG key คุณจะได้ข้อมูลสำคัญเหล่านี้:

| ส่วนประกอบ | ความหมาย |
|---|---|
| **Key ID** | รหัสสั้น ๆ ใช้อ้างอิงถึง key นี้ (เช่น 8 หรือ 16 ตัวอักษรสุดท้ายของ fingerprint) |
| **Fingerprint** | รหัสยาวเต็มที่ไม่ซ้ำกันเลยในโลก ใช้ยืนยันตัวตนของ key อย่างแม่นยำที่สุด |
| **User ID (UID)** | ชื่อและอีเมลที่ผูกกับ key นี้ (เช่น `Somchai Jaidee <somchai@example.com>`) |
| **Public key** | ส่วนที่เอาไปเผยแพร่/อัปโหลดขึ้น GitHub, GitLab, หรือ keyserver |
| **Private key** | ส่วนที่เก็บเป็นความลับสุดยอด อยู่ในเครื่องเราเท่านั้น |
| **Passphrase** | รหัสผ่านที่ใช้ปกป้อง private key อีกชั้นหนึ่ง (แนะนำให้ตั้งเสมอ) |

ใน Step ถัดไป เราจะลงมือสร้าง GPG key ชุดจริงและนำ public key ไปเพิ่มเข้าบัญชี GitHub/GitLab

---

## Step 783: สร้าง GPG key และเพิ่ม public key เข้า GitHub/GitLab

### 783.1 ตรวจสอบว่าเครื่องมี GPG ติดตั้งอยู่แล้วหรือยัง

```bash
gpg --version
```

ถ้าเห็นเวอร์ชันขึ้นมา (เช่น `gpg (GnuPG) 2.4.3`) แปลว่าพร้อมใช้งานแล้ว ถ้ายังไม่มี ให้ติดตั้งตามระบบปฏิบัติการ:

```bash
# macOS (ผ่าน Homebrew)
brew install gnupg

# Ubuntu / Debian
sudo apt update
sudo apt install gnupg

# Windows
# ดาวน์โหลด Gpg4win จาก https://gpg4win.org
```

### 783.2 สร้าง GPG key ใหม่

ใช้คำสั่ง `gpg --full-generate-key` เพื่อเข้าสู่โหมดสร้าง key แบบเลือกรายละเอียดเองได้ (แนะนำมากกว่า `--quick-generate-key` เพราะควบคุมพารามิเตอร์ได้ครบกว่า):

```bash
gpg --full-generate-key
```

ระบบจะถามคำถามทีละขั้นตอน:

**ขั้นที่ 1 — เลือกประเภทกุญแจ**

```
Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
Your selection?
```

เลือก **(1) RSA and RSA** (ค่าเริ่มต้น) สำหรับกรณีทั่วไป

**ขั้นที่ 2 — เลือกความยาวกุญแจ (key size)**

```
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072)
```

พิมพ์ `4096` เพื่อความปลอดภัยสูงสุดที่แนะนำในปัจจุบัน (ยิ่งบิตมาก ยิ่งปลอดภัย แต่ก็ยิ่งใช้เวลาคำนวณนานขึ้นเล็กน้อย)

**ขั้นที่ 3 — กำหนดวันหมดอายุ**

```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
```

แนะนำให้ตั้งวันหมดอายุ เช่น `1y` (1 ปี) แทนที่จะเลือก "ไม่หมดอายุเลย" เพราะการหมดอายุตามกำหนดช่วยลดความเสี่ยงหากมี key เก่าที่ลืมเพิกถอน (revoke) หลุดออกไป — คุณสามารถต่ออายุ key เดิมได้ภายหลังโดยไม่ต้องสร้างใหม่

**ขั้นที่ 4 — กรอกข้อมูลผู้ใช้ (ต้องตรงกับอีเมลใน Git config)**

```
Real name: Somchai Jaidee
Email address: somchai@example.com
Comment:
```

**สำคัญมาก:** อีเมลที่ใช้ตรงนี้ **ต้องตรงกับอีเมลที่ตั้งไว้ใน `git config user.email`** และต้องเป็นอีเมลที่ยืนยันแล้ว (verified) ในบัญชี GitHub/GitLab ของคุณ ไม่เช่นนั้นระบบจะไม่สามารถจับคู่ commit กับบัญชีผู้ใช้ได้ และจะไม่ขึ้น Verified badge

**ขั้นที่ 5 — ตั้ง Passphrase**

ระบบจะเปิดหน้าต่างแยก (pinentry) ให้ตั้งรหัสผ่านปกป้อง private key ตั้งรหัสที่จำได้และคาดเดายาก **อย่าเว้นว่าง** เพราะถ้า private key หลุดโดยไม่มี passphrase ป้องกัน ใครก็ใช้มันเซ็นชื่อปลอมได้ทันที

หลังจากนี้ GPG จะใช้เวลาสักครู่ในการสร้างเลขสุ่ม (entropy) เพื่อสร้างกุญแจ — ระหว่างนี้ลองขยับเมาส์หรือพิมพ์อะไรเล่น ๆ บนคีย์บอร์ดเพื่อช่วยเพิ่ม entropy ให้ระบบ

### 783.3 ตรวจสอบ key ที่สร้างเสร็จแล้ว

```bash
gpg --list-secret-keys --keyid-format=long
```

ผลลัพธ์ตัวอย่าง (ใช้ placeholder แทนค่าจริง):

```
sec   rsa4096/AAAAAAAAAAAAAAAA 2026-09-26 [SC] [expires: 2027-09-26]
      BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
uid                 [ultimate] Somchai Jaidee <somchai@example.com>
ssb   rsa4096/CCCCCCCCCCCCCCCC 2026-09-26 [E] [expires: 2027-09-26]
```

- บรรทัด `sec` คือ private key หลักที่ใช้เซ็นชื่อ — สังเกตค่า `AAAAAAAAAAAAAAAA` หลัง `rsa4096/` นี่คือ **Key ID** แบบยาว (long format) ที่เราจะใช้อ้างอิงในขั้นตอนถัดไป
- บรรทัดยาว 40 ตัวอักษรใต้บรรทัด `sec` คือ **fingerprint** เต็ม

> หมายเหตุ: ค่า Key ID และ fingerprint ในเอกสารนี้ทั้งหมดเป็น **placeholder เพื่อการสาธิตเท่านั้น** เมื่อคุณสร้าง key จริง ค่าที่เครื่องคุณได้จะไม่เหมือนตัวอย่างนี้เลย และจะไม่มีทางซ้ำกับใครในโลก

### 783.4 Export public key ออกมา

```bash
gpg --armor --export AAAAAAAAAAAAAAAA
```

(แทน `AAAAAAAAAAAAAAAA` ด้วย Key ID จริงของคุณ)

ผลลัพธ์จะเป็นข้อความในรูปแบบ ASCII-armored ที่ขึ้นต้นและลงท้ายแบบนี้:

```
-----BEGIN PGP PUBLIC KEY BLOCK-----

mQINBGX....(ตัวอักษรจำนวนมาก ตัดออกเพื่อความกระชับ)....
=AbCd
-----END PGP PUBLIC KEY BLOCK-----
```

คัดลอกข้อความทั้งหมดนี้ (รวมบรรทัด `BEGIN`/`END`) ไปใช้ในขั้นตอนถัดไป

### 783.5 เพิ่ม public key เข้า GitHub

1. ไปที่ **Settings → SSH and GPG keys** (หน้าเดียวกับที่เราเพิ่ม SSH key ไว้ใน Part 18)
2. คลิกปุ่ม **New GPG key**
3. ตั้งชื่อ (Title) เพื่อจำได้ง่าย เช่น `laptop-work-2026`
4. วางเนื้อหา public key ที่ export มาลงในช่อง **Key**
5. คลิก **Add GPG key**
6. ระบบอาจให้ยืนยันรหัสผ่านบัญชีอีกครั้ง (sudo mode)

หลังเพิ่มสำเร็จ หน้า SSH and GPG keys จะแสดง Key ID, วันหมดอายุ และวันที่เพิ่มไว้ให้ตรวจสอบ

### 783.6 เพิ่ม public key เข้า GitLab

ขั้นตอนคล้ายกันมาก:

1. ไปที่ **Preferences → GPG Keys** (User Settings)
2. วางเนื้อหา public key ในช่อง **Key**
3. คลิก **Add key**

GitLab จะตรวจสอบ key และจับคู่กับ commit ที่เซ็นชื่อด้วย key นั้นโดยอัตโนมัติ ตราบใดที่อีเมลใน UID ของ key ตรงกับอีเมลที่ยืนยันแล้วในบัญชี GitLab ของคุณ

---

## Step 784: ตั้งค่า Git ให้ sign commit ด้วย GPG อัตโนมัติ

### 784.1 บอก Git ว่าจะใช้ GPG key ตัวไหนในการเซ็นชื่อ

```bash
git config --global user.signingkey AAAAAAAAAAAAAAAA
```

(แทน `AAAAAAAAAAAAAAAA` ด้วย Key ID ของคุณจาก Step 783.3)

### 784.2 บอก Git ว่าจะใช้รูปแบบการเซ็นชื่อแบบ GPG (ค่านี้เป็นค่าเริ่มต้นอยู่แล้ว แต่ตั้งชัดเจนไว้เพื่อความแน่ใจ)

```bash
git config --global gpg.format openpgp
```

### 784.3 สั่งให้ Git เซ็นชื่อทุก commit โดยอัตโนมัติ

```bash
git config --global commit.gpgsign true
```

การตั้งค่านี้จะทำให้ **ทุกครั้งที่รัน `git commit` ในทุก repository บนเครื่อง** ระบบจะเซ็นชื่อ commit นั้นด้วย GPG key ที่กำหนดไว้โดยอัตโนมัติ โดยไม่ต้องเติม flag อะไรเพิ่มเลย

หากต้องการเซ็นชื่อ tag ด้วยเช่นกัน (แนะนำสำหรับ release tag) ให้ตั้งค่าเพิ่ม:

```bash
git config --global tag.gpgsign true
```

### 784.4 ทดลอง commit และดูผลลัพธ์

```bash
cd /tmp/spoof-demo   # หรือ repository ทดสอบใด ๆ
git commit --allow-empty -m "ทดสอบ GPG signing"
```

ถ้าตั้ง passphrase ไว้ ระบบจะเปิดหน้าต่าง pinentry ให้กรอกรหัสผ่านของ private key (หรือถามผ่าน terminal ถ้าใช้ pinentry-curses)

### 784.5 ตรวจสอบว่า commit ถูกเซ็นชื่อสำเร็จหรือไม่

```bash
git log --show-signature -1
```

ผลลัพธ์เมื่อสำเร็จ:

```
commit a1b2c3d4e5f6...
gpg: Signature made Sat 26 Sep 2026 10:00:00 AM +07 using RSA key AAAAAAAAAAAAAAAA
gpg: Good signature from "Somchai Jaidee <somchai@example.com>" [ultimate]
Author: Somchai Jaidee <somchai@example.com>
Date:   Sat Sep 26 10:00:00 2026 +0700

    ทดสอบ GPG signing
```

บรรทัด `gpg: Good signature from ...` คือหลักฐานว่า commit นี้ถูกเซ็นชื่อสำเร็จและตรวจสอบผ่านในเครื่อง local แล้ว

### 784.6 ตั้งค่าเฉพาะบาง repository เท่านั้น (ไม่ใช้ global)

ถ้าไม่ต้องการเซ็นชื่อทุก commit ในทุก repository (เช่น มี repository ทดลองที่ไม่สำคัญ) ให้ตัด `--global` ออก แล้วรันคำสั่งภายในโฟลเดอร์ของ repository นั้นแทน:

```bash
cd path/to/important-repo
git config commit.gpgsign true
git config user.signingkey AAAAAAAAAAAAAAAA
```

การตั้งค่าแบบ local (ไม่มี `--global`) จะมีผลเฉพาะ repository นั้นเท่านั้น และจะ override ค่า global ถ้ามีการตั้งไว้ทั้งสองระดับ

### 784.7 ปัญหาที่พบบ่อยเมื่อใช้ GPG signing

| อาการ | สาเหตุที่พบบ่อย | วิธีแก้ |
|---|---|---|
| `gpg: signing failed: No secret key` | Key ID ผิด หรือ key ถูกลบไปแล้ว | ตรวจสอบด้วย `gpg --list-secret-keys --keyid-format=long` แล้วตั้งค่าใหม่ |
| `gpg failed to sign the data` | GPG หา terminal สำหรับถาม passphrase ไม่เจอ (มักเกิดใน SSH session หรือ script) | ตั้งค่า `export GPG_TTY=$(tty)` ใน shell config |
| commit ไม่ขึ้น Verified บน GitHub | อีเมลใน key ไม่ตรงกับอีเมลที่ยืนยันแล้วในบัญชี | ตรวจสอบว่าอีเมลของ UID ตรงกับอีเมลที่ verified ในบัญชี GitHub/GitLab |
| ต้องกรอก passphrase ทุกครั้งที่ commit | ยังไม่ได้ใช้ gpg-agent cache | gpg-agent จะ cache passphrase ไว้ชั่วคราวตามเวลาที่กำหนดค่าไว้ (`default-cache-ttl`) |

---

## Step 785: SSH commit signing — ทางเลือกที่ง่ายกว่า GPG มาก

### ทำไม SSH signing ถึงเป็นที่นิยมมากขึ้นเรื่อย ๆ

ตั้งแต่ **Git 2.34** เป็นต้นมา Git รองรับการใช้ **SSH key** ในการเซ็นชื่อ commit ได้โดยตรง โดยไม่ต้องพึ่งพา GPG เลย นี่คือข่าวดีสำหรับทุกคนที่เรียนจบ **Part 18** มาแล้ว เพราะ:

> **คุณสามารถใช้ SSH key ตัวเดียวกับที่ใช้ push/pull repository อยู่แล้ว มาเซ็นชื่อ commit ได้เลยทันที ไม่ต้องสร้าง key ชุดใหม่ ไม่ต้องติดตั้งโปรแกรมเพิ่ม ไม่ต้องจำ passphrase อีกชุด**

เปรียบเทียบความยุ่งยากระหว่างสองวิธี:

| ขั้นตอน | GPG signing | SSH signing |
|---|---|---|
| ต้องติดตั้งโปรแกรมเพิ่มไหม | ต้อง (GPG) | ไม่ต้อง (ใช้ `ssh-keygen`/`ssh-agent` ที่มีอยู่แล้ว) |
| ต้องสร้าง key ใหม่ไหม | ต้องสร้างแยกต่างหาก | ใช้ SSH key เดิมที่มีอยู่แล้วได้เลย (หรือสร้างใหม่ก็ได้ถ้าต้องการแยก) |
| ความซับซ้อนของขั้นตอนสร้าง key | หลายขั้นตอน มีตัวเลือกเยอะ | ขั้นตอนเดียว คุ้นเคยอยู่แล้วจาก Part 18 |
| การจัดการวันหมดอายุ | มีระบบ built-in | ต้องจัดการเองถ้าต้องการ |
| รองรับบน GitHub/GitLab | รองรับมานาน | รองรับตั้งแต่ราวปี 2022 เป็นต้นมา (เวอร์ชันปัจจุบันรองรับครบ) |

จากตารางนี้จะเห็นว่า **สำหรับคนส่วนใหญ่ SSH signing คือทางเลือกที่แนะนำที่สุด** โดยเฉพาะถ้าคุณตั้งค่า SSH key สำหรับ authentication ไว้แล้วตาม Part 18

### 785.1 ตรวจสอบเวอร์ชัน Git ว่ารองรับ SSH signing หรือไม่

```bash
git --version
```

ต้องเป็น **Git 2.34 ขึ้นไป** ถ้าเวอร์ชันเก่ากว่านี้ ให้อัปเดต Git ก่อน

### 785.2 ตั้งค่า Git ให้ใช้ SSH เป็นรูปแบบการเซ็นชื่อ

```bash
git config --global gpg.format ssh
```

คำสั่งนี้เปลี่ยนโหมดการเซ็นชื่อของ Git จาก GPG (default) ไปเป็น SSH

### 785.3 ระบุ SSH key ที่จะใช้เซ็นชื่อ

ใช้ **public key file** เดิมที่สร้างไว้ตั้งแต่ Part 18 (ไฟล์ `.pub` เท่านั้น ไม่ใช่ private key):

```bash
git config --global user.signingkey ~/.ssh/id_ed25519.pub
```

ถ้าจำ path ของ public key ไม่ได้ ตรวจสอบด้วย:

```bash
ls -la ~/.ssh/*.pub
```

### 785.4 เปิดใช้งาน auto-sign เหมือนเดิม

```bash
git config --global commit.gpgsign true
```

(ใช้ config key ชื่อ `commit.gpgsign` เหมือนเดิม แม้จะเปลี่ยน `gpg.format` เป็น `ssh` แล้วก็ตาม — นี่เป็นชื่อ config ที่ Git เก็บไว้แบบเดิมด้วยเหตุผลทางประวัติศาสตร์)

สรุปคำสั่งตั้งค่า SSH signing ทั้งหมดในที่เดียว:

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

### 785.5 สร้างไฟล์ allowed_signers เพื่อตรวจสอบลายเซ็นแบบ local

ต่างจาก GPG ที่มีระบบ keyring จัดการ trust ให้อยู่แล้ว SSH signing ต้องการไฟล์ที่บอก Git ว่า "public key ตัวไหนที่เชื่อถือได้ ผูกกับอีเมลไหน" เรียกว่า **allowed signers file**

สร้างไฟล์นี้ขึ้นมา:

```bash
mkdir -p ~/.ssh
touch ~/.ssh/allowed_signers
```

เพิ่มบรรทัดข้อมูลของตัวเองลงไป (รูปแบบ: `อีเมล ประเภทกุญแจ เนื้อหา-public-key`):

```bash
echo "somchai@example.com $(cat ~/.ssh/id_ed25519.pub)" >> ~/.ssh/allowed_signers
```

จากนั้นบอก Git ให้รู้จักไฟล์นี้:

```bash
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
```

การตั้งค่านี้ทำให้คำสั่ง `git log --show-signature` สามารถตรวจสอบลายเซ็น SSH ได้ในเครื่อง local โดยไม่ต้องพึ่งพา GitHub/GitLab เลย

### 785.6 ทดลอง commit และตรวจสอบ

```bash
git commit --allow-empty -m "ทดสอบ SSH signing"
git log --show-signature -1
```

ผลลัพธ์ที่คาดหวัง:

```
commit f6e5d4c3b2a1...
Good "git" signature with ED25519 key SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Author: Somchai Jaidee <somchai@example.com>
Date:   Sat Sep 26 10:15:00 2026 +0700

    ทดสอบ SSH signing
```

บรรทัด `Good "git" signature` คือหลักฐานว่า SSH signing ทำงานถูกต้อง

### 785.7 เพิ่ม SSH key เป็น "Signing Key" ในบัญชี GitHub (สำคัญมาก)

นี่คือจุดที่คนมักพลาดบ่อยที่สุด: **การเพิ่ม SSH key เข้า GitHub เพื่อใช้ push/pull (authentication key) กับการเพิ่มเพื่อใช้เซ็นชื่อ (signing key) เป็นคนละรายการกัน แม้จะใช้ key เดียวกันก็ตาม**

ใน Part 18 (Step 174) เราเพิ่ม SSH key เป็นประเภท **Authentication Key** ไปแล้ว สำหรับ Part นี้ต้องเพิ่ม **อีกครั้ง** โดยเลือกประเภทเป็น **Signing Key**:

1. ไปที่ **Settings → SSH and GPG keys**
2. คลิก **New SSH key**
3. ตั้ง Title เช่น `laptop-signing-key`
4. ที่ **Key type** เลือก **Signing Key** (ไม่ใช่ Authentication Key)
5. วางเนื้อหา public key เดิม (`cat ~/.ssh/id_ed25519.pub`)
6. คลิก **Add SSH key**

คุณสามารถเพิ่ม SSH key ตัวเดียวกันได้ทั้งสองประเภทพร้อมกัน (ทั้ง Authentication และ Signing) ในบัญชีเดียว — GitHub จะแสดงทั้งสองรายการแยกกันในหน้า settings แม้จะเป็น key เดียวกันก็ตาม

ถ้าไม่เพิ่มเป็น Signing Key แม้ commit จะถูกเซ็นชื่อสำเร็จในเครื่อง (`Good signature` ตอน `git log --show-signature`) แต่เมื่อ push ขึ้น GitHub จะ **ไม่ขึ้น Verified badge** เพราะ GitHub ไม่รู้ว่า public key นี้ผูกกับบัญชีของคุณในฐานะ signing key

### 785.8 เพิ่ม SSH signing key เข้า GitLab

ขั้นตอนคล้ายกัน:

1. ไปที่ **Preferences → SSH Keys**
2. วาง public key
3. ที่ช่อง **Usage type** เลือก **Signing** หรือ **Authentication & Signing** (แล้วแต่เวอร์ชันของ GitLab)
4. คลิก **Add key**

---

## Step 786: Verified badge บน GitHub/GitLab คืออะไร ทำงานอย่างไร

### หน้าตาของ Verified badge

เมื่อคุณ push commit ที่ถูกเซ็นชื่อถูกต้องขึ้น GitHub หรือ GitLab หน้าเว็บที่แสดงประวัติ commit (commit history) จะมีป้ายกำกับสีเขียวเขียนว่า **"Verified"** ต่อท้ายชื่อผู้เขียนและ commit message เช่น:

```
ทดสอบ SSH signing        somchai   ✅ Verified   f6e5d4c
```

ถ้าคลิกที่ป้าย Verified จะเห็นรายละเอียดเพิ่มเติม เช่น key ID ที่ใช้เซ็นชื่อ และวันที่เพิ่ม key นั้นเข้าบัญชี

### เงื่อนไขทั้งหมดที่ต้องครบเพื่อให้ขึ้น Verified

การจะได้ Verified badge ต้องผ่านเงื่อนไขทุกข้อต่อไปนี้พร้อมกัน:

1. **Commit ต้องถูกเซ็นชื่อจริง** ด้วย GPG หรือ SSH key (ไม่ใช่แค่มี `Signed-off-by` ในข้อความ ซึ่งเป็นคนละเรื่องกัน — `Signed-off-by` เป็นแค่ข้อความยืนยันตามธรรมเนียม DCO ไม่ใช่ cryptographic signature)
2. **Public key ที่ใช้ตรวจสอบ ต้องถูกเพิ่มเข้าบัญชีของผู้เขียนแล้ว** ในฐานะ signing key (GPG key หรือ SSH signing key)
3. **อีเมลของผู้เขียน commit (`user.email`) ต้องตรงกับอีเมลที่ยืนยันแล้ว (verified email) ในบัญชีเดียวกับที่เพิ่ม key ไว้**
4. **Key ต้องยังไม่หมดอายุหรือถูกเพิกถอน (revoke) ในขณะที่สร้าง commit นั้น** (สำหรับ GPG ที่ตั้งวันหมดอายุไว้)

ถ้าขาดข้อใดข้อหนึ่งไป GitHub/GitLab จะแสดงสถานะอื่นแทน Verified เช่น:

| สถานะ | ความหมาย |
|---|---|
| **Verified** | ผ่านทุกเงื่อนไข ตรวจสอบสำเร็จ |
| **Unverified** | มีการเซ็นชื่อ แต่ตรวจสอบไม่ผ่าน (เช่น อีเมลไม่ตรง หรือ key ไม่ได้ผูกกับบัญชี) |
| **No signature / ไม่มีป้ายใด ๆ** | commit ไม่ได้ถูกเซ็นชื่อเลย |
| **Partially verified** (กรณี merge commit) | บาง commit ในกลุ่มที่ merge มาถูกเซ็นชื่อ บางอันไม่ถูก |

### ทำไม Verified badge ถึงสำคัญในทางปฏิบัติ

- **ผู้รีวิวโค้ด (reviewer)** สามารถมั่นใจได้ในระดับหนึ่งว่า commit ที่กำลังรีวิวมาจากบัญชีที่อ้างตัวจริง ไม่ใช่ถูกปลอมแปลงชื่อ
- **ทีม security/compliance** ใช้เป็นหลักฐานประกอบการ audit ว่าการเปลี่ยนแปลงโค้ดแต่ละครั้งมาจากตัวตนที่ตรวจสอบได้
- **โปรเจกต์ Open Source ขนาดใหญ่** มักกำหนดเป็นนโยบายว่า contributor ต้อง sign commit ก่อนจะรับ pull request เข้า main branch เพื่อป้องกันการปลอมแปลงจากคนนอก

---

## Step 787: Vigilant mode — เปิดใช้เพื่อจับ commit ปลอมที่อ้างเป็นเรา

### ปัญหาที่ Vigilant mode แก้ไข

ลองนึกภาพสถานการณ์นี้: คุณเปิดใช้ GPG/SSH signing เรียบร้อยแล้ว และ commit ของคุณเองขึ้น Verified ทุกอัน แต่ **ถ้ามีใครสักคนปลอมชื่อ `user.name`/`user.email` เป็นคุณ แล้วสร้าง commit ที่ไม่ได้เซ็นชื่อ** commit นั้นจะปรากฏในประวัติโดย **ไม่มีป้าย Verified และไม่มีป้าย Unverified ให้เห็นด้วยซ้ำ** — มันจะดูเหมือน commit ธรรมดาทั่วไปที่ยังไม่ได้เซ็นชื่อ ซึ่งคนดูทั่วไปอาจไม่ทันสังเกตว่าเป็น commit ปลอม เพราะ commit ที่ไม่ได้เซ็นชื่อของคนอื่นก็หน้าตาเหมือนกันหมด

**Vigilant mode** คือฟีเจอร์ของ GitHub ที่แก้ปัญหานี้โดยตรง

### Vigilant mode คืออะไร

เมื่อเปิดใช้ **Vigilant mode** ในบัญชี GitHub ระบบจะ:

> **แสดงป้าย "Unverified" อย่างชัดเจนบน commit ทุกอันที่อ้างว่าเป็นของคุณ (ใช้อีเมลของคุณ) แต่ไม่ได้ถูกเซ็นชื่อด้วย key ที่ผูกกับบัญชีของคุณ**

พูดง่าย ๆ คือ เมื่อไม่ได้เปิด Vigilant mode: "commit ที่ไม่ได้เซ็นชื่อ" จะไม่มีป้ายอะไรเลย (เงียบ ๆ ไป) แต่เมื่อเปิด Vigilant mode: "commit ที่อ้างว่าเป็นคุณแต่ไม่ได้เซ็นชื่อ" จะขึ้นป้าย **Unverified** สีเทา/แดงให้เห็นชัดเจน เป็นการเตือนทุกคนที่ดูประวัติว่า **"นี่อาจไม่ใช่ commit จากตัวจริง ระวังไว้"**

### วิธีเปิดใช้งาน Vigilant mode

1. ไปที่ **Settings → SSH and GPG keys**
2. เลื่อนลงมาที่ส่วน **Vigilant mode**
3. เปิดสวิตช์ **"Flag unsigned commits as unverified"**

หลังจากเปิดแล้ว ทุก commit ในอนาคตที่ใช้อีเมลของคุณแต่ไม่ได้เซ็นชื่อ (ไม่ว่าจะสร้างจากเครื่องไหน โดยใครก็ตาม) จะขึ้นป้าย Unverified ทันที

### ผลกระทบที่ควรรู้ก่อนเปิดใช้

- **commit เก่าของตัวเองที่ยังไม่เคย sign มาก่อน** ก็จะขึ้น Unverified ย้อนหลังไปด้วย (ไม่ใช่แค่ commit ใหม่) ซึ่งเป็นเรื่องปกติและถูกต้องแล้ว เพราะ commit เหล่านั้นก็ไม่ได้ถูกเซ็นชื่อจริง ๆ
- ถ้าคุณยังคอมมิตจากบางเครื่องที่ยังไม่ได้ตั้งค่า signing (เช่น เครื่องสำรอง หรือ CI/CD ที่ commit อัตโนมัติ) commit จากเครื่องเหล่านั้นจะขึ้น Unverified ทั้งหมด จนกว่าจะตั้งค่า signing ให้ครบทุกเครื่องที่ใช้งาน
- แนะนำให้เปิด Vigilant mode **หลังจาก** ตั้งค่า signing เรียบร้อยในทุกเครื่องที่ใช้งานประจำแล้วเท่านั้น เพื่อไม่ให้เกิดความสับสนโดยไม่จำเป็น

### GitLab มีฟีเจอร์คล้ายกันหรือไม่

GitLab ไม่มีฟีเจอร์ชื่อ "Vigilant mode" ตรงตัวเหมือน GitHub แต่มีแนวทางที่ใกล้เคียงกันผ่านการตั้งค่า **"Require signed commits"** ในระดับ project (ซึ่งเราจะพูดถึงในมุมของ GitHub Branch Protection ใน Step 788) ที่ปฏิเสธ commit ที่ไม่ได้เซ็นชื่อไม่ให้เข้า branch ที่ป้องกันไว้ตั้งแต่ต้น แทนที่จะปล่อยให้เข้าแล้วค่อยติดป้ายเตือน

---

## Step 788: บังคับ signed commits ทั้งหมดด้วย Branch Protection Rule

### เชื่อมโยงกับ Part 37

ใน **Part 37 (Step 366)** เราได้เกริ่นถึงกฎ **"Require signed commits"** ในหน้า Branch Protection Rules ไว้แล้ว และบอกไว้ว่ารายละเอียดเชิงลึกจะอยู่ใน Part นี้ ตอนนี้เราพร้อมแล้วที่จะเข้าใจกลไกนี้อย่างเต็มรูปแบบ

### ทำไมแค่ "ตั้งใจเอง" ให้ sign commit ยังไม่พอ

จนถึงตอนนี้เราตั้งค่า `commit.gpgsign true` ไว้ในเครื่องของเราเอง ซึ่งหมายความว่า **เราเลือกที่จะ sign commit ของเราเอง** แต่ไม่มีอะไรบังคับให้คนอื่นในทีมทำแบบเดียวกัน ถ้าเพื่อนร่วมทีมอีกคนไม่ได้ตั้งค่า signing ไว้ commit ของเขาก็จะเข้า repository ได้ตามปกติโดยไม่มีลายเซ็นใด ๆ

การจะทำให้ **ทุกคนในทีม ทุก commit ต้องถูกเซ็นชื่อโดยไม่มีข้อยกเว้น** ต้องใช้กลไกระดับ repository/organization ที่บังคับใช้จากฝั่ง server ซึ่งก็คือ **Branch Protection Rule** นั่นเอง

### ขั้นตอนเปิดใช้ "Require signed commits" บน GitHub

1. ไปที่ repository ที่ต้องการ → **Settings → Branches**
2. คลิก **Add branch protection rule** (หรือแก้ไข rule ที่มีอยู่แล้ว)
3. กำหนด branch name pattern เช่น `main` หรือ `release/*`
4. เลื่อนหาช่อง **☑ Require signed commits**
5. คลิก **Create** หรือ **Save changes**

### ผลลัพธ์เมื่อเปิดใช้งาน

เมื่อเปิดกฎนี้แล้ว **ทุก commit ที่พยายาม push เข้า branch ที่ถูกป้องกัน จะต้องมีลายเซ็นดิจิทัลที่ตรวจสอบผ่าน (verified) เท่านั้น** ถ้าใครพยายาม push commit ที่ไม่ได้เซ็นชื่อเข้า branch นั้น GitHub จะปฏิเสธทันทีพร้อมข้อความ error ประมาณนี้:

```
remote: error: GH006: Protected branch update failed for refs/heads/main.
remote: error: Changes must have valid commit signatures.
To github.com:organization/repository.git
 ! [remote rejected] main -> main (protected branch hook declined)
error: failed to push some refs to 'github.com:organization/repository.git'
```

สังเกตว่าการปฏิเสธนี้เกิดขึ้น **ที่ฝั่ง server** ไม่ใช่แค่คำเตือนในเครื่อง local เท่านั้น ทำให้ไม่มีทางที่ commit ที่ไม่ได้เซ็นชื่อจะเข้า branch นี้ได้เลย ไม่ว่าจะพยายามด้วยวิธีใดก็ตาม (รวมถึงผ่าน pull request ด้วย — pull request ที่มี commit ไม่ได้เซ็นชื่ออยู่ใน branch ต้นทาง จะไม่สามารถ merge เข้า branch ที่ป้องกันไว้ได้จนกว่าจะแก้ไข)

### ผลกระทบต่อ Pull Request ที่มี commit ไม่ได้เซ็นชื่อ

ถ้ามี pull request ที่ branch ต้นทางมี commit บางอันไม่ได้เซ็นชื่อ GitHub จะแสดงสถานะ check ที่ล้มเหลวในหน้า pull request พร้อมรายการว่า commit ไหนบ้างที่ยังไม่ผ่าน วิธีแก้ไขทั่วไปคือ:

1. ตั้งค่า signing ให้ถูกต้องในเครื่อง
2. ใช้ `git rebase` เพื่อเซ็นชื่อ commit เก่าใหม่ทั้งหมด เช่น:

```bash
git rebase --exec 'git commit --amend --no-edit -S' -i main
```

หรือถ้าต้องการ sign commit ล่าสุดแค่อันเดียวที่ยังไม่ผ่าน (เช่นลืมตั้งค่าไว้ตอน commit):

```bash
git commit --amend -S --no-edit
git push --force-with-lease
```

(แฟล็ก `-S` คือการบังคับให้เซ็นชื่อ commit นั้นแบบเจาะจง แม้จะยังไม่ได้เปิด `commit.gpgsign true` ไว้ก็ตาม)

### การตั้งค่าเดียวกันใน GitLab

GitLab มีแนวคิดเทียบเท่ากันผ่าน **Push Rules** ระดับ project หรือ instance:

1. ไปที่ **Settings → Repository → Push Rules**
2. เปิดใช้ **"Reject unsigned commits"**
3. บันทึก

ผลลัพธ์เหมือนกับฝั่ง GitHub คือ commit ที่ไม่ได้เซ็นชื่อจะถูกปฏิเสธตั้งแต่ตอน push โดยตรง

### เมื่อไหร่ควรเปิดกฎนี้ ควรเปิดกับทุกทีมหรือไม่

ตามที่กล่าวไว้ใน Part 37 การเปิด "Require signed commits" มีต้นทุนด้าน onboarding เพราะสมาชิกทีมทุกคนต้องตั้งค่า GPG/SSH signing ให้เรียบร้อยก่อนจึงจะ push ได้ แนวทางที่แนะนำในทางปฏิบัติ:

| สถานการณ์ | คำแนะนำ |
|---|---|
| Repository ทั่วไปของทีมภายในที่ไว้ใจกันสูง | เปิด Vigilant mode ก็เพียงพอ ไม่จำเป็นต้องบังคับเต็มรูปแบบ |
| Repository ที่เกี่ยวกับระบบการเงิน ข้อมูลอ่อนไหว หรือต้องผ่าน compliance | เปิด Require signed commits เต็มรูปแบบ |
| Open source project ที่รับ contribution จากคนนอก | เปิด Require signed commits เพื่อป้องกันการปลอมแปลง maintainer |
| Repository สำหรับทดลอง/prototype | ไม่จำเป็นต้องเปิด เพิ่มความยุ่งยากโดยไม่คุ้มค่า |

---

## Step 789: ข้อจำกัดของ commit signing

### ข้อความสำคัญที่สุดของ Part นี้

ก่อนจะจบเนื้อหา มีสิ่งหนึ่งที่ต้องเข้าใจให้ชัดเจนที่สุด เพราะเป็นความเข้าใจผิดที่พบบ่อยมากแม้ในหมู่วิศวกรที่มีประสบการณ์:

> **Commit signing ยืนยันแค่ "ใครเป็นคนสร้าง commit นี้" เท่านั้น มันไม่ได้การันตีเลยว่าโค้ดข้างในนั้นปลอดภัย ถูกต้อง หรือไม่มีบั๊ก**

ลองไล่ดูสิ่งที่ signed commit **ไม่ได้** พิสูจน์ทีละข้อ:

### 1. ไม่ได้พิสูจน์ว่าโค้ดปลอดภัย

ผู้เขียน commit ตัวจริง (ที่มี private key จริง) สามารถเขียนโค้ดที่มีช่องโหว่ด้านความปลอดภัย มี backdoor หรือมี logic ผิดพลาดร้ายแรงได้ตามปกติ แล้วเซ็นชื่อ commit นั้นด้วย key ของตัวเองอย่างถูกต้องสมบูรณ์ signed commit แค่บอกว่า "คนคนนี้ตั้งใจสร้าง commit นี้จริง" ไม่ได้บอกว่า "โค้ดนี้ผ่านการตรวจสอบความปลอดภัยแล้ว"

### 2. ไม่ได้แทนที่การทำ Code Review

ทีมยังคงต้องทำ code review, automated testing, static analysis, และ security scanning ตามปกติ signed commit ไม่ใช่เกตที่แทนที่กระบวนการเหล่านี้ได้เลย มันเป็นแค่ **ชั้นการยืนยันตัวตนของผู้เขียน (identity layer)** ที่ทำงานคู่ขนานไปกับกระบวนการตรวจสอบคุณภาพโค้ด

### 3. ไม่ได้ป้องกัน key ที่ถูกขโมยหรือหลุด

ถ้า private key (GPG หรือ SSH) ของคุณหลุดไปอยู่ในมือคนร้าย ไม่ว่าจะจากเครื่องถูกแฮ็ก หรือไฟล์ key ถูกขโมยไป คนร้ายจะสามารถสร้าง commit ที่ verified สมบูรณ์ในนามของคุณได้ทันที เพราะระบบตรวจสอบแค่ว่า "ลายเซ็นตรงกับ public key ที่ลงทะเบียนไว้หรือไม่" มันไม่มีทางรู้ว่าใครเป็นคนกดคำสั่งจริง ๆ เบื้องหลังกุญแจนั้น ด้วยเหตุนี้การตั้ง **passphrase** ป้องกัน private key และการเก็บรักษา key ให้ปลอดภัย (เช่น ผ่าน hardware security key อย่าง YubiKey) จึงสำคัญไม่แพ้การเปิดใช้ signing เอง

### 4. ไม่ได้ป้องกันการโจมตีที่จุดอื่นของ supply chain

signed commit ป้องกันแค่การปลอมแปลงตัวตนที่ระดับ commit ในตัว repository เอง แต่ไม่ได้ครอบคลุมความเสี่ยงอื่นในห่วงโซ่อุปทานซอฟต์แวร์ (software supply chain) เช่น:

- Dependency ภายนอกที่ดึงมาใช้ (npm package, pip package) อาจมีโค้ดอันตรายซ่อนอยู่ แม้ repository หลักจะบังคับ signed commit ก็ตาม
- Build pipeline หรือ CI/CD server ที่ถูกแฮ็ก อาจแทรกโค้ดอันตรายเข้าไปใน artifact สุดท้าย โดยที่ source code ใน Git ยังคงสะอาดและ signed ถูกต้องทุกอย่าง
- เนื้อหาที่ commit ยืนยันคือ source code ใน repository เท่านั้น ไม่ได้ครอบคลุมถึงขั้นตอน build, package, หรือ deploy

เรื่องนี้จะถูกอธิบายอย่างละเอียดใน **Part 80: Dependency Scanning และ Supply Chain Security** ซึ่งเป็น Part ถัดไปที่ต่อยอดจากความเข้าใจเรื่องนี้โดยตรง

### 5. ไม่ได้ยืนยันว่า commit ถูกสร้างในเวลาที่ระบุจริง

Timestamp ของ commit เป็นค่าที่ผู้สร้าง commit กำหนดเองได้ (ผ่าน `GIT_AUTHOR_DATE`/`GIT_COMMITTER_DATE`) การเซ็นชื่อยืนยันว่าเนื้อหา commit (รวมถึง timestamp ที่ระบุ) มาจากเจ้าของ key จริง แต่ไม่ได้ยืนยันว่า timestamp นั้น "ตรงกับเวลาจริงในโลก" — ผู้เขียนสามารถตั้ง timestamp ย้อนหลังหรือล่วงหน้าแล้วเซ็นชื่อได้ตามปกติ (สำหรับกรณีที่ต้องการหลักฐานเวลาที่เชื่อถือได้ระดับสูงกว่านี้ จะต้องใช้กลไกเพิ่มเติมอย่าง trusted timestamping ซึ่งอยู่นอกขอบเขตของหลักสูตรนี้)

### สรุปมุมมองที่ถูกต้องเกี่ยวกับ commit signing

ตารางสรุปว่า signed commit ทำอะไรได้และทำอะไรไม่ได้:

| commit signing ทำได้ | commit signing ทำไม่ได้ |
|---|---|
| ยืนยันว่า commit มาจากเจ้าของ private key ที่ลงทะเบียนไว้จริง | ยืนยันว่าโค้ดในนั้นปลอดภัยหรือไม่มีบั๊ก |
| ป้องกันการปลอมแปลง `user.name`/`user.email` แบบง่าย ๆ | ป้องกันคนที่ขโมย private key ไปใช้ |
| ตรวจจับได้ทันทีถ้าเนื้อหา commit ถูกแก้ไขหลังเซ็นชื่อ | ครอบคลุมความปลอดภัยของ dependency ภายนอก |
| เพิ่มความน่าเชื่อถือให้ประวัติของ repository | แทนที่ code review หรือ security testing |
| ใช้เป็นหลักฐานประกอบการ audit ตัวตนผู้เขียน | ยืนยันเวลาจริงที่ commit ถูกสร้าง |

จำหลักการนี้ไว้ให้แม่น: **signed commit คือ "การพิสูจน์ตัวตนของผู้เขียน" ไม่ใช่ "ใบรับรองความปลอดภัยของโค้ด"** สองเรื่องนี้เป็นคนละมิติกันโดยสิ้นเชิง และต้องใช้เครื่องมือคนละชุดในการจัดการ

---

## Step 790: แบบฝึกหัด — ตั้งค่า SSH commit signing จนได้ Verified badge จริงบน GitHub

ถึงเวลาลงมือทำจริงทั้งหมด แบบฝึกหัดนี้จะพาคุณตั้งค่า **SSH commit signing** (แนะนำเพราะง่ายกว่า GPG มาก) ตั้งแต่ต้นจนขึ้น Verified badge บน GitHub จริง

### เป้าหมายของแบบฝึกหัด

เมื่อทำเสร็จ คุณจะได้:

1. ตั้งค่า Git ให้เซ็นชื่อ commit ด้วย SSH key อัตโนมัติ
2. เพิ่ม SSH key เป็น Signing Key ในบัญชี GitHub
3. Push commit ที่ขึ้นป้าย **Verified** สีเขียวบนหน้าเว็บ GitHub จริง
4. เข้าใจวิธีตรวจสอบผลลัพธ์ทั้งจากฝั่ง local และฝั่งเว็บ

### ขั้นตอนที่ 1: ตรวจสอบว่ามี SSH key จาก Part 18 อยู่แล้วหรือไม่

```bash
ls -la ~/.ssh/*.pub
```

ถ้าไม่มี ให้สร้างใหม่ตามที่เคยเรียนใน Part 18:

```bash
ssh-keygen -t ed25519 -C "somchai@example.com"
```

(กด Enter ผ่านค่า default ทั้งหมด หรือกำหนด path/passphrase ตามต้องการ)

### ขั้นตอนที่ 2: ตั้งค่า Git ให้ใช้ SSH signing

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

**ตรวจสอบให้แน่ใจว่าอีเมลตรงกัน:**

```bash
git config --global user.email
```

ต้องเป็นอีเมลเดียวกับที่ยืนยันแล้ว (verified) ในบัญชี GitHub ของคุณ ถ้าไม่ตรง ให้แก้ไข:

```bash
git config --global user.email "your-verified-email@example.com"
```

### ขั้นตอนที่ 3: สร้าง allowed_signers file (สำหรับตรวจสอบใน local)

```bash
mkdir -p ~/.ssh
echo "$(git config --global user.email) $(cat ~/.ssh/id_ed25519.pub)" >> ~/.ssh/allowed_signers
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
```

### ขั้นตอนที่ 4: เพิ่ม SSH key เป็น Signing Key บน GitHub

1. คัดลอก public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

2. ไปที่ GitHub → **Settings → SSH and GPG keys → New SSH key**
3. Title: `signing-key-part79-exercise`
4. Key type: **Signing Key**
5. วางเนื้อหา public key ลงในช่อง Key
6. คลิก **Add SSH key**

### ขั้นตอนที่ 5: สร้าง repository ทดสอบและ commit จริง

```bash
mkdir ~/git-course/part-79-signing-exercise
cd ~/git-course/part-79-signing-exercise
git init
echo "# Signed Commit Exercise" > README.md
git add README.md
git commit -m "เพิ่ม README พร้อม SSH signing"
```

ตรวจสอบผลลัพธ์ในเครื่องก่อน:

```bash
git log --show-signature -1
```

ควรเห็นบรรทัด `Good "git" signature with ED25519 key SHA256:...`

### ขั้นตอนที่ 6: สร้าง repository บน GitHub แล้ว push ขึ้นจริง

สร้าง repository ใหม่บนเว็บ GitHub (เช่นชื่อ `part-79-signing-exercise`) แล้วเชื่อมกับเครื่อง:

```bash
git remote add origin git@github.com:your-username/part-79-signing-exercise.git
git branch -M main
git push -u origin main
```

### ขั้นตอนที่ 7: ตรวจสอบ Verified badge บนเว็บ

เปิดหน้า repository บน GitHub แล้วไปที่แท็บ **commits** (หรือดูตรงหน้าหลักของ repo ที่แสดง commit ล่าสุด) ควรเห็น:

```
เพิ่ม README พร้อม SSH signing    your-username   ✅ Verified   a1b2c3d
```

คลิกที่ป้าย **Verified** เพื่อดูรายละเอียด ควรเห็นข้อมูล key ที่ใช้เซ็นชื่อและวันที่เพิ่ม key เข้าบัญชี ตรงกับที่คุณเพิ่งเพิ่มไปในขั้นตอนที่ 4

### ขั้นตอนที่ 8 (ต่อยอด): ทดสอบกรณี commit ที่ "ไม่" ถูก verify

เพื่อให้เข้าใจความแตกต่างชัดเจนขึ้น ลองปิดการ sign ชั่วคราวแล้ว commit เพิ่มดู:

```bash
git commit --allow-empty -m "commit ที่ไม่ได้เซ็นชื่อ" --no-gpg-sign
git push
```

เปิดดูบนเว็บอีกครั้ง จะเห็นว่า commit นี้ **ไม่มีป้าย Verified** (หรือถ้าคุณเปิด Vigilant mode ไว้ตาม Step 787 จะขึ้นป้าย **Unverified** สีแดง/เทาแทน) เปรียบเทียบกับ commit แรกที่ขึ้น Verified ชัดเจน จะช่วยให้เห็นภาพความแตกต่างของกลไกนี้แบบเป็นรูปธรรม

### ขั้นตอนที่ 9 (ทางเลือก): ลองเปิด Branch Protection บน repository ทดสอบนี้

ถ้าต้องการฝึกฝนเพิ่มเติมตาม Step 788 ลองไปที่ **Settings → Branches** ของ repository ทดสอบนี้ เปิด **Require signed commits** บน branch `main` แล้วลอง push commit ที่ไม่ได้เซ็นชื่อ (`--no-gpg-sign`) ดูอีกครั้ง คราวนี้ควรเห็นข้อความปฏิเสธจาก server โดยตรงแทนที่จะแค่ไม่มีป้าย Verified

### Checklist แบบฝึกหัด

- [ ] ตั้งค่า `gpg.format ssh`, `user.signingkey`, `commit.gpgsign true` เรียบร้อย
- [ ] อีเมลใน `git config user.email` ตรงกับอีเมลที่ยืนยันแล้วในบัญชี GitHub
- [ ] สร้าง `allowed_signers` file และตั้งค่า `gpg.ssh.allowedSignersFile`
- [ ] เพิ่ม SSH key เป็น **Signing Key** (แยกจาก Authentication Key) ในบัญชี GitHub
- [ ] `git log --show-signature` แสดงผล `Good "git" signature` สำเร็จในเครื่อง
- [ ] Push commit ขึ้น GitHub จริงและเห็นป้าย **Verified** สีเขียวบนหน้าเว็บ
- [ ] ทดลอง commit แบบไม่ sign ด้วย `--no-gpg-sign` แล้วเห็นความแตกต่างชัดเจน
- [ ] (ทางเลือก) ทดลองเปิด Require signed commits แล้วเห็น server ปฏิเสธ commit ที่ไม่ได้เซ็นชื่อ

---

## สรุป Part 79

ใน Part นี้เราได้เจาะลึกเต็มรูปแบบเรื่อง Git Security ด้าน commit signing ซึ่งเคยเกริ่นไว้ตั้งแต่ Part 18 และ Part 37:

1. **ปัญหาที่ commit signing แก้ไข** คือ Git ปกติไม่ตรวจสอบตัวตนผู้เขียนเลย ทำให้ปลอมชื่อ author/committer ได้ง่ายมาก
2. **GPG** ใช้หลักการ public/private key cryptography — private key เก็บเป็นความลับใช้เซ็นชื่อ, public key เผยแพร่ใช้ตรวจสอบ ปลอมแปลงลายเซ็นไม่ได้ในทางปฏิบัติถ้าไม่มี private key จริง
3. การสร้าง GPG key ใช้ `gpg --full-generate-key` แล้วนำ public key ไปเพิ่มเข้าบัญชี GitHub/GitLab ผ่านหน้า SSH and GPG keys
4. ตั้งค่า Git ให้ sign อัตโนมัติด้วย `git config user.signingkey`, `git config commit.gpgsign true`
5. **SSH commit signing** คือทางเลือกที่ง่ายกว่ามาก เพราะใช้ SSH key เดิมที่มีอยู่แล้วจาก Part 18 มาเซ็นชื่อได้เลย ผ่าน `gpg.format ssh` — ต้องเพิ่ม key เดียวกันเข้าบัญชีอีกครั้งในฐานะ **Signing Key** แยกจาก Authentication Key
6. **Verified badge** สีเขียวบน GitHub/GitLab แสดงเมื่อ commit ถูกเซ็นชื่อถูกต้อง และมี key ผูกกับบัญชีที่อีเมลตรงกัน
7. **Vigilant mode** เปิดใช้เพื่อจับ commit ที่อ้างเป็นเราแต่ไม่ได้เซ็นชื่อ ทำให้มันขึ้นป้าย Unverified อย่างชัดเจนแทนที่จะเงียบไปเฉย ๆ
8. **Branch Protection Rule "Require signed commits"** บังคับให้ทุก commit ที่เข้า branch ที่ป้องกันต้องผ่านการเซ็นชื่อ ปฏิเสธจากฝั่ง server โดยตรงถ้าไม่ผ่าน
9. **ข้อจำกัดสำคัญที่สุด**: commit signing ยืนยันแค่ "ใครเป็นคนสร้าง commit นี้" เท่านั้น ไม่ได้การันตีว่าโค้ดปลอดภัยหรือถูกต้อง ไม่แทนที่ code review, ไม่ป้องกัน key ที่ถูกขโมย และไม่ครอบคลุมความเสี่ยงอื่นในห่วงโซ่อุปทานซอฟต์แวร์
10. แบบฝึกหัดพาตั้งค่า SSH commit signing ตั้งแต่ต้นจนได้ Verified badge จริงบน GitHub รวมถึงทดลองเปรียบเทียบ commit ที่ sign กับไม่ sign

**ต่อไป:** [Part 80: Dependency Scanning และ Supply Chain Security](./part-080-dependency-scanning-supply-chain.md)
