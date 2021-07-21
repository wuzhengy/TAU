libTAU 
------
To make devices communicatable to others, the state-of-art tech will require some server to faciliate them or require public IP address for p2p connections. 
TAU team thinks that IP protocol is really designed for server's global inter networking.
For personal smart phones or smaller devices, the public key is the real identifier, since it is too costly or risky to give each devices a long term IP address and it is difficult to achieve this in mobile network. 
Public key to public key indepandent communication is the true form of Internet for every entity, even as small as a lamp. 
PCM - Phone Crypto Mining
TAU dev wants to:
* provide developers to use TAU communication for any devices supporting java runtime.

```
 use sqlite file based database for exchange data and control
 
 develper's app **<-SQLite Java->** sqlite **<-SQLite Java->** TAU protocol

* Java Interface
  * secrete seed
  * bandwidth
  * messages
  * blockchain
  * friends lists
* Initiation

  publicKey TAU.start( secret,  // the secret of key pairs, could be any random number
  [
  (blockchain ID1, [peer1, peer2, ...]), // the blockchains participated, default will join TAUcoin chain
  (blockchain ID2, [peer1, peer2, ...]),
  ...
  ],
  [ 
  (friend 1, flag), // friends list
  (friend 2, flag),
  ...
  ]
  )
```
------

Width Priority and Depth Priorty search in DHT communication
* We need to achieve fast communication and low data footprints at the same time, where regular DHT table are not customeized to do that since they are only transmitting meta data. 
* We will use short time-out to regulate depth search, the longer the short time-out, the higher priority is given to depth. Along with this, we will make replace bucket as memory for history failed invoke to prevent waste data. options are 0.5, 1, 1.5 or 2 seconds.   
* We use main loop frequence to perform width search, however faster frequency will cause high data consumption.  options are 200ms, 500ms, 1s.
* We use long time-out to control the wait time for data collection completeness, but long time-out has nothing to do with searching.  

------
libTAU design to solve a few key problems that famous projects such as libtorrent, IFPS and libP2P do not solve. 
1. What is the incentive for nodes to relay data for others? 
2. How to prevent Sybil and Eclipse attack to nodes? 
3. How to prevent routers to detect this protocol? 
4. How to avoid ISP blocks boostrap nodes and apply time attack? 
