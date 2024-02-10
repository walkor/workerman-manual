# بروتوكول الإطار (Frame Protocol)

``` (يتطلب Workerman الإصدار 3.3.0 أو أحدث) ```

قام Workerman بتعريف بروتوكول يُعرف ببروتوكول الإطار (Frame Protocol)، حيث يكون تنسيق البروتوكول عبارة عن ``` الحزمة الكلية + الجسم ```، حيث يكون طول الحزمة عبارة عن عدد صحيح مكون من 4 بايتات وبالترتيب الشبكوي، ويمكن أن يكون الجسم عبارة عن نصوص عادية أو بيانات ثنائية.

وفيما يلي تنفيذ بروتوكول الإطار.

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

