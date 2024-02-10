# Not: Zamanlayıcı Kullanımı İçin Dikkat Edilmesi Gerekenler
1. Sadece ```onXXXX``` geri çağrıları içinde zamanlayıcı ekleyebilirsiniz. Genel zamanlayıcılar için ```onWorkerStart``` geri çağrısında, belirli bir bağlantı için zamanlayıcılar için ise ```onConnect``` içinde ayarlama yapmanız önerilir.

2. Eklenen zamanlayıcı görevleri mevcut işlemde çalışır (yeni bir işlem veya iş parçacığı başlatmaz) ve görev ağır ise (özellikle ağ IO içeren görevler) bu işlemdeki diğer işlemlerin geçici olarak işlenememesine neden olabilir. Bu nedenle uzun süren görevleri ayrı bir işlemde çalıştırmak daha iyidir, örneğin bir veya daha fazla Worker işlemi oluşturarak.

3. Mevcut işlem diğer işlerle meşgulken veya bir görev beklenen sürede tamamlanmazsa, bu durumda bir sonraki çalışma döngüsü geldiğinde mevcut görev tamamlanana kadar bekler ve bu durum zamanlayıcıların beklenen sürede çalışmamasına neden olabilir. Yani mevcut işlemler iş parçacıklarında seri olarak çalışır, çoklu işlemler ise işlemler arasında paralel olarak çalışır.

4. Birden fazla işlem için zamanlayıcı görevleri ayarlanması, eş zamanlı problemi yaratabilir, örneğin aşağıdaki kod her saniye 5 kez yazdırır.
```php
$worker = new Worker();
// 5 işlem
$worker->count = 5;
$worker->onWorkerStart = function(Worker $worker) {
    // 5 işlem, her bir işlemde böyle bir zamanlayıcı var
    Timer::add(1, function(){
        echo "hi\r\n";
    });
};
Worker::runAll();
```
Sadece bir işlemin zamanlayıcıyı çalıştırmasını istiyorsanız, [Timer::add Örneği 2](add.md) kısmına bakın.

5. 1 milisaniye kadar bir hata olabilir.

6. Zamanlayıcılar işlem arası silinemez, örneğin a işlemi tarafından ayarlanan bir zamanlayıcıyı b işlemi doğrudan Timer::del arayüzü ile silemez.

7. Farklı işlemler arasında zamanlayıcı kimliği tekrarlanabilir, ancak aynı işlem içinde oluşturulan zamanlayıcı kimliği tekrarlanmaz.

8. Sistem zamanını değiştirmek, zamanlayıcının davranışını etkileyebilir, bu nedenle sistem zamanını değiştirdikten sonra yeniden başlatmayı öneririz.
