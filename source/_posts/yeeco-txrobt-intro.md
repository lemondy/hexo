title: yeeco 公链自动交易机器人介绍
date: 2019-06-03 22:41:55

tags: [yeeco, 公链, 交易机器人]
---

在 yeeco 的 github 仓库中，在 `tests/txbot/main.go` 代码中提供了一个自动交易的机器人示例。本篇文章介绍下自动交易机器人的实现原理。

<!-- more -->

### 概览

自动交易机器按照如下的步骤执行操作：

1. 解析命令行中传递的参数。主要包括log日志存储地址、本地节点ip地址和端口、dht的端口、本地账户的密码、一个批次涉及到的交易次数和每一批交易之间的时间间隔；
2. 从本地加载本地节点的相关数据，例如交易高度信息等；
3. 加载本地的账户信息；
4. 创建p2p 网络节点同时启动节点
5. 搜索p2p 网络中的节点，直到p2p网络连接成功；
6. 启动routine 发送交易数据；
7. 如果收到 ctrl+c 就退出程序，否则循环的执行交易转账操作+sleep操作

其中上面说了从本地加载相关数据，yeeco 公链默认会在 `～/Library/YeeChain`这个路径下存放公链相关的数据。

执行的流程图如下所示：

<div align=center>![txbot flowchart](/images/yeeco/txbot.png)

### 执行流程与代码关联

主要分享交易相关的操作，其他函数是初始化相关操作。交易转账的逻辑在 `genTxs` 函数中，这个函数定义如下

```go
func genTxs(n *node.Node, signers []crypto.Signer, addrs []common.Address,
    batchCnt int, batchInterval time.Duration, resetCount int,
    quitCh chan struct{}, wg sync.WaitGroup)
```

#### 初始化操作
在 genTxs 函数开始我们可以看到会进行如下的初始化操作，在进行转账操作之前，需要先获取链的 ID，构造一个 nonce 数组，其大小等于账户的个数。

在进行转账操作前，触发链上交易块的同步进程，将交易块同步到本地。

```go
    c := n.Core()
    chainID := uint32(c.Chain().ChainID())
    nonces := make([]uint64, len(signers))

    // trigger chain sync before start
    c.TriggerSync()

    // wait before sending txs
    time.Sleep(20 * time.Second)
```

#### 重置账户的nonce
一个账户的交易是按照nonce的顺序进行打包的，对于已经打包的nonce发送到网络中是不会被处理。而nonce过大的交易块也不会处理，直到nonce前面的交易块都被处理了，较大的nonce交易块才会被打包处理。

在机器人自动交易中，偶尔会出现交易失败，交易块丢失的情况，如果不重置nonce，之后的交易就会一直不会被打包。

```go
var nonceResetFunc = func() {
        log.Info("nonce reset start")
        c.TriggerSync()
        func() {
            var ticker = time.NewTicker(time.Second)
            defer ticker.Stop()
            for {
                select {
                case <-ticker.C:
                }
                if c.IsSyncing() {
                    log.Info("wait for sync")
                } else {
                    log.Info("sync finished")
                    return
                }
            }
        }()
        newNonces := make([]uint64, len(signers))
        for i, addr := range addrs {
            // 获取本地地址当前nonce值，充值发送交易中的nonce
            account := c.Chain().LastBlock().GetAccount(addr)
            if account == nil {
                newNonces[i] = 0
            } else {
                newNonces[i] = account.Nonce()
            }
        }
        var sb = new(strings.Builder)
        for i, oldN := range nonces {
            newN := newNonces[i]
            if oldN != newN {
                _, err := fmt.Fprintf(sb, "%d:[%d -> %d]", i, oldN, newN)
                if err != nil {
                    log.Error("format nonce reset change", err)
                }
            }
        }
        log.Info("nonce reset result", sb.String())
        nonces = newNonces
    }
```

#### 批量转账操作

批量转账操作就是两层 for 循环，遍历本地账户。然后调用 `NewTransaction` 生成交易块，调用 `TxBroadcast` 将交易块发送到网络中确认。

```go
for {
        // batch transfer
        for j, toAddr := range addrs {
            for i, signer := range signers {
                if j == i {
                    continue
                }
                // reset inMem nonce if needed
                if totalTxs%resetCount == 0 {
                    nonceResetFunc()
                }
                // batch pause if needed
                if totalTxs%batchCnt == 0 {
                    log.Info("Total txs sent", totalTxs)
                    // batch reached
                    select {
                    case <-quitCh:
                        ticker.Stop()
                        break Exit
                    case <-ticker.C:
                        // send txs
                    }
                }
                // 调用交易方法形成交易块
                tx := core.NewTransaction(chainID, nonces[i], &toAddr, big.NewInt(100))
                if err := tx.Sign(signer); err != nil {
                    log.Error("tx sign failed", "err", err)
                    continue
                }
                // 交易信息发送到网络中
                if err := c.TxBroadcast(tx); err != nil {
                    log.Error("tx broadcast failed", "err", err)
                    continue
                }
                nonces[i]++
                totalTxs++
            }
        }
    }
    log.Info("Total txs sent", totalTxs)
```

### 交易机器人的编译

修改 Makefile 中的配置，即可进行编译。我是直接复用原先的 `Makefile` 的方式，在 `cmd` 目录下新建了一个 txbot 文件夹，将原先 `tests/txbot/main.go` 拷贝到 `cmd/txbot/` 目录下，然后修改 `Makefile`，之后就可以正常的编译通过了。 

```bash
.PHONY: all
all: bootnode gyee txbot
    @echo "Done building all"

#
# CMD targets
#

.PHONY: bootnode
bootnode: ${OUT_BIN}/bootnode

.PHONY: gyee
gyee: ${OUT_BIN}/gyee

.PHONY: txbot
txbot: ${OUT_BIN}/txbot

${OUT_BIN}/%: env
    @mkdir -p '${OUT_BIN}'
    go build -o $@ ${GO_LD_FLAGS} ./cmd/$(@F)
    @echo "Done building cmd $(@F)"
```
#### 运行
在编译通过后，可以在 `build/bin/txbot` 这个路径下找到可执行程序。我们就可以使用如下的命令启动期总交易。注意要最后面的 `password` 替换成你本地账户的密码。

```
./txbot -batch=100 -batchTime=1000 -password=password
```
最后，正常的情况下 `go` 支持交叉编译，即可以在 `mac` 下编译产出适合 `linux` 运行的可执行程序，但是由于 `yeeco` 中依赖了 `cgo` 代码，因此交叉编译不可行，或者说不能简单修改下 `GOOS` 就成功的。