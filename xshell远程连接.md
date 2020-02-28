ifconfig ens33设置对应IP

【网卡设备名】IP地址

一次有效，重启失效

永久需修改配置文件

/etc/sysconfig/network-scripts/ifcfg-eth33网卡配置文件

ONBOOT开机自动启用，一般YES

BOOTPROTO：DHCP static

IPADDR=地址

systemctl restart network重启服务

。。。status。。。查看状态





/etc/resolv.conf	DNS配置文件



/etc/hosts

格式：IP主机名 别名

先读这个文件，没有再找DNS



mail abcs

subject: abctest

EOT

cat /var/spool/main/sure

ping -c 6 www.baidu.com发送6个数据包

hostname [要设置的主机名]

hostname显示主机名





SSH远程连接

rpm -qa|grep openssh

systemctl status sshd

ifconfig

## ROOT远程登录

如果你需要root用户远程登录（大多数情况下普通用户就够了，如果是传文件，可以先传到其他目录，在使用root修改过去。

所需工具：openssh-server	(可以使用apt安装)

1.  root登录你的Linux

2.  vim /etc/ssh/sshd_config编辑这个配置文件如下:

     [![001](https://images0.cnblogs.com/blog/635602/201505/231014529215923.png)](http://images0.cnblogs.com/blog/635602/201505/231014519376521.png)

3.  因为root设置了密码，所以还要修改如下的内容，如果已经no了，那么就不用修改了：[![002](https://images0.cnblogs.com/blog/635602/201505/231014545774696.png)](http://images0.cnblogs.com/blog/635602/201505/231014536243538.png)

4.  重启SSH服务:

    [![003](https://images0.cnblogs.com/blog/635602/201505/231014559521683.png)](http://images0.cnblogs.com/blog/635602/201505/231014552802311.png)





### 2019.4.28更新

**Ubuntu18.04固定IP设置**

在u18中，更改了网卡配置文件之前的版本网卡配置信息配置在**/etc/network/interfaces**文件，可以如下配置，

```
auto ens33
iface ens33 inet static
address 192.168.0.111
netmask 255.255.255.0
gateway 192.168.0.1
```

在18.04上也是可以用的，只是要重启才能生效。通过**service networking restart**无效。

下面介绍一下在18.04上新采用的**netplan**命令。网卡信息配置在**/etc/netplan/01-network-manager-all.yaml**文件，需做如下配置，

```yaml
# Let NetworkManager manage all devices on this system
network:
  version: 2
  # renderer: NetworkManager
  ethernets:
          ens33:
                  addresses: [192.168.0.121/24]
                  # 后面的24是子网掩码，256=2^8,3*8=24,也就是255.255.255.0
                  gateway4: 192.168.0.1
                  nameservers:
                        addresses: [114.114.114.114, 8.8.8.8]
```

然后使用以下命令使配置即时生效，

```sh
netplan apply
```

以上操作均在root用户下进行，如在普通用户，请自行加上**sudo**。

这里有几点需要注意：
1、将renderer: NetworkManager注释，否则netplan命令无法生效；
2、ip配置信息要按如上格式，使用yaml语法格式，每个配置项使用空格缩进表示层级；
3、对应配置项后跟着冒号，之后要接个空格，否则netplan命令也会报错。

---

ubuntu默认不带ssh，不过安装系统时会提示是否安装ssh，也可以安装完成后再单独装。