title: yeeco公链代码编译篇
date: 2019-04-26 22:07:29
tags: [yeecall, 公链, 编译]
---
`YEECO` 主要由革命性的区块链平台 `YEECO BlockChain`、高效大吞吐量 `P2P` 网络 `YeeNet` 和编码分片存储网络 `CDHT` 这三个主要部分构成，新一代分布式互联网逻辑上可以表示为由无数个 `YEECO` 区块链平台构成的去中心化云计算平台，可以通过不断地扩充 `YEECO` 区块链平台和划分子网来动态扩容。本文主要记录的是 `yeeco` 公链编译中遇到的问题和解决办法方法。

> 上面简介摘抄自yeeco [白皮书](http://dcdn.yeecall.com/yee/YEECO_Technical_White_Paper_ZH_v0.1_Draft.pdf) 

<!-- more -->

# 环境准备

为了编译和开发 `go` 项目，我们需要提前做如下几件事：
1. `go`环境安装配置
2. IDE选择
3. 科学上网或者使用类似AWS EC2的开发机
4. `git`
5. `docker` [可选]

## go 环境安装配置

首先在 [go安装包官方网页](https://golang.org/dl/) 下载与你当前所使用系统对应的go开发包。go 安装的详细操作步骤可以参看 [go语言的安装和环境变量的配置](https://www.cnblogs.com/zhangym/p/5570169.html) 中介绍的。注意 <font color=red>**go 语言版本要求 1.12 及以上**</font>

> 我在mac os 中曾经已经使用 brew 安装了 go，结果安装的是 1.11 版本，最好是使用和 yeeco 代码中依赖的 go 版本一直，至少是不要低于该版本。

## IDE 选择

一款好的 IDE 能够一定程度上提高码代码的效率，让自己更快的发现代码中浅显问题（当然使用vim编码的牛逼程序猿不在此列）。
go语言常用的几款 IDE 如：`liteIDE`、`GoLand`、`VsCode`（安装go插件）、`eclipse`（go插件）和`sublime`（编辑器）。

IDE 看自己的喜好，我这里推荐使用`GoLand`，使用起来很方便。

## 科学上网或者国外开发机

由于 `go` 是谷歌家推出的，很多代码需要从谷歌的相关仓库中下载，而谷歌在国内基本上是用不了，所以需要科学上网或者国外的开发机。

对于 `go` 项目编译出现无法下载依赖代码的时候，还有一种工具：`goproxy`

## git

代码都是托管在git开源的，git是必须要安装的。安装教程可参考 [廖大神手把手教学入门git](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/)

## docker [可选]
`Docker` 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 `Linux` 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。【摘抄自百科】

由于 `yeeco` 的镜像会发布到 `docker` 仓库，因此如果想体验官方发布的程序，可以安装下 `docker`。

# 代码编译步骤

> 本操作步骤均是在 mac os 下完成的，不完全适用 windows。

## 下载源代码

```
cd /home/workspace
git clone https://github.com/yeeco/gyee.git
```

查看当前路径下面拥有的文件如下所示

![ls](/images/yeeco/ls.png)

## 编译

其中与编译相关的两个文件是：`Dockerfile` 和 `Makefile`，其中 `Dockerfile` 是用来编译生成 `docker` 镜像，`Makefile` 中说明具体的编译流程。

### 使用 docker 生成镜像的命令为：

```
docker image build -t yeeco/gyee .
```

编译过程中会出现如下的错误

![docker build](/images/yeeco/docker-build.png)

`docker` 编译中出现的问题主要是下载 `golang.org/x`、`google.golang.org/genproto` 和 `google.golang.org/grpc` 库的代码会出现超时的情况。主要原因是在 `docker` 的虚拟容器中无法访问谷歌的相关代码库。当前我也尝试过在 `Dockerfile` 中将本地的这些已经下载好的库代码给 `COPY` 到虚拟容器中，但是并没有成功。由于我们无需向 `docker` 仓库中发布镜像，所以就算 `docker` 这个无法编译成功，影响也不是很大，所以可以选择本地编译的方式。

### docker 编译更正

经群里老铁指导，可以直接在 Dockerfile 中增加 env 设置环境变量，解决上面说的docker编译问题。
具体参看下面中增加的两行代码

```
FROM golang:1.12-alpine as builder

# 增加如下两行代码
ENV GOPROXY https://goproxy.io
ENV GO111MODULE on

RUN apk add --no-cache gcc git linux-headers make musl-dev

ADD . /gyee
RUN cd /gyee && make all

# Pull gyee from builder to deploy container
FROM alpine:latest

RUN apk add --no-cache ca-certificates
COPY --from=builder /gyee/build/bin/* /usr/local/bin/

CMD ["gyee"]
```

### 本地编译

在 gyee 所在的目录执行如下命令直接编译整个项目

```
make all
```

成功的情况下就会看到类似如下的输出

![build success](/images/yeeco/build-success.png)

接下来就可以执行 yeeco 的程序了。
启动两个 shell，分别执行如下的程序，就可以看到相应的输出

```shell1
./build/bin/bootnode
```

![bootnode](/images/yeeco/bootnode.png)

```shell2
./build/bin/gyee
```

![gyee-exec](/images/yeeco/gyee-exec.png)

至此编译已经完成，剩下的就是开始源代码相关的阅读和贡献了！

# 附录



