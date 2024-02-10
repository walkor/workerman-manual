# 사용자

## 설명:
```php
string Worker::$user
```

현재 Worker 인스턴스가 실행될 사용자를 설정합니다.이 속성은 현재 사용자가 root 인 경우에만 적용됩니다. 설정되지 않은 경우 기본적으로 현재 사용자로 실행됩니다.

```$user```를 낮은 권한을 가진 사용자로 설정하는 것이 좋습니다. 예를 들어 www-data, apache, nobody 등입니다.

참고: 이 속성은 ```Worker::runAll();```가 실행되기 전에 설정해야만 합니다. Windows 시스템은 이 기능을 지원하지 않습니다.

## 예시

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 인스턴스의 실행 사용자 설정
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// worker 실행
Worker::runAll();
```
