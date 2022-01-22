TAU Whitepaper 2 - Perishable Proof of Transaction - drafting...
Date: Feb 1st, 2022

In order to make Proof of Transaction consensus operatable on a mobile device, we propose a pershable blockchain. The history beyond certain time frame will be expired to relief the burden of mobile full nodes. 
There are a few benefits other than smaller storage for perishable POT chain. Blockchain mining power will be more evenly distributed, due to expire of history power. We also make power untransferrable to make the chain immune to popular POS power shiftting attack. 

Perishable POT drafting

1. nounce: for message and transactions. 
2. six months time frame.
3. transaction que and mesasge que. 
4. voting for consensus point. 
5. why 6 months, and it is configurable. 
6. off chain archive. 

for both que, maxium is 100 members, each one has up to 10 txs. 

消息nounce的问题可以在消息内带入consensus block hash, 只有在本consensus 范围内才可以上链，且一次。 这个覆盖范围是有共识的，而且长度不长。

attack resistance: power shifitinig attack. 
perishable make power distribution more even than stake concentration. 
#### Blockchain bootstrap
TAU blockchain need initial members to provide ledger data, on IP2, we no longer give bootstrap IP addresses for chain bootstrap such as all other chains built on top IP1. When some one received the invitation of a blockchain either from message or other blockchain, along with chain ID, bootstrap nodes public key information has to be attached. The more such information the better. Along the data sync with blockchain nodes, each node will learn more blockchain bootstrap and keep them in the local data base.  
miner end point: * add sender and miner end points, IP + port. This is for trustless bootstrap.

#### nodes gossipping - link gossipping communication on ip2

#### block mining.
* start with seeking consensus point through community voting
* synching blocks and populate state database
* generating blocks and transaction and gossippinig


## 挖矿算法
* chain id: 32字节，包含社区名字和建立时间戳`hash(GenesisMinerPubkey + timestamp) 8 bytes定长``community name变长 24 bytes` ，每个区块内部都含有chain id，类似IPFS的multi-addressing的思路，一个区块链只要获得一些区块，就可以开始收集其他节点。
* consensus epoch: 在某个区块链中，节点成员对当前区块288个区块前的位置的区块投票结果(block hash, block number)，简单多数获胜，这个点是随时在变化，可以前进可以后退。当网络只有一个节点时，这个节点的投票结果就是consensus point。 Consensus point 可以定义为离开目前tip，倒数第二个结尾区块号为00的区块，这样可以投票集中。从而投票周期是每200 block内一个节点只能投1票，就是每天记录一票。
* stateless blockchain: 在每个区块里面要把状态变化补全，由于部分状态会过期，导致丢失nonce。stateless statechain 可能是更好的名称, stateless是不能使用UTXO，由于区块丢失，余额计算不准确，不可能为每个utxo建立世界戳；账号系统中只要账号存在，就是准确的，不会每个区块进步都发生余额度变化。consensus epoch号最小为0，0之前的区块也不需要验证，所以我们其实有个负区块的概念，就是社区创立时，最初的地址里面的币是可以人为设置到负数区块里面。最前面的负数区块的向前连接的hash可以是null. 这样解决一个起初社区建立初始社群的问题。0号区块之前的区块都是在consensus epoch之前的都不需要数学验证。 
* 验证：区块入blockDB完全不需要验证，数学验证就是从consensus epoch之后开始, consensus epoch之前的区块在回溯过程中直接填写入statedb，epoch之后的数学验证使用statedb中已经验证的kv。 节点可以根据自己的statedb，对交易池做初步验证入池。节点自己发出的区块要严格验证。relay其他区块传递包括tip，历史区块都不需要验证。发生回滚时，新验证的节点代替旧验证的state。如果回滚数学验证失败，则把数学验证失败的共钥拉入黑明单，清除分叉点后面的state kv。
* 挖矿过程
  * 节点A收到UI给出的区块链的邀请信号，本质是个mutable item target, chain id + 推荐者公钥, 64字节。如果有多个推荐者，可以从UI多次给出64字节的邀请信号target。一个社区chain id和多个推荐者的公钥，可以放入二维码一起携带。
    * 这里需要修改mutable item合法性算法，就是前后半32位都可以验证通过签署内容。由于如果把chainID放在salt里面，就是target的前32位，会导致一个区块链的中继节点集中到某几个节点。所以这个位置和朋友mutable内容正好相反，朋友列表中发送者在target后32位可以签署，区块链是发送者在target前32位可以签署。 
  * A建立chain id的社区节点列表，类似朋友列表，第一个成员是推荐者公钥



  * A根据当前时间戳计算出5个 unchoked peers，计算方法是把时间戳哈希后分成5个随机数，每个随机数带入节点列表哈希，寻找最近自己的节点。加上每次随机选取的1个节点，构成当前每5分钟的5个固定+1个可变的通信成员。 
  * 每次循环A从6个成员中随机选取一个通信对象，获得区块链在线信号:
    * 包含consensus point hash和block number，这类区块要经过自己数学验证才能推荐 
    * 自己的挖出的区块end point，需要经过数学验证
    * 自己提供的其他区块，包括tip，的immutable data hash和endpoint，这类区块要不需要经过自己数学验证
    * 自己需要的其他区块的hash 或 block number，
    * 交易池（通信池）
      * 交易池(通信池)哈希数组
      * 每个节点放入本地交易池条件：根据自己手里的statedb中已经验证的kv，来初步验证这比交易是否进入交易池，未上链的public key，不参与choking通信。
      * 出块所用的本地私有交易缓存：长度10；交易费大的交易，可以验证的交易，这个不向社区公布。
      * 对于最近5分钟内的消息，每个节点按照交易费大小来排序，发到区块链在线消息，区块链在线消息结构填满为止，就是说你的消息给的交易费不够，可能无法被中继出去。一个消息hash到2个字节，每个在线信号可以存储约200个消息。

  * A不断本地存放累计收到的所有区块，啥数据结构如何管理，KV数据库
  * A试图通过hash link连接当前最难tip到genesis的区块，在连接过程中如果发现包含consensus point的block number区块，则认为是合法链接。不包含则为非法链接。非法链接区块和推荐者不进入黑名单。获得难度更高区块tip 时，立即试图本地链接区块到consensus point, 如果本地连接不成（验证hash链中断），就不认可难度，只接受state，继续积累区块。
  * 在连接成功的基础上，即包括consensus point：
      * state 数据库数据回到分叉点，从这个分叉点开始数学验证后续区块。分叉点可以是genesis block。数学验证如果发生错误，发生错误的区块签发者公钥，进入黑名单。发生数学计算错误的概率应该极低。state数据结构：key, value, block number
      * 被回滚的数据state，分叉点后的数据，目前处理方式是直接删除，假设新分叉可以验证通过。我们应该认为包含consensus point的数学验证应该是100%通过的。当然这个不绝对，但是目前情况下，我觉得可以容忍。如果发生验证不通过，这个是明显恶意打击，这个公钥被列入黑名单。系统重新寻找最难区块。
      * 在没有充分验证链的情况下，先相信区块内涵的state，这样用户可以先获得部分区块链信息。所以每个state KV，要有一个标志verified
      * 头部的过期区块可以直接从state删除
