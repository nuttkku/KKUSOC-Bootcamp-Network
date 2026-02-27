# Lab 7: ACL และ Firewall Rules

## วัตถุประสงค์
- เข้าใจหลักการทำงานของ Access Control List (ACL)
- ตั้งค่า Standard ACL และ Extended ACL บน Cisco Router
- ใช้ ACL กรอง traffic ระหว่าง VLAN และออก Internet
- เข้าใจการทำงานของ Implicit Deny และลำดับการประมวลผล ACL
- ประยุกต์ใช้ ACL เพื่อความปลอดภัยเครือข่าย

## ระยะเวลา
60-90 นาที

## ความรู้พื้นฐานที่ต้องมี
- Lab 4: VLAN Configuration
- Lab 6: NAT Configuration
- Network Security Architecture (อ่านจาก [README.md](../../README.md#network-security-architecture))

## อุปกรณ์ที่ใช้ใน Lab
- Router 2911 (1 เครื่อง)
- Switch 2960 (1 เครื่อง)
- PC (4 เครื่อง — แยกตาม VLAN)
- Server (2 เครื่อง — Web Server, Admin Server)

---

## ส่วนที่ 1: ทำความเข้าใจ ACL

### ACL คืออะไร?

**ACL (Access Control List)** คือชุดกฎที่ Router/Switch ใช้ตัดสินใจว่า packet ใดจะผ่านได้หรือถูก block

### หลักการสำคัญของ ACL

1. **ลำดับการตรวจสอบ**: ตรวจจากบนลงล่าง หยุดที่กฎแรกที่ match
2. **Implicit Deny**: ทุก ACL มีกฎ `deny any any` ท้ายสุดโดยอัตโนมัติ (มองไม่เห็น)
3. **ทิศทาง**: ติด ACL บน Interface ระบุว่า `in` (traffic เข้า) หรือ `out` (traffic ออก)
4. **Wildcard Mask**: ใช้แทน Subnet Mask (ตรงข้ามกัน: 0=must match, 1=don't care)

### Wildcard Mask vs Subnet Mask

| Subnet Mask | Wildcard Mask | ความหมาย |
|-------------|---------------|---------|
| 255.255.255.255 | 0.0.0.0 | Host เดียว |
| 255.255.255.0 | 0.0.0.255 | Subnet /24 |
| 255.255.0.0 | 0.0.255.255 | Subnet /16 |
| 0.0.0.0 | 255.255.255.255 | ทุก IP (any) |

### ประเภท ACL

| ประเภท | เลขที่ | กรองตาม | ตำแหน่งที่วาง |
|--------|--------|---------|--------------|
| Standard ACL | 1-99 | Source IP เท่านั้น | ใกล้ Destination |
| Extended ACL | 100-199 | Src IP, Dst IP, Protocol, Port | ใกล้ Source |
| Named ACL | ชื่อ | เหมือน Standard/Extended | ใกล้ Source (Extended) |

---

## ส่วนที่ 2: Network Topology

### Network Diagram

```
                    R0 (192.168.x.1 Gateway)
                    Gi0/0 ─── Internet/Server
                    Gi0/1 ─── SW0

SW0 Port Layout:
Fa0/1  → PC0 (VLAN 10 - Users:   192.168.10.10)
Fa0/2  → PC1 (VLAN 10 - Users:   192.168.10.20)
Fa0/3  → PC2 (VLAN 20 - IT:      192.168.20.10)
Fa0/4  → PC3 (VLAN 30 - Servers: 192.168.30.10) [จำลอง Admin]
Fa0/10 → WebServer (VLAN 30:     192.168.30.100)
Fa0/24 → R0 Gi0/1 (Trunk)

VLAN 10: Users    192.168.10.0/24  Gateway: 192.168.10.1
VLAN 20: IT Dept  192.168.20.0/24  Gateway: 192.168.20.1
VLAN 30: Servers  192.168.30.0/24  Gateway: 192.168.30.1
```

### ขั้นตอนที่ 1: ตั้งค่า Switch (ทำตาม Lab 4)

สร้าง VLAN 10, 20, 30 และตั้งค่า Access/Trunk Port ตามที่เรียนไปแล้ว

### ขั้นตอนที่ 2: ตั้งค่า Router — Sub-interfaces

```bash
Router>enable
Router#configure terminal
Router(config)#hostname R0

# VLAN 10 - Users
R0(config)#interface gigabitEthernet 0/1.10
R0(config-subif)#encapsulation dot1Q 10
R0(config-subif)#ip address 192.168.10.1 255.255.255.0
R0(config-subif)#exit

# VLAN 20 - IT
R0(config)#interface gigabitEthernet 0/1.20
R0(config-subif)#encapsulation dot1Q 20
R0(config-subif)#ip address 192.168.20.1 255.255.255.0
R0(config-subif)#exit

# VLAN 30 - Servers
R0(config)#interface gigabitEthernet 0/1.30
R0(config-subif)#encapsulation dot1Q 30
R0(config-subif)#ip address 192.168.30.1 255.255.255.0
R0(config-subif)#exit

R0(config)#interface gigabitEthernet 0/1
R0(config-if)#no shutdown
R0(config-if)#exit
```

---

## ส่วนที่ 3: Standard ACL

**Standard ACL** กรองตาม Source IP เท่านั้น — วางใกล้ Destination

### Scenario: ปิดไม่ให้ VLAN 10 เข้าถึง Server VLAN 30

Policy: `VLAN 10 (Users) → VLAN 30 (Servers): DENY`

```bash
R0#configure terminal

# สร้าง Standard ACL หมายเลข 10
R0(config)#access-list 10 deny   192.168.10.0 0.0.0.255
R0(config)#access-list 10 permit any

# ผูก ACL กับ Interface ของ VLAN 30 (ใกล้ Destination)
R0(config)#interface gigabitEthernet 0/1.30
R0(config-subif)#ip access-group 10 out
R0(config-subif)#exit

R0(config)#exit
R0#write memory
```

### ทดสอบ Standard ACL

```bash
# จาก PC0 (VLAN 10) ping ไป WebServer (VLAN 30) — ควร FAIL
ping 192.168.30.100

# จาก PC2 (VLAN 20) ping ไป WebServer (VLAN 30) — ควร PASS
ping 192.168.30.100
```

---

## ส่วนที่ 4: Extended ACL

**Extended ACL** กรองได้ละเอียดกว่า: Source IP, Destination IP, Protocol, Port

### Syntax

```
access-list [100-199] [permit|deny] [protocol] [src] [wildcard] [dst] [wildcard] [eq port]
```

### Scenario: Security Policy สำหรับองค์กร

| Policy | กฎ |
|--------|----|
| Users (VLAN 10) เข้าเว็บได้ (HTTP/HTTPS) | permit tcp VLAN10 any eq 80/443 |
| Users ห้าม telnet/ssh | deny tcp VLAN10 any eq 22/23 |
| IT (VLAN 20) เข้าถึงทุกอย่างได้ | permit ip VLAN20 any |
| ทุก VLAN ห้ามเข้า Server Port 3306 | deny tcp any Server eq 3306 |

### ตั้งค่า Extended ACL

```bash
R0#configure terminal

# Named Extended ACL สำหรับ VLAN 10 (Users)
R0(config)#ip access-list extended USERS-POLICY
R0(config-ext-nacl)#remark === USERS VLAN 10 POLICY ===
R0(config-ext-nacl)#permit tcp 192.168.10.0 0.0.0.255 any eq 80
R0(config-ext-nacl)#permit tcp 192.168.10.0 0.0.0.255 any eq 443
R0(config-ext-nacl)#permit tcp 192.168.10.0 0.0.0.255 any eq 53
R0(config-ext-nacl)#permit udp 192.168.10.0 0.0.0.255 any eq 53
R0(config-ext-nacl)#permit icmp 192.168.10.0 0.0.0.255 any
R0(config-ext-nacl)#deny   tcp 192.168.10.0 0.0.0.255 any eq 22
R0(config-ext-nacl)#deny   tcp 192.168.10.0 0.0.0.255 any eq 23
R0(config-ext-nacl)#deny   tcp 192.168.10.0 0.0.0.255 any eq 3389
R0(config-ext-nacl)#deny   ip  192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
R0(config-ext-nacl)#exit

# ผูก ACL กับ Interface VLAN 10 (ใกล้ Source — direction IN)
R0(config)#interface gigabitEthernet 0/1.10
R0(config-subif)#ip access-group USERS-POLICY in
R0(config-subif)#exit

# Named Extended ACL สำหรับ Servers — ป้องกัน Database
R0(config)#ip access-list extended PROTECT-DB
R0(config-ext-nacl)#remark === Block direct DB access ===
R0(config-ext-nacl)#deny   tcp any 192.168.30.0 0.0.0.255 eq 3306
R0(config-ext-nacl)#deny   tcp any 192.168.30.0 0.0.0.255 eq 5432
R0(config-ext-nacl)#permit ip any any
R0(config-ext-nacl)#exit

# ผูกกับ Interface VLAN 30 (direction IN — กรอง traffic ที่จะเข้า Server)
R0(config)#interface gigabitEthernet 0/1.30
R0(config-subif)#ip access-group PROTECT-DB in
R0(config-subif)#exit

R0(config)#exit
R0#write memory
```

---

## ส่วนที่ 5: ทดสอบ ACL

### ทดสอบที่ 1: Users เข้า Web Server ได้

จาก **PC0 (VLAN 10)** เปิด Web Browser ไปที่ `http://192.168.30.100`

✅ ควรจะเข้าถึงได้ (HTTP port 80 ถูก permit)

### ทดสอบที่ 2: Users ห้าม SSH

จาก **PC0 (VLAN 10)** เปิด Command Prompt:
```
telnet 192.168.30.100
```
❌ ควรจะถูก block (Telnet port 23 ถูก deny)

### ทดสอบที่ 3: IT ทำได้ทุกอย่าง

จาก **PC2 (VLAN 20)** ping ไป Server, เข้า Web — ทุกอย่างควร pass

### ทดสอบที่ 4: ดูสถิติ ACL

```bash
R0#show access-lists
```

**ผลลัพธ์**:
```
Extended IP access list USERS-POLICY
    10 permit tcp 192.168.10.0 0.0.0.255 any eq www (15 matches)
    20 permit tcp 192.168.10.0 0.0.0.255 any eq 443 (8 matches)
    ...
    60 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255 (3 matches)
```

ตัวเลข `(X matches)` แสดงว่ากฎนั้น match กี่ครั้ง

---

## ส่วนที่ 6: ACL Troubleshooting

### คำสั่งตรวจสอบ

```bash
# ดู ACL ทั้งหมดพร้อม match count
R0#show access-lists

# ดู ACL ที่ผูกกับ interface
R0#show ip interface gigabitEthernet 0/1.10

# ล้าง ACL counters เพื่อทดสอบใหม่
R0#clear ip access-list counters

# ดู ACL รายการเดียว
R0#show ip access-lists USERS-POLICY
```

### ข้อผิดพลาดที่พบบ่อย

| ปัญหา | สาเหตุ | แก้ไข |
|-------|-------|-------|
| Block ทุกอย่าง | ลืม `permit any` ท้าย / Implicit Deny | เพิ่ม permit ก่อน apply |
| ไม่ block อะไรเลย | ผูก ACL ผิด interface หรือผิด direction | ตรวจ `show ip interface` |
| Block มากเกินไป | ลำดับกฎผิด — deny อยู่ก่อน permit | ต้องลบ ACL แล้วสร้างใหม่ตามลำดับ |

### แก้ไข ACL โดยไม่ต้องลบใหม่ (Named ACL)

```bash
R0#configure terminal
R0(config)#ip access-list extended USERS-POLICY

# ดู sequence number ปัจจุบัน
R0(config-ext-nacl)#do show ip access-lists USERS-POLICY

# ลบเฉพาะ entry ที่ต้องการแก้
R0(config-ext-nacl)#no 30

# เพิ่ม entry ใหม่ที่ sequence number ที่ต้องการ
R0(config-ext-nacl)#30 permit tcp 192.168.10.0 0.0.0.255 any eq 8080
R0(config-ext-nacl)#exit
```

---

## ส่วนที่ 7: ACL กับ Cybersecurity

### สิ่งที่ ACL ทำได้

- ✅ กรอง traffic ตาม IP/Port ได้ละเอียด
- ✅ ป้องกันการ scan Port ที่ไม่จำเป็น
- ✅ จำกัดสิทธิ์การเข้าถึง Server ตาม VLAN
- ✅ Block Protocol ที่ไม่ปลอดภัย (Telnet, FTP)

### สิ่งที่ ACL ทำไม่ได้

- ❌ ไม่เข้าใจ Application Layer (ไม่รู้ว่า HTTP request นั้น malicious หรือไม่)
- ❌ ไม่ตรวจสอบ Payload ของ Packet
- ❌ ไม่ติดตาม State ของ Connection (Stateless)
- ❌ ต้องการ NGFW/WAF สำหรับ Deep Inspection

### ตัวอย่าง ACL Security Policy จริง

```bash
# ปิดไม่ให้ใครเข้า Management Interface นอกจาก Jump Server
ip access-list extended PROTECT-MGMT
 permit tcp host 192.168.20.5 any eq 22    ! Jump Server SSH
 permit tcp host 192.168.20.5 any eq 443   ! Jump Server HTTPS
 deny   tcp any any eq 22 log              ! Block SSH จากทุกที่ + Log
 deny   tcp any any eq 23 log              ! Block Telnet + Log
 permit ip any any
```

---

## ส่วนที่ 8: คำถามท้ายบท

1. **อธิบายความแตกต่างระหว่าง Standard ACL กับ Extended ACL ทั้งในแง่ความสามารถและตำแหน่งที่วาง**
2. **ทำไม Extended ACL จึงควรวางใกล้ Source?**
3. **Implicit Deny คืออะไร? ทำไมต้องระวัง?**
4. **ถ้าต้องการอนุญาตเฉพาะ Host 192.168.1.50 ต้องเขียน ACL อย่างไร?**
5. **ACL entry: `deny tcp any any eq 23 log` — `log` มีประโยชน์อย่างไร?**
6. **ทำไม Named ACL ดีกว่า Numbered ACL?**
7. **ออกแบบ ACL Policy สำหรับ:**
   - HR VLAN เข้าถึง HR Server ได้เท่านั้น
   - IT VLAN เข้าถึงได้ทุกอย่าง
   - Guest VLAN เข้า Internet ได้เท่านั้น ห้ามเข้า Internal

---

## ส่วนที่ 9: การส่งงาน

### สิ่งที่ต้องส่ง:
1. ไฟล์ `.pkt` (ตั้งชื่อ `Lab7-YourName.pkt`)
2. Screenshot ของ:
   - Network Topology
   - `show access-lists` หลังจากทดสอบ (พร้อม match count)
   - ผลการทดสอบแต่ละ scenario (pass/fail พร้อม screenshot)
3. เอกสาร Word/PDF:
   - อธิบาย ACL Policy ที่ตั้งค่าไว้ทั้งหมด
   - ออกแบบ ACL Policy ใหม่ตามโจทย์ข้อ 7

---

## สรุป

ใน Lab นี้คุณได้เรียนรู้:
- ✅ หลักการทำงานของ ACL และ Implicit Deny
- ✅ ความแตกต่างระหว่าง Standard และ Extended ACL
- ✅ การตั้งค่า Named Extended ACL แบบละเอียด
- ✅ การผูก ACL กับ Interface ทั้ง in/out direction
- ✅ การตรวจสอบและแก้ไข ACL
- ✅ ข้อจำกัดของ ACL เทียบกับ NGFW

**Lab ถัดไป**: [Lab 8: OSPF Dynamic Routing](../lab8-ospf/README.md)

---

[⬅️ กลับไปหน้าหลัก](../../README.md) | [⬅️ Lab ก่อนหน้า](../lab6-nat/README.md)
