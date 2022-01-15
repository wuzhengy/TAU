## Title:	Internet Protocol 2 - IP2 

Version:	0.1 - on going draft

Last-Modified:	Feb 1st, 2022

Author:	TAU Cambridge Ltd. 


Internet Protocol uses management assigned addresses ether IPv4 or IPv6 to communicate. IP2 uses self-generated 256 bits public key as address for universal transmission. IP2 uses a "distributed sloppy hash table" (DHT) for traversing, sending and capturing data, ED25519 key pairs for addressing and content randomized transmission, and UDP for Internet communication.

Please note the terminology used in this document to avoid confusion. A "public key" is self generated to be used as address for nodes. A "node" is a client/server with an "public key" listening on a UDP socket implementing the distributed hash table protocol.  

### Overview

Each node has a globally unique self generated public key, which is generated from random seed from ED25519 encyption scheme. Multiple nodes can share the same public key. When several network devices share the same public key, the data flow control will becoming more complex and therefore powerful, which will employ Leveinstein distance array in the future connection based communication topic. 

A "distance metric" is used to compare two addresses for "closeness". The distance metric is XOR and the result is interpreted as an unsigned integer. distance(A,B) = |A xor B| Smaller values are closer. Nodes must maintain a routing table containing the contact information for a number of other nodes. As opposed to the traditional Kademlia[1] "The routing table becomes more detailed as IDs get closer to the node's own ID", IP2 prefer a big single layer routing table, in order to saving traversal data consumption. 

When a node wants to find another node for sending data, it uses the distance metric to compare the public key of the receiver with the records of the nodes in its own routing table. It then contacts the nodes it knows about with public key closest to the target node and asks them for the contact information of nodes. If a contacted node knows about the target public key, the contact information is returned with the response. Otherwise, the contacted node must respond with the contact information of the nodes in its routing table that are closest to the public key. The original node iteratively queries nodes that are closer to the target until it cannot find any closer nodes. 

After the traverse is exhausted, the client either already touch the target or rely on captureswarm to push the payload to target, which mare mostly behind NAT. 

The sender node for a query includes an value known as the "captureSwarm." The receiver will use the capture swarm to find sender efficiently if sender is resricted by firewall rules or NAT/proxy. 

When a node want to receive communication, it will traverse the "closest" nodes to register itself into those nodes routing table. Collectively, we call these nodes capture swarm. Later on, when capture swarm received information targetting to the node, it will push the data to the node when they are behind NAT or restricted by firewall rules. The capture swarm is novo idea from TAU Cambridge on top of classical Kademlia protocol. The swarm is also used to prevent "suffocating attack" to a public key. 

Since each node has key pairs, all the UDP payload between nodes are encrypted with sender private key and only target node can read it. Internet routers can not tell the pattern of an encrypted payload, so that it is not differciable from other video and voices services. The cost of block IP2 packets is to turn down entire UDP protocol. 

IP2 nodes uses user input, self-learning and pre-coded nodes IP addresses for bootstrap of routing table. We view this risk is low due to any contact or leakage of node to public nodes will fill up routing table very quickly. 

### Routing Table

Every node maintains a routing table of known good nodes. The nodes in the routing table are used as starting points for queries in the DHT. Nodes from the routing table are returned in response to queries from other nodes.

Not all nodes that we learn about are equal. Some are "good" and some are not. Many nodes using the DHT are able to send queries and receive responses, but are not able to respond to queries from other nodes. It is important that each node's routing table must contain only known good nodes. A good node is a node has responded to one of our queries within the last 15 minutes. A node is also good if it has ever responded to one of our queries and has sent us a query within the last 15 minutes. After 15 minutes of inactivity, a node becomes questionable. Nodes become bad when they fail to respond to multiple queries in a row. Nodes that we know are good are given priority over nodes with unknown status.

The routing table covers the entire node space from 0 to 2160. The routing table is subdivided into "buckets" that each cover a portion of the space. An empty table has one bucket with an ID space range of min=0, max=2160. When a node with ID "N" is inserted into the table, it is placed within the bucket that has min &lt;= N &lt; max. An empty table has only one bucket so any node must fit within it. Each bucket can only hold K nodes, currently eight, before becoming "full." When a bucket is full of known good nodes, no more nodes may be added unless our own node ID falls within the range of the bucket. In that case, the bucket is replaced by two new buckets each with half the range of the old bucket and the nodes from the old bucket are distributed among the two new ones. For a new table with only one bucket, the full bucket is always split into two new buckets covering the ranges 0..2159 and 2159..2160.

When the bucket is full of good nodes, the new node is simply discarded. If any nodes in the bucket are known to have become bad, then one is replaced by the new node. If there are any questionable nodes in the bucket have not been seen in the last 15 minutes, the least recently seen node is pinged. If the pinged node responds then the next least recently seen questionable node is pinged until one fails to respond or all of the nodes in the bucket are known to be good. If a node in the bucket fails to respond to a ping, it is suggested to try once more before discarding the node and replacing it with a new good node. In this way, the table fills with stable long running nodes.

Each bucket should maintain a "last changed" property to indicate how "fresh" the contents are. When a node in a bucket is pinged and it responds, or a node is added to a bucket, or a node in a bucket is replaced with another node, the bucket's last changed property should be updated. Buckets that have not been changed in 15 minutes should be "refreshed." This is done by picking a random ID in the range of the bucket and performing a find_nodes search on it. Nodes that are able to receive queries from other nodes usually do not need to refresh buckets often. Nodes that are not able to receive queries from other nodes usually will need to refresh all buckets periodically to ensure there are good nodes in their table when the DHT is needed.

Upon inserting the first node into its routing table and when starting up thereafter, the node should attempt to find the closest nodes in the DHT to itself. It does this by issuing find_node messages to closer and closer nodes until it cannot find any closer. The routing table should be saved between invocations of the client software.



### Packet structure

The structure is a mechanism consisting of bencoded dictionaries sent over UDP. A single query packet is sent out and a single packet is sent in response. There is no retry. There are three message types: query, response, and error. For the DHT protocol, there are three queries: ping, find_node, relay.

A message is a single dictionary with three keys common to every message and additional keys depending on the type of message. Every message has a key "t" with a string value representing a transaction ID. This transaction ID is generated by the querying node and is echoed in the response, so responses may be correlated with multiple queries to the same node. The transaction ID should be encoded as a short string of binary numbers, typically 2 characters are enough as they cover 2^16 outstanding queries. Every message also has a key "y" with a single character value describing the type of message. The value of the "y" key is one of "q" for query, "r" for response, or "e" for error. A key "v" should be included in every message with a client version string. 

Contact Encoding

Contact information for nodes is encoded as a 38-byte string. Also known as "Compact node info" the 32-byte Node public key in network byte order has the compact IP-address/port info concatenated to the end.

Queries

Queries, or message dictionaries with a "y" value of "q", contain two additional keys; "q" and "a". Key "q" has a string value containing the method name of the query. Key "a" has a dictionary value containing named arguments to the query.

Responses

Responses, or  message dictionaries with a "y" value of "r", contain one additional key "r". The value of "r" is a dictionary containing named return values. Response messages are sent upon successful completion of a query.

Errors

Errors, or  message dictionaries with a "y" value of "e", contain one additional key "e". The value of "e" is a list. The first element is an integer representing the error code. The second element is a string containing the error message. Errors are sent when a query cannot be fulfilled. The following table describes the possible error codes:

Code	Description
```
201	Generic Error
202	Server Error
203	Protocol Error, such as a malformed packet, invalid arguments, or bad token
204	Method Unknown
```
Example Error Packets:
```
generic error = {"t":"aa", "y":"e", "e":[201, "A Generic Error Ocurred"]}
bencoded = d1:eli201e23:A Generic Error Ocurrede1:t2:aa1:y1:ee
DHT Queries
All queries have an "id" key and value containing the node ID of the querying node. All responses have an "id" key and value containing the node ID of the responding node.
```
ping

The most basic query is a ping. "q" = "ping" A ping query has a single argument, "id" the value is a 20-byte string containing the senders node ID in network byte order. The appropriate response to a ping has a single key "id" containing the node ID of the responding node.
```
arguments:  {"id" : "<querying nodes id>"}

response: {"id" : "<queried nodes id>"}
```
Example Packets
```
ping Query = {"t":"aa", "y":"q", "q":"ping", "a":{"id":"abcdefghij0123456789"}}
bencoded = d1:ad2:id20:abcdefghij0123456789e1:q4:ping1:t2:aa1:y1:qe
Response = {"t":"aa", "y":"r", "r": {"id":"mnopqrstuvwxyz123456"}}
bencoded = d1:rd2:id20:mnopqrstuvwxyz123456e1:t2:aa1:y1:re
```
find_node

Find node is used to find the contact information for a node given its ID. "q" == "find_node" A find_node query has two arguments, "id" containing the node ID of the querying node, and "target" containing the ID of the node sought by the queryer. When a node receives a find_node query, it should respond with a key "nodes" and value of a string containing the compact node info for the target node or the K (8) closest good nodes in its own routing table.
```
arguments:  {"id" : "<querying nodes id>", "target" : "<id of target node>"}

response: {"id" : "<queried nodes id>", "nodes" : "<compact node info>"}
```
Example Packets
```
find_node Query = {"t":"aa", "y":"q", "q":"find_node", "a": {"id":"abcdefghij0123456789", "target":"mnopqrstuvwxyz123456"}}
bencoded = d1:ad2:id20:abcdefghij01234567896:target20:mnopqrstuvwxyz123456e1:q9:find_node1:t2:aa1:y1:qe
Response = {"t":"aa", "y":"r", "r": {"id":"0123456789abcdefghij", "nodes": "def456..."}}
bencoded = d1:rd2:id20:0123456789abcdefghij5:nodes9:def456...e1:t2:aa1:y1:re
```

relay: ???

Relay is used to travrse and send data. "q" == "r" A r query has two arguments, "id" containing the node ID of the querying node, and "target" containing the ID of the node sought by the queryer. When a node receives a find_node query, it should respond with a key "nodes" and value of a string containing the compact node info for the target node or the K (8) closest good nodes in its own routing table.
```
arguments:  {"id" : "<querying nodes id>", "target" : "<id of target node>"}

response: {"id" : "<queried nodes id>", "nodes" : "<compact node info>"}
```
Example Packets
```
find_node Query = {"t":"aa", "y":"q", "q":"find_node", "a": {"id":"abcdefghij0123456789", "target":"mnopqrstuvwxyz123456"}}
bencoded = d1:ad2:id20:abcdefghij01234567896:target20:mnopqrstuvwxyz123456e1:q9:find_node1:t2:aa1:y1:qe
Response = {"t":"aa", "y":"r", "r": {"id":"0123456789abcdefghij", "nodes": "def456..."}}
bencoded = d1:rd2:id20:0123456789abcdefghij5:nodes9:def456...e1:t2:aa1:y1:re
```
References
  
[1]	Peter Maymounkov, David Mazieres, "Kademlia: A Peer-to-peer Information System Based on the XOR Metric", IPTPS 2002.
  
[2]	Andrew Loewenstern <drue@bittorrent.com>, Arvid Norberg <arvid@bittorrent.com> "DHT Protocol", Bittorent.org, 31-Jan-2008.
  
[3] Information Sciences Institute University of Southern California, "INTERNET PROTOCOL", IETF RFC:791, September 1981

Copyright
  
This document has been placed in the public domain.
  
