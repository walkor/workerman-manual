# Kết nối AsyncUdpConnection

**(Yêu cầu workerman>=3.0.8)**

AsyncUdpConnection có thể được sử dụng như là một client udp để giao tiếp với server udp từ xa.

Thực ra, udp là không có kết nối, nhưng để dễ sử dụng, đây giữ nguyên tên và giao diện với quy tắc đặt tên và AsyncTcpConnection cơ bản.

**Chú ý: Khác với AsyncTcpConnection, AsyncUdpConnection không hỗ trợ các thuộc tính hoặc phương thức sau:**
1. Không có thuộc tính connection->id
2. Không có thuộc tính connection->worker
3. Không có thuộc tính connection->transport
4. Không có thuộc tính connection->maxSendBufferSize
5. Không có thuộc tính connection->defaultMaxSendBufferSize
6. Không có thuộc tính connection->maxPackageSize
7. Không có callback connection->onBufferFull
8. Không có callback connection->onBufferDrain
9. Không có callback connection->onError
10. Không có interface connection->destroy()
11. Không có interface connection->pauseRecv()
12. Không có interface connection->resumeRecv()
13. Không có interface connection->pipe()
14. Không có interface connection->reconnect()

**Các thuộc tính hoặc phương thức được hỗ trợ bởi AsyncUdpConnection:**
1. Hỗ trợ thuộc tính connection->protocol
2. Hỗ trợ callback connection->onMessage
3. Hỗ trợ phương thức connection->connect()
4. Hỗ trợ phương thức connection->send()
5. Hỗ trợ phương thức connection->getRemoteIp()
6. Hỗ trợ phương thức connection->getRemotePort()
7. Hỗ trợ callback connection->onClose
Chú ý: Do tcp dựa trên kết nối, thường khi bất kỳ bên nào gọi close để cắt kết nối, cả hai bên đều có thể kích hoạt onClose. Nhưng udp không có kết nối, việc gọi phương thức connection->close() chỉ kích hoạt callback onClose cục bộ và không thể kích hoạt callback onClose từ phía đối tác.
