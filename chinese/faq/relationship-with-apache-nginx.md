# 与Apache和Nginx的关系
**问：**

Workerman和Apache/nginx/php-fpm是什么关系？Workerman和Apache/nginx/php-fpm
冲突么？

**答：**
Workerman和Apache/nginx/php-fpm没有任何关系，并且Workerman的运行不依赖于Apache/nginx/php-fpm。他们都是独立的容器，互不干扰，也不会冲突（在不监听同一个端口的情况下）。

Workerman是一个通用的socket服务器框架，支持长连接，支持各种协议如HTTP、WebSocket以及自定义协议。而Apache/nginx/php-fpm一般来说只用于开发HTTP协议的Web项目。

如果服务器已经部署了Apache/nginx/php-fpm，部署Workerman不会影响到它们的运行。
