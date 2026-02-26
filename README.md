# KKUSOC Bootcamp - Network Infrastructure Training

## การอบรมเชิงปฏิบัติการเพื่อพัฒนางานโครงสร้างพื้นฐานระบบดิจิทัล

Digital Infrastructure Services
Office of Digital Technology, Khon Kaen University

---

## สารบัญ

1. [OSI Model และ TCP/IP](#osi-model-และ-tcpip)
2. [IP Address และ Network Fundamentals](#ip-address-และ-network-fundamentals)
3. [สายเครือข่าย (Network Cables)](#สายเครือข่าย-network-cables)
4. [องค์ประกอบของระบบเครือข่าย](#องค์ประกอบของระบบเครือข่าย)
5. [VLAN และการจัดการเครือข่าย](#vlan-และการจัดการเครือข่าย)
6. [ระบบเครือข่ายไร้สาย (Wireless Network)](#ระบบเครือข่ายไร้สาย-wireless-network)
7. [การวัดประสิทธิภาพเครือข่าย](#การวัดประสิทธิภาพเครือข่าย)

---

## OSI Model และ TCP/IP

### OSI 7 Layer Model

โมเดล OSI (Open Systems Interconnection) แบ่งการสื่อสารเครือข่ายออกเป็น 7 ชั้น:

| Layer | ชื่อ | หน้าที่ | ตัวอย่างอุปกรณ์/โปรโตคอล |
|-------|------|---------|--------------------------|
| 7 | Application | รับส่งข้อมูลแอปพลิเคชัน | HTTP, FTP, Telnet, DNS, DHCP |
| 6 | Presentation | จัดรูปแบบข้อมูล | Data Formatting |
| 5 | Session | สร้างและยกเลิกการเชื่อมต่อ | Connection Management |
| 4 | Transport | ส่งข้อมูลแบบ End-to-End | TCP, UDP |
| 3 | Network | กำหนดเส้นทางข้อมูล | IP, Router, Layer 3 Switch |
| 2 | Data Link | ส่งข้อมูลระหว่างอุปกรณ์ | MAC Address, Switch |
| 1 | Physical | สื่อกลางทางกายภาพ | สาย, Hub, Repeater |

### TCP/IP Model

โมเดล TCP/IP เป็นโมเดลที่ใช้งานจริงในอินเทอร์เน็ต:

- **Application Layer**: HTTP, FTP, Telnet, NTP, DHCP, PING
- **Transport Layer**: TCP, UDP
- **Network Layer**: IP, ARP, ICMP, IGMP
- **Network Interface**: Ethernet

---

## IP Address และ Network Fundamentals

### MAC Address

- **MAC Address** คือหมายเลขประจำเครื่อง ซึ่งจะไม่มีการเปลี่ยนแปลง
- ตัวอย่าง: `04:18:d6:20:64:67`, `00:27:22:d4:45:a0`
- เครื่องมือตรวจสอบ: http://aruljohn.com/mac.pl

### IP Address

**IP Address** คือหมายเลขประจำเครื่องคอมพิวเตอร์ที่เปลี่ยนไปตามระบบเครือข่ายที่เชื่อมต่อ

#### IPv4
- ตัวอย่าง: `169.12.254.2`, `192.168.0.1`, `172.16.0.25`, `10.10.0.254`
- ตรวจสอบ Public IP: https://whatismyipaddress.com/

#### IPv6
- ตัวอย่าง: `2001:0db8:0000:0000:0000:ff00:0042:8329`
- รูปแบบย่อ: `2001:db8::ff00:42:8329`
- ตรวจสอบ: http://ipv6-test.com/

### Subnet Mask

ตัวเลขที่ใช้ในการแบ่งกลุ่ม IP Address:

- `255.255.255.0` = Mask bit 24
- `255.255.0.0` = Mask bit 16
- `255.0.0.0` = Mask bit 8
- `255.255.254.0` = Mask bit 23
- `255.255.252.0` = Mask bit 22

### IP Gateway

หมายเลข IP ที่ทำหน้าที่เป็นทางออกสู่ระบบเครือข่ายอื่นๆ หรือออกสู่อินเทอร์เน็ต

### DNS (Domain Name System)

บริการที่ใช้เปลี่ยนการเชื่อมต่อจาก IP Address ให้เป็นชื่อที่จำได้ง่าย:

- `www.kapook.com` → `202.183.165.45`
- `www.kku.ac.th` → `202.12.97.4`

### เครื่องมือตรวจสอบเครือข่าย

**Subnet Calculator**: http://www.subnet-calculator.com

---

## สายเครือข่าย (Network Cables)

### ประเภทของสายที่ใช้ในระบบเครือข่าย

| ประเภทสาย | ระยะทาง | การนำไปใช้งาน |
|-----------|---------|---------------|
| Console | - | เชื่อมต่ออุปกรณ์เครือข่ายเพื่อตั้งค่า |
| UTP (CAT6) | ไม่เกิน 100 เมตร | เชื่อมต่อระหว่างอุปกรณ์เครือข่ายกับอุปกรณ์ใช้งาน |
| Fiber Optic Multi mode (MM) | ไม่เกิน 550m - 2km | เชื่อมต่ออุปกรณ์ภายในอาคาร |
| Fiber Optic Single mode (SM) | มากกว่า 2km ขึ้นไป | เชื่อมต่อระหว่างอาคาร |

### มาตรฐานสาย UTP

| มาตรฐานสาย | ประเภทของสาย | การนำไปใช้งาน |
|------------|--------------|---------------|
| CAT 5e | UTP | PC, NB, AP ไม่มี PoE |
| CAT 6 | UTP | AP PoE, PC, NB |
| CAT 6A | U/FTP, F/UTP | AP PoE, Data Center |
| CAT 7 | F/FTP, S/FTP | 10G Data Center |
| CAT 7A | F/FTP, S/FTP | 100G Data Center |

### มาตรฐานสาย EIA/TIA T568B

การเรียงสายแลน (Ethernet Patch Cable):

| Pin | สี |
|-----|-----|
| 1 | ส้ม-ขาว |
| 2 | ส้ม |
| 3 | เขียว-ขาว |
| 4 | น้ำเงิน |
| 5 | น้ำเงิน-ขาว |
| 6 | เขียว |
| 7 | น้ำตาล-ขาว |
| 8 | น้ำตาล |

### สายสัญญาณ Fiber Optic

- **Multi-mode**: สำหรับระยะทางสั้น (550m-2km)
- **Single-mode**: สำหรับระยะทางไกล (>2km)

---

## องค์ประกอบของระบบเครือข่าย

### อุปกรณ์หลักในระบบเครือข่าย

1. **Firewall & Security** - ป้องกันและควบคุมความปลอดภัย
2. **Router** - กำหนดเส้นทางข้อมูล
3. **Authentication Server** - ตรวจสอบสิทธิ์ผู้ใช้งาน
4. **Layer 3 Switch** - สวิตช์ที่ทำงาน Routing
5. **Access Switch** - สวิตช์สำหรับเชื่อมต่ออุปกรณ์ปลายทาง
6. **Access Point** - จุดเชื่อมต่อไร้สาย

### หมายเลข Port บน TCP และ UDP

| Port | Protocol | Service | Note |
|------|----------|---------|------|
| 21 | TCP, UDP | FTP | File Transfer Protocol |
| 22 | TCP, UDP | SSH, SFTP | Secure Shell |
| 23 | TCP, UDP | Telnet | Unencrypted text |
| 25 | TCP, UDP | SMTP | Simple Mail Transfer Protocol |
| 53 | TCP, UDP | DNS | Domain Name System |
| 80 | TCP, UDP | HTTP | Hypertext Transfer Protocol |
| 123 | TCP, UDP | NTP | Network Time Protocol |
| 443 | TCP, UDP | HTTPS | HTTP with TLS/SSL |
| 3306 | TCP, UDP | MySQL | MySQL Database System |

รายละเอียดเพิ่มเติม: https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers

---

## VLAN และการจัดการเครือข่าย

### VLAN คืออะไร?

**VLAN (Virtual LAN)** คือเครือข่าย LAN เสมือน เพื่อทำการแบ่ง group port ของ switch และจำกัดการเกิด broadcast domain ให้อยู่ในวงที่จำกัด

### การแบ่ง VLAN

ควรคำนึงถึง:
- แบ่งตามกลุ่มผู้ใช้งาน
- แบ่งเพื่อจำกัด Security ของข้อมูล
- แบ่งเพื่อจำกัด Broadcast domain

### IP Management Switch

IP Management Switch ใช้ในการบริหารจัดการอุปกรณ์ Switch โดย:
- แยก Traffic การควบคุมอุปกรณ์ออกจากการใช้งานปกติของ User
- สามารถบริหารจัดการอุปกรณ์ได้อย่างปลอดภัย
- สามารถเข้าถึงอุปกรณ์ได้แม้กรณีเกิด Loop ในวงของ User

### Access Mode และ Trunk Mode

- **Access Mode**: ทำให้อุปกรณ์ปลายทางเข้าไปอยู่ใน Network เดียวกันกับเลข VLAN
- **Trunk Mode**: การส่งผ่าน VLAN หลายๆ VLAN บน interface เดียว

### IP Helper (DHCP Relay)

ช่วยให้สามารถเรียกใช้งาน DHCP Server จาก Server ที่อยู่ในระบบเพียงตัวเดียว แต่ให้บริการวงเครือข่ายได้ทุกวงภายในระบบ

---

## ระบบเครือข่ายไร้สาย (Wireless Network)

### มาตรฐานระบบเครือข่ายไร้สาย (IEEE 802.11)

| Standard | Release | 2.4 GHz | 5 GHz | 6 GHz | Bandwidth | MIMO | Data Rate | MU-MIMO |
|----------|---------|---------|-------|-------|-----------|------|-----------|---------|
| 802.11n | 2009 | ✓ | ✓ | - | 20, 40 | 4 | 600 Mbps | - |
| 802.11ac Wave 1 | 2012 | - | ✓ | - | 20, 40, 80 | 3 | 1.3 Gbps | - |
| 802.11ac Wave 2 | 2016 | - | ✓ | - | 20, 40, 80, 160 | 4 | 3.46 Gbps | DL |
| 802.11ax (Wi-Fi 6) | 2018 | ✓ | ✓ | ✓ | 80+80 | 8 | 10.53 Gbps | UL/DL |
| 802.11be (Wi-Fi 7) | 2024 | ✓ | ✓ | ✓ | 160+160 | 16 | ~40 Gbps | UL/DL |

### ช่องสัญญาณ 2.4 GHz

ในประเทศไทยใช้ได้ Channel 1-13 (ส่วนใหญ่แนะนำให้ใช้ Channel 1, 6, 11 เพื่อหลีกเลี่ยงการรบกวนกัน)

### ช่องสัญญาณ 5 GHz

แบ่งเป็น:
- **UNII-1**: 36, 40, 44, 48
- **UNII-2**: 52, 56, 60, 64
- **UNII-2 Extended**: 100, 104, 108, 112, 116, 120, 124, 128, 132, 136, 140, 144
- **UNII-3**: 149, 153, 157, 161, 165

### กฎหมายควบคุมกำลังส่งสัญญาณไร้สาย (ประเทศไทย)

| ช่วงความถี่ (GHz) | กำลังส่ง E.I.R.P (W) | กำลังส่ง (dBm) |
|------------------|---------------------|---------------|
| 2.400 - 2.500 | 0.1 | 20 |
| 5.150 - 5.350 | 0.2 | 23 |
| 5.470 - 5.725 | 1.0 | 30 |
| 5.725 - 5.850 | 1.0 | 30 |

ที่มา: สำนักงาน กสทช.

### การบริหารจัดการ Wireless Access Point (WAP)

1. **Standalone**: WAP ทำงานด้วยตัวเอง ต้องตั้งค่าทีละตัว
2. **Hardware Controller**: AP ถูกควบคุมด้วย Hardware Box
3. **Software/Virtual Controller**: ใช้ Software ในการควบคุม AP
4. **Build-in Virtual Controller**: AP ตัวใดตัวหนึ่งเป็น Controller
5. **Cloud Controller**: บริหารจัดการผ่าน Cloud Service

### การเข้ารหัสความปลอดภัยของระบบไร้สาย

1. **Open Authentication** - ไม่มีการเข้ารหัส
2. **Hotspot Authentication** - ระบุตัวตนผ่าน Web Portal
3. **WEP** - เก่าและไม่ปลอดภัย (ไม่แนะนำ)
4. **WPA Pre-shared Key** - ใช้รหัสผ่านร่วมกัน
5. **WPA2 Pre-shared Key** - ปลอดภัยกว่า WPA
6. **WPA Enterprise (802.1x with RADIUS)** - ใช้ Username/Password
7. **WPA2 Enterprise (802.1x with RADIUS)** - ปลอดภัยที่สุด แนะนำสำหรับองค์กร

### การจ่ายไฟให้อุปกรณ์ Wireless Access Point

1. จ่ายไฟผ่าน Adapter
2. Passive PoE with adapter (ปล่อยไฟเท่ากับ Adapter)
3. Passive PoE 24V
4. **IEEE 802.3af** - 48V, 15.4W
5. **IEEE 802.3at (PoE+)** - 48V/54V, 30W
6. **IEEE 802.3bt** - 50-57V, 60-100W

---

## การวัดประสิทธิภาพเครือข่าย

### Speedtest คืออะไร?

Speedtest ทำงานโดยการส่ง Packet และเพิ่มขนาดของ Packet ให้ใหญ่ขึ้นเรื่อยๆ ผ่านช่องทางการสื่อสารที่มีอยู่ "จากต้นทาง ไปถึงปลายทาง"

### ระหว่างทางของ Speedtest ผ่านอะไรบ้าง?

1. เครื่องคอมพิวเตอร์ของท่าน
2. ไฟร์วอลล์ของโรงเรียน
3. เราเตอร์ของ ISP (ระดับโรงเรียน)
4. เราเตอร์ของ ISP (ระดับ ISP 1-2)
5. เราเตอร์ของผู้ให้บริการอินเตอร์เน็ตต่างประเทศ
6. โพสบนกลางแห่งจังหวัด ทรวงพัฒน
7. Speedtest Server

### คำสั่งตรวจสอบเส้นทาง

```bash
# Windows
tracert xxx.xxx.com

# Linux/Mac
traceroute xxx.xxx.com
```

### เว็บไซต์ Speedtest แนะนำ

**โดยผู้ให้บริการอินเทอร์เน็ต (ISP)**:
- http://speedtest1.totbroadband.com/
- http://speedtest.3bb.co.th/
- http://speedtest.trueinternet.co.th/
- http://catspeedtest.net/

**โดยผู้ให้บริการรายอื่นๆ**:
- https://www.nperf.com/en/
- http://speedtest.adslthailand.com/
- http://www.speedtest.net/

### การทดสอบ Latency

การวัด Latency คือการวัดว่าจากเครื่องของเราไปหา Server ของ Service นั้นๆ ใช้เวลาเท่าไหร่ Server จึงจะตอบข้อมูลกลับมา

```bash
ping xxx.xxx.com
```

---

## ค่าที่จำเป็นต้องปรับจูน

การตั้งค่าที่สำคัญสำหรับ Wireless Access Point:

1. **TX Power** - กำลังส่งสูงสุดที่ทำได้
2. **Band** - ย่านความถี่ที่จะให้บริการ (2.4GHz, 5GHz)
3. **Channel Width** - ความกว้างของช่องสัญญาณ (20, 40, 80, 160 MHz)
4. **Channel/Frequency** - ย่านความถี่หรือช่องสัญญาณ
5. **Support Rate** - ความเร็วที่จะให้บริการแก่ผู้ใช้งาน
6. **Minimum RSSI** - ค่าสัญญาณต่ำที่สุดที่จะยอมให้ใช้บริการ
7. **Country** - ประเทศที่ให้บริการ

### ทำไมจึงต้องจูนอุปกรณ์หลังการติดตั้ง?

- การทำงานของอุปกรณ์ปล่อยสัญญาณจะทำงานได้ดีที่สุดเมื่อค่าต่างๆ คงที่
- เพื่อให้ได้ค่าที่เหมาะสมต่อการให้บริการในพื้นที่นั้นๆ
- เพื่อให้ User ไม่รู้สึกว่าการใช้งานติดขัดหรือหลุดบ่อย
- จำกัดบริเวณการใช้งานให้เหมาะสม
- ลดพลังงานที่จ่ายให้กับตัวอุปกรณ์และยืดระยะเวลาการใช้งาน

---

## เครื่องมือที่ใช้ในการวิเคราะห์

### โปรแกรมสำรวจสัญญาณ Wi-Fi

- **Chanalyzer** - วิเคราะห์ช่องสัญญาณ
- **Wi-Spy DBx** - อุปกรณ์วัดสัญญาณ
- **Acrylic Wi-Fi** - สแกนเครือข่าย Wi-Fi

### โปรแกรมวิเคราะห์ Wireless Packet

- **Wireshark** - https://www.wireshark.org/
- **Eye P.A.** - วิเคราะห์ Air Time และ Packet
- **MetaGeek** - วิเคราะห์การใช้งาน Wi-Fi

---

## OLT และ ONU

### OLT (Optical Line Terminal)

อุปกรณ์ที่ติดตั้งที่ฝั่ง ISP สำหรับควบคุมและจัดการสัญญาณไฟเบอร์ออปติก

### ONU (Optical Network Unit)

อุปกรณ์ที่ติดตั้งที่ฝั่งผู้ใช้งาน สำหรับรับสัญญาณจาก OLT และแปลงเป็นสัญญาณ Ethernet

### ข้อดีของ GPON

- ระยะทางการส่งสัญญาณไกล (~20km)
- ความเร็วสูง (Up to 2.488Gbps)
- Anti-jamming & Noise สูง
- Network Architecture แบบ point-to-multipoint
- Maintenance costs ต่ำ

### PON Standard Variance

| Technology | DS/US (Gbps) | Split Ratio | Standard |
|------------|--------------|-------------|----------|
| GPON | DS: 2.488, US: 1.244 | 1:128 | G.984 (2004) |
| XG-PON1 | DS: 9.95328, US: 2.48832 | 1:256 | G.987 (2012) |
| XGS-PON | DS: 9.95328, US: 9.95328 | 1:128 to 1:256 | G.9807.1 (2016) |
| NG-PON2 | 4 to 10.2-5/ 40.2 | 1:256 | G.989 (2013) |

---

## IoT และ Smart Home

### Internet of Things (IoT)

IoT คือการเชื่อมต่ออุปกรณ์ต่างๆ เข้ากับอินเทอร์เน็ต เช่น:

- **Personal IoT**: Smartphones, Wearables, Voice Assistants
- **Home IoT**: Lighting, Climate Control, Home Appliances, Entertainment
- **Smart City**: Analytics, In-flight Services, Vehicles

### ตัวอย่างระบบ Smart Home

**Mi Home** - ระบบ Smart Home ของ Xiaomi
- Mi Control Hub
- Door/Window Sensor
- Motion Sensor
- Temperature & Humidity Sensor
- Smart Plug

---

## ข้อควรระวังด้านความปลอดภัย

### การถอดรหัสความปลอดภัยระบบไร้สาย

**คำเตือน**: การถอดรหัส Wi-Fi ของผู้อื่นโดยไม่ได้รับอนุญาตเป็นการกระทำที่ผิดกฎหมาย

เครื่องมือที่ใช้ในการทดสอบความปลอดภัย (สำหรับระบบของตัวเองเท่านั้น):
- Kali Linux
- Aircrack-ng
- Wireshark

### แนวทางปฏิบัติเพื่อความปลอดภัย

1. ใช้ WPA2 Enterprise หรือสูงกว่า
2. เปลี่ยนรหัสผ่านเป็นประจำ
3. แบ่ง VLAN แยกระหว่างผู้ใช้งานและอุปกรณ์
4. ติดตาม Log และ Monitor การใช้งาน
5. Update Firmware อุปกรณ์เป็นประจำ

---

## อ้างอิง

- http://www.ubnt.com/
- http://community.ubnt.com
- http://en.wikipedia.org/wiki/List_of_WLAN_channels
- http://en.wikipedia.org/wiki/IEEE_802.11ac
- http://en.wikipedia.org/wiki/MIMO
- http://en.wikipedia.org/wiki/Power_over_Ethernet
- https://www.wireshark.org/
- http://www.subnet-calculator.com
- https://whatismyipaddress.com/

---

## ผู้จัดทำ

**Mr. Wanut Padee**
Computer Technical Officer
(Network Engineer, System Administrator, IoT Developer)

Digital Infrastructure Services
Office of Digital Technology, Khon Kaen University

**ติดต่อ**:
- Email: wanut@kku.ac.th, wanut@kkumail.com
- Phone: 087-213-8052
- Facebook: https://www.facebook.com/wanut.padee

---

**หมายเหตุ**: เอกสารนี้จัดทำขึ้นเพื่อการศึกษาและพัฒนาบุคลากรด้าน IT เท่านั้น
