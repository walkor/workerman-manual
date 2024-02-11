# Giriş

**Workerman, yüksek performanslı PHP uygulama konteyneri**

## Workerman nedir?
Workerman, saf PHP ile geliştirilmiş, açık kaynaklı, yüksek performanslı bir PHP uygulama konteyneridir.

Workerman tekerlek yeniden icat etmek değil, MVC çerçevesi değil, daha alt seviyede ve daha genel amaçlı bir hizmet çerçevesidir. Onunla TCP proxy, VPN proxy, oyun sunucusu, e-posta sunucusu, ftp sunucusu, hatta PHP sürümü redis, php versiyonu veritabanı, php versiyonu nginx, php versiyonu php-fpm vb. geliştirebilirsiniz. Workerman, PHP alanında bir inovasyon olarak adlandırılabilir, geliştiriciyi PHP'nin sadece WEB için sınırlı olduğundan tamamen kurtarır.

Aslında Workerman, bir PHP sürümü nginx gibi, temelde çoklu işlem + Epoll + blok olmayan giriş/çıkış (IO). Her bir Workerman işlemi on binlerce eş zamanlı bağlantıyı sürdürebilir. Kendi başına sürekli olarak bellekte bulunan, Apache, nginx, php-fpm gibi konteynerlere bağımlı olmayan, son derece yüksek performansa sahiptir. Aynı zamanda TCP, UDP, UNIXSOCKET'leri destekler, uzun süreli bağlantıları destekler, Websocket, HTTP, WSS, HTTPS ve diğer özel iletişim protokollerini destekler. Zamanlayıcı, asenkron soket istemcisi, asenkron Redis, asenkron Http, asenkron mesaj kuyruğu gibi birçok yüksek performanslı bileşeni bulunmaktadır.

## Workerman'ın bazı uygulama alanları
Workerman, geleneksel MVC çerçevelerinden farklıdır. Workerman, sadece Web geliştirme için değil, aynı zamanda anlık iletişim, nesnelerin interneti, oyunlar, hizmet yönetimi, diğer sunucular veya ara yazılımlar gibi daha geniş bir uygulama alanına sahiptir, bu da PHP geliştiricilerin bakış açısını büyük oranda artırır. Şu anda bu alanlarda PHP geliştiricileri oldukça nadirdir. Eğer PHP alanında kendi teknik avantajınızı elde etmek, her gün CRUD işlerinden sıkılmak istemiyorsanız veya mimar veya teknik uzman yönelimine gitmek istiyorsanız, Workerman kesinlikle öğrenmeye değer bir çerçeve. Geliştiricilere sadece kullanmayı öğretmekle kalmayıp, kendi açık kaynaklı projelerini geliştirmelerini, becerilerini geliştirmelerini ve etkilerini artırmalarını da tavsiye ederim, örneğin [Beanbun çok işlemli crawling çerçevesi](https://github.com/kiddyuchina/Beanbun) bunun çok iyi bir örneğidir, henüz yeni yayınlandı ve çok sayıda olumlu eleştiri aldı.

Workerman'ın bazı uygulama alanları şunlardır:

1. Anlık iletişim
Örneğin web tabanlı anlık sohbet, anlık mesaj gönderme, WeChat miniprogram, mobil uygulama mesaj gönderme, PC yazılımı mesaj gönderme vb.
[[Örnek workerman-chat sohbet odası](https://www.workerman.net/workerman-chat) 、 [web mesaj gönderme](https://www.workerman.net/web-sender) 、 [workerman-todpole küçük sohbet odası](https://www.workerman.net/workerman-todpole)]

2. Nesnelerin İnterneti
Örneğin Workerman ile yazıcı ile iletişim, mikrodenetleyici ile iletişim, akıllı bileklik, akıllı ev eşyaları, paylaşımlı bisiklet vb.
[Müşteri örneği: EasyLink Cloud, EasyPark Time]

3. Oyun sunucuları
Örneğin kart oyunları, MMORPG oyunları vb. [[Örnek browserquest-php](https://www.workerman.net/browserquest)]

4. HTTP hizmetleri
Yüksek performanslı HTTP API'leri, yüksek performanslı web siteleri. Eğer HTTP ile ilgili bir hizmet veya site istiyorsanız şiddetle [webman](https://github.com/walkor/webman)'ı öneririm.

5. SOA servisleri
Workerman'ı kullanarak mevcut işlev birimlerini bir araya getirerek, dış dünyaya birleşik bir arayüz sağlayarak işletme dokusunu gevşek hale getirme, bakımı kolaylaştırma, yüksek kullanılabilirlik ve esneklik sağlama. [[Örnek workerman-json-rpc](https://github.com/walkor/workerman-jsonrpc)、 [workerman-thrift](https://github.com/walkor/workerman-thrift)]

6. Diğer sunucu yazılımı
Örneğin [GatewayWorker](https://www.workerman.net/doc/gateway-worker), [PHPSocket.IO](https://www.workerman.net/phpsocket_io), [HTTP proxy](https://github.com/walkor/php-http-proxy), [sock5 proxy](https://github.com/walkor/php-socks5), [dağıtık iletişim bileşeni](https://github.com/walkor/Channel), [dağıtık değişken paylaşım bileşeni](https://github.com/walkor/GlobalData), [mesaj kuyruğu](https://github.com/walkor/workerman-queue), DNS sunucusu, Web Sunucusu, CDN sunucusu, FTP sunucusu vb.

7. Bileşenler
Örneğin [asenkron redis](components/workerman-redis.md), [asenkron http istemcisi](components/workerman-http-client.md), [nesnelerin interneti mqtt istemcisi](components/workerman-mqtt.md), mesaj kuyruğu [workerman/redis-queue](components/workerman-redis-queue.md)、 [workerman/stomp](components/workerman-stomp.md)、[workerman/rabbitmq](components/workerman-rabbitmq.md)  ，[dosya izleme bileşeni](components/file-monitor.md), ve diğer üçüncü taraf geliştirilmiş bileşen çerçeveleri vb.

Görünüşe göre geleneksel MVC çerçeveleri yukarıdaki işlevleri gerçekleştirmekte zorlanmaktadır, bu yüzden Workerman'ın doğuş nedeni de budur.

## Workerman felsefesi
Sadelik, kararlılık, yüksek performans, dağıtılmış yapı.

### **Sadelik**
Küçük güzeldir, Workerman çekirdeği son derece basittir, sadece birkaç PHP dosyası ve sadece birkaç arabirim açığa çıkar, öğrenmesi çok basittir. Tüm diğer işlevler bileşenler şeklinde genişletilir.

Workerman, kapsamlı belgelere, otoriter ana sayfaya, canlı bir topluluğa, birkaç bin kişilik QQ grubuna, birçok yüksek performanslı bileşene ve birçok örneğe sahiptir, bu da geliştiricilerin kullanımını daha kolay hale getirir.

### **Kararlılık**
Workerman uzun yıllardır açık kaynağa açık, birçok halka açık şirket tarafından büyük ölçekte kullanılmaktadır, son derece kararlıdır. Bazı sunucular 2 yıldan fazla bir süredir yeniden başlatılmamıştır ve hala hızlı bir şekilde çalışmaktadır. Core dump, bellek sızıntısı, hata bulunmamaktadır.

### **Yüksek performans**
Workerman nedeniyle sürekli olarak hafızada bulunduğundan, Apache/nginx/php-fpm'ye bağımlı değildir, PHP'nin konteynerle iletişiminin yükünü taşımaz, her isteğin başlatılması ve yok edilmesi olmadığından, geleneksel MVC çerçevelerinden onlarca kez daha yüksek performansa sahiptir. PHP 7 altında ab stres testiyle QPS'nin hatta tek başına nginx'ten daha yüksek olabileceği yüksek performansa sahiptir.

### **Dağıtılmış yapı**
Artık tek başına savaşan bir çağ değil, tek bir sunucunun performansı ne kadar güçlü olursa olsun, sınırları vardır, dağıtık çoklu sunucu depolama kraldır. Workerman, doğrudan bir uzun süreli bağlantı dağıtık iletişim çözümü [GatewayWorker çerçevesi](https://doc2.workerman.net) sağlar, sadece basit yapılandırma ve başlatma ile sunucu ekleyebilir, iş kodu hiç değişmez, sistem kapasitesi kat kat artar. Eğer TCP uzun bağlantı uygulaması geliştiriyorsanız direk [GatewayWorker](https://doc2.workerman.net) kullanmanızı öneririm, bu, Workerman'ın bir sarmalama olduğu ve uzun bağlantı uygulamaları için daha zengin bir arabirim ve güçlü dağıtık işleme yeteneği sağladığı anlamına gelir.

## Bu dokümantasyonun kapsamı
Workerman 3.x - 4.x sürümleri

## Windows kullanıcıları (Okunmalı)
Workerman, hem linux hem de windows sistemlerini destekler. Windows sürümü Workerman kendisi **hiçbir uzantıya bağlı değil**, sadece PHP ortam değişkenlerini yapılandırmanız gerekir, **Windows sürümü Workerman kurulumu ve dikkat edilmesi gerekenler için [windows kullanıcıları için önemli](https://www.workerman.net/windows)** bölümüne bakın.

## İstemci
Workerman'ın iletişim protokolü açık ve özelleştirilebilir olduğu için teorik olarak Workerman, herhangi bir platformun istemcisi tarafından herhangi bir iletişim protokolü kullanarak iletişim kurabilir. Kullanıcı, istemci geliştirirken ilgili iletişim protokolüne dayalı olarak sunucu ile iletişim kurabilir.
