# Verificar estado de ejecución

Ejecutando ```php start.php status``` se puede verificar el estado de ejecución de WorkerMan, similar al siguiente ejemplo:

```plaintext
----------------------------------------------ESTADO GLOBAL-----------------------------------------------------
Versión de Workerman: 3.5.13          Versión de PHP: 5.5.9-1ubuntu4.24
Hora de inicio: 2018-02-03 11:48:20   ejecución 112 días 2 horas 
carga promedio: 0, 0, 0              event-loop:\Workerman\Events\Event
4 workers       11 procesos
Nombre del trabajador        estado_salida      contador_salida
ChatBusinessWorker            0                  0
ChatGateway                   0                  0
Register                      0                  0
WebServer                     0                  0
----------------------------------------------ESTADO DEL PROCESO-----------------------------------------------------
pid	memory  listening                Nombre del trabajador        conexiones envio_fallido temporizadores  solicitudes_totales qps    estado
18306	2.25M   ninguno                 ChatBusinessWorker           5           0           0               11                0       [inactivo]
18307	2.25M   ninguno                 ChatBusinessWorker           5           0           0               8                 0       [inactivo]
18308	2.25M   ninguno                 ChatBusinessWorker           5           0           0               3                 0       [inactivo]
18309	2.25M   ninguno                 ChatBusinessWorker           5           0           0               14                0       [inactivo]
18310	2M      websocket://0.0.0.0:7272 ChatGateway                  8           0           1               31                0       [inactivo]
18311	2M      websocket://0.0.0.0:7272 ChatGateway                  7           0           1               26                0       [inactivo]
18312	2M      websocket://0.0.0.0:7272 ChatGateway                  6           0           1               21                0       [inactivo]
18313	1.75M   websocket://0.0.0.0:7272 ChatGateway                  5           0           1               16                0       [inactivo]
18314	1.75M   texto://0.0.0.0:1236     Register                     8           0           0               8                 0       [inactivo]
18315	1.5M    http://0.0.0.0:55151     WebServer                    0           0           0               0                 0       [inactivo]
18316	1.5M    http://0.0.0.0:55151     WebServer                    0           0           0               0                 0       [inactivo]
----------------------------------------------ESTADO DEL PROCESO-----------------------------------------------------
Resumen	18M     -                        -                           54          0           4               138               0       [Resumen]
```

## Descripción

### ESTADO GLOBAL

Desde esta sección, se puede observar 

La versión de WorkerMan ```versión:3.5.13```

El tiempo de inicio ```2018-02-03 11:48:20```, con una ejecución de ```112 días 2 horas ```

La carga del servidor ```carga promedio: 0, 0, 0```, mostrando la carga promedio del sistema en los últimos 1, 5 y 15 minutos respectivamente

El uso de la biblioteca de eventos de E/S, ```event-loop:\Workerman\Events\Event```

 ```4 workers``` (3 tipos de procesos, incluyendo procesos de ChatGateway, ChatBusinessWorker, Registro y WebServer)

 ``` 11 procesos ``` (un total de 11 procesos)

 ``` nombre_del_trabajador ``` (nombre del proceso del trabajador)

 ``` estado_salida ``` (código de estado de salida del proceso del trabajador)

 ``` contador_salida ``` (número de veces que este código de estado de salida se ha producido)

En general, un estado_salida de 0 indica una salida normal. Cualquier otro valor indica una terminación inesperada del proceso y generará un mensaje de error similar a "WORKER EXIT UNEXPECTED". Los mensajes de error se registrarán en el archivo especificado por [Worker::logFile](worker/log-file.md).

**Los códigos de estado_salida comunes y sus significados son los siguientes:**

* 0: Indica una salida normal, que ocurre después de un reinicio suave durante una recarga. Es importante notar que el uso de la función exit o die en el código de la aplicación también generará una salida con código 0, y producirá un mensaje de error "WORKER EXIT UNEXPECTED". Workerman no permite que el código de la aplicación contenga las declaraciones exit o die.
* 9: Indica que el proceso fue terminado por la señal SIGKILL. Este código de salida se produce principalmente en detenciones y recargas suaves, debido a que el proceso hijo no responde a tiempo a la señal de recarga del proceso principal (por ejemplo, bloqueos prolongados durante la espera de MySQL, curl u otras operaciones largas, o bucles de código de aplicación). Es importante destacar que enviar la señal SIGKILL al proceso hijo a través del comando kill en la consola de Linux también generará este código de salida.
* 11: Indica que PHP ha generado un archivo de volcado de memoria. Normalmente, esto se debe al uso de extensiones inestables; en ese caso, se debe comentar la extensión correspondiente en php.ini. En algunos casos raros, puede tratarse de un error en PHP, y en ese caso se deberá actualizar PHP.
* 65280: Este código de salida se debe a un error fatal en el código de la aplicación, como llamar a una función inexistente, errores de sintaxis, entre otros. Los detalles de los errores se registrarán en el archivo especificado por [Worker::logFile](worker/log-file.md), o en el archivo especificado por [error_log](https://php.net/manual/zh/errorfunc.configuration.php#ini.error-log) de php.ini (si se ha especificado alguno).
* 64000: Se debe a que el código de la aplicación ha lanzado una excepción que no fue capturada, lo que resulta en la terminación del proceso. Si Workerman se está ejecutando en modo de depuración, la pila de llamadas de la excepción se imprimirá en la consola. Si se ejecuta en modo daemon, la pila de llamadas de la excepción se registrará en el archivo especificado por [Worker::stdoutFile](worker/stdout-file.md).

## ESTADO DEL PROCESO

pid: ID de proceso

memory: Memoria actualmente utilizada por el proceso (excluyendo la memoria ocupada por el propio archivo ejecutable de PHP)

listening: Protocolo de transporte y dirección IP / Puerto de escucha. Si no escucha ningún puerto, aparecerá "ninguno". Ver [Función de construcción de la clase Worker](worker/construct.md)

worker_name: Nombre del servicio ejecutado por el proceso

connections: Cantidad **actual** de instancias de conexión TCP que tiene el proceso. Esto incluye conexiones TcpConnection y AsyncTcpConnection. Este valor es en tiempo real, no acumulativo. Es importante destacar que si una instancia de conexión llama a close y el contador no disminuye, puede ser que el código de la aplicación esté guardando el objeto $connection, evitando que la conexión se destruya.

total_request: Cantidad total de solicitudes recibidas por el proceso desde su inicio. Estas solicitudes incluyen las enviadas por el cliente, así como las solicitudes internas de Workerman generadas por la interacción entre Gateway y BusinessWorker en la arquitectura de GatewayWorker. Este valor es acumulativo.

send_fail: Cantidad de veces que el proceso ha fallado al enviar datos al cliente. Este valor suele ser distinto de 0 en situaciones normales, debido a desconexiones de clientes, ver [razones del envío fallido en el estado](../faq/about-send-fail.md). Este valor es acumulativo.

timers: Cantidad actual de temporizadores activos en el proceso (excluyendo los temporizadores eliminados y los temporizadores de un solo uso que ya se han ejecutado). Nota: Esta característica requiere Workerman versión>=3.4.7.

qps: Cantidad de solicitudes de red que recibe el proceso por segundo. Es importante notar que este valor solo se muestra cuando se utiliza status con la opción ```-d```, de lo contrario se mostrará 0. Esta característica requiere Workerman versión>=3.5.2.

estado: Estado del proceso. Si es idle significa que está inactivo, si es busy significa que está ocupado. Es importante destacar que si el proceso está ocupado por un breve período de tiempo, esto es considerado como normal. Si el proceso permanece ocupado durante un tiempo prolongado, es posible que se haya producido un bloqueo de la aplicación o un bucle infinito, lo que requerirá investigación según el apartado [Depurar procesos ocupados](busy-process.md). Esta característica requiere Workerman versión>=3.5.0.

## Principio

Después de ejecutar el script de estado, el proceso principal enviará la señal ```SIGUSR2``` a todos los procesos trabajadores. Luego, el script de estado entrará en una breve fase de suspensión para esperar los resultados del estado de cada proceso trabajador. Durante este tiempo, los procesos trabajadores inactivos recibirán la señal ```SIGUSR2``` y escribirán su propio estado de ejecución (cantidad de conexiones, solicitudes, etc.) en un archivo específico en disco, mientras que los procesos trabajadores ocupados esperarán hasta que hayan completado su lógica empresarial antes de escribir su información de estado. Después de la breve suspensión, el script de estado leerá los archivos de estado en disco y mostrará los resultados en la consola.

## Nota

Al ejecutar el estado es posible que algunos procesos aparezcan ocupados. Esto se debe a que los procesos están ocupados manejando la lógica empresarial (por ejemplo, bloqueos largos mientras se esperan respuestas de solicitudes a bases de datos o servidores web, o ejecución de bucles extensos), lo que evita que informen su estado, mostrando así el estado de ocupado.

Si ocurren estos problemas, es necesario revisar el código de la aplicación para identificar dónde están ocurriendo los bloqueos y evaluar si el tiempo de bloqueo está dentro de lo esperado. En caso de que no lo sea, es necesario investigar y depurar el código de la aplicación utilizando la sección [Depurar procesos ocupados](busy-process.md).
