# Các hàm không được hỗ trợ

Hàm / Lệnh không được hỗ trợ | Thay thế | Giải thích
----|------|----
pcntl_fork | Thiết lập số tiến trình trước | 
php://input | [`$request->rawBody()`](http/request.md) | Dùng để lấy dữ liệu gốc POST trong ứng dụng dưới giao thức HTTP
exit | return | Sử dụng exit sẽ dẫn đến quá trình kết thúc, nếu muốn trả về thì hãy sử dụng lệnh return trực tiếp.
die | return | Sử dụng die sẽ dẫn đến quá trình kết thúc, nếu muốn trả về thì hãy sử dụng lệnh return trực tiếp.
header cookie session và các hàm liên quan | Xem thông tin chi tiết tại lớp [`$request`](http/request.md) và [`$response`](http/response.md) | 
set_time_limit| Không có | Chỉ có thể thiết lập là 0, nếu không sẽ dẫn đến quá trình workerman kết thúc sau một thời gian nhất định
