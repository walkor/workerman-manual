# Giao thức frame

``` (Yêu cầu phiên bản Workerman >= 3.3.0) ```

Workerman định nghĩa một giao thức gọi là frame, định dạng của giao thức là ```tổng chiều dài gói + nội dung gói```, trong đó chiều dài gói là số nguyên 4 byte theo thứ tự mạng, nội dung gói có thể là văn bản thông thường hoặc dữ liệu nhị phân.

Dưới đây là cách triển khai giao thức frame.

```php
class Frame
{

    public static function input($buffer, TcpConnection $connection)
    {
        if (strlen($buffer) < 4) {
            return 0;
        }
        $unpack_data = unpack('Ntotal_length', $buffer);
        return $unpack_data['total_length'];
    }

    public static function decode($buffer)
    {
        return substr($buffer, 4);
    }

    public static function encode($buffer)
    {
        $total_length = 4 + strlen($buffer);
        return pack('N', $total_length) . $buffer;
    }
}
```
