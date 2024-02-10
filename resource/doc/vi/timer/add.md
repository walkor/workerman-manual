# Thêm
```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
Thực hiện hàm hoặc phương thức của lớp một cách định kỳ.

Lưu ý: Bộ hẹn giờ chạy trong quy trình hiện tại, workerman không tạo ra quy trình hoặc luồng mới để chạy bộ định thời.

### Tham số
``` time_interval ```

Thời gian giữa các lần chạy, tính bằng giây, hỗ trợ số thập phân, có thể chính xác đến 0,001, tức là chính xác đến nano giây.

``` callback ```

Hàm gọi lại``` Lưu ý: Nếu hàm gọi lại là phương thức của lớp, phương thức phải là thông tin công cộng ```

``` args ```

Tham số của hàm gọi lại, phải là mảng, các phần tử mảng là giá trị tham số

``` persistent ```

Là một hằng số, nếu muốn thực hiện theo định kỳ chỉ một lần, chỉ cần truyền false (công việc chỉ thực hiện một lần sẽ tự động bị hủy sau khi hoàn thành, không cần phải gọi ```Timer::del()```). Mặc định là true, tức là thực hiện định kỳ liên tục.

### Giá trị trả về
Trả về một số nguyên, đại diện cho timerid của bộ hẹn giờ, có thể phá hủy bộ hẹn giờ này bằng cách gọi ```Timer::del($timerid)```.

### Ví dụ

#### 1. Hàm định kỳ là hàm vô danh (đóng)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Mở một số quy trình chạy công việc hẹn giờ, hãy chú ý xem xét xem công việc có vấn đề song song không
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Thực hiện mỗi 2.5 giây
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "thực hiện công việc\n";
    });
};

// Chạy worker
Worker::runAll();
```

#### 2. Chỉ đặt bộ hẹn giờ trong quá trình được chỉ định

Một ví dụ được đặt trong một ví dụ, có 4 quá trình,  chỉ đặt bộ hẹn giờ trong quá trình có số định danh là 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->count = 4;
$worker->onWorkerStart = function(Worker $worker)
{
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 quá trình worker, chỉ đặt bộ hẹn giờ ở quá trình số 0\n";
        });
    }
};
// Chạy worker
Worker::runAll();
```

#### 3. Hàm định kỳ là hàm vô danh, sử dụng fluống để truyền tham số
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
// Khi kết nối được thiết lập, đặt bộ hẹn giờ cho kết nối tương ứng
$ws_worker->onConnect = function(TcpConnection $connection)
{
    // Thực hiện mỗi 10 giây
    $time_interval = 10;
    $connect_time = time();
    $connection->timer_id = Timer::add($time_interval, function()use($connection, $connect_time)
    {
         $connection->send($connect_time);
    });
};
// Khi kết nối đóng, xóa bộ hẹn giờ tương ứng
$ws_worker->onClose = function(TcpConnection $connection)
{
    // Xóa bộ hẹn giờ
    Timer::del($connection->timer_id);
};

// Chạy worker
Worker::runAll();
```

#### 4. Hàm định kỳ là hàm vô danh, sử dụng giao diện bộ định thời để truyền tham số
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
// Khi kết nối được thiết lập, đặt bộ hẹn giờ cho kết nối tương ứng
$ws_worker->onConnect = function(TcpConnection $connection)
{
    // thực hiện mỗi 10 giây
    $time_interval = 10;
    $connect_time = time();
    $connection->timer_id = Timer::add($time_interval, function($connection, $connect_time)
    {
         $connection->send($connect_time);
    }, array($connection, $connect_time));
};
// Khi kết nối đóng, xóa bộ hẹn giờ tương ứng
$ws_worker->onClose = function(TcpConnection $connection)
{
    // Xóa bộ hẹn giờ
    Timer::del($connection->timer_id);
};

// Chạy worker
Worker::runAll();
```

#### 5. Hàm định kỳ là hàm thông thường
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

// Hàm thông thường
function send_mail($to, $content)
{
    echo "gửi thư ...\n";
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $to = 'workerman@workerman.net';
    $content = 'xin chào workerman';
    // Thực hiện công việc gửi thư sau 10 giây, tham số cuối cùng truyền false, cho biết chỉ chạy một lần
    Timer::add(10, 'send_mail', array($to, $content), false);
};

// Chạy worker
Worker::runAll();
```

#### 6. Hàm định kỳ là phương thức của lớp
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    //lưu ý, thuộc tính của hàm gọi lại phải là công cộng
    public function send($to, $content)
    {
        echo "gửi thư ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function($task)
{
    $mail = new Mail();
    // Thực hiện gửi thư sau 10 giây một lần
    $to = 'workerman@workerman.net';
    $content = 'xin chào workerman';
    Timer::add(10, array($mail, 'send'), array($to, $content), false);
};

// Chạy worker
Worker::runAll();
```

#### 7. Hàm định kỳ là phương thức của lớp (phương thức bên trong lớp sử dụng hẹn giờ)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // Lưu ý, thuộc tính của hàm gọi lại phải là công cộng
    public function send($to, $content)
    {
        echo "gửi thư ...\n";
    }

    public function sendLater($to, $content)
    {
        // Phương thức gọi lại trong lớp hiện tại, mảng gọi lại được chỉ mục phần tử đầu tiên là $this
        Timer::add(10, array($this, 'send'), array($to, $content), false);
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Gửi một lần thư sau 10 giây
    $mail = new Mail();
    $to = 'workerman@workerman.net';
    $content = 'xin chào workerman';
    $mail->sendLater($to, $content);
};

// Chạy worker
Worker::runAll();
```

#### 8. Hàm định kỳ là phương thức tĩnh của lớp
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // Lưu ý, đây là phương thức tĩnh, thuộc tính hàm gọi lại cũng phải là công cộng
    public static function send($to, $content)
    {
        echo "gửi thư ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Gửi một lần thư sau 10 giây
    $to = 'workerman@workerman.net';
    $content = 'xin chào workerman';
    // Gọi phương thức tĩnh của lớp đến định thời.
    Timer::add(10, array('Mail', 'send'), array($to, $content), false);
};

// Chạy worker
Worker::runAll();
```

#### 9. Hàm định kỳ là phương thức tĩnh của lớp (có không gian tên)
```php
namespace Task;

use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    // Lưu ý, đây là phương thức tĩnh, thuộc tính hàm gọi lại cũng phải là công cộng
    public static function send($to, $content)
    {
        echo "gửi thư ...\n";
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Gửi một lần thư sau 10 giây
    $to = 'workerman@workerman.net';
    $content = 'xin chào workerman';
    // Gọi phương thức tĩnh của lớp có không gian tên đến định thời.
    Timer::add(10, array('\Task\Mail', 'send'), array($to, $content), false);
};

// Chạy worker
Worker::runAll();
```

#### 10. Hủy bộ hẹn giờ trong hàm hẹn giờ (truyền $timer_id bằng phương pháp fluống)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Đếm
    $count = 1;
    // Để $timer_id được truyền đúng vào bên trong hàm gọi lại, phải thêm dấu & phía trước $timer_id
    $timer_id = Timer::add(1, function()use(&$timer_id, &$count)
    {
        echo "Hẹn giờ chạy $count\n";
        // Sau khi chạy 10 lần, hủy bộ hẹn giờ hiện tại
        if($count++ >= 10)
        {
            echo "Timer::del($timer_id)\n";
            Timer::del($timer_id);
        }
    });
};

// Chạy worker
Worker::runAll();
```

#### 11. Hủy bộ hẹn giờ trong hàm hẹn giờ (truyền $timer_id bằng phương pháp tham số)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

class Mail
{
    public function send($to, $content, $timer_id)
    {
        // Thêm một thuộc tính count, ghi lại số lần chạy của bộ hẹn giờ tạm thời cho đối tượng này
        $this->count = empty($this->count) ? 1 : $this->count;
        // Sau khi chạy 10 lần, hủy bộ hẹn giờ hiện tại
        echo "gửi thư {$this->count}...\n";
        if($this->count++ >= 10)
        {
            echo "Timer::del($timer_id)\n";
            Timer::del($timer_id);
        }
    }
}

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    $mail = new Mail();
    // Để $timer_id được truyền đúng vào bên trong hàm gọi lại, phải thêm dấu & phía trước $timer_id
    $timer_id = Timer::add(1, array($mail, 'send'), array('to', 'nội dung', &$timer_id));
};

// Chạy worker
Worker::runAll();
```
