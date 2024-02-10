# Workerman 시작 오류

## 현상 1
시작 후 다음과 유사한 오류가 발생합니다.
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php on line xxxx
```
**키워드**: ```Address already in use```

**원인**: 포트가 사용 중이어서 시작할 수 없습니다.

#### 해결책 1
명령어 ```netstat -anp | grep 포트번호```를 사용하여 어떤 프로그램이 포트를 사용하고 있는지 확인할 수 있습니다. 그런 후 해당 프로그램을 중지하여 포트를 해제합니다.

#### 해결책 2
해당 포트를 사용하는 프로그램을 중지할 수 없는 경우, Workerman의 포트를 변경하여 문제를 해결할 수 있습니다.

#### 해결책 3
만약 Workerman이 사용 중인 포트인데도 중지할 수 없는 경우(일반적으로 pid 파일이 손실되었거나 개발자에 의해 메인 프로세스가 종료된 경우), 다음 두 명령어를 실행하여 Workerman 프로세스를 중지할 수 있습니다. 
``` 
killall php
ps aux|grep WorkerMan|awk '{print $2}'|xargs kill -9
```

#### 해결책 4
만약 실제로 프로그램이 해당 포트를 사용하지 않는다면, 이는 개발자가 Workerman에 두 개 이상의 동일한 포트를 감청하도록 설정하여 발생한 문제일 수 있습니다. 이 경우 개발자는 사용 중인 스크립트를 직접 확인하여 같은 포트를 감청하지 않도록 해야 합니다.

#### 해결책 5
reusePort를 사용하고 있는지 확인하고, 사용 중이라면 비활성화하여 시도해 보십시오.

## 현상 2
시작 후 다음과 유사한 오류가 발생합니다.
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php on line xxxx
```
또는
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (해당 주소를 할당할 수 없음) in ...workerman/Worker.php on line xxxx
```
**키워드**: `Cannot assign requested address` 또는 `해당 주소를 할당할 수 없음`

**실패 원인**: 시작 스크립트의 ip 매개 변수가 잘못되었습니다. 

**해결 방법**: 올바른 로컬 IP 주소를 입력하거나 ```0.0.0.0```(로컬 모든 IP를 감청함)로 작성하십시오.

**팁**: Linux 시스템에서는 ```ifconfig``` 명령어로 모든 네트워크 카드의 IP 주소를 확인할 수 있습니다. 클라우드 서버(예: 알리클라우드, 텐센트클라우드 등) 사용자는 공용 IP가 실제로는 프록시 IP(예: 알리클라우드의 전용 네트워크)일 수 있으므로, 공용 IP로 감청할 수 없습니다. 공용 IP로 감청할 수 없더라도, 여전히 0.0.0.0으로 바인딩할 수 있습니다.

## 현상 3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**실패 원인**: PHP.ini에서 stream_socket_server 함수가 비활성화되어 있습니다.

**해결 방법**

1. ```php --ini``` 명령어를 실행하여 PHP.ini 파일을 찾습니다.
2. PHP.ini를 열고 disable_functions 항목을 찾아 stream_socket_server를 비활성화 항목에서 삭제합니다.

## 현상 4
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```
**실패 원인**: 리눅스에서는 1024보다 작은 포트를 감청하려면 루트 권한이 필요합니다.

**해결 방법**: 1024보다 큰 포트를 사용하거나 root 사용자 권한으로 서비스를 시작하십시오.
