# 目录结构
<pre>
Workerman                      // workerman内核代码
    ├── Connection                 // socket连接相关
    │   ├── ConnectionInterface.php// socket连接接口
    │   ├── TcpConnection.php      // Tcp连接类
    │   ├── AsyncTcpConnection.php // 异步Tcp连接类
    │   └── UdpConnection.php      // Udp连接类
    ├── Events                     // 网络事件库
    │   ├── EventInterface.php     // 网络事件库接口
    │   ├── Libevent.php           // Libevent网络事件库
    │   ├── Ev.php                 // Libev网络事件库
    │   └── Select.php             // Select网络事件库
    ├── Lib                        // 常用的类库
    │   ├── Constants.php          // 常量定义
    │   └── Timer.php              // 定时器
    ├── Protocols                  // 协议相关
    │   ├── ProtocolInterface.php  // 协议接口类
    │   ├── Http                   // http协议相关
    │   │   └── mime.types         // mime类型
    │   ├── Http.php               // http协议实现
    │   ├── Text.php               // Text协议实现
    │   ├── Frame.php              // Frame协议实现
    │   └── Websocket.php          // websocket协议的实现
    ├── Worker.php                 // Worker
    ├── WebServer.php              // WebServer
    └── Autoloader.php             // 自动加载类
</pre>
