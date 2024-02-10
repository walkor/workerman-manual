# Workerman/MySQL

## Introduction
Resident memory programs often encounter the "mysql gone away" error when using MySQL, which is caused by the lack of communication between the program and MySQL for a long time, leading to the connection being kicked off by the MySQL server. This database class can solve this problem by automatically retrying once when the "mysql gone away" error occurs.

## Dependent Extensions
This MySQL class depends on two extensions, [pdo](https://php.net/manual/zh/book.pdo.php) and [pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php). The absence of these extensions will result in an error of "Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ....".

Run the command `php -m` in the command line to list all the PHP cli installed extensions. If pdo or pdo_mysql is missing, please install them on your own.

**Centos System**

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

If the package name cannot be found, try using `yum search php mysql` to search.

**Ubuntu/Debian System**

PHP5.x
```
apt-get install php5-mysql
```

PHP7.x
```
apt-get install php7.0-mysql
```

If the package name cannot be found, try using `apt-cache search php mysql` to search.

**Unable to Install with Above Methods?**

If the above methods fail to install, please refer to [Workerman Manual-Appendix-Extension Installation-Method Three: Installing from Source Code Compilation](../appendices/install-extension.md).

## Installation of Workerman/MySQL
**Method 1:**

It can be installed via composer by running the following command in the command line (The composer source is overseas, and the installation process may be very slow).
```
composer require workerman/mysql
```

After the successful execution of the above command, a vendor directory will be generated. Then, include the autoload.php under the vendor directory in the project.
```php
require_once __DIR__ . '/vendor/autoload.php';
```

**Method 2:**

[Download the source code](https://github.com/walkor/mysql/archive/master.zip), unzip the directory, and place it in your project (location is arbitrary). Then, require the source file directly.
```php
require_once '/your/path/of/mysql-master/src/Connection.php';
```

## Note
It is strongly recommended to initialize the database connection in the onWorkerStart callback to avoid initializing the connection before running `Worker::runAll();`. The connection initialized before running `Worker::runAll();` belongs to the main process, and the child process will inherit this connection. Sharing the same database connection between the main process and child processes will result in errors.

## Example
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // Store the db instance in a global variable (or it can be stored in a static member of a class)
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Obtain the db instance through a global variable
    global $db;
    // Execute SQL
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// Run the worker
Worker::runAll();
```

## Specific Usage of MySQL/Connection
```php
// Initialize db connection
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');

// Get all data
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
// Equivalent to
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
// Equivalent to
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");


// Get a row of data
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
// Equivalent to
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
// Equivalent to
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");


// Get a column of data
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
// Equivalent to
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
// Equivalent to
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// Get a single value
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
// Equivalent to
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
// Equivalent to
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// Complex query
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// Equivalent to
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// Insert
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
Equivalent to
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// Update
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID=1')
->bindValue('sex', 'F')->query();
// Equivalent to
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// Equivalent to
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// Delete
$row_count = $db->delete('Persons')->where('ID=9')->query();
// Equivalent to
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// Transaction
$db->beginTrans();
....
$db->commitTrans(); // or $db->rollBackTrans();
```
