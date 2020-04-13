# Chapter 3 - Sharded Cluster
Sharding is a method for distributing data across multiple machines. MongoDB uses sharding to support deployments with very large data sets and high throughput operations, [more information](https://docs.mongodb.com/manual/sharding/).

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

- Connect to one of the config servers, initiating the CSRS, and creating super user on CSRS, Authenticating as the super user, to add the second and third node to the CSRS:
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

db.auth("m103-admin", "m103-pass")
rs.add("192.168.103.100:26002")
rs.add("192.168.103.100:26003")
rs.status()
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

- Adding new shard to cluster from mongos, and check with `sh.status()` - [Result](result/shStatus.json)
```
mongo --port 26000 --username m103-admin --password m103-pass --authenticationDatabase admin
sh.addShard("m103-repl/m103:27002")
sh.status()
```

## Config DB
* You should generally never write any data to it.
> When should you manually write data to the Config DB? 
>> When directed to by MongoDB documentation or Support Engineers

- Switch to config DB: `use config`
- Query config.databases: `db.databases.find().pretty()`
- Query config.collections: `db.collections.find().pretty()`
```json
{
	"_id" : "config.system.sessions",
	"lastmodEpoch" : ObjectId("5e91c30a14e33b5c89634be3"),
	"lastmod" : ISODate("1970-02-19T17:02:47.296Z"),
	"dropped" : false,
	"key" : {
		"_id" : 1
	},
	"unique" : false,
	"uuid" : UUID("992a0fd4-9a27-4ebb-a0ab-60f087a1f02f")
}

```

- Query config.shards: `db.shards.find().pretty()`
```json
{
	"_id" : "m103-repl",
	"host" : "m103-repl/192.168.103.100:27011,192.168.103.100:27012,192.168.103.100:27013",
	"state" : 1
}
```

- Query config.chunks: `db.chunks.find().pretty()`
```json
{
	"_id" : "config.system.sessions-_id_MinKey",
	"ns" : "config.system.sessions",
	"min" : {
		"_id" : { "$minKey" : 1 }
	},
	"max" : {
		"_id" : { "$maxKey" : 1 }
	},
	"shard" : "m103-repl",
	"lastmod" : Timestamp(1, 0),
	"lastmodEpoch" : ObjectId("5e91c30a14e33b5c89634be3")
}
```

- Query config.mongos: `db.mongos.find().pretty()`
```json
{
	"_id" : "m103:26000",
	"advisoryHostFQDNs" : [ ],
	"mongoVersion" : "3.6.17",
	"ping" : ISODate("2020-04-12T11:36:21.385Z"),
	"up" : NumberLong(280),
	"waiting" : true
}
```

## Shard Keys
> As of MongoDB 4.2, the shard key value is mutable, even though the shard key itself is immutable. If you're interested, you can read more in the [Shard Key documentation](https://docs.mongodb.com/manual/core/sharding-shard-key/index.html).
>> Unsharding a collection is hard - avoid it.

- Create a new db and importe a product collection: `use m103`
```
mongoimport --verbose --port 27001 --username m103-admin --password m103-pass --authenticationDatabase admin --db m103 --collection products products.json

show collections
```

- Enable sharding on the m103 database: `sh.enableSharding("m103")`
```json
{
	"ok" : 1,
	"operationTime" : Timestamp(1586816331, 8),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1586816331, 8),
		"signature" : {
			"hash" : BinData(0,"N/VCwdKXTKc+0LVi1pYMG0fCM3Y="),
			"keyId" : NumberLong("6815321136847388699")
		}
	}
}
```

- Find one document from the products collection, to help us choose a shard key: `db.products.findOne()`
```json
{
	"_id" : ObjectId("573f706ff29313caab7d7395"),
	"sku" : 1000000749,
	"name" : "Gods And Heroes: Rome Rising - Windows [Digital Download]",
	"type" : "Software",
	"regularPrice" : 39.95,
	"salePrice" : 39.95,
	"shippingWeight" : "0.01"
}
```

- Create an index on sku: `db.products.createIndex( { "sku" : 1 } )`
```json
{
	"raw" : {
		"m103-repl/192.168.103.100:27001,m103:27002,m103:27003" : {
			"createdCollectionAutomatically" : false,
			"numIndexesBefore" : 1,
			"numIndexesAfter" : 2,
			"ok" : 1
		}
	},
	"ok" : 1,
	"operationTime" : Timestamp(1586816832, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1586816832, 1),
		"signature" : {
			"hash" : BinData(0,"gBgvcmvSSbP3wW1kjm9+LVwylsw="),
			"keyId" : NumberLong("6815321136847388699")
		}
	}
}
```

