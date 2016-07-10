# WebServer

WorkerMan自带了一个简单的Web服务器，同样也是基于Worker实现的。文件位置在Workerman/WebServer.php。这个WebServer开发的目的主要是为了方便运行一些简单的Web程序，例如workerman-todpole等web界面程序。

## 使用方法

在Applications/YourApp/start.php中添加

```php
use \Workerman\Worker;
use \Workerman\WebServer;
require_once './Workerman/Autoloader.php';

// 这里监听8080端口，如果要监听80端口，需要root权限，并且端口没有被其它程序占用
$webserver = new WebServer('http://0.0.0.0:8080');
// 类似nginx配置中的root选项，添加域名与网站根目录的关联，可设置多个域名多个目录
$webserver->addRoot('www.example.com', '/your/path/of/web/');
$webserver->addRoot('blog.example.com', '/your/path/of/blog/');
// 设置开启多少进程
$webserver->count = 4;

Worker::runAll();
```

# WorkerMan的Webserver与普通Web开发异同

### 1、普通Web程序架构运行机制
一般的Web程序一般都是基于nginx+php-fpm或者apache+php的架构开发的，这些架构的运行机制一般是是每个请求都会经过请求初始化、创建执行环境、词法解析、语法解析、编译生成opcode以及请求关闭释放各种资源（如果有opcode缓存会跳过词法解析、语法解析、编译生成opcode步骤）

### 2、WorkerMan架构Web程序运行机制
WorkerMan是常驻内存的运行机制，只要PHP文件被载入编译过一次，便会常驻内存，不会再去从磁盘读取或者再去编译，并省去了重复的请求初始化、创建执行环境、词法解析、语法解析、编译生成opcode以及请求关闭释放各种资源等诸多耗时的步骤。剩下的只是简单的计算过程，所以性能很高。正因为常驻内存，所以类、函数、常量等定义代码只要运行一次，便可以永久使用，不会被销毁，所以要避免反复加载类、函数、常量等定义文件。比较简单的办法是使用require_once加载文件，避免重复加载重复定义。

### 3、避免使用exit、die语句
同样的，在程序中避免使用exit、die语句，使用exit、die会导致进程退出。可以使用```\Workerman\Protocols\Http::end($msg)```函数替代exit、die函数。

### 4、HTTP相关函数的使用
WorkerMan运行在PHP CLI模式下，PHP CLI模式下无法使用HTTP相关的函数，例如```header、setcookie、session_start```等函数，请使用```/Workerman/Protocols/Http.php```文件中的```header、setcookie、sessionStart```等方法替换。

### 5、Web入口文件
WorkerMan的WebServer默认使用index.php作为Web入口文件，例如配置```$webserver->setRoot('www.example.com', '/home/www/');```，则www.example.com的入口文件为```/home/www/index.php```。当url访问的文件（包括静态文件和PHP文件）不存在时，会自动调用入口文件index.php

### 6、可用的超全局变量
可用的超全局变量有```$_SERVER、$_GET、$_POST、$_FILES、$_COOKIE、$_SESSION、$_REQUEST```。

注意HTTP文件上传中，WorkerMan的```$_FILES```结构与传统PHP中的```$_FILES```结构不同，WorkerMan中```$_FILES```结构类似
```php
var_export($_FILES);
array(
    0 => array(
        'file_name' => 'logo.png', // 文件名称
        'file_size' => 23654,      // 文件大小
        'file_data' => '*****',    // 文件的二进制数据
    ),
    1 => array(
        'file_name' => 'file.tar.gz', // 文件名称
        'file_size' => 128966,        // 文件大小
        'file_data' => '*****',       // 文件的二进制数据
    ),
    ...
);

```

保存文件代码类似
```php
// 例如保存到/tmp目录下
foreach($_FILES as $file_info)
{
    file_put_contents('/tmp/'.$file_info['file_name'], $file_info['file_data']);
}

```

WorkerMan中无法使用```move_uploaded_file() is_uploaded_file()```这些函数。

### 7、可以设置onWorkerStart、onWorkerStop回调
可以设置onWorkerStart、onWorkerStop回调，做进程启动时全局初始化及进程退出（stop等命令）数据保存清理工作

