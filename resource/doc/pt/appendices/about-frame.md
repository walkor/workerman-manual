# Protocolo de quadro
``` (Necessária a versão do Workerman >= 3.3.0) ```

O Workerman define um protocolo chamado de protocolo de quadro, no qual o formato do protocolo é ``` comprimento total do pacote + corpo do pacote ```, onde o comprimento do pacote é um inteiro de 4 bytes em ordem de bytes de rede, e o corpo do pacote pode ser texto simples ou dados binários.

Aqui está a implementação do protocolo de quadro.
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
