# Callback Attributes

Workerman supports event loop driven by callbacks, and there are several callback attributes that can be used to configure the behavior of the event loop.

### `onConnect`

`onConnect` is called when a new connection is established. You can use this callback to perform some initialization tasks for the new connection.

### `onMessage`
`onMessage` is called when new data is received from the client. You can process the received data in this callback.

### `onClose`

`onClose` is called when a connection is closed. You can perform cleanup tasks related to the closed connection in this callback.

### `onError`

`onError` is called when an error occurs during the processing of a connection. You can handle the error and perform necessary actions in this callback.

### `onBufferFull`

`onBufferFull` is called when the send buffer of the connection is full. You can use this callback to pause receiving data from clients until the send buffer becomes available.

### `onBufferDrain`

`onBufferDrain` is called when the send buffer of the connection becomes available again after being full. You can use this callback to resume receiving data from clients.

These callback attributes can be set on worker instances to define the behavior of the event loop for handling new connections, data reception, connection closure, errors, and buffer status.
