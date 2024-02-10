## Linux işletim sisteminde workerman'ın otomatik olarak başlatılması için nasıl yapılandırılır?

/etc/rc.local dosyasını açın ve aşağıdaki kod parçacığını ```exit 0``` satırından önce ekleyin:

```
ulimit -HSn 102400
/usr/bin/env php /disk/yeri/start.php start -d

exit 0
```
