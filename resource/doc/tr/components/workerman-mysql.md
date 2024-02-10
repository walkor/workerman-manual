# Workerman/MySQL

## Açıklama
Sürekli bellekte çalışan bir program, genellikle mysql kullanırken ```mysql gone away``` hatasıyla karşılaşır, bu, programın MySQL ile iletişim kurmaması sonucu uzun süre bağlantının kesilmesine neden olan bir hatadır. Bu veritabanı sınıfı bu sorunu çözebilir, ```mysql gone away``` hatası oluştuğunda otomatik olarak bir kez yeniden deneyecektir.

## Bağımlılıklar
Bu mysql sınıfı [pdo](https://php.net/manual/zh/book.pdo.php) ve [pdo_mysql](https://php.net/manual/zh/ref.pdo-mysql.php) iki uzantıya bağlıdır, eksik uzantılar ```Undefined class constant 'MYSQL_ATTR_INIT_COMMAND' in ....``` hatasına neden olabilir.

Komut satırında ```php -m``` komutunu çalıştırarak yüklü PHP CLI uzantılarının listesini görebilirsiniz, eğer pdo veya pdo_mysql bulamazsanız lütfen kendiniz yükleyin.

**Centos Sistemi**

PHP5.x
```bash
yum install php-pdo
yum install php-mysql
```

PHP7.x
```bash
yum install php70w-pdo_dblib.x86_64
yum install php70w-mysqlnd.x86_64
```
Eğer paket adını bulamazsanız, ```yum search php mysql``` komutunu kullanarak arayabilirsiniz.

**Ubuntu/Debian Sistemi**

PHP5.x
```bash
apt-get install php5-mysql
```

PHP7.x
```bash
apt-get install php7.0-mysql
```
Eğer paket adını bulamazsanız, ```apt-cache search php mysql``` komutunu kullanarak arayabilirsiniz.

**Yukarıdaki yöntemlerle yüklenemiyor mu?**

Eğer yukarıdaki yöntemlerle yüklenemiyorsa, lütfen [Workerman el kitabı - Ekler - Uzantıları Yükleme - Üçüncü Yöntem: Kaynak Kodunu Derleyerek Yükleme](../appendices/install-extension.md) bölümüne bakın.

## Workerman/MySQL Kurulumu
**Yöntem 1:**

Composer aracılığıyla aşağıdaki komutu çalıştırarak yükleyebilirsiniz (composer kaynağı yurtdışında olduğu için kurulum süreci oldukça yavaş olabilir).

```bash
composer require workerman/mysql
```

Yukarıdaki komut başarıyla çalıştıktan sonra vendor dizini oluşturulur, ardından projeye bu dizinin altındaki autoload.php dosyasını ekleyin.

```php
require_once __DIR__ . '/vendor/autoload.php';
```

**Yöntem 2:**

[Kaynak kodunu indirin](https://github.com/walkor/mysql/archive/master.zip), ardından zip dosyasını açarak (herhangi bir konuma) projenize kopyalayın, sonra kaynak dosyasını doğrudan require edin.

```php
require_once '/sizin/dizininiz/mysql-master/src/Connection.php';
```

## Not
Database bağlantısını ```onWorkerStart``` geri çağrı fonksiyonunun içinde başlatmanızı şiddetle tavsiye ederiz, ```Worker::runAll();``` çalıştırılmadan önce bağlantıyı başlatırsanız, bu bağlantı ana sürece ait olur ve alt süreçler bu bağlantıyı devralır, ana süreç ve alt süreçlerin aynı veritabanı bağlantısını paylaşması hatalara neden olabilir.

## Örnek
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    // db örneğini global değişken içinde saklayın (aynı şekilde bir sınıfın statik üyesinde saklayabilirsiniz)
    global $db;
    $db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Global değişken aracılığıyla db örneğine erişin
    global $db;
    // SQL'i çalıştırın
    $all_tables = $db->query('show tables');
    $connection->send(json_encode($all_tables));
};
// Worker'ı çalıştırın
Worker::runAll();
```

## Workerman/MySQL/Connection Kullanımı
```php
// db bağlantısını başlatın
$db = new \Workerman\MySQL\Connection('host', 'port', 'user', 'password', 'db_name');

// Tüm verileri alın
$db->select('ID,Sex')->from('Persons')->where('sex= :sex AND ID = :id')->bindValues(array('sex'=>'M', 'id' => 1))->query();
//eşdeğeri
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' AND ID = 1")->query();
//eşdeğeri
$db->query("SELECT ID,Sex FROM `Persons` WHERE sex='M' AND ID = 1");

// Bir satır veri al
$db->select('ID,Sex')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->row();
//eşdeğeri
$db->select('ID,Sex')->from('Persons')->where("sex= 'M' ")->row();
//eşdeğeri
$db->row("SELECT ID,Sex FROM `Persons` WHERE sex='M'");

// Bir sütun veri al
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->column();
//eşdeğeri
$db->select('ID')->from('Persons')->where("sex= 'F' ")->column();
//eşdeğeri
$db->column("SELECT `ID` FROM `Persons` WHERE sex='M'");

// Tek bir değer al
$db->select('ID')->from('Persons')->where('sex= :sex')->bindValues(array('sex'=>'M'))->single();
//eşdeğeri
$db->select('ID')->from('Persons')->where("sex= 'F' ")->single();
//eşdeğeri
$db->single("SELECT ID FROM `Persons` WHERE sex='M'");

// Karmaşık sorgular
$db->select('*')->from('table1')->innerJoin('table2','table1.uid = table2.uid')->where('age > :age')->groupBy(array('aid'))->having('foo="foo"')->orderByASC/*orderByDESC*/(array('did'))
->limit(10)->offset(20)->bindValues(array('age' => 13));
// eşdeğeri
$db->query('SELECT * FROM `table1` INNER JOIN `table2` ON `table1`.`uid` = `table2`.`uid`
WHERE age > 13 GROUP BY aid HAVING foo="foo" ORDER BY did LIMIT 10 OFFSET 20');

// Ekleme
$insert_id = $db->insert('Persons')->cols(array(
    'Firstname'=>'abc',
    'Lastname'=>'efg',
    'Sex'=>'M',
    'Age'=>13))->query();
eşdeğeri
$insert_id = $db->query("INSERT INTO `Persons` ( `Firstname`,`Lastname`,`Sex`,`Age`)
VALUES ( 'abc', 'efg', 'M', 13)");

// Güncelleme
$row_count = $db->update('Persons')->cols(array('sex'))->where('ID=1')
->bindValue('sex', 'F')->query();
// eşdeğeri
$row_count = $db->update('Persons')->cols(array('sex'=>'F'))->where('ID=1')->query();
// eşdeğeri
$row_count = $db->query("UPDATE `Persons` SET `sex` = 'F' WHERE ID=1");

// Silme
$row_count = $db->delete('Persons')->where('ID=9')->query();
// eşdeğeri
$row_count = $db->query("DELETE FROM `Persons` WHERE ID=9");

// İşlem
$db->beginTrans();
....
$db->commitTrans(); // veya $db->rollBackTrans();
```
