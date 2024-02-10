# Método send
```php
void AsyncUdpConnection::send(string $data)
```
Realiza la operación de conexión asincrónica. Este método devuelve inmediatamente.

### Parámetros
``` $data ```
Los datos que se enviarán al servidor. El tamaño de los datos no puede exceder los 65507 bytes (el tamaño máximo de transmisión de un paquete UDP es de 65507 bytes), de lo contrario el envío fracasará.

### Valor de retorno
Sin valor de retorno

### Ejemplo

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 segundo después, inicia un cliente UDP, se conecta al puerto 1234 y envía la cadena "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Recibe datos devueltos por el servidor "hello"
            echo "recv $data\r\n";
            // Cierra la conexión
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Recibe datos enviados por el cliente AsyncUdpConnection, devuelve la cadena "hello"
    $connection->send("hello");
};
Worker::runAll();             
```

Después de ejecutar, imprime algo similar a:
````
recv hello
```
