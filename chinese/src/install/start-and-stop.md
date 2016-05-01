# 启动与停止

注意Workerman启动停止等命令都是在命令行中完成的。

要启动Workerman，首先需要有一个启动入口文件，里面定义了服务监听的端口及协议。可以参考[入门指引--简单开发实例部分](/getting-started/simple-example.html)


这里以[workerman-chat](https://github.com/walkor/workerman-chat)为例，它的启动入口为start.php。
### 启动

以debug（调试）方式启动

```php start.php start```

以daemon（守护进程）方式启动

```php start.php start -d```

### 停止
```php start.php stop```

### 重启
```php start.php restart```

### 平滑重启
```php start.php reload```

### 查看状态
```php start.php status```

### 强行杀死所有workerman进程

``` （要求workerman版本>=3.2.2） ```

```php start.php kill```

## debug和daemon方式区别

1、以debug方式启动，代码中echo、var_dump、print等打印函数会直接输出在终端。

2、以daemon方式启动，代码中echo、var_dump、print等打印会默认重定向到/dev/null文件，可以通过设置```Worker::$stdoutFile = '/your/path/file';```来设置这个文件路径。

3、以debug方式启动，终端关闭后workerman会随之关闭并退出。

4、以daemon方式启动，终端关闭后workerman继续后台正常运行。



## 什么是平滑重启？

平滑重启不同于普通的重启，平滑重启可以做到在不影响用户的情况下重启服务，以便重新载入PHP程序，完成业务代码更新。

平滑重启一般应用于业务更新或者版本发布过程中，能够避免因为代码发布重启服务导致的暂时性服务不可用的影响。

**注意：只有在on{...}回调中载入的文件平滑重启后才会自动更新，启动脚本中直接载入的文件或者写死的代码运行reload不会自动更新。**

## 平滑重启原理

WorkerMan分为主进程和子进程，主进程负责监控子进程，子进程负责接收客户端的连接和连接上发来的请求数据，做相应的处理并返回数据给客户端。当业务代码更新时，其实我们只要更新子进程，便可以达到更新代码的目的。

当WorkerMan主进程收到平滑重启信号时，主进程会向其中一个子进程发送安全退出(让对应进程处理完毕当前请求后才退出)信号，当这个进程退出后，主进程会重新创建一个新的子进程（这个子进程载入了新的PHP代码），然后主进程再次向另外一个旧的进程发送停止命令，这样一个进程一个进程的重启，直到所有旧的进程全部被置换为止。

我们看到平滑重启实际上是让旧的业务进程逐个退出然后并逐个创建新的进程做到的。为了在平滑重启时不影响客用户，这就要求进程中不要保存用户相关的状态信息，即业务进程最好是无状态的，避免由于进程退出导致信息丢失。


