```php
Kullanıcıyı etkinleştir

Kullandığınız zaman dilimi içinde veri göndermeyen bir istemciyi otomatik olarak kapatmak için nasıl kapatılacağını öğrenmek istiyorsunuz,
örneğin, 30 saniye boyunca bir veri almadan bu istemci bağlantısını otomatik olarak kapatmak, amacı belirli bir süre içinde kimliği doğrulanmamış bağlantıların kimliğini belirlemektir.

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // Geçici olarak $connection nesnesine bir auth_timer_id özelliği ekleyin ve zamanlayıcı kimliği depolayın
    // 30 saniye içinde bağlantıyı kapatma, istemci 30 saniye içinde doğrulama göndermezse zamanlayıcıyı sil
    $connection->auth_timer_id = Timer::add(30, function()use($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
    case 'login':
        ...Kısaltılmış
        // Doğrulama başarılı, zamanlayıcıyı sil ve bağlantının kapatılmasını engelle
        Timer::del($connection->auth_timer_id);
        break;
         ... Kısaltılmış
    }
    ... Kısaltılmış
}
```
