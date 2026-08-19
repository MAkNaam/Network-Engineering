
<img width="951" height="456" alt="image" src="https://github.com/user-attachments/assets/f7d71ccb-c712-4530-a56a-66c9a697a2d6" />

สาเหตุที่ต้องออกแบบด้วยวิธีนี้ มีเหตุผลหลักๆ 

1. ประหยัดพอร์ตของ Router (Cost-Effective)
พอร์ต (Interface) บน Router มีราคาแพงและมีจำนวนจำกัด หากคุณไม่ใช้วิธีนี้ แล้วต้องการให้แผนก HR (VLAN 10) คุยกับ FINANCE (VLAN 20) ได้ คุณจะต้องลากสายแลน 2 เส้นจาก Switch ไปเสียบเข้า Router 2 พอร์ต (1 พอร์ตต่อ 1 แผนก) ซึ่งจะสิ้นเปลืองทรัพยากรมาก การใช้สาย Trunk เพียงเส้นเดียวเชื่อม Switch กับ Router จึงประหยัดกว่ามาก

2. ใช้ Sub-interface ในการจัดการแทน (Scalability)
จากโค้ดที่คุณคอนฟิกไปก่อนหน้านี้ (int gig0/0.10 และ int gig0/0.20) จะเห็นว่าเราสามารถจำลองพอร์ตเสมือน (Sub-interface) ขึ้นมาบนพอร์ตจริง (gig0/0) ได้เลย ทำให้ในอนาคตถ้ามีแผนกเพิ่ม เช่น IT (VLAN 30) หรือ Sales (VLAN 40) คุณก็ไม่ต้องเดินสายแลนเส้นใหม่ไปที่ Router แค่เข้าไปสร้าง Sub-interface เพิ่มในซอฟต์แวร์ก็ใช้งานได้ทันที

3. แยก Broadcast Domain ลดความหนาแน่นของเครือข่าย
การแบ่งเป็น HR และ FINANCE ช่วยแยกขอบเขตการกระจายข้อมูล (Broadcast Domain) ออกจากกัน ทำให้คอมพิวเตอร์ฝั่ง HR จะไม่ได้รับ Broadcast ขยะจากฝั่ง FINANCE เครือข่ายจึงทำงานได้เร็วและมีเสถียรภาพมากขึ้น แต่เมื่อต้องการให้สองแผนกนี้ส่งไฟล์หากันได้ จึงต้องพึ่งพา Router เป็นตัวกลางในการหาเส้นทาง (Routing) ข้าม VLAN 

4. รวมศูนย์การจัดการความปลอดภัย (Security)
เมื่อทราฟฟิกข้ามแผนกต้องวิ่งผ่าน Router (หรือเปรียบเสมือน Gateway ของแต่ละ VLAN) คุณสามารถเขียนกฎรักษาความปลอดภัย (Access Control List - ACL) ไว้ที่ Router ได้เลย เช่น อนุญาตให้ HR เข้าถึงเซิร์ฟเวอร์ของ FINANCE ได้เฉพาะบางคนเท่านั้น

สรุปคือ: เป็นการออกแบบเพื่อแก้ปัญหา "จะทำอย่างไรให้คอมพิวเตอร์ที่อยู่คนละ VLAN สามารถคุยกันได้ โดยใช้สายแลนและพอร์ตของ Router ให้น้อยที่สุด" 

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
```
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
```
ส่วนที่ 2: การตั้งค่าบน Router0
เปิดใช้งานพอร์ตหลัก และสร้าง Sub-interface เพื่อทำหน้าที่เป็น Default Gateway ให้แต่ละ VLAN
```
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
```
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


