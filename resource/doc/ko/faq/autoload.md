# 자동로딩

프로젝트에서 자동로딩이 구현되어 있지 않다면 composer를 사용하여 자동로딩을 생성할 수 있습니다. 다음은 그 방법입니다.

**1. composer.json 수정**  
composer.json 파일에서 autoload.psr-4에 `"" : "./"`를 추가합니다. 예를 들어,
```json
{
    "require": {
        "workerman/workerman" : "^v4.0.0"
    },
    "autoload": {
        "psr-4": {
            "" : "./"
        }
    }
}
```

**2. autoload.php 생성**  
`composer dumpautoload`를 실행합니다.

**3. autoload.php 로드**  
프로젝트의 start.php 파일 상단에 `vendor/autoload.php`를 로드합니다. 예를 들어,
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. workerman 재시작**  
`php start.php restart`

