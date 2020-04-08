# Chapter 3 - Sharded Cluster

## Setting Up a Sharded Cluster
- Configuration files for each nodes
```
sudo mkdir -p /var/mongodb/db/csrs{1,2,3}
sudo chown vagrant:vagrant -R /var/mongodb/db
sudo cp mongod-csrs-*.conf /etc/mongod/

mongod -f /etc/mongod/mongod-csrs-1.conf
mongod -f /etc/mongod/mongod-csrs-2.conf
mongod -f /etc/mongod/mongod-csrs-3.conf
```

- Connect to one of the config servers, initiating the CSRS, and creating super user on CSRS:
```
mongo --port 26001
rs.initiate()

use admin
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles: [
    {role: "root", db: "admin"}
  ]
})
```
- Authenticating as the super user, to add the second and third node to the CSRS:
```
db.auth("m103-admin", "m103-pass")
rs.add("192.168.103.100:26002")
rs.add("192.168.103.100:26003")
```

- Create the mongos config (mongos.conf), connect to mongos, and check sharding status:
```
sudo cp mongos.conf /etc/mongod/mongos.conf
mongos -f /etc/mongod/mongos.conf
mongo --port 26000 --username m103-admin --password m103-pass --authenticationDatabase admin
sh.status()
```

- Updating nodes config file to change the clusterRole `shardsvr`
``` 
sudo cp node*.conf /etc/mongod/
mongod -f /etc/mongod/node1.conf
mongod -f /etc/mongod/node2.conf
mongod -f /etc/mongod/node3.conf
```

- Connecting directly to secondary node(s) (note that if an election has taken place in your replica set, the specified node may have become primary), and Shutting down node:
```
mongo --port 27012 -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
use admin
db.shutdownServer()
```

- Stepping down current primary:
```
rs.stepDown()
```

- Adding new shard to cluster from mongos, and check with `sh.status()` - [Result](result/shStatus.txt)
```
mongo --port 26000 --username m103-admin --password m103-pass --authenticationDatabase admin
sh.addShard("replExample/m103:27012")
sh.status()
```

## Lab - Configure a Sharded Cluster
- Bring up the config server replica set (CSRS)
```
rm -rf /var/mongodb/db/csrs{1,2,3}
sudo mkdir -p /var/mongodb/db/csrs{1,2,3}
sudo chown vagrant:vagrant /var/mongodb/db/csrs{1,2,3}
sudo cp mongod-csrs-*.conf /etc/mongod/

mongod -f /etc/mongod/mongod-csrs-1.conf
mongod -f /etc/mongod/mongod-csrs-2.conf
mongod -f /etc/mongod/mongod-csrs-3.conf

mongo --port 26001
rs.initiate()

use admin
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles: [
    {role: "root", db: "admin"}
  ]
})

db.auth("m103-admin", "m103-pass")
rs.add("192.168.103.100:26002")
rs.add("192.168.103.100:26003")
rs.isMaster()
```

- Bring up the mongos, and try to authenticate to mongos immediately as `m103-admin` user:
```
sudo cp mongos.conf /etc/mongod/mongos.conf
mongos -f /etc/mongod/mongos.conf
mongo --port 26000 --username m103-admin --password m103-pass --authenticationDatabase admin
```

- Reconfigure m103-repl, add sharding > clusterRole: shardsvr on configs files:
```
sharding:
  clusterRole: shardsvr 

sudo cp mongod-csrs-*.conf /etc/mongod/
```

- Connecting directly to secondary node(s) and Shutting down nodes, update the config files and turn up all secondary nodes.
```
mongo --port 27002 -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
use admin
db.shutdownServer()
mongod -f /etc/mongod/node2.conf

mongo --port 27003 -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
use admin
db.shutdownServer()
exit
mongod -f /etc/mongod/node3.conf
```

- Stepping down current primary:
```
mongo --port 27001 -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
rs.stepDown()
use admin
db.shutdownServer()

mongod -f /etc/mongod/node1.conf
```

- Add m103-repl as the first shard, Once m103-repl has sharding enabled, you can add it as the primary shard with, 
Check the output of `sh.status()` to make sure it's included as a shard.
```
mongo --port 26000 --username m103-admin --password m103-pass --authenticationDatabase admin
sh.addShard("m103-repl/192.168.103.100:27011")
sh.status()
```