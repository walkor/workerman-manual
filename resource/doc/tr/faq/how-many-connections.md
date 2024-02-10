# WorkerMan, aşkınları destekler

**Concurrency**

Concurrency kavramı oldukça belirsizdir. Burada ölçülebilir iki gösterge olan **concurrent connection** (eş zamanlı bağlantı) ve **concurrent request** (eş zamanlı istek) ile açıklanmaktadır.

**Concurrent connection** (eş zamanlı bağlantı), sunucunun bir anda kaç TCP bağlantısını sürdürdüğünü ifade eder, bu bağlantılarda ise veri iletişimi olup olmadığı önemli değildir. Örneğin, bir bildirim sunucusu, nadir veri iletişimi olması nedeniyle yüz binlerce cihaz bağlantısını sürdürebilir. Bu durumda sunucunun yükü neredeyse sıfır olabilir, yeterli bellek varsa daha fazla bağlantıyı kabul edebilir.

**Concurrent request** (eş zamanlı istek), genellikle QPS (saniyede işlenen istek sayısı) ile ölçülür. Sunucunun bir anda kaç TCP bağlantısı olduğu çok fazla önemli değildir. Örneğin, bir sunucunun sadece 10 müşteri bağlantısı olsa bile, her bir müşteri bağlantısında saniyede 10.000 istek olsa, sunucu en azından 10*10.000=100.000 isteği karşılayabilmelidir (QPS). Örneğin, 100,000 isteği saniyede karşılama kapasitesi bu sunucunun sınırıysa, her bir müşterinin sunucuya saniyede 1 istek göndermesi durumunda, sunucu 100,000 müşteriyi destekleyebilir.

**Concurrent connection** (eş zamanlı bağlantı), genellikle sunucu hafızasına bağlıdır. Genellikle 24GB belleğe sahip bir Workerman sunucusu yaklaşık olarak **120 milyon** eş zamanlı bağlantıyı destekleyebilir.

**Concurrent request** (eş zamanlı istek), genellikle sunucunun CPU işleme kapasitesine bağlıdır. 24 çekirdekli bir Workerman sunucusu en fazla **450,000** istek/saniye (QPS)’ye ulaşabilir. Aslında bu değer işletme karmaşıklığı ve kod kalitesine göre değişebilir.

## Dikkat

Yüksek hemenlikli senaryolar event eklentisinin kurulu olmasını gerektirir. Kurulum ve konfigürasyon bölümüne bakınız. Ayrıca, Linux çekirdeğini, özellikle de işlem dosya sayısı sınırlamasını optimize etmek gerekmektedir. Bu konuda Kernel Optimization bölümüne bakınız.

## Stres testi verileri 
> Veriler, üçüncü taraf yetkili stres testi kuruluşu techempower.com'un 20. tur stres testinden alınmıştır.
https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf

**Sunucu yapılandırması:**
Toplam Çekirdek sayısı 14, Toplam Thread 28, 32 GB bellek, Cisco tarafından sağlanan 10 gigabit Ethernet anahtarı
**İş mantığı:**
Veritabanı sorgulama iş mantığı, PostgreSQL veri tabanı, PHP8+jit ile
QPS 390 bin üzerinde
![](../images/screenshot_1636522357217.png)
