# Channel Distributed Communication Component
**``` (Requires Workerman version >= 3.3.0) ```**

Source code: https://github.com/walkor/Channel

Channel is a distributed communication component used for inter-process or inter-server communication.

## Features
1. Based on the publish-subscribe model
2. Non-blocking IO

## Principle
Channel consists of Channel/Server as the server end and Channel/Client as the client end.

Channel/Client connects to Channel/Server through the connect interface and maintains a long connection.

Channel/Client informs Channel/Server of which events it is interested in by calling the on interface and registers event callback functions (callbacks occur in the process where Channel/Client is located).

Channel/Client publishes a certain event and its related data to Channel/Server through the publish interface.

After receiving the event and data, Channel/Server distributes them to the Channel/Clients that are interested in this event.

Upon receiving the event and data, Channel/Client triggers the callback set by the on interface.

Channel/Client will only receive events it is interested in and trigger callbacks.

## Installation
`composer require workerman/channel`

## Note
Channel can only be used in the Workerman environment and cannot be used in the php-fpm environment.
