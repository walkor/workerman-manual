# Linux Kernel Tuning

In order to support greater concurrency in the system, in addition to the necessary [installation of the event extension](../install/install.md), optimizing the Linux kernel is also of **utmost importance**. Each optimization below is **extremely important**, so please be sure to complete each one sequentially.

**Parameter Explanation:**

> **max-file**: Represents the number of file handles that can be opened at the **system level**. This is for the entire OS, not for individual users.
> 
> **ulimit -n**: Controls the number of file handles that can be opened at the **process level**. This applies to the available file handle control for the current user of the shell and the processes it launches.

To view the number of file handles that can be opened at the **system level**: `cat /proc/sys/fs/file-max`

Open the file /etc/sysctl.conf and add the following settings:
```conf
# This parameter sets the number of TIME_WAIT states in the system. If it exceeds the default value, it will be cleared immediately.
net.ipv4.tcp_max_tw_buckets = 20000
# Defines the maximum listen queue length for each port in the system. This is a global parameter.
net.core.somaxconn = 65535
# The maximum number of outstanding connection requests in the queue for connections that have not yet received an acknowledgment from the other end.
net.ipv4.tcp_max_syn_backlog = 262144
# When the rate at which the kernel receives packets on each network interface is faster than the rate at which it processes these packets, the maximum number of packets allowed in the queue to be sent.
net.core.netdev_max_backlog = 30000
# This option causes clients in NAT networks to time out, and it is recommended to be set to 0. The tcp_tw_recycle configuration was removed from Linux kernel 4.12, so if you encounter the error "No such file or directory," please ignore it.
net.ipv4.tcp_tw_recycle = 0
# The total number of files that all processes in the system can open.
fs.file-max = 6815744
# Size of the firewall connection tracking table. Note: If the firewall is not enabled, it will prompt an error "net.netfilter.nf_conntrack_max" is an unknown key. You can ignore it.
net.netfilter.nf_conntrack_max = 2621440
net.ipv4.ip_local_port_range = 10240 65000
```
Run `sysctl -p` to take effect immediately.

**Note:**

/etc/sysctl.conf offers many configurable options, and other options can be set according to the specific environment needs.

## Opening File Limits

Set the system's open file limit to resolve the "too many open files" issue under high concurrency, which directly affects the maximum number of client connections a single process can handle.

Soft open files is a Linux system parameter that affects the maximum number of file handles a single process can open, which in turn impacts the number of user connections a process can maintain in long-lived connection applications, such as chat applications. Running `ulimit -n` displays this parameter value. If it is 1024, it means a single process can only maintain a maximum of 1024 or even fewer connections (due to other file handles being opened). If 4 processes are running to handle user connections, the entire application can only maintain a maximum of 4*1024 connections, meaning it can support a maximum of 4x1024 users online. Increasing this setting allows the service to maintain more TCP connections.

**Three methods to modify soft open files:**

First method: Run `ulimit -HSn 102400` directly in the terminal and then restart Workerman. This change is only effective in the current terminal, and the open files will revert to the default value after exiting.

Second method: Add `ulimit -HSn 102400` at the end of the file `/etc/profile`. This way, it will be executed automatically every time the terminal is logged in. After modification, Workerman needs to be restarted.

Third method: To permanently apply the change in the value of open files, it is necessary to modify the configuration file: `/etc/security/limits.conf`. Add the following lines to the end of this file:

```
* soft nofile 1024000
* hard nofile 1024000
root soft nofile 1024000
root hard nofile 1024000
```

This method requires a server restart to take effect.
