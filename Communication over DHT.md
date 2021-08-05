# TAU communication on DHT
TAU communicaiton protocol is modified from libtorrent Mainline DHT. Major changes has been introduced in incentive system, ID, routing, bootstrap,time service, data item structure and blockchain support. TAU uses public_key based communication, than IP based. Therefore, when a device does not have a real IP address, it still can communicate independantly without attached to a server. DHT peers cloud overall become relay services. TAU uses blockchain ledger for bootstraping network and community timestamp service. 

## Public Key format
* 32 bytes ED25519 key-pair generated from random seed
* Node ID is the public key for the incentive framework, nodes with closer prefix will have an incentive to help each other relaying the data. When a node refuses to relay data for other similar prefix nodes, it will found hard to get its needed data, since peers will not put them into routing table for referral. Nodes referral is a constant process in libTAU to locate other peers.  
* ECEIS is used for peer to peer content encryption. We use this to achieve deep level UDP payload encryption. 

## Routing table in DHT
* Tuple: Node_ID, IP, Port, referee list, rtt, timestamp, ping status
  * referee list is who has reported this nodeID and IP/Port combination. The more referees, the safer this node is reachable from public internet. 
* Meta data: last seen, last communicated, failure counter
* Single Layer routing table rather than multiple layers: node will communication with target ID, which is (receiver-sender). This will make routing vector filled with multiple clusters(prefix groups). 

## Main loop and depth traversal
* Kademlia DHT uses depth traversal to find nodes. 
* libTAU added a top layer mainloop to constantly locate friends and blockchain unchoked peers. This is the key to avoid dedicated server. 
  * the returned nodes will be added to replacement buckets for future main looping selection. 

## Communication token 
* Kademlia uses communication token in the get process to control spam
* libTAU removed mutable put function, in return, every mutable data is called exchange. We will rely node id checking to avoid spam. 


### Design live nodes, replacement buckets, m_list, alpha and beta
* Live nodes: in libtorrent, there are initial 8 nodes are picked into m_list for traversal model to start searching. but initial searching are start from alpha, which is set as 5. 
* replacement buckets: in libtorrent, there is where incoming node id, referred nodes or some failed live nodes are stored to server as candidates for m_list
* m_list: in libtorrent, this is the temporary list for traveral with sorted distance to the target
* alpha is initial selected number of nodes from live vector, beta is the nodes from replacement. 
* in libTAU, we inherites this arrangement; however due to libTAU traversal is no long recursive but limited to fixed number of friends exchanging live signals. every time traveral model only pick up alpha + beta number for nodes from m_list to invoke query the finish. alpha, we will use alpha as 1, beta as 1, which basically give chances to both live and replacement buckets. i think this will avoid local optimization also increase the main loop frequence. 

#### Choking communication
In peer friends communication, peers life signal to other friends. In the same idea, blockchain ID is a special type of friend, one peer will send three signals into its blockchain mutable life signal, as well as payload end point: 
* immutable point, diffitulty and blocknumber of `current tip` and blocks demand, 
* transaction pool levenstein distance array 
* messages history levenstein distance array
* payload end points

The receiving peers will then return the missing blocks and messages through immutable data item. If a node does not have response from other peers, it means the node has not been widely accepted, so it will just randomly collect blocks and messages. 

Choking peers selection: hash(own ID + timestamp_base_5minutes) XOR peer list

 
Replacement Vector
* This now plays more important roles as: remember invoke failure to avoid local optimization problem, provide candidates to invoke list, holding failed routing vecgor nodes, holding other responsed nodes entry for potential invoke. 
* The vector is limited in size, the reflesh is based on the time a node stay in the vector. 

### Live Signal and Payload ( modified from BEP44 data item)
* a mutable data, each TAU node will remit signal when mainloop provides time slot, in the sigal, it will send to all friends include itself, the timestamp, device id, messages hash levenstein vector, gossip of other nodes( for XX situation, random own friend ID), payload location (hash, node id, end point). 
  * put life signal: direct put, expecting response with referral nodes potentially closer than routing table entries. 
  * get life signal: expecting value and referral nodes
  * Mutable Data
    * Target: 32 bytes
      * first half of the sender public key must match the second half of the target
    * Value: 1000 bytes
      * sender need to sign this value  

Target of Mutable Data: libTAU mutable data aims to exchange data than storage, expecting lots of records overlaping like in the routing table
* 256 bits long
* First 128 bits: First half of the **Friend**  public key
  * If the receiver has public IP and online, the data will be put into receiver memory directly. This design is to create incentive for data relay provider to get a public IP/port and keep alive. This is also why we **do not** hash (salt + pubkey). 
  * The more data provided, the provider's Node ID has more places in other peers routing table
* Second 128 bits: First half of own public key
* Hash and endpoint of Payload item, this item is not intend to store in XOR close nodes, but the end point which life Signal pointing to. 
  * put: direct put into end points, without expectation of nodes referral
  * get: direct get, expect return value without return any referral

### Interface to app developer
* Java package: libTAU4J; C++ lib: libTAU; for x86_64 and arm64

### Mainloop frequency
* each device could setup the range of walking frequency from 1 - 20s, this will also limit the highest data consumption. 

### Bootstrap and time: nodes can get these information from both central and decentral sources 
* from third party bootstrap and time server such as ISP or TAU Dev.
* from community blockchain content, libTAU can config serveral community chains to start follow.
  * blockchain content is safer to validate true time and right phone swarm, however it is slower than third party service. So we adopt a combined approach with blockchain as part of statistical calculation. 
  * all the added blockchains in the friends list will be treated as boot and time info potential providers equivalent to TAU chain.

## Public Key to Public Key communication
The IP protocol requires sender and receiver IP addresses. When IP address is behind NAT or in the private range, the IP connnection between devices is hard to establish. Ideally, each device will have public key. The communcation is conducted between key to key. The under-neath IP connection and routing is handled by protocol. 
We devide TAU server-less communicaiton to be an application layer protocol to faciliate peer to peer connection in following simple command:
* Get in mutable data item
   * When a node, A, wants to get a data. It will search local memory firstly. If not found locally, A will do the `dht_get` 


