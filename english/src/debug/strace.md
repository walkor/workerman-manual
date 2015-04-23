# 跟踪系统调用

当想知道一个进程在做什么事情的时候，可以通过```strace```命令跟踪一个进程的所有系统调用。

1、运行 php start.php status 能看到workerman相关进程的信息 如下：


```
Hello admin
---------------------------------------GLOBAL STATUS--------------------------------------------
WorkerMan version:3.0.1
start time:2014-08-12 17:42:04   run 0 days 1 hours
load average: 3.34, 3.59, 3.67
1 users          8 workers       14 processes
worker_name       exit_status     exit_count
BusinessWorker    0                0
ChatWeb           0                0
FileMonitor       0                0
Gateway           0                0
Monitor           0                0
StatisticProvider 0                0
StatisticWeb      0                0
StatisticWorker   0                0
---------------------------------------PROCESS STATUS-------------------------------------------
pid	memory      listening        timestamp  worker_name       total_request packet_err thunder_herd client_close send_fail throw_exception suc/total
10352	1.5M    tcp://0.0.0.0:55151  1407836524 ChatWeb           12             0          0            2            0         0               100%
10354	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          0            0            0         0               100%
10355	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           0              0          1            0            0         0               100%
10365	1.25M   tcp://0.0.0.0:55757  1407836524 StatisticWeb      0              0          0            0            0         0               100%
10358	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          2            0            0         0               100%
10364	1.25M   tcp://0.0.0.0:55858  1407836524 StatisticProvider 0              0          0            0            0         0               100%
10356	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          2            0            0         0               100%
10366	1.25M   udp://0.0.0.0:55656  1407836524 StatisticWorker   55             0          0            0            0         0               100%
10349	1.25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    5              0          0            0            0         0               100%
10350	1.25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    0              0          0            0            0         0               100%
10351	1.5M    tcp://127.0.0.1:7373 1407836524 BusinessWorker    5              0          0            0            0         0               100%
10348	1.25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    2              0          0            0            0         0               100%
```

2、例如我们想知道pid为10354的gateway进程在做什么，则可以运行命令 strace -p 10354 (可能需要root权限) 类似如下：

```
sudo strace -p 10354
Process 10354 attached - interrupt to quit
clock_gettime(CLOCK_MONOTONIC, {118627, 242986712}) = 0
gettimeofday({1407840609, 102439}, NULL) = 0
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Interrupted system call)
--- SIGUSR2 (User defined signal 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (mask now [])
clock_gettime(CLOCK_MONOTONIC, {118627, 699623319}) = 0
gettimeofday({1407840609, 559092}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118627, 699810499}) = 0
gettimeofday({1407840609, 559277}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Resource temporarily unavailable)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Interrupted system call)
--- SIGUSR2 (User defined signal 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (mask now [])
clock_gettime(CLOCK_MONOTONIC, {118628, 699497204}) = 0
gettimeofday({1407840610, 558937}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118628, 699588603}) = 0
gettimeofday({1407840610, 559023}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Resource temporarily unavailable)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Interrupted system call)
--- SIGUSR2 (User defined signal 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (mask now [])
```

3、其中每一行是一个系统调用，从这个信息中我们很容易看到进程在做一些什么事情，可以定位到进程卡在哪里，卡在链接还是读取网络数据等


