# Başlatma ve Durdurma

Workerman başlatma ve durdurma gibi komutlar tümüyle komut isteminden gerçekleştirilir.

Workerman'ı başlatmak için öncelikle hizmetin hangi bağlantı noktalarını dinleyeceği ve hangi protokolü kullanacağı gibi bilgilerin tanımlandığı bir başlangıç dosyasına ihtiyaç vardır. [Başlangıç Kılavuzu - Basit Geliştirme Örnekleri Bölümü](../getting-started/simple-example.md)ne bakabilirsiniz.

Örneğin, [workerman-chat](https://www.workerman.net/workerman-chat) örneği, başlatma dosyası start.php olarak adlandırılmıştır.

### Başlatma

Hata ayıklama (debug) modunda başlatma

 ```php start.php start```

Daemon (Arkaplan işlemi) modunda başlatma

 ```php start.php start -d```

### Durdurma
 ```php start.php stop```

### Yeniden başlatma
 ```php start.php restart```

### Yumuşak yeniden başlatma
 ```php start.php reload```

### Durumun Görüntülenmesi
 ```php start.php status```

### Bağlantı Durumunun Görüntülenmesi (Workerman sürümü >=3.5.0 gerektirir)
```php start.php connections```

## Debug ve Daemon Modu Farkı

1. Hata ayıklama modunda başlatma, kod içindeki echo, var_dump, print gibi çıktı fonksiyonları doğrudan komut istemine gönderilir.
2. Daemon modunda başlatma, kod içindeki echo, var_dump, print gibi çıktılar varsayılan olarak /dev/null dosyasına yönlendirilir ve ```Worker::$stdoutFile = '/sizin/yolunuz/dosya';``` şeklinde belirtilen bir dosya yoluna ayarlanabilir.
3. Hata ayıklama modunda başlatma, komut istemi kapatıldığında Workerman'ı da kapatır ve çıkar.
4. Daemon modunda başlatma, komut istemi kapatıldığında Workerman arka planda normal bir şekilde çalışmaya devam eder.

## Yumuşak Yeniden Başlatma Nedir?

[Smooth Reload Prensibi](../faq/reload-principle.md)ne bakınız.
