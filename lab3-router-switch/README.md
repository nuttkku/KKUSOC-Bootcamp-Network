# Lab 3: Router และ Switch Configuration

## วัตถุประสงค์
- เรียนรู้การตั้งค่า Router และ Switch ผ่าน CLI (Command Line Interface)
- เข้าใจคำสั่ง Cisco IOS พื้นฐาน
- ตั้งค่า Hostname, Password และ Banner
- กำหนด IP Address สำหรับ Router Interface
- บันทึกและโหลดค่า Configuration
- เรียนรู้การ Routing แบบ Static

## ระยะเวลา
45-60 นาที

## ความรู้พื้นฐานที่ต้องมี
- Lab 1: Basic Network และ IP Configuration
- Lab 2: Cable Connection และ Device Setup
- องค์ประกอบของระบบเครือข่าย (อ่านจาก [README.md](../../README.md#องค์ประกอบของระบบเครือข่าย))

## อุปกรณ์ที่ใช้ใน Lab
- Router 2911 (2 เครื่อง)
- Switch 2960 (2 เครื่อง)
- PC (4 เครื่อง)
- สาย Copper Straight-Through
- สาย Console (สำหรับตั้งค่า)

---

## ส่วนที่ 1: รู้จักกับ Cisco IOS Command Modes

### โหมดการทำงานของ Cisco IOS

| โหมด | Prompt | คำอธิบาย | เข้าสู่โหมด |
|------|--------|----------|-----------|
| **User EXEC Mode** | `Router>` | โหมดผู้ใช้ทั่วไป (ดูข้อมูลพื้นฐาน) | เริ่มต้น |
| **Privileged EXEC Mode** | `Router#` | โหมดผู้ดูแลระบบ (ดูข้อมูลทั้งหมด) | พิมพ์ `enable` |
| **Global Configuration Mode** | `Router(config)#` | ตั้งค่าทั่วไปของอุปกรณ์ | พิมพ์ `configure terminal` |
| **Interface Configuration Mode** | `Router(config-if)#` | ตั้งค่า interface | พิมพ์ `interface <name>` |
| **Line Configuration Mode** | `Router(config-line)#` | ตั้งค่า console/vty | พิมพ์ `line console 0` |

### คำสั่งพื้นฐาน

```bash
# เข้าสู่โหมด Privileged
Router> enable
Router#

# เข้าสู่โหมด Global Configuration
Router# configure terminal
Router(config)#

# กลับไปโหมดก่อนหน้า
Router(config)# exit
Router#

# กลับไป Privileged Mode ทันที
Router(config-if)# end
Router#

# ออกจากระบบ
Router# exit
```

---

## ส่วนที่ 2: ตั้งค่าพื้นฐานของ Router

### Network Topology
```
[PC0: 192.168.1.10] --- [Switch0] --- [Router0] --- [Router1] --- [Switch1] --- [PC1: 192.168.2.10]
                                   Gi0/0      Gi0/1 Gi0/0      Gi0/1
                             192.168.1.1   10.0.0.1  10.0.0.2  192.168.2.1
```

### ขั้นตอนที่ 1: สร้าง Topology
1. วาง Router 2 เครื่อง (Router0 และ Router1)
2. วาง Switch 2 เครื่อง (Switch0 และ Switch1)
3. วาง PC 2 เครื่อง (PC0 และ PC1)
4. เชื่อมต่อตาม Diagram:
   - PC0 → Switch0
   - Switch0 → Router0 (Gi0/0)
   - Router0 (Gi0/1) → Router1 (Gi0/0)
   - Router1 (Gi0/1) → Switch1
   - Switch1 → PC1

### ขั้นตอนที่ 2: ตั้งค่า Router0
1. คลิกที่ Router0
2. เลือกแท็บ **CLI**
3. กด Enter และพิมพ์:

```bash
Router>enable
Router#configure terminal

# ตั้งชื่อ Router
Router(config)#hostname R0
R0(config)#

# ตั้ง Banner (ข้อความต้อนรับ)
R0(config)#banner motd #
Enter TEXT message. End with the character '#'.
*** Authorized Access Only! KKU Network Lab ***
#

# ตั้งรหัสผ่านสำหรับ Privileged Mode
R0(config)#enable secret cisco123

# ตั้งรหัสผ่านสำหรับ Console
R0(config)#line console 0
R0(config-line)#password console123
R0(config-line)#login
R0(config-line)#exit

# ตั้งรหัสผ่านสำหรับ VTY (Telnet/SSH)
R0(config)#line vty 0 4
R0(config-line)#password vty123
R0(config-line)#login
R0(config-line)#exit

# เข้ารหัสรหัสผ่านทั้งหมด
R0(config)#service password-encryption
```

### ขั้นตอนที่ 3: ตั้งค่า Interface ของ Router0

```bash
# ตั้งค่า GigabitEthernet 0/0 (ฝั่ง LAN)
R0(config)#interface gigabitEthernet 0/0
R0(config-if)#description Link to Switch0 (LAN)
R0(config-if)#ip address 192.168.1.1 255.255.255.0
R0(config-if)#no shutdown
R0(config-if)#exit

# ตั้งค่า GigabitEthernet 0/1 (ฝั่ง WAN)
R0(config)#interface gigabitEthernet 0/1
R0(config-if)#description Link to Router1 (WAN)
R0(config-if)#ip address 10.0.0.1 255.255.255.252
R0(config-if)#no shutdown
R0(config-if)#exit
```

### ขั้นตอนที่ 4: ตั้งค่า Static Route

```bash
# กำหนด route ไปยังวงเครือข่าย 192.168.2.0/24
R0(config)#ip route 192.168.2.0 255.255.255.0 10.0.0.2

# กำหนด Default Route (ออก Internet)
R0(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2
```

### ขั้นตอนที่ 5: บันทึกค่า Configuration

```bash
R0(config)#exit
R0#copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]
```

หรือใช้คำสั่งย่อ:
```bash
R0#write memory
```

---

## ส่วนที่ 3: ตั้งค่า Router1

ทำซ้ำขั้นตอนเดียวกับ Router0 แต่เปลี่ยนค่าดังนี้:

```bash
Router>enable
Router#configure terminal
Router(config)#hostname R1

# ตั้ง Banner
R1(config)#banner motd #
*** Router R1 - Authorized Access Only! ***
#

# ตั้งรหัสผ่าน
R1(config)#enable secret cisco123
R1(config)#line console 0
R1(config-line)#password console123
R1(config-line)#login
R1(config-line)#exit

R1(config)#line vty 0 4
R1(config-line)#password vty123
R1(config-line)#login
R1(config-line)#exit

R1(config)#service password-encryption

# ตั้งค่า Interface
R1(config)#interface gigabitEthernet 0/0
R1(config-if)#description Link to Router0 (WAN)
R1(config-if)#ip address 10.0.0.2 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#exit

R1(config)#interface gigabitEthernet 0/1
R1(config-if)#description Link to Switch1 (LAN)
R1(config-if)#ip address 192.168.2.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#exit

# Static Route
R1(config)#ip route 192.168.1.0 255.255.255.0 10.0.0.1
R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.1

# บันทึก
R1(config)#exit
R1#write memory
```

---

## ส่วนที่ 4: ตั้งค่า Switch

### ขั้นตอนที่ 1: ตั้งค่า Switch0

```bash
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW0

# ตั้ง Banner
SW0(config)#banner motd #
*** Switch SW0 - Access Switch ***
#

# ตั้งรหัสผ่าน
SW0(config)#enable secret cisco123
SW0(config)#line console 0
SW0(config-line)#password console123
SW0(config-line)#login
SW0(config-line)#exit

SW0(config)#service password-encryption

# ตั้ง Management IP (VLAN 1)
SW0(config)#interface vlan 1
SW0(config-if)#ip address 192.168.1.254 255.255.255.0
SW0(config-if)#no shutdown
SW0(config-if)#exit

# ตั้ง Default Gateway
SW0(config)#ip default-gateway 192.168.1.1

# บันทึก
SW0(config)#exit
SW0#write memory
```

### ขั้นตอนที่ 2: ตั้งค่า Switch1

ทำเหมือนกับ Switch0 แต่เปลี่ยนค่า:

```bash
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW1

SW1(config)#banner motd #
*** Switch SW1 - Access Switch ***
#

SW1(config)#enable secret cisco123
SW1(config)#line console 0
SW1(config-line)#password console123
SW1(config-line)#login
SW1(config-line)#exit

SW1(config)#service password-encryption

SW1(config)#interface vlan 1
SW1(config-if)#ip address 192.168.2.254 255.255.255.0
SW1(config-if)#no shutdown
SW1(config-if)#exit

SW1(config)#ip default-gateway 192.168.2.1

SW1(config)#exit
SW1#write memory
```

---

## ส่วนที่ 5: ตั้งค่า PC และทดสอบ

### ตั้งค่า PC0
- IP Address: `192.168.1.10`
- Subnet Mask: `255.255.255.0`
- Default Gateway: `192.168.1.1`

### ตั้งค่า PC1
- IP Address: `192.168.2.10`
- Subnet Mask: `255.255.255.0`
- Default Gateway: `192.168.2.1`

### ทดสอบการเชื่อมต่อ

**จาก PC0**:
```bash
# Ping Gateway
ping 192.168.1.1

# Ping WAN Interface ของ R0
ping 10.0.0.1

# Ping WAN Interface ของ R1
ping 10.0.0.2

# Ping Gateway ของฝั่ง R1
ping 192.168.2.1

# Ping PC1 (ข้ามวงเครือข่าย)
ping 192.168.2.10
```

✅ ทุกคำสั่งควรจะ ping ผ่าน!

---

## ส่วนที่ 6: คำสั่ง Show ที่ใช้บ่อย

### คำสั่งสำหรับ Router

```bash
# แสดงข้อมูล interface ทั้งหมด
R0#show ip interface brief

# แสดง Routing Table
R0#show ip route

# แสดง Configuration ที่ใช้งานอยู่
R0#show running-config

# แสดง Configuration ที่บันทึกไว้
R0#show startup-config

# แสดงข้อมูล interface ละเอียด
R0#show interface gigabitEthernet 0/0

# แสดงสถานะ interface
R0#show ip interface gigabitEthernet 0/0

# แสดง Version และ IOS
R0#show version
```

### คำสั่งสำหรับ Switch

```bash
# แสดง MAC Address Table
SW0#show mac address-table

# แสดง VLAN
SW0#show vlan brief

# แสดง Interface
SW0#show interface status

# แสดง Spanning Tree
SW0#show spanning-tree
```

---

## ส่วนที่ 7: ทดสอบ Traceroute

### จาก PC0 ไป PC1

```bash
tracert 192.168.2.10
```

**ผลลัพธ์ที่ควรได้**:
```
Tracing route to 192.168.2.10 over a maximum of 30 hops:

  1   <1 ms   <1 ms   <1 ms   192.168.1.1    (R0 Gi0/0)
  2   <1 ms   <1 ms   <1 ms   10.0.0.2       (R1 Gi0/0)
  3   <1 ms   <1 ms   <1 ms   192.168.2.10   (PC1)

Trace complete.
```

**อธิบาย**:
- Hop 1: ออกจาก PC0 ไปยัง Gateway (R0)
- Hop 2: จาก R0 ไปยัง R1 ผ่าน WAN
- Hop 3: จาก R1 ไปถึง PC1

---

## ส่วนที่ 8: การ Backup และ Restore Configuration

### Backup Configuration ไปยัง TFTP Server

1. วาง **Server** ลงในพื้นที่ทำงาน
2. ตั้งค่า IP Server: `192.168.1.100` / `255.255.255.0`
3. เปิด **Services** > **TFTP** บน Server
4. บน Router:

```bash
R0#copy running-config tftp:
Address or name of remote host []? 192.168.1.100
Destination filename [r0-confg]? r0-backup.cfg
!!
1465 bytes copied in 0.058 secs
```

### Restore Configuration จาก TFTP Server

```bash
R0#copy tftp: running-config
Address or name of remote host []? 192.168.1.100
Source filename []? r0-backup.cfg
Destination filename [running-config]?
Accessing tftp://192.168.1.100/r0-backup.cfg...
Loading r0-backup.cfg from 192.168.1.100: !
[OK - 1465 bytes]

1465 bytes copied in 0.081 secs
```

---

## ส่วนที่ 9: การ Reset Configuration

### ลบ Configuration และเริ่มใหม่

```bash
# ลบ Startup Configuration
R0#erase startup-config
Erasing the nvram filesystem will remove all configuration files! Continue? [confirm]
[OK]
Erase of nvram: complete

# Restart Router
R0#reload
System configuration has been modified. Save? [yes/no]: no
Proceed with reload? [confirm]
```

**หมายเหตุ**: Router จะรีสตาร์ทและกลับไปสู่สถานะเริ่มต้น

---

## ส่วนที่ 10: คำถามท้ายบท

1. **อธิบายความแตกต่างระหว่าง running-config และ startup-config**
2. **ทำไมต้องใช้คำสั่ง `no shutdown` หลังจากตั้งค่า interface?**
3. **Static Route คืออะไร? ต่างจาก Dynamic Routing อย่างไร?**
4. **คำสั่ง `service password-encryption` ทำอะไร?**
5. **ถ้า PC0 ไม่สามารถ ping ไป PC1 ได้ จะตรวจสอบอย่างไร? (ระบุขั้นตอน)**
6. **ทำไมต้องตั้ง Default Gateway ให้กับ Switch?**
7. **อธิบาย Routing Table ที่เห็นจากคำสั่ง `show ip route`**

---

## ส่วนที่ 11: การส่งงาน

### สิ่งที่ต้องส่ง:
1. ไฟล์ `.pkt` ของ Lab นี้ (ตั้งชื่อว่า `Lab3-YourName.pkt`)
2. Screenshot ของ:
   - Network Topology ที่สมบูรณ์
   - คำสั่ง `show running-config` จาก Router0 และ Router1
   - คำสั่ง `show ip interface brief` จากทั้ง 2 Router
   - คำสั่ง `show ip route` จากทั้ง 2 Router
   - ผลการ ping และ tracert จาก PC0 ไป PC1
3. เอกสาร Word/PDF:
   - บันทึกขั้นตอนการทำ Lab
   - ตอบคำถามท้ายบท
   - อธิบาย Routing Table ที่ได้

---

## ปัญหาที่พบบ่อย (Troubleshooting)

### ปัญหา: Ping ข้ามวงไม่ผ่าน
**วิธีแก้**:
1. ตรวจสอบ Static Route ด้วย `show ip route`
2. ตรวจสอบ Default Gateway ของ PC
3. ตรวจสอบว่า Interface ทั้งหมดเปิดใช้งาน (up/up)
4. ทดสอบทีละ hop:
   - PC0 → Gateway (R0)
   - R0 → R1
   - R1 → Gateway
   - Gateway → PC1

### ปัญหา: Static Route ไม่ทำงาน
**วิธีแก้**:
```bash
# ตรวจสอบ Routing Table
R0#show ip route

# ลบ Route เก่า (ถ้ามี)
R0(config)#no ip route 192.168.2.0 255.255.255.0 10.0.0.2

# เพิ่ม Route ใหม่
R0(config)#ip route 192.168.2.0 255.255.255.0 10.0.0.2
```

### ปัญหา: ลืมรหัสผ่าน enable
**วิธีแก้**:
- ใน Packet Tracer: ลบอุปกรณ์และสร้างใหม่
- ในอุปกรณ์จริง: ต้องทำ Password Recovery (นอกเหนือจาก Lab นี้)

---

## สรุป

ใน Lab นี้คุณได้เรียนรู้:
- ✅ โหมดการทำงานของ Cisco IOS
- ✅ คำสั่งพื้นฐานในการตั้งค่า Router และ Switch
- ✅ การกำหนด Hostname, Password และ Banner
- ✅ การตั้งค่า IP Address สำหรับ Interface
- ✅ การสร้าง Static Route
- ✅ การบันทึกและโหลด Configuration
- ✅ คำสั่ง Show ที่สำคัญ
- ✅ การ Backup และ Restore Configuration

**Lab ถัดไป**: [Lab 4: VLAN Configuration](../lab4-vlan/README.md)

---

[⬅️ กลับไปหน้าหลัก](../../README.md) | [⬅️ Lab ก่อนหน้า](../lab2-cables-devices/README.md)
