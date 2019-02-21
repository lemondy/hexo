title: C# Mysql连接问题记录
date: 2019/02/10
tags: [C#, mysql]

---

最近代码中有需要通过 C# 来连接 MySql 数据库，但是在使用过程中遇到了一些问题，现在记录如下，以备后面再遇到有个查找的地方。

> 其实连接 Mysql 有问题出现了两次，第一次想着下次再出现该怎么怎办，而到了再次遇到问题，第一次很多细节已经想不起来了。好记性不如烂笔头确实是有道理的。

## 环境说明

操作系统： windows 10 64位  
编程语言环境：C#、visual studio enterprise 2017  
Mysql版本：Mysql 8.0.12

## 前置条件准备
为了在 C# 中连接 Mysql，需要提前对相关环境进行配置。需要先下载两份文件。  

```
1. [c#mysql-connector-dll.rar](https://pan.baidu.com/s/1O0rEDoBNZsHbj2vWksNFOw) 提取码：0061   
2. [mysql-connector-net-8.0.15](https://pan.baidu.com/s/1F5kwi6vBMeLcelfAhUVRxA) 提取码：6evg 

```

上面的第一个压缩文件中的dll 是用来在 C# 程序中调用 Mysql相关接口来操作数据库的。 按照如下的操作就可以在 C#程序中使用 Mysql 接口了。  
![添加引用](images/c#-mysql/add_reference.png)  
![选择dll文件](images/c#-mysql/choose_dll.png)

第二个可执行文件是mysql 官方发布的插件，供 C# 连接Mysql 使用。 直接安装就好。
如果上面的执行完毕后还是不能很好的连接，可以试着将上面的 dll 文件都拷贝到系统级别目录下。我的电脑是64位，我就讲这些文件拷贝到目录：

```
C:\Windows\SysWOW64
```

## 坑点1：`caching_sha2_passwd`

在 Mysql 8.0 中用户密码默认加密算法由原先的 `mysql_native_password` 变成了现在的 `caching_sha2_password`算法。可以使用 sql 语句查看当前mysql中的配置情况：

```
    select host,user,plugin from mysql.user;
```

如果访问 mysql 出现如下的情况，那么基本上就是这个问题：

```
<font color=red>Authentication method 'caching_sha2_password' not supported by any of the available plugins.</font>
```

针对这个问题有如下解决办法：
1. 重置身份认证插件位以前的方式（或者使用低版本的mysql），修改mysql配置文件，windows系统一般默认安装在 `C:\ProgramData` 路径下，但是这个目录默认是隐藏的。 

> [mysqld]  
> default_authentication_plugin=mysql_native_password

该设置允许8.0之前的客户端连接到8.0服务器，但是，该设置应被视为临时设置，而不是长期或永久性解决方案，因为它会导致使用有效设置创建的新帐户放弃提供的改进的身份验证安全性 caching_sha2_password。

上面配置文件修改后，需要重启mysql服务。

2. 将某个用户的身份认证改为 mysql_native_password

执行如下的sql 语句：

```ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
```

上面 BY 后面是root 的登录密码。

## 参考文献

1. [详解关于MySQL 8.0走过的坑](https://www.jb51.net/article/148068.htm)