### Start a MongoDB cluster with Docker Compose

Pending issue: Unable to connect to the MongoDB cluster after it starts up.

1、Start a MongoDB instances

```
docker-compose up -d
```

2、Log in to the primary node (choose any one randomly)

```
mongosh mongodb://user:pass@127.0.0.1:27017
```

3、Configure the primary node

```
rs.initiate(
  {
    _id: "rs1",
    version: 1,
    members: [
      { _id: 0, host : "mongo_1:27017", priority: 1 }
    ]
  }
);
rs.conf()
rs.status()
```

4、Add members to the replica set

```
rs.add({ host: 'mongo_2:27017', priority: 0 })
rs.add({ host: 'mongo_3:27017', priority: 0 })
rs.status()
```

5、Create a user

```
use naxx;
db.createUser({
  user: 'user',
  pwd: 'pass',
  roles: [
    {
      role: 'dbOwner',
      db: 'naxx'
    }
  ]
});
```

6、Connect to the cluster

Error occurred when connecting to the cluster, unable to connect

```
mongosh mongodb://user:pass@127.0.0.1:27017,127.0.0.1:27018,127.0.0.1:27019/naxx?replicaSet=rs1
```

The connection also failed with Mongoose

```
yarn
node connect.js

```

### Multiple attempts

- Check the cluster status

It's OK

```
mongosh mongodb://user:pass@127.0.0.1:27017/naxx
rs.status()

```

- Connecting to a single node with mongosh is OK.

```
mongosh mongodb://user:pass@127.0.0.1:27017/naxx
mongosh mongodb://user:pass@127.0.0.1:27018/naxx
mongosh mongodb://user:pass@127.0.0.1:27019/naxx
```

- The master-slave data synchronization of the cluster is OK. After logging into the primary node and inserting a record, then logging into the two secondary nodes, the data inserted by the primary node is successfully synchronized.

```
mongosh mongodb://user:pass@127.0.0.1:27017/naxx
db.students.insertOne({ name: 'happy' })
mongosh mongodb://user:pass@127.0.0.1:27018/naxx
db.students.find()
```

- Inside the containers of each node, it is possible to connect to the databases of the other two nodes.

```
docker ps
docker exec -it 24147a63b33f /bin/bash
mongosh mongodb://user:pass@mongo_1:27017/naxx
mongosh mongodb://user:pass@mongo_2:27017/naxx
mongosh mongodb://user:pass@mongo_3:27017/naxx
```

So where is the problem? I am very puzzled and out of ideas. I am calling for help and will be very grateful for any assistance.
