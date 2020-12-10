# TAU communication on DHT
TAU server-less communicaiton is an application level protocol based on libtorrent DHT. Libtorrent DHT built a network of node_id based decentral communcation. TAU adds public_key based communication. Public key rides on nodes to make communication. 
TAU adopts a loose-coupling communication to DHT engine with multiple sessions concurrency.
In each session, we use both dht_direct and dht_recursive. We start with direct udp bencode get the data from remote public key's node, if the response does not come back in 1 second, we will start the dht recursive mode, which takes longer time and more data. This will give quick user response even the backend is unstable. DHT cloud become relay services without dedicated servers. The DHT relay service provide external IP and port discovery to all peers.   
DHT engine provides an access channel for D-DAG virtual space. However D-DAG in nature is only provide data integrity but not availability.
## Public Key to Public Key 
The IP protocol requires sender and receiver IP addresses. When IP address is behind NAT or in the private range, the connnection between devices is hard to establish. Ideally, each device will have public key. The communcation is conducted between key to key. The under-neath IP connection and routing is handled by protocol. 
We devide TAU server-less communicaiton to be an application layer protocol to faciliate peer to peer connection in following simple command:
* Get: both dht_direct and dht_recursive retrieve content from remote public key and exchange gossip information (remote public key, type, hash, gossip vector) `Get`: TAU will `get` data directly then recursively, if un-successful, app can put the request into `gossip` servived hopefully by some peers. 
   * When a node, A, wants to get a data. It will search local memory firstly. If not found locally, A will do the `get`, if DHT reponse is nil, A will put the key in A's `gossip` channel, then move on. 
   * When aother node B reads from `gossip` channel, if B has such data locally, then B will put the data. <br><br>
The TAU DHT engine will setup a get unique queue to ensure the key uniqueness, so the request will not flood the system. 
* Put: only DHT_recursive
  * Immutable data in demand
  * Mutable data channel with salt 'Gossip' : through mutable item or dht_direct get/response, gossip will annouce own nickName, data demand and message records 
      * The gossip concept is created to make each peer constantly in gossip state by exchange their observation of the swarm in terms of message and demand. This will increase the network efficiency. 
  * `Put` / `Put and Forget`: When a node wants to put data item, it will call DHT recursively to put data into network cache Non-blocking, and then move to next steps. The `put` action does not cause blocking and do not require response, since it does not need to care about whether data is really put or not.

## DHT and DDAG
TAU data is permanently stored in the distributed dag, which is spread among many different phones and has noting to do with DHT. 
DHT is an access and cache layer to operate DDAG data for blockchain and messenger to use. DHT temporarily load portion of dag data into cache for app to use and also serve as communication laywer among peers collectively storing DDAG. 

## Mutable
In the salt, we put friend pk_id, chainID, channel name, time slot(the valid time window for the message) and other protocol information to form up mutable data item key, on which we will build many commication protocol support blockchain and chat.  
### Mutable items format in chat
After B scanned A's QR code(public key), B start to post gossip to A, then expect read from A's response, given A has B's QR code scanned into A's peer list as well.
1. gossip: see later discussion
2. message. A post message back to B, A will also publish related immtuable data such as msg and contentRoot1..5
```
*  Salt = "msg" + "receiver pk" + timestamp in 10 minutes
 Mutable Data item from A to B: 
   { 
   username: string; this is where user can change name.
   immutable msgRoot .. n: include current message root and previous ones to help B get information faster.
   contentRoot1..5
   }
   * immutable msgRoot= { verion; type(text,image); timestamp; contentRoot1..5; previousMsgDAGRoot}
```

### Immutable
Immutable data in nature is the history data. Peers will publish these data uppon seeing the request. Therefore, peer need to request those data before getting them. 

## Re-announcement or Re-provide issue and its fix
In both IPFS and libtorrent, for certain data, the protocol will automatically ask peers to re-annouce or re-provide by put the data into dht space again. This is an awkward operation, firstly you do not know when data is required and what part of data is required, it engages some kind of constants to regualate such behavior, and it is hard to get right constants. <br>
TAU DHT will not do static re-provide, all TAU peers will put data into DHT only at following two ocations
* When publish gossip
* When some peer request the data through gossip

## Salt channels
Each topic of blk, msg, tx has mutable channels for publishing <br>
* `blkTip` channel, the mutable item pointing to tip block. Content is the latest block hash when blockchain grows, the tip could be own block or other miner's block. Node A publishs the new block via immutable item, A put block key into mutable item, then publish the mutable tip with timeslot. 
   * example: peerXpubkey+chainID+blkTip
* `msgTip` channel, the mutable item pointing to latest own message hash
   * example. peerXpubkey+chainID+msgTip
* `txTip` pool channel, it the highest tx fee transaction in the pool or own tx

```
Signal Types for mining: 
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
Gossip is an idea that each public key will talk about its observation and own demand, so that to pass around to facilate data put and get. 
## Gossip format
### Chat
In the chat function, each public key gossips on one channel. 
* mutable item key: salt("gossip"); 
  * value: (sender, receiver, type, hash, timestamp)
    * pk_id is the last 4 bytes of a public key, to reduce the size of message. 4 bytes is good enough for each peer to find out peers. 
* example: 
```
sender X pk_id, receiver pk_id, "chainHash" or demand of immutable hash; timestamp }; 
sender Y pk_id, receiver's friend target pk2_id, "c" or d_hash;  timestamp in minutes  }; e.g { y67b, a6g8, "dm", timestemp in minutes }
   
* flow of chain
Both A and B are friends to each other, and A maintain a hash chain of messages A>B; B maintain a hash chain of messages B>A. For example, on A>B chain,
A will hash link B>A chain immutable as response to B's message "read" status. Therefore, A>B and B>A forms up a dual chain DAG. 
1. A gossip if new root for B
2. B discover A gossip, get from A
3. A may received the demand from B, then put A>B chain data items
4. B sync up A>B chain data items. when B generate new data items, always link A>B chain's root. 
   
``` 
* Example: A -> B, Mutable item Salt = "gossip"
   * Assume in A friend list, public key peer list: A as defualt, A1, A2, A3, B, B2, C
   * Assume in B friend list, we have public key: B as defaulft, B1, B2, B3, A, A2
```
gossip - A's friends as `receiver`. 
      {
      A -> B, ...;
      A2 -> B2 ...; 
      D -> B ..., 
       } 
gossip - B's friends as receiver
      {
      A2 -> B2
      E -> B3
      }
```
## Chat communication routine by peer main loop
Each public key will check friends gossip according to round robin.
Each public key also generate gossip if state changes. 
## DHT middle tier
annouce own public key each sessions CIDR and seek other public key's CIDR for dht direct.  middle tier need to build own channel for CIDR info discovery.
For a public key's own NAT or connected other public key, either one of them will be non-symetric, because symetic can not connect to symetric.

## gossip types in peer to peer texting
### from UI layer
* signal a message that has sent to friend with its MsgDAGRoot
* signal current gossip frequency,1, 5, 10, 30, 60, so other peers can form up mutable salt to get this gossip quicker. this frequency hope to increase the performance. also when user has message to send, it should gossip immediately. 
* demand some immutable data item
### from chain layer
* demand some immutable data item
* put gossip according to freqence
* checking friends msgDAGroot update and make sure both side recorded other's public key. for the friends not 100% complete, it will send out gossip item request gossip answer from other peer.(50% never connected, connected, last msg, last seen )
