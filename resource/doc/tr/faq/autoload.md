# Otomatik Yükleme

Projeniz otomatik yükleme uygulamıyorsa, bu işlemi Composer ile oluşturabilirsiniz. Aşağıdaki adımları takip edin.

**1. composer.json Dosyasını Değiştirin**
composer.json dosyasında autoload.psr-4'e `"" : "./"` ekleyin, örneğin
```json
{
    "require": {
        "workerman/workerman" : "^v4.0.0"
    },
    "autoload": {
        "psr-4": {
            "" : "./"
        }
    }
}
```

**2. autoload.php Dosyasını Oluşturun**
`composer dumpautoload` komutunu çalıştırın.

**3. autoload.php Dosyasını Yükleyin** 
Projenizin start.php dosyasının en üst kısmına `vendor/autoload.php` dosyasını yükleyin, örneğin
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. Workerman'ı Yeniden Başlatın**
`php start.php restart`
