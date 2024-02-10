# Kênh - Linh kiện truyền thông phân tán

**``` (Yêu cầu phiên bản Workerman >= 3.3.0) ```**

Địa chỉ nguồn mã nguồn: https://github.com/walkor/Channel

Kênh là một linh kiện truyền thông phân tán, được sử dụng để truyền thông giữa các tiến trình hoặc giữa các máy chủ.

## Đặc điểm
1. Dựa trên mô hình đăng ký và phát hành
2. Sử dụng IO không đồng bộ

## Nguyên lý
Kênh bao gồm máy chủ kênh (Channel/Server) và máy khách kênh (Channel/Client).

Máy khách kênh (Channel/Client) kết nối với máy chủ kênh (Channel/Server) thông qua giao diện connect và duy trì kết nối lâu dài.

Máy khách kênh (Channel/Client) thông báo với máy chủ kênh (Channel/Server) thông qua gọi giao diện on về việc quan tâm đến những sự kiện nào và đăng ký hàm gọi lại sự kiện (hàm gọi lại xảy ra trong tiến trình của máy khách kênh (Channel/Client)).

Máy khách kênh (Channel/Client) thông qua giao diện publish để phát hành một sự kiện cụ thể và dữ liệu liên quan đến sự kiện đó đến máy chủ kênh (Channel/Server).

Máy chủ kênh (Channel/Server) nhận sự kiện và dữ liệu sau đó sẽ phân phối cho máy khách kênh (Channel/Client) quan tâm đến sự kiện này.

Máy khách kênh (Channel/Client) nhận sự kiện và dữ liệu sau đó kích hoạt hàm gọi lại được thiết lập thông qua giao diện on.

Máy khách kênh (Channel/Client) chỉ nhận được sự kiện mình quan tâm và kích hoạt hàm gọi lại.

## Cài đặt

`composer require workerman/channel`

## Lưu ý
Kênh chỉ có thể sử dụng trong môi trường Workerman, không thể sử dụng trong môi trường php-fpm.
