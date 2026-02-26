# Lab 2: Cable Connection และ Device Setup

## วัตถุประสงค์
- เรียนรู้ประเภทของสายเครือข่ายและการใช้งาน
- เข้าใจความแตกต่างระหว่าง Straight-Through และ Crossover Cable
- ทดลองเชื่อมต่ออุปกรณ์ประเภทต่างๆ
- เรียนรู้การใช้งาน Hub, Switch และ Router

## ระยะเวลา
30-45 นาที

## ความรู้พื้นฐานที่ต้องมี
- สายเครือข่าย (อ่านจาก [README.md](../../README.md#สายเครือข่าย-network-cables))
- องค์ประกอบของระบบเครือข่าย (อ่านจาก [README.md](../../README.md#องค์ประกอบของระบบเครือข่าย))

## อุปกรณ์ที่ใช้ใน Lab
- PC (3 เครื่อง)
- Switch (1 เครื่อง)
- Router (1 เครื่อง)
- Hub (1 เครื่อง - สำหรับเปรียบเทียบ)
- สาย Copper Straight-Through
- สาย Copper Crossover
- สาย Console

---

## ส่วนที่ 1: ทำความรู้จักกับสายเครือข่าย

### ประเภทของสายใน Cisco Packet Tracer

| ประเภทสาย | ใช้เชื่อมต่อ | ตัวอย่าง |
|-----------|-------------|----------|
| **Copper Straight-Through** (เส้นทึบสีดำ) | อุปกรณ์ต่างประเภท | PC → Switch, Switch → Router |
| **Copper Cross-Over** (เส้นประสีดำ) | อุปกรณ์เดียวกัน | PC → PC, Switch → Switch, Router → Router |
| **Console Cable** (เส้นสีฟ้า) | เชื่อมต่อเพื่อตั้งค่า | PC → Router/Switch Console Port |
| **Fiber Optic** (เส้นสีส้ม) | ระยะทางไกล | Switch → Switch (Long Distance) |

### หลักการเลือกใช้สาย

**Straight-Through Cable** (สายตรง):
- PC → Switch
- Switch → Router
- Router → Modem

**Crossover Cable** (สายไขว้):
- PC → PC
- Switch → Switch
- Router → Router

**ในปัจจุบัน**: อุปกรณ์สมัยใหม่รองรับ Auto-MDIX ทำให้ใช้สายแบบไหนก็ได้

---

## ส่วนที่ 2: ทดลองเชื่อมต่อด้วย Straight-Through Cable

### ขั้นตอนที่ 1: สร้าง Topology
1. วาง PC 2 เครื่อง และ Switch 1 เครื่อง
2. เชื่อมต่อด้วย **Copper Straight-Through**:
   - PC0 FastEthernet0 → Switch0 FastEthernet0/1
   - PC1 FastEthernet0 → Switch0 FastEthernet0/2

```
PC0 ----[Straight]---- Switch0 ----[Straight]---- PC1
```

### ขั้นตอนที่ 2: ตั้งค่า IP Address
- **PC0**: `192.168.1.10` / `255.255.255.0`
- **PC1**: `192.168.1.20` / `255.255.255.0`

### ขั้นตอนที่ 3: ทดสอบ
```bash
# จาก PC0
ping 192.168.1.20
```

**ผลลัพธ์**: ✅ ควรจะ ping ผ่าน

---

## ส่วนที่ 3: ทดลองเชื่อมต่อด้วย Crossover Cable

### ทดลอง 1: เชื่อมต่อ PC กับ PC โดยตรง
1. วาง PC 2 เครื่อง (ไม่ใช้ Switch)
2. เชื่อมต่อด้วย **Copper Cross-Over**:
   - PC2 FastEthernet0 → PC3 FastEthernet0

```
PC2 ----[Crossover]---- PC3
```

3. ตั้งค่า IP:
   - **PC2**: `192.168.2.10` / `255.255.255.0`
   - **PC3**: `192.168.2.20` / `255.255.255.0`

4. ทดสอบ:
```bash
# จาก PC2
ping 192.168.2.20
```

**ผลลัพธ์**: ✅ ควรจะ ping ผ่าน

### ทดลอง 2: ใช้ Straight-Through แทน Crossover
1. ลบสาย Crossover ออก
2. เปลี่ยนเป็น **Straight-Through Cable**
3. ทดสอบ ping อีกครั้ง

**ผลลัพธ์ (ใน Packet Tracer รุ่นเก่า)**:
- ❌ อาจจะ ping ไม่ผ่าน (เพราะไม่รองรับ Auto-MDIX)
- ✅ Packet Tracer รุ่นใหม่ อาจจะ ping ผ่าน (มี Auto-MDIX)

**บันทึก**: ในอุปกรณ์จริง PC สมัยใหม่รองรับ Auto-MDIX แล้ว

---

## ส่วนที่ 4: เปรียบเทียบ Hub vs Switch

### ทดลอง 1: ใช้ Hub
1. วาง PC 3 เครื่อง และ **Hub-PT** 1 เครื่อง
2. เชื่อมต่อ:
   - PC0 → Hub Port 0
   - PC1 → Hub Port 1
   - PC2 → Hub Port 2

3. ตั้งค่า IP:
   - **PC0**: `192.168.3.10` / `255.255.255.0`
   - **PC1**: `192.168.3.20` / `255.255.255.0`
   - **PC2**: `192.168.3.30` / `255.255.255.0`

4. ทดสอบด้วย **Simulation Mode**:
   - คลิกปุ่ม **Simulation** (มุมล่างขวา)
   - ส่ง Simple PDU จาก PC0 → PC1
   - สังเกตว่า **Hub ส่ง packet ไปทุก port** (broadcast)

### ทดลอง 2: ใช้ Switch
1. แทนที่ Hub ด้วย **Switch**
2. เชื่อมต่ออุปกรณ์เหมือนเดิม
3. ส่ง Simple PDU จาก PC0 → PC1
4. สังเกตว่า **Switch ส่ง packet เฉพาะ port ปลายทาง**

### สรุปความแตกต่าง

| คุณสมบัติ | Hub | Switch |
|----------|-----|--------|
| Layer ของ OSI | Layer 1 (Physical) | Layer 2 (Data Link) |
| การส่งข้อมูล | ส่งไปทุก port (Broadcast) | ส่งเฉพาะ port ปลายทาง |
| ความเร็ว | แบ่งกันใช้ bandwidth | แต่ละ port มี bandwidth เต็ม |
| ความปลอดภัย | ต่ำ (ทุกเครื่องเห็นข้อมูล) | สูงกว่า (แยก traffic) |
| ราคา | ถูก (เลิกผลิตแล้ว) | ปานกลาง-แพง |
| แนะนำใช้ | ❌ ไม่แนะนำ | ✅ แนะนำ |

---

## ส่วนที่ 5: เชื่อมต่อ Router เข้ากับเครือข่าย

### ขั้นตอนที่ 1: สร้าง Topology แบบมี Router
```
[PC0] ---|            |--- [Router] --- [Server]
[PC1] ---| Switch0    |
[PC2] ---|            |
```

1. วาง **Router 2911** ลงในพื้นที่ทำงาน
2. วาง **Switch** 1 เครื่อง
3. วาง **PC** 3 เครื่อง
4. วาง **Server** 1 เครื่อง

### ขั้นตอนที่ 2: เชื่อมต่ออุปกรณ์
1. เชื่อมต่อ PC กับ Switch:
   - PC0 FastEthernet0 → Switch0 FastEthernet0/1
   - PC1 FastEthernet0 → Switch0 FastEthernet0/2
   - PC2 FastEthernet0 → Switch0 FastEthernet0/3

2. เชื่อมต่อ Switch กับ Router:
   - Switch0 FastEthernet0/24 → Router0 GigabitEthernet0/0

3. เชื่อมต่อ Router กับ Server:
   - Router0 GigabitEthernet0/1 → Server0 FastEthernet0

### ขั้นตอนที่ 3: ตั้งค่า IP Address

**วง LAN (ฝั่ง PC)**:
- PC0: `192.168.1.10` / `255.255.255.0` / Gateway: `192.168.1.1`
- PC1: `192.168.1.20` / `255.255.255.0` / Gateway: `192.168.1.1`
- PC2: `192.168.1.30` / `255.255.255.0` / Gateway: `192.168.1.1`

**Router GigabitEthernet0/0** (ฝั่ง LAN):
1. คลิกที่ Router
2. เลือกแท็บ **CLI**
3. กด Enter จนขึ้น `Router>` แล้วพิมพ์:
```
Router>enable
Router#configure terminal
Router(config)#interface gigabitEthernet 0/0
Router(config-if)#ip address 192.168.1.1 255.255.255.0
Router(config-if)#no shutdown
Router(config-if)#exit
```

**วง LAN (ฝั่ง Server)**:
- Server0: `10.0.0.10` / `255.255.255.0` / Gateway: `10.0.0.1`

**Router GigabitEthernet0/1** (ฝั่ง Server):
```
Router(config)#interface gigabitEthernet 0/1
Router(config-if)#ip address 10.0.0.1 255.255.255.0
Router(config-if)#no shutdown
Router(config-if)#exit
Router(config)#exit
Router#
```

### ขั้นตอนที่ 4: ทดสอบการเชื่อมต่อ

**ทดสอบ 1**: PC0 ping ไป PC1 (ในวงเดียวกัน)
```bash
ping 192.168.1.20
```
✅ ควรจะ ping ผ่าน (ไม่ต้องผ่าน Router)

**ทดสอบ 2**: PC0 ping ไป Gateway
```bash
ping 192.168.1.1
```
✅ ควรจะ ping ผ่าน (ถึง Router)

**ทดสอบ 3**: PC0 ping ไป Server (ต่างวง)
```bash
ping 10.0.0.10
```
✅ ควรจะ ping ผ่าน (ผ่าน Router)

---

## ส่วนที่ 6: ใช้ Console Cable เพื่อตั้งค่า Router

### ขั้นตอนที่ 1: เชื่อมต่อ Console
1. เชื่อมต่อ PC กับ Router:
   - PC FastEthernet0 → ไม่ต้องเสียบ
   - PC **RS232** → Router **Console**
   - ใช้สาย **Console Cable** (สีฟ้า)

### ขั้นตอนที่ 2: เปิด Terminal
1. คลิกที่ PC
2. เลือกแท็บ **Desktop**
3. คลิก **Terminal**
4. ตั้งค่า:
   - Bits per second: 9600
   - Data bits: 8
   - Parity: None
   - Stop bits: 1
   - Flow Control: None
5. คลิก **OK**

### ขั้นตอนที่ 3: เข้าสู่ CLI ของ Router
จะเห็นหน้าจอ:
```
Router>
```

ลองพิมพ์คำสั่ง:
```bash
Router>enable
Router#show ip interface brief
```

จะเห็นรายการ interface และ IP address ทั้งหมด

---

## ส่วนที่ 7: ทดสอบ Fiber Optic Cable

### ขั้นตอน:
1. วาง Switch 2 เครื่อง ห่างกันไกลๆ
2. เชื่อมต่อด้วย **Fiber Optic Cable**:
   - Switch0 GigabitEthernet0/1 → Switch1 GigabitEthernet0/1
3. เชื่อมต่อ PC ทั้งสองฝั่ง
4. ตั้งค่า IP ให้อยู่ในวงเดียวกัน
5. ทดสอบ ping

**ข้อสังเกต**:
- Fiber Optic ใช้สำหรับระยะทางไกล (>100m)
- ความเร็วสูง (1Gbps-100Gbps)
- ไม่รับสัญญาณรบกวนจากคลื่นแม่เหล็กไฟฟ้า

---

## ส่วนที่ 8: คำถามท้ายบท

1. **ทำไมต้องใช้สาย Crossover เมื่อเชื่อมต่อ PC กับ PC?**
2. **Hub และ Switch ต่างกันอย่างไร? ทำไมปัจจุบันไม่ใช้ Hub แล้ว?**
3. **Router ทำหน้าที่อะไรในเครือข่าย?**
4. **Console Cable ใช้ทำอะไร?**
5. **Fiber Optic Cable มีข้อดีอะไรบ้างเมื่อเทียบกับสาย Copper?**
6. **ในโรงเรียน/มหาวิทยาลัยควรใช้สายประเภทไหนในการเดินสาย?**

---

## ส่วนที่ 9: การส่งงาน

### สิ่งที่ต้องส่ง:
1. ไฟล์ `.pkt` ของ Lab นี้ (ตั้งชื่อว่า `Lab2-YourName.pkt`)
2. Screenshot ของ:
   - Topology ที่มี PC, Switch, Router, และ Server
   - ผลการ ping ข้ามวง (จาก PC ไป Server)
   - หน้าจอ Terminal ที่เชื่อมต่อผ่าน Console Cable
   - การเปรียบเทียบ Hub vs Switch ใน Simulation Mode
3. เอกสาร Word/PDF ตอบคำถามท้ายบท

---

## ปัญหาที่พบบ่อย (Troubleshooting)

### ปัญหา: Router Interface เป็นสีแดง
**วิธีแก้**:
- ต้องใช้คำสั่ง `no shutdown` เพื่อเปิด interface
```
Router(config)#interface gigabitEthernet 0/0
Router(config-if)#no shutdown
```

### ปัญหา: Ping ข้ามวงไม่ผ่าน
**วิธีแก้**:
1. ตรวจสอบ Default Gateway ของ PC
2. ตรวจสอบ IP Address ของ Router Interface ทั้งสองฝั่ง
3. ตรวจสอบว่า Router Interface เปิดใช้งาน (`no shutdown`)

### ปัญหา: Console Cable ไม่ทำงาน
**วิธีแก้**:
1. ตรวจสอบว่าเสียบเข้า Console Port ของ Router (ไม่ใช่ Ethernet Port)
2. ตรวจสอบว่าเสียบเข้า RS232 Port ของ PC
3. ตั้งค่า Terminal ให้ถูกต้อง (9600 baud)

---

## สรุป

ใน Lab นี้คุณได้เรียนรู้:
- ✅ ประเภทของสายเครือข่ายและการเลือกใช้
- ✅ ความแตกต่างระหว่าง Straight-Through และ Crossover Cable
- ✅ ความแตกต่างระหว่าง Hub และ Switch
- ✅ การเชื่อมต่อ Router เข้ากับเครือข่าย
- ✅ การใช้ Console Cable เพื่อตั้งค่าอุปกรณ์
- ✅ การใช้ Fiber Optic Cable

**Lab ถัดไป**: [Lab 3: Router และ Switch Configuration](../lab3-router-switch/README.md)

---

[⬅️ กลับไปหน้าหลัก](../../README.md) | [⬅️ Lab ก่อนหน้า](../lab1-basic-network/README.md)
