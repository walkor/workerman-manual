# workerman/redis-queue

Message queue based on Redis that supports delayed message processing.

## Project Address:
https://github.com/walkor/redis-queue

## Installation:
``` 
composer require workerman/redis-queue
```

## Example
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // Subscribe
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // Subscribe
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // Send messages to the queue at regular intervals
    Timer::add(1, function() use ($client){
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

Create an instance

  * `$address`  similar to `redis://ip:6379`, must start with redis.

  * `$options`  includes the following options:
    * `auth`: authentication information, default is ''
    * `db`: database, default is 0
    * `max_attempts`: number of retry attempts after consumption failure, default is 5
    * `retry_seconds`: retry interval in seconds, default is 5

> Consumption failure means that the business throws an exception `Exception` or `Error`. After consumption failure, the message will be placed in the delay queue for retry. The number of retries is controlled by `max_attempts`, and the retry interval is jointly controlled by `retry_seconds` and `max_attempts`. For example, if `max_attempts` is 5 and `retry_seconds` is 10, the interval for the first retry is `1*10` seconds, the interval for the second retry is `2*10` seconds, and so on, until the fifth retry. If the number of retries exceeds the `max_attempts`, the message is placed in the failed queue with the key `{redis-queue}-failed` (before version 1.0.5, it was `redis-queue-failed`).

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

Send a message to the queue

* `$queue` queue name, type `String`
* `$data` specific message being published, can be an array or a string, type `Mixed`
* `$dely` delay in consumption time, default is 0, type `Int`
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

Subscribe to a single queue or multiple queues

* `$queue` queue name, can be a string or an array containing multiple queue names
* `$callback` callback function in the format `function (Mixed $data)`, where `$data` is the same as `$data` in `send($queue, $data)`.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

Unsubscribe from a single queue or multiple queues

* `$queue` queue name or an array containing multiple queue names

-------------------------------------------------------

## Sending messages to the queue in a non-workerman environment
Sometimes, some projects run in an Apache or PHP-FPM environment and cannot use the workerman/redis-queue project. You can refer to the following function to implement sending:
```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; //Before version 1.0.5, it was redis-queue-waiting
    $queue_delay = '{redis-queue}-delayed'; //Before version 1.0.5, it was redis-queue-delayed
    
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
Where the parameter `$redis` is the redis instance. For example, the usage of the redis extension is similar to the following:
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
````
