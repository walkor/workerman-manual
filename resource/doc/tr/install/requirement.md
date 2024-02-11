# Ortam Gereksinimleri

## Windows Kullanıcıları
Workerman, 3.5.3 sürümünden itibaren hem Linux hem de Windows sistemlerini desteklemektedir.

1. PHP'nin >=5.4 sürümü gereklidir ve PHP ortam değişkenleri doğru bir şekilde yapılandırılmalıdır.

2. Windows sürümü için Workerman'ın herhangi bir uzantıya ihtiyacı yoktur.

3. Kurulum, kullanım ve kullanım sınırları için [**buraya**](https://www.workerman.net/windows) bakınız.

4. Workerman'ın Windows altında bazı kullanım sınırlamaları nedeniyle, canlı ortamlarda Linux sistemi önerilirken, Windows sistemi sadece geliştirme ortamı için önerilir.

``` ====Bu sayfa yalnızca Linux kullanıcıları için geçerlidir, Windows kullanıcıları lütfen dikkate almayınız. ====```

## Linux Kullanıcıları (Mac OS dahil)
Linux kullanıcıları sadece Linux sürümüne sahip olan Workerman'ı kullanabilir.

1. PHP'nin >=5.4 sürümü yüklü olmalıdır ve pcntl ve posix uzantıları da yüklü olmalıdır.

2. Event uzantısını yüklemeniz önerilir ancak zorunlu değildir (Event uzantısının PHP'nin >=5.4 sürümü gerektirdiğine dikkat edin.)

### Linux Ortamı Kontrol Scripti
Linux kullanıcıları yerel ortamın Workerman gereksinimlerini karşılayıp karşılamadığını kontrol etmek için aşağıdaki komut dosyasını çalıştırabilirler.

```curl -Ss https://www.workerman.net/check | php```

Eğer script tarafından tümü "ok" olarak bildirilirse, Workerman çalışma ortamınızın gereksinimleri karşıladığı anlamına gelir.

(Not: Kontrol scripti event uzantısını kontrol etmez, eğer eşzamanlı bağlantı sayısı 1024'ten fazlaysa event uzantısını yüklemenizi öneririz. Kurulum yöntemi için bir sonraki bölüme bakınız.)

## Detaylı Açıklama

### PHP-CLI Hakkında

Workerman, [PHP Command Line Interface (PHP-CLI)](https://php.net/manual/zh/features.commandline.php) moduna dayalı olarak çalışır. PHP-CLI, PHP-FPM veya Apache'nin MOD-PHP'si ile bağımsız bir yürütülebilir programdır, aralarında çakışma veya karşılıklı bağımlılık yoktur, tamamen bağımsızdır.

### Workerman'ın Bağımlı Olduğu Uzantılar Hakkında

1. [pcntl Uzantısı](https://cn2.php.net/manual/zh/book.pcntl.php)

pcntl uzantısı, Linux ortamında PHP için önemli bir işlem kontrolü uzantısıdır, Workerman, işlem oluşturma, sinyal kontrolü, zamanlayıcı, işlem durumu izleme gibi özelliklerinden yararlanır. Bu uzantı Windows platformunda desteklenmez.

2. [posix Uzantısı](https://cn2.php.net/manual/zh/book.posix.php)

posix uzantısı, PHP'nin Linux ortamında POSIX standardı tarafından sağlanan arabirimlere erişimini sağlar. Workerman, bu uzantıyı kullanarak demonize etme, kullanıcı grubu kontrolü gibi işlevleri gerçekleştirir. Bu uzantı da Windows platformunda desteklenmez.

3. [Event Uzantısı](https://php.net/manual/zh/book.event.php) veya [libevent Uzantısı](https://cn2.php.net/manual/en/book.libevent.php)

Event uzantısı, PHP'nin Epoll, Kqueue gibi gelişmiş olay işleme mekanizmalarını kullanmasını sağlar, bu da Workerman'ın yüksek eşzamanlı bağlantı durumlarında CPU kullanımını belirgin bir şekilde artırabilir. Yüksek eşzamanlı uzun süreli bağlantı uygulamalarında çok önemlidir. libevent uzantısı (veya event uzantısı) zorunlu değildir, eğer yüklü değilse, varsayılan olarak PHP'nin yerel Seçim olay işleme mekanizması kullanılır.

## Uzantıları Nasıl Yükleyebilirsiniz

Uzantı yükleme yöntemleri için [uzantı yükleme](../appendices/install-extension.md) bölümüne bakınız.
