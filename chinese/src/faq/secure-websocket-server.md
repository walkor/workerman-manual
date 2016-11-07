# 创建wss服务

**问：**

Workerman如何创建一个wss服务(或https服务)，使得客户端可以用过wss协议(或https协议)来连接通讯。


**答：**

可以利用nginx作为ssl的透明代理，实现wss转换成ws或者https转换成http转发到Workerman。

nginx作为ssl透明代理的配置参考如下，此配置同样适用于其它ssl加密的协议，包括https。

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

