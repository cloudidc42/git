# Part 73: Automated Testing ใน Pipeline: Unit, Integration, E2E

> **Step ในหลักสูตรนี้:** Step 721–730
> **เฟส:** 7 — CI/CD เต็มรูปแบบด้วย GitHub Actions และ GitLab CI/CD
> **เป้าหมายของ Part นี้:** เข้าใจว่าทำไม Automated Testing ต้องเป็นส่วนหนึ่งของ pipeline ไม่ใช่แค่ขั้นตอนที่ทำแยกต่างหาก เข้าใจแนวคิด Test Pyramid และนำไปใช้ออกแบบชุดทดสอบจริง เขียน Unit test, Integration test และ E2E test ให้รันอัตโนมัติได้ทั้งบน GitHub Actions และ GitLab CI/CD จัดการเรื่อง test report, coverage report, parallel testing, flaky test และวาง Quality Gate เพื่อป้องกันไม่ให้โค้ดคุณภาพต่ำถูก merge เข้า branch หลัก

---

## สารบัญของ Part นี้

- Step 721: ทำไม Automated Testing ใน Pipeline ถึงสำคัญมาก
- Step 722: Test Pyramid — โครงสร้างชุดทดสอบที่สมดุล
- Step 723: Unit Test ใน Pipeline — ตัวอย่างจริงด้วย Jest และ pytest
- Step 724: Integration Test ใน Pipeline — ทดสอบกับ Service จริงอย่าง Database
- Step 725: E2E Test ใน Pipeline — ตัวอย่างด้วย Playwright และ Cypress
- Step 726: Test Reports และ Coverage Reports ใน CI UI
- Step 727: Parallel Testing — แบ่งชุดทดสอบรันขนานเพื่อความเร็ว
- Step 728: Flaky Tests — ปัญหาที่กัดกร่อนความเชื่อมั่นใน Pipeline
- Step 729: Quality Gates — บล็อกการ Merge ด้วยเกณฑ์คุณภาพ
- Step 730: แบบฝึกหัด — เพิ่ม Unit Test พร้อม Coverage Report เข้า Pipeline จริง

---

## Step 721: ทำไม Automated Testing ใน Pipeline ถึงสำคัญมาก

ใน Part ก่อนหน้านี้เราได้เรียนรู้วิธีสร้าง pipeline ที่ build โค้ด, lint โค้ด และ deploy อัตโนมัติ แต่ยังขาดหัวใจสำคัญที่สุดชิ้นหนึ่งไป นั่นคือ **การทดสอบอัตโนมัติ (Automated Testing)** ที่ฝังอยู่ในทุกครั้งที่มีการเปลี่ยนแปลงโค้ด

### ปัญหาของการทดสอบแบบ Manual

ลองนึกภาพทีมที่ยังไม่มี automated testing ใน pipeline:

1. นักพัฒนาแก้โค้ด แล้ว push ขึ้น branch
2. เปิด Pull Request / Merge Request
3. QA เข้ามาทดสอบด้วยมือ ตามเช็คลิสต์ที่จำไว้ในหัว หรือเขียนไว้ใน spreadsheet
4. ถ้า QA ลืมทดสอบบางเคส บั๊กก็หลุดไปถึง production
5. เมื่อทีมโตขึ้น ฟีเจอร์เยอะขึ้น การทดสอบด้วยมือใช้เวลานานขึ้นเรื่อย ๆ จนกลายเป็นคอขวด (bottleneck) ของทั้งทีม

ปัญหาเหล่านี้ไม่ได้เกิดจากความไม่เอาใจใส่ของ QA แต่เกิดจาก **ธรรมชาติของมนุษย์ที่ไม่สามารถจำและทำซ้ำสิ่งเดิม ๆ ได้แม่นยำ 100% ทุกครั้ง** ยิ่งระบบซับซ้อนขึ้น ยิ่งมี edge case มากขึ้น ความเสี่ยงที่จะพลาดก็สูงขึ้นตามไปด้วย

### Automated Testing ใน Pipeline แก้ปัญหาอะไรบ้าง

| ปัญหาจากการทดสอบด้วยมือ | สิ่งที่ Automated Testing ใน Pipeline แก้ให้ |
|---|---|
| พึ่งพาความจำและความละเอียดของคน | Test case ถูกเขียนเป็นโค้ด รันซ้ำได้แม่นยำทุกครั้งเหมือนเดิม 100% |
| ทดสอบช้า กว่าจะรู้ผลต้องรอ QA ว่าง | รันอัตโนมัติทันทีที่มีการ push หรือเปิด PR/MR รู้ผลภายในไม่กี่นาที |
| ทดสอบไม่ครบทุกครั้ง (สุ่มเลือกเทสเคส) | รันทุก test case ที่มีทุกครั้ง ไม่มีการข้าม |
| บั๊กหลุดไปถึง production บ่อย | จับบั๊กได้ตั้งแต่ก่อน merge เข้า branch หลัก ก่อนถึงมือผู้ใช้จริง |
| ไม่มีหลักฐานว่าทดสอบอะไรไปบ้าง | มี test report และ log เก็บไว้ทุกครั้งที่รัน ตรวจสอบย้อนหลังได้ |
| การ refactor โค้ดน่ากลัวเพราะไม่รู้ว่าจะพังตรงไหน | มี test suite คอยเป็น "ตาข่ายนิรภัย (safety net)" คอยเตือนทันทีถ้า refactor แล้วพฤติกรรมเปลี่ยน |

### หลักการสำคัญ: "Shift Left"

แนวคิดสำคัญของวงการ DevOps คือ **Shift Left Testing** — หมายถึงการเลื่อนจุดที่ตรวจพบปัญหาให้เร็วขึ้น (ไปทางซ้ายของ timeline) แทนที่จะไปเจอปัญหาตอนหลัง

```
แบบเดิม (ตรวจพบช้า แพงมาก):
Code ──▶ Build ──▶ Deploy ──▶ Manual QA ──▶ Production ──▶ ผู้ใช้เจอบั๊ก ──▶ ลูกค้าโทรมาบ่น

แบบ Shift Left (ตรวจพบเร็ว ถูกกว่ามาก):
Code ──▶ [Automated Test รันทันที] ──▶ พบบั๊ก ──▶ แก้ก่อน merge ──▶ Build ──▶ Deploy อย่างมั่นใจ
```

งานวิจัยด้าน Software Engineering หลายชิ้นชี้ตรงกันว่า **ต้นทุนในการแก้บั๊กจะเพิ่มขึ้นแบบทวีคูณ** ตามระยะที่บั๊กเดินทางไปไกลจากจุดที่มันถูกสร้างขึ้น:

- แก้บั๊กตอนเขียนโค้ด (ก่อน commit): ต้นทุนต่ำที่สุด แทบไม่มีค่าใช้จ่าย
- แก้บั๊กที่ automated test จับได้ใน pipeline: ต้นทุนต่ำ ใช้เวลาไม่กี่นาที
- แก้บั๊กที่ QA เจอตอนทดสอบด้วยมือ: ต้นทุนปานกลาง ต้องรอรอบ QA ใหม่
- แก้บั๊กที่หลุดไป production แล้วลูกค้าเจอ: ต้นทุนสูงมาก ทั้งค่าใช้จ่ายในการ hotfix, ความเสียหายต่อชื่อเสียง, และอาจสูญเสียลูกค้า

นี่คือเหตุผลที่ทุกบริษัทเทคโนโลยีชั้นนำในโลก ตั้งแต่บริษัท startup เล็ก ๆ ไปจนถึง Google, Netflix, Amazon ต่างวาง automated testing ไว้เป็นหัวใจของ CI/CD pipeline โดยไม่มีข้อยกเว้น

### Automated Testing ไม่ได้แทนที่ Manual Testing ทั้งหมด

ต้องเข้าใจให้ชัดว่า automated testing ไม่ได้มาแทนที่ QA มนุษย์ 100% แต่มาช่วย**แบ่งเบาภาระงานที่ซ้ำซากและตรวจสอบได้ชัดเจน** ให้เครื่องจักรทำแทน เพื่อให้ QA มนุษย์มีเวลาไปโฟกัสกับสิ่งที่เครื่องทำไม่ได้ดีเท่า เช่น:

- Exploratory testing (ทดสอบแบบสำรวจ หาบั๊กที่ไม่มีใครคาดคิด)
- Usability testing (ทดสอบว่าใช้งานง่ายหรือไม่ ในมุมมองมนุษย์จริง)
- Edge case ที่ซับซ้อนทางธุรกิจซึ่งต้องใช้วิจารณญาณ

ใน Part นี้เราจะโฟกัสที่การนำ automated test ทั้งสามระดับ — **Unit, Integration, E2E** — เข้าไปฝังใน pipeline ทั้งบน **GitHub Actions** และ **GitLab CI/CD** เพื่อให้คุณนำไปปรับใช้ได้ไม่ว่าทีมของคุณจะใช้แพลตฟอร์มไหน

---

## Step 722: Test Pyramid — โครงสร้างชุดทดสอบที่สมดุล

เมื่อเข้าใจแล้วว่าทำไมต้องมี automated test คำถามถัดไปคือ **"แล้วเราควรเขียนเทสแบบไหน สัดส่วนเท่าไหร่?"** คำตอบมาตรฐานของวงการคือแนวคิดที่เรียกว่า **Test Pyramid** ซึ่งถูกเสนอครั้งแรกโดย Mike Cohn

### ไดอะแกรมของ Test Pyramid

```
                    ▲
                   ╱ ╲
                  ╱E2E╲              ← น้อยที่สุด, ช้าที่สุด, เปราะบางที่สุด, แพงที่สุด
                 ╱─────╲                (ทดสอบทั้งระบบผ่าน UI จริง)
                ╱       ╲
               ╱Integration╲         ← ปานกลาง
              ╱─────────────╲           (ทดสอบการทำงานร่วมกันของหลาย component/service)
             ╱               ╲
            ╱   Unit Tests    ╲     ← เยอะที่สุด, เร็วที่สุด, เสถียรที่สุด, ถูกที่สุด
           ╱───────────────────╲       (ทดสอบฟังก์ชัน/class เดี่ยว ๆ แยกจากกัน)
          ╱_____________________╲
```

### ทำความเข้าใจแต่ละชั้น

**1. Unit Test (ฐานพีระมิด — เยอะที่สุด)**

ทดสอบ "หน่วยที่เล็กที่สุด" ของโค้ด เช่น ฟังก์ชันเดียว, method เดียว, class เดียว โดย**แยกออกจากส่วนอื่นของระบบอย่างสมบูรณ์** (isolated) มักใช้ mock/stub แทนสิ่งที่อยู่ภายนอก เช่น database, API ภายนอก

ตัวอย่างสิ่งที่ unit test ทดสอบ:
```javascript
// ฟังก์ชันธรรมดา ไม่พึ่งพาอะไรภายนอกเลย
function calculateDiscount(price, discountPercent) {
  if (discountPercent < 0 || discountPercent > 100) {
    throw new Error("เปอร์เซ็นต์ส่วนลดต้องอยู่ระหว่าง 0-100");
  }
  return price - (price * discountPercent / 100);
}
```

**คุณสมบัติของ Unit Test:**
- รันเร็วมาก (มิลลิวินาทีต่อเทส) เพราะไม่ต้องเชื่อมต่อ network, database, filesystem
- เสถียรมาก แทบไม่มี flaky (ไม่ผ่านบ้างไม่ผ่านบ้างโดยไม่มีเหตุผล)
- เขียนง่าย ดูแลรักษาง่าย
- ควรมีจำนวนมากที่สุดในระบบ (โดยทั่วไปแนะนำ 70-80% ของ test suite ทั้งหมด)

**2. Integration Test (ตรงกลางพีระมิด — น้อยลง)**

ทดสอบว่า**หลาย ๆ component ทำงานร่วมกันได้ถูกต้อง** เช่น โค้ดเชื่อมต่อกับ database จริงได้ไหม, API endpoint คุยกับ service อื่นได้ถูกต้องไหม

ตัวอย่าง: ทดสอบว่าฟังก์ชัน `createUser()` เขียนข้อมูลลง PostgreSQL จริง ๆ ได้ถูกต้อง ไม่ใช่แค่ mock database

**คุณสมบัติของ Integration Test:**
- ช้ากว่า unit test เพราะต้องมี service จริงรันอยู่ (database, message queue ฯลฯ)
- ตรวจจับปัญหาที่ unit test มองไม่เห็น เช่น query SQL ผิด, schema ไม่ตรงกัน
- ควรมีจำนวนปานกลาง (ประมาณ 15-20% ของ test suite)

**3. E2E Test (ยอดพีระมิด — น้อยที่สุด)**

ทดสอบ**ทั้งระบบจากมุมมองผู้ใช้จริง** ผ่าน UI จริงหรือ API จริงแบบครบวงจร เช่น เปิด browser จำลอง คลิกปุ่ม login กรอกฟอร์ม ตรวจสอบว่า flow ทั้งหมดทำงานถูกต้องตั้งแต่ต้นจนจบ

**คุณสมบัติของ E2E Test:**
- ช้าที่สุด (อาจใช้เวลาเป็นวินาทีหรือนาทีต่อเทสหนึ่งเคส)
- เปราะบางที่สุด (flaky ได้ง่ายที่สุด เพราะขึ้นกับหลายปัจจัย เช่น network latency, timing ของ UI)
- ดูแลรักษายากที่สุด เพราะเมื่อ UI เปลี่ยน เทสก็ต้องแก้ตาม
- ควรมีจำนวนน้อยที่สุด เน้นเฉพาะ critical user journey เท่านั้น (ประมาณ 5-10% ของ test suite)

### ทำไมต้องเป็น "รูปพีระมิด" ไม่ใช่รูปอื่น

นี่คือคำถามสำคัญที่หลายทีมเข้าใจผิด บางทีมเขียน E2E test เยอะมากเพราะ "รู้สึกมั่นใจกว่า" เนื่องจากมันทดสอบเหมือนผู้ใช้จริงที่สุด แต่กลับกลายเป็น **Ice Cream Cone Anti-pattern** (พีระมิดหัวกลับ) ซึ่งเป็นปัญหาใหญ่:

```
Anti-pattern: Ice Cream Cone (E2E เยอะเกินไป)

          ╱─────────────────────╲
         ╱      E2E Tests        ╲    ← เยอะที่สุด (ผิด!)
        ╱─────────────────────────╲
       ╱    Integration Tests      ╲
      ╱─────────────────────────────╲
     ╱        Unit Tests             ╲  ← น้อยที่สุด (ผิด!)
    ╲_________________________________╱
```

**ปัญหาของ Ice Cream Cone:**

1. **Pipeline ช้ามาก** — ถ้ามี E2E test 500 เคส แต่ละเคสใช้เวลา 10-30 วินาที รวมแล้ว pipeline อาจใช้เวลาเป็นชั่วโมง ทำให้ feedback loop ช้าลงมหาศาล นักพัฒนาต้องรอนานกว่าจะรู้ว่าโค้ดที่แก้ไปพังหรือไม่
2. **Flaky มาก จนไม่มีใครเชื่อผล test อีกต่อไป** — เมื่อ E2E test ล้มเหลวบ่อยโดยไม่เกี่ยวกับบั๊กจริง (เช่น network กระตุก, element โหลดช้า) ทีมจะเริ่มเพิกเฉยต่อผล test ("อ๋อ มันแค่ flaky rerun อีกทีก็ผ่าน") ซึ่งอันตรายมาก เพราะสุดท้ายจะพลาดบั๊กจริงไปด้วย
3. **หา root cause ยาก** — เมื่อ E2E test fail มันบอกแค่ว่า "flow นี้พังตรงไหนสักที่" แต่ไม่บอกว่าฟังก์ชันไหนหรือบรรทัดไหนที่ทำให้พัง ต้อง debug ยาวนานกว่า unit test ที่ชี้จุดผิดตรง ๆ
4. **ค่าใช้จ่ายในการดูแลรักษาสูง** — ทุกครั้งที่ UI เปลี่ยนแม้เพียงเล็กน้อย (เช่น เปลี่ยน CSS class ของปุ่ม) อาจทำให้ E2E test หลายสิบเคสพังพร้อมกัน

### หลักการเลือกว่าจะเขียน Test ระดับไหน

ใช้หลักคิดง่าย ๆ นี้:

| คำถาม | คำตอบชี้ไปที่ |
|---|---|
| ต้องการทดสอบ logic ของฟังก์ชันเดี่ยว ๆ เช่น การคำนวณ, การ validate | Unit Test |
| ต้องการทดสอบว่าโค้ดคุยกับ database/API ภายนอกได้ถูกต้อง | Integration Test |
| ต้องการทดสอบว่า user journey ทั้งหมด (เช่น สมัครสมาชิก → login → checkout) ทำงานถูกต้องจากต้นจนจบ | E2E Test (เฉพาะ critical path เท่านั้น) |

กฎทองที่จำง่าย ๆ คือ:

> **"เขียน Unit Test ให้มากที่สุดเท่าที่จะทำได้ เขียน Integration Test เท่าที่จำเป็น และเขียน E2E Test เฉพาะ flow ที่สำคัญที่สุดของธุรกิจเท่านั้น"**

ใน Step ถัดไปเราจะเริ่มลงมือเขียน pipeline ที่รันเทสทั้งสามระดับนี้จริง ๆ ทั้งบน GitHub Actions และ GitLab CI/CD

---

## Step 723: Unit Test ใน Pipeline — ตัวอย่างจริงด้วย Jest และ pytest

มาเริ่มลงมือกันจริง ๆ ในระดับฐานของพีระมิด นั่นคือ Unit Test เราจะใช้ **Jest** (สำหรับโปรเจกต์ JavaScript/Node.js) และ **pytest** (สำหรับโปรเจกต์ Python) เป็นตัวอย่าง

### 723.1 ตัวอย่างโค้ดและ Unit Test ด้วย Jest (JavaScript)

สมมติเรามีฟังก์ชันคำนวณส่วนลดในไฟล์ `src/discount.js`:

```javascript
// src/discount.js
function calculateDiscount(price, discountPercent) {
  if (typeof price !== "number" || price < 0) {
    throw new Error("ราคาต้องเป็นตัวเลขที่ไม่ติดลบ");
  }
  if (discountPercent < 0 || discountPercent > 100) {
    throw new Error("เปอร์เซ็นต์ส่วนลดต้องอยู่ระหว่าง 0-100");
  }
  return Math.round((price - (price * discountPercent) / 100) * 100) / 100;
}

module.exports = { calculateDiscount };
```

และไฟล์ทดสอบ `src/discount.test.js`:

```javascript
// src/discount.test.js
const { calculateDiscount } = require("./discount");

describe("calculateDiscount", () => {
  test("คำนวณส่วนลด 10% จากราคา 100 ได้ถูกต้อง", () => {
    expect(calculateDiscount(100, 10)).toBe(90);
  });

  test("ส่วนลด 0% ราคาต้องเท่าเดิม", () => {
    expect(calculateDiscount(500, 0)).toBe(500);
  });

  test("ส่วนลด 100% ราคาต้องเป็น 0", () => {
    expect(calculateDiscount(250, 100)).toBe(0);
  });

  test("โยน error เมื่อราคาติดลบ", () => {
    expect(() => calculateDiscount(-10, 5)).toThrow(
      "ราคาต้องเป็นตัวเลขที่ไม่ติดลบ"
    );
  });

  test("โยน error เมื่อ discountPercent เกิน 100", () => {
    expect(() => calculateDiscount(100, 150)).toThrow(
      "เปอร์เซ็นต์ส่วนลดต้องอยู่ระหว่าง 0-100"
    );
  });
});
```

`package.json` ต้องมี script สำหรับรันเทส:

```json
{
  "name": "discount-service",
  "version": "1.0.0",
  "scripts": {
    "test": "jest --ci --coverage"
  },
  "devDependencies": {
    "jest": "^29.7.0"
  }
}
```

### 723.2 รัน Jest บน GitHub Actions

สร้างไฟล์ `.github/workflows/unit-tests.yml`:

```yaml
name: Unit Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  unit-test-js:
    name: Unit Test (Node.js / Jest)
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm test
```

### 723.3 ตัวอย่างโค้ดและ Unit Test ด้วย pytest (Python)

สมมติเรามีฟังก์ชันเดียวกันในภาษา Python ที่ `app/discount.py`:

```python
# app/discount.py
def calculate_discount(price: float, discount_percent: float) -> float:
    if price < 0:
        raise ValueError("ราคาต้องเป็นตัวเลขที่ไม่ติดลบ")
    if discount_percent < 0 or discount_percent > 100:
        raise ValueError("เปอร์เซ็นต์ส่วนลดต้องอยู่ระหว่าง 0-100")
    return round(price - (price * discount_percent / 100), 2)
```

และไฟล์ทดสอบ `tests/test_discount.py`:

```python
# tests/test_discount.py
import pytest
from app.discount import calculate_discount


def test_discount_10_percent():
    assert calculate_discount(100, 10) == 90


def test_discount_zero_percent():
    assert calculate_discount(500, 0) == 500


def test_discount_100_percent():
    assert calculate_discount(250, 100) == 0


def test_negative_price_raises_error():
    with pytest.raises(ValueError, match="ราคาต้องเป็นตัวเลขที่ไม่ติดลบ"):
        calculate_discount(-10, 5)


def test_discount_over_100_raises_error():
    with pytest.raises(ValueError, match="เปอร์เซ็นต์ส่วนลดต้องอยู่ระหว่าง 0-100"):
        calculate_discount(100, 150)
```

`requirements-dev.txt`:

```
pytest==8.2.0
pytest-cov==5.0.0
```

### 723.4 รัน pytest บน GitLab CI/CD

สร้างไฟล์ `.gitlab-ci.yml` (หรือเพิ่มลงไฟล์ที่มีอยู่แล้ว):

```yaml
stages:
  - test

unit-test-python:
  stage: test
  image: python:3.12-slim
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'
    - if: '$CI_COMMIT_BRANCH == "develop"'
  before_script:
    - pip install --no-cache-dir -r requirements-dev.txt
  script:
    - pytest --cov=app --cov-report=term --cov-report=xml tests/
  artifacts:
    paths:
      - coverage.xml
    expire_in: 7 days
```

### 723.5 สังเกตความแตกต่างและความเหมือนของทั้งสองแพลตฟอร์ม

| แนวคิด | GitHub Actions | GitLab CI/CD |
|---|---|---|
| จุดเริ่มต้น pipeline | `on:` (push, pull_request) | `rules:` หรือ `only`/`except` |
| หน่วยงานย่อยที่สุด | `job` ภายใต้ `jobs:` | `job` ที่ประกาศเป็น key ระดับบนสุด |
| กำหนดลำดับ job | `needs:` | `stage:` + `stages:` |
| environment ที่ job รัน | `runs-on:` (ต้องใช้ runner ที่มี tool อยู่แล้ว หรือ setup เอง) | `image:` (ใช้ Docker image ได้ทันที) |
| เก็บไฟล์ผลลัพธ์ | `actions/upload-artifact@v4` | `artifacts:` |

ทั้งสองแพลตฟอร์มทำสิ่งเดียวกันได้ เพียงแต่ syntax และ mental model ต่างกันเล็กน้อย — GitLab มักจะสะดวกกว่าเรื่อง Docker image (เพราะ `image:` ฝังอยู่ในทุก job) ส่วน GitHub Actions มักสะดวกกว่าเรื่อง Marketplace Actions ที่มีให้เลือกเยอะมาก

### 723.6 ข้อควรระวังสำคัญของ Unit Test ใน Pipeline

1. **ต้องไม่พึ่งพา service ภายนอกเด็ดขาด** — ถ้า unit test ของคุณต้องต่อ internet หรือต่อ database จริง แปลว่านั่นไม่ใช่ unit test แล้ว แต่เป็น integration test ที่ปนกัน ให้แยกไฟล์ให้ชัดเจน (เช่น `*.test.js` สำหรับ unit, `*.integration.test.js` สำหรับ integration)
2. **ใช้ `--ci` flag เสมอเมื่อรันบน pipeline** — Jest มี flag `--ci` ที่ปรับพฤติกรรมให้เหมาะกับ CI environment เช่น ไม่พยายาม interactive watch mode
3. **ตั้ง exit code ให้ถูกต้อง** — ถ้า test fail แม้แต่เคสเดียว คำสั่งต้อง exit ด้วยรหัสที่ไม่ใช่ 0 เพื่อให้ pipeline รู้ว่าต้อง fail job นั้น (Jest และ pytest ทำแบบนี้เป็นค่า default อยู่แล้ว)

---

## Step 724: Integration Test ใน Pipeline — ทดสอบกับ Service จริงอย่าง Database

Unit test ทดสอบโค้ดแบบแยกส่วน แต่ในโลกจริง แอปพลิเคชันส่วนใหญ่ต้องคุยกับ database, cache (เช่น Redis), หรือ message queue เราจึงต้องมี **Integration Test** ที่ทดสอบการทำงานร่วมกันจริง ๆ

ความท้าทายคือ pipeline runner ปกติไม่มี database รันอยู่ ดังนั้นทั้ง GitHub Actions และ GitLab CI/CD จึงมีฟีเจอร์ **`services:`** ที่ช่วยสร้าง container เสริม (เช่น PostgreSQL, MySQL, Redis) ให้รันคู่ขนานไปกับ job ของเรา

### 724.1 ตัวอย่างโค้ด Integration Test

สมมติเรามีฟังก์ชันที่เชื่อมต่อ PostgreSQL จริง ที่ `src/userRepository.js`:

```javascript
// src/userRepository.js
const { Pool } = require("pg");

function createPool() {
  return new Pool({
    host: process.env.DB_HOST || "localhost",
    port: process.env.DB_PORT || 5432,
    user: process.env.DB_USER || "postgres",
    password: process.env.DB_PASSWORD || "postgres",
    database: process.env.DB_NAME || "app_test",
  });
}

async function createUser(pool, { name, email }) {
  const result = await pool.query(
    "INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id, name, email",
    [name, email]
  );
  return result.rows[0];
}

module.exports = { createPool, createUser };
```

ไฟล์ทดสอบ `src/userRepository.integration.test.js`:

```javascript
// src/userRepository.integration.test.js
const { createPool, createUser } = require("./userRepository");

let pool;

beforeAll(async () => {
  pool = createPool();
  await pool.query(`
    CREATE TABLE IF NOT EXISTS users (
      id SERIAL PRIMARY KEY,
      name VARCHAR(255) NOT NULL,
      email VARCHAR(255) UNIQUE NOT NULL
    )
  `);
});

afterEach(async () => {
  await pool.query("TRUNCATE TABLE users RESTART IDENTITY CASCADE");
});

afterAll(async () => {
  await pool.end();
});

describe("createUser (integration กับ PostgreSQL จริง)", () => {
  test("บันทึก user ลง database ได้จริง", async () => {
    const user = await createUser(pool, {
      name: "สมชาย ใจดี",
      email: "somchai@example.com",
    });

    expect(user.id).toBeDefined();
    expect(user.name).toBe("สมชาย ใจดี");
    expect(user.email).toBe("somchai@example.com");
  });

  test("บันทึก email ซ้ำต้อง error เพราะมี UNIQUE constraint", async () => {
    await createUser(pool, { name: "คนแรก", email: "dup@example.com" });

    await expect(
      createUser(pool, { name: "คนที่สอง", email: "dup@example.com" })
    ).rejects.toThrow();
  });
});
```

### 724.2 Integration Test บน GitHub Actions ด้วย `services:`

```yaml
name: Integration Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  integration-test:
    name: Integration Test (Node.js + PostgreSQL)
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: app_test
        ports:
          - 5432:5432
        # ตรวจสอบว่า PostgreSQL พร้อมรับ connection ก่อนเริ่มรันเทส
        options: >-
          --health-cmd "pg_isready -U postgres"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run integration tests
        run: npm run test:integration
        env:
          DB_HOST: localhost
          DB_PORT: 5432
          DB_USER: postgres
          DB_PASSWORD: postgres
          DB_NAME: app_test
```

**จุดสำคัญที่ต้องสังเกต:**

- `services:` ใน GitHub Actions ประกาศไว้ระดับ **job** และรันเป็น Docker container แยกต่างหาก แต่แชร์ network เดียวกับ job container ทำให้เข้าถึงผ่าน `localhost` ได้เลย (เมื่อรันบน `ubuntu-latest` runner)
- `options:` กับ `--health-cmd` สำคัญมาก เพราะถ้าไม่มี health check, job อาจเริ่มรันเทสก่อนที่ PostgreSQL จะพร้อมรับ connection จริง ทำให้เทส fail แบบ flaky

### 724.3 Integration Test บน GitLab CI/CD ด้วย `services:`

```yaml
stages:
  - test

integration-test:
  stage: test
  image: node:20-slim
  services:
    - name: postgres:16
      alias: postgres
  variables:
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: postgres
    POSTGRES_DB: app_test
    DB_HOST: postgres
    DB_PORT: "5432"
    DB_USER: postgres
    DB_PASSWORD: postgres
    DB_NAME: app_test
  before_script:
    - npm ci
    # รอให้ PostgreSQL พร้อมก่อน (GitLab ไม่มี health-check แบบ built-in เหมือน GitHub Actions)
    - apt-get update -qq && apt-get install -y -qq postgresql-client
    - |
      until pg_isready -h postgres -p 5432 -U postgres; do
        echo "รอ PostgreSQL พร้อมใช้งาน..."
        sleep 2
      done
  script:
    - npm run test:integration
```

**จุดสำคัญที่ต้องสังเกต:**

- GitLab CI ใช้ `services:` ระดับ job เช่นกัน แต่การเข้าถึง service ทำผ่าน **`alias`** ที่กำหนด (ในที่นี้คือ `postgres`) ไม่ใช่ `localhost`
- GitLab ไม่มี built-in health check เหมือน GitHub Actions ในรูปแบบ `options:` ดังนั้นทีมส่วนใหญ่จะเขียน loop รอด้วย `pg_isready` เองใน `before_script`
- ตัวแปร `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` เป็นชื่อ environment variable มาตรฐานที่ image ทางการของ `postgres` รู้จักและใช้ตอน initialize database

### 724.4 เปรียบเทียบ Services ระหว่างสองแพลตฟอร์ม

| หัวข้อ | GitHub Actions | GitLab CI/CD |
|---|---|---|
| การเข้าถึง service | ผ่าน `localhost` (บน Linux runner) | ผ่าน alias ที่ตั้งชื่อ (หรือชื่อ image ถ้าไม่ตั้ง alias) |
| Health check | มี `options` รองรับ health-cmd ในตัว | ไม่มี built-in ต้องเขียน wait script เอง |
| จำนวน services ต่อ job | ได้หลายตัว | ได้หลายตัว |
| รูปแบบ | key `services:` เป็น map ของชื่อ service | key `services:` เป็น list ของ image |

### 724.5 ตัวอย่างเดียวกันด้วย Redis (แสดงว่าใช้ service อื่นก็ทำแบบเดียวกันได้)

GitHub Actions:

```yaml
services:
  redis:
    image: redis:7
    ports:
      - 6379:6379
    options: >-
      --health-cmd "redis-cli ping"
      --health-interval 10s
      --health-timeout 5s
      --health-retries 5
```

GitLab CI:

```yaml
services:
  - name: redis:7
    alias: redis
```

หลักการเหมือนกันทุกประการไม่ว่าจะเป็น service ประเภทไหน — สิ่งที่ต้องจำคือ **integration test จะช้ากว่า unit test เสมอเพราะต้องรอ service เริ่มทำงาน** ดังนั้นควรแยก job/stage ของ integration test ออกจาก unit test เพื่อให้ unit test ยังคงรันเร็วและให้ feedback ไวอยู่เสมอ

---

## Step 725: E2E Test ใน Pipeline — ตัวอย่างด้วย Playwright และ Cypress

ยอดพีระมิดคือ E2E Test (End-to-End Test) ที่ทดสอบระบบทั้งหมดผ่าน UI จริงเหมือนผู้ใช้งานจริง เราจะดูตัวอย่างด้วยสองเครื่องมือยอดนิยมที่สุดในปัจจุบัน: **Playwright** (จาก Microsoft) และ **Cypress**

### 725.1 ตัวอย่าง E2E Test ด้วย Playwright

สมมติเว็บแอปของเรามีหน้า login ที่ `https://staging.example.com/login` เราต้องการทดสอบ flow การ login จริง

```javascript
// e2e/login.spec.js
const { test, expect } = require("@playwright/test");

test.describe("Login Flow", () => {
  test("ผู้ใช้ login สำเร็จด้วย credential ที่ถูกต้อง", async ({ page }) => {
    await page.goto("/login");

    await page.fill('input[name="email"]', "test@example.com");
    await page.fill('input[name="password"]', "SecurePass123!");
    await page.click('button[type="submit"]');

    // รอให้ redirect ไปหน้า dashboard และตรวจสอบข้อความต้อนรับ
    await expect(page).toHaveURL(/.*dashboard/);
    await expect(page.locator("h1")).toContainText("ยินดีต้อนรับ");
  });

  test("แสดง error เมื่อกรอกรหัสผ่านผิด", async ({ page }) => {
    await page.goto("/login");

    await page.fill('input[name="email"]', "test@example.com");
    await page.fill('input[name="password"]', "wrong-password");
    await page.click('button[type="submit"]');

    await expect(page.locator(".error-message")).toContainText(
      "อีเมลหรือรหัสผ่านไม่ถูกต้อง"
    );
  });
});
```

`playwright.config.js`:

```javascript
// playwright.config.js
const { defineConfig } = require("@playwright/test");

module.exports = defineConfig({
  testDir: "./e2e",
  timeout: 30000,
  retries: process.env.CI ? 2 : 0,
  use: {
    baseURL: process.env.BASE_URL || "http://localhost:3000",
    screenshot: "only-on-failure",
    video: "retain-on-failure",
    trace: "retain-on-failure",
  },
  reporter: [
    ["list"],
    ["junit", { outputFile: "playwright-results/junit.xml" }],
    ["html", { outputFolder: "playwright-report", open: "never" }],
  ],
});
```

### 725.2 รัน Playwright บน GitHub Actions

```yaml
name: E2E Tests (Playwright)

on:
  pull_request:
    branches: [main]

jobs:
  e2e-test:
    name: E2E Test (Playwright)
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium

      - name: Build application
        run: npm run build

      - name: Start application in background
        run: npm run start &

      - name: Wait for application to be ready
        run: npx wait-on http://localhost:3000

      - name: Run E2E tests
        run: npx playwright test
        env:
          BASE_URL: http://localhost:3000

      - name: Upload Playwright report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 14
```

**จุดสำคัญ:** ขั้นตอน `npx playwright install --with-deps chromium` จำเป็นมาก เพราะ runner ของ GitHub Actions ไม่มี browser และ dependency ของระบบ (เช่น library กราฟิก) ติดตั้งมาให้ล่วงหน้า ต้องดึงมาเองทุกครั้ง

### 725.3 ตัวอย่าง E2E Test ด้วย Cypress

```javascript
// cypress/e2e/checkout.cy.js
describe("Checkout Flow", () => {
  beforeEach(() => {
    cy.visit("/products");
  });

  it("เพิ่มสินค้าลงตะกร้าและ checkout สำเร็จ", () => {
    cy.get('[data-testid="product-card"]').first().click();
    cy.get('[data-testid="add-to-cart"]').click();
    cy.get('[data-testid="cart-icon"]').click();

    cy.get('[data-testid="checkout-button"]').click();

    cy.get('input[name="address"]').type("123 ถนนสุขุมวิท กรุงเทพฯ");
    cy.get('input[name="phone"]').type("0812345678");
    cy.get('[data-testid="confirm-order"]').click();

    cy.contains("สั่งซื้อสำเร็จ").should("be.visible");
    cy.url().should("include", "/order-confirmation");
  });
});
```

`cypress.config.js`:

```javascript
// cypress.config.js
const { defineConfig } = require("cypress");

module.exports = defineConfig({
  e2e: {
    baseUrl: process.env.CYPRESS_BASE_URL || "http://localhost:3000",
    video: true,
    screenshotOnRunFailure: true,
    reporter: "junit",
    reporterOptions: {
      mochaFile: "cypress/results/results-[hash].xml",
    },
  },
});
```

### 725.4 รัน Cypress บน GitLab CI/CD

```yaml
stages:
  - test

e2e-test:
  stage: test
  image: cypress/browsers:node-20.11.1-chrome-123.0.6312.86-1-ff-124.0.1-edge-123.0.2420.65-1
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  before_script:
    - npm ci
    - npm run build
    - npm run start &
    - npx wait-on http://localhost:3000
  script:
    - npx cypress run --browser chrome
  artifacts:
    when: always
    paths:
      - cypress/videos/
      - cypress/screenshots/
      - cypress/results/
    reports:
      junit: cypress/results/results-*.xml
    expire_in: 14 days
```

**จุดสำคัญ:** GitLab มี Docker image ทางการชื่อ `cypress/browsers` ที่ติดตั้ง browser (Chrome, Firefox, Edge) และ dependency ที่จำเป็นมาให้ครบแล้ว ทำให้ไม่ต้องเสียเวลาติดตั้ง browser เองเหมือนฝั่ง GitHub Actions (แม้ Cypress จะมี Docker image ทางการให้ใช้บน GitHub Actions ได้เช่นกัน ผ่าน `container:` key)

### 725.5 ข้อควรระวังที่สำคัญที่สุดของ E2E Test ใน Pipeline

1. **ต้อง deploy หรือ start แอปพลิเคชันให้พร้อมก่อนรันเทสเสมอ** — E2E test ต้องมีระบบจริงให้ทดสอบ ไม่ว่าจะเป็นการ build แล้ว start ใน pipeline เอง (ตามตัวอย่างข้างต้น) หรือรันเทสเล็งไปที่ staging environment ที่ deploy ไว้แล้ว
2. **ใช้ `wait-on` หรือเครื่องมือคล้ายกันเสมอ** — อย่าใช้ `sleep 10` แบบเดา เพราะเวลา build/start แอปอาจไม่คงที่ ใช้เครื่องมือที่ poll จนกว่า endpoint จะตอบสนองจริง
3. **เก็บ screenshot/video เมื่อ fail เสมอ** — เพราะ E2E test debug ยากที่สุด การมีภาพหน้าจอหรือวิดีโอตอน fail ช่วยลดเวลาสืบสวนได้มาก ทั้ง Playwright และ Cypress รองรับฟีเจอร์นี้ในตัว
4. **จำกัดจำนวน E2E test ให้น้อยที่สุดเท่าที่จำเป็น** ตามหลัก Test Pyramid ใน Step 722 — เลือกเฉพาะ critical user journey เช่น login, checkout, payment เท่านั้น ไม่ต้องทดสอบทุกหน้าทุกปุ่มด้วย E2E

---

## Step 726: Test Reports และ Coverage Reports ใน CI UI

การรันเทสแล้วดูแค่ log ดิบ ๆ ใน terminal ไม่มีประสิทธิภาพพอสำหรับทีมขนาดใหญ่ ทั้ง GitHub Actions และ GitLab CI/CD จึงมีระบบแสดงผล **test report** และ **coverage report** ที่สวยงามอ่านง่ายบน UI ของแพลตฟอร์มโดยตรง

### 726.1 รูปแบบไฟล์มาตรฐาน: JUnit XML

ทั้งสองแพลตฟอร์มใช้มาตรฐานเดียวกันในการอ่าน test report นั่นคือรูปแบบไฟล์ **JUnit XML** (`.xml`) ซึ่ง test framework เกือบทุกตัวในทุกภาษารองรับการ export ออกมาในรูปแบบนี้:

- Jest → ใช้ package เสริม `jest-junit`
- pytest → มี flag `--junitxml` ในตัวอยู่แล้ว
- Playwright → มี reporter `junit` ในตัวอยู่แล้ว
- Cypress → ใช้ `mochawesome` หรือ `junit` reporter

### 726.2 Test Report บน GitLab CI/CD

GitLab มีฟีเจอร์ `reports.junit` ที่ทำให้ผล test แสดงเป็นตารางสวยงามใน UI ของ Merge Request โดยตรง พร้อมไอคอนติ๊กถูก/กากบาทข้างแต่ละเทสเคส:

```yaml
unit-test-python:
  stage: test
  image: python:3.12-slim
  before_script:
    - pip install --no-cache-dir -r requirements-dev.txt
  script:
    - pytest --junitxml=report.xml --cov=app --cov-report=xml:coverage.xml --cov-report=term tests/
  coverage: '/TOTAL.+ ([0-9]{1,3}%)/'
  artifacts:
    when: always
    reports:
      junit: report.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
```

**อธิบายทีละส่วน:**

- `reports.junit: report.xml` — บอก GitLab ให้อ่านไฟล์นี้แล้วแสดงผลเป็นตาราง test result ใน tab "Tests" ของ pipeline และใน Merge Request widget
- `reports.coverage_report` — บอก GitLab ให้อ่านไฟล์ coverage แล้วแสดง**เส้น highlight สีเขียว/แดงทับบนโค้ด diff** ใน Merge Request โดยตรง (บรรทัดไหนถูกทดสอบจะเป็นสีเขียว บรรทัดไหนไม่ถูกทดสอบจะเป็นสีแดง) — ต้องใช้ format `cobertura` เท่านั้น
- `coverage:` (regex ที่ระดับ job) — เป็นคนละอันกับ `coverage_report` โดย key นี้ใช้ดึงตัวเลข % coverage จาก log แบบ text ธรรมดา เพื่อไปแสดงเป็น badge บนหน้า project และในหน้า pipeline overview

ผลลัพธ์คือเมื่อเปิด Merge Request บน GitLab จะเห็น:
1. Widget สรุปว่า "Unit tests: 24 passed, 0 failed"
2. เมื่อกด diff ของไฟล์ที่แก้ จะเห็นเส้นสีเขียว/แดงบอกว่าโค้ดที่แก้ถูกทดสอบครอบคลุมแค่ไหน

### 726.3 Test Report บน GitHub Actions

GitHub Actions ไม่มี key สำเร็จรูปแบบ GitLab แต่ใช้วิธีผ่าน **Actions จาก Marketplace** ร่วมกับ **Job Summary** (GITHUB_STEP_SUMMARY):

```yaml
name: Unit Tests with Reports

on:
  pull_request:
    branches: [main]

jobs:
  unit-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - run: npm ci

      - name: Run tests with coverage
        run: npm test -- --ci --coverage --reporters=default --reporters=jest-junit
        env:
          JEST_JUNIT_OUTPUT_DIR: "./test-results"
          JEST_JUNIT_OUTPUT_NAME: "junit.xml"

      - name: Publish test report
        uses: dorny/test-reporter@v1
        if: always()
        with:
          name: Jest Test Results
          path: "test-results/junit.xml"
          reporter: jest-junit

      - name: Add coverage summary to Job Summary
        if: always()
        run: |
          echo "## รายงานความครอบคลุมของ Test" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          cat coverage/coverage-summary.json | \
            node -e "
              const data = JSON.parse(require('fs').readFileSync(0, 'utf8'));
              const t = data.total;
              console.log('| ประเภท | เปอร์เซ็นต์ |');
              console.log('|---|---|');
              console.log('| Lines | ' + t.lines.pct + '% |');
              console.log('| Statements | ' + t.statements.pct + '% |');
              console.log('| Functions | ' + t.functions.pct + '% |');
              console.log('| Branches | ' + t.branches.pct + '% |');
            " >> $GITHUB_STEP_SUMMARY

      - name: Upload coverage HTML report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/lcov-report/
          retention-days: 14
```

**อธิบายทีละส่วน:**

- `dorny/test-reporter@v1` — Action ยอดนิยมที่อ่านไฟล์ JUnit XML แล้วสร้าง "Check" แยกต่างหากบนหน้า Pull Request แสดงจำนวน pass/fail และรายละเอียดแต่ละเทสเคสที่คลิกดูได้
- `$GITHUB_STEP_SUMMARY` — เป็นไฟล์พิเศษที่ GitHub เตรียมไว้ให้ เขียน Markdown ลงไปแล้วจะถูกแสดงเป็นสรุปสวยงามในหน้า workflow run โดยอัตโนมัติ เหมาะมากสำหรับสรุป coverage เป็นตาราง
- `actions/upload-artifact` — เก็บรายงาน HTML แบบละเอียดไว้ให้ดาวน์โหลดดูภายหลังได้ ถ้าต้องการดูบรรทัดไหนบ้างที่ไม่ถูกทดสอบแบบละเอียด

### 726.4 การแสดง Coverage Badge บน README

หลายทีมนิยมใส่ badge แสดง % coverage ไว้บน README เพื่อให้เห็นสถานะล่าสุดได้ทันที ทั้งสองแพลตฟอร์มรองรับ:

**GitLab:** ไปที่ Settings → CI/CD → General pipelines แล้ว copy URL ของ badge ที่ GitLab สร้างให้อัตโนมัติจาก regex `coverage:` ที่ตั้งไว้ในไฟล์ `.gitlab-ci.yml`

**GitHub:** มักใช้บริการภายนอกอย่าง [Codecov](https://codecov.io) หรือ [Coveralls](https://coveralls.io) ร่วมกับ GitHub Actions โดยเพิ่ม step อัปโหลดไฟล์ coverage:

```yaml
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/coverage-final.json
          token: ${{ secrets.CODECOV_TOKEN }}
```

### 726.5 สรุปเปรียบเทียบ

| หัวข้อ | GitHub Actions | GitLab CI/CD |
|---|---|---|
| แสดงผล test pass/fail ใน UI | ต้องใช้ Action เสริม เช่น `dorny/test-reporter` | มี built-in `reports.junit` |
| แสดง diff coverage ทับโค้ดใน MR/PR | ต้องพึ่งบริการภายนอก (Codecov, Coveralls) | มี built-in `reports.coverage_report` |
| สรุปผลแบบ custom | `$GITHUB_STEP_SUMMARY` (Markdown) | ไม่มี built-in เทียบเท่าโดยตรง มักใช้ artifact + badge แทน |
| Coverage badge | ผ่านบริการภายนอกเป็นหลัก | มี built-in badge จาก regex `coverage:` |

---

## Step 727: Parallel Testing — แบ่งชุดทดสอบรันขนานเพื่อความเร็ว

เมื่อโปรเจกต์โตขึ้น จำนวน test case ก็เพิ่มขึ้นตาม ถ้ามี test 2,000 เคสแล้วรันทีละเคสตามลำดับ (sequential) pipeline อาจใช้เวลานานหลายสิบนาที วิธีแก้คือ **Parallel Testing** — แบ่ง test suite ออกเป็นหลายส่วนแล้วรันพร้อมกันบนหลาย job/runner

### 727.1 Parallel Testing บน GitLab CI/CD ด้วย `parallel:`

GitLab มีฟีเจอร์ `parallel:` ที่ทำให้ job เดียวถูก "โคลน" ออกเป็นหลาย instance รันพร้อมกันโดยอัตโนมัติ พร้อมส่ง environment variable `CI_NODE_INDEX` และ `CI_NODE_TOTAL` เข้าไปให้แต่ละ instance รู้ว่าตัวเองเป็นลำดับที่เท่าไหร่จากทั้งหมดกี่ตัว

```yaml
stages:
  - test

unit-test-parallel:
  stage: test
  image: node:20-slim
  parallel: 4
  before_script:
    - npm ci
  script:
    - >
      npx jest
      --ci
      --shard=${CI_NODE_INDEX}/${CI_NODE_TOTAL}
      --coverage
      --coverageDirectory="coverage-shard-${CI_NODE_INDEX}"
  artifacts:
    paths:
      - coverage-shard-*/
    expire_in: 7 days
```

ในตัวอย่างนี้ Jest จะถูกรันเป็น 4 instance พร้อมกัน (`parallel: 4`) แต่ละ instance ใช้ flag `--shard` ของ Jest เอง เพื่อแบ่งไฟล์ทดสอบออกเป็น 4 ส่วนเท่า ๆ กันตามลำดับที่ได้รับ (`CI_NODE_INDEX` มีค่าตั้งแต่ 1 ถึง `CI_NODE_TOTAL`)

**Parallel แบบ Matrix ของ GitLab (parallel:matrix)**

GitLab ยังรองรับการรันขนานแบบระบุค่าตัวแปรต่างกันในแต่ละ instance ได้ด้วย:

```yaml
integration-test-matrix:
  stage: test
  image: node:20-slim
  parallel:
    matrix:
      - DB_ENGINE: postgres
        DB_VERSION: "16"
      - DB_ENGINE: postgres
        DB_VERSION: "15"
      - DB_ENGINE: mysql
        DB_VERSION: "8"
  services:
    - name: "${DB_ENGINE}:${DB_VERSION}"
      alias: database
  script:
    - npm run test:integration -- --db-engine=$DB_ENGINE
```

รูปแบบนี้เหมาะมากสำหรับการทดสอบว่าโค้ดของเราทำงานถูกต้องกับหลายเวอร์ชันของ database พร้อมกันในคราวเดียว

### 727.2 Parallel Testing บน GitHub Actions ด้วย `matrix`

GitHub Actions ใช้ **strategy matrix** ในการทำสิ่งเดียวกัน:

```yaml
name: Unit Tests Parallel

on:
  pull_request:
    branches: [main]

jobs:
  unit-test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - run: npm ci

      - name: Run tests (shard ${{ matrix.shard }}/4)
        run: npx jest --ci --shard=${{ matrix.shard }}/4 --coverage

      - name: Upload coverage shard
        uses: actions/upload-artifact@v4
        with:
          name: coverage-shard-${{ matrix.shard }}
          path: coverage/
```

**อธิบาย:** `strategy.matrix.shard: [1, 2, 3, 4]` ทำให้ GitHub Actions สร้าง job 4 ชุดพร้อมกัน แต่ละ job ได้รับค่า `matrix.shard` ต่างกัน (1, 2, 3, 4) นำไปใช้กับ flag `--shard` ของ Jest เหมือนกับตัวอย่าง GitLab

`fail-fast: false` สำคัญมาก — ถ้าไม่ตั้งค่านี้ (ค่า default คือ `true`) เมื่อ shard ใดหนึ่ง fail, GitHub Actions จะ**ยกเลิก matrix job ที่เหลือทันที** ทำให้เราไม่เห็นผลลัพธ์ของ shard อื่นเลย ซึ่งไม่เหมาะกับ test suite ที่เราอยากเห็นภาพรวมทั้งหมด

### 727.3 รวมผล Coverage จากหลาย Shard เข้าด้วยกัน

เมื่อรัน test แบบ shard แล้ว แต่ละ shard จะได้ coverage เป็นของตัวเอง ต้องมี job แยกต่างหากเพื่อรวมผลลัพธ์ทั้งหมดเข้าด้วยกัน:

GitHub Actions:

```yaml
  merge-coverage:
    needs: unit-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Download all coverage shards
        uses: actions/download-artifact@v4
        with:
          pattern: coverage-shard-*
          path: coverage-shards/
          merge-multiple: false

      - run: npm install -g nyc
      - name: Merge coverage reports
        run: |
          mkdir -p .nyc_output
          nyc merge coverage-shards .nyc_output/out.json
          nyc report --reporter=text --reporter=lcov
```

GitLab CI:

```yaml
merge-coverage:
  stage: report
  needs: ["unit-test-parallel"]
  image: node:20-slim
  script:
    - npm install -g nyc
    - mkdir -p .nyc_output
    - nyc merge coverage-shard-1 .nyc_output/1.json
    - nyc merge coverage-shard-2 .nyc_output/2.json
    - nyc merge coverage-shard-3 .nyc_output/3.json
    - nyc merge coverage-shard-4 .nyc_output/4.json
    - nyc report --reporter=text --reporter=cobertura
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
```

### 727.4 ผลลัพธ์ของการทำ Parallel Testing

สมมติ test suite เดิมใช้เวลา 20 นาทีถ้ารันทีละเคส เมื่อแบ่งเป็น 4 shard รันขนานกัน (โดยสมมติว่า infrastructure รองรับ runner พร้อมกัน 4 ตัว) เวลาจะลดลงเหลือประมาณ **5 นาที** (บวก overhead เล็กน้อยสำหรับ setup แต่ละ job) ซึ่งเป็นการเพิ่มความเร็วแบบมีนัยสำคัญมาก โดยเฉพาะเมื่อทีมมี pull request จำนวนมากต่อวัน

**ข้อควรระวัง:** การเพิ่มจำนวน parallel job มากเกินไปอาจไม่คุ้มค่า เพราะ overhead ของการ setup environment (checkout code, install dependency) ในแต่ละ job ก็ใช้เวลาเช่นกัน ต้องหาจุดสมดุลระหว่างจำนวน shard กับเวลาที่ประหยัดได้จริง โดยทั่วไปแนะนำให้เริ่มจาก 2-4 shard แล้ววัดผลจริงก่อนปรับเพิ่ม

---

## Step 728: Flaky Tests — ปัญหาที่กัดกร่อนความเชื่อมั่นใน Pipeline

### 728.1 Flaky Test คืออะไร

**Flaky Test** คือ test case ที่ **บางครั้งผ่าน (pass) บางครั้งไม่ผ่าน (fail) โดยที่โค้ดไม่ได้ถูกแก้ไขเลย** — รันซ้ำด้วยโค้ดชุดเดียวกันเป๊ะ ๆ แต่ผลลัพธ์ไม่คงที่

```
รันครั้งที่ 1: ✅ PASS
รันครั้งที่ 2: ✅ PASS
รันครั้งที่ 3: ❌ FAIL   ← โค้ดไม่ได้เปลี่ยนอะไรเลย!
รันครั้งที่ 4: ✅ PASS
```

### 728.2 สาเหตุที่พบบ่อยของ Flaky Test

1. **Race condition / Timing issue** — โดยเฉพาะใน E2E test ที่รอ element บน UI โผล่มาไม่ทันเวลาที่กำหนด (เช่น network ช้ากว่าปกติในบางรอบ)
2. **การพึ่งพาลำดับการรันเทส (Test Order Dependency)** — เทสตัวหนึ่งทิ้งข้อมูลไว้ใน database หรือ shared state แล้วเทสตัวถัดไปพึ่งพาข้อมูลนั้นโดยไม่ตั้งใจ ถ้าลำดับการรันเปลี่ยน (เช่นตอนรันแบบ parallel) ก็จะ fail
3. **การใช้เวลาจริง (System time) ในเทส** — เช่นเทสที่เขียนแบบ `expect(result).toBe(new Date())` ซึ่งอาจต่างกันไม่กี่มิลลิวินาที
4. **Resource ไม่เพียงพอบน CI runner** — runner ที่ share resource กับ job อื่นอาจทำให้ timeout ที่ตั้งไว้แน่นเกินไปพลาดในบางรอบ
5. **การสุ่ม (Randomness) ที่ไม่ได้ seed ค่าคงที่** — เทสที่ใช้ข้อมูลสุ่มโดยไม่ fix seed อาจสร้าง input ที่ trigger edge case แบบสุ่มเป็นบางครั้ง
6. **External dependency ที่ไม่เสถียร** — เทสที่ยิง API ภายนอกจริง (ไม่ได้ mock) แล้ว service นั้นล่มหรือช้าเป็นบางครั้ง

### 728.3 ทำไม Flaky Test ถึงอันตรายกว่าที่คิด

Flaky test ไม่ใช่แค่ "น่ารำคาญ" แต่เป็นภัยเงียบต่อวัฒนธรรมคุณภาพของทีม:

> **เมื่อทีมเจอ test fail บ่อย ๆ โดยไม่เกี่ยวกับบั๊กจริง พฤติกรรมตามธรรมชาติของมนุษย์คือเริ่ม "เพิกเฉย" ต่อผล test — กด rerun โดยไม่คิดอะไร จนวันหนึ่งที่ test fail เพราะบั๊กจริง ก็จะถูกมองข้ามไปเหมือนเป็น flaky อีกครั้ง**

นี่คือปรากฏการณ์ที่เรียกว่า **"alert fatigue"** ในบริบทของ automated testing และเป็นสาเหตุอันดับต้น ๆ ที่ทำให้ automated testing ล้มเหลวทั้งระบบ แม้ว่าจะเขียนเทสไว้เยอะแล้วก็ตาม

### 728.4 วิธีจัดการ Flaky Tests

**1. Retry อัตโนมัติ (แก้ปัญหาระยะสั้น)**

ทั้ง Playwright, Cypress, Jest มีกลไก retry ในตัว:

Playwright:
```javascript
module.exports = defineConfig({
  retries: process.env.CI ? 2 : 0, // retry สูงสุด 2 ครั้งบน CI เท่านั้น
});
```

GitHub Actions (ระดับ job/step):
```yaml
      - name: Run E2E tests with retry
        uses: nick-fields/retry@v3
        with:
          timeout_minutes: 10
          max_attempts: 2
          command: npx playwright test
```

GitLab CI (ระดับ job):
```yaml
e2e-test:
  script:
    - npx playwright test
  retry:
    max: 2
    when:
      - script_failure
```

**สำคัญมาก:** retry เป็นแค่ **"พลาสเตอร์ปิดแผล" ระยะสั้น** ไม่ใช่การแก้ปัญหาที่ต้นเหตุ ทีมต้องไม่ใช้ retry เป็นข้ออ้างที่จะไม่แก้ flaky test จริง ๆ

**2. Quarantine (กักกัน) Flaky Test**

เมื่อพบว่า test ไหน flaky บ่อยมาก ให้ **แยกออกจาก pipeline หลักชั่วคราว** ไปไว้ใน suite แยกต่างหาก (เรียกว่า quarantine suite) เพื่อไม่ให้มันบล็อกการ merge ของทีมทั้งหมด ในขณะที่ยังคงรันมันอยู่เบื้องหลังเพื่อเก็บข้อมูลไว้วิเคราะห์

Jest:
```javascript
// ใช้ test.skip ชั่วคราวพร้อม comment อธิบายเหตุผลและ ticket ติดตาม
test.skip("โหลดข้อมูล dashboard ภายใน 2 วินาที (flaky - ดู JIRA-1234)", async () => {
  // ...
});
```

หรือแยก tag ด้วย Playwright:
```javascript
test("โหลดข้อมูล dashboard @quarantine", async ({ page }) => {
  // ...
});
```

แล้วรันแยก job:
```yaml
e2e-test-stable:
  script:
    - npx playwright test --grep-invert "@quarantine"

e2e-test-quarantine:
  script:
    - npx playwright test --grep "@quarantine"
  allow_failure: true   # GitLab: ไม่บล็อก pipeline แม้ job นี้ fail
```

GitHub Actions มี key เทียบเท่ากันคือ `continue-on-error: true`:
```yaml
      - name: Run quarantined tests
        run: npx playwright test --grep "@quarantine"
        continue-on-error: true
```

**3. ตั้งกฎของทีม: Flaky Test ต้องถูกแก้ภายในเวลาที่กำหนด**

ทีมที่มีวินัยดีจะตั้งกฎชัดเจน เช่น "test ที่ถูก quarantine ต้องถูกแก้ไขหรือลบทิ้งภายใน 2 sprint ไม่เช่นนั้นจะถูกลบออกจาก suite ถาวร" เพื่อไม่ให้ quarantine suite กลายเป็นที่ทิ้งขยะถาวรที่ไม่มีใครกลับไปดูอีกเลย

**4. เครื่องมือตรวจจับ Flaky Test อัตโนมัติ**

หลายทีมสร้างระบบเก็บสถิติผลการรัน test ย้อนหลัง (เช่นเก็บผลลัพธ์ทุก pipeline run ลง database) แล้วคำนวณ "flakiness score" ของแต่ละ test case โดยอัตโนมัติ ถ้า test ไหนมีอัตรา fail/pass สลับกันเกินเกณฑ์ที่ตั้งไว้ (เช่น fail มากกว่า 5% ของการรันทั้งหมดโดยไม่มีการแก้โค้ดที่เกี่ยวข้อง) ระบบจะแจ้งเตือนทีมโดยอัตโนมัติหรือ auto-quarantine ให้เลย

### 728.5 หลักการป้องกัน Flaky Test ตั้งแต่ต้น (ดีที่สุด)

- **หลีกเลี่ยง fixed `sleep()`** เปลี่ยนไปใช้ explicit wait ที่รอ condition จริง เช่น `await page.waitForSelector(...)` แทน `await sleep(3000)`
- **ทำให้แต่ละ test case เป็นอิสระจากกันอย่างสมบูรณ์** (test isolation) — ล้างข้อมูลก่อนและหลังทุกเทส ไม่พึ่งพาลำดับการรัน
- **Mock external service ที่ไม่เสถียรเสมอ** สำหรับ unit และ integration test อย่าให้เทสยิง API ภายนอกจริงถ้าไม่จำเป็น
- **Fix seed ของข้อมูลสุ่มเสมอ** เพื่อให้ test reproducible

---

## Step 729: Quality Gates — บล็อกการ Merge ด้วยเกณฑ์คุณภาพ

### 729.1 Quality Gate คืออะไร

**Quality Gate** คือ**ด่านตรวจสอบอัตโนมัติ**ที่กำหนดเกณฑ์ขั้นต่ำด้านคุณภาพไว้ล่วงหน้า แล้ว**บล็อกไม่ให้ merge โค้ดเข้า branch หลักได้** หากไม่ผ่านเกณฑ์นั้น ไม่ว่าจะเป็นเรื่อง test coverage, จำนวน test ที่ผ่าน, หรือ code quality metric อื่น ๆ

แนวคิดนี้เปลี่ยนคำแนะนำเรื่องคุณภาพจาก **"ขอความร่วมมือ"** ให้กลายเป็น **"กฎที่บังคับใช้จริงโดยระบบ"** — ไม่ว่านักพัฒนาจะตั้งใจหรือไม่ตั้งใจ ถ้าไม่ผ่านเกณฑ์ ระบบจะไม่ยอมให้ merge เข้า branch หลักโดยเด็ดขาด

### 729.2 Quality Gate ด้วย Coverage Threshold — GitHub Actions

วิธีที่ตรงไปตรงมาที่สุดคือตั้งค่า minimum coverage ไว้ใน config ของ test runner เอง แล้วให้ command exit ด้วย error code ถ้าไม่ผ่าน:

`jest.config.js`:
```javascript
module.exports = {
  collectCoverage: true,
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    // ตั้งเกณฑ์เข้มกว่าสำหรับไฟล์ที่สำคัญมาก เช่น payment
    "./src/payment/**/*.js": {
      branches: 95,
      functions: 95,
      lines: 95,
      statements: 95,
    },
  },
};
```

เมื่อรัน `jest --coverage` แล้ว coverage ต่ำกว่าเกณฑ์ที่ตั้งไว้ Jest จะ**exit ด้วย error code ที่ไม่ใช่ 0 โดยอัตโนมัติ** ทำให้ job ใน pipeline fail ทันที และ GitHub จะแสดงสถานะ ❌ บน Pull Request

```yaml
name: Quality Gate

on:
  pull_request:
    branches: [main]

jobs:
  quality-gate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
      - run: npm ci
      - name: Run tests and enforce coverage threshold
        run: npm test -- --coverage   # จะ fail อัตโนมัติถ้าต่ำกว่า threshold ใน jest.config.js
```

จากนั้นไปตั้งค่าที่ **Settings → Branches → Branch protection rules** ของ repository เลือก branch `main` แล้วเปิด **"Require status checks to pass before merging"** พร้อมเลือก job `quality-gate` เป็น required check — เมื่อตั้งค่านี้แล้ว ปุ่ม "Merge" บน Pull Request จะถูกล็อกไว้จนกว่า check นี้จะผ่านสีเขียว ไม่ว่าใครจะพยายาม merge ด้วยวิธีไหนก็ตาม (นอกจาก admin ที่ตั้งค่า bypass ไว้เท่านั้น)

### 729.3 Quality Gate ด้วย Coverage Threshold — GitLab CI/CD

pytest มี plugin `pytest-cov` ที่รองรับ `--cov-fail-under` ในตัว:

```yaml
stages:
  - test

quality-gate:
  stage: test
  image: python:3.12-slim
  before_script:
    - pip install --no-cache-dir -r requirements-dev.txt
  script:
    # ถ้า coverage รวมต่ำกว่า 80% คำสั่งนี้จะ exit ด้วย error code != 0 ทันที
    - pytest --cov=app --cov-report=term --cov-report=xml --cov-fail-under=80 tests/
  coverage: '/TOTAL.+ ([0-9]{1,3}%)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
```

จากนั้นไปที่ **Settings → Merge requests** ของ project แล้วเปิดใช้งาน **"Pipelines must succeed"** ภายใต้หัวข้อ Merge checks — เมื่อเปิดแล้ว ปุ่ม "Merge" จะถูกปิดใช้งานถ้า pipeline ยังไม่ผ่าน (รวมถึง job `quality-gate` ที่เราสร้างไว้)

GitLab ยังมีฟีเจอร์ขั้นสูงกว่านั้นคือ **"Merge Request Approval Rules"** ที่สามารถกำหนดเงื่อนไขซับซ้อนกว่าได้ เช่น ต้องมี code coverage ไม่ลดลงจากเดิม (ไม่ใช่แค่เกินเกณฑ์คงที่) โดยใช้ **coverage_check** approval rule ร่วมกับ GitLab Premium/Ultimate

### 729.4 Quality Gate แบบ "Coverage ต้องไม่ลดลง" (Diff Coverage)

เกณฑ์ % coverage รวมทั้งโปรเจกต์อย่างเดียวมีจุดอ่อน คือโปรเจกต์เก่าที่มี coverage ต่ำอยู่แล้วอาจไม่มีทางไปถึง 80% ได้เร็ว ๆ นี้ วิธีที่ทีมมืออาชีพนิยมใช้แทนคือ **Diff Coverage** — ตรวจสอบเฉพาะโค้ดที่**เปลี่ยนแปลงใหม่ใน PR/MR นี้เท่านั้น** ต้องมี coverage สูงกว่าเกณฑ์ (เช่น 90%) แม้ว่า coverage รวมของทั้งโปรเจกต์จะยังต่ำอยู่ก็ตาม

ตัวอย่างด้วยเครื่องมือ `diff-cover` (Python):

```yaml
diff-coverage-check:
  stage: test
  image: python:3.12-slim
  script:
    - pip install diff-cover pytest pytest-cov
    - pytest --cov=app --cov-report=xml tests/
    - diff-cover coverage.xml --compare-branch=origin/main --fail-under=90
```

คำสั่งนี้จะเทียบเฉพาะบรรทัดที่เปลี่ยนไปจาก `main` เท่านั้น แล้วบังคับว่าบรรทัดใหม่เหล่านั้นต้องถูกทดสอบครอบคลุมอย่างน้อย 90% วิธีนี้ยุติธรรมกว่าและปฏิบัติได้จริงมากกว่าสำหรับ legacy project

### 729.5 Quality Gate อื่น ๆ ที่นิยมใช้ร่วมกัน

Quality Gate ไม่ได้จำกัดแค่เรื่อง test coverage เท่านั้น ทีมมักผสมเกณฑ์หลายอย่างเข้าด้วยกัน:

| เกณฑ์ | เครื่องมือที่นิยมใช้ตรวจ |
|---|---|
| Test coverage ขั้นต่ำ | Jest coverageThreshold, pytest-cov --cov-fail-under |
| ต้องไม่มี test fail แม้แต่เคสเดียว | test runner exit code |
| Static code analysis / code smell | SonarQube Quality Gate |
| ช่องโหว่ด้านความปลอดภัย (Security) | Snyk, Trivy, npm audit |
| Lint ต้องผ่านไม่มี error | ESLint, Pylint |
| Bundle size ต้องไม่เกินขนาดที่กำหนด | bundlesize, size-limit |

ตัวอย่างการรวม Quality Gate หลายอย่างไว้ใน job เดียวบน GitLab:

```yaml
quality-gate-combined:
  stage: test
  image: node:20-slim
  script:
    - npm ci
    - npm run lint                              # ต้องไม่มี lint error
    - npm test -- --coverage --ci                # ต้องผ่าน coverageThreshold ใน jest.config.js
    - npm audit --audit-level=high                # ต้องไม่มีช่องโหว่ระดับ high ขึ้นไป
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

ถ้าคำสั่งใดคำสั่งหนึ่งใน `script` fail (exit code != 0) job ทั้งหมดจะ fail ทันที ทำให้ Quality Gate ทำหน้าที่เป็น**ด่านเดียวที่รวมเกณฑ์ทุกอย่างเข้าไว้ด้วยกัน**

### 729.6 ปรัชญาสำคัญของ Quality Gate

> **Quality Gate ที่ดีไม่ควรเข้มงวดจนทีมรู้สึกว่าเป็นอุปสรรคจนหาทางเลี่ยง (เช่น force-merge, ปิด branch protection ชั่วคราว) แต่ควรตั้งเกณฑ์ที่ทีมยอมรับร่วมกันว่าเป็นมาตรฐานขั้นต่ำที่สมเหตุสมผล แล้วค่อย ๆ ปรับเกณฑ์ให้สูงขึ้นทีละน้อยตามความเป็นจริงของโปรเจกต์**

การตั้งเกณฑ์ 100% coverage ตั้งแต่วันแรกสำหรับ legacy project ขนาดใหญ่มักจบลงด้วยการที่ทีมหาทาง bypass Quality Gate แทนที่จะแก้ปัญหาจริง ควรเริ่มจากเกณฑ์ที่ทำได้จริง (เช่น "ห้ามให้ coverage ลดลงจากเดิม") แล้วค่อย ๆ ไต่ระดับขึ้นไปตามเวลา

---

## Step 730: แบบฝึกหัด — เพิ่ม Unit Test พร้อม Coverage Report เข้า Pipeline จริง

ถึงเวลาลงมือปฏิบัติจริงแล้ว แบบฝึกหัดนี้จะให้คุณนำทุกสิ่งที่เรียนมาใน Part นี้มาประกอบร่างเป็น pipeline ที่ใช้งานได้จริง คุณสามารถเลือกทำได้ทั้งบน **GitHub Actions** หรือ **GitLab CI/CD** ตามแพลตฟอร์มที่ทีมของคุณใช้งานอยู่

### โจทย์

สมมติคุณมีโปรเจกต์ Node.js ที่มี pipeline พื้นฐานอยู่แล้ว (build + lint) จาก Part ก่อนหน้า แต่ยังไม่มี automated test เลย ให้คุณ:

1. เขียนฟังก์ชันใหม่อย่างน้อย 1 ฟังก์ชันที่มี logic ให้ทดสอบได้จริง (ไม่ใช่แค่ `return true`)
2. เขียน unit test ให้ครอบคลุมฟังก์ชันนั้นอย่างน้อย 5 เทสเคส (รวม happy path และ edge case)
3. ตั้งค่า coverage threshold ขั้นต่ำ 70% ในไฟล์ config ของ test framework
4. เพิ่ม job ใหม่เข้าไปใน pipeline ที่มีอยู่ ให้รัน unit test พร้อม coverage ทุกครั้งที่มีการเปิด Pull Request/Merge Request
5. ตั้งค่าให้ test report แสดงผลใน UI ของแพลตฟอร์มที่เลือก
6. ตั้งค่า branch protection / merge request rule ให้บล็อกการ merge ถ้า job นี้ fail

### ขั้นตอนแนะนำ (ตัวอย่างด้วย GitHub Actions + Jest)

**ขั้นที่ 1:** สร้างฟังก์ชันตัวอย่างที่ `src/passwordValidator.js`:

```javascript
// src/passwordValidator.js
function validatePassword(password) {
  const errors = [];

  if (password.length < 8) {
    errors.push("รหัสผ่านต้องมีความยาวอย่างน้อย 8 ตัวอักษร");
  }
  if (!/[A-Z]/.test(password)) {
    errors.push("รหัสผ่านต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว");
  }
  if (!/[0-9]/.test(password)) {
    errors.push("รหัสผ่านต้องมีตัวเลขอย่างน้อย 1 ตัว");
  }
  if (!/[!@#$%^&*]/.test(password)) {
    errors.push("รหัสผ่านต้องมีอักขระพิเศษอย่างน้อย 1 ตัว (!@#$%^&*)");
  }

  return {
    isValid: errors.length === 0,
    errors,
  };
}

module.exports = { validatePassword };
```

**ขั้นที่ 2:** เขียนเทสให้ครอบคลุมที่ `src/passwordValidator.test.js`:

```javascript
// src/passwordValidator.test.js
const { validatePassword } = require("./passwordValidator");

describe("validatePassword", () => {
  test("รหัสผ่านที่ถูกต้องครบทุกเงื่อนไขต้องผ่าน", () => {
    const result = validatePassword("Secure1!");
    expect(result.isValid).toBe(true);
    expect(result.errors).toHaveLength(0);
  });

  test("รหัสผ่านสั้นเกินไปต้องไม่ผ่าน", () => {
    const result = validatePassword("Ab1!");
    expect(result.isValid).toBe(false);
    expect(result.errors).toContain(
      "รหัสผ่านต้องมีความยาวอย่างน้อย 8 ตัวอักษร"
    );
  });

  test("รหัสผ่านที่ไม่มีตัวพิมพ์ใหญ่ต้องไม่ผ่าน", () => {
    const result = validatePassword("secure1!");
    expect(result.errors).toContain(
      "รหัสผ่านต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว"
    );
  });

  test("รหัสผ่านที่ไม่มีตัวเลขต้องไม่ผ่าน", () => {
    const result = validatePassword("SecurePass!");
    expect(result.errors).toContain("รหัสผ่านต้องมีตัวเลขอย่างน้อย 1 ตัว");
  });

  test("รหัสผ่านที่ไม่มีอักขระพิเศษต้องไม่ผ่าน", () => {
    const result = validatePassword("SecurePass1");
    expect(result.errors).toContain(
      "รหัสผ่านต้องมีอักขระพิเศษอย่างน้อย 1 ตัว (!@#$%^&*)"
    );
  });

  test("รหัสผ่านที่ผิดหลายเงื่อนไขต้องรวม error ทั้งหมด", () => {
    const result = validatePassword("ab");
    expect(result.isValid).toBe(false);
    expect(result.errors.length).toBeGreaterThanOrEqual(3);
  });
});
```

**ขั้นที่ 3:** ตั้ง coverage threshold ใน `jest.config.js`:

```javascript
module.exports = {
  collectCoverage: true,
  coverageReporters: ["text", "lcov", "json-summary"],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
};
```

**ขั้นที่ 4:** เพิ่ม job ใหม่เข้า workflow ที่มีอยู่ `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
      - run: npm ci
      - run: npm run lint

  # job ใหม่ที่เพิ่มเข้ามาสำหรับแบบฝึกหัดนี้
  unit-test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - run: npm ci

      - name: Run unit tests with coverage
        run: npx jest --ci --coverage --reporters=default --reporters=jest-junit
        env:
          JEST_JUNIT_OUTPUT_DIR: "./test-results"
          JEST_JUNIT_OUTPUT_NAME: "junit.xml"

      - name: Publish test report to PR
        uses: dorny/test-reporter@v1
        if: always()
        with:
          name: Unit Test Results
          path: "test-results/junit.xml"
          reporter: jest-junit

      - name: Show coverage summary
        if: always()
        run: |
          echo "## Coverage Summary" >> $GITHUB_STEP_SUMMARY
          node -e "
            const d = require('./coverage/coverage-summary.json').total;
            console.log('| Metric | Coverage |');
            console.log('|---|---|');
            console.log('| Lines | ' + d.lines.pct + '% |');
            console.log('| Branches | ' + d.branches.pct + '% |');
            console.log('| Functions | ' + d.functions.pct + '% |');
            console.log('| Statements | ' + d.statements.pct + '% |');
          " >> $GITHUB_STEP_SUMMARY

      - name: Upload coverage artifact
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
          retention-days: 14
```

**ขั้นที่ 5:** ไปที่ **Settings → Branches → Add branch protection rule** ของ repository ตั้งค่าให้ branch `main`:

- เปิด **"Require a pull request before merging"**
- เปิด **"Require status checks to pass before merging"** แล้วเลือก job `unit-test` และ `lint` เป็น required check
- (แนะนำ) เปิด **"Require branches to be up to date before merging"** เพื่อให้แน่ใจว่าเทสถูกรันกับโค้ดล่าสุดของ `main` เสมอ

### ขั้นตอนแนะนำ (ตัวอย่างด้วย GitLab CI/CD + pytest) — ทางเลือก

ถ้าทีมของคุณใช้ GitLab และ Python ให้ทำแบบเดียวกันนี้แทน:

```yaml
stages:
  - lint
  - test

lint:
  stage: lint
  image: python:3.12-slim
  script:
    - pip install pylint
    - pylint app/

unit-test:
  stage: test
  image: python:3.12-slim
  needs: ["lint"]
  before_script:
    - pip install --no-cache-dir pytest pytest-cov
  script:
    - pytest --cov=app --cov-report=term --cov-report=xml --cov-fail-under=70 --junitxml=report.xml tests/
  coverage: '/TOTAL.+ ([0-9]{1,3}%)/'
  artifacts:
    when: always
    reports:
      junit: report.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
    expire_in: 14 days
```

จากนั้นไปที่ **Settings → Merge requests** ของ project เปิด **"Pipelines must succeed"** เพื่อบล็อกการ merge ถ้า `unit-test` fail

### เกณฑ์ตรวจสอบว่าทำสำเร็จ

ให้ตรวจสอบตัวเองด้วยรายการนี้:

- [ ] เปิด Pull Request/Merge Request ทดสอบ แล้วเห็น job unit test รันอัตโนมัติ
- [ ] ลองแก้โค้ดให้ test fail โดยตั้งใจ (เช่น comment เงื่อนไขบางอันออก) แล้วเห็นว่า pipeline เปลี่ยนเป็นสีแดงและปุ่ม merge ถูกล็อก
- [ ] ลองลด coverage โดยตั้งใจ (เช่นลบเทสบางเคสออก) แล้วเห็นว่า job fail เพราะ coverage ต่ำกว่าเกณฑ์ที่ตั้งไว้ ไม่ใช่เพราะเทส fail
- [ ] เห็น test report แสดงผลใน UI ของแพลตฟอร์ม (Check บน GitHub PR หรือ tab Tests บน GitLab MR)
- [ ] แก้โค้ดกลับให้ถูกต้อง แล้วเห็นว่า pipeline กลับมาเป็นสีเขียว และปุ่ม merge เปิดใช้งานได้อีกครั้ง

### แบบฝึกหัดขยาย (ถ้ามีเวลา)

1. เพิ่ม integration test ที่เชื่อมต่อกับ database จริงผ่าน `services:` ตามที่เรียนใน Step 724
2. ลองแบ่ง unit test ให้รันแบบ parallel ด้วย matrix/parallel ตามที่เรียนใน Step 727 แล้ววัดเวลาที่ประหยัดได้จริง
3. จงใจสร้าง flaky test ขึ้นมา (เช่นใช้ `Math.random()` โดยไม่ fix seed) แล้วลองใช้เทคนิค retry และ quarantine ตามที่เรียนใน Step 728 มาจัดการมัน

---

## สรุป Part 73

ใน Part นี้เราได้เรียนรู้ว่า:

1. Automated Testing ใน pipeline สำคัญเพราะจับบั๊กได้เร็วตั้งแต่ก่อน merge ไม่ต้องพึ่งพาความจำของมนุษย์ ตามหลัก Shift Left Testing ที่ยิ่งพบบั๊กเร็วเท่าไหร่ ต้นทุนในการแก้ก็ยิ่งต่ำเท่านั้น
2. Test Pyramid บอกเราว่าควรมี Unit Test เยอะที่สุด (เร็ว เสถียร ถูก), Integration Test ปานกลาง, และ E2E Test น้อยที่สุด (ช้า เปราะบาง แพง) — การทำกลับด้าน (Ice Cream Cone) นำไปสู่ pipeline ที่ช้าและไม่น่าเชื่อถือ
3. Unit Test เขียนได้ด้วย Jest (JavaScript) หรือ pytest (Python) รันได้ทั้งบน GitHub Actions และ GitLab CI/CD ด้วยหลักการเดียวกัน
4. Integration Test ต้องมี service ประกอบ เช่น database ใช้ `services:` ของทั้งสองแพลตฟอร์มได้ โดยต้องระวังเรื่อง health check และการเข้าถึง service (localhost บน GitHub, alias บน GitLab)
5. E2E Test ทดสอบผ่าน UI จริงด้วย Playwright หรือ Cypress ต้อง start แอปให้พร้อมก่อนเสมอ และเก็บ screenshot/video เมื่อ fail
6. Test Report และ Coverage Report แสดงผลใน CI UI ได้ผ่าน `reports.junit`/`reports.coverage_report` ของ GitLab หรือ test reporter action และ `$GITHUB_STEP_SUMMARY` ของ GitHub Actions
7. Parallel Testing แบ่ง test suite รันขนานด้วย `parallel:` ของ GitLab หรือ `strategy.matrix` ของ GitHub Actions ช่วยลดเวลา pipeline ได้อย่างมีนัยสำคัญ
8. Flaky Test คือเทสที่ผลลัพธ์ไม่คงที่โดยไม่ได้แก้โค้ด จัดการได้ด้วย retry (ระยะสั้น) และ quarantine (พร้อมกำหนดเวลาต้องแก้จริง) แต่ต้องแก้ที่ต้นเหตุในระยะยาว
9. Quality Gate บล็อกการ merge ด้วยเกณฑ์คุณภาพที่ตั้งไว้ล่วงหน้า เช่น coverage threshold ผ่าน `coverageThreshold` ของ Jest หรือ `--cov-fail-under` ของ pytest ร่วมกับ branch protection / merge request rule ของแพลตฟอร์ม
10. เราได้ลงมือปฏิบัติจริงด้วยการเพิ่ม unit test พร้อม coverage report เข้า pipeline ที่มีอยู่ ครบตั้งแต่เขียนเทส ตั้งเกณฑ์ coverage ไปจนถึงบล็อกการ merge

### Checklist ก่อนไป Part ถัดไป

- [ ] เข้าใจว่าทำไม automated testing ต้องอยู่ใน pipeline ไม่ใช่แค่ทำแยกต่างหาก
- [ ] อธิบาย Test Pyramid ได้ และรู้ว่าทำไมต้องมี unit test เยอะที่สุด
- [ ] เขียนและรัน unit test ด้วย Jest หรือ pytest ใน pipeline ได้ทั้งสองแพลตฟอร์ม
- [ ] ตั้งค่า `services:` เพื่อรัน integration test กับ database จริงได้
- [ ] เขียน E2E test พื้นฐานด้วย Playwright หรือ Cypress และรันใน pipeline ได้
- [ ] ตั้งค่าให้ test report และ coverage report แสดงผลใน UI ของแพลตฟอร์มได้
- [ ] แบ่ง test suite รันแบบ parallel ได้ทั้งด้วย `parallel:` และ `matrix`
- [ ] รู้จักสาเหตุของ flaky test และวิธีจัดการด้วย retry/quarantine
- [ ] ตั้ง Quality Gate ด้วย coverage threshold ที่บล็อกการ merge ได้จริง
- [ ] ทำแบบฝึกหัดจริง เพิ่ม unit test พร้อม coverage เข้า pipeline สำเร็จ

**ต่อไป:** [Part 74: Deploy อัตโนมัติ: Docker, Kubernetes, Cloud](./part-074-deploy-อัตโนมัติ.md)
