# Falla al detener

## Síntomas:
Al ejecutar ```php start.php stop``` aparece el mensaje ```stop fail```.

### Posibilidad 1
Si Workerman se inició en modo de depuración y el desarrollador envió la señal ```SIGSTOP``` a Workerman presionando ```ctrl z``` en la terminal, Workerman pasa a segundo plano y se suspende, lo que impide que responda al comando de detener (señal ```SIGINT```).

**Solución:**
En la terminal donde se inició Workerman, ingrese ```fg``` (envía la señal ```SIGCONT```) y presione Enter para traer de nuevo a primer plano a Workerman, luego presione ```ctrl c``` (envía la señal ```SIGINT```) para detener a Workerman.

Si no se puede detener, intente ejecutar los siguientes comandos:
```shell
killall -9 php
```
```shell
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

### Posibilidad 2
El usuario que intenta detener a Workerman no es el mismo que lo inició, lo que significa que el usuario de detener no tiene permisos para detener a Workerman.

**Solución:**
Cambia al usuario que inició Workerman o utiliza un usuario con permisos más altos para detener a Workerman.

### Posibilidad 3
El archivo pid del proceso principal de Workerman ha sido eliminado, lo que hace que el script no pueda encontrar el proceso pid y falle al detenerlo.

**Solución:**
Guarda el archivo pid en una ubicación segura, consulta el manual [Worker::$pidFile](../worker/pid-file.md).

### Posibilidad 4
El proceso correspondiente al archivo pid del proceso principal de Workerman no es un proceso de Workerman.

**Solución:**
Abre el archivo pid del proceso principal de Workerman para ver el pid del proceso principal. El archivo pid predeterminado está en el mismo directorio que Workerman. Ejecuta el comando ```ps aux | grep pid-del-proceso-principal``` para ver si el proceso correspondiente es un proceso de Workerman. Si no lo es, puede ser que el servidor se haya reiniciado, lo que hace que el pid guardado por Workerman esté desactualizado y este pid esté siendo utilizado por otro proceso, lo que provoca la falla al detener. En este caso, eliminar el archivo pid debería ser suficiente.

### Posibilidad 5
Si se ha instalado la extensión grpc pero no se ha configurado la variable de entorno correspondiente, al iniciar se creará un proceso montado adicional, lo que ocasionará la falla al detener.

**Solución:**
