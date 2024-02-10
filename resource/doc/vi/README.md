# Lời nói đầu

**Workerman, một máy chủ ứng dụng PHP hiệu suất cao**

## Workerman là gì?
Workerman là một máy chủ ứng dụng PHP mã nguồn mở hiệu suất cao được phát triển hoàn toàn bằng PHP.

Workerman không phải là một framework MVC, mà là một framework dịch vụ phổ quát hơn, bạn có thể sử dụng nó để phát triển proxy TCP, proxy VPN, server trò chơi, server email, server FTP, thậm chí là phát triển một phiên bản PHP của Redis, cơ sở dữ liệu, Nginx, PHP-FPM v.v. Workerman có thể coi là một đổi mới trong lĩnh vực PHP, giúp các nhà phát triển hoàn toàn thoát khỏi việc PHP chỉ có thể sử dụng cho web.

Workerman thực tế tương tự như một phiên bản PHP của Nginx, nó cũng sử dụng nhiều tiến trình + Epoll + IO không chặn. Mỗi tiến trình của Workerman có thể duy trì hàng ngàn kết nối đồng thời. Bởi vì nó luôn ở trong bộ nhớ, không phụ thuộc vào các máy chủ như Apache, Nginx, PHP-FPM, nó có hiệu suất cực kỳ cao. Đồng thời hỗ trợ TCP, UDP, UNIXSOCKET, hỗ trợ kết nối lâu dài, hỗ trợ Websocket, HTTP, WSS, HTTPS và nhiều thông tin giao thức tùy chỉnh khác. Workerman còn có khả năng hẹn giờ, client socket không đồng bộ, Redis không đồng bộ, HTTP không đồng bộ, hàng loạt các thành phần hiệu suất cao khác.

## Một số hướng ứng dụng của Workerman
Workerman không giống như các framework MVC truyền thống, nó không chỉ có thể sử dụng cho phát triển web, mà còn có một lĩnh vực ứng dụng rộng lớn hơn, chẳng hạn như giao tiếp trực tiếp, IoT, trò chơi, quản lý dịch vụ, server hoặc middleware khác, điều này rõ ràng nâng cao tầm nhìn của nhà phát triển PHP. Hiện nay, những nhà phát triển PHP trong các lĩnh vực này rất khan hiếm. Nếu muốn có lợi thế về công nghệ trong lĩnh vực PHP, không hài lòng với việc thêm, sửa, xoá hàng ngày, hoặc muốn phát triển theo hướng kiến trúc hệ thống hoặc chuyên gia công nghệ, Workerman là một framework rất đáng để học. Chúng tôi khuyến nghị các nhà phát triển không chỉ sử dụng, mà còn có thể phát triển dự án mã nguồn mở của riêng mình dựa trên Workerman, nâng cao kỹ năng và tăng cường ảnh hưởng của bản thân, ví dụ như [Beanbun mô hình spider mạng lưới đa tiến trình](https://github.com/kiddyuchina/Beanbun) là một ví dụ tốt, chỉ sau một thời gian ngắn đã nhận được rất nhiều đánh giá tích cực.

Một số hướng ứng dụng của Workerman bao gồm:

1. Loại giao tiếp trực tiếp
  Ví dụ: trò chuyện trực tuyến trên trang web, push tin nhắn trực tiếp, ứng dụng nhỏ WeChat, push tin nhắn cho ứng dụng di động, push tin nhắn cho phần mềm PC, v.v.
  [[Ví dụ: phòng trò chuyện workerman-chat](https://www.workerman.net/workerman-chat), [web push message](https://www.workerman.net/web-sender), [phòng trò chuyện tadpole](https://www.workerman.net/workerman-todpole)]

2. Loại IoT
  Ví dụ: Giao tiếp với máy in, giao tiếp với vi điều khiển, vòng đeo tay thông minh, nhà thông minh, xe đạp chia sẻ, v.v.
  [Ví dụ khách hàng như Yilian Cloud, Ebaoshi era]

3. Loại server game
  Ví dụ: trò chơi bài, trò chơi MMORPG, v.v.
  [[Ví dụ về browserquest-php](https://www.workerman.net/browserquest)]

4. Dịch vụ HTTP
  Ví dụ: Phát triển giao diện HTTP hiệu suất cao, website hiệu suất cao. Nếu muốn phát triển dịch vụ liên quan đến HTTP, chúng tôi rất khuyến khích [webman](https://github.com/walkor/webman)

5. Dịch vụ SOA
  Sử dụng Workerman để đóng gói các đơn vị chức năng khác nhau của doanh nghiệp hiện có, cung cấp một giao diện thống nhất cho bên ngoài dưới dạng dịch vụ, đạt được mục tiêu giảm độ ràng buộc, dễ bảo trì, cao sẵn có, dễ mở rộng hệ thống. [[Ví dụ workerman-json-rpc](https://github.com/walkor/workerman-jsonrpc), [workerman-thrift](https://github.com/walkor/workerman-thrift)]

6. Các phần mềm server khác
  Ví dụ: [GatewayWorker](https://www.workerman.net/doc/gateway-worker), [PHPSocket.IO](https://www.workerman.net/phpsocket_io), [proxy HTTP](https://github.com/walkor/php-http-proxy), [proxy socks5](https://github.com/walkor/php-socks5), [thành phần truyền thông phân tán](https://github.com/walkor/Channel), [thành phần chia sẻ biến phân tán](https://github.com/walkor/GlobalData), [hàng đợi tin nhắn](https://github.com/walkor/workerman-queue), DNS server, WebServer, CDN server, FTP server v.v.

7. Các thành phần
  Ví dụ: [Redis không đồng bộ](components/workerman-redis.md), [client HTTP không đồng bộ](components/workerman-http-client.md), [client mqtt IoT](components/workerman-mqtt.md), hàng đợi tin nhắn [workerman/redis-queue](components/workerman-redis-queue.md), [workerman/stomp](components/workerman-stomp.md), [workerman/rabbitmq](components/workerman-rabbitmq.md), [thành phần giám sát tệp](components/file-monitor.md), cũng như rất nhiều framework thành phần của bên thứ ba khác.

Rõ ràng, là rất khó để framework MVC truyền thống có thể thực hiện các chức năng trên, vì vậy đó cũng là lý do mà Workerman ra đời.

## Triết lý của Workerman
Đơn giản, ổn định, hiệu suất cao, phân tán.

### **Đơn giản**
Nhỏ là xinh đẹp, lõi của Workerman rất đơn giản, chỉ vài tập tin PHP và chỉ mở ra vài giao diện, rất dễ học. Tất cả các chức năng khác được mở rộng thông qua các thành phần.

Workerman có tài liệu hoàn chỉnh + trang chủ uy tín + cộng đồng sôi động + nhiều nhóm QQ có hàng nghìn thành viên + rất nhiều thành phần hiệu suất cao + hàng loạt ví dụ, tất cả những điều này giúp các nhà phát triển sử dụng dễ dàng hơn.

### **Ổn định**
Workerman đã mã nguồn mở từ nhiều năm, được sử dụng rộng rãi bởi nhiều công ty lớn, cực kỳ ổn định. Một số dịch vụ không được khởi động lại trong hơn 2 năm vẫn hoạt động mạnh. Không có core dump, không có rò rỉ bộ nhớ, không có lỗi.

### **Hiệu suất cao**
Do việc luôn ở trong bộ nhớ, không phụ thuộc vào apache/nginx/php-fpm, không có chi phí liên lạc giữa các máy chủ và PHP, không có chi phí khởi tạo và hủy bỏ tất cả mọi thứ mỗi yêu cầu, nó có hiệu suất cực kỳ cao, so với framework MVC truyền thống, hiệu suất cao hơn vài chục lần, dưới áp lực của máy chủ PHP7, cái nào cao hơn cả nginx độc lập.

### **Phân tán**
Hiện nay không còn thời đại của một mình một dạo, hiệu suất của một máy chủ đơn lẻ có hạn chế dù có mạnh mẽ thế nào, triển khai nhiều máy chủ phân tán mới là chìa khóa. Workerman cung cấp một giải pháp liên lạc phân tán với kết nối lâu dài [GatewayWorker framework](https://doc2.workerman.net), thêm máy chủ chỉ cần cấu hình đơn giản rồi khởi động, mã nguồn không cần thay đổi, khả năng chịu tải của hệ thống tăng lên gấp đôi. Nếu bạn đang phát triển ứng dụng kết nối TCP lâu dài, chúng tôi khuyến nghị sử dụng trực tiếp [GatewayWorker](https://doc2.workerman.net), đó là một bọc của Workerman, cung cấp giao diện phong phú và khả năng xử lý phân tán mạnh mẽ cho các ứng dụng kết nối lâu dài.

## Phạm vi của hướng dẫn này
Phiên bản Workerman 3.x - 4.x

## Người dùng Windows (cần đọc)
Workerman hỗ trợ cả hệ thống linux và hệ thống windows. Phiên bản windows của Workerman **không phụ thuôc vào bất kỳ phần mở rộng nào**, chỉ cần cấu hình biến môi trường PHP đúng là được, **quy trình cài đặt và các điều cần lưu ý cho phiên bản windows xem tại [Cần đọc cho người dùng windows](https://www.workerman.net/windows)**.

## Máy khách
Giao thức liên lạc của WorkerMan là mở, có thể tùy chỉnh, do đó, lý thuyết WorkerMan có thể tương tác với bất kỳ máy khách nào sử dụng bất kỳ giao thức nào. Khi người dùng phát triển máy khách, họ có thể hoàn tất việc tương tác với máy chủ dựa trên giao thức liên lạc tương ứng.


