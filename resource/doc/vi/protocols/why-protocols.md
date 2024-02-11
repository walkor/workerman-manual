# Vai trò của giao thức truyền thông
Do TCP được xây dựng trên cơ sở dòng dữ liệu, dữ liệu yêu cầu được gửi từ client sẽ tràn vào máy chủ như dòng nước, máy chủ phải kiểm tra xem dữ liệu có hoàn chỉnh không, bởi vì có thể chỉ là một phần dữ liệu yêu cầu đến máy chủ, thậm chí có thể là nhiều yêu cầu kết hợp đến máy chủ. Làm cách nào để xác định xem yêu cầu đã đến hoàn toàn chưa hoặc tách yêu cầu từ nhiều yêu cầu kết hợp đến máy chủ, điều đó cần có một bộ giao thức truyền thông.

## Tại sao cần xác định giao thức trong Workerman?

Việc phát triển PHP truyền thống dựa trên Web, về cơ bản đều là giao thức HTTP, việc phân tích và xử lý giao thức HTTP đều do WebServer tự chịu trách nhiệm, do đó người phát triển không cần quan tâm đến vấn đề giao thức. Tuy nhiên, khi chúng ta cần phát triển dựa trên giao thức không phải HTTP, người phát triển sẽ phải quan tâm đến vấn đề giao thức.

## Các giao thức mà Workerman đã hỗ trợ
Hiện tại, Workerman đã hỗ trợ giao thức HTTP, websocket, text (xem phần phụ lục), frame (xem phần phụ lục), ws (xem phần phụ lục). Khi cần sử dụng các giao thức này, có thể sử dụng trực tiếp bằng cách: xác định giao thức khi khởi tạo Worker, ví dụ như sau:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket://0.0.0.0:2345 xác định sử dụng giao thức websocket trên cổng 2345
$websocket_worker = new Worker('websocket://0.0.0.0:2345');

// giao thức text
$text_worker = new Worker('text://0.0.0.0:2346');

// giao thức frame
$frame_worker = new Worker('frame://0.0.0.0:2347');

// Worker tcp, truyền trực tiếp trên socket, không sử dụng bất kỳ giao thức ứng dụng nào
$tcp_worker = new Worker('tcp://0.0.0.0:2348');

// Worker udp, không sử dụng bất kỳ giao thức ứng dụng nào
$udp_worker = new Worker('udp://0.0.0.0:2349');

// Worker thư mục unix, không sử dụng bất kỳ giao thức ứng dụng nào
$unix_worker = new Worker('unix:///tmp/wm.sock');

```

## Sử dụng giao thức tuỳ chỉnh
Khi giao thức truyền thông mặc định của Workerman không đáp ứng nhu cầu phát triển, người phát triển có thể tùy chỉnh giao thức truyền thông của mình, cách thức tùy chỉnh được trình bày trong phần tiếp theo.

**Lưu ý:**

Workerman tích hợp một giao thức text, định dạng giao thức là văn bản + kí tự xuống dòng. Giao thức text dễ dàng và thuận tiện cho việc phát triển và gỡ lỗi, có thể sử dụng cho hầu hết các tình huống giao thức tùy chỉnh, và hỗ trợ gỡ lỗi bằng telnet. Nếu người phát triển muốn phát triển giao thức ứng dụng của riêng mình, có thể sử dụng giao thức text ngay, không cần phải phát triển riêng lẻ.

Xem thêm thông tin về giao thức text trong phần phụ lục.
