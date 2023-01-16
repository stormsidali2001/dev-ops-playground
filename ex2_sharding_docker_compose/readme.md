### create a replica set composed of 2 servers
```
docker-compose exec cfgsvr1 mongosh
rsconf = {
    _id:"rsconfig",
    configsvr:true,
    members:[
        {"_id":0,"host":"cfgsvr1:27017"},
        {"_id":1,"host":"cfgsvr2:27017"}

    ]
}

```

### create a replica set for the first shard
```
docker-compose exec shard1svr1 mongosh
rsconf = {
    "_id":"shard1rs",
    members:[
        {"_id":0,"host":"shard1svr1:27017"},
        {"_id":1,"host":"shard1svr2:27017"}

    ]
}
rs.initiate(rsconf)
```
### create a replica set for the second shard

```
docker-compose exec shard2svr1 mongosh
rsconf = {
    "_id":"shard2rs",
    members:[
        {"_id":0,"host":"shard2svr1:27017"},
        {"_id":1,"host":"shard2svr2:27017"}

    ]
}
rs.initiate(rsconf)

```

### adding the shards servers to the router mongos
```
docker-compose exec mongos mongosh
```
> shard1:
```
sh.addShard("shard1rs/shard1svr1:27017")
sh.addShard("shard1rs/shard1svr2:27017")
```
> shard2
```
sh.addShard("shard2rs/shard2svr1:27017")
sh.addShard("shard2rs/shard2svr2:27017")
```

> allow sharding on the database
```
sh.enableSharding("dblp");
```
> define the distribution key (type + year)
```
db.adminCommand(
{ shardCollection: "dblp.publis", key: {type:"hashed", year: 1 } } )
```
> importing data to the router
```
docker cp dblp.json mongos:/dblp.json

docker exec mongos mongoimport -d dblp -c publis /dblp.json
```

> show some stats

```
Sh.status() or db.printShardingStatus()
odb.publis.getShardDistribution();
```