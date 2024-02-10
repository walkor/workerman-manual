# Protocole frame
``` (Nécessite Workerman version >=3.3.0) ```

Workerman définit un protocole appelé frame, dont le format est ```longueur totale du paquet + corps du paquet```, où la longueur du paquet est un entier de 4 octets en ordre de octets réseau, et le corps du paquet peut être du texte ordinaire ou des données binaires.

Voici l'implémentation du protocole frame.
```php
class Frame
{

    public static function input($buffer, TcpConnection $connection)
    {
        if(strlen($buffer) < 4)
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
