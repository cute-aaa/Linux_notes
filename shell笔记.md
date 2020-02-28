vi hello.sh

#! /bin/bash	申明该脚本由bash来解释，便于移植

echo "HelloWorld"



执行：

sh hello.sh

./hello.sh

任意处执行：

改变权限并加入到.bash_profile中，添加到搜索路径，也就是环境变量

path = $path:

source .bash_profile立即生效



#### shell变量

- 系统变量：如$#命令行参数的个数$0 $1
  - env或export来查看
  - hostname主机名
  - shell默认值
  - term信息
  - 历史命令大小
  - mail邮箱
  - path查找路径
- 环境变量
- 用户变量

编写脚本：

#! bin/bash

echo "my hostname is $HOSTNAME"

echo "username is $USER"

echo "uuid is EUID"

echo "user's home is $HOME";

##### 用户变量：

shell中可以将任意非空字符串作为一个用户变量

不预先声明就能用等号对用户变量进行赋值

如：变量名 = 字符串或整型数字

等号前后不能有空格。

如果要将带空格的字符串赋值，应该用单引号括起来

读取：

read 变量名

输入回车结束

read name

echo $name;

读取：

read 提示：

read -p "enter your name " name1, name2

用户变量 = \`命令\`

current_time2=$(date)

系统变量之位置变量

位置变量用于存放传递给命令行上shell程序或shell脚本的参数。

hello.sh a b c d传递参数

echo ${$0}

echo {[${0} ${1}, ${2}};



shift	改变参数位置	shift	无参数，执行之后所有参数左移，最左边删除最右边空

echo $#	显示参数个数

?变量存放shell程序最后一条命令的返回码	echo $?

*或@	是啥来着，	echo $@