# 開発ガイド

## アプリケーションディレクトリ

アプリケーションディレクトリは任意の場所に配置できます。

## エントリファイル

Nginx + PHP-FPM環境下のPHPアプリケーションと同様に、Workermanのアプリケーションもエントリファイルが必要であり、エントリファイルの名前に厳格な要件はなく、このエントリファイルはPHP Cliで実行されます。

エントリファイルにはリスニングプロセスに関連するコードが含まれており、以下はWorkermanベースの開発コードの例です。

test.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 2345ポートでHTTPプロトコルを使用するWorkerを作成
$http_worker = new Worker("http://0.0.0.0:2345");

// 4つのプロセスを起動してサービスを提供
$http_worker->count = 4;

// ブラウザからのデータを受信した際に"hello world"をブラウザに返信
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // ブラウザに"hello world"を送信
    $connection->send('hello world');
};

Worker::runAll();

```

## Workermanのコーディング規約

1. クラス名はキャメルケースで最初の文字を大文字にし、ファイル名とクラス名が一致するようにする必要があります。例：
```php
class UserInfo
{
...
```

2. 名前空間を使用し、名前空間はディレクトリパスに対応し、開発者のプロジェクトのルートディレクトリを基準にします。

例えば、プロジェクトMyApp/があり、クラスファイルMyApp/MyClass.phpはプロジェクトのルートディレクトリにあるため、名前空間は省略されます。クラスファイルMyApp/Protocols/MyProtocol.phpはMyProtocol.phpがMyAppプロジェクトのProtocolsディレクトリにあるため、```namespace Protocols;``` を追加する必要があります。以下は例です：
```php
namespace Protocols;
class MyProtocol
{
....
```

3. 一般的な関数と変数名には、小文字とアンダースコアを使用します。例：
```php
$connection_list = array();
function get_connection_list()
{
....
```

4. クラスのメンバーとクラスのメソッドには、キャメルケースで最初の文字を小文字にします。例：
```php
public $connectionList;
public function getConnectionList();
```

5. 関数やクラスのパラメータには、小文字とアンダースコアを使用します。例：
```php
function get_connection_list($one_param, $tow_param)
{
....
```
