# Belirli Süreçlerde Yoğun İstekler

### Olay
Bazen `php start.php status` komutunu kullanarak, isteklerin belirli süreçlerde yoğun bir şekilde işlendiğini görüyoruz ve diğer süreçler tamamen boş durumda.

### Kazanma Mekanizması
Workerman'da birden çok süreç bağlantıyı almak için **varsayılan olarak** **öncelikli** bir şekilde çalışır. Yani istemci bir bağlantı başlattığında, tüm boş süreçler bu bağlantıyı almak için bir şans sahibi olur ve hızlı olan önce alır. Kimin daha hızlı olduğu, işletim sistemi çekirdek zamanlama tarafından belirlenir. İşletim sistemi genellikle en son kullanılan süreci CPU kullanımı yetkisi almak üzere tercih edebilir çünkü CPU kayıtlarında önceki sürecin bağlam bilgisi olabilir, bu da bağlam değişim masrafını azaltabilir. Bu nedenle işletme yeterince hızlı olduğunda veya sıkıştırma sürecinde, bazı süreçlerin bağlantıları işlemesi durumu daha sık görülebilir, çünkü bu strateji sık sık süreç değişikliğini önleyebilir ve performans genellikle en iyisidir, herhangi bir sorun yoktur.

### Dolaşım Mekanizması
Workerman, bağlantı alımını **dolaşım** şekline getirerek `$worker->reusePort = true;` ayarını yaparak değiştirebilir. Dolaşım yöntemiyle, çekirdek bağlantıları neredeyse tüm süreçlere eşit bir şekilde dağıtacaktır, bu sayede tüm süreçler talepleri birlikte işleyecektir.

### Yanılgı
Çoğu geliştirici, tüm süreçlerin isteği işlemesi durumunun daha iyi performans sağlayacağını düşünür, ancak gerçek böyle değil. İş yeterince basit olduğunda, işlemeye katılan süreç sayısı CPU çekirdek sayısına ne kadar yakınsa sunucunun geçiş hızı o kadar yüksek olacaktır. Örneğin, 4 çekirdekli sunucuda, süreç sayısı 4 olarak ayarlandığında, helloworld stres testinde QPS genellikle en yüksektir. İşlemeye dahil olan süreç sayısı çok fazla olursa, işlem bağlam masrafı o kadar büyük olur ve performans aksine kötü olur. Genellikle veritabanı işi olduğunda, süreç sayısı CPU çekirdek sayısının 3 katı ile 6 katı arasında ayarlanırsa performans daha iyi olabilir.
