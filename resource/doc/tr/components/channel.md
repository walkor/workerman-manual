# Kanal Dağıtılmış İletişim Bileşeni
**``` (Workerman sürümü >=3.3.0 gerektirir) ```**

Kaynak kod: https://github.com/walkor/Channel

Channel, işlem arası iletişimi veya sunucu arası iletişimi tamamlamak için kullanılan bir dağıtılmış iletişim bileşenidir.

## Özellikler
1. Abone Ol / Yayınla modeline dayalı
2. Engelleme yok IO

## İlke
Channel, Channel/Server sunucusu ve Channel/Client istemcisi içerir.

Channel/Client, Channel/Server'a bağlanmak için connect interfacesini kullanır ve uzun süreli bağlantıyı korur.

Channel/Client, on interfacesini çağırarak Channel/Server'a hangi olayları takip ettiğini bildirir ve olay geri çağırma işlevlerini kaydeder (geri çağırma, Channel/Client'ın bulunduğu işlemde gerçekleşir).

Channel/Client, publish interfacesini kullanarak bir olayı ve olayla ilgili verileri Channel/Server'a gönderir.

Channel/Server, olay ve verileri aldıktan sonra bu olaya ilgi duyan Channel/Client'lara dağıtır.

Channel/Client, olay ve verileri aldığında on interfacesi tarafından belirlenen geri aramayı tetikler.

Channel/Client, yalnızca takip ettiği olayları alır ve geri çağırır.

## Kurulum
`composer require workerman/channel`

## Not
Channel yalnızca workerman ortamında kullanılabilir, php-fpm ortamında kullanılamaz.
