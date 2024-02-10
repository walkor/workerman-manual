# Desactivar la comprobación de funciones

Usa este script para comprobar si hay funciones desactivadas. Ejecuta el comando en la línea de comandos: ```curl -Ss https://www.workerman.net/check | php```.

Si aparece el mensaje ```Function nombre_de_función puede estar deshabilitada. Por favor, verifica las disable_functions en php.ini```, significa que las funciones necesarias para workerman están deshabilitadas y que es necesario habilitarlas en php.ini para poder usar workerman correctamente.

Para habilitarlas, puedes elegir cualquiera de los siguientes dos métodos.

## Método 1: Habilitar con el script

Ejecuta el script `curl -Ss https://www.workerman.net/fix | php` para habilitar las funciones.

## Método 2: Habilitar manualmente

**Sigue los siguientes pasos:**

1. Ejecuta `php --ini` para encontrar la ubicación del archivo php.ini utilizado por php cli.

2. Abre php.ini, encuentra el elemento `disable_functions` y habilita las funciones correspondientes.

**Funciones necesarias**
Para usar workerman, es necesario habilitar las siguientes funciones:
```plaintext
stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
