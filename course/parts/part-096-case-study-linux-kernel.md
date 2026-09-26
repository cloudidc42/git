# Part 96: Case Study: Linux Kernel Git Workflow

> **Step ในหลักสูตรนี้:** Step 951–960
> **เฟส:** 10 — ระดับโลก / Leadership (Part นี้คือ Part แรกของเฟสสุดท้ายในหลักสูตรทั้งหมด)
> **เป้าหมายของ Part นี้:** เข้าสู่เฟสสุดท้ายของหลักสูตรด้วยการศึกษา **กรณีศึกษาระดับโลกจริง** เริ่มต้นที่โปรเจกต์ที่สำคัญที่สุดในประวัติศาสตร์ของ Git นั่นคือ **Linux Kernel** — โปรเจกต์ที่ Git ถูกสร้างขึ้นมาเพื่อมันโดยตรงตั้งแต่วันแรก เราจะเจาะลึก Workflow จริงที่ Linux Kernel ใช้ในการพัฒนามาตลอด 30 กว่าปี ตั้งแต่โครงสร้าง Maintainer แบบลำดับชั้น การส่ง patch ทาง mailing list แทนการใช้ Pull Request บน GitHub คำสั่ง `git format-patch` และ `git am` ที่เป็นหัวใจของ workflow นี้ แนวคิด `Signed-off-by` และ Developer Certificate of Origin กลไก merge window และ release cycle แบบ rc1–rc7 ไปจนถึง linux-next tree ที่ใช้ทดสอบการรวมโค้ดจากหลายสาย ก่อนจะปิดท้ายด้วยบทเรียนที่นำไปประยุกต์ใช้กับโปรเจกต์ทั่วไปได้แม้จะไม่ได้ใช้ mailing list ก็ตาม

---

## เกริ่นก่อนเข้าเฟส 10: ระดับโลก / Leadership

ตลอด 9 เฟสที่ผ่านมา เราเดินทางจากศูนย์ — เรียนรู้ว่า Version Control คืออะไร ฝึกใช้คำสั่งพื้นฐาน ทำงานเป็นทีมด้วย Workflow มาตรฐาน เจาะลึก Git Internals ทำ CI/CD เต็มรูปแบบ จัดการ DevOps และ Security ระดับองค์กร ไปจนถึงฝึกทักษะมืออาชีพอย่างการเป็น Maintainer, Release Management และการรับมือ Incident วิกฤต

เฟส 10 ซึ่งเป็นเฟสสุดท้ายของหลักสูตรนี้ (Step 951–1000) จะพาคุณออกจากการฝึกฝนในโปรเจกต์ขนาดเล็กหรือขนาดกลาง ไปสู่การมอง **โปรเจกต์ระดับโลกจริง** ที่มีผู้เกี่ยวข้องนับพันนับหมื่นคน เพื่อดูว่าเมื่อ Git ถูกใช้งานในสเกลที่ใหญ่ที่สุดเท่าที่จะเป็นไปได้ ผู้คนต้องปรับ Workflow และวินัยการทำงานอย่างไรบ้าง สิ่งที่เราจะได้เห็นในเฟสนี้ไม่ใช่แค่ "คำสั่ง Git ขั้นสูงกว่าเดิม" แต่คือ **แนวคิดเชิงกระบวนการและวัฒนธรรมการทำงาน** ที่ทำให้โปรเจกต์ขนาดมหึมาเหล่านี้เดินหน้าต่อไปได้อย่างมีระเบียบ

Part แรกของเฟสนี้ต้องเป็น **Linux Kernel** อย่างไม่มีข้อโต้แย้ง เพราะเหตุผลง่าย ๆ คือ — ถ้าไม่มี Linux Kernel ก็จะไม่มี Git เลยด้วยซ้ำ ย้อนกลับไปที่ **Part 01 Step 4** เราได้เล่าไว้แล้วว่า Linus Torvalds สร้าง Git ขึ้นมาในปี 2005 เพราะปัญหาลิขสิทธิ์กับ BitKeeper และตั้งเป้าหมายการออกแบบไว้ชัดเจนเพื่อรองรับการพัฒนา Linux Kernel โดยเฉพาะ — เร็ว กระจายศูนย์เต็มรูปแบบ รองรับ non-linear development และมีความสมบูรณ์ของข้อมูลสูง ดังนั้นการทำความเข้าใจว่า Linux Kernel ใช้ Git อย่างไรในปัจจุบัน จึงเปรียบเหมือนการย้อนกลับไปดู "ต้นแบบดั้งเดิม" ที่ทุกอย่างเริ่มต้นมาจากที่นั่น

---

## สารบัญของ Part นี้

- Step 951: ย้อนกลับไปจุดเริ่มต้น — Linux Kernel คือโปรเจกต์ที่ Git ถูกสร้างมาเพื่อมันโดยตรง
- Step 952: ขนาดของ Linux Kernel ในปัจจุบัน — หลายล้านบรรทัดโค้ด หลายพัน contributor ต่อ release cycle
- Step 953: Hierarchical Maintainer Model — จาก Linus Torvalds ลงไปถึง maintainer ย่อยของแต่ละ driver/module
- Step 954: Mailing List Workflow แทน Pull Request — เหตุผลที่ Linux Kernel ไม่ใช้ GitHub PR
- Step 955: `git format-patch` และ `git am` — คำสั่งหัวใจของการส่ง/รับ patch ทาง email
- Step 956: `Signed-off-by` Trailer และ Developer Certificate of Origin (DCO)
- Step 957: Merge Window และ Release Cycle — จาก rc1 ถึง rc7 ก่อนออก stable release
- Step 958: linux-next Tree คืออะไร — สนามทดสอบก่อนเข้า mainline จริง
- Step 959: บทเรียนจาก Linux Kernel ที่นำไปใช้กับโปรเจกต์ทั่วไปได้ (แม้ไม่ใช้ mailing list)
- Step 960: แบบฝึกหัด — จำลอง Workflow แบบ Linux Kernel ด้วย `git format-patch` และ `git am`

---

## Step 951: ย้อนกลับไปจุดเริ่มต้น — Linux Kernel คือโปรเจกต์ที่ Git ถูกสร้างมาเพื่อมันโดยตรง

ใน Part 01 Step 4 เราเล่าไว้ว่า Git เกิดขึ้นในปี 2005 หลังจากทีมพัฒนา Linux Kernel เผชิญวิกฤตครั้งใหญ่: เครื่องมือ VCS เชิงพาณิชย์ที่ใช้อยู่ในตอนนั้นคือ **BitKeeper** ถูกยกเลิกสิทธิ์การใช้งานฟรีสำหรับชุมชน Open Source อย่างกะทันหัน (จากข้อพิพาทระหว่าง Andrew Tridgell ผู้พยายาม reverse-engineer โปรโตคอลของ BitKeeper กับบริษัท BitMover เจ้าของผลิตภัณฑ์) Linus Torvalds ซึ่งขณะนั้นดูแลโปรเจกต์ Linux Kernel มากว่า 14 ปี ต้องเผชิญคำถามใหญ่: จะหาเครื่องมืออะไรมาแทนที่การจัดการซอร์สโค้ดของ kernel ที่มีขนาดมหาศาลและมีนักพัฒนากระจายอยู่ทั่วโลก

สิ่งสำคัญที่ต้องเข้าใจให้ชัดคือ Git **ไม่ได้ถูกออกแบบมาเป็นเครื่องมือทั่วไปแล้วบังเอิญมี Linux Kernel มาใช้** แต่ตรงกันข้าม — Git ถูกออกแบบมา **โดยมี Linux Kernel เป็นโจทย์ตั้งต้นเพียงโจทย์เดียว** Linus เขียน Git เวอร์ชันแรกภายในเวลาประมาณ 10 วัน โดยตั้งข้อกำหนดที่มาจากปัญหาจริงของการพัฒนา kernel โดยตรง:

1. **ต้องเร็วมากในการ diff และ merge** เพราะ kernel มีไฟล์นับหมื่นไฟล์ และมี branch การพัฒนาจำนวนมากที่ต้องรวมกันตลอดเวลา
2. **ต้องรองรับ workflow แบบกระจายศูนย์เต็มรูปแบบ (fully distributed)** เพราะนักพัฒนา kernel ไม่ได้มี "สิทธิ์เขียน" เข้า repository กลางเดียวกันทุกคน แต่ทำงานผ่านการส่ง patch ให้ maintainer หลายชั้นตรวจสอบก่อน
3. **ต้องรองรับ non-linear history** เพราะ kernel มีสาขาการพัฒนา (subsystem) แยกกันหลายสิบสายที่ทำงานคู่ขนานและมารวมกันเป็นระยะ
4. **ต้องมีกลไกตรวจสอบความสมบูรณ์ของข้อมูลในระดับ cryptographic** เพราะโค้ดของ kernel เป็นโครงสร้างพื้นฐานสำคัญของโลก การถูกแก้ไขโดยไม่มีใครรู้ตัวคือความเสี่ยงด้านความปลอดภัยระดับสูงสุด

ด้วยเหตุนี้ **ทุกฟีเจอร์หลักของ Git ที่เราเรียนมาตลอดหลักสูตรนี้ — snapshot-based storage, cheap branching, distributed model, SHA hash integrity — ล้วนมีร่องรอยที่ย้อนกลับไปหาความต้องการเฉพาะของการพัฒนา Linux Kernel ทั้งสิ้น** เมื่อ Git เติบโตขึ้นและถูกนำไปใช้กับโปรเจกต์อื่น ๆ หลายสิบล้านโปรเจกต์ทั่วโลก ผ่าน GitHub, GitLab, Bitbucket และแพลตฟอร์มอื่น ๆ Workflow ที่คนส่วนใหญ่คุ้นเคย (fork → branch → Pull Request → merge ผ่านหน้าเว็บ) กลับเป็น **workflow ที่ Linux Kernel เองแทบไม่ได้ใช้เลย**

นี่คือความย้อนแย้งที่น่าสนใจที่สุดของ Part นี้: โปรเจกต์ที่ให้กำเนิด Git กลับไม่ได้ใช้วิธีทำงานแบบที่คนส่วนใหญ่รู้จัก Git ผ่านมันในปัจจุบัน Linux Kernel ยังคงยึดมั่นกับ workflow ดั้งเดิมที่ใกล้เคียงกับสิ่งที่ Git ถูกออกแบบมาให้ทำตั้งแต่ต้นมากที่สุด นั่นคือการทำงานผ่าน **mailing list และ email patch** ซึ่งเราจะเจาะลึกใน Step ถัดไป

ก่อนไปต่อ ลองมาดูภาพรวมขนาดของโปรเจกต์นี้ในปัจจุบันกันก่อน เพื่อให้เห็นว่าความท้าทายที่ Git ต้องรับมือมันใหญ่ขนาดไหนจริง ๆ

---

## Step 952: ขนาดของ Linux Kernel ในปัจจุบัน

Linux Kernel ไม่ใช่แค่โปรเจกต์ Open Source ธรรมดา แต่เป็นหนึ่งในซอฟต์แวร์ที่มีขนาดใหญ่และมีผู้เกี่ยวข้องมากที่สุดในประวัติศาสตร์วิศวกรรมซอฟต์แวร์ ตัวเลขต่อไปนี้ (อ้างอิงจากลักษณะของโปรเจกต์ในช่วงหลายปีที่ผ่านมาและรายงาน Linux Kernel Development Report ที่ Linux Foundation เผยแพร่เป็นระยะ) ช่วยให้เห็นภาพว่า Git ต้องรับมือกับสเกลระดับใด:

### ขนาดของซอร์สโค้ด

- ซอร์สโค้ดของ kernel มีขนาด **มากกว่า 30 ล้านบรรทัด** เมื่อรวม driver, architecture support, และ subsystem ทั้งหมด
- จำนวนไฟล์ในต้นไม้ของ source code มีมากกว่า **80,000 ไฟล์**
- repository เพียงอย่างเดียว (`.git` directory) เมื่อ clone แบบเต็มประวัติ มีขนาดหลาย gigabyte เพราะเก็บประวัติการเปลี่ยนแปลงย้อนหลังไปนับล้าน commit

### จำนวนคนที่เกี่ยวข้อง

- ในแต่ละ **release cycle** (ซึ่งกินเวลาประมาณ 9–10 สัปดาห์ต่อรอบ) มีนักพัฒนา **มากกว่า 1,500–2,000 คน** ที่ส่ง patch เข้ามาอย่างสม่ำเสมอ
- จำนวน commit ต่อ release cycle หนึ่งรอบมักอยู่ที่ประมาณ **15,000–18,000 commit**
- นับตั้งแต่เริ่มใช้ Git ในปี 2005 เป็นต้นมา จำนวน commit สะสมทั้งหมดในประวัติของ kernel มีมากกว่า **1.3 ล้าน commit**
- นักพัฒนาที่เคยส่ง patch เข้า Linux Kernel ตลอดประวัติศาสตร์ของโปรเจกต์มีมากกว่า **20,000 คน** จากทั่วโลก
- บริษัทที่ส่งพนักงานเข้ามาร่วมพัฒนา kernel อย่างต่อเนื่องมีทั้งบริษัทขนาดใหญ่ระดับโลก เช่น Intel, Google, Red Hat, AMD, ARM, Huawei, IBM, Samsung, Meta และอีกหลายสิบบริษัท — kernel ไม่ใช่โปรเจกต์อาสาสมัครล้วน ๆ แต่เป็นความร่วมมือระดับอุตสาหกรรม

### ความเร็วในการเปลี่ยนแปลง

- โดยเฉลี่ยแล้วมี commit ใหม่เข้าสู่ kernel ในอัตรา **ประมาณ 8–10 commit ต่อชั่วโมง** ตลอด 24 ชั่วโมงทุกวัน หากคิดเป็นค่าเฉลี่ยตลอดทั้งปี
- Linux Kernel ออก major release ใหม่ทุก **9–10 สัปดาห์** อย่างสม่ำเสมอมาตลอดกว่า 20 ปี — นี่คือความสม่ำเสมอที่หาได้ยากมากในโปรเจกต์ซอฟต์แวร์ขนาดใหญ่

ตัวเลขเหล่านี้อธิบายได้ว่าทำไม Linux Kernel ถึงไม่สามารถใช้ workflow แบบ "ทุกคน fork แล้วเปิด Pull Request บนเว็บเดียวกัน" ได้ — เพราะการมี maintainer เพียงกลุ่มเดียวมานั่งไล่ตรวจสอบ Pull Request นับพันรายการต่อสัปดาห์บนหน้าเว็บเดียวเป็นเรื่องที่จัดการไม่ไหวในทางปฏิบัติ สิ่งที่ Linux Kernel ใช้แทนคือโครงสร้างการกระจายอำนาจแบบลำดับชั้น ซึ่งเราจะดูรายละเอียดใน Step ถัดไป

---

## Step 953: Hierarchical Maintainer Model ของ Linux

หัวใจของการที่ Linux Kernel สามารถรับมือกับ contributor นับพันคนได้โดยไม่ล้มเหลว คือโครงสร้าง **Maintainer แบบลำดับชั้น (Hierarchical Maintainer Model)** ซึ่งกระจายความรับผิดชอบในการตรวจสอบและ merge โค้ดออกเป็นหลายระดับ แทนที่จะกระจุกอยู่ที่คนเดียว

### โครงสร้างแบบพีระมิด

```
                    Linus Torvalds
                  (Final integrator)
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
   Subsystem          Subsystem          Subsystem
   Maintainer         Maintainer         Maintainer
   (เช่น Networking)  (เช่น Memory Mgmt) (เช่น Filesystems)
        │                 │                 │
   ┌────┼────┐       ┌────┼────┐       ┌────┼────┐
   │    │    │       │    │    │       │    │    │
 Driver Driver Driver ...                  ...
 Maintainer (ระดับย่อยลงไปอีก เช่น driver การ์ดจอเฉพาะยี่ห้อ)
        │
   Individual Contributor (ผู้ส่ง patch)
```

### บทบาทแต่ละระดับ

**1. Linus Torvalds (Top-level Maintainer)**

Linus ทำหน้าที่เป็น **final integrator** ของ kernel ทั้งหมด เขาไม่ได้อ่าน patch ทุกบรรทัดที่เข้ามาในโปรเจกต์ (เป็นไปไม่ได้ในทางปฏิบัติ) แต่เขาจะ **pull** การเปลี่ยนแปลงจาก subsystem maintainer ระดับบนสุดจำนวนหลายสิบคน โดยเชื่อใจในกระบวนการตรวจสอบที่เกิดขึ้นในแต่ละชั้นก่อนหน้าแล้ว งานหลักของ Linus คือการตัดสินใจว่าจะ merge อะไรเข้าไปใน mainline ในแต่ละรอบ release และเป็นผู้ตัดสินใจสุดท้ายในกรณีที่เกิดข้อขัดแย้งระหว่าง subsystem

**2. Subsystem Maintainer**

Kernel ถูกแบ่งออกเป็น **subsystem** จำนวนมาก เช่น Networking, Memory Management, Filesystems (ext4, btrfs, XFS), Device Drivers (แยกย่อยตามประเภทฮาร์ดแวร์), Architecture-specific code (x86, ARM, RISC-V), Security, Scheduler เป็นต้น แต่ละ subsystem มี **maintainer ประจำ** ที่รับผิดชอบตรวจสอบ patch ทั้งหมดที่เกี่ยวกับพื้นที่ของตัวเอง รายชื่อ maintainer และขอบเขตความรับผิดชอบของแต่ละคนถูกบันทึกไว้อย่างเป็นทางการในไฟล์ **`MAINTAINERS`** ที่อยู่ใน root ของ source tree ของ kernel เอง ไฟล์นี้ระบุชัดเจนว่าไฟล์หรือโฟลเดอร์ใดอยู่ในความดูแลของใคร ควรส่ง patch ไปที่ mailing list ไหน และควร CC ใครบ้าง

**3. Driver/Module Maintainer (ระดับย่อยลงไปอีก)**

ในหลาย subsystem ที่มีขนาดใหญ่มาก เช่น Networking หรือ Device Drivers ยังมีการแบ่งย่อยลงไปอีกเป็นระดับที่ 3 เช่น driver ของการ์ดเครือข่ายยี่ห้อหนึ่งอาจมี maintainer เฉพาะของตัวเอง ที่รับผิดชอบเฉพาะไฟล์ในโฟลเดอร์นั้น ๆ

**4. Individual Contributor**

นักพัฒนาทั่วไปที่ต้องการส่งการแก้ไขเข้า kernel จะเริ่มต้นจากการส่ง patch ไปยัง maintainer ที่ดูแลไฟล์นั้น ๆ (และ mailing list ที่เกี่ยวข้อง) **ไม่ใช่ส่งตรงไปหา Linus**

### ทำไม Model นี้ถึงสำคัญ

โครงสร้างนี้ทำให้การตรวจสอบโค้ดถูกกระจายออกไปตามความเชี่ยวชาญ — คนที่ตรวจสอบ patch เกี่ยวกับ network driver คือคนที่เข้าใจ subsystem นั้นลึกที่สุด ไม่ใช่ Linus ที่ต้องดูแลภาพรวมทั้งหมด ผลลัพธ์คือ Linus จะเห็นแค่ **pull request ระดับ subsystem** (ไม่ใช่ patch ระดับบุคคลนับพันรายการ) ทำให้เขาสามารถบริหารจัดการโปรเจกต์ขนาดมหาศาลนี้ได้โดยไม่ต้องอ่านทุกบรรทัดโค้ดด้วยตัวเอง

ในเชิงเทคนิค การ "pull" ในที่นี้หมายถึงคำสั่ง `git pull` ที่ดึงจาก Git tree (repository) ของ subsystem maintainer แต่ละคนโดยตรง ซึ่งแต่ละคนมี tree ของตัวเองที่ host แยกกัน (มักจะอยู่บน kernel.org) ไม่ใช่การ merge ผ่านหน้าเว็บใด ๆ ทั้งสิ้น — นี่คือจุดที่นำไปสู่หัวข้อถัดไป: ทำไม Linux Kernel ถึงไม่ใช้ Pull Request แบบที่เรารู้จักบน GitHub

---

## Step 954: Mailing List Workflow แทน Pull Request

หนึ่งในสิ่งที่ทำให้มือใหม่ที่คุ้นเคยกับ GitHub รู้สึกแปลกใจที่สุดเมื่อได้ศึกษา Linux Kernel workflow คือ **kernel แทบไม่ใช้ Pull Request ผ่านหน้าเว็บเลย** แม้ว่า Linux Kernel เองจะมีการ mirror repository ไว้บน GitHub (ที่ `github.com/torvalds/linux`) แต่ mirror นั้นเป็น **read-only** และทางโปรเจกต์ระบุชัดเจนว่า **ไม่รับ Pull Request ที่ส่งผ่าน GitHub**

### วิธีที่ Linux Kernel ทำงานจริง: Patch ทาง Email

แทนที่จะเปิด Pull Request บนเว็บ นักพัฒนา Linux Kernel จะ:

1. เขียนโค้ดแก้ไขในเครื่องตัวเองโดยใช้ Git ตามปกติ (commit ทีละเรื่อง แยกเป็น logical change เล็ก ๆ)
2. ใช้คำสั่ง `git format-patch` เพื่อแปลง commit เหล่านั้นให้เป็นไฟล์ patch ในรูปแบบที่ส่งทาง email ได้
3. ส่งไฟล์ patch เหล่านั้นไปยัง **mailing list ที่เกี่ยวข้อง** ผ่าน `git send-email` หรือโปรแกรม email อื่น ๆ (LKML คือ mailing list หลักของ Linux Kernel และมี mailing list ย่อยของแต่ละ subsystem อีกมากมาย เช่น `linux-fsdevel`, `netdev`, `linux-mm`)
4. Maintainer และคนอื่น ๆ ในชุมชน **รีวิว patch ผ่านการตอบกลับ email แบบ inline** (comment ใต้บรรทัดโค้ดที่เกี่ยวข้องโดยตรงในเนื้อหา email แบบ quote-reply)
5. ผู้ส่ง patch แก้ไขตามคำแนะนำ แล้วส่ง **เวอร์ชันใหม่ (v2, v3, ...)** กลับเข้า mailing list อีกครั้ง จนกว่า maintainer จะพอใจ
6. เมื่อ patch ผ่านการรีวิวแล้ว maintainer จะใช้คำสั่ง `git am` เพื่อนำ patch นั้นเข้าไปใน Git tree ของตัวเอง

### เหตุผลเชิงประวัติศาสตร์และเชิงปฏิบัติ

**เหตุผลที่ 1 — ประวัติศาสตร์:** Linux Kernel เริ่มพัฒนามาตั้งแต่ปี 1991 ก่อนที่ GitHub จะถือกำเนิดขึ้นมาถึง 17 ปี (GitHub เปิดตัวปี 2008) วัฒนธรรมการทำงานผ่าน mailing list เป็นมาตรฐานของโลก Open Source ยุคนั้นอยู่แล้ว (เช่นเดียวกับโปรเจกต์ GNU และ Apache) เมื่อวัฒนธรรมนี้ฝังรากลึกและใช้งานได้ดีมาหลายทศวรรษ การเปลี่ยนไปใช้ระบบใหม่ทั้งหมดจึงไม่มีความจำเป็น

**เหตุผลที่ 2 — ไม่ผูกติดกับแพลตฟอร์มใดแพลตฟอร์มหนึ่ง:** Email เป็นโปรโตคอลเปิดที่ทำงานได้ทุกที่ ไม่ต้องพึ่งพาบริการของบริษัทใดบริษัทหนึ่ง (ต่างจาก GitHub ซึ่งเป็นของ Microsoft) การพึ่งพาระบบที่บริษัทเดียวควบคุมทั้งหมดถือเป็นความเสี่ยงเชิงกลยุทธ์สำหรับโครงสร้างพื้นฐานระดับโลกอย่าง kernel

**เหตุผลที่ 3 — ประสิทธิภาพสำหรับ maintainer ที่ต้องจัดการ patch จำนวนมหาศาล:** Maintainer ระดับสูงของ kernel ต้องจัดการ patch หลายร้อยฉบับต่อสัปดาห์ การใช้ email client ที่มีระบบ filter, scripting และ automation ที่ยืดหยุ่นสูง (เช่นการเขียน script กรอง patch อัตโนมัติตาม pattern) มีประสิทธิภาพกว่าการคลิกผ่านหน้าเว็บทีละรายการมาก

**เหตุผลที่ 4 — Patch เป็นหน่วยที่ตรวจสอบได้ละเอียดกว่า:** ระบบ patch ทาง email เอื้อให้แยกการเปลี่ยนแปลงเป็น **patch series** ที่มีลำดับชัดเจน แต่ละ patch ทำเรื่องเดียวและมีคำอธิบายของตัวเอง ทำให้รีวิวทีละขั้นตอนได้ง่ายกว่าการดู diff ก้อนใหญ่ก้อนเดียวแบบใน Pull Request บางกรณี

**เหตุผลที่ 5 — Offline-first และเบามาก:** การรีวิวผ่าน plain-text email ใช้ bandwidth น้อยมาก ทำงานได้แม้ในพื้นที่ที่อินเทอร์เน็ตไม่เสถียร ซึ่งสำคัญมากเพราะ contributor ของ kernel กระจายอยู่ทั่วโลกรวมถึงในประเทศที่โครงสร้างพื้นฐานอินเทอร์เน็ตยังไม่ดีนัก

### เปรียบเทียบ Mailing List Workflow กับ GitHub Pull Request Workflow

| แง่มุม | Mailing List (Linux Kernel) | Pull Request (GitHub/GitLab ทั่วไป) |
|---|---|---|
| ที่เก็บการสนทนารีวิว | Email thread บน mailing list (archive สาธารณะแบบ plain text) | Comment thread บนหน้าเว็บของแพลตฟอร์ม |
| หน่วยที่รีวิว | Patch series ทีละ commit เรียงลำดับ | มักดู diff รวมทั้ง branch พร้อมกัน (แม้จะดูทีละ commit ได้) |
| การอนุมัติ | Maintainer ตอบ `Acked-by:` / `Reviewed-by:` เป็น trailer ใน commit | ปุ่ม "Approve" บนหน้าเว็บ |
| เครื่องมือที่ต้องใช้ | Email client ที่รองรับ plain text + `git send-email` | เบราว์เซอร์ + บัญชีบนแพลตฟอร์ม |
| การพึ่งพาแพลตฟอร์ม | ไม่พึ่งพาบริษัทใดบริษัทหนึ่ง (email เป็นมาตรฐานเปิด) | ผูกกับผู้ให้บริการนั้น ๆ |
| ความเหมาะสมกับสเกล | เหมาะกับ contributor นับพันคนกระจายทั่วโลก | เหมาะกับทีมขนาดเล็กถึงกลางที่ใช้แพลตฟอร์มเดียวกัน |

ตารางนี้ไม่ได้มีไว้เพื่อบอกว่าแบบใดดีกว่ากันโดยรวม แต่ชี้ให้เห็นว่าแต่ละ workflow ถูกออกแบบมาให้เหมาะกับบริบทที่ต่างกัน Pull Request บนเว็บเหมาะกับทีมที่รวมศูนย์อยู่บนแพลตฟอร์มเดียว ในขณะที่ mailing list เหมาะกับเครือข่ายผู้พัฒนาที่กระจายตัวสุดขั้วและไม่มีใครเป็นเจ้าของแพลตฟอร์มกลาง

### ข้อควรรู้: ไม่ได้แปลว่า Pull Request ใน Git ไม่มีประโยชน์

สิ่งสำคัญที่ต้องเข้าใจให้ถูกต้องคือ Linux Kernel ไม่ได้ปฏิเสธแนวคิด "pull request" ในความหมายทั่วไปของ Git (คือการขอให้อีกฝ่าย pull การเปลี่ยนแปลงจาก tree ของตน) — จริง ๆ แล้ว subsystem maintainer ก็ส่งคำขอแบบนี้ให้ Linus อยู่เสมอ โดยใช้คำสั่ง `git request-pull` เพื่อสร้างข้อความสรุปที่ระบุ URL ของ Git tree, branch, และรายการ commit ที่ขอให้ pull เข้าไป แล้วส่งข้อความนั้นทาง email — นี่คือ "pull request" ในความหมายดั้งเดิมที่ Git ตั้งชื่อคำสั่งไว้ตรงตัว เพียงแต่ไม่ได้ทำผ่านหน้าเว็บของแพลตฟอร์มใด ๆ เท่านั้นเอง

---

## Step 955: `git format-patch` และ `git am`

สองคำสั่งนี้คือหัวใจทางเทคนิคของ workflow แบบ mailing list ทั้งหมด มาดูรายละเอียดการทำงานของแต่ละคำสั่ง

### `git format-patch` — แปลง commit ให้เป็นไฟล์ patch ที่ส่งทาง email ได้

คำสั่งนี้จะแปลง commit หนึ่งตัวหรือหลายตัวให้กลายเป็นไฟล์ text ในรูปแบบมาตรฐานที่เรียกว่า **"mbox format"** ซึ่งเป็นฟอร์แมตเดียวกับที่ email client เก่าแก่ใช้เก็บข้อความ

ตัวอย่างการใช้งาน:

```bash
# สร้าง patch จาก commit ล่าสุด 3 ตัว บน branch ปัจจุบัน
git format-patch -3

# สร้าง patch จากทุก commit ที่มีอยู่บน branch ปัจจุบันแต่ไม่มีใน main
git format-patch main

# สร้าง patch พร้อม cover letter อธิบายภาพรวมของ patch series ทั้งชุด
git format-patch --cover-letter -3 -o outgoing/
```

ผลลัพธ์ที่ได้คือไฟล์ในลักษณะนี้ (ตัวอย่างคร่าว ๆ):

```
0001-fix-null-pointer-dereference-in-driver-probe.patch
0002-add-missing-error-handling-in-init-path.patch
0003-update-documentation-for-new-sysfs-attribute.patch
```

แต่ละไฟล์มีโครงสร้างสำคัญคือ:

```
From <commit-hash> Mon Sep 17 00:00:00 2001
From: ชื่อผู้เขียน <email@example.com>
Date: <วันที่ commit>
Subject: [PATCH 1/3] fix: null pointer dereference in driver probe

รายละเอียดคำอธิบายของการแก้ไข (commit message body)

Signed-off-by: ชื่อผู้เขียน <email@example.com>
---
 drivers/example/driver.c | 5 +++--
 1 file changed, 3 insertions(+), 2 deletions(-)

diff --git a/drivers/example/driver.c b/drivers/example/driver.c
...
```

จะเห็นว่าไฟล์นี้ประกอบด้วย **metadata ของ commit ครบถ้วน** (ผู้เขียน วันที่ ข้อความอธิบาย) บวกกับ **diff ของโค้ด** ในไฟล์เดียว ทำให้สามารถส่งเป็นเนื้อหา email ได้โดยตรง และผู้รับสามารถนำกลับมาเป็น commit ที่มีข้อมูลครบถ้วนเหมือนต้นฉบับทุกประการ

การส่งจริงมักใช้คำสั่ง `git send-email` ต่อจากนั้น ซึ่งจะอ่านไฟล์ patch เหล่านี้และส่งออกไปเป็น email จริงผ่าน SMTP server ที่ตั้งค่าไว้ โดยรักษารูปแบบให้ mailing list และ maintainer ปลายทางสามารถประมวลผลได้อย่างถูกต้อง (patch series ที่มีหลายไฟล์จะถูกส่งเป็น email แบบ threaded คือ reply ต่อกันเป็นชุดเดียว)

### `git am` — นำ patch จาก email กลับมาเป็น commit

เมื่อ maintainer ได้รับ patch ทาง email (ไม่ว่าจะ save เป็นไฟล์ mbox หรือดึงมาจาก mail client) จะใช้คำสั่ง `git am` (ย่อมาจาก "apply mailbox") เพื่อแปลงไฟล์ patch นั้นกลับเป็น commit บน repository ของตัวเอง โดยยังคง **ผู้เขียนดั้งเดิม วันที่ดั้งเดิม และข้อความ commit ดั้งเดิม** ไว้ครบถ้วน — นี่คือจุดสำคัญมาก เพราะแปลว่า maintainer ที่รับ patch **ไม่ได้กลายเป็นผู้เขียน commit นั้น** เครดิตยังคงเป็นของผู้เขียนตัวจริงเสมอ

ตัวอย่างการใช้งาน:

```bash
# apply patch ไฟล์เดียว
git am 0001-fix-null-pointer-dereference-in-driver-probe.patch

# apply ทั้ง patch series ในโฟลเดอร์เดียว (เรียงตามชื่อไฟล์)
git am incoming/*.patch

# ถ้า apply แล้วเกิด conflict สามารถแก้ conflict แล้วสั่งให้ทำต่อได้
git am --continue

# ถ้าต้องการยกเลิกกระบวนการ apply ทั้งหมด กลับไปสถานะก่อนเริ่ม
git am --abort
```

Workflow ที่สมบูรณ์ในทางปฏิบัติจึงเป็นวงจรแบบนี้:

```
[Contributor เขียนโค้ด] → git commit → git format-patch → git send-email
        │
        ▼
[Mailing List] → maintainer/community รีวิวผ่าน reply email
        │
        ▼ (ถ้าต้องแก้ไข ส่งกลับไปให้ contributor แก้แล้วส่ง v2 ใหม่)
        │
        ▼ (เมื่อผ่านการรีวิวแล้ว)
[Maintainer] → บันทึก patch เป็นไฟล์ → git am → commit เข้า tree ของตัวเอง
        │
        ▼
[Subsystem Maintainer] → git request-pull → ส่งให้ Linus
        │
        ▼
[Linus] → git pull → merge เข้า mainline
```

คำสั่งทั้งสองนี้ทำให้ email กลายเป็น **transport layer ของ Git commit ที่สมบูรณ์แบบ** ไม่ใช่แค่การส่งไฟล์ diff ธรรมดา — นี่คือเหตุผลที่ workflow นี้ยังคงใช้งานได้ดีมาตลอดหลายสิบปีแม้เทคโนโลยีจะเปลี่ยนไปมากแล้วก็ตาม

### เครื่องมือเสริมในยุคหลัง: `b4`

ในช่วงหลายปีหลัง ชุมชน Linux Kernel ได้พัฒนาเครื่องมือเสริมชื่อ **`b4`** ขึ้นมาเพื่อลดความยุ่งยากของการทำงานกับ patch ทาง email โดย `b4` ช่วยให้ maintainer สามารถดึง patch series ทั้งชุดจาก mailing list archive (เช่นจากบริการ `lore.kernel.org` ซึ่งเป็น archive กลางของ mailing list ต่าง ๆ ของ kernel) มาแปลงเป็นไฟล์ mbox พร้อม apply ด้วย `git am` ได้ในคำสั่งเดียว ทั้งยังตรวจสอบ `Signed-off-by`, trailer อื่น ๆ (เช่น `Reviewed-by`, `Tested-by`) และสถานะการรีวิวล่าสุดของแต่ละเวอร์ชันของ patch series ให้โดยอัตโนมัติ เครื่องมือนี้สะท้อนให้เห็นว่าแม้ workflow หลักจะยังเป็น email แบบดั้งเดิม แต่ชุมชนก็ไม่หยุดพัฒนาเครื่องมือรอบข้างให้ทำงานได้สะดวกขึ้นเรื่อย ๆ

---

## Step 956: `Signed-off-by` Trailer และ Developer Certificate of Origin (DCO)

เมื่อดูตัวอย่างไฟล์ patch ใน Step ก่อนหน้า จะสังเกตเห็นบรรทัด `Signed-off-by: ชื่อผู้เขียน <email@example.com>` ปรากฏอยู่ท้าย commit message เสมอ — บรรทัดนี้ไม่ใช่แค่ธรรมเนียมปฏิบัติทั่วไป แต่เป็น **ข้อกำหนดบังคับ (mandatory requirement)** สำหรับทุก patch ที่จะถูกรับเข้า Linux Kernel

### Signed-off-by คืออะไร

`Signed-off-by` เป็น **trailer** (บรรทัดพิเศษท้าย commit message ที่มีรูปแบบ `Key: Value`) ที่ผู้เขียน patch ต้องใส่ในทุก commit เพื่อยืนยันว่าตนเองมีสิทธิ์ที่จะส่งโค้ดนี้เข้าโปรเจกต์ ในทางเทคนิค นักพัฒนาสามารถเพิ่ม trailer นี้ได้ง่าย ๆ ด้วยการใช้ flag `-s` (หรือ `--signoff`) ตอน commit:

```bash
git commit -s -m "fix: null pointer dereference in driver probe"
```

ซึ่งจะเพิ่มบรรทัด `Signed-off-by: <ชื่อและอีเมลตาม git config ของผู้ commit>` ต่อท้าย commit message โดยอัตโนมัติ

### Developer Certificate of Origin (DCO)

การใส่ `Signed-off-by` มีความหมายทางกฎหมายที่ชัดเจน โดยอ้างอิงจากเอกสารที่เรียกว่า **Developer Certificate of Origin (DCO)** ซึ่งถูกสร้างขึ้นโดยชุมชน Linux Kernel เองในปี 2004 (ก่อน Git จะถือกำเนิดด้วยซ้ำ) เพื่อตอบสนองต่อข้อกังวลด้านทรัพย์สินทางปัญญาในกรณีพิพาท SCO v. IBM ที่เกิดขึ้นในช่วงเวลานั้น

เนื้อหาของ DCO โดยสรุปคือ เมื่อนักพัฒนาใส่ `Signed-off-by` ในบรรทัดหนึ่งของ commit นั้น เท่ากับว่าเขากำลังรับรองอย่างเป็นทางการว่า:

1. เขาเป็นผู้เขียนโค้ดนี้ทั้งหมดหรือบางส่วนด้วยตนเอง และมีสิทธิ์ที่จะส่งมันภายใต้ license ของโปรเจกต์ (สำหรับ kernel คือ GPLv2) หรือ
2. โค้ดนี้มาจากงานก่อนหน้าที่อยู่ภายใต้ license ที่เหมาะสมอยู่แล้ว และเขามีสิทธิ์ที่จะส่งมันต่อภายใต้เงื่อนไขเดียวกัน หรือ
3. โค้ดนี้ถูกส่งมาจากบุคคลอื่นที่ได้รับรองข้อ 1 หรือ 2 ไว้แล้ว และเขาไม่ได้แก้ไขมัน
4. เขาเข้าใจว่า patch นี้และข้อมูล `Signed-off-by` (รวมถึงข้อมูลส่วนตัวที่แนบมา เช่น ชื่อและอีเมล) จะถูกเก็บบันทึกไว้ถาวรและอาจถูกส่งต่อสาธารณะ พร้อมกับ patch นี้เอง

พูดง่าย ๆ คือ `Signed-off-by` เป็น **การยืนยันความเป็นเจ้าของและสิทธิ์ทางกฎหมายของโค้ด** แบบไม่ต้องเซ็นเอกสารทางกฎหมายแยกต่างหาก (ต่างจากโปรเจกต์ Open Source บางแห่งที่ใช้ Contributor License Agreement หรือ CLA ที่ต้องเซ็นเอกสารแยก) นี่คือกลไกที่เบาแต่ทรงพลัง เพราะฝังอยู่ใน commit history โดยตรง ตรวจสอบย้อนหลังได้ตลอดไป และไม่ต้องมีกระบวนการทางกฎหมายแยกต่างหากที่ยุ่งยาก

### Maintainer ปฏิเสธ patch ที่ไม่มี Signed-off-by

Maintainer ของ Linux Kernel จะ **ปฏิเสธ patch ที่ไม่มีบรรทัด `Signed-off-by`** โดยอัตโนมัติ ไม่ว่าคุณภาพของโค้ดจะดีแค่ไหนก็ตาม เพราะขาดการยืนยันทางกฎหมายที่จำเป็น นี่คือเหตุผลที่นักพัฒนา kernel ทุกคนต้องคุ้นเคยกับ flag `-s` ในคำสั่ง `git commit` ตั้งแต่ patch แรกที่ส่งเข้าโปรเจกต์

### ข้อสังเกต: ไม่ใช่แค่ Linux Kernel

แนวคิด DCO ที่เริ่มต้นจาก Linux Kernel ได้ถูกนำไปใช้อย่างแพร่หลายในโปรเจกต์ Open Source อื่น ๆ อีกจำนวนมากในปัจจุบัน เช่นบาง repository บน GitHub ตั้งค่า **DCO bot** ที่จะตรวจสอบอัตโนมัติว่าทุก commit ใน Pull Request มี `Signed-off-by` ครบถ้วนหรือไม่ก่อนอนุญาตให้ merge — เป็นตัวอย่างที่ดีว่าแนวคิดจาก workflow ของ Linux Kernel แพร่กระจายไปสู่วงการ Open Source ในวงกว้างได้อย่างไร แม้โปรเจกต์เหล่านั้นจะใช้ GitHub Pull Request ตามปกติ ไม่ได้ใช้ mailing list เหมือน kernel เลยก็ตาม

---

## Step 957: Merge Window และ Release Cycle ของ Linux

Linux Kernel มีวินัยเรื่องตารางเวลาการออก release ที่เข้มงวดและสม่ำเสมออย่างน่าทึ่ง โดยดำเนินมาในรูปแบบเดียวกันมานานกว่า 20 ปี รูปแบบนี้เรียกว่า **time-based release model**

### โครงสร้างของ Release Cycle หนึ่งรอบ

Release cycle หนึ่งรอบของ kernel ใช้เวลาประมาณ **9–10 สัปดาห์** แบ่งออกเป็น 2 ช่วงหลัก:

**ช่วงที่ 1 — Merge Window (ประมาณ 2 สัปดาห์)**

ทันทีที่ kernel เวอร์ชันก่อนหน้าถูกปล่อยออกมาเป็น stable release (เช่น เมื่อ 6.7 ถูกปล่อยออกมา) **merge window** ของเวอร์ชันถัดไป (6.8) จะเปิดขึ้นทันที ในช่วงนี้ Linus จะรับ pull request จาก subsystem maintainer ทั้งหมด เพื่อดึงฟีเจอร์ใหม่ ๆ ที่ **พัฒนาและทดสอบมาล่วงหน้าแล้ว** (ไม่ใช่เริ่มเขียนตอนนี้) เข้าสู่ mainline การเปลี่ยนแปลงเชิงใหญ่ ฟีเจอร์ใหม่ driver ใหม่ ส่วนใหญ่จะถูกรวมเข้ามาในช่วงสองสัปดาห์นี้เท่านั้น

**ช่วงที่ 2 — Stabilization Period ผ่าน Release Candidate (rc1–rc7 หรือมากกว่า)**

หลังจาก merge window ปิดลง Linus จะออก **rc1 (release candidate 1)** ซึ่งหมายถึงการ "ปิดประตู" ไม่ให้มีฟีเจอร์ใหม่เข้ามาอีก (กฎนี้เรียกว่า **"no new features after rc1"**) จากจุดนี้เป็นต้นไป การเปลี่ยนแปลงที่ยอมรับได้จะจำกัดอยู่แค่:

- การแก้ไขบั๊ก (bug fixes)
- การแก้ไขปัญหาด้านความปลอดภัย (security fixes)
- เอกสารประกอบ (documentation)
- driver ใหม่ที่ **ไม่กระทบโค้ดส่วนอื่น** ในกรณีพิเศษบางกรณี

ทุกสัปดาห์ Linus จะออก release candidate ใหม่ตามลำดับ: rc1 → rc2 → rc3 → ... โดยทั่วไปมักอยู่ที่ **rc7** ก่อนจะปล่อยเป็น stable release แต่บางครั้งถ้ายังพบบั๊กร้ายแรงอยู่ ก็อาจขยายไปถึง rc8 หรือมากกว่านั้นได้ Linus เป็นผู้ตัดสินใจโดยพิจารณาจากปริมาณและความรุนแรงของบั๊กที่ยังพบในแต่ละสัปดาห์

### ทำไม Time-based Release ถึงสำคัญ

รูปแบบนี้แตกต่างจาก **feature-based release** (ที่ปล่อยเมื่อฟีเจอร์ที่ต้องการเสร็จสมบูรณ์ ไม่ว่าจะใช้เวลานานแค่ไหน) ข้อดีของ time-based release สำหรับโปรเจกต์ขนาดนี้คือ:

1. **คาดการณ์ได้** — ทุกคนในระบบนิเวศ (distro maker, hardware vendor, enterprise user) รู้ล่วงหน้าว่า kernel เวอร์ชันถัดไปจะออกเมื่อไหร่ วางแผนงานของตัวเองตามได้
2. **ลดแรงกดดันในการยัดเยียดฟีเจอร์ที่ยังไม่พร้อม** — ถ้าฟีเจอร์ไม่ทันรอบนี้ ก็รอรอบถัดไปได้ ไม่ต้องรีบเร่งจนโค้ดคุณภาพต่ำ
3. **จำกัดขอบเขตความเสี่ยงในแต่ละรอบ** — เพราะ merge window สั้นแค่ 2 สัปดาห์ ปริมาณการเปลี่ยนแปลงใหญ่ ๆ ที่เข้ามาในแต่ละรอบจึงถูกจำกัดและตรวจสอบได้ง่ายกว่า

### รูปแบบการตั้งชื่อเวอร์ชัน

Linux Kernel ใช้รูปแบบการตั้งชื่อเวอร์ชันที่เรียบง่ายและสอดคล้องกับ Git tag โดยตรง เช่น `v6.8-rc1`, `v6.8-rc2`, ... จนถึง `v6.8` (stable release) ตัวเลขตัวแรก (major) จะขยับขึ้นเป็นระยะตามดุลยพินิจของ Linus โดยไม่ได้ผูกกับความหมายเชิง Semantic Versioning แบบที่โปรเจกต์ทั่วไปคุ้นเคย (major.minor.patch ที่สื่อถึง breaking change) — Linus เคยอธิบายไว้หลายครั้งว่าตัวเลข version ของ kernel ไม่ได้บอกอะไรเกี่ยวกับขนาดของการเปลี่ยนแปลงเลย เป็นเพียงตัวนับที่เดินหน้าไปเรื่อย ๆ ตามรอบเวลาเท่านั้น จุดนี้เป็นความแตกต่างที่น่าสนใจเมื่อเทียบกับหลักการ Semantic Versioning ที่หลักสูตรนี้เคยพูดถึงใน Part 88

### Stable Kernel และ Long-Term Support (LTS)

หลังจากออก stable release แล้ว (เช่น 6.8) ทีมงาน **stable kernel maintainer** (นำโดย Greg Kroah-Hartman เป็นหลักในช่วงหลายปีที่ผ่านมา) จะยังคง backport การแก้ไขบั๊กและช่องโหว่ความปลอดภัยที่สำคัญกลับไปยังเวอร์ชันนั้นต่อไปอีกระยะหนึ่ง บาง release ถูกเลือกให้เป็น **LTS (Long-Term Support)** ซึ่งจะได้รับการดูแลนานถึงหลายปี (บางเวอร์ชันนานถึง 6 ปี) เพื่อรองรับอุปกรณ์และระบบที่ต้องการความเสถียรระยะยาว เช่น อุปกรณ์ embedded หรือระบบในอุตสาหกรรม

---

## Step 958: linux-next Tree คืออะไร

ด้วยจำนวน subsystem หลายสิบสายที่พัฒนาคู่ขนานกันตลอดเวลา คำถามสำคัญคือ: จะรู้ได้อย่างไรว่าเมื่อรวม subsystem ทั้งหมดเข้าด้วยกันในช่วง merge window แล้วจะไม่เกิดปัญหาขัดแย้งกัน (เช่น สอง subsystem แก้ไข API เดียวกันในทางที่ขัดแย้งกัน) นี่คือปัญหาที่ **linux-next** ถูกสร้างขึ้นมาเพื่อแก้โดยเฉพาะ

### linux-next คืออะไร

**linux-next** คือ Git tree พิเศษที่ดูแลโดย Stephen Rothwell (ผู้ริเริ่มและดูแลมาตั้งแต่ปี 2008) ทำหน้าที่เป็น **"สนามซ้อมรวม" (integration testing tree)** ที่รวบรวม tree ของ subsystem maintainer เกือบทั้งหมด (มากกว่า 200 tree ย่อย) มา **merge รวมกันทุกวัน** ก่อนที่การเปลี่ยนแปลงเหล่านั้นจะถูกส่งเข้า mainline จริงในรอบ merge window ถัดไป

### กระบวนการทำงานของ linux-next

```
Subsystem Tree 1 (networking)      ┐
Subsystem Tree 2 (memory mgmt)     │
Subsystem Tree 3 (filesystems)     ├──▶ linux-next (รวมทุกวัน)
Subsystem Tree 4 (drivers)         │         │
... (มากกว่า 200 tree)             ┘         ▼
                                    Build + Boot Test อัตโนมัติ
                                    บนหลาย architecture
                                              │
                                              ▼
                                   รายงานปัญหากลับไปยัง
                                   subsystem maintainer ที่เกี่ยวข้อง
                                              │
                                              ▼
                          (เมื่อ merge window เปิด) Linus pull
                          จาก subsystem tree จริง เข้า mainline
```

ทุกวัน linux-next จะถูก build ใหม่ทั้งหมดโดยดึงจาก tree ของทุก subsystem ที่เข้าร่วม แล้วนำไปทดสอบ compile และ boot บนหลายสถาปัตยกรรม (x86, ARM, และอื่น ๆ) ด้วยระบบทดสอบอัตโนมัติ ถ้าพบว่าการรวม tree สองสายทำให้เกิด conflict หรือ build พัง ปัญหานั้นจะถูกตรวจพบ **ก่อน** ที่มันจะไปถึง Linus ในช่วง merge window จริง ทำให้ subsystem maintainer ที่เกี่ยวข้องมีเวลาแก้ไขล่วงหน้า

### ทำไม linux-next ถึงจำเป็น

ถ้าไม่มี linux-next ปัญหาการรวม subsystem ที่ขัดแย้งกันจะถูกค้นพบก็ต่อเมื่อ Linus พยายาม merge จริงในช่วง merge window ซึ่งเป็นช่วงเวลาที่กดดันที่สุดและสั้นที่สุด (แค่ 2 สัปดาห์) การพบปัญหาช้าขนาดนั้นจะทำให้กระบวนการทั้งหมดล่าช้าอย่างรุนแรง linux-next จึงทำหน้าที่เป็น **early warning system** ที่ทำให้ปัญหาการรวมโค้ดถูกพบล่วงหน้าหลายสัปดาห์ก่อนที่มันจะกลายเป็นปัญหาจริงในรอบ release

นอกจากนี้ linux-next ยังเป็นที่ที่นักพัฒนาและผู้ทดสอบ (tester) ที่ต้องการทดลองฟีเจอร์ล่าสุดสุด ๆ ก่อนใครสามารถ clone มาใช้งานได้ แม้จะมีความเสี่ยงว่าอาจไม่เสถียรเท่า mainline หรือ stable release ก็ตาม

### ความคล้ายคลึงกับแนวคิดสมัยใหม่

แนวคิดของ linux-next นั้นคล้ายกับสิ่งที่ทีมพัฒนาซอฟต์แวร์สมัยใหม่เรียกว่า **integration branch** หรือ **staging environment ระดับ source code** — คือมีจุดกลางจุดหนึ่งที่รวมงานจากหลายทีมมาทดสอบร่วมกันก่อนที่จะปล่อยจริง เพียงแต่ Linux Kernel ทำสิ่งนี้ในสเกลที่ใหญ่กว่ามาก (มากกว่า 200 สายพร้อมกัน) และทำมันมานานกว่าที่คำว่า "CI/CD" จะเป็นที่นิยมในวงการเสียอีก

---

## Step 959: บทเรียนจาก Linux Kernel ที่นำไปประยุกต์ใช้กับโปรเจกต์ทั่วไปได้

โปรเจกต์ของคุณอาจไม่มีนักพัฒนาหลายพันคน ไม่มี mailing list และไม่จำเป็นต้องมี maintainer หลายสิบชั้นแบบ Linux Kernel แต่หลักการเบื้องหลัง workflow ของ kernel มีหลายอย่างที่นำไปปรับใช้ได้จริงในโปรเจกต์ขนาดใดก็ตาม แม้จะไม่ได้ใช้ email เลยก็ตาม

### บทเรียนที่ 1: แยกความรับผิดชอบเป็นชั้น (Ownership ตามพื้นที่)

แนวคิด `MAINTAINERS` file ของ kernel ที่ระบุว่าใครดูแลไฟล์หรือโฟลเดอร์ไหน สามารถนำมาใช้ในโปรเจกต์ทั่วไปได้ทันทีผ่านกลไกที่ GitHub และ GitLab มีให้อยู่แล้ว เช่นไฟล์ **`CODEOWNERS`** ซึ่งกำหนดว่าเมื่อมีการแก้ไขไฟล์ในโฟลเดอร์หนึ่ง ระบบจะขอ review อัตโนมัติจากคนหรือทีมที่รับผิดชอบพื้นที่นั้นโดยเฉพาะ แทนที่จะให้ maintainer คนเดียวต้องรีวิวทุกอย่าง

### บทเรียนที่ 2: แยก Commit เป็นหน่วยเล็กที่ตรวจสอบได้ (Atomic, Logical Commits)

Workflow ของ kernel บังคับให้ผู้ส่ง patch แยกการเปลี่ยนแปลงเป็น commit เล็ก ๆ ที่แต่ละ commit ทำเรื่องเดียวและอธิบายตัวเองได้ครบถ้วน (self-contained) นี่คือวินัยที่มีคุณค่ามากในทุกโปรเจกต์ ไม่ว่าจะใช้ Pull Request หรือไม่ก็ตาม — commit ที่เล็กและมีจุดประสงค์ชัดเจนทำให้ code review ง่ายขึ้นมาก ทำให้ `git bisect` มีประสิทธิภาพมากขึ้น และทำให้การ revert เฉพาะจุดทำได้โดยไม่กระทบส่วนอื่น

### บทเรียนที่ 3: Signed-off-by / DCO เป็นทางเลือกที่เบากว่า CLA

โปรเจกต์ Open Source ที่ต้องการความชัดเจนทางกฎหมายเกี่ยวกับสิทธิ์ในโค้ดที่ contributor ส่งมา ไม่จำเป็นต้องสร้างกระบวนการ Contributor License Agreement ที่ซับซ้อนเสมอไป การใช้ DCO ผ่าน `Signed-off-by` ที่ฝังอยู่ใน commit โดยตรง (และมี bot ตรวจสอบอัตโนมัติบน Pull Request) เป็นวิธีที่เบากว่ามากและได้ผลลัพธ์ทางกฎหมายที่เพียงพอสำหรับหลายกรณี

### บทเรียนที่ 4: กำหนดกฎ "ปิดรับฟีเจอร์ใหม่" ก่อน Release

หลักการ "no new features after rc1" ของ kernel คือแนวคิดของ **feature freeze** ซึ่งทีมพัฒนาซอฟต์แวร์ทั่วไปนำไปใช้ได้เช่นกัน — กำหนดจุดตัดที่ชัดเจนก่อนวันปล่อย release แต่ละครั้งว่าจะไม่รับฟีเจอร์ใหม่เข้ามาอีก โดยยอมรับเฉพาะ bug fix หลังจากจุดนั้น ช่วยลดความเสี่ยงที่จะเกิดบั๊กใหม่ในนาทีสุดท้ายก่อนปล่อยจริง

### บทเรียนที่ 5: มี Integration Branch ก่อนรวมงานจากหลายทีมจริง

แนวคิด linux-next สามารถแปลงมาเป็น **integration branch** หรือ **nightly build pipeline** ในโปรเจกต์ทั่วไปได้ โดยตั้ง branch หรือ pipeline ที่ merge งานจากหลาย feature branch มาทดสอบร่วมกันโดยอัตโนมัติทุกวัน ก่อนที่จะถูก merge เข้า main จริง เพื่อจับปัญหาการรวมโค้ดที่ขัดแย้งกันล่วงหน้า แนวคิดนี้คือรากฐานของสิ่งที่ทีมสมัยใหม่เรียกว่า **Continuous Integration** นั่นเอง

### บทเรียนที่ 6: Time-based Release สร้างความคาดการณ์ได้

การกำหนดตารางเวลาปล่อย release ที่แน่นอนและสม่ำเสมอ (เช่น ทุก 2 สัปดาห์ หรือทุกเดือน) แทนที่จะปล่อยเมื่อ "ฟีเจอร์เสร็จ" ช่วยให้ทั้งทีมพัฒนาและผู้ใช้งานวางแผนงานได้ดีขึ้นมาก และลดแรงกดดันที่จะยัดเยียดฟีเจอร์ที่ยังไม่พร้อมเข้าไปในรอบเดียว

### บทเรียนที่ 7: กระจายความไว้วางใจผ่านหลายชั้น ไม่ใช่รวมศูนย์ที่คนเดียว

โครงสร้างที่ Linus ไม่ต้องอ่าน patch ทุกฉบับด้วยตัวเอง แต่ไว้วางใจ subsystem maintainer ที่ผ่านการพิสูจน์ฝีมือมานาน คือบทเรียนสำคัญสำหรับทีมที่กำลังโตขึ้น — เมื่อทีมมีขนาดใหญ่จนหัวหน้าทีมคนเดียวไม่สามารถรีวิวทุกอย่างได้ทัน การมอบอำนาจตัดสินใจให้ senior engineer ที่เชี่ยวชาญเฉพาะด้าน (แทนที่จะพยายามควบคุมทุกอย่างจากศูนย์กลาง) คือทางออกที่ยั่งยืนกว่า และเป็นหัวข้อที่เราจะไปเจาะลึกต่อในเรื่อง Engineering Leadership ช่วงท้ายของเฟส 10 นี้

### บทเรียนที่ 8: `git format-patch` / `git am` ยังมีประโยชน์แม้ไม่ใช้ Mailing List

แม้โปรเจกต์ของคุณจะไม่ได้ทำงานผ่าน email แต่คำสั่งเหล่านี้ยังมีประโยชน์ในสถานการณ์เฉพาะ เช่น การส่งชุดการแก้ไขให้ทีมที่ไม่มีสิทธิ์เข้าถึง repository เดียวกัน (เช่น partner บริษัทภายนอก) การสำรอง commit ไว้เป็นไฟล์ก่อนทำ operation เสี่ยง หรือการย้ายชุด commit ข้าม repository ที่ไม่มีประวัติร่วมกัน — Step ถัดไปเราจะได้ลองใช้มันจริงในแบบฝึกหัด

---

## Step 960: แบบฝึกหัด — จำลอง Workflow แบบ Linux Kernel

มาลองสัมผัสประสบการณ์การทำงานแบบ mailing list ของ Linux Kernel ด้วยตัวเองในโปรเจกต์ทดลอง โดยจำลองบทบาทสองฝ่าย: **Contributor** (ผู้ส่ง patch) และ **Maintainer** (ผู้รับและ apply patch)

### เตรียมสภาพแวดล้อม

สร้างโฟลเดอร์ฝึกฝนสองโฟลเดอร์แยกกัน โฟลเดอร์หนึ่งแทน repository ของ maintainer อีกโฟลเดอร์หนึ่งแทน clone ของ contributor:

```bash
mkdir -p ~/git-course/part-96-linux-workflow
cd ~/git-course/part-96-linux-workflow

# สร้าง "maintainer repo" จำลอง
git init maintainer-repo
cd maintainer-repo
echo "# mini-kernel" > README.md
mkdir -p drivers
echo "int probe(void) { return 0; }" > drivers/example.c
git add .
git commit -m "initial commit: add mini-kernel skeleton"
cd ..

# contributor clone จาก maintainer repo
git clone maintainer-repo contributor-repo
```

### ขั้นตอนที่ 1: Contributor เขียนโค้ดและสร้าง commit แบบมีวินัย

ในบทบาท contributor ให้เข้าไปที่ `contributor-repo` แล้วตั้งค่าชื่อ/อีเมลสำหรับฝึก (ถ้ายังไม่เคยตั้งในเครื่อง) จากนั้นสร้างการแก้ไข 2 เรื่องแยกกันเป็น 2 commit:

```bash
cd contributor-repo
git config user.name "Somchai Contributor"
git config user.email "somchai@example.com"

# แก้ไขเรื่องที่ 1: แก้บั๊ก null pointer
cat >> drivers/example.c << 'EOF'
/* fix: check pointer before use */
EOF
git add drivers/example.c
git commit -s -m "drivers: example: fix potential null pointer in probe

ตรวจสอบ pointer ก่อนใช้งานใน probe() เพื่อป้องกัน crash
ในกรณีที่ hardware ไม่ตอบสนองตามที่คาดไว้"

# แก้ไขเรื่องที่ 2: เพิ่มเอกสาร
echo "## Example driver usage" >> README.md
git add README.md
git commit -s -m "docs: add usage note for example driver"
```

สังเกตว่าใช้ flag `-s` ทุกครั้งเพื่อเพิ่ม `Signed-off-by` ตามธรรมเนียมของ kernel และแยกเป็น 2 commit เพราะเป็นคนละเรื่องกัน

### ขั้นตอนที่ 2: สร้าง Patch ด้วย `git format-patch`

```bash
git format-patch origin/master --cover-letter -o outgoing/
ls outgoing/
```

เปิดดูไฟล์ `0000-cover-letter.patch` แล้วแก้ไขให้มีคำอธิบายภาพรวมของ patch series (ในการใช้งานจริง cover letter จะอธิบายว่าชุด patch นี้ทำอะไรโดยรวม และทำไมถึงจำเป็น) จากนั้นลองเปิดดูไฟล์ `0001-...patch` และ `0002-...patch` เพื่อสังเกตโครงสร้าง metadata และ diff ที่อยู่ในไฟล์เดียวกัน

### ขั้นตอนที่ 3: จำลองการ "ส่งทาง email" — คัดลอกไฟล์ patch ไปยัง maintainer

ในสถานการณ์จริงขั้นตอนนี้คือการใช้ `git send-email` ส่งไฟล์ผ่าน SMTP แต่ในแบบฝึกหัดนี้เราจำลองด้วยการคัดลอกไฟล์ตรง ๆ:

```bash
cd ..
mkdir -p maintainer-repo/incoming
cp contributor-repo/outgoing/000{1,2}-*.patch maintainer-repo/incoming/
```

### ขั้นตอนที่ 4: บทบาท Maintainer — ตรวจสอบและ Apply ด้วย `git am`

```bash
cd maintainer-repo
git config user.name "Maintainer Team"
git config user.email "maintainer@example.com"

# ตรวจสอบเนื้อหา patch ก่อน apply เสมอ (เทียบเท่าการอ่านใน email)
cat incoming/0001-*.patch

# apply ทั้งชุดตามลำดับ
git am incoming/0001-*.patch incoming/0002-*.patch

# ตรวจสอบว่า commit ที่ apply เข้ามายังคงเครดิตผู้เขียนเดิม
git log --format="%h %an <%ae> - %s" -3
```

สังเกตผลลัพธ์ของ `git log` ให้ดี — แม้ commit เหล่านี้จะถูก apply โดย "Maintainer Team" แต่ `%an`/`%ae` (author name/email) ยังคงแสดงเป็น "Somchai Contributor" ตามต้นฉบับเสมอ นี่คือหลักฐานว่า `git am` รักษาความเป็นเจ้าของ commit ไว้ครบถ้วน ต่างจากการ copy-paste diff ด้วยมือที่จะทำให้ข้อมูลผู้เขียนเดิมหายไป

### ขั้นตอนที่ 5: ทดลองสถานการณ์ Conflict และการแก้ไข

ลองสร้างสถานการณ์ที่ maintainer มีการเปลี่ยนแปลงไฟล์เดียวกันไปแล้วก่อนที่ patch จะมาถึง เพื่อจำลอง conflict:

```bash
cd ../maintainer-repo
git commit --allow-empty -m "unrelated maintainer change"
echo "extra line from maintainer" >> drivers/example.c
git add drivers/example.c
git commit -m "drivers: example: unrelated maintainer edit"

# ลอง apply patch อีกชุดที่แก้ไฟล์เดียวกันในบรรทัดใกล้เคียง (สมมติมี incoming2/)
# ถ้าเกิด conflict จะเห็นข้อความแนะนำให้แก้ไขด้วยมือ
git am incoming/0003-*.patch 2>&1 || true

# ถ้า apply ไม่สำเร็จ ให้ตรวจดูสถานะ
git status

# แก้ conflict ในไฟล์ที่ขัดแย้งด้วยมือ แล้วสั่งต่อ
# git add <ไฟล์ที่แก้แล้ว>
# git am --continue

# หรือยกเลิกกระบวนการทั้งหมดกลับไปสถานะก่อนเริ่ม
# git am --abort
```

### ขั้นตอนที่ 6: ทำความเข้าใจ `git request-pull` (มุมมอง Subsystem Maintainer)

ลองจำลองว่าตัวเองเป็น subsystem maintainer ที่ต้องการขอให้อีกฝ่าย (สมมติว่าเป็น "Linus" ระดับบนสุด) pull งานจาก tree ของตัวเอง:

```bash
cd contributor-repo
git request-pull origin/master . HEAD
```

คำสั่งนี้จะพิมพ์ข้อความสรุปออกมา ประกอบด้วย URL ของ repository, branch, hash ของ commit ล่าสุด, และ diffstat สรุปว่ามีการเปลี่ยนแปลงกี่ไฟล์กี่บรรทัด — ข้อความนี้คือสิ่งที่ subsystem maintainer จริงจะ copy ไปวางในเนื้อหา email เพื่อขอให้ Linus pull เข้า mainline

### คำถามท้ายแบบฝึกหัด (ลองตอบด้วยตัวเองก่อนเปิดดูคำใบ้)

1. ทำไม `git am` ถึงสามารถรักษาผู้เขียนดั้งเดิมของ commit ไว้ได้ ทั้งที่คนที่รัน `git am` คือคนละคนกับผู้เขียน patch
2. ถ้า patch หนึ่งไม่มีบรรทัด `Signed-off-by` maintainer ของ Linux Kernel จริงจะทำอย่างไรกับ patch นั้น และทำไม
3. สมมติว่าคุณเป็น subsystem maintainer ที่มี patch 50 ฉบับรอ merge เข้า mainline ในรอบ merge window ที่เหลือเวลาอีกแค่ 3 วัน คุณจะใช้ประโยชน์จาก linux-next อย่างไรก่อนส่ง `git request-pull`
4. ในโปรเจกต์ของคุณเอง (ที่อาจใช้ GitHub Pull Request) มีส่วนไหนที่พอจะทำ "feature freeze" แบบ rc1 ของ kernel ได้บ้าง

**คำใบ้:** คำตอบข้อ 1 อยู่ในโครงสร้างของไฟล์ patch ที่ `git format-patch` สร้างขึ้น (ลองเปิดไฟล์ `.patch` ดูอีกครั้งแล้วสังเกตบรรทัด `From:` และ `Date:` ที่อยู่ด้านบนสุด) — นี่คือ metadata ที่ `git am` อ่านกลับมาใช้สร้าง commit ใหม่โดยตรง ไม่ได้ใช้ข้อมูลของคนที่รันคำสั่ง

### ขั้นตอนเสริม: ตรวจสอบผลลัพธ์ทั้งหมดด้วย `git log --format`

ปิดท้ายแบบฝึกหัดด้วยการตรวจสอบว่าทั้ง repository ของ contributor และ maintainer ยังคง history ที่สอดคล้องกัน แม้จะถูกสร้างผ่านคนละกระบวนการ (commit ปกติ กับ apply patch):

```bash
cd ../contributor-repo
git log --format="%h %an - %s"

cd ../maintainer-repo
git log --format="%h %an - %s"
```

สังเกตว่า hash ของ commit (`%h`) ระหว่างสอง repository อาจไม่ตรงกันเป๊ะ 100% แม้เนื้อหาจะเหมือนกัน เพราะ `git am` สร้าง commit ใหม่ในบริบทของ tree ปลายทาง (parent commit ต่างกัน) แต่ **เนื้อหาของโค้ด ผู้เขียน และข้อความ commit ยังคงเหมือนเดิมทุกประการ** — นี่คือหลักฐานเชิงประจักษ์ว่า patch-based workflow ถ่ายทอด "การเปลี่ยนแปลง" ได้ครบถ้วนโดยไม่ต้องมีประวัติ Git ร่วมกันมาก่อนเลยด้วยซ้ำ ซึ่งต่างจาก `git merge` หรือ `git cherry-pick` ที่มักต้องอาศัย object ร่วมกันในบาง operation

---

## สรุป Part 96

ใน Part นี้เราได้ศึกษากรณีศึกษาระดับโลกกรณีแรกของเฟส 10 คือ Linux Kernel ซึ่งเป็นโปรเจกต์ที่ Git ถือกำเนิดขึ้นมาเพื่อมันโดยตรง สรุปประเด็นสำคัญ:

1. Git ถูกสร้างขึ้นในปี 2005 โดยมี Linux Kernel เป็นโจทย์ตั้งต้นเพียงโจทย์เดียว ทุกคุณสมบัติหลักของ Git (snapshot, distributed, cheap branching, hash integrity) ล้วนมาจากความต้องการของ kernel โดยตรง
2. Linux Kernel ในปัจจุบันมีขนาดมากกว่า 30 ล้านบรรทัดโค้ด มีนักพัฒนามากกว่า 1,500–2,000 คนต่อ release cycle และมี commit สะสมมากกว่า 1.3 ล้าน commit
3. Linux Kernel ใช้ **Hierarchical Maintainer Model** กระจายความรับผิดชอบเป็นชั้น ๆ ตั้งแต่ Linus Torvalds ลงไปถึง subsystem maintainer และ driver maintainer ระดับย่อย
4. Linux Kernel ใช้ **Mailing List Workflow** แทน Pull Request บนเว็บ โดยส่ง patch ทาง email และรีวิวผ่านการ reply แบบ inline
5. `git format-patch` แปลง commit เป็นไฟล์ patch ที่ส่งทาง email ได้ และ `git am` นำ patch นั้นกลับมาเป็น commit พร้อมรักษาผู้เขียนดั้งเดิมไว้ครบถ้วน
6. `Signed-off-by` และ Developer Certificate of Origin (DCO) เป็นกลไกยืนยันสิทธิ์ทางกฎหมายของโค้ดที่เบากว่า CLA และฝังอยู่ใน commit history โดยตรง
7. Release cycle ของ kernel แบ่งเป็น merge window (2 สัปดาห์) และช่วง stabilization ผ่าน release candidate ตั้งแต่ rc1 ถึงประมาณ rc7 ก่อนปล่อย stable release ทุก 9–10 สัปดาห์
8. **linux-next** ทำหน้าที่เป็น integration tree ที่รวม tree จากกว่า 200 subsystem มาทดสอบร่วมกันทุกวัน เพื่อจับปัญหาการรวมโค้ดล่วงหน้าก่อนถึง merge window จริง
9. บทเรียนจาก Linux Kernel เช่น CODEOWNERS, atomic commits, DCO, feature freeze, integration branch และ time-based release สามารถนำไปประยุกต์ใช้กับโปรเจกต์ทั่วไปได้ทั้งหมด แม้จะไม่ได้ใช้ mailing list เลยก็ตาม
10. เราได้ลงมือฝึกจำลอง workflow แบบ Linux Kernel ด้วย `git format-patch`, `git send-email` (แนวคิด), `git am`, และ `git request-pull` ด้วยตัวเองแล้ว

Linux Kernel คือบทพิสูจน์ว่า Git ไม่ได้เป็นแค่เครื่องมือสำหรับทีมเล็ก ๆ แต่สามารถขยายสเกลไปรองรับโปรเจกต์ที่ใหญ่ที่สุดในโลกได้จริง ด้วยวินัยของกระบวนการที่ถูกออกแบบมาอย่างรอบคอบ ใน Part ถัดไปเราจะไปดูกรณีศึกษาจากอีกฟากหนึ่งของอุตสาหกรรม — บริษัทเทคโนโลยียักษ์ใหญ่อย่าง Google, Meta และ Microsoft ที่เลือกแนวทางตรงกันข้ามกับ Linux Kernel โดยสิ้นเชิง นั่นคือการรวมโค้ดทั้งบริษัทไว้ใน **Monorepo** ขนาดมหึมาเพียงที่เดียว

**ต่อไป:** [Part 97: Case Study: Google, Meta, Microsoft ใช้ Git/Monorepo อย่างไร](./part-097-case-study-google-meta-microsoft.md)
