# Config/Store 配置

###配置作用
配置Gateway/Worker模型中存储驱动类型及存储位置，以便Store类存储Gateway与BusinessWorker之间的通讯ip与端口，以及存储各个客户端的通讯ip与端口。

### 配置示例及说明

```php
class Store
{
    // 使用文件存储，注意使用文件存储无法支持workerman分布式部署
    const DRIVER_FILE = 1;
    // memcache存储
    const DRIVER_MC = 2;
    // redis存储
    const DRIVER_REDIS = 3;

    /* 使用哪种存储驱动 文件存储DRIVER_FILE 或者 memcache存储DRIVER_MC或者redis存储DRIVER_REDIS，为了更好的性能请使用DRIVER_REDIS。正式环境建议使用redis存储
     */
    public static $driver = self::DRIVER_FILE;

    // 如果是memcache/redis存储，则在这里设置memcache/redis的ip端口，注意确保你安装了memcache/redis扩展
    public static $gateway = array(
        '127.0.0.1:22301',
    );

    public static $room = array(
        '127.0.0.1:22301',
    );

    /* 如果使用文件存储（$driver = self::DRIVER_FILE），则在这里设置数据存储的目录，默认/tmp/下
     */
    public static $storePath = '/tmp/workerman-chat/';
}
```

1、Store类有三种存储驱动，文件存储（DRIVER_FILE）及memcache存储(DRIVER_MC)以及redis存储(DRIVER_REDIS)。正式环境建议使用redis存储。

2、如果使用文件存储，存储文件会放置于```$storePath```指定的路径下。如果运行多个Gateway/Worker模型的项目，请确保多个项目之间的```$storePath```路径不要冲突。

3、如果是memcache/redis存储，请安装memcached/redis服务端及php的memcached/redis扩展。并设定```$gateway``` ```$room```（如果有的话）中的ip和端口为memcache/redis服务端的ip端口。

4、如果是memcache/redis存储，并且业务想添加自己的存储实例，可以开启新的memcache/redis服务端ip端口，然后添加一项配置。例如新增加一个user存储实例，memcache/redis ip端口为192.168.1.2:11211，则只需要增加如下一项到\Config\Store类中
```php
public static $user= array(
    '127.0.0.1:11211',
);
```
使用的时候可以这样使用
```php
\GatewayWorker\Lib\Store::instance('user')->get/set..
```

### 请开发者注意
1、线上环境请使用redis存储，即设置
```php
public static $driver = self::DRIVER_REDIS
```

2、开发者不要使用```Store::instance('gateway');```、```Store::instance('room');```这两个实例，也要避免业务新增配置memcache/redis的ip端口与gateway或者room的配置相同，以免造成业务数据与框架数据冲突。

3、redis扩展安装方法

centos系统：yum install php-pecl-redis

ubuntu/debian系统：apt-get install php5-redis

pecl 命令安装：pecl install redis

其他方法参见手册11.2安装扩展章节

