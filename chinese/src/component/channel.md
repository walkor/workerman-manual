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


## 下载安装
可以使用composer安装，或者直接下载zip包https://github.com/walkor/Channel/archive/master.zip 。

文件目录可以根据需要放在任意位置，使用时能够require到文件src/Server.php 和src/Client.php 两个文件即可。注意require路径最好使用绝对路径。具体使用方法参考下面章节channelServer和channelClient的说明。


## 注意
Channel只能用在workerman环境，php-fpm环境下无法使用。


