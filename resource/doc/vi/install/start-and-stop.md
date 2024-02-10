# Khởi động và dừng lại

Lưu ý rằng các lệnh khởi động và dừng Workerman đều được thực hiện thông qua dòng lệnh.

Để khởi động Workerman, trước tiên cần có một tệp nhập khẩu khởi động, trong đó xác định cổng lắng nghe dịch vụ và giao thức. Bạn có thể tham khảo [hướng dẫn bắt đầu - mục ví dụ phát triển đơn giản](../getting-started/simple-example.md)

Ở đây, chúng ta sẽ lấy ví dụ của [workerman-chat](https://www.workerman.net/workerman-chat), tệp nhập khẩu khởi động của nó là start.php.

### Khởi động

Khởi động theo chế độ debug

 ```php start.php start```

Khởi động theo chế độ daemon

 ```php start.php start -d```

### Dừng lại
 ```php start.php stop```

### Khởi động lại
 ```php start.php restart```

### Khởi động lại mượt mà
 ```php start.php reload```

### Kiểm tra trạng thái
 ```php start.php status```
 
### Kiểm tra trạng thái kết nối (yêu cầu Workerman phiên bản >=3.5.0)
```php start.php connections```

## Sự khác biệt giữa chế độ debug và daemon

1. Khởi động theo chế độ debug, các hàm in ra như echo, var_dump, print sẽ xuất ra trực tiếp trên cửa sổ terminal.

2. Khởi động theo chế độ daemon, các hàm in ra như echo, var_dump, print mặc định sẽ được điều hướng đến tệp /dev/null, có thể thiết lập đường dẫn tệp này bằng cách sử dụng ```Worker::$stdoutFile = '/your/path/file';```

3. Khởi động theo chế độ debug, khi đóng cửa sổ terminal, Workerman sẽ đóng và thoát theo.

4. Khởi động theo chế độ daemon, khi đóng cửa sổ terminal, Workerman vẫn tiếp tục chạy bình thường ở nền.

## Khái niệm khởi động mượt mà là gì?

Xem [Nguyên lý khởi động mượt mà](../faq/reload-principle.md)
