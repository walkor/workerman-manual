# 在其它项目中利用WorkerMan给客户端推送数据

**问：**

我有一个普通的web项目，想在这个项目中调用WorkerMan的接口，给客户端推送数据。


**答：**


**基于Workerman可以参考以下连接**

1、[Channel组件推送例子](../components/channel-examples.md)（支持多进程/服务器集群，需要下载Channel组件）

2、[基于Worker推送](https://wenda.workerman.net/?/question/508)(单进程，最简单)



**基于GatewayWorker参考下面连接**

[在其它项目中通过GatewayWorker推送](https://doc2.workerman.net/326107)(支持多进程/服务器集群，支持分组、组播、单个发送)


**基于PHPSocket.IO参考下面连接**

[web消息推送](https://www.workerman.net/web-sender)(默认单进程，基于socket.io，浏览器兼容性最好)
