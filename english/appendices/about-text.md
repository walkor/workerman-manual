# Text Protocol
Workerman defines a text protocol called "text", with the format of the protocol being ```message+newline```, which means adding a newline at the end of each message to indicate the end of the package.

For example, the following buffer1 and buffer2 strings conform to the text protocol:

```php
// Text plus a carriage return
$buffer1 = 'abcdefghijklmn
';
// In PHP, \n in double quotes represents a newline character, for example, "\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// Establish a socket connection with the server
$client = stream_socket_client('tcp://127.0.0.1:5678');
// Send buffer1 data using the text protocol
fwrite($client, $buffer1);
// Send buffer2 data using the text protocol
fwrite($client, $buffer2);
```

The text protocol is very simple and easy to use. If developers need their own protocol, such as transmitting data to a mobile app or communicating with hardware, they can consider using the text protocol, which is very convenient for development and debugging.

**Text Protocol Debugging**
The text protocol can be debugged using a telnet client. For example, in the following example:

Create a new file test.php

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

Run ```php test.php start``` and the output will be as follows:

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

Open a new terminal and use telnet to test (recommended to use the telnet of a Linux system).

Assuming it is a local test,
Run telnet 127.0.0.1 5678 in the terminal
Then enter 'hi' and press enter
Data 'hello world' followed by a newline will be received

```
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
