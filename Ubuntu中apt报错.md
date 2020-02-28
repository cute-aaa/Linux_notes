# Ubuntu中apt报错

## E: Unable to lock the administration directory (/var/lib/dpkg/), is another process using it?

原因：

apt没有正常关闭，还在后台活着



**解决方法1：**

重启解决90%问题

**解决方法2：**

**1、找到并且杀掉所有的apt-get 和apt进程**

 

​    运行下面的命令来生成所有含有 apt 的进程列表，你可以使用ps和grep命令并用管道组合来得到含有apt或者apt-get的进程。

```
ps -A | grep apt
```

找出所有的 apt 以及 apt-get 进程

```
$ sudo kill -9 processnumber
或者
$ sudo kill -SIGKILL processnumber
```

 

```
比如，下面命令中的9是 SIGKILL 的信号数，它会杀掉第一个 apt 进程
```

 

```
$ sudo kill -9 进程ID
或者
$ sudo kill -SIGKILL  进程ID
```

 



 **解决方法3、删除锁定文件**

 

 锁定的文件会阻止 Linux 系统中某些文件或者数据的访问，这个概念也存在于 Windows 或者其他的操作系统中。

 一旦你运行了 apt-get 或者 apt 命令，锁定文件将会创建于 `/var/lib/apt/lists/`、`/var/lib/dpkg/`、`/var/cache/apt/archives/` 中。

 这有助于运行中的 apt-get 或者 apt 进程能够避免被其它需要使用相同文件的用户或者系统进程所打断。当该进程执行完毕后，锁定文件将会删除。

​    当你没有看到 apt-get 或者 apt 进程的情况下在上面两个不同的文件夹中看到了锁定文件，这是因为进程由于某个原因被杀掉了，因此你需要删除锁定文件来避免该错误。

 首先运行下面的命令来移除 `/var/lib/dpkg/` 文件夹下的锁定文件：

```
$ sudo rm /var/lib/dpkg/lock
```

 之后像下面这样强制重新配置软件包：

```
$ sudo dpkg --configure -a
```

 也可以删除 `/var/lib/apt/lists/` 以及缓存文件夹下的锁定文件：

```
$ sudo rm /var/lib/apt/lists/lock
$ sudo rm /var/cache/apt/archives/lock
```

 接下来，更新你的软件包源列表：

```
$ sudo apt update
或者
$ sudo apt-get update
```



## dpkg安装应用时的依赖问题

```sh
root@sure:~/下载# dpkg -i virtualbox-6.0_6.0.8-130520_Ubuntu_bionic_amd64.deb 
正在选中未选择的软件包 virtualbox-6.0。
(正在读取数据库 ... 系统当前共安装有 162020 个文件和目录。)
正准备解包 virtualbox-6.0_6.0.8-130520_Ubuntu_bionic_amd64.deb  ...
正在解包 virtualbox-6.0 (6.0.8-130520~Ubuntu~bionic) ...
dpkg: 依赖关系问题使得 virtualbox-6.0 的配置工作不能继续：
 virtualbox-6.0 依赖于 libqt5opengl5 (>= 5.0.2)；然而：
  未安装软件包 libqt5opengl5。
 virtualbox-6.0 依赖于 libqt5printsupport5 (>= 5.0.2)；然而：
  未安装软件包 libqt5printsupport5。
 virtualbox-6.0 依赖于 libqt5x11extras5 (>= 5.6.0)；然而：
  未安装软件包 libqt5x11extras5。

dpkg: 处理软件包 virtualbox-6.0 (--install)时出错：
 依赖关系问题 - 仍未被配置
正在处理用于 ureadahead (0.100.0-21) 的触发器 ...
正在处理用于 systemd (237-3ubuntu10.23) 的触发器 ...
正在处理用于 gnome-menus (3.13.3-11ubuntu1.1) 的触发器 ...
正在处理用于 desktop-file-utils (0.23-1ubuntu3.18.04.2) 的触发器 ...
正在处理用于 mime-support (3.60ubuntu1) 的触发器 ...
正在处理用于 hicolor-icon-theme (0.17-2) 的触发器 ...
正在处理用于 shared-mime-info (1.9-2) 的触发器 ...
在处理时有错误发生：
 virtualbox-6.0
```

解决：使用apt

```sh
apt -fy install
```

会自动检测没有安装成功的软件包，并下载依赖

然后再使用dpkg安装一次就可以了





## snapd引起的问题

删除已有的出错的snapd：

vim /var/lib/dpkg/info/snapd.prerm ，第二行加上 exit 0，就是骗过shell脚本。

再运行：dpkg –purge –force-all snapd 强制删除



【可选】

然后检查：vim /etc/apt/sources.list ，注释掉所有的包含“trusty”的源，它是造成错误的根源，因为版本不同，这个是14版本，需要的是16版本。

再运行apt-get update，和 apt-get -f install 修复。