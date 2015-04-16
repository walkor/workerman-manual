# 查看运行状态

运行 ```php start.php status```
可以查看到WorkerMan的运行状态，类似如下:

```
---------------------------------------GLOBAL STATUS--------------------------------------------
Workerman version:3.0.3          PHP version:5.3.29-1~dotdeb.0
start time:2014-07-21 18:05:47   run 86 days 22 hours
load average: 0, 0, 0
3 workers       10 processes
worker_name           exit_status     exit_count
TodpoleGateway        0                0
TodpoleBusinessWorker 0                4
TodpoleBusinessWorker 9                1
WebServer             0                2
---------------------------------------PROCESS STATUS-------------------------------------------
pid     memory  listening                worker_name           connections total_request send_fail throw_exception
936     2.15M   Websocket://0.0.0.0:8585 TodpoleGateway        13         355            0         0
937     2.03M   Websocket://0.0.0.0:8585 TodpoleGateway        5          181            0         0
938     2M      Websocket://0.0.0.0:8585 TodpoleGateway        4          171            0         0
939     2.03M   Websocket://0.0.0.0:8585 TodpoleGateway        5          177            0         0
948     2.15M   none                     TodpoleBusinessWorker 4          32             0         0
949     2.16M   none                     TodpoleBusinessWorker 4          54             0         0
953     2.16M   none                     TodpoleBusinessWorker 4          50             0         0
957     2.15M   none                     TodpoleBusinessWorker 4          53             0         0
954     1.84M   http://0.0.0.0:8686      WebServer             0          61             0         0
955     1.84M   http://0.0.0.0:8686      WebServer             1          59             0         0
```

## 说明

### GLOBAL STATUS

从这以栏中我们可以看到

WorkerMan的版本```version:3.0.3```

启动时间```2015-02-21 18:05:47```，运行了```run 6 days 22 hours```

服务器负载 ```load average: 0, 0, 0```

```3 workers```（3种进程，包括ChatGateway、ChatBusinessWorker、WebServer进程）

```14 processes```(共10个进程)

```worker_name```(服务名)

```exit_status```（退出状态值）

```exit_count```（该状态的退出次数）


其中exit_status为0为正常退出，如果为其它值，代表进程是异常退出的，比如exit_status为65280，exit_count为60，代表业务代码有FatalError，导致进程退出60次，需要根据php的错误日志查找FatalError原因

## PROCESS STATUS

pid：进程pid

memory：该进程占用内存（不包括php自身可执行文件的占用的内存）

listening：传输层协议及监听ip端口

timestamp：该进程启动时间戳

worker_name：该进程运行的服务服务名

connections:该进程当前有多少个TCP连接，包括客户端连接和WorkerMan内部通讯的连接

total_request：该进程接收多少请求，注意每个连接上可能有多个请求

send_fail：该进程向客户端发送数据失败次数，失败原因一般为客户端连接断开

throw_exception：该进程内业务未捕获的异常数量



