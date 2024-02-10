# Workerman是否支持多線程？

Workerman有一個依賴[pthreads擴展](https://php.net/manual/zh/book.pthreads.php)的[MT多線程版本](https://github.com/walkor/workerman-MT)，但由於pthreads擴展尚不夠穩定，所以這個Workerman多線程版本已經不再維護。

**目前Workerman及其周邊產品都是基於多進程單線程的。**
