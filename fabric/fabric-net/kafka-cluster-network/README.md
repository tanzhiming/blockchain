# 操作指南


## 生成构件

```shell
./bin/cryptogen generate --config=./crypto-config.yaml 
```

```shell
./bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```

```shell
./bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/mychannel.tx -channelID mychannel
```

## 替换CA证书

`2dd2dd126764ae2b8ede1bf08f456ef5c0b274da9d1bced2009d7eb8416b946a_sk`需要替换

```yaml
  ca:
    container_name: ca
    image: hyperledger/fabric-ca
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca
      - FABRIC_CA_SERVER_TLS_ENABLED=false
      - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem
      - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/2dd2dd126764ae2b8ede1bf08f456ef5c0b274da9d1bced2009d7eb8416b946a_sk
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/2dd2dd126764ae2b8ede1bf08f456ef5c0b274da9d1bced2009d7eb8416b946a_sk -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/org1.example.com/ca/:/etc/hyperledger/fabric-ca-server-config

```


## 创建通道安装链码

`docker exec -it cli bash`进入客户端


### peer0.org1.example.com

创建通道

```shell
peer channel create -o orderer0.example.com:7050 -c mychannel -t 50s -f ./channel-artifacts/mychannel.tx
```

加入通道

```shell
peer channel join -b mychannel.block
```

安装合约

```shell
peer chaincode install -n mycc -p github.com/chaincode/chaincode_example02/go/ -v 1.0
```

合约实例化

```shell
peer chaincode instantiate -o orderer0.example.com:7050 -C mychannel -n mycc -c '{"Args":["init", "A", "10", "B", "10"]}' -P "OR ('Org1MSP.member', 'Org2MSP.member')" -v 1.0
```

合约查询

```shell
peer chaincode query -C mychannel -n mycc -c '{"Args": ["query", "A"]}'
```

### foo27.org2.example.com

将`peer0.org1.example.com`节点的`mychannel.block`复制到其客户端peer目录下

进入客户端

加入通道

```shell
peer channel join -b mychannel.block
```

安装合约

```shell
peer chaincode install -n mycc -p github.com/chaincode/chaincode_example02/go/ -v 1.0
```

查询合约

```shell
peer chaincode query -C mychannel -n mycc -c '{"Args": ["query", "A"]}'
```


### bar.org2.example.com

同`foo27.org2.example.com`