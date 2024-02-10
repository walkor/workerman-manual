## 리눅스 시스템에서 workerman을 부팅시 자동으로 시작하는 방법

/etc/rc.local 파일을 열고 ```exit 0``` 이전에 다음과 유사한 코드를 추가합니다.

```
ulimit -HSn 102400
/usr/bin/env php /디스크/경로/start.php start -d

exit 0
```
