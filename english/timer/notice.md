# Note
## Precautions for Using Timers
1. Timers can only be added in the `onXXXX` callback. Global timers are recommended to be set in the `onWorkerStart` callback, and timers for a specific connection are recommended to be set in the `onConnect` callback.

2. The added timed tasks are executed in the current process (no new process or thread will be started). If a task is heavy (especially tasks involving network I/O), it may cause the process to block and temporarily unable to handle other business. It is best to run time-consuming tasks in a separate process, for example, by setting up one or more Worker processes.

3. If the current process is busy with other tasks or if a task has not completed within the expected time, and it is time for the next run period, the current task will wait for the completion of the current task before it runs. This may cause the timer to not run at the expected time interval. In other words, the business of the current process is executed in a serial manner, while in multiple processes, the tasks between processes are executed in parallel.

4. It is important to note that setting timers for multiple processes may cause concurrency issues. For example, the following code will print 5 times per second.
```php
$worker = new Worker();
// 5 processes
$worker->count = 5;
$worker->onWorkerStart = function(Worker $worker) {
    // 5 processes, each with a timer like this
    Timer::add(1, function(){
        echo "hi\r\n";
    });
};
Worker::runAll();
```
If you only want one process to run the timer, refer to [Timer::add Example 2](add.md).

5. There may be an error of around 1 millisecond.

6. Timers cannot be deleted across processes. For example, a timer set by process A cannot be directly deleted by calling the Timer::del interface in process B.

7. The timer IDs generated in different processes may overlap, but the timer IDs generated within the same process will not overlap.

8. Changing the system time will affect the behavior of the timer, so it is recommended to restart after changing the system time.
