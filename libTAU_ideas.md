### libTAU
* C++ library
* SWIG interface for Java, especially android platform
* Shell interfece for Linux command line

### Phone Crypto Mining basic process
* Adopt the right immutable point and highest block
    * find own immutable point from voting, happen in each block, process will always do in memory voting in the recent 100 responses, pick majority of 100 unique responses to switch to, own node ID provide 1 vote.
    * after decide new immutable point, if the new block immutable point blocknumber post to immutable point, highest difficulty dorminates
         * Seeking immutable point from mutable blockchain signal (know peers + chain ID),(block hash, ip+port)
* immutable point is decided by voting, but tip block is decided by crypto difficulty and data integrity
