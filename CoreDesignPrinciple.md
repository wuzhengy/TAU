* Every data item in TAU has both `content` and `hash link`. Any item can be viewed as entry into a linked list. 
  * For mutable item, content is the key of an immutable item, and hash link is another public key
  * For immutable item, content is the data, and hash link is another key of immutable item <br><br>
* Every mutable item's content is the root of a `data schema` or the first immutable item starting the schema.<br><br>
* Every communication is under scope of a chain, which is a type of consensus. This is called `peer to consensus`.
