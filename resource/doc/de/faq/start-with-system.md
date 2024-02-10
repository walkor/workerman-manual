## Wie man Workerman unter Linux automatisch beim Start startet

Öffnen Sie die Datei /etc/rc.local und fügen Sie vor ```exit 0``` einen ähnlichen Code hinzu:

```bash
ulimit -HSn 102400
/usr/bin/env php /pfad/zu/deiner/datei/start.php start -d

exit 0
```
