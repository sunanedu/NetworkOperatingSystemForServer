# ใบงานการตั้งค่าเครือข่ายใน Cisco Packet Tracer v.8.2.2 (ต่อ)



## 5. DHCP Snooping (2 ใบงาน)

### ใบงาน 5.1: DHCP Snooping พื้นฐาน
**วัตถุประสงค์**: ตั้งค่า DHCP Snooping เพื่อป้องกัน DHCP Server ปลอม  
**อุปกรณ์ที่ใช้**:
- Router: Cisco 2911 (1 ตัว, R1)
- Switch: Cisco 2960-24TT (1 ตัว, SW1)
- PC: 3 เครื่อง (PC1, PC2, PC3)

**การเชื่อมต่อสาย**:
- PC1 --(Straight-through)--> SW1 (Fa0/1)
- PC2 --(Straight-through)--> SW1 (Fa0/2)
- PC3 --(Straight-through)--> SW1 (Fa0/3)
- SW1 (Gi0/1) --(Cross-over)--> R1 (Gi0/0)

**การกำหนด IP Address, Subnet, Gateway**:
- PC1: DHCP (คาดว่าได้ 192.168.10.2/24, Gateway: 192.168.10.1)
- PC2: DHCP (คาดว่าได้ 192.168.10.3/24, Gateway: 192.168.10.1)
- PC3: จะต้องเปลี่ยนเป็น server ในอนาคตเป็น DHCP Server ปลอม (192.168.10.100/24)
- R1: Gi0/0 = 192.168.10.1/24

**ภาพการเชื่อมต่อ (Text)**:
```
[PC1] -- [SW1 (Fa0/1)]
[PC2] -- [SW1 (Fa0/2)]
[PC3] -- [SW1 (Fa0/3)]
          [SW1 (Gi0/1)] -- [R1 (Gi0/0)]
```
![5.1](5.1.png)

**การกำหนดชื่อและการตั้งค่า**:
- **R1**:
```ruby
Would you like to enter the initial configuration dialog? [yes/no]: no
Router> enable
Router# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)# hostname R1

____ความหมายคำสั่ง Configure the gateway interface
R1(config)# interface gigabitethernet0/0
R1(config-if)# ip address 192.168.10.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

____ความหมายคำสั่ง Configure the DHCP pool for clients
R1(config)# ip dhcp pool LAN
R1(dhcp-config)# network 192.168.10.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.10.1
R1(dhcp-config)# exit

R1(config)# exit

R1# copy running-config startup-config
Destination filename [startup-config]?    <-- กด Enter
Building configuration...
[OK]
R1# 
R1# show ip dhcp binding     <-- รันตอนตั้งค่าเสร็จทั้งหมด
```
- **SW1**:
```ruby
Switch> enable
Switch# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)# hostname SW1

____ความหมายคำสั่ง Enable DHCP Snooping globally and for VLAN 1
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 1

____ความหมายคำสั่ง Configure ports for client PCs (these are Untrusted by default)
SW1(config)# interface range fastethernet0/1 - 3
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 1
SW1(config-if-range)# exit

____ความหมายคำสั่ง Configure the port connected to the legitimate DHCP Server (R1)
SW1(config)# interface gigabitethernet0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 1

____ความหมายคำสั่ง Set this port as trusted to allow DHCP server messages
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# exit

____ปิดฟีเจอร์ Option 82 คือการสั่งให้ SW1 หยุดการเพิ่มข้อมูล Option 82 เข้าไปในแพ็กเกจ DHCP
SW1(config)# no ip dhcp snooping information option
SW1(config)# exit

SW1# copy running-config startup-config
Destination filename [startup-config]?    <-- กด Enter
Building configuration...
[OK]
SW1#
SW1# show ip dhcp snooping     <-- รันตอนตั้งค่าเสร็จทั้งหมด
```

**การทดสอบการทำงาน**:
- ใช้คำสั่ง `show ip dhcp snooping` บน SW1 เพื่อยืนยันว่า Gi0/1 เป็น Trusted Port
- ใช้คำสั่ง `show ip dhcp binding` บน R1 เพื่อยืนยันว่า PC1 และ PC2 ได้รับ IP
- ตั้งค่า PC3 เป็น DHCP Server ปลอมและตรวจสอบว่า PC1 และ PC2 ยังได้รับ IP จาก R1
- จาก PC1 ping PC2 (ควรสำเร็จ)

![ping pc1 to pc2/3](5.1-2.png)

### การตั้งค่า PC3 เป็น DHCP Server ปลอม

เราสามารถทำให้ PC3 ทำหน้าที่เป็น DHCP Server ปลอมได้ง่ายๆ ด้วยขั้นตอนต่อไปนี้:
**ลบอุปกรณ์เดิม:** ให้คุณลบอุปกรณ์ PC3 ที่เป็นไอคอนรูป PC ทิ้งไปก่อน (คลิกที่ PC3 แล้วกดปุ่ม Delete บนคีย์บอร์ด)
**เลือกอุปกรณ์ใหม่:**
* ไปที่มุมซ้ายล่างของโปรแกรม Packet Tracer ตรงแถบเลือกอุปกรณ์
* คลิกที่ **End Devices** (ไอคอนรูปคอมพิวเตอร์ Server)


1. **เปิดหน้าต่างตั้งค่า:** ดับเบิลคลิกที่ไอคอนของ **Server**

2. **ไปที่แท็บ Services:** ที่หน้าต่างของ Server ให้คลิกไปที่แท็บ **Services**

3. **เลือกเมนู DHCP:** ในเมนูย่อยทางด้านซ้าย ให้คลิกที่ **DHCP**

4. **ตั้งค่า DHCP Pool ปลอม:** คุณจะเห็นหน้าสำหรับตั้งค่า DHCP Server ให้กรอกข้อมูลดังนี้

   * **Service:** เลือกเป็น **On** เพื่อเปิดการทำงาน

   * **Pool Name:** ตั้งชื่ออะไรก็ได้ เช่น `RoguePool`

   * **Default Gateway:** **(จุดสำคัญของการโจมตี)** ใส่ IP ปลอม เช่น `192.168.10.100` (IP ของตัว PC3 เอง)

   * **DNS Server:** ใส่ IP ปลอม เช่น `8.8.4.4` หรือ `192.168.10.100`

   * **Start IP Address:** กำหนดช่วง IP ที่จะแจก เช่น `192.168.10.150`

   * **Subnet Mask:** `255.255.255.0`

   * **Maximum number of Users:** กำหนดจำนวนเครื่องที่จะแจก เช่น `50`

5. **บันทึกการตั้งค่า:** กดปุ่ม **Add** เพื่อบันทึก Pool ที่สร้างขึ้น (ใน Packet Tracer บางเวอร์ชันอาจเป็นปุ่ม **Save**)

![dhcp](5.1-3.png)

เพียงเท่านี้ **ก็จะเริ่มทำงานเป็น DHCP Server ปลอมทันที** มันจะคอยดักฟัง DHCP Request จากเครื่องอื่นและพยายามแจก IP ตามค่าที่เราตั้งไว้ เพื่อแข่งกับ R1 ที่เป็น Server ตัวจริงครับ

ซึ่งนี่คือสถานการณ์จำลองที่เราต้องการ เพื่อทดสอบว่า DHCP Snooping ที่เราตั้งค่าไว้บน SW1 สามารถบล็อกการแจก IP จากพอร์ต Fa0/3 ที่เป็น Untrusted Port ได้สำเร็จหรือไม่

**กำหนดข้อยกเว้น (Trusted Port):** จากนั้น คำสั่ง `ip dhcp snooping trust` ที่เราใช้กับพอร์ต `gigabitethernet0/1` ก็เปรียบเสมือนการที่เราเดินไปบอกสวิตช์ว่า
 "เดี๋ยวก่อน... มีพอร์ตนี้พอร์ตเดียวนะที่เชื่อใจได้ เพราะต่ออยู่กับ Server ตัวจริง ให้ทำเครื่องหมายว่าเป็นคนพิเศษ **(Trusted)**"

---

### ใบงาน 5.2: DHCP Snooping กับ VLAN
**วัตถุประสงค์**: ตั้งค่า DHCP Snooping สำหรับหลาย VLAN  
**อุปกรณ์ที่ใช้**:
- Router: Cisco 2911 (1 ตัว, R1)
- Switch: Cisco 2960-24TT (2 ตัว, SW1, SW2)
- PC: 2 เครื่อง (PC1, PC2)
- Server-PT: 1 เครื่อง (DHCP Server RoguePool)

**การเชื่อมต่อสาย**:
- PC1 --(Straight-through)--> SW1 (Fa0/1)
- PC2 --(Straight-through)--> SW2 (Fa0/1)
- Server-PT --(Straight-through)--> SW2 (Fa0/2)
- SW1 (Fa0/24) --(Cross-over)--> SW2 (Fa0/24)
- SW1 (Gi0/1) --(Cross-over)--> R1 (Gi0/0)

**การกำหนด IP Address, Subnet, Gateway**:
- PC1: DHCP (คาดว่าได้ 192.168.10.2/24, Gateway: 192.168.10.1, VLAN 10)
- PC2: DHCP (คาดว่าได้ 192.168.20.2/24, Gateway: 192.168.20.1, VLAN 20)
- Server-PT: DHCP Server ปลอม (192.168.20.100/24, VLAN 20)
- R1: Gi0/0.10 = 192.168.10.1/24, Gi0/0.20 = 192.168.20.1/24

**ภาพการเชื่อมต่อ (Text)**:
```
[PC1 (VLAN 10)] -- [SW1 (Fa0/1)]
                    [SW1 (Fa0/24)] -- [SW2 (Fa0/24)]
[PC2 (VLAN 20)] -- [SW2 (Fa0/1)]
[Server-PT (VLAN 20)] -- [SW2 (Fa0/2)]
                    [SW1 (Gi0/1)] -- [R1 (Gi0/0)]
```

![dhcp](5.2.png)

**การกำหนดชื่อและการตั้งค่า**:
- **R1**:
```ruby
Would you like to enter the initial configuration dialog? [yes/no]: no
Router> enable
Router# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)# hostname R1

____ความหมายคำสั่ง Configure sub-interfaces for VLANs
R1(config)# interface gigabitethernet0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# no shutdown
R1(config-subif)# exit

R1(config)# interface gigabitethernet0/0.20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config-subif)# no shutdown
R1(config-subif)# exit

____ความหมายคำสั่ง Turn on the physical interface
R1(config)# interface gigabitethernet0/0
R1(config-if)# no shutdown
R1(config-if)# exit

____ความหมายคำสั่ง Configure DHCP pools
R1(config)# ip dhcp pool VLAN10
R1(dhcp-config)# network 192.168.10.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.10.1
R1(dhcp-config)# exit

R1(config)# ip dhcp pool VLAN20
R1(dhcp-config)# network 192.168.20.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.20.1
R1(dhcp-config)# exit

R1(config)# exit

R1# copy running-config startup-config
Destination filename [startup-config]?   <-- กด Enter
Building configuration...
[OK]
R1#
R1# show ip dhcp binding     <-- รันตอนตั้งค่าเสร็จทั้งหมด

```

- **SW1**:
```ruby
Switch> enable
Switch# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)# hostname SW1

____ความหมายคำสั่ง Create VLANs
SW1(config)# vlan 10
SW1(config-vlan)# name SALES
SW1(config-vlan)# exit
SW1(config)# vlan 20
SW1(config-vlan)# name ENGINEERING
SW1(config-vlan)# exit

____ความหมายคำสั่ง Enable DHCP Snooping globally and for specific VLANs
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 10,20

____ความหมายคำสั่ง Configure access port for PC1 (Untrusted by default)
SW1(config)# interface fastethernet0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# exit

____ความหมายคำสั่ง Configure trunk port to R1 (DHCP Server)
SW1(config)# interface gigabitethernet0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,20

____ความหมายคำสั่ง Set this port as trusted because it leads to the legitimate DHCP server
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# exit

____ความหมายคำสั่ง Configure trunk port to SW2
SW1(config)# interface fastethernet0/24
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,20

____ความหมายคำสั่ง This trunk must also be trusted to pass DHCP replies to SW2
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# exit

____ปิดฟีเจอร์ Option 82 คือการสั่งให้ SW1 หยุดการเพิ่มข้อมูล Option 82 เข้าไปในแพ็กเกจ DHCP
SW1(config)# no ip dhcp snooping information option
 
SW1(config)# exit

SW1# copy running-config startup-config
Destination filename [startup-config]?   <-- กด Enter 
Building configuration...
[OK]
SW1#
SW1# show ip dhcp snooping     <-- รันตอนตั้งค่าเสร็จทั้งหมด
```

- **SW2**:
```ruby
Switch> enable
Switch# configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)# hostname SW2

____ความหมายคำสั่ง Create VLANs
SW2(config)# vlan 10
SW2(config-vlan)# name SALES
SW2(config-vlan)# exit
SW2(config)# vlan 20
SW2(config-vlan)# name ENGINEERING
SW2(config-vlan)# exit

____ความหมายคำสั่ง Enable DHCP Snooping globally and for specific VLANs
SW2(config)# ip dhcp snooping
SW2(config)# ip dhcp snooping vlan 10,20

____ความหมายคำสั่ง Configure access port for PC2 (Untrusted by default)
SW2(config)# interface fastethernet0/1
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 20
SW2(config-if)# exit

____ความหมายคำสั่ง Configure access port for Rogue DHCP Server (Untrusted by default)
SW2(config)# interface fastethernet0/2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 20
SW2(config-if)# exit

____ความหมายคำสั่ง Configure trunk port to SW1
SW2(config)# interface fastethernet0/24
SW2(config-if)# switchport mode trunk
SW2(config-if)# switchport trunk allowed vlan 10,20
ความหมายคำสั่ง This trunk must be trusted to allow DHCP replies from SW1 to reach clients
SW2(config-if)# ip dhcp snooping trust
SW2(config-if)# exit

____ปิดฟีเจอร์ Option 82 คือการสั่งให้ SW1 หยุดการเพิ่มข้อมูล Option 82 เข้าไปในแพ็กเกจ DHCP
SW1(config)# no ip dhcp snooping information option
    
SW2(config)# exit

SW2# copy running-config startup-config
Destination filename [startup-config]?   <-- กด Enter 
Building configuration...
[OK]
SW2#
SW2# show ip dhcp snooping     <-- รันตอนตั้งค่าเสร็จทั้งหมด
```


**การทดสอบการทำงาน**:
- ใช้คำสั่ง `show ip dhcp snooping` บน SW1 และ SW2 เพื่อยืนยันว่า Gi0/1 บน SW1 เป็น Trusted Port
- ใช้คำสั่ง `show ip dhcp binding` บน R1 เพื่อยืนยันว่า PC1 และ PC2 ได้รับ IP
- ตั้งค่า PC3 เป็น DHCP Server ปลอมและตรวจสอบว่า PC2 ยังได้รับ IP จาก R1
- จาก PC1 ping PC2 (ควรสำเร็จ)

