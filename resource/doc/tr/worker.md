# Worker Sınıfı
Workerman'de Worker ve Connection olmak üzere iki önemli sınıf bulunmaktadır.

Worker sınıfı, port dinlemeyi sağlar ve istemci bağlantısı olayı, bağlantı üzerindeki mesaj olayı, bağlantının kesilme olayı için geri çağırma işlevlerini ayarlayabilir, bu sayede işlemleri gerçekleştirebilir.

Worker örneğinin işlem sayısını (count özelliği) ayarlayabilirsiniz. Worker ana işlem, count adet alt işlemi çocuk işlem olarak çıkararak aynı portu dinler ve istemci bağlantılarını eş zamanlı olarak kabul eder, bağlantı olaylarını işler.
