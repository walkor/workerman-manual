# Code changes not taking effect

**Reason:**

Workerman runs in a persistent memory environment, which avoids repeated disk reads and PHP interpretation and compilation in order to achieve maximum performance. Therefore, after modifying business code, a manual reload or restart is required for the changes to take effect.

Meanwhile, Workerman provides a file monitoring service that automatically runs a reload when it detects file updates, reloading PHP files. Developers can integrate it into the project for it to start along with the project.

Note: Windows systems do not support reload and cannot use the monitoring service.

**File monitoring service download links:**

1. No dependency version: https://github.com/walkor/workerman-filemonitor

2. Dependency on inotify version: https://github.com/walkor/workerman-filemonitor-inotify

**Differences between the two versions:**

The version at address 1 uses a method of polling file update times every second to determine if a file has been updated.

The version at address 2 leverages the Linux kernel's [inotify](https://baike.baidu.com/view/2645027.htm) mechanism, through which the system actively notifies Workerman when a file is updated.

In general, the no-dependency version at address 1 can be used.
