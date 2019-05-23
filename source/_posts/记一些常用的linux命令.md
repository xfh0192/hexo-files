---
title: 记一些常用的linux命令
date: 2019-03-20 00:20:13
tags: 
- linux
---


## lsof

通过lsof工具能够查看应用程序打开文件的描述符列表。

- 在linux环境下，++任何事物都以文件的形式存在++，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。所以如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等，系统在后台都为该应用程序分配了一个++文件描述符++，无论这个文件的本质如何，该文件描述符为应用程序与基础操作系统之间的交互提供了通用接口。
- 因为应用程序打开文件的描述符列表提供了大量关于这个应用程序本身的信息，因此通过lsof工具能够查看这个列表对系统监测以及排错将是很有帮助的。

语法：
```
lsof [options]
```

选项：
```
-a：列出打开文件存在的进程；
-c<进程名>：列出指定进程所打开的文件；
-g：列出GID号进程详情；
-d<文件号>：列出占用该文件号的进程；
+d<目录>：列出目录下被打开的文件；
+D<目录>：递归列出目录下被打开的文件；
-n<目录>：列出使用NFS的文件；
-i<条件>：列出符合条件的进程。（4、6、协议、:端口、 @ip ）
-p<进程号>：列出指定进程号所打开的文件；
-u：列出UID号进程详情；
-h：显示帮助信息；
-v：显示版本信息。
```

例如：
> lsof -i:<port>

```
# lsof -i:8000
COMMAND   PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
lwfs    22065 root    6u  IPv4 4395053      0t0  TCP *:irdmi (LISTEN)
```

输出column含义：
```
lsof输出各列信息的意义如下：

COMMAND：进程的名称
PID：进程标识符
PPID：父进程标识符（需要指定-R参数）
USER：进程所有者
PGID：进程所属组
FD：文件描述符，应用程序通过文件描述符识别该文件。
```

文件描述符列表：
```
cwd：表示current work dirctory，即：应用程序的当前工作目录，这是该应用程序启动的目录，除非它本身对这个目录进行更改
txt：该类型的文件是程序代码，如应用程序二进制文件本身或共享库，如上列表中显示的 /sbin/init 程序
lnn：library references (AIX);
er：FD information error (see NAME column);
jld：jail directory (FreeBSD);
ltx：shared library text (code and data);
mxx ：hex memory-mapped type number xx.
m86：DOS Merge mapped file;
mem：memory-mapped file;
mmap：memory-mapped device;
pd：parent directory;
rtd：root directory;
tr：kernel trace file (OpenBSD);
v86  VP/ix mapped file;
0：表示标准输出
1：表示标准输入
2：表示标准错误
```

文件类型：
```
文件类型：

DIR：表示目录。
CHR：表示字符类型。
BLK：块设备类型。
UNIX： UNIX 域套接字。
FIFO：先进先出 (FIFO) 队列。
IPv4：网际协议 (IP) 套接字。
DEVICE：指定磁盘的名称
SIZE：文件的大小
NODE：索引节点（文件在磁盘上的标识）
NAME：打开文件的确切名称
```
## ps

ps命令用于报告当前系统的进程状态。

ps命令用于报告当前系统的进程状态。可以搭配kill指令随时中断、删除不必要的程序。ps命令是最基本同时也是非常强大的进程查看命令，使用该命令可以确定有哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵死、哪些进程占用了过多的资源等等，总之大部分信息都是可以通过执行该命令得到的。

语法：
> ps [options]

```
常用
ps aux | grep [进程名称]

1.查进程
    ps命令查找与进程相关的PID号：
    ps a 显示现行终端机下的所有程序，包括其他用户的程序。
    ps -A 显示所有程序。
    ps c 列出程序时，显示每个程序真正的指令名称，而不包含路径，参数或常驻服务的标示。
    ps -e 此参数的效果和指定"A"参数相同。
    ps e 列出程序时，显示每个程序所使用的环境变量。
    ps f 用ASCII字符显示树状结构，表达程序间的相互关系。
    ps -H 显示树状结构，表示程序间的相互关系。
    ps -N 显示所有的程序，除了执行ps指令终端机下的程序之外。
    ps s 采用程序信号的格式显示程序状况。
    ps S 列出程序时，包括已中断的子程序资料。
    ps -t<终端机编号> 指定终端机编号，并列出属于该终端机的程序的状况。
    ps u 以用户为主的格式来显示程序状况。
    ps x 显示所有程序，不以终端机来区分。
   
    最常用的方法是ps aux,然后再通过管道使用grep命令过滤查找特定的进程,然后再对特定的进程进行操作。
    ps aux | grep program_filter_word,ps -ef |grep tomcat

如：
ps -ef|grep java|grep -v grep 显示出所有的java进程，去除掉当前的grep进程。
```

> 在使用Linux命令时，如果命令中有管道“|”，则输出的信息中，头（标题）信息丢失，要想看每一列代表什么意思很不方便。

> 这里有一个简单的办法，通过2条命令叠加，获取头和内容。例如ps auxw：

>  ps axuw | head -1;ps axuw | grep java


## netstat

netstat命令用来打印Linux中网络系统的状态信息，可让你得知整个Linux系统的网络情况。

语法：
> netstat [options]

选项：
```
-a或--all：显示所有连线中的Socket；
-A<网络类型>或--<网络类型>：列出该网络类型连线中的相关地址；
-c或--continuous：持续列出网络状态；
-C或--cache：显示路由器配置的快取信息；
-e或--extend：显示网络其他相关信息；
-F或--fib：显示FIB；
-g或--groups：显示多重广播功能群组组员名单；
-h或--help：在线帮助；
-i或--interfaces：显示网络界面信息表单；
-l或--listening：显示监控中的服务器的Socket；
-M或--masquerade：显示伪装的网络连线；
-n或--numeric：直接使用ip地址，而不通过域名服务器；
-N或--netlink或--symbolic：显示网络硬件外围设备的符号连接名称；
-o或--timers：显示计时器；
-p或--programs：显示正在使用Socket的程序识别码和程序名称；
-r或--route：显示Routing Table；
-s或--statistice：显示网络工作信息统计表；
-t或--tcp：显示TCP传输协议的连线状况；
-u或--udp：显示UDP传输协议的连线状况；
-v或--verbose：显示指令执行过程；
-V或--version：显示版本信息；
-w或--raw：显示RAW传输协议的连线状况；
-x或--unix：此参数的效果和指定"-A unix"参数相同；
--ip或--inet：此参数的效果和指定"-A inet"参数相同。
```

```
常用。

netstat -anp | grep [port]

netstat -tunlp | grep [port]

# 参数说明
 -t (tcp) 仅显示tcp相关选项
 -u (udp)仅显示udp相关选项
 -n 拒绝显示别名，能显示数字的全部转化为数字
 -l 仅列出在Listen(监听)的服务状态
 -p 显示建立相关链接的程序名
```

## 记录其他一些常用的命令

结束响应进程
kill -9 <pid>
> kill -9 30123

## 操作

解压
> tar -zxvf mongodb-osx-ssl-x86_64-4.0.2.tgz

pwd 查看当前位置
> pwd

修改权限
> chmod 777 -R data

## 网络
连接 curl
> curl -v 127.0.0.1

### refer
- http://man.linuxde.net/lsof
- http://man.linuxde.net/netstat