# Làm cách nào để tích hợp với các framework khác

**Câu hỏi:**

Làm cách nào để tích hợp với các framework MVC khác (ví dụ như thinkPHP, Yii)?

**Trả lời:**

![workerman-thinkphp](../images/workerman-work-with-thinkphp.png)

Kết hợp với các framework MVC khác **đề xuất** theo cách như hình trên (ví dụ với ThinkPHP):

1. ThinkPHP và Workerman là hai hệ thống độc lập, triển khai độc lập (có thể triển khai trên các máy chủ khác nhau), không tác động lẫn nhau.

2. ThinkPHP cung cấp trang web được hiển thị trên trình duyệt thông qua giao thức HTTP.

3. Trang web do ThinkPHP cung cấp sẽ kết nối websocket bằng js, kết nối tới Workerman.

4. Sau khi kết nối, gửi một gói tin dữ liệu tới Workerman (bao gồm tên người dùng, mật khẩu hoặc một loại token) để xác minh kết nối websocket thuộc về người dùng nào.

5. Chỉ khi cần đẩy dữ liệu đến trình duyệt từ ThinkPHP, mới gọi giao diện socket của Workerman để đẩy dữ liệu.

6. Những yêu cầu khác vẫn được xử lý theo cách thức ban đầu của ThinkPHP thông qua giao thức HTTP.


**Tóm lược:**

Sử dụng Workerman như một kênh có thể đẩy dữ liệu đến trình duyệt, chỉ gọi giao diện Workerman để đẩy dữ liệu khi cần thiết. Tất cả logic kinh doanh được thực hiện trong ThinkPHP.

Cách ThinkPHP gọi giao diện socket của Workerman để đẩy dữ liệu, xem thêm mục [Hỏi đáp phổ biến - Đẩy dữ liệu trong dự án khác](push-in-other-project.md)

**ThinkPHP chính thức hỗ trợ Workerman, xem thêm tại [Sổ tay ThinkPHP5](https://www.kancloud.cn/manual/thinkphp5/235128)**
