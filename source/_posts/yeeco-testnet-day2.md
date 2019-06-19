title: yeeco 公链测试Day2
date: 2019-05-27 22:00:29

tags: [yeeco, 公链, 测试网]
---

本篇文章会介绍 yeeco 公链转账相关操作以及一些输出的日志说明。

<!-- more -->

## 转账操作
在 [公链测试Day1](http://lemondy.github.io/2019/05/24/yeeco-testnet-test/) 文章中已经说了我们需要打开两个 `terminal` 终端窗口，在 yeeco 客户端所在的路径下，依次在各个窗口中执行如下两个命令

```shell
./gyee
```

```shell
./gyee attach
```

接下来就可以在attach 窗口中进行交互了。转账按照如下命令操作。

```js
// 1. 查看账户地址
> bridge.Accounts()
addresses:"01058d61de374215a6a067dc17165d948ec7101622805b0cf228"

// 2. 解锁账户，需要输入密码
> bridge.unlockAccount('01058d61de374215a6a067dc17165d948ec7101622805b0cf228')

// 3. 查看当前账户nonce值
> bridge.getAccountState('01058d61de374215a6a067dc17165d948ec7101622805b0cf228')
address:"8d61de374215a6a067dc17165d948ec710162280" nonce:23 balance:"999999999999999999637"

// 4. 发起转账操作，会打印出交易的Tx hash值
> bridge.sendTransaction({from:'01058d61de374215a6a067dc17165d948ec7101622805b0cf228', to:'0105d14510b15ddf8ed7de50f943920ebf85f22f8e3845b15fc9', amount:'10', nonce:23})
hash:"4dfce2809d047d85e6985e643d111e07e0248dd7e747d814e6da6cd13158466d"

// 5. 查询交易信息，一般几分钟内就可以查询的交易块的信息（在块同步进度和链上一致，并且地址相关参数无误的情况)
> bridge.getTxByHash('4dfce2809d047d85e6985e643d111e07e0248dd7e747d814e6da6cd13158466d')
hash:"4dfce2809d047d85e6985e643d111e07e0248dd7e747d814e6da6cd13158466d" nonce:23 from:"8d61de374215a6a067dc17165d948ec710162280" recipient:"d14510b15ddf8ed7de50f943920ebf85f22f8e38" amount:"10"
```

为了执行转账相关操作，有几点需要注意：
1. 新创建的账户需要有一笔转入的交易才可以在网络中查询的到，否则改地址只是本地地址；
2. 转账前要解锁账户，一般解锁后账户会在几分钟后自动锁上，所以下次转账操作如果间隔较久需要再次输入密码；
3. nonce 很关键，创建账户成功后切账户有余额的第一转账可以不设置nonce，会默认置为0，后面每次转账nonce值应该设置为当前账户显示的nonce值。否则只有交易不会被确认，直到小于nonce值的交易都确认后才会打包。

## 控制台输出日志
执行 `./gyee` 命令后可以在控制台看到如下的滚动的信息：

1. 可以看到本地节点在启动的时候，会进行节点的启动，本进程的fd配置，区块链pool配置，dht配置，读取本地区块高度信息;
2. 本地节点会向 p2 p网络中发送广播，当看到 `chainReady4User: ok` 这个就说明 p2p 网络连接成功
3. 网络连接成功后，就主要执行的是两种操作：`sync` 同步交易块，`processBlock` 验证交易块

```php
INFO[2019-05-27T23:03:41+08:00] fdLimit raise 4864 -> 10240, max 10240
INFO[2019-05-27T23:03:41+08:00] Create new node[]
INFO[2019-05-27T23:03:41+08:00] Create new core[]
INFO[2019-05-27T23:03:41+08:00] Create New Blockchain[]
INFO[2019-05-27T23:03:41+08:00] Loaded local block[number 43168 Hash 5c8f78f507a0808227806779f234acbeb5f9929cae61bc669c83347635c9348b]
INFO[2019-05-27T23:03:41+08:00] Create New BlockPool[]
INFO[2019-05-27T23:03:41+08:00] Create New TransactionPool[]
INFO[2019-05-27T23:03:41+08:00] OsnServiceConfig: node[0.0.0.0:30303:30303]
INFO[2019-05-27T23:03:41+08:00] OsnServiceConfig: dht[0.0.0.0:40405]
INFO[2019-05-27T23:03:41+08:00] Node Start...[]
INFO[2019-05-27T23:03:41+08:00] Core Start...[]
INFO[2019-05-27T23:03:41+08:00] BlockPool Start...[]
INFO[2019-05-27T23:03:41+08:00] TransactionPool Start...[]
INFO[2019-05-27T23:03:41+08:00] Node Started[]
INFO[2019-05-27T23:03:41+08:00] P2pStart: what: 1, inst: chain_0
INFO[2019-05-27T23:03:41+08:00] P2pStart: what: 0, inst: chain_0
INFO[2019-05-27T23:03:41+08:00] p2p Started[]
INFO[2019-05-27T23:03:41+08:00] ActivePeerSnapshot: sdl: chain_0, actHis: [0 0 0 0 0 0 0 0]
INFO[2019-05-27T23:03:41+08:00] IPC Started[]
INFO[2019-05-27T23:03:41+08:00] Node Wait for shutdown...[]
INFO[2019-05-27T23:03:42+08:00] ActivePeerSnapshot: sdl: chain_0, actHis: [0 0 0 0 0 0 0 0]
INFO[2019-05-27T23:03:42+08:00] showRoute: sdl: chain_0, numberOfBucketNode: 1
INFO[2019-05-27T23:03:43+08:00] showRoute: sdl: chain_0, numberOfBucketNode: 2
INFO[2019-05-27T23:03:43+08:00] showRoute: sdl: chain_0, numberOfBucketNode: 3
INFO[2019-05-27T23:03:43+08:00] showRoute: sdl: chain_0, numberOfBucketNode: 4
INFO[2019-05-27T23:03:43+08:00] showRoute: sdl: chain_0, numberOfBucketNode: 5
INFO[2019-05-27T23:03:43+08:00] ActivePeerSnapshot: sdl: chain_0, actHis: [0 0 0 0 0 0 0 0]
INFO[2019-05-27T23:03:44+08:00] ActivePeerSnapshot: sdl: chain_0, actHis: [0 0 0 0 0 0 0 0]
INFO[2019-05-27T23:03:45+08:00] ActivePeerSnapshot: sdl: chain_0, actHis: [0 0 0 0 0 0 0 0]
INFO[2019-05-27T23:03:46+08:00] ActivePeerSnapshot: sdl: chain_0, actHis: [0 0 0 0 0 2 0 0]
INFO[2019-05-27T23:03:47+08:00] ActivePeerSnapshot: sdl: chain_0, actHis: [0 0 0 0 0 2 2 0]
INFO[2019-05-27T23:03:48+08:00] ActivePeerSnapshot: sdl: chain_0, actHis: [0 0 0 0 0 2 2 2]
INFO[2019-05-27T23:03:48+08:00] chainReady4User: ok, sdl: chain_0, actHis: [0 0 0 0 0 2 2 2]
WARN[2019-05-27T23:03:54+08:00] processBlock() verify fails[err block too far for chain head]
INFO[2019-05-27T23:03:54+08:00] [sync] block pool sync started[localH 43168]
WARN[2019-05-27T23:03:54+08:00] processBlock() verify fails[err block too far for chain head]
WARN[2019-05-27T23:03:54+08:00] processBlock() verify fails[err block too far for chain head]
INFO[2019-05-27T23:03:54+08:00] [sync] remote height[44026]
INFO[2019-05-27T23:03:54+08:00] [sync] got remote block[H 43169 txs 75 hash 38e259c76ac0d4818cc1bb2d7c0202dc47d87a1f6aa300d8df4faa460b3bc843]
INFO[2019-05-27T23:03:54+08:00] processVerifiedBlock[H 43169 hash 38e259c76ac0d4818cc1bb2d7c0202dc47d87a1f6aa300d8df4faa460b3bc843 sigCnt 4 sigs map[5f51e11cb4185a3986f971405a08b88818eb6f87: 813c4238885c5b4b700c35bb3429def0e454a3bb: cd974435e84abde36a90dd1f45e25c820f1e4085: df801dcb78a997d49d0ae042ce0b1320b2082bb3:]]
INFO[2019-05-27T23:03:54+08:00] signature count reached[H 43169 hash 38e259c76ac0d4818cc1bb2d7c0202dc47d87a1f6aa300d8df4faa460b3bc843 sCnt 4 vCnt 4]
INFO[2019-05-27T23:03:54+08:00] [sync] got remote block[H 43170 txs 461 hash 2fcf39f41ac2d7154235aeeb3f6c186d50fd4807e3bb3d2ae622a15ecdcd7d7b]
INFO[2019-05-27T23:03:54+08:00] Persisted trie from memory database[nodes 184 size 30591 time 657.911µs gcnodes 0 gcsize 0 gctime 0s livenodes 1 livesize 0]
INFO[2019-05-27T23:03:54+08:00] Persisted trie from memory database[nodes 0 size 0 time 870ns gcnodes 0 gcsize 0 gctime 0s livenodes 1 livesize 0]
INFO[2019-05-27T23:03:54+08:00] processVerifiedBlock[H 43170 hash 2fcf39f41ac2d7154235aeeb3f6c186d50fd4807e3bb3d2ae622a15ecdcd7d7b sigCnt 4 sigs map[5f51e11cb4185a3986f971405a08b88818eb6f87: 813c4238885c5b4b700c35bb3429def0e454a3bb: cd974435e84abde36a90dd1f45e25c820f1e4085: df801dcb78a997d49d0ae042ce0b1320b2082bb3:]]
INFO[2019-05-27T23:03:54+08:00] signature count reached[H 43170 hash 2fcf39f41ac2d7154235aeeb3f6c186d50fd4807e3bb3d2ae622a15ecdcd7d7b sCnt 4 vCnt 4]
INFO[2019-05-27T23:03:55+08:00] Persisted trie from memory database[nodes 791 size 90738 time 2.113478ms gcnodes 0 gcsize 0 gctime 0s livenodes 1 livesize 0]
INFO[2019-05-27T23:03:55+08:00] Persisted trie from memory database[nodes 0 size 0 time 901ns gcnodes 0 gcsize 0 gctime 0s livenodes 1 livesize 0]
INFO[2019-05-27T23:03:55+08:00] [sync] got remote block[H 43171 txs 369 hash fc1f090834156b09384da419ab3b9abacde3a74afd2560a132fe49ea5eff4b3b]
WARN[2019-05-27T23:03:55+08:00] processBlock() verify fails[err block too far for chain head]
```

## 总结
本次测试转账相关操作体验流程，没发现异常。程序运行十几个小时未出现异常崩溃的情况。由于是笔记本，长时间运行，笔记本会发烫，所以没有一直长时间不间断运行。

每次进行转账的时候，会启动 gyee 客户端，然后执行转账操作。
