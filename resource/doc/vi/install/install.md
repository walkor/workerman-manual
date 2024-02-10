# Hướng dẫn cài đặt
WorkerMan thực sự chỉ là một gói mã nguồn PHP, nếu bạn đã cài đặt môi trường PHP, chỉ cần tải mã nguồn hoặc ví dụ của WorkerMan và chạy nó.

**Cài đặt bằng Composer:**
```sh
composer require workerman/workerman
```

> **Lưu ý**
> Một số mirror proxy của Composer không đầy đủ, hãy sử dụng lệnh trên để gỡ bỏ proxy `composer config -g --unset repos.packagist`

# Người dùng Windows (bắt buộc đọc)
Từ phiên bản workerman3.5.3 trở lên, workerman đã hỗ trợ cả hệ thống windows và linux. Người dùng windows cần cấu hình biến môi trường PHP.

 ` ===Từ đây trở xuống chỉ dành cho môi trường Linux workerman, người dùng windows vui lòng bỏ qua=== `

# Kiểm tra môi trường hệ thống Linux
Hệ thống Linux có thể sử dụng đoạn mã dưới đây để kiểm tra môi trường PHP trên máy tính xem có đáp ứng yêu cầu chạy WorkerMan không.
 `curl -Ss https://www.workerman.net/check | php`

Nếu tất cả đều hiển thị OK, tương đương với việc đáp ứng yêu cầu của WorkerMan, bạn có thể tải demo từ [trang web chính thức](https://www.workerman.net/) để chạy.

Nếu không đáp ứng hết, hãy tham khảo tài liệu dưới đây để cài đặt các phần mở rộng còn thiếu.

(Lưu ý: Đoạn mã kiểm tra không kiểm tra phần mở rộng event, nếu số kết nối đồng thời lớn hơn 1024, bạn cần cài đặt phần mở rộng event và [tối ưu hóa hạt nhân Linux](../appendices/kernel-optimization.md), tham khảo cách cài đặt phần mở rộng dưới đây)

# Cài đặt phần mở rộng thiếu trên môi trường PHP hiện có

## Cài đặt phần mở rộng pcntl và posix:

**Hệ thống centos**
Nếu PHP được cài đặt thông qua yum, chỉ cần chạy dòng lệnh ```yum install php-process``` để cài đặt các phần mở rộng pcntl và posix.

Nếu cài đặt thất bại hoặc PHP không được cài đặt bằng yum, vui lòng tham khảo mục [Appendices - Cài đặt phần mở rộng](../appendices/install-extension.md) phần 3 cài đặt từ mã nguồn.

**Hệ thống debian/ubuntu/mac os**
Tham khảo mục [Appendices - Cài đặt phần mở rộng](../appendices/install-extension.md) phần 3 cài đặt từ mã nguồn.

## Cài đặt phần mở rộng event:
Để hỗ trợ số lượng kết nối đồng thời lớn hơn, bạn cần cài đặt phần mở rộng event và [tối ưu hóa hạt nhân Linux](../appendices/kernel-optimization.md). Cách cài đặt như sau:

**Hệ thống centos**

1. Cài đặt gói libevent-devel cần thiết cho phần mở rộng event, chạy dòng lệnh sau
```shell
yum install libevent-devel -y
# Nếu không thể cài đặt, thử sử dụng dòng lệnh sau
# yum install libevent2-devel -y
```

2. Cài đặt phần mở rộng event, chạy dòng lệnh sau
(phần mở rộng event yêu cầu PHP>=5.4)
```shell
pecl install event
```
Lưu ý: Khi yêu cầu "Include libevent OpenSSL support [yes] :" xuất hiện, hãy nhập "no" rồi nhấn Enter, nhấn Enter tiếp theo cho tất cả các yêu cầu khác.

3. Chạy ```php --ini``` để tìm và mở tệp php.ini, thêm cấu hình sau vào dòng cuối cùng
```shell
extension=event.so
```

**Hệ thống debian/ubuntu**

1. Cài đặt gói libevent-dev cần thiết cho phần mở rộng event, chạy dòng lệnh sau
```shell
apt-get install libevent-dev -y
# Nếu không thể cài đặt, thử sử dụng dòng lệnh sau
# apt-get install libevent2-dev -y
```

2. Cài đặt phần mở rộng event, chạy dòng lệnh sau
```shell
pecl install event
```
Lưu ý: Khi yêu cầu "Include libevent OpenSSL support [yes] :" xuất hiện, hãy nhập "no" rồi nhấn Enter, nhấn Enter tiếp theo cho tất cả các yêu cầu khác.

3. Chạy ```php --ini``` để tìm và mở tệp php.ini, thêm cấu hình sau vào dòng cuối cùng
```shell
extension=event.so
```

**Hướng dẫn cài đặt trên hệ thống mac os**
Trên hệ thống mac, thường được sử dụng làm máy phát triển, không cần cài đặt phần mở rộng event.

# Cài đặt từ đầu (Cài đặt PHP + Phần mở rộng từ đầu)

## Hướng dẫn cài đặt trên hệ thống centos

1. Chạy dòng lệnh sau (bước này đã bao gồm cài đặt chương trình chính php-cli cũng như pcntl, posix, libevent và git)
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```

2. Cài đặt phần mở rộng event, chạy dòng lệnh sau
(Lưu ý: phần mở rộng event yêu cầu PHP>=5.4)
```shell
pecl install event
```
Lưu ý: Khi yêu cầu "Include libevent OpenSSL support [yes] :" xuất hiện, hãy nhập "no" rồi nhấn Enter, nhấn Enter tiếp theo cho tất cả các yêu cầu khác.

3. Chạy ```php --ini``` để tìm và mở tệp php.ini, thêm cấu hình sau vào dòng cuối cùng
```shell
extension=event.so
```

4. Chạy dòng lệnh sau (bước này dung để tải mã nguồn chính của WorkerMan từ github)
```shell
git clone https://github.com/walkor/Workerman
```

5. Tham khảo [Hướng dẫn bắt đầu - Phần ví dụ phát triển đơn giản](../getting-started/simple-example.md) để viết mã vào tập tin lối vào chạy hoặc tải ví dụ đã được đóng gói từ [trang web chính thức](https://www.workerman.net/) để chạy.

## Hướng dẫn cài đặt trên hệ thống debian/ubuntu

1. Chạy dòng lệnh sau (bước này đã bao gồm cài đặt chương trình chính php-cli, libevent và git)
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2. Cài đặt phần mở rộng event, chạy dòng lệnh sau
(Lưu ý: phần mở rộng event yêu cầu PHP>=5.4)
```shell
pecl install event
```
Lưu ý: Khi yêu cầu "Include libevent OpenSSL support [yes] :" xuất hiện, hãy nhập "no" rồi nhấn Enter, nhấn Enter tiếp theo cho tất cả các yêu cầu khác.

3. Chạy ```php --ini``` để tìm và mở tệp php.ini, thêm cấu hình sau vào dòng cuối cùng
```shell
extension=event.so
```

4. Chạy dòng lệnh sau (bước này dung để tải mã nguồn chính của WorkerMan từ github)
```shell
git clone https://github.com/walkor/Workerman
```

5. Tham khảo [Hướng dẫn bắt đầu - Phần ví dụ phát triển đơn giản](../getting-started/simple-example.md) để viết mã vào tập tin lối vào chạy hoặc tải ví dụ đã được đóng gói từ [trang web chính thức](https://www.workerman.net/) để chạy.

## Hướng dẫn cài đặt trên hệ thống mac os
**Phương pháp 1:** Mac thường được sử dụng làm máy phát triển có thể thiếu phần mở rộng ```pcntl```.

1. Tham khảo [Hướng dẫn cài đặt phần mở rộng - Phần 3 từ mã nguồn](../appendices/install-extension.md) để cài đặt phần mở rộng ```pcntl```.

2. Tham khảo [Hướng dẫn cài đặt phần mở rộng - Phần 4 sử dụng phpize](../appendices/install-extension.md) để cài đặt phần mở rộng ```event``` (nhưng với máy phát triển, bước này có thể bỏ qua).

3. Tải mã nguồn chính của WorkerMan từ https://www.workerman.net/download/workermanzip hoặc tải ví dụ từ [trang web chính thức](https://www.workerman.net/) để chạy.

**Phương pháp 2:** Cài đặt php và phần mở rộng tương ứng thông qua lệnh ```brew```:

1. Chạy dòng lệnh sau để cài đặt công cụ ```brew``` (nếu đã cài đặt rồi có thể bỏ qua bước này)
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. Chạy dòng lệnh sau để cài đặt ```php```
```shell
brew install php
```

3. Chạy dòng lệnh sau để cài đặt phần mở rộng ```event```
```shell
brew install php-event
```

4. Tải ví dụ từ [trang web chính thức](https://www.workerman.net/) để chạy.

# Mô tả Phần mở rộng Event
Phần mở rộng [Event](https://php.net/manual/zh/book.event.php) không phải là bắt buộc, khi dự án yêu cầu hỗ trợ số lượng kết nối đồng thời lớn hơn 1000, khuyến nghị cài đặt Event để hỗ trợ kết nối cực lớn. Nếu số kết nối đồng thời của dự án thấp, chẳng hạn dưới 1000 kết nối đồng thời, bạn có thể không cần cài đặt.

## Các vấn đề thường gặp
1. Nếu gặp lỗi như sau:  `checking for include/event2/event.h... not found`, hãy thử xóa gói libevent-dev(el) và cài đặt libevent2-dev(el) thay thế.
Hệ thống centos: yum remove libevent-devel && yum install libevent2-devel
Hệ thống debian/ubuntu: apt-get remove libevent-dev && apt-get install libevent2-dev

2. Nếu gặp lỗi như sau: `NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`.
Hãy thay đổi thứ tự tải event.so và socket.so trong php.ini, tức là trong tệp php.ini, ghi ```extension=socket.so``` trước dòng ```extension=event.so```, cho phép tải phần mở rộng socket trước.
