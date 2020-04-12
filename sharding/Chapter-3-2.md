# Chapter 3 - Sharded Cluster

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
