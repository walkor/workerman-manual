# 原理

### Worker说明
Worker是Workerman中最基本容器，Worker可以开启多个进程监听端口并使用特定协议通讯，类似nginx监听某个端口。每个Worker进程独立运作，采用Epoll(需要装event扩展)+非阻塞IO，每个Worker进程都能上万的客户端连接，并处理这些连接上发来的数据。主进程为了保持稳定性，只负责监控子进程，不负责接收数据也不做任何业务逻辑。

### 客户端与worker进程的关系
![workerman master woker模型](images/Worker.png)


### 主进程与worker子进程关系
![workerman master woker模型](images/Worker2.png)

**特点：**

从图上我们可以看出每个Worker维持着各自的客户端连接，能够方便的实现客户端与服务端的实时通讯，基于这种模型我们可以方便实现一些基本的开发需求，例如HTTP服务器、Rpc服务器、一些智能硬件实时上报数据、服务端推送数据、游戏服务器、微信小程序后台等等。
