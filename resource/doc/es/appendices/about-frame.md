# Protocolo de marco
``` (Requiere Workerman versión >= 3.3.0) ```

Workerman ha definido un protocolo llamado "marco" (frame), con el formato de protocolo siendo ```length+body```, donde "length" es un entero de 4 bytes en orden de bytes de red, y "body" puede ser un texto normal o datos binarios.

A continuación se muestra la implementación del protocolo de marco.

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
