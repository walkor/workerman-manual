# send
## 说明:
```php
mixed Connection::send(mixed $data [,$raw = false])
```

向客户端发送数据

## 参数

 ``` $data ```

要发送的数据，如果在初始化Worker类时指定了协议，则会自动调用协议的encode方法,完成协议打包工作后发送给客户端

 ``` $raw ```
 
是否发送原始数据，即不调用协议的encode方法，默认是false，即自动调用协议的encode方法

## 返回值

true 表示数据已经成功写入到该连接的操作系统层的socket发送缓冲区

null 表示数据已经写入到该连接的应用层发送缓冲区，等待向系统层socket发送缓冲区写入

false 表示发送失败，失败原因可能是客户端连接已经关闭，或者该连接的应用层发送缓冲区已满

## 注意
send返回```true```，仅仅代表数据已经成功写入到该连接的操作系统socket发送缓冲区，并不意味着数据已经成功的发送给对端socket接收缓冲区，更不意味着对端应用程序已经从本地socket接收缓冲区读取了数据。**不过即便如此，只要send不返回false并且网络没有断开，而且客户端接收正常，数据基本上可以看做100%能发到对方的。**

由于socket发送缓冲区的数据是由操作系统异步发送给对端的，操作系统并没有给应用层提供相应确认机制，所以**应用层**无法得知socket发送缓冲区的数据何时开始发送，**应用层**更无法得知socket发送缓冲区的数据是否发送成功。基于以上原因workerman无法直接提消息确认接口。

如果业务需要保证每个消息客户端都收到，可以在业务上增加一种确认机制。确认机制可能根据业务不同而不同，即使同样的业务确认机制也可以有多种方法。

例如聊天系统可以用这样的确认机制。把每条消息都存入数据库，每条消息都有一个是否已读字段。客户端每收到一条消息向服务端发送一个确认包，服务端将对应消息置为已读。当客户端连接到服务端时（一般是用户登录或者断线重连），查询数据库是否有未读的消息，有的话发给客户端，同样客户端收到消息后通知服务端已读。这样可以保证每个消息对方都能收到。当然开发者也可以用自己的确认逻辑。



## 范例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 会自动调用\Workerman\Protocols\Websocket::encode打包成websocket协议数据后发送
    $connection->send("hello\n");
};
// 运行worker
Worker::runAll();
```
