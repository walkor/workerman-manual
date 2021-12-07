# workerman启动失败

## 现象1
启动后报错类似如下：
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php on line xxxx

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

如果确实没有程序监听这个端口，那么可能是开发者在workerman里设置了两个或两个以上的监听，并且监听的端口相同导致，请开发者自行检查启动脚本是否监听了相同的端口。



## 现象2
启动后报错类似如下：
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php on line xxxx
```
或者
```
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (在其上下文中，该请求的地址无效) in ...workerman/Worker.php on line xxxx
```
**关键字：** `Cannot assign requested address`或者`该请求的地址无效`

**失败原因：**

启动脚本监听ip参数写错，不是本机ip，请填写本机ip机或者填写 ```0.0.0.0```（表示监听本机所有ip）即可解决。

**提示**：Linux系统可以通过命令 ```ifconfig```查看本机所有网卡ip。
如果您是云服务器(阿里云/腾讯云等)用户，注意您的公网ip实际可能是个代理ip(例如阿里云的专有网络)，公网ip并不属于当前的服务器，所以无法通过公网ip监听。虽然不能用公网ip监听，但是仍然可以通过0.0.0.0来绑定。

## 现象3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**失败原因：**

stream_socket_server 函数被php.ini禁用

**解决方法**

1、运行```php --ini``` 找到php.ini文件

2、打开php.ini找到disable_functions一项，将stream_socket_server禁用项删掉

## 现象4
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```
**失败原因**

linux下监听端口如果小于1024，需要root权限。

**解决办法**

使用大于1024的端口或者使用root用户启动服务。


