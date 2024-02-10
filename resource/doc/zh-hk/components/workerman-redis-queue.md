# workerman/redis-queue

基於Redis的消息隊列，支援消息延遲處理。

## 項目地址：
https://github.com/walkor/redis-queue

## 安裝：
```composer require workerman/redis-queue
```

## 示例
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // 訂閱
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // 訂閱
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // 定時向隊列發送消息
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

創建實例

  * `$address`  類似 `redis://ip:6379`，必須以redis開頭. 

  * `$options`  包括以下選項:
    * `auth`: 驗證資訊，默認 ''
    * `db`: db，默認 0
    * `max_attempts`: 消費失敗後重試次數，默認5
    * `retry_seconds`: 重試時間間隔，單位秒。默認5

> 消費失敗是指業務拋出異常`Exception`或者`Error`。消費失敗後消息會放到延遲隊列等待重試，重試次數由 `max_attempts`控制，重試間隔由`retry_seconds`和`max_attempts`共同控制。比如`max_attempts`為5，`retry_seconds`為10，第1次重試間隔為`1*10`秒，第2次重試時間間隔為`2*10秒`，第3次重試時間間隔為`3*10秒`，以此類推直到重試5次。如果超過了`max_attempts`設置測重試次數，則消息放入key為`{redis-queue}-failed`(1.0.5版本之前為`redis-queue-failed`)的失敗隊列

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

向隊列發送一條消息

* `$queue` 隊列名, `String` 類型
* `$data` 發布的具體消息，可以是數組或者字符串，`Mixed` 類型
* `$dely` 延遲消費時間，單位秒，默認0, `Int` 類型
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

訂閱一個隊列或者多個隊列

* `$queue` 隊列名，可以是字符串或者包含多個隊列名的數組
* `$callback` 回調函數，格式為  `function (Mixed $data)`，其中`$data`就是`send($queue, $data)`中的`$data`.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

取消訂閱

* `$queue` 隊列名或者包含多個隊列名的數組

-------------------------------------------------------

## 在非workerman環境向隊列發送消息
有時候一些項目運行在apache或者php-fpm環境，無法使用workerman/redis-queue項目，可以參考如下函數實現發送
```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; //1.0.5版本之前為redis-queue-waiting
    $queue_delay = '{redis-queue}-delayed';//1.0.5版本之前為redis-queue-delayed
    
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
其中，參數`$redis`為redis實例。例如redis擴展用法類似如下：
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
````
