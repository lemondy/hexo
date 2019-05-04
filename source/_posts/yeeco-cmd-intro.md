title: yeeco公链命令行使用介绍
date: 2019-04-29 23:25:43
tags: [yeeco, 公链, 命令行]
reward: true
---
本文主要介绍的是 yeeco 公链命令行接口命令的使用方式，同时介绍各个参数的相关含义。

<!-- more -->

公链代码中提供了两个命令行工具，`bootnode` 和 `gyee`。其中 bootnode 工具中我们可以生成一个节点的公钥私钥，生成一个网络中一个节点。可以指定在公链中的 chainID 接入网络和 dht 进行存储的读写。而 gyee 工具就是接入公链网络，可以对公链中的账户进行管理，账户的资金相关操作，比如余额查询，转账等相关操作。
 
# bootnode 介绍

参数列表如下：

```
Usage of ./bootnode:
  -cip string  链的ip地址
        chain ip(b1.b2.b3.b4)
  -cport int   链的端口
        chain port
  -dip string  dht的ip
        dht ip(b1.b2.b3.b4)
  -dport int   dth的端口
        dht port
  -genkey bool 是否生成key   
        generate node key to file
  -nodeDataDir string  node数据存放路径
        node data directory
  -nodeName string     node的名称
        node name
  -writenodeid bool    是否显示nodeid
        write out the node's id and quit
```

```
# 生成私钥，存放在 data/test/nodekey 里面
./bootnode -genkey=true -nodeDataDir=./data -nodeName=test
```

```
# 查看nodeId
./bootnode -writenodeid=true -nodeDataDir=./data -nodeName=test
```

```
# 启动 bootnode
./bootnode -cip=127.0.0.1 -cport=8989 -dip=127.0.0.1 -dport=8808 -nodeDataDir=./data -nodeName=test
```

# gyee 介绍

参数列表及说明

```
NAME:
   gyee - The gyee command line interface

USAGE:
   gyee [global options] command [command options] [arguments...]

COMMANDS:
     help, h  Shows a list of commands or help for one command
   ACCOUNT COMMANDS:
     account  Manage accounts
   CONFIG COMMANDS:
     config  Manage config
   CONSOLE COMMANDS:
     attach   Start an interactive JavaScript console to running node
     console  Start an interactive JavaScript console
   MISC COMMANDS:
     license  Display license information
     version  Print version numbers

GLOBAL OPTIONS:
   --bootnode value            boot node
   --chainid value             chain id (default: 0)
   --coinbase value            coinbase address for node
   --config FILE, -c FILE      load configuration from FILE
   --crash_report_url value    crash report url
   --datadir value             chain data dir
   --enable_crash_report       enable crash report
   --genesis value             genesis file path
   --http_listen value         http listen
   --ipcpath value             ipc path
   --keydir value              key dir
   --logfile value             log file
   --loglevel value            log level
   --metrics_enable            metrics enable
   --metrics_report            metrics enable report
   --metrics_report_url value  metrics report url
   --mine                      mine
   --nodedir value, -d value   gyee node root directory
   --nodename value            gyee node name (default: "MyNode")
   --p2p_listen value          p2p netowrk listen port
   --pwdfile value             pwdfile for coinbase keystore
   --rpc_listen value          rpc listen
   --testnet, -t               test network: pre-configured test network
   --help, -h                  show help

COPYRIGHT:
   Copyright 2017-2018 The gyee Authors
```
