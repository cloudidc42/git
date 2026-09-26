# Part 45: โปรเจกต์ทีม: จำลองการทำงานทีม 4-5 คนแบบมืออาชีพ

> **Step ในหลักสูตรนี้:** Step 441–450
> **เฟส:** 4 — ทำงานเป็นทีมด้วย Workflow มาตรฐาน (Part นี้คือ Part ปิดท้ายของเฟส 4 ทั้งหมด)
> **เป้าหมายของ Part นี้:** นำทุกสิ่งที่เรียนมาตลอดเฟส 4 ตั้งแต่ Part 31 ถึง Part 44 มาประกอบร่างเข้าด้วยกันในโปรเจกต์จำลองเดียว — Workflow models, Git Flow, GitHub Flow, Trunk-Based Development, Naming Convention, CODEOWNERS, Branch Protection, Rebase, Interactive Rebase, Cherry-pick, การแก้ Conflict ขั้นสูง, Bisect, Blame และ Submodules — โดยจำลองทีมพัฒนา 5 คนที่ทำงานพร้อมกันจริงบนโปรเจกต์เดียว ใช้หลายโฟลเดอร์ local แทนเครื่องของสมาชิกแต่ละคน push เข้า bare repository กลางเดียวกัน เพื่อให้เห็นกลไกที่แท้จริงเบื้องหลังการทำงานเป็นทีมแบบมืออาชีพ ก่อนจะข้ามไปเรียนรู้ GitLab ในเฟส 5

---

## สารบัญของ Part นี้

- Step 441: ภาพรวมภารกิจปิดเฟส 4 — จำลองทีม 5 คนทำงานจริงด้วย Workflow มาตรฐาน
- Step 442: ตั้งค่าโปรเจกต์ทีม — Branch Protection, CODEOWNERS, PR Template, Naming Convention
- Step 443: เลือกใช้ GitHub Flow เป็น Workflow หลักของทีม พร้อมเหตุผลประกอบ
- Step 444: จำลองสมาชิกทีมหลายคนทำงานพร้อมกันบนฟีเจอร์ต่างกัน
- Step 445: จำลองการทำ Code Review และขอ Approval จาก Code Owner ตาม CODEOWNERS
- Step 446: จำลอง Conflict ระหว่างสมาชิก 2 คนที่แก้ไฟล์เดียวกัน แก้ด้วยการ Rebase
- Step 447: จำลอง Hotfix ด่วนที่ต้อง Cherry-pick ไปยังหลาย Release Branch
- Step 448: ใช้ `git bisect` หาบั๊กที่แอบแฝงในโปรเจกต์ทีม
- Step 449: ใช้ `git blame` สืบสวนที่มาของโค้ดที่มีปัญหาก่อนแก้ไข
- Step 450: สรุปทบทวนภาพรวมเฟส 4 ทั้งหมด (Part 31–45) พร้อม Cheat Sheet และ Checklist ก่อนเข้าเฟส 5

---

## Step 441: ภาพรวมภารกิจปิดเฟส 4 — จำลองทีม 5 คนทำงานจริงด้วย Workflow มาตรฐาน

ตลอด 14 Part ที่ผ่านมาในเฟส 4 (Part 31–44) คุณได้เรียนรู้แนวคิดและเครื่องมือระดับทีมแยกเป็นชิ้น ๆ มาครบแล้ว: Workflow Models ต่าง ๆ, Git Flow, GitHub Flow, Trunk-Based Development, Branch Naming Convention, CODEOWNERS, Branch Protection Rules, `git rebase`, Interactive Rebase (`rebase -i`), `git cherry-pick`, การแก้ Conflict ในสถานการณ์ซับซ้อน, `git bisect`, `git blame` และ Submodules

Part นี้จะ **ไม่สอนคำสั่งใหม่แม้แต่คำสั่งเดียว** เช่นเดียวกับที่ Part 15 ปิดเฟส 2 และ Part 30 ปิดเฟส 3 — แต่จะเอาทุกอย่างที่เรียนมาทั้งหมดมาร้อยเรียงเข้าด้วยกันในสถานการณ์เดียวที่สมจริงที่สุดเท่าที่จะทำได้: **การจำลองทีมพัฒนาซอฟต์แวร์ 5 คนทำงานบนโปรเจกต์เดียวกันตลอดวงจรชีวิตของฟีเจอร์ ตั้งแต่วางกฎการทำงาน ไปจนถึงการไล่ล่าบั๊กที่แอบซ่อนอยู่ในประวัติ**

### ทำความรู้จักทีมจำลองของเรา

โปรเจกต์ที่เราจะสร้างชื่อ **TeamCommerce** — เว็บไซต์แคตตาล็อกสินค้าเล็ก ๆ ที่แบ่งงานเป็นฝั่ง frontend และ backend อย่างชัดเจน (ยังคงใช้ HTML/CSS/JavaScript ธรรมดาโดยไม่พึ่ง build tool ใด ๆ เพื่อโฟกัสที่กลไกของ Git 100% เหมือนโปรเจกต์ก่อนหน้านี้ในหลักสูตร)

| ชื่อ (สมมติ) | บทบาท | อีเมล | พื้นที่รับผิดชอบ |
|---|---|---|---|
| วีระ อำนวยกิจ | DevOps / Release Manager | weera@teamcommerce.dev | โครงสร้าง repo, CI config, release branch, ไฟล์ root |
| สมชาย ดีชัย | Backend Lead (Code Owner) | somchai@teamcommerce.dev | โฟลเดอร์ `backend/` |
| ปิติ รุ่งเรือง | Backend Developer | piti@teamcommerce.dev | โฟลเดอร์ `backend/` |
| มานี สุขสันต์ | Frontend Lead (Code Owner) | manee@teamcommerce.dev | โฟลเดอร์ `frontend/` |
| ชูใจ สุขใจ | Frontend Developer | chujai@teamcommerce.dev | โฟลเดอร์ `frontend/` |

ทีมนี้แบ่งเป็น 2 ทีมย่อยชัดเจน (**frontend team**: มานี + ชูใจ, **backend team**: สมชาย + ปิติ) โดยมีวีระเป็นผู้ดูแลภาพรวมของ repository ทั้งหมด — โครงสร้างแบบนี้คือโครงสร้างที่พบได้จริงในบริษัทซอฟต์แวร์ขนาดเล็กถึงกลางแทบทุกที่

### วิธีจำลองทีมบนเครื่องเดียว

ในโลกจริง ขั้นตอนทั้งหมดต่อไปนี้จะเกิดขึ้นบน GitHub จริงตามที่คุณเรียนไปแล้วในเฟส 3 (Part 16–30) พร้อม Branch Protection และ CODEOWNERS ของ GitHub เองตามที่เรียนใน Part 33–36 แต่เพื่อให้คุณฝึกได้โดยไม่ต้องพึ่งบัญชี GitHub หลายบัญชีพร้อมกัน และเพื่อให้เห็น **กลไกที่แท้จริงเบื้องหลัง** ของสิ่งที่ GitHub ทำให้อัตโนมัติ Part นี้จะจำลองทุกอย่างด้วย **bare repository กลางบนเครื่องเดียว** แทน "GitHub" และใช้ **หลายโฟลเดอร์ local** แทน "เครื่องของสมาชิกแต่ละคน" — กลไกระดับ Git (push, fetch, merge, rebase, cherry-pick) ที่คุณจะได้ฝึกในนี้คือกลไกเดียวกันเป๊ะ ๆ กับที่ GitHub ใช้อยู่เบื้องหลัง Pull Request ของมันทุกประการ

โครงสร้างโฟลเดอร์ที่เราจะสร้างมีดังนี้:

```
~/git-course/
├── teamcommerce-central.git/     ← bare repo กลาง (จำลอง "GitHub")
└── teamcommerce-team/
    ├── weera/                    ← เครื่องของวีระ
    ├── somchai/                  ← เครื่องของสมชาย
    ├── piti/                     ← เครื่องของปิติ
    ├── manee/                    ← เครื่องของมานี
    └── chujai/                   ← เครื่องของชูใจ
```

แต่ละโฟลเดอร์คือ **git clone แยกกันอิสระ** ของ bare repo กลาง โดยตั้งค่า `user.name`/`user.email` เฉพาะในแต่ละโฟลเดอร์ (local config ไม่ใช่ global) เพื่อให้ commit ที่เกิดขึ้นในแต่ละโฟลเดอร์มีชื่อผู้เขียนถูกต้องตามสมาชิกที่จำลองอยู่จริง

### เตรียมโครงสร้างโฟลเดอร์

```bash
cd ~/git-course
mkdir teamcommerce-central.git
cd teamcommerce-central.git
git init --bare
```

```
Initialized empty Git repository in /home/user/git-course/teamcommerce-central.git/
```

```bash
cd ~/git-course
mkdir -p teamcommerce-team/{weera,somchai,piti,manee,chujai}
```

### แผนงานทั้งหมดของ Part นี้ในสายตาเดียว

ก่อนลงมือ มาดูภาพรวมว่า Part นี้จะพาคุณผ่านสถานการณ์อะไรบ้าง โดยแต่ละแถวคือการนำทักษะจาก Part ก่อนหน้ามาใช้งานจริง:

| Step | เหตุการณ์ | ทักษะที่ใช้จาก Part ก่อนหน้า |
|---|---|---|
| 442 | ตั้งกฎของทีม | CODEOWNERS (Part 36), Branch Protection (Part 33-35), Naming Convention (Part 32) |
| 443 | เลือก Workflow หลัก | Git Flow / GitHub Flow / Trunk-Based (Part 31, 34, 35) |
| 444 | สองฟีเจอร์คู่ขนาน | Branch, Remote, Push (เฟส 2), Workflow ที่เลือก |
| 445 | Review + Approval | CODEOWNERS, Branch Protection, `git request-pull` |
| 446 | Conflict ระหว่างคน | Rebase, Interactive Rebase, การแก้ Conflict ขั้นสูง (Part 37-39) |
| 447 | Hotfix หลาย Release | Cherry-pick (Part 40) |
| 448 | ล่าบั๊กด้วย Bisect | `git bisect` (Part 41-42) |
| 449 | สืบที่มาด้วย Blame | `git blame` (Part 43) |
| 450 | สรุปเฟส 4 ทั้งหมด | ทุก Part 31-45 |

> **หมายเหตุ:** โปรเจกต์นี้จะไม่แตะเรื่อง Submodules โดยตรงในสถานการณ์จำลอง เพราะ Submodules เหมาะกับสถานการณ์ที่มีหลาย repository แยกกันจริง ๆ ซึ่งจะทำให้ Part นี้ซับซ้อนเกินจำเป็น แต่เราจะสรุปคำสั่งและแนวคิดของมันไว้ในตาราง Cheat Sheet ของ Step 450 อย่างครบถ้วน เพื่อทบทวนความรู้ทั้งหมดของเฟส 4

จากนี้ไป เราจะเดินหน้าไปทีละ Step ตามตารางด้านบน โดยสวมบทบาทสลับกันเป็นสมาชิกแต่ละคนของทีม TeamCommerce

---

## Step 442: ตั้งค่าโปรเจกต์ทีม — Branch Protection, CODEOWNERS, PR Template, Naming Convention

วีระในบทบาท DevOps / Release Manager เป็นคนแรกที่ต้องวางรากฐานของ repository ก่อนให้ทีมเริ่มทำงาน

### 442.1 Clone และสร้างโครงสร้างเริ่มต้น

```bash
cd ~/git-course/teamcommerce-team/weera
git clone ../../teamcommerce-central.git .
```

```
Cloning into '.'...
warning: You appear to have cloned an empty repository.
```

ตั้งค่า identity เฉพาะโฟลเดอร์นี้:

```bash
git config user.name "Weera Amnuaykit"
git config user.email "weera@teamcommerce.dev"
```

สร้างโครงสร้างไฟล์เริ่มต้น:

```bash
mkdir -p frontend/css frontend/js backend/routes docs .github

cat > README.md << 'EOF'
# TeamCommerce

เว็บไซต์แคตตาล็อกสินค้าจำลอง สร้างขึ้นเพื่อฝึกการทำงานเป็นทีมด้วย Git
ในหลักสูตร Git 1000 Steps — Part 45 (Capstone เฟส 4)

## โครงสร้างโปรเจกต์

- `frontend/` — ดูแลโดยทีม Frontend (มานี, ชูใจ)
- `backend/` — ดูแลโดยทีม Backend (สมชาย, ปิติ)
- `docs/` — เอกสารประกอบโปรเจกต์
EOF

cat > frontend/index.html << 'EOF'
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>TeamCommerce</title>
  <link rel="stylesheet" href="css/style.css">
</head>
<body>
  <header class="site-header">
    <h1>TeamCommerce</h1>
  </header>
  <main id="product-list"></main>
  <script src="js/app.js"></script>
</body>
</html>
EOF

cat > frontend/css/style.css << 'EOF'
/* style.css: สไตล์หลักของ TeamCommerce */
* { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: sans-serif; color: #222; }
EOF

cat > frontend/js/app.js << 'EOF'
// app.js: จุดเริ่มต้นของ frontend
function renderProducts(products) {
  console.log("rendering", products.length, "products");
}
EOF

cat > backend/server.js << 'EOF'
// server.js: จุดเริ่มต้นของ backend (จำลอง ไม่ได้รันจริง)
const routes = require("./routes");
console.log("TeamCommerce backend starting...");
EOF
```

### 442.2 Commit แรกและ push ขึ้น bare repo กลาง

```bash
git add .
git commit -m "Initial commit: โครงสร้างเริ่มต้นของโปรเจกต์ TeamCommerce"
git branch -M main
git push origin main
```

```
[main (root-commit) a1b2c30] Initial commit: โครงสร้างเริ่มต้นของโปรเจกต์ TeamCommerce
 6 files changed, 28 insertions(+)
...
To ../../teamcommerce-central.git
 * [new branch]      main -> main
```

### 442.3 Naming Convention — ตกลงกฎการตั้งชื่อ branch ของทีม

ตามหลักที่เรียนใน Part 32 ทีมต้องตกลงรูปแบบชื่อ branch ให้ชัดเจนตั้งแต่วันแรก เพื่อไม่ให้เกิดชื่อ branch สะเปะสะปะเมื่อทีมโตขึ้น:

```bash
cat > docs/BRANCH_NAMING.md << 'EOF'
# กฎการตั้งชื่อ Branch ของทีม TeamCommerce

| ประเภท | รูปแบบ | ตัวอย่าง |
|---|---|---|
| ฟีเจอร์ใหม่ | `feature/<ทีม>-<คำอธิบายสั้น>` | `feature/frontend-product-list` |
| แก้บั๊กด่วน | `hotfix/<คำอธิบายสั้น>` | `hotfix/cart-discount-bug` |
| เตรียม release | `release/<เวอร์ชัน>` | `release/1.0` |

## กฎเพิ่มเติม

- ใช้ตัวพิมพ์เล็กและ `-` คั่นคำเสมอ ห้ามใช้ `_` หรือช่องว่าง
- ชื่อ branch ต้องสื่อความหมายว่าทำอะไร ห้ามตั้งชื่อคลุมเครือ เช่น `fix`, `test`, `wip`
- branch ที่ไม่ตรงรูปแบบนี้จะถูก **ปฏิเสธโดยอัตโนมัติ** ตอน push (ดู pre-receive hook ด้านล่าง)
EOF
```

### 442.4 CODEOWNERS — กำหนดเจ้าของโค้ดตามทีม

ตามที่เรียนละเอียดใน Part 36 เราจะสร้างไฟล์ CODEOWNERS ที่แบ่งความเป็นเจ้าของตามโครงสร้างโฟลเดอร์:

```bash
cat > .github/CODEOWNERS << 'EOF'
# CODEOWNERS ของ TeamCommerce
# รูปแบบ: <pattern>  <เจ้าของ>

# ทีม Frontend ดูแลทุกอย่างใน frontend/
/frontend/   manee chujai

# ทีม Backend ดูแลทุกอย่างใน backend/
/backend/    somchai piti

# วีระดูแลไฟล์ตั้งค่าระดับ root และเอกสาร
/docs/       weera
/.github/    weera
*            weera
EOF
```

> **จุดสำคัญที่ต้องจำ:** บน GitHub จริง การเขียน `manee chujai` แบบนี้จะอ้างอิงถึง GitHub username หรือ team handle (เช่น `@manee` หรือ `@teamcommerce/frontend-team`) ตามที่เรียนใน Part 36 — ในสถานการณ์จำลองนี้เราใช้ชื่อธรรมดาแทน เพราะไม่มีบัญชี GitHub จริงให้ผูก แต่หลักการ pattern matching และลำดับความสำคัญ (order of precedence) เหมือนกันทุกประการ

### 442.5 Pull Request Template

แม้ Part นี้จะไม่ได้เปิด PR ผ่านหน้าเว็บ GitHub จริง แต่ทีมยังคงต้องมีมาตรฐานว่า "คำอธิบายการเปลี่ยนแปลง" ควรมีหัวข้ออะไรบ้าง เราจะเก็บไฟล์นี้ไว้เป็นแนวทางที่สมาชิกทุกคนต้องเขียนตามเมื่อขอ review (ในโลกจริงไฟล์นี้จะถูก GitHub ดึงมาแสดงเป็น template อัตโนมัติในหน้าเปิด PR):

```bash
cat > .github/pull_request_template.md << 'EOF'
## คำอธิบายการเปลี่ยนแปลง

<!-- อธิบายว่าเปลี่ยนอะไร ทำไมถึงเปลี่ยน -->

## ประเภทของการเปลี่ยนแปลง

- [ ] ฟีเจอร์ใหม่
- [ ] แก้บั๊ก
- [ ] Hotfix ด่วน
- [ ] ปรับปรุงเอกสาร

## Checklist ก่อนขอ Review

- [ ] ทดสอบด้วยตัวเองแล้วว่าทำงานถูกต้อง
- [ ] ไม่มี console.log หรือโค้ดทดสอบตกค้าง
- [ ] อัปเดตเอกสารที่เกี่ยวข้อง (ถ้ามี)
- [ ] Rebase กับ main ล่าสุดแล้วก่อนขอ review

## Reviewer ที่เกี่ยวข้อง (ตาม CODEOWNERS)
EOF
```

### 442.6 Branch Protection — จำลองด้วย pre-receive hook บน bare repo

บน GitHub จริง เราจะไปตั้งค่าที่ **Settings → Branches → Branch protection rules** ตามที่เรียนใน Part 33–35 เพื่อบังคับว่า:

1. ห้าม push ตรงเข้า `main` โดยไม่ผ่าน Pull Request
2. ต้องมีการ approve จาก Code Owner ก่อน merge เข้า `main` เสมอ
3. ชื่อ branch ต้องตรงตาม naming convention

เนื่องจากเราไม่มี GitHub server จริง เราจะใช้กลไกที่ Git มีให้ในตัวคือ **server-side hook** ชนิด `pre-receive` ที่ทำงานอยู่บน bare repository — hook นี้จะรันทุกครั้งที่มีใคร push เข้ามา และสามารถ **ปฏิเสธการ push** ได้ถ้าไม่ผ่านเงื่อนไขที่กำหนด ซึ่งเป็นกลไกแบบเดียวกันเป๊ะ ๆ กับที่ GitHub ใช้อยู่เบื้องหลัง Branch Protection ของมัน

```bash
cd ~/git-course/teamcommerce-central.git/hooks

cat > pre-receive << 'EOF'
#!/bin/sh
# pre-receive hook จำลอง Branch Protection + CODEOWNERS review requirement
zero="0000000000000000000000000000000000000000"

while read oldrev newrev refname; do
  # ตรวจสอบเฉพาะ branch (refs/heads/*) เท่านั้น — ปล่อยผ่าน tag (refs/tags/*)
  # หรือ ref ประเภทอื่นไปเลย ไม่งั้นกฎ naming convention ของ branch จะไปบล็อกการ push tag ด้วย
  case "$refname" in
    refs/heads/*) : ;;
    *) continue ;;
  esac

  branch=$(echo "$refname" | sed 's#refs/heads/##')

  # ข้ามการลบ branch (newrev เป็นค่า zero)
  if [ "$newrev" = "$zero" ]; then
    continue
  fi

  # กฎที่ 1: บังคับ naming convention เฉพาะตอนสร้าง branch ใหม่เท่านั้น
  if [ "$oldrev" = "$zero" ]; then
    case "$branch" in
      main|feature/*|hotfix/*|release/*) : ;;
      *)
        echo "REJECTED: ชื่อ branch '$branch' ไม่ตรง naming convention"
        echo "          ต้องขึ้นต้นด้วย feature/, hotfix/ หรือ release/ เท่านั้น"
        exit 1
        ;;
    esac
  fi

  # กฎที่ 2: main ต้องมาจาก merge commit ที่มี Reviewed-by เท่านั้น
  if [ "$branch" = "main" ]; then
    parent_count=$(git cat-file -p "$newrev" | grep -c '^parent ')
    if [ "$parent_count" -lt 2 ]; then
      echo "REJECTED: ห้าม push commit เดี่ยวเข้า main โดยตรง"
      echo "          ต้อง merge จาก feature/hotfix branch ที่ผ่านการรีวิวเท่านั้น"
      exit 1
    fi

    message=$(git log -1 --format=%B "$newrev")
    case "$message" in
      *"Reviewed-by:"*) : ;;
      *)
        echo "REJECTED: merge commit เข้า main ต้องมี trailer 'Reviewed-by:' จาก Code Owner"
        exit 1
        ;;
    esac
  fi
done

exit 0
EOF

chmod +x pre-receive
```

> **อธิบายการทำงานของ hook นี้:** Git จะส่งข้อมูล 3 ค่าต่อบรรทัดเข้าทาง stdin สำหรับทุก ref ที่ถูก push มา (`oldrev newrev refname`) โดย hook จะ**กรองเอาเฉพาะ ref ที่เป็น branch (`refs/heads/*`)** มาตรวจสอบเท่านั้น — ปล่อยผ่าน ref ประเภทอื่นอย่าง tag (`refs/tags/*`) ไปเลยด้วย `continue` เพื่อไม่ให้กฎการตั้งชื่อ branch ไปบล็อกการ push tag อย่าง `v1.0.0` โดยไม่ตั้งใจ จากนั้น hook ตรวจสอบสองกฎกับ branch ที่เหลือ: (1) ถ้าเป็นการสร้าง branch ใหม่ (`oldrev` เป็นค่า zero ทั้งหมด) ชื่อ branch ต้องขึ้นต้นด้วย `feature/`, `hotfix/`, `release/` หรือเป็น `main` เท่านั้น (2) ถ้า ref ที่ถูก push คือ `main` commit ปลายทางต้องเป็น **merge commit** (มีมากกว่า 1 parent เสมอ ตรวจด้วย `git cat-file -p` นับจำนวนบรรทัด `parent`) และข้อความ commit ต้องมี `Reviewed-by:` อยู่ด้วย ถ้าเงื่อนไขไหนไม่ผ่าน hook จะ `exit 1` ทำให้ Git ปฏิเสธการ push ทั้งหมดทันที

เราจะเห็น hook นี้ทำงานจริงในสถานการณ์ Step 445

### 442.7 Push การตั้งค่าทั้งหมดขึ้น central

```bash
cd ~/git-course/teamcommerce-team/weera
git add .
git commit -m "เพิ่ม CODEOWNERS, PR template, และเอกสาร branch naming convention"
git push origin main
```

```
[main b2c3d41] เพิ่ม CODEOWNERS, PR template, และเอกสาร branch naming convention
 3 files changed, 41 insertions(+)
...
To ../../teamcommerce-central.git
   a1b2c30..b2c3d41  main -> main
```

> **ข้อควรระวัง:** สังเกตว่า commit นี้ผ่านได้เพราะ hook เราตรวจ `main` เฉพาะกรณีที่ `newrev` เป็น merge commit หรือไม่ — commit ตรงนี้ยังเป็น commit เดี่ยวที่วีระ push เองในฐานะคนตั้งค่า repo ครั้งแรกก่อนกฎจะเริ่มบังคับใช้จริงจัง เมื่อไฟล์ hook ถูกติดตั้งแล้ว **การ push commit เดี่ยวเข้า main ครั้งต่อไปจากนี้จะถูกปฏิเสธทันที** — ทีมทั้งหมดต้องปฏิบัติตามกฎเดียวกันนับจากนี้ ไม่มีข้อยกเว้น (นี่คือหลักการสำคัญของ Branch Protection: กฎต้องบังคับใช้กับทุกคนอย่างเท่าเทียม รวมถึงผู้ดูแล repo เอง)

ตอนนี้ `main` ของทีมอยู่ที่ commit `b2c3d41` พร้อมกฎทั้งหมดวางเรียบร้อยแล้ว — ทีมพร้อมเริ่มทำงานจริง

---

## Step 443: เลือกใช้ GitHub Flow เป็น Workflow หลักของทีม พร้อมเหตุผลประกอบ

ก่อนใครจะเริ่มเขียนโค้ด ทีมต้องตกลงกันก่อนว่าจะใช้ Workflow model แบบไหน ตามที่เรียนมาในเฟส 4 มีตัวเลือกหลักอยู่ 3 แบบ:

| Workflow | จุดเด่น | เหมาะกับ |
|---|---|---|
| **Git Flow** | มี branch แยกละเอียด (`develop`, `release/*`, `hotfix/*`) รองรับหลาย version พร้อมกัน | ซอฟต์แวร์ที่ปล่อยเวอร์ชันเป็นรอบ (เช่น mobile app, software แบบติดตั้ง) |
| **GitHub Flow** | มีแค่ `main` และ `feature/*` เรียบง่าย deploy บ่อยจาก `main` โดยตรง | เว็บแอปที่ deploy ต่อเนื่อง (continuous deployment) |
| **Trunk-Based Development** | ทุกคน commit เข้า `main`/`trunk` บ่อย ๆ ใช้ feature flag แทน branch ยาว | ทีมใหญ่มากที่ต้องการ integration ถี่ที่สุด |

### เหตุผลที่ทีม TeamCommerce เลือก GitHub Flow

หลังจากพิจารณาตามหลักที่เรียนใน Part 31, 34 และ 35 ทีมตัดสินใจเลือก **GitHub Flow** เป็น Workflow หลัก ด้วยเหตุผลดังนี้:

1. **ทีมมีขนาดเล็ก (5 คน)** — Git Flow ที่มี branch หลายชั้น (`develop`, `feature`, `release`, `hotfix`) จะสร้างความซับซ้อนเกินความจำเป็นสำหรับทีมขนาดนี้ ในขณะที่ Trunk-Based ต้องการวินัยสูงมากและ feature flag infrastructure ที่ทีมยังไม่มี
2. **TeamCommerce เป็นเว็บแอปที่ deploy ได้บ่อย** — ไม่ใช่ซอฟต์แวร์ที่ต้องดูแลหลายเวอร์ชันพร้อมกันตลอดเวลาแบบที่ Git Flow ถูกออกแบบมารองรับ (แม้ Step 447 เราจะยังคงมี release branch อยู่บ้างเพื่อรองรับลูกค้าที่ค้างอยู่กับเวอร์ชันเก่า แต่นั่นเป็นข้อยกเว้นเฉพาะกิจ ไม่ใช่ Workflow หลัก)
3. **`main` ต้อง deploy ได้เสมอ** — หลักการของ GitHub Flow คือ `main` ต้องอยู่ในสถานะที่ deploy ได้ตลอดเวลา ทุกฟีเจอร์แยกเป็น `feature/*` branch สั้น ๆ แล้ว merge กลับเข้า `main` ทันทีที่ผ่านการรีวิว
4. **เข้ากับ Branch Protection + CODEOWNERS ที่เพิ่งตั้งค่าไปใน Step 442 ได้พอดี** — GitHub Flow ต้องการแค่กฎเดียวที่ชัดเจน: ห้ามใครแตะ `main` ตรง ๆ ทุกอย่างต้องผ่านการรีวิวก่อนเสมอ ซึ่งตรงกับ hook ที่เราเพิ่งสร้างไปเป๊ะ

บันทึกการตัดสินใจนี้ไว้เป็นเอกสารเพื่อให้สมาชิกใหม่ในอนาคตเข้าใจบริบท (แนวทางปฏิบัติที่ดีขององค์กรวิศวกรรมทุกที่ — บันทึกเหตุผลของการตัดสินใจสำคัญไว้เป็นลายลักษณ์อักษรเสมอ):

```bash
cd ~/git-course/teamcommerce-team/weera
git pull origin main

cat > docs/WORKFLOW.md << 'EOF'
# Workflow ของทีม TeamCommerce

ทีมเลือกใช้ **GitHub Flow** เป็น Workflow หลัก

## กฎการทำงาน

1. `main` ต้องอยู่ในสถานะ deploy ได้เสมอ ห้าม push commit เดี่ยวเข้า `main` โดยตรง
2. ทุกฟีเจอร์แยกเป็น branch `feature/<team>-<description>` จาก `main` ล่าสุด
3. ก่อนขอ review ต้อง rebase กับ `main` ล่าสุดเสมอ (ดู Part 37-39 ของหลักสูตร)
4. ต้องได้รับ approve จาก Code Owner ตามไฟล์ `.github/CODEOWNERS` ก่อน merge ทุกครั้ง
5. Merge เข้า `main` ด้วย `--no-ff` เสมอ เพื่อให้เห็นขอบเขตของแต่ละฟีเจอร์ชัดเจนในประวัติ
6. หลัง merge แล้วให้ลบ feature branch ทิ้งทันที

## ข้อยกเว้น: Release Branch

แม้จะใช้ GitHub Flow เป็นหลัก แต่ทีมยังคงต้องดูแลลูกค้าที่ใช้เวอร์ชันเก่าอยู่บางส่วน
จึงมีการตัด `release/<เวอร์ชัน>` branch เป็นครั้งคราวเพื่อรองรับ hotfix เฉพาะกิจ
(ดูรายละเอียดใน Part 45 Step 447 ของหลักสูตร)
EOF

git add docs/WORKFLOW.md
git commit -m "เพิ่มเอกสารอธิบายการเลือกใช้ GitHub Flow เป็น Workflow หลักของทีม"
git push origin main
```

```
[main c3d4e52] เพิ่มเอกสารอธิบายการเลือกใช้ GitHub Flow เป็น Workflow หลักของทีม
 1 file changed, 21 insertions(+)
...
   b2c3d41..c3d4e52  main -> main
```

`main` อยู่ที่ `c3d4e52` แล้ว — ตอนนี้ทีมพร้อมเริ่มพัฒนาฟีเจอร์จริงตาม Workflow ที่ตกลงกันไว้

---

## Step 444: จำลองสมาชิกทีมหลายคนทำงานพร้อมกันบนฟีเจอร์ต่างกัน

ถึงเวลาที่มานี (frontend) และปิติ (backend) จะเริ่มทำงานพร้อมกันจริง ๆ บนคนละฟีเจอร์ — นี่คือสถานการณ์ที่เกิดขึ้นทุกวันในทีมพัฒนาจริง

### 444.1 มานีเริ่มฟีเจอร์หน้ารายการสินค้า

```bash
cd ~/git-course/teamcommerce-team/manee
git clone ../../teamcommerce-central.git .
git config user.name "Manee Suksan"
git config user.email "manee@teamcommerce.dev"

git switch -c feature/frontend-product-list
```

```
Switched to a new branch 'feature/frontend-product-list'
```

Commit ที่ 1 — โครง HTML:

```bash
cat > frontend/index.html << 'EOF'
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>TeamCommerce</title>
  <link rel="stylesheet" href="css/style.css">
</head>
<body>
  <header class="site-header">
    <h1>TeamCommerce</h1>
  </header>
  <main id="product-list" class="product-grid"></main>
  <script src="js/app.js"></script>
</body>
</html>
EOF

git add frontend/index.html
git commit -m "เพิ่มโครง HTML สำหรับหน้ารายการสินค้า"
```

```
[feature/frontend-product-list d4e5f63] เพิ่มโครง HTML สำหรับหน้ารายการสินค้า
 1 file changed, 1 insertion(+)
```

Commit ที่ 2 — CSS การ์ดสินค้า:

```bash
cat >> frontend/css/style.css << 'EOF'

.product-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
  padding: 2rem;
}

.product-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 1rem;
}
EOF

git add frontend/css/style.css
git commit -m "เพิ่ม CSS จัดสไตล์การ์ดสินค้าแบบ Grid"
```

```
[feature/frontend-product-list e5f6a74] เพิ่ม CSS จัดสไตล์การ์ดสินค้าแบบ Grid
 1 file changed, 12 insertions(+)
```

Commit ที่ 3 — JavaScript render สินค้า:

```bash
cat > frontend/js/app.js << 'EOF'
// app.js: render รายการสินค้าลงในหน้าเว็บ
function renderProducts(products) {
  if (!Array.isArray(products) || products.length === 0) return;
  const container = document.getElementById("product-list");
  container.innerHTML = products
    .map((p) => `<div class="product-card">${p.name} - ${p.price} บาท</div>`)
    .join("");
}
EOF

git add frontend/js/app.js
git commit -m "เพิ่มฟังก์ชัน renderProducts พร้อม guard clause ตรวจสอบข้อมูล"
```

```
[feature/frontend-product-list f6a7b85] เพิ่มฟังก์ชัน renderProducts พร้อม guard clause ตรวจสอบข้อมูล
 1 file changed, 6 insertions(+), 3 deletions(-)
```

Push ขึ้น central (สังเกตว่า push ไปที่ `feature/frontend-product-list` ไม่ใช่ `main` จึงไม่โดน hook ตรวจกฎข้อ 2):

```bash
git push -u origin feature/frontend-product-list
```

```
...
To ../../teamcommerce-central.git
 * [new branch]      feature/frontend-product-list -> feature/frontend-product-list
```

### 444.2 ปิติเริ่มฟีเจอร์ backend cart API พร้อมกัน

ในเวลาเดียวกัน ปิติก็เริ่มงานของตัวเองจากเครื่องอีกโฟลเดอร์หนึ่ง โดยแตก branch จาก `main` ที่จุดเดียวกัน (`c3d4e52`) — เขาไม่รู้เลยว่ามานีกำลังทำอะไรอยู่ และไม่จำเป็นต้องรู้ด้วย เพราะทำงานคนละไฟล์กันโดยสิ้นเชิง:

```bash
cd ~/git-course/teamcommerce-team/piti
git clone ../../teamcommerce-central.git .
git config user.name "Piti Rungrueang"
git config user.email "piti@teamcommerce.dev"

git switch -c feature/backend-cart-api
```

```
Switched to a new branch 'feature/backend-cart-api'
```

Commit ที่ 1:

```bash
mkdir -p backend/routes
cat > backend/routes/cart.js << 'EOF'
// cart.js: จัดการตะกร้าสินค้า (โครงเบื้องต้น)
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price * item.qty, 0);
}

module.exports = { calculateTotal };
EOF

git add backend/routes/cart.js
git commit -m "เพิ่มโครง cart API พร้อมฟังก์ชันคำนวณยอดรวมเบื้องต้น"
```

```
[feature/backend-cart-api 1a2b3c4] เพิ่มโครง cart API พร้อมฟังก์ชันคำนวณยอดรวมเบื้องต้น
 1 file changed, 6 insertions(+)
```

Commit ที่ 2 — เพิ่มระบบส่วนลด:

```bash
cat > backend/routes/cart.js << 'EOF'
// cart.js: จัดการตะกร้าสินค้า
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price * item.qty, 0);
}

function applyDiscount(total, discountPercent) {
  return total - (total * discountPercent) / 100;
}

module.exports = { calculateTotal, applyDiscount };
EOF

git add backend/routes/cart.js
git commit -m "เพิ่มฟังก์ชัน applyDiscount สำหรับคำนวณส่วนลด"
```

```
[feature/backend-cart-api 2b3c4d5] เพิ่มฟังก์ชัน applyDiscount สำหรับคำนวณส่วนลด
 1 file changed, 4 insertions(+)
```

Push ขึ้น central:

```bash
git push -u origin feature/backend-cart-api
```

```
...
To ../../teamcommerce-central.git
 * [new branch]      feature/backend-cart-api -> feature/backend-cart-api
```

### 444.3 ดูภาพรวมทั้งสองฟีเจอร์บน central repo

จากเครื่องของวีระ ลอง fetch แล้วดูกราฟทั้งหมด:

```bash
cd ~/git-course/teamcommerce-team/weera
git fetch origin
git log --oneline --graph --all
```

```
* 2b3c4d5 (origin/feature/backend-cart-api) เพิ่มฟังก์ชัน applyDiscount สำหรับคำนวณส่วนลด
* 1a2b3c4 เพิ่มโครง cart API พร้อมฟังก์ชันคำนวณยอดรวมเบื้องต้น
| * f6a7b85 (origin/feature/frontend-product-list) เพิ่มฟังก์ชัน renderProducts พร้อม guard clause ตรวจสอบข้อมูล
| * e5f6a74 เพิ่ม CSS จัดสไตล์การ์ดสินค้าแบบ Grid
| * d4e5f63 เพิ่มโครง HTML สำหรับหน้ารายการสินค้า
|/
* c3d4e52 (origin/main, main) เพิ่มเอกสารอธิบายการเลือกใช้ GitHub Flow เป็น Workflow หลักของทีม
```

นี่คือภาพที่ตรงกับสิ่งที่เรียนใน Part 15 ทุกประการ เพียงแต่คราวนี้ทั้งสอง branch มาจาก **คนละคนบนคนละเครื่อง** ที่ push เข้ามารวมกันที่ bare repo กลาง — นี่คือแก่นแท้ของการทำงานเป็นทีมแบบ Distributed Version Control

---

## Step 445: จำลองการทำ Code Review และขอ Approval จาก Code Owner ตาม CODEOWNERS

ทั้งสองฟีเจอร์เสร็จพร้อมกันแล้ว ถึงเวลาขอ review ตามกฎที่ CODEOWNERS กำหนดไว้ใน Step 442

### 445.1 มานีขอ review ด้วย git request-pull

ก่อนยุค GitHub มี Pull Request UI นักพัฒนาที่ใช้ Git แบบกระจายศูนย์เต็มรูปแบบ (เช่นทีม Linux Kernel) ใช้คำสั่ง `git request-pull` เพื่อสร้างข้อความสรุปการเปลี่ยนแปลงสำหรับส่งให้คนอื่นรีวิว — คำสั่งนี้ยังมีอยู่ใน Git จนถึงปัจจุบัน และเราจะใช้มันจำลองขั้นตอน "เปิด PR" ในสภาพแวดล้อมที่ไม่มี GitHub UI จริง:

```bash
cd ~/git-course/teamcommerce-team/manee
git request-pull c3d4e52 ../../teamcommerce-central.git feature/frontend-product-list
```

```
The following changes since commit c3d4e52:

  เพิ่มเอกสารอธิบายการเลือกใช้ GitHub Flow เป็น Workflow หลักของทีม (2026-08-01 10:00:00 +0700)

are available in the Git repository at:

  ../../teamcommerce-central.git feature/frontend-product-list

for you to fetch changes up to f6a7b85:

  เพิ่มฟังก์ชัน renderProducts พร้อม guard clause ตรวจสอบข้อมูล (2026-08-01 10:20:00 +0700)

----------------------------------------------------------------
Manee Suksan (3):
      เพิ่มโครง HTML สำหรับหน้ารายการสินค้า
      เพิ่ม CSS จัดสไตล์การ์ดสินค้าแบบ Grid
      เพิ่มฟังก์ชัน renderProducts พร้อม guard clause ตรวจสอบข้อมูล

 frontend/css/style.css | 12 ++++++++++++
 frontend/index.html    |  1 +
 frontend/js/app.js     |  6 +++---
 3 files changed, 13 insertions(+), 3 deletions(-)
```

ข้อความนี้คือ "คำขอ PR" ที่มานีจะส่งให้ทีม — ตาม CODEOWNERS แล้วโฟลเดอร์ `frontend/` เป็นของ `manee chujai` ดังนั้นผู้ที่ต้อง review คือ **ชูใจ** (มานีรีวิวงานตัวเองไม่ได้)

### 445.2 ชูใจตรวจสอบและ approve

```bash
cd ~/git-course/teamcommerce-team/chujai
git clone ../../teamcommerce-central.git .
git config user.name "Chujai Sukjai"
git config user.email "chujai@teamcommerce.dev"

git fetch origin
git diff main origin/feature/frontend-product-list
```

ชูใจอ่าน diff ทั้งหมด ตรวจดูว่า guard clause ใน `renderProducts` ครบถ้วน ไม่มี console.log ตกค้าง ตรงตาม PR template checklist ที่ตั้งไว้ใน Step 442 — ผ่านทุกข้อ

### 445.3 พยายาม merge โดยไม่มี Reviewed-by (hook ปฏิเสธ)

ก่อนอื่น ลองดูว่าถ้าลืมใส่ trailer `Reviewed-by:` จะเกิดอะไรขึ้น (เพื่อพิสูจน์ว่า hook ทำงานจริง):

```bash
git switch main
git merge --no-ff origin/feature/frontend-product-list -m "Merge branch 'feature/frontend-product-list' into main"
git push origin main
```

```
...
remote: REJECTED: merge commit เข้า main ต้องมี trailer 'Reviewed-by:' จาก Code Owner
To ../../teamcommerce-central.git
 ! [remote rejected] main -> main (pre-receive hook declined)
error: failed to push some refs to '../../teamcommerce-central.git'
```

hook ทำงานตามที่ออกแบบไว้พอดี — การ push ถูกปฏิเสธ นี่คือ Branch Protection ในทางปฏิบัติจริง ไม่ใช่แค่ทฤษฎี

### 445.4 แก้ merge commit ให้มี Reviewed-by trailer แล้ว push ใหม่

```bash
git commit --amend -m "Merge branch 'feature/frontend-product-list' into main

Reviewed-by: Chujai Sukjai <chujai@teamcommerce.dev>"

git push origin main
```

```
[main 3c4d5e6] Merge branch 'feature/frontend-product-list' into main
...
To ../../teamcommerce-central.git
   c3d4e52..3c4d5e6  main -> main
```

ครั้งนี้ push สำเร็จ เพราะ merge commit มีทั้งสอง parent (มาจาก merge จริง) และมี trailer `Reviewed-by:` ครบถ้วนตามกฎ

ลบ feature branch ที่ merge เสร็จแล้วทั้งบน local และ remote ตามกฎ Workflow ที่ตกลงกันไว้ใน Step 443:

```bash
git branch -d feature/frontend-product-list
git push origin --delete feature/frontend-product-list
```

### 445.5 สมชายรีวิวและ merge ฟีเจอร์ backend ของปิติในลักษณะเดียวกัน

```bash
cd ~/git-course/teamcommerce-team/somchai
git clone ../../teamcommerce-central.git .
git config user.name "Somchai Deechai"
git config user.email "somchai@teamcommerce.dev"

git fetch origin
git diff main origin/feature/backend-cart-api
```

สมชายตรวจสอบฟังก์ชัน `calculateTotal` และ `applyDiscount` อย่างละเอียด (เขาคือ Code Owner ของ `backend/` ตาม CODEOWNERS) ผ่านการตรวจสอบเรียบร้อย:

```bash
git switch main
git merge --no-ff origin/feature/backend-cart-api -m "Merge branch 'feature/backend-cart-api' into main

Reviewed-by: Somchai Deechai <somchai@teamcommerce.dev>"

git push origin main
```

```
[main 4d5e6f7] Merge branch 'feature/backend-cart-api' into main
...
   3c4d5e6..4d5e6f7  main -> main
```

```bash
git push origin --delete feature/backend-cart-api
```

### 445.6 ตรวจสอบผลลัพธ์รวม

```bash
git log --oneline --graph
```

```
*   4d5e6f7 (HEAD -> main, origin/main) Merge branch 'feature/backend-cart-api' into main
|\
| * 2b3c4d5 เพิ่มฟังก์ชัน applyDiscount สำหรับคำนวณส่วนลด
| * 1a2b3c4 เพิ่มโครง cart API พร้อมฟังก์ชันคำนวณยอดรวมเบื้องต้น
* |   3c4d5e6 Merge branch 'feature/frontend-product-list' into main
|\ \
| |/
|/|
| * f6a7b85 เพิ่มฟังก์ชัน renderProducts พร้อม guard clause ตรวจสอบข้อมูล
| * e5f6a74 เพิ่ม CSS จัดสไตล์การ์ดสินค้าแบบ Grid
| * d4e5f63 เพิ่มโครง HTML สำหรับหน้ารายการสินค้า
|/
* c3d4e52 เพิ่มเอกสารอธิบายการเลือกใช้ GitHub Flow เป็น Workflow หลักของทีม
```

ทั้งสองฟีเจอร์ถูกรวมเข้า `main` เรียบร้อยแล้ว โดยทุก merge commit มีทั้งการรีวิวจริงและ trailer ยืนยันครบถ้วน — วีระตัด tag ปิด milestone แรกของโปรเจกต์และตัด release branch สำหรับลูกค้ากลุ่มแรก (จะใช้ใน Step 447):

```bash
cd ~/git-course/teamcommerce-team/weera
git pull origin main
git tag -a v1.0.0 -m "TeamCommerce v1.0.0: หน้ารายการสินค้า + cart API เบื้องต้น"
git push origin v1.0.0
git branch release/1.0 v1.0.0
git push origin release/1.0
```

---

## Step 446: จำลอง Conflict ระหว่างสมาชิก 2 คนที่แก้ไฟล์เดียวกัน แก้ด้วยการ Rebase

สถานการณ์ที่หลีกเลี่ยงไม่ได้ในทีมจริงคือสองคนแก้ไฟล์เดียวกันในบริเวณเดียวกันโดยไม่รู้ตัว รอบนี้จะเป็นชูใจกับมานีที่ต่างคนต่างเพิ่มองค์ประกอบใหม่ใน header ของ `frontend/index.html` พร้อมกัน

### 446.1 ชูใจเริ่มฟีเจอร์ช่องค้นหาสินค้า

```bash
cd ~/git-course/teamcommerce-team/chujai
git switch main
git pull origin main
git switch -c feature/frontend-search-bar
```

```bash
cat > frontend/index.html << 'EOF'
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>TeamCommerce</title>
  <link rel="stylesheet" href="css/style.css">
</head>
<body>
  <header class="site-header">
    <h1>TeamCommerce</h1>
    <input type="search" id="search-box" placeholder="ค้นหาสินค้า...">
  </header>
  <main id="product-list" class="product-grid"></main>
  <script src="js/app.js"></script>
</body>
</html>
EOF

git add frontend/index.html
git commit -m "เพิ่มช่องค้นหาสินค้าใน header"
git push -u origin feature/frontend-search-bar
```

```
[feature/frontend-search-bar 5e6f7a8] เพิ่มช่องค้นหาสินค้าใน header
 1 file changed, 1 insertion(+)
```

### 446.2 ในเวลาเดียวกัน มานีเริ่มฟีเจอร์ไอคอนตะกร้าสินค้า

มานีแตก branch จาก `main` ที่จุดเดียวกัน (`4d5e6f7`) **ก่อน** ที่ชูใจจะ merge งานของตัวเองเข้าไป — เธอไม่รู้เลยว่าชูใจกำลังแก้ไฟล์เดียวกันอยู่:

```bash
cd ~/git-course/teamcommerce-team/manee
git switch main
git pull origin main
git switch -c feature/frontend-cart-badge
```

```bash
cat > frontend/index.html << 'EOF'
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>TeamCommerce</title>
  <link rel="stylesheet" href="css/style.css">
</head>
<body>
  <header class="site-header">
    <h1>TeamCommerce</h1>
    <span id="cart-badge" class="cart-badge">0</span>
  </header>
  <main id="product-list" class="product-grid"></main>
  <script src="js/app.js"></script>
</body>
</html>
EOF

cat >> frontend/js/app.js << 'EOF'

function updateCartBadge(count) {
  document.getElementById("cart-badge").textContent = String(count);
}
EOF

git add frontend/index.html frontend/js/app.js
git commit -m "เพิ่มไอคอนตะกร้าสินค้าพร้อมตัวเลขจำนวนใน header"
git push -u origin feature/frontend-cart-badge
```

```
[feature/frontend-cart-badge 7a8b9c0] เพิ่มไอคอนตะกร้าสินค้าพร้อมตัวเลขจำนวนใน header
 2 files changed, 5 insertions(+)
```

### 446.3 ชูใจได้รับ review และ merge ก่อน

```bash
cd ~/git-course/teamcommerce-team/manee
git fetch origin
git diff main origin/feature/frontend-search-bar
```

มานีในฐานะ Code Owner ของ frontend อีกคนหนึ่ง review งานของชูใจและ approve:

```bash
git switch main
git merge --no-ff origin/feature/frontend-search-bar -m "Merge branch 'feature/frontend-search-bar' into main

Reviewed-by: Manee Suksan <manee@teamcommerce.dev>"
git push origin main
git push origin --delete feature/frontend-search-bar
```

```
[main 6f7a8b9] Merge branch 'feature/frontend-search-bar' into main
...
   4d5e6f7..6f7a8b9  main -> main
```

`main` ตอนนี้อยู่ที่ `6f7a8b9` ซึ่งมีช่องค้นหาแล้ว **แต่มานียังคง branch `feature/frontend-cart-badge` ของตัวเองไว้ที่จุดเก่า (`4d5e6f7`)** — เธอยังไม่รู้เรื่องนี้เลยจนกว่าจะกลับไปที่ branch ของตัวเอง

### 446.4 มานีกลับไปทำงานต่อ พบว่า main เปลี่ยนไปแล้ว

```bash
git switch feature/frontend-cart-badge
git fetch origin
git log --oneline --graph --all
```

```
* 7a8b9c0 (HEAD -> feature/frontend-cart-badge) เพิ่มไอคอนตะกร้าสินค้าพร้อมตัวเลขจำนวนใน header
| * 6f7a8b9 (origin/main) Merge branch 'feature/frontend-search-bar' into main
| * 5e6f7a8 เพิ่มช่องค้นหาสินค้าใน header
|/
* 4d5e6f7 Merge branch 'feature/backend-cart-api' into main
```

ตามหลักที่เรียนใน Part 37–39 มานีต้อง **rebase branch ของตัวเองเข้ากับ `main` ล่าสุดก่อนขอ review เสมอ** (ตรงตามข้อ 3 ของ `docs/WORKFLOW.md` ที่ตกลงกันไว้) แทนที่จะ merge main เข้ามา เพื่อให้ประวัติของ feature branch เรียงเป็นเส้นตรงสะอาด ๆ ก่อน merge

### 446.5 Rebase และเจอ Conflict จริง

```bash
git rebase origin/main
```

```
Auto-merging frontend/index.html
CONFLICT (content): Merge conflict in frontend/index.html
error: could not apply 7a8b9c0... เพิ่มไอคอนตะกร้าสินค้าพร้อมตัวเลขจำนวนใน header
Resolved 'frontend/index.html' using previous resolution.
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
```

เปิดไฟล์ดู:

```bash
cat frontend/index.html
```

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>TeamCommerce</title>
  <link rel="stylesheet" href="css/style.css">
</head>
<body>
  <header class="site-header">
    <h1>TeamCommerce</h1>
<<<<<<< HEAD
    <input type="search" id="search-box" placeholder="ค้นหาสินค้า...">
=======
    <span id="cart-badge" class="cart-badge">0</span>
>>>>>>> 7a8b9c0 (เพิ่มไอคอนตะกร้าสินค้าพร้อมตัวเลขจำนวนใน header)
  </header>
  <main id="product-list" class="product-grid"></main>
  <script src="js/app.js"></script>
</body>
</html>
```

> **จุดสำคัญที่ต้องจำ:** สังเกตว่าฝั่ง `HEAD` ตอนนี้คือ `main` (โค้ดของชูใจ) ไม่ใช่ branch ของมานีเหมือนตอน merge — เพราะระหว่าง rebase Git จะ "เล่นซ้ำ" commit ของมานีทีละตัวบนฐานของ `main` ใหม่ ทำให้ `HEAD` ระหว่าง rebase หมายถึงจุดที่กำลัง apply commit ทับอยู่ (คือ main ปัจจุบัน) ส่วนอีกฝั่งคือ commit ของมานีที่กำลังพยายาม apply เข้าไป ตรงข้ามกับตอน merge ที่ `HEAD` จะเป็นฝั่งของตัวเอง

### 446.6 แก้ conflict โดยรวมทั้งสองความตั้งใจเข้าด้วยกัน

```bash
cat > frontend/index.html << 'EOF'
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>TeamCommerce</title>
  <link rel="stylesheet" href="css/style.css">
</head>
<body>
  <header class="site-header">
    <h1>TeamCommerce</h1>
    <input type="search" id="search-box" placeholder="ค้นหาสินค้า...">
    <span id="cart-badge" class="cart-badge">0</span>
  </header>
  <main id="product-list" class="product-grid"></main>
  <script src="js/app.js"></script>
</body>
</html>
EOF

grep -n "<<<<<<<\|=======\|>>>>>>>" frontend/index.html
```

ไม่มีผลลัพธ์ — ไฟล์สะอาดแล้ว ดำเนินการต่อ:

```bash
git add frontend/index.html
git rebase --continue
```

```
[detached HEAD 8b9c0d1] เพิ่มไอคอนตะกร้าสินค้าพร้อมตัวเลขจำนวนใน header
 2 files changed, 5 insertions(+)
Successfully rebased and updated refs/heads/feature/frontend-cart-badge.
```

> **สิ่งที่เกิดขึ้นตอนนี้ (ที่จะสำคัญมากใน Step 448):** ระหว่างพิมพ์เนื้อหาไฟล์ `index.html` ใหม่ทั้งไฟล์เพื่อแก้ conflict มานีรีบทำโดยไม่ทันสังเกตว่า **ไม่ได้แตะไฟล์ `frontend/js/app.js` เลย** ซึ่งไฟล์นั้นยังมีฟังก์ชัน `updateCartBadge` เหมือนเดิม แต่ปัญหาคือ commit ผลลัพธ์จาก rebase นี้ (`8b9c0d1`) จะกลายเป็นจุดที่เราต้องสืบสวนกลับมาดูอีกครั้งใน Step 448 และ 449 — จำ hash `8b9c0d1` ไว้ให้แม่น

### 446.7 Force-push feature branch ที่ถูก rebase แล้ว

เนื่องจาก rebase เขียนประวัติของ `feature/frontend-cart-badge` ใหม่ทั้งหมด (commit hash เปลี่ยนจาก `7a8b9c0` เป็น `8b9c0d1`) การ push ปกติจะถูกปฏิเสธเพราะไม่ใช่ fast-forward เราต้องใช้ `--force-with-lease` ตามที่เรียนใน Part 37 เพื่อความปลอดภัย (ป้องกันการเขียนทับงานของคนอื่นโดยไม่ตั้งใจ หากมีคนอื่น push เข้า branch นี้ไปแล้วโดยที่เราไม่รู้):

```bash
git push --force-with-lease origin feature/frontend-cart-badge
```

```
...
 + 7a8b9c0...8b9c0d1 feature/frontend-cart-badge -> feature/frontend-cart-badge (forced update)
```

### 446.8 ขอ review และ merge

```bash
git request-pull 6f7a8b9 ../../teamcommerce-central.git feature/frontend-cart-badge
```

ชูใจ review และ approve (ทดสอบว่าทั้งช่องค้นหาและตะกร้าอยู่ร่วมกันได้ปกติ):

```bash
cd ~/git-course/teamcommerce-team/chujai
git fetch origin
git switch main
git pull origin main
git merge --no-ff origin/feature/frontend-cart-badge -m "Merge branch 'feature/frontend-cart-badge' into main

Reviewed-by: Chujai Sukjai <chujai@teamcommerce.dev>"
git push origin main
git push origin --delete feature/frontend-cart-badge
```

```
[main 9c0d1e2] Merge branch 'feature/frontend-cart-badge' into main
...
   6f7a8b9..9c0d1e2  main -> main
```

`main` อยู่ที่ `9c0d1e2` แล้ว ทีมปิด milestone ที่สอง วีระตัด tag และ release branch อีกชุด:

```bash
cd ~/git-course/teamcommerce-team/weera
git pull origin main
git tag -a v1.1.0 -m "TeamCommerce v1.1.0: เพิ่มช่องค้นหาและไอคอนตะกร้าสินค้า"
git push origin v1.1.0
git branch release/1.1 v1.1.0
git push origin release/1.1
```

ตอนนี้ทีมมี 2 เวอร์ชันที่ให้บริการลูกค้าอยู่พร้อมกัน: `release/1.0` (ลูกค้ากลุ่มแรก) และ `release/1.1` (ลูกค้ากลุ่มใหม่ล่าสุด) — สถานการณ์นี้คือสิ่งที่จะทำให้ Step 447 น่าสนใจมาก

---

## Step 447: จำลอง Hotfix ด่วนที่ต้อง Cherry-pick ไปยังหลาย Release Branch

### 447.1 รายงานบั๊กร้ายแรงเข้ามา

ฝ่ายสนับสนุนลูกค้ารายงานเข้ามาว่า **ยอดชำระเงินในตะกร้าติดลบ** เมื่อใส่ส่วนลดมากกว่า 100% โดยไม่ได้ตั้งใจ (เช่น ใส่ค่า `discountPercent = 150` เข้าไปจากหน้าโปรโมชัน) และปัญหานี้เกิดขึ้น **ทั้งบน `release/1.0` และ `release/1.1`** ที่ลูกค้าทั้งสองกลุ่มกำลังใช้งานอยู่จริง — นี่คือสถานการณ์ hotfix แบบเร่งด่วนที่สุดที่ทีมพัฒนาต้องเจอ

### 447.2 สมชายสร้าง hotfix branch จาก main

ตามหลักการที่ถูกต้อง (เรียนใน Part 40) hotfix ควรแก้ที่ `main` ก่อนเสมอ (เพื่อให้ทุกเวอร์ชันในอนาคตไม่มีบั๊กนี้อีก) แล้วค่อย cherry-pick ย้อนกลับไปยัง release branch ที่ยังต้องดูแลอยู่:

```bash
cd ~/git-course/teamcommerce-team/somchai
git switch main
git pull origin main
git switch -c hotfix/cart-discount-bug
```

```bash
cat > backend/routes/cart.js << 'EOF'
// cart.js: จัดการตะกร้าสินค้า
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price * item.qty, 0);
}

function applyDiscount(total, discountPercent) {
  const safePercent = Math.min(Math.max(discountPercent, 0), 100);
  return total - (total * safePercent) / 100;
}

module.exports = { calculateTotal, applyDiscount };
EOF

git add backend/routes/cart.js
git commit -m "Hotfix: จำกัดค่า discountPercent ให้อยู่ระหว่าง 0-100 เพื่อป้องกันยอดรวมติดลบ"
```

```
[hotfix/cart-discount-bug a1b2c3d] Hotfix: จำกัดค่า discountPercent ให้อยู่ระหว่าง 0-100 เพื่อป้องกันยอดรวมติดลบ
 1 file changed, 1 insertion(+), 1 deletion(-)
```

จำ hash ของ commit นี้ไว้ให้แม่น: **`a1b2c3d`** — นี่คือ commit ที่เราจะ cherry-pick ไปยังหลาย branch ในอีกไม่กี่ขั้นตอนถัดไป

```bash
git push -u origin hotfix/cart-discount-bug
```

### 447.3 review และ merge เข้า main ทันที (ข้ามคิวปกติเพราะเป็นเรื่องด่วน)

```bash
git request-pull main ../../teamcommerce-central.git hotfix/cart-discount-bug
```

Weera ในฐานะผู้ดูแล root ช่วยเร่ง review ร่วมกับปิติ (Code Owner คนที่สองของ backend) เนื่องจากเป็นเหตุฉุกเฉิน:

```bash
cd ~/git-course/teamcommerce-team/piti
git fetch origin
git switch main
git pull origin main
git merge --no-ff origin/hotfix/cart-discount-bug -m "Merge branch 'hotfix/cart-discount-bug' into main

Reviewed-by: Piti Rungrueang <piti@teamcommerce.dev>"
git push origin main
```

```
[main b2c3d4e] Merge branch 'hotfix/cart-discount-bug' into main
...
   9c0d1e2..b2c3d4e  main -> main
```

### 447.4 Cherry-pick commit hotfix ไปยัง release/1.1 ก่อน (ใกล้เคียง main ที่สุด)

```bash
git switch release/1.1
git pull origin release/1.1
git cherry-pick -x a1b2c3d
```

```
[release/1.1 c3d4e5f] Hotfix: จำกัดค่า discountPercent ให้อยู่ระหว่าง 0-100 เพื่อป้องกันยอดรวมติดลบ
 Date: ...
 1 file changed, 1 insertion(+), 1 deletion(-)
```

สังเกตว่า flag `-x` ตามที่เรียนใน Part 40 จะเติมบรรทัด `(cherry picked from commit a1b2c3d...)` ต่อท้าย commit message โดยอัตโนมัติ ทำให้ทุกคนที่มาอ่านประวัติของ `release/1.1` ภายหลังรู้ทันทีว่า commit นี้มีต้นตอมาจากไหน — cherry-pick ครั้งนี้ไม่มี conflict เพราะ `release/1.1` แยกออกมาจาก `main` ไม่นาน โครงสร้างไฟล์ `cart.js` ยังใกล้เคียงกันมาก

```bash
git tag -a v1.1.1 -m "Hotfix: แก้บั๊กส่วนลดติดลบ"
git push origin release/1.1
git push origin v1.1.1
```

### 447.5 Cherry-pick commit เดียวกันไปยัง release/1.0 (เจอ conflict เพราะไฟล์ต่างกันมากกว่า)

```bash
git switch release/1.0
git pull origin release/1.0
git cherry-pick -x a1b2c3d
```

เนื่องจาก `release/1.0` ถูกตัดออกมาตั้งแต่ก่อนที่ปิติจะเพิ่มฟังก์ชัน `applyDiscount` เวอร์ชันล่าสุด (มันหยุดอยู่ที่โครงสร้างไฟล์เก่ากว่า) การ cherry-pick จึงเกิด conflict:

```
Auto-merging backend/routes/cart.js
CONFLICT (content): Merge conflict in backend/routes/cart.js
error: could not apply a1b2c3d... Hotfix: จำกัดค่า discountPercent ให้อยู่ระหว่าง 0-100 เพื่อป้องกันยอดรวมติดลบ
hint: after resolving the conflicts, mark the corrected files with
hint: "git add <paths>", then run "git cherry-pick --continue".
```

```bash
cat backend/routes/cart.js
```

```js
// cart.js: จัดการตะกร้าสินค้า (โครงเบื้องต้น)
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price * item.qty, 0);
}

function applyDiscount(total, discountPercent) {
<<<<<<< HEAD
  return total - (total * discountPercent) / 100;
=======
  const safePercent = Math.min(Math.max(discountPercent, 0), 100);
  return total - (total * safePercent) / 100;
>>>>>>> a1b2c3d (Hotfix: จำกัดค่า discountPercent ให้อยู่ระหว่าง 0-100 เพื่อป้องกันยอดรวมติดลบ)

module.exports = { calculateTotal, applyDiscount };
```

แก้ conflict โดยยึดตรรกะของ hotfix เป็นหลัก (ปลอดภัยที่สุด) แต่คงโครงไฟล์เดิมของ `release/1.0`:

```bash
cat > backend/routes/cart.js << 'EOF'
// cart.js: จัดการตะกร้าสินค้า (โครงเบื้องต้น)
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price * item.qty, 0);
}

function applyDiscount(total, discountPercent) {
  const safePercent = Math.min(Math.max(discountPercent, 0), 100);
  return total - (total * safePercent) / 100;
}

module.exports = { calculateTotal, applyDiscount };
EOF

git add backend/routes/cart.js
git cherry-pick --continue
```

```
[release/1.0 d4e5f6a] Hotfix: จำกัดค่า discountPercent ให้อยู่ระหว่าง 0-100 เพื่อป้องกันยอดรวมติดลบ
 (cherry picked from commit a1b2c3d...)
 1 file changed, 1 insertion(+), 1 deletion(-)
```

```bash
git tag -a v1.0.1 -m "Hotfix: แก้บั๊กส่วนลดติดลบ (backport)"
git push origin release/1.0
git push origin v1.0.1
```

> **จุดสำคัญที่ต้องจำ:** cherry-pick แก้ปัญหาได้ตรงจุดที่สุดในสถานการณ์แบบนี้ เพราะเราต้องการ **แค่การเปลี่ยนแปลงเดียว** (การแก้บั๊ก) ไม่ใช่ทั้งประวัติของ `main` ไปติดตั้งใน release branch ที่จงใจแยกไว้ไม่ให้มีฟีเจอร์ใหม่ปนเข้าไป — นี่คือความแตกต่างสำคัญระหว่าง `cherry-pick` กับ `merge`/`rebase` ที่เรียนมาตลอดเฟส 4

ทั้งสามที่ (`main`, `release/1.0`, `release/1.1`) ตอนนี้ปลอดภัยจากบั๊กนี้แล้วทั้งหมด

---

## Step 448: ใช้ `git bisect` หาบั๊กที่แอบแฝงในโปรเจกต์ทีม

ไม่กี่วันต่อมา QA รายงานเข้ามาอีกครั้งว่า **หน้ารายการสินค้าเกิด error และไม่แสดงผลอะไรเลยในบางกรณี** — เมื่อตรวจสอบเบื้องต้นพบว่าปัญหานี้ **ไม่เกิดที่ tag `v1.0.0`** (ทดสอบแล้วใช้งานได้ปกติ) แต่ **เกิดที่ `main` ปัจจุบัน** ระหว่างนั้นมีหลาย commit เกิดขึ้นจากหลายคน — ไม่มีใครรู้แน่ชัดว่า commit ไหนเป็นต้นเหตุ นี่คือสถานการณ์ที่ `git bisect` ถูกออกแบบมาแก้โดยเฉพาะ

### 448.1 เตรียม test script สำหรับตรวจสอบอัตโนมัติ

ในสถานการณ์จริง เราอยากให้ Git หาต้นเหตุให้อัตโนมัติแทนที่จะเช็คเองทีละ commit เราจึงเขียน script ตรวจสอบง่าย ๆ ที่ตรวจว่าไฟล์ `frontend/js/app.js` ยังมี guard clause ที่ปลอดภัยอยู่หรือไม่ (ในสถานการณ์จริงคุณอาจรัน automated test จริงแทน แต่หลักการเดียวกัน — script ต้อง exit 0 เมื่อ "ดี" และ exit ไม่ใช่ 0 เมื่อ "แย่"):

```bash
cd ~/git-course/teamcommerce-team/weera
git switch main
git pull origin main

cat > check-render-guard.sh << 'EOF'
#!/bin/sh
# ตรวจสอบว่าฟังก์ชัน renderProducts ยังมี guard clause ตรวจสอบ products ครบถ้วนหรือไม่
if grep -q "Array.isArray(products)" frontend/js/app.js; then
  exit 0   # ดี: guard clause ยังอยู่ครบ
else
  exit 1   # แย่: guard clause หายไปหรือถูกแก้ไขผิดพลาด
fi
EOF
chmod +x check-render-guard.sh
```

> **หมายเหตุ:** นี่คือการจำลอง automated test อย่างง่ายที่สุดเท่าที่จะทำได้เพื่อให้ `git bisect run` ทำงานได้โดยไม่ต้องพึ่ง test framework จริง ในการทำงานจริงคุณจะเขียน unit test จริง ๆ (เช่นด้วย Jest หรือ Mocha) แล้วให้ `git bisect run npm test` แทน — หลักการที่ `git bisect` ใช้เหมือนกันทุกประการไม่ว่า test จะซับซ้อนแค่ไหน

### 448.2 เริ่ม bisect

```bash
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
```

```
Bisecting: 4 revisions left to test after this (roughly 2 steps)
[6f7a8b9] Merge branch 'feature/frontend-search-bar' into main
```

Git กระโดดไปกึ่งกลางระหว่าง `v1.0.0` กับ `HEAD` ให้อัตโนมัติ

### 448.3 รันแบบอัตโนมัติด้วย git bisect run

แทนที่จะเช็คเองทีละ commit ด้วยมือ เราสั่งให้ Git รัน script ให้เองทั้งหมดในคำสั่งเดียว ตามที่เรียนใน Part 41–42:

```bash
git bisect run ./check-render-guard.sh
```

```
running ./check-render-guard.sh
Bisecting: 2 revisions left to test after this (roughly 1 step)
[4d5e6f7] Merge branch 'feature/backend-cart-api' into main
running ./check-render-guard.sh
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[9c0d1e2] Merge branch 'feature/frontend-cart-badge' into main
running ./check-render-guard.sh
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[8b9c0d1] เพิ่มไอคอนตะกร้าสินค้าพร้อมตัวเลขจำนวนใน header
running ./check-render-guard.sh
8b9c0d1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a is the first bad commit
commit 8b9c0d1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a
Author: Manee Suksan <manee@teamcommerce.dev>
Date:   ...

    เพิ่มไอคอนตะกร้าสินค้าพร้อมตัวเลขจำนวนใน header

 frontend/index.html | 3 ++-
 frontend/js/app.js   | 5 +++++
 2 files changed, 1 insertion(+), 1 deletion(-)
bisect found first bad commit
```

**พบต้นเหตุแล้ว: commit `8b9c0d1`** — นี่คือ commit ผลลัพธ์จากการ rebase แก้ conflict ของมานีใน Step 446 นั่นเอง! `git bisect` ทำหน้าที่ของมันได้อย่างแม่นยำ ไล่ทดสอบทีละครึ่งจากทั้งหมด 6 commit ที่อยู่ระหว่าง `v1.0.0` กับ `HEAD` โดยใช้เวลาทดสอบแค่ 4 ครั้งเท่านั้น (log2 ของจำนวน commit) แทนที่จะต้องเช็คทีละตัวถึง 6 ครั้ง

### 448.4 ออกจากโหมด bisect

```bash
git bisect reset
```

```
Previous HEAD position was 8b9c0d1 เพิ่มไอคอนตะกร้าสินค้าพร้อมตัวเลขจำนวนใน header
Switched to branch 'main'
```

`git bisect reset` พาเรากลับไปที่ branch เดิมก่อนเริ่ม bisect เสมอ — ห้ามลืมขั้นตอนนี้ มิฉะนั้น repository จะค้างอยู่ในสถานะ detached HEAD

ลบไฟล์ script ทดสอบทิ้ง เพราะไม่ใช่ส่วนหนึ่งของโค้ดโปรเจกต์จริง:

```bash
rm check-render-guard.sh
```

ตอนนี้เรารู้แล้วว่า **commit ไหน** เป็นต้นเหตุ แต่ยังไม่รู้ว่า **เกิดจากอะไรกันแน่** — Step ถัดไปจะพาไปสืบลึกลงไปอีกขั้น

---

## Step 449: ใช้ `git blame` สืบสวนที่มาของโค้ดที่มีปัญหาก่อนแก้ไข

### 449.1 ดูโค้ดปัจจุบันของไฟล์ที่น่าสงสัย

```bash
cat frontend/js/app.js
```

```js
// app.js: render รายการสินค้าลงในหน้าเว็บ
function renderProducts(products) {
  if (!Array.isArray(products) || products.length === 0) return;
  const container = document.getElementById("product-list");
  container.innerHTML = products
    .map((p) => `<div class="product-card">${p.name} - ${p.price} บาท</div>`)
    .join("");
}

function updateCartBadge(count) {
  document.getElementById("cart-badge").textContent = String(count);
}
```

ดูเผิน ๆ โค้ดนี้ดูปกติดี guard clause `if (!Array.isArray(products) || products.length === 0) return;` ยังอยู่ครบ แต่ปัญหาจริงคือฟังก์ชัน `updateCartBadge` ถูกเพิ่มเข้ามาโดยที่ **ไม่มี guard clause ป้องกัน element ที่ยังไม่ถูกสร้างในหน้า** — เมื่อหน้าเว็บโหลดยังไม่เสร็จสมบูรณ์ (หรือถูกเรียกใช้ก่อนที่ `#cart-badge` จะถูกวาดในบางลำดับการโหลด) `document.getElementById("cart-badge")` จะคืนค่า `null` แล้วเรียก `.textContent` ต่อจาก `null` ทำให้เกิด `TypeError` ทันที ซึ่งบั๊กที่ `git bisect` เจอไม่ใช่การ "ลบ" อะไรออกไป แต่คือการ "เพิ่ม" โค้ดที่ไม่มีการป้องกันความปลอดภัยเข้ามาแบบเดียวกับที่เกิดขึ้นจริงเวลาสองคนรวมโค้ดของกันและกันอย่างเร่งรีบตอนแก้ conflict

### 449.2 ใช้ git blame ยืนยันว่าใครและเมื่อไหร่ที่เพิ่มบรรทัดนี้เข้ามา

```bash
git blame -L 10,12 frontend/js/app.js
```

```
8b9c0d1c (Manee Suksan 2026-08-05 14:32:10 +0700 10)
8b9c0d1c (Manee Suksan 2026-08-05 14:32:10 +0700 11) function updateCartBadge(count) {
8b9c0d1c (Manee Suksan 2026-08-05 14:32:10 +0700 12)   document.getElementById("cart-badge").textContent = String(count);
```

`git blame` ยืนยันตรงกับผลของ `git bisect` เป๊ะ: commit `8b9c0d1` โดยมานี วันที่ 5 สิงหาคม — ตรงกับตอนที่เธอ rebase แก้ conflict ใน Step 446 ทุกประการ

### 449.3 ดูบริบทเต็มของ commit นั้นด้วย git show

```bash
git show 8b9c0d1
```

```
commit 8b9c0d1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a
Author: Manee Suksan <manee@teamcommerce.dev>
Date:   Wed Aug 5 14:32:10 2026 +0700

    เพิ่มไอคอนตะกร้าสินค้าพร้อมตัวเลขจำนวนใน header

diff --git a/frontend/index.html b/frontend/index.html
...
diff --git a/frontend/js/app.js b/frontend/js/app.js
index 1234567..89abcde 100644
--- a/frontend/js/app.js
+++ b/frontend/js/app.js
@@ -5,3 +5,8 @@ function renderProducts(products) {
     .map((p) => `<div class="product-card">${p.name} - ${p.price} บาท</div>`)
     .join("");
 }
+
+function updateCartBadge(count) {
+  document.getElementById("cart-badge").textContent = String(count);
+}
```

ตอนนี้ทีมเข้าใจต้นเหตุครบทั้งสามมิติแล้ว: **commit ไหน** (`git bisect` บอก), **ใครและเมื่อไหร่** (`git blame` บอก), และ **เกิดอะไรขึ้นจริง ๆ ในโค้ด** (`git show` บอก) — ครบสูตรการสืบสวนบั๊กแบบมืออาชีพด้วยเครื่องมือของ Git ล้วน ๆ โดยไม่ต้องพึ่งเครื่องมือภายนอกใด ๆ เลย

> **ข้อสังเกตสำคัญ:** นี่ไม่ใช่การตำหนิมานี — ในทีมที่ดี `git blame` ไม่เคยถูกใช้เพื่อ "จับผิด" แต่ใช้เพื่อ **เข้าใจบริบท** ว่าทำไมโค้ดถึงเป็นแบบนี้ ในกรณีนี้เหตุผลชัดเจนมาก: มานีกำลังรีบแก้ conflict ระหว่าง rebase และพลาดที่จะเพิ่ม guard clause ให้ครบ ซึ่งเป็นความผิดพลาดที่เกิดขึ้นได้กับทุกคน โดยเฉพาะตอนแก้ conflict ที่ต้องรวมโค้ดจากสองที่มาเข้าด้วยกันอย่างเร่งรีบ

### 449.4 แก้ไขให้ถูกต้อง

```bash
git switch -c fix/cart-badge-null-check
```

```bash
cat > frontend/js/app.js << 'EOF'
// app.js: render รายการสินค้าลงในหน้าเว็บ
function renderProducts(products) {
  if (!Array.isArray(products) || products.length === 0) return;
  const container = document.getElementById("product-list");
  container.innerHTML = products
    .map((p) => `<div class="product-card">${p.name} - ${p.price} บาท</div>`)
    .join("");
}

function updateCartBadge(count) {
  const badge = document.getElementById("cart-badge");
  if (!badge) return;
  badge.textContent = String(count);
}
EOF

git add frontend/js/app.js
git commit -m "แก้บั๊ก: เพิ่ม null check ใน updateCartBadge ป้องกัน TypeError เมื่อ element ยังไม่พร้อม"
git push -u origin fix/cart-badge-null-check
```

```
[fix/cart-badge-null-check e5f6a7b] แก้บั๊ก: เพิ่ม null check ใน updateCartBadge ป้องกัน TypeError เมื่อ element ยังไม่พร้อม
 1 file changed, 2 insertions(+), 1 deletion(-)
```

### 449.5 ขอ review จาก Code Owner แล้ว merge เข้า main พร้อม cherry-pick ไปยัง release/1.1

```bash
git request-pull main ../../teamcommerce-central.git fix/cart-badge-null-check
```

มานีในฐานะผู้เขียนไม่สามารถ approve งานตัวเองได้ ชูใจรับหน้าที่รีวิวและ merge:

```bash
cd ~/git-course/teamcommerce-team/chujai
git fetch origin
git switch main
git pull origin main
git merge --no-ff origin/fix/cart-badge-null-check -m "Merge branch 'fix/cart-badge-null-check' into main

Reviewed-by: Chujai Sukjai <chujai@teamcommerce.dev>"
git push origin main
git push origin --delete fix/cart-badge-null-check
```

```
[main f6a7b8c] Merge branch 'fix/cart-badge-null-check' into main
...
   b2c3d4e..f6a7b8c  main -> main
```

เนื่องจากฟีเจอร์ตะกร้าสินค้าถูกปล่อยไปพร้อมกับ `release/1.1` เท่านั้น (ไม่มีใน `release/1.0` ที่ตัดออกไปก่อนฟีเจอร์นี้จะถูกสร้าง) จึงต้อง cherry-pick ไปแค่ `release/1.1` แห่งเดียว:

```bash
git switch release/1.1
git pull origin release/1.1
git cherry-pick -x e5f6a7b
git push origin release/1.1
git tag -a v1.1.2 -m "แก้บั๊ก: null check ใน updateCartBadge"
git push origin v1.1.2
```

```
[release/1.1 g7h8i9j] แก้บั๊ก: เพิ่ม null check ใน updateCartBadge ป้องกัน TypeError เมื่อ element ยังไม่พร้อม
 (cherry picked from commit e5f6a7b...)
```

ปัญหาถูกแก้ครบถ้วนทั้งใน `main` และ `release/1.1` — วงจรทั้งหมดตั้งแต่ตรวจพบบั๊ก → หาต้นเหตุด้วย bisect → สืบบริบทด้วย blame → แก้ไขผ่านกระบวนการรีวิวปกติ → กระจายไปยัง release ที่เกี่ยวข้องด้วย cherry-pick ปิดจบสมบูรณ์

---

## Step 450: สรุปทบทวนภาพรวมเฟส 4 ทั้งหมด (Part 31–45) พร้อม Cheat Sheet และ Checklist ก่อนเข้าเฟส 5

### 450.1 ภาพรวมกราฟทั้งหมดของโปรเจกต์ TeamCommerce

```bash
cd ~/git-course/teamcommerce-team/weera
git fetch origin
git log --oneline --graph --all --decorate
```

```
*   f6a7b8c (origin/main, main) Merge branch 'fix/cart-badge-null-check' into main
|\
| * e5f6a7b แก้บั๊ก: เพิ่ม null check ใน updateCartBadge
|/
*   b2c3d4e Merge branch 'hotfix/cart-discount-bug' into main
|\
| * a1b2c3d Hotfix: จำกัดค่า discountPercent ให้อยู่ระหว่าง 0-100
|/
*   9c0d1e2 (tag: v1.1.0) Merge branch 'feature/frontend-cart-badge' into main
|\
| * 8b9c0d1 เพิ่มไอคอนตะกร้าสินค้าพร้อมตัวเลขจำนวนใน header
|/
*   6f7a8b9 Merge branch 'feature/frontend-search-bar' into main
|\
| * 5e6f7a8 เพิ่มช่องค้นหาสินค้าใน header
|/
*   4d5e6f7 (tag: v1.0.0) Merge branch 'feature/backend-cart-api' into main
|\
| * 2b3c4d5 เพิ่มฟังก์ชัน applyDiscount สำหรับคำนวณส่วนลด
| * 1a2b3c4 เพิ่มโครง cart API พร้อมฟังก์ชันคำนวณยอดรวมเบื้องต้น
* |   3c4d5e6 Merge branch 'feature/frontend-product-list' into main
|\ \
| |/
|/|
| * f6a7b85 เพิ่มฟังก์ชัน renderProducts พร้อม guard clause
| * e5f6a74 เพิ่ม CSS จัดสไตล์การ์ดสินค้าแบบ Grid
| * d4e5f63 เพิ่มโครง HTML สำหรับหน้ารายการสินค้า
|/
* c3d4e52 เพิ่มเอกสารอธิบายการเลือกใช้ GitHub Flow
* b2c3d41 เพิ่ม CODEOWNERS, PR template, และ branch naming convention
* a1b2c30 Initial commit: โครงสร้างเริ่มต้นของโปรเจกต์ TeamCommerce
```

ลองอ่านกราฟนี้ทีละบรรทัดด้วยตัวเอง — ถ้าคุณอธิบายได้ว่าแต่ละจุดคือเหตุการณ์อะไร ใครทำ และทำไมถึงต้องทำแบบนั้น แปลว่าคุณเข้าใจการทำงานเป็นทีมด้วย Git อย่างแท้จริงแล้ว ไม่ใช่แค่จำคำสั่งได้

### 450.2 Cheat Sheet รวมคำสั่งและแนวคิดทั้งหมดของเฟส 4 (Part 31–45)

| หัวข้อ (Part) | คำสั่ง / แนวคิดหลัก | ใช้เมื่อไหร่ |
|---|---|---|
| Workflow Models (31) | Git Flow, GitHub Flow, Trunk-Based Development | เลือกตามขนาดทีมและรูปแบบการ deploy |
| Naming Convention (32) | `feature/*`, `hotfix/*`, `release/*` | ตั้งชื่อ branch ให้สื่อความหมายและตรวจสอบอัตโนมัติได้ |
| Branch Protection (33-35) | GitHub Settings → Branches / `pre-receive` hook | ห้าม push ตรงเข้า main, บังคับ review ก่อน merge |
| CODEOWNERS (36) | ไฟล์ `.github/CODEOWNERS` + pattern matching | auto-request review ตามเจ้าของไฟล์/โฟลเดอร์ |
| Rebase พื้นฐาน (37) | `git rebase <branch>` | ทำประวัติ feature branch เรียงเป็นเส้นตรงก่อน merge |
| Interactive Rebase (38) | `git rebase -i HEAD~n` (pick/squash/fixup/reword/drop) | จัดระเบียบ commit ก่อนขอ review |
| Rebase Conflict ขั้นสูง (39) | `git rebase --continue` / `--abort` / `--skip` | แก้ conflict ทีละ commit ระหว่าง rebase |
| Cherry-pick (40) | `git cherry-pick [-x] <hash>` | ย้าย commit เดียวข้าม branch โดยไม่เอาประวัติทั้งหมด |
| Conflict ขั้นสูง (41-ish) | 3-way merge, `git diff3`, `git checkout --ours/--theirs` | สถานการณ์ conflict ที่ซับซ้อนกว่าปกติ |
| Bisect (41-42) | `git bisect start/bad/good/run/reset` | หา commit ที่ทำให้เกิดบั๊กแบบ binary search |
| Blame (43) | `git blame -L <start>,<end> <file>` | หาว่าใครและเมื่อไหร่ที่แก้บรรทัดใดบรรทัดหนึ่ง |
| Submodules (44) | `git submodule add/update --init --recursive` | ผูก repository อื่นเข้ามาเป็นส่วนหนึ่งของโปรเจกต์ |
| Capstone (45) | ผสมทุกอย่างข้างต้นในสถานการณ์จริง | ปิดเฟส 4 ก่อนเข้าเฟส 5 (GitLab) |

### 450.3 ตารางสรุปคำสั่ง Git ที่ใช้บ่อยที่สุดตลอด Part นี้

| คำสั่ง | ความหมาย |
|---|---|
| `git switch -c feature/<name>` | สร้างและสลับไป branch ใหม่ตาม naming convention |
| `git push -u origin <branch>` | push branch ใหม่พร้อมตั้ง upstream tracking |
| `git request-pull <base> <url> <branch>` | สร้างสรุปการเปลี่ยนแปลงเพื่อขอ review แบบไม่ใช้ GitHub UI |
| `git merge --no-ff <branch> -m "... Reviewed-by: ..."` | merge พร้อม trailer ยืนยันการรีวิว |
| `git rebase origin/main` | ปรับ feature branch ให้อยู่บนฐาน main ล่าสุดก่อนขอ review |
| `git push --force-with-lease` | push ประวัติที่ถูกเขียนใหม่จาก rebase อย่างปลอดภัย |
| `git cherry-pick -x <hash>` | คัด commit เดียวไปยัง release branch พร้อมอ้างอิงที่มา |
| `git bisect start/bad/good/run/reset` | หา commit ต้นเหตุของบั๊กด้วย binary search อัตโนมัติ |
| `git blame -L <n>,<m> <file>` | สืบว่าใครแก้บรรทัดไหนและเมื่อไหร่ |
| `git cat-file -p <hash>` | ตรวจสอบ object ดิบของ commit (ใช้ตรวจนับ parent ใน hook) |

### 450.4 Checklist ทบทวนภาพรวมเฟส 4 ทั้งหมด (Part 31–45) ก่อนเข้าสู่เฟส 5

ก่อนไปต่อ Part 46 (เริ่มต้นเฟส 5: GitLab) ให้ตรวจสอบตัวเองอย่างละเอียดตามรายการนี้ ถ้าข้อไหนยังไม่มั่นใจ แนะนำให้ย้อนกลับไปอ่าน Part ที่เกี่ยวข้องอีกครั้งก่อน:

**Workflow และการวางแผนทีม**
- [ ] อธิบายความแตกต่างระหว่าง Git Flow, GitHub Flow และ Trunk-Based Development ได้ พร้อมยกตัวอย่างสถานการณ์ที่เหมาะกับแต่ละแบบ
- [ ] เลือก Workflow ที่เหมาะกับทีมของตัวเองได้พร้อมให้เหตุผลประกอบ ไม่ใช่แค่เลือกตามความเคยชิน
- [ ] ตั้งกฎ naming convention ให้ทีมได้และอธิบายได้ว่าทำไมชื่อ branch ที่สื่อความหมายถึงสำคัญ

**Governance และ Branch Protection**
- [ ] เขียนไฟล์ `CODEOWNERS` ที่รองรับหลายทีมในโปรเจกต์เดียว (monorepo) ได้ถูก syntax
- [ ] อธิบายได้ว่า Branch Protection Rule แต่ละแบบ (require review, require status check, restrict push) ป้องกันอะไร
- [ ] เข้าใจว่า server-side hook อย่าง `pre-receive` คือกลไกจริงที่อยู่เบื้องหลัง Branch Protection ของ GitHub/GitLab

**Rebase**
- [ ] อธิบายความแตกต่างระหว่าง `merge` กับ `rebase` ได้ชัดเจนทั้งในแง่ประวัติที่ได้และความเสี่ยง
- [ ] ใช้ `git rebase -i` จัดระเบียบ commit (squash, fixup, reword, reorder) ได้คล่อง
- [ ] แก้ conflict ระหว่าง rebase ได้โดยไม่สับสนว่าฝั่งไหนคือ `HEAD`
- [ ] รู้จักใช้ `--force-with-lease` แทน `--force` เสมอเมื่อ push ประวัติที่ถูกเขียนใหม่

**Cherry-pick**
- [ ] เข้าใจว่าเมื่อไหร่ควรใช้ cherry-pick แทน merge หรือ rebase
- [ ] ใช้ `-x` เพื่อบันทึกที่มาของ commit ที่ถูก cherry-pick เสมอ
- [ ] แก้ conflict ระหว่าง cherry-pick ได้เมื่อ branch ปลายทางมีโครงสร้างต่างจากต้นทาง

**การสืบสวนปัญหา**
- [ ] ใช้ `git bisect` (ทั้งแบบ manual และแบบ `run` อัตโนมัติ) หา commit ต้นเหตุของบั๊กได้ด้วยตัวเอง
- [ ] ใช้ `git blame` สืบที่มาของโค้ดได้ และเข้าใจว่ามันมีไว้เพื่อเข้าใจบริบท ไม่ใช่จับผิดเพื่อนร่วมทีม
- [ ] เชื่อมโยงผลจาก `bisect` และ `blame` เข้าด้วยกันเพื่อเข้าใจปัญหาอย่างครบวงจรก่อนลงมือแก้

**Submodules**
- [ ] เข้าใจว่า Submodule คืออะไร แก้ปัญหาอะไร และมีข้อจำกัดอะไรบ้าง
- [ ] เพิ่ม, อัปเดต, และ clone โปรเจกต์ที่มี submodule ได้โดยไม่ทำข้อมูลหาย

**ภาพรวมโปรเจกต์ทีม**
- [ ] จำลองทีมหลายคนทำงานพร้อมกันด้วยหลาย local clone ผ่าน bare repo กลางได้ครบทุกขั้นตอนด้วยตัวเองจริง
- [ ] อ่านกราฟ `git log --oneline --graph --all --decorate` ของโปรเจกต์ทีมที่มี merge, rebase, cherry-pick ปนกันได้เข้าใจทุกจุด
- [ ] อธิบายวงจรชีวิตทั้งหมดของฟีเจอร์หนึ่งตัว ตั้งแต่แตก branch จนถึง merge เข้า main ให้คนอื่นฟังได้อย่างเป็นระบบ

ถ้าคุณติ๊กครบทุกข้อ (หรือเกือบครบ) แปลว่าคุณพร้อมสำหรับเฟส 5 อย่างแท้จริงแล้ว — เฟส 4 คือเฟสที่เปลี่ยนคุณจากคนที่ "ใช้ Git เป็น" ไปสู่คนที่ "ทำงานเป็นทีมด้วย Git ได้อย่างมืออาชีพ" ซึ่งเป็นทักษะที่แยกวิศวกรระดับ mid ออกจากระดับ junior อย่างชัดเจนที่สุดในโลกการทำงานจริง

---

## สรุป Part 45

ใน Part นี้เราได้จำลองทีมพัฒนาซอฟต์แวร์ 5 คนทำงานร่วมกันบนโปรเจกต์ TeamCommerce ตั้งแต่ต้นจนจบ โดยนำทุกทักษะที่เรียนมาตลอดเฟส 4 มาใช้งานร่วมกันในสถานการณ์ที่จำลองมาจากการทำงานจริง:

1. วางโครงสร้างทีมและจำลองสมาชิก 5 คนด้วยหลาย local clone ผ่าน bare repo กลาง (Step 441)
2. ตั้งค่า Branch Protection ด้วย `pre-receive` hook, CODEOWNERS แบ่งทีม frontend/backend, PR template และ naming convention (Step 442)
3. เลือกใช้ GitHub Flow เป็น Workflow หลักพร้อมเหตุผลประกอบที่ชัดเจน (Step 443)
4. จำลองสองฟีเจอร์ที่พัฒนาพร้อมกันโดยคนละคนบนคนละเครื่อง (Step 444)
5. ทำ Code Review และขอ Approval ตาม CODEOWNERS ผ่าน `git request-pull` และเห็น Branch Protection ปฏิเสธการ push ที่ไม่ผ่านกฎจริง (Step 445)
6. แก้ Conflict ระหว่างสมาชิก 2 คนที่แก้ไฟล์เดียวกันด้วย `git rebase` (Step 446)
7. รับมือ Hotfix ด่วนที่กระทบหลายเวอร์ชัน แล้วกระจายการแก้ไขด้วย `git cherry-pick -x` (Step 447)
8. ใช้ `git bisect` ไล่หา commit ต้นเหตุของบั๊กแบบอัตโนมัติด้วย `bisect run` (Step 448)
9. ใช้ `git blame` และ `git show` สืบบริบทของโค้ดที่มีปัญหาก่อนจะแก้ไขอย่างถูกวิธี (Step 449)
10. ทบทวนภาพรวมคำสั่งและแนวคิดทั้งหมดของเฟส 4 ผ่าน Cheat Sheet และ Checklist ครบทุกด้าน (Step 450)

Part นี้คือจุดปิดฉากของ **เฟส 4: ทำงานเป็นทีมด้วย Workflow มาตรฐาน (Part 31–45, Step 301–450)** อย่างสมบูรณ์ คุณได้ผ่านการฝึกฝนทักษะที่แยกวิศวกรที่ "ใช้ Git คนเดียวเป็น" ออกจากวิศวกรที่ "ทำงานเป็นทีมได้จริง" มาครบทุกมิติแล้ว ตั้งแต่การวาง Workflow, การบังคับใช้กฎผ่าน Branch Protection และ CODEOWNERS, การจัดการประวัติด้วย Rebase, การกระจาย hotfix ด้วย Cherry-pick, ไปจนถึงการสืบสวนปัญหาด้วย Bisect และ Blame

จากนี้ไป หลักสูตรจะพาคุณเข้าสู่ **เฟส 5: เจาะลึก GitLab และ CI/CD เบื้องต้น (Part 46–55, Step 451–550)** ซึ่งจะแนะนำแพลตฟอร์มทางเลือกที่ได้รับความนิยมสูงในองค์กรขนาดใหญ่และหน่วยงานที่ต้องการควบคุมข้อมูลของตัวเอง พร้อมเริ่มต้นเข้าสู่โลกของ CI/CD อย่างเป็นทางการ

**ต่อไป:** [Part 46: GitLab คืออะไร ต่างจาก GitHub อย่างไร](./part-046-gitlab-คืออะไร.md)
