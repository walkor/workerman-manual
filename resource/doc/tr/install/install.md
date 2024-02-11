# Kurulum Talimatları
Workerman aslında bir PHP kod paketidir, PHP ortamınız kurulu ise Workerman kaynak kodunu veya demo'yu indirip çalıştırabilirsiniz.

**Composer ile Kurulum:**
```sh
composer require workerman/workerman
```

> **Not**
> Bazı composer aynaları eksik olabilir, yukarıdaki komutu kullanırken `composer config -g --unset repos.packagist` komutunu kullanarak aynaları kaldırın.

# Windows Kullanıcıları (Okunmalı)
Workerman 3.5.3 sürümünden itibaren Workerman artık hem Windows hem de Linux sistemlerini desteklemektedir. Windows kullanıcılarının PHP ortam değişkenlerini yapılandırması gerekmektedir.

 `=== Bu sayfa aşağıdaki sadece Linux ortamlarda çalışan workerman için geçerlidir, Windows kullanıcıları lütfen yok sayın ===`

# Linux Sistem Ortam Testi
Linux sistemleri için aşağıdaki betiği kullanarak yerel PHP ortamının Workerman çalışma gereksinimlerini karşılayıp karşılamadığını test edebilirsiniz.
 `curl -Ss https://www.workerman.net/check | php`

Yukarıdaki komut tümünü "ok" olarak görüntülerse, Workerman gereksinimleri karşılandı anlamına gelir, doğrudan [resmi web sitesinden](https://www.workerman.net/) örnekleri indirebilir ve çalıştırabilirsiniz.

Eğer tümü "ok" olarak görüntülenmiyorsa, eksik olan uzantıları aşağıdaki belgeden yararlanarak kurabilirsiniz.

(Not: Bu test betiği event uzantısını test etmez, eğer iş yükü 1024'ten fazla eşzamanlı bağlantı gerektiriyorsa, event uzantısını ve [Linux kernelini](../appendices/kernel-optimization.md) optimize etmelisiniz, uzantı kurulumu için aşağıdaki açıklamalara bakın)

# Mevcut PHP Ortamına Eksik Uzantıları Kurma

## pcntl ve posix uzantılarını kurma:

**Centos sistemleri**
Eğer PHP yum ile kurulduysa, aşağıdaki komutu çalıştırarak pcntl ve posix uzantılarını kurabilirsiniz: 
```sh
yum install php-process
```

Eğer kurulum başarısız olursa veya PHP başka bir yöntemle kurulmuşsa, [ek uzantıları kurma](../appendices/install-extension.md) bölümünde bulunan üçüncü yöntemi kullanabilirsiniz.

**Debian/Ubuntu/Mac OS sistemleri**
[Ek uzantıları kurma](../appendices/install-extension.md) bölümünde bulunan üçüncü yöntemi kullanabilirsiniz.

## event uzantısını kurma:
Daha fazla eşzamanlı bağlantıyı desteklemek için event uzantısını kurmanız gerekmektedir ve [Linux kernelini](../appendices/kernel-optimization.md) optimize etmelisiniz. Kurulum adımları aşağıdaki gibidir:

**Centos sistemleri**

1. libevent-devel paketini kurmak için aşağıdaki komutu çalıştırın:
```shell
yum install libevent-devel -y
# Kurulum yapamıyorsanız aşağıdaki komutu deneyin
# yum install libevent2-devel -y
```

2. Event uzantısını kurmak için aşağıdaki komutu çalıştırın (event uzantısı PHP>=5.4 gerektirir):
```shell
pecl install event
```
Not: `Include libevent OpenSSL support [yes] :` sorusuna "no" giriniz ve devamını boş bırakınız.

3. `php --ini` komutunu çalıştırarak php.ini dosyasını bulun ve aşağıdaki yapılandırmayı en sondaki satıra ekleyin:
```shell
extension=event.so
```

**Debian/Ubuntu sistemleri**

1. libevent-dev paketini kurmak için aşağıdaki komutu çalıştırın:
```shell
apt-get install libevent-dev -y
# Kurulum yapamıyorsanız aşağıdaki komutu deneyin
# apt-get install libevent2-dev -y
```

2. Event uzantısını kurmak için aşağıdaki komutu çalıştırın:
```shell
pecl install event
```
Not: `Include libevent OpenSSL support [yes] :` sorusuna "no" giriniz ve devamını boş bırakınız.

3. `php --ini` komutunu çalıştırarak php.ini dosyasını bulun ve aşağıdaki yapılandırmayı en sondaki satıra ekleyin:
```shell
extension=event.so
```

**Mac OS sistemleri için kurulum kılavuzu**

Mac işletim sistemi genellikle geliştirme için kullanılır, o yüzden event uzantısını kurmanıza gerek yoktur.

# Yeni bir Sistem Kurulumu (PHP+Uzantıları ile)

## Centos sistemleri için kurulum kılavuzu

1. Aşağıdaki komutu çalıştırarak (bu adım, php-cli ana programı, pcntl, posix, libevent kütüphanesi ve git programını kurmaktadır):
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```

2. Event uzantısını kurmak için aşağıdaki komutu çalıştırın (event uzantısı PHP>=5.4 gerektirir):
```shell
pecl install event
```
Not: `Include libevent OpenSSL support [yes] :` sorusuna "no" giriniz ve devamını boş bırakınız.

3. `php --ini` komutunu çalıştırarak php.ini dosyasını bulun ve aşağıdaki yapılandırmayı en sondaki satıra ekleyin:
```shell
extension=event.so
```

4. Aşağıdaki komutu çalıştırarak (bu adım Workerman ana programını github üzerinden indirir):
```shell
git clone https://github.com/walkor/Workerman
```

5. [Getting Started - Simple Example](../getting-started/simple-example.md) bölümünü referans alarak giriş dosyasını oluşturun ve çalıştırın. 
Veya [resmi web sitesinden](https://www.workerman.net/) demo dosyasını indirip çalıştırabilirsiniz.

## Debian/Ubuntu sistemleri için kurulum kılavuzu

1. Aşağıdaki komutu çalıştırarak (bu adım, php-cli ana programı, libevent kütüphanesi ve git programını kurmaktadır):
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2. Event uzantısını kurmak için aşağıdaki komutu çalıştırın (event uzantısı PHP>=5.4 gerektirir):
```shell
pecl install event
```
Not: `Include libevent OpenSSL support [yes] :` sorusuna "no" giriniz ve devamını boş bırakınız.

3. `php --ini` komutunu çalıştırarak php.ini dosyasını bulun ve aşağıdaki yapılandırmayı en sondaki satıra ekleyin:
```shell
extension=event.so
```

4. Aşağıdaki komutu çalıştırarak (bu adım Workerman ana programını github üzerinden indirir):
```shell
git clone https://github.com/walkor/Workerman
```

5. [Getting Started - Simple Example](../getting-started/simple-example.md) bölümünü referans alarak giriş dosyasını oluşturun ve çalıştırın. 
Veya [resmi web sitesinden](https://www.workerman.net/) demo dosyasını indirip çalıştırabilirsiniz.

## Mac OS sistemleri için kurulum kılavuzu
**Yöntem 1:** Mac sistemi PHP Cli'yi varsayılan olarak içerir, ancak ```pcntl``` uzantısını eksik olabilir.

1. [Ek uzantıları kurma](../appendices/install-extension.md) bölümünde üçüncü yöntemi kullanarak ```pcntl``` uzantısını kurun.

**Yöntem 2:** ```brew``` komutu ile php ve uygun uzantıları kurabilirsiniz

1. Aşağıdaki komutu çalıştırarak ```brew``` aracını kurun (zaten kurulu ise bu adımı atlayabilirsiniz):
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. Aşağıdaki komutu çalıştırarak ```php```'yi kurun:
```shell
brew install php
```

3. Aşağıdaki komutu çalıştırarak ```event``` uzantısını kurun:
```shell
brew install php-event    
```

4. [Resmi web sitesinden](https://www.workerman.net/) demo dosyasını indirip çalıştırabilirsiniz.

# Event Uzantısı Açıklaması
[Event uzantısı](https://php.net/manual/zh/book.event.php) zorunlu değildir, ancak iş yükü 1000'den fazla eşzamanlı bağlantı gerektirdiğinde, Event uzantısını kurmanızı ve devasa eşzamanlı bağlantıları destekleyebilmenizi öneririz. Eğer iş yükü düşük eşzamanlı bağlantılar gerektiriyorsa, örneğin 1000'den az eşzamanlı bağlantı, o zaman kurmanız gerekmez.

## Sık Sorulan Sorular
1. Eğer aşağıdaki hatayı alırsanız `checking for include/event2/event.h... not found`, öncelikle libevent-dev(el) kütüphanesini kaldırın ve libevent2-dev(el) kütüphanesini kurun.
Centos sistemleri: yum remove libevent-devel && yum install libevent2-devel
Debian/Ubuntu sistemleri: apt-get remove libevent-dev && apt-get install libevent2-dev

2. Eğer aşağıdaki hatayı alırsanız `NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`.
Lütfen event.so ve socket.so yüklenme sırasını değiştirin, yani socket uzantısını önce yükleyerek php.ini dosyasında `extension=event.so` satırını `extension=socket.so` satırından önce ekleyin.
