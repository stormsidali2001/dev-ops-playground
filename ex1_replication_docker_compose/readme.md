```
1. we will create a replicat set containing 4 servers (1 primary , 2 secondary , 1 arbiter)
2. 
* go inside  mongodb1 container , connect as a client using mongosh : 
```
docker-compose exec mongodb1 mongosh
```
* intialize the rsconf varaible
```javascript
rsconf = {
    _id:"rsmongo",
    members:[
        {"_id":0,"host":"172.17.0.1:27017"},
        {"_id":1,"host":"172.17.0.1:27018"},
        {"_id":2,"host":"172.17.0.1:27019"},
        {"_id":3,"host":"172.17.0.1:27020","arbiterOnly":true}
    ]
}

rs.initiate(rsconf)
```

* type the below command to display the cluster status (containers ...)

```
rs.status()
```

* copy the dbp.json file into the mongodb1 container
```
sudo docker cp dblp.json mongodb1:/dblp.json
```

* import the collection **publis** (in dblp database) to mongodb1 container's mongodb instance
```
sudo docker exec mongodb1 mongoimport -d dblp -c publis /dblp.json

```

* re-enter into the container's mongodb1 (primary) server) and connect to mongo instance as a client using mongosh
> switching to dblp database
> 118015 documents should be present there
```
docker-compose exec mongodb1 mongosh
use dblp;
db.publis.countDocuments()
```

* now connect to mongodb1 container's mongosh  
```
docker-compose exec mongodb2 mongosh
```
* try showing the number of documents in this mongo instance
```
rsmongo [direct: secondary] dblp> db.publis.countDocuments()
MongoServerError: not primary and secondaryOk=false - consider using db.getMongo().setReadPref() or readPreference in the connection string

```
> An error will be returned as we didn't authorize the server to replicat the data
> To solve this issue lets give it the authorization:
```
rs.secondaryOk()
```
> now the issue is solved
```
db.countDocuments() // 118015
```

* try now to insert a dumy document in the primary server (mongodb1 container) and check out if its replicated on the secondaty server  (mongodb2 container)
```
### mongodb1
rsmongo [direct: primary] dblp> db.publis.insertOne({value:20})
{
  acknowledged: true,
  insertedId: ObjectId("63c5c47268196fe70a30948c")
}
rsmo
### mongodb2
rsmongo [direct: secondary] dblp> db.publis.findOne({value:20})
{ _id: ObjectId("63c5c47268196fe70a30948c"), value: 20 }

```

* connect to the cluster using mongocompas (by connecting to the primary server):  
```
mongodb://localhost:27017/?replicaSet=rsmongo
mongodb://localhost:27017
```