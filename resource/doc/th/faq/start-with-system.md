## การเริ่มต้นอัตโนมัติของ Workerman ในระบบ Linux

เปิดไฟล์ /etc/rc.local และเพิ่มโค้ดที่คล้ายกับตัวอย่างต่อไปนี้ ไปที่ ```exit 0```

```shell
ulimit -HSn 102400
/usr/bin/env php /path/to/your/disk/start.php start -d

exit 0
```
