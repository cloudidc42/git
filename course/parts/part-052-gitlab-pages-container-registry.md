# Part 52: GitLab Pages และ Container Registry

> **Step ในหลักสูตรนี้:** Step 511–520
> **เฟส:** 5 — เจาะลึก GitLab และ CI/CD เบื้องต้นของมัน
> **เป้าหมายของ Part นี้:** เข้าใจว่า GitLab Pages คืออะไร ตั้งค่า deploy เว็บไซต์ static ผ่าน `.gitlab-ci.yml` ได้ ผูก custom domain พร้อม TLS ได้ เข้าใจว่า Container Registry ในตัวของ GitLab คืออะไร push/pull Docker image ได้จริง เชื่อม CI/CD ให้ build และ push image อัตโนมัติด้วยตัวแปรของ GitLab เอง รู้จัก Package Registry เบื้องต้นสำหรับ npm/Maven/PyPI/Composer ตั้งค่า cleanup policy จัดการ image เก่า และเข้าใจเรื่องความปลอดภัยของ registry ก่อนลงมือ deploy เว็บไซต์และ Docker image จริงในแบบฝึกหัดท้าย Part

---

## สารบัญของ Part นี้

- Step 511: GitLab Pages คืออะไร
- Step 512: ตั้งค่า deploy GitLab Pages ผ่าน `.gitlab-ci.yml`
- Step 513: Custom domain สำหรับ GitLab Pages
- Step 514: Container Registry ในตัวของ GitLab คืออะไร
- Step 515: การ push/pull Docker image ไปยัง GitLab Container Registry
- Step 516: เชื่อม CI/CD กับ Container Registry อัตโนมัติ
- Step 517: Package Registry ของ GitLab เบื้องต้น
- Step 518: Retention Policy — จัดการ image/package เก่า
- Step 519: ความปลอดภัยของ Registry
- Step 520: แบบฝึกหัด — deploy เว็บไซต์และ push Docker image จริงผ่าน CI/CD

---

## Step 511: GitLab Pages คืออะไร

ใน **Part 25** เราเรียนรู้ GitHub Pages ไปแล้วว่าเป็นบริการ host เว็บไซต์ static ฟรีที่ผูกกับ repository โดยตรง **GitLab Pages** ทำหน้าที่เหมือนกันทุกประการในเชิงแนวคิด แต่มีรายละเอียดปลีกย่อยที่ต่างกันพอสมควร เพราะ GitLab ผูก Pages เข้ากับระบบ CI/CD ของตัวเองอย่างแนบแน่นกว่า GitHub มาก

### นิยาม

> **GitLab Pages คือบริการ host เว็บไซต์ static ฟรีที่มาพร้อมกับทุก project บน GitLab โดยเนื้อหาเว็บไซต์จะถูกสร้างขึ้นผ่าน GitLab CI/CD pipeline แล้วเผยแพร่ออกไปเป็น URL สาธารณะ (หรือส่วนตัวก็ได้)**

จุดที่ต่างจาก GitHub Pages อย่างชัดเจนคือ **GitLab Pages ไม่มีแนวคิดเรื่อง "deploy จาก branch โดยตรง" แบบที่ GitHub มี** (เช่นเลือก branch `gh-pages` แล้วให้ระบบ build ให้อัตโนมัติ) แต่ **GitLab Pages ถูกออกแบบมาให้ deploy ผ่าน CI/CD pipeline เท่านั้น** — นี่คือหัวใจสำคัญที่ต้องเข้าใจตั้งแต่ต้น

### ทำไมต้องผ่าน CI/CD เสมอ

เพราะ GitLab มองว่า Pages เป็นเพียง **ผลลัพธ์ (artifact) อย่างหนึ่งของ pipeline** ไม่ต่างจากไฟล์ binary หรือรายงานการทดสอบ วิธีนี้มีข้อดีคือ:

1. คุณสามารถ build เว็บไซต์จากภาษาหรือ static site generator อะไรก็ได้ (Hugo, Jekyll, Next.js export, VuePress, plain HTML) เพราะขั้นตอน build เป็นแค่ job หนึ่งใน pipeline
2. คุณควบคุมได้เต็มที่ว่าจะ deploy เมื่อไหร่ (เฉพาะ branch `main`, เฉพาะ tag, เฉพาะเมื่อ merge request ถูก merge ฯลฯ) ด้วยกฎ `rules`/`only`/`except` แบบเดียวกับ job อื่น ๆ ที่เราเรียนไปใน Part ก่อนหน้า
3. Pipeline เดียวกันสามารถรัน test, lint, build, deploy ต่อเนื่องกันได้ในขั้นตอนเดียว ไม่ต้องสลับไปใช้ mechanism อื่น

### URL ที่ได้จาก GitLab Pages

รูปแบบ URL ของ GitLab Pages จะขึ้นอยู่กับว่าใช้ GitLab.com (SaaS) หรือ self-hosted GitLab instance:

| ประเภท Project | รูปแบบ URL บน GitLab.com |
|---|---|
| Project ทั่วไป | `https://<namespace>.gitlab.io/<project-slug>` |
| Project ที่ชื่อ `<namespace>.gitlab.io` เป๊ะ ๆ (Group/User Pages) | `https://<namespace>.gitlab.io` (ไม่มี path ต่อท้าย) |
| Group ย่อยซ้อนกัน (subgroup) | `https://<namespace>.gitlab.io/<subgroup>/<project-slug>` |

ตัวอย่างเช่นถ้า username ของคุณคือ `chalermsak` และสร้าง project ชื่อ `portfolio` เว็บไซต์จะอยู่ที่ `https://chalermsak.gitlab.io/portfolio`

บน self-hosted GitLab instance ผู้ดูแลระบบต้องเปิดใช้งาน Pages daemon และตั้งค่า domain กลางไว้ล่วงหน้า (เช่น `*.pages.example.com`) ซึ่งถ้าคุณใช้ GitLab ขององค์กร ให้สอบถามทีม infra ว่า Pages domain ขององค์กรคืออะไร

### สิ่งที่ GitLab Pages รองรับ

- **Static site เท่านั้น** — HTML, CSS, JavaScript ที่ compile/build เสร็จแล้ว ไม่มีการรัน server-side code (เช่น PHP, Node.js runtime) บน Pages โดยตรง
- **HTTPS ฟรีอัตโนมัติ** สำหรับ URL แบบ `*.gitlab.io` (ใช้ certificate ที่ GitLab จัดการให้)
- **รองรับทุก static site generator** เพราะ GitLab ไม่สนใจว่าไฟล์ output มาจากไหน ขอแค่ผลลัพธ์สุดท้ายเป็นไฟล์ static ที่วางในตำแหน่งที่ถูกต้อง (รายละเอียดใน Step 512)
- **Access control ระดับ project visibility** — ถ้า project เป็น private เว็บไซต์ที่ deploy ออกมาก็เข้าถึงได้เฉพาะคนที่ login และมีสิทธิ์เข้า project เท่านั้น (ฟีเจอร์นี้เรียกว่า Pages access control)

### เปรียบเทียบ GitHub Pages กับ GitLab Pages โดยสรุป

| คุณสมบัติ | GitHub Pages | GitLab Pages |
|---|---|---|
| วิธี deploy พื้นฐาน | เลือก branch/folder ได้ตรง ๆ หรือผ่าน Actions | ต้องผ่าน CI/CD pipeline (job ชื่อ `pages`) เท่านั้น |
| ชื่อ job/branch พิเศษ | branch `gh-pages` (แบบดั้งเดิม) หรือ workflow ที่ deploy ผ่าน Actions | job ชื่อ `pages` ใน `.gitlab-ci.yml` |
| โฟลเดอร์ output ที่ต้องใช้ | ขึ้นกับวิธีที่เลือก | ต้องชื่อ `public` เท่านั้น (บังคับ) |
| Custom domain | รองรับ | รองรับ |
| Private site (access control) | รองรับใน plan ที่สูงกว่า | รองรับตั้งแต่ free tier (self-hosted/SaaS ต่างกันเล็กน้อย) |
| จำนวนเว็บไซต์ต่อ 1 account/project | 1 เว็บไซต์ต่อ repo (user/org site มีจำกัด) | 1 เว็บไซต์ต่อ project เช่นกัน แต่ subgroup ซ้อนได้ลึกกว่า |

เมื่อเข้าใจภาพรวมแล้ว มาดูกันว่าการตั้งค่าจริงต้องทำอย่างไรใน Step ถัดไป

---

## Step 512: ตั้งค่า deploy GitLab Pages ผ่าน `.gitlab-ci.yml`

นี่คือหัวใจสำคัญที่สุดของ Part นี้ในส่วนของ Pages เพราะทุกอย่างขึ้นอยู่กับไฟล์ `.gitlab-ci.yml` ที่เราเรียนพื้นฐานไปแล้วใน Part ก่อนหน้าของเฟสนี้

### กฎเหล็ก 2 ข้อที่ต้องจำให้ขึ้นใจ

> **1. Job ที่จะทำหน้าที่ deploy Pages ต้อง**ชื่อ**`pages` เท่านั้น (สงวนชื่อไว้โดย GitLab)**
> **2. ไฟล์ที่จะถูกเผยแพร่เป็นเว็บไซต์ต้องอยู่ใน artifacts path ที่ชื่อ `public` เท่านั้น**

ถ้าคุณตั้งชื่อ job อื่น หรือใส่ output ไว้ในโฟลเดอร์อื่นที่ไม่ใช่ `public` ระบบจะไม่ deploy ให้ ไม่ว่า pipeline จะสำเร็จแค่ไหนก็ตาม เพราะ GitLab Pages daemon จะมองหา artifact ที่ชื่อ `public` จาก job ที่ชื่อ `pages` โดยเฉพาะเท่านั้น

### ตัวอย่างที่ 1: เว็บไซต์ static ธรรมดา (ไม่ต้อง build อะไรเลย)

สมมติว่าคุณมีไฟล์ `index.html`, `style.css` อยู่ใน root ของ repository อยู่แล้ว และต้องการ deploy ตรง ๆ

```yaml
pages:
  stage: deploy
  script:
    - mkdir public
    - cp index.html public/
    - cp style.css public/
  artifacts:
    paths:
      - public
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

อธิบายทีละบรรทัด:

- `pages:` — ชื่อ job ต้องเป๊ะแบบนี้เท่านั้น (ตัวพิมพ์เล็กทั้งหมด)
- `stage: deploy` — ควรอยู่ใน stage ท้าย ๆ ของ pipeline (ไม่บังคับชื่อ stage แต่ควรมาหลัง build/test)
- `script` — สร้างโฟลเดอร์ `public` แล้วก็อบปี้ไฟล์ที่จะ deploy เข้าไป
- `artifacts.paths: [public]` — บอก GitLab ว่าโฟลเดอร์นี้แหละคือสิ่งที่ต้องเก็บเป็น artifact
- `rules` — จำกัดให้ deploy เฉพาะตอน push เข้า branch `main` เท่านั้น (ป้องกันไม่ให้ทุก branch ทดลองไป deploy ทับเว็บไซต์จริง)

### ตัวอย่างที่ 2: เว็บไซต์ที่ต้อง build ด้วย static site generator (เช่น Hugo)

```yaml
image: registry.gitlab.com/pages/hugo:latest

variables:
  GIT_SUBMODULE_STRATEGY: recursive

pages:
  stage: deploy
  script:
    - hugo --minify
  artifacts:
    paths:
      - public
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

จุดสำคัญ: **Hugo build ออกมาที่โฟลเดอร์ `public` เป็นค่า default อยู่แล้ว** ดังนั้นเราแค่ระบุ `artifacts.paths: [public]` ตรง ๆ โดยไม่ต้อง `mkdir`/`cp` เอง — นี่คือเหตุผลที่ static site generator หลายตัว (Hugo, Jekyll แบบที่ตั้งค่า `destination: public`) เลือกใช้ชื่อโฟลเดอร์ output เป็น `public` เป็นค่าเริ่มต้น เพราะออกแบบมาให้เข้ากับ GitLab Pages ได้พอดี

### ตัวอย่างที่ 3: เว็บไซต์ที่ build ด้วย Node.js (React/Vue ที่ build ออกมาเป็นโฟลเดอร์ `dist` หรือ `build`)

หลาย framework (เช่น Vite, Create React App) จะ build ออกมาเป็นโฟลเดอร์ที่ไม่ใช่ `public` (เช่น `dist` หรือ `build`) ดังนั้นต้อง copy/rename ให้ตรงตามกฎ

```yaml
pages:
  stage: deploy
  image: node:20
  script:
    - npm ci
    - npm run build          # สมมติว่า build ออกมาที่โฟลเดอร์ dist/
    - mv dist public
  artifacts:
    paths:
      - public
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

### การใช้ `pages.path_prefix` (GitLab Pages แบบหลาย version พร้อมกัน)

GitLab เวอร์ชันใหม่ ๆ (ตั้งแต่ 16.7 เป็นต้นมา) รองรับ **Pages multiple deployments** ผ่าน keyword `pages.path_prefix` ทำให้คุณ deploy Pages หลายเวอร์ชันพร้อมกันได้ เช่น deploy ทุก branch เป็น preview site แยกกัน:

```yaml
pages:
  stage: deploy
  script:
    - mkdir public
    - cp -r dist/* public/
  pages:
    path_prefix: '$CI_COMMIT_REF_SLUG'
  artifacts:
    paths:
      - public
  rules:
    - if: '$CI_COMMIT_BRANCH'
```

ผลลัพธ์คือ URL จะกลายเป็น `https://namespace.gitlab.io/project/feature-x/` แยกกันไปตามแต่ละ branch ซึ่งมีประโยชน์มากสำหรับทีมที่ต้องการ preview แต่ละ feature branch ก่อน merge

> **หมายเหตุ:** ฟีเจอร์นี้ต้องเปิดใช้งาน "Pages multiple deployments" ใน Settings > Pages ก่อน ถ้าเป็น self-hosted GitLab ผู้ดูแลระบบต้องเปิด feature flag นี้ในระดับ instance ด้วย

### วิธีดูว่า deploy สำเร็จหรือยัง

1. ไปที่ **Settings > Pages** ใน project เพื่อดู URL ที่ได้และสถานะ deployment
2. ไปที่ **Deploy > Pages** (เมนูใหม่ในบาง version) เพื่อดูประวัติการ deploy แต่ละครั้ง
3. ตรวจสอบ pipeline job log ของ job `pages` ว่าจบด้วย status `passed` และมีข้อความยืนยันว่า artifact ถูกอัปโหลด

### ข้อผิดพลาดที่พบบ่อย

| อาการ | สาเหตุที่พบบ่อยที่สุด |
|---|---|
| Job รันผ่านแต่เว็บไซต์ไม่อัปเดต | ตั้งชื่อ job ไม่ใช่ `pages` |
| ขึ้น 404 ทั้งเว็บไซต์ | artifact path ไม่ใช่ `public` หรือไฟล์ `index.html` ไม่ได้อยู่ใน root ของ `public` |
| Deploy ช้ามาก (หลายนาที) | ไฟล์ในโฟลเดอร์ `public` มีขนาดใหญ่เกินไป หรือใช้ runner ที่ทรัพยากรจำกัด |
| Job ไม่ทำงานเลยแม้ push แล้ว | เงื่อนไขใน `rules`/`only` ไม่ตรงกับ branch ที่ push |

---

## Step 513: Custom domain สำหรับ GitLab Pages

เหมือนกับ GitHub Pages ที่เราเรียนใน Part 25 การใช้ domain ของตัวเองแทน `*.gitlab.io` ทำได้ 2 ขั้นตอนหลัก: **ตั้งค่า DNS** และ **ตั้งค่าใน GitLab** พร้อมจัดการ **TLS certificate**

### ขั้นตอนที่ 1: เพิ่ม Domain ใน GitLab

1. ไปที่ **Settings > Pages** ของ project
2. คลิก **New Domain**
3. กรอก domain ของคุณ เช่น `blog.mycompany.com`
4. GitLab จะสร้าง **Verification code** ให้ ต้องนำไปใส่เป็น TXT record ที่ DNS provider เพื่อพิสูจน์ว่าคุณเป็นเจ้าของ domain จริง

### ขั้นตอนที่ 2: ตั้งค่า DNS Record

ต้องเพิ่ม 2 record หลักที่ DNS provider ของคุณ (เช่น Cloudflare, Route 53, GoDaddy):

| ประเภท Record | ค่า (Value) | จุดประสงค์ |
|---|---|---|
| `TXT` | รหัส verification code ที่ GitLab สร้างให้ | พิสูจน์ความเป็นเจ้าของ domain |
| `CNAME` (สำหรับ subdomain เช่น `blog.mycompany.com`) | `namespace.gitlab.io` | ชี้ domain ของคุณไปยัง GitLab Pages |
| `A` (สำหรับ root/apex domain เช่น `mycompany.com`) | IP address ของ GitLab Pages (ดูได้จากเอกสารของ GitLab.com หรือถาม admin ของ self-hosted instance) | ชี้ domain ไปยัง GitLab Pages เมื่อไม่สามารถใช้ CNAME กับ apex ได้ |

ตัวอย่าง record ใน DNS provider:

```
Type: CNAME
Name: blog
Value: mynamespace.gitlab.io
TTL: 3600

Type: TXT
Name: _gitlab-pages-verification-code.blog
Value: gitlab-pages-verification-code=abcdef1234567890
TTL: 3600
```

> **หมายเหตุสำคัญ:** สำหรับ apex domain (เช่น `mycompany.com` โดยไม่มี subdomain) มาตรฐาน DNS ไม่อนุญาตให้ใช้ `CNAME` กับ root ได้ ต้องใช้ `A` record ชี้ไปยัง IP ของ GitLab Pages โดยตรง หรือใช้ฟีเจอร์ `ALIAS`/`ANAME` ถ้า DNS provider รองรับ (เช่น Cloudflare CNAME flattening)

### ขั้นตอนที่ 3: TLS Certificate (HTTPS)

GitLab Pages รองรับ HTTPS สำหรับ custom domain ได้ 2 แบบ:

**แบบที่ 1: Let's Encrypt อัตโนมัติ (แนะนำ)**

GitLab.com (และ self-hosted ที่เปิดฟีเจอร์นี้) จะออก certificate จาก **Let's Encrypt** ให้อัตโนมัติทันทีที่ DNS ถูกตั้งค่าถูกต้องและ verify ผ่าน ไม่ต้องทำอะไรเพิ่มเติม ระบบจะต่ออายุ certificate ให้เองทุกครั้งก่อนหมดอายุ

วิธีเปิดใช้งาน (ถ้ายังไม่ได้เปิดโดยอัตโนมัติ): ไปที่หน้า domain ที่เพิ่มไว้ แล้วติ๊ก **Automatic certificate management using Let's Encrypt**

**แบบที่ 2: อัปโหลด Certificate เอง**

ถ้าองค์กรของคุณมี certificate จาก CA อื่น (เช่น DigiCert, internal CA ขององค์กร) สามารถอัปโหลดเองได้:

1. ไปที่หน้า domain ที่เพิ่มไว้ใน **Settings > Pages**
2. วาง **Certificate (PEM)** และ **Private Key (PEM)** ลงในช่องที่กำหนด
3. บันทึก — GitLab จะใช้ certificate นี้แทนการออกเองผ่าน Let's Encrypt

> **ข้อควรระวัง:** ต้องต่ออายุ certificate ที่อัปโหลดเองก่อนหมดอายุด้วยตัวเอง GitLab จะไม่ต่ออายุให้อัตโนมัติในกรณีนี้ ต่างจากแบบ Let's Encrypt ที่จัดการให้ทั้งหมด

### ระยะเวลาที่ DNS propagate

การเปลี่ยนแปลง DNS อาจใช้เวลา **ไม่กี่นาทีถึง 48 ชั่วโมง** กว่าจะ propagate ไปทั่วโลก (ขึ้นอยู่กับค่า TTL ที่ตั้งไว้และ DNS resolver ของแต่ละที่) ระหว่างรอสามารถตรวจสอบสถานะด้วยคำสั่ง:

```bash
dig blog.mycompany.com CNAME
dig blog.mycompany.com TXT
```

หรือใช้เว็บไซต์ตรวจสอบ DNS propagation แบบ online เพื่อดูว่า record กระจายไปถึงที่ต่าง ๆ ทั่วโลกหรือยัง

---

## Step 514: Container Registry ในตัวของ GitLab คืออะไร

ต่อไปเราจะเปลี่ยนหัวข้อจากการ host เว็บไซต์ static ไปสู่การ host **Docker image** ซึ่งเป็นอีกหนึ่งความสามารถเด่นของ GitLab ที่ทำให้มันถูกเรียกว่า "DevOps Platform ครบวงจร"

### นิยาม

> **GitLab Container Registry คือระบบเก็บและกระจาย Docker image (image registry) ที่ผูกมากับทุก project บน GitLab โดยอัตโนมัติ ไม่ต้องไปสมัครหรือติดตั้งบริการภายนอกเพิ่มเติมเลย**

พูดง่าย ๆ คือ **ทุก project บน GitLab มี Docker Hub ของตัวเองในตัว** โดยไม่มีค่าใช้จ่ายเพิ่มเติม (ภายใต้ข้อจำกัดของ storage ตาม plan ที่ใช้)

### ทำไมถึงสำคัญ

ก่อนหน้านี้ทีมพัฒนาต้องพึ่งพา registry ภายนอกแยกต่างหาก เช่น:

- **Docker Hub** — public registry ที่มีข้อจำกัดเรื่อง rate limit และ private repo มีจำนวนจำกัดในแผนฟรี
- **Amazon ECR**, **Google Artifact Registry**, **Azure Container Registry** — ต้องสมัครบริการ cloud แยกต่างหาก ตั้งค่า credential เพิ่ม

การมี Container Registry ในตัว GitLab ทำให้:

1. **ไม่ต้องสมัครบริการเพิ่ม** — เปิดใช้งานได้ทันทีจาก project settings
2. **Authentication ใช้ระบบเดียวกับ GitLab** — ใช้ personal access token, deploy token หรือ CI/CD job token ที่มีอยู่แล้ว ไม่ต้องจัดการ credential แยกชุด
3. **ผูกกับ Permission ของ project โดยอัตโนมัติ** — ใครมีสิทธิ์เข้า project เท่าไหร่ ก็มีสิทธิ์เข้า registry เท่านั้นเป็นค่าเริ่มต้น
4. **เชื่อมกับ CI/CD ได้แนบเนียนที่สุด** — เพราะ GitLab CI/CD รู้จัก URL ของ registry ตัวเองผ่านตัวแปรที่มีให้พร้อมใช้ (จะพูดถึงใน Step 516)

### โครงสร้างของ Registry ต่อ Project

แต่ละ project จะได้ registry namespace ของตัวเองในรูปแบบ:

```
registry.gitlab.com/<namespace>/<project-slug>
```

ตัวอย่างเช่นถ้า project อยู่ที่ `https://gitlab.com/mycompany/backend-api` registry ของมันจะอยู่ที่:

```
registry.gitlab.com/mycompany/backend-api
```

และภายใน 1 project สามารถมี **หลาย image repository ย่อย** ได้ด้วย โดยการตั้งชื่อ path ต่อท้าย เช่น:

```
registry.gitlab.com/mycompany/backend-api               (image หลัก)
registry.gitlab.com/mycompany/backend-api/worker         (image ย่อยสำหรับ worker)
registry.gitlab.com/mycompany/backend-api/migration-job  (image ย่อยสำหรับ migration)
```

### วิธีเปิดใช้งาน Container Registry

โดยปกติ Container Registry จะถูกเปิดใช้งานเป็นค่า default อยู่แล้วสำหรับ project ใหม่บน GitLab.com ตรวจสอบ/เปิดได้ที่:

1. ไปที่ **Settings > General > Visibility, project features, permissions**
2. เลื่อนหาหัวข้อ **Container Registry** แล้วเปิด toggle ให้เป็น ON
3. เลือกระดับ visibility ของ registry (จะอธิบายละเอียดใน Step 519)

เมื่อเปิดใช้งานแล้ว เมนู **Deploy > Container Registry** (หรือ **Packages and registries > Container Registry** ในบาง version) จะปรากฏขึ้นในแถบเมนูซ้ายของ project ซึ่งเป็นที่ที่คุณจะเห็นรายการ image ทั้งหมดที่เคย push เข้าไป พร้อมขนาดไฟล์ (size), tag, และวันที่ push ล่าสุด

---

## Step 515: การ push/pull Docker image ไปยัง GitLab Container Registry

มาลงมือจริงกันว่าการนำ Docker image ขึ้นไปเก็บใน GitLab Container Registry ทำอย่างไร (สมมติว่าเครื่องของคุณติดตั้ง Docker ไว้แล้ว)

### ขั้นตอนที่ 1: Login เข้า Registry

```bash
docker login registry.gitlab.com
```

ระบบจะถาม **Username** และ **Password** ซึ่ง**ไม่ใช่รหัสผ่านบัญชี GitLab โดยตรง** แต่ต้องใช้อย่างใดอย่างหนึ่งต่อไปนี้แทน:

| วิธี Authentication | เหมาะกับ |
|---|---|
| **Personal Access Token** (scope: `read_registry`, `write_registry`) | ใช้งานจากเครื่อง local ของตัวเอง |
| **Deploy Token** (scope: `read_registry`, `write_registry`) | ใช้งานจาก server หรือระบบภายนอกที่ไม่ใช่ user จริง |
| **CI/CD Job Token** (`$CI_JOB_TOKEN`) | ใช้อัตโนมัติภายใน pipeline เท่านั้น (ไม่ต้องสร้างเองมือ) |

ตัวอย่าง login ด้วย Personal Access Token:

```bash
docker login registry.gitlab.com -u your_username -p glpat-xxxxxxxxxxxxxxxxxxxx
```

> **คำเตือนด้านความปลอดภัย:** การใส่ password ต่อท้ายด้วย `-p` ตรง ๆ ใน command line จะถูกบันทึกไว้ใน shell history ควรใช้วิธี pipe ผ่าน stdin แทนเพื่อความปลอดภัย:
>
> ```bash
> echo "glpat-xxxxxxxxxxxxxxxxxxxx" | docker login registry.gitlab.com -u your_username --password-stdin
> ```

### ขั้นตอนที่ 2: Build Image พร้อม Tag ให้ตรงกับ Registry Path

Docker image ที่จะ push ไปยัง registry ใด ๆ ต้องถูก tag ด้วยชื่อ registry นั้นนำหน้าเสมอ:

```bash
docker build -t registry.gitlab.com/mycompany/backend-api:1.0.0 .
```

หรือถ้า build image ไว้แล้วด้วยชื่ออื่น สามารถใช้ `docker tag` เพื่อสร้าง alias ให้ตรงตาม registry path:

```bash
docker build -t backend-api:1.0.0 .
docker tag backend-api:1.0.0 registry.gitlab.com/mycompany/backend-api:1.0.0
```

### ขั้นตอนที่ 3: Push Image ขึ้น Registry

```bash
docker push registry.gitlab.com/mycompany/backend-api:1.0.0
```

ผลลัพธ์ที่เห็นจะคล้ายกับการ push ไปยัง Docker Hub ทั่วไป คือแสดง layer ที่กำลังอัปโหลดทีละชั้น:

```
The push refers to repository [registry.gitlab.com/mycompany/backend-api]
5f70bf18a086: Pushed
a3ed95caeb02: Pushed
1.0.0: digest: sha256:abcdef1234... size: 1234
```

### ขั้นตอนที่ 4: Pull Image กลับมาใช้งาน

จากเครื่องอื่น (หรือ server ที่จะ deploy) สามารถดึง image กลับมาได้ด้วย:

```bash
docker login registry.gitlab.com
docker pull registry.gitlab.com/mycompany/backend-api:1.0.0
```

### การตรวจสอบ Image ผ่านหน้าเว็บ

หลัง push เสร็จ ไปที่เมนู **Deploy > Container Registry** ใน project จะเห็นรายการ image repository พร้อมรายละเอียด:

- ชื่อ image path เต็ม
- จำนวน tag ทั้งหมด
- ขนาดรวมของแต่ละ tag
- วันที่ push ล่าสุด
- ปุ่มคัดลอกคำสั่ง `docker pull` ไปใช้ได้ทันที

### การตั้งชื่อ Tag ให้เหมาะสม

แนวทางปฏิบัติที่ดีสำหรับการตั้งชื่อ tag ของ image ใน registry มีหลายรูปแบบที่นิยมใช้ร่วมกัน:

```bash
registry.gitlab.com/mycompany/backend-api:latest          # เวอร์ชันล่าสุดของ default branch
registry.gitlab.com/mycompany/backend-api:1.2.3            # Semantic version ตาม release
registry.gitlab.com/mycompany/backend-api:abc1234           # ตาม short commit SHA (ใช้ traceability สูงสุด)
registry.gitlab.com/mycompany/backend-api:main              # ตามชื่อ branch
registry.gitlab.com/mycompany/backend-api:staging            # ตาม environment ที่จะ deploy
```

ในทางปฏิบัติ ทีมมืออาชีพมักจะ push หลาย tag พร้อมกันในครั้งเดียว เช่น push ทั้ง commit SHA และ `latest` ในทุกครั้งที่ build สำเร็จ เพื่อให้ทั้ง traceability (ย้อนกลับไปดู commit ต้นทางได้) และความสะดวก (ใช้ `latest` สำหรับ deploy ทั่วไป) ไปพร้อมกัน

---

## Step 516: เชื่อม CI/CD กับ Container Registry อัตโนมัติ

การ build และ push image ด้วยมือทุกครั้งไม่ใช่วิธีที่ยั่งยืน สิ่งที่ทีมมืออาชีพทำคือให้ **pipeline จัดการ build + push ให้อัตโนมัติทุกครั้งที่มีการเปลี่ยนแปลงโค้ด**

### ตัวแปรพิเศษที่ GitLab เตรียมไว้ให้ฟรี

GitLab CI/CD มีตัวแปร (predefined variables) ที่เกี่ยวกับ registry ให้ใช้ได้ทันทีโดยไม่ต้องตั้งค่าอะไรเพิ่ม:

| ตัวแปร | ความหมาย |
|---|---|
| `$CI_REGISTRY` | URL ของ container registry (เช่น `registry.gitlab.com`) |
| `$CI_REGISTRY_IMAGE` | Path เต็มของ registry สำหรับ project นี้ (เช่น `registry.gitlab.com/mycompany/backend-api`) |
| `$CI_REGISTRY_USER` | Username ที่ CI/CD job ใช้ login (ค่ามาตรฐานคือ `gitlab-ci-token`) |
| `$CI_JOB_TOKEN` | Token ชั่วคราวที่สร้างขึ้นอัตโนมัติเฉพาะ job นั้น ใช้แทน password ตอน login |
| `$CI_COMMIT_SHORT_SHA` | Commit SHA แบบย่อ เหมาะใช้เป็นส่วนหนึ่งของ tag |
| `$CI_COMMIT_REF_SLUG` | ชื่อ branch/tag ที่ถูกแปลงให้ปลอดภัยสำหรับใช้เป็นส่วนหนึ่งของ URL/tag |

### ตัวอย่าง `.gitlab-ci.yml` แบบสมบูรณ์ (Build + Push อัตโนมัติ)

```yaml
stages:
  - build

build-image:
  stage: build
  image: docker:24.0
  services:
    - docker:24.0-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_JOB_TOKEN" "$CI_REGISTRY"
  script:
    - docker build -t "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA" -t "$CI_REGISTRY_IMAGE:latest" .
    - docker push "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA"
    - docker push "$CI_REGISTRY_IMAGE:latest"
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

อธิบายส่วนสำคัญ:

- **`image: docker:24.0`** — ใช้ Docker image ที่มีคำสั่ง `docker` ติดตั้งไว้แล้วเป็น image หลักของ job (เรียกว่า Docker-in-Docker หรือ **DinD**)
- **`services: [docker:24.0-dind]`** — รัน Docker daemon เป็น service เพื่อให้ job สามารถสั่งงาน `docker build`/`docker push` ได้จริงภายใน container
- **`DOCKER_TLS_CERTDIR: "/certs"`** — ตั้งค่าให้ daemon ใช้ TLS ระหว่าง client กับ daemon (มาตรฐานความปลอดภัยที่ GitLab แนะนำ)
- **`docker login -u "$CI_REGISTRY_USER" -p "$CI_JOB_TOKEN" "$CI_REGISTRY"`** — login เข้า registry โดยใช้ token ที่ GitLab สร้างขึ้นเฉพาะ job นี้ **ไม่ต้องสร้าง credential เองเลย** และ token นี้จะหมดอายุอัตโนมัติทันทีที่ job จบ
- **Push 2 tag พร้อมกัน** — ทั้ง commit SHA (สำหรับ traceability) และ `latest` (สำหรับใช้งานทั่วไป)
- **`rules`** — จำกัดให้ build+push เฉพาะตอน push เข้า default branch เท่านั้น ป้องกันไม่ให้ทุก feature branch push image ทับ `latest` โดยไม่ตั้งใจ

### สิทธิ์ของ `$CI_JOB_TOKEN` กับ Registry

ค่า default ของ GitLab คือ `$CI_JOB_TOKEN` ที่สร้างขึ้นในแต่ละ job จะมีสิทธิ์ **push และ pull ได้เฉพาะ registry ของ project ตัวเองเท่านั้น** เว้นแต่จะไปตั้งค่าเพิ่มใน **Settings > CI/CD > Token Access** เพื่ออนุญาตให้ project อื่นเรียกใช้ token นี้ข้าม project ได้ (ใช้ในกรณีที่มี pipeline ของ project A ต้องการ pull image จาก registry ของ project B)

### การ Build แบบ Multi-stage เพื่อลดขนาด Image

แนะนำให้ใช้ multi-stage build ใน `Dockerfile` เพื่อให้ image สุดท้ายที่ push ขึ้น registry มีขนาดเล็กที่สุด ตัวอย่างสำหรับแอปพลิเคชัน Node.js:

```dockerfile
# Stage 1: build
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: production image
FROM node:20-slim
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/main.js"]
```

การเชื่อม CI/CD เข้ากับ registry แบบนี้คือรากฐานของ **Continuous Delivery** ที่แท้จริง — เมื่อ image ถูก push ขึ้น registry อัตโนมัติแล้ว ขั้นตอนถัดไปคือให้ job อื่นใน pipeline (หรือระบบ orchestration เช่น Kubernetes) ดึง image นั้นไป deploy ต่อ ซึ่งเราจะเจาะลึกเรื่อง deployment แบบเต็มรูปแบบใน **เฟส 7: CI/CD เต็มรูปแบบ**

---

## Step 517: Package Registry ของ GitLab เบื้องต้น

นอกจาก Container Registry สำหรับ Docker image แล้ว GitLab ยังมี **Package Registry** ที่ทำหน้าที่คล้ายกันแต่สำหรับ **package ของภาษาโปรแกรมมิ่งต่าง ๆ** แทนที่จะเป็น Docker image

### นิยาม

> **Package Registry คือระบบเก็บและกระจาย package/library ของภาษาโปรแกรมมิ่งต่าง ๆ (เช่น npm package, Maven artifact, Python wheel, PHP Composer package) โดยผูกอยู่กับ GitLab project เช่นเดียวกับ Container Registry**

พูดง่าย ๆ คือถ้า Container Registry คือ "Docker Hub ในตัว" Package Registry ก็คือ **"npm registry / Maven Central / PyPI ในตัว"** ที่ private และผูกกับสิทธิ์ของ project โดยตรง

### รูปแบบ Package ที่รองรับ

GitLab Package Registry รองรับ format หลากหลายภาษา ได้แก่:

| Format | ใช้กับภาษา/เครื่องมือ |
|---|---|
| **npm** | JavaScript/Node.js (`npm install`, `yarn add`) |
| **Maven** | Java/Kotlin (`mvn`, `gradle`) |
| **PyPI** | Python (`pip install`) |
| **Composer** | PHP (`composer require`) |
| **NuGet** | .NET/C# |
| **Conan** | C/C++ |
| **Helm** | Kubernetes Helm charts |
| **Generic packages** | ไฟล์ใด ๆ ที่ไม่มี format เฉพาะ (เช่น เก็บไฟล์ build artifact ทั่วไป) |
| **Terraform Module** | Terraform infrastructure-as-code modules |

### ตัวอย่างเบื้องต้น: Publish npm Package ขึ้น GitLab Package Registry

ขั้นตอนที่ 1 — ตั้งค่า `.npmrc` ให้ชี้ไปยัง GitLab แทน npm registry สาธารณะ (สำหรับ scoped package):

```
@mycompany:registry=https://gitlab.com/api/v4/packages/npm/
//gitlab.com/api/v4/packages/npm/:_authToken=${CI_JOB_TOKEN}
```

ขั้นตอนที่ 2 — ตั้งชื่อ package ใน `package.json` ให้เป็น scoped package ตรงกับ namespace:

```json
{
  "name": "@mycompany/shared-utils",
  "version": "1.0.0",
  "publishConfig": {
    "@mycompany:registry": "https://gitlab.com/api/v4/packages/npm/"
  }
}
```

ขั้นตอนที่ 3 — publish จาก CI/CD job:

```yaml
publish-package:
  stage: deploy
  image: node:20
  script:
    - npm publish
  rules:
    - if: '$CI_COMMIT_TAG'
```

Job นี้จะ publish package เฉพาะตอนที่มีการสร้าง tag เท่านั้น (เช่นเวลา release เวอร์ชันใหม่)

### ตัวอย่างเบื้องต้น: Publish Maven Package

สำหรับโปรเจกต์ Java ที่ใช้ Maven ต้องเพิ่ม distribution management ใน `pom.xml`:

```xml
<distributionManagement>
  <repository>
    <id>gitlab-maven</id>
    <url>https://gitlab.com/api/v4/projects/${CI_PROJECT_ID}/packages/maven</url>
  </repository>
</distributionManagement>
```

แล้วสั่ง deploy ผ่าน CI/CD:

```yaml
deploy-jar:
  stage: deploy
  image: maven:3.9-eclipse-temurin-17
  script:
    - mvn deploy -s ci_settings.xml
  rules:
    - if: '$CI_COMMIT_TAG'
```

### ทำไม Package Registry ถึงสำคัญในองค์กร

1. **แชร์ shared library ภายในทีม/องค์กรได้โดยไม่ต้อง publish เป็น public** — เช่น utility function ที่ใช้ร่วมกันหลาย microservice
2. **ควบคุม dependency ภายในองค์กรได้เต็มที่** — ไม่ต้องพึ่งพา npm/PyPI สาธารณะสำหรับ internal package
3. **ใช้ authentication และ permission ระบบเดียวกับ GitLab** — ไม่ต้องจัดการ credential แยกชุดเหมือนใช้ registry ภายนอก
4. **เห็นทุกอย่างในที่เดียว** — Source code, CI/CD, Container image, และ Package library อยู่ในแพลตฟอร์มเดียวกันหมด สอดคล้องกับแนวคิด "DevOps Platform ครบวงจร" ที่เราพูดถึงใน Part 01

การใช้งาน Package Registry แบบละเอียดในแต่ละภาษาจะไม่ได้ครอบคลุมลึกทั้งหมดใน Part นี้ เพราะเป้าหมายหลักของ Part 52 คือ Pages และ Container Registry แต่หลักการพื้นฐานที่เรียนไปนี้เพียงพอให้คุณเริ่มต้นใช้งานได้จริงเมื่อจำเป็น

---

## Step 518: Retention Policy — จัดการ image/package เก่า

ปัญหาที่พบบ่อยเมื่อใช้งาน Container Registry และ Package Registry ไปนาน ๆ คือ **พื้นที่จัดเก็บ (storage) เต็มไปด้วย image/package เก่าที่ไม่มีใครใช้แล้ว** เพราะทุกครั้งที่ pipeline รัน มันจะ push image ใหม่เข้าไปเรื่อย ๆ โดยไม่มีการลบของเก่าออกเอง

### Cleanup Policy คืออะไร

> **Cleanup Policy คือกฎที่กำหนดให้ GitLab ลบ image tag เก่าใน Container Registry ออกโดยอัตโนมัติตามเงื่อนไขที่ตั้งไว้ ทำงานเป็น background job ตามรอบเวลาที่กำหนด**

### วิธีตั้งค่า Cleanup Policy

ไปที่ **Settings > Packages and registries > Container Registry** แล้วมองหาหัวข้อ **Cleanup policy** ซึ่งมีตัวเลือกให้กำหนดดังนี้:

| ตัวเลือก | ความหมาย |
|---|---|
| **Cleanup policy status** | เปิด/ปิดการทำงานของ cleanup policy |
| **Run cleanup policy** | ความถี่ที่ policy จะทำงาน (ทุกวัน, ทุกสัปดาห์, ทุกเดือน) |
| **Keep the most recent** | จำนวน tag ล่าสุดที่จะเก็บไว้เสมอ ไม่ว่าจะเก่าแค่ไหน (เช่น เก็บ 10 tag ล่าสุด) |
| **Keep tags matching** | Regular expression ของชื่อ tag ที่ต้องการเก็บไว้เสมอ ไม่ให้ถูกลบ (เช่น `^(latest\|main\|v.*)$`) |
| **Remove tags older than** | ลบ tag ที่เก่ากว่าระยะเวลาที่กำหนด (เช่น 90 วัน) |
| **Remove tags matching** | Regular expression ของชื่อ tag ที่ต้องการให้ลบทิ้ง (เช่น tag ที่ตั้งชื่อตาม branch ทดลองที่ลบไปแล้ว) |

### ตัวอย่างการตั้งค่าที่ใช้งานจริงบ่อย

สถานการณ์ทั่วไป: ทีมต้องการเก็บ tag ที่สำคัญไว้เสมอ (release version และ `latest`) แต่ลบ tag ที่เป็น commit SHA ของ feature branch ทดลองที่เก่าเกิน 30 วัน:

```
Keep the most recent: 5
Keep tags matching: ^(latest|v\d+\.\d+\.\d+)$
Remove tags older than: 30 days
Remove tags matching: .*
```

**ลำดับการทำงาน:** GitLab จะเช็ค "Keep" ก่อนเสมอ (เก็บ 5 tag ล่าสุด และ tag ที่ตรงกับ pattern ที่ให้ไว้) จากนั้นค่อยพิจารณาลบ tag ที่เหลือที่ตรงกับเงื่อนไข "Remove" — ดังนั้น tag สำคัญจะไม่มีวันถูกลบทิ้งโดยไม่ตั้งใจ

### ตั้งค่าผ่าน API หรือ `.gitlab-ci.yml` ก็ได้

นอกจากตั้งค่าผ่านหน้าเว็บ ยังสามารถตั้งค่า cleanup policy ผ่าน GitLab API ได้ เช่น:

```bash
curl --request PUT \
  --header "PRIVATE-TOKEN: <your_access_token>" \
  "https://gitlab.com/api/v4/projects/<project_id>" \
  --data "container_expiration_policy_attributes[cadence]=1d" \
  --data "container_expiration_policy_attributes[keep_n]=10" \
  --data "container_expiration_policy_attributes[older_than]=30d" \
  --data "container_expiration_policy_attributes[name_regex_keep]=^(latest|main)$"
```

วิธีนี้เหมาะกับทีมที่ต้องการตั้งค่า cleanup policy ให้เหมือนกันในหลาย project พร้อมกันผ่าน script อัตโนมัติ

### Package Registry ก็มี Cleanup Policy เช่นกัน

ตั้งแต่ GitLab เวอร์ชันใหม่ ๆ มีฟีเจอร์ **Package Registry cleanup policy** แยกต่างหากด้วย เพื่อลบ package เวอร์ชันเก่าที่ไม่ได้ใช้แล้วออกจากพื้นที่จัดเก็บ ตั้งค่าได้ที่ **Settings > Packages and registries > Package Registry**

### ทำไมต้องใส่ใจเรื่องนี้จริงจัง

หากไม่มี cleanup policy เลย เมื่อใช้งาน CI/CD แบบ build+push image ทุก commit ไปนาน ๆ (เช่น 1 ปี ที่มี pipeline รันวันละหลายสิบครั้ง) พื้นที่ registry อาจบวมขึ้นเป็นหลายสิบหรือหลายร้อย GB โดยไม่จำเป็น ซึ่งอาจทำให้:

1. เข้าใกล้ storage quota ของ plan ที่ใช้อยู่ (โดยเฉพาะ GitLab.com free tier ที่มี quota จำกัด)
2. ทำให้การค้นหา/แสดงผลรายการ tag ในหน้าเว็บช้าลง
3. เพิ่มค่าใช้จ่ายโดยไม่จำเป็นสำหรับ self-hosted GitLab ที่เก็บ storage บน cloud (เช่น S3-compatible storage)

การตั้ง cleanup policy ตั้งแต่เริ่มต้นโปรเจกต์จึงเป็นแนวปฏิบัติที่ดีที่ควรทำเป็นมาตรฐาน ไม่ใช่รอให้ storage เต็มแล้วค่อยมาแก้ปัญหาทีหลัง

---

## Step 519: ความปลอดภัยของ Registry

เรื่องความปลอดภัยของทั้ง Container Registry และ Package Registry เป็นประเด็นที่ต้องเข้าใจให้ถูกต้อง เพราะ image/package ที่หลุดออกไปอาจมีข้อมูลลับ (secret, API key ที่ฝังในโค้ด) หรือเปิดช่องให้คนนอกดึง source code ที่ compile ไว้ไปแกะกลับได้

### 1. Visibility ของ Registry ผูกกับ Visibility ของ Project

> **กฎพื้นฐานที่สุด: ถ้า project เป็น private, Container Registry และ Package Registry ของ project นั้นก็จะเป็น private ตามไปด้วยโดยอัตโนมัติ**

| Project Visibility | ผลต่อ Container Registry |
|---|---|
| **Private** | เฉพาะ member ของ project ที่มีสิทธิ์อย่างน้อยระดับ Reporter ขึ้นไปเท่านั้นที่ pull image ได้ ต้อง login ก่อนเสมอ |
| **Internal** | เฉพาะผู้ใช้ที่ login เข้า GitLab instance นั้นแล้วเท่านั้นที่เข้าถึงได้ (ใช้ได้เฉพาะ self-hosted) |
| **Public** | ทุกคนสามารถ `docker pull` ได้โดยไม่ต้อง login แต่การ **push ยังคงต้อง authenticate เสมอ** ไม่ว่า project จะ public แค่ไหนก็ตาม |

นอกจากนี้ GitLab ยังมีการตั้งค่าเพิ่มเติมคือ **"Container Registry visibility"** ที่แยกจาก visibility ของ project ได้ในบางกรณี (เช่นตั้งให้ registry เข้มงวดกว่า project) ผ่าน **Settings > General > Visibility, project features, permissions**

### 2. Deploy Token — วิธีที่ปลอดภัยกว่าการใช้ Personal Access Token

**Deploy Token** คือ token พิเศษที่สร้างขึ้นมาเพื่อให้ **ระบบภายนอก** (เช่น production server, third-party CI system) เข้าถึง registry ได้โดย**ไม่ต้องผูกกับ user account จริงคนใดคนหนึ่ง**

ข้อดีของการใช้ Deploy Token แทน Personal Access Token:

1. **ไม่ผูกกับบัญชีบุคคล** — ถ้าพนักงานคนที่สร้าง Personal Access Token ลาออกและถูกลบบัญชี token นั้นจะใช้งานไม่ได้ทันที แต่ Deploy Token ยังใช้งานต่อได้เพราะผูกกับ project ไม่ใช่ user
2. **จำกัดสิทธิ์ได้ละเอียดกว่า** — เลือกได้ว่าจะให้แค่ `read_registry` (pull อย่างเดียว) หรือ `write_registry` (push ได้ด้วย) โดยไม่ต้องให้สิทธิ์อื่นของ project เลย
3. **ตั้งวันหมดอายุได้** และสามารถ revoke ทิ้งได้ทันทีโดยไม่กระทบ credential อื่น

วิธีสร้าง Deploy Token: ไปที่ **Settings > Repository > Deploy tokens** กรอกชื่อ, วันหมดอายุ (ถ้าต้องการ), เลือก scope (`read_registry`, `write_registry`) แล้วกด **Create deploy token** — ระบบจะแสดง username และ token ให้ครั้งเดียวเท่านั้น ต้องคัดลอกเก็บไว้ทันที

```bash
docker login registry.gitlab.com -u <deploy-token-username> -p <deploy-token-value>
```

### 3. อย่า Hardcode Credential ลงใน Dockerfile หรือ Image

ข้อผิดพลาดร้ายแรงที่พบบ่อยคือการใส่ secret, API key, หรือ credential ต่าง ๆ ลงใน `Dockerfile` โดยตรง เช่น:

```dockerfile
# ผิดมาก ห้ามทำแบบนี้
ENV DATABASE_PASSWORD=supersecret123
```

เพราะค่าที่อยู่ใน layer ของ image (แม้จะลบออกใน layer ถัดไป) **ยังคงถูกเก็บไว้ใน layer history และสามารถแกะออกมาดูได้เสมอ** ด้วยคำสั่งง่าย ๆ อย่าง `docker history` หรือการ inspect layer โดยตรง วิธีที่ถูกต้องคือส่ง secret เข้าไปตอน **runtime** ผ่าน environment variable ของ container orchestration หรือใช้ **Docker BuildKit secret mount** (`RUN --mount=type=secret`) ที่ไม่ฝัง secret ลงใน layer สุดท้าย

### 4. Scan Image หาช่องโหว่ก่อน Deploy จริง

GitLab มีฟีเจอร์ **Container Scanning** (ส่วนหนึ่งของ GitLab Ultimate/Premium หรือใช้ open source scanner เองใน pipeline) ที่ตรวจสอบ image ที่ push ขึ้น registry ว่ามี known vulnerability (CVE) ในตัว base image หรือ dependency หรือไม่ ก่อนที่จะปล่อยให้ deploy ขึ้น production จริง เรื่องนี้จะเจาะลึกอีกครั้งใน **เฟส 8: DevOps, Security, Compliance ระดับองค์กร**

### 5. หลักการ Least Privilege สำหรับสิทธิ์เข้าถึง Registry

สรุปหลักปฏิบัติที่ควรยึดถือ:

- ให้สิทธิ์ **pull only** (`read_registry`) กับระบบที่แค่ต้องดึง image ไปรัน เช่น production server
- ให้สิทธิ์ **push** (`write_registry`) เฉพาะกับ CI/CD pipeline เท่านั้น ไม่ควรให้ developer push image ด้วยมือจากเครื่องตัวเองเข้า production registry โดยตรง
- ใช้ `$CI_JOB_TOKEN` แทน Personal Access Token ภายใน pipeline เสมอเมื่อทำได้ เพราะมีอายุสั้นและจำกัดสิทธิ์อัตโนมัติ
- หมั่นตรวจสอบและ revoke Deploy Token / Personal Access Token ที่ไม่ได้ใช้งานแล้วเป็นประจำ

---

## Step 520: แบบฝึกหัด — deploy เว็บไซต์และ push Docker image จริงผ่าน CI/CD

ถึงเวลาลงมือทำจริงทั้งสองส่วนที่เรียนมาใน Part นี้ แบบฝึกหัดนี้แบ่งเป็น 2 ภารกิจหลัก ทำต่อเนื่องกันในโปรเจกต์เดียวได้เลย

### เตรียมความพร้อม

สร้าง project ใหม่บน GitLab (หรือใช้ project ฝึกฝนที่มีอยู่แล้วจาก Part ก่อนหน้าในเฟสนี้) ชื่อเช่น `pages-registry-lab` แล้ว clone ลงเครื่อง:

```bash
git clone https://gitlab.com/<your-namespace>/pages-registry-lab.git
cd pages-registry-lab
```

### ภารกิจที่ 1: Deploy เว็บไซต์ผ่าน GitLab Pages

**ขั้นตอนที่ 1** — สร้างไฟล์เว็บไซต์เบื้องต้น:

```bash
mkdir site
```

สร้างไฟล์ `site/index.html`:

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>ทดสอบ GitLab Pages</title>
</head>
<body>
  <h1>สวัสดี GitLab Pages</h1>
  <p>เว็บไซต์นี้ deploy ผ่าน CI/CD Part 52 ของหลักสูตร</p>
</body>
</html>
```

**ขั้นตอนที่ 2** — สร้าง `.gitlab-ci.yml` ที่ root ของ repository:

```yaml
stages:
  - deploy

pages:
  stage: deploy
  script:
    - mkdir public
    - cp site/index.html public/
  artifacts:
    paths:
      - public
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

**ขั้นตอนที่ 3** — Commit และ push:

```bash
git add site/index.html .gitlab-ci.yml
git commit -m "เพิ่มเว็บไซต์ทดสอบและตั้งค่า GitLab Pages"
git push origin main
```

**ขั้นตอนที่ 4** — ตรวจสอบผลลัพธ์:

1. ไปที่ **CI/CD > Pipelines** รอให้ job `pages` รันจนจบด้วยสถานะ **passed**
2. ไปที่ **Settings > Pages** จะเห็น URL ของเว็บไซต์ปรากฏขึ้น (รูปแบบ `https://<namespace>.gitlab.io/pages-registry-lab`)
3. เปิด URL นั้นในเบราว์เซอร์ ต้องเห็นข้อความ "สวัสดี GitLab Pages" ที่เขียนไว้

**เกณฑ์ผ่าน:** เข้าถึงเว็บไซต์ผ่าน URL สาธารณะได้จริงและเห็นเนื้อหาที่ถูกต้อง

### ภารกิจที่ 2: Build และ Push Docker Image ผ่าน CI/CD

**ขั้นตอนที่ 1** — สร้างแอปพลิเคชันง่าย ๆ สำหรับทดสอบ สร้างไฟล์ `Dockerfile` ที่ root:

```dockerfile
FROM nginx:alpine
COPY site/index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

**ขั้นตอนที่ 2** — เพิ่ม job สำหรับ build และ push image เข้าไปใน `.gitlab-ci.yml` เดิม (เพิ่ม stage ใหม่):

```yaml
stages:
  - build
  - deploy

build-image:
  stage: build
  image: docker:24.0
  services:
    - docker:24.0-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_JOB_TOKEN" "$CI_REGISTRY"
  script:
    - docker build -t "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA" -t "$CI_REGISTRY_IMAGE:latest" .
    - docker push "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA"
    - docker push "$CI_REGISTRY_IMAGE:latest"
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'

pages:
  stage: deploy
  script:
    - mkdir public
    - cp site/index.html public/
  artifacts:
    paths:
      - public
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

**ขั้นตอนที่ 3** — Commit และ push:

```bash
git add Dockerfile .gitlab-ci.yml
git commit -m "เพิ่ม job build และ push Docker image ไปยัง Container Registry"
git push origin main
```

**ขั้นตอนที่ 4** — ตรวจสอบผลลัพธ์:

1. ไปที่ **CI/CD > Pipelines** ตรวจสอบว่า job `build-image` และ `pages` รันผ่านทั้งคู่
2. ไปที่ **Deploy > Container Registry** ต้องเห็น image repository พร้อม 2 tag คือ commit SHA และ `latest`
3. ทดสอบ pull image กลับมาที่เครื่อง local เพื่อยืนยันว่าใช้งานได้จริง:

```bash
docker login registry.gitlab.com
docker pull registry.gitlab.com/<your-namespace>/pages-registry-lab:latest
docker run -p 8080:80 registry.gitlab.com/<your-namespace>/pages-registry-lab:latest
```

เปิดเบราว์เซอร์ไปที่ `http://localhost:8080` ต้องเห็นเนื้อหาเดียวกับที่ deploy ไว้บน GitLab Pages

**เกณฑ์ผ่าน:** pull image จาก Container Registry มารันบนเครื่อง local ได้สำเร็จ และเนื้อหาที่แสดงผลถูกต้อง

### ภารกิจเสริม (ไม่บังคับ แต่แนะนำให้ลองทำ)

1. ตั้งค่า **Cleanup Policy** ให้เก็บแค่ 3 tag ล่าสุด และลบ tag ที่เก่ากว่า 7 วัน (ตามที่เรียนใน Step 518)
2. สร้าง **Deploy Token** ที่มีสิทธิ์ `read_registry` เท่านั้น แล้วลองใช้ token นั้น `docker pull` image จากเครื่องอื่นดู เพื่อพิสูจน์ว่า token ที่มีสิทธิ์จำกัดใช้งานได้จริงแม้ไม่มีสิทธิ์ push
3. ลองเปลี่ยน project ให้เป็น **Public** แล้วสังเกตว่า `docker pull` โดยไม่ login ทำได้แล้ว แต่ `docker push` ยังคง login ไม่ได้ถ้าไม่มี token ที่ถูกต้อง (ตามหลักที่เรียนใน Step 519)

### Checklist ก่อนไป Part 53

ก่อนไปต่อ Part 53 ให้ตรวจสอบว่าคุณ:

- [ ] เข้าใจว่า GitLab Pages ต้อง deploy ผ่าน CI/CD เท่านั้น ไม่มีทางลัดแบบเลือก branch ตรง ๆ เหมือน GitHub Pages
- [ ] จำกฎเหล็ก 2 ข้อได้ขึ้นใจ: job ต้องชื่อ `pages` และ artifact path ต้องชื่อ `public`
- [ ] ตั้งค่า custom domain พร้อม DNS record และเข้าใจว่า Let's Encrypt จัดการ TLS ให้อัตโนมัติ
- [ ] เข้าใจว่าทุก project บน GitLab มี Container Registry ในตัวโดยไม่ต้องสมัครบริการเพิ่ม
- [ ] `docker login`, `docker build`, `docker tag`, `docker push` ไปยัง `registry.gitlab.com` ได้ด้วยตัวเอง
- [ ] เขียน `.gitlab-ci.yml` ที่ใช้ `$CI_REGISTRY_IMAGE` และ `$CI_JOB_TOKEN` เพื่อ build+push image อัตโนมัติได้
- [ ] รู้จัก Package Registry เบื้องต้นและรู้ว่ารองรับ format ใดบ้าง
- [ ] ตั้งค่า Cleanup Policy เพื่อจัดการ image เก่าได้
- [ ] เข้าใจความสัมพันธ์ระหว่าง project visibility กับ registry visibility และรู้จักการใช้ Deploy Token
- [ ] Deploy เว็บไซต์ผ่าน GitLab Pages และ push Docker image ไปยัง Container Registry สำเร็จจริงทั้งคู่

---

## สรุป Part 52

ใน Part นี้เราได้เรียนรู้ว่า:

1. **GitLab Pages** คือบริการ host เว็บไซต์ static ฟรีที่ผูกกับทุก project แต่ต่างจาก GitHub Pages ตรงที่ต้อง deploy ผ่าน CI/CD pipeline เท่านั้น โดยมีกฎเหล็ก 2 ข้อคือ job ต้องชื่อ `pages` และ artifact path ต้องชื่อ `public`
2. Custom domain สำหรับ GitLab Pages ตั้งค่าผ่าน DNS record (`CNAME`/`A` และ `TXT` verification) และ TLS certificate สามารถให้ Let's Encrypt จัดการอัตโนมัติ หรืออัปโหลดเองก็ได้
3. **Container Registry** คือระบบเก็บ Docker image ในตัวของทุก project บน GitLab ไม่ต้องพึ่งพา Docker Hub หรือ registry ภายนอก
4. การ push/pull image ทำผ่านคำสั่ง Docker มาตรฐาน (`docker login`, `docker build`, `docker tag`, `docker push`) โดยเปลี่ยนแค่ registry path เป็น `registry.gitlab.com/<namespace>/<project>`
5. CI/CD เชื่อมกับ Container Registry ได้อย่างไร้รอยต่อผ่านตัวแปรสำเร็จรูปอย่าง `$CI_REGISTRY_IMAGE` และ `$CI_JOB_TOKEN` ที่ไม่ต้องสร้าง credential เอง
6. **Package Registry** รองรับ package หลากหลายภาษา (npm, Maven, PyPI, Composer, NuGet, Helm ฯลฯ) ทำให้แชร์ library ภายในองค์กรได้โดยไม่ต้อง publish สู่สาธารณะ
7. **Cleanup Policy** ช่วยจัดการ image/package เก่าที่ไม่ได้ใช้แล้วให้ถูกลบอัตโนมัติตามเงื่อนไข ป้องกัน storage เต็มโดยไม่จำเป็น
8. ความปลอดภัยของ registry ผูกกับ project visibility โดยตรง และควรใช้ Deploy Token หรือ CI/CD Job Token แทน Personal Access Token เมื่อเป็นไปได้ ตามหลัก Least Privilege
9. ปิดท้ายด้วยการลงมือ deploy เว็บไซต์จริงผ่าน GitLab Pages และ build+push Docker image จริงผ่าน CI/CD จนสามารถ pull กลับมารันบนเครื่องได้สำเร็จ

**ต่อไป:** [Part 53: GitLab Groups, Permission และการจัดการทีม](./part-053-gitlab-groups-permission.md)
