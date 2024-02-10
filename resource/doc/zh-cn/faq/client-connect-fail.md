# 客户端连接失败原因

连接失败客户端一般会有两种报错，```connection refuse``` 和 ```connection timeout```

## connection refuse（连接拒绝）

一般是以下原因:
1、客户端连接的端口错了
2、客户端连接的域名或者ip错了
3、如果客户端使用了域名连接，域名可能指向了错误的服务器ip
4、服务器使用了cdn等加速代理，导致连接的实际ip与预期ip不一致
5、服务端没有启动或者端口没有被监听
6、使用了网络代理软件
7、服务端监听ip与访问地址不在一个地址段。例如服务端监听127.0.0.1，则客户端只能通过127.0.0.1连接，不能通过局域网ip或者外网ip连接。建议监听地址设置为0.0.0.0，这样本机、内网、外网都可以连接。

## connection timeout（连接超时）
 
一般是以下原因：
1、服务器防火墙阻止了连接，可以临时关闭防火墙试下
2、如果是云服务器，安全组也可能会阻止连接建立，需要到管理后台开放对应端口
3、如果用了宝塔等面板，需要在宝塔中开放对应端口
4、服务器不存在或者没有启动
5、如果客户端使用了域名连接，域名可能指向了错误的服务器ip
6、客户端访问的ip是服务器内网ip，并且客户端和服务端不在一个局域网

## cannot assign requested address (无法分配请求地址)
**作为客户端时**，每发起一个连接需要占用本地一个临时端口，一台服务器默认可用临时端口大概在2-3万，如果向特定服务器发起的连接数超过这个值后将无法分配可用端口，会产生这个错误。
可以通过更改内核参数`/etc/sysctl.conf` 中的 `net.ipv4.ip_local_port_range` 来增加本地临时端口数量，例如设置成`10000 65535`(本地端口范围设置成10000 65535，也就是本地端口数增加到55535个)，运行`sysctl -p`生效。
另外连接断开后连接变成TIME_WAIT状态，仍然会占用对应本地端口一段时间，也就是短时间内发起大量(超过2-3w)短连接也会报`Cannot assign requested address`，如果是这种情况可以通过设置内核快速回收TIME_WAIT来解决，参考[内核调优](https://www.workerman.net/doc/workerman/appendices/kernel-optimization.html)。

> **注意**
> 本地端口数限制仅限于客户端，服务端没有本地端口限制，只要资源足够，服务端维持连接数量可以看作是无限。

## 其它报错
如果发生的报错不是```connection refuse``` 和 ```connection timeout```则一般是以下原因：

**1、客户端使用的通讯协议与服务端不一致。**
例如服务端是http通讯协议，客户端使用websocket通讯协议访问是无法连接的。如果客户端用websocket协议连接，那么服务端必须也是websocket协议。如果服务端是http协议的服务，那么客户端必须用http协议访问。

这里的原理类似如果你要和英国人交流，那么要使用英语。如果要和日本人交流，那么要使用日语。这里的语言就类似通讯协议，双方(客户端和服务端)必须使用相同的语言才能交流，否则无法通讯。

**通讯协议不一致导致的常见的报错有：**

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: Unexpected response code: xxx

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: net::ERR_INVALID_HTTP_RESPONSE

**解决办法：**
从上面两条报错看出，客户端使用的是ws连接是websocket协议。服务端也需要是websocket协议才行，服务端监听部分代码需要指定websocket协议才能通讯，例如下面这样

如果是gatewayWorker，监听部分代码类似
```php
// websocket协议，这样客户端才能用ws://...来连。xxxx为端口不用改动
$gateway = new Gateway('websocket://0.0.0.0:xxxx');

```
如果是Workerman则是
```php
// websocket协议，这样客户端才能用ws://...来连。xxxx为端口不用改动
$worker = new Worker('websocket://0.0.0.0:xxxx');
```



