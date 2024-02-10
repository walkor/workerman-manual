# workerman目前支持的事件驅動

| 名稱  | 依賴擴展 | 支持協程 |  優先級  |  workerman版本  |
|-----|------|--|-----|
|  Workerman\Events\Select   |   無   | 不支持  |  內核默認支持   |  >=3.0  ｜
|  Workerman\Events\Revolt   |   event(可選)   | 支持 |  需安裝[revolt/event-loop](https://github.com/revoltphp/event-loop)   |  >=5.0  |
|  Workerman\Events\Event   |   event   | 不支持 |  內核默認支持   |  >=3.0  |
|  Workerman\Events\Swoole   |  [swoole](https://github.com/swoole/swoole-src)   | 支持 |  需手動設置   |  >=4.0  |
|  Workerman\Events\Swow   |   [swow](https://github.com/swow/swow)   | 支持 |  需手動設置   |  >=5.0  |

* 每種內核驅動會提供單獨的特性，例如使用`Revolt`會讓workerman支持PHP內建的[Fiber協程(纖程)](https://www.php.net/manual/zh/language.fibers.php)，使用`Swoole`則會讓workerman支持Swoole的協程
* 各個事件驅動之間是互斥關係，例如使用`Revolt`的Fiber協程時，無法使用Swoole或者Swow的協程
* `Revolt`需要安裝`composer require revolt/event-loop ^1.0.0`，安裝後workerman內核自動將其設置為首選事件驅動
* `Swoole` 和 `Swow`需要手動設置`Worker::$eventLoopClass`才能生效(參見下一段落)
* swoole默認沒有開啟[一鍵協程Runtime](https://wiki.swoole.com/#/runtime?id=runtime)，也就是說基於Pdo、Redis、PHP內建檔案讀寫仍是阻塞式調用
* 如需swoole開啟一鍵協程，需要手動調用 `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);` 

> **注意**
> swow擴展會自動更改一些PHP內建函數的行為，這會導致開啟了swow擴展但是沒有使用swow作為事件驅動時，workerman無法響應請求以及信號，所以如果您不使用swow作為底層驅動時，需要將swow從php.ini中註釋掉

更多參考[workerman協程](../fiber.md)

# 為workerman設置事件驅動

以下是為workerman手動設置事件驅動

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// 手動設置為底層事件驅動
Worker::$eventLoopClass = Workerman\Events\Revolt::class;
//Worker::$eventLoopClass = Workerman\Events\Select::class;
//Worker::$eventLoopClass = Workerman\Events\Event::class;
//Worker::$eventLoopClass = Workerman\Events\Swoole::class;
//Worker::$eventLoopClass = Workerman\Events\Swow::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```
