# 프로세스를 몇 개 열어야 합니까?

## 프로세스 수를 설정하는 방법
프로세스 수는 ```count``` 속성에 의해 결정됩니다(windows 시스템은 프로세스 수 설정을 지원하지 않습니다), 다음과 같은 코드를 예로 들 수 있습니다.
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// 4개의 프로세스를 시작하여 서비스를 제공
$http_worker->count = 4;

...
```

## 프로세스 수 설정 시 고려해야 할 사항
1. CPU 코어 수
2. 메모리 용량
3. IO 중심 비즈니스 또는 CPU 중심 비즈니스

## 프로세스 수 설정 원칙
1. 각 프로세스가 차지하는 메모리 합계는 총 메모리보다 작아야 합니다(보통 각 비즈니스 프로세스가 약 40M 정도의 메모리를 차지합니다).
2. IO 중심적인 경우, 즉 비즈니스에 **블로킹형** IO가 관련된 경우, 일반적으로 Mysql, Redis 등 저장소에 대한 접근은 블로킹형 접근입니다. 프로세스 수를 늘릴 수 있으며, CPU 코어 수의 3배로 구성할 수 있습니다. 비즈니스에서 대기하는 블록이 많을 경우, 프로세스 수를 적절히 늘릴 수 있으며, 예를 들어 CPU 코어 수의 8배 이상을 구성할 수 있습니다. 또한 **비블로킹형** IO는 CPU 중심적이며 IO 중심적이 아닙니다.
3. CPU 중심적인 경우, 즉 비즈니스에서 **블로킹형** IO 비용이 없는 경우, 비동기 IO를 사용하여 네트워크 리소스를 읽는 경우, 프로세스는 비즈니스 코드에 의해 차단되지 않습니다. 이 경우 프로세스 수를 CPU 코어 수와 동일하게 설정할 수 있습니다.

## 프로세스 수 설정 참고 값
비즈니스 코드가 IO 중심적인 경우, IO 중심성에 따라 프로세스 수를 설정할 수 있습니다. 예를 들어 CPU 코어 수의 3-8배입니다.

비즈니스 코드가 CPU 중심적인 경우, 프로세스 수를 CPU 코어 수로 설정할 수 있습니다.

## 주의 사항
Workerman의 IO는 모두 블로킹되지 않으며, 예를 들어 ```Connection->send```는 모두 블로킹되지 않고 CPU 중심적인 작업에 해당됩니다. 본인이 어떤 유형에 비즈니스에 속하는지 알 수 없는 경우, 프로세스 수를 CPU 코어 수의 약 3배로 설정하면 됩니다. 또한, 프로세스 수가 많다고 해서 항상 좋은 것은 아니며, 프로세스가 많을수록 프로세스 전환이 늘어나며 성능에 일정한 영향을 미칠 수 있습니다.
