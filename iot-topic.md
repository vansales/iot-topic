# 📡 IoT Topic Specification: Locker System (32-Channel)

IoT Topic และ Payload สำหรับระบบ Locker ที่ควบคุมด้วย MOSFET 32 ช่อง พร้อมหน้าจอแสดงผลและระบบ QR Payment

---

## ✅ 1. Heartbeat Check

### 📌 Topic

```
device/{machine_id}/status
```

* ใช้สำหรับส่ง heartbeat ทุก 5 นาที จากอุปกรณ์มายัง Cloud
* ใช้เพื่อตรวจสอบว่าเครื่องยังออนไลน์อยู่หรือไม่

### ⏱ Publish Frequency

Every 5 minutes

### 🔁 Direction

Device → Cloud

### 🧾 Sample Payload

```json
{
  "machine_id": "mac_31287",
  "timestamp": 1702817270,
  "status": "online",
  "version": "1.0.0"
}
```
> หมายเหตุ: ค่า status: "online" เป็นเพียงค่าที่บอกว่าอุปกรณ์ยังทำงานอยู่ในขณะส่งข้อความเท่านั้น หากไม่มีการส่ง heartbeat ภายในเวลาที่กำหนด (เช่น 5 นาที) ระบบฝั่ง Cloud ควรตีความว่าอุปกรณ์อาจ "offline" โดยอิงจากการไม่มีข้อความเข้ามา ไม่ใช่จากค่า status นี้
---

## 🛠 2. Command (Cloud → Device)

### 📌 Topic

```
device/{machine_id}/cmd
```

* ใช้สำหรับส่งคำสั่งจาก Cloud ไปยัง Locker เช่น แสดง QR หรือ ปลดล็อกช่อง

### 🔁 Direction

Cloud → Device

### 🧾 Sample Payloads

#### 🔓 ปลดล็อกช่อง
ส่งคำสั่งปลดล็อกช่อง Locker

```json
{
  "id": "9ac2f7d0-d053-4a3f-bb6c-2cfde3451aa7",
  "machine_id": "mac_31287",
  "command": "open",
  "params": {
    "locker": 16 
    "order_no": "ORD-20250622-0001",
  }
}
```

#### ชำระเงินสำเร็จ
ส่งคำสั่งปลดล็อกช่อง Locker

```json
{
  "id": "9ac2f7d0-d053-4a3f-bb6c-2cfde3451aa7",
  "machine_id": "mac_31287",
  "command": "payment_received",
  "params": {
    "amount": 65.50 
    "order_no": "ORD-20250622-0001",
  }
}
```

#### ปรับราคาช่องจำหน่ายสินค้า
ส่งคำสั่งปรับราคาสินค้าแต่ละตู้

```json
{
  "id": "9ac2f7d0-d053-4a3f-bb6c-2cfde3451aa7",
  "machine_id": "mac_31287",
  "command": "update_price",
  "params": {}
}
```

#### 💳 แสดง QR Code

```json
{
  "id": "63e77a8a-36d4-44fd-865c-5584b7de0917",
  "machine_id": "mac_31287",
  "command": "display_qr",
  "params": {
    "qr_base64": "iVBORw0KGgoAAAANSUhEUgAA..."
    "order_no": "ORD-20250622-0001",
  }
}
```

#### 📝 แสดงข้อความ

```json
{
  "id": "7f13c9f4-55b6-4c7f-b269-4814b0701cd5",
  "machine_id": "mac_31287",
  "command": "msg_display",
  "params": {
    "message": "ขอบคุณที่ใช้บริการ"
  }
}
```

#### ♻️ เติมสินค้า (refill ทุกช่อง)

```json
{
  "id": "e7b455c2-3a8f-4a33-8872-267317705157",
  "machine_id": "mac_31287",
  "command": "refill_all"
}
```

#### Reset

```json
{
  "id": "e7b455c2-3a8f-4a33-8872-267317705157",
  "machine_id": "mac_31287",
  "command": "reset"
}
```

#### Reset

```json
{
  "id": "e7b455c2-3a8f-4a33-8872-267317705157",
  "machine_id": "mac_31287",
  "command": "restart"
}
```

---

## 📬 3. Action Response (Device → Cloud)

### 📌 Topic

```
device/{machine_id}/act
```

* ตอบกลับหลังได้รับคำสั่งจาก Cloud เช่น ผลลัพธ์ของการเปิดล็อกหรือแสดง QR

### 🔁 Direction

Device → Cloud

### 🧾 Sample Payload

Command ที่ได้รับจาก Server ส่งกลับเพื่อยืนยัน ว่าได้รบและทำงานแล้วถูกต้อง

```json
{
  "id": "9ac2f7d0-d053-4a3f-bb6c-2cfde3451aa7",
  "machine_id": "mac_31287",
  "command": "open", 
  "timestamp": 1702817285,
  "result": "success"
}
```

---

## 🧠 4. User Event (Device → Cloud)

### 📌 Topic

```
device/{machine_id}/event
```

* แจ้ง event ที่เกิดจากผู้ใช้งาน เช่น เลือกสินค้า เปิด/ปิดประตู หรือกรณีผิดปกติ

### 🔁 Direction

Device → Cloud

### 🧾 Sample Payloads

#### 🛒 เริ่มคำสั่งซื้อ

```json
{
  "event": "order",
  "machine_id": "mac_31287",
  "timestamp": 1702817300,
  "parameters": {
    "pack_type": "6_bottle_pack",
    "quantity": 2,
    "size": "1.5L"
  },
  "amount": 999
}
```


#### 🔐 ประตูถูกปิด
```json
{
  "event": "locker_closed",
  "machine_id": "mac_31287",
  "locker": 16,
  "timestamp": 1702817310
}
````

#### 🔓 ประตูถูกเปิด
```json
{
  "event": "locker_open",
  "machine_id": "mac_31287",
  "locker": 16,
  "timestamp": 1702817320
}
````

#### ⚠️ ไม่ชำระเงินในเวลาที่กำหนด

```json
{
  "event": "payment_timeout",
  "machine_id": "mac_31287",
  "timestamp": 1702817400
}
```

#### ❌ เกิดข้อผิดพลาดขณะเปิดล็อกเกอร์

```json
{
  "event": "error",
  "machine_id": "mac_31287",
  "code": "locker_jam",
  "locker": 16,
  "message": "Cannot open locker",
  "timestamp": 1702817450
}
```

---

## 📋 Summary of Topics

| Topic                        | Direction      | Description                    |
| ---------------------------- | -------------- | ------------------------------ |
| `device/{machine_id}/status` | Device → Cloud | Heartbeat ทุก 5 นาที           |
| `device/{machine_id}/cmd`    | Cloud → Device | ส่งคำสั่งไปยังเครื่อง          |
| `device/{machine_id}/act`    | Device → Cloud | ตอบกลับเมื่อทำคำสั่งสำเร็จ     |
| `device/{machine_id}/event`  | Device → Cloud | แจ้ง Event ที่เกิดจากผู้ใช้งาน |

---

## 🧠 Best Practices

* ใส่ `machine_id` ในทุก payload สำหรับ trace และการจัดการที่แม่นยำ
* ใช้ `uuid` สำหรับ `id` เพื่อป้องกันการซ้ำและรองรับ distributed system
* ใช้ `result` แทน `status` ใน response เพื่อไม่สับสนกับสถานะเครื่อง
* ใช้ timestamp แบบ UNIX Epoch (10 หลัก) หรือ ISO 8601 ตามความเหมาะสม
* ใช้ `snake_case` สำหรับชื่อคำสั่งและ event เพื่อความสม่ำเสมอ
* ใช้ `version` เฉพาะใน `status` เพื่อตรวจสอบ firmware/device version โดยรวม
* เวลา `timestamp` เป็นแบบ `UTC` เพื่อให้รองรับการ scale ไปยังหลายประเทศ
