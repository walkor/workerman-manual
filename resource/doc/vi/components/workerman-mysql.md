# Workerman/MySQL

## Giới thiệu
Trong quá trình sử dụng chương trình lưu trữ liên tục trong bộ nhớ với MySQL, thường xuyên gặp phải lỗi "mysql gone away", điều này xuất hiện do kết nối giữa chương trình và MySQL không giao tiếp trong thời gian dài, dẫn đến việc máy chủ MySQL ngắt kết nối. Lớp cơ sở dữ liệu này có thể giải quyết vấn đề này, khi phát sinh lỗi "mysql gone away", nó sẽ tự động thử lại một lần.

## Dependents
Lớp MySQL này phụ thuộc vào hai phần mở rộng [pdo](https://php.net/manual/zh/book.pdo.php) và [pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php), thiếu mở rộng sẽ gây ra lỗi "Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ....".

Chạy lệnh ```php -m``` trong dòng lệnh sẽ liệt kê tất cả các phần mở rộng đã được cài đặt trên php cli, nếu không có pdo hoặc pdo_mysql, vui lòng tự cài đặt.

**Trên hệ điều hành centos**

PHP5.x
````
yum install php-pdo
yum install php-mysql
````
PHP7.x
````
yum install php70w-pdo_dblib.x86_64
yum install php70w-mysqlnd.x86_64
````
Nếu không tìm thấy tên gói, thử sử dụng ```yum search php mysql``` để tìm kiếm.

**Trên hệ điều hành ubuntu/debian**

PHP5.x
````
apt-get install php5-mysql
````

PHP7.x
````
apt-get install php7.0-mysql
````

Nếu không tìm thấy tên gói, thử sử dụng ```apt-cache search php mysql``` để tìm kiếm.

**Không thể cài đặt theo cách trên?**

Nếu không thể cài đặt theo cách trên, vui lòng tham khảo [workerman manual - phụ lục - cách cài đặt mở rộng - cách 3: cài đặt bằng mã nguồn](../appendices/install-extension.md).

## Cài đặt Workerman/MySQL
**Cách 1:**
Có thể cài đặt thông qua composer, chạy lệnh sau trong dòng lệnh (nguồn composer ở nước ngoài, quá trình cài đặt có thể rất chậm)..
````
composer require workerman/mysql
````

Sau khi lệnh trên chạy thành công, thư mục vendor sẽ được tạo ra, sau đó nhập autoload.php trong thư mục vendor vào dự án.
```php
require_once __DIR__ . '/vendor/autoload.php';
```

**Cách 2:**
[Tải mã nguồn](https://github.com/walkor/mysql/archive/master.zip), giải nén thư mục sau đó bỏ vào dự án của bạn (vị trí tùy ý), sau đó require tệp nguồn trực tiếp.
```php
require_once '/your/path/of/mysql-master/src/Connection.php';
```

## Lưu ý
Nên khuyến nghị rằng việc khởi tạo kết nối cơ sở dữ liệu trong lời gọi lại onWorkerStart, tránh việc khởi tạo kết nối trước khi chạy ```Worker::runAll();```, kết nối được khởi tạo trước khi chạy ```Worker::runAll();``` thuộc về tiến trình chính, các tiến trình con sẽ kế thừa kết nối này, việc chia sẻ kết nối cơ sở dữ liệu giữa tiến trình chính và tiến trình con sẽ gây ra lỗi.

## Ví dụ
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // Lưu trữ thể hiện db trong biến toàn cục (cũng có thể lưu trữ trong thuộc tính tĩnh của một lớp nào đó)
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Nhận thể hiện db thông qua biến toàn cục
    global $db;
    // Thực thi SQL
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// Khởi chạy worker
Worker::runAll();
```

## Cách sử dụng cụ thể của MySQL/Connection
```php
// Khởi tạo kết nối db
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');

// Lấy tất cả dữ liệu
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
//tương đương
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
//tương đương
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");


// Lấy một hàng dữ liệu
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
//tương đương
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
//tương đương
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");


// Lấy một cột dữ liệu
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
//tương đương
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
//tương đương
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// Lấy một giá trị duy nhất
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
//tương đương
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
//tương đương
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// Truy vấn phức tạp
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// Tương đương
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// Chèn dữ liệu
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
tương đương
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// Cập nhật dữ liệu
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID=1')
->bindValue('sex', 'F')->query();
// Tương đương
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// Tương đương
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// Xóa dữ liệu
$row_count = $db->delete('Persons')->where('ID=9')->query();
// Tương đương
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// Giao dịch
$db->beginTrans();
....
$db->commitTrans(); // hoặc $db->rollBackTrans();
```
