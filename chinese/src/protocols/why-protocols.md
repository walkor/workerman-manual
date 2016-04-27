# 通讯协议的作用
由于TCP是基于流的，客户端发送的请求数据是像水流一样流入到服务端，服务端探测到有数据到来后应该检查数据是否是完整的，因为可能只是一个请求的部分数据到达服务端，甚至可能是多个请求连在一起到达服务端。如何判断请求是否全部到达或者从多个连在一起的请求中分离请求，就需要规定一套通讯协议。

## 在WorkerMan中为什么要制定协议？

传统PHP开发都是基于Web的，基本上都是HTTP协议，HTTP协议的解析处理都由WebServer独自承担了，所以开发者不会关心协议方面的事情。然而当我们需要基于非HTTP协议开发时，开发者就需要考虑协议的事情了。

## WorkerMan已经支持的协议
WorkerMan目前已经支持HTTP、websocket、text协议(见附录说明)、frame协议(见附录说明)，ws协议(见附录说明)，需要基于这些协议通讯时可以直接使用，使用方法及时在初始化Worker时指定协议，例如
```php
use Workerman\Worker;
// websocket://0.0.0.0:2345 表明用websocket协议监听2345端口
$websocket_worker = new Worker('websocket://0.0.0.0:2345');

// text协议
$text_worker = new Worker('text://0.0.0.0:2346');

// frame协议
$frame_worker = new Worker('frame://0.0.0.0:2347');

// tcp Worker，直接基于socket传输，不使用任何应用层协议
$tcp_worker = new Worker('tcp://0.0.0.0:2348');

// udp Worker，不使用任何应用层协议
$udp_worker = new Worker('udp://0.0.0.0:2349');

// unix domain Worker，不使用任何应用层协议
$unix_worker = new Worker('unix:///tmp/wm.sock');

```

## 使用自定义的通讯协议
当WorkerMan自带的通讯协议满足不了开发需求时，开发者可以定制自己的通讯协议，定制方法见下一节内容。

**提示：**

Workerman内置了一个text协议，协议格式为文本+换行符。text协议开发调试都非常简单，可用于绝大多数自定义协议的场景，并且支持telnet调试。如果开发者要开发自己的应用协议，可以直接使用text协议，不用再单独开发。

text协议说明参考《附录 Text协议部分》
