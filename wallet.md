## 一、以太坊钱包开发原理

1.目前提供以太坊后端API及服务的有如下四个

* Infura  地址：[infura.io](https://infura.io/)
* Cloudflare 地址：[cloudflare-eth.com](https://cloudflare-eth.com/)
* Nodesmith  地址：[nodesmith.io](https://nodesmith.io/network/ethereum/)
* ChainStack 地址：[chainstack.com](https://chainstack.com/)

2.以Infura为例，注册Infura后，创建一个项目，之后在项目的设置项可以看到项目的ID,在下方设置好以太坊网络后，会在下方看到一个以太坊节点API，我们通过https或者json rpc 访问这个API获取节点数据。
![](/img/1.png)

3.Infura的以太坊API结构，https://<network>.infura.io/v3/YOUR-PROJECT-ID 其中 network就是选择的以太坊网络，YOUR-PROJECT-ID 就是创建的项目ID

4.可以在项目设置里面设置密码，作为请求流量的额外保护。

5.可以启用允许列表进行保护项目，如果项目未启动允许列表，则接受任何请求，每个项目每个类型最多允许30个允许列表条目，每种允许列表都与在一起，相同类型的多个条目进行“或”运算。

------------

## 二、常用API

* eth_accounts  返回客户端拥有的地址列表
* eth_blockNumber 返回当前的“最新块号”
* eth_call 立即执行新的消息调用，而无需在区块链上创建事务，为了防止滥用此api,gas参数不得超过当前区块限制的10倍，一般在本地测试环境中创建此行为。

* eth_chainId 返回当前配置链的ID，这是EIP-155引入的用于受重播保护的交易签名中的值。
* eth_eatimateGas 生成并返回完成交易所需要的gas的估算值，交易将不会添加到区块链中，一般估算值可能大大超过实际使用的值，方法跟eth_call类似，也会消耗gas，所以一般在测试环境中创建
* eth_gas 返回以wei为单位的当前gas
* eth_getBalance 返回给定地址的账户余额
* eth_getBlockByHash 按哈希返回有关块的信息
* eth_getBlockByNumber 按哈希返回有关块的信息
* eth_getBlockTransactionCountByHash 返回具有给定区块哈希值得区块中的事务数
* eth_getBlockTransactionCountByNumber 返回具有给定区块号的区块中的交易数量 
eth_getCode 返回给定地址的已编译智能合约代码（如果有）
* eth_getLogs 返回与给定过滤器对象匹配的所有日志的数组
为了防止查询消耗太多资源，eth_getLogs当前请求受到两个约束的限制：单个查询最多可返回10,000个结果，查询时间不得超过10秒，否则会返回错误
* eth_getStorageAt 从给定地址的储存位置返回值。
* eth_getTransactionByBlockHashAndIndex 通过区块哈希和交易索引位置返回有关交易的信息
* eth_getTransactionByBlockNumberAndIndex 通过块号和交易索引位置返回有关交易的信息
* eth_getTransactionByHash 返回有关给定哈希值的交易的信息。
* eth_getTransactionCount 返回从地址发送的交易数。
* eth_getTransactionReceipt 按交易哈希值返回交易的收据。请注意，收据不适用于未决交易。
* eth_getUncleByBlockHashAndIndex 通过散列和Uncle索引位置返回有关块的“ Uncle”的信息。
* eth_getUncleByBlockNumberAndIndex 通过散列和Uncle索引位置返回有关块的“ Uncle”的信息。
* eth_getUncleCountByBlockHash 从与给定块哈希匹配的块中返回块中的叔叔数。 
* eth_getUncleCountByBlockNumber 从与给定块号匹配的块中返回一个块中的叔叔数。
* eth_getWork  返回当前块的哈希，seedHash和要满足的边界条件（“目标”）。 
* eth_hashrate  返回节点每秒挖掘的哈希数。仅在节点正在挖掘时适用。
* eth_mining 如果客户端正在积极挖掘新块，则返回true。
* eth_protocolVersion 返回当前的以太坊协议版本。
* eth_sendRawTransaction 提交预签名交易以广播到以太坊网络。
* eth_SubmitWork 用于提交工作量证明解决方案。
* eth_syncing  返回带有有关同步状态或false数据的对象。
* eth_net_listening 如果客户端正在积极侦听网络连接，则返回true。
* eth_net_peerCount  返回当前连接到客户端的对等体的数量。
* eth_net_version 返回当前的网络ID。
* eth_web3_clientVersion 返回当前客户端版本。

------------------------------------

## 三、速率限制
帐户超出其每日请求限制后将受到费率限制。当受到速率限制后，会返回错误码及错误数据，下面是解决办法：
* 确保在URL中使用了项目ID。没有项目ID的请求将受到严格的速率限制，甚至被完全阻止。
* 在本地缓存以太坊数据。除非异常罕见地对链进行深度重组，否则可以无限期地缓存超过链头下方几个块的块。要求一次数据，然后将其保存在本地。
* 在DApp启动时限制RPC。同样，限制DApp在启动时立即调用的RPC数量。用户访问DApp的那部分时仅请求数据，并在下一次缓存较旧块中的任何内容。
* 不要紧紧轮询Infura。新块大约每15秒出现一次，因此以更快的速度请求新数据通常是没有意义的。eth_subscribe当有新块可用时，请考虑使用通知。
----------------
## 四、存档数据
1.存档节点：一个归档节点是识别归档模式运行的复仇充满节点的简化方法。以太坊全节点需要无限期地存储诸如标题和块内容之类的块数据，但是它们不需要为所有块持久保存每次块验证产生的世界状态。完整节点仅需要保持特定于块的状态足够长的时间，以便在发生区块链重组时能够重新计算（默认情况下，大多数实现中为128个块）。

在没有存档模式的情况下运行的全节点将修剪生成的状态以节省磁盘空间。这有助于节点的同步时间，并大大降低了存储和计算成本。由于以太坊账户和合约存储的工作方式，这意味着只有存档节点才能为某些早于128个块的RPC方法提供API请求。
2.存档数据：
```
getBalance
getCode
getTransactionCount
getStorageAt
call
```

[以太坊官方网址](https://ethereum.org/zh/developers/ "eth官网")


[Infura官方网址](https://infura.io/docs/ipfs/post/block-put "infura官网")
  







