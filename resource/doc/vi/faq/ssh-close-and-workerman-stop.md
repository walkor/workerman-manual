# Dừng dịch vụ khi đóng cửa sổ terminal

**Câu hỏi:**

Tại sao khi tôi đóng cửa sổ terminal thì Workerman cũng tự đóng?

**Trả lời:**

Workerman có hai chế độ khởi động, chế độ debug và chế độ daemon.

Chạy ```php xxx.php start``` là để vào chế độ debug, dành cho việc phát triển và gỡ lỗi, khi đóng cửa sổ terminal thì Workerman sẽ tự đóng theo sau.

Chạy ```php xxx.php start -d``` là để vào chế độ daemon, khi đóng cửa sổ terminal thì không ảnh hưởng đến Workerman.

Nếu muốn Workerman không bị ảnh hưởng bởi cửa sổ terminal, có thể sử dụng chế độ daemon để khởi động.
