* Every data item in TAU has both `content` and `hash link`. Any item is an entry into a linked DAG forest. 
  * For mutable item, content is the key of an immutable item, and hash link is another public key. Republish tip follows block frequency. 
  * For immutable item, content is the data, and hash link is another key of immutable item. Republish another immutable item is required to keep data life. <br><br>
* Every mutable item's content is the root of a `data schema` or the first immutable item starting the schema.<br><br>
* Every communication is under scope of a chain, which is a type of consensus. This is called `peer to consensus`.
