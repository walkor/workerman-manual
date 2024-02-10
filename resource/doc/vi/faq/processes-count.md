# Cần mở bao nhiêu tiến trình

## Làm cách nào để thiết lập số tiến trình
Số tiến trình được quyết định bởi thuộc tính `count` (hệ thống windows không hỗ trợ thiết lập số tiến trình), ví dụ như đoạn mã dưới đây:

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// ## Khởi động 4 tiến trình để cung cấp dịch vụ ngoại vi ##
$http_worker->count = 4;

...
```

## Các điều kiện cần xem xét khi thiết lập số tiến trình
1. Số lõi CPU
2. Dung lượng bộ nhớ
3. Hướng đi kinh doanh theo hướng tập trung vào IO hay CPU

## Nguyên tắc thiết lập số tiến trình
1. Tổng dung lượng bộ nhớ của mỗi tiến trình cần nhỏ hơn tổng dung lượng bộ nhớ (thông thường mỗi tiến trình kinh doanh chiếm khoảng 40M)
2. Nếu theo hướng tập trung vào IO, nghĩa là trong kinh doanh có những IO **đợi chặn** như truy cập thông thường đến lưu trữ như Mysql, Redis, thì có thể mở nhiều tiến trình hơn, ví dụ như thiết lập thành ba lần số lõi CPU. Nếu có rất nhiều IO đợi chặn trong kinh doanh, có thể tăng số tiến trình thêm một chút, ví dụ như tám lần số lõi CPU hoặc cao hơn. Lưu ý rằng IO không đợi chặn thuộc dạng tập trung vào CPU, không phải IO.
3. Nếu theo hướng tập trung vào CPU, nghĩa là trong kinh doanh không có chi phí **đợi chặn** như việc sử dụng đọc bất đồng bộ để đọc tài nguyên mạng, tiến trình sẽ không bị chặn bởi mã kinh doanh, có thể thiết lập số tiến trình giống như số lõi CPU.

## Giá trị tham khảo khi thiết lập số tiến trình
Nếu mã kinh doanh tập trung vào IO, sau đó thiết lập số tiến trình tùy thuộc vào mức độ tập trung vào IO, ví dụ như ba đến tám lần số lõi CPU.

Nếu mã kinh doanh tập trung vào CPU, sau đó có thể thiết lập số tiến trình là số lõi CPU.

## Chú ý
Workerman tự nó là IO không đợi chặn, ví dụ như ```Connection->send```, đều là thao tác không đợi chặn, thuộc dạng tập trung vào CPU. Nếu không chắc chắn hướng tập trung vào dạng nào, có thể thiết lập số tiến trình là khoảng ba lần số lõi CPU. Ngoài ra, số tiến trình không phải càng nhiều càng tốt, nếu mở quá nhiều tiến trình, chi phí chuyển đổi tiến trình sẽ tăng, ảnh hưởng đến hiệu suất một chút.
