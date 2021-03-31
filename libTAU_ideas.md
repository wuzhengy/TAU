
Public Key
* 32 bytes ED25519 Pubkey Key generated from random seed

Node ID
* First 160 bits of the public key

Distrubted Routing Table
* Tuple: Node ID, IP, Port
* Meta data: last seen, last communicated, failure counter
------

Mutable Data Cache Item
* Target: 80 bytes
* Value: 1000 bytes
  * sender X need to sign this value
  * first half of the sender Node ID has match 2nd half of the target.
* Appendix key-value (optional)
  * any random sender can update this field without signature
  * sender public key - 32 bytes, value - 32 bytes. 

Target of Mutable Data: libTAU mutable data aims to exchange data than storage, expecting lots of records overlaping like in the routing table
* 160 bits long
* First 80 bits: First half of the receiver Node ID
  * This design is to create relay incentive for data cache provider. 
  * The more data provided, the provider's Node ID has more places in other peers routing table
* Second 80 bits: First half of the sender Node ID
  * when first half equal to second half, this is a self data channel and possible to receive any sender's ping message and update the appendix value. This could be used to add anonymous friends. 

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
