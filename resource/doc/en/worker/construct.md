# Constructor __construct

## Description:
```php
Worker::__construct([string $listen , array $context])
```

Initialize a Worker container instance, which can set some attributes of the container and callback interfaces to accomplish specific functions.


## Parameters
#### **```$listen```** (optional parameter, not specifying means not listening on any port)

If the ```$listen``` parameter is set, the socket will be listened to.

The format of $listen is <protocol>://<listening address>.

**<protocol> can be in the following formats:**
tcp: for example ```tcp://0.0.0.0:8686```
udp: for example ```udp://0.0.0.0:8686```
unix: for example ```unix:///tmp/my_file``` ```(requires Workerman>=3.2.7)```
http: for example ```http://0.0.0.0:80```
websocket: for example ```websocket://0.0.0.0:8686```
text: for example ```text://0.0.0.0:8686``` ```(text is the built-in text protocol in Workerman, compatible with telnet, see the Text Protocol section in the appendix for details)```
and other custom protocols, see the Custom Communication Protocol section in this manual

**<listening address> can be in the following format:**
If it is a Unix domain socket, the address is a local disk path
For non-Unix domain sockets, the address format is <local IP>:<port number>
<local IP> can be ```0.0.0.0``` to listen on all network cards on this machine, including internal IP and external IP as well as local loopback 127.0.0.1
If <local IP> is ```127.0.0.1```, it means listening on the local loopback, only accessible to the local machine, and inaccessible from the outside
If <local IP> is an internal IP, similar to ```192.168.xx.xx```, it means only listening on the internal IP, so external users cannot access it
If the value of <local IP> is not a local IP, it cannot be listened to, and an error ```Cannot assign requested address``` is prompted
**Note:** <port number> cannot be greater than 65535. If <port number> is less than 1024, root permissions are required to listen. The port being listened to must be a port not in use on this machine, otherwise, it cannot be listened to, and an error ```Address already in use``` is prompted.

#### **```$context```**

An array for passing socket context options, see [Socket Context Options](https://php.net/manual/en/context.socket.php)


## Examples

Worker acts as an HTTP container to listen and handle HTTP requests
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

// Run the worker
Worker::runAll();
```

Worker acts as a WebSocket container to listen and handle WebSocket requests
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Run the worker
Worker::runAll();
```

Worker acts as a TCP container to listen and handle TCP requests
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Run the worker
Worker::runAll();
```

Worker acts as a UDP container to listen and handle UDP requests
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Run the worker
Worker::runAll();
```

Worker listens to the Unix domain socket ```(requires Workerman version >=3.2.7)```
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Run the worker
Worker::runAll();
```

A Worker container that does not listen to any ports and is used to handle some scheduled tasks
```php
use \Workerman\Worker;
use \Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // Execute every 2.5 seconds
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// Run the worker
Worker::runAll();
```

**Worker listens on a port with a custom protocol**

Final directory structure
```
├── Protocols              // This is the Protocols directory to be created
│   └── MyTextProtocol.php // This is the custom protocol file to be created
├── test.php  // This is the test script to be created
└── Workerman // Workerman source code directory, do not modify the code inside
```

1. Create the Protocols directory and create a protocol file
Protocols/MyTextProtocol.php (based on the directory structure above)

```php
// The namespace for user-defined protocols is uniformly Protocols
namespace Protocols;
// Simple text protocol, the protocol format is text + line break
class MyTextProtocol
{
    // Fragmentation function, returns the length of the current package
    public static function input($recv_buffer)
    {
        // Find the line break
        $pos = strpos($recv_buffer, "\n");
        // If no line break is found, it means it is not a complete package, return 0 to continue waiting for data
        if($pos === false)
        {
            return 0;
        }
        // If a line break is found, return the length of the current package, including the line break
        return $pos+1;
    }

    // After receiving a complete package, the decode function is automatically executed through decode, here just trims off the line break
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // Before sending data to the client, it will be automatically encoded through encode, and then sent to the client. Here, a line break is added
    public static function encode($data)
    {
        return $data."\n";
    }
}
```

2. Use the MyTextProtocol protocol to listen and handle requests

Create the test.php file based on the final directory structure mentioned above

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// #### MyTextProtocol worker ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * After receiving a complete data (ending with a line break), MyTextProtocol::decode('received data') is automatically executed
 * The result is passed to the onMessage callback via $data
 */
$text_worker->onMessage =  function(TcpConnection $connection, $data)
{
    var_dump($data);
    /*
     * Send data to the client, MyTextProtocol::encode('hello world') is automatically called for protocol encoding
     * and then sent to the client
     */
    $connection->send("hello world");
};

// Run all workers
Worker::runAll();
```

3. Testing

Open the terminal, go to the directory where the test.php is located, and execute ```php test.php start```
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

Open the terminal and test using telnet (Linux telnet is recommended)

Assuming it is a local test,
Execute telnet 127.0.0.1 5678 in the terminal
Then enter hi and press Enter
You will receive the data hello world\n
```
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world

```
