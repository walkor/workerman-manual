# Connection Sınıfının Sağladığı Arayüz

Workerman'de Worker ve Connection gibi iki önemli sınıf bulunmaktadır.

Her istemci bağlantısı bir Connection nesnesine karşılık gelir. Bu nesnelerin onMessage, onClose vb. geri çağrıları ayarlanabilir. Aynı zamanda, send arayüzü aracılığıyla istemciye veri gönderme ve close arayüzü aracılığıyla bağlantıyı kapatma gibi işlevler sunar. Başka bazı gerekli arayüzleri de sağlar.

Worker, istemci bağlantılarını dinleyen bir konteynerdir ve bağlantıyı Connection nesneleri şeklinde geliştiricilere sunar.
