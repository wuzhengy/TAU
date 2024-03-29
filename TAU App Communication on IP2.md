# TAU app communication on IP2
TAU app has two types of communication
* public key to public key chatting: this is used for user direct messaging, in current mobile network condition, we will only suggest sending text. For image and video, it is better to send a link pointing to a server resource. 
* blockchain gossipping: this is for reaching PPOT consensus among nodes. TAU app makes each phone a full node disregard phone's network location in restricted or public. 

## TAU focuses on saving user mobile data to fit into data expensive region. 

### libTAU is an implementation of chatting and PPOT consensus on IP2
* C++ open source library
* SWIG interface for Java, especially android platform
* Shell interfece for Linux command line

### P2P chatting algorithm
Assume data flow is X ->(Y relay)->Y and Y->(X relay)->X; Y relay is a concept in IP2 protocol belong to Y capture swarm

```
分为探测、缓存、“信息发送”和“暂不联系”四个阶段
1. 探测：X当app新启动（不包含doze）、有给Y的新信息，或有未送达Y的以往信息每到15分钟，发送任意一个未送达携带来文斯坦数组的信息给Y(由于缓存，需要带消息)，如果Y有回复，则进入所有“信息发送”阶段；Y收到X探测信息，也进入信息发送。X没有任何可以发送Y的信息时，不探测，以节约流量。
2. 缓存：X当app新启动(不包含doze)和每60分钟收取一次全朋友列表信息缓存，作为兜底通信。DHT层将设计put/get压盏策略，保留多个信息。从缓存收到新信息，不会触发探测和发送动作。
3. 发送：X收到Y的消息后，则进入信息发送阶段，发送未送达所有信息，和用来文斯坦数组反馈新收到的信息。
4. 不联系：X在24小时没有收到来自Y的任何缓存和发送信息，则不再联系，直到自己用户发送新信息，和接收到对方Y的探测信息，重置24小时计时。
* 来温斯坦数组放在正文里面发送。
```
chatting receiver status: last seen, last communicated
* last seen: last time any signal is received.
* last comminicate: last time a message or confirm of message recieved.

### Blockchain gossipping
In blockchain gossipping, we use non-cached transmission on the assumption that blockchain relay on multiple nodes to remember the consensus. Please check IP2 protocol API (cached vs non-cached)

Each node will maintain a **sync list**  of connected nodes to keep hot gossip channels. The list will include a few nodes such as between 2 to 5. When list has less than 2 nodes, it will recruite more nodes; when more than 5, it will kick out low score nodes. A scoring strategy is applied. High score is 100, when a node got 0, it will be kick out and added to **punish list**. TAU also maintains **off-line list**.
In the off-line list, nodes will be blocked from request for 5 minutes, but responsing to this node will take such node out of the list. The punished time is from 5 minutes up to 1 hours, each time the node enter into off-line list, the punish time increases.  
Node will also maintain **history list**, which remembers the activities of a node to give score to a certain node. 
Data forwarding: the protocol maintains two lists. 
* list for wiring transaction gossip: apply "always forward highest fee transaction for each account", use nounce to control replay attack
* list for note transaction gossip: apply "always forward transaction fee above median fee happenning in recent 15 minutes", note does not affact transaction nounce. 

```

4个列表：
* 访问列表：保存于内存，记录当前访问的peer及其积分，列表peer数量可在2-5之间波动，低于2则随机找节点，高于5则踢出低分节点。访问列表记录各个节点表现打分情况，进入评分列表则有初始分30分，最低可扣至0分（只有零分者会被踢出至惩罚节点），最高可积累分数100分
* off-line list不在线名单, any nodes in off-line list will have to wait sqrt(total nodes) time. 
* 惩罚列表：保存于内存，记录罚分的节点，以及惩罚次数和惩罚起始时间，最多可记录链上所有节点。惩罚时间可以与惩罚次数挂钩，比如正比关系或者加倍关系，每多进一次惩罚列表，就多惩罚5min或者多一倍惩罚时间，起始惩罚可设为5min，最高惩罚时间可设为1h; 也可以有减刑策略，比如自己如果主动挑中访问，可减刑3s；
* 历史列表：会记录一些历史信息，用于辅助加罚分。比如上次与列表节点通信阶段、数据类型及时间等通信历史信息，用于控制通信内容等。节点从第一次进入访问列表，请求任务的顺序为：1. 请求投票->2. 请求head block->3. 请求其它block/推送自己的block等一般性请求

每次交互都必有应答，不管是请求数据还是推送数据，也就是严格请求-应答模式。对于发出的请求，如果成功应答，+3分；无应答则-5分；对于推送的有效数据，比如head/max fee tx，也+1分；对于发来的请求，响应并-1分。

主循环任务：
1. 控制task发送间隔，每次循环只发一个
2. 检查各peer所处的访问阶段，是否需要增删peer以及是否需要向各peer发起新请求
3. 其它区块链业务，包括检查出块（出块了则添加至tasks队列，等待通知访问列表peer）,检查自己是否有同步需求（有需求则加入tasks队列，等待向访问列表peer请求）等

转发集合分成两个列表：转账A和消息B
A 列表的转发规则是地址交易费贵优先，nonce保证发送地址唯一性,中位数以上的消息。 
B 的转发规则是最近的15分钟时间窗口，没有nonce问题，只要是上链地址和在A列表里面未来要上链的地址就行。对B来说，策略是转发15分钟内中位数以上的消息。

任务类型：
主动请求任务类型：1. 请求投票；2. 请求head block; 3. 请求block;
主动推送任务类型：1. 推送head block; 2. 推送有效tx

每个步骤最小时间间隔50ms；当UI关注在某个community时，80%的主循环资源给到这个Z

快速跟踪链和慢速跟踪链。 对于自己币多的链，快速跟踪。自己不持币的链，满足跟踪。每个节点都第一时间请求自己的state信息。
```

### Mainloop frequency
* each device could setup the range of walking frequency from 1 - 20s, this will also limit the highest data consumption according to user perference. 
