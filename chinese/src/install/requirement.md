# 环境要求

## 运行所需环境

1、WorkerMan 要求运行在Linux环境下（centos、RedHat、Ubuntu、debian等）,也可以运行在mac os下。windows版本及注意事项参见[**这里**](http://www.workerman.net/windows)。

2、安装有PHP-CLI(版本不小于5.4)，并安装了pcntl、posix扩展

3、建议安装event或者libevent扩展，但不是必须的

## 环境检查脚本
可以运行以下脚本检查本地环境是否满足WorkerMan要求

```curl -Ss http://www.workerman.net/check.php | php```

如果脚本中全部提示ok，则代表满足WorkerMan运行环境

## 详细说明

### 关于PHP-CLI

WorkerMan是以[PHP命令行](http://php.net/manual/zh/features.commandline.php)的模式运行的，所以需要安装PHP-CLI。

### 关于WorkerMan依赖的扩展

1、[pcntl扩展](http://cn2.php.net/manual/zh/book.pcntl.php)

pcntl扩展是PHP在Linux环境下进程控制的重要扩展，WorkerMan用到了其[进程创建](http://cn2.php.net/manual/zh/function.pcntl-fork.php)、[信号控制](http://cn2.php.net/manual/zh/function.pcntl-signal.php)、[定时器](http://cn2.php.net/manual/zh/function.pcntl-alarm.php)、[进程状态监控](http://cn2.php.net/manual/zh/function.pcntl-waitpid.php)等特性。此扩展win平台不支持。

2、[posix扩展](http://cn2.php.net/manual/zh/book.posix.php)

posix扩展使得PHP在Linux环境可以调用系统通过[POSIX标准](http://baike.baidu.com/view/209573.htm)提供的接口。WorkerMan主要使用了其相关的接口实现了守护进程化、用户组控制等功能。此扩展win平台不支持。

3、[libevent扩展](http://cn2.php.net/manual/en/book.libevent.php) 或者 [Event扩展](http://php.net/manual/zh/book.event.php)

libevent扩展(或者event扩展)使得PHP可以使用系统[Epoll](http://baike.baidu.com/view/1385104.htm)、Kqueue等高级事件处理机制，能够显著提高WorkerMan在高并发连接时CPU利用率。在高并发长连接相关应用中非常重要。libevent扩展(或者event扩展)不是必须的，如果没安装，则默认使用PHP原生Select事件处理机制。


## 如何安装扩展

参见 [附录-安装扩展](http://doc3.workerman.net/appendices/install-extension.html) 章节


