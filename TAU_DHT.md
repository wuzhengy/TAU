# TAU DHT extention to libtorrent/bitorrent mainline dht
## draft 0.01
For blockchain operation, we modify dht "put" item method on top of mainline dht, which is an form of kademilia dht. 
The purpose of the extention is that mainline dht does not implement the republish and RSS hash chain. We are using blockchain to replace the RSS hash chain. 
The global TAU DHT network is viewed as "memory" of the ultimate global computer, each personal devices storage will exchange data between "memory" and local disk. The simple search for a key will incur a O(N) level complexity, which is not acceptable for computer. Hash-link technology aims to reduce to O(logN). The hash is a local peer knowledge of blockchain and timely state of messaging. This idea inherits from Dynamic Programming course of MIT 6.006, Prof. Erik Demaine. 

## Mutable item put
Each mutalble item A put/value will include a pointer to another public key, B, under the same salt. The B is the lastest change public key in A's knowledge. this helps the latest knowledge to distribute in DHT. 
This action will republish B under the same salt. 
## Immutable item put
Implement republish/reannouce of a random history blocks on the same chain(salt). 

# p2p messaging extention
p2p messaging via logN strategy.
each p2p message to A, its content includes a hash which is another public key recently messaging to A. 

in each TAU message either immtuable or mutable, there are hash point in content to make search N complexity to log(N). this is a type of DQ algorithm.
