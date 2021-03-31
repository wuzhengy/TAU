
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
* Appendix key-value (optional)
  * any random sender can update this field without signature
  * sender public key -32 bytes, value - 4 bytes. 

Target of Mutable Data: libTAU mutable data aims to exchange data than storage, expecting lots of records overlaping like in the routing table
* 160 bits long
* First 80 bits: First half of the receiver Node ID
  * This design is to create relay incentive for data cache provider. 
  * The more data provided, the provider's Node ID has more places in other peers routing table
* Second 80 bits: First half of the sender Node ID
  * when first half equal to second half, this is a self data channel and possible to receive any sender's ping message and update the appendix value. 
