# cas
**``` (要求Workerman版本>=3.3.0) ```**
```php
bool \GlobalData\Client::cas(string $key, mixed $old_value, mixed $new_value)
```
原子替換，用```$new_value```替換```$old_value```。
僅在當前客戶端最後一次取值後，該key對應的值沒有被其他客戶端修改的情況下，才能夠將值寫入。

## 參數

 ``` $key ```

鍵值。（例如```$global->abc```，```abc```就是鍵值）

 ``` $old_value ```

舊數據


 ``` $new_value ```

新數據

## 返回值
替換成功返回true，否則返回false。

## 說明：

多進程同時操作同一個共享變量時，有時候要考慮並發問題。

例如A B兩個進程同時給用戶列表添加一個成員。
A B進程當前用戶列表都為```$global->user_list = array(1,2,3)```。
A進程操作```$global->user_list```變量，添加一個用戶4。
B進程操作```$global->user_list```變量，增加一個用戶5。
A進程設置變量```$global->user_list = array(1,2,3,4)```成功。
B進程設置變量```$global->user_list = array(1,2,3,5)```成功。
此時B進程設置的變量將A進程設置的變量覆蓋，導致數據丟失。

以上由於讀取和設置不是一個原子操作，導致並發問題。
要解決這種並發問題，可以使用cas原子替換接口。
cas接口在改變一個值之前，
會根據```$old_value```判斷這個值是否被其它進程更改過，
如果有更改，則不替換，返回false。否則替換返回true。
見下面示例。

 **注意：** 
有些共享數據被並發覆蓋是沒問題的，例如競拍系統某拍賣物當前最大報價，例如某商品當前庫存等。


## 範例

```php
$global = new GlobalData\Client('127.0.0.1:2207');

// 初始化列表
$global->user_list = array(1,2,3);

// 向user_list原子添加一個值
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```
