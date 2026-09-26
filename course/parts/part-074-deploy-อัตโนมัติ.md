# Part 74: Deploy อัตโนมัติ: Docker, Kubernetes, Cloud

> **Step ในหลักสูตรนี้:** Step 731–740
> **เฟส:** 7 — CI/CD เต็มรูปแบบด้วย GitHub Actions และ GitLab CI/CD
> **เป้าหมายของ Part นี้:** เข้าใจ deployment strategies หลักทั้ง 4 แบบ (Recreate, Rolling, Blue-Green, Canary) พร้อมรู้ว่าแต่ละแบบเหมาะกับสถานการณ์ไหน สามารถเขียน pipeline job ที่ build และ push Docker image ไปยัง Container Registry อัตโนมัติ ต่อยอดไปสู่การ deploy image นั้นไปยัง Kubernetes ด้วย `kubectl apply` จาก pipeline โดยใช้ kubeconfig ผ่าน CI variable รู้จักภาพรวมผิวเผินของการ deploy ไปยัง Cloud provider หลัก (AWS/GCP/Azure) เกริ่นแนวคิด Infrastructure as Code เป็นสะพานไปสู่ Part 77 เข้าใจการแยก configuration ตาม environment เขียน rollback strategy เมื่อ deploy พัง ทำ health check และ smoke test หลัง deploy เข้าใจแนวคิด zero-downtime deployment และปิดท้ายด้วยแบบฝึกหัดลงมือ deploy จริงไปยัง platform ฟรีอัตโนมัติจาก pipeline

---

## สารบัญของ Part นี้

- Step 731: Deployment Strategies ภาพรวม — Recreate, Rolling, Blue-Green, Canary
- Step 732: Deploy Docker Image ไปยัง Container Registry อัตโนมัติจาก Pipeline
- Step 733: Deploy ไปยัง Kubernetes เบื้องต้นด้วย `kubectl apply` จาก Pipeline
- Step 734: Deploy ไปยัง Cloud Provider ภาพรวมผิวเผิน (AWS/GCP/Azure)
- Step 735: เกริ่น Infrastructure as Code (IaC)
- Step 736: Environment-specific Configuration — แยก Config ของ Dev/Staging/Production
- Step 737: Rollback Strategy เมื่อ Deploy พัง
- Step 738: Health Checks และ Smoke Tests หลัง Deploy
- Step 739: Zero-downtime Deployment — แนวคิดที่ผู้ใช้ไม่รู้สึกถึง Downtime
- Step 740: แบบฝึกหัด — Deploy อัตโนมัติไปยัง Platform ฟรีเมื่อ Merge เข้า Main

---

## Step 731: Deployment Strategies ภาพรวม — Recreate, Rolling, Blue-Green, Canary

ก่อนจะลงมือเขียน pipeline job สำหรับ deploy จริง เราต้องเข้าใจก่อนว่า **"deploy" ไม่ได้มีวิธีเดียว** การเลือกวิธี deploy ที่เหมาะสมส่งผลโดยตรงต่อ **downtime**, **ความเสี่ยง**, และ **ต้นทุนทรัพยากร** ของระบบ

> **Deployment Strategy คือรูปแบบ/ขั้นตอนที่ใช้ในการนำเวอร์ชันใหม่ของแอปพลิเคชันไปแทนที่เวอร์ชันเก่าที่กำลังรันอยู่จริง (production)**

เราจะเจาะลึก 4 แบบหลักที่ใช้กันมากที่สุดในอุตสาหกรรม

### 1. Recreate — หยุดของเก่า แล้วเริ่มของใหม่

วิธีที่ง่ายและตรงไปตรงมาที่สุด: **ปิด instance เวอร์ชันเก่าทั้งหมดก่อน แล้วค่อยเริ่ม instance เวอร์ชันใหม่**

```
เวลา:     t0          t1          t2
          │           │           │
เวอร์ชันเก่า: [■■■■■■■■■]     (ปิดทั้งหมด)
เวอร์ชันใหม่:            (รอ)    [■■■■■■■■■] (เริ่มทำงาน)
                         ▲
                    ช่วง DOWNTIME
                  (ไม่มี instance ใดทำงานเลย)
```

**ข้อดี:**
- ง่ายที่สุดในการทำความเข้าใจและ implement
- ไม่ต้องกังวลเรื่องมี 2 เวอร์ชันรันพร้อมกัน (ไม่มีปัญหาความเข้ากันได้ของ schema/API ระหว่างเวอร์ชัน)
- ใช้ทรัพยากรน้อยที่สุด (ไม่ต้องมี instance ซ้ำซ้อนระหว่างเปลี่ยนผ่าน)

**ข้อเสีย:**
- **มี downtime แน่นอน** — ผู้ใช้จะเข้าระบบไม่ได้ในช่วงเปลี่ยนผ่าน
- ไม่เหมาะกับระบบที่ต้องการ availability สูง (เช่น e-commerce, banking)

**เหมาะกับ:** ระบบ internal ที่ยอมรับ downtime สั้น ๆ ได้, batch job, environment สำหรับพัฒนา/ทดสอบ

### 2. Rolling Update — ทยอยเปลี่ยนทีละส่วน

แทนที่จะปิดทุก instance พร้อมกัน **Rolling Update จะทยอยแทนที่ instance เก่าด้วย instance ใหม่ทีละตัว (หรือทีละกลุ่มเล็ก ๆ)** โดยตลอดเวลายังมี instance บางส่วนทำงานรับ traffic อยู่เสมอ

```
รอบที่ 1:  [เก่า][เก่า][เก่า][เก่า]  →  [ใหม่][เก่า][เก่า][เก่า]
รอบที่ 2:  [ใหม่][เก่า][เก่า][เก่า]  →  [ใหม่][ใหม่][เก่า][เก่า]
รอบที่ 3:  [ใหม่][ใหม่][เก่า][เก่า]  →  [ใหม่][ใหม่][ใหม่][เก่า]
รอบที่ 4:  [ใหม่][ใหม่][ใหม่][เก่า]  →  [ใหม่][ใหม่][ใหม่][ใหม่]

ตลอดกระบวนการ: มี instance อย่างน้อย 3 ตัวรับ traffic ได้เสมอ → ไม่มี downtime
```

**ข้อดี:**
- **ไม่มี downtime** (หรือมีน้อยมาก) เพราะมี instance ทำงานอยู่เสมอ
- ใช้ทรัพยากรเพิ่มขึ้นเพียงเล็กน้อยระหว่างการเปลี่ยนผ่าน (ไม่ต้อง double ทั้งหมด)
- เป็นค่า default ของ Kubernetes Deployment อยู่แล้ว (`RollingUpdate` strategy)

**ข้อเสีย:**
- **มีช่วงที่ 2 เวอร์ชันรันพร้อมกัน** — ต้องออกแบบให้ทั้งเวอร์ชันเก่าและใหม่เข้ากันได้กับ database schema และ API เดียวกัน (backward compatible)
- Rollback ช้ากว่า Blue-Green เพราะต้องทยอยย้อนกลับทีละส่วนเช่นกัน
- ตรวจับปัญหาได้ยากกว่าถ้า bug เกิดเฉพาะบาง instance

**เหมาะกับ:** งานส่วนใหญ่ในระดับ production ทั่วไป เป็นค่า default ที่แนะนำสำหรับ web application/API

### 3. Blue-Green — สลับสองสภาพแวดล้อมทั้งชุด

แนวคิดคือมี **สภาพแวดล้อมสองชุดที่เหมือนกันทุกประการ** เรียกว่า **Blue** (เวอร์ชันปัจจุบันที่รับ traffic จริง) และ **Green** (เวอร์ชันใหม่ที่เพิ่ง deploy เสร็จ แต่ยังไม่รับ traffic จริง)

```
                    ┌─────────────┐
                    │Load Balancer│
                    └──────┬──────┘
                           │ (traffic ทั้งหมดชี้มาที่ Blue)
              ┌────────────┴────────────┐
              ▼                          ▼
      ┌───────────────┐         ┌───────────────┐
      │  BLUE (v1.0)   │         │  GREEN (v2.0)  │
      │  รับ traffic   │         │  รอทดสอบ       │
      │  จริง 100%     │         │  (ยังไม่รับ    │
      │                │         │   traffic จริง) │
      └───────────────┘         └───────────────┘

หลังทดสอบ Green ผ่านหมด → สลับ Load Balancer ให้ชี้ไป Green ทันที
              ▼
      ┌───────────────┐         ┌───────────────┐
      │  BLUE (v1.0)   │         │  GREEN (v2.0)  │
      │  standby       │         │  รับ traffic   │
      │  (เผื่อ rollback)│        │  จริง 100%     │
      └───────────────┘         └───────────────┘
```

**ข้อดี:**
- **สลับได้แทบจะทันที (instant switch)** — เพียงเปลี่ยนปลายทางของ load balancer/DNS
- **Rollback เร็วมาก** — แค่สลับกลับไปที่ Blue เท่านั้น เพราะ Blue ยังคงรันอยู่เป็น standby
- ทดสอบ Green ในสภาพแวดล้อมที่เหมือน production เป๊ะได้ก่อนปล่อยจริง

**ข้อเสีย:**
- **ใช้ทรัพยากรสองเท่า** ระหว่างช่วงที่ทั้งสองชุดยังรันพร้อมกัน (ต้นทุนสูงกว่า Rolling)
- ปัญหาเรื่อง database/state ที่ใช้ร่วมกัน — ถ้า schema เปลี่ยน ต้องออกแบบให้รองรับทั้งสองเวอร์ชันในช่วงเปลี่ยนผ่าน

**เหมาะกับ:** ระบบที่ต้องการ rollback เร็วมาก และยอมรับต้นทุนทรัพยากรที่เพิ่มขึ้นได้ เช่น ระบบการเงิน, ระบบที่มี SLA เข้มงวด

### 4. Canary — ปล่อยเวอร์ชันใหม่ให้ผู้ใช้กลุ่มเล็กก่อน

**Canary Deployment** ปล่อยเวอร์ชันใหม่ให้รับ traffic จริงเพียง **สัดส่วนเล็ก ๆ** ก่อน (เช่น 5%) แล้วค่อย ๆ เพิ่มสัดส่วนขึ้นเรื่อย ๆ ถ้าไม่พบปัญหา

```
เฟส 1:  Traffic 95% → v1.0 (เวอร์ชันเก่า)
        Traffic  5% → v2.0 (เวอร์ชันใหม่ / Canary)
                        │
                 ตรวจสอบ metrics (error rate, latency)
                        │
เฟส 2:  Traffic 70% → v1.0
        Traffic 30% → v2.0
                        │
                 ตรวจสอบต่อ ถ้ายังปกติ...
                        │
เฟส 3:  Traffic  0% → v1.0
        Traffic 100% → v2.0  (deploy สำเร็จเต็มรูปแบบ)
```

ชื่อ "Canary" มาจากธรรมเนียมของคนงานเหมืองถ่านหินในอดีตที่นำ **นกคีรีบูน (canary)** ลงไปในเหมืองก่อน เพราะนกไวต่อแก๊สพิษมากกว่ามนุษย์ — ถ้านกเป็นอันตราย คนงานจะรู้ทันทีว่าอากาศในเหมืองไม่ปลอดภัยโดยที่ยังไม่ต้องเสี่ยงชีวิตคนเข้าไปเต็มจำนวน

**ข้อดี:**
- **ความเสี่ยงต่ำที่สุด** — ถ้าเวอร์ชันใหม่มีบั๊ก จะกระทบผู้ใช้แค่ส่วนน้อย ไม่ใช่ทั้งระบบ
- สามารถเก็บ metrics จริงจากผู้ใช้จริงมาตัดสินใจก่อนปล่อยเต็มรูปแบบ
- รวมกับ **feature flag** ได้อย่างลงตัว

**ข้อเสีย:**
- **ซับซ้อนที่สุดในการ implement** — ต้องมีระบบ traffic splitting ที่ควบคุมสัดส่วนได้ (เช่น Service Mesh อย่าง Istio, หรือ Ingress Controller ที่รองรับ canary)
- ต้องมีระบบ monitoring/metrics ที่ดีมากเพื่อตัดสินใจว่าจะเพิ่มสัดส่วนหรือ rollback
- ใช้เวลานานกว่าจะ deploy เต็มรูปแบบ (เป็นชั่วโมงหรือเป็นวัน ไม่ใช่ทันที)

**เหมาะกับ:** บริษัทเทคโนโลยีขนาดใหญ่ที่มี traffic มหาศาลและต้องการลดความเสี่ยงให้ต่ำที่สุด เช่น Google, Netflix, Facebook

### ตารางเปรียบเทียบสรุป

| Strategy | Downtime | ต้นทุนทรัพยากร | ความเร็วในการ Rollback | ความซับซ้อน |
|---|---|---|---|---|
| Recreate | มี | ต่ำสุด | เร็ว (deploy กลับเวอร์ชันเก่า) | ต่ำสุด |
| Rolling Update | ไม่มี/น้อยมาก | ต่ำ-ปานกลาง | ปานกลาง | ปานกลาง |
| Blue-Green | ไม่มี | สูง (double) | เร็วที่สุด | ปานกลาง-สูง |
| Canary | ไม่มี | ปานกลาง-สูง | เร็ว (ลดสัดส่วนกลับเป็น 0) | สูงสุด |

ตลอด Part นี้เราจะเน้นที่ **Rolling Update** เป็นหลัก เพราะเป็นค่า default ของ Kubernetes และเหมาะกับงานส่วนใหญ่ที่สุด แต่จะพูดถึงแนวคิด Blue-Green และ Canary ประกอบเพื่อให้เห็นภาพครบทุกทางเลือก

---

## Step 732: Deploy Docker Image ไปยัง Container Registry อัตโนมัติจาก Pipeline

ใน **Part 52** เราเรียนรู้ GitLab Container Registry ไปแล้วว่าเป็น registry ในตัวที่มาพร้อมทุก project และเรียนรู้วิธี build + push image ด้วยมือ รวมถึงตัวอย่าง pipeline job ที่ push อัตโนมัติเมื่อเข้า default branch ใน Part นี้เราจะต่อยอดแนวคิดนั้นให้เป็นขั้นตอนแรกของ **pipeline การ deploy แบบเต็มรูปแบบ**

### ทบทวนแนวคิด: Registry คือ "สะพาน" ระหว่าง CI กับ Deploy

```
┌─────────┐    ┌──────────┐    ┌──────────────┐    ┌─────────┐
│  Commit │───▶│   Build  │───▶│   Registry   │───▶│  Deploy │
│  โค้ด   │    │  Docker  │    │ (เก็บ image) │    │  ไปยัง  │
│         │    │  Image   │    │              │    │  Server │
└─────────┘    └──────────┘    └──────────────┘    └─────────┘
```

Registry ทำหน้าที่เป็น **จุดกลาง** ที่แยก "ขั้นตอน build" ออกจาก "ขั้นตอน deploy" อย่างชัดเจน — job ที่ build ไม่จำเป็นต้องรู้เลยว่า image จะถูก deploy ไปที่ไหน และ job ที่ deploy ก็ไม่จำเป็นต้อง build เอง เพียงแค่ **pull image ที่มี tag ตรงกันจาก registry** มาใช้งาน

### Pipeline ตัวอย่างที่แยก Stage ชัดเจน

```yaml
stages:
  - build
  - push
  - deploy

variables:
  IMAGE_TAG: "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA"

build-and-push:
  stage: push
  image: docker:24.0
  services:
    - docker:24.0-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_JOB_TOKEN" "$CI_REGISTRY"
  script:
    - docker build -t "$IMAGE_TAG" -t "$CI_REGISTRY_IMAGE:latest" .
    - docker push "$IMAGE_TAG"
    - docker push "$CI_REGISTRY_IMAGE:latest"
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'

deploy-production:
  stage: deploy
  image: alpine:3.19
  script:
    - echo "จะ deploy image ${IMAGE_TAG} ไปยัง production ในขั้นตอนต่อไป"
  needs:
    - build-and-push
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

จุดสำคัญที่ต้องสังเกต:

1. **`variables.IMAGE_TAG`** — กำหนด tag ของ image ไว้ครั้งเดียวที่ระดับบนสุดของไฟล์ แล้วให้ทุก job อ้างอิงตัวแปรเดียวกัน เพื่อไม่ให้เกิดความผิดพลาดจากการพิมพ์ tag ไม่ตรงกันระหว่าง job build กับ job deploy
2. **`needs: [build-and-push]`** — บังคับให้ job `deploy-production` รอให้ job push สำเร็จก่อนเสมอ และยังทำให้ pipeline รันเร็วขึ้นด้วย (ไม่ต้องรอ stage อื่นที่ไม่เกี่ยวข้อง ถ้ามี)
3. **ใช้ commit SHA เป็น tag หลัก ไม่ใช่ `latest`** — เพราะ `latest` เป็น tag ที่เปลี่ยนความหมายไปเรื่อย ๆ ทุกครั้งที่ push การ deploy ด้วย tag ที่อิง commit SHA ทำให้เรารู้แน่ชัดเสมอว่า "production กำลังรันโค้ดจาก commit ไหน" ซึ่งสำคัญมากสำหรับการ debug และ rollback (จะพูดถึงใน Step 737)

### เชื่อมกับ GitHub Actions และ GitHub Container Registry (ghcr.io)

ถ้าใช้ GitHub Actions แนวคิดเดียวกันนี้ทำได้ผ่าน GitHub Container Registry (`ghcr.io`) เช่นกัน:

```yaml
name: build-and-push

on:
  push:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:${{ github.sha }}
            ghcr.io/${{ github.repository }}:latest
```

จะเห็นว่าหลักการเหมือนกันทุกประการไม่ว่าจะใช้ platform ไหน: **login → build → tag ด้วยข้อมูลที่ traceable (commit SHA) → push** ความแตกต่างมีแค่ syntax และชื่อตัวแปรที่แต่ละ platform เตรียมให้

### สิ่งที่ต้องระวังเมื่อ Automate การ Push Image

- **อย่า push ทุก branch ขึ้น registry เดียวกันโดยไม่มี prefix/tag แยก** — จะทำให้ registry เต็มไปด้วย image ที่ไม่มีใครใช้ ควรใช้ `rules`/`if` จำกัดเฉพาะ branch ที่ต้องการจริง ๆ (เช่น `main`, tag ที่ขึ้นต้นด้วย `v`)
- **ตั้ง Cleanup Policy** ตามที่เรียนไปใน Part 52 เพื่อไม่ให้ image เก่าสะสมจนเปลืองพื้นที่
- **อย่า hardcode credential ในไฟล์ pipeline** — ใช้ตัวแปรที่ระบบเตรียมให้ (`$CI_JOB_TOKEN`, `secrets.GITHUB_TOKEN`) เสมอ ไม่ต้องสร้าง token เองในกรณีที่ deploy ไปยัง registry ของ platform เดียวกัน

---

## Step 733: Deploy ไปยัง Kubernetes เบื้องต้นด้วย `kubectl apply` จาก Pipeline

เมื่อ image อยู่ใน registry เรียบร้อยแล้ว ขั้นตอนถัดไปคือการสั่งให้ **Kubernetes cluster** ไป pull image เวอร์ชันใหม่มารันแทนเวอร์ชันเก่า Step นี้จะโฟกัสที่การทำสิ่งนี้ **จากภายใน pipeline โดยอัตโนมัติ** ไม่ใช่การรันคำสั่งด้วยมือจากเครื่อง local

> หมายเหตุ: Part นี้สอนเฉพาะแนวคิดและวิธีเชื่อม CI/CD เข้ากับ Kubernetes ที่มีอยู่แล้วเท่านั้น ส่วนพื้นฐานของ Kubernetes เอง (Pod, Deployment, Service, YAML manifest) จะไม่ลงลึกซ้ำในนี้ เพราะถือว่าอยู่นอกขอบเขตของหลักสูตร Git/CI-CD สายนี้ — สมมติว่ามี cluster และ Deployment manifest พร้อมใช้งานอยู่แล้ว

### แนวคิดหลัก: Pipeline คือ "ผู้สั่งงาน" ไม่ใช่ "ผู้รัน"

Pipeline job ไม่ได้รันแอปพลิเคชันเอง แต่ทำหน้าที่ **สื่อสารกับ Kubernetes API Server** เพื่อบอกว่า "ให้เปลี่ยนไปใช้ image เวอร์ชันนี้" แล้ว Kubernetes control plane จะเป็นผู้จัดการ Rolling Update ให้เองทั้งหมด

```
┌───────────┐    kubectl apply     ┌─────────────────┐
│  Pipeline │ ────────────────────▶│  Kubernetes API  │
│   Job     │  (แก้ image tag)      │      Server      │
└───────────┘                      └─────────┬────────┘
                                              │ สั่งงาน
                                    ┌─────────▼────────┐
                                    │  Rolling Update   │
                                    │  ทยอยเปลี่ยน Pod   │
                                    └───────────────────┘
```

### การเตรียม Kubeconfig สำหรับใช้ใน Pipeline

`kubectl` ต้องมีไฟล์ **kubeconfig** เพื่อรู้ว่าจะเชื่อมต่อไปยัง cluster ไหนและยืนยันตัวตนอย่างไร ปัญหาคือเราไม่สามารถเก็บไฟล์นี้ไว้ใน repository ได้เพราะมันมี credential ที่มีสิทธิ์เข้าถึง cluster เต็มรูปแบบ วิธีที่ถูกต้องคือ:

1. **Export kubeconfig ของ service account ที่มีสิทธิ์จำกัดเฉพาะที่ต้องใช้** (ไม่ใช้ credential แบบ cluster-admin เต็มรูปแบบ)
2. **แปลงไฟล์เป็น base64** เพื่อเก็บเป็นข้อความบรรทัดเดียว:

```bash
cat kubeconfig.yaml | base64 -w 0
```

3. **นำค่าที่ได้ไปตั้งเป็น CI/CD Variable** ชื่อ เช่น `KUBE_CONFIG_B64` โดยตั้งค่าเป็น **Protected** (ใช้ได้เฉพาะ protected branch/tag) และ **Masked** (ไม่แสดงค่าจริงใน job log) — ทั้งสอง flag นี้เรียนไปแล้วใน Part ก่อนหน้าเรื่อง CI/CD Variables

### Pipeline Job สำหรับ Deploy ไปยัง Kubernetes

```yaml
deploy-k8s:
  stage: deploy
  image: bitnami/kubectl:1.29
  script:
    - mkdir -p ~/.kube
    - echo "$KUBE_CONFIG_B64" | base64 -d > ~/.kube/config
    - kubectl config current-context
    - kubectl set image deployment/backend-api
        backend-api="$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA"
        --namespace=production
    - kubectl rollout status deployment/backend-api --namespace=production --timeout=180s
  environment:
    name: production
    url: https://app.example.com
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

อธิบายทีละบรรทัด:

- **`echo "$KUBE_CONFIG_B64" | base64 -d > ~/.kube/config`** — ถอดรหัส base64 กลับเป็นไฟล์ kubeconfig จริง แล้ววางไว้ในตำแหน่งที่ `kubectl` มองหาโดยอัตโนมัติ
- **`kubectl set image deployment/backend-api backend-api=<image>`** — คำสั่งหลักที่สั่งให้ Deployment ชื่อ `backend-api` เปลี่ยน container ชื่อ `backend-api` ไปใช้ image tag ใหม่ คำสั่งนี้จะทำให้ Kubernetes เริ่ม **Rolling Update ทันที** ตามที่อธิบายไว้ใน Step 731
- **`kubectl rollout status ... --timeout=180s`** — รอจนกว่า Rolling Update จะเสร็จสมบูรณ์ (หรือ timeout ใน 180 วินาที) คำสั่งนี้สำคัญมาก เพราะทำให้ pipeline **รอผลจริง** ไม่ใช่แค่ยิงคำสั่งแล้วจบทันที ถ้า rollout ล้มเหลว (เช่น image pull ไม่ได้ หรือ container crash) job จะ **fail ทันที** ทำให้ทีมรู้ปัญหาเร็วที่สุด
- **`environment: name: production`** — ใช้ฟีเจอร์ GitLab Environments เพื่อให้เห็นประวัติการ deploy ทั้งหมดของ environment นี้ในหน้า **Deployments > Environments** พร้อมลิงก์ไปยัง URL จริงของแอป

### ทางเลือกอื่นนอกจาก `kubectl set image`

ในทีมที่จัดการ manifest แบบเต็มรูปแบบ มักใช้คำสั่งที่กว้างกว่านี้:

```bash
kubectl apply -f k8s/deployment.yaml -f k8s/service.yaml --namespace=production
```

วิธีนี้เหมาะเมื่อ manifest มีการเปลี่ยนแปลงโครงสร้างอื่นด้วย (ไม่ใช่แค่เปลี่ยน image tag) เช่น เพิ่ม environment variable ใหม่ เปลี่ยนจำนวน replica หรือเพิ่ม resource limit โดย manifest มักถูกเก็บไว้ในโฟลเดอร์ `k8s/` ภายใน repository เดียวกัน และใช้เครื่องมืออย่าง `envsubst` หรือ `sed` แทนที่ placeholder ของ image tag ก่อน apply:

```bash
sed "s|__IMAGE_TAG__|$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA|g" k8s/deployment.yaml.template > k8s/deployment.yaml
kubectl apply -f k8s/deployment.yaml --namespace=production
```

แนวทางนี้เป็นจุดเริ่มต้นง่าย ๆ ของแนวคิด **templating manifest** ซึ่งในโลกจริงมักใช้เครื่องมือที่ทรงพลังกว่า เช่น **Helm** หรือ **Kustomize** — แต่สำหรับ Part นี้ขอเน้นแค่หลักการพื้นฐานให้เข้าใจก่อน

---

## Step 734: Deploy ไปยัง Cloud Provider ภาพรวมผิวเผิน (AWS/GCP/Azure)

Kubernetes ไม่ใช่ทางเลือกเดียวในการรันแอปพลิเคชัน หลายทีมเลือก deploy ตรงไปยังบริการของ **Cloud Provider** รายใหญ่แทน Step นี้จะให้ภาพรวม **ผิวเผินระดับพอเข้าใจ** ว่าแต่ละเจ้ามีเครื่องมืออะไรให้ใช้ในบริบทของ CI/CD บ้าง — ไม่ได้ลงลึกถึงระดับ hands-on เพราะแต่ละ provider มีรายละเอียดมากพอที่จะเป็นหลักสูตรแยกต่างหากได้เลย

### ภาพรวม: ทุก Cloud Provider มี CLI Tool เป็นของตัวเอง

หัวใจของการ deploy ไปยัง cloud จาก pipeline คือการติดตั้ง **CLI tool** ของ provider นั้น ๆ ลงใน job แล้วใช้ **credential ที่เก็บเป็น CI/CD Variable** ยืนยันตัวตน จากนั้นสั่งคำสั่ง deploy ผ่าน CLI เหมือนที่เราทำกับ `kubectl`

### Amazon Web Services (AWS)

| เครื่องมือ | ใช้ทำอะไร |
|---|---|
| **AWS CLI** (`aws`) | คำสั่งพื้นฐานสำหรับสื่อสารกับบริการ AWS แทบทุกตัว |
| **Amazon ECR (Elastic Container Registry)** | Container Registry ของ AWS เอง (คู่แข่งของ GitLab/GitHub Registry) |
| **Amazon ECS (Elastic Container Service)** | รัน container โดยไม่ต้องจัดการ Kubernetes เอง |
| **Amazon EKS (Elastic Kubernetes Service)** | Kubernetes cluster ที่ AWS จัดการ control plane ให้ |
| **AWS Elastic Beanstalk** | Deploy แอปพลิเคชันแบบง่าย ไม่ต้องยุ่งกับ infrastructure เลย |
| **AWS CodeDeploy** | บริการ deploy อัตโนมัติในตัวของ AWS |

ตัวอย่างคำสั่งคร่าว ๆ ใน pipeline job (แนวคิด ไม่ใช่ค่าจริง):

```bash
aws ecs update-service \
  --cluster my-cluster \
  --service backend-api-service \
  --force-new-deployment
```

Credential ที่ต้องตั้งเป็น CI/CD Variable: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`

### Google Cloud Platform (GCP)

| เครื่องมือ | ใช้ทำอะไร |
|---|---|
| **gcloud CLI** | CLI หลักของ GCP ครอบคลุมแทบทุกบริการ |
| **Artifact Registry** | Container/Package Registry ของ GCP (รุ่นใหม่แทน Container Registry เดิม) |
| **GKE (Google Kubernetes Engine)** | Kubernetes cluster ที่ GCP จัดการให้ |
| **Cloud Run** | รัน container แบบ serverless จ่ายตามการใช้งานจริง เหมาะกับ API/เว็บที่ traffic ไม่สม่ำเสมอ |

ตัวอย่างคำสั่งคร่าว ๆ:

```bash
gcloud run deploy backend-api \
  --image="asia-southeast1-docker.pkg.dev/my-project/my-repo/backend-api:latest" \
  --region=asia-southeast1 \
  --platform=managed
```

Credential มักใช้ **Service Account key (JSON)** เก็บเป็น CI/CD Variable แบบ File type แล้วสั่ง `gcloud auth activate-service-account` ก่อนใช้งาน

### Microsoft Azure

| เครื่องมือ | ใช้ทำอะไร |
|---|---|
| **Azure CLI** (`az`) | CLI หลักของ Azure |
| **Azure Container Registry (ACR)** | Container Registry ของ Azure |
| **AKS (Azure Kubernetes Service)** | Kubernetes cluster ที่ Azure จัดการให้ |
| **Azure App Service** | Platform สำหรับ deploy เว็บแอป/API แบบไม่ต้องจัดการ server |

ตัวอย่างคำสั่งคร่าว ๆ:

```bash
az webapp deployment container config \
  --name backend-api \
  --resource-group my-resource-group \
  --docker-custom-image-name "myregistry.azurecr.io/backend-api:latest"
```

Credential มักใช้ **Service Principal** (คล้าย service account) ยืนยันตัวตนผ่าน `az login --service-principal`

### หลักการที่เหมือนกันในทุก Cloud Provider

แม้ชื่อคำสั่งและบริการจะต่างกัน แต่รูปแบบการเชื่อม CI/CD เข้ากับ cloud provider ใด ๆ ก็ตามมีโครงสร้างเดียวกันเสมอ:

1. **ติดตั้ง CLI tool ของ provider นั้นใน job** (ผ่าน `image` ที่มี CLI ติดตั้งไว้แล้ว หรือติดตั้งเองใน `before_script`)
2. **ยืนยันตัวตนด้วย credential ที่เก็บเป็น CI/CD Variable** (ไม่มีทาง hardcode ใน repository เด็ดขาด)
3. **สั่งคำสั่ง deploy** ที่ provider นั้นเตรียมไว้ให้
4. **รอผลลัพธ์และตรวจสอบสถานะ** ก่อนให้ job จบสำเร็จ

ด้วยเหตุนี้ ไม่ว่าจะต้อง deploy ไปยัง provider ไหน ทักษะ CI/CD ที่เรียนมาตลอดหลักสูตรนี้ (variables, secrets, stages, rules, environments) ก็ยังใช้ได้เหมือนเดิมทั้งหมด สิ่งที่เปลี่ยนแค่ **เครื่องมือปลายทาง** เท่านั้น

---

## Step 735: เกริ่น Infrastructure as Code (IaC)

จนถึงตอนนี้เราสมมติว่า **Kubernetes cluster มีอยู่แล้ว**, **service ของ cloud provider ถูกสร้างไว้แล้ว** — แต่คำถามคือ **ใครเป็นคนสร้างสิ่งเหล่านั้นตั้งแต่แรก และสร้างอย่างไร**

ในอดีต ทีม Ops มักสร้าง infrastructure ด้วยมือผ่านหน้าเว็บ console ของ cloud provider (คลิกปุ่มสร้าง server, ตั้งค่า network เอง) วิธีนี้มีปัญหาใหญ่คล้ายกับปัญหาที่ Version Control เคยแก้ให้กับซอร์สโค้ดมาแล้ว:

- **ไม่มีประวัติ** ว่าใครเปลี่ยนอะไรใน infrastructure เมื่อไหร่
- **สร้างซ้ำไม่ได้** — ถ้า server พังจะสร้างสภาพแวดล้อมเดิมขึ้นมาใหม่แบบเป๊ะ ๆ แทบเป็นไปไม่ได้
- **Environment ไม่ตรงกัน** ระหว่าง dev/staging/production เพราะสร้างด้วยมือคนละครั้ง คนละคน

> **Infrastructure as Code (IaC) คือแนวคิดการนิยาม infrastructure (server, network, database, Kubernetes cluster ฯลฯ) ด้วยไฟล์ configuration ที่เป็นข้อความ (เก็บใน Git ได้เหมือนซอร์สโค้ด) แทนการสร้างด้วยมือผ่านหน้าเว็บ**

### เครื่องมือ IaC ที่นิยมใช้

| เครื่องมือ | ลักษณะเด่น |
|---|---|
| **Terraform** | รองรับ Cloud provider แทบทุกเจ้าผ่านระบบ "provider" — เป็นเครื่องมือที่นิยมที่สุดในตลาดปัจจุบัน |
| **AWS CloudFormation** | เครื่องมือ IaC เฉพาะของ AWS |
| **Pulumi** | เขียน infrastructure ด้วยภาษาโปรแกรมจริง (Python, TypeScript) แทน syntax เฉพาะ |
| **Ansible** | เน้น configuration management มากกว่าสร้าง infrastructure ใหม่ทั้งหมด |

### ตัวอย่างแนวคิด (ไม่ใช่โค้ดที่รันได้จริง เพื่อให้เห็นภาพ)

```hcl
resource "cloud_kubernetes_cluster" "main" {
  name       = "production-cluster"
  node_count = 3
  region     = "asia-southeast1"
}
```

ไฟล์แบบนี้ถูก commit เข้า Git repository เหมือนซอร์สโค้ดทั่วไป และสามารถผ่าน pipeline ของตัวเองได้เช่นกัน (เรียกว่า **GitOps** เมื่อนำ Git มาเป็นแหล่งความจริงเดียวของทั้ง code และ infrastructure) — เมื่อมีคน commit การเปลี่ยนแปลงไฟล์นี้ pipeline จะรัน `terraform plan` เพื่อแสดงว่าจะมีอะไรเปลี่ยนแปลงบ้าง แล้วให้คนอนุมัติก่อนรัน `terraform apply` จริง

### ทำไม Part นี้ไม่ลงลึกเรื่อง IaC

Infrastructure as Code เป็นหัวข้อที่ใหญ่พอที่จะต้องมี Part แยกเฉพาะ เพราะเกี่ยวข้องกับแนวคิดเพิ่มเติมอีกมาก เช่น state file, module, provider configuration, การจัดการ secret ภายใน IaC เอง Part นี้ขอเพียงให้เข้าใจว่า **IaC มีอยู่จริง และเป็นขั้นก่อนหน้าของสิ่งที่เราเพิ่งเรียนใน Step 733-734** (นั่นคือ ก่อนจะ `kubectl apply` ได้ ต้องมี cluster อยู่ก่อน และ IaC คือวิธีที่ทีมมืออาชีพใช้สร้าง cluster นั้นอย่างเป็นระบบ)

เราจะเจาะลึกเรื่อง Infrastructure as Code แบบเต็มรูปแบบ ทั้ง Terraform, state management, และการผูกเข้ากับ CI/CD pipeline ใน **Part 77** ของหลักสูตรนี้

---

## Step 736: Environment-specific Configuration — แยก Config ของ Dev/Staging/Production

แอปพลิเคชันเดียวกันมักต้องรันในหลาย environment ที่มี configuration ต่างกัน เช่น connection string ของ database, URL ของ API ภายนอก, ระดับการ log, feature flag ที่เปิด/ปิดต่างกัน **ห้ามเด็ดขาด**ที่จะ hardcode ค่าเหล่านี้ลงในโค้ดหรือ Docker image โดยตรง เพราะจะทำให้ต้อง build image แยกสำหรับแต่ละ environment ซึ่งขัดกับหลักการที่ว่า **"image เดียวกันควรรันได้ในทุก environment โดยเปลี่ยนแค่ configuration"**

### หลักการ: Twelve-Factor App เรื่อง Config

แนวคิดนี้มาจากหลักการที่รู้จักกันในชื่อ **Twelve-Factor App** ซึ่งเป็นแนวปฏิบัติมาตรฐานสำหรับสร้างแอปพลิเคชันสมัยใหม่:

> **"Store config in the environment"** — เก็บ configuration ที่เปลี่ยนไปตาม environment ไว้ใน **environment variable** ไม่ใช่ในไฟล์โค้ด

```
                    ┌──────────────────────┐
                    │   Docker Image        │
                    │   (image เดียวกัน      │
                    │    ทุก environment)    │
                    └───────────┬────────────┘
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                    ▼
      ┌───────────────┐ ┌───────────────┐   ┌───────────────┐
      │      Dev       │ │    Staging     │   │  Production    │
      │ DB_HOST=dev-db │ │DB_HOST=stg-db  │   │DB_HOST=prod-db │
      │ LOG_LEVEL=debug│ │LOG_LEVEL=info  │   │LOG_LEVEL=warn  │
      └───────────────┘ └───────────────┘   └───────────────┘
```

### การตั้งค่า CI/CD Variable แยกตาม Environment (Scoped Variables)

ทั้ง GitLab และ GitHub รองรับการตั้งค่าตัวแปรที่ **ใช้เฉพาะบาง environment เท่านั้น** (เรียกว่า **Environment Scope** ใน GitLab หรือ **Environment Secrets** ใน GitHub)

ตัวอย่างการตั้งค่าใน GitLab (**Settings > CI/CD > Variables**):

| Key | Value | Environment scope |
|---|---|---|
| `DATABASE_URL` | `postgres://dev-db.internal:5432/app` | `development` |
| `DATABASE_URL` | `postgres://staging-db.internal:5432/app` | `staging` |
| `DATABASE_URL` | `postgres://prod-db.internal:5432/app` | `production` |

ตัวแปรชื่อเดียวกันสามารถมีได้หลายค่า โดยระบบจะเลือกใช้ค่าที่ตรงกับ environment ของ job นั้น ๆ โดยอัตโนมัติ ทำให้ pipeline job ไม่ต้องมี logic แยกเงื่อนไขเองเลย — แค่ประกาศ `environment: name: staging` ใน job นั้น GitLab ก็จะดึงค่าที่ scope ตรงกันมาให้อัตโนมัติ

```yaml
deploy-staging:
  stage: deploy
  environment:
    name: staging
    url: https://staging.example.com
  script:
    - echo "Deploying with DATABASE_URL scoped to staging"
    - kubectl set env deployment/backend-api DATABASE_URL="$DATABASE_URL" --namespace=staging

deploy-production:
  stage: deploy
  environment:
    name: production
    url: https://app.example.com
  script:
    - echo "Deploying with DATABASE_URL scoped to production"
    - kubectl set env deployment/backend-api DATABASE_URL="$DATABASE_URL" --namespace=production
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

### วิธีส่ง Config เข้าไปในแอปพลิเคชันที่รันจริง

ในระดับ Kubernetes การส่ง environment variable เข้า Pod ทำได้หลายวิธี:

1. **ConfigMap** — สำหรับค่าที่ไม่ลับ เช่น `LOG_LEVEL`, `API_BASE_URL`
2. **Secret** — สำหรับค่าที่ลับ เช่น `DATABASE_PASSWORD`, `API_KEY` (เก็บแบบ base64-encoded และควรผูกกับระบบจัดการ secret ภายนอกในโปรดักชันจริง)
3. **`kubectl set env`** — เปลี่ยนค่า environment variable ของ Deployment ที่มีอยู่แล้วโดยตรงจาก pipeline (ตามตัวอย่างด้านบน)

### ข้อควรระวัง

- **อย่าใส่ค่าลับ (secret) เป็น plain text ในไฟล์ manifest ที่ commit เข้า Git** — ควรใช้ Kubernetes Secret ที่จัดการแยกจาก manifest หลัก หรือใช้เครื่องมือจัดการ secret ภายนอก เช่น HashiCorp Vault, AWS Secrets Manager
- **แยกให้ชัดว่าตัวแปรไหนเป็น build-time (ต้องมีตอน build image) กับ runtime (มีตอนรันเท่านั้น)** — โดยทั่วไป configuration ที่เปลี่ยนตาม environment ควรเป็น runtime เสมอ เพื่อให้ image เดียวกันใช้ได้ทุกที่ตามหลักการ Twelve-Factor App ที่กล่าวไปข้างต้น
- **ตั้งชื่อตัวแปรให้สื่อความหมายชัดเจนและสอดคล้องกันทุก environment** เช่น ใช้ `DATABASE_URL` เหมือนกันทุกที่ ต่างกันแค่ค่า ไม่ใช่ตั้งชื่อ `DEV_DB_URL`, `PROD_DATABASE_CONNECTION` ที่ไม่สอดคล้องกัน

---

## Step 737: Rollback Strategy เมื่อ Deploy พัง

ไม่ว่าจะระวังแค่ไหน **การ deploy พังคือเรื่องที่เกิดขึ้นได้เสมอ** สิ่งที่แยกทีมมืออาชีพออกจากทีมมือใหม่ไม่ใช่การไม่เคย deploy พังเลย แต่คือ **ความเร็วในการกู้คืนกลับสู่สถานะปกติ**

### หลักการสำคัญ: Rollback ต้องเร็วกว่า Debug เสมอ

เมื่อ production มีปัญหา priority อันดับหนึ่งคือ **หยุดผลกระทบต่อผู้ใช้ก่อน** ไม่ใช่นั่งหาสาเหตุกลางดึกว่าทำไมมันพัง วิธีที่เร็วที่สุดในการหยุดผลกระทบคือการ **ย้อนกลับไปยังเวอร์ชันก่อนหน้าที่รู้อยู่แล้วว่าทำงานได้ปกติ** แล้วค่อยไปหาสาเหตุอย่างสงบทีหลัง

### วิธีที่ 1: Revert to Previous Image Tag

เพราะเราใช้ **commit SHA เป็น image tag** เสมอ (ตามที่ทำไว้ใน Step 732) การ rollback จึงทำได้ง่ายมาก — แค่สั่งให้ Kubernetes เปลี่ยนกลับไปใช้ tag ของ commit ก่อนหน้า:

```bash
kubectl set image deployment/backend-api \
  backend-api="$CI_REGISTRY_IMAGE:abc1234previous" \
  --namespace=production
```

ในทางปฏิบัติ ทีมมักเตรียม **pipeline job สำหรับ rollback โดยเฉพาะ** ที่รับ commit SHA เป็นพารามิเตอร์ผ่าน manual trigger:

```yaml
rollback-production:
  stage: deploy
  image: bitnami/kubectl:1.29
  script:
    - mkdir -p ~/.kube
    - echo "$KUBE_CONFIG_B64" | base64 -d > ~/.kube/config
    - |
      if [ -z "$ROLLBACK_TO_SHA" ]; then
        echo "กรุณาระบุตัวแปร ROLLBACK_TO_SHA ก่อนรัน job นี้"
        exit 1
      fi
    - kubectl set image deployment/backend-api
        backend-api="$CI_REGISTRY_IMAGE:$ROLLBACK_TO_SHA"
        --namespace=production
    - kubectl rollout status deployment/backend-api --namespace=production
  environment:
    name: production
  when: manual
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

`when: manual` ทำให้ job นี้ **ไม่รันอัตโนมัติ** แต่รอให้คนกดปุ่ม trigger เองใน GitLab UI พร้อมกรอกค่า `ROLLBACK_TO_SHA` — ป้องกันการ rollback โดยไม่ตั้งใจ แต่ยังทำให้ทำได้เร็วมากในสถานการณ์ฉุกเฉินจริง (ไม่ต้องแก้โค้ดใหม่หรือรอ pipeline ทั้งชุดรันใหม่)

### วิธีที่ 2: `kubectl rollout undo`

Kubernetes เก็บ **ประวัติของ Deployment ไว้ในตัวเองโดยอัตโนมัติ** (revision history) ทำให้สามารถสั่ง rollback กลับไปยังเวอร์ชันก่อนหน้าได้โดยไม่ต้องรู้ image tag เดิมด้วยซ้ำ:

```bash
# ดูประวัติ revision ทั้งหมดของ Deployment นี้
kubectl rollout history deployment/backend-api --namespace=production

# ย้อนกลับไปยัง revision ก่อนหน้าล่าสุด (revision - 1)
kubectl rollout undo deployment/backend-api --namespace=production

# ย้อนกลับไปยัง revision ที่ระบุเจาะจง
kubectl rollout undo deployment/backend-api --namespace=production --to-revision=3
```

คำสั่งนี้ทำงานได้เพราะ Kubernetes เก็บ `ReplicaSet` เก่าไว้เบื้องหลังตามจำนวนที่กำหนดใน `revisionHistoryLimit` ของ Deployment (ค่า default คือ 10) เมื่อสั่ง `rollout undo` มันจะสลับกลับไปใช้ `ReplicaSet` เก่านั้นทันที ด้วยกลไก Rolling Update แบบเดียวกับตอน deploy ปกติ (จึงไม่มี downtime ระหว่าง rollback เช่นกัน)

### เปรียบเทียบสองวิธี

| | `kubectl set image` ย้อนกลับ | `kubectl rollout undo` |
|---|---|---|
| ต้องรู้ image tag เดิมไหม | ต้องรู้ | ไม่ต้องรู้ (Kubernetes จำให้) |
| ควบคุมได้ละเอียดแค่ไหน | ระบุ tag ใดก็ได้ตามต้องการ | จำกัดตาม revision history ที่เก็บไว้ |
| เหมาะกับ | Pipeline ที่ integrate กับระบบ tracking image tag ของตัวเองอยู่แล้ว | สถานการณ์ฉุกเฉินเร่งด่วนที่ต้องการคำสั่งเดียวจบ |

### หลักปฏิบัติที่ดีเพิ่มเติม

1. **เก็บ image เก่าไว้ใน registry เสมอ อย่าลบทิ้งเร็วเกินไป** — Cleanup Policy ควรเก็บอย่างน้อย 5-10 เวอร์ชันล่าสุดไว้เผื่อต้อง rollback
2. **แจ้งเตือนทีมทันทีเมื่อมีการ rollback เกิดขึ้น** — ควรเชื่อม pipeline เข้ากับ Slack/Discord/Email เพื่อให้ทุกคนรู้ทันที (ตามที่เคยเรียนเรื่อง notification ใน CI/CD)
3. **บันทึกเหตุการณ์ (post-mortem)** ทุกครั้งหลัง rollback ว่าเกิดอะไรขึ้นและจะป้องกันอย่างไรในอนาคต ไม่ใช่แค่ rollback แล้วจบ

---

## Step 738: Health Checks และ Smoke Tests หลัง Deploy

การ deploy job รันจนจบโดยไม่มี error ใน log **ไม่ได้แปลว่าแอปพลิเคชันทำงานถูกต้องจริง** อาจเป็นไปได้ว่า container เริ่มทำงานสำเร็จ แต่ตัวแอปพลิเคชันข้างในเชื่อมต่อ database ไม่ได้ หรือ endpoint สำคัญ return error 500 ทุกครั้ง วิธีเดียวที่จะรู้แน่ชัดคือ **ทดสอบจริงหลัง deploy เสร็จ**

### Health Check คืออะไร

> **Health Check คือ endpoint พิเศษในแอปพลิเคชันที่ตอบกลับสถานะว่าระบบพร้อมทำงานหรือไม่ โดยไม่ต้องมี logic ทางธุรกิจซับซ้อน**

แอปพลิเคชันส่วนใหญ่มักเปิด endpoint ชื่อ `/health` หรือ `/healthz` ที่ตอบกลับง่าย ๆ:

```
GET /health

HTTP/1.1 200 OK
Content-Type: application/json

{"status": "ok", "database": "connected", "uptime_seconds": 3600}
```

### Liveness Probe และ Readiness Probe ใน Kubernetes

Kubernetes มีกลไก health check ในตัวสองแบบที่ทำงานต่างกัน:

| ประเภท | ตรวจสอบอะไร | ถ้าล้มเหลว Kubernetes จะทำอะไร |
|---|---|---|
| **Liveness Probe** | container ยัง "มีชีวิต" อยู่ไหม (ไม่ค้าง/ไม่ deadlock) | **restart container** นั้นทันที |
| **Readiness Probe** | container พร้อม **รับ traffic จริง** หรือยัง | **ถอด Pod ออกจาก Service ชั่วคราว** (ไม่ restart) จนกว่าจะพร้อม |

ตัวอย่างการกำหนดใน Deployment manifest:

```yaml
containers:
  - name: backend-api
    image: registry.example.com/backend-api:abc1234
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 15
    readinessProbe:
      httpGet:
        path: /health/ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
```

Readiness Probe คือกลไกสำคัญที่ทำให้ **Rolling Update ปลอดภัย** เพราะ Kubernetes จะไม่ส่ง traffic ไปยัง Pod ใหม่จนกว่า Readiness Probe จะผ่านก่อน — นี่คือสิ่งที่ทำให้ Rolling Update ไม่มี downtime อย่างแท้จริง ไม่ใช่แค่ทฤษฎีเฉย ๆ

### Smoke Test คืออะไร ต่างจาก Health Check อย่างไร

> **Smoke Test คือชุดทดสอบเล็ก ๆ ที่ยิง request จริงไปยังระบบที่เพิ่ง deploy เสร็จ เพื่อตรวจสอบว่า flow สำคัญที่สุดของระบบยังทำงานถูกต้อง**

ต่างจาก Health Check ตรงที่ Health Check แค่บอกว่า "container รันอยู่" แต่ Smoke Test ตรวจสอบว่า **ฟังก์ชันสำคัญจริง ๆ ใช้งานได้** เช่น login ได้จริง, ดึงข้อมูลสินค้าได้จริง, สร้าง order ได้จริง

### เพิ่ม Smoke Test เป็น Job ใน Pipeline

```yaml
smoke-test-production:
  stage: verify
  image: curlimages/curl:8.7.1
  script:
    - |
      echo "รอให้ service พร้อมก่อนทดสอบ..."
      sleep 10
    - |
      STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://app.example.com/health)
      if [ "$STATUS" != "200" ]; then
        echo "Health check ล้มเหลว: ได้ HTTP status $STATUS"
        exit 1
      fi
      echo "Health check ผ่าน (HTTP $STATUS)"
    - |
      RESPONSE=$(curl -s https://app.example.com/api/products?limit=1)
      echo "$RESPONSE" | grep -q '"id"' || (echo "Smoke test ล้มเหลว: response ไม่มีข้อมูลสินค้า" && exit 1)
      echo "Smoke test: ดึงข้อมูลสินค้าสำเร็จ"
  needs:
    - deploy-k8s
  environment:
    name: production
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

จุดสำคัญ:

- **Job นี้รันหลัง job deploy เสมอ** (ผ่าน `needs`) เพื่อทดสอบสิ่งที่เพิ่ง deploy จริง ไม่ใช่ทดสอบระบบเก่า
- **ถ้า smoke test ล้มเหลว job จะ fail** ทำให้ทีมรู้ทันทีว่า deploy มีปัญหา แม้ job deploy เองจะรันผ่านก็ตาม
- ควรพิจารณาผูก job นี้เข้ากับ **rollback job อัตโนมัติ** ในทีมที่ต้องการความปลอดภัยสูงสุด (เช่น ถ้า smoke test ล้มเหลว ให้ trigger job rollback ต่อทันทีโดยไม่ต้องรอคนกด)

### แนวทางออกแบบ Smoke Test ที่ดี

1. **ทดสอบเฉพาะ critical path เท่านั้น** — ไม่ใช่การทดสอบครบทุก feature (นั่นคือหน้าที่ของ automated test suite ที่รันตอน build ไม่ใช่ตอน deploy)
2. **ให้รันเร็ว** — ควรเสร็จภายในไม่กี่วินาทีถึงไม่กี่สิบวินาที เพื่อไม่ให้ pipeline ช้าเกินไป
3. **ไม่ทำ side effect ที่แก้ไขข้อมูลจริงถาวร** — ถ้าต้องทดสอบการสร้างข้อมูล ควรใช้ endpoint/ข้อมูลทดสอบเฉพาะที่ลบทิ้งได้ทันที

---

## Step 739: Zero-downtime Deployment — แนวคิดที่ผู้ใช้ไม่รู้สึกถึง Downtime

เมื่อรวมทุกอย่างที่เรียนมาใน Part นี้เข้าด้วยกัน — Rolling Update, Readiness Probe, Health Check — เราจะได้สิ่งที่เรียกว่า **Zero-downtime Deployment** ซึ่งเป็นเป้าหมายสูงสุดของทีมวิศวกรรมสมัยใหม่แทบทุกที่

> **Zero-downtime Deployment คือการ deploy เวอร์ชันใหม่ของแอปพลิเคชันโดยที่ผู้ใช้จริงไม่รู้สึกถึงการหยุดชะงักของบริการเลยแม้แต่วินาทีเดียว**

### กลไกที่ทำให้ Zero-downtime เป็นไปได้จริง

```
                     Load Balancer / Service
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │  Pod เก่า │   │  Pod เก่า │   │  Pod ใหม่ │
        │  (ready) │   │  (ready) │   │(กำลัง start)│
        └──────────┘   └──────────┘   └──────────┘
                                              │
                                    Readiness Probe
                                      ยังไม่ผ่าน
                                              │
                                    Kubernetes ไม่ส่ง
                                    traffic มาที่ Pod นี้
                                    จนกว่าจะ ready จริง
```

ขั้นตอนที่เกิดขึ้นเบื้องหลังเมื่อ Kubernetes ทำ Rolling Update:

1. **สร้าง Pod ใหม่ขึ้นมาก่อน** (ไม่ได้ปิด Pod เก่าทันที)
2. **รอ Readiness Probe ของ Pod ใหม่ผ่าน** — ถ้าแอปพลิเคชันต้องใช้เวลาโหลด configuration หรือเชื่อมต่อ database ก่อนพร้อมทำงาน Kubernetes จะรอจนกว่าจะพร้อมจริง
3. **เมื่อ Pod ใหม่ ready แล้ว ค่อยเริ่มส่ง traffic เข้าไป** พร้อมกับเริ่มทยอยปิด Pod เก่าทีละตัว
4. **ก่อนปิด Pod เก่า จะมีช่วง "Graceful Shutdown"** — Kubernetes ส่งสัญญาณ `SIGTERM` ไปยัง Pod เก่าก่อน ให้เวลาแอปพลิเคชัน **ทำงานที่ค้างอยู่ให้เสร็จ (connection draining)** เช่น ตอบ request ที่กำลังประมวลผลอยู่ให้เสร็จก่อน แล้วค่อยปิดตัวจริง (ถ้าไม่เสร็จภายใน `terminationGracePeriodSeconds` ที่กำหนด Kubernetes จะบังคับปิดด้วย `SIGKILL`)

### สิ่งที่แอปพลิเคชันต้องรองรับเพื่อให้ Zero-downtime ทำงานได้จริง

Zero-downtime ไม่ใช่แค่การตั้งค่า Kubernetes ให้ถูกต้อง แต่ **ตัวแอปพลิเคชันเองก็ต้องถูกออกแบบให้รองรับด้วย**:

1. **ต้องมี Readiness endpoint ที่สะท้อนสถานะจริง** — ถ้า readiness endpoint ตอบ `200 OK` ทันทีที่ process เริ่มทำงาน (ทั้งที่ยังเชื่อมต่อ database ไม่เสร็จ) จะทำให้ traffic ถูกส่งไปยัง Pod ที่ยังไม่พร้อมจริง เกิด error กับผู้ใช้
2. **ต้องรองรับ Graceful Shutdown** — แอปพลิเคชันต้องดักจับสัญญาณ `SIGTERM` แล้วหยุดรับ request ใหม่ แต่ยังทำ request เก่าที่ค้างอยู่ให้เสร็จก่อนปิดตัว
3. **ต้อง Backward Compatible ระหว่างเวอร์ชัน** — เพราะระหว่าง Rolling Update จะมีทั้งเวอร์ชันเก่าและใหม่รันพร้อมกันเสมอ (ตามที่อธิบายไว้ใน Step 731) ถ้าเวอร์ชันใหม่เปลี่ยนโครงสร้าง API หรือ database schema แบบที่เวอร์ชันเก่าใช้ไม่ได้ จะทำให้ผู้ใช้บางคนเจอ error ระหว่างช่วงเปลี่ยนผ่านทันที
4. **Database Migration ต้องออกแบบให้ปลอดภัยกับทั้งสองเวอร์ชัน** — เช่น การเพิ่มคอลัมน์ใหม่ควรทำแบบ **expand-contract pattern** (เพิ่มคอลัมน์ใหม่แบบ nullable ก่อน → deploy โค้ดที่ใช้ทั้งคอลัมน์เก่าและใหม่ → ค่อย migrate ข้อมูล → สุดท้ายค่อยลบคอลัมน์เก่าในภายหลัง) แทนที่จะเปลี่ยนโครงสร้างตารางแบบทำลายของเก่าทันทีในการ deploy ครั้งเดียว

### สรุปห่วงโซ่ที่ทำให้ Zero-downtime เกิดขึ้นจริง

```
Rolling Update Strategy
        +
Readiness Probe ที่แม่นยำ
        +
Graceful Shutdown ในแอปพลิเคชัน
        +
Backward-compatible API/Schema
        │
        ▼
  Zero-downtime Deployment
```

ทุกองค์ประกอบต้องทำงานร่วมกัน ขาดอันใดอันหนึ่งไปก็ไม่สามารถการันตี zero-downtime ได้อย่างแท้จริง แม้จะตั้งค่า `strategy: RollingUpdate` ไว้ถูกต้องแล้วก็ตาม

---

## Step 740: แบบฝึกหัด — Deploy อัตโนมัติไปยัง Platform ฟรีเมื่อ Merge เข้า Main

ถึงเวลาลงมือทำจริง! แบบฝึกหัดนี้จะให้คุณสร้าง pipeline job ที่ **deploy โปรเจกต์ไปยัง platform ฟรีสำหรับฝึกฝนโดยอัตโนมัติทุกครั้งที่ merge เข้า branch `main`** โดยไม่ต้องมี server หรือ Kubernetes cluster ของตัวเองเลย

### เลือก Platform ที่จะใช้ฝึก

Platform ฟรีที่นิยมใช้ฝึกฝนมี 3 เจ้าหลัก:

| Platform | เหมาะกับ | วิธี Deploy จาก CI/CD |
|---|---|---|
| **Render** | Web service, API backend | Deploy Hook (URL เดียว ยิง POST เพื่อสั่ง deploy) |
| **Railway** | Web service, database ขนาดเล็ก | CLI tool (`railway up`) ยืนยันตัวตนด้วย Token |
| **Netlify** | เว็บไซต์ static, frontend framework | CLI tool (`netlify deploy`) ยืนยันตัวตนด้วย Token |

เลือกใช้ตัวใดตัวหนึ่งตามลักษณะโปรเจกต์ของคุณ (backend เลือก Render/Railway, frontend เลือก Netlify)

### แบบที่ 1: Deploy ผ่าน Deploy Hook (แนวทางของ Render)

Platform บางเจ้าให้ **Deploy Hook URL** มาเลย ซึ่งเป็นวิธีที่ง่ายที่สุด เพียงยิง HTTP request ไปยัง URL นั้น platform ก็จะเริ่ม deploy ให้เอง

1. สร้าง Web Service บน platform แล้วเชื่อมกับ repository ของคุณ
2. คัดลอก **Deploy Hook URL** ที่ platform สร้างให้ (รูปแบบทั่วไปจะคล้าย `https://api.<platform>.com/deploy/<service-id>?key=<deploy-key>`)
3. นำ URL ทั้งหมด (รวม key) ไปเก็บเป็น CI/CD Variable ชื่อ `DEPLOY_HOOK_URL` แบบ **Masked** เสมอ เพราะ URL นี้เทียบเท่ากับ credential

```yaml
deploy-render:
  stage: deploy
  image: curlimages/curl:8.7.1
  script:
    - curl -X POST "$DEPLOY_HOOK_URL"
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

### แบบที่ 2: Deploy ผ่าน CLI Tool พร้อม Token (แนวทางของ Railway/Netlify)

Platform ที่มี CLI tool ของตัวเองมักให้ความยืดหยุ่นมากกว่า เช่น เลือก environment, ดู log การ deploy แบบ real-time ได้

ตัวอย่างสำหรับ Netlify (deploy เว็บไซต์ static/frontend):

```yaml
deploy-netlify:
  stage: deploy
  image: node:20
  script:
    - npm install -g netlify-cli
    - npm run build
    - netlify deploy
        --dir=dist
        --prod
        --auth="$NETLIFY_AUTH_TOKEN"
        --site="$NETLIFY_SITE_ID"
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

ตัวอย่างสำหรับ Railway:

```yaml
deploy-railway:
  stage: deploy
  image: node:20
  script:
    - npm install -g @railway/cli
    - railway up --service backend-api --token "$RAILWAY_TOKEN"
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

> ค่า `NETLIFY_AUTH_TOKEN`, `NETLIFY_SITE_ID`, `RAILWAY_TOKEN` ในตัวอย่างข้างต้นล้วนเป็น **placeholder** ที่ต้องแทนที่ด้วยค่าจริงจากบัญชีของคุณเอง โดยนำไปตั้งเป็น CI/CD Variable แบบ Protected และ Masked เสมอ — **ห้าม hardcode ค่าจริงลงในไฟล์ `.gitlab-ci.yml` หรือ `.github/workflows/*.yml` เด็ดขาด**

### ขั้นตอนทำแบบฝึกหัดแบบเต็ม

1. **สมัครบัญชีฟรี** กับ platform ที่เลือก (Render, Railway หรือ Netlify)
2. **สร้าง service ใหม่** แล้วเชื่อมกับ repository บน GitHub/GitLab ของคุณ (บาง platform เชื่อม repository แล้ว deploy อัตโนมัติให้เองโดยไม่ต้องเขียน pipeline เพิ่มเลยด้วยซ้ำ — แต่แบบฝึกหัดนี้ต้องการให้คุณ **ควบคุมการ deploy ผ่าน pipeline ของคุณเอง** เพื่อฝึกทักษะที่เรียนมาทั้ง Part)
3. **สร้าง Token/Deploy Hook** ตามวิธีของ platform นั้น
4. **นำค่าไปตั้งเป็น CI/CD Variable** ในโปรเจกต์ของคุณ
5. **เพิ่ม deploy job** ต่อท้าย pipeline ที่มีอยู่แล้ว (ต่อจาก stage test/build เดิมที่เคยทำใน Part ก่อนหน้า) โดยกำหนดเงื่อนไขให้รันเฉพาะเมื่อ merge เข้า `main` เท่านั้น
6. **เพิ่ม job สำหรับ smoke test** ต่อท้าย deploy job ตามที่เรียนใน Step 738 เพื่อยิง request ไปตรวจสอบว่า service ที่ deploy ไปตอบสนองปกติจริง เช่น:

```yaml
smoke-test-free-platform:
  stage: verify
  image: curlimages/curl:8.7.1
  script:
    - sleep 15
    - |
      STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$APP_PUBLIC_URL")
      if [ "$STATUS" != "200" ]; then
        echo "Smoke test ล้มเหลว: ได้ HTTP status $STATUS จาก $APP_PUBLIC_URL"
        exit 1
      fi
      echo "Smoke test ผ่าน: service ตอบสนองปกติที่ $APP_PUBLIC_URL"
  needs:
    - deploy-netlify
  rules:
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

7. **ทดสอบทั้ง flow** — สร้าง merge request/pull request เล็ก ๆ แก้ไขอะไรสักอย่างที่มองเห็นได้ชัด (เช่น เปลี่ยนข้อความในหน้าแรก) แล้ว merge เข้า `main` ดู pipeline รันทั้งชุด: test → build → deploy → smoke test โดยอัตโนมัติ
8. **เปิด URL จริงของแอป** ที่ platform สร้างให้ ตรวจสอบว่าการเปลี่ยนแปลงที่ทำไปแสดงผลจริงบนเว็บ

### เกณฑ์ความสำเร็จของแบบฝึกหัด

- [ ] Push/Merge เข้า `main` แล้ว pipeline เริ่มทำงานเองโดยไม่ต้องกดปุ่มใด ๆ
- [ ] Deploy job สำเร็จและแอปพลิเคชันเวอร์ชันใหม่ขึ้นให้เห็นจริงบน URL สาธารณะ
- [ ] Smoke test job รันหลัง deploy เสร็จเสมอ และรายงานผลถูกต้องทั้งกรณีสำเร็จและล้มเหลว
- [ ] Credential ทั้งหมด (token, deploy hook) ถูกเก็บเป็น CI/CD Variable แบบ Masked ไม่มีค่าจริงหลงเหลือในไฟล์ pipeline เลย
- [ ] Deploy job รันเฉพาะเมื่อเข้า `main` เท่านั้น ไม่รันตอน push เข้า feature branch

---

## สรุป Part 74

ใน Part นี้เราได้เรียนรู้ว่า:

1. **Deployment Strategy มี 4 แบบหลัก** — Recreate (ง่ายแต่มี downtime), Rolling Update (มาตรฐานทั่วไป ไม่มี downtime), Blue-Green (สลับเร็ว rollback เร็วที่สุดแต่ต้นทุนสูง), Canary (ความเสี่ยงต่ำสุดแต่ซับซ้อนที่สุด)
2. **Registry คือสะพานเชื่อมระหว่าง build กับ deploy** — ต่อยอดจาก GitLab Container Registry ใน Part 52 ด้วยการแยก stage build/push/deploy ให้ชัดเจน และใช้ commit SHA เป็น tag หลักเพื่อ traceability
3. **`kubectl apply`/`kubectl set image` จาก pipeline** คือวิธีสั่งงาน Kubernetes อัตโนมัติ โดย kubeconfig ต้องเก็บเป็น CI/CD Variable แบบ base64 พร้อม Protected และ Masked เสมอ
4. **แต่ละ Cloud Provider (AWS/GCP/Azure) มี CLI tool และบริการของตัวเอง** แต่หลักการเชื่อมเข้ากับ CI/CD เหมือนกันหมด: ติดตั้ง CLI → ยืนยันตัวตนด้วย credential ที่เก็บปลอดภัย → สั่ง deploy → ตรวจสอบผล
5. **Infrastructure as Code** คือแนวคิดที่นิยาม infrastructure เป็นไฟล์ข้อความเก็บใน Git แทนการสร้างด้วยมือ ซึ่งเราจะเจาะลึกเต็มรูปแบบใน Part 77
6. **Environment-specific Configuration** ทำได้ด้วยการใช้ image เดียวกันทุก environment แล้วแยกค่าผ่าน CI/CD Variable ที่มี Environment Scope ตามหลักการ Twelve-Factor App
7. **Rollback ต้องเร็วกว่าการ debug เสมอ** — ทำได้ทั้งการ deploy กลับ image tag เดิม หรือใช้ `kubectl rollout undo` ที่ Kubernetes จำ revision history ให้อัตโนมัติ
8. **Health Check และ Smoke Test** คือด่านสุดท้ายที่ยืนยันว่า deploy สำเร็จจริง ไม่ใช่แค่ job ไม่ error — แยกจาก Liveness/Readiness Probe ที่ Kubernetes ใช้ควบคุม traffic ระหว่าง Rolling Update
9. **Zero-downtime Deployment** เกิดจากการทำงานร่วมกันของ Rolling Update, Readiness Probe ที่แม่นยำ, Graceful Shutdown ในแอปพลิเคชัน และการออกแบบให้ backward compatible ระหว่างเวอร์ชัน
10. เราได้ลงมือสร้าง pipeline ที่ deploy จริงไปยัง platform ฟรีอัตโนมัติเมื่อ merge เข้า `main` พร้อม smoke test ต่อท้าย ครบทุกขั้นตอนตั้งแต่ commit จนถึงแอปพลิเคชันขึ้นจริงบนโลกอินเทอร์เน็ต

จาก Part นี้ คุณมีความสามารถ deploy อัตโนมัติได้ครบทุกระดับ ตั้งแต่ container เดี่ยว ไปจนถึง Kubernetes และ Cloud Provider ระดับองค์กร ขั้นตอนถัดไปคือการนำทุกอย่างที่เรียนมาตลอดหลักสูตรนี้มาประกอบร่างเป็นโปรเจกต์ฝึกหัดขนาดใหญ่ชิ้นเดียวที่ครบวงจรตั้งแต่ต้นจนจบ

**ต่อไป:** [Part 75: โปรเจกต์ฝึกหัด: สร้าง CI/CD Pipeline ครบวงจร](./part-075-cicd-pipeline-ครบวงจร.md)
