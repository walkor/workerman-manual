# インクリメント
**```（Workermanバージョン>=3.3.0が必要です）```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```
アトミックインクリメント。指定されたサイズのステップを数値要素に追加します。 要素の値が数値型でない場合、0として扱ってから追加します。要素が存在しない場合はfalseを返します。

## パラメーター

 ``` $key ```

キー値。（例：```$global->abc```、```abc```がキー値です）

 ``` $value ```

要素の値を増やすサイズ。

## 戻り値
成功した場合はtrueを、それ以外の場合はfalseを返します。

## 例

```php
$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// 非アトミックインクリメント
$global->some_key++;

echo $global->some_key."\n";

// アトミックインクリメント
$global->increment('some_key');

echo $global->some_key."\n";
```
