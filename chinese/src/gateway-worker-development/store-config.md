# Config/Store 配置

###配置作用
配置Gateway/Worker模型中存储驱动类型及存储位置（数据文件位置或者memcache通讯ip端口）。以便Store类存储Gateway与BusinessWorker之间的通讯ip与端口，以及存储各个客户端的通讯ip与端口。

### 配置示例及说明

```php
class Store
{
    // 使用文件存储，注意使用文件存储无法支持workerman分布式部署
    const DRIVER_FILE = 1;
    // 使用memcache存储，支持workerman分布式部署
    const DRIVER_MC = 2;

    /* 使用哪种存储驱动 文件存储DRIVER_FILE 或者 memcache存储DRIVER_MC，为了更好的性能请使用DRIVER_MC
     * 注意： DRIVER_FILE只适合开发环境，生产环境或者压测请使用DRIVER_MC，需要php cli 安装memcache扩展
     */
    public static $driver = self::DRIVER_FILE;

    // 如果是memcache存储，则在这里设置memcache的ip端口，注意确保你安装了memcache扩展
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

1、Store类有两种存储驱动，文件存储（DRIVER_FILE）及memcache存储(DRIVER_MC)。文件存储适合开发调试使用，memcache存储适合生产环境使用。

2、如果使用文件存储，存储文件会放置于```$storePath```指定的路径下。如果运行多个Gateway/Worker模型的项目，请确保多个项目之间的```$storePath```路径不要冲突。

3、如果是memcache存储，请安装memcached服务端及php的memcached扩展。并设定```$gateway``` ```$room```（如果有的话）中的ip和端口为memcache服务端的ip端口。

4、如果是memcache存储，并且业务想添加自己的存储实例，可以开启新的memcache服务端ip端口，然后添加一项配置。例如新增加一个user存储实例，memcache ip端口为192.168.1.2:11211，则只需要增加如下一项到\Config\Store类中
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
1、线上环境请使用memcache存储，即设置
```php
public static $driver = self::DRIVER_MC
```

2、开发者不要使用```Store::instance('gateway');```、```Store::instance('room');```，也要避免业务新增配置memcache的ip端口与gateway或者room的配置相同，以免造成业务数据与框架数据冲突。

