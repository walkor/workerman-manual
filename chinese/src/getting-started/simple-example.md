# 简单的开发实例

## 实例一、使用HTTP协议对外提供Web服务
**创建http_test.php文件**
```php
<?php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

// 创建一个Worker监听2345端口，使用http协议通讯
$http_worker = new Worker("http://0.0.0.0:2345");

// 启动4个进程对外提供服务
$http_worker->count = 4;

// 接收到浏览器发送的数据时回复hello world给浏览器
$http_worker->onMessage = function($connection, $data)
{
    // 向浏览器发送hello world
    $connection->send('hello world');
};

// 运行worker
Worker::runAll();
```

**命令行运行**
```shell
php http_test.php start

```

**测试**


假设服务端ip为127.0.0.1

在浏览器中访问url http://127.0.0.1:2345


## 实例二、使用WebSocket协议对外提供服务
**创建ws_test.php文件**
```php
<?php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

// 创建一个Worker监听2346端口，使用websocket协议通讯
$ws_worker = new Worker("websocket://0.0.0.0:2346");

// 启动4个进程对外提供服务
$ws_worker->count = 4;

// 当收到客户端发来的数据后返回hello $data给客户端
$ws_worker->onMessage = function($connection, $data)
{
    // 向客户端发送hello $data
    $connection->send('hello ' . $data);
};

// 运行worker
Worker::runAll();
```

**命令行运行**
```shell
php ws_test.php start

```

**测试**

打开chrome浏览器，按F12打开调试控制台，在Console一栏输入(或者把下面代码放入到html页面用js运行)

```javascript
// 假设服务端ip为127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2346");
ws.onopen = function() {
    alert("连接成功");
    ws.send('tom');
    alert("给服务端发送一个字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```

## 实例三、直接使用TCP传输数据
**创建tcp_test.php**

```php
<?php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

// 创建一个Worker监听2347端口，不使用任何应用层协议
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// 启动4个进程对外提供服务
$tcp_worker->count = 4;

// 当客户端发来数据时
$tcp_worker->onMessage = function($connection, $data)
{
    // 向客户端发送hello $data
    $connection->send('hello ' . $data);
};

// 运行worker
Worker::runAll();
```

**命令行运行**

```shell
php tcp_test.php start

```

**测试**
```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```
