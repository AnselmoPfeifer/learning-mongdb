# Chapter 2: Replication

## Setting Up a Replica Set
```
# Configuring
sudo mkdir -p /var/mongodb/pki/ /var/mongodb/db/node{1,2,3} /etc/mongod/
sudo chown vagrant:vagrant -R /var/mongodb/pki/ /var/mongodb/db
openssl rand -base64 741 > /var/mongodb/pki/keyfile
chmod 400 /var/mongodb/pki/keyfile

# Starting
sudo cp node*.conf /etc/mongod/
mongod -f /etc/mongod/node1.conf
mongod -f /etc/mongod/node2.conf
mongod -f /etc/mongod/node3.conf

# Checking
ss -tlnp
ps -aef | grep [m]ongod
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

## Lab - Initiate a Replica Set Locally
```
PRIMARY: 
    - mongod-repl-1.conf
    - dbPath: /var/mongodb/db/1
    - port: 27001
    - logPath: /var/mongodb/db/mongod1.log
    - keyFile: /var/mongodb/pki/m103-keyfile
    - bindIP: localhost,192.168.103.100
SECONDARY: 
    - mongod-repl-2.conf
    - dbPath: /var/mongodb/db/2
    - port: 27002
    - logPath: /var/mongodb/db/mongod1.log
    - keyFile: /var/mongodb/pki/m103-keyfile
    - bindIP: localhost,192.168.103.100
SECONDARY: 
    - mongod-repl-3.conf
    - dbPath: /var/mongodb/db/3
    - port: 27003
    - logPath: /var/mongodb/db/mongod1.log
    - keyFile: /var/mongodb/pki/m103-keyfile
    - bindIP: localhost,192.168.103.100
``` 
```
sudo rm -rf /var/mongodb/db/{1,2,3}
sudo mkdir -p /var/mongodb/db/{1,2,3} 
sudo chown vagrant:vagrant -R /var/mongodb/db

sudo cp mongod-repl-*.conf /etc/mongod/
mongod -f /etc/mongod/mongod-repl-1.conf
mongod -f /etc/mongod/mongod-repl-2.conf
mongod -f /etc/mongod/mongod-repl-3.conf
mongo --port 27001
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

mongo --host "m103-repl/192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
rs.status()
rs.add("192.168.103.100:27002")
rs.add("192.168.103.100:27003")
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
```
use m103
db.messages.updateMany( {}, { $set: { author: 'norberto' } } )
use local
db.oplog.rs.find( { "ns": "m103.messages" } ).sort( { $natural: -1 } )
```
* Remember, even though you can write data to the local db, you should not.

## Reconfiguring a Running Replica Set:
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

## Lab - Remove and Re-Add a Node:
- The configuration of the nodes should not change - the hostname m103 is already bound to the IP address 192.168.103.100
- The nodes should still run on ports 27001, 27002, 27003
- The name of your replica set should still be m103-repl
```
sudo rm -rf /var/mongodb/db/{1,2,3}
sudo mkdir -p /var/mongodb/db/{1,2,3} 
sudo chown vagrant:vagrant -R /var/mongodb/db

mongod -f /etc/mongod/mongod-repl-1.conf
mongod -f /etc/mongod/mongod-repl-2.conf
mongod -f /etc/mongod/mongod-repl-3.conf
mongo --port 27001
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

mongo --host "m103-repl/m103:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
rs.add("m103:27002")
rs.add("m103:27003")

rs.remove("192.168.103.100:27001")
rs.add("m103:27001")
rs.status()
```