# workerman başlatma hatası

## Belirti 1
Başlatıldıktan sonra benzer bir hata alıyorsunuz:
```php
php start.php start
PHP Uyarısı:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php satır xxxx

```
**Anahtar kelimeler**: ```Address already in use```

**Kök neden**: Port kullanımda ve başlatma yapılamıyor.

#### Çözüm 1


```netstat -anp | grep port_number``` komutunu kullanarak hangi programın portu kullandığını bulabilirsiniz.
Ardından ilgili programı durdurarak portun serbest bırakılmasını sağlayabilirsiniz.

#### Çözüm 2
Eğer ilgili portu durduramıyorsanız, workerman'ın portunu değiştirerek sorunu çözebilirsiniz.

#### Çözüm 3
Eğer Workerman'ın kullandığı bir port ise ve stop komutu ile durdurulamıyorsa (genellikle pid dosyasının eksik olmasından veya geliştirici tarafından ana işlemi sonlandırılmasından dolayı), aşağıdaki komutları çalıştırarak Workerman sürecini sonlandırabilirsiniz.

```killall php
ps aux|grep WorkerMan|awk '{print $2}'|xargs kill -9
```

#### Çözüm 4
Eğer gerçekten bu portu dinleyen bir program yoksa, bunun sebebi geliştiricinin workerman içinde aynı portu dinleyen iki veya daha fazla dinleyici ayarlamış olması olabilir. Geliştiricinin başlatma betiğini kontrol etmesi gerekmektedir.

#### Çözüm 5
reusePort'un etkinleştirilip etkinleştirilmediğini kontrol edin, reusePort'u kapatmayı deneyin.


## Belirti 2
Başlatıldıktan sonra benzer bir hata alıyorsunuz:
```php
PHP Uyarısı:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php satır xxxx
```
veya
```
PHP Uyarısı:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Cannot assign requested address) in ...workerman/Worker.php satır xxxx
```
**Anahtar kelimeler**: `Cannot assign requested address` veya `该请求的地址无效`

**Hata sebebi**: Başlatma betiğindeki ip parametresi yanlış girilmiş, yerel ip değil. Lütfen yerel ip adresini veya ```0.0.0.0``` (tüm ip'leri dinle) girerek çözebilirsiniz.

**Not**: Linux sistemlerinde tüm ağ kartlarının ip adreslerini görmek için ```ifconfig``` komutunu kullanabilirsiniz.
Eğer bir bulut sunucusu (Alibaba Cloud, Tencent Cloud vb.) kullanıyorsanız, genel ip adresinizin aslında bir proxy ip olduğunu unutmayın (örneğin, Alibaba Cloud'un özel ağı). Bu nedenle genel ip adresi mevcut sunucuya ait olmadığından genel ip üzerinde dinleme yapılamaz. Yine de ```0.0.0.0``` ile bağlanabilirsiniz.

## Belirti 3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**Hata sebebi**: stream_socket_server fonksiyonu php.ini'de devre dışı bırakıldı.

**Çözüm**: 
1、```php --ini``` komutunu kullanarak php.ini dosyasını bulun

2、php.ini dosyasını açın ve disable_functions bulun, buradan stream_socket_server'ı silin

## Belirti 4
```php
PHP Uyarısı:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```
**Hata sebebi**: Linux'ta 1024'ten küçük bir port dinlemek için root izinlerine ihtiyaç vardır.

**Çözüm**: 1024'ten büyük bir port kullanarak veya root kullanıcısıyla hizmeti başlatarak çözebilirsiniz。
