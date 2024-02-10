# Chú thích

Từ phiên bản 4.x trở đi, Workerman đã củng cố hỗ trợ dịch vụ HTTP. Đã đưa vào sử dụng lớp yêu cầu, lớp phản hồi, lớp phiên và [SSE](SSE.md). Nếu bạn muốn sử dụng dịch vụ HTTP của Workerman, đề nghị nghiêm túc sử dụng phiên bản 4.x của Workerman hoặc các phiên bản cao hơn sau này.

**Lưu ý rằng tất cả đều là cách sử dụng phiên bản 4.x của Workerman, không tương thích với phiên bản 3.x.**

# Chú ý

- Trừ khi gửi phản hồi chunk hoặc SSE, nếu không không cho phép gửi nhiều phản hồi trong một yêu cầu, có nghĩa là không cho phép gọi $connection->send() nhiều lần trong một yêu cầu.
- Mỗi yêu cầu cuối cùng đều cần gọi $connection->send() để gửi phản hồi, nếu không client sẽ luôn chờ đợi.

## Phản hồi nhanh chóng
Khi không cần thay đổi mã trạng thái HTTP (mặc định là 200), hoặc tùy chỉnh header, cookie, bạn có thể gửi chuỗi trực tiếp đến client để hoàn tất phản hồi.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Gửi trực tiếp chuỗi "this is body" đến client
    $connection->send("this is body");
};

// Chạy worker
Worker::runAll();
```

## Thay đổi mã trạng thái
Khi cần tùy chỉnh mã trạng thái, header, cookie, bạn cần sử dụng lớp phản hồi `Workerman\Protocols\Http\Response`. Ví dụ dưới đây sẽ trả về mã trạng thái 404 khi truy cập vào đường dẫn `/404`, với nội dung là `<h1>Xin lỗi, tập tin không tồn tại</h1>`.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->path() === '/404') {
        $connection->send(new Response(404, [], '<h1>Xin lỗi, tập tin không tồn tại</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// Chạy worker
Worker::runAll();
```
Khi lớp `Response` đã được khởi tạo, muốn thay đổi mã trạng thái sử dụng phương thức sau.
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## Gửi header
Tương tự, để gửi header bạn cũng cần sử dụng lớp phản hồi `Workerman\Protocols\Http\Response`.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [
        'Content-Type' => 'text/html',
        'X-Header-One' => 'Header Value'
    ], 'this is body');
    $connection->send($response);
};

// Chạy worker
Worker::runAll();
```
Khi lớp `Response` đã được khởi tạo, muốn thêm hoặc thay đổi header sử dụng phương thức sau.
```php
$response = new Response(200);
// Thêm hoặc thay đổi một header
$response->header('Content-Type', 'text/html');
// Thêm hoặc thay đổi nhiều header
$response->withHeaders([
    'Content-Type' => 'application/ json',
    'X-Header-One' => 'Header Value'
]);
$connection->send($response);
```

## Chuyển hướng
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)

$worker = new Worker('http://0.0.0.0:8080');
$worker->onMessage = function($connection, $request)
{
    $location = '/test_location';
    $response = new Response(302, ['Location' => $location]);
    $connection->send($response);
};
Worker::runAll();
```


## Gửi cookie
Tương tự, để gửi cookie bạn cũng cần sử dụng lớp phản hồi `Workerman\Protocols\Http\Response`.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [], 'this is body');
    $response->cookie('name', 'tom');
    $connection->send($response);
};

// Chạy worker
Worker::runAll();
```
## Gửi tệp tin
Tương tự, để gửi tệp tin, bạn cần sử dụng lớp phản hồi `Workerman\Protocols\Http\Response`.

Để gửi tệp tin, sử dụng cú pháp sau:
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
- Workerman hỗ trợ gửi tệp tin cực kỳ lớn
- Đối với tệp tin lớn (lớn hơn 2M), Workerman sẽ không đọc toàn bộ tệp tin vào bộ nhớ một lần, mà sẽ phân đoạn đọc tệp tin và gửi tại thời điểm phù hợp
- Workerman sẽ tối ưu hóa tốc độ đọc và gửi tệp tin dựa trên tốc độ nhận của máy khách, đồng thời đảm bảo gửi tệp tin nhanh nhất có thể và giảm thiểu sử dụng bộ nhớ
- Việc gửi dữ liệu là không chặn, không ảnh hưởng đến việc xử lý yêu cầu khác
- Khi gửi tệp tin, sẽ tự động thêm tiêu đề `Last-Modified` để phục vụ lần yêu cầu sau đó, giúp tiết kiệm truyền tải tệp tin và tăng hiệu suất
- Tệp tin được gửi sẽ tự động sử dụng tiêu đề `Content-Type` phù hợp để gửi đến trình duyệt
- Nếu tệp tin không tồn tại, sẽ tự động chuyển thành phản hồi 404

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = '/đường/dẫn/tới/tệp';
    // Kiểm tra tiêu đề if-modified-since để xác định xem tệp đã được sửa đổi chưa
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        // Nếu tệp không được sửa đổi, trả về phản hồi 304
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // Tệp bị sửa đổi hoặc không có tiêu đề if-modified-since, gửi tệp tin
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// Chạy worker
Worker::runAll();
```

## Gửi dữ liệu http chunk
- Phải gửi một phản hồi Response kèm tiêu đề `Transfer-Encoding: chunked` đến máy khách trước
- Sử dụng lớp `Workerman\Protocols\Http\Chunk` để gửi dữ liệu chunk sau đó
- Cuối cùng, phải gửi một chunk trống để kết thúc phản hồi

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
use Workerman\Protocols\Http\Chunk;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Đầu tiên, gửi một phản hồi Response kèm tiêu đề Transfer-Encoding: chunked
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // Sử dụng lớp Workerman\Protocols\Http\Chunk để gửi dữ liệu chunk sau đó
    $connection->send(new Chunk('Dữ liệu đoạn thứ nhất'));
    $connection->send(new Chunk('Dữ liệu đoạn thứ hai'));
    $connection->send(new Chunk('Dữ liệu đoạn thứ ba'));
   //  Cuối cùng, phải gửi một chunk trống để kết thúc phản hồi
    $connection->send(new Chunk(''));
};

// Chạy worker
Worker::runAll();
```
