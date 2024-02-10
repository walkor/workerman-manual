# WorkerMan Özellikleri

### 1. Saf PHP Geliştirme
WorkerMan ile geliştirilen uygulamalar, php-fpm, apache, nginx gibi konteynerlara bağlı olmadan bağımsız olarak çalışabilir. Bu durum, PHP geliştiricilerinin uygulama geliştirmesini, dağıtmasını ve hata ayıklamasını son derece kolaylaştırır.

### 2. PHP Çok İşlem Desteği
Sunucunun çoklu CPU performansını tam olarak kullanmak için, WorkerMan varsayılan olarak çoklu işlemi destekler. WorkerMan, bir ana işlem ve birden fazla alt işlemi dış dünyaya hizmet vermek üzere başlatır. Ana işlem alt işlemleri izlerken, alt işlemler ağ bağlantılarını dinler, veri alır, işler ve gönderir. Basit işlem modeli sayesinde WorkerMan daha istikrarlı ve verimli hale gelir.

### 3. TCP ve UDP Desteği
WorkerMan, TCP ve UDP olmak üzere iki tür iletişim katmanı protokolünü destekler. Protokolün değiştirilmesi istendiğinde, sadece bir özelliği değiştirmek yeterlidir. İş kodu değişikliği gerektirmez.

### 4. Uzun Süreli Bağlantı Desteği
Birçok durumda, PHP uygulamasının istemciyle uzun süreli bağlantı kurması gerekebilir. Örneğin, sohbet odaları, oyunlar vb. ancak geleneksel PHP konteynerleri (apache, nginx, php-fpm) bu durumu zorlaştırır. WorkerMan ile, sunucu işlemi bağlantıyı etkin bir şekilde sonlandırmadığı sürece PHP uzun süreli bağlantıları kullanılabilir. WorkerMan tek bir işlemde on binlerce eşzamanlı bağlantıyı destekleyebilirken, çoklu işlem on binlerce hatta milyonlarca eşzamanlı bağlantıyı destekler.

### 5. Çeşitli Uygulama Katmanı Protokollerini Destekleme
WorkerMan arayüzü çeşitli uygulama katmanı protokollerini, özelleştirilmiş protokolleri destekler. WorkerMan'de protokol değiştirme oldukça basittir, sadece bir alanı yapılandırmak yeterlidir, protokol otomatik olarak değiştirilir, iş kodunu değiştirmek gerekmez, hatta farklı protokoller için birden fazla bağlantı noktası açılabilir, farklı istemci gereksinimlerini karşılamak için.

### 6. Yüksek Eşzamanlılık Desteği
WorkerMan, Libevent olay atama kitaplığını destekler (event uzantısı kurulu olması gerekir). Uzun süreli bağlantı ve yüksek eşzamanlılık durumunda Event kullanımı son derece olağanüstü performansı sağlar. Event uzantısı kurulu değilse, PHP'nin yerleşik Select ilgili sistem çağrılarını kullanır, performansı yine de oldukça güçlüdür.

### 7. Yumuşak Servis Yeniden Başlatma Desteği
Servisin yeniden başlatılması gerektiğinde (örneğin sürüm yayınlama), kullanıcı isteklerini işleyen işlemlerin hemen sonlandırılmasını istemeyiz, hatta yeniden başlatma anında istemci iletişiminde başarısızlık meydana gelmesini istemeyiz. WorkerMan, yumuşak yeniden başlatma fonksiyonu sağlar, hizmeti pürüzsüz bir şekilde yükseltmeyi, istemcilerin kullanımını etkilememeyi garanti eder.

### 8. Dosya Güncelleme Kontrol ve Otomatik Yükleme Desteği
Geliştirme sürecinde, kod değişikliklerinin hemen etkili olmasını istiyoruz. WorkerMan, [FileMonitor dosya izleme bileşenini](../components/file-monitor.md) sunar, dosya güncellendiğinde, WorkerMan otomatik olarak yeniden yüklenir, yeni dosyaları yüklemek için gereklidir.

### 9. Belirli Kullanıcı İle İşleme Alt İşlem Çalıştırma Desteği
Alt işlem, kullanıcı isteklerini gerçekten işleyen işlemdir, güvenlik endişeleri nedeniyle, alt işlemin çok yüksek bir izne sahip olmaması gerekir, bu nedenle WorkerMan, alt işlem çalıştırma iznini belirli bir kullanıcı olarak ayarlamanıza olanak tanır, sunucunuzu daha güvenli hale getirir.

### 10. Nesne veya Kaynağı Kalıcı Tutma Desteği
WorkerMan çalışırken, PHP dosyasını bir kez okuyup ayrıştırır ve sonra bellekte sürekli olarak kalır, bu durum sınıfların ve fonksiyonların beyanları, PHP çalışma ortamı, sembol tablosu vb. tekrar tekrar oluşturulup yok edilmez. Bu, Web konteynerinde çalışan PHP mekanizmasından tamamen farklıdır. WorkerMan'de, bir işlem süresi boyunca statik üyeler veya global değişkenler, etkin bir şekilde kullanılabilir. Örneğin, bir işlem içinde bir kere veritabanı bağlantısı başlatılırsa, bu işlemdeki tüm istekler bu veritabanı bağlantısını tekrar kullanabilir, bu da sık sık veritabanına bağlantı kurma sürecinden kaynaklanan TCP üçlü el sıkışma, veritabanı izni doğrulama, bağlantıyı kapatma esnasında TCP dörtüz el sıkışma sürecini önler, uygulama verimliliğini büyük ölçüde artırır.

### 11. Yüksek Performans
PHP dosyası diskten okunduktan ve ayrıştırıldıktan sonra bellekte sürekli olarak kalır, bir sonraki kullanımda doğrudan bellekteki opcode'ı kullanır, bu disk IO'sunu ve PHP'de isteğin başlatılmasını, yürütme ortamının oluşturulması, leksikal analiz, sözdizimi analizi, opcode derleme, isteğin kapatılması ve diğer zaman alıcı süreçleri büyük ölçüde azaltır, ayrıca nginx, apache gibi konteynerlara bağımlılığı olmadığı için, nginx, apache gibi konteynerlerle PHP arasındaki iletişim giderlerini azaltır, en önemlisi kaynaklar kalıcı olabilir, her seferinde veritabanı bağlantısı vb. başlatılabilir, bu nedenle WorkerMan ile uygulama geliştirmek, oldukça yüksek performans sunar.

### 12. HHVM Desteği
HHVM sanal makinesinde çalışmaya uygun, PHP performansını katlanarak artırabilir. Özellikle işlemci yoğun işlemlerde performansı oldukça iyidir. Gerçek baskı testleri karşılaştırıldığında, yük olmayan işlerde, WorkerMan'ın HHVM'de çalıştırılması, Zend PHP5.6'da ağın işlem hacmi üzerinde %30-80 daha fazla artış sağlar.

### 13. Dağıtılmış Dağıtım Desteği

### 14. Gözcü İşlem Desteği

### 15. Birden Fazla Bağlantı Noktası Dinleme Desteği

### 16. Standart Giriş Çıkış Yönlendirmesi Desteği
