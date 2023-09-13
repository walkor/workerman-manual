# workerman/redis-queue

基于Redis的消息队列，支持消息延迟处理。

## 项目地址：
https://github.com/walkor/redis-queue

## 安装：
```
composer require workerman/redis-queue
```

## 示例
```
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // 订阅
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // 订阅
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // 定时向队列发送消息
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

创建实例

  * `$address`  类似 `redis://ip:6379`，必须以redis开头. 

  * `$options`  包括以下选项:
    * `auth`: 鉴权信息，默认 ''
    * `db`: db，默认 0
    * `max_attempts`: 消费失败后重试次数，默认5
    * `retry_seconds`: 重试时间间隔，单位秒。默认5

> 消费失败是指业务抛出异常`Exception`或者`Error`。消费失败后消息会放到延迟队列等待重试，重试次数由 `max_attempts`控制，重试间隔由`retry_seconds`和`max_attempts`共同控制。比如`max_attempts`为5，`retry_seconds`为10，第1次重试间隔为`1*10`秒，第2次重试时间间隔为`2*10秒`，第3次重试时间间隔为`3*10秒`，以此类推直到重试5次。如果超过了`max_attempts`设置测重试次数，则消息放入key为`{redis-queue}-failed`(1.0.5版本之前为`redis-queue-failed`)的失败队列

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

向队列发送一条消息

* `$queue` 队列名, `String` 类型
* `$data` 发布的具体消息，可以是数组或者字符串，`Mixed` 类型
* `$dely` 延迟消费时间，单位秒，默认0, `Int` 类型
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

订阅一个队列或者多个队列

* `$queue` 队列名，可以是字符串或者包含多个队列名的数组
* `$callback` 回调函数，格式为  `function (Mixed $data)`，其中`$data`就是`send($queue, $data)`中的`$data`.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

取消订阅

* `$queue` 队列名或者包含多个队列名的数组

-------------------------------------------------------

## 在非workerman环境向队列发送消息
有时候一些项目运行在apache或者php-fpm环境，无法使用workerman/redis-queue项目，可以参考如下函数实现发送
```
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; //1.0.5版本之前为redis-queue-waiting
    $queue_delay = '{redis-queue}-delayed';//1.0.5版本之前为redis-queue-delayed
    
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
其中，参数`$redis`为redis实例。例如redis扩展用法类似如下：
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
````
