# Tinh chỉnh nhân Linux

Để hỗ trợ đa nhiệm mạnh mẽ hơn, ngoài việc phải [cài đặt tiện ích mở rộng sự kiện](../install/install.md) thì tinh chỉnh nhân Linux cũng là **rất rất quan trọng**. Mỗi mục tinh chỉnh dưới đây đều rất rất quan trọng, xin vui lòng thực hiện từng bước một.

**Giải thích về các tham số:**

> **max-file**: Đại diện cho số lượng tay cầm tập tin có thể mở được ở cấp độ **hệ thống**. Đối với toàn bộ hệ điều hành, không phải là đối với người dùng.
> 
> **ulimit -n**: Đại diện cho số lượng tay cầm tập tin có thể mở được ở cấp độ **tiến trình**. Điều này áp dụng vào số lượng tay cầm tập tin có sẵn cho người dùng của `shell` hiện tại và tiến trình khởi động của họ.

Kiểm tra số lượng tay cầm tập tin có thể mở được ở cấp độ **hệ thống**: `cat /proc/sys/fs/file-max`

Mở tập tin /etc/sysctl.conf và thêm các cài đặt sau:
```conf
# Tham số này xác định số lượng TIME_WAIT của hệ thống, nếu vượt quá giá trị mặc định thì sẽ bị xoá ngay lập tức
net.ipv4.tcp_max_tw_buckets = 20000
# Định nghĩa số lượng lớn nhất của hàng đợi lắng nghe cho mỗi cổng trong toàn hệ thống
net.core.somaxconn = 65535
# Số lượng kết nối chờ xác nhận từ phía khác mà hệ thống có thể giữ trong hàng đợi
net.ipv4.tcp_max_syn_backlog = 262144
# Khi tốc độ nhận gói dữ liệu trên mỗi giao diện mạng nhanh hơn tốc độ xử lý của hạt nhân, số lượng gói dữ liệu tối đa có thể đưa vào hàng đợi
net.core.netdev_max_backlog = 30000
# Tùy chọn này sẽ dẫn đến việc các khách hàng trong mạng NAT bị hết hạn. Khuyến nghị đặt là 0. Từ phiên bản kernel 4.12 trở đi, Linux đã loại bỏ cấu hình tcp_tw_recycle. Nếu báo lỗi "No such file or directory", vui lòng bỏ qua
net.ipv4.tcp_tw_recycle = 0
# Số lượng tập tin mà toàn bộ quá trình hệ thống có thể mở được
fs.file-max = 6815744
# Kích thước bảng theo dõi tường lửa. Lưu ý: Nếu tường lửa không được bật, sẽ báo lỗi: "net.netfilter.nf_conntrack_max" is an unknown key, vui lòng bỏ qua
net.netfilter.nf_conntrack_max = 2621440
net.ipv4.ip_local_port_range = 10240 65000
```
Chạy `sysctl -p` để áp dụng ngay lập tức.

**Lưu ý:**

/etc/sysctl.conf có rất nhiều tùy chọn có thể được thiết lập, các tùy chọn khác có thể được thiết lập dựa trên môi trường cụ thể của bạn

## Mở số tập tin

Cài đặt số tập tin có thể mở trên hệ thống, giải quyết vấn đề ```too many open files``` trong môi trường đa nhiệm cao. Tùy chọn này ảnh hưởng trực tiếp đến số kết nối khách hàng mà một tiến trình có thể chứa. 

Mở tập tin mềm là tham số của hệ thống Linux, ảnh hưởng đến số lượng tối đa tay cầm tập tin mà một tiến trình có thể mở, giá trị này ảnh hưởng đến số lượng kết nối mà một tiến trình có thể duy trì như trong ứng dụng chat. Chạy `ulimit -n` để xem giá trị tham số này, nếu là 1024, tức là mỗi tiến trình chỉ có thể duy trì tối đa 1024 kết nối cùng một lúc hoặc ít hơn (do có tay cầm tập tin khác đã được mở). Nếu mở 4 tiến trình để duy trì kết nối người dùng, thì ứng dụng có thể duy trì tối đa 4*1024 kết nối, tức là có thể hỗ trợ tối đa 4*1024 người dùng trực tuyến. Bạn có thể tăng giá trị này để ứng dụng có thể duy trì nhiều kết nối hơn.

**Có 3 cách để sửa đổi số tập tin mở mềm:**

Cách thứ nhất: Chạy `ulimit -HSn 102400` trực tiếp trong terminal, sau đó khởi động lại workerman.

Điều này chỉ có hiệu lực trong terminal hiện tại, sau khi thoát khỏi terminal, số tập tin mở sẽ trở về giá trị mặc định.

Cách thứ hai: Thêm dòng `ulimit -HSn 102400` vào cuối tập tin `/etc/profile`, điều này sẽ tự động thực hiện mỗi khi đăng nhập vào terminal. Sau khi thay đổi, cần khởi động lại workerman.

Cách thứ ba: Để giá trị số tập tin mở có hiệu lực vĩnh viễn, bạn phải sửa tập tin cấu hình: `/etc/security/limits.conf`. Thêm dòng sau vào cuối tập tin:

``` 
* soft nofile 1024000
* hard nofile 1024000
root soft nofile 1024000
root hard nofile 1024000
```

Phương pháp này yêu cầu khởi động lại máy chủ để có hiệu lực.
