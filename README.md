# TAU - Phone Crypto Mining
TAU is a research project to create a phone based blockchain mining and communication protocol without tieing to dedicated servers, called libTAU. Devices could communicate with each other through public keys rather than IP addresses. To demonstrate the libTAU, Phone Crypto Mining app is built to enable many communities to build independent blockchains. 
TAUcoin is one of the blockchain coins in the TAU blockchains community. TAUcoin public ledger will provide basic bootstrapping and time services for other communities. Community does not need taucoin to operate own communities. TAU software is free to use. 

## Applications built on Phone Crypto Mining network
We will experiment to build a demonstration purpose uber service on the phone crypto mining network, that will cut the commission of connecting rider and drivers. 

## 挖矿算法
* chain id: 32字节，包含社区名字和建立时间戳`hash(GenesisMinerPubkey + timestamp)定长``community name变长` ，每个区块内部都含有chain id，类似IPFS的multi-addressing的思路，一个区块链只要获得一些区块，就可以开始收集其他节点。
* consensus point: 在某个区块链中，节点成员对当前区块288个区块前的位置的区块投票结果(block hash, block number)，简单多数获胜，这个点是随时在变化，可以前进可以后退。当网络只有一个节点时，这个节点的投票结果就是consensus point。 
* 挖矿过程
  * 节点A收到UI给出的区块链的邀请信号，本质是个mutable item target, chain id + 推荐者公钥, 64字节。如果有多个推荐者，可以从UI多次给出64字节的邀请信号target。一个社区chain id和多个推荐者的公钥，可以放入二维码一起携带。
  * A建立chain id的社区节点列表，类似朋友列表，第一个成员是推荐者公钥
  * A根据当前时间戳计算出4个 unchoked peers，计算方法是把时间戳哈希后分成4个随机数，每个随机数带入节点列表哈希，寻找最近自己的节点。加上随机选取的2个节点，构成当前每5分钟的6个通信成员。 
  * A从6个成员中随机选取通信对象，获得区块链在线信号，包含consensus point hash, 当前tip block的payload immutable hash和endpoint，自己能够提供的其他区块的immutable hash和endpoint，自己需要的其他区块block number
  * A不断本地存放累计收到的所有区块，啥数据结构如何管理？
  * A试图通过hash link连接当前最难tip到genesis的区块，在连接过程中如果发现包含consensus point的block number区块，则认为是合法链接。不包含则为非法链接。非法链接区块和推荐者不进入黑名单。
  * 在连接成功的基础上：
      * state 数据库数据回到分叉点，从这个分叉点开始数学验证后续区块。分叉点可以是genesis block。数学验证如果发生错误，发生错误的区块签发者公钥，进入黑名单。发生数学计算错误的概率应该极低。state数据结构：block number, key, value
      * 被回滚的数据state，在新的state没有验证完之前，目前处理方式是直接删除。我们应该认为包含consensus point的数学验证应该是100%通过的。当然这个不绝对，但是目前情况下，我觉得可以容忍。
      * 头部的过期区块可以直接从state删除
### Key concepts in TAU Phone Mining.
Core UI experienses
- decentralized community with crypto-coins circulation
   * Create a chat/community/blockchain 
      * Give a name to new blockchain and its coin
      * Creation of a blockchain with 1 million coins on 5 minutes per block generation rate
   * Transactions on community blockchain：社区主界面包含节点打包交易出块，节点确认区块，发送交易，消息，和选择器
      * Forum Note，Mark down serverless web. 每个节点的note可以视为一个连续的“html file"
      * Wiring Transaction
      * Genesis
   * Text messaging with people and group members. 
   * Rename/blacklist - users can rename or blacklist a peer
   * Support unlimited multiple devices sharing same address/account. 
- Dashboard
  * app system: network data consumption, nodes status and devices status
  * chain status: 链的长度，难度，节点数，节点出块数量，总流通币量，前十名持币地址和币量，前10名power地址和pot，共识点投票前三名的区块号和哈希最后2个bytes的16进制,tip前三名区块号和哈希，当前分叉点区块号和哈希，全部节点列表可以点击添加
  * friend status: last seen, last communication, common chains

## Concepts
- Version 1 default parameters: 
  - 5 minutes average to generate a block. It can be upgraded when network enhance. 
  - One block has one transaction. One block can include more transaction by encoding the hash of other transactions, which may be implmented in future version. 
- Community ChainID := `hash(GenesisMinerPubkey + timestamp)定长``community name变长`
  - Community will choose its own `community name`. 
  - example: TAUcoin ID is TAUcoin#hash 
- Public key is used as the crypto address: balance identifier under different chains; holds the power and perform mining. "Seed" generates privatekey and public key. 
- POT defines power as transaction volumn, the power annually increase according to Fibonacci sequence.
- Genesis block power: give one year power to genesis public key to make admin airdrop possible.  
- URL TAUchain:?bs=pk1&bs=pk2&dn=chainID // maybe 10 bootstrap publickeys provided

### Genesis 
```
// build genesis block
blockJSON  = { 
1. version;
2. chain id; 32 bytes `hash(GenesisMinerPubkey + timestamp)定长``community name变长`
2. timestamp; 
3. Domain name, IP address and port
4. blockNumber;
5. previousBlockRoot = null; // genesis is built from null.
6. basetarget = 0x21D0369D036978;
7. cummulative difficulty int64; 
8. generation signature;
11. msg;
12. `TsenderTAUpk`Noune = 365*24*12
13. `Tsender`Balance = 1,000,000;
14. `TminerTAUpk`Balance= 1,000,000; // in the genesis, Tsender = Tminer
15. `Treceiver`Balance = null;
16. ED25519 public key
17. ED25519 signature
}

```

## Constants
* MutableRange:  288 blocks, range could touch geneis block as extreme. This is a concept of blocks, unrelated to time. 
* GenesisBasetarget:  0x21D0369D036978; simulated 1 million blocks with average 60 seconds.
* DefaultMinBlockTime:  60 seconds, this is fixed block time. do not let user choose as for now.
* DefaultMaxBlockTime:  540 seconds, when no body mining, you can generate blocks
* DefaultBlockTime: 300 seconds
