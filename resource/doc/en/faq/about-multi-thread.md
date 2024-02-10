# Does Workerman support multi-threading?

Workerman has a [MT multi-threading version](https://github.com/walkor/workerman-MT) which depends on the [pthreads extension](https://php.net/manual/zh/book.pthreads.php). However, due to the instability of the pthreads extension, this multi-threading version of Workerman is no longer maintained.

**Currently, Workerman and its peripheral products are all based on multi-process single-threading.**
