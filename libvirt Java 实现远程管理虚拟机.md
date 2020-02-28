# libvirt Java 实现远程管理虚拟机

**from csdn.kyyee**



## 虚拟化简介

虚拟化是将计算机的各种实体资源（CPU、内存、存储、网络等）进行抽象后呈现出来，即是将一台物理计算机分割成多台计算机，实现在一台计算机上运行多台虚拟机，每台虚拟机可运行不同的操作系统，可包含不同的虚拟硬件，并且它们在相互独立的空间内运行而互不影响。

![图1](http://or2qjfdhl.bkt.clouddn.com/20170612134012.png)

在虚拟化软件层次结构中引入了虚拟化层，这个通常称为虚拟机监控器（Virtual Machine Manager，VMM）。虚拟机监控器运行的环境就是物理机，也称为宿主机。虚拟出来的平台通常称为客户机。

虚拟机使用软件的方法重新定义划分计算机资源，有如下优势：

-   隔离：虽然虚拟机可共享一台物理机，但它们之间完全隔离，它们就是完全不同的物理计算机。
-   可靠：虚拟服务器独立于硬件进行工作，灾难恢复更容易实现，当一台虚拟服务器故障，可短时间内迁移恢复到另一台虚拟服务器上。
-   成本：当物理机资源较少时，也可通过虚拟化实现使用更多服务器才能做的事情。
-   便于管理：通过虚拟机集群管理系统，一个管理员可管理更多服务器。



### KVM/QEMU 简介

2008 年，[KVM](https://www.linux-kvm.org/page/Main_Page) 被 RedHat 收购，成为红帽开源项目的一员。KVM 全称 Kernel-based Virtual Machine，即基于内核的虚拟机，是一个 x86 平台上的全虚拟化解决方案。它是 Linux 内核的一个可加载模块，包括核心虚拟化模块 kvm.ko，以及特定 CPU 的模块 kvm-inet.ko 或 kvm-amd.ko，其实现需要宿主机的 CPU 支持**硬件虚拟化**。
KVM 可运行多个虚拟机，无论是未经修改的 Linux 镜像还是 Windows 镜像。每个虚拟机都有私有的虚拟化硬件如：网卡、磁盘、显卡驱动等。
从 Linux 内核版本 2.6.20 开始，KVM 核心组件被包含在 Linux 内核中。从 QEMU 版本 1.3 开始，KVM 用户空间组件被包含其中。
在 x86 平台下 CPU 的硬件虚拟化技术有 Intel 的 VT-X 和 AMD 的 AMD-V。
KVM 的特征请查看官网，[传送门](https://www.linux-kvm.org/page/KVM_Features)

**注意**：BIOS 需要开启虚拟化支持，VMware 的虚拟机需要开启虚拟化支持。

Linux 内核加载了 KVM 模块后，就可以使用 KVM 模块实现虚拟机的内存分配、虚拟 CPU 的读写及管理虚拟 CPU 的运行。但是仅有 KVM 模块远远不够，因为用户无法直接控制内核模块，所以还需要一个用户空间的工具。这个用户空间的工具就是开源虚拟化软件 QEMU，使用它来模拟 PC 硬件的用户空间组件、I/O 设备及提供访问外设的途径。

![图2](http://or2qjfdhl.bkt.clouddn.com/20170612143031.png)

QEMU 利用 KVM 提供的应用程序接口，通过 ioctl 系统调用创建和运行虚拟机。KVM Driver 使得整个 Linux 成为一个虚拟机监控器。并且在原有的 Linux 两种执行模式（内核模式和用户模式）的基础上新增了客户模式，客户模式拥有自己的内核模式和用户模式。

在虚拟机运行下，三种模式的分工如下：

-   客户模式：执行非 I/O 操作。虚拟机运行在客户模式下。
-   内核模式：实现到客户模式的切换，处理因为 I/O 或者其他指令所引起的从客户模式退出。KVM Driver 工作在内核模式下。
-   用户模式：代表客户机执行 I/O 指令。QEMU 运行在用户模式下。

### QEMU 与 KVM 的区别与联系

上一节已经明确 KVM 仅仅是一个内核模块，它可以模拟虚拟机的 CPU 和内存，但是我们还需要 I/O 设备，这些 I/O 设备就是通过 QEMU 这个用户空间模拟器来模拟的。[QEMU](http://www.qemu.org/) 本身是一套通用开源的机器仿真器和虚拟程序，它有两种使用方式。

#### 全系统仿真（单独使用）

这种方式对宿主机硬件没有要求，也不需要宿主机 CPU 支持虚拟化，QEMU 为虚拟机操作系统模拟整套硬件环境，虚拟机操作系统感觉不到自己运行在模拟的硬件环境中。这种纯软件模拟效率很低，它可模拟出各种硬件设备。

#### 用户空间仿真（配合 KVM 等）

这种方式与内核模块 KVM 配合完成硬件环境的模拟。在 QEMU 1.3 之前，它有一个专门的分支版本 qemu-kvm 作为 KVM 的用户空间程序，qemu-kvm 通过 ioctl 调用 /dev/kvm 这个接口与 KVM 交互，这样 KVM 在内核空间模拟虚拟机 CPU，qemu-kvm 负责模拟虚拟机 I/O 设备。在 QEMU 1.3 以后的版本中，qemu-kvm 分支代码已经合并到 QEMU 的 master 分支中。在编译 QEMU 时开启 –enable-kvm 选项，就能够使 QEMU 支持 KVM。



### libvirt 简介

通过 QEMU 命令来管理虚拟机非常麻烦，RedHat 发布了一个开源项目 libvirt，它是为了更加方便的管理各种虚拟化引擎而设计的。libvirt 作为中间适配层，让底层虚拟化引擎对上层用户空间的管理工具做到完全透明，因为 libvirt 屏蔽了底层各种虚拟化的细节，为上层管理工具提供了一个统一的接口。

libvirt 支持多种虚拟化引擎，即支持包括 KVM、QEMU、Xen、VMware、VirtualBox 等在内的平台虚拟化方案，又支持 OpenVZ、LXC 等 Linux 容器虚拟化系统，还支持用户态 Linxu（UML）的虚拟化。

libvirt 是目前使用最广泛的对 KVM 虚拟机进行管理的工具和应用程序接口，而且一些常用的虚拟机管理工具和云计算框架平台（如 OpenStack、CloudStack 等）都在底层使用了 libvirt 的应用程序接口。

![图3](http://or2qjfdhl.bkt.clouddn.com/20170612170155.png)

libvirt 通过相同的方式管理不同的虚拟化引擎，支持虚拟机的创建、启动、关闭、暂停、睡眠、迁移、删除，以及对虚拟机 CPU、内存、磁盘、网卡等多种设备的热添加。libvirt 还支持通过远程连接的方式管理虚拟机。

#### libvirt API

[libvirt](http://libvirt.org/) 为很多操作系统（如 QEMU，KVM，Xen，LXC 等）提供一套轻便、高效、长期稳定的 API ，libvirt API 最初是用 C 语言实现的，在原生 C API 的基础上提供了 Python，Perl，[Java](https://github.com/libvirt/libvirt-java)，Ruby，PHP，C# 等众多语言的 [API](https://github.com/libvirt)。本系列文章先介绍 [C API](http://libvirt.org/docs/libvirt-appdev-guide/en-US/html/)，然后主要讲解 Java API。官方 [Python API](http://libvirt.org/docs/libvirt-appdev-guide-python/en-US/html/) 的文档最全，本文的 [Java API](http://libvirt.org/sources/java/javadoc/)（2017-06-24 13:00 浏览 libvirt 官网找到 Java API 文档） 讲解主要参考自 Python API。

#### libvirt 术语

在介绍 API 之前，先了解一下 libvirt 中几个重要的[术语](http://libvirt.org/goals.html)，如表所示。

| 名词           | 解释                                                         |
| -------------- | ------------------------------------------------------------ |
| Hypervisor     | 一个虚拟机的软件层，也称为虚拟机监视器（VMM）                |
| Domain         | 域，在 Hypervisor 上运行的一台客户机操作系统实例。客户机操作系统（Guest OS）和虚拟机（Virtual Machine）同义 |
| Node           | 节点，一台物理机，也就是宿主机，上面运行着多台虚拟机。Hypervisor 和 Domain 都运行在 Node 之上 |
| Storage Pool   | 存储池，比如物理硬盘。存储池可被划分为小的容器，称作存储卷。通常是宿主机（物理机）的硬盘，默认名称为 default |
| Storage Volume | 存储卷，从存储池分配的存储空间。一个存储卷被分配给一个域，通常被作为域的虚拟硬盘，实际使用中主要是 qcow2 镜像文件 |

Node、Hypervisor、Domain、Storage Pool 和 Storage Volume 之间的关系如图所示。



#### libvirt C API

libvirt C API 可分为如下几类。

-   连接 Hypervisor 的 API，以 virtConnect 开头的一系列函数；
-   Domain（客户机）管理 API，以 virtDomain 开头的一系列函数；
-   Node（宿主机）管理 API，以 virtNode 开头的一系列函数；
-   网络管理 API，以 virtNetwork 和 virtInterface 开头的一系列函数；
-   存储卷管理 API，以 virtStorageVol 开头的一系列函数；
-   存储池管理 API，以 virtStorage Pool 开头的一系列函数；
-   事件管理 API，以 virtEvent 开头的一系列函数。



#### libvirt Java API

Java 本身不能与用 C 开发的函数库直接交互，但我们仍然要使用 libvirt C 库。这时，JNI（Java Native Interface，Java 本地接口）就派上用场了，它可以与使用 C、C++ 开发的函数库进行交互。然而 JNI 很麻烦，可移植性糟糕。于是出现了 JNA（Java Native Access），它本质上是 JNI 的一个抽象层，JNA 在功能上和 JNI 相差无几。相比于 JNI，它开发简便，不需要编写额外的本地代码，直接在接口中调用 C 的函数。这样 libvirt Java API 调用 libvirt C API 的过程就变成了：Java 微服务=>libvirt Java API=>JNA=>JNI=>libvirt C API。

libvirt Java API 可分为如下几类。

-   连接 Hypervisor 的 API，与 Connect 对象相关的属性与方法。
-   Domain（客户机）管理 API，与 Domain，DomainInfo 等 Domain 开头的对象相关的属性与方法。
-   Node（宿主机）管理 API，与 Connect，NodeInfo 对象相关的属性与方法。
-   网络管理 API，与 Network，Interface 开头的对象相关的属性与方法。
-   存储卷管理 API，与 StorageVol，StorageVolInfo 对象相关的属性与方法。
-   存储池管理 API，与 StoragePool，StoragePoolInfo 对象相关的属性与方法。
-   ~~事件管理 API，暂未发现有 Java API 事件管理相关 API。~~



## 安装所需工具

KVM 虚拟化环境需要安装 QEMU 和 libvirt，可以通过 APT 源或源码编译的方式安装，通过 APT 源方式安装简单，如果想使用更高版本的软件则需要通过源码编译安装。

**kvm 已集成到 Linux 内核，无需安装。**



### Linux 系统环境

虚拟机：VMware 15
系统版本：Ubuntu 18.04 64 位

### APT 源安装

#### qemu、libvirt、bridge-utils、virt-manager安装

1.BIOS 需要开启虚拟化支持，VMWare 需要在虚拟机 CPU 配置页面开启虚拟化支持。

2.查看 cpu 是否支持安装，输出 vmx 表示支持。

```shell
egrep "(svm|vmx)" /proc/cpuinfo
```

3.安装 QMEU 和 libvirt。

```shell
sudo apt-get install -y qemu libvirt-bin bridge-utils virt-manager
```

bridge-utils 是网桥管理工具。
[virt-manager](https://virt-manager.org/) 是一个通用的桌面管理工具。它即可以本地访问 Hypervisors ，也可以远程访问。针对家庭和小型办公室用来管理10-20台主机和虚拟机。

4.配置安全策略关闭 apparmor（可不关闭）。

```shell
sudo /etc/init.d/apparmor teardown
sudo update-rc.d -f apparmor remove
```

5.重启 libvirtd。

```shell
sudo systemctl restart libvirt-bin
```

6.查看 kvm qemu virt-managers 是否安装成功。

```shell
sudo virt-manager
```

打开 virt-manager 并创建一个虚拟机。



**针对 java 微服务程序所在的服务器或容器未安装 libvirt-bin 的情况，若已安装 libvirt-bin 可不安装libvirt-dev。**

使用 libvirt Java API 管理虚拟机，Java 微服务程序所在的服务器或容器需要安装 libvirt 的动态链接库，在 Ubuntu 16.04 下使用 APT 源和 docker 容器的方式安装 libvirt-dev 包。

#### libvirt-dev 安装

##### libvirt-dev APT 源安装

```cmd
sudo apt-get install -y libvirt-dev1
```

安装完成后，在 usr/lib 或 /usr/lib/x86_64-linux-gnu 中可以看到以 libvirt 开头的相关链接库文件。如图所示。

![图1](http://or2qjfdhl.bkt.clouddn.com/20170624140137.png)

##### docker 容器安装（如不使用 docker 容器可忽略）

提供一种基于 Java 基镜像制作 libvirt-dev 基镜像的方法。

制作 libvirt-dev 镜像的 Dockerfile 文件内容如下。

```docker
FROM java
MAINTAINER kyyee "kyyee.com"
RUN apt-get update -y
RUN apt-get upgrade -y
# Installing the Dynamic Link Library: libvirt-dev
RUN apt-get install -y libvirt-dev
# clean the backup
RUN apt-get clean
```

在安装 libvirt-dev 过程中有**一个 Error**，但这个基镜像仍然可以正常使用，暂不知道会有什么其他影响。

后续的微服务程序可基于此基镜像打包。



### 源码编译安装

**转：** 以下内容均来自 YY 游戏云平台组。

1.下载 QEMU 源码。

```shell
cd /tmp
wget http://download.qemu-project.org/qemu-2.9.0.tar.xz
tar xvJf qemu-2.9.0.tar.xz
cd qemu-2.9.01234
```

2.安装 QEMU 相关依赖。

```shell
sudo apt-get install -y build-essential pkg-config zliblg-dev
sudo apt-get install -y libglib2.0-dev libaio-dev librdb-dev
sudo apt-get install -y autoconf automake libtool123
```

3.配置和编译安装 QEMU。

```shell
# 编译参数详情参考 ./configure --help
sudo ./configure --prefix=/usr/local/qemu2.9\
--target-list=x86_64-softmmu\
--enable-kvm --disable-docs\
--enable-linux-aio\
--disable-guest-agent\
--enable-vnc\
--enable-vhost-net\
--disable-xen\
--enable-rdb
sudo make && sudo make install
sudo ln -s /usr/local/qemu2.9/bin/* /usr/local/bin/123456789101112
```

4.下载 libvirt 源码。

```cmd
cd /tmp
wget http://libvirt.org/sources/libvirt-2.0.0.tar.xz
tar -xvf libvirt-2.0.0.tar.xz
cd libvirt-2.0.0.tar.xz1234
```

5.安装 libvirt 相关依赖。

```shell
sudo apt-get install -y libyajl-dev libxml2-dev
sudo apt-get install -y libdevmapper1.02.1 libdevmapper-dev
sudo apt-get install -y libnl-3-dev libnl-route-3-dev123
```

6.配置和编译安装 libvirt。

```shell
sudo ./configure --prefix=/usr/local/libvirt2.0.0
sudo make && sudo make install
LIBVIRT_HOME=/usr/local/libvirt2.0.0
sudo ln -s $LIBVIRT_HOME/bin/* /usr/bin/
sudo ln -s $LIBVIRT_HOME/etc/libvirt /etc/libvirt/
sudo ln -s $LIBVIRT_HOME/lib/systemd/system/* /lib/systemd/system/
sudo systemctl enable libvirtd
sudo ln -s /lib/systemd/system/libvirtd.service /lib/systemd/system/libvirt-bin.service
12345678
```



## 连接

### 项目加入 jar 包依赖

#### maven

引入 JNA 和 libvirt Java SDK 库。

在 [mvnrepository](https://mvnrepository.com/) 搜索 [libvirt](https://mvnrepository.com/artifact/org.libvirt/libvirt)，选择最新版本 [0.5.1](https://mvnrepository.com/artifact/org.libvirt/libvirt/0.5.1)，网页下面可以看到 JNA 依赖，JNA 的依赖版本为 [3.5.0](https://mvnrepository.com/artifact/net.java.dev.jna/jna/3.5.0)，也可引入 JNA 最新版 [4.4.0](https://mvnrepository.com/artifact/net.java.dev.jna/jna/4.4.0)。

```mvn
<!-- https://mvnrepository.com/artifact/org.libvirt/libvirt -->
<dependency>
    <groupId>org.libvirt</groupId>
    <artifactId>libvirt</artifactId>
    <version>0.5.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/net.java.dev.jna/jna -->
<dependency>
    <groupId>net.java.dev.jna</groupId>
    <artifactId>jna</artifactId>
    <version>3.5.0</version>
</dependency>
```

#### gradle

和 maven 类似，不在过多介绍。

```gradle
// https://mvnrepository.com/artifact/org.libvirt/libvirt
compile group: 'org.libvirt', name: 'libvirt', version: '0.5.1'
// https://mvnrepository.com/artifact/net.java.dev.jna/jna
compile group: 'net.java.dev.jna', name: 'jna', version: '3.5.0'
```



要想通过 libvirt 创建虚拟机，首先要和 Hypervisor 建立连接。libvirt Java API提供了 Connect 对象来建立连接。连接成功后，所有操作都是在该连接上实现，如 libvirt Java API 提供了 NodeInfo 对象来管理宿主机（节点），提供了 Domain、DomainInfo、MemoryStatistic（用于客户机内存监控） 对象来管理客户机（虚拟机）；提供了 StoragePool、StoragePoolInfo 对象来管理宿主机（节点）硬盘，提供了 StorageVolume、StorageVolumeInfo 对象来管理客户机（虚拟机）硬盘；提供了 Interface、Network 对象来管理网络。

### Linux 系统环境

虚拟机：VMware 12.1.0
系统版本：Ubuntu 16.04 64 位

### Connect 建立连接

Connect 对象的构造函数有 3 个重载方法。

-   Connect(String uri)
    此方法建立的连接支持读写。
-   Connect(String uri, boolean readOnly)
    此方法建立的连接可设置是否只读。
-   Connect(String uri, ConnectAuth auth, int flags)
    此方法建立的连接可传入一个安全验证对象。

构造函数中的 uri 参数可以是本地 URI，也可以是远程 URI。远程 URI 可以建立到远程宿主机（节点）的连接。

#### 本地 URI

[URI](http://libvirt.org/remote.html#Remote_basic_usage) 的格式如下：

```uri
driver:///path1
```

-   driver 表示虚拟化引擎如 qemu、xen、test 等。
-   path 表示资源范围，有两个值：system 和 session，其中 system 表示所有虚拟化资源，session 表示当前用户的虚拟化资源，通常使用 system。

示例代码：

```java
    private void localConnect() throws LibvirtException {
        logger.info("local connect execute succeeded");
        Connect connect = new Connect("qemu:///system", true);
        logger.info("连接到的宿主机的主机名：{}", connect.getHostName());
        logger.info("JNI连接的libvirt库版本号：{}", connect.getLibVirVersion());
        logger.info("连接的URI：{}", connect.getURI());
    }1234567
```

本地调用可用于熟悉 libvirt Java API，远程调用才是本系列文章的重点。

#### 远程 URI

[URI](http://libvirt.org/remote.html#Remote_URI_reference) 的格式如下：

```uri
driver[+transport]://[username@][hostname][:port]/[path][?extraparameters]1
```

-   driver 表示虚拟化引擎如 qemu、xen、test 等。
-   transport 表示传输协议如 tcp 等。
-   username 表示要连接的远程宿主机的用户名，示例代码为：kyyee。（可不填）
-   hostname 表示要连接的远程宿主机的 IP 地址，示例代码为：192.168.10.105。
-   port 表示要连接的远程宿主机的端口。（如果传输协议需要的话）
-   path 表示资源范围，有两个值：system 和 session，其中 system 表示所有虚拟化资源，session 表示当前用户的虚拟化资源，通常使用 system。
-   extraparameters 表示使用特定传输协议的额外参数。

transport 除支持 tcp 外还支持的传输协议有：tls，unix，ssh，ext，libssh2，libssh，如果没有特别说明，默认的传输协议是tls。有关这些协议的更多说明可参考官网，[传送门](http://libvirt.org/remote.html#Remote_transports)。
extraparameters 的更多用法可参考官网，[传送门](http://libvirt.org/remote.html#Remote_URI_parameters)。

下文所有的示例代码均使用 **tcp** 的方式连接宿主机。
tcp 是一种未加密的 TCP/IP socket，不推荐在生产环境使用，本系列文章使用 tcp 只是为了方便测试。

libvirt 默认配置没有开启 tcp 传输协议支持。按照如下方式可以开启 TCP 连接。

修改 /etc/libvirt/libvirtd.conf

```cmd
sudo gedit /etc/libvirt/libvirtd.conf1
# 禁用 tls 传输协议
listen_tls = 0
# 启用 tcp 传输协议
listen_tcp = 1
# tcp 端口 16509
tcp_port = "16509"
listen_addr = "0.0.0.0"
# 关闭 tcp 认证
auth_tcp = "none"123456789
```

修改/etc/default/libvirt-bin

```cmd
sudo gedit /etc/default/libvirt-bin1
start_libvirtd="yes"
libvirtd_opts="-d -l"12
```

libvirtd.conf 的更多配置方式可参考官网，[传送门](http://libvirt.org/remote.html#Remote_libvirtd_configuration)

重启 libvirtd

```cmd
sudo /etc/init.d/libvirt-bin restart1
```

重启成功后就可以使用 tcp 传输协议连接到 libvirtd 了，但像上面这样设置后连接是无安全认证的，不推荐在生产环境使用。

认证授权的更多用法可参考官网，[传送门](http://libvirt.org/auth.html)。

示例代码远程连接 libvirtd 的 uri 为：qemu+tcp://192.168.10.105:16509/system

连接示例代码：

```java
    private void remoteConnectByTcp() throws LibvirtException {
        logger.info("remote connect execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.105:16509/system", true);
        logger.info("连接到的宿主机的主机名：{}", connect.getHostName());
        logger.info("JNI连接的libvirt库版本号：{}", connect.getLibVirVersion());
        logger.info("连接的URI：{}", connect.getURI());
        logger.info("连接到的宿主机的剩余内存：{}", connect.getFreeMemory());
        logger.info("连接到的宿主机的最大Cpu输了：{}", connect.getMaxVcpus(null));
        logger.info("hypervisor的名称：{}", connect.getType());
        logger.info("hypervisor的容量（返回的是一个xml字符串，用处不大）：\n{}", connect.getCapabilities());
    }1234567891011
```

Connect 对象的 getMaxVcpus(String type) 的方法，这个地方传入的参数可以是 null，也可以是虚拟机对应 xml 文件中元素的 type 属性，也就是说 qemu 是 kvm ，xen 是 xen，推荐直接填 null。

Capabilities 描述 xml 的更多信息可参考官网：[传送门](http://libvirt.org/formatcaps.html)

### 管理镜像

QEMU 自带一个虚拟机磁盘管理工具 qemu-img，它可以用来创建或克隆虚拟机，KVM 虚拟机常用的镜像格式有 raw 和 qcow2，raw 是简单的二进制镜像文件格式，qcow2 是主流的一种虚拟化镜像文件格式。qcow2 比 raw 的优势在于：更小的磁盘空间占用，支持写时复制，支持快照，支持压缩和加密。

使用 qemu-img 创建或克隆镜像不是本系列文章的重点，下面将用示例代码的方式演示如何调用 libvirt Java API 创建和克隆镜像。

创建镜像是在宿主机上，准确的说是在宿主机的物理硬盘上创建镜像，而 libvirt 提供的 StoragePool 就是专门用来管理宿主机物理硬盘的。

#### 管理宿主机物理硬盘

libvirt 为我们提供了一个默认的 StoragePool，名称叫 default，对应宿主机的 /var/lib/libvirt/images/ 物理空间。

##### 遍历 StoragePool

libvirt 提供了一个接口可以遍历宿主机的 StoragePool。

遍历示例代码：

```java
    private void listStoragePool() throws LibvirtException {
        logger.info("list storage pool execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.105:16509/system", true);

        String[] pools = connect.listStoragePools();
        logger.info("存储池个数：{}", pools.length);
        for (String pool : pools) {
            logger.info("存储池名称：{}", pool);
        }
    }12345678910
```

![图1](http://or2qjfdhl.bkt.clouddn.com/20170626212447.png)

这段代码也证实了 libvirt 提供的默认 StoragePool 的名称为：default。

##### 查询 StoragePool

Connect 对象提供了 4 种方式获得 StoragePool 对象，除去一种已过时的方法，还有：

-   StoragePool storagePoolLookupByName(String name)
    通过唯一的名称获取 storage pool。
-   StoragePool storagePoolLookupByUUID(UUID uuid)
    通过全局唯一的 id 获取 storage pool。
-   StoragePool storagePoolLookupByUUIDString(String UUID)
    通过全局唯一的 id 获取 storage pool。

通过唯一的名称获取 storage pool 的示例代码：

```java
    private void getStoragePoolbyName(@RequestParam String name) throws LibvirtException {
        logger.info("get storage pool by name execute succeeded");
        logger.info("request parameter:{}",name);
        Connect connect = new Connect("qemu+tcp://192.168.10.105:16509/system", true);

        StoragePool storagePool = connect.storagePoolLookupByName("default");
        StoragePoolInfo storagePoolInfo = storagePool.getInfo();

        logger.info("存储池的状态：{}", storagePoolInfo.state);
        logger.info("存储池的容量：{}GB", storagePoolInfo.capacity / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储池的可用容量：{}GB", storagePoolInfo.available / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储池的已用容量：{}GB", storagePoolInfo.allocation / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储池的描述xml：\n {}", storagePool.getXMLDesc(0));
    }1234567891011121314
```

**备注：**这几个属性取得的容量值的单位都是bytes。

![图2](http://or2qjfdhl.bkt.clouddn.com/20170626213845.png)

![图3](http://or2qjfdhl.bkt.clouddn.com/20170626214408.png)

从图中可以看出，获取到的磁盘容量有部分损失，作为监控宿主机和分配资源使用还是可行的。

##### 创建 StoragePool

StoragePool 对象的方法 String getXMLDesc(int flags) 可以取得 StoragePool 配置描述 xml，可依照这个描述自定义 StoragePool 配置描述 xml。

示例 StoragePool 配置描述 xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<pool type="dir">
    <name>virtimages</name> <!--名称必须唯一-->
    <source>
    </source>
    <capacity unit='GiB'>40</capacity> <!--StoragePool 的容量-->
    <allocation>0</allocation> <!--StoragePool 的已用容量-->
    <target>
        <path>/home/kyyee/images</path> <!--StoragePool 在宿主机的路径-->
        <permissions> <!--权限-->
            <mode>0711</mode>
            <owner>0</owner>
            <group>0</group>
        </permissions>
    </target>
</pool>1234567891011121314151617
```

StoragePool 配置描述 xml 的更多用法可参考官网：[传送门](http://libvirt.org/formatstorage.html#StoragePool)

有了 StoragePool 配置描述 xml，就可以很轻松的创建或定义 StoragePool 了。

创建 storage pool 的示例代码（此代码未验证）：

```java
    private void createStoragePool() throws LibvirtException, DocumentException {
        logger.info("create storage pool execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.105:16509/system");

        // xml 文件 => Dom4j 文档 => String
        SAXReader reader = new SAXReader();
        Document document = reader.read(new File("xml/kvmdemo-storage-pool.xml"));
        String xmlDesc = document.asXML();

        StoragePool storagePool = connect.storagePoolCreateXML(xmlDesc, 0);
        StoragePoolInfo storagePoolInfo = storagePool.getInfo();

        logger.info("存储池的状态：{}", storagePoolInfo.state);
        logger.info("存储池的容量：{}GB", storagePoolInfo.capacity / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储池的可用容量：{}GB", storagePoolInfo.available / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储池的已用容量：{}GB", storagePoolInfo.allocation / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储池的描述xml：\n {}", storagePool.getXMLDesc(0));
    }123456789101112131415161718
```

##### 定义 StoragePool

定义 storage pool 的示例代码（此代码未验证）：

```java
    private void defineStoragePool() throws LibvirtException, DocumentException {
        logger.info("define storage pool execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.231:16509/system");

        // xml 文件 => Dom4j 文档 => String
        SAXReader reader = new SAXReader();
        Document document = reader.read(new File("xml/kvmdemo-storage-pool.xml"));
        String xmlDesc = document.asXML();
        logger.info("defineStoragePool description:\n{}", xmlDesc);

        StoragePool storagePool = connect.storagePoolDefineXML(xmlDesc, 0);
        StoragePoolInfo storagePoolInfo = storagePool.getInfo();

        logger.info("存储池的状态：{}", storagePoolInfo.state);
        logger.info("存储池的容量：{}GB", storagePoolInfo.capacity / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储池的可用容量：{}GB", storagePoolInfo.available / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储池的已用容量：{}GB", storagePoolInfo.allocation / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储池的描述xml：\n {}", storagePool.getXMLDesc(0));
    }12345678910111213141516171819
```

创建 storage pool 除了可调用 StoragePool storagePoolCreateXML(String xmlDesc, int flags) 方法外，还可调用 StoragePool storagePoolDefineXML(String xml, int flags) 。
具体的，storagePoolCreateXML 创建的是一个临时的 pool ，删除它只需调用 destroy()，或者重启宿主机，就会消失；而 storagePoolDefineXML 定义的是一个持久化的 pool，除非明确调用 undefine()，否则它一直存在。

##### 删除 StoragePool

删除 storage pool 示例代码：

```java
    private void deleteStoragePool() throws LibvirtException, DocumentException {
        logger.info("delete storage pool execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.231:16509/system");

        StoragePool storagePool = connect.storagePoolLookupByName("default");
        logger.info("存储池名称：{}", storagePool.getName());
//        storagePool.free();
        storagePool.destroy();
        storagePool.undefine();
    }12345678910
```

#### 管理客户机（虚拟机）镜像

前面讲了这么多，都是为了给创建虚拟机镜像做铺垫。libvirt 提供的 StorageVolume 就是专门用来管理客户机物理硬盘的，由于 StorageVolume 与 StoragePool 关系密切，前面的内容是实现创建虚拟机镜像的基础。

下面将根据讲解 StoragePool 的层次结构来讲解 StorageVolume。

##### 遍历 StorageVolume

libvirt 提供了一个接口可以遍历 StoragePool 下的所有的 StorageVolume。

遍历示例代码：

```java
    private void listStorageVolume() throws LibvirtException {
        logger.info("list storage volume execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.231:16509/system", true);

        StoragePool storagePool = connect.storagePoolLookupByName("default");
        String[] volumes = storagePool.listVolumes();
        logger.info("存储卷个数：{}", volumes.length);
        for (String volume : volumes) {
            if (volume.contains("iso")) continue; // 过滤掉 iso 文件
            logger.info("存储卷名称：{}", volume);
        }
    }123456789101112
```

![图4](http://or2qjfdhl.bkt.clouddn.com/20170626225416.png)

![图5](http://or2qjfdhl.bkt.clouddn.com/20170626225718.png)

这里有一个文件是 iso 文件，被过滤了，没有显示。

##### 查询 StorageVolume

StoragePool 对象提供了 1 种方式获得 StorageVol 对象：

-   StorageVol storageVolLookupByName(String name)
    通过唯一的名称获取 storage volume。

通过唯一的名称获取 storage volume 的示例代码：

```java
    private void getStorageVolumebyName() throws LibvirtException {
        logger.info("get storage volume by name execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.231:16509/system", true);

        StoragePool storagePool = connect.storagePoolLookupByName("default");
        String[] volumes = storagePool.listVolumes();
        for (String volume : volumes) {
            if (volume.contains("iso")) continue; // 过滤掉 iso 文件

            StorageVol storageVol = storagePool.storageVolLookupByName(volume);
            StorageVolInfo storageVolInfo = storageVol.getInfo();

            logger.info("存储卷名称：{}", volume);
            logger.info("存储卷的类型：{}", storageVolInfo.type);
            logger.info("存储卷的容量：{} GB", storageVolInfo.capacity / 1024.00 / 1024.00 / 1024.00);
            logger.info("存储卷的可用容量：{} GB", (storageVolInfo.capacity - storageVolInfo.allocation) / 1024.00 / 1024.00 / 1024.00);
            logger.info("存储卷的已用容量：{} GB", storageVolInfo.allocation / 1024.00 / 1024.00 / 1024.00);
            logger.info("存储卷的描述xml：\n {}", storageVol.getXMLDesc(0));
        }
    }1234567891011121314151617181920
```

**备注：**这几个属性取得的容量值的单位都是bytes。

![图6](http://or2qjfdhl.bkt.clouddn.com/20170626230631.png)

![图7](http://or2qjfdhl.bkt.clouddn.com/20170626230711.png)

获取客户机（虚拟机）磁盘容量非常准确，我给客户机分配的磁盘空间就是 15 GB，将这个 api 作为监控客户机（虚拟机）使用完全没有问题。

##### 创建 StorageVolume

StorageVol 对象的方法 String getXMLDesc(int flags) 可以取得 StorageVol 配置描述 xml，可依照这个描述自定义 StorageVol 配置描述 xml。

示例 StorageVol 配置描述 xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<volume type='file'>
    <name>kvmdemo.qcow2</name> <!--名称必须唯一-->
    <source>
    </source>
    <capacity unit='GiB'>10</capacity> <!--StorageVol 的容量-->
    <allocation>0</allocation> <!--StorageVol 的已用容量-->
    <target>
        <path>/var/lib/libvirt/images/kvmdemo.qcow2</path> <!--StorageVol 在宿主机的路径-->
        <format type='qcow2'/> <!--文件类型，通常都是 qcow2，也可以是 raw-->
        <permissions> <!--权限-->
            <mode>0600</mode>
            <owner>0</owner>
            <group>0</group>
        </permissions>
    </target>
</volume>123456789101112131415161718
```

StorageVol 配置描述 xml 的更多用法可参考官网：[传送门](http://libvirt.org/formatstorage.html#StorageVol)

有了 StorageVol 配置描述 xml，就可以很轻松的创建或克隆 StorageVol 了。

创建 storage volume 的示例代码：

```java
    private void createStorageVolume() throws LibvirtException, DocumentException {
        logger.info("create storage volume execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.231:16509/system");

        // xml 文件 => Dom4j 文档 => String
        SAXReader reader = new SAXReader();
        Document document = reader.read(new File("xml/kvmdemo-storage-vol.xml"));
        String xmlDesc = document.asXML();
        logger.info("createStorageVolume description:\n{}", xmlDesc);

        StoragePool storagePool = connect.storagePoolLookupByName("default");
        logger.info("This could take some times at least 3min...");
        StorageVol storageVol = storagePool.storageVolCreateXML(xmlDesc, 0);
        logger.info("create success");
        logger.info("createStorageVolume name:{}", storageVol.getName());
        logger.info("createStorageVolume path:{}", storageVol.getPath());
        StorageVolInfo storageVolInfo = storageVol.getInfo();

        logger.info("存储卷的类型：{}", storageVolInfo.type);
        logger.info("存储卷的容量：{} GB", storageVolInfo.capacity / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储卷的可用容量：{} GB", (storageVolInfo.capacity - storageVolInfo.allocation) / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储卷的已用容量：{} GB", storageVolInfo.allocation / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储卷的描述xml：\n {}", storageVol.getXMLDesc(0));
    }123456789101112131415161718192021222324
```

这里需要一个 xml 工具包来操作 xml 文件，示例代码使用的是 [dom4j](https://dom4j.github.io/)

创建 storage volume 除了可调用 StorageVol storageVolCreateXML(String xmlDesc, int flags) 方法外，还可调用 StorageVol storageVolCreateXMLFrom(String xmlDesc, StorageVol cloneVolume, int flags) 。
具体的，storageVolCreateXML 创建的一个全新的镜像，如果虚拟机基于这个镜像文件创建，那么还需要挂载光盘驱动器安装操作系统才能使用虚拟机，而 storageVolCreateXMLFrom 基于传入的 cloneVolume 派生（克隆）一个镜像，但是要保证派生（克隆）的镜像的名称和路径唯一，否则派生（克隆）不成功。

##### 克隆 StorageVolume

克隆 storage volume 的示例代码：

```java
    private void cloneStorageVolume() throws LibvirtException, DocumentException {
        logger.info("clone storage volume execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.231:16509/system");

        // xml 文件 => Dom4j 文档 => String
        SAXReader reader = new SAXReader();
        Document document = reader.read(new File("xml/kvmdemo-storage-vol.xml"));
        String xmlDesc = document.asXML();
        logger.info("createStorageVolume description:\n{}", xmlDesc);

        StoragePool storagePool = connect.storagePoolLookupByName("default");
        // 克隆的基镜像，这个镜像需要自己制作，可使用 virt-manager 制作基镜像，本示例代码采用的基镜像是 Ubuntu 16.04 64位
        StorageVol genericVol = storagePool.storageVolLookupByName("generic.qcow2");
        logger.info("This could take some times at least 3min...");
        StorageVol storageVol = storagePool.storageVolCreateXMLFrom(xmlDesc, genericVol, 0);
        logger.info("clone success");
        logger.info("createStorageVolume name:{}", storageVol.getName());
        logger.info("createStorageVolume path:{}", storageVol.getPath());
        StorageVolInfo storageVolInfo = storageVol.getInfo();

        logger.info("存储卷的类型：{}", storageVolInfo.type);
        logger.info("存储卷的容量：{} GB", storageVolInfo.capacity / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储卷的可用容量：{} GB", (storageVolInfo.capacity - storageVolInfo.allocation) / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储卷的已用容量：{} GB", storageVolInfo.allocation / 1024.00 / 1024.00 / 1024.00);
        logger.info("存储卷的描述xml：\n {}", storageVol.getXMLDesc(0));
    }1234567891011121314151617181920212223242526
```

克隆镜像的基镜像需要自己制作，可使用 virt-manager 制作基镜像，本示例代码使用 virt-manager 制作的 Ubuntu 16.04 64 位基镜像。

##### 删除 StorageVolume

删除 storage volume 示例代码：

```java
    private void deleteStorageVolume() throws LibvirtException, DocumentException {
        logger.info("delete storage volume execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.231:16509/system");

        StoragePool storagePool = connect.storagePoolLookupByName("default");
        StorageVol storageVol = storagePool.storageVolLookupByName("kvmdemo.qcow2");
        logger.info("存储卷名称：{}", storageVol.getName());
        storageVol.wipe();
        storageVol.delete(0);
    }12345678910
```

### 管理虚拟机

前面讲了那么多“废话”，终于到了最关键的时候，接下来我们将调用 libvirt Java API 创建虚拟机。但是在这之前，我们还有一件重要的事情，需将宿主机（节点）的网卡设置为桥接模式。

#### 将宿主机（节点）网络设置为桥接模式

##### 固定 IP 的桥接模式

笔者使用的是这种方式，优势在于 IP 固定，方便开发，不用担心 IP 更改，造成服务不可用，但是配置稍复杂。

控制台输入如下命令。

```cmd
sudo gedit /etc/network/interfaces1
```

打开 gedit，默认情况下有如下文本：

```gedit
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback123
```

末尾换行追加如下文本：

```gedit
auto ens33
iface ens33 inet manual
auto br0
iface br0 inet static
address 192.168.10.231
netmask 255.255.255.0
broadcast 192.168.10.255
gateway 192.168.10.1
dns-nameserver 192.168.10.1
bridge_ports ens33
bridge_stp off
bridge_fd 0
bridge_maxwait 012345678910111213
```

address 是你希望设置的 IP 地址，桥接模式下的 IP 需与物理主机的 IP 在同一网段。
dns-nameserver 是 DNS 地址，与物理主机的 DNS 地址一致即可，如果不配置 dns-nameserver 将无法识别公网 url，如 <http://blog.csdn.net/kyyee>。

保存退出后，控制台输入如下命令重启网络。

```cmd
sudo /etc/init.d/networking restart1
```

##### DHCP 获取 IP 等信息的桥接模式

这种方式，宿主机（节点）直接从 DHCP 服务器获取 IP 等信息，免于配置 IP 等信息，缺点就是 IP 随时可能会变。

控制台输入如下命令。

```cmd
sudo gedit /etc/network/interfaces1
```

打开 gedit，默认情况下有如下文本：

```gedit
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback123
```

末尾换行追加如下文本：

```gedit
auto ens33
iface ens33 inet manual
auto br0
iface br0 inet dhcp
bridge_ports ens33
bridge_stp off
bridge_fd 01234567
```

保存退出后，控制台输入如下命令重启网络。

```cmd
sudo /etc/init.d/networking restart1
```

#### 管理虚拟机生命周期

在管理镜像的章节，我们创建了一个名为：**kvmdemo.qcow2**的镜像，现在这个镜像终于派上用场了，接下来的虚拟机将使用该镜像作为自己的虚拟硬盘，libvirt Java API 提供的与虚拟机生命周期相关的对象是 Domian。

下面将根据讲解 StorageVolume 的层次结构来讲解 Domain。

##### 遍历宿主机上的客户机（虚拟机）

libvirt 提供了一个接口可以遍历 Connect 下的所有的 Domain。

遍历示例代码：

```java
    private void listDomain() throws LibvirtException {
        logger.info("list domain execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.231:16509/system", true);

        int[] idsOfDomain = connect.listDomains();
        logger.info("正在运行的虚拟机个数：{}", idsOfDomain.length);
        for (int id : idsOfDomain) {
            logger.info("正在运行的虚拟机id：{}", id);
        }

        String[] namesOfDefinedDomain = connect.listDefinedDomains();
        logger.info("已定义未运行的虚拟机个数：{}", namesOfDefinedDomain.length);
        for (String name : namesOfDefinedDomain) {
            logger.info("已定义未运行的虚拟机名称：{}", name);
        }
    }
```

![图8](http://or2qjfdhl.bkt.clouddn.com/20170628231223.png)

![图9](http://or2qjfdhl.bkt.clouddn.com/20170628231336.png)

##### 查询客户机（虚拟机）

Connect 对象提供了 5 种方式获得 Domain 对象：

-   Domain domainLookupByID(int id)
    通过唯一的 id 获取客户机（虚拟机）。
-   Domain domainLookupByName(String name)
    通过唯一的名称获取客户机（虚拟机）。
-   Domain domainLookupByUUID(int[] UUID)
    通过唯一的 UUID 获取客户机（虚拟机）。
-   Domain domainLookupByUUID(UUID uuid)
    通过唯一的 UUID 获取客户机（虚拟机）。
-   Domain domainLookupByUUIDString(String UUID)
    通过唯一的 UUID 获取客户机（虚拟机）。

这几个通过 UUID 获取客户机（虚拟机）的方法，传入的参数有些微差异，这是根据你定义虚拟机时使用的 UUID 规则决定的，本系列文章主要使用名称获取客户机（虚拟机），不会使用 UUID 获取客户机（虚拟机），这里不在过多介绍。

通过唯一的id和名称获取 domain 的示例代码：

```java
    private void getDomainbyIdOrName() throws LibvirtException {
        logger.info("get domain by id execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.231:16509/system", true);

        int[] idsOfDomain = connect.listDomains();
        logger.info("正在运行的虚拟机个数：{}", idsOfDomain.length);
        for (int id : idsOfDomain) {
            Domain domain = connect.domainLookupByID(id);
            logger.info("虚拟机的id：{}", domain.getID());
            logger.info("虚拟机的uuid：{}", domain.getUUIDString());
            logger.info("虚拟机的名称：{}", domain.getName());
            logger.info("虚拟机的是否自动启动：{}", domain.getAutostart());
            logger.info("虚拟机的状态：{}", domain.getInfo().state);
        }

        String[] uuidsOfDefinedDomain = connect.listDefinedDomains();
        logger.info("已定义未运行的虚拟机个数：{}", uuidsOfDefinedDomain.length);
        for (String uuid : uuidsOfDefinedDomain) {
            Domain domain = connect.domainLookupByName(uuid);
            logger.info("虚拟机的id：{}", domain.getID());
            logger.info("虚拟机的uuid：{}", domain.getUUIDString());
            logger.info("虚拟机的名称：{}", domain.getName());
            logger.info("虚拟机的是否自动启动：{}", domain.getAutostart());
            logger.info("虚拟机的状态：{}", domain.getInfo().state);
        }
    }
```

![图10](http://or2qjfdhl.bkt.clouddn.com/20170628233702.png)

![图11](http://or2qjfdhl.bkt.clouddn.com/20170628233634.png)

##### 创建客户机（虚拟机）

Domain 对象的方法 String getXMLDesc(int flags) 可以取得客户机（虚拟机）配置描述 xml，可依照这个描述自定义客户机（虚拟机）配置描述 xml。

示例客户机（虚拟机）配置描述 xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<domain type='kvm'>
    <name>kvmdemo</name> <!--名称必须唯一-->
    <uuid>c6e408f3-7750-47ca-8bd1-d19837271472</uuid> <!--uuid必须唯一，可使用 java.util.UUID 随机生成-->
    <memory unit='MiB'>512</memory> <!--最大可用内存配置-->
    <currentMemory unit='MiB'>512</currentMemory>
    <vcpu placement='static'>1</vcpu> <!--配置cpu-->
    <os>
        <type arch='x86_64' machine='pc'>hvm</type>
        <boot dev='hd'/> <!--硬盘启动-->
        <boot dev='cdrom'/> <!--光驱启动-->
    </os>
    <features>
        <acpi/>
        <apic/>
        <pae/>
    </features>
    <clock offset='localtime'/>
    <on_poweroff>destroy</on_poweroff>
    <on_reboot>restart</on_reboot>
    <on_crash>restart</on_crash>
    <devices>
        <emulator>/usr/bin/qemu-system-x86_64</emulator> <!--模拟器所在路径，视自己情况配置-->
        <disk type='file' device='disk'>
            <driver name='qemu' type='qcow2'/>
            <source file='/var/lib/libvirt/images/kvmdemo.qcow2'/> <!--虚拟硬盘配置，这个地方填生成的镜像文件所在的路径即可-->
            <target dev='hda' bus='ide'/>
        </disk>
        <!--<disk type='file' device='cdrom'>
            <source file='/var/lib/libvirt/images/ubuntu-16.04-desktop-amd64.iso'/>
            <target dev='hdb' bus='ide'/>
            <readonly/>
        </disk>-->
        <interface type='bridge'> <!--网络配置，本示例配置成桥接模式-->
            <mac address='52:54:00:f4:06:03'/> <!--mac 地址必须唯一-->
            <source bridge='br0'/>
        </interface>
        <console type='pty'> <!--控制台配置，如果需要使用 virsh 命令登陆虚拟机，则必须添加-->
            <target port='0'/>
        </console>
        <input type='tablet' bus='usb'/>
        <input type='mouse' bus='ps2'/>
        <input type='keyboard' bus='ps2'/>
        <graphics type='vnc' autoport='yes' keymap='en-us'
                  listen='0.0.0.0'/> <!--VNC配置，autoport="yes"表示自动分配VNC端口，推荐使用，listen="0.0.0.0"表示监听所有IP-->
        <memballoon model="virtio"> <!--内存监控配置，添加此配置，才能正常取得内存使用情况-->
            <stats period="10"/><!--每10s钟收集一次-->
        </memballoon>
    </devices>
</domain>
```

关于如何随机生成 Mac 地址可查看我的另一篇博文：[传送门](http://blog.csdn.net/kyyee/article/details/72873148)

客户机（虚拟机）配置描述 xml 的更多用法可参考官网：[传送门](http://libvirt.org/formatdomain.html)

有了客户机（虚拟机）配置描述 xml，就可以很容易的创建或定义客户机（虚拟机）了。

创建客户机（虚拟机）的示例代码：

```java
    private void createDomain() throws LibvirtException, DocumentException {
        logger.info("create domain execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.231:16509/system");

        // xml 文件 => Dom4j 文档 => String
        SAXReader reader = new SAXReader();
        Document document = reader.read(new File("xml/kvmdemo.xml"));
        String xmlDesc = document.asXML();
        logger.info("createDomain description:\n{}", xmlDesc);

        Domain domain = connect.domainCreateXML(xmlDesc, 0);

        logger.info("虚拟机的id：{}", domain.getID());
        logger.info("虚拟机的uuid：{}", domain.getUUIDString());
        logger.info("虚拟机的名称：{}", domain.getName());
        logger.info("虚拟机的是否自动启动：{}", domain.getAutostart());
        logger.info("虚拟机的状态：{}", domain.getInfo().state);
        logger.info("虚拟机的描述xml：\n{}", domain.getXMLDesc(0));
    }
```

##### 定义客户机（虚拟机）

定义客户机（虚拟机）示例代码：

```java
    private void defineDomain() throws LibvirtException, DocumentException {
        logger.info("define domain execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.231:16509/system");

        // xml 文件 => Dom4j 文档 => String
        SAXReader reader = new SAXReader();
        Document document = reader.read(new File("xml/kvmdemo.xml"));
        String xmlDesc = document.asXML();
        logger.info("defineDomain description:\n{}", xmlDesc);

        Domain domain = connect.domainDefineXML(xmlDesc);
//        domain.abortJob();
        // 是否随宿主机开机自动启动
        domain.setAutostart(false);

        domain.create(); // 定义完后直接启动
        logger.info("虚拟机的id：{}", domain.getID());
        logger.info("虚拟机的uuid：{}", domain.getUUIDString());
        logger.info("虚拟机的名称：{}", domain.getName());
        logger.info("虚拟机的是否自动启动：{}", domain.getAutostart());
        logger.info("虚拟机的状态：{}", domain.getInfo().state);
        logger.info("虚拟机的描述xml：\n{}", domain.getXMLDesc(0));
    }
```

这里需要一个 xml 工具包来操作 xml 文件，示例代码使用的是 [dom4j](https://dom4j.github.io/)

这里之所以列出了 Domain domainCreateXML(String xmlDesc, int flags) 和 Domain domainDefineXML(String xmlDesc) 的示例代码，是由于这两个方法在官方文档中的说明不够详细。

具体的，domainCreateXML 创建的客户机（虚拟机）是一个临时的，当调用 destroy() 或者宿主机重启后，就会消失，这和 storagePoolCreateXML 类似；而 domainDefineXML 定义的是一个持久化的客户机（虚拟机），除非明确调用 undefine()，否则它一直存在，此外，domainDefineXML 会覆盖之前的定义，但是有些操作会阻止这个操作，比如 block copy 操作，要先使用 abortJob() 操作取消这些块拷贝操作。

##### 删除客户机（虚拟机）

删除客户机（虚拟机）示例代码：

```java
    private void undefineDomain() throws LibvirtException, DocumentException {
        logger.info("undefine domain execute succeeded");
        Connect connect = new Connect("qemu+tcp://192.168.10.231:16509/system");

        Domain domain = connect.domainLookupByName("kvmdemo");
        logger.info("虚拟机的id：{}", domain.getID());
        logger.info("虚拟机的uuid：{}", domain.getUUIDString());
        logger.info("虚拟机的名称：{}", domain.getName());
        logger.info("虚拟机的是否自动启动：{}", domain.getAutostart());
        logger.info("虚拟机的状态：{}", domain.getInfo().state);
        domain.destroy(); // 强制关机
        domain.undefine();
    }
```

未完待续，后续将继续发布客户机启动、关机、强制关机、重启、强制重启；宿主机监控，客户机监控等。

