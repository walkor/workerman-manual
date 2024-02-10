＃追加
**```（Workermanのバージョン>=3.3.0が必要です）```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
アトミックな追加。キーが既に存在する場合、falseを返します。

## パラメータ

 ``` $key ```
キーの値。（例えば```$global->abc```の場合、```abc```がキーの値です）

 ``` $value ```
保存する値。

## 戻り値
成功した場合はtrue、それ以外の場合はfalseを返します。

## 例

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // $global->some_keyへの代入に成功しました
    echo "add success " , $global->some_key;
}
else
{
    // $global->some_keyはすでに存在するため、代入に失敗しました
    echo "add fail " , var_export($global->some_key);
}
```
