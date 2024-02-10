# Giải thích

Từ phiên bản 4.x trở đi, workerman đã tăng cường hỗ trợ dịch vụ HTTP. Đã giới thiệu lớp yêu cầu, lớp phản hồi, lớp phiên và [SSE](SSE.md). Nếu bạn muốn sử dụng dịch vụ HTTP của workerman, nên sử dụng workerman phiên bản 4.x trở lên.

**Chú ý: Tất cả các ví dụ dưới đây đều sử dụng phiên bản 4.x của workerman, không tương thích với workerman 3.x.**


# Lấy đối tượng phiên
```php
$session = $request->session();
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
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Chạy worker
Worker::runAll();
```
**Lưu ý**
- Phiên phải được thao tác trước khi gọi `$connection->send()`.
- Phiên sẽ tự động lưu thay đổi khi đối tượng bị hủy, vì vậy không nên lưu đối tượng được trả về từ `$request->session()` vào mảng toàn cục hoặc thành viên của lớp, có thể dẫn đến việc phiên không thể được lưu.
- Mặc định, phiên được lưu trữ trong tệp trên đĩa, nếu bạn muốn hiệu suất tốt hơn, nên sử dụng redis.


## Lấy tất cả dữ liệu phiên
```php
$session = $request->session();
$all = $session->all();
```
Trả về một mảng. Nếu không có dữ liệu phiên nào, thì sẽ trả về một mảng rỗng.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send(var_export($session->all(), true));
};

// Chạy worker
Worker::runAll();
```


## Lấy giá trị của phiên
```php
$session = $request->session();
$name = $session->get('name');
```
Nếu dữ liệu không tồn tại, thì sẽ trả về null.

Bạn cũng có thể truyền vào một giá trị mặc định cho phương thức get, nếu không tìm thấy giá trị tương ứng trong mảng phiên thì sẽ trả về giá trị mặc định. Ví dụ: 
```php
$session = $request->session();
$name = $session->get('name', 'tom');
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
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// Chạy worker
Worker::runAll();
```


## Lưu trữ phiên
Khi lưu trữ một dữ liệu, sử dụng phương thức set.
```php
$session = $request->session();
$session->set('name', 'tom');
```
Phương thức set không trả về giá trị, phiên sẽ tự động lưu khi đối tượng bị hủy.

Khi lưu trữ nhiều giá trị, sử dụng phương thức put.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
Tương tự, phương thức put cũng không trả về giá trị.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send($session->get('name'));
};

// Chạy worker
Worker::runAll();
```


## Xóa dữ liệu phiên
Khi xóa một hoặc nhiều dữ liệu phiên, sử dụng phương thức `forget`.
```php
$session = $request->session();
// Xóa một mục
$session->forget('name');
// Xóa nhiều mục
$session->forget(['name', 'age']);
```

Ngoài ra, hệ thống cũng cung cấp phương thức delete, khác biệt so với phương thức forget là delete chỉ có thể xóa một mục.
```php
$session = $request->session();
// Tương đương với $session->forget('name');
$session->delete('name');
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
    $request->session()->forget('name');
    $connection->send('ok');
};

// Chạy worker
Worker::runAll();
```


## Lấy và xóa một giá trị phiên
```php
$session = $request->session();
$name = $session->pull('name');
```
Hiệu quả tương đương với đoạn mã sau
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
Nếu phiên tương ứng không tồn tại, thì sẽ trả về null.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->session()->pull('name'));
};

// Chạy worker
Worker::runAll();
```


## Xóa tất cả dữ liệu phiên
```php
$request->session()->flush();
```
Phương thức không trả về giá trị, phiên sẽ tự động xóa khỏi bộ nhớ khi đối tượng bị hủy.

**Ví dụ**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $request->session()->flush();
    $connection->send('ok');
};

// Chạy worker
Worker::runAll();
```


## Kiểm tra xem dữ liệu phiên tương ứng có tồn tại không
```php
$session = $request->session();
$has = $session->has('name');
```
Nếu phiên tương ứng không tồn tại hoặc giá trị phiên tương ứng là null, thì sẽ trả về false, ngược lại trả về true.

```
$session = $request->session();
$has = $session->exists('name');
```
Đoạn mã trên cũng được sử dụng để kiểm tra xem dữ liệu phiên có tồn tại không, khác biệt là khi giá trị mục phiên tương ứng là null thì cũng trả về true.
