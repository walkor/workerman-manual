# Asenkron Görevlerin Nasıl Gerçekleştirileceği

**Soru:**

Nasıl ağır işlemleri asenkron olarak ele alabiliriz, ana işlem uzun süre engellenmeden devam edebilir? Örneğin, 1000 kullanıcıya e-posta göndermek istiyorum, bu süreç çok yavaş ve birkaç saniyeye kadar sürebilir. Bu süreçte ana işlem engellendiği için sonraki istekleri etkileyebilir. Bu tür ağır görevleri başka bir sürece nasıl asenkron olarak aktarabiliriz?

**Cevap:**

Makinede veya başka sunucularda hatta sunucu kümesinde ağır işlemleri işleyen bazı görev süreçleri önceden oluşturabilirsiniz, görev süreci sayısı, örneğin CPU'nun 10 katı olabilir, ardından çağıran taraf, verileri AsyncTcpConnection kullanarak bu görev süreçlerine asenkron olarak göndererek işleme alabilir ve asenkron olarak sonuç alabilir.

Görev süreci sunucusu
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// görev işçisi, Text protokolü kullanıyor
$task_worker = new Worker('text://0.0.0.0:12345');
// görev süreci sayısı gerektiğinde artırılabilir
$task_worker->count = 100;
$task_worker->name = 'GörevSüreci';
$task_worker->onMessage = function (TcpConnection $connection, $task_data) {
     // varsayılan olarak JSON verileri gönderildiğini farz edelim
     $task_data = json_decode($task_data, true);
     // task_data'ya göre ilgili görev mantığını işleyin.... Sonuç alın, burada atlanıyor....
     $task_result = ......
     // Sonucu gönder
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

Workerman'da çağrı yapma

```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket hizmeti
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function (TcpConnection $ws_connection, $message) {
    // Uzak görev hizmetine asenkron bağlantı kur, ip uzak görev hizmetinin ip'sidir, eğer yerel ise 127.0.0.1, küme ise lvs'nin ip'si
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // Görev ve parametre verileri
    $task_data = array(
        'function' => 'send_mail',
        'args' => array('from' => 'xxx', 'to' => 'xxx', 'contents' => 'xxx'),
    );
    // Veri gönder
    $task_connection->send(json_encode($task_data));
    // Asenkron sonuç alın
    $task_connection->onMessage = function (AsyncTcpConnection $task_connection, $task_result) use ($ws_connection) {
         // Sonuç
         var_dump($task_result);
         // Sonuç alındıktan sonra asenkron bağlantıyı kapatmayı unutmayın
         $task_connection->close();
         // İlgili websocket istemcisine görev tamamlandı bildirimi gönder
         $ws_connection->send('görev tamamlandı');
    };
    // Asenkron bağlantıyı gerçekleştir
    $task_connection->connect();
};

Worker::runAll();
```

Bu şekilde, ağır görevleri yerel makine veya başka sunucuların süreçlerine yaptırabilir ve görev tamamlandığında asenkron olarak sonuç alabilirsiniz. İşlem süreci engellenmeyecektir.
