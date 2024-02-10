# コンストラクタ __construct

## 説明:
```php
Worker::__construct([string $listen , array $context])
```

Workerコンテナのインスタンスを初期化し、コンテナのいくつかのプロパティやコールバックインターフェースを設定し、特定の機能を実現します。

## パラメータ
#### **``` $listen ```** （オプション、未指定の場合はどのポートもリッスンしません）

```$listen``` パラメータが設定されている場合、ソケットのリッスンが実行されます。

$listen の形式は <プロトコル>://<リッスンアドレス>

**<プロトコル> には以下の形式が使用できます：**

tcp: 例 ```tcp://0.0.0.0:8686```
udp: 例 ```udp://0.0.0.0:8686```
unix: 例 ```unix:///tmp/my_file ``` ```(Workerman>=3.2.7が必要)```
http: 例 ```http://0.0.0.0:80```
websocket: 例 ```websocket://0.0.0.0:8686```
text: 例 ```text://0.0.0.0:8686``` ```(textはWorkermanの内蔵テキストプロトコルであり、telnetと互換性があります。詳細は付録のTextプロトコルを参照)```
その他のカスタムプロトコルは、本書の通信プロトコルのカスタマイズセクションを参照してください。

**<リッスンアドレス> には以下の形式が使用できます：**
Unixソケットの場合、アドレスはローカルディスクパスです。
Unixソケット以外の場合、アドレスの形式は <ローカルIP>:<ポート番号>
<ローカルIP> には ```0.0.0.0``` を指定すると、すべてのネットワークカード（内部IP、外部IP、そしてローカルループバック127.0.0.1）がリッスンされます。
<ローカルIP> が```127.0.0.1```の場合、ローカルループバックだけがリッスンされ、外部からのアクセスはできません。
<ローカルIP>が内部IPアドレス（たとえば```192.168.xx.xx```）である場合は、内部IPアドレスのみがリッスンされ、外部ユーザーからのアクセスはできません。
指定した<ローカルIP>がこのホストのIPアドレスではない場合、リッスンができず、```Cannot assign requested address```エラーが表示されます。

**注意：**<ポート番号>は65535を超えることはできません。<ポート番号>は1024より小さい場合、リッスンするにはroot権限が必要です。リッスンするポートはこのホストで使用されていないポートでなければなりません。そうでない場合、```Address already in use```エラーが表示されます。

#### **``` $context ```**
配列。ソケットのコンテキストオプションを渡すためのものです。[ソケットコンテキストオプション](https://php.net/manual/zh/context.socket.php)を参照してください。

## 例

Workerとしてhttpコンテナを使用してhttpリクエストを処理する

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send("hello");
};

// Workerを実行
Worker::runAll();
```

Workerとしてwebsocketコンテナを使用してwebsocketリクエストを処理する
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Workerを実行
Worker::runAll();
```

Workerとしてtcpコンテナを使用してtcpリクエストを処理する
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Workerを実行
Worker::runAll();
```

Workerとしてudpコンテナを使用してudpリクエストを処理する
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Workerを実行
Worker::runAll();
```

Workerがunixドメインソケットをリッスンする```（Workermanのバージョン>=3.2.7が必要）```
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Workerを実行
Worker::runAll();
```

リスンしていないWorkerコンテナで、定期的なタスクを処理する
```php
use \Workerman\Worker;
use \Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // 2.5秒ごとに実行
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// Workerを実行
Worker::runAll();
```

**Workerがカスタムプロトコルのポートをリッスンする**

最終的なディレクトリ構造
```
├── Protocols              // 作成するProtocolsディレクトリ
│   └── MyTextProtocol.php // 作成するカスタムプロトコルファイル
├── test.php  // 作成するtestスクリプト
└── Workerman // Workermanのソースコードディレクトリ、その中身は修正しないでください
```

1. ディレクトリProtocolsを作成し、プロトコルファイルを作成します
Protocols/MyTextProtocol.php（上記のディレクトリ構造を参照してください）

```php
// ユーザー定義プロトコルの名前空間はProtocolsに統一されています
namespace Protocols;
//シンプルなテキストプロトコルで、プロトコル形式は テキスト+改行 です
class MyTextProtocol
{
    // パッケージの長さを返します
    public static function input($recv_buffer)
    {
        // 改行を検索
        $pos = strpos($recv_buffer, "\n");
        // 改行が見つからない場合、完全なパッケージではないことを意味し、データを待機するために0を返します
        if($pos === false)
        {
            return 0;
        }
        // 改行が見つかった場合、改行を含む現在のパッケージの長さを返します
        return $pos+1;
    }

    // 完全なパッケージを受信した後、自動的にdecodeが実行されます。今回は単にtrimして改行を削除します
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // クライアントにデータを送信する前に、encodeによって自動的にエンコードされます。ここでは改行が追加されます
    public static function encode($data)
    {
        return $data."\n";
    }
}
```

2. MyTextProtocolプロトコルを使用してリクエストを処理します

上記の最終的なディレクトリ構造に従ってtest.phpファイルを作成してください。

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// #### MyTextProtocol worker ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * 完全なデータ（末尾が改行）を受け取った後、自動的にMyTextProtocol::decode('受信したデータ')が実行されます
 * 結果は$data経由でonMessageコールバックに送信されます
 */
$text_worker->onMessage =  function(TcpConnection $connection, $data)
{
    var_dump($data);
    /*
     * クライアントにデータを送信する際、自動的にMyTextProtocol::encode('hello world')によってプロトコルがエンコードされ、
     * その後クライアントに送信されます
     */
    $connection->send("hello world");
};

// すべてのワーカーを実行
Worker::runAll();
```

3. テスト

ターミナルを開き、test.phpがあるディレクトリに移動し、```php test.php start```を実行します。
```php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

別のターミナルを開き、telnetを使用してテストします（Linuxシステムのtelnetを使用することをお勧めします）

ローカルでテストする場合、
ターミナルで telnet 127.0.0.1 5678 を実行し、その後に hiと入力してEnterキーを押します。
そうするとデータhello world\nを受け取れます。

```telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
