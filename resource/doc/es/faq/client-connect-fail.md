# Motivos de falla en la conexión del cliente

Generalmente, hay dos tipos de errores cuando la conexión del cliente falla: ```connection refuse``` (conexión rechazada) y ```connection timeout``` (tiempo de espera de conexión).

## connection refuse (conexión rechazada)

Por lo general, las razones son las siguientes:
1. El cliente se conecta a un puerto incorrecto.
2. El cliente se conecta a un nombre de dominio o IP incorrecto.
3. Si el cliente utiliza un nombre de dominio para la conexión, es posible que el nombre de dominio apunte a una IP de servidor incorrecta.
4. El servidor utiliza servicios de aceleración como CDN, lo que hace que la IP real de la conexión no coincida con la IP esperada.
5. El servidor no está en ejecución o el puerto no está siendo escuchado.
6. Se está utilizando un software de proxy de red.
7. La IP escuchada por el servidor y la dirección de acceso no están en el mismo segmento de direcciones. Por ejemplo, si el servidor escucha en 127.0.0.1, el cliente solo puede conectarse a través de 127.0.0.1, no a través de la IP de LAN o la IP pública. Se recomienda configurar la dirección de escucha como 0.0.0.0, de modo que la conexión pueda ser establecida desde el host local, la LAN y el internet público.

## connection timeout (tiempo de espera de conexión)

Por lo general, las razones son las siguientes:
1. El firewall del servidor está bloqueando la conexión. Se puede intentar desactivar temporalmente el firewall para probar.
2. En el caso de servidores en la nube, es posible que el grupo de seguridad esté bloqueando el establecimiento de la conexión, por lo que es necesario abrir el puerto correspondiente en el panel de control.
3. Si se está utilizando un panel de control como Baota, es necesario abrir el puerto correspondiente en Baota.
4. El servidor no existe o no está en ejecución.
5. Si el cliente utiliza un nombre de dominio para la conexión, es posible que el nombre de dominio apunte a una IP de servidor incorrecta.
6. El cliente accede a una IP interna del servidor, y el cliente y el servidor no están en la misma red local.

## cannot assign requested address (no se puede asignar la dirección solicitada)

**Como cliente**, cada vez que se inicia una conexión, se utiliza un puerto temporal local. Por defecto, un servidor puede utilizar aproximadamente de 20,000 a 30,000 puertos temporales. Si el número de conexiones realizadas a un servidor específico excede esta cantidad, no se podrán asignar más puertos temporales, lo que ocasionará este error.
Se puede aumentar la cantidad de puertos temporales locales modificando el parámetro del kernel `/etc/sysctl.conf` llamado `net.ipv4.ip_local_port_range`, por ejemplo, estableciéndolo en `10000 65535` (aumentando así la cantidad de puertos locales a 55,535). Posteriormente, se debe ejecutar el comando `sysctl -p` para que surtan efecto.
Además, al cerrarse la conexión, esta pasa al estado TIME_WAIT, ocupando el puerto local correspondiente durante un tiempo, lo que significa que si se realizan muchas conexiones cortas en un corto período de tiempo (más de 20,000 a 30,000), también se generará el error `Cannot assign requested address`. En este caso, se puede solucionar configurando la rápida reutilización de puertos TIME_WAIT a nivel del kernel. Para más información, consultar la [optimización del kernel](https://www.workerman.net/doc/workerman/appendices/kernel-optimization.html).

> **Nota**
> La limitación de la cantidad de puertos locales es exclusiva del cliente. El servidor no tiene limitaciones en cuanto a la cantidad de puertos locales. Si los recursos son suficientes, el servidor puede mantener una cantidad prácticamente ilimitada de conexiones.

## Otros errores
Si surge un error que no es ```connection refuse``` o ```connection timeout```, generalmente se debe a lo siguiente:

**1. El protocolo de comunicación utilizado por el cliente no coincide con el del servidor.**
Por ejemplo, si el servidor utiliza el protocolo de comunicación HTTP, el cliente no podrá conectarse utilizando el protocolo de comunicación de WebSocket. Si el cliente utiliza el protocolo de WebSocket, entonces el servidor también debe utilizar dicho protocolo. Si el servidor es un servicio HTTP, el cliente debe utilizar el protocolo HTTP para la conexión.

El principio detrás de esto es similar a la idea de que si quieres comunicarte con un inglés, debes usar inglés. Si quieres comunicarte con un japonés, debes usar japonés. Aquí, el idioma se asemeja al protocolo de comunicación, donde ambas partes (cliente y servidor) deben utilizar el mismo idioma para poder comunicarse, de lo contrario, no podrán hacerlo.

**Errores comunes que surgen por la falta de coherencia en el protocolo de comunicación:**

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: Unexpected response code: xxx

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: net::ERR_INVALID_HTTP_RESPONSE

**Solución:**
A partir de los dos mensajes de error anteriores, se deduce que el cliente está utilizando una conexión `ws`, que es el protocolo de WebSocket. Por lo tanto, el servidor también debe ser capaz de usar este protocolo para poder establecer la comunicación. Por ejemplo, en el caso de gatewayWorker, el código de escucha del servidor sería similar a lo siguiente:
```php
// Protocolo de WebSocket, para que los clientes puedan conectarse a través de ws://...; xxxx es el puerto que no necesita modificarse
$gateway = new Gateway('websocket://0.0.0.0:xxxx');
```
En el caso de Workerman, el código sería similar a lo siguiente:
```php
// Protocolo de WebSocket, para que los clientes puedan conectarse a través de ws://...; xxxx es el puerto que no necesita modificarse
$worker = new Worker('websocket://0.0.0.0:xxxx');
```
