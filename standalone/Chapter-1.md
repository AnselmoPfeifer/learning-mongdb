# Chapter 1 

## Introduction & Setup
```
vagrant ssh
mongo 
show dbs
mongo admin --eval 'db.shutdownServer()'

mongod --port 30000 --dbpath /var/lib/mongod --logpath /var/mongod/mongod.log --fork
mongo admin --port 30000 --eval 'db.shutdownServer()'
```

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
- Connect to mongod and authenticate as root, and run DB stats: ([Result](result/stats.json))
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
- Show role privileges: [Result](result/showPrivileges.json)
```
db.runCommand( { rolesInfo: { role: "dbOwner", db: "playground" }, showPrivileges: true} ) 
```
