```php
void \Channel\Client::unsubscribe(string $event_name)
```
Отменяет подписку на определенное событие. Это событие больше не будет вызывать обратный вызов ```$callback```, зарегистрированный с помощью ```on($event_name, $callback)```.

### Параметры
```$event_name```

Название события

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
    $event_name = 'user_login';
    Channel\Client::on($event_name, function($event_data){
        var_dump($event_data);
    });
    Channel\Client::unsubscribe($event_name);
};

Worker::runAll();
```
