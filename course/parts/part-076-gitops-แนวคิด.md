# Part 76: GitOps: แนวคิดและการประยุกต์ใช้งานจริง

> **Step ในหลักสูตรนี้:** Step 751–760
> **เฟส:** 8 — DevOps, Security, Compliance ระดับองค์กร
> **เป้าหมายของ Part นี้:** เข้าใจแนวคิด GitOps อย่างถ่องแท้ ตั้งแต่นิยาม หลักการ 4 ข้อ ความแตกต่างระหว่าง push-based กับ pull-based deployment ไปจนถึงเครื่องมือยอดนิยมอย่าง ArgoCD และ Flux พร้อมเห็นภาพ workflow จริงและข้อจำกัดที่ต้องรู้ก่อนนำไปใช้งานในองค์กร

---

## เกริ่นก่อนเข้าเฟส 8: จากการส่งมอบโค้ด สู่การดูแลทั้งระบบ

ตลอดเฟส 7 (Step 651–750) เราใช้เวลาอยู่กับ **CI/CD** อย่างเข้มข้น — เราเรียนรู้วิธีสร้าง pipeline ด้วย GitHub Actions และ GitLab CI/CD เพื่อ build, test และ deploy โค้ดโดยอัตโนมัติ เป้าหมายของเฟสนั้นคือการทำให้ **"โค้ดที่เขียนเสร็จ" เดินทางไปถึง "โค้ดที่รันอยู่บน production" ได้เร็ว ปลอดภัย และซ้ำได้ทุกครั้ง**

แต่ถ้าสังเกตให้ดี pipeline แบบ CI/CD ที่เราเรียนมาทั้งหมดนั้นเป็น pipeline ที่ **"push" การเปลี่ยนแปลงออกไปหาระบบปลายทาง** เช่น เมื่อ merge เข้า `main` แล้ว runner ของ CI/CD จะรัน `kubectl apply`, `terraform apply`, `docker push` หรือ script deploy ไปยัง server/cluster โดยตรง ซึ่งวิธีนี้ใช้งานได้ดีมาโดยตลอด แต่เมื่อระบบขยายใหญ่ขึ้น มีหลาย cluster หลาย environment (dev, staging, production) และมีทีมจำนวนมากที่ต้อง deploy พร้อมกัน คำถามใหม่ ๆ ก็เริ่มเกิดขึ้น:

- ใครมีสิทธิ์ยิงคำสั่ง `kubectl apply` เข้า production cluster ได้บ้าง และเรารู้ได้อย่างไรว่าใครทำอะไรไปแล้วบ้าง
- ถ้ามีคนแอบเข้าไปแก้ config ใน cluster ตรง ๆ (ไม่ผ่าน pipeline) เราจะรู้ได้อย่างไร และจะดึงกลับมาให้ตรงกับที่ควรจะเป็นได้อย่างไร
- ถ้าต้องการย้อนกลับ (rollback) การเปลี่ยนแปลงทั้ง infrastructure ทั้งหมด ไม่ใช่แค่โค้ดแอปพลิเคชัน เราจะทำอย่างไรให้เชื่อถือได้ 100%
- CI/CD credential ที่ใช้ยิงเข้า production cluster โดยตรงนั้น ถ้าหลุดออกไป ความเสียหายจะรุนแรงแค่ไหน

นี่คือจุดเริ่มต้นของ **เฟส 8: DevOps, Security, Compliance ระดับองค์กร (Step 751–850)** ซึ่งเป็นเฟสที่เราจะขยายขอบเขตความรับผิดชอบของ Git ออกไปไกลกว่าแค่ "การส่งมอบโค้ด" (delivery) ไปสู่ **"การดูแลทั้งระบบตลอดวงจรชีวิต"** (operations, security, compliance) — Git จะไม่ใช่แค่ที่เก็บซอร์สโค้ดอีกต่อไป แต่จะกลายเป็น **แหล่งความจริงเดียว (single source of truth)** ของทั้งโครงสร้างพื้นฐาน (infrastructure), นโยบายความปลอดภัย (security policy) และหลักฐานการตรวจสอบ (audit/compliance evidence) ขององค์กรทั้งหมด

Part แรกของเฟสนี้จะพาไปรู้จักแนวคิดที่เป็นรากฐานสำคัญที่สุดของการเปลี่ยนแปลงนี้ นั่นคือ **GitOps** — แนวคิดที่จะพลิกวิธีคิดเรื่อง deployment จาก "ระบบ CI ส่งของเข้าไปหา" มาเป็น "ระบบปลายทางดึงของออกมาเอง โดยมี Git เป็นความจริงหนึ่งเดียวที่ทุกฝ่ายเชื่อถือได้"

---

## สารบัญของ Part นี้

- Step 751: GitOps คืออะไร ใครบัญญัติศัพท์นี้
- Step 752: หลักการ 4 ข้อของ GitOps
- Step 753: Git เป็น "Single Source of Truth" สำหรับ Infrastructure ไม่ใช่แค่ Application Code
- Step 754: Push-based Deployment vs Pull-based Deployment ต่างกันอย่างไร
- Step 755: เครื่องมือ GitOps ยอดนิยม (ArgoCD, Flux) ภาพรวม
- Step 756: GitOps Workflow ตัวอย่าง — จากแก้ Manifest ถึง Cluster Sync อัตโนมัติ
- Step 757: ข้อดีของ GitOps (Audit Trail, Rollback, Self-Healing)
- Step 758: GitOps กับ Kubernetes — ความสัมพันธ์ที่แนบแน่นที่สุดในทางปฏิบัติ
- Step 759: ข้อจำกัดและความท้าทายของ GitOps
- Step 760: แบบฝึกหัด — ออกแบบ GitOps Workflow อย่างง่ายสำหรับ Deploy แอปเข้า Kubernetes Cluster จำลอง

---

## Step 751: GitOps คืออะไร ใครบัญญัติศัพท์นี้

### นิยามของ GitOps

**GitOps** คือแนวปฏิบัติ (practice) ในการบริหารจัดการ infrastructure และการ deploy แอปพลิเคชัน โดยใช้ **Git repository เป็นแหล่งความจริงเดียว (single source of truth)** สำหรับสถานะที่ "ควรจะเป็น" (desired state) ของระบบทั้งหมด แล้วอาศัย **agent อัตโนมัติ** ที่คอยเปรียบเทียบสถานะจริงของระบบ (actual state) กับสถานะที่ประกาศไว้ใน Git (desired state) และปรับให้ตรงกันอยู่ตลอดเวลา

พูดให้เป็นประโยคเดียวที่กระชับที่สุด:

> **GitOps = ใช้ Git เป็นจุดศูนย์กลางของการตัดสินใจว่าระบบ "ควรจะมีหน้าตาเป็นอย่างไร" แล้วให้ซอฟต์แวร์อัตโนมัติเป็นคนทำให้ระบบจริงตรงกับสิ่งที่ Git บอกไว้เสมอ**

จุดสำคัญที่ทำให้ GitOps ต่างจาก "การใช้ Git เก็บไฟล์ config" ธรรมดา ๆ คือ **กลไกการ reconcile (ปรับให้ตรงกัน) แบบต่อเนื่องและอัตโนมัติ** ไม่ใช่แค่การเก็บไฟล์ YAML ไว้ใน repo แล้วรัน `kubectl apply` มือเป็นครั้งคราว

### ใครบัญญัติศัพท์นี้

คำว่า **"GitOps"** ถูกบัญญัติขึ้นในปี **2017** โดยบริษัท **Weaveworks** (บริษัทที่พัฒนาเครื่องมือด้าน cloud-native networking และ observability สำหรับ Kubernetes) โดยเฉพาะในบทความของ **Alexis Richardson** ซึ่งขณะนั้นดำรงตำแหน่ง CEO ของ Weaveworks

แนวคิดนี้เกิดขึ้นจากประสบการณ์จริงของทีมวิศวกรที่ Weaveworks เอง ซึ่งค้นพบว่าการดำเนินงาน Kubernetes cluster ของบริษัทตัวเองได้อย่างมีประสิทธิภาพและปลอดภัยที่สุด คือการทำให้ **ทุกการเปลี่ยนแปลงต่อ cluster ต้องผ่าน Pull Request บน Git ก่อนเสมอ** แล้วให้ tooling อัตโนมัติเป็นคนนำการเปลี่ยนแปลงนั้นไปใช้กับ cluster จริง แทนที่จะให้วิศวกรรัน `kubectl` คำสั่งต่าง ๆ ใส่ cluster ตรง ๆ

Weaveworks ยังเป็นผู้พัฒนาเครื่องมือ GitOps ตัวแรก ๆ ที่ชื่อ **Flux** (ซึ่งต่อมาบริจาคให้ Cloud Native Computing Foundation หรือ CNCF) เพื่อเป็นตัวอย่างอ้างอิงของแนวคิดนี้

### ทำไม GitOps ถึงเกิดขึ้นในช่วงเวลานั้น

ปี 2017 คือช่วงเวลาที่ **Kubernetes** เริ่มเป็นมาตรฐานอุตสาหกรรมสำหรับการรัน container ในระดับ production และปัญหาที่ทีม DevOps ทั่วโลกเจอร่วมกันคือ

1. Kubernetes manifest (YAML) มีจำนวนมากและซับซ้อนขึ้นเรื่อย ๆ ตามจำนวน microservice
2. การจัดการ cluster หลายสภาพแวดล้อม (dev/staging/prod) ด้วยมือทำให้เกิดความไม่สอดคล้องกัน (configuration drift)
3. ทีม Ops ต้องการวิธีที่ปลอดภัยกว่าการแจก credential เข้า cluster ให้ CI/CD pipeline ภายนอกโดยตรง
4. อุตสาหกรรมต้องการ audit trail ที่ชัดเจนว่า "ใครเปลี่ยนอะไร เมื่อไหร่ ทำไม" สำหรับระบบ production

GitOps จึงเกิดขึ้นมาเป็นคำตอบของปัญหาเหล่านี้ โดยอาศัยจุดแข็งที่ Git มีอยู่แล้ว (ประวัติที่ตรวจสอบได้, การ review ผ่าน Pull Request, ความสามารถในการ revert) มาใช้กับการบริหาร infrastructure โดยตรง ไม่ใช่แค่กับ application code เหมือนที่ผ่านมา

---

## Step 752: หลักการ 4 ข้อของ GitOps

องค์กร **OpenGitOps** (โครงการภายใต้ CNCF ที่ตั้งขึ้นเพื่อกำหนดมาตรฐานกลางของ GitOps อย่างเป็นทางการ โดยไม่ผูกกับเครื่องมือใดเครื่องมือหนึ่ง) ได้กำหนดหลักการหลัก 4 ข้อที่ระบบหนึ่ง ๆ ต้องมีครบจึงจะเรียกว่าเป็น GitOps อย่างแท้จริง ดังนี้

### หลักการที่ 1: Declarative (ประกาศสถานะที่ต้องการ ไม่ใช่ขั้นตอนการทำ)

ระบบทั้งหมดต้องถูกอธิบายในรูปแบบ **declarative configuration** คือบอกว่า **"ระบบควรมีหน้าตาเป็นอย่างไร" (สถานะปลายทาง)** ไม่ใช่ **"ต้องทำขั้นตอนอะไรบ้างเพื่อไปถึงจุดนั้น" (imperative script)**

ตัวอย่างเปรียบเทียบ:

```yaml
# แบบ Declarative (ถูกต้องตามหลัก GitOps)
# บอกว่า "ต้องการ Pod ของ nginx จำนวน 3 ตัว" — ไม่สนใจว่าตอนนี้มีกี่ตัว
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25.3
```

```bash
# แบบ Imperative (ไม่ใช่ GitOps)
# สั่งเป็นขั้นตอน "ให้ scale จากที่เป็นอยู่เพิ่มอีก 1 ตัว"
kubectl scale deployment nginx-app --replicas=+1
```

คำสั่งแบบ imperative นั้นพึ่งพา "สถานะปัจจุบัน" เป็นจุดอ้างอิง ซึ่งถ้าไม่รู้ว่าสถานะปัจจุบันคืออะไรแน่ ๆ ผลลัพธ์ก็จะไม่แน่นอน ในขณะที่ declarative จะให้ผลลัพธ์เดียวกันเสมอไม่ว่าจะรันกี่ครั้งก็ตาม (คุณสมบัตินี้เรียกว่า **idempotency**)

### หลักการที่ 2: Versioned and Immutable (มีเวอร์ชันและไม่ถูกแก้ไขย้อนหลัง)

การประกาศสถานะ (declarative state) ต้องถูกเก็บไว้ในระบบที่ **บันทึกเวอร์ชันไว้อย่างครบถ้วนและไม่สามารถแก้ไขประวัติย้อนหลังได้โดยไม่มีร่องรอย** — และระบบที่ตอบโจทย์นี้ได้ดีที่สุดในทางปฏิบัติก็คือ **Git** นั่นเอง

คุณสมบัติที่ Git มอบให้ตรงนี้พอดี:

- ทุก commit มี hash ที่ไม่ซ้ำกัน ตรวจสอบความสมบูรณ์ของข้อมูลได้
- ประวัติทั้งหมดถูกเก็บไว้ ย้อนดูได้ทุกจุดในอดีต
- การเปลี่ยนแปลงทุกครั้งมีผู้เขียน (author), เวลา และข้อความอธิบาย (commit message) กำกับ

### หลักการที่ 3: Pulled Automatically (ถูกดึงเข้าไปโดยอัตโนมัติ)

นี่คือหัวใจที่ทำให้ GitOps แตกต่างจาก CI/CD แบบดั้งเดิมอย่างชัดเจนที่สุด — ซอฟต์แวร์ agent ที่ทำงานอยู่ **ภายในระบบเป้าหมายเอง** (เช่น อยู่ใน Kubernetes cluster) จะต้องเป็นฝ่าย **"ดึง" (pull)** การเปลี่ยนแปลงจาก Git repository เข้ามาเองโดยอัตโนมัติ แทนที่จะให้ระบบภายนอก (เช่น CI/CD pipeline) เป็นฝ่าย **"ส่ง" (push)** เข้าไปหา

รายละเอียดเรื่องนี้เราจะเจาะลึกกันใน Step 754

### หลักการที่ 4: Continuously Reconciled (ถูกปรับให้ตรงกันอย่างต่อเนื่อง)

Agent ต้องคอย **เปรียบเทียบสถานะจริงของระบบ (observed/actual state) กับสถานะที่ประกาศไว้ใน Git (desired state) อยู่ตลอดเวลาแบบ loop ไม่สิ้นสุด** และเมื่อพบว่าทั้งสองไม่ตรงกัน (เรียกว่าเกิด **drift**) agent จะต้องพยายามปรับสถานะจริงให้กลับมาตรงกับที่ Git ประกาศไว้โดยอัตโนมัติ

กระบวนการนี้เรียกว่า **reconciliation loop** ซึ่งมีลักษณะเป็นวงจรไม่สิ้นสุดประมาณนี้:

```
┌─────────────────────────────────────────────────┐
│                                                   │
│   ┌──────────────┐        ┌──────────────────┐  │
│   │  อ่านสถานะที่   │        │  อ่านสถานะจริง     │  │
│   │  ต้องการจาก Git │        │  ของระบบ (cluster) │  │
│   └───────┬──────┘        └─────────┬─────────┘  │
│           │                         │             │
│           └───────────┬─────────────┘             │
│                        ▼                           │
│              ┌───────────────────┐                │
│              │  เปรียบเทียบสองสถานะ  │                │
│              └─────────┬─────────┘                │
│                        │                           │
│              ตรงกันไหม? │                           │
│           ┌────────────┴────────────┐             │
│           │ ไม่ตรง (มี drift)         │ ตรงกันแล้ว      │
│           ▼                         ▼             │
│   ┌───────────────┐         ┌──────────────┐      │
│   │ ปรับสถานะจริงให้ │         │ ไม่ต้องทำอะไร  │      │
│   │ ตรงกับ Git       │         │ รอรอบถัดไป    │      │
│   └───────┬───────┘         └──────┬───────┘      │
│           │                        │              │
│           └────────────┬───────────┘              │
│                         │  (รอสักครู่แล้ววนใหม่)      │
│                         └──────────────────────────┘
```

ทั้ง 4 หลักการนี้ต้อง **ทำงานร่วมกันครบทั้งหมด** ระบบที่มีแค่บางข้อ (เช่น แค่เก็บ config ไว้ใน Git แต่ยังคง `kubectl apply` ด้วยมือหรือผ่าน CI pipeline แบบ push) ยังไม่นับว่าเป็น GitOps เต็มรูปแบบตามนิยามนี้ อย่างมากก็เรียกได้แค่ว่าเป็น "Infrastructure as Code ที่เก็บใน Git" เท่านั้น

---

## Step 753: Git เป็น "Single Source of Truth" สำหรับ Infrastructure ไม่ใช่แค่ Application Code

### ความหมายของ "Single Source of Truth"

**Single Source of Truth (SSOT)** หมายถึง **แหล่งข้อมูลเพียงหนึ่งเดียวที่ทุกฝ่ายยอมรับร่วมกันว่าเป็นความจริง** เมื่อเกิดคำถามว่า "ระบบตอนนี้ควรมีหน้าตาเป็นอย่างไร" คำตอบที่ถูกต้องจะต้องมาจากแหล่งเดียวนี้เท่านั้น ไม่ใช่มาจากการเดา ไม่ใช่มาจากการถามคนในทีม และไม่ใช่มาจากการ SSH เข้าไปดู server ตรง ๆ

ในโลกก่อน GitOps แหล่งความจริงของ infrastructure มักกระจัดกระจายอยู่หลายที่:

- config บางส่วนอยู่ใน script ที่วิศวกรรันด้วยมือ
- ค่าบางส่วนถูกตั้งผ่าน cloud console (web UI) โดยตรง ไม่มีบันทึกไว้ที่ไหน
- documentation ที่อธิบายว่า "ควรตั้งค่าอย่างไร" ล้าสมัยไปนานแล้ว
- สถานะจริงบน production ต่างจากที่ทุกคนเข้าใจ เพราะมีคนแก้ไขฉุกเฉินไปแล้วลืมอัปเดตที่อื่น

ปรากฏการณ์นี้เรียกว่า **configuration drift** และเป็นสาเหตุอันดับต้น ๆ ของ incident ในระบบขนาดใหญ่ทั่วโลก

### GitOps ทำให้ Git เป็น SSOT ของ Infrastructure ทั้งหมด

ก่อนหน้านี้ในหลักสูตร เราใช้ Git เก็บ **application source code** เป็นหลัก — ไฟล์ `.py`, `.js`, `.go` ฯลฯ ที่ประกอบกันเป็นโปรแกรม แนวคิด GitOps ขยายบทบาทของ Git ให้ครอบคลุมไปถึง:

| สิ่งที่เก็บใน Git ภายใต้แนวคิด GitOps | ตัวอย่างไฟล์ |
|---|---|
| Kubernetes manifest | `deployment.yaml`, `service.yaml`, `ingress.yaml` |
| Helm chart values | `values.yaml`, `Chart.yaml` |
| Infrastructure as Code | ไฟล์ Terraform `.tf` (จะเจาะลึกใน Part 77) |
| นโยบายความปลอดภัย (security policy) | `network-policy.yaml`, OPA/Gatekeeper policy |
| การตั้งค่า monitoring/alerting | Prometheus rule, Grafana dashboard JSON |
| จำนวน replica, resource limit, environment variable ของแต่ละ environment | `overlays/production/kustomization.yaml` |

เมื่อทุกอย่างเหล่านี้ถูกเก็บไว้ใน Git repository เดียวกัน (หรือชุด repository ที่มีโครงสร้างชัดเจน) ผลลัพธ์ที่ได้คือ:

1. **ต้องการรู้ว่า production ตอนนี้ควรมีหน้าตาอย่างไร?** — เปิด repository แล้วดู branch ที่กำหนดไว้สำหรับ production ก็จบ ไม่ต้อง SSH เข้า server
2. **ต้องการรู้ว่าเมื่อไหร่มีการเปลี่ยนแปลง network policy ครั้งล่าสุด ใครเป็นคนแก้ ทำไมถึงแก้?** — ดู `git log` และ commit message/Pull Request ที่เกี่ยวข้อง
3. **ต้องการเปรียบเทียบว่า staging กับ production ต่างกันตรงไหนบ้าง?** — ใช้ `git diff` ระหว่าง branch หรือ folder ของแต่ละ environment ได้ทันที
4. **ต้องการ audit เพื่อผ่านมาตรฐาน compliance (เช่น SOC 2, ISO 27001)?** — ประวัติ Git ทั้งหมดคือหลักฐานที่ตรวจสอบย้อนหลังได้อย่างสมบูรณ์ (เรื่องนี้จะเจาะลึกในเฟส 8 ต่อไปในหัวข้อ compliance)

### ข้อควรระวัง: SSOT ต้องมีจริงแค่แหล่งเดียว

หัวใจสำคัญของหลักการนี้คือ **ต้องไม่มีการแก้ไขระบบจริงโดยตรงแบบข้าม Git** เด็ดขาด (หรือถ้าจำเป็นต้องทำในกรณีฉุกเฉิน ต้องมีกระบวนการดึงการเปลี่ยนแปลงนั้นกลับเข้า Git ทันทีในภายหลัง) เพราะถ้ามีการแก้ cluster ตรง ๆ โดยไม่ผ่าน Git บ่อยครั้ง ความหมายของ "Single" ใน Single Source of Truth ก็จะพังทลายลงทันที ระบบจะกลับไปมีความจริงหลายเวอร์ชันเหมือนเดิม

---

## Step 754: Push-based Deployment vs Pull-based Deployment ต่างกันอย่างไร

นี่คือหัวใจทางเทคนิคที่สำคัญที่สุดของ GitOps และเป็นจุดที่แยก GitOps ออกจาก CI/CD pipeline แบบดั้งเดิมอย่างชัดเจน

### Push-based Deployment (แบบเดิมใน CI/CD)

ใน CI/CD pipeline แบบดั้งเดิมที่เราเรียนมาในเฟส 7 ขั้นตอนการ deploy มักมีลักษณะดังนี้:

```
┌──────────────┐    push     ┌─────────────────┐    push     ┌──────────────────┐
│  Developer   │───────────▶ │  CI/CD Runner    │───────────▶│  Target System    │
│  (git push)  │             │  (GitHub Actions/│  (kubectl   │  (Kubernetes      │
│              │             │   GitLab CI)     │   apply /   │   Cluster /       │
│              │             │                  │   terraform │   Production      │
│              │             │                  │   apply)    │   Server)         │
└──────────────┘             └─────────────────┘             └──────────────────┘
```

ลักษณะสำคัญของ push-based:

1. **CI/CD runner เป็นฝ่าย "ยื่น" การเปลี่ยนแปลงเข้าไปหาระบบเป้าหมายโดยตรง** ผ่านคำสั่งอย่าง `kubectl apply`, `ssh` เข้าไปรัน script, หรือเรียก cloud API
2. **CI/CD runner ต้องถือ credential ที่มีสิทธิ์เข้าถึงระบบเป้าหมายโดยตรง** เช่น kubeconfig ของ production cluster, SSH key ของ production server, หรือ cloud IAM role ที่มีสิทธิ์สูง
3. ระบบเป้าหมาย **ไม่รู้ตัวว่ากำลังถูกเปลี่ยนแปลง** จนกว่าคำสั่งจะมาถึง — มันเป็นเพียง "ผู้รับคำสั่ง" แบบ passive
4. ถ้ามีใครเข้าไปแก้ระบบเป้าหมายตรง ๆ โดยไม่ผ่าน CI/CD (เช่น SSH เข้าไปแก้ config ฉุกเฉิน) **จะไม่มีอะไรตรวจจับหรือแก้ไขความไม่ตรงกันนี้โดยอัตโนมัติเลย** จนกว่าจะมีการ deploy รอบถัดไป

### Pull-based Deployment (แบบ GitOps)

ใน GitOps ขั้นตอนจะกลับด้านกัน:

```
┌──────────────┐    push     ┌──────────────────┐
│  Developer   │───────────▶ │  Git Repository   │
│  (git push)  │             │  (Source of Truth)│
└──────────────┘             └─────────┬─────────┘
                                        │
                                        │ pull (agent ดึงเข้ามาเอง)
                                        │ ตรวจสอบทุก N วินาที/นาที
                                        ▼
                              ┌──────────────────┐
                              │  GitOps Agent      │
                              │  (ArgoCD / Flux)   │
                              │  ทำงานอยู่ภายใน     │
                              │  Target System     │
                              └─────────┬─────────┘
                                        │ apply การเปลี่ยนแปลง
                                        │ ภายในระบบตัวเอง
                                        ▼
                              ┌──────────────────┐
                              │  Target System     │
                              │  (Kubernetes       │
                              │   Cluster)         │
                              └──────────────────┘
```

ลักษณะสำคัญของ pull-based:

1. **GitOps agent ทำงานอยู่ภายในระบบเป้าหมายเอง** (เช่น รันเป็น Pod อยู่ใน Kubernetes cluster นั้น ๆ) ไม่ใช่รันอยู่ข้างนอกแบบ CI/CD runner
2. **Agent เป็นฝ่าย "เดินไปดู" Git repository เองเป็นระยะ ๆ** (polling) หรือรับการแจ้งเตือนเมื่อมีการเปลี่ยนแปลง (webhook) แล้วดึง (pull) การเปลี่ยนแปลงเข้ามา apply เอง
3. **ไม่มี credential ของระบบเป้าหมายรั่วไหลออกไปนอกระบบเลย** เพราะ CI/CD pipeline ภายนอกไม่จำเป็นต้องมีสิทธิ์เข้าถึง production cluster โดยตรงอีกต่อไป (มีแค่สิทธิ์ push โค้ด/manifest เข้า Git ซึ่งควบคุมผ่าน Pull Request ได้)
4. **Agent ทำงานแบบ reconciliation loop ต่อเนื่อง** — ถ้าใครแก้ cluster ตรง ๆ โดยไม่ผ่าน Git, agent จะตรวจพบความไม่ตรงกัน (drift) ในรอบถัดไปและ **ปรับกลับให้ตรงกับ Git โดยอัตโนมัติ** (นี่คือคุณสมบัติ self-healing ที่จะพูดถึงใน Step 757)

### ตารางเปรียบเทียบสรุป

| ประเด็น | Push-based (CI/CD ดั้งเดิม) | Pull-based (GitOps) |
|---|---|---|
| ใครเริ่มการ deploy | ระบบภายนอก (CI/CD runner) | Agent ภายในระบบเป้าหมายเอง |
| Credential ของระบบเป้าหมาย | ต้องมอบให้ CI/CD runner ภายนอก | ไม่ต้องออกจากระบบเป้าหมายเลย |
| ตรวจจับ configuration drift | ตรวจไม่ได้จนกว่าจะ deploy รอบถัดไป | ตรวจจับได้ต่อเนื่องแบบ real-time (ตาม loop interval) |
| การแก้ไขเมื่อมี drift | ต้องมีคนสั่ง deploy ใหม่ด้วยมือหรือรอ pipeline | Agent ปรับกลับให้อัตโนมัติ (self-healing) |
| ความเสี่ยงด้านความปลอดภัย | สูงกว่า (credential กระจายไปหลาย CI/CD job) | ต่ำกว่า (credential รวมศูนย์อยู่ในระบบเป้าหมาย) |
| ความซับซ้อนในการติดตั้งเริ่มต้น | ต่ำกว่า (คุ้นเคยกันมานาน) | สูงกว่าเล็กน้อย (ต้องติดตั้ง agent เพิ่ม) |

ข้อควรเข้าใจ: **push-based ไม่ได้แปลว่าแย่เสมอไป** สำหรับหลาย ๆ ระบบ (เช่น deploy static website, serverless function, หรือระบบขนาดเล็กที่ไม่ซับซ้อน) push-based ก็ยังเป็นตัวเลือกที่เหมาะสมและง่ายกว่า แต่สำหรับระบบที่ซับซ้อน มีหลาย environment และต้องการความน่าเชื่อถือ/ความปลอดภัยระดับสูง GitOps แบบ pull-based จะให้ประโยชน์ที่คุ้มค่ากว่ามาก

---

## Step 755: เครื่องมือ GitOps ยอดนิยม (ArgoCD, Flux) ภาพรวม

ปัจจุบันมีเครื่องมือ GitOps อยู่หลายตัว แต่ที่ได้รับความนิยมสูงสุดในระบบนิเวศ Kubernetes มีอยู่ 2 ตัวหลัก ทั้งคู่เป็นโครงการภายใต้ **Cloud Native Computing Foundation (CNCF)**

### ArgoCD

**ArgoCD** เป็นโครงการที่เริ่มพัฒนาโดยบริษัท **Intuit** และภายหลังบริจาคเข้าสู่ CNCF จนปัจจุบันเป็น **Graduated Project** (ระดับความเป็นผู้ใหญ่สูงสุดของ CNCF)

**คุณสมบัติเด่นของ ArgoCD:**

1. **มี Web UI ที่ใช้งานง่ายและสวยงามมาก** — สามารถเห็น dependency tree ของ resource ทั้งหมดใน application เป็นภาพกราฟ พร้อมสถานะ sync แบบ real-time
2. **ทำงานเป็น Kubernetes Custom Resource** — ประกาศ "Application" ที่ต้องการ sync ผ่าน Custom Resource Definition (CRD) ชื่อ `Application`
3. **รองรับหลายรูปแบบของ manifest** — plain YAML, Helm chart, Kustomize, Jsonnet
4. **มีปุ่ม Manual Sync และ Auto Sync ให้เลือก** — สามารถตั้งให้ sync อัตโนมัติทันทีที่ Git เปลี่ยน หรือจะให้รอ approve ด้วยมือก่อนก็ได้ (เหมาะกับ production ที่ต้องการ manual gate)
5. **รองรับ multi-cluster** — ArgoCD instance เดียวสามารถบริหารจัดการหลาย Kubernetes cluster พร้อมกันได้

ตัวอย่างไฟล์ประกาศ Application ของ ArgoCD แบบง่าย:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-web-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://git.example.internal/team/my-web-app-manifests.git
    targetRevision: main
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true      # ลบ resource ที่ไม่มีอยู่ใน Git ออกจาก cluster ด้วย
      selfHeal: true    # ถ้ามีคนแก้ cluster ตรง ๆ ให้ปรับกลับอัตโนมัติ
```

### Flux (Flux CD)

**Flux** เป็นเครื่องมือที่พัฒนาโดย **Weaveworks** เอง (บริษัทเดียวกับที่บัญญัติศัพท์ GitOps) และเป็นหนึ่งในเครื่องมือ GitOps ตัวแรกสุดของโลก ปัจจุบัน Flux เวอร์ชัน 2 (มักเรียกว่า **Flux v2** หรือ **Flux CD**) เป็นโครงการ CNCF เช่นกัน (ระดับ Graduated)

**คุณสมบัติเด่นของ Flux:**

1. **ออกแบบเป็นชุดของ controller ขนาดเล็กที่ทำงานร่วมกัน (GitOps Toolkit)** เช่น `source-controller` (ดึงข้อมูลจาก Git/Helm repo), `kustomize-controller` (apply Kustomize manifest), `helm-controller` (จัดการ Helm release)
2. **เน้นการทำงานแบบ CLI และ Kubernetes-native เป็นหลัก** ไม่มี Web UI ในตัวเหมือน ArgoCD โดยตรง (แต่สามารถใช้ Weave GitOps UI หรือเครื่องมือ dashboard อื่นเสริมได้)
3. **น้ำหนักเบาและยืดหยุ่นสูง** เหมาะกับทีมที่ต้องการปรับแต่ง pipeline การ sync แบบละเอียด
4. **รองรับ multi-tenancy ที่ดี** เหมาะกับองค์กรขนาดใหญ่ที่มีหลายทีมใช้ cluster เดียวกัน

ตัวอย่างไฟล์ประกาศแบบง่ายของ Flux (`GitRepository` + `Kustomization`):

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: my-web-app-source
  namespace: flux-system
spec:
  interval: 1m
  url: https://git.example.internal/team/my-web-app-manifests.git
  ref:
    branch: main
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-web-app
  namespace: flux-system
spec:
  interval: 5m
  sourceRef:
    kind: GitRepository
    name: my-web-app-source
  path: "./overlays/production"
  prune: true
  targetNamespace: production
```

### ตารางเปรียบเทียบโดยสรุป

| คุณสมบัติ | ArgoCD | Flux |
|---|---|---|
| ผู้พัฒนาเดิม | Intuit | Weaveworks |
| Web UI ในตัว | มี และโดดเด่นมาก | ไม่มีในตัว (ต้องใช้เครื่องมือเสริม) |
| สถานะ CNCF | Graduated | Graduated |
| รูปแบบ | Application เดียวจัดการหลาย resource | ชุด controller ย่อยทำงานร่วมกัน |
| เหมาะกับ | ทีมที่ต้องการเห็นภาพรวมผ่าน UI ชัดเจน | ทีมที่ต้องการความยืดหยุ่นระดับ controller |
| การเรียนรู้เบื้องต้น | ง่ายกว่าเพราะมี UI ช่วย | ต้องคุ้นเคยกับแนวคิด Kubernetes controller มากกว่า |

ทั้งสองเครื่องมือนี้สามารถทำสิ่งเดียวกันได้ในระดับหลักการ คือ implement หลักการ 4 ข้อของ GitOps ได้ครบถ้วน การเลือกใช้ตัวไหนขึ้นอยู่กับความถนัดของทีมและความต้องการด้าน UI/tooling เป็นหลัก ไม่มีตัวไหนที่ "ถูกกว่า" หรือ "ผิดกว่า" ในเชิงแนวคิด

---

## Step 756: GitOps Workflow ตัวอย่าง — จากแก้ Manifest ถึง Cluster Sync อัตโนมัติ

มาดูตัวอย่าง workflow แบบละเอียดทีละขั้นตอน เพื่อให้เห็นภาพว่า GitOps ทำงานจริงอย่างไรตั้งแต่ต้นจนจบ สมมติสถานการณ์: ทีมต้องการเปลี่ยนจำนวน replica ของแอปพลิเคชัน `payment-service` จาก 3 เป็น 5 บน production

### ขั้นตอนที่ 1: นักพัฒนาแก้ไข manifest บนเครื่องตัวเอง

นักพัฒนาสร้าง branch ใหม่และแก้ไขไฟล์ manifest ใน repository ที่เก็บ configuration (มักเรียกว่า **"config repo"** หรือ **"manifests repo"** ซึ่งอาจแยกจาก repo ของ source code แอปพลิเคชันโดยตรง — แนวคิดนี้เรียกว่า **repository separation** ซึ่งเป็นแนวปฏิบัติที่นิยมใน GitOps):

```bash
git checkout -b bump-payment-service-replicas
```

แก้ไขไฟล์ `overlays/production/payment-service/deployment.yaml`:

```diff
 apiVersion: apps/v1
 kind: Deployment
 metadata:
   name: payment-service
 spec:
-  replicas: 3
+  replicas: 5
```

### ขั้นตอนที่ 2: เปิด Pull Request เพื่อขอ review

```bash
git add overlays/production/payment-service/deployment.yaml
git commit -m "เพิ่ม replica ของ payment-service เป็น 5 ตัว เพื่อรองรับ load ช่วงโปรโมชัน"
git push origin bump-payment-service-replicas
```

จากนั้นเปิด Pull Request บน GitHub/GitLab เพื่อขอให้ทีม (เช่น SRE หรือ tech lead) ตรวจสอบก่อนว่า:

- ค่า replica ที่เพิ่มขึ้นเหมาะสมกับ resource ของ cluster หรือไม่
- มีผลกระทบกับ cost หรือไม่
- สอดคล้องกับนโยบายขององค์กรหรือไม่ (เช่น อาจมี policy check อัตโนมัติที่รันตรวจ manifest ผ่าน CI ก่อน merge ได้)

นี่คือจุดที่ **Pull Request กลายเป็น "ประตูอนุมัติ" (approval gate) สำหรับการเปลี่ยนแปลง production** แทนที่จะให้ใครก็ได้รัน `kubectl scale` ได้ทันทีโดยไม่มีการตรวจสอบ

### ขั้นตอนที่ 3: Merge เข้า branch หลัก

เมื่อได้รับการอนุมัติแล้ว Pull Request จะถูก merge เข้า branch `main` (หรือ branch ที่กำหนดไว้สำหรับ production เช่น `production`)

### ขั้นตอนที่ 4: GitOps agent ตรวจจับการเปลี่ยนแปลง

ภายใน cluster, agent (เช่น ArgoCD หรือ Flux) ที่ตั้งค่าให้เฝ้าดู repository นี้อยู่แล้ว จะตรวจพบว่ามี commit ใหม่เข้ามาใน branch ที่ตนเฝ้าดู วิธีตรวจจับมีสองแบบหลัก:

1. **Polling** — agent เข้าไปเช็ค repository เป็นระยะตามช่วงเวลาที่ตั้งไว้ (เช่น ทุก 1–5 นาที) ว่ามี commit ใหม่หรือไม่
2. **Webhook** — repository ส่ง webhook แจ้ง agent ทันทีที่มีการ push เข้า branch ที่เฝ้าดูอยู่ ทำให้ agent ตอบสนองได้เร็วเกือบทันที ไม่ต้องรอรอบ polling ถัดไป

### ขั้นตอนที่ 5: Agent เปรียบเทียบสถานะและ Sync

Agent จะดึง manifest ล่าสุดจาก Git มาเปรียบเทียบกับสถานะปัจจุบันของ resource ใน cluster:

```
สถานะที่ต้องการ (จาก Git):     replicas: 5
สถานะจริงใน cluster ตอนนี้:    replicas: 3
ผลการเปรียบเทียบ:              ไม่ตรงกัน (OutOfSync)
```

เมื่อพบว่า "OutOfSync" agent จะดำเนินการ apply การเปลี่ยนแปลงให้ cluster ตรงกับ Git (ถ้าตั้งค่า auto-sync ไว้) หรือรอให้มีคนกดปุ่ม "Sync" ด้วยมือใน UI (ถ้าตั้งค่าเป็น manual sync สำหรับ production ที่ต้องการความระมัดระวังสูง)

### ขั้นตอนที่ 6: Cluster ถูกปรับให้ตรงกับ Git โดยอัตโนมัติ

```
┌────────────────────┐
│  Deployment:         │
│  payment-service     │
│  replicas: 3 → 5      │  ← Agent สั่ง apply เปลี่ยนแปลง
└────────────────────┘
        │
        ▼
Kubernetes control plane สร้าง Pod เพิ่มอีก 2 ตัว
เพื่อให้ครบ 5 ตัวตามที่ manifest กำหนด
```

### ขั้นตอนที่ 7: ตรวจสอบผลลัพธ์

ทีมสามารถเข้าไปดูสถานะผ่าน dashboard ของ ArgoCD/Flux ได้ทันทีว่า application กลับมาอยู่ในสถานะ **"Synced"** และ **"Healthy"** แล้ว

### สรุป workflow ทั้งหมดเป็นแผนภาพเดียว

```
Developer          Git Repo            GitOps Agent         Kubernetes Cluster
   │                   │                    │                      │
   │── git push ──────▶│                    │                      │
   │  (branch ใหม่)      │                    │                      │
   │                   │                    │                      │
   │── เปิด PR ─────────▶│                    │                      │
   │                   │                    │                      │
   │◀── ทีม review ─────│                    │                      │
   │                   │                    │                      │
   │── approve+merge ─▶│                    │                      │
   │                   │                    │                      │
   │                   │◀── polling/webhook─│                      │
   │                   │── ส่ง manifest ────▶│                      │
   │                   │                    │── เทียบสถานะ ──────────▶│
   │                   │                    │◀── สถานะจริงปัจจุบัน ───│
   │                   │                    │── apply เปลี่ยนแปลง ────▶│
   │                   │                    │                      │── สร้าง/ลบ Pod
   │                   │                    │◀── ยืนยันสถานะใหม่ ─────│   ตามที่ต้องการ
   │                   │                    │                      │
```

จุดที่ควรสังเกตให้ชัดคือ **ไม่มีขั้นตอนไหนเลยที่นักพัฒนาหรือ CI/CD pipeline ต้องมี credential เข้าถึง cluster โดยตรง** — ทุกอย่างเกิดขึ้นผ่าน Git เพียงอย่างเดียว ส่วน agent ที่มีสิทธิ์เข้าถึง cluster นั้นทำงานอยู่ *ภายใน* cluster เองตั้งแต่แรก

---

## Step 757: ข้อดีของ GitOps (Audit Trail, Rollback, Self-Healing)

หลังจากเห็นภาพ workflow แล้ว มาสรุปข้อดีเชิงลึกของ GitOps อย่างเป็นระบบ

### 1. Audit Trail ครบถ้วนจากประวัติ Git

ทุกการเปลี่ยนแปลงต่อระบบ ไม่ว่าจะเป็นการเพิ่ม replica, เปลี่ยน image version, แก้ network policy หรือปรับ resource limit ล้วนถูกบันทึกไว้ในประวัติ Git อย่างสมบูรณ์ ซึ่งประกอบด้วย:

- **ใคร** เป็นคนเสนอการเปลี่ยนแปลง (commit author)
- **เมื่อไหร่** ที่เปลี่ยนแปลง (commit timestamp)
- **ทำไม** ถึงเปลี่ยน (commit message และคำอธิบายใน Pull Request)
- **ใครอนุมัติ** การเปลี่ยนแปลงนั้น (ผู้ approve ใน Pull Request)
- **การเปลี่ยนแปลงมีเนื้อหาอะไรบ้างเป๊ะ ๆ** (git diff)

สิ่งนี้มีความสำคัญมากสำหรับองค์กรที่ต้องผ่านมาตรฐาน compliance ต่าง ๆ เช่น SOC 2, ISO 27001, PCI-DSS ที่มักกำหนดให้ต้องมีหลักฐานการเปลี่ยนแปลงระบบที่ตรวจสอบย้อนหลังได้อย่างสมบูรณ์ — GitOps ให้สิ่งนี้มาโดยธรรมชาติ โดยไม่ต้องสร้างระบบ audit log แยกต่างหาก (เราจะเจาะลึกเรื่อง compliance ในเฟส 8 ต่อไปในหัวข้อหลัง ๆ)

### 2. Rollback ง่ายด้วย `git revert`

เมื่อการ deploy ครั้งล่าสุดทำให้เกิดปัญหา การย้อนกลับสามารถทำได้ง่ายเหมือนการย้อนกลับ commit ธรรมดา:

```bash
# ดูว่า commit ไหนคือตัวที่ทำให้เกิดปัญหา
git log --oneline overlays/production/payment-service/

# ย้อนกลับ commit นั้นด้วยการสร้าง commit ใหม่ที่ทำตรงข้ามกัน
git revert <commit-hash>
git push origin main
```

เมื่อ commit revert นี้ถูก push เข้า branch ที่ GitOps agent เฝ้าดูอยู่ agent จะตรวจพบการเปลี่ยนแปลงและ sync cluster ให้กลับไปเป็นสถานะเดิมโดยอัตโนมัติ **โดยไม่ต้องมีใคร SSH เข้า production หรือรันคำสั่งฉุกเฉินใด ๆ เลย** — กระบวนการ rollback ใช้ workflow เดียวกันเป๊ะกับการ deploy ปกติ (ผ่าน Pull Request, ผ่าน review) ทำให้แม้แต่การ rollback ก็ยังมี audit trail และการตรวจสอบครบถ้วนเช่นเดียวกับการเปลี่ยนแปลงปกติ

ข้อดีเพิ่มเติมคือ เพราะ Git เก็บ **สถานะทั้งระบบเป็น snapshot** (ตามที่เราเรียนมาใน Part 01 เรื่อง Git internals) การ revert จึงนำระบบกลับไปสู่สถานะที่ **สมบูรณ์และสอดคล้องกันทั้งหมด** ไม่ใช่แค่ค่าใดค่าหนึ่งที่แก้ไข ลดความเสี่ยงของการ rollback ที่ทำได้ไม่ครบถ้วน (partial rollback) ซึ่งมักเกิดปัญหาตามมาในระบบที่ไม่ได้ใช้ GitOps

### 3. Self-Healing เมื่อมีคนแก้ Cluster ตรง ๆ

นี่คือคุณสมบัติที่ผลิตผลโดยตรงจากหลักการข้อ 4 (Continuously Reconciled) สมมติสถานการณ์: วิศวกรคนหนึ่งกำลังแก้ปัญหาเร่งด่วนตอนกลางดึก แล้วรัน `kubectl edit deployment payment-service` เพื่อแก้ environment variable ตรง ๆ ใน cluster โดยไม่ผ่าน Git (เพราะรีบ ต้องการแก้ให้เร็วที่สุด)

ในระบบที่ไม่มี GitOps สถานการณ์นี้จะสร้างปัญหาระยะยาว เพราะ:

- ค่าที่แก้ไปจะไม่มีใครรู้ว่าเปลี่ยนไปแล้ว จนกว่าจะมีคนไปเจอเอง
- ถ้ามี deploy รอบถัดไปโดยไม่รู้เรื่อง การเปลี่ยนแปลงฉุกเฉินนั้นอาจถูกเขียนทับหายไปโดยไม่ได้ตั้งใจ หรือในทางกลับกัน อาจตกค้างอยู่แบบไม่มีใครรู้ตัวเป็นเวลานาน

แต่ในระบบที่มี GitOps และเปิดใช้ **selfHeal** (ตามตัวอย่างใน ArgoCD ที่แสดงไว้ใน Step 755) agent จะตรวจพบว่าสถานะจริงของ cluster ไม่ตรงกับที่ Git ประกาศไว้ (drift) ในรอบ reconciliation ถัดไป (ซึ่งมักเกิดขึ้นภายในไม่กี่นาที) แล้ว **ปรับ cluster ให้กลับไปตรงกับ Git โดยอัตโนมัติ** — ทำให้ Git ยังคงเป็น single source of truth ที่แท้จริงอยู่เสมอ ไม่ว่าจะมีใครพยายามแก้ไขระบบข้ามขั้นตอนไปกี่ครั้งก็ตาม

ข้อควรระวังในทางปฏิบัติ: ถ้าการแก้ไขฉุกเฉินนั้นเป็นสิ่งที่ *ต้อง* คงอยู่จริง ๆ (เช่น เป็นการแก้ปัญหาที่ถูกต้องแล้ว) วิศวกรต้อง **นำการเปลี่ยนแปลงนั้นเข้า Git ด้วย** (ผ่าน Pull Request ตามปกติ) ไม่เช่นนั้น self-healing จะเขียนทับการแก้ไขฉุกเฉินนั้นกลับไปเป็นค่าเดิมโดยไม่ตั้งใจ ซึ่งเป็นข้อจำกัดที่ต้องเข้าใจให้ดี (จะกล่าวถึงเพิ่มเติมใน Step 759)

### สรุปข้อดีในภาพรวม

| ข้อดี | กลไกเบื้องหลัง |
|---|---|
| Audit trail สมบูรณ์ | ประวัติ commit + Pull Request ของ Git |
| Rollback ง่ายและปลอดภัย | `git revert` + agent sync อัตโนมัติ |
| Self-healing จาก drift | Reconciliation loop ตรวจสอบต่อเนื่อง |
| ลดพื้นผิวการโจมตี (attack surface) | Credential ของ cluster ไม่ต้องออกไปนอกระบบ |
| Consistency ระหว่าง environment | ทุก environment สะท้อนจาก Git ตรง ๆ ลดการเดา |

---

## Step 758: GitOps กับ Kubernetes — ความสัมพันธ์ที่แนบแน่นที่สุดในทางปฏิบัติ

แม้ในทางทฤษฎีแล้ว หลักการ 4 ข้อของ GitOps จะไม่ได้ผูกติดกับเทคโนโลยีใดเทคโนโลยีหนึ่งโดยเฉพาะ (สามารถนำไปใช้กับระบบใดก็ได้ที่มีแนวคิด declarative state) แต่ในทางปฏิบัติจริง **GitOps แทบจะเป็นคำพ้องความหมายกับ Kubernetes** ไปแล้วในอุตสาหกรรม เหตุผลมีดังนี้

### เหตุผลที่ Kubernetes กับ GitOps เข้ากันได้ดีที่สุด

1. **Kubernetes ถูกออกแบบมาให้เป็น declarative โดยธรรมชาติอยู่แล้ว** — Kubernetes API มีแนวคิด "desired state" ฝังอยู่ในตัวมันเองตั้งแต่ต้น (เช่น `Deployment` ที่บอกว่า "ต้องการ Pod กี่ตัว") ทำให้ Kubernetes มี **reconciliation loop ในระดับ control plane ของตัวเองอยู่แล้ว** (เรียกว่า **controller pattern**) — GitOps เพียงแค่เพิ่มชั้นการ reconcile อีกชั้นหนึ่งที่คอยเทียบระหว่าง Git กับ Kubernetes API แทนที่จะให้คนเป็นคนสั่ง `kubectl apply` เอง

2. **Kubernetes มี API ที่เป็นมาตรฐานเดียวกันทุกที่** — ไม่ว่าจะเป็น cluster บน cloud provider ไหน หรือ on-premise ก็ตาม Kubernetes API มีรูปแบบเดียวกัน ทำให้เครื่องมือ GitOps อย่าง ArgoCD/Flux สามารถทำงานร่วมกับ Kubernetes cluster ใดก็ได้โดยไม่ต้องปรับแก้อะไรมาก

3. **Custom Resource Definition (CRD)** — ความสามารถของ Kubernetes ที่ให้นักพัฒนาสร้าง resource type ของตัวเองได้ (เช่น `Application` ของ ArgoCD, `Kustomization` ของ Flux) ทำให้เครื่องมือ GitOps สามารถ **ฝังตัวเองเข้าไปเป็นส่วนหนึ่งของ Kubernetes ecosystem ได้อย่างเป็นธรรมชาติ** ไม่ต้องสร้างระบบแยกต่างหากจาก Kubernetes เลย

4. **Ecosystem เครื่องมือจัดการ manifest ที่ครบครัน** — เครื่องมืออย่าง **Helm** (package manager สำหรับ Kubernetes) และ **Kustomize** (เครื่องมือปรับแต่ง manifest ตาม environment โดยไม่ต้อง copy ไฟล์ซ้ำ) ล้วนถูกออกแบบมาให้ทำงานร่วมกับแนวคิด GitOps ได้อย่างลงตัว

### ภาพความสัมพันธ์

```
                    ┌────────────────────────────────────┐
                    │           แนวคิด GitOps               │
                    │  (Declarative, Versioned, Pulled,     │
                    │   Continuously Reconciled)             │
                    └───────────────┬────────────────────┘
                                    │ นำไปปฏิบัติได้ดีที่สุดบน
                                    ▼
                    ┌────────────────────────────────────┐
                    │             Kubernetes                │
                    │  - Declarative API โดยธรรมชาติ        │
                    │  - Controller pattern / Reconcile loop │
                    │  - Custom Resource Definition (CRD)    │
                    │  - มาตรฐานเดียวกันทุก cloud/on-prem     │
                    └───────────────┬────────────────────┘
                                    │ implement ผ่านเครื่องมือ
                                    ▼
                    ┌────────────────────────────────────┐
                    │        ArgoCD / Flux (และอื่น ๆ)        │
                    └────────────────────────────────────┘
```

### แล้ว GitOps ใช้กับระบบที่ไม่ใช่ Kubernetes ได้ไหม

ได้ — ในทางทฤษฎีสามารถประยุกต์แนวคิด GitOps กับระบบอื่นได้เช่นกัน เช่น การใช้ Git ควบคุม Terraform state เพื่อจัดการ cloud infrastructure (ซึ่งเราจะเรียนแบบละเอียดใน **Part 77: Infrastructure as Code ร่วมกับ Git**) หรือการใช้ Git ควบคุม configuration ของ network device แต่ในทางปฏิบัติ เครื่องมือ GitOps ที่ครบวงจร (มี agent, reconciliation loop อัตโนมัติเต็มรูปแบบ) ส่วนใหญ่ยังคงถูกสร้างขึ้นมาโดยเฉพาะสำหรับ Kubernetes เป็นหลัก ระบบที่ไม่ใช่ Kubernetes มักต้องอาศัย CI/CD pipeline ทำหน้าที่คล้าย ๆ กัน (เช่น Terraform ที่รันผ่าน CI แล้ว plan/apply อัตโนมัติ) ซึ่งยังถือว่าเป็น push-based มากกว่า pull-based อย่างแท้จริง เว้นแต่จะมีการออกแบบ agent เพิ่มเติมโดยเฉพาะ

---

## Step 759: ข้อจำกัดและความท้าทายของ GitOps

GitOps ไม่ใช่ยาวิเศษที่แก้ปัญหาได้ทุกอย่าง มันมีข้อจำกัดและความท้าทายที่ต้องเข้าใจก่อนนำไปใช้จริง

### ความท้าทายที่ 1: Secret Management ยากขึ้น

นี่คือปัญหาที่สำคัญที่สุดและพบบ่อยที่สุดของทีมที่เริ่มใช้ GitOps เพราะหลักการข้อ 2 ของ GitOps (Versioned and Immutable) บอกว่า **ทุกอย่างต้องถูกเก็บใน Git** แต่ในทางปฏิบัติ เราไม่สามารถเก็บ **secret** เช่น database password, API key, TLS private key ไว้เป็น plaintext ใน Git repository ได้เลย เพราะ:

- Git repository อาจถูกเข้าถึงโดยคนจำนวนมากกว่าที่ควรจะรู้ secret นั้น
- ประวัติ Git เก็บทุกอย่างไว้ตลอดไป **แม้จะลบไฟล์ทิ้งในภายหลัง secret เก่าก็ยังอยู่ในประวัติ** (เว้นแต่จะทำ history rewrite ซึ่งมีความเสี่ยงและยุ่งยากมาก)
- ถ้า repository รั่วไหลออกไป (เช่น ตั้ง private repo ผิดเป็น public โดยไม่ตั้งใจ) secret ทั้งหมดจะรั่วไหลไปด้วย

วิธีแก้ปัญหานี้ในทางปฏิบัติมีหลายแนวทาง (ซึ่งหลักสูตรนี้จะเจาะลึกในเฟส 8 ต่อไปในหัวข้อ Secret Management โดยเฉพาะ) ตัวอย่างแนวทางที่นิยมใช้ร่วมกับ GitOps ได้แก่:

1. **Sealed Secrets** — เข้ารหัส secret ก่อนเก็บลง Git แล้วให้ controller ใน cluster เป็นคนถอดรหัสตอน apply เท่านั้น (คนอื่นที่ไม่มี private key ของ cluster นั้นจะถอดรหัสไม่ได้แม้เห็นไฟล์ใน Git)
2. **External Secrets Operator** — เก็บ manifest ใน Git แค่ "อ้างอิง" ไปยัง secret ที่เก็บอยู่จริงใน secret manager ภายนอก เช่น HashiCorp Vault, AWS Secrets Manager, Azure Key Vault แล้วให้ controller ดึงค่าจริงมาใส่ตอน runtime
3. **SOPS (Secrets OPerationS)** — เข้ารหัสเฉพาะค่า field ที่เป็น secret ในไฟล์ YAML/JSON ก่อน commit เข้า Git โดยใช้ key จาก PGP หรือ cloud KMS

ตัวอย่างแนวคิดของ External Secrets Operator (ใช้ placeholder ทั้งหมด ไม่ใช่ของจริง):

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: payment-service-db-credentials
spec:
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: payment-service-db-credentials
  data:
    - secretKey: password
      remoteRef:
        key: secret/data/payment-service
        property: db_password
```

สังเกตว่าไฟล์นี้ **ไม่มี secret จริงอยู่เลย** มีแค่ "การอ้างอิง" ไปยังตำแหน่งที่เก็บ secret จริงในระบบภายนอก ทำให้ไฟล์นี้ commit เข้า Git ได้อย่างปลอดภัย

### ความท้าทายที่ 2: Learning Curve

การเปลี่ยนจาก push-based deployment แบบเดิมมาเป็น pull-based แบบ GitOps ต้องการความเข้าใจแนวคิดใหม่หลายอย่างพร้อมกัน:

- ทีมต้องเข้าใจแนวคิด declarative configuration อย่างลึกซึ้ง ไม่ใช่แค่เขียน script แบบ imperative เหมือนเดิม
- ทีมต้องเรียนรู้เครื่องมือใหม่ (ArgoCD หรือ Flux) รวมถึงแนวคิดของ Kubernetes CRD และ controller pattern
- การ debug ปัญหาเปลี่ยนรูปแบบไป — เดิมทีถ้า deploy ล้มเหลว จะเห็น error ตรง ๆ ใน CI/CD log แต่ใน GitOps บางครั้งปัญหาอาจซ่อนอยู่ใน sync status ของ agent ซึ่งต้องรู้วิธีอ่าน log และ event ของ agent เอง
- การจัดโครงสร้าง repository ให้เหมาะสม (เช่น จะแยก config repo ออกจาก application repo หรือไม่ จะจัดการหลาย environment อย่างไรด้วย Kustomize overlay หรือ Helm values) ต้องอาศัยประสบการณ์และการออกแบบที่ดีตั้งแต่ต้น มิเช่นนั้นจะกลายเป็นความยุ่งเหยิงในภายหลัง

### ความท้าทายที่ 3: การจัดการ Order/Dependency ระหว่าง Resource

บางครั้งการ deploy มี resource ที่ต้องพึ่งพากันตามลำดับ (เช่น ต้องสร้าง database schema ก่อนที่ application จะ start ได้) การจัดการลำดับการ apply ใน GitOps agent (เช่นการใช้ sync wave ใน ArgoCด หรือ dependsOn ใน Flux) มีความซับซ้อนกว่าการเขียน script แบบ step-by-step ตรง ๆ

### ความท้าทายที่ 4: Drift ที่เกิดจากภายนอกระบบ Kubernetes เอง

ทรัพยากรบางประเภทถูกจัดการโดยกลไกอื่นที่ไม่ใช่ manifest ตรง ๆ เช่น Horizontal Pod Autoscaler (HPA) ที่ปรับ replica count ขึ้นลงอัตโนมัติตาม load — ถ้า agent ตั้งค่า selfHeal ไว้แบบเข้มงวดเกินไป อาจเกิดการ "แย่งกัน" ควบคุมค่า replica ระหว่าง GitOps agent (ที่อยากให้ตรงกับ Git) กับ HPA (ที่อยากปรับตาม load จริง) ซึ่งต้องมีการตั้งค่ายกเว้น (ignore difference) ให้ถูกต้องตั้งแต่แรก

### สรุปข้อจำกัด

| ความท้าทาย | ผลกระทบ | แนวทางบรรเทา |
|---|---|---|
| Secret management | เก็บ secret plaintext ใน Git ไม่ได้ | Sealed Secrets, External Secrets Operator, SOPS |
| Learning curve | ทีมต้องเรียนรู้แนวคิดและเครื่องมือใหม่ | ฝึกฝนแบบค่อยเป็นค่อยไป เริ่มจาก non-critical environment ก่อน |
| Resource ordering | Deploy ผิดลำดับอาจทำให้ระบบพัง | ใช้ sync wave / dependency annotation ของเครื่องมือ |
| Drift จากกลไกอัตโนมัติอื่น (เช่น HPA) | Agent กับกลไกอื่นแย่งกันควบคุม resource | ตั้งค่า ignoreDifferences ให้ถูกต้อง |

---

## Step 760: แบบฝึกหัด — ออกแบบ GitOps Workflow อย่างง่ายสำหรับ Deploy แอปเข้า Kubernetes Cluster จำลอง

ถึงเวลาลงมือทำความเข้าใจให้แน่นด้วยการออกแบบเอง โจทย์ของแบบฝึกหัดนี้คือ

> **ออกแบบ GitOps workflow อย่างง่าย สำหรับ deploy แอปพลิเคชันสมมติชื่อ `notes-api` เข้าสู่ Kubernetes cluster จำลอง โดยวาดไดอะแกรมและอธิบายทุกขั้นตอนอย่างละเอียด**

### โจทย์กำหนดสถานการณ์

- ทีมมี Kubernetes cluster จำลอง 2 ชุด คือ `staging` และ `production`
- แอป `notes-api` เก็บซอร์สโค้ดไว้ที่ repository ชื่อ `notes-api` (application repo)
- แยก repository สำหรับเก็บ manifest ไว้ต่างหากชื่อ `notes-api-manifests` (config repo) ซึ่งมีโครงสร้างโฟลเดอร์ดังนี้:

```
notes-api-manifests/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── staging/
    │   └── kustomization.yaml
    └── production/
        └── kustomization.yaml
```

- ใช้ ArgoCD เป็น GitOps agent ที่ติดตั้งอยู่ใน cluster ทั้งสองชุด

### ขั้นตอนที่ 1: ร่างไดอะแกรมของ workflow ทั้งหมด

ลองวาดไดอะแกรมของตัวเองก่อนตามความเข้าใจ แล้วเทียบกับตัวอย่างด้านล่างนี้:

```
┌───────────────────┐        ┌──────────────────────────┐
│   notes-api          │        │   notes-api-manifests       │
│   (application repo) │        │   (config repo)              │
│                      │        │                              │
│  1. เขียนโค้ดฟีเจอร์ใหม่ │        │                              │
│  2. push เข้า branch   │        │                              │
│  3. CI build image     │        │                              │
│     tag ใหม่ เช่น        │        │                              │
│     v1.4.0             │        │                              │
│  4. push image ขึ้น      │        │                              │
│     container registry │        │                              │
└──────────┬───────────┘        └──────────────┬───────────────┘
           │                                    ▲
           │  5. CI ของ notes-api ยิง PR อัตโนมัติ  │
           │     มาที่ manifests repo เพื่อ           │
           │     อัปเดต image tag เป็น v1.4.0        │
           └────────────────────────────────────┘
                                                 │
                                     6. ทีมรีวิว PR แล้ว merge
                                                 │
                                                 ▼
                              ┌──────────────────────────┐
                              │  overlays/staging/          │
                              │  kustomization.yaml         │
                              │  image: notes-api:v1.4.0     │
                              └──────────────┬───────────┘
                                             │
                          7. ArgoCD (staging instance)
                             ตรวจพบการเปลี่ยนแปลง แล้ว sync
                                             │
                                             ▼
                              ┌──────────────────────────┐
                              │   Staging Cluster            │
                              │   notes-api v1.4.0 รันอยู่     │
                              └──────────────┬───────────┘
                                             │
                          8. ทีมทดสอบบน staging ผ่านแล้ว
                                             │
                          9. เปิด PR แยกต่างหาก เพื่อ
                             promote image tag เดียวกัน
                             เข้า overlays/production/
                                             │
                         10. ทีม (เช่น tech lead) approve PR
                                             │
                                             ▼
                              ┌──────────────────────────┐
                              │  overlays/production/       │
                              │  kustomization.yaml         │
                              │  image: notes-api:v1.4.0     │
                              └──────────────┬───────────┘
                                             │
                        11. ArgoCD (production instance)
                            ตั้งเป็น manual sync — รอ
                            คนกด "Sync" ใน UI ด้วยตัวเอง
                            (เพราะ production ต้องระวังสูง)
                                             │
                        12. Tech lead กด Sync ใน ArgoCD UI
                                             │
                                             ▼
                              ┌──────────────────────────┐
                              │   Production Cluster         │
                              │   notes-api v1.4.0 รันอยู่     │
                              └──────────────────────────┘
```

### ขั้นตอนที่ 2: อธิบายเหตุผลเบื้องหลังการออกแบบแต่ละจุด

ลองตอบคำถามต่อไปนี้เพื่อฝึกความเข้าใจ (เฉลยแนวคิดอยู่ด้านล่าง อย่าเพิ่งเลื่อนไปดูจนกว่าจะลองคิดเอง):

1. **ทำไมต้องแยก `notes-api` (application repo) ออกจาก `notes-api-manifests` (config repo)?**
2. **ทำไม staging ตั้งเป็น auto-sync แต่ production ตั้งเป็น manual sync?**
3. **ถ้าทีมพบว่า `notes-api v1.4.0` บน production มีบั๊กร้ายแรงหลัง deploy ไปแล้ว ควรทำอย่างไรตาม workflow แบบ GitOps?**
4. **ทำไมต้องใช้ Kustomize overlay แยกโฟลเดอร์ staging/production แทนที่จะมี manifest ชุดเดียวใช้ร่วมกันทั้งหมด?**

### แนวทางเฉลย (ลองคิดเองก่อนแล้วค่อยเทียบ)

1. **การแยก repository** ช่วยแยกความรับผิดชอบ (separation of concerns) ระหว่าง "โค้ดแอปพลิเคชันเปลี่ยนบ่อยแค่ไหน" กับ "การตั้งค่า deploy เปลี่ยนบ่อยแค่ไหน" นักพัฒนาแอปไม่จำเป็นต้องมีสิทธิ์เข้าถึงหรือแก้ไข manifest ของ production โดยตรง และทีม platform/SRE สามารถควบคุมสิทธิ์ในการแก้ manifest repo ได้อย่างเข้มงวดกว่า โดยไม่ไปยุ่งกับ workflow การพัฒนาโค้ดตามปกติ

2. **staging** เป็นสภาพแวดล้อมที่ความเสี่ยงต่ำกว่า ต้องการ feedback loop ที่เร็ว การตั้ง auto-sync ทำให้ทีมเห็นผลลัพธ์ของการเปลี่ยนแปลงเกือบจะทันทีหลัง merge ในขณะที่ **production** ต้องการจุดตรวจสอบสุดท้ายจากมนุษย์ก่อนที่การเปลี่ยนแปลงจะมีผลกระทบกับผู้ใช้จริง การตั้ง manual sync คือการเพิ่ม "ประตูอนุมัติ" อีกชั้นหนึ่งที่แยกจาก Pull Request approval

3. ตาม workflow ของ GitOps การ rollback ควรทำผ่าน Git เสมอ ไม่ใช่เข้าไปแก้ cluster ตรง ๆ — วิธีที่ถูกต้องคือเปิด Pull Request เพื่อเปลี่ยนค่า image tag ใน `overlays/production/kustomization.yaml` กลับไปเป็นเวอร์ชันก่อนหน้า (เช่น `v1.3.2`) แล้วให้ผ่านกระบวนการ review/approve ตามปกติ หรือใช้ `git revert` กับ commit ที่ทำให้เกิด v1.4.0 ก็ได้ผลลัพธ์เดียวกัน จากนั้น ArgoCD จะ sync cluster กลับสู่เวอร์ชันเดิมให้เอง

4. Kustomize overlay ช่วยให้ **ค่าพื้นฐาน (base) ที่เหมือนกันในทุก environment ถูกเขียนไว้ที่เดียว** (ไม่ต้อง copy ซ้ำ) และมีแค่ **ส่วนต่าง (patch)** ที่แตกต่างกันในแต่ละ environment เท่านั้นที่ต้องระบุแยกไว้ใน overlay ของ environment นั้น ๆ เช่น production อาจตั้ง `replicas: 5` และ resource limit สูงกว่า staging ที่ตั้ง `replicas: 1` เพื่อประหยัด cost — วิธีนี้ลดการเขียนซ้ำ (DRY) และลดความเสี่ยงที่ staging กับ production จะเบี่ยงเบนออกจากกันโดยไม่ได้ตั้งใจ

### ส่วนขยาย (ถ้าต้องการฝึกเพิ่ม)

ลองขยายไดอะแกรมของตัวเองต่อ โดยเพิ่มสถานการณ์เหล่านี้เข้าไป:

- เพิ่ม environment `dev` เข้ามาอีกหนึ่งชุด ที่ sync ทันทีจากทุก branch (ไม่ใช่แค่ `main`) เพื่อให้นักพัฒนาทดสอบงานระหว่างทำได้เร็วที่สุด
- เพิ่ม policy check อัตโนมัติก่อน merge PR เข้า manifests repo (เช่น ตรวจว่า resource limit ต้องไม่เกินเพดานที่กำหนดไว้) — ลองคิดว่าควรใส่ตรงจุดไหนของ workflow
- ลองนึกภาพว่าถ้าใช้ Flux แทน ArgoCD workflow ข้างต้นจะเปลี่ยนไปตรงไหนบ้าง (คำใบ้: หลักการโดยรวมเหมือนเดิมทุกอย่าง เปลี่ยนแค่รายละเอียดของ Custom Resource ที่ใช้ประกาศ)

การฝึกออกแบบ workflow ด้วยตัวเองแบบนี้คือทักษะที่สำคัญที่สุดสำหรับวิศวกรที่จะต้องนำ GitOps ไปใช้จริงในองค์กร เพราะในทางปฏิบัติแต่ละองค์กรจะมีรายละเอียดที่แตกต่างกันไปตามขนาดทีม จำนวน environment และระดับความเสี่ยงที่ยอมรับได้ ไม่มีสูตรสำเร็จตายตัวที่ใช้ได้กับทุกที่

---

## สรุป Part 76

ใน Part นี้เราได้เรียนรู้ว่า:

1. **GitOps** คือแนวปฏิบัติที่ใช้ Git เป็นแหล่งความจริงเดียว (single source of truth) สำหรับสถานะที่ต้องการของระบบ แล้วให้ agent อัตโนมัติปรับระบบจริงให้ตรงกับ Git อยู่เสมอ — ศัพท์นี้ถูกบัญญัติโดย Weaveworks ในปี 2017
2. หลักการ 4 ข้อที่ระบบต้องมีครบเพื่อจะเรียกว่าเป็น GitOps อย่างแท้จริง ได้แก่ **Declarative, Versioned and Immutable, Pulled Automatically, Continuously Reconciled**
3. Git ในบริบทของ GitOps ไม่ได้เก็บแค่ application source code แต่ขยายไปเก็บ Kubernetes manifest, Infrastructure as Code, security policy และการตั้งค่าอื่น ๆ ของทั้งระบบ
4. **Push-based deployment** (CI/CD ดั้งเดิม) ให้ระบบภายนอกยิงคำสั่งเข้าไปหา target ในขณะที่ **pull-based deployment** (GitOps) ให้ agent ภายใน target เป็นฝ่ายดึงการเปลี่ยนแปลงเข้ามาเอง ซึ่งปลอดภัยและตรวจจับ drift ได้ดีกว่า
5. **ArgoCD** และ **Flux** คือเครื่องมือ GitOps ยอดนิยมที่สุดในระบบนิเวศ Kubernetes ทั้งคู่เป็นโครงการ Graduated ของ CNCF
6. GitOps workflow ตัวอย่างแสดงให้เห็นว่าการเปลี่ยนแปลงทุกอย่างไหลผ่าน Pull Request ก่อนเข้าสู่ระบบจริงเสมอ โดยไม่ต้องมีใครมี credential เข้าถึง production โดยตรง
7. ข้อดีหลักของ GitOps คือ **audit trail ที่สมบูรณ์, rollback ที่ทำได้ง่ายด้วย `git revert`, และ self-healing** เมื่อมีใครแก้ไขระบบข้ามขั้นตอน
8. GitOps กับ Kubernetes มีความสัมพันธ์แนบแน่นที่สุดในทางปฏิบัติ เพราะ Kubernetes ถูกออกแบบมาให้เป็น declarative และมี reconciliation loop อยู่ในตัวเองแล้ว
9. ข้อจำกัดสำคัญของ GitOps คือ **การจัดการ secret** ที่ต้องอาศัยเครื่องมือเสริม (Sealed Secrets, External Secrets Operator, SOPS) และ **learning curve** ที่ทีมต้องปรับตัว
10. การฝึกออกแบบ GitOps workflow ด้วยตัวเองเป็นทักษะสำคัญที่สุดที่จะนำไปประยุกต์ใช้ได้จริงในองค์กรที่มีบริบทแตกต่างกัน

**ต่อไป:** [Part 77: Infrastructure as Code ร่วมกับ Git (Terraform + Git)](./part-077-infrastructure-as-code.md)
