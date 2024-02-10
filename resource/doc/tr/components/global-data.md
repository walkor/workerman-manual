# GlobalData değişken paylaşım bileşeni
**```(Workerman sürümü >=3.3.0 gerektirir)```**

Kaynak kod adresi: https://github.com/walkor/GlobalData

## Uyarı
GlobalData'nın Workerman sürümü >=3.3.0 gerektirir

## Yükleme

`composer require workerman/globaldata`

## Prensip

PHP'nin ```__set __get __isset __unset``` büyücü yöntemlerini kullanarak GlobalData sunucusu ile iletişime geçilir ve gerçek değişkenler GlobalData sunucusunda depolanır. Örneğin, bir istemci sınıfına mevcut olmayan bir özellik atandığında ```__set``` büyücü yöntemi tetiklenir, istemci sınıfı ```__set``` yönteminde GlobalData sunucusuna bir istek göndererek bir değişkeni saklar. Bir istemci sınıfında mevcut olmayan bir değişkene erişildiğinde, sınıfın ```__get``` yöntemi tetiklenir, istemci GlobalData sunucusuna bir istek gönderir ve bu değeri okur, böylece süreçler arası değişken paylaşımı tamamlanmış olur.


```php
require_once __DIR__ . '/vendor/autoload.php';

// Global Data sunucusuna bağlan
$global = new GlobalData\Client('127.0.0.1:2207');

// $global->somedata var mı diye sorgulamak için $global->__isset('somedata') tetiklenir
isset($global->somedata);

// $global->somedata'ya array(1,2,3) değerini saklamak için $global->__set('somedata',array(1,2,3)) tetiklenir
$global->somedata = array(1,2,3);

// $global->somedata'nın değerini sorgulamak için $global->__get('somedata') tetiklenir
var_export($global->somedata);

// $global->somedata'yı silmek için $global->__unset('somedata') tetiklenir, bu da sunucudan somedata ve karşılık gelen değerini siler
unset($global->somedata);
```
