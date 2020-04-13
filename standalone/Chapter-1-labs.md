# Chapter 1

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
