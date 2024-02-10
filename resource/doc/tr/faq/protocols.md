# WorkerMan hangi protokolleri destekler

WorkerMan, arabirim açısından ```ConnectionInterface``` arabirimini takip eden her türlü protokolü destekler (özel iletişim protokolü bölümüne bakınız).

Geliştiricilerin kullanımını kolaylaştırmak için, WorkerMan HTTP protokolünü, WebSocket protokolünü ve oldukça basit metin protokolünü sağlar, ayrıca ikili iletim için kullanılabilen çerçeve protokolünü sunar. Geliştiriciler bu protokollerini doğrudan kullanabilir ve tekrar geliştirme yapmak zorunda kalmaz. Eğer bu protokoller gereksinimleri karşılamıyorsa, geliştiriciler özel protokollerini uygulamak için özel protokol bölümüne bakabilirler.

Geliştiriciler ayrıca doğrudan tcp veya udp protokolü üzerine de çalışabilir.

Protokol kullanımı örneği
```php
// http protokolü
$worker1 = new Worker('http://0.0.0.0:1221');
// websocket protokolü
$worker2 = new Worker('websocket://0.0.0.0:1222');
// metin protokolü (telnet protokolü)
$worker3 = new Worker('text://0.0.0.0:1223');
// çerçeve protokolü (ikili iletim için kullanılabilir)
$worker3 = new Worker('frame://0.0.0.0:1223');
// doğrudan tcp iletimi
$worker4 = new Worker('tcp://0.0.0.0:1224');
// doğrudan udp iletimi
$worker5 = new Worker('udp://0.0.0.0:1225');
```
