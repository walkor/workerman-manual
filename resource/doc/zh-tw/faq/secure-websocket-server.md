# 建立wss服務

**問:**

如何使用Workerman创建一个wss服务，以便客户端可以通过wss协议连接进行通讯，比如在微信小程序中连接到服务端。

**答:**

wss协议实际上是[websocket](https://baike.baidu.com/item/WebSocket)+[SSL](https://baike.baidu.com/item/ssl)，即在websocket协议上加入[SSL](https://baike.baidu.com/item/ssl)层，类似于[https](https://baike.baidu.com/item/https)([http](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl))。
因此，只需要在[websocket](https://baike.baidu.com/item/WebSocket)协议的基础上启用[SSL](https://baike.baidu.com/item/ssl)即可支持wss协议。

## 方法一、使用nginx/apache代理SSL(推薦)

**通訊原理與過程**

1. 客戶端發起wss連接到nginx/apache。

2. nginx/apache將wss協議的數據轉換成ws協議數據，然後轉發到Workerman的websocket協議端口。

3. Workerman收到數據後進行業務邏輯處理。

4. 當Workerman向客戶端發送消息時，則是相反的過程，數據經過nginx/apache轉換成wss協議，然後發送給客戶端。

## nginx配置參考
**前提條件與準備工作：**

1. 已安裝nginx，版本不低於1.3。

2. 假設Workerman監聽的是8282端口（websocket協議）。

3. 已經申請了證書（pem/crt文件和key文件），假設放在/etc/nginx/conf.d/ssl下。

4. 打算使用nginx開啟443端口對外提供wss代理服務（根據需要更改端口）。

5. 通常nginx作爲網站伺服器運行，為了不影響原有站點使用，這裡使用地址 ```域名.com/wss``` 作爲wss的代理入口。也就是客戶端連接地址爲 wss://域名.com/wss。

**nginx配置類似如下**：
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
// 证书是会检查域名的，请使用域名连接。注意这里不写端口
ws = new WebSocket("wss://域名.com/wss");

ws.onopen = function() {
    alert("连接成功");
    ws.send('tom');
    alert("给服务端发送一个字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```

## 使用apache代理wss

也可以使用apache作為wss代理轉發給workerman。

準備工作：

1. GatewayWorker 監聽 8282 端口（websocket協議）。

2. 已經申請了ssl證書，假設放在了/server/httpd/cert/ 下。

3. 使用apache轉發443端口至指定端口8282。

4. 已加載httpd-ssl.conf。

5. 已安裝openssl。

**啟用 proxy_wstunnel_module 模塊**
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

# 添加 SSL 協議支持協議,去掉不安全的協議
SSLProtocol all -SSLv2 -SSLv3
# 修改加密套件如下
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# 證書公鑰配置
SSLCertificateFile /server/httpd/cert/your.pem
# 證書私鑰配置
SSLCertificateKeyFile /server/httpd/cert/your.key
# 證書鏈配置,
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**測試**
```javascript
// 证书是会检查域名的，请使用域名连接。注意没有端口
ws = new WebSocket("wss://域名.com/wss");

ws.onopen = function() {
    alert("连接成功");
    ws.send('tom');
    alert("给服务端发送一个字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```


## 方法二，直接使用Workerman開啟SSL(不推薦)

> **注意**
> nginx/apache代理SSL和Workerman設置SSL二選一，不能同時啟用。

**準備工作：**

1. Workerman版本>=3.3.7

2. PHP安裝了openssl擴展

3. 已經申請了證書（pem/crt文件和key文件），放在任意目錄

**代碼：**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 證書最好是申請的證書
$context = array(
    // 更多ssl選項請參考手冊 http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // 請使用絕對路徑
        'local_cert'        => '磁盤路徑/server.pem', // 也可以是crt文件
        'local_pk'          => '磁盤路徑/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, //如果是自簽名證書需要開啟此選項
    )
);
// 這裡設置的是websocket協議（端口任意，但是需要確保沒被其他程序占用）
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// 設置transport開啟ssl，websocket+ssl即wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

通過以上代碼，Workerman就監聽了wss協議，客戶端就可以通過wss協議來連接Workerman實現安全即時通訊。

**測試**

打開chrome瀏覽器，按F12打開調試控制台，在Console一欄輸入 (或者將下面代碼放入到html頁面用js運行)

```javascript
// 证书是会检查域名的，请使用域名连接，注意这里有端口号
ws = new WebSocket("wss://域名.com:8282");
ws.onopen = function() {
    alert("连接成功");
    ws.send('tom');
    alert("给服务端发送一个字符串：tom");
};
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```

**注意：**

1. 如果必須使用443端口請使用上面第一種方案nginx/apache代理方式實現wss。

2. wss端口只能通過wss協議訪問，ws無法訪問wss端口。

3. 證書通常與域名綁定，因此在測試時請客戶端使用域名連接，不要使用IP連接。

4. 如果無法訪問，請檢查服務器防火牆。

5. 此方法要求PHP版本>=5.6，因微信小程序要求tls1.2，而PHP5.6以下版本不支持tls1.2。

相關文章：  
[透過代理獲取客戶端真實ip](get-real-ip-from-proxy.md)  
[workerman的ssl上下文選項參考](https://php.net/manual/zh/context.ssl.php)
