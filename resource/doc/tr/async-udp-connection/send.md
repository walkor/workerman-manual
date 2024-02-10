# send metodu
```php
void AsyncUdpConnection::send(string $data)
```
Asenkron bağlantı işlemini gerçekleştirir. Bu yöntem hemen geri döner.

### Parametre
 ``` $data ```
Sunucuya gönderilecek veri, veri boyutu 65507 baytı aşmamalıdır (udp tek veri paketinin maksimum iletim boyutu 65507 bayttır), aksi takdirde gönderme başarısız olacaktır.

### Dönüş Değeri
Dönüş değeri bulunmamaktadır.

### Örnek 

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 saniye sonra udp istemcisini başlat, 1234 bağlantı noktasına bağlan ve hi dizisini gönder
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Sunucudan gelen veri hello
            echo "recv $data\r\n";
            // Bağlantıyı kapat
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // AsyncUdpConnection istemcisi tarafından gönderilen veri alındı, hello dizisini gönder
    $connection->send("hello");
};
Worker::runAll();             
```

Çalıştırıldığında benzer şekilde yazdırır:
``` 
recv hello 
```
