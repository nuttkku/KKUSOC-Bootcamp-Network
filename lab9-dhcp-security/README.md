# Lab 9: DHCP Snooping และ Network Security

## วัตถุประสงค์
- เข้าใจภัยคุกคาม Rogue DHCP Server และ ARP Spoofing
- ตั้งค่า DHCP Snooping บน Switch เพื่อป้องกัน Rogue DHCP
- ตั้งค่า Dynamic ARP Inspection (DAI) เพื่อป้องกัน ARP Poisoning
- ตั้งค่า Port Security เพื่อจำกัดอุปกรณ์ที่เชื่อมต่อ
- เข้าใจการโจมตีระดับ Layer 2 และวิธีป้องกัน

## ระยะเวลา
60-90 นาที

## ความรู้พื้นฐานที่ต้องมี
- Lab 4: VLAN Configuration
- Common Network Attacks และ Defense (อ่านจาก [README.md](../../README.md#common-network-attacks-และ-defense))

## อุปกรณ์ที่ใช้ใน Lab
- Router 2911 (1 เครื่อง — DHCP Server + Gateway)
- Switch 2960 (1 เครื่อง)
- PC (3 เครื่อง — ผู้ใช้งาน)
- PC (1 เครื่อง — จำลอง Attacker / Rogue DHCP)

---

## ส่วนที่ 1: ทำความเข้าใจการโจมตี Layer 2

### Rogue DHCP Server Attack

**Rogue DHCP** คือ DHCP Server ที่ Attacker นำมาวางใน Network โดยไม่ได้รับอนุญาต

**ผลที่เกิด**:
```
ปกติ:
Client ── DHCP Discover ──► [Legitimate DHCP Server]
          DHCP Offer ◄──── IP: 192.168.1.x, GW: 192.168.1.1

หลัง Rogue DHCP:
Client ── DHCP Discover ──► [Rogue DHCP Server] (Attacker)
          DHCP Offer ◄──── IP: 192.168.1.x, GW: 10.0.0.1 ← ปลอม!
```

เมื่อ Client ได้ Gateway ปลอม → Traffic ทั้งหมดวิ่งผ่าน Attacker → **MITM Attack**

### ARP Spoofing Attack

**ARP Spoofing** คือการส่ง ARP Reply ปลอมเพื่อเปลี่ยน ARP Cache ของเหยื่อ

```
ปกติ:
PC1 ARP Cache: 192.168.1.1 → aa:bb:cc:dd:ee:ff (MAC จริงของ Router)

หลัง ARP Spoofing:
PC1 ARP Cache: 192.168.1.1 → 11:22:33:44:55:66 (MAC ของ Attacker!)

ผลที่เกิด: Traffic ของ PC1 ที่ส่งไป Gateway → ส่งไป Attacker แทน
```

---

## ส่วนที่ 2: Network Topology

### Network Diagram

```
Router0 (DHCP Server + Gateway: 192.168.1.1)
    |
    | Gi0/0
    |
SW0 ──────────────────────────────────
 |       |          |           |
Fa0/1  Fa0/2      Fa0/3       Fa0/10
 |       |          |           |
PC0    PC1         PC2      AttackerPC
(User) (User)    (User)    (Rogue DHCP)

Trusted Port: Fa0/24 (ไปยัง Router)
Untrusted Port: Fa0/1-23 (ทุก Port ของ Client)
```

### ขั้นตอนที่ 1: ตั้งค่าพื้นฐาน

**Router0 (DHCP Server + Gateway)**:
```bash
Router>enable
Router#configure terminal
Router(config)#hostname R0

R0(config)#interface gigabitEthernet 0/0
R0(config-if)#ip address 192.168.1.1 255.255.255.0
R0(config-if)#no shutdown
R0(config-if)#exit

# ตั้งค่า DHCP Server
R0(config)#ip dhcp pool LAN-POOL
R0(dhcp-config)#network 192.168.1.0 255.255.255.0
R0(dhcp-config)#default-router 192.168.1.1
R0(dhcp-config)#dns-server 8.8.8.8
R0(dhcp-config)#exit

# Exclude IP สำหรับ Router และ Attacker
R0(config)#ip dhcp excluded-address 192.168.1.1 192.168.1.20

R0(config)#exit
R0#write memory
```

**PC0, PC1, PC2**: ตั้งเป็น DHCP (ควรได้ IP 192.168.1.21 เป็นต้นไป)

**AttackerPC**: ตั้ง Static IP `192.168.1.200` เพื่อจำลอง Attacker

---

## ส่วนที่ 3: จำลอง Rogue DHCP Attack

**ก่อนเปิด DHCP Snooping** — เรียนรู้ว่า attack เกิดขึ้นอย่างไร

### ตั้งค่า Rogue DHCP บน AttackerPC

1. คลิกที่ **AttackerPC**
2. เลือกแท็บ **Services** → **DHCP**
3. เปิด DHCP Service:
   - Pool Name: `ROGUE`
   - Default Gateway: `192.168.1.200` ← IP ของ Attacker เอง
   - DNS Server: `192.168.1.200`
   - Start IP: `192.168.1.50`
   - Max Users: 50
4. คลิก **Save** และเปิด Service

### สังเกตผล

1. ลอง **PC2** Release/Renew DHCP:
```
ipconfig /release
ipconfig /renew
```

2. สังเกต PC2 ได้ Gateway เป็น `192.168.1.200` (ของ Attacker) หรือ `192.168.1.1` (ของจริง)?

> **ขึ้นอยู่กับว่า Rogue DHCP ตอบก่อน** — ใครตอบเร็วกว่า ชนะ

---

## ส่วนที่ 4: ตั้งค่า DHCP Snooping

**DHCP Snooping** แบ่ง Port เป็น 2 ประเภท:
- **Trusted Port**: อนุญาต DHCP Reply/Offer ผ่านได้ (Port ที่ต่อกับ DHCP Server จริง)
- **Untrusted Port**: บล็อก DHCP Reply — อนุญาตเฉพาะ DHCP Request จาก Client

```bash
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW0

# เปิด DHCP Snooping ทั้ง Switch
SW0(config)#ip dhcp snooping

# เปิดสำหรับ VLAN ที่ต้องการ (VLAN 1 หรือ VLAN ที่ใช้)
SW0(config)#ip dhcp snooping vlan 1

# กำหนด Trusted Port — เฉพาะ Port ที่ต่อกับ Router/DHCP Server จริง
SW0(config)#interface fastEthernet 0/24
SW0(config-if)#ip dhcp snooping trust
SW0(config-if)#exit

# Untrusted Port — ทุก Client Port เป็น Untrusted โดย default
# ตั้ง Rate Limit เพื่อป้องกัน DHCP Flooding
SW0(config)#interface range fastEthernet 0/1 - 10
SW0(config-if-range)#ip dhcp snooping limit rate 10
SW0(config-if-range)#exit

SW0(config)#exit
SW0#write memory
```

### ทดสอบ DHCP Snooping

1. PC2 Release/Renew DHCP ใหม่ → ควรได้ Gateway `192.168.1.1` จาก Server จริงเท่านั้น
2. AttackerPC จะไม่สามารถส่ง DHCP Offer ออกได้อีก

### ตรวจสอบ DHCP Snooping

```bash
# ดู DHCP Snooping Binding Table
SW0#show ip dhcp snooping binding

# ผลลัพธ์:
# MacAddress          IpAddress        Lease(sec) Type    VLAN  Interface
# ------------------  ---------------  ---------- ------- ----  -------------------
# 00:11:22:33:44:55   192.168.1.21     86400      dynamic 1     FastEthernet0/1
# 00:aa:bb:cc:dd:ee   192.168.1.22     86400      dynamic 1     FastEthernet0/2

# ดูสถานะ DHCP Snooping
SW0#show ip dhcp snooping
```

---

## ส่วนที่ 5: ตั้งค่า Dynamic ARP Inspection (DAI)

**DAI** ใช้ DHCP Snooping Binding Table ตรวจสอบว่า ARP packet มาจาก Host ที่ถูกต้องหรือไม่

```bash
SW0#configure terminal

# เปิด DAI สำหรับ VLAN 1
SW0(config)#ip arp inspection vlan 1

# กำหนด Trusted Port (เหมือนกับ DHCP Snooping)
SW0(config)#interface fastEthernet 0/24
SW0(config-if)#ip arp inspection trust
SW0(config-if)#exit

# ตั้ง ARP Rate Limit บน Client Port
SW0(config)#interface range fastEthernet 0/1 - 10
SW0(config-if-range)#ip arp inspection limit rate 100
SW0(config-if-range)#exit

SW0(config)#exit
SW0#write memory
```

### ทดสอบ DAI

1. ลอง Ping จาก PC0 → PC1 ก่อน (ควร pass — ARP ถูกต้อง)
2. บน AttackerPC ลอง set ARP Entry ปลอม (ถ้าทำได้ใน Packet Tracer)
3. ดู DAI Log:

```bash
SW0#show ip arp inspection
SW0#show ip arp inspection statistics
```

---

## ส่วนที่ 6: ตั้งค่า Port Security

**Port Security** จำกัดจำนวน MAC Address ที่อนุญาตบนแต่ละ Port

```bash
SW0#configure terminal

# ตั้งค่า Port Security บน Port ของ PC0
SW0(config)#interface fastEthernet 0/1
SW0(config-if)#switchport mode access
SW0(config-if)#switchport port-security
SW0(config-if)#switchport port-security maximum 1
SW0(config-if)#switchport port-security mac-address sticky
SW0(config-if)#switchport port-security violation shutdown
SW0(config-if)#exit

# ทำซ้ำสำหรับ Port อื่นๆ
SW0(config)#interface range fastEthernet 0/1 - 3
SW0(config-if-range)#switchport mode access
SW0(config-if-range)#switchport port-security
SW0(config-if-range)#switchport port-security maximum 1
SW0(config-if-range)#switchport port-security mac-address sticky
SW0(config-if-range)#switchport port-security violation restrict
SW0(config-if-range)#exit

SW0(config)#exit
SW0#write memory
```

**Violation Mode Options**:

| Mode | เกิดอะไรเมื่อ MAC ผิด |
|------|---------------------|
| `shutdown` | Port ถูกปิดทันที (err-disabled) |
| `restrict` | Drop packet + Log + Counter เพิ่ม |
| `protect` | Drop packet เงียบๆ ไม่ Log |

### ตรวจสอบ Port Security

```bash
# ดูสถานะ Port Security ทุก Port
SW0#show port-security

# ดูรายละเอียด Port เดียว
SW0#show port-security interface fastEthernet 0/1

# ผลลัพธ์:
# Port Security              : Enabled
# Port Status                : Secure-up
# Violation Mode             : Shutdown
# Aging Time                 : 0 mins
# Maximum MAC Addresses      : 1
# Total MAC Addresses        : 1
# Configured MAC Addresses   : 0
# Sticky MAC Addresses       : 1
# Security Violation Count   : 0
```

### จำลอง Port Security Violation

1. ถอดสาย **AttackerPC** แล้วต่อเข้า Port ของ **PC0**
2. สังเกตว่า Port ถูก shutdown (ไฟแดง)
3. ตรวจสอบ:
```bash
SW0#show port-security interface fastEthernet 0/1
# Port Status จะเป็น Secure-shutdown
```
4. กู้คืน Port:
```bash
SW0(config)#interface fastEthernet 0/1
SW0(config-if)#shutdown
SW0(config-if)#no shutdown
```

---

## ส่วนที่ 7: BPDU Guard (ป้องกัน Rogue Switch)

**BPDU Guard** ป้องกันการนำ Switch อื่นมาเสียบเข้า Access Port

```bash
SW0#configure terminal

# เปิด BPDU Guard บน Access Port ทั้งหมด
SW0(config)#spanning-tree portfast bpduguard default

# หรือตั้งทีละ Port
SW0(config)#interface range fastEthernet 0/1 - 10
SW0(config-if-range)#spanning-tree portfast
SW0(config-if-range)#spanning-tree bpduguard enable
SW0(config-if-range)#exit

SW0(config)#exit
SW0#write memory
```

**ผล**: ถ้ามีใครนำ Switch มาเสียบ Port ที่เปิด BPDU Guard จะถูก err-disable ทันที

---

## ส่วนที่ 8: สรุป Security Features Layer 2

| Feature | ป้องกันอะไร | ตั้งค่าที่ไหน |
|---------|-----------|------------|
| DHCP Snooping | Rogue DHCP Server | Switch |
| Dynamic ARP Inspection | ARP Spoofing / Poisoning | Switch |
| Port Security | MAC Flooding, Unauthorized Access | Switch |
| BPDU Guard | Rogue Switch | Switch |
| 802.1X (EAP) | ผู้ใช้ไม่ได้รับอนุญาต | Switch + RADIUS |

### Defense-in-Depth ที่ Layer 2

```
[User Device]
    │ Port Security (MAC)
    │ BPDU Guard (No Rogue Switch)
    │ DHCP Snooping (No Rogue DHCP)
    │ DAI (No ARP Spoof)
    ▼
[Access Switch]
    │ VLAN Segregation
    │ Trunk with Native VLAN ≠ 1
    ▼
[Core/Distribution Switch/Router]
    │ ACL Between VLANs
    │ Routing Protocol Authentication
    ▼
[Firewall / NGFW]
```

---

## ส่วนที่ 9: คำถามท้ายบท

1. **อธิบายขั้นตอนที่ Rogue DHCP Attack เกิดขึ้น และผลที่ตามมา**
2. **DHCP Snooping ทำงานอย่างไร? Trusted vs Untrusted Port คืออะไร?**
3. **ทำไม DAI ต้องพึ่งพา DHCP Snooping Binding Table?**
4. **Port Security Violation Mode "restrict" vs "shutdown" — ใช้แบบไหนดีกว่า ขึ้นอยู่กับอะไร?**
5. **ถ้า Attacker ใช้ Static IP แทน DHCP DHCP Snooping และ DAI ยังป้องกันได้ไหม?**
6. **อธิบาย 5 Layer 2 Security Feature และสถานการณ์ที่ควรใช้แต่ละอัน**

---

## ส่วนที่ 10: การส่งงาน

### สิ่งที่ต้องส่ง:
1. ไฟล์ `.pkt` (ตั้งชื่อ `Lab9-YourName.pkt`)
2. Screenshot ของ:
   - `show ip dhcp snooping binding` หลังจาก PC ทั้งหมดได้ IP
   - `show ip arp inspection`
   - `show port-security` จาก Switch
   - ตอนที่ Port Security Violation เกิดขึ้น (Port สีแดง)
   - Rogue DHCP ถูก Block (PC ได้ IP จาก Server จริง)
3. เอกสาร Word/PDF:
   - อธิบาย Attack แต่ละประเภทและการป้องกัน
   - ตอบคำถามท้ายบท
   - วาด Diagram แสดง Security Feature ที่ตั้งค่า

---

## สรุป

ใน Lab นี้คุณได้เรียนรู้:
- ✅ หลักการของ Rogue DHCP Attack และ ARP Spoofing
- ✅ DHCP Snooping — ป้องกัน DHCP Server ที่ไม่ได้รับอนุญาต
- ✅ Dynamic ARP Inspection — ป้องกัน ARP Cache Poisoning
- ✅ Port Security — จำกัด MAC Address บน Access Port
- ✅ BPDU Guard — ป้องกัน Rogue Switch
- ✅ แนวคิด Defense-in-Depth ที่ Layer 2

**Lab ถัดไป**: [Lab 10: Wireshark Network Analysis](../lab10-wireshark/README.md)

---

[⬅️ กลับไปหน้าหลัก](../../README.md) | [⬅️ Lab ก่อนหน้า](../lab8-ospf/README.md)
