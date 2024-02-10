# publish
**``` (требуется Workerman версии >=3.3.0) ```**

```php
void \Channel\Client::publish(string $event_name, mixed $event_data)
```
Публикует определенное событие; все подписчики этого события получат это событие и вызовут обратный вызов ```$callback```, зарегистрированный через ```on($event_name, $callback)```.


### Параметры
 ``` $event_name ```

Имя опубликованного события, может быть любой строкой. Если у события нет ни одного подписчика, оно будет проигнорировано.

 ``` $event_data ```

Связанные с событием данные, могут быть числом, строкой или массивом.

### Возвращаемые значения
void



### Пример
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function($http_worker)
{
    Channel\Client::connect('127.0.0.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $event_name = 'user_login';
    $event_data = array('uid'=>123, 'uname'=>'tom');
    Channel\Client::publish($event_name, $event_data );
};

Worker::runAll();
```
