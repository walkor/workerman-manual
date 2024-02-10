# workerman/redis-queue

Redisをベースとしたメッセージキューで、メッセージの遅延処理をサポートしています。

## プロジェクトのアドレス：
https://github.com/walkor/redis-queue

## インストール：
```composer require workerman/redis-queue```

## 例
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // 購読
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // 購読
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // キューに定期的にメッセージを送信
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

インスタンスを作成します。

  * `$address`  `redis://ip:6379`のような形式で、redisで始まる必要があります。 

  * `$options`  以下のオプションを含みます:
    * `auth`: 認証情報、デフォルトは''
    * `db`: db、デフォルトは0
    * `max_attempts`: 消費失敗後の再試行回数、デフォルトは5
    * `retry_seconds`: 再試行間隔、秒単位。デフォルトは5

> 消費失敗とは、ビジネスが例外`Exception`または`Error`をスローしたことを指します。消費失敗後、メッセージは再試行するための遅延キューに配置され、再試行回数は`max_attempts`で制御され、再試行間隔は`retry_seconds`と`max_attempts`によって共同で制御されます。 たとえば、`max_attempts`が5で`retry_seconds`が10の場合、1回目の再試行間隔は`1*10`秒、2回目の再試行間隔は`2*10秒`、3回目の再試行間隔は`3*10秒`、以降も同様に5回まで続きます。`max_attempts`で設定された再試行回数を超えると、メッセージは`{redis-queue}-failed`(バージョン1.0.5以前は`redis-queue-failed`)という失敗キューに配置されます。

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

キューにメッセージを送信します。

* `$queue` キュー名、`String`タイプ
* `$data` 送信される具体的なメッセージ、配列または文字列であり、`Mixed`タイプ
* `$dely` 遅延消費時間、デフォルトは0、`Int`タイプ
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

キューまたは複数のキューを購読します。

* `$queue` キュー名、文字列または複数のキュー名を含む配列が可能です
* `$callback` コールバック関数、形式は `function (Mixed $data)`で、`$data`が`send($queue, $data)`の`$data`に対応します。

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

購読をキャンセルします。

* `$queue` キュー名または複数のキュー名を含む配列

-------------------------------------------------------

## workerman環境以外でキューにメッセージを送信する
いくつかのプロジェクトは、apacheやphp-fpm環境で実行されており、workerman/redis-queueプロジェクトを使用できない場合があります。次の関数を参考にして実装できます。
```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting';//バージョン1.0.5以前はredis-queue-waiting
    $queue_delay = '{redis-queue}-delayed';//バージョン1.0.5以前はredis-queue-delayed
    
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
ここで、パラメータ`$redis`はredisのインスタンスです。たとえば、redisの拡張機能の使用方法は以下のようになります:
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```
