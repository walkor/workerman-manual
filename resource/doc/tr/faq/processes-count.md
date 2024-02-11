# Kaç tane işlem açılmalı

## İşlem sayısının nasıl ayarlanacağı
İşlem sayısı, "count" özelliği tarafından belirlenir (Windows sistemleri işlem sayısını desteklemez), aşağıdaki gibi bir kod örneğiyle:
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Dış dünyaya hizmet vermek için 4 işlem başlat
$http_worker->count = 4;

...
```

## İşlem sayısı ayarı aşağıdaki durumları dikkate almalıdır
1. CPU çekirdek sayısı
2. Bellek miktarı
3. İşletme IO yoğunluğu mu yoksa CPU yoğunluğu mu

## İşlem sayısı ayarı prensipleri

1. Her işlem için kullanılan toplam bellek miktarı toplam bellek miktarından küçük olmalıdır (genellikle her işlem için yaklaşık 40 MB bellek kullanılır).
2. IO yoğunluğu durumunda, yani işte bazı **engelleme tabanlı** IO bulunan durumlarda genellikle işlem sayısı yüksek olabilir, örneğin CPU çekirdek sayısının 3 katı olarak ayarlanabilir. Eğer işte bekleyen engelleme çok fazlaysa, işlem sayısı daha da artırılabilir, örneğin CPU çekirdek sayısının 8 katı veya daha fazlası. **Engelleme tabanlı** IO olmayan durumlar CPU yoğunluğuna dahil edilir.
3. CPU yoğunluğu durumunda, yani işte **engelleme tabanlı** IO yoksa, örneğin işletmenin asenkron IO kullanarak ağ kaynaklarını okuyarak işlem kodunu engellemeyeceği durumlarda, işlem sayısını CPU çekirdek sayısıyla aynı ayarlayabilirsiniz.

## İşlem sayısı referans değerleri
İşletme kodu genellikle IO yoğunluğu durumunda, IO yoğunluğu seviyesine göre işlem sayısını ayarlayabilirsiniz, örneğin CPU çekirdek sayısının 3-8 katı. 
İşletme kodu genellikle CPU yoğunluğu durumunda ise işlem sayısını CPU çekirdek sayısına ayarlayabilirsiniz.

## Dikkat
Workerman'ın kendi IO işlemleri engelleme olmaksızın olduğundan, örneğin "Connection->send" gibi işlemler, CPU yoğunluğuna dahildir. İşletmenin hangi türde olduğunu bilmiyorsanız, işlem sayısını genellikle CPU çekirdek sayısının yaklaşık 3 katı olarak ayarlayabilirsiniz.
Ayrıca, işlem sayısı ne kadar fazla olursa, işlem değiştirme maliyetinin artacağını unutmayın, bu da performansı etkileyebilir.
