# 실패한 중지

## 현상:
```php start.php stop```을 실행하면 ```stop fail```이라고 표시됩니다.

### 첫 번째 가능성
Workerman을 debug 모드로 시작했고, 개발자가 터미널에서 ```ctrl z```를 눌러 ```SIGSTOP``` 신호를 보낸 경우, Workerman이 백그라운드로 들어가고 일시 중지되어 있기 때문에 중지 명령(```SIGINT``` 신호)에 응답할 수 없습니다.

**해결책:**
Workerman을 시작한 터미널에서 ```fg```(```SIGCONT``` 신호를 보냄)를 입력하고 Enter를 눌러서 Workerman을 다시 전면으로 가져와서 ```ctrl c```(```SIGINT``` 신호를 보냄)를 눌러 Workerman을 중지합니다.

만약 이렇게 중지되지 않으면 다음 두 명령어를 시도해보십시오.
``` 
killall -9 php
```
```
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```
### 두 번째 가능성
중지를 시도하는 사용자와 Workerman을 시작한 사용자가 다르므로, 중지 사용자에게 Workerman을 중지하는 권한이 없습니다.

**해결책:**
Workerman을 시작한 사용자로 전환하거나 더 높은 권한을 가진 사용자로 Workerman을 중지합니다.


### 세 번째 가능성
Workerman의 주 프로세스 pid 파일이 삭제되어 스크립트가 pid 프로세스를 찾을 수 없어 중지에 실패합니다.

**해결:**
pid 파일을 안전한 위치에 저장하고, 매뉴얼 [Worker::$pidFile](../worker/pid-file.md)을 참조하십시오.


### 네 번째 가능성
Workerman의 주 프로세스 pid 파일에 해당하는 프로세스가 Workerman 프로세스가 아닙니다.

**해결:**
Workerman의 주 프로세스의 pid 파일을 열어 주 프로세스의 pid를 확인하고, pid 파일은 기본적으로 Workerman과 동일한 디렉터리에 저장됩니다. 명령어 ```ps aux | grep 주 프로세스 pid```를 실행하여 해당 프로세스가 Workerman 프로세스인지 확인하세요. 그렇지 않은 경우, 서버가 다시 부팅되어 Workerman이 저장한 pid가 만료된 pid일 수 있고, 이 pid가 다른 프로세스에 의해 사용되어 중지에 실패하는 경우가 있습니다. 이 경우에는 pid 파일을 삭제하시면 됩니다.

### 다섯 번째 가능성
grpc 확장 기능을 설치했지만 해당 확장 기능에 대한 환경 변수를 설정하지 않은 경우에는 시작시 부모 fork 프로세스가 추가로 생성되어 중지가 실패할 수 있습니다.
