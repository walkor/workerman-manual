# workerman开发者必须知道的几个问题
**1、windows环境限制**

windows系统下workerman单个进程仅支持200+个连接。
windows系统下无法使用count参数设置多进程。
windows系统下无法使用status、stop、reload、restart等命令。
windows系统下无法守护进程，cmd窗口关掉后服务即停止。
windows系统下无法在一个文件中初始化多个监听。
linux系统无上面的限制，建议正式环境用linux系统，开发环境可以选择用windows系统。

**2、workerman不依赖apache或者nginx**

workerman本身已经是一个类似apache/nginx的容器，只要[PHP环境OK](315116) workerman就可以运行。

**3、workerman是命令行启动的**

启动方式类似apache使用命令启动(一般网页空间无法使用workerman)。启动界面类似下面
![](image/screenshot_1495622774534.png)

**4、长连接必须加心跳**

长连接必须加心跳，长连接必须加心跳，长连接必须加心跳，重要的话说三遍。 
长连接长时间不通讯会被路由节点清理导致连接关闭。
[workerman心跳说明](faq/heartbeat.md)、 [gatewayWorker心跳说明](https://www.workerman.net/doc/gateway-worker/heartbeat.html)

**5、客户端和服务端协议一定要对应才能通讯**

这个是开发者非常常见的问题。例如客户端是用websocket协议，服务端必须也是websocket协议(服务端```new Worker('websocket://0.0.0.0...')```)才能连得上，才能通讯。 
不要尝试在浏览器地址栏访问websocket协议端口，不要尝试用webscoket协议访问裸tcp协议端口，协议一定要对应。

这里的原理类似如果你要和英国人交流，那么要使用英语。如果要和日本人交流，那么要使用日语。这里的语言就类似与通讯协议，双方(客户端和服务端)必须使用相同的语言才能交流，否则无法通讯。 

**6、连接失败可能的原因**

刚开始使用workerman时很常见的一个问题是客户端连接服务端失败。 原因一般如下： 
1、服务器防火墙(包括云服务器安全组)阻止了连接 （50%几率是这个）
2、客户端和服务端使用的协议不一致 （30%几率）
3、ip或者端口写错了 (15%的几率)
4、服务端没启动 


**7、不要使用exit die sleep语句**

业务执行exit die语句会导致进程退出，并显示WORKER EXIT UNEXPECTED错误。当然，进程退出了会立刻重启一个新的进程继续服务。如果需要返回，可以调用return。sleep语句会让进程睡眠，睡眠过程中不会执行任何业务，框架也会停止运行，会导致该进程的所有客户端请求都无法处理。

**8、不要使用pcntl_fork函数**

`pcntl_fork`用来动态创建新的进程，如果在业务代码中使用`pcntl_fork`，它可能会产生无法回收孤儿进程，导致业务出现异常。业务中`pcntl_fork`还会影响连接、消息、连接关闭、定时器等事件的处理，导致不可预知的异常。

**9、业务代码里不要有死循环**

业务代码里不要有死循环，否则会导致控制权无法交还给workerman框架，导致无法接收处理其它客户端消息。

**10、改代码要重启**

workerman是常驻内存的框架，改代码要重启workerman才能看到新代码的效果。

**11、长连接应用建议用GatewayWorker框架**

很多开发者使用workerman是要开发**长连接**应用，例如即时通讯、物联网等，**长连接**应用建议直接使用GatewayWorker框架，它专门在workerman的基础上再次封装，做起长连接应用后台更简单、更易用。

**12、支持更高并发**
如果业务并发连接数超过1000同时在线，请务必[优化linux内核](appendices/kernel-optimization.md)，并[安装event扩展](appendices/install-extension.md)。





