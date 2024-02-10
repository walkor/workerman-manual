```php
# ekle
**``` (Workerman sürümü >=3.3.0 olmalıdır) ```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
Atomik ekleme. Eğer anahtar zaten varsa false döner.

## Parametreler

 ``` $key ```

Anahtar değeri. (Örneğin ```$global->abc```, ```abc``` anahtar değeridir)


 ``` $value ```

Depolanacak değer.

## Dönüş Değeri
Başarıyla eklenirse true, aksi halde false döner.


## Örnek

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // $global->some_key değeri başarıyla eklendi
    echo "ekleme başarılı " , $global->some_key;
}
else
{
    // $global->some_key zaten var, ekleme başarısız
    echo "ekleme başarısız " , var_export($global->some_key);
}
```
