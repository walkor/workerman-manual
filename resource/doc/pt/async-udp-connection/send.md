# Método send
```php
void AsyncUdpConnection::send(string $data)
```
Realiza a operação de conexão assíncrona. Este método retorna imediatamente.

### Parâmetros
```$data```
Os dados a serem enviados para o servidor. O tamanho dos dados não pode exceder 65507 bytes (o tamanho máximo de transmissão de um pacote UDP é de 65507 bytes), caso contrário o envio falhará.

### Valor de retorno
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
    // Inicia um cliente UDP um segundo depois, conecta-se à porta 1234 e envia a string "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Recebe os dados "hello" enviados pelo servidor
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

Após a execução, será impresso algo semelhante a:
``` 
recv hello
```
