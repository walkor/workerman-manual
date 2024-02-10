# increment
**``` (要求Workerman版本>=3.3.0) ```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```
原子增加。將一個數值元素增加參數step指定的大小。如果元素的值不是數值類型，將其作為0再做增加處理。如果元素不存在返回false。

## 參數

 ``` $key ```

鍵值。（例如```$global->abc```，```abc```就是鍵值）

 ``` $value ```

要將元素的值增加的大小。

## 返回值
成功返回true，否則返回false。

## 范例

```php
$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// 非原子增加
$global->some_key++;

echo $global->some_key."\n";

// 原子增加
$global->increment('some_key');

echo $global->some_key."\n";
```
