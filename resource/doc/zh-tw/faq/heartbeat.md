# 心跳

注意：長連結應用必須加心跳，否則連結可能由於長時間未通訊被路由節點強行斷開。

心跳作用主要有兩個：

1、客戶端定時給服務端發送點數據，防止連結由於長時間沒有通訊而被某些節點的防火牆關閉導致連結斷開的情況。

2、服務端可以通過心跳來判斷客戶端是否在線，如果客戶端在規定時間內沒有發來任何數據，就認為客戶端下線。這樣可以檢測到客戶端由於極端情況(斷電、斷網等)下線的事件。

心跳間隔建議值：

建議客戶端發送心跳間隔小於60秒，比如55秒。

> 心跳的數據格式沒有要求，服務端能識別即可。

## 心跳示例
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 心跳間隔55秒
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // 給connection臨時設置一個lastMessageTime屬性，用來記錄上次收到消息的時間
    $connection->lastMessageTime = time();
    // 其它業務邏輯...
};

// 進程啟動後設置一個每10秒運行一次的定時器
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function()use($worker){
        $time_now = time();
        foreach($worker->connections as $connection) {
            // 有可能該connection還沒收到過消息，則lastMessageTime設置為當前時間
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // 上次通訊時間間隔大於心跳間隔，則認為客戶端已經下線，關閉連結
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

以上配置為如果客戶端超過55秒沒有發送任何數據給服務端，則服務端認為客戶端已經掉線，服務端關閉連結並觸發onClose。

## 斷線重連(重要)

不管是客戶端發送心跳還是服務端發送心跳，連結都有斷開的可能。例如瀏覽器最小化js被暫停、瀏覽器切換到其它tab頁面js被暫停、電腦進入睡眠等等、移動端切換網絡、信號變弱、手機黑屏、手機應用切換到後臺、路由故障、業務主動斷開等。尤其是外網環境複雜，很多路由節點會清理1分鐘內不活躍的連結，這也是為什麼心跳間隔推薦小於1分鐘的原因。

連結在外網環境很容易被斷開，所以斷線重連是長連結應用必須具備的功能(斷線重連只能客戶端做，服務端無法實現)。例如瀏覽器websocket需要監聽onclose事件，當發生onclose時建立新的連結(為避免需崩可延建立連結)。更嚴格一點，服務端也應該定時發起心跳數據，並且客戶端需要定時監測服務端的心跳數據是否超時，超過規定時間未收到服務端心跳數據應該認定連結已經斷開，需要執行close關閉連結，並重新建立新的連結。
