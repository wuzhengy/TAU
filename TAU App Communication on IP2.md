# TAU app communication on IP2
TAU app has two types of communication
* public key to public key chatting: this is used for user direct messaging, in current mobile network condition, we will only suggest sending text. For image and video, it is better to send a link pointing to a server resource. 
* blockchain gossipping: this is for reaching PPOT consensus among nodes. TAU app makes each phone a full node disregard phone's network location in restricted or public. 

### libTAU is an implementaiton of chatting and PPOT consensus on IP2
* C++ open source library
* SWIG interface for Java, especially android platform
* Shell interfece for Linux command line

### P2P chatting algorithm
Assume data flow is X ->(Y relay)->Y and Y->(X relay)->X; relay is a concept in IP2 protocol and a node belong to a node capture swarm vector

* X will maintain a list of self-originated messages with receiving status. For each message, X will send cached message up to 8 times until Y confirms. There are 5 minutes between X each sending. "Y relay" will store the latest message for Y to capture when Y resuming online status. 
* X's app UI will show the status of each message, so that user can engage to resend or accept that message status. The Y capture swarm relay will make best effort for cache and deliver to Y. 
* Y will scan Y relay for caches messages each 30 minutes and each time in off-line for 5 minutes. 
* Chatting natively only support text. When user wants to send an image, TAU will provide a optional free picture server for low resolution image transfer. Sender just sends image link, the receiver will use the link to download the picture. This server is only for optinal image use and not affacting text transmission decentralized nature. 

```
有缓存的点对点通信简化版：
* X维护一个最近自己发出的消息表比如30个消息，每个新消息发送8次，每次隔开5分钟；这里使用有中继缓存传输。
* Y收到消息后反馈接受到，也是发送8次，间隔5分钟，雷同新消息。
* 当X收到Y对这个消息回复，则提示UI消息收到；否则消息就是处于发送状态。
* Y每30分钟接受下中继缓存处理新消息，或者每下线5分钟。
```

#### chatting receiver status: last seen, last communicated
* last seen: last time any signal is received.
* last comminicate: last time a message or confirm of message recieved.

### Blockchain gossipping
In blockchain gossipping, we do not implement cached sending, everything is instant basis. Please check IP2 protocol API (cached vs non-cached)

Data forwarding: the protocol maintains two lists. 
* list for wiring transaction gossip: apply "always forward highest fee transaction for each account", use nounce to control replay attack
* list for note transaction gossip: apply "always forward transaction fee above median fee happenning in recent 15 minutes", note does not affact transaction nounce. 

```
转发集合分成两个列表：转账A和消息B
A列表的转发规则是地址交易费贵优先，nonce保证发送地址唯一性,中位数以上的消息, 是否有 100名问题?。 
B的转发规则是最近的5分钟时间窗口，没有nonce问题，只要是上链地址和在A列表里面未来要上链的地址就行。对B来说，策略是转发5分钟内中位数以上的消息。

任务类型：
主动请求任务类型：1. 请求投票；2. 请求head block; 3. 请求block;
主动推送任务类型：1. 推送head block; 2. 推送有效tx

三个列表：
访问列表：保存于内存，记录当前访问的peer及其积分，列表peer数量可在2-5之间波动，低于2则随机找节点，高于5则踢出低分节点。访问列表记录各个节点表现打分情况，进入评分列表则有初始分30分，最低可扣至0分（只有零分者会被踢出至惩罚节点），最高可积累分数100分
惩罚列表：保存于内存，记录罚分的节点，以及惩罚次数和惩罚起始时间，最多可记录链上所有节点。惩罚时间可以与惩罚次数挂钩，比如正比关系或者加倍关系，每多进一次惩罚列表，就多惩罚5min或者多一倍惩罚时间，起始惩罚可设为5min，最高惩罚时间可设为1h; 也可以有减刑策略，比如自己如果主动挑中访问，可减刑3s；
历史列表：会记录一些历史信息，用于辅助加罚分。比如上次与列表节点通信阶段、数据类型及时间等通信历史信息，用于控制通信内容等。节点从第一次进入访问列表，请求任务的顺序为：1. 请求投票->2. 请求head block->3. 请求其它block/推送自己的block等一般性请求

每次交互都必有应答，不管是请求数据还是推送数据，也就是严格请求-应答模式。所有主动请求或者主动推送：alpha=1, beta=4, invoke_number = 8，所有回应：alpha=1, beta=2, invoke_number = 2。对于发出的请求，如果成功应答，+3分；无应答则-5分；对于推送的有效数据，比如head/max fee tx，也+1分；对于发来的请求，响应并-1分。

主循环任务：
1. 控制task发送间隔，每次循环只发一个
2. 检查各peer所处的访问阶段，是否需要增删peer以及是否需要向各peer发起新请求
3. 其它区块链业务，包括检查出块（出块了则添加至tasks队列，等待通知访问列表peer）,检查自己是否有同步需求（有需求则加入tasks队列，等待向访问列表peer请求）等
```


### Mainloop frequency
* each device could setup the range of walking frequency from 1 - 20s, this will also limit the highest data consumption according to user perference. 
