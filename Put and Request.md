# TAU app communication with DHT
TAU application adopts a special "one way" communication to DHT engine. This will give dAPP quick user response even the backend is very loose and random DHT space. The business logic is implemented based on two  "one way non-waiting" methods:
1. `Put` / `Put and Forget`: app can put data item (mutable/immutable) into DHT space. When a node wants to put either mutable or immutable item, it will call DHT Engine and then forget, which is moving to next steps. The `put` action does not cause waiting. 
2. `Request` / `Request and Forget`: app can put request through `mutable item` through a certain `request channel`, then delegate TAU DHT engine to get data from DHT space. 
   * When a node, A, wants to get a data 
      - for immutable key, A will search local memory firstly. If not found locally, A will put the key in A's `Request` channel, then move on, which is to leave DHT engine to do `DHT get` for loading into local memory. 
      - for mutable key, A will not search local. A will put the key such as `pubkeyChainIDblktip` in A's `request` channel, then DHT get will seek it and call back the A. 
   * When aother node B reads from request channel, if B has such data locally, the B will put the content. <br><br>
Therefore, the `get` method is replaced by request and callback. The TAU DHT engine will setup a get queue to ensure the key uniqueness, so the request will not flood the system. 
### Request based communication than `constant re-publishing`
Nodes routinely checking community members requests and response with putting data. In mainline DHT and IPFS, it requires nodes to republish content periodically to keep data live in the DHT space. For larger amount of data, it is impossible to achieve all data holding on DHT. DHT space is mostly for communication caching rather than archiving.
```
The Non-waiting design of `request` and `put` will allow messenger to communicate to DHT freely without worrying the bandwidth and latency. 
```
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
