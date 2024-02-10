# connect  Metodu
```php
void AsyncUdpConnection::connect()
```
Asenkron bağlantı işlemini gerçekleştirir. Bu yöntem derhal geri döner.

### Parametre
Parametre yok

### Dönüş Değeri
Dönüş değeri yok

### Örnek

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 saniye sonra UDP istemcisini başlat, 1234 portuna bağlan ve 'hi' dizesini gönder
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Sunucudan dönen 'hello' verisini aldı
            echo "recv $data\r\n";
            // Bağlantıyı kapat
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // AsyncUdpConnection istemcisinden gelen veriyi aldı, 'hello' dizesi döndürdü
    $connection->send("hello");
};
Worker::runAll();  
```

Çalıştırıldıktan sonra şuna benzer bir çıktı üretecek:
```  
recv hello
```
