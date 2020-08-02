# TAU - Decentralized communication with high-scaling blockchain economy.
### Design notes recording key concepts in TAU
Core UI experienses
- decentralized community social chat with crypto-coins circulation
   * Create a Chat 
      * Give a name to new blockchain and its coin
      * Creation of a blockchain with 10 million coins on 5 minutes per block generation rate
   * Transactions on community blockchain - DHT mutable key: hash(public key + `chainID#blk`)
      * Regular forum note and comments
      * New community annoucement
      * Wiring Transaction 
   * Instant Chat Messages - DHT mutable key: public key + `chainID#msg`
      * Peers can pub chat messages and hope other peers to find it through subscription
   * Decentralized blacklist - easy to blacklist an address or exit a community
- Dashboard:  Wifi Data * Kb/s; Telecom Date * Kb/s; DHT nodesinfo
- TAUcoin chain prebuilt: a place to publish annoucements.
--- 
## Persistence
1.  Chains  map[ChainID] config; 
2.  CurrentBlockRoot    map[ChainID]; // the map holding the recent blocks
3.  MutableRange    map[ChainID]uInt  // blocks in the mutable range is subject to change any time 
4.  Peers       map[ChainID]map[TAUpk]config; // for chains, when discovery new TAU peers, adding here with config info.
5.  SelfTxsQueue         map[ChainID]map[TxHASH]config // account's own transactions history
6.  ImmutablePointBlock    map[ChainID] uInt 

## Data flow: StateDB, BlocksDB and DHT
  - statedb is the local database holding account power and balance
  - blockdb is the local database holding the blocks content that will be put and get through DHT
  - DHT is the network key-value cache
  - Data flow
    - `blockdb` ---> `tauDHT put` ---> `DHT`
    - `blockdb` <--- `tauDHT get` <--- `DHT`
    - `statedb` <--> `blockdb`
  - myth: p2p IP level communication is not possible to implement, given firewalls and personal device security restrictions. DHT is the overlay p2p communication. When peer is not off-line, the content is still available in DHT cache for exchange. 
## Key Concepts
- Version 1 default parameters: 
  - 5 minutes average to generate a block. It can be upgraded when network infrastructure upgrading. 
  - One block has one transaction, which fits both DHT easy lookup and account state update. One block can include more transaction by encoding the hash of other transactions, which may be implmented in future version. 
- blockchain and hash-chain: blockchain reflects to community consensus, hash-chain stores personal instant chat messages on one blockchain. The format looks like, community ID: Shanghai#600#hash(pK+timestamp); hash chain salt: Shanghai#600#hash(pk+timestamp)`#msg`. All the data in TAU is an entry to a chain either blockchain or hash chain. 
- p2c, peer to consensus for reducing the DHT sybil attack by relying on membership information on chain. 
- channels: BLK - blockchain; TX - transaction candidates; MSG - instant chat
- Community ChainID := `community name`#`optional block time interval in seconds`#`hash(GenesisMInerPubkey + timestamp)` 
  - Community chain will choose its own name. 
  - Coin volumn is 10 million
  - Default block time is 300 seconds
  - example: TAUcoin ID is TAUcoin##hash; community ID: Shanghai#600#hash, which is a chain name Shanghai with 10 million coins and 600 seconds block time. 
  - ChainID#channel is defined as libtorrent salt
- Public key is used as the crypto address: balance identifier under different chains; holds the power and perform mining. "Seed" generates privatekey and public key. In new TAU, we use "seed" to import and export the account identifier. 
- POT defines power as square root of the nounce.
- genesis block power: give one year power to genesis public key to make admin airdrop possible. 
- Provide opitional miner manual approval on transactions, expecially the negative value and problem content. 
- One secrete key per device, not recommend to copy secrete key between devices. 
- member with power/balance(0/0) in a chain = read only. 
- bootstrap ports 6881 is considered level ONE cache boostrap, software should remember these ips for future bootstrap and software release. 
- voting process: please check seperate doc voting.md

### Genesis 
```
// build genesis block
blockJSON  = { 
1. version;
2. timestamp; 
3. BlockNumber:=0;
4. PreviousBlockRoot = null; // genesis is built from null.
5. basetarget = 0x21D0369D036978;
6. cummulative difficulty int64; 
7. generation signature;
9. msg; // {genesis state k-v} 
10. `TsenderTAUpk`Noune = null
11. `Tsender`Balance = null;
12. `TminerTAUpk`Balance= null;
13. `Treceiver`Balance = null;
14. ED25519 public key as TAUaddress
15. ED25519 signature
}

```
## Normal Block content
```
blockJSON  = { 
1. version;               // long
2. timestamp;             // long
3. BlockNumber;           // long
4. PreviousBlockHash; // for verification                   // list[long]
5. ImmutablePointBlockHash; // for voting, simple skip list // list[long]
6. basetarget;                // long
7. cummulative difficulty;    // long
8. generation signature;      // list[long]
9. msg; // transaction content with txType, content. // List[long/java compact string/list[long]]
10. `TsenderTAUpk`Noune       // long
11. `Tsender`Balance;         // long
12. `TminerTAUpk`Balance;     // long 
13. `Treceiver`Balance;       // long
14. ED25519 public key as TAUaddress  // list[long]
15. ED25519 signature                 // list[long]
}
```
list[long] replaces byte[] for arbitory data not require utf-8 or unicode encoding

## Constants
* MutableRange:  864 blocks, 3 days, use block as unit since no censensus
* WarningRange: 3 x MutableRange
* WakeUpTime: sleeping mode wake up random range 10 minutes
* GenesisCoins: default coins 10,000,000. 
* GenesisBasetarget:  0x21D0369D036978; simulated 1 million blocks with average 60 seconds.
* DefaultMinBlockTime:  60 seconds, this is fixed block time. do not let user choose as for now.
* DefaultMaxBlockTime:  540 seconds, when no body mining, you have to generate blocks.
* DefaultBlockTime: 300 seconds
* TXtimeout: 12 Hours

---
## Mining Process sketch: Votings, chain choice and block generation.   
```
1. get chain ID. 

3. Choose a Peer from  TAUpeers[`ChainID`]
   If chainID+Peer is requested within DefaultBlockTime, go to (9) // do not revisit same peer within default block time
      
4. DHT_get(`hTAUpk+chainID`); get `ChainID` tip block from DHT;
   if not_found go to (9);
   TAUpeers[ChainID][Peer].update(timestamp) // for verifying the revisit time

6. if received block shows a valid higher difficulty than current difficulty  {
    if the fork happens prior to the ImmutablePointBlock and within the WarningRange, 
    go to (7) for voting. 
    if the fork happens piror to the WarningRange, throw warning to user, go to (9)
    verify this chain's transactions from the ImmutablePointBlock;
    if verification successful and populate new states, new longest block identified. 
    }
   go to (9) 
   
7. collecting all peers from mutable range for voting. 
   DHT_get(ImmutablePointBlockHash);

8. new ImmutablePointBlock voted.
   goto (1)

9. if TAUpk not qualifies POT requirment; go to (1) 
   generate new block
   put into DHT
   populate leveldb database. 

10. go to step (1)

```
---
# System config
  - Wifi Only: ON, when turn off, it will ask for time to allow telecom data operating
  - Charging ON: wake lock ON. 
  - Charging OFF: wake lock OFF. random wake up between 1..WakeUpTime to restart service
  - Internet OFF: wake lock OFF. random wake up between 1..WakeUpTime to restart service
  - Server mode: default OFF; when turn ON, it will turn on wake lock, when phone reboot, tau will auto start. 
# Database: leveldb andriod
# Muliple platform support on android, ios, pc, chromeOS, macOS, linux ...
  Since `secrete key` is supposed to be on only on device for operation, we focus on use android as core platform. Other platform will use browser to connect andoid TAU app. As long as other devices can access the android phone via IP network, they can operate a TAU node. <br>
  Linux dht command line interfact will be provided to interact with TAU. <br>
# To do and other notes
- resource management process
- TAU dev might provide centralized DHT search engine: centralized engine to find new community and links.
