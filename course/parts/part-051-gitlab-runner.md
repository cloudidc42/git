# Part 51: GitLab Runner: การตั้งค่าและใช้งาน

> **Step ในหลักสูตรนี้:** Step 501–510
> **เฟส:** 5 — เจาะลึก GitLab และ CI/CD เบื้องต้นของมัน
> **เป้าหมายของ Part นี้:** เข้าใจว่า GitLab Runner คืออะไร ทำหน้าที่อะไรในระบบ CI/CD ของ GitLab แยกความแตกต่างระหว่าง Shared Runner กับ Self-hosted Runner ติดตั้งและลงทะเบียน Runner ของตัวเองได้จริง เข้าใจเรื่อง tags, executor, concurrency, autoscaling และประเด็นความปลอดภัยของ self-hosted runner จนสามารถแก้ปัญหา pipeline ค้างเพราะไม่มี runner ได้ด้วยตัวเอง

---

## สารบัญของ Part นี้

- Step 501: GitLab Runner คืออะไร ทำหน้าที่อะไร
- Step 502: Shared Runners vs Specific/Self-hosted Runners
- Step 503: การติดตั้ง Runner ของตัวเอง (ภาพรวม)
- Step 504: การลงทะเบียน runner กับ project/group
- Step 505: Runner tags — กำหนดว่า job ไหนรันบน runner ไหน
- Step 506: Docker executor เจาะลึก (image, services)
- Step 507: Runner concurrency และ autoscaling เบื้องต้น
- Step 508: การ debug เมื่อ pipeline ค้างเพราะไม่มี runner ที่ match
- Step 509: Security ของ self-hosted runner
- Step 510: แบบฝึกหัด — ติดตั้งและลงทะเบียน GitLab Runner แบบ Docker executor เอง

---

## Step 501: GitLab Runner คืออะไร ทำหน้าที่อะไร

ใน Part ก่อน ๆ ของเฟส 5 เราเห็นแล้วว่า `.gitlab-ci.yml` คือไฟล์ที่ **บอก** GitLab ว่าอยากให้ pipeline ทำอะไรบ้าง แบ่งเป็น stage และ job อย่างไร แต่มีคำถามสำคัญที่ยังไม่ได้ตอบ:

> **ใครเป็นคนรันคำสั่งใน `script:` ของแต่ละ job จริง ๆ?**

คำตอบคือ **GitLab Runner**

### นิยาม

**GitLab Runner** คือ **โปรแกรม (agent/application)** แบบ open source เขียนด้วยภาษา Go มีหน้าที่เดียวคือ:

> **รับงาน (job) จาก GitLab instance มาประมวลผลจริง แล้วรายงานผลลัพธ์กลับไป**

จุดสำคัญที่ต้องเข้าใจให้แม่นตั้งแต่ต้นคือ **GitLab เองไม่ได้รันโค้ดของคุณ** ตัว GitLab instance (ไม่ว่าจะเป็น GitLab.com หรือ self-managed) ทำหน้าที่เป็นเพียง **"ผู้ประสานงาน" (coordinator)**:

- สร้าง pipeline และ job ตามที่ `.gitlab-ci.yml` กำหนด
- เก็บ job เหล่านั้นไว้ใน "คิว" (queue) พร้อมสถานะ `pending`
- รอให้ **Runner** มาติดต่อขอรับงานไปทำ
- รับผลลัพธ์ (log, exit code, artifacts) กลับมาแสดงผลในหน้าเว็บ

ส่วน **Runner** ต่างหากที่เป็นคนลงมือ:

1. **Poll (สอบถามเป็นระยะ)** ไปยัง GitLab instance ผ่าน API ว่า "มีงานให้ทำไหม"
2. เมื่อพบ job ที่ตรงเงื่อนไข (tags, executor ที่รองรับ) ก็ **รับงานมา (claim)**
3. **สร้างสภาพแวดล้อมสำหรับรันงาน (execution environment)** ตาม `executor` ที่ตั้งค่าไว้ เช่น container ใหม่, shell บนเครื่อง, หรือ Pod ใน Kubernetes
4. **Clone โค้ดจาก repository** ลงมาในสภาพแวดล้อมนั้น
5. **รันคำสั่งใน `script:`** ของ job นั้นทีละบรรทัด
6. **สตรีม log กลับไปแบบ real-time** ให้เห็นในหน้า GitLab
7. เก็บ **artifacts/cache** (ถ้ามีการตั้งค่า) แล้วอัปโหลดกลับไปที่ GitLab
8. รายงานผลลัพธ์สุดท้าย (`success`, `failed`, `canceled` ฯลฯ) กลับไปที่ GitLab

### ภาพรวมของสถาปัตยกรรม

```
Developer push code
        │
        ▼
┌─────────────────────┐
│   GitLab Instance    │  ← "Coordinator" ไม่รันโค้ดเอง
│  (GitLab.com หรือ     │
│   self-managed)      │
│                      │
│  - อ่าน .gitlab-ci.yml│
│  - สร้าง Pipeline/Job │
│  - เก็บ job ในคิว      │
└──────────┬───────────┘
           │ polling ผ่าน HTTPS API
           │ (Runner ถามหางานเป็นระยะ)
           ▼
┌─────────────────────┐
│    GitLab Runner      │  ← โปรแกรม agent ที่คุณติดตั้งเอง
│  (บนเครื่องคุณ/server/ │     หรือ GitLab.com เตรียมให้ (shared)
│   VM/Kubernetes)      │
└──────────┬───────────┘
           │ สร้าง executor (container/shell/pod)
           ▼
┌─────────────────────┐
│   Execution Environment│  ← ที่ script: ของ job รันจริง ๆ
│ (เช่น Docker container│
│  จาก image: ที่ระบุ)   │
└──────────────────────┘
```

### Runner รันอยู่ที่ไหนก็ได้

Runner ไม่จำเป็นต้องอยู่ในเครื่องเดียวกับ GitLab instance เลย มันเป็นแค่ **client** ที่คุยกับ GitLab ผ่าน HTTPS ดังนั้นคุณสามารถ:

- ติดตั้ง Runner บนแล็ปท็อปตัวเองเพื่อทดสอบ
- ติดตั้งบน VM/server แยกต่างหาก
- รันเป็น container ใน Kubernetes cluster
- ให้ GitLab.com จัดหา Runner ให้ฟรีแบบ "shared" (จะพูดถึงใน Step 502)

### ตารางสรุปบทบาทของแต่ละองค์ประกอบ

| องค์ประกอบ | ทำหน้าที่ | รันโค้ดของเราเองหรือไม่ |
|---|---|---|
| **GitLab instance** | เก็บ repo, อ่าน `.gitlab-ci.yml`, สร้างและจัดคิว job, แสดงผล UI | **ไม่** |
| **GitLab Runner** | poll หางาน, สร้าง environment, สั่งให้ executor รันงาน, ส่งผลกลับ | ไม่โดยตรง (สั่งให้ executor รัน) |
| **Executor** (Docker container/shell/Kubernetes pod ฯลฯ) | สภาพแวดล้อมจริงที่คำสั่งใน `script:` ถูก execute | **ใช่ — ที่นี่คือจุดที่โค้ดรันจริง** |

จำหลักการนี้ไว้ให้แม่น เพราะมันจะช่วยให้เข้าใจ Step ที่เหลือทั้งหมดของ Part นี้ได้ง่ายขึ้นมาก:

> **GitLab บอกว่า "ต้องทำอะไร" (.gitlab-ci.yml) — Runner เป็นคนหางานมาทำ — Executor เป็นที่ที่งานนั้นถูกรันจริง**

---

## Step 502: Shared Runners vs Specific/Self-hosted Runners

เมื่อคุณสร้าง project ใหม่บน **GitLab.com** แล้วเพิ่ม `.gitlab-ci.yml` เข้าไป pipeline มักจะรันได้ทันทีโดยที่คุณไม่ต้องติดตั้งอะไรเองเลย นั่นเป็นเพราะ GitLab.com มี **Shared Runners** เตรียมไว้ให้อยู่แล้ว

### Shared Runners (หรือชื่อใหม่: Instance Runners)

**Shared Runners** คือ runner ที่ **GitLab เป็นผู้ดูแลและจัดหาให้** โดยมีลักษณะสำคัญ:

- ใช้ **ร่วมกันได้กับทุก project** ในระบบ (บน GitLab.com คือทุก project สาธารณะ/ส่วนตัวที่เปิดใช้ shared runner ไว้)
- GitLab เป็นผู้รับผิดชอบเรื่อง infrastructure, การอัปเดตเวอร์ชัน, ความพร้อมใช้งานทั้งหมด
- บน **GitLab.com** shared runners มี **โควตาการใช้งานเป็นนาที** ต่อเดือนต่อ namespace (เรียกว่า **compute minutes/compute quota**) — แผนฟรีจะได้โควตาจำกัดต่อเดือน แผนที่เสียเงินจะได้โควตาสูงขึ้นหรือซื้อ minutes เพิ่มได้ ตัวเลขโควตาที่แน่นอนเปลี่ยนแปลงได้ตามนโยบายของ GitLab จึงควรตรวจสอบค่าปัจจุบันที่หน้า **Settings > Usage Quotas** ของ namespace เสมอ
- เมื่อโควตาหมด pipeline จะไม่สามารถรันบน shared runner ต่อได้จนกว่าจะรีเซ็ตรอบบิล หรือคุณเพิ่ม runner ของตัวเองเข้าไป
- รองรับหลาย OS: Linux, Windows, macOS (สำหรับ build ที่ต้องใช้ macOS เช่น iOS app)
- บน **GitLab self-managed** (ที่ติดตั้งเอง เช่นผ่าน Omnibus package) "Shared Runner" หมายถึง runner ที่แอดมินขององค์กรลงทะเบียนไว้ในระดับ **instance** ให้ทุก project ในเซิร์ฟเวอร์นั้นใช้ร่วมกันได้

> **หมายเหตุเรื่องศัพท์:** ตั้งแต่ GitLab เปลี่ยนกระบวนการลงทะเบียน runner ใหม่ (GitLab 15.10 เป็นต้นมา และเต็มรูปแบบใน 16.0) เอกสารทางการเริ่มใช้คำว่า **instance runner** แทน "shared runner", **group runner** แทน "specific runner ระดับกลุ่ม" และ **project runner** แทน "specific runner ระดับ project" แต่ในบทสนทนาทั่วไปและใน UI บางส่วนยังคงเห็นคำเดิมปะปนอยู่ ในหลักสูตรนี้จะใช้ทั้งสองคำสลับกันเพื่อให้คุ้นเคยกับทั้งสองแบบ

### Specific Runners / Self-hosted Runners (Project & Group Runners)

**Specific Runner** (หรือ **Project Runner** / **Group Runner** ในศัพท์ใหม่) คือ runner ที่:

- **คุณติดตั้งและดูแลเอง** บนเครื่อง/เซิร์ฟเวอร์ของคุณเอง (self-hosted)
- **ผูกกับ project หรือ group ที่กำหนดเท่านั้น** — ไม่ได้แชร์ใช้กับทุกคนบน GitLab.com
- **ไม่มีโควตานาทีจาก GitLab** — ใช้งานได้ไม่จำกัดตามข้อจำกัดฮาร์ดแวร์และเวลาที่เครื่องของคุณเองมี
- เหมาะกับกรณี:
  - ต้องการควบคุม environment เอง (ติดตั้ง library เฉพาะทาง, license เฉพาะ)
  - ต้อง deploy เข้า network ภายในองค์กรที่ shared runner จาก internet เข้าไม่ถึง
  - ต้องการ hardware พิเศษ เช่น GPU สำหรับ machine learning build
  - งาน build ใช้เวลานานมาก จนโควตา shared runner ไม่พอ
  - ต้องการความปลอดภัยสูงกว่า ไม่อยากให้โค้ด/ความลับไปวิ่งบน infrastructure ของ GitLab

### ตารางเปรียบเทียบ

| หัวข้อ | Shared / Instance Runner | Specific / Self-hosted Runner |
|---|---|---|
| ดูแลโดย | GitLab (GitLab.com หรือแอดมิน instance) | คุณเอง |
| ใช้ร่วมกับ project อื่นไหม | ใช่ (ทุก project ที่เปิดใช้งาน) | ไม่ — ผูกกับ project/group ที่ลงทะเบียนเท่านั้น (เว้นแต่ตั้งเป็น group runner ให้ project ลูกในกลุ่มใช้ร่วมกัน) |
| ค่าใช้จ่าย | มีโควตานาทีฟรีต่อเดือน เกินโควตาต้องจ่ายเพิ่ม | ค่าใช้จ่ายคือค่าเครื่อง/เซิร์ฟเวอร์ของคุณเอง ไม่มีโควตานาทีจาก GitLab |
| ปรับแต่ง environment | จำกัดตาม image ที่ GitLab เตรียมให้ | ปรับแต่งได้เต็มที่ตามต้องการ |
| เหมาะกับ | โปรเจกต์ทั่วไป, Open Source, เริ่มต้นเรียนรู้ | องค์กร, งาน deploy เข้าเครือข่ายภายใน, งานที่ต้องการ hardware พิเศษ |
| ตั้งค่า tags เอง | ทำได้ (แต่ต้องรอ GitLab อนุญาต) | ทำได้เต็มที่ตอนลงทะเบียน |

### จะเปิด/ปิด Shared Runners ของ project ได้อย่างไร

ไปที่ **Project > Settings > CI/CD > Runners** จะเห็น section "Instance runners" (หรือ "Shared runners" ในเวอร์ชันเก่า) มี toggle เปิด/ปิดการใช้งาน shared runner สำหรับ project นั้น ถ้าปิดไว้ pipeline จะรอ **เฉพาะ** specific runner ที่ลงทะเบียนกับ project นี้เท่านั้นมารับงาน — และถ้าไม่มี specific runner เลย pipeline จะค้าง (เรื่องนี้จะกลับมาพูดถึงอีกครั้งใน Step 508)

ในหน้าเดียวกันนี้เองที่เราจะใช้ลงทะเบียน Runner ของตัวเองใน Step ถัดไป

---

## Step 503: การติดตั้ง Runner ของตัวเอง (ภาพรวม)

ก่อนจะลงมือติดตั้งจริงใน Step 510 มาทำความเข้าใจภาพรวมของกระบวนการติดตั้งและตัวเลือกต่าง ๆ กันก่อน

### ระบบปฏิบัติการ/แพลตฟอร์มที่รองรับ

GitLab Runner รองรับการติดตั้งบนแทบทุกแพลตฟอร์ม:

| แพลตฟอร์ม | วิธีติดตั้งหลัก |
|---|---|
| Linux (Debian/Ubuntu, RHEL/CentOS ฯลฯ) | package repository อย่างเป็นทางการ (`packages.gitlab.com`) ผ่าน `apt`/`yum` หรือ binary ตรง ๆ |
| Windows | ติดตั้งเป็น Windows service ผ่าน `.exe` binary |
| macOS | ติดตั้งผ่าน Homebrew หรือ binary ตรง ๆ |
| FreeBSD | binary ตรง ๆ |
| Docker | รันเป็น container จาก image ทางการ `gitlab/gitlab-runner` |
| Kubernetes | ติดตั้งผ่าน Helm chart หรือ GitLab Runner Operator |

### ตัวอย่างการติดตั้งบน Linux (Debian/Ubuntu)

```bash
# เพิ่ม repository อย่างเป็นทางการ
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash

# ติดตั้งตัวโปรแกรม
sudo apt-get install gitlab-runner
```

### ตัวอย่างการติดตั้งผ่าน Docker (นิยมมากในการเรียนรู้/ทดสอบ)

```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

คำสั่งนี้จะรัน gitlab-runner เป็น container ที่:
- เก็บไฟล์ config (`config.toml`) ไว้ที่ host path `/srv/gitlab-runner/config` (persist ข้ามการ restart container)
- mount `docker.sock` เข้าไปด้วย เพื่อให้ runner สามารถสั่งสร้าง container ลูกอื่น ๆ บน host เดียวกันได้ (จำเป็นสำหรับ Docker executor)

### แนวคิดเรื่อง `gitlab-runner register`

ไม่ว่าจะติดตั้งด้วยวิธีไหน ขั้นตอนถัดไปที่ต้องทำเสมอคือ **การลงทะเบียน (register)** runner เข้ากับ GitLab instance/project ผ่านคำสั่ง:

```bash
gitlab-runner register
```

คำสั่งนี้จะถามข้อมูลแบบ interactive ทีละขั้น (หรือใส่ flag แบบ non-interactive ก็ได้):

1. **GitLab instance URL** — เช่น `https://gitlab.com/`
2. **Registration token หรือ Runner authentication token** (รายละเอียดใน Step 504)
3. **คำอธิบาย (description)** ของ runner นี้
4. **Tags** — คำสำคัญที่ใช้จับคู่กับ job (Step 505)
5. **Executor** — เลือกว่าจะรันงานแบบไหน (รายละเอียดด้านล่าง)
6. ค่าตั้งต้นเฉพาะของ executor นั้น ๆ เช่น default image สำหรับ Docker executor

ผลลัพธ์คือไฟล์ `config.toml` (มักอยู่ที่ `/etc/gitlab-runner/config.toml` บน Linux) จะถูกสร้าง/แก้ไข เพิ่ม section `[[runners]]` ใหม่เข้าไป

### ประเภทของ Executor — หัวใจสำคัญที่ต้องเข้าใจ

**Executor** คือ **วิธีที่ runner ใช้สร้างสภาพแวดล้อมสำหรับรัน job** แต่ละแบบมีข้อดี/ข้อเสียต่างกัน:

| Executor | อธิบาย | ข้อดี | ข้อเสีย/ข้อควรระวัง |
|---|---|---|---|
| **shell** | รันคำสั่งตรง ๆ บน shell ของเครื่อง host เลย (bash, sh, PowerShell, cmd) | ตั้งค่าง่ายที่สุด เร็วที่สุด (ไม่ต้อง provision environment ใหม่) | **ไม่มีการแยก (isolation)** ระหว่าง job — ไฟล์/process ที่หลงเหลือจาก job ก่อนอาจกระทบ job ถัดไป เสี่ยงเรื่อง security สูงถ้ารันโค้ดที่ไม่น่าเชื่อถือ |
| **docker** | รัน job แต่ละครั้งใน **container ใหม่** ตาม `image:` ที่ระบุใน `.gitlab-ci.yml` | isolate สะอาด ทำซ้ำได้ (reproducible) นิยมใช้มากที่สุด | ต้องมี Docker daemon บน host, ใช้เวลา pull image ครั้งแรก |
| **docker-windows** | เหมือน docker แต่สำหรับ Windows container | ใช้กับ workload ที่ต้อง build บน Windows | ต้องมี Windows host ที่รองรับ |
| **docker+machine** (docker-machine) | executor สำหรับ **autoscaling** — provision/destroy VM ผ่าน Docker Machine ตามความต้องการ | สเกลอัตโนมัติ ประหยัดค่าใช้จ่ายเวลาไม่มีงาน | **โปรเจกต์ Docker Machine ถูก archive ไปแล้วตั้งแต่ปี 2021** ไม่มีการพัฒนาต่อ GitLab แนะนำให้ย้ายไปใช้ GitLab Runner Autoscaler แทน (ดู Step 507) |
| **kubernetes** | รันแต่ละ job เป็น **Pod** ใหม่ในคลัสเตอร์ Kubernetes | scale ได้ดีในสภาพแวดล้อม cloud-native, ใช้ resource ของคลัสเตอร์ที่มีอยู่แล้ว | ต้องมีความรู้ Kubernetes, ตั้งค่า RBAC/namespace ให้ถูกต้อง |
| **custom** | ให้คุณเขียน script (`config_exec`, `prepare_exec`, `run_exec` ฯลฯ) กำหนดพฤติกรรม executor เอง | ยืดหยุ่นสูงสุด รองรับ infrastructure แปลก ๆ ได้ | ซับซ้อน ต้องเขียนและดูแลเอง |
| ssh / parallels / virtualbox | รันงานผ่าน SSH หรือใน VM | เหมาะกับ legacy infrastructure บางกรณี | ปัจจุบันใช้น้อยลงมาก |
| **instance** (ใหม่ ตั้งแต่ GitLab Runner 16.6+) | executor สำหรับ autoscaling รุ่นใหม่ ทำงานร่วมกับ Fleeting plugin | ทดแทน docker-machine อย่างเป็นทางการ | ยังค่อนข้างใหม่ ต้องศึกษาเอกสารล่าสุด |

สำหรับหลักสูตรนี้ (และแบบฝึกหัดใน Step 510) เราจะใช้ **Docker executor** เป็นหลัก เพราะเป็นตัวเลือกที่สมดุลที่สุดระหว่างความง่ายในการตั้งค่ากับความปลอดภัย/ความสามารถในการทำซ้ำผลลัพธ์ (reproducibility) และเป็น executor ที่ทีมส่วนใหญ่ในโลกจริงใช้กันมากที่สุด

---

## Step 504: การลงทะเบียน runner กับ project/group

การลงทะเบียน (registration) คือขั้นตอนที่ทำให้ runner ที่ติดตั้งไว้ **"รู้จัก"** ว่าตัวเองต้องไปขอรับงานจาก GitLab instance ไหน และ project/group ใด GitLab มีสองวิธีหลักในการลงทะเบียน ซึ่งเป็นเรื่องสำคัญมากที่ต้องเข้าใจความแตกต่าง เพราะ GitLab เปลี่ยนแนวทางนี้ครั้งใหญ่

### วิธีเก่า: Registration Token (Legacy)

ก่อนหน้านี้ ในหน้า **Settings > CI/CD > Runners** ของ project หรือ group จะมี **Registration Token** ตัวเดียวแสดงอยู่ตลอดเวลา (เป็นค่าคงที่สำหรับ project/group นั้น) แล้วนำไปใช้กับคำสั่ง:

```bash
sudo gitlab-runner register \
  --url "https://gitlab.com/" \
  --registration-token "GR1348941xxxxxxxxxxxxxxxxxxxx" \
  --executor "docker" \
  --docker-image "alpine:latest"
```

**ปัญหาของวิธีนี้:**

- Registration token เป็น **ความลับที่ใช้ได้ซ้ำได้ไม่จำกัดจำนวนครั้ง** — ใครก็ตามที่มี token นี้สามารถลงทะเบียน runner ปลอมเข้ามารับงานของ project ได้ (เสี่ยงต่อการรั่วไหลของ secrets ในงาน CI)
- ไม่สามารถระบุความเป็นเจ้าของ/ตรวจสอบย้อนหลังได้ชัดเจนว่า runner แต่ละตัวถูกสร้างโดยใครหรือเมื่อไหร่
- ด้วยเหตุผลด้านความปลอดภัยนี้ **GitLab จึงประกาศ deprecate วิธีนี้** และตั้งแต่ **GitLab 17.0 เป็นต้นไป การลงทะเบียนด้วย registration token ถูกปิดใช้งานเป็นค่าเริ่มต้น** (แอดมิน instance ยังสามารถเปิดกลับมาใช้ชั่วคราวได้ในบาง self-managed instance แต่ไม่แนะนำ)

### วิธีใหม่ (แนะนำ): Runner Authentication Token

ตั้งแต่ **GitLab 16.0** เป็นต้นไป (เริ่มทยอยใช้ตั้งแต่ 15.10) กระบวนการเปลี่ยนเป็น:

1. **สร้าง runner เป็น "entity" ในหน้าเว็บก่อน** ที่ **Settings > CI/CD > Runners > New project runner** (หรือ New group runner สำหรับระดับ group)
2. ในหน้าสร้าง runner คุณจะกำหนดค่าล่วงหน้าได้เลย เช่น:
   - **Tags** ที่ runner นี้จะมี
   - **Run untagged jobs** (จะอนุญาตให้รับ job ที่ไม่มี tags ไหม)
   - **Protected** (จำกัดให้รันได้เฉพาะ protected branch/tag เท่านั้น — ดู Step 509)
   - **Maximum job timeout**
   - Platform ที่จะใช้ (Linux/macOS/Windows) เพื่อให้ GitLab แสดงคำสั่งติดตั้งที่ตรง OS ให้อัตโนมัติ
3. กด **Create runner** — GitLab จะสร้าง **Runner authentication token** ที่ขึ้นต้นด้วย `glrt-` ให้ **แสดงเพียงครั้งเดียว** พร้อมคำสั่ง `gitlab-runner register` ที่พร้อมใช้งานให้คัดลอกไปรันได้ทันที
4. นำ token นั้นไปรันคำสั่งลงทะเบียนบนเครื่อง runner:

```bash
sudo gitlab-runner register \
  --url "https://gitlab.com/" \
  --token "glrt-xxxxxxxxxxxxxxxxxxxx" \
  --executor "docker" \
  --docker-image "alpine:latest"
```

**ข้อดีของวิธีใหม่:**

- Token ผูกกับ **runner ตัวเดียวโดยเฉพาะ** ไม่ใช่ token กลางที่ใช้ซ้ำได้ทุกคน
- ตั้งค่า tags/protected/description ได้ **ก่อน** ลงทะเบียนจริง ผ่าน UI ที่ตรวจสอบได้
- Revoke (เพิกถอน) token ของ runner ตัวใดตัวหนึ่งได้โดยไม่กระทบ runner ตัวอื่น
- ตรวจสอบย้อนหลังได้ว่า runner แต่ละตัวถูกสร้างจากที่ไหน โดยใคร เมื่อไหร่ (audit log)

### ระดับของการลงทะเบียน

| ระดับ | ลงทะเบียนที่ไหน | ใครใช้ได้ |
|---|---|---|
| **Project runner** | Project > Settings > CI/CD > Runners | เฉพาะ project นั้น (หรือ project ที่ share ให้เพิ่มเติมได้) |
| **Group runner** | Group > Settings > CI/CD > Runners | ทุก project ที่อยู่ภายใต้ group นั้น (รวม subgroup) |
| **Instance runner** (เดิมเรียก Shared runner) | Admin Area > CI/CD > Runners (ต้องเป็นแอดมินของ instance) | ทุก project บน instance นั้นทั้งหมด |

### ตรวจสอบว่าลงทะเบียนสำเร็จ

หลังลงทะเบียนเสร็จ กลับไปที่หน้า **Settings > CI/CD > Runners** ของ project จะเห็น runner ของเราปรากฏในรายการ พร้อมสถานะ (ควรขึ้นเป็น **Online** สีเขียวภายในเวลาไม่กี่วินาทีถ้า runner process กำลังรันอยู่และเชื่อมต่อ GitLab ได้)

---

## Step 505: Runner tags — กำหนดว่า job ไหนรันบน runner ไหน

เมื่อองค์กรมี runner หลายตัวที่มีความสามารถต่างกัน (บางตัวมี GPU, บางตัวอยู่ใน network ที่เข้าถึง production ได้, บางตัวรันบน macOS) เราจำเป็นต้อง **บอก GitLab ว่า job ไหนควรไปรันบน runner ตัวไหน** นี่คือหน้าที่ของ **tags**

### หลักการทำงาน

- ตอนลงทะเบียน (หรือแก้ไขภายหลังผ่าน UI) เราจะกำหนด **ชุดของ tags** ให้ runner แต่ละตัว เช่น `docker`, `linux`, `production`, `gpu`
- ใน `.gitlab-ci.yml` แต่ละ job สามารถระบุ `tags:` เป็นรายการคำที่ต้องการได้
- **กฎการจับคู่:** job จะถูกส่งไปรันบน runner ตัวใดก็ได้ **ที่มี tag ครบทุกตัวตามที่ job ระบุ** (runner มี tag เป็น superset ของ tag ที่ job ต้องการ — runner จะมี tag เกินกว่านั้นก็ได้ ไม่เป็นไร)
- ถ้า job **ไม่ระบุ `tags:` เลย** (job แบบ "untagged") มันจะถูกส่งไปรันได้เฉพาะ runner ที่เปิดตัวเลือก **"Run untagged jobs"** ไว้เท่านั้น — runner ที่ปิดตัวเลือกนี้จะไม่รับ job ที่ไม่มี tags

### ตัวอย่าง

สมมติมี runner 3 ตัว:

| ชื่อ runner | Tags |
|---|---|
| `runner-A` | `docker`, `linux` |
| `runner-B` | `docker`, `linux`, `gpu` |
| `runner-C` | `shell`, `windows` |

และมี `.gitlab-ci.yml`:

```yaml
build-app:
  stage: build
  tags:
    - docker
    - linux
  script:
    - echo "build ทั่วไป"

train-model:
  stage: test
  tags:
    - docker
    - linux
    - gpu
  script:
    - echo "งานที่ต้องใช้ GPU"

legacy-windows-build:
  stage: build
  tags:
    - windows
  script:
    - echo "งาน windows"
```

ผลการจับคู่:

- `build-app` ต้องการ `docker` + `linux` → **`runner-A` หรือ `runner-B` รับได้ทั้งคู่** (เพราะมี tag ครบตามที่ขอ)
- `train-model` ต้องการ `docker` + `linux` + `gpu` → **มีแค่ `runner-B` เท่านั้น** ที่มี tag ครบ
- `legacy-windows-build` ต้องการ `windows` → **มีแค่ `runner-C`**

### จุดที่ผิดพลาดบ่อย

1. **พิมพ์ tag ผิด/ตัวพิมพ์เล็กใหญ่ไม่ตรงกัน** — tags เป็น case-sensitive และต้องตรงตัวอักษรทุกตัว (`Docker` ≠ `docker`)
2. **ลืมเปิด "Run untagged jobs"** บน runner แล้วสงสัยว่าทำไม job ที่ไม่มี tags ไม่ยอมรัน
3. **ระบุ tag ที่ไม่มี runner ตัวไหนมีเลย** → job จะค้างเป็น `pending` ตลอดไป (รายละเอียดวิธีวินิจฉัยอยู่ใน Step 508)

### การแก้ไข tags ภายหลัง

ไม่จำเป็นต้อง unregister แล้ว register ใหม่ทุกครั้งที่อยากเปลี่ยน tags — ไปที่ **Settings > CI/CD > Runners** แล้วกด **Edit** (ไอคอนดินสอ) ที่ runner ตัวนั้นในหน้าเว็บได้เลย ระบบจะอัปเดต tags ทันทีโดยไม่ต้องแตะ config บนเครื่อง runner แต่อย่างใด

---

## Step 506: Docker executor เจาะลึก (image, services)

เนื่องจาก Docker executor เป็นตัวเลือกที่ใช้กันมากที่สุด มาเจาะลึกสองคีย์เวิร์ดสำคัญที่สุดที่ใช้คู่กับมันใน `.gitlab-ci.yml`: `image:` และ `services:`

### `image:` — container ที่ job จะรันอยู่ข้างใน

เมื่อ runner ใช้ Docker executor ทุกครั้งที่รับ job มา มันจะ:

1. `docker pull` (หรือใช้ image ที่มีอยู่แล้วตาม `pull_policy`) ตาม `image:` ที่ระบุ
2. สร้าง container ใหม่จาก image นั้น
3. Clone repository เข้าไปใน container
4. รันคำสั่งใน `script:` ข้างในนั้น
5. ทำลาย container ทิ้งเมื่อ job จบ (ไม่ว่าจะสำเร็จหรือ fail)

```yaml
test-node-app:
  image: node:20
  stage: test
  script:
    - npm ci
    - npm test
```

`image:` กำหนดได้ทั้งระดับ **global** (ใน `default:` block) และ **override เฉพาะ job** ได้ ถ้า job ไหนไม่ระบุ `image:` เอง จะใช้ค่า default ที่ตั้งไว้ใน `config.toml` ของ runner (ตอน register ด้วย `--docker-image`) หรือค่า `default: image:` ใน `.gitlab-ci.yml`

### `services:` — container เสริมที่รันคู่กันไปด้วย

หลาย job ต้องพึ่งพา service ภายนอก เช่น database, cache, message queue ระหว่างทดสอบ Docker executor รองรับสิ่งนี้ผ่าน `services:` ซึ่งจะสร้าง **container เพิ่มเติม** ที่เชื่อมต่อเครือข่ายเดียวกับ container หลักของ job โดยอัตโนมัติ

```yaml
variables:
  POSTGRES_DB: myapp_test
  POSTGRES_USER: runner
  POSTGRES_PASSWORD: "secret-password"
  POSTGRES_HOST_AUTH_METHOD: trust

test-with-database:
  image: python:3.12
  services:
    - postgres:15
  stage: test
  script:
    - pip install -r requirements.txt
    - python manage.py test
```

หลักการสำคัญ:

- container ของ service จะ **เข้าถึงได้ผ่าน hostname ที่ตรงกับชื่อ image** (ในตัวอย่างคือ `postgres`) — โค้ดในแอปจึงต้อง connect ไปที่ host `postgres` (ไม่ใช่ `localhost`)
- ตัวแปร CI/CD ที่ตั้งไว้ใน `variables:` (เช่น `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`) จะถูกส่งเข้าไปให้ **ทั้ง job container และ service container** โดยอัตโนมัติ — official Postgres image จะอ่านตัวแปรเหล่านี้ตอน initialize database ให้เอง
- ตั้งชื่อ alias เองได้ด้วยไวยากรณ์แบบเต็ม เผื่อกรณีต้องใช้ image เดียวกันหลาย service หรือ hostname ที่ต่างไปจากชื่อ image:

```yaml
services:
  - name: postgres:15
    alias: db
```

### Docker-in-Docker (DinD) — เมื่อ job ต้องสั่งงาน Docker เอง

ถ้า job ต้อง **build Docker image** เอง (เช่น ทำ `docker build` ภายใน pipeline) จะต้องใช้ pattern พิเศษที่เรียกว่า **Docker-in-Docker**:

```yaml
build-image:
  image: docker:24
  services:
    - docker:24-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  stage: build
  script:
    - docker build -t myapp:latest .
```

จุดที่ต้องระวัง:

- ต้องเปิด `privileged = true` ใน config ของ runner (ใน `[runners.docker]`) ให้กับ Docker executor เพราะ DinD ต้องการสิทธิ์ระดับสูงในการรัน daemon ซ้อนใน container — **นี่คือความเสี่ยงด้าน security ที่ต้องพิจารณาให้ดี** (จะกล่าวถึงเพิ่มใน Step 509)
- ทางเลือกที่ปลอดภัยกว่าคือใช้เครื่องมืออย่าง **Kaniko** หรือ **Buildah** ที่สามารถ build container image ได้โดยไม่ต้องขอสิทธิ์ `privileged`

### ตัวเลือกสำคัญใน `config.toml` ของ Docker executor

```toml
[[runners]]
  name = "docker-runner-01"
  url = "https://gitlab.com/"
  token = "glrt-xxxxxxxxxxxxxxxxxxxx"
  executor = "docker"
  [runners.docker]
    image = "alpine:latest"
    privileged = false
    pull_policy = ["if-not-present"]
    volumes = ["/cache"]
    network_mode = ""
```

| ค่า | ความหมาย |
|---|---|
| `image` | image เริ่มต้นถ้า job ไม่ได้ระบุ `image:` เอง |
| `privileged` | เปิดสิทธิ์ container แบบเต็ม (ต้องเปิดสำหรับ DinD) |
| `pull_policy` | นโยบายการดึง image ใหม่ (`always`, `if-not-present`, `never`) — มีผลต่อทั้งความเร็วและความปลอดภัย |
| `volumes` | mount path ที่แชร์กับทุก container ที่สร้างขึ้น เช่นใช้เก็บ cache |
| `network_mode` / `extra_hosts` | ปรับแต่งเครือข่ายของ container ที่สร้างขึ้น |

---

## Step 507: Runner concurrency และ autoscaling เบื้องต้น

### `concurrent` — จำกัดจำนวนงานที่รันพร้อมกันได้

ในไฟล์ `config.toml` จะมีค่า **`concurrent`** อยู่นอก section `[[runners]]` (เป็นค่าระดับ global ของ runner process ทั้งตัว):

```toml
concurrent = 4

[[runners]]
  name = "docker-runner-01"
  url = "https://gitlab.com/"
  token = "glrt-xxxxxxxxxxxxxxxxxxxx"
  executor = "docker"
  limit = 2
  [runners.docker]
    image = "alpine:latest"
```

- `concurrent = 4` หมายถึง **runner process ตัวนี้รันได้สูงสุด 4 job พร้อมกัน** (รวมทุก `[[runners]]` section ที่อยู่ในไฟล์เดียวกัน)
- `limit` ภายใน `[[runners]]` แต่ละอัน จำกัดจำนวน job พร้อมกัน **เฉพาะ runner ตัวนั้น** (ต้องไม่เกินค่า `concurrent` รวม)

ถ้าตั้งค่าต่ำเกินไป job จะต้องรอคิว (`pending`) แม้ว่า runner จะ online อยู่ก็ตาม เพราะ slot การรันเต็มหมดแล้ว — นี่เป็นอีกสาเหตุหนึ่งของ pipeline ที่ดูเหมือนค้าง (จะกล่าวถึงใน Step 508)

### Autoscaling — เมื่อ runner คงที่ไม่พอ

ถ้า workload ของทีมขึ้นลงไม่แน่นอน (บางช่วงมี pipeline วิ่งพร้อมกันเป็นสิบ บางช่วงว่างสนิท) การมี runner แบบ fixed จำนวนคงที่ตลอดเวลาจะเปลืองทรัพยากร/ค่าใช้จ่ายเวลาที่ไม่มีงาน หรือไม่พอเวลามีงานเยอะ วิธีแก้คือ **autoscaling**

**แนวทางเดิม (legacy): docker+machine executor**

ในอดีต GitLab Runner รองรับการ autoscale ผ่าน executor แบบ `docker+machine` ซึ่งใช้เครื่องมือ **Docker Machine** ในการ provision/destroy VM instance บน cloud provider (เช่น AWS EC2, Google Compute Engine) ตามความต้องการ ตัวอย่างแนวคิดของ config (เพื่อความเข้าใจภาพรวมเท่านั้น):

```toml
[[runners]]
  name = "autoscale-runner"
  url = "https://gitlab.com/"
  token = "glrt-xxxxxxxxxxxxxxxxxxxx"
  executor = "docker+machine"
  limit = 20
  [runners.machine]
    IdleCount = 1
    IdleTime = 1800
    MaxBuilds = 100
    MachineDriver = "amazonec2"
    MachineName = "gitlab-runner-autoscale-%s"
    MachineOptions = [
      "amazonec2-instance-type=t3.medium",
      "amazonec2-region=ap-southeast-1"
    ]
```

หลักการ: เมื่อมี job เข้าคิวเกินกว่า `IdleCount` runner จะสั่ง provision VM ใหม่ผ่าน driver ที่ระบุ (เช่น `amazonec2`) รันงานเสร็จแล้วปล่อยให้ idle รอสักพัก (`IdleTime`) ถ้าไม่มีงานใหม่ก็จะ destroy VM ทิ้งเพื่อประหยัดค่าใช้จ่าย

> **ข้อควรทราบสำคัญ:** โปรเจกต์ **Docker Machine ถูก archive (หยุดพัฒนา) อย่างเป็นทางการไปแล้วตั้งแต่ปี 2021** แม้ GitLab Runner จะยังรองรับ executor นี้อยู่เพื่อความเข้ากันได้กับระบบเดิม แต่ **ไม่แนะนำให้เริ่มต้นระบบใหม่ด้วย docker+machine อีกต่อไป**

**แนวทางใหม่ที่แนะนำ: GitLab Runner Autoscaler (Fleeting)**

GitLab พัฒนาสถาปัตยกรรม autoscaling รุ่นใหม่ที่เรียกว่า **Fleeting** ซึ่งทำงานผ่าน plugin เฉพาะของแต่ละ cloud provider เช่น `fleeting-plugin-aws`, `fleeting-plugin-googlecloud`, `fleeting-plugin-azure` โดยจับคู่กับ executor รุ่นใหม่อย่าง **`instance`** หรือ **`docker-autoscaler`** สถาปัตยกรรมนี้ถูกออกแบบมาแทนที่ docker-machine โดยเฉพาะ รองรับการจัดการ fleet ของ VM ได้อย่างมีประสิทธิภาพและปลอดภัยกว่าเดิม

สำหรับหลักสูตรระดับนี้ ให้จำหลักการสำคัญไว้พอ: **autoscaling มีไว้เพื่อสร้าง/ทำลาย compute resource ตามปริมาณงานจริงโดยอัตโนมัติ** ส่วนรายละเอียดการตั้งค่า Fleeting plugin แบบเต็มรูปแบบ เราจะกลับมาเจาะลึกอีกครั้งในเฟส DevOps ขั้นสูงของหลักสูตรนี้

---

## Step 508: การ debug เมื่อ pipeline ค้างเพราะไม่มี runner ที่ match

อาการ **"pipeline ค้าง"** (job สถานะเป็น `pending` ค้างอยู่นานผิดปกติ) เป็นปัญหาที่พบบ่อยที่สุดสำหรับคนที่เพิ่งเริ่มตั้งค่า runner เอง GitLab มักจะแสดงข้อความเตือนในหน้า job ตรง ๆ เช่น:

> "This job is stuck because you don't have any active runners that can run this job."

หรือ

> "This job is stuck, because the project doesn't have any runners online assigned to it."

### Checklist การวินิจฉัย (ไล่ตามลำดับนี้)

**1. ตรวจสอบว่า runner ออนไลน์อยู่จริงไหม**

ไปที่ **Settings > CI/CD > Runners** ดูสถานะ runner ควรเป็นจุดสีเขียว **"Online"** พร้อมเวลา "Last contact" ที่เพิ่งอัปเดตไม่เกิน 1-2 นาทีที่ผ่านมา ถ้าขึ้น **"Offline"** หรือ "Never contacted" แปลว่า runner process ไม่ได้กำลังรันอยู่ หรือเชื่อมต่อ GitLab ไม่ได้

**2. ตรวจสอบ tags ให้ตรงกัน**

เทียบ `tags:` ใน `.gitlab-ci.yml` ของ job นั้นกับ tags ของ runner ที่มีอยู่จริง — พิมพ์ผิดตัวอักษรเดียวก็ทำให้ไม่ match แล้ว ถ้า job ไม่มี `tags:` เลย ให้ตรวจว่า runner เปิด **"Run untagged jobs"** ไว้หรือไม่

**3. ตรวจสอบว่า Shared Runner ถูกปิดอยู่หรือเปล่า**

ถ้าโปรเจกต์ปิด "Instance/Shared runners" ไว้ (ตั้งใจหรือไม่ตั้งใจก็ตาม) และไม่มี specific runner ของตัวเองเลย pipeline จะค้างทันที เพราะไม่มี runner ตัวไหนเข้าเงื่อนไขให้จับงานได้เลย

**4. ตรวจสอบ Protected Runner กับ branch**

ถ้า runner ถูกตั้งเป็น **Protected** (ดู Step 509) มันจะรับงานได้ **เฉพาะ job ที่รันบน protected branch/tag เท่านั้น** ถ้า job มาจาก feature branch ธรรมดาที่ไม่ได้ถูกตั้งเป็น protected ก็จะไม่มี runner ตัวนี้มารับงาน

**5. ตรวจสอบว่า concurrency เต็มหรือยัง**

ถ้า runner ตั้ง `concurrent`/`limit` ไว้ต่ำ และมี job อื่นค้างอยู่เต็ม slot แล้ว job ใหม่ก็ต้องรอต่อคิวไปเรื่อย ๆ (ลองดูใน UI ว่ามี job อื่นสถานะ `running` ค้างอยู่จำนวนเท่าไหร่)

**6. ตรวจสอบสถานะของ Docker daemon / executor บนเครื่อง runner**

ถ้า runner online แต่ job ยัง fail ตั้งแต่ขั้นตอน "Preparing the runner environment" ให้ตรวจว่า Docker daemon บนเครื่อง host ยังทำงานปกติอยู่ไหม (`sudo systemctl status docker`)

**7. ตรวจสอบเรื่อง network/firewall**

Runner ต้องสามารถเชื่อมต่อ **ขาออก (outbound)** ไปยัง URL ของ GitLab instance ผ่าน HTTPS ได้เสมอ ถ้าอยู่หลัง proxy หรือ firewall ที่บล็อกการเชื่อมต่อ ต้องตั้งค่า proxy ให้ runner หรือเปิด allowlist ให้ถูกต้อง สำหรับ self-managed GitLab ที่ใช้ self-signed certificate ต้องเพิ่ม CA certificate ให้ runner รู้จักผ่านค่า `tls-ca-file` ใน `config.toml` ไม่เช่นนั้นจะเชื่อมต่อไม่ได้เลย

### คำสั่ง CLI ที่มีประโยชน์สำหรับ debug

```bash
# ตรวจสอบว่า runner ที่ลงทะเบียนไว้ยังคุยกับ GitLab ได้ปกติไหม
sudo gitlab-runner verify

# แสดงรายการ runner ทั้งหมดที่ config.toml เครื่องนี้รู้จัก
sudo gitlab-runner list

# ตรวจสอบสถานะ service ของ runner (บน Linux แบบ systemd)
sudo systemctl status gitlab-runner

# ดู log แบบ real-time
sudo journalctl -u gitlab-runner -f
```

### ตารางสรุป: อาการ → สาเหตุที่เป็นไปได้ → วิธีแก้

| อาการ | สาเหตุที่เป็นไปได้ | วิธีแก้ |
|---|---|---|
| Job ค้าง `pending` ตลอดกาล ไม่มี runner ไหนรับ | tags ไม่ตรงกับ runner ตัวไหนเลย | แก้ tags ให้ตรง หรือเปิด "Run untagged" |
| Job ค้าง แม้ runner ขึ้น "Online" | Protected runner แต่ job มาจาก branch ที่ไม่ protected | ปิด Protected ชั่วคราว หรือเปลี่ยนไป test บน protected branch |
| Runner ขึ้น "Offline"/"Never contacted" | runner process ไม่ทำงาน หรือเชื่อมต่อ GitLab ไม่ได้ | เช็ค service/journalctl, เช็ค network/firewall/TLS |
| Job หลาย job รอคิวพร้อมกัน แม้ runner online | ถึงขีดจำกัด `concurrent`/`limit` | เพิ่มค่า `concurrent`/`limit` หรือเพิ่มจำนวน runner |
| Job fail ตอน "Preparing environment" | Docker daemon บน host ล่ม/ไม่พร้อม | restart Docker service, ตรวจ `docker.sock` permission |

---

## Step 509: Security ของ self-hosted runner

การรัน CI/CD ของตัวเองมาพร้อมกับความรับผิดชอบด้านความปลอดภัยที่ต้องเข้าใจให้ดี เพราะ **self-hosted runner คือเครื่องคอมพิวเตอร์จริงที่รันโค้ดตามใจที่ `.gitlab-ci.yml` สั่ง** — ถ้าโค้ดนั้นมาจากคนที่ไม่น่าเชื่อถือ อันตรายก็ตามมาด้วย

### ความเสี่ยงหลัก: Job จาก forked merge request

สถานการณ์อันตรายที่พบบ่อยที่สุดในโปรเจกต์ Open Source คือ:

1. คนแปลกหน้า fork โปรเจกต์ของคุณ
2. แก้ `.gitlab-ci.yml` ในฟอร์กของตัวเอง ใส่คำสั่งอันตรายเข้าไป เช่น สคริปต์ที่พยายามอ่านค่า environment variable ที่เป็นความลับ (secrets/API keys) แล้วส่งออกไปยัง server ภายนอก
3. เปิด merge request กลับมายัง project ต้นฉบับ
4. ถ้า pipeline ของ merge request นี้ถูกรันบน **runner ของคุณเอง** ที่มีสิทธิ์เข้าถึง CI/CD variables ที่เป็นความลับ (เช่น deploy key, cloud credential) โค้ดอันตรายนั้นก็จะรันด้วยสิทธิ์เดียวกับ job ปกติทันที

นี่คือเหตุผลที่ต้องเข้าใจและตั้งค่าป้องกันให้ดี ไม่ใช่แค่ "ติดตั้ง runner แล้วจบ"

### แนวทางป้องกันที่สำคัญ

**1. ใช้ Protected Runner**

runner ที่ตั้งค่า **Protected** ไว้ (แก้ได้ที่หน้า edit runner ใน Settings > CI/CD > Runners) จะ **รับงานได้เฉพาะ job ที่รันอยู่บน protected branch หรือ protected tag เท่านั้น** เนื่องจาก branch/tag ที่เป็น protected ปกติจะจำกัดสิทธิ์การ push ไว้เฉพาะ maintainer ที่เชื่อถือได้ การผูก runner ที่เข้าถึงความลับสำคัญไว้กับ protected branch เท่านั้น จึงช่วยกันไม่ให้โค้ดจาก branch แปลกปลอมหรือ merge request จากคนนอกมาแตะ runner ตัวนั้นได้

**2. แยก Pool ของ runner ตาม "ระดับความน่าเชื่อถือ"**

แนวทางที่ดีที่สุดคือ **อย่าใช้ runner ตัวเดียวกันสำหรับทั้ง job ทดสอบทั่วไปของ contributor ภายนอก และ job deploy ที่ต้องใช้ credential การเข้าถึง production** ควรแยกเป็นอย่างน้อย 2 กลุ่ม:

- Runner สำหรับ CI ทั่วไป (build/test) ที่ไม่ถือ credential สำคัญใด ๆ — เปิดให้ทุก merge request (รวมจาก fork) ใช้ได้
- Runner สำหรับ deploy production ที่ถือ credential สำคัญ — ตั้งเป็น Protected และจำกัดให้รันได้เฉพาะ pipeline บน `main`/protected branch เท่านั้น

**3. ระวังการเปิด `privileged = true`**

อย่างที่กล่าวใน Step 506 การเปิด `privileged` mode ให้ Docker executor (สำหรับ Docker-in-Docker) หมายความว่า container ที่รันมีสิทธิ์เกือบเทียบเท่ากับ host เอง ถ้าโค้ดอันตรายหลุดเข้ามารันในโหมดนี้ ความเสียหายอาจลุกลามไปถึงตัว host เครื่อง runner เองได้เลย ควรเปิดเฉพาะเมื่อจำเป็นจริง ๆ และพิจารณาใช้ Kaniko/Buildah แทนถ้าเป็นไปได้

**4. ใช้ Protected Variables คู่กัน**

ตัวแปร CI/CD ที่ตั้งค่า **Protected** ไว้ (Settings > CI/CD > Variables) จะถูกส่งเข้า pipeline **เฉพาะเมื่อรันบน protected branch/tag เท่านั้น** เช่นกัน ควรใช้คู่กับ Protected Runner เสมอสำหรับ credential ที่สำคัญ เพื่อให้มีการป้องกันสองชั้น

**5. ใช้ Executor แบบ ephemeral (สร้างใหม่ทุกครั้ง)**

Docker executor และ Kubernetes executor มีข้อดีด้าน security อย่างหนึ่งคือ **container/pod ที่ใช้รันงานจะถูกทำลายทิ้งทันทีหลัง job จบ** ทำให้ไม่มีไฟล์ตกค้าง (เช่น credential ที่ถูก dump ไว้ชั่วคราวระหว่างรัน) เหลืออยู่ให้ job ถัดไปเข้าถึง ต่างจาก **shell executor** ที่รันตรงบนเครื่อง host โดยไม่มีการล้าง state ระหว่าง job เลย จึงมีความเสี่ยงสูงกว่ามากถ้าต้องรับ job จากคนที่ไม่น่าเชื่อถือ

**6. อัปเดต gitlab-runner ให้เป็นเวอร์ชันล่าสุดอยู่เสมอ**

เหมือนซอฟต์แวร์ทุกตัว ตัว GitLab Runner เองก็อาจมีช่องโหว่ (CVE) ที่ถูกแพตช์ในแต่ละเวอร์ชัน ควรติดตาม release note และอัปเดตสม่ำเสมอ

**7. จำกัดสิทธิ์เครือข่ายของเครื่อง runner**

ถ้า runner ไม่จำเป็นต้องเข้าถึง internal network ที่สำคัญ (เช่น database production) ก็ไม่ควรวางเครื่อง runner ไว้ใน network segment เดียวกับระบบเหล่านั้น ควรแยก network/VLAN ให้ชัดเจนตามหลัก least privilege

### สรุปเป็นตาราง

| ความเสี่ยง | มาตรการป้องกัน |
|---|---|
| Job จาก fork เข้าถึง secrets | Protected Runner + Protected Variables |
| Docker-in-Docker (`privileged`) เปิดช่องโหว่ระดับ host | จำกัดการใช้เฉพาะที่จำเป็น พิจารณา Kaniko/Buildah |
| Shell executor ทิ้ง state ข้าม job | ใช้ Docker/Kubernetes executor แทนเมื่อรับงานจากคนนอก |
| Runner เวอร์ชันเก่ามีช่องโหว่ | อัปเดต gitlab-runner สม่ำเสมอ |
| Runner เข้าถึง network ภายในเกินจำเป็น | แยก network segment ตาม least privilege |

---

## Step 510: แบบฝึกหัด — ติดตั้งและลงทะเบียน GitLab Runner แบบ Docker executor เอง

ถึงเวลาลงมือทำจริงทั้งหมด ตั้งแต่สร้าง runner บนหน้าเว็บ ติดตั้งจริงด้วย Docker executor ลงทะเบียนกับ project และรัน pipeline จนสำเร็จ

### สิ่งที่ต้องมีก่อนเริ่ม

- Project บน GitLab (GitLab.com หรือ self-managed) ที่คุณเป็นเจ้าของหรือมีสิทธิ์ Maintainer ขึ้นไป
- เครื่องที่ติดตั้ง **Docker** ไว้แล้ว (จะใช้เครื่องเดียวกับที่รัน gitlab-runner ก็ได้)

### ขั้นตอนที่ 1: สร้าง Runner บนหน้าเว็บ GitLab ก่อน

1. เข้าไปที่ project ของคุณ > **Settings > CI/CD**
2. ขยาย section **Runners**
3. กด **New project runner**
4. เลือก **Platform: Linux**
5. ใส่ **Tags** เป็น `docker` และ `demo`
6. เปิดหรือปิด **Run untagged jobs** ตามต้องการ (สำหรับแบบฝึกหัดนี้ปล่อยปิดไว้ก็ได้ เพราะเราจะระบุ tags ใน job อยู่แล้ว)
7. ใส่ **Description** เช่น `docker-runner-demo`
8. กด **Create runner**
9. GitLab จะแสดง **Runner authentication token** (ขึ้นต้นด้วย `glrt-`) และคำสั่ง `gitlab-runner register` ที่พร้อมใช้ — **คัดลอก token นี้เก็บไว้ (จะแสดงให้เห็นแค่ครั้งเดียว)**

### ขั้นตอนที่ 2: รัน gitlab-runner ผ่าน Docker

สร้างโฟลเดอร์สำหรับเก็บ config บนเครื่อง host ก่อน:

```bash
mkdir -p /srv/gitlab-runner/config
```

รัน container ของ gitlab-runner แบบถาวร:

```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

### ขั้นตอนที่ 3: ลงทะเบียน Runner แบบ non-interactive

รันคำสั่งลงทะเบียนโดยใส่ token ที่คัดลอกไว้จากขั้นตอนที่ 1:

```bash
docker run --rm -it \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --token "glrt-xxxxxxxxxxxxxxxxxxxx" \
  --executor "docker" \
  --docker-image "alpine:latest" \
  --description "docker-runner-demo" \
  --docker-volumes "/var/run/docker.sock:/var/run/docker.sock"
```

คำอธิบาย flag ที่ใช้:

| Flag | ความหมาย |
|---|---|
| `--url` | ที่อยู่ของ GitLab instance |
| `--token` | Runner authentication token จากขั้นตอนที่ 1 |
| `--executor docker` | เลือกใช้ Docker executor |
| `--docker-image alpine:latest` | image เริ่มต้นถ้า job ไม่ระบุ `image:` เอง |
| `--docker-volumes` | mount `docker.sock` เข้าไปในทุก container ที่ runner สร้าง (จำเป็นถ้า job ต้องสั่งงาน Docker ต่อ) |

ถ้าลงทะเบียนสำเร็จจะเห็นข้อความประมาณ `Runner registered successfully.` และไฟล์ `/srv/gitlab-runner/config/config.toml` จะถูกสร้าง/แก้ไขให้มี section `[[runners]]` ใหม่เข้ามา

### ขั้นตอนที่ 4: ตรวจสอบสถานะบนหน้าเว็บ

กลับไปที่ **Settings > CI/CD > Runners** ของ project — ควรเห็น runner `docker-runner-demo` ปรากฏพร้อมสถานะ **Online** (จุดสีเขียว) ภายในเวลาไม่นาน

### ขั้นตอนที่ 5: เขียน `.gitlab-ci.yml` ให้ยิงตรงไปที่ runner ตัวนี้

สร้างหรือแก้ไขไฟล์ `.gitlab-ci.yml` ที่ root ของ repository:

```yaml
stages:
  - test

hello-from-my-runner:
  stage: test
  image: alpine:latest
  tags:
    - docker
    - demo
  script:
    - echo "สวัสดีจาก Runner ของเราเอง!"
    - uname -a
    - cat /etc/os-release
```

จุดสำคัญ: `tags:` ในไฟล์นี้ต้องตรงกับ tags ที่ตั้งไว้ตอนสร้าง runner (`docker`, `demo`) ทุกตัว ไม่เช่นนั้น job จะค้าง `pending` ตามที่อธิบายไว้ใน Step 508

### ขั้นตอนที่ 6: Commit และ Push

```bash
git add .gitlab-ci.yml
git commit -m "ทดสอบรัน pipeline บน self-hosted runner"
git push
```

### ขั้นตอนที่ 7: ตรวจสอบผลลัพธ์

1. เข้าไปที่แท็บ **CI/CD > Pipelines** ของ project จะเห็น pipeline ใหม่กำลังรัน
2. คลิกเข้าไปดู job `hello-from-my-runner` — สถานะควรเปลี่ยนจาก `pending` เป็น `running` แล้วเป็น `passed` ในเวลาไม่นาน
3. เปิด log ของ job จะเห็นบรรทัดประมาณ `Running with gitlab-runner ... on docker-runner-demo (...)` ซึ่งยืนยันว่า job นี้ถูกรันโดย runner ที่เราเพิ่งติดตั้งและลงทะเบียนเอง **ไม่ใช่ shared runner ของ GitLab**
4. เห็นผลลัพธ์จากคำสั่ง `uname -a` และ `cat /etc/os-release` แสดงข้อมูลของระบบปฏิบัติการภายใน container ที่ runner สร้างขึ้นมาให้

### (ทางเลือก) ขั้นตอนทำความสะอาดหลังฝึกเสร็จ

ถ้าไม่ต้องการเก็บ runner นี้ไว้ต่อ สามารถลบออกได้ทั้งสองฝั่ง:

```bash
# ยกเลิกการลงทะเบียนจากฝั่ง runner
docker run --rm -it \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest unregister --name "docker-runner-demo"

# หยุดและลบ container ของ gitlab-runner
docker stop gitlab-runner
docker rm gitlab-runner
```

หรือจะเข้าไปกด **Delete runner** จากหน้า **Settings > CI/CD > Runners** บนเว็บก็ได้เช่นกัน

### Checklist แบบฝึกหัด

- [ ] สร้าง project runner ผ่านหน้าเว็บสำเร็จ พร้อมได้ Runner authentication token
- [ ] รัน gitlab-runner เป็น container ด้วย Docker สำเร็จ
- [ ] ลงทะเบียน runner ด้วย `gitlab-runner register --non-interactive` สำเร็จ
- [ ] เห็นสถานะ runner เป็น **Online** บนหน้าเว็บ
- [ ] เขียน `.gitlab-ci.yml` ที่มี `tags:` ตรงกับ runner ที่สร้าง
- [ ] Push แล้ว pipeline รันจนสถานะเป็น `passed`
- [ ] เปิด job log แล้วยืนยันได้ว่า job รันบน runner ของตัวเอง ไม่ใช่ shared runner

---

## สรุป Part 51

ใน Part นี้เราได้เรียนรู้ว่า:

1. **GitLab Runner** คือ agent ที่ทำหน้าที่รับงานจาก GitLab instance (ซึ่งเป็นแค่ coordinator) มารันจริงผ่าน **executor** — GitLab เองไม่ได้รันโค้ดของเรา
2. **Shared/Instance Runner** คือ runner ที่ GitLab จัดหาให้ใช้ร่วมกันพร้อมโควตานาทีต่อเดือน ส่วน **Specific/Self-hosted Runner** คือ runner ที่เราติดตั้งดูแลเองและผูกกับ project/group ที่กำหนด
3. การติดตั้ง Runner ทำได้หลายวิธี (package, binary, Docker, Kubernetes) และมี **executor** ให้เลือกหลายแบบ — **shell**, **docker**, **kubernetes**, **docker-machine** (deprecated) — โดย Docker executor เป็นตัวเลือกที่นิยมและสมดุลที่สุด
4. การลงทะเบียน runner ยุคใหม่ใช้ **Runner authentication token** (`glrt-...`) ที่สร้างจาก UI ก่อน แทนที่ **Registration token** แบบเก่าที่ถูก deprecate ไปด้วยเหตุผลด้านความปลอดภัย
5. **Tags** คือกลไกจับคู่ job กับ runner — job จะรันได้เฉพาะบน runner ที่มี tag ครบตามที่ job ระบุเท่านั้น
6. Docker executor ใช้ `image:` กำหนด container หลักของ job และ `services:` เพิ่ม container เสริมอย่าง database ที่เชื่อมต่อผ่าน hostname เดียวกับชื่อ image
7. `concurrent`/`limit` ควบคุมจำนวนงานที่รันพร้อมกันได้ ส่วน autoscaling ในอดีตใช้ docker-machine (ปัจจุบันเลิกพัฒนาแล้ว) และแนวทางใหม่คือ GitLab Runner Autoscaler ผ่าน Fleeting plugin
8. เมื่อ pipeline ค้าง ให้ไล่ตรวจตามลำดับ: runner online ไหม → tags ตรงกันไหม → shared runner ถูกปิดไหม → protected runner กับ branch ตรงกันไหม → concurrency เต็มไหม → network/TLS มีปัญหาไหม
9. Self-hosted runner ต้องระวังเรื่องความปลอดภัยเป็นพิเศษ โดยเฉพาะ job จาก forked merge request — ใช้ **Protected Runner** คู่กับ **Protected Variables** และแยก pool ของ runner ตามระดับความน่าเชื่อถือ
10. ลงมือติดตั้ง ลงทะเบียน และรัน pipeline จริงบน Runner ของตัวเองผ่าน Docker executor จนสำเร็จแล้ว

**ต่อไป:** [Part 52: GitLab Pages และ Container Registry](./part-052-gitlab-pages-container-registry.md)
