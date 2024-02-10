# cas
**``` (Workerman 버전 >= 3.3.0 이상 필요) ```**
```php
bool \GlobalData\Client::cas(string $key, mixed $old_value, mixed $new_value)
```
원자적으로 대체하여 ```$new_value```로 ```$old_value```를 대체합니다.
현재 클라이언트가 특정 키 값의 값을 다른 클라이언트에 의해 변경되지 않은 경우에만 값을 쓸 수 있습니다.

## 매개변수

 ``` $key ```

키 값입니다. (예를 들어, ```$global->abc```에서 ```abc```가 키 값입니다.)

 ``` $old_value ```

이전 데이터


 ``` $new_value ```

새로운 데이터

## 반환 값
대체가 성공하면 true를, 그렇지 않으면 false를 반환합니다.

## 설명:

여러 프로세스가 동시에 동일한 공유 변수를 조작할 때 종종 동시성 문제를 고려해야 합니다.

예를 들어 A와 B 두 프로세스가 동시에 사용자 목록에 멤버를 추가합니다.
현재 A와 B 프로세스의 사용자 목록이 ```$global->user_list = array(1,2,3)```입니다.
A 프로세스는 ```$global->user_list``` 변수를 조작하여 사용자 4를 추가합니다.
B 프로세스는 ```$global->user_list``` 변수를 조작하여 사용자 5를 추가합니다.
A 프로세스는 변수를 설정하여 ```$global->user_list = array(1,2,3,4)``` 성공합니다.
B 프로세스는 변수를 설정하여 ```$global->user_list = array(1,2,3,5)``` 성공합니다.
이때 B 프로세스가 A 프로세스가 설정한 변수를 덮어쓰면서 데이터가 손실됩니다.

**위와 같이 읽고 설정이 원자적인 작업이 아니라 동시성 문제가 발생합니다.**
이러한 동시성 문제를 해결하기 위해 cas(Compare and Swap) 원자적 대체 인터페이스를 사용할 수 있습니다.
cas 인터페이스는 값을 변경하기 전에 ```$old_value```를 기반으로 이 값이 다른 프로세스에 의해 변경되었는지 확인하고, 변경되었으면 교체하지 않고 false를 반환합니다. 그렇지 않으면 true를 반환합니다.

 **참고:** 
일부 공유 데이터가 동시에 덮어쓰여도 문제가 되지 않습니다. 예를 들어 경매 시스템의 현재 최고 입찰가, 상품의 현재 재고 등이 해당됩니다.

## 예시

```php
$global = new GlobalData\Client('127.0.0.1:2207');

// 리스트 초기화
$global->user_list = array(1,2,3);

// user_list에 값을 원자적으로 추가
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```
