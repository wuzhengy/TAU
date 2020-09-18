* DAG - directed acyclic graph: every data item in TAU has both `content` and `link`. TAU content network can be viewed as a DAG. Any data connects another data for representing blocks, messages or images. 
  * Mutable item `content` is a pointer to a DAG node,the key of an immutable item; and link is another public key. 
   - The public key with a salt can form a new pointer. The new pointer nature is much dependent on channels. 
     - For #blk channle, it is another miner. 
     - For #msg, it is a latest message sending address. This pointer is used to make searching more efficient by every peers contributing knowledge.
  * Immutable item is a DAG node. The item `content` is the part of data schema, and `link` is pointing to another immutable item. Each immutable item also include a skip list pointer such as in block structure to point into a history item for speed up searching. 
* Data Schema - every mutable item's content is the root of a `data schema` or the first immutable item starting the schema.
  - Schema is series of immutable item together to present a data structure. IPLD protocol has built example of data schema. 
<br><br>
* P2C - Peer to Consensus: every communication is under scope of a chain, which is a type of consensus. This is called `peer to consensus`.The consensus will regulate spam and make content searching efficient within limited nodes. 
* Avoiding constants - we want to use as little as constants as possible to let system to self adjust to the environment such as consumption of memory and bandwidth with performance results. For example, how many libtorrent nodes to initiate. 

## Knowledge building blocks
Along the way of developing TAU, we have adopted many key ideas from many open-source community projects. Following are the key components we are adopting. 
#### Bitcoin
Hash linked chain, central-less and permission-less concensus
#### Ethereum
Account state transition of balance and nounce
#### NXT
Generation signature for mining, proof of stake, chain accumulative difficulty
#### IPLD/IPFS/LibP2P
Data Schema on blocks, content addressing
#### Libtorrent
Distributed cache table for data item, immutable data and mutable data, salt, ed25519 encryption
