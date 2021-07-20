working draft ...
### Key Points
* TAU is designed as protocol for data communication over blockchain. Its applications include crypto coins mining on phones, community serverless communication and low cost IOT interaction. 
  * some known difficulties: 
    * personal phones and devices do not have public accessble IP addresses, they have to relay through certain servers such as whatsApp, telegram, youTube. 
    * lack of incentive for peers to provide data to public internet
    * internet routers and firewalls love to block certain crypto related traffic
    * no trust worthy time and bootstrap service available on internet
### knowledge building blocks
Along the way of developing TAU, we have adopted many key ideas from many open-source community projects. Following are the key components we are adopting. 
#### Bitcoin
Hash linked chain, central-less and permission-less concensus, on-chain UTXO state
#### Ethereum
Accounts design of balance and nounce, stateless future plan
#### NXT
Generation signature for proof of stake mining
#### Libtorrent
DHT based metadata communication
#### Levenshtein [Distance](https://en.wikipedia.org/wiki/Levenshtein_distance)
Building up data transmission integrity in the UDP network, where nodes are randomly relaying different parts of imformation. 
TAU uses Levensthtein array than sequence number in TCP to achieve data transmission integrity. 
#### Bencode, UDP level Encryption
* libTAU use bencode to compose blocks and transactions. Bencode has mixed benefits of partial human readable and binary serilazation. 
* libTAU DHT communication use bencode to build mutable and immutable data item for transmission.
* libTAU UDP package: the payload is encrypted senders public key and data. Only receiver with right private key can decode. The UDP payload level encryption will make router not be able to recognize the protocol for the data. 
#### Friend public key and device ID
* each libTAU session will keep one public key friend list, which is in sync with UI
* one public key node might include many device ID sharing same private key. The device ID is an optional but important information in live signal. System will record this field and report it in the alert to UI, but will not process it. As long as the device has private key, its messages will be processed disregard what device ID it has. As long as private key is same, all devices are treated same. 
#### TAU community ledger serve as bootstrap and time server
For a decentralized system, it is hard to do a trust bootstrap and find a server to trust for time. In TAU, since the choking (multiple prisoners dilemma) requires good time to reach win win solution, nodes need to find out what is the consensused time in the network. 
Rather than bootstrap from fixed nodes, TAU nodes will use current ledger nodes for bootstrap and time calculation. 
##### Bootstraping
libTAU nodes will use all locally available ledger to bootstrap itself. In each TAU block, we recommend adding "end point" into block. When the block is accepted by the node, it will use this info into local bootstrap list. Initial software will come with part of TAUcoin blockchain ledger, which will include initial bootstrap nodes. 


------
* 1000  stateful
TAU blockchain will only keep a chain state for 1000 days, any state beyond will be forgotten forever. This will make the blockchain in flat size, good for phones. You can view this as **epoch stateful mining** and **stateless verification** combined. 
- mining need be stateful with 1000 x 288 blocks information. 
- verification could be purely stateless. 

* Some ranges: 
mutable range: one day, 5 x 12 x 24 = 288 blocks.
stateful range: 1 year, 288 x 1000= 288K blocks

* Block and Transaction special feature
 * add sender and miner IP addresses
### tech components to solve problems
* 64 bits, android 5.1, api 22.
* token: IP address random checking. 
* Data Schema - every mutable item's content is the root of a `data schema` or the first immutable item starting the schema.
  - Schema is series of immutable item together to present a data structure. IPLD protocol has built example of data schema. 
<br><br>
* P2C - Peer to Consensus: every communication is under scope of a chain, which is a type of consensus. This is called `peer to consensus`.The consensus will regulate spam and make content searching efficient within limited nodes. 
* Network session concurrency - TAU config each libtorrent session to minimum foot print, the first session is always ReadOnly to ensure basie communitcation, the 2nd session starts to support DHT read. Within the session, each request is spaced with 1 second to make sure the not causing ban. The current get is congestion-based get, will experiment non-congestion in the future. 
  * We have several parameter on concurrency
    - number of the task queues
    - lengh of the task queues
    - number of sessions
    - put/get time interval in sessions
    - alpha of the search branches
* Avoiding constants - we want to use as little as constants as possible to let system to self adjust to the environment such as consumption of memory and bandwidth with performance results. For example, how many libtorrent nodes to initiate. 
* Gossip protocol to relay signals and enhance the performance
* Comments on Ethereum Stateless and TAU Epoch Statefulness
Each 12 months, on a predetermined date, eth miners will choose to forget all state information prior to the year. The purpose is to make state storage below a flat size. The eth dev calls this “epoch expiry” or “partial stateless”. 
As a miner, you will still need entire information for the recent 12 months. As transaction maker, if you want to make a transaction which need information, such as balance, prior to a year, you need to pay gas fee to bring the state back to the new “epoch”. 
The eth decision shows that as a blockchain, remembering the entire history is not viable, if the blockchain wants to survive for decades.  However the fixed cut-off date is problematic too. Ethereum states are inter-connected among blocks, such as a deFi loan will need history information on interest rate years ago, which might be cut off when new epoch starts. Each dApp has to select what state to bring back into new epoch. This will be quite expensive for dApp users. 
I guess this is quite messy and it might be the reason “stateless” has been in discussion for 7 years. 
As TAU is in the design stage, we need to learn from this trouble. The root cause is the eth smart contracts generating massive states. Bitcoin does not support smart contracts like eth, so its state storage are much lower and acceptable for the chain to keep all the history. 
TAU is designed to run a phones, so the situation is even worse than the server based. I think we are going to be avoid of both “stateful - remember the whole history” and “smart contracts”. 
I am designing an TAU chain with “epoch stateful with 365 days rolling base memory without smart contract”. There will be no cut-off line, the state memory is on rolling basis to keep storage flat for each blockchain around maximum 100mb.  
Without smart contract, the blockchain will be purely for coins wiring and text. I think these are the most important things and sufficient for dApps to build logic such as javascript can be viewed as text. Assume in the future, all devices will need libTAU communication for server-less messaging, we want to make this layer to be cheap and efficient in computing resources consumption.
* Reduce the peer number space. We design serverless communication blockchain to only make one transaction per block, which means one sender one receiver, this will bring the total peer numbers under 288 * 365 = 105,120；the 6th root of which is 7, which is the swarm size of blockchain communication. 6th root is a good social distance. 
* This means one TAU community chain can only hold 105,120 peers at current network phone condition, until next personal phone upgrade like 5G complete mature. 

------
### Multimedia
TAU communication is a connection-less protocol, at the current stage, it is not ideal for large amount of data transfer such as video streaming and big files. 
The current app will only support: 
1. Text: some mark-down language
2. Image: compress to 20k webP format which is designed for phone screen size. 
3. On-Chain web site: index note transaction with markdown language to form a linkable site. 

------
Special block elements
* total number of peers in recent 1000 days, this is for group communication peers selection random distribution
* lastNewPeer, this is for fast sync peers list <br><br>

In many ways, TAU is really a blockchain designed for communication. 

------
Blockchain v3 should a technology for communication
* Blockchain v1, bitcoin, is a technology for server-less digital asset, which turns our to be quite successful. Blockchain v2, ethereum, is a technology for server-less financial transactions. The momentum is quite recognized recently.
So what is blockchain v3? One of the most important modern world features is digital communication. I think blockchain technology will renovate this sector.
* Here is a big technical problem for server-less group communication. When peers number increases in the group, the synchronization efficiency becomes low due to no central place to host information. Assume you have 1 million users in a group, the sync complexity is N square, which is 1 m x 1m, which is not possible for current network to handle. We have to come up with a better design. The TAU current plan is to restrict total chain size so as the peer numbers. The 1 years length, 1 tx per block, 5 minutes per block plan will bring us about 100,000 peers on the whole chain. The tx speed is low here but if blockchain as communication peer registration, it should be acceptable since the real messaging volume is unbounded.
* Further, we plan to engage torrent choking design, means in any moment, a peer only exchange data with a set number peers on the "tit for tat" basis. The number is 6th root of whole chain peers N. The 6th root is a magic number that in social media 6th steps will bring connection between any two persons in the world. This allows the whole traverse complexity of a message to be under O(6) which is quite good.
* The clarity of above thinking will make following design decision much easier to proceed:
* epoch stateful chain rather than stateful and stateless choice, which is ethereum future plan
* no smart contract on the chain
* block structure includes the number N and last new peer.
* assume P2P idea is wrong... the phones is not supposed to communicate with peer to peer in reciprocal tight format ... but loose coupling pub/sub fashion. It will waste lots of traffic data, but data cost follows moore's law. The classical telecom theory is about connection. What if the connection is not the way to solve serveless problem.

* choking mechanism in blockchain mining, for each nodes synching blockchain ledger, collectively there are less incentive for any node to give data to stranger nodes. so far the working last resort is the torrent choking plan, in any certain time window, only a subset of nodes are suppose to open access to each other and each will use tit for tac accounting for exchange data. So that the nodes do not want to share data at all will found hard to receive data from the un-choked peers at a certain time frame. in libTAU, we use blockchain time to arrange planned communications paire between nodes, in each time, each nodes will know exactly what nodes to talk to. when communication starts, bothsides will have to provide data to each other. this is probabaly the most important thing in libTAU blockchain data exchange. 

