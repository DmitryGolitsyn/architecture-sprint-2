# pymongo-api

## Как запустить

Запускаем mongodb и приложение

```shell
docker compose up -d
```

# Инициализируем и заполняем mongodb данными

### Инициализируем серверов конфигов

```shell
docker exec -it configsrv1 mongosh --port 27017

rs.initiate({
  _id: "config_server",
  configsvr: true,
  members: [
    { _id: 0, host: "configsrv1:27017" },
  ]
});

exit();
```


### Инициализируем сервера шардов

```shell
docker exec -it shard1-1 mongosh --port 27018
rs.initiate({
  _id: "shard1-1",
  members: [
        { _id: 0, host: "shard1-1:27018" },
        { _id: 1, host: "shard1-2:27028" },
        { _id: 2, host: "shard1-3:27038" },
    ]
});
exit();
````
```shell
docker exec -it shard2-1 mongosh --port 27019
rs.initiate({
  _id: "shard2-1",
  members: [
        { _id: 0, host: "shard2-1:27019" },
        { _id: 1, host: "shard2-2:27029" },
        { _id: 2, host: "shard2-3:27039" },
    ]});
exit();
```

### Добавление шардов в кластер

```shell
docker exec -it mongos1 mongosh --port 27020

sh.addShard("shard1-1/shard1-1:27018");
sh.addShard("shard2-1/shard2-1:27019");

sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" });

use somedb
for(var i = 0; i < 1000; i++) db.helloDoc.insertOne({age:i, name:"ly"+i})
exit();
```

### Проверка шардирования

```shell
docker exec -it shard1-1 mongosh --port 27018
use somedb;
db.helloDoc.countDocuments();
exit();
```
```shell
docker exec -it shard2-1 mongosh --port 27019
use somedb;
db.helloDoc.countDocuments();
exit();
```
```shell
docker exec -it mongos1 mongosh --port 27020
use somedb;
db.helloDoc.countDocuments();
exit();
```

### Если вы запускаете проект на локальной машине

Откройте в браузере http://localhost:8080

