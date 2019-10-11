

title: memcached 入门到精通
date: 2019-09-30 11:57:50

tags: [memcache, 分布式缓存, KV存储]
---

Memcached 是高性能、分布式的内存对象缓存系统（Memory Object Cache System）；是一种典型的 C/S 架构服务器，对外提供基于 key-value (键/值)映射的快速数据存储和检索服务。

<!-- more -->

**Memcached 采用基于文档的协议，使用空格作为分隔符，使用\r\n作为结尾符。通过telnet可手动访问存储在memcached中的数据。**

Memcached 既能提供基于 TCP 的服务，也能提供基于UDP的服务。该如何选择呢？文档给出的建议是：当客户端并发请求过多，TCP 连接成瓶颈时，可以考虑采用 UDP 接口。众所周知，UDP协议不会保证传输的正确性，因此 UDP 接口只能用于GET 这样并不一定需要成功的操作，即使失败，也只会导致 cache 未命中，影响不会很大。

> 备注：在memcached启动的时候，通过参数-p（TCP端口号）/-U（UDP端口号）区分



### memcached安装配置

安装和配置可以参看：[Memcached安装及配置](https://blog.csdn.net/wlddhj/article/details/84321220)

启动示例

```shell
memcached -l 10.224.86.41 -p 8899 -m 1024 -k -d -P /tmp/memcached.pid
```

参数说明如下：

| 参数名                                                       | 解释                                                 |
| :----------------------------------------------------------- | :--------------------------------------------------- |
| -p <num> TCP port number to listen on (default: 11211)       | TCP协议监听端口号 默认：11211                        |
| -U <num> UDP port number to listen on (default: 0, off)      | UDP监听端口 默认不打开                               |
| -s <file> unix socket path to listen on (disables network support) |                                                      |
| -a <mask> access mask for unix socket, in octal (default 0700) | 访问权限 unix socket访问权限 默认0700                |
| -l <ip_addr> interface to listen on, default is INDRR_ANY    | 监听地址 默认INDRR_ANY                               |
| -d run as a daemon                                           | 以daemon进程运行                                     |
| -r maximize core file limit                                  | 核心文件极限最大值                                   |
| -u <username> assume identity of <username> (only when run as root) | 用户名，访问标示 一般以root运行                      |
| -m <num> max memory to use for items in megabytes, default is 64 MB | 内存池可使用的最大内存值 单位MB，默认64              |
| -M return error on memory exhausted (rather than removing items) | 内存耗尽，返回异常 不删除数据项                      |
| -c <num> max simultaneous connections, default is 1024       | 最大并发连接数 默认1024个                            |
| -k lock down all paged memory                                | 将申请的内存锁定，防止被交换到磁盘                   |
| -v verbose (print errors/warnings while in event loop)       | 冗余度 当 会打印error/warning                        |
| -vv very verbose (also print client commands/reponses)       |                                                      |
| -h print this help and exit                                  | 帮助信息                                             |
| -i print memcached and libevent license                      | 打印memcached 和libevent的执照信息                   |
| -b run a managed instanced (mnemonic: buckets)               | 运行管理实例，如buckets                              |
| -P <file> save PID in <file>, only used with -d option       | 将pid写入指定文件，以备 kill 使用； 与-d参数联合使用 |
| -f <factor> chunk size growth factor, default 1.25           | chunk 大小增长倍数 默认1.25                          |
| -n <bytes> minimum space allocated for key+value+flags, default 48 | 为key + value + flag分配的最小变长单位byte，默认48   |



### 功能详情

#### 连接

使用 telnet 连接到memcached： `telnet 127.0.0.1 11211`

#### 基本命令

```
格式：command <key> <flags> <expiration time> <bytes>
      <value>
```

各个参数说明如下：

| 参数            | 用法                                                         |
| :-------------- | :----------------------------------------------------------- |
| value           | 存储的值（始终位于第二行）                                   |
| key             | key 用于查找缓存值                                           |
| flags           | 可以包括键值对的整型参数，客户机使用它存储关于键值对的额外信息 |
| expiration time | 在缓存中保存键值对的时间长度（以秒为单位，0 表示永远）       |
| command         | 命令，包括：set、get、replace、add、delete                   |
| bytes           | 在缓存中存储的字节点                                         |

下面将详细说明各个命令的使用

1. **set：**用于向缓存添加新的键值对。如果键已经存在，则之前的值将被替换。使用 `set` 命令正确设定了键值对，服务器将使用单词 **STORED** 进行响应。需要输入第一行的命令和第二行的value

   ```
   set uuid 0 3600 5
   12345
   STORED
   ```

2. **get：**用于检索与之前添加的键值对相关的值。您将使用 get 执行大多数检索操作

   ```
   set uuid 0 3600 5
   12345
   STORED
   ```
   ```
   get uuid
   VALUE uuid 0 5
   12345
   END

   ```

3. **replace：**仅当键已经存在时，`replace` 命令才会替换缓存中的键。如果缓存中不存在键，那么您将从 memcached 服务器接受到一条 **NOT_STORED** 响应。

   ```
   replace uuid 0 3600 6
   123456
   STORED
     
   replace uuid2 0 3600 7
   1234567
   NOT_STORED
   ```

4. **delete：**用于删除 memcached 中的任何现有值。您将使用一个键调用 delete，如果该键存在于缓存中，则删除该值。如果不存在，则返回一条 NOT_FOUND 消息。 

   ```
   delete uuid2
   NOT_FOUND
     
   delete uuid
   DELETED
   ```

5. add：仅当缓存中不存在键时，add 命令才会向缓存中添加一个键值对。如果缓存中已经存在键，则之前的值将仍然保持相同，并且您将获得响应NOT_STORED

   ```
   add uuid 0 3600 5
   12345
   NOT_STORED
    
   add uuid2 0 3600 7
   0987654
   STORED
   ```

6. **gets：**`gets` 命令的功能类似于基本的 `get` 命令。两个命令之间的差异在于，`gets` 返回的信息稍微多一些：64 位的整型值非常像名称/值对的 “版本” 标识符

   ```
   gets uuid
   VALUE uuid 0 5 4
   12345
   END

   replace uuid 0 3600 6
   123456
   STORED

   gets uuid
   VALUE uuid 0 6 6
   123456
   END
   ```

7. **cas：**用于设置名称/值对的值（如果该名称/值对在您上次执行 `gets` 后没有更新过）。它使用与 `set`命令相类似的语法，但包括一个额外的值：`gets` 返回的额外值

   ```
   set uuid 0 0 5
   12345
   STORED
    
   gets uuid      
   VALUE uuid 0 5 8
   12345
   END
    
   cas uuid 0 0 5 8
   23456
   STORED
    
    
   gets uuid
   VALUE uuid 0 5 9
   23456
   END
   ```

8. **stats：**转储所连接的 memcached 实例的当前统计数据

9. **flush_all：**仅用于清理缓存中的所有名称/值对。如果您需要将缓存重置到干净的状态，则 `flush_all` 能提供很大的用处

   ```
   set uuid 0 3600 5
   12345
   STORED
     
   get uuid
   VALUE uuid 0 5
   12345
   END
     
   flush_all
     
   get uuid
   END
   ```
   