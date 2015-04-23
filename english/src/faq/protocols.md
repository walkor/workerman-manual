# WorkerMan支持哪些协议

WorkerMan在接口上支持各种协议，只要符合```ConnectionInterface```接口即可（参见定制通讯协议章节）。

为了方便开发者，WorkerMan实现了HTTP协议、WebSocket协议以及非常简单的Text文本协议。开发者可以直接使用这些协议，不必再二次开发。如果这些协议都不满足需要，开发者可以参照定制协议章节实现自己的协议。
