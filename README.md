# README - Configuração de Cluster MongoDB com Docker

Este guia contém todos os comandos necessários para configurar e testar um cluster MongoDB utilizando Docker.

## Criando o cluster:
```sh
docker network create mongoCluster
```

## Criando os nós do MongoDB:
```sh
docker run -d --rm -p 27017:27017 --name mongo10 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip_all
docker run -d --rm -p 27018:27017 --name mongo20 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip_all
docker run -d --rm -p 27019:27017 --name mongo30 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip_all
docker run -d --rm -p 27020:27017 --name mongo40 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip_all
docker run -d --rm -p 27021:27017 --name mongo50 --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip_all
```

## Inicializando o Replica Set:
```sh
docker exec -it mongo10 mongosh
```
Dentro do terminal do MongoDB:
```sh
rs.initiate({
  _id: "myReplicaSet",
  members:[
    {_id:0, host: "mongo10:27017"},
    {_id:1, host: "mongo20:27017"},
    {_id:2, host: "mongo30:27017"},
    {_id:3, host: "mongo40:27017"},
    {_id:4, host: "mongo50:27017"}
  ]
})
exit
```

## Verificando o status do Replica Set:
```sh
docker exec -it mongo10 mongosh --eval "rs.status()"
```

## Conectando via MongoDB Compass:
1. Abra o **MongoDB Compass**
2. Clique em **[+ Add New Connection]**
3. Cole a URL do cluster
4. Clique em **[Save & Connect]**

## Inserindo dados no banco:
```sh
use CorporeSystem;

db.cliente.insertOne({codigo:1, nome: "Ana Maria"});
db.cliente.insertOne({codigo:2, nome: "Maria Jose"});
db.cliente.insertOne({codigo:3, nome: "Jose Silva"});
db.cliente.insertOne({codigo:4, nome: "Luis Souza"});
db.cliente.insertOne({codigo:5, nome: "Fernanda Silva"});
```

## Consultando os dados inseridos:
```sh
db.cliente.find()
```

## Testando a resiliência do cluster:
### Derrubando o nó primário:
```sh
docker stop mongo10
```

### Verificando a eleição de um novo primário:
```sh
docker exec -it mongo20 mongosh --eval "rs.status()"
```

### Derrubando dois nós secundários:
```sh
docker stop mongo20 mongo30
```

## Criando um nó árbitro:
```sh
docker run -d --rm --name mongoArbiter --network mongoCluster mongodb/mongodb-community-server:latest --replSet myReplicaSet --bind_ip_all
```

## Adicionando o árbitro ao Replica Set:
```sh
docker exec -it mongo10 mongosh --eval 'rs.addArb("mongoArbiter:27017")'
```

## Verificando a configuração do Replica Set:
```sh
docker exec -it mongo10 mongosh --eval "rs.conf()"
```