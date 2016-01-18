# workerman启动失败

## 现象1
启动后报错类似如下：
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in /home/workerman-chat/Workerman/Worker.php on line xxxx

```
**关键字**： ```Address already in use```

**失败原因1：**

workerman已经在运行，无法再次启动。可以运行 ```php start.php restart``` 重新启动

**失败原因2：**

如果restart仍然报```Address already in use```，一般是由于启动脚本找不到主进程pid文件导致的，workerman 3.2.2之前的版本主进程pid文件默认存储在```/tmp/```下，有些系统会定时清理```/tmp/```目录，导致无法启动。

解决方法：

方法1：升级workerman到3.2.2或以上版本，workerman3.2.2 版本不在将pid文件存储在```/tmp/```下，另外提供了```php xxx.php kill``` 命令，方便强行杀死workerman进程。


方法2：可以运行 ```ps aux | grep start.php | awk '{print $2}' | xargs kill -9``` 强行杀死进程。然后参考手册[pidFile](/worker-development/pid_file.html)将pid文件存储在安全的地方。

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


