# Part 47: ติดตั้งและตั้งค่า GitLab (Cloud/Self-hosted เบื้องต้น)

> **Step ในหลักสูตรนี้:** Step 461–470
> **เฟส:** 5 — GitLab
> **เป้าหมายของ Part นี้:** สมัครและตั้งค่าบัญชี GitLab.com (SaaS) ให้พร้อมใช้งานจริงตั้งแต่ profile, 2FA, SSH key ไปจนถึง Personal Access Token และ GitLab CLI (`glab`) พร้อมทั้งเข้าใจภาพรวมว่า GitLab Self-Managed ติดตั้งอย่างไรทั้งแบบ Docker และแบบ Omnibus package โดยไม่ลงรายละเอียดเชิง infrastructure ลึกเกินไป เพื่อให้คุณแยกแยะได้ว่าเมื่อไหร่ควรใช้ SaaS เมื่อไหร่ควรพิจารณา self-hosted และปิดท้ายด้วยความคุ้นเคยเบื้องต้นกับ GitLab UI และฟีเจอร์ AI อย่าง GitLab Duo ก่อนจะไปสร้าง repository และ Merge Request จริงใน Part ถัดไป

---

## สารบัญของ Part นี้

- Step 461: สมัครบัญชี GitLab.com (SaaS) ทีละขั้นตอน
- Step 462: ภาพรวมการติดตั้ง GitLab Self-Managed ด้วย Docker
- Step 463: GitLab Omnibus package คืออะไร
- Step 464: ตั้งค่าบัญชี GitLab.com ให้พร้อมใช้งาน (Profile, 2FA)
- Step 465: SSH key สำหรับ GitLab
- Step 466: Personal Access Token ของ GitLab
- Step 467: GitLab CLI (`glab`) เบื้องต้น
- Step 468: การนำทาง GitLab UI ภาพรวม — ต่างจาก GitHub ตรงไหนบ้าง
- Step 469: แนะนำ GitLab Duo / AI features เบื้องต้น
- Step 470: แบบฝึกหัด — ตั้งค่าบัญชี GitLab เต็มรูปแบบพร้อมเชื่อมต่อผ่าน SSH สำเร็จ

---

## Step 461: สมัครบัญชี GitLab.com (SaaS) ทีละขั้นตอน

ใน Part 46 คุณได้เรียนรู้ภาพรวมแล้วว่า GitLab คืออะไรและต่างจาก GitHub อย่างไร ใน Part นี้เราจะลงมือจริง เริ่มจากการสมัครบัญชีบน **GitLab.com** ซึ่งเป็นบริการ **SaaS (Software as a Service)** ที่ GitLab Inc. เป็นผู้ดูแล server ให้ทั้งหมด คุณไม่ต้องติดตั้งหรือดูแล infrastructure ใด ๆ เอง เหมาะที่สุดสำหรับผู้เริ่มต้นและสำหรับหลักสูตรนี้ตลอดทั้งเฟส 5

### 461.1 ทางเลือกในการสมัคร

เมื่อเข้าไปที่ https://gitlab.com/users/sign_up คุณจะเห็นทางเลือกหลายแบบ:

1. **สมัครด้วยอีเมลโดยตรง** — กรอกอีเมล ตั้งชื่อผู้ใช้ (username) และรหัสผ่านเอง
2. **สมัครผ่านบัญชี Google**
3. **สมัครผ่านบัญชี GitHub** — สะดวกมากถ้าคุณมีบัญชี GitHub อยู่แล้วจากเฟสก่อนหน้าของหลักสูตรนี้
4. **สมัครผ่านบัญชี Salesforce**

การสมัครผ่านบัญชีที่มีอยู่แล้ว (Google/GitHub) จะเร็วกว่าและไม่ต้องจำรหัสผ่านเพิ่ม แต่ในหลักสูตรนี้เราจะแนะนำวิธี **สมัครด้วยอีเมลโดยตรง** เพื่อให้คุณเห็นขั้นตอนแบบเต็มรูปแบบและควบคุมบัญชีได้อิสระที่สุด

### 461.2 ขั้นตอนสมัครด้วยอีเมล

1. เปิดเบราว์เซอร์ไปที่ https://gitlab.com/users/sign_up
2. เลือกแท็บ **Register** (ไม่ใช่ Sign in)
3. กรอกข้อมูล:
   - **First name / Last name** — ชื่อจริง/นามสกุลของคุณ
   - **Username** — ชื่อผู้ใช้ที่จะปรากฏใน URL ของ GitLab เช่น `gitlab.com/yourusername` (ต้องไม่ซ้ำกับคนอื่น ตัวอักษร ตัวเลข และขีดกลางเท่านั้น)
   - **Email** — ใช้อีเมลจริงที่เข้าถึงได้ เพราะต้องใช้ยืนยันตัวตน
   - **Password** — ควรมีความยาวอย่างน้อย 8 ตัวอักษร ผสมตัวพิมพ์เล็ก-ใหญ่ ตัวเลข และสัญลักษณ์
4. ติ๊กยอมรับ **Terms of Service and Privacy Policy**
5. กดปุ่ม **Continue**

### 461.3 ยืนยันตัวตนด้วย CAPTCHA และอีเมล

GitLab จะแสดง CAPTCHA เพื่อยืนยันว่าคุณไม่ใช่บอท จากนั้นระบบจะส่งอีเมลยืนยันไปยังกล่องจดหมายที่คุณกรอกไว้ พร้อมรหัสยืนยัน 6 หลัก (verification code)

```
Subject: Confirmation instructions
กรอกรหัส 6 หลักในหน้าเว็บที่เปิดค้างไว้เพื่อยืนยันอีเมล
```

เปิดอีเมล คัดลอกรหัส แล้ววางลงในหน้าที่ GitLab เปิดรออยู่ กดยืนยัน

### 461.4 ตอบแบบสอบถามเริ่มต้น (Onboarding questions)

หลังยืนยันอีเมลสำเร็จ GitLab มักถามคำถามสั้น ๆ เพื่อปรับแต่งประสบการณ์เริ่มต้น เช่น:

- คุณจะใช้ GitLab ทำอะไร (เรียนรู้ส่วนตัว, งานบริษัท, โปรเจกต์โอเพนซอร์ส)
- บทบาทของคุณ (Developer, Student, DevOps Engineer ฯลฯ)
- ขนาดของทีม/องค์กร

คำถามเหล่านี้ **ไม่มีผลต่อฟีเจอร์ที่คุณใช้ได้** เป็นเพียงข้อมูลสถิติที่ GitLab เก็บไว้ปรับปรุงผลิตภัณฑ์ ตอบตามความเป็นจริงหรือเลือก "Skip" ก็ได้ถ้ามีตัวเลือกนั้น

### 461.5 สร้าง Namespace แรก (Personal namespace)

หลังสมัครเสร็จ GitLab จะพาไปหน้าสร้างโปรเจกต์แรก แต่ในขั้นตอนนี้เราจะยังไม่สร้างโปรเจกต์ (จะทำใน Part 48) ให้เลือก **"Skip this for now"** หรือกดปิดหน้าต่างแนะนำไปก่อน เพื่อไปตั้งค่าบัญชีให้เรียบร้อยก่อนใน Step ถัดไป

### 461.6 ข้อควรรู้เรื่อง Free Tier ของ GitLab.com

บัญชี GitLab.com ฟรีจะได้รับสิทธิ์:

| ฟีเจอร์ | โควตาระดับ Free |
|---|---|
| Private/Public repository | ไม่จำกัดจำนวน repository |
| จำนวนสมาชิกต่อ namespace ฟรี | สูงสุด 5 คนต่อ top-level group |
| CI/CD pipeline minutes | มีโควตาต่อเดือนให้ใช้ฟรี (ตัวเลขที่ GitLab กำหนดเปลี่ยนแปลงได้ตามนโยบาย ควรตรวจสอบหน้า pricing ปัจจุบันเสมอ) |
| พื้นที่เก็บข้อมูล (storage) | มีโควตาต่อ namespace ให้ใช้ฟรีในระดับที่เพียงพอสำหรับโปรเจกต์เรียนรู้และโปรเจกต์ขนาดเล็ก |
| Issue, Merge Request, Wiki | ใช้งานได้เต็มรูปแบบไม่จำกัด |

สำหรับหลักสูตรนี้ **บัญชี Free เพียงพอสำหรับทุก Step ที่จะเรียน** ไม่จำเป็นต้องอัปเกรดเป็น Premium หรือ Ultimate เลย

---

## Step 462: ภาพรวมการติดตั้ง GitLab Self-Managed ด้วย Docker

Step นี้เป็น Step แนวคิดระดับสูง (conceptual overview) — เราจะไม่ลงมือติดตั้งจริงในหลักสูตรระดับนี้ เพราะการดูแล GitLab Self-Managed แบบเต็มรูปแบบต้องใช้ทรัพยากร server ที่มากพอสมควร (CPU, RAM, disk) และมีรายละเอียดเชิง infrastructure ที่ควรเรียนแยกเป็นหลักสูตร DevOps/SysAdmin โดยเฉพาะ แต่การเข้าใจภาพรวมไว้ก่อนจะมีประโยชน์มากเมื่อคุณต้องทำงานในองค์กรที่ self-host GitLab เอง

### 462.1 GitLab Self-Managed คืออะไร

**GitLab Self-Managed** (บางเอกสารเก่าเรียกว่า GitLab Community Edition/Enterprise Edition แบบติดตั้งเอง) คือการนำซอฟต์แวร์ GitLab ทั้งชุด — เว็บแอป, ฐานข้อมูล, ระบบ CI/CD, Container Registry ฯลฯ — มาติดตั้งรันบน server ของคุณเองหรือขององค์กรเอง แทนที่จะใช้ server ของ GitLab Inc. (แบบ GitLab.com)

องค์กรเลือก self-host ด้วยเหตุผลหลัก ๆ เช่น:

- **Data sovereignty** — ข้อมูลโค้ดต้องอยู่ในประเทศหรือใน network ขององค์กรเท่านั้น (พบมากในธนาคาร หน่วยงานรัฐ)
- **Compliance** — ข้อกำหนดด้านกฎหมายหรือมาตรฐานความปลอดภัยเฉพาะอุตสาหกรรม
- **Customization ระดับลึก** — ต้องการปรับแต่ง configuration ที่ SaaS ไม่เปิดให้ทำ
- **ควบคุมต้นทุนระยะยาว** — สำหรับองค์กรขนาดใหญ่ที่มีผู้ใช้จำนวนมาก การ self-host อาจคุ้มค่ากว่าจ่ายค่า subscription รายหัว

### 462.2 แนวคิดการรัน GitLab ด้วย Docker

วิธีที่ง่ายและเร็วที่สุดสำหรับการ**ทดลอง** GitLab Self-Managed (ไม่ใช่สำหรับใช้งาน production จริงจัง) คือการรันผ่าน **Docker container** ด้วยคำสั่งประมาณนี้:

```bash
docker run --detach \
  --hostname gitlab.example.local \
  --publish 443:443 --publish 80:80 --publish 2222:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  --shm-size 256m \
  gitlab/gitlab-ce:latest
```

อธิบายแนวคิดของแต่ละส่วนแบบภาพรวม (ไม่ลงลึกด้าน Docker networking):

- `gitlab/gitlab-ce:latest` — image ทางการของ GitLab Community Edition ที่ GitLab เผยแพร่บน Docker Hub มี GitLab ทั้งชุดอัดรวมอยู่ใน image เดียว (แนวคิดเดียวกับ Omnibus ที่จะพูดถึงใน Step 463)
- `--publish` — เปิด port ของ container ออกมาให้เข้าถึงจากนอกเครื่องได้ (80/443 สำหรับเว็บ, 2222 สำหรับ SSH เพื่อไม่ให้ชนกับ SSH ของเครื่อง host เอง)
- `--volume` — ผูกโฟลเดอร์บนเครื่อง host เข้ากับโฟลเดอร์ใน container เพื่อให้ **ข้อมูล config, logs และข้อมูลโปรเจกต์ทั้งหมดไม่หายไปเมื่อ container ถูกลบหรือรีสตาร์ท**
- `--shm-size` — กำหนดขนาดหน่วยความจำที่ใช้ร่วมกัน (shared memory) ซึ่ง GitLab ต้องการมากกว่าค่าเริ่มต้นของ Docker เนื่องจากมีหลายบริการภายในทำงานพร้อมกัน

### 462.3 ข้อควรรู้ก่อนตัดสินใจใช้ Docker สำหรับ GitLab

| ประเด็น | รายละเอียด |
|---|---|
| ทรัพยากรขั้นต่ำ | GitLab แนะนำ RAM อย่างน้อย 4 GB (ในทางปฏิบัติ 8 GB ขึ้นไปจะลื่นไหลกว่ามาก) และ CPU อย่างน้อย 4 core สำหรับการใช้งานที่มีผู้ใช้มากกว่าหยิบมือ |
| เวลาที่ container พร้อมใช้งาน | หลังรันคำสั่ง `docker run` ครั้งแรก ระบบภายในต้องตั้งค่าฐานข้อมูลและบริการต่าง ๆ เอง อาจใช้เวลา 5–10 นาทีกว่าจะเข้าเว็บได้ |
| การอัปเดตเวอร์ชัน | ทำโดย pull image เวอร์ชันใหม่แล้วสร้าง container ใหม่ทับของเดิม (ข้อมูลยังอยู่เพราะผูกกับ volume ภายนอก) |
| เหมาะกับ | การทดลองเรียนรู้, demo, dev environment ขนาดเล็ก |
| ไม่เหมาะกับ | Production ระดับองค์กรที่มีผู้ใช้จำนวนมาก ซึ่งควรพิจารณา Omnibus package บน server จริง หรือ Kubernetes (Helm chart) แทน เพื่อให้ scale และจัดการ backup ได้ดีกว่า |

### 462.4 ทำไม Part นี้ไม่ให้ลงมือติดตั้งจริง

การติดตั้ง GitLab Self-Managed แบบใช้งานจริงต้องคำนึงถึงเรื่อง reverse proxy, TLS certificate, backup strategy, monitoring, และการอัปเกรดเวอร์ชันอย่างปลอดภัย ซึ่งเป็นเนื้อหาระดับ DevOps/Infrastructure ที่ลึกเกินขอบเขตของหลักสูตรสาย Git/GitLab พื้นฐานนี้ หลักสูตรนี้จะใช้ **GitLab.com (SaaS)** เป็นสภาพแวดล้อมหลักตลอดทั้งเฟส 5 เพื่อให้คุณโฟกัสที่การเรียนรู้ฟีเจอร์และ workflow ของ GitLab ได้เต็มที่โดยไม่ต้องเสียเวลาดูแล server เอง

> **สรุปสั้น ๆ:** ถ้าคุณแค่อยากเรียนรู้และฝึกใช้งาน ให้ใช้ GitLab.com ถ้าองค์กรของคุณจำเป็นต้อง self-host จริง ให้นำความรู้จาก Step นี้ไปต่อยอดศึกษาเอกสารการติดตั้งอย่างเป็นทางการของ GitLab (`docs.gitlab.com/install`) อย่างละเอียดอีกครั้ง

---

## Step 463: GitLab Omnibus package คืออะไร

### 463.1 ปัญหาที่ Omnibus แก้ไข

ก่อนที่ Omnibus package จะถือกำเนิด การติดตั้ง GitLab บน Linux server ต้องติดตั้งส่วนประกอบทีละตัวด้วยมือ ได้แก่ Ruby, Ruby on Rails, PostgreSQL, Redis, Nginx, Sidekiq (background job processor) และอื่น ๆ อีกจำนวนมาก แต่ละตัวต้องตั้งค่าให้เข้ากันได้พอดี ซึ่งซับซ้อนและเสี่ยงต่อความผิดพลาดสูงมาก

### 463.2 Omnibus package คืออะไร

**GitLab Omnibus package** คือแพ็กเกจติดตั้งแบบ **"all-in-one"** ที่รวมทุกส่วนประกอบที่ GitLab ต้องการไว้ในแพ็กเกจเดียว (`.deb` สำหรับ Debian/Ubuntu หรือ `.rpm` สำหรับ RHEL/CentOS/Fedora) ผู้ดูแลระบบเพียงติดตั้งแพ็กเกจเดียว แล้ว Omnibus จะจัดการติดตั้งและตั้งค่าส่วนประกอบทั้งหมดให้โดยอัตโนมัติ พร้อมไฟล์ config หลักเพียงไฟล์เดียวคือ `/etc/gitlab/gitlab.rb`

```
gitlab-ce (หรือ gitlab-ee) Omnibus package
├── Ruby + Ruby on Rails (ตัวแอปหลักของ GitLab)
├── PostgreSQL (ฐานข้อมูล)
├── Redis (cache และ job queue)
├── Nginx (web server/reverse proxy หน้าเว็บ)
├── Sidekiq (ประมวลผลงานเบื้องหลัง เช่นส่งอีเมล)
├── GitLab Shell (จัดการการเชื่อมต่อ SSH เข้าถึง Git repository)
├── GitLab Workhorse (reverse proxy เฉพาะทางสำหรับ request ขนาดใหญ่ เช่น git push/pull, upload ไฟล์)
└── ส่วนประกอบอื่น ๆ อีกจำนวนมาก
```

### 463.3 ภาพรวมขั้นตอนติดตั้งแบบ Omnibus (แนวคิดเท่านั้น)

```bash
# ตัวอย่างสำหรับ Ubuntu/Debian (แนวคิดระดับสูง ไม่ใช่คำแนะนำให้ลงมือทำใน Part นี้)
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo EXTERNAL_URL="https://gitlab.example.com" apt-get install gitlab-ce
```

หลังติดตั้งเสร็จ ผู้ดูแลระบบแก้ไข config หลักได้ที่ไฟล์เดียว:

```bash
sudo nano /etc/gitlab/gitlab.rb
# แก้ค่าต่าง ๆ เช่น external_url, การตั้งค่า SMTP สำหรับส่งอีเมล ฯลฯ
sudo gitlab-ctl reconfigure   # สั่งให้ GitLab อ่านค่าใหม่และปรับ service ทั้งหมดให้ตรงกัน
```

คำสั่ง `gitlab-ctl` คือเครื่องมือควบคุมส่วนกลางของ Omnibus ใช้สั่ง start/stop/restart บริการทั้งหมด ตรวจสอบสถานะ และสั่ง reconfigure ได้จากคำสั่งเดียว

### 463.4 เปรียบเทียบ Docker vs Omnibus vs Helm Chart (Kubernetes)

| วิธีติดตั้ง | เหมาะกับ | ความซับซ้อน |
|---|---|---|
| **Docker (`docker run`)** | ทดลอง เรียนรู้ demo ระยะสั้น | ต่ำที่สุด ตั้งค่าเร็ว |
| **Omnibus package** | Production บน server จริง (VM หรือ bare-metal) ขนาดเล็กถึงกลาง | ปานกลาง จัดการง่ายกว่าติดตั้งเองทีละส่วนมาก |
| **Helm Chart บน Kubernetes** | องค์กรขนาดใหญ่ที่ต้องการ scale แต่ละส่วนประกอบแยกกัน (เช่น scale Sidekiq แยกจากเว็บแอป) และมีทีม Kubernetes อยู่แล้ว | สูงที่สุด ต้องมีความรู้ Kubernetes ที่ดี |

### 463.5 สรุป

Omnibus คือคำตอบที่ทำให้การ self-host GitLab เป็นไปได้จริงในทางปฏิบัติสำหรับองค์กรทั่วไปที่ไม่มีทีม Infrastructure ขนาดใหญ่ — แทนที่จะต้องตั้งค่าส่วนประกอบนับสิบตัวเอง ก็ใช้แพ็กเกจเดียวและไฟล์ config เดียว (`gitlab.rb`) ควบคุมทุกอย่าง เนื้อหาการดูแล GitLab Self-Managed แบบละเอียด (backup, upgrade, HA) ไม่ได้อยู่ในขอบเขตหลักสูตรนี้ แต่การเข้าใจภาพนี้ไว้จะช่วยให้คุณสื่อสารกับทีม Infrastructure ในที่ทำงานได้อย่างเข้าใจตรงกัน

---

## Step 464: ตั้งค่าบัญชี GitLab.com ให้พร้อมใช้งาน (Profile, 2FA)

กลับมาที่บัญชี GitLab.com ที่สมัครไว้ใน Step 461 ตอนนี้เรามาตั้งค่าให้พร้อมใช้งานจริงกัน

### 464.1 ตั้งค่า Profile

1. คลิกรูป avatar มุมขวาบน แล้วเลือก **Edit profile** (หรือไปที่ `https://gitlab.com/-/profile`)
2. ตั้งค่าข้อมูลพื้นฐาน:
   - **Full name** — ชื่อเต็มที่จะแสดงในหน้าโปรไฟล์และ commit ที่ทำผ่านเว็บ
   - **Public email** — อีเมลที่จะแสดงต่อสาธารณะ (แนะนำให้ใช้อีเมลเดียวกับที่ตั้งค่าใน Git config หรือใช้อีเมลแบบ noreply ที่ GitLab เตรียมไว้ให้เพื่อความเป็นส่วนตัว)
   - **Bio, Location, Organization** — ข้อมูลเสริม ไม่บังคับ
   - **Avatar** — อัปโหลดรูปโปรไฟล์
3. กด **Update profile settings** เพื่อบันทึก

### 464.2 ตั้งค่า Two-Factor Authentication (2FA)

**2FA** คือการยืนยันตัวตนสองชั้น นอกจากรหัสผ่านแล้วต้องมีรหัสชั่วคราวจากอุปกรณ์ที่คุณถืออยู่ด้วย ช่วยป้องกันบัญชีอย่างมากแม้รหัสผ่านจะรั่วไหล **แนะนำอย่างยิ่งให้เปิดใช้งานเสมอ** โดยเฉพาะถ้าบัญชีนี้จะเชื่อมกับโปรเจกต์งานจริง

ขั้นตอนเปิดใช้งาน:

1. ไปที่ **Avatar > Edit profile > Account** (หรือ URL `https://gitlab.com/-/profile/account`)
2. ในหัวข้อ **Two-Factor Authentication** คลิก **Enable two-factor authentication**
3. GitLab จะแสดง QR code บนหน้าจอ
4. เปิดแอป Authenticator บนมือถือ เช่น **Google Authenticator**, **Microsoft Authenticator**, **Authy**, หรือ **1Password** แล้วสแกน QR code นั้น
5. แอปจะสร้างรหัส 6 หลักที่เปลี่ยนทุก 30 วินาที นำรหัสปัจจุบันมากรอกในช่อง **Pin code** บนหน้าเว็บ
6. กด **Register with two-factor app**

### 464.3 บันทึก Recovery codes ให้ปลอดภัย

หลังเปิด 2FA สำเร็จ GitLab จะแสดง **Recovery codes** ชุดหนึ่ง (รหัสสำรองใช้ครั้งเดียวต่อรหัส) สำหรับกรณีที่คุณทำมือถือหายหรือเข้าแอป Authenticator ไม่ได้

```
Recovery codes ตัวอย่าง (ของจริงจะเป็นชุดตัวอักษร/ตัวเลขสุ่ม):
a1b2c3d4
e5f6g7h8
i9j0k1l2
...
```

**สิ่งที่ต้องทำทันที:**

- ดาวน์โหลดหรือคัดลอก recovery codes เก็บไว้ในที่ปลอดภัย เช่น password manager
- **ห้ามเก็บไว้ในที่เดียวกับอุปกรณ์ที่ใช้ authenticator** เพราะถ้าอุปกรณ์นั้นหายไปพร้อมกัน คุณจะเข้าบัญชีไม่ได้เลย
- แต่ละรหัสใช้ได้เพียงครั้งเดียว ถ้าใช้ไปหมดแล้วต้องสร้างชุดใหม่จากหน้า Account settings

### 464.4 ตรวจสอบ Active sessions และ Security log

ในหน้า **Account settings** เดียวกัน ยังมีส่วน **Active sessions** ที่แสดงอุปกรณ์/เบราว์เซอร์ที่ล็อกอินเข้าบัญชีอยู่ปัจจุบัน หากพบอุปกรณ์แปลกปลอมที่ไม่ใช่ของคุณ สามารถกด **Revoke** เพื่อบังคับ logout อุปกรณ์นั้นได้ทันที ควรตรวจสอบหน้านี้เป็นระยะเพื่อความปลอดภัยของบัญชี

---

## Step 465: SSH key สำหรับ GitLab

Part 18 ของหลักสูตรนี้สอนการสร้างและตั้งค่า SSH key สำหรับ GitHub ไปแล้วอย่างละเอียด **ข่าวดีคือคำสั่งทั้งหมดเหมือนเดิมทุกประการ** เพราะ SSH เป็นโปรโตคอลมาตรฐานที่ไม่ขึ้นกับผู้ให้บริการ สิ่งที่ต่างกันมีเพียง **หน้าเว็บที่ใช้วาง public key** เท่านั้น

### 465.1 ใช้ key เดิมได้หรือต้องสร้างใหม่

คุณสามารถ **ใช้ SSH key ตัวเดียวกับที่สร้างไว้สำหรับ GitHub ใน Part 18 ได้เลย** เพราะ SSH key ไม่ได้ผูกกับผู้ให้บริการรายใดรายหนึ่ง คุณสามารถนำ public key ตัวเดียวไปเพิ่มได้ทั้งใน GitHub, GitLab, Bitbucket หรือที่ไหนก็ได้พร้อมกัน

อย่างไรก็ตาม ถ้าต้องการแยก key ระหว่างผู้ให้บริการ (เช่นเพื่อจัดการสิทธิ์แยกกันชัดเจน หรือเพื่อความปลอดภัยที่ถ้า key หนึ่งหลุดจะไม่กระทบอีกที่) ก็สามารถสร้าง key ใหม่แยกต่างหากได้ตามหลักการเดียวกับ Step 176 ใน Part 18

### 465.2 สร้าง SSH key ใหม่สำหรับ GitLab (ถ้าต้องการแยก)

```bash
ssh-keygen -t ed25519 -C "somchai@example.com" -f ~/.ssh/id_ed25519_gitlab
```

คำสั่งนี้เหมือนกับที่อธิบายไว้ใน Step 172 ของ Part 18 ทุกประการ เพียงระบุ `-f` เพื่อตั้งชื่อไฟล์แยกไม่ให้ทับ key เดิมที่ใช้กับ GitHub

จากนั้นเพิ่มเข้า ssh-agent เหมือนเดิม:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_gitlab
```

### 465.3 คัดลอก public key

```bash
# Linux
xclip -selection clipboard < ~/.ssh/id_ed25519_gitlab.pub

# macOS
pbcopy < ~/.ssh/id_ed25519_gitlab.pub

# Windows (Git Bash)
clip < ~/.ssh/id_ed25519_gitlab.pub

# หรือแสดงผลแล้วคัดลอกด้วยมือ
cat ~/.ssh/id_ed25519_gitlab.pub
```

### 465.4 เพิ่ม public key เข้าบัญชี GitLab

จุดนี้คือความต่างเดียวจาก GitHub — หน้าเว็บที่ใช้วาง key:

1. ไปที่ **Avatar > Edit profile** แล้วเลือกเมนู **SSH Keys** ในแถบด้านซ้าย (หรือ URL ตรง `https://gitlab.com/-/user_settings/ssh_keys`)
2. วางเนื้อหา public key ในช่อง **Key**
3. ตั้งค่า **Title** ให้สื่อความหมาย เช่น `MacBook ส่วนตัว` หรือ `Work Laptop`
4. (ทางเลือก) ตั้งค่า **Expiration date** — GitLab รองรับการกำหนดวันหมดอายุของ SSH key ได้ในตัว ซึ่งเป็นฟีเจอร์ที่ดีต่อความปลอดภัยระยะยาว เมื่อถึงวันหมดอายุ GitLab จะปฏิเสธการเชื่อมต่อจาก key นั้นโดยอัตโนมัติ (ต่างจาก GitHub ที่ไม่มีฟีเจอร์กำหนดวันหมดอายุของ SSH key ในลักษณะเดียวกัน)
5. คลิก **Add key**

### 465.5 ทดสอบการเชื่อมต่อ

```bash
ssh -T git@gitlab.com
```

สังเกตว่าคำสั่งเหมือน GitHub ทุกประการ เพียงเปลี่ยน host จาก `github.com` เป็น `gitlab.com` ผลลัพธ์เมื่อสำเร็จ:

```
Welcome to GitLab, @yourusername!
```

ข้อความต่างจาก GitHub เล็กน้อย (GitHub ตอบว่า "successfully authenticated" ส่วน GitLab ตอบแบบทักทายว่า "Welcome to GitLab") แต่ความหมายเดียวกันคือ **การเชื่อมต่อ SSH สำเร็จสมบูรณ์**

### 465.6 กรณีมีหลายบัญชี/หลายผู้ให้บริการพร้อมกัน

ถ้าคุณมีทั้งบัญชี GitHub และ GitLab พร้อมกัน และใช้ key แยกกัน ให้ตั้งค่า `~/.ssh/config` แบบเดียวกับที่เรียนใน Step 176 ของ Part 18 เพียงเพิ่ม Host block สำหรับ `gitlab.com` เข้าไปด้วย:

```
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519
  IdentitiesOnly yes

Host gitlab.com
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/id_ed25519_gitlab
  IdentitiesOnly yes
```

ด้วยการตั้งค่านี้ ทั้ง `git clone git@github.com:...` และ `git clone git@gitlab.com:...` จะเลือกใช้ key ที่ถูกต้องให้อัตโนมัติโดยไม่ต้องทำอะไรเพิ่มเติม

---

## Step 466: Personal Access Token ของ GitLab

### 466.1 ทำไมต้องมี Personal Access Token (PAT)

เช่นเดียวกับ GitHub, GitLab ไม่รับรหัสผ่านบัญชีตรง ๆ สำหรับการยืนยันตัวตนผ่าน HTTPS หรือผ่าน API อีกต่อไป **Personal Access Token (PAT)** คือรหัสยาวที่สร้างขึ้นมาแทนรหัสผ่าน ใช้สำหรับ:

- `git clone`/`git push`/`git pull` ผ่าน HTTPS แทนการใช้ SSH
- เรียกใช้ **GitLab API** จากสคริปต์หรือเครื่องมือภายนอก
- ยืนยันตัวตนกับ **GitLab CLI (`glab`)** (จะสอนใน Step 467)
- เชื่อมต่อกับเครื่องมือ CI/CD ภายนอกหรือ integration อื่น ๆ

### 466.2 สร้าง Personal Access Token

1. ไปที่ **Avatar > Edit profile > Access Tokens** (หรือ URL `https://gitlab.com/-/user_settings/personal_access_tokens`)
2. กรอกข้อมูล:
   - **Token name** — ชื่อที่สื่อความหมาย เช่น `glab-cli-macbook` หรือ `ci-integration-script`
   - **Expiration date** — GitLab **บังคับ**ให้กำหนดวันหมดอายุเสมอ (ไม่มีตัวเลือก "ไม่มีวันหมดอายุ" เหมือน GitHub เวอร์ชันเก่า) สูงสุดที่กำหนดได้มักอยู่ที่ 365 วัน ทั้งนี้ควรเลือกระยะเวลาสั้นที่สุดเท่าที่ยังใช้งานสะดวก
   - **Select scopes** — เลือกสิทธิ์ที่ token นี้จะมี (อธิบายรายละเอียดในหัวข้อถัดไป)
3. คลิก **Create personal access token**
4. GitLab จะแสดง token **เพียงครั้งเดียว** ทันทีหลังสร้าง ให้คัดลอกเก็บไว้ในที่ปลอดภัยทันที (เช่น password manager) เพราะถ้าปิดหน้าเว็บไปแล้วจะไม่สามารถดูค่าเดิมซ้ำได้อีก ต้องสร้าง token ใหม่แทนหากลืมคัดลอก

### 466.3 รายการ Scopes ที่สำคัญของ GitLab PAT

| Scope | ความหมาย |
|---|---|
| **api** | สิทธิ์เต็มรูปแบบในการเข้าถึง GitLab API ทั้งหมด (อ่าน/เขียน/ลบ) รวมถึงจัดการ repository, issue, merge request, CI/CD pipeline ฯลฯ — เป็น scope ที่กว้างที่สุด ควรใช้เฉพาะเมื่อจำเป็นจริง ๆ |
| **read_api** | อ่านข้อมูลผ่าน API ได้อย่างเดียว ไม่สามารถแก้ไขหรือสร้างอะไรได้ |
| **read_user** | อ่านข้อมูลโปรไฟล์ผู้ใช้ผ่าน API |
| **read_repository** | โคลนและดึงข้อมูล (pull/fetch) จาก repository ผ่าน Git ได้ แต่ push ไม่ได้ |
| **write_repository** | push โค้ดเข้า repository ได้ (ต้องใช้คู่กับ `read_repository` โดยปริยายเพื่อให้ทำงานได้ครบวงจร) |
| **read_registry** | ดึง (pull) image จาก GitLab Container Registry |
| **write_registry** | push image เข้า GitLab Container Registry |
| **create_runner** | สร้างและลงทะเบียน GitLab Runner ใหม่ผ่าน API |
| **k8s_proxy** | ใช้เชื่อมต่อกับ Kubernetes cluster ที่ผูกไว้กับ GitLab ผ่านกลไก proxy ของ GitLab |
| **sudo** (เฉพาะ Administrator ของ instance) | สามารถกระทำการแทนผู้ใช้คนอื่นผ่าน API ได้ — ใช้เฉพาะระดับดูแลระบบเท่านั้น |

### 466.4 หลักการเลือก Scope ให้เหมาะสม (Principle of Least Privilege)

แนวทางปฏิบัติที่ดีที่สุดคือ **เลือก scope ให้แคบที่สุดเท่าที่จำเป็น** ตัวอย่างสถานการณ์:

- ต้องการแค่ `git clone`/`git pull` จากสคริปต์อัตโนมัติ → เลือกแค่ `read_repository` พอ
- ต้องการให้ CI script push ผลลัพธ์กลับเข้า repository → เลือก `read_repository` + `write_repository`
- ต้องการเขียนสคริปต์ดึงรายชื่อ Issue ทั้งหมดของโปรเจกต์ → เลือก `read_api` พอ ไม่จำเป็นต้องใช้ `api` เต็มรูปแบบ
- ใช้กับ GitLab CLI สำหรับงานประจำวันทั่วไป (สร้าง MR, ดู pipeline, จัดการ issue) → มักต้องใช้ `api` เพราะ `glab` เรียกหลายปลายทางของ API

### 466.5 ตัวอย่างการใช้ PAT แทนรหัสผ่านผ่าน HTTPS

```bash
git clone https://gitlab.com/yourusername/your-project.git
Username: yourusername
Password: <วาง Personal Access Token ตรงนี้>
```

หรือฝัง token ไว้ใน URL โดยตรง (สะดวกสำหรับสคริปต์อัตโนมัติ แต่ต้องระวังไม่ commit URL นี้ลงไฟล์ที่แชร์กับใคร):

```bash
git clone https://oauth2:<your_token>@gitlab.com/yourusername/your-project.git
```

### 466.6 การจัดการและเพิกถอน Token

หน้า **Access Tokens** เดิมจะแสดงรายการ token ทั้งหมดที่คุณสร้างไว้ พร้อมวันหมดอายุและ scope ที่มี หากสงสัยว่า token หลุดหรือไม่ใช้แล้ว กดปุ่ม **Revoke** ข้าง token นั้นได้ทันที ผลคือคำขอใด ๆ ที่ใช้ token นี้จะถูกปฏิเสธทันทีโดยไม่ต้องรอให้หมดอายุตามกำหนด

### 466.7 Project Access Token และ Group Access Token (เกริ่นสั้น ๆ)

นอกจาก Personal Access Token ที่ผูกกับบัญชีผู้ใช้ GitLab ยังมี **Project Access Token** (ผูกกับ project เดียว) และ **Group Access Token** (ผูกกับ group) ซึ่งทำงานคล้าย Deploy Key ที่เรียนใน Part 18 — คือจำกัดขอบเขตให้แคบกว่าการใช้ token ส่วนตัว เหมาะสำหรับใช้ใน CI/CD pipeline หรือ integration ที่ไม่ควรผูกกับบัญชีคนใดคนหนึ่งโดยเฉพาะ เนื้อหานี้จะกล่าวถึงอย่างละเอียดอีกครั้งเมื่อเราเรียนเรื่อง GitLab Groups และ Permission ใน **Part 53**

---

## Step 467: GitLab CLI (`glab`) เบื้องต้น

### 467.1 `glab` คืออะไร

**GitLab CLI** หรือ `glab` คือเครื่องมือ command line ที่เป็นทางการของ GitLab ทำหน้าที่คล้ายกับ `gh` ของ GitHub — ให้คุณจัดการ Issue, Merge Request, Pipeline และฟีเจอร์อื่น ๆ ของ GitLab ได้โดยไม่ต้องเปิดเว็บเบราว์เซอร์เลย เหมาะมากสำหรับนักพัฒนาที่ทำงานบน terminal เป็นหลัก

### 467.2 ติดตั้ง `glab`

**macOS (ผ่าน Homebrew):**

```bash
brew install glab
```

**Windows (ผ่าน Winget หรือ Scoop):**

```powershell
winget install GLab.GLab
# หรือ
scoop install glab
```

**Linux (สคริปต์ติดตั้งอย่างเป็นทางการ ติดตั้ง binary ให้ทันทีในคำสั่งเดียว ไม่ต้องต่อด้วย apt):**

```bash
curl -s https://gitlab.com/gitlab-org/cli/-/raw/main/scripts/install.sh | sudo bash
```

**Linux (Debian/Ubuntu ผ่าน apt โดยใช้ community repository อย่าง WakeMeOps):**

```bash
curl -sSL "https://raw.githubusercontent.com/upciti/wakemeops/main/assets/install_repository" | sudo bash
sudo apt install glab
```

**Linux (ผ่าน Homebrew for Linux) หรือดาวน์โหลด binary โดยตรง:**

```bash
brew install glab
```

ตรวจสอบว่าติดตั้งสำเร็จ:

```bash
glab version
```

### 467.3 Login เข้าสู่ระบบด้วย `glab auth login`

```bash
glab auth login
```

`glab` จะถามคำถามแบบ interactive ทีละขั้น:

```
? What GitLab instance do you want to log into?  gitlab.com
? How would you like to sign in?
  ▸ Web
    Token
```

- เลือก **Web** — เปิดเบราว์เซอร์ให้ login ผ่านหน้าเว็บตามปกติแล้วยืนยันกลับมาที่ terminal อัตโนมัติ (คล้าย OAuth flow) วิธีนี้สะดวกที่สุดสำหรับการใช้งานทั่วไป
- เลือก **Token** — ใช้ Personal Access Token ที่สร้างไว้ใน Step 466 วางเข้าไปตรง ๆ เหมาะสำหรับสภาพแวดล้อมที่ไม่มีเบราว์เซอร์ เช่น server หรือ CI

ตัวอย่างการ login ด้วย token โดยตรง (ไม่ต้อง interactive):

```bash
glab auth login --hostname gitlab.com --token <your_personal_access_token>
```

### 467.4 ตรวจสอบสถานะการ login

```bash
glab auth status
```

ผลลัพธ์ตัวอย่าง:

```
gitlab.com
  ✓ Logged in to gitlab.com as yourusername
  ✓ Git operations for gitlab.com configured to use ssh protocol
  ✓ Token: glpat-xxxxxxxxxxxxxxxxxxxx
```

### 467.5 คำสั่งพื้นฐานที่ใช้บ่อย

```bash
# ดูรายการ Merge Request ของโปรเจกต์ปัจจุบัน (ต้องรันในโฟลเดอร์ที่ clone repo GitLab ไว้)
glab mr list

# สร้าง Merge Request ใหม่แบบ interactive
glab mr create

# ดูรายการ Issue ทั้งหมด
glab issue list

# สร้าง Issue ใหม่อย่างรวดเร็ว
glab issue create --title "แก้บั๊กหน้า login" --description "รายละเอียด..."

# ดูสถานะ CI/CD pipeline ล่าสุด
glab pipeline status

# เปิด repository ปัจจุบันบนเว็บเบราว์เซอร์ทันที
glab repo view --web

# Clone repository ผ่าน glab (เลือก protocol ที่ตั้งค่าไว้อัตโนมัติ)
glab repo clone yourusername/your-project
```

### 467.6 ตั้งค่าให้ `glab` ใช้ SSH เป็นค่าเริ่มต้น

```bash
glab config set git_protocol ssh
```

การตั้งค่านี้ทำให้ทุกครั้งที่ `glab repo clone` หรือคำสั่งอื่นที่ต้องอ้างอิง URL ของ repository จะใช้รูปแบบ `git@gitlab.com:...` (SSH) แทน `https://gitlab.com/...` โดยอัตโนมัติ สอดคล้องกับ SSH key ที่เราตั้งค่าไว้ใน Step 465

---

## Step 468: การนำทาง GitLab UI ภาพรวม — ต่างจาก GitHub ตรงไหนบ้าง

ก่อนเข้าสู่ Part 48 ที่จะสร้าง repository และ Merge Request จริง มาทำความคุ้นเคยกับหน้าตาของ GitLab UI กันก่อน โดยเฉพาะจุดที่ต่างจาก GitHub อย่างชัดเจน ซึ่งมักทำให้ผู้ใช้ GitHub ที่มาลอง GitLab ครั้งแรกงงเล็กน้อย

### 468.1 โครงสร้าง Sidebar หลัก (ซ้ายมือ)

เมื่อ login เข้า GitLab.com หน้าแรกจะมี **sidebar ด้านซ้าย** ที่เป็นเมนูหลักตลอดการใช้งาน ต่างจาก GitHub ที่เน้นเมนูบนแถบบน (top navigation bar) เป็นหลัก:

```
┌─────────────────────┐
│ 🔍 Search GitLab     │
├─────────────────────┤
│ 🏠 Your work         │
│ 📁 Projects          │
│ 👥 Groups            │
│ ⭐ Starred projects  │
│ 🔔 Explore           │
├─────────────────────┤
│ Recent items         │
│ (โปรเจกต์/กลุ่มที่    │
│  เพิ่งเปิดล่าสุด)     │
└─────────────────────┘
```

- **Your work** — ศูนย์รวม dashboard ส่วนตัว: Issue ที่ถูก assign, Merge Request ที่รอรีวิว, To-Do list
- **Projects** — รายการ repository (ใน GitLab เรียกว่า "Project" ไม่ใช่ "Repository" เฉย ๆ เพราะ 1 Project อาจมีมากกว่าแค่ code repository เช่นมี wiki, issue tracker, CI/CD ในตัว)
- **Groups** — แนวคิดคล้าย Organization ของ GitHub แต่ยืดหยุ่นกว่า เพราะ Group สามารถมี **subgroup** ซ้อนกันเป็นชั้น ๆ ได้ไม่จำกัดระดับ (จะอธิบายลึกใน Part 53)

### 468.2 หน้า Project Overview

เมื่อเปิดเข้าไปในโปรเจกต์หนึ่ง ๆ sidebar ด้านซ้ายจะเปลี่ยนเป็นเมนูเฉพาะของโปรเจกต์นั้น:

```
Project Name
├── 📊 Manage        (Members, Labels, Activity)
├── 📝 Plan          (Issues, Boards, Milestones, Wiki)
├── 💻 Code          (Repository, Branches, Commits, Tags, Merge Requests)
├── 🏗️ Build         (Pipelines, Jobs, Artifacts, Pipeline Editor)
├── 🚀 Deploy        (Releases, Container Registry, Environments)
├── 🔒 Secure        (Security dashboard, Dependency list — บางส่วนต้อง Premium/Ultimate)
├── 📈 Monitor       (Metrics, Alerts — บางส่วนต้อง Premium/Ultimate)
├── ⚙️ Settings      (General, Members, Integrations, CI/CD, Repository)
```

สังเกตว่า GitLab จัดกลุ่มเมนูตาม **แนวคิด DevOps lifecycle** (Plan → Code → Build → Deploy → Secure → Monitor) ซึ่งสะท้อนปรัชญาการเป็น "DevOps Platform ครบวงจร" ที่กล่าวถึงใน Part 46 ชัดเจนมาก ต่างจาก GitHub ที่จัดเมนูแบบแท็บเรียบง่ายกว่า (Code, Issues, Pull requests, Actions, Projects, Wiki, Security, Insights, Settings)

### 468.3 ตารางเทียบศัพท์และตำแหน่งเมนู GitHub vs GitLab

| แนวคิด/ฟีเจอร์ | GitHub | GitLab |
|---|---|---|
| ที่เก็บโค้ด | Repository | Project (ซึ่งครอบคลุมมากกว่าแค่โค้ด) |
| องค์กรที่รวมหลาย repo | Organization | Group (รองรับ subgroup ซ้อนได้) |
| คำขอรวมโค้ด | Pull Request (PR) | Merge Request (MR) |
| CI/CD | GitHub Actions (`.github/workflows/*.yml`) | GitLab CI/CD (`.gitlab-ci.yml` ไฟล์เดียวที่ root) |
| หน้าเว็บ static hosting | GitHub Pages | GitLab Pages |
| Container registry ในตัว | GitHub Container Registry (`ghcr.io`) | GitLab Container Registry (ในตัวทุกโปรเจกต์) |
| To-Do รวมทุกโปรเจกต์ | Notifications tab | To-Do List (มี badge นับจำนวนแยกชัดเจนบน sidebar) |
| การค้นหาโค้ดทั่วทั้งระบบ | GitHub code search | GitLab Advanced Search (บางฟีเจอร์ต้อง Premium/Ultimate) |

### 468.4 ความต่างเชิงปรัชญาโดยสรุป

GitHub ออกแบบ UI ให้ดูเรียบง่าย เน้นความเร็วในการเข้าถึงโค้ดและ Pull Request เป็นหลัก ส่วน GitLab ออกแบบ UI ให้สะท้อนวงจรชีวิตของซอฟต์แวร์ทั้งหมด ตั้งแต่วางแผนงานไปจนถึง monitor ระบบ production ทำให้เมนูของ GitLab ดูเยอะกว่าในตอนแรก แต่เมื่อคุ้นเคยแล้วจะพบว่าทุกอย่างที่ต้องใช้ในงาน DevOps อยู่ในที่เดียวโดยไม่ต้องพึ่งเครื่องมือภายนอกมากเท่า GitHub

---

## Step 469: แนะนำ GitLab Duo / AI features เบื้องต้น

Step นี้เป็นการแนะนำแบบผิวเผินเพื่อให้รู้จักไว้ก่อน ไม่ได้ลงลึกวิธีใช้งานแบบละเอียด เพราะฟีเจอร์กลุ่มนี้พัฒนาเปลี่ยนแปลงเร็วมากและหลายส่วนอยู่ในระดับแพ็กเกจ Premium/Ultimate

### 469.1 GitLab Duo คืออะไร

**GitLab Duo** คือชื่อรวมของฟีเจอร์ผู้ช่วย AI ที่ GitLab ผนวกเข้ากับแพลตฟอร์มของตัวเอง ครอบคลุมความสามารถหลายอย่างตลอดวงจรการพัฒนาซอฟต์แวร์ ไม่ใช่แค่การเขียนโค้ดอย่างเดียว

### 469.2 ความสามารถหลักที่ GitLab Duo มี (ภาพรวม)

| ความสามารถ | คำอธิบายคร่าว ๆ |
|---|---|
| **Code Suggestions** | แนะนำโค้ดอัตโนมัติขณะพิมพ์ใน IDE (คล้าย GitHub Copilot) รองรับการเชื่อมกับ VS Code, JetBrains IDEs และอื่น ๆ |
| **Duo Chat** | แชทถามตอบกับ AI ภายใน GitLab UI โดยตรง เช่น ถามให้อธิบายโค้ด สรุป Merge Request ที่ยาวมาก หรือช่วยเขียน commit message |
| **Merge Request Summary** | สรุปการเปลี่ยนแปลงใน Merge Request ยาว ๆ ให้เข้าใจง่ายก่อนรีวิว |
| **Vulnerability explanation** | อธิบายช่องโหว่ด้านความปลอดภัยที่ระบบ Security Scanning ตรวจพบ พร้อมแนะนำแนวทางแก้ไข |
| **Root cause analysis สำหรับ CI/CD** | ช่วยวิเคราะห์ว่าทำไม pipeline ถึง fail โดยไม่ต้องไล่อ่าน log ทั้งหมดเอง |

### 469.3 ระดับการเข้าถึง

GitLab Duo แบ่งเป็นหลายระดับตามแพ็กเกจการสมัครสมาชิก (เช่นระดับพื้นฐานบางส่วนอาจรวมอยู่ในแพ็กเกจที่สูงกว่า Free ในขณะที่บางฟีเจอร์เปิดให้ทดลองใช้ฟรีในบางช่วงเวลา) รายละเอียดเรื่องแพ็กเกจและราคาเปลี่ยนแปลงได้บ่อย ควรตรวจสอบหน้า pricing อย่างเป็นทางการของ GitLab เสมอเพื่อดูสิทธิ์ล่าสุดที่ตรงกับบัญชีของคุณ

### 469.4 ทำไมหลักสูตรนี้ไม่เน้นฟีเจอร์นี้มาก

หลักสูตรนี้มุ่งเน้นให้คุณ **เข้าใจกลไกพื้นฐานของ Git และ GitLab อย่างแท้จริง** ก่อน เพราะเป็นทักษะพื้นฐานที่ไม่เปลี่ยนแปลงไปตามกาลเวลา ส่วนฟีเจอร์ AI อย่าง GitLab Duo เป็นเครื่องมือเสริมที่ช่วยเพิ่มประสิทธิภาพการทำงาน แต่รายละเอียดปลีกย่อยเปลี่ยนแปลงเร็วมากในแต่ละเวอร์ชัน การรู้จักไว้คร่าว ๆ ก็เพียงพอในจุดนี้ ถ้าองค์กรของคุณเปิดใช้งาน GitLab Duo แนะนำให้ลองเปิด Duo Chat ดูจากหน้าโปรเจกต์ (มักมีไอคอนรูปดาวหรือสัญลักษณ์ AI อยู่มุมหน้าจอ) เพื่อสัมผัสประสบการณ์จริงด้วยตัวเอง

---

## Step 470: แบบฝึกหัด — ตั้งค่าบัญชี GitLab เต็มรูปแบบพร้อมเชื่อมต่อผ่าน SSH สำเร็จ

ถึงเวลาลงมือทำจริงทุกขั้นตอนที่เรียนมาใน Part นี้ ให้ทำตามลำดับต่อไปนี้ครบทุกข้อ ก่อนไป Part 48

### 470.1 รายการงานที่ต้องทำ

1. **สมัครบัญชี GitLab.com** ด้วยอีเมลของคุณเอง (ถ้ายังไม่มีบัญชี) ตามขั้นตอนใน Step 461
2. **ยืนยันอีเมล** และผ่านหน้า onboarding questions เรียบร้อย
3. **ตั้งค่า Profile** — ใส่ชื่อเต็ม, อัปโหลด avatar อย่างน้อย 1 รูป
4. **เปิดใช้งาน 2FA** ด้วยแอป Authenticator ที่คุณเลือก และ **บันทึก recovery codes** ไว้ในที่ปลอดภัย
5. **สร้าง SSH key** (ใช้ key เดิมจาก Part 18 หรือสร้างใหม่แยกสำหรับ GitLab ก็ได้) แล้วเพิ่ม public key เข้าบัญชี GitLab พร้อมตั้งวันหมดอายุ
6. **ทดสอบการเชื่อมต่อ SSH** ด้วยคำสั่ง `ssh -T git@gitlab.com` จนเห็นข้อความ `Welcome to GitLab, @yourusername!`
7. **สร้าง Personal Access Token** อย่างน้อย 1 ตัว กำหนด scope ให้เหมาะสม (ลองสร้างแบบ scope แคบ เช่น `read_repository` เท่านั้น เพื่อฝึกหลักการ Least Privilege) และบันทึก token ไว้ในที่ปลอดภัย
8. **ติดตั้ง `glab`** และรัน `glab auth login` จน `glab auth status` แสดงสถานะ login สำเร็จ
9. **ตั้งค่า `glab config set git_protocol ssh`** ให้เรียบร้อย
10. **สำรวจ GitLab UI** — ลองคลิกผ่านเมนู Your work, Projects, Groups และเปิด Duo Chat ดูสักครั้ง (ถ้าบัญชีของคุณมีสิทธิ์เข้าถึง) เพื่อความคุ้นเคย

### 470.2 วิธีตรวจสอบว่าทำสำเร็จครบทุกข้อ

รันชุดคำสั่งตรวจสอบนี้ในเครื่องของคุณ:

```bash
# ตรวจสอบการเชื่อมต่อ SSH กับ GitLab
ssh -T git@gitlab.com

# ตรวจสอบว่า glab ติดตั้งและ login สำเร็จ
glab version
glab auth status

# ตรวจสอบว่า glab ใช้ SSH protocol
glab config get git_protocol
```

ผลลัพธ์ที่ถูกต้องควรมี:

- ข้อความ `Welcome to GitLab, @yourusername!` จากคำสั่ง `ssh -T`
- `glab auth status` แสดง `✓ Logged in to gitlab.com as yourusername`
- `glab config get git_protocol` แสดงค่า `ssh`

### 470.3 หากติดปัญหา

| อาการ | สาเหตุที่เป็นไปได้ | วิธีแก้ |
|---|---|---|
| `Permission denied (publickey)` ตอน `ssh -T git@gitlab.com` | ยังไม่ได้เพิ่ม public key เข้าบัญชี หรือ key หมดอายุไปแล้ว | ตรวจสอบหน้า SSH Keys ในบัญชี ลองสร้าง key ใหม่และเพิ่มใหม่ |
| `glab auth login` ค้างที่ขั้นตอนเปิดเบราว์เซอร์ | เบราว์เซอร์เริ่มต้นของเครื่องตั้งค่าไม่ถูกต้อง หรือรันบน server ที่ไม่มีเบราว์เซอร์ | เปลี่ยนไปใช้ตัวเลือก Token แทนตอน login |
| ไม่เห็นเมนู Duo Chat | บัญชี Free อาจยังไม่ได้เปิดสิทธิ์เข้าถึงฟีเจอร์นี้ | ข้ามได้ ไม่กระทบการเรียน Part ถัดไปแต่อย่างใด |

### 470.4 Checklist ก่อนไป Part 48

- [ ] มีบัญชี GitLab.com ที่ยืนยันอีเมลแล้ว
- [ ] ตั้งค่า Profile และเปิดใช้งาน 2FA พร้อมบันทึก recovery codes แล้ว
- [ ] เพิ่ม SSH public key เข้าบัญชี และทดสอบ `ssh -T git@gitlab.com` สำเร็จ
- [ ] สร้าง Personal Access Token อย่างน้อย 1 ตัว และเข้าใจความหมายของ scope หลัก ๆ
- [ ] ติดตั้งและ login `glab` สำเร็จ พร้อมตั้งค่า git protocol เป็น SSH
- [ ] เข้าใจภาพรวมว่า GitLab Self-Managed ติดตั้งได้ทั้งแบบ Docker และแบบ Omnibus package ต่างกันอย่างไร
- [ ] คุ้นเคยกับโครงสร้าง Sidebar และเมนูของ GitLab UI พอที่จะนำทางได้เองในบทถัดไป

---

## สรุป Part 47

ใน Part นี้เราได้เรียนรู้และลงมือทำว่า:

1. สมัครบัญชี GitLab.com (SaaS) ตั้งแต่ต้นจนเสร็จสมบูรณ์ พร้อมเข้าใจโควตาของ Free tier
2. GitLab Self-Managed ติดตั้งได้ในระดับแนวคิดผ่าน **Docker** (`docker run gitlab/gitlab-ce`) สำหรับทดลอง และผ่าน **Omnibus package** สำหรับใช้งานจริงบน server โดยไม่ลงรายละเอียด infrastructure ลึกเกินไป
3. ตั้งค่าบัญชี GitLab ให้พร้อมใช้งานจริง: Profile, Two-Factor Authentication และ Recovery codes
4. สร้างและเพิ่ม SSH key เข้า GitLab — ใช้หลักการและคำสั่งเดียวกันกับ GitHub ใน Part 18 ทุกประการ ต่างแค่หน้าเว็บที่วาง key
5. สร้าง Personal Access Token พร้อมเข้าใจ scope สำคัญอย่าง `api`, `read_repository`, `write_repository` และหลักการ Least Privilege
6. ติดตั้งและใช้งาน GitLab CLI (`glab`) เบื้องต้น ตั้งแต่ login จนถึงคำสั่งจัดการ MR, Issue และ Pipeline พื้นฐาน
7. เข้าใจโครงสร้าง GitLab UI ที่จัดกลุ่มตามวงจร DevOps (Plan, Code, Build, Deploy, Secure, Monitor) และเทียบศัพท์กับ GitHub ได้อย่างชัดเจน
8. รู้จัก GitLab Duo และฟีเจอร์ AI เบื้องต้นแบบผิวเผิน เพื่อเตรียมพร้อมสำหรับการใช้งานจริงในอนาคต

ตอนนี้บัญชี GitLab ของคุณพร้อมใช้งานเต็มรูปแบบแล้ว ทั้ง SSH, Personal Access Token และ CLI พร้อมสำหรับการสร้าง repository และเรียนรู้ Merge Request workflow ใน Part ถัดไป

**ต่อไป:** [Part 48: GitLab Repository และ Merge Request เบื้องต้น](./part-048-gitlab-repository-merge-request.md)
