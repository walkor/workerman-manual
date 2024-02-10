# workerman Geliştiricilerin Bilmesi Gereken Bazı Konular
**1. Windows Ortamı Kısıtlamaları**

Windows sistemlerinde workerman tek bir işlem için sadece 200+ bağlantıyı destekler.
Windows sistemlerinde count parametresiyle birden fazla işlem belirlemek mümkün değildir.
Windows sistemlerinde status, stop, reload, restart gibi komutlar kullanılamaz.
Windows sistemlerinde daemon işlemi yapılamaz, cmd penceresi kapatıldığında hizmet durur.
Windows sistemlerinde aynı anda birden fazla dinleyici dosyası başlatılamaz.
Yukarıdaki kısıtlamalar Linux sistemlerinde bulunmamaktadır, bu nedenle canlı ortamlarda Linux sistemi tercih edilirken, geliştirme ortamında Windows sistemi seçilebilir.

**2. workerman'ın apache veya nginx'e Bağımlılığı Yoktur**

workerman zaten apache/nginx gibi bir kap içermektedir, bu nedenle PHP ortamı uygunsa workerman çalıştırılabilir.

**3. workerman Komut Satırından Başlatılır**

workerman, apache'nin komutla başlatılma şekline benzer şekilde başlatılır (genellikle web alanları workerman'ı kullanamaz). Başlatma ekranı aşağıdaki gibi olacaktır:
![](image/screenshot_1495622774534.png)

**4. Uzun Bağlantılar Heartbeat İle Desteklenmelidir**

Uzun bağlantılar mutlaka heartbeat ile desteklenmelidir, bunu üç kez vurgulamak önemlidir.
Uzun süreli iletişim olmadığında yönlendirici düğümünün bağlantıyı kapatması nedeniyle bağlantılar kapatılabilir.
[workerman heartbeat açıklaması](faq/heartbeat.md), [gatewayWorker heartbeat açıklaması](https://www.workerman.net/doc/gateway-worker/heartbeat.html)

**5. İstemci ve Sunucu Protokolleri Kesinlikle Eşleşmelidir**

Bu, geliştiricilerin sıkça karşılaştığı bir problemdir. Örneğin, istemci websocket protokolü kullanıyorsa, sunucunun da websocket protokolünü kullanması gerekir (sunucu `new Worker('websocket://0.0.0.0...')` şeklinde).
Tarayıcı adres çubuğunda websocket protokolü bağlantı noktasına gitmeyi denemeyin, raw tcp protokolünü websocket protokolüyle denemeyin, protokol mutlaka eşleşmelidir.

Buradaki prensip, İngilizce konuşmak istiyorsanız İngilizce kullanmanız gerektiği gibidir. Japonlarla iletişim kurmak istiyorsanız, Japonca kullanmanız gerekmektedir. İletişim protokolleri buradaki dil benzeri, taraflar (istemci ve sunucu) iletişim kurabilmek için aynı dilin kullanılmasını gerektirir, aksi takdirde iletişim mümkün olmaz.

**6. Bağlantı Başarısız Olabilir**

workerman'ı yeni kullanmaya başlarken, istemcinin sunucuya bağlanamama sorunu oldukça yaygındır. Genellikle sebepler şunlardır:
1. Sunucu güvenlik duvarı (bulut sunucu güvenlik grubu dahil) bağlantıyı engelledi (bu durumun %50'si).
2. İstemci ve sunucu tarafında kullanılan protokoller eşleşmiyor (%30'luk bir olasılık).
3. IP veya port yanlış yazılmış (%15'lik bir olasılık).
4. Sunucu başlatılmamıştır.

**7. exit die sleep Komutlarını Kullanmayın**

İşletme exit die komutları, işlem çıkışına ve WORKER EXIT UNEXPECTED hatasının görüntülenmesine neden olur. Tabii ki, işlem çıktığında hemen yeni bir işlem başlatılır. Geri dönmek gerekiyorsa, return komutu çağrılmalıdır. sleep komutu işlemi uyutur, uyku sırasında herhangi bir işlem gerçekleştirilmez, çerçeve de durdurulur, bu da işlemin tüm istemci isteklerinin işlenememesine neden olur.

**8. pcntl_fork Fonksiyonunu Kullanmayın**

`pcntl_fork`, dinamik olarak yeni işlem oluşturmak için kullanılır, işletme kodunda `pcntl_fork` kullanılması, geri alınamayan yetim işlem sorunlarına yol açabilir, bu da işletmeye istenmeyen etkiler bırakabilir. İşletmede `pcntl_fork` kullanmak, bağlantıları, mesajları, bağlantı kapatmaları, zamanlayıcılar gibi olayların işlenmesini etkileyebilir, öngörülemeyen sorunlara yol açabilir.

**9. İşletme Kodunda Sonsuz Döngü Olmamalı**

İşletme kodunda sonsuz döngü olmamalıdır, aksi takdirde kontrol workerman çerçevesine geri verilmez ve diğer istemci mesajlarını almak mümkün olmaz.

**10. Kod Değişikliklerinden Sonra Yeniden Başlatın**

workerman sürekli bellekte duran bir çerçevedir, kod değişikliklerini etkileyebilmek için workerman'ı yeniden başlatmak gereklidir.

**11. Uzun Bağlantı Uygulamaları İçin GatewayWorker Çerçevesini Kullanın**

Birçok geliştirici workerman'ı anlık iletişim, nesnelerin interneti gibi uzun bağlantı uygulamaları için kullanmak istemektedir, uzun bağlantı uygulamaları için GatewayWorker çerçevesini kullanmak, workerman temelinde özel olarak geliştirilmiştir, uzun bağlantı uygulamalarının arka planında daha basit, daha kullanışlı hale getirir.

**12. Daha Yüksek Eş Zamanlı Bağlantıları Destekler**
Eğer işletme eş zamanlı bağlantı sayısı 1000'den fazla ise, lütfen [linux çekirdeğini optimize etmeyi](appendices/kernel-optimization.md) ve [event eklentisini yüklemeyi](appendices/install-extension.md) unutmayın.
