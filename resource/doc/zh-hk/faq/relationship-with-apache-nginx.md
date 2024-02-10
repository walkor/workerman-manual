# 與Apache和Nginx的關係
**問：**

Workerman和Apache/nginx/php-fpm是什麼關係？Workerman和Apache/nginx/php-fpm
衝突嗎？

**答：**
Workerman和Apache/nginx/php-fpm沒有任何關係，而且Workerman的運行不依賴於Apache/nginx/php-fpm。他們都是獨立的容器，互不干擾，也不會衝突（在不監聽同一個端口的情況下）。

Workerman是一個通用的socket伺服器框架，支持長連接，支持各種協議如HTTP、WebSocket以及自定義協議。而Apache/nginx/php-fpm一般來說只用於開發HTTP協議的Web專案。

如果伺服器已經部署了Apache/nginx/php-fpm，部署Workerman不會影響到它們的運行。
