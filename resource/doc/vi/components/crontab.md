# workerman/crontab

# Hướng dẫn
`workerman/crontab` là một chương trình công việc định kỳ dựa trên workerman, tương tự như crontab trên linux. `workerman/crontab` hỗ trợ định kỳ tính theo giây.

> Để sử dụng `workerman/crontab`, bạn cần thiết lập múi giờ của PHP trước, nếu không có thể dẫn đến kết quả chạy không như mong đợi.

## Giải thích về thời gian
```plaintext
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ ngày trong tuần (0 - 6) (Chủ nhật = 0)
|   |   |   |   +------ tháng (1 - 12)
|   |   |   +-------- ngày trong tháng (1 - 31)
|   |   +---------- giờ (0 - 23)
|   +------------ phút (0 - 59)
+-------------- giây (0-59)[có thể bỏ qua, nếu không có bit 0, thì đơn vị thời gian nhỏ nhất là phút]
```

# Cài đặt
```plaintext
composer require workerman/crontab
```

# Ví dụ
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// Thiết lập múi giờ, tránh kết quả chạy không như mong đợi
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // Thực hiện vào phút thứ 1 của mỗi phút.
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // Thực hiện vào lúc 7 giờ 50 phút hàng ngày, lưu ý đây là bỏ qua bit giây.
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> Lưu ý: Công việc định kỳ không thực thi ngay lập tức, tất cả công việc định kỳ bắt đầu tính thời gian tính từ phút tiếp theo.

# Giao diện
**Crontab::destroy()**

Hủy bỏ bộ định thời
```plaintext
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```
