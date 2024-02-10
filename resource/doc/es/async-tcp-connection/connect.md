# Método connect
```php
void AsyncTcpConnection::connect()
```
Realiza una operación de conexión asincrónica. Este método devuelve de inmediato.

Nota: Si es necesario configurar el callback onError para la conexión asincrónica, debe hacerse antes de llamar a connect. De lo contrario, es posible que el callback onError no se active. Por ejemplo, en el siguiente caso, el callback onError puede no activarse y no capturar el evento de falla en la conexión asincrónica.

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// El onError callback no ha sido configurado antes de realizar la conexión
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
``` 

### Parámetros
Sin parámetros

### Valor de retorno
Sin valor de retorno

### Ejemplo de proxy Mysql

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Dirección real de mysql, asumiendo que está en el puerto 3306 de esta máquina
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// El servidor proxy escucha en el puerto 4406 del localhost
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // Establece una conexión asincrónica con el servidor mysql real
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // Cuando la conexión mysql envía un mensaje, reenviarlo a la conexión del cliente correspondiente
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer)use($connection)
    {
        $connection->send($buffer);
    };
    // Cuando la conexión mysql se cierra, cierra la conexión proxy correspondiente al cliente
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // Cuando hay un error en la conexión mysql, cierra la conexión proxy correspondiente al cliente
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // Realiza la conexión asincrónica
    $connection_to_mysql->connect();

    // Cuando el cliente envía un mensaje, reenviarlo a la conexión mysql correspondiente
    $connection->onMessage = function(TcpConnection $connection, $buffer)use($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // Cuando se cierra la conexión del cliente, cierra la conexión mysql correspondiente
    $connection->onClose = function(TcpConnection $connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // Cuando hay un error en la conexión del cliente, cierra la conexión mysql correspondiente
    $connection->onError = function(TcpConnection $connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

};
// Ejecutar el worker
Worker::runAll();
```

**Prueba**

```sh
mysql -uroot -P4406 -h127.0.0.1 -p
``` 

Bienvenido al monitor de MySQL. Los comandos terminan con ';' o '\g'.
El ID de conexión de MySQL es 25004
Versión del servidor: 5.5.31-1~dotdeb.0 (Debian)

Copyright (c) 2000, 2013, Oracle y/o sus filiales. Todos los derechos reservados.

Oracle es una marca registrada de Oracle Corporation y/o sus filiales.
Otros nombres pueden ser marcas comerciales de sus respectivos propietarios.

Escriba 'help;' o '\h' para obtener ayuda. Escriba '\c' para borrar la declaración de entrada actual.

mysql>
```
