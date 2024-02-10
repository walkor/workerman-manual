# Tập trung yêu cầu vào một số tiến trình

### Hiện tượng
Đôi khi khi chúng ta sử dụng lệnh `php start.php status`, chúng ta có thể thấy rằng các yêu cầu được tập trung xử lý trong một số tiến trình cụ thể, các tiến trình khác hoàn toàn không hoạt động.

### Cơ chế tranh chấp
Cách mà workerman nhiều tiến trình lấy kết nối mặc định là theo cách **tranh chấp**, có nghĩa là khi có kết nối từ phía máy khách, tất cả các tiến trình trống rỗng đều có cơ hội lấy kết nối này, người nhanh chóng nhất sẽ giành chiến thắng. Ai sẽ nhanh nhất, phụ thuộc vào lịch trình lập trình hạt nhân của hệ điều hành. Hệ điều hành có thể ưu tiên chọn tiến trình đã sử dụng gần đây nhất để cấp quyền sử dụng CPU, vì trong thanh ghi CPU có thể vẫn còn thông tin ngữ cảnh của tiến trình trước đó, điều này có thể giảm thiểu chi phí chuyển đổi ngữ cảnh. Điều này có nghĩa là khi tốc độ kinh doanh đủ nhanh hoặc trong quá trình kiểm tra hiệu suất, việc kết nối bị tập trung vào một số tiến trình cụ thể là phổ biến, vì chiến lược này có thể tránh được việc chuyển đổi tiến trình thường xuyên, hiệu suất thường là tốt nhất và không phải là vấn đề gì cả.

### Cơ chế vòng quay
Workerman có thể thông qua việc thiết lập `$worker->reusePort = true;` để thay đổi cách lấy kết nối thành cách **vòng quay**, trong cách này, hạt nhân sẽ phân phối kết nối một cách gần như đồng đều cho tất cả các tiến trình, như vậy tất cả các tiến trình sẽ xử lý yêu cầu cùng một lúc.

### Sự hiểu lầm
Rất nhiều nhà phát triển nghĩ rằng tất cả các tiến trình tham gia xử lý yêu cầu sẽ làm tăng hiệu suất, thực tế không nhất thiết như vậy. Khi kinh doanh đủ đơn giản, số lượng tiến trình tham gia xử lý yêu cầu càng gần với số lõi CPU, thì lưu lượng server thường là cao nhất. Ví dụ, trên máy chủ có 4 lõi, khi số lượng tiến trình được thiết lập là 4, QPS của thử nghiệm "helloworld" thường là cao nhất. Nếu số lượng tiến trình tham gia xử lý vượt quá nhiều lõi CPU, chi phí ngữ cảnh tiến trình sẽ càng lớn, hiệu suất sẽ càng kém. Trong trường hợp kinh doanh thông thường với cơ sở dữ liệu, việc thiết lập số lượng tiến trình thành 3-6 lần số lõi CPU có thể làm tăng hiệu suất.
