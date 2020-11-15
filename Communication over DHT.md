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
### Timestamp
For tip data time, TAU does not use timestamp in salt to achieve higher availability of the data. 
For demand, TAU adds timestamp to make sure demand will be forgetten soon. 

### Peer Data contribution plan
In order for community members to receive data needed for mining, peers will check the `demand` channel for providing data to the community as contribution.
We use mutable range block number divided by active peers in a block cycle to decide how many data item to serve to the community demand for each peer. In the main loop, each iteration, peer will check whether the required put number fulfilled? If not, then keep on service, if fulfilled, then just skip the service. <br> 
Nodes can opt to service more data if the notes holding big stake or power. 

## Gossip in the mutable data
The communication through DHT is not stable. There is no garantee of delivery. Gossip is an idea that each peer will talk about the observation, so that to relay the demand and message events. 
### Gossip format
#### Chat
* mutable item key: pk + salt("target pk"); value: 
```
demand{ sender* pk id target pk id + "profile"/"msg"/"immutableItem"; time stamp }; 
messageLog { (target pk friend 1 pk ID + "sender pk ID" + timestamp) ; 
demand{ sender* pk id target pk2 id + "profile/msg"/"immtutableItem" + timestamp) ... }
``` 
   * pk_id: 4 bytes shorter version of pk. 

* what they see regarding a target pk or chain
* what they want from a target pk or chain, this is demand<br><br>
in the network in its mutable data left-over space. For example, peer A put a mutable data item of its profile, in the remaining space, peer A will put into gossip info.<br>
In gossip, the pk key will be shortened to 4 bytes due to less critical than funds wiring. 

## Chat communication
Each public key peer will check friend's mutable item for demand and publish according to round robin and gossip info. For each peer, we have one `demand` channel for asking all kinds of information, we have peers, profile, msg channels to put information. A nil get will trigger demand put. 
### Demand is a type of gossip
* We use gossip as an channel. . Gossip information has relay nature among peers.<br><br>
Demand channel is maintained by each peer for own chat peers and each chains particiapted. Whatever data is not found will be put into demand, as well as gossip information. 
Each node will maintain a gossip pool in its own memory, logging its friends' communication history. <br>
#### Demand Example in Chat:
* mutable key:  pk + salt("gossip"); value: target pk + "peerlists"/"profile"/"**msgroot**"; immutable hash; gossip of sending data to pk's friends.
   * the reason we do not put target pk into salt is that we want gossiper to send this message quickly to target, rather than waiting target round robin.
#### Demand Exmaple in community:
* mutable key:  pk + salt("demand" + "chainID"); value: immutable hash1, hash2; gossip of missing data of mutable and latest sent data of blk/tx
When a node X send Y some mutable item, we will fill in gossip info to remaining space to help update Y's gossip pool for future making traversal decision. Therefore, a mutable item shall always be full <br>
Gossip data format: { sender; receiver; timestamp }
* A -> B, Mutable item Salt = "Receiver B Peer's Public Key" + "msg" 
   * Assume in A peerList, public key peer list: A as defualt, A1, A2, A3, B, B2, C
   * Assume in B peerList, we have public key: B as defaulft, B1, B2, B3, A, A2
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
