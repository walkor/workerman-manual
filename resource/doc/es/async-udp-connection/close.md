```php
void Connection::close(mixed $data = '')
```

Cerrar de forma segura la conexión y desencadenar la devolución de llamada ```onClose``` de la conexión.

Aunque UDP es sin conexión, el objeto AsyncUdpConnection correspondiente se mantiene en memoria y debe llamarse al método close para liberar el objeto de conexión UDP correspondiente, de lo contrario, este objeto de conexión UDP seguirá existiendo en la memoria, causando una fuga de memoria.

## Parámetros

 ``` $data ```

Parámetro opcional, los datos a enviar (si se especifica un protocolo, se llamará automáticamente al método encode del protocolo para empaquetar los datos de ```$data```), cuando se envían los datos, se cierra la conexión y luego se desencadena la devolución de llamada onClose.

El tamaño de los datos no puede superar los 65507 bytes, de lo contrario el envío fallará.

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
            // Recibe los datos devueltos por el servidor "hello"
            echo "recv $data\r\n";
            // Cierra la conexión
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Recibe los datos enviados por el cliente AsyncUdpConnection y devuelve la cadena "hello"
    $connection->send("hello");
};
Worker::runAll();            
```

Después de ejecutar, imprime algo similar a:
```php
recv hello
```
