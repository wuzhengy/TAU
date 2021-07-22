**libTAU** adopts a type of serialization fitting for minimum packet UDP transmission, blockchain ledger and DHT RPC call on an unstable global public network. It means it has to be small, that is binary, with a internal map structure for kv storage of blockchain needs. 
`Bencode`, used by torrent world, is a partial human readable binary encoding design. It has human readable interface, so an editor is possible for human to construct the content. Assuming to build a on-blockchain web, bencoded content can be converted into HTML for display. This would be a **serverless website**. <br>

### libTAU has two type of data structure: mutable and immutable
* mutable data is used for each peer to signal others of its status include timestamp, public key, device ID and immutable data endpoints. It is used for nodes live signal, which is different from libtorrent for arbitrary data type transmission. Live signal is keeping basic beacon communicaition between nodes without carrying real content. The purpose is to populate routing table and sync-up meta-data. The content data will be stored as immutable item and storage address is carried in the life signal. Life signal need high frequency and smaller data packet size. 
* immutable data is used for real content transmission include content and hash. 
This is real content. libTAU starts with 1K bytes of content. Along the network getting accepted, this data size can be increased to increase the bandwidth.

### UDP deep encryption is used to prevent router attack and spam. 
UDP payload in libTAU looks like this: （32 bytes public key) + (ECIES encrypted bencode entry). 
* UDP payload for immutable item:（public key) + (ECIES encrypted bencode entry(hash, content)). 
* UDP payload for mutable item:（public key) + (ECIES encrypted bencode entry(time, public key, device ID, immutable data hash, endpoints...)). 
<br><br>
The router blocking certain traffice is the same nature as someone try to spam a certain services. With public key, binaries, as inistial prefix of an encrypted payload. It will be possible for receiver to identify whether it is a friendly traffic or malicous. It will also be impossible for routers to differenciate libTAU binaries from others, since we do not have recoginzation pattern in udp packets. 
