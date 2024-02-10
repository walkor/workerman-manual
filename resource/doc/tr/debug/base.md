# Temel Hata Ayıklama

WorkerMan'ın iki çalışma modu vardır: hata ayıklama modu ve daemon çalışma modu.

```php start.php start``` komutunu çalıştırarak hata ayıklama moduna girebilirsiniz, bu durumda ```echo, var_dump, var_export``` gibi fonksiyonların çıktıları terminalde görüntülenecektir. ```php start.php start``` ile çalışan WorkerMan'in tüm süreçleri terminal kapatıldığında sona ereceğini unutmayın.

```php start.php start -d``` komutu ise daemon moduna girmenizi sağlar, yani canlı ortamda kullanılan çalışma modudur ve terminal kapatıldığında etkilenmez. 

Eğer daemon modunda da ```echo, var_dump, var_export``` gibi fonksiyonların çıktılarını görmek istiyorsanız, Worker::$stdoutFile özelliğini ayarlayabilirsiniz, örneğin:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Ekran çıktısını Worker::$stdoutFile tarafından belirtilen dosyaya yönlendirin
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```
Bu şekilde tüm ```echo, var_dump, var_export``` gibi fonksiyonların çıktıları ```Worker::$stdoutFile``` tarafından belirtilen dosyaya yazılacaktır. ```Worker::$stdoutFile``` tarafından belirtilen yolun yazma iznine sahip olmasına dikkat edin.
