# Geliştirme Öncesi Okunması Gerekenler

Workerman uygulaması geliştirmek için aşağıdaki konuları anlamanız gerekmektedir:

## Birinci, Workerman Geliştirme ile Normal PHP Geliştirme Arasındaki Farklar

Workerman geliştirme ile normal PHP geliştirme arasında, HTTP protokolü ile ilgili değişken fonksiyonların doğrudan kullanılamaması dışında büyük farklılık bulunmamaktadır.

### 1. Uygulama Katmanı Protokolü Farklılığı
* Normal PHP geliştirme genellikle HTTP uygulama katmanı protokolüne dayanır, Web Sunucusu geliştiriciler için protokolün çözümü yapılmıştır.
* Workerman, çeşitli protokolleri destekler, şu anda HTTP, WebSocket gibi protokoller yerleşiktir. Workerman, geliştiricilere daha basit özel iletişim protokollerini kullanmalarını önerir.

*  HTTP protokolü geliştirmesi için lütfen [HTTP hizmet bölümüne](../http/request.md) bakınız.

### 2. İstek Döngüsü Farkı
* PHP web uygulamasında bir istekten sonra tüm değişkenleri ve kaynakları serbest bırakacaktır.
* Workerman geliştirmesi yapılan uygulama, ilk yükleme ve çözme sonrasında sürekli bellekte kalır, bu nedenle sınıf tanımları, genel nesneler, sınıfın statik üyeleri serbest bırakılmaz, tekrar kullanım için kolaylaştırır.

### 3. Sınıf ve Sabit Tanımlarının Tekrar Define Edilmesinden Kaçının
* Workerman derlenmiş PHP dosyalarını önbelleğe alacağından, aynı sınıf veya sabit tanım dosyalarını birden fazla kez gerektirmekten kaçınmak gerekir. Genellikle require_once/include_once kullanımını öneririz.

### 4. Tekil Moddaki Bağlantı Kaynağının Serbest Bırakılmasına Dikkat Edin
* Workerman, her istek sonrası genel nesneleri ve sınıfın statik üyelerini serbest bırakmayacağı için, veritabanı gibi tekil modda bulunan kaynakları genellikle veritabanı sınıfının içindeki (içeriden bir veritabanı soket bağlantısı içeren) veritabanı örneğini saklar, böylece Workerman işlem ömrü boyunca bu veritabanı soket bağlantısını yeniden kullanır. Dikkat edilmesi gereken nokta, veritabanı sunucusunun belirli bir süre boyunca etkin olmadığı bir bağlantıyı kapatma eğiliminde olmasıdır, bu durumda bu veritabanı örneğini tekrar kullandığınızda (hata mesajı genellikle mysql gone away gibi) hata alırsınız. Workerman, [veritabanı sınıfını](../components/workerman-mysql.md) sunar, kesintisiz yeniden bağlanma işlevi mevcuttur, geliştiriciler bu işlevi doğrudan kullanabilirler.

### 5. exit, die gibi ifadeleri kullanmaktan kaçının
* Workerman, PHP komut satırı modunda çalışır, exit, die çıkış ifadeleri çağrıldığında, mevcut işlemi sonlandırır. Alt süreçlerin çıkışından sonra hemen aynı alt sürecin yeniden oluşturulmasına rağmen, işletme üzerinde hala olumsuz bir etkiye neden olabilir.

### 6. Kodu Değiştirdikten Sonra Hizmeti Yeniden Başlatmanız Gerekir
Workerman sürekli bellekte olduğundan, php sınıfı ve fonksiyon tanımları bir kez yüklendikten sonra sürekli bellekte kalır, tekrar diskten okunmaz, bu nedenle işletme kodunu değiştirdikten sonra her seferinde başlatılması gerekir.

## İkinci, Temel Kavramları Anlamak

### 1. TCP Taşıma Katmanı Protokolü
TCP, güvenilir, IP tabanlı, bağlantı odaklı bir iletişim protokolüdür. TCP, veri akışı tabanlı bir özelliğe sahiptir, istemcinin isteği sürekli olarak sunucuya gönderilebilir, sunucu tarafından alınan veri tam bir istek olmayabilir, aynı zamanda birden fazla isteği bir arada içerebilir. Bu nedenle bu sürekli olarak veri akışı içinde her isteğin sınırını ayırt etmemiz gerekmektedir. Uygulama katmanı protokolü genellikle isteğin sınırını tanımlar ve istek verilerinin karmaşık olmasını engeller.

### 2. Uygulama Katmanı Protokolü
Uygulama katmanı protokolü (application layer protocol), farklı uç sistemlerde (istemci, sunucu) çalışan uygulama programlarının nasıl iletişim sağlayacağını tanımlar, örneğin HTTP, WebSocket gibi uygulama katmanı protokolleri buna örnektir. Örneğin, basit bir uygulama katmanı protokolü şu şekilde olabilir: ```{"module":"kullanıcı", "aksiyon":"getBilgi", "uid":456} \n```. Bu protokol, isteğin bitişi ```"\n"``` (burada ```"\n"```, satır sonu olduğuna dikkat edin) ile belirtilir ve mesaj gövdesi bir dizedir.

### 3. Kısa Bağlantılar
Kısa bağlantı, iletişim iki taraf arasında veri alışverişi olduğunda bir bağlantı kurulur, veri gönderildikten sonra bu bağlantı kesilir, yani her bağlantı sadece bir işin gönderilmesini tamamlar. Web sitelerinin HTTP hizmetlerinin genellikle kısa bağlantı kullandığını belirtmekte fayda var.

*Kısa bağlantılı uygulama geliştirme süreci temel geliştirme akışına bakılabilir.*

### 4. Uzun Bağlantılar
Uzun bağlantı, bir bağlantı üzerinden ardışık çok sayıda veri paketi gönderilebilir.

Not: Uzun bağlantı kullanan uygulamaların [kalp atışı](../faq/heartbeat.md) eklemeleri gerekir, aksi takdirde bağlantı uzun süre etkin olmadığından bir yönlendirme düğümü veya güvenlik duvarı tarafından kesilebilir.

Uzun bağlantı, sık işlemler, noktadan noktaya iletişim gibi durumlarda kullanılır. Her TCP bağlantısı üç el sıkışma gerektirir, bu zaman alıcıdır, her işlemden sonra bağlantının kesilmesi yerine, ardışık veri paketleri gönderilmediği sürece aynı bağlantıyı korumak daha uygundur. Örneğin, veritabanı bağlantısı uzun bağlantı kullanır, eğer kısa bağlantı kullanılırsa sık sık socket hatalarına neden olur ve sık sık soket oluşturmak da kaynak israfıdır.

*İstemci tarafından veri aktarımı yapılması gereken durumlarda, örneğin sohbet, anlık oyun, mobil bildirim gibi uygulamalarda uzun bağlantıya ihtiyaç vardır.*

### 5. Düzgün Yeniden Başlatma
Normal bir yeniden başlatma süreci, tüm süreçleri duraklatır ve ardından tüm yeni hizmet süreçlerini oluşturur. Bu süreçte bir süre hizmet dışı kalınacağı için yüksek talep anında istek başarısızlığına neden olabilir.

Düzgün yeniden başlatma ise tüm süreçleri aynı anda durdurmak yerine, her bir süreci duraklatıp hemen ardından yeni bir süreç oluşturarak, eski tüm süreçler yerine yenileriyle değiştirene kadar devam eder.

Workerman'da, ```php your_file.php reload``` komutu, uygulama programını güncellerken hizmet kalitesini etkilemeden işlem yapmanızı sağlar.

**Not: Yalnızca on{...} geri çağırma işlevleri içe aktarılan dosyaları yeniden başlatır, başlatma betiğine doğrudan aktarılan dosyalar veya sert kodlar yeniden başlatma yapmaz.**

## Üçüncü, Ana Süreç ve Alt Süreç Arasındaki Farkı Belirleme
Kodun ana süreçte mi yoksa alt süreçte mi çalıştığını belirlemek önemlidir, genellikle ```Worker::runAll();``` çağrılmadan önce çalışan kodlar ana süreçte çalışır, onXXX geri çağırma işlevleri alt sürece aittir. ```Worker::runAll();```'den sonraki kodunuz hiçbir zaman çalıştırılmayacaktır.

Aşağıdaki kod örneği gibi:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Ana süreçte çalışır
$tcp_worker = new Worker("tcp://0.0.0.0:2347");
// Atama işlemi ana süreçte çalışır
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Bu bölüm alt süreçte çalışır
    $connection->send('hello ' . $data);
};

Worker::runAll();
```

**Not: Ana süreçte veritabanı, memcache, redis gibi bağlantı kaynaklarını başlatmayın, çünkü ana süreçte başlatılan bağlantılar alt süreçler tarafından otomatik olarak devralınabilir (özellikle tekil modda kullanılıyorsa), tüm süreçler aynı bağlantıyı tutar ve sunucu tarafından döndürülen veriler tüm süreçlerde okunabilir, bu da veri karmaşasına neden olur. Benzer şekilde, herhangi bir süreç bağlantıyı kapatırsa (örneğin daemon modunda ana süreç çıkacağı için), tüm alt süreç bağlantıları birlikte kapatılır ve öngörülemeyen hatalar meydana gelebilir, örneğin mysql gone away hatası. Bağlantı kaynaklarını "onWorkerStart" içinde başlatmanızı öneririz.**
