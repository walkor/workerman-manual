# 传输加密-ssl/tsl

**问：**

如何保证和Workerman之间通讯安全？

**答：**

实际上通讯安全属于应用层解决的问题，比较方便的做法是增加一层ssl/tsl协议，比如浏览器的wss、https协议都是基于ssl/tsl加密传输的，非常安全。

在Workerman前增加一层Nginx或者STunnel的ssl/tsl代理，让Nginx或者STunnel负责加解密数据，在Workerman和客户端之间转发数据，即可非常方便的实现ssl/tsl加密传输。
配置方法可以参考网上Nginx代理ssl/tsl相关的文章。

当然开发者也可以基于某些加解密算法实现一套自己的加解密机制。

