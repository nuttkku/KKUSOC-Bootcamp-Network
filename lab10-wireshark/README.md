# Lab 10: Wireshark Network Analysis

## วัตถุประสงค์
- เรียนรู้การใช้งาน Wireshark เพื่อ capture และวิเคราะห์ network traffic
- อ่านและเข้าใจโครงสร้างของ Packet แต่ละ Layer
- ใช้ Filter เพื่อค้นหา traffic ที่ต้องการ
- วิเคราะห์ TCP Three-way Handshake, ARP, DNS, HTTP
- ตรวจจับสัญญาณผิดปกติใน network traffic

## ระยะเวลา
45-60 นาที

## ความรู้พื้นฐานที่ต้องมี
- TCP/IP เชิงลึก (อ่านจาก [README.md](../../README.md#tcpip-เชิงลึก-สำหรับ-cybersecurity))
- OSI Model และ TCP/IP (อ่านจาก [README.md](../../README.md#osi-model-และ-tcpip))

## สิ่งที่ต้องเตรียม
- **Wireshark** (ติดตั้งบนเครื่องจริง — ไม่ใช่ Packet Tracer)
  - ดาวน์โหลดฟรีจาก https://www.wireshark.org/
  - รองรับ Windows, macOS, Linux
- เครื่องคอมพิวเตอร์ที่เชื่อมต่อ Network

> **หมายเหตุ**: Lab นี้ทำบนเครื่องจริงหรือ VM ที่มี Wireshark ติดตั้งแล้ว ไม่ใช้ Cisco Packet Tracer

---

## ส่วนที่ 1: ทำความรู้จัก Wireshark

### Wireshark คืออะไร?

Wireshark เป็น **Packet Analyzer** (Network Sniffer) ที่ช่วยให้มองเห็น traffic ที่วิ่งผ่าน Network Interface ของเครื่องตัวเอง

**การใช้งานที่ถูกกฎหมาย**:
- ✅ วิเคราะห์ traffic บนเครือข่ายของตัวเอง
- ✅ Debug network problems
- ✅ เรียนรู้ Protocol ต่างๆ
- ✅ Security monitoring บนระบบที่ได้รับอนุญาต
- ❌ ห้ามดักจับ traffic ของผู้อื่นโดยไม่ได้รับอนุญาต

### Interface ของ Wireshark

```
┌─────────────────────────────────────────────────────────┐
│ Menu Bar: File | Edit | View | Go | Capture | Analyze   │
├─────────────────────────────────────────────────────────┤
│ Filter Bar: [ip.addr == 192.168.1.1          ] [→]      │
├─────────────────────────────────────────────────────────┤
│ Packet List (บน):                                        │
│ No. | Time  | Source      | Dest        | Proto | Info  │
│ 1   | 0.000 | 192.168.1.10| 192.168.1.1 | TCP   | SYN   │
│ 2   | 0.001 | 192.168.1.1 | 192.168.1.10| TCP   | SYN-ACK│
├─────────────────────────────────────────────────────────┤
│ Packet Details (กลาง):                                   │
│ ▶ Frame 1: 74 bytes                                      │
│ ▶ Ethernet II                                            │
│ ▶ Internet Protocol Version 4                            │
│ ▶ Transmission Control Protocol                         │
├─────────────────────────────────────────────────────────┤
│ Packet Bytes (ล่าง): Raw Hex + ASCII                    │
└─────────────────────────────────────────────────────────┘
```

---

## ส่วนที่ 2: เริ่มต้น Capture Traffic

### ขั้นตอนที่ 1: เลือก Network Interface

1. เปิด Wireshark
2. หน้าแรกจะแสดงรายการ Interface:
   - **Ethernet** / **Wi-Fi** — Interface หลักที่เชื่อมต่ออินเทอร์เน็ต
   - **Loopback** — traffic บน localhost (127.0.0.1)
3. **ดับเบิลคลิก** บน Interface ที่ต้องการ Capture

### ขั้นตอนที่ 2: เริ่ม Capture

- คลิกปุ่ม **฿** (Shark Fin สีน้ำเงิน) หรือ `Ctrl+E`
- Wireshark จะเริ่มบันทึก Packet ทันที

### ขั้นตอนที่ 3: หยุด Capture

- คลิกปุ่ม **■** (Stop สีแดง) หรือ `Ctrl+E` อีกครั้ง

---

## ส่วนที่ 3: การใช้ Filter

Filter ช่วยกรองเฉพาะ traffic ที่ต้องการออกมาจาก noise ทั้งหมด

### Display Filter (กรองหลัง Capture)

พิมพ์ใน **Filter Bar** ด้านบน:

| Filter | ความหมาย |
|--------|---------|
| `ip.addr == 192.168.1.1` | Packet ที่เกี่ยวข้องกับ IP นี้ |
| `ip.src == 192.168.1.10` | Packet ที่ส่งมาจาก IP นี้ |
| `ip.dst == 8.8.8.8` | Packet ที่ส่งไปยัง IP นี้ |
| `tcp.port == 80` | TCP Port 80 |
| `http` | HTTP traffic ทั้งหมด |
| `dns` | DNS traffic ทั้งหมด |
| `arp` | ARP traffic ทั้งหมด |
| `tcp.flags.syn == 1` | TCP SYN Packet |
| `tcp.flags.reset == 1` | TCP RST Packet |
| `icmp` | ICMP (ping) traffic |
| `!(arp or dns)` | ทุกอย่าง ยกเว้น ARP และ DNS |

### การรวม Filter

```
# AND
ip.addr == 192.168.1.10 && tcp.port == 80

# OR
http || dns

# NOT
!ip.addr == 10.0.0.1

# ช่วง IP
ip.addr >= 192.168.1.1 && ip.addr <= 192.168.1.254
```

---

## ส่วนที่ 4: Lab Exercise 1 — วิเคราะห์ ARP

### ขั้นตอน

1. เปิด Wireshark, เลือก Interface, เริ่ม Capture
2. เปิด Command Prompt แล้วพิมพ์:
```
arp -d *        (ล้าง ARP Cache — Windows, ต้องการ Admin)
ping 192.168.1.1
```
3. หยุด Capture
4. ใช้ Filter: `arp`

### สิ่งที่จะเห็น

```
No.  Time  Source              Dest                Protocol  Info
1    0.00  YourMac             Broadcast           ARP       Who has 192.168.1.1? Tell 192.168.1.x
2    0.00  RouterMac           YourMac             ARP       192.168.1.1 is at aa:bb:cc:dd:ee:ff
```

### สิ่งที่ต้องบันทึก

1. **ARP Request**: ใครถาม? ถามหาอะไร? ส่งไป Broadcast (ff:ff:ff:ff:ff:ff)?
2. **ARP Reply**: ใครตอบ? MAC Address ของ Gateway คืออะไร?
3. คลิก Packet แล้วขยาย **Ethernet II** — ดู Source/Destination MAC
4. ขยาย **Address Resolution Protocol** — ดู Sender/Target IP และ MAC

---

## ส่วนที่ 5: Lab Exercise 2 — วิเคราะห์ DNS

### ขั้นตอน

1. เริ่ม Capture
2. เปิด Command Prompt:
```
nslookup google.com
nslookup kku.ac.th
```
3. หยุด Capture
4. ใช้ Filter: `dns`

### สิ่งที่จะเห็น

```
No. Time Source         Dest           Protocol  Info
1   0.0  192.168.1.x    8.8.8.8        DNS       Standard query A google.com
2   0.0  8.8.8.8        192.168.1.x    DNS       Standard query response A 142.250.x.x
```

### สิ่งที่ต้องบันทึก

1. **DNS Query**: เครื่องถามอะไร? ถามไปที่ DNS Server ไหน?
2. **DNS Response**: ได้ IP Address อะไร?
3. คลิก Packet ขยาย **Domain Name System**:
   - `Transaction ID`: ใช้ match Query กับ Response
   - `Questions`: ชื่อที่ถาม
   - `Answers`: IP ที่ตอบกลับ
4. **โปรโตคอลอะไร?** TCP หรือ UDP? Port อะไร?

---

## ส่วนที่ 6: Lab Exercise 3 — วิเคราะห์ TCP Three-way Handshake

### ขั้นตอน

1. เริ่ม Capture
2. เปิด Command Prompt:
```
curl http://example.com
หรือ
telnet example.com 80
```
3. หยุด Capture
4. ใช้ Filter: `tcp.port == 80`

### สิ่งที่จะเห็น

```
No. Time Source         Dest           Protocol  Flags    Info
1   0.00 192.168.1.x    93.184.216.34  TCP       [SYN]    → Port 80
2   0.00 93.184.216.34  192.168.1.x    TCP       [SYN,ACK]← ตอบรับ
3   0.00 192.168.1.x    93.184.216.34  TCP       [ACK]    → ยืนยัน
4   0.00 192.168.1.x    93.184.216.34  HTTP      [PSH,ACK]→ GET / HTTP/1.1
5   0.00 93.184.216.34  192.168.1.x    HTTP      [PSH,ACK]← HTTP/1.1 200 OK
```

### สิ่งที่ต้องบันทึก

1. **Packet 1 (SYN)**: Sequence Number เท่าไหร่?
2. **Packet 2 (SYN-ACK)**: Sequence Number และ ACK Number เท่าไหร่? Acknowledge ค่าอะไร?
3. **Packet 3 (ACK)**: ACK Number เท่าไหร่?
4. คลิกที่ Packet แล้วขยาย **Transmission Control Protocol**:
   - Source Port / Destination Port
   - Sequence Number / Acknowledgment Number
   - Flags (สังเกต bit ที่เปิดอยู่)
   - Window Size

### Follow TCP Stream

1. คลิกขวาที่ Packet ใดก็ได้
2. เลือก **Follow → TCP Stream**
3. จะเห็น HTTP Request และ Response แบบ Human-readable

---

## ส่วนที่ 7: Lab Exercise 4 — วิเคราะห์ HTTP Traffic

### ขั้นตอน

1. เริ่ม Capture
2. เปิด Browser แล้วเข้าเว็บที่ใช้ HTTP (ไม่ใช่ HTTPS) เช่น `http://example.com`
3. หยุด Capture
4. ใช้ Filter: `http`

### สิ่งที่จะเห็น

```
No. Time Source         Dest           Protocol  Info
1   0.0  192.168.1.x    93.184.216.34  HTTP      GET / HTTP/1.1
2   0.0  93.184.216.34  192.168.1.x    HTTP      HTTP/1.1 200 OK
```

### สิ่งที่ต้องบันทึก

1. **GET Request**: URL ที่ขอ? Host header? User-Agent?
2. **Response**: HTTP Status Code? Content-Type?
3. **ดู HTTP Headers**:
   - คลิกที่ GET Packet
   - ขยาย **Hypertext Transfer Protocol**
   - สังเกต Header ที่ส่งออกไป
4. **สำคัญ**: HTTP ส่งข้อมูลแบบ Plain Text — รวมถึง Cookie และ Form Data
5. ลอง Follow HTTP Stream เพื่อเห็น Request/Response ทั้งหมด

> **บทเรียนด้าน Security**: ดูว่าข้อมูล HTTP อ่านได้ง่ายแค่ไหน — นี่คือเหตุผลที่ต้องใช้ HTTPS

---

## ส่วนที่ 8: Lab Exercise 5 — ตรวจจับความผิดปกติ

### Scenario 1: Port Scan Detection

เมื่อมี Port Scan เกิดขึ้น จะเห็น Pattern ต่อไปนี้:

```
Filter: ip.src == [IP ที่สงสัย] && tcp.flags.reset == 1

สัญญาณ:
- RST packet จำนวนมากจาก IP เดียว
- Connection ไปยัง Port ต่างๆ หลาย Port ในเวลาสั้น
- SYN ไม่มี ACK ตอบกลับ
```

**วิเคราะห์ด้วย Statistics**:
1. ไปที่เมนู **Statistics → Conversations**
2. เลือกแท็บ **TCP**
3. Sort ตาม Packets — Port ที่ถูก scan จะมี Packet จำนวนน้อยต่อ Connection

### Scenario 2: ตรวจจับ ARP ที่ผิดปกติ

```
Filter: arp

สัญญาณ ARP Spoofing:
- ARP Reply ที่ไม่มี ARP Request นำหน้า
- MAC Address เดียวอ้างว่าเป็น IP หลายตัว
- IP เดียวมี MAC หลายตัว (Gratuitous ARP flooding)
```

**Wireshark ช่วยตรวจ ARP ได้เอง**:
1. ไปที่ **Analyze → Expert Information**
2. Wireshark จะแจ้งเตือน "Duplicate IP address detected" หรือ "ARP packet storm"

### Scenario 3: ตรวจจับ Cleartext Credentials

```
Filter: http.request.method == "POST"

หา Form Data ที่มี username/password
คลิกที่ Packet → Follow HTTP Stream → ดู POST body
```

> **เป็นตัวอย่างให้เห็นว่า HTTP ไม่ปลอดภัย — ไม่ควรส่ง Credential ผ่าน HTTP**

---

## ส่วนที่ 9: Useful Wireshark Features

### Statistics Menu

| Feature | วิธีเข้าถึง | ใช้ทำอะไร |
|---------|-----------|---------|
| Protocol Hierarchy | Statistics → Protocol Hierarchy | ดูสัดส่วน Protocol ทั้งหมด |
| Conversations | Statistics → Conversations | ดูคู่ที่สื่อสารกัน |
| I/O Graph | Statistics → I/O Graph | กราฟ Traffic ตามเวลา |
| Endpoints | Statistics → Endpoints | รายการ IP ทั้งหมด |

### Expert Information

**Analyze → Expert Information** แสดงปัญหาที่ Wireshark ตรวจพบเอง:
- Errors: Malformed Packet, TCP Checksum Error
- Warnings: TCP Retransmission, Duplicate ACK
- Notes: TCP Window Full, TCP Fast Retransmission
- Chats: Connection Establish, TCP Segment Len

### Save และ Export

```
# บันทึก Capture เพื่อวิเคราะห์ภายหลัง
File → Save As → *.pcapng

# Export เฉพาะ Packet ที่ Filter
File → Export Specified Packets

# Export Object (รูป, ไฟล์ใน HTTP)
File → Export Objects → HTTP
```

---

## ส่วนที่ 10: tcpdump — Wireshark บน Command Line

สำหรับ Linux/Mac หรือ Remote Server ที่ไม่มี GUI:

```bash
# Capture บน interface eth0 แสดงบนหน้าจอ
tcpdump -i eth0

# Capture แล้วบันทึกเป็นไฟล์ (เปิดด้วย Wireshark ได้)
tcpdump -i eth0 -w capture.pcap

# กรองเฉพาะ HTTP
tcpdump -i eth0 port 80

# กรองเฉพาะ traffic จาก IP
tcpdump -i eth0 src 192.168.1.10

# กรองเฉพาะ DNS
tcpdump -i eth0 port 53

# แสดง Packet Content เป็น Hex + ASCII
tcpdump -i eth0 -XX port 80

# จำกัดจำนวน Packet ที่ Capture
tcpdump -i eth0 -c 100 -w capture.pcap
```

**เปรียบเทียบ Wireshark vs tcpdump**:

| | Wireshark | tcpdump |
|---|---|---|
| GUI | ✅ | ❌ (command line) |
| Remote | ยาก | ✅ (SSH) |
| Real-time Filter | ✅ | ✅ |
| ขนาดโปรแกรม | ใหญ่ | เล็กมาก |
| ใช้ใน Production | ไม่ค่อย | บ่อยมาก |

---

## ส่วนที่ 11: คำถามท้ายบท

1. **จาก TCP Handshake ที่ capture ได้ — Sequence Number ทำงานอย่างไร? ACK Number คำนวณมาจากอะไร?**
2. **ทำไม HTTP traffic ถึงอันตราย? ควรป้องกันอย่างไร?**
3. **ARP ทำหน้าที่อะไร? เกิดขึ้นที่ Layer ไหน? ทำไมจึงถูก Spoof ได้ง่าย?**
4. **DNS ใช้ UDP เป็นหลัก ทำไม? มีกรณีไหนที่ DNS ใช้ TCP?**
5. **จาก Expert Information ถ้าเห็น "TCP Retransmission" จำนวนมาก หมายความว่าอะไร?**
6. **อธิบาย 3 สัญญาณที่บ่งบอกว่ากำลังถูก Port Scan**

---

## ส่วนที่ 12: การส่งงาน

### สิ่งที่ต้องส่ง:
1. ไฟล์ Capture `.pcapng` จากการทดสอบอย่างน้อย 1 ไฟล์
2. Screenshot ของ:
   - ARP Request/Reply พร้อมอธิบาย Field สำคัญ
   - DNS Query/Response พร้อมอธิบาย
   - TCP Three-way Handshake พร้อมอธิบาย SYN/SYN-ACK/ACK
   - HTTP GET Request (แสดง Headers)
   - Statistics → Protocol Hierarchy
3. เอกสาร Word/PDF:
   - อธิบาย Packet ที่ capture ได้ในแต่ละ Exercise
   - ตอบคำถามท้ายบท
   - บันทึกสิ่งที่เรียนรู้จาก Network ของตัวเอง

---

## แหล่งเรียนรู้เพิ่มเติม

- [Wireshark Official Documentation](https://www.wireshark.org/docs/)
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)
- [Sample Captures (ไฟล์ตัวอย่างสำหรับฝึก)](https://wiki.wireshark.org/SampleCaptures)
- [PacketLife.net Captures](https://packetlife.net/captures/)

---

## สรุป

ใน Lab นี้คุณได้เรียนรู้:
- ✅ การใช้ Wireshark capture และ filter traffic
- ✅ การอ่าน ARP, DNS, TCP, HTTP Packet
- ✅ การวิเคราะห์ TCP Three-way Handshake
- ✅ การตรวจจับ Pattern ผิดปกติ (Port Scan, ARP Anomaly)
- ✅ เข้าใจว่า HTTP ส่งข้อมูล Plain Text — เหตุผลที่ต้องใช้ HTTPS
- ✅ tcpdump สำหรับ command-line analysis

**ยินดีด้วย! คุณทำ Lab ทั้ง 10 ครบแล้ว** — ตอนนี้คุณมีพื้นฐาน Network ที่แข็งแรงสำหรับการเรียน Cybersecurity ต่อไป

---

[⬅️ กลับไปหน้าหลัก](../../README.md) | [⬅️ Lab ก่อนหน้า](../lab9-dhcp-security/README.md)
