# Linux Kernel Optimization

Sistemimizin daha fazla eşzamanlı kullanımı desteklemesi için, [event eklentisinin kurulması](../install/install.md) gerektiği gibi, Linux çekirdeğinin optimize edilmesi de **çok önemlidir**. Aşağıdaki her bir iyileştirme **çok çok önemlidir** ve her birini ayrı ayrı tamamlamanız gerekmektedir.

**Parametre Açıklamaları:**

> **max-file**: Tüm işletim sistemi için açılabilen dosya tanıtıcılarının sayısını temsil eder. Bu tüm işletim sistemi için geçerlidir ve kullanıcılar için değil.
> 
> **ulimit -n**: İşlem düzeyinde açılabilecek dosya tanıtıcılarının sayısını kontrol eder. Bu, mevcut `shell` altındaki geçerli kullanıcı ve başlatılan işlemlerin kullanılabilir dosya tanıtıcılarını kontrol eder.

Sistem düzeyinde açılabilecek dosya tanıtıcılarının sayısını kontrol etmek için: `cat /proc/sys/fs/file-max`

/etc/sysctl.conf dosyasını açın ve aşağıdaki ayarları ekleyin
```conf
# Bu parametre, sistemde TIME_WAIT sayısını ayarlar, varsayılan değeri aşarsa derhal temizlenir
net.ipv4.tcp_max_tw_buckets = 20000
# Sistemde her bir bağlantı noktası için maksimum dinleme kuyruğunun uzunluğunu tanımlar, bu genel bir parametredir
net.core.somaxconn = 65535
# Karşı tarafın onayını almayan bağlantı istekleri için kuyrukta saklanabilecek maksimum sayı
net.ipv4.tcp_max_syn_backlog = 262144
# Her ağ arayüzünde çekirdek bu paketleri işleme hızından daha hızlı bir şekilde veri paketleri alındığında, kuyruğa iletilmesine izin verilen maksimum veri paketi sayısı
net.core.netdev_max_backlog = 30000
# Bu seçenek, NAT ağındaki istemci zaman aşımına neden olabilir, 0 olarak ayarlanması önerilir. Linux, 4.12 çekirdek sürümünden itibaren tcp_tw_recycle yapılandırmasını kaldırmıştır. "No such file or directory" hatası alıyorsanız, bu hatayı yoksayın
net.ipv4.tcp_tw_recycle = 0
# Sistem genelinde tüm süreçlerin açabileceği dosya sayısı
fs.file-max = 6815744
# Firewall izleme tablosunun boyutu. Not: Eğer bir firewall açık değilse "net.netfilter.nf_conntrack_max" bilinmeyen bir anahtar hatası verebilir, bunu görmezden gelebilirsiniz
net.netfilter.nf_conntrack_max = 2621440
net.ipv4.ip_local_port_range = 10240 65000
```
`sysctl -p` komutunu çalıştırarak hemen etkinleştirin.

**Not:** 

/etc/sysctl.conf dosyasında yapılabilecek diğer ayarlar, kendi ortamınıza göre ayarlanabilir.

## Dosya Sayısının Artırılması

Sistemin dosya açma sayısını ayarlayarak yüksek eşzamanlılık altında `too many open files` hatasını çözebilirsiniz. Bu seçenek, doğrudan tek bir işlemin barındırabileceği istemci bağlantı sayısını etkiler.

Soft open files, Linux sistemi parametresidir ve bir işlemin açabileceği maksimum dosya tanıtıcı sayısını etkiler. Bu değer, uzun bağlantı uygulamalarını etkiler; örneğin, tek bir işlemin sürdürebileceği kullanıcı bağlantı sayısını etkiler. `ulimit -n` komutunu çalıştırarak bu parametre değerini görebilirsiniz. Örneğin, eğer 1024 ise, tek bir işlem en fazla 1024 hatta daha azını sürdürebilir (çünkü başka dosyalar da açıktır). Eğer 4 işlem başlatılırsa, uygulama aynı anda en fazla 4*1024 bağlantıyı koruyabilir, yani aynı anda en fazla 4x1024 kullanıcıyı destekleyebilir; bu da demektir ki bu ayarı artırarak daha fazla TCP bağlantısını destekleyebilirsiniz.

**Open Files Ayarını Değiştirme Üç Yöntemi:**

Birinci yöntem: Terminalde `ulimit -HSn 102400` komutunu çalıştırın ve ardından workerman'ı yeniden başlatın.

Bu, sadece geçerli terminal için geçerlidir; çıkış yapıldıktan sonra açık dosyaların değeri tekrar varsayılan değere döner.

İkinci yöntem: `/etc/profile` dosyasının sonuna `ulimit -HSn 102400` satırını ekleyin, böylece her oturum açıldığında otomatik olarak çalışır. Değişikliklerden sonra workerman'ı yeniden başlatmanız gerekmektedir.

Üçüncü yöntem: Açık dosya sayısını kalıcı şekilde değiştirmek için `/etc/security/limits.conf` dosyasını düzenlemeniz gerekmektedir. Bu dosyaya şunları ekleyin:

``` 
* soft nofile 1024000
* hard nofile 1024000
root soft nofile 1024000
root hard nofile 1024000
```

Bu yöntem sadece sunucuyu yeniden başlatarak etkin olacaktır.
