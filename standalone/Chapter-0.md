# Chapter 0 - Introduction & Setup
```
vagrant ssh
mongo 
show dbs
mongo admin --eval 'db.shutdownServer()'

mongod --port 30000 --dbpath /var/lib/mongod --logpath /var/mongod/mongod.log --fork
mongo admin --port 30000 --eval 'db.shutdownServer()'
```