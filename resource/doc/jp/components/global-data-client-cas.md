```php
bool \GlobalData\Client::cas(string $key, mixed $old_value, mixed $new_value)
```
CASは、```$new_value```で```$old_value```を置き換えます。
現在のクライアントの最後の値を取得した後、そのキーに対応する値が他のクライアントによって変更されていない場合にのみ、値を書き込むことができます。

## パラメータ

``` $key ```

キー（例：```$global->abc```の場合、```abc```がキーです）

``` $old_value ```

古いデータ

``` $new_value ```

新しいデータ

## 戻り値
置換成功でtrueを返し、それ以外の場合はfalseを返します。

## 説明：

複数のプロセスが同じ共有変数を同時に操作する場合、同時実行の問題を考慮する必要があります。

例えば、AとBの二つのプロセスが同時にユーザーリストにメンバーを追加する必要がある場合を考えてみましょう。
AとBのプロセスは現在、ユーザーリストが```$global->user_list = array(1,2,3)``` であるとします。
Aプロセスが```$global->user_list```変数を操作し、ユーザー4を追加します。
Bプロセスが```$global->user_list```変数を操作し、ユーザー5を追加します。
Aプロセスが変数を設定し```$global->user_list = array(1,2,3,4)```に成功します。
Bプロセスが変数を設定し```$global->user_list = array(1,2,3,5)```に成功します。
この時、Bプロセスの設定がAプロセスの設定を上書きし、データが失われてしまいます。

これは、読み取りと設定が原子操作ではないため、同時実行の問題が発生します。
この種の同時実行の問題を解決するために、CAS（比較交換）原子置換インターフェースを使用することができます。
CASインタフェースは、値を変更する前に、```$old_value```に基づいてその値が他のプロセスによって変更されたかどうかを判断し、変更があれば置換せずにfalseを返します。そうでなければtrueを返します。
以下に例を示します。

**注：**
競りシステムでの競り物の最大現在価格、商品の在庫など、一部の共有データは同時に上書きされても問題ありません。

## 例

```php
$global = new GlobalData\Client('127.0.0.1:2207');

// リストの初期化
$global->user_list = array(1,2,3);

// user_listに値を原子的に追加する
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```
