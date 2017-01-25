# 创建https服务

**问：**

Workerman如何创建一个https服务，使得客户端可以用过https协来连接通讯。


**答：**

可以利用nginx作为ssl的代理。通讯原理及流程是：

1、客户端发起https连接连到nginx

2、nginx将https协议的数据转换成http协议并转发到Workerman

3、Workerman收到数据后做业务逻辑处理，返回http协议的数据给nginx

4、nginx再将http协议的数据转换成https，转发给客户端


## nginx配置参考
**前提条件及准备工作：**

1、假设Workerman监听的是80端口(http协议)

2、已经申请了证书（pem/crt文件及key文件）放在了/etc/nginx/conf.d/ssl下

3、打算利用nginx开启4431端口对外提供wss代理服务（端口可以根据需要修改）

**nginx配置类似如下**：

```
server {
  listen 4431;

  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/laychat/laychat.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/laychat/laychat.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /
  {
    proxy_pass http://127.0.0.1:80;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
  }
}
```

