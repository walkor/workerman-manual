# GlobalData 변수 공유 구성 요소
**``` (Workerman 버전 >= 3.3.0이 필요합니다) ```**

소스 코드 위치: https://github.com/walkor/GlobalData

## 주의
GlobalData는 Workerman 버전 >= 3.3.0이 필요합니다.

## 설치

`composer require workerman/globaldata`

## 원리

PHP의 ```__set __get __isset __unset``` 매직 메소드를 이용하여 GlobalData 서버와 통신을 유도하여, 실제 변수는 GlobalData 서버에 저장됩니다. 예를 들어, 클라이언트 클래스에 존재하지 않는 속성을 설정할 때, ```__set``` 매직 메소드가 유도됩니다. 이때 클라이언트 클래스는 ```__set``` 메소드에서 GlobalData 서버로 요청을 보내어 변수를 저장합니다. 마찬가지로, 존재하지 않는 변수에 접근할 때는 클래스의 ```__get``` 메서드가 유도되며, 클라이언트는 GlobalData 서버로 요청을 보내어 해당 값을 읽어오게 됩니다. 이렇게 하여 프로세스 간 변수를 공유할 수 있습니다.

```php
require_once __DIR__ . '/vendor/autoload.php';

// Global Data 서버에 연결
$global = new GlobalData\Client('127.0.0.1:2207');

// $global->__isset('somedata')를 유도하여 서버에 저장된 키 'somedata'의 값이 있는지 확인합니다
isset($global->somedata);

// $global->__set('somedata',array(1,2,3))를 유도하여 서버에 'somedata' 키에 해당하는 값으로 array(1,2,3)을 저장하도록 합니다
$global->somedata = array(1,2,3);

// $global->__get('somedata')를 유도하여 서버에서 'somedata'에 해당하는 값을 조회합니다
var_export($global->somedata);

// $global->__unset('somedata')를 유도하여 서버에서 'somedata'와 해당 값이 삭제되도록 합니다
unset($global->somedata);
```
