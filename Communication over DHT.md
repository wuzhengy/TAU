# TAU communication on DHT
TAU application adopts a loose-coupling communication to DHT engine with multiple sessions concurrency. This will give quick user response even the backend is unstable. DHT engine provides an access layer for D-DAG virtual space:
1. `Put` / `Put and Forget`: When a node wants to put data item, it will call DHT Engine and then move to next steps. The `put` action does not cause blocking and do not require response code, since it does not need to care about whether data is really put or not. 
    * mutable: app will put mutable data item such as blockchainTip, according to blocktime or new longest chain identified. 
    * immutable: app will put immutable data uppon other peers request. 
2. `Demand` / `Demand and Forget`: app can put request immutable data through a certain `demand channel`, then delegate TAU DHT engine to get data from DHT space and put into memory. 
   * When a node, A, wants to get an immutable data. It will search local memory firstly. If not found locally, A will get the key from DHT, if DHT reponse is nil, A will put the key in A's `Demand` channel, then move on, which is to leave DHT engine to publish and `get` for loading into local memory. 
      * A 节点需要一个immutable item, 先从内存搜索，如果没有就从dht get, 如果返回nil, A 节点把这个需求放入demand频道，等待dht中间件寻找结果。
   * When aother node B reads from `demand` channel, if B has such data locally, then B will put the content. <br><br>
Therefore, the `get` method is replaced by request and callback. The TAU DHT engine will setup a get queue to ensure the key uniqueness, so the request will not flood the system. 

## Pub/Sub and P2p
Using DHT as loose coupling cache and blockchain as peer index, we are proposing new communication ways to implement pub/sub and p2p on top of pub/sub.
The classical peer to peer direct communication or through relay has problems to deal with firewalls and proxies. It also fails when peers getting offline. TAU uses community consensus as base for communication between peers or among group members. 
In each data item such as in a block, we will add two hash: vertical links and horizontal links. 
* vertical links: provide next 50 blocks immutable hash
* horizontal links: provide current time 50 messages immutable hash. 
At the same time, each data item is self-explain with type meta-info. 
The vertial and horizontal links serves as DAG with redundant connections on both depth or width.
### Pub/sub
A peer publishes value through mutable item key to announce a new block. Pub-key + ChainID + Blk. The same idea applys chatting. 
The new block publisher can also publish 50 immutable previous blocks into a mutable item. Pub-key+ChainID+blk+previous, X1 .. X50, for nodes to sync. 
This helps peers not on chain be able to access chain data for readonly purpose. 
Each put immutable item will always need get testing on dht to avoid flooding. 
* Since DHT does not garantee data availbility, different peers will see different things, some peers will get the data, some peers will not. Therefore, we use following way to establish "p2p over dht" communication. It uses batch data putting rather then one after one. It has potential to be hacked, but since key X is essentially a secrete, so third party will find hard to capture this request. 
### P2P over DHT pub/sub
On chain peers communicate to each other for demand and request, therefore the value is encrypted using receiver's pub-key. 
```
A requests data from B
1. A put immutable key X into mutable demand item via channel: chainID + blkDemand + timestamp ( value: immutable key X)
2. B put 50 immutable key X1 .. X 50 into mutable response item and send A, via: chainID + blkResponse + key X ( value: key X2 .. X50)
3. B put 50 immutbake data item into DHT space, before put always testing the data availability though get from dht
4. A receive B's mutable response item (X1..X50)
5. A get X1 .. X50
```
* If provide more than 50 items, it can add set1, set2, set3 into mutable items. 
### Mutable
Each peer will assume other peers will publish mutable data according to certain protocol and use `salt` to indicate the signal. Therefore, peer does not request for mutable data, and peer will just get those data according to a time schedule. This time schedule will related to how much bandwidth that a peer want to consume. 
<br>
Mutable data key includes public key and salt. In the salt, we put chainID, channel name, time slot(the valid time window for the message) and other protocol information. The goal is to increase the efficiency. For example, one peer publishs `demand for a range of blocks` and hope to get these data from the community. The peer needs to provide chainID, how many blocks needed and time slot. 

### Immutable
Immutable data in nature is the history data. Peers will publish these data uppon seeing the request. Therefore, peer need to request those data before getting them. 



## Salt channels
Each topic of blk, msg, tx has two mutable channels for `demand` and `publish`.<br>
*  `blkTip` channel, the mutable item pointing to tip block. Content is the latest block hash when blockchain grows, the tip could be own block or other miner's block. Node A publishs the new block via immutable item, A put block key into mutable item, then publish the mutable tip with timeslot. 
   * example: peerXpubkey+chainID+blkTip
* `blkDemand` 
   * the history block with key of the immutable item
   * example: peerXpubkey+chainID+Demand+target+timestamp
   * 每个服务节点会在收到demand后，先get下这个hash，如果可以获得就不提供服务。如果无法获得，就提供1个range的服务。把dht理解成cache，需求节点和服务节点都先检查cache中的数据。 
<br><br>
* `msgTip` channel, the mutable item pointing to latest own message hash
   * example. peerXpubkey+chainID+msgTip
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
### Timestamp
For tip data time, TAU does not use timestamp in salt to achieve higher availability of the data. 
For demand, TAU adds timestamp to make sure demand will be forgetten soon. 

## Peer Data contribution plan
In order for community members to receive data needed for mining, peers will check the `demand` channel for providing data to the community as contribution.
We use mutable range block number divided by active peers in a block cycle to decide how many data item to serve to the community demand for each peer. In the main loop, each iteration, peer will check whether the required put number fulfilled? If not, then keep on service, if fulfilled, then just skip the service. <br> 
Nodes can opt to service more data if the notes holding big stake or power. 

## Chat communication
Each public key peer will keep a personal channel with linked items, where it will keep contacts, user name, icon, joined communities. This is where each public key to keep its history.It is easier for user to move public key to another device and do sync-up cross different devices. 
### Personal Channel
* Salt = "Own Public Key"
   * mutable item: { userName; iconRoot; peerListRoot }
      * immutable item: peerListRoot: { peer, peerMsgRoot; peer 2, peer2MsgRoot; .. ; peer n, peernMsgRoot; previousPeerListRoot}
* peer will publish data through "own public key" channel, other peers will read this channel to find out connected peers and user name, etc. 
      
### Msg Channel

* A -> B, Mutable item Salt = "Receiver B Peer's Public Key"

```
   * Assume in A peerList, public key peer list: A as defualt, A1, A2, A3, B, B2, C
   * Assume in B peerList, we have public key: B as defaulft, B1, B2, B3, A, A2
 Data item from A to B: 
   { 
   immutable msgRoot; 
   gossip - messages log with B's peer list as participant sender or receiver. 
      {
      A -> B3, timestamp of A told other peers that A has sent info to B3; this is not the true observation of the message, 
            it is a gossip to help traverse the channels. 
      A2 -> B, timestamp of A2 told other peers that A2 has sent info to B;
      A2 -> B2, timestamp; 
      B3 -> A3, timestemp; A3 is not in B's peer list, but B3 is in B peer list.
      C -> B3, time stamp; C is not in B peer list, but C sent message to B3 which is in the B peer list, 
            this is the 2nd degree connection 
      }
   }
   ```
   * msgRoot= { msg of `A->B`; previousMsgRoot }
