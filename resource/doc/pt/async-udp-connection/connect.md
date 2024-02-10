# Método connect
```php
void AsyncUdpConnection::connect()
```
Realiza a operação de conexão assíncrona. Este método retorna imediatamente.

### Parâmetros
Sem parâmetros

### Valor de Retorno
Sem valor de retorno

### Exemplo

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Inicia um cliente UDP um segundo depois, conecta à porta 1234 e envia a string "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Recebe os dados retornados pelo servidor "hello"
            echo "recv $data\r\n";
            // Fecha a conexão
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Recebe os dados enviados pelo cliente AsyncUdpConnection e retorna a string "hello"
    $connection->send("hello");
};
Worker::runAll();             
```

Após a execução, o resultado impresso será semelhante a:
``` 
recv hello
```
