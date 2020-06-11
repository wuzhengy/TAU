# TAU - Torrent Publishing on Mobile Blockchains
Torrent information publishing on permission-less parallel blockchains using proof-of-transaction consensus, mobile devices and distributed-hash-table.

Core UI experienses:= {
- **"(+)"** floating button with three functions:
   * Publish Video.
      * Make a Magnet Link sharing transaction
      * Describe the content and paste the link, app verifies the format of the link
      * Chose community/chain to publish //support multi-chains publishing
      * Reminder: make user actually file is seeded on a public IP torrent client.
   * Create Publishing Community: build a blockchain of community for torrent sharing. 
      * Give a name to new blockchain // auto gen a city name for choice 
      * Creation of a blockchain with 10 million coins at 5 minutes per block generation rate
      * Annouce the new chain on TAU, given that TAUcoin balance is enough.
   * Other Transactions
      * Message  //support multi-chains pub
      * Chain Annoucement //support multi-chains pub
      * DHT BootStrap Node Annoucement //support multi-chains pub
      * Wiring Transaction // single chain
      * Coins trading telegram group annoucement
- Dashboard:  "DHT Exploring Kb/s" 
  - Change effective for 1 hour, 3h, 12h, forever. Default is `1 hour`.
    - on telecom data: 
      - DHT_get only, daily maximum data 30M, 60M, Unlimited. (default `60M`). 
      - except when sending transaction, enable DHT_put
    - on wifi: 
      - Daily maximum data 200M, Unlimited. (default `Unlimited`)
  - Common config, do not display to UI
    - Charging ON: wake lock ON. 
    - Charging OFF: wake lock OFF. random wake up between 1..WakeUpTime
    - Internet OFF: wake lock OFF. random wake up between 1..WakeUpTime
- Chains prebuilt into TAU app.
    - TAUcoin chain: provides place to publish new community. App will read TAUcoin chain for app operation such as bootstrap DHT node.
    - TAU-T chain: provides default place to publish video magnet links and dht bootstrap node.
- Search engine: internal search
--- 
## Persistence variables
```
1.  Chains  map[ChainID] config; 
2.  CurrentBlockRoot    map[ChainID]; // the map holding the recent blocks
3.  MutableRange    map[ChainID]uInt  // blocks in the mutable range is subject to change any time 
5.  TAUpeers       map[ChainID]map[TAUpk]config;// for chains, when discovery new TAU peers, adding here with config info.
6.  TAUselfTxsPool         map[ChainID]map[TxHASH]config
7.  TotalUsedData
8.  ImmutablePointBlock    map[ChainID] uInt 
9.  VotesCountingPointBlock   map[ChainID] uInt 
```
## Data flow: StateDB, BlocksDB and DHT
  - statedb is the local database holding account power and balance
  - blockdb is the local database holding the blocks content that will be put and get through DHT
  - DHT is the network key-value database
  - Data flow
    - `blockdb` ---> `libtorrent put` ---> `DHT`
    - `blockdb` <--- `libtorrent get` <--- `DHT`
    - `statedb` <--> `blockdb`
  - myth: p2p communication is not possible to implement given firewalls and personal device security restrictions. DHT is used to be p2p communication substrate. When peer is not online, the content is still available to exchange. 
## Design Concepts
- Version 1 operation parameters: 5 minutes a block for single community. These numbers can be upgraded when network infrastructure upgrading. 
- One block has one transaction for both DHT easy lookup and account state update. Lookup block is the same as transaction. This keeps DHT key value table simple. 
- Community ChainID := `community name`#`optional block time interval in seconds`#`hash(GenesisMInerPubkey + timestamp)` 
  - Community chain will choose its own name. 
  - Coin volumen is 10 million
  - Default block time is 300 seconds, which will be enhanced by software and device improvement
  - example: TAUcoin ID is TAUcoin##hash; community ID: Shanghai#600#hash, which is a chain name Shanghai with 10 million coins and 600 seconds block time. 
- TAUpk: balance identifier under different chains; holds the power and perform mining. Seed generate privatekey then public key. In TAU, we use seed to import and export account. 
- New POT use power as square root the nounce.
- 投票策略设计。
   - 投票范围：当前ImmutablePointBlock **往未来** 的 `MutableRange` 周期为计算票范围。这个范围应该部分达到未来。
   - 每到新的VotesCountingPointBlocks结束时，统计投票产生新的ImmutablePointBlock和新的VotesCountingPointBlocks, 得票最高的当选, 同样票数时间最近的胜利。
      - 如果投出来的新ImmutablePointBlock, 这个block不在目前链的mutable range内，把自己当作新节点处理。检查下自己的历史交易是否在新链上，不在新链上的放回交易池。。
      - 新节点上线，快速启动随机相信一个能够覆盖到全部历史的链获得数据，设置链的（顶端- (`MutableRange` - DefaultMaxBlockTime)）为ImmutablePointBlock，开始出块做交易，等待下次投票结果。
      - 如果投票出的新ImmutablePointBlock，root在同一链上，说明是链的正常发展，继续发现最长链，在找到新的最长链的情况下，检查下自己以前已经上链的交易是否在新链上，不在新链上的放回交易池，交易池要维护自己地址交易到。// 新的ImmutablePointBlock root不可能早于目前ImmutablePointBlock的。
   - 获得新root，如果是longest chain开始进入验证流程，如果不是进入计票流程.  
   - If the longest chain forks prior to mutable range, going to new node process; if fork prior to 3x mutable range, ignore. therefore, 3x mutablerange is really the finality. 
- **libtorrent dht as storage and communication**
  * salt = chainID
  * immutable message = block content
  * mutable message's public key = TAUpk public key
  * mutable message by hash(public key + salt) = value is the block hash of TAU pkess future prediction on a chain
- get message (mutable)
  - Get future block associated with a TAUpk + chainID (pub key + salt).  If the queried node has the block, it is returned in a key "values" as a list of strings. If the queried node has no such block, a key "nodes" is returned containing the K nodes in the queried nodes routing table closest to the hash supplied in the query.  TAU dev public DHT nodes will store all info hash space for all chain_id.
  http://www.bittorrent.org/beps/bep_0044.html
```
Request:
{ "a":
    { "id": <20 byte id of sending node (string)>,
        "seq": <optional sequence number (integer)>,
        "target:" <20 byte SHA-1 hash of public key and salt (string)>
    },
    "t": <transaction-id (string)>,
    "y": "q",
    "q": "get"
}
Response:
{  "r":
    { "id": <20 byte id of sending node (string)>,
        "k": <ed25519 public key (32 bytes string)>,
        "nodes": <IPv4 nodes close to 'target'>,
        "nodes6": <IPv6 nodes close to 'target'>,
        "seq": <monotonically increasing sequence number (integer)>,
        "sig": <ed25519 signature (64 bytes string)>,
        "token": <write-token (string)>,
        "v": <any bencoded type, whose encoded size <= 1000>
    },
    "t": <transaction-id (string)>,
    "y": "r",
}
```  
- put block
```
Request:
{ "a":
    { "cas": <optional expected seq-nr (int)>,
        "id": <20 byte id of sending node (string)>,
        "k": <ed25519 public key (32 bytes string)>, TAU public key
        "salt": <optional salt to be appended to "k" when hashing (string)>
        "seq": <monotonically increasing sequence number (integer)>,
        "sig": <ed25519 signature (64 bytes string)>,
        "token": <write-token (string)>,
        "v": <any bencoded type, whose encoded size < 1000>
    },
    "t": <transaction-id (string)>,
    "y": "q",
    "q": "put"
}
{ "r": { "id": <20 byte id of sending node (string)> },
    "t": <transaction-id (string)>,
    "y": "r",
}
```
- DHT persistence storage: DHT includes nodes and items
  - DHT nodes state and items table is transient
  - Consensused block content will be persisted via libretorrent app db. 
  - Each time DHT nodes start with zero information, the app will put into information to be announced. 
- Overall user experiece is text based torrent file info and messages. TAU app does not offer upload and download of video, it is up to each user using torrent client. TAU app is for publishing and coins circulation. 
- Provide miner optional manual approval function for admit transactions expecially the negative value and problem content. 
- dht is a public kv database. 
- URL: TAUchain:?bs=`hash(tau pk1 + salt)`&bs=`hash(tau pk1 + salt)`  // maybe 10 bs provided
      - dht_get(hash(tau pk1 + salt)) is the mutable item, which is a hash to immutable item Y.
      - dht_get(Y) return immutable item, which is block content (previous_hash, chainID, timestamp ...)

## Block content

```
blockJSON  = { 
1. version;
2. timestamp; 
3. BlockNumber;
4. PreviousBlockRoot; // for verification
5. ImmutablePointBlockRoot; // for voting, simple skip list
5. basetarget;
6. cummulative difficulty;
7. generation signature;
9. msg; // transaction content with ChainID, txType, content. 
10. ChainID
11. `TsenderTAUpk`Noune
12. `Tsender`Balance;
13. `TminerTAUpk`Balance;
14. `Treceiver`Balance;
15. TAUsignature;
}
```
## Constants
* MutableRange:  864 blocks, 3 days, use block as unit since no censensus
* WakeUpTime: sleeping mode wake up random range 10 minutes
* GenesisCoins: default coins 1,000,000. Integer, no decimals. 
* GenesisBasetarget:  0x21D0369D036978 ; 仿真100万个地址，平均出块时间60s
* DefaultMinBlockTime:  60 seconds, this is fixed block time. do not let user choose as for now.
* DefaultMaxBlockTime:  540 seconds, when no body mining, you have to generate blocks.
* DefaultBlockTime: 300 seconds

## Blockchain structure and processes
### Genesis 
```
// build genesis block
blockJSON  = { 
1. version;
2. timestamp; 
3. BlockNumber:=0; //创世区块号 0
4. PreviousBlockRoot = null; // genesis is built from null.
5. basetarget = 0x21D0369D036978;
6. cummulative difficulty int64; // ???
7. generation signature;
9. msg; // {genesis state k-v 初始账户信息列表} 
10. ChainID  
// genesis 0号区块，没有矿工奖励，余额都在初始状态表
11. signature;
}

```
---
## Mining Process: Votings, chain choice and block generation. 
This process is for multiple chains paralell execution.  
```
1. Pick up a random `chainID` in the Chains[] map.

2. If the (current time -  `ChainID` current-block time ) is bigger than DefaultMaxBlockTime, 
    go to (9) to use current safe root to generate the new current block 
    // not find new block than DefaultMaxBlockTime, everyonce is entitled to generate a block

3. Choose a Peer from  P:=TAUpeers[`ChainID`]
      If chainID+Peer is requested within DefaultBlockTime, go to (1) //不要对一个chainID+Peer 组合重复访问
    connect to P;
      if err go to (1)
      
4. DHT_get(`hash(TAUpk+chainID)`); 
    if not_found go to (1) 
    TAUpeers[ChainID][Peer].update(timestamp) // for counting the DefaultBlockTime

5. if root/PreviousBlockRoot/timestamp is later than present time, 不能在未来再次预测未来, aka n+2 , go to (1)

6. if received root and block shows a higher difficulty than current difficulty  {
    if the fork point out of the ImmutablePointBlock, go to (7) to collect voting. //如果难度更高的链在immutablePoint之前，则计入投票不做验证。
    verify this chain's transactions from the ImmutablePointBlock; // 任何验证只做immutable point之后的检查。
    if verification successful and populate new states
    go to (9) to generate new block; // found longest chain   
    }
   
7. // voting on low difficulty or forked chain.
   DHT_get(a random block in the MutableRange); //这里要考虑用simple skip list to increase reversal efficiency
   collected block is put into voting pool. 

8. If the present block number equal  VotesCountingPointBlocks[`ChainID`], {
    accounting all the voting results for that MutableRange to get the consensused block //统计方法是所有的root的计权重，选最高最新。
    calculate new ImmutablePointBlock[`ChainID`] and new  VotesCountingPointBlocks[`ChainID`]
   }
   clear voting pool to restart votes pool
   goto (1)

9. generate new block 
   if TAUpk on this chain balance is null; go to (1) // this is the follower
   generate currentBlockroot
      - contract
      - send, receive
      - coinbase tx
      - finish contract execution
   DHT_put()
   populate leveldb database variables. 

10. go to step (1)

```
## Controller: process manager and app initiation
1. When no database
  * create local level db.
  * with loca database 
    * dht_put(blocks)
2. When no TAU privatekey
  * generate seed and key pair
  * greate TAU key pair
  * load URLs of `TAU`, `Taut`  into level db.
3. check system resources and daily data consumption, start Mining process according system resources availability.
  * Mining process will follow TAU and TAUT
4. Writing local DB data into DHT kv db with time interval, random writing. 
<br/>
note: initial tau and taut data need to be manually populated into local DB and DHT 

---
## App UI 界面
### Blockchains - explore community and display mining info 区块链浏览器
- Community: community pkess, coin numbers, members number, magnet links numbers
    - Follow (mining) chains：mining data, power 链端
    - Unfollow chains those are recorded in the ANN message from the followed chains. 
    - Blacklist Chain
- User: balance, power
    - follow TAUpk
    - unfollow TAUpk
    - blackist TAUpk
### Messages - from followed chains, allow filter chains，选择框类似选择照片
  - magnet link display
  - ANN; include initial bootstrap (bs) for getting block info.
  - Messages
### Vidoes - Magnet links smart display
### Locator & Search
  - locate transaction URL
    - TAUtx:?blockroot=`hash`
  - locate chain URL
    - URL: TAUchain:?bs=`hash(tau pk1 + salt)`&bs=`hash(tau pk1 + salt)`  // maybe 10 bs provided
    - 搜索chainID URL自动归入followed chains.
  - internal search only
### System
  - Auto Start : ON, 2am every day.
  - Sync when you sleep: ON, 2am - 6am
  - Start when device start: ON
  - Auto stream waiting time: 5, 10, never
  - Roaming download: off
  - CPU alive: off
  - Battery limit: 15%, when below this turn off mining process B
  - TCP Proxy
  - Blockchain Storage
  - Connection limit: high water and low water
  
# To do 
- [ ] resource management process,
- [ ] re-announce management
- [ ] for community admin, provide pc and linux tool for hosting dht. 

