# Fiber協程
從5.0.0版開始，workerman支援[Fiber協程(纖程)](https://www.php.net/manual/zh/language.fibers.php)

> **注意**
> Fiber特性需要PHP>=8.1並安裝 `composer require revolt/event-loop ^1.0.0`

### 介紹

Fiber是php內建的協程(纖程)，它可以中斷PHP程式碼，然後在需要的時候恢復其運行。它的最大作用是讓開發者可以用同步的方式寫非同步非阻塞程式碼，這極大地增強了程式碼的可維護性。

### 範例
下面透過範例來對比協程和異步回調程式設計的區別。
假設我們有一個需求要求需要調用一個HTTP接口，然後延遲一秒響應，異步回調及協程寫法分別如下。

**異步回調法**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // 請求HTTP接口
    $http->get('http://example.com/', function ($response) use ($connection) {
        // 延遲一秒發送
        Timer::add(1, function() use ($connection, $response) {
            // 向瀏覽器發送數據
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**協程用法**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // 調用HTTP接口
    $response = $http->get('http://example.com/');
    // 延遲1秒
    Timer::sleep(1);
    // 發送數據
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **注意**
> 以上程式碼需要安裝 composer require workerman/http-client ^2.0.0

這兩種寫法都是異步非阻塞執行的，運行效率都很好，但是協程用法比異步回調更容易閱讀，更容易維護。


### Fiber注意事項
* Fiber協程並不支援將Pdo、Redis、及PHP內部阻塞函數協程化，也就是說使用這些擴展及函數仍然是阻塞式呼叫
* 目前可用的Fiber協程客戶端有 [workerman/http-client](../components/workerman-http-client.md)，[workerman/redis](../components/workerman-redis.md)

# Swoole協程
workerman v5 同時支援將Swoole作為底層事件驅動


```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// 這裡需要手動將Swoole設置為底層事件驅動
Worker::$eventLoopClass = Workerman\Events\Swoole::class;
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
**提示**
* 建議使用 swoole5.0 或者後續更高版本
* 把swoole作為底層事件驅動可以讓workerman支持swoole的協程
* 使用swoole作為底層事件驅動時可以不安裝event擴展
* swoole默認沒有開啟一鍵協程，也就是說基於Pdo、Redis、PHP內建檔案讀寫是阻塞式呼叫
* 如需開啟一鍵協程，需要手動調用 `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`

更多請參考[Swoole手冊](https://wiki.swoole.com/)

更多請參考[事件驅動](appendices/event.md)

# 關於協程
首先沒必過份迷信協程，當資料庫、Redis等存儲都在內網時，多進程阻塞式呼叫很多時候比協程更快。從[techempower.com 3年來的壓測數據](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db)來看workerman阻塞式資料庫呼叫性能要優於swoole資料庫連接池+協程，甚至比go語言的gin、echo等協程框架性能高近1倍。

workerman已經將PHP應用的性能提升了數倍甚至數十倍，絕大部分workerman項目加上協程可能不會有更大的性能提升。
如果你的系統裡有慢呼叫，例如有外部HTTP呼叫，可以考慮使用協程來提升性能。
