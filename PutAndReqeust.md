# TAU app communication with DHT
TAU application adopts a special "one way" communication to DHT engine. The business logic is implemented based on two  "one way non-waiting" methods:
1. `Put` / `Put and Forget`: app can put data item (mutable/immutable) into DHT space. When a node wants to put either mutable or immutable item, it will call DHT API directly and then forget, which is moving to next steps. The `put` action does not cause waiting. 
2. `Request` / `Request and Forget`: app can put request through `mutable item` through a certain `Salt channel`, then delegate TAU DHT engine to get data from DHT space. 
   * When a node, A, wants to get a data with either a mutable or immutable key, A will search local memory firstly. If not found locally, A will put the key into mutable item and publish in the `Request` channel. A will then move on, which is to leave DHT engine to do `DHT get` for loading into local memory. 
   * When aother node B reads a mutable item from request channel, if B has such data locally, the B will put the content; if not, B will put public-key of requesting node A into own mutable referral component, the content part is nil, then put. This helps broadcasting the original request. <br><br>

### Request based communication than re-publishing
Nodes routinely checking community members requests and response with putting data. In mainline DHT and IPFS, it requires nodes to republish content periodically to keep data live in the DHT space. For larger amount of data, it is impossible to achieve this holding all data on DHT. DHT space is mostly for communication caching rather than history archive. 
We believe that request-based put is more fitting for DHT. 

## Mutable item
For optimizing community new data searching, mutable data item of DHT is used as both putting and requesting data item. Each mutable data item includes two components:
* the target content: the key of the target data, which could be either a requesting or a publishing of tip block or new message
* a referral public key: for opitmizing the latest information searching, the public key is at nodes best knowledge of the latest peer that communicates. The referral will potentially speed up searching process to Log(N) through community group effort. 

## Salt channels
Each topic of blk, msg, tx has two mutable channels for both request and response.<br>
*  `blkTip` channel, the mutable item pointing to tip block. Content is the latest block hash when blockchain grows, the tip could be own block or other miner's block. Uppon receive request, Node A publish a new block via immutable item, A put block key into mutable item, then publish both mutable tip channel. 
* `blkRequest` 
  * the history block with key of immutable
  * `tip` block with key of mutable:  e.g. peerXpubkey+chainID+blkTip
 <br><br>
* `msgTip` channel, the mutable item pointing to latest own message hash and lastest community msg pubkey
* `msgRequest` channel, the msg hash on demand
  * the history msg with key of immutable
  * `tip` msg with key of mutable:  e.g. peerXpubkey+chainID+msgTip
 <br><br>
* `txTip` pool channel, it the highest tx fee transaction. 
* `txRequest` channel, the tx data schema hash on demand
  * the history tx data with key of immutable
  * the top tx with key of mutable:  e.g. peerXpubkey+chainID+txTip
