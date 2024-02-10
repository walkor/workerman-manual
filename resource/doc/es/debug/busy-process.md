# Depurar procesos ocupados
A veces, al ejecutar el comando `php start.php status`, podemos ver que hay procesos en estado `busy`, lo que indica que el proceso respectivo está ocupado manejando una tarea. Normalmente, después de completar la tarea, el proceso debería volver al estado `idle`, y por lo general, esto no suele ser un problema. Sin embargo, si el proceso permanece en estado `busy` y no vuelve a `idle` durante mucho tiempo, esto indica que hay un bloqueo o un bucle infinito en el negocio del proceso. Puede localizarlo mediante los siguientes métodos.

## Ubicación utilizando los comandos strace + lsof

**1. Encontrar el PID del proceso ocupado en el estado `busy` en la lista de estados**
Después de ejecutar `php start.php status`, se mostrará algo como esto:
![](../images/d1903ed65ef2f3b0850e84ccbedc52aa.png)
En la imagen, los PIDs de los procesos en estado `busy` son `11725` y `11748`.

**2. Rastrear el proceso usando strace**
Selecciona uno de los PIDs (en este caso, elegimos `11725`) y ejecuta `strace -ttp 11725`, el resultado será algo así:
![](../images/7ce9f36da926f670949609dcdc593ab4.png)
Puedes ver que el proceso está realizando constantemente la llamada al sistema `poll([{fd=16, events=....`, esperando a que el descriptor `fd=16` esté listo para la lectura, es decir, esperando que este descriptor devuelva datos.

Si no muestra ninguna llamada al sistema, mantén esa terminal abierta y abre otra terminal. Luego ejecuta `kill -SIGALRM 11725` (enviar una señal de alarma al proceso) y verifica si la terminal de strace responde o si está bloqueada en alguna llamada al sistema. Si todavía no muestra ninguna llamada al sistema, es probable que el programa esté en un bucle infinito en el negocio, consulta el 2º punto de solución de las causas que hicieron que el proceso permaneciera en estado `busy` por mucho tiempo.

Si el sistema se bloquea en la llamada al sistema epoll_wait o select, esto indica que el proceso ya está en estado `idle`.

**3. Ver los descriptores de archivo del proceso usando lsof**
Ejecuta `lsof -nPp 11725`, y verás algo como esto:
![](../images/27bd629c3a1ac93f9f4b535d01df2ac1.png)
El descriptor 16 corresponde al registro 16u en la última línea, y se puede ver que el descriptor `fd=16` es una conexión TCP con una dirección remota en `101.37.136.135:80`, lo que indica que el proceso está accediendo a un recurso HTTP. El bucle `poll([{fd=16, events=....` significa que el proceso está esperando constantemente a que el servidor HTTP devuelva datos, explicando por qué el proceso está en estado `busy`.

**Solución:**
Una vez que se localiza el bloqueo del proceso, es fácil de solucionar. Por ejemplo, en el caso anterior, después de la localización se determinó que el negocio está realizando una llamada curl y la URL correspondiente no devuelve datos durante mucho tiempo, lo que hace que el proceso espere indefinidamente. En este caso, se puede localizar la razón detrás de la lenta respuesta de la URL contactando al proveedor de la URL, y al mismo tiempo se puede agregar un parámetro de tiempo de espera al llamar a curl, por ejemplo, un tiempo de espera de 2 segundos para evitar bloqueos prolongados (esto puede hacer que el proceso esté en estado `busy` durante aproximadamente 2 segundos).

## Otras razones que causan que el proceso permanezca en estado `busy` por mucho tiempo
Además de causar bloqueos o mantener el proceso en estado `busy`, también hay otras razones que pueden mantener el proceso en estado `busy`.

**1. Error fatal en la aplicación que hace que el proceso salga continuamente**
**Síntoma:** En este caso, la carga del sistema es bastante alta, y el `load average` que se muestra en `status` es 1 o más. El número de `exit_count` del proceso es alto y sigue aumentando.
**Solución:** Ejecuta workerman en modo de depuración (`php start.php start` sin el indicador `-d`) para ver los errores en la aplicación y solucionarlos.

**2. Bucle infinito en el código**
**Síntoma:** En la parte superior, se puede ver que el proceso ocupado está utilizando demasiada CPU, y el comando `strace -ttp pid` no muestra información de ninguna llamada al sistema.
**Solución:** Consulta el artículo del "Bird Brother" y localiza el código PHP a través de [GDB y el código fuente de PHP](https://www.laruence.com/2011/12/06/2381.html). Los pasos resumidos serían aproximadamente los siguientes:
   1. Verifica la versión de PHP con `php -v`.
   2. [Descarga el código fuente correspondiente de PHP](https://www.php.net/releases/).
   3. Ejecuta `gdb --pid=pid del proceso ocupado`.
   4. `source ruta del código fuente de PHP/.gdbinit`.
   5. `zbacktrace` para imprimir la pila de llamadas.
   Con el último paso, puedes ver la pila de llamadas actual del código PHP, es decir, la ubicación del bucle infinito en el código PHP.
   Nota: Si `zbacktrace` no muestra la pila de llamadas, es posible que no hayas compilado PHP con el parámetro `-g`, por lo que necesitarás recompilar PHP y luego reiniciar workerman para la localización.

**3. Agregar temporizadores indefinidamente**
El código de la aplicación agrega temporizadores continuamente pero no los elimina, lo que hace que el proceso tenga cada vez más temporizadores, eventualmente llevando al proceso a ejecutar temporizadores indefinidamente. Por ejemplo, el código a continuación:
```php
$worker = new Worker;
$worker->onConnect = function($con){
    Timer::add(10, function(){});
};
Worker::runAll();
````
En este código, al conectarse un cliente, se agrega un temporizador, pero no hay lógica en el código de la aplicación para eliminar el temporizador. Con el paso del tiempo, se agregarán cada vez más temporizadores al proceso, lo que eventualmente provocará que el proceso ejecute los temporizadores indefinidamente, manteniéndolo en estado `busy`.
Código correcto:
```php
$worker = new Worker;
$worker->onConnect = function($con){
    $con->timer_id = Timer::add(10, function(){});
};
$worker->onClose = function($con){
    Timer::del($con->timer_id);
};
Worker::runAll();
````
En este código corregido, se elimina el temporizador en el evento de cierre, evitando que los temporizadores se acumulen indefinidamente.
