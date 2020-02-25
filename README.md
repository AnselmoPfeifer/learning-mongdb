# learning-mongdb

## Chapter 0
````
vagrant ssh
mongo 
show dbs
mongo admin --eval 'db.shutdownServer()'

mongod --port 30000 --dbpath /var/lib/mongod --logpath /var/mongod/mongod.log --fork
mongo admin --port 30000 --eval 'db.shutdownServer()'
````

## Chapter 1
````
mongod --port 30000 --dbpath first_mongod --logpath first_mongod/mongod.log --fork
mongo admin --port 30000 --eval 'db.shutdownServer()'

mongod --port 27000 --dbpath /data/db/ --bind_ip 192.168.103.100,127.0.0.1 --logpath /var/log/mongodb/mongod.log --fork --auth --config /etc/mongod.conf
mongo --port 27000
mongo admin --host localhost:27000 --eval '
  db.createUser({
    user: "m103-admin",
    pwd: "m103-pass",
    roles: [
      {role: "root", db: "admin"}
    ]
  })'
  
mongo --port 27000 

mongod --dbpath /data/db --logpath /data/log/mongod.log --fork --replSet "M103" --keyFile /data/keyfile --bind_ip "127.0.0.1,192.168.103.100" --tlsMode requireTLS --tlsCAFile "/etc/tls/TLSCA.pem" --tlsCertificateKeyFile "/etc/tls/tls.pem"

mongod --port 27000 --dbpath /data/db/ --bind_ip 192.168.103.100,127.0.0.1 --logpath /var/log/mongodb/mongod.log --fork --auth

mongo --port 27000
  use admin
  db.shutdownServer()
  quit()

mongod --config /etc/mongod.conf
````
* Basic commands
- User management commands:
````
db.createUser()
db.dropUser()
````
- Collection management commands:
````
db.renameCollection()
db.collection.createIndex()
db.collection.drop()

db.<collection>.renameCollection()
db.<collection>.createIndex()
db.<collection>.drop()
````

- Database management commands:
````
db.dropDatabase()
db.createCollection()
````
- Database status command:
````
db.serverStatus()
````
- Creating index with Database Command:
````
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
````
- Creating index with Shell Helper:
````
db.<collection>.createIndex(
  { "product": 1 },
  { "name": "name_index" }
)
````
- Introspect a Shell Helper:
````
db.<collection>.createIndex
````

* Logging Basics
- Get the logging components:
````
db.getLogComponents()
````
- Get the logging components:
````
db.setLogLevel(0, "index")
````
- View the logs through the Mongo shell:
````
db.adminCommand({ "getLog": "global" })
````
* Profiling the Database
````
mongo --port 2700
use newDataBase
db.getProfilingLevel()
db.setProfilingLevel(1)
db.new_collection.insert( { "a": 1 } )
````
* Basic Security:
Cheking and set the permission
````
security:
  authorization: enabled
````
````
mongo --host 127.0.0.1 --port 27000
 db.stats() // Permission denied:)
db.createUser(
  {
    user: "root", 
    pwd: "root", 
    roles: ["root"]
  }
) 
````
- Cheking new user permision 
````
mongo --host 127.0.0.1 --port 27000 --username root --password root --authenticationDatabase admin
db.stats()
````
* Built-In Roles:
- Create security officer:
````
db.createUser(
  {
    user: "security_officer", 
    pwd: "h3ll0th3r3", 
    roles: [{
      db: "admin",
      role: "userAdmin"
    }]
  }
) 
````
- Create database administrator:
````
db.createUser(
  { user: "dba",
    pwd: "c1lynd3rs",
    roles: [ { db: "m103", role: "dbAdmin" } ]
  }
) 
db.createUser(
  { user: "m103-application-user",
    pwd: "m103-application-pass",
    roles: [ { db: "applicationData", role: "readWrite" } ]
  }
) 

````
- Grant role to user and Show role privileges:
````
db.grantRolesToUser( "dba",  [ { db: "playground", role: "dbOwner"  } ] )
db.runCommand( { rolesInfo: { role: "dbOwner", db: "playground" }, showPrivileges: true} )
````

* Server Tools Overview
- List mongodb binaries:
````
find /usr/bin/ -name "mongo*"
````
- Create new dbpath and launch mongod:
````
mkdir -p ~/first_mongod
mongod --port 30000 --dbpath ~/first_mongod --logpath ~/first_mongod/mongodb.log --fork
````
- Use mongostat to get stats on a running mongod process:
````
mongostat --help
mongostat --port 30000
````
- Use mongostat to get stats on a running mongod process:
````
mongodump --help
mongodump --port 30000 --db applicationData --collection products
ls dump/applicationData/
cat dump/applicationData/products.metadata.json
````
- Use mongorestore to restore a MongoDB collection from a BSON dump:
````
mongorestore --drop --port 30000 dump/
````
- Use mongoexport and mongoimport
````
mongoexport --help
mongoexport --port 30000 --db applicationData --collection products
mongoexport --port 30000 --db applicationData --collection products -o products.json
mongoimport --port 30000 --username m103-application-user --password m103-application-pass --authenticationDatabase admin products.json
````

##  Links
- [Mongo University](https://university.mongodb.com/dashboard)
sudo cp mongod.conf /etc/mongod.conf
sudo mkdir -p /var/mongodb/db
sudo touch /var/mongodb/db/mongod.log
sudo chown -R vagrant:vagrant /var/mongodb/

mongo admin --host localhost:27000 --eval '
  db.createUser({
    user: "m103-admin",
    pwd: "m103-pass",
    roles: [
      {role: "root", db: "admin"}
    ]
  })'

mongoimport --port 27000 --username m103-application-user --password m103-application-pass --authenticationDatabase admin products.json