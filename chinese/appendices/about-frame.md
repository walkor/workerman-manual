# frame协议
``` (需要Workerman版本>=3.3.0) ```

workerman定义了一种叫做frame的协议，协议格式为 ```总包长+包体```，其中包长为4字节网络字节序的整数，包体可以是普通文本或者二进制数据。

以下是frame协议实现。
```php
class Frame
{

    public static function input($buffer ,TcpConnection $connection)
    {
        if(strlen($buffer)<4)
        {
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
        return pack('N',$total_length) . $buffer;
    }
}
```




