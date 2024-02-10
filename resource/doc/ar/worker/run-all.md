# runAll
```php
void Worker::runAll(void)
```
تشغيل جميع مثيلات الـ Worker.

**ملاحظة:**
بمجرد تنفيذ Worker::runAll()، سيتم منع التنفيذ بشكل دائم، وهذا يعني أن الشيفرة التي تأتي بعد Worker::runAll() لن تتم تنفيذها. يجب على جميع مثيلات الـ Worker أن تكون قد تم إنشاؤها قبل Worker::runAll().

### البارامترات
لا يوجد

### العودة
لا شيء

## Exemple - تشغيل عدة مثيلات من الـ Worker

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// تشغيل جميع مثيلات الـ Worker
Worker::runAll();
```

**ملاحظة:**
لا يدعم إصدار workerman لنظام Windows إنشاء عدة مثيلات من الـ Worker في نفس الملف.
لا يمكن تشغيل المثال أعلاه في إصدار workerman لنظام Windows.

يجب وضع إنشاء مثيلات الـ Worker في ملفات منفصلة في إصدار workerman لنظام Windows، مثل المثال التالي:

start_http.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

// تشغيل جميع مثيلات الـ Worker (هنا يوجد مثيل واحد فقط)
Worker::runAll();
```

start_websocket.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// تشغيل جميع مثيلات الـ Worker (هنا يوجد مثيل واحد فقط)
Worker::runAll();
```
