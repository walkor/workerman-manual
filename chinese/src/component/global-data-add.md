# add
**``` (要求Workerman版本>=3.3.0) ```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
原子添加。如果key已经存在，会返回false.

## 参数

``` $key ```

键值。（例如```$global->abc```，```abc```就是键值）


``` $value ```

存储的值。

## 返回值
成功返回true，否则返回false。


## 范例

```php
require_once __DIR__ . '/../src/Client.php';

$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // $global->some_key赋值成功
    echo "add success " , $global->some_key;
}
else
{
    // $global->some_key已经存在，赋值失败
    echo "add fail " , var_export($global->some_key);
}
```
