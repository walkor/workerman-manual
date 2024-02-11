# Sistem Çağrılarını İzleme

Bir işlemin ne yaptığını öğrenmek istediğinizde, `strace` komutuyla bir işlemin tüm sistem çağrılarını izleyebilirsiniz.

1. `php start.php status` komutunu çalıştırarak workerman ile ilgili işlem bilgilerini görebilirsiniz:

```plaintext
Merhaba admin
---------------------------------------GLOBAL DURUM--------------------------------------------
Workerman sürümü: 3.0.1
başlangıç zamanı: 2014-08-12 17:42:04   0 gün 1 saat çalışma
ortalama yük: 3.34, 3.59, 3.67
1 kullanıcı          8 çalışan       14 işlem
çalışan_adı       çıkış_durumu     çıkış_sayısı
BusinessWorker    0                0
ChatWeb           0                0
FileMonitor       0                0
Gateway           0                0
Monitor           0                0
StatisticProvider 0                0
StatisticWeb      0                0
StatisticWorker   0                0
---------------------------------------İŞLEM DURUMU-------------------------------------------
pid	bellek      dinlenen        zaman damgası  worker_adı       toplam_talep paket_hata yıldırım_surusu istemci_kapatma gonderme_basarisiz istisna_atma başarılı/toplam
10352	1.5M    tcp://0.0.0.0:55151  1407836524 ChatWeb           12             0          0            2            0         0               100%
10354	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          0            0            0         0               100%
10355	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           0              0          1            0            0         0               100%
...
```

2. Örneğin, 10354 pid'li gateway işleminin ne yaptığını öğrenmek istediğimizde, aşağıdaki gibi bir komutu çalıştırabiliriz: 

```plaintext
sudo strace -p 10354
İşlem 10354 eklendi - çıkmak için kesin
clock_gettime(CLOCK_MONOTONIC, {118627, 242986712}) = 0
gettimeofday({1407840609, 102439}, NULL) = 0
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Bekleyen sistem çağrısı kesintisi)
--- SIGUSR2 (Kullanıcı tanımlı sinyal 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (şimdi maskesi [])
clock_gettime(CLOCK_MONOTONIC, {118627, 699623319}) = 0
gettimeofday({1407840609, 559092}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118627, 699810499}) = 0
gettimeofday({1407840609, 559277}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Kaynak geçici olarak uygun değil)
...
```

3. Burada her satır bir sistem çağrısını temsil etmektedir. Bu bilgilerden işlemin ne yaptığını kolayca görebilir, işlemin nerede takıldığını, bağlantıda mı yoksa ağ verisi okumada mı takıldığını belirleyebiliriz.
