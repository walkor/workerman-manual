# Disabling Function Check

Use this script to check for any disabled functions. Run the command in the command line `curl -Ss https://www.workerman.net/check | php`

If you see the prompt `Function function_name may be disabled. Please check disable_functions in php.ini`, it means the functions that workerman depends on are disabled. They need to be enabled in the php.ini file for workerman to function properly.
To enable them, choose one of the two methods below.

## Method One: Script Enablement

Execute the script `curl -Ss https://www.workerman.net/fix | php` to enable the disabled functions.

## Method Two: Manual Enablement

**Steps:**

1. Run `php --ini` to find the location of the php.ini file used by PHP CLI.

2. Open php.ini and locate the `disable_functions` directive to enable the corresponding functions.

**Dependent Functions**

To use workerman, the following functions need to be enabled:

```
stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
