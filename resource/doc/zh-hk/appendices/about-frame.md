# frame協議
```（需要Workerman版本>=3.3.0）```

workerman定義了一種叫做frame的協議，協議格式為```總包長+包體```，其中包長為4字節網絡字節序的整數，包體可以是普通文本或者二進制數據。

以下是frame協議實現。
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
