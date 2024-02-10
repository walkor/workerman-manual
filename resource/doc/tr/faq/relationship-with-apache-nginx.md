# Apache ve Nginx ile İlişkisi
**Soru:**

Workerman ve Apache/nginx/php-fpm arasındaki ilişki nedir? Workerman ve Apache/nginx/php-fpm birbiriyle çatışır mı?

**Cevap:**
Workerman ve Apache/nginx/php-fpm arasında herhangi bir ilişki yoktur ve Workerman'ın çalışması Apache/nginx/php-fpm'ye bağlı değildir. Bunlar bağımsız konteynerlerdir, birbirlerini etkilemezler ve çatışmazlar (aynı portu dinlemedikleri sürece).

Workerman genel amaçlı bir socket sunucusu çerçevesidir, uzun süreli bağlantıyı destekler, HTTP, WebSocket ve özel protokoller gibi çeşitli protokolleri destekler. Apache/nginx/php-fpm ise genellikle yalnızca HTTP protokolü için web projeleri geliştirmek için kullanılır.

Eğer sunucu Apache/nginx/php-fpm ile zaten dağıtılmışsa, Workerman'ın dağıtılması onların çalışmasını etkilemeyecektir.
