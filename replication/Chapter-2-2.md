# Chapter 2: Replication

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

Failover and Elections

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