# ใบงานการตั้งค่าเครือข่ายใน Cisco Packet Tracer v.8.2.2 (ต่อ)

เอกสารนี้ต่อจากส่วนที่เหลือของใบงานปฏิบัติการเครือข่ายสำหรับหัวข้อ **NAT** (Dynamic NAT) และ **Access Control List (ACL)** โดยใช้ Cisco Packet Tracer v.8.2.2 ระดับความยากปานกลางถึงยาก

---

## 9. NAT (ต่อจากใบงาน 9.1)

### ใบงาน 9.2: Dynamic NAT
**วัตถุประสงค์**: ตั้งค่า Dynamic NAT เพื่อให้ PC ในเครือข่ายภายในเข้าถึงเครือข่ายภายนอกโดยใช้ Pool ของ Public IP  
**อุปกรณ์ที่ใช้**:
- Router: Cisco 2911 (2 ตัว, R1, R2)
- Switch: Cisco 2960-24TT (1 ตัว, SW1)
- PC: 2 เครื่อง (PC1, PC2)
- Server: 1 เครื่อง (Server1)

**การเชื่อมต่อสาย**:
- PC1 --(Straight-through)--> SW1 (Fa0/1)
- PC2 --(Straight-through)--> SW1 (Fa0/2)
- SW1 (Fa0/24) --(Cross-over)--> R1 (Gi0/0)
- R1 (Gi0/1) --(Cross-over)--> R2 (Gi0/0)
- Server1 --(Straight-through)--> R2 (Gi0/1)

**การกำหนด IP Address, Subnet, Gateway**:
- PC1: 192.168.10.2/24, Gateway: 192.168.10.1
- PC2: 192.168.10.3/24, Gateway: 192.168.10.1
- Server1: 203.0.113.2/24, Gateway: 203.0.113.1
- R1: Gi0/0 = 192.168.10.1/24, Gi0/1 = 172.16.1.1/30
- R2: Gi0/0 = 172.16.1.2/30, Gi0/1 = 203.0.113.1/24
- NAT Pool: 172.16.1.10 - 172.16.1.20

**การกำหนดชื่อและการตั้งค่า**:
- **R1**:
```plaintext
Router>enable
Router#configure terminal
Router(config)#hostname R1
R1(config)#interface gi0/0
R1(config-if)#ip address 192.168.10.1 255.255.255.0
R1(config-if)#ip nat inside
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#interface gi0/1
R1(config-if)#ip address 172.16.1.1 255.255.255.252
R1(config-if)#ip nat outside
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#ip nat pool NAT_POOL 172.16.1.10 172.16.1.20 netmask 255.255.255.252
R1(config)#access-list 1 permit 192.168.10.0 0.0.0.255
R1(config)#ip nat inside source list 1 pool NAT_POOL
R1(config)#ip route 203.0.113.0 255.255.255.0 172.16.1.2
R1(config)#write memory
```
- **R2**:
```plaintext
Router>enable
Router#configure terminal
Router(config)#hostname R2
R2(config)#interface gi0/0
R2(config-if)#ip address 172.16.1.2 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#interface gi0/1
R2(config-if)#ip address 203.0.113.1 255.255.255.0
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#ip route 172.16.1.0 255.255.255.0 172.16.1.1
R2(config)#write memory
```
- **SW1**:
```plaintext
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW1
SW1(config)#interface fa0/1
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 1
SW1(config-if)#exit
SW1(config)#interface fa0/2
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 1
SW1(config-if)#exit
SW1(config)#interface fa0/24
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 1
SW1(config-if)#exit
SW1(config)#write memory
```

**การทดสอบการทำงาน**:
- ใช้คำสั่ง `show ip nat translations` บน R1 เพื่อยืนยันว่า PC1 และ PC2 ใช้ IP จาก NAT Pool (172.16.1.10 - 172.16.1.20)
- จาก PC1 ping Server1 (203.0.113.2, ควรสำเร็จ)
- จาก PC2 ping Server1 (203.0.113.2, ควรสำเร็จ)
- ใช้คำสั่ง `show ip nat statistics` เพื่อยืนยันจำนวนการแปล NAT

**ภาพการเชื่อมต่อ (Text)**:
```
[PC1] -- [SW1 (Fa0/1)]
[PC2] -- [SW1 (Fa0/2)]
          [SW1 (Fa0/24)] -- [R1 (Gi0/0)]
                              [R1 (Gi0/1)] -- [R2 (Gi0/0)]
[Server1] -- [R2 (Gi0/1)]
```

---

## 10. Access Control List (2 ใบงาน)

### ใบงาน 10.1: Standard ACL
**วัตถุประสงค์**: ตั้งค่า Standard ACL เพื่อจำกัดการเข้าถึงจากเครือข่ายหนึ่งไปยังอีกเครือข่ายหนึ่ง  
**อุปกรณ์ที่ใช้**:
- Router: Cisco 2911 (2 ตัว, R1, R2)
- Switch: Cisco 2960-24TT (2 ตัว, SW1, SW2)
- PC: 2 เครื่อง (PC1, PC2)

**การเชื่อมต่อสาย**:
- PC1 --(Straight-through)--> SW1 (Fa0/1)
- PC2 --(Straight-through)--> SW2 (Fa0/1)
- SW1 (Fa0/24) --(Cross-over)--> R1 (Gi0/0)
- SW2 (Fa0/24) --(Cross-over)--> R2 (Gi0/0)
- R1 (Gi0/1) --(Cross-over)--> R2 (Gi0/1)

**การกำหนด IP Address, Subnet, Gateway**:
- PC1: 192.168.10.2/24, Gateway: 192.168.10.1
- PC2: 192.168.20.2/24, Gateway: 192.168.20.1
- R1: Gi0/0 = 192.168.10.1/24, Gi0/1 = 172.16.1.1/30
- R2: Gi0/0 = 192.168.20.1/24, Gi0/1 = 172.16.1.2/30

**การกำหนดชื่อและการตั้งค่า**:
- **R1**:
```plaintext
Router>enable
Router#configure terminal
Router(config)#hostname R1
R1(config)#interface gi0/0
R1(config-if)#ip address 192.168.10.1 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#interface gi0/1
R1(config-if)#ip address 172.16.1.1 255.255.255.252
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#ip route 192.168.20.0 255.255.255.0 172.16.1.2
R1(config)#write memory
```
- **R2**:
```plaintext
Router>enable
Router#configure terminal
Router(config)#hostname R2
R2(config)#interface gi0/0
R2(config-if)#ip address 192.168.20.1 255.255.255.0
R2(config-if)#ip access-group 1 in
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#interface gi0/1
R2(config-if)#ip address 172.16.1.2 255.255.255.252
R2(config-if)#no shutdown
R2(config-if)#exit
R2(config)#access-list 1 deny 192.168.10.2 0.0.0.0
R2(config)#access-list 1 permit any
R2(config)#ip route 192.168.10.0 255.255.255.0 172.16.1.1
R2(config)#write memory
```
- **SW1**:
```plaintext
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW1
SW1(config)#interface fa0/1
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 1
SW1(config-if)#exit
SW1(config)#interface fa0/24
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 1
SW1(config-if)#exit
SW1(config)#write memory
```
- **SW2**:
```plaintext
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW2
SW2(config)#interface fa0/1
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 1
SW2(config-if)#exit
SW2(config)#interface fa0/24
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 1
SW2(config-if)#exit
SW2(config)#write memory
```

**การทดสอบการทำงาน**:
- ใช้คำสั่ง `show access-lists` บน R2 เพื่อยืนยัน ACL
- จาก PC1 ping PC2 (192.168.20.2, ควรล้มเหลว เนื่องจากถูก ACL บล็อก)
- จาก PC2 ping PC1 (192.168.10.2, ควรสำเร็จ)
- เพิ่ม PC3 (192.168.10.3/24) ใน SW1 และ ping PC2 (ควรสำเร็จ เพราะ ACL บล็อกเฉพาะ PC1)

**ภาพการเชื่อมต่อ (Text)**:
```
[PC1] -- [SW1 (Fa0/1)] -- [Fa0/24] -- [R1 (Gi0/0)]
                                        [R1 (Gi0/1)] -- [R2 (Gi0/1)]
[PC2] -- [SW2 (Fa0/1)] -- [Fa0/24] -- [R2 (Gi0/0)]
```

---

### ใบงาน 10.2: Extended ACL
**วัตถุประสงค์**: ตั้งค่า Extended ACL เพื่อจำกัดการเข้าถึงเฉพาะโปรโตคอล (เช่น HTTP) จาก VLAN หนึ่งไปยังอีก VLAN  
**อุปกรณ์ที่ใช้**:
- Router: Cisco 2911 (1 ตัว, R1)
- Switch: Cisco 2960-24TT (1 ตัว, SW1)
- PC: 2 เครื่อง (PC1, PC2)
- Server: 1 เครื่อง (Server1)

**การเชื่อมต่อสาย**:
- PC1 --(Straight-through)--> SW1 (Fa0/1)
- PC2 --(Straight-through)--> SW1 (Fa0/2)
- Server1 --(Straight-through)--> SW1 (Fa0/3)
- SW1 (Fa0/24) --(Cross-over)--> R1 (Gi0/0)

**การกำหนด IP Address, Subnet, Gateway**:
- PC1: 192.168.10.2/24, Gateway: 192.168.10.1 (VLAN 10)
- PC2: 192.168.10.3/24, Gateway: 192.168.10.1 (VLAN 10)
- Server1: 192.168.20.2/24, Gateway: 192.168.20.1 (VLAN 20)
- R1: Gi0/0.10 = 192.168.10.1/24, Gi0/0.20 = 192.168.20.1/24

**การกำหนดชื่อและการตั้งค่า**:
- **R1**:
```plaintext
Router>enable
Router#configure terminal
Router(config)#hostname R1
R1(config)#interface gi0/0.10
R1(config-subif)#encapsulation dot1Q 10
R1(config-subif)#ip address 192.168.10.1 255.255.255.0
R1(config-subif)#ip access-group 100 in
R1(config-subif)#no shutdown
R1(config-subif)#exit
R1(config)#interface gi0/0.20
R1(config-subif)#encapsulation dot1Q 20
R1(config-subif)#ip address 192.168.20.1 255.255.255.0
R1(config-subif)#no shutdown
R1(config-subif)#exit
R1(config)#interface gi0/0
R1(config-if)#no shutdown
R1(config-if)#exit
R1(config)#access-list 100 permit tcp 192.168.10.0 0.0.0.255 192.168.20.2 0.0.0.0 eq 80
R1(config)#access-list 100 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
R1(config)#access-list 100 permit ip any any
R1(config)#write memory
```
- **SW1**:
```plaintext
Switch>enable
Switch#configure terminal
Switch(config)#hostname SW1
SW1(config)#vlan 10
SW1(config-vlan)#name SALES
SW1(config-vlan)#exit
SW1(config)#vlan 20
SW1(config-vlan)#name ENGINEERING
SW1(config-vlan)#exit
SW1(config)#interface fa0/1
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 10
SW1(config-if)#exit
SW1(config)#interface fa0/2
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 10
SW1(config-if)#exit
SW1(config)#interface fa0/3
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 20
SW1(config-if)#exit
SW1(config)#interface fa0/24
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport trunk allowed vlan 10,20
SW1(config-if)#switchport trunk native vlan 10
SW1(config-if)#exit
SW1(config)#write memory
```
- **Server1**: ตั้งค่า HTTP Service ผ่าน GUI ใน Packet Tracer

**การทดสอบการทำงาน**:
- ใช้คำสั่ง `show access-lists` บน R1 เพื่อยืนยัน ACL
- จาก PC1 และ PC2 เข้าเว็บเบราว์เซอร์ไปที่ 192.168.20.2 (ควรเข้าถึงหน้าเว็บได้)
- จาก PC1 ping Server1 (192.168.20.2, ควรล้มเหลว เนื่องจากถูก ACL บล็อก)
- จาก PC2 ping Server1 (192.168.20.2, ควรล้มเหลว)
- จาก Server1 ping PC1 (ควรสำเร็จ เพราะ ACL ใช้ทิศ in บน Gi0/0.10)

**ภาพการเชื่อมต่อ (Text)**:
```
[PC1 (VLAN 10)] -- [SW1 (Fa0/1)]
[PC2 (VLAN 10)] -- [SW1 (Fa0/2)]
[Server1 (VLAN 20)] -- [SW1 (Fa0/3)]
                    [SW1 (Fa0/24)] -- [R1 (Gi0/0)]
```

---

## คำแนะนำ
- **การแก้ปัญหา**: หากการทดสอบล้มเหลว (เช่น ping ไม่สำเร็จ):
  - ตรวจสอบสถานะพอร์ตด้วย `show interfaces status` หรือ `show ip interface brief`
  - ตรวจสอบสายเชื่อมต่อใน Packet Tracer (Straight-through สำหรับ PC-Switch, Cross-over สำหรับ Switch-Router หรือ Switch-Switch)
  - รีเซ็ตอุปกรณ์และตั้งค่าใหม่หากจำเป็น
- **การปรับแต่ง**: หากต้องการเพิ่มความซับซ้อน (เช่น เพิ่ม VLAN, ใช้ OSPF แทน Static Route, หรือเพิ่มเงื่อนไขใน ACL) สามารถแจ้งเพื่อปรับใบงาน
- **อุปกรณ์ที่ใช้**: ใช้ Cisco 2911 และ Cisco 2960-24TT สำหรับความเข้ากันได้กับ Packet Tracer v.8.2.2 (Cisco 3560-24PS ใช้ในบางใบงานที่ต้องการ Layer 3 Switching)
- **บันทึก**: ทุกใบงานบันทึกการตั้งค่าด้วย `write memory` เพื่อให้แน่ใจว่าการตั้งค่าถาวร