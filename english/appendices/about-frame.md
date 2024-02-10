# Frame Protocol
``` (Requires Workerman version >= 3.3.0) ```

Workerman defines a protocol called frame, with the format of ```total length + body```, where the total length is a 4-byte integer in network byte order, and the body can be plain text or binary data.

The following is the implementation of the frame protocol.
```php
class Frame
{
    public static function input($buffer, TcpConnection $connection)
    {
        if (strlen($buffer) < 4)
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
        return pack('N', $total_length) . $buffer;
    }
}
```
