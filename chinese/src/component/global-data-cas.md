# cas
**``` (要求Workerman版本>=3.3.0) ```**
```php
bool \GlobalData\Client::cas(string $key, mixed $old_value, mixed $new_value)
```
原子替换，用```$new_value```替换```$old_value```。
仅在当前客户端最后一次取值后，该key对应的值没有被其他客户端修改的情况下， 才能够将值写入。

## 参数

``` $key ```

键值。（例如```$global->abc```，```abc```就是键值）

``` $old_value ```

老数据


``` $new_value ```

新数据

## 返回值
替换成功返回true，否则返回false。

## 说明：

多进程同时操作同一个共享变量时，有时候要考虑并发问题。

例如A B两个进程同时给用户列表添加一个成员。<br>
A B进程当前用户列表都为```$global->user_list = array(1,2,3)```。<br>
A进程操作```$global->user_list```变量，添加一个用户4。<br>
B进程操作```$global->user_list```变量，增加一个用户5。<br>
A进程设置变量```$global->user_list = array(1,2,3,4)```成功。<br>
B进程设置变量```$global->user_list = array(1,2,3,5)```成功。<br>
此时B进程设置的变量将A进程设置的变量覆盖，导致数据丢失。<br>

以上由于读取和设置不是一个原子操作，导致并发问题。<br>
要解决这种并发问题，可以使用cas原子替换接口。<br>
cas接口在改变一个值之前，<br>
会根据```$old_value```判断这个值是否被其它进程更改过，<br>
如果有更改，则不替换，返回false。否则替换返回true。<br>
见下面示例。

**注意： **  <br>
有些共享数据被并发覆盖是没问题的，例如竞拍系统某拍卖物当前最大报价，例如某商品当前库存等。


## 范例

```php
require_once __DIR__ . '/../src/Client.php';

$global = new GlobalData\Client('127.0.0.1:2207');

// 初始化列表
$global->user_list = array(1,2,3);

// 向user_list原子添加一个值
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```
