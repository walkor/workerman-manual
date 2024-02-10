# 與Apache和Nginx的關係
**問：**

Workerman和Apache/nginx/php-fpm有什麼關係？Workerman會和Apache/nginx/php-fpm有衝突嗎？

**答：**
Workerman和Apache/nginx/php-fpm沒有任何關係，而且Workerman的運行不依賴於Apache/nginx/php-fpm。它們都是獨立的容器，互不干擾，也不會有衝突（在不監聽同一個端口的情況下）。

Workerman是一個通用的socket伺服器框架，支援長連接，並支援各種協議，如HTTP、WebSocket以及自定義協議。而一般而言，Apache/nginx/php-fpm主要用於開發使用HTTP協議的Web項目。

如果伺服器已經部署了Apache/nginx/php-fpm，部署Workerman不會影響它們的運行。
