# Reasons for send_fail in status

**Symptom:**

When running the status command, if there are cases of send_fail, what is the reason?

**Answer:**

In general, a send_fail is not usually a major issue, and it is usually caused by the client actively closing the connection or being unable to receive data.

There are two reasons for a send_fail:

1. When the send interface is called to send data to the client, it is found that the client has already disconnected, and then the send_fail count is increased by 1. Because the client actively disconnected, this is a normal phenomenon that can generally be ignored.

2. The server's data transmission speed is greater than the client's reception speed, causing the data to constantly accumulate in the server's buffer (Workerman establishes a send buffer for each client). If the buffer size exceeds the limit (TcpConnection::$maxSendBufferSize is 1M by default), the data will be discarded, triggering the onError event (if exists), and resulting in an increase of the send_fail count.

For example, when a browser is minimized, JavaScript may pause, causing the browser to pause receiving server data. If data accumulates in the buffer for a long time and exceeds the limit, each call to send will increase the send_fail count.

**Conclusion:**

`send_fail` caused by a client disconnecting is generally nothing to worry about.

If `send_fail` is caused by the client stopping receiving data, it is necessary to check if the client is functioning properly.

If the client's data reception speed continues to be lower than the server's sending speed, it is necessary to consider optimizing the business process or improving the client's performance. If the slow transmission is due to bandwidth constraints, consider increasing the server's bandwidth.
