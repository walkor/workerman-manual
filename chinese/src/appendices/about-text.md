# text协议
Workerman定义了一种叫做text的文本协议，协议格式为 ```数据包+换行符```，即在每个数据包末尾加上一个换行符表示包的结束。

例如下面的buffer1和buffer2字符串符合text协议

```php
// 文本加一个回车
$buffer1 = 'abcdefghijklmn
';
// 在php中双引号中的\n代表一个换行符，例如"\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// 与服务端建立socket连接
$client = new stream_socket_client('tcp://127.0.0.1:5678');
// 以text协议发送buffer1数据
fwrite($client, $buffer1);
// 以text协议发送buffer2数据
fwrite($client, $buffer2);
```

text协议非常简单易用，如果开发者需要一个属于自己的协议，例如与手机App传输数据或者与硬件通讯等等，可以考虑使用text协议，开发调试都非常方便。

**text协议调试**

text协议可以使用telnet客户端调试，例如下面的例子。

test.php

```php
require_once './Workerman/Autoloader.php';
use Workerman\Worker;

$text_worker = new Worker("text://0.0.0.0:5678");

$text_worker->onMessage =  function($connection, $data)
{
    var_dump($data);
    $connection->send("hello world");
};

Worker::runAll();
```

执行```php test.php start```显示如下
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

重新打开一个终端，利用telnet测试（建议用linux系统的telnet）

假设是本机测试，
终端执行 telnet 127.0.0.1 5678
然后输入 hi回车
会接收到数据hello world\n
```
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world

```

