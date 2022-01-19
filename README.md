TAU App - Create crypto coins and trade community for free.
TAU helps you to create P2P crypto trade community independant from central servers. You can issue genuine blockchain coins free from third party transaction fee. TAU do not require users to buy any TAUcoins for enjoying full functions, so that community leaders can airdrop coins to participants freely.

We want to help individuals from everywhere to build crypto community and increase value of community coins; therefore TAU makes the full blockchain node system working on smart phones with low data consumption, which is important for developing regions such as Africa.

About the core technologies
Perishable Proof of Transactions(PPOT) Blockchain Consensus Tailored for individual phones
TAU discovered the PPOT blockchain concensus, the more transactions history made, the higher mining power a node has. Without intensive puzzel solving competition such as in bitcoin, PPOT is a light weight consensus.

The perishable ledger will forget any block and state 6 months older, to make full blockchain size under 50Mbytes. Unlike immutable blockchains such as ethereum and bitcoin designed for long term financial contracts, PPOT is light for day to day commuinciation oriented apps. PPOT blocks are designed to register user, essential wiring and high capacity communication.

PPOT enables a smart phone to hold and mine 100+ full blockchains. Collectively, the parallel blockchains bring to the ecosystem unlimted transaction capacity. TAU Cambridge provides an opensource C++ reference implementation of PPOT parallel mining and messaging on github.

Internet Protocol 2 - IP2
Internet Protocol 2(IP2) enables user to choose self-generated 256-bits "public key" as address, while classical Internet Protocol(IP1) appoints hierarchically and often dynamically addresses. IP1 has caused "uncertain device reachability" in type of addresses and connections, especially when same device is moving through different networks or locating in unknown firewall filtering rules; therefore additional proxy or name server infrastructure burden is needed for each application.

IP2 is designed for "public key direct to public key" overlay communication. Thanks to the innovation of XOR distance to form up the local capture swarm network, the nodes are incentivated to share "public key to IP address" naming and relaying for restricted nodes, disregard of application types.

On top of IP2, traditional TCP or UDP type of services could be rebuilt without worrying about static/dynamic, v4/v6, local/public, wifi/cellular types of IP1 addresses, as well as their dynamic address translation and filtering restrictions. This potentially reduces cost of operating IOT devices.

The technology stack includes "distributed routing and capture vectors", ED25519 assymetric encryption for premission-less and colision-free unique addressing, and pattern randomized transmission on UDP. TAU Cambridge provides an opensource C++ reference implementation libIP2 on github.

PPOT blockchain consensus and messaging are applications of IP2.


### Key concepts in TAU Phone Mining.
Core UI experienses
- decentralized community with crypto-coins circulation
   * Create a chat/community/blockchain 
      * Give a name to new blockchain and its coin
      * Creation of a blockchain with 1 million coins on 5 minutes per block generation rate
   * Transactions on community blockchain：社区主界面包含节点打包交易出块，发送交易，消息，和选择器 (Tip Blocks, Wiring Tx, Notes TX and Messages)
      * Onchain Note TX，Mark down serverless web. 每个节点的note可以视为一个连续的“markdown file", GFM mark down style.
      * Offchain message, regular text
      * Wiring Transaction
      * Tip Blocks
   * Text messaging with people and group members. 
   * Rename/blacklist - users can rename or blacklist a peer
   * Support unlimited multiple devices sharing same address/account. 
- Dashboard
  * app system: network data consumption, nodes status and devices status
  * chain status: 链的长度，难度，节点数，节点出块数量，总流通币量，前十名持币地址和币量，前10名power地址和pot，共识点投票前三名的区块号和哈希最后2个bytes的16进制,tip前三名区块号和哈希，当前分叉点区块号和哈希，全部节点列表可以点击添加朋友； （ Chain Length, Difficulty, Total Peers, Peers & Blocks, Total coins, Top 10 Peers and Coins, Top 10 Peers and Power, Top 3 Consensus Blocknumber and Hash, Top 3 Tip Blocknumber and Hash, Current Fork Blocknumber and Hash, Full Peers List) 
  * friend status: last seen, last communication, common chains

## Concepts
- Version 1 default parameters: 
  - 5 minutes average to generate a block. It can be upgraded when network enhance. 
  - One block has one transaction. One block can include more transaction by encoding the hash of other transactions, which may be implmented in future version. 
- Community ChainID := `hash(GenesisMinerPubkey + timestamp)定长``community name变长`
  - Community will choose its own `community name`. 
  - example: TAUcoin ID is hash#TAUcoin 
- Public key is used as the crypto address: balance identifier under different chains; holds the power and perform mining. "Seed" generates privatekey and public key. 
- POT defines power as transaction volumn, the power annually increase according to Fibonacci sequence.
- Genesis block power: give one year power to genesis public key to make admin airdrop possible.  
- URL TAUchain:?bs=pk1&bs=pk2&dn=chainID // maybe 10 bootstrap publickeys provided

### Genesis 
```
// build genesis block
blockJSON  = { 
1. version;
2. chain id; 32 bytes `hash(GenesisMinerPubkey + timestamp)定长``community name变长`
3. timestamp; 
5. blockNumber; // for each 105,120 blocks, we will increase power unit follow fibonacci series - 1,1,2,3,...
6. previousBlockRoot = null; // genesis is built from null.
7. basetarget = 0x21D0369D036978;
8. cummulative difficulty int64; 
9. generation signature;
10. payload, starting with domain and ip info;
11. `TsenderTAUpk`Nonce = 1;
12. `Tsender`Balance = 1,000,000;
13. `TminerTAUpk`Balance= 1,000,000; // in the genesis, Tsender = Tminer = Treceiver
14. `TminerTAUpk`Nonce= 1;
15. `Treceiver`Balance = 1,000,000;
16. `Treceiver`Nonce= 1;
17. ED25519 public key
18. ED25519 signature
}

```

## Constants
* MutableRange:  288 blocks, range could touch geneis block as extreme. This is a concept of blocks, unrelated to time. 
* GenesisBasetarget:  0x21D0369D036978; simulated 1 million blocks with average 60 seconds.
* DefaultMinBlockTime:  60 seconds, this is fixed block time. do not let user choose as for now.
* DefaultMaxBlockTime:  540 seconds, when no body mining, you can generate blocks
* DefaultBlockTime: 300 seconds
