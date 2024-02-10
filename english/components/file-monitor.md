# File Monitoring Component

**Background:**

Workerman runs in memory, which can avoid repeated disk reading and PHP recompiling to achieve the highest performance. Therefore, after changing the business code, manual reload or restart is required to take effect.

At the same time, Workerman provides a file update monitoring service. This service automatically runs a reload when it detects a file update, reloading the PHP files. Developers can include it in the project and have it start with the project.

**File Monitoring Service Download Links:**

1. No dependency version: [https://github.com/walkor/workerman-filemonitor](https://github.com/walkor/workerman-filemonitor)
2. Dependency on inotify version: [https://github.com/walkor/workerman-filemonitor-inotify](https://github.com/walkor/workerman-filemonitor-inotify) (Requires installation of [inotify extension](https://php.net/manual/zh/book.inotify.php))

**Differences between the two versions:**

The version at address 1 uses the method of polling file update time every second to determine if the file has been updated.

The version at address 2 utilizes the Linux kernel's [inotify](https://baike.baidu.com/view/2645027.htm) mechanism, where the system will actively notify Workerman when a file is updated.

Generally, the first version without dependencies can be used.

**Usage:**

Copy the Applications/FileMonitor directory to the Applications directory of your project.

If your project does not have an Applications directory, you can copy the start.php file from Applications/FileMonitor to any location in your project, and then require it in the startup script.

The monitoring component defaults to monitoring the Applications directory. If you need to change this, you can modify the `$monitor_dir` variable in Applications/FileMonitor/start.php, and it is recommended to use an absolute path for the value of `$monitor_dir`.

**Note:**

* Windows systems do not support reload and cannot use this monitoring service.
* It only takes effect in debug mode and will not execute file monitoring under daemon mode (see explanation below for why daemon mode is not supported).
* Only the files loaded after Worker::runAll is called can be hot updated, or in other words, only the files loaded in the onXXX callback can be hot updated.

**Why does it not support daemon mode?**

Daemon mode is generally used for running in the production environment. When a production version is released, multiple files are usually released at once, and there may be dependencies between the files. Since it takes some time for multiple files to synchronize to the disk, there may be a moment when the files on the disk are incomplete. If the file is monitored and reload is executed at this time, there is a risk of fatal errors due to missing files.

In addition, in the production environment, there may be a need to debug online, and if the code is directly edited and saved, it will take effect immediately. This may lead to syntax errors causing the online service to be unavailable. The correct method would be to save the code, check for syntax errors using `php -l yourfile.php`, and then reload to update the code.

If a developer does need to enable file monitoring and automatic updating in daemon mode, they can modify the code in Applications/FileMonitor/start.php by removing the determination of Worker::$daemonize.
