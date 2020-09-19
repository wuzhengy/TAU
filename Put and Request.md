# TAU app communication with DHT
### Mutable
Each peer will assume other peers will publish mutable data according to certain protocol and use `salt` to indicate the signal. Therefore, peer does not request for mutable data, and peer will just get those data according to a time schedule. This time schedule will related to how much bandwidth that a peer want to consume. 
<br>
Mutable data key includes public key and salt. In the salt, we put chainID, channel name, time slot(the valid time window for the message) and other protocol information. The goal is to increase the efficiency. For example, one peer publishs `demand for a range of blocks` and hope to get these data from the community. The peer needs to provide chainID, how many blocks needed and time slot. 

### Immutable
Immutable data in nature is the history data. Peers will publish these data uppon seeing the request. Therefore, peer need to request those data before getting them. 

TAU application adopts a loose-coupling communication to DHT engine with multiple nodes concurrency. This will give dAPP quick user response even the backend is unstable. All app logical communication is implemented based on two  "one way non-waiting" methods:
1. `Put` / `Put and Forget`: When a node wants to put data item, it will call DHT Engine and then move to next steps. The `put` action does not cause waiting. 
    * mutable: app will put mutable data item such as blockchainTip, according to blocktime or new longest chain identified. 
    * immutable: app will put immutable data uppon other peers request. 
2. `Demand` / `Demand and Forget`: app can put request immutable data through a certain `demand channel`, then delegate TAU DHT engine to get data from DHT space and put into memory. 
   * When a node, A, wants to get an immutable data. It will search local memory firstly. If not found locally, A will get the key from DHT, if DHT reponse is nil, A will put the key in A's `Demand` channel, then move on, which is to leave DHT engine to publish and `get` for loading into local memory. 
   * When aother node B reads from `demand` channel, if B has such data locally, then B will put the content. <br><br>
Therefore, the `get` method is replaced by request and callback. The TAU DHT engine will setup a get queue to ensure the key uniqueness, so the request will not flood the system. 

## Salt channels
Each topic of blk, msg, tx has two mutable channels for `demand` and `publish`.<br>
*  `blkTip` channel, the mutable item pointing to tip block. Content is the latest block hash when blockchain grows, the tip could be own block or other miner's block. Node A publishs the new block via immutable item, A put block key into mutable item, then publish the mutable tip with timeslot. 
   * example: peerXpubkey+chainID+blkTip+TimeSlot
* `blkDemand` 
   * the history block with key of the immutable item
   * 每个服务节点会建立, unique queue, 来判断是否在5分钟，一个blocktime，内已经服务过这个immutable item，如果服务过，就不再重复向dht put这个数据。这个queue维持一个时间段内服务过的immutable item队列。 
<br><br>
* `msgTip` channel, the mutable item pointing to latest own message hash
   * example. peerXpubkey+chainID+msgTip+TimeSlot
* `msgDemand` channel, the msg hash on demand
   * the history msg with key of immutable
 <br><br>
* `txTip` pool channel, it the highest tx fee transaction in the pool or own tx
* `txDemand` channel, the tx data schema hash on demand
   * the history tx data with key of immutable

```
Signal Types: 
    TIP_BLOCK_FROM_PEER_FOR_MINING, // get from tip channel, mutable, for randomly getting community members tip of blockchain

    GET_HISTORY_BLOCK_FOR_MINING, // get from peers on immutable block - getting a history block by providing block hash
    GET_HISTORY_BLOCK_TX_FOR_MINING, // get from peers on immutable tx - getting a transaction under above block

    BLOCK_DEMAND_FROM_PEER, // get demand from peers, mutable item: request block hash - receiving block hash under requesting by peers
    BLOCK_TX_DEMAND_FROM_PEER, // mutable item: request tx hash
    
    GET_HISTORY_BLOCK_FOR_SYNC, // get from peers immutable block for sync
    GET_HISTORY_TX_FOR_SYNC, // get from peers immutable tx for sync

    TIP_BLOCK_FROM_PEER_FOR_VOTING, // mutable block for voting
    GET_HISTORY_BLOCK_FOR_VOTING, // immutable block for voting

    TIP_TX_FOR_MINING, // mutable tx for mining
    GET_TX_FOR_MINING, // immutable tx for mining
    
  
    TIP_BLOCK_FROM_PEER_FOR_VOTING, // mutable block for voting
    HISTORY_BLOCK_REQUEST_FOR_VOTING, // immutable block for voting

    TIP_TX_FOR_MINING, // mutable tx for pool
    TX_FOR_MINING, // immutable tx for pool  
```
## Peer Data contribution strategy
In order for community members to receive data needed for mining, peers will check the `demand` channel for providing data to the community as contribution.
We use mutable range block number divided by active peers in a block cycle to decide how many data item to serve to the community demand for each peer. In the main loop, each iteration, peer will check whether the required put number fulfilled? If not, then keep on service, if fulfilled, then just skip the service. <br> 
Nodes can opt to service more data if the notes holding big stake or power. 
