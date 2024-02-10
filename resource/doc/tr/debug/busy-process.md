# Meşgul Süreçleri Hata Ayıklama
Bazen ```php start.php status``` komutunu kullanarak ```meşgul (busy)``` durumdaki süreçleri görebiliriz, bu durum ilgili sürecin işlemlerle meşgul olduğunu gösterir. Normal şartlarda işlemler tamamlandığında ilgili süreç ```boşta (idle)``` durumuna dönecektir, genellikle bu durumda herhangi bir sorun olmaz. Ancak sürekli meşgul durumda ve hiçbir zaman ```boşta (idle)``` duruma dönmemişse, bu durum süreç içindeki işlemlerin engellendiğini veya sonsuz döngüde olduğunu gösterir. Bu durumu tespit etmek için aşağıdaki yöntemleri kullanabiliriz.

## strace+lsof Komutunu Kullanarak Konum Belirleme

**1. Meşgul süreçlerin PID'sini status'ta bulma**
```php start.php status``` komutunu çalıştırdıktan sonra aşağıdaki gibi bir çıktı alabiliriz:
![](../images/d1903ed65ef2f3b0850e84ccbedc52aa.png)
Görseldeki ```meşgul``` durumundaki süreçlerin ```pid``` değerleri ```11725``` ve ```11748```'dir.

**2. strace ile süreci izleme**
Seçilen bir sürecin ```pid``` değerini seçerek (burada ```11725``` seçildi) ```strace -ttp 11725``` komutunu çalıştırarak aşağıdaki gibi bir çıktı alabiliriz:
![](../images/7ce9f36da926f670949609dcdc593ab4.png)
Bu çıktıda sürecin ```poll([{fd=16, events=....``` sistem çağrısını sürekli bir döngü içinde yaptığını görebiliriz, bu durum fd değeri 16 olan tanımlayıcının okuma olayını beklediği anlamına gelir. Yani bu tanımlayıcıdan veri döndürülmesi bekleniyor.

Eğer hiçbir sistem çağrısı görüntülenmiyorsa, mevcut terminali kapatın ve yeni bir terminal açın, ardından ```kill -SIGALRM 11725``` komutunu çalıştırarak (sürece bir alarm sinyali göndererek) strace terminalinde bir yanıt alıp almadığını veya herhangi bir sistem çağrısında takılıp takılmadığını kontrol edin. Eğer hala hiçbir sistem çağrısı görüntülenmiyorsa, muhtemelen sürecin işletme içinde sonsuz döngüde olduğu anlamına gelir, bu durumun çözümü için alttaki "Uzun Süre Meşgul Olmasına Neden Olan Diğer Nedenler" bölümündeki 2. maddeye bakın.

Eğer sistem epoll_wait veya select sistem çağrısında takılı kaldıysa bu normal bir durumdur, bu durum sürecin zaten ```boşta (idle)``` durumda olduğunu gösterir.

**3. lsof ile süreç tanımlayıcılarını görüntüleme**
```lsof -nPp 11725``` komutunu çalıştırarak aşağıdaki gibi bir çıktı alabiliriz:
![](../images/27bd629c3a1ac93f9f4b535d01df2ac1.png)
Tanımlayıcı 16'yı belirten veri (en alt satır), fd=16 olan tanımlayıcının uzak adresi ```101.37.136.135:80``` olan bir tcp bağlantısı olduğunu gösterir, bu durum sürecin bir http kaynağına erişmeye çalıştığını ve sürekli ```poll([{fd=16, events=....``` döngüsünde olduğunu açıklar, bu da sürecin neden ```meşgul (busy)``` durumda olduğunu gösterir.

**Çözüm:**
Sürecin hangi noktada takıldığını bildiğimizde, sorunu çözmek kolay olacaktır. Yukarıdaki tespitler sonucunda, örneğin curl kullanımı sırasında ilgili URL uzun süre yanıt vermediğinden dolayı sürecin sürekli beklediğini tespit ettiyseniz, bu durumda URL sağlayıcısından URL'nin neden uzun süre yanıt vermediğini öğrenmeli ve aynı zamanda curl kullanırken bir zaman aşımı parametresi eklemelisiniz, bu şekilde 2 saniye içinde yanıt alamazsa zaman aşımına uğrar ve uzun süreli takılmaları önlersiniz (bu durumda süreç muhtemelen 2 saniyelik bir ```meşgul``` duruma girecektir).

## Süreci Uzun Süre Meşgul Eden Diğer Nedenler
Sürecin ```meşgul (busy)``` durumda olmasına neden olan, sürekli döngüde veya bloklanma gibi durumların yanı sıra aşağıdaki nedenler de olabilir.

**1. İşlemdeki kritik hatalar nedeniyle sürecin sürekli olarak sonlanması**
**Belirti:** Bu durumda sistem yükünün oldukça yüksek olduğu, ```status``` bölümündeki ```load average```'ın 1 veya daha yüksek olduğu görülebilir. Sürecin ```exit_count``` sayısının oldukça yüksek ve sürekli arttığı gözlemlenebilir.
**Çözüm:** Hata ayıklama modunda (`php start.php start` komutunu kullanarak -d olmadan çalıştırarak) Workerman'ı çalıştırarak işlemdeki hataları incelemek ve gidermek gerekmektedir.

**2. Kod içinde sonsuz döngüler**
**Belirti:** ```top``` komutu kullanılarak meşgul sürecin CPU kullanımının yüksek olduğu görülebilir, ```strace -ttp pid``` komutu hiçbir sistem çağrısı bilgisi yazdırmıyorsa.
**Çözüm:** Bu durum için Rasmus Lerdorf'un yazısı olan [gdb ve php kaynak kodu belirleme](https://www.laruence.com/2011/12/06/2381.html) makalesine bakabilirsiniz, temel adımlar şunlardır:
1. ```php -v``` komutunu kullanarak sürümü kontrol edin
2. [İlgili php sürümünün kaynak kodunu indirin](https://www.php.net/releases/)
3. ```gdb --pid=meşgul sürecin pid'si``` komutunu kullanarak debugger'ı başlatın
4. ```source php kaynak kodu yolu/.gdbinit``` komutunu kullanarak kaynak kodu ekleyin
5. ```zbacktrace``` komutunu kullanarak çağrı yığını çıktısını yazdırın
Bu adımlardan sonra php kodunun hangi bölümünde sonsuz döngüde olduğunu görebilirsiniz.
Not: Eğer `zbacktrace` çağrı yığınını yazdırmıyorsa, derlerken `-g` parametresini kullanmadığınız anlamına gelir, bu durumda php'yi yeniden derlemeniz ve ardından Workerman'ı yeniden başlatmanız gerekecektir.

**3. Sonsuz sayıda zamanlayıcı ekleme**
İşlem kodu sürekli olarak zamanlayıcı ekliyor ancak silmiyorsa, sürecin içindeki zamanlayıcılar giderek artar ve sonunda süreci sonsuz zamanlayıcı çalıştırarak durmaksızın çalışır hale getirir. Örneğin:
```php
$worker = new Worker;
$worker->onConnect = function($con){
    Timer::add(10, function(){});
};
Worker::runAll();
```
Yukarıdaki kod, bir istemci bağlandığında bir zamanlayıcı ekler ancak kod içerisinde zamanlayıcıyı silmediği için zamanla süreç içindeki zamanlayıcılar sürekli artar, bu da süreci durmaksızın çalışır hale getirir. Doğru kodlama örneği ise şu şekildedir:
```php
$worker = new Worker;
$worker->onConnect = function($con){
    $con->timer_id = Timer::add(10, function(){});
};
$worker->onClose = function($con){
    Timer::del($con->timer_id);
};
Worker::runAll();
```
Bu kod, istemci bağlandığında bir zamanlayıcı ekler ve kod içerisinde zamanlayıcıyı silerek süreç içindeki zamanlayıcıları kontrol altında tutar.
