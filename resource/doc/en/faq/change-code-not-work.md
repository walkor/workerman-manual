# Change not taking effect after modifying the code

**Reason:**

Workerman runs as a resident process in memory, which can avoid repeated disk reading and PHP interpretation and compilation in order to achieve the highest performance. Therefore, after modifying the business code, it needs to be manually reloaded or restarted to take effect.

At the same time, Workerman provides a file monitoring service, which automatically runs a reload when it detects file updates, re-loading PHP files. Developers can include this service in the project and start it with the project.

Note: The reload feature is not supported on Windows and the monitoring service cannot be used.

**File monitoring service download links:**

1. No dependency version: https://github.com/walkor/workerman-filemonitor

2. Dependency on inotify version: https://github.com/walkor/workerman-filemonitor-inotify

**Difference between the two versions:**

The first version uses a method of polling file modification time every second to determine whether the file has been updated, while the second version utilizes the Linux kernel's [inotify](https://baike.baidu.com/view/2645027.htm) mechanism, through which the system actively notifies Workerman when a file is updated.

In general, using the first version without dependencies is sufficient.
