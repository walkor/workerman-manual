# Seguimiento de las llamadas al sistema

Cuando se quiere saber qué está haciendo un proceso, se puede utilizar el comando ```strace``` para rastrear todas las llamadas al sistema de un proceso.

1. Ejecutar ```php start.php status``` para ver la información de los procesos relacionados con Workerman, como se muestra a continuación:

```
Hola admin
---------------------------------------ESTADO GLOBAL--------------------------------------------
Versión de Workerman: 3.0.1
Hora de inicio: 2014-08-12 17:42:04   en ejecución 0 días 1 horas
carga promedio: 3.34, 3.59, 3.67
1 usuario         8 trabajadores       14 procesos
nombre_del_trabajador       estado_de_salida     conteo_de_salidas
BusinessWorker              0                    0
ChatWeb                     0                    0
FileMonitor                 0                    0
Gateway                     0                    0
Monitor                     0                    0
ProveedorEstadístico        0                    0
EstadísticaWeb              0                    0
TrabajadorEstadístico       0                    0
---------------------------------------ESTADO DEL PROCESO-------------------------------------------
pid	memory      escuchando        marca de tiempo  nombre_del_trabajador       solicitud_total pérdida_de_paquetes     errores_de_interrupción_crepitar cierre_de_cliente fallo_de_envío lanzar_excepción éxitoso/total
10352	1.5M    tcp://0.0.0.0:55151  1407836524 ChatWeb           12             0          0            2            0         0               100%
10354	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          0            0            0         0               100%
10355	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           0              0          1            0            0         0               100%
10365	1.25M   tcp://0.0.0.0:55757  1407836524 EstadísticaWeb      0              0          0            0            0         0               100%
10358	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          2            0            0         0               100%
10364	1.25M   tcp://0.0.0.0:55858  1407836524 ProveedorEstadístico 0              0          0            0            0         0               100%
10356	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          2            0            0         0               100%
10366	1.25M   udp://0.0.0.0:55656  1407836524 TrabajadorEstadístico   55             0          0            0            0         0               100%
10349	1.25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    5              0          0            0            0         0               100%
10350	1.25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    0              0          0            0            0         0               100%
10351	1.5M    tcp://127.0.0.1:7373 1407836524 BusinessWorker    5              0          0            0            0         0               100%
10348	1.25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    2              0          0            0            0         0               100%
```

2. Por ejemplo, si queremos saber qué está haciendo el proceso Gateway con el PID 10354, se puede ejecutar el comando ```strace -p 10354``` (es posible que se requieran permisos de root), similar a lo siguiente:

```
sudo strace -p 10354
Proceso 10354 adjunto - presione interrupción para salir
clock_gettime(CLOCK_MONOTONIC, {118627, 242986712}) = 0
gettimeofday({1407840609, 102439}, NULL) = 0
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Llamada al sistema interrumpida)
--- SIGUSR2 (Señal definida por el usuario 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (ahora máscara [])
clock_gettime(CLOCK_MONOTONIC, {118627, 699623319}) = 0
gettimeofday({1407840609, 559092}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118627, 699810499}) = 0
gettimeofday({1407840609, 559277}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Recursos temporalmente no disponibles)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Llamada al sistema interrumpida)
--- SIGUSR2 (Señal definida por el usuario 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (ahora máscara [])
clock_gettime(CLOCK_MONOTONIC, {118628, 699497204}) = 0
gettimeofday({1407840610, 558937}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118628, 699588603}) = 0
gettimeofday({1407840610, 559023}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Recursos temporalmente no disponibles)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Llamada al sistema interrumpida)
--- SIGUSR2 (Señal definida por el usuario 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (ahora máscara [])
```

3. Cada línea representa una llamada al sistema, a partir de esta información es posible ver fácilmente lo que está haciendo el proceso, permitiendo identificar en qué punto se queda el proceso, ya sea en la conexión o la lectura de datos de la red.
