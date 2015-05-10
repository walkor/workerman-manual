# gateway worker 分离部署

## 什么是Gateway Worker分离部署
Gateway/Worker模式有两组进程，Gateway进程负责网络IO，Worker进程负责业务处理，Gateway与Worker之间使用TCP长连接通讯。当系统出现负载时，一般都是业务进程Worker出现瓶颈。我们可以把Gateway Worker分开部署在不同的服务器上，单独增加Worker服务器提升系统负载能力。同理，如果Gateway进程出现瓶颈，则增加Gateway服务器。

# 部署示例

以Applications/Todpole为例，假如需要部署三台服务器提供高可用服务。瓶颈在worker进程，则可使用1台作为gateway服务器，另外两台做worker服务器。（如果瓶颈在gateway进程（一般是带宽瓶颈），则可以2台gateway机器，1台worker机器，部署方法类似）。


## gateway worker 分离部署扩容步骤
1、首先将进程切分，将Gateway进程部署在一台机器上(假设内网ip为192.168.0.1)，BusinessWorker部署在另外两台机器上（内网ip为192.168.0.2/3）

2、由于192.168.0.1这台机器只部署Gateway进程，所以将该ip上的初始化BusinessWorker示例的地方注释或者删掉，避免运行BusinessWorker进程，例如

打开文件Applications/Todpole/start.php，注释掉bussinessWorker初始化

```php
...
// bussinessWorker
//$worker = new BusinessWorker();
//$worker->name = 'TodpoleBusinessWorker';
//$worker->count = 4;
...
```

3、配置Gateway服务器(192.168.0.1)上的Gateway实例的```lanIp=192.168.0.1```与本机ip一致，Gateway服务器的初始化文件最终类似下面配置(如果有单独的Web服务器运行蝌蚪界面，可以把WebServer初始化部分也去掉)

文件Applications/Todpole/start.php
```php
<?php
use \Workerman\WebServer;
use \GatewayWorker\Gateway;
use \GatewayWorker\BusinessWorker;

// gateway
$gateway = new Gateway("Websocket://0.0.0.0:8282");
$gateway->name = 'TodpoleGateway';
$gateway->count = 4;
$gateway->lanIp = '192.168.0.1';
$gateway->startPort = 2000;
$gateway->pingInterval = 10;
$gateway->pingData = '{"type":"ping"}';

// bussinessWorker
//$worker = new BusinessWorker();
//$worker->name = 'TodpoleBusinessWorker';
//$worker->count = 4;

// WebServer
$web = new WebServer("http://0.0.0.0:8383");
$web->count = 12;
$web->addRoot('kedou.workerman.net', __DIR__.'/Web');
```

3、由于192.168.0.2/3 两台服务器只部署BusinessWorker进程，所以将这两台ip上的Gateway初始化注释掉或者删掉，避免运行Gateway进程，BusinessWorker服务器初始化文件类似下面(如果有单独的Web服务器运行蝌蚪界面，可以把WebServer初始化部分也去掉)

```php
<?php
use \Workerman\WebServer;
use \GatewayWorker\Gateway;
use \GatewayWorker\BusinessWorker;

// gateway
//$gateway = new Gateway("Websocket://0.0.0.0:8282");
//$gateway->name = 'TodpoleGateway';
//$gateway->count = 4;
//$gateway->lanIp = '192.168.0.1';
//$gateway->startPort = 2000;
//$gateway->pingInterval = 10;
//$gateway->pingData = '{"type":"ping"}';

// bussinessWorker
$worker = new BusinessWorker();
$worker->name = 'TodpoleBusinessWorker';
$worker->count = 4;

// WebServer
$web = new WebServer("http://0.0.0.0:8383");
$web->count = 12;
$web->addRoot('kedou.workerman.net', __DIR__.'/Web');
```

4、由于物理机之间需要共享一些数据，需要部署一台redis服务器，假设部署在Gateway（192.168.0.1）这台机器上，redis服务端口为6379

5、给三台服务器的PHP添加redisd或者redis扩展。推荐用redisd扩展，ubuntu/debian可使用 ```sudo apt-get install php5-redis```安装；centos系统使用```yum install php-pecl-redis``` 安装。

6、配置redis，更改三台服务器上```Applications/Todpole/Config/Store.php```中的```driver```、```gateway```两项配置如下，

```php
// 存储驱动改为redis
public static $driver = self::DRIVER_REDIS
// 更改redis ip和端口
public static $gateway = array(
     '192.168.0.1:6379',
);

```

7、首先启动Gateway服务器192.168.0.1，然后启动BusinessWorker的服务器192.168.0.2/3

*至此，WorkerMan分布式部署完毕。*

## 一些问题及解答

### 为什么将Gateway与BusinesWorker分别部署在不同的服务器上？
首先说明的是不一定非要将Gateway BusinessWorker分开部署，但是推荐分开部署，原因如下：

1、由于Gateway只负责网络IO，只要服务器带宽够用，绝大多数情况下Gateway服务器不会成为瓶颈，所以在很长时间我们只需要一台或者少数几台Gateway服务器即可。由于我们不想BusinessWorker影响到Gateway，所以将Gateway和BusinessWorker分开部署

2、BusinessWorker主要负责业务逻辑。当请求量增大时，由于可能BusinessWorker业务比较复杂，负载可能会明显升高，这时我们只要单纯增加BusinessWorker服务器即可，Gateway服务器则一般不需要变动，也就是不用通知客户端Gateway的ip有所变动

3、当系统BusinessWorker负载较低，需要下线服务器时，我们只需要下线BusinessWorker服务器即可，无需变动GateWay服务器，也就不会导致客户端链接因为服务器下线而断开。


### 当BusinessWorker服务器集群负载较低时，需要下线一些机器怎么实施?
只需要停止BusinessWorker的服务，运行```php start.php  stop```，然后下线即可。Gateway服务器会自动感知有BusinessWorker服务器下线，不会再将请求转发给下线的机器，整个下线过程中不影响服务质量。

### 当Gateway服务器集群负载较低时，需要下线一些机器怎么实施?
*首先还是要说明下Gateway服务器一般情况下不会成为系统瓶颈，所以一般你很长时间内Gateway服务器数量是一个稳定的值，一般一台即可*

下线Gateway服务器，首先停止服务，运行```php start.php stop```，此时会导致该服务器上已有的客户端链接断开，然后下线服务器即可。此时BusinessWorker会感知到有Gateway服务器下线，会自动断开与Gateway进程的联系。
