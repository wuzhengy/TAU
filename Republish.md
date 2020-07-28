Mutable item republish.
* According to block frequency, e.g 5 minutes, nodes republish data under a certain salt. 
  - The `content` of item is the key of an immutable item that is latest knowledge learned in public domain by the node in past block time. 
    * In `blk` channel, it is a block at the tip of the believed blockchain. 
    * In `msg` channel, it is a latest message. 
    * In `tx` pool channel, it the highest tx fee transaction. 
  - The `hash link` of the data is the public key of another peer with potential new messages. 
* Mutable item does not prioritize on republishing own information. Nodes own information falls into controlled publish schedule. 

Immutable item republish. 
* According to block frequency, nodes republish one random immutable item in the channel. 
  - In `blk` channel, a item is in consensus
  - In `msg` channel, a item is within own message history
  - In `tx` channel, a item is with own transaction pool
* Immutable item does not republish other nodes message or tx. Republish in immutable schedule is about own history or concensus.
