# Environment Requirements

## For Windows Users
As of version 3.5.3, Workerman has been able to support both Linux and Windows systems simultaneously.

1. Requires PHP>=5.4 and the PHP environment variables properly configured.
2. The Windows version of Workerman does not depend on any extensions.
3. Installation and usage restrictions can be found [**here**](https://www.workerman.net/windows).
4. Due to various usage restrictions for Workerman on Windows, it is recommended to use Linux for production environments, with Windows recommended only for development environments.

 ``` ====The content below on this page is only applicable to Linux users, Windows users please ignore. ====```

## For Linux Users (Including Mac OS)
Linux users can only use the Linux version of Workerman.

1. Requires PHP>=5.4 and installation of pcntl and posix extensions.
2. Recommended to install the event extension, but not mandatory (note that the event extension requires PHP>=5.4).

### Linux Environment Check Script
Linux users can run the following script to check if the local environment meets the requirements for Workerman:

```curl -Ss https://www.workerman.net/check | php```

If the script displays "ok" for all checks, then the Workerman runtime environment is satisfied.

(Note: The script does not check for the event extension. If the concurrent connection count exceeds 1024, it is recommended to install the event extension. Installation instructions can be found in the following section.)

## Detailed Explanation

### About PHP-CLI

Workerman runs based on the [PHP Command Line Interface (PHP-CLI)](https://php.net/manual/en/features.commandline.php) mode. PHP-CLI is an independent executable program, separate from PHP-FPM or Apache's MOD-PHP, and does not conflict or depend on each other.

### About Extensions Required by Workerman

1. [pcntl Extension](https://www.php.net/manual/en/book.pcntl.php)

The pcntl extension is an important extension for process control in PHP under the Linux environment. Workerman utilizes its features such as process creation, signal control, timers, and process status monitoring. This extension is not supported on the Windows platform.

2. [posix Extension](https://www.php.net/manual/en/book.posix.php)

The posix extension enables PHP in a Linux environment to call interfaces provided by the system through the [POSIX standard](https://en.wikipedia.org/wiki/POSIX). Workerman mainly uses its related interfaces to implement daemonization and user group control. This extension is not supported on the Windows platform.

3. [Event Extension](https://php.net/manual/en/book.event.php) or [libevent Extension](https://www.php.net/manual/en/book.libevent.php)

The event extension allows PHP to use advanced event handling mechanisms such as Epoll and Kqueue, significantly improving CPU utilization for Workerman in high-concurrency connections. It is crucial for applications involving high-concurrency long connections. The libevent extension (or event extension) is not mandatory. If not installed, PHP's native Select event handling mechanism will be used by default.

## How to Install Extensions

Refer to the chapter on [Installing Extensions](../appendices/install-extension.md)
