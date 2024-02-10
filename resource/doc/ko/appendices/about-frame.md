# 프레임 프로토콜
``` (Workerman 버전 >= 3.3.0 이 필요) ```

workerman은 "프레임"이라고 불리는 프로토콜을 정의했습니다. 이 프로토콜 형식은 ```총 길이+패킷 본문```으로, 여기서 패킷 길이는 4바이트의 네트워크 바이트 순서의 정수이며, 패킷 본문은 일반 텍스트 또는 이진 데이터가 될 수 있습니다.

아래는 프레임 프로토콜의 구현입니다.
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
