# Part 18: SSH Key และการตั้งค่าความปลอดภัยในการเชื่อมต่อ

> **Step ในหลักสูตรนี้:** Step 171–180
> **เฟส:** 3 — ใช้งาน GitHub อย่างมืออาชีพ
> **เป้าหมายของ Part นี้:** เข้าใจความแตกต่างระหว่างการเชื่อมต่อ GitHub แบบ HTTPS กับ SSH อย่างถ่องแท้ สร้างและจัดการ SSH key ได้เองตั้งแต่ต้นจนจบ ตั้งค่า ssh-agent และไฟล์ `~/.ssh/config` สำหรับใช้งานหลายบัญชีพร้อมกัน รู้จัก Git Credential Manager, แนวคิดเรื่อง commit signing และ Deploy keys เพื่อให้การเชื่อมต่อระหว่างเครื่องของคุณกับ GitHub ปลอดภัยและใช้งานได้จริงในทุกสถานการณ์

---

## สารบัญของ Part นี้

- Step 171: เปรียบเทียบ HTTPS vs SSH สำหรับเชื่อมต่อ GitHub
- Step 172: สร้าง SSH key pair ด้วย `ssh-keygen`
- Step 173: เพิ่ม SSH key เข้า ssh-agent
- Step 174: เพิ่ม public key เข้า GitHub account
- Step 175: ทดสอบการเชื่อมต่อด้วย `ssh -T git@github.com`
- Step 176: จัดการ SSH key หลายตัวสำหรับหลายบัญชี GitHub ด้วย `~/.ssh/config`
- Step 177: Git Credential Manager สำหรับการใช้งานผ่าน HTTPS
- Step 178: เกริ่น GPG/SSH commit signing คืออะไร
- Step 179: Deploy keys คืออะไร ต่างจาก personal SSH key อย่างไร
- Step 180: แบบฝึกหัด — ตั้งค่า SSH เต็มรูปแบบตั้งแต่สร้าง key จนถึง push repo สำเร็จ

---

## Step 171: เปรียบเทียบ HTTPS vs SSH สำหรับเชื่อมต่อ GitHub

เวลาที่คุณ `git clone`, `git push` หรือ `git pull` กับ GitHub เครื่องของคุณต้อง "พิสูจน์ตัวตน" กับ GitHub ก่อนเสมอว่าคุณเป็นใครและมีสิทธิ์เข้าถึง repository นั้นหรือไม่ วิธีการยืนยันตัวตนที่ GitHub รองรับหลัก ๆ มี 2 แบบคือ **HTTPS** และ **SSH**

### URL ของ repository ทั้งสองแบบหน้าตาต่างกันอย่างไร

เวลาคุณกด "Code" บนหน้า repository ของ GitHub จะเห็นตัวเลือก URL สองแบบ:

```
# แบบ HTTPS
https://github.com/username/repository.git

# แบบ SSH
git@github.com:username/repository.git
```

สังเกตความต่างของรูปแบบ: HTTPS ขึ้นต้นด้วย `https://` เหมือน URL เว็บทั่วไป ส่วน SSH จะมีรูปแบบ `user@host:path` ซึ่งเป็นรูปแบบมาตรฐานของโปรโตคอล SSH

### HTTPS ทำงานอย่างไร

เมื่อ clone หรือ push ผ่าน HTTPS ครั้งแรก Git จะถามชื่อผู้ใช้และรหัสผ่าน (ปัจจุบัน GitHub ไม่รับรหัสผ่านบัญชีตรง ๆ แล้ว ต้องใช้ **Personal Access Token (PAT)** แทนรหัสผ่าน หรือใช้ Git Credential Manager ช่วยจัดการ ซึ่งเราจะพูดถึงใน Step 177) หลังจากนั้นระบบปฏิบัติการจะช่วยจดจำ credential ไว้ให้ผ่าน credential helper เพื่อไม่ต้องกรอกซ้ำทุกครั้ง

```bash
git clone https://github.com/username/repository.git
```

### SSH ทำงานอย่างไร

SSH ใช้กลไกการเข้ารหัสแบบ **กุญแจสาธารณะ/กุญแจส่วนตัว (public-key cryptography)** แทนการใช้รหัสผ่านหรือ token คุณสร้าง key pair ไว้ในเครื่องหนึ่งครั้ง เอา public key ไปฝากไว้ที่ GitHub แล้วหลังจากนั้นทุกครั้งที่เชื่อมต่อ เครื่องของคุณจะพิสูจน์ตัวตนด้วย private key โดยอัตโนมัติ ไม่ต้องพิมพ์อะไรซ้ำอีกเลย (ยกเว้นตอนที่คุณตั้ง passphrase ไว้กับ key)

```bash
git clone git@github.com:username/repository.git
```

### ตารางเปรียบเทียบข้อดีข้อเสีย

| ประเด็น | HTTPS | SSH |
|---|---|---|
| การตั้งค่าเริ่มต้น | ง่ายกว่า ไม่ต้องสร้าง key ล่วงหน้า | ต้องสร้าง key pair และนำไปฝากที่ GitHub ก่อน |
| การยืนยันตัวตนแต่ละครั้ง | ต้องใช้ Personal Access Token หรือพึ่ง credential helper ช่วยจำ | ใช้ private key อัตโนมัติ ไม่ต้องพิมพ์อะไรซ้ำ (ถ้าตั้งค่า ssh-agent ไว้) |
| ความปลอดภัยของการเข้ารหัส | เข้ารหัสผ่าน TLS เหมือนเว็บทั่วไป | เข้ารหัสด้วย public-key cryptography ที่ออกแบบมาเพื่อการยืนยันตัวตนโดยเฉพาะ |
| การใช้งานผ่าน Firewall องค์กร | ใช้ port 443 (HTTP/HTTPS) ซึ่งเปิดอยู่แทบทุกที่ | ใช้ port 22 ซึ่งบางองค์กรบล็อกไว้ (แก้ได้ด้วยการตั้งค่าให้ผ่าน port 443 แทน) |
| การเพิกถอนสิทธิ์ | ลบ/หมดอายุ Personal Access Token ได้จากหน้า Settings | ลบ public key ออกจากบัญชีได้ทันทีจากหน้า Settings |
| ใช้กับหลายบัญชีพร้อมกัน | ต้องสลับ token/credential ไปมา ยุ่งยากกว่า | ตั้งค่า `~/.ssh/config` แยก Host ได้ สลับบัญชีลื่นไหลกว่ามาก |
| เหมาะกับ | เครื่องที่ใช้ชั่วคราว, สภาพแวดล้อมที่ตั้งค่า SSH ไม่ได้, CI บางประเภท | เครื่องที่ใช้งานประจำ, นักพัฒนาที่ push/pull บ่อย, ต้องการความสะดวกระยะยาว |

### สรุปคำแนะนำ

ไม่มีแบบไหน "ถูก" หรือ "ผิด" อย่างเด็ดขาด แต่ในทางปฏิบัติ **นักพัฒนาส่วนใหญ่ที่ใช้งานประจำมักเลือก SSH** เพราะตั้งค่าครั้งเดียวแล้วสะดวกไปตลอด ไม่ต้องยุ่งกับการต่ออายุ token บ่อย ๆ ส่วน HTTPS เหมาะกับกรณีที่ตั้งค่า SSH ไม่ได้ เช่น เครื่องสาธารณะ หรือสภาพแวดล้อม container บางประเภทที่ไม่สะดวกฝัง private key ไว้

ใน Part นี้เราจะโฟกัสที่การตั้งค่า SSH ให้ใช้งานได้เต็มรูปแบบ ส่วน Git Credential Manager สำหรับฝั่ง HTTPS จะพูดถึงใน Step 177 เพื่อให้คุณเลือกใช้ได้ทั้งสองทางตามสถานการณ์

---

## Step 172: สร้าง SSH key pair ด้วย `ssh-keygen -t ed25519 -C "email"`

### แนวคิดเรื่อง Public Key / Private Key

SSH ใช้ **กุญแจคู่ (key pair)** ที่มีความสัมพันธ์ทางคณิตศาสตร์กัน:

- **Private Key (กุญแจส่วนตัว)** — เก็บไว้ในเครื่องของคุณเท่านั้น **ห้ามเผยแพร่หรือส่งให้ใครเด็ดขาด** ใช้สำหรับ "เซ็นชื่อ" พิสูจน์ตัวตนของคุณ
- **Public Key (กุญแจสาธารณะ)** — เผยแพร่ได้อย่างอิสระ นำไปฝากไว้ที่ GitHub เพื่อให้ GitHub ใช้ตรวจสอบว่า "ลายเซ็น" ที่ส่งมาจาก private key คู่กันจริงหรือไม่

หลักการสำคัญคือ **private key ไม่เคยถูกส่งออกจากเครื่องของคุณเลย** — สิ่งที่ถูกส่งไปมาระหว่างเครื่องคุณกับ GitHub คือผลลัพธ์ทางคณิตศาสตร์ที่พิสูจน์ได้ว่าคุณถือ private key อยู่จริง โดยไม่ต้องเปิดเผยตัว private key เอง นี่คือเหตุผลที่ระบบนี้ปลอดภัยกว่าการส่งรหัสผ่านไปมา

### เลือกอัลกอริทึม: ทำไมต้อง Ed25519

Git/SSH รองรับหลายอัลกอริทึม เช่น RSA, DSA, ECDSA, Ed25519 แต่ปัจจุบัน **Ed25519** คือตัวเลือกที่แนะนำที่สุด:

| อัลกอริทึม | ข้อดี | ข้อเสีย |
|---|---|---|
| RSA (เดิมนิยม 2048/4096 บิต) | รองรับในระบบเก่าแทบทุกที่ | key ยาว, ความเร็วต่ำกว่า, ต้องใช้ความยาว 4096 บิตขึ้นไปถึงจะปลอดภัยเพียงพอในปัจจุบัน |
| ECDSA | เร็วกว่า RSA | มีข้อกังวลเรื่องคุณภาพของค่าสุ่มที่ใช้ในบางการ implement |
| **Ed25519** | เร็ว, ปลอดภัยสูง, key สั้นกระชับ, ออกแบบมาเพื่อหลีกเลี่ยงจุดอ่อนที่เคยพบใน algorithm อื่น | ระบบเก่ามาก ๆ (SSH เวอร์ชันโบราณ) อาจไม่รองรับ ซึ่งปัจจุบันแทบไม่ใช่ปัญหาแล้ว |

GitHub แนะนำ Ed25519 อย่างเป็นทางการเช่นกัน เราจะใช้อัลกอริทึมนี้ตลอด Part นี้

### คำสั่งสร้าง SSH key

เปิด Terminal แล้วรันคำสั่ง:

```bash
ssh-keygen -t ed25519 -C "phutjirakul.iam@gmail.com"
```

อธิบายแต่ละส่วนของคำสั่ง:

- `ssh-keygen` — โปรแกรมสร้าง SSH key ที่ติดตั้งมาพร้อม SSH client อยู่แล้วในเกือบทุกระบบปฏิบัติการ (Linux, macOS, และ Windows ผ่าน Git Bash หรือ OpenSSH ที่มากับ Windows 10/11)
- `-t ed25519` — ระบุประเภทของ key ว่าเป็น Ed25519
- `-C "email"` — comment ที่แนบไปกับ public key เพื่อให้จำได้ง่ายว่า key นี้เป็นของใคร/เครื่องไหน มักใช้อีเมลที่ผูกกับบัญชี GitHub

เมื่อรันคำสั่งแล้ว ระบบจะถามคำถามตามลำดับ:

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/user/.ssh/id_ed25519):
```

ถ้าเป็น key แรกในเครื่อง กด **Enter** เพื่อใช้ path ตั้งต้น (`~/.ssh/id_ed25519`) ได้เลย แต่ถ้าคุณมี key อยู่แล้วและต้องการสร้าง key ใหม่แยกไว้อีกชื่อ (เช่นสำหรับบัญชีที่สอง) ให้พิมพ์ path ใหม่ เช่น `/home/user/.ssh/id_ed25519_work`

```
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

ระบบจะถาม **passphrase** — นี่คือรหัสผ่านเพิ่มเติมที่ใช้ปลดล็อก private key ก่อนใช้งาน แนะนำอย่างยิ่งให้ตั้ง passphrase ไว้เสมอ เพราะถ้าเครื่องของคุณถูกขโมยหรือ private key หลุดออกไป คนร้ายจะยังใช้งานไม่ได้ถ้าไม่รู้ passphrase (ในหัวข้อถัดไปเราจะตั้งค่า ssh-agent เพื่อไม่ต้องพิมพ์ passphrase ซ้ำทุกครั้ง)

เมื่อเสร็จแล้วจะเห็นผลลัพธ์ประมาณนี้:

```
Your identification has been saved in /home/user/.ssh/id_ed25519
Your public key has been saved in /home/user/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx phutjirakul.iam@gmail.com
The key's randomart image is:
+--[ED25519 256]--+
|      .o+o..     |
|     .  =o+.     |
|      .o.=+ .    |
|     . +.*+o     |
|      . So=.o    |
|       .+ =.o    |
|      . o + +    |
|       o = = .   |
|        . E.o    |
+----[SHA256]-----+
```

### ตรวจสอบไฟล์ที่ถูกสร้างขึ้น

```bash
ls -la ~/.ssh/
```

ควรเห็นไฟล์ 2 ไฟล์ (อย่างน้อย):

```
id_ed25519       ← Private key (ห้ามแชร์เด็ดขาด)
id_ed25519.pub   ← Public key (แชร์ได้ นำไปฝากที่ GitHub)
```

ตรวจสอบ permission ของไฟล์ private key ให้แน่ใจว่าเข้มงวดพอ (เฉพาะเจ้าของอ่าน/เขียนได้เท่านั้น):

```bash
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

ถ้า permission ของไฟล์ private key หลวมเกินไป (เช่นคนอื่นในเครื่องอ่านได้) โปรแกรม `ssh` อาจปฏิเสธไม่ยอมใช้ key นั้นเลยพร้อม error แจ้งเรื่อง permission

### ดูเนื้อหา public key

```bash
cat ~/.ssh/id_ed25519.pub
```

จะได้ข้อความยาวประมาณนี้ (ตัวอย่าง):

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX phutjirakul.iam@gmail.com
```

นี่คือ public key ที่เราจะนำไปฝากไว้ที่ GitHub ใน Step 174 **อย่าพยายามดูเนื้อหาไฟล์ `id_ed25519` (ไม่มีนามสกุล .pub) เด็ดขาด และไม่ต้องส่งให้ใครดูเลย** เพราะนั่นคือ private key

---

## Step 173: เพิ่ม SSH key เข้า ssh-agent

### ssh-agent คืออะไร ทำไมต้องใช้

ถ้าคุณตั้ง passphrase ไว้กับ private key (ซึ่งแนะนำให้ตั้ง) ปกติแล้วทุกครั้งที่ใช้ key นั้นเชื่อมต่อ SSH คุณจะต้องพิมพ์ passphrase ซ้ำ ๆ ทุกครั้ง ซึ่งน่ารำคาญมากถ้าต้อง push/pull บ่อย ๆ ทั้งวัน

**ssh-agent** คือโปรแกรมพื้นหลัง (background process) ที่ทำหน้าที่ "จำ" private key ที่ปลดล็อกแล้วไว้ในหน่วยความจำชั่วคราว เพื่อให้คุณพิมพ์ passphrase แค่ครั้งเดียวต่อ session แล้วใช้งานได้เรื่อย ๆ โดยไม่ต้องพิมพ์ซ้ำอีก

### เริ่มการทำงานของ ssh-agent

บน Linux และ macOS รันคำสั่ง:

```bash
eval "$(ssh-agent -s)"
```

คำสั่งนี้ทำสองอย่าง:

1. รันโปรแกรม `ssh-agent -s` ซึ่งจะพิมพ์คำสั่ง shell (เช่นตั้งค่า environment variable `SSH_AUTH_SOCK` และ `SSH_AGENT_PID`) ออกมาเป็นข้อความ
2. `eval "$(...)"` จะนำข้อความนั้นมารันเป็นคำสั่งจริงใน shell ปัจจุบัน เพื่อให้ shell รู้จักว่า agent ตัวไหนกำลังทำงานอยู่

ผลลัพธ์ที่เห็นจะประมาณนี้:

```
Agent pid 12345
```

### เพิ่ม private key เข้า agent

```bash
ssh-add ~/.ssh/id_ed25519
```

ถ้าคุณตั้ง passphrase ไว้ ระบบจะถามให้กรอกหนึ่งครั้ง:

```
Enter passphrase for /home/user/.ssh/id_ed25519:
Identity added: /home/user/.ssh/id_ed25519 (phutjirakul.iam@gmail.com)
```

หลังจากขั้นตอนนี้ ตราบใดที่ ssh-agent ยัง run อยู่ (โดยทั่วไปคือจนกว่าจะปิด terminal session หรือรีสตาร์ทเครื่อง) คุณจะไม่ต้องพิมพ์ passphrase ซ้ำอีก

### ตรวจสอบ key ที่อยู่ใน agent

```bash
ssh-add -l
```

จะเห็นรายการ key ที่ agent จำอยู่ พร้อม fingerprint:

```
256 SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx phutjirakul.iam@gmail.com (ED25519)
```

ถ้าต้องการดูรายการแบบเต็ม (fingerprint + randomart) ใช้:

```bash
ssh-add -L
```

(ตัว `-L` พิมพ์ใหญ่จะแสดง public key แบบเต็ม ส่วน `-l` พิมพ์เล็กแสดงแค่ fingerprint สั้น ๆ)

### ลบ key ออกจาก agent

ถ้าต้องการลบ key ทั้งหมดออกจาก agent (เช่นก่อนปิดเครื่องในที่สาธารณะ):

```bash
ssh-add -D
```

หรือลบเฉพาะ key ตัวใดตัวหนึ่ง:

```bash
ssh-add -d ~/.ssh/id_ed25519
```

### ทำให้ ssh-agent เริ่มทำงานอัตโนมัติทุกครั้งที่เปิด terminal (Linux/macOS)

ปัญหาของคำสั่ง `eval "$(ssh-agent -s)"` คือถ้าปิด terminal แล้วเปิดใหม่ agent เดิมจะหายไป ต้องรันคำสั่งใหม่ทุกครั้ง วิธีแก้คือเพิ่มคำสั่งไว้ในไฟล์ config ของ shell เช่น `~/.bashrc` หรือ `~/.zshrc`:

```bash
# เพิ่มบรรทัดนี้ต่อท้ายไฟล์ ~/.bashrc หรือ ~/.zshrc
if [ -z "$SSH_AUTH_SOCK" ]; then
   eval "$(ssh-agent -s)" > /dev/null
   ssh-add -q --apple-use-keychain ~/.ssh/id_ed25519 2>/dev/null || ssh-add -q ~/.ssh/id_ed25519 2>/dev/null
fi
```

จากนั้นรีโหลด config:

```bash
source ~/.bashrc   # หรือ source ~/.zshrc
```

### กรณี macOS: ใช้ Keychain ช่วยจำ passphrase ถาวร

บน macOS ระบบมี Keychain ที่สามารถจำ passphrase ของ SSH key ไว้ได้อย่างถาวร (ไม่ต้องพิมพ์ซ้ำแม้รีสตาร์ทเครื่อง) โดยใช้ flag `--apple-use-keychain`:

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

และควรตั้งค่าไฟล์ `~/.ssh/config` ให้โหลด key จาก Keychain อัตโนมัติทุกครั้งที่เปิด terminal ใหม่ (รายละเอียดไฟล์ config จะอธิบายเพิ่มเติมใน Step 176):

```
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

### กรณี Windows: ใช้ OpenSSH ที่มากับ Windows หรือ Git Bash

บน Windows 10/11 ที่ติดตั้ง OpenSSH client มาให้ในตัว หรือใช้ผ่าน Git Bash คำสั่งเดียวกันทุกตัวใช้งานได้เหมือนกันทุกประการ (`ssh-agent`, `ssh-add`) เพียงแต่ถ้าต้องการให้ ssh-agent เริ่มทำงานอัตโนมัติทุกครั้งที่เปิดเครื่อง ควรตั้งค่า Windows Service ชื่อ `ssh-agent` ให้เป็น Automatic startup type ผ่าน PowerShell (สิทธิ์ Administrator):

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
```

---

## Step 174: เพิ่ม public key เข้า GitHub account

เมื่อ ssh-agent จำ private key ไว้เรียบร้อยแล้ว ขั้นตอนต่อไปคือบอกให้ GitHub รู้จัก public key ของคุณ

### 174.1 คัดลอกเนื้อหา public key

**Linux (มี xclip):**

```bash
xclip -selection clipboard < ~/.ssh/id_ed25519.pub
```

**macOS:**

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

**Windows (Git Bash หรือ PowerShell):**

```bash
clip < ~/.ssh/id_ed25519.pub
```

ถ้าเครื่องไม่มีเครื่องมือคัดลอกเข้า clipboard โดยตรง ใช้คำสั่งนี้แสดงเนื้อหาแล้ว copy ด้วยมือแทน:

```bash
cat ~/.ssh/id_ed25519.pub
```

จากนั้นเลือกและคัดลอกข้อความทั้งบรรทัดที่แสดงออกมา (ตั้งแต่ `ssh-ed25519` ไปจนถึงอีเมลท้ายบรรทัด)

### 174.2 เพิ่ม key เข้าในบัญชี GitHub

1. เข้าเว็บ GitHub แล้ว login เข้าบัญชีของคุณ
2. คลิกรูปโปรไฟล์มุมขวาบน แล้วเลือก **Settings**
3. ในเมนูด้านซ้าย เลือก **SSH and GPG keys**
4. คลิกปุ่ม **New SSH key** (สีเขียว)
5. กรอกข้อมูล:
   - **Title** — ชื่อที่ช่วยให้จำได้ว่า key นี้มาจากเครื่องไหน เช่น `MacBook Pro ส่วนตัว` หรือ `Work Laptop - Ubuntu`
   - **Key type** — เลือก **Authentication Key** (สำหรับ push/pull ปกติ) ต่างจาก **Signing Key** ที่ใช้เซ็นชื่อ commit (จะพูดถึงใน Step 178)
   - **Key** — วางเนื้อหา public key ที่คัดลอกมาลงในช่องนี้
6. คลิก **Add SSH key**
7. ระบบอาจให้ยืนยันรหัสผ่านบัญชีอีกครั้งเพื่อความปลอดภัย (sudo mode)

### 174.3 ตรวจสอบว่า key ถูกเพิ่มสำเร็จ

หลังเพิ่มเสร็จ หน้า **SSH and GPG keys** จะแสดงรายการ key พร้อมข้อมูล:

- Title ที่คุณตั้ง
- ประเภท key (ED25519)
- Fingerprint (ตัวเลข hash สั้น ๆ ที่ใช้ระบุ key แต่ละตัวโดยไม่ต้องแสดงเนื้อหาเต็ม)
- วันที่เพิ่ม และวันที่ใช้งานล่าสุด (Last used)

### ข้อควรระวังเรื่องความปลอดภัย

- **ห้ามวางเนื้อหาไฟล์ `id_ed25519` (private key)** ลงในช่องนี้เด็ดขาด ต้องเป็นไฟล์ `.pub` เท่านั้น
- ถ้าเผลอเพิ่ม key ผิด หรือสงสัยว่า key หลุด ให้กดปุ่ม **Delete** ข้าง key นั้นได้ทันทีจากหน้าเดียวกัน — GitHub จะปฏิเสธการเชื่อมต่อจาก key นั้นทันทีหลังลบ
- ควรตั้งชื่อ Title ให้สื่อความหมายชัดเจน โดยเฉพาะถ้าคุณมีหลายเครื่อง จะได้รู้ว่าควรลบ key ตัวไหนเมื่อเลิกใช้เครื่องนั้นแล้ว

---

## Step 175: ทดสอบการเชื่อมต่อด้วย `ssh -T git@github.com`

หลังเพิ่ม public key เข้า GitHub แล้ว มาทดสอบว่าการเชื่อมต่อทำงานจริงหรือไม่ ก่อนที่จะไปลอง clone/push repo จริง

### รันคำสั่งทดสอบ

```bash
ssh -T git@github.com
```

อธิบายคำสั่ง:

- `ssh` — โปรแกรม SSH client
- `-T` — ปิดการจัดสรร pseudo-terminal เพราะเราแค่ต้องการทดสอบการยืนยันตัวตน ไม่ได้ต้องการเปิด shell แบบโต้ตอบ
- `git@github.com` — เชื่อมต่อไปยัง GitHub ด้วย user `git` (GitHub ใช้ user นี้เป็นมาตรฐานสำหรับการเชื่อมต่อ Git ผ่าน SSH ทุกบัญชี)

### ครั้งแรกที่เชื่อมต่อ: คำถามเรื่อง host fingerprint

ถ้านี่เป็นครั้งแรกที่เครื่องของคุณเชื่อมต่อไปยัง `github.com` ผ่าน SSH จะเห็นข้อความแบบนี้ก่อน:

```
The authenticity of host 'github.com (140.82.121.3)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

นี่คือกลไกป้องกันการโดนโจมตีแบบ Man-in-the-Middle — SSH ต้องการให้คุณยืนยันว่า "เซิร์ฟเวอร์ที่กำลังเชื่อมต่ออยู่นี้คือ GitHub จริง ๆ" fingerprint ข้างต้นสามารถตรวจสอบเทียบกับ fingerprint อย่างเป็นทางการที่ GitHub เผยแพร่ไว้ที่หน้า [GitHub's SSH key fingerprints documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints) ได้

พิมพ์ `yes` แล้วกด Enter เพื่อยืนยัน:

```
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
```

ข้อความนี้บอกว่า fingerprint ของ github.com ถูกบันทึกไว้ในไฟล์ `~/.ssh/known_hosts` แล้ว ครั้งต่อไปที่เชื่อมต่อจะไม่ถามซ้ำอีก (นอกจากว่า fingerprint เปลี่ยนไป ซึ่งจะเป็นสัญญาณเตือนว่าอาจมีการโจมตีเกิดขึ้น)

### ผลลัพธ์เมื่อเชื่อมต่อสำเร็จ

```
Hi phutjirakul-iam! You've successfully authenticated, but GitHub does not provide shell access.
```

ข้อความนี้คือสัญญาณว่า **การตั้งค่า SSH ของคุณสำเร็จสมบูรณ์แล้ว** สังเกตว่าชื่อ `phutjirakul-iam` คือ username GitHub ของคุณที่ผูกกับ key นี้ ส่วนข้อความ "does not provide shell access" เป็นเรื่องปกติ เพราะ GitHub ไม่ได้ให้เข้าใช้ shell จริง ๆ เพียงแค่ใช้ SSH protocol สำหรับยืนยันตัวตนและรับส่งข้อมูล Git เท่านั้น

### ถ้าเชื่อมต่อไม่สำเร็จ ควรทำอย่างไร

ใช้โหมด verbose เพื่อดูรายละเอียดว่าขั้นตอนไหนล้มเหลว:

```bash
ssh -vT git@github.com
```

หรือเพิ่มระดับความละเอียดสูงสุด:

```bash
ssh -vvv git@github.com
```

ปัญหาที่พบบ่อยและวิธีแก้:

| อาการ | สาเหตุที่เป็นไปได้ | วิธีแก้ |
|---|---|---|
| `Permission denied (publickey)` | ยังไม่ได้เพิ่ม key เข้า agent หรือ public key ยังไม่ถูกเพิ่มที่ GitHub | รัน `ssh-add -l` เช็คว่า agent มี key อยู่ไหม, เช็คหน้า Settings > SSH keys บน GitHub |
| ค้างนานไม่ตอบสนอง | Firewall องค์กรบล็อก port 22 | ดูวิธีแก้ใน Step 176 (ตั้งค่าให้ SSH ผ่าน port 443 แทน) |
| `Host key verification failed` | fingerprint ใน known_hosts ไม่ตรงกับของจริง (อาจเป็น attack หรือไฟล์ known_hosts เสียหาย) | ตรวจสอบ fingerprint กับหน้า documentation อย่างเป็นทางการของ GitHub ก่อนแก้ไขไฟล์ known_hosts |
| ใช้ key ผิดตัว (มีหลาย key ในเครื่อง) | agent ส่ง key ผิดตัวไปให้ GitHub ก่อน | ตั้งค่า `~/.ssh/config` ระบุ IdentityFile ให้ชัดเจน (ดู Step 176) |

---

## Step 176: จัดการ SSH key หลายตัวสำหรับหลายบัญชี GitHub ด้วยไฟล์ `~/.ssh/config`

สถานการณ์ที่พบบ่อยมากในการทำงานจริง: คุณมีบัญชี GitHub ส่วนตัว 1 บัญชี และบัญชีที่บริษัทให้ใช้อีก 1 บัญชี (หรือมากกว่านั้น) แต่ละบัญชีควรใช้ SSH key แยกกันคนละตัว เพื่อความปลอดภัยและเพื่อไม่ให้สับสนว่า commit ไหนทำจากบัญชีไหน

### 176.1 สร้าง SSH key ตัวที่สองสำหรับอีกบัญชี

```bash
ssh-keygen -t ed25519 -C "work.email@company.com" -f ~/.ssh/id_ed25519_work
```

สังเกต flag ใหม่ที่เพิ่มเข้ามา:

- `-f ~/.ssh/id_ed25519_work` — ระบุ path และชื่อไฟล์ของ key โดยตรง ไม่ต้องรอให้ระบบถามแบบ interactive อีก (สะดวกเวลาสร้างหลาย key)

ตอนนี้คุณจะมี key อยู่ในเครื่อง 2 ชุด:

```
~/.ssh/id_ed25519           ← key ส่วนตัว (ผูกกับบัญชี personal)
~/.ssh/id_ed25519.pub
~/.ssh/id_ed25519_work       ← key งาน (ผูกกับบัญชี work)
~/.ssh/id_ed25519_work.pub
```

นำ public key ของแต่ละตัวไปเพิ่มในบัญชี GitHub ที่เกี่ยวข้องตามขั้นตอนใน Step 174 (บัญชี personal เพิ่ม `id_ed25519.pub`, บัญชี work เพิ่ม `id_ed25519_work.pub`)

### 176.2 ปัญหาที่จะเกิดถ้าไม่ตั้งค่า config

ถ้าคุณมี key มากกว่า 1 ตัวในเครื่อง แล้วพยายาม `ssh -T git@github.com` โดยไม่ระบุอะไรเพิ่มเติม ssh-agent จะพยายามส่ง key ทีละตัวไปให้ GitHub ตรวจสอบตามลำดับที่ถูกเพิ่มเข้า agent ซึ่งอาจไม่ตรงกับบัญชีที่คุณต้องการใช้งานในสถานการณ์นั้น ทำให้เกิดความสับสนหรือ push ผิดบัญชีได้

### 176.3 สร้างและตั้งค่าไฟล์ `~/.ssh/config`

ไฟล์นี้เป็นไฟล์ config มาตรฐานของโปรแกรม `ssh` (ไม่ใช่แค่สำหรับ Git) ใช้กำหนดพฤติกรรมการเชื่อมต่อแยกตาม "Host alias" ที่เราตั้งขึ้นเอง

สร้าง/แก้ไขไฟล์:

```bash
nano ~/.ssh/config
```

(ใช้ editor อะไรก็ได้ เช่น `vim`, `code` — ถ้าไฟล์ยังไม่มีอยู่ระบบจะสร้างใหม่ให้)

ใส่เนื้อหาแบบนี้:

```
# บัญชีส่วนตัว (ค่าเริ่มต้นเมื่อพิมพ์ github.com ตรง ๆ)
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519
  IdentitiesOnly yes

# บัญชีที่ทำงาน (ใช้ alias ชื่อ github-work)
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work
  IdentitiesOnly yes
```

อธิบายแต่ละ directive:

- `Host` — ชื่อ alias ที่เราจะพิมพ์อ้างอิงใน URL ของ Git แทนที่จะพิมพ์ `github.com` ตรง ๆ (สามารถตั้งชื่ออะไรก็ได้ เช่น `github-work`, `gh-company` เป็นต้น)
- `HostName` — โดเมนจริงที่จะเชื่อมต่อ (ในกรณีนี้คือ `github.com` เสมอ ไม่ว่าจะกี่บัญชีก็ตาม)
- `User` — GitHub กำหนดให้ใช้ user ชื่อ `git` เสมอสำหรับการเชื่อมต่อ Git ผ่าน SSH ไม่ว่าจะเป็นบัญชีไหน
- `IdentityFile` — ระบุ path ของ private key ที่ต้องการใช้กับ Host นี้โดยเฉพาะ
- `IdentitiesOnly yes` — บังคับให้ ssh ใช้แค่ key ที่ระบุใน `IdentityFile` เท่านั้น ไม่ลองส่ง key อื่น ๆ ที่อยู่ใน agent ไปด้วย ป้องกันความสับสนเรื่องใช้ key ผิดตัว

### 176.4 วิธีใช้งาน Host alias เวลา clone/push

สำหรับบัญชี personal ใช้ URL ปกติได้เลย เพราะเราตั้ง `Host github.com` ตรง ๆ ไว้:

```bash
git clone git@github.com:personal-username/my-project.git
```

ส่วนสำหรับบัญชี work ให้แทนที่ `github.com` ด้วย alias ที่ตั้งไว้ คือ `github-work`:

```bash
git clone git@github-work:company-org/company-project.git
```

Git จะมองว่า `github-work` เป็นแค่ hostname หนึ่ง แต่ SSH จะไปค้นในไฟล์ config แล้วรู้ว่าจริง ๆ ต้องเชื่อมต่อไปที่ `github.com` โดยใช้ key `id_ed25519_work`

### 176.5 ตั้งค่า repository ที่ clone ไปแล้วให้ใช้ alias ใหม่

ถ้า repository ถูก clone มาแล้วด้วย URL แบบเดิม (`github.com` ตรง ๆ) แต่อยากเปลี่ยนให้ใช้ alias ของบัญชี work แทน สามารถแก้ remote URL ได้โดยไม่ต้อง clone ใหม่:

```bash
git remote set-url origin git@github-work:company-org/company-project.git
```

ตรวจสอบผลลัพธ์:

```bash
git remote -v
```

```
origin  git@github-work:company-org/company-project.git (fetch)
origin  git@github-work:company-org/company-project.git (push)
```

### 176.6 ทดสอบการเชื่อมต่อของแต่ละ alias แยกกัน

```bash
ssh -T git@github.com
ssh -T git@github-work
```

แต่ละคำสั่งควรตอบกลับด้วยชื่อ username ของบัญชีที่ต่างกัน เป็นการยืนยันว่าการแยก key ทำงานถูกต้อง

### 176.7 กรณี Firewall องค์กรบล็อก port 22

ถ้าอยู่ในเครือข่ายที่บล็อก port 22 (พบบ่อยในองค์กรใหญ่หรือเครือข่ายสาธารณะ) GitHub รองรับการเชื่อมต่อ SSH ผ่าน port 443 แทนได้ โดยเพิ่มการตั้งค่าใน `~/.ssh/config`:

```
Host github.com
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/id_ed25519
  IdentitiesOnly yes
```

ทดสอบด้วยคำสั่งเดิม `ssh -T git@github.com` ได้เช่นเคย ระบบจะเชื่อมต่อผ่าน port 443 แทน port 22 โดยที่ workflow การใช้งาน Git ทั้งหมดเหมือนเดิมทุกประการ

---

## Step 177: Git Credential Manager สำหรับการใช้งานผ่าน HTTPS

แม้ Part นี้จะเน้นเรื่อง SSH เป็นหลัก แต่ในการทำงานจริงบางสถานการณ์คุณอาจเลือกหรือจำเป็นต้องใช้ HTTPS แทน ซึ่งเครื่องมือที่ช่วยให้การใช้ HTTPS สะดวกพอ ๆ กับ SSH คือ **Git Credential Manager (GCM)**

### ปัญหาของ HTTPS แบบดั้งเดิม

ถ้าไม่มี credential helper ใด ๆ ทุกครั้งที่ push/pull ผ่าน HTTPS คุณจะต้องพิมพ์ username และ Personal Access Token ใหม่ทุกครั้ง ซึ่งไม่สะดวกเลย

### Git Credential Manager คืออะไร

**Git Credential Manager (GCM)** เป็นโปรแกรมช่วยจัดการ credential ที่พัฒนาโดย Microsoft (ทำงานร่วมกับ GitHub ได้ดีมาก) ทำหน้าที่:

- เก็บ credential (token) ไว้อย่างปลอดภัยใน keystore ของแต่ละระบบปฏิบัติการ (Windows Credential Manager, macOS Keychain, หรือ libsecret บน Linux)
- รองรับการ login ผ่านเบราว์เซอร์แบบ OAuth ทำให้ไม่ต้องคัดลอก-วาง token ด้วยมือเลย
- ทำงานได้กับทั้ง GitHub, Azure DevOps, Bitbucket และผู้ให้บริการ Git อื่น ๆ

### ติดตั้ง Git Credential Manager

**macOS (ผ่าน Homebrew):**

```bash
brew install --cask git-credential-manager
```

**Windows:** GCM ถูกติดตั้งมาพร้อมกับ **Git for Windows** อยู่แล้วโดยอัตโนมัติ ไม่ต้องติดตั้งเพิ่ม

**Linux (Debian/Ubuntu):**

```bash
curl -L https://github.com/git-ecosystem/git-credential-manager/releases/latest/download/gcm-linux_amd64.deb -o gcm-linux.deb
sudo dpkg -i gcm-linux.deb
git-credential-manager configure
```

### ตั้งค่าให้ Git ใช้ GCM

```bash
git config --global credential.helper manager
```

ตรวจสอบว่าตั้งค่าสำเร็จ:

```bash
git config --global credential.helper
```

ควรได้ผลลัพธ์เป็น `manager`

### ทดลองใช้งาน

```bash
git clone https://github.com/username/repository.git
```

ครั้งแรกที่เชื่อมต่อ GCM จะเปิดเบราว์เซอร์ขึ้นมาอัตโนมัติให้ login เข้าบัญชี GitHub ผ่านหน้าเว็บตามปกติ (รองรับ 2FA ในตัว) เมื่อ login สำเร็จ GCM จะเก็บ token ไว้ใน keystore ของระบบให้อัตโนมัติ ครั้งต่อ ๆ ไปจะไม่ถามซ้ำอีกจนกว่า token จะหมดอายุหรือถูกเพิกถอน

### credential.helper แบบพื้นฐานที่ไม่ใช้ GCM

ถ้าไม่อยากติดตั้งโปรแกรมเพิ่ม ระบบปฏิบัติการหลายตัวก็มี credential helper พื้นฐานติดตั้งมาให้แล้ว:

```bash
# macOS: ใช้ Keychain ในตัว
git config --global credential.helper osxkeychain

# Linux: เก็บไว้ในหน่วยความจำชั่วคราว 15 นาที (ไม่ปลอดภัยเท่า GCM แต่สะดวกกว่าไม่มีเลย)
git config --global credential.helper cache

# เก็บแบบ plain text ในไฟล์ (ไม่แนะนำสำหรับเครื่องที่ใช้ร่วมกับคนอื่น)
git config --global credential.helper store
```

### สร้าง Personal Access Token (PAT) สำหรับใช้กับ HTTPS

ถ้าไม่ใช้ GCM และต้องพิมพ์ credential ด้วยมือ ต้องสร้าง PAT จากหน้า GitHub ก่อน:

1. ไปที่ **Settings > Developer settings > Personal access tokens > Fine-grained tokens** (หรือ Tokens (classic) แล้วแต่ความต้องการ)
2. คลิก **Generate new token**
3. ตั้งชื่อ, วันหมดอายุ, และเลือกสิทธิ์ (scope) ที่จำเป็น เช่น `repo` สำหรับเข้าถึง repository
4. คัดลอก token ที่ได้ทันที (จะแสดงให้เห็นแค่ครั้งเดียว)
5. ใช้ token นี้แทน password เวลาระบบถามรหัสผ่านตอน push/pull ผ่าน HTTPS

```bash
git push https://github.com/username/repository.git
Username: your-username
Password: <วาง Personal Access Token ตรงนี้>
```

### เปรียบเทียบ: เมื่อไหร่ควรใช้ SSH กับเมื่อไหร่ควรใช้ HTTPS + GCM

| สถานการณ์ | แนะนำ |
|---|---|
| เครื่องส่วนตัวใช้งานประจำ | SSH (ตั้งค่าครั้งเดียว ใช้ได้ตลอด) |
| เครื่ององค์กรที่ IT บล็อก port 22 อย่างเข้มงวด | HTTPS + GCM (ใช้ port 443 อยู่แล้ว) |
| Container/CI ที่ต้องการความง่ายในการฝัง credential ชั่วคราว | HTTPS + PAT ที่กำหนดสิทธิ์และวันหมดอายุแคบ ๆ |
| ต้องการ login ผ่านเบราว์เซอร์แบบ SSO ขององค์กร | HTTPS + GCM (รองรับ OAuth/SSO ได้ลื่นไหลกว่า) |

---

## Step 178: เกริ่น GPG/SSH commit signing คืออะไร

### ปัญหาที่ commit signing แก้ไข

โดยปกติแล้ว Git ให้คุณตั้งชื่อผู้เขียน (author) ของ commit ได้อย่างอิสระผ่าน `git config user.name` และ `git config user.email` โดยไม่มีการตรวจสอบใด ๆ เลยว่าคนที่สร้าง commit นั้นเป็นเจ้าของอีเมลนั้นจริงหรือไม่ ซึ่งหมายความว่า **ใครก็ตามสามารถปลอมชื่อผู้เขียนใน commit ให้เป็นคนอื่นได้ง่าย ๆ**

```bash
git config user.name "Someone Else"
git config user.email "someone.else@example.com"
git commit -m "แก้ไขนี้ดูเหมือนมาจากคนอื่น"
```

นี่คือความเสี่ยงด้านความปลอดภัยที่สำคัญ โดยเฉพาะในโปรเจกต์ Open Source ขนาดใหญ่หรือองค์กรที่ต้องการความน่าเชื่อถือของประวัติ commit สูง

### Commit signing คืออะไร

**Commit signing (การเซ็นชื่อ commit)** คือการใช้กุญแจเข้ารหัส (GPG key หรือ SSH key) เพื่อ "เซ็นชื่อ" ยืนยันว่า commit นั้นถูกสร้างโดยเจ้าของกุญแจจริง ๆ ไม่ใช่ใครมาปลอมแปลง เมื่อ commit ถูกเซ็นชื่อและ push ขึ้น GitHub ระบบจะแสดงป้ายกำกับ **Verified** สีเขียวข้าง commit นั้นในหน้าเว็บ ทำให้ทุกคนที่ดูประวัติสามารถมั่นใจได้ว่า commit นี้มาจากตัวจริง

### สองวิธีหลักในการเซ็นชื่อ

1. **GPG signing** — ใช้ GPG (GNU Privacy Guard) key ซึ่งเป็นมาตรฐานการเข้ารหัสที่ใช้กันมานาน ต้องติดตั้งโปรแกรม GPG แยกต่างหาก และสร้าง GPG key คนละชุดกับ SSH key
2. **SSH signing** — ฟีเจอร์ที่ Git เพิ่มมาทีหลัง (Git 2.34+) ให้ใช้ **SSH key ตัวเดียวกับที่ใช้ push/pull** มาเซ็นชื่อ commit ได้เลย ไม่ต้องสร้าง key แยกอีกชุด สะดวกกว่ามากสำหรับคนที่ตั้งค่า SSH ไว้แล้ว

### ตัวอย่างคำสั่งคร่าว ๆ (ภาพรวมเท่านั้น)

```bash
# เปิดใช้งาน SSH signing (ตัวอย่างภาพรวม)
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true

# หลังจากนี้ทุก commit จะถูกเซ็นชื่ออัตโนมัติ
git commit -m "ข้อความ commit"
```

หลัง push ขึ้น GitHub ถ้าตั้งค่าถูกต้องทั้งฝั่งเครื่องและฝั่งบัญชี GitHub (ต้องเพิ่ม key เดียวกันนี้เป็น **Signing Key** ในหน้า SSH and GPG keys ตามที่กล่าวถึงใน Step 174) commit ที่เซ็นชื่อจะแสดงป้าย **Verified** บนหน้าเว็บ

### ทำไม Part นี้ถึงยังไม่ลงลึก

การตั้งค่า GPG/SSH signing แบบเต็มรูปแบบมีรายละเอียดปลีกย่อยพอสมควร เช่น การสร้างและจัดการ GPG key, การตั้งค่า `allowed_signers` สำหรับ SSH signing แบบ local verification, การตั้งค่าบังคับให้ทุก commit ใน repository ต้องถูกเซ็นชื่อผ่าน branch protection rules, และการแก้ปัญหาที่พบบ่อยเวลาเซ็นชื่อไม่ผ่าน เนื้อหาเหล่านี้ครบถ้วนและลงมือปฏิบัติได้จริงทุกขั้นตอนจะถูกอธิบายอย่างละเอียดใน **Part 79** ของหลักสูตรนี้ ตอนนี้ขอให้จำแค่หลักการสำคัญไว้ก่อน:

> **Commit signing คือการพิสูจน์ว่า commit นั้นมาจากเจ้าของกุญแจจริง ไม่ใช่การปลอมชื่อ ซึ่งช่วยเพิ่มความน่าเชื่อถือให้กับประวัติของโปรเจกต์**

---

## Step 179: Deploy keys คืออะไร ต่างจาก personal SSH key ใช้ตอนไหน

### ปัญหาที่ Deploy keys แก้ไข

จนถึงตอนนี้ SSH key ที่เราสร้างทั้งหมดเป็น **personal SSH key** คือ key ที่ผูกกับ**บัญชี GitHub ของคุณ**โดยตรง เมื่อเพิ่ม key นี้เข้าบัญชีแล้ว key นั้นจะมีสิทธิ์เข้าถึง **ทุก repository ที่บัญชีของคุณมีสิทธิ์เข้าถึง** ทั้งหมด

ลองนึกภาพสถานการณ์นี้: คุณมี server สำหรับ deploy โปรเจกต์ (เช่น production server หรือ CI/CD server) ที่ต้องการ clone/pull โค้ดจาก repository เดียวเท่านั้นเพื่อนำไป deploy ถ้าคุณเอา personal SSH key ของคุณไปฝากไว้ใน server นั้น หาก server ถูกแฮ็ก คนร้ายจะได้สิทธิ์เข้าถึง **ทุก repository** ในบัญชีของคุณไปด้วย ซึ่งเป็นความเสี่ยงที่ใหญ่เกินความจำเป็นมาก

### Deploy key คืออะไร

**Deploy key** คือ SSH key ที่ผูกกับ **repository เดียวเท่านั้น** ไม่ได้ผูกกับบัญชีผู้ใช้คนใดคนหนึ่ง ทำให้ขอบเขตสิทธิ์ (scope) แคบลงมาก — server ที่ถือ deploy key จะเข้าถึงได้แค่ repository นั้น repository เดียว ไม่สามารถแตะต้อง repository อื่นได้เลยแม้ว่าเจ้าของบัญชีจะมีสิทธิ์เข้าถึง repository อื่นอยู่ก็ตาม

### ตารางเปรียบเทียบ Personal SSH Key กับ Deploy Key

| ประเด็น | Personal SSH Key | Deploy Key |
|---|---|---|
| ผูกกับ | บัญชีผู้ใช้ (user account) | repository เดียวเท่านั้น |
| ขอบเขตการเข้าถึง | ทุก repository ที่บัญชีนั้นมีสิทธิ์ | เฉพาะ repository ที่เพิ่ม key นี้ไว้เท่านั้น |
| ใช้โดย | คนจริง ๆ ที่ทำงานบนเครื่องของตัวเอง | เซิร์ฟเวอร์, เครื่องมืออัตโนมัติ, CI/CD, ระบบ deploy |
| สิทธิ์อ่าน/เขียน | ตามสิทธิ์ของบัญชีในแต่ละ repo | เลือกได้ตอนเพิ่ม key ว่าจะให้ "Allow write access" หรือแค่อ่านอย่างเดียว |
| จุดที่เพิ่ม key | Settings ระดับบัญชี (Account Settings) | Settings ระดับ repository นั้น ๆ โดยตรง |
| ตัวอย่างการใช้งาน | นักพัฒนา push/pull โค้ดจากเครื่อง laptop ประจำวัน | Production server ดึงโค้ดไป deploy, ระบบ CI ดึงโค้ดไปรัน build/test |

### วิธีเพิ่ม Deploy Key ให้กับ repository

1. เข้าไปที่หน้า repository บน GitHub
2. คลิก **Settings** (ของ repository นั้น ไม่ใช่ของบัญชี)
3. เมนูด้านซ้ายเลือก **Deploy keys**
4. คลิก **Add deploy key**
5. กรอก **Title** (เช่น `Production Server` หรือ `CI Build Runner`)
6. วาง public key ของเครื่อง server นั้น (สร้างด้วย `ssh-keygen` เหมือนที่เราทำใน Step 172 แต่สร้างแยกต่างหากบนเครื่อง server ไม่ใช่นำ personal key มาใช้ซ้ำ)
7. เลือก checkbox **Allow write access** ถ้าต้องการให้ server นั้น push โค้ดกลับมาได้ด้วย (ถ้าเป็นแค่ deploy server ที่ดึงโค้ดไปรันเฉย ๆ ไม่จำเป็นต้องติ๊กช่องนี้ — ควรให้สิทธิ์แค่อ่านอย่างเดียวเสมอถ้าไม่จำเป็นต้องเขียน)
8. คลิก **Add key**

### ตัวอย่างการสร้าง SSH key สำหรับใช้เป็น Deploy key บนเครื่อง server

```bash
# รันบนเครื่อง server (ไม่ใช่เครื่อง laptop ส่วนตัว)
ssh-keygen -t ed25519 -C "deploy-key-production-server" -f ~/.ssh/id_ed25519_deploy -N ""
```

สังเกต flag `-N ""` ซึ่งกำหนด passphrase เป็นค่าว่าง — เหตุผลคือ deploy key มักถูกใช้งานโดยกระบวนการอัตโนมัติที่ไม่มีคนนั่งพิมพ์ passphrase ตอนรัน (เช่น cron job หรือ CI pipeline) จึงมักไม่ตั้ง passphrase ให้กับ deploy key ประเภทนี้ แต่ต้องแลกมาด้วยการดูแลเรื่อง permission ของไฟล์ private key ให้เข้มงวดที่สุดแทน:

```bash
chmod 600 ~/.ssh/id_ed25519_deploy
```

จากนั้นนำ public key ไปเพิ่มในหน้า Deploy keys ตามขั้นตอนด้านบน:

```bash
cat ~/.ssh/id_ed25519_deploy.pub
```

### ข้อจำกัดสำคัญของ Deploy key ที่ควรรู้

- Deploy key หนึ่งตัว ผูกได้กับ **repository เดียวเท่านั้น** ถ้า server ต้องการเข้าถึงหลาย repository ต้องสร้าง deploy key แยกกันสำหรับแต่ละ repo หรือพิจารณาใช้ **GitHub App** / **machine user** แทนถ้าจำนวน repository เยอะมาก
- Deploy key ตัวเดียวกัน **ไม่สามารถนำไปเพิ่มซ้ำในหลาย repository ได้** ถ้าพยายามเพิ่ม public key เดียวกันในอีก repo หนึ่ง GitHub จะแจ้ง error ว่า key นี้ถูกใช้ไปแล้ว
- เหมาะสำหรับกรณี "เครื่อง/ระบบหนึ่งตัว เข้าถึง repository เดียว" ถ้าสถานการณ์ซับซ้อนกว่านั้น (หลายระบบ หลาย repo) ควรพิจารณาโซลูชันระดับองค์กร เช่น GitHub Apps ซึ่งจะกล่าวถึงในภายหลังของหลักสูตร

---

## Step 180: แบบฝึกหัด — ตั้งค่า SSH เต็มรูปแบบตั้งแต่สร้าง key จนถึง clone/push repo สำเร็จ

ถึงเวลาลงมือทำจริงทุกขั้นตอนตั้งแต่ต้นจนจบ แบบฝึกหัดนี้จะพาคุณผ่านกระบวนการทั้งหมดของ Part นี้แบบต่อเนื่อง

### ขั้นที่ 1: ตรวจสอบ SSH key ที่มีอยู่แล้ว (ถ้ามี)

```bash
ls -la ~/.ssh/
```

ถ้ามี `id_ed25519` และ `id_ed25519.pub` อยู่แล้วจาก Step 172 สามารถใช้ตัวเดิมต่อได้เลย ถ้ายังไม่มีให้ทำขั้นที่ 2 ต่อ

### ขั้นที่ 2: สร้าง SSH key ใหม่ (ถ้ายังไม่มี)

```bash
ssh-keygen -t ed25519 -C "phutjirakul.iam@gmail.com"
```

กด Enter เพื่อใช้ path เริ่มต้น แล้วตั้ง passphrase ที่จำได้และปลอดภัย

### ขั้นที่ 3: เริ่ม ssh-agent และเพิ่ม key เข้าไป

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

ตรวจสอบว่า key ถูกเพิ่มสำเร็จ:

```bash
ssh-add -l
```

### ขั้นที่ 4: คัดลอก public key

```bash
cat ~/.ssh/id_ed25519.pub
```

คัดลอกข้อความทั้งบรรทัดที่แสดงออกมา

### ขั้นที่ 5: เพิ่ม public key เข้า GitHub

1. เปิดเว็บ GitHub > Settings > SSH and GPG keys > New SSH key
2. ตั้งชื่อ Title เช่น `แบบฝึกหัด Part 18`
3. เลือก Key type เป็น **Authentication Key**
4. วาง public key ที่คัดลอกมา
5. กด Add SSH key

### ขั้นที่ 6: ทดสอบการเชื่อมต่อ

```bash
ssh -T git@github.com
```

ควรได้ผลลัพธ์ทำนอง:

```
Hi phutjirakul-iam! You've successfully authenticated, but GitHub does not provide shell access.
```

ถ้ายังไม่สำเร็จ ย้อนกลับไปดูตารางแก้ปัญหาใน Step 175 ก่อนไปขั้นตอนถัดไป

### ขั้นที่ 7: สร้าง repository ทดสอบบน GitHub

1. เข้าเว็บ GitHub แล้วกด **New repository**
2. ตั้งชื่อ เช่น `ssh-practice-part18`
3. เลือก **Private** หรือ **Public** ก็ได้
4. ติ๊ก **Add a README file** เพื่อให้ repo มีเนื้อหาเริ่มต้น
5. กด **Create repository**

### ขั้นที่ 8: Clone repository ด้วย SSH URL

คัดลอก SSH URL จากหน้า repo (กดปุ่ม Code > เลือกแท็บ SSH) แล้วรัน:

```bash
cd ~/git-course
git clone git@github.com:phutjirakul-iam/ssh-practice-part18.git
cd ssh-practice-part18
```

ถ้า clone สำเร็จโดยไม่มีการถาม username/password เลย แปลว่าการเชื่อมต่อ SSH ทำงานถูกต้องสมบูรณ์แล้ว

### ขั้นที่ 9: แก้ไขไฟล์ สร้าง commit และ push กลับขึ้นไป

```bash
echo "ทดสอบการ push ผ่าน SSH สำเร็จ" >> README.md
git add README.md
git commit -m "ทดสอบการเชื่อมต่อและ push ผ่าน SSH"
git push origin main
```

ถ้า push สำเร็จโดยไม่มีการถาม credential ใด ๆ เลย (นอกจาก passphrase ที่อาจถูกถามครั้งแรกถ้า ssh-agent ยังไม่ได้ add key ไว้) แสดงว่าคุณตั้งค่า SSH สำเร็จแบบเต็มรูปแบบแล้ว

### ขั้นที่ 10: ตรวจสอบผลลัพธ์บนเว็บ GitHub

เปิดหน้า repository บนเว็บ แล้วรีเฟรชดู ควรเห็น commit ใหม่ที่คุณเพิ่งสร้าง พร้อมข้อความ commit ที่พิมพ์ไว้ และไฟล์ README.md ที่มีเนื้อหาที่เพิ่มเข้าไป

### ขั้นตอนเสริม (ถ้าต้องการฝึกเรื่องหลายบัญชี)

ถ้าคุณมีบัญชี GitHub ที่สอง ลองทำตาม Step 176 ทั้งหมดอีกครั้งด้วยบัญชีที่สอง เพื่อฝึกการตั้งค่า `~/.ssh/config` แบบ Host alias จริง แล้วลองสลับ clone repository จากทั้งสองบัญชีในเครื่องเดียวกันดูว่าทำงานถูกต้องหรือไม่

```bash
ssh -T git@github.com
ssh -T git@github-work
```

ทั้งสองคำสั่งควรตอบกลับด้วย username ของคนละบัญชีกัน

### Checklist ของแบบฝึกหัด Step 180

- [ ] มี SSH key pair (private + public) อยู่ในเครื่อง
- [ ] ssh-agent ทำงานอยู่ และมี key ถูกเพิ่มเข้าไปแล้ว (`ssh-add -l` แสดงผล)
- [ ] Public key ถูกเพิ่มเข้าบัญชี GitHub สำเร็จแล้ว
- [ ] `ssh -T git@github.com` ตอบกลับด้วยชื่อ username ของตัวเอง
- [ ] Clone repository ด้วย SSH URL สำเร็จโดยไม่ต้องกรอก username/password
- [ ] แก้ไขไฟล์ commit และ push กลับขึ้น GitHub สำเร็จผ่าน SSH
- [ ] เห็นการเปลี่ยนแปลงปรากฏบนหน้าเว็บ GitHub จริง

---

## สรุป Part 18

ใน Part นี้เราได้เรียนรู้ว่า:

1. **HTTPS กับ SSH** เป็นสองวิธีหลักในการเชื่อมต่อกับ GitHub — HTTPS ตั้งค่าง่ายกว่าแต่ต้องพึ่ง Personal Access Token หรือ credential helper ส่วน SSH ตั้งค่าครั้งเดียวแล้วสะดวกในระยะยาว เหมาะกับการใช้งานประจำ
2. **SSH key pair** ประกอบด้วย private key (เก็บไว้ในเครื่อง ห้ามแชร์) และ public key (แชร์ได้ นำไปฝากที่ GitHub) สร้างได้ด้วยคำสั่ง `ssh-keygen -t ed25519 -C "email"` โดย Ed25519 คืออัลกอริทึมที่แนะนำที่สุดในปัจจุบัน
3. **ssh-agent** ช่วยจำ private key ที่ปลดล็อกแล้วไว้ในหน่วยความจำ ทำให้ไม่ต้องพิมพ์ passphrase ซ้ำทุกครั้งที่เชื่อมต่อ ผ่านคำสั่ง `eval "$(ssh-agent -s)"` และ `ssh-add`
4. การเพิ่ม public key เข้าบัญชี GitHub ทำผ่านหน้า **Settings > SSH and GPG keys > New SSH key**
5. ทดสอบการเชื่อมต่อได้ด้วยคำสั่ง `ssh -T git@github.com` ซึ่งจะไม่เปิด shell จริง แต่ใช้ตรวจสอบว่าการยืนยันตัวตนสำเร็จหรือไม่
6. การจัดการหลายบัญชี GitHub พร้อมกันในเครื่องเดียวทำได้ด้วยไฟล์ `~/.ssh/config` โดยตั้ง Host alias แยกกันแต่ละบัญชี พร้อมระบุ `IdentityFile` และ `IdentitiesOnly yes` เพื่อป้องกันความสับสนเรื่องใช้ key ผิดตัว
7. **Git Credential Manager (GCM)** ช่วยให้การใช้งาน HTTPS สะดวกพอ ๆ กับ SSH โดยจัดการ token ให้อัตโนมัติผ่าน keystore ของระบบปฏิบัติการ
8. **Commit signing** (GPG หรือ SSH signing) คือกลไกพิสูจน์ว่า commit มาจากเจ้าของกุญแจจริง ป้องกันการปลอมชื่อผู้เขียน commit ซึ่งจะเจาะลึกแบบเต็มรูปแบบใน Part 79
9. **Deploy keys** คือ SSH key ที่ผูกกับ repository เดียวแทนที่จะผูกกับบัญชีผู้ใช้ ใช้สำหรับเซิร์ฟเวอร์หรือระบบอัตโนมัติที่ต้องการสิทธิ์เข้าถึงแคบและปลอดภัยกว่าการใช้ personal SSH key
10. ผ่านแบบฝึกหัดสุดท้าย คุณได้ลงมือตั้งค่า SSH ตั้งแต่สร้าง key จนถึง clone และ push repository จริงสำเร็จครบทุกขั้นตอนด้วยตัวเอง

**ต่อไป:** [Part 19: README.md ที่ดีและ Markdown สำหรับ GitHub](./part-019-readme-markdown.md)
