libTAU - serverless communication, an open source Java library for unblockable p2p(pubkey to pubkey) and blockchain messaging.

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
