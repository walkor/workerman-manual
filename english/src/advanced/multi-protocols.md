# 多协议支持
## 需求
有时我们需要一套应用程序支持多个客户端，进而需要应用支持多个协议。例如一个IM即时通讯应用，可能需要同时支持浏览器使用，又要支持移动App客户端。而二者所使用的协议可能完全不同。

## 如何支持多协议
在WorkerMan中最简单的实现方法是开启多个端口，每个端口使用一种协议。不同客户端使用各自的协议去连特定的端口。

## 示例（小蝌蚪）
[小蝌蚪](https://github.com/walkor/workerman-todpole)应用程序是运行在PC浏览器里面的，使用Websocket协议与WorkerMan通讯，当我们需要把它移植到手机App上却有没有合适的客户端Websocket库时，我们可以使用更简单的协议来实现App与WorkerMan通讯，例如Text文本协议(以换行符为结尾的文本数据包)。

下面是开启多端口支持多协议示例

```php
use \Workerman\WebServer;
use \GatewayWorker\Gateway;
use \GatewayWorker\BusinessWorker;

// gateway 原有代码
$gateway = new Gateway("Websocket://0.0.0.0:8282");
$gateway->name = 'TodpoleGateway';
$gateway->count = 4;
$gateway->lanIp = '127.0.0.1';
$gateway->startPort = 2000;
$gateway->pingInterval = 10;
$gateway->pingData = '{"type":"ping"}';

// ##########新增端口支持Text协议 开始##########
// 新增8283端口，开启Text文本协议
$gateway_text = new Gateway("Text://0.0.0.0:8283");
// 进程名称，主要是status时方便识别
$gateway_text->name = 'TodpoleGatewayText';
// 开启多少text协议的gateway进程
$gateway_text->count = 4;
// 本机ip（分布式部署时需要设置成内网ip）
$gateway_text->lanIp = '127.0.0.1';
// gateway内部通讯起始端口，起始端口不要重复
$gateway_text->startPort = 2500;
// 也可以设置心跳，这里省略
// ##########新增端口支持Text协议 结束##########

// bussinessWorker 原有代码
$worker = new BusinessWorker();
$worker->name = 'TodpoleBusinessWorker';
$worker->count = 4;

// WebServer 原有代码
$web = new WebServer("http://0.0.0.0:8383");
$web->count = 2;
$web->addRoot('www.workerman.net', __DIR__.'/Web');

```

**测试效果**

由于是文本协议，我们可以通过telnet命令方便的模拟文本协议客户端。以下运行telnet命令的结果

```shell
telnet 127.0.0.1 8283
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
{"type":"update","id":156,"angle":3.636,"momentum":0,"x":-64.8,"y":147.1,"life":1,"name":"Guest.156","authorized":false}
{"type":"update","id":156,"angle":4.27,"momentum":0,"x":-64.8,"y":147.1,"life":1,"name":"Guest.156","authorized":false}
{"type":"update","id":156,"angle":5.766,"momentum":0,"x":-64.8,"y":147.1,"life":1,"name":"Guest.156","authorized":false}
{"type":"update","id":156,"angle":6.284,"momentum":3,"x":-58.8,"y":146.7,"life":1,"name":"Guest.156","authorized":false}
```
我们能看到其它PC客户端通过WorkerMan转发来的蝌蚪的实时坐标数据，我们也可以输入自己的坐标数据，然后按回车键，我们就能在PC客户端上看到自己了。这样通过使用telnet客户端+文本协议，我们可以方便的调试数据，开发新的客户端了。


**说明：**

以上是WorkerMan多协议支持示例，我们看到只需要简单的初始化端口及协议即可，服务端的业务代码不用任何更改。开发者也可以使用其它协议初始化端口，也可以参考《定制通讯协议章节》定义自己的协议

以上是Gateway/Worker模型的多协议支持示例，基于Worker的多协议也是同样的道理

支持多协议还有其他的方法，比如通过协议自身的特点区分当前是哪种协议，然后分别调用相应协议的解码方法，这样可以做到只开一个端口就可以支持多种协议的效果


