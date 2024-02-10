# Workerman có hỗ trợ đa luồng không?

Workerman có một phiên bản đa luồng [MT](https://github.com/walkor/workerman-MT) phụ thuộc vào tiện ích [pthreads extension](https://php.net/manual/zh/book.pthreads.php), nhưng vì phần mở rộng pthreads vẫn chưa ổn định đủ nên phiên bản đa luồng của Workerman không còn được duy trì.

**Hiện tại, Workerman và các sản phẩm liên quan dựa trên kiến trúc đa tiến trình đơn luồng.**
