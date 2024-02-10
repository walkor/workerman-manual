```php
# connect
**``` (Workerman sürümü >=3.3.0 olmalıdır) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
Channel/Server'a bağlanmak

### Parametreler
 ``` listen_ip ```

Channel/Server'ın dinlediği IP adresi, belirtilmezse varsayılan ```127.0.0.1```'dir.

 ``` listen_port ```

Channel/Server'ın dinlediği port, belirtilmezse varsayılan 2206'dır.

### Dönüş Değeri
void

### Örnek
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
};

Worker::runAll();
```
