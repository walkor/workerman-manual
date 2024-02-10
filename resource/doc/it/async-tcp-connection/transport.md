# Proprietà di trasporto

```Richiede (workerman >= 3.3.4)```

Imposta la proprietà di trasporto, i valori opzionali sono [tcp](https://baike.baidu.com/subview/32754/8048820.htm) e [ssl](https://baike.baidu.com/view/525499.htm), il default è tcp.

Quando il trasporto è impostato su [ssl](https://baike.baidu.com/view/525499.htm), è necessario che PHP abbia l'estensione [openssl](https://php.net/manual/zh/book.openssl.php) installata.

Quando si utilizza Workerman come client per iniziare una connessione crittografata SSL al server (connessione https, connessione wss, ecc.), si prega di impostare questa opzione su ```ssl```, come nell'esempio seguente.

### Esempio (connessione https)

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Durante l'avvio del processo, stabilisce in modo asincrono una connessione a www.baidu.com e invia dati per ricevere dati
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // Imposta la connessione come crittografata SSL
    $connection_to_baidu->transport = 'ssl';

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Connessione riuscita\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnessione: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Connessione chiusa\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Codice di errore: $code, Messaggio: $msg\n";
    };
    $connection_to_baidu->connect();
};

// Esecuzione del worker
Worker::runAll();
```
