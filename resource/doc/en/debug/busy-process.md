# Debugging Busy Processes
Sometimes when we use the `php start.php status` command, we can see processes in a `busy` state, indicating that the corresponding process is handling business. Under normal circumstances, the process should return to the `idle` state once the business is completed, which normally should not cause any issues. However, if a process remains in the `busy` state without transitioning to the `idle` state, it indicates that there may be blocking or infinite loops within the process, and the following methods can be used to locate these issues.

## Using strace+lsof Commands for Location

**1. Find the PID of the busy process from the status**
After running `php start.php status`, it will display as follows:
![](../images/d1903ed65ef2f3b0850e84ccbedc52aa.png)
In the image, the PIDs of the `busy` processes are `11725` and `11748`.

**2. Trace the Process using strace**
Select a process PID (e.g., `11725`), and run `strace -ttp 11725`. The output will be as follows:
![](../images/7ce9f36da926f670949609dcdc593ab4.png)
It can be observed that the process is continuously looping the system call `poll([{fd=16, events=...`. This indicates that it is waiting for the descriptor with fd 16 to return data.

If no system calls are displayed, keep the current terminal open and open a new terminal. Run `kill -SIGALRM 11725` (send a alarm signal to the process), then check if the strace terminal responds and if it is blocked on a system call. If there are still no system calls displayed, it is very likely that the program is stuck in a business-oriented infinite loop. Refer to the bottom of the page under item 2 for resolution.

If the system is blocked on the `epoll_wait` or `select` system call, this indicates that the process is already in the `idle` state.

**3. View Process Descriptors using lsof**
Run `lsof -nPp 11725` and the output will be as follows:
![](../images/27bd629c3a1ac93f9f4b535d01df2ac1.png)
The descriptor 16 corresponds to the record `16u` (last line), and it can be seen that the descriptor `fd=16` is a TCP connection with a remote address of `101.37.136.135:80`. This indicates that the process is likely accessing an HTTP resource, and the continuous `poll([{fd=16, events=...` loop is waiting for the HTTP service to return data, explaining why the process is in a `busy` state.

**Resolution:**
Once the location of the blocking process is known, it becomes easier to resolve the issue. In the above example, after locating it, it appears that the business is calling a url that takes a long time to return data, causing the process to wait indefinitely. In such cases, it is advisable to contact the provider of the URL to identify the reason for the slow response. Meanwhile, set a timeout parameter when making the curl call, for example, timeout after 2 seconds have passed without a response, to avoid a long blocking, which could cause the process to be in a `busy` state for around 2 seconds.

## Other Reasons for Long Time Busy Processes
Apart from process blockages, the following reasons can also cause a process to remain in a `busy` state.

**1. Fatal errors in business causing processes to continually exit**
**Phenomenon:** In this situation, the system load is relatively high, and the `load average` in the `status` is 1 or higher. The `exit_count` for the process is continuously increasing.
**Resolution:** Run Workerman in debug mode (`php start.php start` without `-d`) to view the business error and resolve it.

**2. Infinite loops in the code**
**Phenomenon:** In `top`, it can be seen that the busy process consumes a lot of CPU, and the command `strace -ttp pid` does not print any system call information.
**Resolution:** Refer to the Bird Brother's article on locating the issue using [gdb and PHP source code](https://www.laruence.com/2011/12/06/2381.html). The general steps are as follows:
1. Run `php -v` to check the version.
2. [Download the corresponding PHP version source code](https://www.php.net/releases/).
3. Run `gdb --pid=busy process pid`.
4. Run `source php source code path/.gdbinit`.
5. Run `zbacktrace` to print the call stack.
The call stack printed in the last step will indicate the location of the infinite loop in the PHP code.
Note: If `zbacktrace` does not print the call stack, it is possible that your PHP was not compiled with the `-g` parameter. In this case, recompile PHP and restart Workerman for location.

**3. Infinite addition of timers**
Continuous addition of timers in the business code without deletion causes the process to run an increasing number of timers, ultimately resulting in an infinite running of timers and a `busy` process. For example, the following code:
```php
$worker = new Worker;
$worker->onConnect = function($con){
    Timer::add(10, function(){});
};
Worker::runAll();
```
In the above code, a timer is added when a client connects, but there is no logic in the entire business code to delete the timer. Consequently, as time passes, the process will continuously add timers, leading to an indefinite running of timers and a `busy` state.
The correct code is:
```php
$worker = new Worker;
$worker->onConnect = function($con){
    $con->timer_id = Timer::add(10, function(){});
};
$worker->onClose = function($con){
    Timer::del($con->timer_id);
};
Worker::runAll();
```
