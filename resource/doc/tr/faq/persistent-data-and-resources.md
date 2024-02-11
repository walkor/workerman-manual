# Nesnelerin ve Kaynakların Kalıcılığı
Geleneksel web geliştirmede, PHP tarafından oluşturulan nesneler, veriler, kaynaklar vb. talep sona erdikten sonra tamamen serbest bırakılır, bu da kalıcı olmalarını zorlaştırır. Ancak Workerman sayesinde bunlar kolayca başarılabilir.

Workerman'de belirli veri kaynaklarını bellekte kalıcı olarak saklamak istiyorsanız, bu kaynakları global değişkenlere veya sınıfın statik üyelerine yerleştirebilirsiniz.

Aşağıdaki kod örneği ile örneğin:

Mevcut işlemdeki istemci bağlantı sayısını saklamak için global bir değişken olan `$connection_count` kullanılır.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Mevcut işlemdeki istemci bağlantı sayısını saklamak için global değişken
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // Yeni istemci bağlandığında bağlantı sayısı+1
    global $connection_count;
    ++$connection_count;
    echo "şu anda connection_count=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // İstemci bağlantısı kapandığında bağlantı sayısı-1
    global $connection_count;
    $connection_count--;
    echo "şu anda connection_count=$connection_count\n";
};
```

## PHP Değişken Kapsamı için Bakınız:
https://php.net/manual/zh/language.variables.scope.php
