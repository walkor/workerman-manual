# Hướng dẫn
Từ phiên bản 4.x trở đi, workerman đã tăng cường hỗ trợ dịch vụ HTTP. Đưa vào các lớp yêu cầu, phản hồi, phiên và SSE (https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events). Nếu bạn muốn sử dụng dịch vụ HTTP của workerman, rất khuyến khích sử dụng phiên bản 4.x của workerman hoặc các phiên bản cao hơn.

**Lưu ý rằng tất cả đều là cách sử dụng của workerman phiên bản 4.x, không tương thích với workerman phiên bản 3.x.**

## Lấy đối tượng yêu cầu
Đối tượng yêu cầu luôn được lấy trong hàm gọi lại onMessage, framework sẽ tự động truyền đối tượng yêu cầu thông qua tham số thứ hai của hàm gọi lại.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request là đối tượng yêu cầu, ở đây không có bất kỳ hành động nào được thực hiện trên đối tượng yêu cầu, chỉ cần trả về "hello" cho trình duyệt
    $connection->send("hello");
};

// Chạy worker
Worker::runAll();
```

Khi trình duyệt truy cập `http://127.0.0.1:8080` sẽ trả về `hello`.

## Lấy tham số get của yêu cầu
**Lấy toàn bộ mảng tham số get**
```php
$get = $request->get();
```
Nếu không có tham số get trong yêu cầu thì sẽ trả về một mảng trống.

**Lấy giá trị của một phần tử trong mảng get**
```php
$name = $request->get('name');
```
Nếu mảng get không chứa giá trị này thì sẽ trả về null.

Bạn cũng có thể truyền vào tham số thứ hai cho phương thức get với giá trị mặc định, nếu mảng get không tìm thấy giá trị tương ứng sẽ trả về giá trị mặc định. Ví dụ:
```php
$name = $request->get('name', 'tom');
```

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->get('name'));
};

// Chạy worker
Worker::runAll();
```

Khi trình duyệt truy cập `http://127.0.0.1:8080?name=jerry&age=12` sẽ trả về `jerry`.

## Lấy tham số post của yêu cầu
**Lấy toàn bộ mảng tham số post**
```php
$post = $request->post();
```
Nếu không có tham số post trong yêu cầu thì sẽ trả về một mảng trống.

**Lấy giá trị của một phần tử trong mảng post**
```php
$name = $request->post('name');
```
Nếu mảng post không chứa giá trị này thì sẽ trả về null.

Tương tự như phương thức get, bạn cũng có thể truyền vào tham số thứ hai cho phương thức post với giá trị mặc định, nếu mảng post không tìm thấy giá trị tương ứng sẽ trả về giá trị mặc định. Ví dụ:
```php
$name = $request->post('name', 'tom');
```

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = $request->post();
    $connection->send(var_export($post, true));
};

// Chạy worker
Worker::runAll();
```


## Lấy body gốc của yêu cầu post
```php
$post = $request->rawBody();
```
Chức năng này tương tự như hoạt động `file_get_contents("php://input")` trong `php-fpm`. Dùng để lấy nội dung gói yêu cầu gốc HTTP. Rất hữu ích khi lấy dữ liệu yêu cầu post không phải là định dạng `application/x-www-form-urlencoded`. 

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = json_decode($request->rawBody());
    $connection->send('hello');
};

// Chạy worker
Worker::runAll();
```

## Lấy header
**Lấy toàn bộ mảng header**
```php
$headers = $request->header();
```
Nếu không có tham số header trong yêu cầu thì sẽ trả về một mảng trống. Lưu ý rằng tất cả các key đều là chữ thường.

**Lấy giá trị của một phần tử trong mảng header**
```php
$host = $request->header('host');
```
Nếu mảng header không chứa giá trị này thì sẽ trả về null. Lưu ý rằng tất cả các key đều là chữ thường.

Tương tự như phương thức get, bạn cũng có thể truyền vào tham số thứ hai cho phương thức header với giá trị mặc định, nếu mảng header không tìm thấy giá trị tương ứng sẽ trả về giá trị mặc định. Ví dụ:
```php
$host = $request->header('host', 'localhost');
```

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->header('connection') === 'keep-alive') {
        $connection->send('hello');
    } else {
        $connection->close('hello');
    }    
};

// Chạy worker
Worker::runAll();
```

## Lấy cookie
**Lấy toàn bộ mảng cookie**
```php
$cookies = $request->cookie();
```
Nếu không có tham số cookie trong yêu cầu thì sẽ trả về một mảng trống.

**Lấy giá trị của một phần tử trong mảng cookie**
```php
$name = $request->cookie('name');
```
Nếu mảng cookie không chứa giá trị này thì sẽ trả về null.

Tương tự như phương thức get, bạn cũng có thể truyền vào tham số thứ hai cho phương thức cookie với giá trị mặc định, nếu mảng cookie không tìm thấy giá trị tương ứng sẽ trả về giá trị mặc định. Ví dụ:
```php
$name = $request->cookie('name', 'tom');
```

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $cookie = $request->cookie();
    $connection->send(var_export($cookie, true));
};

// Chạy worker
Worker::runAll();
```

## Lấy file được tải lên
**Lấy toàn bộ mảng file được tải lên**
```php
$files = $request->file();
```
Định dạng file trả về như sau:
```php
array (
    'avatar' => array (
            'name' => '123.jpg',
            'tmp_name' => '/tmp/workerman.upload.9hjR4w',
            'size' => 1196127,
            'error' => 0,
            'type' => 'application/octet-stream',
      ),
     'anotherfile' =>  array (
            'name' => '456.txt',
            'tmp_name' => '/tmp/workerman.upload.9sirSws',
            'size' => 490,
            'error' => 0,
            'type' => 'text/plain',
      )
)
```
Trong đó:

 - name là tên file
 - tmp_name là vị trí file tạm trên ổ đĩa
 - size là kích thước file
 - error là [mã lỗi](https://www.php.net/manual/zh/features.file-upload.errors.php)
 - type là loại mime của file.

**Lưu ý:**

 - Kích thước file tải lên bị giới hạn bởi [defaultMaxPackageSize](../tcp-connection/default-max-package-size.md), mặc định là 10M, có thể thay đổi.

 - Sau khi yêu cầu kết thúc, file sẽ được tự động xóa.

 - Nếu yêu cầu không có file tải lên thì sẽ trả về một mảng trống.

### Lấy một file tải lên cụ thể
```php
$avatar_file = $request->file('avatar');
```
Trả về dạng tương tự
```php
array (
        'name' => '123.jpg',
        'tmp_name' => '/tmp/workerman.upload.9hjR4w',
        'size' => 1196127,
        'error' => 0,
        'type' => 'application/octet-stream',
  )
```
Nếu file tải lên không tồn tại sẽ trả về null.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = $request->file('avatar');
    if ($file && $file['error'] === UPLOAD_ERR_OK) {
        rename($file['tmp_name'], '/home/www/web/public/123.jpg');
        $connection->send('ok');
        return;
    }
    $connection->send('upload fail');
};

// Chạy worker
Worker::runAll();
```
## Lấy thông tin host
Lấy thông tin host của yêu cầu.
```php
$host = $request->host();
```
Nếu địa chỉ yêu cầu không phải là cổng chuẩn 80 hoặc 443, thông tin host có thể chứa cả cổng, ví dụ `example.com:8080`. Nếu không cần cổng, có thể truyền vào tham số thứ nhất là `true`.

```php
$host = $request->host(true);
```
**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->host());
};

// Chạy worker
Worker::runAll();
```
Khi trình duyệt truy cập `http://127.0.0.1:8080?name=tom` sẽ trả về `127.0.0.1:8080`.
## Lấy phương thức yêu cầu
```php
$method = $request->method();
```
Giá trị trả về có thể là `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `HEAD`.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->method());
};

// Chạy worker
Worker::runAll();
```
## Lấy uri yêu cầu
```php
$uri = $request->uri();
```
Trả về uri của yêu cầu, bao gồm path và queryString.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->uri());
};

// Chạy worker
Worker::runAll();
```
Khi trình duyệt truy cập `http://127.0.0.1:8080/user/get.php?uid=10&type=2` sẽ trả về `/user/get.php?uid=10&type=2`.

## Lấy đường dẫn yêu cầu

```php
$path = $request->path();
```
Trả về phần path của yêu cầu.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->path());
};

// Chạy worker
Worker::runAll();
```
Khi trình duyệt truy cập `http://127.0.0.1:8080/user/get.php?uid=10&type=2` sẽ trả về `/user/get.php`.

## Lấy chuỗi truy vấn yêu cầu

```php
$query_string = $request->queryString();
```
Trả về phần queryString của yêu cầu.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->queryString());
};

// Chạy worker
Worker::runAll();
```
Khi trình duyệt truy cập `http://127.0.0.1:8080/user/get.php?uid=10&type=2` sẽ trả về `uid=10&type=2`.

## Lấy phiên bản HTTP của yêu cầu

```php
$version = $request->protocolVersion();
```
Trả về chuỗi `1.1` hoặc `1.0`.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->protocolVersion());
};

// Chạy worker
Worker::runAll();
```

## Lấy session ID của yêu cầu

```php
$sid = $request->sessionId();
```
Trả về chuỗi gồm chữ cái và số.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->sessionId());
};

// Chạy worker
Worker::runAll();
```
