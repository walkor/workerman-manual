# Supported Protocols in WorkerMan

WorkerMan supports various protocols at the interface level, as long as they conform to the `ConnectionInterface` interface (see the Custom Communication Protocol section).

For the convenience of developers, WorkerMan provides support for the HTTP protocol, WebSocket protocol, as well as a very simple text protocol and a frame protocol for binary transmission. Developers can directly use these protocols without the need for secondary development. If none of these protocols meet the requirements, developers can refer to the Custom Protocol section to implement their own protocol.

Developers can also directly use the TCP or UDP protocols.

Protocol usage examples
```php
// HTTP protocol
$worker1 = new Worker('http://0.0.0.0:1221');
// WebSocket protocol
$worker2 = new Worker('websocket://0.0.0.0:1222');
// Text protocol (telnet protocol)
$worker3 = new Worker('text://0.0.0.0:1223');
// Frame protocol (for binary transmission)
$worker3 = new Worker('frame://0.0.0.0:1223');
// Directly based on TCP transmission
$worker4 = new Worker('tcp://0.0.0.0:1224');
// Directly based on UDP transmission
$worker5 = new Worker('udp://0.0.0.0:1225');
```
