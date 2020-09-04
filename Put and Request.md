# TAU app communication with DHT
### Mutable
Each peer will assume other peers will publish mutable data according to certain protocol. Therefore, peer does not request for mutable data, and peer will just get those data directly according to a time schedule or referral. 
### Immutable
Immutable data is history. Peers will publish these data uppon request. Therefore, peer need to request those data before getting them. 

TAU application adopts a special "one way" communication to DHT engine. This will give dAPP quick user response even the backend is very loose in a random DHT space. All app logical communication is implemented based on two  "one way non-waiting" methods:
1. `Put` / `Put and Forget`: When a node wants to put data item, it will call DHT Engine and then move to next steps. The `put` action does not cause waiting. 
    * mutable: app will put mutable data item such as blockchainTip according to mining blocktime or new longest chain identified. 
    * immutable: app will put immutable data uppon other peers request. 
2. `Request` / `Request and Forget`: app can put request immutable data through a certain `request channel`, then delegate TAU DHT engine to get data from DHT space. 
   * When a node, A, wants to get an immutable data. It will search local memory firstly. If not found locally, A will put the key in A's `Request` channel, then move on, which is to leave DHT engine to do `DHT get` for loading into local memory. 
   * When aother node B reads from request channel, if B has such data locally, the B will put the content. <br><br>
Therefore, the `get` method is replaced by request and callback. The TAU DHT engine will setup a get queue to ensure the key uniqueness, so the request will not flood the system. 

## Mutable item components
For optimizing community new data searching. Each mutable data item includes two components:
* the target content: the key of the target data, which could be either a requesting or a publishing of tip block or new message
* a referral public key: for optimization to O(logN) searching messages channel.  

## Salt channels
Each topic of blk, msg, tx has two mutable channels for both request and response.<br>
*  `blkTip` channel, the mutable item pointing to tip block. Content is the latest block hash when blockchain grows, the tip could be own block or other miner's block. Uppon receive request, Node A publish a new block via immutable item, A put block key into mutable item, then publish both mutable tip channel. 
  * referral component is nil
* `blkRequest` 
  * the history block with key of immutable
  * `tip` block with key of mutable:  e.g. peerXpubkey+chainID+blkTip
  * referral component is nil
 <br><br>
* `msgTip` channel, the mutable item pointing to latest own message hash and lastest community msg pubkey
* `msgRequest` channel, the msg hash on demand
  * the history msg with key of immutable
  * `tip` msg with key of mutable:  e.g. peerXpubkey+chainID+msgTip
  * referral component is the latest communicating peer.
 <br><br>
* `txTip` pool channel, it the highest tx fee transaction in the pool or own tx
  * referral component is nil
* `txRequest` channel, the tx data schema hash on demand
  * the history tx data with key of immutable
  * the top tx with key of mutable:  e.g. peerXpubkey+chainID+txTip
  * referral component is nil
```
Signal Types: 
    TIP_BLOCK_FROM_PEER_FOR_MINING, // mutable block - randomly getting community members tip of blockchain

    HISTORY_BLOCK_REQUEST_FOR_MINING, // immutable block - getting a history block by providing block hash
    HISTORY_BLOCK_TX_REQUEST_FOR_MINING, // immutable tx - getting a transaction under above block

    BLOCK_DEMAND_FROM_PEER, // mutable item: request block hash - receiving block hash under requesting by peers
    BLOCK_TX_DEMAND_FROM_PEER, // mutable item: request tx hash

   
    TIP_BLOCK_FROM_PEER_FOR_VOTING, // mutable block for voting
    HISTORY_BLOCK_REQUEST_FOR_VOTING, // immutable block for voting

    TIP_TX_FOR_MINING, // mutable tx for mining
    TX_FOR_MINING, // immutable tx for mining
```
