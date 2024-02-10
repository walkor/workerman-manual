# Simple Development Example

## Installation

**Install workerman**

Run the following command in an empty directory:

```
composer require workerman/workerman
```

## Example 1: Providing Web Services Externally Using HTTP Protocol

**Create start.php file**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Create a Worker to listen on port 2345 and use the http protocol for communication
$http_worker = new Worker("http://0.0.0.0:2345");

// Start 4 processes to provide external services
$http_worker->count = 4;

// Reply with "hello world" to the browser when receiving data sent by the browser
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Send "hello world" to the browser
    $connection->send('hello world');
};

// Run the worker
Worker::runAll();
```

**Run in command line (for Windows users, use [cmd command line](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn))**
```shell
php start.php start
```

**Testing**

Assuming the server IP is 127.0.0.1

Access the URL http://127.0.0.1:2345 in a web browser

**Note:**

1. If there is an issue with accessing it, please refer to the [reasons for client connection failure](../faq/client-connect-fail.md) section for troubleshooting.

2. The server uses the http protocol and can only communicate with the http protocol. It cannot directly communicate with other protocols such as websocket.

## Example 2: Providing Services Externally Using WebSocket Protocol

**Create ws_test.php file**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Note: Unlike the previous example, use the websocket protocol here
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// Start 4 processes to provide external services
$ws_worker->count = 4;

// When receiving data from the client, return "hello $data" to the client
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Send "hello $data" to the client
    $connection->send('hello ' . $data);
};

// Run the worker
Worker::runAll();
```

**Run in command line**
```shell
php ws_test.php start
```

**Testing**

Open the Chrome web browser, press F12 to open the debug console, and enter the following in the Console tab (or put the following code in an HTML page and run it with JavaScript):

```javascript
// Assuming the server IP is 127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("Connection successful");
    ws.send('tom');
    alert("Sent a string 'tom' to the server");
};
ws.onmessage = function(e) {
    alert("Received a message from the server: " + e.data);
};
```

**Note:**

1. If there is an issue with accessing it, please refer to the [Common Problems in the Manual - Connection Failure](../faq/client-connect-fail.md) section for troubleshooting.

2. The server uses the websocket protocol and can only communicate with the websocket protocol. It cannot directly communicate with other protocols such as http.

## Example 3: Directly Transmitting Data using TCP

**Create tcp_test.php file**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Create a Worker listening on port 2347, without using any application layer protocol
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// Start 4 processes to provide external services
$tcp_worker->count = 4;

// When the client sends data
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Send "hello $data" to the client
    $connection->send('hello ' . $data);
};

// Run the worker
Worker::runAll();
```

**Run in command line**

```shell
php tcp_test.php start
```

**Testing: Run in command line**

(The following is the effect in a Linux command line, which may differ from the effect in Windows)

```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```

**Note:**

1. If there is an issue with accessing it, please refer to the [Common Problems in the Manual - Connection Failure](../faq/client-connect-fail.md) section for troubleshooting.

2. The server uses the raw tcp protocol and cannot directly communicate with protocols such as websocket, http, etc.
