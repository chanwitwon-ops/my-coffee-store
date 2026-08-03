# Technical Design: ระบบแจ้งเตือนวัตถุดิบใกล้หมดสต็อก (Low Stock Alert)

- **วันที่:** 2026-08-03
- **สถานะ:** Draft
- **อ้างอิง requirement:** [[../../01-requirements/01-spec/20260803-03-low-stock-alert|20260803-03-low-stock-alert]]
- **อ้างอิง prototype:** [[../01-prototypes/20260803-03-low-stock-alert|01-prototypes]]

## Data model

| entity | field | type | หมายเหตุ |
|---|---|---|---|
| `ingredient` | `id`, `name`, `unit`, `low_stock_threshold`, `current_quantity` | string/number | `low_stock_threshold` และ `current_quantity` เป็นหน่วยเดียวกับ `unit` ของแต่ละวัตถุดิบ (ไม่แปลงหน่วยข้ามกัน) |
| `ingredient` | `created_at`, `updated_at` | timestamp | ใช้ track ว่ากรอกจำนวนล่าสุดเมื่อไหร่ |

- ไม่มี entity แยกสำหรับ "การกรอกจำนวนคงเหลือ" เป็น log/history — เก็บแค่ค่าล่าสุดที่ `ingredient.current_quantity` (ตามขอบเขตที่ไม่รวม analytics/ประวัติการใช้วัตถุดิบ)
- สถานะ "ใกล้หมด" ไม่ต้องเก็บเป็น field แยก คำนวณสดจาก `current_quantity <= low_stock_threshold` ทุกครั้งที่ query

## API contract (แนวทาง)

- `GET /ingredients` — รายการวัตถุดิบทั้งหมด พร้อม `current_quantity`, `low_stock_threshold`, และ flag คำนวณ `is_low` (เข้าถึงได้ทั้ง barista/manager)
- `POST /ingredients` — เพิ่มวัตถุดิบใหม่ (`manager` เท่านั้น)
- `PATCH /ingredients/{id}` — แก้ไขชื่อ/หน่วย/เกณฑ์ขั้นต่ำ (`manager` เท่านั้น)
- `DELETE /ingredients/{id}` — ลบวัตถุดิบ (`manager` เท่านั้น)
- `PATCH /ingredients/{id}/quantity` — อัปเดตจำนวนคงเหลือ (`barista` และ `manager` เข้าถึงได้)
- `GET /ingredients/low-stock` — คืนเฉพาะรายการที่ `is_low = true` สำหรับ widget บน dashboard (เข้าถึงได้ทั้ง barista/manager)

## Authorization

- แก้ไข master list (`POST`/`PATCH`/`DELETE` บน `/ingredients`) — `manager` เท่านั้น, `barista` เรียกแล้วได้ 403
- อัปเดตจำนวนคงเหลือ (`PATCH .../quantity`) และอ่านข้อมูล (`GET`) — `barista` และ `manager` เข้าถึงได้เท่ากัน

## Validation

- `low_stock_threshold` และ `current_quantity` ต้องเป็นจำนวนที่ไม่ติดลบ
- ลบวัตถุดิบ (`DELETE`) ที่ถูกอ้างอิงอยู่ในเมนู/ออร์เดอร์อื่น (ถ้ามีการเชื่อมโยงในอนาคต) อยู่นอกขอบเขตของเอกสารนี้ — ตอนนี้ `ingredient` เป็น entity อิสระ ไม่ผูกกับเมนู/ออร์เดอร์

## หมายเหตุ

- ยังไม่ได้เลือกเทคโนโลยี/สแต็กของระบบ (repo นี้ยังไม่มี source code) เอกสารนี้เป็น data contract เชิงตรรกะ รอ mapping เข้ากับ schema จริงตอนเริ่ม implement
- ไม่รวมการตัดสต็อกอัตโนมัติจากออร์เดอร์, การแจ้งเตือนผ่านช่องทางภายนอก, และรายงานวิเคราะห์การใช้วัตถุดิบ ตามที่ระบุไว้ใน scope ของ requirement ต้นทาง
