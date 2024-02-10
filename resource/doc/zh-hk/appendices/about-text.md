# text協議
> Workerman定義了一種叫做text的文本協議，協議格式為 ```數據包+換行符```，即在每個數據包末尾加上一個換行符表示包的結束。

例如下面的buffer1和buffer2字符串符合text協議：

```php
// 文本加一个回车
$buffer1 = 'abcdefghijklmn
';
// 在php中雙引號中的\n代表一個換行符，例如"\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// 與服務端建立socket連接
$client = stream_socket_client('tcp://127.0.0.1:5678');
// 以text協議發送buffer1數據
fwrite($client, $buffer1);
// 以text協議發送buffer2數據
fwrite($client, $buffer2);
```

text協議非常簡單易用，如果開發者需要一個屬於自己的協議，例如與手機App傳輸數據或者與硬件通訊等等，可以考慮使用text協議，開發調試都非常方便。

**text協議調試**

> text協議可以使用telnet客戶端調試，例如下面的例子:

新建文件test.php
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
執行```php test.php start```顯示如下
```
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
重新打開一個終端，利用telnet測試（建議用linux系統的telnet）
假設是本機測試，
終端執行 `telnet 127.0.0.1 5678`
然後輸入 hi回車
會接收到數據hello world\n
```
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world

```
