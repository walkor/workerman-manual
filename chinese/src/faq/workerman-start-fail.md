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



## 现象2
启动后报错类似如下：
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in /home/GatewayWorker/Workerman/Worker.php on line xxxx
```
**关键字：** ```Cannot assign requested address```

**失败原因：**

启动脚本ip参数写错，不是本机ip，请填写本机ip机或者填写 ```0.0.0.0```（表示监听本机所有ip）即可解决

## 现象3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**失败原因：**

stream_socket_server 函数被php.ini禁用

**解决方法**

1、运行```php --ini``` 找到php.ini文件

2、打开php.ini找到disable_functions一项，将stream_socket_server禁用项删掉


