* One year stateful
TAU blockchain will only keep a chain state for 1000 days, any state beyond will be forgotten forever. This will make the blockchain in flat size, good for phones. You can view this as **epoch stateful mining** and **stateless verification** combined. 
- mining need be stateful with 1000 x 288 blocks information. 
- verification could be purely stateless. 


* Simple opcodes
TAU aims to enable basic devices to have determined functions in weak networking regions, so we do not add automaton language into transactions and do not engage variable gas concept.
  * type 1:  1 to n wiring, each transaction will allow to send up to 24 addresses. 
  * type 2:  plain text

* Some ranges: 
mutable range: one day, 5 x 12 x 24 = 288 blocks.
new peer range: from current to last new address added block. 
stateful range: 1 year, 288 x 1000= 288K blocks

* Block and Transaction structure
 * add sender and miner IP addresses
 * add new peer pointer

