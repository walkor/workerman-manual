# 如何分布式WorkerMan
##关键点
### 1、设置Gateway实例的```lanIp```与当前服务器内网ip一致
### 2、```Applications/XXX/Config/Store.php``` 中 redis 相关配置
```php
// self::DRIVER_REDIS代表使用redis存储
public static $driver = self::DRIVER_REDIS;
// 改成redis服务器内网ip端口
public static $gateway = array(
     'redis内网ip:redis端口',
);
```

## 部署示例
以Applications/Demo为例，假如需要部署三台服务器(192.168.1.1-3)提供高可用服务。。另外有一台redis服务器（ip 192.168.1.4，端口6379）做全局数据共享。

1、给三台服务器的PHP添加redis扩展。ubuntu/debian可使用 ```sudo apt-get install php5-redis```安装。centos系统使用```yum install php-pecl-redis```。

2、配置三台服务器```Applications/Demo/Config/Store.php```如下

```php
// 存储驱动改为redis
public static $driver = self::DRIVER_REDIS
// 更改redis ip和端口
public static $gateway = array(
     '192.168.1.4:6379',
);
// 如果有其它的配置如workerman-chat中的$room配置，也需要将其改成redis的ip和端口
...
```

3、分别配置三台服务器Gateway对象的```lanIp```为当前服务器的内网ip。例如配置192.168.1.1服务器Gateway实例


Applications/Demo/start.php中设置
```
$gateway = new Gateway("yourProtocol://0.0.0.0:your_port");
....
$gateway->lanIp = 192.168.1.1;
....

```

4、逐台启动WorkerMan，至此WorkerMan分布式部署完毕。

**说明：**

1、三台WorkerMan机器都运行了Gateway进程和Worker进程，客户端连接上任意一台WorkerMan的Gateway端口即可。

2、为了方便前端接入和扩容，可以在Gateway前加一层DNS、LVS等负载均衡策略

3、如果服务器不够用可以使用同样的方法增加服务器

4、如果需要下线服务器，可以停止WorkerMan，然后执行后续停机等下线操作(由于Gateway进程维护着客户端连接，当对应服务器下线时，对应服务器的客户端会掉线一次。如何做到下线机器不影响用户参考下一节)。


