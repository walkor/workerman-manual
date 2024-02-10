# Çalışma Durumunu Kontrol Etme

```php start.php status``` komutunu çalıştırarak WorkerMan'in çalışma durumunu kontrol edebilirsiniz. Aşağıdaki gibi bir çıktı alacaksınız:

```
----------------------------------------------GLOBAL STATUS----------------------------------------------------
Workerman sürümü: 3.5.13          PHP sürümü: 5.5.9-1ubuntu4.24
başlangıç saati: 2018-02-03 11:48:20   toplam 112 gün 2 saat   
yük ortalaması: 0, 0, 0            olay döngüsü:\Workerman\Events\Event
4 işçi       11 işlem
işçi_adı        çıkış_durumu      çıkış_sayısı
ChatBusinessWorker 0                0
ChatGateway        0                0
Register           0                0
WebServer          0                0
----------------------------------------------PROCESS STATUS---------------------------------------------------
pid	bellek  dinleniyor                işçi_adı        bağlantılar gönderme_hatası zamanlayıcılar  toplam_istek qps    durum
18306	2.25M   yok                     ChatBusinessWorker 5           0         0       11            0      [boşta]
18307	2.25M   yok                     ChatBusinessWorker 5           0         0       8             0      [boşta]
18308	2.25M   yok                     ChatBusinessWorker 5           0         0       3             0      [boşta]
18309	2.25M   yok                     ChatBusinessWorker 5           0         0       14            0      [boşta]
18310	2M      websocket://0.0.0.0:7272 ChatGateway        8           0         1       31            0      [boşta]
18311	2M      websocket://0.0.0.0:7272 ChatGateway        7           0         1       26            0      [boşta]
18312	2M      websocket://0.0.0.0:7272 ChatGateway        6           0         1       21            0      [boşta]
18313	1.75M   websocket://0.0.0.0:7272 ChatGateway        5           0         1       16            0      [boşta]
18314	1.75M   text://0.0.0.0:1236      Register           8           0         0       8             0      [boşta]
18315	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [boşta]
18316	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [boşta]
----------------------------------------------PROCESS STATUS---------------------------------------------------
Özet	18M     -                        -                  54          0         4       138           0      [Özet]
```

## Açıklama

### GLOBAL STATUS

Bu bölümden aşağıdakileri görebiliriz:

WorkerMan sürümü: ```version:3.5.13```

Başlangıç zamanı: ```2018-02-03 11:48:20```, çalışma süresi: ```run 112 gün 2 saat```

Sunucu yükü: ```yük ortalaması: 0, 0, 0``` (sırasıyla son 1 dakika, 5 dakika ve 15 dakika içinde sistem ortalaması)

Kullanılan IO olay döngüsü: ```event-loop:\Workerman\Events\Event```

 ```4 işçi``` (ChatGateway, ChatBusinessWorker, Register, WebServer gibi 4 tür işçi)

 ```11 işlem``` (toplam 11 işlem)

 ``` işçi_adı ``` (işçi süreci adı)

 ``` çıkış_durumu ``` (işçi süreci çıkış durumu kodu)

 ``` çıkış_sayısı ``` (bu çıkış durumuna sahip sürecin çıkış sayısı)


Genellikle, çıkış durumu 0 ise normal çıkışı temsil eder. Diğer bir değer ise sürecin anormal bir şekilde çıktığını ve "WORKER EXIT UNEXPECTED" hatasına neden olduğunu gösterir. Hata mesajı genellikle [Worker::logFile](worker/log-file.md) dosyasına kaydedilir.

**Yaygın çıkış durumları ve anlamları şunlardır:**

* 0: Normal çıkışı temsil eder, yeniden yükleme sırasında 0 değerli çıkış kodu meydana gelebilir, bu normaldir. Unutmayın, işletme kodunun exit veya die ifadesini çağırması da çıkış kodunu 0 olarak meydana getirecek ve "WORKER EXIT UNEXPECTED" hatasına neden olacaktır. Workerman'da, işletme kodunun exit veya die ifadesini çağırmasına izin verilmez.
* 9: Süreç SIGKILL sinyali ile öldürüldü. Bu çıkış kodu, genellikle durdurma ve yeniden yükleme işlemlerinde meydana gelir. Bu çıkış koduna neden olan neden, alt sürecin ana süreç reload sinyaline belirlenen süre içinde yanıt vermemesidir (örneğin, mysql, curl gibi uzun süreli engelleme bekleme veya işletme döngüsü vb.), ana süreç SIGKILL sinyali ile zorla öldürülür. Lütfen dikkat, linux komut satırında kill komutunu kullanarak alt sürece SIGKILL sinyali göndermek de bu çıkış koduna neden olabilir.
* 11: PHP'nin core dump oluştuğunu gösterir, genellikle istikrarsız uzantıların kullanılmasından kaynaklanır, ilgili uzantıyı php.ini dosyasından yorumlayarak devre dışı bırakmalısınız; ayrıca nadir durumlarda bu, PHP'nin hatalı olmasından kaynaklanır, bu durumda PHP'yi güncellemeniz gerekecektir.
* 65280: Bu çıkış koduna neden olan neden işletme kodunda ölümcül bir hata olmasıdır, örneğin, var olmayan bir fonksiyonu çağırma, sözdizimsel hata vb. Özel hata bilgileri [Worker::logFile](worker/log-file.md) dosyasına kaydedilir ve varsa [php.ini](https://php.net/manual/zh/ini.list.php) içinde belirtilen [error_log](https://php.net/manual/zh/errorfunc.configuration.php#ini.error-log) dosyasında bulunabilir.
* 64000: Bu çıkış koduna neden olan neden işletme kodunun istisna fırlatması, ancak işletmenin bu istisnayı yakalamaması ve sürecin çıkmasıdır. Workerman debug modunda çalıştığında, istisna çağrı yığını terminalde yazdırılırken, daemon modunda çalıştığında istisna çağrı yığını [Worker::stdoutFile](worker/stdout-file.md) dosyasına kaydedilir.


## PROCESS STATUS
pid: süreç kimliği

bellek: sürecin şu anki bellek kullanımı (php'nin kendisine ait çalıştırılabilir dosyanın bellek kullanımını içermez)

dinleniyor: taşıma katmanı protokolü ve dinlenen IP bağlantı noktası. Hiçbir bağlantı noktası dinlenmiyorsa "yok" olarak gösterilir. [Worker sınıfının oluşturucu işlevine](worker/construct.md) bakınız.

işçi_adı: sürecin çalıştırdığı hizmet adı, [Worker sınıfı ad özelliğinde](worker/name.md) bahsedilir.

bağlantılar: sürecin **şu anda** kaç tane TCP bağlantı örneği olduğu, bağlantı örnekleri TcpConnection ve AsyncTcpConnection örneklerini içerir. Bu değer anlık bir sayıdır, birikmiş değil. Not: Bağlantı örneği close çağrıldığında, ilgili sayacın azalmadığı durumda, işletme kodunun $connection nesnesini saklamasından dolayı bağlantı örneği yok edilemez.

toplam_istek: sürecin başlatıldığı andan itibaren kaç tane istek aldığını gösterir. Buradaki istek sayısı, sadece istemci tarafından gönderilen istekleri değil, aynı zamanda Workerman'ın iç iletişim isteklerini de içerir, örneğin GatewayWorker mimarisinde Gateway ve BusinessWorker arasındaki iletişim isteklerini içerir. Bu değer birikmiş bir değerdir.

send_fail: sürecin istemcilere veri gönderme başarısızlık sayısı, başarısızlık nedeni genellikle istemcinin bağlantısının kopmasıdır, bu öğe 0 olmazsa genellikle normal bir durumdur, [send_fail durumu hakkında](../faq/about-send-fail.md) sayfasına bakınız. Bu değer birikmiş bir değerdir.

zamanlayıcılar: sürecin etkin zamanlayıcı sayısı (silinmiş zamanlayıcıları ve çalıştırılmış tek kullanımlık zamanlayıcıları içermez). Not: Bu özellik, workerman sürümü >= 3.4.7 gerektirir. Bu değer anlık bir sayıdır, birikmiş değil.


qps: sürecin şu anda her saniye aldığı ağ isteği sayısı, not: yalnızca ```-d``` parametresiyle beraber status komutuyla kullanıldığında bu seçenek hesaplanır, aksi takdirde 0 gösterilir. Bu özellik workerman sürümü >= 3.5.2 gerektirir. Bu değer anlık bir sayıdır, birikmiş değil.

durum: süreç durumu, eğer boşta ise idle, eğer meşgulse busy olarak belirtilir. Not: Süreç kısa süreli olarak meşgul olursa bu normal bir durumdur, süreç sürekli meşgul ise işletme bloğu veya işletme döngüsü gibi durumlar olabilir, [meşgul süreçleri hata ayıklama](busy-process.md) bölümüne bakınız. Not: Bu özellik workerman sürümü >= 3.5.0 gerektirir.


## Prensip

status betiği çalıştırıldığında, ana süreç tüm işçi süreçlerine bir ```SIGUSR2``` sinyali gönderir, ardından status betiği kısa bir uyku süresine girer ve işçi süreçlerinin durum istatistik sonuçlarını bekler. Bu esnada boşta olan işçi süreci, ```SIGUSR2``` sinyali aldığında kendi çalışma durumunu (bağlantı sayısı, istek sayısı gibi) hemen belirli bir diske yazacaktır, işle ilgili iş görüşmelerini bekleyen işçi süreçleriyse işlemlerini tamamladıktan sonra kendi durum bilgilerini yazacaklardır. Kısa uyku sonrasında, status betiği diske yazılmış durum dosyalarını okumaya başlayacak ve sonuçları konsola aktaracaktır.
## Not
Durum değeri bazı durumlarda "meşgul" olarak görünebilir. Bunun nedeni, işlemin işlemlerle meşgul olmasıdır (örneğin, iş mantığı uzun süre boyunca curl veya veritabanı isteği gibi işlemlerle meşgul olması veya büyük bir döngüyü çalıştırması).

Bu tür bir sorunla karşılaşıldığında, iş mantığı kodlarını araştırmak ve hangi kısımların uzun süre boyunca kilitlenmeye neden olduğunu belirlemek gerekir. Ayrıca, kilitlenmenin ne kadar süreyle beklenildiği değerlendirilmelidir. Beklentilerle uyuşmuyorsa, [Meşgul İşlem Hata Ayıklama](busy-process.md) bölümüne göre iş mantığı kodlarını ayıklamak gereklidir.
