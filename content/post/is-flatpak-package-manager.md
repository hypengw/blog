+++
title = 'Flatpak 是包管理吗'
date = 2025-01-03T21:25:18+08:00
draft = false
tags = ['科技','Linux']

+++

> 阅读本篇文章需要有所了解：`Linux 发行版`, `Linux 桌面环境(Desktop Envrionment)`, `包管理(Package Manager)`

博主第一次接触 Linux 是在高中，当时似乎是有个软件只有 Linux 版本，然后就参照着网上的文章试着安装 arch，具体的细节已经不太记得了。
对当时的我来说，一堆不明所以的命令挺头疼的，一直害怕把电脑搞坏。 

大学有段时间是双系统，日常使用中，Linux 系统特别容易“坏掉”，坏掉的原因千奇百怪。分层来看， 硬件兼容/内核/驱动/系统库/依赖/桌面环境等等都可能有bug， 这些组件还有点貌合神离，出问题是常有的事情，偶尔解决一两个问题是会有成就感，但是多了就单纯感到麻烦。

有个具体的例子，我看到的时候想笑又感同深受，[Linus Tech 为了安装 steam 而卸载掉了桌面环境](https://www.youtube.com/watch?v=0506yDSgU7M)。

在 2020 年， 偶然接触了 flatpak，不一会就把日常软件都 all in flatpak 了，而后也陆续维护了几个自己想使用的软件到 flathub。日常体验来说，我再也不用去担心哪天更新后软件没法使用了，不用自己手动管理软件的更新，不再担心不安分的闭源软件，不再担心系统更新会破坏我的主力软件。

**在用上 flatpak 以后，我决定把 Linux Desktop 作为自己的主力系统**。

## Flatpak 解决了什么问题

首先，最重要的，**和系统解耦**。具体解决的就是上文 Linus Tech 的遇到的问题。其实类似东西挺早就在服务器上有了，就是容器，也就是 docker 那一套。接触过 Linux 发行版的朋友对[**包管理**](https://en.wikipedia.org/wiki/Package_manager)应该特别熟悉，同时还有**依赖**，**仓库**，**源码**，**软件包**，[**FHS**](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)等等概念，不过我们这里不讨论这套系统具体怎么工作。

我们来看看 Linus Tech 是如何不小心损坏了桌面环境。安装命令 `sudo apt-get install steam`， 而后是一大片输出，其中有一段警告 `WARNING: The following essential packages will be removed`， 而这里面包含了 `pop-desktop`，最后桌面环境就被这么卸载掉了。部分人会指出这是 Linus Tech 不熟悉的原因，或者是安装引导和关键包规避没有处理好，是包管理的问题，又或者是用 Linux 做桌面系统就是傻逼活该。

抛开现象看关键，一是安装用户软件需要`sudo`。有个挺好的类比，就是 Win 系统下的 [UAC](https://en.wikipedia.org/wiki/User_Account_Control)，经常使用 Win 系统的朋友一定听过*重装系统*这个词吧，那为什么你需要时不时重装系统呢，一个比较大的因素就是日常使用的大部分软件它们都请求 UAC 来安装或者运行。用车来比喻，一台原本挺好的车，为了某些功能，被送去了改装店，改装店的人当然能实现这个功能，但是这也不妨碍他们在车上装些额外的东西或者替换关键的零件，改装的次数多了，车慢慢就废掉了。

二是 `apt-get` 同时管理了用户软件和系统组件。以我的观念，**依赖管理是一个复杂的系统问题**，软件包之间杂乱的依赖关系，关系网中的每个节点时常会更新自身，不难得出，这里面的不确定性是随着包的增多而接近指数递增的。这也有个例子，偶尔看使用 `npm` 的开发者吐槽 `node_modules`，特别有意思。

这里我明确区分了**用户软件**和**系统组件**。系统组件，数量少，变化慢且稳定，多是被依赖，这些特点完美契合包管理。而用户软件，数量庞且杂，变化快同时随时可能死掉，依赖大量的库，这完全踩在包管理的雷区。当我们尝试把这两种类型的包融合在一起的时候，系统组件被某个用户软件“炸掉”就是意料之中的事情。

其次，就是引入了**沙箱**。这点解决的问题不用多说，近些年 Microsoft 和 Google 都在收紧各自系统用户软件的权限。国内的用户也在慢慢注意隐私和安全相关的概念。

然后就是**统一**。众所周知，Linux 发行版是相当**碎片化**的，比 Android 还要碎的多。博主就从 `arch` 换到 `debian` 换到 `opensuse` 换回 `debian` 最后固定在 `fedora`，还有数不清的小众中的小众发行版，同时这些发行版还有类似 Android 的版本碎片化。然后同时还要算上桌面环境的碎片化，`gnome/kde/cosmic/xfce/lxqt/...`。这些碎片化带来了相当糟糕(操蛋)的用户体验，也劝退了许多开发者。

最后站在开发者的角度，一直有两个矛盾的点。

一是和用户间系统环境不相同，这是碎片化的问题，不过导致 Linux 用户名声变臭了，比如常见的说法，**用户数量最少，但是提出的问题是最多的**。

二是和发行版间的，核心的矛盾在于发行版和开发者间对于依赖的版本有不同的要求。发行版需要某个组件保持最新版本或者不更新大版本，只更新安全更新。开发者就一点，用和开发环境一样的依赖版本。

## Flatpak 是怎么解决的

### [运行时](https://freedesktop-sdk.io/)

![img](https://s33.postimg.cc/fpqffim4f/diagram.png)

- 运行时囊括了大部分公用基础库，比如 glibc，libstdc++，libcurl，xorg套件，wayland套件等等。
- 版本号 23.08/24.08，每年一次大更新，来引入 breaking change，而平常的维护会确保 [abi 兼容](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/blob/release/24.08/utils/check-abi.py?ref_type=heads)。
- 不在运行时里的库，由软件自行 bundle，同名库优先加载 bundle 版本。类似 Win/Android 的做法 。
- 运行时分为 **Platform** 和 **SDK**，简单来说，Platform 由 SDK 删减而来，主要删减头文件，静态库，编译工具链等等。

基本情况说明了，部分人会疑惑，为什么不直接使用 host 的库呢，这样更节省空间。

**和系统解耦**，不依赖也不假设 host 系统的库，这样就算是老掉牙的 debian oldstable，也能用上最新的库和软件。这也极大的解决了发行版**碎片化**的问题。

对开发者来说，确定的运行环境，可以决定依赖库版本的自由，可以更多把精力放到开发软件上。

### [bubblewrap 隔离](https://github.com/containers/bubblewrap)

借由 [Linux namespaces](https://en.wikipedia.org/wiki/Linux_namespaces) 来实现的非特权沙盒，具体可以阅读项目主页。

它保证了，在指定有限的权限下，让 flatpak 应用和 host 环境隔离开。

### [Portal(xdp)](https://github.com/flatpak/xdg-desktop-portal)

隔离了 host 环境以后，我们需要一套经过验证的 IPC 接口，用来管理一些敏感的 host 资源。

具体可以参考 Android permisson，不过 android 是基于 binder，而 xdp 基于 dbus。

比较有意思的是，xdp 变相统一了 Linux desktop 混乱的桌面环境接口，`qt/gtk/electron/...` 现在都已经适配了 xdp。

## Flatpak 是包管理吗

回到本文标题，Flatpak 是包管理工具吗？我认为不是。

我们对包管理的定义一般是限定在 Linux 发行版里的满足某些功能的软件，flatpak 和这些软件有许多相似之处，但是做法却完全不同。同时，最重要的，flatpak 没有办法替代传统的包管理，它更多是一个补充，让包管理回到“管理系统软件包”的职能上。flatpak 包的颗粒度非常粗，依赖关系也异常浅，无法去构建/操作发行版里的细小组件，比如 freedesktop sdk 自己就不是通过 flatpak 而是 [BuildStream](https://github.com/apache/buildstream) 来构建。

不过这其实是个开放问题，读者可以自己斟酌。

不如看看其他地方，比如 Android 的软件包安装程序是包管理吗?

## Flatpak Tips和技术细节

### [运行时](https://freedesktop-sdk.io/)

#### `/app`

我们进入运行时里去具体看看

```bash
$ flatpak run --command=bash org.freedesktop.Sdk//24.08
[📦 org.freedesktop.Sdk ~]$ ls /
app  bin  dev  etc  lib  lib64  proc  run  sbin  sys  tmp  usr  var
[📦 org.freedesktop.Sdk ~]$ cat /etc/ld.so.conf 
include /run/flatpak/ld.so.conf.d/app-*.conf
include /app/etc/ld.so.conf
/app/lib
include /run/flatpak/ld.so.conf.d/runtime-*.conf
[📦 org.freedesktop.Sdk ~]$ ldd /usr/bin/ls
        linux-vdso.so.1 (0x00007f4737637000)
        libselinux.so.1 => /usr/lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f47375bd000)
        libc.so.6 => /usr/lib/x86_64-linux-gnu/libc.so.6 (0x00007f47373be000)
        libpcre2-8.so.0 => /usr/lib/x86_64-linux-gnu/libpcre2-8.so.0 (0x00007f473731d000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f4737639000)
[📦 org.freedesktop.Sdk ~]$ echo $PATH
/app/bin:/usr/bin
```

可以发现这其实就等效于一个遵循 **FHS** 的最小化发行版。

- `/app`  
  作为每个软件的 Prefix，编译打包中，只有这个目录是可写的。
- `/app/lib`  
  软件放置 Bundle library 的地方
- `/app/bin`  
  默认在容器的 PATH 环境变量里，需要把软件的启动程序安装/链接到这里

#### host share

```bash
[📦 org.freedesktop.Sdk ~]$ echo $XDG_DATA_DIRS
/app/share:/usr/share:/usr/share/runtime/share:/run/host/user-share:/run/host/share
[📦 org.freedesktop.Sdk ~]$ cat /etc/fonts/conf.d/50-flatpak.conf 
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
        <!-- This has to be first so it is written while building the runtime -->
        <cachedir>/usr/cache/fontconfig</cachedir>

        <dir>/app/share/fonts</dir>
        <!-- Then this so it is written when building the app -->
        <cachedir>/app/cache/fontconfig</cachedir>

        <include ignore_missing="yes">/app/etc/fonts/local.conf</include>

        <dir>/run/host/fonts</dir>
        <dir>/run/host/local-fonts</dir>
        <dir>/run/host/user-fonts</dir>

        <!-- This is duplicated from the general config because we want to write there
             before the /run dirs, in case they are ever writable, like e.g with old
             versions of flatpak. -->
        <cachedir prefix="xdg">fontconfig</cachedir>

        <cachedir>/run/host/fonts-cache</cachedir>
        <cachedir>/run/host/user-fonts-cache</cachedir>

        <include>/run/host/font-dirs.xml</include>
</fontconfig>
```

- `/run/host/share/icons `   
  对应 `/usr/share/icons`

- `/run/host/user-share/icons`  
  对应 `${XDG_DATA_HOME}/icons`

- `/run/host/fonts`  
  对应 `/usr/share/fonts`

- `/run/host/user-fonts`  
  对应 `${XDG_DATA_HOME}/fonts`

### 全局权限

flatpak 可以给所有应用预制统一的权限。  
这一点还挺好用的，比如我有一个文件夹，不能让任何应用看到，哪怕有 `host` 权限。

```bash
$ flatpak override --user --filesystem='xdg-config/fontconfig:ro'
$ cat ~/.local/share/flatpak/overrides/global
[Context]
filesystems=xdg-config/gtk-3.0:ro;!xdg-run/keyring;/nix:ro;xdg-config/fontconfig:ro;

[Session Bus Policy]
com.canonical.AppMenu.Registrar=talk

[Environment]
MOZ_ENABLE_WAYLAND=1
```

#### 优先级

应用自己的权限配置是第一优先，而后是全局的。  
不过 `filesystem` 权限不太一样，它自身就有优先级，比如 `host/home` 比一般的路径优先级要低。大致是先满足了 `filesystem` 自己的优先级再来论其他的。

### system-wide and per-user

类似 Win 的把应用 安装到所有用户 和 只为我安装。  
注意这两边是完全独立开来的，都用的话，需要两份 `runtime`  
我是推荐用 `per-user`，如果是家里共享的电脑，推荐 `system-wide`

#### system-wide

安装位置：`/var/lib/flatpak/`  
需要权限来执行命令，发行版基本都有安装 [polkit rule](https://github.com/flatpak/flatpak/blob/1.15.91/system-helper/org.freedesktop.Flatpak.rules.in)，当然你也可以用 `sudo`

#### per-user

安装位置：`$HOME/.local/share/flatpak/`  
需要使用 `--user` flag ，加在命令后面

