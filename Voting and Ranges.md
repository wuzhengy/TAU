## Ranges
When a fork of blockchain is detected, depending on where the fork point locates, TAU uses ranges to decide miner actions. 

* mutable range:  ... <= 1x
   * Mutable range is the range of blocks from the current blockchain frontier, tip, to a specific defined history block, which is called mutable point. Mutable range includes the mutable point block. When fork happens in this range, it is natural longest chain discovery process.  
* voting range:   1x  < ...  <= 3x 
   * Voting range is the blocks between 1 x mutable range to 3x mutable range, the length of this range is twice as mutable range. When fork happens in this range, the miner will go for voting to choose right fork. 
* warning range:  3X  < ...
   * the blocks prior to voting range. When fork happens, the app will warn user a potential attack and allow user to manually switch to that chain. 
```
genesis ... warning range ... 3x ... voting range ... 1x ... mutable range ... tip
```
## POT Voting process
Voting process triggering: 
* When a new peer coming on-line, the peer uses a voting process to chose the right fork to follow. 
* When a fork with higher difficulty falls in `voting range`.  <br><br>
Voting has two phase: 
* Prepare data: collecting a certain block such as "immutable point block" from random block producers in the `mutable range`.
   - voting position: the ImmutablePointBlock, if the current tip block number less than mutable range，it means the blockchain is very young. It will use genesis block as voting position.
   - participating peers: peers in the  range.
   - peers selection Strategy
      * number of peers:  log(n)
      * stake weighted random selection. the higher stake, the hire chance to get selected for voting.
* Compute the data 
   - select the highest voted ImmutablePointBlock
```
      - 新节点上线，随机相信一个TAU URL里面bs节点, 获得最新区块到tip, start voting.
        - URL TAUchain:?bs=`pk1`&bs=`pk2`&dn=`chainID`  // maybe 10 bootstrap publickeys provided
      - 如果投票出的新ImmutablePointBlock，root在同一链上mutable range内，说明是链的正常发展
        - 继续发现最长链，在找到新的最长链的情况下，检查下自己以前已经上链的交易是否在新链上，不在新链上的放回交易池
      - 如果投出来的新ImmutablePointBlock within voting range, go to voting。检查下自己的历史交易是否在新链上，不在新链上的放回交易池。
      - 如果分叉点 within warning range，alert the user of a potential attack。
```
 

