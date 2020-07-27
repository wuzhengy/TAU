Mutable item republish.
* According to block frequency, e.g 5 minutes, nodes republish data under a certain salt. 
  - The `content` of item is the key of an immutable item that is latest message learned by the node in past block time. 
    * In `blk` channel, it is a block hash pointing to the tip of the blockchain. 
    * In `msg` channel, it is the item key of a message. 
    * In `tx` pool channel, it the highest tx fee transaction. 
  - The `hash link` of the data is the public key of another peer with potential new messages. 
* Mutable item does not republish nodes own information. Nodes own information falls into instant publish schedule. 

Immutable item republish. 
* According to block frequency, nodes republish one random immutable item in the channel. 
  - In `blk` channel, the item is block in consensus
  - In `msg` channel, the item is own message
  - In `tx` channel, the item is own transaction
* Immutable item does not republish other nodes message or tx. 
