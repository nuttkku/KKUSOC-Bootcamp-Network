# Lab 8: OSPF Dynamic Routing

## วัตถุประสงค์
- เข้าใจหลักการทำงานของ OSPF (Open Shortest Path First)
- ตั้งค่า OSPF บน Router หลายตัวให้แลกเปลี่ยน Routing Table กันเอง
- เข้าใจ OSPF Cost และการเลือกเส้นทาง
- ตั้งค่า OSPF Authentication เพื่อความปลอดภัย
- เปรียบเทียบ Static Routing กับ Dynamic Routing ในทางปฏิบัติ

## ระยะเวลา
60-90 นาที

## ความรู้พื้นฐานที่ต้องมี
- Lab 3: Router และ Switch Configuration
- Routing Protocols เบื้องต้น (อ่านจาก [README.md](../../README.md#routing-protocols-เบื้องต้น))

## อุปกรณ์ที่ใช้ใน Lab
- Router 2911 (3 เครื่อง — R1, R2, R3)
- Switch 2960 (3 เครื่อง)
- PC (3 เครื่อง — แต่ละ site)

---

## ส่วนที่ 1: ทำความเข้าใจ OSPF

### OSPF ทำงานอย่างไร?

OSPF เป็น **Link State** routing protocol — Router แต่ละตัวสร้าง "แผนที่" ของเครือข่ายทั้งหมด แล้วคำนวณเส้นทางที่ดีที่สุด (Shortest Path) ด้วยอัลกอริทึม Dijkstra

**กระบวนการ OSPF**:
1. Router ส่ง Hello packet เพื่อค้นหา Neighbor
2. Router แลกเปลี่ยน LSA (Link State Advertisement)
3. สร้าง LSDB (Link State Database) — แผนที่ topology ทั้งหมด
4. คำนวณ Shortest Path Tree ด้วย Dijkstra
5. ใส่เส้นทางที่ดีที่สุดลงใน Routing Table

### OSPF Cost

OSPF เลือกเส้นทาง Cost ต่ำที่สุด:

```
Cost = 100,000,000 / Bandwidth (bps)
```

| Interface | Bandwidth | OSPF Cost |
|-----------|-----------|-----------|
| Serial (T1) | 1.544 Mbps | 64 |
| Ethernet | 10 Mbps | 10 |
| FastEthernet | 100 Mbps | 1 |
| GigabitEthernet | 1,000 Mbps | 1 |
| 10G Ethernet | 10,000 Mbps | 1 |

**หมายเหตุ**: FastEthernet และ GigabitEthernet มี Cost เท่ากัน (1) เนื่องจาก Reference Bandwidth คือ 100 Mbps ต้องปรับถ้าต้องการแยกแยะ

### OSPF Neighbor States

```
Down → Init → 2-Way → ExStart → Exchange → Loading → Full
```

- **Full State** = เชื่อมต่อสำเร็จ Router แลก LSDB กันครบแล้ว
- ถ้าค้างที่ Init หรือ 2-Way = มีปัญหา

---

## ส่วนที่ 2: Network Topology

### Network Diagram

```
PC0 ─── SW1 ─── R1 ─────────── R2 ─── SW2 ─── PC1
                 \              /
                  \            /
               10.0.12.0/30  10.0.12.4/30
                    \        /
                     R3 ─── SW3 ─── PC2
                    /
               10.0.13.0/30

Network Segments:
R1-R2 Link:  10.0.12.0/30   (R1: .1, R2: .2)
R1-R3 Link:  10.0.13.0/30   (R1: .1, R3: .2)
R2-R3 Link:  10.0.23.0/30   (R2: .1, R3: .2)

LANs:
Site 1 (R1): 192.168.1.0/24   PC0: 192.168.1.10
Site 2 (R2): 192.168.2.0/24   PC1: 192.168.2.10
Site 3 (R3): 192.168.3.0/24   PC2: 192.168.3.10
```

### ขั้นตอนที่ 1: วางอุปกรณ์
1. วาง Router 2911 จำนวน **3 เครื่อง** (R1, R2, R3)
2. วาง Switch 2960 จำนวน **3 เครื่อง** (SW1, SW2, SW3)
3. วาง PC จำนวน **3 เครื่อง** (PC0, PC1, PC2)

### ขั้นตอนที่ 2: เชื่อมต่ออุปกรณ์
- PC0 → SW1 Fa0/1
- SW1 Fa0/24 → R1 Gi0/0 (LAN Interface)
- R1 Gi0/1 → R2 Gi0/1 (WAN Link R1-R2)
- R1 Gi0/2 → R3 Gi0/1 (WAN Link R1-R3)
- R2 Gi0/0 → SW2 Fa0/24
- R2 Gi0/2 → R3 Gi0/2 (WAN Link R2-R3)
- R3 Gi0/0 → SW3 Fa0/24
- PC1 → SW2 Fa0/1
- PC2 → SW3 Fa0/1

---

## ส่วนที่ 3: ตั้งค่า IP Address

### PC0, PC1, PC2

| อุปกรณ์ | IP Address | Gateway |
|---------|-----------|---------|
| PC0 | 192.168.1.10/24 | 192.168.1.1 |
| PC1 | 192.168.2.10/24 | 192.168.2.1 |
| PC2 | 192.168.3.10/24 | 192.168.3.1 |

### R1 Configuration

```bash
Router>enable
Router#configure terminal
Router(config)#hostname R1

# LAN Interface
R1(config)#interface gigabitEthernet 0/0
R1(config-if)#description LAN - Site1
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#exit

# WAN Link R1-R2
R1(config)#interface gigabitEthernet 0/1
R1(config-if)#description WAN Link to R2
R1(config-if)#ip address 10.0.12.1 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#exit

# WAN Link R1-R3
R1(config)#interface gigabitEthernet 0/2
R1(config-if)#description WAN Link to R3
R1(config-if)#ip address 10.0.13.1 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#exit

R1(config)#exit
R1#write memory
```

### R2 Configuration

```bash
Router>enable
Router#configure terminal
Router(config)#hostname R2

R2(config)#interface gigabitEthernet 0/0
R2(config-if)#description LAN - Site2
R2(config-if)#ip address 192.168.2.1 255.255.255.0
R2(config-if)#no shutdown
R2(config-if)#exit

R2(config)#interface gigabitEthernet 0/1
R2(config-if)#description WAN Link to R1
R2(config-if)#ip address 10.0.12.2 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#exit

R2(config)#interface gigabitEthernet 0/2
R2(config-if)#description WAN Link to R3
R2(config-if)#ip address 10.0.23.1 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#exit

R2(config)#exit
R2#write memory
```

### R3 Configuration

```bash
Router>enable
Router#configure terminal
Router(config)#hostname R3

R3(config)#interface gigabitEthernet 0/0
R3(config-if)#description LAN - Site3
R3(config-if)#ip address 192.168.3.1 255.255.255.0
R3(config-if)#no shutdown
R3(config-if)#exit

R3(config)#interface gigabitEthernet 0/1
R3(config-if)#description WAN Link to R1
R3(config-if)#ip address 10.0.13.2 255.255.255.252
R3(config-if)#no shutdown
R3(config-if)#exit

R3(config)#interface gigabitEthernet 0/2
R3(config-if)#description WAN Link to R2
R3(config-if)#ip address 10.0.23.2 255.255.255.252
R3(config-if)#no shutdown
R3(config-if)#exit

R3(config)#exit
R3#write memory
```

---

## ส่วนที่ 4: ตั้งค่า OSPF

### R1 — เปิด OSPF

```bash
R1#configure terminal

R1(config)#router ospf 1
R1(config-router)#router-id 1.1.1.1
R1(config-router)#network 192.168.1.0 0.0.0.255 area 0
R1(config-router)#network 10.0.12.0  0.0.0.3   area 0
R1(config-router)#network 10.0.13.0  0.0.0.3   area 0
R1(config-router)#passive-interface gigabitEthernet 0/0
R1(config-router)#exit

R1(config)#exit
R1#write memory
```

**คำอธิบาย**:
- `router ospf 1`: Process ID = 1 (ใช้ภายใน Router เดียว ไม่ต้องตรงกัน)
- `router-id 1.1.1.1`: กำหนด Router ID (unique ทุกตัว)
- `network [IP] [Wildcard] area 0`: ประกาศ Network ที่ต้องการเข้าร่วม OSPF
- `passive-interface`: หยุดส่ง Hello packet บน LAN (ไม่มี Router ข้างๆ)

### R2 — เปิด OSPF

```bash
R2#configure terminal

R2(config)#router ospf 1
R2(config-router)#router-id 2.2.2.2
R2(config-router)#network 192.168.2.0 0.0.0.255 area 0
R2(config-router)#network 10.0.12.0  0.0.0.3   area 0
R2(config-router)#network 10.0.23.0  0.0.0.3   area 0
R2(config-router)#passive-interface gigabitEthernet 0/0
R2(config-router)#exit

R2(config)#exit
R2#write memory
```

### R3 — เปิด OSPF

```bash
R3#configure terminal

R3(config)#router ospf 1
R3(config-router)#router-id 3.3.3.3
R3(config-router)#network 192.168.3.0 0.0.0.255 area 0
R3(config-router)#network 10.0.13.0  0.0.0.3   area 0
R3(config-router)#network 10.0.23.0  0.0.0.3   area 0
R3(config-router)#passive-interface gigabitEthernet 0/0
R3(config-router)#exit

R3(config)#exit
R3#write memory
```

---

## ส่วนที่ 5: ตรวจสอบ OSPF

### คำสั่งตรวจสอบหลัก

```bash
# ดู OSPF Neighbor ที่ค้นพบ
R1#show ip ospf neighbor

# ผลลัพธ์ที่คาดหวัง:
# Neighbor ID    Pri  State    Dead Time  Address      Interface
# 2.2.2.2        1    FULL/DR  00:00:35   10.0.12.2    Gi0/1
# 3.3.3.3        1    FULL/DR  00:00:38   10.0.13.2    Gi0/2
```

```bash
# ดู Routing Table — route OSPF จะมีเครื่องหมาย O
R1#show ip route

# ผลลัพธ์:
# O    192.168.2.0/24 [110/2] via 10.0.12.2, 00:01:00, GigabitEthernet0/1
# O    192.168.3.0/24 [110/2] via 10.0.13.2, 00:01:00, GigabitEthernet0/2
```

```bash
# ดูรายละเอียด OSPF
R1#show ip ospf

# ดู OSPF Database (LSDB)
R1#show ip ospf database

# ดู Interface ที่ OSPF ทำงานอยู่
R1#show ip ospf interface
```

### ทดสอบ Connectivity

```bash
# จาก PC0 ping ไป PC1 และ PC2
ping 192.168.2.10
ping 192.168.3.10

# Traceroute เพื่อดูเส้นทาง
tracert 192.168.2.10
```

---

## ส่วนที่ 6: OSPF Authentication (ความปลอดภัย)

**ทำไมต้องใช้ OSPF Authentication?**
- ป้องกัน Rogue Router ที่ Attacker นำเข้ามาเพื่อ inject Route ปลอม
- ป้องกัน Route Poisoning — ส่ง LSA ปลอมเพื่อเปลี่ยน Traffic path

### ตั้งค่า MD5 Authentication บน Link R1-R2

```bash
# บน R1
R1#configure terminal
R1(config)#interface gigabitEthernet 0/1
R1(config-if)#ip ospf authentication message-digest
R1(config-if)#ip ospf message-digest-key 1 md5 SecurePass123
R1(config-if)#exit

# บน R2 (ต้องใช้ Key เดียวกัน)
R2#configure terminal
R2(config)#interface gigabitEthernet 0/1
R2(config-if)#ip ospf authentication message-digest
R2(config-if)#ip ospf message-digest-key 1 md5 SecurePass123
R2(config-if)#exit
```

**ผลที่เกิด**: Router ที่ไม่มี Key ถูกต้องจะไม่สามารถ join OSPF ได้ — Rogue Router ถูก Block

---

## ส่วนที่ 7: ทดสอบ Failover

OSPF ข้อดีคือ ถ้า Link เสีย จะหา Route ทางอื่นอัตโนมัติ

### ทดสอบ
1. Ping จาก PC0 → PC1 ให้ต่อเนื่อง
2. ปิด Interface R1-R2 Link บน R1:
```bash
R1#configure terminal
R1(config)#interface gigabitEthernet 0/1
R1(config-if)#shutdown
```
3. สังเกตว่า ping ขาดช่วงสั้นๆ แล้วกลับมา
4. ดู Routing Table ใหม่:
```bash
R1#show ip route
# 192.168.2.0 ควรเปลี่ยน path → ผ่าน R3 แทน
```
5. เปิด Interface กลับ:
```bash
R1(config)#interface gigabitEthernet 0/1
R1(config-if)#no shutdown
```

---

## ส่วนที่ 8: OSPF กับ Cybersecurity

### ภัยคุกคาม OSPF

| การโจมตี | วิธีการ | ผลที่เกิด |
|---------|---------|---------|
| Rogue Router | เอา Router เข้า network โดยไม่ได้รับอนุญาต | Traffic ถูก redirect ผ่าน Attacker |
| LSA Flooding | ส่ง LSA จำนวนมาก | กิน CPU/Memory ของ Router (DoS) |
| Route Injection | inject Route ปลอม | Traffic hijacking, MITM |

### การป้องกัน

1. **OSPF MD5 Authentication** — บังคับทุก Link ที่มี Router อื่น
2. **passive-interface** — ไม่ส่ง Hello บน Interface ที่ไม่มี Router
3. **OSPF TTL Security** — ป้องกัน Packet จาก Router ที่ไม่ใช่ Neighbor โดยตรง
4. **Route Filtering** — กรอง Route ที่รับจาก Neighbor

```bash
# ตั้งค่า OSPF TTL Security
R1(config)#router ospf 1
R1(config-router)#ttl-security all-interfaces hops 1
```

---

## ส่วนที่ 9: คำถามท้ายบท

1. **OSPF ต่างจาก RIP อย่างไร? ข้อดีข้อเสียของแต่ละ Protocol?**
2. **Router ID ใน OSPF คืออะไร? ถ้าไม่กำหนดเอง OSPF จะเลือกอย่างไร?**
3. **passive-interface มีประโยชน์อะไร ทั้งในแง่ประสิทธิภาพและความปลอดภัย?**
4. **ถ้า R2-R3 Link เสีย traffic จาก PC1 → PC2 จะไปทางไหน? อธิบายตาม Routing Table**
5. **OSPF Authentication ป้องกันอะไร? และไม่ได้ป้องกันอะไร?**
6. **[110/2] ใน Routing Table หมายความว่าอะไร? 110 คืออะไร? 2 คืออะไร?**

---

## ส่วนที่ 10: การส่งงาน

### สิ่งที่ต้องส่ง:
1. ไฟล์ `.pkt` (ตั้งชื่อ `Lab8-YourName.pkt`)
2. Screenshot ของ:
   - Network Topology
   - `show ip ospf neighbor` จากทุก Router
   - `show ip route` จากทุก Router (สังเกต route O)
   - ผลการ ping จาก PC0 → PC1, PC2
   - ผล traceroute ข้าม Router
   - `show ip route` หลังจากปิด Link (Failover Test)
3. เอกสาร Word/PDF:
   - วาด Diagram แสดงเส้นทางที่ OSPF เลือก
   - อธิบาย Failover ที่เกิดขึ้น
   - ตอบคำถามท้ายบท

---

## สรุป

ใน Lab นี้คุณได้เรียนรู้:
- ✅ หลักการทำงานของ OSPF (Link State, Cost, Neighbor)
- ✅ การตั้งค่า OSPF บน Router หลายตัวใน Area 0
- ✅ การตรวจสอบ OSPF Neighbor และ Routing Table
- ✅ OSPF Failover อัตโนมัติเมื่อ Link เสีย
- ✅ OSPF MD5 Authentication เพื่อความปลอดภัย
- ✅ ภัยคุกคามต่อ OSPF และวิธีป้องกัน

**Lab ถัดไป**: [Lab 9: DHCP Snooping และ Network Security](../lab9-dhcp-security/README.md)

---

[⬅️ กลับไปหน้าหลัก](../../README.md) | [⬅️ Lab ก่อนหน้า](../lab7-acl/README.md)
