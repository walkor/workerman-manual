# Ajuste del núcleo de Linux

Para que el sistema pueda admitir una mayor concurrencia, además de la [instalación de la extensión event](../install/install.md) que es obligatoria, la optimización del núcleo de Linux es **de suma importancia**. Cada optimización a continuación es **extremadamente importante**, así que asegúrese de completar cada una de ellas paso a paso.

**Explicación de parámetros:**

> **max-file**: representa la cantidad de mangos de archivo que la **naturaleza del sistema** puede abrir. Se refiere al sistema operativo en su conjunto, no a los usuarios.
> **ulimit -n**: controla la cantidad de mangos de archivo que el proceso puede abrir a nivel de cada **proceso**. Regula el mango de archivo disponible para el usuario actual en el shell y para los procesos iniciados por él.

Para ver la cantidad de mangos de archivo que el **sistema** puede abrir: `cat /proc/sys/fs/file-max`

Abra el archivo /etc/sysctl.conf y agregue la siguiente configuración:

```conf
# Este parámetro establece la cantidad de TIME_WAIT en el sistema. Si supera el valor predeterminado, se eliminará de inmediato
net.ipv4.tcp_max_tw_buckets = 20000
# Define la longitud máxima de la cola de escucha para cada puerto en el sistema. Este es un parámetro global.
net.core.somaxconn = 65535
# El número máximo de conexiones pendientes que aún no han sido confirmadas por el otro lado.
net.ipv4.tcp_max_syn_backlog = 262144
# Cuando la tasa a la que la interfaz de red recibe paquetes es más rápida que la tasa a la que el kernel los procesa, establece el número máximo de paquetes que se permitirán enviar a la cola.
net.core.netdev_max_backlog = 30000
# Esta opción provoca que los clientes en una red NAT caduquen. Se recomienda establecerlo en 0. A partir del kernel 4.12, Linux eliminó la configuración tcp_tw_recycle. Si aparece el error "No such file or directory", se debe ignorar.
net.ipv4.tcp_tw_recycle = 0
# La cantidad total de archivos que todos los procesos del sistema pueden abrir
fs.file-max = 6815744
# Tamaño de la tabla de seguimiento del firewall. Nota: si el firewall no está habilitado, se mostrará el error: "net.netfilter.nf_conntrack_max" is an unknown key. Se debe ignorar.
net.netfilter.nf_conntrack_max = 2621440
net.ipv4.ip_local_port_range = 10240 65000
```

Ejecute `sysctl -p` para que los cambios surtan efecto de inmediato.

**Nota:**

/etc/sysctl.conf tiene muchas opciones configurables. Se pueden configurar otras opciones según las necesidades del entorno.

## Apertura de archivos

Configure el número de archivos abiertos en el sistema para resolver el problema de ```too many open files``` en alta concurrencia. Esta opción afecta directamente la cantidad máxima de conexiones de clientes que puede manejar un solo proceso.

Los archivos abiertos suaves (Soft open files) es un parámetro del sistema Linux que afecta la cantidad máxima de mangos de archivo que un solo proceso puede abrir. Este valor afectará a las aplicaciones de conexión prolongada, como el mantenimiento de la cantidad de conexiones de usuarios que un solo proceso puede manejar en aplicaciones como chat. Al ejecutar `ulimit -n`, se puede ver el valor de este parámetro. Si es 1024, significa que un solo proceso solo puede mantener hasta 1024 o menos (debido a otros mangos de archivo abiertos) conexiones simultáneas. Si se inician 4 procesos para mantener las conexiones de usuarios, el número máximo de conexiones que la aplicación puede manejar simultáneamente no superará las 4 * 1024, es decir, como máximo solo podrá admitir la presencia de 4x1024 usuarios en línea. 

**Tres formas de modificar los archivos abiertos suaves (Soft open files):**

Primera forma: ejecute `ulimit -HSn 102400` directamente en la terminal y luego reinicie workerman.

Esto solo es válido en la terminal actual; una vez que se cierre, el número de archivos abiertos volverá a su valor predeterminado.

Segunda forma: agregue `ulimit -HSn 102400` al final del archivo `/etc/profile`. De esta manera, se ejecutará automáticamente cada vez que se inicie la terminal. Después de realizar el cambio, workerman deberá reiniciarse.

Tercera forma: para que el cambio en el número de archivos abiertos sea permanente, es necesario modificar el archivo de configuración `/etc/security/limits.conf`. Agregue lo siguiente al final del archivo:

``` 
* soft nofile 1024000
* hard nofile 1024000
root soft nofile 1024000
root hard nofile 1024000
```

Este método requiere reiniciar el servidor para que surta efecto.
