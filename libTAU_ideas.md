### libTAU
* C++ library
* SWIG interface for Java, especially android platform
* Shell interfece for Linux command line

### Phone Crypto Mining basic process
* discover: 1. the right immutable point and 2. highest block
    * find own immutable point from voting, happen in each block, process will always do in memory voting in the recent 100 responses, pick majority of 100 responses to switch to, each node ID provide 1 vote include own ID, in tie situation, own ID has priority.
      * Seeking immutable point from mutable blockchain signal (know peers + chain ID),(block hash, ip+port)
    * after decide new immutable point, if the new block immutable point blocknumber post to immutable point, highest difficulty dorminates; if new block immutable point prior to current voted immutable point, then ignore, this fixed to delegate people to make decision long time problem.
        
* in summary, each round immutable point is decided by voting, but tip block is decided by crypto difficulty and data integrity
