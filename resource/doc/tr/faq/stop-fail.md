# Başarısız Durdurma

## Belirti:
```php start.php stop``` komutunun `stop fail` hatası vermesi.

### İlk Olasılık
Workerman'ı debug modunda başlatmak, geliştiricinin terminalde `ctrl z` tuşlarına basarak `SIGSTOP` sinyalini göndermesine neden olmuş olabilir. Bu da Workerman'ın arka planda askıya alınmasına ve durdurma komutuna yanıt verememesine yol açmış olabilir (`SIGINT` sinyali).

**Çözüm:**
Workerman'ı başlatan terminalde `fg` komutunu (```SIGCONT``` sinyalini göndermek) ve ardından Enter tuşuna basarak Workerman'ı ön plana çıkararak çalışmasını sağlayın, ardından `ctrl c` tuşlarına basarak (```SIGINT``` sinyali göndererek) Workerman'ı durdurun.

Durdurulamıyorsa, aşağıdaki komutları deneyin
```bash
killall -9 php
```
```bash
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

### İkinci Olasılık
Durdurma işlemini yapacak kullanıcı ve Workerman'ı başlatan kullanıcı farklıysa, yani durduran kullanıcının Workerman'ı durdurma izni yoksa.

**Çözüm:**
Workerman'ı başlatan kullanıcıya geçiş yapın veya daha yüksek izinlere sahip bir kullanıcı ile Workerman'ı durdurun.

### Üçüncü Olasılık
Workerman ana işlem PID dosyası silindiğinde, betik PID sürecini bulamadığından durdurma başarısız olabilir.

**Çözüm:**
PID dosyasını güvenli bir konuma kaydedin, [Worker::$pidFile](../worker/pid-file.md) belgesine bakın.

### Dördüncü Olasılık
Workerman ana işlem PID dosyasına karşılık gelen işlem Workerman işlemi değilse.

**Çözüm:**
Workerman'ın ana işlem PID dosyasını açın ve ana işlem PID'sini kontrol edin, PID dosyası varsayılan olarak Workerman'ın yanındaki dizine kaydedilir. Aşağıdaki komutu çalıştırarak (`ps aux | grep ana işlem PID`) ilgili işlemin Workerman işlemi olup olmadığını kontrol edin. Eğer değilse, muhtemelen sunucu yeniden başlatıldığından dolayı Workerman'ın sakladığı PID eski bir PID'dir ve bu PID başka bir işlem tarafından kullanılıyordur, bu da durdurmayı başarısız yapar. Bu durumda, PID dosyasını silmek yeterli olacaktır.

### Beşinci Olasılık
grpc eklentisi yüklendi, ancak grpc eklentisi için ilgili ortam değişkenleri ayarlanmadıysa, başlatıldığında ek bir montaj süreci oluşturulabilir ve durdurulamayabilir.

**Çözüm:**
grpc eklentisi için ilgili ortam değişkenlerini ayarlayın.
