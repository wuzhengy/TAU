### key problems and assumption
* mobile phones behind telecom cell tower can not establish direct peer to peer IP connection, due to firewalls restricting incoming unknown IP
* mobile device availability is random and quality of network is unpredictable due to the location moving
* no incentive for server to provide free relay services 
* assume P2P idea is wrong... the phones is not supposed to communicate with peer to peer in reciprocal tight format ... but loose coupling pub/sub fashion. It will waste lots of traffic data, but data cost follows moore's law. The classical telecom theory is about connection. What if the connection is not the way to solve serveless problem.
* when relay become essential, the centralization of data is inevitable. The big data centralization cause over-taxing on individual accounts. 


## knowledge building blocks
Along the way of developing TAU, we have adopted many key ideas from many open-source community projects. Following are the key components we are adopting. 
#### Bitcoin
Hash linked chain, central-less and permission-less concensus, on-chain UTXO state
#### Ethereum
On-chain accounts of balance and nounce
#### NXT
Generation signature for mining, proof of stake, chain accumulative difficulty
#### Libtorrent
Distributed cache table for mutable data, salt, ed25519 encryption
#### Levenshtein [Distance](https://en.wikipedia.org/wiki/Levenshtein_distance)
Building up data transmission completeness in the dht network, where nodes are randomly relay different parts of imformation. 
#### Bloom Filter
Checking new blocks with community members block history bloom filters to make sure no secret chain attack is present. 

------
* 1000  stateful
TAU blockchain will only keep a chain state for 1000 days, any state beyond will be forgotten forever. This will make the blockchain in flat size, good for phones. You can view this as **epoch stateful mining** and **stateless verification** combined. 
- mining need be stateful with 1000 x 288 blocks information. 
- verification could be purely stateless. 


* Simple opcodes
TAU aims to enable basic devices to have determined functions in weak networking regions, so we do not add automaton language into transactions and do not engage variable gas concept.
  * type 1:  1 to n wiring, each transaction will allow to send up to 24 addresses. 
  * type 2:  plain text

* Some ranges: 
mutable range: one day, 5 x 12 x 24 = 288 blocks.
new peer range: from current to last new address added block. 
stateful range: 1 year, 288 x 1000= 288K blocks

* Block and Transaction structure
 * add sender and miner IP addresses
 * add new peer pointer
### tech components to solve problems
* DAG - directed acyclic graph: every data item in TAU has both `content` and `link`. TAU content network can be viewed as a DAG. Any data connects another data for representing blocks, messages or images. 
  * Mutable item `content` is a pointer to a DAG node,the key of an immutable item; and link is another public key. 
   - The public key with a salt can form a new pointer. The new pointer nature is much dependent on channels. 
     - For #blk channle, it is another miner. 
     - For #msg, it is a latest message sending address. This pointer is used to make searching more efficient by every peers contributing knowledge.
  * Immutable item is a DAG node. The item `content` is the part of data schema, and `link` is pointing to another immutable item. Each immutable item also include a skip list pointer such as in block structure to point into a history item for speed up searching. 
* DHT - a search engine on D-DAG
* Peer's gossip channel - a place peers to gossip publishing and demand for own and friends. A blockchain community technically can be viewed as a friend in the gossip channel. 
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
I am designing an TAU chain with “epoch stateful with 1000 days rolling base memory without smart contract”. There will be no cut-off line, the state memory is on rolling basis to keep storage flat for each blockchain around maximum 100mb.  
Without smart contract, the blockchain will be purely for coins wiring and text. I think these are the most important things and sufficient for dApps to build logic such as javascript can be viewed as text. Assume in the future, all devices will need libTAU communication for server-less messaging, we want to make this layer to be cheap and efficient in computing resources consumption.

------
### Multimedia
TAU communication is a connection-less protocol, at the current stage, it is not ideal for large amount of data transfer such as video streaming and big files. 
The current app will only support: 
1. Text: some mark-down language
2. Image: compress to 20k webP format which is designed for phone screen size. 
3. On-Chain web site: index note transaction with markdown language to form a linkable site. 


