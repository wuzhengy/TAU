# TAU - Phone Crypto Mining
TAU is a research project to create a phone blockchain mining and communication protocol called libTAU. Devices could communicate with each other through public keys. To demonstrate the libTAU, Phone Crypto Mining app is built to enable many communities to build independent blockchains. 
TAUcoin is one of the blockchain coins in the TAU blockchains community. TAUcoin public ledger will provide basic bootstrapping and time services for other communities. Community does not need taucoin to operate own communities. TAU software is free to use. 

## Applications built on Phone Crypto Mining network
We will experiment build a demonstration purpose uber service on the phone crypto mining network, that will cut the commission of connecting rider and drivers. 

### Key concepts in TAU Phone Mining.
Core UI experienses
- decentralized community with crypto-coins circulation
   * Create a chat/community/blockchain 
      * Give a name to new blockchain and its coin
      * Creation of a blockchain with 1 million coins on 5 minutes per block generation rate
   * Transactions on community blockchain
      * Forum Note
      * Wiring Transaction
      * Genesis
   * Instant Chat with people and group members. 
   * Rename/blacklist - users can rename or blacklist a peer
   * Support unlimited multiple devices sharing same address/account. 
- Dashboard: network data consumption, nodes status and devices status
--- 


## Data flow: StateDB, BlocksDB and DHT
  - statedb is the local database holding account power and balance
  - blockdb is the local database holding the blocks content that will be put and get through DHT
  - DHT provides key-value mutable data item, TAU has revised libtorrent DHT to keep mutable data structure only, and modified ID and encryption system to prevent sybil and eclipse attack. The basic idea is that routing table entry has to be honest in relaying on-chain data.
  - Data flow
    - `blockdb` ---> `tauDHT put` ---> `DHT`
    - `blockdb` <--- `tauDHT demand/get` <--- `DHT`
    - `statedb` <--> `blockdb`
  - p2p IP level direct communication on phones is not possible to implement, given firewalls/NAT and personal device security restrictions. DHT is the overlay p2p communication. When peer is off-line, the content is still available in DHT cache for exchange. 
## Concepts
- Version 1 default parameters: 
  - 5 minutes average to generate a block. It can be upgraded when network enhance. 
  - One block has one transaction. One block can include more transaction by encoding the hash of other transactions, which may be implmented in future version. 
- p2c, peer to consensus for reducing the sybil attack by requiring nodes all relaying data related onchain peers or friend list. If one node transmitting or relaying content not relating to onchain account, it will be droped from routing table. 
- Community ChainID := `community name`#`hash(GenesisMinerPubkey + timestamp)` 
  - Community will choose its own `community name`. 
  - example: TAUcoin ID is TAUcoin#hash 
- Public key is used as the crypto address: balance identifier under different chains; holds the power and perform mining. "Seed" generates privatekey and public key. 
- POT defines power as transaction volumn, the power annually increase according to Fibonacci sequence.
- Genesis block power: give one year power to genesis public key to make admin airdrop possible.  
- URL TAUchain:?bs=pk1&bs=pk2&dn=chainID // maybe 10 bootstrap publickeys provided

### Genesis 
```
// build genesis block
blockJSON  = { 
1. version;
2. timestamp; 
3. Domain name, IP address and port
4. blockNumber;
5. previousBlockRoot = null; // genesis is built from null.
6. basetarget = 0x21D0369D036978;
7. cummulative difficulty int64; 
8. generation signature;
11. msg; // {genesis state k-v, String chainID} // here is the only place chainID displayed to prevent genesis attack
12. `TsenderTAUpk`Noune = null
13. `Tsender`Balance = null;
14. `TminerTAUpk`Balance= null;
15. `Treceiver`Balance = null;
16. ED25519 public key
17. ED25519 signature
}

```

## Constants
* MutableRange:  288 blocks, range could touch geneis block as extreme. This is a concept of blocks, unrelated to time. 
* GenesisBasetarget:  0x21D0369D036978; simulated 1 million blocks with average 60 seconds.
* DefaultMinBlockTime:  60 seconds, this is fixed block time. do not let user choose as for now.
* DefaultMaxBlockTime:  540 seconds, when no body mining, you can generate blocks
* DefaultBlockTime: 300 seconds
