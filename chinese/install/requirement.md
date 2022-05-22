# 环境要求

## Windows用户
workerman从3.5.3版本开始已经能够同时支持linux系统和windows系统。

1、需要PHP>=5.4，并配置好PHP的环境变量。

2、Windows版本的Workerman不依赖任何扩展。

3、安装使用以及使用限制[**这里**](https://www.workerman.net/windows)。

4、由于Workerman在Windows下有诸多使用限制，所以正式环境建议用Linux系统，windows系统仅建议用于开发环境。

 ``` ====本页面以下只适用于Linux用户，Windows用户请忽略。 ====```


## Linux用户(含Mac OS)
Linux用户只能使用Linux版本的Workerman。

1、安装PHP>=5.4，并安装了pcntl、posix扩展

2、建议安装event扩展，但不是必须的（注意event扩展需要PHP>=5.4）

### Linux环境检查脚本
Linux用户可以运行以下脚本检查本地环境是否满足WorkerMan要求

 ```curl -Ss https://www.workerman.net/check | php```

如果脚本中全部提示ok，则代表满足WorkerMan运行环境

（注意：检测脚本中没有检测event扩展，如果并发连接数大于1024建议安装event扩展，安装方法参见下一节）

## 详细说明

### 关于PHP-CLI

WorkerMan是基于[PHP命令行(PHP-CLI)](https://php.net/manual/zh/features.commandline.php)模式运行的。PHP-CLI与PHP-FPM或者Apache的MOD-PHP是独立的可执行程序，它们之间并不冲突也不会有相互依赖，完全独立。

### 关于WorkerMan依赖的扩展

1、[pcntl扩展](https://cn2.php.net/manual/zh/book.pcntl.php)

pcntl扩展是PHP在Linux环境下进程控制的重要扩展，WorkerMan用到了其[进程创建](https://cn2.php.net/manual/zh/function.pcntl-fork.php)、[信号控制](https://cn2.php.net/manual/zh/function.pcntl-signal.php)、[定时器](https://cn2.php.net/manual/zh/function.pcntl-alarm.php)、[进程状态监控](https://cn2.php.net/manual/zh/function.pcntl-waitpid.php)等特性。此扩展win平台不支持。

2、[posix扩展](https://cn2.php.net/manual/zh/book.posix.php)

posix扩展使得PHP在Linux环境可以调用系统通过[POSIX标准](https://baike.baidu.com/view/209573.htm)提供的接口。WorkerMan主要使用了其相关的接口实现了守护进程化、用户组控制等功能。此扩展win平台不支持。

3、 [Event扩展](https://php.net/manual/zh/book.event.php) 或者 [libevent扩展](https://cn2.php.net/manual/en/book.libevent.php) 

event扩展使得PHP可以使用系统[Epoll](https://baike.baidu.com/view/1385104.htm)、Kqueue等高级事件处理机制，能够显著提高WorkerMan在高并发连接时CPU利用率。在高并发长连接相关应用中非常重要。libevent扩展(或者event扩展)不是必须的，如果没安装，则默认使用PHP原生Select事件处理机制。


## 如何安装扩展

参见[安装扩展](../appendices/install-extension.md)章节


