# Protocollo frame
``` (Richiede Workerman versione >= 3.3.0) ```

Workerman ha definito un protocollo chiamato "frame", il formato del protocollo è ```lunghezza totale del pacchetto + corpo del pacchetto```, dove la lunghezza è un intero in formato di 4 byte in ordine di byte di rete e il corpo del pacchetto può essere testo normale o dati binari.

Di seguito è riportata limplementazione del protocollo frame.
```php
class Frame
{

    public static function input($buffer, TcpConnection $connection)
    {
        if(strlen($buffer) < 4)
        {
            return 0;
        }
        $unpacked_data = unpack('Ntotal_length', $buffer);
        return $unpacked_data['total_length'];
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
