# Channel分布式通讯组件

源码地址：https://github.com/walkor/Channel


Channel是一个分布式通讯组件，用于完成进程间通讯或者服务器间通讯。


## 特点
1、基于订阅发布模型

2、非阻塞式IO

## 原理

Channel包含服务端Channel/Server，和客户端Channel/Client。

Channel/Client通过connect接口连接Channel/Server并建立长连接

Channel/Client通过subscribe接口告诉Channel/Server自己关注（订阅）哪些类型（主题）的消息

Channel/Client通过publish发布某个类型(主题)的消息

Channel/Server收到publish的数据后会分发给对该类型(主题)关注(订阅)的Channel/Client

Channel/Client通过设置onMessage回调，用来接收处理Channel/Server转发来的自己关注(订阅)的某种类型(主题)消息

Channel/Client只会收到自己关注(订阅)的消息，不会收到自己没有关注(订阅)的类型(主题)的消息



## 注意
Channel只能在workerman包括基于workerman的框架(如GatewayWorker等)中使用


