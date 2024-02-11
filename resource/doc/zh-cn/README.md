# 序言

**Workerman，高性能PHP应用容器**

## Workerman是什么？
Workerman是一款纯PHP开发的开源高性能的PHP 应用容器。

Workerman不是重复造轮子，它不是一个MVC框架，而是一个更底层更通用的服务框架，你可以用它开发tcp代理、梯子代理、做游戏服务器、邮件服务器、ftp服务器、甚至开发一个php版本的redis、php版本的数据库、php版本的nginx、php版本的php-fpm等等。Workerman可以说是PHP领域的一次创新，让开发者彻底摆脱了PHP只能做WEB的束缚。

实际上Workerman类似一个PHP版本的nginx，核心也是多进程+Epoll+非阻塞IO。Workerman每个进程能维持上万并发连接。由于本身常驻内存，不依赖Apache、nginx、php-fpm这些容器，拥有超高的性能。同时支持TCP、UDP、UNIXSOCKET，支持长连接，支持Websocket、HTTP、WSS、HTTPS等通讯协议以及各种自定义协议。拥有定时器、异步socket客户端、异步Redis、异步Http、异步消息队列等众多高性能组件。

## Workerman的一些应用方向
Workerman不同于传统MVC框架，Workerman不仅可以用于Web开发，同时还有更广阔的应用领域，例如即时通讯类、物联网、游戏、服务治理、其它服务器或者中间件，这无疑大大提高了PHP开发者的视野。目前这些领域的PHP开发者奇缺，如果想在PHP领域有自己的技术优势，不满足于每天的增删改查工作，或者想向架构师方向或者技术大牛的方向发展，Workerman都是非常值得学习的框架。建议开发者不仅会用，而且能基于Workerman开发出属于自己的开源项目，提升技能增加自己的影响力，比如[Beanbun多进程网络爬虫框架](https://github.com/kiddyuchina/Beanbun)就是一个很好的例子，刚刚上线不久就获得众多好评。

Workerman的一些应用方向如下：

1、即时通讯类
例如网页即时聊天、即时消息推送、微信小程序、手机app消息推送、PC软件消息推送等等
[[示例 workerman-chat聊天室](https://www.workerman.net/workerman-chat) 、 [web消息推送](https://www.workerman.net/web-sender) 、 [小蝌蚪聊天室](https://www.workerman.net/workerman-todpole)]

2、物联网类
例如Workerman与打印机通讯、与单片机通讯、智能手环、智能家居、共享单车等等。
[客户案例如 易联云、易泊时代等]

3、游戏服务器类
例如棋牌游戏、MMORPG游戏等等。[[示例 browserquest-php](https://www.workerman.net/browserquest)]

4、HTTP服务
例如 写高性能HTTP接口、高性能网站。如果想要做HTTP相关的服务或者站点强烈推荐 [webman](https://github.com/walkor/webman)

5、SOA服务化
利用Workerman将现有业务不同功能单元封装起来，以服务的形式对外提供统一的接口，达到系统松耦合、易维护、高可用、易伸缩。[[示例 workerman-json-rpc](https://github.com/walkor/workerman-jsonrpc)、 [workerman-thrift](https://github.com/walkor/workerman-thrift)]

6、其它服务器软件
例如 [GatewayWorker](https://www.workerman.net/doc/gateway-worker)，[PHPSocket.IO](https://www.workerman.net/phpsocket_io)，[http代理](https://github.com/walkor/php-http-proxy)，[sock5代理](https://github.com/walkor/php-socks5)，[分布式通讯组件](https://github.com/walkor/Channel)，[分布式变量共享组件](https://github.com/walkor/GlobalData)，[消息队列](https://github.com/walkor/workerman-queue)、DNS服务器、WebServer、CDN服务器、FTP服务器等等

7、组件
例如[异步redis](components/workerman-redis.md)，[异步http客户端](components/workerman-http-client.md)，[物联网mqtt客户端](components/workerman-mqtt.md)，消息队列 [workerman/redis-queue](components/workerman-redis-queue.md) 、 [workerman/stomp](components/workerman-stomp.md)、[workerman/rabbitmq](components/workerman-rabbitmq.md)  ，[文件监控组件](components/file-monitor.md)，还有很多第三方开发的组件框架等等

显然传统的mvc框架很难实现以上的功能，所以也就是workerman诞生的原因。

## Workerman理念
极简、稳定、高性能、分布式。

### **极简**
小即是美，Workerman内核极简，仅有几个php文件并且只暴露几个接口，学习起来非常简单。所有其它功能通过组件的方式扩展。

Workerman拥有完善的文档+权威的主页+活跃的社区+数个千人QQ群+众多的高性能组件+N多的例子，这一切都让开发者使用起来更得心应手。

### **稳定**
Workerman已经开源数年，被很多上市公司大规模使用，超级稳定。有些服务2年多没重启过仍然在飞速运行。没有coredump、没有内存泄漏、没有bug。

### **高性能**
Workerman因为常驻内存，本身不依赖apache/nginx/php-fpm，没有容器到PHP的通讯开销，没有每个请求初始化一切又销毁一切的开销，具有超高的性能，比起传统的MVC框架，性能要高数十倍，PHP7下通过ab压力测试QPS甚至高于单独的nginx。

### **分布式**
现在早已经不是单枪匹马的时代了，单台服务器性能再强悍也有极限，分布式多服务器部署才是王道。Workerman直接提供了一套长连接分布式通讯方案[GatewayWorker框架](https://doc2.workerman.net)，加服务器只需要简单配置下然后启动即可，业务代码零更改，系统承载能力成倍增加。如果你是开发TCP长连接应用，建议直接用[GatewayWorker](https://doc2.workerman.net)，它是对Workerman的一个包装，针对长连接应用提供了更丰富的接口以及强悍的分布式处理能力。

## 本手册作用范围
Workerman 3.x - 4.x 版本

## windows用户（必读）
workerman同时支持linux系统及windows系统。windows版本workerman本身**不依赖任何扩展**，只需要配置好PHP环境变量即可，**windows版本workerman安装及注意事项参见[windows用户必看](https://www.workerman.net/windows)。**

## 客户端

Workerman的通信协议是开放的，又是可定制的，因此，理论上Workerman可以与使用任意协议的任意平台的客户端进行通信。当用户开发客户端时，可以根据相应的通信协议完成与服务端的通信。



