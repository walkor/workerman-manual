# 創建wss服務

**問：**

如何在Workerman中創建wss服務，以便客戶端可以使用wss協定來進行通信，例如在微信小程序中連接服務端的情況。

**答：**

wss協定實際上是[websocket](https://baike.baidu.com/item/WebSocket)+[SSL](https://baike.baidu.com/item/ssl)，即在websocket協定上添加了[SSL](https://baike.baidu.com/item/ssl)層，類似於[https](https://baike.baidu.com/item/https)（[http](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl)）。
因此，只需在websocket協定的基礎上啟用[SSL](https://baike.baidu.com/item/ssl)即可支持wss協定。

## 方法一、使用nginx/apache代理SSL（推薦）

**通訊原理及流程**

1. 客戶端發起wss連接連到nginx/apache
2. nginx/apache將wss協定的數據轉換成ws協定數據並轉發到Workerman的websocket協定端口
3. Workerman收到數據後進行業務邏輯處理
4. 當Workerman給客戶端發送消息時，則是相反的過程，數據經過nginx/apache轉換成wss協定然後發送給客戶端

## nginx配置參考

**前提條件及準備工作：**

1. 已經安裝nginx，版本不低於1.3
2. 假設Workerman監聽的是8282端口（websocket協定）
3. 已申請了證書（pem/crt文件和key文件），假設放在/etc/nginx/conf.d/ssl下
4. 打算使用nginx在443端口上提供wss代理服務（端口可根據需要修改）
5. nginx通常作為網站伺服器運行其他服務，為了不影響原來的站點使用，請使用地址`域名.com/wss`作為wss的代理入口。也就是客戶端連接地址為wss://域名.com/wss

**nginx設置如下：**
```nginx
server {
  listen 443;
  # 域名配置省略...

  ssl on;
  ssl_certificate /etc/ssl/server.pem;
  ssl_certificate_key /etc/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /wss
  {
    proxy_pass http://127.0.0.1:8282;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # location / {} 站點的其他配置...
}
```

**測試**
```javascript
// 證書是會檢查域名的，請使用域名連接。注意這裡不寫端口
ws = new WebSocket("wss://域名.com/wss");

ws.onopen = function() {
    alert("連接成功");
    ws.send('tom');
    alert("給服務端發送一個字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服務端的消息：" + e.data);
};
```

## 使用apache代理wss

也可以使用apache作為wss代理轉發至workerman。

準備工作：

1. GatewayWorker監聽8282端口（websocket協定）
2. 已申請了SSL證書，假設放在/server/httpd/cert/下
3. 使用apache將443端口轉發到指定端口8282
4. 已加載httpd-ssl.conf
5. 已安裝openssl

**啟用proxy_wstunnel_module模塊**
```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**配置SSL及代理**
```apache
#extra/httpd-ssl.conf
DocumentRoot "/網站/目錄"
ServerName 域名

# Proxy Config
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# 添加SSL協定支持協定,去掉不安全的協定
SSLProtocol all -SSLv2 -SSLv3
# 修改加密套件如下
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# 证书公钥配置
SSLCertificateFile /server/httpd/cert/your.pem
# 证书私钥配置
SSLCertificateKeyFile /server/httpd/cert/your.key
# 证书链配置,
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**測試**
```javascript
// 證書是會檢查域名的，請使用域名連接。注意沒有端口
ws = new WebSocket("wss://域名.com/wss");

ws.onopen = function() {
    alert("連接成功");
    ws.send('tom');
    alert("給服務端發送一個字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服務端的消息：" + e.data);
};
```

## 方法二，直接使用Workerman啟用SSL（不推薦）

> **注意**
> nginx/apache代理SSL和Workerman設置SSL二選一，不能同時啟用。

**準備工作：**

1. Workerman版本>=3.3.7
2. PHP安裝了openssl擴展
3. 已申請了證書（pem/crt文件和key文件）放在磁盤任意目錄

**代碼：**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 证书最好是申请的证书
$context = array(
    // 更多ssl选项请参考手册 http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // 请使用绝对路径
        'local_cert'        => '磁盘路径/server.pem', // 也可以是crt文件
        'local_pk'          => '磁盘路径/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, //如果是自签名证书需要开启此选项
    )
);
// 这里设置的是websocket协议（端口任意，但是需要保证没被其它程序占用）
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// 设置transport开启ssl，websocket+ssl即wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

通過以上的代碼，Workerman就監聽了wss協定，客戶端就可以通過wss協定來連接workerman實現安全即時通訊了。

**測試**

打開chrome瀏覽器，按F12打開調試控制台，在Console一欄輸入（或者將下面代碼放入到html頁面用js運行）

```javascript
// 证书是会检查域名的，请使用域名连接，注意这里有端口号
ws = new WebSocket("wss://域名.com:8282");
ws.onopen = function() {
    alert("連接成功");
    ws.send('tom');
    alert("給服務端發送一個字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服務端的消息：" + e.data);
};
```

**注意：**

1. 如果必須使用443端口請使用上面第一種方案nginx/apache代理方式實現wss。
2. wss端口只能通過wss協定訪問，ws無法訪問wss端口。
3. 證書一般是與域名綁定的，因此測試時客戶端請使用域名連接，不要使用IP去連。
4. 如果出現無法訪問的情況，請檢查伺服器防火牆。
5. 此方法要求PHP版本>=5.6，因為微信小程序要求tls1.2，而PHP5.6以下版本不支持tls1.2。
