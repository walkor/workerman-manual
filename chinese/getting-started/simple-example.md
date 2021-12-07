# 简单的开发实例

## 安装
**设置镜像**
由于国内访问composer比较慢，建议设置阿里云composer镜像，运行如下命令设置阿里云代理

`composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/`

**安装workerman**
在一个空目录中运行
`composer require workerman/workerman`

## 实例一、使用HTTP协议对外提供Web服务
**创建start.php文件**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// 创建一个Worker监听2345端口，使用http协议通讯
$http_worker = new Worker("http://0.0.0.0:2345");

// 启动4个进程对外提供服务
$http_worker->count = 4;

// 接收到浏览器发送的数据时回复hello world给浏览器
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // 向浏览器发送hello world
    $connection->send('hello world');
};

// 运行worker
Worker::runAll();
```

**命令行运行（windows用户用 [cmd命令行](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn)，下同）**
```shell
php start.php start

```

**测试**


假设服务端ip为127.0.0.1

在浏览器中访问url http://127.0.0.1:2345

 **注意：**

1、如果出现无法访问的情况，请参照[客户端连接失败原因](../faq/client-connect-fail.md)一节排查。

2、服务端是http协议，只能用http协议通讯，用websocket等其它协议无法直接通讯。


## 实例二、使用WebSocket协议对外提供服务
**创建ws_test.php文件**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 注意：这里与上个例子不同，使用的是websocket协议
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// 启动4个进程对外提供服务
$ws_worker->count = 4;

// 当收到客户端发来的数据后返回hello $data给客户端
$ws_worker->onMessage = function(TcpConnection $connection, $data)
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
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("连接成功");
    ws.send('tom');
    alert("给服务端发送一个字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```

  **注意：**

1、如果出现无法访问的情况，请参照[手册常见问题-连接失败](../faq/client-connect-fail.md)一节排查。

2、服务端是websocket协议，只能用websocket协议通讯，用http等其它协议无法直接通讯。 

## 实例三、直接使用TCP传输数据
**创建tcp_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 创建一个Worker监听2347端口，不使用任何应用层协议
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// 启动4个进程对外提供服务
$tcp_worker->count = 4;

// 当客户端发来数据时
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
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

**测试：命令行运行**
(以下是linux命令行效果，与windows下效果有所不同)
```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```

**注意：**

1、如果出现无法访问的情况，请参照[手册常见问题-连接失败](../faq/client-connect-fail.md)一节排查。

2、服务端是裸tcp协议，用websoket、http等其它协议无法直接通讯。

