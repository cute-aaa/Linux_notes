## Java连接docker



### 环境

centos7



### 简单连接

要想访问dockerAPI需要先设置一个端口：

```sh
vi /lib/systemd/system/docker.service
```

找到Execstart=/usr/bin/dockerd后加上 -H tcp://0.0.0.0:4399 -H unix://var/run/docker/sock，保存退出。

重新载入守护进程，重启docker：

```sh
systemctl daemon-reload
service docker restart
systemctl status docker		//查看相关内容，看4399端口是否已经设置好
netstat -nlp | grep 4399	//查看4399是否被监听
```

如果是阿里的云服务器，应该需要在安全组中加入4399或其他的操作。

如果是本地虚拟机，则需要把防火墙关闭或在防火墙中开启相应端口供外部使用：

**关闭防火墙：**

```shell
启动：		systemctl start firewalld
关闭：		systemctl stop firewalld
查看状态：	systemctl status firewalld
开机启用：	systemctl enable firewalld
开机禁用：	systemctl disable firewalld
```

**或**

**开启端口：**

```shell
添加端口：	firewall-cmd --zone=public --add-port=4399/tcp --permanent		(permanent为永久生效，没有这个参数则firewall重启失效)
重新载入：	firewall-cmd --reload
查询端口：	firewall-cmd --zone=public --query-port=4399/tcp
删除端口：	firewall-cmd --zone=public --remove-port-4399/tcp --permanent
```



这时就可以通过URL访问docker了。

在浏览器中输入http://xxx.xxx.xxx.xxx:4399/info，返回 json。

更多操作：<https://docs.docker.com/engine/api/v1.29/#operation/ContainerAttach>



**在Java中访问：**

在docker官方文档中有介绍：（最后有一些不是官方的）<https://docs.docker.com/develop/sdk/#unofficial-libraries>

可选择不同语言，如docker-java。

在docker-java的GitHub上官方推荐的连接方式：<https://github.com/docker-java/docker-java>

docker-java推荐使用maven项目引入相关的jar包依赖：

```xml
<dependency>
      <groupId>com.github.docker-java</groupId>
      <artifactId>docker-java</artifactId>
      <!-- use latest version https://github.com/docker-java/docker-java/releases -->
      <version>3.X.Y</version>
</dependency>
```

使用：

```java
DockerClient dockerClient = DockerClientBuilder.getInstance("tcp://xxx.xxx.xxx.xxx:2375").build();
Info info = dockerClient.infoCmd().exec();
System.out.println(info);
```

这种最简单的连接方式是有漏洞的，存在暴露端口的危害。





其他操作：

```shell
systemctl是CentOS7的服务管理工具中主要的工具，它融合之前service和chkconfig的功能于一体。
启动一个服务：systemctl start firewalld.service
关闭一个服务：systemctl stop firewalld.service
重启一个服务：systemctl restart firewalld.service
显示一个服务的状态：systemctl status firewalld.service
在开机时启用一个服务：systemctl enable firewalld.service
在开机时禁用一个服务：systemctl disable firewalld.service
查看服务是否开机启动：systemctl is-enabled firewalld.service
查看已启动的服务列表：systemctl list-unit-files|grep enabled
查看启动失败的服务列表：systemctl --failed
 
配置firewalld-cmd
 
查看版本： firewall-cmd --version
查看帮助： firewall-cmd --help
显示状态： firewall-cmd --state
查看所有打开的端口： firewall-cmd --zone=public --list-ports
更新防火墙规则： firewall-cmd --reload
查看区域信息:  firewall-cmd --get-active-zones
查看指定接口所属区域： firewall-cmd --get-zone-of-interface=eth0
拒绝所有包：firewall-cmd --panic-on
取消拒绝状态： firewall-cmd --panic-off
查看是否拒绝： firewall-cmd --query-panic
```





## 安全连接

如何在服务器上或者本地虚拟机上生成密钥文件可参考官方文档：<https://docs.docker.com/engine/security/https/#create-a-ca-server-and-client-keys-with-openssl>

<https://docs.docker.com/engine/security/https/>

--- 官方生成不了？？？ ---

首先选择一个存放密钥文件的地方 我这里选择/home/user/certs来存放 /user/certs是我自己创建的  进到certs文件中运行openssl genrsa -aes256 -out ca-key.pem 4096 

```shell
[root@ywh certs]# openssl genrsa -aes256 -out ca-key.pem 4096
Generating RSA private key, 4096 bit long modulus
.........................++
..............................................................................................................................................++
e is 65537 (0x10001)
Enter pass phrase for ca-key.pem:               //这里会让设置密码我设置的是123456 这个以后会用到
Verifying - Enter pass phrase for ca-key.pem:   //这里是让你再次输入密码确认密码不会显示出来，你只管输入按回车就可以了
[root@ywh certs]#
```

**命令集**

```shell
[root@ywh certs]$ openssl genrsa -aes256 -out ca-key.pem 4096
[root@ywh certs]$ openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
[root@ywh certs]$ openssl genrsa -out server-key.pem 4096
[root@ywh certs]$ openssl req -subj "/CN=192.168.0.3" -sha256 -new -key server-key.pem -out server.csr
[root@ywh certs]$ echo subjectAltName = DNS:192.168.0.3,IP:192.168.0.3,IP:0.0.0.0,IP:127.0.0.1 >> extfile.cnf
[root@ywh certs]$ echo extendedKeyUsage = serverAuth >> extfile.cnf
[root@ywh certs]$ openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem  -CAcreateserial -out server-cert.pem -extfile extfile.cnf
[root@ywh certs]$ openssl genrsa -out key.pem 4096
[root@ywh certs]$ openssl req -subj '/CN=client' -new -key key.pem -out client.csr
[root@ywh certs]$ echo extendedKeyUsage = clientAuth >> extfile.cnf
[root@ywh certs]$ openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile.cnf
[root@ywh certs]$ rm -v client.csr server.csr
[root@ywh certs]$ chmod -v 0400 ca-key.pem key.pem server-key.pem 
[root@ywh certs]$ chmod -v 0444 ca.pem server-cert.pem cert.pem
```

##### 现在在/home/user/certs下应该有8个文件

```shell
[root@ywh certs]$ ll
总用量 32
-r--------. 1 root root 3326 8月  30 10:44 ca-key.pem
-r--r--r--. 1 root root 2065 8月  30 10:48 ca.pem
-rw-r--r--. 1 root root   17 8月  30 11:04 ca.srl
-r--r--r--. 1 root root 1895 8月  30 11:04 cert.pem
-rw-r--r--. 1 root root  117 8月  30 11:04 extfile.cnf
-r--------. 1 root root 3243 8月  30 11:03 key.pem
-r--r--r--. 1 root root 1899 8月  30 11:02 server-cert.pem
-r--------. 1 root root 3247 8月  30 10:53 server-key.pem
```

##### 找docker.service文件 vi /lib/systemd/system/docker.service下的ExecStart添加

`-D --tlsverify=true --tlscert=/home/user/certs/server-cert.pem --tlskey=/home/user/certs/server-key.pem --tlscacert=/home/user/certs/ca.pem -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock`

```shell
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -D --tlsverify=true --tlscert=/home/user/certs/server-cert.pem --tlskey=/home/user/certs/server-key.pem --tlscacert=/home/user/certs/ca.pem -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
```

运行

```shell
systemctl daemon-reload 
service docker restart  //重启docker
systemctl status docker //这条命令可以看见你是否设置的生效
```

配置好安全连接以后在地址栏中是不可以访问的了，如果还可以访问是不对的 因为你没有使用密钥文件的方式访问

这时候需要通过密钥来认证以后才能访问了，首先把密钥文件下载到本机的磁盘上，可以使用【sz  你的文件名】命令把文件传输到本机磁盘上，如果显示没有sz命令，可以使用yum install lrzsz下载，下载完可以直接使用。

需要下载到本地的文件有：

ca-key.pem     ca.pem    cert.pem    key.pem



docker-java官方推荐连接方式为：（这种连接方式是在你对2375做了安全连接以后才能用的，如果没有做安全连接请使用我上一篇文章的方法进行连接）



导入jar

```xml
<!-- https://mvnrepository.com/artifact/com.github.docker-java/docker-java -->
<dependency>
    <groupId>com.github.docker-java</groupId>
    <artifactId>docker-java</artifactId>
    <version>3.1.2</version>
</dependency>

<!-- 这个包在JDK６-９之中自带 -->
<!-- https://mvnrepository.com/artifact/javax.activation/activation -->
<dependency>
    <groupId>javax.activation</groupId>
    <artifactId>activation</artifactId>
    <version>1.1.1</version>
</dependency>
```



```java
//进行安全认证
DockerClientConfig config = DefaultDockerClientConfig.createDefaultConfigBuilder().withDockerTlsVerify(true)
		.withDockerCertPath("F:/data/local/")
    	.withDockerHost("tcp://192.168.0.3:2375")
    	.withDockerConfig("F:/data/local/")
    	.withApiVersion("1.38")
    	.withRegistryUrl("https://index.docker.io/v1/")
		.withRegistryUsername("dockeruser")
    	.withRegistryPassword("ilovedocker")
		.withRegistryEmail("dockeruser@github.com").build();
DockerCmdExecFactory dockerCmdExecFactory =  new  JerseyDockerCmdExecFactory()
		  .withReadTimeout(1000)
		  .withConnectTimeout(1000)
		  .withMaxTotalConnections(100)
		  .withMaxPerRouteConnections(10);
//进行连接
DockerClient dockerClient = DockerClientBuilder
		.getInstance(config)
    	.withDockerCmdExecFactory(dockerCmdExecFactory).build();
Info info = dockerClient.infoCmd().exec();
System.out.println(info);

//获取镜像列表	//更多操作参照com.github.dockerjava.api.command/
//使用 dockerClient.{想执行的操作}.exec();
List<Image> images = dockerClient.listImagesCmd().exec();
System.out.println(images.toString());
```

dockerHost：地址是你docker所在的宿主机的外网地址以及你开放的端口号

dockercertPath：放入的是你密钥在windows的文件存放地址

dockerconfig：放的是什么我不太清楚，但是我放入密钥文件地址也没出错

apiVersion：dockerAPI的版本，可通过docker version命令在宿主机上获取版本号

RegistryUrl：这个按着默认的写即可

.withRegistryUsername("dockeruser")：默认

.withRegistryPassword("ilovedocker")：默认

.withRegistryEmail("dockeruser@github.com")：默认

输出：跟在地址栏访问获得的结果是一样的，都是以json格式展示

```java
com.github.dockerjava.api.model.Info@2e54db99
[architecture=x86_64,containers=1,containersStopped=1,containersPaused=0,containersRunning=0,cpuCfsPeriod=true,cpuCfsQuota=true,cpuShares=true,cpuSet=true,debug=true,discoveryBackend=<null>,dockerRootDir=/var/lib/docker,driver=overlay2,driverStatuses=[[Backing Filesystem, xfs], [Supports d_type, true], [Native Overlay Diff, true]],systemStatus=<null>,plugins={Volume=[local], Network=[bridge, host, macvlan, null, overlay], 
Authorization=null, Log=[awslogs, fluentd, gcplogs, gelf, journald, json-file, logentries, splunk, syslog]},executionDriver=<null>,
loggingDriver=json-file,
experimentalBuild=false,httpProxy=,httpsProxy=,id=MID4:FUBP:XWQB:TBNV:LPLN:IUIU:32MG:HBYM:6O2K:HBUW:257R:7DY4,ipv4Forwarding=true,bridgeNfIptables=true,
bridgeNfIp6tables=true,images=1,indexServerAddress=https://index.docker.io/v1/,initPath=<null>,initSha1=<null>,kernelVersion=3.10.0-862.el7.x86_64,labels={},
memoryLimit=true,memTotal=1910075392,name=ywh,ncpu=4,nEventsListener=0,nfd=24,nGoroutines=47,noProxy=,oomKillDisable=true,osType=linux,oomScoreAdj=<null>,
operatingSystem=CentOS Linux 7 (Core),registryConfig=com.github.dockerjava.api.model.InfoRegistryConfig@6d24ffa1[indexConfigs={docker.io=com.github.dockerjava.api.model.InfoRegistryConfig$IndexConfig@65a4798f[mirrors=[https://tpp5ie36.mirror.aliyuncs.com/],name=docker.io,official=true,secure=true]},insecureRegistryCIDRs=[127.0.0.0/8],mirrors=[https://tpp5ie36.mirror.aliyuncs.com/]],sockets=<null>,swapLimit=true,systemTime=2018-08-30T11:38:18.472151804+08:00,serverVersion=18.06.1-ce,clusterStore=,clusterAdvertise=,swarm=SwarmInfo[nodeID=,nodeAddr=,localNodeState=INACTIVE,controlAvailable=false,error=,remoteManagers=<null>,nodes=<null>,managers=<null>,clusterInfo=<null>]]
```

以上就可以在java中连接docker-api了，可以进行安全的连接了，没有密钥文件是不可能访问到的。