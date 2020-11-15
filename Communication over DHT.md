# TAU communication on DHT
TAU application adopts a loose-coupling communication to DHT engine with multiple sessions concurrency. This will give quick user response even the backend is unstable. DHT engine provides an access layer for D-DAG virtual space:
1. `Put` / `Put and Forget`: When a node wants to put data item, it will call DHT Engine and then move to next steps. The `put` action does not cause blocking and do not require response, since it does not need to care about whether data is really put or not. 
    * mutable: app will put mutable data item such as blockchainTip or txTip when demanded or own updated. when node not getting mutable item, it will demand mutable data item, due to peers does not follow time schedule to reannounce or reprovide. 
    * immutable: app will put immutable data uppon other peers request or own updated. 
2. `Demand` / `Demand and Forget`: app can put request mutable or immutable data through a certain `demand channel`, then delegate TAU DHT engine to get data from DHT space and put into memory after "demand" is servived hopefully by some peers. 
   * When a node, A, wants to get a data. It will search local memory firstly. If not found locally, A will get the key from DHT, if DHT reponse is nil, A will put the key in A's `Demand` channel, then move on, which is to leave DHT engine to publish and `get` for loading into local memory. 
   * When aother node B reads from `demand` channel, if B has such data locally, then B will put the content. <br><br>
Therefore, the `get` method is replaced by request and callback. The TAU DHT engine will setup a get queue to ensure the key uniqueness, so the request will not flood the system. 

## Pub/Sub and P2p
Using DHT as loose coupling cache and blockchain as peer index, we are proposing new communication ways to implement pub/sub and p2p on top of pub/sub.
The classical peer to peer direct communication or through relay has problems to deal with firewalls and proxies. It also fails when peers getting offline. TAU uses community consensus as base for communication between peers or among group members. 
In each data item such as in a block, we will add two hash: vertical links and horizontal links. 
* vertical links: provide next 50 blocks immutable hash
* horizontal links: provide current time 50 messages immutable hash for social auditing. 
At the same time, each data item is self-explain with type meta-info. 
The vertial and horizontal links serves as DAG with redundant connections on both depth or width.

### Mutable
Mutable data key includes public key and salt. In the salt, we put chainID, channel name, time slot(the valid time window for the message) and other protocol information. 
### Immutable
Immutable data in nature is the history data. Peers will publish these data uppon seeing the request. Therefore, peer need to request those data before getting them. 

## Re-announcement or Re-provide issue and its fix
In both IPFS and libtorrent, for certain data, the protocol will automatically ask peers to re-annouce or re-provide by put the data into dht space again. This is an awkward operation, firstly you do not know when data is required and what part of data is required, it engages some kind of constants to regualate such behavior, and it is hard to get right constants. <br>
TAU DHT will never re-provide, all TAU peers will put data into DHT only at following two ocations
* When new data is generated.
* When some peer request the data, after the peer has exausted to get data from dht space. 

## Salt channels
Each topic of blk, msg, tx has two mutable channels for `demand` and `publish`.<br>
*  `blkTip` channel, the mutable item pointing to tip block. Content is the latest block hash when blockchain grows, the tip could be own block or other miner's block. Node A publishs the new block via immutable item, A put block key into mutable item, then publish the mutable tip with timeslot. 
   * example: peerXpubkey+chainID+blkTip
* `blkDemand` 
   * the history block with key of the immutable item
   * example: peerXpubkey+chainID+Demand+target+timestamp
   * When a node receive demand, it will provide the data.  
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
#### Timestamp in salt
We use timestamp in salt to make sure peers getting latest mutable data. 

## Gossip through the mutable data
The communication on DHT is not stable. There is no garantee of delivery. 
Gossip is an idea that each peer will talk about the observation and demand, so that to pass the demand and message events around to facilate re-publish. 
## Gossip format
### Chat
In the chat function, each peer publish gossip to friend one by one when there is necessity. 
* mutable item key: salt("gossip"+"receiver pk"); 
   * receiver pk is the full public key of target friend
   * pk_id is the last 4 bytes of a public key, to reduce the size of message. 4 bytes is good enough for each peer to find out peers. 
* value: 
```
sender X pk_id, receiver pk_id, "profile"/"msgRequest"/"msgRoot"/"other signal"/immutableItem hash; timestamp }; 
sender Y pk_id, receiver's friend target pk2_id, "profile"/"msgReqeust"/"other signal"/immtutableItem hash; timestamp };
``` 
* Example: A -> B, Mutable item Salt = "gossip" + "Receiver B Peer's Public Key"
   * Assume in A friend list, public key peer list: A as defualt, A1, A2, A3, B, B2, C
   * Assume in B friend list, we have public key: B as defaulft, B1, B2, B3, A, A2
```
gossip - messages log with B's peers as `receiver`. 
      {
      A -> B, timestamp;
      A -> B3, timestamp of A told other peers that A has sent info to B3; this is not the true observation of the message, 
            it is a gossip to help traverse the channels. 
      A2 -> B, timestamp of A2 told other peers that A2 has sent info to B;
      A2 -> B2, timestamp; 
      C -> B3, time stamp; C is not in B peer list, but C sent message to B3 which is in the B peer list, 
            this is the 2nd degree connection 
      }
```
## Chat communication
Each public key peer will check friend's mutable item for gossip and publish according to round robin and gossip intelligence. 
### Demand is a type of gossip
* We use gossip to request sender to publish certain informaton, after we can not find any data value from get. One action data life cycle starts from get, then to demand if not found. 
Each node will maintain a gossip pool in its own memory, logging its friends' communication history. <br>

#### Demand Exmaple in community:  TBD
* mutable key:  pk + salt("demand" + "chainID"); value: immutable hash1, hash2; gossip of missing data of mutable and latest sent data of blk/tx
When a node X send Y some mutable item, we will fill in gossip info to remaining space to help update Y's gossip pool for future making traversal decision. Therefore, a mutable item shall always be full <br>


### Mutable items format
After B scanned A's QR code(public key), B start to post "demand" to A, then expect read from A's response, given A has B's QR code scanned into A's peer list as well.

1. profile: 2. B post mutable demand to A; A will post mutable response to `public`; A will publish these info automatically when update happens.
```
A publish: 
* Salt = "profile"
   * mutable item value: { userName; iconRoot; timestamp; peersRoot1; peersRoot2;...}
```     
2. message. B post mutable demand of msg to A; A will post message back to `B`; A will publish when update happens
```
* msg demand: {msg; gossip}
* msg mutable put actino: Salt = pk + "msg"
 Mutable Data item from A to B: 
   { 
   immutable msgRoot of A and B DAG history; 
   verticle roots
   }
   * immutable msgRoot= { verion; type(text,image); timestamp; contentRoot1..5; previousMsgDAGRoot}
```
