```php
void Connection::close(mixed $data = '')
```

Fecha a conexão de forma segura e aciona a chamada de retorno `onClose` da conexão.

Embora o UDP seja sem conexão, o objeto AsyncUdpConnection correspondente ainda é mantido na memória e deve-se chamar o método close para liberar o objeto de conexão UDP correspondente. Caso contrário, o objeto de conexão UDP ficará na memória permanentemente, resultando em vazamento de memória.

## Parâmetros

``` $data ```

Parâmetro opcional que representa os dados a serem enviados (se um protocolo específico for especificado, o método encode do protocolo será automaticamente chamado para empacotar os dados de `$data`). Após o envio dos dados, a conexão será fechada, e em seguida, a chamada de retorno `onClose` será acionada.

O tamanho dos dados não pode exceder 65507 bytes, caso contrário, o envio falhará.

### Exemplo

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 segundo depois, inicia um cliente UDP, conecta-se à porta 1234 e envia a string "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Recebe os dados de retorno do servidor "hello"
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

Após a execução, imprime algo semelhante a:
```
recv hello
```
