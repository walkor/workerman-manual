# __construct Metodu
```php
void AsyncUdpConnection::__construct(string $remote_address)
```
Bir UDP bağlantısı nesnesi oluşturur.

AsyncUdpConnection, Workerman'ın istemci olarak uzak sunucuyla UDP veri iletimi yapmasını sağlar.

## Parametre
Parametre: `remote_address`

Bağlanılacak adres, örneğin
 ``` udp://192.168.1.1:1234 ```
 ``` frame://192.168.1.1:8080 ```
 ``` text://192.168.1.1:8080 ```

## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 saniye sonra 1234 bağlantı noktasına bir UDP istemcisi başlatılır ve "hi" dizesi gönderilir
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // Sunucudan gelen "hello" verisi alındı
            echo "recv $data\r\n";
            // Bağlantıyı kapat
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
{
    // AsyncUdpConnection istemcisinin gönderdiği veriyi al, "hello" dizisini gönder
    $connection->send("hello");
};
Worker::runAll();             
```

Çıktı:
```bash
recv hello
```
