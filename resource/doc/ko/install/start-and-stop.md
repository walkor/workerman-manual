# 시작 및 중지

Workerman 시작 및 중지 명령은 모두 명령 줄에서 처리됩니다.

Workerman를 시작하려면 먼저 서비스가 수신 대기할 포트 및 프로토콜이 정의된 시작 진입 파일이 필요합니다. [입문 안내서-간단한 개발 예제 부분](../getting-started/simple-example.md)을 참조할 수 있습니다.

여기에서 [workerman-chat](https://www.workerman.net/workerman-chat)을 예로 들겠습니다. 이의 시작 진입은 start.php입니다.

### 시작

디버그(debug) 모드로 시작

```php start.php start```

데몬(daemon) 모드로 시작

```php start.php start -d```

### 중지

```php start.php stop```

### 다시 시작

```php start.php restart```

### 부드러운 재시작

```php start.php reload```

### 상태 확인

```php start.php status```

### 연결 상태 확인 (Workerman 버전 >= 3.5.0이 필요)

```php start.php connections```

## 디버그와 데몬 모드의 차이

1. 디버그 모드로 시작하면 코드 내의 echo, var_dump, print 등의 출력 함수가 직접 터미널에 출력됩니다.

2. 데몬 모드로 시작하면 코드 내의 echo, var_dump, print 등의 출력이 기본적으로 /dev/null 파일로 리디렉션되며, Worker::$stdoutFile = '/your/path/file';를 설정하여 파일 경로를 지정할 수 있습니다.

3. 디버그 모드로 시작하면 터미널을 종료하면 Workerman이 종료되고 종료됩니다.

4. 데몬 모드로 시작하면 터미널을 종료해도 Workerman은 계속해서 백그라운드에서 정상적으로 실행됩니다.

## 부드러운 재시작이란?

자세한 내용은 [부드러운 재시작의 원리](../faq/reload-principle.md)를 참조하십시오.
