# Part 36: CODEOWNERS และการกำหนดผู้รับผิดชอบโค้ด

> **Step ในหลักสูตรนี้:** Step 351–360
> **เฟส:** 4 — ทำงานเป็นทีมด้วย Workflow มาตรฐาน (Git Flow, GitHub Flow, Rebase, การควบคุมคุณภาพโค้ดระดับทีม)
> **เป้าหมายของ Part นี้:** เข้าใจว่าไฟล์ `CODEOWNERS` คืออะไร แก้ปัญหาอะไรให้ทีมที่โตขึ้น เขียน syntax ของมันได้ถูกต้องตามที่ GitHub กำหนดจริง เข้าใจ pattern matching, team-based ownership, order of precedence, การผูกกับ Branch Protection เพื่อบังคับ review จากเจ้าของโค้ด และสามารถออกแบบไฟล์ `CODEOWNERS` สำหรับ monorepo ที่มีหลายทีมดูแลคนละส่วนได้จริงในหน้างาน

---

## สารบัญของ Part นี้

- Step 351: CODEOWNERS คืออะไร แก้ปัญหาอะไร
- Step 352: ไฟล์ CODEOWNERS อยู่ที่ไหนได้บ้าง และ syntax พื้นฐาน
- Step 353: การกำหนดเจ้าของไฟล์/โฟลเดอร์เฉพาะด้วย Pattern Matching
- Step 354: Team-based Ownership เทียบกับ Individual Ownership
- Step 355: การบังคับ Required Review จาก Code Owner ผ่าน Branch Protection
- Step 356: Multiple Owners ในไฟล์เดียวกัน และ Order of Precedence
- Step 357: การทดสอบว่า CODEOWNERS ทำงานถูกต้อง
- Step 358: Best Practices การจัดโครงสร้าง CODEOWNERS สำหรับ Monorepo หลายทีม
- Step 359: ข้อจำกัดและปัญหาที่พบบ่อยของ CODEOWNERS
- Step 360: แบบฝึกหัด — สร้างไฟล์ CODEOWNERS สำหรับโปรเจกต์จำลอง 3 ทีม

---

## Step 351: CODEOWNERS คืออะไร แก้ปัญหาอะไร

### ปัญหาที่เกิดขึ้นจริงเมื่อทีมโตขึ้น

ลองนึกภาพทีมพัฒนาซอฟต์แวร์ที่มีขนาด 20-30 คน แบ่งงานกันดูแลคนละส่วนของโค้ด เช่น บางคนดูแล frontend บางคนดูแล backend บางคนดูแลระบบ infrastructure และบางคนดูแลเอกสารประกอบ เมื่อมีคนเปิด Pull Request ขึ้นมาสักอัน คำถามแรกที่เกิดขึ้นเสมอคือ:

> **"ควรขอ review จากใคร?"**

ในทีมเล็ก ๆ คำถามนี้ตอบง่าย เพราะทุกคนรู้จักกันหมดและรู้ว่าใครถนัดอะไร แต่พอทีมโตขึ้นถึงจุดหนึ่ง ปัญหาที่ตามมาจะเริ่มเกิดขึ้นซ้ำ ๆ:

1. **คนเปิด PR ไม่รู้ว่าควร assign reviewer คนไหน** — ต้องไปถามใน chat ทุกครั้งว่า "ใครดูแลไฟล์นี้", เสียเวลาและทำให้ PR ค้างอยู่โดยไม่มีใครรีวิว
2. **Reviewer สุ่ม ๆ ถูกดึงเข้ามารีวิวโค้ดที่ตัวเองไม่ถนัด** — ทำให้รีวิวได้ไม่ลึก มองข้ามปัญหาสำคัญ หรือใช้เวลานานเกินจำเป็นเพื่อทำความเข้าใจ context
3. **โค้ดสำคัญถูก merge โดยไม่มีคนที่เชี่ยวชาญตรวจสอบเลย** — เช่น ไฟล์ config ของ production, ไฟล์เกี่ยวกับ security, หรือ core library ที่ทุกทีมใช้ร่วมกัน ถูกแก้ไขโดยไม่มีเจ้าของจริง ๆ เห็นก่อน merge
4. **ความรับผิดชอบไม่ชัดเจน** — เมื่อเกิดบั๊กในโปรดักชัน ไม่มีใครรู้แน่ชัดว่า "ใครควรเป็นคนแรกที่ถูกตาม" เพราะไม่มีการบันทึกความเป็นเจ้าของไว้เป็นลายลักษณ์อักษรที่เครื่องมืออ่านได้
5. **การกระจายภาระงานรีวิวไม่สม่ำเสมอ** — บางคนถูกขอ review บ่อยเกินไปเพราะเป็นคนที่ทุกคนรู้จักและนึกถึงชื่อได้ก่อน ในขณะที่คนอื่นที่มีความรู้เท่ากันกลับไม่เคยถูกดึงเข้ามาเลย

### CODEOWNERS คือคำตอบของปัญหานี้

**CODEOWNERS** คือไฟล์พิเศษที่ GitHub (และ GitLab, Bitbucket ก็มีกลไกคล้ายกัน) กำหนดให้วางไว้ในตำแหน่งที่ระบบรู้จัก เพื่อ **"ประกาศ" ว่าไฟล์หรือโฟลเดอร์ส่วนไหนของ repository มีใครหรือทีมไหนเป็นเจ้าของ**

เมื่อมีการตั้งค่านี้ไว้ ระบบจะทำสิ่งเหล่านี้โดยอัตโนมัติ:

- เมื่อมีคนเปิด Pull Request ที่แก้ไขไฟล์ในส่วนที่มีเจ้าของกำหนดไว้ **GitHub จะ auto-request review จากเจ้าของไฟล์นั้นให้ทันที** โดยไม่ต้องมีใครมานั่งนึกเองว่าจะขอ review จากใคร
- ถ้าตั้งค่า Branch Protection Rule เพิ่มเติมแบบ "Require review from Code Owners" ระบบจะ **บังคับ** ว่า PR จะ merge ไม่ได้จนกว่าจะมีเจ้าของไฟล์อย่างน้อยหนึ่งคน approve ก่อน
- เกิดเป็น **เอกสารความรับผิดชอบที่เครื่องอ่านได้ (machine-readable ownership map)** ซึ่งไม่ใช่แค่ comment หรือ wiki ที่คนอาจลืมอัปเดต แต่เป็นไฟล์ที่ผูกเข้ากับ workflow การ review จริง ๆ

พูดให้ตรงประเด็นที่สุด:

> **CODEOWNERS แก้ปัญหา "ไม่รู้ว่าควรขอ review จากใคร" โดยเปลี่ยนคำถามนั้นให้เป็นกฎที่ระบบ enforce ให้อัตโนมัติ แทนที่จะต้องพึ่งความจำหรือการสื่อสารแบบ manual ของมนุษย์**

### ตัวอย่างสถานการณ์ก่อนและหลังมี CODEOWNERS

| สถานการณ์ | ก่อนมี CODEOWNERS | หลังมี CODEOWNERS |
|---|---|---|
| เปิด PR แก้ `payment-service/` | ต้องถามใน Slack ว่าใครดูแล payment | ระบบ auto-assign ทีม `@company/payments-team` ให้ทันที |
| แก้ไฟล์ `.github/workflows/deploy.yml` | ใครก็ merge ได้ถ้าไม่มี rule บังคับ | ต้องได้ approve จาก DevOps ก่อน merge เสมอ |
| Reviewer ใหม่เข้าทีม | ต้องจำเองว่าควรรีวิวโค้ดส่วนไหน | เห็นจากไฟล์ CODEOWNERS ได้ทันทีว่าใครดูแลอะไร |
| เกิดบั๊กใน production | ไล่ถามหาคนรับผิดชอบทีละคน | เปิดไฟล์ CODEOWNERS ดูได้เลยว่าโค้ดจุดนั้นใครเป็นเจ้าของ |

### CODEOWNERS ไม่ใช่แค่ "ของตกแต่ง" — มันคือส่วนหนึ่งของ Governance

หลายทีมมองว่า CODEOWNERS เป็นแค่ฟีเจอร์เสริมเล็ก ๆ แต่ในความเป็นจริง มันคือรากฐานสำคัญของ **Code Governance** ระดับองค์กร เพราะมันตอบคำถามพื้นฐาน 3 ข้อที่ทุกทีมวิศวกรรมต้องมีคำตอบ:

1. **ใครมีสิทธิ์ตัดสินใจเรื่องโค้ดส่วนนี้** (Decision Authority)
2. **ใครควรรู้ก่อนเมื่อมีการเปลี่ยนแปลงเกิดขึ้น** (Notification/Awareness)
3. **ใครควรรับผิดชอบเมื่อเกิดปัญหา** (Accountability)

เราจะเจาะลึกทั้ง 3 มุมนี้ตลอด Part นี้ พร้อมตัวอย่างไฟล์จริงที่ใช้งานได้ทันที

---

## Step 352: ไฟล์ CODEOWNERS อยู่ที่ไหนได้บ้าง และ syntax พื้นฐาน

### ตำแหน่งที่ GitHub ยอมรับ

GitHub จะมองหาไฟล์ชื่อ `CODEOWNERS` (ตัวพิมพ์ใหญ่ทั้งหมด ไม่มีนามสกุลไฟล์) ใน **3 ตำแหน่งเท่านั้น** เรียงตามลำดับที่ GitHub จะค้นหา:

```
1. .github/CODEOWNERS
2. CODEOWNERS               (ที่ root ของ repository)
3. docs/CODEOWNERS
```

**ข้อสำคัญที่ต้องจำ:** GitHub จะใช้ไฟล์แรกที่เจอตามลำดับด้านบนเท่านั้น ถ้ามีไฟล์ `CODEOWNERS` วางอยู่มากกว่า 1 ตำแหน่งพร้อมกัน (เช่น มีทั้งใน `.github/` และที่ root) **GitHub จะใช้เฉพาะไฟล์ใน `.github/CODEOWNERS` เท่านั้น และเพิกเฉยไฟล์อื่นโดยสมบูรณ์** ไม่ใช่การรวมกัน (merge) กันแต่อย่างใด

ในทางปฏิบัติ ทีมส่วนใหญ่นิยมวางไว้ที่ `.github/CODEOWNERS` เพราะ:

- สอดคล้องกับไฟล์ configuration อื่น ๆ ของ GitHub ที่มักอยู่ใน `.github/` เช่น `.github/workflows/`, `.github/ISSUE_TEMPLATE/`, `.github/PULL_REQUEST_TEMPLATE.md`
- ทำให้ root ของ repository ดูสะอาด ไม่รกด้วยไฟล์ configuration
- เป็น convention ที่ทีมส่วนใหญ่ในอุตสาหกรรมใช้กัน ทำให้คนที่ย้ายทีมมาใหม่คุ้นเคยได้ทันที

### Syntax พื้นฐานของไฟล์ CODEOWNERS

ไฟล์ CODEOWNERS มีรูปแบบเรียบง่ายมาก แต่ละบรรทัดประกอบด้วย 2 ส่วนหลัก:

```
<pattern>   <owner1> <owner2> <owner3> ...
```

ตัวอย่างไฟล์ CODEOWNERS แบบพื้นฐานที่สุด:

```gitattributes
# นี่คือ comment — ขึ้นต้นด้วย # เหมือน .gitignore
# บรรทัดว่างจะถูกข้าม ไม่มีผลอะไร

# กำหนดให้ @nan-dev เป็นเจ้าของทุกไฟล์ในโปรเจกต์ (default owner)
*       @nan-dev

# ไฟล์ JavaScript ทั้งหมดให้ @frontend-lead ดูแล
*.js    @frontend-lead

# โฟลเดอร์ backend ทั้งหมดให้ทีม backend ดูแล
/backend/   @company/backend-team
```

### กฎการเขียนพื้นฐาน

1. **แต่ละบรรทัดคือ 1 กฎ** — pattern ตามด้วยรายชื่อเจ้าของ คั่นด้วยช่องว่าง (space) หรือ tab อย่างน้อย 1 ตัว
2. **Comment ขึ้นต้นด้วย `#`** — ใช้แบ่งหมวดหมู่หรืออธิบายเหตุผลของกฎแต่ละข้อได้ (แนะนำให้ทำเสมอในไฟล์จริง)
3. **Owner ต้องขึ้นต้นด้วย `@`** เสมอ ไม่ว่าจะเป็น username, team หรือ email
4. **Owner สามารถระบุได้หลายคน/หลายทีมในบรรทัดเดียว** โดยคั่นด้วยช่องว่าง — เมื่อ PR แก้ไฟล์ที่ match pattern นั้น ทุกคนที่ระบุจะถูก request review พร้อมกันหมด
5. **ไม่จำเป็นต้องระบุ owner ก็ได้** — ถ้าเขียนแค่ pattern เฉย ๆ โดยไม่มีชื่อ owner ต่อท้าย จะหมายถึง "ไฟล์กลุ่มนี้ไม่มีเจ้าของ" ซึ่งมีประโยชน์เวลาต้องการ "ยกเว้น" ไฟล์บางกลุ่มออกจากกฎที่กว้างกว่าด้านบน (จะอธิบายละเอียดใน Step 356)

### รูปแบบของ Owner ที่รองรับ

| รูปแบบ | ตัวอย่าง | ความหมาย |
|---|---|---|
| GitHub username | `@nan-dev` | ผู้ใช้คนเดียวบน GitHub |
| GitHub team | `@company/backend-team` | ทีมทั้งทีมภายใน organization |
| Email address | `nan@company.com` | ต้องเป็นอีเมลที่ผูกกับบัญชี GitHub ของผู้ใช้แล้วเท่านั้น |

**ข้อควรระวัง:** การใช้ email จะใช้ได้ก็ต่อเมื่ออีเมลนั้นถูกตั้งเป็น **verified email** ของบัญชี GitHub ผู้ใช้แล้วเท่านั้น ถ้าอีเมลไม่ตรงกับบัญชีไหนเลย GitHub จะไม่สามารถ resolve ว่าเป็นใคร และจะไม่มีการ request review เกิดขึ้น (แบบเงียบ ๆ ไม่มี error แจ้งเตือน — เราจะพูดถึงข้อจำกัดแบบนี้อีกใน Step 359)

### ข้อกำหนดเรื่องสิทธิ์ (Permission Requirement)

มีข้อกำหนดสำคัญที่หลายคนพลาดบ่อย:

> **ผู้ที่จะถูกระบุเป็น Code Owner (ไม่ว่าจะเป็น user หรือสมาชิกใน team) จะต้องมีสิทธิ์ Write access (อย่างน้อย) ต่อ repository นั้นอยู่แล้ว มิเช่นนั้น GitHub จะไม่สามารถ auto-request review จากคนนั้นได้**

ถ้าคุณระบุ `@username` ที่ยังไม่มีสิทธิ์เข้าถึง repository เลย ระบบจะไม่ error ให้เห็นตรง ๆ แต่ auto-assign reviewer จะไม่เกิดขึ้นสำหรับคนนั้น ดังนั้นก่อนเขียน CODEOWNERS จริง ต้องแน่ใจว่าทุกคน/ทุกทีมที่ระบุไว้มีสิทธิ์เข้าถึง repository อย่างถูกต้องแล้ว

### ตัวอย่างไฟล์ CODEOWNERS ขนาดเล็กที่ใช้งานได้จริง

```gitattributes
# .github/CODEOWNERS
#
# กฎ default: ถ้าไม่มี pattern อื่นด้านล่าง match ก่อน ให้ tech lead เป็นผู้ตรวจสอบ
*                       @nan-dev

# เอกสารทั้งหมดให้ทีมเขียนเอกสารดูแล
*.md                    @company/docs-team

# ไฟล์ที่เกี่ยวกับ CI/CD ให้ DevOps ตรวจสอบเสมอ
/.github/workflows/     @company/devops-team
```

ไฟล์นี้แม้จะมีแค่ 3 กฎ แต่ก็เพียงพอที่จะทำให้ทุก PR ในโปรเจกต์มีคนถูก auto-request review แล้ว ใน Step ถัดไปเราจะมาเจาะลึกเรื่อง pattern matching ซึ่งเป็นหัวใจสำคัญที่สุดของไฟล์นี้

---

## Step 353: การกำหนดเจ้าของไฟล์/โฟลเดอร์เฉพาะด้วย Pattern Matching

### หลักการเดียวกับ `.gitignore` แต่มีรายละเอียดต่างกันเล็กน้อย

Pattern ในไฟล์ CODEOWNERS ใช้ syntax แบบเดียวกับที่คุณเคยเจอใน `.gitignore` (ที่เรียนไปแล้วใน Part 06) เกือบทั้งหมด แต่มีจุดต่างที่สำคัญมากที่ต้องระวัง คือ **ลำดับความสำคัญของกฎในไฟล์ CODEOWNERS จะตรงข้ามกับสัญชาตญาณที่คุ้นเคยจาก `.gitignore`** (รายละเอียดเรื่องนี้จะอธิบายเต็ม ๆ ใน Step 356) แต่สำหรับ Step นี้ เราจะโฟกัสที่ตัว pattern เพียงอย่างเดียวก่อน

### สัญลักษณ์พื้นฐานที่ใช้ได้

| สัญลักษณ์ | ความหมาย | ตัวอย่าง |
|---|---|---|
| `*` | ตัวอักษรใด ๆ ก็ได้ ยกเว้น `/` (ไม่ข้ามระดับโฟลเดอร์) | `*.js` = ไฟล์ `.js` ทุกไฟล์ในทุกระดับ |
| `**` | ตัวอักษรใด ๆ ก็ได้ รวมถึงข้ามหลายระดับโฟลเดอร์ | `docs/**` = ทุกไฟล์ภายใต้ `docs/` ไม่ว่าจะลึกแค่ไหน |
| `/` ที่ขึ้นต้น pattern | ระบุตำแหน่งแบบ absolute จาก root ของ repo | `/build/` = เฉพาะโฟลเดอร์ `build` ที่ root เท่านั้น |
| `/` ที่ท้าย pattern | หมายถึงต้องเป็นโฟลเดอร์เท่านั้น | `/docs/` = โฟลเดอร์ docs เท่านั้น ไม่ match ไฟล์ชื่อ docs |
| ไม่มี `/` เลย | match ได้ทุกระดับความลึกในโปรเจกต์ | `README.md` = match ไฟล์ชื่อนี้ในทุกโฟลเดอร์ |
| `?` | ตัวอักษรเดี่ยวใด ๆ 1 ตัว | `file?.txt` = match `file1.txt`, `fileA.txt` |

### ตัวอย่างที่ใช้บ่อยที่สุดพร้อมคำอธิบาย

```gitattributes
# ---------- 1. ทุกไฟล์ในโปรเจกต์ (ควรอยู่บรรทัดบนสุดเสมอ เป็น fallback) ----------
*                           @nan-dev

# ---------- 2. ไฟล์ตามนามสกุล ไม่สนตำแหน่ง ----------
*.css                       @frontend-team
*.scss                      @frontend-team
*.go                        @backend-team

# ---------- 3. โฟลเดอร์เฉพาะที่ root เท่านั้น (ขึ้นต้นด้วย /) ----------
/scripts/                   @devops-team
/infra/                     @devops-team

# ---------- 4. โฟลเดอร์ชื่อนี้ ไม่ว่าจะอยู่ระดับไหนก็ตาม (ไม่มี / ขึ้นต้น) ----------
tests/                      @qa-team

# ---------- 5. ไฟล์เฉพาะเจาะจง ระบุ path เต็ม ----------
/package.json                @nan-dev @frontend-lead
/go.mod                      @backend-lead

# ---------- 6. ใช้ ** สำหรับข้ามหลายระดับโฟลเดอร์ ----------
apps/**/migrations/          @database-team

# ---------- 7. ไฟล์ในโฟลเดอร์ย่อยระดับใดก็ได้ที่ตรงชื่อ ----------
**/Dockerfile                @devops-team
```

### เจาะลึกความแตกต่างระหว่าง `*` และ `**`

ความสับสนที่พบบ่อยที่สุดคือเรื่องนี้ ลองดูตัวอย่างเปรียบเทียบ:

```
โครงสร้างโปรเจกต์:
src/
├── utils.js
└── components/
    └── Button.js
```

| Pattern | Match `src/utils.js` | Match `src/components/Button.js` |
|---|---|---|
| `*.js` | ✅ (ไม่มี `/` ในตัว pattern เอง ทำให้ match ได้ทุกระดับ) | ✅ |
| `src/*.js` | ✅ | ❌ (เพราะ `*` เดี่ยวไม่ข้ามระดับ `/`) |
| `src/**/*.js` | ✅ | ✅ (`**` ข้ามได้ทุกระดับ) |
| `src/**` | ✅ | ✅ (ทุกอย่างใต้ `src/` ไม่ว่าลึกแค่ไหน) |

**ข้อควรจำที่สำคัญที่สุด:** pattern ที่ไม่มีเครื่องหมาย `/` อยู่ในตัวมันเลย (เช่น `*.js` หรือ `README.md`) จะถูกมองว่าเป็น pattern แบบ "match ได้ทุกที่ในทุกระดับความลึก" เหมือนกับพฤติกรรมของ `.gitignore` ทุกประการ แต่ทันทีที่ pattern มี `/` ปรากฏอยู่ตรงกลางหรือขึ้นต้น (เช่น `src/*.js` หรือ `/src/*.js`) มันจะถูกตีความว่าอ้างอิงจาก root ของ repository ทันที

### ตัวอย่างจริง: โครงสร้างโปรเจกต์ Full-stack และไฟล์ CODEOWNERS ที่สอดคล้องกัน

```
my-app/
├── frontend/
│   ├── src/
│   └── package.json
├── backend/
│   ├── src/
│   └── go.mod
├── infra/
│   ├── terraform/
│   └── k8s/
├── docs/
│   └── architecture.md
└── .github/
    ├── CODEOWNERS
    └── workflows/
```

ไฟล์ CODEOWNERS ที่ตรงกับโครงสร้างนี้:

```gitattributes
# Default owner สำหรับทุกไฟล์ที่ไม่ตรงกฎด้านล่าง
*                     @nan-dev

# Frontend
/frontend/            @company/frontend-team

# Backend
/backend/             @company/backend-team

# Infrastructure และ Deployment
/infra/               @company/devops-team
/.github/workflows/   @company/devops-team

# เอกสาร
/docs/                @company/docs-team
```

ด้วยไฟล์นี้ ถ้ามีคนเปิด PR ที่แก้ไฟล์ทั้งใน `/frontend/` และ `/backend/` พร้อมกัน (เช่น เพิ่มฟีเจอร์ที่ต้องแก้ทั้ง API และ UI) GitHub จะ auto-request review **จากทั้งสองทีมพร้อมกัน** เพราะแต่ละไฟล์ที่เปลี่ยนแปลงจะถูกจับคู่กับกฎที่ match มันเอง ไม่ใช่แค่กฎเดียวสำหรับทั้ง PR

### ข้อควรระวังเรื่อง Trailing Slash

```gitattributes
apps/         @team-a     # หมายถึงโฟลเดอร์ apps เท่านั้น ไม่ match ไฟล์ที่ชื่อ "apps" เฉย ๆ
apps          @team-b     # หมายถึงทั้งไฟล์และโฟลเดอร์ที่ชื่อ apps
```

ถ้าคุณต้องการความชัดเจนและป้องกันความสับสน แนะนำให้ **ใส่ `/` ท้าย pattern ทุกครั้งที่หมายถึงโฟลเดอร์** เพื่อให้อ่านง่ายและลด edge case ที่ไม่คาดคิด

ในหัวข้อถัดไป เราจะพูดถึงการเลือกว่าจะใส่ชื่อ owner เป็น individual username หรือ team name แบบไหนเหมาะกับสถานการณ์ไหน

---

## Step 354: Team-based Ownership เทียบกับ Individual Ownership

### สองรูปแบบของการระบุเจ้าของ

CODEOWNERS รองรับการระบุเจ้าของได้ 2 แบบหลัก:

```gitattributes
# แบบที่ 1: Individual — ระบุชื่อผู้ใช้ตรง ๆ
/payment/    @somchai-dev

# แบบที่ 2: Team-based — ระบุทีมทั้งทีมใน organization
/payment/    @company/payments-team
```

**ข้อกำหนดสำคัญของ Team-based ownership:** การใช้ `@org/team-name` จะใช้ได้ก็ต่อเมื่อ:

1. Repository นั้นอยู่ภายใต้ **GitHub Organization** (ไม่ใช่ personal account) เพราะ personal account ไม่มีแนวคิดเรื่อง "team"
2. ทีมนั้นต้องถูกสร้างไว้แล้วใน organization settings และมี **write access** อย่างน้อยไปยัง repository นี้
3. ทีมนั้นต้องมีการตั้งค่า **visibility เป็น "Visible"** ไม่ใช่ "Secret" — เพราะทีมแบบ secret team ไม่สามารถถูกใช้เป็น code owner ได้ (ระบบจะมองไม่เห็นสมาชิกในทีม)

### เปรียบเทียบข้อดี-ข้อเสียของแต่ละแบบ

| ประเด็น | Individual (`@username`) | Team-based (`@org/team`) |
|---|---|---|
| ความชัดเจนว่าใคร responsible | สูงมาก รู้ตัวบุคคลชัดเจน | ชัดเจนระดับทีม แต่ไม่รู้ว่าใครในทีมจะมารีวิว |
| การดูแลรักษาเมื่อคนลาออก/ย้ายทีม | **ต้องแก้ไฟล์ CODEOWNERS ทุกครั้ง** ที่มีการเปลี่ยนคน | ไม่ต้องแก้ไฟล์เลย แค่จัดการสมาชิกในทีมผ่าน Org Settings |
| กระจายภาระงาน (load balancing) | ตกอยู่ที่คนเดียวเสมอ ถ้าคนนั้นไม่ว่างจะเป็นคอขวด | GitHub สามารถกระจาย request ไปยังสมาชิกในทีมได้ (ผ่าน review assignment) |
| เหมาะกับสถานการณ์ | โปรเจกต์เล็ก, ไฟล์ที่ต้องการความเชี่ยวชาญเฉพาะบุคคลจริง ๆ (เช่น security-critical file ที่มีคนเดียวที่เข้าใจลึก) | ทีมขนาดกลาง-ใหญ่, monorepo, โค้ดที่ดูแลร่วมกันเป็นทีม |
| ความเสี่ยงเมื่อคนคนนั้นลาพัก | PR ค้าง ไม่มีใคร approve ได้ถ้า required review เข้มงวด | สมาชิกคนอื่นในทีมยัง approve แทนได้ |

### แนวทางที่แนะนำในทางปฏิบัติ (Best Practice)

> **ใช้ Team-based ownership เป็นค่าเริ่มต้นเสมอสำหรับโค้ดที่มีคนดูแลมากกว่า 1 คน และสงวน Individual ownership ไว้เฉพาะกรณีพิเศษจริง ๆ เท่านั้น**

เหตุผลหลักคือเรื่อง **Bus Factor** — ถ้าองค์กรผูก ownership กับตัวบุคคลมากเกินไป เมื่อคนคนนั้นลาออก ย้ายทีม หรือลาพักร้อนยาว ๆ ระบบทั้งหมดจะสะดุด ในขณะที่การผูกกับทีมทำให้ knowledge และ responsibility กระจายตัวอยู่เสมอ

ตัวอย่างการผสมทั้งสองแบบในสถานการณ์จริง:

```gitattributes
# ใช้ team สำหรับโค้ดทั่วไปที่ทีมดูแลร่วมกัน
/backend/api/            @company/backend-team

# ใช้ individual เฉพาะไฟล์ที่ต้องการความเชี่ยวชาญเจาะจงจริง ๆ เช่น
# ไฟล์เกี่ยวกับระบบเข้ารหัสที่มีคนเดียวในองค์กรที่ทำ security audit ได้ลึกพอ
/backend/api/crypto/     @company/backend-team @nan-security-expert

# ใช้ individual สำหรับไฟล์ config ที่มีเจ้าของโดยตรงเป็นคนตัดสินใจสุดท้าย (เช่น Tech Lead)
/architecture-decisions/ @nan-dev
```

สังเกตว่าบรรทัดที่ 2 ในตัวอย่างข้างต้น **ใส่ทั้ง team และ individual พร้อมกันได้ในบรรทัดเดียว** — นี่คือการผสมทั้งสองแนวทางเข้าด้วยกัน โดยหมายความว่า PR ที่แก้ไฟล์ในโฟลเดอร์ `crypto/` จะถูก request review ทั้งจากทีม backend ทั้งทีม **และ** จาก `@nan-security-expert` โดยเฉพาะเจาะจงเพิ่มเข้ามาอีกคนหนึ่ง

### Nested Teams (ทีมย่อยซ้อนทีมใหญ่)

GitHub Organization รองรับการสร้างทีมแบบมีลำดับชั้น (parent team / child team) เช่น:

```
@company/engineering          (ทีมใหญ่)
  ├── @company/backend-team   (ทีมย่อย)
  ├── @company/frontend-team  (ทีมย่อย)
  └── @company/devops-team    (ทีมย่อย)
```

เมื่อระบุ `@company/engineering` เป็น owner ระบบจะพิจารณาสมาชิกของทีมย่อยทั้งหมดที่อยู่ภายใต้ทีมใหญ่นั้นด้วย ซึ่งมีประโยชน์เวลาต้องการกำหนด "เจ้าของสำรอง" ระดับองค์กรสำหรับไฟล์สำคัญที่ทุกทีมวิศวกรรมควรมีสิทธิ์รับรู้

---

## Step 355: การบังคับ Required Review จาก Code Owner ผ่าน Branch Protection

### CODEOWNERS เพียงอย่างเดียวยัง "ไม่บังคับ" อะไรเลย

ประเด็นที่หลายทีมเข้าใจผิดบ่อยที่สุดคือ **การมีไฟล์ CODEOWNERS เพียงอย่างเดียว ทำได้แค่ "auto-request reviewer" เท่านั้น มันยังไม่ได้บังคับว่า PR ต้องได้รับการ approve จากเจ้าของก่อนถึงจะ merge ได้**

พูดง่าย ๆ คือ ถ้าไม่ตั้งค่าเพิ่มเติม คนที่มีสิทธิ์ merge ยังสามารถกด **Merge** ได้ทันทีแม้ Code Owner ที่ถูก auto-assign ยังไม่ได้กด approve เลยก็ตาม

### การผูก CODEOWNERS เข้ากับ Branch Protection Rules

เพื่อให้ CODEOWNERS มีผล "บังคับ" จริง ต้องไปตั้งค่าที่ **Branch Protection Rules** (ซึ่งเราจะเรียนแบบละเอียดเต็ม Part ใน **Part 37: Branch Protection Rules และ Merge Strategies**) โดยเปิดใช้ตัวเลือกที่ชื่อว่า:

> **"Require review from Code Owners"**

ขั้นตอนการตั้งค่า (ภาพรวม):

1. ไปที่ **Settings → Branches** ของ repository
2. เลือก branch ที่ต้องการป้องกัน (เช่น `main` หรือ `production`) แล้วกด **Add branch protection rule** หรือแก้ไข rule ที่มีอยู่
3. เปิดใช้ **"Require a pull request before merging"**
4. ภายใต้ตัวเลือกนั้น เปิดใช้ **"Require review from Code Owners"**
5. (แนะนำ) เปิดใช้ **"Dismiss stale pull request approvals when new commits are pushed"** ควบคู่กันไปด้วย เพื่อให้แน่ใจว่าการ approve จะไม่ตกค้างจากโค้ดเวอร์ชันเก่า

เมื่อเปิดใช้ตัวเลือกนี้แล้ว ผลลัพธ์คือ:

```
PR แก้ไฟล์ /backend/payment.go
       │
       ▼
GitHub ตรวจสอบ CODEOWNERS → พบว่า @company/payments-team เป็นเจ้าของ
       │
       ▼
Auto-request review จาก @company/payments-team
       │
       ▼
ปุ่ม "Merge" จะถูก "บล็อก" ไว้ (สีเทา กดไม่ได้)
จนกว่าสมาชิกในทีม payments-team อย่างน้อย 1 คน จะกด "Approve"
```

### ความสัมพันธ์กับ "Require approvals" (จำนวน approve ขั้นต่ำ)

Branch Protection ยังมีตัวเลือกแยกต่างหากคือ **"Require approvals"** ซึ่งกำหนดจำนวน approve ขั้นต่ำจากใครก็ได้ที่มีสิทธิ์รีวิว (ไม่จำเป็นต้องเป็น code owner) ตัวเลือกนี้ทำงาน**ร่วมกัน**กับ "Require review from Code Owners" ไม่ใช่แทนที่กัน:

| ตัวเลือกที่เปิด | ผลลัพธ์ |
|---|---|
| เปิดเฉพาะ "Require approvals: 2" | ต้องมี approve 2 คนขึ้นไป (ใครก็ได้ที่มีสิทธิ์รีวิว ไม่จำเป็นต้องเป็น owner) |
| เปิดเฉพาะ "Require review from Code Owners" | ต้องมี code owner approve อย่างน้อย 1 คน แต่ไม่ได้บังคับจำนวนรวมทั้งหมด |
| เปิดทั้งสองพร้อมกัน | ต้องมี approve รวมตามจำนวนที่กำหนด **และ** ในจำนวนนั้นต้องมีอย่างน้อย 1 คนเป็น code owner ของไฟล์ที่ถูกแก้ |

แนวทางที่หลายองค์กรระดับ production ใช้จริงคือการเปิดทั้งสองตัวเลือกพร้อมกัน เช่น "Require approvals: 2" + "Require review from Code Owners" เพื่อให้มั่นใจว่ามีทั้งปริมาณคนรีวิวที่เพียงพอ **และ** มีคนที่เชี่ยวชาญเฉพาะทางในส่วนนั้นจริง ๆ ยืนยันด้วย

### กรณีพิเศษ: Repository Administrator สามารถ "ข้าม" กฎนี้ได้หรือไม่

Branch Protection มีตัวเลือกแยกต่างหากชื่อ **"Do not allow bypassing the above settings"** ถ้าไม่ได้เปิดตัวเลือกนี้ไว้ ผู้ที่มีสิทธิ์ **Administrator** ของ repository จะยังสามารถ merge ได้แม้ไม่ผ่านเงื่อนไข code owner review (เพื่อรองรับสถานการณ์ฉุกเฉิน เช่น hotfix ที่ต้อง merge ด่วนมากในเวลาที่ไม่มีเจ้าของไฟล์อยู่ออนไลน์) แต่ในทีมที่ต้องการความเข้มงวดสูงสุด (เช่น สาย compliance, การเงิน, ระบบที่ต้องผ่าน audit) ควรเปิดตัวเลือกนี้ไว้เพื่อไม่ให้มีข้อยกเว้นใด ๆ เลยแม้แต่ admin เอง

### สรุปความสัมพันธ์ในรูปแบบตาราง

| องค์ประกอบ | หน้าที่ |
|---|---|
| ไฟล์ `CODEOWNERS` | ประกาศว่าใคร/ทีมไหนเป็นเจ้าของไฟล์ส่วนไหน + auto-request reviewer |
| "Require review from Code Owners" (Branch Protection) | ทำให้การ approve จาก code owner เป็น **เงื่อนไขบังคับ** ก่อน merge ได้ |
| "Require approvals" (Branch Protection) | กำหนดจำนวน approve ขั้นต่ำโดยรวม (ไม่จำกัดว่าต้องเป็น owner) |
| "Do not allow bypassing" (Branch Protection) | ปิดช่องทางให้ admin ข้ามกฎทั้งหมดได้ |

ทั้งสี่องค์ประกอบนี้เมื่อทำงานร่วมกัน จะสร้างระบบ **Governance ที่บังคับใช้ได้จริงโดยไม่ต้องพึ่งวินัยส่วนบุคคล** ซึ่งเป็นเป้าหมายสูงสุดของการทำ Code Review Process ในระดับองค์กร

---

## Step 356: Multiple Owners ในไฟล์เดียวกัน และ Order of Precedence

### กฎที่สำคัญที่สุดข้อเดียวของ CODEOWNERS

ถ้าจะให้จำเรื่อง CODEOWNERS ไว้แค่ 1 ประโยค ควรจะเป็นประโยคนี้:

> **เมื่อมีหลาย pattern ในไฟล์ที่ match ไฟล์เดียวกัน "บรรทัดที่อยู่ล่างสุด (last matching pattern) เท่านั้นที่มีผล" — บรรทัดที่ match ก่อนหน้าจะถูกยกเลิกไปเลย ไม่ใช่การรวมกัน**

นี่คือจุดที่ **แตกต่างจาก `.gitignore` โดยสิ้นเชิง** เพราะใน `.gitignore` กฎที่ตามหลังสามารถ "un-ignore" ไฟล์ที่ถูก ignore ไปแล้วได้ (ด้วย `!pattern`) แต่ตัว pattern ที่ match ไม่ได้ "แทนที่" กันแบบสมบูรณ์เหมือนใน CODEOWNERS

พูดให้ชัดกว่านั้น: **CODEOWNERS ไม่มีแนวคิดเรื่องการรวมกฎจากหลายบรรทัดเข้าด้วยกัน** สำหรับไฟล์หนึ่งไฟล์ กฎที่ match ล่าสุด (บรรทัดที่อยู่ต่ำที่สุดในไฟล์) จะเป็นกฎเดียวที่ถูกใช้ทั้งหมด กฎก่อนหน้าที่ match ไฟล์เดียวกันจะถูกเพิกเฉยไปทั้งบรรทัด

### ตัวอย่างที่แสดงให้เห็นชัดเจน

```gitattributes
*                        @nan-dev
/backend/                @company/backend-team
/backend/legacy/         @somchai-old-timer
```

ลองไล่ดูว่าไฟล์แต่ละไฟล์จะมีใครเป็นเจ้าของ:

| ไฟล์ | Pattern ที่ match ทั้งหมด | Pattern ที่ "ชนะ" (บรรทัดล่างสุด) | เจ้าของจริง |
|---|---|---|---|
| `README.md` | `*` | `*` | `@nan-dev` |
| `/backend/api.go` | `*`, `/backend/` | `/backend/` | `@company/backend-team` |
| `/backend/legacy/old.go` | `*`, `/backend/`, `/backend/legacy/` | `/backend/legacy/` | `@somchai-old-timer` **เท่านั้น** |

**สังเกตให้ดี:** ไฟล์ `/backend/legacy/old.go` แม้จะ match ทั้ง 3 กฎ แต่เจ้าของสุดท้ายคือ `@somchai-old-timer` **เพียงคนเดียว** ไม่ได้รวมกับ `@company/backend-team` หรือ `@nan-dev` เลย เพราะกฎบรรทัดล่างสุดที่ match "แทนที่" กฎด้านบนทั้งหมดโดยสมบูรณ์

### ผลที่ตามมาในทางปฏิบัติ: ต้องเรียงจากกว้างไปแคบเสมอ

จากหลักการนี้ จึงเกิดกฎทองของการเขียนไฟล์ CODEOWNERS คือ:

> **ให้เรียง pattern จากกว้างที่สุด (general) ไปยังแคบที่สุด (specific) เสมอ โดยกฎที่เจาะจงที่สุดต้องอยู่ล่างสุดของไฟล์**

```gitattributes
# ✅ ถูกต้อง: กว้าง → แคบ เรียงจากบนลงล่าง
*                          @nan-dev
/backend/                  @company/backend-team
/backend/payment/          @company/payments-team
/backend/payment/refund.go @nan-dev @somchai-finance-expert
```

ถ้าเขียนสลับลำดับกัน ผลลัพธ์จะผิดทันทีโดยไม่มี error ใด ๆ แจ้งเตือน:

```gitattributes
# ❌ ผิด: กฎเจาะจงอยู่บนกฎกว้าง ทำให้กฎเจาะจงไม่มีผลอะไรเลย
/backend/payment/refund.go @nan-dev @somchai-finance-expert
/backend/                  @company/backend-team
*                          @nan-dev
```

ในตัวอย่างที่ผิดด้านบน ไฟล์ `/backend/payment/refund.go` จะกลายเป็นของ `@nan-dev` (จากกฎ `*` ที่อยู่ล่างสุด) แทนที่จะเป็นของ `@somchai-finance-expert` ตามที่ตั้งใจไว้ — นี่คือบั๊กเงียบที่พบบ่อยที่สุดในไฟล์ CODEOWNERS ที่เขียนโดยไม่เข้าใจเรื่อง precedence

### การกำหนด "ไม่มีเจ้าของ" เพื่อยกเว้นไฟล์บางกลุ่ม

อย่างที่กล่าวไว้ใน Step 352 คุณสามารถเขียน pattern โดยไม่ระบุ owner เพื่อบอกว่า "ไฟล์กลุ่มนี้ไม่มีเจ้าของ" ซึ่งมีประโยชน์มากเวลาต้องการยกเว้นไฟล์บางกลุ่มออกจากกฎกว้างด้านบน:

```gitattributes
# ทุกไฟล์ในโฟลเดอร์ vendor ให้ backend team ดูแล
/backend/                  @company/backend-team

# ยกเว้นไฟล์ vendor ที่เป็นโค้ด third-party — ไม่มีใครเป็นเจ้าของ ไม่ต้อง review พิเศษ
/backend/vendor/           
```

บรรทัดสุดท้ายที่ไม่มีชื่อ owner ต่อท้าย จะทำให้ไฟล์ในโฟลเดอร์ `vendor/` ไม่ถูก auto-request review จากใครเลย แม้ว่าจะอยู่ใต้ `/backend/` ที่มีเจ้าของกำหนดไว้กว้าง ๆ ก็ตาม เพราะกฎนี้อยู่ล่างสุดและ match เจาะจงกว่า

### Multiple Owners ในบรรทัดเดียวกัน (ไม่ใช่คนละบรรทัด)

แยกให้ออกจากเรื่อง precedence ข้างต้น การใส่ **หลาย owner ในบรรทัดเดียวกัน** ไม่ใช่ปัญหาเรื่อง precedence เลย เพราะทุกคน/ทุกทีมที่ระบุในบรรทัดนั้นจะถูก request review พร้อมกันทั้งหมด:

```gitattributes
# ไฟล์นี้ต้องผ่านสายตาทั้ง 3 ฝ่ายก่อน merge
/infra/production-config.yml   @company/devops-team @company/security-team @nan-dev
```

เมื่อมีคนแก้ไฟล์ `production-config.yml` ทั้ง `@company/devops-team`, `@company/security-team` และ `@nan-dev` จะถูก auto-request review **ทั้งหมดพร้อมกัน** ในคราวเดียว — นี่คือสถานการณ์ที่ต่างจากเรื่อง precedence เพราะที่นี่ไม่มีการ "แข่งกัน" ระหว่าง pattern เลย มีแค่ pattern เดียวที่ match

---

## Step 357: การทดสอบว่า CODEOWNERS ทำงานถูกต้อง

### ทำไมต้องทดสอบ — เพราะ CODEOWNERS ไม่มี Syntax Validator ที่เข้มงวด

ปัญหาใหญ่ที่สุดของไฟล์ CODEOWNERS คือ **มันไม่มีระบบแจ้งเตือนที่ชัดเจนเวลาคุณเขียนผิด** ถ้า pattern เขียนผิด หรือ username สะกดผิด ระบบจะไม่ error ให้เห็นตรง ๆ แต่จะแค่ **"ไม่ auto-assign reviewer ให้"** อย่างเงียบ ๆ ซึ่งเป็นสิ่งที่อันตรายมาก เพราะทีมอาจเข้าใจผิดคิดว่ามีการป้องกันอยู่ ทั้งที่จริงแล้วไม่มีเลย

ดังนั้นการทดสอบว่า CODEOWNERS ทำงานจริงตามที่ตั้งใจไว้จึงเป็นขั้นตอนที่ **จำเป็นเสมอ** หลังเขียนหรือแก้ไขไฟล์นี้ทุกครั้ง

### วิธีที่ 1: ใช้หน้า "View file" ของ GitHub ตรวจสอบ Syntax เบื้องต้น

เมื่อ push ไฟล์ `.github/CODEOWNERS` ขึ้นไปแล้ว ให้เปิดดูไฟล์นั้นบนหน้าเว็บ GitHub โดยตรง GitHub จะแสดงผลการตรวจสอบ syntax เบื้องต้นให้ในหน้านั้นเอง — ถ้ามีบรรทัดที่ syntax ผิด (เช่น owner ที่ไม่มีสิทธิ์เข้าถึง repo หรือ pattern ที่เขียนผิดรูปแบบ) GitHub จะ**ขีดเส้นใต้สีแดง**พร้อมข้อความอธิบายตรงบรรทัดนั้นให้เห็นทันที ซึ่งเป็นวิธีตรวจสอบเบื้องต้นที่เร็วและง่ายที่สุด

### วิธีที่ 2: เปิด Pull Request ทดสอบจริง (วิธีที่น่าเชื่อถือที่สุด)

วิธีที่แม่นยำที่สุดคือการทดสอบผ่านสถานการณ์จริง:

1. สร้าง branch ทดสอบใหม่ เช่น `test/codeowners-check`
2. แก้ไฟล์ที่อยู่ในแต่ละกฎที่ต้องการทดสอบ (เช่น แก้ไฟล์ใน `/frontend/`, `/backend/`, `/docs/` อย่างละไฟล์)
3. เปิด Pull Request จาก branch นี้
4. ไปดูที่แถบ **"Reviewers"** ทางด้านขวาของหน้า PR
5. ตรวจสอบว่ารายชื่อทีม/บุคคลที่ปรากฏใน "Reviewers" ตรงกับที่ตั้งใจไว้ในไฟล์ CODEOWNERS หรือไม่

ถ้ารายชื่อที่ควรปรากฏไม่ปรากฏขึ้นมา แปลว่ามีปัญหาอย่างใดอย่างหนึ่งต่อไปนี้:

- Username หรือชื่อทีมสะกดผิด
- Owner ที่ระบุไม่มีสิทธิ์ write access ต่อ repository นี้
- Pattern เขียนผิด ทำให้ไม่ match ไฟล์ตามที่ตั้งใจ
- มีกฎบรรทัดล่างที่ match แคบกว่าและ "ทับ" กฎที่ต้องการทดสอบไปแล้ว (ตาม precedence ใน Step 356)

### วิธีที่ 3: ใช้ GitHub CLI เพื่อตรวจสอบ Reviewer ที่ถูก assign

หลังเปิด PR แล้ว สามารถใช้คำสั่งนี้เพื่อตรวจสอบรายชื่อ reviewer ที่ถูก request แบบไม่ต้องเปิดเว็บ:

```bash
gh pr view <PR_NUMBER> --json reviewRequests
```

ผลลัพธ์จะแสดงรายชื่อ user/team ทั้งหมดที่ถูก auto-request review เข้ามา ทำให้ตรวจสอบผ่าน script หรือ CI ได้ในอนาคตด้วย ถ้าทีมต้องการ automate การตรวจสอบนี้

### วิธีที่ 4: การจำลอง (Dry-run) ด้วย Checklist ก่อน Merge ไฟล์ CODEOWNERS

ก่อน merge การแก้ไขไฟล์ CODEOWNERS เข้า main branch ควรผ่าน checklist นี้เสมอ:

- [ ] เปิดไฟล์บนหน้าเว็บ GitHub แล้วไม่มีเส้นใต้สีแดงเตือน syntax ผิด
- [ ] ทดสอบเปิด PR จริงที่แก้ไฟล์ในทุกกลุ่ม pattern อย่างน้อยกลุ่มละ 1 ไฟล์
- [ ] ตรวจสอบว่า owner ทุกคน/ทุกทีมที่ระบุไว้มี write access ต่อ repository แล้วจริง
- [ ] ตรวจสอบว่าไม่มีกฎที่เจาะจงกว่าเขียนอยู่เหนือกฎที่กว้างกว่า (ผิดหลัก precedence)
- [ ] ถ้าเปิดใช้ "Require review from Code Owners" ไว้ ให้ทดสอบว่าปุ่ม Merge ถูกบล็อกจริงจนกว่าจะมี approve จาก owner

### เหตุการณ์ที่พบบ่อย: CODEOWNERS "ดูเหมือนทำงาน" แต่จริง ๆ ไม่ได้บังคับอะไรเลย

สถานการณ์ที่พบบ่อยมากคือทีมเขียนไฟล์ CODEOWNERS ถูกต้องสมบูรณ์ ทดสอบแล้วเห็น reviewer ถูก auto-assign จริง แต่ลืมไปเปิด **"Require review from Code Owners"** ใน Branch Protection (ตามที่อธิบายใน Step 355) ทำให้ในทางปฏิบัติ ใครก็ยังสามารถ merge PR ได้โดยไม่รอ approve จาก owner เลย — นี่คือเหตุผลว่าทำไม Step 355 และ Step 357 จึงต้องทำควบคู่กันเสมอ การทดสอบที่สมบูรณ์จริง ๆ ต้องรวมถึงการลองกด **Merge** ตอนที่ owner ยังไม่ approve ด้วย เพื่อยืนยันว่าปุ่มถูกบล็อกจริง

---

## Step 358: Best Practices การจัดโครงสร้าง CODEOWNERS สำหรับ Monorepo หลายทีม

### ความท้าทายเฉพาะของ Monorepo

Monorepo คือ repository เดียวที่รวมโค้ดของหลายโปรเจกต์ หลายทีม หรือหลาย service ไว้ด้วยกัน (ตรงข้ามกับ Polyrepo ที่แยก repository ต่อโปรเจกต์) เมื่อทีมจำนวนมากทำงานอยู่ใน repository เดียวกัน ไฟล์ CODEOWNERS จะมีความสำคัญและความซับซ้อนสูงขึ้นมาก เพราะต้องรองรับ:

- ทีมจำนวนมาก (อาจมากกว่า 10 ทีมในองค์กรใหญ่)
- โค้ดที่ใช้ร่วมกันระหว่างทีม (shared libraries)
- ไฟล์ configuration ระดับ root ที่กระทบทุกทีม
- การเปลี่ยนแปลงโครงสร้างโฟลเดอร์บ่อยเมื่อทีมแยก/รวมกัน

### Best Practice 1: จัดกลุ่มด้วย Comment Header ตามทีม/โดเมน

แทนที่จะเรียง pattern แบบสุ่ม ให้จัดกลุ่มตาม domain หรือทีมอย่างชัดเจน พร้อม comment header คั่นแต่ละส่วน เพื่อให้อ่านง่ายเมื่อไฟล์มีขนาดใหญ่:

```gitattributes
# =========================================
# Default Owner (fallback สำหรับทุกไฟล์)
# =========================================
*                              @company/tech-leads

# =========================================
# Frontend Team
# =========================================
/apps/web/                    @company/frontend-team
/apps/mobile/                 @company/mobile-team
/packages/ui-components/      @company/frontend-team

# =========================================
# Backend Team
# =========================================
/services/api/                @company/backend-team
/services/auth/               @company/backend-team @company/security-team
/packages/shared-models/      @company/backend-team

# =========================================
# Documentation Team
# =========================================
/docs/                        @company/docs-team
*.md                          @company/docs-team

# =========================================
# DevOps / Platform Team
# =========================================
/.github/workflows/           @company/devops-team
/infra/                       @company/devops-team
/docker/                      @company/devops-team

# =========================================
# Security-Critical Files (เจาะจงที่สุด ต้องอยู่ล่างสุดของไฟล์)
# =========================================
/services/auth/secrets/       @company/security-team
**/*.pem                      @company/security-team
**/.env.production            @company/security-team
```

### Best Practice 2: กำหนด Default Owner ระดับ Tech Lead เสมอ

ควรมีบรรทัด `*` อยู่บนสุดของไฟล์เสมอ ชี้ไปยังทีมหรือคนที่รับผิดชอบภาพรวม (เช่น Tech Lead หรือ Core Maintainers Team) เพื่อไม่ให้มีไฟล์ใดในโปรเจกต์ที่ "ไม่มีเจ้าของเลย" โดยไม่ได้ตั้งใจ — เพราะไฟล์ที่ไม่มีเจ้าของจะไม่ถูก auto-request review ให้ใครเลย ซึ่งเป็นช่องโหว่ที่อันตราย

### Best Practice 3: แยกไฟล์ CODEOWNERS ระดับ Monorepo ด้วยแนวคิด "Ownership Boundary" ให้ตรงกับโครงสร้างจริง

ในหลายองค์กรที่ทำ monorepo ขนาดใหญ่มาก จะออกแบบโครงสร้างโฟลเดอร์ให้สอดคล้องกับขอบเขตของทีมตั้งแต่แรก (เรียกว่า "Ownership-aligned folder structure") เพื่อให้ไฟล์ CODEOWNERS เขียนง่ายและไม่ซับซ้อน:

```
monorepo/
├── teams/
│   ├── frontend/
│   ├── backend/
│   └── platform/
├── shared/
│   └── libs/
└── docs/
```

```gitattributes
/teams/frontend/    @company/frontend-team
/teams/backend/     @company/backend-team
/teams/platform/    @company/platform-team
/shared/libs/       @company/architecture-council
/docs/              @company/docs-team
```

แนวทางนี้ทำให้ไฟล์ CODEOWNERS มีจำนวนบรรทัดน้อย อ่านง่าย และไม่ต้องแก้บ่อย แม้ทีมจะเพิ่มไฟล์ใหม่ภายในโฟลเดอร์ของตัวเองก็ตาม เพราะกฎถูกผูกกับ "ขอบเขตโฟลเดอร์" ไม่ใช่ไฟล์เดี่ยว ๆ

### Best Practice 4: ตั้ง Owner พิเศษสำหรับไฟล์ที่กระทบข้ามทีม (Cross-cutting Concerns)

ไฟล์บางประเภทที่ส่งผลกระทบต่อทุกทีมพร้อมกันควรมีกฎเฉพาะ เช่น:

```gitattributes
# ไฟล์ที่กระทบ build pipeline ของทุกทีม ต้องให้ Architecture Council เห็นก่อนเสมอ
/package.json                  @company/architecture-council
/pnpm-workspace.yaml           @company/architecture-council
/tsconfig.base.json            @company/architecture-council

# ไฟล์ CODEOWNERS เองก็ควรมีเจ้าของ (ป้องกันคนแก้กฎเองโดยไม่ผ่านใคร)
/.github/CODEOWNERS            @company/tech-leads
```

ข้อสุดท้ายนี้สำคัญมาก — **การกำหนดเจ้าของให้ไฟล์ CODEOWNERS เอง** เป็นเทคนิคป้องกันไม่ให้ใครแอบแก้ไขกฎความเป็นเจ้าของเพื่อ "หลบ" การถูกตรวจสอบ ซึ่งเป็นความเสี่ยงด้าน security ที่มักถูกมองข้าม

### Best Practice 5: ทบทวนไฟล์ CODEOWNERS เป็นระยะ

ไฟล์นี้ควรถูกตรวจทานอย่างสม่ำเสมอ (เช่น ทุกไตรมาส) เพื่อ:

- ลบชื่อคนที่ลาออกหรือย้ายทีมออกจากกฎ (โดยเฉพาะกฎแบบ individual)
- ปรับ pattern ให้ตรงกับโครงสร้างโฟลเดอร์ปัจจุบัน หลังมีการ refactor ครั้งใหญ่
- ตรวจสอบว่าไม่มีทีมไหนถูกลืมเพิ่มเข้าไปหลังมีการตั้งทีมใหม่

---

## Step 359: ข้อจำกัดและปัญหาที่พบบ่อยของ CODEOWNERS

### ข้อจำกัดที่ 1: ไม่มี Error ที่ชัดเจนเมื่อ Syntax ผิด

อย่างที่กล่าวไปแล้วใน Step 357 นี่คือข้อจำกัดที่อันตรายที่สุด — CODEOWNERS **fail แบบเงียบ (silent failure)** เกือบทุกกรณี:

| ความผิดพลาด | สิ่งที่เกิดขึ้น |
|---|---|
| Username สะกดผิด | ไม่มี reviewer ถูก assign ให้ ไม่มี error แจ้งเตือน |
| Owner ไม่มี write access | ไม่มี reviewer ถูก assign ให้ ไม่มี error แจ้งเตือน |
| Pattern เขียนผิดรูปแบบ | อาจไม่ match ไฟล์ตามที่ตั้งใจ โดยไม่มีคำเตือนใด ๆ |
| ทีมเป็น Secret Team | ไม่สามารถใช้เป็น owner ได้ ไม่มี error ชัดเจน |
| ลืมเปิด Branch Protection | ไฟล์ CODEOWNERS ทำงานแค่ auto-assign แต่ไม่บังคับ merge เลย |

ทางแก้คือการสร้างวินัยในการทดสอบตามที่อธิบายไว้ใน Step 357 เป็นขั้นตอนบังคับทุกครั้งที่แก้ไฟล์นี้ ไม่ควรเชื่อว่า "เขียนถูก syntax แล้วต้องทำงานถูก"

### ข้อจำกัดที่ 2: CODEOWNERS ไม่รองรับการ Exclude แบบซับซ้อน

ต่างจาก `.gitignore` ที่รองรับ negation pattern (`!pattern`) เพื่อ "ยกเลิก" การ ignore ไฟล์บางตัว **CODEOWNERS ไม่มี syntax สำหรับ negation โดยตรง** วิธีเดียวที่ทำได้คือการเขียนกฎที่เจาะจงกว่าไว้ด้านล่าง (ตามหลัก precedence ใน Step 356) เพื่อ "ทับ" กฎที่กว้างกว่า ซึ่งทำให้ไฟล์ที่มีเงื่อนไข exclude ซับซ้อนมาก ๆ อ่านยากขึ้นตามไปด้วย

### ข้อจำกัดที่ 3: จำนวน Reviewer ที่ GitHub จะ Auto-request มีเพดานจำกัด

GitHub จำกัดจำนวนคน/ทีมที่จะถูก auto-request review จาก CODEOWNERS ไว้สูงสุด **ไม่เกิน 3 ทีม (team) ต่อ pattern ที่ match หนึ่งรายการ** — ถ้าระบุทีมมากกว่า 3 ทีมในบรรทัดเดียวกัน ระบบจะสุ่มเลือกจากขอบเขตที่กำหนดหรืออาจไม่ทำงานตามที่คาด ดังนั้นไม่ควรออกแบบกฎที่ต้องพึ่งพาทีมจำนวนมากในบรรทัดเดียวกัน ควรแยกความรับผิดชอบให้ชัดเจนกว่านี้แทน

### ข้อจำกัดที่ 4: Team ที่ไม่มี Write Access จะถูกเพิกเฉยแบบเงียบ ๆ

ทีมที่ระบุใน CODEOWNERS แต่ไม่มีสิทธิ์ write access ต่อ repository จะไม่ได้รับการ auto-request review เลย และไม่มีการแจ้งเตือนใด ๆ ปัญหานี้มักเกิดเมื่อมีการสร้างทีมใหม่ในองค์กรแล้วลืมเพิ่มสิทธิ์เข้าถึง repository ให้ทีมนั้นก่อนเพิ่มชื่อในไฟล์ CODEOWNERS

### ข้อจำกัดที่ 5: การเปลี่ยนแปลงไฟล์ CODEOWNERS เองไม่ Retroactive

ถ้าคุณแก้ไฟล์ CODEOWNERS ในระหว่างที่มี PR เปิดค้างอยู่แล้ว **การเปลี่ยนแปลงนั้นจะไม่มีผลย้อนหลังกับ PR ที่เปิดอยู่ก่อนหน้า** reviewer ที่ถูก assign ไปแล้วจะยังคงเป็นไปตามกฎเดิมตอนที่ PR ถูกเปิด (หรือตอนที่มีการ push commit ใหม่ล่าสุด) ไม่ใช่กฎล่าสุดในไฟล์ CODEOWNERS ปัจจุบันเสมอไป

### ข้อจำกัดที่ 6: Draft Pull Request ไม่ Trigger การ Auto-request Review

ถ้า PR ถูกเปิดในสถานะ **Draft** ระบบจะยังไม่ auto-request review จาก code owner จนกว่าจะถูกเปลี่ยนสถานะเป็น **"Ready for review"** ก่อน ทีมที่ใช้ draft PR บ่อยควรระวังจุดนี้ เพราะอาจเข้าใจผิดว่า CODEOWNERS ไม่ทำงาน ทั้งที่จริงแล้วแค่ PR ยังอยู่ในสถานะ draft

### สรุปข้อจำกัดในรูปแบบ Checklist สำหรับ Debug

เมื่อพบว่า CODEOWNERS "ดูเหมือนไม่ทำงาน" ให้ไล่ตรวจตามลำดับนี้:

- [ ] ไฟล์อยู่ในตำแหน่งที่ถูกต้องหรือไม่ (`.github/CODEOWNERS` เป็นอันดับแรก)
- [ ] Username/team name สะกดถูกต้องหรือไม่ (case-sensitive ในบางกรณี)
- [ ] Owner มี write access ต่อ repository แล้วหรือยัง
- [ ] ทีมที่ระบุเป็น Visible team ไม่ใช่ Secret team
- [ ] PR อยู่ในสถานะ "Ready for review" ไม่ใช่ Draft
- [ ] มีกฎอื่นที่เจาะจงกว่าอยู่ล่างกว่าและ "ทับ" กฎที่ต้องการอยู่หรือไม่
- [ ] เปิด "Require review from Code Owners" ใน Branch Protection แล้วหรือยัง (ถ้าต้องการบังคับ)

---

## Step 360: แบบฝึกหัด — สร้างไฟล์ CODEOWNERS สำหรับโปรเจกต์จำลอง 3 ทีม

### โจทย์

สมมติคุณกำลังดูแล repository ของบริษัทสมมติชื่อ **NimbusApp** ซึ่งมีโครงสร้างโฟลเดอร์ดังนี้:

```
nimbus-app/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   └── pages/
│   └── package.json
├── backend/
│   ├── src/
│   │   ├── api/
│   │   └── database/
│   └── go.mod
├── docs/
│   ├── api-reference.md
│   └── onboarding.md
├── shared/
│   └── types/
├── infra/
│   ├── terraform/
│   └── k8s/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
├── README.md
└── package.json
```

องค์กรมี 3 ทีมหลักที่ต้องดูแลคนละส่วน:

1. **`@nimbusapp/frontend-team`** — ดูแลทุกอย่างในโฟลเดอร์ `frontend/`
2. **`@nimbusapp/backend-team`** — ดูแลทุกอย่างในโฟลเดอร์ `backend/`
3. **`@nimbusapp/docs-team`** — ดูแลทุกอย่างในโฟลเดอร์ `docs/` และไฟล์ Markdown ทั้งหมด

นอกจากนี้ยังมีเงื่อนไขพิเศษเพิ่มเติมที่ต้องรองรับ:

- โฟลเดอร์ `shared/types/` ถูกใช้ร่วมกันระหว่าง frontend และ backend จึงต้องให้ **ทั้งสองทีม** ตรวจสอบร่วมกันเสมอ
- โฟลเดอร์ `infra/` และไฟล์ workflow ใน `.github/workflows/` ต้องมี **Tech Lead (`@nan-dev`)** เป็นผู้ตรวจสอบ เพราะยังไม่มีทีม DevOps แยกต่างหาก
- ไฟล์ `package.json` ที่ root (ไม่ใช่ของ frontend) เป็นไฟล์ workspace configuration ที่กระทบทุกทีม ต้องให้ `@nan-dev` ตรวจสอบเป็นพิเศษ
- ทุกไฟล์ที่ไม่ตรงกฎไหนเลย ให้ `@nan-dev` เป็นเจ้าของ default

### เฉลย

```gitattributes
# =====================================================
# .github/CODEOWNERS — NimbusApp
# =====================================================
#
# กฎเรียงจากกว้าง → แคบ (บรรทัดล่างสุดที่ match ชนะเสมอ)

# ---------- Default Owner ----------
*                          @nan-dev

# ---------- Frontend Team ----------
/frontend/                 @nimbusapp/frontend-team

# ---------- Backend Team ----------
/backend/                  @nimbusapp/backend-team

# ---------- Documentation Team ----------
/docs/                     @nimbusapp/docs-team
*.md                       @nimbusapp/docs-team

# ---------- Shared Code (ต้องให้ทั้ง frontend และ backend ตรวจ) ----------
/shared/types/             @nimbusapp/frontend-team @nimbusapp/backend-team

# ---------- Infrastructure และ CI/CD (ยังไม่มีทีม DevOps แยก) ----------
/infra/                    @nan-dev
/.github/workflows/        @nan-dev

# ---------- Root workspace config (กระทบทุกทีม) ----------
/package.json              @nan-dev

# ---------- ป้องกันตัวเอง: ไฟล์ CODEOWNERS ต้องมี Tech Lead ดูแล ----------
/.github/CODEOWNERS        @nan-dev
```

### เดินตรวจสอบผลลัพธ์ทีละไฟล์ (Verify ตามหลัก Precedence)

| ไฟล์ที่ถูกแก้ | กฎที่ match ทั้งหมด | กฎที่ชนะ (ล่างสุด) | เจ้าของจริง |
|---|---|---|---|
| `README.md` | `*`, `*.md` | `*.md` | `@nimbusapp/docs-team` |
| `frontend/src/pages/Home.tsx` | `*`, `/frontend/` | `/frontend/` | `@nimbusapp/frontend-team` |
| `backend/src/api/user.go` | `*`, `/backend/` | `/backend/` | `@nimbusapp/backend-team` |
| `docs/api-reference.md` | `*`, `*.md`, `/docs/` | `/docs/` | `@nimbusapp/docs-team` |
| `shared/types/user.ts` | `*`, `/shared/types/` | `/shared/types/` | `@nimbusapp/frontend-team` และ `@nimbusapp/backend-team` |
| `infra/terraform/main.tf` | `*`, `/infra/` | `/infra/` | `@nan-dev` |
| `.github/workflows/ci.yml` | `*`, `/.github/workflows/` | `/.github/workflows/` | `@nan-dev` |
| `package.json` (root) | `*`, `/package.json` | `/package.json` | `@nan-dev` |
| `frontend/package.json` | `*`, `/frontend/` | `/frontend/` | `@nimbusapp/frontend-team` (สังเกตว่า **ไม่ใช่** `@nan-dev` เพราะ `/package.json` เป็น absolute path ที่ match เฉพาะไฟล์ที่ root เท่านั้น ไม่ match ไฟล์ `frontend/package.json`) |

จุดสุดท้ายในตารางเป็นตัวอย่างที่ดีของความเข้าใจเรื่อง absolute path (`/package.json` ขึ้นต้นด้วย `/`) เทียบกับ pattern ที่ match ทุกระดับ — ถ้าเขียนเป็น `package.json` เฉย ๆ (ไม่มี `/` ขึ้นต้น) แทน มันจะ match ทุกไฟล์ที่ชื่อ `package.json` ในทุกโฟลเดอร์ ทำให้ `frontend/package.json` กลายเป็นของ `@nan-dev` ไปด้วย ซึ่งไม่ตรงกับเจตนาของโจทย์นี้

### ขั้นตอนถัดไปสำหรับผู้ฝึกฝนจริง

ให้ลองทำตามขั้นตอนนี้ในโปรเจกต์ทดลองของตัวเอง (ใช้โฟลเดอร์ `git-course` ที่เตรียมไว้ตั้งแต่ Part 01):

1. สร้าง repository ทดลองบน GitHub พร้อมโครงสร้างโฟลเดอร์ใกล้เคียงกับโจทย์ข้างต้น
2. สร้าง GitHub Organization ทดลอง (ฟรี) แล้วสร้าง 3 ทีมตามโจทย์
3. เชิญบัญชีทดลอง (หรือใช้บัญชีตัวเองในหลายทีม) เข้าเป็นสมาชิกแต่ละทีม พร้อมให้สิทธิ์ write access กับ repository
4. เขียนไฟล์ `.github/CODEOWNERS` ตามเฉลยด้านบน แล้ว push ขึ้นไป
5. เปิด Branch Protection Rule สำหรับ `main` และเปิดใช้ "Require review from Code Owners"
6. เปิด PR ทดสอบที่แก้ไฟล์ในแต่ละโฟลเดอร์ทีละกลุ่ม แล้วตรวจสอบว่า reviewer ที่ถูก auto-assign ตรงกับตารางคำตอบด้านบนหรือไม่
7. ลองกด Merge ก่อนที่ owner จะ approve เพื่อยืนยันว่าปุ่มถูกบล็อกไว้จริง

ถ้าทำครบทั้ง 7 ขั้นตอนนี้ได้ผลลัพธ์ตรงกับตารางคำตอบทุกแถว แสดงว่าคุณเข้าใจกลไกของ CODEOWNERS อย่างถ่องแท้แล้ว และพร้อมนำไปใช้กับ repository จริงในที่ทำงานได้ทันที

---

## สรุป Part 36

ใน Part นี้เราได้เรียนรู้ว่า:

1. **CODEOWNERS แก้ปัญหา "ไม่รู้ว่าควรขอ review จากใคร"** โดยเปลี่ยนคำถามนั้นให้เป็นกฎที่ระบบ auto-request reviewer ให้อัตโนมัติ แทนที่จะพึ่งความจำหรือการถามใน chat
2. ไฟล์ต้องชื่อ `CODEOWNERS` เป๊ะ ๆ และวางได้เพียง 3 ตำแหน่งคือ `.github/CODEOWNERS`, root, หรือ `docs/CODEOWNERS` โดย GitHub จะใช้ไฟล์แรกที่เจอตามลำดับนี้เท่านั้น
3. Pattern matching ใช้หลักการคล้าย `.gitignore` — `*` ไม่ข้ามระดับโฟลเดอร์, `**` ข้ามได้ทุกระดับ, `/` ขึ้นต้นหมายถึง absolute path จาก root
4. **Team-based ownership (`@org/team`) ควรเป็นค่าเริ่มต้น** สำหรับโค้ดที่มีคนดูแลมากกว่า 1 คน เพราะลด Bus Factor และไม่ต้องแก้ไฟล์เมื่อคนย้ายทีม ส่วน Individual ownership ควรสงวนไว้เฉพาะกรณีพิเศษ
5. **ไฟล์ CODEOWNERS เพียงอย่างเดียวไม่ได้บังคับอะไรเลย** — ต้องเปิด "Require review from Code Owners" ใน Branch Protection Rules ควบคู่กัน จึงจะบล็อกปุ่ม Merge ได้จริงจนกว่าเจ้าของจะ approve
6. **กฎที่ match บรรทัดล่างสุดในไฟล์เท่านั้นที่มีผล** (last matching pattern wins) ไม่ใช่การรวมกฎจากหลายบรรทัด ดังนั้นต้องเรียง pattern จากกว้างไปแคบเสมอ
7. การทดสอบ CODEOWNERS ต้องทำผ่าน PR จริงเสมอ เพราะระบบไม่มี error ที่ชัดเจนเวลาเขียนผิด — มันจะ fail แบบเงียบ ๆ
8. สำหรับ Monorepo หลายทีม ควรจัดกลุ่มด้วย comment header, ตั้ง default owner เสมอ, ออกแบบโครงสร้างโฟลเดอร์ให้สอดคล้องกับขอบเขตทีม และกำหนดเจ้าของให้ไฟล์ CODEOWNERS เองด้วย
9. ข้อจำกัดสำคัญที่ต้องระวังคือ: ไม่มี negation pattern, จำกัดจำนวนทีมต่อ pattern, ไม่ทำงานย้อนหลังกับ PR เก่า, และไม่ trigger บน Draft PR

### Checklist ก่อนไป Part 37

ก่อนไปต่อ Part 37 ให้ตรวจสอบว่าคุณ:

- [ ] อธิบายได้ว่า CODEOWNERS แก้ปัญหาอะไรให้ทีมที่โตขึ้น
- [ ] รู้ว่าไฟล์ CODEOWNERS วางได้ที่ไหนบ้าง และ GitHub เลือกไฟล์ไหนเมื่อมีซ้ำกัน
- [ ] เขียน pattern matching ได้ถูกต้อง แยกความแตกต่างระหว่าง `*`, `**`, และ absolute path ที่ขึ้นต้นด้วย `/`
- [ ] เข้าใจข้อดี-ข้อเสียของ Team-based ownership เทียบกับ Individual ownership
- [ ] รู้ว่าต้องเปิด "Require review from Code Owners" ใน Branch Protection เพื่อให้ CODEOWNERS บังคับ merge ได้จริง
- [ ] เข้าใจหลัก "บรรทัดล่างสุดที่ match ชนะ" และเรียง pattern จากกว้างไปแคบได้ถูกต้อง
- [ ] รู้วิธีทดสอบว่า CODEOWNERS ทำงานถูกต้องผ่านการเปิด PR จริง
- [ ] ทำแบบฝึกหัด Step 360 เสร็จ และได้ผลลัพธ์ตรงกับตารางคำตอบทุกแถว

**ต่อไป:** [Part 37: Branch Protection Rules และ Merge Strategies](./part-037-branch-protection-merge-strategies.md)

