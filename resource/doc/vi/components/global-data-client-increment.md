# tăng
**``` (Yêu cầu phiên bản Workerman >= 3.3.0) ```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```
Tăng số nguyên tử. Tăng giá trị của phần tử số nguyên theo kích thước được chỉ định bởi tham số step. Nếu giá trị của phần tử không phải là kiểu số, thì nó sẽ được coi là 0 và sau đó tăng lên. Nếu phần tử không tồn tại, trả về false.

## Tham số

 ``` $key ```

Giá trị khóa. (Ví dụ: ```$global->abc```, thì ```abc``` chính là giá trị khóa)


 ``` $value ```

Kích thước tăng giá trị của phần tử.

## Giá trị trả về
Nếu thành công, trả về true, ngược lại trả về false.


## Ví dụ

```php
$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// Không phải tăng số nguyên tử
$global->some_key++;

echo $global->some_key."\n";

// Tăng số nguyên tử
$global->increment('some_key');

echo $global->some_key."\n";
```
