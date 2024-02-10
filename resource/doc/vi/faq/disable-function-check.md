# Kiểm tra và vô hiệu hóa các hàm

Sử dụng script sau để kiểm tra xem có bất kỳ hàm nào bị vô hiệu hóa không. Chạy lệnh sau trong dòng lệnh `curl -Ss https://www.workerman.net/check | php`

Nếu xuất hiện thông báo `Function tên_hàm có thể bị vô hiệu hóa. Vui lòng kiểm tra disable_functions trong tệp php.ini`, điều đó có nghĩa là các hàm mà workerman phụ thuộc đã bị vô hiệu hóa và cần phải bỏ vô hiệu hóa chúng trong tệp php.ini để có thể sử dụng workerman một cách bình thường.
Có thể thực hiện bỏ vô hiệu hóa bằng cách chọn một trong hai phương pháp sau.

## Phương pháp một: Sử dụng script để bỏ vô hiệu hóa

Thực thi script `curl -Ss https://www.workerman.net/fix | php` để bỏ vô hiệu hóa.

## Phương pháp hai: Bỏ vô hiệu hóa thủ công

**Bước 1:** Chạy `php --ini` để tìm vị trí tệp php.ini mà php cli đang sử dụng

**Bước 2:** Mở tệp php.ini, tìm đến mục `disable_functions` và bỏ vô hiệu hóa các hàm tương ứng.

**Các hàm phụ thuộc:**
Để sử dụng workerman, cần bỏ vô hiệu hóa các hàm sau:
```
stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
