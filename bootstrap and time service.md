TAU tech stack has several layers of bootstrap.
* IP2 level bootstrap
* blockchain initial data bootstrap
TAU will use blockchain content as bootstrapping base. 
When software release, it will include a portion of TAUcoin main ledger with timestamp and IP addresses for bootstrapping. The nodes start to discover the space, then more bootstrap information will be added into blockchain state db. 
It is not necessary to use TAUcoin chain, any community chain with enough IP address can server as bootstrap. 
The boostrap inforamtion also serve as time adjustment to prevent time attack to fool a node in different time to prevent communication. 

As a miner issue, the time server as a central service is not entirely trustworthy. So constantly polling community time as a watch dog to time server is used as well. 
