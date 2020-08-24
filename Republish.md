## Mutable data item includes two data components:
* the target content: the immutable data item key
* a referral public key: important for opitmizing the latest information searching
## Nodes re-publish immutable item protocol
According to the reading of `mutable request channel`, all nodes re-publish the immutable item if owned locally. If not owned, nodes will put public-key of the requesting node into own mutable referral public key and keep `target content` nil.  
``` 
Mutable data item does not follow re-publish protocol
```
## Salt channels
* Tip channel: hash link is the latest tip hash when blockchain grows, the tip could be own block or other miner's block. Node A publish a new block, A put block hash into mutable item, then publish both mutable and immutable item. 
* Requset channel: hash link is the content on demand. When A requests a block, A put a hash into the channel mutable data, then publish it.
* The `content` of mutable item in different channels: 
    * `blkTip` channel, the tip block 
    * `blkRequest`, the block hash on demand
    *
    * `msgTip` channel, the latest own message hash and lastest community msg pubkey
    * `msgRequest` channel, the msg hash on demand
    *
    * `txTip` pool channel, it the highest tx fee transaction. 
    * `txRequest` channel, the tx data schema hash on demand
    
## Mutable data sync mode
A node will control mutable data DHT put and get directly, running as synchronized tight coupling mode. A node will rely on referral to get intelligence for searching. 
## Immutable data a-sync life cycle: 
* When nodes A want to get a immutable data with a key, A will always search local memory. If not found, A will put the key into mutable data item and publish the mutable request. A will then exit the life cycle to leave DHT engine to do DHT get and add into local memory. 
* When other peer B read a mutable item from request channel, if B has such hash immutable content locally, the B will re-publish the immutable content; if not, B will put public-key of requesting node A into own mutable referral component, the content part is nil, then publish it. <br><br>
```
Everything relating to mutable data put/get and immutable data put is controled by application-self in sync mode; immutable get is controlled by TAU DHT middleware.
```

## Nature of data
* mutable item is new data: tight coupling controled by main app. 
* immutable item is old data: TAU dht engine will do get and load into memory for app. 
