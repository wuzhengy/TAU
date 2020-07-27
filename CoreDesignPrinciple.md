* Every data item in TAU has both `content` and `hash link`. Any item is an entry into a DAG forest. 
  * Mutable item is a pointer to a DAG node. The item content is the key of an immutable item, and hash link is another public key. Another public key with salt can form a new pointer. Republish schedule follows block frequency. 
  * Immutable item is a DAG node. The item content is the data, and hash link is pointing to node, another key of immutable item. Republish is required to keep data life according to block fequency. <br><br>
* Every mutable item's content is the root of a `data schema` or the first immutable item starting the schema.<br><br>
* Every communication is under scope of a chain, which is a type of consensus. This is called `peer to consensus`.
