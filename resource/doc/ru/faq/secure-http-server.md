# Создание HTTPS-сервиса

**Вопрос:**

Как создать службу HTTPS с помощью Workerman, чтобы клиенты могли устанавливать соединение через протокол HTTPS.

**Ответ:**

Протокол HTTPS на самом деле представляет собой сочетание протоколов HTTP и SSL, то есть добавление слоя SSL поверх протокола HTTP. Workerman поддерживает протокол HTTP, а также поддерживает SSL (необходимо использовать версию Workerman >= 3.3.7), поэтому для поддержки протокола HTTPS достаточно просто включить SSL поверх протокола HTTP.

Существует два распространенных способа добавления поддержки HTTPS в Workerman: прямое включение SSL в Workerman или использование Nginx в качестве прокси для SSL. Нужно выбрать один из этих методов, нельзя использовать оба сразу.

## Включение SSL в Workerman

**Подготовительные шаги:**

1. Версия Workerman >= 3.3.7
2. Установленное расширение openssl для PHP
3. Сертификат (файлы pem/crt и ключевой файл) размещены в каталоге /etc/nginx/conf.d/ssl

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Желательно использовать корректный сертификат
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // Может быть также crt файл
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Если используется самоподписанный сертификат, эту опцию нужно включить
    )
);
// Здесь устанавливается протокол HTTP
$worker = new Worker('http://0.0.0.0:443', $context);
// Устанавливаем передачу через SSL, превращая HTTP в HTTPS
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

С помощью этого кода в Workerman создается служба HTTPS, что позволяет клиентам устанавливать защищенное шифрование соединение с Workerman.

**Тестирование:**

Введите в адресной строке браузера `https://domain_name:443` для доступа.

**Примечание:**

1. Порт HTTPS должен быть доступен только через протокол HTTPS, не по протоколу HTTP.
2. Обычно сертификат связан с доменным именем, поэтому при тестировании следует использовать доменное имя, а не IP-адрес.
3. Если доступ к HTTPS не работает, проверьте файервол на сервере.

## Использование Nginx в качестве прокси для SSL

Помимо использования SSL в самом Workerman, можно также использовать Nginx в качестве прокси для SSL для реализации HTTPS.

> **Примечание**
> Нельзя использовать одновременно прокси SSL через Nginx и отдельную настройку SSL в Workerman.

Принцип и процесс коммуникации:

1. Клиент устанавливает соединение по протоколу HTTPS через Nginx
2. Nginx преобразует данные HTTPS в протокол HTTP и пересылает их на порт HTTP Workerman
3. Workerman получает данные, производит бизнес-логику и отправляет данные обратно в виде протокола HTTP Nginx
4. Nginx снова преобразует данные в протокол HTTPS и отправляет их клиенту

### Пример конфигурации Nginx

**Предположения и подготовительные шаги:**

1. Предполагается, что Workerman слушает порт 8181 (протокол HTTP)
2. Сертификат (файлы pem/crt и ключевой файл) размещены в каталоге /etc/nginx/conf.d/ssl
3. Планируется использовать порт 443 через Nginx для предоставления сервиса wss (порт можно изменить по необходимости)

**Пример конфигурации Nginx:**

```nginx
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}

server {
  listen 443;
  server_name site_domain.com;
  access_log off;
  
  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location / {
    proxy_pass http://workerman;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
```
