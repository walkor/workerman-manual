# Linux内核调优

打开文件 /etc/sysctl.conf，增加以下设置
```
#该参数设置系统的TIME_WAIT的数量，如果超过默认值则会被立即清除
net.ipv4.tcp_max_tw_buckets = 20000
#定义了系统中每一个端口最大的监听队列的长度，这是个全局的参数
net.core.somaxconn = 65535
#对于还未获得对方确认的连接请求，可保存在队列中的最大数目
net.ipv4.tcp_max_syn_backlog = 262144
#在每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目
net.core.netdev_max_backlog = 30000
#能够更快地回收TIME-WAIT套接字。此选项会导致处于NAT网络的客户端超时，建议为0
net.ipv4.tcp_tw_recycle = 0
#系统所有进程一共可以打开的文件数量
fs.file-max = 6815744
#防火墙跟踪表的大小。注意：如果防火墙没开则会提示error: "net.netfilter.nf_conntrack_max" is an unknown key，忽略即可
net.netfilter.nf_conntrack_max = 2621440
```
运行 ```sysctl -p```即可生效。

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

