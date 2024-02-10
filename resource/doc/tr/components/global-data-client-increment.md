# increment
**``` (Workerman sürümü >=3.3.0 gereklidir) ```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```
Atomik olarak artırır. Belirtilen adımda bir sayısal öğeyi artırır. Öğenin değeri sayısal türde değilse, önce 0 olarak kabul edilir ve artış işlemi yapılır. Öğe mevcut değilse false döner.

## Parametreler

 ``` $key ```

Anahtar değeri. (Örneğin, ```$global->abc``` için ```abc``` anahtar değeridir)

 
 ``` $value ```

Öğenin değerini artırmak istenen miktar.

## Dönüş Değeri
Başarılıysa true, aksi halde false döner.


## Örnekler

```php
$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// Atomik olmayan artırma
$global->some_key++;

echo $global->some_key."\n";

// Atomik artırma
$global->increment('some_key');

echo $global->some_key."\n";
```
