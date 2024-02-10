# Lớp Worker

Trong WorkerMan có hai lớp quan trọng là Worker và Connection.

Lớp Worker được sử dụng để lắng nghe cổng và có thể thiết lập các hàm gọi lại cho sự kiện kết nối của khách hàng, sự kiện nhận thông điệp trên kết nối, sự kiện ngắt kết nối để thực hiện xử lý kinh doanh.

Có thể thiết lập số tiến trình của thực thể Worker (thuộc tính count), quá trình chính của Worker sẽ sao chép ra count tiến trình con đồng thời lắng nghe cùng một cổng, xử lý các sự kiện kết nối.
