# text協議
> Workerman 定義了一種稱為 text 的文本協議，協議格式為 ```數據包+換行符```，即在每個數據包末尾加上一個換行符表示包的結束。

例如下面的 buffer1 和 buffer2 字串符合 text 協議：

```php
// 文本加一个回车
$buffer1 = 'abcdefghijklmn
';
// 在php中雙引號中的\n代表一個換行符，例如"\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// 與服務端建立 socket 連接
$client = stream_socket_client('tcp://127.0.0.1:5678');
// 以 text 協議發送 buffer1 數據
fwrite($client, $buffer1);
// 以 text 協議發送 buffer2 數據
fwrite($client, $buffer2);
```

text 協議非常簡單易用，如果開發者需要一個屬於自己的協議，例如與手機 App 傳輸數據或者與硬件通訊等等，可以考慮使用 text 協議，開發調試都非常方便。

**text 協議調試**

> text 協議可以使用 telnet 客戶端調試，例如下面的例子：

新建文件 test.php

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

```php
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

重新打開一個終端，利用 telnet 測試（建議用 Linux 系統的 telnet）

假設是本機測試，
終端執行 telnet 127.0.0.1 5678
然後輸入 hi 回車
會接收到數據 hello world\n
```php
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world

```
