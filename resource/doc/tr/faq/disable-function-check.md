# Fonksiyon Engelleme Kontrolü Devre Dışı Bırakma

Bu betiği kullanarak engellenmiş fonksiyonları kontrol edin. Komut satırında `curl -Ss https://www.workerman.net/check | php` komutunu çalıştırın.

Eğer `Function fonksiyon_adı may be disabled. Please check disable_functions in php.ini` uyarısı alırsanız, workerman'ın bağımlı olduğu fonksiyonların devre dışı bırakıldığı anlamına gelir. Workerman'ı doğru şekilde kullanabilmek için php.ini dosyasında devre dışı bırakmayı kaldırmanız gerekir.
Engellemeyi kaldırmak için aşağıdaki iki yöntemden birini seçebilirsiniz.

## Yöntem 1: Betikle Kaldırma

Engellemeyi kaldırmak için `curl -Ss https://www.workerman.net/fix | php` komutunu çalıştırın.

## Yöntem 2: Manuel Kaldırma

**Adımlar şunlardır:**

1. `php --ini` komutunu çalıştırarak php cli'nin kullandığı php.ini dosyasının konumunu bulun.

2. php.ini dosyasını açın ve `disable_functions` öğesini bulun, ilgili fonksiyonun engellemesini kaldırın.

**Bağımlı Fonksiyonlar**
Workerman kullanabilmek için aşağıdaki fonksiyonların engellemesini kaldırmanız gerekir
```plaintext
stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
