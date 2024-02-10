# workerman/redis-queue

Redis tabanlı bir mesaj kuyruğu, mesaj gecikmesini destekler.

## Proje adresi:
https://github.com/walkor/redis-queue

## Kurulum:
```composer require workerman/redis-queue```

## Örnek
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
    // Schedule to send messages to the queue
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
### __construct(string $address, [array $options])

Örnek oluşturur

  * `$address`  `redis://ip:6379` gibi, redis ile başlamalıdır.
  
  * `$options`  Aşağıdaki seçenekleri içerir:
    * `auth`: Yetkilendirme bilgisi, varsayılan ''
    * `db`: db, varsayılan 0
    * `max_attempts`: Tüketim başarısız olduğunda yeniden dene sonra tekrarlama sayısı, varsayılan 5
    * `retry_seconds`: Tekrarlama zaman aralığı, saniye cinsinden. Varsayılan 5

> Başarısız tüketim, iş istisna `Exception` veya `Error` fırlattığında oluşur. Başarısız tüketim sonrasında mesaj, yeniden deneme için gecikmeli kuyruğa konur. Tekrarlama sayısı `max_attempts` tarafından kontrol edilir, tekrarlama aralığı `retry_seconds` ve `max_attempts` tarafından birlikte kontrol edilir. Örneğin `max_attempts` değeri 5 ise, `retry_seconds` değeri 10 ise, 1. tekrarlamanın aralığı `1*10` saniye, 2. tekrarlamanın aralığı `2*10` saniye, 3. tekrarlamanın aralığı `3*10` saniye şeklindedir ve tüm bunlar 5 kez tekrarlanır. Eğer `max_attempts` ayarlanan tekrarlama sayısını aşarsa, mesaj `redis-queue}-failed` (1.0.5 sürümünden önce `redis-queue-failed`) adındaki başarısız kuyruğa konur.

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

Kuyruğa bir mesaj gönderir

* `$queue` Kuyruk adı, `String` türü
* `$data` Yayınlanan özel mesaj, dizi veya dize olabilir, `Mixed` türü
* `$dely` Gecikmeli tüketim zamanı, saniye cinsinden varsayılan 0, `Int` türü

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

Bir kuyruğa veya birden fazla kuyruğa abone olur

* `$queue` Kuyruk adı, dize veya birden fazla kuyruk adını içeren dizi olabilir
* `$callback` Geri çağrı fonksiyonu, formatı `function (Mixed $data)` şeklindedir, burada `$data`, `send($queue, $data)` içindeki `$data`dır.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

Aboneliği iptal eder

* `$queue` Kuyruk adı veya birden fazla kuyruk adını içeren dizi

-------------------------------------------------------

## Workerman dışı bir ortamda kuyruğa mesaj gönderme
Bazı projeler bazı projeler apache veya php-fpm ortamında çalışabilir ve workerman/redis-queue projesini kullanamayabilir. Aşağıdaki işlevi kullanarak bunu başarabilirsiniz.
```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; //1.0.5 sürümünden önce redis-queue-waiting
    $queue_delay = '{redis-queue}-delayed';//1.0.5 sürümünden önce redis-queue-delayed
    
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
Burada, `$redis` değişkeni redis örneğidir. Örneğin redis uzantısının kullanımı şu şekildedir:
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```
