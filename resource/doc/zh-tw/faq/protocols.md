# Workerman支持哪些協議

Workerman在接口上支持各種協議，只要符合```ConnectionInterface```介面即可（參見定制通訊協議章節）。

為了方便開發者，Workerman提供了HTTP協議、WebSocket協議以及非常簡單的Text文本協議、可用於二進制傳輸的frame協議。開發者可以直接使用這些協議，不必再二次開發。如果這些協議都不滿足需要，開發者可以參照定制協議章節實現自己的協議。

開發者也可以直接基於tcp或者udp協議。

協議使用示例
```php
// http協議
$worker1 = new Worker('http://0.0.0.0:1221');
// websocket協議
$worker2 = new Worker('websocket://0.0.0.0:1222');
// text文本協議（telnet協議）
$worker3 = new Worker('text://0.0.0.0:1223');
// frame文本協議（可用於二進制數傳輸）
$worker3 = new Worker('frame://0.0.0.0:1223');
// 直接基於tcp傳輸
$worker4 = new Worker('tcp://0.0.0.0:1224');
// 直接基於udp傳輸
$worker5 = new Worker('udp://0.0.0.0:1225');
```
