## Cách tự động khởi động workerman trên hệ thống Linux

Bước đầu tiên là mở tập tin /etc/rc.local và thêm mã tương tự như sau trước dòng ```exit 0```:

```
ulimit -HSn 102400
/usr/bin/env php /đường/dẫn/tới/tệp/start.php start -d

exit 0
```
