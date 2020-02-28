# 安卓源代码-->ISO镜像



官方建议使用Ubuntu来编译

参考

https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/

https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/

https://blog.csdn.net/dd864140130/article/details/51718187

https://blog.csdn.net/louiswangbing/article/details/6635445

## 准备工作

### 安装并配置git

```shell
sudo apt-get install git 
git config --global user.email “test@test.com” 
git config --global user.name “test”
```

### 安装python

必须是python2.7，默认install python 就是2.7

```bash
sudo apt-get install python
```



### 下载源码

这里使用清华的tuna源

通过执行以下命令实现repo工具的下载和安装

```shell
# 新建一个目录，用来存放repo文件及源码
mkdir ~/bin
PATH=~/bin:$PATH
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
chmod +x repo
```

repo的运行过程中会尝试访问官方的git源更新自己，如果想使用tuna的镜像源进行更新，可以将如下内容复制到你的`~/.bashrc`里

或者直接在命令行运行，一次性的

```
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
```

硬件要求：磁盘空间15G以上



### 初始化仓库

建立工作目录:

```bash
mkdir WORKING_DIRECTORY
cd WORKING_DIRECTORY
```

初始化仓库:

```bash
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest
```

如果只需要某个特定的Android版本：

```bash
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-9.0.0_r9
```

  

同步源码树（以后只需执行这条命令来同步）：

```
repo sync
```

---

同步时间会很长，并且经常出错，（一定要确保代码完全同步，不然编译会痛不欲生）

所以我们可以写个脚本

一个简单的shell脚本（最好手打，因为这是win下写的，在Linux跑可能会出现换行错误）:

```bash
#!/bin/bash
#FileName source_asyn.sh
PATH=~/bin:$PATH
# 注意修改成你要编译的版本，比如这里我在mac上编译的是Android-7.1.1——r6
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-7.1.1_r6
repo sync
while [$? = 1]:
do
    echo "=========download failed,again============"
    sleep 5
    repo sync
done
```

下载完成之前会存放在当前的 `.repo` 目录下



## 编译

### 编译环境

源码下载完成后,就可以构建编译环境了.在开始之前,我们先来看看一些编译要求:

**1. 硬件要求:** 
 64位的操作系统只能编译2.3.x以上的版本,如果你想要编译2.3.x以下的,那么需要32位的操作系统. 
 磁盘空间越多越好,至少在100GB以上.意思就是,你可以去买个大点的硬盘了啊 
 如果你想要在是在虚拟机运行linux,那么至少需要16GB的RAM/swap. 
 (实际上,我非常不推荐在虚拟机中编译2.3.x以上的代码.)

**2. 软件要求:** 
 *1. 操作系统要求* 
 在[AOSP开源](https://android.googlesource.com/)中,主分支使用Ubuntu长期版本开发和测试的,因此也建议你使用Ubuntu进行编译,下面我们列出不同版本的的Ubuntu能够编译那些android版本:

  

| Android版本                | 编译要求的Ubuntu最低版本 |
| -------------------------- | ------------------------ |
| Android 6.0至AOSP master   | Ubuntu 14.04             |
| Android 2.3.x至Android 5.x | Ubuntu 12.04             |
| Android 1.5至Android 2.2.x | Ubuntu 10.04             |

   

*2. JDK版本要求* 
 除了操作系统版本这个问题外,我们还需要关注JDK版本问题,为了方便,同样我们也列出的不同Android版本的源码需要用到的JDK版本:

  

| Android版本                  | 编译要求的JDK版本 |
| ---------------------------- | ----------------- |
| AOSP的Android主线            | OpenJDK 8         |
| Android 5.x至android 6.0     | OpenJDK 7         |
| Android 2.3.x至Android 4.4.x | Oracle JDK 6      |
| Android 1.5至Android 2.2.x   | Oracle JDK 5      |

更具体的可以参看:[Google源码编译要求](https://source.android.com/source/requirements.html)

  

我现在在Ubuntu 16.04下编译AOSP主线代码,因此需要安装OpenJDK 8,执行命令如下: 
 `sudo apt-get install openjdk-8-jdk` 
 如果你需要在Ubuntu 14.04下编译AOSP主线代码,同样需要安装OpenJDK 8,此时需要执行如下命令:

​    

```bash
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

  

如果你要编译的是Android 5.x到android 6.0之间的系统版本,需要采用openjdk7.但是在Ubuntu 15.04及之后的版本的在线安装库中只支持openjdk8和openjdk9的安装.因此,如果你想要安装openjdk 7需要首先设置ppa:

```bash
sudo add-apt-repository ppa:openjdk-r/ppa 
sudo apt-get update
```

然后再执行安装命令:

```bash
sudo apt-get install openjdk-7-jdk
```

  

有时候,我们需要编译不同版本的android系统,就可能使用不同的jdk版本.关于jdk版本切换,可以使用如下命令:

```
sudo update-alternative --config java
sudo update-alternative --config javac
```

  

*3. 其他要求*

[Google官方构建编译环境指南](https://source.android.com/source/initializing.html)中已经说明了Ubuntu14.04,Ubuntu 12.04,Ubuntu 10.04需要添加的依赖,这里我们就不做介绍了.我原先以为,Ubuntu16.04的设置和Ubuntu14.04的依赖设置应该差不多,但是只能说too young too simple. 
 下面是Ubuntu16.04中的依赖设置:

​    

```bash
sudo apt-get install libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-dev g++-multilib 
sudo apt-get install -y git flex bison gperf build-essential libncurses5-dev:i386 
sudo apt-get install tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386 
sudo apt-get install dpkg-dev libsdl1.2-dev libesd0-dev
sudo apt-get install git-core gnupg flex bison gperf build-essential  
sudo apt-get install zip curl zlib1g-dev gcc-multilib g++-multilib 
sudo apt-get install libc6-dev-i386 
sudo apt-get install lib32ncurses5-dev x11proto-core-dev libx11-dev 
sudo apt-get install libgl1-mesa-dev libxml2-utils xsltproc unzip m4
sudo apt-get install lib32z-dev ccache
```

(其中几个命令中参数是重复的,但不妨碍我们)

 

#### 初始化编译环境

确保上述过程完成后,接下来我们需要初始化编译环境,命令如下:

```bash
source build/envsetup.sh
```

执行该命令结果如下: 
 ![这里写图片描述](https://img-blog.csdn.net/20160619210441129)

  

不难发现该命令只是引入了其他执行脚本,至于这些脚本做什么,目前不在本文中细说. 
 该命令执行成功后,我们会得到了一些有用的命令,比如最下面要用到的lunch命令.



### 编译源码

初始化编译环境之后,就进入源码编译阶段.这个阶段又包括两个阶段:选择编译目标和执行编译.

### 选择编译目标

通过lunch指令设置编译目标,所谓的编译目标就是生成的镜像要运行在什么样的设备上.这里我们设置的编译目标是aosp_arm64-eng,因此执行指令:

--- 注意 架构 ---

不清楚可以看后面

```
lunch aosp_arm64-eng1
```

**编译目标格式说明** 
 编译目标的格式:BUILD-BUILDTYPE,比如上面的aosp_arm-eng的BUILD是aosp_arm,BUILDTYPE是eng.

**什么是BUILD**

>   BUILD指的是特定功能的组合的特定名称,即表示编译出的镜像可以运行在什么环境.其中,aosp(Android Open Source Project)代表Android开源项目;arm表示系统是运行在arm架构的处理器上,arm64则是指64位arm架构;处理器,x86则表示x86架构的处理器;此外,还有一些单词代表了特定的Nexus设备,下面是常用的设备代码和编译目标,更多参考[官方文档](https://source.android.com/source/running.html)

| 受型号   | 设备代码   | 编译目标                  |
| -------- | ---------- | ------------------------- |
| Nexus 6P | angler     | aosp_angler-userdebug     |
| Nexus 5X | bullhead   | aosp_bullhead-userdebug   |
| Nexus 6  | shamu      | aosp_shamu-userdebug      |
| Nexus 5  | hammerhead | aosp_hammerhead-userdebug |

   

提示:如果你没有Nexus设备,那么通常选择arm或者x86即可

  

**什么是BUILDTYPE**

>   BUILD TYPE则指的是编译类型,通常有三种: 
>      -user:代表这是编译出的系统镜像是可以用来正式发布到市场的版本,其权限是被限制的(如,没有root权限,不鞥年dedug等) 
>      -userdebug:在user版本的基础上开放了root权限和debug权限. 
>      -eng:代表engineer,也就是所谓的开发工程师的版本,拥有最大的权限(root等),此外还附带了许多debug工具

  

了解编译目标的组成之后,我们就可以根据自己目前的情况选择了.那不知道编译目标怎么办? 
 我们只需要执行不带参数的lunch指令,稍后,控制台会列出所有的编译目标,如下: 
 ![这里写图片描述](https://img-blog.csdn.net/20160619205826829) 
 接着我们只需要输入相应的数字即可.

  

来举个例子:你没有Nexus设备,只想编译完后运行看看,那么就可以选择aosp_arm-eng. 
 (我在ubuntu 16.04(64位)中编译完成后启动虚拟机时,卡在黑屏,尝试编译aosp_arm64-eng解决.因此,这里我使用了aosp_arm64-eng)

如果想要打包成iso文件作为虚拟机镜像，应该选择x86_64，因为虚拟机会使用宿主机的CPU，

在安卓系统中

```bash
cat /proc/cpuinfo
```

可以查看CPU信息。

使用adb依然可以正常安装应用，其他方法没有尝试。

```bash
adb.exe install APKPATH
adb push 从主机向安卓机发送文件
adb pull 从安卓机复制文件到主机
更多见adb命令帮助
```















由于android-x86是可以装载电脑上的，所以可以直接编译成ISO镜像，编译命令为 make iso_img -j4，这个4表示进程数。