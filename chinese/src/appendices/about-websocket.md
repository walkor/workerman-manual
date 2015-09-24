# WebSocket协议

WebSocket protocol 是HTML5一种新的协议。它实现了浏览器与服务器全双工通信

## WebSocket与TCP关系

WebSocket和HTTP一样是一种应用层协议，都是基于TCP传输的，WebSocket本身和Socket并没有多大关系，更不能等同。

## WebSocket协议握手
WebSocket协议有一个握手的过程，握手时浏览器和服务端是以HTTP协议通信的，在Workerman中可以这样介入到握手过程。

```php
$ws = new Worker('Websocket://0.0.0.0:8181');
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



