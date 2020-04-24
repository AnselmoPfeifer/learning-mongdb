# Chapter 3 - Sharded Cluster

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

- Bring up the `mongos`, and try to authenticate to mongos immediately as `m103-admin` user:
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
sh.addShard("m103-repl/192.168.103.100:27001")
sh.status()
```

## Lab - Shard a Collection
**Problem:**

At this point, your cluster is now configured for sharding. You should already have a CSRS, mongos, and primary shard.

In this lab, you will do the following with your cluster:
  - Add a second shard
  - Import a dataset onto your primary shard
  - Choose a shard key and shard your collection.

- [Initialize m103-repl-2 as a normal replica set](/replication/Chapter-2-labs.md), and add a Second Shard
```
mkdir /var/mongodb/db/node{4,5,6}
sudo cp node{4,5,6}.conf /etc/mongod/
```

- Connect to mongos to add m103-repl-2 as a shard with the following command:
```
sh.addShard("m103-repl-2/192.168.103.100:27004")
```
As result we'll see:
```json
{
	"shardAdded" : "m103-repl-2",
	"ok" : 1,
	"operationTime" : Timestamp(1587723884, 7),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1587723884, 7),
		"signature" : {
			"hash" : BinData(0,"YlsZWDdXr4Qt7HWL2HYUHVIhwLY="),
			"keyId" : NumberLong("6815321136847388699")
		}
	}
}
```
Checking status:
```
sh.status()
```
As result we'll see:
```json
--- Sharding Status --- 
  sharding version: {
  	"_id" : 1,
  	"minCompatibleVersion" : 5,
  	"currentVersion" : 6,
  	"clusterId" : ObjectId("5e94e277ce31c10fb88f1f14")
  }
  shards:
        {  "_id" : "m103-repl",  "host" : "m103-repl/192.168.103.100:27001,m103:27002,m103:27003",  "state" : 1 }
        {  "_id" : "m103-repl-2",  "host" : "m103-repl-2/192.168.103.100:27004,192.168.103.100:27005,192.168.103.100:27006",  "state" : 1 }
  active mongoses:
        "3.6.17" : 1
  ---
```

- Importing Data onto the Primary Shard
```
mongoimport --drop products.json --port 26000 -u "m103-admin" \
-p "m103-pass" --authenticationDatabase "admin" \
--db m103 --collection products
```

- Sharding the Collection
```
sh.enableSharding("m103")
```
As result we'll see:
```json
{
	"ok" : 1,
	"operationTime" : Timestamp(1587724479, 4),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1587724479, 4),
		"signature" : {
			"hash" : BinData(0,"oTfIHjdvroD6WK8QOz3QFzrpef8="),
			"keyId" : NumberLong("6815321136847388699")
		}
	}
}
```
If you want a better understanding of the distribution and the nature of the fields and their types, you can do so using [Compass](https://www.mongodb.com/products/compass).

- Creating an index on the shard key field:
```
db.products.createIndex({"name": 1})
```
As result we'll see:
```json
{
	"raw" : {
		"m103-repl-2/192.168.103.100:27004,192.168.103.100:27005,192.168.103.100:27006" : {
			"createdCollectionAutomatically" : false,
			"numIndexesBefore" : 2,
			"numIndexesAfter" : 3,
			"ok" : 1
		}
	},
	"ok" : 1,
	"operationTime" : Timestamp(1587757685, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1587757685, 1),
		"signature" : {
			"hash" : BinData(0,"0Tl1I83/DssuIhzRUjAyVslNQQA="),
			"keyId" : NumberLong("6815321136847388699")
		}
	}
}
```

- Once the index is created, shard the collection with the following command:
```
db.adminCommand( { shardCollection: "m103.products", key: { name: 1 } } ) 
```
As result we'll see:
```json
{
	"collectionsharded" : "m103.products",
	"collectionUUID" : UUID("0c72ad3e-1c48-40ba-960f-0413c62cd6aa"),
	"ok" : 1,
	"operationTime" : Timestamp(1587757733, 12),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1587757733, 12),
		"signature" : {
			"hash" : BinData(0,"levGugMPM3egNqQ0IisHG78Rqh0="),
			"keyId" : NumberLong("6815321136847388699")
		}
	}
}
```
