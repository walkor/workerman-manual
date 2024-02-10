# Terminal kapatıldığında hizmet kapanıyor
**Soru:**

Neden ben terminali kapattığımda Workerman kendi kendine kapanıyor?

**Cevap:**

Workerman'ın iki farklı başlatma modu vardır, hata ayıklama modu ve daemon bekleyen işlem modu.

```php xxx.php start``` komutu hata ayıklama moduna girmenizi sağlar, geliştirme ve hata ayıklama için kullanılır, ancak terminal kapatıldığında Workerman da kapanır.

```php xxx.php start -d``` komutu ise daemon bekleyen işlem moduna girmenizi sağlar, terminal kapatıldığında Workerman etkilenmez.

Workerman'ın terminalden etkilenmemesini istiyorsanız daemon modunu kullanarak başlatmalısınız.
