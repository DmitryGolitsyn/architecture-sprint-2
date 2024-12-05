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
docker exec -it shard1 mongosh --port 27018

rs.initiate({
  _id: "shard1",
  members: [{ _id: 0, host: "shard1:27018" }]
});

exit();


docker exec -it shard2 mongosh --port 27019

rs.initiate({
  _id: "shard2",
  members: [{ _id: 0, host: "shard2:27019" }]
});

exit();
```

### Добавление шардов в кластер

```shell
docker exec -it mongos1 mongosh --port 27020

sh.addShard("shard1/shard1:27018");
sh.addShard("shard2/shard2:27019");

# Включение шардирования
sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" });

# Заполнение БД
use somedb
for(var i = 0; i < 1000; i++) db.helloDoc.insertOne({age:i, name:"ly"+i})
exit();
```

### Проверка шардирования

```shell
docker exec -it shard1 mongosh --port 27018
use somedb;
db.helloDoc.countDocuments();
exit();

docker exec -it shard2 mongosh --port 27019
use somedb;
db.helloDoc.countDocuments();
exit();

docker exec -it mongos1 mongosh --port 27020
use somedb;
db.helloDoc.countDocuments();
exit();
```

### Если вы запускаете проект на локальной машине

Откройте в браузере http://localhost:8080

