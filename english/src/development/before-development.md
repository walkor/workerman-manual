# 开发前必读

使用WorkerMan开发应用，你需要了解以下内容：


## 一、WorkerMan开发与普通PHP开发的不同之处

除了与HTTP协议相关的变量函数无法直接使用外，WorkerMan开发与普通PHP开发并没有很大不同。

### 1、应用层协议不同
* 普通PHP开发一般是基于HTTP应用层协议，WebServer已经帮开发者完成了协议的解析
* WorkerMan支持各种协议，目前内置了HTTP、WebSocket等协议。WorkerMan非常推荐开发者使用更简单的自定义协议通讯

*由于非HTTP协议的应用，所以```header()``` ```setcookie()``` ```session_start```等函数无法直接使用，需要使用WorkerMan提供的方法*


### 2、请求周期差异
* PHP在Web应用中一次请求过后会释放所有的变量与资源
* WorkerMan开发的应用程序在第一次载入解析后便常驻内存，使得类的定义、全局对象、类的静态成员不会释放，便于后续重复利用

### 3、注意避免类和常量的重复定义
* 由于WorkerMan会缓存编译后的PHP文件，所以要避免多次require/include相同的类或者常量的定义文件。建议使用require_once/include_once加载文件。

### 4、注意单例模式的链接资源的释放
* 由于WorkerMan不会在每次请求后释放全局对象及类的静态成员，在数据库等单例模式中，往往会将数据库实例（内部包含了一个数据库socket链接）保存在数据库静态成员中，使得WorkerMan在进程生命周期内都复用这个数据库socket链接。需要注意的是当数据库服务器发现某个链接在一定时间内没有活动后可能会主动关闭socket链接，这时再次使用这个数据库实例时会报错，（错误信息类似mysql gone away）。WorkerMan提供了数据库类，有断开重连的功能，开发者可以直接使用。

### 5、注意不要使用exit、die出语句
* WorkerMan运行在PHP命令行模式下，当调用exit、die退出语句时，会导致当前进程退出。虽然子进程退出后会立刻重新创建一个的相同的子进程继续服务，但是还是可能对业务产生影响。


## 二、需要了解的基本概念

### 1、TCP传输层协议
TCP是一种面向连接的、可靠的、基于IP的传输层协议。TCP传输层协议一个重要特点是TCP是基于数据流的，客户端的请求会源源不断的发送给服务端，服务端收到的数据可能不是一个完整的请求，也有可能是多个请求连在一起。这就需要我们在这源源不断的数据流中区分每个请求的边界。而应用层协议主要是为请求边界定义一套规则，避免请求数据混乱。

### 2、应用层协议

应用层协议(application layer protocol)定义了运行在不同端系统上（客户端、服务端）的应用程序进程如何相互传递报文，例如HTTP、WebSocket都属于应用层协议。例如一个简单的应用层次协议可以如下```{"module":"user","action":"getInfo","uid":456}\n"```。此协议是以```"\n"```（注意这里```"\n"```代表的是回车）标记请求结束，消息体是字符串。

### 3、短连接

短连接是指通讯双方有数据交互时，就建立一个连接，数据发送完成后，则断开此连接，即每次连接只完成一项业务的发送。像WEB网站的HTTP服务一般都用短链接。

*短链接应用程序开发可以参考基本开发流程一章*


### 4、长连接

长连接，指在一个连接上可以连续发送多个数据包

长连接多用于操作频繁，点对点的通讯的情况。每个TCP连接都需要三步握手，这需要时间，如果每个操作都是先连接，再操作的话那么处理速度会降低很多。所以长连接在每个操作完后都不断开，下次处理时直接发送数据包就OK了，不用建立TCP连接。例如：数据库的连接用长连接，如果用短连接频繁的通信会造成socket错误，而且频繁的socket 创建也是对资源的浪费。

*当需要主动向客户端推送数据时，例如聊天类、即时游戏类、手机推送等应用需要长连接。*
*长链接应用程序开发可以参考Gateway/Worker开发流程*


### 5、平滑重启

一般的重启的过程是把所有进程全部停止后，再开始创建全新的服务进程。在这个过程中会有一个短暂的时间内是没有进程对外提供服务的，这就会导致服务暂时不可用，这在高并发时势必会导致请求失败。

而平滑重启则不是一次性的停止所有进程，而是一个进程一个进程的停止，每停止一个进程后马上重新创建一个新的进程顶替，直到所有旧的进程都被替换为止。

平滑重启WorkerMan可以使用 ```php your_file.php reload```命令，能够做到在不影响服务质量的情况下更新应用程序

## 三、两种开发模式
在WorkerMan中有两种开发模式

#### 1、基于Worker开发

即直接使用Worker类来开发，例如下面的代码
```php
require_once './Workerman/Autoloader.php';
use Workerman\Worker;

// 创建一个Worker监听2347端口，不使用任何应用层协议
$tcp_worker = new Worker("tcp://0.0.0.0:2347");
// 当客户端发来数据时
$tcp_worker->onMessage = function($connection, $data)
{
    // 向客户端发送hello $data
    $connection->send('hello ' . $data);
};
```

Worker是WorkerMan中最基本的功能单元，提供了接收并维护大量客户端连接的能力，同时也可以做相应的业务逻辑。基于Worker能开发出各种复杂的进程模型，以满足各种应用需求。例如下面讲到的Gateway/Worker进程模型也是基于Worker开发的。

#### 2、基于Gateway/Worker开发
Gateway/Worker也是基于Worker开发的，这种进程模型分为两组进程，Gateway进程和Worker进程，其中Gateway进程只负责网络IO，Worker进程只负责处理业务逻辑。所有客户端与Gateway进程开放的端口建立连接发送及接收数据，Gateway进程将接收到的数据交给Worker进程处理，Worker处理过程中如果需要给其它客户端发送数据，则将数据交给Gateway转发。

Gateway/Worker模型非常适合游戏类、即时IM、聊天室等客户端与客户端需要即时通讯的业务。

下面是基于Gateway/Worker开发小蝌蚪服务端代码片段

```php
use \Workerman\WebServer;
use \GatewayWorker\Gateway;
use \GatewayWorker\BusinessWorker;

require_once __DIR__ . '/../../Workerman/Autoloader.php';

// gateway
$gateway = new Gateway("websocket://0.0.0.0:8585");

$gateway->name = 'TodpoleGateway';

$gateway->count = 4;

$gateway->reloadable = false;

$gateway->lanIp = '127.0.0.1';

$gateway->startPort = 4000;



// bussinessWorker
$worker = new BusinessWorker();

$worker->name = 'TodpoleBusinessWorker';

$worker->count = 4;

```


