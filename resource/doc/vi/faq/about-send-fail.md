# Nguyên nhân của send_fail trong status

**Hiện tượng:**

Chạy lệnh status, bạn thấy có tình trạng send_fail, đó là nguyên nhân gì?

**Trả lời:**

Thường thì việc có send_fail không phải là vấn đề lớn, thường xuất phát từ việc máy khách đóng kết nối một cách chủ động hoặc máy khách không thể nhận dữ liệu dẫn đến việc gửi thất bại.

send_fail có hai nguyên nhân:

1. Khi gọi giao diện send để gửi dữ liệu đến máy khách nhưng phát hiện máy khách đã ngắt kết nối, số lượng send_fail sẽ tăng lên. Vì đây là tình huống máy khách tự ngắt kết nối, nên thường có thể bỏ qua.

2. Tốc độ gửi dữ liệu từ máy chủ lớn hơn tốc độ nhận của máy khách, dẫn đến dữ liệu tiếp tục tích tụ trong bộ đệm của máy chủ (workerman tạo một bộ đệm gửi cho mỗi máy khách), nếu kích thước bộ đệm vượt quá giới hạn (mặc định TcpConnection::$maxSendBufferSize là 1M) thì sẽ bị loại bỏ, kích hoạt sự kiện onError (nếu có) và làm tăng số lượng send_fail.

Ví dụ, sau khi thu nhỏ trình duyệt, JavaScript có thể tạm ngưng chạy, dẫn đến trình duyệt ngưng nhận dữ liệu từ máy chủ, dữ liệu tích tụ trong bộ đệm trong thời gian dài, vượt quá giới hạn sau mỗi lần gọi send sẽ tăng số lượng send_fail.

**Tóm lại:**

Thường thì không cần lo lắng về `send_fail` do việc máy khách ngắt kết nối.

Nếu do máy khách ngừng nhận dữ liệu dẫn đến send_fail, cần kiểm tra xem máy khách có hoạt động bình thường hay không.

Nếu tốc độ nhận dữ liệu từ máy chủ **liên tục** thấp hơn tốc độ gửi từ máy chủ, cần xem xét tối ưu quá trình kinh doanh hoặc tối ưu hóa hiệu suất của máy khách. Nếu vấn đề là do băng thông dẫn đến việc gửi không thông suốt, có thể xem xét tăng băng thông của máy chủ.

