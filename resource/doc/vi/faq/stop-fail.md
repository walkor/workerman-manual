# Lỗi dừng

## Hiện tượng:
Chạy ```php start.php stop``` hiện thông báo ```stop fail```

### Khả năng 1
Điều kiện tiên quyết là workerman được khởi động bằng chế độ debug. Người phát triển đã gửi tín hiệu ```SIGSTOP``` đến workerman bằng cách nhấn ```ctrl z``` trong cửa sổ terminal, làm cho workerman vào chế độ nền và tạm ngừng, do đó không thể phản hồi lệnh dừng (tín hiệu ```SIGINT```).

**Giải quyết:**
Trong cửa sổ terminal khởi động workerman, nhập ```fg``` (gửi tín hiệu ```SIGCONT```) và nhấn Enter để đưa workerman về chạy ở chế độ trước, sau đó nhấn ```ctrl c``` (gửi tín hiệu ```SIGINT```) để dừng workerman.

Nếu không thể dừng, hãy thử chạy hai lệnh sau:
```killall -9 php```
```ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9```

### Khả năng 2
Người dừng và người khởi động workerman không giống nhau, nghĩa là người dừng không có quyền dừng workerman.

**Giải quyết:**
Chuyển sang người dùng khởi động workerman hoặc sử dụng người dùng có quyền cao hơn để dừng workerman.

### Khả năng 3
Tệp pid của quá trình chính của workerman bị xóa, làm cho script không thể tìm thấy tiến trình pid, dẫn đến việc dừng bị lỗi.

**Giải quyết:**
Lưu tệp pid vào một vị trí an toàn, xem chi tiết trong tài liệu hướng dẫn [Worker::$pidFile](../worker/pid-file.md).

### Khả năng 4
Tệp pid của quá trình chính workerman không phải là của quá trình workerman.

**Giải quyết:**
Mở tệp pid của quá trình chính của workerman để xem pid của quá trình chính, tệp pid mặc định sẽ ở cùng thư mục với workerman. Chạy lệnh ```ps aux | grep pid của quá trình chính``` để kiểm tra xem tiến trình tương ứng có phải là tiến trình workerman hay không. Nếu không phải, có thể do máy chủ đã khởi động lại, làm cho pid được lưu trữ bởi workerman là pid đã hết hạn và được sử dụng bởi một tiến trình khác, dẫn đến việc dừng bị lỗi. Nếu đó là trường hợp, chỉ cần xóa tệp pid là được.

### Khả năng 5
Cài đặt extension grpc, nhưng không thiết lập các biến môi trường tương ứng cho extension grpc, khi khởi động sẽ tạo thêm một tiến trình chứa extension này, dẫn đến việc dừng bị lỗi.
