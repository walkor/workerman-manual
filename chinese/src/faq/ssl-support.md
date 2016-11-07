# 传输加密-ssl/tsl

**问：**

如何保证和Workerman之间通讯安全？

**答：**

实际上通讯安全属于应用层解决的问题，比较方便的做法是增加一层ssl/tsl协议，比如浏览器的wss、https协议都是基于ssl/tsl加密传输的，非常安全。

在Workerman前增加一层Nginx或者STunnel的ssl/tsl代理，让Nginx或者STunnel负责加解密数据，在Workerman和客户端之间转发数据，即可非常方便的实现ssl/tsl加密传输。

当然开发者也可以基于某些加解密算法实现一套自己的加解密机制。

以下是nginx作为通用的ssl/tsl代理的配置示例，适用于包括wss和https在内的各种加密协议。


## nginx配置
要求：

1:nginx版本必须大于1.9.0

2:编译的时候必须加上这两个参数　–with-stream –with-stream_ssl_module

```
stream {
    upstream stream_backend {
         server backend1.example.com:8282;
         server backend2.example.com:8282;
         server backend3.example.com:8282;
    }

    server {
        listen                443 ssl;
        proxy_pass            stream_backend;

        ssl_certificate       /etc/ssl/certs/server.crt;
        ssl_certificate_key   /etc/ssl/certs/server.key;
        ssl_protocols         SSLv3 TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers           HIGH:!aNULL:!MD5;
        ssl_session_cache     shared:SSL:20m;
        ssl_session_timeout   4h;
        ssl_handshake_timeout 30s;
     }
}
```

