# Workerman startup failure

## Symptom 1
Error similar to the following after startup:
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php on line xxxx

```
**Keywords**: ```Address already in use```

**Root cause**: The port is already in use and cannot be started.

#### Solution 1


You can use the command ```netstat -anp | grep port number``` to find out which program is using the port.
Then stop the corresponding program to release the port.

#### Solution 2
If you cannot stop the program using the port, you can solve it by changing the port used by Workerman.

#### Solution 3
If the port is occupied by Workerman and cannot be stopped by the stop command (usually due to the loss of the pid file or the main process being killed by the developer), you can kill the Workerman process by running the following two commands.

```shell
killall php
ps aux|grep WorkerMan|awk '{print $2}'|xargs kill -9
```

#### Solution 4

If there is indeed no program listening on this port, it may be because the developer has set up two or more listeners in the workerman, and the ports they listen on are the same, causing conflicts. Please check if the startup script is listening on the same port.

#### Solution 5

Check if the program has enabled reusePort, and try disabling reusePort.

## Symptom 2
Error similar to the following after startup:
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php on line xxxx
```
or
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (The requested address is invalid in its context) in ...workerman/Worker.php on line xxxx
```
**Keywords**: `Cannot assign requested address` or `The requested address is invalid`

**Failure reason**:

The IP parameter in the startup script is written incorrectly, it is not the local IP. Please fill in the local IP or fill in ```0.0.0.0``` (which means to listen on all local IPs) to solve it.

**Tip**: On a Linux system, you can use the ```ifconfig``` command to view all the IP addresses of the local network cards. If you are a cloud server (such as Alibaba Cloud/Tencent Cloud) user, be aware that your public IP may actually be a proxy IP (such as Alibaba Cloud's dedicated network), and the public IP does not belong to the current server, so it cannot be listened on via the public IP. Although listening on the public IP is not possible, you can still use 0.0.0.0 to bind.

## Symptom 3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**Failure reason**:

The stream_socket_server function is disabled in php.ini.

**Solution**:

1. Run ```php --ini``` to find the php.ini file.

2. Open the php.ini file, find the disable_functions item, and remove the disabled item for stream_socket_server.

## Symptom 4
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```
**Failure reason**:

On Linux, if the port is less than 1024, root permission is required to listen on the port.

**Solution**:

Use a port greater than 1024 or start the service with the root user.
