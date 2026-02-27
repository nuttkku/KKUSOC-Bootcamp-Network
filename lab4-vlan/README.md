# Lab 4: VLAN Configuration

## วัตถุประสงค์
- เข้าใจแนวคิดของ VLAN (Virtual LAN)
- สร้างและกำหนด VLAN บน Switch
- ตั้งค่า Access Port และ Trunk Port
- ทำ Inter-VLAN Routing ด้วย Router
- ใช้ IP Helper Address (DHCP Relay)
- เข้าใจความแตกต่างระหว่าง Access Mode และ Trunk Mode

## ระยะเวลา
60-90 นาที

## ความรู้พื้นฐานที่ต้องมี
- Lab 3: Router และ Switch Configuration
- VLAN และการจัดการเครือข่าย (อ่านจาก [README.md](../../README.md#vlan-และการจัดการเครือข่าย))

## อุปกรณ์ที่ใช้ใน Lab
- Router 2911 (1 เครื่อง)
- Switch 2960 (2 เครื่อง)
- PC (6 เครื่อง)
- Server (1 เครื่อง - DHCP Server)
- สาย Copper Straight-Through

---

## ส่วนที่ 1: ทำความเข้าใจ VLAN

### VLAN คืออะไร?

**VLAN (Virtual LAN)** คือการแบ่งวง LAN ทางกายภาพหนึ่งออกเป็นหลายๆ วง LAN เสมือน เพื่อ:
- แบ่งแยก Broadcast Domain
- เพิ่มความปลอดภัย (แยก Traffic ตามกลุ่มผู้ใช้)
- จัดการเครือข่ายได้ง่ายขึ้น
- ลด Broadcast Traffic

### ประเภทของ Port

| ประเภท | ชื่อ | คำอธิบาย | ใช้เชื่อมต่อ |
|--------|------|----------|-------------|
| **Access Port** | Access Mode | ส่งผ่านเฉพาะ 1 VLAN | PC, Printer, Server |
| **Trunk Port** | Trunk Mode | ส่งผ่านหลาย VLAN พร้อมกัน | Switch → Switch, Switch → Router |

### VLAN Tagging

- **Access Port**: ไม่มี VLAN Tag (Untagged)
- **Trunk Port**: มี VLAN Tag (Tagged) ใช้ IEEE 802.1Q

---

## ส่วนที่ 2: สร้าง Network Topology

### Network Topology

```
                    Router0 (Inter-VLAN Routing)
                       Gi0/0 (Trunk)
                          |
                          |
                      Switch0 (Core Switch)
                    /           \
                   /             \
        (Trunk) Fa0/24         Fa0/23 (Trunk)
              /                     \
          Switch1                  Switch2
         /   |   \                /   |   \
       PC0 PC1 PC2            PC3 PC4 PC5
      (V10)(V20)(V30)       (V10)(V20)(V30)

VLAN 10: Sales (192.168.10.0/24) - PC0, PC3
VLAN 20: IT (192.168.20.0/24) - PC1, PC4
VLAN 30: Management (192.168.30.0/24) - PC2, PC5
VLAN 99: Native VLAN (ไม่ใช้งาน)
```

### ขั้นตอนที่ 1: วางอุปกรณ์
1. วาง Router 2911 (1 เครื่อง)
2. วาง Switch 2960 (3 เครื่อง - Switch0, Switch1, Switch2)
3. วาง PC (6 เครื่อง)
4. วาง Server (1 เครื่อง - สำหรับ DHCP)

### ขั้นตอนที่ 2: เชื่อมต่ออุปกรณ์
- Router0 Gi0/0 → Switch0 Fa0/1 (Trunk)
- Switch0 Fa0/24 → Switch1 Fa0/1 (Trunk)
- Switch0 Fa0/23 → Switch2 Fa0/1 (Trunk)
- PC0 → Switch1 Fa0/2 (VLAN 10)
- PC1 → Switch1 Fa0/3 (VLAN 20)
- PC2 → Switch1 Fa0/4 (VLAN 30)
- PC3 → Switch2 Fa0/2 (VLAN 10)
- PC4 → Switch2 Fa0/3 (VLAN 20)
- PC5 → Switch2 Fa0/4 (VLAN 30)
- Server0 → Switch0 Fa0/10 (VLAN 30)

---

## ส่วนที่ 3: ตั้งค่า VLAN บน Switch0 (Core Switch)

### ขั้นตอนที่ 1: สร้าง VLAN

```bash
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW0

# สร้าง VLAN 10 - Sales
SW0(config)#vlan 10
SW0(config-vlan)#name Sales
SW0(config-vlan)#exit

# สร้าง VLAN 20 - IT
SW0(config)#vlan 20
SW0(config-vlan)#name IT
SW0(config-vlan)#exit

# สร้าง VLAN 30 - Management
SW0(config)#vlan 30
SW0(config-vlan)#name Management
SW0(config-vlan)#exit

# สร้าง VLAN 99 - Native VLAN
SW0(config)#vlan 99
SW0(config-vlan)#name Native
SW0(config-vlan)#exit
```

### ขั้นตอนที่ 2: ตั้งค่า Trunk Port ไปยัง Router

```bash
# Port ที่เชื่อมกับ Router (Fa0/1)
SW0(config)#interface fastEthernet 0/1
SW0(config-if)#description Trunk to Router0
SW0(config-if)#switchport mode trunk
SW0(config-if)#switchport trunk native vlan 99
SW0(config-if)#switchport trunk allowed vlan 10,20,30,99
SW0(config-if)#exit
```

### ขั้นตอนที่ 3: ตั้งค่า Trunk Port ไปยัง Switch อื่น

```bash
# Port ที่เชื่อมกับ Switch1 (Fa0/24)
SW0(config)#interface fastEthernet 0/24
SW0(config-if)#description Trunk to Switch1
SW0(config-if)#switchport mode trunk
SW0(config-if)#switchport trunk native vlan 99
SW0(config-if)#switchport trunk allowed vlan 10,20,30,99
SW0(config-if)#exit

# Port ที่เชื่อมกับ Switch2 (Fa0/23)
SW0(config)#interface fastEthernet 0/23
SW0(config-if)#description Trunk to Switch2
SW0(config-if)#switchport mode trunk
SW0(config-if)#switchport trunk native vlan 99
SW0(config-if)#switchport trunk allowed vlan 10,20,30,99
SW0(config-if)#exit
```

### ขั้นตอนที่ 4: ตั้งค่า Access Port สำหรับ Server

```bash
# Port สำหรับ DHCP Server (Fa0/10)
SW0(config)#interface fastEthernet 0/10
SW0(config-if)#description DHCP Server
SW0(config-if)#switchport mode access
SW0(config-if)#switchport access vlan 30
SW0(config-if)#exit
```

### ขั้นตอนที่ 5: ตั้งค่า Management IP

```bash
SW0(config)#interface vlan 30
SW0(config-if)#ip address 192.168.30.252 255.255.255.0
SW0(config-if)#no shutdown
SW0(config-if)#exit

SW0(config)#ip default-gateway 192.168.30.1
SW0(config)#exit
SW0#write memory
```

---

## ส่วนที่ 4: ตั้งค่า Switch1 (Access Switch)

```bash
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW1

# สร้าง VLAN
SW1(config)#vlan 10
SW1(config-vlan)#name Sales
SW1(config-vlan)#exit

SW1(config)#vlan 20
SW1(config-vlan)#name IT
SW1(config-vlan)#exit

SW1(config)#vlan 30
SW1(config-vlan)#name Management
SW1(config-vlan)#exit

SW1(config)#vlan 99
SW1(config-vlan)#name Native
SW1(config-vlan)#exit

# ตั้งค่า Trunk Port ไปยัง SW0
SW1(config)#interface fastEthernet 0/1
SW1(config-if)#description Trunk to SW0
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport trunk native vlan 99
SW1(config-if)#switchport trunk allowed vlan 10,20,30,99
SW1(config-if)#exit

# ตั้งค่า Access Port สำหรับ PC0 (VLAN 10)
SW1(config)#interface fastEthernet 0/2
SW1(config-if)#description PC0 - Sales
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 10
SW1(config-if)#exit

# ตั้งค่า Access Port สำหรับ PC1 (VLAN 20)
SW1(config)#interface fastEthernet 0/3
SW1(config-if)#description PC1 - IT
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 20
SW1(config-if)#exit

# ตั้งค่า Access Port สำหรับ PC2 (VLAN 30)
SW1(config)#interface fastEthernet 0/4
SW1(config-if)#description PC2 - Management
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 30
SW1(config-if)#exit

# Management IP
SW1(config)#interface vlan 30
SW1(config-if)#ip address 192.168.30.253 255.255.255.0
SW1(config-if)#no shutdown
SW1(config-if)#exit

SW1(config)#ip default-gateway 192.168.30.1
SW1(config)#exit
SW1#write memory
```

---

## ส่วนที่ 5: ตั้งค่า Switch2 (Access Switch)

ทำแบบเดียวกับ Switch1:

```bash
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW2

# สร้าง VLAN (เหมือน SW1)
SW2(config)#vlan 10
SW2(config-vlan)#name Sales
SW2(config-vlan)#exit

SW2(config)#vlan 20
SW2(config-vlan)#name IT
SW2(config-vlan)#exit

SW2(config)#vlan 30
SW2(config-vlan)#name Management
SW2(config-vlan)#exit

SW2(config)#vlan 99
SW2(config-vlan)#name Native
SW2(config-vlan)#exit

# Trunk Port
SW2(config)#interface fastEthernet 0/1
SW2(config-if)#description Trunk to SW0
SW2(config-if)#switchport mode trunk
SW2(config-if)#switchport trunk native vlan 99
SW2(config-if)#switchport trunk allowed vlan 10,20,30,99
SW2(config-if)#exit

# Access Ports
SW2(config)#interface fastEthernet 0/2
SW2(config-if)#description PC3 - Sales
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 10
SW2(config-if)#exit

SW2(config)#interface fastEthernet 0/3
SW2(config-if)#description PC4 - IT
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 20
SW2(config-if)#exit

SW2(config)#interface fastEthernet 0/4
SW2(config-if)#description PC5 - Management
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 30
SW2(config-if)#exit

# Management IP
SW2(config)#interface vlan 30
SW2(config-if)#ip address 192.168.30.254 255.255.255.0
SW2(config-if)#no shutdown
SW2(config-if)#exit

SW2(config)#ip default-gateway 192.168.30.1
SW2(config)#exit
SW2#write memory
```

---

## ส่วนที่ 6: ตั้งค่า Router สำหรับ Inter-VLAN Routing

### Router-on-a-Stick Configuration

```bash
Router>enable
Router#configure terminal
Router(config)#hostname R0

# ตั้งค่า Sub-interface สำหรับแต่ละ VLAN
# Sub-interface Gi0/0.10 สำหรับ VLAN 10
R0(config)#interface gigabitEthernet 0/0.10
R0(config-subif)#description Sales VLAN
R0(config-subif)#encapsulation dot1Q 10
R0(config-subif)#ip address 192.168.10.1 255.255.255.0
R0(config-subif)#exit

# Sub-interface Gi0/0.20 สำหรับ VLAN 20
R0(config)#interface gigabitEthernet 0/0.20
R0(config-subif)#description IT VLAN
R0(config-subif)#encapsulation dot1Q 20
R0(config-subif)#ip address 192.168.20.1 255.255.255.0
R0(config-subif)#exit

# Sub-interface Gi0/0.30 สำหรับ VLAN 30
R0(config)#interface gigabitEthernet 0/0.30
R0(config-subif)#description Management VLAN
R0(config-subif)#encapsulation dot1Q 30
R0(config-subif)#ip address 192.168.30.1 255.255.255.0
R0(config-subif)#exit

# Sub-interface Gi0/0.99 สำหรับ Native VLAN
R0(config)#interface gigabitEthernet 0/0.99
R0(config-subif)#description Native VLAN
R0(config-subif)#encapsulation dot1Q 99 native
R0(config-subif)#exit

# เปิด Physical Interface
R0(config)#interface gigabitEthernet 0/0
R0(config-if)#description Trunk to SW0
R0(config-if)#no shutdown
R0(config-if)#exit

R0(config)#exit
R0#write memory
```

---

## ส่วนที่ 7: ตั้งค่า DHCP Server

### ตั้งค่า Server0
1. คลิกที่ Server0
2. เลือกแท็บ **Desktop** > **IP Configuration**
3. ตั้งค่า Static IP:
   - IP Address: `192.168.30.10`
   - Subnet Mask: `255.255.255.0`
   - Default Gateway: `192.168.30.1`

4. เลือกแท็บ **Services** > **DHCP**
5. สร้าง DHCP Pool สำหรับ VLAN 10:
   - Pool Name: `VLAN10-Sales`
   - Default Gateway: `192.168.10.1`
   - DNS Server: `192.168.30.10`
   - Start IP Address: `192.168.10.100`
   - Subnet Mask: `255.255.255.0`
   - Maximum Number of Users: 50
   - คลิก **Save**

6. สร้าง DHCP Pool สำหรับ VLAN 20:
   - Pool Name: `VLAN20-IT`
   - Default Gateway: `192.168.20.1`
   - DNS Server: `192.168.30.10`
   - Start IP Address: `192.168.20.100`
   - Subnet Mask: `255.255.255.0`
   - Maximum Number of Users: 50
   - คลิก **Save**

7. สร้าง DHCP Pool สำหรับ VLAN 30:
   - Pool Name: `VLAN30-Mgmt`
   - Default Gateway: `192.168.30.1`
   - DNS Server: `192.168.30.10`
   - Start IP Address: `192.168.30.100`
   - Subnet Mask: `255.255.255.0`
   - Maximum Number of Users: 50
   - คลิก **Save**

---

## ส่วนที่ 8: ตั้งค่า IP Helper Address บน Router

```bash
R0#configure terminal

# ตั้ง IP Helper สำหรับ VLAN 10
R0(config)#interface gigabitEthernet 0/0.10
R0(config-subif)#ip helper-address 192.168.30.10
R0(config-subif)#exit

# ตั้ง IP Helper สำหรับ VLAN 20
R0(config)#interface gigabitEthernet 0/0.20
R0(config-subif)#ip helper-address 192.168.30.10
R0(config-subif)#exit

# ตั้ง IP Helper สำหรับ VLAN 30
R0(config)#interface gigabitEthernet 0/0.30
R0(config-subif)#ip helper-address 192.168.30.10
R0(config-subif)#exit

R0(config)#exit
R0#write memory
```

---

## ส่วนที่ 9: ตั้งค่า PC ให้ใช้ DHCP

### สำหรับ PC ทั้งหมด (PC0-PC5)
1. คลิกที่ PC
2. เลือกแท็บ **Desktop** > **IP Configuration**
3. เลือก **DHCP**
4. รอจนได้ IP Address อัตโนมัติ

**ผลลัพธ์ที่คาดหวัง**:
- PC0, PC3 (VLAN 10): ได้ IP `192.168.10.x`
- PC1, PC4 (VLAN 20): ได้ IP `192.168.20.x`
- PC2, PC5 (VLAN 30): ได้ IP `192.168.30.x`

---

## ส่วนที่ 10: ทดสอบการเชื่อมต่อ

### ทดสอบ 1: PC ใน VLAN เดียวกัน

**จาก PC0 (VLAN 10)** ping ไป **PC3 (VLAN 10)**:
```bash
ping 192.168.10.x
```
✅ ควรจะ ping ผ่าน (ไม่ต้องผ่าน Router)

### ทดสอบ 2: PC ต่าง VLAN

**จาก PC0 (VLAN 10)** ping ไป **PC1 (VLAN 20)**:
```bash
ping 192.168.20.x
```
✅ ควรจะ ping ผ่าน (ผ่าน Router - Inter-VLAN Routing)

### ทดสอบ 3: Ping Gateway

**จาก PC0**:
```bash
ping 192.168.10.1
```
✅ ควรจะ ping ผ่าน (Gateway ของ VLAN 10)

### ทดสอบ 4: Traceroute ข้าม VLAN

**จาก PC0 ไป PC4**:
```bash
tracert 192.168.20.x
```

**ผลลัพธ์**:
```
1   <1 ms   192.168.10.1    (Gateway VLAN 10)
2   <1 ms   192.168.20.x    (PC4)
```

---

## ส่วนที่ 11: คำสั่งตรวจสอบ VLAN

### บน Switch

```bash
# แสดง VLAN ทั้งหมด
SW0#show vlan brief

# แสดง Trunk Port
SW0#show interfaces trunk

# แสดง VLAN ของ Interface ที่ระบุ
SW0#show interfaces fastEthernet 0/2 switchport

# แสดง MAC Address Table ตาม VLAN
SW0#show mac address-table vlan 10
```

### บน Router

```bash
# แสดง Sub-interface
R0#show ip interface brief

# แสดง Routing Table
R0#show ip route

# แสดงข้อมูล Sub-interface ละเอียด
R0#show interface gigabitEthernet 0/0.10
```

---

## ส่วนที่ 12: Security - Port Security

### เพิ่ม Port Security บน Switch

```bash
SW1#configure terminal

# ตั้งค่า Port Security สำหรับ Fa0/2
SW1(config)#interface fastEthernet 0/2
SW1(config-if)#switchport port-security
SW1(config-if)#switchport port-security maximum 1
SW1(config-if)#switchport port-security mac-address sticky
SW1(config-if)#switchport port-security violation shutdown
SW1(config-if)#exit

# ดู Port Security
SW1#show port-security interface fastEthernet 0/2
```

**คำอธิบาย**:
- `maximum 1`: อนุญาตให้เชื่อมต่อได้ 1 MAC Address
- `mac-address sticky`: จำ MAC Address อัตโนมัติ
- `violation shutdown`: ปิด port ถ้ามี MAC อื่นพยายามเชื่อมต่อ

---

## ส่วนที่ 13: คำถามท้ายบท

1. **อธิบายความแตกต่างระหว่าง Access Port และ Trunk Port**
2. **ทำไม PC ใน VLAN ต่างกันไม่สามารถสื่อสารกันได้โดยตรง?**
3. **Inter-VLAN Routing คืออะไร? มีกี่วิธีในการทำ?**
4. **Native VLAN คืออะไร? ทำไมต้องตั้ง?**
5. **IP Helper Address ทำหน้าที่อะไร?**
6. **อธิบายคำสั่ง `encapsulation dot1Q 10`**
7. **VLAN ช่วยเพิ่มความปลอดภัยได้อย่างไร?**
8. **ถ้าต้องการเพิ่ม VLAN 40 ใหม่ ต้องทำอย่างไรบ้าง?**

---

## ส่วนที่ 14: การส่งงาน

### สิ่งที่ต้องส่ง:
1. ไฟล์ `.pkt` ของ Lab นี้ (ตั้งชื่อว่า `Lab4-YourName.pkt`)
2. Screenshot ของ:
   - Network Topology ที่สมบูรณ์พร้อม VLAN
   - คำสั่ง `show vlan brief` จาก Switch ทั้ง 3 ตัว
   - คำสั่ง `show interfaces trunk` จาก Switch
   - คำสั่ง `show ip interface brief` จาก Router
   - การแสดง IP Address จาก PC แต่ละตัว (ที่ได้จาก DHCP)
   - ผลการ ping ข้าม VLAN (PC0 → PC4)
   - ผลการ tracert ข้าม VLAN
3. เอกสาร Word/PDF:
   - บันทึกขั้นตอนการทำ Lab
   - ตอบคำถามท้ายบท
   - วาด Diagram แสดง VLAN และ IP Address ทั้งหมด
   - อธิบายการทำงานของ Inter-VLAN Routing

---

## ปัญหาที่พบบ่อย (Troubleshooting)

### ปัญหา: PC ไม่ได้ IP จาก DHCP
**วิธีแก้**:
1. ตรวจสอบว่า PC อยู่ใน VLAN ที่ถูกต้อง
2. ตรวจสอบว่า DHCP Server มี Pool สำหรับ VLAN นั้น
3. ตรวจสอบ IP Helper Address บน Router
4. ลอง Release/Renew DHCP:
   ```bash
   ipconfig /release
   ipconfig /renew
   ```

### ปัญหา: Ping ข้าม VLAN ไม่ผ่าน
**วิธีแก้**:
1. ตรวจสอบว่า Router มี Sub-interface สำหรับทุก VLAN
2. ตรวจสอบ `encapsulation dot1Q` ว่าตรงกับ VLAN ID
3. ตรวจสอบ Trunk Port ระหว่าง Switch และ Router
4. ตรวจสอบ Default Gateway ของ PC

### ปัญหา: Trunk Port ไม่ทำงาน
**วิธีแก้**:
```bash
# ตรวจสอบสถานะ Trunk
SW0#show interfaces trunk

# ตรวจสอบว่า VLAN อนุญาตผ่าน Trunk
SW0#show interfaces fastEthernet 0/1 switchport

# ลบและตั้งค่า Trunk ใหม่
SW0(config)#interface fastEthernet 0/1
SW0(config-if)#switchport mode access
SW0(config-if)#switchport mode trunk
```

---

## สรุป

ใน Lab นี้คุณได้เรียนรู้:
- ✅ แนวคิดและการใช้งาน VLAN
- ✅ การสร้างและกำหนด VLAN บน Switch
- ✅ ความแตกต่างระหว่าง Access Port และ Trunk Port
- ✅ การตั้งค่า Trunk Port ระหว่าง Switch
- ✅ Inter-VLAN Routing ด้วย Router-on-a-Stick
- ✅ การใช้ DHCP Server และ IP Helper Address
- ✅ คำสั่งตรวจสอบและแก้ไขปัญหา VLAN
- ✅ การเพิ่มความปลอดภัยด้วย Port Security

**Lab ถัดไป**: [Lab 5: Wireless Network Setup](../lab5-wireless/README.md)

---

[⬅️ กลับไปหน้าหลัก](../../README.md) | [⬅️ Lab ก่อนหน้า](../lab3-router-switch/README.md)
