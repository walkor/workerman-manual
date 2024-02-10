# Các giao diện cung cấp bởi lớp Connection

Trong WorkerMan có hai lớp quan trọng là Worker và Connection.

Mỗi kết nối từ client tương ứng với một đối tượng Connection, có thể thiết lập các callback như onMessage, onClose cho đối tượng này, đồng thời cung cấp các giao diện như send để gửi dữ liệu đến client và close để đóng kết nối, cùng với một số giao diện cần thiết khác.

Có thể nói rằng Worker là một container lắng nghe, chịu trách nhiệm cho việc chấp nhận kết nối từ client và đóng gói kết nối để cung cấp cho các nhà phát triển sử dụng qua đối tượng Connection.
