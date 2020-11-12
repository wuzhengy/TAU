TAU will use torrent network to bootstrap the swarm. We basically merge ourself into torrent network DHT layer without participating seeding and downloading. 
It is estimated around 100m nodes existing in torrent. This is a very robust and big network for bootstrap. 
Our curent observation, in one minute, our app is able to discover about 10,000 torrent nodes with dht capability. 
We will maitain some dynamic DNS records to point to some dev machines IP just for contributing to the torrent network. 
