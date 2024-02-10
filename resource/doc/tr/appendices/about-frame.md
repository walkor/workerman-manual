# frame protokolü
``` (Workerman sürümü >=3.3.0 gerektirir) ```

workerman, "frame" adında bir protokol tanımlar. Protokol formatı ```toplam_paket_uzunluğu+paket_gövdesi``` şeklindedir, burada paket uzunluğu 4 baytlık ağ baytı düzeninde bir tamsayıdır ve paket gövdesi normal metin veya ikili veri olabilir.

Aşağıda frame protokolünün uygulanışı bulunmaktadır.

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
