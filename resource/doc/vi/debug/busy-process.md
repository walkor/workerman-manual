# Debug quá trình xử lý hàng đợi bận rộn
Đôi khi chúng ta có thể thấy các quá trình ở trạng thái "bận rộn" khi chạy lệnh ```php start.php status```, cho thấy rằng quá trình tương ứng đang xử lý kinh doanh. Theo trạng thái bình thường, sau khi hoàn tất xử lý kinh doanh, quá trình tương ứng sẽ được phục hồi về trạng thái "trống rỗng". Thông thường điều này không gây ra vấn đề gì. Tuy nhiên, nếu liên tục ở trạng thái bận rộn mà không bao giờ quay trở lại trạng thái "trống rỗng", thì điều này cho thấy rằng kinh doanh trong quá trình đó có thể bị chặn hoặc vòng lặp vô hạn, có thể được định vị bằng cách sử dụng các phương pháp sau.

## Sử dụng lệnh strace+lsof để xác định vị trí

**1. Tìm kiếm pid của quá trình bận rộn trong trạng thái**
Chạy lệnh ```php start.php status``` sẽ hiển thị như sau:
![](../images/d1903ed65ef2f3b0850e84ccbedc52aa.png)
Trong hình ảnh, pid của quá trình "bận rộn" là "11725" và "11748".

**2. Theo dõi quá trình bằng strace**
Chọn một pid của quá trình (ở đây chọn "11725"), chạy lệnh ```strace -ttp 11725``` hiển thị như sau:
![](../images/7ce9f36da926f670949609dcdc593ab4.png)
Có thể thấy quá trình đang liên tục lặp lại cuộc gọi hệ thống ```poll([{fd=16, events=....``` này để chờ sự kiện có thể đọc từ mô tả vị trí fd=16, tức là quá trình đang chờ đợi dữ liệu từ mô tả vị trí này.

Nếu không có bất kỳ cuộc gọi hệ thống nào được hiển thị, giữ nguyên terminal hiện tại, mở terminal khác và chạy lệnh ```kill -SIGALRM 11725``` (gửi tín hiệu báo thức đến quá trình), sau đó xem terminal của strace có hồi đáp, liệu có bị chặn tại một cuộc gọi hệ thống nào hay không. Nếu vẫn không có bất kỳ cuộc gọi hệ thống nào, có thể rằng chương trình đang bị mắc kẹt trong vòng lặp vô hạn, tham khảo những nguyên nhân khác gây ra tình trạng bận rộn kéo dài ở phần cuối trang này.

Nếu hệ thống bị kẹt tại cuộc gọi hệ thống epoll_wait hoặc select thì đây là tình trạng bình thường, cho thấy quá trình đã ở trong trạng thái "trống rỗng".

**3. Sử dụng lệnh lsof để xem mô tả vị trí của quá trình**
Chạy lệnh ```lsof -nPp 11725``` hiển thị như sau:
![](../images/27bd629c3a1ac93f9f4b535d01df2ac1.png)
Mô tả vị trí 16 tương ứng với bản ghi 16u (dòng cuối cùng), có thể thấy mô tả vị trí fd=16 là một kết nối tcp, địa chỉ từ xa là "101.37.136.135:80", cho thấy quá trình có thể đang truy cập vào một nguồn tài nguyên http, vòng lặp ```poll([{fd=16, events=....``` đang chờ đợi dữ liệu được trả về từ máy chủ http, đây là lí do tại sao quá trình ở trạng thái "bận rộn".

**Giải quyết:**
Khi biết được nơi mà quá trình bị chặn, tiếp theo sẽ dễ dàng giải quyết vấn đề, ví dụ: sau khi xác định vị trí, có thể xác định được rằng kinh doanh đang gọi curl, và URL tương ứng lâu không trả về dữ liệu, dẫn đến việc quá trình chờ đợi. Lúc này có thể liên hệ với người cung cấp URL để xác định nguyên nhân trễ trả về của URL và đồng thời nên thêm thông số Timeout khi gọi curl, ví dụ: 2 giây không trả về thì Timeout, tránh trạng thái bận rộn kéo dài (lúc này quá trình có thể bị bận rộn khoảng 2 giây).

## Những nguyên nhân khác gây ra trạng thái bận rộn kéo dài cho quá trình
Ngoài việc quá trình bị chặn hoặc dẫn đến trạng thái "bận rộn", còn có các nguyên nhân sau có thể gây ra trạng thái "bận rộn" cho quá trình.

**1. Kinh doanh gặp lỗi nghiêm trọng dẫn đến việc quá trình liên tục thoát**
**Hiện tượng:** Trong trường hợp này, tải trọng hệ thống khá cao, "load average" trong "status" là 1 hoặc cao hơn. Có thể thấy số lần "exit_count" của quá trình tăng cao và liên tục tăng.
**Giải quyết:** Chạy cách dò lỗi (chạy ```php start.php start``` mà không có ```-d```) của Workerman để xem lỗi kinh doanh và sửa lỗi đó.

**2. Vòng lặp vô hạn trong mã lệnh**
**Hiện tượng:** Trong top có thể thấy quá trình bận chiếm nhiều CPU, lệnh ```strace -ttp pid``` không hiển thị bất kỳ thông tin cuộc gọi hệ thống nào.
**Giải quyết:** Tham khảo bài viết của Bird Brother để tìm vị trí sử dụng gdb và mã nguồn PHP (https://www.laruence.com/2011/12/06/2381.html), tóm lược các bước như sau:
1. Chạy ```php -v``` để xem phiên bản
2. [Tải xuống mã nguồn PHP tương ứng](https://www.php.net/releases/)
3. Chạy ```gdb --pid=pid của quá trình bận rộn```
4. ```source đường dẫn mã nguồn PHP/.gdbinit```
5. ```zbacktrace``` để in ra stack gọi
Các bước cuối cùng giúp xem xét vị trí hiện tại của mã PHP đang thực thi, tức là nơi xuất hiện vòng lặp vô hạn trong mã PHP.
Lưu ý: Nếu ```zbacktrace``` không in ra stack gọi, có thể là trong quá trình biên dịch PHP không có thêm tham số ```-g```, cần phải biên dịch lại PHP, sau đó khởi động lại Workerman để xác định vị trí.

**3. Thêm vô hạn bộ định thời**
Mã kinh doanh không ngừng thêm bộ định thời mà không xóa, dẫn đến số lượng bộ định thời trong quá trình ngày càng tăng, cuối cùng dẫn đến việc quá trình chạy bộ định thời vô hạn. Ví dụ mã như sau:
```php
$worker = new Worker;
$worker->onConnect = function($con){
    Timer::add(10, function(){});
};
Worker::runAll();
````
Mã trên sẽ thêm một bộ định thời sau khi có kết nối từ khách hàng, nhưng mã kinh doanh không có logic xóa bộ định thời, điều này khi thời gian trôi qua sẽ dẫn đến việc quá trình không ngừng thêm bộ định thời, cuối cùng dẫn đến việc chạy bộ định thời vô hạn. Mã đúng:
```php
$worker = new Worker;
$worker->onConnect = function($con){
    $con->timer_id = Timer::add(10, function(){});
};
$worker->onClose = function($con){
    Timer::del($con->timer_id);
};
Worker::runAll();
````

