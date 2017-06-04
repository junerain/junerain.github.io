---
layout: post
title: "sshpass source install on aix"
keywords: "sshpass,aix,source"
description: ""
category: AIX
tags: [AIX]
---
{% include JB/setup %}
## 1、源码下载:
```bash
wget https://nchc.dl.sourceforge.net/project/sshpass/sshpass/1.06/sshpass-1.06.tar.gz
```

## 2、解压
```bash
gunzip sshpass-1.06.tar.gz
tar -xvf sshpass-1.06.tar
cd sshpass-1.06  
```

## 3、生成makefile
```bash
./configure CC=/usr/vacpp/bin//xlc
```
如果直接用
```bash
./configure
```
不能直接编译通过，会报错误
```bash
225_billmdb%make
        make  all-am
        source='main.c' object='main.o' libtool=no  DEPDIR=.deps depmode=xlc /bin/sh ./depcomp  cc -qlanglvl=extc89 -DHAVE_CONFIG_H -I.      -g -c -o main.o main.c
"main.c", line 54.42: 1506-209 (S) Character constants must end before the end of a line.
"main.c", line 54.20: 1506-076 (W) Character constant 't define posix_openpt' has more than 4 characters. No more than rightmost 4 characters are used.
"main.c", line 343.101: 1506-209 (S) Character constants must end before the end of a line.
"main.c", line 343.69: 1506-076 (W) Character constant 's controlling TTY is now closed' has more than 4 characters. No more than rightmost 4 characters are used.
"main.c", line 435.102: 1506-209 (S) Character constants must end before the end of a line.
"main.c", line 435.53: 1506-076 (W) Character constant 's good enough for matching "Password: ", though.' has more than 4 characters. No more than rightmost 4 characters are used.
"main.c", line 54.1: 1506-046 (S) Syntax error.
make: The error code from the last command is 1.


Stop.
make: The error code from the last command is 2.


Stop.
```
因为默认的aix的c编译器为cc，而cc对于//\*注释的方法不认，需要全部删除掉这种

##  4、make
```bash
make
```
## 5、产生的问题
```bash
225_billmdb%make
        make  all-am
        source='main.c' object='main.o' libtool=no  DEPDIR=.deps depmode=xlc /bin/sh ./depcomp  /usr/vacpp/bin//xlc -DHAVE_CONFIG_H -I.      -g -c -o main.o main.c
        /usr/vacpp/bin//xlc  -g   -o sshpass main.o
ld: 0711-317 ERROR: Undefined symbol: .rpl_malloc
ld: 0711-345 Use the -bloadmap or -bnoquiet option to obtain more information.
make: The error code from the last command is 8.


Stop.
make: The error code from the last command is 2.


Stop.
```
需要将在configure 生产的config.h文件中注释掉或者删掉

```c
/* Define to rpl_malloc if the replacement function should be used. */
~~#define malloc rpl_malloc~~
```

## 6、重新make，大功告成
```
225_billmdb%sshpass
Usage: sshpass [-f|-d|-p|-e] [-hV] command parameters
   -f filename   Take password to use from file
   -d number     Use number as file descriptor for getting password
   -p password   Provide password as argument (security unwise)
   -e            Password is passed as env-var "SSHPASS"
   With no parameters - password will be taken from stdin

   -P prompt     Which string should sshpass search for to detect a password prompt
   -v            Be verbose about what you're doing
   -h            Show help (this screen)
   -V            Print version information
At most one of -f, -d, -p or -e should be used

```
