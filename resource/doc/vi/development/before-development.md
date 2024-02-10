# Đọc trước khi phát triển

Để phát triển ứng dụng bằng WorkerMan, bạn cần hiểu về những nội dung sau:

## I. Sự khác biệt giữa phát triển bằng WorkerMan và phát triển PHP thông thường

Ngoài việc không thể sử dụng trực tiếp các biến và hàm liên quan đến giao thức HTTP, việc phát triển bằng WorkerMan không có nhiều sự khác biệt so với phát triển PHP thông thường.

### 1. Giao thức ứng dụng khác nhau
* Phát triển PHP thông thường thường dựa trên giao thức ứng dụng HTTP, máy chủ web đã hoàn tất việc phân tích giao thức cho người phát triển
* WorkerMan hỗ trợ nhiều loại giao thức, hiện tại hỗ trợ sẵn giao thức HTTP, WebSocket, v.v. WorkerMan khuyến khích người phát triển sử dụng giao thức tùy chỉnh đơn giản hơn

###[ Phần HTTP](../http/request.md) để phát triển theo giao thức HTTP

### 2. Chu kỳ yêu cầu khác nhau
* Trong ứng dụng web PHP, sau mỗi lần yêu cầu, tất cả biến và tài nguyên sẽ được giải phóng
* Ứng dụng được phát triển bằng WorkerMan sẽ ở trong bộ nhớ sau khi được tải và phân tích lần đầu, điều này làm cho việc định nghĩa lớp, đối tượng toàn cục, thành viên tĩnh của lớp không bị giải phóng, từ đó thuận tiện cho việc sử dụng lại trong tương lai

### 3. Chú ý tránh định nghĩa lớp và hằng số trùng lặp
* Vì WorkerMan sẽ cache file PHP sau khi biên dịch, nên cần tránh việc require/include nhiều lần cùng một file định nghĩa lớp hoặc hằng số. Đề xuất sử dụng require_once/include_once để tải tệp.

### 4. Chú ý giải phóng tài nguyên kết nối trong chế độ Singleton
* Vì WorkerMan không giải phóng đối tượng toàn cục và thành viên tĩnh của lớp sau mỗi lần yêu cầu, trong trường hợp Singleton như cơ sở dữ liệu, thường sẽ giữ thể hiện cơ sở dữ liệu (bao gồm kết nối socket cơ sở dữ liệu) trong thành viên tĩnh của cơ sở dữ liệu, làm cho WorkerMan có thể tái sử dụng kết nối socket cơ sở dữ liệu này suốt vòng đời của quy trình. Cần chú ý rằng khi máy chủ cơ sở dữ liệu phát hiện một kết nối đã đóng trong một khoảng thời gian nhất định sẽ tự động đóng kết nối socket này, khi sử dụng lại thể hiện cơ sở dữ liệu này lúc đó sẽ báo lỗi (thông báo lỗi tương tự như "mysql gone away"). WorkerMan cung cấp [lớp cơ sở dữ liệu](../components/workerman-mysql.md), với chức năng kết nối lại, người phát triển có thể sử dụng trực tiếp.

### 5. Chú ý không sử dụng câu lệnh exit, die
* WorkerMan chạy trong chế độ dòng lệnh PHP, khi gọi câu lệnh thoát exit, die sẽ dẫn đến quá trình hiện tại kết thúc. Mặc dù quá trình con sẽ tự động tạo ra một quá trình con tương tự ngay sau khi quá trình con kết thúc, nhưng vẫn có thể ảnh hưởng đến doanh nghiệp.

### 6. Cần khởi động lại dịch vụ để áp dụng các thay đổi
Do WorkerMan là một ứng dụng trong bộ nhớ, khi định nghĩa lớp PHP hoặc hàm, nó chỉ cần tải một lần, không cần đọc lại từ đĩa, vì vậy mỗi lần sửa đổi mã doanh nghiệp cần khởi động lại để áp dụng.


## 二、Các Khái Niệm Cơ Bản Cần Hiểu

### 1. Giao thức TCP tầng truyền
TCP là một loại giao thức tầng truyền hướng kết nối, đáng tin cậy, dựa trên giao thức IP. Một điểm đặc biệt quan trọng của giao thức tầng truyền TCP là TCP dựa trên luồng dữ liệu, yêu cầu từ phía máy khách sẽ liên tục gửi đến máy chủ, dữ liệu nhận được từ máy chủ có thể không phải là một yêu cầu hoàn chỉnh, cũng có thể là nhiều yêu cầu nối tiếp nhau. Điều này cần phải phân biệt ranh giới của mỗi yêu cầu trong luồng dữ liệu liên tục này. Còn giao thức tầng ứng dụng chủ yếu là định nghĩa một tập luật để xác định ranh giới của yêu cầu, tránh sự lộn xộn trong dữ liệu yêu cầu.

### 2. Giao thức tầng ứng dụng
Giao thức tầng ứng dụng (application layer protocol) định nghĩa cách các tiến trình ứng dụng chạy trên các hệ thống kết nối (máy khách, máy chủ) truyền đi các thông điệp cho nhau, ví dụ như HTTP, WebSocket cũng thuộc giao thức tầng ứng dụng. Ví dụ về một giao thức tầng ứng dụng đơn giản có thể như sau: ```{"module":"user","action":"getInfo","uid":456}\n"```. Giao thức này đánh dấu kết thúc yêu cầu bằng ```"\n"``` (lưu ý rằng ```"\n"``` ở đây đại diện cho ký tự xuống dòng), phần thân tin nhắn là chuỗi ký tự.

### 3. Kết nối ngắn
Kết nối ngắn là khi có hoạt động trao đổi dữ liệu giữa hai bên, một kết nối sẽ được thiết lập, và khi dữ liệu được gửi xong, kết nối sẽ đóng ngay sau đó, có nghĩa là mỗi lần kết nối chỉ hoàn thành một nhiệm vụ gửi.

*Quy trình phát triển ứng dụng kết nối ngắn có thể tham khảo trong chương quy trình cơ bản*

### 4. Kết nối dài
Kết nối dài, có nghĩa là trên một kết nối có thể gửi liên tục nhiều gói tin. 

Lưu ý: Ứng dụng kết nối dài phải có [đập thăm](../faq/heartbeat.md), nếu không, kết nối có thể bị tường lửa tắt do không hoạt động trong thời gian dài. Kết nối dài thường được sử dụng trong trường hợp liên tục vận hành, truyền thông trực tiếp. Mỗi kết nối TCP đều cần phải thực hiện ba bước bắt tay, điều này mất thời gian, nếu mỗi lần hoạt động đều kết nối lại trước khi thực hiện, tốc độ xử lý sẽ giảm rất nhiều. Do đó, kết nối dài sẽ không đóng sau mỗi lần hoạt động, lần xử lý sau cùng sẽ gửi gói tin trực tiếp, không cần thiết lập lại kết nối TCP. Ví dụ: kết nối với cơ sở dữ liệu sử dụng kết nối dài, nếu sử dụng kết nối ngắn và truyền thông tạo ra lỗi socket, và việc tạo socket thường là lãng phí tài nguyên.

*Khi cần tích cực đẩy dữ liệu đến máy khách, ví dụ như chat, trò chơi thời gian thực, ứng dụng đẩy thông báo từ điện thoại như hình ảnh, âm thanh cần sử dụng kết nối dài.*

*Quy trình phát triển ứng dụng kết nối dài có thể tham khảo trong quy trình phát triển Gateway/Worker*

### 5. Khởi động lại mềm
Thường thì quy trình khởi động là tắt tất cả các tiến trình rồi bắt đầu tạo mới các tiến trình dịch vụ. Trong quá trình này có một khoảng thời gian ngắn không có tiến trình nào đang phục vụ ngoại trừ việc dịch vụ tạm thời không sẵn sàng, điều này sẽ dẫn đến tạm thời không sử dụng được dịch vụ, và khi có vấn đề về cùng một lúc trong thời gian phục vụ cao điểm sẽ dẫn đến yêu cầu thất bại.

Khởi động lại mềm không phải là tắt hết tất cả các tiến trình mà là từng tiến trình một, mỗi khi tắt một tiến trình thì sẽ ngay lập tức khởi tạo một tiến trình mới thay thế, cho đến khi tất cả các tiến trình cũ đều được thay thế. 

WorkerMan có thể sử dụng lệnh ```php your_file.php reload``` để cập nhật ứng dụng mà không ảnh hưởng tới chất lượng dịch vụ.

**Lưu ý:** Chỉ có các tệp được tải trong các callback on{...} sau khi khởi động lại mềm thì mới được cập nhật tự động, các tệp được tải trực tiếp trong tệp khởi động hoặc mã code cứng không được cập nhật khi chạy lệnh reload.

## Ba, Phân biệt giữa tiến trình chính và tiến trình con
Có ý thức cần phải chú ý tới việc mã code chạy trong tiến trình chính hay tiến trình con, nói chung thì mọi code chạy trước khi gọi ```Worker::runAll();``` thì sẽ chạy trong tiến trình chính, code chạy trong callback onXXX sẽ chạy trong tiến trình con. Chú ý rằng mã code viết sau ```Worker::runAll();``` sẽ không được thực thi bao giờ.

Ví dụ về mã code:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Chạy trong tiến trình chính
$tcp_worker = new Worker("tcp://0.0.0.0:2347");
// Gán giá trị chạy trong tiến trình chính
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Phần này chạy trong tiến trình con
    $connection->send('hello ' . $data);
};

Worker::runAll();
```

**Chú ý:** Không nên khởi tạo kết nối tới cơ sở dữ liệu, memcache, redis và các tài nguyên kết nối khác trong tiến trình chính, vì kết nối được khởi tạo trong tiến trình chính có thể được tự động thừa hưởng bởi tiến trình con (đặc biệt là khi sử dụng singleton), tất cả các tiến trình sẽ giữ cùng một kết nối, dữ liệu trả về từ máy chủ thông qua kết nối này có thể đọc được trên nhiều tiến trình, dẫn đến lộn xộn dữ liệu. Tương tự, nếu một tiến trình nào đó đóng kết nối (ví dụ khi chạy ở chế độ daemon, tiến trình chính sẽ thoát và đóng kết nối), sẽ dẫn đến đóng kết nối trên tất cả tiến trình con, và có thể xảy ra lỗi không thể dự đoán và không ngừng, ví dụ lỗi mysql gone away. Khuyến nghị khởi tạo tài nguyên kết nối trong onWorkerStart.
