# Theo dõi các cuộc gọi hệ thống

Khi muốn biết một tiến trình đang làm gì, bạn có thể sử dụng lệnh ```strace``` để theo dõi tất cả các cuộc gọi hệ thống của một tiến trình.

1. Chạy lệnh php start.php status để xem thông tin về tiến trình liên quan đến workerman như sau:

```
Xin chào quản trị viên
---------------------------------------TRẠNG THÁI TOÀN CỤC--------------------------------------------
Phiên bản Workerman: 3.0.1
Thời gian bắt đầu: 2014-08-12 17:42:04   Đã chạy 0 ngày 1 giờ
Trung bình tải: 3.34, 3.59, 3.67
1 người dùng       8 người làm việc       14 tiến trình
worker_name       exit_status     exit_count
BusinessWorker    0                0
ChatWeb           0                0
FileMonitor       0                0
Gateway           0                0
Monitor           0                0
StatisticProvider 0                0
StatisticWeb      0                0
StatisticWorker   0                0
---------------------------------------TRẠNG THÁI TIẾN TRÌNH-------------------------------------------
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

2. Ví dụ, nếu chúng ta muốn biết tiến trình gateway có pid là 10354 đang làm gì, chúng ta có thể chạy lệnh strace -p 10354 (có thể cần quyền root) như sau:

```
sudo strace -p 10354
Tiến trình 10354 đã bắt đầu - nhấn Ctrl + C để thoát
clock_gettime(CLOCK_MONOTONIC, {118627, 242986712}) = 0
gettimeofday({1407840609, 102439}, NULL) = 0
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Interrupted system call)
--- SIGUSR2 (Tín hiệu người dùng định nghĩa 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (giờ đang []) 
clock_gettime(CLOCK_MONOTONIC, {118627, 699623319}) = 0
gettimeofday({1407840609, 559092}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118627, 699810499}) = 0
gettimeofday({1407840609, 559277}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Resource temporarily unavailable)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Interrupted system call)
--- SIGUSR2 (Tín hiệu người dùng định nghĩa 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (giờ đang [])
clock_gettime(CLOCK_MONOTONIC, {118628, 699497204}) = 0
gettimeofday({1407840610, 558937}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118628, 699588603}) = 0
gettimeofday({1407840610, 559023}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Resource temporarily unavailable)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Interrupted system call)
--- SIGUSR2 (Tín hiệu người dùng định nghĩa 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (giờ đang [])
```

3. Mỗi dòng trong output là một cuộc gọi hệ thống, từ thông tin này, chúng ta dễ dàng nhìn thấy tiến trình đang làm gì, có thể xác định nơi mà tiến trình đang bị kẹt, kẹt ở việc kết nối hay đọc dữ liệu mạng.
