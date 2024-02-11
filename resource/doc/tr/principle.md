# İlkeler

### Worker Açıklaması
Worker, Workerman'de en temel konteynerdir. Worker belirli bir protokol ile birden çok işlemi dinleyebilir ve iletişim kurabilir, bir nevi belirli bir portu dinleyen nginx gibi. Her Worker işlemi bağımsız bir şekilde çalışır, Epoll (event eklentisi kurulu olmalı) + bloke etmeyen G/Ç kullanır, her Worker işlemi on binlerce istemci bağlantısını alabilir ve bu bağlantılardan gelen verileri işleyebilir. Ana işlem, istikrarı korumak için sadece alt işlemleri izler, veri kabul etmez ve herhangi bir iş mantığı yapmaz.

### Müşteri ve işlemci işlemi ilişkisi
![workerman master woker模型](images/Worker.png)

### Ana işlem ile işçi alt işlemi ilişkisi
![workerman master woker模型](images/Worker2.png)

**Özellikler:**

Görüldüğü gibi, her Worker kendi istemci bağlantılarını korur, istemci ve sunucu arasında gerçek zamanlı iletişimi kolayca sağlayabilir, bu model sayesinde temel geliştirme gereksinimlerini kolayca karşılayabiliriz, örneğin HTTP sunucusu, Rpc sunucusu, bazı akıllı donanım gerçek zamanlı veri raporlamaları, sunucu tarafı veri itme, oyun sunucusu, WeChat mini uygulama arka ucu vb. gibi.

