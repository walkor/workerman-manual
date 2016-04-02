# 序言

[PHP](http://www.php.net)是一种被广泛应用的开源脚本语言，绝大多数开发者使用PHP做基于Web的应用程序，并且有了很多非常知名的Web框架，如laravel、Yii、thinkphp等。

传统的PHP应用程序基本上是在Apache等Web容器中运行的，浏览器与Web容器采用HTTP协议通信，然而在很多实际项目中HTTP协议无法满足我们的需求，尤其是在服务端和客户端要保持长连接，做实时双向通讯时，HTTP协议显得力不从心。例如即时IM通讯，游戏服务器通讯，与硬件传感器通讯等等，开发这些应用程序我们无法直接使用nginx/apache + PHP来实现，也更无法使用传统的PHP框架来做。这就迫使我们寻找一种新的解决方案，这时候WorkerMan就是你的最佳选择。

WorkerMan是一款纯PHP开发的开源的高性能的PHP socket服务器框架，基于WorkerMan开发者可以开发出各种网络服务器，例如基于websocket的服务器、游戏服务器、移动通讯服务器、智能家居服务端、物联网服务、web服务器、RPC服务器等等。几乎任何基于TCP/UDP通讯的服务端都可以用WorkerMan来开发。WorkerMan使得开发者摆脱PHP只能用于Web开发的束缚，向更广阔的前景发展。

# 本手册作用范围
WorkerMan有分为Linux版本[WorkerMan](https://github.com/walkor/workerman)和Windows版本[WorkerMan-for-win](https://github.com/walkor/workerman-for-win)，windows版本说明参见[这里](http://www.workerman.net/windows)。Linux版本用于开发调试及正式环境部署，windows版本只适合开发调试使用。

# 客户端

WorkerMan的通信协议是开放的，又是可定制的，因此，理论上WorkerMan可以与使用任意协议的任意平台的客户端进行通信。当用户开发客户端时，可以根据相应的通信协议完成与服务端的通信。



