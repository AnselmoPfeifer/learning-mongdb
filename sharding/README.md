# Chapter 3

## To create a new replica set with 3 nodes run the following commands:
```
sudo mkdir -p /var/mongodb/db/csrs{1,2,3} 
sudo mkdir -p /var/mongodb/db/node{1,2,3} 
sudo chown vagrant:vagrant -R /var/mongodb/db
sudo cp node*.conf /etc/mongod/

mongod -f /etc/mongod/node1.conf
mongod -f /etc/mongod/node2.conf
mongod -f /etc/mongod/node3.conf
```

## Initialize replica set
```
mongo --port 27001
rs.initiate()
rs.status()
```

## Create a new admin user
```
use admin
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles: [
    {role: "root", db: "admin"}
  ]
})
```

## Connect with new user, and add a new hosts
```
mongo --host "m103-repl/192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
rs.status()
rs.add("m103:27002")
rs.add("m103:27003")
rs.status()
```