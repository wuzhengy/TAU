# TAU app communication on IP2
TAU app has two types of communication: public key to public key chatting and blockchain gossipping. 

### P2P chatting
Assume data flow is X ->(Y relay)->Y and Y->(X relay)->X

X will maintain a list of self-originated messages with receiving status. For each message, X will send message up to 30 times until Y confirms. There are 5 minutes between X each sending. Y relay will store the latest message for Y to capture when resuming online status. 
X's UI will show the status of each message, so that user can engage to resend or accept that message status. The IP2 capture swarm relay will make best effort for cache and deliver. 
Y will scan Y relay for caches messages each 30 minutes and each time off line for 5 minutes. 
Chatting natively only support text. When user wants to send an image, we will provide a free picture server to basic low resolution image transfer. Sender just sends image link, the receiver will use the link to download the picture. 


### Blockchain gossipping



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





Read Only 不进入路由表，不成为中继，不成为捕捉网络成员。Read Only节点可以被push协议送达，需要不同于路由表做内存缓存供push协议寻址使用
- 计费节点属于 RO
- NAT后节点属于RO
- 防火墙后节点属于RO
- ipv6全部属于RO，由于目前无法感知是否在防火墙后

Non-Read-Only节点可以进入路由表 
- ipv4 UPNP 并且 DHT探测公网end point和UPNP反馈end point一致
- coded bootstrap public IP addresses is referable, provided by app developers 

路由表live、rb、bootstrap和data item storage需要DHT层的数据库存储，在系统启动时装载

对于本地IPv6地址是否是RO，要模仿libtorrent的voting 方案，就是对与incoming的地址做记录，陌生节点( 非自己路由表节点）访问超过10次，才可以认为是non-RO

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
