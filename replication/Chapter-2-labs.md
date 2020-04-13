# Chapter 2 - Replication

## Lab - Initiate a Replica Set Locally

```
PRIMARY: 
    - mongod-repl-1.conf
    - dbPath: /var/mongodb/db/1
    - port: 27001
    - logPath: /var/mongodb/db/mongod1.log
    - keyFile: /var/mongodb/pki/m103-keyfile
    - bindIP: localhost,192.168.103.100
SECONDARY: 
    - mongod-repl-2.conf
    - dbPath: /var/mongodb/db/2
    - port: 27002
    - logPath: /var/mongodb/db/mongod1.log
    - keyFile: /var/mongodb/pki/m103-keyfile
    - bindIP: localhost,192.168.103.100
SECONDARY: 
    - mongod-repl-3.conf
    - dbPath: /var/mongodb/db/3
    - port: 27003
    - logPath: /var/mongodb/db/mongod1.log
    - keyFile: /var/mongodb/pki/m103-keyfile
    - bindIP: localhost,192.168.103.100
``` 
```
sudo rm -rf /var/mongodb/db/{1,2,3}
sudo mkdir -p /var/mongodb/db/{1,2,3} 
sudo chown vagrant:vagrant -R /var/mongodb/db

sudo cp mongod-repl-*.conf /etc/mongod/
mongod -f /etc/mongod/mongod-repl-1.conf
mongod -f /etc/mongod/mongod-repl-2.conf
mongod -f /etc/mongod/mongod-repl-3.conf
mongo --port 27001
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

mongo --host "m103-repl/192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
rs.status()
rs.add("192.168.103.100:27002")
rs.add("192.168.103.100:27003")
```

## Lab - Remove and Re-Add a Node:

- The configuration of the nodes should not change - the hostname m103 is already bound to the IP address 192.168.103.100
- The nodes should still run on ports 27001, 27002, 27003
- The name of your replica set should still be m103-repl
```
sudo rm -rf /var/mongodb/db/{1,2,3}
sudo mkdir -p /var/mongodb/db/{1,2,3} 
sudo chown vagrant:vagrant -R /var/mongodb/db

mongod -f /etc/mongod/mongod-repl-1.conf
mongod -f /etc/mongod/mongod-repl-2.conf
mongod -f /etc/mongod/mongod-repl-3.conf
mongo --port 27001
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

mongo --host "m103-repl/m103:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
rs.add("m103:27002")
rs.add("m103:27003")

rs.remove("192.168.103.100:27001")
rs.add("m103:27001")
rs.status()
```
