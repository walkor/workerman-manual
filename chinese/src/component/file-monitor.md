# 文件监控组件

**背景：**

Workerman是常驻内存运行的，常驻内存可以避免重复读取磁盘、重复解释编译PHP，以便达到最高性能。所以更改业务代码后需要手动reload或者restart才能生效。

同时workerman提供一个监控文件更新的服务，该服务检测到有文件更新后会自动运行reload，从新载入PHP文件。开发者将其放入到项目中随着项目启动即可。


**文件监控服务下载地址：**

1、无依赖版本：https://github.com/walkor/workerman-filemonitor

2、依赖inotify版本：https://github.com/walkor/workerman-filemonitor-inotify (需要安装[inotify扩展](http://php.net/manual/zh/book.inotify.php))


**两个版本区别：**

地址1版本使用的是每秒轮询文件更新时间的方法判断文件是否更新，

地址2利用Linux内核[inotify](http://baike.baidu.com/view/2645027.htm)机制，文件更新时系统会主动通知workerman。

一般使用第一个无依赖版本即可

**使用方法**

将Applications/FileMonitor目录拷贝到你项目的Applications目录下即可。

如果你的项目没有Applications目录，可以将Applications/FileMonitor/start.php文件拷贝到你的项目任意位置，在启动脚本中require到启动脚本中即可。

监控组件默认监控的是Applications目录，如果需要更改，可以修改Applications/FileMonitor/start.php中的```$monitor_dir```变量，```$monitor_dir```的值建议是绝对路径。

**注意：**

* windows系统不支持reload，无法使用此监控服务。
* 只有在debug模式下才生效，daemon下不会执行文件监控（为何不支持daemon模式见下面说明）。
* 只有在Worker::runAll运行后加载的文件才能热更新，或者说只有在onXXX回调中加载的文件才能热更新。


**为何不支持daemon模式?**

daemon模式一般为线上正式环境运行的模式。正式环境发布版本时，一般一次发布多个文件，文件之间也可能有依赖。由于多个文件同步到磁盘需要一定时间，会存在某个时刻磁盘上的文件不全的情况，如果这时候监控到了文件更新并执行了reload，则会有找不到文件导致致命错误的风险。

另外正式环境中有时候会在线定位bug，如果直接编辑代码保存，就会立刻生效，有可能出现语法错误导致线上服务不可用。正确的方法应该是保存代码后，通过```php -l yourfile.php```检查下是否有语法错误，然后再reload热更新代码。

如果开发者确实需要daemon模式开启文件监控及自动更新，可以自行更改Applications/FileMonitor/start.php代码，将Worker::$daemonize部分的判断去掉即可。
