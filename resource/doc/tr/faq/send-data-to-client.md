```php
<?php
kullanma Workerman\Worker;
kullanma Workerman\Connection\TcpConnection;
__DIR__ . '/vendor/autoload.php' gereklidir;

// 1234 bağlantı noktasını dinleyen bir işçi konteyneri başlatın
$worker = yeni Işçi('websocket://workerman.net:1234');
// ==== Bu özelliği 1 olarak ayarlamak zorunludur ====
$worker->count = 1;
// uidConnections adında bir ekleme özelliği, uid'yi bağlantıya eşlemek için(uid kullanıcı kimliği veya istemciye özgü tanımlayıcı)
$worker->uidConnections = array();
// Bir istemci mesaj gönderdiğinde çalıştırılacak geri çağrı fonksiyonu
$worker->onMessage = işlev(TcpConnection $connection, $data)
{
    global $worker;
    // Mevcut istemcinin doğrulanıp doğrulanmadığını kontrol edin, yani uid'nin ayarlanıp ayarlanmadığını kontrol edin
    if(!isset($connection->uid))
    {
       // Doğrulanmamışsa, ilk paketi uid olarak alın (burada gerçek bir doğrulama olmadığını göstermek için yapılmıştır)
       $connection->uid = $data;
       /* uid'yi bağlantıya eşleme, böylece uid'ye göre bağlantıyı kolayca bulabilirsiniz,
        * belirli bir uid'ye veri yayımlama
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('giriş başarılı, uid\'niz ' . $connection->uid);
    }
    // Diğer mantık, belirli bir uid'ye gönderme veya genel yayın
    // Mesaj formatı uid: mesaj olduğunda belirli uid'ye mesaj gönder
    // uid tüm olduğunda genel yayın
    list($recv_uid, $message) = explode(':', $data);
    // Genel yayın
    if($recv_uid == 'all')
    {
        yayın($message);
    }
    // Belirli uid'ye gönderme
    else
    {
        uidyeMesajGönder($recv_uid, $message);
    }
};

// Bir istemci bağlantısı kesildiğinde
$worker->onClose = işlev(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // Bağlantı kesildiğinde eşlemeyi kaldırın
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Tüm doğrulanmış kullanıcılara veri yayımlama
function yayın($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Uid'ye göre veri yayımlama
function uidyeMesajGönder($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}

// Tüm işçileri çalıştırın(şu anda sadece bir tane tanımlandı)
Worker::runAll();
```
