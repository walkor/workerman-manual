# WebSocket协议

WebSocket protocol 是HTML5一种新的协议。它实现了浏览器与服务器全双工通信

## WebSocket与TCP关系

WebSocket和HTTP一样是一种应用层协议，都是基于TCP传输的，WebSocket本身和Socket并没有多大关系，更不能等同。

## WebSocket协议握手
WebSocket协议有一个握手的过程，握手时浏览器和服务端是以HTTP协议通信的，在Workerman中可以这样介入到握手过程。

```php
$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $http_header)
    {
        // 可以在这里判断连接来源是否合法，不合法就关掉连接
        // $_SERVER['HTTP_ORIGIN']标识来自哪个站点的页面发起的websocket链接
        if($_SERVER['HTTP_ORIGIN'] != 'http://chat.workerman.net')
        {
            $connection->close();
        }
        // onWebSocketConnect 里面$_GET $_SERVER是可用的
        // var_dump($_GET, $_SERVER);
    };
};
```

## WebSocket协议传输二进制数据

websocket协议默认只能传输utf8文本，如果要传输二进制数据，请阅读以下部分。

websocket协议中在协议头中使用一个标记位来标记传输的是二进制数据还是utf8文本数据，浏览器会验证标记和传输的内容类型是否符合，如果不符合则会报错断开连接。

所以服务端发送数据的时候需要根据传输的数据类型设置这个标记位，在Workerman中如果是普通utf8文本，则需要设置（默认就是此值，一般不用再手动设置）
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

如果是二进制数据，则需要设置
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**注意**：如果没设置$connection->websocketType，则$connection->websocketType默认为BINARY_TYPE_BLOB（也就是utf8文本类型）。一般应用传输的都是utf8文本，例如传输的是json数据，所以不用手动设置$connection->websocketType。只有在传输二进制数据时（例如图片数据、protobuffer数据等）才要设置此属性为BINARY_TYPE_ARRAYBUFFER。

## 把workerman作为Websocket客户端
可以利用[AsyncTcpConnection](/worker-development/__construct.html)配合[ws协议](/appendices/about-ws.html)让workerman作为websocket客户端连接远程websocket服务端，完成双向实时通讯。




