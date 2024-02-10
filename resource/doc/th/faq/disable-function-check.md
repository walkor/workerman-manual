การตรวจสอบฟังก์ชันที่ถูกปิดใช้งาน

ใช้สคริปต์นี้เพื่อตรวจสอบว่ามีฟังก์ชันที่ถูกปิดใช้งานหรือไม่ ใช้อะไรไฟล์ ```curl -Ss https://www.workerman.net/check | php``` ผลลัพธ์ ถ้ามีการแจ้งว่า```Function ชื่อฟังก์ชัน may be disabled. Please check disable_functions in php.ini``` นั้นหมายความว่า ฟังก์ชันที่ workerman ต้องการถูกปิดใช้งาน จำเป็นต้องยกเลิกการปิดใช้งานใน php.ini เพื่อให้ workerman ทำงานได้อย่างปกติ
เพื่อยกเลิกการปิดใช้งาน สามารถทำได้โดยองค์หนึ่งที่เลือกใช้สคริปต์ดังนี้

## วิธีการที่ 1: ยกเลิกโดยสคริปต์

รันสคริปต์ `curl -Ss https://www.workerman.net/fix | php`เพื่อยกเลิกการปิดใช้งาน

## วิธีการที่ 2: ยกเลิกโดยการดำเนินการด้วยตนเอง

**ขั้นตอนดังต่อไป:**

1.รัน `php --ini` เพื่อค้นหาตำแหน่งของไฟล์ php.ini ที่ใช้งานโดย php cli

2.เปิด php.ini และค้นหาในส่วนของ `disable_functions` เพื่อยกเลิกการปิดใช้งานของฟังก์ชันที่เกี่ยวข้อง

**ฟังก์ชันที่เกี่ยวข้อง**
การใช้งาน workerman จำเป็นต้องยกเลิกการปิดใช้งานฟังก์ชันต่อไปนี้
``` 
stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
