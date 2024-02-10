# 停止失败

## 现象：
运行 ```php start.php stop``` 提示 ```stop fail```

### 第一种可能性
前提是以debug方式启动的workerman，开发者在终端按了```ctrl z```给workerman发送了```SIGSTOP```信号，导致workerman进入后台并挂起(暂停)，所以无法响应stop命令(```SIGINT```信号)。

**解决：**
在启动workerman的终端输入```fg```(发送```SIGCONT```信号)然后回车，将workerman切回前台运行，按```ctrl c```(发送```SIGINT```信号)停止workerman。

如果无法停止，尝试运行以下两条命令
```
killall -9 php
```
```
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```
### 第二种可能性
运行stop的用户和workerman启动用户不一致，即stop用户没有权限停止workerman。

**解决：**
切换到启动workerman的用户，或者用权限更高的用户停止workerman。



### 第三种可能性
保存workerman主进程pid文件被删除，导致脚本找不到pid进程，导致停止失败。

**解决：**
将pid文件保存到安全的位置，参见手册[Worker::$pidFile](../worker/pid-file.md)。



### 第四种可能性
workerman主进程pid文件对应的进程不是workerman进程。

**解决：**
打开workerman的主进程的pid文件查看主进程pid，pid文件默认在Workerman平行的目录里。运行命令 ```ps aux | grep 主进程pid``` 查看对应的进程是否是Workerman进程，如果不是，可能是服务器重启过，导致workerman保存的pid是过期的pid，而这个pid刚好被其它进程使用，导致停止失败。如果是这种情况，将pid文件删除即可。

### 第五种可能性
安装了grpc扩展，但是没有给grpc扩展设置相应的环境变量，启动后会多fork出一个挂载进程，停止时导致失败。

