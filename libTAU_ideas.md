libTAU 
------
To make devices communicatable to others, the state-of-art tech will require some server to faciliate them or require public IP address for p2p connections. 
TAU team thinks that IP protocol is really designed for server's global inter networking.
For personal smart phones or smaller devices, the public key is the real identifier, since it is too costly or risky to give each devices a long term IP address and it is difficult to achieve this in mobile network. 
Public key to public key indepandent communication is the true form of Internet for every entity, even as small as a lamp. 
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
  (blockchain ID1, [peer1, peer2, ...]), // the blockchains participated
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
libTAU design to solve a few key problems that famous projects such as libtorrent, IFPS and libP2P do not solve. 
1. What is the incentive for nodes to relay data for others? 
2. How to prevent Sybil and Eclipse attack to nodes? 
3. How to prevent routers to detect this protocol? 
4. How to avoid ISP blocks boostrap nodes and apply time attack? 
