# GlobalData Component Client
**``` (Workerman versiyonu >=3.3.0 gerektirir) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

\GlobalData\Client istemci nesnesini örnekler. İstemci nesnesi üzerinde özellik atanarak süreçler arası veri paylaşımı yapılır.

### Parametre
GlobalData sunucu sunucu adresi, formatı```<ip adresi>:<port>```, örneğin```127.0.0.1:2207```.

GlobalData sunucusu kümesi ise adres dizisi aktarın, örneğin```array('10.0.0.10:2207', '10.0.0.0.11:2207')```

## Açıklama
Atama, okuma, isset, unset işlemlerini destekler.
Aynı zamanda cas atomik işlemi destekler.

## Örnek

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// GlobalData Sunucu
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// İşçi başladığında
$worker->onWorkerStart = function()
{
    // Global bir global veri istemcisi başlat
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// Her bir hizmet sunucusu mesaj aldığında
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // $global->somedata değerini değiştir, diğer süreçler bu $global->somedata değişkenini paylaşır
    global $global;
    echo "şimdi global->somedata=".var_export($global->somedata, true)."\n";
    echo "\$global->somedata değeri ayarla=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### Tüm kullanımlar (php-fpm ortamında da kullanılabilir)
```php
require_once __DIR__ . '/vendor/autoload.php';

$global = new Client('127.0.0.1:2207');

var_export(isset($global->abc));

$global->abc = array(1,2,3);

var_export($global->abc);

unset($global->abc);

var_export($global->add('abc', 10));

var_export($global->increment('abc', 2));

var_export($global->cas('abc', 12, 18));

```

## Not:
GlobalData bileşeni kaynak türündeki verileri paylaşamaz, örneğin mysql bağlantısı, soket bağlantısı gibi veriler paylaşılamaz.

GlobalData/Client Workerman ortamında kullanılıyorsa, onXXX geri çağrılarında GlobalData/Client nesnesi örneklendirilmelidir, örneğin onWorkerStart içerisinde örneklendirilmelidir.

Değişken paylaşımı bu şekilde yapılamaz.
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
Bu şekilde yapılabilir
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
