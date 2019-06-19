title: yeeco公链测试Day1
date: 2019-05-24 10:03:49
tags: [yeeco, 公链, 测试]
---

本次测试的主要是程序的运行和账户的创建相关。

<!-- more -->

### 操作步骤

1. 下载最新版本gyee客户端程序
2. 打开两个terminal，在第一个 terminal 中执行 `./gyee` 启动 gyee 客户端节点程序；在第二个 terminal 中执行 `./gyee attach` 命令，就可以在一个交互式的prompt，在这个命令窗口里面可以创建账户，查看交易，查看块信息，进行交易，账户冻结等操作。

### gyee 控制台中操作的命令

#### 成功执行的命令

1. bridge.getLastBlock() 查看本地链路最高块信息
2. bridge.getBlockByHeight(height) 查看某个高度的块信息
3. bridge.getBlockByHash('hash_addr') 查看某个hash地址的块信息
4. bridge.newAccount() 创建本地节点地址，之后需要输入密码，在控制台中输入字符是不可见的
5. bridge.Accounts() 列出本地所有账户地址

#### 未成功执行的命令

由于创建的账户地址是本地地址，需要在网络中发起一笔转入本地地址的交易才能激活本地地址，下面的几个命令暂时未执行成功

1. bridge.getAccountstate('addr') 查看地址状态，会显示当前账户地址、账户余额、nonce值
2. bridge.lockAccount('addr') 锁定地址
3. bridge.unlockAccount('addr') 解锁地址，解锁后才可以进行转账

由于我没有找到 `sync finished` 交易信息，暂时没有查看交易相关的记录。

程序运行正常，未出现异常崩溃退出的情况。

#### 测试中看到的问题

1. `tx nonce too far` ：内部人员解释：压力测试机器人校准状态，重发用户的nonce
2. `err core.chain: block signature mismatch` ：内部人员解释：网络中混入了老的验证节点的数据，数据发不到当前网络中，导致验证失败
3. `err block too far for chain head` ：内部人员解释：别人广播的新块离本地数据太远被忽略的log

