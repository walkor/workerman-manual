Workermanสามารถทำหน้าที่เป็นไคลเอ็นต์ในการรับและประมวลผลข้อมูลจากเซิร์ฟเวอร์ระยะไกลได้หรือไม่?

สามารถใช้ AsyncTcpConnection เพื่อเริ่มการเชื่อมต่อแบบไม่ซิงโครนัส ให้ workerman ทำหน้าที่เป็นไคลเอ็นต์ที่จะติดต่อกับเซิร์ฟเวอร์ได้

ตัวอย่างเช่น:

1. [Workerman ทำหน้าที่เป็นไคลเอ็นต์ของ WebSocket](as-wss-client.md)

2. [Workerman ทำหน้าที่เป็นตัวแทน MySQL](../async-tcp-connection/connect.md)

3. [Workerman ทำหน้าที่เป็นไคลเอ็นต์ HTTP](../async-tcp-connection/construct.md)

4. [Workerman ทำหน้าที่เป็นตัวแทน HTTP](https://github.com/walkor/php-http-proxy)

5. [Workerman ทำหน้าที่เป็นตัวแทน SOCKS5](https://github.com/walkor/php-socks5)
