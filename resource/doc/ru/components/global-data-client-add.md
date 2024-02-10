# add
**``` (Требуется Workerman версии >= 3.3.0) ```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
Атомарное добавление. Если ключ уже существует, будет возвращено false.

## Параметры

``` $key ```

Ключ. (Например, если ```$global->abc```, то ```abc``` - это ключ)

``` $value ```

Хранимое значение.

## Возвращаемые значения
В случае успешного выполнения возвращает true, в противном случае возвращает false.

## Пример

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // Значение $global->some_key успешно установлено
    echo "add success " , $global->some_key;
}
else
{
    // Значение $global->some_key уже существует, установка не удалась
    echo "add fail " , var_export($global->some_key);
}
```
