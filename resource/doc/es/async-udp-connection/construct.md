# Método __construct
```php
void AsyncUdpConnection::__construct(string $remote_address)
```
Crea un objeto de conexión UDP.

AsyncUdpConnection permite que Workerman actúe como cliente para transferir datos UDP a un servidor remoto.

## Parámetros
Parámetro: `remote_address`

La dirección de conexión, por ejemplo
 `udp://192.168.1.1:1234`
 `frame://192.168.1.1:8080`
 `text://192.168.1.1:8080`

## Ejemplo
```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 segundo después, inicia un cliente UDP, se conecta al puerto 1234 y envía la cadena "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // Recibe los datos de retorno del servidor "hello"
            echo "recv $data\r\n";
            // Cierra la conexión
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
{
    // Recibe los datos enviados por el cliente AsyncUdpConnection, devuelve la cadena "hello"
    $connection->send("hello");
};
Worker::runAll();    
```
Al ejecutar, imprime algo similar a:
```
recv hello
```
