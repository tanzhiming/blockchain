# 使用私有数据

以marbles private data chaincode演示。

## 启动网络

1. 清理环境

```shell
cd fabric-samples/first-network
./byfn.sh down
```

如果已经运行过该指南需要删除底层容器和镜像

```shell
docker rm -f $(docker ps -a | awk '($2 ~ /dev-peer.*.marblesp.*/) {print $1}')
docker rmi -f $(docker images | awk '($1 ~ /dev-peer.*.marblesp.*/) {print $3}')
```
2. 启动BYFN网络(使用CouchDB)

```shell
./byfn.sh up -c mychannel -s couchdb
```

## 安装和实例化链码

### 在所有peer安装链码

BYFN网络包括两个组织Org1和Org2，每个组织有两个peer。
* peer0.org1.example.com
* peer1.org1.example.com
* peer0.org2.example.com
* peer1.org2.example.com

进入客户端

```shell
docker exec -it cli bash
```


1. peer0.org1.example.com 安装链码

```shell
peer chaincode install -n marblesp -v 1.0 -p github.com/chaincode/marbles02_private/go/
```

2. peer1.org1.example.com 安装链码

```shell
export CORE_PEER_ADDRESS=peer1.org1.example.com:7051
peer chaincode install -n marblesp -v 1.0 -p github.com/chaincode/marbles02_private/go/
```

3. 切换到Org2

```shell
export CORE_PEER_LOCALMSPID=Org2MSP
export PEER0_ORG2_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_ORG2_CA
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
```

4. peer0.org2.example.com 安装链码

```shell
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
peer chaincode install -n marblesp -v 1.0 -p github.com/chaincode/marbles02_private/go/
```

5. peer1.org2.example.com 安装链码

```shell
export CORE_PEER_ADDRESS=peer1.org2.example.com:7051
peer chaincode install -n marblesp -v 1.0 -p github.com/chaincode/marbles02_private/go/
```

### 实例化链码

实例化marbles私有数据链码

```shell
export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C mychannel -n marblesp -v 1.0 -c '{"Args":["init"]}' -P "OR('Org1MSP.member','Org2MSP.member')" --collections-config  $GOPATH/src/github.com/chaincode/marbles02_private/collections_config.json
```

## 存储私有数据

切换到Org1

```shell
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export PEER0_ORG1_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

调用链码创建私有数据

```shell
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["initMarble","marble1","blue","35","tom","99"]}'
```


## 在授权的peer上查询私有数据



第一查询命令调用`readMarble`方法

```go
// ===============================================
// readMarble - read a marble from chaincode state
// ===============================================

func (t *SimpleChaincode) readMarble(stub shim.ChaincodeStubInterface, args []string) pb.Response {
     var name, jsonResp string
     var err error
     if len(args) != 1 {
             return shim.Error("Incorrect number of arguments. Expecting name of the marble to query")
     }

     name = args[0]
     valAsbytes, err := stub.GetPrivateData("collectionMarbles", name) //get the marble from chaincode state

     if err != nil {
             jsonResp = "{\"Error\":\"Failed to get state for " + name + "\"}"
             return shim.Error(jsonResp)
     } else if valAsbytes == nil {
             jsonResp = "{\"Error\":\"Marble does not exist: " + name + "\"}"
             return shim.Error(jsonResp)
     }

     return shim.Success(valAsbytes)
}
```

第二个查询调用`readMarblePrivateDetails`方法

```go
// ===============================================
// readMarblePrivateDetails - read a marble private details from chaincode state
// ===============================================

func (t *SimpleChaincode) readMarblePrivateDetails(stub shim.ChaincodeStubInterface, args []string) pb.Response {
     var name, jsonResp string
     var err error

     if len(args) != 1 {
             return shim.Error("Incorrect number of arguments. Expecting name of the marble to query")
     }

     name = args[0]
     valAsbytes, err := stub.GetPrivateData("collectionMarblePrivateDetails", name) //get the marble private details from chaincode state

     if err != nil {
             jsonResp = "{\"Error\":\"Failed to get private details for " + name + ": " + err.Error() + "\"}"
             return shim.Error(jsonResp)
     } else if valAsbytes == nil {
             jsonResp = "{\"Error\":\"Marble private details does not exist: " + name + "\"}"
             return shim.Error(jsonResp)
     }
     return shim.Success(valAsbytes)
}
```

查询私有数据命令

查询`marble1`的`name, color, size and owner`

```shell
peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarble","marble1"]}'
```

查询`marble1`的`price`

```shell
peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
```


## 在未授权的peer上查询私有数据

### 切换到Org2

```shell
export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
export CORE_PEER_LOCALMSPID=Org2MSP
export PEER0_ORG2_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_ORG2_CA
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
```

### 查询已授权的私有数据

```shell
peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarble","marble1"]}'
```

### 查询未授权的私有数据

```shell
peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

将出现类似以下的错误：

```
{"Error":"Failed to get private details for marble1: GET_STATE failed:
transaction ID: b04adebbf165ddc90b4ab897171e1daa7d360079ac18e65fa15d84ddfebfae90:
Private data matching public hash version is not available. Public hash
version = &version.Height{BlockNum:0x6, TxNum:0x0}, Private data version =
(*version.Height)(nil)"}
```

## 清理私有数据

私有数据在若干块之后清理掉，只保存hash数据在区块链中。`blockToLive`是私有数据保留到多少块后


切换到Org1中的peer0

```shell
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export PEER0_ORG1_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

在另一个终端中执行

```shell
docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
```

执行结果：

```shell
[pvtdatastorage] func1 -> INFO 023 Purger started: Purging expired private data till block number [0]
[pvtdatastorage] func1 -> INFO 024 Purger finished
[kvledger] CommitWithPvtData -> INFO 022 Channel [mychannel]: Committed block [0] with 1 transaction(s)
[kvledger] CommitWithPvtData -> INFO 02e Channel [mychannel]: Committed block [1] with 1 transaction(s)
[kvledger] CommitWithPvtData -> INFO 030 Channel [mychannel]: Committed block [2] with 1 transaction(s)
[kvledger] CommitWithPvtData -> INFO 036 Channel [mychannel]: Committed block [3] with 1 transaction(s)
[kvledger] CommitWithPvtData -> INFO 03e Channel [mychannel]: Committed block [4] with 1 transaction(s)
```

切换回peer容器，查询marble1 price数据

```shell
peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
``` 

结果：

```shell
{"docType":"marblePrivateDetails","name":"marble1","price":99}
```


创建marble2 

```shell
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["initMarble","marble2","blue","35","tom","99"]}'
```

切换到终端窗口查看日志

```shell
docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
```

切换到peer容器

```shell
peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

结果: 私有数据还没有清理，数据无变化

```
{"docType":"marblePrivateDetails","name":"marble1","price":99}
```

转移marble2给“joe”

```shell
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["transferMarble","marble2","joe"]}'
```

切换到终端查看日志

```shell
docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
```

切换回容器
```shell
peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

结果:
```
{"docType":"marblePrivateDetails","name":"marble1","price":99}
```

转移marble2给"tom"

```shell
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["transferMarble","marble2","tom"]}'
```

查看日志

```shell
docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
```

切回容器

```shell
peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

结果：

```
{"docType":"marblePrivateDetails","name":"marble1","price":99}
```

最后，转移marble2给“jerry”

```shell
peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["transferMarble","marble2","jerry"]}'
```

切回到终端窗口查看日志

```shell
docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
```

切回容器

```shell
peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'
```

结果
```
Error: endorsement failure during query. response: status:500
message:"{\"Error\":\"Marble private details does not exist: marble1\"}"
```



## 使用私有数据索引

索引能够应用于私有数据集合，通过打包到目录`META-INF/statedb/couchdb/collections/<collection_name>/indexes`, 该目录和链码同级。当链码实例化时使用`--collections-config`标志时，索引将自动部署。


