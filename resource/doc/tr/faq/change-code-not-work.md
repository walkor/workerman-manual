# Kod değişikliklerinin etkisiz kalması

**Nedeni:**

Workerman, sürekli bellekte çalışan bir yapıya sahiptir. Sürekli bellekte çalışma, tekrar tekrar disk okumasını ve PHP'nin tekrar tekrar yorumlanıp derlenmesini önleyerek en yüksek performansa ulaşmayı sağlar. Bu nedenle iş kodlarını değiştirdikten sonra manuel olarak yeniden yüklemek veya yeniden başlatmak gereklidir.

Aynı zamanda workerman, dosya güncellemelerini izleyen bir hizmet sunar. Bu hizmet, bir dosyanın güncellendiğini algıladığında otomatik olarak yeniden yükleme işlemini gerçekleştirir ve PHP dosyalarını tekrar yükler. Geliştirici, bu hizmeti projeye ekleyerek projeyi başlatır.

Not: Windows sistemlerinde yeniden yükleme desteklenmemektedir ve izleme hizmeti kullanılamaz.

**Dosya izleme servisi indirme bağlantıları:**

1. Bağımsız sürüm: https://github.com/walkor/workerman-filemonitor

2. [inotify](https://baike.baidu.com/view/2645027.htm) bağımlı sürümü: https://github.com/walkor/workerman-filemonitor-inotify

**İki sürüm arasındaki farklar:**

Adres 1 sürümü, her saniye dosya güncelleme zamanını sorgulayarak dosyanın güncellenip güncellenmediğini kontrol eder.

Adres 2 ise Linux çekirdeğinin inotify mekanizmasını kullanarak, dosya güncellendiğinde sistem workerman'a bildirimde bulunur.

Genellikle, bağımsız sürüm olan adres 1 kullanılabilir.
