# Mối quan hệ với Apache và Nginx

**Câu hỏi:**

Workerman và Apache/nginx/php-fpm có mối quan hệ gì? Workerman và Apache/nginx/php-fpm có xung đột không?

**Trả lời:**

Workerman và Apache/nginx/php-fpm không có mối quan hệ gì, và hoạt động của Workerman không phụ thuộc vào Apache/nginx/php-fpm. Chúng đều là các container độc lập, không tác động lẫn nhau và cũng không xảy ra xung đột (trong trường hợp không lắng nghe cùng một cổng).

Workerman là một framework máy chủ socket thông thường, hỗ trợ kết nối lâu dài và hỗ trợ đa dạng giao thức như HTTP, WebSocket và giao thức tùy chỉnh. Trong khi đó, Apache/nginx/php-fpm thường chỉ được sử dụng cho các dự án Web sử dụng giao thức HTTP.

Nếu máy chủ đã triển khai Apache/nginx/php-fpm, việc triển khai Workerman sẽ không ảnh hưởng đến hoạt động của chúng.
