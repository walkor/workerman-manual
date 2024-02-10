# reConnect Metodu
```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

``` (Workerman sürümü >=3.3.5 olmalıdır) ```

Yeniden bağlanma işlemini gerçekleştirir. Genellikle ```onClose``` geri çağırma işlevi içinde çağrılır ve bağlantının yeniden kurulmasını sağlar.

Ağ sorunları veya karşı tarafın hizmet yeniden başlatması gibi nedenlerle bağlantı koptuğunda, bu yöntemi çağırarak yeniden bağlantı sağlanabilir.

### Parametreler
``` $delay ```

Yeniden bağlanma işleminin ne kadar gecikmeli gerçekleştirileceği. Saniye cinsinden birim, ondalık sayıları ve milisaniyeye kadar hassas olabilir.

Eğer belirtilmezse veya değeri 0 ise hemen yeniden bağlantı sağlanır.

Yeniden bağlantının gecikmeli gerçekleşmesini sağlamak için parametre iletimi tercih edilir, aksi takdirde karşı taraftaki servis sorunları nedeniyle sürekli bağlantı olmaması durumunda yerel CPU'nun aşırı tüketilmesini önler.

### Dönüş Değeri
Dönüş değeri bulunmamaktadır.

### Örnekler

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // Bağlantı koptuysa, 1 saniye sonra yeniden bağlan
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **Not**
> Yeniden bağlantı işlemi başarılı olduğunda, $con'un onConnect yöntemi tekrar çağrılacaktır (eğer ayarlanmışsa). Bazen onConnect yönteminin sadece bir kez çalışmasını ve yeniden bağlanma işleminde tekrar çalışmamasını isteyebiliriz. Aşağıdaki örneğe bakınız:

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        static $is_first_connect = true;
        if (!$is_first_connect) return;
        $is_first_connect = false;
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // Bağlantı koptuysa, 1 saniye sonra yeniden bağlan
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```
