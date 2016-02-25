# increment
**``` (要求Workerman版本>=3.3.0) ```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```
原子增加。将一个数值元素增加参数step指定的大小。 如果元素的值不是数值类型，将其作为0再做增加处理。如果元素不存在返回false。

## 参数

``` $key ```

键值。（例如```$global->abc```，```abc```就是键值）


``` $value ```

要将元素的值增加的大小。

## 返回值
成功返回true，否则返回false。


## 范例

```php
require_once __DIR__ . '/../src/Client.php';

$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// 非原子增加
$global->some_key++;

echo $global->some_key."\n";

// 原子增加
$global->increment('some_key');

echo $global->some_key."\n";
```
