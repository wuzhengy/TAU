# TAU DHT extention to libtorrent/bitorrent mainline dht
## draft 0.01
For blockchain operation, we modify dht "put" item method on top of mainline dht, which is an form of kademilia dht. 
The purpose of the extention is that mainline dht does not implement the republish and RSS hash chain. We are using blockchain to replace the RSS hash chain. 
## Mutable item put
Each mutalble item put/value will include a pointer to another public key under the same salt, for lastest time change knowledge.
## Immutable item put
Implement republish/reannouce of a random history blocks on the same chain. 
