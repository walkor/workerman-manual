# GlobalData变量共享组件
**``` (要求Workerman版本>=3.3.0) ```**

源码地址：https://github.com/walkor/GlobalData

## 注意
GlobalData需要Workerman版本>=3.3.0

## 下载安装

可以使用composer安装，或者直接下载zip包https://github.com/walkor/GlobalData/archive/master.zip 。

## 原理

利用PHP的```__set __get __isset __unset```魔术方法触发与GlobalData服务端通讯，实际变量存储在GlobalData服务端。例如当给客户端类设置一个不存在的属性时，会触发```__set```魔术方法，客户端类在```__set```方法中向GlobalData服务端发送请求，存入一个变量。当访问客户端类一个不存在的变量时，会触发类的```__get```方法，客户端会向GlobalData服务端发起请求，读取这个值，从而完成进程间变量共享。


```php
require_once __DIR__ . '/../src/Client.php';

// 连接Global Data服务端
$global = new GlobalData\Client('127.0.0.1', 2207);

// 触发$global->__isset('somedata')查询服务端是否存储了key为somedata的值
isset($global->somedata);

// 触发$global->__set('somedata',array(1,2,3))，通知服务端存储somedata对应的值为array(1,2,3)
$global->somedata = array(1,2,3);

// 触发$global->__get('somedata')，从服务端查询somedata对应的值
var_export($global->somedata);

// 触发$global->__unset('somedata'),通知服务端删掉somedata及对应的值
unset($global->somedata);

```


