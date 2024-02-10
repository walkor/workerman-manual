Windows 운영 체제에서는 하나의 PHP 파일에서 여러 Worker를 초기화할 수 없습니다.

예를 들어 다음 test.php가 있습니다.

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
위와 같이 시작하면 Windows에서 실행하면 다음과 같은 오류가 발생합니다.

```
하나의 php 파일에서 여러 Worker를 초기화하는 것은 지원되지 않습니다
```

## 해결 방법
해결책은 여러 시작 스크립트를 사용하여 각각의 Worker를 초기화하는 것이거나 각 포트에 대해 시작 파일을 만드는 것입니다.

예를 들면, 두 개의 Worker 인스턴스(tcp 및 websocket)와 하나의 WebServer 인스턴스를 초기화한다면, start\_socket\_server.php와 start\_websocket\_server.php start\_webserver.php 세 개의 시작 파일을 생성해야 합니다.

예를 들어:

파일 start\_socket\_server.php

```php
...
$socket_server = new Worker("tcp://0.0.0.0:5555");
$socket_server->on....
....
```

파일 start\_websocket\_server.php

```php
...
$websocket_server = new Worker("websocket://0.0.0.0:6666");
$websocket_server->on....
 ...
```

파일 start\_webserver.php

```php
...
$webserver = new WebServer("http://0.0.0.0:6666");
$webserver->addRoot(...
 ...
```

시작할 때 다음과 같이 직접 세 개의 스크립트를 실행할 수 있습니다([Windows cmd 명령행](https://baike.baidu.com/view/756438.htm)에서 실행).

```
   php start_socket_server.php start_websocket_server.php start_webserver.php
```
