# Workerman/MySQL

## 說明
長時間未進行通訊的常駐記憶體程序在使用mysql時經常會遇到```mysql gone away```的錯誤。這個問題是由於程序與mysql的連接長時間沒有通訊，導致連接被mysql服務端踢掉造成的。本數據庫類可以解決這個問題，當發生```mysql gone away```錯誤時，會自動重試一次。

## 依賴的擴展
該mysql類依賴[pdo](https://php.net/manual/zh/book.pdo.php)和[pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php)兩個擴展，缺少擴展會報```Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ....```錯誤。

命令行運行```php -m```會列出所有php cli已安裝的擴展，如果沒有pdo 或者 pdo_mysql，請自行安裝。

**centos 系統**

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

如果找不到包名，請嘗試用```yum search php mysql```查找

**ubuntu/debian 系統**

PHP5.x
``` 
apt-get install php5-mysql
```

PHP7.x
``` 
apt-get install php7.0-mysql
```

如果找不到包名，請嘗試用```apt-cache search php mysql```查找

**以上方法無法安裝？**

如果以上方法無法安裝，請參考[workerman手冊-附錄-擴展安裝-方法三源碼編譯安裝](../appendices/install-extension.md)。

## 安裝 Workerman/MySQL
**方法1：**

可以通過composer安裝，命令行運行以下命令(composer源在國外，安裝過程可能會非常慢)。

``` 
composer require workerman/mysql
```

上面命令成功後會生成vendor目錄，然後在專案中引入vendor下的autoload.php。

```php
require_once __DIR__ . '/vendor/autoload.php';
```

**方法2：**

[下載源碼](https://github.com/walkor/mysql/archive/master.zip)，解壓後的目錄放到自己專案中(位置任意)，直接require源文件。

```php
require_once '/your/path/of/mysql-master/src/Connection.php';
```

## 注意
強烈建議在onWorkerStart回調中初始化數據庫連接，避免在```Worker::runAll();```運行前就初始化連接，在```Worker::runAll();```運行前初始化的連接屬於主進程，子進程會繼承這個連接，主進程和子進程共用相同的數據庫連接會導致錯誤。

## 示例
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // 將db實例存儲在全局變數中(也可以存儲在某類的靜態成員中)
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 通過全局變數獲得db實例
    global $db;
    // 執行SQL
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// 運行worker
Worker::runAll();
```

## 具體MySQL/Connection用法
```php
// 初始化db連接
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');

// 獲取所有數據
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
//等價於
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
//等價於
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");


// 獲取一行數據
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
//等價於
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
//等價於
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");


// 獲取一列數據
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
//等價於
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
//等價於
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// 獲取單個值
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
//等價於
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
//等價於
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// 複雜查詢
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// 等價於
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// 插入
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
等價於
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// 更新
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID=1')
->bindValue('sex', 'F')->query();
// 等價於
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// 等價於
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// 刪除
$row_count = $db->delete('Persons')->where('ID=9')->query();
// 等價於
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// 事務
$db->beginTrans();
....
$db->commitTrans(); // or $db->rollBackTrans();
```
