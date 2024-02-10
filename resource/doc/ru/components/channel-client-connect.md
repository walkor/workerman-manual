# подключение
**``` (Требуется версия Workerman >=3.3.0) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
Подключение к Channel/Server

### Параметры
 ``` listen_ip ```

IP-адрес, на который прослушивается Channel/Server. По умолчанию - ```127.0.0.1```

 ``` listen_port ```

Порт, на который прослушивает Channel/Server. По умолчанию - 2206

### Возвращаемое значение
void



### Пример
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
