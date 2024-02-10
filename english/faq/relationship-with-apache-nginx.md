# Relationship with Apache and Nginx

**Question:**

What is the relationship between Workerman and Apache/nginx/php-fpm? Do they conflict with each other?

**Answer:**

Workerman has no relationship with Apache/nginx/php-fpm, and its operation does not depend on Apache/nginx/php-fpm. They are all independent containers, do not interfere with each other, and will not conflict (as long as they do not listen on the same port).

Workerman is a general-purpose socket server framework that supports long connections and various protocols such as HTTP, WebSocket, and custom protocols. Apache/nginx/php-fpm, on the other hand, are generally used for developing HTTP protocol web projects.

If a server has already deployed Apache/nginx/php-fpm, deploying Workerman will not affect their operation.
