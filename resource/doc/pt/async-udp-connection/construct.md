# Método __construct
```php
void AsyncUdpConnection::__construct(string $remote_address)
```
Cria um objeto de conexão UDP.

AsyncUdpConnection permite que o Workerman atue como cliente e transmita dados UDP para um servidor remoto.

## Parâmetros
Parâmetro: ``` remote_address ```

O endereço de conexão, por exemplo
``` udp://192.168.1.1:1234 ``` 
``` frame://192.168.1.1:8080 ``` 
``` text://192.168.1.1:8080 ```

## Exemplo
```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Inicia um cliente UDP após 1 segundo, conecta à porta 1234 e envia a string "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // Recebe os dados "hello" do servidor
            echo "recv $data\r\n";
            // Fecha a conexão
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
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
