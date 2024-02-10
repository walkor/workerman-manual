# Desteklenmeyen Fonksiyonlar

Desteklenmeyen Fonksiyon/İfadeler | Yerine Kullanım | Açıklama
----|------|----
pcntl_fork | Önceden işlem sayısını ayarlayın | 
php://input | [`$request->rawBody()`](http/request.md)| HTTP protokolü altında POST'un ham verisini almak için kullanılır
exit | return | exit kullanımı işlemi sonlandırır, geri dönmek için doğrudan return ifadesini kullanın
die | return | die kullanımı işlemi sonlandırır, geri dönmek için doğrudan return ifadesini kullanın
header cookie session ile ilgili fonksiyonlar | [`$request`](http/request.md) ve [`$response`]([http/response.md) sınıflarına bakınız | 
set_time_limit| Yok | Sadece 0 olarak ayarlanabilir, aksi takdirde workerman işlemi belli bir süre sonra sonlandırır
