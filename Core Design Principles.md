my working notes ... read at own risks
### Key ideas
* TAU is designed as blockchain tech stack based on self-addressing and filter-free data communication. Its applications include crypto coins mining on phones, community serverless communication and low cost IOT operation. Some known difficulties: 
    * personal phones and devices do not have stable IP address and clean firewall pass, they have to relay through certain servers such as whatsApp, telegram, youTube. 
    * lack of incentive for peers to provide data to other peers
    * internet routers and firewalls love to block privacy related traffic
    * hard to build trust worthy bootstrap service available on internet
* Bitcoin: Hash linked chain, central-less and permission-less concensus, on-chain UTXO state
* Ethereum: Accounts design of balance and nounce, stateless plan for only storing one year states. Leading to idea of persishable blockchain.
* NXT: Generation signature for proof of stake mining
* Libtorrent: Kadmelia DHT based pub/sub communication to provide file seeders location.
* Levenshtein [Distance](https://en.wikipedia.org/wiki/Levenshtein_distance): TAU potentially uses Levensthtein array than sequence number in TCP to achieve data transmission integrity; due to each public key can be used by multiple devices. 
* Bencode: libTAU use bencode to compose blocks and transactions. Bencode has mixed benefits of partial human readable and binary serilazation, more space saving by json and flexible than RLP. 
* UDP payload level Encryption: the payload is composed of relay node public key and encrypted bencoded data entry. Only receiver with right private key can decode payload. The UDP payload level encryption will make router not be able to recognize the protocol for the data.
* Device ID: one public key might include many device ID sharing same private key. The device ID is an optional. System will record this field and report it in the alert to UI, but will not process it. As long as the device has private key, its messages will be processed disregard what device ID it has. As long as private key is same, all devices are treated same.
* TAU community ledger serve as bootstrap and time server: For a decentralized system, it is hard to do a trust bootstrap and find a server to trust for time. Rather than bootstrap from fixed nodes, TAU nodes will use current ledger nodes for bootstrap and time calculation. Each node will continously learning network bootstrap IP addresses, which is the biggest vulnerable part of IP2. libTAU nodes will use all locally available ledger to bootstrap itself. In each TAU block, we recommend adding "end point" into block. When the block is accepted by the node, it will use this info into local bootstrap list. Initial software will come with part of TAUcoin blockchain ledger, which will include initial bootstrap nodes. 
* Relay IP address reachability: this is always a problem for a node to discover itself reachable or not by public inbound traffic call. there is no silver bullets solutions, uPNP and PCP protocol can provide some confidence, but still not entirely. This is why even a public node will need capture swarm to ensure it is reachability. 
* PPOT
```
TAU blockchain will only keep the chain state for six months only, any state and ledger beyond will be forgotten forever. This is probably mostly important inovation of TAU. It will make the blockchain in small size, good for communication functions and small community. 
TAU believes blockchain is not for instant cash payment due to the block latency. Financially, it is for digital assets and savings. It is also for community building and communication. 
A limited state size will also make inter-peers communication in controlable volumn. Assume 1 million peers existing in one chain, the data sync effort will be too big for small devices. On TAU chain, one chain will accept mostly 50k participants to make sure data sync is not overwhelmed. This is a quite big community already.
In a perishable chain, the main ledger is in nature a state chain, when each block evolves, it will reflec a new state. So when deleting a history state, basically just delete a history block, which is much easier than deleting a state in MPT in ethereum. Therefore, tau chain should really called as a "state chain", in each block, an account value is updated all time for all participants, sender, receiver and miners. This is why we can not support smart contract on tau, since contract could be deleted in history with its state still in operation, then this is a dizaster.  
As a miner, you will maximum need 365 x 180 blocks information. However mining can start from any time, as soon as consensus point is setup.
```
* State database vs state trie
  * eth use state trie to record each variable and value. In libTAU stateless plan adopted, each variable will only live for certain unique time window, not epochs, the trie key-value data structure is hard to maintain such expiry. 
  * libTAU will use relationship database such as sqlite to hold states.  
* DDOS: TAU network is a swarm, DDOS is mostly targetting to single point of the system to cause failure. We will use basic traffic peak control to flaten the traffic as for now. 
* consensus point: current block number - 200, then cloeset previous block with 00 ending. for example current block is 86789;  86789-200 = 86589, the 86500 is the consensus point. 
* stateful range: 6 months, 288 x 180 = 51840
* Dual stack: ipv4 and ipv6
ipv6 provides a stable bootstrap entry. we should support ipv6 as much as possible when data is not meterred. 
ISP will love to control incoming packets, the local routing table database will need to be smart enough to measure such restrictions. 

### Working mode
libTAU working config file: secrete key, bandwidth, invoke, 
* UI will assign daily bankwidth data and maximum invoke to libTAU.
* UI will deside to start v4 or v6 interface 
* UI will assign friend list of public keys and chain IDs
* UI will meature system cpu and memory and other factors to ajust bandwidth and invoke number

UI will collect platform info such as:
* charging or not
* bandwidth meterred or not and user preference of data spending
* upnp status
* memory
* cpu
* ipv4 or ipv6

Based on these info, UI will config invoke number of libTAU and how many socket interface needed in session
流量和频率控制策略

前台：不控制流量，对于当前聚焦通信的朋友时间间隔参数30秒, beta  = 3, 和区块链给予80%的随机优先级
后台：UI提供流量包来控制区块链频率


输入：流量 和 节点通信信号
网络付费状态和日流量消费设置
APP前后台
chat聚焦情况


##### Why not smart contract
Smart contract is able to generate big numbers of state, TAU is designed to run on a phone and support communication and payments. A Turing complete programable language will swallow the network resources. I am designing an TAU chain with “365 days rolling base memory without smart contract”. There will be no cut-off line, the state memory is on rolling basis to keep storage flat for each blockchain around maximum 100mb.  
Without smart contract, the blockchain will be purely for coins wiring and text. I think these are the most important things and sufficient for dApps to build logic such as javascript can be viewed as text. Assume in the future, all devices will need libTAU communication for server-less messaging, we want to make this layer to be cheap and efficient in computing resources consumption.
Smart contract requires contract to be immutable, which is contradictary to perishable blockchain concept. 
* Reduce the peer number space. We design serverless communication blockchain to only make one transaction per block, which means one sender one receiver, this will bring the total peer numbers under 288 * 365 = 105,120；the 6th root of that is 7, which is the swarm size of blockchain communication. 6th root is a reasonable social distance cap. 
* This means one TAU community chain can only hold 105,120 peers at current network phone condition, until next personal phone upgrade like 5G complete mature. 
Bot

每个地址有个bot的机器人，辅助通信，可以提供airdrop和交易服务。bot默认都是打开。
社区不存在bot，也就是我们不支持上链的智能协约。

##### Why not multimedia on DHT
TAU communication is a connection-less protocol, at the current stage, it is not ideal for large amount of data transfer such as video streaming and big files. 
The current app will only support: 
1. Text: some mark-down language
2. On-Chain web: note transaction with markdown language to form a linkable site. 

##### Choking in limited number of peers
Choking members resources: 4 for onchain peers, 1 for random address, 1 for transaction pool
When peers number increases in the blockchain, the data synchronization efficiency becomes low due to no central place to host information. Assume you have 1 million users in a group, the sync complexity is N square, which is 1 m x 1m, which is not possible for current network to handle. We have to come up with a better design. The TAU current plan is to restrict total chain size, so as the peer numbers. The 1 years length, 1 transaction per block, 5 minutes per block plan will bring us about 105,120 peers on the whole chain. 
* we plan to engage torrent choking design, means in any moment, a peer only exchange data with a set number peers. The number is 6th root of whole chain peers N. The 6th root is a magic number that in social media 6th steps will bring connection between any two persons in the world. This allows the traverse steps of a message to be under O(6) which is quite good.
* For each nodes synching blockchain ledger, collectively there are less incentive for any node to give data to stranger nodes. The choking plan, in any certain time window, only a subset of nodes are suppose to open access to each other and each will use "tit for tac" to exchange data. So that the nodes do not want to share data will found hard to receive data from the un-choked peers at a certain time frame. 
* libTAU, we use blockchain time to arrange planned communications paire between nodes, in each time, each nodes will know exactly what nodes to talk to. when communication starts, bothsides will have to provide data to each other. this is probabaly the most important thing in libTAU blockchain data exchange. 
* libTAU will use 6 blocks hash prior and include the immutable point to decide peers for unchoke. 

### voting the consensus point
Each node will collect voting from all peers through choking communication. One of the 6 choking peers are randomly select from state database. 
The voting is collecting opinions of top 21 higher staker it encounters, the simple majority wins the consenus point. Anything prior to the consensus point is considerred legit and do not need verification. These blocks will enter state db without math checking. 
Each node software can decide own voting strategy, it is a very individual thing for nodes. The variety of such strategy can increase the resistence of attacking. 
The choking communication serve a good base for such voting, since 5 of the 6 time stamp related nodes are suppose to do two way. 1 of the random nodes is only collection information only, kind of one way voting any way. 
The initial consensus point is when user getting information from referral. A node will always have consensus point. 

### Key concepts in TAU Phone Mining.
Core UI experienses
- decentralized community with crypto-coins circulation
   * Create a chat/community/blockchain 
      * Give a name to new blockchain and its coin
      * Creation of a blockchain with 1 million coins on 5 minutes per block generation rate
   * Transactions on community blockchain：社区主界面包含节点打包交易出块，发送交易，消息，和选择器 (Tip Blocks, Wiring Tx, Notes TX and Messages)
      * Onchain Note TX，Mark down serverless web. 每个节点的note可以视为一个连续的“markdown file", GFM mark down style.
      * Offchain message, regular text
      * Wiring Transaction
      * Tip Blocks
   * Text messaging with people and group members. 
   * Rename/blacklist - users can rename or blacklist a peer
   * Support unlimited multiple devices sharing same address/account. 
- Dashboard
  * app system: network data consumption, nodes status and devices status
  * chain status: 链的长度，难度，节点数，节点出块数量，总流通币量，前十名持币地址和币量，前10名power地址和pot，共识点投票前三名的区块号和哈希最后2个bytes的16进制,tip前三名区块号和哈希，当前分叉点区块号和哈希，全部节点列表可以点击添加朋友； （ Chain Length, Difficulty, Total Peers, Peers & Blocks, Total coins, Top 10 Peers and Coins, Top 10 Peers and Power, Top 3 Consensus Blocknumber and Hash, Top 3 Tip Blocknumber and Hash, Current Fork Blocknumber and Hash, Full Peers List) 
  * friend status: last seen, last communication, common chains
#### starting with non-chinese language. 
## Concepts
- Version 1 default parameters: 
  - 5 minutes average to generate a block. It can be upgraded when network enhance. 
  - One block has one transaction. One block can include more transaction by encoding the hash of other transactions, which may be implmented in future version. 
- Community ChainID := `hash(GenesisMinerPubkey + timestamp)定长``community name变长`
  - Community will choose its own `community name`. 
  - example: TAUcoin ID is hash#TAUcoin 
- Public key is used as the crypto address: balance identifier under different chains; holds the power and perform mining. "Seed" generates privatekey and public key. 
- POT defines power as transaction volumn, the power annually increase according to Fibonacci sequence.
- Genesis block power: give one year power to genesis public key to make admin airdrop possible.  
- URL TAUchain:?bs=pk1&bs=pk2&dn=chainID // maybe 10 bootstrap publickeys provided

### Genesis 
```
// build genesis block
block  = { 
1. version;
2. chain id; 32 bytes `hash(GenesisMinerPubkey + timestamp)定长``community name变长`
3. timestamp; 
5. blockNumber; // for each 105,120 blocks, we will increase power unit follow fibonacci series - 1,1,2,3,...
6. previousBlockRoot = null; // genesis is built from null.
7. basetarget = 0x21D0369D036978;
8. cummulative difficulty int64; 
9. generation signature;
10. payload, starting with domain and ip info;
11. `TsenderTAUpk`Nonce = 1;
12. `Tsender`Balance = 1,000,000;
13. `TminerTAUpk`Balance= 1,000,000; // in the genesis, Tsender = Tminer = Treceiver
14. `TminerTAUpk`Nonce= 1;
15. `Treceiver`Balance = 1,000,000;
16. `Treceiver`Nonce= 1;
17. miner end point: * add sender and miner end points, IP + port. This is for trustless bootstrap.
18. ED25519 public key
19. ED25519 signature
}

```

## Constants
* GenesisBasetarget:  0x21D0369D036978; simulated 1 million blocks with average 60 seconds.
* DefaultMinBlockTime:  60 seconds, this is fixed block time. do not let user choose as for now.
* DefaultMaxBlockTime:  540 seconds, when no body mining, you can generate blocks
* DefaultBlockTime: 300 seconds

## Some key concepts
#### TAU Doze
```
大多数操作系统包括android和chromeOS都会对电池电源做精细管理，但是对流量一般不做控制。TAU考虑到非洲地区流量成本，增加这个模式
进入条件：
1. 如果3分钟内（固定参数）用户和TAU app没有“鼠标键盘手指交互”并且没有“android doze过”就进入“TAU Doze”。
2. 没有网络时，节点为0

TAU Doze模式：
app根据网络流量剩余来间歇工作 >70%，>50%，>30%，>10%，就进入“区块链业务进程挂起”对应的0，5，10，20分钟策略。

退出tau doze模式的条件：
1、鼠标键盘手指交互
2、网络任何变化
3、Android doze结束
4、切入前台 foreground
5、流量包选择变化
```

#### Signal of Switch/Stay on foreground
```
当用户切入前台，或者在前台保持10分钟，UI将触发libtau boost 信号。libtau收到这个信号后做：
1. 发送p2p通行中对方没有收到的消息
2. 对所有自己参加的区块链执行类似扫码加入动作（发出在线信号和读取区块信息）

```

#### Group Chat connection list



```
启初状态：是从区块链和状态机中读取各一半节点。
这个表格的增加：接受到外界在线信号，缓存中的gossip live节点，
这个表格删除：在本类别中被新节点替换
节点类别：1/3来自区块，1/3来自状态机，1/3来自链下
```
#### P2P API - IP2 
```
put 参数： 控制命令，发送时间戳 X，信息哈希；由于X会被加密，中继无法知道X内容，中继服务器只能使用自己本地时间，这里有误差
接受方的alert: 收到控制命令，X和信息哈希
get 参数： 信息哈希，时间戳（如果为0则视为immutable信息，任何一个满足的数据都返回）； 如果有合法时间戳是为mutable信息，则只返回大于时间戳的第一个数据
A发送B过程
1. A put 数据到 A swarm
2. A 发送 relay 信号给 B，包括A swarm的辅助信息和信息哈希
3. B 接受到relay信号，用X减去3秒读取A swarm中的信息，如果长超时失败，用X减去6秒再读取一次
```
