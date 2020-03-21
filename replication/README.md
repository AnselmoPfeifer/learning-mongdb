# learning-mongdb

## Chapter 2 - Replica Set Setting Up 
validate_lab_initialize_local_replica_set: `5a4d32f979235b109001c7bc`
- Creating the keyfile and setting permissions on it:
```
sudo mkdir -p /var/mongodb/pki/
sudo chown vagrant:vagrant /var/mongodb/pki/
openssl rand -base64 741 > /var/mongodb/pki/m103-keyfile
chmod 400 /var/mongodb/pki/m103-keyfile

// To execute it in differents machines just copy this key file
```
- Reating the dbpath for node1: 
```
sudo mkdir -p /var/mongodb/db/node1
sudo chown vagrant:vagrant /var/mongodb/db/node1
```
- Starting a mongod with node1.conf:
```
mongod -f node1.conf

ps aux | grep mongo 
vagrant   6430 11.2  2.4 1136016 49648 ?       Sl   21:05   0:00 mongod -f node1.conf
```
- Copying node1.conf to node2.conf and node3.conf:
```
cp node1.conf node2.conf
cp node2.conf node3.conf
```

- Editing node2.conf dbpath, port, and logpath must be changed:
```
storage:
  dbPath: /var/mongodb/db/node2
net:
  bindIp: 192.168.103.100,localhost
  port: 27012
security:
  keyFile: /var/mongodb/pki/m103-keyfile
systemLog:
  destination: file
  path: /var/mongodb/db/node2/mongod.log
  logAppend: true
processManagement:
  fork: true
replication:
  replSetName: m103-example
```

- Editing node2.conf dbpath, port, and logpath must be changed:
```
storage:
  dbPath: /var/mongodb/db/node3
net:
  bindIp: 192.168.103.100,localhost
  port: 27013
security:
  keyFile: /var/mongodb/pki/m103-keyfile
systemLog:
  destination: file
  path: /var/mongodb/db/node3/mongod.log
  logAppend: true
processManagement:
  fork: true
replication:
  replSetName: m103-example
```
- Creating the data directories for node2 and node3:
```
sudo mkdir /var/mongodb/db/{node2,node3}
sudo chown vagrant:vagrant /var/mongodb/db/{node2,node3}
```
- Starting mongod processes with node2.conf and node3.conf:
```
mongod -f node2.conf
mongod -f node3.conf
```

- Connecting to node1:
```
mongo --port 27011

MongoDB shell version v3.6.17
connecting to: mongodb://127.0.0.1:27011/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("df056da2-fac4-49f8-8126-16c4cf011b3e") }
MongoDB server version: 3.6.17
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
        http://docs.mongodb.org/
Questions? Try the support group
        http://groups.google.com/group/mongodb-user
MongoDB Enterprise > 
```

- Initiating the replica set:
```
rs.initiate()
{
        "info2" : "no configuration specified. Using a default configuration for the set",
        "me" : "192.168.103.100:27011",
        "ok" : 1
}
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

Successfully added user: {
        "user" : "m103-admin",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ]
}
```

- Exiting out of the Mongo shell and connecting to the entire replica set:
```

mongo --host "m103-example/192.168.103.100:27011" -u "m103-admin"
-p "m103-pass" --authenticationDatabase "admin"

Use this password: m103-pass
```

- Getting replica set status:
```
rs.status()

{
        "set" : "m103-example",
        "date" : ISODate("2020-03-21T21:18:24.150Z"),
        "myState" : 1,
        "term" : NumberLong(1),
        "syncingTo" : "",
        "syncSourceHost" : "",
        "syncSourceId" : -1,
        "heartbeatIntervalMillis" : NumberLong(2000),
        "optimes" : {
                "lastCommittedOpTime" : {
                        "ts" : Timestamp(1584825500, 1),
                        "t" : NumberLong(1)
                },
                "readConcernMajorityOpTime" : {
                        "ts" : Timestamp(1584825500, 1),
                        "t" : NumberLong(1)
                },
                "appliedOpTime" : {
                        "ts" : Timestamp(1584825500, 1),
                        "t" : NumberLong(1)
                },
                "durableOpTime" : {
                        "ts" : Timestamp(1584825500, 1),
                        "t" : NumberLong(1)
                }
        },
        "members" : [
                {
                        "_id" : 0,
                        "name" : "192.168.103.100:27011",
                        "health" : 1,
                        "state" : 1,
                        "stateStr" : "PRIMARY",
                        "uptime" : 785,
                        "optime" : {
                                "ts" : Timestamp(1584825500, 1),
                                "t" : NumberLong(1)
                        },
                        "optimeDate" : ISODate("2020-03-21T21:18:20Z"),
                        "syncingTo" : "",
                        "syncSourceHost" : "",
                        "syncSourceId" : -1,
                        "infoMessage" : "",
                        "electionTime" : Timestamp(1584825309, 2),
                        "electionDate" : ISODate("2020-03-21T21:15:09Z"),
                        "configVersion" : 1,
                        "self" : true,
                        "lastHeartbeatMessage" : ""
                }
        ],
        "ok" : 1,
        "operationTime" : Timestamp(1584825500, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1584825500, 1),
                "signature" : {
                        "hash" : BinData(0,"YyTli1+6gxuwIdKpiJu4PnsLTsE="),
                        "keyId" : NumberLong("6806772876323061761")
                }
        }
}
```

Adding other members to replica set:
```
rs.add("192.168.103.100:27012")
rs.add("192.168.103.100:27013")


// Output 
{
        "ok" : 1,
        "operationTime" : Timestamp(1584825554, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1584825554, 1),
                "signature" : {
                        "hash" : BinData(0,"dyKaTt9aFoRVauoaC68VsMnJ0lw="),
                        "keyId" : NumberLong("6806772876323061761")
                }
        }
}

{
        "ok" : 1,
        "operationTime" : Timestamp(1584825555, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1584825555, 1),
                "signature" : {
                        "hash" : BinData(0,"/GB4yUizsD//Pw4Y9W85vZTWO2k="),
                        "keyId" : NumberLong("6806772876323061761")
                }
        }
}
```

- Getting an overview of the replica set topology:
```
rs.isMaster()

// Output 
{
        "hosts" : [
                "192.168.103.100:27011",
                "m103:27012",
                "m103:27013"
        ],
        "setName" : "m103-example",
        "setVersion" : 3,
        "ismaster" : true,
        "secondary" : false,
        "primary" : "192.168.103.100:27011",
        "me" : "192.168.103.100:27011",
        "electionId" : ObjectId("7fffffff0000000000000001"),
        "lastWrite" : {
                "opTime" : {
                        "ts" : Timestamp(1584825619, 1),
                        "t" : NumberLong(1)
                },
                "lastWriteDate" : ISODate("2020-03-21T21:20:19Z"),
                "majorityOpTime" : {
                        "ts" : Timestamp(1584825619, 1),
                        "t" : NumberLong(1)
                },
                "majorityWriteDate" : ISODate("2020-03-21T21:20:19Z")
        },
        "maxBsonObjectSize" : 16777216,
        "maxMessageSizeBytes" : 48000000,
        "maxWriteBatchSize" : 100000,
        "localTime" : ISODate("2020-03-21T21:20:30.214Z"),
        "logicalSessionTimeoutMinutes" : 30,
        "minWireVersion" : 0,
        "maxWireVersion" : 6,
        "readOnly" : false,
        "ok" : 1,
        "operationTime" : Timestamp(1584825619, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1584825619, 1),
                "signature" : {
                        "hash" : BinData(0,"fUtbS0J0iy8W9/cNi4u9bUE5D4M="),
                        "keyId" : NumberLong("6806772876323061761")
                }
        }
}
```

Stepping down the current primary:
```
rs.stepDown()
```

- Checking replica set overview after election:
```
rs.isMaster()
```
