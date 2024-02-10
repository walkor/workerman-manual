# Using Workerman to push data to clients in other projects

**Question:**

I have a regular web project and I want to call the Workerman interface in this project to push data to clients.

**Answer:**

**Based on Workerman, you can refer to the following links:**

- [Channel component push example](../components/channel-examples.md) (supports multi-process/server clusters, requires downloading the Channel component)

- [Push based on Worker](https://www.workerman.net/q/508) (single process, simplest)

**Based on webman, refer to the following link:**
- [webman push plugin](https://www.workerman.net/plugin/2)

**Based on GatewayWorker, refer to the following link:**

- [Pushing through GatewayWorker in other projects](https://www.workerman.net/doc/gateway-worker/push-in-other-project.html) (supports multi-process/server clusters, supports grouping, group broadcasts, and individual sends)

**Based on PHPSocket.IO, refer to the following link:**

- [Web message push](https://www.workerman.net/web-sender) (default single process, based on socket.io, best browser compatibility)
