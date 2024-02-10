# Создание wss-сервиса

**Вопрос:**

Как создать wss-сервис в Workerman, чтобы клиенты могли устанавливать связь через протокол wss, например, подключаться к серверу через маленькое приложение WeChat?

**Ответ:**

Протокол wss фактически представляет собой [websocket](https://baike.baidu.com/item/WebSocket)+[SSL](https://baike.baidu.com/item/ssl), то есть добавление слоя [SSL](https://baike.baidu.com/item/ssl) поверх протокола websocket, подобно [https](https://baike.baidu.com/item/https) ([http](https://baike.baidu.com/item/http)+[SSL](https://baike.baidu.com/item/ssl)).
Поэтому для поддержки протокола wss достаточно открыть [SSL](https://baike.baidu.com/item/ssl) поверх протокола [websocket](https://baike.baidu.com/item/WebSocket).

## Метод 1: Использование прокси nginx/apache для SSL (рекомендуется)

**Принцип и процесс связи:**

1. Клиент устанавливает wss-соединение через nginx/apache.
2. Nginx/apache преобразует данные для протокола wss в данные протокола ws и пересылает их на порт протокола websocket в Workerman.
3. Workerman получает данные и обрабатывает их в соответствии с бизнес-логикой.
4. При отправке сообщения клиенту Workerman делает обратное преобразование данных в протокол wss через nginx/apache и передает их клиенту.

## Настройка nginx

**Необходимые условия и подготовительные работы:**

1. Установленный nginx версии не ниже 1.3.
2. Предполагается, что Workerman слушает порт 8282 (протокол websocket).
3. Сертификат успешно запрошен (файлы pem/crt и ключевой файл) и предполагается, что они находятся в /etc/nginx/conf.d/ssl.
4. Предполагается использование nginx для предоставления wss-прокси-сервиса на порту 443.
5. Обычно nginx работает как веб-сервер для других служб. Для того, чтобы не повлиять на работу существующих сайтов, здесь используется адрес "домен.com/wss" для входа в wss-прокси. То есть адрес подключения клиента будет wss://домен.com/wss.

**Пример конфигурации nginx:**
```nginx
server {
  listen 443;
  # Настройки домена опущены...

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
  
  # Дополнительная конфигурация для другого сайта...
}
```
**Тестирование:**
```javascript
// Сертификат проверяет домен, поэтому используйте соединение по домену. Обратите внимание, что здесь не указан порт
ws = new WebSocket("wss://домен.com/wss");

ws.onopen = function() {
    alert("Соединение установлено");
    ws.send('tom');
    alert("Отправлено на сервер строку: tom");
};
ws.onmessage = function(e) {
    alert("Получено сообщение от сервера: " + e.data);
};
```

## Использование apache для проксирования wss

Также можно использовать apache в качестве wss-прокси для передачи данных в Workerman.

Подготовительная работа:

1.  GatewayWorker слушает порт 8282 (протокол websocket).
2. Уже запросился SSL-сертификат и предполагается, что он находится в /server/httpd/cert/.
3. Apache перенаправляет порт 443 на указанный порт 8282.
4. Файл httpd-ssl.conf успешно загружен.
5. OpenSSL установлен.

**Включение модуля proxy_wstunnel**

```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**Настройка SSL и прокси**
```apache
#extra/httpd-ssl.conf
DocumentRoot "/корневая/директория/сайта"
ServerName домен

# Proxy Config
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# Добавление поддержки протокола SSL, удаление не безопасных протоколов
SSLProtocol all -SSLv2 -SSLv3
# Изменение шифрования:
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# Конфигурация публичного ключа сертификата
SSLCertificateFile /server/httpd/cert/your.pem
# Конфигурация приватного ключа сертификата
SSLCertificateKeyFile /server/httpd/cert/your.key
# Конфигурация цепочки сертификатов
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**Тестирование:**
```javascript
// Сертификат проверяет домен, поэтому используйте соединение по домену. Обратите внимание, что здесь не указан порт
ws = new WebSocket("wss://домен.com/wss");

ws.onopen = function() {
    alert("Соединение установлено");
    ws.send('tom');
    alert("Отправлено на сервер строку: tom");
};
ws.onmessage = function(e) {
    alert("Получено сообщение от сервера: " + e.data);
};
```


## Метод 2: Непосредственное использование SSL в Workerman (не рекомендуется)

> **Примечание:**
> Возможность настройки SSL в nginx/apache и в Workerman представляют альтернативы и не могут использоваться одновременно.

**Подготовительная работа:**

1. Версия Workerman >=3.3.7.
2. Расширение PHP openssl установлено.
3. SSL-сертификат (файлы pem/crt и ключевой файл) были успешно запрошены и находятся в любой директории на диске.

**Пример кода:**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Лучше использовать запрошенный сертификат
$context = array(
    // Более много различных ssl-настроек см. в документации на http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // Используйте абсолютные пути
        'local_cert'        => 'путь_к_файлу/server.pem', // Также может быть файлом crt
        'local_pk'          => 'путь_к_файлу/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Необходимо для самоподписанных сертификатов
    )
);
// Здесь устанавливается протокол websocket (порт произвольный, но его нужно поднять так, чтобы другие программы не занимали его)
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// Устанавливаем транспорт для ssl, websocket + ssl = wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

С использованием этого кода Workerman будет прослушивать протокол wss, и клиенты смогут устанавливать безопасное соединение в реальном времени через протокол wss.

**Тестирование**

Откройте браузер Chrome, нажмите F12, чтобы открыть консоль отладки. В разделе "Консоль" введите (или разместите нижеприведенный код в HTML-странице и запустите его через JavaScript):
```javascript
// Сертификат проверяет домен, поэтому используйте соединение по домену. Обратите внимание, что здесь указан порт
ws = new WebSocket("wss://домен.com:8282");
ws.onopen = function() {
    alert("Соединение установлено");
    ws.send('tom');
    alert("Отправлено на сервер строку: tom");
};
ws.onmessage = function(e) {
    alert("Получено сообщение от сервера: " + e.data);
};
```

**Примечания:**

1. Если необходимо использовать порт 443, используйте рекомендуемый первый способ с использованием прокси nginx/apache для внедрения wss.
2. Порт wss доступен только через протокол wss и не может быть доступен через протокол ws.
3. Сертификат обычно привязан к домену, поэтому для тестирования клиенты должны использовать подключение через домен, а не IP-адрес.
4. Если возникают проблемы с доступом, проверьте брандмауэр сервера.
5. Данный метод требует версии PHP >=5.6, поскольку маленькое приложение WeChat требует поддержки tls1.2, а версии PHP ниже 5.6 не поддерживают tls1.2.

Связанные статьи:  
[Получение реального IP-адреса через прокси](get-real-ip-from-proxy.md)  
[Справочник параметров SSL-контекста в Workerman](https://php.net/manual/zh/context.ssl.php)
