# mysql安装、卸载、配置、远程连接

### 卸载

首先用dpkg --list|grep mysql查看自己的mysql有哪些依赖

![img](https://img-blog.csdn.net/20170817163736180?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdzMwNDU4NzI4MTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



先卸载sudo apt-get remove mysql-common

然后：sudo apt-get autoremove --purge mysql-server-5.0 

再用dpkg --list|grep mysql查看，还剩什么就卸载什么

最后清楚残留数据：

dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P



就可以了





### 安装

```shell
apt-get install mysql-server mysql-client
或
yum -y install mysql-server mysql-client
```



### 配置

#### 密码

默认没有密码（有人说安装时会提示输入密码？）

使用Linux的root用户登录

```shell
root@sure# mysql
```

如果不能直接登录：

1.首先输入以下指令：

```
sudo cat /etc/mysql/debian.cnf
```

运行截图如下：

![img](https://img-blog.csdn.net/20180831144759364?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM3OTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

\2. 再输入以下指令：

```
mysql -u debian-sys-maint -p
//注意! 
//这条指令的密码输入是输入第一条指令获得的信息中的 password = ZCt7QB7d8O3rFKQZ 得来。
//请根据自己的实际情况填写！
```

运行截图如下：**(注意! 这步的密码输入的是 ZCt7QB7d8O3rFKQZ，密码是由第一条指令获得的信息中的**

**password = ZCt7QB7d8O3rFKQZ 得来，每个人不一样，请根据自己的实际情况输入，输入就可以得到以下运行情况）**

![img](https://img-blog.csdn.net/20180717234206251?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4NzM3OTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

然后就可以继续了

```mysql
use mysql;
// 下面这句命令有点长，请注意。
update mysql.user set authentication_string=password('root') where user='root' and Host ='localhost';
//这句的作用？？？
//作用是执行之后，再刷新重启mysql之后，就不能通过直接输入mysql式的无密码进入了
update user set plugin="mysql_native_password"; 
flush privileges;
quit;
```

重新启动mysql

```shell
service mysql restart
```

现在需要输入密码了

```shell
mysql -u root -p
Password:
```



### 远程连接

```shell
# 注意：不同 mysql 版本此配置文件位置和名字可能不同
# mariadb的配置文件是/etc/mysql/mariadb.conf.d/*.cnf

# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
# 从上面看出msyql5.0以前应该注释 skip-networking 这一行。
sudo vim /etc/mysql/my.cnf
#找到将bind-address = 127.0.0.1注销
#bind-address            = 127.0.0.1
```

**3.增加允许远程访问的用户或者允许现有用户的远程访问。**

接着上面，删除匿名用户后，给root授予在任意主机（%）访问任意数据库的所有权限。SQL语句如下：

代码如下:

```mysql
mysql> grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
mysql> flush privileges;
```

如果需要指定访问主机，可以把%替换为主机的IP或者主机名。另外，这种方法会在数据库mysql的表user中，增加一条记录。如果不想增加记录，只是想把某个已存在的用户（例如root）修改成允许远程主机访问，则可以使用如下SQL来完成：

代码如下:

```mysql
update user set host='%' where user='root' and host='localhost';
```

重启mysql服务

```shell
service mysql restart
```

检查MySQL服务器占用端口 `netstat -nlt|grep 3306`

```shell
netstat -nlt|grep 3306
  tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN
```

检查MySQL服务器系统进程 `ps -aux|grep mysql`



### 编码配置

默认情况下，MySQL的字符集是latin1，因此在存储中文的时候，会出现乱码的情况，所以我们需要把字符集统一改成UTF-8。
打开mysql配置文件（自己找是哪个文件吧）

（本人改的时候是/etc/mysql/mysql.conf.d/mysqld.cnf）
`sudo vim /etc/mysql/my.cnf`

```
a） 打开mysql配置文件：

                vim/etc/mysql/my.cnf
b） 在[client]下追加：（如果没有client就自己添加）

                default-character-set=utf8
c） 在[mysqld]下追加：

                character-set-server=utf8
                collation-server=utf8_general_ci
d） 在[mysql]下追加：

                default-character-set=utf8
```

修改后，**重启MySQL服务器**,并登录
`mysql -uroot -p`

再次查看字符串编码

```mysql
mysql> show variables like '%char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```