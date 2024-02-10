# connect Metodu
```php
void AsyncTcpConnection::connect()
```
Asenkron bağlantı işlemini gerçekleştirir. Bu metod hemen geri döner.

Not: Asenkron bağlantının onError geri çağrısının ayarlanması gerekiyorsa, bu ayar connect yönteminden önce yapılmalıdır. Aksi takdirde onError geri çağrısı tetiklenmeyebilir, örneğin aşağıdaki örnekte onError geri çağrısının tetiklenemeyeceği ve asenkron bağlantı başarısız olursa yakalanamayacağı durumu söz konusu olabilir.

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// onError geri çağrısı henüz ayarlanmadığı zaman bağlantıyı gerçekleştirir
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### Parametre
Parametre yok.

### Dönüş Değeri
Dönüş değeri yok.

### MySQL Proxy Örneği

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Gerçek mysql adresi, burada örneğin yerel 3306 bağlantı noktası
$GERÇEK_MYSQL_ADRESİ = 'tcp://127.0.0.1:3306';

// Proxy yerel 4406 bağlantı noktasını dinler
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $GERÇEK_MYSQL_ADRESİ;
    // Gerçek mysql sunucusuna asenkron bir bağlantı oluştur
    $connection_to_mysql = new AsyncTcpConnection($GERÇEK_MYSQL_ADRESİ);
    // Mysql bağlantısı veri gönderdiğinde, ilgili istemci bağlantısına iletilir
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer) use ($connection)
    {
        $connection->send($buffer);
    };
    // Mysql bağlantısı kapandığında, ilgili istemciye bağlantısını kapat
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // Mysql bağlantısında bir hata oluşursa, ilgili istemciye bağlantısını kapat
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // Asenkron bağlantıyı gerçekleştir
    $connection_to_mysql->connect();

    // İstemci veri gönderdiğinde, ilgili mysql bağlantısına iletilir
    $connection->onMessage = function(TcpConnection $connection, $buffer) use ($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // İstemci bağlantısı kapandığında, ilgili mysql bağlantısını kapat
    $connection->onClose = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // İstemci bağlantısında bir hata oluşursa, ilgili mysql bağlantısını kapat
    $connection->onError = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
};

// Worker çalıştır
Worker::runAll();
```

 **Test**
```sh
mysql -uroot -P4406 -h127.0.0.1 -p

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 25004
Server version: 5.5.31-1~dotdeb.0 (Debian)

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
