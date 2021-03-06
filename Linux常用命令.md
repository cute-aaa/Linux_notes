## 命令行快捷键

向前删除单词：`ctrl + w`

向后删除单词：`alt + d`

删到行首：`ctrl + u`

删到行尾：`ctrl + k`

左移一个单词：`alt + b`

右移一个单词：`alt + f`

移到行首：`ctrl + a`

移到行尾：`ctrl + e`

粘贴：`ctrl + y`



搜索命令历史：`ctrl + r`

退出命令搜索：`ctrl + g`

执行上条命令：`!!`

执行最近的xx开头的命令：`!xx`

只打印输出而不执行：`!xx:p`

上一条命令的所有参数：`!*`

上条命令的第一个、最后一个、第n个参数（第0个表示命令本身）：`!^` `!$` `!:n`

输出 `!*` 的所有参数：`!*:p`

删除上一条命令中的xx：`^xx`

将上条命令中的xx替换为yy：`^xx^yy`

将上条命令中的xx全部替换为yy：`^xx^yy^`



本条命令的所有参数：`!#` （https://www.zhihu.com/question/304040790）

本条命令的第一个、最后一个、第n个参数：`!#^` `!#$` `!#:n`



## 网络

查询端口（Ubuntu）：`lsof -i:端口`  `netstat -tunlp |grep 端口`



## Vim

### 复制粘贴

选中：按一下 `v` 然后移动光标，y复制

复制一个单词：`yw`

复制N个单词：`Nyw`

复制一行：`yy`  

复制N行：`Nyy`  

复制到行首（不含当前字符）：`y^`  `y0`

复制到行尾（含当前字符）：`y$`

复制到文档首：`yG`

复制到文档尾：`y1G`

粘贴至游标后：`p`  

粘贴至游标后：`P`  



### 插入删除

光标前插入：`i`

所在行首插入：`I`

所在行尾插入：`A`

上方插入一行：`o`

下方插入一行：`O`

替换模式：`R`（esc退出）

删除所在单词：`daw`

删除行：`dd`

删除N行：`Ndd`

删至行首：`d0`

删至行尾：`d$`



### 查找替换

搜索： `/字符串`  

替换一个：`/搜索/s/替换`

替换全部：`/搜索/s/替换/g`



### 所处位置

文件首部： ` :1 `  `gg` 

文件尾部： `:$`  `shift+g` 

到第N行：`Ngg`

上翻半屏：`ctrl + u`

下翻半屏：`ctrl + d`

上翻一屏：`ctrl + f`

下翻一屏：`ctrl + b`



### 撤销与重做

撤销一次：`u`

撤销某行最近所有修改：`U`

重做：`ctrl + r`

