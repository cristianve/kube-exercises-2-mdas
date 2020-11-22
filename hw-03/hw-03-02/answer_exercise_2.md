# Replicaset NGINX 📦🌐


## Creacion replicaSet:
* Especificamos de forma declarativa (fichero YML) las 3 replicas:

![POD](./imatges/replicaSet.PNG)  


## Lo ejecutamos con el siguiente comando:

`` kubectl apply -f nginx-ReplicaSet.yml ``

![POD](./imatges/apply.PNG)  


## Escalar 10 replicas:

`` kubectl scale --replicas=10 rs nginx-server ``

![POD](./imatges/scale.PNG)  

## Si necesito tener una replica en cada uno de los nodos de Kubernetes, ¿qué objeto se adaptaría mejor?

* Un objeto de tipo Deployment porque con el nodeSelector: podemos definir el numero de pods por nodo con la ayuda de affinity/nodeAffinity.