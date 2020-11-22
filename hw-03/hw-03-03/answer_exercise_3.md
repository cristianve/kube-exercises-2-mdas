# Horizontal Pod Autoscaler NGINX 📦🌐




### Horizontal Pod Autoscaler

* Crea un objeto de kubernetes HPA, que escale a partir de las métricas CPU o memoria (a vuestra elección). Establece el umbral al 50% de CPU/memoria utilizada, cuando pase el umbral, automáticamente se deberá escalar al doble de replicas.

`` kubectl create -f service1.yml``

![service1](./imatges/service1.PNG)  
