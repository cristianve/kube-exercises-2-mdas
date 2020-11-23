# StatefulSet NGINX 📦🌐


## Creacion replicaSet:
* Especificamos de forma declarativa (fichero YML) las 3 replicas:
``` kubectl apply -f StatefulSet.yml ```


## Setup Replicaset

``` kubectl exec -it mongod-0 -c mongod-container-app bash ```

## Create Admin Rol MongoDB

```

 db.getSiblingDB("admin").createUser({
...       user : "demoadmin",
...       pwd  : "demopwd123",
...       roles: [ { role: "root", db: "admin" } ]
...  });

```

## Initiate mongodb

rs.initiate({ _id: "MainRepSet", version: 1, 
members: [ 
 { _id: 0, host: "mongod-0.mongodb-service.default.svc.cluster.local:27017" }]});
{
	"ok" : 1,
	"$clusterTime" : {
		"clusterTime" : Timestamp(1601481520, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	},
	"operationTime" : Timestamp(1601481520, 1)
}


![POD](./imatges/replicaSet.PNG)  


