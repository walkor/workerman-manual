# Dosya İzleme Bileşeni

**Arka Plan:**

Workerman, sürekli bellekte çalışan bir yapıya sahiptir. Sürekli bellek, diskten tekrar tekrar okumayı ve PHP'yi tekrar tekrar yorumlamayı önleyerek en yüksek performansa ulaşmayı sağlar. Bu nedenle iş kodlarını değiştirdikten sonra manuel olarak yeniden yüklemek veya yeniden başlatmak gerekmektedir.

Aynı zamanda workerman, dosya güncellemelerini izleyen bir hizmet sunmaktadır. Bu hizmet, bir dosyanın güncellendiğini tespit ettiğinde otomatik olarak yeniden yükleme işlemini gerçekleştirir ve PHP dosyalarını yeniden yükler. Geliştiriciler, bunu projeye ekleyerek proje başlatıldığında kullanabilirler.

**Dosya İzleme Servis İndirme Bağlantısı:**

1. Bağımlılığı Olmayan Sürüm: https://github.com/walkor/workerman-filemonitor
2. inotify Bağımlılıklı Sürüm: https://github.com/walkor/workerman-filemonitor-inotify (inotify uzantısının [kurulması gerekmektedir](https://php.net/manual/zh/book.inotify.php))

**İki Sürüm Arasındaki Farklar:**

Adres 1 sürümü, bir dosyanın güncellendiğini tespit etmek için dosyanın güncellenme zamanını her saniye sorgulayan bir yöntem kullanırken,
Adres 2, Linux çekirdeğinin [inotify](https://baike.baidu.com/view/2645027.htm) mekanizmasını kullanan bir versiyondur. Dosya güncellendiğinde sistem workerman'a aktif bildirimde bulunur.

Genellikle bağımlılık gerektirmeyen ilk sürüm kullanılabilir.

**Kullanım Yöntemi**

Applications/FileMonitor dizinini projenizin Applications dizinine kopyalayın.

Eğer projenizde Applications dizini yoksa, Applications/FileMonitor/start.php dosyasını projenizin istediğiniz herhangi bir yerine kopyalayın ve başlatma betiğine bu dosyayı dahil edin.

İzleme bileşeni varsayılan olarak Applications dizinini izler, ancak bu değiştirilebilir. Applications/FileMonitor/start.php dosyasındaki ```$monitor_dir``` değişkenini değiştirerek izlemek istediğiniz dizini tayin edebilirsiniz. ```$monitor_dir``` değişkeni için kesin dosya yolunu kullanmanızı öneririz.

**Notlar:**

* Windows sistemleri reload'u desteklemediğinden bu izleme servisini kullanamaz.
* Yalnızca hata ayıklama modunda etkilidir, daemon modunda dosya izlemesi gerçekleşmez (daemon modunu neden desteklemediğimiz aşağıdaki açıklamada mevcuttur).
* Sadece Worker::runAll çalıştıktan sonra yüklenen dosyaları güncelleyebilirsiniz veya başka bir deyişle, onXXX geri çağrılarında yüklenen dosyaları güncelleyebilirsiniz.

**Neden daemon modunu desteklemiyoruz?**

Daemon modu genellikle canlı üretim ortamında çalışan bir moddur. Genellikle canlı ortamda birden çok dosya tek seferde yayınlanır ve dosyalar arasında bağımlılıklar olabilir. Çoklu dosyanın diske senkronize edilmesi belli bir süre alacağından, belirli bir zamanda diskteki dosyaların eksik olma durumu olabilir. Eğer bu durumda dosya güncellemesi algılanır ve reload işlemi gerçekleştirilirse, dosya bulunamama nedeniyle ciddi hatalara yol açabilir.

Ayrıca canlı ortamda bazen online hata ayıklama yapılabilir, eğer kodu düzenleyip kaydettiğinizde hemen etkili olursa, hemen servisin kullanılamaz hale gelmesine neden olabilir. Doğru yöntem kodu kaydettikten sonra ```php -l yourfile.php``` komutu ile dilbilgisi hatalarını kontrol etmek ve ardından reload ile kodu güncellemektir.

Eğer geliştirici gerçekten daemon modunda dosya izleme ve otomatik güncelleme yapmak istiyorsa, Applications/FileMonitor/start.php dosyasındaki Worker::$daemonize kısmındaki koşulu kaldırarak kendisi değiştirebilir.
