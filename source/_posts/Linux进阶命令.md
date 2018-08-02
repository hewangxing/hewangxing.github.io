---
title: Linux进阶命令
date: 2018-05-05 23:50:46
categories: 编程
tags: linux
comments: true 
---

(1) find  
作用: 在Linux文件系统中，用于查找某个文件所在绝对路径。

    find 搜索路径 -name "文件名" //这里的搜索路径(绝对路径)越详细越好，可以加快查找的速度。
    //例如: find /home -name "hello.c"
    

(2) grep  
作用: 用于查找某个符号在某些文件中(比如某个单词)的位置。

    grep 参数 "符号名称" 某个文件或文件夹 
    例如：grep -rn "main" * //-r 递归查找、-n 打印行号、* 当前目录
                           //表示在当前目录递归查找main这个符号出现的位置，并打印行号
    

(3) which/whereis  
作用：查找某个命令或者应用程序(二进制文件)的位置。

    which/whereis 命令或应用程序名
    //区别: which只打印二进制文件所在的路径，而whereis打印二进制文件、源码和man手册的路径，更详细一些!
    

(4) uname  
作用: 打印系统信息。

    uname -a //打印系统的所有信息，包括内核版本，日期，操作系统等
    uname -r //打印内核版本的信息
    uname -p //打印CPU的信息
    uname 或者 uname -s //打印内核名字，比如Linux
    

(5) mount/umount 作用：挂载/卸载磁盘到Linux文件系统的某个目录上, 以便对磁盘的进行访问。

(6) shutdown/init  
作用：关机、重启Linux系统。

    shutdown -h now //立即关机
    shutdown -r now //立即重启
    init 0 //关机
    init 2 //重启
    

(7) chmod/chown/chgrp  
作用：对文件的权限(读写、执行、拥有者等)进行管理。

    chmod (change mode, 修改文件的读写、执行权限)   
    chown (change owner, 修改文件的拥有者)  
    chgrp (change group, 修改文件的拥有者所在的组)  
    

chmod对文件的读写、执行权限进行修改有两种方式：

① 用数字编码的方式修改文件的所有权限:

    chmod 编码值 文件名  
    'r' 可读   4  
    'w' 可写   2  
    'x' 可执行 1  
    '-' 无权限 0  
    //例如 chmod 777 文件名 可修改权限值为-rwxrwxrwx
    

② 只修改文件的部分权限

    -rwxr-xr-x   //文件的拥有者u(user)、文件拥有者所在的组g(group)、其它用户o(other)，+增加权限，-减少权限 
    例如：chmod u+x 文件名 //让用户的拥有者具有可执行权限
    

chown/chgrp对文件的拥有者和所在的组进行修改：

    chown 拥有者 文件名 chgrp 拥有者所在的组 用户名
    

(8) tree  
作用：可以很直观的打印出文件/目录的树形结构，不是Linux自带的命令，需要下载安装才能使用。

    tree 文件/目录 //tree默认打印当前的文件/目录的树形结构
    

(9) tar  
作用: 对文件夹打包和解压缩包。

    tar czvf abc.tar.gz abc/   //将abc文件夹打包成abc.tar.gz
    tar xzvf abc.tar.gz        //将abc.tar.gz解压到当前目录
    tar cjvf abc.tar.bz2 abc/  //将abc文件夹打包成abc.tar.bz2
    tar xjvf abc.tar.bz2       //将abc.tar.bz2解压到当前目录