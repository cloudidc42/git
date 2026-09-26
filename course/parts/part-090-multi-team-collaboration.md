# Part 90: Multi-team Collaboration ด้วย Git ในองค์กรขนาดใหญ่

> **Step ในหลักสูตรนี้:** Step 891–900
> **เฟส:** 9 — ทักษะมืออาชีพ: Maintainer, Release Management, Metrics
> **เป้าหมายของ Part นี้:** เข้าใจความท้าทายที่เกิดขึ้นเมื่อองค์กรโตจนมีหลายทีมทำงานบน codebase เดียวกันหรือ codebase ที่พึ่งพากันอย่างใกล้ชิด และเรียนรู้แนวปฏิบัติที่องค์กรระดับโลกใช้จริงในการประสานงานข้ามทีมผ่าน Git — ตั้งแต่โมเดล Platform/Product team, API contract, cross-team code review, shared library governance, การสื่อสารเรื่อง breaking change, feature flag สำหรับ coordinate release, incident response ข้ามทีม ไปจนถึงการใช้เอกสาร (โดยเฉพาะ ADR) เป็นเครื่องมือสื่อสารหลักขององค์กรวิศวกรรมขนาดใหญ่

---

## สารบัญของ Part นี้

- Step 891: ความท้าทายเมื่อหลายทีมทำงานบน codebase เดียวกันหรือที่เกี่ยวข้องกันอย่างใกล้ชิด
- Step 892: Platform team vs Product team model
- Step 893: API contract และ versioning ระหว่างทีม
- Step 894: Cross-team code review policy
- Step 895: Shared library/component management ข้ามทีม
- Step 896: Communication protocol เมื่อจะทำ breaking change ที่กระทบทีมอื่น
- Step 897: Feature flags สำหรับ coordinate release ข้ามทีม
- Step 898: Incident response ข้ามทีมเมื่อเกิดปัญหาจาก dependency ที่ใช้ร่วมกัน
- Step 899: Documentation เป็นเครื่องมือสื่อสารข้ามทีมที่สำคัญที่สุด (ADR)
- Step 900: แบบฝึกหัด — ออกแบบ collaboration model ให้องค์กรสมมติที่มี 5 ทีม

---

## Step 891: ความท้าทายเมื่อหลายทีมทำงานบน codebase เดียวกันหรือที่เกี่ยวข้องกันอย่างใกล้ชิด

จนถึง Part ที่แล้ว หลักสูตรนี้พูดถึง Git workflow ในบริบทของ "ทีมเดียว" หรือ "โปรเจกต์เดียว" เป็นหลัก — Git Flow, GitHub Flow, Trunk-Based Development, Code Review, CI/CD ล้วนออกแบบมาให้ใช้ได้ดีเมื่อมีทีมขนาด 5–15 คนดูแล repository เดียว

แต่ในความเป็นจริงขององค์กรที่เติบโตขึ้น สถานการณ์จะเปลี่ยนไปอย่างสิ้นเชิง เมื่อบริษัทมีวิศวกร 50, 200, หรือ 2,000 คน แบ่งเป็นหลายสิบทีม ปัญหาที่เกิดขึ้นจะไม่ใช่ "ทีมเราทำงานกันอย่างไร" อีกต่อไป แต่กลายเป็น **"หลายทีมที่ไม่ได้นั่งข้างกัน ไม่ได้อยู่ใน stand-up เดียวกัน จะทำงานร่วมกันบนโค้ดที่เกี่ยวข้องกันได้อย่างไรโดยไม่ทำลายกันเอง"**

### 891.1 รูปแบบของ "หลายทีมบน codebase เดียวกัน" ที่พบบ่อย

ในทางปฏิบัติ ความสัมพันธ์ระหว่างทีมกับ codebase มักออกมาในรูปแบบใดรูปแบบหนึ่งต่อไปนี้ (หรือผสมกัน):

1. **Monorepo หลายทีมใช้ร่วมกัน** — ทีม Frontend, Backend, Mobile, Data ทั้งหมดอยู่ใน repository เดียวกัน แชร์ CI pipeline เดียวกัน แชร์ build system เดียวกัน
2. **Polyrepo ที่พึ่งพากันผ่าน API/library** — แต่ละทีมมี repository ของตัวเอง แต่ service ของทีม A เรียก API ของทีม B หรือ import library ที่ทีม C เป็นเจ้าของ
3. **Shared core / shared platform** — มีทีมกลาง (Platform team) ดูแล core library, design system, infrastructure ที่ทุกทีมอื่นต้องพึ่งพา
4. **Modular monolith** — โค้ดอยู่ใน repository เดียว แต่แบ่งเป็น module ที่แต่ละทีมเป็นเจ้าของ module ของตัวเอง (CODEOWNERS แบ่งตาม path)

ไม่ว่าจะเป็นรูปแบบไหน หัวใจของปัญหาคือ **"การเปลี่ยนแปลงของทีมหนึ่ง อาจกระทบกับทีมอื่นที่ไม่รู้ตัวล่วงหน้า"**

### 891.2 ปัญหาหลัก 7 ประการที่เกิดขึ้นจริง

**1. Merge conflict ข้ามทีมที่แก้ยากกว่าปกติมาก**

เมื่อทีม A และทีม B ต่างแก้ไขไฟล์ที่ overlap กัน (เช่น shared configuration, shared schema) โดยไม่รู้ว่าอีกทีมกำลังแก้อยู่ conflict ที่เกิดขึ้นจะแก้ยากกว่าภายในทีมเดียวกันมาก เพราะคนที่ resolve conflict ไม่เข้าใจ context ของอีกฝั่งอย่างละเอียด

**2. Breaking change ที่ไม่มีใครแจ้งล่วงหน้า**

ทีม Backend เปลี่ยนชื่อ field ใน API response จาก `user_id` เป็น `userId` เพื่อความสอดคล้อง แต่ทีม Mobile ที่ deploy แอปไปแล้วหลายเวอร์ชันไม่รู้เรื่อง แอปพัง production ทันทีที่ deploy API ใหม่

**3. Ownership ที่ไม่ชัดเจน**

โค้ดบางส่วนไม่มีใครรู้ว่า "ใครเป็นเจ้าของจริง" — เมื่อเกิดบั๊ก ทุกทีมชี้นิ้วใส่กันว่าไม่ใช่หน้าที่ตัวเอง เวลาผ่านไปหลายวันก่อนที่บั๊กจะถูกแก้

**4. Release ที่ต้อง coordinate กันหลายทีมพร้อมกัน**

ฟีเจอร์ใหม่ต้องการให้ Backend, Frontend, Mobile deploy พร้อมกันในเวลาเดียวกัน (หรือตามลำดับที่ถูกต้อง) ถ้าไม่มีกระบวนการที่ชัดเจน การ deploy ผิดลำดับจะทำให้ระบบพังในช่วงเปลี่ยนผ่าน

**5. Code review กลายเป็นคอขวด**

ถ้าทุกการเปลี่ยนแปลงในไฟล์ shared ต้องรอ approve จากทีมกลางที่มีคนไม่กี่คน แต่มี 20 ทีมส่ง PR เข้ามาพร้อมกัน ทีมกลางจะกลายเป็นคอขวดของทั้งองค์กร

**6. Knowledge silo และ "bus factor" ระดับองค์กร**

แต่ละทีมรู้แค่ส่วนของตัวเอง ไม่มีใครเห็นภาพรวมทั้งระบบ เมื่อมีปัญหาที่กระทบหลายส่วน ไม่มีใครสามารถวิเคราะห์ root cause ได้ครบถ้วนคนเดียว

**7. Incident ที่ลามข้ามทีมและหาต้นตอยาก**

เมื่อระบบ production ล่ม สาเหตุอาจมาจาก dependency ที่ทีมอื่นดูแล การหาว่า "ใครต้องรับผิดชอบแก้" และ "ใครต้องเข้าร่วม war room" กลายเป็นความสับสนที่ทำให้ MTTR (Mean Time To Recovery) ยืดยาวออกไป

### 891.3 ทำไม Git และ workflow ที่ดีอย่างเดียวไม่พอ

สิ่งสำคัญที่ต้องเข้าใจตั้งแต่ต้น Part นี้คือ **Git เป็นเพียงเครื่องมือ ไม่ใช่คำตอบทั้งหมด** การแก้ปัญหา multi-team collaboration ต้องอาศัย 3 องค์ประกอบไปพร้อมกัน:

```
┌─────────────────────────────────────────────┐
│         Multi-team Collaboration              │
│                                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  ORG     │  │ PROCESS  │  │  TOOLING │   │
│  │ (โครงสร้าง│  │ (กระบวนการ│  │ (Git,    │   │
│  │  ทีม)    │  │  สื่อสาร) │  │  CI/CD)  │   │
│  └──────────┘  └──────────┘  └──────────┘   │
│                                                │
└─────────────────────────────────────────────┘
```

- **Org (โครงสร้างองค์กร)** — ทีมถูกแบ่งอย่างไร ใครดูแลอะไร (Step 892)
- **Process (กระบวนการ)** — กฎการ review, การแจ้ง breaking change, RFC (Step 894, 896)
- **Tooling** — CODEOWNERS, feature flags, CI ที่ตรวจ contract อัตโนมัติ (Step 893, 895, 897)

ตลอด Part นี้ เราจะเจาะลึกทั้ง 3 มิติ เพื่อให้คุณสามารถออกแบบและเข้าร่วม collaboration model ในองค์กรขนาดใหญ่ได้อย่างมืออาชีพ

### 891.4 กฎทองข้อแรกของ Multi-team Collaboration

ก่อนเข้า Step อื่น ให้จำหลักการนี้ไว้เป็นแกนกลางของทั้ง Part:

> **"ยิ่งการเปลี่ยนแปลงของคุณกระทบคนไกลตัวมากเท่าไหร่ ต้นทุนในการสื่อสารล่วงหน้าต้องสูงขึ้นตามเท่านั้น — การเปลี่ยนแปลงภายในทีมตัวเองอาจแค่ commit แล้ว merge แต่การเปลี่ยนแปลงที่กระทบทีมอื่นต้องผ่านกระบวนการแจ้งเตือนและขอความเห็นชอบที่เป็นระบบ"**

หลักการนี้จะปรากฏซ้ำในทุก Step ของ Part นี้ในรูปแบบที่ต่างกัน

---

## Step 892: Platform team vs Product team model

หนึ่งในโมเดลโครงสร้างทีมที่องค์กรวิศวกรรมขนาดใหญ่ (Spotify, Netflix, Amazon, Google) ใช้กันอย่างแพร่หลายที่สุดคือการแบ่งทีมออกเป็นสองประเภทหลักที่มีบทบาทต่างกันชัดเจน

### 892.1 Product Team คืออะไร

**Product team** (บางองค์กรเรียก "Stream-aligned team" ตามแนวคิด Team Topologies) คือทีมที่:

- รับผิดชอบ **feature ที่ส่งมอบคุณค่าให้ผู้ใช้ปลายทางโดยตรง** (end user หรือ business)
- วัดผลด้วย metric ทางธุรกิจ เช่น conversion rate, user engagement, revenue
- ทำงานเร็ว ต้อง ship บ่อย ตอบสนอง requirement ที่เปลี่ยนแปลงตลอดเวลา
- ตัวอย่าง: ทีม Checkout, ทีม Search, ทีม Onboarding, ทีม Notification

### 892.2 Platform Team คืออะไร

**Platform team** คือทีมที่:

- สร้าง **เครื่องมือ, infrastructure, library, internal service** ที่ทีม Product ใช้เป็นฐานในการทำงาน
- ไม่ได้สัมผัสผู้ใช้ปลายทางโดยตรง แต่ผู้ใช้ของ Platform team คือ **นักพัฒนาในทีม Product เอง** (จึงมักเรียกว่า "Developer Experience" หรือ "Internal Developer Platform")
- วัดผลด้วย metric เช่น deployment frequency ของทีมอื่น, lead time, จำนวน incident ที่ลดลง, developer satisfaction score
- ตัวอย่าง: ทีม CI/CD Infrastructure, ทีม Design System, ทีม Data Platform, ทีม Auth/Identity, ทีม Observability

### 892.3 ตารางเปรียบเทียบ

| มิติ | Product Team | Platform Team |
|---|---|---|
| ลูกค้าหลัก | ผู้ใช้ปลายทาง / ธุรกิจ | นักพัฒนาในทีมอื่น (internal customer) |
| Metric ความสำเร็จ | Conversion, engagement, revenue | Adoption rate, deployment frequency, incident count |
| ความถี่ในการเปลี่ยนแปลง | สูงมาก ปรับตาม requirement ตลาด | ต้องเสถียร เปลี่ยนช้าและระมัดระวัง |
| Breaking change ยอมรับได้แค่ไหน | ยอมรับได้ในขอบเขตทีมตัวเอง | ต้องหลีกเลี่ยงอย่างที่สุด เพราะกระทบทุกทีมที่ใช้ |
| รูปแบบการสื่อสาร | ภายในทีม/กับ stakeholder ธุรกิจ | ต้องสื่อสารกับ "ลูกค้าภายใน" หลายสิบทีม |
| ตัวอย่าง Git repo | Service เฉพาะของ feature | Shared library, Terraform module, CI template |

### 892.4 ทำไมการแบ่งแบบนี้ถึงช่วยแก้ปัญหา Multi-team Collaboration

ก่อนมีโมเดลนี้ องค์กรจำนวนมากปล่อยให้ **ทุกทีม Product สร้างเครื่องมือของตัวเองซ้ำซ้อนกัน** — แต่ละทีมมี CI pipeline ของตัวเอง มี library เชื่อมต่อ database ของตัวเอง มี component UI ของตัวเอง ผลลัพธ์คือ:

- ความไม่สอดคล้องกันทั่วทั้งองค์กร (inconsistency)
- งานซ้ำซ้อนมหาศาล (duplicated effort)
- คุณภาพและความปลอดภัยแตกต่างกันไปในแต่ละทีม (บางทีมทำดี บางทีมมีช่องโหว่)

การมี Platform team ช่วยให้:

1. **ทีม Product โฟกัสกับ business logic ล้วน ๆ** ไม่ต้องเสียเวลาสร้าง infrastructure ซ้ำ
2. **มาตรฐานเดียวกันทั่วองค์กร** (security, observability, deployment pattern)
3. **จุดรับผิดชอบชัดเจน** — ถ้า CI ล่ม รู้ทันทีว่าต้องคุยกับใคร

### 892.5 กับดักที่ต้องระวัง: Platform team กลายเป็นคอขวด

ปัญหาที่พบบ่อยมากคือ Platform team กลายเป็น **จุดคอขวดของทั้งองค์กร** เพราะทุกทีมต้องรอ Platform team อนุมัติทุกอย่าง วิธีแก้ที่ Team Topologies และ Amazon (แนวคิด "You build it, you run it" + self-service) แนะนำคือ:

> **Platform team ควรสร้าง "self-service platform" ที่ทีม Product ใช้งานได้เองโดยไม่ต้องขออนุญาตทุกครั้ง — บทบาทของ Platform team คือสร้างถนนที่ดี ไม่ใช่เป็นด่านตรวจทุกคันรถ**

ตัวอย่างเชิงปฏิบัติในบริบท Git:

- Platform team สร้าง **CI/CD template** (reusable workflow ใน GitHub Actions หรือ GitLab CI) ให้ทีม Product เรียกใช้เองได้ทันที ไม่ต้องขออนุมัติทุกครั้งที่จะใช้
- Platform team สร้าง **repository template / scaffolding tool** ที่สร้าง repo ใหม่พร้อม CI, linting, security scan ติดตั้งมาให้แล้วตั้งแต่ต้น
- Platform team เปิด **self-service dashboard** ให้ทีม Product ขอ infrastructure (เช่น database instance ใหม่) ผ่าน pull request ต่อ Terraform config แทนที่จะต้องส่ง ticket แล้วรอคน

### 892.6 ความสัมพันธ์ระหว่างสองโมเดลนี้ใน Git

ในทางปฏิบัติ ความสัมพันธ์นี้มักสะท้อนออกมาใน repository structure ดังนี้:

```
org/
├── platform-ci-templates/       ← Platform team ดูแล
├── platform-design-system/      ← Platform team ดูแล
├── platform-auth-service/       ← Platform team ดูแล
├── product-checkout-service/    ← Product team A ดูแล
├── product-search-service/      ← Product team B ดูแล
└── product-notification-service/ ← Product team C ดูแล
```

ทีม Product จะ `import` หรือ `depend on` package จาก repository ของ Platform team ผ่าน package manager (npm, pip, Maven) หรือผ่าน Git submodule/subtree ในบางกรณี — และนี่คือจุดที่ Step 893 (API contract) จะเข้ามาเกี่ยวข้องโดยตรง

---

## Step 893: API contract และ versioning ระหว่างทีม

เมื่อทีมหนึ่งพึ่งพา API หรือ library ของอีกทีม คำถามสำคัญที่สุดคือ: **"ทำอย่างไรให้ทีมที่เปลี่ยน API ไม่ทำให้ทีมที่เรียกใช้พังโดยไม่รู้ตัว"** คำตอบคือการมี **API contract** ที่ชัดเจนและมีระบบ **versioning** ที่รัดกุม

### 893.1 API Contract คืออะไร

**API contract** คือข้อตกลงที่ระบุอย่างชัดเจนว่า API หนึ่ง ๆ:

- รับ input แบบไหน (request schema)
- ส่ง output แบบไหน (response schema)
- error case ไหนจะเกิดขึ้นและมีรูปแบบอย่างไร
- behavior ที่รับประกัน (เช่น idempotency, ordering guarantee)

Contract นี้ต้องถูก **เขียนเป็นเอกสารที่ตรวจสอบได้ (machine-readable)** ไม่ใช่แค่คำอธิบายในหัวคน เครื่องมือที่นิยมใช้:

| ประเภท API | มาตรฐาน Contract |
|---|---|
| REST API | OpenAPI Specification (Swagger) |
| GraphQL | GraphQL Schema (SDL) |
| gRPC / RPC | Protocol Buffers (`.proto`) |
| Event-driven / Message queue | AsyncAPI, Avro Schema, JSON Schema |

### 893.2 เก็บ Contract ไว้ที่ไหนใน Git

แนวทางที่ดีที่สุดคือเก็บไฟล์ contract (เช่น `openapi.yaml` หรือ `.proto`) **ไว้ใน repository เดียวกับโค้ดของ service** และถือว่ามันเป็นส่วนหนึ่งของ pull request review ปกติ:

```
checkout-service/
├── src/
├── contracts/
│   └── openapi.yaml       ← contract ของ API นี้
├── CHANGELOG.md
└── .github/
    └── workflows/
        └── contract-check.yml   ← CI ตรวจว่า contract เปลี่ยนแบบ breaking หรือไม่
```

บางองค์กรขนาดใหญ่ (เช่นที่ใช้ microservices จำนวนมาก) จะมี **repository กลางสำหรับเก็บ contract ทั้งหมดขององค์กร** แยกต่างหาก เพื่อให้ทุกทีม pull schema ล่าสุดไปใช้ generate client code ได้:

```
org/
└── api-contracts/
    ├── checkout-service/v1/openapi.yaml
    ├── user-service/v1/openapi.yaml
    └── payment-service/v2/openapi.yaml
```

### 893.3 Semantic Versioning สำหรับ API

หลักการ **Semantic Versioning (SemVer)** ที่ใช้กับ library (`MAJOR.MINOR.PATCH`) นำมาประยุกต์กับ API ได้เช่นกัน:

- **MAJOR** เพิ่มขึ้น เมื่อมี **breaking change** (ลบ field, เปลี่ยนชื่อ field, เปลี่ยน type, เปลี่ยน error code ที่มีความหมายต่างไป)
- **MINOR** เพิ่มขึ้น เมื่อ **เพิ่มความสามารถใหม่แบบ backward-compatible** (เพิ่ม field ใหม่ที่เป็น optional, เพิ่ม endpoint ใหม่)
- **PATCH** เพิ่มขึ้น เมื่อ **แก้บั๊กโดยไม่เปลี่ยน contract** (แก้ performance, แก้ error message ที่ไม่กระทบ client)

กฎเหล็กที่ต้องยึดถือ:

> **ห้ามทำ breaking change บน API version เดิมเด็ดขาด ถ้าจะเปลี่ยนแบบ breaking ต้องออก MAJOR version ใหม่ และให้ version เก่ายังทำงานได้คู่ขนานไปจนกว่าทุกทีมที่ใช้จะย้ายมา version ใหม่ครบ**

### 893.4 กลยุทธ์การทำ API Versioning ในทางปฏิบัติ

**1. URL Versioning**

```
GET /api/v1/users/123
GET /api/v2/users/123
```

ชัดเจนที่สุด เห็น version ตรง ๆ ใน URL แต่ทำให้ต้อง maintain code หลาย version พร้อมกัน

**2. Header Versioning**

```http
GET /api/users/123
Accept: application/vnd.company.v2+json
```

URL สะอาดกว่า แต่ debug ยากกว่าเล็กน้อยเพราะ version ไม่เห็นตรง ๆ

**3. Field-level deprecation (แบบ GraphQL)**

```graphql
type User {
  id: ID!
  fullName: String!
  name: String @deprecated(reason: "ใช้ fullName แทน จะถูกลบใน Q3 2026")
}
```

วิธีนี้ยืดหยุ่นที่สุดเพราะ deprecate ได้ทีละ field โดยไม่ต้องออก version ใหม่ทั้งหมด

### 893.5 ตรวจจับ Breaking Change อัตโนมัติด้วย CI

องค์กรที่เป็นมืออาชีพจะไม่พึ่งพา "ความจำ" ของนักพัฒนาว่าจะไม่ทำ breaking change แต่จะสร้าง **CI check ที่ตรวจจับ breaking change โดยอัตโนมัติ** ทุกครั้งที่มี pull request แก้ contract:

```yaml
# .github/workflows/contract-check.yml
name: API Contract Compatibility Check

on:
  pull_request:
    paths:
      - 'contracts/openapi.yaml'

jobs:
  check-breaking-change:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: ดึง contract เวอร์ชันปัจจุบันจาก main branch
        run: git show origin/main:contracts/openapi.yaml > /tmp/old-openapi.yaml

      - name: เปรียบเทียบ contract เก่ากับใหม่หา breaking change
        uses: oasdiff/oasdiff-action/breaking@main
        with:
          base: /tmp/old-openapi.yaml
          revision: contracts/openapi.yaml
          fail-on-diff: true
```

ถ้า CI ตรวจพบว่า pull request มี breaking change (เช่น ลบ field ที่เคยเป็น required, เปลี่ยน type ของ field) จะ **fail ทันทีและ block การ merge** จนกว่าผู้เขียนจะ:

- ยืนยันว่าตั้งใจทำ breaking change จริง และผ่านกระบวนการ RFC/แจ้งเตือนแล้ว (Step 896), หรือ
- แก้ให้เป็น backward-compatible แทน

สำหรับ Protocol Buffers เครื่องมือที่นิยมใช้คือ **Buf** ซึ่งมีคำสั่ง `buf breaking` ที่ทำหน้าที่เดียวกัน:

```bash
buf breaking --against '.git#branch=main'
```

### 893.6 Consumer-Driven Contract Testing

อีกเทคนิคหนึ่งที่ทรงพลังมากในองค์กรที่มีหลายทีมเรียก API กันคือ **Consumer-Driven Contract Testing (CDCT)** โดยใช้เครื่องมือเช่น **Pact**

แนวคิดคือ: แทนที่ทีมผู้ให้บริการ (provider) จะเดาเองว่าทีมที่เรียกใช้ (consumer) ต้องการอะไร ให้ทีม **consumer เป็นคนเขียน contract ที่คาดหวัง** ไว้ล่วงหน้า แล้ว publish ขึ้น "Pact Broker" กลาง จากนั้น CI ของทีม provider จะ **รัน test เทียบกับ contract ของทุก consumer ที่ประกาศไว้** ก่อน merge หรือ deploy ทุกครั้ง

```
Consumer (Mobile team)          Pact Broker            Provider (Backend team)
┌──────────────────┐         ┌────────────┐         ┌──────────────────┐
│ เขียน contract     │────────▶│  เก็บ       │────────▶│ CI ดึง contract   │
│ ที่คาดหวังจาก API   │  publish│  contract   │  verify │ มา verify ก่อน    │
│                    │         │  ทุกเวอร์ชัน │         │ merge/deploy      │
└──────────────────┘         └────────────┘         └──────────────────┘
```

ผลลัพธ์คือทีม Backend จะรู้ทันทีตั้งแต่ก่อน merge ว่า **การเปลี่ยนแปลงของตัวเองจะทำให้ทีมไหนพังบ้าง** โดยไม่ต้องรอไปเจอปัญหาตอน integration หรือ production

### 893.7 สรุปหลักการของ Step นี้

> **API contract คือ "สัญญา" ที่ทำให้ทีมทำงานคู่ขนานกันได้โดยไม่ต้องคุยกันทุกบรรทัดโค้ด — ตราบใดที่ทุกฝ่ายเคารพ contract ที่ตกลงกันไว้ และมี CI คอยตรวจจับการละเมิด contract โดยอัตโนมัติ ทีมต่าง ๆ ก็สามารถ deploy อิสระจากกันได้อย่างปลอดภัย**

---

## Step 894: Cross-team code review policy

Code review ภายในทีมเดียวกันมักใช้กฎง่าย ๆ คือ "ต้องมีอย่างน้อย 1 approve จากเพื่อนร่วมทีม" แต่เมื่อการเปลี่ยนแปลงกระทบไฟล์หรือระบบที่ทีมอื่นเป็นเจ้าของ กฎนี้ไม่เพียงพออีกต่อไป

### 894.1 เมื่อไหร่ที่ต้องขอ review จากทีมอื่น

หลักการทั่วไปที่องค์กรใหญ่ใช้ยึดคือ 4 เกณฑ์นี้ ข้อใดข้อหนึ่งเป็นจริงก็ต้องขอ cross-team review:

1. **แก้ไขไฟล์ที่อยู่ภายใต้ path ที่ CODEOWNERS ระบุว่าเป็นของทีมอื่น**
2. **แก้ไข API contract หรือ shared schema** ที่ทีมอื่นพึ่งพา (ตาม Step 893)
3. **แก้ไข shared library / shared component** ที่มีทีมอื่นใช้งานอยู่ (ตาม Step 895)
4. **เปลี่ยนแปลง infrastructure ที่ทีมอื่นแชร์ใช้งาน** เช่น CI template, deployment pipeline กลาง, database schema กลาง

### 894.2 บังคับด้วย CODEOWNERS + Branch Protection

Git และแพลตฟอร์มอย่าง GitHub/GitLab มีกลไกที่ทำให้กฎนี้ **บังคับใช้อัตโนมัติ ไม่ต้องพึ่งความจำใคร** ผ่านไฟล์ `CODEOWNERS`:

```
# CODEOWNERS

# Default owner ของทั้ง repo คือทีม core
* @org/core-team

# Path ที่เป็นของทีม Platform โดยเฉพาะ
/platform/auth/          @org/platform-auth-team
/platform/ci-templates/  @org/platform-infra-team

# API contract ต้องผ่านทั้งทีมเจ้าของ service และทีม API Governance
/contracts/              @org/checkout-team @org/api-governance-team

# Shared design system component
/packages/ui-components/ @org/design-system-team

# Database migration ที่กระทบ schema กลาง ต้องให้ DBA team review ด้วย
/migrations/             @org/checkout-team @org/dba-team
```

จากนั้นเปิด **Branch Protection Rule** ใน GitHub:

```
Settings → Branches → Branch protection rules → main
☑ Require pull request reviews before merging
☑ Require review from Code Owners
☑ Require approval of the most recent reviewable push
```

ผลลัพธ์คือ ถ้ามี pull request แก้ไฟล์ใน `/contracts/` GitHub จะ **บังคับให้ทั้ง `@checkout-team` และ `@api-governance-team` ต้อง approve** ก่อนจึงจะ merge ได้ — ไม่มีทางลืมหรือหลีกเลี่ยงได้ เพราะระบบบังคับเอง ไม่ใช่ขึ้นอยู่กับวินัยของคน

### 894.3 ระดับความเข้มของ Review ตามความเสี่ยง

ไม่ใช่ทุก cross-team change ต้องการความเข้มงวดเท่ากัน องค์กรที่บริหารจัดการดีมักแบ่งเป็น 3 ระดับ:

| ระดับ | ลักษณะการเปลี่ยนแปลง | จำนวน Reviewer ที่ต้องการ | ตัวอย่าง |
|---|---|---|---|
| **Low risk** | เปลี่ยนใน scope ทีมตัวเอง ไม่กระทบ contract | 1 approve จากทีมตัวเอง | แก้ business logic ภายใน service |
| **Medium risk** | เพิ่ม field/endpoint ใหม่แบบ backward-compatible | 1 approve จากทีมตัวเอง + 1 จากทีมที่เกี่ยวข้อง (informational) | เพิ่ม optional field ใน API |
| **High risk** | Breaking change, แก้ shared library, แก้ infra กลาง | Approve จากทุกทีมที่ CODEOWNERS ระบุ + ผ่าน RFC (Step 896) | ลบ field ออกจาก API, เปลี่ยน major version ของ shared library |

### 894.4 บทบาทของ "Domain Expert Reviewer" ข้ามทีม

นอกจาก reviewer จากทีมเจ้าของไฟล์แล้ว องค์กรใหญ่มักมีบทบาทที่เรียกว่า **Domain Expert** หรือ **Staff/Principal Engineer** ที่ทำหน้าที่ review ข้ามทีมในประเด็นเฉพาะทาง เช่น:

- **Security reviewer** — ต้อง review ทุก PR ที่แตะ authentication, authorization, การเก็บข้อมูลส่วนบุคคล
- **Performance reviewer** — ต้อง review การเปลี่ยนแปลงที่กระทบ query ฐานข้อมูลขนาดใหญ่หรือ hot path
- **Accessibility reviewer** — ต้อง review การเปลี่ยนแปลง UI component ที่ใช้ทั่วองค์กร

บทบาทเหล่านี้มักถูกกำหนดผ่าน CODEOWNERS เช่นกัน โดยผูกกับ path หรือ label เฉพาะ (เช่น label `needs-security-review` ที่ trigger ให้ต้องมี security team approve ก่อน merge ผ่าน GitHub Actions ที่ตรวจ label แล้วบังคับ required check)

### 894.5 ปัญหาที่พบบ่อยและวิธีแก้

**ปัญหา: cross-team reviewer กลายเป็นคอขวด (ไม่ตอบใน PR นานเป็นสัปดาห์)**

วิธีแก้:
- กำหนด **SLA การ review** ที่ชัดเจน เช่น "ทีมที่ถูกขอ review ต้องตอบภายใน 2 วันทำการ" และวัดผลเป็น metric
- ใช้ **Rotation** ให้แต่ละคนในทีมผลัดกันรับผิดชอบ cross-team review ในแต่ละสัปดาห์ แทนที่จะให้คนคนเดียวรับภาระตลอด
- ตั้ง bot แจ้งเตือนอัตโนมัติใน Slack/Teams เมื่อ PR ที่ต้องการ cross-team review ค้างเกิน SLA

**ปัญหา: reviewer จากทีมอื่นไม่เข้าใจ context เพียงพอจะ review อย่างมีคุณภาพ**

วิธีแก้:
- ผู้เขียน PR ต้องเขียน **PR description ที่อธิบายบริบทให้ครบ** โดยเฉพาะ "ทำไมถึงต้องเปลี่ยน" และ "กระทบใครบ้าง" ไม่ใช่แค่ "เปลี่ยนอะไร"
- แนบลิงก์ RFC หรือ design doc ที่เกี่ยวข้องเสมอ (เชื่อมกับ Step 899)

### 894.6 ตัวอย่าง PR Template สำหรับ Cross-team Change

```markdown
## สิ่งที่เปลี่ยน
<!-- อธิบายว่าเปลี่ยนอะไร -->

## เหตุผล
<!-- ทำไมถึงต้องเปลี่ยน -->

## ทีมที่ได้รับผลกระทบ
- [ ] ทีม Mobile (เปลี่ยน API response schema)
- [ ] ทีม Analytics (เปลี่ยนชื่อ event tracking)

## Breaking change หรือไม่
- [ ] ใช่ — แนบลิงก์ RFC: <link>
- [ ] ไม่ใช่ — backward compatible 100%

## Rollback plan
<!-- ถ้า deploy แล้วมีปัญหา จะ rollback อย่างไร -->
```

---

## Step 895: Shared library/component management ข้ามทีม

Shared library คือหนึ่งในแหล่งที่มาของปัญหา multi-team collaboration ที่พบบ่อยที่สุด เพราะมันคือจุดที่ **โค้ดของทีมเดียวถูกใช้งานโดยทีมอื่นจำนวนมาก**

### 895.1 ตัวอย่างของ Shared Library ในองค์กรจริง

- **Design system / UI component library** เช่น ปุ่ม, form, modal ที่ทุกทีม Frontend ใช้ร่วมกัน
- **Common utility library** เช่น logging wrapper, HTTP client wrapper, date/time helper
- **Internal SDK** สำหรับเรียก internal service (เช่น SDK เรียก payment service)
- **Shared type definitions** เช่น TypeScript type ของ domain model ที่หลาย service ใช้ร่วมกัน

### 895.2 ใครควรเป็นเจ้าของ Shared Library

หลักการที่ใช้กันแพร่หลายคือ **"library ที่มี consumer มากกว่า 2 ทีมขึ้นไป ควรมีเจ้าของที่ชัดเจนเพียงทีมเดียว"** ไม่ควรปล่อยให้เป็นแบบ "ใครแก้ก็ได้" (tragedy of the commons) เพราะจะนำไปสู่ความไม่สอดคล้องกันและคุณภาพที่ถดถอยลงเรื่อย ๆ

รูปแบบความเป็นเจ้าของที่พบบ่อย:

1. **Dedicated owner team** — มีทีมเฉพาะ (มักเป็น Platform team) ดูแล library นี้เต็มเวลา เหมาะกับ library ที่สำคัญมากและมี consumer จำนวนมาก
2. **Rotating ownership** — ทีมที่สร้าง library ตอนแรกเป็นเจ้าของ แต่หมุนเวียนคนดูแล (maintainer) จากทีมที่ใช้งานบ่อยที่สุด
3. **Federated contribution model** — มี core maintainer กลุ่มเล็ก (2-3 คน) เป็นผู้ approve สุดท้าย แต่เปิดให้ทุกทีมส่ง pull request มาแก้ไขได้เอง (โมเดลคล้าย Open Source ภายในองค์กร — "InnerSource")

### 895.3 InnerSource: นำหลักการ Open Source มาใช้ภายในองค์กร

**InnerSource** คือแนวคิดที่นำวิธีการทำงานแบบ Open Source (ที่คุณเรียนไปแล้วใน Part 15-30 ของหลักสูตรนี้) มาประยุกต์ใช้ภายในองค์กรเดียวกัน หลักการสำคัญ:

- Shared library เปิดให้ **ทุกทีมเห็น source code และส่ง pull request ได้** (ไม่ใช่แค่ทีมเจ้าของเท่านั้นที่แก้ได้)
- มี **maintainer ที่ชัดเจน** (ระบุใน `MAINTAINERS.md` หรือ `CODEOWNERS`) เป็นผู้ตัดสินใจสุดท้ายว่าจะ merge หรือไม่
- มี **Contributing Guide** ที่บอกวิธีการส่ง contribution อย่างชัดเจน คล้ายกับ Open Source project ทั่วไป
- ทีมที่ต้องการ feature ใหม่ในไลบรารีสามารถ **ส่ง PR เอง** แทนที่จะรอให้ทีมเจ้าของทำให้ ลดคอขวดได้มาก

```
shared-ui-library/
├── MAINTAINERS.md          ← ใครเป็น maintainer หลัก
├── CONTRIBUTING.md         ← วิธี contribute
├── CODEOWNERS              ← ใคร approve ได้
├── CHANGELOG.md            ← ประวัติการเปลี่ยนแปลงแบบ SemVer
└── src/
```

### 895.4 ใครมีสิทธิ์ขอ Breaking Change ใน Shared Library

นี่คือคำถามที่สำคัญที่สุดของ Step นี้ กฎที่แนะนำ:

> **ทีมที่ใช้งาน (consumer) เสนอ breaking change ได้เสมอ แต่ "ผู้อนุมัติสุดท้าย" ต้องเป็น maintainer ของ library เท่านั้น และก่อนอนุมัติ maintainer ต้องประเมินผลกระทบต่อ consumer ทุกทีมก่อนเสมอ**

กระบวนการที่แนะนำสำหรับ breaking change ใน shared library:

1. **ทีมที่ต้องการเปลี่ยนเปิด RFC** อธิบายว่าจะเปลี่ยนอะไร ทำไม (Step 896)
2. **Maintainer ทำ impact analysis** — หา consumer ทั้งหมดที่ import library เวอร์ชันปัจจุบัน (ใช้เครื่องมือค้นหา dependency เช่น dependency graph ใน monorepo หรือ internal package registry)
3. **แจ้ง consumer ทุกทีมล่วงหน้า** พร้อม deprecation timeline
4. **ออก version ใหม่แบบ MAJOR bump** ตาม SemVer พร้อม migration guide
5. **คง version เก่าไว้ในช่วง deprecation period** (เช่น 2 sprint หรือ 1 ไตรมาส) ก่อนจะ archive
6. **ติดตาม adoption** ว่าแต่ละทีม migrate ไปหรือยัง ผ่าน dashboard หรือ CI check

### 895.5 การจัดการ Version ของ Shared Library ในทางปฏิบัติ

**กรณี Monorepo:** มักใช้เครื่องมือจัดการ dependency ภายใน เช่น `Nx`, `Turborepo`, `Bazel` ที่ให้ consumer ใช้ library เวอร์ชันล่าสุดใน repo เดียวกันเสมอ (single version policy) ซึ่งบังคับให้ maintainer ต้อง fix breaking change ให้ consumer ทุกตัวใน PR เดียวกันเลย (atomic change) — ข้อดีคือไม่มี version drift แต่ข้อเสียคือ PR แก้ library อาจต้องแก้หลายสิบไฟล์ของทีมอื่นไปพร้อมกัน

**กรณี Polyrepo:** แต่ละ service ประกาศ version ของ library ที่ตัวเอง depend on ใน `package.json` / `requirements.txt` / `pom.xml` เอง ทำให้แต่ละทีมย้าย version ได้ตามจังหวะของตัวเอง แต่ต้องมีระบบติดตามว่ามีทีมไหนยังใช้ version เก่าที่ถูก deprecate ค้างอยู่นานเกินไป (เช่น Dependabot/Renovate ที่เปิด PR อัตโนมัติเมื่อมี version ใหม่ พร้อม dashboard สรุปภาพรวมทั้งองค์กร)

### 895.6 ตัวอย่าง CHANGELOG ที่ดีสำหรับ Shared Library

```markdown
# Changelog

## [3.0.0] - 2026-08-15
### BREAKING CHANGES
- ลบ prop `onClick` ออกจาก `<Button>` — ใช้ `onPress` แทนเพื่อรองรับทั้ง web และ mobile
- Migration guide: https://internal-docs/design-system/v3-migration

### Deprecation timeline
- v2.x จะยังคงได้รับ security patch จนถึง 2026-11-30
- หลังจากนั้นจะถูก archive และไม่รับ PR ใหม่

## [2.4.0] - 2026-06-01
### Added
- เพิ่ม prop `variant="danger"` ใน `<Button>` (backward compatible)
```

CHANGELOG แบบนี้ทำหน้าที่เป็นทั้งประวัติและ **สัญญาการสื่อสาร** กับทุกทีมที่ใช้ library — เชื่อมโยงกับหลักการ SemVer ที่เรียนไปใน Part ก่อนหน้าโดยตรง

---

## Step 896: Communication protocol เมื่อจะทำ breaking change ที่กระทบทีมอื่น

Step ก่อนหน้าพูดถึง "อะไร" ที่ต้องทำเมื่อจะทำ breaking change ส่วน Step นี้จะเจาะลึก "กระบวนการสื่อสาร" อย่างเป็นระบบ ซึ่งเป็นหัวใจที่แท้จริงของการป้องกันไม่ให้ breaking change ทำลายทีมอื่น

### 896.1 ทำไมต้องมี "กระบวนการ" ไม่ใช่แค่ "ความหวังดี"

หลายองค์กรพึ่งพาแค่ "มารยาท" ในการแจ้งทีมอื่นก่อนทำ breaking change เช่น ส่งข้อความใน Slack ว่า "เดี๋ยวผมจะเปลี่ยน API นะ" — วิธีนี้ล้มเหลวเสมอเมื่อองค์กรโตขึ้น เพราะ:

- ข้อความใน chat หายไปในกระแสข้อมูล ไม่มีใครย้อนกลับมาดู
- ไม่มีบันทึกที่เป็นทางการว่า "ใครเห็นด้วย ใครคัดค้าน"
- ไม่มี timeline ที่ชัดเจนว่าจะเกิดขึ้นเมื่อไหร่
- ทีมที่ได้รับผลกระทบจริง (แต่ไม่ได้อยู่ในห้อง chat นั้น) ไม่มีทางรู้เรื่อง

องค์กรที่เป็นมืออาชีพจึงใช้กระบวนการที่เป็นทางการเรียกว่า **RFC (Request for Comments)**

### 896.2 RFC Process คืออะไร

**RFC** คือเอกสารที่เขียนขึ้นก่อนจะลงมือทำการเปลี่ยนแปลงที่มีผลกระทบวงกว้าง เพื่อ**เปิดโอกาสให้ทุกฝ่ายที่เกี่ยวข้องแสดงความเห็นก่อนตัดสินใจจริง** แนวคิดนี้ยืมมาจากกระบวนการมาตรฐานอินเทอร์เน็ต (IETF RFC) และถูกนำมาใช้ในองค์กรอย่าง Rust, React, และบริษัทเทคโนโลยีขนาดใหญ่จำนวนมาก

### 896.3 โครงสร้างของ RFC Document ทั่วไป

```markdown
# RFC-042: เปลี่ยน Authentication Token จาก JWT เป็น Opaque Token

- **สถานะ:** Draft / In Review / Accepted / Rejected / Implemented
- **ผู้เสนอ:** ทีม Platform Auth
- **วันที่เปิด:** 2026-09-01
- **กำหนดปิดรับความเห็น:** 2026-09-15

## 1. สรุปสั้น (Summary)
เปลี่ยนรูปแบบ token จาก JWT เป็น opaque token เพื่อรองรับการ revoke token ได้ทันที

## 2. แรงจูงใจ (Motivation)
ปัจจุบัน JWT ไม่สามารถ revoke ได้ก่อนหมดอายุ ทำให้เกิดความเสี่ยงด้านความปลอดภัย
เมื่อ token รั่วไหล...

## 3. รายละเอียดการออกแบบ (Detailed Design)
...

## 4. ทีมที่ได้รับผลกระทบ (Affected Teams)
- ทีม Mobile: ต้องเปลี่ยนวิธี parse token
- ทีม Web: ต้องเปลี่ยน middleware ตรวจสอบ token
- ทีม Partner API: ต้องแจ้ง partner ภายนอกด้วย

## 5. ทางเลือกอื่นที่พิจารณาแล้ว (Alternatives Considered)
...

## 6. แผนการ Migration และ Timeline
- Week 1-2: เปิดใช้ระบบใหม่แบบคู่ขนาน (dual-write)
- Week 3-6: ให้ทุกทีม migrate มาใช้ endpoint ใหม่
- Week 7: ปิด endpoint เก่า

## 7. Rollback Plan
...

## ความเห็นจากทีมที่เกี่ยวข้อง
- [ ] ทีม Mobile — approve / ต้องการเวลาเพิ่ม
- [ ] ทีม Web — approve
- [ ] ทีม Security — approve (บังคับสำหรับเรื่อง auth)
```

### 896.4 เก็บ RFC ไว้ที่ไหน — ทำไมต้องอยู่ใน Git

แนวทางที่ดีที่สุดคือเก็บ RFC เป็น **Markdown file ใน Git repository** (ไม่ใช่ Google Doc หรือ Confluence เพียงอย่างเดียว) ด้วยเหตุผลสำคัญ:

1. **Pull Request คือกลไก review ที่มีอยู่แล้ว** — เปิด PR เพิ่มไฟล์ RFC ใหม่ ให้ทุกทีมมา comment ผ่าน PR review เหมือน code review ปกติ
2. **มีประวัติที่ตรวจสอบย้อนหลังได้ (git blame, git log)** — รู้ว่าใครแก้ RFC ตรงไหน เมื่อไหร่
3. **Merge PR = RFC ได้รับการยอมรับอย่างเป็นทางการ** — สถานะของ RFC ผูกกับสถานะของ PR โดยธรรมชาติ
4. **ค้นหาย้อนหลังได้ง่าย** ผ่าน full-text search ของ repository

```
org/
└── rfcs/
    ├── 0001-migrate-to-monorepo.md
    ├── 0002-adopt-graphql-federation.md
    ├── 0042-opaque-auth-token.md
    └── README.md   ← อธิบายกระบวนการ RFC และ template
```

### 896.5 Deprecation Notice: การแจ้งเตือนแบบเป็นขั้นเป็นตอน

นอกจาก RFC สำหรับตัดสินใจล่วงหน้าแล้ว เมื่อ breaking change ผ่านการอนุมัติและเริ่มดำเนินการ ต้องมี **deprecation notice** ที่ต่อเนื่องตลอดช่วง migration เพื่อไม่ให้ทีมที่ได้รับผลกระทบลืม

**ช่องทางการแจ้งเตือนที่ควรใช้ร่วมกัน (ไม่พึ่งช่องทางเดียว):**

1. **Compile-time / runtime warning** — เช่น log warning ทุกครั้งที่มีการเรียก API เก่าที่กำลังจะถูกลบ พร้อมระบุ deadline
   ```
   [DEPRECATION WARNING] Endpoint /api/v1/users จะถูกปิดในวันที่ 2026-11-30
   กรุณาเปลี่ยนไปใช้ /api/v2/users — ดูรายละเอียด: https://internal-docs/migration/v2-users
   ```
2. **CI/build-time warning** — เมื่อ consumer build โปรเจกต์ที่ depend on library version เก่า ให้ CI แสดง warning
3. **ประกาศในช่องทางกลางขององค์กร** — Slack channel `#engineering-announcements`, email, หรือ internal engineering newsletter
4. **Tracking issue กลาง** — เปิด GitHub Issue เดียวที่ track สถานะการ migrate ของทุกทีม พร้อม checklist
   ```markdown
   ## Migration Tracking: JWT → Opaque Token (RFC-042)

   - [x] ทีม Web — migrated 2026-09-10
   - [x] ทีม Mobile — migrated 2026-09-18
   - [ ] ทีม Partner API — กำลังดำเนินการ ETA 2026-09-25
   - [ ] ทีม Internal Admin — ยังไม่เริ่ม ⚠️
   ```

### 896.6 Timeline มาตรฐานสำหรับ Breaking Change

องค์กรที่มีวินัยดีมักกำหนด **นโยบายระยะเวลาขั้นต่ำ** ก่อนจะปิด version เก่าได้ เช่น:

| ขนาดผลกระทบ | ระยะเวลา deprecation ขั้นต่ำ |
|---|---|
| กระทบทีมเดียว | 1 sprint (1-2 สัปดาห์) |
| กระทบ 2-5 ทีมภายในองค์กร | 1 เดือน |
| กระทบทั้งองค์กรหรือ external partner | 1 ไตรมาส (3 เดือน) ขึ้นไป |

และต้องมี **"escape hatch"** เสมอ — ถ้าถึง deadline แล้วยังมีทีมที่ migrate ไม่เสร็จ ต้องมีกระบวนการตัดสินใจว่าจะ **ขยายเวลา** หรือ **บังคับปิดตามกำหนดพร้อมความช่วยเหลือเร่งด่วน** ไม่ใช่ปล่อยให้ deadline เลื่อนไม่มีที่สิ้นสุดโดยไม่มีใครตัดสินใจ

### 896.7 สรุปหลักการสื่อสาร

> **RFC ตอบคำถาม "จะทำหรือไม่ทำ และทำอย่างไร" ก่อนเริ่มงาน ส่วน Deprecation Notice ตอบคำถาม "ตอนนี้ถึงไหนแล้ว และเหลือเวลาอีกเท่าไหร่" ระหว่างทำงาน — สองอย่างนี้ต้องใช้คู่กันเสมอสำหรับ breaking change ที่กระทบทีมอื่น**

---

## Step 897: Feature flags สำหรับ coordinate release ข้ามทีม

ใน **Part 34** ของหลักสูตรนี้ เราได้เรียนรู้ Trunk-Based Development และแนวคิด Feature Flag ในบริบทของทีมเดียวไปแล้ว ใน Step นี้เราจะต่อยอดแนวคิดเดียวกัน แต่ในบริบทที่ซับซ้อนขึ้น: **การใช้ feature flag เพื่อ coordinate การ release ระหว่างหลายทีมพร้อมกัน**

### 897.1 ทบทวนสั้น ๆ: Feature Flag คืออะไร

Feature flag (หรือ feature toggle) คือกลไกที่แยก **"การ deploy โค้ด" ออกจาก "การเปิดใช้งานฟีเจอร์"** โค้ดสามารถถูก deploy ขึ้น production ได้โดยที่ฟีเจอร์ยังปิดอยู่ (ควบคุมผ่าน configuration ไม่ใช่ผ่าน code path ที่ต่างกัน) แล้วค่อยเปิดใช้งานทีหลังโดยไม่ต้อง deploy ใหม่

```javascript
if (featureFlags.isEnabled('new-checkout-flow', { userId })) {
  return renderNewCheckoutFlow();
} else {
  return renderLegacyCheckoutFlow();
}
```

### 897.2 ทำไม Multi-team Release ถึงต้องพึ่ง Feature Flag

ลองพิจารณาสถานการณ์: ฟีเจอร์ใหม่ "Checkout ด้วย QR Code" ต้องการทั้ง 3 ทีม:

- **Backend team** ต้อง deploy API ใหม่รองรับ QR payment
- **Mobile team** ต้อง deploy แอปเวอร์ชันใหม่ที่มี UI สแกน QR
- **Payment team** ต้อง deploy service เชื่อมต่อผู้ให้บริการ QR payment ภายนอก

ถ้าไม่มี feature flag ทั้ง 3 ทีมต้อง **deploy ให้ตรงเวลากันเป๊ะ ๆ** ซึ่งในทางปฏิบัติแทบเป็นไปไม่ได้ เพราะแต่ละทีมมี CI/CD pipeline, ตารางงาน, และปัญหาที่ทำให้ deploy ล่าช้าต่างกันไป

Feature flag แก้ปัญหานี้โดยการ **แยก timeline การ deploy โค้ด ออกจาก timeline การเปิดใช้งานฟีเจอร์ให้ผู้ใช้เห็น** อย่างสิ้นเชิง:

```
Timeline:
Day 1:  Backend deploy API ใหม่ (flag: OFF)     ← ไม่มีผลกระทบ เพราะยังปิดอยู่
Day 3:  Payment deploy service ใหม่ (flag: OFF)  ← ไม่มีผลกระทบ
Day 5:  Mobile deploy แอปเวอร์ชันใหม่ (flag: OFF) ← ไม่มีผลกระทบ
Day 7:  ทุกทีมยืนยันพร้อม → เปิด flag พร้อมกันผ่าน config
        → ผู้ใช้เห็นฟีเจอร์ใหม่ทันที โดยไม่ต้อง deploy โค้ดใหม่เลย
```

### 897.3 ระดับของ Feature Flag ในบริบทข้ามทีม

**1. Release Flag (ควบคุมการเปิดตัวฟีเจอร์)**

ใช้สำหรับ coordinate การเปิดตัวข้ามทีมโดยเฉพาะ มักมีอายุสั้น (เปิดแล้วลบทิ้งหลังฟีเจอร์เสถียร)

**2. Ops Flag (ควบคุมการทำงานของระบบ)**

ใช้ปิดฟีเจอร์อย่างรวดเร็วเมื่อเกิดปัญหา (kill switch) โดยไม่ต้อง rollback deploy — สำคัญมากสำหรับ incident response ข้ามทีม (เชื่อมกับ Step 898)

**3. Permission Flag (เปิดเฉพาะบางกลุ่มผู้ใช้)**

ใช้สำหรับ gradual rollout เช่น เปิดให้ internal staff ก่อน แล้วค่อยขยายเป็น 1% → 10% → 100% ของผู้ใช้จริง

### 897.4 Feature Flag ต้องมี "เจ้าของ" และ "อายุการใช้งาน" ที่ชัดเจน

ปัญหาที่พบบ่อยมากในองค์กรที่มีหลายทีมคือ **flag ค้างอยู่ในระบบเป็นปี ๆ ไม่มีใครกล้าลบ** เพราะไม่รู้ว่าทีมไหนยังพึ่งพา flag นั้นอยู่บ้าง ทำให้ codebase เต็มไปด้วย `if/else` ที่ไม่มีความหมายอีกต่อไป

แนวทางป้องกัน:

- ทุก flag ต้องมี **owner ที่ระบุชัดเจน** ใน flag management system (เช่น LaunchDarkly, Unleash, หรือ internal tool) พร้อมวันที่คาดว่าจะลบ
- ตั้ง **CI job ที่แจ้งเตือนอัตโนมัติ** เมื่อ flag มีอายุเกินกำหนด (เช่น 90 วัน) ให้ทีมเจ้าของมาตัดสินใจว่าจะลบ code path เก่าทิ้งหรือไม่
- การลบ flag ที่กระทบหลายทีม **ต้องผ่านกระบวนการเดียวกับ breaking change** (Step 896) เพราะการลบ code path เก่าออกก็คือการตัด "ทางถอย" ที่บางทีมอาจยังต้องใช้อยู่

```yaml
# ตัวอย่าง config บันทึกใน flags.yaml ของ repository
flags:
  - key: new-checkout-qr-flow
    owner: checkout-team
    created: 2026-08-01
    expected_removal: 2026-11-01
    affected_teams: [mobile-team, payment-team, backend-team]
    status: partial-rollout   # off | partial-rollout | full-rollout | removed
```

### 897.5 Feature Flag กับ Trunk-Based Development ข้ามทีม

การผสาน Feature Flag เข้ากับ Trunk-Based Development (Part 34) ทำให้หลายทีมสามารถ **merge โค้ดเข้า trunk/main บ่อย ๆ อย่างต่อเนื่อง** โดยไม่ต้องรอให้ฟีเจอร์เสร็จสมบูรณ์ครบทุกทีมก่อน ซึ่งลดความเสี่ยงของ **long-lived branch ข้ามทีม** ที่ conflict กันหนักเมื่อถึงเวลา merge

```
ไม่มี Feature Flag (อันตราย):
main ──●──●──●───────────────────────●(merge ทีเดียวหลัง 3 เดือน)──▶
              \                      /
               feature/qr-checkout──●   ← 3 ทีม commit สะสม conflict มหาศาล

มี Feature Flag (ปลอดภัยกว่ามาก):
main ──●──●──●──●──●──●──●──●──●──●──●──▶
       ↑ backend commit เข้า main ทุกวัน (flag OFF)
       ↑ mobile commit เข้า main ทุกวัน (flag OFF)
       ↑ payment commit เข้า main ทุกวัน (flag OFF)
                                    ↑ เปิด flag พร้อมกันเมื่อพร้อม
```

### 897.6 Checklist ก่อนเปิด Feature Flag ข้ามทีม

ก่อนจะกด "เปิด flag" ที่กระทบหลายทีม ควรมี checklist ยืนยันร่วมกัน:

- [ ] ทุกทีมที่เกี่ยวข้อง deploy โค้ดที่รองรับ flag เวอร์ชันใหม่ขึ้น production ครบแล้ว
- [ ] มี monitoring/alert พร้อมสำหรับ metric ที่เกี่ยวข้องกับฟีเจอร์ใหม่
- [ ] มีแผน rollback (ปิด flag กลับ) ที่ทดสอบแล้วว่าทำงานได้จริงและเร็ว
- [ ] แจ้ง on-call engineer ของทุกทีมที่เกี่ยวข้องว่าจะเปิด flag เมื่อไหร่
- [ ] กำหนดคนรับผิดชอบตัดสินใจ "go/no-go" ที่ชัดเจนคนเดียว ไม่ใช่ต้องรอ consensus จากทุกทีมแบบ real-time

---

## Step 898: Incident response ข้ามทีมเมื่อเกิดปัญหาจาก dependency ที่ใช้ร่วมกัน

เมื่อระบบมีหลายทีมพึ่งพากันผ่าน shared service, shared library หรือ shared infrastructure ปัญหาที่เกิดในจุดหนึ่งสามารถ **ลามข้ามขอบเขตทีมได้อย่างรวดเร็ว** Step นี้จะพูดถึงวิธีจัดการ incident ที่มีลักษณะข้ามทีมโดยเฉพาะ

### 898.1 ความแตกต่างระหว่าง Incident ภายในทีม vs ข้ามทีม

| มิติ | Incident ภายในทีม | Incident ข้ามทีม |
|---|---|---|
| การหา root cause | ทีมเดียวเห็นโค้ดทั้งหมด หาได้เร็ว | ต้องประสานหลายทีมเพื่อเห็นภาพรวม |
| คนที่ต้องเข้าร่วมแก้ปัญหา | ทีมตัวเอง | หลายทีมที่เกี่ยวข้องกับ dependency |
| การตัดสินใจ rollback | ทีมตัวเองตัดสินใจได้เลย | อาจกระทบทีมอื่น ต้องพิจารณาผลข้างเคียง |
| ความเสี่ยงของการสื่อสารผิดพลาด | ต่ำ (คุยกันในทีมเดียว) | สูงมาก (หลายทีม หลาย context) |

### 898.2 บทบาท Incident Commander (IC) ในบริบทข้ามทีม

หลักการสำคัญที่สุดของ incident response ข้ามทีมคือ **ต้องมีคนเดียวที่ทำหน้าที่ตัดสินใจสุดท้าย (Incident Commander)** เพื่อป้องกันสถานการณ์ "หลายทีมพยายามแก้พร้อมกันแต่คนละทิศทาง" ซึ่งมักทำให้สถานการณ์แย่ลงกว่าเดิม

**บทบาทของ Incident Commander:**

- ไม่จำเป็นต้องเป็นคนที่เก่งด้านเทคนิคที่สุด แต่ต้องเป็นคนที่ **ประสานงานเก่งและตัดสินใจได้**
- ตัดสินใจว่าจะ **rollback ทันที** หรือ **rollforward ด้วย hotfix**
- เป็นจุดศูนย์กลางของการสื่อสาร ไม่ให้แต่ละทีมสื่อสารกันแบบกระจัดกระจาย
- ตัดสินใจว่าต้องเรียกทีมไหนเข้าร่วม "war room" เพิ่มบ้าง

### 898.3 ขั้นตอนมาตรฐานของ Incident Response ข้าม Dependency

```
1. Detect (ตรวจพบ)
   └─ Alert แจ้งเตือนจาก monitoring (เช่น error rate พุ่งสูงผิดปกติ)

2. Triage (ประเมินเบื้องต้น)
   └─ ทีมที่เห็น alert ก่อน ประเมินว่า "ต้นตอน่าจะมาจาก service/library ของทีมไหน"
   └─ ถ้าไม่แน่ใจ ให้ประกาศ incident และเปิด war room ทันที (อย่ารอให้แน่ใจ 100%)

3. Assemble (รวมทีม)
   └─ แต่งตั้ง Incident Commander
   └─ เชิญตัวแทนจากทุกทีมที่ dependency เกี่ยวข้อง เข้า incident channel/war room

4. Investigate (สืบสวนร่วมกัน)
   └─ แชร์ log, trace, dashboard ระหว่างทีมแบบ real-time
   └─ ใช้ distributed tracing (เช่น OpenTelemetry) เพื่อดูว่า request ไหลผ่านทีมไหนบ้าง

5. Mitigate (บรรเทาความเสียหายทันที)
   └─ ตัดสินใจ: ปิด feature flag / rollback deploy / scale resource เพิ่ม
   └─ Incident Commander เป็นผู้อนุมัติการกระทำที่กระทบระบบ

6. Resolve (แก้ไขจนเสถียร)

7. Postmortem (สรุปบทเรียนหลังเหตุการณ์) — แบบ blameless
```

### 898.4 ทำไมต้องรู้ Dependency Graph ล่วงหน้า ก่อนเกิด Incident

จุดที่ทำให้ incident ข้ามทีมแก้ช้าที่สุดคือ **การไม่รู้ว่าใครพึ่งพาใครบ้าง** เมื่อเกิดปัญหาจึงต้องเสียเวลาถามหาว่า "service นี้มีใครเรียกใช้บ้าง" ท่ามกลางความชุลมุน

องค์กรที่เตรียมพร้อมดีจะมี **Service Dependency Map** ที่จัดทำไว้ล่วงหน้า (ไม่ใช่สร้างตอนเกิดเหตุ) ผ่านเครื่องมือเช่น:

- **Service catalog** (เช่น Backstage จาก Spotify) ที่บันทึกว่า service ไหน depend on service ไหน
- **Distributed tracing dashboard** ที่แสดง call graph จริงจาก production traffic
- **README ของแต่ละ repository** ที่ระบุ "Consumers" และ "Dependencies" ชัดเจน

```markdown
# README.md ของ payment-service

## Dependencies (service นี้เรียกใช้)
- user-service (ตรวจสอบสิทธิ์ผู้ใช้)
- fraud-detection-service (ตรวจจับธุรกรรมผิดปกติ)

## Consumers (service ที่เรียกใช้ service นี้)
- checkout-service (ทีม Checkout)
- subscription-service (ทีม Billing)
- mobile-app-backend-for-frontend (ทีม Mobile)

## On-call contact
- Slack: #payment-team-oncall
- PagerDuty: payment-team-primary
```

การมีข้อมูลนี้พร้อมล่วงหน้าทำให้เมื่อเกิด incident **สามารถระบุได้ทันทีว่าต้องแจ้งเตือนใครบ้าง** โดยไม่ต้องเสียเวลาสืบหา

### 898.5 Communication Channel ระหว่าง Incident

แนวทางที่แนะนำสำหรับองค์กรที่มีหลายทีม:

- เปิด **incident channel เฉพาะกิจ** (เช่น `#incident-2026-09-26-checkout-down`) แยกจาก channel ประจำทีม เพื่อไม่ให้ noise ปนกับงานประจำวัน
- ใช้ **incident management tool** (เช่น PagerDuty, Opsgenie, FireHydrant) ที่ track timeline อัตโนมัติ ใครทำอะไรเมื่อไหร่ — สำคัญมากสำหรับการเขียน postmortem ทีหลัง
- Incident Commander โพสต์ **status update เป็นช่วง ๆ อย่างสม่ำเสมอ** (เช่นทุก 15-30 นาที) แม้จะยังไม่มีความคืบหน้าใหม่ ก็ต้องบอกว่า "ยังไม่มีความคืบหน้า กำลังตรวจสอบ X อยู่" เพื่อไม่ให้ทุกทีมรู้สึกถูกทิ้งไว้ในความมืด

### 898.6 Blameless Postmortem ข้ามทีม

หลังเหตุการณ์สงบแล้ว ต้องเขียน **postmortem แบบ blameless** (ไม่กล่าวโทษบุคคล) ที่ครอบคลุมทุกทีมที่เกี่ยวข้อง โดยเก็บไว้ใน Git เช่นเดียวกับ RFC:

```
org/
└── postmortems/
    └── 2026-09-26-checkout-service-outage.md
```

โครงสร้าง postmortem ที่ดีควรมี:

1. **Timeline โดยละเอียด** (ดึงจาก incident management tool)
2. **Root cause** — สาเหตุที่แท้จริง ไม่ใช่แค่อาการ
3. **Impact** — กระทบผู้ใช้กี่คน กระทบทีมไหนบ้าง เสียหายเท่าไหร่
4. **What went well / what went wrong** — รวมมุมมองจากทุกทีมที่เข้าร่วม
5. **Action items พร้อมเจ้าของและ deadline ชัดเจน** — โดยเฉพาะ action item ที่เป็น **cross-team** เช่น "ทีม A ต้องเพิ่ม contract test กับทีม B ภายในสิ้นเดือน"

สิ่งสำคัญที่สุดคือ **action item ต้องถูกติดตามจริงจนเสร็จ** ไม่ใช่เขียน postmortem แล้วจบ — องค์กรที่ดีจะมี dashboard ติดตาม action item ค้างจาก postmortem ทั้งหมด และทบทวนในที่ประชุมระดับ engineering leadership เป็นประจำ

---

## Step 899: Documentation เป็นเครื่องมือสื่อสารข้ามทีมที่สำคัญที่สุด (ADR)

ตลอด Part นี้เราพูดถึง RFC, CHANGELOG, README, postmortem มาแล้วหลายครั้ง — ทั้งหมดนี้คือรูปแบบต่าง ๆ ของ **documentation** ซึ่งเป็นเครื่องมือสื่อสารที่สำคัญที่สุดในองค์กรที่มีหลายทีม เพราะมันคือสิ่งเดียวที่ **ทำงานได้แม้ผู้เขียนไม่ได้อยู่ตรงนั้นแล้ว**

### 899.1 ทำไม Documentation ถึงสำคัญกว่าการพูดคุยแบบ Synchronous

ในทีมเล็ก การเดินไปถามเพื่อนข้าง ๆ ว่า "ทำไมโค้ดตรงนี้ถึงเขียนแบบนี้" เป็นเรื่องปกติและเร็วกว่าเขียนเอกสาร แต่เมื่อองค์กรโตขึ้น วิธีนี้ล้มเหลวโดยสิ้นเชิงด้วยเหตุผล:

1. **คนที่รู้คำตอบอาจอยู่คนละ timezone** — องค์กรระดับโลกมีทีมกระจายทั่วโลก การรอคำตอบแบบ synchronous อาจใช้เวลาข้ามวัน
2. **คนที่ตัดสินใจอาจลาออกไปแล้ว** — ถ้าไม่มีบันทึกเป็นลายลักษณ์อักษร เหตุผลเบื้องหลังการตัดสินใจจะหายไปตลอดกาล
3. **จำนวนคนที่ต้องถามคำถามเดียวกันมีมาก** — เอกสารเขียนครั้งเดียว อ่านซ้ำได้ไม่จำกัดครั้ง คุ้มค่ากว่าตอบคำถามซ้ำ ๆ ทาง chat
4. **การตัดสินใจข้ามทีมต้องการ "หลักฐาน" ที่ตรวจสอบย้อนหลังได้** เมื่อเกิดข้อพิพาทภายหลังว่า "ใครอนุมัติเรื่องนี้"

### 899.2 Architecture Decision Record (ADR) คืออะไร

**ADR (Architecture Decision Record)** คือเอกสารสั้น ๆ ที่บันทึก **การตัดสินใจเชิงสถาปัตยกรรมที่สำคัญ** พร้อมบริบทและเหตุผลประกอบ ณ เวลาที่ตัดสินใจ แนวคิดนี้ถูกเสนอโดย Michael Nygard และถูกใช้อย่างแพร่หลายในองค์กรวิศวกรรมทั่วโลก

ข้อแตกต่างสำคัญระหว่าง ADR กับ RFC (Step 896):

| | RFC | ADR |
|---|---|---|
| จุดประสงค์ | ขอความเห็นชอบ **ก่อน** ตัดสินใจ | บันทึกผล **หลัง** ตัดสินใจแล้ว |
| ขนาด | ยาว ละเอียด มีหลายทางเลือกให้ถกเถียง | สั้น กระชับ บันทึกเฉพาะผลสรุป |
| อายุการใช้งาน | เอกสารมีชีวิตช่วงสั้น (ระหว่างถกเถียง) | เอกสารถาวร (immutable) เป็นหลักฐานทางประวัติศาสตร์ |
| ใครอ่าน | ทีมที่เกี่ยวข้องตอนตัดสินใจ | ใครก็ตามในอนาคตที่สงสัยว่า "ทำไมถึงเลือกแบบนี้" |

ในทางปฏิบัติ ADR มักถูกเขียนขึ้น **หลังจาก RFC ได้รับการอนุมัติแล้ว** เพื่อสรุปผลการตัดสินใจไว้อย่างถาวร

### 899.3 Template มาตรฐานของ ADR

```markdown
# ADR-017: เลือกใช้ Event Sourcing สำหรับ Order Service

## Status
Accepted (2026-09-10)

## Context
Order service ปัจจุบันเก็บเฉพาะ state ล่าสุดของ order ทำให้ไม่สามารถ
ตอบคำถามเชิง audit เช่น "order นี้เปลี่ยนสถานะกี่ครั้ง เมื่อไหร่บ้าง"
ซึ่งทีม Compliance ต้องการสำหรับการตรวจสอบตามกฎหมาย

## Decision
เราจะใช้ Event Sourcing pattern สำหรับ Order Service โดยเก็บทุกการ
เปลี่ยนแปลงสถานะเป็น event แยกต่างหาก แทนที่จะ update state ทับของเดิม

## Consequences
### ผลดี
- Audit trail ครบถ้วนตามที่ทีม Compliance ต้องการ
- สามารถ replay event เพื่อ debug ปัญหาย้อนหลังได้

### ผลเสีย / ต้นทุนที่ต้องยอมรับ
- ทีม Analytics ต้องปรับ query pattern ใหม่ทั้งหมด (ดู RFC-039)
- เพิ่มความซับซ้อนในการ debug เบื้องต้นสำหรับวิศวกรใหม่
- ทีม Reporting ต้องสร้าง read-model แยกต่างหาก

## Alternatives Considered
1. **Audit log แยกต่างหาก** — ง่ายกว่าแต่ไม่รับประกัน consistency กับ state จริง
2. **Database trigger บันทึก history table** — ผูกกับ database เฉพาะ ย้าย database ยาก

## ทีมที่ได้รับผลกระทบและรับทราบแล้ว
- Order team (ผู้เสนอ)
- Compliance team (ผู้ร้องขอ)
- Analytics team (ต้องปรับตัว — ดู migration plan ใน RFC-039)
```

### 899.4 เก็บ ADR ไว้ที่ไหน — หลักการเดียวกับ RFC

ADR ควรเก็บใน Git repository เช่นเดียวกับ RFC ด้วยเหตุผลเดียวกัน (ตรวจสอบย้อนหลังได้, ผูกกับ pull request review, ค้นหาง่าย):

```
order-service/
└── docs/
    └── adr/
        ├── 0001-choose-postgresql-over-mongodb.md
        ├── 0002-adopt-event-sourcing.md
        └── template.md
```

หลักการสำคัญ: **ADR ที่ถูก Accept แล้วห้ามแก้ไขย้อนหลัง (immutable)** ถ้าการตัดสินใจเปลี่ยนไปในอนาคต ให้เขียน ADR ใหม่ที่ระบุว่า **"supersedes ADR-002"** แทนการไปแก้ไฟล์เก่า เพื่อรักษาความถูกต้องของประวัติศาสตร์การตัดสินใจไว้ครบถ้วน

```markdown
# ADR-025: เปลี่ยนจาก Event Sourcing กลับมาใช้ State-based Storage

## Status
Accepted (2027-02-01) — Supersedes ADR-002

## Context
หลังใช้งาน Event Sourcing มา 5 เดือน พบว่า operational complexity สูงเกินไป
เทียบกับประโยชน์ที่ได้ ทีม Compliance เปลี่ยนมาใช้ audit log แยกต่างหากแทน...
```

### 899.5 Documentation อื่น ๆ ที่จำเป็นสำหรับ Multi-team Collaboration

นอกจาก RFC และ ADR แล้ว องค์กรที่จัดการ multi-team collaboration ได้ดีมักมีเอกสารประเภทอื่นครบถ้วนดังนี้:

| ประเภทเอกสาร | จุดประสงค์ | เก็บที่ไหน |
|---|---|---|
| **README ของแต่ละ repo** | บอกว่า service ทำอะไร, dependency คืออะไร, ติดต่อทีมไหน | root ของแต่ละ repository |
| **CODEOWNERS** | บอกว่าใครเป็นเจ้าของ path ไหน | root ของแต่ละ repository |
| **CONTRIBUTING.md** | บอกวิธี contribute (สำหรับ InnerSource) | root ของแต่ละ repository |
| **CHANGELOG.md** | บันทึกการเปลี่ยนแปลงแบบ SemVer | root ของแต่ละ repository |
| **RFC** | ขอความเห็นชอบก่อนตัดสินใจใหญ่ | repository กลาง `org/rfcs` |
| **ADR** | บันทึกผลการตัดสินใจเชิงสถาปัตยกรรม | `docs/adr` ในแต่ละ repository |
| **Postmortem** | สรุปบทเรียนจาก incident | repository กลาง `org/postmortems` |
| **Runbook** | ขั้นตอนแก้ปัญหาที่พบบ่อยสำหรับ on-call | `docs/runbooks` ในแต่ละ repository |
| **Service catalog entry** | ข้อมูล metadata ของ service สำหรับค้นหาทั่วองค์กร | ระบบ catalog กลาง (เช่น Backstage) |

### 899.6 "Documentation as Code" — หลักการที่ทำให้เอกสารไม่ล้าสมัย

ปัญหาคลาสสิกของ documentation คือ **มันล้าสมัยเร็วกว่าที่ใครจะอัปเดตทัน** วิธีแก้ที่องค์กรใหญ่ใช้คือหลักการ **"Documentation as Code"**:

1. **เก็บเอกสารไว้ใน Git คู่กับโค้ด** ไม่ใช่แยกไปอยู่ใน wiki หรือ Google Doc ที่ไม่มีใครดูแล
2. **บังคับให้ pull request ที่เปลี่ยนพฤติกรรมสำคัญ ต้องอัปเดตเอกสารไปพร้อมกัน** ผ่าน CI check หรือ PR template checklist
3. **ทำ link checker อัตโนมัติ** ตรวจจับลิงก์เสียในเอกสารเป็นระยะ
4. **Review เอกสารเหมือน review โค้ด** — ผ่านกระบวนการ pull request เดียวกัน ไม่ใช่แก้แล้วเผยแพร่ทันทีโดยไม่มีใครตรวจ

```yaml
# ตัวอย่าง CI check บังคับอัปเดต CHANGELOG เมื่อแก้ shared library
name: Require Changelog Update
on:
  pull_request:
    paths:
      - 'src/**'

jobs:
  check-changelog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: ตรวจสอบว่า CHANGELOG.md ถูกแก้ไขด้วยหรือไม่
        run: |
          if ! git diff --name-only origin/main...HEAD | grep -q "CHANGELOG.md"; then
            echo "::error::กรุณาอัปเดต CHANGELOG.md เมื่อแก้ไข src/"
            exit 1
          fi
```

### 899.7 สรุปหลักการของ Step นี้

> **ในองค์กรที่มีหลายทีม เอกสารที่ดีไม่ใช่ภาระเพิ่มเติม แต่คือโครงสร้างพื้นฐานที่ทำให้ทีมทำงานคู่ขนานกันได้โดยไม่ต้องประชุมทุกเรื่อง — RFC บอกว่า "จะทำอะไรและทำไม" ก่อนลงมือ ADR บอกว่า "ตัดสินใจอะไรไปแล้วและทำไม" หลังลงมือ และทั้งสองอย่างต้องอยู่ใน Git เพื่อให้ตรวจสอบย้อนหลังได้เหมือนโค้ด**

---

## Step 900: แบบฝึกหัด — ออกแบบ collaboration model ให้องค์กรสมมติที่มี 5 ทีมทำงานเกี่ยวข้องกัน

ถึงเวลานำทุกแนวคิดใน Part นี้มาประยุกต์ใช้จริง แบบฝึกหัดนี้จะให้คุณออกแบบ collaboration model แบบครบวงจรสำหรับองค์กรสมมติ

### 900.1 โจทย์: บริษัท "ShopFlow" (แพลตฟอร์ม E-commerce สมมติ)

บริษัท ShopFlow มี 5 ทีมวิศวกรรมดังนี้:

| ทีม | ความรับผิดชอบ | Repository หลัก |
|---|---|---|
| **Platform Team** | ดูแล shared library, CI/CD template, design system, internal SDK | `platform-shared-libs`, `platform-ci-templates`, `platform-design-system` |
| **Checkout Team** | ระบบตะกร้าสินค้าและชำระเงิน | `checkout-service` |
| **Catalog Team** | ระบบสินค้าและการค้นหา | `catalog-service` |
| **Mobile Team** | แอปมือถือ iOS/Android | `shopflow-mobile-app` |
| **Payment Gateway Team** | เชื่อมต่อผู้ให้บริการชำระเงินภายนอก | `payment-gateway-service` |

ความสัมพันธ์ระหว่างทีม:

```
                    ┌─────────────────┐
                    │  Platform Team    │
                    │ (shared libs,     │
                    │  design system,   │
                    │  CI templates)    │
                    └────────┬─────────┘
                             │ ทุกทีมพึ่งพา
        ┌────────────────────┼────────────────────┐
        │                    │                    │
┌───────▼───────┐   ┌────────▼────────┐   ┌───────▼───────┐
│ Checkout Team  │──▶│  Catalog Team    │   │ Mobile Team    │
│                │   │                  │◀──│                │
└───────┬───────┘   └─────────────────┘   └───────┬───────┘
        │                                          │
        │           เรียก API                      │ เรียก API
        ▼                                          ▼
┌──────────────────────────────────────────────────────┐
│              Payment Gateway Team                      │
└──────────────────────────────────────────────────────┘
```

**ความสัมพันธ์:**
- Checkout Team เรียก API ของ Payment Gateway Team และของ Catalog Team (ดึงข้อมูลราคาสินค้า)
- Mobile Team เรียก API ของ Checkout Team, Catalog Team, และ Payment Gateway Team โดยตรง
- ทุกทีมพึ่งพา Platform Team สำหรับ shared library, design system (Mobile/Web UI component), และ CI/CD template

### 900.2 โจทย์ย่อยที่ต้องทำ

ให้ออกแบบและระบุสิ่งต่อไปนี้อย่างละเอียด (สามารถเขียนเป็น diagram + เอกสารสั้น ๆ):

**1. โครงสร้าง Repository**

กำหนดว่าจะใช้ Monorepo หรือ Polyrepo และเพราะเหตุใด พร้อมวาดโครงสร้างโฟลเดอร์/repository ทั้งหมด

**2. CODEOWNERS และ Review Policy**

เขียนไฟล์ `CODEOWNERS` ตัวอย่างสำหรับ `checkout-service` ที่ครอบคลุม:
- โค้ดทั่วไปที่ทีม Checkout ดูแลเอง
- ไฟล์ contract (`contracts/openapi.yaml`) ที่ต้องให้ Payment Gateway Team และ Catalog Team ร่วม review ด้วย เพราะเป็นผู้บริโภค API
- ไฟล์ CI configuration ที่อ้างอิง template จาก Platform Team

**3. API Contract Strategy**

ระบุว่า Checkout Team จะประกาศ breaking change ต่อ API ของตัวเองอย่างไร ให้ทั้ง Mobile Team และ Payment Gateway Team ทราบล่วงหน้า พร้อมออกแบบ CI check ที่ตรวจจับ breaking change อัตโนมัติ

**4. RFC สำหรับสถานการณ์จำลอง**

เขียน RFC ฉบับย่อ (ใช้ template จาก Step 896) สำหรับสถานการณ์: **Catalog Team ต้องการเปลี่ยนโครงสร้างราคาสินค้าจาก single price เป็น multi-currency price object** ซึ่งกระทบ Checkout Team และ Mobile Team โดยตรง ต้องระบุ:
- ทีมที่ได้รับผลกระทบ
- แผน migration แบบ backward-compatible ชั่วคราว
- Timeline การ deprecate field เก่า

**5. Feature Flag Coordination Plan**

ออกแบบแผนการเปิดตัวฟีเจอร์ **"Buy Now Pay Later"** ที่ต้องการทั้ง Checkout Team, Payment Gateway Team, และ Mobile Team deploy พร้อมกัน โดยใช้ feature flag ควบคุมการเปิดใช้งาน ระบุ:
- ลำดับการ deploy ของแต่ละทีม (ใครก่อนใครหลัง และทำไม)
- เงื่อนไขที่ต้องครบก่อนจะเปิด flag
- แผน rollback หากพบปัญหาหลังเปิด

**6. Incident Response Plan**

สมมติว่า Payment Gateway Team deploy โค้ดใหม่แล้วทำให้ Checkout Team และ Mobile Team ได้รับ error พร้อมกันในเวลา 14:32 น. ให้ระบุ:
- ใครควรเป็น Incident Commander ในสถานการณ์นี้ และทำไม
- ทีมไหนบ้างที่ต้องถูกเชิญเข้า war room
- ขั้นตอนตัดสินใจว่าจะ rollback ของ Payment Gateway Team หรือใช้ feature flag ปิดฟีเจอร์ที่เกี่ยวข้องแทน

**7. ADR สรุปผล**

หลังจากสถานการณ์ในข้อ 4 (multi-currency price) ผ่านไปและถูกนำไปใช้งานจริง ให้เขียน ADR สรุปการตัดสินใจนี้ไว้เป็นบันทึกถาวร

### 900.3 เกณฑ์การประเมินตัวเอง (Self-check)

ตรวจสอบงานของคุณด้วยคำถามเหล่านี้:

- [ ] CODEOWNERS ที่ออกแบบครอบคลุมทุกจุดที่ทีมอื่นควรมีสิทธิ์ร่วม review หรือไม่
- [ ] RFC ที่เขียนมีการระบุ "ทีมที่ได้รับผลกระทบ" อย่างครบถ้วน ไม่ตกหล่นทีมไหน
- [ ] แผน migration เป็นแบบ backward-compatible ในช่วงเปลี่ยนผ่านจริงหรือไม่ (ไม่ใช่ breaking ทันที)
- [ ] แผน feature flag มีขั้นตอน "go/no-go" ที่ชัดเจนว่าใครเป็นผู้ตัดสินใจสุดท้าย
- [ ] แผน incident response ระบุ Incident Commander และ escalation path ชัดเจน ไม่คลุมเครือ
- [ ] ADR ที่เขียนบันทึกทั้งผลดี ผลเสีย และทางเลือกอื่นที่เคยพิจารณา ไม่ใช่แค่ผลสรุปเดียว
- [ ] ทุกเอกสารที่ออกแบบ ถูกวางแผนให้เก็บไว้ใน Git ไม่ใช่เครื่องมือภายนอกที่ตรวจสอบย้อนหลังไม่ได้

### 900.4 แนวทางเฉลยโดยสรุป (แนวคิดหลัก ไม่ใช่คำตอบตายตัว)

**โครงสร้าง Repository:** ในกรณีนี้แนะนำ **Polyrepo** เพราะแต่ละทีมมี deployment lifecycle ต่างกันชัดเจน (Mobile app release ผ่าน app store ต่างจาก backend service ที่ deploy ได้ทุกวัน) แต่ต้องมี `platform-shared-libs` และ `platform-design-system` เป็น package ที่ publish ผ่าน internal package registry ให้ทุกทีม `import` ได้

**CODEOWNERS ตัวอย่างสำหรับ checkout-service:**

```
* @shopflow/checkout-team

/contracts/openapi.yaml   @shopflow/checkout-team @shopflow/mobile-team @shopflow/payment-gateway-team
/.github/workflows/       @shopflow/checkout-team @shopflow/platform-team
```

**หลักคิดสำคัญที่ต้องสะท้อนในทุกคำตอบ:** ยิ่งการเปลี่ยนแปลงกระทบทีมไกลตัวมากเท่าไหร่ ต้องมีกระบวนการที่เป็นทางการ (RFC, CODEOWNERS บังคับ, CI check อัตโนมัติ) มากขึ้นเท่านั้น และทุกการตัดสินใจสำคัญต้องถูกบันทึกไว้ใน Git ในรูปแบบที่ตรวจสอบย้อนหลังได้ ไม่พึ่งพาความจำหรือการสื่อสารปากเปล่าเพียงอย่างเดียว

---

## สรุป Part 90

ใน Part นี้เราได้เรียนรู้ว่า:

1. เมื่อองค์กรโตขึ้นจนมีหลายทีมทำงานบน codebase ที่เกี่ยวข้องกัน ปัญหาที่เกิดขึ้นไม่ใช่เรื่องทักษะ Git ส่วนบุคคลอีกต่อไป แต่เป็นเรื่องของ **organization, process, และ tooling** ที่ต้องทำงานร่วมกัน
2. **Platform team vs Product team model** ช่วยแบ่งความรับผิดชอบชัดเจน โดย Platform team ต้องสร้าง "ถนนที่ดี" แบบ self-service ไม่ใช่เป็นคอขวดที่ทุกทีมต้องรอ
3. **API contract และ SemVer** คือสัญญาที่ทำให้ทีมพัฒนาคู่ขนานกันได้อย่างปลอดภัย โดยต้องมี CI ตรวจจับ breaking change อัตโนมัติ ไม่พึ่งความจำคน
4. **Cross-team code review** ต้องถูกบังคับผ่าน CODEOWNERS และ Branch Protection ไม่ใช่แค่มารยาทของทีม
5. **Shared library** ต้องมีเจ้าของที่ชัดเจน และแนวคิด InnerSource ช่วยลดคอขวดโดยเปิดให้ทีมอื่น contribute ได้เอง
6. **RFC process** คือกลไกขอความเห็นชอบก่อนทำ breaking change และ **deprecation notice** ที่ต่อเนื่องคือกุญแจของการ migrate อย่างราบรื่น
7. **Feature flag** ทำให้แยก "การ deploy โค้ด" ออกจาก "การเปิดใช้งานฟีเจอร์" ทำให้หลายทีม coordinate การ release ได้โดยไม่ต้อง deploy พร้อมกันเป๊ะ ๆ
8. **Incident response ข้ามทีม** ต้องมี Incident Commander ที่ชัดเจน และ Service Dependency Map ที่เตรียมไว้ล่วงหน้า
9. **ADR** คือเครื่องมือบันทึกการตัดสินใจเชิงสถาปัตยกรรมให้ตรวจสอบย้อนหลังได้ตลอดไป และ documentation ทุกประเภทควรอยู่ใน Git เพื่อผ่านกระบวนการ review เหมือนโค้ด
10. หลักการทองที่ยึดโยงทุก Step: **ยิ่งการเปลี่ยนแปลงกระทบคนไกลตัวมากเท่าไหร่ ต้นทุนในการสื่อสารล่วงหน้าและความเป็นทางการของกระบวนการต้องสูงขึ้นตามเท่านั้น**

### Checklist ก่อนไป Part 91

- [ ] เข้าใจความแตกต่างระหว่าง Platform team และ Product team
- [ ] เข้าใจว่า API contract และ SemVer ป้องกัน breaking change ได้อย่างไร และรู้จักเครื่องมือตรวจจับอัตโนมัติ เช่น `oasdiff`, `buf breaking`
- [ ] สามารถเขียน CODEOWNERS ที่บังคับ cross-team review ได้
- [ ] เข้าใจความแตกต่างระหว่าง RFC และ ADR และรู้ว่าแต่ละอย่างควรใช้เมื่อไหร่
- [ ] เข้าใจว่า feature flag ช่วย coordinate release ข้ามทีมได้อย่างไร และรู้จักการจัดการ "อายุ" ของ flag
- [ ] เข้าใจโครงสร้างพื้นฐานของ incident response ข้ามทีม รวมถึงบทบาท Incident Commander
- [ ] ทำแบบฝึกหัดออกแบบ collaboration model สำหรับองค์กร 5 ทีมเสร็จสมบูรณ์ครบทั้ง 7 ข้อ

**ต่อไป:** [Part 91: Git ร่วมกับ Agile/Scrum: เชื่อมโยง Sprint กับ Branch/PR](./part-091-git-agile-scrum.md)
