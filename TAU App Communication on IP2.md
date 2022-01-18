# TAU app communication on IP2
TAU app has two types of communication: public key to public key chatting and blockchain gossipping. 

### P2P chatting
Assume data flow is X ->(Y relay)->Y and Y->(X relay)->X

X will maintain a list of self-originated messages with receiving status. For each message, X will send message up to 30 times until Y confirms. There are 5 minutes between X each sending. Y relay will store the latest message for Y to capture when resuming online status. 
X's UI will show the status of each message, so that user can engage to resend or accept that message status. The IP2 capture swarm relay will make best effort for cache and deliver. 
Y will scan Y relay for caches messages each 30 minutes and each time off line for 5 minutes. 
Chatting natively only support text. When user wants to send an image, we will provide a free picture server to basic low resolution image transfer. Sender just sends image link, the receiver will use the link to download the picture. 

#### chatting target Meta data: last seen, last communicated
* last seen: last time any signal is received.
* last comminicate: last time a message or confirm of message recieved.

### Blockchain gossipping
In blockchain gossipping, we do not implement cache, everything is instant basis. 



路由表live、rb、DHT bootstrap历史等缓存需要DHT层的数据库存储，在系统启动时装载。

路由表基础设置可以考虑非常大的单层路由表。区块链上节点多了后，主要还是多依靠记忆，降低traverse的消耗。多层路由表会剔除离开原点远的节点，而依靠traverse去不断寻找，我们正好是相反的需求，区块链上的节点都是远处需要长期同步的节点。 


路由表live、rb、bootstrap和data item storage需要DHT层的数据库存储，在系统启动时装载

对于本地IPv6地址是否是RO，要模仿libtorrent的voting 方案，就是对与incoming的地址做记录，陌生节点( 非自己路由表节点）访问超过10次，才可以认为是non-RO






### Mainloop frequency
* each device could setup the range of walking frequency from 1 - 20s, this will also limit the highest data consumption. 
