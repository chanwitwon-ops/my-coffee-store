# Technical Design: ตัวเลือกระดับความหวานของเครื่องดื่ม

- **วันที่:** 2026-08-03
- **สถานะ:** Draft
- **อ้างอิง requirement:** [[../../01-requirements/01-spec/20260803-02-drink-sweetness-level|20260803-02-drink-sweetness-level]]
- **อ้างอิง prototype:** [[../01-prototypes/20260803-01-drink-sweetness-level|01-prototypes]]

## Data model

เพิ่ม field ใหม่ในรายการสั่งซื้อ (order item):

| field | type | ค่าที่เป็นไปได้ | default |
|---|---|---|---|
| `sweetness_level` | enum | `none`, `less`, `normal`, `more` | `normal` |

- field นี้ผูกกับ order item ไม่ใช่กับตัวเมนู เพราะลูกค้าเลือกความหวานต่อออเดอร์
- เมนูแต่ละตัวต้องมี flag `is_sweetness_adjustable: boolean` เพื่อกำหนดว่าเมนูนี้แสดงตัวเลือกความหวานหรือไม่

## API contract (แนวทาง)

- endpoint สร้างรายการสั่งซื้อ: แต่ละ order item รับ field `sweetness_level` เพิ่มเติม (optional, backend ใส่ค่า default `normal` ถ้าไม่ส่งมา)
- endpoint/สตรีมที่บาริสต้าใช้ดูใบสั่ง: ต้องคืนค่า `sweetness_level` เป็น label ที่อ่านง่าย เพื่อแสดงในใบสั่ง

## Validation

- ถ้าเมนูที่ `is_sweetness_adjustable = false` แต่มีการส่ง `sweetness_level` มา ให้ backend เพิกเฉย/ไม่บันทึกค่านั้น (ไม่ error)
- ค่า `sweetness_level` ต้องอยู่ใน enum ที่กำหนดเท่านั้น

## หมายเหตุ

- ยังไม่ได้เลือกเทคโนโลยี/สแต็กของระบบ (repo นี้ยังไม่มี source code) เอกสารนี้เป็น data contract เชิงตรรกะ รอ mapping เข้ากับ schema จริงตอนเริ่ม implement
- ไม่รวมเรื่องราคาต่างกันตามระดับความหวาน ตามที่ระบุไว้ใน scope ของ requirement ต้นทาง
