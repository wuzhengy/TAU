## Mutable data item include two data parts:
* a hash link: the target content
* a referral public key: important for msg channel to speed up searching new messages
## Nodes re-publish immutable item: according to the reading of `mutable item request channel`, all nodes re-publish the immutable item if owned locally. If not owned, nodes will put public-key of requesting node into own mutable date item referral and hash link is nil.  
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
## The request life cycle: 
* When nodes A want to request a history data, A will put the hash into mutable data item and publish the mutable data.
* When other peer B read a mutable item from request channel, if B has such hash immutable content locally, the B will re-publish the immutable content; if not, B will put public-key of requesting node A into own request mutable date item referral, the hash link part is nil. <br><br>

