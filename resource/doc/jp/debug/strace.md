システムコールをトレースする

あるプロセスが何をしているか知りたい場合は、```strace```コマンドを使用してそのプロセスのすべてのシステムコールをトレースすることができます。

1. php start.php statusを実行して、以下のようにworkerman関連のプロセス情報を表示できます：

```bash
Hello admin
---------------------------------------GLOBAL STATUS--------------------------------------------
Workerman version:3.0.1
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
...（省略）
```

2. たとえば、pidが10354のgatewayプロセスが何をしているか知りたい場合は、次のように```strace -p 10354```コマンドを実行します（管理者権限が必要なことがあります）：

```bash
sudo strace -p 10354
Process 10354 attached - interrupt to quit
clock_gettime(CLOCK_MONOTONIC, {118627, 242986712}) = 0
gettimeofday({1407840609, 102439}, NULL) = 0
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Interrupted system call)
--- SIGUSR2 (User defined signal 2) @ 0 (0) ---
...（省略）
```

3. ここで、各行は1つのシステムコールを表しており、この情報からプロセスが何をしているか、どこで停止しているか、接続やネットワークデータの読み取りで停止しているかなどを簡単に特定することができます。
