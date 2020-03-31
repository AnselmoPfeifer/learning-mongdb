## Chapter 1

## Launching and shutdown Mongo Server  
- 1: 
```
mongod --port 30000 --dbpath /data/db --logpath /data/log/mongod.log --fork
mongo admin --port 30000 --eval 'db.shutdownServer()'
```
- 2: 
```
mongod --port 27000 --dbpath /data/db/ --bind_ip 192.168.103.100,127.0.0.1 --logpath /data/log/mongod.log --fork --auth --config /etc/mongod.conf
```
```
mongo --port 27000
  use admin
  db.shutdownServer()
  quit()

```

## Connect to the Mongo shell and create the following user.
```
    mongo admin --host localhost:27000 --eval '
        db.createUser({
            user: "m103-admin",
            pwd: "m103-pass",
            roles: [
                {role: "root", db: "admin"}
        ]
    })'
```

## Basic Configuration File
```
mongod --dbpath /data/db --logpath /data/log/mongod.log --fork --replSet "M103" --keyFile /data/keyfile --bind_ip "127.0.0.1,192.168.103.100" --tlsMode requireTLS --tlsCAFile "/etc/tls/TLSCA.pem" --tlsCertificateKeyFile "/etc/tls/tls.pem"
``` 
```
mongod --port 27000 --dbpath /data/db/ --bind_ip 192.168.103.100,127.0.0.1 --logpath /var/log/mongodb/mongod.log --fork --auth
```
```
# mongod --config /etc/mongod.conf
storage:
  dbPath: "/data/db"
systemLog:
  path: "/data/log/mongod.log"
  destination: "file"
replication:
  replSetName: M103
net:
  bindIp : "127.0.0.1,192.168.103.100"
tls:
  mode: "requireTLS"
  certificateKeyFile: "/etc/tls/tls.pem"
  CAFile: "/etc/tls/TLSCA.pem"
security:
  keyFile: "/data/keyfile"
processManagement:
  fork: true
```

## Lab - Configuration File
```
sudo mkdir -p /data/db /data/log
sudo chown -R vagrant:vagrant /data/
mongod --config /etc/mongod.conf
```
```
storage:
  dbPath: "/data/db/"
  journal:
    enabled: true
  
systemLog:
  path: "/data/log/mongod.log"
  destination: "file"
  logAppend: true

net:
  port: 27000
  bindIp: "127.0.0.1,192.168.103.100"

processManagement:
  timeZoneInfo: "/usr/share/zoneinfo"
  fork: true

security:
  authorization: enabled
```

- Create new user:
```
mongo admin --host localhost:27000 --eval '
    db.createUser({
        user: "m103-admin",
        pwd: "m103-pass",
        roles: [
            {role: "root", db: "admin"}
    ]
})'
```

## Lab - Change the Default DB Path
```
sudo mkdir -p /var/mongodb/db /var/mongodb/log
sudo chown -R vagrant:vagrant /var/mongodb
sudo cp mongod.conf /etc/mongod.conf
mongod --config /etc/mongod.conf
```
```
storage:
  dbPath: "/var/mongodb/db/"
  journal:
    enabled: true
  
systemLog:
  path: "/var/mongodb/log/mongod.log"
  destination: "file"
  logAppend: true

net:
  port: 27000
  bindIp: "127.0.0.1,192.168.103.100"

processManagement:
  timeZoneInfo: "/usr/share/zoneinfo"
  fork: true

security:
  authorization: enabled
```

- Create new user:
```
mongo admin --host localhost:27000 --eval '
    db.createUser({
        user: "m103-admin",
        pwd: "m103-pass",
        roles: [
            {role: "root", db: "admin"}
    ]
})'
```

## Basic commands
- User management commands:
```
db.createUser()
db.dropUser()
```
- Collection management commands:
```
db.renameCollection()
db.collection.createIndex()
db.collection.drop()

db.<collection>.renameCollection()
db.<collection>.createIndex()
db.<collection>.drop()
```
- Database management commands:
```
db.dropDatabase()
db.createCollection()
```
- Database status command:
```
db.serverStatus()
```
- Creating index with Database Command:
```
db.runCommand(
  { "createIndexes": <collection> },
  { "indexes": [
    {
      "key": { "product": 1 }
    },
    { "name": "name_index" }
    ]
  }
)
```
- Creating index with Shell Helper:
```
db.<collection>.createIndex(
  { "product": 1 },
  { "name": "name_index" }
)
```
- Introspect a Shell Helper:
```
db.<collection>.createIndex
```

## Logging Basics
- Get the logging components:
```
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.getLogComponents()
' 
or 
db.getLogComponents()
```
- Get the logging components:
```
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.setLogLevel(0, "index")
'
or 
db.setLogLevel(0, "index")
```
- View the logs through the Mongo shell:
```
db.adminCommand({ "getLog": "global" })
```
- View the logs through the command line:
```
tail -f /data/db/mongod.log
```
- Update a document:
```
mongo admin --host 192.168.103.100:27000 -u m103-admin -p m103-pass --eval '
  db.products.update( { "sku" : 6902667 }, { $set : { "salePrice" : 39.99} } )
'
```
- Look for instructions in the log file with grep:
```
grep -i 'update' /data/db/mongod.log
```

## Profiling the Database
- To list all of the collection names you can run this command:
```
db.runCommand({listCollections: 1})
```
- Get profiling level:
```
mongo newDB --host 192.168.103.100:27000 -u m103-admin -p m103-pass --authenticationDatabase admin --eval '
  db.getProfilingLevel()'
```
- Set profiling level:
```
mongo newDB --host 192.168.103.100:27000 -u m103-admin -p m103-pass --authenticationDatabase admin --eval '
  db.setProfilingLevel(1)'
```
- Show collections:
```
mongo newDB --host 192.168.103.100:27000 -u m103-admin -p m103-pass --authenticationDatabase admin --eval '
  db.getCollectionNames()'
// Note: show collections only works from within the shell
```
- Set slowms to 0:
```
mongo newDB --host 192.168.103.100:27000 -u m103-admin -p m103-pass --authenticationDatabase admin --eval '
  db.setProfilingLevel( 1, { slowms: 0 } )'
```
- Insert one document into a new collection:
```
mongo newDB --host 192.168.103.100:27000 -u m103-admin -p m103-pass --authenticationDatabase admin --eval '
  db.new_collection.insert( { "a": 1 } )'
```
- Get profiling data from system.profile:
```
mongo newDB --host 192.168.103.100:27000 -u m103-admin -p m103-pass --authenticationDatabase admin --eval '
  db.system.profile.find().pretty()'
```

## Lab - Logging to a Different Facility
- mongod sends logs to /var/mongodb/db/mongod.log
- mongod is forked and run as a daemon (this will not work without a logpath)
- any query that takes 50ms or longer is logged (remember to specify this in the configuration file!)

```
ps aux | grep mongo
kill -9 <pid>
```
```
operationProfiling:
    mode: "slowOp"
    slowOpThresholdMs: 50
    slowOpSampleRate: 1.0
```
- Starting mongo: `mongod --config /etc/mongod.conf`

## Basic Security:
- Add security authorization:
```
security:
  authorization: enabled
```
- Connection from localhost:
```
mongo --host 127.0.0.1 --port 27000
db.stats() // Permission denied:)

// result error Unauthorized:
MongoDB Enterprise > db.stats() 
{
	"ok" : 0,
	"errmsg" : "not authorized on admin to execute command { dbstats: 1.0, scale: undefined, lsid: { id: UUID(\"c257505d-9616-4537-a1da-4a7f82b514e4\") }, $db: \"admin\" }",
	"code" : 13,
	"codeName" : "Unauthorized"
}
```
- Create new user with the root role (also, named root):
```
db.createUser(
  {
    user: "root", 
    pwd: "root", 
    roles: ["root"]
  }
) 
```
- Connect to mongod and authenticate as root, and run DB stats: ([Result](stats.json))
```
mongo --port 27000 --username root --password root --authenticationDatabase admin
db.stats()
```
- Shutdown the server:
```
use admin
db.shutdownServer()
```

## Built-In Roles:
- Authenticate as root user:
```
mongo admin -u root -p root123 --port 27000
```
- Create security officer: (role: userAdmin)
```
db.createUser(
  { user: "security_officer",
    pwd: "h3ll0th3r3",
    roles: [ { db: "admin", role: "userAdmin" } ]
  }
)
```
- Create database administrator: (role: dbAdmin)
```
db.createUser(
  { user: "dba",
    pwd: "c1lynd3rs",
    roles: [ { db: "admin", role: "dbAdmin" } ]
  }
)
```
- Grant role to user:
```
db.grantRolesToUser( "dba",  [ { db: "playground", role: "dbOwner"  } ] )
```
- Show role privileges: [Result](showPrivileges.json)
```
db.runCommand( { rolesInfo: { role: "dbOwner", db: "playground" }, showPrivileges: true} ) 
```

## Lab - Creating First Application User:
- run on port 27000
- data files are stored in /var/mongodb/db/
- listens to connections from the IP address 192.168.103.100 and localhost
- authentication is enabled
- root user on admin database with username: m103-admin and password: m103-pass
```
db.createUser(
  { user: "m103-admin",
    pwd: "m103-pass",
    roles: [ { db: "admin", role: "dbAdmin" } ]
  }
)
```
- Role: readWrite on applicationData database
- Authentication source: admin
- Username: m103-application-user
- Password: m103-application-pass
```
db.createUser(
  { user: "m103-application-user",
    pwd: "m103-application-pass",
    roles: [ { db: "applicationData", role: "readWrite" } ]
  }
)
```


## Server Tools Overview
- List mongodb binaries:
```
find /usr/bin/ -name "mongo*"
```
- Use mongostat to get stats on a running mongod process:
```
mongostat --help
mongostat --port 27000 --username root --password root --authenticationDatabase admin
```
```result
insert query update delete getmore command dirty used flushes vsize   res qrw arw net_in net_out conn                time
    *0    *0      1      1       0     3|0  0.0% 0.0%       1 1.06G 50.0M 0|0 1|0   158b   61.7k    2 Mar 30 20:00:05.521
    *0    *0     *0     *0       0     2|0  0.0% 0.0%       0 1.06G 50.0M 0|0 1|0   159b   62.2k    2 Mar 30 20:00:06.512
    *0    *0     *0     *0       0     1|0  0.0% 0.0%       0 1.06G 50.0M 0|0 1|0   157b   61.4k    2 Mar 30 20:00:07.516
    *0    *0     *0     *0       0     2|0  0.0% 0.0%       0 1.06G 50.0M 0|0 1|0   158b   61.8k    2 Mar 30 20:00:08.512
    *0    *0     *0     *0       0     1|0  0.0% 0.0%       0 1.06G 50.0M 0|0 1|0   150b   58.8k    2 Mar 30 20:00:09.561
```
- Use mongodump to get a BSON dump of a MongoDB collection:
```
mongodump --help
mongodump --port 27000 --username root --password root --authenticationDatabase admin --db dataBaseName --collection collectionName
ls dump/dataBaseName/
cat dump/dataBaseName/collectionName.metadata.json
```
- Use mongorestore to restore a MongoDB collection from a BSON dump folder:
```
mongorestore --drop --port 27000 --username root --password root --authenticationDatabase admin dump/
```
- Use mongoexport to export a MongoDB collection to JSON or CSV (or stdout!):
```
mongoexport --help
mongoexport --port 27000 --username root --password root --authenticationDatabase admin --db dataBaseName --collection collectionName -o collectionName.json
```
- Use mongoimport to create a MongoDB collection from a JSON or CSV file:
```
mongoimport --verbose --port 27000 --username root --password root --authenticationDatabase admin --db applicationData products.json
```

## Lab - Importing a Dataset:
- Preparing the host
```
sudo mkdir -p /var/mongodb/db /var/mongodb/log
sudo chown -R vagrant:vagrant /var/mongodb
sudo cp mongod.conf /etc/mongod.conf
mongod --config /etc/mongod.conf

mongo --host 127.0.0.1 --port 27000
use admin
db.createUser({
    user: "root", 
    pwd: "root", 
    roles: [ "root" ]
})

db.createUser({
    user: "m103-admin",
    pwd: "m103-pass",
    roles: [ { role: "root", db: "admin" } ]
})

mongo --port 27000 --username root --password root --authenticationDatabase admin
db.createUser(
  { user: "m103-application-user",
    pwd: "m103-application-pass",
    roles: [ { db: "applicationData", role: "readWrite" } ]
  }
)
```
- Using `mongoimport` to import a JSON dataset into MongoDB.
Import the whole dataset with your application's user m103-application-user into a collection called products
```
mongoimport --verbose --port 27000 --username m103-application-user --password m103-application-pass --authenticationDatabase admin --db applicationData --collection products products.json
```
