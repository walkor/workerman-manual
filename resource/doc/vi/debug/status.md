# Xem trạng thái chạy

Chạy lệnh ```php start.php status``` có thể xem trạng thái hoạt động của WorkerMan, tương tự như sau:

```----------------------------------------------GLOBAL STATUS----------------------------------------------------
WorkerMan version:3.5.13          PHP version:5.5.9-1ubuntu4.24
thời gian bắt đầu:2018-02-03 11:48:20   chạy 112 ngày 2 giờ   
tải trung bình: 0, 0, 0            vòng lặp sự kiện: \Workerman\Events\Event
4 workers       11 processes
tên_worker        exit_status      exit_count
ChatBusinessWorker 0                0
ChatGateway        0                0
Register           0                0
WebServer          0                0
----------------------------------------------PROCESS STATUS---------------------------------------------------
pid	memory  listening                worker_name        connections send_fail timers  total_request qps    status
18306	2.25M   none                     ChatBusinessWorker 5           0         0       11            0      [idle]
18307	2.25M   none                     ChatBusinessWorker 5           0         0       8             0      [idle]
18308	2.25M   none                     ChatBusinessWorker 5           0         0       3             0      [idle]
18309	2.25M   none                     ChatBusinessWorker 5           0         0       14            0      [idle]
18310	2M      websocket://0.0.0.0:7272 ChatGateway        8           0         1       31            0      [idle]
18311	2M      websocket://0.0.0.0:7272 ChatGateway        7           0         1       26            0      [idle]
18312	2M      websocket://0.0.0.0:7272 ChatGateway        6           0         1       21            0      [idle]
18313	1.75M   websocket://0.0.0.0:7272 ChatGateway        5           0         1       16            0      [idle]
18314	1.75M   text://0.0.0.0:1236      Register           8           0         0       8             0      [idle]
18315	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [idle]
18316	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [idle]
----------------------------------------------PROCESS STATUS---------------------------------------------------
Tổng hợp	18M     -                        -                  54          0         4       138           0      [Summary]```

## Giải thích

### GLOBAL STATUS

Từ dòng này, chúng ta có thể thấy

- Phiên bản của WorkerMan: ```version:3.5.13```

- Thời gian bắt đầu ```2018-02-03 11:48:20```, chạy được ```112 ngày 2 giờ```

- Tải trung bình của máy chủ (load average): ```load average: 0, 0, 0``` trong 1, 5 và 15 phút gần đây

- Sử dụng thư viện sự kiện IO: ```event-loop:\Workerman\Events\Event```

- ```4 workers``` (3 loại tiến trình, bao gồm ChatGateway, ChatBusinessWorker, tiến trình Register, tiến trình WebServer)

- ```11 processes``` (tổng cộng 11 tiến trình)

- ```worker_name``` (tên của tiến trình worker)

- ```exit_status``` (mã trạng thái thoát của tiến trình worker)

- ```exit_count``` (số lần thoát với mã trạng thái đó)

Thường thì ```exit_status``` là 0 thể hiện việc thoát bình thường. Nếu có giá trị khác, có nghĩa là tiến trình đã thoát một cách bất thường và tạo ra thông báo lỗi tương tự như ```WORKER EXIT UNEXPECTED```. Thông tin lỗi sẽ được ghi vào tệp chỉ định trong [Worker::logFile](worker/log-file.md).

**Các mã ```exit_status``` thông thường và ý nghĩa tương ứng:**

- 0: chỉ số thoát bình thường sau khi khởi động lại, đây là hiện tượng bình thường. Lưu ý rằng việc gọi exit hoặc die trong mã lệnh cũng tạo ra mã thoát là 0 và tạo ra thông báo lỗi ```WORKER EXIT UNEXPECTED```, Workerman không cho phép mã lệnh kinh doanh gọi lệnh exit hoặc die.

- 9: cho biết tiến trình đã bị giết bằng tín hiệu SIGKILL. Mã thoát này chủ yếu xảy ra trong trường hợp dừng và khởi động lại mượt, nguyên nhân là do tiến trình con không phản ứng với tín hiệu khởi động lại của tiến trình chính trong khoảng thời gian quy định (ví dụ: mysql, curl kéo dài hoặc vòng lặp kinh doanh). Điều lưu ý là khi sử dụng lệnh kill trong dòng lệnh Linux để gửi tín hiệu SIGKILL đến tiến trình con cũng có thể gây ra mã thoát này.

- 11: chỉ số core dump của php, thường do sử dụng tiện ích không ổn định gây ra. Hãy tắt các tiện ích tương ứng trong tệp php.ini; một số trường hợp ít đáng kể khác có thể do lỗi của php, khi đó cần nâng cấp phiên bản php.

- 65280: chỉ số thoát này phát sinh khi mã lệnh kinh doanh gây lỗi nghiêm trọng, như gọi hàm không tồn tại, lỗi cú pháp, v.v. Thông tin lỗi cụ thể sẽ được ghi vào tệp chỉ định trong [Worker::logFile](worker/log-file.md) và có thể được tìm thấy trong tệp chỉ định trong [php.ini](https://php.net/manual/zh/ini.list.php) [error_log](https://php.net/manual/zh/errorfunc.configuration.php#ini.error-log) (nếu có tệp được chỉ định).

- 64000: chỉ số thoát này phát sinh khi mã lệnh kinh doanh bắn ra ngoại lệ, nhưng doanh nghiệp không bắt được ngoại lệ này, làm cho tiến trình thoát. Nếu Workerman được chạy trong chế độ gỡ lỗi, ngăn xếp cuộc gọi của ngoại lệ sẽ được ghi vào cửa sổ terminal; nếu Workerman được chạy trong chế độ daemon, ngăn xếp cuộc gọi của ngoại lệ sẽ được ghi vào tệp chỉ định trong [Worker::stdoutFile](worker/stdout-file.md).

## PROCESS STATUS

pid: process id

memory: lượng bộ nhớ hiện tại mà quá trình này chiếm (không bao gồm việc sử dụng bộ nhớ của tệp thực thi php)

listening: giao thức truyền tải và cổng đang nghe. Nếu không nghe bất kỳ cổng nào, chỉ hiển thị none. Xem [Worker class construct function](worker/construct.md) để biết thêm chi tiết.

worker_name: tên dịch vụ mà quá trình này đang chạy, xem [worker name attribute](worker/name.md) của Worker class.

connections: số lượng thực thể kết nối TCP hiện tại của quá trình này, trong đó mỗi thực thể kết nối bao gồm các thực thể TcpConnection và AsyncTcpConnection. Giá trị này thể hiện số lượng hiện thời và không phải là tổng cộng. Lưu ý: nếu thực thể kết nối đóng, nếu số lượng tương ứng không giảm, có thể do mã kinh doanh giữ đối tượng $connection, làm cho thực thể kết nối không thể hủy.

total_request: Đại diện cho tổng số yêu cầu mà quá trình này đã nhận từ khi khởi động. Số yêu cầu này không chỉ bao gồm yêu cầu từ máy khách, mà còn bao gồm yêu cầu giao tiếp nội bộ của Workerman, ví dụ như yêu cầu giao tiếp giữa Gateway và BusinessWorker trong cấu trúc GatewayWorker. Đây là giá trị tích lũy.

send_fail: Số lần quá trình này gửi dữ liệu cho máy khách mà thất bại, nguyên nhân thất bại thường là do kết nối với máy khách bị đóng, giá trị này không phải là 0 thì thường không phải là trạng thái bình thường, xem [lý do của send_fail trong status](../faq/about-send-fail.md). Đây là giá trị tích lũy.

timers: Số lượng bộ hẹn giờ hoạt động của quá trình này (không bao gồm bộ hẹn giờ đã bị xóa hoặc bộ hẹn giờ một lần đã chạy). Lưu ý: Đối tượng caching này yêu cầu phiên bản Workerman >= 3.4.7. Giá trị này thể hiện số hiện tại và không phải là tổng cộng.

qps: Số lượng yêu cầu mạng mà quá trình này nhận được mỗi giây, lưu ý: chỉ khi thêm ```-d``` vào ```status``` thì các mục này mới có thể được thống kê, không có -d thì hiển thị 0. Đối tượng caching này yêu cầu phiên bản Workerman >= 3.5.2. Giá trị này thể hiện số hiện tại và không phải là tổng cộng.

trạng thái: trạng thái của quá trình, nếu idle chỉ tiêu không hoạt động, nếu bận rộn chỉ tiêu đang bận rộn. Lưu ý: nếu quá trình liên tục bận rộn, có thể đã xảy ra tình trạng chặn doanh nghiệp hoặc vòng lặp chết, cần phải kiểm tra theo [cách gỡ lỗi quá trình bận rộn](busy-process.md). Lưu ý: Đối tượng caching này yêu cầu phiên bản Workerman >= 3.5.0.

## Nguyên lý hoạt động

Sau khi chạy lệnh, quá trình chính sẽ gửi tín hiệu ```SIGUSR2``` đến tất cả các quá trình worker, sau đó, tệp lệnh sẽ đi vào giai đoạn ngủ ngắn để đợi kết quả thống kê trạng thái từ các quá trình worker. Lúc này, quá trình worker đang rảnh sẽ viết trạng thái chạy của chính mình (số kết nối, số yêu cầu, v.v.) vào tệp đĩa cụ thể, trong khi quá trình worker đang xử lý quy trình kinh doanh sẽ chờ cho đến khi quá trình kinh doanh được xử lý xong mới ghi trạng thái của chính mình xuống tệp. Sau giai đoạn ngủ ngắn, tệp lệnh status sẽ bắt đầu đọc trạng thái từ các tệp đĩa và hiển thị kết quả trên giao diện điều khiển.

## Chú ý

Khi kiểm tra trạng thái, có thể thấy một số quá trình hiển thị đang bận rộn, điều này do quá trình đang bận rộn xử lý quy trình kinh doanh (ví dụ: chờ lâu để gửi yêu cầu curl hoặc cơ sở dữ liệu, hoặc chạy vòng lặp lớn), không thể báo cáo trạng thái, dẫn đến hiển thị là bận rộn.

Khi gặp vấn đề này, cần kiểm tra mã kinh doanh, xem xét nơi có xảy ra chặn doanh nghiệp trong thời gian dài và đánh giá xem thời gian chặn có phù hợp hay không, nếu không phù hợp, cần phải kiểm tra mã kinh doanh theo [cách gỡ lỗi quá trình bận rộn](busy-process.md).
