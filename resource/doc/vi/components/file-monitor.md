# Bộ theo dõi tập tin

**Nền tảng:**

Workerman chạy liên tục trong bộ nhớ, và việc chạy liên tục trong bộ nhớ giúp tránh việc đọc lại ổ đĩa hoặc biên dịch lại PHP để đạt được hiệu suất cao nhất. Do đó, sau khi thay đổi mã kinh doanh, cần phải tải lại hoặc khởi động lại thì mới có hiệu lực.

Đồng thời, Workerman cung cấp dịch vụ theo dõi cập nhật tập tin, dịch vụ này tự động chạy lại khi có cập nhật tập tin, tải lại tập tin PHP. Người phát triển có thể đặt nó vào dự án và khởi động cùng với dự án.

**Địa chỉ tải dịch vụ theo dõi tập tin:**

1. Phiên bản không phụ thuộc: https://github.com/walkor/workerman-filemonitor

2. Phiên bản phụ thuộc inotify: https://github.com/walkor/workerman-filemonitor-inotify (cần cài đặt [phần mở rộng inotify](https://php.net/manual/zh/book.inotify.php))

**Sự khác biệt giữa hai phiên bản:**

Phiên bản địa chỉ 1 sử dụng phương pháp kiểm tra thời gian cập nhật tập tin mỗi giây để xác định xem tập tin có được cập nhật không.

Phiên bản địa chỉ 2 sử dụng cơ chế [inotify](https://baike.baidu.com/view/2645027.htm) của hạt nhân Linux, khi có cập nhật tập tin, hệ thống sẽ thông báo cho Workerman.

Thường thì chỉ cần sử dụng phiên bản không phụ thuộc là đủ.

**Cách sử dụng**

Hãy sao chép thư mục Applications/FileMonitor vào thư mục Applications của dự án của bạn.

Nếu dự án của bạn không có thư mục Applications, bạn có thể sao chép tệp start.php từ thư mục Applications/FileMonitor vào bất kỳ vị trí nào trong dự án của bạn và gọi require tệp này trong tệp khởi động.

Dịch vụ theo dõi mặc định theo dõi thư mục Applications. Nếu muốn thay đổi, bạn có thể chỉnh sửa biến ```$monitor_dir``` trong tệp start.php trong thư mục Applications/FileMonitor và giá trị của ```$monitor_dir``` nên là đường dẫn tuyệt đối.

**Chú ý:**

* Hệ thống Windows không hỗ trợ reload, không thể sử dụng dịch vụ theo dõi này.
* Chỉ có thể thực hiện trong chế độ debug, không thực hiện trong chế độ daemon (lý do không hỗ trợ chế độ daemon xem giải thích dưới đây).
* Chỉ các tệp được tải sau khi Worker::runAll chạy mới có thể cập nhật nhanh chóng, hoặc nói cách khác, chỉ có thể cập nhật nhanh chóng những tệp được tải trong các cuộc gọi onXXX.

**Tại sao không hỗ trợ chế độ daemon?**

Chế độ daemon thường là chế độ chạy trong môi trường sản xuất. Khi phiên bản sản phẩm được phát hành, thường là cùng một lúc phát hành nhiều tập tin, và có thể có sự phụ thuộc giữa các tập tin. Do việc đồng bộ hóa nhiều tập tin này lên ổ đĩa mất một khoảng thời gian, có thể có trường hợp vào một thời điểm nào đó, có một số tập tin không hoàn chỉnh trên ổ đĩa, nếu theo dõi cập nhật tập tin và thực hiện lại khi có cập nhật, có thể gây ra nguy cơ lỗi nghiêm trọng do không tìm thấy tập tin.

Ngoài ra, trong môi trường sản xuất, đôi khi có thể xảy ra sự cố phát hiện lỗi trực tuyến, nếu chỉ cần chỉnh sửa mã và lưu, mã sẽ có hiệu lực ngay lập tức, có thể gây ra lỗi cú pháp khiến dịch vụ trực tuyến không khả dụng. Phương pháp đúng đắn là sau khi lưu mã, kiểm tra lỗi cú pháp bằng ```php -l yourfile.php```, sau đó mới thực hiện reload để cập nhật mã.

Nếu người phát triển thực sự cần kích hoạt theo dõi tập tin và cập nhật tự động trong chế độ daemon, họ có thể tự chỉnh sửa mã trong tệp start.php trong thư mục Applications/FileMonitor bằng cách loại bỏ phần kiểm tra Worker::$daemonize.
