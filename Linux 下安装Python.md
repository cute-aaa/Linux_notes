## Linux 下安装Python



#### 错误1

```shell
make: *** [install] 错误 1

zipimport.ZipImportError: can't decompress data; zlib not available
```

解决：安装 `zlib` 系列包，只有zlib不行

```shell
yum -y install zlib*
```

进入 python安装包,修改Module路径的Setup文件（有人说python2.7.14是Modules/Setup.dist）：

```shell
vim Module/Setup 
```


找到一下一行代码，去掉注释：

```shell
#zlib zlibmodule.c -I$(prefix)/include -L$(exec_prefix)/lib -lz
```

去掉注释

```shell
zlib zlibmodule.c -I$(prefix)/include -L$(exec_prefix)/lib -lz
```

 另外，在这里说明一下，对于在安装Python安装的过程中遇到这个问题，安装完上面的依赖包后，即可重新进入终端，进入python的安装包路径下执行：

```shell
make && make install 
```

重新编译安装即可，

重新编译安装即可，



#### 错误2

```shell
ModuleNotFoundError: No module named '_ctypes'
```

安装 `libffi-devel`

```shell
yum install libffi-devel
```





### 完整步骤

1.  从<https://www.python.org>下载Python
2.  解压文件
3.  进入解压的文件夹

>   ```shell
>   sudo yum -y install gcc gcc-c++ 
>   sudo yum -y install zlib zlib-devel
>   sudo yum -y install libffi-devel 
>   ./configure --prefix=/usr/lib/Python-3.7.3  # --prefix指定安装目录
>   make
>   make install
>   ```



**后续步骤**

将python添加到命令：

```shell
# 为python的执行文件创建软链接
ln -s /usr/lib/Python-3.7.3/bin/python3.7 /usr/bin/python3
# 不能创建python命令，那个是系统自带的python，yum要用

# 将pip添加到命令
ln -s /usr/lib/Python-3.7.3/bin/pip3.7 /usr/bin/pip3
```

**或**

添加环境变量

```shell
vim ~/.bash_profile

PYTHONHOME=/usr/lib/Python-3.7.3
PATH=$PATH:$PYTHONHOME/bin

source ~/.bash_profile
```

