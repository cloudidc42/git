# Part 88: Release Management และ Semantic Versioning

> **Step ในหลักสูตรนี้:** Step 871–880
> **เฟส:** 9 — ทักษะมืออาชีพ: Maintainer, Release Management, Metrics
> **เป้าหมายของ Part นี้:** เจาะลึก Semantic Versioning (SemVer) เต็มรูปแบบตามสเปกจริงจาก semver.org ต่อยอดจากที่ปูพื้นไว้ใน Part 11 ไปจนถึงการวาง Release Management ระดับมืออาชีพ — ตั้งแต่ pre-release/build metadata, กลยุทธ์ release branch, การเขียน release notes ที่มีคุณภาพ, การใช้ GitHub Releases และ GitLab Releases จริง, การทำ automated versioning ด้วย `semantic-release`, การตัดสินใจเรื่อง release cadence และ deprecation policy ไปจนถึงการลงมือ release เวอร์ชัน v1.0.0 แรกของโปรเจกต์แบบครบวงจร

---

## สารบัญของ Part นี้

- Step 871: ทบทวน Semantic Versioning จาก Part 11 แล้วเจาะลึกเต็มรูปแบบ
- Step 872: Pre-release Versions และ Build Metadata ตามสเปก SemVer เต็มรูปแบบ
- Step 873: Release Branch Strategy — เชื่อมโยงกับ Git Flow Release Branch
- Step 874: การเขียน Release Notes ที่ดี
- Step 875: GitHub Releases — สร้าง Release พร้อมแนบไฟล์ Binary/Asset
- Step 876: GitLab Releases — เทียบเคียงด้วย `release:` Keyword
- Step 877: Automated Versioning ด้วย `semantic-release`
- Step 878: Release Cadence — Time-based vs Feature-based
- Step 879: Deprecation Policy — การแจ้งเลิกใช้ฟีเจอร์อย่างมีความรับผิดชอบ
- Step 880: แบบฝึกหัด — วางแผนและทำ Release v1.0.0 แบบครบวงจร

---

## Step 871: ทบทวน Semantic Versioning จาก Part 11 แล้วเจาะลึกเต็มรูปแบบ

### ทบทวนสิ่งที่เรียนไปแล้วใน Part 11 (Step 105)

ใน Part 11 เราปูพื้นฐาน Semantic Versioning (SemVer) ไว้แบบย่อว่า:

- รูปแบบคือ `MAJOR.MINOR.PATCH` เช่น `2.4.1`
- **MAJOR** เพิ่มเมื่อมี breaking change
- **MINOR** เพิ่มเมื่อเพิ่มฟีเจอร์ใหม่แบบไม่กระทบของเดิม (backward compatible)
- **PATCH** เพิ่มเมื่อแก้บั๊กโดยไม่เปลี่ยนพฤติกรรมที่ตั้งใจไว้
- เมื่อเลขซ้ายเพิ่ม เลขขวาทั้งหมดต้องรีเซ็ตเป็น 0
- `0.x.x` หมายถึงช่วงพัฒนาเริ่มต้น ยังไม่รับประกัน API เสถียร
- มีส่วนขยาย pre-release (`-alpha`, `-beta`, `-rc.1`) และ build metadata (`+build.xxx`)

Part นี้จะไม่พูดซ้ำเรื่องพื้นฐานเหล่านั้นอีก แต่จะพาไปอ่าน **สเปกฉบับเต็มของ Semantic Versioning 2.0.0** ที่เผยแพร่อย่างเป็นทางการที่ [semver.org](https://semver.org) ซึ่งเขียนโดย Tom Preston-Werner (ผู้ร่วมก่อตั้ง GitHub) และกลายเป็นมาตรฐานที่ระบบจัดการ dependency แทบทุกภาษาโปรแกรมมิ่งยึดถือ

### ทำไมต้องมี "สเปก" ไม่ใช่แค่ "แนวทางกว้าง ๆ"

จุดสำคัญที่สุดของ SemVer คือมันไม่ใช่แค่ธรรมเนียมปฏิบัติหลวม ๆ แต่เป็น **ข้อตกลง (contract)** ระหว่างผู้พัฒนาซอฟต์แวร์กับผู้ใช้ซอฟต์แวร์นั้น เมื่อคุณประกาศว่าโปรเจกต์ของคุณ "ใช้ SemVer" นั่นหมายความว่าคุณสัญญาว่าจะปฏิบัติตามกฎอย่างเคร่งครัด เพื่อให้เครื่องมืออัตโนมัติ (เช่น `npm`, `pip`, `cargo`, `composer`) สามารถตัดสินใจอัปเดต dependency ให้อัตโนมัติได้อย่างปลอดภัยโดยไม่ต้องมีมนุษย์มาตรวจสอบทุกครั้ง

ถ้าคุณ bump แค่ PATCH แต่จริง ๆ แล้วมันมี breaking change แอบซ่อนอยู่ นั่นคือการ **ผิดสัญญา SemVer** และจะทำให้ระบบอัตโนมัติของผู้ใช้งานพังโดยไม่รู้ตัว เพราะเขาเชื่อใจว่า PATCH ปลอดภัยเสมอ

### 11 กฎหลักของสเปก SemVer 2.0.0 (สรุปแบบเจาะลึก)

สเปกจริงมีทั้งหมด 11 ข้อ (item 1–11) ต่อไปนี้คือการอธิบายแต่ละข้อแบบละเอียดพร้อมความหมายเชิงปฏิบัติ:

#### ข้อ 1: ซอฟต์แวร์ที่ใช้ SemVer ต้องประกาศ Public API ที่ชัดเจน

> "Software using Semantic Versioning MUST declare a public API."

ก่อนจะพูดเรื่องเลขเวอร์ชัน คุณต้องรู้ก่อนว่า **อะไรคือ "public API" ของโปรเจกต์คุณ** เพราะ SemVer วัด compatibility จาก public API เท่านั้น ไม่ใช่จากโค้ดภายในทั้งหมด

| ประเภทโปรเจกต์ | Public API คืออะไร |
|---|---|
| Library/Package (เช่น npm package, Python package) | ฟังก์ชัน, class, ค่าคงที่ที่ export ออกมาให้คนอื่นเรียกใช้ |
| REST API / Web Service | Endpoint, request/response schema, HTTP status codes ที่สัญญาไว้ในเอกสาร |
| CLI Tool | คำสั่ง, flags, output format ที่ผู้ใช้พึ่งพา (เช่น เอาไปเขียน script ต่อ) |
| Application ทั่วไป (ไม่มีคนอื่นเรียกใช้โค้ดโดยตรง) | อาจไม่มี "public API" ชัดเจน — ทีมสามารถตกลงกันเองว่าจะใช้ SemVer วัดจากอะไร เช่น database schema หรือ config format |

โค้ดภายใน (private/internal implementation) ที่ไม่มีใครเรียกใช้จากภายนอกสามารถเปลี่ยนแปลงได้อย่างอิสระโดยไม่ต้องขยับเลขเวอร์ชันตาม เพราะมันไม่ใช่ส่วนหนึ่งของ "สัญญา" ที่ให้ไว้กับผู้ใช้

#### ข้อ 2: รูปแบบเลขเวอร์ชันต้องเป็น `X.Y.Z` และเป็นจำนวนเต็มไม่ติดลบเท่านั้น

> "A normal version number MUST take the form X.Y.Z where X, Y, and Z are non-negative integers, and MUST NOT contain leading zeroes."

กฎย่อยที่สำคัญมากและมักถูกมองข้าม:

- **X, Y, Z ต้องเป็นจำนวนเต็มไม่ติดลบ** (0, 1, 2, 3, ...) ห้ามเป็นทศนิยมหรือค่าติดลบ
- **ห้ามมีเลข 0 นำหน้า (leading zero)** เช่น `1.02.3` หรือ `01.2.3` เป็นเวอร์ชันที่ผิดสเปก ต้องเขียนเป็น `1.2.3`
- แต่ละตัวเลขต้องเพิ่มขึ้นทีละหน่วยตามลำดับตัวเลข (numerically) เช่น `1.9.0` ต้องมาก่อน `1.10.0` เสมอเมื่อเทียบเชิงตัวเลข — นี่คือเหตุผลที่ Part 11 (Step 104.3) แนะนำให้ใช้ `git tag --sort=v:refname` แทนการเรียงตามตัวอักษรธรรมดา เพราะการเรียงแบบ string จะได้ `1.10.0` มาก่อน `1.9.0` ซึ่งผิดหลักการเปรียบเทียบเวอร์ชัน

#### ข้อ 3: เมื่อ release เวอร์ชันหนึ่งไปแล้ว ห้ามแก้ไขเนื้อหาของเวอร์ชันนั้นอีก

> "Once a versioned package has been released, the contents of that version MUST NOT be modified. Any modifications MUST be released as a new version."

นี่คือกฎที่สำคัญมากในทางปฏิบัติ: **เวอร์ชันที่ปล่อยไปแล้วคือ immutable (แก้ไขไม่ได้อีก)** ถ้าคุณพบว่า `v1.2.3` ที่ปล่อยไปมีบั๊ก คุณ**ห้ามแก้ไฟล์แล้ว publish ทับ tag/package เดิม** แต่ต้องปล่อย `v1.2.4` ใหม่ออกมาแทน

เหตุผลคือถ้าผู้ใช้ pin เวอร์ชันไว้ที่ `v1.2.3` แล้วเนื้อหาข้างในถูกสลับโดยไม่รู้ตัว ระบบของเขาอาจพังหรือมีพฤติกรรมเปลี่ยนไปโดยไม่มีสัญญาณเตือนใด ๆ เลย ซึ่งขัดกับเจตนารมณ์ทั้งหมดของ SemVer — นี่คือเหตุผลเดียวกับที่ Part 11 (Step 107) เตือนไว้ว่าไม่ควรลบ Tag ของ release ที่เผยแพร่ไปแล้ว เพราะหลักการ immutability ของเวอร์ชันสำคัญกว่าความสะดวกในการแก้ไข

#### ข้อ 4: Major version ศูนย์ (`0.y.z`) คือช่วงพัฒนาเริ่มต้น

> "Major version zero (0.y.z) is for initial development. Anything MAY change at any time. The public API SHOULD NOT be considered stable."

ในช่วง `0.y.z` แม้แต่การเพิ่ม MINOR ก็สามารถมี breaking change ได้โดยไม่ผิดสเปก เพราะทั้งหมดยังถือว่าอยู่ในช่วงทดลอง ผู้ใช้ที่ dependency บนเวอร์ชัน `0.x.x` ต้องเข้าใจความเสี่ยงนี้ล่วงหน้า

#### ข้อ 5: เวอร์ชัน `1.0.0` คือจุดที่นิยาม Public API อย่างเป็นทางการ

> "Version 1.0.0 defines the public API."

การปล่อย `1.0.0` คือการประกาศต่อสาธารณะว่า "จากนี้ไป เราจะรักษาสัญญาเรื่อง backward compatibility อย่างจริงจังตามกฎ SemVer" มันไม่ใช่แค่ตัวเลขสวย ๆ แต่เป็น **จุดเปลี่ยนความรับผิดชอบ** ของทีมพัฒนา

#### ข้อ 6: PATCH version ต้องเพิ่มเมื่อมีการแก้บั๊กแบบ backward compatible เท่านั้น

> "Patch version Z (x.y.Z | x > 0) MUST be incremented if only backward compatible bug fixes are introduced. A bug fix is defined as an internal change that fixes incorrect behavior."

สังเกตคำว่า **"internal change"** — การแก้บั๊กที่นับเป็น PATCH ต้องเป็นการแก้พฤติกรรมที่ผิดพลาดโดยไม่เปลี่ยน public API ที่ประกาศไว้เลยแม้แต่นิดเดียว ถ้าการแก้บั๊กนั้นบังเอิญไปเปลี่ยน signature ของฟังก์ชัน หรือเปลี่ยน response format ของ API แม้จะเป็นเจตนาดี ก็ไม่ควรนับเป็น PATCH อีกต่อไป — ต้องพิจารณาว่าเป็น MINOR หรือ MAJOR แทน

#### ข้อ 7: MINOR version ต้องเพิ่มเมื่อเพิ่มฟีเจอร์ใหม่แบบ backward compatible หรือเมื่อ deprecate ฟีเจอร์

> "Minor version Y (x.Y.z | x > 0) MUST be incremented if new, backward compatible functionality is introduced to the public API. It MUST be incremented if any public API functionality is marked as deprecated. It MAY be incremented if substantial new functionality or improvements are introduced within the private code. It MAY include patch level changes. Patch version MUST be reset to 0 when minor version is incremented."

ข้อนี้มีรายละเอียดที่คนมักพลาดคือ **ต้อง (MUST) เพิ่ม MINOR เมื่อ deprecate ฟีเจอร์ด้วย** ไม่ใช่แค่ตอนเพิ่มฟีเจอร์ใหม่เท่านั้น — เราจะพูดเรื่องนี้ละเอียดใน Step 879

#### ข้อ 8: MAJOR version ต้องเพิ่มเมื่อมี breaking change ใด ๆ ต่อ public API

> "Major version X (X.y.z | X > 0) MUST be incremented if any backward incompatible changes are introduced to the public API. It MAY include minor and patch level changes. Patch and minor version MUST be reset to 0 when major version is incremented."

ข้อสังเกตสำคัญ: "**any**" backward incompatible change — ไม่ว่าจะเล็กแค่ไหน ถ้าทำให้โค้ดที่ใช้เวอร์ชันเก่าพังหรือพฤติกรรมเปลี่ยนไปจากที่สัญญาไว้ ต้องขยับ MAJOR เสมอ แม้จะรู้สึกว่า "การเปลี่ยนแปลงนี้เล็กน้อยมาก" ก็ตาม

### ตารางสรุปตัดสินใจแบบใช้งานจริง (Decision Table)

| คำถามที่ต้องถามตัวเอง | คำตอบ "ใช่" | Bump อะไร |
|---|---|---|
| มีอะไรบางอย่างที่เคยใช้งานได้ จะพังหรือพฤติกรรมเปลี่ยนไปหรือไม่ ถ้าผู้ใช้อัปเดตมาเวอร์ชันนี้โดยไม่แก้โค้ดตัวเอง | ใช่ | **MAJOR** |
| เพิ่มความสามารถใหม่ที่ผู้ใช้เดิมไม่จำเป็นต้องแก้อะไรเลยก็ยังใช้งานได้ปกติ | ใช่ | **MINOR** |
| ประกาศ deprecate ฟีเจอร์ (ยังใช้งานได้อยู่ แต่เตือนว่าจะเลิกรองรับในอนาคต) | ใช่ | **MINOR** |
| แก้ไขพฤติกรรมที่ผิดพลาดให้ตรงกับที่เอกสารสัญญาไว้ โดยไม่กระทบ public API | ใช่ | **PATCH** |
| แก้ไขปัญหาด้าน performance/security ภายในโดยไม่เปลี่ยน public API | ใช่ | **PATCH** |

### ข้อผิดพลาดที่พบบ่อยในการใช้ SemVer จริง

1. **เข้าใจว่า "การเปลี่ยนแปลงเล็กน้อย" เท่ากับ PATCH เสมอ** — ทั้งที่จริง ๆ เกณฑ์วัดคือ "กระทบ public API หรือไม่" ไม่ใช่ขนาดของ diff
2. **ลืม deprecate ก่อน remove** — การลบฟีเจอร์ทันทีโดยไม่เคย deprecate มาก่อนถือเป็น breaking change ที่รุนแรงและไม่ให้เกียรติผู้ใช้ (ดู Step 879)
3. **ใช้ MINOR สำหรับ breaking change เพราะ "รู้สึกว่ายังไม่ major พอ"** — SemVer ไม่สนใจว่า breaking change นั้น "ใหญ่" แค่ไหน สนใจแค่ว่ามันทำให้ของเดิมพังหรือไม่
4. **ไม่มี public API ที่ชัดเจนตั้งแต่แรก** — ทำให้ทีมเถียงกันเองว่าอะไรคือ breaking change เพราะไม่มีเอกสารอ้างอิงว่าอะไรคือสัญญาที่ให้ไว้กับผู้ใช้

---

## Step 872: Pre-release Versions และ Build Metadata ตามสเปก SemVer เต็มรูปแบบ

Part 11 แนะนำ pre-release และ build metadata ไว้แบบผิวเผิน ใน Step นี้เราจะดูกฎที่แน่นอนตามสเปกจริง (ข้อ 9, 10, 11 ของ semver.org)

### รูปแบบเต็มของเวอร์ชันตาม SemVer

```
MAJOR.MINOR.PATCH-PRERELEASE+BUILD
```

ตัวอย่างเวอร์ชันที่ถูกต้องตามสเปกครบทุกส่วน:

```
1.0.0-alpha.1+exp.sha.5114f85
```

แยกส่วนได้เป็น:

| ส่วน | ค่า | ความหมาย |
|---|---|---|
| Core version | `1.0.0` | MAJOR.MINOR.PATCH |
| Pre-release | `alpha.1` | บอกว่ายังไม่ใช่เวอร์ชันตัวจริง |
| Build metadata | `exp.sha.5114f85` | ข้อมูลเสริมเกี่ยวกับ build เช่น commit hash |

### กฎของ Pre-release Identifier (ข้อ 9)

> "A pre-release version MAY be denoted by appending a hyphen and a series of dot separated identifiers immediately following the patch version."

กฎย่อยที่ต้องปฏิบัติตามอย่างเคร่งครัด:

1. Identifier แต่ละตัวคั่นด้วยจุด (`.`) เช่น `alpha.1`, `beta.2`, `x.7.z.92`
2. Identifier ต้องประกอบด้วยตัวอักษร ASCII alphanumeric เท่านั้น (`[0-9A-Za-z-]`) และเครื่องหมายขีดกลาง (`-`)
3. **Identifier ต้องไม่เป็นค่าว่าง** (เช่น `1.0.0-` หรือ `1.0.0-alpha..1` ผิดสเปกเพราะมี identifier ว่างอยู่ระหว่างจุด)
4. **Identifier ที่เป็นตัวเลขล้วนต้องไม่มีเลข 0 นำหน้า** เช่น `1.0.0-alpha.01` ผิดสเปก ต้องเขียนเป็น `1.0.0-alpha.1`
5. **เวอร์ชันที่มี pre-release จะมี precedence ต่ำกว่าเวอร์ชันตัวจริงที่ core version เดียวกันเสมอ** เช่น `1.0.0-alpha` < `1.0.0`

### กฎของ Build Metadata (ข้อ 10)

> "Build metadata MAY be denoted by appending a plus sign and a series of dot separated identifiers immediately following the patch or pre-release version."

ข้อแตกต่างสำคัญจาก pre-release:

1. ใช้เครื่องหมาย **บวก (`+`)** นำหน้า ไม่ใช่ขีดกลาง
2. Identifier ยังคงเป็น ASCII alphanumeric + hyphen เท่านั้น และห้ามว่างเปล่าเช่นกัน
3. **Build metadata ไม่มีข้อจำกัดเรื่องเลข 0 นำหน้า** ต่างจาก pre-release identifier
4. **Build metadata ต้องถูกละเลย (ignore) เมื่อคำนวณ precedence** — เวอร์ชันสองตัวที่ core version และ pre-release เหมือนกันทุกอย่าง ต่างกันแค่ build metadata จะถือว่ามี **precedence เท่ากัน** เช่น `1.0.0+build.1` และ `1.0.0+build.2` ถือว่า "เท่ากัน" ในแง่การเปรียบเทียบเวอร์ชัน (แม้เนื้อหาจริงข้างในอาจต่างกัน)

### กฎการเปรียบเทียบ Precedence (ข้อ 11) — เรื่องที่ซับซ้อนที่สุดของสเปก

การเปรียบเทียบว่าเวอร์ชันไหน "ใหม่กว่า" ทำตามลำดับนี้:

**ขั้นที่ 1:** เปรียบเทียบ `MAJOR.MINOR.PATCH` เป็นตัวเลขทีละตำแหน่งจากซ้ายไปขวา ใครมากกว่าคือใหม่กว่าทันที (ไม่ต้องดูส่วนอื่นต่อ)

```
1.0.0 < 2.0.0 < 2.1.0 < 2.1.1
```

**ขั้นที่ 2:** ถ้า core version เท่ากันทั้งหมด ให้เทียบ pre-release:
- เวอร์ชันที่ **ไม่มี** pre-release จะมี precedence **สูงกว่า** เวอร์ชันที่มี pre-release เสมอ (เมื่อ core version เท่ากัน)
  ```
  1.0.0-alpha < 1.0.0
  ```

**ขั้นที่ 3:** ถ้าทั้งคู่มี pre-release ให้เทียบ identifier ทีละตัวจากซ้ายไปขวา ตามกฎ:
- Identifier ที่เป็นตัวเลขล้วน เทียบแบบตัวเลข (numeric)
- Identifier ที่มีตัวอักษรหรือขีดกลางปน เทียบแบบ ASCII lexical order (string comparison)
- **Identifier ตัวเลขล้วนจะมี precedence ต่ำกว่า identifier ที่เป็นตัวอักษรเสมอ** เมื่อเทียบกันในตำแหน่งเดียวกัน
- ถ้า identifier ทั้งหมดที่เทียบมาเท่ากันจนหมด แต่ชุดหนึ่งมี identifier มากกว่า ชุดที่ยาวกว่าจะมี precedence สูงกว่า

### ตัวอย่างลำดับ precedence เต็มรูปแบบตามสเปก (จากเอกสารทางการ)

```
1.0.0-alpha < 1.0.0-alpha.1 < 1.0.0-alpha.beta < 1.0.0-beta
< 1.0.0-beta.2 < 1.0.0-beta.11 < 1.0.0-rc.1 < 1.0.0
```

อธิบายทีละคู่:

| เปรียบเทียบ | เหตุผล |
|---|---|
| `1.0.0-alpha` < `1.0.0-alpha.1` | ชุดแรกมี identifier 1 ตัว (`alpha`) ชุดหลังมี 2 ตัว (`alpha`, `1`) เมื่อตัวแรกเท่ากัน ชุดที่ยาวกว่าชนะ |
| `1.0.0-alpha.1` < `1.0.0-alpha.beta` | ตำแหน่งที่สอง: `1` (ตัวเลข) เทียบกับ `beta` (ตัวอักษร) — ตัวเลขล้วน precedence ต่ำกว่าตัวอักษรเสมอ |
| `1.0.0-alpha.beta` < `1.0.0-beta` | ตำแหน่งแรก: `alpha` เทียบกับ `beta` แบบ ASCII lexical → `alpha` < `beta` |
| `1.0.0-beta` < `1.0.0-beta.2` | เหมือนกรณี alpha ก่อนหน้า ชุดที่ยาวกว่าชนะเมื่อตัวแรกเท่ากัน |
| `1.0.0-beta.2` < `1.0.0-beta.11` | ตำแหน่งที่สอง: `2` เทียบกับ `11` แบบ**ตัวเลข** (ไม่ใช่ string) ดังนั้น `2 < 11` — ถ้าเทียบแบบ string ผิด ๆ จะได้ `"11" < "2"` ซึ่งผิด! |
| `1.0.0-beta.11` < `1.0.0-rc.1` | ตำแหน่งแรก: `beta` < `rc` แบบ ASCII lexical |
| `1.0.0-rc.1` < `1.0.0` | เวอร์ชันไม่มี pre-release มี precedence สูงกว่าเวอร์ชันที่มี pre-release เสมอ |

### Regular Expression อย่างเป็นทางการสำหรับตรวจสอบ SemVer

สเปกทางการให้ regex ไว้สำหรับตรวจสอบว่าสตริงหนึ่งเป็น valid SemVer หรือไม่ (ย่อจากต้นฉบับ semver.org):

```regex
^(?<major>0|[1-9]\d*)\.(?<minor>0|[1-9]\d*)\.(?<patch>0|[1-9]\d*)(?:-(?<prerelease>(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*)(?:\.(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*))*))?(?:\+(?<buildmetadata>[0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?$
```

เครื่องมือ semver library ในแทบทุกภาษา (เช่น `semver` npm package, `python-semver`) ใช้ regex ตัวนี้เป็นมาตรฐานในการ validate

### รูปแบบ Pre-release ที่นิยมใช้จริงในอุตสาหกรรม

| Label | ความหมาย | ตัวอย่าง |
|---|---|---|
| `alpha` | เวอร์ชันทดลองภายในทีม ยังไม่เสถียร ฟีเจอร์อาจยังไม่ครบ | `2.0.0-alpha.1` |
| `beta` | ฟีเจอร์ครบแล้ว แต่ยังอยู่ระหว่างทดสอบเชิงลึก อาจมีบั๊กเหลืออยู่ | `2.0.0-beta.3` |
| `rc` (release candidate) | ผู้สมัครที่คาดว่าจะเป็นตัวจริง ถ้าไม่พบปัญหาสำคัญจะกลายเป็นเวอร์ชันจริงทันที | `2.0.0-rc.1` |
| `next` | บาง ecosystem (โดยเฉพาะ npm) ใช้เป็น dist-tag คู่กับ pre-release เพื่อให้ผู้ใช้ติดตั้งแบบทดลองผ่าน `npm install pkg@next` | `2.0.0-next.5` |

### ตัวอย่างการสร้าง Tag สำหรับ pre-release ใน Git

```bash
# สร้าง release candidate ตัวแรกก่อนปล่อย v2.0.0 ตัวจริง
git tag -a v2.0.0-rc.1 -m "Release candidate 1 for v2.0.0"
git push origin v2.0.0-rc.1

# หลังทดสอบผ่านหมดแล้ว ค่อยติด tag ตัวจริง
git tag -a v2.0.0 -m "Release v2.0.0"
git push origin v2.0.0
```

### Build Metadata ในทางปฏิบัติ

Build metadata มักถูกใช้เพื่อฝัง**ข้อมูลที่ไม่ส่งผลต่อความหมายของเวอร์ชัน** เข้าไปในสตริงเวอร์ชัน เช่น:

```
1.0.0+20240115120000        → timestamp ของ build
1.0.0+sha.a3f9c21            → commit hash ที่ build มาจาก
1.0.0-beta.1+exp.sha.5114f85 → รวม pre-release กับ build metadata เข้าด้วยกัน
```

ระบบ CI/CD จำนวนมากใช้แนวทางนี้เพื่อให้ทุก build ที่ generate ออกมามี identifier เฉพาะตัว โดยไม่ต้องกังวลว่าจะไปชนกับกฎ precedence เพราะ build metadata ถูก "มองไม่เห็น" ในการเปรียบเทียบเวอร์ชันเสมอ

---

## Step 873: Release Branch Strategy — เชื่อมโยงกับ Git Flow Release Branch

### ทบทวน Release Branch จาก Part 32 (Git Flow)

ใน Part 32 (Step 314) เราเรียนรู้ว่า Git Flow มี **Release Branch** เป็นหนึ่งในโครงสร้างหลัก โดยมีหน้าที่:

> เป็น branch ชั่วคราวที่แตกออกมาจาก `develop` เมื่อฟีเจอร์ทั้งหมดสำหรับเวอร์ชันถัดไปพร้อมแล้ว ใช้สำหรับ **"เตรียมความพร้อมก่อนปล่อยจริง"** เช่น แก้บั๊กเล็กน้อยที่พบตอนทดสอบ, ปรับ version number, เขียน documentation สุดท้าย โดย**ไม่รับฟีเจอร์ใหม่เข้ามาอีก** (feature freeze)

Workflow เดิมจาก Git Flow:

```
develop ──┬──────────────────────────────────
          │
          └── release/1.2.0 ──┬── (bump version, fix bugs) ──┐
                               │                              │
                    merge กลับเข้า develop ◄──────────────────┤
                                                               │
                                                    merge เข้า main ──── tag v1.2.0
```

### เชื่อมโยงเข้ากับ Semantic Versioning: Release Branch ควรตั้งชื่ออย่างไร

เมื่อเชื่อม Release Branch เข้ากับ SemVer มีแนวทางตั้งชื่อที่นิยมสองแบบ:

| รูปแบบ | ตัวอย่าง | เหมาะกับ |
|---|---|---|
| ระบุเวอร์ชันเต็ม | `release/1.2.0` | ทีมที่ตัดสินใจเลขเวอร์ชันล่วงหน้าได้ชัดเจนก่อนเริ่ม freeze |
| ระบุแค่ major.minor | `release/1.2.x` หรือ `release/1.2` | ทีมที่ต้องการเปิดทางให้ patch เวอร์ชันย่อยหลายตัว (1.2.0, 1.2.1, 1.2.2) ออกมาจาก branch เดียวกันได้ต่อเนื่อง |

แบบที่สอง (`release/1.2.x`) ได้รับความนิยมมากกว่าในโปรเจกต์ที่ต้องดูแลเวอร์ชันเก่าต่อเนื่องเป็นเวลานาน เพราะ branch นี้จะไม่ถูกลบทิ้งหลัง merge เหมือน Git Flow ดั้งเดิม แต่จะถูกเก็บไว้เป็น **"maintenance branch"** สำหรับออก patch ในอนาคต

### Long-Term Maintenance Branches (การดูแลหลาย Major Version พร้อมกัน)

โปรเจกต์ที่มีผู้ใช้จำนวนมากและ major version หลายตัวใช้งานพร้อมกันในโลกจริง (เช่น library ที่องค์กรต่าง ๆ ยังไม่ยอมอัปเกรด major ใหม่) มักต้องดูแลหลาย branch พร้อมกัน:

```
main ────────────────────────────────────► (3.x.x ปัจจุบัน)
release/2.x ──────────────────────────────► (ยังออก patch ให้ 2.x.x อยู่)
release/1.x ──────────────────────────────► (เฉพาะ security patch เท่านั้น)
```

ตัวอย่างจริงในอุตสาหกรรม: **Node.js** ใช้ระบบ **Even/Odd Release Line** ผสมกับแนวคิด LTS (Long Term Support):

| Release Line | สถานะ | ตัวอย่าง |
|---|---|---|
| เลขคู่ (even major) | ได้สิทธิ์เข้าสู่สถานะ LTS หลังผ่านการทดสอบระยะหนึ่ง ได้รับการดูแลนานหลายปี | Node.js 18, 20, 22 |
| เลขคี่ (odd major) | Current release อายุสั้น ใช้ทดลองฟีเจอร์ใหม่ | Node.js 19, 21 |

### Hotfix ที่ต้อง Backport ไปหลาย Release Branch

เมื่อพบบั๊กร้ายแรง (โดยเฉพาะช่องโหว่ความปลอดภัย) ที่ต้องแก้ในหลายเวอร์ชันพร้อมกัน แนวทางปฏิบัติคือ:

```bash
# 1. แก้บั๊กบน branch ล่าสุดก่อน (main หรือ develop)
git checkout main
git commit -m "fix: patch buffer overflow in parser"

# 2. Cherry-pick การแก้ไขเดียวกันไปยัง release branch เก่าที่ยังดูแลอยู่
git checkout release/2.x
git cherry-pick <commit-hash>
git tag -a v2.8.4 -m "Security patch"
git push origin release/2.x v2.8.4

git checkout release/1.x
git cherry-pick <commit-hash>
git tag -a v1.14.9 -m "Security patch (backport)"
git push origin release/1.x v1.14.9
```

การ backport แบบนี้ทำให้ผู้ใช้ทุก major version ที่ยังอยู่ในระยะ support ได้รับการแก้ไขความปลอดภัย โดยไม่ถูกบังคับให้ต้อง major upgrade ทันที

### เปรียบเทียบ Release Branch Strategy สามรูปแบบ

| Strategy | วิธีทำงาน | ข้อดี | ข้อเสีย |
|---|---|---|---|
| **Git Flow release branch** (จาก Part 32) | สร้าง `release/x.y.0` ชั่วคราวก่อนปล่อยแต่ละครั้ง แล้ว merge/ลบทิ้ง | เหมาะกับ release cadence ที่ไม่ถี่มาก มีเวลา freeze ทดสอบชัดเจน | overhead สูงถ้าต้อง release บ่อย |
| **GitHub Flow แบบ tag-only** (ไม่มี release branch) | Tag ตรงบน `main` ทุกครั้งที่พร้อม release ไม่มี branch แยก | เรียบง่าย เหมาะกับ continuous delivery | ควบคุมช่วง freeze ยากกว่า ต้องพึ่ง feature flag แทน |
| **Long-term maintenance branch** | เก็บ `release/N.x` ไว้ตลอดอายุการ support ของ major version นั้น | รองรับหลาย major version พร้อมกัน ทำ backport ได้เป็นระบบ | ต้องดูแลหลาย branch พร้อมกัน เพิ่มภาระทีม |

### แนวทางเลือก Release Branch Strategy ให้เหมาะกับโปรเจกต์

1. **โปรเจกต์ภายในทีมเล็ก deploy บ่อย (หลายครั้งต่อวัน)** → ไม่จำเป็นต้องมี release branch เลย ใช้ tag บน `main` ตรง ๆ ร่วมกับ feature flag
2. **Library/Package ที่มีผู้ใช้ภายนอกจำนวนมาก** → ควรมี release branch แบบ `release/x.y` เพื่อรองรับการออก patch ย้อนหลังได้
3. **ซอฟต์แวร์ enterprise ที่ลูกค้าจ่ายเงินซื้อ license รายปีและอาจไม่อัปเกรดทันที** → จำเป็นต้องมี long-term maintenance branch หลายสาย พร้อมนโยบาย support ที่ชัดเจนว่าแต่ละ major version จะดูแลถึงเมื่อไหร่

---

## Step 874: การเขียน Release Notes ที่ดี

### Release Notes คืออะไร และทำไมสำคัญ

**Release Notes** คือเอกสารที่อธิบายว่า "เวอร์ชันนี้มีอะไรเปลี่ยนแปลงไปจากเวอร์ชันก่อนหน้า" มันคือจุดเชื่อมต่อสำคัญระหว่างทีมพัฒนากับผู้ใช้ — ถ้าเขียนดี ผู้ใช้จะตัดสินใจอัปเดตได้อย่างมั่นใจ ถ้าเขียนแย่หรือไม่เขียนเลย ผู้ใช้จะกลัวการอัปเดตและอาจติดค้างอยู่กับเวอร์ชันเก่าที่มีช่องโหว่ความปลอดภัยไปเรื่อย ๆ

### Release Notes เขียนให้ "ใคร" อ่าน — จุดที่คนเขียนพลาดบ่อยที่สุด

ความผิดพลาดที่พบบ่อยที่สุดคือการเอา **commit log ดิบ ๆ** มาแปะเป็น release notes ตรง ๆ โดยไม่คำนึงว่าใครจะเป็นคนอ่าน

| ผู้อ่าน | สิ่งที่เขาต้องการรู้ | ภาษาที่ควรใช้ |
|---|---|---|
| **ผู้ใช้ทั่วไป (end user)** | "ฟีเจอร์อะไรใหม่ที่ฉันจะได้ใช้" "อะไรที่อาจกระทบการใช้งานของฉัน" | ภาษาธรรมดา เน้นประโยชน์ ไม่ใช้ศัพท์เทคนิคเกินจำเป็น |
| **นักพัฒนาที่เอา library/API ไปใช้ต่อ** | "อะไร breaking change" "ต้องแก้โค้ดอะไรบ้างถึงจะอัปเกรดได้" | ระบุ API/function ที่เปลี่ยนแปลงชัดเจน พร้อมตัวอย่าง migration |
| **ทีมภายใน / QA** | "commit ไหนแก้ปัญหาอะไร" "เทสต์อะไรที่ต้อง regression" | อ้างอิง ticket/issue number ได้ตรง ๆ |
| **ฝ่ายความปลอดภัย/Compliance** | "มี security fix อะไรบ้าง ระดับความรุนแรงเท่าไหร่" | ระบุ CVE (ถ้ามี) และความรุนแรงชัดเจน |

หลักการสำคัญ: **Release notes ที่ดีมักต้อง "แปล" จาก commit message ทางเทคนิค ให้กลายเป็นภาษาที่มนุษย์ทั่วไปเข้าใจได้** ไม่ใช่แค่ copy-paste commit list

### โครงสร้างมาตรฐานของ Release Notes ที่ดี

```markdown
## v2.4.0 — 2026-03-15

### สรุปโดยย่อ (TL;DR)
เวอร์ชันนี้เพิ่มระบบแจ้งเตือนแบบ real-time และแก้ปัญหาประสิทธิภาพที่กระทบผู้ใช้จำนวนมาก

### Breaking Changes
- ฟังก์ชัน `getUserData()` เปลี่ยนค่าที่คืนกลับจาก object เป็น Promise (ดู [migration guide](./MIGRATION.md))

### ฟีเจอร์ใหม่
- เพิ่มระบบแจ้งเตือนแบบ real-time ผ่าน WebSocket (#412)
- รองรับการ export รายงานเป็นไฟล์ Excel (#420)

### การปรับปรุง
- ลดเวลาโหลดหน้า Dashboard ลง 40% (#405)

### แก้ไขบั๊ก
- แก้ปัญหาการคำนวณยอดรวมผิดพลาดเมื่อมีส่วนลดหลายรายการ (#398)

### Deprecated
- ฟังก์ชัน `oldExportCSV()` ถูก deprecate แล้ว จะถูกลบใน v3.0.0 กรุณาเปลี่ยนไปใช้ `exportReport({format: 'csv'})` แทน

### ขอบคุณผู้ร่วมพัฒนา
ขอบคุณ @somchai และ @malee สำหรับการรายงานบั๊กและ pull request ในเวอร์ชันนี้
```

โครงสร้างนี้ใกล้เคียงกับแนวทาง **Keep a Changelog** ซึ่งแบ่งหมวดหมู่เป็น Added, Changed, Deprecated, Removed, Fixed, Security — เราจะเจาะลึกเรื่องการทำ changelog แบบอัตโนมัติเต็มรูปแบบใน **Part 89** ที่จะพูดถึงเครื่องมือสร้าง changelog จาก commit history โดยตรง ส่วน Part นี้เน้นที่หลักการเขียนเนื้อหาให้มีคุณภาพก่อน

### กฎทองของการเขียน Release Notes ที่ดี

1. **ขึ้นต้นด้วยสิ่งที่สำคัญที่สุดก่อนเสมอ** — ถ้ามี breaking change ต้องอยู่บนสุดและเด่นชัดที่สุด ห้ามซ่อนไว้ท้ายเอกสาร
2. **เขียนจากมุมมองของผู้ใช้ ไม่ใช่จากมุมมองของโค้ด** — แทนที่จะเขียนว่า "Refactored UserService class" ให้เขียนว่า "ระบบล็อกอินเร็วขึ้น 2 เท่า"
3. **ให้ตัวอย่างโค้ดเมื่อมี breaking change** — แสดง "ก่อน" กับ "หลัง" เทียบกันให้เห็นชัด
4. **อ้างอิง issue/PR number เสมอ** เพื่อให้คนที่อยากรู้รายละเอียดเพิ่มเติมตามไปดูได้
5. **อย่าละเลยการให้เครดิตผู้ร่วมพัฒนา** โดยเฉพาะโปรเจกต์ Open Source — สิ่งนี้สร้างแรงจูงใจให้คนอยากช่วย contribute ต่อ
6. **ระบุวันที่ release เสมอ** เพื่อให้ผู้ใช้คำนวณระยะเวลาได้ (เช่น "เวอร์ชันนี้ห่างจากตัวก่อนหน้ากี่เดือน")
7. **เขียนให้อ่านจบได้ในเวลาสั้น ๆ** — ใช้ bullet point ไม่ใช่ย่อหน้ายาว ๆ

### ตัวอย่างเปรียบเทียบ Release Notes แย่ vs ดี

**แบบแย่ (copy commit log ดิบ ๆ):**

```
- fix bug
- update deps
- wip
- merge branch 'feature/x' into develop
- typo fix
- refactor
```

**แบบดี (แปลเป็นภาษาที่มีความหมายกับผู้อ่าน):**

```
### แก้ไขบั๊ก
- แก้ปัญหาระบบค้างเมื่อผู้ใช้กดปุ่มบันทึกซ้ำเร็วเกินไป (#301)

### การปรับปรุงความปลอดภัย
- อัปเดตไลบรารี `axios` เป็น v1.6.2 เพื่อปิดช่องโหว่ CVE-2024-XXXX
```

จะเห็นว่า commit message แบบ `fix bug` หรือ `wip` ไม่มีประโยชน์อะไรกับผู้อ่าน release notes เลย — นี่คือเหตุผลสำคัญที่การมี **Conventional Commits** ตามมาตรฐานที่เรียนไปใน Part 35 ช่วยได้มาก เพราะทำให้เครื่องมืออัตโนมัติสามารถแปลง commit message ที่มีโครงสร้างชัดเจนให้กลายเป็น release notes ที่อ่านรู้เรื่องได้ (รายละเอียดการทำอัตโนมัตินี้จะอยู่ใน Step 877 และ Part 89)

---

## Step 875: GitHub Releases — สร้าง Release พร้อมแนบไฟล์ Binary/Asset

### GitHub Releases คืออะไร

**GitHub Releases** คือฟีเจอร์ของ GitHub ที่สร้างขึ้นจาก **Git Tag** (ตามที่เรียนใน Part 11) แล้วเพิ่มชั้นข้อมูลเสริมเข้าไป ได้แก่ ชื่อ release, release notes แบบ formatted, และไฟล์แนบ (binary/asset) ที่ผู้ใช้ดาวน์โหลดได้โดยตรงโดยไม่ต้อง clone repository ทั้งหมด

ทุก GitHub Release **ต้องผูกกับ Tag เสมอ** — ถ้า Tag นั้นยังไม่มีในเครื่อง ระบบจะสร้าง Tag ใหม่ให้อัตโนมัติที่ commit ซึ่งคุณเลือกตอนสร้าง release

### วิธีที่ 1: สร้าง Release ผ่านหน้าเว็บ GitHub

ขั้นตอนบนเว็บ:

1. ไปที่หน้า repository → แท็บ **Releases** → กด **Draft a new release**
2. เลือก **Choose a tag**: พิมพ์ชื่อ tag ใหม่ (เช่น `v1.0.0`) หรือเลือก tag ที่มีอยู่แล้ว
3. เลือก **Target**: branch หรือ commit ที่ tag นี้จะชี้ไป (ถ้า tag ยังไม่มีอยู่จริง)
4. ใส่ **Release title** (เช่น "v1.0.0 — First Stable Release")
5. เขียน **Release notes** ในกล่องข้อความ (รองรับ Markdown เต็มรูปแบบ) หรือกดปุ่ม **Generate release notes** ให้ GitHub สรุปจาก Pull Request ที่ merge เข้ามาตั้งแต่ release ก่อนหน้าให้อัตโนมัติ
6. ลากไฟล์ binary/asset (เช่น `.zip`, `.tar.gz`, `.exe`, `.dmg`) มาวางในช่อง **Attach binaries**
7. เลือก checkbox **Set as a pre-release** ถ้าเป็นเวอร์ชันทดสอบ (`alpha`/`beta`/`rc`)
8. เลือก **Set as the latest release** เพื่อกำหนดว่า release นี้จะแสดงเป็น "Latest" บนหน้า repository
9. กด **Publish release**

### วิธีที่ 2: สร้าง Release ผ่าน GitHub CLI (`gh`) — แนวทางที่เหมาะกับ Automation

ขั้นตอนบน command line โดยใช้เครื่องมือ `gh` (GitHub CLI):

```bash
# ขั้นที่ 1: สร้าง annotated tag ตามที่เรียนใน Part 11 แล้ว push ขึ้น remote
git tag -a v1.0.0 -m "First stable release"
git push origin v1.0.0

# ขั้นที่ 2: build binary/asset ที่ต้องการแนบ (ตัวอย่างสมมติ)
mkdir -p dist
tar -czf dist/myapp-linux-amd64.tar.gz ./build/linux/myapp
zip -j dist/myapp-windows-amd64.zip ./build/windows/myapp.exe

# ขั้นที่ 3: สร้าง GitHub Release พร้อมแนบไฟล์ในคำสั่งเดียว
gh release create v1.0.0 \
  dist/myapp-linux-amd64.tar.gz \
  dist/myapp-windows-amd64.zip \
  --title "v1.0.0 — First Stable Release" \
  --notes-file CHANGELOG.md
```

flag ที่ใช้บ่อยของ `gh release create`:

| Flag | ความหมาย |
|---|---|
| `--title` | ชื่อ release ที่แสดงบนหน้า GitHub |
| `--notes` / `--notes-file` | ข้อความ release notes โดยตรง หรืออ่านจากไฟล์ |
| `--generate-notes` | ให้ GitHub สร้าง release notes อัตโนมัติจาก PR ที่ merge เข้ามา |
| `--prerelease` | ทำเครื่องหมายว่าเป็น pre-release (alpha/beta/rc) |
| `--draft` | สร้างเป็น draft ก่อน ยังไม่เผยแพร่จริงจนกว่าจะกด publish |
| `--target` | ระบุ branch/commit ที่จะสร้าง tag ให้ (ถ้า tag ยังไม่มีอยู่จริง) |

### การเพิ่มไฟล์แนบเข้า Release ที่มีอยู่แล้ว

```bash
gh release upload v1.0.0 dist/myapp-macos-arm64.tar.gz
```

### แนวปฏิบัติที่ดีเรื่อง Binary/Asset

1. **ตั้งชื่อไฟล์ให้ระบุ platform/architecture ชัดเจน** เช่น `myapp-linux-amd64.tar.gz`, `myapp-darwin-arm64.tar.gz`, `myapp-windows-amd64.zip`
2. **แนบไฟล์ checksum (SHA256) เสมอ** เพื่อให้ผู้ใช้ตรวจสอบความสมบูรณ์ของไฟล์ที่ดาวน์โหลดได้

```bash
sha256sum dist/*.tar.gz dist/*.zip > dist/checksums.txt
gh release upload v1.0.0 dist/checksums.txt
```

3. **ไม่ควรพึ่งพา Source code (zip)/(tar.gz)** ที่ GitHub สร้างให้อัตโนมัติเป็น distribution หลัก เพราะมันคือซอร์สโค้ดดิบที่ยังไม่ได้ build — ควรแนบ binary ที่ build/compile เสร็จแล้วเพิ่มเติมเสมอสำหรับซอฟต์แวร์ที่ผู้ใช้ทั่วไปต้องติดตั้งใช้งาน
4. **ใช้ GitHub Actions ทำ build + upload asset อัตโนมัติ** เมื่อมีการ push tag ใหม่ (จะเรียนเรื่อง workflow เต็มรูปแบบใน Part ที่เกี่ยวกับ GitHub Actions Release automation)

### การดู Release ทั้งหมดของ Repository

```bash
gh release list
gh release view v1.0.0
```

---

## Step 876: GitLab Releases — เทียบเคียงด้วย `release:` Keyword

### GitLab Releases คืออะไร

**GitLab Releases** ทำหน้าที่คล้าย GitHub Releases มาก คือผูกกับ Tag และเก็บ release notes พร้อม asset link แต่จุดต่างสำคัญคือ GitLab ออกแบบให้การสร้าง Release เป็นส่วนหนึ่งของ **CI/CD Pipeline โดยตรง** ผ่านคีย์เวิร์ด `release:` ใน `.gitlab-ci.yml`

### โครงสร้างพื้นฐานของ `release:` keyword

```yaml
stages:
  - build
  - release

build-job:
  stage: build
  script:
    - echo "กำลัง build binary..."
    - mkdir -p dist
    - tar -czf dist/myapp-linux-amd64.tar.gz ./build/myapp
  artifacts:
    paths:
      - dist/

release-job:
  stage: release
  image: registry.gitlab.com/gitlab-org/release-cli:latest
  rules:
    - if: $CI_COMMIT_TAG                # ทำงานเฉพาะตอน push tag เท่านั้น
  script:
    - echo "กำลังสร้าง GitLab Release สำหรับ $CI_COMMIT_TAG"
  release:
    tag_name: '$CI_COMMIT_TAG'
    name: 'Release $CI_COMMIT_TAG'
    description: './CHANGELOG.md'
    assets:
      links:
        - name: 'myapp-linux-amd64.tar.gz'
          url: 'https://example.com/download/myapp-linux-amd64.tar.gz'
        - name: 'Checksums'
          url: 'https://example.com/download/checksums.txt'
```

จุดสำคัญของโครงสร้างนี้:

- **`rules: if: $CI_COMMIT_TAG`** — บังคับให้ job นี้ทำงานเฉพาะตอนที่ pipeline ถูก trigger จากการ push tag เท่านั้น (ไม่ใช่ทุกครั้งที่ push commit ปกติ)
- **`release:` block** — เป็นคีย์เวิร์ดพิเศษของ GitLab CI ที่บอกให้ runner เรียกใช้ `release-cli` เพื่อสร้าง Release object ผ่าน GitLab API โดยอัตโนมัติหลัง job รันสำเร็จ
- **`assets.links`** — ระบุลิงก์ไปยังไฟล์ asset ที่ต้องการแสดงในหน้า Release (มักชี้ไปยัง GitLab Package Registry, Generic Package Registry หรือ external storage เช่น S3)

### การอัปโหลด Asset จริงเข้า GitLab Generic Package Registry ก่อนสร้าง Release

ในทางปฏิบัติมักอัปโหลดไฟล์เข้า GitLab's Generic Package Registry ก่อน แล้วค่อยอ้างอิง URL นั้นใน `assets.links`:

```yaml
release-job:
  stage: release
  rules:
    - if: $CI_COMMIT_TAG
  script:
    - |
      curl --header "JOB-TOKEN: $CI_JOB_TOKEN" \
           --upload-file dist/myapp-linux-amd64.tar.gz \
           "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/generic/myapp/${CI_COMMIT_TAG}/myapp-linux-amd64.tar.gz"
  release:
    tag_name: '$CI_COMMIT_TAG'
    description: 'Release $CI_COMMIT_TAG'
    assets:
      links:
        - name: 'myapp-linux-amd64.tar.gz'
          url: '${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/generic/myapp/${CI_COMMIT_TAG}/myapp-linux-amd64.tar.gz'
```

### ตารางเปรียบเทียบ GitHub Releases vs GitLab Releases

| คุณสมบัติ | GitHub Releases | GitLab Releases |
|---|---|---|
| วิธีสร้างหลัก | หน้าเว็บ หรือ `gh` CLI | ผ่าน `.gitlab-ci.yml` โดยตรง (CI-native) หรือ GitLab CLI/API |
| ผูกกับอะไร | Git Tag | Git Tag |
| การแนบไฟล์ | Drag & drop หรือ `gh release upload` | ต้องอัปโหลดเข้า Package Registry ก่อน แล้วอ้างอิง URL ผ่าน `assets.links` |
| Automation ที่มากับแพลตฟอร์ม | GitHub Actions (แยกจาก release เป็น step หนึ่ง) | `release:` keyword ผูกอยู่ใน pipeline โดยตรง เป็น first-class citizen |
| เหมาะกับ | ทีมที่ผูกกับ GitHub Actions และต้องการ workflow ที่ยืดหยุ่นสูง | ทีมที่ต้องการให้ release เป็นส่วนหนึ่งของ pipeline แบบ end-to-end อัตโนมัติเต็มรูปแบบ |
| การแสดงผล Milestone | เชื่อมกับ GitHub Milestones/Projects | เชื่อมกับ GitLab Milestones ได้โดยตรงในหน้า Release |

### แนวคิดสำคัญ: ทำไม GitLab เลือกผูก Release เข้ากับ CI Pipeline

ปรัชญาของ GitLab คือมองว่า **"Release" เป็นเพียงหนึ่งใน stage ของ pipeline การส่งมอบซอฟต์แวร์** เหมือนกับ build, test, deploy — ไม่ใช่การกระทำแยกต่างหากที่ต้องมีคนมากดปุ่มบนหน้าเว็บเอง สิ่งนี้สอดคล้องกับแนวคิด **GitOps และ Continuous Delivery** ที่ทุกอย่างควรเกิดขึ้นอัตโนมัติเมื่อเงื่อนไขที่กำหนดไว้เป็นจริง (ในที่นี้คือ "เมื่อมีการ push tag ใหม่")

---

## Step 877: Automated Versioning ด้วย `semantic-release`

### ปัญหาที่ semantic-release แก้ไข

การตัดสินใจ bump version ด้วยมือมีปัญหาหลายอย่าง:

1. มนุษย์ลืมหรือตัดสินใจผิดพลาดว่าควร bump MAJOR, MINOR หรือ PATCH
2. การเขียน changelog ด้วยมือใช้เวลานานและไม่สม่ำเสมอ
3. ขั้นตอน tag → build → publish → create release ต้องทำหลายคำสั่งซ้ำ ๆ ทุกครั้ง เสี่ยงต่อความผิดพลาด

**`semantic-release`** คือเครื่องมือ (เริ่มต้นจาก ecosystem ของ JavaScript/npm แต่ตอนนี้มีการนำแนวคิดไปทำ port สำหรับภาษาอื่นด้วย เช่น Python, PHP) ที่ทำให้กระบวนการทั้งหมดนี้เป็นอัตโนมัติเต็มรูปแบบ โดยอาศัย **Conventional Commits** เป็นแหล่งข้อมูลในการตัดสินใจ

### เชื่อมโยงกับ Conventional Commits จาก Part 35

ใน Part 35 เราเรียนรู้ Conventional Commits specification ซึ่งกำหนด commit type มาตรฐาน เช่น `feat:`, `fix:`, `docs:`, `chore:` และการระบุ breaking change ด้วย `BREAKING CHANGE:` ใน footer หรือเครื่องหมาย `!` หลัง type

`semantic-release` ใช้กฎการแปล commit type เป็นระดับการ bump version ดังนี้ (ค่าเริ่มต้นตาม Angular preset ซึ่งเป็น preset มาตรฐานที่นิยมที่สุด):

| Commit Type | ผลต่อ Version |
|---|---|
| `fix:` | Bump **PATCH** |
| `feat:` | Bump **MINOR** |
| commit ที่มี `BREAKING CHANGE:` ใน footer หรือมี `!` ต่อท้าย type (เช่น `feat!:`) | Bump **MAJOR** |
| `docs:`, `style:`, `chore:`, `refactor:`, `test:`, `ci:` (ที่ไม่ใช่ breaking change) | **ไม่กระทบ version** โดยค่าเริ่มต้น |

ตัวอย่าง: ถ้านับตั้งแต่ release ล่าสุด repository มี commit ต่อไปนี้:

```
fix: แก้ปัญหา memory leak ใน connection pool
feat: เพิ่มระบบ export PDF
docs: อัปเดตตัวอย่างใน README
```

`semantic-release` จะวิเคราะห์ว่ามี `feat:` อยู่ (แต่ไม่มี breaking change) จึงตัดสินใจ bump **MINOR** โดยอัตโนมัติ เช่น จาก `1.4.2` เป็น `1.5.0`

### ขั้นตอนการทำงานของ `semantic-release` แบบเต็มรูปแบบ

`semantic-release` ทำงานเป็นขั้นตอนต่อเนื่อง (แต่ละขั้นคือ plugin ที่ทำงานเรียงกัน):

1. **`@semantic-release/commit-analyzer`** — วิเคราะห์ commit ทั้งหมดตั้งแต่ tag ล่าสุด เพื่อตัดสินใจว่ารอบนี้ควร bump MAJOR, MINOR, PATCH หรือไม่ต้อง release เลย
2. **`@semantic-release/release-notes-generator`** — สร้าง release notes จาก commit message โดยจัดกลุ่มตาม type อัตโนมัติ
3. **`@semantic-release/changelog`** — อัปเดตไฟล์ `CHANGELOG.md` ในโปรเจกต์
4. **`@semantic-release/npm`** (หรือ plugin เทียบเท่าสำหรับ package manager อื่น) — bump เลขเวอร์ชันใน `package.json` แล้ว publish ขึ้น npm registry
5. **`@semantic-release/git`** — commit ไฟล์ที่เปลี่ยนแปลง (`CHANGELOG.md`, `package.json`) กลับเข้า repository พร้อมสร้าง Git tag
6. **`@semantic-release/github`** หรือ **`@semantic-release/gitlab`** — สร้าง GitHub Release / GitLab Release อัตโนมัติ พร้อม comment แจ้งใน issue/PR ที่เกี่ยวข้องว่า "แก้ไขนี้ถูก release ไปแล้วในเวอร์ชันไหน"

### ตัวอย่างไฟล์ config `.releaserc.json`

```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/npm",
    ["@semantic-release/git", {
      "assets": ["CHANGELOG.md", "package.json"],
      "message": "chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}"
    }],
    "@semantic-release/github"
  ]
}
```

### ตัวอย่าง CI Workflow ที่รัน semantic-release อัตโนมัติ (GitHub Actions)

```yaml
name: Release
on:
  push:
    branches: [main]

permissions:
  contents: write
  issues: write
  pull-requests: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0    # ต้องดึงประวัติเต็มเพื่อให้ commit-analyzer วิเคราะห์ได้ถูกต้อง

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci

      - name: Run semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: npx semantic-release
```

ทุกครั้งที่มีการ merge เข้า `main` pipeline นี้จะทำงานอัตโนมัติ: วิเคราะห์ commit → ตัดสินใจเวอร์ชันใหม่ → สร้าง tag → publish package → สร้าง GitHub Release ทั้งหมดโดยไม่ต้องมีมนุษย์กดปุ่มใด ๆ เลย

### ข้อควรระวังสำคัญเมื่อใช้ semantic-release

1. **วินัยของ commit message เป็นหัวใจสำคัญที่สุด** — ถ้าทีมเขียน commit message ไม่ตรงตาม Conventional Commits spec เครื่องมือจะตัดสินใจ version ผิดพลาดทันที (เช่น ลืมใส่ `!` หรือ `BREAKING CHANGE:` ทั้งที่จริงมี breaking change จะทำให้ปล่อยเป็นแค่ MINOR ทั้งที่ควรเป็น MAJOR)
2. **กรณีใช้ Squash Merge บน Pull Request** ต้องตั้งค่าให้ชื่อ PR (ซึ่งจะกลายเป็น commit message เดียวหลัง squash) ตรงตาม Conventional Commits ด้วย ไม่ใช่แค่ commit ย่อยภายใน PR — เพราะ commit ย่อยจะถูกรวมหายไปหมดหลัง squash
3. **ควรรัน commit linting (เช่น `commitlint` จาก Part 35) ที่ pre-commit hook หรือ CI ก่อนอนุญาตให้ merge** เพื่อป้องกันไม่ให้ commit ที่ผิดรูปแบบหลุดเข้าไปใน `main` ตั้งแต่ต้นทาง
4. **ทดสอบด้วย dry-run ก่อนใช้งานจริง** ด้วยคำสั่ง `npx semantic-release --dry-run` เพื่อดูว่ามันจะตัดสินใจ version อะไรและสร้าง release notes แบบไหน โดยยังไม่ publish จริง

---

## Step 878: Release Cadence — Time-based vs Feature-based

### Release Cadence คืออะไร

**Release Cadence** คือ "จังหวะความถี่" ที่ทีมตัดสินใจปล่อยเวอร์ชันใหม่ออกสู่ผู้ใช้ เป็นการตัดสินใจเชิงกลยุทธ์ที่ส่งผลกระทบต่อทั้งคุณภาพซอฟต์แวร์ ความพึงพอใจของผู้ใช้ และภาระงานของทีม

มีสองแนวทางหลักที่ขั้วตรงข้ามกัน:

### แนวทางที่ 1: Time-based Release (ปล่อยตามกำหนดเวลาที่แน่นอน)

> "ไม่ว่าฟีเจอร์จะพร้อมกี่เปอร์เซ็นต์ เมื่อถึงวันที่กำหนดไว้ล่วงหน้า จะปล่อย release ทันที"

ตัวอย่างจริง: **Ubuntu** ปล่อยเวอร์ชันใหม่ทุก 6 เดือน (เมษายนและตุลาคมของทุกปี) เสมอ, **Chrome/Firefox** ใช้ release train ปล่อยทุก 4 สัปดาห์

**ข้อดี:**
- ผู้ใช้และทีมวางแผนล่วงหน้าได้แม่นยำ รู้ว่าเวอร์ชันถัดไปจะมาเมื่อไหร่
- สร้างวินัยให้ทีม — ฟีเจอร์ที่ไม่เสร็จทันจะถูกเลื่อนไปรอบถัดไปแทนที่จะดันเข้าไปแบบเร่งรีบ
- ลด "release ที่ใหญ่เกินไป" เพราะแต่ละรอบมีขนาดจำกัดตามเวลาที่มี

**ข้อเสีย:**
- ถ้าฟีเจอร์สำคัญไม่เสร็จทันกำหนดเวลา ต้องเลื่อนไปรอบถัดไปทั้งที่ผู้ใช้อาจรออยู่
- อาจกดดันทีมให้รีบส่งงานที่ยังไม่พร้อมจริง ๆ เพื่อให้ทันกำหนดเวลา (schedule pressure)

### แนวทางที่ 2: Feature-based Release (ปล่อยเมื่อฟีเจอร์พร้อม)

> "ไม่มีกำหนดเวลาตายตัว จะปล่อย release ก็ต่อเมื่อฟีเจอร์/ชุดของการแก้ไขนั้นพร้อมสมบูรณ์แล้วเท่านั้น"

**ข้อดี:**
- คุณภาพของแต่ละ release สูง เพราะไม่มีการรีบส่งงานที่ยังไม่พร้อม
- แต่ละ release มีเนื้อหาที่ "จบในตัวเอง" ชัดเจน ไม่ปนกันระหว่างฟีเจอร์ที่เกี่ยวข้องกันครึ่ง ๆ กลาง ๆ

**ข้อเสีย:**
- คาดเดายากว่าเวอร์ชันถัดไปจะออกเมื่อไหร่ ทำให้ผู้ใช้และทีม sales/marketing วางแผนล่วงหน้ายาก
- เสี่ยงต่อการ "รอให้สมบูรณ์แบบ" จนไม่ยอมปล่อยสักที (perfectionism trap)

### ตารางเปรียบเทียบปัจจัยที่ควรใช้ตัดสินใจ

| ปัจจัย | เอนไปทาง Time-based | เอนไปทาง Feature-based |
|---|---|---|
| ขนาดทีมและจำนวนผู้ใช้ | ทีมใหญ่ ผู้ใช้จำนวนมาก ต้องการความสามารถคาดการณ์ได้ | ทีมเล็ก ผู้ใช้จำกัด ปรับตัวไวได้ |
| ประเภทซอฟต์แวร์ | Mobile app ที่ต้องผ่านกระบวนการ review ของ App Store/Play Store (ยิ่งต้องมีตารางเวลาชัดเจน) | Internal tool หรือ SaaS ที่ deploy เองได้ทันที |
| งบประมาณ QA | มี QA team และ automated test ที่แข็งแรงพอจะรองรับรอบทดสอบสม่ำเสมอ | มีทรัพยากรทดสอบจำกัด ต้องการเวลาพิเศษเมื่อฟีเจอร์ใหญ่เสร็จ |
| ความคาดหวังของลูกค้า enterprise | ลูกค้ามักต้องการ release schedule ที่แน่นอนเพื่อวางแผน IT ของตัวเอง | ลูกค้าปลายทางเป็นผู้บริโภคทั่วไปที่ไม่สนใจตารางเวลา |

### แนวทางผสม (Hybrid) ที่นิยมมากที่สุดในปัจจุบัน

ทีมสมัยใหม่จำนวนมากเลือกใช้แนวทางผสมที่เรียกว่า **"Time-based cadence + Feature Flags"**:

> กำหนดจังหวะ MINOR release ตามเวลาที่แน่นอน (เช่น ทุก 2 สัปดาห์) แต่ฟีเจอร์ที่ยังไม่เสร็จสมบูรณ์จะถูก merge เข้า `main` โดยซ่อนไว้หลัง **feature flag** (ปิดการมองเห็นไว้ก่อน) เพื่อไม่ให้กระทบผู้ใช้ เมื่อฟีเจอร์พร้อมจริงค่อยเปิด flag ในภายหลังโดยไม่ต้องรอรอบ release ถัดไป

วิธีนี้แยก **"การ deploy โค้ด" ออกจาก "การเปิดตัวฟีเจอร์ให้ผู้ใช้เห็น"** อย่างสิ้นเชิง ทำให้ทีมได้ประโยชน์ทั้งสองฝั่ง:

- ได้ความสม่ำเสมอของ time-based release (คาดการณ์ได้)
- ได้ความยืดหยุ่นของ feature-based release (ไม่บังคับดันฟีเจอร์ที่ยังไม่พร้อม)

### ตัวอย่างจังหวะ Release Cadence ที่พบบ่อยในโลกจริง

| จังหวะ | ตัวอย่างองค์กร/โปรเจกต์ | เหมาะกับ |
|---|---|---|
| ทุกวัน/หลายครั้งต่อวัน (Continuous Deployment) | บริษัท SaaS ขนาดใหญ่ที่มี test automation แข็งแรงมาก | ทีมที่ deploy ผ่าน pipeline อัตโนมัติเต็มรูปแบบ |
| ทุก 2 สัปดาห์ (Sprint-based) | ทีมที่ใช้ Scrum/Agile sprint | ทีมพัฒนาซอฟต์แวร์ทั่วไปขนาดกลาง |
| ทุก 4-6 สัปดาห์ | Browser (Chrome, Firefox), หลาย framework | โปรเจกต์ที่ต้องการความสม่ำเสมอแต่ไม่เร่งรีบเกินไป |
| ทุก 6 เดือน หรือปีละครั้ง | Ubuntu LTS, Enterprise software รายใหญ่ | ซอฟต์แวร์ที่ลูกค้าองค์กรต้องวางแผน IT ล่วงหน้านาน |

---

## Step 879: Deprecation Policy — การแจ้งเลิกใช้ฟีเจอร์อย่างมีความรับผิดชอบ

### Deprecation คืออะไร และทำไมต้องมีนโยบายชัดเจน

**Deprecation (การเลิกใช้งาน)** คือกระบวนการประกาศว่าฟีเจอร์/API หนึ่งจะถูก**ลบออกในอนาคต** แต่**ยังคงใช้งานได้ตามปกติในตอนนี้** เพื่อให้ผู้ใช้มีเวลาเตรียมตัวย้ายไปใช้ทางเลือกอื่นก่อนที่ของเดิมจะหายไปจริง

การลบฟีเจอร์ทันทีโดยไม่เคย deprecate มาก่อนคือการทำ breaking change แบบไม่ให้เกียรติผู้ใช้เลย และจะทำลายความไว้วางใจต่อโปรเจกต์อย่างรุนแรง

### ความเชื่อมโยงกับ SemVer ที่ต้องจำให้แม่น (ย้อนกลับไป Step 871)

จำกฎข้อ 7 ของสเปก SemVer ที่กล่าวไว้ใน Step 871:

> **"MUST be incremented if any public API functionality is marked as deprecated"**

พูดง่าย ๆ คือ **การประกาศ deprecate ฟีเจอร์หนึ่ง ต้อง bump MINOR version เสมอ** แม้ว่าฟีเจอร์นั้นจะยังใช้งานได้ปกติทุกประการก็ตาม เพราะการ deprecate คือการเปลี่ยนแปลง "สัญญา" ที่ให้ไว้กับผู้ใช้ (บอกว่า "สิ่งนี้จะหายไปในอนาคต") ซึ่งนับเป็นการเปลี่ยนแปลงที่ผู้ใช้ควรรับรู้ผ่านเลขเวอร์ชัน

### วงจรชีวิตมาตรฐานของการ Deprecate ฟีเจอร์ (Deprecation Lifecycle)

```
ขั้นที่ 1: ประกาศ (Announce)
   ↓
ขั้นที่ 2: เตือนอย่างต่อเนื่อง (Warn) — ยังใช้งานได้ปกติ
   ↓
ขั้นที่ 3: ระยะเปลี่ยนผ่าน (Grace Period) — ให้เวลาย้ายออก
   ↓
ขั้นที่ 4: ลบออกจริง (Remove) — ใน MAJOR version ถัดไป
```

#### ขั้นที่ 1: ประกาศ (Announce)

เมื่อทีมตัดสินใจว่าฟีเจอร์หนึ่งควรถูกเลิกใช้ ต้องประกาศให้ชัดเจนในหลายช่องทางพร้อมกัน:

- Release notes ของเวอร์ชันที่เริ่ม deprecate (ตามโครงสร้างที่เรียนใน Step 874)
- Documentation อัปเดตทันที ระบุคำว่า "Deprecated" กำกับไว้ชัดเจน
- Changelog บันทึกไว้ในหมวด "Deprecated" (ตามแนวทาง Keep a Changelog)

#### ขั้นที่ 2: เตือนอย่างต่อเนื่อง (Warn) ผ่านโค้ดจริง

ฟีเจอร์ที่ถูก deprecate ควรแสดงคำเตือนขณะรันไทม์ เพื่อให้ผู้ใช้ที่อาจไม่เคยอ่าน release notes ก็ยังรับรู้ได้

ตัวอย่างใน JavaScript:

```javascript
function oldExportCSV(data) {
  console.warn(
    "[DEPRECATED] oldExportCSV() จะถูกลบใน v3.0.0 " +
    "กรุณาใช้ exportReport({ format: 'csv' }) แทน " +
    "ดูรายละเอียดที่ https://example.com/migration-guide"
  );
  return exportReport({ format: "csv", data });
}
```

ตัวอย่างใน Python (ใช้ built-in `DeprecationWarning`):

```python
import warnings

def old_export_csv(data):
    warnings.warn(
        "old_export_csv() จะถูกลบใน v3.0.0 กรุณาใช้ export_report(format='csv') แทน",
        DeprecationWarning,
        stacklevel=2
    )
    return export_report(data, format="csv")
```

ตัวอย่างใน REST API (ผ่าน HTTP Header):

```
HTTP/1.1 200 OK
Deprecation: true
Sunset: Sat, 01 Aug 2026 00:00:00 GMT
Link: <https://example.com/migration-guide>; rel="deprecation"
```

การใช้ HTTP header มาตรฐานอย่าง `Deprecation` และ `Sunset` (ตาม RFC ที่เกี่ยวข้อง) ช่วยให้ระบบอัตโนมัติของผู้ใช้สามารถตรวจจับและแจ้งเตือนทีมของเขาได้โดยไม่ต้องอ่านเอกสารด้วยมือ

#### ขั้นที่ 3: ระยะเปลี่ยนผ่าน (Grace Period) — ควรนานแค่ไหน

ไม่มีตัวเลขตายตัวที่ถูกต้องเสมอไป แต่หลักการทั่วไปที่ใช้กันในอุตสาหกรรม:

| ประเภทซอฟต์แวร์ | ระยะเวลา Grace Period ที่แนะนำ |
|---|---|
| Library/Package ทั่วไป | อย่างน้อย 1 รอบ MINOR release เต็ม ก่อนจะลบใน MAJOR ถัดไป |
| REST API สาธารณะที่มีผู้ใช้ภายนอกจำนวนมาก | อย่างน้อย 6 เดือน – 1 ปี |
| Internal tool ที่ทีมควบคุมผู้ใช้ทั้งหมดเอง | สั้นกว่าได้ (เช่น 1-2 สัปดาห์) เพราะสื่อสารตรงกับผู้ใช้ได้ง่าย |
| Enterprise software ที่มีสัญญา SLA | ต้องระบุระยะเวลาชัดเจนไว้ใน**สัญญา**หรือ Support Policy อย่างเป็นทางการ |

หลักการสำคัญที่สุด: **ต้องระบุ "จะลบเมื่อไหร่" อย่างชัดเจนตั้งแต่วันที่ประกาศ deprecate** ไม่ใช่ปล่อยให้เป็นคำเตือนคลุมเครือแบบ "จะลบในอนาคต" โดยไม่มีกำหนดเวลา — เพราะผู้ใช้จะไม่สามารถวางแผนงานของตัวเองได้เลยถ้าไม่รู้เส้นตาย

#### ขั้นที่ 4: ลบออกจริง (Remove) — ต้องเกิดขึ้นพร้อม MAJOR version เท่านั้น

เมื่อครบระยะเปลี่ยนผ่านแล้ว การลบฟีเจอร์ที่เคย deprecate ไว้คือ **breaking change** ตามนิยามของ SemVer เสมอ (แม้จะเตือนมานานแค่ไหนก็ตาม) จึงต้อง bump **MAJOR version** เท่านั้น และต้องระบุไว้ในหมวด **Breaking Changes** ของ release notes อย่างเด่นชัดที่สุด

### Template ข้อความ Deprecation Notice ที่ครบถ้วน

```
[DEPRECATED] `<ชื่อฟีเจอร์/ฟังก์ชัน>` จะถูกลบใน <เวอร์ชันที่จะลบ, เช่น v3.0.0>
(คาดว่าจะปล่อยประมาณ <เดือน/ปี>)

เหตุผล: <อธิบายสั้น ๆ ว่าทำไมถึง deprecate>

สิ่งที่ควรทำแทน: <ระบุฟังก์ชัน/วิธีใหม่ที่ควรใช้แทน พร้อมตัวอย่างโค้ด>

รายละเอียดเพิ่มเติม: <ลิงก์ migration guide>
```

### ตัวอย่างประกาศ Deprecation Policy ระดับโปรเจกต์ (เขียนไว้ใน README หรือเอกสาร)

```markdown
## Deprecation Policy

โปรเจกต์นี้ยึดหลัก Semantic Versioning อย่างเคร่งครัด:

- ฟีเจอร์ที่ถูก deprecate จะถูกประกาศไว้ล่วงหน้าอย่างน้อย 2 รอบ MINOR release
  ก่อนจะถูกลบออกจริงใน MAJOR release ถัดไป
- ทุกการ deprecate จะถูกบันทึกไว้ใน CHANGELOG.md ในหมวด "Deprecated" เสมอ
- ฟังก์ชันที่ deprecate แล้วจะแสดง runtime warning ทุกครั้งที่ถูกเรียกใช้
- เราจะไม่ลบฟีเจอร์ใด ๆ โดยไม่ผ่านกระบวนการ deprecate มาก่อน ยกเว้นกรณีช่องโหว่
  ความปลอดภัยร้ายแรงที่จำเป็นต้องแก้ไขทันที (จะแจ้งแยกต่างหากผ่านช่องทาง Security Advisory)
```

---

## Step 880: แบบฝึกหัด — วางแผนและทำ Release v1.0.0 แบบครบวงจร

ถึงเวลาลงมือปฏิบัติจริง โดยรวบรวมทุกสิ่งที่เรียนมาตลอด Part นี้เข้าด้วยกัน: SemVer, release branch, release notes, และ GitHub Releases

### เป้าหมายของแบบฝึกหัด

จำลองสถานการณ์ว่าคุณกำลังจะปล่อย **v1.0.0** เวอร์ชันแรกของโปรเจกต์ที่พัฒนามาระยะหนึ่งแล้ว (อยู่ในช่วง `0.x.x`) และพร้อมประกาศว่า public API เริ่มนิ่งแล้ว

### ขั้นที่ 1: เตรียมโปรเจกต์ทดลอง

```bash
mkdir ~/git-course/part-88-release
cd ~/git-course/part-88-release
git init
```

สร้างไฟล์เริ่มต้นและจำลองประวัติการพัฒนาเป็นเวอร์ชัน `0.x.x`:

```bash
echo "# My Awesome Tool" > README.md
git add README.md
git commit -m "chore: initial project setup"
git tag -a v0.1.0 -m "Initial prototype"

echo "function login() {}" > app.js
git add app.js
git commit -m "feat: add login function"
git tag -a v0.2.0 -m "Add login prototype"

echo "function logout() {}" >> app.js
git add app.js
git commit -m "feat: add logout function"

echo "fix login bug" >> app.js
git add app.js
git commit -m "fix: correct login validation error"
```

### ขั้นที่ 2: ตัดสินใจว่าพร้อมเป็น v1.0.0 หรือยัง (Checklist การตัดสินใจ)

ก่อนติด tag `v1.0.0` ให้ตอบคำถามเหล่านี้ให้ครบก่อน (ตามหลักการจาก Step 871):

- [ ] Public API ของโปรเจกต์ถูกนิยามไว้ชัดเจนหรือยัง (ฟังก์ชัน/endpoint ไหนคือของสาธารณะที่สัญญาว่าจะรักษาความเข้ากันได้)
- [ ] มั่นใจหรือยังว่า API ที่มีอยู่ตอนนี้นิ่งพอ ไม่ต้องเปลี่ยนแบบ breaking change ในเร็ว ๆ นี้
- [ ] มี test coverage เพียงพอที่จะมั่นใจว่าไม่มีบั๊กร้ายแรงหลงเหลือ
- [ ] มี documentation พื้นฐานพร้อมให้ผู้ใช้เริ่มต้นใช้งานจริง

### ขั้นที่ 3: สร้าง Release Branch (ทางเลือก สำหรับจำลอง flow แบบมืออาชีพ)

```bash
git checkout -b release/1.0.0
```

ที่ release branch นี้ ทำการ "แช่แข็ง" ฟีเจอร์ ปรับปรุงเอกสารครั้งสุดท้าย และแก้บั๊กเล็กน้อยที่พบระหว่างทดสอบ:

```bash
echo "## Installation" >> README.md
echo "npm install my-awesome-tool" >> README.md
git add README.md
git commit -m "docs: add installation instructions for v1.0.0"
```

### ขั้นที่ 4: เขียน CHANGELOG/Release Notes สำหรับ v1.0.0

สร้างไฟล์ `CHANGELOG.md` ตามโครงสร้างที่เรียนใน Step 874:

```markdown
## v1.0.0 — 2026-09-26

### สรุปโดยย่อ
เวอร์ชันแรกที่เสถียรของ My Awesome Tool ประกาศ Public API อย่างเป็นทางการ
พร้อมสัญญาว่าจะรักษา backward compatibility ตามหลัก Semantic Versioning
ตั้งแต่เวอร์ชันนี้เป็นต้นไป

### ฟีเจอร์หลัก
- ระบบ login/logout พื้นฐาน
- เอกสารการติดตั้งฉบับสมบูรณ์

### หมายเหตุ
นี่คือเวอร์ชันแรกที่ยึดหลัก Semantic Versioning อย่างเป็นทางการ
ดู Deprecation Policy และแนวทาง versioning ได้ที่ README.md
```

```bash
git add CHANGELOG.md
git commit -m "docs: add CHANGELOG for v1.0.0"
```

### ขั้นที่ 5: Merge กลับเข้า main แล้วติด Tag

```bash
git checkout main
git merge release/1.0.0 --no-ff -m "chore: merge release/1.0.0 into main"

# สร้าง Annotated Tag ตามหลักที่เรียนใน Part 11 (ต้องเป็น Annotated เสมอสำหรับ release จริง)
git tag -a v1.0.0 -m "First stable release: public API frozen, follows Semantic Versioning"
```

### ขั้นที่ 6: ตรวจสอบความถูกต้องของ Tag ก่อน Push

```bash
git show v1.0.0
git tag --sort=v:refname
```

ตรวจสอบว่า Tag ชี้ไปยัง commit ที่ถูกต้อง และเป็น Annotated Tag (มี Tagger, Date, Message ครบถ้วน) ตามที่เรียนใน Part 11

### ขั้นที่ 7: Push ทั้ง Branch และ Tag ขึ้น Remote

```bash
git push origin main
git push origin v1.0.0
```

### ขั้นที่ 8: สร้าง GitHub Release พร้อมแนบไฟล์

```bash
mkdir -p dist
tar -czf dist/my-awesome-tool-v1.0.0.tar.gz app.js README.md

gh release create v1.0.0 \
  dist/my-awesome-tool-v1.0.0.tar.gz \
  --title "v1.0.0 — First Stable Release" \
  --notes-file CHANGELOG.md
```

หรือถ้าไม่ได้ใช้ `gh` CLI ให้ทำผ่านหน้าเว็บ GitHub ตามขั้นตอนใน Step 875: เลือก tag `v1.0.0` ที่มีอยู่แล้ว, ใส่ title, วางเนื้อหาจาก `CHANGELOG.md` ลงในช่อง release notes, แนบไฟล์ `dist/my-awesome-tool-v1.0.0.tar.gz`, แล้วกด Publish release

### ขั้นที่ 9: ตรวจสอบผลลัพธ์สุดท้าย

```bash
gh release view v1.0.0
git ls-remote --tags origin
```

ตรวจสอบว่า:
- Release แสดงบนหน้า GitHub พร้อม release notes ที่อ่านเข้าใจง่าย
- ไฟล์ asset ดาวน์โหลดได้จริง
- Tag `v1.0.0` ปรากฏบน remote และชี้ไปยัง commit ที่ถูก merge เข้า `main` แล้ว

### ขั้นที่ 10: วางแผนสำหรับอนาคต

เขียนสรุปสั้น ๆ ไว้ในโปรเจกต์ (เช่นในไฟล์ `CONTRIBUTING.md` หรือ `README.md`) ว่าต่อจากนี้ทีมจะ:

- ใช้ release cadence แบบไหน (time-based ทุกกี่สัปดาห์ หรือ feature-based)
- ใช้ release branch strategy แบบไหนเมื่อโปรเจกต์โตขึ้น
- มี deprecation policy ระบุระยะเวลา grace period เท่าไหร่
- จะเริ่มใช้ automated versioning ด้วย `semantic-release` เมื่อใด

การเขียนแผนเหล่านี้ไว้ล่วงหน้าตั้งแต่ v1.0.0 จะช่วยให้ทีมมีมาตรฐานเดียวกันตั้งแต่ต้น ไม่ต้องมาถกเถียงกันใหม่ทุกครั้งที่จะ release เวอร์ชันถัดไป

---

## สรุป Part 88

ใน Part นี้เราได้เจาะลึก Release Management และ Semantic Versioning แบบเต็มรูปแบบ:

1. ทบทวนและเจาะลึกสเปก SemVer 2.0.0 ทั้ง 11 ข้อจาก semver.org อย่างถูกต้องครบถ้วน ตั้งแต่การนิยาม public API ไปจนถึงกฎการ bump MAJOR/MINOR/PATCH ที่แม่นยำ
2. เข้าใจกฎเต็มรูปแบบของ pre-release identifier และ build metadata รวมถึงอัลกอริทึมการเปรียบเทียบ precedence ที่ซับซ้อน
3. เชื่อมโยง Release Branch Strategy เข้ากับ Git Flow จาก Part 32 และเรียนรู้การดูแล long-term maintenance branch หลายสายพร้อมกัน
4. เรียนรู้หลักการเขียน Release Notes ที่ดี โดยคำนึงถึงผู้อ่านแต่ละกลุ่มที่แตกต่างกัน
5. ลงมือสร้าง GitHub Release และ GitLab Release จริง ทั้งผ่านหน้าเว็บ, CLI และ CI/CD pipeline
6. เข้าใจการทำ Automated Versioning ด้วย `semantic-release` ที่เชื่อมโยงกับ Conventional Commits จาก Part 35
7. เข้าใจปัจจัยในการตัดสินใจ Release Cadence ระหว่าง time-based, feature-based และแนวทางผสมด้วย feature flags
8. เข้าใจ Deprecation Policy และวงจรชีวิตของการเลิกใช้ฟีเจอร์อย่างมีความรับผิดชอบ
9. ลงมือทำ Release v1.0.0 แบบครบวงจรตั้งแต่ tag จนถึง GitHub Release จริง

### Checklist ก่อนไป Part ถัดไป

- [ ] อธิบายกฎทั้ง 11 ข้อของ SemVer 2.0.0 ได้อย่างถูกต้อง
- [ ] เข้าใจกฎการเปรียบเทียบ precedence ของ pre-release version ได้ (เช่นทำไม `1.0.0-beta.2` < `1.0.0-beta.11`)
- [ ] เลือก Release Branch Strategy ที่เหมาะกับโปรเจกต์ของตัวเองได้
- [ ] เขียน Release Notes ที่มีคุณภาพ แยกตามกลุ่มผู้อ่านได้
- [ ] สร้าง GitHub Release และ GitLab Release ได้ทั้งแบบ manual และแบบ automation
- [ ] เข้าใจหลักการทำงานของ `semantic-release` และความเชื่อมโยงกับ Conventional Commits
- [ ] ตัดสินใจ release cadence ที่เหมาะกับทีมของตัวเองได้อย่างมีเหตุผล
- [ ] เขียน deprecation notice ที่ครบถ้วนตามหลักการที่ถูกต้อง
- [ ] ผ่านแบบฝึกหัด release v1.0.0 แบบครบวงจรด้วยตัวเองสำเร็จ

**ต่อไป:** [Part 89: Changelog Automation และ Release Notes](./part-089-changelog-automation.md)
