# การออกแบบการเรียนรู้โดยใช้ Cisco Packet Tracer v.8.2.2: การจัดการระบบเครือข่าย

## วัตถุประสงค์
1. เพื่อให้ผู้เรียนเข้าใจและสามารถกำหนดค่า VLAN, Inter-VLAN, IP Routing, DHCP, DHCP Snooping, Loop Guard, Spanning Tree, Link Aggregation, NAT, Access Control List (ACL), Load Balancing, Firewall, VPN และ QoS
2. เพื่อฝึกปฏิบัติการตั้งค่าระบบเครือข่ายใน Cisco Packet Tracer v.8.2.2
3. เพื่อพัฒนาทักษะการจัดการและเพิ่มประสิทธิภาพเครือข่าย

---

### หน่วยที่ 1: พื้นฐานการจัดการระบบเครือข่ายและ VLAN
**หัวข้อ:**
- ความรู้พื้นฐานเกี่ยวกับเครือข่ายและอุปกรณ์ Cisco
- การกำหนดค่า VLAN และ Trunking
- การทดสอบการเชื่อมต่อภายใน VLAN

**ตัวอย่างแล็บ: การตั้งค่า VLAN และทดสอบการเชื่อมต่อ**

#### อุปกรณ์ที่ใช้
- Switch: Cisco 2950 (2 ตัว)
- PC: 4 เครื่อง
- สาย: สาย Straight-through และ Cross-over

#### ถาพตัวอย่าง

![ตัวอย่าง VLAN](https://raw.githubusercontent.com/sunanedu/NetworkOperatingSystemForServer/main/unit2/vlan1-1.png)

#### การเชื่อมต่อสาย
```
PC1 --(Straight-through)--> Switch1 (Fa0/1)
PC2 --(Straight-through)--> Switch1 (Fa0/2)
PC3 --(Straight-through)--> Switch2 (Fa0/1)
PC4 --(Straight-through)--> Switch2 (Fa0/2)
Switch1 (Fa0/24) --(Cross-over)--> Switch2 (Fa0/24)
```

#### การกำหนด IP Address, Subnet, Gateway
- PC1: IP = 192.168.10.2, Subnet = 255.255.255.0, Gateway = 192.168.10.1 (VLAN 10)
- PC2: IP = 192.168.20.2, Subnet = 255.255.255.0, Gateway = 192.168.20.1 (VLAN 20)
- PC3: IP = 192.168.10.3, Subnet = 255.255.255.0, Gateway = 192.168.10.1 (VLAN 10)
- PC4: IP = 192.168.20.3, Subnet = 255.255.255.0, Gateway = 192.168.20.1 (VLAN 20)

#### การกำหนดชื่อและการตั้งค่า
**Switch1**
```plaintext
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW1
SW1(config)#vlan 10
SW1(config-vlan)#name SALES
SW1(config-vlan)#exit
SW1(config)#vlan 20
SW1(config-vlan)#name IT
SW1(config-vlan)#exit
SW1(config)#interface fa0/1
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 10
SW1(config-if)#exit
SW1(config)#interface fa0/2
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 20
SW1(config-if)#exit
SW1(config)#interface fa0/24
SW1(config-if)#switchport mode trunk
SW1(config-if)#exit
```
**Switch2**
```plaintext
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW2
SW2(config)#vlan 10
SW2(config-vlan)#name SALES
SW2(config-vlan)#exit
SW2(config)#vlan 20
SW2(config-vlan)#name IT
SW2(config-vlan)#exit
SW2(config)#interface fa0/1
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 10
SW2(config-if)#exit
SW2(config)#interface fa0/2
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 20
SW2(config-if)#exit
SW2(config)#interface fa0/24
SW2(config-if)#switchport mode trunk
SW2(config-if)#exit
```

#### การทดสอบการทำงาน
- ใช้คำสั่ง `ping` จาก PC1 ไป PC3 (VLAN 10) ควรสำเร็จ
- ใช้คำสั่ง `ping` จาก PC1 ไป PC2 (VLAN 20) ควรล้มเหลว
- ตรวจสอบการทำงานของ VLAN ด้วยคำสั่ง `show vlan brief` บนสวิตช์

#### ภาพการเชื่อมต่อ (Text)
```
[PC1 (VLAN 10)] -- [SW1 (Fa0/1)] -- [Fa0/24 (Trunk)] -- [SW2 (Fa0/24)] -- [PC3 (VLAN 10)]
[PC2 (VLAN 20)] -- [SW1 (Fa0/2)]                       [SW2 (Fa0/1)] -- [PC4 (VLAN 20)]
```
