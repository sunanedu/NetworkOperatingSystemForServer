การออกแบบและตั้งค่าเครือข่าย 2 เครือข่ายด้วย VLAN และ DHCP
เอกสารนี้จะแนะนำขั้นตอนการออกแบบและตั้งค่าเครือข่าย 2 ชุดที่เชื่อมต่อกัน โดยมีการแบ่ง VLAN สำหรับ PC และ Laptop และมีการแจกจ่าย IP Address อัตโนมัติด้วย DHCP Server บน Router

## 1. ภาพรวมของเครือข่าย
เราจะสร้าง 2 เครือข่ายที่เหมือนกัน (Network A และ Network B) ซึ่งแต่ละเครือข่ายประกอบด้วย Router, Switch, Access Point, PC, และ Laptop โดยมีข้อกำหนดดังนี้:

**VLAN**

VLAN 10 (VLAN_PC): สำหรับเครื่อง PC ทั้งสองเครือข่าย
VLAN 20 (VLAN_LAPTOP): สำหรับ Laptop ที่เชื่อมต่อผ่าน Wi-Fi ทั้งสองเครือข่าย

**Routing** Router ทั้งสองตัวจะเชื่อมต่อกันผ่านลิงก์ WAN และใช้ Static Route เพื่อให้เครือข่ายทั้งสองฝั่งสื่อสารกันได้

**DHCP** Router ในแต่ละเครือข่ายจะทำหน้าที่เป็น DHCP Server เพื่อแจก IP Address ให้กับอุปกรณ์ใน VLAN ของตนเอง

Wireless: Laptop จะเชื่อมต่อกับเครือข่ายผ่าน Access Point

## 2. การออกแบบ IP Address และ VLAN
### ตารางออกแบบ IP Address และ VLAN

| อุปกรณ์ | Interface | เครือข่าย / VLAN | IP Address | Subnet Mask | Gateway | หมายเหตุ |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **ลิงก์ระหว่าง Router (WAN)** | | | **`10.0.0.0/30`** | | | |
| R1 | `GigabitEthernet0/0` | WAN | `10.0.0.1` | `255.255.255.252` | - | |
| R2 | `GigabitEthernet0/0` | WAN | `10.0.0.2` | `255.255.255.252` | - | |
| **เครือข่ายที่ 1 (Network A)** | | | | | | |
| R1 | `GigabitEthernet0/1.10` | VLAN 10 (PC) | `192.168.10.1` | `255.255.255.0` | - | Gateway สำหรับ PC1 |
| R1 | `GigabitEthernet0/1.20` | VLAN 20 (Laptop) | `192.168.20.1` | `255.255.255.0` | - | Gateway สำหรับ Laptop1 |
| PC1 | `Ethernet0` | VLAN 10 (PC) | DHCP | DHCP | DHCP | |
| Laptop1 | `Wireless0` | VLAN 20 (Laptop) | DHCP | DHCP | DHCP | |
| **เครือข่ายที่ 2 (Network B)** | | | | | | |
| R2 | `GigabitEthernet0/1.10` | VLAN 10 (PC) | `192.168.30.1` | `255.255.255.0` | - | Gateway สำหรับ PC2 |
| R2 | `GigabitEthernet0/1.20` | VLAN 20 (Laptop) | `192.168.40.1` | `255.255.255.0` | - | Gateway สำหรับ Laptop2 |
| PC2 | `Ethernet0` | VLAN 10 (PC) | DHCP | DHCP | DHCP | |
| Laptop2 | `Wireless0` | VLAN 20 (Laptop) | DHCP | DHCP | DHCP | |

## 3. รูปแบบการเชื่อมต่ออุปกรณ์ (แบบ Text)

ใช้สาย `Copper Straight-Through` สำหรับการเชื่อมต่อส่วนใหญ่ และ `Copper Cross-Over` สำหรับการเชื่อมต่อระหว่าง Router

* **การเชื่อมต่อระหว่าง Router (WAN):**

  * `R1 (GigabitEthernet0/0)` <--> `R2 (GigabitEthernet0/0)` (ใช้สาย `Copper Cross-Over`)

* **การเชื่อมต่อเครือข่ายที่ 1 (Network A):**

  * `R1 (GigabitEthernet0/1)` <--> `SW1 (GigabitEthernet0/1)`

  * `SW1 (FastEthernet0/1)` <--> `PC1`

  * `SW1 (FastEthernet0/2)` <--> `AP1 (Port 0)`

* **การเชื่อมต่อเครือข่ายที่ 2 (Network B):**

  * `R2 (GigabitEthernet0/1)` <--> `SW2 (GigabitEthernet0/1)`

  * `SW2 (FastEthernet0/1)` <--> `PC2`

  * `SW2 (FastEthernet0/2)` <--> `AP2 (Port 0)`

## 4. คำสั่งการตั้งค่า (Configuration Commands)

### 4.1 เครือข่ายที่ 1 (Network A)

#### **Router (R1)**
```yaml
Router>enable
Router#configure terminal
Router(config)#hostname R1

! === ตั้งค่า Interface WAN ที่เชื่อมต่อไปยัง R2 ===
R1(config)#interface GigabitEthernet0/0
R1(config-if)#ip address 10.0.0.1 255.255.255.252
R1(config-if)#description Link to R2
R1(config-if)#no shutdown
R1(config-if)#exit

! === ตั้งค่า 'Router on a Stick' สำหรับ VLANs ===
R1(config)#interface GigabitEthernet0/1
R1(config-if)#no shutdown
R1(config-if)#exit

! --- Sub-interface สำหรับ VLAN 10 (PC) ---
R1(config)#interface GigabitEthernet0/1.10
R1(config-subif)#description Gateway for VLAN_PC
R1(config-subif)#encapsulation dot1Q 10
R1(config-subif)#ip address 192.168.10.1 255.255.255.0
R1(config-subif)#exit

! --- Sub-interface สำหรับ VLAN 20 (Laptop) ---
R1(config)#interface GigabitEthernet0/1.20
R1(config-subif)#description Gateway for VLAN_LAPTOP
R1(config-subif)#encapsulation dot1Q 20
R1(config-subif)#ip address 192.168.20.1 255.255.255.0
R1(config-subif)#exit

! === ตั้งค่า DHCP Server ===
! --- DHCP Pool สำหรับ VLAN 10 (PC) ---
R1(config)#ip dhcp pool VLAN10_PC
R1(dhcp-config)#network 192.168.10.0 255.255.255.0
R1(dhcp-config)#default-router 192.168.10.1
R1(dhcp-config)#exit

! --- DHCP Pool สำหรับ VLAN 20 (Laptop) ---
R1(config)#ip dhcp pool VLAN20_LAPTOP
R1(dhcp-config)#network 192.168.20.0 255.255.255.0
R1(dhcp-config)#default-router 192.168.20.1
R1(dhcp-config)#exit

! === ตั้งค่า Static Route เพื่อให้รู้จักเครือข่ายของ R2 ===
R1(config)#ip route 192.168.30.0 255.255.255.0 10.0.0.2
R1(config)#ip route 192.168.40.0 255.255.255.0 10.0.0.2

R1(config)#exit
R1#write
```

Switch (SW1)
```yaml
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW1

! === สร้าง VLANs ===
SW1(config)#vlan 10
SW1(config-vlan)#name VLAN_PC
SW1(config-vlan)#exit

SW1(config)#vlan 20
SW1(config-vlan)#name VLAN_LAPTOP
SW1(config-vlan)#exit

! === ตั้งค่าพอร์ต Trunk ที่เชื่อมต่อไปยัง R1 ===
SW1(config)#interface GigabitEthernet0/1
SW1(config-if)#description Trunk to R1
SW1(config-if)#switchport mode trunk
SW1(config-if)#exit

! === ตั้งค่าพอร์ต Access สำหรับ PC1 ===
SW1(config)#interface FastEthernet0/1
SW1(config-if)#description To PC1
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 10
SW1(config-if)#exit

! === ตั้งค่าพอร์ต Access สำหรับ AP1 ===
SW1(config)#interface FastEthernet0/2
SW1(config-if)#description To AP1
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 20
SW1(config-if)#exit

SW1(config)#exit
SW1#write
```
#### **Access Point (AP1)**

1. คลิกที่ `AP1` > แท็บ `Config`.

2. ที่เมนู `Port 1`.

3. ตั้งค่า `SSID:` เป็น `WiFi_A`

4. เลือก `Authentication` เป็น `WPA2-PSK`.

5. ใส่ `PSK Pass Phrase:` (รหัสผ่าน) ที่ต้องการ เช่น `cisco123`

***

### 4.2 เครือข่ายที่ 2 (Network B)

#### **Router (R2)**

Plaintext

```
Router>enable
Router#configure terminal
Router(config)#hostname R2

! === ตั้งค่า Interface WAN ที่เชื่อมต่อไปยัง R1 ===
R2(config)#interface GigabitEthernet0/0
R2(config-if)#ip address 10.0.0.2 255.255.255.252
R2(config-if)#description Link to R1
R2(config-if)#no shutdown
R2(config-if)#exit

! === ตั้งค่า 'Router on a Stick' สำหรับ VLANs ===
R2(config)#interface GigabitEthernet0/1
R2(config-if)#no shutdown
R2(config-if)#exit

! --- Sub-interface สำหรับ VLAN 10 (PC) ---
R2(config)#interface GigabitEthernet0/1.10
R2(config-subif)#description Gateway for VLAN_PC
R2(config-subif)#encapsulation dot1Q 10
R2(config-subif)#ip address 192.168.30.1 255.255.255.0
R2(config-subif)#exit

! --- Sub-interface สำหรับ VLAN 20 (Laptop) ---
R2(config)#interface GigabitEthernet0/1.20
R2(config-subif)#description Gateway for VLAN_LAPTOP
R2(config-subif)#encapsulation dot1Q 20
R2(config-subif)#ip address 192.168.40.1 255.255.255.0
R2(config-subif)#exit

! === ตั้งค่า DHCP Server ===
! --- DHCP Pool สำหรับ VLAN 10 (PC) ---
R2(config)#ip dhcp pool VLAN10_PC
R2(dhcp-config)#network 192.168.30.0 255.255.255.0
R2(dhcp-config)#default-router 192.168.30.1
R2(dhcp-config)#exit

! --- DHCP Pool สำหรับ VLAN 20 (Laptop) ---
R2(config)#ip dhcp pool VLAN20_LAPTOP
R2(dhcp-config)#network 192.168.40.0 255.255.255.0
R2(dhcp-config)#default-router 192.168.40.1
R2(dhcp-config)#exit

! === ตั้งค่า Static Route เพื่อให้รู้จักเครือข่ายของ R1 ===
R2(config)#ip route 192.168.10.0 255.255.255.0 10.0.0.1
R2(config)#ip route 192.168.20.0 255.255.255.0 10.0.0.1

R2(config)#exit
R2#write
```

#### **Switch (SW2)**

```
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW2

! === สร้าง VLANs ===
SW2(config)#vlan 10
SW2(config-vlan)#name VLAN_PC
SW2(config-vlan)#exit

SW2(config)#vlan 20
SW2(config-vlan)#name VLAN_LAPTOP
SW2(config-vlan)#exit

! === ตั้งค่าพอร์ต Trunk ที่เชื่อมต่อไปยัง R2 ===
SW2(config)#interface GigabitEthernet0/1
SW2(config-if)#description Trunk to R2
SW2(config-if)#switchport mode trunk
SW2(config-if)#exit

! === ตั้งค่าพอร์ต Access สำหรับ PC2 ===
SW2(config)#interface FastEthernet0/1
SW2(config-if)#description To PC2
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 10
SW2(config-if)#exit

! === ตั้งค่าพอร์ต Access สำหรับ AP2 ===
SW2(config)#interface FastEthernet0/2
SW2(config-if)#description To AP2
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 20
SW2(config-if)#exit

SW2(config)#exit
SW2#write
```

#### **Access Point (AP2)**

1. คลิกที่ `AP2` > แท็บ `Config`.

2. ที่เมนู `Port 1`.

3. ตั้งค่า `SSID:` เป็น `WiFi_B`

4. เลือก `Authentication` เป็น `WPA2-PSK`.

5. ใส่ `PSK Pass Phrase:` (รหัสผ่าน) ที่ต้องการ เช่น `cisco456`

## 5. การตั้งค่าเครื่องลูกข่าย (Client Devices)

#### **PC1 และ PC2**

1. คลิกที่เครื่อง PC.

2. ไปที่แท็บ `Desktop` > `IP Configuration`.

3. เลือก `DHCP`.

4. รอสักครู่ เครื่อง PC จะได้รับ IP Address จาก DHCP Server ใน VLAN ของตนเอง (PC1 จะได้ IP ในกลุ่ม `192.168.10.x`, PC2 จะได้ IP ในกลุ่ม `192.168.30.x`).

#### **Laptop1 และ Laptop2**

1. **เปลี่ยนการ์ดแลน**

   * คลิกที่ Laptop > แท็บ `Physical`.

   * ปิดสวิตช์ Power ของ Laptop (ปุ่มกลมเล็กๆ).

   * ลากการ์ดแลนแบบมีสาย (ที่เป็นช่อง RJ-45) ออกจาก Slot.

   * ลากการ์ด Wireless (`WPC300N`) มาใส่ใน Slot แทน.

   * เปิดสวิตช์ Power ของ Laptop.

2. **เชื่อมต่อ Wi-Fi**

   * ไปที่แท็บ `Desktop` > `PC Wireless`.

   * คลิกที่แท็บ `Connect`.

   * รอสักครู่ จะปรากฏชื่อ SSID ที่เราตั้งไว้ (`WiFi_A` สำหรับ Laptop1, `WiFi_B` สำหรับ Laptop2).

   * คลิกเลือก SSID ที่ต้องการ แล้วกด `Connect`.

   * ใส่รหัสผ่าน (PSK Pass Phrase) ที่ตั้งไว้ แล้วกด `Connect`.

3. **ตรวจสอบ IP Address**

   * ไปที่แท็บ `Desktop` > `IP Configuration`.

   * ตรวจสอบว่าถูกตั้งค่าเป็น `DHCP` และได้รับ IP Address ที่ถูกต้อง (Laptop1 จะได้ IP ในกลุ่ม `192.168.20.x`, Laptop2 จะได้ IP ในกลุ่ม `192.168.40.x`).

## 6. การทดสอบการเชื่อมต่อ

เปิด `Command Prompt` บนเครื่อง PC หรือ Laptop แล้วใช้คำสั่ง `ping` เพื่อทดสอบ

* **ทดสอบ Ping Gateway ของตัวเอง (เช่น จาก PC1):** `ping 192.168.10.1`

* **ทดสอบ Ping ข้าม VLAN ในเครือข่ายเดียวกัน (เช่น จาก PC1 ไป Laptop1):** `ping 192.168.20.x` (ใส่ IP ของ Laptop1)

* **ทดสอบ Ping ไปยังอุปกรณ์ในอีกเครือข่ายหนึ่ง (เช่น จาก PC1 ไป PC2):** `ping 192.168.30.x` (ใส่ IP ของ PC2)

* **ทดสอบ Ping ข้าม VLAN และข้ามเครือข่าย (เช่น จาก PC1 ไป Laptop2):** `ping 192.168.40.x` (ใส่ IP ของ Laptop2)

หากการตั้งค่าทั้งหมดถูกต้อง การ Ping ทั้งหมดควรจะสำเร็จ (อาจมี Request timed out 1-2 ครั้งแรก ขณะที่ ARP กำลังทำงาน)






