
<img width="951" height="456" alt="image" src="https://github.com/user-attachments/assets/f7d71ccb-c712-4530-a56a-66c9a697a2d6" />

รายงานสรุปการตั้งค่าระบบเครือข่าย Inter-VLAN Routing (Router-on-a-Stick)
1. วัตถุประสงค์
เพื่อตั้งค่าและทดสอบระบบเครือข่ายแบบ Router-on-a-Stick ทำให้เครื่องคอมพิวเตอร์ที่อยู่ต่าง VLAN (HR และ FINANCE) สามารถสื่อสารและส่งข้อมูลข้ามเครือข่ายกันได้ผ่านอุปกรณ์ Router

2. อุปกรณ์ที่ใช้ในการจำลอง (Topology)
Router (Router0): รุ่น 2911 จำนวน 1 เครื่อง (ทำหน้าที่เป็น Gateway เชื่อมระหว่าง VLAN)

Switch (Switch0): รุ่น 2960-24TT จำนวน 1 เครื่อง (ทำหน้าที่แบ่ง VLAN และเชื่อมต่อ End Devices)

End Devices (PC): จำนวน 4 เครื่อง (แบ่งเป็น PC0, PC1 สำหรับกลุ่ม HR และ PC2, PC3 สำหรับกลุ่ม FINANCE)

3. การแบ่ง VLAN และหมายเลข IP (IP Addressing Scheme)

<img width="1189" height="140" alt="image" src="https://github.com/user-attachments/assets/cf3d06cc-3fb9-4b8f-8807-e59d3435c936" />

4. ขั้นตอนการตั้งค่าและคำสั่ง (Configuration)
ส่วนที่ 1: การตั้งค่าบน Switch0
สร้าง VLAN, กำหนดพอร์ตให้กับแต่ละ VLAN (Access Mode) และเปิดพอร์ตเชื่อมต่อกับ Router ให้เป็นโหมด Trunk

Switch>en
Switch#conf t
! สร้างและตั้งชื่อ VLAN
Switch(config)#vlan 10
Switch(config-vlan)#name HR
Switch(config-vlan)#vlan 20
Switch(config-vlan)#name FINANCE
Switch(config-vlan)#exit
! กำหนดพอร์ต Access ให้ VLAN 10 และ 20
Switch(config)#int range fa0/1-9
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport access vlan 10
Switch(config-if-range)#exit
Switch(config)#int range fa0/10-19
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport access vlan 20
Switch(config-if-range)#exit
! กำหนดพอร์ต Trunk เชื่อมไปยัง Router
Switch(config)#int gig0/1
Switch(config-if)#switchport mode trunk
Switch(config-if)#exit

ส่วนที่ 2: การตั้งค่าบน Router0
เปิดใช้งานพอร์ตหลัก และสร้าง Sub-interface เพื่อทำหน้าที่เป็น Default Gateway ให้แต่ละ VLAN

Router>en
Router#conf t
! เปิดการทำงานของพอร์ตหลัก
Router(config)#int gig0/0
Router(config-if)#no shut
Router(config-if)#exit
! สร้าง Sub-interface สำหรับ VLAN 10 (HR)
Router(config)#int gig0/0.10
Router(config-subif)#encapsulation dot1q 10
Router(config-subif)#ip address 192.168.10.1 255.255.255.0
Router(config-subif)#exit
! สร้าง Sub-interface สำหรับ VLAN 20 (FINANCE)
Router(config)#int gig0/0.20
Router(config-subif)#encapsulation dot1q 20
Router(config-subif)#ip address 192.168.20.1 255.255.255.0
Router(config-subif)#exit

ส่วนที่ 3: การตั้งค่า End Devices (PC)
เข้าไปที่เมนู Desktop > IP Configuration เพื่อตั้งค่า Static IP

PC0/PC1 (HR): ใส่ IP ในช่วง 192.168.10.x, Subnet 255.255.255.0 และตั้ง Default Gateway เป็น 192.168.10.1

PC2/PC3 (FINANCE): ใส่ IP ในช่วง 192.168.20.x, Subnet 255.255.255.0 และตั้ง Default Gateway เป็น 192.168.20.1

5. ปัญหาที่พบและกระบวนการแก้ไข (Troubleshooting)
ระหว่างการทดสอบระบบ (Ping ข้ามเครือข่าย) พบว่าไม่สามารถส่งข้อมูลระหว่าง VLAN ได้ โดยระบบแจ้งเตือน "Request timed out" ซึ่งได้ดำเนินการแก้ไขตามลำดับดังนี้:

ปัญหาที่ 1: การกำหนด IP Address ซ้ำซ้อนที่พอร์ตหลักของ Router

สาเหตุ: มีการใส่ IP Address ไว้ที่ Interface หลัก (gig0/0) ของ Router0 ทำให้ตาราง Routing เกิดความสับสน เนื่องจากการทำ Router-on-a-Stick จะต้องกำหนด IP ไว้ที่ Sub-interface เท่านั้น

วิธีแก้ไข: เข้าไปที่ Router0 ในโหมด Interface gig0/0 แล้วใช้คำสั่ง no ip address เพื่อลบการตั้งค่า IP เดิมออก

ปัญหาที่ 2: การเชื่อมต่อสายสัญญาณผิดพอร์ตบน Switch (Physical Layer Issue)

สาเหตุ: เมื่อทดสอบปิงจาก PC กลุ่ม FINANCE (VLAN 20) ไปยัง Gateway ของตนเอง (192.168.20.1) ไม่สำเร็จ พบว่าสายเชื่อมต่อจากเครื่อง PC เสียบเข้ากับพอร์ตของ Switch0 ไม่ตรงกับที่กำหนดไว้ (เสียบผิดไปเข้าพอร์ตของกลุ่ม VLAN 10)

วิธีแก้ไข: ทำการลบสายเคเบิลเดิมออก และเดินสายใหม่ (Copper Straight-Through) โดยบังคับเลือกเสียบเข้าที่พอร์ต Fa0/10 ถึง Fa0/19 ซึ่งเป็นพอร์ตที่ตั้งค่า Access VLAN 20 ไว้เรียบร้อยแล้ว

ผลการแก้ไข: หลังจากการแก้ไขทั้ง 2 จุด รวมถึงตรวจสอบความถูกต้องของ Default Gateway ในเครื่อง PC ทั้งหมด ระบบสามารถปิงเชื่อมต่อกันข้าม VLAN ได้สำเร็จ 100% (ได้รับข้อความ Reply และค่า TTL ลดลงตามมาตรฐานการ Routing)
