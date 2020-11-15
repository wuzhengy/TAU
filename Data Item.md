* Image
  * Image will be retrofit mobile phone display, the default is to fitting 1/3 of phone screen. We use progressive image CDT to present pictures in 5k data, which is 5 immutable data item linked list. If user want more clarity of the image, they can gossip more data from community. 
  
<br><br>

Mutable Item has the immutable key of the target data, which could be either a demanding or a publishing of block, transaction or message
Immutable Data type: 0 - block, 1 - transaction, 2 - text, 3 - image
vertical links; horizontal links; pubkey; extention link; 


Immutable Item
The content addressable data scheme such as IPLD, the immutable item space is a DAG. Mutable item is the entry to the DAG. 
* Block: basically the block header and hash link to transactions. 
* Transaction
* Message: own content, link to previous own message, link to another referral message. 

Immutable item will not point to mutable item.
Mutable item only points to immutable item. 
