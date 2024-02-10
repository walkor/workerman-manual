# PHP callbacklerin birkaç farklı yazımı
PHP'de anonim fonksiyonlar aracılığıyla callbackler yazmak en pratik olandır, ancak anonim fonksiyonlar dışında PHP'de diğer callback yazım şekilleri de bulunmaktadır. Aşağıda PHP'de birkaç callback yazım örneği bulunmaktadır.

## 1. Anonim fonksiyon callback
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Anonim fonksiyon callback
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // Tarayıcıya hello world gönder
    $connection->send('hello world');
};

Worker::runAll();
```

## 2. Normal fonksiyon callback
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Anonim fonksiyon callback
$http_worker->onMessage = 'on_message';

// Normal fonksiyon
function on_message(TcpConnection $connection, Request $request)
{
    // Tarayıcıya hello world gönder
    $connection->send('hello world');
}

Worker::runAll();
```

## 3. Sınıf yöntemi olarak callback
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public function __construct(){}
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
Başlatma betiği start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// MyClass'ı yükle
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Bir nesne oluştur
$my_object = new MyClass();

// Sınıfın yöntemini çağır
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

Not: Yukarıdaki kod yapısında, MySQL bağlantısı, Redis bağlantısı, Memcache bağlantısı vb. gibi kaynakları yapılandırmak için yapıcı fonksiyon içinde çalıştırılamaz, çünkü ```$my_object = new MyClass();``` ana süreçte çalışmaktadır. MySQL örneğinde olduğu gibi, ana süreçte başlatılan MySQL bağlantısı gibi kaynaklar alt süreç tarafından devralınacak ve her alt süreç bu veritabanı bağlantısını işleyebilecek, ancak bu bağlantılar MySQL sunucusunda aynı bağlantıya karşılık geleceği için öngörülemeyen hatalara neden olabilir, örneğin ```mysql gone away``` hatası.


Yukarıdaki kod yapısında, sınıfın yapıcı fonksiyonunda kaynak başlatılması gerekiyorsa, aşağıdaki yazım şekli kullanılabilir.
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // Diyelim ki veritabanı bağlantı sınıfı MyDbClass
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
Başlatma betiği start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Sınıfı onWorkerStart içinde başlat
$worker->onWorkerStart = function($worker) {
    // MyClass'ı yükle
    require_once __DIR__.'/MyClass.php';
    
    // Bir nesne oluştur
    $my_object = new MyClass();

    // Sınıfın yöntemini çağır
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

Yukarıdaki kod yapısında, onWorkerStart çalıştığında alt süreçlere ait olduğundan, her alt süreç kendi MySQL bağlantısını oluşturur, böylece paylaşılan bağlantı sorunu olmaz. Ayrıca bu şekilde iş kodu yeniden yüklenebilir. MyClass.php alt süreç tarafından yüklendiği için iş kodu değişiklikleri doğrudan yeniden yüklenerek etkili hale getirilebilir.

## 4. Sınıfın statik yöntemi olarak callback
Statik sınıf MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public static function onWorkerStart(Worker $worker){}
    public static function onConnect(TcpConnection $connection){}
    public static function onMessage(TcpConnection $connection, $message) {}
    public static function onClose(TcpConnection $connection){}
    public static function onWorkerStop(Worker $worker){}
}
```
Başlatma betiği start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// MyClass'ı yükle
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Sınıfın statik yöntemini çağır.
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// Eğer sınıfın namespace'i varsa, aşağıdaki gibi bir yazım şekli kullanılır
// $worker->onWorkerStart = array('your\namesapce\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('your\namesapce\MyClass', 'onConnect');
// $worker->onMessage     = array('your\namesapce\MyClass', 'onMessage');
// $worker->onClose       = array('your\namesapce\MyClass', 'onClose');
// $worker->onWorkerStop  = array('your\namesapce\MyClass', 'onWorkerStop');

Worker::runAll();
```

Not: PHP çalışma mantığına göre, new çağrılmadığı sürece yapılandırıcı fonksiyon çağrılmaz, ayrıca statik sınıf yöntemleri içerisinde ```$this``` kullanımına izin verilmez.
