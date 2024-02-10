# Propiedad de transporte

```Requiere (workerman >= 3.3.4)```

Establece la propiedad de transporte, con valores opcionales [tcp](https://baike.baidu.com/subview/32754/8048820.htm) y [ssl](https://baike.baidu.com/view/525499.htm), siendo tcp el valor predeterminado. 

Cuando se utiliza transport como [ssl](https://baike.baidu.com/view/525499.htm), se requiere que PHP tenga instalada la [extensión openssl](https://php.net/manual/zh/book.openssl.php).

Cuando Workerman actúa como cliente y establece una conexión SSL encriptada con el servidor (conexión https, conexión wss, etc.), por favor configure esta opción como `ssl`, como se muestra en el siguiente ejemplo.

### Ejemplo (conexión https)
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Al iniciar el proceso, se establece de forma asíncrona una conexión al objeto www.baidu.com y se envían y reciben datos.
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // Establecer la conexión encriptada ssl
    $connection_to_baidu->transport = 'ssl';

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Conexión exitosa\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Conexión cerrada\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Código de error: $code, mensaje: $msg\n";
    };
    $connection_to_baidu->connect();
};

// Ejecutar el worker
Worker::runAll();
```
