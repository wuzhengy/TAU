working notes ... read at own risks
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

PePOS - Perishable Prove of Stake
#### Libtorrent
DHT based metadata communication
#### Levenshtein [Distance](https://en.wikipedia.org/wiki/Levenshtein_distance)
Building up data transmission integrity in the UDP network, where nodes are randomly relaying different parts of imformation. 
TAU uses Levensthtein array than sequence number in TCP to achieve data transmission integrity. 
#### Bencode, UDP level Encryption
* libTAU use bencode to compose blocks and transactions. Bencode has mixed benefits of partial human readable and binary serilazation. 
* libTAU DHT communication use bencode to build mutable and immutable data item for transmission.
* libTAU UDP package: the payload is composed of relay node public key and encrypted bencoded data entry, that is signed by original sender. Only receiver with right private key can decode payload. The UDP payload level encryption will make router not be able to recognize the protocol for the data. The risk here is relay node public key is exposed. So our privacy level is the same as bitcoin, pseudo-privacy, that is ip address can be found associated with public key.  
#### Friend public key and device ID
* each libTAU session will keep one public key friend list, which is in sync with UI
* one public key node might include many device ID sharing same private key. The device ID is an optional but important information in live signal. System will record this field and report it in the alert to UI, but will not process it. As long as the device has private key, its messages will be processed disregard what device ID it has. As long as private key is same, all devices are treated same. 
#### TAU community ledger serve as bootstrap and time server
For a decentralized system, it is hard to do a trust bootstrap and find a server to trust for time. In TAU, since the choking (multiple prisoners dilemma) requires good time to reach win win solution, nodes need to find out what is the consensused time in the network. 
Rather than bootstrap from fixed nodes, TAU nodes will use current ledger nodes for bootstrap and time calculation. 
#### IP address reachability and relay
In libTorrent, the routing table entry has 4 fields: ID, IP + Port, round trip time and pinged flag. The pinged flag means whether this node is reachable from host. However this could be bluffed, if only relying on own. Due to libTAU public key, everytime a node to discover the new nodes, in the ping field, it could record referrals public key. When more than two public key referred this node with same IP address, it means that this IP address is reachable by public. Of course, the more referred, the better. <br>
##### Bootstraping
libTAU nodes will use all locally available ledger to bootstrap itself. In each TAU block, we recommend adding "end point" into block. When the block is accepted by the node, it will use this info into local bootstrap list. Initial software will come with part of TAUcoin blockchain ledger, which will include initial bootstrap nodes. 
##### Perishable State Chain - Pepos - Perishable Proof of Stake
TAU blockchain will only keep the chain state for six months only, any state and ledger beyond will be forgotten forever. This is probably mostly important inovation of TAU. It will make the blockchain in small size, good for communication functions and small community. 
TAU believes blockchain is not for instant cash payment due to the block latency. Financially, it is for digital assets and savings. It is also for community building and communication. 
A limited state size will also make inter-peers communication in controlable volumn. Assume 1 million peers existing in one chain, the data sync effort will be too big for small devices. On TAU chain, one chain will accept mostly 50k participants to make sure data sync is not overwhelmed. This is a quite big community already.
In a perishable chain, the main ledger is in nature a state chain, when each block evolves, it will reflec a new state. So when deleting a history state, basically just delete a history block, which is much easier than deleting a state in MPT in ethereum. Therefore, tau chain should really called as a "state chain", in each block, an account value is updated all time for all participants, sender, receiver and miners. This is why we can not support smart contract on tau, since contract could be deleted in history with its state still in operation, then this is a dizaster.  
* As a miner, you will maximum need 365 x 180 blocks information. However mining can start from any time, as soon as consensus point is setup.
* State database vs state trie
  * eth use state trie to record each variable and value. In libTAU stateless plan adopted, each variable will only live for certain unique time window, not epochs, the trie key-value data structure is hard to maintain such expiry. 
  * libTAU will use relationship database such as sqlite to hold state date with time window.  
##### DDOS attack
Creating big number of IP pool to request services from a phone node will abuse phone's computing power and bandwidth. libTorrent uses "token refreshing" to regulate such behavior. But the token acknowledgement will cause extra waste of bandwidth and prolonged latency. 
In libTAU, since each node has its friends, routing table and blockchain peers public key memory, it will be efficient just to use public key to identify the source. One libTau node will allocate 50% of the resources for unknown public key messages, these are mostly relaying data for other nodes with closer publickey prefix, the reward for this, the node's address will be recorded in others routing table for easier discovery and also for relay. 
This feature will be inplemented after mainnet on, assume initially we will not have too many attackers. 
##### Ranges: 
* consensus point: current block number - 200, then cloeset previous block with 00 ending. for example current block is 86789;  86789-200 = 86589, the 86500 is the consensus point. 
* stateful range: 6 months, 288 x 180 = 51840
##### Block and Transaction special feature
 * add sender and miner end points, IP + port. This is for trustless bootstrap.

### ipv4 and ipv6
ipv6 provides a stable bootstrap entry. we should support ipv6 as much as possible when data is not meterred. 
ISP will love to control incoming packets, the local routing table database will need to be smart enough to measure such restrictions. 

### Working mode
libTAU working config file: secrete key, bandwidth, invoke, 
* UI will assign daily bankwidth data and maximum invoke to libTAU.
* UI will deside to start v4 or v6 interface 
* UI will assign friend list of public keys and chain IDs
* UI will meature system cpu and memory and other factors to ajust bandwidth and invoke number

UI will collect platform info such as:
* charging or not
* bandwidth meterred or not and user preference of data spending
* upnp status
* memory
* cpu
* ipv4 or ipv6

Based on these info, UI will config invoke number of libTAU and how many socket interface needed in session

##### Why not smart contract
Smart contract is able to generate big numbers of state, TAU is designed to run on a phone and support communication and payments. A Turing complete programable language will swallow the network resources. I am designing an TAU chain with “365 days rolling base memory without smart contract”. There will be no cut-off line, the state memory is on rolling basis to keep storage flat for each blockchain around maximum 100mb.  
Without smart contract, the blockchain will be purely for coins wiring and text. I think these are the most important things and sufficient for dApps to build logic such as javascript can be viewed as text. Assume in the future, all devices will need libTAU communication for server-less messaging, we want to make this layer to be cheap and efficient in computing resources consumption.
Smart contract requires contract to be immutable, which is contradictary to perishable blockchain concept. 
* Reduce the peer number space. We design serverless communication blockchain to only make one transaction per block, which means one sender one receiver, this will bring the total peer numbers under 288 * 365 = 105,120；the 6th root of that is 7, which is the swarm size of blockchain communication. 6th root is a reasonable social distance cap. 
* This means one TAU community chain can only hold 105,120 peers at current network phone condition, until next personal phone upgrade like 5G complete mature. 

##### Why not multimedia on DHT
TAU communication is a connection-less protocol, at the current stage, it is not ideal for large amount of data transfer such as video streaming and big files. 
The current app will only support: 
1. Text: some mark-down language
2. On-Chain web: note transaction with markdown language to form a linkable site. 

##### Choking in limited number of peers
When peers number increases in the blockchain, the data synchronization efficiency becomes low due to no central place to host information. Assume you have 1 million users in a group, the sync complexity is N square, which is 1 m x 1m, which is not possible for current network to handle. We have to come up with a better design. The TAU current plan is to restrict total chain size, so as the peer numbers. The 1 years length, 1 transaction per block, 5 minutes per block plan will bring us about 105,120 peers on the whole chain. 
* we plan to engage torrent choking design, means in any moment, a peer only exchange data with a set number peers. The number is 6th root of whole chain peers N. The 6th root is a magic number that in social media 6th steps will bring connection between any two persons in the world. This allows the traverse steps of a message to be under O(6) which is quite good.
* For each nodes synching blockchain ledger, collectively there are less incentive for any node to give data to stranger nodes. The choking plan, in any certain time window, only a subset of nodes are suppose to open access to each other and each will use "tit for tac" to exchange data. So that the nodes do not want to share data will found hard to receive data from the un-choked peers at a certain time frame. 
* libTAU, we use blockchain time to arrange planned communications paire between nodes, in each time, each nodes will know exactly what nodes to talk to. when communication starts, bothsides will have to provide data to each other. this is probabaly the most important thing in libTAU blockchain data exchange. 
* libTAU will use 6 blocks hash prior and include the immutable point to decide peers for unchoke. 

### voting the consensus point
Each node will collect voting from all peers through choking communication. One of the 6 choking peers are randomly select from state database. 
The voting is collecting opinions of top 21 higher staker it encounters, the simple majority wins the consenus point. Anything prior to the consensus point is considerred legit and do not need verification. These blocks will enter state db without math checking. 
Each node software can decide own voting strategy, it is a very individual thing for nodes. The variety of such strategy can increase the resistence of attacking. 
The choking communication serve a good base for such voting, since 5 of the 6 time stamp related nodes are suppose to do two way. 1 of the random nodes is only collection information only, kind of one way voting any way. 
The initial consensus point is when user getting information from referral. A node will always have consensus point. 
