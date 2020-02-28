# openssl加密解密



### 常用加密方式

1.  对称加密：发送方和接受方使用同样的一把私钥，私钥用于加密和解密
2.  非对称加密：有一把公钥，有一把私钥，使用公钥加密，只能使用私钥进行解密



但非对称加密比对称加密慢，有时会慢一两个数量级，所以常用方式是：

使用使用对称加密对非对称加密的公私钥进行加密



### 1. 什么是OpenSSL

OpenSSL 是一个安全[套接字](https://link.jianshu.com/?t=https://baike.baidu.com/item/%E5%A5%97%E6%8E%A5%E5%AD%97)层密码库，囊括主要的[密码算法](https://link.jianshu.com/?t=https://baike.baidu.com/item/%E5%AF%86%E7%A0%81%E7%AE%97%E6%B3%95)、常用的[密钥](https://link.jianshu.com/?t=https://baike.baidu.com/item/%E5%AF%86%E9%92%A5)和证书封装管理功能及[SSL](https://link.jianshu.com/?t=https://baike.baidu.com/item/SSL)协议，并提供丰富的应用程序供测试或其它目的使用。

### 2. 基本功能

openssl是一个开源程序的套件、这个套件有三个部分组成：一是libcryto，这是一个具有通用功能的加密库，里面实现了众多的加密库；二是libssl，这个是实现ssl机制的，它是用于实现TLS/SSL的功能；三是openssl，是个多功能命令行工具，它可以实现加密解密，甚至还可以当CA来用，可以让你创建证书、吊销证书。

为了做个更简单的区分，我分成下面3种，可能你看着会亲切一些

-   加解密库（本章只讨论加解密）
-   SSL协议实现
-   一些经过封装，方便你使用加解密和SSL的工具

### 密钥、证书的编码格式和后缀名

>   目前有以下`两种编码`格式.

-   **PEM - Privacy Enhanced Mail**
    打开看文本格式,以"-----BEGIN-----"开头, "-----END-----"结尾,内容是Base64编码，查看PEM格式的信息可以用命令
    `openssl rsa -in my.pem -text -noout`
    Unix服务器偏向于使用这种编码格式.
-   **DER - Distinguished Encoding Rules**
    打开看是二进制格式,不可读，查看DER格式的信息可以用命令
    `openssl rsa -in my.der -inform der -text -noout`
    Java和Windows服务器偏向于使用这种编码格式.

>   我们平时见到的多种后缀名，都是`语义化`的后缀，在生成密钥的时候，比如我们私钥的后缀名可以写`.pem` `.key`，都是可以的，以下几种为常用后缀

-   CRT - CRT应该是certificate的三个字母,其实还是证书的意思,常见于Unix系统,有可能是PEM编码,也有可能是DER编码,大多数应该是PEM编码,相信你已经知道怎么辨别.
-   CER - 还是certificate,还是证书,常见于Windows系统,同样的,可能是PEM编码,也可能是DER编码,大多数应该是DER编码.
-   KEY - 通常用来存放一个公钥或者私钥,并非X.509证书,编码同样的,可能是PEM,也可能是DER.
    查看KEY的办法:
    `openssl rsa -in mykey.key -text -noout`
    如果是DER格式的话,同理应该这样了:
    `openssl rsa -in mykey.key -text -noout -inform der`
-   CSR - Certificate Signing Request,即证书签名请求,这个并不是证书,而是向权威证书颁发机构获得签名证书的申请,其核心内容是一个公钥(当然还附带了一些别的信息),在生成这个申请的时候,同时也会生成一个私钥,私钥要自己保管好，查看的办法:
    `openssl req -noout -text -in my.csr`
    (如果是DER格式的话照旧加上`-inform der`,这里不写了)

### 3. 常用加解密算法使用

我们重点讨论如何使用

>   **1.对称加密**

我们需要用到 `openssl enc` 命令，先看下帮助文档

```shell
$ openssl enc -h 

:<<!
-in <file>     输入文件
-out <file>    输出文件
-pass <arg>    密码
-S             盐，用于加盐加密，请避免人为输入，下面讨论
-e             encrypt 加密操作
-d             decrypt 解密操作
-a/-base64     base64 encode/decode, depending on encryption flag  是否将结果base64编码
-k             已被-pass参数取代
-kfile         已被-pass参数取代
-md            指定密钥生成的摘要算法 默认MD5
-K/-iv         加密所需的key和iv向量，由输入的-pass生成
-[pP]          print the iv/key (then exit if -P)  是否需要在控制台输出生成的 key和iv向量
-bufsize <n>   读写文件的I/O缓存，一般不需要指定
-engine e      指定三方加密设备，没有环境，暂不实验

Cipher Types  以下是部分算法，我们可以选择用哪种算法加密
-aes-128-cbc               -aes-128-cbc-hmac-sha1     -aes-128-cfb              
-aes-128-cfb1              -aes-128-cfb8              -aes-128-ctr              
-aes-128-ecb               -aes-128-gcm               -aes-128-ofb      
…………
!
```

使用，默认从控制台输入密码，如果不指定加密算法，是不会进行加密的，也不会报错，比如我们不指定算法，只指定base64格式输出，就相当于只做了base64编码而已

```shell
/*对文件进行base64编码*/
openssl enc -base64 -in plain.txt -out base64.txt
/*对base64格式文件进行解密*/
openssl enc -base64 -d -in base64.txt -out plain2.txt
/*使用diff命令查看可知解码前后明文一样*/
diff plain.txt plain2.txt
```

----------------------------------------

### 实验

分别在使用和不使用密码加密文档的情况下，对文档进行加密

使用密码解密非加密文档

不使用密码解密加密文档

```shell
[sure@localhost 文档]$ openssl enc -base64 -in hello.c -out basehello.c -pass file:password.txt 
[sure@localhost 文档]$ openssl enc -base64 -in hello.c -out basehello2.c

[sure@localhost 文档]$ openssl enc -base64 -d -in basehello.c -out hello2.c -pass file:password.txt 
[sure@localhost 文档]$ openssl enc -base64 -d -in basehello2.c -out hello3.c -pass file:password.txt 
[sure@localhost 文档]$ openssl enc -base64 -d -in basehello.c -out hello4.c
[sure@localhost 文档]$ cat hello*
#include <stdio.h>
int main(){
	char h = 'h';
	char e = 'e';
	char l = 'l';
	char o = 'o';
	printf("%c%c%c%c%c,", h, e, l, l, o);
	return 0;
}
#include <stdio.h>
int main(){
	char h = 'h';
	char e = 'e';
	char l = 'l';
	char o = 'o';
	printf("%c%c%c%c%c,", h, e, l, l, o);
	return 0;
}
#include <stdio.h>
int main(){
	char h = 'h';
	char e = 'e';
	char l = 'l';
	char o = 'o';
	printf("%c%c%c%c%c,", h, e, l, l, o);
	return 0;
}
#include <stdio.h>
int main(){
	char h = 'h';
	char e = 'e';
	char l = 'l';
	char o = 'o';
	printf("%c%c%c%c%c,", h, e, l, l, o);
	return 0;
}
[sure@localhost 文档]$ 
```

结论：base64不具备文档加密功能。

------------------------



不同输入密码的方式

```shell
/*命令行输入，密码123456*/
openssl enc -aes-128-cbc -in plain.txt -out out.txt -pass pass:123456
/*文件输入，密码123456*/
echo 123456 > passwd.txt
openssl enc -aes-128-cbc -in plain.txt -out out.txt -pass file:passwd.txt
/*环境变量输入，密码123456*/
passwd=123456
export passwd
openssl enc -aes-128-cbc -in plain.txt -out out.txt -pass env:passwd
/*从文件描述输入*/ 
openssl enc -aes-128-cbc -in plain.txt -out out.txt -pass fd:1  
/*从标准输入输入*/ 
openssl enc -aes-128-cbc -in plain.txt -out out.txt -pass stdin 
```

-----------------------------

### 实验

不输入密码：

```shell
[sure@localhost 文档]$ openssl enc -aes-128-cbc -in hello.c -out aeshello.c
enter aes-128-cbc encryption password:
Verifying - enter aes-128-cbc encryption password:
[sure@localhost 文档]$ openssl enc -aes-128-cbc -d -in aeshello.c -out hello2.c
enter aes-128-cbc decryption password:
```

结论：后面也会提示输入密码



使用C语言的helloworld来加密：

```powershell
[sure@localhost 文档]$ openssl enc -aes-128-cbc -in hello.c -out cbchello.c -pass file:password.c 
[sure@localhost 文档]$ openssl enc -aes-128-cbc -d -in cbchello.c -out hello2.c -pass file:password.c 
```

使用 `-p` 来打印盐值，key、iv

```shell
[sure@localhost 文档]$ openssl enc -aes-128-cbc -in hello.c -out cbchello.c -pass pass:123 -p
salt=0A68125B42F565F4
key=1D7F1D7527D570BA9BBC6B9F93774DDE
iv =E4FD51A03AF60603C7BE0FE473DF8E38
[sure@localhost 文档]$ openssl enc -aes-128-cbc -d -in cbchello.c -out hello2.c -pass pass:123 -p
salt=0A68125B42F565F4
key=1D7F1D7527D570BA9BBC6B9F93774DDE
iv =E4FD51A03AF60603C7BE0FE473DF8E38
```

注意！！！！！！！！！！！！！

如果想要加密文件的同时打印盐值，则只能用 `-p` ，不能用 `-P` ，因为 -P 是打印盐值，然后退出。

```shell
-[pP]          print the iv/key (then exit if -P)
```



---



### 生成公私钥

使用 `openssl genrsa` 命令来生成私钥

使用 `openssl rsa` 命令来生成公钥



帮助文档：

```shell
$ openssl genrsa -h       

/*
usage: genrsa [args] [numbits]

 -des           生成的私钥采用DES算法加密
 -des3          生成的私钥采用DES3算法加密 (168 bit key)
 -seed          encrypt PEM output with cbc seed
 -aes128, -aes192, -aes256
                
以上几个都是对称加密算法的指定，因为我们长期会把私钥加密，避免明文存放

 -out file       私钥输出位置
 -passout arg    输出文件的密码，如果我们指定了对称加密算法，也可以不带此参数，会有命令行提示你输入密码
*/
```

```shell
$ openssl rsa -h

/*
 -inform arg     输入文件编码格式，只有pem和der两种
 -outform arg    输出文件编码格式，只有pem和der两种
 -in arg         input file  输入文件
 -sgckey         Use IIS SGC key format
 -passin arg     如果输入文件被对称加密过，需要指定输入文件的密码
 -out arg        输出文件位置
 -passout arg    如果输出文件也需要被对称加密，需要指定输出文件的密码

 -des            对输出结果采用对称加密 des算法
 -des3           对输出结果采用对称加密 des3算法
 -seed           
 -aes128, -aes192, -aes256

 以上几个都是对称加密算法的指定，生成私钥的时候一般会用到，我们不让私钥明文保存

 -text           以明文形式输出各个参数值
 -noout          不输出密钥到任何文件
 -modulus        输出模数值
 -check          检查输入密钥的正确性和一致性
 -pubin          指定输入文件是公钥
 -pubout         指定输出文件是公钥
 -engine e       指定三方加密库或者硬件
*/
```

>   openssl rsa 命令的功能还有很多

1.  rsa 添加 和 去除 密钥的对称加密

```shell
/*生成不加密的RSA密钥*/
$ openssl genrsa -out RSA.pem

Generating RSA private key, 512 bit long modulus
..............++++++++++++
.....++++++++++++
e is 65537 (0x10001)

/*为RSA密钥增加口令保护*/
$ openssl rsa -in RSA.pem -des3 -passout pass:123456 -out E_RSA.pem

/*为RSA密钥去除口令保护*/
$ openssl rsa -in E_RSA.pem -passin pass:123456 -out P_RSA.pem

/*比较原始后的RSA密钥和去除口令后的RSA密钥，是一样*/
$ diff RSA.pem P_RSA.pem
```

2、修改密钥的保护口令和算法

```shell
/*生成RSA密钥*/
$ openssl genrsa -des3 -passout pass:123456 -out RSA.pem

Generating RSA private key, 512 bit long modulus
..................++++++++++++
......................++++++++++++
e is 65537 (0x10001)

/*修改加密算法为aes128，口令是123456*/

$ openssl rsa -in RSA.pem -passin pass:123456 -aes128 -passout pass:123456 -out E_RSA.pem
```

3、查看密钥对中的各个参数

```shell
$ openssl rsa -in RSA.pem -des -passin pass:123456 -text -noout
```

4、提取密钥中的公钥并打印模数值

```shell
/*提取公钥，用pubout参数指定输出为公钥*/
$ openssl rsa -in RSA.pem -passin pass:123456 -pubout -out pub.pem

/*打印公钥中模数值*/
$ openssl rsa -in pub.pem -pubin -modulus -noout

Modulus=C35E0B54041D78466EAE7DE67C1DA4D26575BC1608CE6A199012E11D10ED36E2F7C651D4D8B40D93691D901E2CF4E21687E912B77DCCE069373A7F6585E946EF
```

5、转换密钥的格式

```shell
/*把pem格式转化成der格式，使用outform指定der格式*/
$ openssl rsa -in RSA.pem -passin pass:123456 -des -passout pass:123456 -outform der -out rsa.der

/*把der格式转化成pem格式，使用inform指定der格式*/
$ openssl rsa -in rsa.der -inform der -passin pass:123456 -out rsa.pem
```



---

### 实验

#### 生成私钥

`-passout` 表示为输出的文件进行加密（对称加密）

```shell
[sure@localhost cxx]$ openssl genrsa -out private.key -passout file:password.c -des3
Generating RSA private key, 2048 bit long modulus
...........+++
.....................................+++
e is 65537 (0x10001)
[sure@localhost cxx]$ cat private.key 
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,A887E53A8305150D

/RZGr10e9Y3VV5/rdXXHV5JroceH9C7DirHWdGVmCNKtuYd/HX7YM2XMtZf6eYpO
Zt0/FNt55y80YgnyNdrbzJDE81yemYRaHn9qqwCuS3JWIKhaEGGf/pGcda0azQ4Y
eI2fkb2mapHhVyMepLynN1ahFZ1h+ZEwuCA3wUmkhz1dgT5eXdxDWVy0LZ/lhKuj
LbV7QQnEMQvubWdWrIGs6fpyYc+JNNJtPs0tjhQarF1EJcNd5HqR7fj8VcOwFqcL
esEMMJKlGI+lnKilGuMpBoPah8QQ3C4hhx/aUMiXGrxIxvhc7T0yMVybOTT3vyhi
Vxq44iiPbGO2m1Zh8sCYuniPXGvqduKI/medViU1QdF7V1Abnbi+TnX6BixctqKP
Rz+ap4obvcWZs6AU9uxYuToAKfNOlMn/kvqfTYBDNfiLfGCLU07rGB8m3JLupvEc
TSERNisbE01ZS/acdjT8pCM3M50uS5QWw2E+xRj0B4tVhkvEgQwvqd80Jvy5Za1Y
FlRIilUpK2QVVskG0DEGep5+wyG2SKshpUMeZLdsvUQqaQyj1wgCf0iaGqwy91+o
VOKpwSwETyxzt1NZ3bASMZk1ztRCKZLXOMX5KqXSaOv6jfK6MavYSwk5Jlo5Des3
zfqD9gcw6Vh0l52ctdzKeqdMcrJ74jTGmSzdw8Qy9Nd46SixiuKXjQgllyZQ8XAe
HspS5DnpfeZgxvNA4Nbjs/pWjNQeMhvJmhK+XUqoKDTM8y1rSVR2rxCPoQ3sSst2
L3QSaWrdYVSaCs1UCDLXKTOu0+fZ54hsBUo7TRKHbvLqWr1UwW4GVH8WT5uh3S7F
MmDty5Fm7V1IBHqB8NCZiReIM2kT7126lglP/Dh7OdPWsRZg3417sUVzjh5hYTku
QCB3bn+fHKRAR8P1dRdwhxWabhf6TcMjhU6PuzAmPGQ0c/d5ziXYxvlDXiTehsg4
q4VpOQFvK9t9xAhOEYj2tYlL8vEC3zXD/NrVDA7UWCCMtBvpn2xDTVr+HshKhUI2
DuxGkM8ClYTqHeQQTGM/GwIteDF/nYz2NNMsoMmXWLQ9dfLNhTHOLDu5Zm3zdxus
bO4tlJ6Cj3OSI7nhu46I6KnrGm+25Vo6LYrKgcRdfHET+1sLAvjGPiDqsiKsqt6v
e5KQ256DILsPCCcEOCXcBG4j0fG9pAvIeAsWWH0plLIKbqUrUoJVWvYfkkfx8/kB
S3KB8jPXJSIy3y2f5btkzfMd/nuRkvWpLKPx8Cyqb8ioYNVGyOiphZH8l5ov+il8
srNYuEx5CvHVk8rFyKaq7C8QDDUlU/oTwM55HOrOXv0lBOXKE0doPHp3vqosImZq
do31uKawO7sDnXGQGBTt5FPl2nLGhb7RqSw5tkYqykGJOS1+2VshUOCDWheS+jLM
U+TN2ssxGVsXy4waOMxt7kUyaxcVDhOZJKLsQiQcNk0D0LY8Aq99AsFUN9vG2RXz
1N1W4U0ul16HyAX87WC4CSa8ShdESste3I8HW0S+iFdyfacAYfj4vaQi0gAXpQEh
juIQ3Pw5QptjtDUVGBue1Ewn7FOM8xxirDUdvdXEEGTzlEGxSVSTPA==
-----END RSA PRIVATE KEY-----
[sure@localhost cxx]$ 

```

可以再命令的最后指定私钥长度，默认是512bit

```shell
openssl genrsa -out private.key -des3 1024
```



#### 生成公钥

`-pubout` 指定输出文件是公钥

`-passin` ：如果输入的文件被加密，使用passin进行解密

`-passout` ：对输出的公钥进行加密（对称加密）

```shell
[sure@localhost cxx]$ openssl rsa -in private.key -pubout -out public.key -passin file:password.c 
writing RSA key
[sure@localhost cxx]$ cat public.key 
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0rX+Ik9IcTCeyd2/99+n
n4iKRj0e1EYYTqF1aNQpXNFLZNXpDjhU1QKqLROx7iSTpzn6NJmiJmKnYuGO/39+
4DzDpLCtaKPoAz19ChEHDQJmhaMU1CyCxCtEkCKQsLCfs5su8v95CwfVK5LmLO3s
1raSyK7Hb7mvLXEFiuL8V9zzp5d9WJ0vXhY8cN+GQSh4dIoge74868lhw3Zpgq4+
H7EB775PceN3FXL4NLConqYZbppG0k1tj551MDEcAZoEoUJwx7PBBheacnQCxHya
9LLvX2h2P8c04W4SpfF/D04IwBSZf7Z8yrQhRv5ZeX+0gHLW4yxzNGaPSL7c5B82
WwIDAQAB
-----END PUBLIC KEY-----
[sure@localhost cxx]$ 
```



### RSA其他功能

##### 添加、去除密钥的对称加密

```shell
# 生成无密码的rsa私钥
[sure@localhost cxx]$ openssl genrsa -out nopass.pem
Generating RSA private key, 2048 bit long modulus
.............................................................................................................+++
..................................................................................+++
e is 65537 (0x10001)
[sure@localhost cxx]$ cat nopass.pem 
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAv0hNdmlZr9NcSWWIZk8xI6RTUkybVR19BsSo3G+84ez/zc9C
a8FT3QILukD8HFyHjGHV85yW4yyArUdDyrL+J+mQqUF5Tc/kR/kKwwdPQu39kX6e
5AkxlPzXgjoYd1kzGaQeJqAhLqj/704d/NIpga7CD1F12nor2kIwLKmK/BnnrqHs
vjJZ4SrIRKUsdufLSVUa9SJ9oGQ3Nhzdg/8gTdk18F5x8BIjNZhdvkNZHXQSy2JK
jEGl4ptMqzA1CtmdZBcZJsn3yQxqMPcTG9vA/xJETcUhHwHkFolDTchp8G+PV9e1
B9bqYRiXmRKT0RexGS3K3SBM8whHONjJ+FuG7wIDAQABAoIBAQCX+TT6QEduh4oK
Em4lgwOyoqtEdvLu1AfyqarTwL8b7PVsKiBGhoo/zJFOwLTNP8K+CTk4XRAQm9n8
UeONl1qQkWRK5WcgKGzhtf8T5qnVrkpJH4XT/W30RlJe+BNaN3d/BsKhw5W5gbIe
Cj2PEdbCXvt1ui3dkDVpKi8mPOnc0sR7pXo3bfp4yTiFKw3QU7DdW6bKdjc7rFZj
TWyqQ+4hOTsdYv5KoMQXum2adGknHIECbe8s/LatdzJzALhoL+YQwSSqThdsIAwM
s4iUo+H66VQv9S6LEUwAA+5AJ2quHEDu/2xaSDRlz9vROXJu2RyD7Xet36GNhR6P
bYKTQbGZAoGBAN1WMJRvInkzvAP0NZn9U6T2BudC/5izuHPNXhog9dMdRTfFAwHZ
t383QpGS9O9vrUjX/faq5UoxxekOKxR8mQutZqhy3o9MKYahNKrjtsHwaIdW4iHN
PLcF6CDSOMJrI3iJRWjIL8IzXvbj4KoYpPrGMSstGR34bUHnmDIKN7AtAoGBAN09
Lj/atzuQ50enMMSpRY7DOS2S/Wq220HFcVot11oxeyDQMucGiEXCqlD9jEFQ9qkn
Gr+bd7Snp6umU4PB5iwV1BLHmgphCB2+yw8RDPxZaazK3fNTo6jY9GjzgcuhU6pP
5k9oEbcIrXiQUIo0hkN358FEF+wabVX0vLi/1ukLAoGAYhwkaIdiporyGmaTo/CQ
tRyBLt2Z4pw3dM1hmv9lN/FPj0r67EUPe4qJLXIQtFmyXAmx/zb9cAfkDExFeE1K
ocx5Js3ULXy7I3wtllpd1lW0X9l5XzZUZWRu4q2Mj1FiZbmjVLD3yoNu4s1b9sn5
x1c20EarTYejFoWMBxJUYPkCgYEAhSvjoAoui4twvD/WajqeJQ48Z8N4CXliR5fq
4GaBn8fzHtBUI55Z/uvri27jsxliMHXacwXJK5RTqE4pLUFVJKpLCrbdcWvw776+
CiawU2Ia6yj+Kw7oj6VwkZAqTAGjE/yeXKP/LdbqXI05/ccaHpiZh0tOvw81Sy1T
QD4xxfsCgYEAuSN6V1thw0iBzQQMBV2k/5B9murS9E8ieQ5rRqV1tMhTOMXVDmVd
MlKHFsrqo39N3sXNPV9enyxtApePNSMHhpsS/j5nQHGLUcKSqcFXpjqWyTIqx0PZ
8KP72Md9kLOaekxGUTy6RtZaLeQ1RqLn+1/m8foCao0DQ8YNXCCRYE0=
-----END RSA PRIVATE KEY-----

# 为密钥添加密码（给现有的密钥添加密码，生成另一个密钥）
[sure@localhost cxx]$ openssl rsa -in nopass.pem -des3 -passout pass:123 -out addpass.pem
writing RSA key
[sure@localhost cxx]$ cat addpass.pem 
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,79B6E11D6286C3FC

KOi+SpOnHLOYZp35GMuhq95Ibayx59KvHHZPALnh6UjQf48I0MGircd1xTOzSuJ9
sUPiZD1qq//uYvy5Exj/cWtTDcQSZ4au1V7IE/kO37WqogSakHc1lE+mRZRmeQoP
xvrJwMkUCD3pMAmxZirqcRuFWNnkWiu3tXZX/fJOeh8zh7HMHuZNoKMCP7cCrcTK
xg36ltxf6D0noVLoVDzfNPYOEK8ojjHq++zC81rrluBAJ0Ajg0wVOZAHhzhag34L
j1qT5yBYhM+FdUGxs+hrrETyLTm/23cFyhUI8l2G8uBZGEsTHn3ethYDWnIuJ2PB
XI4yONkF3EcqMJ0uIYFovETkmUW7HUwJwiNdqlSmGeGvNmieDClwZ8wdKPJxWKFB
oQnT4tSkbWerONHKK3VkTbYQWC+HNWW5TO9sim7ao6yM9tAHvq7Gprm02DxymJl9
iTDaXSQXJ6MZWb+BBmZgGE8XJ9BHcUKByfPBi9sYepWQsToqlfnIIvZnzb6J1mY8
2ubr3cL57Rgk349QuGUp5+g6xaX0xfvx9GlZRhTCISxcX4J5DE6OazmyUSViWwo3
1aaDH2K4gOQPIM3je3KAniSPIZYnF5SBHXfITygg8NwZP7lJjmCEFmtF82RtLvfX
S9IywTwSMbeQJPUBHDaXyzIOmYBg3gQxBhj3JeGla8jl9lFu0MhGJJZXfBl6qorW
0LSCqyO09WfUWT2D5jab2hbK2w88qhoxrQ2K2Lb14pGBt6C4E365uiwKKopgHTV7
qAGe4egu5rR+f6ySj2cw9UvLwsVqtbnnVCbTNcor886MXwt4NaAwC8f3Vp3XUAD1
sFElmpYcNzPS0vQnGIn1nIq7vScU1Zzhcz59le+orNy1/XwcwdlcXbnv74lb1aO0
ho+G6RyktkZ40xM3YG6Q5YOSKityOQGVjGeUVihbHX/UkOUeM5deE/cfdHvWt/Xx
fDOmdnckyW2qIt/OotkDGH87YZFhu2laeL3htfcOcSkqncjl0VJH66W4MnrayMBf
ECSYOkU6dTHi6ZB09tTv2b2ibn3czfT6/rjJBdjlp8iZZAIZeMv0w3EkQsKJStfU
z4yTqtB1TSPyrdT0rdHgCpAp8iVMKT9BTOlCGCpnihJoVTdkTjrqRxmzxCiMmqk2
bxjGds7Ep8ppbddSDw5jN2bXMe9jU4YKQ+RfIQTmkI3163j/yC7J0TRPE5ioO6by
oQtgTTdZQuhbOLiAledtfzwiBXx6GDiRiJUyBU8EmgiOxf1nV8iWXqLum9DRwwLK
KDNad3E/FiMQvM4MKkEYJqybpvGw3z7nOpqFvfYtNZSjt4IVIGG5PUSQDgvGGnGD
eCTfHgvKDJWwaOC8VJ0cp2I159OHRPunJkwzzUrchOF4UgyU8zhKkp3iI/WhzIfG
rtg36t5h5A+EoQ8S3WuzebPQlrgetOO1nKPjupxyS/6DtIOMyyqpIs5KLS+xuU0C
Hr9hlxB09GKeSZfHBYBL84LLjdzY379nfzzrzaYG9yOp5BnatEbnhWtWDbrWKLra
mR62vmcMDchZ+p4pRcR2dxIMpaoZd1PCG/gLjFrzm2aCSDIWThOPeawosWbkNKMH
-----END RSA PRIVATE KEY-----

# 去除密钥的密码
[sure@localhost cxx]$ openssl rsa -in addpass.pem -passin pass:123 -out removepass.pem
writing RSA key
# 比较经过加解密后的密钥，是一样的
[sure@localhost cxx]$ diff nopass.pem removepass.pem 


# 更改密码与加密方式	/将密码改为123456并将加密方式改为-aes128
[sure@localhost cxx]$ openssl rsa -in addpass.pem -passin pass:123 -aes128 -out chpass.pem -passout pass:123456
writing RSA key
[sure@localhost cxx]$ 

# 查看密钥对中的各个参数
[sure@localhost cxx]$ openssl rsa -in chpass.pem -aes128 -passin pass:123456 -text -noout
Private-Key: (2048 bit)
modulus:
    00:bf:48:4d:76:69:59:af:d3:5c:49:65:88:66:4f:
    31:23:a4:53:52:4c:9b:55:1d:7d:06:c4:a8:dc:6f:
    bc:e1:ec:ff:cd:cf:42:6b:c1:53:dd:02:0b:ba:40:
    fc:1c:5c:87:8c:61:d5:f3:9c:96:e3:2c:80:ad:47:
    43:ca:b2:fe:27:e9:90:a9:41:79:4d:cf:e4:47:f9:
    0a:c3:07:4f:42:ed:fd:91:7e:9e:e4:09:31:94:fc:
    d7:82:3a:18:77:59:33:19:a4:1e:26:a0:21:2e:a8:
    ff:ef:4e:1d:fc:d2:29:81:ae:c2:0f:51:75:da:7a:
    2b:da:42:30:2c:a9:8a:fc:19:e7:ae:a1:ec:be:32:
    59:e1:2a:c8:44:a5:2c:76:e7:cb:49:55:1a:f5:22:
    7d:a0:64:37:36:1c:dd:83:ff:20:4d:d9:35:f0:5e:
    71:f0:12:23:35:98:5d:be:43:59:1d:74:12:cb:62:
    4a:8c:41:a5:e2:9b:4c:ab:30:35:0a:d9:9d:64:17:
    19:26:c9:f7:c9:0c:6a:30:f7:13:1b:db:c0:ff:12:
    44:4d:c5:21:1f:01:e4:16:89:43:4d:c8:69:f0:6f:
    8f:57:d7:b5:07:d6:ea:61:18:97:99:12:93:d1:17:
    b1:19:2d:ca:dd:20:4c:f3:08:47:38:d8:c9:f8:5b:
    86:ef
publicExponent: 65537 (0x10001)
privateExponent:
    00:97:f9:34:fa:40:47:6e:87:8a:0a:12:6e:25:83:
    03:b2:a2:ab:44:76:f2:ee:d4:07:f2:a9:aa:d3:c0:
    bf:1b:ec:f5:6c:2a:20:46:86:8a:3f:cc:91:4e:c0:
    b4:cd:3f:c2:be:09:39:38:5d:10:10:9b:d9:fc:51:
    e3:8d:97:5a:90:91:64:4a:e5:67:20:28:6c:e1:b5:
    ff:13:e6:a9:d5:ae:4a:49:1f:85:d3:fd:6d:f4:46:
    52:5e:f8:13:5a:37:77:7f:06:c2:a1:c3:95:b9:81:
    b2:1e:0a:3d:8f:11:d6:c2:5e:fb:75:ba:2d:dd:90:
    35:69:2a:2f:26:3c:e9:dc:d2:c4:7b:a5:7a:37:6d:
    fa:78:c9:38:85:2b:0d:d0:53:b0:dd:5b:a6:ca:76:
    37:3b:ac:56:63:4d:6c:aa:43:ee:21:39:3b:1d:62:
    fe:4a:a0:c4:17:ba:6d:9a:74:69:27:1c:81:02:6d:
    ef:2c:fc:b6:ad:77:32:73:00:b8:68:2f:e6:10:c1:
    24:aa:4e:17:6c:20:0c:0c:b3:88:94:a3:e1:fa:e9:
    54:2f:f5:2e:8b:11:4c:00:03:ee:40:27:6a:ae:1c:
    40:ee:ff:6c:5a:48:34:65:cf:db:d1:39:72:6e:d9:
    1c:83:ed:77:ad:df:a1:8d:85:1e:8f:6d:82:93:41:
    b1:99
prime1:
    00:dd:56:30:94:6f:22:79:33:bc:03:f4:35:99:fd:
    53:a4:f6:06:e7:42:ff:98:b3:b8:73:cd:5e:1a:20:
    f5:d3:1d:45:37:c5:03:01:d9:b7:7f:37:42:91:92:
    f4:ef:6f:ad:48:d7:fd:f6:aa:e5:4a:31:c5:e9:0e:
    2b:14:7c:99:0b:ad:66:a8:72:de:8f:4c:29:86:a1:
    34:aa:e3:b6:c1:f0:68:87:56:e2:21:cd:3c:b7:05:
    e8:20:d2:38:c2:6b:23:78:89:45:68:c8:2f:c2:33:
    5e:f6:e3:e0:aa:18:a4:fa:c6:31:2b:2d:19:1d:f8:
    6d:41:e7:98:32:0a:37:b0:2d
prime2:
    00:dd:3d:2e:3f:da:b7:3b:90:e7:47:a7:30:c4:a9:
    45:8e:c3:39:2d:92:fd:6a:b6:db:41:c5:71:5a:2d:
    d7:5a:31:7b:20:d0:32:e7:06:88:45:c2:aa:50:fd:
    8c:41:50:f6:a9:27:1a:bf:9b:77:b4:a7:a7:ab:a6:
    53:83:c1:e6:2c:15:d4:12:c7:9a:0a:61:08:1d:be:
    cb:0f:11:0c:fc:59:69:ac:ca:dd:f3:53:a3:a8:d8:
    f4:68:f3:81:cb:a1:53:aa:4f:e6:4f:68:11:b7:08:
    ad:78:90:50:8a:34:86:43:77:e7:c1:44:17:ec:1a:
    6d:55:f4:bc:b8:bf:d6:e9:0b
exponent1:
    62:1c:24:68:87:62:a6:8a:f2:1a:66:93:a3:f0:90:
    b5:1c:81:2e:dd:99:e2:9c:37:74:cd:61:9a:ff:65:
    37:f1:4f:8f:4a:fa:ec:45:0f:7b:8a:89:2d:72:10:
    b4:59:b2:5c:09:b1:ff:36:fd:70:07:e4:0c:4c:45:
    78:4d:4a:a1:cc:79:26:cd:d4:2d:7c:bb:23:7c:2d:
    96:5a:5d:d6:55:b4:5f:d9:79:5f:36:54:65:64:6e:
    e2:ad:8c:8f:51:62:65:b9:a3:54:b0:f7:ca:83:6e:
    e2:cd:5b:f6:c9:f9:c7:57:36:d0:46:ab:4d:87:a3:
    16:85:8c:07:12:54:60:f9
exponent2:
    00:85:2b:e3:a0:0a:2e:8b:8b:70:bc:3f:d6:6a:3a:
    9e:25:0e:3c:67:c3:78:09:79:62:47:97:ea:e0:66:
    81:9f:c7:f3:1e:d0:54:23:9e:59:fe:eb:eb:8b:6e:
    e3:b3:19:62:30:75:da:73:05:c9:2b:94:53:a8:4e:
    29:2d:41:55:24:aa:4b:0a:b6:dd:71:6b:f0:ef:be:
    be:0a:26:b0:53:62:1a:eb:28:fe:2b:0e:e8:8f:a5:
    70:91:90:2a:4c:01:a3:13:fc:9e:5c:a3:ff:2d:d6:
    ea:5c:8d:39:fd:c7:1a:1e:98:99:87:4b:4e:bf:0f:
    35:4b:2d:53:40:3e:31:c5:fb
coefficient:
    00:b9:23:7a:57:5b:61:c3:48:81:cd:04:0c:05:5d:
    a4:ff:90:7d:9a:ea:d2:f4:4f:22:79:0e:6b:46:a5:
    75:b4:c8:53:38:c5:d5:0e:65:5d:32:52:87:16:ca:
    ea:a3:7f:4d:de:c5:cd:3d:5f:5e:9f:2c:6d:02:97:
    8f:35:23:07:86:9b:12:fe:3e:67:40:71:8b:51:c2:
    92:a9:c1:57:a6:3a:96:c9:32:2a:c7:43:d9:f0:a3:
    fb:d8:c7:7d:90:b3:9a:7a:4c:46:51:3c:ba:46:d6:
    5a:2d:e4:35:46:a2:e7:fb:5f:e6:f1:fa:02:6a:8d:
    03:43:c6:0d:5c:20:91:60:4d


# 提取公钥并打印模数值
# 用pubout指定输出为公钥
[sure@localhost cxx]$ openssl rsa -in chpass.pem -passin pass:123456 -pubout -out chpub.key
writing RSA key
# 打印公钥中模数值
[sure@localhost cxx]$ openssl rsa -in chpub.key -pubin -modulus -noout
Modulus=BF484D766959AFD35C496588664F3123A453524C9B551D7D06C4A8DC6FBCE1ECFFCDCF426BC153DD020BBA40FC1C5C878C61D5F39C96E32C80AD4743CAB2FE27E990A941794DCFE447F90AC3074F42EDFD917E9EE4093194FCD7823A1877593319A41E26A0212EA8FFEF4E1DFCD22981AEC20F5175DA7A2BDA42302CA98AFC19E7AEA1ECBE3259E12AC844A52C76E7CB49551AF5227DA06437361CDD83FF204DD935F05E71F0122335985DBE43591D7412CB624A8C41A5E29B4CAB30350AD99D64171926C9F7C90C6A30F7131BDBC0FF12444DC5211F01E41689434DC869F06F8F57D7B507D6EA611897991293D117B1192DCADD204CF3084738D8C9F85B86EF


# 转换密钥的格式
[sure@localhost cxx]$ openssl rsa -in chpass.pem -passin pass:123456 -des -ut pass:123456 -outform der -out chpass.der
writing RSA key
[sure@localhost cxx]$ openssl rsa -in chpass.der -inform der -passin pass:123456 -out chpass2.pem
writing RSA key
# 前后结果是不同的
[sure@localhost cxx]$ diff chpass.pem chpass2.pem 
2,29c2,26
```



**总结功能**

`openssl genrsa` 用于生成私钥

`openssl rsa` 用于对密钥的各种操作，比如为私钥生成公钥、对密钥设置或解除密码、查看密钥信息、改变密钥格式等

---



### 使用已有的公私钥对，进行非对称加解密

使用 `openssl rsautl` 命令

>   注意：无论是使用公钥加密还是私钥加密，RSA每次能够加密的数据长度不能超过RSA密钥长度，并且根据具体的补齐方式不同输入的加密数据最大长度也不一样，而输出长度则总是跟RSA密钥长度相等。RSA不同的补齐方法对应的输入输入长度如下表

| 数据补齐方式      | 输入数据长度          | 输出数据长度 | 参数字符串 |
| ----------------- | --------------------- | ------------ | ---------- |
| PKCS#1 v1.5       | 少于(密钥长度-11)字节 | 同密钥长度   | -pkcs      |
| PKCS#1 OAEP       | 少于(密钥长度-11)字节 | 同密钥长度   | -oaep      |
| PKCS#1 for SSLv23 | 少于(密钥长度-11)字节 | 同密钥长度   | -ssl       |
| 不使用补齐        | 同密钥长度            | 同密钥长度   | -raw       |

>   使用rsautl进行加密和解密操作，我们还是先看一下帮助文档

```shell
$ openssl rsautl -h
Usage: rsautl [options]                  
-in file        input file                                           //输入文件
-out file       output file                                          //输出文件
-inkey file     input key                                            //输入的密钥
-keyform arg    private key format - default PEM                     //指定密钥格式
-pubin          input is an RSA public                               //指定输入的是RSA公钥
-certin         input is a certificate carrying an RSA public key    //指定输入的是证书文件
-ssl            use SSL v2 padding                                   //使用SSLv23的填充方式
-raw            use no padding                                       //不进行填充
-pkcs           use PKCS#1 v1.5 padding (default)                    //使用V1.5的填充方式
-oaep           use PKCS#1 OAEP                                      //使用OAEP的填充方式
-sign           sign with private key                                //使用私钥做签名
-verify         verify with public key                               //使用公钥认证签名
-encrypt        encrypt with public key                              //使用公钥加密
-decrypt        decrypt with private key                             //使用私钥解密
-hexdump        hex dump output                                      //以16进制dump输出
-engine e       use engine e, possibly a hardware device.            //指定三方库或者硬件设备
-passin arg    pass phrase source                                    //指定输入的密码
```

>   openssl rsautl 基本的加解密使用

```shell
/*生成RSA密钥*/
$ openssl genrsa -des3 -passout pass:123456 -out RSA.pem 

Generating RSA private key, 512 bit long modulus
............++++++++++++
...++++++++++++
e is 65537 (0x10001)

/*提取公钥*/
$ openssl rsa -in RSA.pem -passin pass:123456 -pubout -out pub.pem 

/*使用RSA作为密钥进行加密，实际上使用其中的公钥进行加密*/
$ openssl rsautl -encrypt -in plain.txt -inkey RSA.pem -passin pass:123456 -out enc.txt

/*使用RSA作为密钥进行解密，实际上使用其中的私钥进行解密*/
$ openssl rsautl -decrypt -in enc.txt -inkey RSA.pem -passin pass:123456 -out replain.txt

/*比较原始文件和解密后文件*/
$ diff plain.txt replain.txt 

/*使用公钥进行加密*/
$ openssl rsautl -encrypt -in plain.txt -inkey pub.pem -pubin -out enc1.txt

/*私钥进行解密*/
$ openssl rsautl -decrypt -in enc1.txt -inkey RSA.pem -passin pass:123456 -out replain1.txt

/*比较原始文件和解密后文件*/
$ diff plain.txt replain1.txt
```

>   签名与验证操作

```shell
/*提取PCKS8格式的私钥*/
$ openssl pkcs8 -topk8 -in RSA.pem -passin pass:123456 -out pri.pem -nocrypt

/*使用RSA密钥进行签名，实际上使用私钥进行加密*/
$ openssl rsautl -sign -in plain.txt -inkey RSA.pem -passin pass:123456 -out sign.txt

/*使用RSA密钥进行验证，实际上使用公钥进行解密*/
$ openssl rsautl -verify -in sign.txt -inkey RSA.pem -passin pass:123456 -out replain.txt

/*对比原始文件和签名解密后的文件*/
$ diff plain.txt replain.txt 

/*使用私钥进行签名*/
$ openssl rsautl -sign -in plain.txt -inkey pri.pem  -out sign1.txt

/*使用公钥进行验证*/
$ openssl rsautl -verify -in sign1.txt -inkey pub.pem -pubin -out replain1.txt

/*对比原始文件和签名解密后的文件*/
$ cat plain replain1.txt
```





---

### 实验

一条龙

```shell
# 生成私钥	注意，这里的RSA是一个包含了私钥和公钥的文件，私钥和公钥可以从其中分离出来，所以直接叫私钥似乎不准确
[sure@localhost cxx]$ openssl genrsa -des3 -out RSA.pem -passout file:password.c 
Generating RSA private key, 2048 bit long modulus
............+++
.......................+++
e is 65537 (0x10001)
# 生成公钥
[sure@localhost cxx]$ openssl rsa -in RSA.pem -pubout -out public.pem -passin file:password.c 
writing RSA key
# 使用公钥加密 -pubin指定这是一个公钥
[sure@localhost cxx]$ openssl rsautl -encrypt -in hello.c -out passhello.c -pubin -inkey public.pem -passin file:password.c    
# 使用私钥解密
[sure@localhost cxx]$ openssl rsautl -decrypt -in passhello.c -out hello2.c -inkey RSA.pem -passin file:password.c 
# 比较前后文件
[sure@localhost cxx]$ diff hello.c hello2.c 
[sure@localhost cxx]$ 
```

注意点

```shell
[sure@localhost cxx]$ openssl rsautl -encrypt -in hello.c -out passhello.c -inkey public.pem -passin file:password.c
unable to load Private Key
140324106938256:error:0906D06C:PEM routines:PEM_read_bio:no start line:pem_lib.c:707:Expecting: ANY PRIVATE KEY
```

无法加载私钥，明明是使用公钥加密的，干嘛要去加载私钥呢？

原因：

没有指定是使用公钥加密，就是没有加 `-pubin` 参数，不加参数的话，默认是使用私钥文件进行加密，即在私钥文件中提取公钥再加密，也就是不加 `-pubin` 参数的话，加密命令为：

即：不使用生成的公钥文件，而是直接在私钥中提取公钥进行加密（这意味着从私钥能推导出公钥？）

```shell
[sure@localhost cxx]$ openssl rsautl -encrypt -in hello.c -out passhello.c -inkey RSA.pem -passin file:password.c 
```

解密命令不变

```shell
[sure@localhost cxx]$ openssl rsautl -decrypt -in passhello.c -out hello2.c -inkey RSA.pem -passin file:password.c 
[sure@localhost cxx]$ diff hello.c hello2.c 
[sure@localhost cxx]$ 
```

---





### 签名

签名：A对信息进行签名是确认这个信息是A发出的

加密：确保信息只有B可以获取

一般公钥用来加密，私钥用来签名

**通常发送信息的流程为：**

假如A要发信息给B：

1.  A使用自己的私钥对信息（一般是信息摘要）进行签名。
2.  A使用B的公钥对信息内容和签名信息进行加密

B接到信息后：

1.  使用自己的私钥解密内容
2.  得到解密后的明文后用A的公钥解签（验证）A的签名





---

### 实验

```shell
# -nocrypt 表示不设置私钥密码
[sure@localhost cxx]$ openssl pkcs8 -topk8 -in RSA.pem -passin file:password.c -out private.pem -nocrypt
[sure@localhost cxx]$ openssl rsautl -sign -in hello.c -inkey private.pem -out signhello.c
[sure@localhost cxx]$ openssl rsautl -verify -in signhello.c -inkey public.pem -pubin -out hellosign.c
[sure@localhost cxx]$ diff hello.c hellosign.c 
# 生成私钥时为私钥设置密码
[sure@localhost cxx]$ openssl pkcs8 -topk8 -in RSA.pem -passin file:password.c -out private2.pem
Enter Encryption Password:
Verifying - Enter Encryption Password:
# 使用私钥时输入密码
[sure@localhost cxx]$ openssl rsautl -sign -in hello.c -inkey private2.pem -out signhello.c
Enter pass phrase for private2.pem:
# 使用公钥解签时无需密码
[sure@localhost cxx]$ openssl rsautl -verify -in signhello.c -inkey public.pem -pubin -out hellosign.c
[sure@localhost cxx]$ diff hello.c hellosign.c 
# 直接使用RSA进行签名解签
[sure@localhost cxx]$ openssl rsautl -sign -in hello.c -inkey RSA.pem -passin file:password.c -out hellosign.c
[sure@localhost cxx]$ openssl rsautl -verify -in hellosign.c -inkey RSA.pem -passin file:password.c -out hello2.c
[sure@localhost cxx]$ diff hello.c hello2.c
[sure@localhost cxx]$ 
```

