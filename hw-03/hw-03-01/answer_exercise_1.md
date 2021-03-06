# Ingress Controller / Secrets NGINX 📦🌐


## Creacion Ingress Controller / Secrets NGINX :
* [Ingress Controller / Secrets] Crea los siguientes objetos de forma declarativa con las
siguientes especificaciones

````
Imagen: nginx
• Version: 1.19.4
• 3 replicas
• Label: app: nginx-server
• Exponer el puerto 80 de los pods
• Limits:
CPU: 20 milicores
Memoria: 128Mi
• Requests:
CPU: 20 milicores
Memoria: 128Mi

````

### Ingress Controller
````
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  tls:
    - hosts:
        - "cristian.student.lasalle.com"
      secretName: nginx-secret
  rules:
    - host: "cristian.student.lasalle.com"
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              serviceName: nginx-server-service
              servicePort: 80

````

### ReplicaSet 
````
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
  labels:
    app: nginx-server
    namespace: default

spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-server
  template:
    metadata:
      labels:
        app: nginx-server
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.4

````

### Deployment 
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-server
spec:
  selector:
    matchLabels:
      run: nginx
  replicas: 3
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.4
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 20m
            memory: "128Mi"
          requests:
            cpu: 20m
            memory: "128Mi"
            
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-server-service
spec:
  type: NodePort
  ports:
    - name: http
      targetPort: 80
      port: 80
      nodePort: 31717
````

## Ingress addon install:

* Para poder utilizar el objecto ingress lo añadimos a minikube addons:

``minikube addons enable ingress ``

![IngressAddon](./imatges/ingressaddon.PNG)  


## Generating a Self-Signed Cercificate (OpenSSL):

``openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes -subj "/CN=cristian.student.lasalle.com/O=cristian.student.lasalle.com" ``

![OpenSSL](./imatges/openssl.PNG)  


### Secret 
````
apiVersion: v1
kind: Secret
metadata:
  name: nginx-secret
type: kubernetes.io/tls
data:
  tls.crt: 
     MIIFLzCCAxegAwIBAgIUCAaDVRr44Vq9XFWgcu+lsQvTwIAwDQYJKoZIhvcNAQEL BQAwJzElMCMGA1UEAwwcY3Jpc3RpYW4uc3R1ZGVudC5sYXNhbGxlLmNvbTAeFw0y MDExMjIxODQxNTZaFw0yMTExMjIxODQxNTZaMCcxJTAjBgNVBAMMHGNyaXN0aWFu LnN0dWRlbnQubGFzYWxsZS5jb20wggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIK AoICAQDJ9wI+LVnvi6eQkq2/QFgxnPyOJvbCaR9+pBdm73EjH1wglfnm4de9z5t7 ARmKiFk2ovK5m+ofMwuFhQRZ3ghA515uIlAWGRS5wRdhrAqfxYRr8MhHNkAfQKzq FqnRLWsWwz0Sm3vZqtNYY3v76cYfLX0BJfINyB4fQvN3IBITVOaxXzgYDEC0HW4T PbOkQsIgpcEKK+0VR6wIruQbP6RLWdE2aj7rB/mRXyRo6aR6ZFsHpdwwfYMUZGKm HAfqjNVQ4xHR5hdRnh84Svc6MaF4dE41/LWMv50Fad+ZR5TzkQspqOx2qZ0hZQ5F SA1yYm+eUHlJUr5bnqMF34eo9mER9ob0kZMi8yAf/P9SXnHx9BEpaYGuWZPc6UjG 4Xh+xG3Ris/3BS5MkzaVox0QqEZ2XP7U7ieHO8R0P/3CYDftBEqPdY6K63hhfpfl TJmCjMJEGhi8z/WC/i1RZe++63sE6Y064PEb9PlDgqSq06OditLrX2UCgtdPn+UX QwlB65agQZ22ndov/rsDEhERep++PvgPcBqIP5mfA6kFPTZHRgp5fLiJGteohal4 Mv2zBlnONMjkkSHPEZGIQI0OIu2FBJ/z5z6Zh6y9VwhPoKt0NetV4KfUPcNKtaKa 1YILEfRrPI7NsfuqcM3GWIJO/n6Bu9Pwy37JNXLkLgaKoXGtdwIDAQABo1MwUTAd BgNVHQ4EFgQUcUCDbukdKLZu50KEBGKeA8grs0wwHwYDVR0jBBgwFoAUcUCDbukd KLZu50KEBGKeA8grs0wwDwYDVR0TAQH/BAUwAwEB/zANBgkqhkiG9w0BAQsFAAOC AgEATqJJc9PgvAjdDeFut75EJ4yQ7Eq97rQhYw1gPXb3JnAzUvVor2lMRvxgB3TD L+9xwBqXo1IQVgCDtyUTstPe9MlpJvUiRVclKIU977kUpLZoQf6VKCxfmts7Kg8s 1Dg5+DpEBgJ7yXg/dS95SAP8WuyniLc3HIr8eZ6nGbf5ZWFKiKda5EA3UJBVE7T9 eaeLWZDWaCTn03Zx2d0q5zPjqHY4oqr9nQwwpKNuOfxQC3f4dxkNnNpEtFLLgf5q FWrI+NLTywNpA6OxEFdmF03zCk9cyzGo5/2gGsOXK2ruw2g5ER2qFWvZ6rF+l0ei gVReyFI3w/RdJDDKNJ891AHBANFomgJ7LfjxWZTBezv1xa+109bKvNV+GEF9JCvt L5p2xp6rPKtL+gzPdo2eWsPLpmsddcTAUolmq4R5nirWr4oTOJic0qRLhzKyhTbv p0MVCYQg/WIgG7zwFXn2UFV4aoas9/E3p1gzJvN5JmH3hyCQqO2AcnZ5whsFIL6L 8ewmnmIjE8vqw9Y+CoQTW+iEdTDVEPyyaVc0yw5CzG5gik0Ne2OnQVRfqaLJ46T/ uSGadHKf4fEhLLq6DisDm+biTM6oIwr+d72OhfYEOGis1gI5GM/BxMXHPep9gZFK 6GShnw5RKoIx0VojpnuCG7nVrU3VtWKVZBOyVN+JOg4DURU=
  tls.key: 
    MIIJnDBOBgkqhkiG9w0BBQ0wQTApBgkqhkiG9w0BBQwwHAQIioZOwKPtL7YCAggA MAwGCCqGSIb3DQIJBQAwFAYIKoZIhvcNAwcECOHW4Ip5qYezBIIJSCOHRCfghgNp 3FG9mvkDDmFU+u64gsjuT6i3Jy38S05qO4dw2DUJSZtoAr7ipzZ/pHdI7Dt+310Z Ylf/InxHEVppyVXkI8vIXdWQMiT5RJEpiff46lcgjwvtFiEGmj/1p2IshqLrdjzz 4gygEH/2JjDan98c0KJjQPNloKWmpHmKpN6Gzqu0okgS8S28erkx39ba6u24Jrm2 uHUlaSDX4HzM/MgZQxhUyDEOp85Ms5v1UbbWJX+QD1+8fDF44FXrRI+a02Z3OJkz R1ssU+hVRY3Q2QBROkse0wUK+BQDqVKXNj1hqAT3Ume+Nxulv/AN1lCA4zWbd1Sf 1aY+fH5YsTyQr98k7TTmrDMOhveUK9Dqc005pln7tHXneRl+SIojOfg8fiIlzBYR LRMjwrF8yoWH/UyznGh5DdKtiLXI1HLbX4KlSpWqkr5gOWf7jRe7PwVrOex4t0dR +Gs40P59Or5pOwzqdk5cacQzsuxjFax2q0gYysFKLfEslu9CXH0bB153ORHu0X+X BUYg88yzUxzG0U3S2OkBG3p/NoS4xHM+40P5w9+NHI7qE67kFUBQzpg9dfKPm/Rn o4EmDgPcvIzX+oe9iC/M1x1bAaLj1WKa6FkfwbZ8/qK0jBNJuo4qnF4ULZFbXStn 9YUOaqjgSIAEyr+DTlMcfSej+v4AoWT6NqpSTcJ0UlvgFPl4B58Es47jdnigxfSK w4UzBKj7eAN6U7058AcBfWqhPfK7UOYSEh9CXPieq5mR71gho3PCrurncPqC1jpD fG+WCcL6mgB3ihWxa508BwVNPmDdLZsoh+i2gCCz9Y3SkJt8pzBEhf26ShNc2gNQ QECiqZ57kWhlaxZGQ1r+/qzWoloNzxMQ/YAK6kjIRb33NgmYyVxs0uLIb6W+tCqc H1dasraBOeWj/Hhoy5LDPVw6c+RNnfCDMgycS845bJzW/1yWyMyzi5CiQPSPIWeA xjAgO/LGRBX2E8dkk501YrKttFztKKxSDNfSP4Hp1yDhjNIqsm1JG8knH4EIJ4to fJ5EwYbdJwTkdn4gCxOxwJHrp4MftjOijcEIuGLuw/tkhyRnKEEvGyPXHGe3uOu9 T9nvIODbUyJVDntZ78ujNcpmAJwRWUxXe9/vr3M5P7mO+ni7iVx49bwm2GHxVaJ+ p0ZFC2qOjF7XW0zLyL+NqdGQale9i8PTPvqsTAM5WgXmTLOFuDwnPgqi1tSnCQSd qrc/QJEDPeNy91Zho6X4u+aPsmn6QmQPDb79ORA7cFUnbkKk54+qYC8hbTtSsc8f dMwUiXP1j+QVgNlwDvIDbxb9N1mMl9Y5TdiD7f5UCi55lhWB1SD7otwtLFT/ct2M U2gakCVJ7uNtxjlLX4rblbZ5jYty8rKEafRiToCMjUNl2JNwjzoJ50bCRRZqaOx5 veq5xd9SUD+RpSz+XUTtiVVaBd81VkndPnR7bnYvs41gmOh7AnR7GmbGWnpE203M rANsJYMx49H1//jqGhxrytj5r6S4rOiM76Bw7zHNaovShTY+n6HFRAj3Yb9ItQG/ U20LfZBbaWqxdKgOIo+7P8sfL8R3zk3u6BVxhzxgEJCd6XlGSCA6p+OdMjRD4dde cO4kdJT83GrPRvYlpwR4xrYNk1tkhgLJsNQ7V139nQyLOfOu8m9l6OX76bIFd/ke mN7GMB7Sr1qP6Nniz8PxNhgM6PTPpAqL6j/RSebzLjQiQ7U5EKzeq9oCp8RdO0Ky 4QFlybI+2oEfCqjsPxtfW0pNabNi8vsrfA3MfH+UhTlw7jKuW4STZr3JegzGlkqj R35BFaDgH1nY/fxw4+9F/AFUlU4aIBNZ6bBjeTIZvuBX/D9qYcmdvPw5WolyfoRM k+BjO0bDAI4mMpq1QdN5hfdK2caEmI5z2Pn5gw3QI2aQLMagKq//3RIIgquEMi3j tKIQxkMfAvl3V46vUb0/Jzcmqxft5S9BGLshFPTxskNNxmeKWO7laf3SNMPu1bp2 E8UaM/5fubP9Rq5/IAOGnCkcdwwDRDim3a6vlgnNjadQtPniy6PSwEsOH2Vz3EbG OfypURLglStxPUtpeWc5/W5bbZQMT5QUss9nWvmvsarmYWgIrTjBgsYLOwS+5rsg r0eR+APFqor2l5Q2+jKdr5vDW9wZO0e3GFmiDlsfqhjeA7LQ7aqaeV/WOXneVVED flRIdQazE22hbUEtATIkITzFvgzFT+VxeUl6i8PTSonPhFbywMRFceR4Q7EuxZtr YgzKFjixgB7UF8M0IRc4M1no/ROgX18E7nEjx/Mknj3HwoCkMXBqzwko+yWgO9o1 E5Kfj9tEtHikqcbnMrhcvWTswWBLrkLf+PKZ/k0p0PWj1Srzd5r/pRAm9BS7kMQ1 BUM43aAQv1MpLu7hl+eiVvUAuoQp5yl7NX8sQe+kwKSIBM1tU7gq/XQuAKXju83U 4h797WHaDKdriSi74R9HM7dpnkQ+Q7l08vogMQxNT2utq9Ufr6L0ltNGgi8CsY1m Pc9lT9DkA+HhXP5gHawcCBPfGljysxVro0MmsN69/ZwQGIH8ScjpN6wGW1NECY69 gIpAqWn18lQ3UMzkO//jzFxlomYQvIqwlaHLl4X/7kyZMAuLrCTzel65pIkMQxSv LDCBmaeayNb9dSk60xhyO3R02dKY5t0djoGCpCvB6+CaN7PnyFa0phQWha9wMp2n IONFc57CPARMYcpDsaITf9U/pZuXHtyRRYiDXkZqdDzlkJE5uI/xAq+QWxBilQTQ SzsRIZAHK+Sy3x7ZuRWKOkzfWD19U8mAsCcib7a2VrzLGphZU+OYnEos/X1OszlO S7Yd828CdyaVZUWIQfEDIzYKX5Fk41UqV0ifkupI+CcbCdKFiGkulKM7fPJOHM23 0vyz5CPgVKJ8LkXQDWhkXRct/gWFOS5VGMWZdEuyLVnwvle9WgaH/NcfhiWGpQwj AzeDWo5Rf3Iv9NpHPIV0fCk8cftcohVdSixAz+HT/91VwYZPVFlMAC1/qYAlD7D6 1c9VLPeJuelg4lLAKO7PerdXeTPNs4BWpxZWOX+nxlPinoxbj2TWPIMBwzovwfxt mQpjGvJp5HbgrJ0gP6/R0c63D8GHg/x0vqzgma/YRfRIwgp/QKF1GIk+Z+uYD97x y7/VCYs6O2joPjBLME5yBA==
    
````
![secret](./imatges/secret.PNG)  
## Deployment All:


``kubectl apply -f ReplicaSet.yml ``
``kubectl apply -f Secret.yml ``
``kubectl apply -f Ingress.yml ``
``kubectl apply -f deployment.yml ``



