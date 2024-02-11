# 安装说明
Workerman实际上就是一个PHP代码包，如果你的PHP环境已经装好，只需要把Workerman源代码或者demo下载下来即可运行。

**Composer安装：**
```sh
composer require workerman/workerman
```

> **注意**
> 有些composer代理镜像不全，使用以上命令`composer config -g --unset repos.packagist` 移除代理

# windows用户（必读）

从workerman3.5.3版开始workerman已经可以同时支持windows和linux系统。
windows用户需要配置下php环境变量。

 ` ===本页面以下仅适用于Linux环境workerman，windows用户请忽略=== `

# Linux系统环境检测
Linux系统可以使用以下脚本测试本机PHP环境是否满足Workerman运行要求。
 `curl -Ss https://www.workerman.net/check | php`

上面脚本如果全部显示ok，则代表满足Workerman要求，直接到[官网](https://www.workerman.net/)下载例子即可运行。

如果不是全部ok，则参考下面文档安装缺失的扩展即可。

（注意：检测脚本中没有检测event扩展，如果业务并发连接数大于1024必须安装event扩展，并且[优化Linux内核](../appendices/kernel-optimization.md)，扩展安装方法参照下面说明）

# 已有PHP环境安装缺失扩展

## 安装pcntl和posix扩展：

**centos系统**
如果php是通过yum安装的，则命令行运行 ```yum install php-process```即可安装pcntl和posix扩展。


如果安装失败或者php本身不是用yum安装的请参考手册[附录-安装扩展](../appendices/install-extension.md)一节中方法三源码编译安装。

**debian/ubuntu/mac os系统**
参考手册[附录-安装扩展](../appendices/install-extension.md)一节中方法三源码编译安装。


## 安装event扩展：
为了能支持更大的并发连接数，必须安装event扩展，并且[优化Linux内核](../appendices/kernel-optimization.md)。安装方法如下:

**centos系统**

1、安装event扩展依赖的libevent-devel包，命令行运行
```shell
yum install libevent-devel -y
# 如果无法安装，尝试使用下面的命令
# yum install libevent2-devel -y
```

2、安装event扩展，命令行运行
(event扩展要求PHP>=5.4)
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 时输入```no```回车，其它直接敲回车就行

3、运行```php --ini```找到并打开php.ini文件，在最后一行加入如下配置
```shell
extension=event.so
```

**debian/ubuntu系统安装**

1、安装event扩展依赖的libevent-dev包，命令行运行
```shell
apt-get install libevent-dev -y
# 如果无法安装，请尝试以下命令
# apt-get install libevent2-dev -y
```

2、安装event扩展，命令行运行
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 时输入```no```回车，其它直接敲回车就行


3、运行```php --ini```找到并打开php.ini文件，在最后一行加入如下配置
```shell
extension=event.so
```

**mac os 系统安装教程**

mac 系统一般作为开发机，不必安装event扩展。


# 全新系统安装（全新安装PHP+扩展）

## centos系统安装教程

1、命令行运行（此步骤包含了安装php-cli主程序以及pcntl、posix、libevent库及git程序）
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```

2、安装event扩展，命令行运行
(注意：event扩展要求PHP>=5.4)
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 时输入```no```回车，其它直接敲回车就行

3、运行```php --ini```找到并打开php.ini文件，在最后一行加入如下配置
```shell
extension=event.so
```
4、命令行运行（此步骤是通过github下载Workerman主程序）
```shell
git clone https://github.com/walkor/Workerman
```
5、参考[入门指引--简单开发实例部分](../getting-started/simple-example.md)写入口文件运行。
或者从[官网](https://www.workerman.net/)下载打包好的demo运行。


## debian/ubuntu系统安装教程

1、命令行运行（此步骤包含了安装php-cli主程序、libevent库及git程序）
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2、安装event扩展，命令行运行
(注意：event扩展要求PHP>=5.4)
```shell
pecl install event
```
注意提示：```Include libevent OpenSSL support [yes] :``` 时输入```no```回车，其它直接敲回车就行

3、运行```php --ini```找到并打开php.ini文件，在最后一行加入如下配置
```shell
extension=event.so
```


4、命令行运行（此步骤是通过github下载Workerman主程序）
```shell
git clone https://github.com/walkor/Workerman
```

5、参考[入门指引--简单开发实例部分](../getting-started/simple-example.md)写入口文件运行。
或者从[官网](https://www.workerman.net/)下载打包好的demo运行。

## mac os 系统安装教程
**方法1：** mac系统自带PHP Cli，但是可能缺少```pcntl```扩展。

1、参考手册[附录-安装扩展](../appendices/install-extension.md)一节中方法三源码编译安装```pcntl```扩展。

2、参考手册[附录-安装扩展](../appendices/install-extension.md)一节中方法四利用phpize安装```event```扩展（作为开发机此可省略）。

3、通过https://www.workerman.net/download/workermanzip 下载Workerman主程序，或者到[官网](https://www.workerman.net/)下载例子运行。

**方法2：** 通过```brew```命令安装php及对应扩展

1、命令行运行以下命令安装```brew```工具(如果已经安装过```brew```可以跳过此步骤)
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2、命令行运行以下命令安装```php```
```
brew install php
```

3、命令行运行以下命令安装```event```扩展
```
brew install php-event    
```

4、到[官网](https://www.workerman.net/)下载例子运行


# Event扩展说明
[Event扩展](https://php.net/manual/zh/book.event.php)不是必须的，当业务需要支撑大于1000的并发连接时，推荐安装Event，能够支持巨大的并发连接。如果业务并发连接比较低，例如1000以下并发连接，则可以不用安装。

## 常见问题
1、如果出现如下报错 `checking for include/event2/event.h... not found`，请先尝试删除libevent-dev(el)库安并装libevent2-dev(el)。
centos系统：yum remove libevent-devel && yum install libevent2-devel
debian/ubuntu系统：apt-get remove libevent-dev && apt-get install libevent2-dev

2、如果出现如下报错`NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`。
请更改event.so 和socket.so的加载顺序，既在php.ini中将 `extension=socket.so` 写在 `extension=event.so` 前面，让socket扩展先加载。


