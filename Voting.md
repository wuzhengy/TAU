## POT Voting process: 
When a new peer coming on-line, the peer uses a voting process to chose the right fork to follow. Voting is collecting the "immutable point block" from random block producers in the mutable range. Mutable range is the range of blocks from the current blocchain frontier, tip, to a specific defined history block. 
   - voting position: the ImmutablePointBlock, 如果 tip blocknumber 小于 mutable range，use genesis block as voting position.
   - voting peers: mining peers in the mutable range.
   - select the highest voted ImmutablePointBlock
      - 新节点上线，随机相信一个TAU URL里面bs节点, 获得最新区块到tip, start voting.
        - URL TAUchain:?bs=`pk1`&bs=`pk2`&dn=`chainID`  // maybe 10 bootstrap publickeys provided
      - 如果投票出的新ImmutablePointBlock，root在同一链上mutable range内，说明是链的正常发展
        - 继续发现最长链，在找到新的最长链的情况下，检查下自己以前已经上链的交易是否在新链上，不在新链上的放回交易池
      - 如果投出来的新ImmutablePointBlock, 这个block is within 1x - 3x out of mutable range，把自己当作新节点处理。检查下自己的历史交易是否在新链上，不在新链上的放回交易池。如果分叉点在3x mutable range之外，Alert the user of a potential attack。
