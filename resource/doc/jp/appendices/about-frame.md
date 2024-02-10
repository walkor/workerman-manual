# フレームプロトコル
```（Workermanのバージョン>=3.3.0が必要です）```


Workermanは、フレームと呼ばれるプロトコルを定義しており、このプロトコルの形式は ```総パケット長+パケットのボディ``` です。ここで、パケット長は4バイトのネットワークバイトオーダーの整数であり、パケットのボディには通常のテキストまたはバイナリデータが含まれます。

以下はフレームプロトコルの実装です。
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
