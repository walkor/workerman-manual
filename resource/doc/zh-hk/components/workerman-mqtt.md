# workerman/mqtt
MQTT是一個客戶端服務端架構的發佈/訂閱模式的消息傳輸協議，已經成為物聯網的重要組成部分。它的設計思想是輕巧、開放、簡單、規範，易於實現。這些特點使得它對很多場景來說都是很好的選擇，特別是對於受限的環境如機器與機器的通信（M2M）以及物聯網環境（IoT）。

workerman\mqtt 是一個基於workerman的異步mqtt 客戶端庫，可用於接收或者發送mqtt協議的消息。支持`QoS 0`、`QoS 1`、`QoS 2`。支持`MQTT` `3.1` `3.1.1` `5`版本。


# 專案地址
https://github.com/walkor/mqtt

# 安裝 
```composer require workerman/mqtt```

# 示例
**subscribe.php**
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $mqtt = new Workerman\Mqtt\Client('mqtt://test.mosquitto.org:1883');
    $mqtt->onConnect = function($mqtt) {
        $mqtt->subscribe('test');
    };
    $mqtt->onMessage = function($topic, $content){
        var_dump($topic, $content);
    };
    $mqtt->connect();
};
Worker::runAll();
```
命令行運行  ```php subscribe.php start``` 啟動

**publish.php**
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $mqtt = new Workerman\Mqtt\Client('mqtt://test.mosquitto.org:1883');
    $mqtt->onConnect = function($mqtt) {
       $mqtt->publish('test', 'hello workerman mqtt');
    };
    $mqtt->connect();
};
Worker::runAll();
```

命令行運行 ```php publish.php start``` 啟動。

## workerman\mqtt\Client接口

  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#connect"><code>Client::<b>connect()</b></code></a>
  * <a href="#publish"><code>Client::<b>publish()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>
  * <a href="#disconnect"><code>Client::<b>disconnect()</b></code></a>
  * <a href="#close"><code>Client::<b>close()</b></code></a>
  * <a href="#onConnect"><code>callback <b>onConnect</b></code></a>
  * <a href="#onMessage"><code>callback <b>onMessage</b></code></a>
  * <a href="#onError"><code>callback <b>onError</b></code></a>
  * <a href="#onClose"><code>callback <b>onClose</b></code></a>

-------------------------------------------------------

<a name="construct"></a>
### __construct (string $address, [array $options])

創建一個mqtt客戶端實例。

  * `$address` mqtt服務端地址，格式類似 'mqtt://test.mosquitto.org:1883'. 

  * `$options` 客戶端選項數組，可以設置以下選項：
    * `keepalive`: 客戶端向服務端發送心跳的時間間隔，默認50秒，設置成0代表不啟用心跳
    * `client_id`: 客戶端id，如果沒設定默認是 ```"workerman-mqtt-client-".mt_rand()```
    * `protocol_name`: 協議名，`MQTT`(3.1.1版本)或者 `MQIsdp`(3.1版本)，默認是`MQTT`
    * `protocol_level`:協議等級， `protocol_name`是`MQTT` 時值為`4` ，`protocol_name`是`MQIsdp` 時值是 `3`
    * `clean_session`: 清理會話，默認為`true`。設置為`false`可以接收到`QoS 1`和`QoS 2`級別的離線消息
    * `reconnect_period`: 重連時間間隔，默認 `1` 秒，`0`代表不重連
    * `connect_timeout`: 連接mqtt超時時間，默認`30` 秒
    * `username`: 用戶名，可選
    * `password`: 密碼，可選
    * `will`: 遺囑消息，當客戶端斷線後Broker會自動發送遺囑消息給其它客戶端。 格式為:
      * `topic`: 主題
      * `content`: 內容
      * `qos`: `QoS`等級
      * `retain`: `retain`標記
    * `resubscribe` : 當連接異常斷開並重連後，是否重新訂閱之前的主題，默認為true
    * `bindto` 用來指定本地以哪個ip和端口向Broker發起連接，默認值為''
    * `ssl` ssl選項，默認是 `false`，如果設置為`true`，則以ssl方式連接。同時支持傳入ssl上下文數組，用來配置本地證書等，ssl上下文參考https://php.net/manual/en/context.ssl.php
    * `debug` 是否啟用debug模式，debug模式可以輸出與Broker通訊的詳細信息，默認為`false`

-------------------------------------------------------

<a name="connect"></a>
### connect()

連接Broker

-------------------------------------------------------

<a name="publish"></a>
### publish(String $topic, String $content, [array $options], [callable $callback])

向某個主題發布一條消息

* `$topic` 主題
* `$content` 消息
* `$options` 選項數組，包括
  * `qos` `QoS`等級，默認`0`
  * `retain` retain 標記，默認`false`
  * `dup` 重發標誌，默認`false`
* `$callback` - `function (\Exception $exception = null)`，（QoS為0時不支持此設置）當發生錯誤時或者發布成功時觸發， `$exception` 是異常對象，當沒有錯誤發生時 `$exception` 為`null`，下同。

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $topic, [array $options], [callable $callback])

訂閱一個主題或者多個主題

* `$topic` 是一個字符串(訂閱一個主題)或者數組(訂閱多個主題)，
當訂閱多個主題時，`$topic`是主題是key，`QoS`為值的數組，例如`array('topic1'=> 0, 'topic2'=> 1)`
* `$options` 訂閱選項數組，包含以下設置:
  * `qos` `QoS`等級, 默認 `0`
* `$callback` - `function (\Exception $exception = null, array $granted = [])`
  回調函數，當訂閱成功或者發生錯誤時觸發
  * `exception` 異常對象，無錯誤發生時它是`null`，下同
  * `granted` 訂閱結果數組，類似 `array('topic' => 'qos', 'topic' => 'qos')` 其中:
    * `topic` 是訂閱的主題
    * `qos` Broker接受的`QoS`等級

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $topic, [callable $callback])

取消訂閱

* `$topic` 是一個字符串或者字符串數組，類似`array('topic1', 'topic2')`
* `$callback` - `function (\Exception $e = null)`，成功或者失敗時觸發的回調

-------------------------------------------------------

<a name="disconnect"></a>
### disconnect()

正常斷開與Broker的連接， `DISCONNECT`報文會被發送到Broker.

-------------------------------------------------------

<a name="close"></a>
### close()

強制斷開與Broker的連接，不會發送`DISCONNECT`報文給Broker.

-------------------------------------------------------

<a name="onConnect"></a>
### callback onConnect(Client $mqtt)
當與Broker連接建立完畢後觸發。這時候已經收到了Broker的`CONNACK` 報文。

-------------------------------------------------------

<a name="onMessage"></a>
### callback onMessage(String $topic, String $content, Client $mqtt)
`function (topic, message, packet) {}`

當客戶端收到Publish報文時觸發
* `$topic` 收到的主題
* `$content` 收到的消息內容
* `$mqtt` mqtt客戶端實例.

-------------------------------------------------------

<a name="onError"></a>
### callback onError(\Exception $exception = null)
當連接發生某種錯誤時觸發

-------------------------------------------------------

<a name="onClose"></a>
### callback onClose()
當連接關閉時觸發，無論是客戶端主動關閉還是服務端關閉都會觸發`onClose`
