## 在 Linux 系統中，如何讓 Workerman 在開機時自動啟動

請打開 /etc/rc.local，在 ```exit 0``` 之前添加以下類似的程式碼：

```shell
ulimit -HSn 102400
/usr/bin/env php /磁碟/路徑/start.php start -d

exit 0
```
