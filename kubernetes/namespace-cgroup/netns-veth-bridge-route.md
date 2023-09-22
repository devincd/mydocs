## Linux network namespace，veth，bridge与路由

### Network Namespace
Namespace 是 Linux 提供的一种对于系统全局资源隔离机制，从进程的视角来看，同一个 namespace 中的进程看到的是该 namespace 自己独立的一份去全局资源，
这些资源的变化只在本 namespace 中可见，对其他 namespace 没有影响。容器就是采用 namespace 机制实现了对网络，进程空间等的隔离，不同的容器属于不同 namespace，
实现了容器之间的资源相互隔离，互不影响。

Linux 提供了以下七种 namespace：

| Type | Function |
| --- | ----------- |
| Cgroup | Cgroup root directory |
| IPC | Cgroup root directory |
| Network     | Network devices, stacks, ports, etc. |
| Mount       | Mount points                    |
| PID         | Process IDs                     |
| User        | User and group IDs              |
| UTS         | Hostname and NIS domain name    |

Network namespace 允许你在 Linux 中创建相互隔离的网络视图，每个网络名字空间都有自己独立的"网络栈"。

而所谓的"网络栈"，就包括了：网卡（Network Interface），回环设备（Loopback Device），路由表（Routing Table）和 Iptables 规则。