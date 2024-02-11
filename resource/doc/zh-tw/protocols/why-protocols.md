# 通訊協議的作用
由於TCP是基於流的，客戶端發送的請求數據就像水流一樣流入到服務端，服務端探測到有數據到來後應該檢查數據是否是完整的，因為可能只是一個請求的部分數據到達服務端，甚至可能是多個請求連在一起到達服務端。如何判斷請求是否全部到達或者從多個連在一起的請求中分離請求，就需要規定一套通訊協議。

## 在Workerman中為什麼要制定協議？

傳統PHP開發都是基於Web的，基本上都是HTTP協議，HTTP協議的解析處理都由WebServer獨自承擔了，所以開發者不會關心協議方面的事情。然而當我們需要基於非HTTP協議開發時，開發者就需要考慮協議的事情了。

## Workerman已經支持的協議
Workerman目前已經支持HTTP、websocket、text協議(見附錄說明)、frame協議(見附錄說明)，ws協議(見附錄說明)，需要基於這些協議通訊時可以直接使用，使用方法為：在初始化Worker時指定協議，例如
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket://0.0.0.0:2345 表明用websocket協議監聽2345端口
$websocket_worker = new Worker('websocket://0.0.0.0:2345');

// text協議
$text_worker = new Worker('text://0.0.0.0:2346');

// frame協議
$frame_worker = new Worker('frame://0.0.0.0:2347');

// tcp Worker，直接基於socket傳輸，不使用任何應用層協議
$tcp_worker = new Worker('tcp://0.0.0.0:2348');

// udp Worker，不使用任何應用層協議
$udp_worker = new Worker('udp://0.0.0.0:2349');

// unix domain Worker，不使用任何應用層協議
$unix_worker = new Worker('unix:///tmp/wm.sock');
```

## 使用自定義的通訊協議
當Workerman自帶的通訊協議滿足不了開發需求時，開發者可以定制自己的通訊協議，定制方法見下一節內容。

**提示：**

Workerman內置了一個text協議，協議格式為文本+換行符。text協議開發調試都非常簡單，可用於絕大多數自定義協議的場景，並且支持telnet調試。如果開發者要開發自己的應用協議，可以直接使用text協議，不用再單獨開發。

text協議說明參考《附錄 Text協議部分》
