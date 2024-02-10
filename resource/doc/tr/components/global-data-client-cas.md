# CAS

**``` (Workerman versiyonu >=3.3.0 gerektirir) ```**

```php
bool \GlobalData\Client::cas(string $key, mixed $old_value, mixed $new_value)
```

Atomik olarak eski değeri ```$old_value``` ile yeni değer ile değiştirir. Sadece bu anahtarın değeri diğer istemciler tarafından değiştirilmediği durumlarda son istemci tarafından alınan son değerler üzerinde işlem yapılırken değer yazılabilir.


## Parameters

 ``` $key ```

Anahtar değer. (Örneğin ```$global->abc```, ```abc``` anahtar değeridir)

 ``` $old_value ```

Eski veri

 ``` $new_value ```

Yeni veri

## Return Value

Değiştirme başarılı ise true, aksi halde false döner.

## Açıklama:

Aynı paylaşılan değişkeni aynı anda çoklu işlemle işlerken, bazen eşzamanlılık sorunu göz önünde bulundurulmalıdır.

Örneğin, A ve B iki süreç aynı kullanıcı listesine bir üye ekleyebilir.
A ve B süreçleri mevcut kullanıcı listesi ```$global->user_list = array(1,2,3)``` olabilir.
A süreci ```$global->user_list``` değişkenini düzenler ve bir kullanıcı 4'ü ekler.
B süreci ```$global->user_list``` değişkenini düzenler ve bir kullanıcı 5 ekler.
A süreci değişkeni ```$global->user_list = array(1,2,3,4)``` olarak başarıyla ayarlar.
B süreci değişkeni ```$global->user_list = array(1,2,3,5)``` olarak başarıyla ayarlar.
Bu durumda B süreci tarafından ayarlanan değişken, A süreci tarafından ayarlanan değişkeni üzerine yazarak veri kaybına neden olur.

Yukarıdaki durum, okuma ve yazma işlemlerinin atomik olmadığından dolayı eşzamanlılık sorununa neden olur.
Bu tür eşzamanlılık sorunları çözmek için, cas atomik değiştirme arayüzü kullanılabilir.
cas arayüzü bir değeri değiştirmeden önce,
bu değerin ```$old_value``` tarafından diğer işlemler tarafından değiştirilip değiştirilmediğine karar verir,
eğer değiştirildiyse değiştirmez false döner. Aksi halde true döner.
Aşağıdaki örneğe bakın.

**Not:** Birkaç paylaşılan verinin eşzamanlı olarak üzerine yazılması sorun yaratmaz, örneğin müzayede sistemi, mevcut en yüksek teklif veya mevcut ürün stoku gibi.

## Örnekler

```php
$global = new GlobalData\Client('127.0.0.1:2207');

// Listeyi başlat
$global->user_list = array(1,2,3);

// user_list'e atomik olarak bir değer ekle
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```
