# workerman启动失败

## 现象1
启动后报错类似如下：
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in /home/workerman-chat/Workerman/Worker.php on line xxxx

```
**关键字**： ```Address already in use```

**失败原因：**

端口被占用，无法启动。

可以通过命令```netstat -anp | grep 端口号```来找出哪个程序占用了端口。
然后停止对应的程序释放端口解决。


如果不能停止对应端口的程序，可以通过更换workerman的端口解决。

如果是Workerman占用的端口，又无法通过stop命令停止(一般是丢失pid文件或者主进程被开发者kill了导致)，可以通过运行以下两个命令杀死Workerman进程。

```
killall php
ps aux|grep WorkerMan|awk '{print $2}'|xargs kill -9
```



## 现象2
启动后报错类似如下：
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in /home/GatewayWorker/Workerman/Worker.php on line xxxx
```
**关键字：** ```Cannot assign requested address```

**失败原因：**

启动脚本ip参数写错，不是本机ip，请填写本机ip机或者填写 ```0.0.0.0```（表示监听本机所有ip）即可解决。

**提示**：Linux系统可以通过命令 ```ifconfig```查看本机所有网卡ip。<br>
如果您是腾讯云用户，注意您的公网ip实际是代理服务器ip，公网ip并不属于你的服务器，所以无法通过公网ip绑定，但是可以通过0.0.0.0来绑定。

## 现象3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**失败原因：**

stream_socket_server 函数被php.ini禁用

**解决方法**

1、运行```php --ini``` 找到php.ini文件

2、打开php.ini找到disable_functions一项，将stream_socket_server禁用项删掉


