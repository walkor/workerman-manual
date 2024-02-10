## Cómo iniciar automáticamente Workerman al arrancar en Linux

Abre el archivo /etc/rc.local y agrega un código similar al siguiente antes de "exit 0":

```shell
ulimit -HSn 102400
/usr/bin/env php /ruta/del/disco/start.php start -d

exit 0
```

Asegúrate de reemplazar "/ruta/del/disco" con la ruta real de tu archivo de inicio de Workerman.
