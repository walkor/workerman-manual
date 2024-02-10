# Stop Failure

## Phenomenon:
Running `php start.php stop` prompts `stop fail`.

### The first possibility
The premise is that workerman was started in debug mode, and the developer pressed `ctrl z` in the terminal to send a `SIGSTOP` signal to workerman, causing workerman to enter the background and suspend, so it cannot respond to the stop command (`SIGINT` signal).

**Solution:**
Enter `fg` in the terminal where workerman is started (send `SIGCONT` signal), then press Enter to bring workerman back to the foreground and press `ctrl c` (send `SIGINT` signal) to stop workerman.

If unable to stop, try running the following two commands:
```shell
killall -9 php
```
```shell
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

### The second possibility
The user running the stop command is different from the user who started workerman, meaning the stop user does not have permission to stop workerman.

**Solution:**
Switch to the user who started workerman, or use a user with higher privileges to stop workerman.

### The third possibility
The saved PID file of the workerman main process is deleted, causing the script to not find the PID process and resulting in a stop failure.

**Solution:**
Save the PID file in a secure location, see documentation [Worker::$pidFile](../worker/pid-file.md).

### The fourth possibility
The process corresponding to the workerman main process PID file is not the workerman process.

**Solution:**
Open the PID file of the workerman main process to view the main process PID, the PID file is by default located in the same directory as Workerman. Run the command `ps aux | grep main-process-pid` to check if the corresponding process is a Workerman process. If not, it may be due to a server restart, causing the PID saved by workerman to be an outdated PID that is currently being used by another process, resulting in a stop failure. In this case, simply delete the PID file.

### The fifth possibility
The grpc extension is installed, but the corresponding environment variables are not set for the grpc extension, causing an additional mounted process to be forked when started, which leads to a failure when stopping.
