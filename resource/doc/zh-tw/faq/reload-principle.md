# 平滑重啟原理
## 什麼是平滑重啟？

平滑重啟不同於普通的重啟，它可以在不影響使用者的情況下重啟服務（通常指短連接業務），以便重新載入PHP程序，完成業務代碼更新。

平滑重啟一般應用於業務更新或者版本發佈過程中，能夠避免因代碼發佈重啟服務導致的暫時性服務不可用的影響。

> **注意**
> Windows系統不支持reload。

> **注意**
> 長連接（例如websocket）業務，在進程平滑重啟時連接會被中斷。解決方案是使用類似[gatewayWorker](https://www.workerman.net/doc/gateway-worker)的架構，一組進程專門維持連接，並將這組進程的[reloadable](../worker/reloadable.md)屬性設置為false。業務邏輯啟動另外一組worker進程處理，gateway與worker進程通過TCP通訊的方式互相調用。當業務需要變更時，僅需重啟worker進程即可。

## 限制
**注意：僅有在on{...}回呼中載入的檔案平滑重啟後才會自動更新，啟動腳本中直接載入的檔案或者寫死的程式碼運行reload不會自動更新。**

#### 以下程式reload後不會更新
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('hi'); // 寫死的程式碼不支持熱更新
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/your/path/MessageHandler.php'; // 啟動腳本直接載入的檔案不支持熱更新
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // 假設MessageHandler類裡有一個onMessage方法
```

#### 以下程式reload後會自動更新
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart是進程啟動後觸發的回呼
    require_once __DIR__ . '/your/path/MessageHandler.php'; // 進程啟動後載入的檔案支持熱更新
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
MessageHandler.php改動後執行 `php start.php reload`，MessageHandler.php會重新載入內存達到更新業務邏輯的效果。

> **提示**
> 上面程式為了方便演示，使用了`require_once`語句，如果你的專案支持PSR4自動加載，則無需調用`require_once`語句。

## 平滑重啟原理

Workerman分為主進程和子進程，主進程負責監控子進程，子進程負責接收客戶端的連接和連接上發來的請求數據，做相應的處理並返回數據給客戶端。當業務代碼更新時，其實我們只要更新子進程，便可以達到更新代碼的目的。

當Workerman主進程收到平滑重啟信號時，主進程會向其中一個子進程發送安全退出（讓對應進程處理完畢當前請求後才退出）信號，當這個進程退出後，主進程會重新創建一個新的子進程（這個子進程載入了新的PHP代碼），然後主進程再次向另外一個舊的進程發送停止命令，這樣一個進程一個進程的重啟，直到所有舊的進程全部被置換為止。

我們看到平滑重啟實際上是讓舊的業務進程逐個退出然後並逐個創建新的進程做到的。為了在平滑重啟時不影響客戶用戶，這就要求進程中不要保存使用者相關的狀態信息，即業務進程最好是無狀態的，避免由於進程退出導致信息丟失。
