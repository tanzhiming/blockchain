# 环境准备

以下均在CentOS 7操作系统中安装。

## 安装cURL

如果已经安装的cURL工具在运行curl命令时报错，请下载最新版本的[cURL](https://curl.haxx.se/download.html)。

## Docker 和 Docker Compose安装

Docker version >= 17.06.2-ce

Docker Compose version >= 1.14.0


参考 [Docker和Docker Compose安装](https://github.com/tanzhiming/notes/blob/master/docker/docker-install.md)


检查Docker版本信息：
```shell
docker --version
```

检查 Docker Compose版本：
```
docker-compose --version
```

## Go 编程环境

Go version 1.11.x

1. 下载[Go](https://golang.org/dl/), [GO（中国）](https://golang.google.cn/dl/)

2. 解压到/usr/local
```shell
tar -C /usr/local -xzf go1.11.4.linux-amd64.tar.gz
```

2. 环境变量
```shell
GOROOT=/usr/local/go
GOPATH=$HOME/go

PATH=$PATH:$GOROOT/bin:$GOPATH/bin

export GOROOT GOPATH PATH
```

## Node.js 和 NPM

只支持 version 8.x

下载 [Node.js](https://nodejs.org/en/download/releases/)
```shell
xz -d node-v8.12.0-linux-x64.tar.xz
tar xf node-v8.12.0-linux-x64.tar
mv node-v8.12.0-linux-x64 /usr/local/node
```
2. 环境变量
```shell
PATH=$PATH:/usr/local/node/bin
export PATH
```


## Python 

Python version 2.7

```shell
yum instsall -y python
```

# 安装样例, 二进制和Docker镜像

1. clone样例代码
```shell
git clone https://github.com/hyperledger/fabric-samples.git $GOPATH/src/github.com/hyperledger/fabric-samples
```

2. 下载二进制，镜像
```shell
cd $GOPATH/src/github.com/hyperledger/fabric-samples
git checkout v1.4.0-rc2
./scripts/bootstrap.sh 1.4.0-rc2
```
>./scripts/bootstrap.sh [version] [ca version] [thirdparty_version]

