# El cierre del terminal causa el cierre del servicio

**Pregunta:**

¿Por qué Workerman se cierra automáticamente cuando cierro el terminal?

**Respuesta:**

Workerman tiene dos modos de inicio, el modo de depuración debug y el modo de proceso de demonio daemon.

Ejecutar ```php xxx.php start``` inicia el modo de depuración debug, utilizado para desarrollar y depurar problemas. Cuando se cierra el terminal, Workerman se cerrará automáticamente.

Ejecutar ```php xxx.php start -d``` inicia el modo de proceso de demonio daemon, y el cierre del terminal no afectará a Workerman.

Si se quiere que Workerman no se vea afectado por el cierre del terminal, se puede iniciar en modo de demonio.
