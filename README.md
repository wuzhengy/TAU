# TAU - Decentralized communication with high-scaling blockchain economy.
Core UI experienses
- decentralized community social hub with crypto-coins circulation
- **"(+)"** floating button on home page
   * Create a Community 
      * Give a name to new blockchain and its coin
      * Creation of a blockchain with 10 million coins on 5 minutes per block generation rate
- **"(+)"** floating button on community page
   * Transactions on community blockchain - DHT mutable key: hash(public key + `chainID#blk`)
      * Regular forum note and comments
      * New community annoucement
      * DHT BootStrap Node Annoucement
      * Wiring Transaction (logN as pool upper limit)
      * Personal Identity annoucement
      * Future: Community name and configuration change by majority stake vote. - phase 2
   * Off-chain Peek Messages - DHT mutable key: hash(public key + `chainID#msg`)
      * Messages are coded as key(mutable item key) to value(crypto-graphic addressed message)
        - key: includes sender(public key), target hub(community or chainID)
        - value: includes recevier(public key), signed and encrypted messages depends on specific application
      * Peers can display peek messages and hope other peers to find it
      * Type of messages
        - invitation for chatting, phase 1 ??? what is invitation.
        - tx candidates, phase 1, each account will provide tx candidates through this channel, and pointer to a recent changed address/pk. (n+1 approach is removed)
        - secure p2p message, phase 1
        - transaction pool light update channel, phase 2
   * Decentralized blacklist - easy to blacklist an address from client
- Dashboard:  Wifi Data * Kb/s; Telecom Date * Kb/s; DHT nodesinfo
- Chains prebuilt: TAUcoin chain: provides place to publish news and ads. App will read TAUcoin chain for app operation config such as bootstrap DHT node.
- Find engine: app internal search for name and content
--- 
## Persistence
1.  Chains  map[ChainID] config; 
2.  CurrentBlockRoot    map[ChainID]; // the map holding the recent blocks
3.  MutableRange    map[ChainID]uInt  // blocks in the mutable range is subject to change any time 
4.  Peers       map[ChainID]map[TAUpk]config; // for chains, when discovery new TAU peers, adding here with config info.
5.  SelfTxsQueue         map[ChainID]map[TxHASH]config // account's own transactions history
6.  ImmutablePointBlock    map[ChainID] uInt 

## Data flow: StateDB, BlocksDB and DHT
  - statedb is the local database holding account power and balance
  - blockdb is the local database holding the blocks content that will be put and get through DHT
  - DHT is the network key-value cache
  - Data flow: memory <-> storage <-> cache
    - `blockdb` ---> `libtorrent put` ---> `DHT`
    - `blockdb` <--- `libtorrent get` <--- `DHT`
    - `statedb` <--> `blockdb`
  - myth: p2p IP level communication is not possible to implement, given firewalls and personal device security restrictions. DHT is adopted to be the overlay p2p communication substrate. When peer is not off-line, the content is still available in DHT cache for exchange. 
## Design Concepts
- Version 1 default parameters: 
  - 5 minutes average to generate a block. It can be upgraded when network infrastructure upgrading. 
  - One block has one transaction, which fits both DHT easy lookup and account state update. Lookup block is the same as transaction. This keeps DHT key value table simple. One block can include more transaction by encoding the hash of other transactions, which may be implmented in future version. 
- blockchain and hash-chain: blockchain reflects to community consensus, hash-chain stores personal instant chat messages on one blockchain. The format looks like, community ID: Shanghai#600#hash; hash chain salt: Shanghai#600#hash`#`
- Community ChainID := `community name`#`optional block time interval in seconds`#`hash(GenesisMInerPubkey + timestamp)` 
  - Community chain will choose its own name. 
  - Coin volumen is 10 million
  - Default block time is 300 seconds
  - example: TAUcoin ID is TAUcoin##hash; community ID: Shanghai#600#hash, which is a chain name Shanghai with 10 million coins and 600 seconds block time. 
  - ChainID is the salt in DHT mutable put
- TAUpk is the crypto address: balance identifier under different chains; holds the power and perform mining. "Seed" generates privatekey and public key. In new TAU, we use "seed" to import and export account identifier. 
- New POT defines power as square root of the nounce.
- genesis block power: give one year power to genesis public key to make admin airdrop possible. 
- Voting process: for a new peer coming on-line, the peer uses a voting process to chose the right fork to follow. Voting is collecting a specific block, which is the "immutable point block" from block producers within the mutable range. Mutable range is the range of blocks from the current tip to a specific history block. 
   - voting position: The ImmutablePointBlock, the voting peers are the peers in the mutable range.
   - select the highest voted ImmutablePointBlock hash
      - 新节点上线，快速启动随机相信一个能够覆盖到mutable range的链，获得数据，设置链的（顶端- `MutableRange` ）为ImmutablePointBlock。
      - 如果投票出的新ImmutablePointBlock，root在同一链上mutable range内，说明是链的正常发展
        - 继续发现最长链，在找到新的最长链的情况下，检查下自己以前已经上链的交易是否在新链上，不在新链上的放回交易池，交易池要维护自己地址交易。
      - 如果投出来的新ImmutablePointBlock, 这个block is within 1x - 3x out of mutable range，把自己当作新节点处理。检查下自己的历史交易是否在新链上，不在新链上的放回交易池。如果分叉点在3x mutable range之外，Alert the member of potential attack。  
        - for a most difficult chain, folking within: 1x range, winner; 1x-3x, voting; 3x, alert.
- **libtorrent dht as cache and communication**
  * salt = chainID + `#` + `channel` ; blk is a channel for blocks, such as `shanghaichain#blk`
  * immutable item's value is the block content
  * mutable item's signature public key = TAUpk public key
  * mutable item's key is hash(public key + salt), value is the hash/key of immutable item 
- immutable_item_dhtTAUget: 1. get one immutable item; 2. put back a random immutable item from the same blockchain to keep blockchain data life. 
- Provide opitional miner manual approval on transactions, expecially the negative value and problem content. 
- URL: TAUchain:?bs=`hash(tau pk1 + salt)`&bs=`hash(tau pk1 + salt)`&dn=`chainID`  // maybe 10 bs provided
- Mining and following a community: in TAU, there is no different for this two concept. After voting, POT requires reading to valid chain on its own knowlege.
- Adding a new member into a communtity: 
  - Share the chain link to member via telegram or wechat, ...
  - Send off-chain invite messages to member via community routing. 
- One secrete key per device, not recommend to copy secrete key between devices. 
- power/balance(0/0) = read only. 
- "inviting" -> "connected": inviting link has to include both A and B's public key. inviting link generation with a new salt defining the chatting group. salts will be communication channel.  ????
  - when peer A add peer B's public key into contact list, status of B will show "inviting"; and A will publish "signaling B" on own zero salt #msg channel; A will try to ask B to add A's public key into B's contact list as well through inviting link. 
    - if A and B belong to same community, the status will turn "connected" on B
    - if A reads B's zero salt #msg channel "signaling A", the status will turn "connected" on B
    - A will give B through 3rd party channel A's public key for adding. 
- p2p messaging via logN strategy. 
   - each p2p message include a hash pointing to another public key recently messaging the same receiver. it is like "salt in the zero salt channel" 
  - in each TAU message either immtuable or mutable, there are hash point in content to make search N complexity to log(N). this is a type of DQ algorithm.

## Block content
```
blockJSON  = { 
1. version;
2. timestamp; 
3. BlockNumber;
4. PreviousBlockHash; // for verification
5. ImmutablePointBlockHash; // for voting, simple skip list
6. basetarget;
7. cummulative difficulty;
8. generation signature;
9. msg; // transaction content with ChainID, txType, content. 
10. ChainID
11. `TsenderTAUpk`Noune
12. `Tsender`Balance;
13. `TminerTAUpk`Balance;
14. `Treceiver`Balance;
15. ED25519 public key as TAUaddress
16. ED25519 signature
}
```
## Constants
* MutableRange:  864 blocks, 3 days, use block as unit since no censensus
* WarningRange: 3 x MutableRange
* WakeUpTime: sleeping mode wake up random range 10 minutes
* GenesisCoins: default coins 10,000,000. 
* GenesisBasetarget:  0x21D0369D036978; 仿真100万个地址，平均出块时间60s
* DefaultMinBlockTime:  60 seconds, this is fixed block time. do not let user choose as for now.
* DefaultMaxBlockTime:  540 seconds, when no body mining, you have to generate blocks.
* DefaultBlockTime: 300 seconds
* TXtimeout: 12 Hours

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
11. ED25519 public key as TAUaddress
12. ED25519 signature
}

```
---
## Mining Process sketch: Votings, chain choice and block generation.   
```
1. get chain ID. 

2. If the (current time -  local `ChainID` tip block time ) is bigger than DefaultMaxBlockTime and a valid transaction exist in own tx queue;
   go to (9) to generate a new block 

3. Choose a Peer from  TAUpeers[`ChainID`]
   If chainID+Peer is requested within DefaultBlockTime, go to (9) // do not revisit same peer within default block time
      
4. DHT_get(`hash(TAUpk+chainID)`); get `ChainID` tip block from DHT;
   if not_found go to (9);
   TAUpeers[ChainID][Peer].update(timestamp) // for verifying the revisit time

6. if received block shows a valid higher difficulty than current difficulty  {
    if the fork happens prior to the ImmutablePointBlock and within the WarningRange, 
    go to (7) for voting. 
    if the fork happens piror to the WarningRange, throw warning to user, go to (9)
    verify this chain's transactions from the ImmutablePointBlock;
    if verification successful and populate new states, new longest block identified. 
    }
   go to (9) 
   
7. collecting all peers from mutable range for voting. 
   DHT_get(ImmutablePointBlockHash);

8. new ImmutablePointBlock voted.
   goto (1)

9. if TAUpk not qualifies POT requirment; go to (1) 
   generate new block
   put into DHT
   populate leveldb database. 

10. go to step (1)

```
---

# System config
  - Auto start when device start: ON
  - Wifi Only: ON, when turn off, it will ask for time to allow telecom data operating
  - Charging ON: wake lock ON. 
  - Charging OFF: wake lock OFF. random wake up between 1..WakeUpTime to check status 
  - Internet OFF: wake lock OFF. random wake up between 1..WakeUpTime to check status
  - Server mode: OFF, when turn ON, it will turn on wake lock. 

# database: leveldb andriod
# muliple platform support on android, ios, pc, chromeOS, macOS, linux ...
  Since `secrete key` is supposed to be on only on device for operation, we focus on use android as core platform. Other platform will use browser to connect andoid TAU app to run TAU functions. As long as other devices can access the android phone via IP network, they can operate a TAU node. 
  Linux dht command line interfact will be provided to interact with TAU. 
  Android platform is a great decentral and light OS, given the open-source community support on many components such as leveldb, dht, libtorrent for java, libretorrent and many google UI components. 
  
# To do and other notes
- [ ] resource management process
- [ ] multi-platform
- [ ] peer to peer secure private messages, community DHT becomes a router for peer to peer to send and confirm secure messages.  
- [ ] TAU dev might provide centralized DHT search engine: centralized engine to find new community and links.
- transaction collection strategy of a miner: as soon as getting a transaction with fee higher than median fee, capture it with a new block generation. 
- off-chain message collection: active users having higher change to be displayed off-chain messages. 
- transaction pool light update channel
