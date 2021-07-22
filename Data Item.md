libTAU need to adopt a type of encoding fitting for fast UDP transmission, blockchain ledger and DHT RPC call on a unstable global network. It means it has to be small, that is binary, with a internal map structure for kv storage. 
Bencode, used by torrent world, is a partial human readable binary encoding design. It has human readable interface, so an editor is possible for human to construct. Assume to build a on-chain web, bencode can convert into HTML for display. This would be a nice **serverless website**. <br>

libTAU has two type of data structure: mutable and immutable
* mutable data is used for each peer to signal others of its status include time, public key, IP address and immutable data endpoints. 
* immutable data is used for real content transmission include content and hash. 
<br><br>
libTAU `mutable data` time is only used for nodes live signal, which is vastly different from libtorrent for any data type transmission. libTAU data item idea can be viewed as an application of libtorrent arbitrary data item. Live signal is maintaining basic meta data communicaition between nodes without carrying real content as payload. The content data will be stored as immutable item and storage address is carried in the life signal. 

`immutable item`
The content addressable data scheme such as IPLD, the immutable item space is a DAG. Mutable item is the entry to the DAG. 
* Block: basically the block header and hash link to transactions. 
* Transaction
* Message: own content, link to previous own message, link to another referral message. 

UDP deep encryption to prevent router attack and spam. UDP payload in libTAU looks like this: （32 bytes public key) + (ECIES encrypted bencode entry). 
* UDP payload for immutable item:（public key) + (ECIES encrypted bencode entry(hash, content)). 
* UDP payload for mutable item:（public key) + (ECIES encrypted bencode entry(time, public key, device ID, immutable data hash, endpoints...)). 
