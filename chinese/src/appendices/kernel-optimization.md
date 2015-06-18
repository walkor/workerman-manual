# Linux内核调优
##网络
调整linux内核参数以便满足高并发访问，解决大量time_wait占用太多本地端口导致的 ```Cannot assign requested address``` 问题。

客户端与服务端每建立一个连接，客户端一侧都会占用一个本地端口（假设没有启用SO_REUSEADDR选项），本地端口数量是有限制的（默认是```net.ipv4.ip_local_port_range=32768    61000```），也就是说在没设置socket的SO_REUSEADDR选项时，一台Linux服务器作为**客户端**（**```注意是作为客户端```**）默认只能建立大概3万个TCP连接（服务端没有这个限制），可以更改```net.ipv4.ip_local_port_range```增大作为客户端可发起的并发连接数，但最多不会超过65535个（服务端没有这个限制）。

当Linux服务器作为客户端频繁建立TCP短连接时，本地会可能会产生很多TIME_WAIT状态的连接，客户端侧的TIME_WAIT状态的连接会占用一个本地端口直到达到2MSL（最长分解生命期）的时间，这样会导致本地端口被暂时占用，当短连接建立速度过快时（例如做压测时），会导致```Cannot assign requested address```错误，解决办法有几种，比如像下面这样设置端口复用（复用TIME_WAIT状态的连接）。

打开文件 /etc/sysctl.conf，增加以下设置
```shell
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
```
运行 ```sysctl -p```即可生效

**说明：**

/etc/sysctl.conf 可设置的选项很多，其它选项可以根据自己的环境需要进行设置

## 打开文件数

设置系统打开文件数设置，解决高并发下 ```too many open files``` 问题。此选项直接影响单个进程容纳的客户端连接数。

Soft open files 是Linux系统参数，影响系统单个进程能够打开最大的文件句柄数量，这个值会影响到长链接应用如聊天中单个进程能够维持的用户连接数， 运行```ulimit -n```能看到这个参数值，如果是1024，就是代表单个进程只能同时最多只能维持1024甚至更少（因为有其它文件的句柄被打开）。如果开启4个进程维持用户链接，那么整个应用能够同时维持的连接数不会超过4*1024个，也就是说最多只能支持4x1024个用户在线可以增大这个设置以便服务能够维持更多的TCP连接。

Soft open files 修改方法：

（1）ulimit -HSn 102400

这只是在当前终端有效，退出之后，open files 又变为默认值。

（2）将ulimit -HSn 102400写到/etc/profile中，这样每次登录终端时，都会自动执行/etc/profile。

（3）令修改open files的数值永久生效，则必须修改配置文件：/etc/security/limits.conf. 在这个文件后加上：

```
* soft nofile 1024000
* hard nofile 1024000
root soft nofile 1024000
root hard nofile 1024000
```

这种方法需要重启机器才能生效。

