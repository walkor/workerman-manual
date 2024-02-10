# 簡單的開發實例

## 安裝

**安裝workerman**
在一個空目錄中運行
`composer require workerman/workerman`

## 實例一、使用HTTP協議對外提供Web服務
**創建start.php文件**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// 創建一個Worker監聽2345端口，使用http協議通訊
$http_worker = new Worker("http://0.0.0.0:2345");

// 啟動4個進程對外提供服務
$http_worker->count = 4;

// 接收到瀏覽器發送的數據時回復hello world給瀏覽器
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // 向瀏覽器發送hello world
    $connection->send('hello world');
};

// 運行worker
Worker::runAll();
```

**命令行運行（windows用戶用 [cmd命令行](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn)，下同）**
```shell
php start.php start

```

**測試**

假設服務端ip為127.0.0.1

在瀏覽器中訪問url http://127.0.0.1:2345

 **注意：**

1、如果出現無法訪問的情況，請參照[客戶端連接失敗原因](../faq/client-connect-fail.md)一節排查。

2、服務端是http協議，只能用http協議通訊，用websocket等其他協議無法直接通訊。


## 實例二、使用WebSocket協議對外提供服務
**創建ws_test.php文件**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 注意：這裡與上個例子不同，使用的是websocket協議
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// 啟動4個進程對外提供服務
$ws_worker->count = 4;

// 當收到客戶端發來的數據後返回hello $data給客戶端
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // 向客戶端發送hello $data
    $connection->send('hello ' . $data);
};

// 運行worker
Worker::runAll();
```

**命令行運行**
```shell
php ws_test.php start

```

**測試**

打開chrome瀏覽器，按F12打開調試控制台，在Console一欄輸入(或者把下面代碼放入到html頁面用js運行)

```javascript
// 假設服務端ip為127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("連接成功");
    ws.send('tom');
    alert("給服務端發送一個字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服務端的消息：" + e.data);
};
```

  **注意：**

1、如果出現無法訪問的情況，請參照[手冊常見問題-連接失敗](../faq/client-connect-fail.md)一節排查。

2、服務端是websocket協議，只能用websocket協議通訊，用http等其他協議無法直接通訊。

## 實例三、直接使用TCP傳輸數據
**創建tcp_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 創建一個Worker監聽2347端口，不使用任何應用層協議
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// 啟動4個進程對外提供服務
$tcp_worker->count = 4;

// 當客戶端發來數據時
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // 向客戶端發送hello $data
    $connection->send('hello ' . $data);
};

// 運行worker
Worker::runAll();
```

**命令行運行**

```shell
php tcp_test.php start

```

**測試：命令行運行**
(以下是linux命令行效果，與windows下效果有所不同)
```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```

**注意：**

1、如果出現無法訪問的情況，請參照[手冊常見問題-連接失敗](../faq/client-connect-fail.md)一節排查。

2、服務端是裸tcp協議，用websocket、http等其他協議無法直接通訊。
