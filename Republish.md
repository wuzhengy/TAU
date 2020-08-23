Immutable item republish. 
* According to the reading of `mutable item request channel`, all nodes re-publish the immutable item. 

Mutable item: mutable date item include two data items:
1. a hash link
2. a referral public key to recommend other peers to visit <br> <br>
Channel 1: tip: hash link is the node believed tip hash whenever block grows, the tip could be own block or other miner's block. Node A publish a new block, A put block hash into mutable ite, then publish both mutable and immutable item.  <br>
Channel 2: reqeuset: hash link is the content node wants. Node A request a block, A put a hash into mutable item, then publish it and wait to get it. <br>

The life cycle is: 
* When nodes want to request a history data, it will put the hash into mutable data item and publish the mutable data and DHT get the immutable item after a timeout.
* When other peer A read a mutable item, if A has such hash immutable content, the A will publish the immutable content; if not, A will make same request and DHT get the immutable item after a timeout. 
  - The `content` of item is the key of an immutable item that is latest knowledge learned in public domain by the node in past block time. 
    * `blkTip` channel, the tip block 
    * `blkRequest`
    * `msgTip` channel, the latest message 
    * `msgRequest` channel 
    * `txTip` pool channel, it the highest tx fee transaction. 

* other node B see a mutable item hash: 
  - case 1. B has such hash, then re-publish the immutable item . 
  - case 2. B does not has such hash, then request it via the request channel. 

