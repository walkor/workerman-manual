# runAll
```php
void Worker::runAll(void)
```
Запускает все экземпляры Worker.

**Примечание:**

После выполнения Worker::runAll() произойдет постоянная блокировка, что означает, что код, находящийся после Worker::runAll(), не будет выполнен. Все экземпляры Worker должны быть инициализированы до Worker::runAll().

### Параметры
Без параметров

### Возвращаемое значение
Без возвращаемого значения

## Пример запуска нескольких экземпляров Worker

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// Запуск всех экземпляров Worker
Worker::runAll();
```

**Примечание:**

Версия workerman для Windows не поддерживает инициализацию нескольких экземпляров Worker в одном файле. 

Приведенный выше пример не будет работать в версии workerman для Windows.

Для версии workerman для Windows необходимо инициализировать несколько экземпляров Worker в разных файлах, как показано ниже.

start_http.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

// Запуск всех экземпляров Worker (здесь только один экземпляр)
Worker::runAll();
```

start_websocket.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// Запуск всех экземпляров Worker (здесь только один экземпляр)
Worker::runAll();
```
