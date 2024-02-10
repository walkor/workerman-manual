# workerman/redis-queue

Очередь сообщений, основанная на Redis, поддерживающая отложенную обработку сообщений.

## Ссылка на проект:
https://github.com/walkor/redis-queue

## Установка:
```composer require workerman/redis-queue```

## Пример
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // Подписаться
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // Подписаться
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // Регулярно отправлять сообщения в очередь
    Timer::add(1, function()use($client){
        $client->send('user-1', ['some', 'data']);
    });
};

Worker::runAll();
```

## API
  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#send"><code>Client::<b>send()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>

-------------------------------------------------------

<a name="construct"></a>
### __construct (string $address, [array $options])

Создание экземпляра

  * `$address`  похоже на `redis://ip:6379`, должно начинаться с redis.

  * `$options`  включает следующие опции:
    * `auth`: информация для аутентификации, по умолчанию ''
    * `db`: база данных, по умолчанию 0
    * `max_attempts`: количество попыток повторной обработки при ошибке, по умолчанию 5
    * `retry_seconds`: интервал повторной обработки, в секундах, по умолчанию 5

> Ошибка обработки означает возникновение исключения `Exception` или `Error` в процессе обработки. После ошибки обработки сообщение будет помещено в отложенную очередь и ожидать повторной обработки. Количество попыток повторной обработки управляется параметром `max_attempts`, а интервал повторной обработки управляется параметрами `retry_seconds` и `max_attempts`. Например, если `max_attempts` равно 5, а `retry_seconds` равно 10, то интервал повторной обработки для первой попытки составит `1*10` секунд, для второй попытки - `2*10` секунд, для третьей попытки - `3*10` секунд и так далее до пятой попытки. Если количество попыток превысит установленное значение `max_attempts`, сообщение будет помещено в очередь необработанных сообщений с ключом `{redis-queue}-failed` (до версии 1.0.5 - `redis-queue-failed`).

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

Отправка сообщения в очередь

* `$queue` название очереди, тип `String`
* `$data` конкретное отправляемое сообщение, может быть массивом или строкой, тип `Mixed`
* `$dely` время задержки обработки, в секундах, по умолчанию 0, тип `Int`
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

Подписка на одну или несколько очередей

* `$queue` название очереди, может быть строкой или массивом, содержащим несколько названий очередей
* `$callback` функция обратного вызова в формате `function (Mixed $data)`, где `$data` - это `$data` из `send($queue, $data)`.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

Отмена подписки

* `$queue` название очереди или массив, содержащий несколько названий очередей

-------------------------------------------------------

## Отправка сообщений в очередь вне среды workerman
Иногда некоторые проекты работают в среде Apache или PHP-FPM и не могут использовать проект workerman/redis-queue. В этом случае можно использовать следующую функцию для отправки сообщений:
```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; //до версии 1.0.5 - redis-queue-waiting
    $queue_delay = '{redis-queue}-delayed';//до версии 1.0.5 - redis-queue-delayed
    
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```
Где параметр `$redis` - это экземпляр Redis. Например, использование расширения Redis может выглядеть как показано ниже:
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```
