# Müşteri Bağlantı Başarısızlık Nedenleri

Müşteri tarafındaki bağlantı başarısızlığı genellikle ```bağlantı reddedildi``` ve ```bağlantı zaman aşımına uğradı``` olmak üzere iki tür hata alır.

## bağlantı reddedildi

Bu genellikle aşağıdaki nedenlerden dolayı oluşur:
1. Müşteri yanlış bağlantı noktasına bağlandı
2. Müşteri yanlış bir alan adı veya IP'ye bağlandı
3. Eğer müşteri bir alan adı kullanıyorsa, alan adı yanlış bir sunucu IP'sine işaret edebilir
4. Sunucu bir hızlandırıcı proxy gibi bir CDN kullanıyorsa, bağlantı gerçek IP ile beklenen IP uyumsuzdur
5. Sunucu başlatılmadı veya bağlantı noktası dinlemede değil
6. Ağ proxy yazılımı kullanımı
7. Sunucu dinleme IP'si ve erişilen adres aynı adres aralığında değil. Örneğin, sunucu 127.0.0.1'i dinliyorsa, istemci yalnızca 127.0.0.1 üzerinden bağlanabilir, yerel ağ IP veya dış ağ IP üzerinden bağlanamaz. Önerilen dinleme adresi 0.0.0.0 olarak ayarlanmalıdır, böylece yerel, iç ve dış ağ hepsi bağlanabilir.

## bağlantı zaman aşımına uğradı

Bu genellikle aşağıdaki nedenlerden dolayı oluşur:
1. Sunucu güvenlik duvarı bağlantıyı engelledi, geçici olarak güvenlik duvarını kapatın ve deneyin.
2. Bulut sunucuysa, güvenlik grubu da bağlantıyı engelleyebilir, karşılık gelen bağlantı noktalarını yönetim panelinden açmanız gerekir.
3. BT paneli gibi bir panel kullandıysanız, karşılık gelen bağlantı noktalarını panelde açmanız gerekir.
4. Sunucu mevcut değil veya başlatılmadı.
5. Eğer müşteri bir alan adı kullanıyorsa, alan adı yanlış bir sunucu IP'sine işaret edebilir
6. Müşteri ziyaret ettiği IP sunucu iç ağ IP'si ise ve müşteri ile sunucu aynı yerel ağda değilse

## cannot assign requested address (要求されたアドレスを割り当てることができない)

**İstemci olarak** her bağlantı için geçici olarak yerel bir bağlantı noktası tahsis etmelidir. Bir sunucu varsayılan olarak yaklaşık 2-3 bin geçici bağlantı noktasını kullanabilir. Belirli bir sunucuya yapılan bağlantılar bu değeri aşarsa, kullanılabilir bağlantı noktalarını tahsis edemez ve bu hata oluşur. Yerel geçici bağlantı noktası sayısını artırmak için, `/etc/sysctl.conf` dosyasındaki `net.ipv4.ip_local_port_range` kernel parametresini değiştirebilirsiniz. Örneğin, bunu `10000 65535` olarak ayarlayarak etkinleştirebilirsiniz. Ayrıca, bağlantı kesildikten sonra bağlantılar TIME_WAIT durumuna geçer ve karşılık gelen yerel bağlantı noktasını belirli bir süre için işgal eder. Yani, kısa bir süre içinde (2-3 binin üzerinde) büyük miktarda kısa bağlantılar kurmak, `Cannot assign requested address` hatasına neden olabilir. Bu durumda, bu sorunu çözmek için kernel yapılandırmasını TIME_WAIT'ı hızlı bir şekilde toplamak üzere ayarlayabilirsiniz. Daha fazla bilgi için lütfen [Kernel Optimization](https://www.workerman.net/doc/workerman/appendices/kernel-optimization.html) bağlantısına bakınız.

> **Not:** Yerel port sınırlaması yalnızca istemcilere uygulanır, sunucularda yerel port sınırlaması bulunmaz. Yeterli kaynak varsa, sunucu bağlantı sayısı sınırsız olarak kabul edilebilir.

## Diğer Hatalar
Eğer karşılaşılan hata, `connection refuse` veya `connection timeout` değilse, genellikle aşağıdaki nedenlerden biri olabilir:

**1. Müşterinin kullandığı iletişim protokolü, sunucu ile eşleşmiyorsa.**
Örneğin, sunucu HTTP iletişim protokolünü kullanıyorsa, müşteri WebSocket iletişim protokolünü kullanarak bağlanmaya çalışıyorsa bağlantı kurulamaz. Sunucu WebSocket protokolünü kullanıyorsa, müşteri de WebSocket protokolünü kullanmalıdır. Sunucu HTTP protokolü hizmeti sağlıyorsa, müşteri HTTP protokolünü kullanarak erişim yapmalıdır.

Bu durumda temel kural, İngilizce konuşurken İngilizce kullanmanız gerektiği gibidir. Japonlarla konuşurken Japonca kullanmanız gerekmektedir. Burada dil, iletişim protokolüne benzer ve her iki tarafın da (müşteri ve sunucu) iletişim için aynı dili kullanması gerekmektedir. Aksi takdirde iletişim kurulamaz.

**Farklı iletişim protokollerine göre yaygın hatalar:**

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: Unexpected response code: xxx

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: net::ERR_INVALID_HTTP_RESPONSE

**Çözüm:**
Yukarıdaki iki hatadan anlaşılacağı gibi, müşteri ws bağlantısı kurarken WebSocket protokolünü kullanıyor. Sunucunun da WebSocket protokolünü kullanması gerekir. Bağlantı bölümünde kod, WebSocket protokolünü belirtmelidir. Aşağıda gatewayWorker'ın dinleme bölümünün kod örneği bulunmaktadır.

```php
// Müşterinin ws://... üzerinden bağlanabilmesi için WebSocket protokolünü kullanın. xxxx portunu değiştirmeniz gerekmez
$gateway = new Gateway('websocket://0.0.0.0:xxxx');
```
Workerman için aşağıdaki gibi olacaktır.

```php
// Müşterinin ws://... üzerinden bağlanabilmesi için WebSocket protokolünü kullanın. xxxx portunu değiştirmeniz gerekmez
$worker = new Worker('websocket://0.0.0.0:xxxx');
```
