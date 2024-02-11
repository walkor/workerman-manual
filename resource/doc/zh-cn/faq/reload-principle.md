# 平滑重启原理
## 什么是平滑重启？

平滑重启不同于普通的重启，平滑重启可以做到在不影响用户的情况下重启服务(一般指短链接业务)，以便重新载入PHP程序，完成业务代码更新。

平滑重启一般应用于业务更新或者版本发布过程中，能够避免因为代码发布重启服务导致的暂时性服务不可用的影响。

> **注意**
> Windows系统不支持reload。

> **注意**
> 长连接(例如websocket)业务，进程平滑重启时连接会被断开。解决方案是使用类似[gatewayWorker](https://www.workerman.net/doc/gateway-worker)的架构，一组进程专门维持连接，并将这组进程的[reloadable](../worker/reloadable.md)属性设置为false。业务逻辑启动另外一组worker进程处理，gateway与worker进程通过tcp通讯的方式互相调用。当业务需要变更时，只重启worker进程即可。

## 限制
**注意：只有在on{...}回调中载入的文件平滑重启后才会自动更新，启动脚本中直接载入的文件或者写死的代码运行reload不会自动更新。**

#### 以下代码reload后不会更新
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('hi'); // 写死的代码不支持热更新
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/your/path/MessageHandler.php'; // 启动脚本直接载入的文件不支持热更新
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // 假设MessageHandler类里有一个onMessage方法
```


#### 以下代码reload后会自动更新
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart是进程启动后触发的回调
    require_once __DIR__ . '/your/path/MessageHandler.php'; // 进程启动后载入的文件支持热更新
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
MessageHandler.php改动后执行 `php start.php reload`，MessageHandler.php会重新载入内存达到更新业务逻辑的效果。


> **提示**
> 上面代码为了方便演示，使用了`require_once`语句，如果你的项目支持psr4自动加载，则无需调用`require_once`语句。

## 平滑重启原理

Workerman分为主进程和子进程，主进程负责监控子进程，子进程负责接收客户端的连接和连接上发来的请求数据，做相应的处理并返回数据给客户端。当业务代码更新时，其实我们只要更新子进程，便可以达到更新代码的目的。

当Workerman主进程收到平滑重启信号时，主进程会向其中一个子进程发送安全退出(让对应进程处理完毕当前请求后才退出)信号，当这个进程退出后，主进程会重新创建一个新的子进程（这个子进程载入了新的PHP代码），然后主进程再次向另外一个旧的进程发送停止命令，这样一个进程一个进程的重启，直到所有旧的进程全部被置换为止。

我们看到平滑重启实际上是让旧的业务进程逐个退出然后并逐个创建新的进程做到的。为了在平滑重启时不影响客用户，这就要求进程中不要保存用户相关的状态信息，即业务进程最好是无状态的，避免由于进程退出导致信息丢失。
