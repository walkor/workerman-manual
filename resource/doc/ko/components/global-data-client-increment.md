# increment
**``` (Workerman 버전 >=3.3.0이 필요합니다) ```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```
원자적으로 증가시킵니다. 지정된 step 매개변수 만큼 숫자 요소를 증가시킵니다. 요소의 값이 숫자 유형이 아닌 경우 0으로 간주하여 증가시킵니다. 요소가 존재하지 않으면 false를 반환합니다.

## 매개변수

 ``` $key ```

키 값입니다. (예: ```$global->abc```, ```abc```가 키 값입니다.)

 ``` $value ```

요소의 값을 증가시킬 크기입니다.

## 반환값
성공하면 true를, 그렇지 않으면 false를 반환합니다.

## 예시

```php
$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// 비원자적으로 증가
$global->some_key++;

echo $global->some_key."\n";

// 원자적으로 증가
$global->increment('some_key');

echo $global->some_key."\n";
```
