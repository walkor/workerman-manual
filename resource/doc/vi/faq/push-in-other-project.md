# Sử dụng Workerman trong dự án khác để đẩy dữ liệu đến máy khách

**Câu hỏi:**

Tôi có một dự án web thông thường và muốn gọi API của Workerman trong dự án này để đẩy dữ liệu đến máy khách.

**Trả lời:**

**Dựa trên Workerman có thể tham khảo các liên kết sau:**

- [Ví dụ đẩy thông tin sử dụng thành phần Channel](../components/channel-examples.md) (hỗ trợ nhiều tiến trình/cụm máy chủ, cần tải về thành phần Channel)

- [Đẩy thông tin dựa trên Worker](https://www.workerman.net/q/508) (một tiến trình, đơn giản nhất)

**Dựa trên webman tham khảo liên kết sau**
- [Trình cắm đẩy webman](https://www.workerman.net/plugin/2)

**Dựa trên GatewayWorker, tham khảo liên kết sau**

- [Đẩy thông tin thông qua GatewayWorker trong dự án khác](https://www.workerman.net/doc/gateway-worker/push-in-other-project.html) (hỗ trợ nhiều tiến trình/cụm máy chủ, hỗ trợ nhóm, gửi nhóm, gửi đơn lẻ)

**Dựa trên PHPSocket.IO tham khảo liên kết sau**

- [Đẩy tin nhắn web](https://www.workerman.net/web-sender) (mặc định một tiến trình, dựa trên socket.io, tương thích tốt nhất với trình duyệt)
