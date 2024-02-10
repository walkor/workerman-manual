## Comment démarrer automatiquement Workerman au démarrage sur un système Linux

Ouvrez le fichier /etc/rc.local et ajoutez le code suivant avant ```exit 0``` :

```bash
ulimit -HSn 102400
/usr/bin/env php /chemin/vers/le/fichier/start.php start -d

exit 0
```
