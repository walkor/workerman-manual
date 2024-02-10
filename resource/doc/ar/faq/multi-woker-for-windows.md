في نظام التشغيل Windows ، لا يمكنك تهيئة عدة Workers في ملف PHP واحد.

على سبيل المثال، في ملف test.php التالي:

```php
<?php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
...
```

عند تشغيله في ويندوز ستحصل على الخطأ التالي:

```
multi workers init in one php file are not support
```

## الحل
الحل هو استخدام عدة نصوص تشغيل منفصلة، حيث يتم تشغيل نسخة منّقاة من Worker في كل نص.

على سبيل المثال، لنفترض أنه تم تهيئة Workerين (tcp و websocket) و WebServer واحد، فيجب إنشاء ثلاثة ملفات تشغيل: start_socket_server.php و start_websocket_server.php و start_webserver.php.

على سبيل المثال:

ملف start_socket_server.php:

```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

ملف start_websocket_server.php:

```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
 ...
```

ملف start_webserver.php:

```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
 ...
```

ثم يمكنك بدء تشغيل هذه الملفات مباشرة باستخدام الأوامر التالية في سطر الأوامر في ويندوز:

```bash
php start_socket_server.php
php start_websocket_server.php
php start_webserver.php
```
