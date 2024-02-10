# Workerman/MySQL

## 説明
メモリ内に常駐しているプログラムは、MySQLを使用する際にしばしば ```mysql gone away``` のエラーに遭遇します。これは、プログラムとMySQLの接続が長時間通信していないために、接続がMySQLサーバーにキックアウトされるためです。このデータベースクラスは、 ```mysql gone away``` エラーが発生した場合に自動的に1回再試行します。

## 依存する拡張
このmysqlクラスは [pdo](https://php.net/manual/zh/book.pdo.php) と [pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php) という2つの拡張機能に依存しています。これらの拡張機能が欠如していると、```Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ....```というエラーが発生します。

コマンドラインで ```php -m``` を実行すると、PHP CLIにインストールされているすべての拡張機能がリストされます。もしpdoやpdo_mysqlがなければ、自分でインストールしてください。

**centosシステム**

PHP5.x
``` 
yum install php-pdo
yum install php-mysql
```

PHP7.x
``` 
yum install php70w-pdo_dblib.x86_64
yum install php70w-mysqlnd.x86_64
```

もしパッケージ名が見つからない場合は、```yum search php mysql``` を使用して検索してください。

**ubuntu/debianシステム**

PHP5.x
``` 
apt-get install php5-mysql
```

PHP7.x
``` 
apt-get install php7.0-mysql
```

パッケージ名が見つからない場合は、```apt-cache search php mysql``` を使用して検索してください。

**上記の方法でインストールできない場合は？**

上記の方法でインストールできない場合は、[Workermanマニュアル-付録-拡張機能のインストール-方法3ソースコードでのコンパイルおよびインストール](../appendices/install-extension.md)を参照してください。

## Workerman/MySQLのインストール
**方法1：**

composerを使用してインストールすることができます。以下のコマンドをコマンドラインで実行します（composerのソースは国外にあり、インストールプロセスが非常に遅くなることがあります）。

``` 
composer require workerman/mysql
```

上記のコマンドが成功すると、vendorディレクトリが作成され、プロジェクトでvendorディレクトリ内のautoload.phpをインクルードする必要があります。

```php 
require_once __DIR__ . '/vendor/autoload.php';
```

**方法2：**

[ソースコードをダウンロード](https://github.com/walkor/mysql/archive/master.zip)して展開し、プロジェクト内の任意の場所に配置し、ソースファイルを直接requireします。

```php 
require_once '/your/path/of/mysql-master/src/Connection.php';
```

## 注意
```onWorkerStart``` コールバックでデータベース接続を初期化することを強くお勧めします。```Worker::runAll();``` の実行前に接続を初期化すると、その接続は主プロセスに属し、サブプロセスがこの接続を継承するため、主プロセスとサブプロセスで同じデータベース接続を共有するとエラーが発生します。

## サンプル
```php 
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // dbインスタンスをグローバル変数に保存する（またはクラスの静的メンバに保存することもできます）
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // グローバル変数を使用してdbインスタンスを取得する
    global $db;
    // SQLを実行
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// workerを実行
Worker::runAll();
```
## Workerman/MySQLのConnectionの具体的な使用法
```php 
// db接続の初期化
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');

// すべてのデータを取得
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
// または
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
// または
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");


// 1行のデータを取得
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
// または
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
// または
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");


// 1列のデータを取得
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
// または
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
// または
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// 単一の値を取得
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
// または
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
// または
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// 複雑なクエリ
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// または
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// 挿入
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
または
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// 更新
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID=1')
->bindValue('sex', 'F')->query();
// または
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// または
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// 削除
$row_count = $db->delete('Persons')->where('ID=9')->query();
// または
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// トランザクション
$db->beginTrans();
....
$db->commitTrans(); // または $db->rollBackTrans();

```
