# Thêm
**``` (Yêu cầu Workerman phiên bản >=3.3.0) ```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
Thêm nguyên tử. Nếu khóa đã tồn tại, sẽ trả về false.

## Tham số

 ``` $key ```

Khóa. (Ví dụ: ```$global->abc```, ```abc``` là khóa)


 ``` $value ```

Giá trị lưu trữ.

## Giá trị trả về
Trả về true nếu thành công, ngược lại trả về false.


## Ví dụ

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // Gán giá trị cho $global->some_key thành công
    echo "Thêm thành công " , $global->some_key;
}
else
{
    // $global->some_key đã tồn tại, gán giá trị thất bại
    echo "Thất bại " , var_export($global->some_key);
}
```
