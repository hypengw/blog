+++
title = 'User Namespace 套娃'
date = 2025-10-24T23:11:24+08:00
draft = false
tags = ['linux']

+++

博主已经用了 podman 很长一段时间，老实说有个概念一直一知半解，就是 User Namespace，以及其相关的配置选择  
就是处于试一试，能用就这样的心态，也就一直安然无恙的用下来了  
但是稍微复杂点的需求，这样半吊子就跑不起来了，遂稍微看看是个什么玩意

## 一些简单背景

众说周知，user 这个概念是 system 给的，对应到接口上，就是 `uid`/`gid` 以及内部与其绑定的子系统，通过这些来进行权限判断和访问控制  
同时，构建在 kernel 之上的 system 还需要将 user 和 filesystem 绑定在一起，用于描述 user 和 file 之间的关系，以及某些 file 在 kernel 那边的特殊处理  
传统上 `[0,65535]` 为系统保留的 `uid` 范围，而这其中 `0` 为 `root`，大于 `65535` 的可以被拿去做 `uid_map`  
顺带一提，Linux 发行版新建普通 user 时，`uid` 常见都是从 `1000` 开始分配的 

### [user_namespaces](https://man7.org/linux/man-pages/man7/user_namespaces.7.html)
- A process's user and group IDs can be different inside and outside a user namespace  
- User namespaces can be nested  
- When a user namespace is created, it starts out without a mapping of user IDs (group IDs) to the parent user namespace.  The `/proc/pid/uid_map` and `/proc/pid/gid_map` files (available since Linux 3.5) expose the mappings for user and group IDs inside the user namespace for the process pid  
- After the creation of a new user namespace, the uid_map file of one of the processes in the namespace may be written to once to define the mapping of user IDs in the new user namespace  
- Since Linux 3.8, no privilege is required to create a user namespace  
- kernel api: `clone`, `setns`, `unshare`, `ioctl`  

### [Rootless Containers](https://rootlesscontaine.rs/)
Rootless containers refers to the ability for an unprivileged user to create, run and otherwise manage containers. This term also includes the variety of tooling around containers that can also be run as an unprivileged user.

“Unprivileged user” in this context refers to a user who does not have any administrative rights, and is “not in the good graces of the administrator” (in other words, they do not have the ability to ask for more privileges to be granted to them, or for software packages to be installed).

Pros:  
    - Can mitigate potential container-breakout vulnerabilities (Not a panacea, of course)  
    - Friendly to shared machines, especially in HPC environments
Cons:  
    - Complexity


## podman rootless
> Podman can also be used as non-root user. When podman runs in rootless mode, a user namespace is automatically created for the user, defined in `/etc/subuid` and `/etc/subgid`.

可以见得，podman 的 rootless 就是 continaer with user namespace  

我们简单看看默认情况下的 userns 的情况  
```bash
$ cat /etc/subuid       
my:100000:65536
$ readlink /proc/self/ns/user  
user:[4026531837]
$ cat /proc/self/uid_map 
         0          0 4294967295
$ podman run --rm -it docker.io/library/ubuntu:latest bash
root@e8d3045e9019:/# readlink /proc/self/ns/user
user:[4026533265]
root@e8d3045e9019:/# cat /proc/self/uid_map 
         0       1000          1
         1     100000      65536
```

这里我们先看了当前 shell 所在的 userns，即 host 里的初始 userns
这时 `uid_map` 显示的是没有任何 map 的状态  
而进入 rootless container 后，userns 发生了变化，同时 `uid_map` 如下  
| namespace uid | outside uid | count |
|--|--|--|
|0|1000|1|
|1|100000|65536|

即将 当前用户的 uid `1000` 映射到容器里的 `0(root)`，而之后的容器的 uid 从外部的 `100000` 开始分配

```bash
$ lsns -t user -o NS,PNS,TYPE
        NS        PNS TYPE
4026531837          0 user
4026533265 4026531837 user
```

通过 `lsns` 我们可以看到，容器的 userns 的 parent ns 是 host 的初始 userns

## podman rootless userns 套娃
这里就要说 podman userns 相关的配置选项了，主要是 `--userns` 和 `--uidmap`  
其实两者本质是相同的，对底层的 continaer runtime 来说  
都是在上面说的 container 第一层的 userns 下再套了一层 userns  
这时 podman 把第一层的 userns 的 uid 称为 `intermediate UID`  
文档里的路径为 `host UID -> intermediate UID -> container UID`，但其实没有什么特殊的，就是上层的 userns 而已  

### `--userns=mode`

| Key                     | Host User | Container User                                               |
| ----------------------- | --------- | ------------------------------------------------------------ |
| auto                    | $UID      | nil (Host User UID is not mapped into container.)            |
| host                    | $UID      | 0 (Default User account mapped to root user in container.)   |
| keep-id                 | $UID      | $UID (Map user account to same UID within container.)        |
| keep-id:uid=200,gid=210 | $UID      | 200:210 (Map user account to specified UID, GID value within container.) |
| nomap                   | $UID      | nil (Host User UID is not mapped into container.)            |

> note  
> 这里的 nomap 还是会创建二层 userns，反而是 host 和默认的行为一致，只有一层 userns

### `--uidmap=[flags]container_uid:from_uid[:amount]`
上面的 `--userns` 都可以改为一些 `--uidmap` 的组合  

|Key| Uidmap |
|--|--|
|keep-id| `--uidmap=0:1:1000 --uidmap=1000:0:1 --uidmap=1001:1001:64536`|
|nomap| `--uidmap=0:1:65536`|


