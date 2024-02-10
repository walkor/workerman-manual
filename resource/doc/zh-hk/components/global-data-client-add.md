# 加入
**```（要求Workerman版本>=3.3.0）```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
原子添加。如果key已經存在，會返回false。

## 參數

``` $key ```

鍵值。（例如```$global->abc```，```abc```就是鍵值）

``` $value ```

存儲的值。

## 返回值
成功返回true，否則返回false。

## 範例

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // $global->some_key賦值成功
    echo "add success " , $global->some_key;
}
else
{
    // $global->some_key已經存在，賦值失敗
    echo "add fail " , var_export($global->some_key);
}
```
