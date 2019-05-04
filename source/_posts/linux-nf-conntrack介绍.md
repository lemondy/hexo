title: linux nf_conntrack介绍
date: 2019-02-20 22:25:42
tags: [linux, nf_conntrack, dropping packet, 运维]
---

在 linux 上部署的高并发高性能的服务，在系统流量增长的过程中，经常会遇到”三大新手怪“：
1. nf_conntrack：table full，dropping packet；
2. fd被打满出现 too many open files错误；
3. IP临时端口不足，TIME_WAIT 状态连接过多导致无法建立新连接

今天主要谈谈nf_conntrack 这个问题。当出现 table full， dropping packet 的问题时，系统直接对网络中接受的数据包直接丢弃，也就无法处理正常的请求。其他的两个问题可以参看后续文章。

<!-- more -->

## 查看问题的方法

使用 `dmesg` 命令或者以root账户登陆查看文件 `/var/log/messages`，都会看到大量的信息：
```
kernel: nf_conntrack: table full, dropping packet.
```

## 解决办法

### 方法一：以 root 账户修改系统内核中的参数，增大nf_conntrack 相关的hashtable的大小（下面的取值是各示例，具体多少参看你系统当前的取值，然后看你请求的流量进行评估最大值）

```
vim /etc/sysctl.conf
#加大 ip_conntrack_max 值，这个会增大内存开销
net.ipv4.ip_conntrack_max = 393216
net.ipv4.netfilter.ip_conntrack_max = 393216
#降低 ip_conntrack timeout时间，记录连接保存的时间
net.ipv4.netfilter.ip_conntrack_tcp_timeout_established = 300
net.ipv4.netfilter.ip_conntrack_tcp_timeout_time_wait = 120
net.ipv4.netfilter.ip_conntrack_tcp_timeout_close_wait = 60
net.ipv4.netfilter.ip_conntrack_tcp_timeout_fin_wait = 120
```
参数修改完后，使用 `sysctl -p` 进行生效配置。

这种设置会存在两个问题：
1. iptables 重启后，ip_conntrack_max 这个参数又回恢复到默认的 65535，需要再次执行 `sysctl -p`，很容易忘记执行；
2. 如果服务端的流量还会继续增长，那么服务端还是有可能会再次出现 table full dropping packet的问题。

### 方法二：使用Raw 表， 跳过记录模块
使用raw表，会减少记录连接的次数。具体操作办法如下：

1. 修改 `/etc/sysconfig/iptables` 文件中的 `-A RH-Firewall-1-INPUT -m state --state RELATED,ESTABLISHED, <font color=red>UNTRACKED</font> -j ACCEPT` 行。增加其中的 **UNTRACKED** 标志，保存并restart iptables。

```
service iptables stop
service iptables start
```

2. 运行下面的语句：

```
iptables -t raw -A PREROUTING -p tcp -m tcp --dport 80 -j NOTRACK
iptables -t raw -A OUTPUT -p tcp -m tcp --sport 80 -j NOTRACK
iptables -t raw -A PREROUTING -p tcp -m tcp --sport 80 -j NOTRACK
iptables -t raw -A OUTPUT -p tcp -m tcp --dport 80 -j NOTRACK
```
> **第1步很重要，如果第1处没改，执行后面的语句会造成相应的端口不能访问。**
> **因为设置成不跟踪的连接无法拿到状态，可能会导致keep-alive用不了**

我使用该方法时，就因为没有执行第一步的操作，造成web访问不能使用。
需要特别特别注意的是，被 NOTRACK 的包无法匹配上依赖特定状态的其他规则。比如 INPUT 链上假如有这么一条规则
```
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT
```
那么在增加了上面的 `NOTRACK` 规则后，你会发现这个端口的包都被丢掉了。因为没有了连接状态跟踪，那个 `--state NEW `不可能匹配得上。所以要千万小心，一个很好的解决办法是加上这么一条规则
 
```
iptables -A INPUT -m state --state UNTRACKED -j ACCEPT
```

### 方法三：在用不着防火墙的情况（比如只能在内网访问），不用NAT转发的服务器，可以考虑把防火墙关闭
防火墙关闭的办法比较简单,`sysctl -a`里就没有netfilter相关的参数了

```
# CentOS 7.x
sudo systemctl stop firewalld
sudo systemctl disable firewalld
 
# CentOS 6.x
sudo service iptables stop
# 网上有些文章说关了iptables之后，用 iptables -L -n 之类查看规则也会导致nf_conntrack重新加载，实测并不会
sudo chkconfig --del iptables
```

### 方法四：禁用iptables相关模块(用之前考虑是否真的需要禁用)

1. 先将 `/etc/sysconfig/iptables` 中包含state的语句移除，并restart iptables   
2. 执行语句：
```
modprobe -r xt_NOTRACK nf_conntrack_netbios_ns nf_conntrack_ipv4 xt_state
modprobe -r nf_conntrack
```


## 参考资料
1. https://blog.csdn.net/houzhizhen/article/details/79231082
2. https://testerhome.com/topics/7509
3. [iptables raw 表介绍](https://blog.csdn.net/yanggd1987/article/details/54924725)
