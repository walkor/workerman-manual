# Purpose of Communication Protocol
As TCP is stream-based, the data sent by the client flows into the server like a stream of water. The server needs to check whether the data is complete when it detects incoming data, as it may only be a partial request or even multiple requests concatenated together. To determine if the request has completely arrived or to separate requests from multiple concatenated ones, a set of communication protocols needs to be defined.

## Why Protocol Specification in Workerman?
Traditional PHP development is mostly based on the web and primarily uses the HTTP protocol. The parsing and handling of the HTTP protocol is solely handled by the WebServer, so developers do not need to worry about protocol-related issues. However, when there is a need to develop based on non-HTTP protocols, developers need to consider the protocol specifications.

## Protocols Currently Supported by Workerman
Workerman currently supports HTTP, websocket, text protocol (see appendix for details), frame protocol (see appendix for details), and ws protocol (see appendix for details). When communication based on these protocols is required, they can be used directly by specifying the protocol when initializing a Worker, for example:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket://0.0.0.0:2345 indicates listening on port 2345 using websocket protocol
$websocket_worker = new Worker('websocket://0.0.0.0:2345');

// text protocol
$text_worker = new Worker('text://0.0.0.0:2346');

// frame protocol
$frame_worker = new Worker('frame://0.0.0.0:2347');

// tcp Worker, directly based on socket transmission, no application layer protocol used
$tcp_worker = new Worker('tcp://0.0.0.0:2348');

// udp Worker, no application layer protocol used
$udp_worker = new Worker('udp://0.0.0.0:2349');

// unix domain Worker, no application layer protocol used
$unix_worker = new Worker('unix:///tmp/wm.sock');

```

## Using Custom Communication Protocols
When the built-in communication protocols in Workerman cannot meet development needs, developers can customize their own communication protocols. The method for customizing this is described in the next section.

**Note:**

Workerman has a built-in text protocol, which has a format of text + newline character. The text protocol is simple for development and debugging, and can be used in the majority of custom protocol scenarios, and also supports telnet debugging. If developers need to develop their own application protocol, they can directly use the text protocol without the need for separate development.

For details on the text protocol, refer to the "Appendix: Text Protocol" section.
