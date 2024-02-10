```php
void Connection::close(mixed $data = '')
```

Güvenli bir şekilde bağlantıyı kapatır ve bağlantının `onClose` geri çağırımını tetikler.

UDP bağlantısız olsa da, karşılık gelen AsyncUdpConnection nesnesi hafızada sürekli tutulur ve ilgili udp bağlantı nesnesini serbest bırakmak için close yöntemini çağırmanız gerekir, aksi takdirde bu udp bağlantı nesnesi sürekli olarak bellekte var olacaktır ve bellek sızıntısına neden olabilir.

## Parametreler

 ``` $data ```

İsteğe bağlı parametre, gönderilecek veri (bir protokol belirtilmişse, verileri otomatik olarak kodlamak için protokolün encode yöntemi çağrılır), veri gönderildikten sonra bağlantıyı kapatır ve ardından onClose geri çağırımını tetikler.

Veri boyutu 65507 baytı aşmamalıdır, aksi takdirde gönderme başarısız olur.

### Örnek 

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 saniye sonra 1234 bağlantı noktasına bağlanarak ve 'hi' dizesini gönderen bir udp istemcisi başlat
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Sunucudan dönen 'hello' verisini al
            echo "recv $data\r\n";
            // Bağlantıyı kapat
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // AsyncUdpConnection istemcisinden gelen verileri al ve 'hello' dizesini gönder
    $connection->send("hello");
};
Worker::runAll();             
```

Çalıştırıldığında benzer bir çıktı alınır:
```php
recv hello
```
