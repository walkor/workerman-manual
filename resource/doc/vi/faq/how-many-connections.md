# WorkerMan hỗ trợ bao nhiêu kết nối đồng thời

**Khái niệm về đồng thời** rất mơ hồ, ở đây chúng ta sẽ sử dụng hai chỉ số có thể đo lường: **số lượng kết nối đồng thời** và **số lượng yêu cầu đồng thời** để giải thích.

**Số lượng kết nối đồng thời** là số lượng kết nối TCP mà máy chủ đang duy trì tại thời điểm hiện tại, không quan tâm đến việc có truyền thông dữ liệu trên các kết nối này hay không, ví dụ, một máy chủ thông báo có thể duy trì hàng triệu kết nối thiết bị, vì ít kết nối có truyền thông dữ liệu, nên máy chủ này gánh nặng gần như là 0, chỉ cần đủ bộ nhớ, nó vẫn có thể tiếp tục chấp nhận kết nối.

**Số lượng yêu cầu đồng thời** thường được đo bằng QPS (số lượng yêu cầu máy chủ xử lý mỗi giây), và không quá quan trọng về số lượng kết nối TCP đang có trên máy chủ vào thời điểm đó. Ví dụ, một máy chủ chỉ có 10 kết nối khách hàng, mỗi kết nối có 10.000 yêu cầu mỗi giây, vậy máy chủ sẽ cần ít nhất có khả năng xử lý 10 * 1W = 10W yêu cầu mỗi giây (QPS). Giả sử 10W yêu cầu mỗi giây là giới hạn cuối cùng của máy chủ này, nếu mỗi khách hàng gửi một yêu cầu mỗi giây đến máy chủ, máy chủ này có thể hỗ trợ 10W khách hàng.  
  
**Số lượng kết nối đồng thời** bị hạn chế bởi bộ nhớ máy chủ, một máy chủ workerman với 24G bộ nhớ có thể hỗ trợ khoảng **120W** kết nối đồng thời.

**Số lượng yêu cầu đồng thời** bị hạn chế bởi khả năng xử lý của CPU máy chủ, một máy chủ workerman 24 nhân có thể đạt được **45W** yêu cầu mỗi giây (QPS), giá trị thực tế sẽ thay đổi tùy thuộc vào độ phức tạp của doanh nghiệp và chất lượng mã nguồn.

## Lưu ý

Trong các kịch bản có số lượng lớn kết nối đồng thời, cần cài đặt phần mở rộng sự kiện (event), xem phần cài đặt và cấu hình. Ngoài ra, cần tối ưu hóa kernel Linux, đặc biệt là giới hạn số tệp mở của quá trình, xin xem phần tối ưu hóa kernel trong phần phụ lục.

## Dữ liệu Đo áp

> Dữ liệu được cung cấp bởi tổ chức đo áp uy tín bên thứ ba techempower.com của vòng 20 đo áp
https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf

**Cấu hình máy chủ:**
Tổng số lõi 14, Tổng số luồng 28, 32 GB bộ nhớ, Máy chủ Ethernet Cisco 10 gigabit
**Logic kinh doanh:**
Logic kinh doanh có truy vấn cơ sở dữ liệu, cơ sở dữ liệu pgsql, php8+jit
QPS là 39 nghìn
![](../images/screenshot_1636522357217.png)
