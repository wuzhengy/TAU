
Public Key
* 32 bytes ED25519 Pubkey Key generated from random seed

Node ID
* First 160 bits of the public key

Distrubted Routing Table
* Tuple: Node ID, IP, Port
* Meta data: last seen, last communicated, failure counter
* May consider multiple layers to make find_nodes better fit, 80 layers
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
  * This design is to create relay incentive for data cache provider. 
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
Bootstrap and time: nodes can get these information from both central or decentral source 
* from third party bootstrap and time server
* from community blockchain content
  * blockchain content is safer to validate true time and swarm, however it is slower than third party service. So we adopt a combined approach with blockchain as foundation. 
---
Encryption
* use receiver's public key, it is easy to encrypt all messages relaying to receiver in full UDP packet. 
* relaying nodes can sign the message use own private key, so that receiver knows who relays the messages, in which could be other mesage sender. 
