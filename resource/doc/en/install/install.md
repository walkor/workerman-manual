# Installation Instructions
Workerman is actually a PHP code package. If your PHP environment is already installed, you just need to download the Workerman source code or demo to run it.

**Composer Installation:**
```sh
composer require workerman/workerman
```

> **Note:**
> Some composer proxy mirrors are incomplete. Use the command `composer config -g --unset repos.packagist` to remove the proxy.

# Windows Users (Must Read)
Starting from version 3.5.3, Workerman can now support both Windows and Linux systems. Windows users need to configure the PHP environment variables.

`===This page below is only applicable to the Linux environment workerman, windows users please ignore===`

# Linux System Environment Detection
Linux systems can use the following script to test whether the local PHP environment meets the requirements for running Workerman.
 `curl -Ss https://www.workerman.net/check | php`

If all the OKs are displayed in the script above, it means that the requirements for Workerman are met, and you can directly download the example from the [official website](https://www.workerman.net/) to run.

If not all OK, you can refer to the following documentation to install the missing extensions.

(Note: The detection script does not check the event extension. If the business concurrent connection number is greater than 1024, it is necessary to install the event extension and [optimize the Linux kernel](../appendices/kernel-optimization.md). The installation method for the extension is as described below.)

# Installing Missing Extensions in Existing PHP Environments

## Installing the pcntl and posix Extensions:

**Centos Systems**
If PHP is installed using yum, run the command line `yum install php-process` to install the pcntl and posix extensions.

If the installation fails or PHP itself is not installed using yum, please refer to method three of the manual [Appendix - Installing Extensions](../appendices/install-extension.md) for source code compilation and installation.

**Debian/Ubuntu/Mac OS Systems**
Refer to method three of the manual [Appendix - Installing Extensions](../appendices/install-extension.md) for source code compilation and installation.

## Installing the event Extension:
In order to support a larger number of concurrent connections, it is necessary to install the event extension and [optimize the Linux kernel](../appendices/kernel-optimization.md). The installation method is as follows:

**Centos Systems**

1. Install the libevent-devel package, which is a dependency for the event extension, by running the following command line:
```shell
yum install libevent-devel -y
# If unable to install, try using the following command
# yum install libevent2-devel -y
```

2. Install the event extension by running the following command line:
(the event extension requires PHP>=5.4)
```shell
pecl install event
```
Note: When prompted with `Include libevent OpenSSL support [yes] :` enter `no` and press Enter, for all other prompts just press Enter.

3. Run `php --ini`, find and open the php.ini file, and add the following configuration at the end:
```shell
extension=event.so
```

**Debian/Ubuntu Systems Installation**

1. Install the libevent-dev package, which is a dependency for the event extension, by running the following command line:
```shell
apt-get install libevent-dev -y
# If unable to install, try the following command
# apt-get install libevent2-dev -y
```

2. Install the event extension by running the following command line:
```shell
pecl install event
```
Note: When prompted with `Include libevent OpenSSL support [yes] :` enter `no` and press Enter, for all other prompts just press Enter.

3. Run `php --ini`, find and open the php.ini file, and add the following configuration at the end:
```shell
extension=event.so
```

**Mac OS System Installation Tutorial**

Mac systems are generally used as development machines and do not necessarily require the event extension.

# Installation on a Fresh System (Installing PHP + Extensions)

## Centos System Installation Tutorial

1. Run the command line (this step includes installing the php-cli main program, pcntl, posix, libevent library, and git program)
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```

2. Install the event extension by running the following command line:
(Note: the event extension requires PHP>=5.4)
```shell
pecl install event
```
Note: When prompted with `Include libevent OpenSSL support [yes] :` enter `no` and press Enter, for all other prompts just press Enter.

3. Run `php --ini`, find and open the php.ini file, and add the following configuration at the end:
```shell
extension=event.so
```

4. Run the command line (this step downloads the Workerman main program from GitHub)
```shell
git clone https://github.com/walkor/Workerman
```

5. Refer to the [Getting Started - Simple Development Example](../getting-started/simple-example.md) section to write the entry file for running, or download the packaged demo from the [official website](https://www.workerman.net/) to run.

## Debian/Ubuntu System Installation Tutorial

1. Run the command line (this step includes installing the php-cli main program, libevent library, and git program)
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2. Install the event extension by running the following command line:
(Note: the event extension requires PHP>=5.4)
```shell
pecl install event
```
Note: When prompted with `Include libevent OpenSSL support [yes] :` enter `no` and press Enter, for all other prompts just press Enter.

3. Run `php --ini`, find and open the php.ini file, and add the following configuration at the end:
```shell
extension=event.so
```

4. Run the command line (this step downloads the Workerman main program from GitHub)
```shell
git clone https://github.com/walkor/Workerman
```

5. Refer to the [Getting Started - Simple Development Example](../getting-started/simple-example.md) section to write the entry file for running, or download the packaged demo from the [official website](https://www.workerman.net/) to run.

## Mac OS System Installation Tutorial
**Method 1:** Mac systems come with PHP Cli, but may be missing the `pcntl` extension.

1. Refer to method three in the manual [Appendix - Installing Extensions](../appendices/install-extension.md) for source code compilation and installation of the `pcntl` extension.

2. Refer to method four in the manual [Appendix - Installing Extensions](../appendices/install-extension.md) to use phpize to install the `event` extension (this can be omitted as it is a development machine).

3. Download the Workerman main program through https://www.workerman.net/download/workermanzip, or download examples from the [official website](https://www.workerman.net/) to run.

**Method 2:** Install php and the corresponding extensions using the `brew` command.

1. Run the following command line to install the `brew` tool (you can skip this step if `brew` is already installed)
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. Run the following command line to install `php`
```shell
brew install php
```

3. Run the following command line to install the `event` extension
```shell
brew install php-event    
```

4. Download examples from the [official website](https://www.workerman.net/) to run.

# Event Extension Explanation
The [Event extension](https://php.net/manual/zh/book.event.php) is not mandatory, but is recommended to be installed for business requirements that support more than 1000 concurrent connections, as it can support huge numbers of concurrent connections. If the business requires a lower level of concurrent connections, such as below 1000 concurrent connections, it may not need to be installed.

## Common Problems
1. If the following error occurs: `checking for include/event2/event.h... not found`, please try to delete the libevent-dev(el) library and install libevent2-dev(el) instead.
CentOS System: `yum remove libevent-devel && yum install libevent2-devel`
Debian/Ubuntu System: `apt-get remove libevent-dev && apt-get install libevent2-dev`

2. If the following error occurs: `NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`.
Please change the loading order of `event.so` and `socket.so` by writing `extension=socket.so` in `php.ini` before `extension=event.so` to load the socket extension first.
