# workerman/mqtt
MQTTは、クライアントサーバーアーキテクチャを持つ、パブリッシュ/サブスクライブパターンのメッセージングプロトコルであり、すでにIoTの重要な構成要素となっています。その設計思想は、軽量、オープン、簡単、標準化されており、実装が容易です。これらの特性により、多くのシナリオにとって適しており、特にM2M通信やIoT環境などの制約のある環境にとって非常に適しています。

workerman\mqtt は、workermanをベースにした非同期mqttクライアントライブラリで、mqttプロトコルのメッセージの受信または送信に使用できます。`QoS 0`、`QoS 1`、`QoS 2`をサポートしています。`MQTT` `3.1` `3.1.1` `5` バージョンをサポートしています。

# プロジェクトURL
https://github.com/walkor/mqtt

# インストール
```composer require workerman/mqtt```

# 例
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
コマンドラインで```php subscribe.php start```を実行して起動します。

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
コマンドラインで```php publish.php start```を実行して起動します。

## workerman\mqtt\Client Interface

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

mqttクライアントインスタンスの作成。

  * `$address` mqttサーバーアドレス、フォーマットは 'mqtt://test.mosquitto.org:1883'

  * `$options` クライアントオプション配列で、以下のオプションを設定できます：
    * `keepalive`: クライアントがサーバーに定期的に心拍を送信する間隔、デフォルトは50秒で、0に設定すると心拍を送信しなくなります
    * `client_id`: クライアントID、未設定の場合は ```"workerman-mqtt-client-".mt_rand()``` になります
    * `protocol_name`: プロトコル名、`MQTT`(3.1.1バージョン)または `MQIsdp`(3.1バージョン)、デフォルトは`MQTT`
    * `protocol_level`: プロトコルレベル、`protocol_name`が`MQTT`の場合は`4`、`protocol_name`が`MQIsdp`の場合は`3`
    * `clean_session`: クリーンセッション、デフォルトは`true`。`false`に設定すると、`QoS 1`および`QoS 2`レベルのオフラインメッセージを受信できます。
    * `reconnect_period`: 再接続間隔、デフォルトは `1`秒、`0`は再接続しないことを表します
    * `connect_timeout`: mqtt接続のタイムアウト、デフォルトは`30`秒
    * `username`: ユーザー名、オプション
    * `password`: パスワード、オプション
    * `will`: 遺言メッセージ、クライアントが切断された後、ブローカーは自動的に遺言メッセージを他のクライアントに送信します。以下の形式です:
      * `topic`: トピック
      * `content`: コンテンツ
      * `qos`: `QoS`レベル
      * `retain`: `retain`フラグ
    * `resubscribe` : 接続が異常終了し、再接続後に以前のトピックを再購読するかどうか
    * `bindto` ローカルIPとポートを指定してBrokerに接続するために使用される、デフォルトは''
    * `ssl` sslオプション、デフォルトは `false`、`true`に設定するとssl接続になります。同時に、ローカル証明書の構成など、sslコンテキスト配列を渡すこともサポートしており、sslコンテキストについてはhttps://php.net/manual/en/context.ssl.phpを参照してください
    * `debug` デバッグモードを有効にするかどうか、デフォルトは`false`

-------------------------------------------------------

<a name="connect"></a>
### connect()

ブローカーに接続します。

-------------------------------------------------------

<a name="publish"></a>
### publish(string $topic, string $content, [array $options], [callable $callback])

特定のトピックにメッセージを公開します。

* `$topic` トピック
* `$content` メッセージ
* `$options` オプション配列、含まれるもの
  * `qos` `QoS`レベル、デフォルトは`0`
  * `retain` retainフラグ、デフォルトは`false`
  * `dup` 重複フラグ、デフォルトは`false`
* `$callback` - `function (\Exception $exception = null)`、（QoSが0の場合、この設定はサポートされません）エラーが発生した場合や成功した場合にトリガーされます、 `$exception` は例外オブジェクトであり、エラーが発生しない場合は `$exception` は`null`になります。下記も同様です。

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $topic, [array $options], [callable $callback])

1つまたは複数のトピックを購読します。

* `$topic` は文字列（1つのトピックを購読する場合）または配列（複数のトピックを購読する場合）です。
複数のトピックを購読する場合、`$topic`はトピックがキーで、`QoS`が値の配列です。例: `array('topic1'=> 0, 'topic2'=> 1)`
* `$options` 購読オプション配列、以下の設定が含まれます：
  * `qos` `QoS`レベル、デフォルトは `0`
* `$callback` - `function (\Exception $exception = null, array $granted = [])`、購読が成功するかエラーが発生した場合にトリガーされます。
  * `exception` 例外オブジェクト、エラーがない場合は`null`です

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $topic, [callable $callback])

購読を解除します。

* `$topic` は1つまたは複数の文字列で、例: `array('topic1', 'topic2')`
* `$callback` - `function (\Exception $e = null)`、成功または失敗した場合にトリガーされるコールバック関数

-------------------------------------------------------

<a name="disconnect"></a>
### disconnect()

ブローカーとの接続を正常に切断します。 `DISCONNECT`メッセージがブローカーに送信されます。

-------------------------------------------------------

<a name="close"></a>
### close()

ブローカーとの接続を強制的に切断します。ブローカーには`DISCONNECT`メッセージが送信されません。

-------------------------------------------------------

<a name="onConnect"></a>
### callback onConnect(Client $mqtt)
ブローカーとの接続が確立されると、トリガーされます。この時点で、ブローカーから `CONNACK` メッセージを受信します。

-------------------------------------------------------

<a name="onMessage"></a>
### callback onMessage(string $topic, string $content, Client $mqtt)
`function (topic, message, packet) {}`

クライアントがPublishメッセージを受信した際にトリガーされます。
* `$topic` 受信したトピック
* `$content` 受信したメッセージの内容
* `$mqtt` mqttクライアントインスタンス

-------------------------------------------------------

<a name="onError"></a>
### callback onError(\Exception $exception = null)
接続時にエラーが発生した場合にトリガーされます

-------------------------------------------------------

<a name="onClose"></a>
### callback onClose()
接続が閉じられると、クライアントが自発的に閉じる場合やサーバーが閉じる場合にトリガーされます。
