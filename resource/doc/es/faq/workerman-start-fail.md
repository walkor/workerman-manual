# Error al iniciar workerman

## Síntoma 1
Después de iniciar, aparece un error similar a lo siguiente:
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php on line xxxx

```
**Palabras clave**: ```Address already in use```

**Causa raíz**: El puerto está en uso y no se puede iniciar. 

#### Solución 1
Puede usar el comando ```netstat -anp | grep port``` para encontrar qué programa está usando el puerto.
Luego detenga el programa correspondiente para liberar el puerto.

#### Solución 2
Si no puede detener el programa que usa el puerto correspondiente, puede resolverlo cambiando el puerto de workerman.

#### Solución 3
Si workerman está ocupando el puerto y no se puede detener con el comando stop (generalmente se debe a la pérdida del archivo PID o a que el proceso principal fue terminado por el desarrollador), puede matar el proceso de workerman ejecutando los siguientes dos comandos.

``` 
killall php
ps aux|grep WorkerMan|awk '{print $2}'|xargs kill -9
```

#### Solución 4
Si realmente no hay ningún programa escuchando en este puerto, es posible que el desarrollador haya configurado dos o más escuchas en workerman, y las escuchas tienen el mismo puerto. El desarrollador debe verificar si el script de inicio está escuchando en el mismo puerto.

#### Solución 5
Revise si el programa ha habilitado reusePort, intente desactivar reusePort.

## Síntoma 2
Después de iniciar, aparece un error similar a lo siguiente:
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php on line xxxx
```
o
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (en su contexto, la dirección solicitada no es válida) in ...workerman/Worker.php on line xxxx
```
**Palabras clave**:  `Cannot assign requested address` o `en su contexto, la dirección solicitada no es válida`

**Causa del error**: 
El script de inicio tiene mal el parámetro de escucha de la dirección IP. No es la IP de la máquina local. Por favor, ingrese la IP de la máquina local o use ```0.0.0.0``` (para escuchar en todas las IP de la máquina local) para resolverlo.

**Sugerencia**: En sistemas Linux puede usar el comando ```ifconfig``` para ver todas las direcciones IP de las tarjetas de red de la máquina local.
Si es usuario de un servidor en la nube (como Alibaba Cloud/Tencent Cloud, etc.), tenga en cuenta que su IP pública podría ser una IP de proxy (por ejemplo, la red privada virtual de Alibaba Cloud) que no pertenece al servidor actual, por lo que no se puede escuchar en la IP pública. Aunque no se puede escuchar en la IP pública, aún se puede vincular usando ```0.0.0.0```.

## Síntoma 3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**Causa del error**: 
La función stream_socket_server ha sido deshabilitada en php.ini

**Solución**:

1. Ejecute ```php --ini``` para encontrar el archivo php.ini
2. Abra php.ini y busque la entrada disable_functions, elimine la entrada de deshabilitación de stream_socket_server.

## Síntoma 4
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```
**Causa del error**: 
En Linux, si el puerto es menor que 1024, se necesita permiso de root para escuchar.

**Solución**

Use un puerto mayor a 1024 o inicie el servicio como usuario root.
