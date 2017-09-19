# 原理
# Worker开发适用范围

### Worker说明
Worker是WorkerMan中最基本的功能单元，Worker可以开启多个进程监听端口并使用特定协议通讯。每个Worker进程独立运作，每个Worker进程都能上万的客户端连接，并处理这些连接上发来的数据。主进程只负责监控子进程，不负责接收数据。

### 客户端与worker进程的关系
![workerman master woker模型](http://www.workerman.net/img/Worker.png)


### 主进程与worker子进程关系
![workerman master woker模型](http://wenda.workerman.net/uploads/answer/20140815/5670ea17653a1a6e6811ed5148f77c96.png)

**特点：**

从图上我们可以看出每个Worker维持着各自的客户端连接，能够方便的实现客户端与服务端的实时通讯，基于这种模型我们可以方便实现一些基本的开发需求，例如HTTP服务器、Rpc服务器、一些智能硬件实时上报数据、服务端推送数据、游戏服务器等等。

**端口复用(reusePort)：**

版本要求：

* workerman: >= 3.2.1 
* php: >= php7.0
* Linux 内核: >= 3.9

设置为此参数为true后，不在由主进程监听端口，而是由子进程监听相同端口，由操作系统决定将socket交给哪个进程处理，避免了惊群效应，可以提升多进程短连接应用的性能。

详情参考 [reUsePort的设置](http://doc3.workerman.net/worker-development/reuse-port.html)。

