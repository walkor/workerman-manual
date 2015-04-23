# Gateway类的使用

文件位置：GatewayWorker/Gateway.php

Gateway类是基于基础的Worker开发的。可以给Gateway对象的onWorkerStart、onWorkerStop、onConnect、onClose设置回调函数，但是无法给设置onMessage回调。Gateway的onMessage行为固定为将客户端数据转发给Worker。

## Gateway类可以定制的内容

1、 协议

和Worker一样，在初始化Gateway对象时设置Gateway的协议，例如下面设置Gateway的通讯协议为websocket

```php
use \GatewayWorker\Gateway;

// 指定websocket协议
$gateway = new Gateway("websocket://0.0.0.0:8585");

```

2、name

和Worker一样，可以设置Gateway进程的名称，方便status命令中查看统计

3、count

和Worker一样，可以设置Gateway进程的数量，以便充分利用多cpu资源

4、lanIp

lanIp是Gateway所在服务器的内网IP，只有在分布式部署时才需要设置

5、startPort

Gateway进程启动后会监听一个本机端口，用来给BusinessWorker提供链接服务，然后Gateway与BusinessWorker之间就通过这个连接通讯。这里设置的是Gateway监听本机端口的起始端口。比如启动了4个Gateway进程，startPort为2000，则每个Gateway进程分别启动的本地端口一般为2001、2003、2003、2004。

当本机有多个Gateway/BusinessWorker项目时，需要把每个项目的startPort设置成不同的段

6、心跳设置，具体说明见心跳一节

7、onWorkerStart

和Worker一样，可以设置Gateway进程启动后的回调函数，一般在这个回调里面初始化一些全局数据

8、onWorkerStop

和Worker一样，可以设置Gateway进程关闭的回调函数，一般在这个回调里面做数据清理或者保存数据工作

9、onConnect（比较少用到，开发者一般不用关注）

和Worker一样，可以设置onConnect回调，当有客户端连接上来时触发。与Event::onConnect的区别是Event::onConnect运行在BusinessWorker进程上。Gateway::onConnect是运行在Gateway进程上，无法使用\GatewayWorker\Lib\Gateway类提供的接口

10、onClose（比较少用到，开发者一般不用关注）

和Worker一样，可以设置onClose回调，当有客户端连接关闭时触发。同样与Event::onClose的区别是Gateway::onClose是运行在Gateway进程上，无法使用\GatewayWorker\Lib\Gateway类提供的接口
