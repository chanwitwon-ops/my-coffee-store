# Technical Design: ระบบสั่งอาหารด้วยตนเองจากโต๊ะ (Dine-in Self-Order)

- **วันที่:** 2026-08-03
- **สถานะ:** Draft
- **อ้างอิง requirement:** [[../../01-requirements/01-spec/20260803-01-dine-in-self-order|20260803-01-dine-in-self-order]]
- **อ้างอิง prototype:** [[../01-prototypes/20260803-02-dine-in-self-order|01-prototypes]]

## Data model

| entity | field | type | หมายเหตุ |
|---|---|---|---|
| `table` | `id`, `label`, `qr_token` | string/enum | `qr_token` คือค่าที่เข้ารหัสอยู่ใน QR แต่ละโต๊ะ ใช้ resolve กลับเป็น `table_id` |
| `order` | `id`, `table_id`, `status`, `payment_status`, `created_at` | enum/timestamp | `status`: `pending`, `in_progress`, `done`; `payment_status`: `unpaid`, `paid` (default `unpaid`) |
| `order` | `created_by` | enum | `customer` หรือ `staff` (ใช้แยกออร์เดอร์ที่เกิดจาก fallback) |
| `order_item` | `id`, `order_id`, `menu_id`, `qty`, `options` | — | `options` เก็บตัวเลือกปรับแต่ง (ไซส์, ความหวาน ฯลฯ) |

- **ออร์เดอร์ผูกกับโต๊ะ ไม่ผูกกับลูกค้ารายคน:** ทุกอุปกรณ์ที่เปิดหน้าเว็บด้วย `qr_token` เดียวกันของโต๊ะที่ยังไม่ยืนยันออร์เดอร์ ให้เห็น/แก้ไข "ตะกร้าปัจจุบันของโต๊ะนั้น" ชุดเดียวกัน (shared cart) — implement ด้วย session ที่ผูกกับ `table_id` บวก real-time sync (เช่น websocket/subscription) ไม่ใช่ local state ต่ออุปกรณ์
- ไม่มี field ที่เกี่ยวกับ payment gateway/transaction ใดๆ ในเวอร์ชันนี้ — `payment_status` เป็นแค่ flag ที่พนักงานกดเปลี่ยนเอง

## API contract (แนวทาง)

- `GET /tables/{qr_token}` — resolve QR เป็นโต๊ะ + คืนตะกร้าปัจจุบันของโต๊ะ (ถ้ามี)
- `POST /tables/{table_id}/orders` — ยืนยันออร์เดอร์ของโต๊ะ (ไม่มี field การจ่ายเงินใดๆ ในการเรียกนี้)
- `POST /staff/orders` — พนักงานสร้างออร์เดอร์แทนลูกค้า ระบุ `table_id` เอง, `created_by = staff`
- `PATCH /orders/{id}/status` — พนักงานอัปเดตสถานะออร์เดอร์
- `PATCH /orders/{id}/payment_status` — พนักงานกดสลับสถานะจ่ายเงิน (`unpaid` ↔ `paid`)
- realtime channel/subscription ต่อ:
  - คิวออร์เดอร์ฝั่งพนักงาน (ทุกออร์เดอร์ทุกโต๊ะ)
  - ตะกร้า/สถานะออร์เดอร์ของโต๊ะเดียว (ฝั่งลูกค้า เห็นเฉพาะโต๊ะตัวเอง)
- `GET /manager/daily-summary` — สรุปยอดขายวันนี้ (เฉพาะ role ผู้จัดการ): ยอดขายรวม, จำนวนออร์เดอร์รวม, จำนวนออร์เดอร์แยกตาม `status`, จำนวนออร์เดอร์แยกตาม `payment_status` — คำนวณจากออร์เดอร์ของวันปัจจุบันเท่านั้น ไม่มี query ช่วงวันที่/ตัวกรองอื่น

## Authorization

- แยกสิทธิ์ตาม role: `barista` เข้าถึงคิวออร์เดอร์ + สถานะจ่ายเงินได้ (อ่าน/แก้ไข), เข้าถึง `GET /manager/daily-summary` ไม่ได้ (403)
- `manager` เข้าถึงได้ทุกอย่างที่ `barista` เข้าถึงได้ บวก `GET /manager/daily-summary`

## Validation

- `POST /tables/{table_id}/orders` ต้องมี `order_item` อย่างน้อย 1 รายการ
- `qr_token` ที่ resolve ไม่เจอโต๊ะ → แจ้งลูกค้าว่า QR ไม่ถูกต้อง พร้อมข้อความให้เรียกพนักงาน (ไม่ auto fallback ให้พนักงานเอง ต้องให้ลูกค้าเรียก)
- `PATCH .../payment_status` และ `PATCH .../status` ทำได้จาก role พนักงานเท่านั้น ลูกค้าไม่มีสิทธิ์เรียก endpoint เหล่านี้

## หมายเหตุ

- ยังไม่ได้เลือกเทคโนโลยี/สแต็กของระบบ (repo นี้ยังไม่มี source code) เอกสารนี้เป็น data contract เชิงตรรกะ รอ mapping เข้ากับ schema จริงตอนเริ่ม implement
- ไม่รวมการชำระเงินผ่านระบบ/payment gateway และไม่รวมรายงาน/analytics เชิงลึก ตามที่ระบุไว้ใน scope ของ requirement ต้นทาง
