# Chapter 2 - Replication

## Setting Up a Replica Set
```
sudo mkdir -p /var/mongodb/pki/ /var/mongodb/db/node{1,2,3} /etc/mongod/
sudo chown vagrant:vagrant -R /var/mongodb/pki/ /var/mongodb/db
openssl rand -base64 741 > /var/mongodb/pki/keyfile
chmod 400 /var/mongodb/pki/keyfile

sudo cp node*.conf /etc/mongod/
mongod -f /etc/mongod/node1.conf
mongod -f /etc/mongod/node2.conf
mongod -f /etc/mongod/node3.conf
```

- Connecting to node1:
```
mongo --port 27011
```

- Initiating the replica set:
```
rs.initiate()
```

- Creating a user:
``` 
use admin
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles: [
    {role: "root", db: "admin"}
  ]
})
```

- Connecting to the entire replica set and get replica status: ([Result](result/rsStatus1.json))
```
mongo --host "replExample/192.168.103.100:27011" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
rs.status()
```

- Adding other members to replica set and get replica status: ([Result](result/rsStatus2.json))
```
rs.add("192.168.103.100:27012")
rs.add("192.168.103.100:27013")
```

- Getting an overview of the replica set topology: ([Result](result/rsIsMaster.json))
```
rs.isMaster()
```

- Stepping down the current primary and Checking replica set overview after election: ([Result](result/rsIsMaster2.json))
```
rs.stepDown()
rs.isMaster()
```

## Replication Commands:
- Report health on replica set nodes
```
rs.status()
```

- Describe a node's role in the replica set, a shorter output than previous command
```
rs.isMaster()
```

- Section of the `db.serverStatus()`output and similar to the output of `rs.isMaster()`
```
db.serverStatus()['repl']
```

- Only returns oplog data relative to current node
```
rs.printReplicationInfo()
```

## Local DB: Part 1:
- Querying the oplog after connected to a replica set,
```
use local
db.oplog.rs.find()
```

- Storing oplog stats as a variable called stats:
- Verifying that this collection is capped,
- Getting current size of the oplog,
- Getting size limit of the oplog:
```
var stats = db.oplog.rs.stats()
stats.capped
stats.size
stats.maxSize
```

- Getting current oplog data (including first and last event times, and configured oplog size):
```
rs.printReplicationInfo()
```

## Local DB: Part 2:
- Create new namespace m103.messages:
```
use m103
db.createCollection('messages')
```

- Query the oplog, filtering out the heartbeats ("periodic noop") and only returning the latest entry:
```
use local
db.oplog.rs.find( { "o.msg": { $ne: "periodic noop" } } ).sort( { $natural: -1 } ).limit(1).pretty()
```

- Inserting 100 different documents:
```
use m103
for ( i=0; i< 100; i++) { db.messages.insert( { 'msg': 'not yet', _id: i } ) }
db.messages.count()
```

- Querying the oplog to find all operations related to m103.messages:
```
use local
db.oplog.rs.find({"ns": "m103.messages"}).sort({$natural: -1})
```

- Illustrating that one update statement may generate many entries in the oplog:
Remember, even though you can write data to the local db, you should not.
```
use m103
db.messages.updateMany( {}, { $set: { author: 'norberto' } } )
use local
db.oplog.rs.find( { "ns": "m103.messages" } ).sort( { $natural: -1 } )
```

## Reconfiguring the Replica Set:
- Starting up mongod processes for our fourth node and arbiter:
```
sudo mkdir -p /var/mongodb/db/{node4,arbiter}
sudo chown vagrant:vagrant -R /var/mongodb/db
sudo cp node4.conf arbiter.conf /etc/mongod/
mongod -f /etc/mongod/node4.conf
mongod -f /etc/mongod/arbiter.conf
```

- From the Mongo shell of the replica set, adding the new secondary and the new arbiter:
```
rs.add("m103:27014")
rs.addArb("m103:28000")
```

- Checking replica set makeup after adding two new nodes:
```
rs.isMaster()
```

- Removing the arbiter from our replica set:
```
rs.remove("m103:28000")
```

- Assigning the current configuration to a shell variable we can edit, in order to reconfigure the replica set:
```
cfg = rs.conf()
```

- Editing our new variable cfg to change topology - specifically, by modifying cfg.members:
```
cfg.members[3].votes = 0
cfg.members[3].hidden = true
cfg.members[3].priority = 0
```

- Updating our replica set to use the new configuration cfg:
```
rs.reconfig(cfg)
```

## Reads and Writes on a Replica Set
- Starting and connecting to replSetName: replExample
```
sudo mkdir -p /var/mongodb/db/node{1,2,3} 
sudo chown vagrant:vagrant -R /var/mongodb/db

mongod -f /etc/mongod/node1.conf
mongod -f /etc/mongod/node2.conf
mongod -f /etc/mongod/node3.conf

mongo --port 27011
use admin
rs.initiate()
rs.status()

use admin
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles: [
    {role: "root", db: "admin"}
  ]
})
mongo --host "replExample/m103:27011" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
rs.add("m103:27012")
rs.add("m103:27013")
```

- Checking replica set topology:
```
rs.isMaster()
```

- Inserting one document into a new collection:
```
use newDB
db.new_collection.insert( { "student": "Matt Javaly", "grade": "A+" } )
```

- Connecting directly to a secondary node (this node may not be a secondary in your replica set!):
```
mongo --host "m103:27012" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
```

- Attempting to execute a read command on a secondary node (this should fail):
```
show dbs
```

- Enabling read commands on a secondary node:
```
rs.slaveOk()
```

- Reading from a secondary node:
```
use newDB
db.new_collection.find()
```

- Attempting to write data directly to a secondary node (this should fail, because we cannot write data directly to a secondary):
```
db.new_collection.insert( { "student": "Norberto Leite", "grade": "B+" } )
```

- Shutting down the server (on both secondary nodes)
```
use admin
db.shutdownServer()
```

- Connecting directly to the last healthy node in our set:
```
mongo --host "m103:27011" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
```

- Verifying that the last node stepped down to become a secondary when a majority of nodes in the set were not available:
```
rs.isMaster()
```

## Failover and Elections
- Storing replica set configuration as a variable cfg:
```
mongo --host "replExample/m103:27011" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
cfg = rs.conf()
```

- Setting the priority of a node to 0, so it cannot become primary (making the node "passive"):
```
cfg.members[2].priority = 0
```

- Updating our replica set to use the new configuration cfg:
```
rs.reconfig(cfg)
```

- Checking the new topology of our set:
```
rs.isMaster()
```

- Forcing an election in this replica set (although in this case, we rigged the election so only one node could become primary):
```
rs.stepDown()
```

- Checking the topology of our set after the election:
```
rs.isMaster()
```
