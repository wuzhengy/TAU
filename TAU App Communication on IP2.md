# TAU app communication on IP2
TAU app has two types of communication: public key to public key chatting and blockchain gossipping. 

### P2P chatting
Assume data flow is X ->(Y relay)->Y and Y->(X relay)->X

X will maintain a list of self-originated messages with receiving status. For each message, X will send message up to 30 times until Y confirms. There are 5 minutes between X each sending. Y relay will store the latest message for Y to capture when resuming online status. 
X's UI will show the status of each message, so that user can engage to resend or accept that message status. The IP2 capture swarm relay will make best effort for cache and deliver. 
Y will scan Y relay for caches messages each 30 minutes and each time off line for 5 minutes. 
Chatting natively only support text. When user wants to send an image, we will provide a free picture server to basic low resolution image transfer. Sender just sends image link, the receiver will use the link to download the picture. 


有缓存的点对点通信简化版：
X维护一个最近自己发出的消息表比如30个消息，每个新消息发送8次，每次隔开5分钟；这里使用有中继缓存传输。
Y收到消息后反馈接受到，也是发送8次，没你间隔5分钟，雷同新消息。
当X收到Y对这个消息回复，则提示UI消息收到；否则消息就是处于发送状态。
Y每30分钟接受下中继缓存处理新消息。

A - 基于相互朋友列表的莱文斯坦通信
假设：(X往Y发送数据，mutable target形式为”YX” ) X ->YR(UPNP)-> Y, Y -> XR(UPNP) -> X；XR和YR为中继和目标节点的捕捉网络成员；
维护D - 72小时内成功通信的朋友节点历史缓存(本质是长期没有联系的名单的补集），H: 10* log(D)分钟内成功通信的朋友节点历史缓存（避免请求过于频繁），L：莱文斯坦数组未对齐的朋友列表
X.put(Y public key, alpha, beta, invoke, payload)
1. X有给Y的“最新消息”时，这个是专门给UI驱动的，触发”cache relay“。最新消息触发8次，每次之间隔开5分钟，直到获得成功回复，8次以后就不再认为是最新消息，从而依靠来文斯坦数组对齐策略。
2. 同步通信窗口 - 主循环随机在L中挑选Y，满足Y在D中，并且Y不在H中？”cache relay“ 同步来温斯坦数组。
3. Y收到消息后，根据莱文斯坦数组相应处理，触发”traverse put“，alpha=1, beta=2, invoke_number = 3。X收到消息后，根据本地逻辑处理，回复消息给Y，触发”traverse put“，alpha=1, beta=2, invoke_number = 3（类似第二步细节）。到第2步，直到双方莱文斯坦数组对齐。
4. X每次新上线离开上次H分钟，则从捕捉网络获得消息。
5. 步骤2，3当过程由于某种原因中断。X和Y将等待下个通信窗口，或者自己有新消息
6. DHT每15分钟更新XX捕捉网络，触发一次”traverse”，alpha=1, beta=8，invoke=16，DHT协议中15分钟是个ping的指标点。
7. 每个步骤最小时间间隔50ms，或者UI规定的间隔
8. 当UI关注在某个peer Z时，80%的随机资源给到这个Z，Z不受H和D的限制。”traverse put“，alpha=1, beta=2, invoke_number = 1. UI关注这个Y时，X访问Y的间隔为1分钟。
9. XY每天交换下各自区块链表，寻找共同的链，用于重要信息的中继缓存数据。

#### chatting target Meta data: last seen, last communicated
* last seen: last time any signal is received.
* last comminicate: last time a message or confirm of message recieved.

### Blockchain gossipping
In blockchain gossipping, we do not implement cache, everything is instant basis. 


路由表live、rb、DHT bootstrap历史等缓存需要DHT层的数据库存储，在系统启动时装载。

路由表基础设置可以考虑非常大的单层路由表。区块链上节点多了后，主要还是多依靠记忆，降低traverse的消耗。多层路由表会剔除离开原点远的节点，而依靠traverse去不断寻找，我们正好是相反的需求，区块链上的节点都是远处需要长期同步的节点。 


转发集合分成两个列表：转账A和消息B
A列表的转发规则是地址交易费贵优先，nonce保证发送地址唯一性,中位数以上的消息, 是否有 100名问题。 
B的转发规则是最近的5分钟时间窗口，没有nonce问题，只要是上链地址和在A列表里面未来要上链的地址就行。对B来说，策略是转发5分钟内中位数以上的消息。


方案 I
假设： X本机 ->YR -> Y 
- 建立 4个集合控制节点范围选择：
T：链上所有节点；
D：24小时内成功同步过的T上节点，包括其他人请求(类似评分访问列表)
S：短期阻止访问的列表：sqrt(D)分钟内成功同步过的节点历史缓存
B：长期阻止访问的列表：T - D 或者 1/2T，其中较大的集合。节点下线后，随着时间，T逐步都会进入B，所以需要一个释放策略保证B不全覆盖T，这个策略是随机的1/2的T兜底，就是最多长期阻止一半节点，B就不再增加了，只是替换。
## D，S，B都是T的子集, sqrt(D) 上下限为 1 - 5 分钟

- 任务类型：
主动请求任务类型：1. 请求投票；2. 请求head block; 3. 请求block; 
主动交换任务类型：4. “sqrt(D)分钟”内交易费最贵交易，5. 自己发出的交易。6. 自己新挖的区块
V：访问任务历史列表，用于完成任务的顺序为：1. 请求投票->2. 请求head block->3. 请求其它block/推送自己的block等一般性请求，来产生task
Ta：当前task列表

- 过程
1. X随机在（T- B - S）中挑选节点Y，如果Y为空，则等待（等待是好的节约流量的方案）。
2. X利用V触发“全同步traverse Y”：alpha=1, beta=4, invoke_number = 8。请求投票，tip block和相关历史block，“交换”5分钟内交易费最贵交易，“交换”自己的交易。
3. Y处理X的请求数据，Y不向X发出任何数据请求：Y仅仅反馈区块数据给X，和接受发送相关交易，反馈触发”traverse put“，alpha=1, beta=2, invoke_number =2 。
4. X反馈数据给Y，同步持续固定sqrt(D)分钟，宁可等待的（根据不同任务可以细化这个策略），则到第1步。
5. 每次主循环检查最近sqrt(D)分钟的X产生新交易或者新tip，随机在D- S里面选择一个节点Y，触发traverse: alpha=1, beta=4, invoke_number = 8，持续1分钟。
6. 步骤2和3不用莱文斯坦数组，每个步骤间隔根据UI允许的时间间隔比如50ms，或者UI规定的间隔；所有的消息都是交易，交易费可以为0，就是当链没有交易时，用户可以自己上链。

方案 II
区块链通信相关：
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

发交易任务（目标：允许最多100个人的消息被转发，也即最多100个人同时说话，其它人的交易自己处理，社区不帮忙转发）：
1. 非自己的新交易（所谓新消息，即必须本地没有，收到过第二次不会再转发），按照链上规则只转发能进入交易费前100名账户的新交易，每个账户只能发1笔交易，交易费相同但时间更新可替换同账户交易（想严格控制的话，可以要求同账户同nonce交易费必须必上一笔大某个值），转发给评分前3名的peer
2. 自己的新交易，如果交易费不能超过交易池最后一名交易的交易费，随机发给全社区1/3的节点（或者全部？）或者发送最多1分钟，如果超过交易池最后一名交易的交易费（交易池最多保留100个账户），说明大概率在前100名，则转发该交易给评分前3名的peer，也即交给社区帮忙转发



区块链数据添满原理：对于消息类交易，每次后面的信息尽可能打包前面的同nounce信息，到1000字节。



### Mainloop frequency
* each device could setup the range of walking frequency from 1 - 20s, this will also limit the highest data consumption. 
