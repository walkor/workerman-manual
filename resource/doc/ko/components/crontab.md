# workerman/crontab

# 설명
`workerman/crontab`는 workerman을 기반으로 하는 시간 기반 작업 프로그램으로, Linux의 crontab과 유사합니다. `workerman/crontab`는 초 단위의 타이머를 지원합니다.

>`workerman/crontab`를 사용하려면 먼저 PHP의 시간대를 설정해야 합니다. 그렇지 않으면 실행 결과가 예상과 다를 수 있습니다.

## 시간 설명
```plaintext
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ 요일 (0 - 6) (일요일=0)
|   |   |   |   +------ 월 (1 - 12)
|   |   |   +-------- 월 별 일 (1 - 31)
|   |   +---------- 시간 (0 - 23)
|   +------------ 분 (0 - 59)
+-------------- 초 (0-59)[생략 가능, 0 자리가 없으면 최소 시간 간격은 분 단위임]
```

# 설치
```plaintext
composer require workerman/crontab
```

# 예시
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// 실행 결과와 일치하지 않도록 하기 위해 시간대 설정
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // 매 분 1초에 실행
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // 매일 7시 50분에 실행, 초 자리는 생략했음에 유의
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> 참고: 예약된 작업은 즉시 실행되지 않으며 모든 예약된 작업은 다음 분에 실행됩니다.

# 인터페이스
**Crontab::destroy()**

타이머 파괴
```php
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```
