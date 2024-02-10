## Come avviare automaticamente Workerman all'avvio del sistema su Linux

Apri il file /etc/rc.local e aggiungi il seguente codice prima di "exit 0":

```bash
ulimit -HSn 102400
/usr/bin/env php /percorso/del/disco/start.php start -d

exit 0
```
