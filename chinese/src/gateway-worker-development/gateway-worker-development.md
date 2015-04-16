# 适用范围

### Gateway/Worker说明
Gateway/Worker模型是基于基础的Worker模型开发的一套可以实现客户端与客户端实时通讯的架构，主要用于游戏服务器、聊天服务器等需要客户端之间通讯的项目

### Gateway/Worker 的进程模型
![workerman master woker模型](http://www.workerman.net/img/GatewayWorker.png)

**特点：**

从图上我们可以看出Gateway负责接收客户端的连接以及连接上的数据，然后Worker接收Gateway发来的数据做处理，然后再经由Gateway把结果转发给其它客户端。每个客户端都有很多的路由到达另外一个客户端，例如client⑦与client①可以经由蓝色路径完成数据通讯

**优点：**

1、可以方便的实现客户端之间的通讯

2、Gateway与Worker之间是基于socket长连接通讯，也就是说Gateway、Worker可以部署在不同的服务器上，非常容易实现分布式部署，扩容服务器

3、Gateway进程只负责网络IO，业务实现都在Worker进程上，可以reload Worker进程，实现不影用户的情况下实现代码热更新


**适用范围：**

适用于客户端与客户端需要实时通讯的项目。如果只需要客户端与服务端之间的通讯，可以考虑基于Worker模型开发



