Nodes re-publish immutable item:  according to the reading of `mutable item request channel`, all nodes re-publish the immutable item if owned. If not owned, all nodes will request the same item.  

## Mutable data item include two data parts:
* a hash link
* a referral public key <br> <br>
## Salt channels
* Salt tip channel: hash link is the latest tip hash when blockchain grows, the tip could be own block or other miner's block. Node A publish a new block, A put block hash into mutable ite, then publish both mutable and immutable item. 
* Salt reqeuset channel: hash link is the content on demand. When A requests a block, A put a hash into the channel mutable data, then publish it and wait a time out constant to get it.
The `content` of mutable item: 
    * `blkTip` channel, the tip block 
    * `blkRequest`, the block hash on demand
    * `msgTip` channel, the latest message 
    * `msgRequest` channel, the msg hash on demand
    * `txTip` pool channel, it the highest tx fee transaction. 
## The request life cycle: 
* When nodes A want to request a history data, A will put the hash into mutable data item and publish the mutable data and DHT get the immutable item after a timeout.
* When other peer B read a mutable item from request channel, if B has such hash immutable content locally, the B will publish the immutable content; if not, B will make same request and DHT get the immutable item after a timeout. <br><br>

