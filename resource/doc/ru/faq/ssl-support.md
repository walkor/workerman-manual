# Шифрование передачи данных - ssl/tls

**Вопрос:**

Как обеспечить безопасность общения между сервером и Workerman?

**Ответ:**

Наиболее удобным способом обеспечения безопасности обмена данными является добавление слоя шифрования [SSL](https://baike.baidu.com/item/ssl) над протоколом обмена, например, протоколы wss и [https](https://baike.baidu.com/item/https) основаны на шифровании с использованием [SSL](https://baike.baidu.com/item/ssl) и считаются очень безопасными. Workerman поддерживает [SSL](https://baike.baidu.com/item/ssl) (```требуется Workerman>=3.3.7```), и для его включения достаточно установить соответствующие свойства.

Конечно, разработчики также могут реализовать собственный механизм шифрования на основе определенных алгоритмов шифрования и дешифрования.

## Метод включения ssl в Workerman:

**Подготовительные шаги:**

1. Версия Workerman не ниже 3.3.7
2. PHP установлено расширение openssl
3. Были получены сертификаты (файлы pem/crt и ключевые файлы), размещенные в /etc/nginx/conf.d/ssl

**Код:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Лучше всего получить сертификат
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // Может быть также файлом crt
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Если это самоподписанный сертификат, то это нужно включить
    )
);
// Здесь устанавливается протокол websocket, также можно использовать протокол http или другие
$worker = new Worker('websocket://0.0.0.0:443', $context);
// Устанавливаем транспортное средство для включения ssl
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## Включение указания имени сервера [SNI (Server Name Indication)](https://baike.baidu.com/item/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%90%8D%E7%A7%B0%E6%8C%87%E7%A4%BA) в Workerman
Это позволяет привязывать несколько сертификатов к одному IP и порту.

**Объединение файлов .pem и .key сертификатов:**

Содержимое каждого файла .pem и соответствующего файла .key объединяется путем добавления содержимого файла .key в конец файла .pem. (Если внутри файла .pem уже содержится закрытый ключ, это можно проигнорировать).

**Обратите внимание, что речь идет о единичном сертификате, а не о копировании всех сертификатов в один файл.**

Например, содержимое файла после объединения *host1.com.pem* может выглядеть примерно так:

```text
-----BEGIN CERTIFICATE-----
MIIGXTCBA...
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIFBzCCA...
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAA....
-----END RSA PRIVATE KEY-----
```

**Код:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // Включить SNI
        'SNI_server_certs' => [ // Установить несколько сертификатов
            'host1.com' => '/path/host1.com.pem', // Сертификат 1
            'host2.com' => '/path/host2.com.pem', // Сертификат 2
        ],
        'local_cert' => '/path/default.com.pem', // Сертификат по умолчанию
        'local_pk'   => '/path/default.com.key',
    )
);
$worker = new Worker('websocket://0.0.0.0:443', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
