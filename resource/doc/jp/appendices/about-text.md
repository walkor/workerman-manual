# textプロトコル
Workermanは、textと呼ばれるテキストプロトコルを定義しています。このプロトコル形式は、```データパケット+改行```であり、つまり各データパケットの末尾に改行を追加してパケットの終了を示します。

以下はbuffer1とbuffer2の文字列がtextプロトコルに準拠している例です。

```php
// テキストに改行を追加
$buffer1 = 'abcdefghijklmn
';
// PHPではダブルクォーテーション内の\nは改行を意味し、例えば"\n"となります。
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// サーバーとのソケット接続を確立
$client = stream_socket_client('tcp://127.0.0.1:5678');
// textプロトコルでbuffer1データを送信
fwrite($client, $buffer1);
// textプロトコルでbuffer2データを送信
fwrite($client, $buffer2);
```

textプロトコルは非常にシンプルで使いやすく、例えば開発者が独自のプロトコルを必要とする場合（例：携帯アプリとのデータ転送、ハードウェアとの通信など）、textプロトコルを使用することを検討すると開発とデバッグが非常に簡単になります。

**textプロトコルのデバッグ**

textプロトコルはtelnetクライアントを使用してデバッグすることができます。以下は例です。

新しいファイル test.php を作成してください。

```php
require_once __DIR__ . '/Workerman/Autoloader.php';
use Workerman\Worker;

$text_worker = new Worker("text://0.0.0.0:5678");

$text_worker->onMessage =  function($connection, $data)
{
    var_dump($data);
    $connection->send("hello world");
};

Worker::runAll();
```

```php test.php start```を実行して以下の表示が表示されます。

```bash
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

新しい端末を開き、telnetでテストします（Linuxシステムのtelnetをお勧めします）。

自分自身をテストする場合は、
ターミナルで telnet 127.0.0.1 5678 を実行して、hiを入力してEnterを押します。
すると、データhello worldを受け取ります。

```bash
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
