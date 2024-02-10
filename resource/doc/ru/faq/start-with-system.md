## Как настроить автозапуск workerman в системе Linux?

Откройте файл /etc/rc.local и добавьте следующий код перед строкой ```exit 0```:

```
ulimit -HSn 102400
/usr/bin/env php /путь/к/файлу/start.php start -d

exit 0
```
