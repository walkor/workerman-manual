# send
## 說明:
```php
mixed Connection::send(mixed $data [,$raw = false])
```

向客戶端發送數據

## 參數

 ``` $data ```

要發送的數據，如果在初始化Worker類時指定了協議，則會自動調用協議的encode方法,完成協議打包工作後發送給客戶端

 ``` $raw ```
 
是否發送原始數據，即不調用協議的encode方法，預設是false，即自動調用協議的encode方法

## 返回值

true 表示數據已經成功寫入到該連接的操作系統層的socket發送緩衝區

null 表示數據已經寫入到該連接的應用層發送緩衝區，等待向系統層socket發送緩衝區寫入

false 表示發送失敗，失敗原因可能是客戶端連接已經關閉，或者該連接的應用層發送緩衝區已滿

## 注意
send返回```true```，僅僅代表數據已經成功寫入到該連接的操作系統socket發送緩衝區，並不意味著數據已經成功的發送給對端socket接收緩衝區，更不意味著對端應用程序已經從本地socket接收緩衝區讀取了數據。**不過即便如此，只要send不返回false並且網絡沒有斷開，而且客戶端接收正常，數據基本上可以看做100%能發到對方的。**

由於socket發送緩衝區的數據是由操作系統異步發送給對端的，操作系統並沒有給應用層提供相應確認機制，所以**應用層**無法得知socket發送緩衝區的數據何時開始發送，**應用層**更無法得知socket發送緩衝區的數據是否發送成功。基於以上原因workerman無法直接提消息確認接口。

如果業務需要保證每個消息客戶端都收到，可以在業務上增加一種確認機制。確認機制可能根據業務不同而不同，即使同樣的業務確認機制也可以有多種方法。

例如聊天系統可以用這樣的確認機制。把每條消息都存入數據庫，每條消息都有一個是否已讀字段。客戶端每收到一條消息向服務端發送一個確認包，服務端將對應消息置為已讀。當客戶端連接到服務端時（一般是用戶登錄或者斷線重連），查詢數據庫是否有未讀的消息，有的話發給客戶端，同樣客戶端收到消息後通知服務端已讀。這樣可以保證每個消息對方都能收到。當然開發者也可以用自己的確認邏輯。

## 範例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 會自動調用\Workerman\Protocols\Websocket::encode打包成websocket協議數據後發送
    $connection->send("hello\n");
};
// 運行worker
Worker::runAll();
```
