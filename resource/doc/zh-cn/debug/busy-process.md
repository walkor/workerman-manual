# 调试busy进程
有时候我们通过```php start.php status``` 命令能看到有```busy```状态的进程，说明对应进程正在处理业务，正常情况下业务处理完毕对应进程会恢复为```idle```状态，这一般情况下不会有什么问题。但是如果一直是busy状态没有恢复过```idle```状态，则说明进程内的业务有阻塞或者无限循环，可以通过以下方法定位。

## 利用strace+lsof命令定位

**1、status里找到busy进程的pid**
运行```php start.php status``` 后显示如下
![](../images/d1903ed65ef2f3b0850e84ccbedc52aa.png)
图中```busy```的进程的```pid```为```11725```和```11748```

**2、strace 跟踪进程**
挑选一个进程```pid```(这里选择```11725```)，运行```strace -ttp 11725``` 显示如下
![](../images/7ce9f36da926f670949609dcdc593ab4.png)
可以看到进程在不断的循环```poll([{fd=16, events=....```的系统调用，这是在等待fd为16的描述符可读事件，也就是在等这个描述符返回数据。

如果没有显示任何系统调用，保留当前终端，重新再打开一个终端，运行```kill -SIGALRM 11725```(给进程发送一个闹钟信号)，然后看strace的终端是否有响应，是否阻塞在某个系统调用上。如果仍然没有显示任何系统调用说明程序很可能处于业务死循环中，参考页面下部引起进程长时间busy的其它原因 第2项 解决。

如果系统阻塞在epoll_wait或者select系统调用是正常情况，这说明进程已经处于idle状态。

**3、lsof 查看进程描述符**
运行```lsof -nPp 11725``` 显示如下
![](../images/27bd629c3a1ac93f9f4b535d01df2ac1.png)
述符16对应的是16u的记录(最后一行)，能看```fd=16```的描述符是一个tcp连接，远程地址是```101.37.136.135:80```，说明进程应该是在访问一个http资源，循环```poll([{fd=16, events=....```是一直在等待http服务端返回数据，这解释了为什么进程处于```busy```状态

**解决：**
知道了进程阻塞在哪里，接下来就容易解决了，例如上面经过定位应该是业务在调用curl，而对应的url长时间没有返回数据，导致进程一直等待。这时候可以找url提供者定位url返回慢的原因，同时应该在curl调用的时候加上超时参数，比如2秒没返回就超时，避免长时间阻塞卡死(这样进程可能会出现2秒左右的busy状态)。

## 引起进程长时间 busy 的其它原因
除了进程阻塞或导致进程```busy```，还有以下原因会引起进程处于```busy```状态。

**1、业务有致命错误导致进程不断退出**
**现象：** 这种情况下能看到系统负载比较高，```status```中的```load average```为1或者更高。能看到进程的```exit_count```数字很高，并且不断增长
**解决：** debug方式运行(```php start.php start```不加```-d```)workerman看下业务报错，把报错解决即可

**2、代码里无限死循环**
**现象：** top中能看到busy进程占用cpu很高，```strace -ttp pid```命令没有打印出任何系统调用信息
**解决：** 参考鸟哥的文章通过[gdb和php源码定位](https://www.laruence.com/2011/12/06/2381.html)，步骤总结大概如下：
1、```php -v```查看版本
2、[下载对应php版本的源码](https://www.php.net/releases/)
3、```gdb --pid=busy进程的pid```
4、```source php源码路径/.gdbinit```，
5、```zbacktrace``` 打印调用栈
最后一步可以看到php代码当前执行的调用栈，也就是php代码死循环的位置。
注意：如果```zbacktrace``` 没有打出调用栈，可能你的php编译时没有加入```-g```参数，需要重新编译php，然后重启workerman定位。

**3、无限添加定时器**
业务代码不停的添加定时器又不删除，导致进程内定时器越来越多，最终造成进程无限运行定时器。例如以下代码：
```php
$worker = new Worker;
$worker->onConnect = function($con){
    Timer::add(10, function(){});
};
Worker::runAll();
````
以上代码在有客户端连接上来后会增加一个定时器，但是整个业务代码里没有删除定时器的逻辑，这样随着时间推移，进程内会不断增加定时器，最终导致进程无限运行定时器导致busy。
正确的代码：
```php
$worker = new Worker;
$worker->onConnect = function($con){
    $con->timer_id = Timer::add(10, function(){});
};
$worker->onClose = function($con){
    Timer::del($con->timer_id);
};
Worker::runAll();
````


