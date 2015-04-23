# 目录结构
<pre>
.
├── start.php            // workerman启动脚本，用于启动applications下的所有应用
├── Applications         // 基于workerman开发的所有应用程序
│   └── Todpole          // 基于workerman开发的小蝌蚪应用程序
│       ├── Config       // 配置相关
│       │   ├── Db.php   // 数据库配置
│       │   └── Store.php// memcache存储配置
│       ├── Event.php    // 业务实现【开发主要关注这个文件】
│       ├── start.php    // 启动脚本，定义监听端口、进程数量等等
│       └── Web          // 小蝌蚪应用自身的Web界面文件
│
├── GatewayWorker            // Gateway/Worker模型公共类库
│   ├── BusinessWorker.php   // Worker业务进程
│   ├── Gateway.php          // Gateway进程
│   └── Lib                  // 类库
│       ├── Context.php      // Gateway与Worker通讯的上下文
│       ├── DbConnection.php // 数据库连接类
│       ├── Db.php           // 数据库连接管理类，对应配置在Applications/YourApp/Config/Db.php
│       ├── Gateway.php      // 与客户端通讯的基础类
│       ├── Lock.php         // 锁相关
│       ├── StoreDriver      // 存储引擎相关
│       │   └── File.php     // 文件存储引擎
│       └── Store.php        // 存储管理类，对应配置在Applications/YourApp/Config/Store.php
└── Workerman                      // workerman内核代码
    ├── Autoloader.php             // 自动加载类
    ├── Connection                 // socket连接相关
    │   ├── ConnectionInterface.php// socket连接接口
    │   ├── TcpConnection.php      // Tcp连接类
    │   ├── AsyncTcpConnection.php // 异步Tcp连接类
    │   └── UdpConnection.php      // Udp连接类
    ├── Events                     // 网络事件库
    │   ├── EventInterface.php     // 网络事件库接口
    │   ├── Libevent.php           // Libevent网络事件库
    │   └── Select.php             // Select网络事件库
    ├── Lib                        // 常用的类库
    │   ├── Constants.php          // 常量定义
    │   └── Timer.php              // 定时器
    ├── Protocols                  // 协议相关
    │   ├── ProtocolInterface.php  // 协议接口类
    │   ├── Http                   // http协议相关
    │   │   └── mime.types         // mime类型
    │   ├── Http.php               // http协议实现
    │   ├── Websocket.php          // websocket协议的实现
    │   └── GatewayProtocol.php    // Gateway/Worker模型中的通讯协议
    ├── WebServer.php              // WebServer
    └── Worker.php                 // Worker
</pre>
