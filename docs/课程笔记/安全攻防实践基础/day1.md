# 第 1 天：Linux 操作系统 101

## Shell

Shell 是人和计算机之间进行交流的工具。

### Shell 命令

- ls
- pwd
- cat
- echo
- cp
- mv

## Linux 权限管理

subjects - actions - objects

### 文件权限

文件夹也是一种文件。

每个文件有它的拥有者 owner 和所属的组 group。

文件权限：读 r，写 w，执行 x。每个文件有三组这样的权限，一组给所有者，一组给用户组，一组给其他人。

## 今天的作业

### 装个 Linux 虚拟机

12 代酷睿对 VirtualBox 的支持并不好（或者说就是没法用），不过 VMWare 还是可以装个 Ubuntu 虚拟机的。

### Linux 文件访问控制

`chmod` 命令改变一些文件的权限。

`chown` 命令改变一些文件的所有者。

`chgrp` 命令改变一些文件的所属的群组。

文件拥有者是 bob：`chown bob hello.sh`

文件所属的组是 ctf：`chown :ctf hello.sh` （我是不是该用 `chgrp` 的来着？）

更改权限：`chmod 640 hello.sh`

#### setuid

`setuid`：执行这个文件的时候，身份会变成这个文件的所有者。

比如 passwd 当然属于 root，但是各个用户应该有权限更改自己的密码。而 `setuid` 权限就是为了让普通用户也能更改这属于 root 的文件。

### 通过 ls -l 查看文件的 permissions

`-rw-rw-r-- 1 iotang iotang 51  6月 27 18:57 hello.sh`

`-`：是一个文件。

`rw-rw-r--`：所有者拥有读写权限，所属用户组拥有读写权限，其他人拥有读权限。

`1`：连接的文件数。

`iotang iotang`：拥有人是 iotang，所属用户组是 iotang。

`51`：文件大小。

`6月 27 18:57`：修改时间。

`hello.sh`：文件名。
