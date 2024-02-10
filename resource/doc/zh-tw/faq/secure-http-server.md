# 創建https服務

**問：**

Workerman如何創建一個[https](https://baike.baidu.com/item/https)服務，使得客戶端可以使用[https](https://baike.baidu.com/item/https)協議來進行通訊。

**答：**

[https](https://baike.baidu.com/item/https)協議實際上是[http](https://baike.baidu.com/item/http) + [SSL](https://baike.baidu.com/item/ssl)，就是在[http](https://baike.baidu.com/item/http)協議上加入[SSL](https://baike.baidu.com/item/ssl)層。Workerman支持[http](https://baike.baidu.com/item/http)協議，同時也支持[SSL](https://baike.baidu.com/item/ssl)（需要Workerman版本>=3.3.7），所以只需要在[http](https://baike.baidu.com/item/http)協議的基礎上開啟[SSL](https://baike.baidu.com/item/ssl)即可支持[https](https://baike.baidu.com/item/https)協議。

讓Workerman支持https有兩種通用方案，一種是Workerman直接開啟SSL，另外一種是利用nginx代理SSL。兩種方案擇其一即可，不可同時設置。

## Workerman開啟SSL

**準備工作：**

1、Workerman版本>=3.3.7

2、PHP安裝了openssl擴展

3、已經申請了證書（pem/crt檔案以及key檔案）並放在/etc/nginx/conf.d/ssl下

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 证书最好是申请的证书
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // 也可以是crt文件
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, //如果是自签名证书需要开启此选项
    )
);
// 这里设置的是http协议
$worker = new Worker('http://0.0.0.0:443', $context);
// 设置transport开启ssl，变成http+SSL即https
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

通過Workerman以上的代碼就創建了https服務，客戶端就可以通過https協議來連接Workerman實現安全加密通訊了。

**測試：**

瀏覽器地址欄輸入```https://域名:443```來訪問。

**注意：**

1、https端口必須使用https協議訪問，http協議無法訪問。

2、證書一般與域名綁定，所以測試的時候請使用域名，不要使用IP。

3、如果使用https無法訪問請檢查伺服器防火牆。

## 利用nginx作為SSL的代理

除了用Workerman自身的SSL，也可以利用nginx作為SSL代理實現https。

> **注意**
> nginx代理SSL和Workerman設置SSL二選一，不能同時開啟。

通訊原理及流程為：

1、客戶端發起https連接連到nginx

2、nginx將https協議的數據轉換成http協議並轉發到Workerman的http端口

3、Workerman收到數據後做業務邏輯處理，返回http協議的數據給nginx

4、nginx再將http協議的數據轉換成https，轉發給客戶端

### nginx配置參考

**前提條件及準備工作：**

1、假設Workerman監聽的是8181端口（http協議）

2、已經申請了證書（pem/crt檔案以及key檔案）並放在了/etc/nginx/conf.d/ssl下

3、打算利用nginx開啟443端口對外提供wss代理服務（端口可以根據需要修改）

**nginx配置類似如下**：

```
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}


server {
  listen 443;
  server_name 站點域名.com;
  access_log off;
  
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /
  {
    proxy_pass http://workerman;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
```
