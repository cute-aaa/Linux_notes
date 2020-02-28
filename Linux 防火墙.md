# Linux 防火墙

## Ubuntu

1、查看端口开启状态

```shell
sudo ufw status
```

2、开启某个端口，比如我开启的是8381

```shell
sudo ufw allow 8381
```

3、开启防火墙

```shell
sudo ufw enable
```

 4、关闭防火墙

```shell
sudo ufw disable
```

 5、重启防火墙

```shell
sudo ufw reload
```

6、禁止外部某个端口比如80

```shell
sudo ufw delete allow 80
```

7、查看端口ip（如果安装了net-tools）

```shell
netstat -ltn
```



## CentOS

```shell
firewall-cmd --state
```

![img](https://img-blog.csdnimg.cn/20181112222055135.png)

 

 

停止firewall

```shell
systemctl stop firewalld.service
```

![img](https://img-blog.csdnimg.cn/20181112222243413.png)

![img](https://img-blog.csdnimg.cn/20181112222304620.png)

 

 

启动firewall

![img](https://img-blog.csdnimg.cn/20181112222105352.png)

![img](https://img-blog.csdnimg.cn/20181112222308469.png)

![img](https://img-blog.csdnimg.cn/20181112222418197.png)

 

禁止firewall开机启动

```shell
systemctl disable firewalld.service 
```



### iptables方式

**1、查看**

iptables -nvL --line-number

-L 查看当前表的所有规则，默认查看的是filter表，如果要查看NAT表，可以加上-t NAT参数
-n 不对ip地址进行反查，加上这个参数显示速度会快很多
-v 输出详细信息，包含通过该规则的数据包数量，总字节数及相应的网络接口
–-line-number 显示规则的序列号，这个参数在删除或修改规则时会用到

**或**

netstat命令各个参数说明如下：

　　-t : 指明显示TCP端口

　　-u : 指明显示UDP端口

　　-l : 仅显示监听套接字(所谓套接字就是使应用程序能够读写与收发通讯协议(protocol)与资料的程序)

　　-p : 显示进程标识符和程序名称，每一个套接字/端口都属于一个程序。

　　-n : 不进行DNS轮询，显示IP(可以加速操作)

netstat -ntlp   //查看当前所有tcp端口

netstat -ntulp |grep 80   //查看所有80端口使用情况

netstat -an | grep 3306   //查看所有3306端口使用情况

**2、添加**

添加规则有两个参数：-A和-I。其中-A是添加到规则的末尾；-I可以插入到指定位置，没有指定位置的话默认插入到规则的首部。

当前规则：

```shell
[root@test ~]# iptables -nL --line-number
Chain INPUT (policy ACCEPT)
num target   prot opt source        destination
1  DROP    all -- 192.168.1.1     0.0.0.0/0
2  DROP    all -- 192.168.1.2     0.0.0.0/0
3  DROP    all -- 192.168.1.4     0.0.0.0/0
```


添加一条规则到尾部：

```shell
[root@test ~]# iptables -A INPUT -s 192.168.1.5 -j DROP
```

再插入一条规则到第三行，将行数直接写到规则链的后面：

```shell
[root@test ~]# iptables -I INPUT 3 -s 192.168.1.3 -j DROP
```

放行172.17.79.4

```shell
iptables -I INPUT -s 172.17.79.4 -p tcp --dport 1521 -j ACCEPT
```

查看：

```shell
[root@test ~]# iptables -nL --line-number
Chain INPUT (policy ACCEPT)
num target   prot opt source        destination
1  DROP    all -- 192.168.1.1     0.0.0.0/0
2  DROP    all -- 192.168.1.2     0.0.0.0/0
3  DROP    all -- 192.168.1.3     0.0.0.0/0
4  DROP    all -- 192.168.1.4     0.0.0.0/0
5  DROP    all -- 192.168.1.5     0.0.0.0/0
```

可以看到192.168.1.3插入到第三行，而原来的第三行192.168.1.4变成了第四行。

**3、删除**

删除用-D参数

删除之前添加的规则（iptables -A INPUT -s 192.168.1.5 -j DROP）：

```shell
[root@test ~]# iptables -D INPUT -s 192.168.1.5 -j DROP
```

阻止所有服务器访问1521端口

```shell
iptables -I INPUT -p tcp --dport 1521 -j DROP
```

有时候要删除的规则太长，删除时要写一大串，既浪费时间又容易写错，这时我们可以先使用–line-number找出该条规则的行号，再通过行号删除规则。

```shell
[root@test ~]# iptables -nv --line-number
iptables v1.4.7: no command specified
Try `iptables -h' or 'iptables --help' for more information.
[root@test ~]# iptables -nL --line-number
Chain INPUT (policy ACCEPT)
num target   prot opt source        destination
1  DROP    all -- 192.168.1.1     0.0.0.0/0
2  DROP    all -- 192.168.1.2     0.0.0.0/0
3  DROP    all -- 192.168.1.3     0.0.0.0/0
```

删除第二行规则

```shell
[root@test ~]# iptables -D INPUT 2
```

**4、修改**

修改使用-R参数

先看下当前规则：

```shell
[root@test ~]# iptables -nL --line-number
Chain INPUT (policy ACCEPT)
num target   prot opt source        destination
1  DROP    all -- 192.168.1.1     0.0.0.0/0
2  DROP    all -- 192.168.1.2     0.0.0.0/0
3  DROP    all -- 192.168.1.5     0.0.0.0/0
```

将第三条规则改为ACCEPT：

```bash
[root@test ~]# iptables -R INPUT 3 -j ACCEPT
```

再查看下：

```shell
[root@test ~]# iptables -nL --line-number
Chain INPUT (policy ACCEPT)
num target   prot opt source        destination
1  DROP    all -- 192.168.1.1     0.0.0.0/0
2  DROP    all -- 192.168.1.2     0.0.0.0/0
3  ACCEPT   all -- 0.0.0.0/0      0.0.0.0/0
```

第三条规则的target已改为ACCEPT。

**5、永久生效**

```shell
service iptables save
service iptables restart
```

