# TAU communication on DHT
TAU server-less communicaiton is an application level protocol based on libtorrent DHT. Libtorrent DHT built a network of node_id based decentral communcation. TAU adds public_key based communication. Public key overides on nodes IP to make communication. DHT cloud become relay services without dedicated servers. 
libTAU - serverless communication, an open source Java library for unblockable p2p(pubkey to pubkey) and blockchain messaging.

------

Public Key
* 32 bytes ED25519 Pubkey Key generated from random seed

Node ID
* First 160 bits of the public key

Distrubted Routing Table
* Tuple: Node ID, IP, Port
* Meta data: last seen, last communicated, failure counter
* May consider multiple layers to make find_nodes better fit, 80 layers

------

Interface to app developer
* Java package JAR with embeded c++ swig structure for x86_64 and arm64 ABI
* SQLite as "server-less filesystem based db" provides configuration and message exchange interface between third party and libTAU, so that third party code can work with libTAU in the same process. 
* Provide blockchain time and bootstrap than third partis time and bootstrap server
* Traffic consumption calculation based on protocol main-loop estimation than from hardware interface. Taffic info is important in TAU to save cost for devices. 
------

Mutable Data Cache bucket-tree
* Target: 20 bytes
  * first half of the sender Node ID must match 2nd half of the target to be qualified to sign the value
* Value: 1000 bytes
  * sender X need to sign this value  
* Ping: key-value (optional)
  * any random sender can update this key-value pair without signature, just like ping service
  * sender public key - 32 bytes, value - 32 bytes. 

Target of Mutable Data: libTAU mutable data aims to exchange data than storage, expecting lots of records overlaping like in the routing table
* 160 bits long
* First 80 bits: First half of the receiver Node ID
  * If the receiver has public IP and online, the data will be put into receiver memory directly. This design is to create incentive for data relay provider to get a public IP/port and keep alive. This is also why we **do not** hash (salt + pubkey). 
  * The more data provided, the provider's Node ID has more places in other peers routing table
* Second 80 bits: First half of the sender Node ID
  * when first half equal to second half, this is a self data channel and possible to receive any sender's ping message and update the appendix value. This could be used to add anonymous friends. 
  * only sender Node ID has first 80 bits matching Target second half can sign value field. 

---
Data consumption
* each device with libTAU need to decide daily data usage for achieving balance of contribution and benefits. Generally the more data allocated, the better performance it is. 

Walking frequency
* each device could setup the range of walking frequency from 1 - 20s, this will also limit the highest data consumption. 

---
Bootstrap and time: nodes can get these information from both central and decentral sources 
* from third party bootstrap and time server such as ISP or TAU Dev.
* from community blockchain content, libTAU can config serveral community chains to start follow.
  * blockchain content is safer to validate true time and right phone swarm, however it is slower than third party service. So we adopt a combined approach with blockchain as part of statistical calculation. 
  * all the added blockchains in the friends list will be treated as boot and time info potential providers equivalent to TAU chain.
---
Encryption
* use receiver's public key, it is easy to encrypt all messages relaying to receiver in full UDP packet. 
* relaying nodes can sign the message use own private key, so that receiver knows who relays the messages, in which could be other mesage sender. 


## Public Key to Public Key 
The IP protocol requires sender and receiver IP addresses. When IP address is behind NAT or in the private range, the connnection between devices is hard to establish. Ideally, each device will have public key. The communcation is conducted between key to key. The under-neath IP connection and routing is handled by protocol. 
We devide TAU server-less communicaiton to be an application layer protocol to faciliate peer to peer connection in following simple command:
* Get
   * When a node, A, wants to get a data. It will search local memory firstly. If not found locally, A will do the `dht_get` 
* Put: only DHT_recursive
  * The gossip concept is created to make each peer constantly in gossip state by exchange their observation of the swarm in terms of message and demand. This will increase the network efficiency. 
  * `Put` / `Put and Forget`: When a node wants to put data item, it will call DHT recursively to put data into network cache Non-blocking, and then move to next steps. The `put` action does not cause blocking and do not require response, since it does not need to care about whether data is really put or not.

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


## Salt channels
Each topic of blk, msg, tx has mutable channels for publishing <br>
* `blkTip` channel, the mutable item pointing to tip block. Content is the latest block hash when blockchain grows, the tip could be own block or other miner's block. Node A publishs the new block via immutable item, A put block key into mutable item, then publish the mutable tip with timeslot. 
   * example: peerXpubkey+chainID+blkTip
* `msgTip` channel, the mutable item pointing to latest own message hash
   * example. peerXpubkey+chainID+msgTip
* `txTip` pool channel, it the highest tx fee transaction in the pool or own tx


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
* frequency: each node will indicate the current gossip frequency via mutable data item. It could be 1, 5, 10, 30, 60 seconds between gossips. Remote peer will understand how to get gossip given this time interval. 
### from UI layer
* gossip the new msgDagRoot when a message that has sent to friend immdeiately
* signal current gossip frequency. 
* gossip user behavior: such as gossip "node select A<->B chat", "node type in A<->B chat" with a `new random mutable salt`, in this salt, remote peers can get new information instantly. This can potentially increase user behavior. 
### from chain layer - put gossip according to freqence
* `demand` some immutable data item
* according the default frequency `publish` msgDAGroot. This provide 3 signals: `50% connected`/`connected`, last msg time, last seen time. 
