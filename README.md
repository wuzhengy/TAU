# TAU - Messenger with high-scaling blockchain economy.
Core UI experienses:= {
- decentralized community chatting, not designed as personal messenger 
  - browser and small picture to present http links of youtube, instgram or tiktok
  - torrent client to open magnet link
- **"(+)"** floating button on home page
   * Create a Community 
      * Give a name to new blockchain 
      * Creation of a blockchain with 10 million coins at 5 minutes per block generation rate
- **"(+)"** floating button on community page
   * Transactions on community blockchain
      * Regular forum note and comments
      * New community Annoucement
      * DHT BootStrap Node Annoucement
      * Wiring Transaction
      * Identity annoucement
   * Messages on peer hash chain
      * Private message between two peers, content can be encrypted by receiver's public key.
        * only peers belong one community can send messages.
- Dashboard:  Data * Kb/s
  - Wifi only: on/off, default is ON.  
    - if "Wifi only" turn to Off, ask for how long: 30 minutes/ 1 hour(default)/ 3 hours
  - Internal config:
    - Charging ON: wake lock ON. 
    - Charging OFF: wake lock OFF. random wake up between 1..WakeUpTime
    - Internet OFF: wake lock OFF. random wake up between 1..WakeUpTime
- Chains prebuilt: TAUcoin chain: provides place to publish new community. App will read TAUcoin chain for app operation such as bootstrap DHT node.
- Find engine: app internal search for name and content
- TAU dev might provide DHT search engine: centralized engine to find new community and links.
--- 
## Persistence
1.  Chains  map[ChainID] config; 
2.  CurrentBlockRoot    map[ChainID]; // the map holding the recent blocks
3.  MutableRange    map[ChainID]uInt  // blocks in the mutable range is subject to change any time 
4.  Peers       map[ChainID]map[TAUpk]config;// for chains, when discovery new TAU peers, adding here with config info.
5.  SelfTxsPool         map[ChainID]map[TxHASH]config
6.  ImmutablePointBlock    map[ChainID] uInt 
7.  VotesCountingPointBlock   map[ChainID] uInt 

## Data flow: StateDB, BlocksDB and DHT
  - statedb is the local database holding account power and balance
  - blockdb is the local database holding the blocks content that will be put and get through DHT
  - DHT is the network key-value database, similar to cache
  - Data flow: memory <-> storage <-> cache
    - `blockdb` ---> `libtorrent put` ---> `DHT`
    - `blockdb` <--- `libtorrent get` <--- `DHT`
    - `statedb` <--> `blockdb`
  - myth: p2p communication is not possible to implement given firewalls and personal device security restrictions. DHT is used to be p2p communication substrate. When peer is not online, the content is still available to exchange. 
## Design Concepts
- Version 1 default parameters: 
  - 5 minutes average to generate a block. It can be upgraded when network infrastructure upgrading. 
  - One block has one transaction, which fits both DHT easy lookup and account state update. Lookup block is the same as transaction. This keeps DHT key value table simple. One block can include more transaction by encoding the hash of other transactions, which may be implmented in future version. 
- blockchain and hash-chain: blockchain belong to community consensus, hash-chain belong to personal instant chat messages on that blockchain(`salt`+'#' is the hash-chain ID). 
- Community ChainID := `community name`#`optional block time interval in seconds`#`hash(GenesisMInerPubkey + timestamp)` 
  - Community chain will choose its own name. 
  - Coin volumen is 10 million
  - Default block time is 300 seconds
  - example: TAUcoin ID is TAUcoin##hash; community ID: Shanghai#600#hash, which is a chain name Shanghai with 10 million coins and 600 seconds block time. 
- TAUpk: balance identifier under different chains; holds the power and perform mining. Seed generate privatekey and public key. In new TAU, we use seed to import and export account. 
- New POT use power as square root the nounce.
- genesis block power: give one year power to genesis to make admin airdrop possible. 
- 投票策略设计。For a new peer coming on-line, the peer uses voting to chose the right fork to follow. Voting is collecting a certain sample block prior to the mutable range. Mutable range is the range of blocks from the current block number to a specific history block number. 
   - 投票范围：定位投票点为当前ImmutablePointBlock位置。The voting peers are the peers in the mutable range.
   - 结束时，统计投票产生新的ImmutablePointBlock hash, 得票最高的当选, 同样票数时间最近的胜利。
      - 如果投出来的新ImmutablePointBlock, 这个block不在目前链的mutable range内，把自己当作新节点处理。检查下自己的历史交易是否在新链上，不在新链上的放回交易池。。
      - 新节点上线，快速启动随机相信一个能够覆盖到全部历史的链获得数据，设置链的（顶端- `MutableRange` ）为ImmutablePointBlock，开始出块做交易，等待下次投票结果。
      - 如果投票出的新ImmutablePointBlock，root在同一链上，说明是链的正常发展，继续发现最长链，在找到新的最长链的情况下，检查下自己以前已经上链的交易是否在新链上，不在新链上的放回交易池，交易池要维护自己地址交易。// 新的ImmutablePointBlock root不可能早于目前ImmutablePointBlock的。
   - 获得新root，如果是longest chain开始进入验证流程，如果不是进入计票流程.  
   - for a most difficult chain, folking within: 1x range, winner; 1x-3x, voting; 3x, alert.
- **libtorrent dht as storage and communication**
  * salt = chainID
  * immutable message = block content
  * mutable message's public key = TAUpk public key
  * mutable message by hash(public key + salt) = value is the block hash of TAU pkess future prediction on a chain
- immutable_item_dhtTAUget: 1. get one immutable item; 2. put a random immutable item from the same blockchain. 
- mutable_item_dhtTAUget: 1. get one mutable item; 2. put the same mutable item back into dht; the value of a mutable item is a hash pointing to a block
  - Get future block associated with a TAUpk + chainID (pub key + salt).  If the queried node has the block, it is returned in a key "values" as a list of strings. If the queried node has no such block, a key "nodes" is returned containing the K nodes in the queried nodes routing table closest to the hash supplied in the query.http://www.bittorrent.org/beps/bep_0044.html
- immutable and mutable item PUT: this is the same as mainline dht put.
  
- Provide miner manual approval function for admit transactions, expecially the negative value and problem content. 
- URL: TAUchain:?bs=`hash(tau pk1 + salt)`&bs=`hash(tau pk1 + salt)`  // maybe 10 bs provided
      - dht_get(hash(tau pk1 + salt)) is the mutable item, which is a hash to immutable item Y.
      - dht_get(Y) return immutable item, which is block content (previous_hash, chainID, timestamp ...)

## Block content

```
blockJSON  = { 
1. version;
2. timestamp; 
3. BlockNumber;
4. PreviousBlockHash; // for verification
5. ImmutablePointBlockHash; // for voting, simple skip list
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
```
1. get chain ID. 

2. If the (current time -  `ChainID` current-block time ) is bigger than DefaultMaxBlockTime and a valid transaction exist in tx pool
    go to (9) to generate a new block 

3. Choose a Peer from  P:=TAUpeers[`ChainID`]
      If chainID+Peer is requested within DefaultBlockTime, go to (1) // do not revisit same peer within block time
      
4. DHT_get(`hash(TAUpk+chainID)`); 
    if not_found go to (1) 
    TAUpeers[ChainID][Peer].update(timestamp) // for verifying the revisit time

6. if received root and block shows a valid higher difficulty than current difficulty  {
    if the fork point out of the ImmutablePointBlock, go to (7) to collect voting. //如果难度更高的链在immutablePoint之前，则计入投票不做验证。
    verify this chain's transactions from the ImmutablePointBlock; // 任何验证只做immutable point之后的检查。
    if verification successful and populate new states, new safety block identified. 
    }
   go to (9) 
   
7. // voting on low difficulty or forked chain. collecting all peers from mutable range for voting. 
   DHT_get(ImmutablePointBlockHash);
   collected block is put into voting pool. 

8. new safety block identified.
   clear voting pool to restart votes pool
   goto (1)

9. generate new block 
   if TAUpk not qualify POT requirment; go to (1) 
   generate currentBlockroot
      - contract
      - send, receive
      - coinbase tx
      - finish contract execution
   DHT_put
   populate leveldb database variables. 

10. go to step (1)

```

---
## User Interface: andriod app in firewall and linux cli on public server.
torrent client download - magnet link; browswer open - youtube, tiktok.
  - locate chain URL
    - URL: TAUchain:?salt=`salt`&pk=`public key 1`&pk=`public key 2`  // maybe 10 publickey provided
    - salt: chain_name#hash(genesis_public_key + timestamp)
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
# database: leveldb andriod
  
# To do 
- [ ] resource management process,
- [ ] re-announce management
- [ ] for community admin, provide pc and linux tool for hosting dht, and commandline tool for publish magnet link. Android app and linux cli. 

