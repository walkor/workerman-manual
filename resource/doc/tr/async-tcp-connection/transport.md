# transport özelliği
`Gereksinim (workerman >= 3.3.4)`

Taşıma özelliğini ayarlayın, isteğe bağlı değerler [tcp](https://baike.baidu.com/subview/32754/8048820.htm) ve [ssl](https://baike.baidu.com/view/525499.htm) olabilir, varsayılan tcp'dir.

transport [ssl](https://baike.baidu.com/view/525499.htm) olduğunda, PHP'de [openssl eklentisinin](https://php.net/manual/zh/book.openssl.php) kurulu olması gerekir.

Workerman'ı bir istemci olarak kullanarak sunucuya ssl şifreli bağlantı (https bağlantısı, wss bağlantısı vb.) başlattığınızda, bu seçeneği `ssl` olarak ayarlamanız gerekir, örneğin aşağıdaki örnek.

### Örnek (https bağlantısı)
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// İşlem başlatıldığında www.baidu.com'a karşı asenkron olarak bir bağlantı oluşturur ve veri gönderir
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // ssl şifreli bağlantı olarak ayarla
    $connection_to_baidu->transport = 'ssl';

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "bağlantı başarılı\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "bağlantı kapatıldı\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Hata kodu:$code mesaj:$msg\n";
    };
    $connection_to_baidu->connect();
};

// Worker'ı çalıştır
Worker::runAll();
```
