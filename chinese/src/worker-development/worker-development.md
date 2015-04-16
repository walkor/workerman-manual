# Worker开发适用范围

### Worker说明
Worker是WorkerMan中最基本的功能单元，Worker可以开启多个进程监听端口并使用特定协议通讯。每个Worker进程独立运作，每个Worker进程都能接收无数的客户端连接，并处理这些连接上发来的数据。

### Worker 的进程模型
![workerman master woker模型](http://www.workerman.net/img/Worker.png)

**特点：**

从图上我们可以看出每个Worker维持着各自的客户端连接，能够方便的实现客户端与服务端的实时通讯，基于这种模型我们可以方便实现一些基本的开发需求，例如HTTP服务器、Rpc服务器、一些智能硬件实时上报数据、服务端推送数据等等。

**缺点：**

基于Worker无法方便的实现客户端与可客户端的通讯，例如上图中client①要给client⑤发送数据，由于两个客户端在不同的Worker进程，Worker进程①无法直接给Worker进程②的client⑤发送数据，所以需要我们做一些开发工作，才能实现客户端之间的通讯。

如果需要客户端之间通讯，可以基于WorkerMan的Gateway/Worker开发，见下一章节。

**适用范围：**

其实基于Worker能够开发出几乎所有的网络应用服务端程序，但是如果你需要客户端之间的实时通讯，例如游戏服务器、聊天服务器等，可以直接使用WorkerMan提供的Gateway/Worker开发模式。除此之外，所有的网络服务程序都可以基于Worker开发。



