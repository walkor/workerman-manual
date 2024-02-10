# 查看运行状态

运行 ```php start.php status```
可以查看到WorkerMan的运行状态，类似如下:

```
----------------------------------------------GLOBAL STATUS----------------------------------------------------
Workerman version:3.5.13          PHP version:5.5.9-1ubuntu4.24
start time:2018-02-03 11:48:20   run 112 days 2 hours   
load average: 0, 0, 0            event-loop:\Workerman\Events\Event
4 workers       11 processes
worker_name        exit_status      exit_count
ChatBusinessWorker 0                0
ChatGateway        0                0
Register           0                0
WebServer          0                0
----------------------------------------------PROCESS STATUS---------------------------------------------------
pid	memory  listening                worker_name        connections send_fail timers  total_request qps    status
18306	2.25M   none                     ChatBusinessWorker 5           0         0       11            0      [idle]
18307	2.25M   none                     ChatBusinessWorker 5           0         0       8             0      [idle]
18308	2.25M   none                     ChatBusinessWorker 5           0         0       3             0      [idle]
18309	2.25M   none                     ChatBusinessWorker 5           0         0       14            0      [idle]
18310	2M      websocket://0.0.0.0:7272 ChatGateway        8           0         1       31            0      [idle]
18311	2M      websocket://0.0.0.0:7272 ChatGateway        7           0         1       26            0      [idle]
18312	2M      websocket://0.0.0.0:7272 ChatGateway        6           0         1       21            0      [idle]
18313	1.75M   websocket://0.0.0.0:7272 ChatGateway        5           0         1       16            0      [idle]
18314	1.75M   text://0.0.0.0:1236      Register           8           0         0       8             0      [idle]
18315	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [idle]
18316	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [idle]
----------------------------------------------PROCESS STATUS---------------------------------------------------
Summary	18M     -                        -                  54          0         4       138           0      [Summary]
```

## 说明

### GLOBAL STATUS

从这以栏中我们可以看到

WorkerMan的版本```version:3.5.13```

启动时间 ```2018-02-03 11:48:20```，运行了```run 112 days 2 hours ```

服务器负载 ```load average: 0, 0, 0```，分别是最近1分钟、5分钟、15分钟内系统的平均负载

使用的IO事件库，```event-loop:\Workerman\Events\Event```

 ```4 workers```（3种进程，包括ChatGateway、ChatBusinessWorker、Register进程、WebServer进程）

 ``` 11 processes ```(共11个进程)

 ``` worker_name ```(worker进程名)

 ``` exit_status ```（worker进程退出状态码）

 ``` exit_count ```（该状态码的退出次数）


一般来说exit_status为0表示为正常退出，如果为其它值，代表进程是异常退出的，并产生一条类似```WORKER EXIT UNEXPECTED```错误信息，错误信息会记录到[Worker::logFile](worker/log-file.md)指定的文件中。

**常见的exit_status及其含义如下：**

* 0：表示正常退出，运行reload平滑重启后会出现值为0的退出码，是正常现象。注意在程序中调用exit或die也会导致退出码为0，并产生一条```WORKER EXIT UNEXPECTED```错误信息，workerman中不允许业务代码调用exit或者die语句。
* 9：表示进程被SIGKILL信号杀死了。这个退出码主要发生在stop以及reload平滑重启时，导致这个退出码的原因是由于子进程没有在规定时间内响应主进程reload信号(例如mysql、curl等长时间阻塞等待或者业务死循环等)，被主进程强制使用SIGKILL信号杀死。注意，当在linux命令行中使用kill命令发送SIGKILL信号给子进程也会导致这个退出码。
* 11：表示php发生了coredump，一般是使用了不稳定的扩展导致，请在php.ini中把对应扩展注释掉；另外有少数情况是php的bug，这时需要升级php 
* 65280：导致这个退出码的原因是业务代码有致命错误，例如调用了不存在的函数、语法错误等，具体错误信息会记录到[Worker::logFile](worker/log-file.md)指定的文件中，也可以在[php.ini](https://php.net/manual/zh/ini.list.php)中[error_log](https://php.net/manual/zh/errorfunc.configuration.php#ini.error-log)指定的文件中(如果有指定的话)找到。
* 64000：导致这个退出码的原因是业务代码抛出了异常，但业务没有捕获这个异常，导致进程退出。如果workerman以debug方式运行时异常调用栈会打印到终端，daemon方式运行时异常调用栈会记录到[Worker::stdoutFile](worker/stdout-file.md)指定的文件中。


## PROCESS STATUS

pid：进程pid

memory：该进程当前占用内存（不包括php自身可执行文件的占用的内存）

listening：传输层协议及监听ip端口。如果不监听任何端口则显示none。参见[Worker类构造函数](worker/construct.md)

worker_name：该进程运行的服务服务名，见[Worker类name属性](worker/name.md)

connections：该进程**当前**有多少个TCP连接实例，连接实例包括 TcpConnection和AsyncTcpConnection实例。这个值是实时数值，并非累计值。注意：当连接实例调用close后，如果相应计数没有相应减少，可能是业务代码保存了$connection对象，导致这个连接实例无法销毁。

total_request：表示该进程从启动到现在一共接收了多少个请求。这里的请求数不仅包含客户端发来的请求，也包含Workerman内部通讯请求，例如GatewayWorker架构里Gateway与BusinessWorker之间的通讯请求。这个值是累计值。

send_fail：该进程向客户端发送数据失败次数，失败原因一般为客户端连接断开，此项不为0一般属于正常状态，参见[status里send_fail的原因](../faq/about-send-fail.md)。这个值是累计值。

timers：该进程活动的定时器数量（不包括被删除的定时器以及已经运行过的一次性定时器）。注意：此特性需要workerman版本>=3.4.7。这个值是实时数值，并非累计值。

qps：当前进程每秒收到的网络请求数，注意：只有status时加```-d```才会统计此选项，否则显示0。此特性需要workerman版本>=3.5.2。这个值是实时数值，并非累计值。

status: 进程状态，如果是idle代表空闲，如果是busy代表是繁忙。注意：如果进程进入短暂的繁忙是正常情况，如果进程一直是繁忙状态，则有可能发生了业务阻塞或者业务死循环的情况，需要根据[调试busy进程](busy-process.md)一节排查。注意：此特性需要workerman版本>=3.5.0。

## 原理
status脚本运行后，主进程会向所有worker进程发送一个```SIGUSR2```信号，随后status脚本进入短暂的睡眠阶段，以便等待各个worker进程状态统计结果。这时空闲的worker进程收到```SIGUSR2```信号后会立刻向特定的磁盘文件写入自己的运行状态（连接数、请求数等等），而正在处理业务逻辑的worker进程，则会等待业务逻辑处理完毕才会去写入自己的状态信息。短暂睡眠后，status脚本开始读取磁盘中的状态文件，并展示结果到控制台。

## 注意
status 时可能会发现有些进程显示busy，原因是由于进程忙于处理业务（例如业务逻辑长时间阻塞在curl或者数据库请求上，或者运行大的循环），无法将状态上报，导致显示busy。

出现这种问题需要排查业务代码，看哪里导业务致长时间阻塞，并且评估阻塞耗时是否在预期内，如果不符合预期需要根据[调试busy进程](busy-process.md)一节排查业务代码。