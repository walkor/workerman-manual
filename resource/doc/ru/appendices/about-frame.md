# Протокол frame
``` (Требуется версия Workerman >= 3.3.0) ```

Workerman определяет протокол с именем frame, где формат протокола состоит из ```общей длины пакета + тела пакета```, где общая длина - это целое число из 4 байтов в сетевом порядке байтов, а тело может быть обычным текстом или двоичными данными.

Ниже приведена реализация протокола frame:
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
