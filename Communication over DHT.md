# TAU communication on DHT
TAU communicaiton protocol is modified from libtorrent Mainline DHT. Major changes has been introduced in incentive system, ID, routing, bootstrap,time service, data item structure and blockchain support. TAU uses public_key to identify communication target, than IP endpoint. Therefore, when a device does not have a real IP address, it still can communicate independantly without attached to a server. DHT swarm overall becomes relay services. TAUcoin community is aiming for bootstraping network and community timestamp service. 

通信描述

					莱文斯坦		中继缓存		
A. 朋友列表间通信		1 				1，仅存非莱文斯坦信息		
B. 区块链内通信			0				0	
		
A - 基于相互朋友列表的中继缓存莱文斯坦通信
假设：(X往Y发送数据，target组合YX ) X ->YR（UPNP） -> Y ； Y -> XR -> X；R为中继和目标节点的捕捉网络成员；put(receiver address, alpha, beta, payload, bool cache)
1. （本地有新消息）或（与上次通信间隔10分钟 并且 莱文斯坦数组24小时内始终没有对齐），触发一次”traverse put”，alpha=1, beta=10，信息bencode中含有XR，XR end point 和 “时间戳“ 。 YR缓存非莱文斯坦数组数据。（本地需要建立全朋友列表对方莱文斯坦数组的数据和last seen时间) ，XR是否缓存类似replacement bucket的思路
2. Y收到消息后，根据莱文斯坦数组相应处理，触发”traverse put“，alpha=1, beta=3，把第一步的XR信息融入路由表的m_result一起搜索，信息中附带YR’，YR‘ end point和时间戳。XR缓存消息数据。 
3. X收到消息后，根据本地逻辑处理，回复消息给Y，触发”traverse put“，alpha=1, beta=3（类似第二步细节）。到第2步，直到双方莱文斯坦数组对齐，没有新消息发送。
4. 步骤2和3，当过程由于某种原因中断。X和Y将等待下个10分钟通信窗口，或者自己有新消息，或者重新上线。
5. XR会存储交互的数据，当X重新上线updateCaptureSwarm(本质 “*X” traverse get，alpha=1, beta=20)，XR会把缓存数据提交给X。缓存时间段最长为一天，或者X last seen之后的时间。updateCatureSwarm(alpha, beta, timestamp)。XR把缓存数据交给X的动作应该是XR的DHT业务，原因：1. X如果根据反馈向XR触发所有*X的缓存数据，耗费X流量。2. X每次触发的XR不能保证稳定的，XR是个swarm。
6. 当轮到X -> X的情况触发一次”traverse put”，alpha=1, beta=10。这个动作是更新捕捉网络。
7. 每个步骤最小时间间隔50ms

B - 区块链的无缓存无序列通信
假设： X ->YR -> Y ； Y -> XR -> X 
1. 节点X随机选择一个链上节点Y，先判断自己是否有需要请求的通信内容，内容有tip区块，自己需要的区块，当前5分钟内最贵交易，自己的交易，如果自己收集完全状态则投票。如果存在需要通信的内容，触发一次”traverse put”，alpha=1, 信息bencode中含有XR，XR end point。 YR不缓存数据， cache = false。如果这个节点10分钟内通信过beta=1，如果这个节点10分钟内没有通信过beta = 10
2. Y处理X的请求数据，Y不向X发出数据请求：Y反馈数据给X，触发”traverse put“，alpha=1, beta=1，把第一步的XR信息融入路由表一起搜索，信息中附带YR’，YR‘ end point。XR不缓存数据, cache = false.
3. X通过YR反馈数据给Y，类似第二步。通信持续到（双方15秒内没有新通信需求 或者 通信时间达到1分钟），则X主动选择另外一个区块链的随机节点Y‘。
4. 步骤2和3不用莱文斯坦数组。
5. XR和YR不存储区块链的交互数据
6. 每个步骤间隔根据UI允许的时间间隔控制


缓存节点的遍历。 

今天这个问题是在DDOS范畴内了，以前没仔细思考，目前有这个思路，大家可以辩论下：假设：X - YR - Y情况。如果X和Y是合理的通信，也就是非DDOS，X 是可以从交互中积累Y的签名数据，我们这里引入一个“Y对当前时间戳”的签名的新概念，这个是我们ID体系的独特优势。YR对X的PUSH请求，可以要求X在TO字段携带最近一次Y的时间戳签名，如果可以验证，就执行push。我们这里假设YR是处在UPNP情况下对流量不敏感，而且电信对UPNP会有底层DDOS保护。同时考虑到第一次X对Y通信请求是没有签名信息的，可以对24小时内的X前100次给予放行。对时间戳的签名是足够安全对抗DDOS，这个思路看起来把防火墙DDOS的问题也解决了。其实这个方案就是Y在辅助信息中再增加加时间戳签名，让YR中继把反馈回来的数据放行到push Y。 如果恰好交替在线，100次机会用完了，那估计只有等待第二天联系了；链上100次机会用完，只能等被选中，才能继续通信
是有这个可能，X-YR-Y, Y可以把签名中包含X地址和自己时间戳，中继缓存XR会送达到X，X上线后从缓存中获得时间戳在24小时内YR就可以放行。100次的机会，只要一次通过就reset，所以发送方为了节约流量，可以随机的每80次包含下签名，降低流量。其实辅助网络信息也可以同理降低流量。第一次带辅助信息最重要。这个目前先就是理论探讨，实现不急的。




## Public Key format
* 32 bytes ED25519 key-pair generated from random seed
* Node ID is the public key for the incentive framework, nodes with closer prefix will have an incentive to help each other relaying the data. When a node refuses to relay data for other similar prefix nodes, it will found hard to get its requested data from close prefix neighboors, since peers will not add lazy nodes into routing table for referral. Nodes referral is a constant process in libTAU to locate other peers.  
* ECEIS is used for peer to peer content encryption. We use this to achieve deep level UDP payload encryption. 

## Routing table in DHT
* Tuple: Node_ID, IP, Port, referee list, round trip time - rtt, contact timestamps vector, ping status, failure counts
  * referee list is who has reported this nodeID and IP/Port combination. The more referees, the safer this node is reachable from public internet. 
  * contact timestamp vector - recording the history of communications
  * Single Layer routing table rather than multiple layers like libtorrent. This will make routing vector filled with multiple clusters(prefix groups). So that we need bigger size of the array for routing table. 
  * selection of nodes: each main loop slot chose one feature: closer range, success nodes. 
 
## Friends list Meta data: last seen, last communicated
* last seen: last time life signal is received.
* last comminicate: last time message recieved.

## Main loop and depth traversal
* Kademlia DHT uses depth traversal to find nodes. 
* libTAU added a top layer mainloop to constantly locate friends and blockchain peers. This is the key to avoid dedicated server. 
  * the returned nodes will be added to replacement buckets for future main looping selection. 

## Communication token 
* Kademlia uses communication token in the get process to control spam
* libTAU replace the mutable data concept with every mutable data is called "life signal". We will rely node id whcih is public key checking and limit counting to avoid spamming. 


### Design live nodes, replacement buckets, m_list, alpha and beta
* Live nodes: in libtorrent, there are initial 8 nodes are picked into m_list for traversal model to start searching. but initial searching are start from alpha, which is set as 5. 
* replacement buckets: in libtorrent, there is where incoming node id, referred nodes or some failed live nodes are stored to server as candidates for m_list
* m_list: in libtorrent, this is the temporary list for traveral with sorted distance to the target
* alpha is initial selected number of nodes from live vector, beta is the nodes from replacement. 
* in libTAU, we inherites this arrangement; however due to libTAU traversal is no long recursive but limited to fixed number of friends exchanging live signals. every time traveral model only pick up alpha + beta number for nodes from m_list to invoke query the finish. alpha, we will use alpha as 1, beta as 1, which basically give chances to both live and replacement buckets. i think this will avoid local optimization also increase the main loop frequence. 

#### Choking communication
* range: onchain nodes + transaction pool ID
In peer friends communication, peers life signal to other friends. In the same idea, blockchain ID is a special type of friend, one peer will send three signals into its blockchain mutable life signal, as well as payload end point: 
* `current tip` block(include consens point) and blocks demand, 
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
* First 128 bits: First half of own public key
* 2nd 128 bits: First half of the **Friend**  public key
  * If the receiver has public IP and online, the data will be put into receiver memory directly. This design is to create incentive for data relay provider to get a public IP/port and keep alive. This is also why we **do not** hash (salt + pubkey). 
  * The more data provided, the provider's Node ID has more places in other peers routing table

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
