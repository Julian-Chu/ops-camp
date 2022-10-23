# 梳理各 Namespace 的作用。
```shell
NAME
       namespaces - overview of Linux namespaces

DESCRIPTION
       A namespace wraps a global system resource in an abstraction that makes it appear to the processes within the namespace that
       they have their own isolated instance of the global resource.  Changes to the global resource are visible to other processes
       that are members of the namespace, but are invisible to other processes.  One use of namespaces is to implement containers.

       This  page provides pointers to information on the various namespace types, describes the associated /proc files, and summa‐
       rizes the APIs for working with namespaces.

   Namespace types
       The following table shows the namespace types available on Linux.  The second column of the table shows the flag value  that
       is used to specify the namespace type in various APIs.  The third column identifies the manual page that provides details on
       the namespace type.  The last column is a summary of the resources that are isolated by the namespace type.

       Namespace Flag            Page                  Isolates
       Cgroup    CLONE_NEWCGROUP cgroup_namespaces(7)  Cgroup root directory
       IPC       CLONE_NEWIPC    ipc_namespaces(7)     System V IPC,
                                                       POSIX message queues
       Network   CLONE_NEWNET    network_namespaces(7) Network devices,
                                                       stacks, ports, etc.
       Mount     CLONE_NEWNS     mount_namespaces(7)   Mount points
       PID       CLONE_NEWPID    pid_namespaces(7)     Process IDs
       Time      CLONE_NEWTIME   time_namespaces(7)    Boot and monotonic
                                                       clocks
       User      CLONE_NEWUSER   user_namespaces(7)    User and group IDs
       UTS       CLONE_NEWUTS    uts_namespaces(7)     Hostname and NIS
                                                       domain name
```
可以在 `/proc/[pid]/ns/` 找到 pid 對應的 namespaces

```shell
sudo ls -la /proc/1/ns/
total 0
dr-x--x--x 2 root root 0 Oct 22 12:54 .
dr-xr-xr-x 9 root root 0 Oct 22 12:54 ..
lrwxrwxrwx 1 root root 0 Oct 22 12:54 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Oct 22 12:54 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 net -> 'net:[4026531840]'
lrwxrwxrwx 1 root root 0 Oct 22 12:55 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 time -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 uts -> 'uts:[4026531838]'
```
# 使用 apt/yum/ 二进制安装指定版本的 Docker 。
https://docs.docker.com/engine/install/ubuntu/

設定不需要 sudo 執行
```shell
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

# 熟练使用 Docker 数据卷。
```shell
mkdir ~/data/testapp -p
echo "hello world!" > ~/data/testapp/index.html
# mount 宿主機到目錄到容器
docker run -it -d --name nginx-volume1 -p 83:80 -v ~/data/testapp:/usr/share/nginx/html/testapp nginx:1.23.1-alpine


mk@mk-server:~/data$ curl localhost:83/testapp/
hello world!
```
# 熟练使用 Docker 的 bridge 和 container 模式网络。
```shell
docker run -it -d --name nginx-web1-host-test-container  --net=host nginx:1.23.1-alpine
docker run -it -d --name nginx-web1-bridge-test-container  --net=bridge -p 81:80  nginx:1.23.1-alpine
docker run -it -d --name php-web-container  --net=container:nginx-web1-bridge-test-container php:7.4.30-fpm-alpine

mk@mk-server:~$ docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED          STATUS          PORTS                               NAMES
d5c204ed95b6   php:7.4.30-fpm-alpine   "docker-php-entrypoi…"   14 minutes ago   Up 14 minutes                                       php-web-container
ccb5aa32b510   nginx:1.23.1-alpine     "/docker-entrypoint.…"   30 minutes ago   Up 30 minutes   0.0.0.0:81->80/tcp, :::81->80/tcp   nginx-web1-bridge-test-container
65a2a3f7f7ad   nginx:1.23.1-alpine     "/docker-entrypoint.…"   38 minutes ago   Up 38 minutes                                       nginx-web1-host-test-container

# process 1
mk@mk-server:~$ sudo ls -la /proc/1/ns/
total 0
dr-x--x--x 2 root root 0 Oct 22 12:54 .
dr-xr-xr-x 9 root root 0 Oct 22 12:54 ..
lrwxrwxrwx 1 root root 0 Oct 22 12:54 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Oct 22 12:54 mnt -> 'mnt:[4026531841]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 net -> 'net:[4026531840]'    // network namespace
lrwxrwxrwx 1 root root 0 Oct 22 12:55 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 time -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Oct 22 13:24 uts -> 'uts:[4026531838]'

# host mode
mk@mk-server:~$ sudo ls -la /proc/5710/ns/
total 0
dr-x--x--x 2 root root 0 Oct 23 02:25 .
dr-xr-xr-x 9 root root 0 Oct 23 02:23 ..
lrwxrwxrwx 1 root root 0 Oct 23 02:25 cgroup -> 'cgroup:[4026532416]'
lrwxrwxrwx 1 root root 0 Oct 23 02:25 ipc -> 'ipc:[4026532414]'
lrwxrwxrwx 1 root root 0 Oct 23 02:25 mnt -> 'mnt:[4026532412]'
lrwxrwxrwx 1 root root 0 Oct 23 02:25 net -> 'net:[4026531840]'     // 與宿主機相同
lrwxrwxrwx 1 root root 0 Oct 23 02:25 pid -> 'pid:[4026532415]'
lrwxrwxrwx 1 root root 0 Oct 23 02:25 pid_for_children -> 'pid:[4026532415]'
lrwxrwxrwx 1 root root 0 Oct 23 02:25 time -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 Oct 23 02:25 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 Oct 23 02:25 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Oct 23 02:25 uts -> 'uts:[4026532413]'

# bridge mode
mk@mk-server:~$ sudo ls -la /proc/5922/ns/
total 0
dr-x--x--x 2 root root 0 Oct 23 02:30 .
dr-xr-xr-x 9 root root 0 Oct 23 02:30 ..
lrwxrwxrwx 1 root root 0 Oct 23 02:31 cgroup -> 'cgroup:[4026532482]'
lrwxrwxrwx 1 root root 0 Oct 23 02:31 ipc -> 'ipc:[4026532421]'
lrwxrwxrwx 1 root root 0 Oct 23 02:31 mnt -> 'mnt:[4026532419]'
lrwxrwxrwx 1 root root 0 Oct 23 02:30 net -> 'net:[4026532423]'   // 與宿主機不同
lrwxrwxrwx 1 root root 0 Oct 23 02:31 pid -> 'pid:[4026532422]'
lrwxrwxrwx 1 root root 0 Oct 23 02:31 pid_for_children -> 'pid:[4026532422]'
lrwxrwxrwx 1 root root 0 Oct 23 02:31 time -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 Oct 23 02:31 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 Oct 23 02:31 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Oct 23 02:31 uts -> 'uts:[4026532420]'


# container mode
total 0
dr-x--x--x 2 root root 0 Oct 23 02:47 .
dr-xr-xr-x 9 root root 0 Oct 23 02:47 ..
lrwxrwxrwx 1 root root 0 Oct 23 02:47 cgroup -> 'cgroup:[4026532487]'
lrwxrwxrwx 1 root root 0 Oct 23 02:47 ipc -> 'ipc:[4026532485]'
lrwxrwxrwx 1 root root 0 Oct 23 02:47 mnt -> 'mnt:[4026532483]'
lrwxrwxrwx 1 root root 0 Oct 23 02:47 net -> 'net:[4026532423]'  // 與指定的 container 有相同的 ns
lrwxrwxrwx 1 root root 0 Oct 23 02:47 pid -> 'pid:[4026532486]'
lrwxrwxrwx 1 root root 0 Oct 23 02:47 pid_for_children -> 'pid:[4026532486]'
lrwxrwxrwx 1 root root 0 Oct 23 02:47 time -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 Oct 23 02:47 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx 1 root root 0 Oct 23 02:47 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Oct 23 02:47 uts -> 'uts:[4026532484]'
```
