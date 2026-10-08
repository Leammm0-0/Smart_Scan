# Smart_Scan

Smart Scan คือระบบ Self-Checkout ที่ช่วยให้ผู้ใช้สามารถสแกนสินค้าด้วยตนเอง ตรวจสอบรายการสินค้า และชำระเงิน ผ่าน QR Code โดยมีเป้าหมายเพื่อลดขั้นตอนการชำระเงินและทำให้การซื้อสินค้าสะดวกและรวดเร็วยิ่งขึ้น

---

## 🖥️ ขั้นตอนการใช้งาน

```
Home  →  Scan  →  Review Order  →  Payment (QR)
```
 
| หน้า | หน้าที่ |
|---|---|
| **Home** | หน้าแรก เริ่มต้นใช้งาน |
| **Scan** | เปิดกล้องสแกนสินค้า พร้อมแสดงตะกร้าข้างๆ |
| **Review Order** | ตรวจสอบ / แก้ไขจำนวนสินค้า และดูยอดรวม |
| **Payment** | แสดง QR PromptPay ตามยอดที่ต้องจ่าย |
 
---

## 📷 กล้องใช้ไม่ได้?
 
หน้า Scan จะมีช่อง **"พิมพ์เลข Barcode แทนได้"** โผล่ขึ้นมาเอง
ให้พิมพ์เลขบาร์โค้ดจาก `products.csv` (คอลัมน์ที่ 2) แล้วกด Enter หรือปุ่ม **"เพิ่ม"** สินค้าจะเข้าตะกร้าเหมือนสแกนได้ ที่เหลือใช้งานได้ตามปกติ
 
---

## เทคโนโลยีที่ใช้
 
| ส่วน | เทคโนโลยี |
|---|---|
| ภาษา | Java 17 |
| หน้าจอ (GUI) | Java Swing |
| กล้อง | JavaCV 1.5.11 |
| อ่านบาร์โค้ด / สร้าง QR | ZXing 3.5.3 |
| Build | Maven |
| ข้อมูลสินค้า | ไฟล์ `products.csv` |
 
## โครงสร้างโปรเจกต์
 
```
products.csv     ข้อมูลสินค้า (id,barcode,name,price,description,stock,imagePath)
images/          รูปสินค้า
icon/            ไอคอนบนหน้าจอ
src/main/java/smartscan/
├── Main.java                  จุดเริ่มต้นโปรแกรม
├── model/                     ข้อมูล (ไม่ยุ่งกับหน้าจอ/กล้อง)
│   ├── Product.java           สินค้า
│   ├── CartItem.java          สินค้า 1 รายการในตะกร้า (สินค้า + จำนวน)
│   └── Cart.java              ตะกร้า (เพิ่ม/ลบ/แก้จำนวน/รวมราคา/ส่วนลด)
├── repository/
│   └── ProductRepository.java โหลด/ค้นหาสินค้าจาก products.csv
├── service/                   ขั้นตอนการทำงาน
│   ├── SmartScanner.java      barcode → หาสินค้า → ใส่ตะกร้า
│   ├── BarcodeDecoder.java    อ่าน barcode จากรูป (ZXing)
│   ├── CameraService.java     เปิดกล้อง + วนอ่านภาพ (JavaCV)
│   └── QRCodeGenerator.java   สร้าง QR PromptPay ตามยอดเงิน
└── ui/                        หน้าจอ (Swing)
    ├── MainFrame.java         หน้าต่างหลัก + สลับหน้า
    ├── HomePage.java          หน้า 1: Home
    ├── ScanPage.java          หน้า 2: Scan (กล้อง + ตะกร้า)
    ├── ReviewPage.java        หน้า 3: Review Order
    ├── PaymentPage.java       หน้า 4: Payment (QR)
    ├── CartItemRow.java       แถวสินค้า (ใช้ซ้ำในหน้า Scan และ Review)
    ├── CameraView.java        พื้นที่แสดงภาพกล้อง
    ├── Theme.java             สีและฟอนต์ทั้งแอป
    └── Ui.java                ตัวช่วยสร้างปุ่ม/กล่องโค้งมน/ป้าย
```
 
## 🔄 ลำดับการทำงานของระบบ
 
```
กล้อง (CameraService)
   │ ภาพ
   ▼
BarcodeDecoder ── barcode ──▶ ScanPage.handleBarcode
                                    │
                                    ▼
                         SmartScanner.scanBarcode
                                    │
                                    ▼
                    ProductRepository.findByBarcode
                                    │
                                    ▼
                              Cart.addItem
                                    │
                                    ▼
                    ScanPage.refresh() วาดตะกร้าใหม่
```
 
## 🔧 อยากแก้อะไร ไปที่ไหน
 
| อยากแก้ | ไฟล์ / จุดที่แก้ |
|---|---|
| เพิ่ม/แก้สินค้า | `products.csv` (+ รูปใน `images/`) |
| คอลัมน์ข้อมูลสินค้า | `Product.java` + `ProductRepository.loadFromCsv` |
| ส่วนลด / โปรโมชั่น | `Cart.getDiscount()` |
| เบอร์ PromptPay ที่รับเงิน | `QRCodeGenerator.PROMPTPAY_ID` |
| อายุ QR (5 นาที) | `PaymentPage.QR_LIFETIME_SECONDS` |
| เวลากันสแกนซ้ำ barcode เดิม | `ScanPage.SAME_BARCODE_DELAY_MS` |
| ความถี่อ่าน barcode | `CameraService.SCAN_INTERVAL_MS` |
| เลือกกล้อง (Mac) / ขนาดภาพ | ค่าคงที่บนสุดของ `CameraService.java` |
| ชนิด barcode ที่อ่านได้ | `BarcodeDecoder` (รายการ `formats`) |
| สีธีม / ฟอนต์ | `Theme.java` |
| หน้าตาปุ่ม / กล่อง | `Ui.java` |
| หน้าตาแถวสินค้า | `CartItemRow.java` |
| ข้อความ/ปุ่มของแต่ละหน้า | `HomePage` / `ScanPage` / `ReviewPage` / `PaymentPage` |
| ลำดับการเปลี่ยนหน้า | `MainFrame.showXxx()` |
 
---
 
## 👥 ผู้จัดทำ
 
| รหัสนักศึกษา | ชื่อ | หน้าที่ |
|---|---|---|
| 6821651221 | นาย ทรงกฤษ เหลี่ยมคุณ | Frontend |
| 6821651051 | นาย กรวิชญ์ มาตพรมราช | Backend |
| 6821651752 | นาย วิศิษฐ์กุล ห้วยหงษ์ทอง | Database |

---
 
## Position
 
**ทรงกฤษ เหลี่ยมคุณ (Frontend)**
- ออกแบบ UI หน้าสแกนสินค้า
- ออกแบบ UI หน้าตะกร้าสินค้า
- ออกแบบ UI หน้ายืนยันออเดอร์
- ออกแบบ UI หน้า Payment
- สร้างปุ่มและการเปลี่ยนหน้า
- เชื่อมการทำงานของแต่ละหน้าภายในระบบ


**กรวิชญ์ มาตพรมราช (Backend)**
- เขียน Logic การรับ Barcode
- เขียนระบบเพิ่ม/ลดสินค้า
- คำนวณราคาสินค้าและยอดรวม
- จัดการข้อมูล Order
- เขียน Logic การชำระเงิน
- เชื่อมการทำงานระหว่างแต่ละหน้า


**วิศิษฐ์กุล ห้วยหงษ์ทอง (Database)**
- ออกแบบโครงสร้างข้อมูลสินค้า
- จัดเก็บข้อมูล Barcode สินค้า
- จัดเก็บชื่อและราคาสินค้า
- จัดการการเพิ่ม/แก้ไข/ลบข้อมูล
- ดึงข้อมูลสินค้าเพื่อใช้งานในระบบ
- เชื่อมข้อมูลสินค้ากับ Backend
