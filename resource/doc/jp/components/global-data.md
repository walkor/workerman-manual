# GlobalData変数共有コンポーネント
**```(Workermanのバージョン>=3.3.0が必要です)```**

ソースコードの場所：https://github.com/walkor/GlobalData

## 注記
GlobalDataにはWorkermanのバージョン>=3.3.0が必要です。

## インストール

`composer require workerman/globaldata`

## 原理

PHPの```__set __get __isset __unset```マジックメソッドを使用して、GlobalDataサーバーと通信します。実際の変数はGlobalDataサーバーに保存されます。たとえば、クライアントクラスに存在しないプロパティを設定すると、```__set```マジックメソッドがトリガーされ、クライアントクラスは```__set```メソッドでGlobalDataサーバーにリクエストを送信して変数を保存します。存在しない変数にアクセスすると、クラスの```__get```メソッドがトリガーされ、クライアントはGlobalDataサーバーにリクエストを送信してその値を読み取ります。これによりプロセス間での変数共有が完了します。


```php
require_once __DIR__ . '/vendor/autoload.php';

// Global Dataサーバーに接続
$global = new GlobalData\Client('127.0.0.1:2207');

// $global->__isset('somedata')がトリガーされ、サーバーがキーsomedataの値を保存しているかどうかを確認
isset($global->somedata);

// $global->__set('somedata',array(1,2,3))がトリガーされ、サーバーにsomedataに対応する値としてarray(1,2,3)を保存するよう通知
$global->somedata = array(1,2,3);

// $global->__get('somedata')がトリガーされ、サーバーからsomedataに対応する値を取得
var_export($global->somedata);

// $global->__unset('somedata')がトリガーされ、サーバーにsomedataと対応する値を削除するよう通知
unset($global->somedata);
```
