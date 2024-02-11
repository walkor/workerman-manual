# 시스템 호출 추적

어떤 프로세스가 무엇을 하는지 알고 싶을 때는 ```strace``` 명령어를 사용하여 프로세스의 모든 시스템 호출을 추적할 수 있습니다.

1. php start.php status를 실행하여 workerman 관련 프로세스 정보를 확인할 수 있습니다.

```plaintext
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
(이하 생략)
```

2. 예를 들어 pid가 10354인 gateway 프로세스가 무엇을 하는지 알고 싶다면 다음과 같이 명령을 실행할 수 있습니다. (root 권한이 필요할 수 있습니다)

```plaintext
sudo strace -p 10354
Process 10354 attached - interrupt to quit
clock_gettime(CLOCK_MONOTONIC, {118627, 242986712}) = 0
gettimeofday({1407840609, 102439}, NULL) = 0
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Interrupted system call)
(이하 생략)
```

3. 여기서 각 라인은 하나의 시스템 호출을 나타냅니다. 이 정보를 통해 프로세스가 무엇을 하는지 쉽게 파악할 수 있으며, 프로세스가 어디에서 멈추는지, 연결 또는 네트워크 데이터를 읽는 과정에서 멈추는지 등을 확인할 수 있습니다.
