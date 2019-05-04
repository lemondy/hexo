title: linux中fd相关知识
date: 2019-02-23 17:47:13
tags: [linux, fd, too many open files, 运维]
---

## fd 介绍
简言之：linux 系统中一切皆文件。不仅数据抽象为文件，其他一切操作和资源也都抽象为文件，比如硬件设备，socket，进程，线程，磁盘等。

在操作这些所谓的文件的时候，我们不可能没操作一次就要找一次名字吧，这样会耗费大量的时间和效率。咱们可以每一个文件操作一个索引，这样，要操作文件的时候，我们直接找到索引就可以对其进行操作了。我们将这个索引叫做文件描述符（file descriptor），简称fd，在系统里面是一个非负的整数。每打开或创建一个文件，内核就会向进程返回一个fd，第一个打开文件是0,第二个是1,依次递增。
子线程会共享父进程的fd资源

<!-- more -->

## fd 限制和修改
linux中的fd 限制主要有两种：系统级别和用户级别。

系统级别限制：默认限制为19595906，这个是限制整机总共的fd最大值。通常可以在 `/proc/sys/fs/file-max` 中配置这个参数；

用户级别限制：默认是 10240，这个是系统账户级别进程使用的fd上限，超过之后就会报 `fd open too many`， 无法继续创建新的fd。这个参数可以在 `/etc/security/limit.conf` 中修改。[参看 centos fd修改](https://blog.csdn.net/oklizy/article/details/48049353)

查看当前机器fd连接详情
先找到你想要查看进程的pid，然后使用如下的方式查看连接的fd数目。

```
ps -ef | grep ${proc_name}  
使用 ls -l /proc/$pid/fd 查看进程都打开了哪些fd。   
也可以用 lsof -c $proc_cmd | grep $pid 看到socket类fd的具体上下游信息。  

```
我通常使用 `lsof | grep ${current_machine_name}` 来查看连接到当前机器的fd信息，然后可以利用 awk分组统计下不同机器与本机的连接数目。

## 参考资料
1. https://www.jianshu.com/p/504a53c30c17
2. https://blog.csdn.net/oklizy/article/details/48049353
3. https://blog.csdn.net/mao0514/article/details/51273072
4. https://blog.csdn.net/zhjali123/article/details/72566685