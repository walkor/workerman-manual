# Propriedade de transporte

```requer (workerman >= 3.3.4)```

Define a propriedade de transporte, com os valores opcionais [tcp](https://baike.baidu.com/subview/32754/8048820.htm) e [ssl](https://baike.baidu.com/view/525499.htm), sendo tcp o valor padrão.

Quando a propriedade de transporte é definida como [ssl](https://baike.baidu.com/view/525499.htm), é necessário que o PHP tenha a extensão [openssl](https://php.net/manual/zh/book.openssl.php) instalada.

Ao utilizar o Workerman como cliente para estabelecer uma conexão criptografada SSL com o servidor (conexão HTTPS, conexão WSS, etc.), é necessário configurar esta opção como ```ssl```, conforme o exemplo abaixo.

### Exemplo (conexão HTTPS)
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Ao iniciar o processo, estabelece de forma assíncrona uma conexão com www.baidu.com e envia dados para obter informações
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // Define a conexão como criptografada com SSL
    $connection_to_baidu->transport = 'ssl';

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Conexão estabelecida com sucesso\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Conexão fechada\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Código de erro: $code mensagem: $msg\n";
    };
    $connection_to_baidu->connect();
};

// Executa o worker
Worker::runAll();
```
