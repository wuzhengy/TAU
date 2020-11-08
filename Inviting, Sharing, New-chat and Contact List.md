## Connected Peer List
Each peer owns a contact list of the peers under certain chains. For the peers in the contact list, the status will be:
- Last seen: date
- Last direct communication: date
- Exchange QR code to add friends to contact list. Add members to community, has to add contacts first. 
- only after exchange, member show in contact list. contact list is on personal chain. 

## Inviting(airdrop) and sharing(non-airdrop)
Peer can `invite/airdrop` another peer to join a community(chain) with airdrop to: 
  * contact list members
    - send coins
    - inform members within app
    - share through 3rd party communication tool (optional)
  * arbitrary public key input
    - send coins
    - share through 3rd party communication tool (optional) <br><br>

Peer can `share` the chain URL to another peer without airdrop to:  
  * contact list member
  * any one through 3rd party communication tool
  
## Create a community chat from home(community) view
Create a chat will require user: 
1. ask user to input chain/chat name
2. invite members with airdrop: contact list and arbitrary key
3. share the URL(optional) <br><br>
`chain is the only form of communication for two people chatting, multiple people group chatting, community blockchain.`

## Create a chat from Contact List view
1. ask user to input chat name, or use both members first letter of name(or public key) to create a chain
2. automatic airdrop coins to a member
3. share the URL (optional)

## Create a chat directly use arbitrary public key
1. ask user to input the target public key and chat name, or the chat name could be two first letter of name(or public key) to create a chain
2. automatic airdrop coins to the public key
3. share the URL (optional)

# New peer joining
* A new peer join the group, it will take a while to collect blockchain data. During the mean time, it will rely on other peer's referral link to process the mining, messaging and tx pool. New peer will trust the referral linked public key for the content. 
* When new peer join, app will allow to send messages, however, until it is confirmed onchain by others or peer-self, its message will then be loaded to other app.
* The App should process a phase called "checking own status on chain" to confirm whether a peer can send messages or blk/tx pool candidates. 
