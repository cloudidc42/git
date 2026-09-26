# Part 37: Branch Protection Rules และ Merge Strategies

> **Step ในหลักสูตรนี้:** Step 361–370
> **เฟส:** 4 — ทำงานเป็นทีมด้วย Workflow มาตรฐาน (Git Flow, GitHub Flow, Rebase)
> **เป้าหมายของ Part นี้:** เข้าใจว่า Branch Protection Rules (และ Rulesets รุ่นใหม่ของ GitHub) คืออะไร ทำไมทีมที่ทำงานจริงจังทุกทีมต้องตั้งค่าเหล่านี้ให้กับ branch หลัก เข้าใจการตั้งค่าแต่ละตัวอย่างละเอียด ตั้งแต่การบังคับ Pull Request, การอนุมัติ, การเชื่อมกับ CI, ไปจนถึงการจำกัดสิทธิ์การ push และเข้าใจความแตกต่างของ Merge Strategies ทั้ง 3 แบบ (Merge commit, Squash and merge, Rebase and merge) อย่างลึกซึ้งพอที่จะเลือกใช้ให้เหมาะกับทีมของตัวเองได้

---

## สารบัญของ Part นี้

- Step 361: Branch Protection Rules คืออะไร ทำไมสำคัญกับทีม
- Step 362: "Require a pull request before merging" ป้องกันการ push ตรงเข้า main
- Step 363: "Require approvals" (จำนวนผู้อนุมัติขั้นต่ำ และ dismiss stale reviews)
- Step 364: "Require status checks to pass before merging" เชื่อมกับ CI
- Step 365: "Require conversation resolution before merging"
- Step 366: "Require signed commits"
- Step 367: "Restrict who can push to matching branches"
- Step 368: Merge Strategies เจาะลึก — Merge commit, Squash and merge, Rebase and merge
- Step 369: การตั้งค่า Allowed Merge Methods ระดับ repository
- Step 370: แบบฝึกหัด — ตั้งค่า Branch Protection เต็มรูปแบบให้ main branch ของโปรเจกต์จำลอง

---

## Step 361: Branch Protection Rules คืออะไร ทำไมสำคัญกับทีม

ใน Part ก่อน ๆ เราเรียนรู้เรื่อง Pull Request, Code Review, CODEOWNERS มาแล้ว ซึ่งทั้งหมดนี้ตั้งอยู่บนสมมติฐานว่า **ทุกคนในทีมจะทำตามกระบวนการที่ตกลงกันไว้อย่างตั้งใจ** — แต่ในโลกความเป็นจริง สมมติฐานแบบนั้นไม่ปลอดภัยพอ เพราะ:

- คนอาจลืม เผลอ push ตรงเข้า `main` โดยไม่ได้ตั้งใจ (เช่น พิมพ์ `git push origin main` ผิดมือจาก branch อื่น)
- คนใหม่ในทีมอาจยังไม่รู้ workflow ที่ตกลงกันไว้
- ในสถานการณ์เร่งด่วน คนอาจ "ลัดขั้นตอน" เพื่อความเร็ว โดยไม่รู้ตัวว่ากำลังทำลายความปลอดภัยของโค้ด production
- ถ้าไม่มีการบังคับใด ๆ เลย ระบบ Code Review ที่ตั้งไว้ก็เป็นแค่ "ข้อตกลงปากเปล่า" ที่ไม่มีอะไรค้ำประกัน

**Branch Protection Rules** คือฟีเจอร์ของ GitHub ที่ทำหน้าที่:

> **บังคับ (enforce) กฎบางอย่างในระดับ repository ให้กับ branch ที่ระบุไว้ (เช่น `main`, `release/*`) โดยไม่มีใครสามารถหลบเลี่ยงได้ ไม่ว่าจะตั้งใจหรือไม่ตั้งใจ (ยกเว้นจะได้รับสิทธิ์ยกเว้นชัดเจน)**

พูดง่าย ๆ คือ Branch Protection Rules คือ **"รั้วนิรภัย" ที่ทำให้กระบวนการที่ทีมตกลงกันไว้กลายเป็นกฎที่ระบบบังคับใช้จริง ไม่ใช่แค่ธรรมเนียมปฏิบัติ**

### ทำไมมันสำคัญมากสำหรับทีม

| ปัญหาที่เกิดถ้าไม่มี Branch Protection | ผลลัพธ์ที่ Branch Protection ป้องกันได้ |
|---|---|
| มีคน push โค้ดที่ยังไม่ผ่านการรีวิวเข้า `main` โดยตรง | บังคับให้ทุกการเปลี่ยนแปลงต้องผ่าน Pull Request |
| โค้ดที่ทดสอบไม่ผ่าน (test แดง) ถูก merge เข้าไปเฉย ๆ | บังคับให้ CI ต้องผ่านก่อน merge ได้ |
| คนคนเดียวอนุมัติโค้ดของตัวเอง ไม่มีใครตรวจสอบ | บังคับจำนวนผู้อนุมัติขั้นต่ำจากคนอื่น |
| แก้โค้ดเพิ่มหลัง review ผ่านแล้ว แต่ไม่มีใครรีวิวซ้ำ | บังคับ dismiss การอนุมัติเก่าเมื่อมี commit ใหม่ |
| มีคนถามคำถามใน PR แต่ merge ไปโดยไม่ตอบ | บังคับให้ต้องปิดทุก conversation ก่อน merge |
| ประวัติ commit ปลอมตัวเป็นคนอื่นได้ | บังคับให้ commit ต้องมีลายเซ็นดิจิทัล (signed) |
| ใครก็ push เข้า branch สำคัญได้หมด | จำกัดให้เฉพาะบางคน/บางทีมเท่านั้นที่ push ได้ |

### Branch Protection Rules แบบ "Classic" vs "Rulesets" (สำคัญมาก ต้องเข้าใจให้ตรงกับปัจจุบัน)

ปัจจุบัน GitHub มีวิธีตั้งค่ากฎเหล่านี้อยู่ **2 ระบบคู่ขนานกัน**:

1. **Branch protection rules (Classic)** — ระบบดั้งเดิมที่อยู่ใน **Settings → Branches** ของ repository ตั้งค่าโดยระบุ branch name pattern (เช่น `main`, `release/*`) แล้วเลือกกฎที่ต้องการ เป็นระบบที่ใช้กันมานานและยังใช้งานได้เต็มรูปแบบในปัจจุบัน (2026)
2. **Rulesets** — ระบบใหม่กว่าที่อยู่ใน **Settings → Rules → Rulesets** ซึ่ง GitHub แนะนำให้ใช้แทน Classic Branch Protection ในโปรเจกต์ใหม่ เพราะยืดหยุ่นกว่ามาก:
   - ตั้งกฎได้ทั้งกับ **Branch** และ **Tag** ในที่เดียว
   - เลือก **target ได้หลายรูปแบบพร้อมกัน** เช่น "branch ทุกอันยกเว้น `main`" หรือ "branch ที่ตรงกับ fnmatch pattern หลายแบบ"
   - มี **Bypass list** ที่ยืดหยุ่นกว่า (กำหนดได้ละเอียดว่าใคร/ทีมไหน/app ไหน ข้ามกฎได้ในสถานการณ์ไหน)
   - ดู **Insights / History** ของการเปลี่ยนแปลง ruleset ย้อนหลังได้ (audit trail ในตัว)
   - รองรับการตั้งเป็น **Organization-level ruleset** ที่บังคับใช้กับหลาย repository พร้อมกันในองค์กรเดียว โดยไม่ต้องตั้งซ้ำทีละ repo
   - รองรับสถานะ **Active / Evaluate (dry-run)** — เปิดโหมด "ประเมินผลก่อนบังคับใช้จริง" ได้ ทำให้ทดสอบกฎก่อนเปิดใช้งานจริงได้อย่างปลอดภัย

ในทางปฏิบัติ **แนวคิดของกฎแต่ละข้อ (require PR, require approvals, require status checks ฯลฯ) เหมือนกันทั้งสองระบบ** ต่างกันแค่ตำแหน่งเมนูและความยืดหยุ่นในการกำหนด target เท่านั้น เนื้อหาใน Part นี้จะอธิบายกฎแต่ละข้อโดยอิงตามแนวคิดที่ใช้ร่วมกันทั้งสองระบบ และจะบอกด้วยว่าแต่ละกฎอยู่ตรงไหนในทั้งสองเมนู เพื่อให้คุณใช้งานได้ไม่ว่าจะเจอ repository ที่ใช้ระบบไหนก็ตาม

### ตำแหน่งเมนูของทั้งสองระบบ

**Classic Branch Protection Rules:**

```
Repository → Settings → Branches → Branch protection rules → Add rule
```

**Rulesets (ระบบใหม่):**

```
Repository → Settings → Rules → Rulesets → New branch ruleset
```

หรือในระดับองค์กร:

```
Organization → Settings → Repository → Rulesets → New ruleset
```

### ใครสามารถตั้งค่า Branch Protection ได้บ้าง

- **Repository ส่วนตัว (personal):** เจ้าของ repository เท่านั้น
- **Organization:** ต้องมีสิทธิ์ **Admin** ของ repository นั้น หรือเป็น **Owner** ขององค์กร (สำหรับ Organization-level ruleset)

ทีมทั่วไปจะให้สิทธิ์นี้เฉพาะ Tech Lead, Engineering Manager หรือ DevOps/Platform team เท่านั้น เพราะการเปลี่ยนกฎเหล่านี้กระทบความปลอดภัยของทั้งโปรเจกต์

จาก Step ถัดไป เราจะไล่ดูกฎแต่ละข้อทีละตัวอย่างละเอียด โดยเริ่มจากกฎที่สำคัญที่สุดและควรตั้งเป็นอันดับแรกสำหรับทุกทีม

---

## Step 362: "Require a pull request before merging" ป้องกันการ push ตรงเข้า main

นี่คือกฎที่ **สำคัญที่สุดและควรตั้งเป็นอันดับแรกเสมอ** สำหรับทุก repository ที่ทำงานเป็นทีม

### คืออะไร

เมื่อเปิดใช้งานกฎนี้ ระบบจะ **ปฏิเสธการ push โดยตรง (direct push)** เข้าไปยัง branch ที่ถูกป้องกันไว้ (เช่น `main`) ไม่ว่าคนที่ push จะมีสิทธิ์ Write หรือแม้แต่ Admin ก็ตาม (เว้นแต่จะอยู่ใน bypass list) — **ทุกการเปลี่ยนแปลงต้องมาในรูปแบบ Pull Request เท่านั้น**

### ทำไมกฎนี้ถึงสำคัญที่สุด

ลองนึกภาพ repository ที่ **ไม่มี** กฎนี้:

```bash
# นักพัฒนา A กำลังทำงานอยู่บน branch ของตัวเอง
git checkout feature/payment-fix

# แต่พิมพ์ผิด เผลอสลับไป main
git checkout main

# แล้ว push โค้ดที่ยังไม่เสร็จ ยังไม่ได้ทดสอบ เข้า main โดยตรง
git push origin main
```

หากไม่มีกฎนี้ โค้ดที่ยังไม่พร้อมจะเข้าไปอยู่บน `main` ทันที **โดยไม่มีใครรีวิว ไม่มีการทดสอบอัตโนมัติตรวจสอบ** — และถ้า `main` คือ branch ที่ใช้ deploy ขึ้น production โดยอัตโนมัติ (ซึ่งเป็นเรื่องปกติมากในทีมสมัยใหม่) นี่คือหายนะที่เกิดขึ้นได้ในเสี้ยววินาที

เมื่อเปิดกฎนี้ คำสั่งด้านบนจะได้ผลลัพธ์แบบนี้แทน:

```
$ git push origin main
remote: error: GH006: Protected branch update failed for refs/heads/main.
remote: error: Changes must be made through a pull request.
To github.com:my-org/my-repo.git
 ! [remote rejected] main -> main (protected branch hook declined)
error: failed to push some refs to 'github.com:my-org/my-repo.git'
```

Git และ GitHub **ปฏิเสธการ push ทันที** และบังคับให้ต้องเปิด Pull Request แทน — นี่คือ "รั้วนิรภัย" ตัวแรกที่ทำให้ทุกการเปลี่ยนแปลงต้องผ่านกระบวนการที่ทีมออกแบบไว้เสมอ

### วิธีตั้งค่า (Classic Branch Protection)

1. ไปที่ **Settings → Branches**
2. คลิก **Add branch protection rule** (หรือแก้ไข rule เดิมถ้ามีแล้ว)
3. ใส่ **Branch name pattern** เช่น `main` (หรือ `release/*` เพื่อครอบคลุมหลาย branch ด้วย wildcard)
4. ติ๊กเลือก **☑ Require a pull request before merging**
5. คลิก **Create** หรือ **Save changes**

### วิธีตั้งค่า (Rulesets)

1. ไปที่ **Settings → Rules → Rulesets → New branch ruleset**
2. ตั้งชื่อ ruleset เช่น `Protect main`
3. ที่ **Enforcement status** เลือก **Active**
4. ที่ **Target branches** เลือก **Include default branch** หรือระบุ pattern เอง
5. ในส่วน **Branch rules** ติ๊กเลือก **Require a pull request before merging**
6. กด **Create**

### รายละเอียดย่อยที่ควรรู้: "Do not allow bypassing the above settings"

มีตัวเลือกสำคัญที่มักถูกมองข้ามคือ **"Do not allow bypassing the above settings"** — ถ้า**ไม่ติ๊ก** ตัวเลือกนี้ ผู้ที่มีสิทธิ์ **Repository administrator** จะยังสามารถ push ตรงเข้า branch ที่ถูกป้องกันได้อยู่ (เป็นทางออกฉุกเฉิน) แต่ถ้า**ติ๊ก** จะไม่มีใครหลบเลี่ยงกฎนี้ได้เลย แม้แต่ admin

ทีมที่ทำงานจริงจังส่วนใหญ่แนะนำให้ **ติ๊กตัวเลือกนี้ไว้เสมอสำหรับ `main`** เพราะ:

- ป้องกันความผิดพลาดของ admin เองด้วย (คนก็คือคนเผลอได้เหมือนกัน)
- สร้างวัฒนธรรมที่ชัดเจนว่า "ไม่มีใครใหญ่กว่ากระบวนการ"
- ถ้าจำเป็นต้องแก้ไขฉุกเฉินจริง ๆ ควรใช้ **Pull Request แบบเร่งด่วน (fast-track PR)** ที่ยังผ่านการรีวิวอย่างน้อย 1 คน ดีกว่าการข้ามกระบวนการไปเลย

### ผลกระทบต่อ workflow ประจำวัน

หลังเปิดกฎนี้ workflow มาตรฐานของทุกคนในทีมจะกลายเป็นแบบนี้เสมอ:

```bash
git checkout -b feature/add-search
# ... แก้โค้ด ...
git add .
git commit -m "feat: add search functionality"
git push origin feature/add-search
# แล้วเปิด Pull Request บน GitHub เพื่อขอ merge เข้า main
```

ไม่มีใครสามารถ `git push origin main` ได้อีกต่อไป — ทุกการเปลี่ยนแปลงต้องผ่าน PR เสมอ

---

## Step 363: "Require approvals" (จำนวนผู้อนุมัติขั้นต่ำ และ dismiss stale reviews)

การบังคับให้ใช้ Pull Request อย่างเดียว (Step 362) ยังไม่พอ — เพราะในทางเทคนิค คนที่เปิด PR สามารถกด **Merge** เองได้ทันทีโดยไม่มีใครรีวิวเลย ถ้าไม่มีการบังคับเรื่องการอนุมัติเพิ่มเติม

### "Require approvals" คืออะไร

กฎนี้บังคับว่า **ก่อนที่ปุ่ม Merge จะกดได้ ต้องมีผู้รีวิวคนอื่นกด Approve อย่างน้อยตามจำนวนที่กำหนดไว้ก่อน**

ตั้งค่าอยู่ใต้ **☑ Require a pull request before merging** เมื่อติ๊กแล้วจะมี sub-option ให้เลือกเพิ่ม:

```
☑ Require a pull request before merging
    ☑ Require approvals
        Required number of approvals before merging: [1] ▼ (เลือกได้ตั้งแต่ 1–6)
    ☑ Dismiss stale pull request approvals when new commits are pushed
    ☑ Require review from Code Owners
    ☐ Require approval of the most recent reviewable push
    ☑ Require conversation resolution before merging
```

### กำหนดจำนวนผู้อนุมัติขั้นต่ำ (Required number of approvals)

GitHub ให้เลือกได้ตั้งแต่ 1 ถึง 6 คน ค่าที่นิยมใช้ในทีมจริง:

| จำนวนที่ตั้ง | เหมาะกับทีมแบบไหน |
|---|---|
| **1 คน** | ทีมเล็ก (2–5 คน), startup ระยะเริ่มต้น ที่ต้องการความเร็วแต่ยังอยากมีการตรวจสอบขั้นต่ำ |
| **2 คน** | ทีมขนาดกลางถึงใหญ่ (5–20 คน), โปรเจกต์ที่มีผลกระทบสูง เช่น payment, authentication |
| **3+ คน** | โค้ดที่กระทบระบบ core/infrastructure ระดับองค์กร, โอเพนซอร์สขนาดใหญ่ที่ต้องการ consensus สูง |

**ข้อควรระวัง:** ผู้เปิด PR (author) **ไม่สามารถอนุมัติ PR ของตัวเองได้** — GitHub บังคับกฎนี้โดยอัตโนมัติเสมอ ไม่ว่าจะตั้งค่าอะไรก็ตาม เพื่อป้องกันการ "รีวิวตัวเอง" ที่ไร้ความหมาย

### "Dismiss stale pull request approvals when new commits are pushed" — สำคัญมากแต่คนมักลืมเปิด

นี่คือกฎที่ป้องกันช่องโหว่ร้ายแรงที่สุดข้อหนึ่งของกระบวนการ Code Review ลองดูสถานการณ์นี้:

1. นักพัฒนา A เปิด PR แก้ไข bug เล็กน้อย
2. นักพัฒนา B รีวิวโค้ด เห็นว่าโอเค กด **Approve**
3. นักพัฒนา A **push commit เพิ่มเติม** เข้าไปใน PR เดียวกัน (อาจเป็นการแก้ไขเพิ่มเติมที่ไม่เกี่ยวข้องกับที่รีวิวไปแล้วเลย)
4. ถ้าไม่มีกฎนี้ **การ approve เดิมของ B ยังคงมีผลอยู่** ทำให้ PR สามารถ merge ได้ทันทีโดยไม่มีใครเห็น commit ใหม่ที่เพิ่งถูกเพิ่มเข้ามาเลย

เมื่อเปิด **"Dismiss stale pull request approvals when new commits are pushed"** ทุกครั้งที่มี commit ใหม่ push เข้า PR **การอนุมัติเดิมทั้งหมดจะถูกยกเลิกโดยอัตโนมัติ (dismissed)** และต้องขอ approve ใหม่อีกครั้งก่อน merge ได้

```
เหตุการณ์:                                    สถานะ Approval:
─────────────────────────────────────────────────────────────
1. A เปิด PR                                   (ยังไม่มี approval)
2. B รีวิว กด Approve                          ✅ Approved by B
3. A push commit ใหม่เพิ่ม                      ⚠️ Approval ถูก dismiss อัตโนมัติ
4. สถานะ PR กลับไปเป็น "รอการอนุมัติใหม่"        ❌ ต้อง approve ใหม่อีกครั้ง
```

**ข้อควรรู้:** กฎนี้จะ dismiss การอนุมัติเมื่อมี **commit ใหม่** เท่านั้น ไม่รวมการ push แบบ force-push ที่ rebase โค้ดเดิมโดยไม่มีการเปลี่ยนแปลงเนื้อหาจริง (แต่ในทางปฏิบัติ GitHub มักตีความอย่างระมัดระวังและ dismiss ในเกือบทุกกรณีของการ push ใหม่อยู่ดี เพื่อความปลอดภัย)

### "Require review from Code Owners"

ถ้าติ๊กตัวเลือกนี้ (ซึ่งเราเรียนรายละเอียดของไฟล์ `CODEOWNERS` ไปแล้วใน Part ก่อนหน้า) ระบบจะบังคับว่า **ไฟล์ที่มี Code Owner กำหนดไว้ ต้องได้รับการอนุมัติจาก Code Owner ของไฟล์นั้นด้วยเสมอ** ไม่ใช่แค่จำนวนคนอนุมัติทั่วไปตามที่ตั้งไว้ในข้อก่อนหน้า

ตัวอย่างเช่น ถ้า `CODEOWNERS` กำหนดว่า:

```
/payment/  @finance-team
```

แม้จะตั้ง "Required number of approvals" ไว้แค่ 1 คน แต่ถ้า PR แก้ไขไฟล์ในโฟลเดอร์ `/payment/` ระบบจะไม่ยอมให้ merge จนกว่าจะมีสมาชิกจาก `@finance-team` มาอนุมัติจริง ๆ

### "Require approval of the most recent reviewable push"

นี่คือตัวเลือกที่ตั้งขึ้นมาให้เข้มงวดยิ่งกว่า dismiss stale reviews อีกขั้น: บังคับว่า **การ push ล่าสุดที่มีเนื้อหาให้รีวิว (ไม่ใช่แค่ merge commit ของ base branch) ต้องได้รับการอนุมัติจากคนอื่นที่ไม่ใช่ผู้ push คนนั้นเอง** ก่อนจะ merge ได้ — ป้องกันกรณีที่ reviewer คนหนึ่ง approve ไว้ แล้ว author เอง push แก้ไขเล็กน้อยแล้วรีบ merge เองในจังหวะที่ approval ยังไม่ถูก dismiss (ในเวอร์ชันเก่าของ GitHub)

### สรุปการตั้งค่าที่แนะนำสำหรับทีมส่วนใหญ่

```
☑ Require a pull request before merging
    Required number of approvals: 1–2 (ตามขนาดทีม)
    ☑ Dismiss stale pull request approvals when new commits are pushed
    ☑ Require review from Code Owners
```

---

## Step 364: "Require status checks to pass before merging" เชื่อมกับ CI

กฎก่อนหน้าทั้งหมดเป็นเรื่องของ "คนตรวจสอบคน" แต่กฎนี้คือการเปิดทางให้ **"เครื่องตรวจสอบคน"** — คือการเชื่อม Branch Protection เข้ากับระบบ **Continuous Integration (CI)** เพื่อบังคับว่าโค้ดต้องผ่านการทดสอบอัตโนมัติก่อน merge ได้เสมอ

### คืออะไร

เมื่อเปิดใช้งาน **"Require status checks to pass before merging"** ปุ่ม **Merge pull request** บน GitHub จะถูก **disable (กดไม่ได้)** จนกว่า **status checks** ทั้งหมดที่กำหนดไว้จะรายงานผลเป็น **✅ success**

Status checks เหล่านี้มาจากระบบภายนอกที่รายงานสถานะกลับมาที่ GitHub ผ่าน **Commit Status API** หรือ **Checks API** ซึ่งส่วนใหญ่ในปัจจุบันคือ:

- **GitHub Actions** (เครื่องมือ CI/CD ในตัวของ GitHub เอง — เราจะเรียนแบบเจาะลึกเต็มรูปแบบใน **Part 66–70**)
- ระบบ CI ภายนอกอื่น ๆ ที่เชื่อมผ่าน webhook เช่น CircleCI, Jenkins, Travis CI
- เครื่องมือตรวจสอบคุณภาพโค้ดอัตโนมัติ เช่น SonarQube, CodeQL, Codecov

### ตัวอย่างหน้าตาบน Pull Request เมื่อเปิดกฎนี้

```
Checks (3)

  ✅ build / test (push)                   Successful in 45s
  ✅ lint / eslint (push)                   Successful in 12s
  ❌ test / unit-tests (push)               Failing after 1m 30s

  Some checks were not successful
  1 failing check

  [ Merge pull request ]  ← ปุ่มนี้จะถูก disable โดยอัตโนมัติ
```

ตราบใดที่มี check ใดก็ตามที่กำหนดว่า "required" ยังไม่ผ่าน (ไม่ว่าจะ failing หรือกำลัง pending) ปุ่ม Merge จะกดไม่ได้เลย ไม่มีข้อยกเว้น (ยกเว้นจะมีคนอยู่ใน bypass list และเลือกใช้สิทธิ์นั้น)

### วิธีตั้งค่า

1. เปิด **☑ Require status checks to pass before merging**
2. ในช่องค้นหา ให้พิมพ์ชื่อ check ที่ต้องการบังคับ เช่น `build`, `test`, `lint` — **ชื่อ check เหล่านี้จะปรากฏในรายการก็ต่อเมื่อมันเคยรันบน repository นี้อย่างน้อยหนึ่งครั้งมาก่อนแล้วเท่านั้น** (ต้องมี workflow ที่รันจริงมาก่อน ระบบถึงจะรู้จักชื่อนั้น)
3. เลือก check ที่ต้องการ "บังคับ" (required) จากรายการ — check ที่ไม่ได้เลือกจะยังรันอยู่และแสดงผลให้เห็น แต่จะไม่บล็อกการ merge

### "Require branches to be up to date before merging"

มีตัวเลือกย่อยที่สำคัญคือ **"Require branches to be up to date before merging"** ถ้าเปิดตัวเลือกนี้ Pull Request จะต้อง **merge หรือ rebase เอาโค้ดล่าสุดของ base branch (เช่น `main`) เข้ามาก่อน** ถึงจะ merge ได้ แม้ว่า status check ของ PR เองจะผ่านหมดแล้วก็ตาม

เหตุผลที่ต้องมีกฎนี้คือปัญหาที่เรียกว่า **"Semantic Merge Conflict"** — สถานการณ์ที่ PR สองอันแยกกันทดสอบผ่านหมด แต่พอรวมกันจริง ๆ กลับทำงานผิดพลาด เช่น:

```
PR #1 (ผ่าน CI): ลบฟังก์ชัน calculateTax() ออกจากไฟล์ tax.js
PR #2 (ผ่าน CI): เพิ่มโค้ดที่เรียกใช้ calculateTax() ในไฟล์ checkout.js

ถ้า merge PR #2 เข้าไปหลัง PR #1 merge แล้ว โดยไม่เอา main ล่าสุดมา rebase/merge ก่อน
→ CI ของ PR #2 ที่เคยผ่าน (ตอนที่ calculateTax() ยังอยู่) จะไม่ได้ทดสอบกับสถานการณ์จริงหลัง merge
→ โค้ดพังทันทีหลัง merge แม้ทั้งสอง PR จะ "ผ่าน CI" แยกกัน
```

การเปิด "Require branches to be up to date" บังคับให้ผู้เปิด PR ต้อง sync กับ `main` ล่าสุดก่อนเสมอ (ผ่านการกด **Update branch** บน GitHub หรือ `git merge main` / `git rebase main` ในเครื่อง) ทำให้ CI ที่รันครั้งสุดท้ายสะท้อนสถานการณ์จริงหลัง merge ได้แม่นยำกว่ามาก

**ข้อควรรู้ในปี 2026:** GitHub แนะนำแนวทางใหม่กว่าที่เรียกว่า **Merge Queue** (คิวการ merge อัตโนมัติ) ซึ่งแก้ปัญหานี้ได้ดีกว่าการบังคับ "update ก่อน merge" แบบ manual เพราะ Merge Queue จะทดสอบ PR แต่ละอันร่วมกับสถานะล่าสุดของ `main` โดยอัตโนมัติก่อน merge จริงเสมอ โดยไม่ต้องให้ผู้เขียน PR มา update branch เอง — เราจะเรียนเรื่อง Merge Queue อย่างละเอียดใน Part ที่เกี่ยวกับ CI/CD ขั้นสูงใน **เฟส 7 (Step 651–750)**

### ทำไมกฎนี้ถึงเป็นหัวใจของ "Shift-Left Testing"

การบังคับ status check ก่อน merge คือรากฐานของแนวคิด **CI/CD (Continuous Integration / Continuous Deployment)** ที่ทีมซอฟต์แวร์สมัยใหม่แทบทุกทีมยึดถือ:

> ยิ่งพบปัญหาเร็วเท่าไหร่ (ก่อน merge แทนที่จะเป็นหลัง deploy) ต้นทุนในการแก้ไขยิ่งถูกลงเท่านั้น

เราจะเรียนรู้การสร้าง GitHub Actions Workflow เพื่อสร้าง status check เหล่านี้เองอย่างละเอียดใน **Part 66 ถึง Part 70** ซึ่งจะครอบคลุมตั้งแต่การเขียน workflow YAML พื้นฐาน ไปจนถึงการสร้าง pipeline ทดสอบ build lint และ deploy อัตโนมัติเต็มรูปแบบ

---

## Step 365: "Require conversation resolution before merging"

กฎนี้เล็กแต่สำคัญมากในทางปฏิบัติ และมักถูกมองข้ามโดยทีมที่เพิ่งเริ่มตั้งค่า Branch Protection

### คืออะไร

เมื่อเปิด **"Require conversation resolution before merging"** ระบบจะบังคับว่า **ทุก comment thread (การสนทนา) ที่เปิดไว้บน Pull Request ต้องถูกกด "Resolve conversation" ให้ครบทุกอันก่อน จึงจะสามารถกด Merge ได้**

### ปัญหาที่กฎนี้แก้

ลองนึกภาพสถานการณ์นี้ที่เกิดขึ้นบ่อยมากในทีมจริง:

```
Reviewer B: "ตรงนี้ทำไมไม่ใช้ Promise.all() แทนการ await ทีละตัวในลูปครับ 
             จะได้เร็วขึ้นเยอะ"

Author A: (เห็น comment แต่กำลังยุ่ง ลืมตอบ)

... 3 วันผ่านไป ...

Author A: กด Merge เพราะ CI ผ่านหมดแล้ว และมีคน Approve แล้ว
```

ถ้าไม่มีกฎนี้ **คำถามหรือข้อเสนอแนะที่สำคัญของ reviewer อาจถูกละเลยไปโดยสิ้นเชิง** แม้ว่า reviewer จะกด "Approve" ไปแล้วก็ตาม (เพราะใน GitHub การ Approve กับการ comment เป็นคนละกลไกกัน — reviewer สามารถ Approve พร้อมทิ้ง comment ที่ยังไม่ได้รับคำตอบไว้ได้)

เมื่อเปิดกฎนี้ หน้า Pull Request จะแสดงข้อความแบบนี้จนกว่าจะแก้ไข:

```
⚠️  1 unresolved conversation must be resolved before merging

    [ Merge pull request ]  ← ปุ่มถูก disable
```

Author ต้องกลับไปตอบคำถาม แก้โค้ดตามที่ถูกขอ (หรืออธิบายเหตุผลว่าทำไมไม่แก้) แล้วให้ reviewer (หรือใครก็ตามที่เกี่ยวข้อง) กด **Resolve conversation** ในทุก thread ก่อนถึงจะ merge ได้

### ใครสามารถกด "Resolve conversation" ได้บ้าง

- ผู้ที่เริ่มการสนทนา (คนที่ comment ก่อน)
- ผู้เขียน PR (author)
- ผู้ที่มีสิทธิ์ Write ขึ้นไปใน repository นั้น

**ข้อควรระวัง:** กฎนี้ **ไม่ได้ตรวจสอบว่าคำตอบนั้น "ดีพอ" หรือไม่** มันแค่บังคับว่าต้องมีการกด resolve เท่านั้น — ดังนั้นวัฒนธรรมทีมยังคงสำคัญ: ทีมที่ดีจะกด resolve ก็ต่อเมื่อข้อกังวลนั้นได้รับการแก้ไขหรืออธิบายจริง ๆ ไม่ใช่กด resolve ทิ้งเพื่อให้ merge ได้เฉย ๆ

### ทำไมควรเปิดกฎนี้ควบคู่กับ Require approvals เสมอ

Require approvals ตรวจสอบว่า "มีคนอนุมัติหรือยัง" แต่ไม่ได้ตรวจสอบว่า "คำถามที่ถูกถามได้รับคำตอบหรือยัง" — สองกฎนี้ทำงานเสริมกันคนละมุม:

| กฎ | ตรวจสอบอะไร |
|---|---|
| Require approvals | มีคน "เห็นด้วย" กับ PR นี้ครบตามจำนวนหรือยัง |
| Require conversation resolution | ทุกข้อกังวล/คำถามที่ถูกหยิบยกขึ้นมา ได้รับการจัดการแล้วหรือยัง |

ทีมที่จริงจังเรื่องคุณภาพโค้ดควรเปิดทั้งสองกฎควบคู่กันเสมอ

---

## Step 366: "Require signed commits"

นี่คือกฎที่เกี่ยวข้องกับ **ความปลอดภัยของประวัติ (history integrity)** ในระดับที่ลึกกว่ากฎอื่น ๆ ที่ผ่านมา

### คืออะไร

เมื่อเปิด **"Require signed commits"** ระบบจะบังคับว่า **ทุก commit ที่จะ push เข้า branch ที่ถูกป้องกัน ต้องมีลายเซ็นดิจิทัล (digital signature) ที่ตรวจสอบยืนยันตัวตนได้ (verified)** เท่านั้น — commit ที่ไม่มีลายเซ็น หรือมีลายเซ็นที่ตรวจสอบไม่ผ่าน จะถูกปฏิเสธทันที

### ทำไมเรื่องนี้ถึงสำคัญ

หนึ่งในข้อเท็จจริงที่มือใหม่ Git มักไม่รู้คือ: **ข้อมูล "author" และ "committer" ใน Git commit นั้น ปลอมแปลงได้ง่ายมาก** เพราะมันเป็นแค่ข้อความธรรมดาที่มาจากค่า `user.name` และ `user.email` ใน config ของเครื่องผู้ commit เอง

```bash
# ใครก็ตามสามารถตั้งค่าปลอมตัวเป็นคนอื่นได้ง่าย ๆ แบบนี้
git config user.name "Linus Torvalds"
git config user.email "torvalds@linux-foundation.org"
git commit -m "แก้บั๊กด่วน"

# commit นี้จะแสดงชื่อ "Linus Torvalds" เป็นผู้เขียน ทั้งที่ไม่ใช่ตัวจริงเลย
```

**Signed commits** แก้ปัญหานี้โดยใช้ **cryptographic signature** (ลายเซ็นดิจิทัลด้วยกุญแจส่วนตัว) ที่พิสูจน์ได้ทางคณิตศาสตร์ว่า commit นั้นมาจากเจ้าของกุญแจจริง ๆ ไม่มีใครปลอมแปลงได้ถ้าไม่มีกุญแจส่วนตัวนั้น

เมื่อ commit ถูก sign และตรวจสอบผ่าน จะเห็นป้าย **"Verified"** สีเขียวบน GitHub กำกับไว้ที่ commit นั้นเสมอ:

```
commit a1b2c3d
Author: Somchai Devteam <somchai@company.com>
        ✅ Verified

    fix: correct rounding error in tax calculation
```

### ระบบลายเซ็นที่ GitHub รองรับ

| ระบบ | คำอธิบายสั้น ๆ |
|---|---|
| **GPG** | ระบบดั้งเดิมที่ใช้กันมานาน ใช้กุญแจ GPG คู่ (public/private key) |
| **SSH** | ใช้ SSH key คู่เดียวกับที่ใช้ authenticate การ push/pull ได้เลย (สะดวกกว่า GPG มาก เพราะไม่ต้องสร้างกุญแจแยก) |
| **S/MIME** | ใช้ X.509 certificate มักใช้ในองค์กรที่มีระบบ PKI ขององค์กรอยู่แล้ว |

### เหตุผลที่กฎนี้ยังไม่ใช่กฎที่เปิดกันเป็นค่าเริ่มต้นในทุกทีม

แม้จะสำคัญมากด้านความปลอดภัย แต่ Require signed commits ยังไม่ใช่กฎที่ทีมทั่วไป (โดยเฉพาะทีมเล็ก) เปิดใช้ตั้งแต่แรก เพราะ:

- ต้องให้สมาชิกทุกคนในทีม **ตั้งค่า GPG หรือ SSH signing ในเครื่องตัวเองก่อน** ถึงจะ commit ได้เลย (มีต้นทุนการตั้งค่าเริ่มต้น)
- ถ้าไม่เตรียมทีมให้พร้อมก่อนเปิดกฎนี้ จะเกิดปัญหา "commit ไม่ได้เลย" ทันทีสำหรับทุกคนที่ยังไม่ได้ตั้งค่า
- เหมาะกับองค์กรที่ให้ความสำคัญกับ **supply chain security** สูงเป็นพิเศษ เช่น โครงการ open source ขนาดใหญ่, บริษัทด้าน security, หน่วยงานที่มีข้อกำหนด compliance เข้มงวด (เช่น SOC 2, ISO 27001)

เนื้อหาการตั้งค่า GPG/SSH signing แบบละเอียดทีละขั้นตอน พร้อมวิธีแก้ปัญหาที่พบบ่อยเมื่อเปิดใช้งานจริงในทีม จะถูกสอนอย่างเจาะลึกใน **Part 79** ซึ่งอยู่ในหมวดความปลอดภัยของหลักสูตรนี้ ตอนนี้ขอให้จำไว้แค่ว่า: **Require signed commits คือกฎที่ยกระดับความน่าเชื่อถือของประวัติ Git จาก "เชื่อโดยสุจริตใจ" ไปสู่ "พิสูจน์ได้ทางคณิตศาสตร์"**

---

## Step 367: "Restrict who can push to matching branches" จำกัดสิทธิ์เฉพาะคน/ทีม

กฎทั้งหมดที่ผ่านมาเป็นเรื่องของ "กระบวนการ" (ต้องผ่าน PR, ต้องมีคนอนุมัติ, ต้องผ่าน CI) แต่กฎนี้เป็นเรื่องของ **"ตัวตน"** โดยตรง — ใครมีสิทธิ์ push เข้า branch นี้ได้บ้าง

### คืออะไร

**"Restrict who can push to matching branches"** (ในหน้า Classic Branch Protection) หรือส่วน **Bypass list / Restrict push** (ใน Rulesets) ทำหน้าที่จำกัดว่า **แม้จะผ่านกฎอื่นทั้งหมดแล้ว (PR, approval, status check) ก็ตาม จะมีเฉพาะบุคคล ทีม หรือ GitHub App ที่ระบุไว้เท่านั้นที่มีสิทธิ์ push หรือ merge เข้า branch นี้ได้**

ค่าเริ่มต้นถ้าไม่ตั้งกฎนี้: **ใครก็ตามที่มีสิทธิ์ Write ขึ้นไปใน repository สามารถ merge PR เข้า branch นั้นได้** เมื่อเงื่อนไขอื่น ๆ ครบถ้วน

### ใช้เมื่อไหร่

กฎนี้เหมาะกับสถานการณ์ที่ต้องการควบคุมเข้มงวดกว่าปกติ เช่น:

- **Release branch** (เช่น `release/v2.0`) — อยากให้เฉพาะ Release Manager หรือทีม DevOps เท่านั้นที่ merge เข้าได้ แม้นักพัฒนาทุกคนจะเปิด PR มาได้ก็ตาม
- **Hotfix branch สำหรับ production** — จำกัดให้เฉพาะ Senior Engineer หรือ On-call Engineer เท่านั้น
- **Branch ที่เชื่อมกับการ deploy อัตโนมัติโดยตรง** — ต้องการชั้นความปลอดภัยพิเศษเพิ่มจากกระบวนการ PR ปกติ
- **Compliance requirement** — องค์กรบางแห่งกำหนดให้ต้องมีการควบคุม "separation of duties" คือคนเขียนโค้ดกับคนที่มีสิทธิ์ปล่อยโค้ดขึ้น production ต้องเป็นคนละกลุ่มกันตามนโยบายภายใน

### วิธีตั้งค่า (Classic)

```
☑ Restrict who can push to matching branches
    Search for people, teams, or apps... 
    → เพิ่ม: @release-managers (team)
    → เพิ่ม: somchai-lead (individual user)
```

### วิธีตั้งค่า (Rulesets) — ยืดหยุ่นกว่าผ่าน "Bypass list"

ใน Rulesets แนวคิดจะกลับด้านเล็กน้อยแต่ยืดหยุ่นกว่ามาก: ค่าเริ่มต้นของ ruleset คือ **บังคับใช้กฎทั้งหมดกับทุกคน** แล้วค่อยกำหนด **Bypass list** แยกต่างหากว่า "ใครบ้างที่ข้ามกฎบางข้อได้ในกรณีพิเศษ" เช่น กำหนดให้ทีม `release-managers` bypass กฎ "require approvals" ได้ในสถานการณ์ฉุกเฉิน แต่ยังต้อง bypass ผ่าน PR อยู่ดี — ทำให้ควบคุมได้ละเอียดกว่าระบบ Classic มาก

### ข้อควรระวังสำคัญ: กฎนี้ไม่ได้แทนที่ Require Pull Request

กฎ "Restrict who can push" มักถูกเข้าใจผิดว่าเป็นการ "อนุญาตให้บางคน push ตรงได้" — แต่ในทางปฏิบัติที่ถูกต้อง **ควรใช้ควบคู่กับ Require a pull request before merging เสมอ** ความหมายที่ถูกต้องคือ:

> "ทุกคนต้องผ่าน Pull Request เหมือนเดิม แต่จะมีแค่คนในรายชื่อนี้เท่านั้นที่มีสิทธิ์กดปุ่ม Merge ให้ PR นั้นสำเร็จได้จริง"

ไม่ใช่การเปิดช่องให้คนกลุ่มนี้ push ตรงข้าม PR ไปเลย (นั่นจะขัดกับเจตนารมณ์ของ Branch Protection ทั้งหมดที่เราตั้งมา)

### ตารางสรุปเปรียบเทียบระดับการควบคุมสิทธิ์

| ระดับ | ใครทำอะไรได้ |
|---|---|
| ไม่มีการจำกัดใด ๆ | ทุกคนที่มีสิทธิ์ Write เปิด PR และ merge ได้เอง (ถ้าเงื่อนไขอื่นครบ) |
| Restrict push (จำกัดผู้ merge) | ทุกคนเปิด PR ได้ แต่มีแค่กลุ่มที่ระบุเท่านั้นที่ merge ให้สำเร็จได้ |
| Restrict + CODEOWNERS | เพิ่มเงื่อนไขว่าไฟล์บางส่วนต้องผ่านการอนุมัติจากเจ้าของไฟล์นั้นโดยเฉพาะด้วย |

---

## Step 368: Merge Strategies เจาะลึก — Merge commit, Squash and merge, Rebase and merge

เมื่อ Pull Request ผ่านทุกเงื่อนไข (approve ครบ, CI ผ่าน, conversation resolve หมด) ถึงเวลากด **Merge** — แต่คำถามที่สำคัญไม่แพ้กันคือ **"จะ merge แบบไหน"** เพราะ GitHub ให้เลือกได้ถึง 3 แบบ ซึ่งแต่ละแบบส่งผลต่อ **หน้าตาของประวัติ Git (commit history)** แตกต่างกันอย่างสิ้นเชิง

สมมติเรามี branch `feature/login` ที่มี 3 commits แยกกัน กำลังจะ merge เข้า `main`:

```
main:     A---B---C
                    \
feature:             D---E---F   (3 commits: "wip", "fix typo", "add tests")
```

มาดูว่าแต่ละ merge strategy จะทำให้ประวัติของ `main` ออกมาหน้าตาเป็นอย่างไร

### 1. Merge Commit (ค่าเริ่มต้นของ GitHub)

**วิธีทำงาน:** สร้าง commit ใหม่ 1 อัน เรียกว่า **merge commit** ที่มี **parent 2 อัน** ชี้ไปทั้งจุดสุดท้ายของ `main` (C) และจุดสุดท้ายของ `feature/login` (F) พร้อมกัน โดย **เก็บ commit ทั้งหมดของ branch feature ไว้ครบทุกอัน**

```
main:     A---B---C-------------M   (M = merge commit, parent คือทั้ง C และ F)
                    \           /
feature:             D---E---F
```

**คำสั่ง Git ที่เทียบเท่า:**

```bash
git checkout main
git merge --no-ff feature/login
```

**ข้อดี:**

- **เก็บบริบทของ feature branch ไว้ครบสมบูรณ์ที่สุด** — เห็นชัดเจนว่า commit ไหนอยู่ในฟีเจอร์เดียวกัน อยู่ในกลุ่มไหน
- **ไม่แก้ไข commit hash เดิมของนักพัฒนาเลยแม้แต่ตัวเดียว** — ปลอดภัยต่อประวัติที่สุด ไม่มีความเสี่ยงเรื่อง rewrite history
- เห็น **จุด merge ที่ชัดเจน** ในกราฟประวัติ (`git log --graph` จะเห็นเป็นกิ่งก้านชัดเจน) มีประโยชน์มากเวลาต้องการ revert ทั้ง feature ออกทีเดียว (`git revert -m 1 <merge-commit>`)
- เหมาะกับทีมที่ต้องการ **audit trail แบบละเอียดทุกขั้นตอน** ว่าใครทำอะไรในลำดับไหนบ้างระหว่างพัฒนาฟีเจอร์

**ข้อเสีย:**

- **ประวัติรกมาก** ถ้าทีมมี PR จำนวนมากและแต่ละคน commit บ่อย ๆ ระหว่างพัฒนา (เช่น commit ชื่อ "wip", "fix typo", "oops", "อีกรอบ") — commit เหล่านี้ที่ไม่มีความหมายจะถูกฝังอยู่ใน `main` ตลอดไป
- `git log` แบบเส้นตรง (linear) จะดูซับซ้อนมาก มีกิ่งก้านเต็มไปหมด ทำให้การไล่ดูประวัติเข้าใจยากขึ้นสำหรับคนที่ไม่คุ้นกับกราฟ
- การทำ `git bisect` (หาว่า commit ไหนทำให้เกิดบั๊ก) อาจซับซ้อนขึ้นเล็กน้อยเพราะต้องไล่ผ่าน merge commit

### 2. Squash and Merge

**วิธีทำงาน:** รวม (squash) **ทุก commit ใน feature branch ให้เหลือเพียง 1 commit เดียว** แล้วนำ commit เดียวนั้นไปต่อท้าย `main` แบบเส้นตรง (ไม่มี merge commit แยก ไม่มี parent 2 ทาง)

```
main:     A---B---C---S   (S = 1 commit เดียวที่รวม D+E+F เข้าด้วยกันทั้งหมด)
```

**คำสั่ง Git ที่เทียบเท่า:**

```bash
git checkout main
git merge --squash feature/login
git commit -m "feat: add login functionality (#42)"
```

**ข้อดี:**

- **ประวัติของ `main` สะอาดที่สุด** — เป็นเส้นตรง 1 commit ต่อ 1 Pull Request เสมอ อ่านง่ายมาก
- ไม่ต้องสนใจว่าระหว่างพัฒนา feature นั้นจะมี commit ที่ดูไม่เรียบร้อยกี่อัน (commit "wip", "fix typo" ทั้งหมดจะถูกซ่อนไว้ในประวัติของ PR เท่านั้น ไม่ปรากฏใน `main`)
- `git log main` จะอ่านออกได้ทันทีว่าแต่ละ commit คือ "ฟีเจอร์หรือการแก้ไขอะไร" เพราะ 1 commit = 1 PR เสมอ
- ทำ `git revert` ง่ายมาก — revert 1 commit เท่ากับ revert ทั้ง feature นั้นทั้งหมดในคำสั่งเดียว
- เหมาะมากกับทีมที่มีวัฒนธรรม commit ระหว่างทางแบบ "commit บ่อย ๆ ไม่ต้องสวยงาม" เพราะสุดท้ายมันจะถูกรวมให้สวยงามตอน merge อยู่ดี

**ข้อเสีย:**

- **สูญเสียประวัติละเอียดของแต่ละ commit ย่อยใน feature branch ไปจาก `main`** — ถ้าอยากรู้ว่าระหว่างพัฒนามีการเปลี่ยนใจหรือแก้ไขขั้นตอนไหนบ้าง ต้องไปดูใน PR history บน GitHub เท่านั้น (ซึ่ง GitHub ยังเก็บ commit ย่อยไว้ในหน้า PR แม้จะ squash ไปแล้วก็ตาม แต่ไม่อยู่ใน `git log` ของ `main` โดยตรง)
- ถ้า feature branch มีขนาดใหญ่มาก (หลายสัปดาห์ หลาย commit ที่มีเจตนาต่างกันจริง ๆ ไม่ใช่แค่ "wip") การ squash รวมเป็นก้อนเดียวอาจทำให้เสีย granularity ที่มีประโยชน์จริง ๆ ไป
- ถ้านักพัฒนายัง pull/fetch จาก `feature/login` เดิมต่อหลัง squash merge ไปแล้ว **จะเกิด commit ซ้ำซ้อนหรือ conflict แปลก ๆ ได้** เพราะ commit hash ของ squash commit ใหม่ไม่ตรงกับ commit เดิมในเครื่องของนักพัฒนาเลย (ควรลบ local branch ทิ้งหลัง merge เสมอ)

### 3. Rebase and Merge

**วิธีทำงาน:** นำแต่ละ commit ใน feature branch (D, E, F) มา **"เล่นซ้ำ" (replay) ต่อท้ายปลายของ `main` ทีละอันตามลำดับเดิม** โดยให้แต่ละ commit ได้ **hash ใหม่** แต่ **เนื้อหาการเปลี่ยนแปลงยังคงแยกเป็น commit ย่อยเหมือนเดิมทุกอัน** ไม่มี merge commit เกิดขึ้นเลย

```
main:     A---B---C---D'---E'---F'   (D', E', F' คือ commit เดิมที่ hash เปลี่ยนไป แต่เนื้อหาเหมือนเดิม)
```

**คำสั่ง Git ที่เทียบเท่า:**

```bash
git checkout feature/login
git rebase main
git checkout main
git merge --ff-only feature/login
```

**ข้อดี:**

- **ประวัติเป็นเส้นตรงสมบูรณ์แบบ (linear history)** โดยที่ **ยังคงเก็บ commit ย่อยแต่ละอันไว้ครบ** ต่างจาก squash ที่รวมเป็นก้อนเดียว
- ไม่มี merge commit ที่ "ไม่มีเนื้อหาจริง" มาปนในประวัติเลย ทำให้ `git log` อ่านง่ายและสวยงามมาก
- เหมาะกับทีมที่ commit อย่างมีวินัย (แต่ละ commit มีความหมายชัดเจน มี commit message ที่ดี) และต้องการเก็บรายละเอียดระดับ commit ไว้ แต่ก็ไม่อยากมี merge commit รก ๆ
- ทำ `git bisect` ได้แม่นยำที่สุด เพราะแต่ละ commit ยังคงทดสอบแยกกันได้ทีละอันบนเส้นประวัติเดียว

**ข้อเสีย:**

- **Commit hash เปลี่ยนใหม่ทั้งหมด** — เหมือนกับการ rebase ทั่วไปที่เราเรียนหลักการไปแล้ว การเปลี่ยน hash นี้หมายความว่า **ถ้านักพัฒนายังมี local branch `feature/login` เดิมอยู่ และพยายาม push/pull ต่อ จะเจอปัญหาประวัติไม่ตรงกันทันที** (ต้องลบ branch เดิมทิ้งแล้ว pull `main` ใหม่)
- ถ้า feature branch มี commit ที่ "ไม่เรียบร้อย" เยอะมาก (เช่น "wip", "fix typo" หลายรอบ) การใช้ rebase and merge จะทำให้ commit เหล่านั้นทั้งหมดเข้าไปอยู่ใน `main` แบบถาวร **ต่างจาก squash ที่ซ่อนมันไว้ได้** ดังนั้นกลยุทธ์นี้เหมาะกับทีมที่มีวินัยเรื่อง commit message และจำนวน commit ต่อ PR ที่ดีอยู่แล้วเท่านั้น
- ต้องเข้าใจ rebase ในระดับที่ลึกพอสมควรก่อนจะใช้กลยุทธ์นี้อย่างปลอดภัย (เราจะเจาะลึกความแตกต่างระหว่าง Rebase กับ Merge อีกครั้งอย่างละเอียดสุด ๆ ใน **Part 38** ที่ต่อจาก Part นี้โดยตรง)
- ถ้ามี merge conflict ระหว่าง rebase อาจต้องแก้ conflict **ซ้ำหลายรอบ** (ทีละ commit ที่ชนกัน) ต่างจาก merge commit ที่แก้ conflict แค่ครั้งเดียวตอน merge

### ตารางเปรียบเทียบสรุปทั้ง 3 แบบ

| คุณสมบัติ | Merge Commit | Squash and Merge | Rebase and Merge |
|---|---|---|---|
| หน้าตาประวัติบน `main` | มีกิ่งก้าน (non-linear) | เส้นตรง (linear) | เส้นตรง (linear) |
| จำนวน commit ที่เพิ่มเข้า `main` ต่อ 1 PR | เท่ากับจำนวน commit เดิม + 1 (merge commit) | 1 commit เสมอ | เท่ากับจำนวน commit เดิมทุกอัน |
| Commit hash เปลี่ยนหรือไม่ | ไม่เปลี่ยนเลย | เปลี่ยน (กลายเป็น commit ใหม่) | เปลี่ยนทุกอัน |
| เก็บรายละเอียด commit ย่อยไว้ไหม | เก็บครบ | ไม่เก็บใน `main` (แต่ยังดูได้ใน PR) | เก็บครบ |
| Revert ทั้ง feature ทำง่ายแค่ไหน | ง่าย (revert merge commit เดียว ด้วย `-m 1`) | ง่ายที่สุด (revert 1 commit) | ต้อง revert ทีละหลาย commit หรือใช้ range |
| ความเสี่ยงกับคนที่ยังใช้ local branch เดิมต่อ | ไม่มีความเสี่ยง | มีความเสี่ยง (ต้องลบ local branch ทิ้ง) | มีความเสี่ยง (ต้องลบ local branch ทิ้ง) |
| เหมาะกับทีมแบบไหน | ต้องการ audit trail ละเอียดทุกขั้นตอน | ต้องการประวัติสะอาด ไม่สนใจ commit ย่อยระหว่างทาง | ต้องการประวัติสะอาด แต่ยังอยากเก็บ commit ย่อยที่มีวินัยดี |
| ตัวอย่างองค์กรที่นิยมใช้ | ทีม enterprise ที่เน้น compliance/audit | ทีม product ที่เน้นความเร็วและความสะอาดของ `main` (นิยมมากในสตาร์ทอัพและทีมสมัยใหม่จำนวนมาก) | โปรเจกต์ open source ที่มีวินัยเรื่อง commit สูง (เช่นสไตล์คล้าย Linux Kernel) |

### ผลกระทบต่อการทำงานร่วมกับ Git คำสั่งอื่น ๆ

**`git blame`:** ด้วย Squash และ Rebase ที่ให้ประวัติเส้นตรง `git blame` จะไล่ดูง่ายกว่ามาก เพราะไม่ต้องสับสนกับ merge commit ที่แทรกอยู่กลางทาง

**`git bisect`:** Rebase and Merge ให้ผลลัพธ์ที่แม่นยำที่สุดเพราะทุก commit อยู่บนเส้นตรงและยังคง build ได้อิสระทีละอัน ส่วน Squash ทำให้ bisect หยาบกว่า (ได้แค่ระดับ "PR ไหน" ไม่ใช่ "commit ไหนในนั้น") ส่วน Merge Commit อาจทำให้ bisect ข้ามเข้าไปในกิ่งที่ไม่เกี่ยวข้องได้ถ้าไม่ระวัง

**ขนาด repository:** ในทางเทคนิคความแตกต่างเรื่องขนาดพื้นที่จัดเก็บระหว่างทั้ง 3 แบบมีน้อยมากในทางปฏิบัติ ไม่ควรเป็นปัจจัยหลักในการตัดสินใจเลือก

---

## Step 369: การตั้งค่า Allowed Merge Methods ระดับ repository

หลังจากเข้าใจทั้ง 3 กลยุทธ์แล้ว คำถามต่อไปคือ **ทีมควรเปิดให้เลือกใช้ได้กี่แบบ** — GitHub อนุญาตให้ตั้งค่าระดับ repository ได้ว่าจะ **เปิดหรือปิด** แต่ละ merge method รวมถึงตั้งค่า **default commit message** ของแต่ละแบบได้ด้วย

### ตำแหน่งเมนู

```
Repository → Settings → General → เลื่อนลงไปหาหัวข้อ "Pull Requests"
```

จะเห็นตัวเลือกดังนี้:

```
Allow merge commits
    ☑ Default to PR title and description ▼

Allow squash merging
    ☑ Default to pull request title and commit details ▼

Allow rebase merging

☐ Always suggest updating pull request branches
☑ Allow auto-merge
☑ Automatically delete head branches
```

### เปิดทุกแบบ vs. เปิดแบบเดียว: ควรเลือกแบบไหน

**แนวทางที่ 1: เปิดทั้ง 3 แบบ (ค่าเริ่มต้นของ GitHub)**

- ให้อิสระกับผู้ merge เลือกวิธีที่เหมาะกับ PR แต่ละอันเอง
- **ข้อเสีย:** ถ้าไม่มีแนวทางที่ชัดเจนในทีม จะเกิดความไม่สม่ำเสมอ — บาง PR ใช้ squash บาง PR ใช้ merge commit ทำให้ประวัติของ `main` ปนกันไปมา อ่านยากในระยะยาว

**แนวทางที่ 2 (แนะนำสำหรับทีมส่วนใหญ่): เปิดแค่แบบเดียว แล้วล็อกให้ทุกคนใช้แบบเดียวกันเสมอ**

ตัวอย่างเช่น ปิด **Allow merge commits** และ **Allow rebase merging** ทิ้ง เหลือแค่ **Allow squash merging** เปิดไว้อันเดียว — ผลลัพธ์คือปุ่ม Merge บนทุก PR จะมีตัวเลือกเดียวเท่านั้น ไม่มีทางเลือกอื่นให้สับสนหรือทำผิดพลาด

```
[ Squash and merge ]  ▼   ← ถ้าปิด 2 แบบที่เหลือ ปุ่มจะกลายเป็นตัวเลือกเดียวแบบนี้เสมอ
```

นี่คือแนวทางที่บริษัทเทคโนโลยีจำนวนมากในปัจจุบันเลือกใช้ (โดยเฉพาะแบบ **Squash and merge เป็นค่าเดียว**) เพราะให้ประวัติของ `main` สะอาดสม่ำเสมอ ไม่ต้องพึ่งวินัยของแต่ละคนในการเลือกวิธี merge เอง

### ตั้งค่า Default commit message ของ Squash merge

เมื่อเปิด **Allow squash merging** จะมีตัวเลือกย่อยให้กำหนดว่า commit message ของ squash commit จะถูกสร้างจากอะไรเป็นค่าเริ่มต้น:

| ตัวเลือก | หน้าตา commit message ที่ได้ |
|---|---|
| **Default to pull request title and commit details** | ใช้ชื่อ PR เป็นบรรทัดแรก ตามด้วยรายการ commit ย่อยทั้งหมดในเนื้อหา (body) |
| **Default to pull request title** | ใช้แค่ชื่อ PR เท่านั้น ไม่มีรายละเอียดเพิ่ม |
| **Default to first commit's message** | ใช้ commit message ของ commit แรกสุดในสาขานั้น |
| **Default to a blank commit message** | ปล่อยว่าง ให้ผู้ merge ต้องพิมพ์เอง |

ทีมที่ใช้แนวทาง **Conventional Commits** (ที่เราจะพูดถึงในหลาย Part ถัดจากนี้) มักเลือก **"Default to pull request title"** ควบคู่กับการบังคับให้ชื่อ PR ต้องเขียนตามรูปแบบ `feat: ...`, `fix: ...`, `chore: ...` เพื่อให้ `main` มีประวัติที่อ่านง่ายและใช้สร้าง Changelog อัตโนมัติได้ในอนาคต

### "Automatically delete head branches" — ตัวช่วยเก็บกวาดที่ควรเปิดไว้เสมอ

ตัวเลือกนี้ (ในหน้าเดียวกัน) ทำให้ **feature branch ที่ถูก merge เสร็จแล้วจะถูกลบออกจาก remote โดยอัตโนมัติทันที** ไม่ต้องมาคอยลบเองทีละอัน ช่วยให้รายการ branch บน GitHub ไม่รกไปด้วย branch ที่ merge เสร็จไปนานแล้ว — แนะนำให้เปิดไว้เสมอสำหรับทุก repository ที่ทำงานเป็นทีม

### "Allow auto-merge"

ตัวเลือกนี้เปิดให้ผู้เขียน PR สามารถกด **"Enable auto-merge"** ไว้ล่วงหน้าได้ — เมื่อเงื่อนไข Branch Protection ทั้งหมดครบถ้วน (approve ครบ, CI ผ่าน, conversation resolve หมด) ระบบจะ **merge ให้อัตโนมัติทันทีโดยไม่ต้องมีใครมากดปุ่ม Merge เอง** มีประโยชน์มากในทีมที่มี CI รันนาน เพราะผู้เขียน PR ไม่ต้องมานั่งรอเช็คสถานะเองตลอดเวลา

---

## Step 370: แบบฝึกหัด — ตั้งค่า Branch Protection เต็มรูปแบบให้ main branch ของโปรเจกต์จำลอง

ถึงเวลาลงมือปฏิบัติจริง ในแบบฝึกหัดนี้ เราจะจำลองการตั้งค่า Branch Protection แบบเต็มรูปแบบให้กับ repository ตัวอย่าง โดยรวมเอา CODEOWNERS จาก Part ก่อนหน้าเข้ามาผนวกด้วย

### เตรียมโปรเจกต์จำลอง

สมมติว่าคุณมี repository ชื่อ `payment-service` บน GitHub ที่มีโครงสร้างประมาณนี้:

```
payment-service/
├── .github/
│   └── CODEOWNERS
├── src/
│   ├── api/
│   ├── payment/
│   └── utils/
├── tests/
└── README.md
```

และมีไฟล์ `.github/CODEOWNERS` (ที่เราเรียนไปแล้วใน Part ก่อนหน้า) ตั้งไว้ดังนี้:

```
# .github/CODEOWNERS
*                    @backend-team
/src/payment/        @finance-team @tech-lead-somchai
/.github/            @tech-lead-somchai
```

### ขั้นตอนที่ 1: สร้าง Ruleset สำหรับ `main`

1. ไปที่ `Settings → Rules → Rulesets → New branch ruleset`
2. ตั้งชื่อ: `Protect main branch`
3. **Enforcement status:** `Active`
4. **Target branches:** เลือก `Include default branch`

### ขั้นตอนที่ 2: ตั้งค่ากฎการ Pull Request

เปิดใช้งานกฎต่อไปนี้ทั้งหมดภายใต้หมวด branch rules:

```
☑ Require a pull request before merging
    Required approvals: 2
    ☑ Dismiss stale pull request approvals when new commits are pushed
    ☑ Require review from Code Owners
    ☑ Require approval of the most recent reviewable push
    ☑ Require conversation resolution before merging
```

**เหตุผลของแต่ละค่า:** เนื่องจากนี่คือ `payment-service` ซึ่งกระทบเงินจริงของบริษัท จึงตั้ง required approvals ไว้ที่ 2 คน (สูงกว่าโปรเจกต์ทั่วไป) และเปิด Require review from Code Owners เพื่อให้แน่ใจว่าไฟล์ใน `/src/payment/` ต้องผ่าน `@finance-team` เสมอ ไม่ว่าใครจะเป็นคนแก้ก็ตาม

### ขั้นตอนที่ 3: เชื่อมกับ CI

```
☑ Require status checks to pass before merging
    ☑ Require branches to be up to date before merging
    Required checks:
      ✓ build
      ✓ test / unit-tests
      ✓ lint / eslint
      ✓ security-scan
```

(ในทางปฏิบัติ check เหล่านี้จะปรากฏให้เลือกได้ก็ต่อเมื่อมี GitHub Actions workflow รันผ่าน repository นี้มาแล้วอย่างน้อย 1 ครั้ง — เนื้อหาการสร้าง workflow เหล่านี้จะอยู่ใน Part 66–70)

### ขั้นตอนที่ 4: ตั้งค่าความปลอดภัยของประวัติ

```
☑ Require signed commits
```

(สำหรับ `payment-service` ซึ่งเป็นระบบละเอียดอ่อนด้านการเงิน การบังคับ signed commits ถือเป็นแนวทางที่เหมาะสม แม้จะต้องให้ทีมเตรียมตั้งค่า SSH/GPG signing ก่อนก็ตาม — รายละเอียดการตั้งค่าอยู่ใน Part 79)

### ขั้นตอนที่ 5: จำกัดสิทธิ์ผู้ merge

ในส่วน **Restrict updates** หรือ **Bypass list** กำหนดว่า:

```
เฉพาะทีมต่อไปนี้เท่านั้นที่ merge เข้า main ได้:
  - @backend-team (merge ได้ตามปกติ เมื่อผ่านทุกเงื่อนไข)

Bypass list (ข้ามกฎได้เฉพาะกรณีฉุกเฉิน):
  - @tech-lead-somchai (bypass ได้เฉพาะกรณี hotfix วิกฤต ยังต้องผ่าน PR เสมอ)
```

### ขั้นตอนที่ 6: ตั้งค่า Merge Strategy ระดับ repository

ไปที่ `Settings → General → Pull Requests` แล้วตั้งค่า:

```
☐ Allow merge commits        (ปิด)
☑ Allow squash merging       (เปิดอันเดียว)
    Default: "Default to pull request title"
☐ Allow rebase merging       (ปิด)

☑ Always suggest updating pull request branches
☑ Allow auto-merge
☑ Automatically delete head branches
```

**เหตุผล:** ทีมนี้เลือกใช้ Squash and Merge เป็นมาตรฐานเดียว เพื่อให้ประวัติของ `main` สะอาด อ่านง่ายเป็นเส้นตรง 1 commit ต่อ 1 PR เสมอ เหมาะกับการ trace ย้อนกลับไปดูว่าฟีเจอร์ไหนถูกเพิ่มเมื่อไหร่ได้ง่าย

### ทดสอบผลลัพธ์: จำลองสถานการณ์จริง

ลองจำลองว่านักพัฒนาชื่อ "มานะ" ในทีม `@backend-team` ต้องการแก้ไขโค้ดใน `/src/payment/calculator.js`:

```bash
git checkout -b fix/rounding-error
# แก้ไขโค้ด...
git add src/payment/calculator.js
git commit -m "fix rounding error in tax calculation"
git push origin fix/rounding-error
```

เมื่อมานะ push แล้วลอง `git push origin main` ตรง ๆ โดยไม่ตั้งใจ จะได้ผลลัพธ์:

```
remote: error: GH006: Protected branch update failed for refs/heads/main.
```

มานะจึงต้องเปิด Pull Request แทน ระบบจะตรวจสอบและแสดงผลดังนี้บนหน้า PR:

```
This branch has not been merged yet.

Review required
  ☐ 0 of 2 required approvals — 
    ⚠️ Changes affect src/payment/ which requires review from @finance-team

Checks
  ⏳ build — running...
  ⏳ test — running...

Conversations
  (ยังไม่มี)

[ Merge pull request ]  ← ยังกดไม่ได้จนกว่าทุกเงื่อนไขจะครบ
```

เนื่องจากไฟล์ที่แก้อยู่ใน `/src/payment/` ระบบจะบังคับให้ต้องมีสมาชิกจาก `@finance-team` มา approve เพิ่มเติมโดยเฉพาะ (จาก Require review from Code Owners) นอกเหนือจากจำนวน approval ทั่วไปที่ต้องครบ 2 คน — และปุ่ม Merge จะกลายเป็นแค่ตัวเลือก **Squash and merge** เท่านั้น เพราะทีมปิดอีก 2 แบบไปแล้ว

### Checklist สำหรับแบบฝึกหัดนี้

ให้ตรวจสอบว่าคุณตั้งค่าครบทุกข้อต่อไปนี้จริง ก่อนถือว่าแบบฝึกหัดนี้เสร็จสมบูรณ์:

- [ ] สร้าง Ruleset หรือ Branch Protection Rule ให้ `main` เรียบร้อยแล้ว
- [ ] เปิด Require a pull request before merging พร้อมตั้ง required approvals อย่างน้อย 2 คน
- [ ] เปิด Dismiss stale pull request approvals when new commits are pushed
- [ ] เปิด Require review from Code Owners และมีไฟล์ CODEOWNERS ที่ถูกต้องรองรับ
- [ ] เปิด Require status checks to pass before merging พร้อมเลือก check ที่จำเป็นครบ
- [ ] เปิด Require conversation resolution before merging
- [ ] ทดลองเปิด/ปิด Require signed commits และเข้าใจผลกระทบที่ตามมา
- [ ] ตั้งค่า Restrict push ให้เฉพาะทีมที่กำหนดเท่านั้นที่ merge ได้
- [ ] เลือก Allowed merge methods เหลือแบบเดียว และเข้าใจเหตุผลของการเลือกแบบนั้น
- [ ] เปิด Automatically delete head branches
- [ ] ทดสอบ push ตรงเข้า `main` แล้วยืนยันว่าถูกปฏิเสธจริง

---

## สรุป Part 37

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Branch Protection Rules** (และระบบใหม่กว่าอย่าง **Rulesets**) คือกลไกที่เปลี่ยนกระบวนการทำงานที่ทีมตกลงกันไว้ ให้กลายเป็นกฎที่ระบบบังคับใช้จริง ไม่ใช่แค่ข้อตกลงปากเปล่า
2. **Require a pull request before merging** คือกฎพื้นฐานที่สุดที่ปิดช่องทางการ push ตรงเข้า branch หลัก บังคับให้ทุกการเปลี่ยนแปลงต้องผ่าน Pull Request เสมอ
3. **Require approvals** ควบคู่กับ **Dismiss stale reviews** และ **Require review from Code Owners** ทำให้มั่นใจได้ว่าโค้ดผ่านสายตาคนอื่นจริง ๆ และการอนุมัติยังคงตรงกับโค้ดล่าสุดเสมอ
4. **Require status checks to pass before merging** คือจุดเชื่อมสำคัญระหว่าง Branch Protection กับระบบ CI/CD ทำให้โค้ดที่ทดสอบไม่ผ่านไม่มีทาง merge เข้าไปได้ (จะเจาะลึกการสร้าง CI จริงใน Part 66–70)
5. **Require conversation resolution** ป้องกันไม่ให้คำถามหรือข้อกังวลของผู้รีวิวถูกละเลยไปเฉย ๆ
6. **Require signed commits** ยกระดับความน่าเชื่อถือของประวัติ Git จาก "เชื่อโดยสุจริตใจ" ไปสู่ "พิสูจน์ได้ทางคณิตศาสตร์" (จะเจาะลึกการตั้งค่าจริงใน Part 79)
7. **Restrict who can push** ควบคุมว่าใครมีสิทธิ์ทำให้ PR merge สำเร็จได้จริง แม้จะผ่านเงื่อนไขอื่นครบแล้วก็ตาม
8. **Merge Strategies ทั้ง 3 แบบ** — Merge commit (เก็บประวัติละเอียดที่สุด แต่ประวัติรก), Squash and merge (ประวัติสะอาดที่สุด แต่เสีย granularity), Rebase and merge (เส้นตรงและเก็บ commit ย่อยครบ แต่ต้องมีวินัยเรื่อง commit สูง) — แต่ละแบบเหมาะกับวัฒนธรรมทีมที่ต่างกัน
9. การล็อก **Allowed merge methods** ให้เหลือแบบเดียวในระดับ repository ช่วยให้ประวัติของ `main` สม่ำเสมอ ไม่ต้องพึ่งวินัยของแต่ละคนในการเลือกวิธี merge
10. การตั้งค่า Branch Protection แบบเต็มรูปแบบต้องผสมผสานทุกกฎเข้าด้วยกัน พร้อมเชื่อมกับ CODEOWNERS เพื่อให้ได้ระบบป้องกันที่ครอบคลุมทั้งกระบวนการ (process) คน (people) และคุณภาพ (quality) ไปพร้อมกัน

Branch Protection Rules คือรากฐานสำคัญของการทำงานเป็นทีมอย่างปลอดภัยบน GitHub แต่ยังมีคำถามที่ค้างอยู่จาก Step 368 ที่เราต้องเจาะลึกต่อ: **เมื่อไหร่ควรใช้ Merge และเมื่อไหร่ควรใช้ Rebase ในระดับ workflow ประจำวันของนักพัฒนาเอง** (ไม่ใช่แค่ตอนกดปุ่มบน GitHub) — นี่คือสิ่งที่เราจะไปเจาะลึกกันต่อใน Part ถัดไป

**ต่อไป:** [Part 38: Rebase vs Merge: เลือกใช้ให้ถูกต้อง](./part-038-rebase-vs-merge.md)
