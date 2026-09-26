# Part 94: การ Debug และแก้ปัญหา Git ที่ซับซ้อนในองค์กร

> **Step ในหลักสูตรนี้:** Step 931–940
> **เฟส:** 9 — ทักษะมืออาชีพ: Maintainer, Release Management, Metrics
> **เป้าหมายของ Part นี้:** ฝึกกรอบความคิดในการ debug ปัญหา Git อย่างเป็นระบบ และเจาะลึกวิธีแก้ปัญหาจริงที่พบบ่อยที่สุดในองค์กร ตั้งแต่ detached HEAD, merge conflict ที่ซับซ้อน, การกู้คืนงานด้วย `git reflog`, repository เสียหาย, push ไม่ผ่านในหลายรูปแบบ, ปัญหา line ending ข้ามทีม, ไปจนถึง submodule ที่ทำงานผิดพลาด — ทุกคำสั่งใน Part นี้ต้องแม่นยำ เพราะเป็นสถานการณ์ "กู้ภัย" ที่ผิดพลาดซ้ำไม่ได้

---

## สารบัญของ Part นี้

- Step 931: กรอบความคิดในการ debug ปัญหา Git อย่างเป็นระบบ
- Step 932: ปัญหา "Detached HEAD" และวิธีกู้คืนงานกลับมาเป็น branch
- Step 933: Merge conflict ที่ซับซ้อนเกินแก้ — วิธี abort อย่างปลอดภัยแล้วเริ่มใหม่แบบเป็นระบบ
- Step 934: `git reflog` เจาะลึก — ตาข่ายนิรภัยที่สำคัญที่สุดของ Git
- Step 935: Repository เสียหาย (corruption) และการใช้ `git fsck` วินิจฉัย/ซ่อม
- Step 936: ทำไม push ไม่ได้ — permission, protected branch, file size limit
- Step 937: ปัญหา line ending ระหว่างทีมที่ใช้ OS ต่างกัน
- Step 938: ปัญหา submodule ทำงานผิดพลาด — วิธี debug แบบเป็นระบบ
- Step 939: เครื่องมือช่วย debug ขั้นสูง (`git log` เจาะลึก, GUI tools)
- Step 940: แบบฝึกหัด — แก้ปัญหา Git 5 สถานการณ์จำลอง

---

## Step 931: กรอบความคิดในการ debug ปัญหา Git อย่างเป็นระบบ

ก่อนจะลงมือแก้ปัญหา Git แต่ละแบบ สิ่งสำคัญที่สุดคือ **กรอบความคิด (mindset)** ที่ถูกต้อง เพราะปัญหา Git ส่วนใหญ่ที่ "ดูเหมือนร้ายแรง" จริง ๆ แล้วแก้ได้อย่างปลอดภัย 100% ถ้าคุณไม่ตื่นตระหนกและไม่รีบใช้คำสั่งทำลายข้อมูลก่อนเข้าใจสถานการณ์

### หลักการข้อที่ 1: Git แทบไม่เคยทำให้ข้อมูล "หายจริง"

Git ถูกออกแบบมาให้เป็นระบบที่ **append-only** เป็นหลัก คือข้อมูลใหม่จะถูกเพิ่มเข้าไปเรื่อย ๆ ไม่ใช่เขียนทับของเก่าโดยตรง แม้แต่ตอนที่คุณคิดว่า commit "หายไป" (เช่นหลัง `git reset --hard` หรือ `git rebase`) ตัว object ของ commit นั้นก็ยังอยู่ใน `.git/objects` จนกว่า Garbage Collector (`git gc`) จะมาเก็บกวาดจริง ๆ ซึ่งปกติจะรอ 30–90 วัน

> **สรุปเป็นประโยคเดียว:** ก่อนจะตื่นตระหนกว่า "งานหายแล้ว" ให้หยุดคิดก่อนว่า Git แทบไม่เคยลบอะไรทิ้งทันที — มันแค่ "ทำให้มองไม่เห็น" เท่านั้น

### หลักการข้อที่ 2: อ่าน Error Message ให้ละเอียดทุกบรรทัด

Git เป็นหนึ่งในเครื่องมือที่ error message มีคุณภาพดีมากเมื่อเทียบกับเครื่องมืออื่น ๆ ข้อความ error ของ Git มักจะบอกสิ่งเหล่านี้ครบในตัวเอง:

1. **เกิดอะไรขึ้น** (what happened)
2. **ทำไมถึงเกิด** (why it happened)
3. **จะแก้อย่างไร** (บ่อยครั้งบอกคำสั่งที่ต้องรันตรง ๆ)

ตัวอย่างเช่น:

```
$ git push origin main
To github.com:company/project.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'github.com:company/project.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

สังเกตว่า Git บอกครบทั้ง 3 อย่าง: ถูก reject เพราะอะไร (remote มีงานที่เราไม่มี) และแนะนำวิธีแก้ (`git pull`) มาให้เลยในส่วน `hint:`

**บทเรียนสำคัญ:** มือใหม่จำนวนมากเห็น error แล้วรีบค้นหาคำตอบจาก Google ทันทีโดยไม่อ่าน error ให้จบก่อน ทำให้พลาดคำใบ้ที่ Git ให้มาฟรี ๆ อยู่ตรงหน้าแล้ว

### หลักการข้อที่ 3: พยายาม Reproduce ปัญหาก่อนแก้เสมอ

ก่อนรันคำสั่งใด ๆ เพื่อ "แก้" ปัญหา ให้ทำตามลำดับนี้เสมอ:

1. **หยุด (STOP)** — อย่ารีบรันคำสั่งเพิ่มเติมทันทีที่เจอ error โดยเฉพาะคำสั่งที่ทำลายข้อมูล เช่น `git reset --hard`, `git clean -fd`, `git push --force`
2. **สังเกต (OBSERVE)** — รัน `git status` และ `git log --oneline -10` เพื่อดูสถานะปัจจุบันให้ครบก่อน
3. **ทำซ้ำ (REPRODUCE)** — ถ้าเป็นไปได้ ลองทำสิ่งที่ทำให้เกิดปัญหาอีกครั้งในสภาพแวดล้อมที่ปลอดภัย (เช่น clone repo มาไว้ที่โฟลเดอร์ทดลองแยกต่างหาก) เพื่อยืนยันว่าปัญหาเกิดจากอะไรจริง ๆ ไม่ใช่แค่เดา
4. **แยกตัวแปร (ISOLATE)** — ถ้าปัญหาซับซ้อน ให้ตัดตัวแปรออกทีละอย่าง เช่น ถ้าสงสัยว่า `.gitattributes` มีปัญหา ให้ลองปิดการใช้งานชั่วคราวแล้วดูว่าอาการหายไปไหม
5. **แก้ไข (FIX)** — เมื่อเข้าใจสาเหตุแท้จริงแล้วเท่านั้นจึงลงมือแก้
6. **ยืนยันผล (VERIFY)** — หลังแก้แล้ว ตรวจสอบซ้ำด้วย `git status`, `git log`, `git fsck` ว่าสถานะกลับมาถูกต้องจริง ก่อนจะ push หรือบอกทีมว่าแก้เสร็จแล้ว

### หลักการข้อที่ 4: สำรองข้อมูลก่อนลงมือกับคำสั่งเสี่ยงเสมอ

ก่อนรันคำสั่งที่อาจทำลายข้อมูล (`reset --hard`, `clean`, `rebase`, `filter-repo`, `push --force`) ให้สร้างจุดสำรองก่อนเสมอ วิธีที่เร็วและปลอดภัยที่สุดคือสร้าง branch สำรอง หรือ backup ทั้งโฟลเดอร์:

```bash
# วิธีที่ 1: สร้าง branch สำรองไว้ก่อนทำอะไรเสี่ยง ๆ
git branch backup-before-fix-$(date +%Y%m%d-%H%M%S)

# วิธีที่ 2: สำรองทั้งโฟลเดอร์ .git แบบตรงไปตรงมาที่สุด (ปลอดภัยที่สุด)
cp -r .git ../my-project-git-backup
```

การมี backup ก่อนเสมอทำให้คุณกล้าทดลองแก้ปัญหาได้อย่างมั่นใจ เพราะถ้าพลาดก็แค่กลับไปจุดเดิมได้ทันที

### ตารางสรุปกรอบความคิด 6 ขั้นตอน

| ขั้นตอน | คำถามที่ต้องถามตัวเอง |
|---|---|
| 1. STOP | ฉันกำลังจะรันคำสั่งที่ทำลายข้อมูลหรือเปล่า |
| 2. OBSERVE | `git status` และ `git log` บอกอะไรฉันบ้าง |
| 3. REPRODUCE | ฉันทำให้ปัญหาเกิดซ้ำได้ในที่ปลอดภัยไหม |
| 4. ISOLATE | ตัวแปรไหนที่เป็นสาเหตุจริง ๆ |
| 5. FIX | มีวิธีแก้ที่ปลอดภัยที่สุด (ไม่ทำลายข้อมูล) ไหม |
| 6. VERIFY | หลังแก้แล้ว สถานะถูกต้องจริงหรือยัง |

ต่อไปนี้เราจะเจาะลึกปัญหา Git ที่พบบ่อยที่สุดในองค์กรทีละแบบ พร้อมคำสั่งที่แม่นยำที่สุดสำหรับแต่ละสถานการณ์

---

## Step 932: ปัญหา "Detached HEAD" และวิธีกู้คืนงานกลับมาเป็น branch

**Detached HEAD** คือปัญหาที่มือใหม่เจอบ่อยที่สุดและตกใจที่สุดในบรรดาปัญหา Git ทั้งหมด

### 932.1 Detached HEAD คืออะไร

ปกติแล้ว `HEAD` ของ Git จะชี้ไปที่ **ชื่อ branch** (เช่น `refs/heads/main`) ซึ่ง branch นั้นจะชี้ไปที่ commit ล่าสุดอีกที เวลาคุณ commit ใหม่ branch จะขยับตามไปด้วยอัตโนมัติ

แต่ **Detached HEAD** คือสถานะที่ `HEAD` ชี้ไปที่ **commit hash โดยตรง** แทนที่จะชี้ผ่านชื่อ branch สถานะนี้เกิดขึ้นเมื่อคุณสั่ง:

```bash
git checkout a1b2c3d          # checkout ไปที่ commit hash ตรง ๆ
git checkout v1.2.0           # checkout ไปที่ tag
git checkout HEAD~3           # checkout ถอยหลัง 3 commit
git checkout origin/feature-x # checkout ไปที่ remote-tracking branch ตรง ๆ
```

เมื่อคุณรันคำสั่งเหล่านี้ Git จะเตือนทันที:

```
Note: switching to 'a1b2c3d'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

HEAD is now at a1b2c3d เพิ่มฟีเจอร์ล็อกอิน
```

ข้อความนี้ Git บอกครบมากแล้ว: คุณอยู่ใน detached HEAD, commit ที่คุณทำในสถานะนี้ **จะไม่ผูกกับ branch ใด ๆ** และถ้าสลับไป branch อื่นโดยไม่บันทึกไว้ก่อน commit เหล่านั้นจะเข้าถึงยากมาก (แต่ไม่ได้หายไปทันที)

### 932.2 สถานการณ์จริงที่พบบ่อย

1. Checkout ไป commit เก่าเพื่อดูโค้ด แล้ว "ลืมตัว" แก้ไขและ commit ต่อไปเรื่อย ๆ ในสถานะ detached
2. Checkout ไป tag เพื่อ build เวอร์ชันเก่า แล้วมีคน hotfix ตรงนั้นโดยไม่รู้ว่าอยู่ใน detached HEAD
3. CI/CD pipeline บางตัว checkout แบบ detached HEAD โดยธรรมชาติ (เพราะ checkout ที่ commit SHA ตรง ๆ) แล้วมีคนเข้าไปแก้ไขต่อในเครื่องนั้นโดยไม่รู้ตัว

### 932.3 วิธีกู้คืน กรณีที่ 1: ยังอยู่ใน detached HEAD (ยังไม่ได้สลับไปไหน)

ถ้าคุณสังเกตทันว่ายังอยู่ใน detached HEAD และมี commit ที่อยากเก็บไว้ วิธีที่ง่ายและปลอดภัยที่สุดคือสร้าง branch ใหม่ ณ ตำแหน่งปัจจุบันทันที:

```bash
# ตรวจสอบสถานะก่อนเสมอ
git status
# HEAD detached at a1b2c3d
# nothing to commit, working tree clean

git log --oneline -5
# f9e8d7c (HEAD) แก้บั๊กเร่งด่วนใน detached HEAD
# a1b2c3d เพิ่มฟีเจอร์ล็อกอิน
# ...

# สร้าง branch ใหม่จากตำแหน่งปัจจุบัน แล้วสลับไปทันที (คำสั่งเดียวจบ)
git switch -c fix/hotfix-from-detached

# หรือใช้คำสั่งรุ่นเก่าที่ทำงานเหมือนกัน
git checkout -b fix/hotfix-from-detached
```

หลังจากนี้ commit ทั้งหมดที่ทำไว้ในสถานะ detached HEAD จะกลายเป็นส่วนหนึ่งของ branch `fix/hotfix-from-detached` อย่างถาวร ปลอดภัย 100%

### 932.4 วิธีกู้คืน กรณีที่ 2: สลับออกจาก detached HEAD ไปแล้ว โดยไม่ได้สร้าง branch ไว้

นี่คือกรณีที่คนส่วนใหญ่ตกใจที่สุด เพราะเมื่อสลับไป branch อื่น (`git checkout main`) commit ที่ทำไว้ตอน detached จะ**ไม่ปรากฏใน `git log` ของ branch ใด ๆ อีก** ทำให้ดูเหมือนหายไปแล้ว

**ขั้นตอนกู้คืน:**

```bash
# ขั้นที่ 1: ใช้ reflog เพื่อหา commit hash ที่หายไป
git reflog

# ตัวอย่างผลลัพธ์:
# a3f5c21 (HEAD -> main) HEAD@{0}: checkout: moving from f9e8d7c to main
# f9e8d7c HEAD@{1}: commit: แก้บั๊กเร่งด่วนใน detached HEAD
# a1b2c3d HEAD@{2}: checkout: moving from main to a1b2c3d

# ขั้นที่ 2: จะเห็นว่า commit f9e8d7c คือ commit ที่หายไปตอน detached HEAD
# ตรวจสอบให้ชัดเจนก่อนว่าใช่ commit ที่ต้องการจริง
git show f9e8d7c

# ขั้นที่ 3: สร้าง branch ใหม่ชี้ไปที่ commit นั้นเพื่อกู้คืนกลับมา
git branch recovered-hotfix f9e8d7c

# ขั้นที่ 4: ตรวจสอบว่ากู้คืนสำเร็จ
git switch recovered-hotfix
git log --oneline -3
```

**ข้อควรจำสำคัญ:** `git reflog` เก็บบันทึกการเคลื่อนที่ของ `HEAD` ไว้เฉพาะในเครื่อง local เท่านั้น (ไม่ถูก push ไปที่ remote และไม่ถูกแชร์กับใคร) และมีอายุ default 90 วันสำหรับ entry ที่ยัง reachable และ 30 วันสำหรับที่ unreachable ก่อนที่ `git gc` จะเก็บกวาดทิ้งจริง ดังนั้นยิ่งรีบกู้คืนเร็วเท่าไหร่ยิ่งปลอดภัยเท่านั้น

### 932.5 วิธีป้องกันไม่ให้เกิดปัญหานี้ซ้ำ

1. **สังเกต prompt เสมอ** — ตั้งค่า terminal prompt ให้แสดงชื่อ branch ปัจจุบัน (เช่นผ่าน `oh-my-zsh`, `starship`) จะเห็นทันทีว่าอยู่ใน `(HEAD detached at a1b2c3d)` แทนที่จะเป็นชื่อ branch ปกติ
2. **ก่อน checkout commit เก่า ให้ตั้งชื่อ branch ไว้ล่วงหน้าเลย** ถ้ารู้ว่าจะแก้ไขต่อ:
   ```bash
   git checkout -b explore-old-version a1b2c3d
   ```
3. **รัน `git status` เป็นนิสัยก่อน commit ทุกครั้ง** — Git จะเตือนว่า `HEAD detached at ...` อยู่แล้วในบรรทัดแรกของ `git status` เสมอ

---

## Step 933: Merge conflict ที่ซับซ้อนเกินแก้ — วิธี abort อย่างปลอดภัยแล้วเริ่มใหม่แบบเป็นระบบ

หลายคนเมื่อเจอ merge conflict ที่มีไฟล์ conflict เยอะมาก (10, 20, หรือ 50 ไฟล์พร้อมกัน) มักจะพยายาม "ฝืนแก้ทีละไฟล์" จนสับสนและพลาดจนโค้ดพัง ทางออกที่ถูกต้องคือ **รู้จัก abort อย่างปลอดภัย** แล้ววางแผนใหม่

### 933.1 ตรวจสอบว่ากำลังอยู่กลาง merge หรือไม่

```bash
git status
```

ถ้ากำลังอยู่กลาง merge ที่มี conflict คุณจะเห็นข้อความประมาณนี้:

```
On branch feature/payment
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   src/payment/checkout.js
	both modified:   src/payment/gateway.js
	both modified:   src/payment/validator.js
	added by us:     src/payment/new-fee.js
	added by them:   src/payment/legacy-fee.js
```

สังเกตว่า Git บอกวิธี abort ไว้ในข้อความเลย: `(use "git merge --abort" to abort the merge)`

อีกวิธีตรวจสอบคือดูว่าไฟล์ `.git/MERGE_HEAD` มีอยู่หรือไม่ (ถ้ามีแปลว่ากำลัง merge ค้างอยู่):

```bash
ls -la .git/MERGE_HEAD
```

### 933.2 การ abort merge อย่างปลอดภัย

```bash
git merge --abort
```

คำสั่งนี้จะ:
- คืน working directory และ staging area กลับไปเหมือนก่อนเริ่ม merge **ทุกประการ**
- ลบไฟล์ `.git/MERGE_HEAD`, `.git/MERGE_MSG` ทิ้ง
- **ไม่มีการสูญเสียข้อมูลใด ๆ** เพราะยังไม่มีการ commit merge เกิดขึ้นจริง

> **ข้อควรระวัง:** `git merge --abort` ใช้ได้เฉพาะตอนที่ยังไม่ commit merge เท่านั้น ถ้าคุณ resolve conflict แล้ว `git add` และ `git commit` ไปแล้ว จะต้องใช้ `git reset --hard ORIG_HEAD` แทน (ORIG_HEAD จะถูกตั้งค่าอัตโนมัติก่อน merge ทุกครั้ง)

ถ้ากำลังทำ **rebase** ที่ conflict (ไม่ใช่ merge) ให้ใช้:

```bash
git rebase --abort
```

ถ้ากำลังทำ **cherry-pick** ที่ conflict:

```bash
git cherry-pick --abort
```

ถ้ากำลังทำ **revert** ที่ conflict:

```bash
git revert --abort
```

### 933.3 วิธีเริ่มใหม่แบบเป็นระบบหลัง abort

หลัง abort แล้ว อย่ารีบ merge ซ้ำแบบเดิมทันที ให้ทำตามลำดับนี้:

**ขั้นที่ 1: สำรวจขอบเขตของ conflict ก่อนโดยไม่ merge จริง**

```bash
# ดูว่าทั้งสอง branch แตกต่างกันตรงไหนบ้าง ก่อนจะลอง merge จริง
git diff main...feature/payment --stat

# ผลลัพธ์ตัวอย่าง:
#  src/payment/checkout.js  | 45 +++++++++++++--------
#  src/payment/gateway.js   | 30 ++++++++------
#  src/payment/validator.js | 12 +++---
#  3 files changed, 51 insertions(+), 36 deletions(-)
```

**ขั้นที่ 2: หา common ancestor เพื่อเข้าใจว่าทั้งสองฝั่งแก้อะไรไปจากจุดเดียวกัน**

```bash
git merge-base main feature/payment
# a1b2c3d4e5f6...

git log --oneline a1b2c3d4e5f6..main
git log --oneline a1b2c3d4e5f6..feature/payment
```

การดู commit ทั้งสองฝั่งตั้งแต่ common ancestor ช่วยให้เข้าใจว่า "ใครแก้อะไรก่อน" และ "conflict น่าจะเกิดจากอะไร" ก่อนจะลงมือจริง

**ขั้นที่ 3: แบ่งไฟล์ conflict ออกเป็นกลุ่มตามความยาก**

แทนที่จะแก้ทุกไฟล์พร้อมกัน ให้แบ่งเป็น:
- ไฟล์ที่ conflict ง่าย (ต่างคนแก้คนละส่วนของไฟล์ ไม่ overlap กันจริง) → แก้ก่อนเพื่อสร้างความมั่นใจ
- ไฟล์ที่ conflict ซับซ้อน (แก้ logic เดียวกันจริง ๆ) → เก็บไว้แก้ทีหลัง อาจต้องคุยกับเจ้าของโค้ดอีกฝั่ง

**ขั้นที่ 4: merge ใหม่ แล้วแก้ทีละไฟล์อย่างมีระบบ**

```bash
git merge feature/payment

# ดูรายชื่อไฟล์ที่ conflict ทั้งหมด
git status --short | grep "^UU\|^AA\|^AU\|^UA"

# แก้ทีละไฟล์ ใช้ git diff เพื่อดู conflict marker ในไฟล์นั้นก่อน
git diff src/payment/checkout.js
```

เมื่อเปิดไฟล์ conflict จะเห็น marker แบบนี้:

```
<<<<<<< HEAD
const feeRate = 0.03; // ค่าธรรมเนียมใหม่
=======
const feeRate = calculateLegacyFee(amount); // ค่าธรรมเนียมแบบเก่า
>>>>>>> feature/payment
```

**เทคนิคสำหรับกรณีที่รู้แน่ชัดว่าจะเลือกฝั่งไหนทั้งไฟล์:**

```bash
# เลือกเวอร์ชันของฝั่งเรา (HEAD) ทั้งไฟล์
git checkout --ours src/payment/legacy-config.js
git add src/payment/legacy-config.js

# เลือกเวอร์ชันของฝั่งที่ merge เข้ามาทั้งไฟล์
git checkout --theirs src/payment/new-config.js
git add src/payment/new-config.js
```

> **ข้อควรระวัง:** `--ours`/`--theirs` ในบริบทของ `git merge` หมายถึง "ฝั่งที่เรากำลังยืนอยู่ (HEAD)" กับ "ฝั่งที่ merge เข้ามา" แต่ถ้าเป็นบริบทของ `git rebase` ความหมายจะ**สลับกัน** เพราะระหว่าง rebase ฝั่ง "ours" คือ branch ปลายทางที่กำลัง rebase ไปหา ต้องระวังจุดนี้ให้มากเป็นพิเศษ

**ขั้นที่ 5: ยืนยันว่า resolve ครบทุกไฟล์ก่อน commit**

```bash
git status
# ต้องไม่มี "Unmerged paths" เหลืออยู่แล้ว

# ตรวจสอบว่าไม่มี conflict marker หลงเหลืออยู่ในโค้ด (สำคัญมาก มือใหม่พลาดบ่อย)
git diff --check

# grep หา marker ที่อาจตกหล่น
grep -rn "<<<<<<<\|=======\|>>>>>>>" src/ --include="*.js"
```

**ขั้นที่ 6: commit และรัน test ก่อน push เสมอ**

```bash
git commit
npm test   # หรือคำสั่งรัน test suite ของโปรเจกต์
```

### 933.4 เทคนิคเสริม: ใช้ `git rerere` เพื่อไม่ต้องแก้ conflict ซ้ำ

ถ้าต้อง merge/rebase branch เดิมซ้ำหลายรอบ (เช่นระหว่าง rebase แบบยาว) เปิดใช้ `rerere` (reuse recorded resolution) เพื่อให้ Git จำวิธีแก้ conflict ที่เคยทำไว้แล้ว:

```bash
git config --global rerere.enabled true
```

หลังจากนั้นถ้า conflict แบบเดิมเกิดซ้ำ Git จะเสนอวิธีแก้ที่คุณเคยทำไว้ให้อัตโนมัติ ลดเวลาการแก้ conflict ซ้ำ ๆ ได้มาก

---

## Step 934: `git reflog` เจาะลึก — ตาข่ายนิรภัยที่สำคัญที่สุดของ Git

ถ้าต้องเลือกคำสั่งเดียวที่ "ช่วยชีวิต" นักพัฒนาได้มากที่สุดเวลาทำพลาดกับ Git คำสั่งนั้นคือ `git reflog`

### 934.1 `git reflog` คืออะไรกันแน่

`git reflog` (reference log) คือบันทึกการเคลื่อนที่ของ **reference** ต่าง ๆ ใน local repository ของคุณ (โดยเฉพาะ `HEAD` และแต่ละ branch) ทุกครั้งที่ตำแหน่งของ `HEAD` เปลี่ยน ไม่ว่าจะด้วยการ commit, checkout, merge, rebase, reset, cherry-pick — Git จะบันทึกการเคลื่อนที่นั้นไว้ในไฟล์ `.git/logs/HEAD`

```bash
git reflog
# หรือเขียนแบบเต็มว่า
git reflog show HEAD
```

ตัวอย่างผลลัพธ์:

```
a3f5c21 (HEAD -> main) HEAD@{0}: reset: moving to HEAD~1
d4e5f67 HEAD@{1}: commit: เพิ่มฟีเจอร์ export CSV
a3f5c21 HEAD@{2}: commit: แก้บั๊กการคำนวณราคา
9c8b7a6 HEAD@{3}: pull: Fast-forward
7f6e5d4 HEAD@{4}: checkout: moving from feature/export to main
3d2c1b0 HEAD@{5}: commit: เริ่มเขียนฟีเจอร์ export
```

**สิ่งสำคัญที่ต้องเข้าใจ:**

- `HEAD@{0}` คือสถานะล่าสุดสุด, `HEAD@{1}` คือก่อนหน้านั้นหนึ่งครั้ง เรียงย้อนกลับไปเรื่อย ๆ
- reflog เป็นข้อมูล **local เท่านั้น** อยู่ใน `.git/logs/` ของเครื่องคุณ ไม่ถูก push, ไม่ถูก clone ไปเครื่องอื่น
- แต่ละ branch ก็มี reflog ของตัวเองแยกต่างหากได้ เช่น `git reflog show main`, `git reflog show feature/export`

### 934.2 ใช้ reflog กู้คืนหลัง `git reset --hard` พลาด

นี่คือ use case ที่พบบ่อยที่สุด:

```bash
# สมมติพลาดรัน reset --hard ไปไกลเกินไป
git reset --hard HEAD~5
# อุ๊ปส์! ลบ commit ไป 5 อันโดยไม่ได้ตั้งใจ

# กู้คืนด้วย reflog
git reflog
# a3f5c21 HEAD@{0}: reset: moving to HEAD~5
# d4e5f67 HEAD@{1}: commit: เพิ่มฟีเจอร์ export CSV   <- นี่คือจุดก่อน reset

# กลับไปยังตำแหน่งก่อน reset ทันที
git reset --hard HEAD@{1}

# หรือเจาะจงด้วย commit hash ตรง ๆ ก็ได้ผลเหมือนกัน
git reset --hard d4e5f67
```

### 934.3 ใช้ reflog กู้คืนหลัง `git branch -D` ลบ branch พลาด

```bash
# ลบ branch โดยไม่ได้ merge ก่อน (พลาด!)
git branch -D feature/important-work
# Deleted branch feature/important-work (was f9e8d7c).

# สังเกตว่า Git บอก commit hash สุดท้ายของ branch ที่ลบไว้ในข้อความเลย!
# ถ้าจำ hash ได้ กู้คืนตรง ๆ ได้ทันที
git branch feature/important-work f9e8d7c
```

**ถ้าจำ hash ไม่ได้และไม่เคย checkout branch นั้นเป็น HEAD มาก่อน** ให้ค้นหาผ่าน reflog ของ `HEAD` (ถ้าเคยยืนอยู่บน branch นั้นตอนที่ commit) หรือใช้ `git fsck` (จะอธิบายใน Step 935):

```bash
git reflog | grep "important-work"
# หรือค้นหาแบบกว้าง ๆ จากข้อความ commit
git reflog | grep -i "commit:"
```

### 934.4 ใช้ reflog เพื่อดูว่า "เมื่อกี้ทำอะไรไปบ้าง" ระหว่าง rebase ที่สับสน

```bash
git reflog
# f1a2b3c HEAD@{0}: rebase (finish): returning to refs/heads/feature/x
# f1a2b3c HEAD@{1}: rebase (pick): แก้ไข validation logic
# e2d3c4b HEAD@{2}: rebase (pick): เพิ่ม unit test
# d3c4b5a HEAD@{3}: rebase (start): checkout main
```

ข้อความ reflog แต่ละบรรทัดจะบอกประเภทของ action ด้วย (`rebase (start)`, `rebase (pick)`, `rebase (finish)`, `commit`, `commit (amend)`, `merge`, `checkout`, `reset`, `cherry-pick`) ทำให้ตามรอยได้ว่าเกิดอะไรขึ้นบ้างในลำดับเวลาที่แน่นอน

### 934.5 อายุของ reflog entry และการตั้งค่า

Git จะเก็บ reflog entry ไว้ตามระยะเวลา default นี้ก่อนที่ `git gc` จะพิจารณาลบทิ้ง:

| ประเภท entry | อายุ default |
|---|---|
| Entry ที่ commit ยัง reachable จาก branch/tag ใด ๆ | 90 วัน (`gc.reflogExpire`) |
| Entry ที่ commit unreachable แล้ว (ไม่มี ref ใดชี้ถึง) | 30 วัน (`gc.reflogExpireUnreachable`) |

ตรวจสอบหรือปรับค่าได้ด้วย:

```bash
git config gc.reflogExpire
git config gc.reflogExpireUnreachable

# ตัวอย่างการยืดอายุ reflog ให้นานขึ้นสำหรับ repo ที่สำคัญมาก
git config gc.reflogExpire "180 days"
git config gc.reflogExpireUnreachable "90 days"
```

### 934.6 ข้อจำกัดของ reflog ที่ต้องรู้

1. **reflog เป็น local เสมอ** — ถ้าเพื่อนร่วมทีมลบ branch บนเครื่องเขา reflog ของคุณช่วยไม่ได้ ต้องดู reflog บนเครื่องของเขาเอง
2. **reflog ไม่ได้บันทึกทุกอย่าง** — มันบันทึกแค่การเคลื่อนที่ของ ref เท่านั้น ไม่ได้บันทึกการเปลี่ยนแปลงใน working directory ที่ไม่เคย commit เลย (ไฟล์ที่ไม่เคย `git add` เลยไม่มีทางกู้คืนผ่าน reflog ได้)
3. **หลัง `git gc` ทำงานจริง entry อาจหายไป** — ถ้า repo รัน `git gc --aggressive` หรือ `git gc --prune=now` entry ที่หมดอายุจะถูกลบถาวร ดังนั้นยิ่งกู้คืนเร็วยิ่งดี

---

## Step 935: Repository เสียหาย (corruption) และการใช้ `git fsck` วินิจฉัย/ซ่อม

Repository เสียหายเป็นปัญหาที่พบไม่บ่อยแต่เมื่อเกิดขึ้นมักน่ากลัวมาก เพราะดูเหมือนทั้ง repo จะใช้งานไม่ได้ทันที สาเหตุที่พบบ่อยคือ ไฟล์ระบบเสียหายระหว่างเขียนดิสก์ (ไฟดับ, เครื่องแฮงค์ระหว่าง commit), disk เสีย, หรือการ sync ผ่านเครื่องมือ cloud sync (Dropbox, Google Drive) ที่ไม่เข้าใจโครงสร้าง `.git`

### 935.1 อาการที่บ่งชี้ว่า repository เสียหาย

```
error: object file .git/objects/4a/f3c8e91d2b5a7f6c3d8e9f0a1b2c3d4e5f6a7b8 is empty
fatal: loose object 4af3c8e91d2b5a7f6c3d8e9f0a1b2c3d4e5f6a7b8 (stored in .git/objects/4a/f3c8e91d2b5a7f6c3d8e9f0a1b2c3d4e5f6a7b8) is corrupt
```

หรือ

```
error: bad object HEAD
fatal: your current branch appears to be broken
```

หรือ

```
fatal: unable to read tree d4e5f67...
error: Could not read d4e5f67...
```

### 935.2 ขั้นตอนวินิจฉัยด้วย `git fsck`

`git fsck` (filesystem check) คือเครื่องมือตรวจสอบความสมบูรณ์ของ object database ทั้งหมดใน repository

```bash
# ตรวจสอบแบบละเอียดที่สุด ครอบคลุมทุก object รวมถึงที่ unreachable
git fsck --full

# ตัวอย่างผลลัพธ์เมื่อพบปัญหา
# error: object file .git/objects/4a/f3c8e91d... is empty
# error: 4af3c8e91d2b5a7f6c3d8e9f0a1b2c3d4e5f6a7b8: object corrupt or missing
# missing blob 4af3c8e91d2b5a7f6c3d8e9f0a1b2c3d4e5f6a7b8
```

**ธง (flags) สำคัญของ `git fsck`:**

| Flag | ความหมาย |
|---|---|
| `--full` | ตรวจสอบทุก object รวมถึงที่อยู่ใน pack file (ไม่ใช่แค่ loose object) |
| `--strict` | เข้มงวดขึ้น ตรวจสอบเรื่องสิทธิ์ไฟล์และรูปแบบ tree object ด้วย |
| `--unreachable` | แสดง object ที่ไม่มี ref ใดชี้ถึง (dangling) โดยไม่ถือว่าเป็นข้อผิดพลาด |
| `--no-reflogs` | ไม่นับ object ที่ถูกอ้างอิงจาก reflog เป็น "reachable" — ใช้เมื่อต้องการหา object ที่ไม่มีใครใช้จริง ๆ |
| `--lost-found` | เขียน dangling commit/blob ทั้งหมดลงในโฟลเดอร์ `.git/lost-found/` ให้ตรวจสอบทีหลัง |

### 935.3 ขั้นตอนซ่อมแซมเมื่อพบ loose object เสียหาย

> **คำเตือนสำคัญ:** **ห้ามรัน `git gc` หรือ `git repack` ทันทีที่พบ corruption** เพราะอาจทำให้ object ที่เสียหายถูกเก็บกวาดทิ้งถาวรก่อนที่คุณจะกู้คืนได้ ให้วินิจฉัยและกู้คืนให้เสร็จก่อนเสมอ

**ทางเลือกที่ 1: กู้คืน object จากแหล่งอื่นที่ยังสมบูรณ์ (แนะนำที่สุด)**

ถ้ามีเพื่อนร่วมทีมที่ clone repo เดียวกันไว้ หรือมี remote ที่ยังมี object นั้นครบ วิธีที่ปลอดภัยที่สุดคือคัดลอกไฟล์ object ที่เสียหายมาจากแหล่งที่สมบูรณ์:

```bash
# object hash ที่เสียหายคือ 4af3c8e9...
# ไฟล์จะอยู่ที่ .git/objects/4a/f3c8e91d2b5a7f6c3d8e9f0a1b2c3d4e5f6a7b8

# คัดลอกไฟล์เดียวกันจากเครื่องเพื่อนร่วมทีม (ผ่าน scp, USB, หรือ shared drive)
scp teammate@host:/path/to/project/.git/objects/4a/f3c8e91d2b5a7f6c3d8e9f0a1b2c3d4e5f6a7b8 \
    .git/objects/4a/f3c8e91d2b5a7f6c3d8e9f0a1b2c3d4e5f6a7b8

# ตรวจสอบว่า object ใช้งานได้แล้ว
git cat-file -t 4af3c8e91d2b5a7f6c3d8e9f0a1b2c3d4e5f6a7b8
git fsck --full
```

**ทางเลือกที่ 2: ดึงข้อมูลใหม่จาก remote (ถ้า remote ยังมี object ครบ)**

```bash
# ลบ ref ที่ชี้ไปยังส่วนที่เสียหาย แล้วดึงใหม่จาก remote ทั้งหมด
git fetch origin --force

# ถ้า branch local เสียหาย ให้รีเซ็ตให้ตรงกับ remote (ระวัง: จะเสีย commit local ที่ยังไม่ push)
git reset --hard origin/main
```

**ทางเลือกที่ 3: กู้ dangling commit ที่ยังอยู่ด้วย `git fsck --lost-found`**

```bash
git fsck --full --no-reflogs --unreachable

# ตัวอย่างผลลัพธ์
# dangling commit d4e5f67890abcdef1234567890abcdef12345678
# dangling blob a1b2c3d4e5f6789012345678901234567890abcd

# ตรวจสอบเนื้อหาของ dangling commit ว่าใช่สิ่งที่ต้องการหรือไม่
git show d4e5f67890abcdef1234567890abcdef12345678

# ถ้าใช่ ให้สร้าง branch กู้คืนออกมา
git branch recovered-commit d4e5f67890abcdef1234567890abcdef12345678
```

**ทางเลือกสุดท้าย: Clone ใหม่จาก remote แล้วย้ายงาน local ที่ยังไม่ push กลับเข้าไป**

ถ้า repository เสียหายรุนแรงจนซ่อมไม่ไหว และ remote คือ source of truth ที่เชื่อถือได้ วิธีที่ปลอดภัยและเร็วที่สุดคือ:

```bash
# 1. เก็บ patch ของงานที่ยังไม่ push ไว้ก่อน (ถ้ายังรันคำสั่งพื้นฐานได้)
git format-patch origin/main --stdout > /tmp/my-unpushed-work.patch

# 2. clone repo ใหม่สะอาด ๆ ไปโฟลเดอร์อื่น
git clone https://github.com/company/project.git project-fresh

# 3. นำ patch ที่เก็บไว้กลับมา apply ใน repo ใหม่
cd project-fresh
git am /tmp/my-unpushed-work.patch
```

### 935.4 การตรวจสุขภาพ repository เป็นประจำเพื่อป้องกันล่วงหน้า

```bash
# รันตรวจสุขภาพแบบไม่ทำลายอะไร เหมาะทำเป็นประจำ (เช่นทุกสัปดาห์ใน repo สำคัญ)
git fsck --full --strict

# บีบอัด object ให้มีประสิทธิภาพ (ควรทำหลังยืนยันว่า repo สมบูรณ์แล้วเท่านั้น)
git gc
```

---

## Step 936: ทำไม push ไม่ได้ — permission, protected branch, file size limit

ปัญหา "push ไม่ได้" มีสาเหตุหลายรูปแบบมาก และแต่ละแบบต้องแก้ต่างกันโดยสิ้นเชิง เราจะไล่ทีละแบบ

### 936.1 Permission denied (publickey)

```
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

**สาเหตุที่เป็นไปได้และวิธีตรวจสอบทีละขั้น:**

```bash
# ขั้นที่ 1: ตรวจสอบว่า SSH key ทำงานได้จริงหรือไม่
ssh -T git@github.com
# ผลลัพธ์ที่ถูกต้อง: Hi <username>! You've successfully authenticated...

# ขั้นที่ 2: ถ้าไม่ผ่าน ตรวจสอบว่ามี SSH key อยู่ในเครื่องหรือไม่
ls -la ~/.ssh/
# ควรเห็น id_ed25519 (หรือ id_rsa) และ id_ed25519.pub

# ขั้นที่ 3: ถ้าไม่มี key ให้สร้างใหม่
ssh-keygen -t ed25519 -C "your_email@example.com"

# ขั้นที่ 4: เพิ่ม key เข้า ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# ขั้นที่ 5: คัดลอก public key ไปเพิ่มใน GitHub/GitLab settings > SSH Keys
cat ~/.ssh/id_ed25519.pub
```

**ตรวจสอบด้วยว่า remote URL ใช้ protocol แบบ SSH จริงหรือเปล่า:**

```bash
git remote -v
# origin  git@github.com:company/project.git (fetch)   <- SSH protocol ถูกต้อง
# origin  https://github.com/company/project.git (fetch) <- HTTPS protocol ต้องใช้ credential คนละแบบ

# ถ้าอยากสลับจาก HTTPS เป็น SSH
git remote set-url origin git@github.com:company/project.git
```

ถ้าใช้ HTTPS แล้วเจอ permission denied มักเกิดจาก personal access token หมดอายุหรือไม่มีสิทธิ์ `repo` scope — ต้องสร้าง token ใหม่จากหน้า Settings ของ GitHub/GitLab

### 936.2 "! [rejected] ... (fetch first)" — Non-fast-forward push

```
! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'github.com:company/project.git'
```

**สาเหตุ:** มีคนอื่น push commit ใหม่ขึ้น remote ไปแล้วตั้งแต่คุณ fetch ครั้งล่าสุด ทำให้ branch ของคุณ "ตามหลัง" remote

**วิธีแก้ (เลือกวิธีใดวิธีหนึ่งตาม convention ของทีม):**

```bash
# วิธีที่ 1: rebase local commit ของเราไปวางต่อจาก remote (ประวัติสะอาดกว่า)
git pull --rebase origin main
git push origin main

# วิธีที่ 2: merge remote เข้ามา (จะมี merge commit เพิ่ม)
git pull origin main
git push origin main

# วิธีที่ 3: fetch แยกจาก merge/rebase เพื่อดูก่อนตัดสินใจ (ปลอดภัยที่สุด)
git fetch origin
git log HEAD..origin/main --oneline   # ดูว่า remote มี commit อะไรใหม่บ้าง
git rebase origin/main                # หรือ git merge origin/main
git push origin main
```

### 936.3 Protected branch rejected

```
remote: error: GH006: Protected branch update failed for refs/heads/main.
remote: error: Required status check "ci/build" is expected.
To github.com:company/project.git
 ! [remote rejected] main -> main (protected branch hook declined)
error: failed to push some refs to 'github.com:company/project.git'
```

**สาเหตุ:** branch `main` ถูกตั้งค่า **branch protection rule** ไว้ (เช่น ห้าม push ตรง ต้องผ่าน Pull Request, ต้องผ่าน status check, ต้องมี reviewer approve อย่างน้อย 1 คน)

**วิธีแก้ — ไม่มีทางลัดที่ปลอดภัย ต้องทำตามกระบวนการที่ทีมกำหนดไว้:**

```bash
# 1. สร้าง branch ใหม่จากงานของคุณ (ถ้ายังไม่มี)
git checkout -b fix/update-main-content

# 2. push branch ใหม่นี้แทน (ไม่ใช่ main โดยตรง)
git push -u origin fix/update-main-content

# 3. เปิด Pull Request บน GitHub/GitLab เพื่อขอ merge เข้า main
#    แล้วรอ required status check ผ่านและรอ reviewer อนุมัติตามกฎที่ตั้งไว้
```

**ถ้าจำเป็นต้องแก้กฎจริง ๆ** (เช่นเป็น emergency hotfix และมีสิทธิ์ admin) ต้องเข้าไปที่ Settings → Branches ของ repository แล้วปรับ/ปิด branch protection ชั่วคราว **แต่ไม่ควรทำเป็นเรื่องปกติ** เพราะขัดกับเจตนาของการตั้ง protection ไว้ตั้งแต่แรก

### 936.4 File size limit exceeded

```
remote: error: File assets/large-video.mp4 is 150.00 MB; this exceeds GitHub's file size limit of 100.00 MB
remote: error: GH001: Large files detected. You may want to try Git Large File Storage
To github.com:company/project.git
 ! [remote rejected] main -> main (pre-receive hook declined)
error: failed to push some refs to 'github.com:company/project.git'
```

**กรณีที่ 1: ไฟล์ใหญ่อยู่ใน commit ล่าสุดที่ยังไม่ push เท่านั้น**

```bash
# แก้ commit ล่าสุดโดยเอาไฟล์ใหญ่ออก
git rm --cached assets/large-video.mp4
echo "assets/large-video.mp4" >> .gitignore
git add .gitignore
git commit --amend --no-edit

# ตรวจสอบว่าไฟล์หลุดออกจาก commit แล้วจริง
git show --stat HEAD

git push origin main
```

**กรณีที่ 2: ไฟล์ใหญ่ถูก commit ไปหลาย commit ก่อนหน้าแล้ว (ฝังอยู่ในประวัติลึก)**

ต้องใช้เครื่องมือลบไฟล์ออกจากประวัติทั้งหมด เช่น `git filter-repo` (เครื่องมือที่ Git แนะนำอย่างเป็นทางการในปัจจุบัน แทนที่ `git filter-branch` รุ่นเก่าที่ช้าและอันตรายกว่า):

```bash
# ติดตั้ง git-filter-repo ก่อน (ผ่าน pip หรือ package manager)
pip install git-filter-repo

# ลบไฟล์ออกจากทุก commit ในประวัติทั้งหมด
git filter-repo --path assets/large-video.mp4 --invert-paths

# เพิ่มเข้า .gitignore เพื่อป้องกันไม่ให้ถูก commit ซ้ำ
echo "assets/large-video.mp4" >> .gitignore
git add .gitignore
git commit -m "เพิ่ม .gitignore ป้องกันไฟล์วิดีโอขนาดใหญ่"

# ต้อง force push เพราะประวัติถูกเขียนใหม่ทั้งหมด (ต้องแจ้งทีมล่วงหน้าเสมอ)
git push origin main --force-with-lease
```

> **คำเตือนสำคัญ:** การเขียนประวัติใหม่ (`git filter-repo`, `git filter-branch`, หรือ `BFG Repo-Cleaner`) จะเปลี่ยน commit hash ของทุก commit ที่ได้รับผลกระทบ **ทุกคนในทีมต้อง re-clone หรือ hard reset local branch ของตัวเองใหม่** มิฉะนั้นจะเกิดปัญหาประวัติแตกกันไปคนละทาง ควรแจ้งทีมและวางแผนช่วงเวลาที่ไม่มีใครทำงานค้างอยู่ก่อนทำเสมอ

**สำหรับอนาคต — ป้องกันปัญหานี้ตั้งแต่ต้นด้วย Git LFS:**

```bash
git lfs install
git lfs track "*.mp4"
git lfs track "*.psd"
git add .gitattributes
git add assets/large-video.mp4
git commit -m "ย้ายไฟล์วิดีโอไปใช้ Git LFS"
git push origin main
```

### 936.5 ตารางสรุปปัญหา push แบบต่าง ๆ

| อาการ error | สาเหตุ | วิธีแก้หลัก |
|---|---|---|
| `Permission denied (publickey)` | SSH key ไม่ถูกต้อง/ไม่ได้เพิ่ม | ตรวจสอบ `ssh -T`, เพิ่ม key ใหม่ |
| `(fetch first)` | remote มี commit ใหม่กว่า | `git pull --rebase` แล้ว push ใหม่ |
| `protected branch hook declined` | branch มี protection rule | เปิด PR แทนการ push ตรง |
| `exceeds file size limit` | ไฟล์ใหญ่เกินที่ remote อนุญาต | ลบออกจาก commit/ประวัติ หรือใช้ Git LFS |
| `403` ทั่วไป | ไม่มีสิทธิ์เขียนใน repo นั้น | ตรวจสอบสิทธิ์ collaborator กับ admin |

---

## Step 937: ปัญหา line ending ระหว่างทีมที่ใช้ OS ต่างกัน

ปัญหานี้เป็นปัญหาคลาสสิกของทีมที่มีทั้งคนใช้ Windows และ macOS/Linux ทำงานร่วมกัน

### 937.1 อาการของปัญหา

```
warning: LF will be replaced by CRLF in src/utils/helper.js.
The file will have its original line endings in your working directory
```

หรืออาการที่ร้ายแรงกว่าคือเปิด diff แล้วเห็นว่า **ทั้งไฟล์ถูกมองว่าเปลี่ยนแปลงทั้งหมด** ทั้งที่จริง ๆ แล้วแก้แค่บรรทัดเดียว เพราะ line ending ต่างกันทั้งไฟล์ (CRLF vs LF)

```bash
git diff --stat
#  src/utils/helper.js | 200 +++++++++++++++++++++++++++++++++++++++++++++++++
#  1 file changed, 100 insertions(+), 100 deletions(-)
```

ทั้งที่จริง ๆ ไฟล์นี้มี 100 บรรทัดเท่าเดิม แค่ปลายบรรทัดต่างกัน (`\r\n` vs `\n`) ก็ทำให้ Git มองว่าทุกบรรทัดถูกลบแล้วเพิ่มใหม่หมด

### 937.2 ย้อนดูรากของปัญหา: `core.autocrlf` (จาก Part 02)

ตามที่เรียนไปแล้วใน Part 02 ค่า `core.autocrlf` ควบคุมการแปลง line ending ตอน checkout/commit:

| ค่า | พฤติกรรม | เหมาะกับ |
|---|---|---|
| `true` | checkout เป็น CRLF, commit แปลงกลับเป็น LF อัตโนมัติ | Windows |
| `input` | checkout ตามที่เก็บไว้ (ไม่แปลง), commit แปลง CRLF→LF เสมอ | macOS/Linux |
| `false` | ไม่แปลงอะไรเลยทั้งสองทาง | ไม่แนะนำสำหรับทีมผสม OS |

**ปัญหาที่แท้จริงคือ:** เมื่อสมาชิกในทีมแต่ละคนตั้งค่า `core.autocrlf` ไม่เหมือนกัน (บางคน `true`, บางคน `false`, บางคน `input`) การ commit จากแต่ละเครื่องจะได้ line ending ที่ไม่สม่ำเสมอกันไปฝังอยู่ใน repository ทำให้เกิดปัญหาไล่ไม่จบ

### 937.3 ทางแก้ที่ถูกต้องระดับทีม: `.gitattributes` (จาก Part 61)

การพึ่งค่า `core.autocrlf` ส่วนตัวของแต่ละคนไม่มีทางแก้ปัญหาได้ถาวร เพราะเป็นการตั้งค่าฝั่ง client ที่บังคับคนอื่นไม่ได้ วิธีที่ถูกต้องคือกำหนดกฎ line ending ไว้ที่ระดับ repository ผ่านไฟล์ `.gitattributes` ซึ่งจะมีผลกับทุกคนที่ clone repo นี้เหมือนกันหมด ไม่ว่าจะตั้งค่า `core.autocrlf` ส่วนตัวไว้อย่างไร:

```gitattributes
# บังคับให้ Git normalize line ending อัตโนมัติสำหรับไฟล์ที่ตรวจพบว่าเป็น text
* text=auto

# source code ต้องเป็น LF เสมอ ไม่ว่าจะ commit จาก OS ไหน
*.js text eol=lf
*.ts text eol=lf
*.jsx text eol=lf
*.json text eol=lf
*.css text eol=lf
*.md text eol=lf

# ไฟล์ shell script ต้องเป็น LF เสมอ (สำคัญมาก ถ้าเป็น CRLF จะรันบน Linux ไม่ได้เลย)
*.sh text eol=lf

# ไฟล์เฉพาะ Windows ที่ต้องการ CRLF จริง ๆ
*.bat text eol=crlf
*.ps1 text eol=crlf

# ไฟล์ binary ห้ามแตะ line ending เด็ดขาด
*.png binary
*.jpg binary
*.ico binary
*.pdf binary
```

### 937.4 การตรวจวินิจฉัยว่าไฟล์ปัจจุบันมี line ending แบบไหน

```bash
# ตรวจสอบด้วยคำสั่ง file (บน Linux/macOS)
file src/utils/helper.js
# src/utils/helper.js: ASCII text, with CRLF line terminators   <- พบปัญหา

# ตรวจสอบว่า attribute ปัจจุบันมีผลกับไฟล์นี้อย่างไร
git check-attr text eol -- src/utils/helper.js
# src/utils/helper.js: text: set
# src/utils/helper.js: eol: lf

# ตรวจสอบค่า core.autocrlf ปัจจุบันของตัวเอง
git config core.autocrlf
```

### 937.5 การแก้ไขไฟล์ที่ commit ไปแล้วด้วย line ending ผิด ๆ

หลังจากเพิ่ม `.gitattributes` ใหม่ ไฟล์ที่ **commit ไปแล้วในอดีต** จะยัง**ไม่**ถูกแก้ไขย้อนหลังโดยอัตโนมัติ ต้องสั่งให้ Git renormalize ไฟล์ที่มีอยู่:

```bash
# ขั้นที่ 1: เพิ่ม/แก้ .gitattributes ให้เรียบร้อยก่อน แล้ว commit
git add .gitattributes
git commit -m "เพิ่มกฎ line ending ผ่าน .gitattributes"

# ขั้นที่ 2: สั่งให้ Git renormalize ไฟล์ที่มีอยู่ทั้งหมดตามกฎใหม่
git add --renormalize .

# ขั้นที่ 3: ตรวจสอบว่ามีไฟล์ไหนถูกแก้ line ending บ้าง
git status

# ขั้นที่ 4: commit การ renormalize แยกเป็น commit เดี่ยว ๆ ต่างหาก
# (เพื่อไม่ให้ diff ของ commit นี้ปนกับการแก้โค้ดจริง ทำให้ git blame ยังใช้งานได้ดี)
git commit -m "Normalize line endings ทั้งโปรเจกต์ตาม .gitattributes"
```

> **เคล็ดลับสำคัญ:** ทำ commit สำหรับ renormalize line ending แยกต่างหากเสมอ และแจ้งทีมให้รู้ commit hash นี้ไว้ เพื่อให้ทุกคนใช้ `git blame --ignore-rev <hash>` หรือใส่ hash นี้ในไฟล์ `.git-blame-ignore-revs` เพื่อไม่ให้ประวัติ blame ของโค้ดจริงถูกบดบังด้วย commit ที่แค่เปลี่ยน line ending

### 937.6 ทดสอบว่าตั้งค่าถูกต้องหลังแก้ไข

```bash
# ลอง clone repo ใหม่ในโฟลเดอร์ทดสอบเพื่อยืนยันว่า .gitattributes ทำงานถูกต้องสำหรับคนที่ clone ใหม่
git clone https://github.com/company/project.git /tmp/test-clone
cd /tmp/test-clone
file src/utils/helper.js
# src/utils/helper.js: ASCII text   <- ไม่มี CRLF แล้ว ถูกต้อง
```

---

## Step 938: ปัญหา submodule ทำงานผิดพลาด — วิธี debug แบบเป็นระบบ

Submodule (ที่เรียนละเอียดใน Part 44) เป็นหนึ่งในฟีเจอร์ของ Git ที่สร้างปัญหาปวดหัวให้ทีมบ่อยที่สุด เพราะพฤติกรรมของมันไม่ค่อยตรงกับสัญชาตญาณ

### 938.1 อาการที่ 1: โฟลเดอร์ submodule ว่างเปล่าหลัง clone

```bash
git clone https://github.com/company/main-project.git
cd main-project
ls libs/shared-utils/
# (ว่างเปล่า ไม่มีไฟล์อะไรเลย)
```

**สาเหตุ:** `git clone` ธรรมดา **ไม่ได้** ดึงเนื้อหาของ submodule มาด้วยอัตโนมัติ มันแค่สร้างโฟลเดอร์เปล่าไว้เป็น placeholder เท่านั้น

**วิธีแก้:**

```bash
# วิธีที่ 1: ถ้ายังไม่ได้ clone ให้ clone แบบดึง submodule มาด้วยตั้งแต่ต้น
git clone --recurse-submodules https://github.com/company/main-project.git

# วิธีที่ 2: ถ้า clone ไปแล้วแบบธรรมดา ให้ init และ update submodule ทีหลัง
git submodule update --init --recursive
```

`--recursive` สำคัญมากในกรณีที่ submodule นั้นมี submodule ซ้อนอยู่ข้างในอีกที (nested submodules)

### 938.2 อาการที่ 2: `git status` ที่ parent repo แสดง submodule ว่า "modified content"

```
Changes not staged for commit:
	modified:   libs/shared-utils (modified content, untracked content)
```

**ขั้นตอน debug แบบเป็นระบบ:**

```bash
# ขั้นที่ 1: ดูสถานะ submodule ทั้งหมดในโปรเจกต์ก่อน
git submodule status

# ผลลัพธ์ตัวอย่างพร้อมความหมายของ prefix:
#  a1b2c3d libs/shared-utils (heads/main)     <- ไม่มี prefix = sync ตรงกับที่ parent record ไว้
# +f9e8d7c libs/another-lib (heads/main)      <- '+' = commit ปัจจุบันไม่ตรงกับที่ parent เก็บ SHA ไว้
# -e5f6a7b libs/uninit-lib                    <- '-' = ยังไม่ได้ init เลย
# U1a2b3c4 libs/conflicted-lib                <- 'U' = มี merge conflict ค้างอยู่ในตัว submodule เอง

# ขั้นที่ 2: เข้าไปดูข้างในตัว submodule โดยตรง เหมือนเป็น repo อิสระ
cd libs/shared-utils
git status
git log -1 --oneline
git remote -v
cd ../..
```

**ถ้าเจอ prefix `+` (commit ไม่ตรงกับที่ parent เก็บไว้):**

```bash
# เช็คว่าเป็นเพราะมีคนเข้าไปแก้ไข/commit ข้างในตัว submodule เองโดยไม่ได้ตั้งใจ
cd libs/another-lib
git log --oneline -5
git diff <sha-ที่-parent-record-ไว้> HEAD

# ถ้าต้องการให้ submodule กลับไปตรงกับ commit ที่ parent กำหนดไว้ (ทิ้งการเปลี่ยนแปลงในนั้น)
cd ../..
git submodule update libs/another-lib

# ถ้าต้องการ "ยอมรับ" commit ใหม่ในตัว submodule แล้วอัปเดต pointer ของ parent
cd libs/another-lib
git checkout main
git pull
cd ../..
git add libs/another-lib
git commit -m "อัปเดต submodule another-lib เป็นเวอร์ชันล่าสุด"
```

### 938.3 อาการที่ 3: `.gitmodules` กับ `.git/config` ไม่ตรงกัน (URL ของ submodule เปลี่ยนไป)

เกิดขึ้นเมื่อมีการเปลี่ยน URL ของ submodule ใน `.gitmodules` (เช่นย้าย repo ไปโฮสต์ที่อื่น) แต่ local config ของแต่ละคนยังจำ URL เก่าอยู่:

```bash
error: Server does not allow request for unadvertised object ...
fatal: clone of 'https://old-url.com/shared-utils.git' into submodule path 'libs/shared-utils' failed
```

**วิธีแก้:**

```bash
# sync URL ของ submodule จาก .gitmodules เข้าไปที่ .git/config ให้ตรงกันใหม่
git submodule sync --recursive

# จากนั้น update ใหม่อีกครั้ง
git submodule update --init --recursive
```

### 938.4 อาการที่ 4: submodule พังหนักมากจนต้อง reset ทั้งหมด

เมื่อ submodule อยู่ในสถานะสับสนมาก (เช่น commit ที่ parent อ้างอิงไม่มีอยู่บน remote แล้ว, หรือโฟลเดอร์เสียหาย) วิธีที่ปลอดภัยที่สุดคือ deinit แล้วเริ่มใหม่ทั้งหมด:

```bash
# ขั้นที่ 1: deinit submodule (ลบเนื้อหาออกจาก working directory แต่ .gitmodules ยังอยู่)
git submodule deinit -f libs/shared-utils

# ขั้นที่ 2: ลบโฟลเดอร์ cache ของ submodule ใน .git ด้วย (สำคัญ ไม่งั้น re-init อาจติด error เดิม)
rm -rf .git/modules/libs/shared-utils

# ขั้นที่ 3: init และ update ใหม่ทั้งหมดตั้งแต่ต้น
git submodule update --init --recursive libs/shared-utils
```

### 938.5 อาการที่ 5: commit ที่ parent อ้างอิงไม่มีอยู่บน remote ของ submodule แล้ว

```
fatal: remote error: upload-pack: not our ref a1b2c3d4e5f6...
Fetched in submodule path 'libs/shared-utils', but it did not contain
a1b2c3d4e5f6... Direct fetching of that commit failed.
```

**สาเหตุ:** commit ที่ parent repo บันทึก pointer ไว้ถูกลบทิ้งไปจาก history ของ submodule แล้ว (เช่นมีคน force-push หรือ rebase history ของ submodule)

**วิธีวินิจฉัยและแก้:**

```bash
# ขั้นที่ 1: เข้าไปดูว่า commit ที่ parent ต้องการ อยู่ที่ branch ไหนของ submodule บ้าง (ถ้ายังพอหาได้)
cd libs/shared-utils
git log --all --oneline | grep a1b2c3d
git branch -a --contains a1b2c3d4e5f6

# ขั้นที่ 2: ถ้าหาไม่เจอเลย ต้องคุยกับทีมที่ดูแล submodule ว่า commit หายไปจริงหรือ URL เปลี่ยน
# ขั้นที่ 3: ถ้าตัดสินใจได้ว่าจะอัปเดตไปใช้ commit ใหม่ล่าสุดแทน
git checkout main
git pull
cd ../..
git add libs/shared-utils
git commit -m "อัปเดต submodule shared-utils เพราะ commit เดิมหายไปจาก history"
```

### 938.6 เช็คลิสต์ debug submodule แบบเป็นระบบ

1. `git submodule status` — ดู prefix ก่อนเสมอ (`+`, `-`, `U`, หรือไม่มี)
2. `cd` เข้าไปในตัว submodule แล้วปฏิบัติกับมันเหมือน repo อิสระตัวหนึ่ง (`git status`, `git log`, `git remote -v`)
3. `git submodule sync` เมื่อสงสัยว่า URL ไม่ตรงกัน
4. `git submodule update --init --recursive` เมื่อสงสัยว่ายังไม่ได้ดึงเนื้อหามา
5. deinit + ลบ `.git/modules/<path>` + init ใหม่ เมื่อพังหนักจนซ่อมจุดเดียวไม่พอ

---

## Step 939: เครื่องมือช่วย debug ขั้นสูง (`git log` เจาะลึก, GUI tools)

นอกจากคำสั่งพื้นฐานแล้ว Git ยังมีเครื่องมือขั้นสูงที่ช่วยสืบหาสาเหตุของปัญหาได้แม่นยำกว่ามาก

### 939.1 `git log --all --graph --oneline --decorate` — เห็นภาพรวมทั้งหมด

```bash
git log --all --graph --oneline --decorate
```

คำสั่งนี้แสดง:
- `--all` — ทุก branch/tag ในเครื่อง ไม่ใช่แค่ branch ปัจจุบัน
- `--graph` — วาดเส้นกราฟให้เห็นการแตกและรวมของ branch ชัดเจน
- `--oneline` — แสดงแต่ละ commit บรรทัดเดียว กระชับ
- `--decorate` — แสดงว่า branch/tag ไหนชี้ไปที่ commit ไหนบ้าง

มีประโยชน์มากเวลาต้องการเข้าใจว่า "commit หายไปอยู่ตรงไหนของ tree" หรือ "branch ไหน merge เข้าอะไรไปแล้วบ้าง"

### 939.2 `git log -S` (pickaxe) — หาว่าโค้ดชิ้นหนึ่งถูกเพิ่ม/ลบตอนไหน

```bash
# หา commit ที่ทำให้จำนวนครั้งที่ปรากฏของ string นี้เปลี่ยนไป (เพิ่มขึ้นหรือลดลง)
git log -S"calculateDiscount" --oneline -- src/pricing/

# ดูรายละเอียดพร้อม diff ของแต่ละ commit ที่เจอ
git log -S"calculateDiscount" -p -- src/pricing/
```

`-S` ใช้เวลาต้องการหาว่า "ฟังก์ชันนี้ถูกลบไปตอนไหน" หรือ "ใครเป็นคนเพิ่มโค้ดบรรทัดนี้เข้ามาครั้งแรก" ซึ่งมีประโยชน์มากเวลา debug บั๊กที่เกิดจากการเปลี่ยนแปลงในอดีต

### 939.3 `git log -G` — ค้นหาด้วย regular expression

```bash
# ต่างจาก -S ตรงที่ -G จะจับคู่กับ diff ที่ "เนื้อหาการเปลี่ยนแปลง" match กับ regex
git log -G"function\s+calculate\w+" --oneline
```

`-S` เหมาะกับการนับจำนวนครั้งที่ string ปรากฏเปลี่ยนไป ส่วน `-G` เหมาะกับการหา pattern ที่ซับซ้อนกว่าในเนื้อหา diff โดยตรง

### 939.4 `git bisect` — หา commit ที่ทำให้เกิดบั๊กแบบ binary search

เมื่อรู้ว่าโค้ด "เคยดีอยู่" แต่ตอนนี้พังแล้ว และไม่รู้ว่า commit ไหนเป็นต้นเหตุในบรรดา commit นับร้อย `git bisect` จะช่วยหาให้อย่างมีประสิทธิภาพด้วยวิธี binary search:

```bash
# เริ่มกระบวนการ bisect
git bisect start

# บอกว่า commit ปัจจุบัน (HEAD) มีบั๊ก
git bisect bad

# บอกว่า commit เก่า ๆ ที่รู้ว่ายังดีอยู่คือตัวไหน
git bisect good v1.5.0

# Git จะ checkout ไปที่ commit กึ่งกลางให้อัตโนมัติ ให้ทดสอบแล้วบอกผล
# ทดสอบเสร็จแล้วบอก Git ว่า commit นี้ดีหรือแย่
git bisect good   # หรือ git bisect bad

# ทำซ้ำจนกว่า Git จะบอก commit ที่เป็นต้นเหตุแน่ชัด
# ผลลัพธ์ตัวอย่าง:
# a1b2c3d is the first bad commit

# เมื่อหาเจอแล้ว ออกจากโหมด bisect กลับสู่สถานะเดิม
git bisect reset
```

**เทคนิคขั้นสูง:** ถ้ามีสคริปต์ทดสอบอัตโนมัติที่ return exit code ถูกต้อง (0 = ดี, ไม่ใช่ 0 = แย่) ให้ใช้ `git bisect run` เพื่อให้ทำงานอัตโนมัติทั้งหมดโดยไม่ต้องเข้าไปกดยืนยันทีละรอบ:

```bash
git bisect start HEAD v1.5.0
git bisect run npm test
```

### 939.5 `git blame` และ `git log --follow` — ตามรอยไฟล์และบรรทัด

```bash
# ดูว่าแต่ละบรรทัดของไฟล์ถูกแก้ล่าสุดโดยใครและ commit ไหน
git blame -L 40,60 src/payment/checkout.js

# ดูประวัติของไฟล์แม้จะเคยถูกเปลี่ยนชื่อมาก่อน (--follow ตามรอยการ rename ให้)
git log --follow --oneline -- src/payment/checkout.js

# ดู diff ของแต่ละ commit ที่แก้ไฟล์นี้ด้วย
git log --follow -p -- src/payment/checkout.js
```

### 939.6 ภาพรวมสั้น ๆ ของ GUI tools สำหรับ debug ที่ซับซ้อน

แม้ command line จะให้ความแม่นยำและควบคุมได้เต็มที่ที่สุด แต่บางสถานการณ์ — โดยเฉพาะการดูกราฟ branch ที่ซับซ้อนมาก หรือแก้ conflict ทีละไฟล์แบบเห็นภาพ 3 ฝั่ง (ours/theirs/base) — เครื่องมือ GUI ช่วยลดความผิดพลาดได้มาก:

| เครื่องมือ | จุดเด่น | เหมาะกับ |
|---|---|---|
| **GitKraken** | แสดงกราฟ branch สวยงาม ลากวางเพื่อ drag-to-merge/rebase ได้ | ทีมที่มี branch ซับซ้อนจำนวนมาก |
| **Sourcetree** | ฟรี, ใช้ง่าย, มี visual conflict resolution ในตัว | มือใหม่ถึงระดับกลางที่อยากเห็นภาพชัด |
| **GitHub Desktop** | เรียบง่าย ผูกกับ GitHub โดยตรง | ทีมที่ใช้ GitHub เป็นหลักและต้องการเครื่องมือเบา ๆ |
| **lazygit** | เป็น terminal UI (TUI) เร็วมาก ควบคุมด้วยคีย์บอร์ดล้วน | สาย command line ที่อยากได้ภาพรวมเร็ว ๆ โดยไม่ออกจาก terminal |
| **VS Code (GitLens extension)** | ดู blame แบบ inline ในโค้ดได้ทันที เจาะลึกประวัติแต่ละบรรทัด | นักพัฒนาที่ใช้ VS Code เป็นหลักอยู่แล้ว |

**ข้อแนะนำสำคัญ:** ไม่ว่าจะใช้ GUI tool ใดก็ตาม ควรเข้าใจว่ามันทำงานอย่างไรผ่านคำสั่ง Git เบื้องหลังเสมอ เพราะเมื่อเจอสถานการณ์ที่ GUI จัดการไม่ได้หรือแสดงผลผิดพลาด ความรู้พื้นฐานเรื่องคำสั่ง Git แบบ command line คือสิ่งเดียวที่จะช่วยให้แก้ปัญหาต่อได้เสมอ

---

## Step 940: แบบฝึกหัด — แก้ปัญหา Git 5 สถานการณ์จำลอง

ถึงเวลาลงมือฝึกจริง ตั้งค่าสถานการณ์จำลองทั้ง 5 แบบด้วยตัวเอง แล้วฝึกกู้คืนตามขั้นตอนที่เรียนมา

### เตรียมโฟลเดอร์ฝึกฝน

```bash
mkdir ~/git-course/part-94-debugging
cd ~/git-course/part-94-debugging
git init incident-lab
cd incident-lab
git config user.email "you@example.com"
git config user.name "Your Name"

echo "console.log('hello');" > app.js
git add app.js
git commit -m "commit แรก"

echo "console.log('feature 1');" >> app.js
git commit -am "เพิ่มฟีเจอร์ 1"

echo "console.log('feature 2');" >> app.js
git commit -am "เพิ่มฟีเจอร์ 2"
```

### สถานการณ์ที่ 1: Detached HEAD

**จำลองปัญหา:**

```bash
git log --oneline
# ดู hash ของ commit แรก แล้ว checkout ไปที่นั่น
git checkout <hash-ของ-commit-แรก>

echo "console.log('งานลับที่ทำใน detached HEAD');" >> app.js
git commit -am "งานที่ทำใน detached HEAD"

# แล้วสลับออกไป main แบบไม่ทันคิด (จำลองความผิดพลาด)
git checkout main
```

**โจทย์:** กู้คืนงานที่ทำไว้ใน detached HEAD กลับมาเป็น branch ชื่อ `recovered-work` ให้สำเร็จ

**แนวทางเฉลย:**

```bash
git reflog
# หา entry ที่เป็น "commit: งานที่ทำใน detached HEAD"
git branch recovered-work <hash-ที่เจอ>
git log recovered-work --oneline
```

### สถานการณ์ที่ 2: กู้คืนด้วย reflog หลัง reset --hard พลาด

**จำลองปัญหา:**

```bash
git checkout main
git log --oneline
git reset --hard HEAD~2
git log --oneline
# จะเห็นว่า 2 commit ล่าสุดหายไปจาก log แล้ว
```

**โจทย์:** กู้คืน branch `main` ให้กลับไปมี commit ครบทั้งหมดเหมือนก่อน reset

**แนวทางเฉลย:**

```bash
git reflog
# หา entry ก่อนบรรทัด "reset: moving to HEAD~2"
git reset --hard HEAD@{1}
git log --oneline
```

### สถานการณ์ที่ 3: ใช้ `git fsck` ตามหา commit ที่ถูกลบทิ้ง

**จำลองปัญหา:**

```bash
git checkout -b temp-branch
echo "งานสำคัญที่เกือบหาย" >> app.js
git commit -am "งานสำคัญ"

git checkout main
git branch -D temp-branch
# สมมติว่าลืม hash ไปแล้วด้วย (จำลองสถานการณ์เลวร้ายที่สุด)
```

**โจทย์:** ใช้ `git fsck` ตามหา commit ที่หายไปโดยไม่พึ่ง reflog เลย แล้วกู้คืนกลับมา

**แนวทางเฉลย:**

```bash
git fsck --full --unreachable
# หา "unreachable commit <hash>" ที่มีข้อความตรงกับที่คาดไว้
git show <hash-ที่เจอ>
git branch recovered-temp <hash-ที่เจอ>
```

### สถานการณ์ที่ 4: Push rejected เพราะ non-fast-forward

**จำลองปัญหา (ใช้ 2 clone จำลอง 2 คนในทีม):**

```bash
cd ~/git-course/part-94-debugging
git clone incident-lab incident-lab-clone-a
git clone incident-lab incident-lab-clone-b

# คนที่ A แก้และ push ก่อน
cd incident-lab-clone-a
echo "แก้จากเครื่อง A" >> app.js
git commit -am "แก้จากเครื่อง A"
git push origin main

# คนที่ B แก้ (คนละบรรทัด) แล้วพยายาม push โดยยังไม่ pull ก่อน
cd ../incident-lab-clone-b
echo "แก้จากเครื่อง B" >> app.js
git commit -am "แก้จากเครื่อง B"
git push origin main
# คาดว่าจะถูก reject
```

**โจทย์:** แก้ปัญหาให้ push จากเครื่อง B สำเร็จ โดยไม่ทำให้งานของเครื่อง A หายไป

**แนวทางเฉลย:**

```bash
cd incident-lab-clone-b
git fetch origin
git log HEAD..origin/main --oneline
git pull --rebase origin main
# ถ้ามี conflict ให้แก้ตามขั้นตอน Step 933 ก่อน
git push origin main
```

### สถานการณ์ที่ 5: Submodule พัง

**จำลองปัญหา:**

```bash
cd ~/git-course/part-94-debugging
git init shared-lib
cd shared-lib
echo "shared code" > lib.js
git add lib.js
git commit -m "commit แรกของ shared-lib"
cd ..

cd incident-lab
git submodule add ../shared-lib libs/shared-lib
git commit -m "เพิ่ม submodule shared-lib"
cd ..

# จำลอง clone ใหม่แบบไม่ระบุ --recurse-submodules (ปัญหาที่พบบ่อยที่สุด)
git clone incident-lab incident-lab-fresh
cd incident-lab-fresh
ls libs/shared-lib/
# พบว่าโฟลเดอร์ว่างเปล่า
```

**โจทย์:** ทำให้โฟลเดอร์ `libs/shared-lib` มีเนื้อหาไฟล์ `lib.js` ปรากฏขึ้นมาให้ถูกต้อง

**แนวทางเฉลย:**

```bash
git submodule status
git submodule update --init --recursive
cat libs/shared-lib/lib.js
```

### เช็คลิสต์ยืนยันว่าทำแบบฝึกหัดครบ

- [ ] สถานการณ์ 1: กู้คืน detached HEAD กลับมาเป็น branch สำเร็จ
- [ ] สถานการณ์ 2: กู้คืน commit หลัง `reset --hard` ด้วย `git reflog` สำเร็จ
- [ ] สถานการณ์ 3: ใช้ `git fsck --unreachable` ตามหา commit ที่ branch ถูกลบไปแล้วสำเร็จ
- [ ] สถานการณ์ 4: แก้ปัญหา push rejected แบบ non-fast-forward โดยไม่ทำงานของใครหาย
- [ ] สถานการณ์ 5: แก้ปัญหา submodule ว่างเปล่าหลัง clone สำเร็จ

---

## สรุป Part 94

ใน Part นี้เราได้ฝึกฝนทักษะการ debug ปัญหา Git ที่ซับซ้อนที่สุดที่จะพบได้ในองค์กรจริง:

1. **กรอบความคิด STOP → OBSERVE → REPRODUCE → ISOLATE → FIX → VERIFY** คือรากฐานสำคัญที่สุดก่อนลงมือแก้ปัญหาใด ๆ และต้องจำไว้เสมอว่า Git แทบไม่เคยทำให้ข้อมูลหายจริงในทันที
2. **Detached HEAD** แก้ได้ง่ายด้วย `git switch -c <branch>` ถ้ายังไม่สลับออกไป หรือใช้ `git reflog` ตามหา commit แล้วสร้าง branch กู้คืนถ้าสลับออกไปแล้ว
3. **Merge conflict ที่ซับซ้อน** ควร `git merge --abort` (หรือ `rebase --abort`/`cherry-pick --abort`) อย่างปลอดภัยก่อนเสมอ แล้ววางแผนใหม่แบบเป็นระบบ แบ่งไฟล์ตามความยาก
4. **`git reflog`** คือตาข่ายนิรภัยที่สำคัญที่สุด เก็บบันทึกการเคลื่อนที่ของ `HEAD` ไว้ในเครื่อง local ช่วยกู้คืนได้เกือบทุกสถานการณ์ ตราบใดที่ `git gc` ยังไม่เก็บกวาดทิ้ง
5. **Repository เสียหาย** วินิจฉัยด้วย `git fsck --full` และห้ามรัน `git gc` ก่อนกู้คืนให้เสร็จเด็ดขาด ทางเลือกที่ปลอดภัยที่สุดคือกู้ object จากแหล่งอื่นที่สมบูรณ์
6. **ปัญหา push ไม่ได้** มีหลายรูปแบบ ต้องวินิจฉัยจาก error message ให้ถูกก่อน (permission, non-fast-forward, protected branch, file size limit) แล้วแก้ตามสาเหตุที่แท้จริง
7. **ปัญหา line ending** แก้ได้ถาวรด้วย `.gitattributes` ระดับ repository ไม่ใช่พึ่งแค่ `core.autocrlf` ส่วนตัวของแต่ละคน และใช้ `git add --renormalize .` เพื่อแก้ไฟล์เก่าที่ commit ไปแล้ว
8. **ปัญหา submodule** ต้อง debug อย่างเป็นระบบ เริ่มจาก `git submodule status` ดู prefix แล้วเข้าไปตรวจสอบข้างในตัว submodule เหมือนเป็น repo อิสระ
9. **เครื่องมือขั้นสูง** เช่น `git log -S`, `git bisect`, `git blame` ช่วยสืบหาสาเหตุของบั๊กในประวัติได้แม่นยำ ส่วน GUI tools ช่วยให้เห็นภาพรวมชัดขึ้นในสถานการณ์ซับซ้อน
10. การฝึกแก้ปัญหาจำลองทั้ง 5 สถานการณ์ทำให้มั่นใจได้ว่าเมื่อเจอสถานการณ์จริงในที่ทำงาน คุณจะมีสติและรู้ขั้นตอนที่ถูกต้องแม่นยำในการกู้คืน

ทักษะการ debug ที่ฝึกใน Part นี้คือสิ่งที่แยกนักพัฒนาที่ "ใช้ Git เป็น" ออกจากนักพัฒนาที่ "เข้าใจ Git จริง ๆ" — เพราะทุกคนใช้คำสั่งพื้นฐานได้ตอนทุกอย่างราบรื่น แต่มีเพียงคนที่เข้าใจกลไกเบื้องลึกเท่านั้นที่จะกู้สถานการณ์วิกฤตได้อย่างสงบและแม่นยำ

**ต่อไป:** [Part 95: โปรเจกต์ฝึกหัด: จำลองสถานการณ์วิกฤต (Incident) และการกู้คืน](./part-095-สถานการณ์วิกฤต-incident.md)
