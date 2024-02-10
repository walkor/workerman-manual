# La propriété de transport

```Requis (workerman >= 3.3.4)```

Définit la propriété de transport, les valeurs possibles sont [tcp](https://baike.baidu.com/subview/32754/8048820.htm) et [ssl](https://baike.baidu.com/view/525499.htm), par défaut c'est tcp.

Lorsque le transport est défini comme [ssl](https://baike.baidu.com/view/525499.htm), PHP doit avoir l'extension [openssl](https://php.net/manual/zh/book.openssl.php) installée.

Lorsque Workerman est utilisé comme client pour établir une connexion chiffrée SSL avec le serveur (connexion https, wss, etc.), veuillez définir cette option sur ```ssl```, comme dans l'exemple ci-dessous.

### Exemple (connexion https)

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Lorsque le processus démarre, établir de manière asynchrone une connexion avec www.baidu.com et envoyer des données pour obtenir des données
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // Définir la connexion comme étant chiffrée SSL
    $connection_to_baidu->transport = 'ssl';

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Connexion réussie\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Connexion fermée\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Code d'erreur : $code, message : $msg\n";
    };
    $connection_to_baidu->connect();
};

// Exécuter le worker
Worker::runAll();
```
