# Reasons for Client Connection Failure

When a client connection fails, there are generally two types of errors reported: `connection refuse` and `connection timeout`.

## Connection Refuse

This is usually due to the following reasons:
1. The client is connecting to the wrong port.
2. The client is using the wrong domain name or IP address to connect.
3. If the client is using a domain name for the connection, the domain name may be pointing to the wrong server IP.
4. The server is using a CDN or other acceleration proxy, resulting in the actual IP of the connection not matching the expected IP.
5. The server is not running or the port is not being listened to.
6. The use of network proxy software.
7. The server listening IP and the access address are not in the same address range. For example, if the server is listening on 127.0.0.1, the client can only connect via 127.0.0.1 and cannot connect via a local area network IP or external IP. It is recommended to set the listening address to 0.0.0.0 so that connections can be made from the local machine, local network, and external network.

## Connection Timeout

This is usually due to the following reasons:
1. The server firewall is blocking the connection. Temporarily disabling the firewall may resolve the issue.
2. If using a cloud server, the security group may also block the connection establishment. In such cases, the corresponding port needs to be opened in the management console.
3. If using a control panel such as Baota, the corresponding port needs to be opened in the control panel.
4. The server does not exist or is not running.
5. If the client is using a domain name for the connection, the domain name may be pointing to the wrong server IP.
6. The client is accessing an internal IP of the server, and the client and server are not in the same local area network.

## Cannot Assign Requested Address

When acting as a client, each connection attempt requires a local temporary port. A server typically has around 20,000 to 30,000 available temporary ports. If the number of connections initiated to a specific server exceeds this value, the system will be unable to assign an available port, resulting in this error.
To resolve this, the number of local temporary ports can be increased by changing the kernel parameter in `/etc/sysctl.conf` to `net.ipv4.ip_local_port_range`. For example, setting it to `10000 65535` (which increases the local port range to 10,000-65,535) and running `sysctl -p` will take effect.
Additionally, when connections are closed, they enter a TIME_WAIT state and continue to occupy the corresponding local port for a period of time. Therefore, initiating a large number of short-lived connections (exceeding 20,000-30,000) in a short period of time will also result in the `Cannot assign requested address` error. In such cases, resolving it involves setting the kernel to quickly recycle TIME_WAIT, as detailed in the [Kernel Optimization](https://www.workerman.net/doc/workerman/appendices/kernel-optimization.html) guide.
 
> **Note**
> The limitation on the number of local ports only applies to the client. The server does not have a local port limitation; as long as the resources are sufficient, the server can maintain an almost unlimited number of connections.

## Other Errors
If the reported error is not `connection refuse` or `connection timeout`, then it is generally due to the following reasons:

**1. The client is using a communication protocol that is not consistent with the server.**
For example, if the server is using the HTTP communication protocol, the client cannot connect using the WebSocket communication protocol. If the client is using the WebSocket protocol, the server must also use the WebSocket protocol. If the server is an HTTP service, the client must use the HTTP protocol to access it.

The principle here is similar to using English to communicate with an English person or using Japanese to communicate with a Japanese person. In this case, the language is analogous to the communication protocol; both the client and server must use the same language to communicate, otherwise communication is not possible.

**Common errors resulting from inconsistent communication protocols include:**

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: Unexpected response code: xxx

> WebSocket connection to 'ws://xxx.com:xx/' failed: Error during WebSocket handshake: net::ERR_INVALID_HTTP_RESPONSE

**Solution:**
From the above error messages, it is evident that the client is using a WebSocket connection, which requires the server to also use the WebSocket protocol. In the server listening part of the code, the WebSocket protocol must be specified to enable communication, as shown below.

If using gatewayWorker, the listening part of the code is similar to:
```php
// WebSocket protocol, so clients can connect using ws://...
$gateway = new Gateway('websocket://0.0.0.0:xxxx');
```
If using Workerman, it is:
```php
// WebSocket protocol, so clients can connect using ws://...
$worker = new Worker('websocket://0.0.0.0:xxxx');
```
