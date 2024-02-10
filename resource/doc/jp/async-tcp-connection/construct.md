# __construct メソッド
```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
非同期接続オブジェクトを作成します。

AsyncTcpConnectionを使用すると、Workermanはクライアントとしてリモートサーバーに非同期接続し、sendインターフェイスおよびonMessageコールバックを使用してデータの非同期送信および接続上のデータの処理が可能です。

## パラメータ
パラメータ：``` remote_address ```

接続先のアドレス。
例：
 ``` tcp://www.baidu.com:80 ```
 ``` ssl://www.baidu.com:443 ```
 ``` ws://echo.websocket.org:80 ```
 ``` frame://192.168.1.1:8080 ```
 ``` text://192.168.1.1:8080 ```

パラメータ：``` $context_option ```

``` このパラメータは（workerman >= 3.3.5）が必要です。 ```

ソケットコンテキストを設定するために使用され、```bindto```を利用してどの（ネットワークカード）IPとポートで外部ネットワークにアクセスするか、またはSSL証明書を設定することができます。

参考：[stream_context_create](https://php.net/manual/en/function.stream-context-create.php)、 [ソケットコンテキストのオプション](https://php.net/manual/zh/context.socket.php)、[SSLコンテキストのオプション](https://php.net/manual/zh/context.ssl.php)

## 注意

現在AsyncTcpConnectionがサポートするプロトコルは、[tcp](https://baike.baidu.com/subview/32754/8048820.htm)、[ssl](https://baike.baidu.com/view/525499.htm)、[ws](appendices/about-ws.md)、[frame](appendices/about-frame.md)、[text](appendices/about-text.md)です。

また、カスタムプロトコルもサポートされており、[カスタムプロトコルの作成方法](../protocols/how-protocols.md)を参照してください。

なお、[ssl](https://baike.baidu.com/view/525499.htm)はWorkerman>=3.3.4および[openssl拡張](https://php.net/manual/zh/book.openssl.php)のインストールが必要です。

現時点では、[http](https://baike.baidu.com/view/9472.htm)プロトコルのAsyncTcpConnectionはサポートされていません。

```new AsyncTcpConnection('ws://...')```のように、WorkermanでブラウザのようにリモートWebSocketサーバーに接続することができますが、```new AsyncTcpConnection('websocket://...')```のようにWorkermanでWebSocket接続を確立することはできません。

## 例

### 例1：外部のhttpサービスへの非同期アクセスの例
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// ワーカーの起動時にwww.baidu.comへの接続を非同期で確立し、データを送信して取得します
$task->onWorkerStart = function($task)
{
    // 直接httpを指定することはできませんが、TCPを使用してテストブラウザのようにhttpプロトコルのデータを送信できます。
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // 接続が確立した場合は、httpリクエストデータを送信します
    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "connect success\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "connection closed\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Error code:$code msg:$msg\n";
    };
    $connection_to_baidu->connect();
};

// ワーカーを実行する
Worker::runAll();
```

### 例2：外部WebSocketサービスへの非同期アクセスおよびローカルIPおよびポートの設定の例
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // 相手のホストにアクセスするためのローカルIPおよびポートの設定（各ソケット接続はローカルポートを占有します）
    $context_option = array(
        'socket' => array(
            // ipはローカルネットワークカードipである必要があり、相手のホストにアクセス可能でなければならない
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

### 例3：外部wssポートへの非同期アクセスとローカルSSL証明書の設定の例
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // 相手のホストにアクセスするためのローカルIPおよびポートおよびSSL証明書の設定
    $context_option = array(
        'socket' => array(
            // ipはローカルネットワークカードipである必要があり、相手のホストにアクセス可能でなければならない
            'bindto' => '114.215.84.87:2333',
        ),
        // SSLオプション、詳細はhttps://php.net/manual/zh/context.ssl.phpを参照してください。
        'ssl' => array(
            // ローカル証明書のパス。PEM形式でなければならず、ローカル証明書と秘密鍵を含んでいなければなりません。
            'local_cert'        => '/your/path/to/pemfile',
            // local_certファイルのパスワード。
            'passphrase'        => 'your_pem_passphrase',
            // 自己署名証明書を許可するかどうか。
            'allow_self_signed' => true,
            // SSL証明書を検証する必要があるかどうか。
            'verify_peer'       => false
        )
    );

    // 非同期接続を確立する
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // SSL暗号化方式でアクセスするように設定する
    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```
