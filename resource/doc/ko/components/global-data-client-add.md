# 추가
**``` (Workerman 버전 >= 3.3.0 이상이 필요합니다) ```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
원자적으로 추가합니다. 키가 이미 존재하는 경우에는 false를 반환합니다.

## 매개변수

 ``` $key ```

키 값입니다. (예: ```$global->abc```에서 ```abc```는 키 값입니다)

 ``` $value ```

저장할 값입니다.

## 반환 값
성공하면 true를, 그렇지 않으면 false를 반환합니다.

## 예시

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // $global->some_key에 값이 성공적으로 할당되었습니다.
    echo "add success " , $global->some_key;
}
else
{
    // $global->some_key가 이미 존재하여 값이 할당되지 않았습니다.
    echo "add fail " , var_export($global->some_key);
}
```
