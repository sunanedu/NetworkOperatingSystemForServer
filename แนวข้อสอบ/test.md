# หน่วยที่ 1: พื้นฐานเครือข่ายและการจัดการ VLAN

ยินดีต้อนรับสู่หน่วยการเรียนรู้แรกครับ! 🚀 ในหน่วยนี้ เราจะมาปูพื้นฐานที่สำคัญที่สุดอย่างหนึ่งในการจัดการเครือข่ายสมัยใหม่ นั่นก็คือ **VLAN** หรือเครือข่ายเสมือน เราจะมาดูกันว่ามันคืออะไร, มีประโยชน์อย่างไร, และจะตั้งค่าบนอุปกรณ์สวิตช์ของ Cisco ได้อย่างไร เพื่อให้พร้อมสำหรับหัวข้อที่ซับซ้อนขึ้นในหน่วยถัดไป

## VLAN คืออะไร และทำไมเราต้องใช้? 

ลองจินตนาการว่าในออฟฟิศของคุณมีสวิตช์ (Switch) อยู่หนึ่งตัว และมีคอมพิวเตอร์จากหลายแผนก เช่น ฝ่ายขาย (Sales), ฝ่ายบัญชี (Accounting), และฝ่ายบุคคล (HR) มาเสียบใช้งานที่สวิตช์ตัวเดียวกันนี้

โดยปกติแล้ว คอมพิวเตอร์ทุกเครื่องที่ต่อกับสวิตช์ตัวเดียวกันจะสามารถ "คุย" หรือส่งข้อมูลหากันได้หมด ซึ่งอาจจะไม่ใช่เรื่องดีเสมอไป โดยเฉพาะในด้านความปลอดภัยและความเป็นระเบียบ

**VLAN (Virtual Local Area Network)** คือเทคโนโลยีที่เข้ามาช่วยแก้ปัญหานี้ โดยเป็นการ **"แบ่ง"** สวิตช์ 1 ตัว ให้ทำงานเหมือนมีสวิตช์เสมือนหลายๆ ตัวอยู่ข้างใน เราสามารถจัดกลุ่มพอร์ตบนสวิตช์เพื่อสร้างเป็นเครือข่ายย่อยๆ แยกออกจากกันอย่างชัดเจน ถึงแม้จะใช้อุปกรณ์ทางกายภาพตัวเดียวกันก็ตาม

!(<https://www.google.com/search?q=VLAN+diagram+simple&tbm=isch>)

### ประโยชน์หลักของ VLAN

1. **เพิ่มความปลอดภัย (Security):**  ทำให้คอมพิวเตอร์ของฝ่ายขาย คุยได้เฉพาะกับฝ่ายขายด้วยกัน ไม่สามารถมองเห็นหรือส่งข้อมูลข้ามไปหาฝ่ายบัญชีได้โดยตรง ช่วยป้องกันข้อมูลสำคัญรั่วไหล

2. **ลดขนาดของ Broadcast Domain:**  เวลาคอมพิวเตอร์ส่งข้อมูลแบบกระจาย (Broadcast) เช่น การค้นหา DHCP Server, ข้อมูลนั้นจะถูกส่งไปแค่ในกลุ่ม VLAN เดียวกันเท่านั้น ไม่วิ่งไปรบกวนคอมพิวเตอร์ใน VLAN อื่น ทำให้เครือข่ายโดยรวมทำงานได้ดีขึ้น ไม่หนาแน่นโดยไม่จำเป็น

3. **จัดการง่ายและยืดหยุ่น (Flexibility & Management):**  ไม่ว่าพนักงานจะย้ายโต๊ะไปนั่งทำงานที่ไหน ตราบใดที่พอร์ตของสวิตช์ถูกตั้งค่าเป็น VLAN เดิม พนักงานคนนั้นก็จะยังคงอยู่ในเครือข่ายเดิมเสมอ ไม่ต้องเดินสายใหม่ให้วุ่นวาย

***

## ส่วนประกอบสำคัญของ VLAN: Access Port vs. Trunk Port

เพื่อให้ VLAN ทำงานได้ เราต้องกำหนด "โหมด (Mode)" การทำงานให้พอร์ตบนสวิตช์ ซึ่งมี 2 โหมดหลักๆ ที่ต้องรู้จัก

### 1. Access Port

* **หน้าที่:** ใช้สำหรับเชื่อมต่อกับอุปกรณ์ปลายทาง (End Devices) เช่น คอมพิวเตอร์, ปริ้นเตอร์, หรือ IP Phone

* **คุณสมบัติ:** 1 Access Port จะเป็นสมาชิกได้ **เพียง 1 VLAN เท่านั้น** (บวกกับอีก 1 Voice VLAN ถ้ามี)

* **ตัวอย่าง:** พอร์ตที่ต่อกับ PC ของฝ่ายขาย จะถูกตั้งค่าเป็น Access Port ของ VLAN 10 (SALES)

### 2. Trunk Port

* **หน้าที่:** ใช้สำหรับเชื่อมต่อ **ระหว่างสวิตช์ด้วยกัน** หรือเชื่อมต่อสวิตช์กับเราเตอร์

* **คุณสมบัติ:** เป็นเหมือน "ท่อส่งข้อมูลหลัก" ที่สามารถ **ส่งข้อมูลของหลายๆ VLAN** วิ่งผ่านไปได้ในสายเส้นเดียว โดยจะมีการใส่ "ป้ายกำกับ" (VLAN Tag) เพื่อระบุว่าข้อมูลแต่ละชิ้นเป็นของ VLAN ไหน

* **ตัวอย่าง:** ถ้าเรามีสวิตช์ 2 ตัว และทั้งสองตัวมี VLAN 10 และ 20 อยู่, เราต้องเชื่อมต่อสวิตช์ทั้งสองด้วย Trunk Port เพื่อให้ PC ใน VLAN 10 ของสวิตช์ตัวแรก คุยกับ PC ใน VLAN 10 ของสวิตช์ตัวที่สองได้

!(<https://www.google.com/search?q=Access+Port+vs+Trunk+Port+diagram&tbm=isch>)

### กรณีพิเศษ: Voice VLAN 

สำหรับอุปกรณ์อย่าง IP Phone ที่ต้องการคุณภาพสัญญาณเสียงที่นิ่งและต่อเนื่อง เรามักจะสร้าง **Voice VLAN** แยกออกมาต่างหาก เพื่อให้ Traffic ของเสียงไม่ถูกรบกวนจาก Traffic ข้อมูลปกติ โดยพอร์ตที่เชื่อมต่อกับ IP Phone จะมีการตั้งค่าที่พิเศษคือ:

* กำหนด **Access VLAN** สำหรับคอมพิวเตอร์ที่มาต่อพ่วงกับ IP Phone

* กำหนด **Voice VLAN** สำหรับตัว IP Phone เอง

***

## Lab 1: การตั้งค่า VLAN, Trunking, และ Management IP

ใน Lab นี้ เราจะจำลองสถานการณ์ที่มีสวิตช์ 2 ตัว และมีคอมพิวเตอร์จาก 2 แผนก (SALES, ENGINEERING) และเราจะทำการตั้งค่าเพื่อให้คอมพิวเตอร์ในแผนกเดียวกันที่อยู่คนละสวิตช์สามารถสื่อสารกันได้ และปิดท้ายด้วยการตั้งค่า Management IP เพื่อให้เราสามารถรีโมทเข้าไปจัดการสวิตช์ได้

### แผนภาพเครือข่าย (Topology)

* **อุปกรณ์:**

  * Switch 2960: 2 ตัว (SW1, SW2)

  * PC: 4 เครื่อง (PC1, PC2, PC3, PC4)

* **VLAN Plan:**

  * **VLAN 10:** SALES

  * **VLAN 20:** ENGINEERING

  * **VLAN 99:** MANAGEMENT (สำหรับจัดการสวิตช์)

* **การเชื่อมต่อ:**

  * PC1 (VLAN 10) -> SW1 (พอร์ต Fa0/1)

  * PC2 (VLAN 20) -> SW1 (พอร์ต Fa0/2)

  * PC3 (VLAN 10) -> SW2 (พอร์ต Fa0/1)

  * PC4 (VLAN 20) -> SW2 (พอร์ต Fa0/2)

  * SW1 (พอร์ต Fa0/24) <--> SW2 (พอร์ต Fa0/24) **(Trunk Link)**

### ขั้นตอนการตั้งค่า (Configuration)

#### **1. ตั้งค่าบน SW1**

```
Switch> enable
Switch# configure terminal

! === สร้าง VLAN และตั้งชื่อ ===
Switch(config)# vlan 10
Switch(config-vlan)# name SALES
Switch(config-vlan)# exit
Switch(config)# vlan 20
Switch(config-vlan)# name ENGINEERING
Switch(config-vlan)# exit
Switch(config)# vlan 99
Switch(config-vlan)# name MANAGEMENT
Switch(config-vlan)# exit

! === กำหนด Access Port ให้กับ PC ===
! พอร์ตสำหรับ PC1 (SALES)
Switch(config)# interface FastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit

! พอร์ตสำหรับ PC2 (ENGINEERING)
Switch(config)# interface FastEthernet0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config-if)# exit

! === กำหนด Trunk Port สำหรับเชื่อมต่อไปยัง SW2 ===
Switch(config)# interface FastEthernet0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# exit

! === (Optional) ตั้งค่า Management IP ===
Switch(config)# interface vlan 99
Switch(config-if)# ip address 192.168.99.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

! === บันทึกการตั้งค่า ===
Switch(config)# end
Switch# copy running-config startup-config
```

#### **2. ตั้งค่าบน SW2** (ทำคล้ายกับ SW1)

```
Switch> enable
Switch# configure terminal

! === สร้าง VLAN และตั้งชื่อ ===
Switch(config)# vlan 10
Switch(config-vlan)# name SALES
Switch(config-vlan)# exit
Switch(config)# vlan 20
Switch(config-vlan)# name ENGINEERING
Switch(config-vlan)# exit

! === กำหนด Access Port ให้กับ PC ===
! พอร์ตสำหรับ PC3 (SALES)
Switch(config)# interface FastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit

! พอร์ตสำหรับ PC4 (ENGINEERING)
Switch(config)# interface FastEthernet0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
Switch(config-if)# exit

! === กำหนด Trunk Port สำหรับเชื่อมต่อไปยัง SW1 ===
Switch(config)# interface FastEthernet0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# exit

! === บันทึกการตั้งค่า ===
Switch(config)# end
Switch# copy running-config startup-config
```

### การตรวจสอบและทดสอบ (Verification & Testing)

1. **ตรวจสอบการสร้าง VLAN:** บน SW1 และ SW2 ใช้คำสั่ง:

   ```
   show vlan brief
   ```

   * **ผลที่คาดหวัง:** คุณจะเห็น VLAN 10, 20, 99 ถูกสร้างขึ้น และพอร์ต Fa0/1, Fa0/2 ถูกกำหนดให้กับ VLAN ที่ถูกต้อง

2. **ตรวจสอบสถานะ Trunk Link:** บน SW1 และ SW2 ใช้คำสั่ง:

   ```
   show interfaces trunk
   ```

   * **ผลที่คาดหวัง:** คุณจะเห็นว่าพอร์ต Fa0/24 อยู่ในโหมด `on` และสถานะ `trunking` และอนุญาตให้ทุก VLAN (หรือ VLAN 10, 20) วิ่งผ่านได้

3. **ทดสอบการเชื่อมต่อ:**

   * **Ping ภายใน VLAN เดียวกัน:**

     * จาก PC1 (VLAN 10) `ping` ไปยัง PC3 (VLAN 10) -> **ควรจะสำเร็จ (Success)**

     * จาก PC2 (VLAN 20) `ping` ไปยัง PC4 (VLAN 20) -> **ควรจะสำเร็จ (Success)**

   * **Ping ข้าม VLAN:**

     * จาก PC1 (VLAN 10) `ping` ไปยัง PC2 (VLAN 20) -> **ควรจะล้มเหลว (Failed)**

     * จาก PC1 (VLAN 10) `ping` ไปยัง PC4 (VLAN 20) -> **ควรจะล้มเหลว (Failed)**

   * **ทดสอบ Management IP:** (ถ้าตั้งค่าไว้)

     * กำหนด IP ให้ PC เครื่องใดก็ได้ ให้อยู่ในวง 192.168.99.0/24 (เช่น 192.168.99.2) แล้วนำไปต่อที่พอร์ตที่เป็นสมาชิกของ VLAN 99

     * `ping` ไปยัง 192.168.99.1 -> **ควรจะสำเร็จ (Success)**



# หน่วยที่ 2: การสื่อสารข้าม VLAN (Inter-VLAN Routing) และ DHCP

ในหน่วยที่ 1 เราได้ทำการ "แบ่ง" เครือข่ายออกเป็น VLAN ย่อยๆ ซึ่งช่วยเพิ่มความปลอดภัยและจัดระเบียบได้เป็นอย่างดี แต่ก็มีข้อจำกัดคือ **อุปกรณ์ที่อยู่คนละ VLAN ไม่สามารถคุยกันได้** ในหน่วยนี้ เราจะมาทลายกำแพงนั้นด้วย **Inter-VLAN Routing** และทำให้การจัดการ IP Address เป็นเรื่องง่ายด้วย **DHCP**

## ทำไมต้องทำ Inter-VLAN Routing? 

ลองนึกภาพตามนะครับ:

* **VLAN 10 (SALES):** มีเครื่องคอมพิวเตอร์ของฝ่ายขาย

* **VLAN 20 (SERVER):** มีเครื่อง File Server ที่เก็บข้อมูลสำคัญ

ถ้าฝ่ายขายต้องการดึงไฟล์งานจาก Server พวกเขาจะทำไม่ได้ เพราะอยู่คนละ VLAN กัน เปรียบเสมือนอยู่คนละหมู่บ้านที่ไม่มีสะพานเชื่อมถึงกัน

**Inter-VLAN Routing** ก็คือการสร้าง "สะพาน" หรือ "เส้นทาง" นี้ขึ้นมานั่นเอง โดยอาศัยอุปกรณ์ Layer 3 (เช่น Router หรือ Layer 3 Switch) มาทำหน้าที่เป็น **"คนกลาง"** ในการส่งข้อมูลข้าม VLAN ให้

### วิธีที่ 1: "Router-on-a-Stick" (เราเตอร์บนไม้เสียบ)

เป็นวิธีที่นิยมมากที่สุดสำหรับการทำ Inter-VLAN Routing ในเครือข่ายขนาดเล็กถึงกลาง ชื่ออาจจะฟังดูแปลก แต่มันคือเทคนิคการใช้ **Interface เดียวบน Router** เพื่อรับ-ส่งข้อมูลให้กับ **ทุกๆ VLAN** โดยอาศัยการเชื่อมต่อแบบ Trunk Link กับสวิตช์

* **หลักการทำงาน:**

  1. เราจะสร้าง **Sub-interfaces** (อินเทอร์เฟซย่อยเสมือน) ขึ้นมาบน Router ให้มีจำนวนเท่ากับ VLAN ที่เราต้องการจะทำ Routing

  2. แต่ละ Sub-interface จะถูกกำหนดให้รับผิดชอบ 1 VLAN และมี IP Address เป็นของตัวเอง

  3. IP Address ของ Sub-interface นี้ จะกลายเป็น **Default Gateway** ให้กับคอมพิวเตอร์ใน VLAN นั้นๆ

  4. เมื่อ PC จาก VLAN 10 ต้องการส่งข้อมูลไปหา Server ใน VLAN 20, PC จะส่งข้อมูลไปที่ Gateway ของตัวเอง (ซึ่งก็คือ Sub-interface บน Router), จากนั้น Router จะดูเส้นทางแล้วส่งข้อมูลต่อไปยัง VLAN 20 ให้เอง

!(<https://www.google.com/search?q=Router-on-a-Stick+diagram&tbm=isch>)

### วิธีที่ 2: ใช้ Layer 3 Switch

สำหรับเครือข่ายขนาดใหญ่ที่มีปริมาณข้อมูลวิ่งข้าม VLAN เยอะๆ การใช้ "Router-on-a-Stick" อาจจะกลายเป็นคอขวดได้ **Layer 3 Switch** จึงเป็นทางเลือกที่ดีกว่า เพราะมันคือสวิตช์ที่มีความสามารถในการ Routing ในตัว!

* **หลักการทำงาน:**

  * เราจะสร้างสิ่งที่เรียกว่า **Interface VLAN** หรือ **SVI (Switched Virtual Interface)** ขึ้นมาบนตัวสวิตช์เอง

  * SVI แต่ละอันจะถูกกำหนด IP Address และทำหน้าที่เป็น Gateway ให้กับ VLAN นั้นๆ โดยตรง

  * ข้อดีคือการ Routing จะเกิดขึ้นด้วยความเร็วสูงระดับ Hardware ภายในตัวสวิตช์เลย ไม่ต้องวิ่งออกไปที่ Router ภายนอก

***

## ทำให้ชีวิตง่ายขึ้นด้วย DHCP 

การต้องเดินไปตั้งค่า IP Address, Subnet Mask, Gateway, DNS ให้กับคอมพิวเตอร์ทุกเครื่องในออฟฟิศคงเป็นเรื่องที่น่าเบื่อและเสียเวลามาก **DHCP (Dynamic Host Configuration Protocol)** คือพระเอกที่เข้ามาช่วยแก้ปัญหานี้

* **หน้าที่หลัก:** เป็นบริการที่คอย **"แจกจ่าย IP Address และข้อมูลเครือข่ายอื่นๆ โดยอัตโนมัติ"** ให้กับเครื่องลูกข่าย (Clients) ที่ร้องขอเข้ามา

* **กระบวนการ (DORA):**

  1. **D**iscover: เครื่องลูกข่ายเปิดเครื่องมาแล้วตะโกนถามในเครือข่ายว่า "มี DHCP Server ไหม?"

  2. **O**ffer: DHCP Server ได้ยิน ก็จะเสนอ IP Address ชุดหนึ่งให้ "เอ้านี่! เอา IP นี้ไปใช้สิ"

  3. **R**equest: เครื่องลูกข่ายตอบกลับว่า "โอเค! ผมขอใช้ IP ชุดนี้แหละ"

  4. **A**cknowledgment: DHCP Server ยืนยันการให้ใช้ IP "ตกลง! IP นี้เป็นของเธอแล้วนะ"

### DHCP Relay Agent (ip helper-address)

แล้วถ้า DHCP Server กับเครื่องลูกข่ายอยู่คนละ VLAN (คนละเครือข่าย) กันล่ะ? โดยปกติแล้ว DHCP Discover เป็น Broadcast message ซึ่งไม่สามารถวิ่งข้ามเครือข่ายได้ เราจึงต้องมี "ผู้ช่วย" ที่เรียกว่า **DHCP Relay Agent**

* **หน้าที่:** เราจะไปตั้งค่าที่ Router (ตรง Gateway ของฝั่งลูกข่าย) ด้วยคำสั่ง `ip helper-address [IP ของ DHCP Server]`

* เมื่อ Router ได้รับ DHCP Discover จากลูกข่าย มันจะแปลง Broadcast message นั้นให้เป็น Unicast message แล้ว "ส่งต่อ" ไปยัง DHCP Server ที่อยู่เครือข่ายอื่นให้โดยตรง

***

## Lab 2: ตั้งค่า Inter-VLAN Routing และ DHCP Server

ใน Lab นี้ เราจะนำความรู้จากหน่วยที่ 1 มาต่อยอด โดยจะใช้ Router 1 ตัว ทำหน้าที่เป็นทั้ง Gateway ให้กับ 2 VLAN และเป็น DHCP Server เพื่อแจก IP ให้กับคอมพิวเตอร์ในแต่ละ VLAN โดยอัตโนมัติ

### แผนภาพเครือข่าย (Topology)

* **อุปกรณ์:**

  * Router 2911: 1 ตัว (R1)

  * Switch 2960: 1 ตัว (SW1)

  * PC: 4 เครื่อง (PC1, PC2, PC3, PC4)

* **VLAN Plan:**

  * **VLAN 10:** SALES (Network: 192.168.10.0/24)

  * **VLAN 20:** IT (Network: 192.168.20.0/24)

* **การเชื่อมต่อ:**

  * PC1, PC2 (VLAN 10) -> SW1 (พอร์ต Fa0/1, Fa0/2)

  * PC3, PC4 (VLAN 20) -> SW1 (พอร์ต Fa0/3, Fa0/4)

  * SW1 (พอร์ต Fa0/24) <--> R1 (พอร์ต Gi0/0) **(Trunk Link)**

### ขั้นตอนการตั้งค่า (Configuration)

#### **1. ตั้งค่าบน SW1 (เหมือนเดิม เพิ่มเติมคือ Interface Range)**

Plaintext

```
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1

! === สร้าง VLANs ===
SW1(config)# vlan 10
SW1(config-vlan)# name SALES
SW1(config-vlan)# exit
SW1(config)# vlan 20
SW1(config-vlan)# name IT
SW1(config-vlan)# exit

! === กำหนด Access Port (ใช้ Range เพื่อความรวดเร็ว) ===
! พอร์ตสำหรับ VLAN 10
SW1(config)# interface range FastEthernet0/1 - 2
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 10
SW1(config-if-range)# exit

! พอร์ตสำหรับ VLAN 20
SW1(config)# interface range FastEthernet0/3 - 4
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 20
SW1(config-if-range)# exit

! === กำหนด Trunk Port ===
SW1(config)# interface FastEthernet0/24
SW1(config-if)# switchport mode trunk
SW1(config-if)# exit

SW1(config)# end
SW1# copy running-config startup-config
```

#### **2. ตั้งค่าบน R1 (Router-on-a-Stick และ DHCP Server)**

Plaintext

```
Router> enable
Router# configure terminal
Router(config)# hostname R1

! === เปิด Interface หลักก่อน ===
Router(config)# interface GigabitEthernet0/0
Router(config-if)# no shutdown
Router(config-if)# exit

! === สร้าง Sub-interface สำหรับ VLAN 10 ===
Router(config)# interface GigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit

! === สร้าง Sub-interface สำหรับ VLAN 20 ===
Router(config)# interface GigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Router(config-subif)# exit

! === ตั้งค่า DHCP Server ===
! สร้าง Pool สำหรับแจก IP ให้ VLAN 10 (SALES)
Router(config)# ip dhcp pool SALES_POOL
Router(dhcp-config)# network 192.168.10.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.10.1
Router(dhcp-config)# exit

! สร้าง Pool สำหรับแจก IP ให้ VLAN 20 (IT)
Router(config)# ip dhcp pool IT_POOL
Router(dhcp-config)# network 192.168.20.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.20.1
Router(dhcp-config)# exit

! === บันทึกการตั้งค่า ===
Router(config)# end
Router# copy running-config startup-config
```

### การตรวจสอบและทดสอบ (Verification & Testing)

1. **ตั้งค่า PC ให้รับ IP อัตโนมัติ:**

   * ไปที่ PC1, PC2, PC3, และ PC4

   * คลิกแท็บ `Desktop` > `IP Configuration`

   * เลือกเป็น `DHCP`

2. **ตรวจสอบการได้รับ IP:**

   * บน **Router R1**, ใช้คำสั่ง:

     ```
     show ip dhcp binding
     ```

     * **ผลที่คาดหวัง:** คุณควรจะเห็นรายการ IP Address ที่ถูกแจกจ่ายให้กับ MAC Address ของ PC ทั้ง 4 เครื่อง โดย PC1/PC2 จะได้ IP ในวง `192.168.10.x` และ PC3/PC4 จะได้ IP ในวง `192.168.20.x`

3. **ทดสอบการเชื่อมต่อ:**

   * **Ping Gateway ของตัวเอง:**

     * จาก PC1 (VLAN 10) `ping 192.168.10.1` -> **ควรจะสำเร็จ (Success)**

   * **Ping ข้าม VLAN:**

     * จาก PC1 (VLAN 10) `ping` ไปยัง PC3 (VLAN 20) -> **ควรจะสำเร็จ (Success)**




# หน่วยที่ 3: การเชื่อมต่อระหว่างเครือข่าย, NAT, และ Access Control List (ACL)

ในสองหน่วยที่ผ่านมา เราได้สร้างและจัดการเครือข่ายภายในองค์กร (LAN) ได้อย่างสมบูรณ์แล้ว แต่ในโลกความจริง องค์กรมักจะมีหลายสาขา หรือต้องการให้พนักงานสามารถเข้าถึงอินเทอร์เน็ตได้ หน่วยนี้จะสอนวิธีเชื่อมต่อเครือข่ายของเราออกสู่โลกภายนอก และวิธีควบคุม "ใคร... ทำอะไร... ที่ไหน" ได้บ้าง

## IP Routing: การหาเส้นทางข้ามเครือข่าย 

เมื่อคอมพิวเตอร์ต้องการส่งข้อมูลไปยังเครือข่ายอื่น (เช่น PC สาขากรุงเทพฯ จะส่งข้อมูลไปหา Server ที่สาขาเชียงใหม่) มันจะส่งข้อมูลไปที่ **Gateway** ของตัวเอง ซึ่งก็คือ **Router** นั่นเอง

**Router** เปรียบเสมือน "บุรุษไปรษณีย์" หรือ "ทางแยกอัจฉริยะ" ที่มี **"ตารางเส้นทาง" (Routing Table)** อยู่ในมือ หน้าที่ของมันคือดู "ที่อยู่ปลายทาง" ของข้อมูล แล้วตัดสินใจว่าจะต้องส่งข้อมูลนี้ออกไปทางประตูไหนจึงจะไปถึงที่หมายได้เร็วและดีที่สุด

การสร้างตารางเส้นทางนี้ทำได้ 2 วิธีหลักๆ:

### 1. Static Routing (บอกเส้นทางด้วยตัวเอง)

เป็นวิธีที่เรา (ผู้ดูแลระบบ) ต้องเข้าไปตั้งค่าที่ Router แต่ละตัวแบบ Manual เลยว่า "ถ้าจะไปเครือข่าย X, ต้องส่งออกไปทางประตู Y นะ"

* **ข้อดี:** 👍

  * **ปลอดภัย:** เพราะเราควบคุมเส้นทางได้ทั้งหมด ไม่มีการประกาศเส้นทางออกไปให้ใครรู้

  * **ประหยัดพลัง Router:** ไม่ต้องใช้ CPU ประมวลผลเพื่อคำนวณเส้นทาง

* **ข้อเสีย:** 👎

  * **ไม่ยืดหยุ่น:** ถ้าเส้นทางหลักล่ม Router จะไม่หาเส้นทางสำรองให้เองโดยอัตโนมัติ เราต้องเข้าไปแก้เอง

  * **ไม่เหมาะกับเครือข่ายขนาดใหญ่:** ถ้ามี 100 สาขา การตั้งค่าด้วยมือทั้งหมดคงเป็นฝันร้าย!

### 2. Dynamic Routing (ให้ Router คุยกันเอง)

เป็นวิธีที่ฉลาดขึ้น เราแค่เปิดใช้งาน "โปรโตคอลการหาเส้นทาง" (Routing Protocol) จากนั้น Router ก็จะเริ่มพูดคุยแลกเปลี่ยนข้อมูลเส้นทางกับเพื่อนบ้าน และสร้างตารางเส้นทางขึ้นมาเองโดยอัตโนมัติ แถมยังปรับเปลี่ยนเส้นทางให้เองเมื่อมีปัญหาอีกด้วย!

* **RIP (Routing Information Protocol):**

  * **เปรียบเทียบ:** เหมือนการ "ตะโกนบอกทาง" Router จะบอกเพื่อนบ้านที่อยู่ติดกันว่า "ฉันไปทางนี้ได้นะ" แล้วเพื่อนบ้านก็จะไปบอกต่อๆ กันไป

  * **การตัดสินใจ:** เลือกเส้นทางที่ผ่าน Router **จำนวนน้อยที่สุด (Hop Count)**

  * **ข้อจำกัด:** เป็นโปรโตคอลรุ่นเก่า ไม่ค่อยฉลาด อาจเลือกเส้นทางที่ช้ากว่าแต่ผ่าน Router น้อยกว่าได้

* **OSPF (Open Shortest Path First):**

  * **เปรียบเทียบ:** เหมือนการที่ Router ทุกตัวได้รับ **"แผนที่" (Map)** ของเครือข่ายทั้งหมด แล้วแต่ละตัวก็ใช้ GPS (อัลกอริทึม) คำนวณหาเส้นทางที่ดีที่สุดจากแผนที่นั้นด้วยตัวเอง

  * **การตัดสินใจ:** เลือกเส้นทางจากค่า **Cost** ซึ่งมักจะคำนวณจาก **ความเร็ว (Bandwidth)** ของลิงก์ ทำให้ได้เส้นทางที่ดีและเร็วกว่า

  * **ข้อดี:** ฉลาด, ทำงานเร็ว, เหมาะกับเครือข่ายทุกขนาด

***

## NAT: ประตูสู่อินเทอร์เน็ต 

คุณเคยสงสัยไหมว่าทำไม IP ที่บ้านเรามักจะเป็น `192.168.1.x` ซึ่งอาจจะซ้ำกับบ้านของคนอื่นได้ แล้วเราใช้อินเทอร์เน็ตพร้อมกันทั่วโลกได้อย่างไร? คำตอบคือ **NAT (Network Address Translation)**

* **ปัญหา:** Public IP Address (IP จริงบนโลกอินเทอร์เน็ต) มีจำนวนจำกัดและมีราคาแพง แต่เรามีอุปกรณ์ที่ต้องการใช้อินเทอร์เน็ตมากมาย (คอม, มือถือ, แท็บเล็ต)

* **ทางออก:** เราจะใช้ **Private IP** (เช่น `192.168.x.x`, `10.x.x.x`) ซึ่งเป็น IP ที่ใช้ซ้ำได้เฉพาะในเครือข่ายภายในของเรา และมี Router ที่ทำหน้าที่เป็น **"นักแปลภาษา"** คอยแปลง Private IP ของเราให้เป็น Public IP ที่มีอยู่จำกัดก่อนจะส่งข้อมูลออกไปสู่อินเทอร์เน็ต

**ประเภทของ NAT ที่ควรรู้จัก:**

* **Static NAT:** เป็นการจับคู่แบบ 1 ต่อ 1 ระหว่าง Private IP กับ Public IP แบบถาวร เหมาะสำหรับ Server ที่ต้องการให้คนข้างนอกเข้าถึงได้ตลอดเวลา (เช่น Web Server)

* **Dynamic NAT:** เป็นการจับคู่ Private IP กับ Public IP จากกลุ่ม (Pool) ที่เราเตรียมไว้ ใครมาก่อนได้ใช้ก่อน เมื่อเลิกใช้ก็จะคืน Pool ให้คนอื่นใช้ต่อ

* **PAT (Port Address Translation) หรือ NAT Overload:** **นิยมใช้มากที่สุด!** เป็นการทำให้อุปกรณ์ภายในหลายๆ เครื่อง สามารถใช้ Public IP **เพียง IP เดียว** เพื่อออกอินเทอร์เน็ตได้พร้อมกัน โดย Router จะใช้ Port Number ที่แตกต่างกันในการแยกแยะว่าข้อมูลชุดไหนเป็นของคอมพิวเตอร์เครื่องไหน

!(<https://www.google.com/search?q=NAT+Overload+(PAT)+diagram&tbm=isch>)

***

## Access Control List (ACL): ยามรักษาความปลอดภัย 

เมื่อเราเชื่อมต่อทุกอย่างเข้าด้วยกันแล้ว ขั้นตอนสุดท้ายคือการควบคุมความปลอดภัย **ACL** คือชุดของ "กฎ" ที่เราเขียนขึ้นมาใส่ไว้ที่ Router เพื่อทำหน้าที่กรอง Packet ข้อมูล

ACL สามารถสั่ง **`permit` (อนุญาต)** หรือ **`deny` (ปฏิเสธ)** ข้อมูลได้ โดยพิจารณาจากเงื่อนไขต่างๆ เช่น

### 1. Standard ACL (เบสิค แต่เร็ว)

* **เงื่อนไข:** ตรวจสอบเฉพาะ **"ที่อยู่ผู้ส่ง" (Source IP Address)** เท่านั้น

* **หลักการวาง:** ควรวางไว้ให้ **"ใกล้กับปลายทาง (Destination)"** มากที่สุด เพื่อไม่ให้ไปบล็อก Traffic อื่นๆ ที่ไม่เกี่ยวข้องโดยไม่ตั้งใจ

### 2. Extended ACL (ละเอียดและทรงพลัง)

* **เงื่อนไข:** ตรวจสอบได้ละเอียดมาก!

  * ที่อยู่ผู้ส่ง (Source IP)

  * ที่อยู่ผู้รับ (Destination IP)

  * ประเภทของโปรโตคอล (TCP, UDP, ICMP)

  * หมายเลขพอร์ต (เช่น Port 80 สำหรับเว็บ, Port 22 สำหรับ SSH)

* **หลักการวาง:** ควรวางไว้ให้ **"ใกล้กับต้นทาง (Source)"** มากที่สุด เพื่อกรอง Traffic ที่ไม่ต้องการทิ้งไปตั้งแต่เนิ่นๆ ไม่ให้วิ่งไปเปลืองทรัพยากรในเครือข่าย

***

## Lab 3: เชื่อม 2 สาขา, ออกเน็ต, และสร้างกฎความปลอดภัย

ใน Lab สุดท้ายนี้ เราจะจำลองสถานการณ์ที่สมจริงขึ้น คือมีสำนักงาน 2 สาขา (BKK และ CNX) ที่เชื่อมต่อกันด้วย OSPF, ทั้งสองสาขาต้องการออกอินเทอร์เน็ต, และเราจะสร้างกฎเพื่อควบคุมการเข้าถึงระหว่างกัน

### แผนภาพเครือข่าย (Topology)

* **อุปกรณ์:**

  * Router 2911: 2 ตัว (R1-BKK, R2-CNX)

  * Switch 2960: 2 ตัว (SW1, SW2)

  * PC: 2 เครื่อง (PC1, PC2)

  * เพิ่ม Cloud หรือ Router อีกตัวเพื่อจำลองเป็น "Internet"

* **Network Plan:**

  * **BKK LAN:** 192.168.10.0/24

  * **CNX LAN:** 192.168.20.0/24

  * **Link ระหว่างสาขา:** 10.0.0.0/30

  * **Internet:** 8.8.8.8 (จำลองเป็น Google DNS)

### ขั้นตอนการตั้งค่า (Configuration)

#### **1. ตั้งค่า IP Address พื้นฐาน และ Routing (OSPF)**

* **บน R1-BKK:**

  ```
  hostname R1-BKK
  ! Interface ไปยัง LAN
  interface GigabitEthernet0/0
   ip address 192.168.10.1 255.255.255.0
   ip nat inside
   no shutdown
  ! Interface ไปยัง R2-CNX (สมมติว่าเป็น WAN Link)
  interface GigabitEthernet0/1
   ip address 10.0.0.1 255.255.255.252
   no shutdown
  ! Interface ไปยัง Internet (สมมติว่าเป็น Gi0/2)
  interface GigabitEthernet0/2
   ip address 203.0.113.1 255.255.255.0  ! (Public IP สมมติ)
   ip nat outside
   no shutdown

  ! ตั้งค่า OSPF
  router ospf 1
   network 192.168.10.0 0.0.0.255 area 0
   network 10.0.0.0 0.0.0.3 area 0
  ```

* **บน R2-CNX:** (ทำคล้ายกัน)

  ```
  hostname R2-CNX
  interface GigabitEthernet0/0
   ip address 192.168.20.1 255.255.255.0
   no shutdown
  interface GigabitEthernet0/1
   ip address 10.0.0.2 255.255.255.252
   no shutdown

  router ospf 1
   network 192.168.20.0 0.0.0.255 area 0
   network 10.0.0.0 0.0.0.3 area 0
  ```

* **ทดสอบ:** PC1 (192.168.10.x) ควรจะ `ping` ไปหา PC2 (192.168.20.x) ได้สำเร็จ

#### **2. ตั้งค่า NAT เพื่อให้ออกอินเทอร์เน็ต (ทำบน R1-BKK)**

```
! บน R1-BKK
! 1. สร้าง ACL เพื่อระบุว่าใครมีสิทธิ์ออกเน็ตได้บ้าง (ในที่นี้คือทั้งสองสาขา)
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255

! 2. ตั้งค่า NAT ให้แปลง IP จาก ACL ข้อ 1 ออกไปเป็น IP ของขา Gi0/2 (ขาออกเน็ต)
ip nat inside source list 1 interface GigabitEthernet0/2 overload

! 3. สร้าง Default Route เพื่อบอกว่าถ้าไม่รู้จักเส้นทางไหน ให้ส่งไปที่ Internet
ip route 0.0.0.0 0.0.0.0 [Next-Hop IP ของ ISP]
```

* **ทดสอบ:** PC1 และ PC2 ควรจะ `ping 8.8.8.8` ได้สำเร็จ

#### **3. ตั้งค่า ACL เพื่อสร้างกฎ (เช่น ห้ามสาขา CNX เข้าถึง File Server ที่ BKK)**

* สมมติว่ามี File Server ที่ BKK ที่ IP `192.168.10.100`

* **บน R1-BKK:**

  ```
  ! สร้าง Extended ACL
  access-list 101 deny ip 192.168.20.0 0.0.0.255 host 192.168.10.100
  access-list 101 permit ip any any

  ! นำ ACL ไปใช้งานที่ขาเข้าจากสาขา CNX
  interface GigabitEthernet0/1
   ip access-group 101 in
  ```

* **ทดสอบ:**

  * PC2 (CNX) `ping 192.168.10.100` -> **ควรจะล้มเหลว (Failed)**

  * PC2 (CNX) `ping` ไปยัง PC1 ที่ BKK -> **ควรจะสำเร็จ (Success)** (เพราะกฎข้อที่ 2 คือ permit ip any any)
