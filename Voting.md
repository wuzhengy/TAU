## POT Voting process: 
When a new peer coming on-line, the peer uses a voting process to chose the right fork to follow. Voting is collecting a specific block, which is the "immutable point block" from random block producers within the mutable range. Mutable range is the range of blocks from the current tip to a specific history block. 
   - voting position: The ImmutablePointBlock, the voting peers are the peers in the mutable range.
   - select the highest voted ImmutablePointBlock hash
      - 新节点上线，随机相信一个TAU URL里面bs节点时间戳在当前mutable range内的链，获得最新区块从mutable range到tip数据，设置链的（顶端- `MutableRange` ）为ImmutablePointBlock。如果tip/best blocknumber 小于 mutable range，use genesis block as voting position. 
        - URL TAUchain:?bs=`pk1`&bs=`pk2`&dn=`chainID`  // maybe 10 bs provided
      - 如果投票出的新ImmutablePointBlock，root在同一链上mutable range内，说明是链的正常发展
        - 继续发现最长链，在找到新的最长链的情况下，检查下自己以前已经上链的交易是否在新链上，不在新链上的放回交易池
      - 如果投出来的新ImmutablePointBlock, 这个block is within 1x - 3x out of mutable range，把自己当作新节点处理。检查下自己的历史交易是否在新链上，不在新链上的放回交易池。如果分叉点在3x mutable range之外，Alert the user of a potential attack。  
        - for a most difficult chain, folking within: 1x range, winner; 1x-3x, voting; 3x, alert.
