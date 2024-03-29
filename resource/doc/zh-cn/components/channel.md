# Channel分布式通讯组件
**``` (要求Workerman版本>=3.3.0) ```**

源码地址：https://github.com/walkor/Channel


Channel是一个分布式通讯组件，用于完成进程间通讯或者服务器间通讯。


## 特点
1、基于订阅发布模型

2、非阻塞式IO

## 原理

Channel包含Channel/Server服务端和Channel/Client客户端

Channel/Client通过connect接口连接Channel/Server并保持长连接

Channel/Client通过调用on接口告诉Channel/Server自己关注哪些事件，并注册事件回调函数（回调发生在Channel/Client所在进程中）

Channel/Client通过publish接口向Channel/Server发布某个事件及事件相关的数据

Channel/Server接收事件及数据后会分发给关注这个事件的Channel/Client

Channel/Client收到事件及数据后触发on接口设置的回调

Channel/Client只会收到自己关注事件并触发回调


## 安装

`composer require workerman/channel`

## 注意
Channel只能用在workerman环境，php-fpm环境下无法使用。


