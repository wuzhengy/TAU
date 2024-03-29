# Internet Protocol 2 - IP2 

Version:	0.1 - on going draft

Last-Modified:	Feb 1st, 2022, TAU Cambridge Ltd. 

## Overview
Internet Protocol 2(IP2) aims to provide the best effort universal p2p connectivity. 

First of all, IP2 uses self-generated 256-bits "public key" as immutable address or ID, while classical Internet Protocol(IP1) appoints hierarchically and often dynamically IP addresses. IP1 has caused "uncertain reachability" in the type of address and connection, especially when the same device is moving among different networks or locates in unknown firewall rules; therefore additional proxy or name server infrastructure burden is needed for each application. 

On top of IP2, traditional TCP or UDP type of services could be rebuilt without worrying about static/dynamic, v4/v6, local/public, wifi/cellular types of IP1 addresses and their arbitrary address translation and filtering restrictions. This potentially reduces cost of operating IOT devices, as well.  

The technology stack includes "distributed vectors" for seeking, sending and capturing data, ED25519 assymetric encryption for premission-less and colision-free unique addressing, and pattern randomized transmission on UDP. TAU Cambridge provides an opensource C++ reference implementation libIP2 on github(...). Although IP2 is built on top of UDP, it is still a transmission level protocol extending the ability of address space, so that in the IP2 packets, there are not timestamp, sequence numbers, authentication token or signatures as in libtorrent type of data structure. 

* Please note the terminology used in this document to avoid confusion. A "public key" is self generated to be used as address for nodes. A "node" is a client/server with an "public key" listening on a UDP socket. Due to self-addressing, multiple nodes can share one public key. This will make classical connection based communication such as TCP hard to use the sequencial method, since one public key can have many devices and hard to make end to end connection. The levenstein distance array will come into the picture for public key to public key communication. This makes communciction potentially more powerful by including the broadcast function with just one public key. libTAU provides a sample of connection based communication over IP2. 

IP2 is designed for "public key direct to public key" overlay communication. Thanks to the innovation of XOR distance to form up the prefix prefix swarm network, the nodes are incentivated to share "public key to IP address" naming and relaying for restricted nodes, disregard of application types. A "distance metric" is used to compare two addresses for "closeness". The metric is XOR and the result is interpreted as an unsigned integer. distance(A,B) = |A xor B| Smaller values are closer. Nodes must maintain a routing vector containing the contact information for a number of other nodes. As opposed to the traditional Kademlia[1] "The routing table becomes more detailed as IDs get closer to the node's own ID", IP2 prefer a big single layer routing vector, in order to saving traversal data consumption. IP2 API gives more effort in saving data transmission, due to most of the Internet devices are personal devices on batteries and meterred network. 

When a node wants to find another node for sending data, it uses the distance metric to compare the public key of the receiver with the records of the nodes in its own routing table. It then contacts the nodes it knows about with public key closest to the target node and asks them for the contact information of nodes. If a contacted node knows about the target public key, the contact information is returned with the response. Otherwise, the contacted node must respond with the contact information of the nodes in its routing vector that are closest to the public key. The original node iteratively queries nodes that are closer to the target until it cannot find any closer nodes. 

After the traverse is exhausted, the client either already contacted the target or rely on nodes in prefix swarm to push the payload to target, which mare most likely behind NAT. The sender node includes an value known as the "prefix Swarm node" The receiver will use these nodes as relay to communicate with sender efficiently. Each node will maintain a prefix swarm cache for receivers.

When a node, B, want to receive communication, it will traverse the "closest" nodes to register itself to those nodes' routing vector. Collectively, we call these nodes prefix swarm for B. Later on, when prefix swarm received information targetting to the node B, it will push the data to B when B is behind NAT or restricted by firewall rules. The prefix swarm is a novol idea from TAU Cambridge on top of classical Kademlia protocol. The swarm is also used to prevent "suffocating attack" to a public key. When some hacker generates many "closest" public key and uses them to surround a node B to block communication, B can change strategy to put "time sensitive relay nodes" into prefix swarm for sender A to setup up alternative relay.

Since each node has key pairs, all the UDP payload between nodes are encrypted with sender private key to make only target node can read it. Internet routers can not tell the pattern of an encrypted payload, so that it can not differenciate from video and voice services. The cost of block IP2 packets will be turning down entire UDP protocol. 

IP2 nodes uses app-input, self-learning and pre-coded IP/Port addresses for bootstrap of routing vector. We view this as risk, due to any contact or leakage of node to public IP2 nodes will fill up routing vector very quickly. Each node will learn IP2 bootstrap nodes during the life, the longer a nodes online, the more bootstrap knowledge it has as accumulation of such memory. 

### The best effort data transmission
IP2 uses two types of tramsission to achieve "the best effort" due to dynamic nature of all nodes. 
* Non-cached, the relay does not store data. This is for sequence un-senstive data transmission such as in blockchain. Non-cached transmission is implemented using API: relay
* Cached, the relay nodes will store data from sender for 24 hours. The receiver can "get" the stored data. This is ideal for peer to peer messaging, where chat messages are critical for delivery.  Cached transmission is implemented using API: put/get

#### transmission traversal control: invoke_window and invoke_limit
In order to achieve best effort universal connectivity, IP2 needs to find the target node or its prefix swarm from the internet, before data could be deliverred. Essentially in IP2, data are sent from sender to receiver swarm nodes to enhance the success rate. 
The searching complexity is O(nodes number of Internet), which is too big. Thanks to Kadmelia DHT, it reduces the searching complexity to O(logN). We use the same traverse strategy. To help traverse running efficiently. App developer can set some control parameters, depending on communication status such as initiation, normal, best effort, fast, slow or doze. 
* invoke_window, is the range in the searching candidates, m_results, indexed by distance for invoking selection. m_results is the temporary list for traveral with sorted distance to the target. Bigger window will prevent "local optimization" problem. 
* invoke_limit is the total invoke number that one full traverse will perform. This controls the depth of the searching.  

A sequence of a traversal process could look like this: 
```
* copy invoke_window number of nodes from routing vector to m_result temporary list
* randomly select alpha number nodes from invoke_window  of m_result, invoke these request and waiting for short timeout
* refresh m_result, and invoke more request in the invoke_window until invoke limit or invoke_window depletes.  
```

## Three distributed vectors to handle best effort connectivity
The implementation of IP2 global address seeking needs a few vectors. 
* Routing table - storing good relay node, which is traditional Kadmalia DHT to find prefix neighbour of a public key. Alive vector - prefix swarm.  
* Advisory vector - storing receiver sended abitratry relay nodes for fast and randomized response path. It prevents "suffocating" on a public key; vs traversed routing table relay nodes for receiver. 
* Income list - storing temporary non-relay nodes which are under firewall inbound control for "hole punching"
 
### Routing Vector and its replacement buckets

Every node maintains a routing vector of known good nodes. The nodes in the routing vector are used as starting points for queries in the DHT. Nodes from the routing vector are returned in response to queries from other nodes. IP2 uses vector here than Kadmelia buckets table to save traveral data consumption. We trade storage for less traffic. 

Not all nodes that we learn about are equal. Some are "good" and some are not. Many nodes using the DHT are able to send queries and receive responses, but are not able to respond to queries from other nodes. It is important that each node's routing table must contain only known good nodes. A good node is a node has responded to one of our queries within the last 15 minutes. A node is also good if it has ever responded to one of our queries and has sent us a query within the last 15 minutes. After 15 minutes of inactivity, a node becomes questionable. Nodes become bad when they fail to respond to multiple queries in a row. Nodes that we know are good are given priority over nodes with unknown status. When multiple IP addresses mapping to the same public key, the newer IP address the higher priority they are. If any nodes in the bucket are known to have become bad, then one is replaced by the new node. If there are any questionable nodes in the bucket have not been seen in the last 15 minutes, the least recently seen node is pinged. If the pinged node responds then the next least recently seen questionable node is pinged until one fails to respond or all of the nodes in the bucket are known to be good. If a node in the bucket fails to respond to a ping, it is suggested to try once more before discarding the node and replacing it with a new good node. In this way, the table fills with stable long running nodes.

Routing vector should maintain a "last changed" property to indicate how "fresh" the contents are. When a node is pinged and it responds, or a node is added, or a node in a bucket is replaced, the last changed property should be updated. Vector that have not been changed in 15 minutes should be "refreshed." This is done by picking a random ID in the range of the vector and performing a find_node search on it. Nodes that are able to receive queries from other nodes usually do not need to refresh buckets often. Nodes that are not able to receive queries from other nodes usually will need to refresh all buckets periodically to ensure there are good nodes in their table when the DHT is needed.

Upon inserting the first node into its routing vector and when starting up thereafter, the node should attempt to find the closest nodes in the DHT to itself. It does this by issuing find_node messages to closer and closer nodes until it cannot find any closer. The routing vector should be saved between invocations of the client software.

### Capture swarm advisory vector
Along with routing vector for storing good direct contact information for nodes, we define a capture swarm vector to record the alternative relay nodes for receiver nodes. The relay nodes will go extra "mile" to push the information to receiver, especially when receivers are behind restictions; while Kadmelia DHT does not "push".

IP2 receiver node relys on "capture swarm" to increase the probability of receiving inbound data. Even for a node conciously enabled with public static address, there is still no garantee to receive data due to unknown regional firewall filtering. Each node A will use a distance strategy to select 8 internet nodes, B, to register in B's capture swarm, R. 

Five of them maybe based on closest XOR distance. Three of them are randomly selected from routing vector. This is to prevent hacker to generate set of close nodes to suffocating A's inbound data. After A registering its existence to R, R will add A into local capture swarm vector. 

The B will do two things for its capture swarm friends, since capture swarm meant to be mutual benefitical: 
* providing data relay, mostly when node A is behind public accessibility such as behind NAT or filter restrictions. R will have the ability to send data to A, because, A has openned up an inbound connection for R on the routing. Internet devices have to open outbound route for A to R, so reverse traffic is open for average 15 minutes according to convention port openning rules. Every 15 minutes, A should register to its capture swarm for inbound relay. 
* temporary data storage when target nodes are off line. This is a last resort for helping nodes to transmit critical data. This is for some application such as instant messaging to assure important data, link, hash or texts are deliverred at the best effort. Each node will maintain such storage for 1000 unites of 1 kb data item. 

Quality accredit of swarm member
* When receiver response data to sender, in the packet structure, it will include optional relay suggestion. When the suggestion confirms member of local swarm member, it means the member is a high quality member, since it is a closed loop.

### Push vector
For nodes under firewall control, they need to register themself to public freedom nodes to receive external inbound data. Overall, there is no static rule to decide whether a node is under control or not. We do best effort to make decision on each nodes. The registration information will be stored in push vector for future use when the relay need to push data to nodes under inbound control, as known as "hole punching".

Under control - Non Relay: added to capture swarm vector for linking relays, not routing vector. NR nodes are restricted so that can not become relay or direct into routing vector. It will need capture swarm to receive data. 
- Meterred nodes
- Devices running on battery
- IPv4 without UPNP port openning

Relay: 
- IPV4 UPNP and DHT response same external IP as UPNP response.
- coded DHT bootstrap public IP addresses is Relay, provided by app developers like TAU.
- IPv6 PCP inbound traffic openning accepted. However, tt is tricky to decide IPv6 relay nodes. Most of IPv6 does not go through NAT, and firewall filtering strategy is unknown. 

When relay nodes receive members of push vector, which are out-bound initiated temporary connection, it can not be put into routing vector or capture swarm, which are both defined for good public accessible nodes. Push vector is setup for these potentially restricted nodes. However, if one nodes claim to be relay, we can put those into routing table replacement bucket for testing. 
#### adding resolving NAT short memory of port mapping issue
While IPv4 and IPv6 coexisting in mobile network, the carrier NAT tends to allocate shorter memory time of incoming port mapping due to significant requests from internal network. The capture swarm nodes fit nicely to help internal nodes to establish constant port mapping relationship. We are now using 15 seconds as interval to register internal nodes to capture swarm, so that when new messages arrives, the pushing nodes can send information to internal nodes. 
While we do the each 15 seconds resigerting, the relay nodes will send cached messages within 5 minutes history to internal nodes. 
By these two means, the peer to peer communication will be rather robust. 

## Payload structure
IP2 uses UDP as substrate, the UPD payload is composed of : 
* The first 32 bytes are sender's public key for receiver to decrypt payload.
* The remaining payload is the encrypted content with sender private key and receiver public key.
The total payload need to be less than 1000 bytes to fit into MTU of 1500 bytes. 

The content structure is a mechanism consisting of bencoded dictionaries. Bencode is a flexible and space-saving binary encoding scheme comparing to json or xml. A single query packet is sent out and a single packet is sent in response. There is no retry. 

There are 3 message types: query, response, and error. A message is a single dictionary with three keys common to every message and additional keys depending on the type of message. 
* Every message has a key "t" with a string value representing a **transaction** ID. This transaction ID is generated by the querying node and is echoed in the response, so responses may be correlated with multiple queries to the same node. The transaction ID should be encoded as a short string of binary numbers, typically 2 characters are enough as they cover 2^16 outstanding queries. 
* Every message also has a key "y" with a single character value describing the **type** of message. The value of the "y" key is one of "q" for query, "r" for response, or "e" for error. 

* A key "v" should be included in every message with a client **version** string. 

#### Contact Encoding

Contact information for nodes is encoded as a 38-byte string. Also known as "Compact node info" the 32-byte public key with the compact IP-address/port info concatenated to the end.

#### Queries

* Queries, or message dictionaries with a "y" value of "q", contain two additional keys; "q" and "a". Key "q" has a string value containing the method name of the query. There are three queries: ping, find_node, relay. 
* Key "a" has a dictionary value containing named arguments to the query.

#### Responses

Responses, or message dictionaries with a "y" value of "r", contain one additional key "r". The value of "r" is a dictionary containing named return values. Response messages are sent upon successful completion of a query.

#### Errors

Errors, or message dictionaries with a "y" value of "e", contain one additional key "e". The value of "e" is a list. The first element is an integer representing the error code. The second element is a string containing the error message. Errors are sent when a query cannot be fulfilled. The following table describes the possible error codes:

#### Code	Description
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
```
#### DHT Queries

All queries have an "p" key and value containing the public key of the querying node. All responses have an "p" key and value containing the public key of the responding node.

**ping**

The most basic query is a ping. "q" = "p" A ping query has a single argument, "p" the value is the senders public key. The appropriate response to a ping has a single key "p" containing the public key of the responding node.

Example Packets
```
ping Query = {"t":"aa", "y":"q", "q":"p", "a":{"p":"..."}}
bencoded = d1:ad1:p3:...e1:q1:p1:t2:aa1:y1:qe
Response = {"t":"aa", "y":"r", "r": {"p":"..."}}
bencoded = d1:rd1:p3:...e1:t2:aa1:y1:re
```
**find_node**

Find node is used to find the contact information for a node given its ID. "q" == "f" A find_node query has two arguments, "p" containing the node public key of the querying node, and "target" containing the public key of the node sought by the queryer. When a node receives a find_node query, it should respond with a key "nodes" and value of a string containing the compact node info for the target node or the K (8) closest good nodes in its own routing table.

Example Packets

```
find_node Query = {"t":"aa", "y":"q", "q":"r", "a": {"p":"...", "t":"..."}}  // todo adding payload.
bencoded = d1:ad1:p3:...1:t3:...e1:q1:f1:t2:aa1:y1:qe
Response = {"t":"aa", "y":"r", "r": {"p":"...", "n": "..."}}
bencoded = d1:rd1:n3:...1:p3:...e1:t2:aa1:y1:re
```
**relay**

Relay is used to traverse and send data. "q" == "r" A r query has two arguments, "p" containing the node public key of the sender node, and "t" containing the public key of the receiver node. When a node receives a relay query, it should respond with a key "nodes" and value of a string containing the compact node info for the target node or the K (8) closest good nodes in its own routing table. The node will be the receiver, if not, the node will check local relay vector for the receiver node, if existing, the node will send the payload to receiver to satisfy the relay function. 

Example Packets
```
relay Query = {"t":"aa", "y":"q", "q":"f", "a": {"p":"...", "t":"..."}} 
bencoded = d1:ad1:p3:...1:t3:...e1:q1:f1:t2:aa1:y1:qe
Response = {"t":"aa", "y":"r", "r": {"p":"...", "n": "..."}}
bencoded = d1:rd1:n3:...1:p3:...e1:t2:aa1:y1:re
```

**put**

Put is used to traverse, send and store data. "q" == "p" A p query has two arguments, "p" containing the node public key of the sender node, and "t" containing the public key of the receiver node. When a node receives a put query, it should respond with a key "nodes" and value of a string containing the compact node info for the target node or the K (8) closest good nodes in its own routing table. The node will be the receiver, if not, the node will check local relay vector for the receiver node, if existing, the node will send the payload to receiver to satisfy the put function; at the same time, it will store the value for future get. 

Example Packets
```
relay Query = {"t":"aa", "y":"q", "q":"p", "a": {"p":"...", "t":"..."}} 
bencoded = d1:ad1:p3:...1:t3:...e1:q1:f1:t2:aa1:y1:qe
Response = {"t":"aa", "y":"r", "r": {"p":"...", "n": "..."}}
bencoded = d1:rd1:n3:...1:p3:...e1:t2:aa1:y1:re
```

**get**

Get is used to traverse and get data from relay nodes cache. "q" == "g".  A g query has two arguments, "p" containing the node public key of the sender node, and "t" containing the public key of the receiver node. When a node receives a get query, it should respond with a key "nodes" and value of a string containing the compact node info for the target node or the K (8) closest good nodes in its own routing table. The node will check local storage, if existing, the node will send the payload to receiver to satisfy the get function. 

Example Packets
```
relay Query = {"t":"aa", "y":"q", "q":"g", "a": {"p":"...", "t":"..."}} 
bencoded = d1:ad1:p3:...1:t3:...e1:q1:f1:t2:aa1:y1:qe
Response = {"t":"aa", "y":"r", "r": {"p":"...", "n": "..."}}
bencoded = d1:rd1:n3:...1:p3:...e1:t2:aa1:y1:re
```

## PCP, UPnP, NATPMP, ipv4, ipv6, NAT64, NAT66, NAT44, DS light, XLAT464 and firewall filtering discussions

While the networks the evolving, address translation, filtering, bridging and space extension protocols end up coexisting with each other. We are facing a mixture today. It is hardly to see IPv4 going away, even IPv6 only networks start to appear. IP2 nodes should not bother with underneath IP protocol of v4 or v6. In IPv6 only cellular network, data transmission between IPv4 nodes to IPv6 nodes will require relay nodes supporting dual stack. The self-addressing scheme is a way of smooth up v4 and v6 network.  

The dream of P2P direct communication requires address independance and filtering resistant connectivity. Network protocols on the transport layer are not be able to solve these two requirements; because transporting is concerned of that how data flows between phyical end points, not logical sender and receiver. 

An overlay protocol such as IP2 is required to smooth up these building block edges. In the core of IP2, it is the capture swarm nodes collectively serving as relay. Even when a node has publicly accessible IP address, the node is still subject to firewall filtering from regional operators. The node can not solve this connectivity problem by own power, it has to rely on a nodes community randomly spread on global internet. The choices of such community and making such community available for global access is not a straight forward task. If too close to node public key, it will enable hacker's suffocating attack; if too far, it will bring sender searching difficulty. It might have to engage some time-sensitive address transform algorithm to make filter difficult to track the changing capture swarm.

## Future security consideration
One key attack to self-addressing is to use brutal force to generate public key that is very close to a target public key. These keys can join the network to polute the searching. Some scheme of using random relay nodes based on time to join the capture swarm to maintain most basic signal is critical. 


Main References
  
[1]	Peter Maymounkov, David Mazieres, "Kademlia: A Peer-to-peer Information System Based on the XOR Metric", IPTPS 2002.
  
[2]	Andrew Loewenstern, Arvid Norberg, "DHT Protocol", Bittorent.org, 31-Jan-2008.
  
[3] Information Sciences Institute University of Southern California, "INTERNET PROTOCOL", IETF RFC:791, September 1981

Copyright
  
This document has been placed in the public domain.
  
