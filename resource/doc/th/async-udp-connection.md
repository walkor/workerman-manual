# การเชื่อมต่อแบบไม่สม่ำเสมอแบบไม่ทันสมัย

**(ต้องใช้ workerman เวอร์ชั่น 3.0.8 ขึ้นไป)**

AsyncUdpConnection สามารถทำหน้าที่เป็นไคลเอ่นต์ udp ที่เชื่อมต่อกับเซิร์ฟเวอร์ udp ระยะไกล

โดยทั่วไปแล้ว udp ไม่มีการเชื่อมต่อ แต่เพื่อความสะดวก ที่นี่ AsyncUdpConnection มีชื่อและอินเทอร์เฟซที่คล้ายกับ AsyncTcpConnection

**โปรดทราบ: ต่างจาก AsyncTcpConnection  AsyncUdpConnection ไม่รองรับคุณสมบัติหรือเมธอดต่อไปนี้**
1. ไม่มีคุณสมบัติ connection->id
2. ไม่มีคุณสมบัติ connection->worker
3. ไม่มีคุณสมบัติ connection->transport
4. ไม่มีคุณสมบัติ connection->maxSendBufferSize
5. ไม่มีคุณสมบัติ connection->defaultMaxSendBufferSize
6. ไม่มีคุณสมบัติ connection->maxPackageSize
7. ไม่มีคุณสมบัติ connection->onBufferFull
8. ไม่มีคุณสมบัติ connection->onBufferDrain
9. ไม่มีคุณสมบัติ connection->onError
10. ไม่มีเม็ดท์ที่ชื่อ destroy()
11. ไม่มีเม็ดที่ชื่อ pauseRecv()
12. ไม่มีเม็ดที่ชื่อ resumeRecv()
13. ไม่มีเม็ดที่ชื่อ pipe()
14. ไม่มีเม็ดที่ชื่อ reconnect()

**คุณสมบัติหรือเมธอดที่รองรับของ AsyncUdpConnection**
1. รองรับคุณสมบัติ connection->protocol
2. รองรับเมธอด onMessage ของ connection
3. รองรับเมธอด connect() ของ connection
4. รองรับเมธอด send() ของ connection
5. รองรับเมธอด getRemoteIp() ของ connection
6. รองรับเมธอด getRemotePort() ของ connection
7. รองรับเมธอด onClose โดยนำทางคลายลง

โปรดทราบ: เนื่องจาก tcp ขึ้นอยู่กับการเชื่อมต่อ มักจะเกิดการทำตัวตั้งเรียกระเบิดเมื่อบริษัทใดครึ่งหนึ่งเรียกระเบิดการทำตัวปิดต่อ แต่ udp ไม่มีการเชื่อมต่อ การเรียกระเบิดเมียาที่ชื่อ connection->close() เท่านั้นจะทำให้การทำตัวคลายในท้องถิ่นเรียกระเบิด และไม่สามารถทำให้การทำตัวคลายของคู่ค้าได้
