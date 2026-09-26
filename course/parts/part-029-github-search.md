# Part 29: GitHub Search และการค้นหาโค้ด/โปรเจกต์อย่างมีประสิทธิภาพ

> **Step ในหลักสูตรนี้:** Step 281–290
> **เฟส:** 3 — ใช้งาน GitHub อย่างมืออาชีพ ทำ Pull Request, Code Review, Open Source
> **เป้าหมายของ Part นี้:** ใช้ GitHub Search ได้อย่างคล่องแคล่วในทุกรูปแบบ ตั้งแต่การค้นหาพื้นฐานไปจนถึง search syntax ขั้นสูง เพื่อค้นหา repository, โค้ด, issue/PR, ผู้ใช้ และหมวดหมู่โปรเจกต์ได้อย่างแม่นยำ รวมถึงรู้จักใช้ Trending และ Explore เพื่อติดตามความเคลื่อนไหวของวงการ Open Source และประเมินความน่าเชื่อถือของ library ก่อนนำมาใช้งานจริง

---

## สารบัญของ Part นี้

- Step 281: การค้นหาพื้นฐานบน GitHub (repository, code, issues, users, commits)
- Step 282: Search syntax ขั้นสูง (`in:name`, `language:python`, `stars:>1000`, `size:>1000`)
- Step 283: ค้นหาโค้ดเฉพาะ (code search) พร้อมตัวอย่างการค้นหา function/pattern เฉพาะ
- Step 284: การค้นหา issue/PR ด้วย filter ซับซ้อน (`is:pr is:merged review:approved`, `is:issue is:open label:bug`)
- Step 285: Advanced search UI ของ GitHub (github.com/search/advanced)
- Step 286: การใช้ Topics ค้นหาโปรเจกต์ตามหมวดหมู่ (`topic:machine-learning`)
- Step 287: Trending repositories — หาโปรเจกต์ดังประจำวัน/สัปดาห์ (github.com/trending)
- Step 288: GitHub Explore และการติดตาม (follow) ผู้ใช้/organization
- Step 289: การใช้ search ช่วยหา library/dependency ที่น่าเชื่อถือ
- Step 290: แบบฝึกหัด — ฝึกค้นหาโปรเจกต์/โค้ดด้วยเทคนิคขั้นสูง 8 สถานการณ์จริง

---

## Step 281: การค้นหาพื้นฐานบน GitHub (repository, code, issues, users, commits)

GitHub ไม่ได้เป็นแค่ที่เก็บโค้ด แต่เป็น **เสิร์ชเอนจินขนาดใหญ่ที่รวมโค้ดของมนุษยชาติไว้ในที่เดียว** มีทั้ง repository นับร้อยล้าน โค้ดหลายพันล้านบรรทัด และ issue/PR ที่บันทึกการแก้ปัญหาจริงเอาไว้มหาศาล การรู้จัก "ค้นหาให้เป็น" คือทักษะที่แยกโปรแกรมเมอร์มืออาชีพออกจากมือใหม่ได้ชัดเจนมาก

### 281.1 ช่องค้นหาหลักอยู่ตรงไหน

เมื่อคุณ login เข้า GitHub แล้ว จะเห็นแถบค้นหาอยู่บนสุดของทุกหน้า (หรือกด **`/`** เพื่อโฟกัสไปที่ช่องค้นหาได้ทันทีโดยไม่ต้องเอาเมาส์ไปคลิก) พิมพ์คำค้นหาแล้วกด Enter จะพาไปยังหน้า `github.com/search?q=...` ซึ่งเป็นหน้าผลลัพธ์การค้นหาหลักของทั้งแพลตฟอร์ม

```
https://github.com/search?q=react+hooks
```

### 281.2 ประเภทของการค้นหา (Search Types)

หน้าผลลัพธ์การค้นหาของ GitHub แบ่งออกเป็นแท็บ (tab) หลายแบบ แต่ละแท็บค้นหาข้อมูลคนละประเภทกัน:

| แท็บ | ค้นหาอะไร | ตัวอย่างการใช้งาน |
|---|---|---|
| **Repositories** | ชื่อ, คำอธิบาย, README ของ repository | หา repo ที่เกี่ยวกับ "task manager" |
| **Code** | เนื้อหาจริงในไฟล์โค้ดทั้งหมดบน GitHub | หาว่ามีใครเขียน function ชื่อ `debounce` แบบไหนบ้าง |
| **Commits** | ข้อความ commit message | หา commit ที่พูดถึงการแก้ CVE เฉพาะเจาะจง |
| **Issues** | Issue ที่เปิดอยู่หรือปิดแล้วทุก repo | หาว่ามีคนเจอบั๊กเดียวกับเราไหม |
| **Pull requests** | PR ทุก repo | หาว่ามีใครส่ง PR แก้ปัญหานี้แล้วหรือยัง |
| **Discussions** | กระทู้สนทนาใน Discussions | หาคำถาม/คำตอบเกี่ยวกับหัวข้อหนึ่ง |
| **Users** | บัญชีผู้ใช้ | หาโปรไฟล์ของนักพัฒนาคนหนึ่ง |
| **Packages** | Package ที่ publish บน GitHub Packages | หา package เฉพาะ |
| **Marketplace** | GitHub Actions / Apps ใน Marketplace | หา Action สำเร็จรูปมาใช้ใน CI/CD |
| **Topics** | หมวดหมู่ที่ repo ติด tag ไว้ | หาโปรเจกต์ทั้งหมดที่ติด topic เดียวกัน |
| **Wikis** | เนื้อหาใน Wiki ของ repo | หาเอกสารประกอบโปรเจกต์ |

### 281.3 ตัวอย่างค้นหาแบบพื้นฐานที่สุด (ไม่ใช้ syntax พิเศษ)

**ค้นหา repository:**

```
express
```

พิมพ์คำเดียวแบบนี้ GitHub จะค้นหาจากชื่อ repo, คำอธิบาย (description) และ README แล้วเรียงผลลัพธ์ตาม **relevance (ความเกี่ยวข้อง)** เป็นค่าเริ่มต้น ซึ่งคำนวณจากหลายปัจจัยรวมกัน เช่น คำค้นหาตรงกับชื่อ repo มากแค่ไหน จำนวน star จำนวนครั้งที่ถูก fork และความสด (recency) ของกิจกรรมล่าสุด

**ค้นหาโค้ด:**

```
useEffect cleanup function
```

ค้นหาข้อความนี้ในเนื้อหาไฟล์โค้ดทุกไฟล์ที่ public บน GitHub (โค้ด search มีข้อจำกัดบางอย่างที่จะอธิบายเพิ่มใน Step 283)

**ค้นหา issue:**

```
memory leak
```

จะเจอ issue จากทุก repository ทั่วโลกที่มีคำว่า "memory leak" อยู่ในหัวข้อหรือเนื้อหา ซึ่งมีประโยชน์มากเวลาคุณเจอบั๊กแปลก ๆ แล้วอยากรู้ว่ามีใครเจอปัญหาเดียวกันมาก่อนหรือยัง

**ค้นหา user:**

```
torvalds
```

จะเจอโปรไฟล์ของ Linus Torvalds ผู้สร้าง Git และ Linux รวมถึง user คนอื่นที่ username หรือชื่อจริงตรงกับคำค้นหานี้

### 281.4 การเรียงลำดับผลลัพธ์ (Sort)

ที่มุมขวาบนของหน้าผลลัพธ์ค้นหา มีตัวเลือก **Sort** ให้เปลี่ยนจาก relevance (ค่าเริ่มต้น) เป็นแบบอื่น ๆ ได้ เช่น:

- **Most stars** — เรียงตามจำนวน star มากไปน้อย เหมาะกับตอนหา repo ยอดนิยม
- **Fewest stars** — เรียงตามจำนวน star น้อยไปมาก เหมาะกับตอนหา "เพชรที่ยังไม่เจียระไน" (hidden gem)
- **Most forks** — เรียงตามจำนวน fork มากไปน้อย
- **Recently updated** — เรียงตามวันที่มีการ commit ล่าสุด เหมาะกับตอนอยากรู้ว่าโปรเจกต์นี้ยัง maintain อยู่ไหม
- **Least recently updated** — เรียงจากโปรเจกต์ที่นิ่งนานที่สุด

### 281.5 การกรองด้วย Sidebar (Filters)

หน้าผลลัพธ์ค้นหายังมี sidebar ด้านซ้ายให้กรองผลลัพธ์เพิ่มเติมโดยไม่ต้องพิมพ์ syntax เอง เช่น กรองตาม **Language**, **Owner (org/user)**, **Repository**, **Updated (ช่วงเวลาที่อัปเดต)** ซึ่งจริง ๆ แล้วเมื่อคุณคลิกตัวกรองพวกนี้ GitHub จะแปลงมันเป็น search syntax ต่อท้าย query ให้อัตโนมัติ — นี่คือจุดเชื่อมโยงไปสู่ Step 282 ที่เราจะเรียนรู้ syntax เหล่านี้โดยตรง เพื่อพิมพ์เองได้เร็วกว่าคลิกทีละอัน

### 281.6 ทำไมต้องเรียนรู้การค้นหาให้ลึก

เหตุผลที่ Part นี้อยู่ในเฟส 3 (การใช้งาน GitHub อย่างมืออาชีพ) เพราะการค้นหาเก่งมีผลโดยตรงต่อการทำงานจริง:

1. **ลดเวลาแก้บั๊ก** — แทนที่จะเขียนคำถามใน Google เฉย ๆ การค้นหาตรงใน issue ของ GitHub ทำให้เจอคำตอบจาก maintainer ตัวจริงเร็วกว่า
2. **หา pattern การเขียนโค้ดที่ถูกต้อง** — ก่อน implement อะไรใหม่ ลองค้นหาว่าโปรเจกต์ดัง ๆ เขาเขียนแบบไหน
3. **ประเมิน dependency ก่อนติดตั้ง** — ก่อนจะ `npm install` อะไรสักตัว ควรรู้วิธีเช็กว่ามันน่าเชื่อถือแค่ไหน (รายละเอียดเต็มใน Step 289)
4. **Contribute ให้ Open Source ได้ตรงจุด** — การหา "good first issue" ที่เหมาะกับระดับตัวเองต้องอาศัย search filter ที่แม่นยำ (จะใช้จริงใน Part 30)

---

## Step 282: Search syntax ขั้นสูง (`in:name`, `language:python`, `stars:>1000`, `size:>1000`)

นี่คือหัวใจสำคัญที่สุดของ Part นี้ GitHub Search รองรับ **qualifier (ตัวระบุเงื่อนไข)** จำนวนมาก เขียนในรูปแบบ `qualifier:value` ต่อท้ายหรือปนกับคำค้นหาปกติได้เลย ยิ่งรู้ syntax เยอะเท่าไหร่ ยิ่งกรองผลลัพธ์ได้แม่นเท่านั้น

### 282.1 หลักการเขียน Query พื้นฐาน

```
<คำค้นหาทั่วไป> <qualifier1>:<value1> <qualifier2>:<value2> ...
```

- ค่าไม่มีช่องว่างพิมพ์ตรง ๆ ได้เลย เช่น `language:python`
- ค่าที่มีช่องว่าง ต้องครอบด้วยเครื่องหมายคำพูด เช่น `"task manager"`
- ใส่ `-` ข้างหน้า qualifier เพื่อ **ยกเว้น (NOT)** เช่น `-language:javascript`
- ใช้ `OR` (ตัวพิมพ์ใหญ่) เพื่อค้นหาแบบ "อย่างใดอย่างหนึ่ง" เช่น `language:python OR language:go`
- Default ระหว่าง qualifier หลายตัวคือ **AND** เสมอ (ต้องตรงทุกเงื่อนไข)

### 282.2 `in:` — ระบุว่าคำค้นหาต้องอยู่ตรงไหน

`in:` บอก GitHub ว่าให้ค้นหาคำนั้นเฉพาะในส่วนไหนของ repository/ไฟล์

**สำหรับค้นหา repository:**

| Syntax | ความหมาย |
|---|---|
| `in:name` | ค้นเฉพาะในชื่อ repo |
| `in:description` | ค้นเฉพาะในคำอธิบาย repo |
| `in:topics` | ค้นเฉพาะใน topics ที่ repo ติดไว้ |
| `in:readme` | ค้นเฉพาะในเนื้อหา README |

ตัวอย่าง:

```
task manager in:name
```

จะได้ repo ที่มีคำว่า "task" และ "manager" อยู่ **ในชื่อ repo เท่านั้น** ไม่นับที่เจอใน README เฉย ๆ ทำให้ผลลัพธ์ตรงประเด็นกว่าการค้นหาแบบกว้าง ๆ มาก

**สำหรับค้นหาโค้ด:**

| Syntax | ความหมาย |
|---|---|
| `in:file` | ค้นในเนื้อหาไฟล์ (ค่าเริ่มต้น) |
| `in:path` | ค้นในชื่อไฟล์/path |

ตัวอย่าง:

```
config in:path extension:yaml
```

หาไฟล์ที่ path มีคำว่า "config" และเป็นไฟล์นามสกุล `.yaml`

### 282.3 `language:` — กรองตามภาษาโปรแกรม

qualifier นี้ใช้ได้ทั้งกับการค้นหา repository และ code

```
language:python
```

```
web scraper language:python
```

```
in:readme cli tool language:rust
```

**เทคนิค:** ชื่อภาษาต้องตรงกับที่ GitHub รู้จัก (อ้างอิงจาก [Linguist](https://github.com/github-linguist/linguist)) เช่น `language:c++`, `language:c#`, `language:typescript`, `language:jupyter-notebook` — ถ้าพิมพ์ผิดหรือใช้ชื่อที่ไม่มีในระบบ ผลลัพธ์อาจว่างเปล่าหรือไม่ตรงตามคาด

### 282.4 `stars:` — กรองตามจำนวนดาว

ใช้ตัวดำเนินการเปรียบเทียบได้ครบ: `>`, `>=`, `<`, `<=`, `..` (ช่วงระหว่างสองค่า)

| Syntax | ความหมาย |
|---|---|
| `stars:>1000` | มากกว่า 1,000 ดาว |
| `stars:>=500` | มากกว่าหรือเท่ากับ 500 ดาว |
| `stars:<100` | น้อยกว่า 100 ดาว |
| `stars:10..50` | ระหว่าง 10 ถึง 50 ดาว (รวมทั้งสองค่า) |

ตัวอย่างการใช้งานจริง — หาไลบรารี state management ของ React ที่ได้รับความนิยมสูง:

```
state management language:typescript stars:>1000
```

ตัวอย่างการหา "hidden gem" — โปรเจกต์ดีแต่ยังไม่มีใครรู้จักเยอะ:

```
markdown editor stars:10..100 pushed:>2026-06-01
```

### 282.5 `size:` — กรองตามขนาดของ repository

`size:` วัดหน่วยเป็น **กิโลไบต์ (KB)** ของขนาด repository (ไม่รวมไฟล์ที่ถูก .gitignore)

```
size:>1000
```

หมายถึง repo ที่มีขนาดใหญ่กว่า 1,000 KB (คือมากกว่า ~1 MB) ขึ้นไป มีประโยชน์เวลาต้องการกรองโปรเจกต์ demo/tutorial เล็ก ๆ ออก แล้วเน้นหาโปรเจกต์ที่มีขนาดใหญ่พอจะเป็น production-grade

```
size:<50
```

กลับกัน หากอยากหา "ตัวอย่างโค้ดสั้น ๆ" หรือ boilerplate ขนาดเล็กเพื่อศึกษาไว จะใช้ค่านี้แทน

### 282.6 qualifier สำคัญอื่น ๆ ที่ควรจำ (ตารางรวม)

| Qualifier | ความหมาย | ตัวอย่าง |
|---|---|---|
| `user:` | จำกัด scope เฉพาะบัญชีผู้ใช้คนเดียว | `user:torvalds` |
| `org:` | จำกัด scope เฉพาะ organization | `org:facebook` |
| `repo:` | จำกัด scope เฉพาะ repo เดียว (รูปแบบ `owner/repo`) | `repo:facebook/react` |
| `forks:` | จำนวน fork | `forks:>500` |
| `followers:` | จำนวนผู้ติดตาม (ใช้กับ user) | `followers:>1000` |
| `created:` | วันที่สร้าง | `created:>2025-01-01` |
| `pushed:` | วันที่ push ล่าสุด (ใช้เช็กว่ายัง active ไหม) | `pushed:>2026-01-01` |
| `license:` | สัญญาอนุญาต | `license:mit` |
| `archived:` | เป็น repo ที่ถูก archive แล้วหรือไม่ | `archived:false` |
| `mirror:` | เป็น mirror repo หรือไม่ | `mirror:false` |
| `is:public` / `is:private` | ระดับการมองเห็น (private ต้องมีสิทธิ์เข้าถึง) | `is:public` |
| `topic:` | ติด topic นี้ไว้หรือไม่ | `topic:docker` |
| `filename:` | ชื่อไฟล์ตรงตามนี้ (ใช้กับ code search) | `filename:package.json` |
| `extension:` | นามสกุลไฟล์ (ใช้กับ code search) | `extension:go` |
| `path:` | path ของไฟล์ต้องตรงกับรูปแบบนี้ | `path:src/components` |

### 282.7 ตัวอย่าง query ที่ผสมหลาย qualifier เข้าด้วยกัน

**หา library CSS-in-JS ยอดนิยมสำหรับ React ที่ยัง maintain อยู่:**

```
css-in-js language:typescript stars:>2000 pushed:>2026-01-01 archived:false
```

**หาโปรเจกต์ Rust ขนาดใหญ่ที่ใช้ license แบบ MIT เท่านั้น:**

```
language:rust license:mit size:>5000 stars:>500
```

**หา repo ของ Google บน GitHub ที่เกี่ยวกับ Kubernetes:**

```
kubernetes org:google in:name,description
```

(สังเกตว่า `in:` รับหลายค่าคั่นด้วย comma ได้ในกรณีของการค้นหา repository)

**หา repository ที่สร้างในปีนี้และกำลังมาแรง:**

```
machine learning created:>2026-01-01 stars:>200
```

### 282.8 เทคนิคการใช้เครื่องหมายคำพูดและ exact match

ถ้าต้องการให้ค้นหาคำที่ติดกันแบบ **วลีที่ตรงเป๊ะ** (exact phrase) ให้ใช้เครื่องหมายคำพูดครอบ:

```
"rate limiter" language:go
```

ผลลัพธ์จะต่างจากการพิมพ์ `rate limiter language:go` แบบไม่มีคำพูด เพราะแบบไม่มีคำพูด GitHub อาจตีความว่าเป็นคำแยกกันสองคำ (rate, limiter) ที่ไม่จำเป็นต้องติดกัน

---

## Step 283: ค้นหาโค้ดเฉพาะ (code search) พร้อมตัวอย่างการค้นหา function/pattern เฉพาะ

Code Search คือฟีเจอร์ที่ทรงพลังที่สุดอย่างหนึ่งของ GitHub เพราะมันให้คุณค้นหา **เนื้อหาจริงในไฟล์โค้ด** ของ repository public (และ private ที่คุณมีสิทธิ์) หลายร้อยล้าน repo ทั่วโลก

### 283.1 ประวัติย่อและสถานะปัจจุบันของ Code Search

GitHub เคยมี code search แบบเก่าที่จำกัดความสามารถมาก (ค้นได้แค่ repo ที่มี star มากกว่าค่าหนึ่ง และรองรับ regex ได้จำกัด) ต่อมา GitHub เปิดตัว **GitHub Code Search (ใหม่)** ที่สร้างจาก index engine ตัวใหม่ทั้งหมด ทำให้:

- ค้นหาได้ครอบคลุมโค้ด public เกือบทั้งหมดบน GitHub โดยไม่จำกัดจำนวน star ขั้นต่ำ
- รองรับ **regular expression (regex)** แบบเต็มรูปแบบ
- รองรับ syntax เฉพาะทาง เช่น ค้นหา symbol (ชื่อฟังก์ชัน, class) ได้แม่นยำกว่าการค้นหาข้อความทั่วไป
- แสดงผลพร้อม syntax highlighting และลิงก์ตรงไปยังบรรทัดที่เจอ

เข้าถึงได้ที่แท็บ **Code** ในหน้าผลลัพธ์ค้นหา หรือเข้าตรง ๆ ที่ `github.com/search?q=...&type=code`

### 283.2 ค้นหา function เฉพาะเจาะจง

สมมติต้องการดูว่าโปรเจกต์อื่นเขียนฟังก์ชัน `debounce` ใน JavaScript แบบไหนบ้าง เพื่อเรียนรู้ pattern ที่หลากหลาย:

```
function debounce language:javascript
```

หรือถ้าอยากเจาะจงมากขึ้นว่าต้องเป็นการ "ประกาศฟังก์ชัน" จริง ๆ ไม่ใช่แค่คำว่า debounce ปรากฏในคอมเมนต์:

```
"function debounce(" language:javascript
```

การใส่วงเล็บเปิดในเครื่องหมายคำพูดช่วยกรองผลลัพธ์ที่เป็น comment หรือ string ธรรมดาออกไปได้เยอะ

### 283.3 ค้นหา pattern การเขียนโค้ดเฉพาะทาง

**ตัวอย่างที่ 1 — หาวิธีตั้งค่า connection pool ของ PostgreSQL ใน Go:**

```
pgxpool.New language:go
```

**ตัวอย่างที่ 2 — หาวิธีเขียน custom hook สำหรับ fetch ข้อมูลใน React:**

```
"useQuery" "useEffect" language:typescript path:hooks
```

**ตัวอย่างที่ 3 — หา error handling pattern ของ Rust ที่ใช้ `?` operator ร่วมกับ custom error type:**

```
impl From<reqwest::Error> for language:rust
```

**ตัวอย่างที่ 4 — หาวิธี implement JWT middleware ใน Express.js:**

```
jwt.verify middleware language:javascript filename:middleware.js
```

### 283.4 การใช้ Regular Expression ใน Code Search

Code Search รองรับโหมด regex โดยพิมพ์คำค้นหาครอบด้วย `/` เช่น:

```
/function\s+debounce\s*\(/ language:javascript
```

regex นี้จะจับคำว่า `function` ตามด้วยช่องว่างหนึ่งตัวขึ้นไป ตามด้วยคำว่า `debounce` แล้วตามด้วยวงเล็บเปิด (มี/ไม่มีช่องว่างก่อนวงเล็บก็ได้) — มีประโยชน์มากเวลาต้องการความแม่นยำสูงในการหา pattern ที่ซับซ้อน เช่น หาการเรียกใช้ API ที่ล้าสมัย (deprecated) เพื่อสำรวจว่ามีโปรเจกต์ไหนยังใช้อยู่บ้าง

```
/api\.deprecated_method\(/
```

### 283.5 ข้อจำกัดสำคัญของ Code Search ที่ต้องรู้

1. **ไฟล์ที่มีขนาดใหญ่เกินไปอาจไม่ถูก index** — ไฟล์ที่มีขนาดเกินขีดจำกัด (โดยทั่วไปหลายร้อย KB ขึ้นไป) จะไม่ถูกนำมา index เพื่อค้นหา
2. **ไฟล์ generated/minified มักถูกกรองออก** — เช่นไฟล์ `.min.js`, ไฟล์ที่ auto-generate จาก build tool อาจไม่ถูกจัดลำดับความสำคัญในการ index
3. **ต้อง login ก่อนใช้งาน** — Code Search เวอร์ชันใหม่กำหนดให้ต้อง sign in เข้า GitHub ก่อนถึงจะใช้ค้นหาโค้ดได้เต็มรูปแบบ
4. **ผลลัพธ์อาจไม่ real-time 100%** — การ index โค้ดใหม่ที่เพิ่ง push อาจใช้เวลาสักครู่กว่าจะปรากฏในผลการค้นหา
5. **บาง repo ปิดกั้นการค้นหาได้** — เจ้าของ repo สามารถตั้งค่าไม่ให้เนื้อหาถูกนำไป index สำหรับ code search ได้ในบางกรณี

### 283.6 เทคนิคผสม `path:` และ `filename:` เพื่อความแม่นยำ

การผสม `path:` เข้ากับคำค้นหาช่วยจำกัด scope ให้แคบลงมาก เช่น การหาตัวอย่างการตั้งค่า GitHub Actions:

```
"runs-on: ubuntu-latest" path:.github/workflows
```

หรือหาตัวอย่างไฟล์ Dockerfile ที่ multi-stage build สำหรับ Go:

```
"FROM golang" "FROM scratch" filename:Dockerfile
```

### 283.7 กรณีการใช้งานจริงในชีวิตประจำวันของนักพัฒนา

- **เรียนรู้วิธีใช้ library ที่เอกสารไม่ครบ** — ค้นหาชื่อฟังก์ชันของ library นั้นตรง ๆ ใน code search จะเจอตัวอย่างการใช้งานจริงจากโปรเจกต์อื่นนับพัน
- **ตรวจสอบว่าโค้ดของตัวเองหลุดไปที่ public repo ไหม** — ค้นหา string เฉพาะ เช่น internal API key pattern บางส่วน (ไม่ควรค้นหา secret จริงเพื่อความปลอดภัย แต่ใช้ pattern ทั่วไปตรวจสอบได้)
- **หาตัวอย่าง configuration file** — เช่น `.eslintrc`, `tsconfig.json` จากโปรเจกต์ที่มีมาตรฐานสูง
- **สำรวจว่ามีใครใช้ deprecated API ของ library ที่เรา maintain อยู่บ้าง** — เพื่อประเมินผลกระทบก่อนจะ breaking change

---

## Step 284: การค้นหา issue/PR ด้วย filter ซับซ้อน (`is:pr is:merged review:approved`, `is:issue is:open label:bug`)

การค้นหา Issue และ Pull Request มี qualifier เฉพาะทางจำนวนมาก ที่ทำให้กรองสถานะการทำงานร่วมกันของทีมได้ละเอียดมาก

### 284.1 พื้นฐาน `is:` สำหรับแยกประเภท

| Syntax | ความหมาย |
|---|---|
| `is:issue` | เฉพาะ issue เท่านั้น |
| `is:pr` | เฉพาะ pull request เท่านั้น |
| `is:open` | สถานะยังเปิดอยู่ |
| `is:closed` | สถานะปิดแล้ว |
| `is:merged` | PR ที่ถูก merge แล้ว (ใช้กับ `is:pr` เท่านั้น) |
| `is:unmerged` | PR ที่ปิดโดยไม่ได้ merge |
| `is:draft` | PR ที่อยู่ในสถานะ draft |
| `is:locked` | thread ที่ถูก lock การสนทนา |

### 284.2 ตัวอย่าง: `is:pr is:merged review:approved`

Query นี้มีประโยชน์มากตอนต้องการดูว่า PR ที่ถูก approve โดย reviewer แล้วมีลักษณะอย่างไร (เช่น ตอนศึกษาว่า PR ที่ผ่านมาตรฐานการรีวิวของโปรเจกต์ดัง ๆ หน้าตาเป็นอย่างไร):

```
repo:facebook/react is:pr is:merged review:approved
```

**อธิบาย qualifier `review:`:**

| Syntax | ความหมาย |
|---|---|
| `review:none` | ยังไม่มีใคร review |
| `review:required` | ต้องมี review ก่อนถึงจะ merge ได้ (ตาม branch protection) |
| `review:approved` | มีการ approve แล้วอย่างน้อยหนึ่งครั้ง |
| `review:changes_requested` | มีการขอให้แก้ไข (request changes) |

### 284.3 ตัวอย่าง: `is:issue is:open label:bug`

ใช้หา issue ที่ยังเปิดอยู่และติด label ว่า bug ซึ่งเป็นจุดเริ่มต้นที่ดีมากสำหรับคนที่อยากช่วย contribute แก้บั๊กให้โปรเจกต์:

```
repo:vuejs/core is:issue is:open label:bug
```

สามารถระบุ label หลายอันพร้อมกันได้ (จะทำงานแบบ AND คือต้องมีครบทุก label):

```
is:issue is:open label:bug label:"good first issue"
```

**หมายเหตุ:** ถ้าชื่อ label มีช่องว่าง (เช่น `good first issue`) ต้องครอบด้วยเครื่องหมายคำพูด

### 284.4 qualifier อื่น ๆ สำหรับ Issue/PR ที่ควรรู้จัก

| Qualifier | ความหมาย | ตัวอย่าง |
|---|---|---|
| `author:` | ผู้เปิด issue/PR | `author:octocat` |
| `assignee:` | ผู้ถูก assign ให้รับผิดชอบ | `assignee:octocat` |
| `mentions:` | มีการ mention ผู้ใช้นี้ในเนื้อหา | `mentions:octocat` |
| `commenter:` | ผู้ที่เคยคอมเมนต์ใน thread นี้ | `commenter:octocat` |
| `involves:` | เกี่ยวข้องในรูปแบบใดก็ได้ (author, assignee, mention, commenter) | `involves:octocat` |
| `team:` | มอบหมายให้ทีมนี้ (ต้องเป็น org) | `team:myorg/backend` |
| `milestone:` | อยู่ใน milestone ที่ระบุ | `milestone:"v2.0"` |
| `project:` | อยู่ใน GitHub Project board ที่ระบุ | `project:myorg/1` |
| `comments:` | จำนวนคอมเมนต์ | `comments:>10` |
| `no:label` | ไม่มี label ใด ๆ เลย | `is:issue no:label` |
| `no:assignee` | ยังไม่มีใครรับผิดชอบ | `is:issue no:assignee` |
| `no:milestone` | ยังไม่ถูกจัดเข้า milestone | `is:issue no:milestone` |
| `linked:pr` | issue ที่มี PR เชื่อมโยงอยู่แล้ว | `is:issue linked:pr` |
| `head:` | ชื่อ branch ต้นทางของ PR | `is:pr head:feature/login` |
| `base:` | ชื่อ branch ปลายทางของ PR | `is:pr base:main` |
| `draft:` | เป็น draft หรือไม่ (true/false) | `is:pr draft:true` |
| `status:` | สถานะของ check/CI (pending, success, failure) | `is:pr status:failure` |

### 284.5 ตัวอย่าง query ระดับมืออาชีพที่ใช้จริงในการทำงาน

**หา PR ของตัวเองที่ยัง pending review ค้างอยู่ ในทุก repo ที่เกี่ยวข้อง:**

```
is:pr is:open review-requested:@me
```

**หา issue ที่ไม่มีใครรับผิดชอบเลย และเปิดมานานแล้ว เหมาะจะหยิบมาทำ:**

```
repo:kubernetes/kubernetes is:issue is:open no:assignee label:"help wanted" sort:created-asc
```

**หา PR ที่ CI พังอยู่ในโปรเจกต์ตัวเอง เพื่อไปตามแก้:**

```
org:mycompany is:pr is:open status:failure
```

**หา issue ที่เกี่ยวกับ security และยังไม่ถูกปิด:**

```
is:issue is:open label:security in:title vulnerability
```

**หา PR ที่ merge เข้า main ในเดือนที่ผ่านมา เพื่อทำ release note:**

```
repo:myorg/myrepo is:pr is:merged merged:>2026-08-26 base:main
```

### 284.6 การบันทึก query ที่ใช้บ่อยเป็น Saved Search

GitHub อนุญาตให้บันทึก query การค้นหาที่ใช้บ่อย ๆ ไว้เป็น **Saved search** เพื่อเรียกกลับมาใช้ซ้ำได้เร็วโดยไม่ต้องพิมพ์ใหม่ทุกครั้ง เข้าไปที่หน้าผลการค้นหา แล้วมองหาปุ่ม "Save this search" (มักอยู่แถบด้านบนของผลลัพธ์) ตั้งชื่อที่จำง่าย เช่น "PR ของฉันที่รอ review" แล้วครั้งต่อไปสามารถเข้าถึงได้จากเมนู search อย่างรวดเร็ว

---

## Step 285: Advanced search UI ของ GitHub (github.com/search/advanced)

สำหรับคนที่ยังไม่คุ้นกับการพิมพ์ syntax เอง หรือต้องการเช็กว่า qualifier ที่จำได้ยังถูกต้องอยู่ไหม GitHub มีหน้า UI สำเร็จรูปสำหรับสร้าง query แบบไม่ต้องจำ syntax ทั้งหมดด้วยตัวเอง

### 285.1 เข้าถึงหน้า Advanced Search

เข้าได้โดยตรงที่:

```
https://github.com/search/advanced
```

หรือจากหน้าผลลัพธ์การค้นหาทั่วไป มักมีลิงก์ "Advanced search" ให้กดเข้าไปได้เช่นกัน (ตำแหน่งอาจเปลี่ยนไปตามการอัปเดต UI ของ GitHub แต่ URL ด้านบนใช้เข้าตรงได้เสมอ)

### 285.2 องค์ประกอบของหน้า Advanced Search

หน้านี้แบ่งเป็นสองส่วนหลัก คือ **แบบฟอร์มการค้นหา repository** และ **แบบฟอร์มการค้นหา code** โดยแต่ละส่วนมีช่องกรอกข้อมูลแยกเป็นหมวดชัดเจน เช่น:

**สำหรับค้นหา Repository:**

- **Keywords** — คำค้นหาทั่วไป พร้อมตัวเลือกย่อยว่าต้องมี "all these words", "exact phrase", "any of these words" หรือ "none of these words"
- **Search within** — เลือกว่าให้ค้นในชื่อ, คำอธิบาย, README, topics
- **Repository size** — กรอกช่วงขนาดเป็น KB
- **Number of stars/forks** — กรอกช่วงตัวเลข
- **Created / Updated** — เลือกช่วงวันที่จาก date picker
- **License** — เลือก license จาก dropdown
- **Language** — เลือกภาษาโปรแกรมจาก dropdown
- **Visibility** — public/private
- **Owner** — ระบุ user หรือ org

**สำหรับค้นหา Code:**

- **In file** — คำที่ต้องอยู่ในเนื้อหาไฟล์
- **In file/path** — จำกัด path
- **Owned by** — เจ้าของ repo
- **Filename** — ชื่อไฟล์
- **Extension** — นามสกุลไฟล์
- **Size** — ขนาดไฟล์

### 285.3 ทำไม Advanced Search UI ยังมีประโยชน์ทั้งที่รู้ syntax แล้ว

แม้ผู้ใช้ที่ชำนาญมักพิมพ์ query เองโดยตรงในช่องค้นหา แต่ Advanced Search UI ยังมีประโยชน์ในสถานการณ์เหล่านี้:

1. **เรียนรู้ qualifier ใหม่ที่ยังไม่เคยรู้จัก** — ฟอร์มจะแสดง qualifier ที่มีอยู่ทั้งหมดให้เห็นเป็นระบบ
2. **ป้องกันการพิมพ์ syntax ผิด** — โดยเฉพาะรูปแบบวันที่ (ต้องเป็น `YYYY-MM-DD`) ซึ่งฟอร์มมี date picker ช่วยไม่ให้พิมพ์ผิด format
3. **สร้าง query ที่ซับซ้อนมาก ๆ โดยไม่งง** — เมื่อต้องใส่เงื่อนไขพร้อมกันหลายสิบตัว การกรอกฟอร์มทีละช่องช่วยลดโอกาสพลาด
4. **ใช้สอนคนอื่น** — เวลาสอนเพื่อนร่วมทีมที่ยังไม่คุ้น syntax การเปิดหน้า advanced search ให้ดูเป็นภาพเข้าใจง่ายกว่าอธิบาย syntax ปากเปล่า

### 285.4 เมื่อกด Search แล้วเกิดอะไรขึ้น

เมื่อกรอกฟอร์มเสร็จแล้วกดปุ่ม Search ระบบจะแปลงค่าที่กรอกทั้งหมดให้เป็น **query string เดียว** (เหมือนที่เราพิมพ์เองใน Step 282-284) แล้วนำไปค้นหาต่อในหน้าผลลัพธ์ปกติ พูดง่าย ๆ Advanced Search UI คือ "ตัวช่วยสร้าง query" ไม่ใช่ระบบค้นหาที่แยกต่างหาก — ผลลัพธ์สุดท้ายที่ได้จะเหมือนกันเป๊ะกับการพิมพ์ syntax เองถ้าใส่เงื่อนไขตรงกัน

**เคล็ดลับ:** ลองกรอกฟอร์มแล้วสังเกต URL ที่ได้หลังกด Search จะช่วยให้คุณเรียนรู้ syntax ไปในตัวโดยอัตโนมัติ เพราะ URL จะแสดง query string เต็ม ๆ ให้เห็นว่าแต่ละช่องที่กรอกแปลงเป็น syntax แบบไหน

---

## Step 286: การใช้ Topics ค้นหาโปรเจกต์ตามหมวดหมู่ (`topic:machine-learning`)

### 286.1 Topics คืออะไร

**Topics** คือ tag ที่เจ้าของ repository ใส่ไว้เพื่อจัดหมวดหมู่โปรเจกต์ของตัวเอง เช่น repo เกี่ยวกับ deep learning อาจติด topics ไว้ว่า `machine-learning`, `deep-learning`, `pytorch`, `neural-network` เป็นต้น การติด topics ช่วยให้คนอื่นค้นเจอโปรเจกต์ของคุณง่ายขึ้นมาก แม้ชื่อ repo หรือคำอธิบายจะไม่มีคำนั้นตรง ๆ ก็ตาม

### 286.2 วิธีดู Topics ของ repo

Topics จะแสดงเป็น "ป้าย" เล็ก ๆ สีฟ้าอยู่ใต้ชื่อ repository บนหน้าหลักของ repo นั้น เช่น เปิด repo ของ TensorFlow จะเห็นป้ายอย่าง `machine-learning`, `deep-learning`, `tensorflow`, `python` เรียงกันอยู่

### 286.3 การค้นหาด้วย qualifier `topic:`

```
topic:machine-learning
```

จะได้ repo ทั้งหมดที่ติด topic นี้ไว้ เรียงตาม relevance/star ตามค่า sort ที่เลือก

**ผสมกับ qualifier อื่นได้ตามปกติ:**

```
topic:machine-learning language:python stars:>5000
```

```
topic:react-native topic:typescript
```

(การใส่ `topic:` สองครั้งหมายถึงต้องมีทั้งสอง topic พร้อมกัน)

### 286.4 หน้า Topics โดยเฉพาะ (github.com/topics)

นอกจากค้นหาผ่านช่อง search แล้ว GitHub ยังมีหน้ารวม Topics โดยเฉพาะที่:

```
https://github.com/topics
```

หน้านี้แสดง topics ยอดนิยมเป็นหมวดหมู่สวยงาม พร้อมคำอธิบายสั้น ๆ ของแต่ละ topic เช่น `github.com/topics/machine-learning` จะแสดง:

- คำอธิบายว่า machine learning คืออะไรโดยย่อ
- Repository เด่น ๆ ที่ติด topic นี้ เรียงตามความนิยม
- ลิงก์ topics ที่เกี่ยวข้อง (related topics) เช่น topic `machine-learning` อาจ link ไปยัง `deep-learning`, `neural-networks`, `data-science`

### 286.5 ทำไม Topics ถึงมีประโยชน์มากกว่าการค้นหาคำทั่วไป

1. **แม่นยำกว่า keyword search ธรรมดา** — เพราะเจ้าของ repo เป็นคนติด tag เองอย่างตั้งใจ ไม่ใช่การเดาจากคำในคำอธิบาย
2. **ช่วยสำรวจ ecosystem ทั้งวงการได้เร็ว** — เข้าไปที่ topic เดียวก็เห็นภาพรวมของโปรเจกต์เด่น ๆ ในสายนั้นทั้งหมด
3. **หาโปรเจกต์ข้าม-ภาษาโปรแกรมมิ่งได้** — เช่น topic `rest-api` จะรวมโปรเจกต์จากทั้ง Python, Go, Node.js, Java เข้าด้วยกัน ต่างจากการกรองด้วย `language:` ที่จำกัดแค่ภาษาเดียว

### 286.6 การติด Topics ให้ repository ของตัวเอง

ถ้าคุณเป็นเจ้าของ repo และอยากให้คนอื่นค้นเจอโปรเจกต์ผ่าน topics ได้ ทำได้ง่าย ๆ ดังนี้:

1. เข้าไปที่หน้าหลักของ repo
2. คลิกไอคอนรูปเฟือง (⚙) ที่อยู่ข้าง "About" บน sidebar ด้านขวา
3. ในช่อง **Topics** พิมพ์คำที่เกี่ยวข้อง แล้วกด Enter ทีละคำ (ใส่ได้หลายคำ)
4. กด **Save changes**

**คำแนะนำ:** ควรเลือก topics ที่เป็นที่นิยมอยู่แล้วในวงการ (ตรวจสอบผ่าน `github.com/topics`) แทนการตั้งคำเฉพาะตัวที่ไม่มีใครค้นหา เพราะจะทำให้คนเจอโปรเจกต์ของคุณยากขึ้นแทนที่จะง่ายขึ้น

---

## Step 287: Trending repositories — หาโปรเจกต์ดังประจำวัน/สัปดาห์ (github.com/trending)

### 287.1 หน้า Trending คืออะไร

**GitHub Trending** เป็นหน้าที่รวบรวม repository ที่ได้รับความสนใจสูง (วัดจากจำนวน star ที่เพิ่มขึ้นในช่วงเวลาสั้น ๆ) เข้าถึงได้ที่:

```
https://github.com/trending
```

ต่างจากการเรียงตาม "Most stars" ในผลการค้นหาทั่วไป (ซึ่งจะได้โปรเจกต์เก่าแก่ที่สะสม star มานานหลายปี) หน้า Trending จะแสดง **โปรเจกต์ที่กำลังมาแรงในช่วงเวลานั้นจริง ๆ** ทำให้เห็นเทรนด์ปัจจุบันของวงการได้ชัดเจนกว่ามาก

### 287.2 การเลือกช่วงเวลา (Date Range)

หน้า Trending มีตัวเลือกช่วงเวลาให้ดูสามแบบ:

| ช่วงเวลา | เหมาะกับ |
|---|---|
| **Today** | ดูว่าวันนี้มีอะไรฮอตในวงการ เหมาะกับการติดตามข่าวสารรายวัน |
| **This week** | ดูภาพรวมที่กว้างขึ้น ตัดสัญญาณรบกวนจากกระแสชั่วครู่ (viral ข้ามวันเดียว) ออกไปบ้าง |
| **This month** | ดูเทรนด์ระยะยาวขึ้น เหมาะกับการวางแผนเรียนรู้เทคโนโลยีใหม่ที่กำลังเป็นที่นิยมจริงจัง |

### 287.3 การกรองตามภาษาโปรแกรม

หน้า Trending มี dropdown ให้เลือกกรองเฉพาะภาษาที่สนใจ เช่น:

```
https://github.com/trending/python?since=weekly
```

```
https://github.com/trending/typescript?since=daily
```

```
https://github.com/trending/rust?since=monthly
```

การกรองแบบนี้มีประโยชน์มากสำหรับคนที่ต้องการติดตามเฉพาะ ecosystem ของภาษาที่ตัวเองใช้งานอยู่ เช่น นักพัฒนา Python ที่อยากรู้ว่าตอนนี้มี library ตัวไหนกำลังมาแรงในวงการ data science

### 287.4 ข้อมูลที่แสดงในแต่ละรายการของ Trending

แต่ละรายการใน Trending จะแสดง:

- ชื่อ repo และเจ้าของ
- คำอธิบายสั้น ๆ
- ภาษาโปรแกรมหลักที่ใช้ (พร้อมสีบ่งชี้ตามมาตรฐานของ GitHub)
- จำนวน star ทั้งหมด
- จำนวน fork
- **"X stars today/this week"** — ตัวเลขที่สำคัญที่สุดของหน้านี้ คือจำนวน star ที่เพิ่มขึ้นในช่วงเวลาที่เลือก ซึ่งเป็นตัวชี้วัดว่ากำลัง "แรง" แค่ไหนจริง ๆ
- รายชื่อผู้ร่วมพัฒนา (contributor) ที่โดดเด่นบางส่วน พร้อมรูปโปรไฟล์

### 287.5 Trending Developers

นอกจาก Trending Repositories แล้ว ยังมีหน้า **Trending Developers** ที่แสดงนักพัฒนาที่มีผลงานโดดเด่นในช่วงเวลานั้น เข้าถึงได้ที่:

```
https://github.com/trending/developers
```

หน้านี้มีประโยชน์สำหรับการหาคนเก่ง ๆ มาติดตาม (follow) เพื่อเรียนรู้แนวทางการเขียนโค้ดหรือแนวคิดของพวกเขา รวมถึงเป็นแหล่งค้นหา "ตัวจริง" ในวงการที่บริษัทจำนวนมากใช้เป็นช่องทาง sourcing (คัดสรร) ผู้สมัครงานด้านเทคนิคด้วย

### 287.6 การใช้ Trending อย่างมีวิจารณญาณ

Trending ไม่ได้แปลว่าโปรเจกต์นั้นจะดีเสมอไป ควรพิจารณาร่วมกับปัจจัยอื่น ๆ เสมอ:

1. **โปรเจกต์ใหม่ที่ viral อาจยังไม่เสถียร** — star พุ่งเร็วเพราะกระแสในโซเชียลมีเดีย ไม่ได้แปลว่าโค้ดมีคุณภาพสูงหรือ production-ready
2. **บาง repo ติด Trending เพราะเป็นทรัพยากรการเรียนรู้ ไม่ใช่ tool ที่ใช้งานจริง** — เช่น repo รวม roadmap, cheatsheet, awesome-list ต่าง ๆ
3. **ควรตรวจสอบคุณภาพเพิ่มเติมเสมอ** ด้วยเทคนิคที่จะอธิบายใน Step 289 ก่อนจะนำไปใช้งานจริงในโปรเจกต์สำคัญ

---

## Step 288: GitHub Explore และการติดตาม (follow) ผู้ใช้/organization

### 288.1 GitHub Explore คืออะไร

**GitHub Explore** เป็นหน้ารวมศูนย์สำหรับการค้นพบสิ่งใหม่ ๆ บน GitHub ที่ปรับแต่ง (personalize) ให้เหมาะกับความสนใจของแต่ละบัญชี เข้าถึงได้ที่:

```
https://github.com/explore
```

เนื้อหาในหน้านี้จะพิจารณาจากพฤติกรรมของคุณ เช่น repo ที่คุณ star ไว้, ภาษาที่คุณเขียนบ่อย, คนที่คุณ follow อยู่ แล้วแนะนำโปรเจกต์ใหม่ ๆ ที่น่าจะถูกใจ

### 288.2 องค์ประกอบหลักของหน้า Explore

- **Recommended for you** — repo ที่ GitHub คิดว่าคุณน่าจะสนใจ โดยพิจารณาจากประวัติการใช้งานของคุณ
- **Trending repositories** — ลิงก์ย่อยไปยังหน้า Trending ที่อธิบายใน Step 287
- **Collections** — ชุดโปรเจกต์ที่ GitHub หรือทีมงานคัดสรรมาตามธีม เช่น "Great for beginners", "Deep learning", "Game development" ซึ่งมักมีคำอธิบายประกอบว่าทำไมโปรเจกต์เหล่านั้นถึงถูกเลือกมา
- **Topics ยอดนิยม** — บล็อกแสดง topics ที่มีคนสนใจเยอะในช่วงนี้

### 288.3 การ Follow ผู้ใช้ (User)

การ Follow ผู้ใช้คนหนึ่งบน GitHub ทำได้โดยเข้าไปที่หน้าโปรไฟล์ของเขา แล้วกดปุ่ม **Follow** ที่อยู่ใต้รูปโปรไฟล์

**ผลลัพธ์ของการ Follow:**

1. กิจกรรมสาธารณะของคนที่คุณ follow (เช่น การสร้าง repo ใหม่, การ star repo, การเปิด PR) จะปรากฏใน **News Feed** ของคุณที่หน้า `github.com` (หน้า dashboard หลังล็อกอิน)
2. คุณจะเห็นจำนวน follower/following สะสมในโปรไฟล์ของทั้งสองฝ่าย
3. ไม่มีการแจ้งเตือนแบบ real-time ไปหาคนที่ถูก follow (ต่างจาก social media ทั่วไปที่มักแจ้งเตือนทันที) เพียงแต่เขาจะเห็นในหน้า follower list ของตัวเอง

### 288.4 การติดตาม Organization

Organization บน GitHub (เช่น `facebook`, `google`, `microsoft`) ไม่มีปุ่ม "Follow" แบบเดียวกับ user แต่มีวิธีติดตามความเคลื่อนไหวได้หลายทาง:

1. **Watch repository เฉพาะตัวที่สนใจ** — กดปุ่ม **Watch** ที่หน้า repo แล้วเลือกระดับการแจ้งเตือน:
   - **Participating and @mentions** — แจ้งเตือนเฉพาะเมื่อมีคน mention คุณหรือคุณมีส่วนร่วมใน thread นั้น
   - **All Activity** — แจ้งเตือนทุกความเคลื่อนไหว (issue, PR, comment, release ใหม่)
   - **Custom** — เลือกเองว่าอยากรับแจ้งเตือนเรื่องอะไรบ้าง เช่น เฉพาะ Releases, เฉพาะ Discussions
   - **Ignore** — ปิดแจ้งเตือนทั้งหมดสำหรับ repo นี้

2. **ดูหน้า People ของ organization** — หลาย org เปิดให้เห็นสมาชิกทีมสาธารณะที่หน้า `github.com/orgs/<org>/people` แล้ว follow สมาชิกคนสำคัญเป็นรายบุคคลแทน

3. **ติดตามผ่าน RSS feed** — GitHub เปิดให้ subscribe RSS ของ release หรือ commit ของ repo ได้ เช่น `https://github.com/<owner>/<repo>/releases.atom` เหมาะสำหรับคนที่อยากติดตามผ่านเครื่องมืออ่าน RSS แทนการเข้าเว็บทุกวัน

### 288.5 การใช้ News Feed อย่างมีประสิทธิภาพ

หน้า dashboard หลัก (`github.com` หลัง login) จะรวมกิจกรรมจากทั้งคนที่คุณ follow และ repo ที่คุณ watch ไว้เป็น timeline เดียว ข้อแนะนำในการใช้งาน:

- **Follow เฉพาะคนที่งานของเขาตรงกับสายที่คุณสนใจจริง ๆ** — ถ้า follow เยอะเกินไป feed จะรกจนหาข้อมูลสำคัญยาก
- **ใช้ Watch แบบ Custom กับ repo ที่ใช้งานประจำ** — เพื่อรับแจ้งเตือนเฉพาะ Release ใหม่ โดยไม่ต้องรก inbox ด้วย issue comment ทุกอัน
- **Unwatch repo ที่ไม่เกี่ยวข้องแล้ว** — โดยเฉพาะ repo ที่เคย fork หรือเคย contribute ไปนานแล้วแต่ตอนนี้ไม่ได้ใช้งานต่อ

---

## Step 289: การใช้ search ช่วยหา library/dependency ที่น่าเชื่อถือ

นี่คือหนึ่งในทักษะที่สำคัญที่สุดในการทำงานจริง เพราะทุกครั้งที่คุณเพิ่ม dependency ใหม่เข้าโปรเจกต์ (`npm install`, `pip install`, `go get`, `cargo add`) คุณกำลังนำโค้ดของคนอื่นเข้ามาเป็นส่วนหนึ่งของระบบที่คุณต้องรับผิดชอบ การเลือกไม่ดีอาจนำไปสู่ security vulnerability, ความไม่เสถียร หรือ maintenance ที่ถูกทิ้งร้างกลางทาง

### 289.1 หลักการประเมิน 5 ด้าน

เวลาจะประเมิน library ตัวหนึ่งบน GitHub ก่อนตัดสินใจใช้งาน ให้ตรวจสอบตามหัวข้อเหล่านี้:

#### 1. จำนวน Star (Popularity)

ดูจำนวน star เป็นตัวชี้วัดเบื้องต้นของความนิยม ค้นหาได้ด้วย:

```
repo:owner/name stars:>1000
```

**ข้อควรระวัง:** star ไม่ใช่ทุกอย่าง มี library ที่ star น้อยแต่คุณภาพสูงมาก (เพราะเป็นเรื่องเฉพาะทาง) และมี library star เยอะแต่ทิ้งร้างไปแล้วก็มีเหมือนกัน ต้องดูร่วมกับปัจจัยอื่นเสมอ

#### 2. วันที่ Commit ล่าสุด (Last Commit / Activity)

ใช้ qualifier `pushed:` เพื่อเช็กว่า repo ยังมีการอัปเดตอยู่หรือไม่:

```
repo:owner/name pushed:>2026-06-01
```

หรือเข้าไปดูตรง ๆ ที่หน้า repo ใต้ชื่อไฟล์แต่ละไฟล์จะมีบอกว่า commit ล่าสุดเมื่อไหร่ และที่แถบด้านบนจะมีคำว่า เช่น "X commits" พร้อมวันที่ commit ล่าสุดของทั้ง repo

**สัญญาณอันตราย:** ถ้า repo ไม่มีการ commit มาเกิน 1-2 ปี และมี issue ค้างจำนวนมากไม่ถูกตอบ อาจแปลว่าโปรเจกต์ถูกทิ้งร้าง (abandoned) ควรระวังเป็นพิเศษถ้าเป็น dependency สำคัญของระบบ production

#### 3. License (สัญญาอนุญาต)

ตรวจสอบ license ด้วย qualifier `license:` หรือดูตรง ๆ ที่ sidebar ของหน้า repo (มักมีไอคอนและชื่อ license แสดงอยู่):

```
repo:owner/name license:mit
```

ตัวอย่าง license ที่พบบ่อยและระดับความอนุญาต:

| License | ลักษณะ |
|---|---|
| **MIT** | อนุญาตกว้างมาก ใช้ในเชิงพาณิชย์ได้ แก้ไขได้ แทบไม่มีข้อผูกมัด |
| **Apache 2.0** | คล้าย MIT แต่มีเงื่อนไขเรื่อง patent grant เพิ่มเติม |
| **BSD** | คล้าย MIT อนุญาตกว้าง |
| **GPL / AGPL** | มีข้อผูกมัดแบบ copyleft — ถ้านำโค้ดไปใช้ในโปรเจกต์อื่น อาจต้องเปิดเผยซอร์สโค้ดของโปรเจกต์นั้นด้วยตามเงื่อนไข ต้องอ่านให้ละเอียดก่อนใช้ในโปรเจกต์เชิงพาณิชย์
| **ไม่มี license เลย** | อันตรายที่สุด — ตามหลักกฎหมายลิขสิทธิ์ทั่วไป การไม่มี license ระบุไว้แปลว่า**สงวนสิทธิ์ทั้งหมด** ไม่ได้แปลว่าใช้ได้ฟรี ควรหลีกเลี่ยงการนำไปใช้ในโปรเจกต์จริงจนกว่าเจ้าของจะระบุ license ชัดเจน |

#### 4. จำนวน Open Issues และอัตราการตอบสนอง

เช็กจำนวน issue ที่เปิดค้างอยู่:

```
repo:owner/name is:issue is:open
```

ให้สังเกตสองอย่าง:

- **จำนวน issue เปิดค้างมากผิดปกติเทียบกับขนาดโปรเจกต์** อาจแปลว่า maintainer ตามไม่ทัน
- **วันที่ตอบล่าสุดของ maintainer ใน issue** — ลองเปิด issue ที่เพิ่งสร้างไม่กี่วันที่ผ่านมาดูว่ามีการตอบกลับไหม ถ้า maintainer ยังตอบ issue ใหม่ ๆ อยู่สม่ำเสมอ แปลว่าโปรเจกต์ยัง active และดูแลดี

เปรียบเทียบสัดส่วน issue ที่ปิดกับที่เปิดก็ช่วยได้เช่นกัน (ดูได้จากแถบ Issues ที่หน้า repo ซึ่งจะโชว์ตัวเลข "X Open" และ "Y Closed")

#### 5. จำนวน Contributor และ Bus Factor

ดูที่แถบ **Insights → Contributors** ของ repo เพื่อดูว่ามีคน maintain กี่คน

- ถ้ามี **maintainer เดียว** ความเสี่ยงคือถ้าคนนั้นเลิกดูแล โปรเจกต์อาจหยุดพัฒนาทันที (เรียกว่า "bus factor = 1")
- ถ้ามี **contributor หลากหลายคนสม่ำเสมอ** ความเสี่ยงจะกระจายออกไป โปรเจกต์มีโอกาสอยู่รอดต่อได้แม้คนใดคนหนึ่งหายไป

### 289.2 ตัวอย่าง Query ที่รวมทุกปัจจัยเข้าด้วยกัน

**หา HTTP client library สำหรับ Python ที่น่าเชื่อถือ:**

```
http client language:python stars:>3000 pushed:>2026-01-01 license:mit archived:false
```

**หา component library สำหรับ Vue 3 ที่ยัง active:**

```
vue3 component library language:typescript stars:>1000 pushed:>2025-12-01
```

### 289.3 Checklist สรุปก่อนตัดสินใจติดตั้ง dependency

ก่อนพิมพ์คำสั่ง install จริง ให้ไล่เช็กตามลิสต์นี้:

- [ ] จำนวน star อยู่ในระดับที่เหมาะสมกับความสำคัญของ dependency นี้ในระบบ
- [ ] มี commit ล่าสุดภายในช่วงเวลาที่สมเหตุสมผล (ไม่ทิ้งร้างเกินไป)
- [ ] มี license ที่ชัดเจนและเข้ากันได้กับเงื่อนไขของโปรเจกต์คุณ (โดยเฉพาะถ้าเป็นโปรเจกต์เชิงพาณิชย์)
- [ ] จำนวน open issue ไม่มากผิดปกติ และ maintainer ยังตอบ issue ใหม่อยู่
- [ ] มี contributor มากกว่า 1 คนที่ยัง active (ลด bus factor risk)
- [ ] อ่าน README/documentation แล้วเข้าใจวิธีใช้งานได้ชัดเจน
- [ ] ตรวจสอบ security advisory ของ package นี้ (ถ้ามี ผ่านแท็บ **Security** ของ repo หรือหน้า GitHub Advisory Database)
- [ ] เช็กว่าขนาดของ dependency (bundle size สำหรับ frontend) เหมาะสมกับสิ่งที่ต้องการใช้งานจริง ไม่ใช่ import library ใหญ่มาแค่ใช้ function เดียว

การฝึกใช้ checklist นี้จนเป็นนิสัยจะช่วยลดความเสี่ยงด้าน security และ maintenance ของโปรเจกต์ในระยะยาวได้มาก ซึ่งเป็นทักษะที่ทีมวิศวกรรมระดับมืออาชีพให้ความสำคัญมากในการทำ dependency review ก่อน merge PR ที่เพิ่ม dependency ใหม่เข้ามา

---

## Step 290: แบบฝึกหัด — ฝึกค้นหาโปรเจกต์/โค้ดด้วยเทคนิคขั้นสูงอย่างน้อย 8 สถานการณ์จริง

ถึงเวลาลงมือฝึกจริงแล้ว! ทำตามสถานการณ์ทั้ง 8 ข้อด้านล่างนี้ทีละข้อ โดยเปิดเบราว์เซอร์ไปที่ `github.com` แล้วลองพิมพ์ query ตามที่กำหนด สังเกตผลลัพธ์ที่ได้ และลองปรับแต่ง query ให้แม่นยำขึ้นด้วยตัวเอง

### สถานการณ์ที่ 1: หาไลบรารี validation สำหรับ TypeScript ที่น่าเชื่อถือ

**โจทย์:** คุณกำลังเริ่มโปรเจกต์ backend ใหม่ด้วย TypeScript และต้องการ library สำหรับ validate ข้อมูล input ที่ทั้งได้รับความนิยมสูงและยังมีการดูแลอย่างต่อเนื่อง

**ลองพิมพ์:**

```
schema validation language:typescript stars:>5000 pushed:>2026-01-01
```

**สิ่งที่ต้องฝึกวิเคราะห์:** เปรียบเทียบผลลัพธ์ 3 อันดับแรก ดูจำนวน contributor, license, และวันที่ commit ล่าสุดของแต่ละตัว แล้วเลือกว่าตัวไหนเหมาะกับโปรเจกต์ของคุณมากที่สุด พร้อมให้เหตุผลประกอบ

### สถานการณ์ที่ 2: หาตัวอย่างการเขียน custom React Hook สำหรับจัดการ WebSocket

**โจทย์:** คุณต้องการเรียนรู้ pattern การเขียน hook ที่จัดการ WebSocket connection ใน React จากโปรเจกต์จริง

**ลองพิมพ์ (Code Search):**

```
"useWebSocket" "useCallback" language:typescript path:hooks
```

**สิ่งที่ต้องฝึก:** เปิดดูไฟล์ที่เจอ 2-3 ไฟล์ เปรียบเทียบว่าแต่ละคนจัดการเรื่อง reconnect logic ต่างกันอย่างไร

### สถานการณ์ที่ 3: หา "good first issue" ที่เหมาะกับสายภาษา Python ในโปรเจกต์ดัง

**โจทย์:** คุณอยากเริ่ม contribute ให้ Open Source เป็นครั้งแรก โดยเลือกจากโปรเจกต์ Python ที่มีชื่อเสียง

**ลองพิมพ์:**

```
language:python is:issue is:open label:"good first issue" no:assignee stars:>2000
```

**สิ่งที่ต้องฝึก:** เลือก issue มา 1 อัน อ่านรายละเอียดทั้งหมด แล้วลองเขียนสรุปในใจ (ไม่ต้องส่งจริง) ว่าถ้าจะแก้ปัญหานี้ต้องเริ่มจากตรงไหนของโค้ด (เราจะฝึก contribute จริงใน Part 30)

### สถานการณ์ที่ 4: ตรวจสอบว่า deprecated method ตัวหนึ่งยังมีใครใช้อยู่บ้าง

**โจทย์:** สมมติคุณเป็น maintainer ของ library หนึ่ง กำลังจะ deprecate ฟังก์ชันชื่อ `fetchLegacyData()` และต้องการรู้ว่ามีโปรเจกต์อื่นเรียกใช้ฟังก์ชันนี้อยู่มากแค่ไหนก่อนตัดสินใจ

**ลองพิมพ์ (regex code search):**

```
/fetchLegacyData\(/
```

**สิ่งที่ต้องฝึก:** ลองปรับ regex ให้แม่นยำขึ้น เช่น จำกัดเฉพาะการเรียกที่มี `.then(` ตามหลัง เพื่อกรองเฉพาะรูปแบบการใช้งานที่คาดว่าจะเป็น production code จริง ไม่ใช่แค่คอมเมนต์

### สถานการณ์ที่ 5: หา PR ตัวอย่างที่ผ่านการ approve ในโปรเจกต์มาตรฐานสูง เพื่อศึกษาสไตล์การเขียน commit message

**โจทย์:** คุณอยากรู้ว่าโปรเจกต์อย่าง Kubernetes เขียน PR description และ commit message อย่างไรถึงจะผ่านการรีวิว

**ลองพิมพ์:**

```
repo:kubernetes/kubernetes is:pr is:merged review:approved
```

**สิ่งที่ต้องฝึก:** เปิดดู PR 3 อันแรก สังเกตโครงสร้างของ PR description (มี checklist อะไรบ้าง, อ้างอิง issue อย่างไร) แล้วจดสิ่งที่น่านำไปปรับใช้กับ PR ของตัวเอง

### สถานการณ์ที่ 6: สำรวจโปรเจกต์ Rust ที่กำลังมาแรงในเดือนนี้

**โจทย์:** คุณกำลังศึกษาภาษา Rust และอยากรู้ว่าตอนนี้วงการ Rust กำลังให้ความสนใจโปรเจกต์ประเภทไหนมากที่สุด

**ลองเปิด URL:**

```
https://github.com/trending/rust?since=monthly
```

**สิ่งที่ต้องฝึก:** จดชื่อ 5 โปรเจกต์แรกพร้อมหมวดหมู่ (เช่น CLI tool, web framework, game engine) แล้วเลือก 1 โปรเจกต์ที่น่าสนใจที่สุดมาอ่าน README แบบละเอียด

### สถานการณ์ที่ 7: หา configuration file ตัวอย่างสำหรับตั้งค่า GitHub Actions ที่ deploy ไป AWS

**โจทย์:** คุณกำลังจะตั้งค่า CI/CD pipeline ที่ deploy แอปไป AWS ผ่าน GitHub Actions และต้องการดูตัวอย่างจริงจากโปรเจกต์อื่น

**ลองพิมพ์ (Code Search):**

```
"aws-actions/configure-aws-credentials" path:.github/workflows
```

**สิ่งที่ต้องฝึก:** เปรียบเทียบไฟล์ workflow ที่เจอ 2-3 ไฟล์ ดูว่าแต่ละโปรเจกต์จัดการ secret และ environment variable ต่างกันอย่างไร

### สถานการณ์ที่ 8: ประเมิน dependency ตัวหนึ่งแบบครบวงจรก่อนตัดสินใจใช้จริง

**โจทย์:** ทีมของคุณกำลังพิจารณาจะใช้ library ชื่อสมมติ `awesome-date-lib` (หรือเลือก library จริงที่คุณสนใจจริง ๆ เช่น `dayjs`, `date-fns`) ในโปรเจกต์ production จงประเมินตาม checklist ของ Step 289 ให้ครบทุกข้อ

**ขั้นตอนที่ต้องทำ:**

1. ค้นหา repo นั้นด้วย `repo:owner/name`
2. เช็กจำนวน star และ pattern การเติบโต (ดูจาก Insights → Community Standards หรือ star history ถ้ามีเครื่องมือภายนอกช่วยดู)
3. ค้นหา `pushed:` เพื่อดูวันที่ commit ล่าสุด
4. เปิดดู license ที่ sidebar
5. ค้นหา `is:issue is:open` เพื่อดูจำนวน issue ค้าง
6. เปิดแท็บ Insights → Contributors เพื่อดู bus factor
7. สรุปผลเป็นตารางสั้น ๆ ในใจ (หรือจดใส่ notepad ของตัวเอง) แล้วตัดสินใจว่า "ใช้" หรือ "ไม่ใช้" พร้อมเหตุผลประกอบอย่างน้อย 3 ข้อ

**เป้าหมายสุดท้ายของแบบฝึกหัดนี้:** เมื่อทำครบทั้ง 8 สถานการณ์ คุณควรจะสามารถหยิบ GitHub Search มาใช้แก้ปัญหาจริงในการทำงานได้ทันที ไม่ว่าจะเป็นการหา library, ศึกษา pattern การเขียนโค้ด, หา issue ที่เหมาะจะ contribute หรือประเมินความน่าเชื่อถือของ dependency ก่อนนำเข้าสู่ระบบ production

### Checklist ก่อนไป Part 30

- [ ] ค้นหา repository, code, issue, PR, user ได้คล่องทั้งแบบพื้นฐานและแบบมี qualifier
- [ ] ใช้ syntax `in:`, `language:`, `stars:`, `size:` และ qualifier อื่น ๆ ผสมกันได้อย่างถูกต้อง
- [ ] ใช้ Code Search ค้นหา function/pattern เฉพาะ รวมถึงใช้ regex ค้นหาแบบแม่นยำได้
- [ ] ใช้ filter ของ Issue/PR เช่น `is:pr is:merged review:approved`, `is:issue is:open label:bug` ได้คล่อง
- [ ] รู้จักและใช้งานหน้า Advanced Search UI ได้
- [ ] ใช้ Topics ค้นหาโปรเจกต์ตามหมวดหมู่ และรู้วิธีติด topics ให้ repo ของตัวเอง
- [ ] เข้าใจและใช้งานหน้า Trending เพื่อติดตามเทรนด์วงการได้
- [ ] เข้าใจการ Follow user และการ Watch repository/organization
- [ ] ประเมิน library/dependency ก่อนใช้งานได้ครบทั้ง 5 ด้าน (star, activity, license, issues, contributors)
- [ ] ฝึกค้นหาจริงครบทั้ง 8 สถานการณ์ใน Step 290

---

## สรุป Part 29

ใน Part นี้เราได้เรียนรู้ว่า:

1. GitHub Search แบ่งการค้นหาออกเป็นหลายประเภท (repository, code, issue, PR, user, topics) แต่ละแบบมีจุดประสงค์และวิธีใช้งานต่างกัน
2. Search syntax ขั้นสูงอย่าง `in:`, `language:`, `stars:`, `size:` และ qualifier อื่น ๆ ทำให้กรองผลลัพธ์ได้แม่นยำกว่าการพิมพ์คำค้นหาเปล่า ๆ มาก
3. Code Search เป็นเครื่องมือทรงพลังสำหรับหา pattern การเขียนโค้ดจริงจากทั่วโลก รองรับทั้งการค้นหาข้อความและ regular expression
4. การค้นหา Issue/PR ด้วย filter ซับซ้อนอย่าง `is:pr is:merged review:approved` หรือ `is:issue is:open label:bug` ช่วยให้ทำงานเป็นทีมและ contribute ให้ Open Source ได้อย่างมีประสิทธิภาพ
5. Advanced Search UI ที่ `github.com/search/advanced` เป็นตัวช่วยสร้าง query แบบไม่ต้องจำ syntax ทั้งหมดด้วยตัวเอง
6. Topics และหน้า `github.com/topics` ช่วยค้นหาโปรเจกต์ตามหมวดหมู่ได้แม่นยำกว่าการค้นหาคำทั่วไป
7. หน้า Trending ที่ `github.com/trending` ช่วยติดตามโปรเจกต์ที่กำลังมาแรงในแต่ละวัน/สัปดาห์/เดือน แยกตามภาษาโปรแกรมได้
8. GitHub Explore และการ Follow/Watch ช่วยให้ติดตามความเคลื่อนไหวของผู้ใช้และ organization ที่สนใจได้อย่างเป็นระบบ
9. การประเมิน library/dependency ก่อนนำมาใช้งานจริงต้องพิจารณาครบทั้ง 5 ด้าน คือ ความนิยม, ความ active, license, จำนวน open issue, และจำนวน contributor
10. ทักษะการค้นหาที่ฝึกใน Part นี้เป็นพื้นฐานสำคัญที่จะนำไปใช้จริงในการ contribute ให้โปรเจกต์ Open Source ใน Part ถัดไป

**ต่อไป:** [Part 30: โปรเจกต์ฝึกหัด: Contribute ให้โปรเจกต์ Open Source จริง](./part-030-contribute-open-source-จริง.md)
