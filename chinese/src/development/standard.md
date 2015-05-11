# 开发规范

## 应用程序目录

应用程序目录可以放到任意位置

## 入口文件

和nginx+PHP-FPM下的PHP应用程序一样，WorkerMan中的应用程序也需要一个入口文件，入口文件名没有要求，并且这个入口文件是以PHP Cli方式运行的。

 入口文件中是创建监听进程相关的代码，例如下面的基于Worker开发的代码片段

 test.php
```php
<?php
use Workerman\Worker;

// 创建一个Worker监听2345端口，使用http协议通讯
$http_worker = new Worker("http://0.0.0.0:2345");

// 启动4个进程对外提供服务
$http_worker->count = 4;

// 接收到浏览器发送的数据时回复hello world给浏览器
$http_worker->onMessage = function($connection, $data)
{
    // 向浏览器发送hello world
    $connection->send('hello world');
};

Worker::runAll();

```

## WorkerMan中的代码规范

1、类采用首字母大写的驼峰式命名，类文件名称必须与文件内部类名相同，以便自动加载。例如：
```php
class UserInfo
{
...
```

2、使用命名空间，命名空间名字与目录路径对应，并以开发者的项目根目录为基准。

例如项目MyApp/，类文件MyApp/MyClass.php因为在项目根目录，所以命名空间省略。类文件MyApp/Protocols/MyProtocol.php因为MyProtocol.php在MyApp项目的Protocols目录下，所以要加上命名空间 ```namespace Protocols;```，如下：
```php
namespace Protocols;
class MyProtocol
{
....
```

3、普通函数及变量名采用小写加下划线方式 例如
```php
$connection_list = array();
function get_connection_list()
{
....
```

4、类成员及类的方法采用首字母小写的驼峰形式 例如：
```php
public $connectionList;
public function getConnectionList();
```

5、函数及类的参数采用小写加下划线方式
```php
function get_connection_list($one_param, $tow_param)
{
....

```






