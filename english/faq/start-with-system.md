## How to Automatically Start Workerman on Linux System

To automatically start Workerman on boot-up, you can open the **/etc/rc.local** file and add the following code before the **exit 0** line:

```shell
ulimit -HSn 102400
/usr/bin/env php /path/to/your/directory/start.php start -d
```

Make sure to replace **/path/to/your/directory/** with the actual path to your Workerman installation. After adding this code, save the file and Workerman will start automatically on system boot-up.
