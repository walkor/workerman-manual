# GlobalData變數共享組件
**``` (要求Workerman版本>=3.3.0) ```**

源碼地址：https://github.com/walkor/GlobalData

## 注意
GlobalData需要Workerman版本>=3.3.0

## 安裝
```
composer require workerman/globaldata
```

## 原理
利用PHP的```__set __get __isset __unset```魔術方法觸發與GlobalData服務端通訊，實際變量存儲在GlobalData服務端。例如當給客戶端類設置一個不存在的屬性時，會觸發```__set```魔術方法，客戶端類在```__set```方法中向GlobalData服務端發送請求，存入一個變量。當訪問客戶端類一個不存在的變量時，會觸發類的```__get```方法，客戶端會向GlobalData服務端發起請求，讀取這個值，從而完成進程間變量共享。


```php
require_once __DIR__ . '/vendor/autoload.php';

// 連接Global Data服務端
$global = new GlobalData\Client('127.0.0.1:2207');

// 觸發$global->__isset('somedata')查詢服務端是否存儲了key為somedata的值
isset($global->somedata);

// 觸發$global->__set('somedata',array(1,2,3))，通知服務端存儲somedata對應的值為array(1,2,3)
$global->somedata = array(1,2,3);

// 觸發$global->__get('somedata')，從服務端查詢somedata對應的值
var_export($global->somedata);

// 觸發$global->__unset('somedata'),通知服務端刪掉somedata及對應的值
unset($global->somedata);
```
