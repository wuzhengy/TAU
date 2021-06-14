The current libTAU aims to support text communication in the data payload. At current latency of global internet and considering Africa region especially, we believe video and pictures are better serviced centrally, while text such as block, messages and transactions are well fit in the ability of global internet to achieve communication decentrally and permissionlessly. 
  
<br><br>
libTAU `mutable data` time is only used for nodes live signal, which is vastly different from libtorrent for any data type transmission. libTAU data item idea can be viewed as an application of libtorrent arbitrary data item. Live signal is maintaining basic meta data communicaition between nodes without carrying real content as payload. The content data will be stored as immutable item and storage address is carried in the life signal. 

`immutable item`
The content addressable data scheme such as IPLD, the immutable item space is a DAG. Mutable item is the entry to the DAG. 
* Block: basically the block header and hash link to transactions. 
* Transaction
* Message: own content, link to previous own message, link to another referral message. 
