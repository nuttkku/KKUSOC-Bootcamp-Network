# คู่มือการดาวน์โหลดและติดตั้ง Cisco Packet Tracer

> **ที่มาของข้อมูล**: คู่มือนี้อ้างอิงจากเอกสารทางการของ Cisco Networking Academy
> - [Cisco Packet Tracer Download and Installation Instructions](https://www.netacad.com/skillsforall/files/Cisco_Packet_Tracer_Download_and_Installation_Instructions.pdf) — Cisco NetAcad Official
> - [Cisco NetAcad Resource Hub](https://www.netacad.com/resources/lab-downloads) — หน้าดาวน์โหลดทางการ
> - [Download Cisco Packet Tracer 9.0](https://www.packettracernetwork.com/download/download-packet-tracer.html) — PacketTracerNetwork.com

---

## ข้อกำหนดสำคัญ

> **คำเตือน**: ดาวน์โหลด Cisco Packet Tracer จาก **เว็บไซต์ Cisco NetAcad เท่านั้น**
> ห้ามดาวน์โหลดจากแหล่งอื่น เว็บไซต์อื่น หรือ Torrent โดยเด็ดขาด

- ใช้งานได้ฟรีสำหรับนักศึกษาและอาจารย์
- ต้องมีบัญชี Cisco NetAcad ก่อนดาวน์โหลด
- ใช้เพื่อการศึกษาเท่านั้น ห้ามใช้เชิงพาณิชย์

---

## ระบบที่รองรับ (Packet Tracer 9.0.0)

| ระบบปฏิบัติการ | เวอร์ชันที่รองรับ |
|---------------|-----------------|
| **Windows** | Windows 10, Windows 11 (64-bit) |
| **macOS** | macOS 12 (Monterey) หรือใหม่กว่า |
| **Linux** | Ubuntu 22.04 LTS หรือ 24.04 LTS (64-bit) |

### ความต้องการขั้นต่ำของระบบ

| ส่วนประกอบ | ขั้นต่ำ |
|-----------|--------|
| **RAM** | 8 GB ขึ้นไป (แนะนำ) |
| **พื้นที่ดิสก์** | 1.4 GB ขึ้นไป |
| **สถาปัตยกรรม** | 64-bit เท่านั้น |

---

## ขั้นตอนที่ 1 — สมัครบัญชี Cisco NetAcad

1. เปิดเบราว์เซอร์แล้วไปที่ **[https://www.netacad.com](https://www.netacad.com)**

2. คลิกปุ่ม **"Sign Up"** หรือ **"Create Account"** มุมขวาบน

3. กรอกข้อมูล:
   - **Email**: ใช้อีเมล์มหาวิทยาลัยหรืออีเมล์ส่วนตัว
   - **Password**: กำหนดรหัสผ่านที่ปลอดภัย
   - **ชื่อ-นามสกุล**: กรอกให้ถูกต้อง

4. ตรวจสอบอีเมล์แล้วคลิก **Verify Email** เพื่อยืนยันบัญชี

> หากมีบัญชีอยู่แล้ว ข้ามไปขั้นตอนที่ 2 ได้เลย

---

## ขั้นตอนที่ 2 — ลงทะเบียนหลักสูตรฟรี

ต้องลงทะเบียนหลักสูตรก่อนจึงจะดาวน์โหลดได้

1. ไปที่ **[https://www.netacad.com/courses/getting-started-cisco-packet-tracer](https://www.netacad.com/courses/getting-started-cisco-packet-tracer)**

2. คลิก **"Get Started"** หรือ **"Enroll"**

3. หลักสูตรนี้:
   - ชื่อ: **Getting Started with Cisco Packet Tracer**
   - ค่าใช้จ่าย: **ฟรี**
   - เนื้อหา: พื้นฐานการใช้งาน Packet Tracer

4. หลังจาก Enroll สำเร็จ ระบบจะให้สิทธิ์ดาวน์โหลดโปรแกรมโดยอัตโนมัติ

---

## ขั้นตอนที่ 3 — ดาวน์โหลด Cisco Packet Tracer

1. Login เข้าระบบที่ **[https://www.netacad.com](https://www.netacad.com)**
   - สามารถ Login ด้วย Google account ได้เช่นกัน

2. ไปที่ **Resource Hub**:
   **[https://www.netacad.com/resources/lab-downloads](https://www.netacad.com/resources/lab-downloads)**

3. เลือกเวอร์ชันที่ตรงกับระบบปฏิบัติการของท่าน:

   | ระบบปฏิบัติการ | ไฟล์ที่ได้รับ |
   |---------------|-------------|
   | Windows | `Packet_Tracer900_64bit_setup_signed.exe` |
   | macOS | `Packet_Tracer900_mac.dmg` |
   | Ubuntu Linux | `Packet_Tracer900_amd64_signed.deb` |

4. คลิกดาวน์โหลด แล้วรอจนไฟล์โหลดเสร็จ

> **ตรวจสอบความถูกต้อง**: Cisco จะให้ SHA256 Checksum ไว้ตรวจสอบว่าไฟล์ที่ได้รับไม่ถูกแก้ไข

---

## ขั้นตอนที่ 4 — ติดตั้งโปรแกรม

### Windows

1. ดับเบิลคลิกไฟล์ `.exe` ที่ดาวน์โหลดมา
2. ถ้ามีหน้าต่าง User Account Control (UAC) ขึ้นมา คลิก **"Yes"**
3. อ่านและยอมรับ **License Agreement** แล้วคลิก **"I Agree"**
4. เลือก Folder ที่ต้องการติดตั้ง (ค่าเริ่มต้นแนะนำ)
5. คลิก **"Install"** แล้วรอจนเสร็จ
6. คลิก **"Finish"**

### macOS

1. เปิดไฟล์ `.dmg`
2. ลากไอคอน Packet Tracer ไปวางใน **Applications**
3. เปิดโปรแกรมจาก Applications หรือ Spotlight

### Ubuntu Linux

```bash
# ติดตั้งจาก .deb package
sudo dpkg -i Packet_Tracer900_amd64_signed.deb

# ถ้ามี dependency error
sudo apt-get install -f
```

---

## ขั้นตอนที่ 5 — เปิดใช้งานครั้งแรก

1. เปิดโปรแกรม **Cisco Packet Tracer**

2. โปรแกรมจะขอให้ Login — ใช้บัญชี **NetAcad** เดิมที่สมัครไว้

3. เลือก **"Log In with NetAcad"**

4. กรอก Email และ Password แล้วคลิก **"Login"**

5. หน้าหลักของ Packet Tracer จะปรากฏขึ้น — พร้อมใช้งาน

> ต้องมีการ Login ด้วย NetAcad account ทุกครั้งที่เปิดโปรแกรม (ต้องต่ออินเทอร์เน็ต)

---

## ปัญหาที่พบบ่อย

### ไม่เห็นปุ่มดาวน์โหลดใน Resource Hub
- ตรวจสอบว่า Login อยู่
- ตรวจสอบว่า Enroll หลักสูตรสำเร็จแล้ว
- ลอง Refresh หน้าเว็บ

### Login ไม่ได้เมื่อเปิดโปรแกรม
- ตรวจสอบการเชื่อมต่ออินเทอร์เน็ต
- ลอง Login ผ่าน Guest Mode ชั่วคราว (จำกัดฟีเจอร์บางส่วน)

### ติดตั้งแล้วโปรแกรมไม่เปิด (Windows)
- ตรวจสอบว่าเป็น Windows 64-bit
- ลองรันในฐานะ Administrator (คลิกขวา → Run as Administrator)

---

## ทรัพยากรเพิ่มเติมจาก Cisco

| แหล่งข้อมูล | ลิงก์ |
|-----------|-------|
| Cisco NetAcad (ทางการ) | [netacad.com](https://www.netacad.com) |
| Resource Hub / Download | [netacad.com/resources/lab-downloads](https://www.netacad.com/resources/lab-downloads) |
| หลักสูตร Packet Tracer ฟรี | [Getting Started with Packet Tracer](https://www.netacad.com/courses/getting-started-cisco-packet-tracer) |
| Wireshark (Lab 10) | [wireshark.org](https://www.wireshark.org/) |

---

> **Credit**: ข้อมูลในคู่มือนี้อ้างอิงจาก
> - *"Cisco Packet Tracer Download and Installation Instructions"* — Cisco Networking Academy (NetAcad)
> - *"Download Cisco Packet Tracer 9.0"* — [PacketTracerNetwork.com](https://www.packettracernetwork.com/download/download-packet-tracer.html)
>
> Cisco, Cisco Networking Academy และ Packet Tracer เป็นเครื่องหมายการค้าของ Cisco Systems, Inc.

---

[กลับไปหน้าหลัก](README.md)
