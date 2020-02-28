# 远程连接docker上的SQL_Server

###### 参考：

官方文档<https://docs.microsoft.com/zh-cn/sql/linux/quickstart-install-connect-docker?view=sql-server-2017&pivots=cs1-bash>

个人博客<https://blog.csdn.net/Linjingke32/article/details/78154406>



## 下载容器

使用管理员身份运行：

```sh
sudo docker pull mcr.microsoft.com/mssql/server:2017-latest
```

前一个命令请求最新的 SQL Server 2017 容器映像。 如果想请求某个特定映像，需添加一个冒号和标记名称（例如 `mcr.microsoft.com/mssql/server:2017-GA-ubuntu`。 若要查看所有可用映像，请参阅[mssql server Docker 中心页](https://hub.docker.com/r/microsoft/mssql-server)。



## 运行镜像（坑）

运行docker容器

```sh
sudo docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=<YourStrong!Passw0rd>' \
   -p 1433:1433 --name sql1 \
   -d mcr.microsoft.com/mssql/server:2017-latest
```

<u>**这里的密码必须是<YourStrong!Passw0rd>，不能改成自己的密码！不然容器启动之后就会立刻退出！！**</u>



| 参数                                           | Description                                                  |
| :--------------------------------------------- | :----------------------------------------------------------- |
| **-e 'ACCEPT_EULA=Y'**                         | 将 **ACCEPT_EULA**变量设置为任意值，以确认接受[最终用户许可协议](https://go.microsoft.com/fwlink/?LinkId=746388)。SQL Server 映像的必需设置。 |
| **-e 'SA_PASSWORD=<YourStrong!Passw0rd>'**     | 指定至少包含 8 个字符且符合 [SQL Server 密码要求](https://docs.microsoft.com/zh-cn/sql/relational-databases/security/password-policy?view=sql-server-2017)的强密码。 SQL Server 映像的必需设置。 |
| **-p 1433:1433**                               | 建立主机环境（第一个值）上的 TCP 端口与容器（第二个值）中 TCP 端口的映射。 在此示例中，SQL Server 侦听容器中的 TCP 1433 并公开的端口 1433，在主机上。 |
| **--name sql1**                                | 为容器指定一个自定义名称，而不是使用随机生成的名称。 如果运行多个容器，则无法重复使用相同的名称。 |
| **mcr.microsoft.com/mssql/server:2017-latest** | SQL Server 2017 Linux 容器映像。                             |

要查看 Docker 容器，请使用 `docker ps` 命令。

bash复制

```bash
sudo docker ps -a
```

1.  如果“状态”列显示“正常运行”，则 SQL Server 将在容器中运行，并侦听“端口”列中指定的端口。 如果 SQL Server 容器的“状态”列显示“已退出”，则参阅[配置指南的疑难解答部分](https://docs.microsoft.com/zh-cn/sql/linux/sql-server-linux-configure-docker?view=sql-server-2017#troubleshooting)。

`-h`（主机名）参数也非常有用，但为了简单起见，本教程中不使用它。 这会将容器的内部名称更改为一个自定义值。 也就是以下 Transact-SQL 查询中返回的名称：

SQL复制

```sql
SELECT @@SERVERNAME,
    SERVERPROPERTY('ComputerNamePhysicalNetBIOS'),
    SERVERPROPERTY('MachineName'),
    SERVERPROPERTY('ServerName')
```

将 `-h` 和 `--name` 设为相同的值是一种很好的方法，可以轻松地识别目标容器。



## 更改密码

SA 帐户是安装过程中在 SQL Server 实例上创建的系统管理员。 创建 SQL Server 容器后，通过在容器中运行 `echo $MSSQL_SA_PASSWORD`，可发现指定的 `MSSQL_SA_PASSWORD` 环境变量。 出于安全考虑，请考虑更改 SA 密码。

1.  选择 SA 用户要使用的强密码。

2.  使用 `docker exec` 运行sqlcmd（也可以直接进入容器终端修改），以使用 Transact-SQL 更改密码。 在以下示例中，替换的旧密码`<YourStrong!Passw0rd>`，和新密码， `<YourNewStrong!Passw0rd>`，使用你自己的密码值。

    ```bash
    sudo docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd \
       -S localhost -U SA -P '<YourStrong!Passw0rd>' \
       -Q 'ALTER LOGIN SA WITH PASSWORD="<YourNewStrong!Passw0rd>"'
    ```

密码应符合 SQL Server 默认密码策略，否则容器无法设置 SQL Server，将停止工作。 默认情况下，密码必须至少为 8 个字符长，且包含三个以下四种字符集的字符：大写字母、 小写字母、 十进制数字和符号。 你可以通过执行 [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) 命令检查错误日志。



## 本地连接

SQL Server默认用户：SA



下列步骤在容器内部使用 SQL Server 命令行工具 **sqlcmd** 来连接 SQL Server。

1.  使用 `docker exec -it` 命令在运行的容器内部启动交互式 Bash Shell。 在下面的示例中，`sql1` 是在创建容器时由 `--name` 参数指定的名称。

    bash复制

    ```bash
    sudo docker exec -it sql1 "bash"
    ```

2.  一旦位于容器内部，使用 sqlcmd 进行本地连接。 默认情况下，sqlcmd 不在路径之中，因此需要指定完整路径。

    bash复制

    ```bash
    /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P '<YourNewStrong!Passw0rd>'
    ```

     提示

    可以省略命令行上提示要输入的密码。

3.  如果成功，应会显示 sqlcmd 命令提示符：`1>`。



## 远程连接

这里使用 **Navicat** 进行连接

所需环境：Microsoft SQL Server 20** Native Client

![1561277265041](C:\Users\I\AppData\Roaming\Typora\typora-user-images\1561277265041.png)

这个软件可以在navicat安装目录下找到

![1561277369146](C:\Users\I\AppData\Roaming\Typora\typora-user-images\1561277369146.png)

连接时会提示安装，如果没有就手动安装一下

默认数据库有这几个：

master                                                                                                                          
tempdb                                                                                                                          
model                                                                                                                           
msdb   

一般使用master数据库

用户名sa

密码就是刚才自己设的密码

![1561277526348](C:\Users\I\AppData\Roaming\Typora\typora-user-images\1561277526348.png)