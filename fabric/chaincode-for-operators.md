# 链码（运维角色）

## 链码的生命周期

Fabric提供了四个命令来管理链码的生命周期：`package`, `install`, `instantiate`, `upgrade`。 将来的版本中会增加`stop`和`start`来禁用和重新启用链码而不需要卸载它。


## 打包

chaincdoe包包括以下三部分:

* chaincode本身，其由ChaincodeDeploymentSpec或CDS定义。CDS根据代码及一些其他属性（名称，版本等）来定义chaincode。
* 一个可选的实例化策略，该策略可被*背书策略*描述。
* 一组表示chaincode所有权的签名。


### 创建包

打包chaincode有两种方式。第一种是当你想要让chaincode有多个所有者的时候，此时就需要让chaincode包被多个所有者签名。这种情况下需要我们创建一个被签名的chaincode包（SignedCDS），这个包依次被每个所有者签名。

另一种就比较简单了，这是当你要建立只有一个节点的签名的时候（该节点执行install交易）

我们先来看看更复杂的情况。当然，如果您对多用户的情况不感兴趣，您可以直接跳到后面的安装chaincode部分。

要创建一个签名过的chaincode包，请用下面的指令：

```shell
peer chaincode package -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/example02/cmd -v 0 -s -S -i "AND('OrgA.admin')" ccpack.out
```

### 包的签名

ChaincodeDeploymentSpec 可以选择被全部所有者签名并创建一个 SignedChaincodeDeploymentSpec（SignedCDS），SignedCDS包含三个部分：

* CDS包含chaincode的源码、名称与版本
* 一个chaincode实例化策略，其表示为背书策略
* chaincode所有者的列表，由Endorsement定义

```shell
peer chaincode signpackage ccpack.out signedccpack.out
```


### 安装chaincode

install交易的过程会将chaincode的源码以一种被称为ChaincodeDeploymentSpec（CDS）的规定格式打包，并把它安装在一个将要运行该chaincode的peer节点上。

```shell
peer chaincode install -n asset_mgmt -v 1.0 -p sacc
```


### 实例化chaincode

```shell
peer chaincode instantiate -n sacc -v 1.0 -c '{"Args":["john","0"]}' -P "OR ('Org1.member','Org2.member')"
```

### 升级chaincode

```shell
peer chaincode upgrade -n sacc -v 2.0 -c '{"Args":["john","0"]}' -P "OR ('Org1.member','Org2.member')"
```