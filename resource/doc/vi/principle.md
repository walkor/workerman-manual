# Nguyên lý

### Worker giải thích
Worker là một trong những container cơ bản nhất trong Workerman. Worker có thể mở nhiều tiến trình để lắng nghe cổng và sử dụng giao thức cụ thể để giao tiếp, tương tự như nginx lắng nghe một cổng nào đó. Mỗi tiến trình Worker đều hoạt động độc lập, sử dụng Epoll (yêu cầu cài đặt event extension) và IO không chặn. Mỗi tiến trình Worker đều có thể kết nối với hàng ngàn kết nối từ khách hàng, và xử lý dữ liệu gửi từ những kết nối này. Tiến trình chính chỉ để duy trì sự ổn định, chỉ giám sát các tiến trình con, không chịu trách nhiệm nhận dữ liệu hoặc làm bất kỳ logic nào.

### Mối quan hệ giữa khách hàng và tiến trình Worker
![workerman master woker模型](images/Worker.png)

### Mối quan hệ giữa tiến trình chính và tiến trình Worker con
![workerman master woker模型](images/Worker2.png)

**Điểm đặc biệt:**

Từ hình vẽ, chúng ta có thể thấy mỗi Worker duy trì các kết nối của họ và có thể dễ dàng thực hiện giao tiếp thời gian thực giữa khách hàng và máy chủ. Dựa trên mô hình này, chúng ta có thể dễ dàng triển khai một số yêu cầu phát triển cơ bản, chẳng hạn như máy chủ HTTP, máy chủ Rpc, báo cáo dữ liệu thời gian thực từ các thiết bị thông minh, đẩy dữ liệu từ máy chủ, máy chủ trò chơi, máy chủ ứng dụng WeChat Mini, v.v.
