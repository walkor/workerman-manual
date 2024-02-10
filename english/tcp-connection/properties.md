## 类属性

Workerman框架内部有一些类属性，可以在实例化Worker类前通过修改这些属性来改变全局设置。

### 支持的类属性

- `Worker::$_stdoutFile`

    标准输出重定向文件路径。如果设置了这个属性，那么框架不再将标准输出打印到终端，而是将输出写入到指定文件。可以通过设置`Worker::$_daemonize = true`来后台模式运行，并结合`Worker::$_stdoutFile`将标准输出写入到文件中。

- `Worker::$_daemonize`

    是否以守护进程的方式运行。设置为`true`后，框架将以守护进程的方式运行，子进程会脱离终端，终端关闭不会影响框架运行。

- `Worker::$_pidFile`

    存放master进程pid的文件。在运行中，master进程的pid会写入到指定文件中，用于实现进程管理等功能。

- `Worker::$_logFile`

    日志文件路径。框架运行中产生的日志都会写入到这个文件中。

- `Worker::$_reloadable`

    是否支持平滑重启。设置为`true`后，可以通过发送`kill -USR1 [master_pid]`信号来进行平滑重启。

- `Worker::$_onWorkerStart`
    
    Worker进程启动时的回调函数。可以设置一个回调函数，用于在Worker进程启动时执行一些初始化操作。

- `Worker::$_onWorkerStop`
    
    Worker进程停止时的回调函数。可以设置一个回调函数，用于在Worker进程停止时执行一些清理操作。

- `Worker::$_onError`
    
    运行时发生错误时的回调函数。可以设置一个回调函数，用于处理运行时发生的各种错误。

- `Worker::$_protocols`
    
    支持的协议列表。可以注册自定义协议，从而支持更多的网络协议。

- `Worker::$_reusePort`

    是否开启SO_REUSEPORT端口重用。设置为`true`后，可以让多个监听同一端口的Worker进程共享这个端口，提高并发连接数。

- `Worker::$_maxSendBufferSize`

    发送缓冲区大小限制。设置发送缓冲区的大小限制，可以避免发送速度过快导致内存溢出。

- `Worker::$_maxPackageSize`

    接收数据包大小限制。设置接收数据包的大小限制，可以避免恶意攻击造成的内存溢出。

以上是Workerman框架的一些类属性，可以在实例化Worker类前通过修改这些属性来改变全局设置。
