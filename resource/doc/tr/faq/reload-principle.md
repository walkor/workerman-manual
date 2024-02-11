# Düzgün Yeniden Başlatma Prensibi
## Düzgün Yeniden Başlatma Nedir?

Düzgün yeniden başlatma, normal yeniden başlatmadan farklıdır. Düzgün yeniden başlatma, (genellikle kısa bağlantı işi anlamına gelir) hizmetleri yeniden başlatarak PHP programını yeniden yükleyerek işletme kodu güncellemesini tamamlamak için kullanıcıları etkilemeden hizmeti yeniden başlatmayı başarabilir.

Düzgün yeniden başlatma genellikle işletme güncellemeleri veya sürüm yayınlama sürecinde uygulanır ve kod yayını nedeniyle geçici hizmet kullanılamazlığına yol açan etkileri önleyebilir.

> **Not**
> Windows işletim sistemi yeniden yükleme işlemini desteklemez.

> **Not**
> Uzun süreli bağlantılar (örneğin websocket) işlemin yeniden başlatılmasıyla bağlantı kopacaktır. Bu durumu çözmek için [gatewayWorker](https://www.workerman.net/doc/gateway-worker) gibi bir mimari kullanabilir ve bir grup işlemi bağlantıları korumak için ayrı tutabilir ve bu işlemlerin [reloadable](../worker/reloadable.md) özelliğini false olarak ayarlayabilirsiniz. İşletme mantığı için başka bir grup işlem başlatılır ve gateway ve işlem süreçlerini birbirleriyle tcp iletişimi yoluyla çağırabilirsiniz. İşletme değişikliği gerektiğinde sadece işçi süreçlerini yeniden başlatmanız yeterlidir.

## Kısıtlamalar
**Not: Sadece on{...} geri aramasında yüklenen dosyalar yeniden başlatıldığında otomatik olarak güncellenir, başlatma betiğinde doğrudan yüklene dosyalar veya sabitlenmiş kodların yeniden yüklenmesi otomatik olarak güncellenmez.**

#### Aşağıdaki kodlar yeniden yükleme sonrası güncellenmeyecektir
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('hi'); // Sabitlenmiş kodlar güncellenmeyi desteklemez
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/your/path/MessageHandler.php'; // Başlatma betiği tarafından doğrudan yüklenen dosyalar güncellenmeyi desteklemez
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // Varsayalım ki MessageHandler sınıfının bir onMessage metodu var
```


#### Aşağıdaki kodlar yeniden yükleme sonrası otomatik olarak güncellenecektir
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart işlem başladıktan sonra tetiklenen geri aramadır
    require_once __DIR__ . '/your/path/MessageHandler.php'; // İşlem başladıktan sonra yüklenen dosyalar yeniden yüklemeyi destekler
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
MessageHandler.php değiştirildiğinde `php start.php reload` komutu çalıştırılır ve MessageHandler.php yeniden belleğe yüklenerek işletme mantığının güncellenme etkisi sağlanır.

> **İpucu**
> Yukarıdaki kodlar sadeleştirme amacıyla `require_once` ifadesini kullandı, eğer projeniz psr4 otomatik yükleme desteği sağlıyorsa `require_once` ifadesini çağırmanıza gerek yoktur.

## Düzgün Yeniden Başlatma Prensibi

Workerman, ana süreç ve alt süreç olmak üzere ikiye ayrılır. Ana süreç alt süreçleri izler, alt süreçler istemci bağlantılarını alır ve bağlantı üzerinden gelen istek verilerini işler ve istemcilere yanıt verir. İşletme kodu güncellendiğinde, aslında sadece alt süreçleri güncellememiz yeterli olacaktır.

Workerman ana süreç, düzgün yeniden başlatma sinyali aldığında, ana süreç bir alt süreç için güvenli çıkış (ilgili süreç mevcut isteği tamamladıktan sonra çıkmak) sinyali gönderir. Bu süreç çıktıktan sonra, ana süreç yeni bir alt süreç oluşturur (bu alt süreç yeni PHP kodlarını yükler), ardından ana süreç başka bir eski sürece durdurma komutu gönderir, bu şekilde bir süreç bir süreç yeniden başlatılır, tüm eski süreçler tamamen değiştirilene kadar bu işleme devam edilir.

Gördüğümüz gibi, düzgün yeniden başlatma aslında eski iş süreçlerinin sırayla çıkmasını ve yeni süreçlerin sırayla oluşturulmasını gerçekleştirir. Müşterileri etkilememek için, bu, süreçlerin kullanıcı ile ilgili durum bilgilerini saklamamasını gerektirir, yani iş süreçlerinin mümkünse durumsuz olması, süreç çıkışı nedeniyle bilgi kaybını önlemek için iyidir.
