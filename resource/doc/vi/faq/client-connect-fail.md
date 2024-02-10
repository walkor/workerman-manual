# Lý do kết nối khách hàng thất bại

Khi kết nối khách hàng thất bại, thường có hai lỗi phổ biến là ```connection refuse``` (từ chối kết nối) và ```connection timeout``` (kết nối quá thời gian).

## Kết nối bị từ chối (connection refuse)

Thường có các lý do sau:
1. Khách hàng kết nối đến cổng sai
2. Khách hàng kết nối đến tên miền hoặc địa chỉ IP sai
3. Nếu khách hàng sử dụng tên miền để kết nối, có thể tên miền đang chỉ đến địa chỉ IP của máy chủ sai
4. Máy chủ sử dụng CDN hoặc proxy tăng tốc, dẫn đến địa chỉ IP thực tế của kết nối không khớp với địa chỉ IP dự kiến
5. Máy chủ chưa khởi động hoặc cổng không được lắng nghe
6. Sử dụng phần mềm proxy mạng
7. Địa chỉ IP lắng nghe của máy chủ và địa chỉ truy cập không ở trong cùng một dải địa chỉ. Ví dụ, máy chủ lắng nghe 127.0.0.1, thì khách hàng chỉ có thể kết nối qua 127.0.0.1, không thể kết nối qua địa chỉ IP trong mạng cục bộ hoặc địa chỉ IP ngoại tuyến. Đề xuất đặt địa chỉ lắng nghe là 0.0.0.0, điều này có nghĩa là máy tính cục bộ, mạng nội bộ và mạng ngoại tuyến đều có thể kết nối.

## Kết nối quá thời gian (connection timeout)

Thường có các lý do sau:
1. Tường lửa máy chủ ngăn chặn các kết nối, có thể tạm thời tắt tường lửa để kiểm tra
2. Đối với máy chủ đám mây, nhóm bảo mật cũng có thể ngăn chặn các kết nối, cần mở cổng tương ứng trong giao diện quản lý
3. Nếu sử dụng các bảng điều khiển như Baota, cần mở cổng tương ứng trong Baota
4. Máy chủ không tồn tại hoặc chưa khởi động
5. Nếu khách hàng sử dụng tên miền để kết nối, có thể tên miền đang chỉ đến địa chỉ IP của máy chủ sai
6. IP mà khách hàng truy cập là IP trong mạng nội bộ của máy chủ và các thiết bị không ở trong cùng một mạng cục bộ

## Không thể chỉ định địa chỉ yêu cầu (cannot assign requested address)

**Khi là khách hàng**, mỗi lần khởi tạo một kết nối đều cần sử dụng một cổng tạm thời cục bộ, một máy chủ mặc định có khoảng 2-3 vạn cổng tạm thời, nếu số lượng kết nối tới máy chủ cụ thể vượt quá giá trị này, sẽ không thể chỉ định được cổng có sẵn và sẽ gây ra lỗi này.
Có thể tăng số lượng cổng tạm thời cục bộ bằng cách thay đổi tham số nhân hệ thống `/etc/sysctl.conf` như `net.ipv4.ip_local_port_range`, ví dụ, cài đặt thành `10000 65535` (phạm vi cổng cục bộ được thiết lập thành 10000 65535, có nghĩa là số cổng cục bộ được tăng lên 55535), chạy lệnh `sysctl -p` để có hiệu lực.
Ngoài ra, sau khi kết nối bị ngắt, kết nối vẫn ở trạng thái TIME_WAIT và vẫn sẽ sử dụng cổng cục bộ tương ứng trong một khoảng thời gian, nghĩa là sự cố có thể xảy ra khi có lượng lớn kết nối ngắn (vượt quá 2-3 vạn), nếu đó là trường hợp, có thể giải quyết bằng cách cài đặt tự động thu hồi trạng thái TIME_WAIT của hệ điều hành, xem thêm tại [tinh chỉnh hệ điều hành](https://www.workerman.net/doc/workerman/appendices/kernel-optimization.html).

> **Chú ý**
> Hạn chế số cổng tạm thời cục bộ chỉ áp dụng cho khách hàng, máy chủ không có giới hạn cổng tạm thời cục bộ, miễn là tài nguyên đủ, số lượng kết nối máy chủ có thể coi như là không giới hạn.

## Lỗi khác
Nếu lỗi không phải là ```connection refuse``` và ```connection timeout```, thì thường có các lý do sau:

**1. Khách hàng sử dụng giao thức truyền thông không phù hợp với máy chủ.**
Ví dụ, máy chủ sử dụng giao thức truyền thông HTTP, khách hàng sử dụng giao thức truyền thông WebSocket sẽ không thể kết nối. Nếu khách hàng sử dụng giao thức truyền thông WebSocket, thì máy chủ phải cũng sử dụng giao thức truyền thông WebSocket. Nếu máy chủ là dịch vụ giao thức HTTP, thì khách hàng phải sử dụng giao thức truyền thông HTTP.

Nguyên lý ở đây tương tự như khi bạn muốn giao tiếp với người Anh, bạn cần sử dụng tiếng Anh. Nếu muốn giao tiếp với người Nhật, bạn cần sử dụng tiếng Nhật. Ở đây, ngôn ngữ tương tự như giao thức truyền thông, cả hai bên (khách hàng và máy chủ) phải sử dụng cùng một ngôn ngữ để giao tiếp, nếu không sẽ không thể truyền thông.

**Các lỗi thông thường do không phù hợp với giao thức truyền thông:**

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: Unexpected response code: xxx

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: net::ERR_INVALID_HTTP_RESPONSE

**Phương pháp giải quyết:**
Từ hai thông báo lỗi ở trên, có thể thấy rằng khách hàng đang sử dụng kết nối ws là giao thức truyền thông WebSocket. Máy chủ cũng cần sử dụng giao thức truyền thông WebSocket để có thể kết nối. Đoạn mã nguồn lắng nghe trên máy chủ cần chỉ định giao thức truyền thông WebSocket như sau:

Nếu đó là gatewayWorker, đoạn mã lắng nghe tương tự như sau
```php
// giao thức websocket, để khách hàng có thể kết nối qua ws://... . xxxx là cổng không cần thay đổi
$gateway = new Gateway('websocket://0.0.0.0:xxxx');
```
Nếu đó là Workerman thì là
```php
// giao thức websocket, để khách hàng có thể kết nối qua ws://... . xxxx là cổng không cần thay đổi
$worker = new Worker('websocket://0.0.0.0:xxxx');
```

