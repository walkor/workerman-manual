# Giới thiệu
Từ phiên bản 4.x trở đi, workerman đã tăng cường hỗ trợ dịch vụ HTTP. Nó đã giới thiệu lớp yêu cầu, lớp phản hồi, lớp session và [SSE](SSE.md). Nếu bạn muốn sử dụng dịch vụ HTTP của workerman, hãy nên sử dụng phiên bản 4.x trở lên.

**Lưu ý rằng tất cả các ví dụ dưới đây đều sử dụng phiên bản 4.x của workerman và không tương thích với workerman 3.x.**

## Thay đổi engine lưu trữ session
Workerman cung cấp hai engine lưu trữ session: engine lưu trữ tệp và engine lưu trữ redis. Mặc định, nó sử dụng engine lưu trữ tệp. Nếu bạn muốn chuyển đổi sang engine lưu trữ redis, vui lòng tham khảo mã sau đây.
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// Cấu hình redis
$config = [
    'host'     => '127.0.0.1', // Tham số bắt buộc
    'port'     => 6379,        // Tham số bắt buộc
    'timeout'  => 2,           // Tham số tùy chọn
    'auth'     => '******',    // Tham số tùy chọn
    'database' => 1,           // Tham số tùy chọn
    'prefix'   => 'session_'   // Tham số tùy chọn
];
// Sử dụng phương thức Workerman\Protocols\Http\Session::handlerClass để thay đổi lớp engine dưới cùng của session
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```

## Thiết lập vị trí lưu trữ session
Khi sử dụng engine lưu trữ mặc định, dữ liệu session được lưu trữ mặc định trên đĩa, vị trí mặc định là vị trí được trả về bởi `session_save_path()`. Bạn có thể sử dụng phương thức sau để thay đổi vị trí lưu trữ.
```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Thiết lập vị trí lưu trữ tệp session
FileSessionHandler::sessionSavePath('/tmp/session');

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

## Xóa tập tin session
Khi sử dụng engine lưu trữ mặc định, có nhiều tập tin session trên đĩa và workerman sẽ xóa các tập tin session hết hạn dựa trên các tùy chọn `session.gc_probability`, `session.gc_divisor`, `session.gc_maxlifetime` được cấu hình trong php.ini. Chi tiết về ba tùy chọn này có thể tham khảo trang [tài liệu PHP](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability).

## Thay đổi driver lưu trữ
Ngoài engine lưu trữ tệp và engine lưu trữ redis, workerman cho phép bạn thêm engine lưu trữ session mới thông qua giao diện chuẩn [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php), ví dụ như engine lưu trữ session mangoDb, engine lưu trữ session MySQL, v.v.

**Quy trình thêm engine lưu trữ session mới**
  1.  Thực hiện giao diện [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php)
  2. Sử dụng phương thức `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)` để thay thế giao diện gốc của SessionHandler
  
**Thực hiện giao diện SessionHandlerInterface**

Trình điều khiển lưu trữ session tùy chỉnh cần thực hiện giao diện SessionHandlerInterface. Giao diện này bao gồm các phương thức sau:
```php
SessionHandlerInterface {
    /* Phương thức */
    abstract public read ( string $session_id ) : string
    abstract public write ( string $session_id , string $session_data ) : bool
    abstract public destroy ( string $session_id ) : bool
    abstract public gc ( int $maxlifetime ) : int
    abstract public close ( void ) : bool
    abstract public open ( string $save_path , string $session_name ) : bool
}
```
**Giải thích về SessionHandlerInterface**
 - Phương thức read được sử dụng để đọc tất cả dữ liệu session tương ứng với session_id từ lưu trữ. Vui lòng không thực hiện thao tác deserialize dữ liệu, framework sẽ tự động thực hiện.
 - Phương thức write được sử dụng để ghi dữ liệu session tương ứng với session_id vào lưu trữ. Vui lòng không thực hiện thao tác serialize dữ liệu, framework đã tự động thực hiện.
 - Phương thức destroy được sử dụng để hủy dữ liệu session tương ứng với session_id.
 - Phương thức gc được sử dụng để xóa dữ liệu session hết hạn, lưu trữ phải xóa tất cả các session có thời gian sửa đổi cuối cùng lớn hơn maxlifetime.
 - Phương thức close không cần thực hiện bất kỳ thao tác nào, chỉ cần trả về true.
 - Phương thức open không cần thực hiện bất kỳ thao tác nào, chỉ cần trả về true.

**Thay thế driver lưu trữ gốc**

Sau khi triển khai giao diện SessionHandlerInterface, sử dụng phương thức sau để thay đổi driver lưu trữ session.
```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
```
 - $class_name là tên của lớp SessionHandler thực hiện giao diện SessionHandlerInterface. Nếu có namespace, bạn cần cung cấp đầy đủ tên miền namespace
 - $config là các tham số của hàm tạo của lớp SessionHandler

**Triển khai cụ thể**

*Chú ý, lớp MySessionHandler chỉ được sử dụng để minh họa quy trình thay đổi driver lưu trữ session và không thể được sử dụng trong môi trường sản xuất.*
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

class MySessionHandler implements SessionHandlerInterface
{
    protected static $store = [];
    
    public function __construct($config) {
        // ['host' => 'localhost']
        var_dump($config);
    }
   
    public function open($save_path, $name)
    {
        return true;
    }

    public function read($session_id)
    {
        return isset(static::$store[$session_id]) ? static::$store[$session_id]['content'] : '';
    }

    public function write($session_id, $session_data)
    {
        static::$store[$session_id] = ['content' => $session_data, 'timestamp' => time()];
    }

    public function close()
    {
        return true;
    }

    public function destroy($session_id)
    {
        unset(static::$store[$session_id]);
        return true;
    }

    public function gc($maxlifetime) {
        $time_now = time();
        foreach (static::$store as $session_id => $info) {
            if ($time_now - $info['timestamp'] > $maxlifetime) {
                unset(static::$store[$session_id]);
            }
        }
    }
}

// Giả sử lớp SessionHandler mới cần một số cấu hình được truyền vào
$config = ['host' => 'localhost'];
// Sử dụng phương thức Workerman\Protocols\Http\Session::handlerClass($class_name, $config) để thay đổi lớp driver lưu trữ session
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
