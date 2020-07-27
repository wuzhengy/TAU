* Every data item in TAU has both `content` and `hash link`. TAU content network can be viewed as a DAG(directed acyclic graph). Any data connects another data for blocks, messages or images. 
  * Mutable item is a pointer to a DAG node. The item content is the key of an immutable item, and hash link is another public key. 
   - The public key with a salt can form a new pointer. The new pointer nature is much dependent on channels. For #blk channle, it is another miner. For #msg, it is a latest message sending address. This pointer is used to make searching more efficient by every peers contributing knowledge.
   - Republish schedule follows block frequency. The republish strategy is specified under "republish doc". 
  * Immutable item is a DAG node. The item content is the part of data schema, and hash link is pointing to node, another key of immutable item. Republish is required to keep data life according to block fequency. 
* Every mutable item's content is the root of a `data schema` or the first immutable item starting the schema.
  - Schema is series of immutable item together to present a data structure. 
<br><br>
* Every communication is under scope of a chain, which is a type of consensus. This is called `peer to consensus`.
