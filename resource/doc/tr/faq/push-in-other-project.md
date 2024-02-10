# WorkerMan'ı diğer projelerde istemciye veri göndermek için nasıl kullanabilirim?

**Soru:**

Normal bir web projesine sahibim ve bu projede WorkerMan'ın API'sini kullanarak istemcilere veri göndermek istiyorum.

**Cevap:**

**Workerman'a dayalı çözümler için aşağıdaki bağlantıları inceleyebilirsiniz:**

- [Channel bileşeni örnekleri](../components/channel-examples.md) (Çoklu işlem/sunucu kümesini destekler, Channel bileşenini indirmeniz gerekir)

- [Worker üzerinden gönderme](https://www.workerman.net/q/508) (Tek işlem, en basit olanı)

**Webman'a dayalı çözümler için aşağıdaki bağlantıları inceleyebilirsiniz:**

- [webman push eklentisi](https://www.workerman.net/plugin/2)

**GatewayWorker'a dayalı çözümler için aşağıdaki bağlantıları inceleyebilirsiniz:**

- [Diğer projeler üzerinden GatewayWorker ile gönderme](https://www.workerman.net/doc/gateway-worker/push-in-other-project.html) (Çoklu işlem/sunucu kümesini destekler, gruplandırma, grup yayını, bireysel gönderme destekler)

**PHPSocket.IO'ya dayalı çözümler için aşağıdaki bağlantıları inceleyebilirsiniz:**

- [Web mesajı gönderme](https://www.workerman.net/web-sender) (Varsayılan olarak tek işlem, socket.io'ya dayalı, tarayıcı uyumluluğu en iyisi)
