# Frame-Protokoll
``` (Workerman-Version >= 3.3.0 erforderlich) ```

Workerman definiert ein Protokoll namens Frame-Protokoll, das das Format ```Gesamtpaketlänge+Paketkörper``` verwendet. Dabei ist die Paketlänge eine 4-Byte-Ganzahl im Netzwerk-Byte-Order-Format, und der Paketkörper kann gewöhnlicher Text oder binäre Daten sein.

Die Implementierung des Frame-Protokolls ist wie folgt.
```php
class Frame
{

    public static function input($buffer, TcpConnection $connection)
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
