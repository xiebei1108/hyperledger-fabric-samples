# 牛奶食品溯源项目

## 业务逻辑

对于食品而言，其生产链通常较长，为了保证食品安全，消费者有去了解食品生产过程的需求。因为多个数据链较长，数据比较分散，需要通过区块链来建立一个从食品生产到中间配送，以及最后销售终端的整个食品跟踪数据网络。利用区块链，可以有效地解决数据中心化导致的容易丢失、篡改等问题，而且利用fabric的信任背书机制，能够保证数据真实可信。

在这个milk sample中，构建了一个牛奶数据溯源系统。在本系统中，有三个参与组织：奶牛场(farm)、加工厂(factory)、销售终端(seller)。

- 奶牛场，负责将奶牛健康数据上报
- 加工厂，负责将牛奶分装数据报上区块链网络
- 销售终端，负责将配送情况上报

在获得多方上报的数据后，消费者可以根据牛奶的id，获取其生产、加工、配送整个过程中的可信数据。

## 数据模型

对于每个组织，都有其自己的chaincode，根据牛奶生产过程中的不同状态，可以设计这样的数据模型：

- farm chaincode
  - cow_id
  - time
  - report
- factory chaincode
  - milk_id
  - machine_id
  - cow_id
  - time
  - operation
- seller chaincode
  - milk_id
  - time
  - operation

各个chaincode提供的接口，以及其参数：

- farm chaincode
  - cowReport: cow_id, report
  - getCowHistory: cow_id
- factory chaincode
  - milkReport: milk_id, cow_id, machine_id, operation
  - getMilkHistory: milk_id
- seller chaincode
  - milkReport: milk_id, operation
  - getFullMilkHistory: milk_id

## 搭建网络 && 安装chaincode

```shell
cd network/
sh network.down # if network is running
sh network.sh
```

## example
首先分别给三个组织都添加数据：
1.  `peer chaincode invoke -o milk-orderer:7050 -C milkchannel --waitForEvent --peerAddresses milk-partya:7051 milk-partyb:7051 -c '{"Args":["putCowReport", "cow id",  "report data for cow lallala "]}' -n farm `
2.  `peer chaincode invoke -o milk-orderer:7050 -C milkchannel --waitForEvent --peerAddresses milk-partya:7051 milk-partyb:7051 -c '{"Args":["putMilkReport", "milk id", "cow id", "machine id",  "report data aaaa"]}' -n factory `
3.  `peer chaincode invoke -o milk-orderer:7050 -C milkchannel --waitForEvent --peerAddresses milk-partya:7051 milk-partyb:7051 -c '{"Args":["putMilkReport", "milk id", "seller operation"]}' -n seller`

最后查询牛奶id为`milk id`的生产链条数据：
` peer chaincode invoke -o milk-orderer:7050 -C milkchannel --waitForEvent --peerAddresses milk-partya:7051 milk-partyb:7051 -c '{"Args":["getFullMilkHistory", "milk id", "seller operation"]}' -n seller`。

获取到的数据为:
```json
{
    "report_map": {
        "factory": [
            {
                "milk_id": "milk id",
                "cow_id": "cow id",
                "machine_id": "machine id",
                "report_data": "report data aaaa",
                "timestamp": "2020-02-11 01:52:20 PM"
            }
        ],
        "farm": [
            {
                "milk_id": "",
                "cow_id": "cow id",
                "machine_id": "",
                "report_data": "report data for cow lallala ",
                "timestamp": "2020-02-11 01:51:59 PM"
            }
        ],
        "seller": [
            {
                "milk_id": "milk id",
                "cow_id": "",
                "machine_id": "",
                "report_data": "seller operation",
                "timestamp": "2020-02-11 01:56:01 PM"
            }
        ]
    }
}
```

## 应用程序


