## Cách tùy chỉnh giao thức

Thực tế, việc thiết lập giao thức của riêng mình là một điều khá đơn giản. Một giao thức đơn giản thường bao gồm hai phần:
 * Nhận dạng biên giới dữ liệu
 * Định dạng dữ liệu

## Một ví dụ

### Định nghĩa giao thức
Ở đây, giả sử nhận dạng biên giới dữ liệu là ký tự xuống dòng "\n" (lưu ý rằng dữ liệu yêu cầu bên trong không được chứa ký tự xuống dòng), định dạng dữ liệu là Json, ví dụ dưới đây là một gói yêu cầu phù hợp với quy tắc này.

<pre>
{"type":"message","content":"hello"}
</pre>

Lưu ý rằng cuối dữ liệu yêu cầu ở trên có một ký tự xuống dòng (được biểu diễn trong PHP bằng chuỗi **cặp dấu ngoặc kép** "\n"), đại diện cho việc kết thúc một yêu cầu.

### Bước thực hiện
Trong Workerman, nếu muốn thực hiện giao thức như trên, giả sử giao thức có tên là JsonNL, nằm trong dự án là MyApp, bước cần thiết như sau:

1. Đặt tệp giao thức vào thư mục của dự án Protocols, ví dụ: tệp MyApp/Protocols/JsonNL.php

2. Thực hiện lớp JsonNL, với `namespace Protocols;` là không gian tên, phải triển khai ba phương thức tĩnh là input, encode, decode

Lưu ý: Workerman sẽ tự động gọi ba phương thức tĩnh này để thực hiện chia gói, giải gói, đóng gói. Xem quy trình cụ thể trong phần giải thích quá trình thực hiện bên dưới.

### Quá trình tương tác giữa Workerman và lớp giao thức
1. Giả sử client gửi một gói dữ liệu cho server, sau khi nhận được dữ liệu (có thể chỉ là một phần dữ liệu), server sẽ ngay lập tức gọi phương thức `input` của giao thức để kiểm tra độ dài của gói này, phương thức `input` trả về giá trị độ dài `$length` cho framework Workerman.
2. Workerman nhận được giá trị `$length`, sau đó kiểm tra xem liệu trong buffer dữ liệu hiện tại đã nhận đủ dữ liệu có độ dài `$length` chưa, nếu chưa thì tiếp tục chờ đợi dữ liệu cho đến khi chiều dài của buffer dữ liệu không nhỏ hơn `$length`.
3. Khi độ dài buffer đủ, Workerman sẽ cắt ra một đoạn dữ liệu có độ dài `$length` (tức là **chia gói**), sau đó gọi phương thức `decode` của giao thức để **giải gói**, dữ liệu giải được là `$data`.
4. Sau khi giải gói, Workerman sẽ truyền dữ liệu `$data` dưới dạng callback `$connection, $data)` cho logic kinh doanh thông qua hàm gọi lại `onMessage`, tại đây, logic kinh doanh có thể sử dụng biến `$data` để nhận toàn bộ gói dữ liệu đã được giải gói từ phía client.
5. Khi logic kinh doanh trong hàm `onMessage` cần gửi dữ liệu cho client thông qua lệnh `$connection->send($buffer)`, Workerman sẽ tự động sử dụng phương thức `encode` của giao thức để **đóng gói** `$buffer`, sau đó mới gửi cho client.

### Thực hiện cụ thể

**Thực hiện trong tệp MyApp/Protocols/JsonNL.php**

```php
namespace Protocols;
class JsonNL
{
    /**
     * Kiểm tra tính toàn vẹn của gói
     * Nếu có thể lấy được độ dài gói từ $buffer thì trả về độ dài toàn bộ gói, ngược lại trả về 0 để tiếp tục chờ dữ liệu
     * Nếu giao thức gặp vấn đề, có thể trả về false, kết nối của client sẽ bị đóng
     * @param string $buffer
     * @return int
     */
    public static function input($buffer)
    {
        // Lấy vị trí ký tự xuống dòng "\n"
        $pos = strpos($buffer, "\n");
        // Không có ký tự xuống dòng, không lấy được độ dài gói, trả về 0 để tiếp tục chờ dữ liệu
        if($pos === false)
        {
            return 0;
        }
        // Có ký tự xuống dòng, trả về độ dài hiện tại của gói (bao gồm ký tự xuống dòng)
        return $pos+1;
    }

    /**
     * Đóng gói, khi gửi dữ liệu cho client sẽ tự động gọi
     * @param string $buffer
     * @return string
     */
    public static function encode($buffer)
    {
        // Chuyển đổi thành chuỗi Json và thêm ký tự xuống dòng để làm dấu kết thúc yêu cầu
        return json_encode($buffer)."\n";
    }

    /**
     * Giải gói, khi số lượng byte dữ liệu nhận được bằng với giá trị trả về từ input (lớn hơn 0) sẽ tự động gọi
     * và truyền vào tham số $data của hàm gọi lại onMessage
     * @param string $buffer
     * @return string
     */
    public static function decode($buffer)
    {
        // Xóa ký tự xuống dòng, trả về dưới dạng mảng
        return json_decode(trim($buffer), true);
    }
}
```

Đến đây, giao thức JsonNL đã được triển khai hoàn chỉnh, có thể sử dụng trong dự án MyApp, cách sử dụng như sau:

Tệp: MyApp\start.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data chính là dữ liệu client gửi tới, dữ liệu đã được xử lý bởi JsonNL::decode
    echo $data;
    
    // Dữ liệu gửi bằng $connection->send sẽ tự động gọi phương thức JsonNL::encode để đóng gói, sau đó mới gửi tới client
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
...
```

> **Lưu ý**
> Workerman sẽ cố gắng tải giao thức trong không gian tên `Protocols`, ví dụ: `new Worker('JsonNL://0.0.0.0:1234')` sẽ cố tải giao thức `Protocols\JsonNL`.
> Nếu gặp lỗi `Class 'Protocols\JsonNL' not found`, vui lòng tham khảo [tự động tải](../faq/autoload.md) để triển khai tải tự động.

### Giải thích giao diện giao thức
Trong Workerman, lớp giao thức phải triển khai ba phương thức tĩnh là input, encode, decode, giao diện giao thức được giải thích trong Workerman/Protocols/ProtocolInterface.php, định nghĩa như sau:

```php
namespace Workerman\Protocols;

use \Workerman\Connection\ConnectionInterface;

/**
 * Giao diện giao thức
* @author walkor <walkor@workerman.net>
 */
interface ProtocolInterface
{
    /**
     * Dùng để chia gói trong recv_buffer
     *
     * Nếu có thể lấy được độ dài gói từ $recv_buffer thì sẽ trả về độ dài toàn bộ gói
     * Ngược lại trả về 0, có nghĩa cần dữ liệu hơn nữa để lấy độ dài gói hiện tại
     * Nếu trả về false hoặc giá trị âm là yêu cầu không hợp lệ, kết nối sẽ bị đóng
     *
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     * @return int|false
     */
    public static function input($recv_buffer, ConnectionInterface $connection);

    /**
     * Dùng để giải gói yêu cầu
     *
     * Nếu giá trị trả về từ input lớn hơn 0, và Workerman đã nhận đủ dữ liệu, thì sẽ tự động gọi decode
     * Sau đó gọi lại hàm gọi lại onMessage và truyền dữ liệu giải gói từ decode cho tham số thứ hai của hàm gọi lại onMessage
     * Có nghĩa là khi nhận được yêu cầu từ client hoàn chỉnh, sẽ tự động gọi decode để giải mã, không cần phải gọi thủ công trong mã logic kinh doanh
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     */
    public static function decode($recv_buffer, ConnectionInterface $connection);

    /**
     * Dùng để đóng gói yêu cầu
     *
     * Khi cần gửi dữ liệu cho client thông qua lệnh $connection->send($data), sẽ tự động đóng gói $data bằng encode, biến thành định dạng dữ liệu theo giao thức, sau đó mới gửi cho client
     * Có nghĩa dữ liệu gửi cho client sẽ tự động đóng gói bằng encode, không cần gọi thủ công trong mã logic kinh doanh
     * @param ConnectionInterface $connection
     * @param mixed $data
     */
    public static function encode($data, ConnectionInterface $connection);
}
```

## Chú ý:
Trong Workerman, không bắt buộc lớp giao thức phải triển khai theo giao diện ProtocolInterface, thực tế chỉ cần chứa ba phương thức tĩnh input, encode, decode là đủ.
