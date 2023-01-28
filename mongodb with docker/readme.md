### Loading the publis collection into the container
```javascript
docker cp /dblp.json mongo-esi:/dblp.json
docker exec mongo-esi mongoimport -d dblp.json -c publis /dblp.json
```