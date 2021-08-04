### libTAU
* C++ library
* SWIG interface for Java, especially android platform
* Shell interfece for Linux command line

### Phone Crypto Mining basic process
* discover voting process: 1. the consensused immutable point; 2. tip block; 3. forking point
    * find own immutable point from voting: in each blockchain life signal, peers send immutable point hash, immutale point block number, levenstein array from the block to tip, block payload hash and endpoint. The process will do in memory voting for the recent 288 network responses, to choose the unique majority of 288 responses as immutable point. Each peer node ID provides 1 vote include own, in tie situation, random select one. Immutable point only move according to voting, so it could be moved slower than the real clock.
      * Seeking immutable point from mutable blockchain signal (know peers + chain ID),(peers send immutable point hash, immutale point block number, levenstein array from the block to tip, block payload hash and endpoint). Remove blockchain immutable point hash field. 
    * after decide new immutable point, if the new block levenstein array fork out post to immutable point, then highest difficulty dorminates the selection; if prior to current voted immutable point, then ignore such block payload. This plan fixed to make people making decision for fork selection.
* merge: this process allow forked subcommunity to merge back to mainstream, especially in the network isolation situation. 
        
* in summary, each round immutable point is decided by voting, but tip block is decided by forking position, crypto difficulty and data integrity

### State range
Each account balance and nounce will have a valid range starting from the time of change to future 1 year. This range info need to be in database, when switching forks. Therefore, tau account can not use MPT trie, but to use sql structure for easier deletion and rolling back. Each state change for both balance and valid range will be recorded in database. 
