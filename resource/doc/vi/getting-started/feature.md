# Đặc điểm của WorkerMan

### 1. Phát triển bằng PHP thuần
Ứng dụng phát triển bằng WorkerMan không phụ thuộc vào php-fpm, apache, nginx và các container khác mà vẫn có thể chạy độc lập. Điều này giúp cho nhà phát triển PHP dễ dàng phát triển, triển khai và gỡ lỗi ứng dụng.

### 2. Hỗ trợ nhiều tiến trình PHP
Để tận dụng tối đa hiệu suất của máy chủ có nhiều CPU, WorkerMan mặc định hỗ trợ nhiều tiến trình nhiều nhiệm vụ. WorkerMan khởi chạy một tiến trình chính và nhiều tiến trình con để phục vụ bên ngoài, tiến trình chính giám sát các tiến trình con, các tiến trình con độc lập lắng nghe kết nối mạng và xử lý dữ liệu, nhờ mô hình tiến trình đơn giản, WorkerMan trở nên ổn định và hiệu quả hơn.

### 3. Hỗ trợ TCP, UDP
WorkerMan hỗ trợ hai loại giao thức tầng vận chuyển TCP và UDP, chỉ cần thay đổi một thuộc tính có thể chuyển đổi giao thức tầng vận chuyển mà không cần thay đổi mã nguồn. 

### 4. Hỗ trợ kết nối kéo dài
Trong nhiều trường hợp, ứng dụng PHP cần duy trì kết nối kéo dài với client, ví dụ như phòng trò chuyện, trò chơi, nhưng các container PHP truyền thống (như apache, nginx, php-fpm) khó thực hiện điều này. Sử dụng WorkerMan, chỉ cần dịch vụ máy chủ không gọi giao diện đóng kết nối, có thể sử dụng kết nối dài của PHP. Một tiến trình WorkerMan có thể hỗ trợ hàng nghìn kết nối đồng thời, nhiều tiến trình có thể hỗ trợ hàng chục nghìn hoặc thậm chí hàng triệu kết nối đồng thời.

### 5. Hỗ trợ nhiều loại giao thức ứng dụng
Giao diện của WorkerMan hỗ trợ nhiều loại giao thức ứng dụng, bao gồm cả giao thức tùy chỉnh. Trong WorkerMan, việc chuyển đổi giao thức cũng rất đơn giản, chỉ cần cấu hình một trường, giao thức sẽ tự động chuyển đổi, mã nguồn không thay đổi, thậm chí có thể mở nhiều cổng với các giao thức khác nhau, đáp ứng yêu cầu của các client khác nhau.

### 6. Hỗ trợ đa nhiệm
WorkerMan hỗ trợ thư viện Event của Libevent (yêu cầu cài đặt extension event), sử dụng Event ở môi trường kết nối kéo dài đa nhiệm hiệu suất rất tuyệt vời, nếu không cài đặt extension Event thì sử dụng lời gọi hệ thống Select có sẵn trong PHP, hiệu suất cũng rất mạnh mẽ. 

### 7. Hỗ trợ khởi động lại dịch vụ mượt mà
Khi cần khởi động lại dịch vụ (ví dụ như phát hành phiên bản mới), chúng ta không muốn các tiến trình đang xử lý yêu cầu của người dùng bị chấm dứt ngay lập tức, cũng không muốn tại thời điểm khởi động lại làm người dùng giao tiếp thất bại. WorkerMan cung cấp chức năng khởi động lại mượt mà, có thể đảm bảo dịch vụ được nâng cấp mượt mà, không ảnh hưởng đến người dùng.

### 8. Hỗ trợ kiểm tra và tải tự động tập tin
Trong quá trình phát triển, chúng ta muốn tập tin được cập nhật ngay khi thay đổi mã nguồn để kiểm tra kết quả. WorkerMan cung cấp [mô-đun theo dõi tệp tin](../components/file-monitor.md), chỉ cần tệp tin cập nhật, WorkerMan sẽ tự động chạy lại để tải tệp tin mới, làm cho nó có hiệu lực.

### 9. Hỗ trợ chạy tiến trình con với người dùng cụ thể
Vì tiến trình con là tiến trình xử lý yêu cầu của người dùng, vì lý do an ninh, tiến trình con không nên có quyền quản lý quá cao, vì vậy WorkerMan hỗ trợ cài đặt người dùng chạy tiến trình con, làm cho máy chủ của bạn an toàn hơn.

### 10. Hỗ trợ duy trì vĩnh viễn đối tượng hoặc tài nguyên
WorkerMan trong quá trình chạy chỉ cần tải và phân tích tệp PHP một lần, sau đó vĩnh viễn ở trong bộ nhớ, điều này làm cho khai báo lớp và hàm, môi trường thực thi PHP, bảng ký hiệu và các loại khác không được tạo hoặc phá hủy lặp đi lặp lại, hoàn toàn khác biệt so với cơ chế chạy PHP dưới container Web. Trong WorkerMan, các thành viên tĩnh hoặc biến toàn cục trong vòng đời của một tiến trình sẽ được duy trì vĩnh viễn nếu không được phá hủy tự động, điều này có nghĩa là đối tượng hoặc kết nối và tài nguyên khác được đặt trong biến toàn cục hoặc thành viên tĩnh của lớp thì tất cả các yêu cầu cho tiến trình này có thể tái sử dụng. Ví dụ chỉ cần một lần khởi tạo kết nối cơ sở dữ liệu trong một tiến trình thì tất cả các yêu cầu sau này của tiến trình này có thể tái sử dụng kết nối cơ sở dữ liệu đó, tránh quá trình thấp điểm kết nối cơ sở dữ liệu, quá trình xác minh quyền truy cập của cơ sở dữ liệu và quá trình kết thúc kết nối cũng vì thế mà tăng hiệu quả của ứng dụng.

### 11. Hiệu quả cao
Do tập tin PHP chỉ cần đọc và phân tích từ đĩa một lần sau đó cư trú vĩnh viễn trong bộ nhớ, lần sử dụng tiếp theo sử dụng opcode trong bộ nhớ trực tiếp, làm giảm đáng kể việc đọc ghi từ đĩa cắm IO và quá trình thiết lập, tạo môi trường thực thi, phân tích từ vựng, phân tích cú pháp, biên dịch opcode, đóng yêu cầu PHP cơ bản khác, và không phụ thuộc vào nginx, apache và các container khác, giảm sự tốn kém khi giao tiếp với PHP của nginx và các container khác, quan trọng nhất là các tài nguyên có thể duy trì vĩnh viễn, không cần phải khởi tạo kết nối cơ sở dữ liệu và các thứ khác lặp đi lặp lại, do đó, khi phát triển ứng dụng với WorkerMan, hiệu suất rất cao.

### 12. Hỗ trợ HHVM
Hỗ trợ chạy trên máy ảo HHVM, có thể cải thiện hiệu suất PHP gấp đôi. Đặc biệt là trong các ngành nghề tính năng công việc tốn nhiều tài nguyên CPU, hiệu suất rất tốt. Thông qua so sánh kiểm tra áp lực thực tế, khi không có công việc tải nặng, WorkerMan chạy trong HHVM tăng khả năng thông lưu mạng 30-80% so với được chạy dưới Zend PHP5.6.

### 13. Hỗ trợ triển khai phân tán

### 14. Hỗ trợ tiến trình thông qua daemon

### 15. Hỗ trợ nghe nhiều cổng

### 16. Hỗ trợ tái chuyển hướng nhập xuất tiêu chuẩn
