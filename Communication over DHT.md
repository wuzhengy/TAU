# TAU communication on DHT
TAU adopts a loose-coupling communication to DHT engine with multiple sessions concurrency. This will give quick user response even the backend is unstable. DHT engine provides an access layer for D-DAG virtual space. However DHT in nature is only provide data integrity but not availability, we need to design protocol to enhance reliability. 
## Key to Key 
The IP protocol requires sender and receiver IP addresses. When IP address is behind NAT or in the private range, the connnection between devices is hard to establish. Ideally, each device will have public key. The communcation is conducted between key to key. The under-neath IP connection and routing is handled by protocol. 
We devide TAU server-less communicaiton to be an application layer protocol to faciliate peer to peer connection in following simple command:
* Gossip: own public key, data demand and message records (sender, receiver, type, hash, timestamp)
* Get: retrieve content from remote public key and exchange gossip information ( remote public key, type, hash, gossip vector)
User shall have no idea about 

The gossip concept is created to make each peer constantly in gossip state by exchange they observation of the swarm in terms of message transfer and demand state. Only when nodes see the gossip signal satisfy, they will put/get messages. 
The basic operations are put and get: these are for mining, **TBD**

1. `Put` / `Put and Forget`: When a node wants to put data item, it will call DHT to put data into network cache, and then move to next steps. The `put` action does not cause blocking and do not require response, since it does not need to care about whether data is really put or not. 
    * mutable data put: app will put mutable data item such as messages, blockchainTip or txTip when demanded.
    * immutable data put: app will put immutable data uppon other peers request. 
2. `Get` / `Demand and Forget`: TAU will `get` data, if un-successful, app can put the request into `gossip`, then delegate TAU DHT engine to get data from DHT and put into memory after "request" is servived hopefully by some peers. 
   * When a node, A, wants to get a data. It will search local memory firstly. If not found locally, A will post demand in gossip, get the key from DHT, if DHT reponse is nil, A will put the key in A's `gossip` channel, then move on, which is to leave DHT engine to publish. 
   * When aother node B reads from `gossip` channel, if B has such data locally, then B will put the data. <br><br>
The TAU DHT engine will setup a get unique queue to ensure the key uniqueness, so the request will not flood the system. 

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
Gossip is an idea that each peer will talk about the observation and demand, so that to pass the demand and message events around to facilate re-publish. 
## Gossip format
### Chat
In the chat function, each peer publish gossip to friend one by one when there is necessity. 
* mutable item key: salt("gossip"+"receiver pk", timestamp in 10 minutes); 
   * receiver pk is the full public key of target friend
   * pk_id is the last 4 bytes of a public key, to reduce the size of message. 4 bytes is good enough for each peer to find out peers. 
* value: 
```
sender X pk_id, receiver pk_id, "chainHash" or demand of immutable hash; timestamp }; 
sender Y pk_id, receiver's friend target pk2_id, "c" or d_hash;  timestamp in minutes  }; e.g { y67b, a6g8, "dm", timestemp in minutes }
   
* flow of chain
Both A and B are friends to each other, and A maintain a hash chain of messages A>B; B maintain a hash chain of messages B>A. For example, on A>B chain,
A will hash link B>A chain immutable as response to B's message "read" status. Therefore, A>B and B>A forms up a dual chain DAG. 
1. A gossip to everyone, in the gossip, there is the latest A>B_Chain_Root, random list of A's friends. 
2. B discover A gossip, then gossip to the world, in the gossip there is a demand of A>B_chain_Root and B>A_Chain_Root. One gossip data item can contain many demands and messages.
3. A received the demand, then put A>B chain data items
4. B sync up A>B chain data items. when B generate new data items, always link A>B chain's root. 
   
``` 
* Example: A -> B, Mutable item Salt = "gossip" + "Receiver B Peer's Public Key"
   * Assume in A friend list, public key peer list: A as defualt, A1, A2, A3, B, B2, C
   * Assume in B friend list, we have public key: B as defaulft, B1, B2, B3, A, A2
```
gossip - messages log with A's peers as `receiver`. 
      {
      A -> B, timestamp;
      A2 -> B, timestamp of A2 told other peers that A2 has sent info to B;
      A2 -> B2, timestamp; 
      D -> B, 
       } 
```
## Chat communication routine by peer main loop
Each public key peer will check friend's mutable item for gossip according to round robin.
Each public key also generate gossip when state update. 

A gossip#0
4bytes, 4 bytes, 20 bytes
A_pk_id, chainID, root
A, pk_id, 
A, pk_id 1, pk_id 2, 2
B, pk_id4, 4

29 x 33 =990
