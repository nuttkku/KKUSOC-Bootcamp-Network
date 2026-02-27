# Lab 6: NAT และ PAT Configuration

## วัตถุประสงค์
- เข้าใจหลักการทำงานของ NAT (Network Address Translation)
- ตั้งค่า Static NAT, Dynamic NAT และ PAT บน Cisco Router
- ทดสอบการเชื่อมต่อผ่าน NAT
- เข้าใจความสำคัญของ NAT ต่อความปลอดภัยเครือข่าย

## ระยะเวลา
45-60 นาที

## ความรู้พื้นฐานที่ต้องมี
- Lab 3: Router และ Switch Configuration
- NAT และ PAT (อ่านจาก [README.md](../../README.md#nat-และ-pat))

## อุปกรณ์ที่ใช้ใน Lab
- Router 2911 (2 เครื่อง — Router ภายใน และ Router จำลอง ISP)
- Switch 2960 (1 เครื่อง)
- PC (3 เครื่อง)
- Server (1 เครื่อง — จำลอง Web Server บน Internet)

---

## ส่วนที่ 1: ทำความเข้าใจ NAT

### ทำไมต้องใช้ NAT?

- IPv4 Address มีจำกัด (~4.3 พันล้านตัว) ไม่พอสำหรับทุกอุปกรณ์
- Private IP Address ไม่สามารถ route บน Internet ได้
- NAT ช่วยให้หลายเครื่องใช้ Public IP เดียวกันได้

**Private IP Ranges (RFC 1918)**:
| Range | Subnet Mask | ใช้งาน |
|-------|-------------|-------|
| `10.0.0.0` – `10.255.255.255` | /8 | องค์กรใหญ่ |
| `172.16.0.0` – `172.31.255.255` | /12 | องค์กรกลาง |
| `192.168.0.0` – `192.168.255.255` | /16 | บ้าน/ออฟฟิศเล็ก |

### NAT Terminology

| คำศัพท์ | ความหมาย | ตัวอย่าง |
|--------|---------|---------|
| Inside Local | Private IP ของเครื่องภายใน | 192.168.1.10 |
| Inside Global | Public IP ที่ใช้แทนเมื่อออก Internet | 203.0.113.1 |
| Outside Local | IP ของ Server ตามมุมมองภายใน | 8.8.8.8 |
| Outside Global | IP จริงของ Server ภายนอก | 8.8.8.8 |

---

## ส่วนที่ 2: สร้าง Network Topology

### Network Diagram

```
[PC0] 192.168.1.10 ─┐
[PC1] 192.168.1.20 ─┤─ SW0 ─── R0 (NAT Router) ─── R1 (ISP) ─── Server0
[PC2] 192.168.1.30 ─┘    Gi0/1: 192.168.1.1    Gi0/0: 203.0.113.1/30   203.0.113.2/30    172.16.0.1

Inside Network:          NAT Border               Outside / Internet
192.168.1.0/24           Public IP Pool           172.16.0.0/24
                         203.0.113.1-5
```

### ขั้นตอนที่ 1: วางอุปกรณ์
1. วาง Router 2911 จำนวน **2 เครื่อง** (R0=NAT Router, R1=ISP)
2. วาง Switch 2960 จำนวน **1 เครื่อง** (SW0)
3. วาง PC จำนวน **3 เครื่อง** (PC0, PC1, PC2)
4. วาง Server จำนวน **1 เครื่อง** (Server0 — จำลอง Internet)

### ขั้นตอนที่ 2: เชื่อมต่ออุปกรณ์
- PC0 FastEthernet0 → SW0 Fa0/1
- PC1 FastEthernet0 → SW0 Fa0/2
- PC2 FastEthernet0 → SW0 Fa0/3
- SW0 Fa0/24 → R0 Gi0/1
- R0 Gi0/0 → R1 Gi0/0 (ใช้สาย Copper Cross-Over หรือ Serial)
- R1 Gi0/1 → Server0 FastEthernet0

---

## ส่วนที่ 3: ตั้งค่า IP Address ทุกอุปกรณ์

### PC0, PC1, PC2

| อุปกรณ์ | IP Address | Subnet Mask | Gateway |
|---------|-----------|-------------|---------|
| PC0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC1 | 192.168.1.20 | 255.255.255.0 | 192.168.1.1 |
| PC2 | 192.168.1.30 | 255.255.255.0 | 192.168.1.1 |

### Server0 (จำลอง Web Server)
- IP: `172.16.0.10`
- Subnet Mask: `255.255.255.0`
- Gateway: `172.16.0.1`

### R1 (จำลอง ISP Router)

```bash
Router>enable
Router#configure terminal
Router(config)#hostname R1

R1(config)#interface gigabitEthernet 0/0
R1(config-if)#ip address 203.0.113.2 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#exit

R1(config)#interface gigabitEthernet 0/1
R1(config-if)#ip address 172.16.0.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#exit

R1(config)#exit
R1#write memory
```

### R0 (NAT Router) — ตั้งค่า Interface

```bash
Router>enable
Router#configure terminal
Router(config)#hostname R0

# Interface ฝั่ง Inside (LAN)
R0(config)#interface gigabitEthernet 0/1
R0(config-if)#description Inside - LAN
R0(config-if)#ip address 192.168.1.1 255.255.255.0
R0(config-if)#ip nat inside
R0(config-if)#no shutdown
R0(config-if)#exit

# Interface ฝั่ง Outside (WAN/Internet)
R0(config)#interface gigabitEthernet 0/0
R0(config-if)#description Outside - WAN
R0(config-if)#ip address 203.0.113.1 255.255.255.252
R0(config-if)#ip nat outside
R0(config-if)#no shutdown
R0(config-if)#exit
```

**สำคัญ**: ต้องระบุ `ip nat inside` และ `ip nat outside` ให้ถูกต้อง

---

## ส่วนที่ 4: ตั้งค่า PAT (NAT Overload)

PAT คือ NAT แบบที่ใช้งานบ่อยที่สุด — PC หลายเครื่องใช้ Public IP เดียว

```bash
R0#configure terminal

# สร้าง ACL กำหนด traffic ที่จะทำ NAT (วง 192.168.1.0/24)
R0(config)#access-list 1 permit 192.168.1.0 0.0.0.255

# ตั้งค่า PAT โดยใช้ IP ของ Interface Outside
R0(config)#ip nat inside source list 1 interface gigabitEthernet 0/0 overload

# เพิ่ม Default Route ออก Internet
R0(config)#ip route 0.0.0.0 0.0.0.0 203.0.113.2

R0(config)#exit
R0#write memory
```

### ทดสอบ PAT

1. จาก **PC0** ping ไปยัง Server0:
```
ping 172.16.0.10
```
✅ ควรจะ ping ผ่าน

2. ดู NAT Translation Table บน R0:
```bash
R0#show ip nat translations
```

**ผลลัพธ์ที่คาดหวัง**:
```
Pro  Inside global       Inside local        Outside local       Outside global
icmp 203.0.113.1:1       192.168.1.10:1      172.16.0.10:1       172.16.0.10:1
icmp 203.0.113.1:2       192.168.1.20:2      172.16.0.10:2       172.16.0.10:2
```

สังเกต: PC0 (192.168.1.10) และ PC1 (192.168.1.20) ใช้ Public IP เดียวกัน (203.0.113.1) แต่ **Port** ต่างกัน

---

## ส่วนที่ 5: ตั้งค่า Static NAT (Port Forwarding)

สมมติต้องการให้ภายนอกเข้าถึง Web Server ที่ IP 192.168.1.10 ผ่าน Port 80:

```bash
R0#configure terminal

# Static NAT: 203.0.113.1:8080 → 192.168.1.10:80
R0(config)#ip nat inside source static tcp 192.168.1.10 80 203.0.113.1 8080

R0(config)#exit
R0#write memory
```

### ทดสอบ Static NAT

เปิด Web Browser บน Server0 หรือ PC ภายนอก แล้วเข้า `http://203.0.113.1:8080`
ควรจะถึง Web Server ที่ 192.168.1.10 ได้

---

## ส่วนที่ 6: คำสั่งตรวจสอบ NAT

```bash
# ดู NAT Translation Table ปัจจุบัน
R0#show ip nat translations

# ดู NAT Translation แบบ Verbose
R0#show ip nat translations verbose

# ดูสถิติการทำ NAT
R0#show ip nat statistics

# ล้าง NAT Table (ใช้ debug เมื่อทดสอบ)
R0#clear ip nat translation *

# ดู interface ที่ตั้งค่า NAT
R0#show ip interface brief
```

**ผล show ip nat statistics**:
```
Total active translations: 3 (1 static, 2 dynamic; 2 extended)
Peak translations: 5, occurred 00:02:10 ago
Outside interfaces: GigabitEthernet0/0
Inside interfaces: GigabitEthernet0/1
Hits: 24  Misses: 0
```

---

## ส่วนที่ 7: NAT กับ Cybersecurity

### NAT ให้ความปลอดภัยอะไรบ้าง?

1. **ซ่อน Internal IP**: Attacker มองจากภายนอกเห็นเฉพาะ Public IP
2. **Stateful NAT**: Connection ที่เริ่มจากภายนอกโดยไม่มี Static NAT จะถูก Drop อัตโนมัติ
3. **ลด Attack Surface**: อุปกรณ์ภายในไม่ expose โดยตรงบน Internet

### NAT **ไม่ได้** ป้องกันอะไร?

- ❌ ไม่ filter malware ที่มากับ HTTP traffic
- ❌ ไม่ป้องกัน Phishing หรือ Social Engineering
- ❌ ไม่ป้องกัน Attack ที่เริ่มจากภายในเครือข่าย
- ❌ ไม่ใช่ Firewall — ต้องใช้ร่วมกันเสมอ

### การโจมตีผ่าน NAT

**NAT Slipstreaming**: Browser-based attack ที่หลอก NAT ให้เปิด port ที่ต้องการ
**Port Scanning NAT**: สแกน Public IP เพื่อหา Static NAT rules ที่ตั้งไว้

---

## ส่วนที่ 8: คำถามท้ายบท

1. **PAT ต่างจาก Static NAT อย่างไร? ให้ยกตัวอย่างการใช้งานของแต่ละประเภท**
2. **ดู NAT Table แล้วอธิบายว่า PC แต่ละเครื่องถูก translate อย่างไร**
3. **ทำไม NAT จึงไม่ใช่ Firewall? และควรใช้ร่วมกับอะไร?**
4. **Port Forwarding (Static NAT) เพิ่มความเสี่ยงอะไร? ควรระวังอะไรบ้าง?**
5. **ถ้า PC ภายในส่ง traffic ออกไป 100 connection พร้อมกัน NAT Table จะมีอะไรบ้าง?**

---

## ส่วนที่ 9: การส่งงาน

### สิ่งที่ต้องส่ง:
1. ไฟล์ `.pkt` (ตั้งชื่อ `Lab6-YourName.pkt`)
2. Screenshot ของ:
   - Network Topology
   - `show ip nat translations` หลังจาก ping จากทุก PC
   - `show ip nat statistics`
   - ผลการ ping จาก PC0, PC1, PC2 ไปยัง Server0
3. เอกสาร Word/PDF:
   - อธิบาย NAT Table ที่ได้ว่า PC แต่ละเครื่องถูก translate อย่างไร
   - ตอบคำถามท้ายบท

---

## ปัญหาที่พบบ่อย (Troubleshooting)

### ปัญหา: ping ออก Internet ไม่ได้
**วิธีแก้**:
1. ตรวจสอบ `ip nat inside` / `ip nat outside` ว่าตั้งถูก interface
2. ตรวจสอบ ACL ว่า match กับ IP ของ PC
3. ตรวจสอบ Default Route: `show ip route`
4. ดู NAT Table: `show ip nat translations` ถ้าว่างเปล่า = NAT ไม่ทำงาน

### ปัญหา: show ip nat translations ว่างเปล่า
**วิธีแก้**:
```bash
# ตรวจสอบ ACL
R0#show access-lists

# ตรวจสอบ NAT Config
R0#show running-config | include nat

# Debug NAT (ระวัง: output เยอะมาก)
R0#debug ip nat
```

---

## สรุป

ใน Lab นี้คุณได้เรียนรู้:
- ✅ หลักการทำงานของ NAT และ PAT
- ✅ การตั้งค่า PAT (NAT Overload) สำหรับเครือข่ายทั่วไป
- ✅ การตั้งค่า Static NAT สำหรับ Server ที่รับ connection จากภายนอก
- ✅ การอ่าน NAT Translation Table
- ✅ บทบาทของ NAT ต่อความปลอดภัยและข้อจำกัด

**Lab ถัดไป**: [Lab 7: ACL และ Firewall Rules](../lab7-acl/README.md)

---

[⬅️ กลับไปหน้าหลัก](../../README.md) | [⬅️ Lab ก่อนหน้า](../lab5-wireless/README.md)
