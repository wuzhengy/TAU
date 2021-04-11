* One year stateful
TAU blockchain will only keep a chain state for one year, any state beyond will be forgotten forever. This will make the blockchain in flat size, good for phones. You can view this as **epoch stateful mining** and **stateless verification** combined. 
- mining need be stateful for the recent 1 year
- verification could be purely stateless. 


* Simple opcodes
TAU aims to enable basic devices to have determined functions in weak networking regions, so we do not add automaton language into transactions and do not engage variable gas concept.
  * type 1:  1 to n wiring, each transaction will allow to send up to 24 addresses. 
  * type 2:  plain text

* Some ranges: 
mutable range: one day, 288 blocks.
new peer range: from current to last new address added block. 
stateful range: 1 year, 288 x 365= 105,120 blocks

