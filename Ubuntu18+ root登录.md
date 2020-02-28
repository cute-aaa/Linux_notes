# Ubuntu18+ root登录

```sh
sed -i 's/auth\trequired\tpam_succeed_if.so user != root quiet_success/# &/g' /etc/pam.d/gdm-autologin /etc/pam.d/gdm-password
sed -i 's/mesg n || true/tty -s \&\& &/' /root/.profile
sed -i '33i PermitRootLogin yes' /etc/ssh/sshd_config
systemctl restart ssh

# 修改启动级别
systemctl set-default runlevel3.target
```

1.进入/etc/pam.d目录

  

![img](http://i2.51cto.com/images/blog/201806/18/f3a39fb45e07f7a5da5de1625664cdd8.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

  

\2. 进入/etc/pam.d目录下，编辑#vim gdm-autologin和 #vim gdm-password **（注意是两个文件）**屏蔽掉
       \#auth   required        pam_succeed_if.so user != root quiet_success

  

![img](http://i2.51cto.com/images/blog/201806/18/9e5b8e33cad0e6c8b31f038518881683.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

  

\3. 使用#passwd root，添加root，并设置root的登录密码2次，我们使用简单的1234567密码，见图-41。
 然后编辑/root/.profile文件，#vim /root/.profile

  

![img](http://i2.51cto.com/images/blog/201806/18/02173d237ca8072dfe985c8c701d83fe.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

  

按图 红框去修改，先屏蔽mesg n || true，然后后面添加：
 tty –s && mesg n || true

  

4.重启系统 reboot



## ROOT远程登录

如果你需要root用户远程登录（大多数情况下普通用户就够了，如果是传文件，可以先传到其他目录，在使用root修改过去。

所需工具：openssh-server	(可以使用apt安装)

1.  root登录你的Linux

2.  vim /etc/ssh/sshd_config编辑这个配置文件如下:

     [![001](https://images0.cnblogs.com/blog/635602/201505/231014529215923.png)](http://images0.cnblogs.com/blog/635602/201505/231014519376521.png)

3.  因为root设置了密码，所以还要修改如下的内容，如果已经no了，那么就不用修改了：[![002](https://images0.cnblogs.com/blog/635602/201505/231014545774696.png)](http://images0.cnblogs.com/blog/635602/201505/231014536243538.png)

4.  重启SSH服务:

    [![003](https://images0.cnblogs.com/blog/635602/201505/231014559521683.png)](http://images0.cnblogs.com/blog/635602/201505/231014552802311.png)





## 修改启动级别

参考https://blog.csdn.net/weixin_42776979/article/details/81512370

一般很少用到图形界面，可以设置默认启动命令行

```sh
sudo systemctl set-default multi-user.target
或者
sudo systemctl set-default runlevel3.target
```

然后重启就行了

补充：

- Runlevel 0 : 关闭系统
- Runlevel 1 : 维护模式
- Runlevel 3 : 多用户，无图形系统
- Runlevel 4 : 多用户，无图形系统
- Runlevel 5 : 多用户，图形化系统
- Runlevel 6 : 关闭并重启机器

切换快捷键（不管用？）

Ctrl+Alt+F5  文本模式   

Ctrl+Alt+F1  图形模式

可用：执行startx进入，注销退出（或其他桌面）

其他问题：

1、直接切换

![img](https://img-blog.csdn.net/20180808173851270?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mjc3Njk3OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2、使用 sudo vi /etc/default/grub (在18.04中已失效)

将GRUB_CMDLINE_LINUX_DEFAULT=”quiet”
更改为
GRUB_CMDLINE_LINUX_DEFAULT=”text”
保存并退出，

然后运行下sudo update-grub2

下面是 Ubuntu 18.04

![img](https://img-blog.csdn.net/20180808174134481?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mjc3Njk3OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)