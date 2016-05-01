# Workerman是否支持多线程？

Workerman有一个依赖[pthreads扩展](http://php.net/manual/zh/book.pthreads.php)的[MT多线程版本](https://github.com/walkor/workerman-MT)，但是由于pthreads扩展还不够稳定，所以这个Workerman多线程版本已经不再维护。

**目前Workerman及其周边产品都是基于多进程单线程的。**
