# TAU DHT extention to libtorrent/bittorrent mainline dht (bep 44)
## draft 0.01
The purpose of the extention is that mainline dht does not specify the strategy for republish and hash-chain. In order to run a blockchain, TAU DHT will set up such strategy.  
The global TAU DHT network is viewed as **"memory"** of the ultimate global computer, storage of each personal devices will exchange data between "memory" and local disk. However, a simple search for a value will incur O(N) level complexity, which is not acceptable for decentralized app. Block hash chain technology and referral public key aims to reduce to O(logN) for searching. The hash is a local peer knowledge of blockchain and timely state of messaging. This idea inherits from Dynamic Programming course of MIT 6.006, Prof. Erik Demaine. <br>
Innovation:
* in Mutable item: include another public key in the value for traverse the dht under same salt.
* in Immutable item: implement repub of a history block

## Mutable item put
Each mutalble item, such as A, value will refer another public key, B. The B is the lastest changed public key in A's knowledge under the same salt. this helps use the latest knowledge of each peer to search in DHT. 
No republish scheme in mutable item.
## Immutable item put
Implement republish/reannouce of a random history blocks on the same chain(salt). any block include a hash link to parent block. 

# p2p messaging extention
p2p messaging via logN strategy.
each p2p message to A, its content includes a hash which is another public key recently messaging to A. 

in each TAU message either immtuable or mutable, there are hash point in content to make search N complexity to log(N). this is a type of DQ algorithm.
contact: public key + salt1..saltN
# dht global memory 
key: public key + salt
value: blockHash/chat/pk_encrypted message + hashlink/affiliated public key

invite link schema:  salt + sender PK + receiver PK
salt schema: chainID#channel; chatgroupID#channel; 
