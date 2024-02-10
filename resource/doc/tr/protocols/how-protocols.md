## Protokolü Nasıl Özelleştirebilirim

Gerçekte kendi protokolünü oluşturmak oldukça basittir. Basit bir protokol genellikle iki bölümden oluşur:

* Veri sınırlarını ayırt eden belirleyici
* Veri biçimini tanımlama

## Bir Örnek

### Protokol Tanımı
Burada veri sınırlarını ayırt eden belirleyicinin bir satır sonu "\n" olduğunu varsayalım (belirli veri içinde satır sonu karakteri olmamalıdır), veri biçimi ise Json formatında olduğunu varsayalım. Aşağıdaki örnek, bu kurala uygun bir istek paketi göstermektedir.

<pre>
{"type":"message","content":"hello"}
</pre>

Yukarıdaki istek verisinin sonunda bir satır sonu karakteri bulunmaktadır (PHP'de **çift tırnaklı** dize olarak "\n" olarak temsil edilir), bu bir isteğin sonunu temsil eder.

### Uygulama Adımları
WorkerMan'da yukarıdaki protokolü uygulamak istiyorsak, protokolümüzün adının JsonNL olduğunu ve proje adının MyApp olduğunu varsayalım. Bu durumda aşağıdaki adımları takip etmemiz gerekir.

1. Protokol dosyasını projenin Protocols klasörüne koymalısınız, örneğin dosya şu şekilde olacaktır: MyApp/Protocols/JsonNL.php
2. JsonNL sınıfını uygulamalıyız ve ```namespace Protocols;``` olarak adlandırmalıyız. Bu sınıfın input, encode ve decode olmak üzere üç adet statik yöntemi uygulamamız gerekir.

Not: Workerman bu üç statik yöntemi otomatik olarak çağıracaktır, paketleme, paket açma ve paketleme işlemlerini gerçekleştirmek için. Detaylı işlem akışı için aşağıdaki açıklamalara bakın.

### Workerman ve Protokol Sınıfı Etkileşimi Akışı
1. Varsayalım ki bir istemci, bir veri paketini sunucuya gönderdi. Sunucu, veriyi aldıktan sonra (muhtemelen kısmi veri alacak), hemen protokolün ```input``` yöntemini çağırarak bu paketin uzunluğunu kontrol eder ve ```input``` yöntemi workerman çerçevesine bir uzunluk değeri olan ```$length```ı döndürür.
2. Workerman çerçevesi bu ```$length``` değerini aldıktan sonra, mevcut veri önbelleğinde zaten ```$length``` uzunluğunda veri alınıp alınmadığını kontrol eder. Eğer yoksa, veri beklemeye devam eder, veri önbelleğindeki verinin uzunluğu ```$length``` değerinden küçük olana kadar bekler.
3. Veri önbelleğindeki veri uzunluğu yeterli olduğunda, workerman veri önbelleğinden ```$length``` uzunluğunda veriyi keser (yani **paketleme**) ve protokolün ```decode``` yöntemini **açma** işlemi için çağırır, ardından açılan veri ```$data``` olarak atanır.
4. Açılan veri sonrasında, workerman veriyi ```onMessage($connection, $data)``` geri çağırımına veri olarak ileterek iş akışına devam eder. İş akışı içerisinde, işletme ```$data``` değişkenini kullanarak istemciden gelen tam ve açılmış veriyi alabilir.
5. Eğer iş akışı içerisinde ```$connection->send($buffer)``` yöntemi ile istemciye veri göndermek gerekirse, workerman otomatik olarak protokolün ```encode``` yöntemini kullanarak veriyi **paketleyecek** ve ardından istemciye gönderecektir.

### Özel Uygulama

**MyApp/Protocols/JsonNL.php Uygulaması**

```php
namespace Protocols;
class JsonNL
{
    /**
     * Paketin bütünlüğünü kontrol et
     * Eğer paket uzunluğu belirlenebiliyorsa, paketin tüm uzunluğunu döndürür, aksi takdirde 0 döndürerek daha fazla veri beklemesi gerektiğini belirtir
     * Eğer protokol hatalıysa, yanlış bir değer veya negatif bir değer döndürülebilir, bu durumda bağlantı kapatılır
     * @param string $buffer
     * @return int
     */
    public static function input($buffer)
    {
        // Satır sonu karakteri "\n" konumunu bul
        $pos = strpos($buffer, "\n");
        // Satır sonu yoksa, paket uzunluğu bilinemez, daha fazla veri beklemesi için 0 döndür
        if($pos === false)
        {
            return 0;
        }
        // Satır sonu varsa, mevcut paket uzunluğunu (satır sonu dahil) döndür
        return $pos+1;
    }

    /**
     * Paketleme, istemciye veri gönderildiğinde otomatik olarak çağrılır
     * @param string $buffer
     * @return string
     */
    public static function encode($buffer)
    {
        // Json serileştirme ve istek sonunu belirten satır sonu eklemesi
        return json_encode($buffer)."\n";
    }

    /**
     * Açma, alınan veri baytları input dönüş değerine eşit olduğunda (0'dan büyük bir değer) otomatik olarak çağrılır
     * ve onMessage geri çağırımına $data parametresi olarak iletilir
     * @param string $buffer
     * @return string
     */
    public static function decode($buffer)
    {
        // Satır sonunu kaldır ve diziye geri dönüştür
        return json_decode(trim($buffer), true);
    }
}
```

Bu şekilde, JsonNL protokolü uygulaması tamamlanmış olur ve MyApp projesinde kullanılabilir hale gelir. Aşağıdaki gibi kullanılabilir:

Dosya: MyApp\start.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data, istemciden gelen JsonNL::decode işlemi yapılmış veridir
    echo $data;
    
    // $connection->send ile gönderilen veriler otomatik olarak JsonNL::encode yöntemi ile paketlenir ve ardından istemciye gönderilir
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
...
```

> **Not**
> Workerman, `Protocols` adlı namespace altındaki protokollerin yüklenmeye çalışacaktır, örneğin `new Worker('JsonNL://0.0.0.0:1234')` ifadesi `Protocols\JsonNL` protokolünü yüklemeye çalışacaktır.
> "Class 'Protocols\JsonNL' not found" hatası alınırsa, [otomatik yükleme](../faq/autoload.md) konusunda bilgi almak için bu bağlantıya göz atabilirsiniz.

### ProtocolInterface Hakkında Bilgi
Workerman'da geliştirilen protokol sınıflarının input, encode ve decode olmak üzere üç statik yöntemi uygulaması gerekmektedir. Protokol arayüzü için detaylı bilgiye Workerman/Protocols/ProtocolInterface.php adresinden ulaşabilirsiniz. Tanımı aşağıdaki gibidir:

```php
namespace Workerman\Protocols;

use \Workerman\Connection\ConnectionInterface;

/**
 * Protokol arayüzü
 * @author walkor <walkor@workerman.net>
 */
interface ProtocolInterface
{
    /**
     * Alınan recv_buffer içinde paket ayarlamak için kullanılır
     *
     * Eğer $recv_buffer içindeki istek paketinin uzunluğu belirlenebiliyorsa, geriye paketin tam uzunluğunu döndürür.
     * Aksi takdirde 0 döndürerek bu isteğin mevcut uzunluğunu belirlemek için daha fazla veri beklenmesi gerektiğini belirtir.
     * Eğer false veya negatif bir değer dönerse, bu hatalı bir isteği temsil eder ve bağlantı kesilir.
     *
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     * @return int|false
     */
    public static function input($recv_buffer, ConnectionInterface $connection);

    /**
     * İstek açmak için kullanılır
     *
     * input değeri 0'dan büyük ve WorkerMan istenilen miktarda veri alındığında otomatik olarak decode yöntemini çağırır
     * ve onMessage geri çağırımını tetikler ve açılmış veriyi onMessage geri çağırımının ikinci parametresi olarak iletilir
     * Yani, istemci isteğinin tamamını alınca otomatik olarak decode yöntemini çağırmak için kod yazmaya gerek yoktur.
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     */
    public static function decode($recv_buffer, ConnectionInterface $connection);

    /**
     */
    public static function encode($data, ConnectionInterface $connection);
}
```

## Notlar:
Workerman'da protokol sınıflarının ProtocolInterface'e dayalı olmak zorunda olmadığını belirtmek gerekir. Aslında, protokol sınıfları, sadece input, encode ve decode üç statik yöntemi içeriyorsa çalışabilir.
